# Consistency Patterns

## What is Consistency?

**Consistency** in distributed systems defines what guarantees a system makes about the state of data across multiple nodes.

**Question:** After a write, when will a read see that write?

Different consistency models provide different answers, with trade-offs between performance, availability, and correctness.

## Consistency Spectrum

```
Stronger ← → Weaker

Linearizable → Sequential → Causal → Eventual
(Slowest)                             (Fastest)
(Most consistent)              (Least consistent)
```

## Strong Consistency Models

### 1. Linearizability (Strongest)

**Guarantee:** Operations appear to execute instantaneously at some point between start and completion.

**Simple explanation:** Like a single-threaded system. Once a write completes, all subsequent reads see it.

```
Timeline:
T1: Write X=1 completes
T2: Read X → Must return 1
T3: Read X → Must return 1

No read can return old value after write completes.
```

**Example: Bank Account**
```
Account balance: $100
T1: Withdraw $50 → Balance = $50
T2: Check balance → Must show $50
T3: Deposit $25 → Balance = $75

Every operation sees consistent,up-to-date value.
```

**Pros:**
- ✅ Easiest to reason about
- ✅ Behaves like single machine
- ✅ No surprises for developers

**Cons:**
- ❌ Expensive to implement
- ❌ Requires coordination (slow)
- ❌ Lower availability

**Use cases:** 
- Financial transactions
- Inventory systems
- Coordination services (Zookeeper)

---

### 2. Sequential Consistency

**Guarantee:** All nodes see operations in same order, but not necessarily real-time order.

**Difference from linearizability:** Operations don't need to respect real-time ordering.

```
Actual order:
P1: Write X=1
P2: Write Y=2

All nodes might see:
- X=1, then Y=2, OR
- Y=2, then X=1

But all nodes see SAME order.
```

**Use cases:**
- Distributed databases
- Replicated systems

---

## Causal Consistency

**Guarantee:** Operations that are causally related are seen in same order by all nodes.

**Causally related:** If operation A influenced operation B.

```
Example:
Alice posts: "What's for dinner?"
Bob replies: "Pizza!"

Causal consistency ensures everyone sees question before answer.

But unrelated posts can be seen in any order:
Charlie: "Nice weather!" (independent)
```

**Pros:**
- ✅ More available than strong consistency
- ✅ Preserves intuitive ordering
- ✅ Good for social features

**Cons:**
- ❌ More complex to implement
- ❌ Need to track causality

**Use cases:**
- Social media (comments, replies)
- Collaborative editing
- Chat systems

---

## Weak Consistency Models

### 3. Eventual Consistency

**Guarantee:** If no new updates, eventually all replicas will converge to same value.

**Timeline:**
```
T0: Write X=1 to Node A
T1: Read from Node B → Might still see X=0 (stale)
T2: Read from Node C → Might still see X=0 (stale)
...
T10: All nodes have X=1 (eventually consistent)
```

**Convergence time:** Usually seconds, depends on replication lag.

**Pros:**
- ✅ High availability
- ✅ Low latency
- ✅ Good performance
- ✅ Partition tolerant

**Cons:**
- ❌ Can read stale data
- ❌ Temporary inconsistencies
- ❌ Application must handle conflicts

**Use cases:**
- DNS
- Caching
- Social media feeds
- Shopping carts

**Real example: Amazon Shopping Cart**
```
Add item on phone → Write to Node A
View cart on laptop → Read from Node B
Item might not appear for few seconds
Eventually appears everywhere
```

---

### 4. Read-Your-Writes Consistency

**Guarantee:** User always sees their own writes.

**Problem it solves:**
```
User posts comment
User refreshes page
User doesn't see their own comment (frustrating!)
```

**Solution:**
```
Write comment → Store in session: "user wrote comment #123"
Read comments → If session says wrote #123, read from primary
                Otherwise, read from replica
```

**Pros:**
- ✅ Better user experience
- ✅ Relatively simple
- ✅ Solves common problem

**Cons:**
- ❌ More complex than pure eventual consistency
- ❌ Session management needed

**Use cases:** Any user-facing app with writes

---

### 5. Monotonic Reads

**Guarantee:** Once you read a value, you never read an older value.

**Problem it solves:**
```
Read 1: See 10 comments (from Replica A)
Read 2: See 8 comments (from Replica B, lagging)
Time went backwards!
```

**Solution:**
```
Stick user to same replica (session affinity)
Or track version, read from replica that's up-to-date
```

**Use cases:**
- Reading feeds
- Viewing lists
- Any repeated reads

---

### 6. Monotonic Writes

**Guarantee:** Writes from same client are applied in order.

**Example:**
```
User updates profile:
Write 1: Set name = "Alice"
Write 2: Set bio = "Engineer"

Guarantee: Name updated before bio, always.
```

**Use cases:**
- User profile updates
- Configuration changes

---

## Consistency + Replication Strategies

### Leader-Follower + Consistency Levels

**Option 1: Read from Leader (Strong)**
```
Writes → Leader
Reads → Leader
Result: Linearizable (but leader bottleneck)
```

**Option 2: Read from Followers (Eventual)**
```
Writes → Leader → Async replicate → Followers
Reads → Any follower
Result: Eventual consistency (fast but stale possible)
```

**Option 3: Quorum Reads/Writes (Tunable)**
```
Write to W nodes
Read from R nodes
Where W + R > N (total nodes)

Example: N=5, W=3, R=3
Write confirmed by 3 nodes
Read from 3 nodes
Guaranteed: At least 1 overlap → See latest
```

---

## Quorum Consistency

**Formula:** W + R > N

Where:
- **N** = Total replicas
- **W** = Write quorum (nodes that must acknowledge write)
- **R** = Read quorum (nodes to read from)

**Examples:**

**Strong consistency:**
```
N=3, W=3, R=1
(Write to all, read from one)
Slow writes, fast reads
```

**Eventual consistency:**
```
N=3, W=1, R=1
(Write to one, read from one)
Fast everything, but can be stale
```

**Balanced:**
```
N=3, W=2, R=2
(Quorum: majority)
Balance between speed and consistency
```

---

## Conflict Resolution

When eventual consistency allows conflicts:

### Strategy 1: Last-Write-Wins (LWW)
```
Use timestamp to determine winner
Latest timestamp wins
Simple but can lose data
```

### Strategy 2: Version Vectors
```
Track versions from each node
Merge when possible
Show conflict to user if can't auto-merge
```

### Strategy 3: CRDTs (Conflict-Free Replicated Data Types)
```
Data structures that automatically merge
Example: Counters, Sets, Maps
Used in: Redis, Riak
```

### Strategy 4: Application Logic
```
App decides how to merge
Example: Shopping cart → Union of items
```

---

## Choosing Consistency Level

### Use Strong Consistency (Linearizable) when:
- Financial transactions
- Inventory (can't oversell)
- Booking systems (no double-booking)
- Critical operations where wrong data is catastrophic

### Use Eventual Consistency when:
- Social media feeds
- Analytics and reporting
- Caching
- DNS
- Shopping carts
- User preferences

### Use Causal Consistency when:
- Comments/replies (need ordering)
- Collaborative editing
- Chat systems
- Activity feeds

---

## Real-World Examples

### Example 1: DynamoDB (Tunable)

**Default:** Eventual consistency
**Option:** Strong consistency on reads
```python
# Eventual consistency (fast)
response = table.get_item(Key={'id': '123'})

# Strong consistency (slower but guaranteed latest)
response = table.get_item(
    Key={'id': '123'},
    ConsistentRead=True
)
```

---

### Example 2: Cassandra (Tunable Quorum)

```sql
-- Write with quorum
INSERT INTO users (id, name) VALUES (1, 'Alice')
USING CONSISTENCY QUORUM;

-- Read with quorum
SELECT * FROM users WHERE id = 1
USING CONSISTENCY QUORUM;

-- Options: ONE, QUORUM, ALL
```

---

### Example 3: MongoDB

**Default:** Eventually consistent reads from secondaries
**Option:** Read from primary for strong consistency
```javascript
// Eventually consistent
db.users.find().readPref('secondary')

// Strong consistency
db.users.find().readPref('primary')
```

---

## Summary

- **Linearizability**: Strongest, behaves like single machine
- **Sequential**: Same order on all nodes
- **Causal**: Respects cause-effect relationships
- **Eventual**: High performance, eventual convergence
- **Read-Your-Writes**: See your own updates
- **Monotonic Reads**: Never go back in time
- **Quorum**: Tunable consistency (W + R > N)

**Trade-off:** Stronger consistency = Lower performance/availability

## Next Steps

Continue to **[API Design](../10-api-design/)** to learn how to expose your consistent data through well-designed APIs!

## Implementation Approaches

### Approach 1: Strong Consistency
- **Description**: All clients see same data at same time. Reads return most recent write.
- **Pros**: Simple to reason about, no stale data, predictable behavior.
- **Cons**: Higher latency, lower availability, coordination overhead.
- **When to use**: Financial systems, inventory management, critical data accuracy.

### Approach 2: Eventual Consistency
- **Description**: Data will become consistent eventually, but may be temporarily inconsistent.
- **Pros**: High availability, low latency, scales well.
- **Cons**: Complex application logic, temporary inconsistencies, harder to debug.
- **When to use**: Social media, DNS, caching, most web applications.

### Approach 3: Causal Consistency
- **Description**: Respects cause-and-effect relationships. Related operations appear in order.
- **Pros**: More intuitive than eventual, less overhead than strong.
- **Cons**: More complex to implement, requires tracking causality.
- **When to use**: Collaborative editing, messaging, comment threads.

## Trade-offs

| Aspect | Strong | Eventual |
|--------|--------|----------|
| Consistency | Always current | Temporarily stale |
| Availability | Lower | Higher |
| Latency | Higher | Lower |
| Partition Tolerance | Lower | Higher |
| Complexity | Simple app logic | Complex app logic |

## Interview Tips

**CAP Theorem**: "Choose 2 of 3: Consistency, Availability, Partition Tolerance. Partitions are inevitable, so choose CP or AP."

**Explain Patterns**: "Strong consistency for banking. Eventual for social media. Read-your-writes for user profiles."

**Quorum Reads/Writes**: "W + R > N ensures strong consistency. W=2, R=2, N=3 for balance."

**Real Examples**: "DynamoDB offers both strong and eventual reads. MongoDB has tunable consistency."

