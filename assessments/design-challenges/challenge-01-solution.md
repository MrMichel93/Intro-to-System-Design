# Design Challenge 1: URL Shortener - Sample Solution

## 1. Capacity Estimation

### Given
- 100 million URL shortenings per month
- Read/write ratio: 100:1
- Estimate for 5 years

### Calculations

**Write Operations (URL Creation):**
- Per month: 100 million
- Per day: 100M / 30 = 3.3 million
- Per second: 3.3M / 86,400 ≈ 38 writes/second

**Read Operations (Redirects):**
- With 100:1 ratio: 38 × 100 = 3,800 reads/second
- Peak traffic (3x average): ~11,400 reads/second

**Storage Requirements:**
```
Assumptions:
- Each URL record contains:
  - Short URL: 7 bytes
  - Original URL: 500 bytes (average)
  - Created timestamp: 8 bytes
  - Click count: 4 bytes
  - Total per record: ~520 bytes

For 5 years:
- Total URLs: 100M/month × 12 × 5 = 6 billion URLs
- Storage: 6B × 520 bytes = 3.12 TB
- With overhead and indexes: ~5 TB
```

**Bandwidth:**
```
Writes: 38 requests/sec × 0.5 KB = 19 KB/s (negligible)
Reads: 3,800 requests/sec × 0.5 KB = 1.9 MB/s
Peak: 11,400 × 0.5 KB = 5.7 MB/s
```

---

## 2. API Design

### Create Short URL
```
POST /api/v1/shorten
Content-Type: application/json

Request Body:
{
  "original_url": "https://www.example.com/very/long/url/path",
  "custom_alias": "mylink" (optional)
}

Response (201 Created):
{
  "short_url": "https://short.ly/aB3xY9",
  "original_url": "https://www.example.com/very/long/url/path",
  "created_at": "2025-01-15T10:30:00Z"
}

Error Responses:
- 400 Bad Request: Invalid URL format
- 409 Conflict: Custom alias already taken
- 429 Too Many Requests: Rate limit exceeded
```

### Redirect to Original URL
```
GET /{short_code}

Response (301 Moved Permanently or 302 Found):
Location: https://www.example.com/very/long/url/path

Error Response:
- 404 Not Found: Short URL doesn't exist
```

---

## 3. Database Schema

### Using SQL (Relational)

```sql
CREATE TABLE urls (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    short_code VARCHAR(10) UNIQUE NOT NULL,
    original_url VARCHAR(2048) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    click_count BIGINT DEFAULT 0,
    INDEX idx_short_code (short_code),
    INDEX idx_created_at (created_at)
);
```

### Alternative: NoSQL (Key-Value Store)

**DynamoDB/Cassandra:**
```
Table: urls
Primary Key: short_code (String)

Attributes:
- original_url (String)
- created_at (Number - timestamp)
- click_count (Number)
```

**Justification:** 
- Simple key-value lookups (short_code → URL)
- High read performance
- Easy to scale horizontally
- NoSQL preferred for this use case due to simple access patterns

---

## 4. High-Level Architecture

```
┌─────────────┐
│   Clients   │
│  (Browsers) │
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│  Load Balancer  │
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌────────┐ ┌────────┐
│  API   │ │  API   │  (Stateless Application Servers)
│ Server │ │ Server │
└───┬────┘ └───┬────┘
    │          │
    ├──────────┴─────────┐
    │                    │
    ▼                    ▼
┌─────────┐        ┌──────────┐
│  Cache  │        │ Database │
│ (Redis) │◄──────►│(NoSQL DB)│
└─────────┘        └──────────┘
```

### Components:

**1. Load Balancer:**
- Distributes traffic across API servers
- Performs health checks
- Uses Round Robin or Least Connections

**2. API Servers:**
- Stateless application servers
- Handle URL creation and redirect logic
- Can scale horizontally

**3. Cache Layer (Redis):**
- Stores frequently accessed short_code → original_url mappings
- TTL: Not needed (URLs don't change)
- Cache-aside pattern
- 80/20 rule: Cache 20% most popular URLs serving 80% of traffic

**4. Database (NoSQL - Cassandra/DynamoDB):**
- Stores all URL mappings
- Optimized for key-value lookups
- Easily scalable horizontally

---

## 5. Algorithm for Generating Short URLs

### Approach: Base62 Encoding with Auto-Incrementing ID

**Character Set:** [a-z, A-Z, 0-9] = 62 characters

**Algorithm:**
1. Generate unique ID using database auto-increment or distributed ID generator
2. Encode ID to Base62
3. Result is the short code

**Example:**
```
ID: 125 → Base62: "cb"
ID: 3844 → Base62: "zy"
ID: 238327 → Base62: "W7d"
```

**Length Calculation:**
- 6 characters: 62^6 = 56 billion URLs
- 7 characters: 62^7 = 3.5 trillion URLs
- 8 characters: 62^8 = 218 trillion URLs

**For 6 billion URLs over 5 years, 6 characters is sufficient.**

### Alternative: Hash-Based Approach

Use MD5/SHA-256 hash of original URL + timestamp, take first 6-8 characters.
- Pros: Deterministic, no database lookup for generation
- Cons: Collision handling needed, less compact

### Handling Custom Aliases

1. Check if custom alias is available (query database)
2. If available, use it directly (skip ID generation)
3. If taken, return 409 Conflict error

---

## 6. Trade-offs and Optimizations

### Key Design Decisions

**1. NoSQL vs SQL:**
- **Choice:** NoSQL (Cassandra/DynamoDB)
- **Reasoning:** Simple key-value access pattern, easy horizontal scaling, high read throughput
- **Trade-off:** Less flexible queries, but we don't need complex queries

**2. 301 vs 302 Redirect:**
- **Choice:** 302 Found (Temporary)
- **Reasoning:** Allows tracking every click; 301 causes browsers to cache, preventing analytics
- **Trade-off:** Slightly more server load, but analytics are valuable

**3. Cache Strategy:**
- **Choice:** Cache-aside with no TTL
- **Reasoning:** URLs are immutable; popular URLs benefit from caching
- **Trade-off:** Cache memory usage, but worth it for performance

### Potential Bottlenecks

**1. Database Write Throughput:**
- At 38 writes/sec, not a bottleneck initially
- Solution: Shard database by hash of short_code if needed

**2. Database Read Throughput:**
- 3,800-11,400 reads/sec can strain a single database
- Solution: Read replicas + aggressive caching

**3. Single Point of Failure:**
- Load balancer, cache, database all can fail
- Solution: Redundancy at every layer

**4. ID Generation:**
- Single auto-increment can be a bottleneck
- Solution: Use distributed ID generator (Snowflake, UUID)

### Scaling Strategy

**Phase 1 (0-1M daily active users):**
- Single application server
- Single database with read replica
- Redis cache

**Phase 2 (1M-10M daily active users):**
- Multiple application servers behind load balancer
- Database sharding by hash of short_code
- Redis cluster for cache

**Phase 3 (10M+ daily active users):**
- Geographic distribution with CDN
- Multiple database clusters across regions
- Rate limiting to prevent abuse
- Analytics offloaded to separate data warehouse

### Additional Optimizations

**1. Pre-generate Short URLs:**
- Generate short codes in advance during low-traffic periods
- Store in a pool; assign when user creates URL
- Improves creation latency

**2. Bloom Filter:**
- Before checking database for custom alias availability
- Quick negative check: "definitely not available"
- Reduces unnecessary database queries

**3. Rate Limiting:**
- Prevent abuse (users creating millions of URLs)
- Use token bucket algorithm
- Limit: 100 URLs per user per hour

**4. Analytics Optimization:**
- Update click_count asynchronously
- Write to message queue instead of synchronous database update
- Batch updates every few seconds

### Security Considerations

**1. URL Validation:**
- Validate original URL format
- Check against malicious domains
- Prevent open redirect vulnerabilities

**2. Rate Limiting:**
- Prevent DDoS on URL creation
- Limit based on IP address or API key

**3. Short Code Predictability:**
- Random-appearing codes prevent enumeration
- Base62 encoding provides this naturally

---

## Summary

This design handles:
- ✅ 100M URL shortenings per month
- ✅ 100:1 read/write ratio through caching
- ✅ Low latency (< 100ms) via cache and NoSQL
- ✅ High availability through redundancy
- ✅ Scalability through horizontal scaling

The system is cost-effective, performant, and can scale to billions of URLs with minimal modifications.
