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
