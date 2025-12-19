# Self-Check Quiz Answers

This document contains answers to all self-check quizzes. Use this to verify your understanding after completing each quiz.

---

## Module 0: System Design Foundations

### Multiple Choice Answers
1. **B** - To understand what features the system should provide
2. **C** - The system should handle 10,000 requests per second (this is a performance/throughput requirement)
3. **B** - Less than 9 hours per year (99.9% = 8.76 hours downtime per year)
4. **B** - Sacrificing one characteristic to improve another
5. **B** - Vertical scaling (adding more resources to existing server)

### Short Answer Sample Answers

**Question 6:** Functional requirements define what the system does (e.g., "Users can post photos"). Non-functional requirements define how well it performs (e.g., "Posts must load within 200ms").

**Question 7:** Clarifying constraints prevents building the wrong solution. A system for 100 users is vastly different from one for 100 million users. Assumptions about scale, geography, budget, and traffic patterns fundamentally shape architecture decisions.

**Question 8:** 
- How many URLs will be shortened per day/month?
- What is the expected read/write ratio?
- Should shortened URLs expire, and if so, after how long?
- Do we need analytics on click-through rates?
- What is the expected lifespan of the service?

**Question 9:** Social media "likes" count - it's acceptable if users see slightly different like counts temporarily. High availability ensures users can always interact, even during network issues. Eventual consistency is acceptable since exact counts aren't critical.

**Question 10:** Scalability and cost are inversely related initially. Scaling requires more resources (servers, bandwidth, storage), increasing costs. However, good design can improve cost-efficiency at scale through caching, CDNs, and efficient resource usage. Design decisions must balance current costs with future scalability needs.

---

## Module 1: Back-of-Envelope Calculations

### Multiple Choice Answers
1. **B** - 86,400 seconds (24 hours × 60 minutes × 60 seconds)
2. **B** - ~1,000 QPS (100,000,000 / 86,400 ≈ 1,157)
3. **B** - 100 MB (1,000,000 × 100 bytes = 100,000,000 bytes = ~95 MB ≈ 100 MB)
4. **B** - 100 nanoseconds
5. **C** - 80 seconds (1 GB = 8,000 Mb; 8,000 Mb / 100 Mbps = 80 seconds)

### Short Answer Sample Answers

**Question 6:** 
- Total bandwidth = 10 million users × 5 Mbps
- = 50,000,000 Mbps
- = 50,000 Gbps or 50 Tbps

**Question 7:**
- Total photos = 500 million users × 50 photos = 25 billion photos
- Total storage = 25 billion × 2 MB = 50 billion MB = 50,000 TB = 50 PB

**Question 8:** Approximations are acceptable because: (1) Requirements often change, making precision unnecessary; (2) Exact numbers are unknown early in design; (3) The goal is to understand order of magnitude, not exact values; (4) Saves time during interviews and planning; (5) Systems have built-in margins anyway.

**Question 9:**
a) 1000 writes/sec × 1 KB = 1000 KB/s ≈ 1 MB/s
b) 1 MB/s × 86,400 seconds/day = 86,400 MB/day ≈ 84 GB/day

**Question 10:** The 80/20 rule (Pareto principle) states that 80% of effects come from 20% of causes. In system design: 80% of requests come from 20% of users, or 80% of traffic hits 20% of content. This means caching the "hot" 20% can satisfy most requests, dramatically reducing database load.

---

## Module 2: CAP Theorem

### Multiple Choice Answers
1. **B** - Consistency, Availability, Partition tolerance
2. **B** - Any two (you can only guarantee two out of three)
3. **C** - Availability and Partition tolerance (AP)
4. **B** - The system continues to operate despite network failures
5. **C** - MongoDB with default settings (CP system)

### Short Answer Sample Answers

**Question 6:** Eventual consistency means replicas may temporarily have different values but will eventually converge. Example: Social media likes - if you like a post, other users might not see your like immediately, but within seconds, everyone will see the updated count. This is acceptable for non-critical data.

**Question 7:** Modern distributed systems span multiple networks and data centers. Network failures (partitions) are inevitable due to router failures, cable cuts, or misconfigurations. Since partitions will happen, designers must choose between consistency and availability during partitions, making P non-negotiable.

**Question 8:** 
- **CP System (e.g., banking)**: Prioritizes consistency and partition tolerance. During network issues, the system may become unavailable rather than show inconsistent data. Use case: ATM withdrawals where showing incorrect balance could lead to overdrafts.
- **AP System (e.g., shopping cart)**: Prioritizes availability and partition tolerance. Users can always add items to cart, even if replicas aren't perfectly synchronized. Use case: E-commerce where availability is more important than perfect consistency.

**Question 9:** Banking transactions - if you choose availability over consistency during a partition, users in different regions might see different account balances. One user could withdraw money that doesn't actually exist, leading to overdrafts or fraud.

**Question 10:** Prioritize availability (AP). Reasoning: Users should always be able to like posts even during network issues. If likes are temporarily inconsistent across regions (one user sees 100 likes, another sees 101), it doesn't significantly impact user experience. The system can resolve conflicts later when the partition heals. User engagement is more important than perfect like counts.

---

## Module 3: Scalability

### Multiple Choice Answers
1. **B** - Vertical is upgrading existing servers, horizontal is adding more servers
2. **B** - Physical hardware limits (you can only add so much RAM/CPU to one machine)
3. **C** - Better fault tolerance through redundancy
4. **B** - Load balancer
5. **B** - The server doesn't store session-specific data locally

### Short Answer Sample Answers

**Question 6:** Horizontal scaling is preferred because: (1) No physical limits - you can keep adding servers; (2) Better fault tolerance - if one server fails, others continue; (3) Geographic distribution - servers can be placed worldwide; (4) More cost-effective at scale. Vertical scaling eventually hits hardware limits and creates a single point of failure.

**Question 7:** Database scaling challenges: (1) Data must be split (partitioning/sharding); (2) Transactions spanning multiple databases become complex; (3) Joins across databases are difficult; (4) Maintaining consistency is harder; (5) Foreign key relationships may span shards. Web servers are typically stateless, making them easier to scale horizontally.

**Question 8:** 
- **Scaling out (horizontal)**: Adding more servers. Appropriate for web applications with growing user base. Example: Adding 10 more web servers to handle Black Friday traffic.
- **Scaling up (vertical)**: Adding more power to existing servers. Appropriate for databases with complex queries that benefit from more RAM. Example: Upgrading database server from 32GB to 128GB RAM.

**Question 9:** Stateless architecture means servers don't store user session data locally. Benefits: (1) Any server can handle any request; (2) Easy to add/remove servers; (3) Simple load balancing. Trade-offs: (1) Session data must be stored elsewhere (cache, database); (2) Additional network calls to fetch session data; (3) Slightly increased latency.

**Question 10:**
- **Milestone 1 (1K-10K users)**: Single server with database on same machine
- **Milestone 2 (10K-100K users)**: Separate database server, add caching layer, use CDN for static assets
- **Milestone 3 (100K-500K users)**: Multiple web servers behind load balancer, read replicas for database
- **Milestone 4 (500K-1M users)**: Database sharding, message queues for async processing, microservices for key features

---

## Module 4: Load Balancing

### Multiple Choice Answers
1. **B** - To distribute incoming traffic across multiple servers
2. **C** - Least Connections
3. **B** - Session persistence (sticky sessions)
4. **B** - Transport Layer (TCP/UDP)
5. **B** - L7 (Layer 7) load balancer

### Short Answer Sample Answers

**Question 6:** 
- **L4 Load Balancer**: Operates at transport layer (TCP/UDP). Routes based on IP address and port. Faster, simpler, but less flexible. Use when: High throughput is critical, simple routing is sufficient.
- **L7 Load Balancer**: Operates at application layer (HTTP). Routes based on content (URL, headers, cookies). Slower but more flexible. Use when: Need content-based routing, SSL termination, or complex routing logic.

**Question 7:** Health checks periodically ping servers to verify they're responsive. If a server fails health checks, the load balancer stops sending traffic to it until it recovers. This prevents users from being routed to broken servers, improving overall reliability.

**Question 8:** Sticky sessions are necessary when application state is stored locally on servers (e.g., shopping cart in memory). Implementation methods: (1) Cookie-based - load balancer sets a cookie identifying the server; (2) IP hash - same client IP always routes to same server; (3) Application-level session tokens.

**Question 9:** 
- **Round Robin**: Simple, even distribution. Good for servers with equal capacity. May not account for different request complexities or server loads.
- **Least Connections**: Routes to server with fewest active connections. Better for long-lived connections or requests with varying processing times. Requires tracking connection counts.

**Question 10:** Load balancers can be a single point of failure. Solutions: (1) Active-passive failover - backup load balancer takes over if primary fails; (2) Active-active - multiple load balancers share traffic using DNS round-robin or anycast IP; (3) Health monitoring and automatic failover; (4) Geographic distribution with multiple load balancers.

---

## Module 5: Caching

### Multiple Choice Answers
1. **C** - Faster data retrieval and reduced latency
2. **B** - Write-through
3. **B** - When requested data is not found in the cache
4. **B** - LRU (Least Recently Used)
5. **B** - Redis

### Short Answer Sample Answers

**Question 6:** 
**Cache-aside (Lazy Loading)**: Application checks cache first. On miss, loads from database and updates cache.
- **Advantages**: Only caches data actually requested; cache failures don't bring down system.
- **Disadvantages**: First request is slow (cache miss); potential for stale data; cache and database can become inconsistent.

**Question 7:** Thundering herd occurs when a popular cached item expires and many requests simultaneously try to regenerate it, overwhelming the database. Mitigations: (1) Use locks/semaphores so only one request regenerates; (2) Probabilistic early expiration (refresh before expiration); (3) Set cache items to never expire, update proactively; (4) Use request coalescing.

**Question 8:**
- **Write-through**: Writes to cache and database simultaneously. Pro: Cache and database always consistent. Con: Slower writes, every write hits both systems.
- **Write-back**: Writes to cache only, asynchronously writes to database. Pro: Faster writes. Con: Risk of data loss if cache fails before database write.

Use write-through for critical data (financial transactions), write-back for non-critical data where speed matters (user activity logs).

**Question 9:** Cache invalidation is hard because: (1) Difficult to know when data changes; (2) Multiple caches may hold same data; (3) Timing issues - invalidation might arrive before update; (4) Partial updates can leave inconsistent state.

Example: E-commerce product price updates. If price changes in database but cache isn't invalidated, users see old prices. If cache is invalidated too early, system floods database with requests.

**Question 10:**
Strategy: 
- **Cache layer**: Redis with short TTL (5-10 minutes)
- **Pattern**: Write-through for price updates (critical accuracy)
- **TTL**: Short expiration to balance freshness and performance
- **Invalidation**: Event-driven - when seller updates product, invalidate that product's cache entry
- **Fallback**: On cache miss, load from database and update cache
- **Monitoring**: Track cache hit rate; if too low, adjust TTL or cache size

---

## Module 6: Database Design

### Multiple Choice Answers
1. **B** - NoSQL Document database
2. **C** - ACID transactions and strong consistency
3. **C** - NoSQL Graph database
4. **A** - Atomicity, Consistency, Isolation, Durability
5. **C** - Wide-column database

### Short Answer Sample Answers

**Question 6:** 
- **Normalization**: Organizing data to reduce redundancy by splitting into multiple related tables. Saves space, maintains consistency, but requires joins.
- **Denormalization**: Deliberately adding redundancy by duplicating data across tables. Faster reads (no joins), but more storage and potential inconsistency.

Use denormalization when: Read performance is critical, data doesn't change often, storage is cheap, and the complexity of joins hurts performance.

**Question 7:**
- **SQL**: Strong consistency, ACID transactions, complex queries, structured data, well-defined schema. Best for: Financial systems, inventory management, complex reporting.
- **NoSQL**: Horizontal scalability, flexible schema, eventual consistency, simple queries. Best for: Social media, IoT data, real-time analytics, rapidly evolving data models.

Trade-offs: SQL offers data integrity but harder to scale horizontally. NoSQL scales easily but sacrifices some consistency guarantees.

**Question 8:** Indexes speed up data retrieval by creating a data structure (B-tree, hash table) that allows quick lookups. Costs: (1) Extra storage for index structure; (2) Slower writes (must update index); (3) Memory overhead. Use indexes on frequently queried columns but avoid over-indexing.

**Question 9:** BASE stands for Basically Available, Soft state, Eventual consistency. It contrasts with ACID:
- **ACID**: Strong consistency, immediate accuracy, may sacrifice availability.
- **BASE**: High availability, accepts temporary inconsistency, eventually becomes consistent.

BASE is used in distributed NoSQL systems (Cassandra, DynamoDB) where availability and partition tolerance are prioritized over immediate consistency.

**Question 10:** Use a **hybrid approach**:
- **SQL for**: User profiles, authentication data (structured, ACID needed)
- **NoSQL (Document DB) for**: Posts and user feeds (flexible schema, high read/write volume, eventual consistency acceptable)
- **NoSQL (Graph DB) for**: Social relationships/follows (optimized for relationship queries)

Justification: Social platforms need horizontal scalability for posts/feeds (billions of items). User data benefits from SQL's consistency. Relationships need graph database's efficient traversal. This hybrid approach balances scalability, consistency, and query efficiency.

---

## Module 7: Data Partitioning

### Multiple Choice Answers
1. **B** - To distribute data across multiple databases for scalability
2. **B** - Range-based partitioning
3. **B** - Difficulty with range queries
4. **B** - Minimizing data movement when adding/removing nodes
5. **B** - Joins across shards

### Short Answer Sample Answers

**Question 6:** A hot shard (hotspot) occurs when one partition receives disproportionately more traffic than others. Causes: Poor partition key choice (e.g., date in time-series data), celebrity users, geographic concentration.

Prevention: (1) Choose partition keys with even distribution; (2) Use composite keys; (3) Add randomization/salt to keys; (4) Split hot shards into smaller partitions; (5) Cache heavily accessed data.

**Question 7:**
- **Hash-based**: Applies hash function to partition key. Pros: Even distribution. Cons: Range queries are inefficient. Use case: User IDs where range queries aren't needed.
- **Range-based**: Divides data by key ranges. Pros: Efficient range queries. Cons: Risk of hot shards if data isn't evenly distributed. Use case: Time-series data where queries often filter by date ranges.

**Question 8:** Consistent hashing distributes data across nodes such that adding/removing nodes only affects adjacent nodes, not the entire cluster. Uses a ring structure where both data keys and nodes are hashed to positions.

Importance: Traditional hashing (key % N) causes massive data movement when N changes. Consistent hashing minimizes reorganization, making dynamic scaling practical for caches like Memcached.

**Question 9:** Resharding challenges: (1) Moving massive amounts of data takes time; (2) Must maintain service during migration; (3) Risk of data loss or corruption; (4) Complex coordination.

Strategies: (1) Double-write to old and new shards during migration; (2) Migrate in phases (one shard at a time); (3) Use consistent hashing to minimize movement; (4) Schedule during low-traffic periods; (5) Have rollback plan.

**Question 10:** Choose **user_id** as partition key because:
- Even distribution (assuming user IDs are sequential or hashed)
- Most queries filter by user (feeds, profiles, posts)
- Avoids geographic hotspots
- Simple to implement
- Scales linearly with user growth

Alternative: Use hash(user_id) to ensure even distribution across shards. Monitor for celebrity users and consider sub-partitioning their data. Plan for resharding when individual shards exceed capacity thresholds.

---

## Module 8: Replication

### Multiple Choice Answers
1. **B** - To improve data availability and reliability
2. **B** - Only to the master
3. **B** - Lower latency for writes in multiple regions
4. **B** - The delay between a write on the master and its appearance on replicas
5. **B** - Synchronous replication

### Short Answer Sample Answers

**Question 6:**
- **Synchronous**: Master waits for replica acknowledgment before confirming write. Pros: No data loss, strong consistency. Cons: Higher latency, reduced availability if replica is down.
- **Asynchronous**: Master confirms write immediately, replicates in background. Pros: Low latency, high availability. Cons: Potential data loss if master fails before replication.

Choose synchronous for critical data (financial transactions). Choose asynchronous for high-throughput systems where slight inconsistency is acceptable.

**Question 7:** Split-brain occurs when network partition causes multiple nodes to believe they're the master, potentially accepting conflicting writes and causing data divergence.

Prevention: (1) Use quorum-based consensus (require majority vote); (2) Implement fencing tokens to invalidate old masters; (3) Use external coordination service (ZooKeeper, etcd); (4) Employ STONITH (Shoot The Other Node In The Head) in clusters.

**Question 8:** Read replicas handle read traffic, reducing load on the primary database. Benefits: (1) Better read performance through load distribution; (2) Geographic placement for lower latency; (3) Isolate analytics queries from transactional workload.

Consistency issues: Replication lag means replicas may have stale data. Users might read old data after writing. Solutions: (1) Read-your-writes consistency by routing user's reads to master; (2) Show timestamps to indicate data freshness; (3) Use caching to mask inconsistency.

**Question 9:** Write conflict resolution strategies:
1. **Last-write-wins (LWW)**: Use timestamp; newer write wins. Simple but may lose data.
2. **Version vectors**: Track causality; merge non-conflicting changes, flag conflicts for manual resolution.
3. **Application-level resolution**: Define business logic (e.g., for counters, add values).
4. **Conflict-free replicated data types (CRDTs)**: Use data structures that merge automatically.

Example: Shopping cart - use CRDT set that unions items from both replicas (add all items from both carts).

**Question 10:**
Strategy:
- **Master-master replication** in each region (NA, EU, Asia)
- **Asynchronous replication** between regions (for speed)
- **Writes**: Directed to local region's master (low latency)
- **Reads**: From local replicas (fast)
- **Conflict resolution**: Last-write-wins with timestamps
- **Critical data**: Route to single master for strong consistency (payments)
- **Product catalog**: Replicate globally (read-heavy, eventual consistency OK)
- **User sessions**: Store in local region, don't replicate

This balances performance, consistency, and user experience across regions.

---

## Module 9: Consistency Patterns

### Multiple Choice Answers
1. **B** - Strong consistency
2. **B** - All replicas will eventually converge to the same value
3. **C** - Causal consistency
4. **B** - Increased latency and reduced availability
5. **B** - Social media post likes count

### Short Answer Sample Answers

**Question 6:**
- **Strong consistency**: All clients see the same data at the same time. After a write, all reads immediately reflect that write. Example: Bank account balances (critical accuracy).
- **Eventual consistency**: Replicas may temporarily differ but will eventually synchronize. Example: Social media likes (temporary inconsistency is acceptable).

Strong consistency requires coordination (locks, consensus), increasing latency. Eventual consistency allows high availability and performance but risks showing stale data.

**Question 7:** Read-your-writes consistency guarantees that after you write data, your subsequent reads will reflect your write (even if other users might not see it yet).

Importance: Improves user experience by preventing confusion. Example: User posts a comment and immediately sees it in the thread, even though other users in different regions might see it seconds later. Without this, users might think their action failed.

**Question 8:** Causal consistency preserves cause-effect relationships. If operation A causally precedes B, all nodes see A before B.

Difference from eventual consistency: Eventual consistency provides no ordering guarantees. Causal consistency maintains logical order.

Example need: Chat/messaging applications where message order matters. If Alice sends "Hello" then "How are you?", all users must see messages in that order. Causal consistency ensures this while allowing eventual delivery.

**Question 9:** CRDTs (Conflict-free Replicated Data Types) are data structures designed to be replicated across nodes and merged automatically without conflicts.

Examples: Counters (add operations commute), Sets (union on merge), Registers (last-write-wins).

How they help: CRDTs allow concurrent updates without coordination. When replicas sync, CRDTs merge deterministically, achieving eventual consistency without conflict resolution logic. Used in collaborative editing, distributed databases, mobile apps with offline support.

**Question 10:** Choose **weak consistency with local updates**.

Reasoning: 
- Strong consistency would introduce unacceptable latency (100-200ms+ for global coordination)
- Players need immediate feedback (local updates feel instant)
- Use client-side prediction: Player sees their movement immediately
- Server periodically syncs authoritative state
- For critical actions (firing weapons), use server validation
- Trade-off: Players might see opponents in slightly different positions (acceptable in fast-paced games)
- Use lag compensation and interpolation to smooth inconsistencies

This approach prioritizes responsiveness (essential for gameplay) while maintaining fairness through server authority on critical events.

---

## Module 10: API Design

### Multiple Choice Answers
1. **B** - GET
2. **B** - GraphQL allows clients to request exactly what they need, REST returns fixed structures
3. **B** - 201 Created
4. **B** - Breaking existing clients when making API changes
5. **B** - Rate limiting

### Short Answer Sample Answers

**Question 6:** RESTful API design principles:
1. **Stateless**: Each request contains all needed information; server doesn't store session state
2. **Resource-based**: URLs represent resources (nouns), not actions
3. **HTTP methods**: Use GET (read), POST (create), PUT/PATCH (update), DELETE properly
4. **Uniform interface**: Consistent naming, structure, and response formats
5. **HATEOAS**: Responses include links to related resources

**Question 7:**
- **PUT**: Replaces entire resource. Send complete object. Idempotent (multiple identical requests have same effect).
- **PATCH**: Partially updates resource. Send only changed fields. May or may not be idempotent.

Use PUT when: Updating entire resource, have complete data. Use PATCH when: Updating specific fields, want to save bandwidth, resource is large.

Example: PUT /users/123 with full user object vs PATCH /users/123 with {"email": "new@email.com"}

**Question 8:** N+1 problem occurs when fetching a list requires one query, then N additional queries for related data.

Example: GET /posts returns 10 posts. Then for each post, need separate GET /posts/{id}/author (10 more queries = 11 total).

GraphQL solution: Single query specifies exactly what's needed:
```
query {
  posts {
    title
    author { name }
  }
}
```
Backend can optimize with single database join, preventing N+1 problem.

**Question 9:**
- **API Keys**: Simple string tokens. Pros: Easy to implement, good for public APIs. Cons: Less secure, no user context, hard to revoke selectively.
- **OAuth**: Token-based authorization with scopes. Pros: More secure, supports user delegation, granular permissions. Cons: More complex to implement, requires token refresh logic.

Use API keys for: Server-to-server, internal APIs, simple authentication. Use OAuth for: User-facing apps, third-party integrations, when need to act on user's behalf.

**Question 10:**
```
# Posts
POST   /posts              Create post       201 Created
GET    /posts              List posts        200 OK
GET    /posts/{id}         Get post          200 OK / 404 Not Found
PUT    /posts/{id}         Update post       200 OK / 404 Not Found
DELETE /posts/{id}         Delete post       204 No Content / 404 Not Found

# Comments
POST   /posts/{id}/comments         Create comment    201 Created
GET    /posts/{id}/comments         List comments     200 OK
GET    /posts/{id}/comments/{cid}   Get comment       200 OK / 404 Not Found
PUT    /posts/{id}/comments/{cid}   Update comment    200 OK / 404 Not Found
DELETE /posts/{id}/comments/{cid}   Delete comment    204 No Content / 404 Not Found
```

All endpoints return 401 Unauthorized if not authenticated, 403 Forbidden if not authorized.

---

## Module 11: Message Queues

### Multiple Choice Answers
1. **B** - Decoupling components and enabling asynchronous processing
2. **B** - Messages remain in the queue until the consumer recovers
3. **C** - At-least-once
4. **B** - Apache Kafka
5. **B** - A queue for storing messages that failed processing after multiple attempts

### Short Answer Sample Answers

**Question 6:** 
- **Message Queue**: Simple component that stores and forwards messages between producer and consumer. Basic FIFO functionality.
- **Message Broker**: Advanced system with additional features: routing (topics, exchanges), message transformation, protocol translation, persistence, delivery guarantees, monitoring.

Examples: RabbitMQ (broker) adds routing rules, message persistence, and acknowledgment patterns beyond simple queuing.

**Question 7:**
- **Pull (Queue)**: Consumer actively requests messages. Pros: Consumer controls rate, no overload. Cons: Polling overhead, potential delays.
- **Push (Pub/Sub)**: Broker pushes messages to subscribers. Pros: Immediate delivery, lower latency. Cons: Can overload slow consumers, requires backpressure handling.

Use pull for: Batch processing, varying consumer speeds, consumer-controlled throughput. Use push for: Real-time notifications, event streaming, multiple subscribers.

**Question 8:** Poison message is a message that repeatedly fails processing (due to malformed data, bugs, or incompatibility), blocking the queue.

Handling strategies:
1. **Max retry limit**: After N failures, move to dead letter queue
2. **Error isolation**: Separate queue for problematic messages
3. **Exponential backoff**: Increase delay between retries
4. **Monitoring**: Alert when same message fails multiple times
5. **Manual intervention**: Human review of dead letter queue
6. **Validation**: Reject invalid messages early

**Question 9:** Message queues improve fault tolerance by:
1. **Persistence**: Messages stored durably; not lost if consumer crashes
2. **Acknowledgment**: Consumer confirms processing; unacknowledged messages are redelivered
3. **Retry logic**: Failed messages automatically retried
4. **Decoupling**: Producer continues even if consumer is down

When node fails mid-operation: Message stays in queue (not acknowledged), another consumer picks it up. Ensures no message is lost, though may be processed twice (at-least-once delivery).

**Question 10:**
```
Order Placement:
  Customer → API → Order Queue

Payment Processing:
  Payment Service ← Order Queue
  Payment Service → Payment Confirmation Queue

Inventory Update:
  Inventory Service ← Payment Confirmation Queue
  Inventory Service → Inventory Updated Queue

Shipping Notification:
  Notification Service ← Inventory Updated Queue
  Notification Service → Email/SMS to customer
```

Benefits:
- **Decoupling**: Services don't need to know about each other
- **Reliability**: If inventory service is down, messages wait in queue
- **Scalability**: Can scale each service independently
- **Asynchronous**: Order placement returns immediately; processing happens in background
- **Fault tolerance**: Failed steps automatically retry

---

## Module 12: Microservices

### Multiple Choice Answers
1. **B** - Applications broken into small, independent services
2. **B** - Independent deployment and scaling of services
3. **C** - APIs (REST, gRPC) or message queues
4. **B** - Automatically locating the network address of service instances
5. **B** - Circuit breaker

### Short Answer Sample Answers

**Question 6:**
- **Monolithic**: Single codebase, shared database, deployed as one unit. Pros: Simple to develop and deploy initially, easier to test. Cons: Difficult to scale, tight coupling, long deployment cycles.
- **Microservices**: Multiple small services, separate databases, independent deployment. Pros: Independent scaling, technology flexibility, fault isolation. Cons: Operational complexity, distributed system challenges, network latency.

Trade-off: Microservices offer flexibility and scalability at the cost of complexity.

**Question 7:** API Gateway is a single entry point for all client requests that routes to appropriate microservices.

Problems solved:
1. **Client simplification**: Clients call one endpoint, not many services
2. **Cross-cutting concerns**: Handles authentication, logging, rate limiting centrally
3. **Protocol translation**: Converts between client protocols (HTTP) and internal protocols (gRPC)
4. **Request aggregation**: Combines multiple service calls into one client response
5. **Load balancing**: Distributes requests across service instances

**Question 8:** Bounded context defines boundaries where a domain model applies. Each context has its own ubiquitous language and models.

Relevance to microservices: Each microservice should align with one bounded context. This ensures:
- Clear service boundaries
- Minimal coupling between services
- Independent evolution of models
- Clear ownership and responsibility

Example: In e-commerce, "Product" in Catalog context (name, description) differs from "Product" in Inventory context (quantity, location).

**Question 9:** Distributed transaction challenges:
- Two-phase commit is slow and blocks resources
- Network failures can leave transactions in unknown state
- Reduces availability
- Tight coupling between services

**Saga pattern**: Breaks transaction into sequence of local transactions. Each step has compensating transaction for rollback.

Example: Order processing saga:
1. Reserve inventory (compensate: release inventory)
2. Charge payment (compensate: refund)
3. Ship order (compensate: cancel shipment)

If step 3 fails, execute compensating transactions in reverse order.

**Question 10:**
E-commerce microservices:
1. **User Service**: Authentication, profiles. Owns user database.
2. **Product Catalog Service**: Product information, search. Owns catalog database.
3. **Inventory Service**: Stock levels, reservations. Owns inventory database.
4. **Order Service**: Order processing, history. Owns orders database.
5. **Payment Service**: Payment processing. Integrates with payment gateways.
6. **Shipping Service**: Fulfillment, tracking. Integrates with carriers.

Interactions:
- User Service authenticates; API Gateway validates tokens
- Product Catalog provides search via REST API
- Order Service calls Inventory (check stock) → Payment (charge) → Shipping (fulfill)
- Use message queue for order events (eventual consistency)
- Each service has separate database (no shared data)

---

## Module 13: Availability Patterns

### Multiple Choice Answers
1. **B** - The system remains operational and accessible even when failures occur
2. **C** - Active-passive failover
3. **B** - Failover switches to backup, failback returns to primary
4. **B** - Mean Time Between Failures
5. **C** - Redundancy

### Short Answer Sample Answers

**Question 6:**
- **Active-Passive**: Primary handles traffic; backup is standby. On failure, backup activates. Pros: Simpler, lower cost (backup not processing). Cons: Wasted resources, slower failover, potential data loss.
- **Active-Active**: Both instances handle traffic simultaneously. Pros: Better resource utilization, instant failover, no data loss. Cons: More complex, higher cost, need data synchronization.

Use active-passive for: Cost-sensitive, infrequent traffic, acceptable brief downtime. Use active-active for: Critical systems, high traffic, zero downtime required.

**Question 7:** Graceful degradation means continuing to operate with reduced functionality when components fail, rather than complete outage.

Example: E-commerce site - if recommendation engine fails, show generic popular products instead. If payment provider is down, queue orders and process when recovered. Users can still browse and add to cart.

Improves experience by: Maintaining core functionality, keeping users engaged, showing helpful messages instead of errors.

**Question 8:**
For components in series, multiply their availability:
- System availability = A × B = 0.999 × 0.995 = 0.994005 = 99.4%

Series components have lower availability than any individual component (weakest link). Each component in the chain reduces overall availability.

**Question 9:** Health check endpoint (e.g., /health) returns server status.

Importance: Enables load balancers and orchestrators to detect failures, route traffic only to healthy instances, and trigger automated recovery.

Should include:
- HTTP 200 for healthy, 503 for unhealthy
- Database connectivity status
- Dependent service status
- Disk space, memory usage
- Response time (under timeout threshold)
- Application-specific checks (cache connectivity, queue accessibility)

**Question 10:**
High availability banking architecture:
1. **Geographic redundancy**: Deploy in multiple data centers (different cities/regions)
2. **Load balancer**: Active-active pair with health checks
3. **Application tier**: Multiple servers in each datacenter behind load balancers
4. **Database**: Master-master replication with synchronous writes, automatic failover
5. **Network**: Redundant network paths, multiple ISPs
6. **Monitoring**: Real-time health checks, automated alerting
7. **Backup**: Real-time backup to separate location, regular restore testing

Failover strategy:
- Database: Use quorum-based consensus (3 or 5 nodes); automatic master election
- Application: Load balancer detects failure via health check; routes traffic to healthy instances
- Datacenter: DNS failover or global load balancer redirects to secondary site
- RTO: < 60 seconds, RPO: 0 (no data loss with synchronous replication)

---

## Module 14: CDN

### Multiple Choice Answers
1. **B** - To deliver content to users from geographically closer servers
2. **B** - Static assets like images, CSS, and JavaScript
3. **B** - A server located close to end users that caches content
4. **B** - Fetches content from the origin server and caches it
5. **B** - Geographic proximity and server load

### Short Answer Sample Answers

**Question 6:**
Performance improvements:
- Reduced latency: Content served from nearby edge server (10-50ms vs 200-500ms)
- Faster load times: Parallel downloads from multiple edge servers
- Reduced origin server load: Majority of requests served from cache

Reliability improvements:
- Geographic redundancy: If one edge server fails, routes to another
- DDoS protection: Distributed architecture absorbs attacks
- Origin shielding: Protects origin from traffic spikes
- Automatic failover: Users transparently served from backup locations

**Question 7:** Cache invalidation is removing or updating outdated content from CDN edge servers.

Challenges:
- Content spread across hundreds of edge servers globally
- Eventual consistency: Invalidation takes time to propagate
- Cost: Frequent invalidation negates caching benefits
- Version conflicts: Different users may see different versions temporarily

Strategies:
1. **TTL-based**: Set appropriate expiration times
2. **Purge/invalidation API**: Manually trigger cache clear
3. **Versioned URLs**: Add version to filename (style.v2.css)
4. **Immutable content**: Never change files; new versions have new names
5. **Soft purge**: Mark stale, serve while revalidating

**Question 8:**
- **Pull CDN**: Edge servers fetch content on-demand when requested (cache-on-miss). Pros: No upfront upload, only popular content cached. Cons: First request slow, origin traffic on misses.
- **Push CDN**: Upload content to CDN proactively. Pros: Guaranteed availability, faster first request. Cons: Storage costs for all content, manual upload process.

Use pull for: User-generated content, large catalogs, dynamic popularity. Use push for: Marketing campaigns, product launches, critical assets.

**Question 9:** CDN reduces bandwidth costs by:
- Serving content from edge servers instead of origin (saves origin bandwidth)
- Compression: CDN compresses content before delivery
- Caching: Repeated requests don't hit origin
- Efficient routing: Optimized network paths

Example: Video streaming platform with 1 million views/day
- Without CDN: 1M × 100MB = 100TB from origin ($5,000-10,000/month)
- With CDN: 95% cache hit rate = 5TB from origin ($250-500/month)
- Savings: ~$4,500-9,500/month

**Question 10:**
Video streaming CDN strategy:

**Content types:**
- Video files (HLS/DASH segments): Cache at edge, long TTL (24 hours)
- Thumbnails: Cache at edge, medium TTL (6 hours)
- Metadata (titles, descriptions): Cache at edge, short TTL (1 hour)
- User data: Don't cache (dynamic, user-specific)

**Cache duration:**
- Popular content: 24-48 hours
- Long-tail content: 6-12 hours with adaptive TTL
- Live streams: 5-30 seconds

**Video transcoding:**
- Origin: Transcode to multiple resolutions (360p, 720p, 1080p, 4K)
- Adaptive bitrate: Store multiple versions; CDN serves based on client bandwidth
- Edge computing: Some CDNs support edge transcoding for less popular content
- Formats: HLS for iOS, DASH for others, fallback to MP4

**Additional considerations:**
- Geographic distribution: More edge servers in high-traffic regions
- Bandwidth detection: Serve appropriate bitrate automatically
- Pre-warming: Push popular content to edge servers proactively
- Analytics: Track which content is popular to optimize caching
