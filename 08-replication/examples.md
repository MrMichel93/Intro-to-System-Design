# Replication Examples

## Example 1: PostgreSQL Leader-Follower

Setup: 1 leader + 3 followers
- Writes: Leader only
- Reads: Any replica
- Replication: Async (seconds lag)

Perfect for: Read-heavy web apps

## Example 2: Cassandra Leaderless

Setup: 5-node cluster, RF=3
- Write: 3 nodes
- Read: 2 nodes (quorum)
- No single leader

Perfect for: High availability, multi-datacenter

## Example 3: MySQL with Semi-Sync

Setup: 1 primary + 2 replicas
- Write waits for 1 replica
- Faster than full sync
- Safer than async

Perfect for: E-commerce, important data

See [README](./README.md) for detailed examples.
