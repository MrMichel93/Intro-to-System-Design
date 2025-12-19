# CDN - Challenges & Questions

## 🎯 Multiple Choice Questions

### Question 1
What is the main purpose of a CDN?

A) Store backups of your website  
B) Deliver content from servers near users for faster loading  
C) Increase security only  
D) Host your database  

<details>
<summary>Answer</summary>

**B) Deliver content from servers near users for faster loading**

CDN's primary purpose is to reduce latency by serving content from geographically closer servers. This significantly improves load times and user experience.
</details>

---

### Question 2
Which type of content is BEST suited for CDN caching?

A) User's bank account balance  
B) Static images and CSS files  
C) Live chat messages  
D) User's current location  

<details>
<summary>Answer</summary>

**B) Static images and CSS files**

Static content that doesn't change frequently is perfect for CDN caching. Dynamic, user-specific, or real-time data shouldn't be heavily cached.
</details>

---

### Question 3
What is cache invalidation?

A) Deleting all files from CDN  
B) Removing specific cached content when it's updated  
C) Disabling the CDN  
D) Blocking users from accessing cache  

<details>
<summary>Answer</summary>

**B) Removing specific cached content when it's updated**

Cache invalidation (or purging) removes outdated content from CDN caches so the CDN fetches fresh content from the origin server.
</details>

---

### Question 4
What is a good cache hit ratio?

A) 10%  
B) 50%  
C) 80%+  
D) It doesn't matter  

<details>
<summary>Answer</summary>

**C) 80%+**

A cache hit ratio of 80% or higher means most requests are served from cache, which is efficient. Below 80% means too many origin server requests.
</details>

---

## 🧩 Scenario-Based Challenges

### Challenge 1: Configure CDN for Blog

You have a blog with:
- HTML pages (updated daily)
- Images (never change)
- CSS/JS (updates weekly)
- User avatars (uploaded by users)

**Task:** Define cache strategies for each content type.

<details>
<summary>Sample Solution</summary>

**Cache Strategy:**

```
1. HTML Pages:
   Cache-Control: public, max-age=900, s-maxage=900
   (15 minutes - balance freshness and performance)
   
2. Images (blog content):
   Cache-Control: public, max-age=31536000, immutable
   (1 year - use versioned URLs like image-v2.jpg)
   
3. CSS/JS:
   Cache-Control: public, max-age=604800
   (1 week, with version in filename: style-v1.2.css)
   
4. User Avatars:
   Cache-Control: public, max-age=86400
   (1 day - users may update avatars)
```

**CDN Configuration:**
```javascript
// Cloudflare Page Rule example
{
  "url": "*.jpg",
  "settings": {
    "cache_level": "cache_everything",
    "edge_cache_ttl": 31536000
  }
}
```

**File Versioning:**
```html
<!-- Bad: Hard to update -->
<link href="/style.css">
<img src="/logo.png">

<!-- Good: Cache busting -->
<link href="/style.css?v=1.2.3">
<img src="/logo-abc123.png">
```
</details>

---

### Challenge 2: Reduce Bandwidth Costs

Your site serves 10TB of images monthly. CDN costs $0.08/GB.

**Questions:**
1. Calculate monthly CDN cost
2. How can you reduce it?
3. What cache hit ratio target should you set?

<details>
<summary>Sample Solution</summary>

**1. Current Cost:**
```
10 TB = 10,000 GB
Cost = 10,000 GB × $0.08 = $800/month
```

**2. Reduction Strategies:**

**A. Optimize Images:**
- Convert to WebP (60-80% smaller)
- Compress with tools (ImageOptim, TinyPNG)
- Serve responsive images (different sizes for mobile/desktop)

```html
<picture>
  <source srcset="image-small.webp" media="(max-width: 600px)">
  <source srcset="image-large.webp" media="(min-width: 601px)">
  <img src="image.jpg" alt="Fallback">
</picture>
```

**Savings:** 70% reduction = 3TB served
**New cost:** 3TB × 1000 GB/TB × $0.08/GB = $240/month
**Savings:** $560/month

**B. Increase Cache Hit Ratio:**
- Set longer cache times
- Use proper cache headers
- Pre-warm cache for popular content

**Target:** 95% cache hit ratio
- 95% served from cache (CDN internal, often free or cheaper)
- 5% from origin = 500 GB origin traffic

**C. Lazy Loading:**
```html
<img src="image.jpg" loading="lazy">
```
- Only load images when user scrolls to them
- Reduces unnecessary loads by ~30%

**3. Cache Hit Ratio Target:**
- **Current:** 80% = $800/month
- **Target:** 95% = Significant savings
  - Less origin bandwidth
  - Lower CDN egress costs
  - Better performance

**Total Potential Savings:**
- Image optimization: -70% data
- Better caching: -15% origin traffic  
- Lazy loading: -30% unnecessary loads
- **Combined:** ~$200-300/month (vs $800)
</details>

---

### Challenge 3: CDN for Global Video Platform

Design CDN strategy for video streaming service:
- 1 million users worldwide
- Videos: 100MB-5GB each
- Users expect < 2 second start time
- Some videos very popular, others rarely watched

<details>
<summary>Sample Solution</summary>

**CDN Strategy:**

**1. Multi-Tier Caching:**
```
[Origin Storage] → [Regional CDN] → [Edge CDN] → [Users]
    (S3/GCS)        (Major cities)    (Worldwide)
```

**2. Adaptive Bitrate Streaming:**
```
Video available in multiple qualities:
- 4K (10 Mbps)
- 1080p (5 Mbps)
- 720p (2.5 Mbps)
- 480p (1 Mbps)
- 360p (500 Kbps)

User gets best quality for their connection
```

**3. Segment-Based Caching:**
```
Instead of caching entire 2GB video:
- Split into 10-second segments
- Cache popular segments at edge
- Fetch rare segments from regional cache

video-segment-001.ts
video-segment-002.ts
...
```

**4. Cache Strategy by Popularity:**
```
Popular videos (80% of views):
- Pre-warm cache at all edge locations
- Cache-Control: max-age=86400 (1 day)

Medium popularity:
- Cache on-demand at edge
- Keep in regional cache
- Cache-Control: max-age=21600 (6 hours)

Rare videos (long tail):
- Fetch from origin as needed
- Don't cache at edge
- Cache in regional only
```

**5. Smart Routing:**
```javascript
// Edge logic to route to best origin
if (isPopularVideo(videoId)) {
  return fetchFromEdge(videoId)
} else if (isRegionalPopular(videoId, userRegion)) {
  return fetchFromRegional(videoId, userRegion)
} else {
  return fetchFromOrigin(videoId)
}
```

**6. Cost Optimization:**
```
Storage costs:
- Origin: All videos (cold storage)
- Regional CDN: Top 20% videos
- Edge CDN: Top 5% videos

Expected results:
- 95% of requests from edge/regional
- < 2 second start time (buffering first segments)
- Lower bandwidth costs (intelligent caching)
```

**7. Metrics to Monitor:**
```
- Cache hit ratio by region (target: 90%+)
- Start time (target: < 2 seconds)
- Buffering ratio (target: < 1%)
- Bandwidth usage
- Cost per GB delivered
```
</details>

---

## 🤔 Critical Thinking Questions

1. **Why can't you cache everything forever?**
   (Think about updates, storage costs, stale content)

2. **How would you handle a situation where you need to immediately update content that's heavily cached?**
   (Consider cache invalidation, versioning strategies)

3. **What are the trade-offs between a CDN and serving everything from your own servers?**
   (Cost, complexity, performance, control)

---

## 🎉 Congratulations!

You've completed the System Design course! You now understand:
- ✅ Scalability
- ✅ Load Balancing
- ✅ Caching
- ✅ Database Design
- ✅ API Design
- ✅ Microservices
- ✅ Message Queues
- ✅ CDN

**Next Steps:**
1. Review all topics
2. Practice designing systems (Twitter, Uber, Netflix)
3. Build a project using these concepts
4. Keep learning and stay curious!
