# Design Challenge 3: Distributed Rate Limiter - Sample Solution

## 1. Algorithm Selection

### Comparison of Rate Limiting Algorithms:

**1. Token Bucket:**
- Tokens added at fixed rate; each request consumes token
- Pros: Allows bursts, smooth traffic flow
- Cons: More complex implementation
- Best for: Variable traffic with burst tolerance

**2. Leaky Bucket:**
- Requests queued and processed at fixed rate
- Pros: Smooth, predictable output rate
- Cons: Doesn't handle bursts well, adds latency
- Best for: Smoothing erratic traffic

**3. Fixed Window Counter:**
- Count requests in fixed time windows (e.g., 1 minute)
- Pros: Simple, memory efficient
- Cons: Burst at window boundaries (2x limit possible)
- Best for: Approximate rate limiting

**4. Sliding Window Log:**
- Store timestamp of each request, count requests in sliding window
- Pros: Very accurate
- Cons: Memory intensive (stores all request timestamps)
- Best for: When precision is critical

**5. Sliding Window Counter:**
- Hybrid of fixed window and sliding window log
- Pros: Accurate, memory efficient
- Cons: Slightly complex calculation
- Best for: Production systems needing accuracy and efficiency

### **Choice: Sliding Window Counter**

**Justification:**
- Balances accuracy with memory efficiency
- Handles window boundary burst problem
- Simple enough for low-latency requirements (< 10ms)
- Works well in distributed environment

---

## 2. API Design

### Rate Limiter Interface

**Middleware/Service Call:**
```python
result = rate_limiter.check_limit(
    key="user:12345",
    limit=100,
    window_seconds=60
)

if result.allowed:
    # Process request
else:
    # Return 429 Too Many Requests
```

### HTTP Response Headers

**On Success (within limit):**
```
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1642234567
```

**On Rate Limit Exceeded:**
```
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1642234567
Retry-After: 23

{
  "error": "Rate limit exceeded",
  "message": "You have exceeded the rate limit of 100 requests per minute",
  "retry_after_seconds": 23
}
```

### Configuration API

```
POST /api/v1/rate-limits
{
  "rule_id": "api_endpoint_limit",
  "key_pattern": "endpoint:/api/search",
  "limit": 1000,
  "window_seconds": 60,
  "user_tier": "premium"
}

GET /api/v1/rate-limits/{rule_id}
PUT /api/v1/rate-limits/{rule_id}
DELETE /api/v1/rate-limits/{rule_id}
```

---

## 3. Data Model

### Rate Limit Rules (Configuration)

```json
{
  "rule_id": "user_free_tier",
  "key_pattern": "user:{user_id}",
  "limit": 100,
  "window_seconds": 60,
  "tier": "free",
  "enabled": true
}
```

### Redis Data Structures for Counters

**Sliding Window Counter Implementation:**

```
Key: ratelimit:{key}:{window_timestamp}
Value: request_count
TTL: 2 × window_seconds (keep two windows)

Example:
Key: ratelimit:user:12345:1642234500
Value: 45
TTL: 120 seconds
```

**User Tier Configuration:**
```
Key: user:tier:{user_id}
Value: "free" | "premium" | "enterprise"
TTL: 3600 (refresh periodically)
```

**Current Window State:**
```
Key: ratelimit:user:12345:current
Value: {
  "count": 45,
  "window_start": 1642234567,
  "limit": 100
}
TTL: window_seconds
```

---

## 4. High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                      Clients                            │
└────────────────────┬────────────────────────────────────┘
                     │
         ┌───────────┴───────────┐
         │                       │
    ┌────▼─────┐           ┌────▼─────┐
    │   API    │           │   API    │
    │  Server  │           │  Server  │
    │    1     │           │    2     │
    └────┬─────┘           └────┬─────┘
         │                       │
         │  ┌───────────────────┘
         │  │
    ┌────▼──▼─────────────────────────┐
    │   Rate Limiter Middleware        │
    │  (On each server)                │
    └────┬────────────────────┬────────┘
         │                    │
         │                    │
    ┌────▼────────┐     ┌────▼──────────┐
    │   Redis     │     │  Config       │
    │   Cluster   │     │  Service      │
    │  (Counters) │     │  (Rules)      │
    └─────────────┘     └───────────────┘
         │                    │
    ┌────▼────────────────────▼─────┐
    │     Monitoring & Alerts        │
    └────────────────────────────────┘
```

### Components:

**1. Rate Limiter Middleware:**
- Runs on each API server
- Evaluates requests before processing
- Low-latency checks (< 10ms)
- Stateless (state in Redis)

**2. Redis Cluster:**
- Stores rate limit counters
- High-performance key-value operations
- Cluster mode for high availability
- Atomic increment operations

**3. Configuration Service:**
- Stores rate limit rules
- Provides rule discovery and updates
- Caches rules locally on API servers
- Supports dynamic updates without restart

**4. Monitoring:**
- Tracks rate limit hits/misses
- Alerts on unusual patterns
- Dashboard for visibility

### Request Flow:

```
1. Client → API Server
2. API Server → Rate Limiter Middleware
3. Middleware → Local Config Cache (get rules)
4. Middleware → Redis (increment counter)
5. Redis → Return current count
6. Middleware → Calculate if within limit
7. If allowed: Continue to API handler
   If denied: Return 429 error
```

---

## 5. Handling Edge Cases

### 1. Clock Synchronization Issues

**Problem:** Different servers may have slightly different clocks

**Solution:**
- Use Unix timestamp rounded to window intervals
- All servers round to same window start time
- Example: Window=60s, all timestamps rounded to minute boundaries
- Acceptable drift: < 1 second (use NTP)

```python
window_start = int(current_timestamp / window_seconds) * window_seconds
```

### 2. Race Conditions

**Problem:** Multiple servers incrementing same counter simultaneously

**Solution using Redis Lua Script:**
```lua
-- Atomic check and increment
local key = KEYS[1]
local limit = tonumber(ARGV[1])
local window = tonumber(ARGV[2])
local current_time = tonumber(ARGV[3])

local current_count = redis.call('GET', key) or 0
current_count = tonumber(current_count)

if current_count < limit then
    redis.call('INCR', key)
    redis.call('EXPIRE', key, window)
    return 1  -- Allowed
else
    return 0  -- Denied
end
```

**Redis guarantees:**
- Lua scripts execute atomically
- Single-threaded execution prevents race conditions
- INCR operation is atomic

### 3. Cache (Redis) Failures

**Problem:** Redis cluster becomes unavailable

**Solution - Fail-open strategy:**
```python
try:
    result = check_redis_rate_limit(key, limit, window)
    return result
except RedisConnectionError:
    # Log error, alert monitoring
    logger.error("Redis unavailable, allowing request")
    # Fail-open: Allow request (favor availability over strict limiting)
    return RateLimitResult(allowed=True, is_fallback=True)
```

**Alternative - Local rate limiting fallback:**
- Keep in-memory counters on each server
- Less accurate but prevents total failure
- Limits are per-server, not global
- Example: If global limit is 1000/min, use 200/min per server (for 5 servers)

### 4. Sudden Traffic Spikes

**Problem:** DDoS or legitimate viral traffic spike

**Solution - Multiple defense layers:**

**a) Adaptive Rate Limiting:**
```python
if detect_anomaly(traffic_pattern):
    # Temporarily reduce limits by 50%
    effective_limit = configured_limit * 0.5
```

**b) Burst Tolerance:**
- Allow short bursts above limit (token bucket behavior)
- Example: Allow 120 requests in burst, but average of 100/min

**c) Priority Queuing:**
```python
if user.tier == "enterprise":
    limit = base_limit * 10
elif user.tier == "premium":
    limit = base_limit * 3
else:  # free tier
    limit = base_limit
```

**d) Circuit Breaker for Backend:**
- Even if request passes rate limiter, protect backend
- Reject requests if backend health is degraded

### 5. Distributed Counter Accuracy

**Problem:** Eventual consistency in distributed Redis cluster

**Solution - Sliding Window Counter Algorithm:**

```python
def check_rate_limit(key, limit, window_seconds):
    current_time = int(time.time())
    current_window = int(current_time / window_seconds) * window_seconds
    previous_window = current_window - window_seconds
    
    # Get counts from current and previous windows
    current_count = redis.get(f"{key}:{current_window}") or 0
    previous_count = redis.get(f"{key}:{previous_window}") or 0
    
    # Calculate weight of previous window
    elapsed_in_current = current_time - current_window
    weight = (window_seconds - elapsed_in_current) / window_seconds
    
    # Estimated count in sliding window
    estimated_count = current_count + (previous_count * weight)
    
    if estimated_count < limit:
        redis.incr(f"{key}:{current_window}")
        redis.expire(f"{key}:{current_window}", window_seconds * 2)
        return True
    return False
```

This algorithm provides smooth transition between windows and ~99% accuracy.

---

## 6. Trade-offs and Optimizations

### Key Design Decisions:

**1. Redis vs Database:**
- **Choice:** Redis
- **Reasoning:** Sub-millisecond latency, atomic operations, TTL support
- **Trade-off:** In-memory storage limits capacity, but sufficient for counters

**2. Fail-open vs Fail-closed:**
- **Choice:** Fail-open (allow requests when Redis is down)
- **Reasoning:** Favors availability over strict limiting
- **Trade-off:** Potential abuse during outages, but rare and detected quickly

**3. Centralized vs Edge Rate Limiting:**
- **Choice:** Centralized (Redis cluster)
- **Reasoning:** Accurate global limits, easier to manage
- **Trade-off:** Network latency to Redis, but < 10ms with proper setup

### Performance Optimizations:

**1. Local Config Caching:**
- Cache rate limit rules in memory on each API server
- TTL: 60 seconds
- Reduces config service load by 99%

**2. Connection Pooling:**
- Maintain persistent connections to Redis
- Reuse connections across requests
- Reduces connection overhead

**3. Pipelining:**
- Batch multiple Redis commands
- Reduces round trips

**4. Hot Key Optimization:**
- Detect hot keys (e.g., popular API endpoints)
- Use Redis Cluster with sharding to distribute load
- Consider local rate limiting for extremely hot keys

### Monitoring and Alerting:

**Metrics to Track:**
- Rate limit hit rate (% of requests denied)
- Redis latency (p50, p95, p99)
- Error rate (Redis failures)
- Top rate-limited users/IPs
- Rule effectiveness

**Alerts:**
- Redis unavailable
- Rate limit hit rate > 20% (potential attack)
- Redis latency > 10ms
- Configuration errors

---

## Summary

This design provides:
- ✅ Low latency (< 10ms overhead)
- ✅ High availability (fail-open strategy)
- ✅ Distributed consistency (Redis + Lua scripts)
- ✅ Scalable to 100K+ req/sec
- ✅ Accurate rate limiting (~99% accuracy)
- ✅ No single point of failure (Redis cluster)

The sliding window counter algorithm with Redis provides an excellent balance of accuracy, performance, and simplicity for production use.
