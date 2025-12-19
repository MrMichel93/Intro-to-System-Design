# Data Partitioning Examples

## Example 1: Social Media Platform (Instagram-like)

**Partition by user_id:**
- Users: 2 billion
- Shards: 4096
- Users per shard: ~500K

**Queries:**
- Get user profile: Single shard
- Get user's posts: Single shard
- Get user's followers: Single shard

**Win:** Most queries hit one shard!

## Example 2: E-commerce Orders

**Partition by order_date:**
- Shard 1: 2024-Q1
- Shard 2: 2024-Q2
- Shard 3: 2024-Q3
- Shard 4: 2024-Q4

**Win:** Recent orders (hot data) on separate shard, easy to archive old orders.

## Example 3: Multi-Tenant SaaS

**Partition by tenant_id:**
- Each customer gets data in specific shard
- Isolation between customers
- Easy billing and data export per tenant

For more examples, see the [README](./README.md).
