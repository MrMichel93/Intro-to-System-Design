# Case Study: Twitter/X

## Problem Statement

Design a social media platform where users can:
- Post short messages (tweets, up to 280 characters)
- Follow other users
- View a personalized timeline of tweets from people they follow
- Like, retweet, and reply to tweets
- Search for tweets and users
- Receive notifications

## Requirements

### Functional Requirements
- Users can post tweets (text, images, videos)
- Users can follow/unfollow others
- Users see timeline of tweets from people they follow
- Users can like, retweet, reply
- Search functionality
- Trending topics
- Notifications

### Non-Functional Requirements
- **Scale**: 400M monthly active users, 200M daily active
- **Tweets**: 500M tweets per day (~6,000 tweets/sec average, 20K/sec peak)
- **Timeline reads**: 300M daily active users × 50 timeline views = 15B reads/day
- **Latency**: Timeline loads in < 200ms
- **Availability**: 99.9% uptime
- **Read-heavy**: 100:1 read-to-write ratio

## Architecture Overview

```
[Users/Clients]
       ↓
[Load Balancer]
       ↓
[API Gateway]
    ↓    ↓    ↓
[Tweet] [Timeline] [Search] [Notification]
Service  Service   Service   Service
    ↓        ↓         ↓          ↓
[Tweet DB] [Timeline] [Search] [Notification]
         [Cache]    [Index]    [Queue]
```

## Key Components

### 1. Tweet Service
**Responsibilities:**
- Store tweets
- Retrieve tweets
- Handle likes, retweets, replies

**Database Design:**
```sql
Tweets:
- tweet_id (primary key)
- user_id (author)
- content (text)
- media_urls (array)
- created_at
- like_count
- retweet_count
- reply_count

Partitioning: Shard by tweet_id
```

**Scale:**
- 500M tweets/day = 6K writes/sec (average)
- Storage: 500M × 280 bytes = 140 GB/day (text only)
- With media: ~2 TB/day
- Yearly: 730 TB

### 2. Timeline Service
**Two Approaches:**

**Approach A: Fan-Out on Write (Used for most users)**
```
User posts tweet → Push to all followers' timelines
- Write: Slower (update millions of timelines)
- Read: Very fast (pre-computed)
- Best for: Regular users
```

**Approach B: Fan-Out on Read (Used for celebrities)**
```
Celebrity posts → Just store tweet
Timeline requested → Fetch celebrity tweets on-demand
- Write: Fast
- Read: Slower (compute on demand)
- Best for: Users with millions of followers
```

**Hybrid Solution:**
```python
if user.followers_count < 1_000_000:
    # Fan-out on write
    for follower in user.followers:
        push_to_timeline(follower, tweet)
else:
    # Celebrity: fan-out on read
    store_tweet(tweet)
    # Timelines fetch celebrity tweets separately
```

**Timeline Storage:**
```
Redis Cache:
Key: user_id
Value: List of tweet_ids (recent 800 tweets)

Example:
user:12345:timeline = [tweet999, tweet998, tweet997, ...]
```

### 3. Social Graph Service
**Stores following/followers relationships**

```
Following Table:
- user_id
- follows_user_id
- created_at

Index: (user_id, follows_user_id)
Index: (follows_user_id, user_id) for followers lookup
```

**Scale:**
- 400M users
- Average 200 following each
- 80B relationships
- Storage: 80B × 16 bytes = 1.3 TB

### 4. Search Service
**Technology:** Elasticsearch

**Indexed Fields:**
- Tweet content
- Hashtags
- User mentions
- Timestamp

**Search Queries:**
```
Search tweets by keyword
Search tweets by hashtag
Search tweets by user
Search users by name/username
```

**Performance:**
- Index updates: Near real-time (1-2 second lag)
- Search latency: < 100ms
- Scale: Distributed across multiple nodes

### 5. Notification Service
**Types:**
- New follower
- Tweet liked
- Tweet retweeted
- Mentioned in tweet
- Direct message

**Implementation:**
```
Message Queue (Kafka):
- Event produced: User A likes User B's tweet
- Consumer: Notification service
- Action: Create notification for User B
- Delivery: Push notification, in-app notification
```

## Data Flow Examples

### Example 1: Posting a Tweet

```
1. User posts tweet via mobile app
2. Request → Load Balancer → Tweet Service
3. Tweet Service:
   - Validate tweet (length, content)
   - Generate tweet_id
   - Store in database
   - Upload media to S3/CDN (if any)
4. Timeline Service:
   - Get user's followers (from Social Graph Service)
   - If < 1M followers: Fan-out to follower timelines (async)
   - If > 1M followers: Mark as celebrity tweet
5. Search Service:
   - Index tweet asynchronously
6. Return success to user

Timeline: ~200ms for user, fan-out happens in background
```

### Example 2: Loading Timeline

```
1. User opens app, requests timeline
2. Request → Timeline Service
3. Timeline Service:
   - Check Redis cache: user:12345:timeline
   - If hit: Return cached tweet IDs
   - If miss: Fetch from database
4. For each tweet ID:
   - Fetch tweet details (parallel requests)
   - Fetch author info (cached)
   - Fetch engagement counts
5. Mix in celebrity tweets (fan-out on read)
6. Sort by timestamp
7. Return to user

Timeline: < 200ms total
```

## Capacity Estimation

### Storage (5 years)
```
Tweets:
- 500M tweets/day × 365 days × 5 years = 912B tweets
- Metadata: 912B × 500 bytes = 456 TB
- Media: ~2 TB/day × 365 × 5 = 3,650 TB = 3.6 PB
- Total: ~4 PB

Timelines (cache):
- 200M daily active users
- 800 tweets per timeline
- 200M × 800 × 8 bytes = 1.3 TB (can fit in distributed Redis)
```

### Bandwidth
```
Writes:
- 6K tweets/sec × 500 bytes = 3 MB/sec (metadata)
- With media: ~200 MB/sec

Reads:
- 15B timeline views/day = 174K requests/sec
- Each timeline: 20 tweets × 500 bytes = 10 KB
- 174K × 10 KB = 1.7 GB/sec
- With images: ~20 GB/sec (CDN handles this)
```

## Technology Stack

- **Application Servers**: Java/Scala
- **Databases**: MySQL (sharded), Cassandra for some data
- **Cache**: Redis (timelines), Memcached
- **Message Queue**: Kafka
- **Search**: Elasticsearch
- **Storage**: S3 for media
- **CDN**: CloudFront, Fastly
- **Load Balancer**: HAProxy, Nginx

## Key Design Decisions & Trade-Offs

### Decision 1: Hybrid Fan-Out Strategy

**Trade-off:**
- Fan-out on write: Better read performance, but expensive for celebrities
- Fan-out on read: Cheaper writes, but slower reads

**Solution:** Hybrid approach based on follower count

**Why:** Optimizes for the common case (most users have < 1M followers)

### Decision 2: Eventual Consistency for Timelines

**Trade-off:**
- Strong consistency: Slower, more complex
- Eventual consistency: Faster, simpler, brief delays OK

**Choice:** Eventual consistency

**Why:** Users don't notice 1-2 second delay in timeline updates

### Decision 3: Redis for Timeline Cache

**Trade-off:**
- Database only: Slower, expensive queries
- Cache: Much faster, but need cache invalidation

**Choice:** Heavy caching with Redis

**Why:** Read-heavy workload benefits massively from caching

## Scaling Evolution

### Phase 1: Early Days (< 1M users)
- Monolithic application
- Single MySQL database
- Simple architecture

### Phase 2: Growth (1-10M users)
- Separate services (tweets, timeline, search)
- Database read replicas
- Redis caching introduced
- CDN for media

### Phase 3: Scale (10-100M users)
- Database sharding
- Fan-out on write for timelines
- Multiple datacenters
- Kafka for message queuing

### Phase 4: Massive Scale (100M+ users)
- Hybrid fan-out strategy
- Extensive caching layers
- Global CDN
- Custom optimization everywhere

## Lessons Learned

1. **Start simple, scale when needed**
2. **Cache aggressively for read-heavy workloads**
3. **Different users need different strategies** (regular vs celebrity)
4. **Eventual consistency is often acceptable**
5. **Monitor everything** to catch issues early
6. **Plan for failures** - redundancy is critical

## Summary

Twitter demonstrates:
- **Hybrid approaches** work well (fan-out strategies)
- **Caching is critical** for performance at scale
- **Different optimization** for different user types
- **Trade eventual consistency** for performance
- **Start simple**, add complexity as you grow

This design handles billions of reads and millions of writes per day while maintaining sub-200ms response times!
