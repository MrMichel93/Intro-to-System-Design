# Design Challenge 5: Web Crawler - Sample Solution

## 1. Capacity Estimation

### Target: 1 billion pages per month

```
Pages per day: 1B / 30 = 33.3 million
Pages per second: 33.3M / 86,400 ≈ 385 pages/sec
Peak (3x): ~1,200 pages/sec
```

### Storage Requirements:
```
Average page size: 100KB (HTML)
Metadata per page: 500 bytes
Total per page: ~100.5 KB

Monthly storage: 1B × 100.5KB = 100.5 TB
5-year storage: 100.5 TB × 12 × 5 = 6 PB
With compression (3:1): ~2 PB
```

### Bandwidth:
```
Download: 385 pages/sec × 100KB = 38.5 MB/s
With metadata, DNS lookups: ~50 MB/s
Upload (storage): Similar, ~50 MB/s
```

### Crawler Machines:
```
Assume each crawler handles 10 pages/sec:
Required: 385 / 10 = 39 crawler machines
With redundancy: 50 machines
```

---

## 2. URL Frontier Design

### Multi-Queue Architecture:

```
┌─────────────────────────────────────────────────┐
│              URL Frontier                        │
├─────────────────────────────────────────────────┤
│                                                  │
│  Prioritizer                                     │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │Priority │  │Priority │  │Priority │         │
│  │ Queue 1 │  │ Queue 2 │  │ Queue 3 │         │
│  │ (High)  │  │ (Medium)│  │  (Low)  │         │
│  └────┬────┘  └────┬────┘  └────┬────┘         │
│       │            │            │               │
│       └────────────┴────────────┘               │
│                    │                            │
│       Front Queue Selector                      │
│                    │                            │
├────────────────────┼────────────────────────────┤
│                    │                            │
│  Politeness Enforcer                            │
│  ┌──────────────────────────────────────┐      │
│  │ Domain Queue Map                     │      │
│  │ example.com → Queue A (ready: 1s)    │      │
│  │ google.com  → Queue B (ready: 0.5s)  │      │
│  │ github.com  → Queue C (ready: 2s)    │      │
│  └──────────────────────────────────────┘      │
│                                                  │
└──────────────────────────────────────────────┬──┘
                                                 │
                                          ┌──────▼──────┐
                                          │  Crawlers   │
                                          └─────────────┘
```

### Priority Calculation:

```python
def calculate_priority(url, page_rank, last_crawl_time):
    # Factors:
    freshness = time.now() - last_crawl_time
    importance = page_rank  # PageRank or similar
    
    # Priority = importance × age_factor
    priority = importance * (1 + freshness / (30 * 86400))  # 30-day normalization
    return priority
```

### Politeness Implementation:

**Per-Domain Queues:**
```python
class PolitenessManager:
    def __init__(self):
        self.domain_queues = {}  # domain → queue
        self.last_access = {}    # domain → timestamp
        self.crawl_delay = {}    # domain → delay_seconds
    
    def get_next_url(self):
        # Find domain that's ready to be crawled
        for domain, queue in self.domain_queues.items():
            if not queue.empty():
                elapsed = time.now() - self.last_access[domain]
                delay = self.crawl_delay.get(domain, 1.0)  # Default 1 second
                
                if elapsed >= delay:
                    url = queue.get()
                    self.last_access[domain] = time.now()
                    return url
        return None
```

**Robots.txt Handling:**
```python
class RobotsTxtCache:
    def __init__(self):
        self.cache = {}  # domain → robots.txt rules
    
    def can_crawl(self, url):
        domain = extract_domain(url)
        if domain not in self.cache:
            self.cache[domain] = fetch_robots_txt(domain)
        
        rules = self.cache[domain]
        return rules.is_allowed(url)
```

---

## 3. System Architecture

```
┌────────────────┐
│  Seed URLs     │
└───────┬────────┘
        │
        ▼
┌────────────────────────────────────────────────────┐
│            URL Frontier (Distributed Queue)        │
│     (Kafka / RabbitMQ / Custom Queue System)       │
└───────┬────────────────────────────────────────────┘
        │
    ┌───┴────┬────────┬────────┐
    │        │        │        │
┌───▼───┐ ┌─▼────┐ ┌─▼────┐ ┌─▼────┐
│Crawler│ │Crawler│ │Crawler│ │Crawler│
│ Node 1│ │ Node 2│ │ Node 3│ │ Node N│
└───┬───┘ └──┬───┘ └──┬───┘ └──┬───┘
    │        │        │        │
    └────────┴────┬───┴────────┘
                  │
    ┌─────────────┼──────────────┐
    │             │              │
┌───▼───┐  ┌─────▼──────┐  ┌───▼────────┐
│ URL   │  │  Content   │  │   Bloom    │
│Dedupe │  │  Storage   │  │   Filter   │
│Service│  │(HDFS/S3)   │  │  (Redis)   │
└───────┘  └────────────┘  └────────────┘
```

### Core Components:

**1. URL Frontier:**
- Distributed queue system (Kafka or custom)
- Handles prioritization and politeness
- Partitioned by domain for load distribution
- Persistent (survives restarts)

**2. Crawler Nodes:**
```python
class CrawlerNode:
    def run(self):
        while True:
            # Get next URL respecting politeness
            url = url_frontier.get_next()
            if not url:
                sleep(1)
                continue
            
            # Check if already crawled
            if url_dedupe.is_seen(url):
                continue
            
            # Fetch page
            try:
                response = http_client.get(url, timeout=10)
                
                # Store content
                content_storage.save(url, response.html, response.metadata)
                
                # Extract links
                links = extract_links(response.html, url)
                
                # Add new URLs to frontier
                for link in links:
                    if not url_dedupe.is_seen(link):
                        url_frontier.add(link, calculate_priority(link))
                        url_dedupe.mark_seen(link)
            
            except Exception as e:
                # Log error, potentially retry
                handle_error(url, e)
```

**3. URL Deduplication Service:**
```python
# Using Bloom Filter + Database

class URLDedupeService:
    def __init__(self):
        self.bloom_filter = BloomFilter(expected_urls=10_000_000_000, 
                                        false_positive_rate=0.001)
        self.url_database = URLDatabase()
    
    def is_seen(self, url):
        normalized = normalize_url(url)
        
        # Quick check with Bloom filter
        if not self.bloom_filter.might_contain(normalized):
            return False
        
        # Confirm with database (to avoid false positives)
        return self.url_database.exists(normalized)
    
    def mark_seen(self, url):
        normalized = normalize_url(url)
        self.bloom_filter.add(normalized)
        self.url_database.insert(normalized, timestamp=time.now())
```

**4. Content Storage:**
```
Storage Strategy:
- Raw HTML: HDFS or S3 (distributed file storage)
- Metadata: NoSQL database (Cassandra, HBase)
  - url, domain, crawl_time, content_hash, status_code
- Indexed data: Elasticsearch (for search if needed)
```

---

## 4. Handling Challenges

### 1. Duplicate URLs

**URL Normalization:**
```python
def normalize_url(url):
    # Parse URL
    parsed = urlparse(url)
    
    # Convert to lowercase
    domain = parsed.netloc.lower()
    path = parsed.path.lower()
    
    # Remove trailing slash
    path = path.rstrip('/')
    
    # Sort query parameters
    params = sorted(parse_qs(parsed.query).items())
    
    # Remove fragments
    # Remove session IDs, tracking parameters
    
    # Reconstruct
    return f"{parsed.scheme}://{domain}{path}?{urlencode(params)}"
```

**Bloom Filter + Database:**
- Bloom filter: Fast negative checks (99%+ of URLs)
- Database: Confirm positives, store metadata
- Trade-off: Small false positive rate acceptable

### 2. URL Traps (Spider Traps)

**Problem:** Infinite loops, calendar pages, dynamic URLs

**Solutions:**
```python
# Max depth per domain
MAX_DEPTH_PER_DOMAIN = 20

# Max URLs per domain
MAX_URLS_PER_DOMAIN = 10000

# URL pattern detection
def is_trap(url):
    # Detect calendar patterns
    if re.match(r'.*/\d{4}/\d{2}/\d{2}/.*', url):
        return True
    
    # Detect excessive query parameters
    if url.count('?') > 1 or url.count('&') > 10:
        return True
    
    # Detect repeated path segments
    segments = urlparse(url).path.split('/')
    if len(segments) != len(set(segments)):
        return True
    
    return False

# Domain crawl budget
domain_url_count[domain] += 1
if domain_url_count[domain] > MAX_URLS_PER_DOMAIN:
    skip_domain(domain)
```

### 3. Failed Requests

**Retry Strategy:**
```python
class CrawlRequest:
    def __init__(self, url, max_retries=3):
        self.url = url
        self.retries = 0
        self.max_retries = max_retries
        self.backoff = 1  # Initial backoff in seconds
    
    def should_retry(self, error):
        # Retry on transient errors
        transient = [ConnectionError, Timeout, 503, 429]
        
        if type(error) in transient or error.status_code in transient:
            if self.retries < self.max_retries:
                self.retries += 1
                self.backoff *= 2  # Exponential backoff
                return True
        return False
```

**Dead Letter Queue:**
- URLs that fail after max retries go to DLQ
- Manual review or delayed retry (e.g., 24 hours later)

### 4. Load Distribution

**Consistent Hashing for Domain Assignment:**
```python
class DomainRouter:
    def __init__(self, crawler_nodes):
        self.hash_ring = ConsistentHashRing(crawler_nodes)
    
    def get_crawler_for_url(self, url):
        domain = extract_domain(url)
        # Route same domain to same crawler for politeness
        crawler = self.hash_ring.get_node(domain)
        return crawler
```

**Benefits:**
- Same domain always goes to same crawler
- Politeness enforcement is local (no distributed coordination needed)
- Adding/removing crawlers only affects adjacent domains

---

## 5. Additional Optimizations

### DNS Caching:
```python
class DNSCache:
    def __init__(self, ttl=300):
        self.cache = {}
        self.ttl = ttl
    
    def resolve(self, domain):
        if domain in self.cache:
            entry, timestamp = self.cache[domain]
            if time.now() - timestamp < self.ttl:
                return entry
        
        ip = socket.gethostbyname(domain)
        self.cache[domain] = (ip, time.now())
        return ip
```

### Connection Pooling:
- Maintain persistent connections per domain
- Reuse TCP connections (HTTP Keep-Alive)
- Reduces connection overhead

### Priority Refresh:
```python
# High-priority pages recrawled more frequently
def recrawl_interval(url_priority):
    if url_priority > 0.9:  # Very important
        return 1 * 86400  # Daily
    elif url_priority > 0.7:
        return 7 * 86400  # Weekly
    else:
        return 30 * 86400  # Monthly
```

---

## 6. Trade-offs

**Politeness vs Throughput:**
- Strict politeness (1 req/sec per domain) limits throughput
- Solution: Crawl more diverse domains simultaneously

**Freshness vs Coverage:**
- Frequent recrawls use bandwidth/storage
- Solution: Priority-based recrawl intervals

**Deduplication Accuracy vs Speed:**
- Bloom filter: Fast but false positives
- Database: Accurate but slower
- Solution: Two-stage check (Bloom filter → Database)

---

## Summary

This distributed web crawler achieves:
- ✅ 1 billion pages/month (385 pages/sec)
- ✅ Politeness (1 req/sec per domain)
- ✅ Deduplication (Bloom filter + DB)
- ✅ Scalability (horizontal scaling)
- ✅ Resilience (retry logic, DLQ)

The architecture efficiently crawls the web while being respectful to servers and handling edge cases gracefully.
