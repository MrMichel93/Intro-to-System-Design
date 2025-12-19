# Back-of-Envelope Calculations

## What are Back-of-Envelope Calculations?

**Back-of-envelope calculations** are quick, rough estimates used to understand the scale and requirements of a system. The name comes from literally jotting calculations on the back of an envelope!

In system design, these calculations help you:
- Estimate storage needs
- Calculate bandwidth requirements
- Determine how many servers you need
- Identify potential bottlenecks
- Make informed architecture decisions

**Key Principle**: These are **estimates**, not exact numbers. Being within 10-20% of the correct answer is usually good enough for design decisions.

## Why Do We Need Them?

### Scenario: Design a photo-sharing app

**Without calculations:**
> "We'll need some servers and databases. Let's start building!"

**With calculations:**
> "We expect 10 million photos per month. At 2MB each, that's 20TB/month. We need 240TB storage for a year, plus bandwidth for serving them..."

Calculations help you:
1. **Make realistic plans**
2. **Estimate costs accurately**
3. **Avoid over/under-engineering**
4. **Communicate with stakeholders**
5. **Succeed in interviews**

## Essential Numbers to Know

### Power of 2 (Computer Storage)

```
Byte (B)       = 8 bits
Kilobyte (KB)  = 1,000 bytes     ≈ 10^3  (actually 1,024)
Megabyte (MB)  = 1,000,000 bytes ≈ 10^6
Gigabyte (GB)  = 1 billion bytes ≈ 10^9
Terabyte (TB)  = 1 trillion bytes ≈ 10^12
Petabyte (PB)  = 1,000 TB         ≈ 10^15
```

**Tip**: For estimates, use 1,000 instead of 1,024. It's easier and close enough!

### Time Units

```
1 second       = 1,000 milliseconds (ms)
1 millisecond  = 1,000 microseconds (μs)
1 microsecond  = 1,000 nanoseconds (ns)

Per day:
1 day          = 86,400 seconds ≈ 100,000 seconds (easy math!)
1 month        = 30 days ≈ 2.5 million seconds
1 year         = 365 days ≈ 31.5 million seconds
```

**Tip**: Round to make calculations easier! 86,400 → 100,000 is fine for estimates.

### Latency Numbers Every Programmer Should Know

These are approximate and change over time, but the orders of magnitude remain useful:

```
L1 cache reference                    0.5 ns
L2 cache reference                    7 ns
RAM access                            100 ns
Send 1K bytes over 1 Gbps network     10 μs
Read 1 MB sequentially from RAM       250 μs
Round trip within same datacenter     500 μs
Read 1 MB sequentially from SSD       1 ms
Disk seek                             10 ms
Read 1 MB sequentially from disk      20 ms
Send packet CA → Netherlands → CA     150 ms
```

**Key Takeaways:**
- Memory is FAST (nanoseconds)
- Disk is SLOW (milliseconds)
- Network is in between (microseconds to milliseconds)
- Cross-continent network is VERY SLOW (100+ milliseconds)

### Typical Data Sizes

```
Tiny:
- Short tweet (140 chars)         : ~140 bytes
- SMS message                     : ~160 bytes

Small:
- URL                             : ~100 bytes
- Email                           : ~10 KB
- Webpage (HTML)                  : ~100 KB

Medium:
- Photo (compressed)              : ~2 MB
- Music (MP3, 3 minutes)          : ~3 MB
- Ebook                           : ~5 MB

Large:
- High-quality photo (RAW)        : ~20 MB
- Movie (1080p, 2 hours)          : ~4 GB
- Movie (4K, 2 hours)             : ~20 GB
- Video game                      : ~50 GB
```

### Request Rates

```
Low traffic  : 10 requests/second
Medium       : 1,000 requests/second (1K)
High         : 100,000 requests/second (100K)
Very high    : 1,000,000 requests/second (1M)
```

### Availability Numbers (Nines)

```
99% availability     = 3.65 days downtime/year
99.9% (three nines)  = 8.76 hours downtime/year
99.99% (four nines)  = 52.6 minutes downtime/year
99.999% (five nines) = 5.26 minutes downtime/year
```

## Calculation Strategies

### Strategy 1: Break Down the Problem

**Example: Calculate storage for a video platform**

Instead of guessing, break it down:
1. How many users?
2. How many videos per user per month?
3. Average video size?
4. How long do we keep videos?

**Calculation:**
```
Users: 1 million
Videos per user per month: 0.1 (most users don't upload)
Video size: 100 MB (compressed)
Storage period: Forever

Monthly uploads = 1M users × 0.1 videos × 100 MB
                = 100,000 videos × 100 MB
                = 10,000,000 MB
                = 10 TB per month
                = 120 TB per year
```

### Strategy 2: Work in Scientific Notation

Makes large numbers easier!

**Example: Daily requests**
```
Instead of: 86,400 seconds × 1,000 requests = 86,400,000 requests
Use: ~10^5 seconds × 10^3 requests = 10^8 requests = 100 million
```

### Strategy 3: Use Ratios

**Example: Read vs. Write ratio**

If your system has 100:1 read-to-write ratio:
```
Total requests: 1,000/sec
Writes: ~10/sec
Reads: ~990/sec

Design accordingly:
- Optimize for reads (caching)
- Writes can be slower
- More read replicas than write database
```

### Strategy 4: Round Liberally

**Example:**
```
Instead of: 37.2 GB
Use: 40 GB (easier to work with)

Instead of: 1,234 requests/sec
Use: 1,000 requests/sec or even 10^3 requests/sec
```

## Common Calculations

### 1. Storage Calculation

**Formula:**
```
Storage = Number of items × Average size × Retention period
```

**Example: Twitter-like app**
```
Given:
- 100 million users
- 20% post daily
- Average tweet: 100 bytes (text) + 1 MB (if image, 10% include image)
- Keep forever

Calculation:
Daily tweets = 100M × 20% = 20M tweets
Text storage = 20M × 100 bytes = 2 GB
Image tweets = 20M × 10% = 2M images
Image storage = 2M × 1 MB = 2 TB

Daily storage = 2 GB + 2 TB ≈ 2 TB
Monthly storage = 2 TB × 30 = 60 TB
Yearly storage = 60 TB × 12 = 720 TB
```

### 2. Bandwidth Calculation

**Formula:**
```
Bandwidth = (Data size × Requests) / Time period
```

**Example: Image hosting site**
```
Given:
- 1 million page views per day
- Each page has 10 images
- Average image: 200 KB

Calculation:
Images per day = 1M pages × 10 images = 10M images
Data per day = 10M × 200 KB = 2,000,000 MB = 2 TB

Bandwidth = 2 TB / day
          = 2 TB / 100,000 seconds
          = 0.02 GB/second
          = 20 MB/second
          = 160 Megabits/second (Mbps)
```

### 3. Server Count Estimation

**Formula:**
```
Servers = Total requests per second / Requests per server
```

**Example: API service**
```
Given:
- 10,000 requests per second at peak
- Each server handles 100 requests/sec

Calculation:
Servers needed = 10,000 / 100 = 100 servers

Add buffer for:
- Traffic spikes: +50% = 150 servers
- Redundancy: +20% = 180 servers
- Rolling updates: +10% = 200 servers

Final: ~200 servers
```

### 4. Database Queries Per Second (QPS)

**Formula:**
```
QPS = Requests per second × Queries per request
```

**Example: Social media feed**
```
Given:
- 1,000 users viewing feeds per second
- Each feed requires:
  - 1 user profile query
  - 1 posts query (gets 20 posts)
  - 20 author profile queries

Calculation:
Queries per request = 1 + 1 + 20 = 22 queries
Total QPS = 1,000 × 22 = 22,000 queries/sec

Optimization ideas:
- Cache user profiles: Reduce from 21 to 2 queries
- New QPS = 1,000 × 2 = 2,000 queries/sec (10x improvement!)
```

### 5. Cache Size Estimation

**Formula (80/20 Rule):**
```
Cache size = 20% of hot data
```

**Example: Video platform**
```
Given:
- Total videos: 1 million
- Average video metadata: 1 KB
- 80% of traffic goes to 20% of videos

Calculation:
Hot videos = 1M × 20% = 200,000 videos
Cache size = 200,000 × 1 KB = 200 MB

For video files (if caching):
Hot videos = 200,000
Average size = 10 MB (compressed)
Cache size = 200,000 × 10 MB = 2 TB (need distributed cache!)
```

## Real-World Examples

### Example 1: Design Instagram

**Requirements:**
- 500 million daily active users
- Each user views 50 photos per day
- Each user uploads 1 photo per day
- Photo size: 2 MB (original), 200 KB (compressed)

**Storage Calculation:**
```
Daily uploads = 500M × 1 = 500M photos
Original storage = 500M × 2 MB = 1,000,000 MB = 1 PB/day
Compressed storage = 500M × 200 KB = 100,000 GB = 100 TB/day

Yearly storage (compressed) = 100 TB × 365 = 36,500 TB ≈ 36 PB
```

**Bandwidth Calculation:**
```
Daily photo views = 500M users × 50 photos = 25 billion views
Data transfer = 25B × 200 KB = 5,000,000 GB = 5 PB/day

Bandwidth = 5 PB / 86,400 seconds
          = ~60 GB/second
          = 480 Gigabits/second
```

**Conclusions:**
- Need petabyte-scale storage solution
- Must use CDN (can't serve 480 Gbps from one datacenter)
- Compression is critical (1 PB → 100 TB = 10x savings)
- Caching essential (most views go to recent/popular photos)

---

### Example 2: Design WhatsApp

**Requirements:**
- 2 billion users
- 60 messages per user per day
- Average message: 100 bytes
- 10% messages include media (average 1 MB)
- Messages kept for 30 days

**Storage Calculation:**
```
Daily messages = 2B × 60 = 120B messages

Text messages = 120B × 90% × 100 bytes = 10,800 GB ≈ 11 TB
Media messages = 120B × 10% × 1 MB = 12,000,000 GB = 12 PB

Total daily = 11 TB + 12 PB ≈ 12 PB
30-day storage = 12 PB × 30 = 360 PB
```

**Bandwidth Calculation:**
```
Messages per second = 120B / 86,400 = ~1.4M messages/sec

Text bandwidth = 1.4M × 90% × 100 bytes = 126 MB/sec
Media bandwidth = 1.4M × 10% × 1 MB = 140 GB/sec

Total = ~140 GB/sec
```

**Conclusions:**
- Media dominates storage (12 PB vs 11 TB)
- Need distributed storage across many datacenters
- Media should be compressed and optimized
- Consider limiting media storage duration
- Peak times (evening) likely 3-5x average

---

### Example 3: Design TinyURL

**Requirements:**
- 100 million URLs shortened per month
- Read-to-write ratio: 100:1
- Each URL entry: 500 bytes
- Keep URLs for 5 years

**Storage Calculation:**
```
Monthly new URLs = 100M
Yearly new URLs = 100M × 12 = 1.2B
5-year URLs = 1.2B × 5 = 6B URLs

Storage = 6B × 500 bytes = 3,000 GB = 3 TB
```

**Request Rate Calculation:**
```
Write requests:
- 100M per month = ~40 writes/sec

Read requests:
- 100M × 100 = 10B per month
- 10B / 2.5M seconds = 4,000 reads/sec
```

**Conclusions:**
- Storage is manageable (3 TB)
- Read-heavy (4,000 reads vs 40 writes)
- Cache popular URLs (80/20 rule)
- Cache size: 0.6B URLs × 500 bytes = 300 GB (fits in RAM!)

---

## Tips and Tricks

### 1. Always State Assumptions
"Assuming 1 million users..." helps clarify your thinking.

### 2. Use Round Numbers
1,234,567 → 1 million. Makes calculations much easier!

### 3. Check Your Units
KB vs MB vs GB. Common source of 1000x errors!

### 4. Sanity Check Results
"1 PB per day" - does that sound reasonable? Could a company afford that?

### 5. Consider Peak vs. Average
Systems must handle peak traffic, not just average:
- E-commerce: Black Friday = 10x normal
- Streaming: Evening = 5x daytime
- Social media: Breaking news = unpredictable spikes

### 6. Don't Forget Replication
Production systems typically have:
- Database replicas: 3-5x storage
- Backups: +1x storage
- So "1 TB" becomes "4-6 TB" in reality

### 7. Include Overhead
Systems aren't 100% efficient:
- Database indexes: +20-30% storage
- Network protocols: +10-20% bandwidth
- OS overhead: +10-20% CPU/memory

## Practice Problems

### Problem 1: YouTube Estimation
**Calculate storage for YouTube:**
- 500 hours of video uploaded per minute
- Average video: 10 minutes = 1 GB
- Store for 10 years

<details>
<summary>Solution</summary>

```
Uploads per minute = 500 hours = 500 × 60 = 30,000 minutes
Videos per minute = 30,000 / 10 = 3,000 videos
Storage per minute = 3,000 × 1 GB = 3 TB

Per day = 3 TB × 60 × 24 = 4,320 TB ≈ 4 PB
Per year = 4 PB × 365 = 1,460 PB ≈ 1.5 exabytes (EB)
Per 10 years = 1.5 EB × 10 = 15 EB
```

**Conclusion**: Need exabyte-scale storage!

</details>

### Problem 2: Uber Trip Calculation
**Estimate database writes for Uber:**
- 10 million trips per day
- Each trip creates:
  - 1 trip record
  - 2 location updates per second
  - Average trip: 20 minutes

<details>
<summary>Solution</summary>

```
Trips per day = 10M

Trip records = 10M writes/day

Location updates per trip:
- 2 per second × 60 seconds × 20 minutes = 2,400 updates

Total location updates = 10M × 2,400 = 24B updates/day

Total writes/day = 10M + 24B = 24B (location updates dominate)

Writes per second = 24B / 100,000 = 240,000 writes/sec
```

**Conclusion**: Location updates are the bottleneck. Need strategy:
- Batch updates
- Use time-series database
- Sample location (maybe 1 update per 5 seconds instead)

</details>

## Summary

Key points for back-of-envelope calculations:

✅ **Break down problems** into smaller pieces  
✅ **Use round numbers** for easier math  
✅ **State assumptions** clearly  
✅ **Check units** (KB vs MB vs GB)  
✅ **Sanity check** results  
✅ **Consider peak traffic**, not just average  
✅ **Add buffers** for replication and overhead  
✅ **Focus on order of magnitude**, not exact numbers  

**Remember**: Being roughly right is better than being precisely wrong!

## Next Steps

Now that you can estimate system requirements, move on to **[CAP Theorem](../02-cap-theorem/)** to understand the fundamental trade-offs in distributed systems!
