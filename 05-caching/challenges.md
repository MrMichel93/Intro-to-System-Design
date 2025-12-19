# Caching - Challenges & Questions

Test your understanding of caching concepts!

## 🎯 Multiple Choice Questions

### Question 1
Which of these should you ALWAYS cache?

A) User's bank account balance  
B) Current stock prices  
C) Website logo images  
D) Live sports scores  

<details>
<summary>Answer</summary>

**C) Website logo images**

Static content like logos rarely changes and is accessed frequently, making it perfect for caching. The other options need to be up-to-date in real-time.
</details>

---

### Question 2
Your cache hit rate is 30%. What does this mean?

A) Your cache is very effective  
B) 70% of requests are going to the database  
C) You should remove the cache  
D) Your cache is too large  

<details>
<summary>Answer</summary>

**B) 70% of requests are going to the database**

A 30% hit rate means only 30% of requests are served from cache, while 70% miss and go to the database. A good hit rate is 80%+ so you should investigate why the hit rate is low.
</details>

---

### Question 3
What is a "cache stampede"?

A) When the cache server crashes  
B) When many requests hit the database simultaneously after cache expires  
C) When cache memory is full  
D) When you have too many cache servers  

<details>
<summary>Answer</summary>

**B) When many requests hit the database simultaneously after cache expires**

A cache stampede (or thundering herd) happens when a popular cached item expires and many requests try to fetch it from the database at the same time, overwhelming the database.
</details>

---

### Question 4
Which cache eviction policy removes the item that hasn't been used for the longest time?

A) LFU (Least Frequently Used)  
B) FIFO (First In, First Out)  
C) LRU (Least Recently Used)  
D) Random  

<details>
<summary>Answer</summary>

**C) LRU (Least Recently Used)**

LRU removes the item that hasn't been accessed recently, assuming recently accessed items are more likely to be accessed again.
</details>

---

### Question 5
In the Cache-Aside pattern, when do you write to the cache?

A) Before writing to the database  
B) After reading from the database (on cache miss)  
C) Never, the database updates the cache  
D) Every hour  

<details>
<summary>Answer</summary>

**B) After reading from the database (on cache miss)**

In Cache-Aside (lazy loading), you check the cache first. On a miss, you fetch from the database and then store in the cache for future requests.
</details>

---

### Question 6
What is the main advantage of Write-Back caching over Write-Through?

A) Better data consistency  
B) Faster write operations  
C) No risk of data loss  
D) Simpler implementation  

<details>
<summary>Answer</summary>

**B) Faster write operations**

Write-Back writes to cache immediately and to the database later (asynchronously), making writes very fast. However, it risks data loss if the cache fails before the database write happens.
</details>

---

## 🧩 Scenario-Based Challenges

### Challenge 1: Cache Strategy Selection

For each application, choose the best caching strategy and explain why:

**Application A**: News website with articles that update occasionally
- Strategy: _______________
- TTL: _______________
- Why: _______________

**Application B**: E-commerce product catalog (prices change frequently)
- Strategy: _______________
- TTL: _______________
- Why: _______________

**Application C**: User profile data (name, email, preferences)
- Strategy: _______________
- TTL: _______________
- Why: _______________

<details>
<summary>Sample Answers</summary>

**Application A: News Website**
- **Strategy**: Cache-Aside with CDN
- **TTL**: 5-15 minutes for homepage, 1 hour for individual articles
- **Why**: Articles don't change often once published. Homepage needs fresher content. CDN can serve static content globally. Users can tolerate slightly stale data for speed.

**Application B: E-commerce Product Catalog**
- **Strategy**: Cache-Aside with short TTL or Write-Through
- **TTL**: 1-5 minutes (prices must be relatively current)
- **Why**: Prices change frequently, but some staleness is acceptable. Cache popular products. Use Write-Through for price updates to keep cache synchronized.

**Application C: User Profiles**
- **Strategy**: Cache-Aside with manual invalidation
- **TTL**: 30 minutes to 1 hour
- **Why**: Profile data changes infrequently. When user updates profile, manually invalidate cache. High read frequency (user views their profile often).
</details>

---

### Challenge 2: Cache Size Calculation

You're building a caching layer for a user profile system:

**Given:**
- 1,000,000 registered users
- Average profile size: 5 KB
- 10% of users are active daily
- Cache hit rate goal: 90%

**Questions:**
1. How much cache memory do you need?
2. What eviction policy would you use?
3. How would you handle user profile updates?

<details>
<summary>Sample Answer</summary>

**1. Cache Memory Calculation:**
```
Active daily users: 1,000,000 × 10% = 100,000 users
Cache needed: 100,000 × 5 KB = 500,000 KB = 500 MB

Add 20% buffer: 500 MB × 1.2 = 600 MB
```

To achieve 90% hit rate, cache the most active users' profiles.

**2. Eviction Policy:**
Use **LRU (Least Recently Used)** because:
- Recently active users are likely to be active again
- Automatically keeps most frequently accessed profiles
- Simple and effective for user data

**3. Handling Updates:**
```python
def update_user_profile(user_id, new_data):
    # Write to database
    database.update(user_id, new_data)
    
    # Invalidate cache (or update it)
    cache.delete(f"user_profile:{user_id}")
    
    # OR update cache directly
    cache.set(f"user_profile:{user_id}", new_data, ttl=3600)
```

**Strategy**: Write-Through or manual invalidation
- **Write-Through**: Update both database and cache (always consistent)
- **Manual Invalidation**: Delete from cache, will be re-cached on next read
</details>

---

### Challenge 3: Troubleshooting Cache Issues

**Problem 1:**
Your application uses caching, but users are complaining about seeing outdated product prices.

**System Info:**
- Cache strategy: Cache-Aside
- TTL: 24 hours
- Cache hit rate: 95%

**Questions:**
1. What's causing the problem?
2. How would you fix it?
3. What trade-offs are involved?

<details>
<summary>Answer</summary>

**1. Problem:**
TTL is too long (24 hours). When prices update in the database, cached prices remain stale for up to 24 hours.

**2. Solutions:**

**Option A: Reduce TTL**
```python
cache.set(product_id, product_data, ttl=300)  # 5 minutes
```
- Pro: Fresher data
- Con: More cache misses, higher database load

**Option B: Manual Invalidation**
```python
def update_product_price(product_id, new_price):
    database.update(product_id, new_price)
    cache.delete(f"product:{product_id}")  # Invalidate
```
- Pro: Immediate consistency
- Con: Need to invalidate cache in all places prices change

**Option C: Write-Through**
```python
def update_product_price(product_id, new_price):
    database.update(product_id, new_price)
    cache.set(f"product:{product_id}", new_price)  # Update cache too
```
- Pro: Cache always up-to-date
- Con: Slightly slower writes

**3. Trade-offs:**
- **Consistency vs Performance**: Shorter TTL or manual invalidation = fresher data but more database hits
- **Complexity vs Speed**: Manual invalidation adds code complexity but keeps performance high
- Recommendation: Use manual invalidation + 5-minute TTL as safety net
</details>

---

**Problem 2:**
Your cache memory usage keeps growing until the server runs out of RAM.

**Questions:**
1. What's likely happening?
2. What should you check?
3. How do you fix it?

<details>
<summary>Answer</summary>

**1. Likely Causes:**
- No TTL set (data never expires)
- No eviction policy (cache grows forever)
- Memory leak in cache implementation
- Caching too much data or data that's too large

**2. What to Check:**
```python
# Check current cache size
cache_info = cache.info()
print(f"Items: {cache_info['keys']}")
print(f"Memory: {cache_info['memory_usage']}")

# Check for keys without TTL
keys_without_ttl = cache.keys_without_ttl()

# Check largest keys
large_keys = cache.get_largest_keys(limit=10)
```

**3. Fixes:**

**Immediate:**
```python
# Set max memory limit
cache.config_set('maxmemory', '2gb')
cache.config_set('maxmemory-policy', 'allkeys-lru')
```

**Long-term:**
- Always set TTL: `cache.set(key, value, ttl=3600)`
- Configure eviction policy (LRU)
- Monitor cache size
- Don't cache large objects (compress or store references)
- Set maximum cache size limit
</details>

---

### Challenge 4: Multi-Layer Caching

Design a multi-layer caching system for a video streaming platform:

**Requirements:**
- Serve videos to global users
- 10 million users
- 100,000 videos
- Average video size: 500 MB
- Users watch ~3 videos per session

**Your Design:**
1. What layers of caching will you use?
2. What gets cached at each layer?
3. How do you handle cache invalidation?
4. Calculate approximate cache sizes needed

<details>
<summary>Sample Solution</summary>

**Multi-Layer Architecture:**

```
User
  ↓
[Browser Cache] - Recently watched video segments
  ↓
[CDN Cache] - Popular videos by region
  ↓
[Application Cache] - Video metadata, user preferences
  ↓
[Database] - All videos and data
```

**Layer 1: Browser Cache**
- **Cache**: Video chunks/segments (5-10 second segments)
- **Size**: ~100-500 MB per user
- **TTL**: 1 hour
- **Purpose**: Re-watching parts, buffering

**Layer 2: CDN Cache (Edge Servers)**
- **Cache**: Full videos, especially popular ones
- **Strategy**: Cache top 10% popular videos per region
- **Size per region**: 100,000 videos × 10% × 500 MB = 5 TB per region
- **TTL**: 7 days for popular, 1 day for others
- **Purpose**: Low latency delivery, reduce origin load

**Layer 3: Application Cache (Redis)**
- **Cache**: Video metadata (title, duration, thumbnail URLs), user watch history, recommendations
- **Size**: 
  - Video metadata: 100,000 videos × 5 KB = 500 MB
  - User sessions: 1M active users × 10 KB = 10 GB
  - Total: ~15 GB
- **TTL**: 
  - Metadata: 1 hour
  - User data: 30 minutes
- **Purpose**: Fast API responses, reduce database queries

**Layer 4: Database Cache (Query Cache)**
- **Built-in**: MySQL query cache or Redis for complex queries
- **Size**: 5-10 GB
- **Purpose**: Cache aggregated data, analytics

**Invalidation Strategy:**

1. **Video Upload/Update:**
```python
def update_video(video_id, new_data):
    # Update database
    database.update(video_id, new_data)
    
    # Invalidate application cache
    cache.delete(f"video_metadata:{video_id}")
    
    # Purge from CDN
    cdn.purge(f"/videos/{video_id}/*")
```

2. **Popular Videos:**
- Track view counts
- Background job updates CDN cache with trending videos
- Pre-fetch predicted popular videos to edge servers

3. **Regional Strategy:**
- CDN automatically handles geographic distribution
- Cache popular videos per region separately
- US users see US-cached videos, EU users see EU-cached

**Cost Optimization:**
- Don't cache all videos everywhere (impossible - 50 TB!)
- Cache top 10-20% popular videos = 80%+ of views
- Use LRU at CDN level
- Compress videos (adaptive bitrate streaming)
- Monitor which videos are actually watched
</details>

---

## 🔬 Practical Exercise

### Exercise 1: Implement Simple Cache

Implement a simple LRU cache in your favorite programming language:

**Requirements:**
- Fixed size (e.g., 3 items)
- get(key) operation
- set(key, value) operation
- When full, remove least recently used item

**Starter Code (Python):**
```python
class LRUCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache = {}
        # TODO: Track access order
    
    def get(self, key):
        # TODO: Return value and mark as recently used
        pass
    
    def set(self, key, value):
        # TODO: Add to cache, evict if needed
        pass

# Test
cache = LRUCache(3)
cache.set('A', 1)
cache.set('B', 2)
cache.set('C', 3)
cache.get('A')  # A is now most recent
cache.set('D', 4)  # Should evict B (least recently used)
```

**Test Cases:**
1. Add items until full
2. Access an item and add new one (should evict correct item)
3. Update existing item (shouldn't evict)

---

### Exercise 2: Cache Performance Analysis

**Scenario**: Your API endpoint `/products` takes 200ms without cache.

**Questions:**
1. With 80% hit rate and 1ms cache access, what's average response time?
2. How many database queries saved per 1000 requests?
3. If database can handle 100 queries/sec, what's max throughput without cache? With cache?

<details>
<summary>Answers</summary>

**1. Average Response Time:**
```
80% cache hits: 1ms
20% cache misses: 200ms

Average = (0.8 × 1ms) + (0.2 × 200ms)
        = 0.8ms + 40ms
        = 40.8ms

Improvement: 200ms → 40.8ms (4.9x faster!)
```

**2. Database Queries Saved:**
```
Without cache: 1000 queries
With cache (80% hit rate): 200 queries (only misses)
Saved: 800 queries (80%)
```

**3. Throughput:**
```
Without cache:
Database limit: 100 queries/sec
Throughput: 100 requests/sec

With cache (80% hit):
Database needs: 100 queries/sec serves 500 requests/sec
(Because only 20% hit database)
Throughput: 500 requests/sec

Improvement: 5x throughput!
```
</details>

---

## 🤔 Critical Thinking Questions

1. **Why is cache invalidation considered one of the hardest problems in computer science?**
   (Hint: Think about distributed systems, timing, consistency)

2. **When might caching actually make your system SLOWER?**
   (Consider cache miss penalty, serialization overhead, network latency)

3. **How would you cache user-specific data (like a personalized feed) efficiently?**
   (Hint: Can't cache individually for millions of users)

4. **What happens if your cache servers fail? How do you design for this?**
   (Think about graceful degradation)

---

## 📊 Cache Metrics Exercise

Your system has these statistics over 1 hour:
- Total requests: 100,000
- Cache hits: 75,000
- Cache misses: 25,000
- Cache memory used: 1.5 GB / 2 GB
- Evictions: 5,000

**Calculate:**
1. Hit rate
2. Miss rate
3. Eviction rate
4. Memory utilization
5. Is the cache sized correctly?

<details>
<summary>Answers</summary>

1. **Hit Rate**: 75,000 / 100,000 = 75% (okay, but could be better)
2. **Miss Rate**: 25,000 / 100,000 = 25%
3. **Eviction Rate**: 5,000 / 100,000 = 5% (acceptable)
4. **Memory Utilization**: 1.5 GB / 2 GB = 75% (good - not too full, not wasting)
5. **Assessment**: Cache is reasonably sized. Hit rate of 75% is decent but ideally would be 80%+. Consider:
   - Increasing to 3 GB if hit rate doesn't improve
   - Analyzing which items are being evicted
   - Adjusting TTL values
   - Better eviction policy
</details>

---

## Next Steps

Excellent work on mastering caching! Next, dive into **[Database Design](../04-database-design/)** to learn how to structure your data layer effectively!
