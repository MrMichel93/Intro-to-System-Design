# Database Design

## What is Database Design?

**Database Design** is the process of organizing and structuring data so it can be stored, accessed, and managed efficiently. Think of it like organizing a library - you need to decide how to categorize books, where to place them, and how to find them quickly.

Good database design is crucial because it affects:
- **Performance**: How fast you can read/write data
- **Scalability**: How well your system handles growth
- **Data Integrity**: Keeping data accurate and consistent
- **Maintainability**: How easy it is to modify and update

## Types of Databases

### 1. SQL (Relational) Databases
**Structure**: Tables with rows and columns, relationships between tables

**Examples**: MySQL, PostgreSQL, SQL Server, Oracle

**Best for**:
- Structured data with clear relationships
- When you need ACID transactions (banking, e-commerce)
- Complex queries with joins
- Data that fits a schema

**Example:**
```
Users Table:
| ID | Name    | Email           |
|----|---------|-----------------|
| 1  | Alice   | alice@email.com |
| 2  | Bob     | bob@email.com   |

Orders Table:
| ID | UserID | Product | Price |
|----|--------|---------|-------|
| 1  | 1      | Laptop  | 1000  |
| 2  | 1      | Mouse   | 25    |
| 3  | 2      | Keyboard| 75    |
```

---

### 2. NoSQL Databases

#### A. Document Databases (MongoDB, CouchDB)
**Structure**: Store data as documents (JSON-like)

**Best for**: 
- Flexible schema
- Nested/hierarchical data
- Rapid development

**Example:**
```json
{
  "id": 1,
  "name": "Alice",
  "email": "alice@email.com",
  "orders": [
    {"product": "Laptop", "price": 1000},
    {"product": "Mouse", "price": 25}
  ]
}
```

#### B. Key-Value Stores (Redis, DynamoDB)
**Structure**: Simple key → value mapping

**Best for**:
- Caching
- Session storage
- Simple lookups

**Example:**
```
user:1 → {"name": "Alice", "email": "alice@email.com"}
session:abc123 → {"userId": 1, "loginTime": "..."}
```

#### C. Column-Family Stores (Cassandra, HBase)
**Structure**: Data organized by columns, not rows

**Best for**:
- Time-series data
- Analytics
- Write-heavy workloads

#### D. Graph Databases (Neo4j)
**Structure**: Nodes and relationships (edges)

**Best for**:
- Social networks
- Recommendation engines
- Network analysis

**Example:**
```
(Alice) -[FRIENDS_WITH]-> (Bob)
(Alice) -[LIKES]-> (Laptop)
(Bob) -[LIKES]-> (Laptop)
```

---

## SQL vs NoSQL: When to Use Each

| Aspect | SQL | NoSQL |
|--------|-----|-------|
| Schema | Fixed, predefined | Flexible, dynamic |
| Scaling | Vertical (more powerful server) | Horizontal (more servers) |
| Transactions | Strong ACID guarantees | Eventual consistency (usually) |
| Query Language | SQL (standardized) | Varies by database |
| Best Use Case | Complex queries, relationships | Flexible data, high scale |

**Real-World Examples:**
- **SQL**: Banking (need ACID), CRM systems, accounting
- **NoSQL**: Social media feeds, IoT sensor data, real-time analytics

---

## Database Design Principles

### 1. Normalization (SQL)

**Goal**: Reduce data redundancy and improve data integrity

#### Denormalized (Bad):
```
Orders Table:
| ID | CustomerName | CustomerEmail  | Product | Price |
|----|--------------|----------------|---------|-------|
| 1  | Alice        | alice@email.com| Laptop  | 1000  |
| 2  | Alice        | alice@email.com| Mouse   | 25    |
```
**Problem**: Alice's email is duplicated. If she changes email, must update multiple rows!

#### Normalized (Good):
```
Customers Table:
| ID | Name  | Email           |
|----|-------|-----------------|
| 1  | Alice | alice@email.com |

Orders Table:
| ID | CustomerID | Product | Price |
|----|------------|---------|-------|
| 1  | 1          | Laptop  | 1000  |
| 2  | 1          | Mouse   | 25    |
```
**Better**: Customer info stored once. Join tables when needed.

**Normal Forms (simplified):**
- **1NF**: No repeating groups, atomic values
- **2NF**: 1NF + no partial dependencies
- **3NF**: 2NF + no transitive dependencies

**When to Denormalize**: Sometimes you intentionally denormalize for performance (faster reads, fewer joins)

---

### 2. Indexing

**What**: Data structure that improves query speed

**Analogy**: Like an index in a textbook - instead of reading every page to find "Database", you check the index and jump to page 147.

**Without Index:**
```sql
SELECT * FROM users WHERE email = 'alice@email.com';
-- Checks every row (slow for millions of users)
```

**With Index:**
```sql
CREATE INDEX idx_email ON users(email);
SELECT * FROM users WHERE email = 'alice@email.com';
-- Uses index to jump directly to the row (fast!)
```

**Trade-offs:**
- ✅ Faster reads (SELECT)
- ❌ Slower writes (INSERT, UPDATE) - must update index
- ❌ Uses extra storage

**Best Practices:**
- Index columns used in WHERE, JOIN, ORDER BY
- Don't index everything (overhead)
- Monitor slow queries and add indexes as needed

---

### 3. Primary Keys

**Definition**: Unique identifier for each row

**Example:**
```sql
CREATE TABLE users (
    id INT PRIMARY KEY,  -- Every user has unique ID
    email VARCHAR(255),
    name VARCHAR(100)
);
```

**Why Important:**
- Ensures no duplicate records
- Used for relationships (foreign keys)
- Often auto-incrementing

**Types:**
- **Auto-increment**: 1, 2, 3, 4... (simple but predictable)
- **UUID**: Random unique ID (better for distributed systems)

---

### 4. Foreign Keys

**Definition**: Links tables together by referencing primary key in another table

**Example:**
```sql
CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    product VARCHAR(100),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**Benefits:**
- Enforces referential integrity
- Can't add order for non-existent user
- Cascading deletes (optional)

---

## Scaling Databases

### 1. Vertical Scaling
**Definition**: Upgrade to a more powerful server

**Pros**: Simple, no application changes
**Cons**: Limited, expensive, single point of failure

---

### 2. Read Replicas
**How it works**: Main database handles writes, copies (replicas) handle reads

```
        [Main DB]
        (Writes)
            ↓ (Replication)
    ┌───────┼───────┐
    ↓       ↓       ↓
[Replica 1] [Replica 2] [Replica 3]
  (Reads)    (Reads)     (Reads)
```

**Use case**: Read-heavy applications (blogs, news sites)

**Trade-off**: Replication lag (replicas might be slightly behind)

---

### 3. Sharding (Partitioning)
**Definition**: Split data across multiple databases

**Example - Sharding by User ID:**
```
Users 1-1M     → Database 1
Users 1M-2M    → Database 2
Users 2M-3M    → Database 3
```

**Strategies:**
1. **Range-based**: Divide by ranges (IDs, dates)
2. **Hash-based**: Hash user ID to determine shard
3. **Geographic**: Users by location

**Pros**: 
- Distributes load
- More storage capacity
- Better performance

**Cons**:
- Complex to implement
- Hard to query across shards
- Rebalancing is difficult

---

### 4. Database Clustering
**Definition**: Multiple database servers working together

**Benefits**: High availability, no single point of failure

---

## Database Performance Optimization

### 1. Query Optimization

**Slow Query:**
```sql
-- Fetches all columns, no index
SELECT * FROM orders 
WHERE customer_id = 123 
ORDER BY created_at DESC;
```

**Optimized Query:**
```sql
-- Only needed columns, uses index
SELECT id, product, price 
FROM orders 
WHERE customer_id = 123  -- indexed
ORDER BY created_at DESC  -- indexed
LIMIT 10;  -- Don't fetch more than needed
```

**Tips:**
- Use EXPLAIN to analyze queries
- Avoid SELECT * (fetch only needed columns)
- Use LIMIT when possible
- Index properly
- Avoid complex subqueries

---

### 2. Connection Pooling

**Problem**: Opening database connection is slow (100ms)

**Solution**: Reuse connections from a pool

```python
# Instead of:
for request in requests:
    conn = database.connect()  # Slow!
    result = conn.query(...)
    conn.close()

# Use connection pool:
pool = ConnectionPool(size=10)
for request in requests:
    conn = pool.get_connection()  # Fast! Reuses existing
    result = conn.query(...)
    pool.return_connection(conn)
```

---

### 3. Caching

**Strategy**: Cache frequently accessed data

```python
def get_user(user_id):
    # Check cache first
    user = cache.get(f"user:{user_id}")
    if user:
        return user
    
    # Cache miss - query database
    user = database.query("SELECT * FROM users WHERE id = ?", user_id)
    cache.set(f"user:{user_id}", user, ttl=3600)
    return user
```

---

## ACID Properties (SQL Transactions)

### A - Atomicity
**All or nothing**: Transaction either completes fully or not at all

**Example:**
```sql
BEGIN TRANSACTION;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```
If second UPDATE fails, first UPDATE is rolled back automatically.

### C - Consistency
**Valid state**: Database moves from one valid state to another

**Example**: Balance can't be negative (constraint enforced)

### I - Isolation
**Concurrent transactions don't interfere**: Each transaction runs as if it's alone

**Example**: Two people booking last concert ticket won't both succeed

### D - Durability
**Permanent changes**: Once committed, data survives crashes

**Example**: Bank transfer committed → power outage → money still transferred

---

## Common Database Patterns

### 1. Master-Slave Replication
```
[Master] ← Writes
    ↓ (Replication)
[Slave 1] [Slave 2] ← Reads
```

### 2. Multi-Master Replication
```
[Master 1] ↔ [Master 2]
(Both accept writes)
```
**Complex**: Need conflict resolution

### 3. Database per Service (Microservices)
```
[User Service] → [User DB]
[Order Service] → [Order DB]
[Payment Service] → [Payment DB]
```
**Pros**: Independence, scaling  
**Cons**: No cross-database joins

---

## Real-World Examples

### Instagram
- PostgreSQL for main data
- Cassandra for photo metadata
- Redis for caching
- Sharded by user ID

### Netflix
- Multiple databases for different needs
- Cassandra for viewing history
- MySQL for billing
- Caching heavily used

### Uber
- Sharded MySQL
- Geographic sharding (by city)
- Real-time data in Redis

---

## Best Practices

1. **Choose the Right Database**: SQL for structured, NoSQL for flexible
2. **Index Strategically**: Index what you query, not everything
3. **Normalize to Reduce Redundancy**: But denormalize for performance when needed
4. **Use Transactions**: For operations that must be atomic
5. **Monitor Performance**: Track slow queries, optimize them
6. **Plan for Scale**: Design with growth in mind
7. **Backup Regularly**: Always have backups and test restoring
8. **Use Connection Pooling**: Don't open/close connections frequently
9. **Cache Wisely**: Cache frequently read data
10. **Test with Real Data**: Performance with 10 rows ≠ 10 million rows

---

## Common Pitfalls

1. **No Indexes**: Queries become slow with large datasets
2. **Too Many Indexes**: Slows down writes
3. **Over-Normalization**: Too many joins hurt performance
4. **Ignoring N+1 Problem**: Making N queries in a loop
5. **No Monitoring**: Don't know what's slow
6. **Storing Files in Database**: Use object storage (S3) instead

---

## Summary

- Database design affects performance, scalability, and maintainability
- **SQL**: Structured data, ACID transactions, relationships
- **NoSQL**: Flexible schema, horizontal scaling, various models
- **Normalization**: Reduces redundancy
- **Indexing**: Speeds up reads, slows writes
- **Scaling**: Read replicas, sharding, clustering
- **Optimization**: Query tuning, connection pooling, caching
- **ACID**: Ensures transaction reliability (SQL)

---

## Next Steps

Master these concepts with the challenges, then explore **[API Design](../05-api-design/)** to learn how applications communicate!

## Implementation Approaches

### Approach 1: Relational Database (SQL)
- **Description**: Structured data with predefined schemas, ACID guarantees, SQL queries.
- **Pros**: Strong consistency, mature tooling, complex queries, joins, transactions, data integrity.
- **Cons**: Harder to scale horizontally, rigid schema, can be slower for simple key-value ops.
- **When to use**: Financial systems, complex relationships, need transactions, structured data, reporting needs.

### Approach 2: NoSQL Document Store (MongoDB, Couch DB)
- **Description**: Store data as flexible JSON-like documents.
- **Pros**: Flexible schema, easy to scale horizontally, fast for document retrieval, good for hierarchical data.
- **Cons**: No joins, eventual consistency, less mature, complex queries difficult.
- **When to use**: Content management, user profiles, product catalogs, rapid development, flexible data models.

### Approach 3: NoSQL Key-Value Store (Redis, DynamoDB)
- **Description**: Simple key-value pairs, optimized for fast reads/writes.
- **Pros**: Extremely fast, simple model, easy to scale, good for caching, session storage.
- **Cons**: Limited query capabilities, no complex data relationships, not for complex data models.
- **When to use**: Caching, session storage, real-time analytics, simple data models, high throughput needs.

### Approach 4: Polyglot Persistence
- **Description**: Use multiple database types for different needs within same application.
- **Pros**: Optimal database for each use case, flexibility, performance optimization.
- **Cons**: Operational complexity, data consistency challenges, more moving parts, higher costs.
- **When to use**: Large applications, diverse data needs, microservices, mature teams.

## Trade-offs

| Aspect | SQL | NoSQL |
|--------|-----|-------|
| Schema | Rigid, predefined | Flexible, schema-less |
| Scalability | Vertical (mainly) | Horizontal |
| Consistency | Strong (ACID) | Eventual (BASE) |
| Joins | Native support | No joins |
| Transactions | Full ACID | Limited |
| Query Language | SQL (standard) | Custom APIs |
| Use Case | Complex relations | Simple access patterns |

## Capacity Calculations

### Database Sizing Example
```
Users: 10,000,000
User data: 2 KB per user
Total: 20 GB

Orders: 50,000,000
Order data: 1 KB per order
Total: 50 GB

Products: 100,000
Product data: 10 KB per product
Total: 1 GB

Database size: 71 GB
With indexes (2x): 142 GB
With replication (3x): 426 GB
Growth buffer (30%): 550 GB
```

## Anti-Patterns

**No Indexes**: Slow queries scanning entire tables. Always index foreign keys and frequently queried columns.

**SELECT ***: Retrieving unnecessary columns wastes bandwidth. Specify needed columns explicitly.

**N+1 Queries**: Loading parent then querying for each child. Use joins or eager loading.

**No Connection Pooling**: Creating new connection per request is expensive. Reuse connections.

## Interview Tips

**SQL vs NoSQL**: Explain use cases for each. "SQL for complex transactions, NoSQL for scale and flexibility."

**Normalization**: "Reduce redundancy through normalization. Denormalize for read performance when needed."

**Indexing Strategy**: "Index frequently queried columns. Balance read speed vs write overhead."

**Scaling Approaches**: "Read replicas for read-heavy. Sharding for write-heavy. Caching to reduce database load."

