# Design Exercise: Netflix Request Routing System

## 1. Design Problem

### Problem Statement
Design a comprehensive load balancing and request routing system for a video streaming platform like Netflix. The system must intelligently distribute millions of concurrent requests across thousands of servers, ensure high availability, optimize content delivery, and provide a seamless viewing experience for users worldwide.

### Context and Constraints
- Millions of users streaming video simultaneously across the globe
- Multiple types of traffic: API calls, video streaming, web page requests, metadata queries
- Different content types require different routing strategies
- Server failures are inevitable and must be handled transparently
- Periodic maintenance and deployments must not disrupt service
- Geographic distribution with multiple data centers
- Peak traffic during evening hours (5-10x normal traffic)
- Need to route based on user location, content type, and server health

### Requirements

#### Functional Requirements
- Route user requests to healthy backend servers
- Support multiple routing algorithms (round-robin, least connections, weighted)
- Perform health checks on backend servers
- Remove unhealthy servers from rotation automatically
- Support SSL/TLS termination at load balancer
- Route different traffic types to appropriate services (API vs. streaming)
- Enable gradual traffic migration for deployments
- Provide session persistence where needed
- Support geographic routing (users to nearest data center)
- Enable A/B testing by routing percentage of traffic to new features

#### Non-Functional Requirements
- **Scale**: 200 million concurrent users, 100,000 requests/second per data center
- **Performance**: Add < 5ms latency, 99.99% success rate
- **Availability**: 99.99% uptime (4.38 minutes downtime/month max)
- **Throughput**: 100+ Gbps traffic per data center
- **Failover**: Detect failures within 3 seconds, route around within 1 second
- **Geographic**: 20+ data centers globally, < 100ms latency for 95% of users
- **Efficiency**: Evenly distribute load, avoid server overload

## 2. Guided Questions

### Understanding Load Balancing Basics
1. **What happens without load balancing?**
   - Hint: Think about what occurs if all users connect to a single server
   - Consider: Server overload, single point of failure, wasted capacity

2. **Why do we need multiple layers of load balancing?**
   - Hint: Different levels serve different purposes
   - Think about: DNS, network load balancers, application load balancers

3. **What's the difference between Layer 4 and Layer 7 load balancing?**
   - Hint: What information can each layer see?
   - Think about: IP addresses vs. HTTP headers, speed vs. intelligence

### Health Checks and Failures
4. **How do we detect when a server fails?**
   - Hint: Can't just wait for users to report problems
   - Think about: Active health checks, passive monitoring, heartbeats

5. **What if a server fails in the middle of a user's video stream?**
   - Hint: User shouldn't notice if possible
   - Think about: Retry logic, client-side buffering, stateless design

### Routing Strategies
6. **When would you use round-robin vs. least connections?**
   - Hint: Consider different types of requests
   - Think about: Quick API calls vs. long-lived video streams

7. **How do we route users to the nearest data center?**
   - Hint: Multiple approaches at different layers
   - Think about: DNS, anycast, client detection

### Advanced Scenarios
8. **How do we deploy new code without downtime?**
   - Hint: Can't take all servers offline at once
   - Think about: Blue-green deployment, canary releases, gradual rollout

9. **What if one data center becomes overloaded?**
   - Hint: Need to overflow to other regions
   - Think about: Global load balancing, capacity planning, cross-region routing

## 3. Step-by-Step Guidance

### Step 1: Identify Load Balancing Layers
Netflix-scale requires multiple layers of load balancing:

**Layer 1 - DNS Load Balancing (Global)**
- Routes users to appropriate data center
- Geographic routing based on user location
- Provides disaster recovery (failover to backup regions)

**Layer 2 - Network Load Balancer (Regional)**
- Layer 4 load balancing (TCP/UDP)
- Distributes traffic within data center
- High throughput, low latency

**Layer 3 - Application Load Balancer (Service-Level)**
- Layer 7 load balancing (HTTP/HTTPS)
- Content-based routing
- SSL termination

**Layer 4 - Client-Side Load Balancing (Microservices)**
- Service discovery and routing
- Direct service-to-service communication

### Step 2: Design Health Check System
Health checks determine which servers receive traffic:

**Active Health Checks:**
```
Load Balancer → Server: TCP connection attempt
Load Balancer → Server: HTTP GET /health
Server → Load Balancer: 200 OK (healthy) or timeout/error (unhealthy)
```

**Parameters to Configure:**
- Check interval: Every 10 seconds
- Timeout: 3 seconds per check
- Unhealthy threshold: 3 consecutive failures
- Healthy threshold: 2 consecutive successes

**What to Check:**
- TCP connection successful
- HTTP endpoint responds with 200 OK
- Response time < threshold
- Application-specific metrics (CPU, memory, disk)

### Step 3: Choose Routing Algorithms
Different algorithms for different use cases:

**For API Requests (Short-lived):**
- Round Robin: Simple, evenly distributes
- Weighted Round Robin: Account for server capacity differences
- Random: Similar to round robin, simpler

**For Video Streaming (Long-lived):**
- Least Connections: Route to server with fewest active streams
- Resource-Based: Route based on CPU/memory availability

**For Sticky Sessions (If needed):**
- IP Hash: Same client always routes to same server
- Cookie-Based: More reliable than IP hashing

### Step 4: Handle Different Traffic Types
Netflix has different types of traffic requiring different routing:

**Video Streaming:**
- Route to streaming servers (optimized for bandwidth)
- Use least connections algorithm
- Longer timeouts (streams can last hours)
- Large connection pools

**API Calls:**
- Route to API servers (optimized for compute)
- Use round robin or random
- Short timeouts (< 1 second)
- High request rate

**Web Pages:**
- Route to web servers (serve HTML/CSS/JS)
- Can be cached aggressively at CDN
- Medium timeouts

**Path-Based Routing:**
```
/api/*      → API Service Cluster
/stream/*   → Streaming Service Cluster
/browse/*   → Web Service Cluster
/search/*   → Search Service Cluster
```

### Step 5: Geographic Distribution
Route users to nearest data center for low latency:

**DNS-Based Geographic Routing:**
```
User in US East → us-east.netflix.com → US East Data Center
User in Europe → eu-west.netflix.com → EU West Data Center
User in Asia   → asia.netflix.com    → Asia Data Center
```

**Benefits:**
- Lower latency (shorter distance)
- Data sovereignty compliance
- Regional failover capability

**Challenges:**
- DNS caching (can't instantly redirect)
- Uneven geographic distribution
- Cross-region data synchronization

### Step 6: Implement Failover Strategy
What happens when things go wrong:

**Server Failure:**
1. Health check fails 3 consecutive times
2. Load balancer marks server unhealthy
3. No new connections routed to server
4. Existing connections allowed to drain (30-60 seconds)
5. Alert sent to operations team

**Data Center Failure:**
1. Multiple servers fail health checks
2. Global load balancer detects data center degraded
3. DNS updated to route traffic to backup regions
4. DNS TTL = 60 seconds (fast failover)
5. Users automatically reconnected to healthy region

**Gradual Degradation:**
- If capacity reduced by 50%, overload remaining servers
- Solution: Rate limiting, queue requests, show user-friendly errors

### Step 7: Enable Zero-Downtime Deployments
Deploy new code without interrupting users:

**Blue-Green Deployment:**
```
1. Blue environment: Current version (100% traffic)
2. Green environment: Deploy new version (0% traffic)
3. Test green environment
4. Switch: Green gets 100% traffic, Blue becomes standby
5. If issues: Instant rollback to Blue
```

**Canary Deployment:**
```
1. Deploy new version to 1 server (1% traffic)
2. Monitor metrics: Error rates, latency, crashes
3. If healthy: Gradually increase (5%, 10%, 25%, 50%, 100%)
4. If problems: Stop rollout, investigate
5. Automated rollback on error rate increase
```

## 4. Sample Solution

### Complete Architecture

```
                         [Users Worldwide]
                                |
                        ┌───────┴────────┐
                        │   GeoDNS       │
                        │  (Route 53)    │
                        └───┬────────┬───┘
                            |        |
                    ┌───────┘        └───────┐
                    |                        |
              [US Data Center]         [EU Data Center]
                    |                        |
            ┌───────┴────────┐               |
            │   Layer 4 LB   │               |
            │   (AWS NLB)    │               |
            └────────┬───────┘               |
                     |                       |
         ┌───────────┼──────────┐            |
         |           |          |            |
    ┌────▼────┐ ┌───▼────┐ ┌───▼────┐       |
    │Layer 7  │ │Layer 7 │ │Layer 7 │       |
    │  LB 1   │ │  LB 2  │ │  LB 3  │       |
    │(AWS ALB)│ │        │ │        │       |
    └────┬────┘ └────┬───┘ └────┬───┘       |
         |           |          |            |
    ┌────┴─────┬─────┴────┬─────┴──────┐    |
    |          |          |            |    |
[API Servers] [Stream] [Web]      [Search] |
   (3-AZ)    Servers  Servers     Servers   |
             (3-AZ)   (3-AZ)      (3-AZ)    |
                                            |
                            [Similar Architecture]
```

### Detailed Component Design

#### 1. DNS Layer (Route 53)
**Purpose**: Route users to optimal data center

**Configuration:**
```
netflix.com A RECORD (Geoproximity Routing)
├── US-EAST:  us-east-nlb.netflix.com
├── US-WEST:  us-west-nlb.netflix.com
├── EU-WEST:  eu-west-nlb.netflix.com
├── ASIA-SE:  asia-se-nlb.netflix.com
└── SA-EAST:  sa-east-nlb.netflix.com

Health Check: HTTP GET /health every 30s
Failover: Automatic to next nearest region
TTL: 60 seconds (fast failover)
```

**Routing Policy:**
- Primary: Route to nearest data center (lowest latency)
- Secondary: If primary unhealthy, route to next nearest
- Tertiary: If multiple failures, route to any healthy region

#### 2. Network Load Balancer (Layer 4)
**Purpose**: High-throughput traffic distribution within data center

**Configuration:**
```yaml
Type: Network Load Balancer (TCP/UDP)
Throughput: 100+ Gbps
Latency: < 1ms additional
Availability Zones: 3 (for redundancy)

Listeners:
  - Port 443 (HTTPS) → Target Group: ALB Fleet
  - Port 80 (HTTP) → Redirect to 443

Health Checks:
  Protocol: TCP
  Port: 443
  Interval: 10 seconds
  Healthy Threshold: 2
  Unhealthy Threshold: 3
```

**Why Layer 4 Here?**
- Handles massive throughput (millions of packets/sec)
- Very low latency (< 1ms overhead)
- Simple TCP/UDP routing
- Passes traffic to Layer 7 LBs for intelligent routing

#### 3. Application Load Balancer (Layer 7)
**Purpose**: Content-based routing and SSL termination

**Configuration:**
```yaml
Type: Application Load Balancer (HTTP/HTTPS)
SSL Termination: Yes (certificates managed by ACM)
Availability Zones: 3

Routing Rules:
  - Path: /api/*
    Target: API Server Target Group
    Algorithm: Round Robin
    
  - Path: /stream/*
    Target: Streaming Server Target Group
    Algorithm: Least Outstanding Requests
    
  - Path: /browse/*
    Target: Web Server Target Group
    Algorithm: Round Robin
    
  - Path: /search/*
    Target: Search Server Target Group
    Algorithm: Random

Health Checks:
  Path: /health
  Protocol: HTTP
  Port: 8080
  Interval: 10 seconds
  Timeout: 5 seconds
  Healthy Threshold: 2
  Unhealthy Threshold: 3
  Success Codes: 200
```

**Sticky Sessions (if needed):**
```yaml
Session Affinity: Cookie-based
Cookie Name: AWSALB
Duration: 1 hour
```

#### 4. Health Check Endpoint Implementation
Each server implements a health endpoint:

```python
from flask import Flask, jsonify
import psutil

app = Flask(__name__)

@app.route('/health', methods=['GET'])
def health_check():
    # Check system resources
    cpu_percent = psutil.cpu_percent(interval=1)
    memory_percent = psutil.virtual_memory().percent
    disk_percent = psutil.disk_usage('/').percent
    
    # Define health thresholds
    if cpu_percent > 90:
        return jsonify({
            'status': 'unhealthy',
            'reason': 'CPU overload',
            'cpu_percent': cpu_percent
        }), 503
    
    if memory_percent > 90:
        return jsonify({
            'status': 'unhealthy',
            'reason': 'Memory overload',
            'memory_percent': memory_percent
        }), 503
    
    if disk_percent > 90:
        return jsonify({
            'status': 'unhealthy',
            'reason': 'Disk full',
            'disk_percent': disk_percent
        }), 503
    
    # Check critical dependencies
    try:
        # Test database connection
        db.execute('SELECT 1')
    except:
        return jsonify({
            'status': 'unhealthy',
            'reason': 'Database unavailable'
        }), 503
    
    return jsonify({
        'status': 'healthy',
        'cpu_percent': cpu_percent,
        'memory_percent': memory_percent,
        'disk_percent': disk_percent
    }), 200
```

### Load Balancing Algorithms in Detail

#### Round Robin Implementation
```python
class RoundRobinLoadBalancer:
    def __init__(self, servers):
        self.servers = servers
        self.current_index = 0
        self.lock = threading.Lock()
    
    def get_next_server(self):
        with self.lock:
            # Filter healthy servers
            healthy_servers = [s for s in self.servers if s.is_healthy]
            
            if not healthy_servers:
                raise NoHealthyServersException()
            
            # Get next server
            server = healthy_servers[self.current_index % len(healthy_servers)]
            self.current_index += 1
            
            return server
```

#### Least Connections Implementation
```python
class LeastConnectionsLoadBalancer:
    def __init__(self, servers):
        self.servers = servers
        self.connection_counts = {s: 0 for s in servers}
        self.lock = threading.Lock()
    
    def get_next_server(self):
        with self.lock:
            # Filter healthy servers
            healthy_servers = [s for s in self.servers if s.is_healthy]
            
            if not healthy_servers:
                raise NoHealthyServersException()
            
            # Find server with least connections
            server = min(healthy_servers, 
                        key=lambda s: self.connection_counts[s])
            
            self.connection_counts[server] += 1
            return server
    
    def release_connection(self, server):
        with self.lock:
            self.connection_counts[server] -= 1
```

#### Weighted Round Robin
```python
class WeightedRoundRobinLoadBalancer:
    def __init__(self, servers):
        # servers is list of (server, weight) tuples
        # e.g., [(server1, 3), (server2, 2), (server3, 1)]
        self.servers = []
        for server, weight in servers:
            # Add server 'weight' times to the list
            self.servers.extend([server] * weight)
        self.current_index = 0
        self.lock = threading.Lock()
    
    def get_next_server(self):
        with self.lock:
            healthy_servers = [s for s in self.servers if s.is_healthy]
            
            if not healthy_servers:
                raise NoHealthyServersException()
            
            server = healthy_servers[self.current_index % len(healthy_servers)]
            self.current_index += 1
            
            return server
```

### Zero-Downtime Deployment Strategy

#### Canary Deployment Implementation
```python
class CanaryDeploymentController:
    def __init__(self, load_balancer):
        self.lb = load_balancer
        self.old_version_servers = []
        self.new_version_servers = []
        self.canary_percentage = 0
    
    def start_canary(self, new_servers, initial_percentage=5):
        """Start canary deployment with small traffic percentage"""
        self.new_version_servers = new_servers
        self.canary_percentage = initial_percentage
        self.update_routing()
        
        # Monitor metrics
        self.monitor_deployment()
    
    def update_routing(self):
        """Update load balancer routing based on canary percentage"""
        total_weight = 100
        old_weight = 100 - self.canary_percentage
        new_weight = self.canary_percentage
        
        self.lb.set_weights({
            'old_version': old_weight,
            'new_version': new_weight
        })
    
    def monitor_deployment(self):
        """Monitor error rates and latency"""
        old_metrics = self.get_metrics(self.old_version_servers)
        new_metrics = self.get_metrics(self.new_version_servers)
        
        # Compare error rates
        if new_metrics['error_rate'] > old_metrics['error_rate'] * 1.5:
            # 50% increase in errors - rollback
            self.rollback()
            return
        
        # Compare latency
        if new_metrics['p99_latency'] > old_metrics['p99_latency'] * 1.2:
            # 20% increase in latency - rollback
            self.rollback()
            return
        
        # If healthy, increase traffic
        if self.canary_percentage < 100:
            self.increase_canary_traffic()
    
    def increase_canary_traffic(self):
        """Gradually increase traffic to new version"""
        progression = [5, 10, 25, 50, 100]
        current_index = progression.index(self.canary_percentage)
        
        if current_index < len(progression) - 1:
            self.canary_percentage = progression[current_index + 1]
            self.update_routing()
            
            # Wait and monitor before next increase
            time.sleep(300)  # 5 minutes
            self.monitor_deployment()
    
    def rollback(self):
        """Rollback to old version"""
        self.canary_percentage = 0
        self.update_routing()
        alert("Canary deployment failed - rolled back")
```

### Design Choices Explained

#### Why Multi-Layer Load Balancing?
**Layer 1 (DNS)**:
- Geographic routing (user to nearest data center)
- Disaster recovery (data center failover)
- Cost optimization (route to cheaper regions)

**Layer 2 (Network LB)**:
- High throughput (100+ Gbps)
- Low latency (< 1ms)
- Simple, reliable, fast

**Layer 3 (Application LB)**:
- Intelligent routing (path-based)
- SSL termination (offload encryption)
- Content inspection

**Trade-off**: Complexity vs. flexibility and performance

#### Why Active Health Checks Over Passive?
**Active checks**:
- Proactive detection before user impact
- Configurable check frequency
- Can test specific functionality

**Passive checks**:
- Only detect after user requests fail
- No overhead when traffic is low
- React to actual user experience

**Our choice**: Active checks every 10 seconds
- Fast failure detection (< 30 seconds)
- Prevents user impact
- Small overhead (1 request/10s per server)

### Alternative Approaches

#### Alternative 1: Client-Side Load Balancing
Instead of centralized load balancers, clients make routing decisions:

**Architecture:**
```
Client → Service Discovery (Consul/Eureka)
       → Client has list of servers
       → Client picks server (round robin/random)
       → Direct connection to server
```

**Pros**:
- No single point of failure (load balancer)
- Lower latency (no extra hop)
- Scales infinitely (clients do the work)

**Cons**:
- More complex clients
- Harder to enforce policies
- Less visibility and control

**When to use**: Microservices architecture, service-to-service calls

#### Alternative 2: DNS Round Robin
Use DNS to distribute traffic:

**Configuration:**
```
netflix.com A 192.168.1.1
netflix.com A 192.168.1.2
netflix.com A 192.168.1.3
```

**Pros**:
- Simple, no additional infrastructure
- Built into DNS

**Cons**:
- No health checks
- No sophisticated algorithms
- DNS caching prevents fast updates
- Uneven distribution

**When to use**: Small scale, simple deployments, cost-sensitive

#### Alternative 3: Hardware Load Balancers
Use physical load balancer appliances (F5, Citrix):

**Pros**:
- Very high performance (dedicated hardware)
- Advanced features (WAF, DDoS protection)
- Mature, battle-tested

**Cons**:
- Expensive ($10K-$100K+)
- Limited scalability (physical limits)
- Slow to deploy/configure
- Single vendor lock-in

**When to use**: On-premise data centers, legacy environments, specific compliance needs

## 5. Extension Challenges

### Challenge 1: Handle DDoS Attack
**Scenario**: Attacker sends 1 million requests/second to overwhelm system

**What would you do?**
<details>
<summary>Hints and Solution</summary>

**Challenges**:
- Massive traffic volume exhausts load balancer capacity
- Legitimate users can't access service
- Need to distinguish attack traffic from real users

**Solutions**:

1. **Rate Limiting at Multiple Layers**:
   ```
   - WAF (Web Application Firewall): 100 requests/minute per IP
   - Load Balancer: 1000 requests/second per client
   - Application: 10 requests/second per user account
   ```

2. **Geographic Filtering**:
   - If attack originates from specific regions, block at DNS/WAF
   - Allowlist known good regions during attack

3. **Challenge-Response (CAPTCHA)**:
   - Require CAPTCHA for suspicious traffic
   - Rate limit IPs that fail challenges

4. **DDoS Protection Services**:
   - AWS Shield (automatic DDoS protection)
   - Cloudflare DDoS protection
   - Akamai Prolexic

5. **Elastic Scaling**:
   - Auto-scale load balancers and servers
   - Absorb attack with massive capacity
   - Cost vs. availability trade-off

6. **Traffic Prioritization**:
   - Known users get priority
   - New/anonymous users get queued
   - Premium subscribers always get through

**Implementation**:
```python
class DDoSProtection:
    def __init__(self):
        self.ip_request_counts = {}
        self.blocked_ips = set()
    
    def should_block(self, ip_address):
        # Check if already blocked
        if ip_address in self.blocked_ips:
            return True
        
        # Count requests
        count = self.ip_request_counts.get(ip_address, 0)
        count += 1
        self.ip_request_counts[ip_address] = count
        
        # Block if exceeds threshold
        if count > 1000:  # 1000 requests in window
            self.blocked_ips.add(ip_address)
            alert(f"Blocked IP for DDoS: {ip_address}")
            return True
        
        return False
```
</details>

### Challenge 2: Cross-Region Failover
**Scenario**: Entire US East data center goes offline

**How do you handle this?**
<details>
<summary>Hints and Solution</summary>

**Challenges**:
- Millions of users suddenly lose service
- Need to redirect to other regions
- Other regions must handle 2x traffic
- DNS caching delays redirection
- Data may not be fully replicated

**Solutions**:

1. **Automated DNS Failover**:
   ```yaml
   Primary: US-EAST (healthy check every 30s)
   Secondary: US-WEST (standby)
   TTL: 60 seconds
   
   On failure:
   - Route 53 detects unhealthy primary
   - Updates DNS to point to US-WEST
   - Within 60 seconds, traffic redirected
   ```

2. **Anycast Routing**:
   - Same IP announced from multiple data centers
   - Network automatically routes to nearest healthy center
   - Instant failover (no DNS delay)

3. **Over-Provisioning Backup Regions**:
   - Each region runs at 50% capacity
   - Can absorb traffic from failed region
   - Cost: 2x infrastructure, but prevents outage

4. **Data Replication Strategy**:
   ```
   - Critical data: Sync replication (always consistent)
   - User data: Async replication (eventual consistency)
   - Video content: Replicated to all regions (read-heavy)
   ```

5. **Graceful Degradation**:
   - If backup region overwhelmed, show cached content
   - Disable non-critical features (recommendations)
   - Prioritize video streaming over browsing

6. **Client-Side Retry Logic**:
   ```javascript
   async function fetchWithFailover(url) {
     const regions = [
       'https://us-east.netflix.com',
       'https://us-west.netflix.com',
       'https://eu-west.netflix.com'
     ];
     
     for (const region of regions) {
       try {
         const response = await fetch(`${region}${url}`, { timeout: 5000 });
         return response;
       } catch (error) {
         console.log(`Failed to reach ${region}, trying next...`);
         continue;
       }
     }
     
     throw new Error('All regions unavailable');
   }
   ```

**Monitoring & Alerting**:
- Detect: Multiple server failures within 1 minute
- Alert: Page on-call engineers immediately
- Automate: Trigger failover without human intervention
- Communicate: Update status page, notify users
</details>

### Challenge 3: Gradual Traffic Migration
**Scenario**: Migrate from old infrastructure to new cloud infrastructure

**How do you safely migrate traffic?**
<details>
<summary>Hints and Solution</summary>

**Challenges**:
- Can't migrate all at once (too risky)
- Need to compare performance (old vs. new)
- Must be able to rollback quickly
- Some users on old, some on new (inconsistency)

**Migration Strategy**:

**Phase 1: Shadow Traffic (0% users affected)**
```
Primary: Old infrastructure (100% traffic)
Shadow: New infrastructure (receives copies, responses discarded)

Purpose: Test new infrastructure without risk
Duration: 1 week
```

**Phase 2: Canary Release (1% users)**
```
Old: 99% traffic
New: 1% traffic

Monitor:
- Error rates (should be equal)
- Latency (should be similar or better)
- Resource utilization

Duration: 3 days
```

**Phase 3: Gradual Rollout**
```
Week 1: 5% → New
Week 2: 10% → New
Week 3: 25% → New
Week 4: 50% → New
Week 5: 75% → New
Week 6: 100% → New

At each step:
- Monitor for 3 days
- If issues: Rollback immediately
- If healthy: Proceed to next step
```

**Implementation**:
```python
class TrafficMigrationController:
    def __init__(self):
        self.old_infrastructure = OldLoadBalancer()
        self.new_infrastructure = NewLoadBalancer()
        self.migration_percentage = 0
    
    def route_request(self, request):
        # Determine which infrastructure to use
        random_value = random.randint(1, 100)
        
        if random_value <= self.migration_percentage:
            # Route to new infrastructure
            return self.new_infrastructure.handle(request)
        else:
            # Route to old infrastructure
            return self.old_infrastructure.handle(request)
    
    def increase_migration(self, new_percentage):
        # Only increase if metrics are healthy
        if self.are_metrics_healthy():
            self.migration_percentage = new_percentage
            log(f"Increased migration to {new_percentage}%")
        else:
            alert("Metrics unhealthy - not increasing migration")
    
    def are_metrics_healthy(self):
        old_metrics = self.old_infrastructure.get_metrics()
        new_metrics = self.new_infrastructure.get_metrics()
        
        # Compare error rates (new should be similar or better)
        if new_metrics.error_rate > old_metrics.error_rate * 1.1:
            return False
        
        # Compare latency (new should be similar or better)
        if new_metrics.p99_latency > old_metrics.p99_latency * 1.1:
            return False
        
        return True
    
    def rollback(self):
        self.migration_percentage = 0
        alert("Rolled back migration to 0%")
```

**Rollback Plan**:
```
If any issues detected:
1. Immediately set migration_percentage to 0
2. All traffic routes to old infrastructure
3. Investigate issues in new infrastructure
4. Fix and re-test in shadow mode
5. Restart migration from Phase 2
```
</details>

### Challenge 4: Optimize for Cost
**Scenario**: Reduce load balancing infrastructure costs by 50%

**What would you optimize?**
<details>
<summary>Hints and Solution</summary>

**Current Costs** (example for Netflix scale):
- Network Load Balancers: $500K/month (20 NLBs @ $25K each)
- Application Load Balancers: $300K/month (100 ALBs @ $3K each)
- Data Transfer: $400K/month (outbound traffic)
- Health Checks: $50K/month (millions of checks)
- **Total**: $1.25M/month

**Optimization Strategies**:

1. **Consolidate Load Balancers** (-30% LB cost):
   - Combine multiple small ALBs into fewer large ones
   - Use path-based routing instead of separate LBs
   - Savings: ~$240K/month

2. **Optimize Health Checks** (-80% health check cost):
   ```
   Before: Check every 10 seconds (360 checks/hour per server)
   After: Check every 30 seconds (120 checks/hour per server)
   
   Before: HTTP GET with full response
   After: TCP connection only (faster, cheaper)
   ```
   - Savings: ~$40K/month

3. **Client-Side Load Balancing for Internal Traffic** (-40% internal LB cost):
   - Microservices communicate directly
   - Use service discovery (Consul)
   - Eliminate internal load balancers
   - Savings: ~$100K/month

4. **DNS Routing Instead of NLB** (-50% NLB cost):
   - For some use cases, DNS round-robin sufficient
   - Reduce NLB count from 20 to 10
   - Savings: ~$250K/month

5. **Optimize Data Transfer** (-30% transfer cost):
   - Keep traffic within availability zones
   - Use VPC endpoints (no internet gateway charges)
   - Compression for API responses
   - Savings: ~$120K/month

**Total Savings**: ~$750K/month (60% reduction) ✓

**Trade-offs**:
- Slightly slower failure detection (30s vs 10s health checks)
- Less redundancy (fewer load balancers)
- More complexity (client-side load balancing)
- Slightly less reliable (DNS vs NLB)
</details>

### Challenge 5: Support WebSocket Connections
**Scenario**: Add real-time features requiring persistent WebSocket connections

**How do you load balance WebSockets?**
<details>
<summary>Hints and Solution</summary>

**Challenges**:
- WebSockets are long-lived (hours/days)
- Connection state must be maintained
- Can't use simple round-robin (creates imbalance over time)
- Need to handle server restarts without dropping connections

**Solutions**:

1. **Use Least Connections Algorithm**:
   ```python
   # WebSocket connections are long-lived
   # Route new connections to servers with fewest connections
   algorithm = "least_outstanding_requests"
   
   # Monitor active WebSocket count per server
   for server in servers:
       server.websocket_count = count_active_websockets(server)
   
   # Route new connection to server with lowest count
   def route_new_websocket(request):
       server = min(servers, key=lambda s: s.websocket_count)
       server.websocket_count += 1
       return server
   ```

2. **Connection Draining for Deployments**:
   ```
   When deploying new version:
   1. Mark old servers as "draining"
   2. Stop routing new WebSocket connections to them
   3. Wait for existing connections to close naturally
   4. After timeout (1 hour), forcefully close remaining
   5. Shutdown old servers
   ```

3. **Sticky Sessions (If Stateful)**:
   ```yaml
   # If WebSocket requires server-side state
   Session Affinity: Cookie-based
   Cookie Name: WS_SERVER_ID
   Duration: 24 hours
   
   # User always reconnects to same server
   # State doesn't need to be replicated
   ```

4. **Reconnection Strategy (Client-Side)**:
   ```javascript
   class WebSocketClient {
     connect(url) {
       this.ws = new WebSocket(url);
       
       this.ws.onclose = () => {
         // Exponential backoff reconnect
         setTimeout(() => this.connect(url), this.getBackoff());
       };
       
       this.ws.onerror = () => {
         console.log('WebSocket error, reconnecting...');
       };
     }
     
     getBackoff() {
       // 1s, 2s, 4s, 8s, max 30s
       return Math.min(1000 * Math.pow(2, this.retryCount++), 30000);
     }
   }
   ```

5. **Health Checks for WebSocket Servers**:
   ```python
   @app.route('/health')
   def health_check():
       # Check active WebSocket count
       active_connections = get_active_websocket_count()
       
       # If too many connections, mark unhealthy
       if active_connections > 10000:
           return jsonify({
               'status': 'unhealthy',
               'reason': 'Too many active connections'
           }), 503
       
       return jsonify({
           'status': 'healthy',
           'active_connections': active_connections
       }), 200
   ```

6. **Load Balancer Configuration**:
   ```yaml
   Target Group:
     Protocol: HTTP
     Port: 8080
     Algorithm: Least Outstanding Requests
     
   Stickiness:
     Enabled: true
     Type: application_cookie
     Cookie Name: WS_SERVER_ID
     Duration: 86400 seconds (24 hours)
   
   Health Check:
     Path: /health
     Interval: 30 seconds
     Unhealthy Threshold: 3
   
   Deregistration Delay: 3600 seconds (1 hour)
     # Allow long-lived connections to drain
   ```

**Architecture**:
```
[Client] 
   ↓ WebSocket connection
[Application Load Balancer]
   ↓ Sticky sessions enabled
[WebSocket Server 1] [WebSocket Server 2] [WebSocket Server 3]
   ↓                     ↓                     ↓
[Redis] (for state sync if needed)
```
</details>

---

## Summary

This design exercise demonstrates how to build a production-ready load balancing system by:
- Using multi-layer load balancing for optimal performance
- Implementing robust health checks for reliability
- Choosing appropriate algorithms for different traffic types
- Designing for zero-downtime deployments
- Planning for geographic distribution and failover
- Optimizing for cost while maintaining performance

**Key Takeaways**:
1. **Layer appropriately**: Different layers solve different problems
2. **Monitor everything**: Can't route intelligently without data
3. **Plan for failure**: Servers will fail; design for it
4. **Gradual changes**: Never migrate 100% at once
5. **Geographic awareness**: Route users to nearby servers
6. **Cost vs. reliability**: Over-provisioning prevents outages but costs more
