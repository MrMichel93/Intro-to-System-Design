# Back-of-Envelope Design Exercises

Practice capacity estimation with these hands-on exercises!

## Exercise 1: School Assignment Submission System

### Scenario
Design a system for students to submit assignments digitally.

### Given
- 10,000 students
- Each student submits 10 assignments per semester (4 months)
- Average assignment: PDF, 5 MB
- Assignments kept for 2 years
- Peak time: Last day before deadline (30% submit on last day)

### Calculate
1. Storage needed for 2 years
2. Bandwidth during peak time
3. Requests per second during peak
4. How many servers if each handles 100 concurrent uploads?

<details>
<summary>Solution</summary>

**1. Storage Calculation:**
```
Assignments per semester: 10,000 students × 10 = 100,000 assignments
Semesters in 2 years: 2 years × 2 = 4 semesters
Total assignments: 100,000 × 4 = 400,000 assignments
Storage: 400,000 × 5 MB = 2,000,000 MB = 2 TB
```

**2. Peak Bandwidth:**
```
Assignments on last day: 100,000 × 30% = 30,000 assignments
Assume spread over 12 hours: 30,000 / 12 = 2,500 per hour
Per second: 2,500 / 3,600 ≈ 0.7 uploads/sec
Bandwidth: 0.7 × 5 MB = 3.5 MB/sec
Peak (with bursts): 10 MB/sec
```

**3. Servers Needed:**
```
Peak concurrent uploads: ~70 (assuming 100 seconds per upload)
Servers: 70 / 100 = 1 server (with buffer: 2-3 servers)
```

**Recommendations:**
- Use cloud storage (S3)
- Implement upload queue
- Send confirmation emails asynchronously
- Add deadline warnings to spread submissions

</details>

---

## Exercise 2: Gaming Leaderboard System

### Scenario
Real-time leaderboard for a mobile game.

### Given
- 5 million active players
- Each player plays 5 games per day
- Each game updates score (write)
- Players check leaderboard 10 times per day (read)
- Store top 1 million players
- Leaderboard entry: Player ID (8 bytes) + Score (4 bytes) + Name (50 bytes)

### Calculate
1. Read and write QPS
2. Storage for leaderboard
3. How to handle hot leaderboard queries?

<details>
<summary>Solution</summary>

**1. QPS Calculation:**
```
Writes (score updates):
- Games per day: 5M × 5 = 25M games
- Per second: 25M / 86,400 ≈ 290 writes/sec
- Peak (10x): 2,900 writes/sec

Reads (leaderboard views):
- Views per day: 5M × 10 = 50M views
- Per second: 50M / 86,400 ≈ 580 reads/sec
- Peak (10x): 5,800 reads/sec
```

**2. Storage:**
```
Per entry: 8 + 4 + 50 = 62 bytes
Top 1M players: 1M × 62 bytes = 62 MB
✅ Easily fits in memory!
```

**3. Optimization Strategy:**
```
- Keep entire leaderboard in Redis (62 MB)
- Use sorted sets (Redis ZADD, ZRANK)
- Update scores in real-time
- Database as backup, not primary
- Cache top 100 separately (most queried)
```

</details>

---

## Exercise 3: Design a Pastebin Service

### Scenario
Users can paste code snippets and share links.

### Given
- 1 million pastes per day
- Average paste: 10 KB
- Read/Write ratio: 10:1
- Pastes expire after 30 days
- Store paste content + metadata

### Calculate
1. Storage for 30 days
2. Read/Write QPS
3. Bandwidth requirements
4. Design short URL generation

<details>
<summary>Solution</summary>

**1. Storage:**
```
Daily pastes: 1M
30 days: 1M × 30 = 30M pastes
Content: 30M × 10 KB = 300,000 MB = 300 GB
Metadata (200 bytes each): 30M × 200 = 6 GB
Total: ~306 GB ✅ Single database sufficient
```

**2. QPS:**
```
Writes: 1M / 86,400 ≈ 12 writes/sec
Reads: 1M × 10 = 10M / 86,400 ≈ 116 reads/sec
Peak (5x): 60 writes/sec, 580 reads/sec
```

**3. Bandwidth:**
```
Write: 12 × 10 KB = 120 KB/sec
Read: 116 × 10 KB = 1.16 MB/sec
Peak read: 5.8 MB/sec
```

**4. Short URL Design:**
```
Base62 encoding (a-z, A-Z, 0-9): 62 characters
7 characters: 62^7 = 3.5 trillion combinations
30M pastes = 0.0009% utilization ✅ Plenty of space

Generation strategy:
- Use counter or random ID
- Convert to base62
- Check for collision (rare)
- Store mapping in database
```

**Architecture:**
```
[User] → [Load Balancer] → [Web Servers]
                               ↓
                          [Redis Cache]
                               ↓
                         [Database (SQL)]
```

</details>

---

## Exercise 4: Design Instagram Stories

### Scenario
Stories feature: 24-hour temporary photo/video posts.

### Given
- 500 million daily active users
- 30% post a story daily = 150M stories
- Average story: 2 MB (photo) or 10 MB (video)
- 70% photos, 30% videos
- Each story viewed by 100 people on average
- Stories expire after 24 hours

### Calculate
1. Storage needed (for 24 hours)
2. Daily views and bandwidth
3. Storage savings from 24-hour expiry
4. CDN requirements

<details>
<summary>Solution</summary>

**1. Storage (24 hours):**
```
Photos: 150M × 70% × 2 MB = 210,000 GB = 210 TB
Videos: 150M × 30% × 10 MB = 450,000 GB = 450 TB
Total: 660 TB for 24 hours

Without expiry (30 days): 660 TB × 30 = 19,800 TB = 20 PB
Savings from 24h expiry: 20 PB - 0.66 PB = Massive savings!
```

**2. Daily Views:**
```
Views: 150M stories × 100 viewers = 15 billion views/day
Views per second: 15B / 86,400 ≈ 174,000 views/sec
Peak (3x): 522,000 views/sec
```

**3. Bandwidth:**
```
Photo views: 174K × 70% × 2 MB = 243,600 MB/sec = 244 GB/sec
Video views: 174K × 30% × 10 MB = 522,000 MB/sec = 522 GB/sec
Total: ~766 GB/sec = 6 Terabits/sec
Peak: 18 Terabits/sec
```

**4. Design Implications:**
```
✅ MUST use CDN (cannot serve 6 Tbps from origin)
✅ Aggressive caching (popular stories)
✅ Separate storage for stories vs permanent posts
✅ Automated cleanup (delete after 24 hours)
✅ Compress videos heavily
✅ Multiple quality versions (adaptive streaming)
```

**Architecture:**
```
[CDN Edge Locations] ← Hot stories (80/20 rule)
         ↑
[Origin Storage] ← All stories
         ↑
[Upload Servers] ← [Users]
```

</details>

---

## Exercise 5: Design Google Drive (Simplified)

### Scenario
Cloud file storage and sync service.

### Given
- 100 million users
- Each user stores 5 GB on average
- 10% of users are heavy users (50 GB each)
- Each user accesses 10 files per day
- Average file: 2 MB
- File sync happens 5 times per day per user

### Calculate
1. Total storage needed
2. Daily sync traffic
3. File metadata storage
4. Bandwidth requirements

<details>
<summary>Solution</summary>

**1. Total Storage:**
```
Regular users: 100M × 90% × 5 GB = 450M GB = 450 PB
Heavy users: 100M × 10% × 50 GB = 500M GB = 500 PB
Total: 950 PB ≈ 1 Exabyte (EB)

With replication (3x): 3 EB
```

**2. Daily Sync Traffic:**
```
Sync operations: 100M users × 5 syncs = 500M syncs/day
Assume 10% of file changed: 2 MB × 10% = 200 KB per sync
Traffic: 500M × 200 KB = 100,000 GB = 100 TB/day
```

**3. File Metadata:**
```
Estimate 20 files per user: 100M × 20 = 2B files
Per file metadata: 500 bytes (name, path, size, timestamps, permissions)
Total: 2B × 500 = 1,000 GB = 1 TB
```

**4. Bandwidth:**
```
File accesses: 100M × 10 = 1B file accesses/day
Per second: 1B / 86,400 ≈ 11,600 files/sec
Bandwidth: 11,600 × 2 MB = 23,200 MB/sec ≈ 23 GB/sec
Peak (5x): 115 GB/sec
```

**Design Considerations:**
```
✅ Block-level sync (don't upload entire file if only part changed)
✅ Deduplication (same file uploaded by multiple users)
✅ Compression
✅ Differential sync (only changes)
✅ Multiple data centers for reliability
✅ Separate cold storage for rarely accessed files
```

**Optimization Example:**
```
Block-level sync: 2 MB file, 10 KB changed
- Without: Upload 2 MB
- With: Upload 10 KB
- Savings: 99.5%!
```

</details>

---

## Exercise 6: Estimate Resources for YouTube

### Scenario
Design capacity estimates for a video platform like YouTube.

### Given
- 500 hours of video uploaded every minute
- Average video length: 10 minutes
- Average video size: 1 GB (after compression)
- 5 billion video views per day
- Average view watches 50% of video
- Store videos for 10 years
- Multiple quality versions (360p, 720p, 1080p, 4K)

### Your Task
Calculate:
1. **Daily upload volume** (storage needed per day)
2. **Total storage for 10 years** (with multiple quality versions)
3. **Daily bandwidth for views** (serving videos to users)
4. **Number of CDN edge locations** (assuming 100 Gbps per location)
5. **Storage cost estimation** (at $0.02 per GB per month)

<details>
<summary>Solution</summary>

**1. Daily Upload Volume:**
```
Uploads per minute: 500 hours = 500 × 60 = 30,000 minutes of video
Number of videos (10 min each): 30,000 / 10 = 3,000 videos per minute

Storage per minute: 3,000 videos × 1 GB = 3,000 GB = 3 TB
Storage per hour: 3 TB × 60 = 180 TB
Storage per day: 180 TB × 24 = 4,320 TB ≈ 4.3 PB per day
```

**2. Total Storage for 10 Years:**
```
Original quality per year: 4.3 PB × 365 = 1,570 PB ≈ 1.6 EB per year
10 years: 1.6 EB × 10 = 16 EB

Multiple quality versions:
- 360p: 0.3 GB (30% of original)
- 720p: 0.6 GB (60% of original)  
- 1080p: 1 GB (100% - this is our baseline)
- 4K: 4 GB (400% of original)

Total storage multiplier: 0.3 + 0.6 + 1 + 4 = 5.9x
Total storage: 16 EB × 5.9 = 94.4 EB ≈ 95 exabytes

With replication (3x): 95 EB × 3 = 285 EB
```

**3. Daily Bandwidth for Views:**
```
Views per day: 5 billion
Average watch duration: 50% of 10 minutes = 5 minutes

Most views are 720p: ~0.6 GB per video
Bandwidth per view: 0.6 GB × 50% = 0.3 GB

Total data transfer: 5B views × 0.3 GB = 1.5B GB = 1.5 PB per day
Bandwidth: 1.5 PB / 86,400 seconds = ~17.4 GB/second = 139 Gigabits/sec

Peak traffic (3x average): 417 Gigabits/sec
```

**4. CDN Edge Locations:**
```
Peak bandwidth needed: 417 Gbps
Capacity per location: 100 Gbps

Locations needed: 417 / 100 = 4.17 ≈ 5 locations (minimum)

For redundancy and global coverage: 50-100 edge locations
(Distribute across continents, major cities)

Reasoning:
- Not all traffic to one location
- Serve users from nearest location
- Need redundancy for failures
- Consider regional regulations
```

**5. Storage Cost Estimation:**
```
Total storage: 95 EB (without replication)
= 95,000,000,000 GB

Monthly cost: 95,000,000,000 GB × $0.02 = $1,900,000,000
Annual cost: $1.9B × 12 = $22.8 billion per year

With replication (3x): $68.4 billion per year

Cost optimizations:
- Use cheaper cold storage for old videos
- Compress older videos more aggressively
- Delete videos with zero views after X years
- Use tiered storage (hot/warm/cold)
- Negotiate bulk pricing

Realistic costs with optimizations: $5-10 billion per year
```

**Additional Considerations:**

**Transcoding:**
```
Need to convert uploaded videos to multiple formats:
- 3,000 videos per minute
- ~5 formats per video
- 15,000 transcoding jobs per minute
- Requires massive transcoding infrastructure
```

**Database (Metadata):**
```
Per video metadata: 10 KB
Daily uploads: 3,000 videos/min × 60 × 24 = 4.32M videos/day
Daily metadata: 4.32M × 10 KB = 43.2 GB/day
10 years: 43.2 GB × 365 × 10 = 157.7 TB (manageable)
```

**Comments and Interactions:**
```
Assume 1% of videos get comments
Average 100 comments per video
Comment storage: ~1 KB each

Daily comment storage: 4.32M × 1% × 100 × 1 KB = 432 GB/day
10 years: 432 GB × 365 × 10 = 1.58 PB (small compared to videos)
```

**Cache Strategy:**
```
80/20 rule: 20% of videos get 80% of views
Hot videos to cache: 20% of library
Cache size needed: 95 EB × 20% = 19 EB (still too large)

Better strategy: Cache by recency
- Videos from last 7 days: 4.3 PB × 7 = 30 PB
- This represents most views
- Much more manageable cache size
```

**System Design Insights:**

1. **Storage is the biggest cost** - Need to optimize aggressively
2. **CDN is essential** - Can't serve 400+ Gbps from origin
3. **Transcoding pipeline is critical** - Must be highly scalable
4. **Tiered storage is necessary** - Can't keep everything hot
5. **Intelligent caching** - Cache recent and popular, not everything
6. **Geographic distribution** - Must have servers worldwide
7. **Cost management is key** - Storage costs dominate, must optimize

**Technology Stack Suggestions:**
- **Storage**: Object storage (S3, GCS) with lifecycle policies
- **CDN**: Global CDN (CloudFront, Akamai, Fastly)
- **Transcoding**: Distributed transcoding clusters using FFmpeg (with Kubernetes/job queues)
- **Database**: Distributed SQL for metadata (Spanner, CockroachDB)
- **Cache**: Redis/Memcached for hot metadata
- **Analytics**: Data warehouse (BigQuery, Snowflake) for view tracking

</details>

---

## Practice Template

Use this template for your own calculations:

```
System: _______________________

Given Requirements:
- Users:
- Actions per user:
- Data sizes:
- Retention period:
- Other constraints:

Calculations:

1. Storage:
   - Formula:
   - Result:

2. QPS (Queries Per Second):
   - Writes:
   - Reads:
   - Peak:

3. Bandwidth:
   - Upload:
   - Download:
   - Peak:

4. Special Considerations:
   -
   -

Design Decisions:
- Database choice:
- Caching strategy:
- CDN usage:
- Scaling approach:
```

---

## Tips for Success

1. **Always state assumptions** - Make it clear what you're assuming
2. **Use round numbers** - 86,400 → 100,000 makes math easier
3. **Check your units** - MB vs GB vs TB
4. **Consider peak vs average** - Design for peak!
5. **Don't forget overhead** - Replication, backups, indexes
6. **Sanity check** - Does 1 PB/day sound reasonable?

## Next Steps

Once comfortable with calculations, move to [Trade-Offs](./trade-offs.md) to learn about making design decisions based on these numbers!
