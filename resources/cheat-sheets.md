# System Design Cheat Sheets

## Core Concepts

### Scalability
- **Vertical**: Upgrade single machine (limited)
- **Horizontal**: Add more machines (scalable)
- **Key**: Design for horizontal scaling

### CAP Theorem
- **Can only have 2 of 3**: Consistency, Availability, Partition Tolerance
- **In practice**: Choose C or A (P is mandatory in distributed systems)
- **CP**: Banks, inventory
- **AP**: Social media, caching

### Load Balancing Algorithms
- **Round Robin**: Equal distribution
- **Least Connections**: Send to least busy
- **IP Hash**: Same client → Same server
- **Weighted**: Based on server capacity

### Caching Strategies
- **Cache-Aside**: App checks cache, loads from DB if miss
- **Write-Through**: Write to cache and DB simultaneously
- **Write-Behind**: Write to cache, async write to DB
- **TTL**: Time To Live for cache entries

### Database Types
- **SQL**: ACID, relations, structured
- **NoSQL Document**: Flexible schema (MongoDB)
- **NoSQL Wide-Column**: High write throughput (Cassandra)
- **NoSQL Key-Value**: Simple, fast (Redis)

### Replication
- **Leader-Follower**: Writes to leader, reads from followers
- **Multi-Leader**: Multiple write nodes, conflict resolution
- **Leaderless**: All nodes equal, quorum-based

### Sharding
- **Range**: By value ranges (e.g., A-M, N-Z)
- **Hash**: Hash function determines shard
- **Directory**: Lookup table for shard mapping

## Quick Decision Trees

### Choose Database
```
Need ACID transactions? → SQL
Flexible schema? → MongoDB
High write throughput? → Cassandra
Simple key-value? → Redis
Full-text search? → Elasticsearch
```

### Choose Consistency
```
Financial data? → Strong Consistency (CP)
Social feed? → Eventual Consistency (AP)
Inventory? → Strong Consistency (CP)
Caching? → Eventual Consistency (AP)
```

### Choose Replication
```
Read-heavy? → Leader-follower with read replicas
Multi-region? → Multi-leader or leaderless
Critical data? → Synchronous replication
High availability? → Asynchronous replication
```

## Common Numbers

### Request Rates
- Low: 10/sec
- Medium: 1,000/sec
- High: 100,000/sec
- Extreme: 1,000,000/sec

### Storage Sizes
- Tweet: 280 bytes
- Email: 10 KB
- Photo: 2 MB
- Video (HD, 1 hour): 2 GB

### Latency
- Memory: 100 ns
- SSD: 1 ms
- HDD: 10 ms
- Network (datacenter): 500 μs
- Network (cross-continent): 150 ms

## Technology Stack Templates

### Small App (< 10K users)
- Server: Single EC2/VM
- Database: Single PostgreSQL
- Cache: Redis
- Storage: S3

### Medium App (10K - 1M users)
- Servers: Load balancer + 3-5 app servers
- Database: Primary + read replicas
- Cache: Redis cluster
- Queue: RabbitMQ
- Storage: S3 + CDN

### Large App (1M+ users)
- Servers: Auto-scaling groups, multiple regions
- Database: Sharded + replicated
- Cache: Distributed Redis/Memcached
- Queue: Kafka
- Search: Elasticsearch
- Storage: S3 + Global CDN
- Monitoring: Prometheus, Grafana

## Interview Checklist

✅ Clarify requirements  
✅ Define scale (users, requests, data)  
✅ Calculate capacity (storage, bandwidth)  
✅ Design high-level architecture  
✅ Define APIs  
✅ Design database schema  
✅ Identify bottlenecks  
✅ Discuss trade-offs  
✅ Consider scaling strategies  
✅ Address failure scenarios  

## Common Patterns

### API Gateway
- Single entry point
- Auth, rate limiting, routing
- Used by: Netflix, Amazon

### Circuit Breaker
- Prevent cascading failures
- Open/Closed/Half-Open states
- Fail fast when service down

### CQRS (Command Query Responsibility Segregation)
- Separate read and write models
- Optimize each independently
- Complex but powerful

### Event Sourcing
- Store events, not state
- Rebuild state from events
- Audit trail included

### Service Mesh
- Service-to-service communication
- Load balancing, retries, monitoring
- Example: Istio, Linkerd

## Red Flags to Avoid

❌ Not asking clarifying questions  
❌ Jumping to detailed design too quickly  
❌ Ignoring scale numbers  
❌ Not considering failure scenarios  
❌ Over-engineering simple problems  
❌ Not explaining trade-offs  
❌ Poor time management  
❌ Not validating design with requirements  

## Pro Tips

💡 Start high-level, then drill down  
💡 Draw diagrams (boxes and arrows)  
💡 Think out loud  
💡 Consider cost implications  
💡 Know when to use which database  
💡 Understand CAP theorem deeply  
💡 Practice common problems  
💡 Stay calm and methodical  

Use this as a quick reference during interviews and design sessions!
