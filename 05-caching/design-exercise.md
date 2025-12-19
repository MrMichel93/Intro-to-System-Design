# Design Exercise: News Website Caching Strategy

## 1. Design Problem

### Problem Statement
Design a comprehensive caching strategy for a large-scale news website like CNN or BBC News. The system must serve millions of users with low latency while minimizing database load and infrastructure costs. News articles are published frequently, updated occasionally, and have varying popularity patterns.

### Context and Constraints
- News articles are published throughout the day (peak: 50-100 articles/hour)
- Breaking news can cause sudden traffic spikes (10-100x normal traffic)
- Most users read recent articles (80% of traffic to articles < 24 hours old)
- Articles rarely change after publication (except breaking news updates)
- Homepage and category pages change frequently
- User comments are added continuously
- Global audience with users in different time zones
- Mobile and desktop clients with different requirements

### Requirements

#### Functional Requirements
- Serve article content (text, images, videos)
- Display homepage with latest articles
- Show category pages (politics, sports, business, etc.)
- Support article search
- Display article comments (real-time or near-real-time)
- Handle breaking news updates
- Support personalized recommendations
- Serve static assets (CSS, JavaScript, images)

#### Non-Functional Requirements
- **Scale**: 50 million daily users, 500 million page views per day
- **Performance**: Page load time < 1 second, Time to First Byte < 200ms
- **Availability**: 99.9% uptime
- **Read/Write Ratio**: 1000:1 (heavily read-dominant)
- **Cache Hit Rate**: Target 95%+ to minimize database load
- **Cost**: Optimize infrastructure costs through effective caching
- **Freshness**: Breaking news updates visible within 30 seconds

## 2. Guided Questions

### Understanding Caching Basics
1. **Why can't we just serve everything from the database?**
   - Hint: Consider the scale - 500 million page views per day
   - Think about: Database capacity, query time, cost per query

2. **What types of data should be cached vs. not cached?**
   - Hint: Consider how often data changes and how important freshness is
   - Think about: Static content, popular articles, user-specific data

3. **Where should we place caches in the architecture?**
   - Hint: Multiple cache layers can work together
   - Think about: Browser, CDN, application server, database

### Cache Design Decisions
4. **How do we keep cached content fresh when articles are updated?**
   - Hint: Articles can be edited after publication
   - Think about: Cache invalidation strategies, TTL

5. **How do we handle traffic spikes during breaking news?**
   - Hint: Thousands of users might request the same article simultaneously
   - Think about: Cache warming, thundering herd problem

6. **What happens when cache is full?**
   - Hint: Need to remove something to make space
   - Think about: Eviction policies (LRU, LFU, TTL)

### Advanced Scenarios
7. **How do we cache personalized content?**
   - Hint: Homepage is different for each logged-in user
   - Think about: Fragment caching, edge-side includes

8. **How do we measure cache effectiveness?**
   - Hint: Need metrics to know if caching is working
   - Think about: Hit rate, latency reduction, cost savings

## 3. Step-by-Step Guidance

### Step 1: Identify Cacheable Content
Categorize content by caching suitability:

**Highly Cacheable** (can cache for hours/days):
- Published article content (changes rarely)
- Article images and media
- Static assets (CSS, JavaScript, fonts)
- User profiles (change infrequently)

**Moderately Cacheable** (cache for minutes):
- Homepage (updates every few minutes)
- Category pages
- Popular articles list
- Search results

**Minimally Cacheable** (cache for seconds or not at all):
- Breaking news updates
- Article comments (real-time)
- User-specific notifications
- View counts, like counts

### Step 2: Design Cache Layers
Multiple cache layers provide cumulative benefits:

```
User Request
    ↓
1. Browser Cache (local)
    ↓ [MISS]
2. CDN Edge Cache (regional)
    ↓ [MISS]
3. Application Cache (Redis)
    ↓ [MISS]
4. Database Query Cache
    ↓ [MISS]
5. Database
```

**Each layer serves a purpose:**
- Browser: Zero latency, no bandwidth cost
- CDN: Low latency, reduces origin load
- Application: Fast access to computed results
- Database Cache: Reduces query execution time

### Step 3: Choose Caching Strategies
Different strategies for different needs:

**Cache-Aside (Lazy Loading)**:
```python
def get_article(article_id):
    # Check cache first
    article = cache.get(f"article:{article_id}")
    if article:
        return article  # Cache hit
    
    # Cache miss - fetch from database
    article = database.query(article_id)
    
    # Store in cache for future requests
    cache.set(f"article:{article_id}", article, ttl=3600)
    return article
```

**Write-Through**:
```python
def publish_article(article):
    # Write to database
    database.insert(article)
    
    # Immediately write to cache
    cache.set(f"article:{article.id}", article, ttl=3600)
    return article
```

**Write-Behind (Write-Back)**:
```python
def update_view_count(article_id):
    # Update cache immediately
    cache.increment(f"views:{article_id}")
    
    # Queue database update (asynchronous)
    queue.publish('update_views', article_id)
    # Background worker will update database
```

### Step 4: Handle Cache Invalidation
"There are only two hard things in Computer Science: cache invalidation and naming things." - Phil Karlton

**Time-Based Invalidation (TTL)**:
```python
# Different TTLs for different content
cache.set("homepage", data, ttl=300)        # 5 minutes
cache.set("article:123", data, ttl=3600)    # 1 hour
cache.set("static:logo.png", data, ttl=86400)  # 24 hours
```

**Event-Based Invalidation**:
```python
def update_article(article_id, new_content):
    # Update database
    database.update(article_id, new_content)
    
    # Invalidate cache
    cache.delete(f"article:{article_id}")
    # Next request will fetch fresh data
```

**Proactive Cache Refresh**:
```python
def refresh_popular_articles():
    popular_ids = get_trending_article_ids()
    for article_id in popular_ids:
        # Refresh cache before TTL expires
        article = database.query(article_id)
        cache.set(f"article:{article_id}", article, ttl=3600)
```

### Step 5: Design for Breaking News
Breaking news requires special handling:

**Problem**: Thousands of requests for the same breaking news article within seconds
**Solution**: Proactive cache warming + short TTL

```python
def publish_breaking_news(article):
    # Mark as breaking news
    article.is_breaking = True
    article.ttl = 30  # Short TTL (30 seconds)
    
    # Write to database
    database.insert(article)
    
    # Warm all cache layers immediately
    cache.set(f"article:{article.id}", article, ttl=30)
    cdn.purge(f"/articles/{article.id}")  # Clear CDN
    cdn.prefetch(f"/articles/{article.id}")  # Pre-load to CDN
    
    # Add to homepage cache
    homepage = cache.get("homepage")
    homepage.breaking_news = article
    cache.set("homepage", homepage, ttl=30)
```

### Step 6: Prevent Thundering Herd
When cache expires, thousands of requests might hit database simultaneously.

**Problem**:
```
Time: 10:00:00 - Cache entry expires
Time: 10:00:01 - 1000 requests arrive simultaneously
               - All see cache miss
               - All query database
               - Database overloaded
```

**Solution 1: Cache Locking**
```python
def get_article_with_lock(article_id):
    article = cache.get(f"article:{article_id}")
    if article:
        return article
    
    # Try to acquire lock
    lock_key = f"lock:article:{article_id}"
    if cache.set(lock_key, "locked", nx=True, ex=10):
        # This request won the lock, fetch from database
        article = database.query(article_id)
        cache.set(f"article:{article_id}", article, ttl=3600)
        cache.delete(lock_key)
        return article
    else:
        # Another request is fetching, wait and retry
        time.sleep(0.1)
        return get_article_with_lock(article_id)
```

**Solution 2: Probabilistic Early Expiration**
```python
import random

def get_article_with_early_refresh(article_id):
    article, ttl_remaining = cache.get_with_ttl(f"article:{article_id}")
    
    if article:
        # Probabilistically refresh before expiration
        # Higher probability as expiration approaches
        refresh_probability = 1 - (ttl_remaining / 3600)
        if random.random() < refresh_probability:
            # Async refresh cache
            background_refresh(article_id)
        return article
    
    # Cache miss
    article = database.query(article_id)
    cache.set(f"article:{article_id}", article, ttl=3600)
    return article
```

### Step 7: Implement Cache Monitoring
Track cache effectiveness:

```python
class CacheMetrics:
    def __init__(self):
        self.hits = 0
        self.misses = 0
        self.latency_cache = []
        self.latency_db = []
    
    def record_hit(self, latency_ms):
        self.hits += 1
        self.latency_cache.append(latency_ms)
    
    def record_miss(self, latency_ms):
        self.misses += 1
        self.latency_db.append(latency_ms)
    
    def get_hit_rate(self):
        total = self.hits + self.misses
        return (self.hits / total * 100) if total > 0 else 0
    
    def get_average_latency(self):
        cache_avg = sum(self.latency_cache) / len(self.latency_cache)
        db_avg = sum(self.latency_db) / len(self.latency_db)
        return {
            'cache': cache_avg,
            'database': db_avg,
            'improvement': ((db_avg - cache_avg) / db_avg * 100)
        }
```

## 4. Sample Solution

### Complete Architecture

```
┌─────────────────────────────────────────────┐
│          User Devices (Browsers)            │
│      Cache: 24 hours (static assets)        │
│      Cache: 5 minutes (HTML pages)          │
└──────────────┬──────────────────────────────┘
               │
┌──────────────▼──────────────────────────────┐
│     CDN (CloudFront / Cloudflare)           │
│  - Static assets: 24 hours cache            │
│  - Article pages: 1 hour cache              │
│  - Homepage: 5 minutes cache                │
│  - Breaking news: 30 seconds cache          │
└──────────────┬──────────────────────────────┘
               │
┌──────────────▼──────────────────────────────┐
│           Load Balancer                     │
└──────────────┬──────────────────────────────┘
               │
    ┌──────────┼──────────┐
    │          │          │
┌───▼───┐  ┌───▼───┐  ┌───▼───┐
│ Web   │  │ Web   │  │ Web   │
│Server │  │Server │  │Server │
│   1   │  │   2   │  │   3   │
└───┬───┘  └───┬───┘  └───┬───┘
    └──────────┼──────────┘
               │
┌──────────────▼──────────────────────────────┐
│     Application Cache (Redis Cluster)       │
│  - Article content: 1 hour                  │
│  - Homepage feed: 5 minutes                 │
│  - User sessions: 24 hours                  │
│  - Search results: 10 minutes               │
└──────────────┬──────────────────────────────┘
               │
┌──────────────▼──────────────────────────────┐
│     PostgreSQL Database (with query cache)  │
│  - Primary + Read Replicas                  │
└─────────────────────────────────────────────┘
```

### Redis Cache Structure

```python
# Article content cache
CACHE_KEY_ARTICLE = "article:{article_id}"
CACHE_TTL_ARTICLE = 3600  # 1 hour

# Homepage cache
CACHE_KEY_HOMEPAGE = "homepage:{user_type}"  # guest/logged_in
CACHE_TTL_HOMEPAGE = 300  # 5 minutes

# Category pages
CACHE_KEY_CATEGORY = "category:{category}:page:{page}"
CACHE_TTL_CATEGORY = 600  # 10 minutes

# Trending articles
CACHE_KEY_TRENDING = "trending:articles"
CACHE_TTL_TRENDING = 300  # 5 minutes

# User session
CACHE_KEY_SESSION = "session:{session_id}"
CACHE_TTL_SESSION = 86400  # 24 hours

# Article view counts (write-behind cache)
CACHE_KEY_VIEWS = "views:{article_id}"
# No TTL - persisted to DB periodically
```

### Implementation: Article Serving

```python
from redis import Redis
from flask import Flask, request, jsonify
import time

app = Flask(__name__)
redis_client = Redis(host='redis-cluster', port=6379, decode_responses=True)

class ArticleCache:
    def __init__(self):
        self.metrics = CacheMetrics()
    
    @app.route('/articles/<int:article_id>')
    def get_article(self, article_id):
        start_time = time.time()
        
        # Try cache first
        cache_key = f"article:{article_id}"
        cached_article = redis_client.get(cache_key)
        
        if cached_article:
            # Cache hit
            latency = (time.time() - start_time) * 1000
            self.metrics.record_hit(latency)
            
            # Increment view count (write-behind)
            redis_client.incr(f"views:{article_id}")
            
            return jsonify(json.loads(cached_article))
        
        # Cache miss - fetch from database
        article = self.fetch_from_database(article_id)
        
        if not article:
            return jsonify({'error': 'Article not found'}), 404
        
        # Store in cache
        redis_client.setex(
            cache_key,
            3600,  # TTL: 1 hour
            json.dumps(article)
        )
        
        latency = (time.time() - start_time) * 1000
        self.metrics.record_miss(latency)
        
        return jsonify(article)
    
    def fetch_from_database(self, article_id):
        # Query database
        query = """
            SELECT id, title, content, author, published_at, category
            FROM articles
            WHERE id = %s AND published = true
        """
        result = database.execute(query, (article_id,))
        return result.fetchone()
```

### Implementation: Homepage with Fragment Caching

```python
@app.route('/')
def homepage():
    user_type = 'guest' if not request.user else 'logged_in'
    
    # Check cache
    cache_key = f"homepage:{user_type}"
    cached_html = redis_client.get(cache_key)
    
    if cached_html:
        return cached_html
    
    # Build homepage from fragments
    breaking_news = get_breaking_news()  # Cache: 30 seconds
    top_stories = get_top_stories()      # Cache: 5 minutes
    trending = get_trending()            # Cache: 5 minutes
    categories = get_category_previews() # Cache: 10 minutes
    
    # Personalized section (if logged in)
    personalized = None
    if request.user:
        personalized = get_personalized_recommendations(request.user.id)
        # Don't cache personalized content
    
    # Render template
    html = render_template('homepage.html',
                          breaking_news=breaking_news,
                          top_stories=top_stories,
                          trending=trending,
                          categories=categories,
                          personalized=personalized)
    
    # Cache only for guest users (non-personalized)
    if user_type == 'guest':
        redis_client.setex(cache_key, 300, html)  # 5 minutes
    
    return html

def get_breaking_news():
    """Cached for 30 seconds"""
    cache_key = "breaking_news"
    cached = redis_client.get(cache_key)
    if cached:
        return json.loads(cached)
    
    breaking = database.query("""
        SELECT * FROM articles 
        WHERE is_breaking = true 
        ORDER BY published_at DESC 
        LIMIT 3
    """)
    
    redis_client.setex(cache_key, 30, json.dumps(breaking))
    return breaking

def get_top_stories():
    """Cached for 5 minutes"""
    cache_key = "top_stories"
    cached = redis_client.get(cache_key)
    if cached:
        return json.loads(cached)
    
    stories = database.query("""
        SELECT * FROM articles 
        WHERE published_at > NOW() - INTERVAL '24 hours'
        ORDER BY view_count DESC 
        LIMIT 10
    """)
    
    redis_client.setex(cache_key, 300, json.dumps(stories))
    return stories
```

### Implementation: Cache Invalidation

```python
class CacheInvalidator:
    @staticmethod
    def on_article_published(article):
        """Called when new article is published"""
        # Add to cache immediately
        redis_client.setex(
            f"article:{article.id}",
            3600,
            json.dumps(article)
        )
        
        # Invalidate related caches
        redis_client.delete("homepage:guest")
        redis_client.delete("homepage:logged_in")
        redis_client.delete(f"category:{article.category}:page:1")
        redis_client.delete("trending:articles")
        
        # Warm CDN cache
        cdn.prefetch(f"/articles/{article.id}")
    
    @staticmethod
    def on_article_updated(article_id, updates):
        """Called when article is edited"""
        # Invalidate article cache
        redis_client.delete(f"article:{article_id}")
        
        # Purge from CDN
        cdn.purge(f"/articles/{article_id}")
    
    @staticmethod
    def on_breaking_news(article):
        """Called when breaking news is published"""
        # Very short TTL for breaking news
        redis_client.setex(
            f"article:{article.id}",
            30,  # Only 30 seconds
            json.dumps(article)
        )
        
        # Add to breaking news list
        breaking_news = redis_client.get("breaking_news") or "[]"
        breaking_list = json.loads(breaking_news)
        breaking_list.insert(0, article)
        redis_client.setex("breaking_news", 30, json.dumps(breaking_list[:3]))
        
        # Invalidate homepage
        redis_client.delete("homepage:guest")
        redis_client.delete("homepage:logged_in")
        
        # Push notification to connected clients
        websocket_broadcast("breaking_news", article)
```

### Implementation: Write-Behind Cache for View Counts

```python
class ViewCountAggregator:
    """Background worker to persist view counts"""
    
    def __init__(self):
        self.batch_size = 1000
        self.batch_interval = 60  # seconds
    
    def run(self):
        while True:
            self.aggregate_and_persist()
            time.sleep(self.batch_interval)
    
    def aggregate_and_persist(self):
        # Get all view count keys
        view_keys = redis_client.keys("views:*")
        
        if not view_keys:
            return
        
        # Batch process
        updates = []
        for key in view_keys:
            article_id = key.split(":")[1]
            count = int(redis_client.get(key))
            
            if count > 0:
                updates.append((article_id, count))
                
                # Reset counter (atomic operation)
                redis_client.set(key, 0)
        
        if updates:
            # Batch update database
            database.execute_batch("""
                UPDATE articles 
                SET view_count = view_count + %s 
                WHERE id = %s
            """, updates)
            
            print(f"Persisted {len(updates)} view count updates")
```

### CDN Configuration

```nginx
# CloudFront / Cloudflare configuration

# Static assets (CSS, JS, images)
location ~* \.(css|js|jpg|jpeg|png|gif|ico|svg|woff|woff2|ttf)$ {
    cache-control: public, max-age=86400;  # 24 hours
    expires: 1d;
}

# Article pages
location /articles/ {
    cache-control: public, max-age=3600;  # 1 hour
    expires: 1h;
    
    # Vary cache by these headers
    vary: Accept-Encoding, Cookie;
}

# Homepage
location = / {
    cache-control: public, max-age=300;  # 5 minutes
    expires: 5m;
}

# Breaking news (short cache)
location /breaking {
    cache-control: public, max-age=30;  # 30 seconds
    expires: 30s;
}

# API endpoints (no cache)
location /api/ {
    cache-control: no-cache, no-store, must-revalidate;
    expires: 0;
}
```

### Cache Monitoring Dashboard

```python
class CacheMonitoring:
    def get_metrics(self):
        # Redis info
        info = redis_client.info()
        
        metrics = {
            'redis': {
                'connected_clients': info['connected_clients'],
                'used_memory': info['used_memory_human'],
                'used_memory_peak': info['used_memory_peak_human'],
                'hit_rate': self.calculate_hit_rate(info),
                'ops_per_sec': info['instantaneous_ops_per_sec'],
                'evicted_keys': info['evicted_keys'],
            },
            'application': {
                'cache_hits': self.app_metrics.hits,
                'cache_misses': self.app_metrics.misses,
                'hit_rate_percent': self.app_metrics.get_hit_rate(),
                'avg_cache_latency': self.app_metrics.get_avg_cache_latency(),
                'avg_db_latency': self.app_metrics.get_avg_db_latency(),
            },
            'cdn': {
                'bandwidth_saved': self.get_cdn_bandwidth_savings(),
                'cache_hit_ratio': self.get_cdn_hit_ratio(),
            }
        }
        
        return metrics
    
    def calculate_hit_rate(self, redis_info):
        hits = redis_info.get('keyspace_hits', 0)
        misses = redis_info.get('keyspace_misses', 0)
        total = hits + misses
        
        if total == 0:
            return 0
        
        return (hits / total) * 100
    
    def alert_if_needed(self, metrics):
        # Alert if hit rate drops below threshold
        if metrics['redis']['hit_rate'] < 90:
            alert(f"Redis hit rate low: {metrics['redis']['hit_rate']}%")
        
        # Alert if memory usage high
        if info['used_memory'] > info['maxmemory'] * 0.9:
            alert("Redis memory usage > 90%")
        
        # Alert if eviction rate high
        if info['evicted_keys'] > 1000:
            alert(f"High eviction rate: {info['evicted_keys']} keys")
```

### Design Choices Explained

#### Why Multi-Layer Caching?
Each layer provides cumulative benefits:
- **Browser Cache**: Zero cost, instant, but user-specific
- **CDN**: Near-user, reduces origin load by 80-90%
- **Application Cache (Redis)**: Reduces database load by 95%+
- **Database Query Cache**: Speeds up unavoidable DB queries

**Trade-off**: Complexity vs. performance and cost savings

#### Why Different TTLs for Different Content?
Content freshness requirements vary:
- Static assets (24h): Change rarely, expensive to fetch
- Articles (1h): Published and rarely change
- Homepage (5min): Updates frequently with new articles
- Breaking news (30s): Critical to show latest updates

**Trade-off**: Freshness vs. cache effectiveness

#### Why Write-Behind for View Counts?
View counts don't need real-time accuracy:
- Batching reduces database writes from millions to thousands
- 1-minute delay in count updates is acceptable
- Massive cost savings (database write capacity)

**Trade-off**: Slight staleness vs. performance and cost

### Alternative Approaches

#### Alternative 1: Edge Computing (Lambda@Edge)
Generate personalized content at CDN edge:

**Pros**:
- Lower latency (compute near user)
- Can cache partially personalized content
- Reduces origin load

**Cons**:
- More complex
- Limited compute time/memory
- Harder to debug

**When to use**: Global audience, heavy personalization needs

#### Alternative 2: Full-Page Caching with ESI
Cache entire pages, with dynamic fragments:

```html
<html>
  <head>Cache this header</head>
  <body>
    <div>Cache this content</div>
    <esi:include src="/dynamic/user-widget" />
  </body>
</html>
```

**Pros**:
- Very high cache hit rates
- Simple application logic

**Cons**:
- Requires ESI-capable CDN/cache
- Limited flexibility

**When to use**: Content-heavy sites with minimal personalization

## 5. Extension Challenges

### Challenge 1: Add Real-Time Comments
**Requirement**: Display comments in real-time without refreshing

**How would you cache this?**
<details>
<summary>Hints and Solution</summary>

**Challenges**:
- Comments added continuously
- Can't cache comments with article (too dynamic)
- Need to show latest comments
- High read volume on popular articles

**Solution**:

1. **Separate Comment Caching**:
   ```python
   # Cache article content (1 hour)
   article = cache.get(f"article:{article_id}")
   
   # Cache comments separately (30 seconds)
   comments = cache.get(f"comments:{article_id}")
   
   # Combine at render time
   return render(article, comments)
   ```

2. **Pagination + Cache**:
   ```python
   # Cache paginated comments
   page1 = cache.get(f"comments:{article_id}:page:1", ttl=30)
   page2 = cache.get(f"comments:{article_id}:page:2", ttl=60)
   # Older pages cached longer
   ```

3. **Real-Time Updates via WebSocket**:
   ```javascript
   // Client subscribes to article comments
   socket.on('new_comment', (comment) => {
     // Append to page without full refresh
     appendComment(comment);
   });
   ```

4. **Write-Through Cache**:
   ```python
   def post_comment(article_id, comment):
       # Write to database
       db.insert_comment(comment)
       
       # Invalidate comment cache
       cache.delete(f"comments:{article_id}:page:1")
       
       # Broadcast to WebSocket subscribers
       websocket.emit(f"article:{article_id}", comment)
   ```

**Result**: Article stays cached (1 hour), comments update frequently (30s cache + real-time WebSocket)
</details>

### Challenge 2: Personalized Homepage
**Requirement**: Show different homepage for each user based on interests

**How would you cache this?**
<details>
<summary>Hints and Solution</summary>

**Challenges**:
- Can't cache entire homepage (different for each user)
- 50M users = 50M variations
- Still need good cache hit rate

**Solution: Fragment Caching**

```python
def get_personalized_homepage(user_id):
    # Cache common fragments (shared across users)
    breaking_news = cache_get_or_compute(
        "breaking_news", 
        fetch_breaking_news, 
        ttl=30
    )
    
    top_stories = cache_get_or_compute(
        "top_stories",
        fetch_top_stories,
        ttl=300
    )
    
    # Cache user-specific fragments
    user_interests = cache_get_or_compute(
        f"user_interests:{user_id}",
        lambda: fetch_user_interests(user_id),
        ttl=3600
    )
    
    personalized_articles = cache_get_or_compute(
        f"personalized:{user_id}",
        lambda: recommend_articles(user_interests),
        ttl=600
    )
    
    # Compose homepage (fast - all from cache)
    return compose_homepage(
        breaking_news,
        top_stories,
        personalized_articles
    )
```

**Benefits**:
- Common content cached once, shared by all users
- User-specific content cached per user
- High cache hit rate even with personalization
- Fast page load (all fragments from cache)

**Alternative: Edge-Side Includes (ESI)**:
```html
<html>
  <body>
    <!-- Cached globally -->
    <esi:include src="/fragments/breaking-news" ttl="30"/>
    <esi:include src="/fragments/top-stories" ttl="300"/>
    
    <!-- Cached per user -->
    <esi:include src="/fragments/personalized?user_id={{user_id}}" ttl="600"/>
  </body>
</html>
```
</details>

### Challenge 3: Handle Cache Stampede
**Scenario**: Popular article cache expires, 10,000 requests hit simultaneously

**How do you prevent database overload?**
<details>
<summary>Hints and Solution</summary>

**Problem**: Thundering Herd
```
Time 10:00:00 - Article cache expires
Time 10:00:01 - 10,000 requests arrive
              - All see cache miss
              - All query database simultaneously
              - Database overloaded, slow responses
              - Users see timeouts
```

**Solution 1: Request Coalescing**
```python
# Only one request fetches from DB, others wait
class RequestCoalescer:
    def __init__(self):
        self.pending_requests = {}
    
    async def get_article(self, article_id):
        # Check cache
        article = cache.get(f"article:{article_id}")
        if article:
            return article
        
        # Check if someone else is already fetching
        key = f"article:{article_id}"
        if key in self.pending_requests:
            # Wait for other request to complete
            return await self.pending_requests[key]
        
        # Create future for other requests to wait on
        future = asyncio.Future()
        self.pending_requests[key] = future
        
        try:
            # Fetch from database (only this request)
            article = await db.fetch_article(article_id)
            cache.set(key, article, ttl=3600)
            
            # Notify waiting requests
            future.set_result(article)
            return article
        finally:
            del self.pending_requests[key]
```

**Solution 2: Probabilistic Early Refresh**
```python
def should_refresh_early(ttl_remaining, total_ttl):
    """
    Refresh cache before expiration with probability
    that increases as expiration approaches
    """
    # Example: 1 hour TTL, 5 minutes remaining
    # Probability = 1 - (300 / 3600) = 0.916 (91.6%)
    probability = 1 - (ttl_remaining / total_ttl)
    return random.random() < probability

def get_article(article_id):
    article, ttl = cache.get_with_ttl(f"article:{article_id}")
    
    if article:
        # Refresh early if close to expiration
        if should_refresh_early(ttl, 3600):
            # Async refresh, return cached version immediately
            background_task(refresh_article, article_id)
        return article
    
    # True cache miss
    article = db.fetch_article(article_id)
    cache.set(f"article:{article_id}", article, ttl=3600)
    return article
```

**Solution 3: Lock-Based Approach**
```python
def get_article_with_lock(article_id):
    # Try cache first
    article = cache.get(f"article:{article_id}")
    if article:
        return article
    
    # Try to acquire lock
    lock_key = f"lock:article:{article_id}"
    got_lock = cache.set(lock_key, "1", nx=True, ex=10)
    
    if got_lock:
        # This request won the lock
        try:
            article = db.fetch_article(article_id)
            cache.set(f"article:{article_id}", article, ttl=3600)
            return article
        finally:
            cache.delete(lock_key)
    else:
        # Another request has the lock, wait briefly
        time.sleep(0.1)
        # Retry (likely in cache now)
        return get_article_with_lock(article_id)
```

**Solution 4: Stale-While-Revalidate**
```python
def get_article_stale_ok(article_id):
    # Try fresh cache
    article = cache.get(f"article:{article_id}")
    if article:
        return article
    
    # Try stale cache (expired but kept around)
    stale_article = cache.get(f"stale:article:{article_id}")
    if stale_article:
        # Return stale, refresh async
        background_task(refresh_article, article_id)
        return stale_article
    
    # No cache at all
    article = db.fetch_article(article_id)
    cache.set(f"article:{article_id}", article, ttl=3600)
    # Keep stale copy
    cache.set(f"stale:article:{article_id}", article, ttl=7200)
    return article
```
</details>

### Challenge 4: Multi-Region Caching
**Requirement**: Serve users globally with low latency

**How do you handle cache consistency across regions?**
<details>
<summary>Hints and Solution</summary>

**Challenges**:
- Users in US, Europe, Asia need low latency
- Article updates must propagate globally
- Cache invalidation across regions
- Network latency between regions

**Solution**:

1. **Regional Redis Clusters**:
   ```
   [US Redis] --- [EU Redis] --- [Asia Redis]
        ↓              ↓              ↓
   [US Users]    [EU Users]    [Asia Users]
   ```

2. **Cache Invalidation via Pub/Sub**:
   ```python
   def update_article(article_id):
       # Update database (replicated globally)
       db.update_article(article_id)
       
       # Publish invalidation message
       pubsub.publish('cache_invalidate', {
           'type': 'article',
           'id': article_id
       })
   
   # Each region subscribes
   def on_invalidate_message(message):
       if message['type'] == 'article':
           cache.delete(f"article:{message['id']}")
   ```

3. **Active-Active Replication**:
   ```python
   # Write to local cache
   local_cache.set(f"article:{id}", article, ttl=3600)
   
   # Async replicate to other regions
   for region in other_regions:
       async_replicate(region, f"article:{id}", article)
   ```

4. **Eventual Consistency Strategy**:
   - Accept brief inconsistency between regions
   - Breaking news: Active invalidation (30 sec max delay)
   - Regular articles: Lazy invalidation (let TTL expire)
   
5. **Smart Routing**:
   ```python
   def get_article(article_id, user_region):
       # Try local cache first
       cache_key = f"article:{article_id}"
       article = local_cache.get(cache_key)
       
       if article:
           return article
       
       # Try nearby region caches
       for nearby_region in get_nearby_regions(user_region):
           article = fetch_from_region(nearby_region, cache_key)
           if article:
               # Cache locally for future requests
               local_cache.set(cache_key, article, ttl=3600)
               return article
       
       # Fetch from database
       article = db.fetch_article(article_id)
       local_cache.set(cache_key, article, ttl=3600)
       return article
   ```

**Trade-offs**:
- Consistency vs. Latency
- Cost (multi-region infrastructure) vs. Performance
- Complexity vs. Reliability
</details>

### Challenge 5: Cache Cost Optimization
**Requirement**: Reduce caching costs by 60% while maintaining performance

**What would you change?**
<details>
<summary>Hints and Solution</summary>

**Current Costs** (example):
- Redis Cluster: $500K/year (10 nodes, high memory)
- CDN: $300K/year (bandwidth)
- **Total**: $800K/year

**Optimization Strategies**:

1. **Optimize Cache Storage (-40% Redis cost)**:
   ```python
   # Before: Store full HTML
   cache.set("article:123", full_html_5mb)  # 5MB
   
   # After: Store compressed JSON
   import gzip
   compressed = gzip.compress(json.dumps(article).encode())
   cache.set("article:123", compressed)  # 500KB (10x smaller)
   ```
   - Savings: $200K/year

2. **Tiered Caching (-30% Redis cost)**:
   ```python
   # Hot tier (Redis): Recent popular articles (1% of content)
   # Warm tier (Memcached): Recent articles (9% of content)  
   # Cold tier (Database): Old articles (90% of content)
   
   def get_article(article_id):
       # Try hot cache (Redis)
       article = redis.get(article_id)
       if article:
           return article
       
       # Try warm cache (Memcached - cheaper)
       article = memcached.get(article_id)
       if article:
           return article
       
       # Fetch from database
       article = db.fetch(article_id)
       
       # Determine tier based on popularity
       if is_popular(article_id):
           redis.set(article_id, article, ttl=3600)
       else:
           memcached.set(article_id, article, ttl=7200)
       
       return article
   ```
   - Savings: $150K/year

3. **CDN Optimization (-40% CDN cost)**:
   - Aggressive compression (Brotli instead of gzip)
   - Image optimization (WebP format, lazy loading)
   - Longer TTLs for static assets
   - Savings: $120K/year

4. **Smart Eviction Policy**:
   ```python
   # Instead of LRU, use weighted LFU
   # (evict based on frequency and recency)
   
   class SmartEviction:
       def calculate_value(self, item):
           frequency = item.access_count
           recency = time.now() - item.last_access
           size = item.size_bytes
           
           # Higher score = more valuable
           score = (frequency / recency) * (1000 / size)
           return score
       
       def evict(self):
           # Evict lowest scoring items
           items = sorted(cache.items(), key=self.calculate_value)
           for item in items[:100]:  # Evict batch
               cache.delete(item.key)
   ```
   - Better cache hit rate with same memory
   - Savings: $50K/year (can reduce cluster size)

5. **Request Collapsing**:
   ```python
   # Prevent duplicate requests to origin
   # (multiple users requesting same uncached article)
   # Reduces origin load and data transfer
   ```
   - Savings: $30K/year

**Total Savings**: $550K/year (69% reduction) ✓

**Trade-offs**:
- More complexity (multiple cache tiers)
- Slightly higher latency (compression overhead)
- More monitoring needed
</details>

---

## Summary

This design exercise demonstrates how to build an effective caching system by:
- Implementing multi-layer caching for cumulative benefits
- Using appropriate TTLs based on content freshness needs
- Handling cache invalidation properly
- Preventing cache stampedes
- Monitoring cache effectiveness
- Optimizing for cost while maintaining performance

**Key Takeaways**:
1. **Cache at multiple layers**: Browser, CDN, application, database
2. **Different content needs different strategies**: Static vs. dynamic, popular vs. niche
3. **Invalidation is hard**: Use TTL + event-based invalidation
4. **Monitor everything**: Hit rate, latency, costs
5. **Plan for failures**: Thundering herd, cache stampede
6. **Optimize continuously**: Compression, tiering, eviction policies
