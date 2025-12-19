# Data Replication

## What is Replication?

**Replication** is the process of copying and maintaining database objects (data) in multiple database servers. It ensures data is available even if one server fails, improves read performance, and enables geographic distribution.

**Simple Analogy**: Like having multiple copies of an important document stored in different safes.

## Why Replicate Data?

### 1. High Availability
- If primary fails, replica takes over
- No data loss
- Minimal downtime

### 2. Performance
- Distribute read load across replicas
- Serve users from nearest replica
- Reduce latency

### 3. Disaster Recovery
- Backup in different location
- Survive datacenter failures
- Meet compliance requirements

### 4. Scalability
- Add read replicas as traffic grows
- Scale reads independently from writes

## Replication Architectures

### 1. Leader-Follower (Primary-Replica)

**Most common pattern**

```
        Writes
          ↓
    [Leader/Primary]
          ↓
    (Replication)
      ↙    ↓    ↘
[Follower] [Follower] [Follower]
    ↓         ↓         ↓
  Reads     Reads     Reads
```

**How it works:**
- All writes go to leader
- Leader replicates to followers
- Reads from any replica
- Leader election if primary fails

**Pros:**
- ✅ Simple to understand
- ✅ Good read scalability
- ✅ Data consistency (leader is source of truth)

**Cons:**
- ❌ Single point of failure for writes
- ❌ Replication lag possible
- ❌ Leader can be bottleneck

**Use cases**: MySQL, PostgreSQL, MongoDB, Redis

---

### 2. Multi-Leader (Multi-Primary)

**Multiple nodes accept writes**

```
[Leader 1]  ⟷  [Leader 2]
   ↕              ↕
[Replica]      [Replica]
```

**How it works:**
- Multiple leaders in different regions
- Each leader accepts writes
- Leaders sync with each other
- Conflict resolution needed

**Pros:**
- ✅ Better write performance
- ✅ Geographic distribution
- ✅ Works during network partitions

**Cons:**
- ❌ Complex conflict resolution
- ❌ Harder to reason about
- ❌ Potential data conflicts

**Use cases**: Multi-region apps, CouchDB, Some MySQL setups

---

### 3. Leaderless (Peer-to-Peer)

**All nodes are equal**

```
[Node 1] ⟷ [Node 2]
   ⟷         ⟷
[Node 3] ⟷ [Node 4]
```

**How it works:**
- Write to multiple nodes
- Read from multiple nodes
- Quorum-based consistency
- No single leader

**Pros:**
- ✅ High availability
- ✅ No single point of failure
- ✅ Fault tolerant

**Cons:**
- ❌ Complex consistency model
- ❌ Conflict resolution needed
- ❌ More network traffic

**Use cases**: Cassandra, DynamoDB, Riak

## Replication Methods

### Synchronous Replication

**Wait for replica confirmation before completing write**

```
Client → [Leader] → [Replica] ✓
          ↓ (wait)      ↓
       Confirm    Confirm
          ↓           ↓
Client ← [Complete!]
```

**Pros:**
- ✅ Guaranteed up-to-date replica
- ✅ No data loss on leader failure
- ✅ Strong consistency

**Cons:**
- ❌ Slower writes (wait for network)
- ❌ Unavailable if replica down
- ❌ Higher latency

**When to use**: Financial systems, critical data

---

### Asynchronous Replication

**Confirm write immediately, replicate in background**

```
Client → [Leader] → Confirm immediately
          ↓
      Background
      replication
          ↓
      [Replica] (updated later)
```

**Pros:**
- ✅ Fast writes
- ✅ Leader not blocked by replicas
- ✅ Works even if replicas slow/down

**Cons:**
- ❌ Data loss possible if leader fails
- ❌ Replication lag
- ❌ Stale reads possible

**When to use**: Social media, caching, most web apps

---

### Semi-Synchronous Replication

**Hybrid: Wait for at least one replica**

```
Client → [Leader] → [Replica 1] ✓ (wait)
          ↓         [Replica 2] (async)
      Confirm     [Replica 3] (async)
```

**Balance between safety and performance**

## Handling Replication Lag

**Replication lag**: Time between write on leader and visibility on replica

### Problem Scenarios:

**1. Read Your Own Writes**
```
User posts comment → (write to leader)
User refreshes page → (read from replica)
Comment not visible yet! (replica lagging)
```

**Solution**: Read user's own writes from leader

**2. Monotonic Reads**
```
Read 1: See comment X (from replica 1)
Read 2: Don't see comment X (from replica 2, lagging)
Time going backwards!
```

**Solution**: Stick user to one replica (session affinity)

**3. Consistent Prefix Reads**
```
Alice: "What's 2+2?"
Bob: "It's 4"

User sees:
Bob: "It's 4"  (replica 1, fast)
Alice: "What's 2+2?" (replica 2, slow)

Wrong order!
```

**Solution**: Related writes go to same partition

## Conflict Resolution

When multiple nodes accept writes, conflicts happen.

### Strategy 1: Last Write Wins (LWW)

```
Node 1: Update name to "Alice" at 10:00:01
Node 2: Update name to "Bob" at 10:00:02

Result: Name = "Bob" (most recent timestamp)
```

**Pros**: Simple
**Cons**: Data loss (Alice's update lost)

---

### Strategy 2: Version Vectors

```
Track versions from each node:
Node A: v1
Node B: v1, v2
Node C: v1

Merge: v1 from A, v2 from B
```

**Pros**: No data loss
**Cons**: Complex merging

---

### Strategy 3: Application-Level Resolution

```
Show conflict to user:
"Your cart was modified on another device. Which version?"
[Version 1: 3 items] [Version 2: 5 items]
```

**Pros**: User decides
**Cons**: User friction

---

### Strategy 4: Operational Transformation

```
Concurrent edits to document:
User A: Insert "a" at position 0
User B: Insert "b" at position 0

Transform operations to converge:
Result: "ab" or "ba" (depends on transform rules)
```

**Pros**: Automatic resolution
**Cons**: Complex to implement

## Real-World Examples

### Example 1: Netflix (Multi-Region Replication)

**Setup:**
- Active-active multi-region
- Cassandra for data store
- Asynchronous replication

**Why:**
- Global user base
- High availability critical
- Eventual consistency acceptable

---

### Example 2: Banking (Synchronous Replication)

**Setup:**
- Leader-follower
- Synchronous replication
- Strong consistency

**Why:**
- Money must be accurate
- No data loss acceptable
- Brief delay acceptable

---

### Example 3: Instagram (Asynchronous)

**Setup:**
- PostgreSQL leader-follower
- Async replication to read replicas
- Hundreds of read replicas

**Why:**
- Read-heavy (viewing photos)
- Write-light (posting photos)
- Scale reads independently

## Best Practices

### 1. Choose Appropriate Replication Mode
- **Synchronous**: Critical data (financial)
- **Asynchronous**: Most web apps
- **Semi-synchronous**: Balance

### 2. Monitor Replication Lag
- Track lag metrics
- Alert if exceeds threshold
- Plan for lag in application

### 3. Test Failover Regularly
- Practice leader failure scenarios
- Automate failover if possible
- Document runbooks

### 4. Plan for Split-Brain
- Use quorum-based leader election
- Fencing to prevent old leader
- Monitor cluster health

### 5. Consider Geographic Distribution
- Replicas near users
- Comply with data residency laws
- Balance latency vs consistency

## Summary

- **Replication** creates copies of data for availability and performance
- **Leader-Follower**: Most common, simple, good for read-heavy
- **Multi-Leader**: Better for multi-region, complex conflicts
- **Leaderless**: High availability, quorum-based
- **Synchronous**: Strong consistency, slower
- **Asynchronous**: Fast, eventual consistency
- **Handle replication lag** and **conflicts** appropriately

## Next Steps

Continue to **[Consistency Patterns](../09-consistency-patterns/)** to learn about different consistency guarantees in replicated systems!
