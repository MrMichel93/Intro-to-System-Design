# System Design Resources

Additional learning materials, cheat sheets, and references.

## Quick Reference

### [Cheat Sheets](./cheat-sheets.md)
- Key concepts summary
- Common patterns
- Technology choices
- Capacity planning formulas

### [Glossary](./glossary.md)
- 145+ system design terms with definitions
- Cross-referenced to course modules
- Organized alphabetically
- Quick reference by module

### [Numbers to Know](./numbers-to-know.md)
- Latency numbers for various operations
- Storage capacities and data sizes
- Network bandwidth benchmarks
- QPS (Queries Per Second) benchmarks
- Cloud cost estimates
- Availability calculations
- Practical estimation examples

### [Architecture Patterns](./architecture-patterns.md)
- 26+ common system design patterns
- When to use each pattern
- Trade-offs and considerations
- Real-world examples
- Pattern combination strategies

### [Reading List](./reading-list.md)
- Essential and advanced books
- Technical papers and research
- Engineering blogs (Netflix, Uber, etc.)
- Video series and courses
- Podcasts and communities
- Learning path recommendations

### [Tools Guide](./tools-guide.md)
- Diagramming tools (Excalidraw, draw.io, LucidChart, etc.)
- Calculation and estimation tools
- Practice platforms for interviews
- Database and development tools
- Monitoring and observability tools
- Load testing tools
- Cloud platforms comparison

## Estimation Cheat Sheet

### Powers of 2
```
1 KB = 1,000 bytes ≈ 10^3
1 MB = 1,000,000 bytes ≈ 10^6
1 GB = 1,000,000,000 bytes ≈ 10^9
1 TB = 1,000,000,000,000 bytes ≈ 10^12
1 PB = 1,000 TB ≈ 10^15
```

### Time Units
```
1 day = 86,400 seconds ≈ 100,000 seconds
1 month ≈ 2.5 million seconds
1 year ≈ 31.5 million seconds
```

### Latency Numbers
```
RAM access: 100 ns
SSD read: 1 ms
Network within datacenter: 500 μs
Network cross-continent: 150 ms
```

### Availability
```
99% = 3.65 days downtime/year
99.9% = 8.76 hours downtime/year
99.99% = 52.6 minutes downtime/year
99.999% = 5.26 minutes downtime/year
```

## Common Patterns Quick Reference

### Caching
- **When**: Read-heavy workloads
- **Technologies**: Redis, Memcached
- **Pattern**: Cache-aside, write-through

### Load Balancing
- **When**: Need to distribute traffic
- **Algorithms**: Round-robin, least connections
- **Technologies**: Nginx, HAProxy, AWS ELB

### Database Replication
- **When**: High read traffic, availability
- **Types**: Leader-follower, multi-leader
- **Trade-off**: Consistency vs availability

### Sharding
- **When**: Single database can't handle load
- **Methods**: Range, hash, list
- **Challenge**: Cross-shard queries

### Message Queues
- **When**: Async processing, decoupling
- **Technologies**: Kafka, RabbitMQ, SQS
- **Pattern**: Producer-consumer

### CDN
- **When**: Static content, global users
- **Technologies**: CloudFront, Cloudflare
- **Benefit**: Reduce latency, origin load

## Technology Comparison

### Databases
| Type | Example | Use Case |
|------|---------|----------|
| SQL | PostgreSQL, MySQL | Structured data, ACID needed |
| NoSQL Document | MongoDB | Flexible schema, hierarchical data |
| NoSQL Wide-Column | Cassandra, HBase | Time-series, high write throughput |
| NoSQL Key-Value | Redis, DynamoDB | Simple lookups, caching |
| Search | Elasticsearch | Full-text search, analytics |

### Cache
| Technology | Speed | Persistence | Use Case |
|------------|-------|-------------|----------|
| Redis | Very fast | Optional | Cache, pub/sub, queues |
| Memcached | Fastest | No | Pure caching |

### Message Queue
| Technology | Throughput | Persistence | Use Case |
|------------|------------|-------------|----------|
| Kafka | Highest | Yes | Event streaming, logs |
| RabbitMQ | High | Yes | Task queues, RPC |
| SQS | Medium | Yes | AWS integration, simple queues |

## Learning Path

1. Start with [Foundations](../00-foundations/)
2. Study [Back-of-Envelope](../01-back-of-envelope/)
3. Understand [CAP Theorem](../02-cap-theorem/)
4. Learn core concepts (Modules 3-6)
5. Deep dive into advanced topics (Modules 7-13)
6. Study [Case Studies](../case-studies/)
7. Practice with [Design Templates](../design-templates/)
8. Prepare for [Interviews](../interview-prep/)

## External Resources

### Books
- "Designing Data-Intensive Applications" by Martin Kleppmann
- "System Design Interview" by Alex Xu
- "Building Microservices" by Sam Newman

### Websites
- High Scalability Blog
- Martin Fowler's Blog
- Engineering blogs (Netflix, Uber, etc.)

### Courses
- Grokking the System Design Interview
- System Design Primer (GitHub)

## Community

- Join discussions on system design forums
- Follow tech company engineering blogs
- Attend meetups and conferences
- Practice with peers

Happy learning!
