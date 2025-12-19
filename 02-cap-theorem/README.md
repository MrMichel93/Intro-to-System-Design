# CAP Theorem

## What is CAP Theorem?

**CAP Theorem** states that in a distributed database system, you can only guarantee **two out of three** of the following properties at the same time:

- **C**onsistency: All nodes see the same data at the same time
- **A**vailability: Every request receives a response (success or failure)
- **P**artition Tolerance: System continues to operate despite network failures

**Discovered by**: Eric Brewer in 2000, proven by Seth Gilbert and Nancy Lynch in 2002.

## Why It Matters

When you design a distributed system (which is necessary for any large-scale application), network failures **will happen**. This means:

> **In reality, you must choose between Consistency and Availability during network partitions.**

This fundamental trade-off shapes how you design databases and distributed systems.

## The Three Properties Explained

### Consistency (C)

**Definition**: Every read receives the most recent write or an error.

**Example**: Banking System
```
You have $100 in your account
You withdraw $50 from ATM A
Immediately check balance at ATM B
You MUST see $50 (not $100)
```

If ATM B showed $100 (old data), that's **inconsistent** and could let you withdraw another $50, overdrawing your account!

**Consistency means**: All nodes have the exact same data at any moment.

---

### Availability (A)

**Definition**: Every request gets a (non-error) response, even if it's not the most recent data.

**Example**: Social Media Feed
```
You post a photo
Your friend immediately views their feed
Your post might not appear yet (eventual consistency)
But they still see *something* (their feed loads)
```

**Availability means**: The system always responds, never says "I'm unavailable."

---

### Partition Tolerance (P)

**Definition**: System continues to work even when network messages are lost or delayed between nodes.

**Example**: E-commerce across datacenters
```
Datacenter A (USA) and Datacenter B (Europe)
Network cable between them is cut
Both datacenters keep working independently
```

**Partition Tolerance means**: System survives network failures between components.

**Important**: In a distributed system, network partitions **WILL happen**. You must handle them. So in practice, you're choosing between C and A when P occurs.

## The Trade-Off

### CA (Consistency + Availability)
**Give up**: Partition Tolerance

**Reality**: This is only possible in a **non-distributed** system (single server).

**Why**: If your system is distributed and a network partition happens, you MUST handle it. You can't ignore partitions.

**Example**: Single database server (not distributed)

---

### CP (Consistency + Partition Tolerance)
**Give up**: Availability

**Behavior**: During a partition, the system returns errors or timeouts rather than potentially wrong data.

**Example**: Banking systems

```
Scenario:
- Bank database split due to network issue
- User tries to withdraw money
- System can't verify balance across both parts
- System REFUSES transaction (unavailable)
- Better unavailable than wrong!
```

**Real Systems**: MongoDB, HBase, Redis (with strong consistency), Zookeeper

**When to choose CP**:
- Financial transactions
- Inventory systems
- Any system where wrong data is worse than no data

---

### AP (Availability + Partition Tolerance)
**Give up**: Consistency (strict consistency)

**Behavior**: System always responds, even if the data might be stale or inconsistent.

**Example**: Social media platforms

```
Scenario:
- Social network split due to network issue
- User posts a photo
- System accepts it (available!)
- Other users might not see it immediately
- Eventually, everyone will see it (eventual consistency)
```

**Real Systems**: Cassandra, DynamoDB, Couchbase, DNS

**When to choose AP**:
- Social media feeds
- Shopping carts (can handle temporary inconsistency)
- Caching layers
- Analytics and monitoring

## Detailed Examples

### Example 1: Twitter (AP System)

**Choice**: Availability + Partition Tolerance

**How it works**:
```
You tweet: "Hello World!"

Write goes to one datacenter first (available!)
Datacenter 1: ✅ Tweet saved
Datacenter 2: ⏳ Will sync soon
Datacenter 3: ⏳ Will sync soon

Some users see your tweet immediately
Others see it a few seconds later (eventual consistency)
```

**Why this works for Twitter**:
- Better to show slightly old feed than no feed
- Posting should never fail
- Temporary inconsistency is acceptable
- Eventually, everyone sees everything

---

### Example 2: Bank Account (CP System)

**Choice**: Consistency + Partition Tolerance

**How it works**:
```
You have $100
You try to withdraw $60 from ATM

System checks ALL datacenters for your balance
If network partition exists and it can't confirm:
→ REJECT transaction (unavailable for this user)
→ Better safe than sorry!

Only process transaction if 100% sure about balance
```

**Why this works for banking**:
- Wrong balance = disaster
- Temporary unavailability = annoying but safe
- Consistency is non-negotiable

---

### Example 3: Shopping Cart (AP System)

**Choice**: Availability + Partition Tolerance

**Amazon's approach**:
```
Add item to cart during network partition
→ Accept the addition (available!)
→ Might show different cart contents temporarily

When partition heals:
→ Merge both versions
→ Worst case: Item added twice
→ User can remove duplicate

Better to risk duplicate than lose sale!
```

**Why AP for shopping carts**:
- Availability = more sales
- Temporary inconsistency acceptable
- Easy to resolve conflicts
- Business value trumps perfect consistency

---

### Example 4: Uber/Lyft Location (CP System)

**Choice**: Consistency + Partition Tolerance

**How it works**:
```
Driver location must be accurate
If system can't get consistent location:
→ Show "Unable to find drivers" (unavailable)
→ Don't show wrong location!

Showing driver 5 miles away when they're actually 1 mile away = bad experience
```

**Why CP for ride-sharing**:
- Location accuracy critical
- Users prefer no result over wrong result
- Safety implications

---

## How to Choose: Decision Framework

### Choose CP (Consistency + Partition Tolerance) when:

✅ **Data correctness is critical**
- Financial transactions
- Inventory management
- Booking systems (flights, hotels)
- Medical records
- Trading platforms

✅ **Wrong data causes problems**
- Duplicate charges
- Double bookings
- Stock overselling

✅ **Users can tolerate temporary unavailability**
- Better to wait than get wrong answer

---

### Choose AP (Availability + Partition Tolerance) when:

✅ **Availability is more important than instant consistency**
- Social media feeds
- Shopping carts
- User profiles
- Comments and likes
- Analytics dashboards

✅ **Eventual consistency is acceptable**
- Users won't notice brief inconsistencies
- Data will be correct "soon enough"

✅ **System must always respond**
- Downtime loses money
- User frustration from errors

---

## Eventual Consistency Explained

**Eventual Consistency** is a consistency model used in AP systems.

**Concept**: Given enough time (without new updates), all replicas will eventually have the same data.

**Timeline**:
```
Time 0: User posts photo
Time 1: Server A has photo ✅
Time 2: Server B syncs, has photo ✅
Time 3: Server C syncs, has photo ✅
Time 4: All consistent! ✅✅✅
```

**Acceptable delay**: Usually seconds to minutes, depending on system.

**Real-world analogy**: 
- Like news spreading in a town
- Not everyone hears it at the exact same moment
- But eventually everyone knows

---

## Strategies to Handle Trade-Offs

### 1. Quorum Reads/Writes
Balance between consistency and availability by requiring majority agreement.

```
3 replicas total
Write succeeds when 2 replicas confirm (quorum)
Read from 2 replicas, take newer value
Guarantees: No stale reads if W + R > N
```

**Cassandra uses this**: Tunable consistency

---

### 2. Multi-Version Concurrency Control (MVCC)
Keep multiple versions of data to serve reads while updates happen.

---

### 3. Conflict Resolution
When inconsistencies occur, have a strategy:
- **Last Write Wins**: Timestamp-based
- **Application-Level**: Let app decide
- **User Choice**: Show conflict, let user merge

**Example**: Shopping cart shows "Item added twice during sync issue. Remove extra?"

---

### 4. Hybrid Approaches
Different consistency for different data:

**Example: E-commerce site**
```
Product catalog: AP (eventual consistency OK)
Inventory count: CP (must be accurate)
Shopping cart: AP (availability important)
Payment: CP (must be consistent)
```

---

## CAP in Practice: Real Systems

### MongoDB (CP)
- Chooses consistency over availability
- Primary node must be reachable for writes
- If network partition, only partition with primary is available
- Other partition is read-only or unavailable

### Cassandra (AP)
- Chooses availability over strict consistency
- Writes always accepted
- Eventual consistency across replicas
- Tunable consistency levels

### DynamoDB (AP)
- Highly available
- Eventual consistency by default
- Strong consistency available as option
- Multi-region replication

### Redis (Can be CP or AP)
- Single instance: CP (consistent but goes down if fails)
- Cluster mode: Tunable (can choose)
- Often used as AP (cache, eventual consistency OK)

---

## Common Misconceptions

### ❌ Myth 1: "CP means zero availability"
**Reality**: CP systems are usually available. They become unavailable only during network partitions when they can't guarantee consistency.

### ❌ Myth 2: "AP means no consistency at all"
**Reality**: AP systems have eventual consistency. They're not randomly inconsistent - data converges to correct state.

### ❌ Myth 3: "You choose CAP once for entire system"
**Reality**: Different parts of your system can make different choices. Hybrid approaches are common.

### ❌ Myth 4: "CAP is about normal operation"
**Reality**: CAP is specifically about behavior during network partitions.

---

## Summary

- **CAP Theorem**: Can only guarantee 2 of 3 (Consistency, Availability, Partition Tolerance)
- **In distributed systems**: Must handle partitions, so choose C or A
- **CP Systems**: Consistent but may be unavailable during partitions
- **AP Systems**: Always available but may be temporarily inconsistent
- **Choose based on your needs**: What's worse - wrong data or no data?
- **Hybrid approaches**: Different CAP choices for different parts of system

### Quick Decision Guide:
```
Can you tolerate being briefly down? 
├─ Yes → Consider CP (Consistency)
└─ No → Consider AP (Availability)

Is wrong data worse than no data?
├─ Yes → Choose CP
└─ No → Choose AP
```

## Next Steps

Now that you understand CAP theorem, you're ready to explore **[Scalability](../03-scalability/)** where you'll see how these principles apply to growing systems!
