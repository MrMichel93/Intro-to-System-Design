# Availability Pattern Design Exercises

## Exercise 1: Calculate Availability

Given:
- Load Balancer: 99.9%
- 2 Web Servers (parallel): 99% each
- Database: 99.9%

Calculate total system availability.

<details>
<summary>Solution</summary>

**Series:** LB → Servers → DB

**Servers (parallel):**
1 - (1-0.99)×(1-0.99) = 1 - 0.0001 = 0.9999 = 99.99%

**Total (series):**
0.999 × 0.9999 × 0.999 = 0.997 = 99.7%

**Downtime:** ~26 hours/year

</details>

## Exercise 2: Design for 99.99%

Current: Single server, single database = 99% availability
Target: 99.99% availability

What patterns to add?

<details>
<summary>Solution</summary>

**Add:**
1. Load balancer with 2+ servers (redundancy)
2. Database replication with automatic failover
3. Health checks and monitoring
4. Multi-AZ deployment

**Result:**
- Servers: 2× at 99% = 99.99%
- DB: Primary + replica with failover
- Load balancer: 99.99%
- Total: ~99.99%

</details>

More exercises in [design-exercise.md](./design-exercise.md).
