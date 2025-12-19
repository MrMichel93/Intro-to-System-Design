# Design Template: URL Shortener

## Problem Statement

Design a service like bit.ly or TinyURL that converts long URLs into short URLs and redirects users.

## Requirements

### Functional
- Given long URL, generate short URL
- Given short URL, redirect to original URL
- Optional: Custom short URLs
- Optional: Expiration time
- Optional: Analytics (click tracking)

### Non-Functional
- **Scale**: 100M URLs shortened per month
- **Reads**: 100:1 read-to-write ratio = 10B redirects/month
- **Latency**: Redirect in < 50ms
- **Availability**: 99.9%+
- **URL lifespan**: 10 years

## Capacity Estimation

### Storage
```
URLs per month: 100M
URLs per year: 1.2B
10 years: 12B URLs

Per URL:
- Original URL: 200 bytes
- Short code: 7 bytes
- Metadata: 100 bytes
- Total: ~300 bytes

Storage: 12B × 300 bytes = 3.6 TB ✓ (manageable)
```

### Traffic
```
Writes: 100M/month = ~40 writes/sec
Reads: 10B/month = 4,000 reads/sec
Peak (2x): 80 writes/sec, 8,000 reads/sec
```

### Bandwidth
```
Write: 40 writes/sec × 300 bytes = 12 KB/sec
Read: 4,000 reads/sec × 300 bytes = 1.2 MB/sec
```

## High-Level Architecture

```
┌──────────┐
│  Users   │
└─────┬────┘
      │
┌─────▼────────┐
│ Load Balancer│
└─────┬────────┘
      │
┌─────▼──────────────────┐
│   API Servers          │
│  - Shorten URL         │
│  - Redirect            │
└─────┬──────────────────┘
      │
┌─────▼─────┐    ┌─────────┐
│   Cache   │←───│Database │
│  (Redis)  │    │(SQL/NoSQL)│
└───────────┘    └─────────┘
```

## API Design

### 1. Shorten URL
```http
POST /api/shorten
Content-Type: application/json

Request:
{
  "long_url": "https://example.com/very/long/url",
  "custom_alias": "my-link",  // optional
  "expiry_date": "2025-12-31" // optional
}

Response:
{
  "short_url": "https://short.ly/abc123",
  "short_code": "abc123",
  "created_at": "2024-01-15T10:30:00Z"
}
```

### 2. Redirect
```http
GET /abc123

Response:
HTTP/1.1 301 Moved Permanently
Location: https://example.com/very/long/url
```

### 3. Get Stats (Optional)
```http
GET /api/stats/abc123

Response:
{
  "short_code": "abc123",
  "long_url": "https://example.com/very/long/url",
  "clicks": 1523,
  "created_at": "2024-01-15T10:30:00Z"
}
```

## Database Schema

### SQL Option
```sql
CREATE TABLE urls (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    short_code VARCHAR(7) UNIQUE NOT NULL,
    long_url TEXT NOT NULL,
    user_id BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP,
    clicks INT DEFAULT 0,
    INDEX idx_short_code (short_code),
    INDEX idx_user_id (user_id)
);
```

### NoSQL Option (MongoDB)
```javascript
{
  _id: ObjectId,
  short_code: "abc123",
  long_url: "https://example.com/very/long/url",
  user_id: "user123",
  created_at: ISODate("2024-01-15T10:30:00Z"),
  expires_at: ISODate("2025-12-31T00:00:00Z"),
  clicks: 1523
}

Index: { short_code: 1 } unique
Index: { user_id: 1 }
```

## Short Code Generation

### Option 1: Base62 Encoding

**Base62**: a-z, A-Z, 0-9 (62 characters)

```python
def encode_base62(num):
    chars = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
    base = len(chars)
    result = ""
    while num > 0:
        result = chars[num % base] + result
        num //= base
    return result

# Example:
encode_base62(12345) → "3D7"
```

**7 characters**: 62^7 = 3.5 trillion combinations ✓

### Option 2: Random Generation + Collision Check

```python
import random
import string

def generate_short_code(length=7):
    chars = string.ascii_letters + string.digits
    while True:
        code = ''.join(random.choice(chars) for _ in range(length))
        if not exists_in_database(code):
            return code
```

### Option 3: Hash Function

```python
import hashlib

def generate_short_code(long_url):
    hash_val = hashlib.md5(long_url.encode()).hexdigest()
    # Take first 7 characters
    return hash_val[:7]
    # Note: Need collision handling!
```

## Caching Strategy

### Cache Layer (Redis)

```
Key: short_code
Value: long_url
TTL: 24 hours

Example:
"abc123" → "https://example.com/very/long/url"
```

### Cache Flow
```
1. User requests /abc123
2. Check Redis: GET abc123
3. If HIT: Redirect to cached URL (fast!)
4. If MISS: Query database, cache result, redirect

Cache hit rate: 80%+ (popular links accessed repeatedly)
```

## Scalability Considerations

### Database Sharding
```
Shard by short_code:
shard = hash(short_code) % num_shards

Shard 1: a-f
Shard 2: g-m
Shard 3: n-s
Shard 4: t-z
```

### Read Replicas
```
1 Primary (writes)
3 Read Replicas (reads)

Distribute 4,000 reads/sec across replicas
= 1,000 reads/sec per replica
```

### Rate Limiting
```
Per user: 100 shortens per hour
Per IP: 1,000 redirects per hour
```

## Trade-Offs

### 1. Short Code Generation

**Base62 Encoding:**
- ✅ Predictable, sequential
- ❌ Can guess next URL

**Random:**
- ✅ Unpredictable
- ❌ Need collision handling

**Hash:**
- ✅ Deterministic (same URL → same code)
- ❌ Collisions more likely

**Choose**: Random for security, Base62 for simplicity

### 2. Redirect Type

**301 (Permanent):**
- ✅ Cached by browsers
- ❌ Can't track clicks accurately

**302 (Temporary):**
- ✅ Can track all clicks
- ❌ Slower (not cached)

**Choose**: 302 if analytics needed, 301 otherwise

### 3. Database Choice

**SQL:**
- ✅ ACID guarantees
- ✅ Good for relational data
- ❌ Harder to scale horizontally

**NoSQL:**
- ✅ Easy horizontal scaling
- ✅ High write throughput
- ❌ No ACID guarantees

**Choose**: NoSQL (Cassandra/MongoDB) for scale

## Optimizations

### 1. Pre-generate Short Codes
```
Background job generates millions of short codes
Store in database: unused_short_codes table
When user shortens URL: Pop from table (fast!)
```

### 2. Bloom Filter
```
Before checking database, check Bloom filter:
"Does this short code exist?"
If NO: Definitely doesn't exist (skip DB query)
If YES: Maybe exists (check DB)

Saves ~40% of DB queries for invalid codes
```

### 3. Analytics Async
```
Don't increment clicks synchronously
- Write to message queue (Kafka)
- Consumer updates database
- User redirect not blocked
```

## Complete Implementation Flow

### Shorten URL
```
1. User submits long URL
2. Validate URL (format, length)
3. Check if URL already shortened (optional)
4. Generate short code
5. Check for collision
6. Store in database
7. Return short URL to user

Time: ~50ms
```

### Redirect
```
1. User visits short URL
2. Extract short code
3. Check Redis cache
4. If miss: Query database
5. If found: Cache + redirect
6. If not found: 404 error
7. (Async) Track click

Time: ~10ms (cache hit), ~50ms (cache miss)
```

## Summary

URL Shortener teaches:
- ✅ Hash/encoding algorithms
- ✅ Caching strategies
- ✅ Database design
- ✅ Read-heavy optimization
- ✅ API design
- ✅ Scale estimation

**Key Insight**: Simple problem, but teaches many fundamental concepts!

Use this template as a foundation and customize based on specific requirements.
