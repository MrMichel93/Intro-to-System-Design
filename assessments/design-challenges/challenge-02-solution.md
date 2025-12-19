# Design Challenge 2: News Feed - Sample Solution

## 1. Capacity Estimation

### Given
- 500 million total users
- 50 million DAU (10% daily active)
- Each user follows 200 people on average
- Maximum 10,000 posts per user

### Calculations

**Post Creation:**
- Assume each DAU creates 2 posts per day on average
- Total posts per day: 50M × 2 = 100 million posts/day
- Posts per second: 100M / 86,400 ≈ 1,157 posts/sec
- Peak (3x): ~3,500 posts/sec

**Feed Reads:**
- Assume each DAU checks feed 10 times per day
- Total feed requests: 50M × 10 = 500 million/day
- Requests per second: 500M / 86,400 ≈ 5,787 reads/sec
- Peak: ~17,400 reads/sec

**Storage:**
```
Per post: ~300 bytes (user_id, timestamp, text)
Posts per year: 100M/day × 365 = 36.5 billion posts
Storage per year: 36.5B × 300 bytes = ~11 TB/year
5-year storage: ~55 TB
```

---

## 2. API Design

### Create Post
```
POST /api/v1/posts
Authorization: Bearer {token}

Request:
{
  "content": "Hello world! This is my post."
}

Response (201):
{
  "post_id": "123456789",
  "user_id": "user_001",
  "content": "Hello world! This is my post.",
  "created_at": "2025-01-15T10:30:00Z"
}
```

### Follow User
```
POST /api/v1/users/{user_id}/follow
Authorization: Bearer {token}

Response (200):
{
  "following": true,
  "followed_user_id": "user_002"
}
```

### Get News Feed
```
GET /api/v1/feed?limit=20&cursor={next_page_token}
Authorization: Bearer {token}

Response (200):
{
  "posts": [
    {
      "post_id": "789",
      "user_id": "user_002",
      "username": "john_doe",
      "content": "Latest post...",
      "created_at": "2025-01-15T11:00:00Z"
    }
  ],
  "next_cursor": "encoded_cursor_token"
}
```

---

## 3. Database Schema

### Users Table
```sql
CREATE TABLE users (
    user_id BIGINT PRIMARY KEY,
    username VARCHAR(50) UNIQUE,
    created_at TIMESTAMP,
    follower_count INT DEFAULT 0,
    following_count INT DEFAULT 0,
    is_celebrity BOOLEAN DEFAULT FALSE  -- Users with >1M followers
);
```

### Posts Table
```sql
CREATE TABLE posts (
    post_id BIGINT PRIMARY KEY,
    user_id BIGINT,
    content VARCHAR(280),
    created_at TIMESTAMP,
    INDEX idx_user_time (user_id, created_at DESC)
);
```

### Follows Table (Graph relationship)
```sql
CREATE TABLE follows (
    follower_id BIGINT,
    followee_id BIGINT,
    created_at TIMESTAMP,
    PRIMARY KEY (follower_id, followee_id),
    INDEX idx_follower (follower_id),
    INDEX idx_followee (followee_id)
);
```

### News Feed Cache Table (Pre-computed feeds)
```sql
CREATE TABLE news_feed_cache (
    user_id BIGINT,
    post_id BIGINT,
    post_timestamp TIMESTAMP,
    PRIMARY KEY (user_id, post_timestamp, post_id)
);
```

---

## 4. High-Level Architecture

```
                    ┌──────────────┐
                    │   Clients    │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │Load Balancer │
                    └──────┬───────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
    ┌─────▼─────┐   ┌─────▼─────┐   ┌─────▼─────┐
    │   API     │   │   API     │   │   API     │
    │  Servers  │   │  Servers  │   │  Servers  │
    └─────┬─────┘   └─────┬─────┘   └─────┬─────┘
          │               │               │
          └───────┬───────┴───────┬───────┘
                  │               │
         ┌────────▼──────┐  ┌────▼─────────┐
         │  Fan-out      │  │   Feed       │
         │  Service      │  │  Service     │
         └────────┬──────┘  └────┬─────────┘
                  │               │
    ┌─────────────┼───────────────┼──────────────┐
    │             │               │              │
┌───▼───┐  ┌─────▼─────┐  ┌─────▼─────┐  ┌─────▼─────┐
│ Posts │  │  Follows  │  │   Feed    │  │   Cache   │
│  DB   │  │    DB     │  │  Cache    │  │  (Redis)  │
└───────┘  └───────────┘  └───────────┘  └───────────┘
                  │
          ┌───────▼────────┐
          │ Message Queue  │
          │   (Kafka)      │
          └────────────────┘
```

### Flow for Creating a Post:

1. User submits post via API
2. API server writes to Posts DB
3. Post event published to message queue
4. Fan-out service consumes event:
   - For regular users: Push post to all followers' feed caches
   - For celebrities: Mark that new post exists (pull on-demand)
5. Return success to user

### Flow for Reading Feed:

1. User requests feed
2. API server queries Feed Cache (Redis)
3. If cache hit: Return cached feed
4. If cache miss: 
   - Query follows to get list of followees
   - Fetch recent posts from those users
   - Merge and sort by timestamp
   - Cache result
5. Return feed to user

---

## 5. Feed Generation Strategy

### Hybrid Approach: Fan-out on Write + Fan-out on Read

**For Regular Users (< 1 million followers):**
- **Fan-out on Write**: When user posts, push to all followers' pre-computed feeds
- Pros: Fast feed reads (already computed)
- Cons: Slower post creation, storage overhead

**For Celebrity Users (> 1 million followers):**
- **Fan-out on Read**: Don't pre-compute feeds; fetch celebrity posts on-demand when feed is read
- Pros: Fast post creation, no storage overhead
- Cons: Slightly slower feed reads (but acceptable)

### Implementation Details:

**1. Feed Cache Structure (Redis):**
```
Key: feed:{user_id}
Value: Sorted Set (score = timestamp)
  - member: post_id
  - score: post_timestamp

ZADD feed:user_123 1642234567 "post_789"
ZRANGE feed:user_123 0 19 REV  # Get 20 most recent
```

**2. Fan-out Service Algorithm:**
```
When user_X posts:
  1. Check if user_X is celebrity (follower_count > 1M)
  2. If NOT celebrity:
     - Query followers of user_X
     - For each follower:
       - Add post_id to their feed cache (sorted by timestamp)
       - Trim to keep only latest 1000 posts
  3. If celebrity:
     - Skip fan-out (will be fetched on-demand)
```

**3. Feed Generation Algorithm:**
```
When user_Y requests feed:
  1. Fetch from feed cache (pre-computed)
  2. Identify celebrities user_Y follows
  3. Fetch recent posts from those celebrities
  4. Merge cached feed + celebrity posts
  5. Sort by timestamp (descending)
  6. Return top 20 posts
  7. Cache merged result for 1 minute
```

### Handling Timeline Freshness:

**Cache TTL Strategy:**
- Pre-computed feeds: No TTL (updated on write)
- Merged feeds (with celebrity posts): 1 minute TTL
- Celebrity posts: 5 minute TTL

**Message Queue for Async Processing:**
- Use Kafka for fan-out events
- Multiple consumer groups for parallel processing
- Partitioned by user_id for ordered processing
- Dead letter queue for failed fan-outs

---

## 6. Trade-offs and Optimizations

### Key Design Decisions:

**1. Hybrid Fan-out Strategy:**
- **Decision**: Fan-out on write for regular users, fan-out on read for celebrities
- **Trade-off**: Complexity vs performance
- **Justification**: Optimizes for 99% of users while handling celebrity edge cases

**2. Feed Cache Storage:**
- **Decision**: Store post IDs in sorted sets, not full post content
- **Trade-off**: Extra lookup needed for post details vs storage efficiency
- **Justification**: More cache-efficient; post details fetched in batch

**3. Eventual Consistency:**
- **Decision**: Accept up to 1 minute delay for post visibility
- **Trade-off**: Consistency vs availability
- **Justification**: Users don't expect real-time; 1 minute is acceptable for social feed

### Potential Bottlenecks:

1. **Fan-out Service**: Writing to millions of caches
   - Solution: Async processing with message queue
2. **Feed Cache Storage**: Large memory requirement
   - Solution: LRU eviction, store only recent posts
3. **Celebrity Post Fetches**: Repeatedly fetching same celebrity posts
   - Solution: Aggressive caching with short TTL

### Additional Optimizations:

**1. Pagination:**
- Use cursor-based pagination (not offset-based)
- Cursor encodes last post timestamp and ID
- More efficient than OFFSET for large datasets

**2. Connection Warmup:**
- Pre-fetch first page of feed when user logs in
- Predictive pre-loading based on user patterns

**3. Feed Ranking (Future):**
- Currently chronological
- Could add ML-based ranking for engagement
- Score based on: recency, user affinity, post engagement

---

## Summary

This design handles:
- ✅ 500M users with 50M DAU
- ✅ Fast feed generation (< 200ms)
- ✅ Celebrity users with millions of followers
- ✅ Scalable fan-out architecture
- ✅ Efficient storage and caching

The hybrid approach balances write and read performance while remaining cost-effective and scalable.
