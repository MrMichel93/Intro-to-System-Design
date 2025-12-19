# Data Partitioning (Sharding)

## What is Data Partitioning?

**Data partitioning** (also called **sharding**) is the process of splitting a large database into smaller, more manageable pieces called **partitions** or **shards**. Each shard is a separate database that holds a subset of the total data.

**Why do we need it?**
- Single database can't handle billions of records
- Need to distribute load across multiple machines
- Improve query performance
- Enable horizontal scaling

**Simple Analogy**: Instead of one huge library, create multiple smaller libraries organized by topic or author.

## When to Partition?

### Signs You Need Partitioning:
- ✅ Database size > 1 TB
- ✅ Query performance degrading despite indexes
- ✅ Single server can't handle traffic
- ✅ Backup/restore takes too long
- ✅ Need to scale horizontally

### When NOT to Partition (Yet):
- ❌ Database < 100 GB
- ❌ Single server handles load fine
- ❌ Vertical scaling still an option
- ❌ Small team (adds complexity)

**Rule of Thumb**: Don't partition until you need to!

## Partitioning Strategies

### 1. Horizontal Partitioning (Sharding)
**Split rows across multiple databases**

**Example: User Database**
```
Users Table (10 million users)

Shard 1: user_id 1-2,500,000
Shard 2: user_id 2,500,001-5,000,000
Shard 3: user_id 5,000,001-7,500,000
Shard 4: user_id 7,500,001-10,000,000
```

Each shard has same schema, different data.

---

### 2. Vertical Partitioning
**Split columns across databases**

**Example: User Profile**
```
User Basic Info (Shard A):
- user_id
- name
- email

User Extended Info (Shard B):
- user_id
- bio
- preferences
- profile_picture
```

Separate frequently accessed data from rarely accessed data.

---

### 3. Functional Partitioning
**Split by function/feature**

**Example: E-commerce**
```
Products Database
Orders Database
Users Database
Reviews Database
```

Each function gets its own database.

## Partition Key Selection

**The most critical decision in sharding!**

### Good Partition Keys:
- **Evenly distributed** (no hot spots)
- **Predictable** (know which shard to query)
- **Immutable** (doesn't change)
- **High cardinality** (many unique values)

### Common Partition Keys:

**1. User ID**
```
Good for: User data, profiles, settings
Why: Even distribution, users don't change IDs
Shard = user_id % num_shards
```

**2. Geographic Location**
```
Good for: Location-based apps
Why: Query local data, regulatory compliance
Shard by country/region
```

**3. Time/Date**
```
Good for: Time-series data, logs
Why: Recent data accessed more
Shard by month or year
```

**4. Hash of Key**
```
Good for: Even distribution
Why: Prevents hot spots
Shard = hash(key) % num_shards
```

## Partition Methods Explained

### Method 1: Range-Based Partitioning

**Divide data by ranges**

**Example: User IDs**
```
Shard 1: 0-999,999
Shard 2: 1,000,000-1,999,999
Shard 3: 2,000,000-2,999,999
```

**Pros:**
- ✅ Simple to implement
- ✅ Range queries efficient
- ✅ Easy to add new ranges

**Cons:**
- ❌ Can create hot spots
- ❌ Uneven distribution possible

**Use Case:** Time-series data, sequential IDs

---

### Method 2: Hash-Based Partitioning

**Use hash function to determine shard**

```python
def get_shard(user_id, num_shards):
    return hash(user_id) % num_shards

# Examples:
user_123 → hash(123) % 4 = 3 → Shard 3
user_456 → hash(456) % 4 = 0 → Shard 0
```

**Pros:**
- ✅ Even distribution
- ✅ No hot spots
- ✅ Predictable placement

**Cons:**
- ❌ Range queries hard
- ❌ Rebalancing difficult (changing num_shards)

**Use Case:** User profiles, key-value stores

---

### Method 3: List-Based Partitioning

**Explicitly assign values to shards**

**Example: By Country**
```
Shard 1 (US): USA, Canada, Mexico
Shard 2 (EU): UK, France, Germany
Shard 3 (ASIA): China, Japan, India
```

**Pros:**
- ✅ Full control over distribution
- ✅ Regulatory compliance easy
- ✅ Optimize by usage patterns

**Cons:**
- ❌ Manual management
- ❌ Can become imbalanced

**Use Case:** Multi-tenant apps, geographic distribution

---

### Method 4: Consistent Hashing

**Advanced: Minimize rebalancing when shards added**

```
Concept: Virtual nodes on a ring
- Each shard manages multiple points on ring
- Keys hashed to ring position
- Assigned to next shard clockwise
```

**Pros:**
- ✅ Adding/removing shards affects few keys
- ✅ Flexible rebalancing
- ✅ Used by Cassandra, DynamoDB

**Cons:**
- ❌ More complex
- ❌ Harder to debug

**Use Case:** Dynamic cluster sizes, cloud databases

## Challenges in Partitioning

### Challenge 1: Cross-Shard Queries

**Problem:**
```sql
SELECT * FROM users WHERE country = 'USA' AND age > 25
```
If partitioned by user_id, must query ALL shards!

**Solutions:**
- Denormalize data (duplicate across shards)
- Secondary indexes (performance trade-off)
- Application-level joins
- Accept slower queries

---

### Challenge 2: Rebalancing

**Problem:** Shard becomes too large or hot

**Solutions:**

**A. Split Hot Shard**
```
Shard 2 (overloaded):
→ Split into Shard 2A and Shard 2B
→ Migrate half the data
```

**B. Add More Shards**
```
4 shards → 8 shards
Must rehash and move data
(This is expensive!)
```

---

### Challenge 3: Hotspots

**Problem:** One shard gets disproportionate traffic

**Example:**
```
Celebrity user with 100M followers
→ All reads go to their shard
→ That shard overloaded
```

**Solutions:**
- Cache heavily for hot data
- Replicate hot data across shards
- Use celebrity user detection + special handling

---

### Challenge 4: Transactions

**Problem:** ACID transactions across shards are hard

**Example:**
```
Transfer money from User A (Shard 1) to User B (Shard 2)
→ Both must succeed or both fail
→ Distributed transaction needed
```

**Solutions:**
- Avoid cross-shard transactions (design around them)
- Two-phase commit (slow, complex)
- Eventual consistency (accept temporary inconsistency)
- Saga pattern (application-level)

---

### Challenge 5: Joins

**Problem:** JOIN across shards is expensive

```sql
-- Users in Shard 1, Orders in Shard 2
SELECT users.name, orders.total
FROM users JOIN orders ON users.id = orders.user_id
```

**Solutions:**
- Denormalize (duplicate data)
- Application-level joins
- Partition related data together
- Accept slower performance

## Real-World Examples

### Example 1: Instagram Photos

**Strategy:** Shard by user_id

```
Why:
- Users' photos naturally grouped
- Queries mostly for single user
- Even distribution of users

Shard calculation:
shard = user_id % 4096

Result:
- Each shard ~500K users
- Queries hit single shard
- Scales horizontally
```

---

### Example 2: Uber Trips

**Strategy:** Shard by geographic region

```
Why:
- Trips are location-based
- Drivers/riders in same region
- Query by location common

Partition:
- San Francisco → Shard 1
- New York → Shard 2
- London → Shard 3

Result:
- Fast location queries
- Scales per city
- Regulatory compliance easy
```

---

### Example 3: Time-Series Logs

**Strategy:** Shard by time

```
Why:
- Recent logs accessed most
- Old logs can be archived
- Queries usually by time range

Partition:
- 2024-01 → Shard A
- 2024-02 → Shard B
- 2024-03 → Shard C

Result:
- Hot shard for current month
- Cold shards for history
- Easy to archive old data
```

## Best Practices

### 1. Choose Partition Key Carefully
- Analyze query patterns
- Test with production-like data
- Hard to change later!

### 2. Start with Fewer Shards
- Over-sharding adds complexity
- Start with 4-8 shards
- Add more as needed

### 3. Plan for Growth
- Leave room in partition key space
- Use hash functions that support growth
- Document resharding strategy

### 4. Monitor Shard Health
- Watch shard sizes
- Monitor query distribution
- Detect hotspots early

### 5. Keep Related Data Together
- Co-locate data that's queried together
- Reduces cross-shard queries
- Better performance

### 6. Denormalize When Needed
- Duplicate data to avoid joins
- Trade storage for performance
- Keep denormalized data synced

## Implementation Example

### Simple Sharding in Python

```python
class ShardManager:
    def __init__(self, num_shards=4):
        self.num_shards = num_shards
        self.shards = {
            i: Database(f"shard_{i}")
            for i in range(num_shards)
        }
    
    def get_shard(self, key):
        """Determine which shard for this key"""
        shard_id = hash(key) % self.num_shards
        return self.shards[shard_id]
    
    def get(self, key):
        """Get value from appropriate shard"""
        shard = self.get_shard(key)
        return shard.get(key)
    
    def put(self, key, value):
        """Put value in appropriate shard"""
        shard = self.get_shard(key)
        return shard.put(key, value)
    
    def query_all_shards(self, query):
        """Query all shards and combine results"""
        results = []
        for shard in self.shards.values():
            results.extend(shard.query(query))
        return results
```

## Summary

- **Partitioning** splits large databases into manageable pieces
- **Choose partition key** based on query patterns
- **Methods**: Range, Hash, List, Consistent Hashing
- **Challenges**: Cross-shard queries, rebalancing, hotspots
- **Best practice**: Don't partition until you need to
- **Goal**: Horizontal scalability while maintaining performance

## Next Steps

Continue to **[Replication](../08-replication/)** to learn how to create copies of your partitioned data for reliability!

## Interview Tips

**Explain Partitioning Purpose**: "Splits large datasets across multiple databases for horizontal scalability."

**Partition Key Selection**: "Choose keys that distribute data evenly. User ID works well as users are evenly distributed."

**Address Cross-Shard Queries**: "Avoid when possible through denormalization. When needed, query all shards and merge results."

**Rebalancing Strategy**: "Use consistent hashing to minimize data movement. Plan for 2x current capacity."

**Real Examples**: "Instagram shards by user_id. Uber shards by geographic region. Twitter uses snowflake IDs."

**Calculate Shard Count**: "With 100M users and 1M per shard, need 100 shards. Add buffer for growth: 150 shards."

