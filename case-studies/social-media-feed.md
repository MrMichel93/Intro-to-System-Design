# Case Study: Social Media Feed (Intermediate)

## Problem Statement

Design a social media feed system similar to Twitter/Facebook Timeline that allows users to post content, follow other users, and view a personalized feed of posts from people they follow. The feed should be fast, scalable, and handle both regular users and celebrities with millions of followers.

## Requirements Analysis

### Functional Requirements

**Core Features:**
1. **Post Creation**: Users can create posts (text, images, videos)
2. **Follow/Unfollow**: Users can follow and unfollow other users
3. **Feed Generation**: Users see a timeline of posts from people they follow
4. **Interactions**: Users can like, comment, and share posts
5. **Notifications**: Users receive notifications for interactions
6. **Search**: Search for users and posts by keywords/hashtags

**Feed Characteristics:**
- Posts appear in reverse chronological order (newest first)
- Feed should load quickly even for users following thousands
- Real-time or near real-time updates preferred
- Support pagination for infinite scroll

### Non-Functional Requirements

**Scale Requirements:**
- **Users**: 500 million monthly active users (MAU)
  - 200 million daily active users (DAU)
  - Average 200 following per user
- **Posts**: 400 million posts per day
  - ~4,600 posts/second average
  - ~15,000 posts/second peak
- **Feed Reads**: 50 billion feed loads per day
  - ~580,000 reads/second average
  - ~1.5 million reads/second peak
- **Read-to-Write Ratio**: 100:1 (read-heavy workload)

**Performance Requirements:**
- **Feed Load Time**: < 200ms for p95
- **Post Creation**: < 500ms for p95
- **Feed Freshness**: New posts appear within 5 seconds
- **Availability**: 99.95% uptime

**Data Characteristics:**
- Average post size: 500 bytes (text + metadata)
- 20% of posts include images (average 200KB)
- 2% of posts include videos (average 10MB)
- User follows: 80 billion relationships (200M users × 200 avg + 100M × 1000)

## Capacity Estimation

### Storage Calculation

```
**Posts Storage (per year):**
Posts per day: 400M
Posts per year: 146B

Text posts (80%): 146B × 0.8 × 500 bytes = 58.4 TB
Posts with images (18%): 146B × 0.18 × 200 KB = 5.25 PB
Posts with videos (2%): 146B × 0.02 × 10 MB = 29.2 PB

Total per year: ~34.5 PB
5-year storage: ~173 PB (with compression: ~100 PB)

**Timeline Cache Storage:**
200M DAU × 800 posts per timeline × 8 bytes = 1.28 TB
(Stores only post IDs in cache)

**Social Graph Storage:**
500M users × 200 following average = 100B relationships
100B × 16 bytes (follower_id, followed_id) = 1.6 TB
```

### Traffic Estimation

```
**Write Traffic:**
Posts: 4,600/sec average, 15,000/sec peak
Likes: 50,000/sec average
Comments: 10,000/sec average
Follows: 500/sec average

**Read Traffic:**
Timeline loads: 580,000/sec average, 1.5M/sec peak
Post details: 100,000/sec average
User profiles: 50,000/sec average
```

### Bandwidth Requirements

```
**Ingress:**
Post creation: 15,000/sec × 500 bytes = 7.5 MB/sec
Images: 15,000 × 0.18 × 200 KB = 540 MB/sec
Videos: 15,000 × 0.02 × 10 MB = 3 GB/sec
Total: ~3.5 GB/sec peak

**Egress:**
Timeline feeds: 1.5M/sec × 20 posts × 500 bytes = 15 GB/sec
Images (CDN): 50 GB/sec
Videos (CDN): 200 GB/sec
Total: ~265 GB/sec peak (mostly CDN)
```

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           CLIENT LAYER                                   │
│     [Web Apps]  [iOS Apps]  [Android Apps]  [Third-party Clients]      │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │
┌────────────────────────────────▼────────────────────────────────────────┐
│                    API GATEWAY / LOAD BALANCER                           │
│              Request routing, Rate limiting, Authentication              │
└────────────────────┬───────────────────────┬────────────────────────────┘
                     │                       │
        ┌────────────▼──────────┐   ┌───────▼────────────┐
        │   POST SERVICE        │   │   TIMELINE SERVICE  │
        │   - Create posts      │   │   - Generate feeds  │
        │   - Edit/delete       │   │   - Fan-out logic   │
        └──────┬────────────────┘   └───────┬─────────────┘
               │                            │
        ┌──────▼─────────┐          ┌──────▼──────────┐
        │ POST STORAGE   │          │ TIMELINE CACHE  │
        │ (Cassandra)    │          │ (Redis)         │
        │ - Posts table  │          │ - User timelines│
        └────────────────┘          └─────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│                      SUPPORTING SERVICES                                │
├────────────────┬──────────────────┬──────────────────┬─────────────────┤
│ SOCIAL GRAPH   │ NOTIFICATION     │ MEDIA SERVICE    │ SEARCH SERVICE  │
│ - Follow graph │ - Push/email     │ - Image upload   │ - Elasticsearch │
│ (Neo4j/MySQL)  │ - Queue based    │ - Video encoding │ - Posts index   │
└────────────────┴──────────────────┴──────────────────┴─────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│                          MESSAGE QUEUE (Kafka)                          │
│   [Post Created] [Like Event] [Comment Event] [Follow Event]          │
└────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│                     STORAGE & CDN LAYER                                 │
│   [Object Storage - S3] → [CDN - CloudFront/Cloudflare]               │
└────────────────────────────────────────────────────────────────────────┘
```

## Fan-Out Strategies

### The Core Challenge

When a user posts, we need to deliver that post to all their followers' timelines. Two main approaches:

### Approach 1: Fan-Out on Write (Push Model)

**How It Works:**
```
User A (1,000 followers) creates a post:
1. Store post in database
2. Get all 1,000 followers from social graph
3. For each follower, insert post ID into their timeline cache
4. All 1,000 timelines now have the post

When follower loads timeline:
1. Read from pre-computed timeline cache
2. Fetch post details
3. Return immediately (FAST!)
```

**Pros:**
- ✅ Extremely fast reads (timeline pre-computed)
- ✅ Simple timeline generation logic
- ✅ Handles high read traffic efficiently
- ✅ Good for users with few followers

**Cons:**
- ❌ Expensive writes for popular users (fan-out to millions)
- ❌ Wasted work if followers never check timeline
- ❌ High memory usage (duplicate data)
- ❌ Delay in post creation for celebrities

**Implementation:**
```python
async def fan_out_on_write(user_id: int, post_id: int):
    """Push post to all followers' timelines."""
    # Get all followers
    followers = await social_graph.get_followers(user_id)
    
    # Batch insert into Redis timelines
    pipeline = redis.pipeline()
    for follower_id in followers:
        timeline_key = f"timeline:{follower_id}"
        pipeline.lpush(timeline_key, post_id)
        pipeline.ltrim(timeline_key, 0, 999)  # Keep latest 1000
    
    await pipeline.execute()
    
    # Also index for search
    await search_service.index_post(post_id)

# Time complexity: O(N) where N = number of followers
# For 1M followers: ~2-3 seconds (too slow!)
```

### Approach 2: Fan-Out on Read (Pull Model)

**How It Works:**
```
User A (1M followers) creates a post:
1. Store post in database
2. Done! (No fan-out)

When follower loads timeline:
1. Get list of users they follow
2. Query recent posts from each followed user
3. Merge and sort by timestamp
4. Return timeline (SLOWER but doable)
```

**Pros:**
- ✅ Fast writes (no fan-out needed)
- ✅ No wasted work (compute only when requested)
- ✅ Works well for celebrity accounts
- ✅ Lower memory footprint

**Cons:**
- ❌ Slower reads (compute on demand)
- ❌ Complex merge logic
- ❌ High database load on reads
- ❌ Difficult to scale read traffic

**Implementation:**
```python
async def fan_out_on_read(user_id: int, limit: int = 20):
    """Compute timeline on-demand."""
    # Get users this person follows
    following = await social_graph.get_following(user_id)
    
    # Fetch recent posts from each followed user
    posts = []
    for followed_id in following:
        recent = await post_db.get_user_posts(
            followed_id, 
            limit=100  # Get last 100 posts per user
        )
        posts.extend(recent)
    
    # Sort by timestamp and return top N
    posts.sort(key=lambda p: p.created_at, reverse=True)
    return posts[:limit]

# Time complexity: O(N × M × log(N×M))
# N = following count, M = posts per user
# For 200 following × 100 posts = 20K posts to sort (slow!)
```

### Approach 3: Hybrid Strategy (Best of Both Worlds)

**Intelligent Fan-Out Based on User Type:**

```python
class FanOutStrategy:
    CELEBRITY_THRESHOLD = 100_000  # 100K followers
    
    async def handle_new_post(self, user_id: int, post_id: int):
        """Choose fan-out strategy based on user type."""
        follower_count = await social_graph.get_follower_count(user_id)
        
        if follower_count < self.CELEBRITY_THRESHOLD:
            # Regular user: Fan-out on write
            await self.fan_out_on_write(user_id, post_id)
        else:
            # Celebrity: Fan-out on read
            await self.mark_as_celebrity_post(user_id, post_id)
    
    async def generate_timeline(self, user_id: int, limit: int = 20):
        """Generate timeline using hybrid approach."""
        # 1. Get cached timeline (fan-out on write posts)
        cached_timeline = await redis.lrange(
            f"timeline:{user_id}", 0, limit * 2
        )
        
        # 2. Get posts from celebrities user follows (fan-out on read)
        celebrity_posts = await self.get_celebrity_posts(user_id, limit)
        
        # 3. Merge both sources
        all_posts = cached_timeline + celebrity_posts
        all_posts.sort(key=lambda p: p.created_at, reverse=True)
        
        return all_posts[:limit]
    
    async def get_celebrity_posts(self, user_id: int, limit: int):
        """Get recent posts from celebrities user follows."""
        celebrities = await social_graph.get_celebrity_following(user_id)
        
        posts = []
        for celeb_id in celebrities:
            recent = await post_db.get_user_posts(celeb_id, limit=50)
            posts.extend(recent)
        
        return posts
```

**Hybrid Benefits:**
- ✅ Fast reads for most users (cached timelines)
- ✅ Fast writes for celebrities (no expensive fan-out)
- ✅ Scales to billions of users
- ✅ Optimal resource usage

## Caching Layers

### Multi-Level Caching Strategy

```
┌────────────────────────────────────────────────────────────┐
│ LAYER 1: TIMELINE CACHE (Redis)                            │
│ - Stores post IDs for each user's timeline                 │
│ - TTL: 24 hours                                            │
│ - Hit rate: 95%                                            │
│ Key: "timeline:{user_id}" → [post_id_1, post_id_2, ...]   │
└────────────────────────────────────────────────────────────┘
                              ↓ (on miss)
┌────────────────────────────────────────────────────────────┐
│ LAYER 2: POST CACHE (Redis)                                │
│ - Stores full post objects                                 │
│ - TTL: 6 hours                                             │
│ - Hit rate: 90%                                            │
│ Key: "post:{post_id}" → {post_data}                        │
└────────────────────────────────────────────────────────────┘
                              ↓ (on miss)
┌────────────────────────────────────────────────────────────┐
│ LAYER 3: USER CACHE (Redis)                                │
│ - Stores user profile data                                 │
│ - TTL: 1 hour                                              │
│ - Hit rate: 85%                                            │
│ Key: "user:{user_id}" → {user_data}                        │
└────────────────────────────────────────────────────────────┘
                              ↓ (on miss)
┌────────────────────────────────────────────────────────────┐
│ LAYER 4: DATABASE (Cassandra)                              │
│ - Authoritative source of truth                            │
│ - All posts, users, relationships                          │
└────────────────────────────────────────────────────────────┘
```

### Cache Invalidation Strategies

**Write-Through Cache:**
```python
async def create_post(user_id: int, content: str):
    """Create post with cache-aside pattern."""
    # 1. Write to database first
    post = await db.insert_post(user_id, content)
    
    # 2. Write to cache immediately
    await redis.setex(
        f"post:{post.id}",
        21600,  # 6 hours
        json.dumps(post)
    )
    
    # 3. Fan out to followers asynchronously
    await kafka.produce("new_post", {
        "user_id": user_id,
        "post_id": post.id
    })
    
    return post
```

**Cache Warming:**
```python
async def warm_timeline_cache(user_id: int):
    """Pre-populate timeline cache for active users."""
    # Get following list
    following = await social_graph.get_following(user_id)
    
    # Fetch recent posts from followed users
    posts = await db.get_posts_from_users(following, limit=500)
    
    # Sort and cache
    posts.sort(key=lambda p: p.created_at, reverse=True)
    post_ids = [p.id for p in posts[:1000]]
    
    await redis.delete(f"timeline:{user_id}")
    await redis.rpush(f"timeline:{user_id}", *post_ids)
    await redis.expire(f"timeline:{user_id}", 86400)
```

## API Design

### 1. Create Post

```http
POST /api/v1/posts
Authorization: Bearer <token>
Content-Type: application/json

Request:
{
  "content": "Just learned about system design! 🚀",
  "media_urls": ["https://cdn.example.com/image1.jpg"],
  "visibility": "public"  // public, followers, private
}

Response 201 Created:
{
  "post_id": "1234567890",
  "user_id": "user_123",
  "content": "Just learned about system design! 🚀",
  "media_urls": ["https://cdn.example.com/image1.jpg"],
  "created_at": "2024-01-15T10:30:00Z",
  "likes_count": 0,
  "comments_count": 0,
  "shares_count": 0
}
```

### 2. Get Timeline

```http
GET /api/v1/timeline?limit=20&cursor=abc123
Authorization: Bearer <token>

Response 200 OK:
{
  "posts": [
    {
      "post_id": "1234567890",
      "user": {
        "user_id": "user_456",
        "username": "alice",
        "display_name": "Alice Smith",
        "avatar_url": "https://cdn.example.com/avatar.jpg"
      },
      "content": "System design is fascinating!",
      "created_at": "2024-01-15T10:30:00Z",
      "likes_count": 42,
      "comments_count": 5,
      "has_liked": false
    }
    // ... more posts
  ],
  "next_cursor": "xyz789",
  "has_more": true
}
```

### 3. Like/Unlike Post

```http
POST /api/v1/posts/{post_id}/like
Authorization: Bearer <token>

Response 200 OK:
{
  "post_id": "1234567890",
  "likes_count": 43,
  "has_liked": true
}

DELETE /api/v1/posts/{post_id}/like
Response 200 OK:
{
  "post_id": "1234567890",
  "likes_count": 42,
  "has_liked": false
}
```

### 4. Follow/Unfollow User

```http
POST /api/v1/users/{user_id}/follow
Authorization: Bearer <token>

Response 200 OK:
{
  "user_id": "user_456",
  "following": true,
  "followers_count": 1523
}
```

## Database Schema

### Posts Table (Cassandra)

```sql
CREATE TABLE posts (
    post_id UUID PRIMARY KEY,
    user_id UUID,
    content TEXT,
    media_urls LIST<TEXT>,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    likes_count COUNTER,
    comments_count COUNTER,
    shares_count COUNTER,
    visibility TEXT,
    is_deleted BOOLEAN
);

-- Index for querying user's posts
CREATE TABLE posts_by_user (
    user_id UUID,
    created_at TIMESTAMP,
    post_id UUID,
    PRIMARY KEY (user_id, created_at, post_id)
) WITH CLUSTERING ORDER BY (created_at DESC);

-- Index for trending posts
CREATE TABLE posts_by_engagement (
    time_bucket TEXT,  -- e.g., "2024-01-15-10" (hourly)
    engagement_score INT,
    post_id UUID,
    PRIMARY KEY (time_bucket, engagement_score, post_id)
) WITH CLUSTERING ORDER BY (engagement_score DESC);
```

### Social Graph (MySQL or Neo4j)

```sql
-- MySQL approach
CREATE TABLE follows (
    follower_id BIGINT NOT NULL,
    followed_id BIGINT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    PRIMARY KEY (follower_id, followed_id),
    INDEX idx_followed (followed_id, follower_id)
);

-- For fast queries:
-- "Who does Alice follow?" → SELECT followed_id WHERE follower_id = alice_id
-- "Who follows Alice?" → SELECT follower_id WHERE followed_id = alice_id
```

### Likes Table (Cassandra)

```sql
CREATE TABLE likes (
    post_id UUID,
    user_id UUID,
    created_at TIMESTAMP,
    PRIMARY KEY (post_id, user_id)
);

-- For checking if user liked a post
CREATE TABLE user_likes (
    user_id UUID,
    post_id UUID,
    created_at TIMESTAMP,
    PRIMARY KEY (user_id, post_id)
);
```

## Trade-Off Discussions

### 1. Consistency vs Availability

**Strong Consistency:**
- ✅ All users see same data immediately
- ✅ No conflicting updates
- ❌ Slower writes (coordination overhead)
- ❌ Reduced availability (CAP theorem)

**Eventual Consistency:**
- ✅ High availability
- ✅ Fast writes
- ❌ Temporary inconsistencies (like count might be stale)
- ❌ Conflict resolution needed

**Recommendation**: Use eventual consistency for likes/counts (acceptable to be stale by seconds), strong consistency for follow relationships.

### 2. Normalized vs Denormalized Data

**Normalized:**
- ✅ No data duplication
- ✅ Easy updates
- ❌ Requires joins (slow for reads)
- ❌ Complex queries

**Denormalized:**
- ✅ Fast reads (no joins)
- ✅ Self-contained records
- ❌ Data duplication
- ❌ Update complexity

**Recommendation**: Denormalize for read-heavy data (embed user info in posts for timeline API).

### 3. Real-Time vs Batch Processing

**Real-Time Fan-Out:**
- ✅ Immediate timeline updates
- ✅ Fresh content
- ❌ High computational cost
- ❌ Scaling challenges

**Batch Fan-Out:**
- ✅ Efficient resource usage
- ✅ Better throughput
- ❌ Delayed updates (minutes)
- ❌ Stale timelines

**Recommendation**: Real-time for regular users (< 5 sec), batch for celebrities (< 60 sec acceptable).

### 4. Ranking Algorithm

**Chronological (Simple):**
- ✅ Easy to implement
- ✅ Predictable
- ❌ Low-quality content shown equally
- ❌ User might miss important posts

**Algorithmic (ML-based):**
- ✅ Personalized experience
- ✅ Higher engagement
- ❌ Complex to build and maintain
- ❌ "Black box" concerns

**Recommendation**: Start with chronological + simple engagement boost (recent posts with high likes ranked higher).

## Scaling Strategies

### Phase 1: MVP (< 100K users)

```
[Load Balancer]
      ↓
[API Servers] → [Redis] → [PostgreSQL]
```

- Monolithic application
- Single database
- Simple fan-out on write
- Handles 1,000 posts/day

### Phase 2: Growing (100K - 10M users)

```
[Load Balancer]
      ↓
[API Servers x5] → [Redis Cluster] → [PostgreSQL Primary]
                                            ↓
                                      [Read Replicas x3]
```

- Microservices separation (post, timeline, social graph)
- Database read replicas
- Redis cluster for caching
- Handles 100K posts/day

### Phase 3: Large Scale (10M - 100M users)

```
[Global Load Balancer]
      ↓
[API Gateway + CDN]
      ↓
[Post Service] [Timeline Service] [Social Graph]
      ↓              ↓                  ↓
[Cassandra]    [Redis Cluster]      [MySQL Sharded]
      ↓
[Kafka] → [Fan-out Workers x100]
```

- Cassandra for posts (horizontal scaling)
- Async fan-out with Kafka
- Sharded social graph
- Hybrid fan-out strategy
- Handles 10M posts/day

### Phase 4: Massive Scale (100M+ users)

```
[Multi-Region Deployment]
      ↓
[Region-Specific Clusters]
      ↓
[Microservices Architecture]
  - Post Service (100+ instances)
  - Timeline Service (200+ instances)
  - Fan-out Service (500+ workers)
  - Social Graph Service (50+ instances)
      ↓
[Distributed Databases]
  - Cassandra (100+ nodes)
  - Redis Clusters (region-specific)
  - Graph Database (Neo4j cluster)
```

- Multi-region deployment
- Geo-distributed databases
- Advanced ML-based feed ranking
- Handles 400M posts/day

## Interview Talking Points

### Key Discussion Points

**1. Clarifying Questions:**
- "What's the expected ratio of active posters vs passive consumers?"
- "Do we need chronological or algorithmic feed?"
- "Should posts be real-time or near real-time?"
- "What's the celebrity user threshold (followers count)?"

**2. System Bottlenecks:**
- "Fan-out for celebrities is the main bottleneck"
  - Solution: Hybrid strategy (fan-out on read for celebrities)
- "Timeline generation at scale is expensive"
  - Solution: Aggressive caching with 24-hour TTL
- "Database writes can't keep up with post creation"
  - Solution: Migrate to Cassandra for horizontal scaling

**3. Optimization Techniques:**
- "Cache timeline post IDs (8 bytes) not full posts (500+ bytes)"
- "Pre-compute timelines during off-peak hours"
- "Batch database updates for like counts"
- "Use CDN for media-heavy posts"

**4. Failure Scenarios:**
- "What if Redis cache fails?"
  - Fall back to database, regenerate timeline on-demand
- "What if fan-out service is slow?"
  - Queue posts, process when service recovers
- "What if database becomes unavailable?"
  - Serve stale cached data, queue writes for replay

**5. Advanced Features:**
- "How would you add stories (24-hour content)?"
  - Separate Redis cache with TTL=24h, no database persistence
- "How would you implement mentions/hashtags?"
  - Extract during post creation, index in Elasticsearch
- "How would you do content moderation?"
  - Async processing with ML models, queue flagged content

### Scalability Checklist

✅ **Achieved at Scale:**
- 400M posts/day
- 50B timeline reads/day
- <200ms p95 timeline load time
- 99.95% availability
- Handles celebrity users with millions of followers

✅ **Cost Optimizations:**
- Fan-out on read for celebrities saves 90% of fan-out compute
- Caching reduces database load by 95%
- CDN reduces egress costs by 80%

## Summary

The social media feed system demonstrates advanced distributed system concepts:

**Key Design Decisions:**
1. **Hybrid Fan-Out**: Combines best of push and pull models
2. **Multi-Level Caching**: Reduces database load by 95%+
3. **Async Processing**: Decouples post creation from fan-out
4. **Eventual Consistency**: Acceptable for likes, counts, timelines

**Critical Components:**
- Post Service: Handles content creation and storage
- Timeline Service: Generates personalized feeds efficiently
- Social Graph: Manages follow relationships at scale
- Fan-Out Service: Distributes posts to followers intelligently

**Scalability Achievements:**
- Scales from 0 to 500M users
- Handles celebrity users (10M+ followers)
- Sub-200ms timeline load times
- 99.95% availability with multi-region deployment

This design mirrors real-world implementations from Twitter, Instagram, and Facebook, proving that hybrid strategies and intelligent caching are essential for feed systems at scale.
