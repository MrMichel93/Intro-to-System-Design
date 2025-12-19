# Data Partitioning Trade-Offs

## 1. Horizontal Partitioning vs. Vertical Partitioning

**Horizontal (Sharding):**
- Pros: Better scalability, distribute load
- Cons: Cross-shard queries hard

**Vertical:**
- Pros: Separate hot/cold data
- Cons: Limited scalability

## 2. Range vs. Hash Partitioning

**Range:**
- Pros: Range queries easy, ordered data
- Cons: Hot spots possible

**Hash:**
- Pros: Even distribution, no hot spots
- Cons: Range queries require all shards

## 3. More Shards vs. Fewer Shards

**More Shards:**
- Pros: Finer granularity, easier rebalancing
- Cons: More operational complexity

**Fewer Shards:**
- Pros: Simpler management
- Cons: Harder to rebalance, coarser distribution

Choose based on current scale and growth rate!

For detailed analysis, see [trade-offs.md](./trade-offs.md).
