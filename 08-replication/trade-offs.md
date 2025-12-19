# Replication Trade-Offs

## 1. Synchronous vs. Asynchronous

**Synchronous:**
- Pros: No data loss, strong consistency
- Cons: Slower writes, availability issues

**Asynchronous:**
- Pros: Fast writes, high availability
- Cons: Data loss possible, eventual consistency

**Choose:** Sync for critical data, async for most apps.

## 2. More Replicas vs. Fewer Replicas

**More Replicas:**
- Pros: Better availability, more read capacity
- Cons: More cost, more replication overhead

**Fewer Replicas:**
- Pros: Lower cost, simpler
- Cons: Less fault tolerance

**Sweet spot:** 2-3 replicas for most cases.

## 3. Single Leader vs. Multi-Leader

**Single Leader:**
- Pros: Simpler, easier consistency
- Cons: Write bottleneck, single point of failure

**Multi-Leader:**
- Pros: Better write performance, geographic distribution
- Cons: Conflict resolution needed, complex

**Choose:** Single-leader for most, multi-leader for global apps.

See [trade-offs.md](./trade-offs.md) for detailed analysis.
