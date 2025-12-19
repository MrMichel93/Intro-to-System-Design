# Database Design - Challenges & Questions

Test your understanding of database design concepts!

## 🎯 Multiple Choice Questions

### Question 1
Which database would be BEST for a banking application?

A) MongoDB (Document)  
B) Redis (Key-Value)  
C) PostgreSQL (SQL)  
D) Neo4j (Graph)  

<details>
<summary>Answer</summary>

**C) PostgreSQL (SQL)**

Banking requires ACID transactions to ensure money transfers are atomic (all-or-nothing) and consistent. SQL databases excel at this. NoSQL databases prioritize availability and partition tolerance over strong consistency.
</details>

---

### Question 2
What is the main purpose of database indexing?

A) To compress data  
B) To speed up read queries  
C) To backup data  
D) To encrypt sensitive information  

<details>
<summary>Answer</summary>

**B) To speed up read queries**

Indexes create a data structure that allows the database to find rows quickly without scanning the entire table, like an index in a book helps you find topics without reading every page.
</details>

---

### Question 3
You have a table with 10 million users and query `SELECT * FROM users WHERE email = 'test@example.com'` is slow. What should you do FIRST?

A) Switch to NoSQL  
B) Add an index on the email column  
C) Buy a bigger server  
D) Delete old users  

<details>
<summary>Answer</summary>

**B) Add an index on the email column**

Since you're querying by email, an index on the email column will dramatically speed up this query. It's the simplest and most effective solution.

```sql
CREATE INDEX idx_email ON users(email);
```
</details>

---

### Question 4
What is database sharding?

A) Making database backups  
B) Splitting data across multiple database servers  
C) Compressing database files  
D) Encrypting sensitive data  

<details>
<summary>Answer</summary>

**B) Splitting data across multiple database servers**

Sharding (or partitioning) distributes data across multiple databases to handle more data and traffic than a single database could manage. For example, users 1-1M on DB1, users 1M-2M on DB2, etc.
</details>

---

### Question 5
In a normalized database, where should customer email address be stored if customers can place multiple orders?

A) In the orders table with each order  
B) In a separate customers table referenced by orders  
C) In both tables for faster access  
D) In a text file  

<details>
<summary>Answer</summary>

**B) In a separate customers table referenced by orders**

Normalization says store data in one place. Customer email belongs in a customers table. Orders table has a customer_id foreign key pointing to the customer. This prevents data duplication and update anomalies.
</details>

---

### Question 6
What does ACID stand for in database transactions?

A) Access, Control, Input, Data  
B) Atomicity, Consistency, Isolation, Durability  
C) Automated, Controlled, Indexed, Distributed  
D) Available, Consistent, Isolated, Durable  

<details>
<summary>Answer</summary>

**B) Atomicity, Consistency, Isolation, Durability**

ACID guarantees that database transactions are processed reliably:
- **Atomicity**: All or nothing
- **Consistency**: Valid state to valid state
- **Isolation**: Transactions don't interfere
- **Durability**: Changes persist after commit
</details>

---

## 🧩 Scenario-Based Challenges

### Challenge 1: Database Selection

Choose the best database for each application and explain why:

**Application A**: Real-time chat application
- Requirements: Fast reads/writes, user presence, message history
- Expected scale: 100K concurrent users
- Your choice: _______________
- Why: _______________

**Application B**: E-commerce product catalog
- Requirements: Product details, categories, inventory, orders
- Need: Complex queries, transactions for orders
- Your choice: _______________
- Why: _______________

**Application C**: Social network
- Requirements: Users, friendships, posts, likes, comments
- Need: Friend recommendations, "friends of friends"
- Your choice: _______________
- Why: _______________

<details>
<summary>Sample Answers</summary>

**Application A: Real-time Chat**
**Choice**: Redis (with PostgreSQL backup)
- **Redis**: In-memory storage for real-time messages and presence
  - Sub-millisecond latency
  - Pub/sub for real-time updates
  - Keep recent messages (last 24 hours)
- **PostgreSQL**: Permanent message storage
  - Store full message history
  - Search functionality
  - User metadata

**Application B: E-commerce**
**Choice**: PostgreSQL (or MySQL)
- Need ACID transactions for orders (money involved!)
- Complex queries (filter by category, price, rating)
- Relationships between products, orders, customers, inventory
- Strong consistency required for inventory (prevent overselling)

**Application C: Social Network**
**Choice**: 
- **Neo4j** (Graph DB) for relationships and recommendations
  - Model friendships as graph edges
  - Efficient "friends of friends" queries
  - Friend recommendations based on mutual connections
- **PostgreSQL** for user profiles and posts
  - Structured user data
  - Posts and comments with transactions
- **Redis** for feeds and caching
  - Cache user timelines
  - Fast access to recent posts

**Or**: MongoDB for simpler implementation if complex graph queries not needed
</details>

---

### Challenge 2: Schema Design

Design a database schema for a library management system:

**Requirements:**
- Books (title, ISBN, author, publication year)
- Authors (can write multiple books)
- Members (can borrow multiple books)
- Borrowing records (who borrowed what, when, return date)

**Your Task:**
1. Design the tables
2. Identify primary keys
3. Identify foreign keys
4. What indexes would you create?

<details>
<summary>Sample Solution</summary>

**Schema Design:**

```sql
-- Authors Table
CREATE TABLE authors (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    birth_year INT,
    country VARCHAR(100)
);
CREATE INDEX idx_author_name ON authors(name);

-- Books Table
CREATE TABLE books (
    id INT PRIMARY KEY AUTO_INCREMENT,
    isbn VARCHAR(13) UNIQUE NOT NULL,
    title VARCHAR(500) NOT NULL,
    author_id INT NOT NULL,
    publication_year INT,
    total_copies INT DEFAULT 1,
    available_copies INT DEFAULT 1,
    FOREIGN KEY (author_id) REFERENCES authors(id)
);
CREATE INDEX idx_book_title ON books(title);
CREATE INDEX idx_book_author ON books(author_id);
CREATE UNIQUE INDEX idx_isbn ON books(isbn);

-- Members Table
CREATE TABLE members (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    join_date DATE NOT NULL,
    membership_status ENUM('active', 'suspended') DEFAULT 'active'
);
CREATE INDEX idx_member_email ON members(email);

-- Borrowing Records Table
CREATE TABLE borrowings (
    id INT PRIMARY KEY AUTO_INCREMENT,
    book_id INT NOT NULL,
    member_id INT NOT NULL,
    borrow_date DATE NOT NULL,
    due_date DATE NOT NULL,
    return_date DATE,
    status ENUM('borrowed', 'returned', 'overdue') DEFAULT 'borrowed',
    FOREIGN KEY (book_id) REFERENCES books(id),
    FOREIGN KEY (member_id) REFERENCES members(id)
);
CREATE INDEX idx_borrowing_member ON borrowings(member_id);
CREATE INDEX idx_borrowing_book ON borrowings(book_id);
CREATE INDEX idx_borrowing_status ON borrowings(status);
CREATE INDEX idx_borrowing_due_date ON borrowings(due_date);
```

**Why These Indexes?**
1. `idx_author_name`: Search books by author name
2. `idx_book_title`: Search books by title
3. `idx_book_author`: Join books with authors
4. `idx_isbn`: Unique lookup by ISBN
5. `idx_member_email`: Login/search by email
6. `idx_borrowing_member`: Find all books borrowed by a member
7. `idx_borrowing_book`: Find who borrowed a specific book
8. `idx_borrowing_status`: Find all overdue books
9. `idx_borrowing_due_date`: Check for books due soon

**Sample Queries:**

```sql
-- Find all books by an author
SELECT b.* FROM books b
JOIN authors a ON b.author_id = a.id
WHERE a.name = 'J.K. Rowling';

-- Check if book is available
SELECT available_copies FROM books WHERE isbn = '9780439139595';

-- Find all currently borrowed books
SELECT m.name, b.title, br.due_date
FROM borrowings br
JOIN members m ON br.member_id = m.id
JOIN books b ON br.book_id = b.id
WHERE br.status = 'borrowed';

-- Find overdue books
SELECT m.name, m.email, b.title, br.due_date
FROM borrowings br
JOIN members m ON br.member_id = m.id
JOIN books b ON br.book_id = b.id
WHERE br.status = 'borrowed' AND br.due_date < CURRENT_DATE;
```
</details>

---

### Challenge 3: Scaling Strategy

Your social media app is growing rapidly:

**Current Situation:**
- Single PostgreSQL database
- 100K users, growing to 10M in 6 months
- 1M posts, adding 100K posts/day
- Database CPU at 80% during peak hours
- Slow queries for user feeds

**Questions:**
1. What's the first optimization you'd make?
2. How would you scale reads?
3. How would you scale writes?
4. Should you shard? If yes, how?

<details>
<summary>Sample Solution</summary>

**1. First Optimization (Quick Wins):**

**A. Add Indexes**
```sql
-- Assuming feed query: "Get recent posts from friends"
CREATE INDEX idx_posts_user_created ON posts(user_id, created_at DESC);
CREATE INDEX idx_friendships_user ON friendships(user_id);
```

**B. Implement Caching (Redis)**
```python
# Cache user feeds
def get_user_feed(user_id):
    # Check cache
    feed = redis.get(f"feed:{user_id}")
    if feed:
        return feed
    
    # Generate feed from database
    feed = database.query("""
        SELECT posts.* FROM posts
        JOIN friendships ON posts.user_id = friendships.friend_id
        WHERE friendships.user_id = ?
        ORDER BY posts.created_at DESC
        LIMIT 50
    """, user_id)
    
    # Cache for 5 minutes
    redis.setex(f"feed:{user_id}", 300, feed)
    return feed
```

**C. Query Optimization**
- Use EXPLAIN to find slow queries
- Avoid SELECT *, fetch only needed columns
- Add LIMIT to feed queries

**2. Scale Reads (0-6 months):**

**Read Replicas**
```
[Primary DB] ← Writes (Posts, Likes, Comments)
    ↓ (Replication)
┌───┼───┼───┐
↓   ↓   ↓   ↓
[R1][R2][R3][R4] ← Reads (Feeds, Profiles)
```

**Strategy:**
- 1 primary for writes
- 3-5 replicas for reads
- Route feed queries to replicas
- Cache layer (Redis) reduces database hits by 80%

**3. Scale Writes (6+ months, 10M users):**

**A. Separate Database by Function**
```
[User DB] - User profiles, authentication
[Post DB] - Posts, comments
[Social Graph DB] - Friendships, follows
[Analytics DB] - Metrics, reporting (can be delayed)
```

**B. Async Processing**
```python
# Don't block user waiting for all processing
def create_post(user_id, content):
    # Quick: Save post
    post_id = database.insert_post(user_id, content)
    
    # Async: Update followers' feeds
    queue.enqueue('update_feeds', post_id, user_id)
    
    return post_id
```

**4. Sharding Strategy (10M+ users):**

**Yes, shard! Shard by User ID:**

```
Users 0-2.5M     → Shard 1
Users 2.5M-5M    → Shard 2
Users 5M-7.5M    → Shard 3
Users 7.5M-10M   → Shard 4
```

**Implementation:**
```python
def get_shard(user_id):
    # Simple hash-based sharding
    return user_id % NUM_SHARDS

def get_user_posts(user_id):
    shard = get_shard(user_id)
    return shards[shard].query("SELECT * FROM posts WHERE user_id = ?", user_id)
```

**Challenges:**
- **Cross-shard queries**: If users on different shards are friends
  - **Solution**: Cache social graph in Redis
  - Denormalize: Store friend posts in user's shard
  
- **Rebalancing**: Adding new shards
  - Use consistent hashing
  - Plan for 2x growth

**Complete Architecture (10M users):**
```
[Load Balancer]
       ↓
[App Servers] ← [Redis Cache] (Feeds, Sessions)
       ↓
[Database Router]
       ↓
┌──────┼──────┼──────┐
↓      ↓      ↓      ↓
[Shard 1] [Shard 2] [Shard 3] [Shard 4]
    ↓         ↓         ↓         ↓
[Replica] [Replica] [Replica] [Replica]
```

**Timeline:**
- **Month 0-3**: Optimize queries, add caching
- **Month 3-6**: Add read replicas
- **Month 6-12**: Separate databases by function
- **Month 12+**: Implement sharding
</details>

---

### Challenge 4: Transaction Design

You're building a ticket booking system. Design the database transaction for booking a ticket:

**Requirements:**
- Check if tickets are available
- Reserve the ticket
- Update available count
- Create booking record
- Charge payment
- If payment fails, rollback everything

**Your Task:**
Write the transaction logic (pseudocode or SQL)

<details>
<summary>Sample Solution</summary>

**SQL Transaction:**

```sql
BEGIN TRANSACTION;

-- 1. Lock the event row to prevent concurrent bookings
SELECT available_tickets 
FROM events 
WHERE event_id = 123 
FOR UPDATE;  -- Row-level lock

-- 2. Check availability
SET @available = (SELECT available_tickets FROM events WHERE event_id = 123);

IF @available < 1 THEN
    ROLLBACK;
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'No tickets available';
END IF;

-- 3. Create booking record
INSERT INTO bookings (user_id, event_id, ticket_count, status, created_at)
VALUES (456, 123, 1, 'pending', NOW());

SET @booking_id = LAST_INSERT_ID();

-- 4. Update available tickets
UPDATE events 
SET available_tickets = available_tickets - 1 
WHERE event_id = 123;

-- 5. Process payment (in application code)
-- payment_result = payment_gateway.charge(user_id, amount)

-- 6A. If payment succeeds
UPDATE bookings 
SET status = 'confirmed', payment_id = @payment_id
WHERE booking_id = @booking_id;

COMMIT;

-- 6B. If payment fails (in exception handler)
ROLLBACK;
```

**Application Code (Python):**

```python
def book_ticket(user_id, event_id):
    connection = db.get_connection()
    
    try:
        connection.begin_transaction()
        
        # 1. Check and lock
        cursor = connection.execute("""
            SELECT available_tickets 
            FROM events 
            WHERE event_id = ? 
            FOR UPDATE
        """, [event_id])
        
        available = cursor.fetchone()['available_tickets']
        
        if available < 1:
            connection.rollback()
            return {"error": "No tickets available"}
        
        # 2. Create booking
        cursor = connection.execute("""
            INSERT INTO bookings (user_id, event_id, status, created_at)
            VALUES (?, ?, 'pending', NOW())
        """, [user_id, event_id])
        
        booking_id = cursor.lastrowid
        
        # 3. Decrease available tickets
        connection.execute("""
            UPDATE events 
            SET available_tickets = available_tickets - 1
            WHERE event_id = ?
        """, [event_id])
        
        # 4. Process payment (external service)
        try:
            payment_result = payment_gateway.charge(
                user_id=user_id,
                amount=get_ticket_price(event_id)
            )
        except PaymentError as e:
            connection.rollback()
            return {"error": "Payment failed"}
        
        # 5. Confirm booking
        connection.execute("""
            UPDATE bookings 
            SET status = 'confirmed', payment_id = ?
            WHERE booking_id = ?
        """, [payment_result['id'], booking_id])
        
        # All good - commit!
        connection.commit()
        
        return {
            "success": True,
            "booking_id": booking_id
        }
        
    except Exception as e:
        connection.rollback()
        log_error(e)
        return {"error": "Booking failed"}
    
    finally:
        connection.close()
```

**Key Points:**

1. **FOR UPDATE**: Locks the row so concurrent requests wait
2. **Transaction**: All-or-nothing - either everything succeeds or everything rolls back
3. **Order matters**: Lock first, then modify
4. **Error handling**: Always rollback on failure
5. **Timeout**: Set transaction timeout to prevent deadlocks

**Alternative: Optimistic Locking**

```python
def book_ticket_optimistic(user_id, event_id):
    # Read current version
    event = db.query("SELECT available_tickets, version FROM events WHERE event_id = ?", [event_id])
    
    if event['available_tickets'] < 1:
        return {"error": "No tickets available"}
    
    # Try to update with version check
    result = db.execute("""
        UPDATE events 
        SET available_tickets = available_tickets - 1,
            version = version + 1
        WHERE event_id = ? AND version = ?
    """, [event_id, event['version']])
    
    if result.rows_affected == 0:
        # Someone else updated - retry
        return book_ticket_optimistic(user_id, event_id)
    
    # Continue with booking...
```

**When to use each:**
- **Pessimistic (FOR UPDATE)**: High contention (popular events)
- **Optimistic**: Low contention (less popular events)
</details>

---

## 🔧 Practical Exercise

### Exercise 1: N+1 Query Problem

**Problem:** This code has the N+1 query issue:

```python
# Get all posts
posts = db.query("SELECT * FROM posts LIMIT 10")

# For each post, get author name
for post in posts:
    author = db.query("SELECT name FROM users WHERE id = ?", [post.user_id])
    print(f"{post.title} by {author.name}")

# Result: 11 queries! (1 for posts + 10 for authors)
```

**Your Task:** Fix this to use only 2 queries.

<details>
<summary>Solution</summary>

**Solution 1: JOIN**
```python
# Single query with JOIN
results = db.query("""
    SELECT posts.*, users.name as author_name
    FROM posts
    JOIN users ON posts.user_id = users.id
    LIMIT 10
""")

for row in results:
    print(f"{row.title} by {row.author_name}")

# Result: 1 query!
```

**Solution 2: Eager Loading**
```python
# Get all posts
posts = db.query("SELECT * FROM posts LIMIT 10")

# Get all authors in one query
user_ids = [post.user_id for post in posts]
authors = db.query("""
    SELECT id, name FROM users WHERE id IN (?, ?, ?, ...)
""", user_ids)

# Create author lookup
author_map = {author.id: author for author in authors}

# Print
for post in posts:
    author = author_map[post.user_id]
    print(f"{post.title} by {author.name}")

# Result: 2 queries!
```
</details>

---

### Exercise 2: Design a Schema

Design a database schema for a music streaming service like Spotify:

**Requirements:**
- Songs (title, duration, file URL)
- Artists (can have multiple songs)
- Albums (contain multiple songs by same artist)
- Playlists (users can create playlists with any songs)
- Users (can follow artists, create playlists)

**Consider:**
- What tables do you need?
- What are the relationships?
- What indexes would improve performance?

---

## 🤔 Critical Thinking Questions

1. **When would you choose to denormalize a database intentionally?**
   (Hint: Think about read vs write performance)

2. **Why can't you just add indexes on every column?**
   (Consider trade-offs)

3. **How would you design a database for a global application with users worldwide?**
   (Think about latency, regulations like GDPR)

4. **What happens if two transactions try to update the same row simultaneously?**
   (Think about locking, isolation levels)

---

## Next Steps

Excellent work on database design! Next, learn about **[API Design](../05-api-design/)** to understand how applications communicate with databases and each other!
