# Case Study: Video Streaming Service (Intermediate)

## Problem Statement

Design a video streaming platform like YouTube or Netflix that allows users to upload videos, watch videos with smooth playback, discover content, and stream video efficiently across different network conditions and devices globally.

## Requirements Analysis

### Functional Requirements

**Core Features:**
1. **Video Upload**: Users can upload videos (various formats and resolutions)
2. **Video Playback**: Stream videos with minimal buffering
3. **Adaptive Bitrate**: Adjust quality based on network conditions
4. **Search & Discovery**: Search for videos by title, tags, categories
5. **Recommendations**: Suggest videos based on viewing history
6. **User Interactions**: Like, comment, subscribe, share
7. **Content Management**: Creators can manage their videos
8. **Live Streaming**: Support for live video broadcasts

**Video Characteristics:**
- Support multiple formats (MP4, AVI, MOV, etc.)
- Support multiple resolutions (360p, 480p, 720p, 1080p, 4K)
- Average video length: 10 minutes
- Maximum video length: 8 hours

### Non-Functional Requirements

**Scale Requirements:**
- **Users**: 2 billion registered users
  - 500 million daily active users (DAU)
  - 1 million concurrent viewers at peak
- **Videos**: 500 million total videos
  - 500,000 new videos uploaded daily
  - 5 billion video views per day
- **Storage**: 1 billion hours of video content

**Performance Requirements:**
- **Video Start Time**: < 2 seconds to begin playback
- **Buffering**: < 1% rebuffer rate
- **Upload Success Rate**: > 99%
- **Availability**: 99.99% uptime (52 minutes downtime per year)
- **Global Latency**: < 100ms to nearest CDN edge

**Bandwidth & Storage:**
- Average video size: 500 MB (1080p, 10 minutes)
- Daily upload: 500K videos × 500 MB = 250 TB/day
- Daily streaming: 5B views × 10 min × 5 Mbps = 37.5 PB/day

## Capacity Estimation

### Storage Calculation

```
**Original Videos:**
500K uploads/day × 500 MB average = 250 TB/day
Annual: 250 TB × 365 = 91.25 PB/year
5-year total: 456 PB

**Transcoded Videos (multiple qualities):**
For each video, generate 5 qualities (360p, 480p, 720p, 1080p, 4K)
Storage multiplier: ~3x (including original)
Total: 456 PB × 3 = 1.37 EB (5 years)

**With Replication (3x for reliability):**
1.37 EB × 3 = 4.1 EB

**Metadata Storage:**
500M videos × 2 KB metadata = 1 TB
Negligible compared to video storage
```

### Traffic Estimation

```
**Upload Traffic:**
500K videos/day = ~6 uploads/second average
Peak: ~20 uploads/second
Bandwidth: 20 uploads/sec × 500 MB = 10 GB/sec

**Streaming Traffic:**
5B views/day = ~58,000 views/second average
Peak: ~150,000 views/second
Average bitrate: 5 Mbps (1080p)
Bandwidth: 150K views × 5 Mbps = 750 Gbps = 93.75 GB/sec

**API Traffic:**
Search queries: 100M/day = 1,200/sec
Metadata requests: 200M/day = 2,300/sec
```

### CDN Requirements

```
**Edge Locations:**
Minimum 200 PoPs globally for <100ms latency
Each PoP caches hot content (top 20% of videos)

**Cache Storage per PoP:**
500M videos × 20% = 100M hot videos
100M × 500 MB average = 50 PB
Distributed across 200 PoPs = 250 TB per PoP

**Cache Hit Rate:**
Target: 95% from CDN (only 5% origin fetch)
95% of 93.75 GB/sec = 89 GB/sec served from CDN
5% of 93.75 GB/sec = 4.7 GB/sec from origin
```

## High-Level Architecture

```
┌────────────────────────────────────────────────────────────────────────┐
│                          CLIENT LAYER                                   │
│  [Web Players] [Mobile Apps] [Smart TVs] [Game Consoles]              │
└────────────────────────────────┬───────────────────────────────────────┘
                                 │
┌────────────────────────────────▼───────────────────────────────────────┐
│                    CDN (Content Delivery Network)                       │
│         [Cloudflare/Akamai/CloudFront] - 200+ Edge Locations          │
│              - Video caching (95% hit rate)                            │
│              - Adaptive bitrate selection                              │
│              - Geographic routing                                       │
└────────────────────────────────┬───────────────────────────────────────┘
                                 │
                    ┌────────────┴────────────┐
                    ▼                         ▼
┌──────────────────────────┐       ┌──────────────────────────┐
│   API GATEWAY            │       │   VIDEO ORIGIN           │
│   - Authentication       │       │   (Object Storage - S3)  │
│   - Rate limiting        │       │   - Master copies        │
│   - Request routing      │       │   - All qualities        │
└──────────┬───────────────┘       └──────────────────────────┘
           │
┌──────────▼────────────────────────────────────────────────────────────┐
│                        MICROSERVICES LAYER                             │
├──────────────────┬─────────────────┬─────────────────┬────────────────┤
│ VIDEO SERVICE    │ UPLOAD SERVICE  │ SEARCH SERVICE  │ USER SERVICE   │
│ - Metadata       │ - Upload mgmt   │ - Elasticsearch │ - Auth         │
│ - Playback info  │ - Validation    │ - Recommendations│ - Profiles    │
│ - Analytics      │ - S3 upload     │ - Trending      │ - Subscriptions│
└──────────┬───────┴────────┬────────┴────────┬────────┴────────┬───────┘
           │                │                 │                 │
           └────────────────┴─────────────────┴─────────────────┘
                                    │
┌───────────────────────────────────▼────────────────────────────────────┐
│                        MESSAGE QUEUE (Kafka)                            │
│   [Upload Complete] [Transcode Job] [View Event] [Analytics]          │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
┌──────────────────────────┐           ┌──────────────────────────┐
│ TRANSCODING CLUSTER      │           │ ANALYTICS PIPELINE       │
│ - FFmpeg workers (1000+) │           │ - View counting          │
│ - Multiple qualities     │           │ - Watch time tracking    │
│ - Parallel processing    │           │ - Recommendation ML      │
└──────────────────────────┘           └──────────────────────────┘

┌───────────────────────────────────────────────────────────────────────┐
│                          DATA LAYER                                    │
├─────────────────┬─────────────────┬─────────────────┬─────────────────┤
│ VIDEO METADATA  │ USER DATA       │ ANALYTICS DB    │ CACHE           │
│ (Cassandra)     │ (PostgreSQL)    │ (Hadoop/BigQuery)│ (Redis)        │
│ - Video info    │ - User profiles │ - View logs     │ - Hot metadata  │
│ - Comments      │ - Subscriptions │ - Aggregations  │ - Thumbnails    │
└─────────────────┴─────────────────┴─────────────────┴─────────────────┘
```

## Video Upload & Transcoding Pipeline

### Upload Flow

```
┌─────────────┐
│ User Uploads│ 
│   Video     │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────────────┐
│ 1. UPLOAD SERVICE                       │
│    - Generate upload ID                 │
│    - Create S3 pre-signed URL           │
│    - Return upload URL to client        │
└──────┬──────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│ 2. CLIENT UPLOADS DIRECTLY TO S3       │
│    - Chunked upload (multipart)         │
│    - Progress tracking                  │
│    - Resume on failure                  │
└──────┬──────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│ 3. S3 TRIGGERS EVENT                    │
│    - ObjectCreated event                │
│    - Publish to Kafka                   │
└──────┬──────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│ 4. TRANSCODING SERVICE                  │
│    - Pull video from S3                 │
│    - Extract metadata (duration, etc.)  │
│    - Generate thumbnail                 │
│    - Start transcoding jobs             │
└──────┬──────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│ 5. PARALLEL TRANSCODING                 │
│    Job 1: 360p encode                   │
│    Job 2: 480p encode                   │
│    Job 3: 720p encode                   │
│    Job 4: 1080p encode                  │
│    Job 5: 4K encode (if source quality) │
└──────┬──────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│ 6. UPLOAD ENCODED VIDEOS TO S3          │
│    - Store in quality-specific folders  │
│    - Generate HLS/DASH manifests        │
│    - Update CDN cache                   │
└──────┬──────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│ 7. UPDATE METADATA                      │
│    - Mark video as "ready"              │
│    - Store video URLs                   │
│    - Notify user                        │
└─────────────────────────────────────────┘
```

### Transcoding at Scale

```python
class TranscodingService:
    QUALITIES = {
        '360p': {'resolution': '640x360', 'bitrate': '800k'},
        '480p': {'resolution': '854x480', 'bitrate': '1400k'},
        '720p': {'resolution': '1280x720', 'bitrate': '2800k'},
        '1080p': {'resolution': '1920x1080', 'bitrate': '5000k'},
        '4K': {'resolution': '3840x2160', 'bitrate': '20000k'}
    }
    
    async def transcode_video(self, video_id: str, source_url: str):
        """Transcode video into multiple qualities."""
        # Download original
        source_file = await self.download_from_s3(source_url)
        
        # Analyze source video
        metadata = await self.extract_metadata(source_file)
        source_resolution = metadata['resolution']
        
        # Create transcoding jobs for applicable qualities
        jobs = []
        for quality, params in self.QUALITIES.items():
            # Don't upscale - only transcode to equal or lower quality
            if self.should_transcode(source_resolution, params['resolution']):
                job = {
                    'video_id': video_id,
                    'quality': quality,
                    'input': source_file,
                    'output': f"s3://videos/{video_id}/{quality}/video.mp4",
                    'params': params
                }
                jobs.append(job)
        
        # Submit jobs to distributed queue
        for job in jobs:
            await kafka.produce('transcode_jobs', job)
    
    async def process_transcode_job(self, job: dict):
        """Worker function to process a single transcode job."""
        ffmpeg_cmd = f"""
        ffmpeg -i {job['input']} \
               -c:v libx264 \
               -preset medium \
               -b:v {job['params']['bitrate']} \
               -vf scale={job['params']['resolution']} \
               -c:a aac -b:a 128k \
               -movflags +faststart \
               {job['output']}
        """
        
        # Execute FFmpeg
        await self.run_ffmpeg(ffmpeg_cmd)
        
        # Generate HLS segments for adaptive streaming
        await self.generate_hls_segments(job['output'], job['quality'])
        
        # Update database
        await self.mark_quality_ready(job['video_id'], job['quality'])
```

## Adaptive Bitrate Streaming (ABR)

### HLS/DASH Implementation

```
Video stored in multiple quality segments:

/video_123/
  ├── 360p/
  │   ├── segment_0.ts (10 seconds)
  │   ├── segment_1.ts
  │   └── ...
  ├── 480p/
  │   ├── segment_0.ts
  │   └── ...
  ├── 720p/
  │   ├── segment_0.ts
  │   └── ...
  ├── 1080p/
  │   ├── segment_0.ts
  │   └── ...
  └── master.m3u8 (manifest)

Master Playlist (master.m3u8):
#EXTM3U
#EXT-X-STREAM-INF:BANDWIDTH=800000,RESOLUTION=640x360
360p/playlist.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=1400000,RESOLUTION=854x480
480p/playlist.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=2800000,RESOLUTION=1280x720
720p/playlist.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=5000000,RESOLUTION=1920x1080
1080p/playlist.m3u8
```

### Client-Side ABR Logic

```javascript
class AdaptiveBitratePlayer {
    constructor(videoElement, manifestUrl) {
        this.video = videoElement;
        this.manifestUrl = manifestUrl;
        this.qualities = [];
        this.currentQuality = null;
        this.bandwidthEstimate = 5000000; // Start with 5 Mbps
    }
    
    async initialize() {
        // Fetch master playlist
        const manifest = await this.fetchManifest(this.manifestUrl);
        this.qualities = this.parseQualities(manifest);
        
        // Select initial quality based on estimated bandwidth
        this.currentQuality = this.selectQuality(this.bandwidthEstimate);
        
        // Start playback
        await this.startPlayback();
        
        // Monitor network and adjust quality
        this.startQualityMonitoring();
    }
    
    selectQuality(bandwidth) {
        // Select highest quality that fits within bandwidth
        // Use 80% of bandwidth for safety margin
        const safeBandwidth = bandwidth * 0.8;
        
        let selected = this.qualities[0]; // Start with lowest
        for (const quality of this.qualities) {
            if (quality.bandwidth <= safeBandwidth) {
                selected = quality;
            } else {
                break;
            }
        }
        return selected;
    }
    
    startQualityMonitoring() {
        setInterval(() => {
            const stats = this.getPlaybackStats();
            
            // Update bandwidth estimate
            this.updateBandwidthEstimate(stats);
            
            // Check if quality switch is needed
            if (this.shouldSwitchQuality(stats)) {
                const newQuality = this.selectQuality(this.bandwidthEstimate);
                if (newQuality !== this.currentQuality) {
                    this.switchQuality(newQuality);
                }
            }
        }, 5000); // Check every 5 seconds
    }
    
    shouldSwitchQuality(stats) {
        // Switch up if: buffer is healthy AND bandwidth allows
        if (stats.bufferLevel > 30 && 
            this.bandwidthEstimate > this.currentQuality.bandwidth * 1.5) {
            return true;
        }
        
        // Switch down if: buffer is low OR bandwidth insufficient
        if (stats.bufferLevel < 10 || 
            this.bandwidthEstimate < this.currentQuality.bandwidth * 0.8) {
            return true;
        }
        
        return false;
    }
    
    updateBandwidthEstimate(stats) {
        // Exponential moving average
        const measured = stats.downloadSpeed;
        this.bandwidthEstimate = 
            0.8 * this.bandwidthEstimate + 0.2 * measured;
    }
}
```

## CDN Strategy

### Multi-Tier CDN Architecture

```
┌────────────────────────────────────────────────────────────┐
│                    TIER 1: EDGE LOCATIONS                   │
│         200+ PoPs worldwide (closest to users)             │
│  - Cache hot content (top 20% of videos)                   │
│  - Serve 90% of requests                                   │
│  - TTL: 24 hours                                           │
└────────────────────┬───────────────────────────────────────┘
                     │ (Cache miss)
                     ▼
┌────────────────────────────────────────────────────────────┐
│              TIER 2: REGIONAL CACHE                         │
│           20 regional data centers                         │
│  - Cache warm content (top 50% of videos)                  │
│  - Serve 8% of requests                                    │
│  - TTL: 7 days                                             │
└────────────────────┬───────────────────────────────────────┘
                     │ (Cache miss)
                     ▼
┌────────────────────────────────────────────────────────────┐
│              TIER 3: ORIGIN STORAGE (S3)                    │
│              All videos permanently stored                 │
│  - Serve 2% of requests (long-tail content)                │
│  - Replicated across regions                               │
└────────────────────────────────────────────────────────────┘

Cache Hit Rates:
- Edge: 90% (most requests served here)
- Regional: 8% (edge misses)
- Origin: 2% (rare/old content)
```

### CDN Cache Warming Strategy

```python
class CDNCacheWarmer:
    async def warm_popular_content(self):
        """Pre-cache trending videos to CDN edges."""
        # Get trending videos from analytics
        trending = await self.get_trending_videos(limit=10000)
        
        for video in trending:
            # Push to all edge locations
            await self.push_to_cdn(video)
    
    async def push_to_cdn(self, video_id: str):
        """Push video to CDN edge locations."""
        cdn_zones = self.get_cdn_zones()
        
        for zone in cdn_zones:
            # Trigger CDN to fetch and cache
            await self.cdn_client.cache_video(
                zone=zone,
                url=f"https://origin.example.com/videos/{video_id}/master.m3u8"
            )
    
    async def predict_and_cache(self):
        """ML-based prediction of videos that will trend."""
        # Get videos published in last 2 hours
        recent = await self.get_recent_videos(hours=2)
        
        for video in recent:
            # Predict if video will go viral
            score = await self.ml_model.predict_virality(video)
            
            if score > 0.8:  # High probability of trending
                await self.push_to_cdn(video.id)
```

## API Design

### 1. Upload Video

```http
POST /api/v1/videos/upload/init
Authorization: Bearer <token>
Content-Type: application/json

Request:
{
  "title": "Amazing System Design Tutorial",
  "description": "Learn how to design Netflix",
  "tags": ["system-design", "tutorial"],
  "category": "education",
  "visibility": "public",
  "file_size": 524288000,  // 500 MB
  "file_name": "video.mp4"
}

Response 200 OK:
{
  "upload_id": "upload_abc123",
  "upload_url": "https://s3.amazonaws.com/uploads/abc123?signature=...",
  "chunk_size": 10485760,  // 10 MB chunks
  "expires_at": "2024-01-15T11:30:00Z"
}
```

### 2. Get Video Playback Info

```http
GET /api/v1/videos/{video_id}/playback
Authorization: Bearer <token>

Response 200 OK:
{
  "video_id": "video_123",
  "title": "Amazing System Design Tutorial",
  "manifest_url": "https://cdn.example.com/video_123/master.m3u8",
  "thumbnail_url": "https://cdn.example.com/video_123/thumb.jpg",
  "duration": 600,  // seconds
  "qualities": [
    {"resolution": "1080p", "bitrate": 5000000},
    {"resolution": "720p", "bitrate": 2800000},
    {"resolution": "480p", "bitrate": 1400000}
  ],
  "view_count": 1523456,
  "like_count": 45000,
  "created_at": "2024-01-10T10:00:00Z"
}
```

### 3. Track View Analytics

```http
POST /api/v1/videos/{video_id}/view
Authorization: Bearer <token>
Content-Type: application/json

Request:
{
  "timestamp": "2024-01-15T10:30:00Z",
  "duration_watched": 300,  // seconds
  "quality": "1080p",
  "device_type": "mobile",
  "location": "US-CA"
}

Response 202 Accepted:
{
  "status": "accepted"
}
```

## Database Schema

### Video Metadata (Cassandra)

```sql
CREATE TABLE videos (
    video_id UUID PRIMARY KEY,
    user_id UUID,
    title TEXT,
    description TEXT,
    duration INT,
    upload_date TIMESTAMP,
    view_count COUNTER,
    like_count COUNTER,
    comment_count COUNTER,
    status TEXT,  // processing, ready, failed
    thumbnail_url TEXT,
    manifest_url TEXT,
    tags LIST<TEXT>,
    category TEXT
);

-- Index for user's videos
CREATE TABLE videos_by_user (
    user_id UUID,
    upload_date TIMESTAMP,
    video_id UUID,
    PRIMARY KEY (user_id, upload_date, video_id)
) WITH CLUSTERING ORDER BY (upload_date DESC);

-- Index for category browsing
CREATE TABLE videos_by_category (
    category TEXT,
    upload_date TIMESTAMP,
    video_id UUID,
    view_count BIGINT,
    PRIMARY KEY (category, view_count, video_id)
) WITH CLUSTERING ORDER BY (view_count DESC);
```

### Video Files (S3 Structure)

```
videos/
  ├── {video_id}/
  │   ├── original/
  │   │   └── video.mp4
  │   ├── 360p/
  │   │   ├── playlist.m3u8
  │   │   ├── segment_000.ts
  │   │   ├── segment_001.ts
  │   │   └── ...
  │   ├── 720p/
  │   │   └── ...
  │   ├── 1080p/
  │   │   └── ...
  │   ├── thumbnails/
  │   │   ├── thumb_1.jpg
  │   │   ├── thumb_2.jpg
  │   │   └── thumb_3.jpg
  │   └── master.m3u8
```

### Analytics (BigQuery/Redshift)

```sql
CREATE TABLE video_views (
    view_id STRING,
    video_id STRING,
    user_id STRING,
    timestamp TIMESTAMP,
    duration_watched INT,
    quality STRING,
    device_type STRING,
    country STRING,
    city STRING,
    bandwidth_kbps INT,
    buffer_events INT
)
PARTITION BY DATE(timestamp);

-- Aggregated metrics for quick access
CREATE TABLE video_metrics_daily (
    video_id STRING,
    date DATE,
    view_count INT64,
    unique_viewers INT64,
    total_watch_time INT64,
    avg_watch_percentage FLOAT64,
    PRIMARY KEY (video_id, date)
);
```

## Trade-Off Discussions

### 1. Storage Replication

**Full Replication (3x):**
- ✅ High durability (99.999999999%)
- ✅ Fast reads from multiple regions
- ❌ 3x storage cost
- ❌ Expensive for 4+ EB of data

**Erasure Coding (1.5x):**
- ✅ Lower cost (50% less than replication)
- ✅ Good durability
- ❌ Slower reconstruction on failures
- ❌ More complex

**Recommendation**: Use erasure coding for cold content (>6 months old), full replication for hot content.

### 2. Transcoding: On-Upload vs On-Demand

**On-Upload Transcoding:**
- ✅ Immediate availability after processing
- ✅ Predictable user experience
- ❌ Wastes compute on videos never watched
- ❌ High upfront cost

**On-Demand Transcoding:**
- ✅ Only transcode popular videos
- ✅ Lower compute cost
- ❌ First viewer waits for transcoding
- ❌ Unpredictable latency

**Recommendation**: Transcode to 480p immediately, transcode higher qualities on-demand based on view count.

### 3. CDN: Build vs Buy

**Build Own CDN:**
- ✅ Full control
- ✅ Lower cost at massive scale (Netflix)
- ❌ Huge upfront investment
- ❌ Maintenance complexity

**Use Third-Party CDN:**
- ✅ Quick to deploy
- ✅ Global reach immediately
- ❌ Ongoing costs scale with traffic
- ❌ Less control

**Recommendation**: Use commercial CDN initially (Cloudflare/Akamai), build own when traffic exceeds 100 Tbps.

### 4. Video Format: HLS vs DASH

**HLS (HTTP Live Streaming):**
- ✅ Native support on iOS/Safari
- ✅ Simpler implementation
- ❌ Less efficient than DASH

**DASH (Dynamic Adaptive Streaming):**
- ✅ Better quality adaptation
- ✅ Industry standard
- ❌ Requires JavaScript player

**Recommendation**: Support both (generate both manifests) for maximum compatibility.

## Scaling Strategies

### Phase 1: MVP (< 10K videos)

```
[Simple Upload] → [S3] → [Lambda Transcode] → [CloudFront CDN]
```

- Serverless transcoding
- Single region
- CDN for delivery
- Handles 100 concurrent viewers

### Phase 2: Growth (10K - 1M videos)

```
[Upload Service] → [S3] → [Transcoding Workers x10] → [Multi-region S3]
                                ↓
                          [CloudFront CDN]
```

- Dedicated transcoding cluster
- Multi-region storage
- Metadata database (PostgreSQL)
- Handles 10K concurrent viewers

### Phase 3: Scale (1M - 100M videos)

```
[Multi-region Upload] → [S3 Multi-region] → [Transcoding Cluster x100]
                              ↓
[Regional CDN Caches] ← [Global CDN]
                              ↓
                      [Cassandra Cluster]
```

- Parallel transcoding at scale
- Regional CDN presence
- Cassandra for metadata
- ML-based recommendations
- Handles 100K concurrent viewers

### Phase 4: Global Scale (100M+ videos)

```
[Global Load Balancer]
        ↓
[Multi-region Architecture per Continent]
        ↓
[Microservices (1000+ instances)]
        ↓
[Distributed Transcoding (10K+ workers)]
        ↓
[Multi-tier CDN with Edge Computing]
        ↓
[Petabyte-scale Storage]
```

- Own CDN infrastructure
- Edge computing for personalization
- Advanced ML recommendations
- Real-time analytics
- Handles 1M+ concurrent viewers

## Interview Talking Points

### Key Discussion Areas

**1. Clarification Questions:**
- "Do we need live streaming or just VOD?"
- "What's the expected upload-to-view ratio?"
- "Should we optimize for cost or latency?"
- "Do we need DRM for content protection?"

**2. Bottlenecks:**
- "Transcoding is the main bottleneck"
  - Solution: Massive parallel processing, selective transcoding
- "Storage costs dominate at scale"
  - Solution: Compression, erasure coding for old content
- "CDN egress costs are significant"
  - Solution: Build own CDN at 100 Tbps+

**3. Optimizations:**
- "Transcode only popular videos to 4K"
- "Use CDN cache warming for trending content"
- "Implement lazy deletion (mark as deleted, purge later)"
- "Compress old videos more aggressively"

**4. Advanced Features:**
- "How would you add live streaming?"
  - Use RTMP ingest → transcode in real-time → push to CDN
- "How would you handle copyright detection?"
  - Generate video fingerprints, compare against database
- "How would you implement DVR for live streams?"
  - Store live stream segments, allow time-shifted playback

## Summary

Video streaming platform design demonstrates:

**Core Challenges:**
1. **Massive Storage**: 4+ EB for 5 years of content
2. **High Bandwidth**: 100+ Tbps at peak
3. **Global Distribution**: <100ms latency worldwide
4. **Adaptive Streaming**: Smooth playback on any connection

**Key Solutions:**
- CDN-first architecture (90%+ cache hit rate)
- Parallel transcoding pipeline
- HLS/DASH for adaptive bitrate
- Multi-tier caching strategy
- Intelligent quality selection

**Scale Achievements:**
- 500M daily active users
- 5B video views per day
- 1M concurrent viewers
- 99.99% availability
- <2 second video start time

This design mirrors Netflix, YouTube, and other major streaming platforms, proving that CDN optimization and intelligent transcoding are critical for video streaming at scale.
