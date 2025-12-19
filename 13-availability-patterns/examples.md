# Availability Pattern Examples

## Example 1: Database Failover

**Pattern:** Active-Passive with automatic failover
- Primary database serves traffic
- Replica synced in real-time
- Heartbeat every 5 seconds
- Failover in 30 seconds if primary down

## Example 2: API Gateway with Circuit Breaker

**Pattern:** Circuit breaker prevents cascading failures
- Try calling payment service
- If 5 failures in 30s: Open circuit
- Fail fast for 60s
- Half-open: Test if recovered

## Example 3: Multi-Region E-commerce

**Pattern:** Active-Active multi-region
- US and EU regions both active
- Users route to nearest
- Cross-region replication
- If one region down, other handles all

See [README](./README.md) for more examples.
