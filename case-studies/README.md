# System Design Case Studies

Real-world examples of how popular systems are designed and built.

## Available Case Studies

### Beginner Level

#### 1. [URL Shortener](./url-shortener.md) (~2000 words)
- Complete design walkthrough with all components explained
- Hash generation and collision handling
- Database design and caching strategies
- Scale considerations (billions of URLs)
- API design and redirect optimization
- Base62 encoding, Redis caching
- Interview talking points

### Intermediate Level

#### 2. [Social Media Feed](./social-media-feed.md) (~3000 words)
- Twitter/Facebook timeline design
- Fan-out strategies (on-write vs on-read vs hybrid)
- Multi-level caching layers
- Timeline generation at scale
- Handling celebrity users with millions of followers
- Real-time updates and notifications
- Feed ranking algorithms

#### 3. [Video Streaming Service](./video-streaming.md) (~3000 words)
- YouTube/Netflix architecture
- CDN usage and multi-tier caching strategy
- Adaptive bitrate streaming (HLS/DASH)
- Video transcoding pipeline (FFmpeg at scale)
- Upload and processing flow
- Global content delivery
- Storage optimization

### Intermediate-Advanced Level

#### 4. [Ride-Sharing App](./ride-sharing.md) (~3500 words)
- Uber/Lyft system design
- Real-time driver-rider matching algorithm
- Location tracking with geospatial indexing (Redis GEORADIUS, S2, H3)
- Dynamic pricing (surge pricing)
- Payment processing with two-phase commit
- WebSocket for real-time updates
- Handling race conditions in driver assignment

### Advanced Level

#### 5. [E-commerce Platform](./ecommerce-platform.md) (~4000 words)
- Amazon/eBay architecture
- Inventory management with strong consistency
- Payment processing and fraud detection
- Multi-warehouse optimization
- Two-phase commit for transactions
- Search with Elasticsearch
- Order fulfillment and tracking
- Handling Black Friday scale (2000+ orders/sec)

#### 6. [Distributed File Storage](./distributed-file-storage.md) (~4000 words)
- Dropbox/Google Drive design
- File chunking and deduplication (30-50% storage savings)
- Conflict resolution strategies (keep-both, three-way merge)
- Sync protocol and delta sync
- Real-time synchronization with WebSocket
- Multi-device support
- Version history and recovery
- 99.999999999% durability

### Additional Case Studies

#### 7. [Twitter/X - Social Media Platform](./twitter.md)
- Timeline generation
- Tweet storage and retrieval
- Following/followers graph
- Real-time feed updates
- Scaling challenges

## How to Use These Case Studies

### Learning Path by Difficulty

**Start Here (Beginner):**
1. Begin with [URL Shortener](./url-shortener.md) to learn fundamentals
   - Core concepts: hashing, caching, database design
   - Complexity: Low, ~2000 words
   - Time: 1-2 hours to read and understand

**Progress To (Intermediate):**
2. Move to [Social Media Feed](./social-media-feed.md) for distributed systems
   - Learn: Fan-out strategies, caching layers, eventual consistency
   - Complexity: Medium, ~3000 words
   - Time: 2-3 hours

3. Then [Video Streaming](./video-streaming.md) for content delivery
   - Learn: CDN strategies, transcoding, adaptive streaming
   - Complexity: Medium, ~3000 words
   - Time: 2-3 hours

**Advance To (Intermediate-Advanced):**
4. Study [Ride-Sharing App](./ride-sharing.md) for real-time systems
   - Learn: Geospatial indexing, real-time matching, WebSocket
   - Complexity: Medium-High, ~3500 words
   - Time: 3-4 hours

**Master (Advanced):**
5. Tackle [E-commerce Platform](./ecommerce-platform.md) for transactions
   - Learn: Strong consistency, payment processing, inventory management
   - Complexity: High, ~4000 words
   - Time: 4-5 hours

6. Finish with [Distributed File Storage](./distributed-file-storage.md)
   - Learn: Deduplication, conflict resolution, sync protocols
   - Complexity: High, ~4000 words
   - Time: 4-5 hours

### How to Study Each Case

1. **Read the problem statement** - Understand requirements (functional & non-functional)
2. **Study capacity estimation** - Learn how to calculate storage, bandwidth, QPS
3. **Analyze the architecture** - Understand component interactions
4. **Deep dive into key components** - Study implementation details
5. **Review trade-offs** - Why these choices? What alternatives exist?
6. **Study scaling strategies** - How does it grow from 0 to millions of users?
7. **Practice interview talking points** - Prepare for system design interviews
8. **Design variations** - Try designing with different constraints

## What Each Case Study Includes

Every case study follows a comprehensive structure:

1. **Problem Statement**: Clear description of what needs to be built
2. **Requirements Analysis**: 
   - Functional requirements (features)
   - Non-functional requirements (scale, performance, availability)
3. **Capacity Estimation**: 
   - Storage calculations
   - Traffic estimates
   - Bandwidth requirements
4. **High-Level Architecture**: Visual diagram with all major components
5. **Detailed Component Design**: Deep dive into critical components with code examples
6. **API Design**: RESTful API endpoints with request/response examples
7. **Database Schema**: Complete schema with indexes and rationale
8. **Trade-off Discussions**: Pros/cons of different approaches with recommendations
9. **Scaling Strategies**: Evolution from startup to massive scale (4 phases)
10. **Interview Talking Points**: Key questions, bottlenecks, optimizations to discuss

## Key Concepts Covered Across Case Studies

### Data Structures & Algorithms
- **Hash functions**: URL shortener (Base62 encoding)
- **Geospatial indexing**: Ride-sharing (Redis GEORADIUS, S2, H3)
- **Tries**: E-commerce search autocomplete
- **Bloom filters**: Duplicate detection, cache optimization
- **Consistent hashing**: Load balancing, cache distribution

### System Design Patterns
- **Fan-out on write vs read**: Social media feed
- **CQRS**: Separate read/write paths in e-commerce
- **Event sourcing**: Order tracking, audit logs
- **Saga pattern**: Distributed transactions in e-commerce
- **Circuit breaker**: Fault tolerance in all systems

### Caching Strategies
- **Multi-level caching**: Social media, video streaming
- **Cache-aside**: URL shortener
- **Write-through**: E-commerce inventory
- **CDN edge caching**: Video streaming, file storage

### Database Design
- **Sharding**: Social media feed, e-commerce orders
- **Replication**: All systems for high availability
- **Denormalization**: Read-heavy workloads
- **Time-series data**: Location tracking, analytics

### Consistency Models
- **Strong consistency**: E-commerce inventory, payments
- **Eventual consistency**: Social media feeds, file metadata
- **Conflict resolution**: Distributed file storage

### Scalability Techniques
- **Horizontal scaling**: Add more servers
- **Vertical scaling**: Bigger machines
- **Database sharding**: Partition data
- **Read replicas**: Handle read traffic
- **Asynchronous processing**: Background jobs with Kafka

## Interview Preparation Guide

### By Company Type

**Social Media Companies (Meta, Twitter, LinkedIn):**
- Focus on: Social Media Feed, Real-time systems
- Key topics: Fan-out strategies, caching, eventual consistency

**E-commerce Companies (Amazon, eBay, Shopify):**
- Focus on: E-commerce Platform
- Key topics: Inventory management, transactions, payment processing

**Streaming Companies (Netflix, YouTube, Spotify):**
- Focus on: Video Streaming
- Key topics: CDN, transcoding, adaptive bitrate

**Ride-sharing/Maps (Uber, Lyft, Google Maps):**
- Focus on: Ride-Sharing App
- Key topics: Geospatial indexing, real-time matching, location tracking

**Cloud Storage (Dropbox, Google Drive, OneDrive):**
- Focus on: Distributed File Storage
- Key topics: Deduplication, sync, conflict resolution

**General Product Companies:**
- Start with: URL Shortener (fundamentals)
- Progress through all case studies for breadth

### Interview Practice Routine

**Week 1-2: Fundamentals**
- Study URL Shortener
- Practice capacity estimation
- Learn to draw architecture diagrams

**Week 3-4: Core Systems**
- Study Social Media Feed and Video Streaming
- Practice explaining trade-offs
- Mock interviews on these systems

**Week 5-6: Advanced Topics**
- Study Ride-Sharing and E-commerce
- Practice complex scenarios
- Focus on edge cases

**Week 7-8: Mastery**
- Study Distributed File Storage
- Design variants of all systems
- Conduct mock interviews

## Next Steps

- **Beginner?** Start with [URL Shortener](./url-shortener.md)
- **Interview Prep?** Review [Interview Prep Guide](../interview-prep/)
- **Want Templates?** Check [Design Templates](../design-templates/)
- **Need More Examples?** Study [Twitter case study](./twitter.md) for another perspective
