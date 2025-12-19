# Back-of-Envelope Examples

Real-world examples of capacity estimation and calculations for different systems.

## Example 1: Twitter-Scale System

### Given Requirements
- **Users**: 400 million monthly active users (MAU)
- **Daily active users (DAU)**: 200 million (50% of MAU)
- **Tweets per user per day**: 0.5 (average, including silent users)
- **Read/Write ratio**: 100:1 (people read 100 tweets for every 1 they post)
- **Tweet size**: 280 characters ≈ 280 bytes (text only)
- **With media**: 10% of tweets include an image (200 KB average)

### Calculations

#### 1. Write Traffic (Tweets Posted)
```
Daily tweets = 200M users × 0.5 tweets = 100M tweets/day
Tweets per second = 100M / 86,400 ≈ 100M / 100K = 1,000 tweets/sec (average)
Peak (3x average) = 3,000 tweets/sec
```

#### 2. Read Traffic (Timeline Views)
```
Daily reads = 100M tweets × 100 (read/write ratio) = 10B reads/day
Reads per second = 10B / 100K = 100,000 reads/sec (average)
Peak (3x average) = 300,000 reads/sec
```

#### 3. Storage Requirements
```
Text-only tweets (90%):
- 100M tweets/day × 90% = 90M tweets
- 90M × 280 bytes = 25,200 MB ≈ 25 GB/day

Tweets with images (10%):
- 100M tweets/day × 10% = 10M tweets
- Text: 10M × 280 bytes = 2.8 GB
- Images: 10M × 200 KB = 2,000,000 MB = 2 TB

Total per day = 25 GB + 2 TB ≈ 2 TB
Total per year = 2 TB × 365 = 730 TB
```

#### 4. Bandwidth Requirements
```
Write bandwidth (uploads):
- Text: 1,000 tweets/sec × 280 bytes = 280 KB/sec
- Images: 100 tweets/sec × 200 KB = 20 MB/sec
- Total writes: ≈ 20 MB/sec

Read bandwidth (downloads):
- 100,000 reads/sec × 280 bytes = 28 MB/sec (text)
- 10,000 reads/sec × 200 KB = 2,000 MB/sec = 2 GB/sec (images)
- Total reads: ≈ 2 GB/sec

Peak bandwidth: 2 GB/sec × 3 = 6 GB/sec = 48 Gigabits/sec
```

### Design Implications

**Must have:**
- ✅ CDN for images (can't serve 2 GB/sec from one location)
- ✅ Aggressive caching (read-heavy workload)
- ✅ Horizontal scaling (300K reads/sec from multiple servers)
- ✅ Database read replicas (distribute read load)

**Nice to have:**
- Image compression (200 KB → 50 KB = 4x savings)
- Timeline pre-computation (fan-out on write for faster reads)
- Multiple data centers (global reach)

---

## Example 2: Netflix-Scale Video Streaming

### Given Requirements
- **Users**: 230 million subscribers
- **Daily active users**: 30% watch something = 70 million
- **Average viewing time**: 2 hours per day
- **Video bitrates**:
  - SD (480p): 1 Mbps
  - HD (720p): 3 Mbps
  - Full HD (1080p): 5 Mbps
  - 4K: 25 Mbps
- **Quality mix**: 30% SD, 40% HD, 20% Full HD, 10% 4K
- **Content library**: 10,000 titles
- **Average movie length**: 90 minutes

### Calculations

#### 1. Concurrent Viewers (Peak Time)
```
DAU watching: 70M users
Peak time (evening): Assume 30% watch simultaneously
Concurrent viewers = 70M × 30% = 21M concurrent streams
```

#### 2. Bandwidth Requirements
```
SD streams: 21M × 30% × 1 Mbps = 6.3M Mbps = 6.3 Terabits/sec (Tbps)
HD streams: 21M × 40% × 3 Mbps = 25.2 Tbps
Full HD: 21M × 20% × 5 Mbps = 21 Tbps
4K: 21M × 10% × 25 Mbps = 52.5 Tbps

Total peak bandwidth = 6.3 + 25.2 + 21 + 52.5 = 105 Tbps
```

#### 3. Storage Requirements
```
Content per title (all quality versions):
- SD: 90 min × 1 Mbps = 5,400 Mbit = 675 MB
- HD: 90 min × 3 Mbps = 2 GB
- Full HD: 90 min × 5 Mbps = 3.4 GB
- 4K: 90 min × 25 Mbps = 17 GB
- Total per title: 23 GB (all qualities)

Total library: 10,000 titles × 23 GB = 230,000 GB = 230 TB

With multiple regions (3 copies): 230 TB × 3 = 690 TB
```

#### 4. Daily Data Transfer
```
70M users × 2 hours × average bitrate (5 Mbps) 
= 70M × 2 × 5 Mbps × 3,600 sec
= 2,520,000,000,000 Mb
= 315,000 TB
= 315 Petabytes per day
```

### Design Implications

**Absolutely required:**
- ✅ Global CDN (Open Connect boxes in ISPs)
- ✅ Adaptive bitrate streaming (adjust based on user's bandwidth)
- ✅ Pre-encoding in multiple qualities
- ✅ Edge caching (popular content near users)

**Key optimizations:**
- Cache popular 20% of content at edges (serves 80% of traffic)
- Compress videos efficiently (H.264, H.265, AV1 codecs)
- Pre-position popular content before peak hours
- Multiple data centers for disaster recovery

---

## Example 3: Uber Ride-Hailing Service

### Given Requirements
- **Daily active riders**: 100 million
- **Rides per day**: 20 million
- **Average ride duration**: 20 minutes
- **Location updates**: Every 4 seconds while riding
- **Active drivers**: 5 million
- **Driver location updates**: Every 4 seconds while online

### Calculations

#### 1. Location Update Rate
```
During rides:
- 20M rides at any moment (some completing, some starting)
- Average active rides = 20M rides / (24 hours / 20 min) = ~280K concurrent rides
- Updates: 280K rides × 2 devices (driver + rider) × 1 update/4sec = 140K updates/sec

Idle drivers searching:
- 5M drivers - 280K active = 4.7M idle drivers
- Updates: 4.7M × 1 update/4sec = 1.2M updates/sec

Total: 140K + 1,200K = 1,340K ≈ 1.4M location updates/sec
```

#### 2. Storage per Update
```
Each location update:
- User/Driver ID: 8 bytes
- Latitude: 8 bytes
- Longitude: 8 bytes
- Timestamp: 8 bytes
- Accuracy: 4 bytes
- Total: ~36 bytes

Storage per day:
- 1.4M updates/sec × 36 bytes = 50 MB/sec
- Per day: 50 MB/sec × 86,400 = 4,320,000 MB = 4.3 TB/day
```

#### 3. Trip Database Writes
```
Each trip creates:
- Trip record: 1 KB
- Payment record: 500 bytes
- Rating: 200 bytes
- Total per trip: ~2 KB

Daily trip storage: 20M trips × 2 KB = 40,000 MB = 40 GB/day
```

#### 4. Request Matching
```
Ride requests per second:
- 20M rides / 86,400 sec = 231 requests/sec (average)
- Peak hours (3x): 693 requests/sec

For each request:
- Query nearby drivers (within 5km radius)
- Typical urban density: 100 drivers within range
- Need to evaluate 100 drivers per request
- Total evaluations: 693 × 100 = 69,300 per second (peak)
```

### Design Implications

**Critical components:**
- ✅ Geospatial database (PostGIS or MongoDB with geo-indexing)
- ✅ Real-time location tracking (WebSockets)
- ✅ Efficient matching algorithm (sub-second response)
- ✅ Time-series database for location history

**Optimizations:**
- Reduce location update frequency (4 sec → 10 sec saves 60% writes)
- Grid-based driver lookup (faster than radius search)
- Cache nearby drivers in memory
- Batch database writes for location updates

---

## Example 4: WhatsApp Messaging

### Given Requirements
- **Users**: 2 billion
- **Daily active users**: 60% = 1.2 billion
- **Messages per user per day**: 50
- **Message size**: 100 bytes (text average)
- **Media messages**: 10% include media (1 MB average)
- **Group messages**: 30% are in groups (average 10 people)

### Calculations

#### 1. Message Volume
```
Daily messages:
- Individual: 1.2B users × 50 msgs × 70% = 42B messages
- Group: 1.2B users × 50 msgs × 30% = 18B messages (sent)
- Group delivered: 18B × 10 people = 180B (received)
- Total delivered: 42B + 180B = 222B messages/day

Messages per second:
- 222B / 86,400 ≈ 2.6M messages/sec (average)
- Peak (2x): 5.2M messages/sec
```

#### 2. Storage Requirements
```
Text messages (90%):
- 222B × 90% × 100 bytes = 19,980 GB ≈ 20 TB/day

Media messages (10%):
- 222B × 10% × 1 MB = 22,200,000 GB = 22 PB/day

Total: 22 PB/day
If stored for 30 days: 22 PB × 30 = 660 PB
```

#### 3. Connection Management
```
Concurrent connections:
- Assume 30% of DAU online simultaneously
- 1.2B × 30% = 360M concurrent WebSocket connections

Connection memory:
- 360M connections × 10 KB per connection = 3.6 TB of RAM
- Distributed across servers: 3.6 TB / 10 KB per conn = 360K connections per server
- If each server handles 100K connections: Need 3,600 servers
```

#### 4. Bandwidth
```
Incoming (user sends):
- 2.6M messages/sec × 100 bytes = 260 MB/sec (text)
- 260K messages/sec × 1 MB = 260 GB/sec (media)
- Total: ≈ 260 GB/sec

Outgoing (server delivers):
- With group multiplier: 5.2M messages/sec
- Text: 520 MB/sec
- Media: 520 GB/sec
- Total: ≈ 520 GB/sec
```

### Design Implications

**Required architecture:**
- ✅ WebSocket servers (maintain persistent connections)
- ✅ Message queue system (handle delivery bursts)
- ✅ Distributed storage (media files across many servers)
- ✅ End-to-end encryption (privacy requirement)

**Key decisions:**
- Don't store messages on server long-term (privacy + cost)
- Compress media heavily (1 MB → 200 KB)
- Peer-to-peer for calls (don't route through servers)
- Eventual delivery OK (doesn't need to be instant)

---

## Example 5: URL Shortener (TinyURL)

### Given Requirements
- **New URLs per month**: 100 million
- **Read/Write ratio**: 100:1
- **URL lifespan**: 5 years
- **Short URL length**: 7 characters (base62: a-z, A-Z, 0-9)

### Calculations

#### 1. Total URLs Stored
```
Monthly: 100M URLs
Yearly: 100M × 12 = 1.2B URLs
5 years: 1.2B × 5 = 6B URLs

Short URL space:
- 62^7 = 3.5 trillion possible combinations
- 6B / 3.5T = 0.17% utilization ✅ (plenty of space)
```

#### 2. Storage Requirements
```
Per URL entry:
- Original URL: 200 bytes (average)
- Short code: 7 bytes
- Metadata (created, expiry, clicks): 100 bytes
- Total: ~300 bytes per entry

Total storage:
- 6B URLs × 300 bytes = 1,800 GB = 1.8 TB ✅ (easily manageable)
```

#### 3. Request Rates
```
Writes (shortening):
- 100M/month = ~40 writes/sec (average)
- Peak (2x): 80 writes/sec

Reads (redirects):
- 100M × 100 = 10B/month
- 10B / 2.6M sec = 3,850 reads/sec (average)
- Peak (2x): 7,700 reads/sec
```

#### 4. Cache Sizing
```
Hot URLs (20% get 80% of traffic):
- 6B × 20% = 1.2B URLs
- 1.2B × 300 bytes = 360 GB

Realistically, cache top 1% (gets 50% of traffic):
- 60M URLs × 300 bytes = 18 GB ✅ (fits in memory!)
```

### Design Implications

**System is simple:**
- ✅ Single database sufficient (1.8 TB easily handled by modern DB)
- ✅ Cache top URLs in Redis (18 GB)
- ✅ Read-heavy → Multiple read replicas
- ✅ Hash or generate unique IDs for short codes

**Interesting optimization:**
- Pre-generate short codes in batches (faster than on-demand)
- Use bloom filter to check if URL already shortened
- Separate hot storage (frequently accessed) from cold storage

---

## Summary: Common Patterns

### Storage Scaling
```
Small:    < 100 GB     → Single server OK
Medium:   100 GB - 10 TB → Need planning, but manageable
Large:    10 TB - 1 PB  → Distributed storage required
Massive:  > 1 PB        → Specialized infrastructure (Netflix, YouTube)
```

### Request Rates
```
Low:     < 1K req/sec   → Single server
Medium:  1K - 100K      → Load balancer + multiple servers
High:    100K - 1M      → Distributed system, caching critical
Extreme: > 1M           → Custom infrastructure, edge computing
```

### Key Takeaways

1. **Media dominates storage** (images, videos dwarf text)
2. **Read/write ratios matter** (optimize for the common case)
3. **Peak traffic ≠ average** (design for 2-5x average)
4. **Caching is critical** (80/20 rule applies everywhere)
5. **Location matters** (CDNs essential for global services)
6. **Connections are expensive** (millions of WebSockets need planning)

## Next Steps

Practice with the [Design Exercises](./design-exercise.md) to apply these calculation techniques to new scenarios!
