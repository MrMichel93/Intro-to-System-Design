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

## The System Design Process

### Step 1: Understand Requirements
- Ask clarifying questions
- Define functional requirements
- Identify non-functional requirements
- Establish constraints

### Step 2: Establish Scope
- Define what's in scope
- What's out of scope
- Expected timeline
- Available resources

### Step 3: High-Level Design
- Identify major components
- Draw architecture diagram
- Define data flow
- Choose technologies

### Step 4: Detailed Design
- Deep dive into critical components
- Design database schema
- Define APIs
- Plan for scaling

### Step 5: Identify Bottlenecks
- Single points of failure?
- Performance bottlenecks?
- Security vulnerabilities?
- Scaling limitations?

### Step 6: Optimize and Iterate
- Add caching
- Implement load balancing
- Database optimization
- Consider trade-offs

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
