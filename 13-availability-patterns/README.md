# Availability Patterns

## What is Availability?

**Availability** is the percentage of time a system is operational and accessible.

**Formula:**
```
Availability = (Uptime / Total Time) × 100%
```

**Example:**
```
365 days in a year = 8,760 hours
99.9% availability = 8,751.24 hours uptime
                   = 8.76 hours downtime per year
```

## Availability Levels (Nines)

| Level | Uptime % | Downtime/Year | Downtime/Month | Use Case |
|-------|----------|---------------|----------------|----------|
| 90% | 90% | 36.5 days | 3 days | Internal tools |
| 99% | 99% | 3.65 days | 7.2 hours | Small business |
| 99.9% | 99.9% | 8.76 hours | 43 minutes | SaaS products |
| 99.99% | 99.99% | 52.6 minutes | 4.3 minutes | Enterprise |
| 99.999% | 99.999% | 5.26 minutes | 26 seconds | Mission-critical |

**Cost increases exponentially with each nine!**

## Common Causes of Downtime

### 1. Hardware Failures
- Server crashes
- Disk failures
- Network issues
- Power outages

**Mitigation:** Redundancy, monitoring, hot-swaps

### 2. Software Bugs
- Application crashes
- Memory leaks
- Deadlocks
- Race conditions

**Mitigation:** Testing, monitoring, quick rollback

### 3. Human Error
- Bad deployments
- Configuration mistakes
- Accidental deletions

**Mitigation:** Automation, canary deployments, backups

### 4. External Dependencies
- Cloud provider outages
- Third-party API failures
- DNS issues

**Mitigation:** Multiple providers, circuit breakers, fallbacks

## Availability Patterns

### 1. Redundancy

**Eliminate single points of failure**

```
Before:
[Load Balancer] → [Single Server] → [Single DB]
(If any fails, system down)

After:
[LB1] [LB2]
   ↓     ↓
[Server1] [Server2] [Server3]
       ↓     ↓
    [DB Primary] [DB Replica]
```

**Types of Redundancy:**

**Active-Active:**
- All nodes handle traffic
- Maximize resource utilization
- Complex to manage

**Active-Passive (Hot Standby):**
- Active handles traffic
- Passive ready to take over
- Simpler, some waste

**Cold Standby:**
- Backup not running
- Must start up when needed
- Cheapest, longer failover

---

### 2. Failover

**Automatic or manual switch to backup when primary fails**

**Automatic Failover:**
```
1. Primary fails
2. Health check detects failure
3. Automatic switch to secondary
4. DNS/Load balancer updated
5. Traffic flows to secondary

Downtime: Seconds to minutes
```

**Manual Failover:**
```
1. Primary fails
2. Alert sent to on-call
3. Human investigates
4. Human triggers failover
5. Traffic flows to secondary

Downtime: Minutes to hours
```

**Components:**
- Health checks (heartbeat)
- Failover logic
- DNS/Load balancer updates
- Data synchronization

**Challenges:**
- Split-brain (both think they're primary)
- Data loss during switch
- False positives (unnecessary failover)

---

### 3. Replication

**Multiple copies of data**

```
[Primary DB] → Replicate → [Replica 1]
                        → [Replica 2]
                        → [Replica 3]
```

**Synchronous Replication:**
- Write confirmed only after replicas updated
- No data loss
- Slower writes

**Asynchronous Replication:**
- Write confirmed immediately
- Data loss possible
- Faster writes

**Use:** Read scaling, disaster recovery

---

### 4. Load Balancing

**Distribute traffic across servers**

```
             [Load Balancer]
           /      |      \
    [Server1] [Server2] [Server3]
```

**Benefits:**
- No single server overwhelmed
- Automatic failover (remove failed server)
- Scale by adding servers

**Algorithms:**
- Round Robin
- Least Connections
- IP Hash
- Weighted

---

### 5. Health Checks & Monitoring

**Detect failures quickly**

```python
def health_check():
    """Check every 10 seconds"""
    if can_connect_to_db():
        if response_time < 1000ms:
            return "healthy"
    return "unhealthy"
```

**What to Monitor:**
- Server response time
- Error rates
- Database connectivity
- Disk space
- Memory usage
- CPU usage

**Alerting:**
- PagerDuty, OpsGenie for on-call
- Alert on trends (disk 90% full)
- Alert on thresholds (error rate > 1%)

---

### 6. Graceful Degradation

**System stays partially functional when components fail**

**Example: E-commerce Site**
```
Recommendations service down:
→ Don't show recommendations
→ But still allow purchases ✓

Payment service slow:
→ Queue payments
→ But still allow browsing ✓
```

**Implementation:**
- Try-catch around non-critical features
- Default values when service unavailable
- Clear user messaging

---

### 7. Circuit Breaker

**Stop calling failing service to prevent cascading failures**

```
States:
CLOSED → Requests pass through normally
OPEN → Requests fail immediately (no calls to service)
HALF-OPEN → Test if service recovered

Transition:
CLOSED → (failures exceed threshold) → OPEN
OPEN → (timeout expires) → HALF-OPEN
HALF-OPEN → (test succeeds) → CLOSED
HALF-OPEN → (test fails) → OPEN
```

**Example:**
```python
circuit_breaker = CircuitBreaker(
    failure_threshold=5,  # Open after 5 failures
    timeout=60            # Try again after 60 seconds
)

@circuit_breaker
def call_external_api():
    return requests.get('https://api.example.com')
```

---

### 8. Retry with Backoff

**Retry failed requests with increasing delays**

```python
def retry_with_backoff(func, max_retries=3):
    for i in range(max_retries):
        try:
            return func()
        except Exception:
            if i < max_retries - 1:
                time.sleep(2 ** i)  # 1s, 2s, 4s
            else:
                raise

# Usage
result = retry_with_backoff(call_external_api)
```

**Exponential Backoff:**
- Retry 1: Wait 1 second
- Retry 2: Wait 2 seconds
- Retry 3: Wait 4 seconds
- Retry 4: Wait 8 seconds

**Add jitter (randomness) to prevent thundering herd**

---

### 9. Rate Limiting

**Prevent overload from too many requests**

```
User sends 1000 requests/second
Rate limit: 100 requests/second per user

Result:
- Accept first 100 requests
- Reject remaining 900 (HTTP 429: Too Many Requests)
```

**Benefits:**
- Protect from DDoS
- Ensure fair usage
- Prevent accidental overload

---

### 10. Caching

**Store frequently accessed data in memory**

```
Request → [Cache] (hit?) → Return from cache
                     ↓ (miss)
                  [Database] → Store in cache → Return
```

**Benefits:**
- Reduce database load
- Faster responses
- System survives database issues (briefly)

**Challenges:**
- Cache invalidation
- Memory limits
- Stale data

---

### 11. Bulkheads

**Isolate failures to prevent cascade**

**Analogy:** Ship bulkheads - if one compartment floods, others stay dry.

**Example:**
```
Separate thread pools:
- Critical requests: 50 threads
- Non-critical requests: 10 threads

If non-critical floods, critical still works!
```

**Implementation:**
- Separate databases
- Separate connection pools
- Separate servers/containers

---

### 12. Chaos Engineering

**Intentionally break things to test resilience**

**Netflix Chaos Monkey:**
- Randomly terminates instances
- Forces engineers to build resilient systems
- Finds weaknesses before customers do

**Practice:**
- Test failover regularly
- Simulate database failures
- Cut network connections
- Overload systems

**"Chaos in production" reveals real issues**

---

## Multi-Region Availability

### Active-Active Multi-Region

```
[US Region]        [EU Region]
Users → [LB]       Users → [LB]
     ↓  ↓  ↓            ↓  ↓  ↓
    [Servers]          [Servers]
        ↓                  ↓
     [DB] ←→ Replicate ←→ [DB]
```

**Benefits:**
- Low latency globally
- Survives region failure
- High availability

**Challenges:**
- Data replication complexity
- Consistency issues
- Higher cost

---

### Active-Passive Multi-Region

```
[Primary Region]           [Backup Region]
Active ✓                   Standby (ready)
```

**Benefits:**
- Disaster recovery
- Lower cost than active-active
- Simpler

**Challenges:**
- Longer failover time
- Wasted capacity in backup

---

## Real-World Examples

### Example 1: Netflix

**Availability Patterns:**
- Multi-region active-active
- Chaos engineering (Chaos Monkey)
- Circuit breakers everywhere
- Extensive caching (CDN)
- Graceful degradation
- Auto-scaling

**Result:** 99.99%+ availability

---

### Example 2: Gmail

**Availability Patterns:**
- Multi-region replication
- Automatic failover
- Multiple datacenters
- Extensive redundancy

**Result:** 99.978% availability (claimed)

---

### Example 3: AWS

**They provide:**
- Multiple Availability Zones (AZs)
- Multiple Regions
- Load balancers (ELB)
- Auto-scaling
- Health checks
- S3 (99.999999999% durability)

**You use these to build highly available apps**

---

## Calculating Availability

### Series (Chain):
```
[Component A] → [Component B]
Availability = A × B

Example:
Load Balancer (99.9%) → Server (99.9%)
Total: 0.999 × 0.999 = 0.998 = 99.8%
```

### Parallel (Redundant):
```
    [Component A]
  /              \
Start            End
  \              /
    [Component B]

Availability = 1 - (1-A) × (1-B)

Example:
Two servers (99% each):
1 - (1-0.99) × (1-0.99) = 1 - 0.0001 = 99.99%
```

**More redundancy → Higher availability**

---

## Best Practices

### 1. Design for Failure
- Assume everything will fail
- Plan for failures
- Test failure scenarios

### 2. Automate Recovery
- Automatic failover
- Auto-scaling
- Self-healing systems

### 3. Monitor Everything
- Metrics, logs, traces
- Alert on anomalies
- Fast detection → Fast recovery

### 4. Use Redundancy
- Multiple servers
- Multiple datacenters
- Multiple providers

### 5. Practice Failover
- Test disaster recovery
- Document runbooks
- Train team

### 6. Limit Blast Radius
- Bulkheads
- Circuit breakers
- Gradual rollouts

### 7. Have Rollback Plans
- Easy rollback of deployments
- Database backups
- Configuration version control

---

## Summary

- **Availability** = Uptime / Total Time
- **Patterns**: Redundancy, failover, replication, load balancing, circuit breakers
- **Trade-off**: Cost vs. availability (diminishing returns)
- **Design for failure**, not success
- **Automate** recovery where possible
- **Test** regularly (chaos engineering)
- **Monitor** everything for fast detection

**Key Insight:** Each "nine" of availability costs exponentially more!

## Next Steps

Continue to **[CDN](../14-cdn/)** to learn how content delivery networks improve both availability and performance globally!
