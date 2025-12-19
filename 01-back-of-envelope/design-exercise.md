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
