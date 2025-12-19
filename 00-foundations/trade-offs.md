# System Design Trade-Offs

In system design, there's rarely a perfect solution. Every decision involves trade-offs - choosing one benefit often means accepting a drawback. This document explores common trade-offs you'll encounter.

## The Golden Rule

> **There is no single "best" solution - only solutions that best fit your specific requirements, constraints, and context.**

## Core Trade-Off Categories

### 1. Performance vs. Consistency

#### The Trade-Off
Fast systems sometimes sacrifice data consistency. Consistent systems sometimes sacrifice speed.

#### Scenario: Social Media Feed

**Option A: Strong Consistency (Slow)**
- When user posts, system updates ALL followers' feeds immediately
- Guarantees everyone sees posts in exact order
- Can take several seconds for millions of followers
- **Trade-off**: Slower posting, but perfect consistency

**Option B: Eventual Consistency (Fast)**
- User's post is accepted immediately
- Followers' feeds updated asynchronously over next few seconds
- Some followers see post before others
- **Trade-off**: Faster posting, but temporary inconsistencies

**Real-World Example:**
- **Twitter**: Uses eventual consistency (fast posts, feeds update gradually)
- **Banking**: Uses strong consistency (transactions must be exact, slower is acceptable)

**When to Choose:**
- **Strong Consistency**: Financial transactions, inventory counts, booking systems
- **Eventual Consistency**: Social media feeds, view counts, recommendations

---

### 2. Scalability vs. Simplicity

#### The Trade-Off
Highly scalable systems are complex. Simple systems are harder to scale.

#### Scenario: Blog Platform

**Option A: Simple Monolith**
```
[Single Server with App + Database]
```
- One codebase, one deployment
- Easy to develop and debug
- Works great for < 10,000 users
- **Trade-off**: Simple to build, but hits limits quickly

**Option B: Microservices**
```
[User Service] [Post Service] [Comment Service]
     ↓              ↓              ↓
[Load Balancer] → [API Gateway]
```
- Each service scales independently
- More complex deployment
- Requires orchestration (Kubernetes)
- **Trade-off**: Scales to millions, but much more complex

**Real-World Example:**
- **Small Startup**: Monolith (fast development, prove market fit)
- **Large Company**: Microservices (need scale, have resources)

**When to Choose:**
- **Simple**: Starting out, small team, proving concept, < 100K users
- **Complex/Scalable**: Growing fast, large team, need reliability, millions of users

---

### 3. Availability vs. Consistency (CAP Theorem)

#### The Trade-Off
In a distributed system during network partition, you must choose between availability and consistency.

**CAP Theorem**: You can only have 2 of 3:
- **C**onsistency: All nodes see same data
- **A**vailability: System responds to requests (even if data might be stale)
- **P**artition Tolerance: System works despite network failures

#### Scenario: Multi-Region Database

**Option A: Choose Consistency (CP)**
- If one region loses connection, system refuses writes
- Prevents inconsistent data
- **Trade-off**: More reliable data, but system might be unavailable

**Option B: Choose Availability (AP)**
- System accepts writes even if regions can't communicate
- Might have conflicting data temporarily
- **Trade-off**: Always available, but might have temporary inconsistencies

**Real-World Example:**
- **CP System**: Banking (can't risk inconsistent account balances)
- **AP System**: Amazon shopping cart (better to show potentially stale items than be down)

---

### 4. Cost vs. Performance

#### The Trade-Off
Better performance costs more. Saving money might mean slower systems.

#### Scenario: Image Storage

**Option A: Premium CDN (Fast, Expensive)**
- Images served from edge locations worldwide
- Sub-100ms load times globally
- Cost: $1000/month for 1TB
- **Trade-off**: Users happy with speed, higher costs

**Option B: Basic Object Storage (Slower, Cheap)**
- Images served from single region
- Can be 500ms+ for distant users
- Cost: $100/month for 1TB
- **Trade-off**: Saves $900/month, but worse user experience

**Hybrid Approach:**
- Cache popular images on CDN (10% of images)
- Keep long-tail images in object storage
- Cost: $300/month
- **Trade-off**: Balance between cost and performance

**When to Choose:**
- **Premium/Expensive**: High-value users, competitive advantage
- **Basic/Cheap**: Starting out, non-critical assets, tight budget
- **Hybrid**: Most common - optimize for what matters

---

### 5. Latency vs. Throughput

#### The Trade-Off
Optimizing for individual request speed vs. total requests handled.

#### Scenario: API Server

**Option A: Optimize for Latency**
- Handle each request as fast as possible
- Simple, synchronous processing
- Average response: 50ms
- Handles 1,000 req/sec
- **Trade-off**: Fast responses, but limited total capacity

**Option B: Optimize for Throughput**
- Batch requests together
- Process more efficiently in groups
- Average response: 200ms (slower)
- Handles 10,000 req/sec
- **Trade-off**: Higher capacity, but individual requests slower

**Real-World Example:**
- **Latency-Optimized**: Real-time chat (every message must be instant)
- **Throughput-Optimized**: Batch analytics processing (total volume matters more)

---

### 6. Read vs. Write Performance

#### The Trade-Off
Optimizing for reading data vs. writing data.

#### Scenario: Social Media Platform

**Option A: Optimize for Reads (Fan-Out on Write)**
- When user posts, write to all followers' feeds immediately
- **Write**: Slow (must update millions of feeds)
- **Read**: Very fast (pre-computed feeds)
- **Trade-off**: Posting slower, but feeds load instantly

**Option B: Optimize for Writes (Fan-Out on Read)**
- When user posts, just store the post
- **Write**: Fast (single database insert)
- **Read**: Slow (must compute feed for each user)
- **Trade-off**: Fast posting, but feed loading slower

**Real-World Usage:**
- **Twitter**: Fan-out on write (reading is more common than posting)
- **Analytics System**: Fan-out on read (writing is constant, reading is occasional)

**Hybrid Approach:**
- Regular users: Fan-out on write
- Celebrities (millions of followers): Fan-out on read (too expensive to write to millions)

---

### 7. Normalization vs. Denormalization

#### The Trade-Off
Database design: organized vs. redundant data.

#### Scenario: E-commerce Product Database

**Option A: Normalized (No Redundancy)**
```sql
Products: id, name, category_id
Categories: id, name, description
```
- Each piece of data stored once
- Need JOIN to get product with category
- **Trade-off**: Clean data structure, but slower queries

**Option B: Denormalized (Redundant Data)**
```sql
Products: id, name, category_id, category_name, category_description
```
- Category info duplicated in every product
- No JOIN needed
- If category changes, must update all products
- **Trade-off**: Faster queries, but data duplication and update complexity

**When to Choose:**
- **Normalized**: Data changes frequently, storage is expensive, data integrity critical
- **Denormalized**: Read-heavy workload, storage is cheap, performance matters most

---

### 8. SQL vs. NoSQL Databases

#### The Trade-Off
Structured relational databases vs. flexible document databases.

#### Comparison:

| Aspect | SQL (e.g., PostgreSQL) | NoSQL (e.g., MongoDB) |
|--------|----------------------|----------------------|
| **Schema** | Fixed, predefined | Flexible, can change |
| **Relationships** | Natural (foreign keys) | Manual (references) |
| **Scalability** | Vertical (bigger servers) | Horizontal (more servers) |
| **Consistency** | Strong (ACID) | Eventually consistent |
| **Queries** | Powerful (SQL) | Simpler |
| **Use Case** | Complex relationships | Hierarchical data |

**Scenario: User Profile System**

**SQL Approach:**
```sql
-- Structured, relational
users: id, name, email
addresses: id, user_id, street, city
preferences: id, user_id, key, value
```
- Clear relationships
- Easy to query "all users in city X"
- **Trade-off**: Schema changes require migrations

**NoSQL Approach:**
```json
// Flexible document
{
  "id": "123",
  "name": "Alice",
  "email": "alice@example.com",
  "addresses": [
    {"street": "123 Main St", "city": "NYC"}
  ],
  "preferences": {
    "theme": "dark",
    "notifications": true
  }
}
```
- Flexible structure (easy to add fields)
- Everything in one document (fast reads)
- **Trade-off**: Harder to query across relationships

**When to Choose:**
- **SQL**: Complex relationships, need transactions, business applications
- **NoSQL**: Flexible schema, high write volume, hierarchical data, need horizontal scaling

---

### 9. Synchronous vs. Asynchronous Processing

#### The Trade-Off
Wait for operation to complete vs. process in background.

#### Scenario: User Registration

**Option A: Synchronous (Wait for Everything)**
```
1. User submits form
2. Create account ⏱️ 100ms
3. Send welcome email ⏱️ 2000ms
4. Create default preferences ⏱️ 50ms
5. Update analytics ⏱️ 100ms
6. Show success message
Total: 2250ms (user waits)
```
- User sees exactly what happened
- Must wait for all operations
- **Trade-off**: Complete feedback, but slow

**Option B: Asynchronous (Background Processing)**
```
1. User submits form
2. Create account ⏱️ 100ms
3. Queue: email, preferences, analytics
4. Show success message immediately
Total: 100ms (user waits)
Background: Process queue items
```
- Fast response to user
- Email might arrive a few seconds later
- User doesn't see if email fails
- **Trade-off**: Much faster, but less immediate feedback

**When to Choose:**
- **Synchronous**: Payment processing, critical operations, user needs confirmation
- **Asynchronous**: Emails, notifications, analytics, image processing

---

### 10. Caching vs. Fresh Data

#### The Trade-Off
Fast but potentially stale data vs. slow but always current.

#### Scenario: Product Prices

**Option A: Always Query Database (Fresh Data)**
```
User requests product price
→ Query database
→ Return current price
⏱️ 100ms per request
```
- Always accurate
- Every request hits database
- **Trade-off**: Always correct, but slower and more database load

**Option B: Cache Aggressively (Fast, Potentially Stale)**
```
User requests product price
→ Check cache (exists)
→ Return cached price
⏱️ 1ms per request
Cache refreshes every 5 minutes
```
- Very fast
- Might show old price for up to 5 minutes
- **Trade-off**: Much faster, but can be outdated

**Strategies:**
- **Cache with TTL**: Cache for 5 minutes, then refresh
- **Cache with Invalidation**: Update cache when data changes
- **Write-Through Cache**: Update both database and cache together

**When to Choose:**
- **Always Fresh**: Prices, inventory, personal account data
- **Cached**: Product descriptions, images, public data, search results

---

## Multi-Dimensional Trade-Offs

### Example: Design a Video Streaming Platform

You need to make multiple connected trade-offs:

#### Trade-Off 1: Video Quality vs. Bandwidth
- **Higher Quality**: Better experience, uses 10x more bandwidth
- **Lower Quality**: Worse experience, saves costs
- **Solution**: Adaptive streaming (adjust based on user's internet)

#### Trade-Off 2: Storage Cost vs. Processing
- **Store Multiple Versions**: Fast serving, but 5x storage cost
- **Encode On-Demand**: Saves storage, but slower and CPU intensive
- **Solution**: Pre-encode popular videos, encode long-tail on-demand

#### Trade-Off 3: Availability vs. Consistency
- **Allow Stale View Counts**: System always available
- **Exact View Counts**: Might be unavailable under load
- **Solution**: Eventually consistent view counts (acceptable for this use case)

#### Trade-Off 4: Global Reach vs. Cost
- **CDN Everywhere**: Fast globally, expensive
- **Single Region**: Cheaper, but slow for distant users
- **Solution**: CDN in high-traffic regions, origin server for rest

---

## How to Make Trade-Off Decisions

### 1. Understand Your Requirements
**Questions to ask:**
- What's the primary use case?
- Who are the users?
- What are the scale requirements?
- What's the budget?
- What absolutely cannot fail?

### 2. Identify Constraints
**Common constraints:**
- Budget limitations
- Time to market
- Team expertise
- Existing infrastructure
- Legal/regulatory requirements

### 3. Prioritize
**Use a framework:**
1. **Must Have**: Non-negotiable (e.g., secure payments)
2. **Should Have**: Important but flexible (e.g., sub-second load times)
3. **Nice to Have**: Would be great (e.g., real-time analytics)

### 4. Consider Context
**Different stages, different trade-offs:**

**Startup (Months 1-12):**
- Prioritize: Speed of development, proving market fit
- Accept: Technical debt, limited scalability
- Choice: Monolith, managed services, simple architecture

**Growth Stage (Year 1-3):**
- Prioritize: Scalability, reliability
- Accept: Higher costs, more complexity
- Choice: Start breaking into services, add caching, scale horizontally

**Mature (Year 3+):**
- Prioritize: Efficiency, optimization
- Accept: Complex infrastructure
- Choice: Microservices, custom solutions, multi-region

### 5. Measure and Adjust
**Always:**
- Monitor actual usage patterns
- Measure key metrics
- Be ready to change decisions
- Don't over-optimize prematurely

---

## Common Mistakes

### ❌ Mistake 1: Premature Optimization
**Problem**: Over-engineering for scale you don't have yet
**Example**: Building microservices for 100 users
**Solution**: Start simple, scale when needed

### ❌ Mistake 2: Ignoring Cost
**Problem**: Choosing most performant solution without considering budget
**Example**: Using premium CDN for internal admin tool
**Solution**: Match cost to business value

### ❌ Mistake 3: Following Trends Blindly
**Problem**: Using technology because it's popular, not because it fits
**Example**: "Everyone uses Kubernetes, so we should too"
**Solution**: Choose based on your specific needs

### ❌ Mistake 4: Not Considering Team
**Problem**: Choosing technology team doesn't know
**Example**: Picking Go when team only knows JavaScript
**Solution**: Consider team expertise and learning curve

### ❌ Mistake 5: One-Size-Fits-All
**Problem**: Using same approach for different problems
**Example**: Using same database for all use cases
**Solution**: Different problems need different solutions

---

## Trade-Off Decision Template

Use this template when making design decisions:

```
Decision: [What you need to decide]

Option A: [First approach]
Pros:
- 
- 
Cons:
- 
- 
Cost: 
Complexity: 

Option B: [Second approach]
Pros:
- 
- 
Cons:
- 
- 
Cost: 
Complexity: 

Context:
- Current scale:
- Expected scale in 1 year:
- Budget:
- Team expertise:
- Critical requirements:

Recommendation: [Your choice]
Reasoning: [Why this choice fits your context]
Future Reconsideration: [When to revisit this decision]
```

---

## Summary

Key principles for making trade-off decisions:

✅ **No perfect solution** - only solutions that fit your context  
✅ **Start simple** - add complexity only when needed  
✅ **Know your priorities** - what matters most for your use case?  
✅ **Consider constraints** - budget, time, team, scale  
✅ **Be ready to change** - as you grow, trade-offs change  
✅ **Measure don't guess** - use data to make decisions  
✅ **Document decisions** - explain why you chose each approach  

---

## Next Steps

Now that you understand trade-offs, you're ready to move on to **[Back-of-Envelope Calculations](../01-back-of-envelope/)** to learn how to estimate system requirements and make data-driven trade-off decisions!
