# Caching

## What is Caching?

**Caching** is the process of storing frequently accessed data in a faster storage location to reduce retrieval time. Think of it like keeping your favorite book on your desk instead of going to the library every time you want to read it.

In computing, caching means storing data in memory (RAM) instead of fetching it from slower sources like databases or external APIs.

## Why is Caching Important?

### Speed Comparison:
- **RAM (Cache)**: 0.0001 seconds ⚡
- **SSD**: 0.001 seconds
- **Hard Disk**: 0.01 seconds
- **Database Query**: 0.1+ seconds
- **External API**: 1+ seconds 🐌

**Example:**
Without cache: User requests product list → Database query (100ms) → 100ms response time
With cache: User requests product list → Get from RAM (1ms) → 1ms response time

**Result**: 100x faster! 🚀

## When to Use Caching

✅ **Good for Caching:**
- Data that is read frequently but changes rarely
- Expensive computations or database queries
- External API responses
- Session data
- Static content (images, CSS, JavaScript)

❌ **Not Good for Caching:**
- Data that changes constantly (real-time stock prices)
- User-specific data that's different for everyone
- Data that must always be up-to-date (bank account balance)
- Large data that won't fit in memory

## Types of Caching

### 1. Client-Side Caching (Browser Cache)
**Where**: User's browser
**What**: Images, CSS, JavaScript files, HTML pages

**Example:**
First visit to website:
- Download logo.png (50KB) - Takes 1 second

Next visit:
- Load logo.png from cache - Takes 0.01 seconds

**Control with HTTP Headers:**
```
Cache-Control: max-age=3600  (cache for 1 hour)
Cache-Control: no-cache      (always check server)
```

---

### 2. CDN Caching (Content Delivery Network)
**Where**: Edge servers around the world
**What**: Static content (images, videos, files)

**How it works:**
```
User in Japan → Japan CDN Server (fast!)
User in USA → USA CDN Server (fast!)
```
Instead of everyone fetching from one origin server.

**Popular CDNs**: Cloudflare, AWS CloudFront, Akamai

---

### 3. Application Cache
**Where**: In your application server's memory
**What**: Frequently used data, computation results

**Example:**
```python
cache = {}

def get_user(user_id):
    if user_id in cache:
        return cache[user_id]  # Fast!
    
    user = database.query(user_id)  # Slow
    cache[user_id] = user
    return user
```

---

### 4. Database Cache
**Where**: Separate cache server (Redis, Memcached)
**What**: Database query results, session data

**Architecture:**
```
Application → Check Redis → If miss → Database
                   ↑               ↓
                   └──── Store ────┘
```

---

## Cache Strategies (When to Update Cache)

### 1. Cache-Aside (Lazy Loading)
**How it works:**
1. Check cache
2. If data exists (hit), return it
3. If data doesn't exist (miss), fetch from database
4. Store in cache for next time

```python
def get_product(product_id):
    # Check cache first
    product = cache.get(product_id)
    
    if product:
        return product  # Cache hit!
    
    # Cache miss - fetch from database
    product = database.query(product_id)
    
    # Store in cache
    cache.set(product_id, product, expire=3600)  # 1 hour
    
    return product
```

**Pros**: Only caches what's actually needed  
**Cons**: First request is always slow (cache miss)

**Best for**: Read-heavy applications

---

### 2. Write-Through
**How it works:**
1. Write to database AND cache simultaneously
2. Cache is always up-to-date

```python
def update_product(product_id, data):
    # Update database
    database.update(product_id, data)
    
    # Update cache immediately
    cache.set(product_id, data)
```

**Pros**: Cache is always synchronized  
**Cons**: Slower writes (double write), may cache unused data

**Best for**: Read-heavy with critical consistency

---

### 3. Write-Back (Write-Behind)
**How it works:**
1. Write to cache immediately
2. Write to database later (asynchronously)

**Pros**: Fastest writes  
**Cons**: Risk of data loss if cache fails before database write

**Best for**: Write-heavy with less critical data

---

### 4. Refresh-Ahead
**How it works:**
1. Predict which data will be needed
2. Refresh cache before it expires

**Pros**: No cache misses for predicted data  
**Cons**: Complex, may refresh unnecessary data

**Best for**: Predictable access patterns

---

## Cache Eviction Policies

What happens when cache is full? Something must be removed!

### 1. LRU (Least Recently Used)
Remove the item that hasn't been used for the longest time.

**Example:**
Cache size: 3 items
```
Access: A, B, C, D
State: [A, B, C]
Access: D → Remove A (oldest) → [B, C, D]
Access: B → [C, D, B] (B moves to front)
```

**Use case**: Most common, works well for general purposes

---

### 2. LFU (Least Frequently Used)
Remove the item that has been accessed the fewest times.

**Example:**
```
Item A: accessed 10 times
Item B: accessed 5 times
Item C: accessed 2 times ← Remove this
```

**Use case**: When access frequency matters more than recency

---

### 3. FIFO (First In, First Out)
Remove the oldest item regardless of usage.

**Use case**: Simple but not optimal for most cases

---

### 4. TTL (Time To Live)
Items expire after a set time.

**Example:**
```python
cache.set('user_123', user_data, ttl=3600)  # Expires in 1 hour
```

**Use case**: Combine with other policies for data freshness

---

## Common Caching Issues

### 1. Cache Stampede (Thundering Herd)
**Problem**: Cache expires, many requests hit database simultaneously

**Scenario:**
```
Popular item expires at 12:00 PM
1000 users request at 12:00 PM
All 1000 queries hit database! 💥
```

**Solution**: Lock or temporary cache extension
```python
def get_with_lock(key):
    value = cache.get(key)
    if value:
        return value
    
    # Only one process fetches from DB
    with lock(key):
        value = cache.get(key)  # Check again
        if not value:
            value = database.query(key)
            cache.set(key, value)
    
    return value
```

---

### 2. Cache Invalidation
**Problem**: "There are only two hard things in Computer Science: cache invalidation and naming things." - Phil Karlton

**Challenge**: Keeping cache synchronized with database

**Solutions:**
1. **TTL**: Set expiration time (simple but may serve stale data)
2. **Manual Invalidation**: Clear cache when data updates
3. **Event-Based**: Listen to database changes and update cache

```python
def update_user(user_id, data):
    database.update(user_id, data)
    cache.delete(user_id)  # Invalidate cache
```

---

### 3. Cold Start
**Problem**: Empty cache when application starts

**Solution**: Cache warming
```python
def warm_cache():
    # Pre-load frequently accessed data
    popular_products = database.query_popular()
    for product in popular_products:
        cache.set(product.id, product)
```

---

### 4. Memory Management
**Problem**: Running out of cache memory

**Solutions:**
1. Increase cache size
2. Better eviction policy
3. Reduce data size (compress, store only needed fields)
4. Partition cache across multiple servers

---

## Cache Metrics

Monitor these metrics to ensure cache is effective:

### 1. Hit Rate
```
Hit Rate = Cache Hits / Total Requests
```
**Good**: 80%+  
**Excellent**: 95%+

### 2. Miss Rate
```
Miss Rate = Cache Misses / Total Requests = 1 - Hit Rate
```

### 3. Eviction Rate
How often items are removed from cache

### 4. Memory Usage
Don't let cache use all available memory!

---

## Popular Caching Technologies

### 1. Redis
- In-memory data store
- Supports complex data structures (lists, sets, hashes)
- Persistence options
- Pub/sub messaging
- **Use for**: Session storage, real-time analytics, queues

### 2. Memcached
- Simple key-value store
- Very fast
- No persistence
- **Use for**: Simple caching needs

### 3. Varnish
- HTTP cache
- Sits in front of web servers
- **Use for**: Caching web pages

### 4. Browser Cache
- Built into web browsers
- Controlled by HTTP headers
- **Use for**: Static assets

---

## Real-World Examples

### Facebook
- Caches user profiles, friend lists, news feed
- Uses Memcached extensively
- Reduces database queries by 99%

### Twitter
- Caches timelines
- Pre-computes popular tweets
- Uses Redis for real-time features

### Netflix
- CDN caching for videos
- Caches user preferences
- Pre-fetches likely-to-watch content

---

## Best Practices

1. **Cache Frequently Accessed Data**: Monitor what's accessed most
2. **Set Appropriate TTL**: Balance freshness vs performance
3. **Monitor Hit Rate**: Aim for 80%+ hit rate
4. **Handle Cache Failures**: Application should work without cache
5. **Don't Cache Everything**: Only cache what provides value
6. **Use Multiple Cache Layers**: Browser → CDN → Application → Database
7. **Compress Large Objects**: Save memory
8. **Version Your Cache Keys**: Easier invalidation (user:123:v2)

---

## Cache Architecture Pattern

**Typical Multi-Layer Caching:**
```
User Request
    ↓
Browser Cache (images, CSS, JS)
    ↓ (if miss)
CDN Cache (static content)
    ↓ (if miss)
Application Cache (dynamic data)
    ↓ (if miss)
Database
```

Each layer is progressively slower but has more complete data.

---

## Summary

- Caching stores frequently accessed data in fast storage (RAM)
- Can improve performance by 10-100x
- Multiple types: client-side, CDN, application, database
- Common strategies: Cache-Aside, Write-Through, Write-Back
- Eviction policies: LRU, LFU, FIFO, TTL
- Watch out for cache stampede and invalidation issues
- Monitor hit rate (aim for 80%+)
- Use appropriate caching technology for your needs

---

## Next Steps

Complete the challenges to master caching concepts, then explore **[Database Design](../04-database-design/)** to learn how to structure data efficiently!

## Implementation Approaches

### Approach 1: Application-Level Caching
- **Description**: Cache implemented within application code using libraries (in-memory cache per instance).
- **Pros**: Very fast (no network), simple to implement, no external dependencies, low latency (<1ms).
- **Cons**: Not shared across instances, limited by RAM, cache lost on restart, difficult to invalidate across instances.
- **When to use**: Single server apps, read-heavy data that rarely changes, development/testing, session-specific data.

### Approach 2: Distributed Cache (Redis, Memcached)
- **Description**: Standalone cache server shared across all application instances.
- **Pros**: Shared cache across servers, persistence options (Redis), data structures support, scalable, centralized invalidation.
- **Cons**: Network latency (1-5ms), additional infrastructure, cost, single point of failure (without clustering).
- **When to use**: Multi-server deployments, need shared cache, session storage, rate limiting, most production systems.

### Approach 3: CDN Caching
- **Description**: Cache static assets at edge locations near users globally.
- **Pros**: Reduces origin load dramatically, global performance, built-in DDoS protection, bandwidth savings.
- **Cons**: Not for dynamic content, cache invalidation delays, costs money, less control.
- **When to use**: Static assets (images, CSS, JS), public content, global user base, high bandwidth costs.

### Approach 4: Database Query Caching
- **Description**: Cache database query results either at database level or application level.
- **Pros**: Reduces database load significantly, faster response times, can cache complex queries.
- **Cons**: Invalidation complexity, stale data risks, memory overhead, query key generation complexity.
- **When to use**: Expensive queries, read-heavy applications, queries with consistent results, aggregation queries.

## Trade-offs

| Aspect | Write-Through | Write-Behind | Cache-Aside |
|--------|---------------|--------------|-------------|
| Write Performance | Slower (wait for cache + DB) | Fast (async write) | Fast (skip cache) |
| Read Performance | Fast (always in cache) | Fast | Moderate (miss penalty) |
| Consistency | Strong | Eventual | Eventual |
| Complexity | Moderate | High | Low |
| Data Loss Risk | None | Possible | None |
| Use Case | Read-heavy, consistency critical | Write-heavy | General purpose |

| Aspect | Small TTL (5 min) | Large TTL (24 hours) |
|--------|------------------|---------------------|
| Data Freshness | Very fresh | Potentially stale |
| Database Load | Higher | Lower |
| Cache Hit Rate | Lower | Higher |
| Use Case | Frequently changing | Rarely changing |

## Capacity Calculations

### Cache Memory Sizing
```
Active users: 1,000,000
Average session data: 10 KB per user
Session cache: 1M × 10 KB = 10 GB

Product catalog: 100,000 products
Average product data: 5 KB
Product cache: 100K × 5 KB = 500 MB

API responses cached: 10,000 unique endpoints
Average response: 20 KB
Response cache: 10K × 20 KB = 200 MB

Total cache need: ~11 GB
With overhead (30%): 14-15 GB
Recommended: 32 GB Redis instance
```

### Cache Hit Ratio Impact
```
Database capacity: 10,000 QPS
Total requests: 100,000 QPS

Cache hit ratio: 90%
Database queries: 100K × 0.1 = 10K QPS ✓
Cache queries: 90K QPS (handled by cache)

Cache hit ratio: 80%
Database queries: 100K × 0.2 = 20K QPS ✗ (exceeds capacity)

Need: 85%+ hit ratio to stay within DB capacity
```

### Eviction Rate Monitoring
```
Cache size: 16 GB
Eviction rate: 100 keys/second
Average key size: 10 KB

Memory pressure: 100 × 10 KB = 1 MB/s evicted
Daily evictions: 86 GB (cache churning heavily)

Action needed: Increase cache size or reduce TTL
Target: <1% of cache size evicted per hour
```

## Common Patterns

**Cache-Aside (Lazy Loading)**: Application checks cache first, on miss fetches from DB and populates cache. Most common pattern. Simple and effective.

**Write-Through**: Write to cache and database simultaneously. Ensures cache always has latest data. Slower writes but consistent reads.

**Write-Behind (Write-Back)**: Write to cache immediately, asynchronously write to database. Fast writes but risk of data loss if cache fails.

**Refresh-Ahead**: Automatically refresh cached items before they expire based on access patterns. Prevents cache miss storms on popular items.

**Cache Warming**: Pre-populate cache with likely-needed data on startup. Prevents cold start performance issues. Common for product catalogs, popular content.

**Read-Through**: Cache intercepts all reads. On miss, cache fetches from database automatically. Simplifies application code.

**Multi-Layer Caching**: Browser cache → CDN → Application cache → Database query cache. Each layer reduces load on next.

## Anti-Patterns (What NOT to Do)

**Caching Everything**: Caching rarely-accessed or fast-changing data wastes memory. Cache only frequently accessed, expensive-to-compute data.

**No Cache Expiration**: Setting infinite TTL. Stale data accumulates. Old users see outdated information. Always set appropriate TTL.

**Cache Stampede**: Cache expires, many requests simultaneously hit database. Use locking or probabilistic early expiration to prevent.

**Ignoring Memory Limits**: Filling cache beyond capacity causes thrashing. Monitor memory usage and set size limits. Configure eviction policies.

**Not Monitoring Hit Rates**: Assuming cache is working. Low hit rate means ineffective cache. Monitor and optimize based on access patterns.

**Caching User-Specific Data Globally**: Caching data that contains user-specific information accessible to other users. Security vulnerability.

**Complex Cache Keys**: Using complex objects as keys. Difficult to invalidate. Use simple, predictable keys (user_id:123, product:456).

**No Invalidation Strategy**: Caching without thinking about updates. Users see stale data indefinitely. Plan invalidation from the start.

## Interview Tips

**Explain Caching Levels**: "Browser → CDN → Application → Database. Each layer reduces load on next layer."

**Discuss Hit Ratios**: "Target 80-90% hit ratio. 90% means 10x less database load. Monitor with cache.hits / (cache.hits + cache.misses)."

**Calculate Capacity**: "1M users × 10KB session = 10GB cache. Add 30% overhead = 13GB. Recommend 16GB instance."

**Eviction Policies**: "LRU for general purpose. LFU for stable access patterns. TTL for time-sensitive data. Choose based on access pattern."

**Invalidation Strategies**: "TTL for automatic expiration. Active invalidation on updates. Cache tags for grouped invalidation. Version keys for guaranteed freshness."

**Failure Handling**: "Cache failures should not break application. Treat cache as performance enhancement, not requirement. Fallback to database on cache miss."

**Real Examples**: "Facebook caches friend lists in Memcached. Netflix caches movie metadata in EVCache. Reddit uses Redis for voting and comments."

## See It In Action

Complete caching design challenges: **[Caching Challenges](./challenges.md)**

## Additional Resources

**Documentation**:
- [Redis Documentation](https://redis.io/documentation)
- [Memcached Wiki](https://github.com/memcached/memcached/wiki)
- [AWS ElastiCache Best Practices](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/BestPractices.html)

**Articles**:
- [Caching Best Practices - AWS](https://aws.amazon.com/caching/best-practices/)
- [Facebook's Memcached Architecture](https://www.facebook.com/notes/facebook-engineering/scaling-memcached-at-facebook/39391378919/)
- [Cache is King - High Scalability](http://highscalability.com/blog/2009/10/26/facebooks-memcached-multiget-hole-more-machines-more-capacit.html)

**Tools**:
- **Redis**: In-memory data structure store
- **Memcached**: High-performance distributed memory caching
- **Varnish**: HTTP accelerator/caching proxy
- **redis-cli**: Command line interface for Redis debugging
