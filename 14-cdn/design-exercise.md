# Design Exercise: Global Media Streaming CDN

## 1. Design Problem

### Problem Statement
Design a Content Delivery Network (CDN) for a global video streaming platform like Netflix or YouTube. The CDN must deliver video content to users worldwide with low latency, high availability, and cost efficiency. It must handle massive scale, support adaptive bitrate streaming, and minimize origin server load.

### Context and Constraints
- Videos range from minutes to hours in length
- Multiple quality levels (SD, HD, 4K, 8K)
- Global user base across all continents
- Peak traffic during evening hours in each timezone
- Need to minimize bandwidth costs
- Origin servers have limited capacity
- CDN edge locations worldwide
- Must support both live streaming and on-demand content
- Mobile and desktop clients with varying network conditions

### Requirements

#### Functional Requirements
- Deliver video content globally with low latency
- Support adaptive bitrate streaming (ABR)
- Cache popular content at edge locations
- Stream live events to millions simultaneously
- Support video thumbnails and preview clips
- Handle both streaming and downloadable content
- Provide analytics (views, engagement, quality metrics)
- DRM (Digital Rights Management) support

#### Non-Functional Requirements
- **Scale**: 200 million concurrent users, 1 PB of content
- **Performance**: Time to first byte < 100ms, rebuffering < 1%
- **Availability**: 99.99% uptime
- **Bandwidth**: 10 Tbps peak throughput
- **Latency**: < 100ms for 95% of users
- **Cost**: Optimize bandwidth and storage costs
- **Cache Hit Rate**: > 90% for popular content

## 2. Guided Questions

### CDN Basics
1. **What is the main purpose of a CDN?**
   - Hint: Why not serve all content from origin servers?
   - Consider: Latency, bandwidth, load

2. **Where should CDN edge servers be located?**
   - Hint: Close to users or close to origin?
   - Consider: Major cities, ISPs, internet exchange points

### Content Delivery
3. **How do you route users to the nearest CDN edge?**
   - Hint: Multiple ways to determine "nearest"
   - Consider: DNS, anycast, geolocation

4. **What content should be cached vs. fetched from origin?**
   - Hint: Not all videos are equally popular
   - Consider: Popular vs. niche, new vs. old, cost vs. hit rate

### Video Streaming
5. **How does adaptive bitrate streaming work?**
   - Hint: Different qualities for different network conditions
   - Consider: HLS, DASH, chunking

6. **How do you handle a viral video that suddenly gets millions of views?**
   - Hint: Cache isn't warmed yet
   - Consider: Cache warming, flash crowds, origin protection

## 3. Step-by-Step Guidance

### Step 1: CDN Architecture Overview
Multi-tier CDN architecture:

```
┌──────────────┐
│    Users     │
│  (Globally)  │
└──────┬───────┘
       │
       ↓
┌──────────────────────┐
│     DNS / GSLB       │ ← Route to nearest edge
└──────────┬───────────┘
           │
    ┌──────┴──────┬──────────┐
    ↓             ↓          ↓
┌────────┐  ┌────────┐  ┌────────┐
│ Edge   │  │ Edge   │  │ Edge   │
│ (US)   │  │ (EU)   │  │ (Asia) │
└───┬────┘  └───┬────┘  └───┬────┘
    │           │            │
    └───────────┼────────────┘
                ↓
        ┌───────────────┐
        │  Regional     │ ← Mid-tier cache
        │  Cache        │
        └───────┬───────┘
                ↓
        ┌───────────────┐
        │  Origin       │ ← Source of truth
        │  Servers      │
        └───────────────┘
```

**Tiers:**
1. **Edge**: Closest to users (100+ locations)
2. **Regional**: Mid-tier cache (10-20 locations)
3. **Origin**: Source servers (2-3 locations)

### Step 2: Content Routing (GeoDNS)
Route users to nearest edge:

```
User in San Francisco requests video.netflix.com
    ↓
DNS query
    ↓
GeoDNS sees request from US West Coast
    ↓
Returns IP of San Jose edge server
    ↓
User connects to nearby edge (10ms latency vs. 150ms to origin)
```

**DNS Response:**
```
Query: video.netflix.com
Response (for US user): 203.0.113.10 (US West edge)
Response (for EU user): 198.51.100.5 (EU West edge)
```

### Step 3: Adaptive Bitrate Streaming (ABR)
Multiple quality levels:

**Video Encoding:**
```
Original Video (4K, 60fps)
    ↓ Transcoding
    ├─ 4K (3840x2160, 25 Mbps)
    ├─ 1080p (1920x1080, 8 Mbps)
    ├─ 720p (1280x720, 5 Mbps)
    ├─ 480p (854x480, 2.5 Mbps)
    └─ 360p (640x360, 1 Mbps)
```

**HLS/DASH Chunking:**
```
Video split into 6-second chunks
- chunk_0001_4k.m4s
- chunk_0001_1080p.m4s
- chunk_0001_720p.m4s
- chunk_0001_480p.m4s
- chunk_0001_360p.m4s

Manifest file (playlist.m3u8):
#EXTM3U
#EXT-X-STREAM-INF:BANDWIDTH=25000000,RESOLUTION=3840x2160
4k/playlist.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=8000000,RESOLUTION=1920x1080
1080p/playlist.m3u8
...
```

**Client Behavior:**
```javascript
// Client measures bandwidth
currentBandwidth = measure_bandwidth();

if (currentBandwidth > 8_000_000) {
    quality = '1080p';
} else if (currentBandwidth > 5_000_000) {
    quality = '720p';
} else {
    quality = '480p';
}

// Request appropriate chunk
fetch(`https://cdn.example.com/video/chunk_0001_${quality}.m4s`);
```

### Step 4: Cache Strategy
Intelligent caching decisions:

**Cache Tiers:**
```
Edge Cache (SSD, 10TB per location):
- Top 1% most popular content
- Recently requested in that region
- Live streams

Regional Cache (SSD, 100TB per location):
- Top 10% popular content
- Serves edges

Origin Storage (S3, unlimited):
- All content
- Serves regional caches
```

**Cache Decision Algorithm:**
```python
class CacheStrategy:
    def should_cache(self, video_id, location):
        # Check popularity
        views = analytics.get_views(video_id, last_24h=True)
        if views > 10000:
            return True
        
        # Check regional popularity
        regional_views = analytics.get_views(video_id, location=location)
        if regional_views > 100:
            return True
        
        # Check trending
        if video_id in trending.get_trending_videos():
            return True
        
        return False
```

### Step 5: Cache Warming
Proactively cache popular content:

```python
class CacheWarmer:
    def warm_cache_for_new_release(self, video_id):
        # Get user locations and preferences
        target_locations = analytics.predict_popular_regions(video_id)
        
        for location in target_locations:
            # Pre-load to edge servers
            edge_servers = get_edge_servers(location)
            for server in edge_servers:
                server.prefetch(video_id, qualities=['1080p', '720p'])
    
    def warm_cache_for_trending(self):
        trending = analytics.get_trending_videos()
        for video_id in trending:
            # Gradually warm cache across all edges
            self.distribute_to_edges(video_id)
```

### Step 6: Origin Protection
Prevent origin overload:

**Cache Shield:**
```
Many Edge Servers → Few Regional Caches → Origin

Without Shield:
1000 edges, each cache misses → 1000 requests to origin

With Shield:
1000 edges → 10 regional caches → 10 requests to origin
```

**Implementation:**
```
# Edge server
def get_video(video_id):
    # Try local cache
    if local_cache.has(video_id):
        return local_cache.get(video_id)
    
    # Try regional cache (shield)
    regional = get_regional_cache(my_region)
    video = regional.get(video_id)
    
    # Cache locally for next request
    local_cache.set(video_id, video)
    return video

# Regional cache
def get_video(video_id):
    # Try regional cache
    if cache.has(video_id):
        return cache.get(video_id)
    
    # Fetch from origin (only if not cached)
    video = origin.get(video_id)
    cache.set(video_id, video)
    return video
```

### Step 7: Live Streaming
Special handling for live content:

```
┌─────────────┐
│  Encoder    │ ← Live source
│  (Stadium)  │
└──────┬──────┘
       │ RTMP/WebRTC
       ↓
┌──────────────────┐
│  Origin          │ ← Ingest point
│  (Transcode)     │
└──────┬───────────┘
       │ HLS/DASH chunks
       ↓
┌──────────────────┐
│  CDN Edges       │ ← Global distribution
│  (Cached 30s)    │
└──────────────────┘
       ↓
┌──────────────────┐
│  Viewers         │
│  (Millions)      │
└──────────────────┘
```

**Live Streaming Characteristics:**
- Very short cache TTL (6-30 seconds)
- All users watch same content simultaneously
- Ultra-low latency requirement for sports
- Need to handle sudden viewer spikes

## 4. Sample Solution

### Complete CDN Architecture

```
┌──────────────────────────────────────────────┐
│              Users (Global)                  │
└───────────────────┬──────────────────────────┘
                    ↓
┌───────────────────▼──────────────────────────┐
│          DNS Load Balancing (GeoDNS)         │
│  - Route to nearest edge                     │
│  - Health checks                             │
│  - Failover                                  │
└───────────────────┬──────────────────────────┘
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
┌────────────┐ ┌────────────┐ ┌────────────┐
│  Edge      │ │  Edge      │ │  Edge      │
│  US West   │ │  EU West   │ │  Asia      │
│            │ │            │ │  Pacific   │
│ - Cache    │ │ - Cache    │ │ - Cache    │
│ - 10TB SSD │ │ - 10TB SSD │ │ - 10TB SSD │
└─────┬──────┘ └─────┬──────┘ └─────┬──────┘
      │              │              │
      └──────────────┼──────────────┘
                     ↓
        ┌────────────────────────┐
        │  Regional Cache        │
        │  (Shield/Mid-Tier)     │
        │  - 100TB cache         │
        │  - Reduce origin load  │
        └────────────┬───────────┘
                     ↓
        ┌────────────────────────┐
        │  Origin Cluster        │
        │  - Video storage (S3)  │
        │  - Transc coding       │
        │  - Metadata DB         │
        └────────────────────────┘
```

### Video Delivery Flow

**On-Demand Video:**
```
1. User requests video
   GET https://video.example.com/watch?v=abc123

2. DNS routes to nearest edge
   User → Edge Server (San Jose)

3. Edge checks cache
   IF cached:
     Serve from edge (< 10ms)
   ELSE:
     Fetch from regional cache
     Cache locally
     Serve to user

4. Client requests video chunks
   GET /video/abc123/720p/chunk_0001.m4s
   GET /video/abc123/720p/chunk_0002.m4s
   (Adaptive based on network speed)
```

### Key Design Decisions

#### 1. Three-Tier Architecture
**Decision**: Edge → Regional → Origin

**Why**:
- Edge: Low latency for users
- Regional: Reduces origin load
- Origin: Source of truth

**Alternative**: Two-tier (Edge → Origin)
- Simpler but higher origin load

#### 2. Chunk-Based Streaming
**Decision**: Split videos into 6-second chunks

**Why**:
- Adaptive bitrate switching
- Faster seek times
- Better caching (cache per chunk)
- Resume playback easily

#### 3. Cache Everything Policy
**Decision**: Cache all content, with LRU eviction

**Why**:
- Simplifies logic
- High cache hit rate
- Let usage drive what stays cached

**Alternative**: Selective caching
- More complex
- Risk missing popular content

## 5. Extension Challenges

### Challenge 1: Reduce Bandwidth Costs
**Requirement**: Cut bandwidth costs by 40%

**How would you optimize?**
<details>
<summary>Solution</summary>

1. **Better Compression (AV1 codec)**:
   - 30% smaller than H.264
   - Same quality, less bandwidth
   - Savings: 30% bandwidth reduction

2. **Aggressive Caching**:
   - Increase cache size
   - Longer TTLs for older content
   - Pre-warm cache for anticipated demand
   - Savings: 10% fewer origin fetches

3. **Peer-to-Peer (WebRTC)**:
   - Users share chunks with nearby users
   - Reduce CDN load
   - Savings: 20% CDN bandwidth

4. **Smart Quality Selection**:
   - Don't serve 4K to mobile phones
   - Detect screen size and connection
   - Savings: 15% bandwidth

</details>

### Challenge 2: Ultra-Low Latency Live Streaming
**Requirement**: < 1 second latency for live sports

**How would you achieve this?**
<details>
<summary>Solution</summary>

Use **WebRTC** or **Low-Latency HLS**:

```
Traditional HLS: 6-30 second latency
Low-Latency HLS: 2-3 second latency
WebRTC: < 1 second latency

Implementation:
- Smaller chunks (200ms instead of 6s)
- HTTP/2 push
- Reduced buffering
- Edge transcoding
```

Trade-offs:
- Higher CPU cost (more chunks)
- More complex client logic
- Requires newer client support

</details>

---

## Summary

This design exercise demonstrates how to build a global CDN by:
- Using multi-tier architecture (edge, regional, origin)
- Implementing intelligent caching strategies
- Supporting adaptive bitrate streaming
- Routing users to nearest edge servers
- Protecting origin servers from overload
- Optimizing costs through compression and caching

**Key Takeaways**:
1. **Location matters**: Edge servers close to users
2. **Cache strategically**: Popular content at edge
3. **Protect origin**: Use regional shield
4. **Adaptive streaming**: Match quality to network
5. **Monitor everything**: Cache hit rate, latency, errors
6. **Cost vs. performance**: Balance caching with costs
