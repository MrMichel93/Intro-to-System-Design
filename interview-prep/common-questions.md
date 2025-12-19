# 20 Most Common System Design Interview Questions

A comprehensive list of the most frequently asked system design questions, with solution approaches and key topics to cover.

## How to Use This Guide

For each question:
1. **Problem Statement** - What you're being asked to design
2. **Difficulty Level** - Easy, Medium, or Hard
3. **Solution Approach** - High-level strategy to solve it
4. **Key Topics to Cover** - Critical areas to discuss
5. **Related Modules** - Links to relevant course content

---

## Easy Questions (Complexity Level 1-2)

### 1. Design a URL Shortener (like bit.ly)

**Difficulty:** ⭐ Easy  
**Estimated Time:** 45 minutes

**Problem Statement:**
Design a system that takes long URLs and generates short, unique URLs that redirect to the original URL.

**Solution Approach:**

1. **Core functionality:**
   - Generate unique short code (6-8 characters)
   - Store mapping: short code → long URL
   - Redirect users from short URL to original URL

2. **High-level architecture:**
```
Client → Load Balancer → API Servers → Database
                                    ↓
                                  Cache
```

3. **Key design decisions:**
   - Use base62 encoding (a-z, A-Z, 0-9) for short codes
   - Hash function or auto-incrementing ID for uniqueness
   - Store in key-value database (Redis/DynamoDB)
   - Cache frequently accessed URLs

**Key Topics to Cover:**
- ✅ **URL shortening algorithms** (hashing vs. auto-increment)
- ✅ **Collision handling** (how to deal with duplicate codes)
- ✅ **Database choice** (SQL vs NoSQL - NoSQL is better here)
- ✅ **Caching strategy** (cache hot URLs for fast redirects)
- ✅ **Scale estimation** (QPS, storage needs)
- ✅ **Custom short URLs** (allowing users to choose their code)
- ✅ **Analytics** (tracking clicks, location, time)
- ✅ **Expiration** (should URLs expire after N days?)

**Related Modules:**
- [Database Design](../06-database-design/)
- [Caching](../05-caching/)
- [Case Study: URL Shortener](../case-studies/url-shortener.md)

---

### 2. Design a Pastebin (like pastebin.com)

**Difficulty:** ⭐ Easy  
**Estimated Time:** 45 minutes

**Problem Statement:**
Design a system where users can paste text content and get a unique URL to share it. Content may expire after a certain time.

**Solution Approach:**

1. **Core functionality:**
   - Users paste text and get unique URL
   - Others can view the content via URL
   - Optional: expiration times, syntax highlighting

2. **High-level architecture:**
```
Client → API Gateway → App Servers → Object Storage (S3)
                            ↓
                        Database (metadata)
                            ↓
                          Cache
```

3. **Key design decisions:**
   - Store large pastes in object storage (S3)
   - Store metadata (paste ID, user, expiration) in database
   - Generate unique IDs similar to URL shortener
   - Implement TTL for automatic expiration

**Key Topics to Cover:**
- ✅ **Unique ID generation** (similar to URL shortener)
- ✅ **Storage strategy** (database vs object storage vs blob storage)
- ✅ **Expiration mechanism** (TTL, background job cleanup)
- ✅ **Access control** (public vs private pastes)
- ✅ **Size limits** (how large can pastes be?)
- ✅ **Content type** (text, code, markdown)
- ✅ **Syntax highlighting** (client-side or server-side?)

**Related Modules:**
- [Database Design](../06-database-design/)
- [CDN](../14-cdn/)
- [Caching](../05-caching/)

---

### 3. Design a Key-Value Store (like Redis)

**Difficulty:** ⭐⭐ Easy-Medium  
**Estimated Time:** 45 minutes

**Problem Statement:**
Design a distributed key-value store that supports GET, PUT, and DELETE operations with high availability and scalability.

**Solution Approach:**

1. **Core functionality:**
   - Basic operations: GET(key), PUT(key, value), DELETE(key)
   - In-memory storage for fast access
   - Persistence to disk for durability

2. **High-level architecture:**
```
Client → API Layer → Partition Manager → Storage Nodes (sharded)
                            ↓
                    Replication (Master-Slave)
```

3. **Key design decisions:**
   - Consistent hashing for data distribution
   - Replication for availability (master-slave or multi-master)
   - Write-ahead log (WAL) for durability
   - Eventual consistency model

**Key Topics to Cover:**
- ✅ **Data partitioning** (consistent hashing)
- ✅ **Replication strategy** (synchronous vs asynchronous)
- ✅ **Consistency model** (strong vs eventual)
- ✅ **Durability** (persistence mechanisms)
- ✅ **CAP theorem trade-offs** (AP or CP?)
- ✅ **Failure handling** (node failures, network partitions)
- ✅ **Data structures** (hash table, skip list, LSM tree)

**Related Modules:**
- [Data Partitioning](../07-data-partitioning/)
- [Replication](../08-replication/)
- [Consistency Patterns](../09-consistency-patterns/)
- [CAP Theorem](../02-cap-theorem/)

---

### 4. Design a Rate Limiter

**Difficulty:** ⭐⭐ Easy-Medium  
**Estimated Time:** 45 minutes

**Problem Statement:**
Design a system that limits the number of requests a user or service can make in a given time window.

**Solution Approach:**

1. **Core functionality:**
   - Limit requests per user/IP/API key
   - Configurable time windows (per second, minute, hour)
   - Return error when limit exceeded

2. **Algorithms to consider:**
   - **Token bucket** (recommended)
   - **Leaky bucket**
   - **Fixed window counter**
   - **Sliding window log**

3. **High-level architecture:**
```
Request → Rate Limiter (Redis) → API Gateway → Backend Services
                ↓
            Rules Engine
```

**Key Topics to Cover:**
- ✅ **Rate limiting algorithms** (token bucket, leaky bucket, sliding window)
- ✅ **Storage** (Redis with TTL for counters)
- ✅ **Distributed rate limiting** (handling multiple servers)
- ✅ **Rules configuration** (different limits for different users/endpoints)
- ✅ **Response handling** (HTTP 429, retry-after header)
- ✅ **Failure mode** (fail open vs fail closed)

**Related Modules:**
- [Caching](../05-caching/)
- [API Design](../10-api-design/)

---

## Medium Questions (Complexity Level 3-4)

### 5. Design Twitter/X

**Difficulty:** ⭐⭐⭐ Medium  
**Estimated Time:** 45-60 minutes

**Problem Statement:**
Design a social media platform where users post short messages, follow others, and view a personalized timeline.

**Solution Approach:**

1. **Core functionality:**
   - Post tweets (text, images, videos)
   - Follow/unfollow users
   - View timeline (tweets from followed users)
   - Like, retweet, reply

2. **Critical design choice: Feed generation**
   - **Fan-out on write** (push model): Pre-compute feeds
   - **Fan-out on read** (pull model): Compute feeds on demand
   - **Hybrid**: Fan-out for regular users, pull for celebrities

3. **High-level architecture:**
```
Users → Load Balancer → API Gateway
            ↓
    [Tweet Service] [Timeline Service] [Social Graph]
            ↓               ↓                ↓
    [Tweet DB]      [Feed Cache]      [Graph DB]
```

**Key Topics to Cover:**
- ✅ **Fan-out strategies** (write vs read vs hybrid)
- ✅ **Timeline generation** (how to build personalized feeds)
- ✅ **Social graph storage** (followers/following relationships)
- ✅ **Media handling** (image/video upload and storage)
- ✅ **Scalability** (read-heavy workload, 100:1 ratio)
- ✅ **Caching strategy** (timeline caching, user caching)
- ✅ **Notifications** (async processing with message queues)
- ✅ **Search** (full-text search on tweets)

**Related Modules:**
- [Message Queues](../11-message-queues/)
- [Caching](../05-caching/)
- [Data Partitioning](../07-data-partitioning/)
- [Case Study: Twitter](../case-studies/twitter.md)

---

### 6. Design Instagram

**Difficulty:** ⭐⭐⭐ Medium  
**Estimated Time:** 45-60 minutes

**Problem Statement:**
Design a photo-sharing platform where users can upload photos, follow users, and view a feed of photos from people they follow.

**Solution Approach:**

1. **Core functionality:**
   - Upload photos/videos
   - Follow users
   - View feed
   - Like and comment on posts

2. **Key differences from Twitter:**
   - Larger media files (images/videos)
   - Less time-sensitive (eventual consistency OK)
   - More read-heavy (users scroll feeds extensively)

3. **High-level architecture:**
```
Users → CDN (images) → Load Balancer → App Servers
                            ↓
            [Upload Service] [Feed Service] [Graph Service]
                    ↓              ↓              ↓
            [Object Storage]  [Feed Cache]  [SQL/NoSQL DB]
```

**Key Topics to Cover:**
- ✅ **Photo storage** (object storage like S3, not database)
- ✅ **CDN strategy** (distribute images globally)
- ✅ **Image processing** (thumbnails, compression, multiple sizes)
- ✅ **Feed generation** (similar to Twitter but less real-time)
- ✅ **Graph database** (storing social connections)
- ✅ **Metadata storage** (likes, comments, captions)
- ✅ **Timeline ranking** (chronological vs algorithmic)
- ✅ **Stories feature** (ephemeral content with 24h expiration)

**Related Modules:**
- [CDN](../14-cdn/)
- [Database Design](../06-database-design/)
- [Caching](../05-caching/)

---

### 7. Design Netflix/YouTube (Video Streaming)

**Difficulty:** ⭐⭐⭐⭐ Medium-Hard  
**Estimated Time:** 60 minutes

**Problem Statement:**
Design a video streaming platform that allows users to upload, store, and stream videos at scale.

**Solution Approach:**

1. **Core functionality:**
   - Upload videos
   - Stream videos (adaptive bitrate)
   - Search and discover content
   - Recommendations

2. **Critical challenges:**
   - Massive storage requirements
   - High bandwidth consumption
   - Global content delivery
   - Video transcoding

3. **High-level architecture:**
```
Users → CDN (video delivery) → Origin Servers
            ↓
    [Upload Service] → [Transcoding Pipeline] → [Object Storage]
            ↓                                          ↓
    [Metadata DB] ← [Search Service] ← [Recommendation Engine]
```

**Key Topics to Cover:**
- ✅ **Video storage** (object storage, distributed file system)
- ✅ **Transcoding** (multiple formats, resolutions, bitrates)
- ✅ **CDN strategy** (edge caching, content distribution)
- ✅ **Adaptive bitrate streaming** (HLS, DASH)
- ✅ **Recommendation system** (collaborative filtering, ML)
- ✅ **Search** (Elasticsearch for content discovery)
- ✅ **DRM** (content protection)
- ✅ **Analytics** (watch time, engagement metrics)
- ✅ **Cost optimization** (storage tiers, bandwidth management)

**Related Modules:**
- [CDN](../14-cdn/)
- [Message Queues](../11-message-queues/)
- [Database Design](../06-database-design/)
- [Case Study: Video Streaming](../case-studies/video-streaming.md)

---

### 8. Design WhatsApp/Messenger (Messaging System)

**Difficulty:** ⭐⭐⭐⭐ Medium-Hard  
**Estimated Time:** 60 minutes

**Problem Statement:**
Design a real-time messaging system supporting one-on-one and group chats with message delivery guarantees.

**Solution Approach:**

1. **Core functionality:**
   - Send/receive messages (one-on-one)
   - Group chats
   - Online/offline status
   - Message delivery confirmation (sent, delivered, read)
   - Media sharing

2. **Critical requirements:**
   - Real-time delivery
   - Ordered message delivery
   - Reliability (no message loss)
   - End-to-end encryption

3. **High-level architecture:**
```
Clients (WebSocket) ↔ Connection Servers → Message Queue
                            ↓                    ↓
                    [Presence Service]    [Message Service]
                            ↓                    ↓
                    [Cache (Redis)]      [Message DB]
```

**Key Topics to Cover:**
- ✅ **Real-time communication** (WebSocket, long polling)
- ✅ **Message delivery** (at-least-once, exactly-once semantics)
- ✅ **Message ordering** (sequence numbers, vector clocks)
- ✅ **Presence system** (online/offline/typing status)
- ✅ **Message storage** (recent messages in cache, older in DB)
- ✅ **Group chats** (fan-out, message distribution)
- ✅ **Media handling** (separate pipeline from text)
- ✅ **Encryption** (end-to-end encryption)
- ✅ **Offline messages** (queue messages for offline users)
- ✅ **Read receipts** (tracking message status)

**Related Modules:**
- [Message Queues](../11-message-queues/)
- [Consistency Patterns](../09-consistency-patterns/)
- [Replication](../08-replication/)

---

### 9. Design Uber/Lyft (Ride-Sharing)

**Difficulty:** ⭐⭐⭐⭐⭐ Hard  
**Estimated Time:** 60 minutes

**Problem Statement:**
Design a ride-sharing platform that matches riders with nearby drivers in real-time and handles payments.

**Solution Approach:**

1. **Core functionality:**
   - Riders request rides
   - Match riders with nearby drivers
   - Real-time location tracking
   - Dynamic pricing (surge pricing)
   - Payment processing
   - Trip history and ratings

2. **Critical challenges:**
   - Geospatial indexing and search
   - Real-time matching algorithms
   - Location updates at scale
   - Surge pricing calculation

3. **High-level architecture:**
```
Riders/Drivers → WebSocket Server → Matching Service
                        ↓                   ↓
            [Location Service]      [Pricing Service]
                    ↓                       ↓
            [Geospatial DB]         [Payment Service]
               (PostGIS)                    ↓
                                    [Trip/User DB]
```

**Key Topics to Cover:**
- ✅ **Geospatial indexing** (QuadTree, Geohash, S2 geometry)
- ✅ **Location updates** (efficient real-time tracking)
- ✅ **Matching algorithm** (finding nearby drivers)
- ✅ **ETA calculation** (routing, traffic data)
- ✅ **Surge pricing** (supply-demand algorithm)
- ✅ **WebSocket connections** (real-time updates)
- ✅ **Payment integration** (Stripe, payment gateways)
- ✅ **Trip state management** (FSM for trip lifecycle)
- ✅ **Notification system** (driver found, trip updates)
- ✅ **Maps integration** (Google Maps, Mapbox)

**Related Modules:**
- [Database Design](../06-database-design/)
- [Real-time Systems](../11-message-queues/)
- [Case Study: Ride Sharing](../case-studies/ride-sharing.md)

---

### 10. Design Dropbox/Google Drive (File Storage)

**Difficulty:** ⭐⭐⭐⭐ Medium-Hard  
**Estimated Time:** 60 minutes

**Problem Statement:**
Design a cloud storage system that allows users to upload, store, sync, and share files across devices.

**Solution Approach:**

1. **Core functionality:**
   - Upload/download files
   - Sync across devices
   - Share files with others
   - Version control
   - Offline access

2. **Critical challenges:**
   - Efficient syncing (only changed parts)
   - Conflict resolution (concurrent edits)
   - Large file handling
   - Bandwidth optimization

3. **High-level architecture:**
```
Clients → Load Balancer → API Servers
                ↓              ↓
        [Metadata Service] [Block Service]
                ↓              ↓
        [Metadata DB]    [Object Storage]
                           (S3/HDFS)
```

**Key Topics to Cover:**
- ✅ **File chunking** (split files into blocks for efficient sync)
- ✅ **Delta sync** (only transfer changed blocks)
- ✅ **Metadata management** (file hierarchy, permissions)
- ✅ **Conflict resolution** (last-write-wins, operational transforms)
- ✅ **Deduplication** (avoid storing duplicate files)
- ✅ **Compression** (reduce storage and bandwidth)
- ✅ **Versioning** (keeping file history)
- ✅ **Sharing and permissions** (access control)
- ✅ **Offline sync** (queue changes, sync when online)
- ✅ **Notification service** (file changes across devices)

**Related Modules:**
- [Data Partitioning](../07-data-partitioning/)
- [Consistency Patterns](../09-consistency-patterns/)
- [Database Design](../06-database-design/)

---

## Hard Questions (Complexity Level 5)

### 11. Design Google Search

**Difficulty:** ⭐⭐⭐⭐⭐ Hard  
**Estimated Time:** 60+ minutes

**Problem Statement:**
Design a web search engine that crawls, indexes, and ranks billions of web pages and serves results in milliseconds.

**Solution Approach:**

1. **Core components:**
   - Web crawler (discover and fetch pages)
   - Indexer (process and index content)
   - Ranker (rank results by relevance)
   - Query processor (handle search queries)

2. **High-level architecture:**
```
[Crawler] → [Index Builder] → [Distributed Index]
                                      ↓
User Query → [Query Processor] → [Ranker] → Results
                    ↓
            [Cache Layer]
```

**Key Topics to Cover:**
- ✅ **Web crawling** (distributed crawlers, politeness policy)
- ✅ **Indexing** (inverted index, posting lists)
- ✅ **Ranking algorithms** (PageRank, relevance scoring)
- ✅ **Distributed systems** (sharding index across many servers)
- ✅ **Query processing** (parsing, spell correction, synonyms)
- ✅ **Caching** (query results, frequent searches)
- ✅ **Freshness** (re-crawling, incremental indexing)
- ✅ **Scale** (billions of pages, millions of queries/sec)
- ✅ **Personalization** (search history, location)
- ✅ **Auto-complete** (suggestions as user types)

**Related Modules:**
- [Data Partitioning](../07-data-partitioning/)
- [Caching](../05-caching/)
- [Scalability](../03-scalability/)

---

### 12. Design Amazon (E-commerce Platform)

**Difficulty:** ⭐⭐⭐⭐⭐ Hard  
**Estimated Time:** 60+ minutes

**Problem Statement:**
Design a large-scale e-commerce platform with product catalog, shopping cart, orders, payments, and inventory management.

**Solution Approach:**

1. **Core services:**
   - Product catalog
   - Search and discovery
   - Shopping cart
   - Order processing
   - Inventory management
   - Payment processing
   - Recommendation system

2. **High-level architecture:**
```
Users → API Gateway
         ↓
[Product Service] [Cart Service] [Order Service] [Payment Service]
         ↓              ↓              ↓               ↓
    [Catalog DB]  [Cache/DB]    [Order DB]    [Payment Gateway]
                                      ↓
                            [Inventory Service]
```

**Key Topics to Cover:**
- ✅ **Product catalog** (search, filtering, categorization)
- ✅ **Shopping cart** (session management, persistence)
- ✅ **Inventory management** (real-time stock tracking)
- ✅ **Order processing** (order state machine, saga pattern)
- ✅ **Payment integration** (PCI compliance, multiple gateways)
- ✅ **Transaction handling** (ACID guarantees, two-phase commit)
- ✅ **Recommendation engine** (collaborative filtering, ML)
- ✅ **Search** (Elasticsearch, faceted search)
- ✅ **Scalability** (microservices, event-driven)
- ✅ **Fraud detection** (real-time anomaly detection)

**Related Modules:**
- [Microservices](../12-microservices/)
- [Message Queues](../11-message-queues/)
- [Database Design](../06-database-design/)
- [Case Study: E-commerce](../case-studies/ecommerce-platform.md)

---

### 13. Design Facebook News Feed

**Difficulty:** ⭐⭐⭐⭐⭐ Hard  
**Estimated Time:** 60+ minutes

**Problem Statement:**
Design a news feed system that aggregates posts from friends, pages, and groups, and ranks them by relevance.

**Solution Approach:**

1. **Core functionality:**
   - Generate personalized feed
   - Real-time updates
   - Ranking by relevance (not chronological)
   - Multiple content types (text, images, videos, links)
   - Engagement (likes, comments, shares)

2. **Critical challenges:**
   - Scale (billions of users, trillions of posts)
   - Personalization (ML-based ranking)
   - Real-time updates
   - Multiple data sources

3. **High-level architecture:**
```
User → Feed Service → [Fan-out Service] → Feed Cache
                           ↓
                    [Ranking Service]
                           ↓
                    [ML Models, Signals]
```

**Key Topics to Cover:**
- ✅ **Feed generation** (fan-out on write vs read)
- ✅ **Ranking algorithm** (EdgeRank, ML-based scoring)
- ✅ **Personalization** (user interests, engagement history)
- ✅ **Social graph** (friends, followers, pages)
- ✅ **Real-time updates** (new posts appearing in feed)
- ✅ **Content types** (handling diverse media)
- ✅ **Engagement tracking** (likes, comments, shares)
- ✅ **Caching strategy** (feed cache, post cache)
- ✅ **Privacy** (post visibility, audience targeting)
- ✅ **Scale** (billions of users, real-time processing)

**Related Modules:**
- [Caching](../05-caching/)
- [Data Partitioning](../07-data-partitioning/)
- [Message Queues](../11-message-queues/)

---

### 14. Design Ticketmaster (Event Booking)

**Difficulty:** ⭐⭐⭐⭐ Medium-Hard  
**Estimated Time:** 60 minutes

**Problem Statement:**
Design a ticket booking system that handles concurrent bookings, prevents double-booking, and manages inventory for events.

**Solution Approach:**

1. **Core functionality:**
   - Browse events and venues
   - Select seats
   - Book tickets (prevent double-booking)
   - Payment processing
   - Ticket generation (QR codes)

2. **Critical challenges:**
   - Concurrency (multiple users booking same seat)
   - Flash sales (high traffic spikes)
   - Fair access (prevent bots)
   - Inventory management

3. **High-level architecture:**
```
Users → Load Balancer → API Servers
                ↓           ↓
        [Booking Service] [Payment Service]
                ↓
        [Inventory DB with Locking]
```

**Key Topics to Cover:**
- ✅ **Concurrency control** (pessimistic vs optimistic locking)
- ✅ **Inventory management** (seat availability, reservations)
- ✅ **Queue system** (virtual waiting room for high demand)
- ✅ **Payment flow** (reservation, payment, confirmation)
- ✅ **Timeout handling** (release reserved seats after N minutes)
- ✅ **Database transactions** (ACID properties critical)
- ✅ **Rate limiting** (prevent bots, ensure fairness)
- ✅ **Scalability** (handling traffic spikes)
- ✅ **Ticket generation** (unique IDs, QR codes)
- ✅ **Fraud prevention** (duplicate bookings, fake tickets)

**Related Modules:**
- [Consistency Patterns](../09-consistency-patterns/)
- [Database Design](../06-database-design/)
- [Scalability](../03-scalability/)

---

### 15. Design LinkedIn (Professional Network)

**Difficulty:** ⭐⭐⭐⭐ Medium-Hard  
**Estimated Time:** 60 minutes

**Problem Statement:**
Design a professional networking platform with profiles, connections, job postings, and a news feed.

**Solution Approach:**

1. **Core functionality:**
   - User profiles
   - Connections (network graph)
   - Job postings and applications
   - News feed
   - Messaging
   - Search (people, jobs, companies)

2. **High-level architecture:**
```
Users → API Gateway
         ↓
[Profile Service] [Connection Service] [Job Service] [Feed Service]
         ↓               ↓                  ↓             ↓
    [User DB]      [Graph DB]         [Job DB]    [Feed Cache]
```

**Key Topics to Cover:**
- ✅ **Graph database** (social connections, degrees of separation)
- ✅ **Search** (people search, job search, skills)
- ✅ **Recommendation engine** (connection suggestions, jobs)
- ✅ **News feed** (similar to Twitter/Facebook)
- ✅ **Profile storage** (rich data, work history, skills)
- ✅ **Privacy settings** (visibility control)
- ✅ **Job matching** (candidates to jobs)
- ✅ **Messaging** (real-time chat)
- ✅ **Analytics** (profile views, post engagement)

**Related Modules:**
- [Database Design](../06-database-design/)
- [Data Partitioning](../07-data-partitioning/)
- [Caching](../05-caching/)

---

### 16. Design Airbnb (Vacation Rental)

**Difficulty:** ⭐⭐⭐⭐ Medium-Hard  
**Estimated Time:** 60 minutes

**Problem Statement:**
Design a vacation rental platform where hosts list properties and guests can search and book accommodations.

**Solution Approach:**

1. **Core functionality:**
   - Property listings (hosts)
   - Search and filtering (guests)
   - Booking and reservations
   - Reviews and ratings
   - Payment processing

2. **High-level architecture:**
```
Users → API Gateway
         ↓
[Listing Service] [Search Service] [Booking Service] [Payment Service]
         ↓               ↓               ↓                ↓
  [Listing DB]   [Elasticsearch]  [Booking DB]    [Payment Gateway]
```

**Key Topics to Cover:**
- ✅ **Geospatial search** (find properties near location)
- ✅ **Availability calendar** (prevent double-booking)
- ✅ **Search and filtering** (price, amenities, dates)
- ✅ **Booking flow** (request, approval, confirmation)
- ✅ **Payment splitting** (host payout, platform fee)
- ✅ **Reviews and ratings** (two-way reviews)
- ✅ **Pricing** (dynamic pricing, seasonal rates)
- ✅ **Recommendation** (personalized suggestions)
- ✅ **Photo storage and delivery** (CDN)
- ✅ **Fraud prevention** (fake listings, payment fraud)

**Related Modules:**
- [Database Design](../06-database-design/)
- [Message Queues](../11-message-queues/)
- [CDN](../14-cdn/)

---

### 17. Design Slack (Team Communication)

**Difficulty:** ⭐⭐⭐⭐ Medium-Hard  
**Estimated Time:** 60 minutes

**Problem Statement:**
Design a team communication platform with channels, direct messages, file sharing, and search.

**Solution Approach:**

1. **Core functionality:**
   - Channels (public/private)
   - Direct messages
   - File sharing
   - Search (messages, files)
   - Presence (online/offline)
   - Notifications

2. **High-level architecture:**
```
Clients (WebSocket) → Gateway → [Message Service] [Channel Service]
                                      ↓                  ↓
                              [Message DB]        [Channel DB]
                                      ↓
                              [Search Index]
```

**Key Topics to Cover:**
- ✅ **Real-time messaging** (WebSocket, message delivery)
- ✅ **Channel management** (public, private, DMs)
- ✅ **Message storage** (hot vs cold storage)
- ✅ **Search** (full-text search on messages)
- ✅ **File sharing** (upload, storage, sharing)
- ✅ **Presence system** (online status)
- ✅ **Notifications** (push, email, mobile)
- ✅ **Threads** (threaded conversations)
- ✅ **Integrations** (bots, webhooks, apps)
- ✅ **Read receipts** (message seen status)

**Related Modules:**
- [Message Queues](../11-message-queues/)
- [Consistency Patterns](../09-consistency-patterns/)
- [CDN](../14-cdn/)

---

### 18. Design Pinterest (Visual Discovery)

**Difficulty:** ⭐⭐⭐⭐ Medium-Hard  
**Estimated Time:** 60 minutes

**Problem Statement:**
Design a visual discovery platform where users can save and organize images (pins) on boards.

**Solution Approach:**

1. **Core functionality:**
   - Save pins (images/videos)
   - Create boards
   - Follow users and boards
   - Personalized feed
   - Search and discovery

2. **High-level architecture:**
```
Users → CDN (images) → API Gateway
                          ↓
        [Pin Service] [Board Service] [Feed Service] [Search Service]
              ↓              ↓              ↓              ↓
        [Object Storage] [Board DB]   [Feed Cache]  [Elasticsearch]
```

**Key Topics to Cover:**
- ✅ **Image storage and delivery** (CDN, object storage)
- ✅ **Image processing** (thumbnails, sizes, optimization)
- ✅ **Board organization** (pins to boards mapping)
- ✅ **Feed generation** (personalized recommendations)
- ✅ **Visual search** (image similarity, ML)
- ✅ **Graph storage** (user follows, board follows)
- ✅ **Recommendation system** (pins you might like)
- ✅ **Search** (image search, keyword search)
- ✅ **Analytics** (pin engagement, impressions)

**Related Modules:**
- [CDN](../14-cdn/)
- [Caching](../05-caching/)
- [Database Design](../06-database-design/)

---

### 19. Design Zoom (Video Conferencing)

**Difficulty:** ⭐⭐⭐⭐⭐ Hard  
**Estimated Time:** 60+ minutes

**Problem Statement:**
Design a video conferencing platform that supports real-time audio/video communication for multiple participants.

**Solution Approach:**

1. **Core functionality:**
   - Create/join meetings
   - Real-time audio/video streaming
   - Screen sharing
   - Chat
   - Recording

2. **Architecture approaches:**
   - **P2P** (peer-to-peer): Good for 1-1, doesn't scale
   - **SFU** (Selective Forwarding Unit): Server routes streams (recommended)
   - **MCU** (Multipoint Control Unit): Server mixes streams

3. **High-level architecture:**
```
Clients → WebRTC → SFU Servers → Media Servers
                        ↓
                [Signaling Service]
                        ↓
                [Meeting DB]
```

**Key Topics to Cover:**
- ✅ **WebRTC** (peer-to-peer media streaming)
- ✅ **SFU architecture** (media routing strategy)
- ✅ **Signaling** (establishing connections)
- ✅ **Codec selection** (VP8, H.264 for video; Opus for audio)
- ✅ **Bandwidth adaptation** (adjust quality based on network)
- ✅ **Recording** (server-side recording, storage)
- ✅ **Screen sharing** (screen capture streaming)
- ✅ **Chat** (text messaging during call)
- ✅ **Scalability** (handling large meetings)
- ✅ **Quality of Service** (jitter buffers, packet loss handling)

**Related Modules:**
- [Real-time Systems](../11-message-queues/)
- [Scalability](../03-scalability/)
- [Load Balancing](../04-load-balancing/)

---

### 20. Design TikTok (Short Video Platform)

**Difficulty:** ⭐⭐⭐⭐⭐ Hard  
**Estimated Time:** 60+ minutes

**Problem Statement:**
Design a short-form video sharing platform with an algorithmic feed that keeps users engaged.

**Solution Approach:**

1. **Core functionality:**
   - Upload short videos (15-60 seconds)
   - Infinite scroll feed (For You page)
   - Like, comment, share
   - Follow users
   - Algorithmic recommendations

2. **Critical differentiator:**
   - **Algorithmic feed** (not chronological, not social graph)
   - Uses ML to predict what videos you'll engage with

3. **High-level architecture:**
```
Users → CDN (videos) → API Gateway
                          ↓
    [Upload Service] [Feed Service] [Recommendation Engine]
           ↓                ↓                 ↓
    [Video Storage]  [Video Metadata]   [ML Models]
    [Transcoding]        [Analytics]    [User Signals]
```

**Key Topics to Cover:**
- ✅ **Video upload and storage** (object storage, S3)
- ✅ **Video transcoding** (multiple formats, resolutions)
- ✅ **CDN delivery** (global distribution, edge caching)
- ✅ **Recommendation algorithm** (ML-based, not social graph)
- ✅ **User signals** (watch time, completion rate, engagement)
- ✅ **Feed generation** (personalized, infinite scroll)
- ✅ **Video processing** (thumbnails, effects, filters)
- ✅ **Analytics** (video performance, user behavior)
- ✅ **Content moderation** (ML for inappropriate content)
- ✅ **Scalability** (billions of videos, millions of users)

**Related Modules:**
- [CDN](../14-cdn/)
- [Message Queues](../11-message-queues/)
- [Data Partitioning](../07-data-partitioning/)
- [Caching](../05-caching/)

---

## How to Practice These Questions

### Recommended Practice Sequence

**Week 1-2: Easy Questions (1-4)**
- Focus on fundamentals
- Get comfortable with basic components
- Practice drawing diagrams

**Week 3-4: Medium Questions (5-10)**
- Build on fundamentals
- Practice trade-off discussions
- Time yourself (45-60 minutes)

**Week 5-6: Hard Questions (11-20)**
- Challenge yourself with complex systems
- Focus on depth and breadth
- Do mock interviews

### Practice Tips

1. **Time yourself** - Stick to 45-60 minutes
2. **Don't memorize solutions** - Understand the approach
3. **Draw diagrams** - Visual representation is critical
4. **Discuss trade-offs** - Every decision has pros/cons
5. **Practice out loud** - Simulate real interview conditions
6. **Get feedback** - Practice with peers or mentors
7. **Iterate** - Redo questions after a few weeks

### Study Approach for Each Question

1. **Attempt on your own** (30-45 minutes)
   - Write requirements
   - Draw architecture
   - Identify key components

2. **Review sample solutions** (15-20 minutes)
   - Compare your approach
   - Note what you missed
   - Understand alternatives

3. **Study related modules** (1-2 hours)
   - Deep dive into concepts
   - Understand underlying technologies

4. **Practice again** (30-45 minutes)
   - Attempt the same question after 1-2 weeks
   - See how your approach improved

---

Remember: Interviewers aren't looking for perfect solutions. They want to see how you think, communicate, and make trade-offs. Focus on demonstrating your problem-solving process!

Happy practicing! 🚀
