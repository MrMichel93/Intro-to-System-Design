# Load Balancing

## Overview

Load balancing is the practice of distributing network traffic and computational workload across multiple servers to optimize resource utilization, maximize throughput, minimize response time, and avoid overloading any single server. It acts as an intelligent traffic distributor that sits between clients and servers, making real-time decisions about where to route each request. Load balancing is fundamental to achieving high availability, reliability, and scalability in modern distributed systems.

## The Problem It Solves

**Scenario**: You've deployed your web application on three servers, each capable of handling 100 requests per second. Without load balancing, users must somehow know which server to connect to. In practice, all users end up hitting the same server (often whichever IP address is in DNS), while the other two sit idle. That single server becomes overwhelmed at 100+ concurrent requests, causing timeouts and crashes, even though you have 300 requests/second total capacity sitting unused.

Load balancing solves this by providing a single entry point that intelligently distributes incoming traffic. It monitors server health, routes requests to available servers, removes failed servers from rotation, and enables seamless scaling. Without load balancing, horizontal scaling is nearly impossible—you can add more servers, but you can't effectively use them. Load balancing is the mechanism that makes multiple servers function as a single, highly available system.

## Real-World Examples

**Netflix**: Uses AWS Elastic Load Balancing (ELB) with over 1,000 load balancers handling millions of requests per second globally. Their Zuul gateway provides application-level load balancing, routing requests based on content type (streaming vs. API), user location, and service health. During peak hours, their load balancers automatically distribute traffic across thousands of microservice instances.

**Google Search**: Employs multi-tier load balancing—DNS-based geographic routing sends users to the nearest data center, then Layer 4 load balancers distribute to appropriate clusters, finally Layer 7 load balancers route to specific services. This hierarchy handles 8.5 billion searches per day (over 99,000 per second) with millisecond response times.

**Amazon**: Uses Application Load Balancers (ALB) to route traffic based on URL paths—`/products` goes to product service clusters, `/checkout` to payment services, `/recommendations` to ML-powered recommendation services. During Prime Day 2023, their load balancing infrastructure handled traffic spikes 10x normal levels by dynamically scaling target groups.

**WhatsApp**: Built custom Erlang-based load balancers to maintain persistent WebSocket connections for 2+ billion users. Their load balancing strategy minimizes connection drops during server maintenance by gracefully draining connections before taking servers offline—critical for real-time messaging where connection stability matters most.

## Core Concepts

### 1. Load Balancing Algorithms
Different algorithms suit different use cases. **Round Robin** distributes requests sequentially (Server 1, 2, 3, 1, 2, 3...), simple but ignores server load. **Least Connections** sends traffic to servers with fewest active connections, ideal for varying request processing times. **Weighted algorithms** account for server capacity differences—a 16-core server might receive 4x traffic of a 4-core server. **IP Hash** consistently routes specific users to same servers, useful for caching but can create hot spots. **Least Response Time** considers both connection count and server speed, optimizing for performance but requiring health monitoring overhead.

### 2. Layer 4 vs Layer 7 Load Balancing
**Layer 4 (Transport Layer)** operates at TCP/UDP level, making routing decisions based solely on IP addresses and ports. It's fast (minimal packet inspection), handles any protocol, and operates at line speed. Cannot inspect application data or make content-based decisions. **Layer 7 (Application Layer)** understands HTTP/HTTPS, can route based on URL paths, headers, cookies, or request content. Enables sophisticated routing like sending mobile traffic to mobile-optimized servers or routing API calls to different microservices. Requires SSL termination and more processing but provides application-aware intelligence.

### 3. Health Checks and Failover
Load balancers continuously monitor backend server health through periodic checks. **Active health checks** send requests (ping, TCP connection, HTTP GET) at regular intervals (every 10-30 seconds). **Passive health checks** monitor actual traffic and mark servers unhealthy after consecutive failures. When a server fails checks (typically 3 consecutive failures), it's removed from the pool immediately. Once healthy again (typically 2 consecutive successes), it's gradually added back. This automatic failover provides high availability without manual intervention.

### 4. Session Persistence  
Some applications require requests from the same client to reach the same server (**sticky sessions**). Implemented via cookie-based routing, IP hashing, or dedicated session tables in the load balancer. While convenient, sticky sessions reduce load balancing effectiveness and complicate scaling. Modern architectures prefer **stateless services** with externalized session storage (Redis, database) so any server can handle any request. This enables true horizontal scaling and simplified failure handling.

### 5. SSL/TLS Termination
Load balancers can decrypt HTTPS traffic, inspect it, then forward unencrypted to backend servers (**SSL offloading**). Benefits include centralized certificate management, reduced CPU load on application servers (encryption is expensive), and enables Layer 7 routing. Alternatively, **SSL passthrough** forwards encrypted traffic directly to backends, maintaining end-to-end encryption at cost of losing Layer 7 capabilities. **SSL bridging** decrypts at load balancer, makes routing decisions, then re-encrypts to backends—provides both security and routing intelligence but highest overhead.

### 6. Global Server Load Balancing (GSLB)
For multi-datacenter deployments, GSLB routes users to geographically nearest or best-performing datacenter. Uses DNS-based routing with short TTLs to enable rapid failover. Considers datacenter health, capacity, and latency. If US-East datacenter fails, GSLB redirects traffic to US-West within minutes. Essential for global applications requiring low latency and disaster recovery across regions.

## How Load Balancers Work

```
                [Load Balancer]
                       |
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
   [Server 1]     [Server 2]     [Server 3]
```

**Process:**
1. User sends request to load balancer
2. Load balancer picks a server based on algorithm
3. Server processes request and returns response
4. Load balancer sends response back to user

## Load Balancing Algorithms

### 1. Round Robin
**How it works**: Distribute requests in circular order (1 → 2 → 3 → 1 → 2 → 3...)

**Example:**
- Request 1 → Server A
- Request 2 → Server B
- Request 3 → Server C
- Request 4 → Server A (back to start)

**Pros**: Simple, fair distribution  
**Cons**: Doesn't consider server load or capacity

**Best for**: Servers with similar specifications handling similar tasks

---

### 2. Least Connections
**How it works**: Send traffic to the server with fewest active connections

**Example:**
- Server A: 10 connections
- Server B: 5 connections ← Next request goes here
- Server C: 8 connections

**Pros**: Better for long-running connections  
**Cons**: More complex to track

**Best for**: Chat applications, streaming, database connections

---

### 3. Weighted Round Robin
**How it works**: Assign weights to servers based on capacity

**Example:**
```
Server A (weight: 3) gets 3 requests
Server B (weight: 2) gets 2 requests
Server C (weight: 1) gets 1 request
```

**Pros**: Accounts for different server capacities  
**Cons**: Need to configure weights properly

**Best for**: Mixed server specifications (some powerful, some less powerful)

---

### 4. IP Hash
**How it works**: Use user's IP address to determine which server to use

**Example:**
- User with IP 192.168.1.1 always goes to Server A
- User with IP 192.168.1.2 always goes to Server B

**Pros**: Same user always hits same server (useful for sessions)  
**Cons**: Can lead to uneven distribution

**Best for**: Applications needing session persistence (though better solutions exist)

---

### 5. Least Response Time
**How it works**: Send traffic to server with fastest response time

**Pros**: Optimizes for performance  
**Cons**: Requires health checks and monitoring

**Best for**: Performance-critical applications

## Types of Load Balancers

### 1. Hardware Load Balancers
- Physical devices (expensive)
- High performance
- Used by large enterprises
- Examples: F5, Citrix

### 2. Software Load Balancers
- Run on standard servers
- More flexible and cost-effective
- Examples: Nginx, HAProxy, Apache

### 3. Cloud Load Balancers
- Managed by cloud providers
- Easy to set up and scale
- Examples: AWS ELB, Google Cloud Load Balancing, Azure Load Balancer

## Layer 4 vs Layer 7 Load Balancing

### Layer 4 (Transport Layer)
- Routes based on IP address and TCP/UDP port
- Faster (less processing)
- Can't see application data
- Example: Forward all traffic on port 443 to any server

### Layer 7 (Application Layer)
- Routes based on content (URL, headers, cookies)
- Can make smart decisions
- More processing overhead
- Example: Send `/api/users` requests to User servers, `/api/products` to Product servers

**Analogy:**
- **Layer 4**: Mail sorter who only looks at zip code
- **Layer 7**: Mail sorter who reads full address and content type

## Health Checks

Load balancers need to know which servers are healthy!

**Health Check Process:**
```
Load Balancer → Ping Server → Check Response
                    ↓
            Is Server Healthy?
            /              \
          Yes               No
          ↓                 ↓
    Send Traffic      Stop Sending Traffic
```

**Types of Health Checks:**
1. **Ping Check**: Is the server reachable?
2. **TCP Check**: Can we establish a connection?
3. **HTTP Check**: Does the server respond with 200 OK?
4. **Custom Check**: Run specific application test

**Configuration Example:**
- Check every 30 seconds
- 3 failed checks = mark unhealthy
- 2 successful checks = mark healthy again

## Session Persistence (Sticky Sessions)

**Problem**: User logs in on Server 1, next request goes to Server 2 (doesn't have session data)

**Solutions:**

### 1. Sticky Sessions
Load balancer remembers which server a user used and keeps sending them there
- **Pro**: Simple
- **Con**: Defeats some benefits of load balancing

### 2. Session Store (Better!)
Store sessions in a shared database or cache (Redis)
- **Pro**: Any server can handle any request
- **Con**: Requires architectural change

## SSL/TLS Termination

Load balancers can handle HTTPS encryption/decryption:

```
User (HTTPS) → Load Balancer → Server (HTTP)
```

**Benefits:**
- Offload CPU-intensive SSL processing from application servers
- Centralized certificate management
- Servers can focus on application logic

## Real-World Examples

### Netflix
- Uses AWS Elastic Load Balancer
- Distributes millions of requests per second
- Auto-scales during peak hours

### Facebook
- Custom load balancers for massive scale
- Handles billions of requests daily
- Geographic load balancing (routes to nearest data center)

### E-commerce During Black Friday
- Pre-scale servers before sale
- Load balancer distributes traffic
- Health checks remove crashed servers automatically

## Common Architecture Patterns

### 1. Single Load Balancer
```
[Users] → [Load Balancer] → [Servers]
```
Simple but load balancer is single point of failure

### 2. Load Balancer with Failover
```
[Users] → [Primary LB] → [Servers]
          [Backup LB]  ↗
```
Backup takes over if primary fails

### 3. Multi-Tier Load Balancing
```
[Users] → [External LB] → [Web Servers] → [Internal LB] → [App Servers]
```
Different layers for different purposes

### 4. Geographic Load Balancing
```
                    [Global LB]
                    /          \
            [US LB]             [EU LB]
            /    \              /     \
    [US Servers] [US Servers] [EU Servers] [EU Servers]
```
Route users to nearest data center

## Best Practices

1. **Always Use Health Checks**: Don't send traffic to broken servers
2. **Monitor Everything**: Track request rates, error rates, server health
3. **Plan for Failure**: Have backup load balancers
4. **Use Appropriate Algorithm**: Match algorithm to your use case
5. **Avoid Sticky Sessions**: Use shared session storage instead
6. **Enable SSL Termination**: Simplify certificate management
7. **Test Failover**: Regularly test what happens when servers fail

## How It Works

**Step 1 - Client Request Initiation**: User types URL in browser. DNS resolves to load balancer's IP address. All traffic funnels through this single entry point.

**Step 2 - Load Balancer Receives Request**: Load balancer accepts the incoming TCP connection. Maintains connection state if necessary. Logs request for monitoring.

**Step 3 - Algorithm Selection**: Load balancer applies configured algorithm to select target server from healthy backend pool.

**Step 4 - Health Check Validation**: Before routing, verifies selected server passed recent health checks. If unhealthy, selects alternative server.

**Step 5 - Request Forwarding**: Forwards request to selected backend server. May modify headers (add X-Forwarded-For with client IP).

**Step 6 - Server Processing**: Backend server processes request and generates response. Load balancer waits while potentially handling thousands of other requests.

**Step 7 - Response Return**: Server sends response back to load balancer. Load balancer forwards to original client.

**Step 8 - Connection Management**: For HTTP/1.1, may keep connection alive for subsequent requests. Tracks metrics for capacity planning.

## Visual Architecture

```
             Internet / Users
                   |
           ┌───────▼────────┐
           │  Load Balancer │
           │  - Routes req  │
           │  - Health chk  │
           │  - SSL term    │
           └────┬──┬──┬────┘
                |  |  |
       ┌────────┘  |  └────────┐
       ▼           ▼           ▼
┌───────────┐ ┌─────────┐ ┌─────────┐
│ Server 1  │ │Server 2 │ │Server 3 │
│ Health:✓  │ │Health:✓ │ │Health:✗ │
└───────────┘ └─────────┘ └─────────┘
```

## Implementation Approaches

### Approach 1: Hardware Load Balancer
- **Description**: Dedicated physical appliances optimized for load balancing with specialized ASICs.
- **Pros**: Extremely high performance, advanced features, vendor support.
- **Cons**: Very expensive, vendor lock-in, limited flexibility.
- **When to use**: Enterprise with budget, extreme performance needs, on-premise datacenters.

### Approach 2: Software Load Balancer  
- **Description**: Open-source software (Nginx/HAProxy) running on standard servers.
- **Pros**: Cost-effective, highly flexible, runs anywhere.
- **Cons**: Requires server management, need expertise to configure.
- **When to use**: Most production applications, teams with DevOps capability, cost-conscious.

### Approach 3: Cloud-Native Load Balancers
- **Description**: Managed services from cloud providers (AWS ALB/NLB, GCP LB, Azure LB).
- **Pros**: Zero operational overhead, automatic scaling, pay-per-use.
- **Cons**: Vendor lock-in, can be expensive at scale, less customization.
- **When to use**: Cloud-first applications, startups, want managed infrastructure.

### Approach 4: Service Mesh
- **Description**: Application-layer networking with sidecar proxies (Envoy/Istio).
- **Pros**: Advanced traffic management, observability built-in, microservices-aware.
- **Cons**: Complex operational model, resource overhead, steep learning curve.
- **When to use**: Microservices architecture, Kubernetes environments.

## Trade-offs

| Aspect | Layer 4 LB | Layer 7 LB |
|--------|------------|------------|
| Performance | Very High | Moderate |
| Latency | <1ms | 1-10ms |
| Routing | IP/Port only | URL, headers, content |
| SSL Inspection | No | Yes |
| Use Case | High throughput | Application-aware |

| Aspect | Sticky Sessions | Stateless + Session Store |
|--------|-----------------|--------------------------|
| Load Distribution | Uneven | Even |
| Failure Handling | Lost sessions | No impact |
| Scaling | Difficult | Easy |

## Capacity Calculations

### Load Balancer Sizing
```
Target: 10,000 RPS
Request: 10 KB, Response: 50 KB

Bandwidth:
- Input: 100 MB/s = 800 Mbps
- Output: 500 MB/s = 4 Gbps
- Need 10 Gbps link

Layer 4: 1 instance (handles 10M RPS)
Layer 7: 1 instance (handles 50K RPS)
HA: 2x instances
```

### Backend Pool
```
Request rate: 10,000 RPS
Server capacity: 500 RPS each
Minimum: 20 servers
With 50% buffer: 30 servers
```

## Common Patterns

**Round Robin with Health Checks**: Simplest pattern. Even distribution, automatic failover.

**Least Connections for Long-Polling**: WebSocket/SSE connections need connection-aware routing.

**Weighted Routing for Canary**: Route 95% to stable, 5% to new version for safe deployment.

**Geographic Routing**: DNS-based routing to nearest datacenter for global applications.

## Anti-Patterns (What NOT to Do)

**Sticky Sessions Without Justification**: Reduces scalability. Externalize state to Redis/database instead.

**Single Load Balancer**: No redundancy creates single point of failure. Always deploy pairs.

**No Health Checks**: Basic ping doesn't verify application health. Use HTTP endpoint checks.

**Ignoring Load Balancer Limits**: Connection tables have limits. Monitor before hitting them.

**Over-Complex Routing Logic**: Keep routing simple. Complex logic belongs in application code.

## Interview Tips

**Explain Purpose**: Load balancing enables horizontal scaling and high availability.

**Discuss Algorithm Trade-offs**: "Round Robin for similar servers, Least Connections for varying durations."

**Address High Availability**: "Deploy pair of load balancers with keepalived for failover."

**Layer 4 vs Layer 7**: "Layer 7 for path-based routing to microservices. Layer 4 for high throughput."

**Calculate Numbers**: "For 10K RPS with 100 RPS per server, need 100 servers plus redundancy."

**Real-World Examples**: Reference Netflix, Amazon load balancing at massive scale.

## See It In Action

Complete the load balancing design challenges: **[Load Balancing Challenges](./challenges.md)**

## Additional Resources

**Documentation**:
- [Nginx Load Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/)
- [HAProxy Configuration](http://www.haproxy.org/)
- [AWS Elastic Load Balancing](https://docs.aws.amazon.com/elasticloadbalancing/)

**Articles**:
- [Google SRE Book - Load Balancing at Frontend](https://sre.google/sre-book/load-balancing-frontend/)
- [Introduction to Modern Network Load Balancing](https://blog.envoyproxy.io/introduction-to-modern-network-load-balancing-and-proxying-a57f6ff80236)

**Tools**:
- Nginx, HAProxy, Envoy: Load balancers
- AWS ALB/NLB: Managed cloud load balancing
- k6, Apache Bench: Load testing tools
