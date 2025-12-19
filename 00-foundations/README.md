# System Design Foundations

## What is a System?

A **system** is a collection of components that work together to achieve a specific goal. In software engineering, a system combines hardware, software, data, and networks to provide functionality to users.

**Simple Examples:**
- **Email System**: Servers, databases, clients (web/mobile), networking protocols
- **Gaming System**: Game servers, player devices, matchmaking service, leaderboard database
- **School Portal**: Web application, student database, authentication service, file storage

Think of a system like a restaurant:
- **Kitchen** = Backend servers (processing)
- **Waiters** = APIs (communication)
- **Menu** = User interface
- **Storage** = Database
- **Delivery service** = Network/CDN

## Why Study System Design?

### 1. Real-World Relevance
Every app you use is a complex system:
- Instagram handles billions of photos
- Netflix streams millions of videos simultaneously
- Google processes billions of searches per day

### 2. Career Skills
- Required for technical interviews at top companies
- Essential for senior engineering roles
- Helps you build better applications

### 3. Problem-Solving
System design teaches you to:
- Break down complex problems
- Make informed trade-offs
- Think at scale
- Consider costs and constraints

## Key Concepts in System Design

### 1. Requirements Gathering

Before designing any system, you must understand:

#### Functional Requirements
**What should the system do?**

Examples for a social media app:
- Users can create accounts
- Users can post photos
- Users can follow other users
- Users can like and comment on posts
- Users receive notifications

#### Non-Functional Requirements
**How should the system behave?**

**Performance:**
- Response time: Pages should load in < 200ms
- Throughput: Handle 10,000 requests per second

**Scalability:**
- Support 10 million users
- Store 100 million photos

**Availability:**
- 99.9% uptime (less than 9 hours downtime per year)
- System should work even if some servers fail

**Reliability:**
- No data loss
- Consistent behavior

**Security:**
- Encrypted data transmission
- Secure authentication
- Protection against attacks

**Cost:**
- Infrastructure budget
- Operational costs

### 2. Constraints and Assumptions

Always clarify:
- **User base**: 100 users or 100 million?
- **Geographic distribution**: Single country or worldwide?
- **Traffic patterns**: Steady or spiky (e.g., Black Friday)?
- **Data volume**: Megabytes or petabytes?
- **Budget**: Unlimited resources or cost-conscious?

**Example Conversation:**

> **Interviewer**: Design a URL shortener like bit.ly
> 
> **You**: Great! Let me clarify the requirements:
> - How many URLs do we shorten per day? (Scale)
> - Do we need analytics on clicks? (Features)
> - Should short URLs expire? (Behavior)
> - What's our expected read/write ratio? (Traffic patterns)
> - Do we need custom short URLs? (Functionality)

## Components of a System

### 1. Client
**What users interact with**
- Web browsers
- Mobile apps
- Desktop applications
- IoT devices

**Example**: Instagram mobile app on your phone

### 2. Server
**Processes requests and executes business logic**
- Web servers (handle HTTP requests)
- Application servers (run business logic)
- Background workers (process async tasks)

**Example**: Instagram's backend that processes photo uploads

### 3. Database
**Stores and retrieves data**
- **SQL Databases**: Structured data (MySQL, PostgreSQL)
- **NoSQL Databases**: Flexible schemas (MongoDB, Cassandra)
- **In-Memory**: Fast access (Redis, Memcached)

**Example**: Instagram's database storing user profiles and posts

### 4. Load Balancer
**Distributes incoming traffic across multiple servers**
- Prevents any single server from being overwhelmed
- Improves availability

**Example**: Directs your Instagram request to one of many servers

### 5. Cache
**Stores frequently accessed data in memory**
- Reduces database load
- Improves response times

**Example**: Your Instagram feed cached for quick loading

### 6. CDN (Content Delivery Network)
**Delivers static content from servers close to users**
- Images, videos, CSS, JavaScript
- Reduces latency

**Example**: Instagram photos served from a server near you

### 7. Message Queue
**Enables asynchronous communication between services**
- Decouples components
- Handles traffic spikes

**Example**: Instagram queues notification delivery tasks

## Basic System Architecture Patterns

### 1. Monolithic Architecture
**Everything in one application**

```
┌─────────────────────────────────┐
│    Single Application           │
│                                 │
│  ┌─────────┐  ┌──────────┐    │
│  │   UI    │  │ Business │    │
│  │ Layer   │─▶│  Logic   │    │
│  └─────────┘  └──────────┘    │
│                    │            │
│              ┌─────▼─────┐     │
│              │ Database  │     │
│              └───────────┘     │
└─────────────────────────────────┘
```

**Pros:**
- Simple to develop and deploy
- Easy to test
- Good for small applications

**Cons:**
- Hard to scale specific features
- Entire app must be redeployed for any change
- Can become complex over time

**Example**: A small school website

### 2. Client-Server Architecture
**Clients request services from servers**

```
┌─────────┐     Request      ┌─────────┐
│ Client  │─────────────────▶│ Server  │
│ (Web/   │                  │         │
│ Mobile) │◀─────────────────│         │
└─────────┘     Response     └────┬────┘
                                  │
                             ┌────▼─────┐
                             │ Database │
                             └──────────┘
```

**Example**: Most web applications

### 3. Three-Tier Architecture
**Separated into presentation, logic, and data layers**

```
┌──────────────┐
│ Presentation │  (Web/Mobile UI)
└──────┬───────┘
       │
┌──────▼───────┐
│   Business   │  (Application Logic)
│     Logic    │
└──────┬───────┘
       │
┌──────▼───────┐
│     Data     │  (Database)
└──────────────┘
```

**Benefits:**
- Separation of concerns
- Each tier can scale independently
- Easier maintenance

**Example**: E-commerce websites

### 4. Microservices Architecture
**Application divided into small, independent services**

```
┌─────────┐
│  User   │
│ Service │
└────┬────┘
     │
┌────▼─────┐    ┌──────────┐    ┌─────────┐
│   API    │    │  Order   │    │ Payment │
│ Gateway  │───▶│ Service  │───▶│ Service │
└──────────┘    └──────────┘    └─────────┘
     │
┌────▼──────┐
│Notification│
│  Service   │
└───────────┘
```

**Pros:**
- Services scale independently
- Technology flexibility
- Fault isolation

**Cons:**
- Complex deployment
- Network overhead
- Harder to test

**Example**: Netflix, Amazon

## Design Process Framework

Follow this 8-step framework when designing any system:

### Step 1: Clarify Requirements
- Ask clarifying questions to understand the problem
- Define functional requirements (what the system should do)
- Identify non-functional requirements (how the system should behave)
- Establish constraints and assumptions
- Understand the scope

**Example Questions:**
- How many users will the system support?
- What are the key features?
- What's the expected traffic pattern?
- Are there any specific performance requirements?

### Step 2: Estimate Scale
- Calculate expected traffic (requests per second)
- Estimate storage requirements
- Determine bandwidth needs
- Plan for growth over time
- Use back-of-envelope calculations

**Example:**
- Daily active users: 10 million
- Average requests per user: 20
- Total daily requests: 200 million
- Requests per second: ~2,300 (200M / 86,400)

### Step 3: Define APIs
- Design the key interfaces
- Specify API endpoints (REST, GraphQL, etc.)
- Define request/response formats
- Consider API versioning
- Document expected behavior

**Example APIs for a URL shortener:**
```
POST /api/shorten - Create short URL
GET  /api/:shortUrl - Redirect to original URL
GET  /api/stats/:shortUrl - Get click statistics
```

### Step 4: Define Data Model
- Identify entities and their relationships
- Design database schema
- Choose appropriate data types
- Plan for data access patterns
- Consider indexing strategy

**Example Schema:**
```
Users: id, email, created_at
URLs: id, original_url, short_code, user_id, created_at
Clicks: id, url_id, timestamp, user_agent, ip_address
```

### Step 5: High-Level Design
- Draw architecture diagram
- Identify major components (clients, servers, databases, caches)
- Show data flow between components
- Choose technology stack
- Keep it simple initially

### Step 6: Detailed Design
- Deep dive into critical components
- Design algorithms for core features
- Plan for data consistency
- Consider security measures
- Design for reliability

### Step 7: Identify Bottlenecks
- Find single points of failure
- Identify performance bottlenecks
- Discover scaling limitations
- Consider security vulnerabilities
- Think about edge cases

**Common Bottlenecks:**
- Database becomes read/write bottleneck
- Single server can't handle traffic
- Network bandwidth limitations
- Storage capacity constraints

### Step 8: Scale the Design
- Add caching layers
- Implement load balancing
- Use database replication and sharding
- Add CDN for static content
- Consider microservices for specific components
- Plan for horizontal scaling

**Scaling Strategies:**
- Cache frequently accessed data
- Distribute load across multiple servers
- Partition data across databases
- Use message queues for async processing

## Common System Design Principles

### 1. Keep It Simple (KISS)
Start simple, add complexity only when needed.

**Bad**: Over-engineering a blog with microservices  
**Good**: Start with a monolith, split when necessary

### 2. Don't Repeat Yourself (DRY)
Reuse components and avoid duplication.

### 3. Separation of Concerns
Each component should have a single, well-defined purpose.

### 4. Fail Fast
Detect and report errors immediately.

### 5. Design for Failure
Assume components will fail and plan accordingly.

### 6. Stateless When Possible
Stateless services are easier to scale and maintain.

## Thinking in Trade-offs

**The Golden Rule of System Design**: Every decision has costs and benefits. There is no "perfect" solution—only solutions that are appropriate for specific requirements and constraints.

### Why Trade-offs Matter

When you design a system, you're constantly making choices:
- SQL or NoSQL database?
- Synchronous or asynchronous processing?
- Strong consistency or high availability?
- Monolithic or microservices architecture?

**Each choice involves trade-offs**. Understanding these trade-offs is what separates good system designers from great ones.

### Common Trade-offs in System Design

#### 1. Performance vs Scalability
- **Performance**: How fast a single request is processed
- **Scalability**: How many requests the system can handle

**Trade-off**: Sometimes optimizations for a single request make it harder to scale.

**Example:**
- Complex in-memory cache = Fast (high performance)
- But hard to scale across multiple servers (low scalability)

#### 2. Consistency vs Availability
See CAP Theorem (Module 02) for deep dive.

- **Strong Consistency**: All nodes see same data (may be unavailable during failures)
- **High Availability**: System always responds (may serve stale data)

**Example:**
- Banking: Choose consistency (wrong balance is unacceptable)
- Social media: Choose availability (delayed posts are acceptable)

#### 3. Latency vs Throughput
- **Latency**: Time to complete a single operation
- **Throughput**: Number of operations completed per unit time

**Trade-off**: Batching increases throughput but adds latency.

**Example:**
- Send each email immediately = Low latency, lower throughput
- Batch 100 emails and send together = Higher latency, higher throughput

#### 4. Read Performance vs Write Performance
- **Optimize for reads**: Use caching, read replicas (but writes become complex)
- **Optimize for writes**: Simple write path (but reads may be slower)

**Example:**
- Twitter: Read-heavy (optimize for fast timeline reads)
- Log system: Write-heavy (optimize for fast log writes)

#### 5. Cost vs Everything Else
More resources generally improve performance, availability, and reliability—but at a cost.

**Trade-off:**
- Running on 10 servers = More reliable, more expensive
- Running on 3 servers = Less reliable, more affordable

**Startups often choose**: Acceptable reliability at minimal cost

### How to Evaluate Trade-offs

When facing a design decision, ask:

1. **What are we optimizing for?**
   - Speed? Cost? Reliability? Simplicity?

2. **What are the requirements?**
   - What does the business actually need?
   - What can we compromise on?

3. **What are the constraints?**
   - Budget limitations?
   - Time to market?
   - Team expertise?

4. **What happens at scale?**
   - Does this decision scale to 10x, 100x users?

5. **Can we change our mind later?**
   - Is this decision reversible?
   - What's the cost of changing?

### Trade-off Decision Framework

```
For each design choice:
┌─────────────────────────────────────┐
│  What are the options?              │
│  (List 2-3 viable approaches)       │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│  What are the pros and cons?        │
│  (For each option)                  │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│  Which aligns with requirements?    │
│  (Match to your specific needs)     │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│  Make decision and document why     │
│  (Explain reasoning for later)      │
└─────────────────────────────────────┘
```

### Real-World Trade-off Examples

#### Example 1: Choosing a Database

**Scenario**: Building a social media app

**Options:**
1. **SQL Database (PostgreSQL)**
   - ✅ Pros: ACID guarantees, complex queries, mature ecosystem
   - ❌ Cons: Harder to scale horizontally, schema changes difficult

2. **NoSQL Database (MongoDB)**
   - ✅ Pros: Flexible schema, easy horizontal scaling, fast writes
   - ❌ Cons: Weaker consistency, complex queries harder

**Decision Factors:**
- Data structure: User profiles (structured) vs posts (semi-structured)
- Scale: Need to handle millions of users
- Consistency needs: User profiles need consistency, posts can be eventual

**Choice**: Hybrid approach
- PostgreSQL for user accounts and relationships (need consistency)
- MongoDB for posts and feeds (need scale and flexibility)

#### Example 2: Caching Strategy

**Scenario**: E-commerce product catalog

**Options:**
1. **Cache Everything**
   - ✅ Pros: Fastest possible reads
   - ❌ Cons: High memory costs, stale data issues, cache invalidation complexity

2. **Cache Nothing**
   - ✅ Pros: Always fresh data, simple architecture
   - ❌ Cons: Slow reads, high database load

3. **Selective Caching** (80/20 rule)
   - ✅ Pros: Good performance, reasonable cost, balanced
   - ❌ Cons: More complex than extremes

**Choice**: Selective caching
- Cache top 20% products (generate 80% traffic)
- Cache with 5-minute TTL (balance freshness vs performance)
- Invalidate cache on price/inventory changes

### Key Takeaways

✅ **No perfect solutions** - Only appropriate solutions for specific contexts  
✅ **Document your reasoning** - Future you (or your team) will thank you  
✅ **Be willing to change** - Requirements change, so should your design  
✅ **Consider scale** - What works for 100 users may not work for 100 million  
✅ **Understand the business** - Technical decisions should serve business needs  

**Remember**: The best system designers don't memorize solutions—they understand trade-offs and make informed decisions based on requirements.

## Real-World Example: Designing a Todo App

Let's apply these concepts to a simple todo app.

### Requirements
**Functional:**
- Create, read, update, delete todos
- Mark todos as complete
- User authentication

**Non-Functional:**
- Support 1,000 concurrent users
- 99% availability
- < 100ms response time

### Simple Architecture

```
┌──────────┐     HTTPS      ┌───────────┐
│  Users   │───────────────▶│ Web Server│
│ (Browser)│                │  (Node.js)│
└──────────┘                └─────┬─────┘
                                  │
                            ┌─────▼──────┐
                            │  Database  │
                            │ (PostgreSQL)│
                            └────────────┘
```

### Components:
1. **Frontend**: React app (client-side)
2. **Backend**: Node.js REST API (server-side)
3. **Database**: PostgreSQL (data storage)
4. **Authentication**: JWT tokens

### Data Model:
```sql
Users Table:
- id (primary key)
- email
- password_hash
- created_at

Todos Table:
- id (primary key)
- user_id (foreign key)
- title
- description
- completed (boolean)
- created_at
- updated_at
```

### API Endpoints:
```
POST   /api/auth/register  - Create account
POST   /api/auth/login     - Login
GET    /api/todos          - Get all todos
POST   /api/todos          - Create todo
PUT    /api/todos/:id      - Update todo
DELETE /api/todos/:id      - Delete todo
```

This simple design works for 1,000 users. As we grow, we'd add:
- Caching (Redis)
- Load balancer
- Database replicas
- CDN for static assets

## Summary

- A **system** is interconnected components working together
- Always start by understanding **requirements** (functional and non-functional)
- Define **constraints** and **assumptions** early
- Systems have common **components**: clients, servers, databases, caches, etc.
- Use appropriate **architecture patterns** based on needs
- Follow a structured **design process**
- Apply fundamental **principles**: simplicity, separation of concerns, design for failure

## Key Takeaways

✅ Always clarify requirements before designing  
✅ Start simple, add complexity as needed  
✅ Consider trade-offs in every decision  
✅ Think about scalability from the start  
✅ Design for failure, not just success  

## Next Steps

Now that you understand the foundations, move on to **[Back-of-Envelope Calculations](../01-back-of-envelope/)** to learn how to estimate system capacity and requirements!
