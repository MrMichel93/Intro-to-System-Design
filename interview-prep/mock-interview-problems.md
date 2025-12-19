# Mock Interview Problems

10 progressive practice problems designed to simulate real system design interviews. Each problem includes difficulty level, time allocation, and links to relevant modules.

## How to Use These Problems

**For self-practice:**
1. Set a timer (45-60 minutes)
2. Work through the problem systematically
3. Don't look at hints until you're stuck
4. Compare your solution with linked resources afterward

**For peer practice:**
1. Take turns being interviewer/interviewee
2. Interviewer can provide hints if needed
3. Give feedback after each session
4. Focus on communication and thought process

**Progressive approach:**
- Start with Problem 1 and work your way up
- Don't skip levels - build your skills progressively
- Repeat problems after 2-3 weeks to measure improvement

---

## Problem 1: Design a Parking Lot System

**Difficulty:** ⭐ Easy (Beginner)  
**Time:** 45 minutes  
**Level:** Entry-level warm-up

### Problem Statement

Design a parking lot management system that tracks vehicle entry/exit and available spots.

### Core Requirements

**Functional:**
- Park vehicle (assign spot)
- Remove vehicle (free spot)
- Check available spots
- Calculate parking fee
- Support multiple vehicle types (compact, regular, large)

**Non-Functional:**
- 1000 parking spots
- 500 vehicles entering/exiting per day
- Real-time spot availability
- Payment processing

### Key Challenges to Consider

- How do you assign spots efficiently?
- How do you track which spots are occupied?
- How do you calculate fees (hourly rate)?
- How do you handle multiple entry/exit gates?

### Interviewer Follow-ups

1. "How would you handle different pricing for vehicle types?"
2. "What if we need to reserve spots in advance?"
3. "How would you handle payment failures?"

### Expected Components

```
Entry/Exit Gates → Parking Management Service → Database
                          ↓
                   Spot Allocation
                   Fee Calculator
                   Payment Service
```

### Key Topics to Cover

- ✅ Spot assignment algorithm (nearest, first-available)
- ✅ State management (available, occupied, reserved)
- ✅ Fee calculation (time-based pricing)
- ✅ Database design (spots, vehicles, transactions)
- ✅ Concurrency (multiple gates accessing same spot)

### Related Modules

- [Database Design](../06-database-design/)
- [System Design Foundations](../00-foundations/)

### Success Criteria

- Clear database schema for spots and transactions
- Algorithm for spot assignment
- Fee calculation logic
- Handling concurrent requests

---

## Problem 2: Design a Library Management System

**Difficulty:** ⭐ Easy  
**Time:** 45 minutes  
**Level:** Beginner

### Problem Statement

Design a system for managing a library's book inventory and member borrowing.

### Core Requirements

**Functional:**
- Add/remove books from catalog
- Members can borrow books
- Members can return books
- Track due dates and overdue fines
- Search books by title, author, ISBN
- Reserve books

**Non-Functional:**
- 10,000 books
- 5,000 members
- 1,000 transactions per day
- Search results in < 1 second

### Key Challenges to Consider

- How do you prevent borrowing unavailable books?
- How do you calculate overdue fines?
- How do you handle book reservations?
- What happens when multiple people want the same book?

### Interviewer Follow-ups

1. "How would you implement a waiting list for popular books?"
2. "How do you notify users when reserved books become available?"
3. "What if we want to track book condition and damages?"

### Expected Components

```
Users → API Gateway → [Book Service] [Member Service] [Transaction Service]
                            ↓              ↓              ↓
                        [Book DB]    [Member DB]   [Transaction DB]
                            ↓
                      [Search Index]
```

### Key Topics to Cover

- ✅ Database schema (books, members, transactions)
- ✅ Book availability tracking
- ✅ Fine calculation algorithm
- ✅ Search functionality (Elasticsearch or SQL queries)
- ✅ Reservation system (queue management)
- ✅ Notification system (email/SMS)

### Related Modules

- [Database Design](../06-database-design/)
- [API Design](../10-api-design/)
- [Message Queues](../11-message-queues/)

### Success Criteria

- Complete database schema
- Availability checking logic
- Fine calculation approach
- Basic search implementation

---

## Problem 3: Design a Food Delivery App (Basic)

**Difficulty:** ⭐⭐ Easy-Medium  
**Time:** 45 minutes  
**Level:** Intermediate

### Problem Statement

Design a food delivery platform like DoorDash or Uber Eats (simplified version).

### Core Requirements

**Functional:**
- Browse restaurants and menus
- Place orders
- Track order status (received, preparing, out for delivery, delivered)
- Match orders with delivery drivers
- Payment processing

**Non-Functional:**
- 100,000 daily active users
- 10,000 orders per day
- Real-time order tracking
- 99% availability

### Key Challenges to Consider

- How do you match orders with available drivers?
- How do you handle real-time location updates?
- What if a driver cancels?
- How do you optimize delivery routes?

### Interviewer Follow-ups

1. "How would you implement surge pricing during peak hours?"
2. "How do you handle order modifications after placing?"
3. "What if the restaurant is out of an item?"
4. "How would you scale to 1M orders per day?"

### Expected Components

```
Customers/Drivers → Load Balancer → API Gateway
                                          ↓
        [Restaurant Service] [Order Service] [Driver Service] [Payment Service]
                 ↓                 ↓               ↓                ↓
           [Restaurant DB]    [Order DB]   [Location Service] [Payment Gateway]
```

### Key Topics to Cover

- ✅ Order state machine (status transitions)
- ✅ Driver matching algorithm (nearby, available)
- ✅ Real-time location tracking (WebSocket or polling)
- ✅ Payment flow (authorization, capture, refunds)
- ✅ Restaurant menu management
- ✅ Notification system (order updates)
- ✅ Search and filtering restaurants

### Related Modules

- [Database Design](../06-database-design/)
- [Message Queues](../11-message-queues/)
- [API Design](../10-api-design/)
- [Case Study: Ride Sharing](../case-studies/ride-sharing.md) (similar location matching concepts)

### Success Criteria

- Order lifecycle clearly defined
- Driver matching strategy
- Real-time communication approach
- Payment integration plan

---

## Problem 4: Design a Polling/Voting System

**Difficulty:** ⭐⭐ Medium  
**Time:** 50 minutes  
**Level:** Intermediate

### Problem Statement

Design a system like StrawPoll where users can create polls, share them, and collect votes.

### Core Requirements

**Functional:**
- Create polls with multiple options
- Vote on polls
- View real-time results
- Share polls via URL
- Prevent duplicate voting (per IP or user)
- Anonymous and authenticated voting modes

**Non-Functional:**
- Handle 1M votes per day
- Real-time result updates
- Highly available (99.9%)
- Low latency (< 100ms for voting)

### Key Challenges to Consider

- How do you prevent duplicate voting?
- How do you handle high concurrent votes?
- How do you update results in real-time?
- How do you scale vote counting?

### Interviewer Follow-ups

1. "What if someone tries to manipulate votes with bots?"
2. "How would you implement a 'close poll' feature?"
3. "How do you handle vote counts for viral polls (millions of votes)?"
4. "What about weighted voting or ranked-choice voting?"

### Expected Components

```
Users → Load Balancer → API Servers → Cache (Results)
                            ↓              ↓
                       [Poll Service]    Redis
                            ↓
                       [Vote DB]
                            ↓
                  [Analytics Service]
```

### Key Topics to Cover

- ✅ Duplicate vote prevention (cookies, IP tracking, user accounts)
- ✅ Vote counting (increment counters, eventual consistency)
- ✅ Real-time updates (WebSocket, polling, Server-Sent Events)
- ✅ Caching strategy (cache results, invalidate on vote)
- ✅ Database choice (SQL vs NoSQL for vote storage)
- ✅ Bot prevention (rate limiting, CAPTCHA)
- ✅ Scaling vote counting (sharded counters)

### Related Modules

- [Caching](../05-caching/)
- [Database Design](../06-database-design/)
- [Scalability](../03-scalability/)
- [API Design](../10-api-design/)

### Success Criteria

- Duplicate voting prevention strategy
- Vote counting mechanism
- Real-time update approach
- Anti-fraud measures

---

## Problem 5: Design a News Feed Aggregator

**Difficulty:** ⭐⭐⭐ Medium  
**Time:** 50 minutes  
**Level:** Intermediate-Advanced

### Problem Statement

Design a system like Feedly that aggregates content from multiple RSS/Atom feeds and presents a personalized feed to users.

### Core Requirements

**Functional:**
- Users can subscribe to RSS/Atom feeds
- Fetch and parse feeds periodically
- Display aggregated feed to users
- Mark articles as read/unread
- Save articles for later
- Search articles

**Non-Functional:**
- 1M users
- 100K RSS feeds
- 1M new articles per day
- Feeds updated every 5-60 minutes
- Low latency feed display (< 200ms)

### Key Challenges to Consider

- How do you efficiently fetch 100K feeds?
- How do you parse different feed formats?
- How do you detect duplicate articles?
- How do you personalize feed ordering?
- How do you handle feed errors?

### Interviewer Follow-ups

1. "How would you prioritize which feeds to fetch first?"
2. "What if a feed becomes unavailable?"
3. "How would you implement full-text search?"
4. "How do you handle feeds with millions of subscribers?"

### Expected Components

```
Users → API Gateway → [Feed Service] [User Service]
                           ↓              ↓
                    [Crawler Service] [User DB]
                           ↓
                    [Feed DB, Article DB]
                           ↓
                    [Search Index]
```

### Key Topics to Cover

- ✅ Feed crawler (distributed, scheduled jobs)
- ✅ Parsing strategy (different formats: RSS, Atom, JSON Feed)
- ✅ Duplicate detection (content hashing)
- ✅ Storage strategy (articles, feed metadata)
- ✅ Feed generation (user subscriptions → personalized feed)
- ✅ Ranking/sorting (chronological, popularity, personalized)
- ✅ Caching (frequently accessed articles)
- ✅ Search (Elasticsearch for full-text)
- ✅ Error handling (retry logic, dead feeds)

### Related Modules

- [Message Queues](../11-message-queues/)
- [Caching](../05-caching/)
- [Data Partitioning](../07-data-partitioning/)
- [Database Design](../06-database-design/)

### Success Criteria

- Crawler architecture and scheduling
- Feed parsing approach
- Duplicate detection mechanism
- Personalized feed generation
- Search implementation

---

## Problem 6: Design a Collaborative Code Editor

**Difficulty:** ⭐⭐⭐⭐ Medium-Hard  
**Time:** 60 minutes  
**Level:** Advanced

### Problem Statement

Design a real-time collaborative code editor like Google Docs but for code, similar to CodePen or Repl.it.

### Core Requirements

**Functional:**
- Multiple users edit same document simultaneously
- See other users' cursors and selections
- Real-time synchronization
- Version history
- Syntax highlighting (client-side)
- Save/load documents
- Share documents via URL

**Non-Functional:**
- Sub-100ms synchronization latency
- Handle conflicts from concurrent edits
- 99.9% availability
- Support 10 users per document (up to 100 for scale question)

### Key Challenges to Consider

- How do you handle concurrent edits to the same line?
- How do you synchronize changes in real-time?
- What conflict resolution strategy do you use?
- How do you maintain document consistency?
- How do you handle network delays?

### Interviewer Follow-ups

1. "What if two users edit the exact same character at the same time?"
2. "How would you implement undo/redo?"
3. "What happens if a user goes offline and comes back?"
4. "How would you scale this to 100 concurrent users on one document?"

### Expected Components

```
Clients (WebSocket) ↔ Collaboration Server → Operational Transform
                              ↓
                      [Document Service]
                              ↓
                      [Document DB]
                              ↓
                      [Version History]
```

### Key Topics to Cover

- ✅ **Operational Transformation (OT)** or **CRDT** for conflict resolution
- ✅ Real-time communication (WebSocket)
- ✅ Document state management (server-side or distributed)
- ✅ Cursor position synchronization
- ✅ Version control (snapshots, deltas)
- ✅ Offline editing support (queue changes, sync later)
- ✅ Presence system (who's online, active users)
- ✅ Performance optimization (minimize data sent)
- ✅ Consistency guarantees (eventual consistency)

### Related Modules

- [Consistency Patterns](../09-consistency-patterns/)
- [Real-time Systems](../11-message-queues/)
- [Replication](../08-replication/)

### Success Criteria

- Conflict resolution strategy (OT or CRDT)
- Real-time sync mechanism
- Handling edge cases (network delays, offline)
- Version history approach

---

## Problem 7: Design a Distributed Cache

**Difficulty:** ⭐⭐⭐⭐ Medium-Hard  
**Time:** 60 minutes  
**Level:** Advanced

### Problem Statement

Design a distributed caching system like Memcached or Redis Cluster that stores key-value pairs across multiple nodes.

### Core Requirements

**Functional:**
- GET(key) - Retrieve value
- PUT(key, value) - Store value
- DELETE(key) - Remove value
- Support TTL (time-to-live)
- Eviction policy (LRU, LFU)

**Non-Functional:**
- Low latency (< 1ms for most operations)
- High throughput (1M+ ops/sec)
- Horizontal scalability
- High availability (99.99%)
- Data replication

### Key Challenges to Consider

- How do you distribute data across nodes?
- How do you handle node failures?
- How do you add/remove nodes without downtime?
- How do you maintain consistency?
- What eviction policy do you use when memory is full?

### Interviewer Follow-ups

1. "How would you implement consistent hashing?"
2. "What happens if a node fails while serving a request?"
3. "How do you handle hot keys (very popular keys)?"
4. "How would you implement replication?"
5. "How do you handle cache invalidation across multiple nodes?"

### Expected Components

```
Clients → Client Library (Consistent Hashing)
                ↓
          Cache Nodes (Sharded)
                ↓
          Replication (Master-Slave)
                ↓
          Persistence Layer (optional)
```

### Key Topics to Cover

- ✅ **Consistent hashing** for data distribution
- ✅ Replication strategy (master-slave, multi-master)
- ✅ Consistency model (eventual vs strong)
- ✅ Eviction policies (LRU, LFU, TTL)
- ✅ Node failure handling (replica promotion)
- ✅ Adding/removing nodes (consistent hashing helps)
- ✅ Hot key handling (replication, local cache)
- ✅ Persistence (optional: write-ahead log, snapshots)
- ✅ Client-side vs server-side partitioning

### Related Modules

- [Data Partitioning](../07-data-partitioning/)
- [Replication](../08-replication/)
- [Consistency Patterns](../09-consistency-patterns/)
- [Caching](../05-caching/)
- [CAP Theorem](../02-cap-theorem/)

### Success Criteria

- Clear explanation of consistent hashing
- Replication and failover strategy
- Eviction policy choice and rationale
- Handling node additions/removals

---

## Problem 8: Design a Recommendation System

**Difficulty:** ⭐⭐⭐⭐⭐ Hard  
**Time:** 60 minutes  
**Level:** Advanced

### Problem Statement

Design a recommendation system like Netflix's movie recommendations or Amazon's product recommendations.

### Core Requirements

**Functional:**
- Recommend items to users based on their history
- Handle both new users (cold start) and existing users
- Real-time and batch recommendations
- Personalized recommendations
- Similar items recommendations
- Trending items

**Non-Functional:**
- 10M users
- 100K items (movies, products, etc.)
- Generate recommendations in < 100ms
- Update models regularly
- High accuracy

### Key Challenges to Consider

- How do you generate recommendations for new users?
- How do you balance exploration vs exploitation?
- How do you update recommendations in real-time?
- How do you train and deploy ML models?
- How do you measure recommendation quality?

### Interviewer Follow-ups

1. "What algorithms would you use for recommendations?"
2. "How do you handle the cold start problem?"
3. "How do you incorporate both user behavior and item features?"
4. "How would you A/B test different recommendation algorithms?"
5. "How do you avoid filter bubbles (showing only similar content)?"

### Expected Components

```
Users → API Gateway → [Recommendation Service]
                            ↓
                    [Real-time Service] [Batch Service]
                            ↓                 ↓
                    [Feature Store]    [ML Models]
                            ↓                 ↓
                    [User/Item DB]   [Model Training Pipeline]
```

### Key Topics to Cover

- ✅ **Recommendation algorithms:**
  - Collaborative filtering (user-based, item-based)
  - Content-based filtering
  - Matrix factorization
  - Deep learning models (neural collaborative filtering)
- ✅ **Cold start solutions:**
  - New users: Popular items, demographic-based
  - New items: Content-based features
- ✅ **Real-time vs batch:**
  - Batch: Pre-compute recommendations daily
  - Real-time: Adjust based on current session
- ✅ **Feature engineering:**
  - User features (demographics, history, preferences)
  - Item features (category, tags, description)
  - Context features (time, location, device)
- ✅ **Model training pipeline:**
  - Data collection and labeling
  - Model training (offline)
  - Model evaluation (A/B testing)
  - Model deployment and serving
- ✅ **Scalability:**
  - Store pre-computed recommendations
  - Approximate nearest neighbor for real-time
  - Distributed training

### Related Modules

- [Database Design](../06-database-design/)
- [Message Queues](../11-message-queues/)
- [Caching](../05-caching/)
- [Data Partitioning](../07-data-partitioning/)

### Success Criteria

- Algorithm choice with justification
- Handling cold start problem
- Real-time and batch architecture
- Model training and deployment pipeline
- Scalability considerations

---

## Problem 9: Design a Distributed Task Scheduler

**Difficulty:** ⭐⭐⭐⭐⭐ Hard  
**Time:** 60 minutes  
**Level:** Advanced

### Problem Statement

Design a distributed task scheduling system like Apache Airflow or Kubernetes CronJobs that executes tasks across multiple workers.

### Core Requirements

**Functional:**
- Schedule tasks (one-time, recurring)
- Execute tasks on worker nodes
- Handle task dependencies (Task B runs after Task A)
- Retry failed tasks
- Monitor task status
- Cancel running tasks

**Non-Functional:**
- 100K tasks per day
- 1000 worker nodes
- Task execution within 1 second of scheduled time
- 99.9% reliability
- Handle worker failures

### Key Challenges to Consider

- How do you distribute tasks across workers?
- How do you handle worker failures mid-task?
- How do you manage task dependencies (DAG)?
- How do you ensure tasks run exactly once?
- How do you scale to millions of tasks?

### Interviewer Follow-ups

1. "How do you handle a worker that becomes unresponsive?"
2. "What if a task takes longer than expected?"
3. "How do you implement task priorities?"
4. "How would you visualize task execution (monitoring)?"
5. "How do you handle tasks that need to run in a specific order?"

### Expected Components

```
Users → API Gateway → [Scheduler Service]
                            ↓
                    [Task Queue (Priority Queue)]
                            ↓
                    [Worker Nodes] ← [Coordinator]
                            ↓
                    [Task Status DB]
                            ↓
                    [Monitoring Service]
```

### Key Topics to Cover

- ✅ **Task scheduling:**
  - Cron-like scheduling
  - Priority-based scheduling
  - Dependency resolution (DAG execution)
- ✅ **Task distribution:**
  - Work queue (message queue like RabbitMQ, Kafka)
  - Worker pull vs push model
- ✅ **Worker management:**
  - Health checks
  - Heartbeat mechanism
  - Worker registration and discovery
- ✅ **Fault tolerance:**
  - Task retry with exponential backoff
  - Dead letter queue for failed tasks
  - Worker failure detection and task reassignment
- ✅ **Exactly-once execution:**
  - Idempotency tokens
  - Distributed locking (prevent duplicate execution)
- ✅ **Dependency management:**
  - DAG representation
  - Topological sort for execution order
- ✅ **Monitoring:**
  - Task status tracking
  - Execution metrics
  - Alerting on failures

### Related Modules

- [Message Queues](../11-message-queues/)
- [Consistency Patterns](../09-consistency-patterns/)
- [Availability Patterns](../13-availability-patterns/)
- [Microservices](../12-microservices/)

### Success Criteria

- Task distribution mechanism
- Handling worker failures
- Dependency execution (DAG)
- Fault tolerance strategy
- Exactly-once guarantees

---

## Problem 10: Design a Global Content Delivery Network (CDN)

**Difficulty:** ⭐⭐⭐⭐⭐ Hard  
**Time:** 60 minutes  
**Level:** Expert

### Problem Statement

Design a global content delivery network like Cloudflare or Akamai that caches and serves content from edge locations worldwide.

### Core Requirements

**Functional:**
- Cache static content (images, videos, HTML, CSS, JS)
- Serve content from nearest edge location
- Invalidate/purge cache
- Handle origin server failures
- Support both public and private content
- DDoS protection

**Non-Functional:**
- 100+ edge locations globally
- 10TB/sec aggregate bandwidth
- < 50ms latency for cached content
- 99.99% availability
- Cache hit rate > 90%

### Key Challenges to Consider

- How do you route users to the nearest edge?
- How do you decide what to cache?
- How do you handle cache invalidation globally?
- How do you update content across all edges?
- How do you protect against DDoS attacks?

### Interviewer Follow-ups

1. "How do you handle dynamic content that can't be cached?"
2. "What if an edge location goes down?"
3. "How do you implement cache invalidation across 100 locations?"
4. "How do you handle streaming video (large files)?"
5. "How would you implement request routing based on geography?"

### Expected Components

```
Users → DNS (GeoDNS) → Edge Locations (POP)
                              ↓
                        [Edge Cache]
                              ↓
                     [Regional Servers]
                              ↓
                     [Origin Servers]
                              ↓
                    [Cache Invalidation Service]
```

### Key Topics to Cover

- ✅ **Request routing:**
  - GeoDNS (route to nearest POP)
  - Anycast routing
  - Latency-based routing
- ✅ **Caching strategy:**
  - Cache-Control headers
  - TTL-based expiration
  - LRU eviction
  - Hot object caching
- ✅ **Cache invalidation:**
  - Purge requests from origin
  - Versioned URLs
  - Surrogate keys
  - Propagation to all edges
- ✅ **Origin shield:**
  - Reduce load on origin
  - Regional caching layer
- ✅ **Content delivery:**
  - Range requests for large files
  - Byte-range caching
  - Adaptive bitrate for video
- ✅ **Reliability:**
  - Multiple origin servers
  - Failover to other edges
  - Health checks
- ✅ **Security:**
  - DDoS mitigation (rate limiting, filtering)
  - TLS/SSL termination at edge
  - Token-based authentication for private content

### Related Modules

- [CDN](../14-cdn/)
- [Caching](../05-caching/)
- [Load Balancing](../04-load-balancing/)
- [Availability Patterns](../13-availability-patterns/)
- [Scalability](../03-scalability/)

### Success Criteria

- Request routing mechanism (DNS/Anycast)
- Caching strategy and invalidation
- Handling origin failures
- DDoS protection approach
- Global distribution architecture

---

## Practice Schedule Recommendation

### Week 1-2: Easy Problems
- **Day 1:** Problem 1 (Parking Lot)
- **Day 3:** Problem 2 (Library System)
- **Day 5:** Review both, identify weak areas
- **Day 7:** Re-attempt one problem

### Week 3-4: Easy-Medium Problems
- **Day 1:** Problem 3 (Food Delivery - Basic)
- **Day 3:** Problem 4 (Polling System)
- **Day 5:** Problem 5 (News Aggregator)
- **Day 7:** Mock interview with peer (any Easy-Medium problem)

### Week 5-6: Medium-Hard Problems
- **Day 1:** Problem 6 (Collaborative Editor)
- **Day 3:** Problem 7 (Distributed Cache)
- **Day 5:** Review and re-attempt one
- **Day 7:** Mock interview with peer

### Week 7-8: Hard Problems
- **Day 1:** Problem 8 (Recommendation System)
- **Day 3:** Problem 9 (Task Scheduler)
- **Day 5:** Problem 10 (CDN)
- **Day 7:** Full mock interview with random problem

## Tips for Using These Problems

1. **Don't memorize solutions** - Focus on understanding the approach
2. **Time yourself strictly** - Real interviews have time pressure
3. **Practice drawing** - Diagrams are critical
4. **Think out loud** - Even when practicing alone, narrate your thoughts
5. **Get feedback** - Practice with peers or mentors when possible
6. **Review related modules** - After attempting, study relevant course modules
7. **Track your progress** - Note what went well and what needs improvement

---

Good luck with your practice! Remember, consistency matters more than speed. Work through these problems systematically, and you'll build the skills and confidence needed for real interviews. 🚀
