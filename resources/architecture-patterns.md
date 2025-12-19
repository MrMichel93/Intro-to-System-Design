# Architecture Patterns Catalog

A comprehensive guide to common architecture patterns in system design, including when to use each pattern, trade-offs, and real-world examples.

## Table of Contents
- [Caching Patterns](#caching-patterns)
- [Database Patterns](#database-patterns)
- [Communication Patterns](#communication-patterns)
- [Scalability Patterns](#scalability-patterns)
- [Reliability Patterns](#reliability-patterns)
- [Data Management Patterns](#data-management-patterns)
- [Microservices Patterns](#microservices-patterns)
- [Performance Patterns](#performance-patterns)

---

## Caching Patterns

### 1. Cache-Aside (Lazy Loading)

**Description:**
Application code directly manages the cache. On read, check cache first; if miss, read from database and populate cache. On write, update database and invalidate cache.

**When to Use:**
- Read-heavy workloads
- Infrequently changing data
- You want fine-grained control over caching

**Pros:**
- Only caches data that's actually requested
- Cache failures don't break the application
- Simple to implement

**Cons:**
- Cache misses result in three trips (cache read, DB read, cache write)
- Data inconsistency possible between cache and DB
- Initial requests are slow (cold start)

**Example:**
```python
def get_user(user_id):
    # Try cache first
    user = cache.get(f"user:{user_id}")
    if user is None:
        # Cache miss - read from database
        user = database.query(f"SELECT * FROM users WHERE id = {user_id}")
        # Populate cache
        cache.set(f"user:{user_id}", user, ttl=3600)
    return user

def update_user(user_id, data):
    # Update database
    database.update(f"UPDATE users SET ... WHERE id = {user_id}")
    # Invalidate cache
    cache.delete(f"user:{user_id}")
```

**Real-world Examples:** Amazon product pages, Facebook user profiles

**Related Module:** [Module 5: Caching](../05-caching/)

---

### 2. Read-Through Cache

**Description:**
Cache sits between application and database. On cache miss, the cache itself loads data from the database.

**When to Use:**
- Read-heavy workloads
- You want to abstract cache management from application code
- Consistent data access patterns

**Pros:**
- Simpler application code
- Cache manages data loading
- Consistent interface for all reads

**Cons:**
- Cache becomes a critical dependency
- First access is still slow (cold start)
- Less flexibility in cache logic

**Example Use Cases:** Content management systems, e-commerce catalogs

**Related Module:** [Module 5: Caching](../05-caching/)

---

### 3. Write-Through Cache

**Description:**
All writes go through the cache first, then to the database. Cache and database are always in sync.

**When to Use:**
- Strong consistency required
- Read-heavy with occasional writes
- Data must be immediately available after write

**Pros:**
- Cache is always consistent with database
- No data loss on cache failure
- Reads always hit warm cache for recently written data

**Cons:**
- Every write has higher latency (two operations)
- May cache data that's never read
- More complex to implement

**Example Use Cases:** Financial transactions, inventory systems

**Related Module:** [Module 5: Caching](../05-caching/)

---

### 4. Write-Behind Cache (Write-Back)

**Description:**
Writes go to cache immediately and are asynchronously written to the database later.

**When to Use:**
- Write-heavy workloads
- Can tolerate eventual consistency
- Need to reduce write latency

**Pros:**
- Very fast writes (cache only)
- Can batch database writes for efficiency
- Reduces database load

**Cons:**
- Risk of data loss if cache fails before DB write
- More complex to implement
- Eventual consistency between cache and DB

**Example Use Cases:** Analytics data collection, logging systems, gaming leaderboards

**Related Module:** [Module 5: Caching](../05-caching/)

---

## Database Patterns

### 5. Database Replication (Leader-Follower)

**Description:**
One primary (leader) database handles writes; multiple replica (follower) databases handle reads. Changes are replicated from leader to followers.

**When to Use:**
- Read-heavy workloads (90%+ reads)
- Need high availability for reads
- Geographic distribution of users

**Pros:**
- Scales read capacity horizontally
- High availability (if leader fails, promote follower)
- Reduced latency with geo-distributed replicas

**Cons:**
- Replication lag (eventual consistency)
- Writes don't scale (single leader)
- More complex failover logic

**Diagram:**
```
          ┌─────────┐
          │ Leader  │ ← All Writes
          │   DB    │
          └────┬────┘
               │
        ┌──────┼──────┐
        ▼      ▼      ▼
    ┌───────┐ ┌───────┐ ┌───────┐
    │Replica│ │Replica│ │Replica│ ← Reads
    │  DB 1 │ │  DB 2 │ │  DB 3 │
    └───────┘ └───────┘ └───────┘
```

**Real-world Examples:** Reddit, GitHub, most web applications

**Related Module:** [Module 8: Replication](../08-replication/)

---

### 6. Database Sharding (Horizontal Partitioning)

**Description:**
Split data across multiple databases based on a shard key. Each shard is an independent database.

**When to Use:**
- Single database can't handle the load
- Data grows beyond single server capacity
- Need to scale both reads and writes

**Pros:**
- Scales both reads and writes
- Reduces impact of failures (only one shard affected)
- Can optimize storage per shard

**Cons:**
- Complex implementation
- Cross-shard queries are difficult
- Resharding is expensive
- Need to choose good shard key

**Sharding Strategies:**
- **Hash-based:** `shard = hash(user_id) % num_shards`
- **Range-based:** `shard = user_id / 1000000`
- **Geographic:** `shard = user_location`

**Example Use Cases:** Instagram (by user_id), Uber (by geographic region)

**Related Module:** [Module 7: Data Partitioning](../07-data-partitioning/)

---

### 7. CQRS (Command Query Responsibility Segregation)

**Description:**
Separate models for reading (query) and writing (command) data. Read model is optimized for queries; write model for updates.

**When to Use:**
- Complex domain logic
- Very different read and write patterns
- Need to scale reads and writes independently

**Pros:**
- Optimizes reads and writes separately
- Can use different storage for each
- Enables event sourcing
- Scales independently

**Cons:**
- Increased complexity
- Eventual consistency between models
- More infrastructure to maintain

**Diagram:**
```
Commands (Writes)          Queries (Reads)
      │                         │
      ▼                         ▼
┌──────────┐              ┌──────────┐
│  Write   │   Events     │   Read   │
│  Model   │─────────────▶│  Model   │
└──────────┘              └──────────┘
      │                         │
      ▼                         ▼
┌──────────┐              ┌──────────┐
│ Write DB │              │ Read DB  │
│(Postgres)│              │ (Elastic)│
└──────────┘              └──────────┘
```

**Real-world Examples:** E-commerce platforms, complex booking systems

**Related Module:** [Module 6: Database Design](../06-database-design/)

---

## Communication Patterns

### 8. Request-Response (Synchronous)

**Description:**
Client sends request and waits for response. Direct, blocking communication.

**When to Use:**
- Need immediate response
- Simple, sequential workflows
- Tight coupling is acceptable

**Pros:**
- Simple to implement and understand
- Immediate feedback
- Easy error handling

**Cons:**
- Blocking (caller waits)
- Tight coupling between services
- Cascading failures possible
- Poor for long-running operations

**Example Use Cases:** REST APIs, RPC calls, database queries

**Related Module:** [Module 10: API Design](../10-api-design/)

---

### 9. Publish-Subscribe (Pub/Sub)

**Description:**
Publishers send messages to topics; subscribers receive messages from topics they're interested in. Decoupled communication.

**When to Use:**
- Multiple consumers need same data
- Event-driven architecture
- Loose coupling desired
- Fan-out scenarios

**Pros:**
- Loose coupling (publishers don't know subscribers)
- Easy to add new subscribers
- Scales well for fan-out
- Asynchronous processing

**Cons:**
- No immediate response
- Message delivery guarantees vary
- Debugging is harder
- Potential message ordering issues

**Diagram:**
```
┌───────────┐       ┌────────┐       ┌──────────────┐
│ Publisher │──────▶│ Topic  │──────▶│ Subscriber 1 │
└───────────┘       │ (User  │       └──────────────┘
                    │Created)│       ┌──────────────┐
                    └────────┘──────▶│ Subscriber 2 │
                                     └──────────────┘
                                     ┌──────────────┐
                                ────▶│ Subscriber 3 │
                                     └──────────────┘
```

**Real-world Examples:** YouTube notifications, Slack messages, email systems

**Related Module:** [Module 11: Message Queues](../11-message-queues/)

---

### 10. Message Queue (Point-to-Point)

**Description:**
Messages are sent to a queue; consumers pull and process messages one at a time.

**When to Use:**
- Asynchronous task processing
- Work distribution among workers
- Need guaranteed processing
- Rate limiting/smoothing

**Pros:**
- Decouples producers and consumers
- Load leveling/smoothing
- Retry and dead letter queues
- Guaranteed delivery

**Cons:**
- Added complexity
- Latency introduced
- Need to handle failures
- Message ordering can be tricky

**Example Use Cases:** Email sending, image processing, order fulfillment

**Related Module:** [Module 11: Message Queues](../11-message-queues/)

---

### 11. API Gateway Pattern

**Description:**
Single entry point for all client requests. Routes requests to appropriate services and handles cross-cutting concerns.

**When to Use:**
- Microservices architecture
- Multiple client types (web, mobile)
- Need centralized auth, rate limiting, logging

**Pros:**
- Single entry point
- Centralized cross-cutting concerns
- Protocol translation
- Request/response transformation

**Cons:**
- Single point of failure
- Can become a bottleneck
- Added latency
- Increased complexity

**Responsibilities:**
- Authentication/Authorization
- Rate limiting
- Request routing
- Response transformation
- Monitoring/Logging
- Caching

**Real-world Examples:** Netflix Zuul, Amazon API Gateway, Kong

**Related Module:** [Module 12: Microservices](../12-microservices/)

---

## Scalability Patterns

### 12. Load Balancing

**Description:**
Distribute incoming requests across multiple servers to improve availability and performance.

**When to Use:**
- Multiple server instances
- Need high availability
- Traffic exceeds single server capacity

**Algorithms:**
- **Round Robin:** Distribute requests evenly
- **Least Connections:** Send to server with fewest active connections
- **Weighted:** Prefer more powerful servers
- **IP Hash:** Same client always goes to same server

**Pros:**
- Horizontal scaling
- High availability
- No single point of failure
- Health checking

**Cons:**
- Added complexity
- Session management challenges
- Load balancer can be bottleneck

**Real-world Examples:** All major web applications (Facebook, Google, Netflix)

**Related Module:** [Module 4: Load Balancing](../04-load-balancing/)

---

### 13. Auto-Scaling

**Description:**
Automatically adjust the number of servers based on current load metrics.

**When to Use:**
- Variable/unpredictable traffic
- Cost optimization important
- Cloud infrastructure

**Metrics to Monitor:**
- CPU utilization
- Memory usage
- Request count
- Response time
- Custom metrics

**Pros:**
- Handles traffic spikes automatically
- Cost efficient (scale down when not needed)
- Improves availability

**Cons:**
- Scaling delay (time to provision)
- Can be expensive if misconfigured
- Cold start issues

**Real-world Examples:** E-commerce during sales, news sites during breaking news

**Related Module:** [Module 3: Scalability](../03-scalability/)

---

### 14. Content Delivery Network (CDN)

**Description:**
Geographically distributed network of servers that cache and serve content from locations near users.

**When to Use:**
- Global user base
- Static content (images, videos, CSS, JS)
- High bandwidth content
- Need to reduce origin load

**Pros:**
- Reduced latency (geographically closer)
- Reduced origin server load
- Better availability
- DDoS protection

**Cons:**
- Cost (CDN services)
- Cache invalidation complexity
- Not suitable for dynamic content

**Content Types:**
- Static assets (images, CSS, JS)
- Video streaming
- Software downloads
- API responses (with caching)

**Real-world Examples:** Netflix, YouTube, large websites

**Related Module:** [Module 14: CDN](../14-cdn/)

---

## Reliability Patterns

### 15. Circuit Breaker

**Description:**
Prevent cascading failures by detecting failures and stopping requests to failing service temporarily.

**States:**
- **Closed:** Normal operation, requests pass through
- **Open:** Too many failures, block all requests
- **Half-Open:** Trial period, allow limited requests

**When to Use:**
- Calling external services
- Microservices architecture
- Need to prevent cascading failures

**Pros:**
- Prevents cascading failures
- Fails fast
- Allows service to recover
- Can provide fallback responses

**Cons:**
- Added complexity
- Need to tune thresholds
- May block valid requests

**Example:**
```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, timeout=60):
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.state = 'CLOSED'
        self.last_failure_time = None
    
    def call(self, func):
        if self.state == 'OPEN':
            if time.now() - self.last_failure_time > self.timeout:
                self.state = 'HALF_OPEN'
            else:
                raise CircuitBreakerOpen()
        
        try:
            result = func()
            if self.state == 'HALF_OPEN':
                self.state = 'CLOSED'
                self.failure_count = 0
            return result
        except Exception as e:
            self.failure_count += 1
            self.last_failure_time = time.now()
            if self.failure_count >= self.failure_threshold:
                self.state = 'OPEN'
            raise e
```

**Real-world Examples:** Netflix Hystrix, Resilience4j

**Related Module:** [Module 13: Availability Patterns](../13-availability-patterns/)

---

### 16. Bulkhead Pattern

**Description:**
Isolate resources for different parts of the system to prevent failure in one area from affecting others.

**When to Use:**
- Different services or operations have different priorities
- Want to prevent resource exhaustion
- Multi-tenant systems

**Pros:**
- Failure isolation
- Predictable resource usage
- Prevents one service from starving others

**Cons:**
- Resource underutilization
- More complex configuration
- May need more total resources

**Example:**
```
Thread Pool for Critical Operations (20 threads)
Thread Pool for Background Tasks (5 threads)
Thread Pool for Analytics (10 threads)
```

**Real-world Examples:** Connection pools, thread pools in application servers

**Related Module:** [Module 13: Availability Patterns](../13-availability-patterns/)

---

### 17. Retry with Exponential Backoff

**Description:**
Automatically retry failed operations with increasing delays between attempts.

**When to Use:**
- Transient failures (network blips, temporary overload)
- External service calls
- Distributed systems

**Pros:**
- Handles transient failures automatically
- Reduces load during recovery
- Simple to implement

**Cons:**
- May hide underlying problems
- Can increase overall latency
- Need to be idempotent

**Example:**
```python
def retry_with_backoff(func, max_retries=5):
    for attempt in range(max_retries):
        try:
            return func()
        except TransientError as e:
            if attempt == max_retries - 1:
                raise
            wait_time = (2 ** attempt) + random.uniform(0, 1)  # Jitter
            time.sleep(wait_time)
```

**Backoff Strategy:**
```
Attempt 1: Immediate
Attempt 2: Wait 1 second
Attempt 3: Wait 2 seconds
Attempt 4: Wait 4 seconds
Attempt 5: Wait 8 seconds
```

**Related Module:** [Module 13: Availability Patterns](../13-availability-patterns/)

---

### 18. Health Check Pattern

**Description:**
Regularly check the health of services and remove unhealthy instances from load balancer rotation.

**When to Use:**
- Multiple service instances
- Load balancing
- Need high availability

**Types:**
- **Shallow:** Simple ping or HTTP 200 response
- **Deep:** Check database connections, dependencies, disk space

**Pros:**
- Automatic failure detection
- Prevents requests to unhealthy instances
- Enables auto-healing

**Cons:**
- Health checks add load
- False positives possible
- Need to define "healthy"

**Example:**
```
GET /health
Response:
{
  "status": "healthy",
  "database": "connected",
  "cache": "connected",
  "disk_space": "85% used"
}
```

**Related Module:** [Module 13: Availability Patterns](../13-availability-patterns/)

---

## Data Management Patterns

### 19. Event Sourcing

**Description:**
Store all changes to application state as a sequence of events rather than just the current state.

**When to Use:**
- Need complete audit trail
- Temporal queries (state at any point in time)
- Complex business logic
- Event-driven architecture

**Pros:**
- Complete history of all changes
- Can reconstruct state at any time
- Natural fit for event-driven systems
- Enables time travel debugging

**Cons:**
- More complex to implement
- Event schema evolution challenges
- Querying current state can be slower
- Storage grows continuously

**Example:**
```
Events:
1. UserCreated(user_id=1, email="user@example.com")
2. EmailChanged(user_id=1, new_email="new@example.com")
3. UserDeactivated(user_id=1)

Current State = Replay all events
```

**Real-world Examples:** Banking systems, inventory systems

**Related Module:** [Module 9: Consistency Patterns](../09-consistency-patterns/)

---

### 20. Saga Pattern

**Description:**
Manage distributed transactions across multiple services using a sequence of local transactions with compensating actions.

**When to Use:**
- Microservices architecture
- Need distributed transactions
- Can't use 2PC (two-phase commit)

**Types:**
- **Choreography:** Each service listens to events and decides what to do
- **Orchestration:** Central coordinator tells services what to do

**Pros:**
- Works across service boundaries
- Better availability than 2PC
- Flexible failure handling

**Cons:**
- Complex to implement
- Need compensating transactions
- Eventual consistency only

**Example: E-commerce Order**
```
1. Reserve Inventory → Success
2. Process Payment → Success
3. Ship Order → Failure

Compensations:
3. (skip - never executed)
2. Refund Payment
1. Release Inventory
```

**Related Module:** [Module 12: Microservices](../12-microservices/)

---

### 21. Materialized View Pattern

**Description:**
Pre-compute and store query results for complex or frequently accessed data.

**When to Use:**
- Complex queries that are expensive to compute
- Frequently accessed data
- Acceptable to have slightly stale data

**Pros:**
- Fast reads (pre-computed)
- Reduced database load
- Can optimize for specific queries

**Cons:**
- Storage overhead
- Stale data (eventual consistency)
- Complexity in keeping views updated

**Example:**
```
Original: Complex join of 5 tables for user dashboard
Materialized View: Pre-joined data updated every 5 minutes
Result: 100x faster dashboard loads
```

**Real-world Examples:** Analytics dashboards, reporting systems

**Related Module:** [Module 6: Database Design](../06-database-design/)

---

## Microservices Patterns

### 22. Strangler Fig Pattern

**Description:**
Gradually replace a legacy system by incrementally building a new system around it.

**When to Use:**
- Migrating from monolith to microservices
- Can't afford big-bang rewrite
- Need to maintain service during migration

**Pros:**
- Incremental migration
- Reduced risk
- Can validate each step
- Business continuity maintained

**Cons:**
- Long migration period
- Complexity of running both systems
- Need routing layer

**Strategy:**
```
1. Identify a bounded context
2. Build new microservice
3. Route traffic to new service
4. Deprecate old code
5. Repeat for next context
```

**Related Module:** [Module 12: Microservices](../12-microservices/)

---

### 23. Sidecar Pattern

**Description:**
Deploy helper components alongside main application to provide supporting features.

**When to Use:**
- Cross-cutting concerns (logging, monitoring, security)
- Multiple languages/frameworks
- Need to add features without changing code

**Pros:**
- Language/framework agnostic
- Separation of concerns
- Reusable across services
- Isolated updates

**Cons:**
- Increased resource usage
- More complex deployment
- Network hop overhead

**Common Sidecars:**
- Logging and monitoring
- Service mesh proxies
- Secret management
- Configuration management

**Real-world Examples:** Istio, Envoy, Linkerd

**Related Module:** [Module 12: Microservices](../12-microservices/)

---

### 24. Service Mesh Pattern

**Description:**
Infrastructure layer that handles service-to-service communication with features like load balancing, service discovery, and observability.

**When to Use:**
- Many microservices (10+)
- Need consistent communication patterns
- Complex networking requirements

**Pros:**
- Centralized traffic management
- Observability built-in
- Security features (mTLS)
- Traffic control (retries, timeouts, circuit breakers)

**Cons:**
- Added complexity
- Operational overhead
- Performance impact
- Learning curve

**Features:**
- Service discovery
- Load balancing
- Failure recovery
- Metrics and monitoring
- Distributed tracing
- mTLS encryption

**Real-world Examples:** Istio, Linkerd, Consul Connect

**Related Module:** [Module 12: Microservices](../12-microservices/)

---

## Performance Patterns

### 25. Database Connection Pooling

**Description:**
Maintain a pool of reusable database connections instead of creating new connections for each request.

**When to Use:**
- High-traffic applications
- Database connections are expensive
- Multiple concurrent requests

**Pros:**
- Reduced connection overhead
- Better resource utilization
- Predictable performance

**Cons:**
- Connection leaks possible
- Need to tune pool size
- Stale connections

**Configuration:**
```
Min Connections: 5
Max Connections: 50
Connection Timeout: 30 seconds
Idle Timeout: 10 minutes
```

**Related Module:** [Module 6: Database Design](../06-database-design/)

---

### 26. Rate Limiting / Throttling

**Description:**
Limit the number of requests a user or service can make in a given time period.

**When to Use:**
- Prevent abuse
- Protect against DDoS
- Fair resource allocation
- API monetization

**Algorithms:**
- **Token Bucket:** Allow bursts, maintain average rate
- **Leaky Bucket:** Fixed rate processing
- **Fixed Window:** Simple but can burst at boundaries
- **Sliding Window:** More accurate, more complex

**Pros:**
- Protects system from overload
- Fair resource allocation
- Prevents abuse

**Cons:**
- Legitimate users may be throttled
- Complexity in distributed systems
- Need to tune limits

**Example:**
```
Rate Limit: 100 requests per minute per user
Burst: Up to 20 requests in a second
```

**Related Module:** [Module 10: API Design](../10-api-design/)

---

## Pattern Selection Guide

### By Use Case

**High Read Traffic:**
- Caching (Cache-Aside, Read-Through)
- Database Replication (Leader-Follower)
- CDN
- Load Balancing

**High Write Traffic:**
- Database Sharding
- Write-Behind Cache
- Message Queues
- CQRS

**High Availability:**
- Load Balancing
- Database Replication
- Circuit Breaker
- Health Checks
- Multi-Region Deployment

**Global Users:**
- CDN
- Multi-Region Database
- Geographic Sharding
- Edge Computing

**Microservices:**
- API Gateway
- Service Mesh
- Saga Pattern
- Circuit Breaker

**Event Processing:**
- Pub/Sub
- Message Queue
- Event Sourcing
- Stream Processing

---

## Combining Patterns

Patterns are often used together. Here are common combinations:

### E-commerce Platform
- Load Balancing (distribute traffic)
- Cache-Aside (product catalog)
- Database Sharding (orders by user)
- Message Queue (order processing)
- Circuit Breaker (payment service)
- CDN (static assets)

### Social Media Platform
- CDN (images, videos)
- Cache-Aside (user profiles, feeds)
- Database Replication (read scaling)
- Pub/Sub (notifications)
- Sharding (by user ID)
- Rate Limiting (API protection)

### Video Streaming Service
- CDN (video distribution)
- Adaptive Bitrate Streaming
- Database Replication (metadata)
- Message Queue (encoding jobs)
- Cache (user preferences)
- Load Balancing (API servers)

---

## References

- [Module 0: Foundations](../00-foundations/)
- [Module 4: Load Balancing](../04-load-balancing/)
- [Module 5: Caching](../05-caching/)
- [Module 6: Database Design](../06-database-design/)
- [Module 7: Data Partitioning](../07-data-partitioning/)
- [Module 8: Replication](../08-replication/)
- [Module 11: Message Queues](../11-message-queues/)
- [Module 12: Microservices](../12-microservices/)
- [Module 13: Availability Patterns](../13-availability-patterns/)

For practical applications of these patterns, see the [Case Studies](../case-studies/) directory.
