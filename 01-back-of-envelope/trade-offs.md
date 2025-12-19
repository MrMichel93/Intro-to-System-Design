# Trade-Offs in Capacity Planning

Capacity estimation involves making trade-offs between accuracy, cost, performance, and complexity.

## 1. Accuracy vs. Speed of Estimation

### Trade-Off
Precise calculations take time. Quick estimates are less accurate.

**Precise Approach (Slow):**
- Count every edge case
- Account for all overhead
- Detailed breakdowns
- Time: Hours

**Quick Estimate (Fast):**
- Round numbers aggressively
- Focus on order of magnitude
- Ignore minor factors
- Time: Minutes

**When to Use Each:**
- **Quick**: Initial planning, interviews, brainstorming
- **Precise**: Final architecture, budget planning, capacity purchasing

---

## 2. Over-Provisioning vs. Under-Provisioning

### Trade-Off
Plan for too much capacity (waste money) vs. too little (poor performance).

**Over-Provisioning (2x needed capacity):**
- ✅ Handles unexpected traffic spikes
- ✅ Room for growth
- ❌ Higher costs
- ❌ Wasted resources

**Under-Provisioning (exact capacity):**
- ✅ Lower costs
- ✅ Efficient resource use
- ❌ Risk of outages during spikes
- ❌ No room for growth

**Sweet Spot:**
- Plan for 1.5x expected peak traffic
- Use auto-scaling for elasticity
- Monitor and adjust

---

## 3. Storage: Cost vs. Performance

### Trade-Off
Fast storage is expensive. Cheap storage is slow.

**Options:**

| Storage Type | Speed | Cost | Use Case |
|--------------|-------|------|----------|
| NVMe SSD | Fastest | Highest | Databases, hot data |
| Regular SSD | Fast | Medium | Application data |
| HDD | Slow | Low | Archives, backups |
| Cold Storage | Slowest | Lowest | Long-term archives |

**Example: 1 PB storage**
- All SSD: $100,000/month
- Tiered (10% SSD, 90% HDD): $20,000/month
- Cold storage: $4,000/month

**Strategy:** Use storage tiers based on access patterns (hot/warm/cold data).

---

## 4. Replication: Reliability vs. Cost

### Trade-Off
More copies = more reliable, but more expensive.

**Options:**
- **1 copy**: Cheap, but if it fails, data is lost
- **2 copies**: Basic redundancy (2x cost)
- **3 copies**: Industry standard (3x cost)
- **5+ copies**: Maximum reliability (5x+ cost)

**Common Strategy:**
- **Critical data** (payments, user accounts): 3+ copies, multiple regions
- **User content** (photos, videos): 2-3 copies
- **Temporary data** (caches, logs): 1 copy or none

---

## 5. Peak vs. Average Capacity Planning

### Trade-Off
Design for average traffic (cheaper) or peak traffic (better user experience).

**Design for Average:**
- Lower costs (smaller infrastructure)
- Poor performance during peaks
- Risk of outages

**Design for Peak:**
- Higher costs (larger infrastructure)
- Wasted capacity most of the time
- Better user experience

**Best Practice: Elastic Scaling**
```
Base capacity: 1.2x average
Auto-scale: Up to 3x average for peaks
Cost: Balanced
Experience: Good
```

---

## 6. Bandwidth: User Experience vs. Cost

### Trade-Off
High-quality content uses more bandwidth.

**Example: Video Streaming**

| Quality | Bitrate | User Experience | Bandwidth Cost |
|---------|---------|-----------------|----------------|
| SD | 1 Mbps | Basic | $X |
| HD | 3 Mbps | Good | $3X |
| Full HD | 5 Mbps | Great | $5X |
| 4K | 25 Mbps | Excellent | $25X |

**Strategy:**
- Adaptive streaming (adjust based on user's connection)
- Limit quality for free users
- Compress aggressively
- Use CDN to reduce costs

---

## 7. Real-Time vs. Batch Processing

### Trade-Off
Process data immediately (expensive) or in batches (cheaper but delayed).

**Real-Time Processing:**
- ✅ Immediate results
- ✅ Better user experience
- ❌ Higher server costs
- ❌ More complex infrastructure

**Batch Processing:**
- ✅ Lower costs (process overnight)
- ✅ Simpler infrastructure
- ❌ Delayed results
- ❌ Stale data

**When to Use:**
- **Real-time**: Payments, messaging, live updates
- **Batch**: Analytics, reports, recommendations, backups

---

## 8. Caching: Freshness vs. Performance

### Trade-Off
Cache longer (faster, might be stale) vs. cache shorter (slower, always fresh).

**Long Cache TTL (1 hour):**
- Fast (most requests hit cache)
- Users might see old data
- Lower database load

**Short Cache TTL (1 minute):**
- Slower (more cache misses)
- Users see fresher data
- Higher database load

**Strategy by Data Type:**
```
Static content (images, CSS): 1 day+
Product catalog: 5-15 minutes
Prices/inventory: 30-60 seconds
User session data: No cache (always fresh)
```

---

## 9. Redundancy: Reliability vs. Complexity

### Trade-Off
More redundancy = more reliable, but more complex to manage.

**Minimal Redundancy:**
- Single database, single region
- Simple to manage
- Risk of total failure

**High Redundancy:**
- Multiple databases, multiple regions
- Complex synchronization
- Survives datacenter failure

**Balanced Approach:**
```
Development: Minimal redundancy
Production: Database replicas + backups
Mission-Critical: Multi-region, active-active
```

---

## 10. Estimation Accuracy: Time vs. Value

### Trade-Off
Spend more time on estimation vs. start building faster.

**Detailed Estimation (Weeks):**
- Accurate capacity planning
- Better cost prediction
- Delays project start
- Risk of over-planning

**Quick Estimation (Days):**
- Start building sooner
- Learn from real data
- Adjust as you go
- Risk of costly mistakes

**Agile Approach:**
```
Phase 1: Quick estimate (80% confidence)
Phase 2: Build MVP
Phase 3: Measure actual usage
Phase 4: Refine estimates based on real data
Phase 5: Scale appropriately
```

---

## Real-World Example: Startup Journey

### Month 1-6: Under-Estimate is OK
- Quick estimates sufficient
- Learning phase
- Low traffic
- Easy to scale up

### Month 6-12: Need Better Estimates
- Growing pains
- Cost optimization matters
- Traffic patterns emerging
- Plan for 6 months ahead

### Year 2+: Detailed Capacity Planning
- Predictable patterns
- Cost is significant
- Detailed forecasting
- Annual capacity planning

---

## Decision Framework

When making capacity planning trade-offs, consider:

1. **Current Stage:**
   - Startup: Optimize for speed
   - Growth: Optimize for reliability
   - Mature: Optimize for cost

2. **Budget:**
   - Limited: Choose cheaper options, accept limitations
   - Flexible: Balance cost and performance
   - Unlimited: Optimize for user experience

3. **SLA (Service Level Agreement):**
   - 99%: Can afford downtime, choose cheaper options
   - 99.9%: Need redundancy
   - 99.99%+: Need multi-region, expensive

4. **Data Criticality:**
   - **Critical** (payments, accounts): Max redundancy
   - **Important** (user content): Good redundancy
   - **Nice-to-have** (logs, analytics): Minimal redundancy

---

## Summary

Key principles for capacity planning trade-offs:

✅ **Start with rough estimates** - Refine as you learn  
✅ **Over-provision initially** - Scale down as patterns emerge  
✅ **Use storage tiers** - Hot/warm/cold based on access  
✅ **Plan for peak, not average** - But use elastic scaling  
✅ **Cache aggressively** - Adjust TTL based on data type  
✅ **Choose redundancy by criticality** - Not everything needs 5 copies  
✅ **Monitor and adjust** - Real data beats estimates  

**Remember:** Perfect estimates are impossible. Good-enough estimates that you can act on quickly are better than perfect estimates that take too long.

## Next Steps

Now that you understand estimation trade-offs, move on to **[CAP Theorem](../02-cap-theorem/)** to learn about fundamental trade-offs in distributed systems!
