# Availability Pattern Trade-Offs

## 1. High Availability vs. Cost

**High Availability (99.99%):**
- Pros: Reliable, customer trust
- Cons: Expensive (multi-region, redundancy)

**Lower Availability (99%):**
- Pros: Much cheaper
- Cons: More downtime, customer impact

**Choose based on:** Business impact of downtime

## 2. Automatic vs. Manual Failover

**Automatic:**
- Pros: Fast recovery (seconds)
- Cons: Can trigger unnecessarily, complex

**Manual:**
- Pros: Human judgment, simpler
- Cons: Slower (minutes to hours)

**Best:** Automatic for critical, manual for less critical

## 3. Redundancy Level

**Active-Active:**
- Pros: Max availability, no waste
- Cons: Most expensive, complex

**Active-Passive:**
- Pros: Good availability, simpler
- Cons: Wasted capacity

**No Redundancy:**
- Pros: Cheapest
- Cons: Single point of failure

**Choose:** Based on budget and criticality

See [trade-offs.md](./trade-offs.md) for detailed analysis.
