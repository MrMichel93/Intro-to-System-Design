# Consistency Pattern Trade-Offs

## 1. Strong vs. Eventual Consistency

**Strong:**
- Pros: Correct data always, simple reasoning
- Cons: Slower, less available, coordination overhead

**Eventual:**
- Pros: Fast, highly available, scalable
- Cons: Temporary inconsistencies, conflict resolution needed

## 2. Synchronous vs. Asynchronous Replication

**Synchronous:**
- Pros: No data loss, strong consistency
- Cons: Slower, availability depends on all nodes

**Asynchronous:**
- Pros: Fast, available even if replicas down
- Cons: Data loss possible, replication lag

## 3. Quorum Size

**Large Quorum (W=3, R=3 in N=5):**
- Pros: Strong guarantees
- Cons: Slower, needs more nodes available

**Small Quorum (W=1, R=1):**
- Pros: Fast
- Cons: Weak guarantees, stale reads possible

Choose based on data criticality!

See [trade-offs.md](./trade-offs.md) for detailed analysis.
