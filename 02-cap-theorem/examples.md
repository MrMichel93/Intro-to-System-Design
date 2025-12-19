# CAP Theorem Real-World Examples

## Example 1: Amazon DynamoDB (AP)

**System Type**: Distributed NoSQL database  
**CAP Choice**: Availability + Partition Tolerance

**How It Works:**
- Multiple replicas across availability zones
- Writes accepted even if some replicas are unreachable
- Eventually consistent by default
- Optional strong consistency for critical reads

**Real Scenario:**
```
User adds item to shopping cart
→ Write goes to nearest replica (fast!)
→ Replicates to other zones asynchronously
→ If user views cart immediately, might see old state
→ Within seconds, all replicas consistent
```

**Why AP**: Cart availability more important than instant consistency. Better to show slightly stale cart than error page.

---

## Example 2: Google Spanner (CP with tricks)

**System Type**: Globally distributed SQL database  
**CAP Choice**: Consistency + Partition Tolerance (with high availability)

**How It Works:**
- Uses atomic clocks and GPS for time synchronization
- Provides strong consistency across regions
- During partitions, only majority partition remains available
- TrueTime API for consistent timestamps

**Real Scenario:**
```
Bank transaction in New York
→ Must check balance in all regions
→ If California unreachable and no majority
→ Transaction rejected (unavailable)
→ Consistency guaranteed
```

**Why CP**: Financial data must be consistent. Brief unavailability acceptable.

---

## Example 3: Cassandra (AP, Tunable)

**System Type**: Wide-column NoSQL database  
**CAP Choice**: Primarily AP, but tunable

**Tunable Consistency:**
```
Write/Read levels:
- ONE: Fastest, least consistent
- QUORUM: Balanced (majority must agree)
- ALL: Slowest, most consistent

Common: Write at QUORUM, Read at QUORUM
Result: Strong consistency with good availability
```

**Use Case: Social Media**
```
User profile updates:
- Write at QUORUM (2 of 3 nodes)
- Available even if 1 node down
- Consistent for majority reads
```

---

## Example 4: Zookeeper (CP)

**System Type**: Coordination service  
**CAP Choice**: Consistency + Partition Tolerance

**How It Works:**
- Used for distributed coordination
- Leader election via majority vote
- If leader unreachable, system unavailable until new leader
- Guarantees all nodes see same state

**Use Case: Configuration Management**
```
Store critical config for microservices
→ All services must see same config
→ If partition occurs, minority partition stops serving
→ Prevents split-brain scenarios
```

**Why CP**: Wrong configuration could break entire system. Better unavailable than inconsistent.

---

## Example 5: DNS (AP)

**System Type**: Domain Name System  
**CAP Choice**: Availability + Partition Tolerance

**How It Works:**
- Distributed across thousands of servers worldwide
- Caches heavily (TTL: Time To Live)
- Eventually consistent
- Always responds (even with stale data)

**Real Scenario:**
```
Update DNS: example.com → New IP address
→ Update propagates gradually
→ Some users see old IP (cached)
→ Some users see new IP
→ Eventually (minutes to hours) all consistent
```

**Why AP**: Better to show old IP than no IP. Websites need to be reachable.

---

## Example 6: MongoDB (CP by default)

**System Type**: Document database  
**CAP Choice**: Consistency + Partition Tolerance

**How It Works:**
- Primary-Secondary replication
- Writes only to primary
- If primary unreachable, elect new primary
- During election, system briefly unavailable

**Real Scenario:**
```
Primary node crashes
→ System detects failure (heartbeat)
→ Election begins (few seconds)
→ New primary elected
→ System available again
→ Brief unavailability ensured consistency
```

**Why CP**: Document databases often used for important data. Consistency matters.

---

## Example 7: Redis (Various Modes)

**System Type**: In-memory data store  
**CAP Choice**: Depends on configuration

**Single Instance: CP**
- Consistent (single source of truth)
- Not partition tolerant (if it fails, unavailable)

**Master-Replica: AP**
- Async replication (eventual consistency)
- Replicas available if master fails
- Might serve stale data

**Redis Cluster: AP**
- Distributed across nodes
- Eventual consistency
- High availability

---

## Comparison Table

| System | Type | CAP | Use Case | Trade-Off |
|--------|------|-----|----------|-----------|
| DynamoDB | NoSQL | AP | Shopping carts, sessions | Eventual consistency |
| Spanner | SQL | CP* | Financial, AdWords | Complex, expensive |
| Cassandra | NoSQL | AP | Social media, IoT | Tunable consistency |
| Zookeeper | Coordination | CP | Config, leader election | Unavailable during elections |
| MongoDB | Document | CP | Applications, CMS | Brief unavailability possible |
| Redis | Cache | Varies | Caching, sessions | Depends on setup |

---

## Decision Patterns

### Pattern 1: Different Choices for Different Data

**E-commerce Site:**
```
Product Catalog → AP (eventual consistency OK)
Inventory Count → CP (must be accurate)
Shopping Cart → AP (availability important)
Orders → CP (must be consistent)
User Reviews → AP (eventual consistency OK)
```

### Pattern 2: Degrade Gracefully

**Video Streaming:**
```
Normal operation → Strong consistency
Network issues → Serve cached/stale data (AP mode)
User doesn't know difference
Better than error page
```

### Pattern 3: Compensating Actions

**Ride Sharing:**
```
During partition → Accept bookings (AP)
After partition heals → Check for conflicts
Resolve: Call customer, offer alternatives
Business process handles inconsistency
```

---

## Summary

Different systems make different CAP trade-offs based on their requirements:

- **AP Systems** (DynamoDB, Cassandra, DNS): Prioritize availability, accept eventual consistency
- **CP Systems** (Zookeeper, MongoDB): Prioritize consistency, accept brief unavailability
- **Hybrid** (Cassandra with quorums): Tune based on needs
- **Context Matters**: Same company might use different systems for different data

**Key Insight**: There's no universally "best" choice - it depends on your specific requirements!

## Next Steps

Explore [Design Exercises](./design-exercise.md) to practice applying CAP theorem to real scenarios!
