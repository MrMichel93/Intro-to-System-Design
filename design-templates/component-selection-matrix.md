# Component Selection Matrix

Decision frameworks and comparison matrices for choosing the right technologies for your system design.

## How to Use This Guide

1. **Identify the decision** you need to make (e.g., database choice)
2. **Review the comparison matrix** for that component type
3. **Consider your requirements** (scale, consistency, cost, etc.)
4. **Evaluate trade-offs** for each option
5. **Make an informed decision** based on your specific needs

---

## Database Selection

### SQL vs NoSQL Decision Matrix

| Factor | SQL (Relational) | NoSQL (Document/Key-Value) | NoSQL (Column-Family) | NoSQL (Graph) |
|--------|------------------|----------------------------|----------------------|---------------|
| **Data Structure** | Structured, fixed schema | Flexible, schema-less | Wide columns | Relationships |
| **Relationships** | Complex joins, foreign keys | Limited, denormalized | Limited | Native graph queries |
| **Consistency** | Strong (ACID) | Eventual (BASE) | Tunable | Strong/Eventual |
| **Scalability** | Vertical (harder horizontal) | Horizontal (easy) | Horizontal (easy) | Varies |
| **Query Flexibility** | Very flexible (SQL) | Limited to keys/indexes | Column-based queries | Graph traversals |
| **Best For** | Transactions, complex queries | High write volume, flexibility | Time-series, analytics | Social networks, recommendations |
| **Examples** | PostgreSQL, MySQL | MongoDB, DynamoDB | Cassandra, HBase | Neo4j, Amazon Neptune |
| **When to Choose** | ↓ See decision tree below ↓ |

### Decision Tree: SQL vs NoSQL

```
Start: Do you need ACID transactions?
│
├─ YES → Do you have complex relationships?
│   │
│   ├─ YES → Are they graph-like (social network)?
│   │   ├─ YES → Use Graph DB (Neo4j)
│   │   └─ NO → Use SQL (PostgreSQL, MySQL)
│   │
│   └─ NO → Use SQL (PostgreSQL, MySQL)
│
└─ NO → Do you need massive write scalability?
    │
    ├─ YES → Is data time-series or wide-column?
    │   ├─ YES → Use Column-Family DB (Cassandra)
    │   └─ NO → Is schema flexibility important?
    │       ├─ YES → Use Document DB (MongoDB)
    │       └─ NO → Use Key-Value DB (DynamoDB, Redis)
    │
    └─ NO → Is data structured and relational?
        ├─ YES → Use SQL (with read replicas)
        └─ NO → Use Document DB (MongoDB)
```

### Detailed Comparison

#### PostgreSQL / MySQL (SQL)
**Use when:**
- ✅ Need ACID transactions (e-commerce orders, banking)
- ✅ Complex queries with joins across multiple tables
- ✅ Data has clear relationships
- ✅ Need referential integrity
- ✅ Schema is well-defined and stable

**Avoid when:**
- ❌ Need to scale writes beyond millions/sec
- ❌ Schema changes frequently
- ❌ Data is unstructured or varies by record

**Examples:**
- E-commerce order management
- Banking transactions
- User authentication systems
- Inventory management

---

#### MongoDB (Document NoSQL)
**Use when:**
- ✅ Schema flexibility needed (varying document structures)
- ✅ High write throughput required
- ✅ Need horizontal scaling
- ✅ Data is document-oriented (JSON-like)
- ✅ Eventual consistency acceptable

**Avoid when:**
- ❌ Need complex transactions across documents
- ❌ Require complex joins
- ❌ Strong consistency is critical

**Examples:**
- Content management systems
- User profiles with varying attributes
- Product catalogs with different properties
- Log storage and analysis

---

#### Cassandra (Column-Family NoSQL)
**Use when:**
- ✅ Massive write volumes (millions of writes/sec)
- ✅ Time-series data
- ✅ Need high availability (no single point of failure)
- ✅ Data access patterns are predictable
- ✅ Multi-datacenter replication needed

**Avoid when:**
- ❌ Need complex queries or joins
- ❌ Don't know access patterns in advance
- ❌ Small-scale applications

**Examples:**
- IoT sensor data
- Time-series metrics
- Event logging
- Message history (chat apps)

---

#### DynamoDB (Key-Value NoSQL)
**Use when:**
- ✅ Simple key-based lookups
- ✅ Need predictable performance at any scale
- ✅ Serverless architecture
- ✅ Want managed service (no operations)
- ✅ Single-digit millisecond latency required

**Avoid when:**
- ❌ Need complex queries
- ❌ Want to minimize cloud vendor lock-in
- ❌ Cost-sensitive (can be expensive at scale)

**Examples:**
- Session storage
- Shopping carts
- User preferences
- Gaming leaderboards

---

#### Neo4j (Graph Database)
**Use when:**
- ✅ Data is highly connected (social networks)
- ✅ Need to traverse relationships efficiently
- ✅ Recommendations based on connections
- ✅ Complex relationship queries

**Avoid when:**
- ❌ Data is primarily tabular
- ❌ Simple CRUD operations
- ❌ Don't need relationship traversals

**Examples:**
- Social networks (friends of friends)
- Recommendation engines
- Fraud detection (connected patterns)
- Network topology

---

## Caching Strategy Selection

### Cache Type Comparison

| Cache Type | Use Case | Pros | Cons | Example |
|------------|----------|------|------|---------|
| **In-Memory (Redis)** | Shared cache across servers | Fast, persistent option available | Additional infrastructure | Redis, Memcached |
| **Application Cache** | Single server caching | Simple, no network overhead | Not shared, duplicate data | In-process cache |
| **CDN** | Static content delivery | Global distribution, reduced latency | Not for dynamic content | CloudFront, Cloudflare |
| **Database Query Cache** | Repeated queries | Automatic, built-in | Limited control | MySQL query cache |
| **Browser Cache** | Client-side assets | Reduces server load | User controls invalidation | HTTP cache headers |

### When to Use Each Caching Strategy

#### Redis / Memcached
**Use when:**
- ✅ Multiple servers need shared cache
- ✅ Need sub-millisecond reads
- ✅ Have frequently accessed data (hot data)
- ✅ Read-heavy workloads
- ✅ Need advanced features (pub/sub, sorted sets)

**Cache Patterns:**
- **Cache-Aside:** Application checks cache, loads from DB if miss
- **Write-Through:** Write to cache and DB simultaneously
- **Write-Behind:** Write to cache, async write to DB

**Best for:**
- Session storage
- Database query results
- API responses
- Rate limiting counters

---

#### CDN (Content Delivery Network)
**Use when:**
- ✅ Serving static assets (images, videos, CSS, JS)
- ✅ Global user base
- ✅ Need to reduce origin server load
- ✅ Improve page load times

**Best for:**
- Media files (images, videos)
- Static website content
- Software downloads
- Streaming content

---

## API Protocol Selection

### REST vs GraphQL vs gRPC vs WebSockets

| Factor | REST | GraphQL | gRPC | WebSockets |
|--------|------|---------|------|------------|
| **Communication** | Request/Response | Request/Response | Request/Response or Streaming | Bidirectional |
| **Data Format** | JSON/XML | JSON | Protocol Buffers (binary) | Any (often JSON) |
| **Transport** | HTTP/HTTPS | HTTP/HTTPS | HTTP/2 | TCP/HTTP |
| **Performance** | Good | Good | Excellent | Excellent |
| **Learning Curve** | Low | Medium | Medium-High | Medium |
| **Tooling** | Excellent | Good | Good | Good |
| **Real-time** | No (polling needed) | Subscriptions | Server streaming | Yes (native) |
| **Browser Support** | Universal | Universal | Limited | Universal |
| **Best For** | CRUD, public APIs | Flexible queries, mobile apps | Microservices, low latency | Real-time updates, chat |

### Decision Tree: API Protocol

```
What type of communication do you need?
│
├─ Real-time bidirectional → WebSockets
│   Examples: Chat, live gaming, collaborative editing
│
├─ Request/Response → Continue...
│   │
│   ├─ Internal microservices → gRPC
│   │   (Performance critical, typed contracts)
│   │
│   ├─ Public API or mobile app → GraphQL or REST
│   │   │
│   │   ├─ Need flexible queries → GraphQL
│   │   │   (Mobile apps, varying data needs)
│   │   │
│   │   └─ Standard CRUD → REST
│   │       (Simple, widely understood)
│   │
│   └─ Streaming data from server → gRPC Server Streaming
│       (Logs, metrics, live updates)
```

### Detailed Protocol Comparison

#### REST (Representational State Transfer)
**Use when:**
- ✅ Building public APIs
- ✅ Simple CRUD operations
- ✅ Need wide compatibility
- ✅ Stateless operations
- ✅ Caching is important

**Avoid when:**
- ❌ Need real-time updates
- ❌ Complex nested data fetching
- ❌ Performance is critical (high throughput)

**Examples:**
- Public APIs (Twitter, GitHub)
- Traditional web applications
- Mobile app backends (simple)

---

#### GraphQL
**Use when:**
- ✅ Clients need flexible data queries
- ✅ Mobile apps (reduce over-fetching)
- ✅ Complex, nested data relationships
- ✅ Multiple client types with different needs
- ✅ Need real-time subscriptions

**Avoid when:**
- ❌ Simple CRUD is sufficient
- ❌ File uploads/downloads
- ❌ Team lacks GraphQL experience

**Examples:**
- Mobile applications
- Single-page applications (SPAs)
- Dashboards with complex data needs

---

#### gRPC
**Use when:**
- ✅ Internal microservices communication
- ✅ Need low latency / high throughput
- ✅ Strongly typed contracts important
- ✅ Streaming data (server or bidirectional)
- ✅ Polyglot environment (multiple languages)

**Avoid when:**
- ❌ Browser-based clients (limited support)
- ❌ Need human-readable messages
- ❌ Team unfamiliar with Protocol Buffers

**Examples:**
- Microservices architecture
- Real-time data pipelines
- Internal APIs
- Mobile apps (with library support)

---

#### WebSockets
**Use when:**
- ✅ Need real-time bidirectional communication
- ✅ Server needs to push updates to client
- ✅ Low latency is critical
- ✅ Persistent connection acceptable

**Avoid when:**
- ❌ Simple request/response sufficient
- ❌ Connection overhead is concern
- ❌ Need caching (REST is better)

**Examples:**
- Chat applications
- Live notifications
- Collaborative editing (Google Docs)
- Real-time gaming
- Stock tickers
- Live sports scores

---

## Load Balancer Selection

### Load Balancer Types

| Type | Layer | Pros | Cons | Use Case |
|------|-------|------|------|----------|
| **Hardware LB** | L4/L7 | High performance, dedicated | Expensive, not flexible | Enterprise, high traffic |
| **Software LB (Nginx)** | L7 | Flexible, cost-effective | Requires setup/maintenance | Most applications |
| **Cloud LB (ALB/ELB)** | L4/L7 | Managed, auto-scaling | Vendor lock-in, cost | Cloud-native apps |
| **DNS Load Balancing** | L3 | Simple, global distribution | No health checks, cache TTL issues | Multi-region routing |
| **Service Mesh (Envoy)** | L7 | Advanced features, observability | Complex setup | Microservices |

### Load Balancing Algorithms

| Algorithm | How It Works | Best For | Avoid When |
|-----------|--------------|----------|------------|
| **Round Robin** | Requests distributed sequentially | Equal servers, stateless apps | Servers have different capacities |
| **Least Connections** | Send to server with fewest connections | Long-lived connections | Short requests |
| **IP Hash** | Same client → same server (by IP) | Session affinity needed | Client IPs change (mobile) |
| **Weighted Round Robin** | Distribute based on server capacity | Heterogeneous servers | All servers are identical |
| **Least Response Time** | Send to fastest responding server | Variable server performance | Need simplicity |

**Decision guide:**
- **Stateless app, equal servers** → Round Robin
- **Need session stickiness** → IP Hash or Cookie-based
- **Servers have different specs** → Weighted Round Robin
- **Long-lived connections (WebSockets)** → Least Connections

---

## Message Queue Selection

### Queue Comparison

| Feature | RabbitMQ | Apache Kafka | Amazon SQS | Redis Streams |
|---------|----------|--------------|------------|---------------|
| **Message Ordering** | Per queue | Per partition | FIFO queues | Per stream |
| **Throughput** | Medium (50K msg/sec) | Very High (millions/sec) | High (scalable) | High |
| **Persistence** | Yes | Yes (configurable) | Yes | Optional |
| **Delivery Guarantee** | At-least-once | At-least-once, Exactly-once | At-least-once | At-least-once |
| **Retention** | Until consumed | Time-based (days/weeks) | 14 days max | Time/size-based |
| **Consumers** | Multiple (competing) | Consumer groups | Multiple (competing) | Consumer groups |
| **Use Case** | Task queues, work distribution | Event streaming, logs | Decoupling, serverless | Real-time streams |
| **Complexity** | Medium | High | Low (managed) | Low |

### When to Use Each

#### RabbitMQ
**Use when:**
- ✅ Need complex routing (topic, fanout, headers)
- ✅ Task queue with work distribution
- ✅ Priority queues needed
- ✅ Traditional pub/sub patterns

**Best for:**
- Background job processing
- Email notifications
- Task distribution

---

#### Apache Kafka
**Use when:**
- ✅ High throughput event streaming
- ✅ Need message replay
- ✅ Event sourcing architecture
- ✅ Real-time data pipelines
- ✅ Multiple consumers need same data

**Best for:**
- Activity tracking
- Log aggregation
- Stream processing
- Event sourcing

---

#### Amazon SQS
**Use when:**
- ✅ Want fully managed service
- ✅ Serverless architecture (Lambda)
- ✅ Don't want operational overhead
- ✅ Simple queue semantics

**Best for:**
- AWS-native applications
- Decoupling microservices
- Buffering requests

---

#### Redis Streams
**Use when:**
- ✅ Already using Redis
- ✅ Need simple streaming
- ✅ Low latency important
- ✅ Moderate message volumes

**Best for:**
- Real-time notifications
- Activity feeds
- Lightweight event streaming

---

## Storage Selection

### Object Storage vs Block Storage vs File Storage

| Type | Use Case | Performance | Cost | Best For |
|------|----------|-------------|------|----------|
| **Object Storage (S3)** | Unstructured data, backups | Moderate | Low | Images, videos, backups, static assets |
| **Block Storage (EBS)** | Database volumes, boot disks | High | Medium | Databases, VM disks |
| **File Storage (EFS/NFS)** | Shared files across servers | Moderate | Medium-High | Shared application data, home directories |

### Decision Matrix

```
What are you storing?
│
├─ Structured data (tables) → Database (see database section)
│
├─ Files/Media → Continue...
│   │
│   ├─ Shared across servers? 
│   │   ├─ YES → File Storage (EFS, NFS)
│   │   └─ NO → Object Storage (S3, GCS)
│   │
│   ├─ Need high IOPS?
│   │   ├─ YES → Block Storage (EBS, SAN)
│   │   └─ NO → Object Storage
│   │
│   └─ Frequent access or archival?
│       ├─ Frequent → Object Storage (Standard)
│       └─ Archival → Object Storage (Glacier, Archive tier)
```

---

## Search Technology Selection

### Search Engine Comparison

| Technology | Best For | Pros | Cons |
|------------|----------|------|------|
| **Elasticsearch** | Full-text search, logs, analytics | Powerful queries, scalable | Complex, resource-heavy |
| **Algolia** | Instant search, autocomplete | Fast, managed, great UX | Expensive, vendor lock-in |
| **Database Full-Text** | Simple search in existing DB | No additional infrastructure | Limited features, slower |
| **Solr** | Enterprise search, complex queries | Mature, feature-rich | Complex configuration |

**Use Elasticsearch when:**
- ✅ Need powerful full-text search
- ✅ Log/metric aggregation
- ✅ Complex search queries
- ✅ Can manage infrastructure

**Use Algolia when:**
- ✅ Need instant search as-you-type
- ✅ Want managed service
- ✅ Focus on end-user search experience

**Use Database Full-Text Search when:**
- ✅ Simple search requirements
- ✅ Want to avoid additional services
- ✅ Data already in relational DB

---

## Decision-Making Framework

### Step-by-Step Process

1. **Understand Requirements**
   - What are you building?
   - What are the constraints?
   - What is the scale?

2. **Identify Decision Points**
   - Which components need selecting?
   - What are the key trade-offs?

3. **Use Comparison Matrices**
   - Review relevant matrix from this guide
   - List pros/cons for your specific case

4. **Evaluate Trade-offs**
   - Use [Trade-off Analysis Framework](./trade-off-analysis-framework.md)
   - Consider: performance, cost, complexity, scalability

5. **Make Decision**
   - Choose option that best fits requirements
   - Document reasoning

6. **Plan for Change**
   - No decision is permanent
   - Design for flexibility where possible

---

## Quick Reference Cheat Sheet

### When to Use What

**SQL Database:** Transactions, complex queries, structured data  
**NoSQL Document:** Flexible schema, high writes, JSON-like data  
**NoSQL Key-Value:** Simple lookups, sessions, high scale  
**NoSQL Column-Family:** Time-series, massive writes, predictable queries  
**Graph Database:** Social networks, recommendations, connected data

**REST:** Public APIs, standard CRUD, wide compatibility  
**GraphQL:** Flexible queries, mobile apps, complex data  
**gRPC:** Microservices, low latency, typed contracts  
**WebSockets:** Real-time, bidirectional, chat/gaming

**Redis Cache:** Shared cache, sub-ms reads, sessions  
**CDN:** Static content, global distribution, media  
**Application Cache:** Single server, simple caching

**RabbitMQ:** Task queues, complex routing  
**Kafka:** Event streaming, high throughput, replay  
**SQS:** Managed, serverless, AWS-native  

---

## Practice Exercise

**Scenario:** Design the data layer for an e-commerce platform

**Consider:**
- Product catalog (varying attributes per category)
- User accounts and authentication
- Order processing
- Shopping cart
- Search functionality

**Questions:**
1. Which database(s) would you choose for each component?
2. What caching strategy would you use?
3. How would you handle search?
4. What API protocol for mobile app?

**Try answering before checking the solution below!**

<details>
<summary>Click for Example Solution</summary>

**Product Catalog:** MongoDB (flexible schema, varying product attributes)  
**User Accounts:** PostgreSQL (structured, ACID for auth)  
**Orders:** PostgreSQL (transactions critical)  
**Shopping Cart:** Redis (fast, temporary data)  
**Search:** Elasticsearch (full-text product search)  
**Caching:** Redis for hot products, CDN for images  
**Mobile API:** GraphQL (flexible queries, reduce over-fetching)

**Reasoning:**
- Products have different attributes by category → NoSQL flexibility
- Orders need ACID guarantees → SQL
- Cart is temporary, needs speed → Redis
- Search needs full-text capabilities → Elasticsearch
- Mobile benefits from flexible queries → GraphQL

</details>

---

## Next Steps

- Start with [Problem Analysis Template](./problem-analysis-template.md) to define requirements
- Use this guide to select appropriate components
- Draw your architecture with [Architecture Diagram Guide](./architecture-diagram-guide.md)
- Analyze decisions with [Trade-off Analysis Framework](./trade-off-analysis-framework.md)
