# Load Balancing

## What is Load Balancing?

**Load Balancing** is the process of distributing incoming network traffic across multiple servers. Think of it like a traffic cop at a busy intersection, directing cars to different lanes to prevent congestion.

When you have multiple servers (horizontal scaling), you need a way to decide which server handles each request. That's where load balancers come in!

## Why Do We Need Load Balancers?

Imagine you have 5 servers running your application:
- **Without Load Balancer**: Users connect randomly, some servers get overloaded while others sit idle
- **With Load Balancer**: Traffic is evenly distributed, all servers work efficiently

### Benefits:
1. **Even Distribution**: No single server gets overworked
2. **High Availability**: If one server fails, traffic goes to healthy servers
3. **Scalability**: Easily add or remove servers
4. **Better Performance**: Reduces response time by using servers efficiently
5. **Maintenance**: Take servers offline for updates without downtime

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

## Challenges

1. **Single Point of Failure**: Load balancer itself can fail (use redundancy)
2. **Bottleneck**: Load balancer can become the bottleneck at very high scale
3. **Cost**: Hardware load balancers are expensive
4. **Complexity**: More moving parts to configure and monitor

## Summary

- Load balancers distribute traffic across multiple servers
- Many algorithms available (Round Robin, Least Connections, etc.)
- Layer 4 vs Layer 7 routing offer different capabilities
- Health checks ensure traffic only goes to healthy servers
- Essential for high availability and scalability
- Avoid sticky sessions; use shared session storage instead

## Next Steps

Complete the challenges to test your knowledge, then move on to **[Caching](../03-caching/)** to learn how to make your applications even faster!
