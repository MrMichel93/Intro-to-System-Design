# Numbers Every System Designer Should Know

Essential performance metrics, latency numbers, capacities, and benchmarks for making informed design decisions.

## Table of Contents
- [Latency Numbers](#latency-numbers)
- [Storage Capacities](#storage-capacities)
- [Network Bandwidth](#network-bandwidth)
- [Data Sizes](#data-sizes)
- [QPS Benchmarks](#qps-benchmarks)
- [Cost Estimates](#cost-estimates)
- [Availability Calculations](#availability-calculations)
- [Power of Two Conversions](#power-of-two-conversions)
- [Time Conversions](#time-conversions)

---

## Latency Numbers

Understanding latency is critical for system design. Here are typical latencies you should know:

### Hardware Operations
```
L1 cache reference                       0.5 ns
L2 cache reference                       7 ns       (14x L1)
Branch mispredict                        5 ns
Mutex lock/unlock                        100 ns
Main memory reference                    100 ns     (200x L1, 20x L2)
Compress 1KB with Snappy                 10,000 ns  (10 μs)
Send 1KB over 1 Gbps network            10,000 ns  (10 μs)
Read 1 MB sequentially from memory      250,000 ns  (250 μs)
Round trip within same datacenter       500,000 ns  (500 μs)
Disk seek                                10,000,000 ns (10 ms)
Read 1 MB sequentially from SSD         1,000,000 ns (1 ms)
Read 1 MB sequentially from disk        30,000,000 ns (30 ms)
Send packet CA→Netherlands→CA           150,000,000 ns (150 ms)
```

### Visualized Latency Comparison
```
Operation                          Time (approximate)
─────────────────────────────────────────────────────
L1 cache                          0.5 ns          ▏
L2 cache                          7 ns            █
Main memory                       100 ns          ████████████████████
Compress 1KB (Snappy)             10 μs           ████████ (2,000 blocks)
SSD random read                   150 μs          ███████████████████ (30,000 blocks)
SSD sequential read (1MB)         1 ms            ████████ (200,000 blocks)
HDD seek                          10 ms           █████ (2,000,000 blocks)
HDD sequential read (1MB)         30 ms           ████████ (6,000,000 blocks)
Packet round-trip (CA→Europe→CA)  150 ms          ████████ (30,000,000 blocks)
```

### Key Takeaways
- **Memory is fast**: RAM access is ~100 ns
- **SSD is faster than HDD**: 100-1000x faster for random access
- **Network within datacenter**: ~500 μs round trip
- **Cross-continental network**: ~150 ms round trip
- **Disk seeks are expensive**: Avoid random disk access when possible

---

## Storage Capacities

### Data Size Units
```
Bit (b)                          Basic unit of information
Byte (B)                         8 bits
Kilobyte (KB)                    1,000 bytes           ≈ 10^3
Megabyte (MB)                    1,000,000 bytes       ≈ 10^6
Gigabyte (GB)                    1,000,000,000 bytes   ≈ 10^9
Terabyte (TB)                    1,000 GB              ≈ 10^12
Petabyte (PB)                    1,000 TB              ≈ 10^15
Exabyte (EB)                     1,000 PB              ≈ 10^18
```

### Typical Storage Capacities
```
Component                        Typical Capacity
──────────────────────────────────────────────────
L1 Cache                         32-64 KB
L2 Cache                         256 KB - 1 MB
L3 Cache                         8-64 MB
RAM (Server)                     16-512 GB (up to several TB)
SSD (Consumer)                   250 GB - 4 TB
SSD (Enterprise)                 1-30 TB
HDD (Consumer)                   1-16 TB
HDD (Enterprise)                 4-20 TB
Cloud Object Storage             Unlimited (pay per GB)
```

### Typical Data Sizes
```
Type                             Size
────────────────────────────────────────
ASCII character                  1 byte
Unicode character (UTF-8)        1-4 bytes (average ~2 bytes)
Integer (32-bit)                 4 bytes
Long (64-bit)                    8 bytes
UUID                             16 bytes (128 bits)
IPv4 Address                     4 bytes
IPv6 Address                     16 bytes
Timestamp (Unix epoch)           8 bytes (64-bit)

Social Media Post
──────────────────────
Short text post (280 chars)      ~500 bytes
Tweet with metadata              ~2 KB
Instagram post metadata          ~2 KB
TikTok video (15 sec, 720p)      ~5-10 MB

Images
──────
Small thumbnail (100x100)        ~5 KB
Profile picture (400x400)        ~50 KB
High-res photo (4K)              ~2-5 MB
RAW photo                        ~20-50 MB

Videos
──────
1 min video (480p)               ~10 MB
1 min video (720p)               ~50 MB
1 min video (1080p)              ~100 MB
1 min video (4K)                 ~400 MB
Netflix movie (1080p, 2hr)       ~3-5 GB
Netflix movie (4K, 2hr)          ~14-20 GB

Audio
─────
1 min audio (low quality)        ~1 MB
1 min audio (high quality)       ~3 MB
Music track (3 min, MP3)         ~3-5 MB
Music track (3 min, FLAC)        ~30-50 MB

Documents
─────────
Plain text document (10 pages)   ~20 KB
PDF document (10 pages)          ~100 KB - 1 MB
PowerPoint presentation          ~5-20 MB
```

---

## Network Bandwidth

### Network Connection Speeds
```
Connection Type                  Bandwidth          Throughput (ideal)
──────────────────────────────────────────────────────────────────────
Dial-up                          56 Kbps            ~7 KB/s
DSL                              1-100 Mbps         125 KB/s - 12.5 MB/s
Cable Internet                   10-500 Mbps        1.25-62.5 MB/s
Fiber (Home)                     100 Mbps - 1 Gbps  12.5-125 MB/s
4G LTE                           5-50 Mbps          0.6-6.25 MB/s
5G                               100-1,000 Mbps     12.5-125 MB/s
Corporate Network                100 Mbps - 10 Gbps 12.5 MB/s - 1.25 GB/s
Datacenter (within rack)         10-100 Gbps        1.25-12.5 GB/s
Cross-datacenter backbone        100 Gbps - 1 Tbps  12.5 GB/s - 125 GB/s
```

### Bandwidth Calculations
```
To transfer 1 GB of data:

Connection        Time
────────────────────────
1 Mbps            2 hours 13 minutes
10 Mbps           13 minutes
100 Mbps          80 seconds
1 Gbps            8 seconds
10 Gbps           0.8 seconds
```

### Practical Bandwidth Usage
```
Activity                         Required Bandwidth
──────────────────────────────────────────────────
Web browsing                     1-5 Mbps
Email                            1 Mbps
Music streaming                  128-320 Kbps
SD video streaming (480p)        3-5 Mbps
HD video streaming (1080p)       5-8 Mbps
4K video streaming               25-50 Mbps
Video conferencing (HD)          2-4 Mbps
Online gaming                    3-6 Mbps
```

---

## QPS Benchmarks

### Database Operations (Single Server)

```
Database                  Reads/sec        Writes/sec      Notes
───────────────────────────────────────────────────────────────────
MySQL (optimized)         ~10,000          ~5,000          SSD, indexed queries
PostgreSQL (optimized)    ~10,000          ~5,000          SSD, indexed queries
MongoDB (simple docs)     ~15,000          ~10,000         SSD, simple documents
Redis (cache)             ~100,000         ~80,000         In-memory, simple ops
Cassandra (per node)      ~5,000           ~5,000          Write-optimized
Elasticsearch             ~3,000           ~2,000          Complex search queries
DynamoDB                  Unlimited*        Unlimited*      Pay per capacity unit
```
*DynamoDB and other cloud services scale based on provisioned capacity

### Cache Performance

```
Operation                        QPS (per instance)
─────────────────────────────────────────────────
Redis GET (single key)           ~100,000
Redis SET (single key)           ~80,000
Redis MGET (pipeline)            ~200,000+
Memcached GET                    ~100,000
Memcached SET                    ~90,000
```

### Web Server Performance

```
Server                    Requests/sec     Notes
──────────────────────────────────────────────────
Nginx (static content)    ~50,000          Simple file serving
Nginx (reverse proxy)     ~20,000          Proxying to backend
Apache (static)           ~10,000          Traditional MPM
Node.js (simple API)      ~10,000          Single-threaded
Go (simple API)           ~30,000          Highly concurrent
Python/Django             ~1,000           WSGI with Gunicorn
Ruby/Rails                ~500             Traditional stack
```

### Message Queue Throughput

```
System                    Messages/sec     Notes
─────────────────────────────────────────────────
Kafka (per broker)        ~500,000         Batching enabled
RabbitMQ                  ~20,000          Standard config
AWS SQS (standard)        Unlimited*       Batching recommended
AWS Kinesis (per shard)   ~1,000           1 MB/sec per shard
```

### API Gateway Performance

```
Gateway                   Requests/sec     Notes
─────────────────────────────────────────────────
AWS API Gateway           10,000*          Account limit, can increase
Kong                      ~10,000          Per instance
Nginx                     ~20,000          As reverse proxy
Envoy                     ~15,000          Per instance
```

---

## Cost Estimates

### Cloud Compute (AWS EC2, approximate monthly costs)

```
Instance Type       vCPU    Memory      Storage     Cost/month
──────────────────────────────────────────────────────────────
t3.micro            2       1 GB        EBS only    ~$8
t3.small            2       2 GB        EBS only    ~$16
t3.medium           2       4 GB        EBS only    ~$32
m5.large            2       8 GB        EBS only    ~$70
m5.xlarge           4       16 GB       EBS only    ~$140
m5.2xlarge          8       32 GB       EBS only    ~$280
m5.4xlarge          16      64 GB       EBS only    ~$560
c5.large (compute)  2       4 GB        EBS only    ~$60
r5.large (memory)   2       16 GB       EBS only    ~$115
```

### Cloud Storage (approximate costs)

```
Service                     Cost per GB/month    Notes
──────────────────────────────────────────────────────
AWS S3 (Standard)           $0.023               First 50 TB
AWS S3 (Infrequent)         $0.0125              Retrieval fees apply
AWS S3 (Glacier)            $0.004               Archive storage
AWS EBS (SSD)               $0.10                Attached to EC2
AWS EBS (HDD)               $0.045               Throughput optimized
Google Cloud Storage        $0.020               Standard tier
Azure Blob Storage          $0.018               Hot tier
```

### Database Costs (managed services, approximate)

```
Service                     Cost/month           Configuration
────────────────────────────────────────────────────────────────
RDS MySQL (db.t3.small)     ~$30                 2 vCPU, 2 GB RAM
RDS PostgreSQL (db.m5.large) ~$150               2 vCPU, 8 GB RAM
DynamoDB (on-demand)        $1.25 per million reads  $1.25 per million writes
MongoDB Atlas (M10)         ~$60                 2 GB RAM, 10 GB storage
Redis ElastiCache (small)   ~$40                 cache.t3.small
```

### Data Transfer Costs (egress from cloud)

```
Provider                    First 1 GB    1-10 TB       10-50 TB
───────────────────────────────────────────────────────────────
AWS                         Free          $0.09/GB      $0.085/GB
Google Cloud                Free          $0.12/GB      $0.11/GB
Azure                       Free          $0.087/GB     $0.083/GB
Cloudflare (CDN)            Free          Free          Free (on paid plans)
```

### CDN Costs

```
Provider                    Cost per GB      Notes
─────────────────────────────────────────────────
AWS CloudFront              $0.085           First 10 TB
Cloudflare                  $0.04-0.06       Pro/Business plans
Fastly                      $0.12            Request fees also apply
Akamai                      Custom           Enterprise pricing
```

---

## Availability Calculations

### Uptime vs Downtime

```
Availability  Downtime/Year   Downtime/Month  Downtime/Week   Nines
────────────────────────────────────────────────────────────────────
90%           36.5 days       72 hours        16.8 hours      1 nine
95%           18.25 days      36 hours        8.4 hours       -
99%           3.65 days       7.2 hours       1.68 hours      2 nines
99.5%         1.83 days       3.6 hours       50.4 minutes    -
99.9%         8.76 hours      43.2 minutes    10.1 minutes    3 nines
99.95%        4.38 hours      21.6 minutes    5.04 minutes    -
99.99%        52.6 minutes    4.32 minutes    1.01 minutes    4 nines
99.995%       26.3 minutes    2.16 minutes    30.2 seconds    -
99.999%       5.26 minutes    25.9 seconds    6.05 seconds    5 nines
99.9999%      31.5 seconds    2.59 seconds    0.605 seconds   6 nines
```

### Serial vs Parallel System Availability

**Serial (Chain)**: Components in sequence - overall availability is the product
```
Component A (99.9%) → Component B (99.9%)
Overall = 99.9% × 99.9% = 99.8%

Component A (99%) → Component B (99%) → Component C (99%)
Overall = 99% × 99% × 99% = 97%
```

**Parallel (Redundancy)**: Components in parallel - availability improves
```
Component A (99%) in parallel with Component B (99%)
Overall = 1 - (1 - 99%) × (1 - 99%) = 99.99%

Three components at 90% each in parallel:
Overall = 1 - (1 - 90%)³ = 99.9%
```

---

## Power of Two Conversions

### Exact Values
```
Power   Exact Value      Approximate     Human Readable
──────────────────────────────────────────────────────
2^10    1,024            ~1 thousand     1 KB
2^16    65,536           ~65 thousand    64 KB
2^20    1,048,576        ~1 million      1 MB
2^30    1,073,741,824    ~1 billion      1 GB
2^32    4,294,967,296    ~4 billion      4 GB
2^40    1,099,511,627,776 ~1 trillion    1 TB
2^50    ~1.13 × 10^15    ~1 quadrillion  1 PB
```

### Quick Estimation Rules
```
1 byte = 8 bits
1 KB = 2^10 bytes ≈ 1,000 bytes
1 MB = 2^20 bytes ≈ 1,000 KB ≈ 1,000,000 bytes
1 GB = 2^30 bytes ≈ 1,000 MB ≈ 1,000,000,000 bytes
1 TB = 2^40 bytes ≈ 1,000 GB ≈ 10^12 bytes
1 PB = 2^50 bytes ≈ 1,000 TB ≈ 10^15 bytes
```

---

## Time Conversions

### Seconds in Common Periods
```
Period                  Seconds         Approximation
─────────────────────────────────────────────────────
1 minute                60              ~60
1 hour                  3,600           ~3,600
1 day                   86,400          ~100,000 (10^5)
1 week                  604,800         ~600,000
1 month (30 days)       2,592,000       ~2.5 million (2.5 × 10^6)
1 year (365 days)       31,536,000      ~30 million (3 × 10^7)
```

### Quick Calculations
```
Requests per second to daily:
  100 RPS × 86,400 sec/day = 8.64 million requests/day
  1,000 RPS = 86.4 million requests/day
  10,000 RPS = 864 million requests/day

Daily to per second:
  10 million requests/day ÷ 86,400 = ~116 RPS
  100 million requests/day = ~1,157 RPS
  1 billion requests/day = ~11,574 RPS
```

### Cache TTL Recommendations
```
Data Type                   Recommended TTL
───────────────────────────────────────────
Static assets               1 year (31,536,000 sec)
User sessions               30 minutes (1,800 sec)
API responses (frequent)    1-5 minutes (60-300 sec)
Database query results      5-30 minutes (300-1,800 sec)
User profile data           1 hour (3,600 sec)
Product catalog             1 hour - 1 day
DNS records                 1-24 hours
```

---

## Practical Estimation Examples

### Example 1: Twitter-like Service

**Requirements:**
- 100 million daily active users (DAU)
- Each user views 50 tweets/day
- Each user posts 2 tweets/day
- Average tweet size: 500 bytes

**Read Operations:**
```
Reads/day = 100M users × 50 tweets = 5 billion reads/day
Reads/sec = 5B ÷ 86,400 ≈ 58,000 QPS
Peak (3x average) ≈ 174,000 QPS
```

**Write Operations:**
```
Writes/day = 100M users × 2 tweets = 200 million writes/day
Writes/sec = 200M ÷ 86,400 ≈ 2,300 QPS
Peak (3x average) ≈ 6,900 QPS
```

**Storage:**
```
New tweets/day = 200M × 500 bytes = 100 GB/day
Storage/year = 100 GB × 365 = 36.5 TB/year
With replication (3x) = 109.5 TB/year
```

### Example 2: Video Streaming Service

**Requirements:**
- 50 million DAU
- Average watch time: 1 hour/day
- Average video quality: 5 Mbps (1080p)

**Bandwidth:**
```
Concurrent users (10% of DAU) = 5 million
Total bandwidth = 5M users × 5 Mbps = 25,000 Gbps = 25 Tbps
With CDN caching (80% hit rate):
  Origin bandwidth = 25 Tbps × 20% = 5 Tbps
```

**Storage:**
```
Total watch hours/day = 50M users × 1 hour = 50M hours
If 20% are unique videos:
  Unique hours = 10M hours
Storage/hour (multi-bitrate) = 2 GB
New storage/day = 10M × 2 GB = 20 PB/day (unrealistic without dedup)

More realistic with 1% unique content:
  Unique hours = 500K hours
  New storage/day = 500K × 2 GB = 1 PB/day
```

### Example 3: Instagram-like Service

**Requirements:**
- 200 million DAU
- 20% of users upload 1 photo/day
- Average photo size: 2 MB
- Photos stored in 3 sizes (original, medium, thumbnail)

**Uploads:**
```
Photos/day = 200M × 20% × 1 = 40 million photos/day
Uploads/sec = 40M ÷ 86,400 ≈ 463 uploads/sec
Peak (5x average) ≈ 2,315 uploads/sec
```

**Storage:**
```
Storage/photo = 2 MB + 500 KB + 50 KB ≈ 2.5 MB
Storage/day = 40M photos × 2.5 MB = 100 TB/day
Storage/year = 100 TB × 365 = 36.5 PB/year
With replication (3x) = 109.5 PB/year
```

**Bandwidth:**
```
Views/day = 200M users × 100 photos viewed = 20B views
Assuming medium size (500 KB):
  Download/day = 20B × 500 KB = 10 PB/day
  Bandwidth = 10 PB ÷ 86,400 sec ≈ 116 GB/sec ≈ 928 Gbps
```

---

## Key Principles for Estimation

### Rule of 72
Approximate time for exponential growth:
```
Years to double ≈ 72 ÷ growth rate (%)

Example:
  At 10% annual growth: 72 ÷ 10 = 7.2 years to double
  At 100% annual growth: 72 ÷ 100 = 0.72 years (≈9 months) to double
```

### Capacity Planning Buffer
```
Always plan for headroom:
- CPU: Run at < 70% average utilization
- Memory: Keep 20-30% free
- Disk: Keep 20-30% free
- Network: Plan for 3-5x average load for peaks
```

### Peak vs Average Load
```
Different services have different peak patterns:
- Social media: 3-5x average during peak hours
- E-commerce: 10-20x average during sales/holidays
- Video streaming: 5-10x average during evening hours
- Gaming: 3-5x average during weekends
```

---

## References

These numbers are based on:
- Jeff Dean's "Numbers Everyone Should Know" (Google)
- Production metrics from major tech companies
- AWS, GCP, and Azure pricing and performance documentation
- Open-source benchmark results
- Industry standard practices

**Note:** These are approximate values for estimation purposes. Actual performance varies based on:
- Hardware specifications
- Software configuration
- Network conditions
- Workload patterns
- Optimization techniques

Always benchmark your specific use case for production planning.

---

**Related Modules:**
- [Module 1: Back-of-Envelope Calculations](../01-back-of-envelope/)
- [Module 3: Scalability](../03-scalability/)
- [Module 5: Caching](../05-caching/)
- [Module 6: Database Design](../06-database-design/)
