# Case Study: URL Shortener (Beginner)

## Problem Statement

Design a URL shortening service like bit.ly or TinyURL that converts long URLs into short, manageable links and redirects users from short URLs back to the original long URLs.

## Requirements Analysis

### Functional Requirements

**Core Features:**
1. **URL Shortening**: Given a long URL, generate a unique short URL
2. **URL Redirection**: When accessing a short URL, redirect to the original long URL
3. **Custom Aliases**: Allow users to specify custom short URL aliases
4. **Expiration**: Support optional expiration dates for shortened URLs
5. **Analytics**: Track basic metrics (click count, last accessed time)

**Out of Scope (for this version):**
- User authentication and account management
- URL editing after creation
- Advanced analytics (geographic data, referrers, browsers)
- URL preview or safety checking

### Non-Functional Requirements

**Scale Requirements:**
- **Write Operations**: 100 million new URLs per month (~40 URLs/second average, ~100 URLs/second peak)
- **Read Operations**: 10 billion redirects per month (100:1 read-to-write ratio)
  - ~4,000 redirects/second average
  - ~10,000 redirects/second peak
- **URL Lifespan**: Support 10 years of data retention
- **Storage**: Accommodate 12 billion URLs over 10 years

**Performance Requirements:**
- **Redirect Latency**: < 50ms for 99th percentile
- **Shortening Latency**: < 200ms for 99th percentile
- **Availability**: 99.9% uptime (8.76 hours downtime per year)

**Data Integrity:**
- No duplicate short URLs (uniqueness guaranteed)
- Short URLs are immutable once created
- Original URLs can be up to 2048 characters

## Capacity Estimation

### Storage Calculation

```
URLs created per month: 100M
URLs per year: 1.2B
10-year total: 12B URLs

Per URL storage:
- Short code: 7 bytes (base62 encoding)
- Original URL: ~200 bytes average (up to 2048 max)
- Metadata (created_at, user_id, expires_at): ~50 bytes
- Analytics (click_count, last_accessed): ~20 bytes
- Total per record: ~280 bytes

Total storage: 12B × 280 bytes = 3.36 TB

With replication (3x): ~10 TB
With overhead (20%): ~12 TB over 10 years
```

### Traffic Estimation

```
**Write Traffic:**
- 40 writes/sec average
- 100 writes/sec peak
- Data rate: 100 writes/sec × 280 bytes = 28 KB/sec

**Read Traffic:**
- 4,000 reads/sec average  
- 10,000 reads/sec peak
- Data rate: 10,000 reads/sec × 280 bytes = 2.8 MB/sec
```

### Bandwidth Requirements

```
**Ingress (shortening):**
- 100 URLs/sec × 280 bytes = 28 KB/sec (~0.22 Mbps)

**Egress (redirects):**
- 10,000 redirects/sec × 280 bytes = 2.8 MB/sec (~22 Mbps)
- Note: Most data is in HTTP response headers (302 redirect)
```

### Cache Sizing

```
Using 80-20 rule: 20% of URLs generate 80% of traffic

Hot URLs to cache: 12B × 0.20 = 2.4B URLs
Cache size: 2.4B × 280 bytes = 672 GB

Practical cache (10% of URLs): 1.2B × 280 bytes = 336 GB
Distributed across multiple Redis instances
```

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                          CLIENT LAYER                            │
│  [Web Browsers] [Mobile Apps] [API Clients] [CLI Tools]        │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                      LOAD BALANCER (HAProxy)                     │
│              Routes traffic, Health checks, SSL termination      │
└────────────────────────────┬────────────────────────────────────┘
                             │
                    ┌────────┴────────┐
                    ▼                 ▼
┌──────────────────────┐    ┌──────────────────────┐
│   API SERVERS        │    │   API SERVERS        │
│   - URL Shortening   │    │   - URL Shortening   │
│   - URL Redirection  │    │   - URL Redirection  │
│   - Analytics        │    │   - Analytics        │
└──────────┬───────────┘    └──────────┬───────────┘
           │                           │
           └───────────┬───────────────┘
                       │
           ┌───────────┴───────────┐
           ▼                       ▼
┌──────────────────────┐   ┌──────────────────────┐
│   REDIS CACHE        │   │   ANALYTICS QUEUE    │
│   - Short URL → Long │   │      (Kafka)         │
│   - TTL: 24h         │   │   - Click events     │
│   - 95% hit rate     │   │   - Async processing │
└──────────┬───────────┘   └──────────┬───────────┘
           │                          │
           │ Cache Miss               │
           ▼                          ▼
┌──────────────────────┐   ┌──────────────────────┐
│   DATABASE CLUSTER   │   │  ANALYTICS SERVICE   │
│   (PostgreSQL)       │   │  - Update counters   │
│   - Master: Writes   │   │  - Aggregate stats   │
│   - Replicas: Reads  │   │  - Store in DB       │
│   - Sharded by hash  │   └──────────────────────┘
└──────────────────────┘
```

## API Design

### 1. Create Short URL

**Endpoint:**
```http
POST /api/v1/shorten
Content-Type: application/json
Authorization: Bearer <optional_api_key>

Request:
{
  "long_url": "https://www.example.com/very/long/url/with/many/parameters?foo=bar",
  "custom_alias": "my-link",      // Optional: user-specified short code
  "expires_at": "2025-12-31",     // Optional: expiration date
  "description": "My landing page" // Optional: for user reference
}

Response 201 Created:
{
  "short_url": "https://short.ly/abc123",
  "short_code": "abc123",
  "long_url": "https://www.example.com/very/long/url/with/many/parameters?foo=bar",
  "created_at": "2024-01-15T10:30:00Z",
  "expires_at": "2025-12-31T00:00:00Z"
}

Error 400 Bad Request:
{
  "error": "invalid_url",
  "message": "The provided URL is not valid"
}

Error 409 Conflict:
{
  "error": "alias_taken",
  "message": "Custom alias 'my-link' is already in use"
}
```

### 2. Redirect to Original URL

**Endpoint:**
```http
GET /{short_code}

Example: GET /abc123

Response 302 Found:
Location: https://www.example.com/very/long/url/with/many/parameters?foo=bar

Response 404 Not Found:
{
  "error": "not_found",
  "message": "Short URL not found or has expired"
}
```

### 3. Get URL Analytics

**Endpoint:**
```http
GET /api/v1/analytics/{short_code}
Authorization: Bearer <api_key>

Response 200 OK:
{
  "short_code": "abc123",
  "long_url": "https://www.example.com/...",
  "created_at": "2024-01-15T10:30:00Z",
  "expires_at": "2025-12-31T00:00:00Z",
  "clicks": 15234,
  "last_accessed": "2024-01-20T15:45:00Z"
}
```

## Database Schema

### Primary Database (PostgreSQL)

```sql
-- Main URL mappings table
CREATE TABLE urls (
    id BIGSERIAL PRIMARY KEY,
    short_code VARCHAR(10) UNIQUE NOT NULL,
    long_url TEXT NOT NULL,
    user_id BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP,
    is_custom BOOLEAN DEFAULT FALSE,
    
    INDEX idx_short_code (short_code),
    INDEX idx_user_id (user_id),
    INDEX idx_created_at (created_at)
);

-- Analytics data (updated asynchronously)
CREATE TABLE url_analytics (
    id BIGSERIAL PRIMARY KEY,
    short_code VARCHAR(10) NOT NULL,
    click_count BIGINT DEFAULT 0,
    last_accessed TIMESTAMP,
    
    FOREIGN KEY (short_code) REFERENCES urls(short_code),
    INDEX idx_short_code (short_code)
);

-- Optional: User table for authenticated API access
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    api_key VARCHAR(64) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_api_key (api_key)
);
```

### Cache Schema (Redis)

```
Key-Value Store:
Key: "url:{short_code}"
Value: {long_url} (string)
TTL: 86400 seconds (24 hours)

Example:
"url:abc123" → "https://www.example.com/very/long/url..."

Key: "analytics:{short_code}:count"
Value: {click_count} (integer)
TTL: 3600 seconds (1 hour, then flush to DB)
```

## Detailed Component Design

### 1. Short Code Generation Service

**Strategy: Base62 Encoding with Auto-Increment Counter**

```python
class ShortCodeGenerator:
    BASE62 = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
    
    def encode_base62(self, num: int) -> str:
        """Convert integer to base62 string."""
        if num == 0:
            return self.BASE62[0]
        
        result = []
        while num > 0:
            result.append(self.BASE62[num % 62])
            num //= 62
        
        return ''.join(reversed(result)).rjust(7, '0')
    
    def generate_short_code(self, url_id: int) -> str:
        """Generate 7-character short code from database ID."""
        return self.encode_base62(url_id)

# Example:
# ID 1 → "0000001"
# ID 62 → "0000010"  
# ID 916132832 → "abc123"

# 62^7 = 3,521,614,606,208 possible codes (3.5 trillion)
# More than enough for 12 billion URLs
```

**Alternative: Random Generation with Collision Detection**

```python
import random
import string

def generate_random_code(length=7):
    """Generate random alphanumeric code."""
    chars = string.ascii_letters + string.digits
    
    max_attempts = 10
    for attempt in range(max_attempts):
        code = ''.join(random.choice(chars) for _ in range(length))
        
        # Check if code exists in database
        if not db.exists(code):
            return code
    
    raise Exception("Failed to generate unique code")
```

### 2. URL Redirection Service

**High-Performance Redirect Flow:**

```python
async def redirect_url(short_code: str) -> str:
    # 1. Check cache first (fast path)
    long_url = await redis.get(f"url:{short_code}")
    
    if long_url:
        # Cache hit - return immediately
        asyncio.create_task(track_click_async(short_code))
        return long_url
    
    # 2. Cache miss - query database
    url_record = await db.query(
        "SELECT long_url, expires_at FROM urls WHERE short_code = ?",
        short_code
    )
    
    if not url_record:
        raise NotFoundError("Short URL not found")
    
    # 3. Check expiration
    if url_record.expires_at and url_record.expires_at < now():
        raise NotFoundError("Short URL has expired")
    
    # 4. Cache the result for future requests
    await redis.setex(
        f"url:{short_code}",
        86400,  # 24 hour TTL
        url_record.long_url
    )
    
    # 5. Track click asynchronously (don't block redirect)
    asyncio.create_task(track_click_async(short_code))
    
    return url_record.long_url

async def track_click_async(short_code: str):
    """Increment click counter without blocking redirect."""
    await kafka.produce("clicks", {
        "short_code": short_code,
        "timestamp": now()
    })
```

### 3. Analytics Service

**Asynchronous Click Tracking:**

```python
class AnalyticsConsumer:
    def __init__(self):
        self.buffer = {}
        self.flush_interval = 60  # Flush every 60 seconds
    
    async def consume_clicks(self):
        """Consume click events from Kafka."""
        async for message in kafka.consume("clicks"):
            short_code = message.short_code
            
            # Buffer clicks in memory
            self.buffer[short_code] = self.buffer.get(short_code, 0) + 1
    
    async def flush_to_database(self):
        """Batch update click counts in database."""
        while True:
            await asyncio.sleep(self.flush_interval)
            
            if not self.buffer:
                continue
            
            # Batch update in single query
            updates = [
                (count, short_code) 
                for short_code, count in self.buffer.items()
            ]
            
            await db.execute_batch(
                """
                UPDATE url_analytics 
                SET click_count = click_count + ?, 
                    last_accessed = NOW()
                WHERE short_code = ?
                """,
                updates
            )
            
            self.buffer.clear()
```

## Trade-off Discussions

### 1. Short Code Generation: Sequential vs Random

**Sequential (Base62 encoding of auto-increment ID):**
- ✅ Simple implementation
- ✅ No collision handling needed
- ✅ Predictable storage growth
- ❌ URLs are guessable (security concern)
- ❌ Exposes number of URLs created

**Random generation:**
- ✅ Unpredictable codes (better security)
- ✅ No sequence information leaked
- ❌ Requires collision detection
- ❌ Slightly slower generation

**Recommendation**: Use sequential for simplicity, add random prefix for security if needed.

### 2. Redirect Type: 301 vs 302

**301 Permanent Redirect:**
- ✅ Cached by browsers (faster for users)
- ✅ Better for SEO
- ❌ Cannot track all clicks accurately
- ❌ Difficult to change destination

**302 Temporary Redirect:**
- ✅ All clicks go through server (accurate analytics)
- ✅ Can change destination
- ❌ Slower (not cached)
- ❌ Worse for SEO

**Recommendation**: Use 302 for analytics, or 301 with server-side logging before redirect.

### 3. Database Choice: SQL vs NoSQL

**SQL (PostgreSQL):**
- ✅ ACID guarantees for consistency
- ✅ Complex queries for analytics
- ✅ Strong data integrity
- ❌ Harder to scale horizontally
- ❌ More expensive for massive scale

**NoSQL (Cassandra/DynamoDB):**
- ✅ Excellent horizontal scalability
- ✅ High write throughput
- ✅ Simple key-value operations
- ❌ No complex queries
- ❌ Eventual consistency

**Recommendation**: PostgreSQL initially (sufficient for billions of URLs), migrate to NoSQL if write load exceeds 10,000/sec.

### 4. Caching Strategy

**Cache Everything:**
- ✅ Maximum performance
- ❌ High memory usage
- ❌ Cache invalidation complexity

**Cache Hot URLs Only (20%):**
- ✅ 80% of traffic cached with 20% of storage
- ✅ Cost-effective
- ❌ Cache misses for long-tail URLs

**Recommendation**: Implement LRU cache with 24-hour TTL, targeting 90%+ hit rate.

## Scaling Strategies

### Phase 1: Single Server (0-1M URLs)

```
[Load Balancer]
      ↓
[API Server + Redis + PostgreSQL on single instance]
```

- Monolithic deployment
- Single database instance
- Redis on same machine
- Handles ~100 requests/second

### Phase 2: Separated Services (1M-100M URLs)

```
[Load Balancer]
      ↓
[API Servers x3] → [Redis Cluster]
                 ↓
           [PostgreSQL Primary]
                 ↓
           [Read Replicas x2]
```

- Multiple API servers
- Dedicated Redis cluster
- Database read replicas
- Handles ~1,000 requests/second

### Phase 3: Horizontal Scaling (100M-1B URLs)

```
[Global Load Balancer]
         ↓
[Regional Load Balancers]
         ↓
[API Servers x10-50 per region]
         ↓
[Redis Clusters (geo-distributed)]
         ↓
[PostgreSQL Sharded x4]
  - Shard by short_code hash
```

- Multiple geographic regions
- Database sharding by hash(short_code)
- Redis clusters per region
- Handles ~10,000 requests/second

### Phase 4: Global Scale (1B+ URLs)

```
[Anycast DNS]
      ↓
[CDN (Cloudflare/Fastly)]
      ↓
[Multi-region deployment]
      ↓
[Cassandra/DynamoDB (globally distributed)]
```

- CDN edge locations for ultra-low latency
- NoSQL for unlimited horizontal scaling
- Global distribution with multi-region writes
- Handles 100,000+ requests/second

## Interview Talking Points

### Key Topics to Discuss

**1. Requirements Clarification:**
- "Do we need user authentication or anonymous URL creation?"
- "What's our expected read-to-write ratio?"
- "Do we need to support custom aliases?"
- "Should shortened URLs expire?"

**2. Capacity Estimation:**
- "100M URLs/month = 40 writes/sec, 4,000 reads/sec"
- "12B URLs over 10 years = ~3.5 TB storage"
- "Cache 10-20% of hot URLs = 350 GB"

**3. System Design Choices:**
- "Base62 encoding ensures uniqueness without collisions"
- "Redis cache provides <10ms redirect latency"
- "Async analytics tracking doesn't block redirects"
- "Database sharding enables horizontal scaling"

**4. Potential Bottlenecks:**
- "Database writes become bottleneck at 1,000+ writes/sec"
  - Solution: Sharding or NoSQL migration
- "Cache memory insufficient for hot URLs"
  - Solution: Larger Redis cluster with LRU eviction
- "Single region deployment has high latency globally"
  - Solution: Multi-region deployment with geo-routing

**5. Follow-up Features:**
- "Link expiration: Add background job to purge expired URLs"
- "Rate limiting: Use Redis counters with sliding window"
- "URL preview: Add metadata scraping service"
- "Custom domains: Add DNS configuration service"

### Common Interview Questions

**Q: How would you handle duplicate URLs?**
A: "Hash the long URL, check if it exists, return existing short code to save storage."

**Q: How do you prevent abuse?**
A: "Implement rate limiting (100 URLs/hour per IP), require API keys for higher limits, validate URLs before shortening."

**Q: What if short codes run out?**
A: "With 7-character base62, we have 3.5 trillion combinations. At 100M/month, that's 35,000 months (2,900 years). We won't run out. If needed, increase to 8 characters for 62x more capacity."

**Q: How do you ensure high availability?**
A: "Deploy multiple API servers behind load balancer, use database replication with automatic failover, implement Redis cluster with persistence, deploy across multiple availability zones."

**Q: How would you add analytics?**
A: "Track clicks asynchronously using message queue (Kafka), batch updates to database every minute, cache recent counts in Redis, provide API endpoint for stats retrieval."

## Summary

This URL shortener design demonstrates several fundamental system design principles:

**Key Learnings:**
1. **Capacity Estimation**: Calculate storage, traffic, and bandwidth requirements
2. **Caching Strategy**: Use Redis to handle 90%+ of reads
3. **Database Design**: Choose appropriate schema and indexes
4. **Async Processing**: Decouple analytics from critical path
5. **Scaling Path**: Start simple, scale horizontally as needed

**System Characteristics:**
- Storage: 3-4 TB for 12B URLs
- Throughput: 10,000 redirects/sec with caching
- Latency: <50ms p99 for redirects
- Availability: 99.9% with proper redundancy

**Real-World Examples:**
- bit.ly: Uses similar architecture with MongoDB
- TinyURL: Started with simple LAMP stack, scaled horizontally
- Google's goo.gl: Used Bigtable for massive scale (now discontinued)

This design provides a solid foundation that can scale from startup to billions of URLs while maintaining excellent performance and reliability.
