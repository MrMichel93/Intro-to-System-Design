# Content Delivery Network (CDN)

## What is a CDN?

A **Content Delivery Network (CDN)** is a network of servers distributed globally that deliver content to users from the nearest server location. Think of it like having multiple local stores instead of one central warehouse - users get their products faster!

**Without CDN:**
```
User in Japan → Request → Server in USA (far away) → Slow (500ms+)
```

**With CDN:**
```
User in Japan → Request → CDN Server in Tokyo (nearby) → Fast (50ms)
```

## Why Use a CDN?

### 1. Speed
- Content served from nearest location
- Reduces latency significantly
- Better user experience

### 2. Scalability
- Handles traffic spikes
- Distributes load across many servers
- Your origin server isn't overwhelmed

### 3. Reliability
- If one server fails, others take over
- Content still available
- Improved uptime

### 4. Bandwidth Savings
- CDN serves content, not your server
- Reduces bandwidth costs
- Less load on origin server

### 5. Security
- DDoS protection
- SSL/TLS termination
- Web Application Firewall (WAF)

## How CDNs Work

### Basic Flow:

1. **User requests content**: `GET https://example.com/image.jpg`
2. **DNS routes to nearest CDN server**: Based on user's location
3. **CDN checks cache**:
   - **Cache HIT**: Return cached content immediately
   - **Cache MISS**: Fetch from origin server, cache it, return to user
4. **Future requests**: Served from cache (fast!)

### Architecture:

```
                    [Origin Server]
                           ↓
                    (Content Source)
                           ↓
            ┌──────────────┼──────────────┐
            ↓              ↓              ↓
    [CDN Edge Server] [CDN Edge]   [CDN Edge]
       Tokyo            London        New York
            ↓              ↓              ↓
      Users in Asia   Users in EU   Users in US
```

## Types of Content Served by CDN

### Static Content (Perfect for CDN)
- Images, CSS, JavaScript
- Videos, PDFs
- Fonts, icons
- Downloads

**Example:**
```html
<!-- Instead of loading from your server -->
<img src="https://yourserver.com/logo.png">

<!-- Load from CDN -->
<img src="https://cdn.yoursite.com/logo.png">
```

### Dynamic Content (Can be cached with rules)
- API responses
- Personalized content (with edge computing)
- Real-time data (short cache time)

## CDN Caching Strategies

### 1. Cache-Control Headers

**Long cache (1 year) for static files:**
```
Cache-Control: public, max-age=31536000, immutable
```

**Short cache (5 minutes) for dynamic content:**
```
Cache-Control: public, max-age=300
```

**No cache for sensitive data:**
```
Cache-Control: no-cache, no-store, must-revalidate
```

### 2. Cache Keys
CDN decides what to cache based on:
- URL: `example.com/image.jpg`
- Query strings: `image.jpg?size=large`
- Headers: User-Agent, Accept-Language
- Cookies (for personalized content)

### 3. Cache Invalidation
When content changes, purge CDN cache:

**Methods:**
1. **Purge by URL**: Remove specific file
2. **Purge by tag**: Remove group of related files
3. **Purge all**: Clear entire cache (use sparingly)

**Example:**
```bash
# Purge single file
curl -X PURGE https://cdn.example.com/old-logo.png

# Using CDN API
cdn.purge(['/images/logo.png', '/css/style.css'])
```

## Popular CDN Providers

### 1. Cloudflare
- Free tier available
- DDoS protection included
- Easy setup

### 2. AWS CloudFront
- Integrated with AWS services
- Pay as you go
- Global edge locations

### 3. Fastly
- Real-time purging
- Edge computing (VCL scripting)
- Good for dynamic content

### 4. Akamai
- Enterprise solution
- Largest CDN network
- Expensive but powerful

### 5. Azure CDN
- Integrated with Azure
- Multiple pricing tiers

## CDN Best Practices

### 1. Use Versioning for Static Assets
```html
<!-- Bad: Hard to update -->
<link href="/style.css">

<!-- Good: Easy cache busting -->
<link href="/style.css?v=2.0.1">
<link href="/style-abc123.css"> (hash in filename)
```

### 2. Set Appropriate Cache Times
- **Images, fonts**: 1 year
- **CSS, JS**: 1 week to 1 year (with versioning)
- **HTML**: 5-15 minutes or no-cache
- **API responses**: 1-5 minutes

### 3. Compress Content
```
Content-Encoding: gzip
```
- Reduce file sizes by 60-80%
- CDN can compress automatically

### 4. Use HTTP/2 or HTTP/3
- Multiplexing (multiple files over one connection)
- Faster loading
- Most CDNs support automatically

### 5. Enable HTTPS
- CDN handles SSL/TLS
- Faster than doing it yourself
- Better security

## Edge Computing

Modern CDNs can run code at edge locations!

**Use Cases:**
- A/B testing
- Personalization
- Authentication
- Request routing
- Header manipulation

**Example (Cloudflare Workers):**
```javascript
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  // Run code at the edge
  const country = request.cf.country
  
  if (country === 'US') {
    return fetch('https://us.example.com' + request.url)
  } else {
    return fetch('https://eu.example.com' + request.url)
  }
}
```

## Real-World Examples

### Netflix
- Uses AWS CloudFront and own CDN
- Caches movies at ISP locations
- Reduces bandwidth costs by billions

### Facebook
- Own global CDN network
- Serves billions of images daily
- Edge computing for personalization

### YouTube
- Massive CDN infrastructure
- Caches popular videos worldwide
- Adaptive streaming

## CDN Monitoring

**Key Metrics:**
1. **Cache Hit Ratio**: Percentage of requests served from cache
   - Good: 80%+
   - Excellent: 95%+
2. **Origin Traffic**: Requests going to origin server (should be low)
3. **Response Time**: Latency at edge locations
4. **Bandwidth Usage**: Data transferred
5. **Error Rates**: 4xx, 5xx errors

## Summary

- CDN is a network of servers that cache and deliver content globally
- Serves content from nearest location to users
- Benefits: Speed, scalability, reliability, cost savings
- Perfect for static content (images, CSS, JS, videos)
- Use cache headers to control caching behavior
- Popular providers: Cloudflare, AWS CloudFront, Fastly
- Monitor cache hit ratio (aim for 80%+)
- Modern CDNs support edge computing

## Next Steps

Complete the challenges to master CDN concepts!

**Congratulations!** You've completed the System Design fundamentals course. Review all topics and practice designing real-world systems!

## Implementation Approaches

### Approach 1: Third-Party CDN Provider
- **Description**: Use established CDN services (Cloudflare, AWS CloudFront, Fastly, Akamai).
- **Pros**: Quick setup, global presence immediately, managed infrastructure, built-in DDoS protection, expert support, minimal maintenance.
- **Cons**: Ongoing costs, less control, potential vendor lock-in, limited customization, egress costs can add up.
- **When to use**: Most applications, startups, don't want to manage CDN infrastructure, need global reach quickly, standard caching needs.

### Approach 2: Multi-CDN Strategy
- **Description**: Use multiple CDN providers with intelligent routing between them.
- **Pros**: Better reliability (failover), performance optimization (choose fastest), negotiate better pricing, avoid vendor lock-in, regional optimization.
- **Cons**: Complex management, higher costs, routing logic complexity, cache fragmentation, more monitoring needed.
- **When to use**: Enterprise applications, critical services needing maximum uptime, global presence with regional differences, very high traffic justifying cost.

### Approach 3: Private CDN
- **Description**: Build your own CDN with servers in data centers globally.
- **Pros**: Full control, no ongoing CDN fees, custom features, better for very high scale, data sovereignty control.
- **Cons**: Huge upfront investment, operational complexity, need expert team, slow to deploy globally, bandwidth costs.
- **When to use**: Massive scale (Netflix-level), specific compliance needs, unique requirements, very long-term planning, have expert team.

### Approach 4: Hybrid Approach
- **Description**: Combination of CDN for static assets and origin servers with caching for dynamic content.
- **Pros**: Cost-effective, flexibility, optimized for different content types, gradual adoption, reduced CDN costs.
- **Cons**: More complex architecture, split brain for caching, consistency challenges, additional configuration.
- **When to use**: Mixed content types, cost optimization important, specific performance requirements for different content.

## Trade-offs

| Aspect | With CDN | Without CDN |
|--------|----------|-------------|
| Latency | 20-100ms | 200-1000ms |
| Origin Load | <10% | 100% |
| Bandwidth Cost | Low (CDN handles) | High (origin egress) |
| Scalability | Excellent | Limited by origin |
| Setup Complexity | Moderate | Low |
| Ongoing Cost | Medium-High | Low |
| Geographic Performance | Consistent | Varies greatly |
| DDoS Protection | Built-in | Need separate solution |

| Aspect | Edge Caching | Origin Shield |
|--------|--------------|---------------|
| Cache Efficiency | Good | Excellent |
| Origin Protection | Moderate | Maximum |
| Cost | Lower | Higher |
| Complexity | Low | Moderate |
| Best For | Standard apps | High traffic apps |

## Capacity Calculations

### CDN Bandwidth Requirements
```
Website traffic: 100,000 users/day
Average page size: 2 MB
Pages per user: 10

Daily bandwidth:
100,000 users × 10 pages × 2 MB = 2,000,000 MB = 2 TB/day

With 80% cache hit rate:
Origin bandwidth: 2 TB × 0.2 = 400 GB/day
CDN bandwidth: 2 TB (handled by CDN)

Monthly bandwidth:
2 TB/day × 30 days = 60 TB/month
```

### Cost Estimation
```
CDN provider (e.g., CloudFront):
First 10 TB: $0.085/GB = $850
Next 40 TB: $0.080/GB = $3,200
Next 10 TB: $0.060/GB = $600
Total: ~$4,650/month

Without CDN (origin bandwidth):
60 TB × $0.09/GB = $5,400/month
(Plus server costs to handle load)

Savings: Origin load reduced by 80%
Actual origin: 12 TB × $0.09 = $1,080
CDN cost: $4,650
Total: $5,730 vs $5,400 origin only
But: Better performance, scalability, DDoS protection
```

### Cache Hit Ratio Impact
```
Origin capacity: 1 Gbps
Total traffic: 10 Gbps

Cache hit ratio: 90%
Origin traffic: 10 Gbps × 0.1 = 1 Gbps ✓ (within capacity)

Cache hit ratio: 70%
Origin traffic: 10 Gbps × 0.3 = 3 Gbps ✗ (exceeds capacity)

Goal: Maintain 85%+ cache hit ratio
```

## Common Patterns

**Static Asset CDN**: All images, CSS, JS, fonts served from CDN. Origin serves only HTML and APIs. Most common and effective pattern.

**Cache Everything Strategy**: Cache both static and dynamic content with appropriate TTLs. Use cache tags for selective invalidation. Maximizes CDN benefit.

**Origin Shield**: Add additional caching layer between edge and origin. Collapses requests from multiple edge locations into single origin request. Protects origin from spikes.

**Geo-Based Content Delivery**: Serve region-specific content from CDN. User in EU gets EU pricing, US gets US pricing. CDN edge functions handle routing.

**Versioned Assets**: Include version/hash in filenames (app.v2.3.1.js or app.abc123.js). Enables aggressive caching (1 year) without stale content concerns.

**Purge on Deploy**: Automated cache invalidation on new deployments. Webhook triggers purge for specific assets. Ensures users get latest version quickly.

## Anti-Patterns (What NOT to Do)

**Caching Without Versioning**: Serving assets without version identifiers makes cache invalidation difficult. Users stuck with old JS/CSS causing bugs.

**Short TTLs for Static Assets**: Setting 5-minute cache on images that never change. Defeats CDN purpose and increases costs. Use long TTLs with versioning instead.

**No Cache-Control Headers**: Letting CDN guess what to cache. Results in suboptimal caching. Always set explicit Cache-Control headers.

**Purging Entire Cache**: Invalidating all CDN cache on every deploy. Expensive and defeats CDN purpose. Use targeted purging or versioning.

**Ignoring Cache Hit Ratio**: Not monitoring CDN effectiveness. Could be paying for CDN that's not helping. Track and optimize cache hit ratio.

**CDN for Everything**: Trying to cache highly dynamic, personalized content. Authentication-required content shouldn't be in CDN. Keep sensitive/user-specific at origin.

**No Origin Rate Limiting**: Assuming CDN protects completely from origin overload during cache miss storms. Configure origin rate limits and timeouts.

## Interview Tips

**Explain CDN Purpose**: CDN reduces latency by serving content from locations near users. Don't assume interviewer knows basics.

**Discuss Cache Strategy**: "Static assets cached for 1 year with versioned filenames. HTML cached for 5 minutes. API responses cached per endpoint requirements."

**Calculate Impact**: "With 1M users and 2MB average page, that's 2TB/day. 80% cache hit saves 1.6TB origin bandwidth."

**Address Invalidation**: "Use versioned asset names for long-term caching. For cache busting, either version URLs or targeted purge API calls."

**Security Considerations**: "CDN provides DDoS protection at edge. SSL/TLS termination at CDN reduces origin load. WAF rules can be applied at CDN layer."

**Performance Metrics**: "Monitor cache hit ratio (target 85%+), origin offload percentage, p95/p99 latency by region, error rates."

**Cost Optimization**: "Compress assets, use WebP for images, serve only necessary quality. Monitor bandwidth by content type to identify optimization opportunities."

**Real Examples**: Reference how Netflix uses custom CDN, how Cloudflare handles millions of sites, how large sites achieve 95%+ cache hits.

## See It In Action

Complete the CDN design challenge: **[CDN Challenges](./challenges.md)**

Design a global CDN strategy for a media-heavy website serving 10M users across 6 continents.

## Additional Resources

**Documentation**:
- [AWS CloudFront Documentation](https://docs.aws.amazon.com/cloudfront/)
- [Cloudflare CDN Documentation](https://developers.cloudflare.com/cache/)
- [Fastly CDN Documentation](https://docs.fastly.com/)

**Articles**:
- [How CDNs Work - High Scalability](http://highscalability.com/blog/2011/2/28/a-practical-guide-to-varnish-why-varnish-matters.html)
- [Netflix's CDN - Open Connect](https://openconnect.netflix.com/en/)
- [Cloudflare: How We Built It](https://blog.cloudflare.com/)

**Tools**:
- **GTmetrix**: Test page load times from different locations
- **WebPageTest**: Detailed performance analysis with CDN metrics  
- **cdnperf.com**: CDN performance comparison tool
- **curl with timing**: Test CDN response times manually

**Books**:
- "High Performance Browser Networking" by Ilya Grigorik (O'Reilly)
