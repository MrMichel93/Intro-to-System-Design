# Design Exercise: Instagram Photo Storage System

## 1. Design Problem

### Problem Statement
Design a photo storage and retrieval system for a social media platform like Instagram. The system must efficiently store, process, and serve photos uploaded by users, while handling massive scale and ensuring high availability.

### Context and Constraints
- Users upload photos from mobile and web clients
- Photos are viewed much more frequently than they are uploaded (read-heavy workload)
- Photos should load quickly regardless of user location
- Original photos must be preserved, but multiple sizes/formats are needed for different devices
- Users expect photos to be available immediately after upload
- The system must be cost-effective as storage costs are significant
- Photos are rarely deleted once uploaded

### Requirements

#### Functional Requirements
- Users can upload photos (JPEG, PNG, up to 10MB)
- Users can view their own photos and photos from followed users
- Generate multiple thumbnail sizes (small, medium, large) for responsive display
- Support photo filters and basic editing
- Store photo metadata (caption, location, timestamp, tags)
- Photos should be accessible via unique URLs
- Support photo search by user, tags, or location

#### Non-Functional Requirements
- **Scale**: 500 million users, 100 million photos uploaded per day
- **Performance**: Upload < 3 seconds, display < 500ms
- **Availability**: 99.99% uptime (< 1 hour downtime per year)
- **Durability**: Zero data loss - photos must never disappear
- **Storage**: ~50PB total storage needed (500 billion photos × 100KB avg)
- **Bandwidth**: Peak upload: 50,000 photos/second, peak view: 500,000 photos/second
- **Reliability**: System should gracefully handle component failures

## 2. Guided Questions

### Understanding the Problem
1. **What makes this system read-heavy vs write-heavy?**
   - Hint: Consider how often users view photos vs. upload them
   - Think about: News feeds, profile viewing, photo browsing

2. **Why can't we just store all photos on a single server?**
   - Hint: Consider the scale - 100 million photos per day
   - Think about: Disk capacity, bandwidth limits, single point of failure

3. **What types of data do we need to store?**
   - Hint: Not just the photo files themselves
   - Think about: User information, photo metadata, relationships

### System Design
4. **Where should we store photos vs. metadata?**
   - Hint: Different storage systems excel at different tasks
   - Think about: Blob storage vs. databases, access patterns

5. **How do we ensure photos load quickly worldwide?**
   - Hint: Users in Tokyo shouldn't wait for photos from California servers
   - Think about: CDN, geographic distribution, caching

6. **How do we generate thumbnails efficiently?**
   - Hint: Should thumbnail generation block the user's upload request?
   - Think about: Asynchronous processing, message queues

7. **How do we prevent duplicate uploads and save storage?**
   - Hint: Multiple users might upload the same photo
   - Think about: Content hashing, deduplication

## 3. Step-by-Step Guidance

### Step 1: High-Level Architecture
Start with the major components:
- **Upload Service**: Handles photo uploads from clients
- **Storage Service**: Manages photo storage (blob storage)
- **Metadata Service**: Stores and queries photo information
- **Processing Service**: Generates thumbnails and applies filters
- **CDN**: Distributes photos globally for fast access

### Step 2: Upload Flow Design
Consider the photo upload process:
1. User selects photo from device
2. Client compresses/resizes if needed
3. Upload to server with metadata
4. Store original photo
5. Trigger thumbnail generation (async)
6. Update database with metadata
7. Return success to user

**Key Decision**: Should we process photos synchronously or asynchronously?
- Synchronous: User waits for all thumbnails → Slow but simpler
- Asynchronous: Return immediately, process in background → Fast but more complex

### Step 3: Storage Strategy
**Photo Storage Options**:
- **Local File System**: Simple but doesn't scale, single point of failure
- **Network Attached Storage (NAS)**: Shared storage but limited capacity
- **Object Storage (S3, GCS)**: Unlimited scale, high durability, cost-effective ✓

**Why Object Storage?**
- Built for storing billions of objects
- Automatic replication across data centers
- Pay only for what you use
- Built-in redundancy (99.999999999% durability)
- HTTP access for easy CDN integration

### Step 4: Metadata Storage
Photos have structured metadata that needs to be queryable:
```
Photo:
- photo_id (UUID)
- user_id
- caption
- location (lat, lng)
- timestamp
- tags
- file_path (S3 URL)
- dimensions
- file_size
```

**Database Choice**:
- **SQL (PostgreSQL)**: Good for structured data, complex queries
- **NoSQL (Cassandra)**: Better for massive scale, simpler queries

For Instagram scale, **Cassandra** or similar NoSQL is preferred for:
- Horizontal scalability (add more nodes easily)
- High write throughput
- Geographic distribution

### Step 5: Thumbnail Generation
Use asynchronous processing:
1. User uploads photo → Immediate response
2. Upload service publishes message to queue
3. Worker services consume queue messages
4. Workers generate multiple thumbnail sizes
5. Upload thumbnails to storage
6. Update metadata database

**Benefits**:
- Non-blocking uploads (better UX)
- Elastic scaling (add/remove workers based on queue depth)
- Retry logic for failed processing
- Separates concerns

### Step 6: Efficient Retrieval
**Challenges**:
- 500,000 photos/second at peak
- Users distributed globally
- Low latency requirements

**Solutions**:
1. **CDN**: Cache photos at edge locations near users
2. **Multi-tier Caching**: Browser → CDN → App Cache → Storage
3. **Read Replicas**: Multiple database copies for queries
4. **Sharding**: Distribute data by user_id or geo-region

### Step 7: Scalability Patterns
**Horizontal Scaling Points**:
- Upload Service: Add more servers behind load balancer
- Worker Service: Add more workers for thumbnail processing
- Database: Add read replicas, implement sharding
- Storage: Object storage scales automatically

**Bottleneck Identification**:
- Monitor: Request rates, response times, queue depths
- Scale: Add capacity to components hitting limits

## 4. Sample Solution

### Architecture Diagram
```
┌─────────────┐
│   Client    │
│ (Mobile/Web)│
└──────┬──────┘
       │ HTTPS
       ↓
┌──────────────────┐
│   Load Balancer  │
└──────┬───────────┘
       │
   ┌───┴───┬──────────┐
   ↓       ↓          ↓
[Upload] [API]  [Web Servers]
Service  Service
   │       │
   │       └──────┐
   │              ↓
   │     ┌────────────────┐
   │     │   Redis Cache  │
   │     │  (Metadata)    │
   │     └────────┬───────┘
   │              ↓
   │     ┌────────────────┐
   │     │   Cassandra    │
   │     │   (Metadata)   │
   │     └────────────────┘
   │
   ├───→ [Message Queue]
   │        (Kafka/SQS)
   │             ↓
   │     ┌───────────────┐
   │     │Image Processing│
   │     │   Workers      │
   │     └───────┬────────┘
   │             ↓
   └────────→ [Object Storage]
              (Amazon S3)
                   ↓
              [CloudFront CDN]
                   ↓
            [Global Users]
```

### Complete Design

#### 1. Upload Flow
```
1. User uploads photo via mobile app
2. Client sends photo to Upload Service
3. Upload Service:
   - Validates file (size, format)
   - Generates unique photo_id (UUID)
   - Uploads original to S3: /photos/original/{user_id}/{photo_id}.jpg
   - Stores metadata in Cassandra
   - Publishes message to Kafka: {photo_id, user_id, s3_path}
   - Returns success response with photo_id
4. Image Processing Workers:
   - Consume message from Kafka
   - Download original from S3
   - Generate thumbnails (150px, 320px, 640px, 1080px)
   - Upload thumbnails to S3: /photos/thumbnail/{size}/{photo_id}.jpg
   - Update metadata in Cassandra (mark as processed)
5. User can view photo immediately (original)
   - Thumbnails available within seconds
```

#### 2. Retrieval Flow
```
1. User requests photo feed
2. API Service queries Cassandra for photo metadata
3. Returns list of photo URLs (CDN URLs)
4. Client requests photos from CDN
5. CDN serves from cache (if available) or fetches from S3
6. Browser caches photo locally
```

#### 3. Database Schema (Cassandra)

```cql
-- Photos table (partitioned by user_id for locality)
CREATE TABLE photos (
    user_id uuid,
    photo_id uuid,
    timestamp timestamp,
    caption text,
    location text,
    tags set<text>,
    original_url text,
    thumbnail_urls map<text, text>,
    dimensions text,
    file_size bigint,
    processed boolean,
    PRIMARY KEY (user_id, timestamp, photo_id)
) WITH CLUSTERING ORDER BY (timestamp DESC);

-- Photo index by photo_id
CREATE TABLE photo_by_id (
    photo_id uuid PRIMARY KEY,
    user_id uuid,
    timestamp timestamp
);

-- User feed (denormalized for fast access)
CREATE TABLE user_feed (
    user_id uuid,
    timestamp timestamp,
    photo_id uuid,
    owner_id uuid,
    thumbnail_url text,
    PRIMARY KEY (user_id, timestamp, photo_id)
) WITH CLUSTERING ORDER BY (timestamp DESC);

-- Tags index
CREATE TABLE photos_by_tag (
    tag text,
    timestamp timestamp,
    photo_id uuid,
    user_id uuid,
    PRIMARY KEY (tag, timestamp, photo_id)
) WITH CLUSTERING ORDER BY (timestamp DESC);
```

#### 4. Storage Structure (S3)
```
bucket: instagram-photos
├── /photos/original/{user_id}/{photo_id}.jpg
├── /photos/thumbnails/150/{user_id}/{photo_id}.jpg
├── /photos/thumbnails/320/{user_id}/{photo_id}.jpg
├── /photos/thumbnails/640/{user_id}/{photo_id}.jpg
└── /photos/thumbnails/1080/{user_id}/{photo_id}.jpg
```

#### 5. API Endpoints
```
POST   /api/photos/upload         - Upload new photo
GET    /api/photos/{photo_id}     - Get photo details
GET    /api/photos/feed           - Get user's photo feed
GET    /api/photos/user/{user_id} - Get user's photos
DELETE /api/photos/{photo_id}     - Delete photo
POST   /api/photos/{photo_id}/like - Like photo
GET    /api/photos/search?tag=sunset - Search by tag
```

### Design Choices Explained

#### Why Object Storage (S3) over File System?
**Advantages**:
- Infinite scalability without manual intervention
- Built-in replication (cross-region, multi-AZ)
- 99.999999999% durability guarantee
- Cost-effective at scale (~$0.023/GB/month)
- No server management required
- Direct integration with CDN

**Trade-offs**:
- Higher latency than local disk (~50-100ms)
- Cost per request ($0.0004 per 1000 requests)
- Cannot modify files in place

**Why it works**: Instagram's access pattern (write once, read many) is perfect for object storage. The latency is mitigated by CDN caching.

#### Why Cassandra over PostgreSQL?
**Advantages**:
- Linear scalability (add nodes to increase capacity)
- High write throughput (millions of writes/sec)
- Multi-datacenter replication
- Tunable consistency
- No single point of failure

**Trade-offs**:
- No joins (must denormalize data)
- Eventual consistency by default
- More complex to operate
- Limited query flexibility

**Why it works**: Instagram's scale requires horizontal scalability. Simple query patterns (get user's photos, get photo by ID) work well with Cassandra's data model.

#### Why Asynchronous Thumbnail Generation?
**Advantages**:
- Fast user experience (upload returns in seconds)
- Elastic processing (scale workers independently)
- Resilience (retry failed jobs)
- Cost optimization (run workers only when needed)

**Trade-offs**:
- Added complexity (message queue, workers)
- Eventual consistency (thumbnails not immediately available)
- Need to handle failures and retries

**Why it works**: Users care about fast uploads. They won't notice if a thumbnail is ready in 2 seconds vs immediately. This allows decoupling and scaling.

### Alternative Approaches

#### Alternative 1: Microservices Architecture
Instead of monolithic services, split into:
- Upload Service
- Metadata Service  
- Image Processing Service
- Feed Service
- Search Service

**Pros**: Independent scaling, technology flexibility, team autonomy
**Cons**: More complexity, network overhead, harder to debug
**When to use**: Teams > 20 engineers, distinct scaling needs per service

#### Alternative 2: Edge Computing for Image Processing
Process images at CDN edge instead of centrally:
- Cloudflare Workers, Lambda@Edge
- Generate thumbnails on-demand
- Cache generated thumbnails at edge

**Pros**: Lower latency, no central processing cluster
**Cons**: Higher compute costs, limited processing time, harder to update
**When to use**: Global user base, cost less important than speed

#### Alternative 3: Distributed File System (HDFS, Ceph)
Use distributed file system instead of object storage:
- HDFS with Hadoop ecosystem
- Ceph for block/object/file storage

**Pros**: More control, lower costs at massive scale
**Cons**: Complex to operate, need dedicated team
**When to use**: Cost optimization priority, expertise available

### Trade-off Analysis

| Aspect | Our Choice | Alternative | Trade-off |
|--------|------------|-------------|-----------|
| **Storage** | S3 (Object) | Distributed FS | Ease of operation vs. cost at extreme scale |
| **Database** | Cassandra | PostgreSQL + Sharding | Scalability vs. query flexibility |
| **Caching** | Multi-tier (CDN+Redis) | Single cache layer | Performance vs. complexity |
| **Processing** | Async (Queue+Workers) | Synchronous | UX speed vs. system complexity |
| **Architecture** | Service-oriented | Microservices | Simplicity vs. independent scaling |

## 5. Extension Challenges

### Challenge 1: 10x User Growth (5 billion users)
**Current**: 500 million users, 100 million photos/day
**New Scale**: 5 billion users, 1 billion photos/day

**What would you change?**
<details>
<summary>Hints and Solution</summary>

**Challenges**:
- 10x storage (500PB → 5000PB = 5EB)
- 10x write throughput (50K → 500K uploads/sec)
- 10x read throughput (500K → 5M photo views/sec)
- Database hotspots (popular users/photos)

**Solutions**:
1. **Geographic Sharding**: 
   - Separate clusters per region (US, EU, Asia)
   - Route users to nearest cluster
   - Reduce cross-region traffic

2. **Advanced Database Sharding**:
   - Shard by user_id range across multiple Cassandra clusters
   - 100+ Cassandra nodes per cluster
   - Automated shard rebalancing

3. **Intelligent Caching**:
   - Implement multi-level cache hierarchy
   - Predictive cache warming (ML-based)
   - Cache popular photos in memory (Redis)

4. **CDN Optimization**:
   - Multi-CDN strategy (CloudFront + Fastly + Cloudflare)
   - Custom CDN for cost optimization
   - Smart routing based on CDN performance

5. **Database Write Optimization**:
   - Batch writes where possible
   - Async replication across regions
   - Write buffers to handle traffic spikes
</details>

### Challenge 2: Global Scale with Low Latency
**Requirement**: Users worldwide should see photos in < 200ms

**What would you change?**
<details>
<summary>Hints and Solution</summary>

**Challenges**:
- Network latency (US to Australia: 150-200ms)
- CDN cache misses still hit origin
- Database queries cross regions

**Solutions**:
1. **Multi-Region Deployment**:
   - Deploy full stack in multiple regions (US, EU, Asia, SA)
   - Each region has own: Upload service, API service, Database replica, S3 bucket
   - Users routed to nearest region via GeoDNS

2. **Active-Active Architecture**:
   - All regions can handle uploads and reads
   - Cross-region async replication for photos and metadata
   - Conflict resolution for rare edge cases

3. **Edge Computing**:
   - Use Lambda@Edge or Cloudflare Workers
   - Process small edits (crop, rotate) at edge
   - Generate thumbnails on-demand at edge

4. **Smart Routing**:
   - Monitor latency and route to fastest region (not just nearest)
   - Failover to secondary region if primary is degraded
   - Load balancing across regions

5. **Optimistic UI**:
   - Show uploaded photo immediately (from local cache)
   - Sync to server in background
   - Handle conflicts gracefully
</details>

### Challenge 3: Strict SLA (99.99% availability, zero data loss)
**Requirement**: System must never lose photos, max 52 minutes downtime/year

**What would you change?**
<details>
<summary>Hints and Solution</summary>

**Challenges**:
- Component failures (servers, disks, networks)
- Data center outages
- Human errors (bad deploys, deleted data)
- Disaster scenarios

**Solutions**:
1. **Multi-Region Redundancy**:
   - Store each photo in 3+ regions automatically
   - S3 cross-region replication enabled
   - Database replicated across 5+ data centers

2. **Automated Failover**:
   - Health checks at every layer
   - Automatic removal of unhealthy nodes
   - Circuit breakers to prevent cascade failures
   - Traffic shifted to healthy regions within seconds

3. **Data Integrity**:
   - Checksum verification on every upload
   - Regular data integrity scans
   - Versioning enabled (recover from accidental deletes)
   - Write-ahead logging for database

4. **Backup Strategy**:
   - Daily snapshots of databases
   - Point-in-time recovery capability
   - Immutable backups (cannot be deleted)
   - Regular restore testing

5. **Operational Excellence**:
   - Gradual rollouts (canary deployments)
   - Automated rollbacks on errors
   - Chaos engineering (intentionally break things to test)
   - Comprehensive monitoring and alerting

6. **Human Error Protection**:
   - Multi-person approval for production changes
   - Immutable infrastructure (no manual changes)
   - Soft deletes (mark as deleted, actual delete after 30 days)
   - Audit logs for all operations
</details>

### Challenge 4: Cost Optimization
**Requirement**: Reduce infrastructure costs by 40% while maintaining performance

**What would you change?**
<details>
<summary>Hints and Solution</summary>

**Current Costs** (estimated for 500M users):
- Storage (S3): $1.15M/month (50PB @ $0.023/GB)
- CDN (CloudFront): $850K/month (50PB transfer)
- Database (Cassandra): $500K/month (200 nodes)
- Compute (EC2): $300K/month (upload/API services)
- **Total**: ~$2.8M/month

**Optimization Strategies**:

1. **Storage Tier Optimization** (-30% storage cost):
   - Hot data (< 30 days): S3 Standard
   - Warm data (30-365 days): S3 Intelligent-Tiering
   - Cold data (> 1 year): S3 Glacier Instant Retrieval
   - Savings: ~$350K/month

2. **Intelligent Compression** (-20% storage):
   - WebP format for photos (30-40% smaller than JPEG)
   - AVIF for supported clients (50% smaller)
   - Aggressive compression for older photos
   - Savings: ~$230K/month

3. **Deduplication** (-10% storage):
   - Hash-based deduplication
   - Single storage of viral photos
   - Savings: ~$115K/month

4. **CDN Optimization** (-25% CDN cost):
   - Multi-CDN strategy (cheaper providers for less critical)
   - Private CDN network for highest traffic
   - Optimal cache TTLs
   - Savings: ~$210K/month

5. **Compute Optimization** (-40% compute):
   - Spot instances for thumbnail workers
   - Auto-scaling based on actual load
   - Reserved instances for predictable workloads
   - ARM-based instances (Graviton)
   - Savings: ~$120K/month

6. **Database Optimization** (-20% database):
   - Right-size instances based on actual usage
   - Archive old/inactive data
   - Optimize queries to reduce nodes
   - Savings: ~$100K/month

**Total Savings**: ~$1.125M/month (40% reduction) ✓

**Trade-offs**:
- Some increased latency for old photos (Glacier)
- More complexity in storage tier management
- Slightly lower quality for very old photos
- More vendor management (multi-CDN)
</details>

### Challenge 5: Video Support
**Requirement**: Add video upload and streaming (15-second videos)

**What would you change?**
<details>
<summary>Hints and Solution</summary>

**New Challenges**:
- Videos are 10-100x larger than photos
- Need to transcode to multiple formats/bitrates
- Streaming requires different delivery method
- Storage costs increase dramatically

**Solutions**:
1. **Video Processing Pipeline**:
   ```
   Upload → S3 → Trigger Lambda → MediaConvert
   → Multiple formats (720p, 1080p, multiple bitrates)
   → Thumbnail extraction (first frame)
   → Store processed videos in S3
   ```

2. **Adaptive Bitrate Streaming**:
   - HLS (HTTP Live Streaming) or DASH
   - Multiple quality levels
   - Client automatically selects based on bandwidth
   - CDN-friendly (uses standard HTTP)

3. **Storage Strategy**:
   - Separate S3 bucket for videos
   - More aggressive tiering (move to Glacier faster)
   - Delete processing artifacts after final output

4. **Processing Infrastructure**:
   - AWS MediaConvert or custom FFmpeg workers
   - Elastic scaling based on queue depth
   - Priority queue (verified users get faster processing)

5. **Database Changes**:
   ```cql
   CREATE TABLE videos (
       video_id uuid PRIMARY KEY,
       user_id uuid,
       duration int,
       formats map<text, text>,  -- resolution -> URL
       thumbnail_url text,
       processing_status text,
       view_count counter
   );
   ```

6. **New API Endpoints**:
   ```
   POST   /api/videos/upload     - Upload video
   GET    /api/videos/{id}/stream - Get streaming manifest
   POST   /api/videos/{id}/view  - Increment view counter
   ```

**Cost Implications**:
- 15-second video: ~5-10MB (10-20x photo size)
- Transcoding: ~$0.015 per minute
- Increased storage: 10x
- Increased CDN: 10x bandwidth
- Need to set limits (duration, size) to control costs
</details>

---

## Summary

This design exercise demonstrates how to build a highly scalable photo storage system by:
- Using object storage (S3) for durability and scalability
- Implementing async processing for better UX and scalability
- Leveraging CDN for global performance
- Choosing the right database (Cassandra) for scale
- Designing for failure (redundancy, replication)
- Optimizing costs through intelligent tiering and caching

**Key Takeaways**:
1. **Separate concerns**: Storage, metadata, processing are independent
2. **Think in terms of trade-offs**: Speed vs. cost, consistency vs. availability
3. **Design for scale from the start**: Easier than retrofitting
4. **Use managed services**: Focus on business logic, not infrastructure
5. **Monitor everything**: You can't optimize what you don't measure
