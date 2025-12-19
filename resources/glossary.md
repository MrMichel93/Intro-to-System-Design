# System Design Glossary

A comprehensive reference of system design terminology. Terms are organized alphabetically and cross-referenced to relevant course modules.

## A

### API (Application Programming Interface)
The interface that allows different software applications to communicate with each other. Defines the methods and data structures for interaction.
- **Module Reference**: [Module 10: API Design](../10-api-design/)
- **Related Terms**: REST, GraphQL, RPC

### API Gateway
A server that acts as a single entry point for all client requests, routing them to appropriate backend services. Handles cross-cutting concerns like authentication, rate limiting, and logging.
- **Module Reference**: [Module 12: Microservices](../12-microservices/)
- **Related Terms**: Load Balancer, Reverse Proxy

### Asynchronous Processing
Processing tasks without blocking the caller, allowing the system to handle other operations while waiting for completion.
- **Module Reference**: [Module 11: Message Queues](../11-message-queues/)
- **Related Terms**: Message Queue, Event-Driven Architecture

### Availability
The percentage of time a system is operational and accessible. Often expressed as "nines" (e.g., 99.9% = "three nines").
- **Module Reference**: [Module 13: Availability Patterns](../13-availability-patterns/)
- **Related Terms**: Uptime, SLA, Reliability

### Availability Zone (AZ)
Isolated locations within a data center region. Used to build highly available and fault-tolerant applications.
- **Module Reference**: [Module 13: Availability Patterns](../13-availability-patterns/)
- **Related Terms**: Region, Multi-AZ Deployment

## B

### Back Pressure
A mechanism to prevent overwhelming downstream systems by slowing or stopping upstream data flow when the system can't keep up.
- **Module Reference**: [Module 11: Message Queues](../11-message-queues/)
- **Related Terms**: Flow Control, Rate Limiting

### Back-of-Envelope Calculation
Quick, approximate calculations used to estimate system capacity, storage, bandwidth, and other requirements.
- **Module Reference**: [Module 1: Back-of-Envelope Calculations](../01-back-of-envelope/)
- **Related Terms**: Capacity Planning, Estimation

### Batch Processing
Processing large amounts of data in groups at scheduled intervals rather than immediately as it arrives.
- **Module Reference**: [Module 11: Message Queues](../11-message-queues/)
- **Related Terms**: Stream Processing, ETL

### Bloom Filter
A space-efficient probabilistic data structure that tests whether an element is a member of a set. May produce false positives but never false negatives.
- **Module Reference**: [Module 5: Caching](../05-caching/)
- **Related Terms**: Probabilistic Data Structure, Cache

### Byzantine Fault
A fault where components may fail in arbitrary ways, including providing conflicting information to different parts of the system.
- **Module Reference**: [Module 2: CAP Theorem](../02-cap-theorem/)
- **Related Terms**: Fault Tolerance, Consensus

## C

### Cache
A high-speed storage layer that stores frequently accessed data to reduce latency and load on backend systems.
- **Module Reference**: [Module 5: Caching](../05-caching/)
- **Related Terms**: Redis, Memcached, Cache Hit

### Cache Hit
When requested data is found in the cache, avoiding a slower backend query.
- **Module Reference**: [Module 5: Caching](../05-caching/)
- **Related Terms**: Cache Miss, Hit Ratio

### Cache Miss
When requested data is not found in the cache, requiring a query to the backend system.
- **Module Reference**: [Module 5: Caching](../05-caching/)
- **Related Terms**: Cache Hit, Cache Eviction

### Cache-Aside Pattern
A caching strategy where the application code manages the cache. On read: check cache first, then database if miss. On write: update database, then invalidate cache.
- **Module Reference**: [Module 5: Caching](../05-caching/)
- **Related Terms**: Lazy Loading, Read-Through Cache

### CAP Theorem
States that a distributed system can provide at most two of three guarantees: Consistency, Availability, and Partition Tolerance.
- **Module Reference**: [Module 2: CAP Theorem](../02-cap-theorem/)
- **Related Terms**: Consistency, Availability, Partition Tolerance

### CDN (Content Delivery Network)
A geographically distributed network of servers that delivers content to users from the nearest location, reducing latency.
- **Module Reference**: [Module 14: CDN](../14-cdn/)
- **Related Terms**: Edge Server, Cache, Origin Server

### Circuit Breaker
A design pattern that prevents cascading failures by detecting failures and stopping requests to a failing service temporarily.
- **Module Reference**: [Module 13: Availability Patterns](../13-availability-patterns/)
- **Related Terms**: Fault Tolerance, Fallback

### Cold Start
The initial period when a system or service starts up and hasn't yet cached data or warmed up connections.
- **Module Reference**: [Module 5: Caching](../05-caching/)
- **Related Terms**: Warm Cache, Cache Warming

### Consistent Hashing
A hashing technique that minimizes reorganization when nodes are added or removed, commonly used in distributed caching and load balancing.
- **Module Reference**: [Module 4: Load Balancing](../04-load-balancing/), [Module 7: Data Partitioning](../07-data-partitioning/)
- **Related Terms**: Hash Ring, Distributed Hash Table

### Consistency
A guarantee that all nodes in a distributed system see the same data at the same time.
- **Module Reference**: [Module 2: CAP Theorem](../02-cap-theorem/), [Module 9: Consistency Patterns](../09-consistency-patterns/)
- **Related Terms**: Strong Consistency, Eventual Consistency

### CQRS (Command Query Responsibility Segregation)
A pattern that separates read and write operations into different models, optimizing each for its specific use case.
- **Module Reference**: [Module 6: Database Design](../06-database-design/)
- **Related Terms**: Event Sourcing, Read Model, Write Model

## D

### Data Partitioning
Splitting large datasets across multiple databases or servers to improve performance and scalability.
- **Module Reference**: [Module 7: Data Partitioning](../07-data-partitioning/)
- **Related Terms**: Sharding, Horizontal Partitioning

### Database Index
A data structure that improves the speed of data retrieval operations at the cost of additional storage and slower writes.
- **Module Reference**: [Module 6: Database Design](../06-database-design/)
- **Related Terms**: B-Tree, Primary Key, Secondary Index

### Database Replication
Creating copies of a database across multiple servers to improve availability, fault tolerance, and read performance.
- **Module Reference**: [Module 8: Replication](../08-replication/)
- **Related Terms**: Leader-Follower, Multi-Leader, Primary-Replica

### Deadlock
A situation where two or more processes are unable to proceed because each is waiting for the other to release a resource.
- **Module Reference**: [Module 6: Database Design](../06-database-design/)
- **Related Terms**: Locking, Transaction

### Denormalization
The process of adding redundant data to a database to improve read performance at the cost of write performance and storage.
- **Module Reference**: [Module 6: Database Design](../06-database-design/)
- **Related Terms**: Normalization, Data Redundancy

### Distributed System
A system whose components are located on different networked computers that communicate and coordinate their actions.
- **Module Reference**: [Module 2: CAP Theorem](../02-cap-theorem/)
- **Related Terms**: Distributed Computing, Node, Cluster

### Distributed Tracing
A method of tracking requests as they flow through multiple services in a distributed system.
- **Module Reference**: [Module 12: Microservices](../12-microservices/)
- **Related Terms**: Observability, Monitoring, Jaeger

### DNS (Domain Name System)
A hierarchical naming system that translates human-readable domain names into IP addresses.
- **Module Reference**: [Module 0: Foundations](../00-foundations/)
- **Related Terms**: DNS Resolution, Name Server

### Docker
A platform for developing, shipping, and running applications in containers.
- **Module Reference**: [Module 12: Microservices](../12-microservices/)
- **Related Terms**: Container, Kubernetes, Containerization

## E

### Edge Computing
Processing data near the source of data generation rather than in a centralized data center.
- **Module Reference**: [Module 14: CDN](../14-cdn/)
- **Related Terms**: Edge Server, CDN

### Elastic Scaling
The ability to automatically add or remove resources based on current demand.
- **Module Reference**: [Module 3: Scalability](../03-scalability/)
- **Related Terms**: Auto-Scaling, Horizontal Scaling

### ETL (Extract, Transform, Load)
A data integration process that extracts data from sources, transforms it into a usable format, and loads it into a target system.
- **Module Reference**: [Module 6: Database Design](../06-database-design/)
- **Related Terms**: Data Pipeline, Batch Processing

### Event-Driven Architecture
An architecture pattern where system components communicate through events, enabling loose coupling and asynchronous processing.
- **Module Reference**: [Module 11: Message Queues](../11-message-queues/)
- **Related Terms**: Event Bus, Pub/Sub, Event Sourcing

### Event Sourcing
A pattern where state changes are stored as a sequence of events rather than just the current state.
- **Module Reference**: [Module 9: Consistency Patterns](../09-consistency-patterns/)
- **Related Terms**: CQRS, Event Log, Audit Trail

### Eventually Consistent
A consistency model where updates to data will eventually propagate to all nodes, but there may be a delay.
- **Module Reference**: [Module 2: CAP Theorem](../02-cap-theorem/), [Module 9: Consistency Patterns](../09-consistency-patterns/)
- **Related Terms**: Weak Consistency, Strong Consistency

## F

### Failover
The process of automatically switching to a redundant or standby system when the primary system fails.
- **Module Reference**: [Module 13: Availability Patterns](../13-availability-patterns/)
- **Related Terms**: High Availability, Redundancy, Active-Passive

### Fan-Out
A pattern where a single message or event is distributed to multiple consumers or destinations.
- **Module Reference**: [Module 11: Message Queues](../11-message-queues/)
- **Related Terms**: Pub/Sub, Broadcast, Message Queue

### Fault Tolerance
The ability of a system to continue operating properly in the event of component failures.
- **Module Reference**: [Module 13: Availability Patterns](../13-availability-patterns/)
- **Related Terms**: Resilience, Redundancy, High Availability

### Federation
Splitting databases by function, where each database handles a specific business domain.
- **Module Reference**: [Module 6: Database Design](../06-database-design/)
- **Related Terms**: Functional Partitioning, Microservices

### Forward Proxy
A server that sits between clients and the internet, forwarding client requests to external servers.
- **Module Reference**: [Module 4: Load Balancing](../04-load-balancing/)
- **Related Terms**: Reverse Proxy, Proxy Server

## G

### Gossip Protocol
A peer-to-peer communication protocol where nodes periodically exchange information about their state with other nodes.
- **Module Reference**: [Module 8: Replication](../08-replication/)
- **Related Terms**: Eventual Consistency, Distributed System

### GraphQL
A query language for APIs that allows clients to request exactly the data they need.
- **Module Reference**: [Module 10: API Design](../10-api-design/)
- **Related Terms**: REST, API, Query Language

### gRPC
A high-performance, open-source RPC framework that uses HTTP/2 and Protocol Buffers.
- **Module Reference**: [Module 10: API Design](../10-api-design/)
- **Related Terms**: RPC, Protocol Buffers, REST

## H

### Hash Function
A function that maps data of arbitrary size to fixed-size values, used in hashing, caching, and partitioning.
- **Module Reference**: [Module 7: Data Partitioning](../07-data-partitioning/)
- **Related Terms**: Consistent Hashing, Hash Table

### Health Check
A mechanism to determine if a service or component is functioning properly, often used by load balancers.
- **Module Reference**: [Module 4: Load Balancing](../04-load-balancing/), [Module 13: Availability Patterns](../13-availability-patterns/)
- **Related Terms**: Heartbeat, Monitoring

### Heartbeat
Regular signals sent by a system to indicate it's alive and functioning properly.
- **Module Reference**: [Module 13: Availability Patterns](../13-availability-patterns/)
- **Related Terms**: Health Check, Keepalive

### Horizontal Scaling (Scale-Out)
Adding more machines or nodes to a system to handle increased load.
- **Module Reference**: [Module 3: Scalability](../03-scalability/)
- **Related Terms**: Vertical Scaling, Distributed System

### Hot Spot
A situation where a disproportionate amount of traffic or load is directed to a specific node or partition.
- **Module Reference**: [Module 7: Data Partitioning](../07-data-partitioning/)
- **Related Terms**: Load Imbalance, Sharding

## I

### Idempotency
A property where performing an operation multiple times has the same effect as performing it once.
- **Module Reference**: [Module 10: API Design](../10-api-design/)
- **Related Terms**: Retry Logic, HTTP Methods

### Immutable Infrastructure
Infrastructure that is replaced rather than modified, ensuring consistency and reducing configuration drift.
- **Module Reference**: [Module 12: Microservices](../12-microservices/)
- **Related Terms**: Containers, Infrastructure as Code

### Index
See **Database Index**

### In-Memory Database
A database that stores data primarily in main memory (RAM) rather than on disk for faster access.
- **Module Reference**: [Module 5: Caching](../05-caching/)
- **Related Terms**: Redis, Cache, RAM

## J

### Jitter
Random variation added to retry intervals or request timings to prevent synchronized behavior that could overwhelm systems.
- **Module Reference**: [Module 13: Availability Patterns](../13-availability-patterns/)
- **Related Terms**: Exponential Backoff, Retry Logic

## K

### Kafka
A distributed streaming platform used for building real-time data pipelines and streaming applications.
- **Module Reference**: [Module 11: Message Queues](../11-message-queues/)
- **Related Terms**: Message Queue, Event Streaming, Pub/Sub

### Kubernetes
An open-source container orchestration platform for automating deployment, scaling, and management of containerized applications.
- **Module Reference**: [Module 12: Microservices](../12-microservices/)
- **Related Terms**: Container Orchestration, Docker, Service Mesh

## L

### Latency
The time delay between a request and its response. Lower latency means faster response times.
- **Module Reference**: [Module 1: Back-of-Envelope Calculations](../01-back-of-envelope/)
- **Related Terms**: Response Time, Throughput, Performance

### Leader-Follower Replication
A replication pattern where one node (leader) handles writes and propagates changes to follower nodes that handle reads.
- **Module Reference**: [Module 8: Replication](../08-replication/)
- **Related Terms**: Primary-Replica, Master-Slave, Replication

### Load Balancer
A device or software that distributes incoming network traffic across multiple servers to ensure no single server is overwhelmed.
- **Module Reference**: [Module 4: Load Balancing](../04-load-balancing/)
- **Related Terms**: Round Robin, Least Connections, Reverse Proxy

### Load Shedding
Intentionally dropping or rejecting requests when a system is overloaded to maintain stability for remaining requests.
- **Module Reference**: [Module 13: Availability Patterns](../13-availability-patterns/)
- **Related Terms**: Circuit Breaker, Rate Limiting

### LRU (Least Recently Used)
A cache eviction policy that removes the least recently accessed items when the cache is full.
- **Module Reference**: [Module 5: Caching](../05-caching/)
- **Related Terms**: Cache Eviction, LFU, FIFO

## M

### Memcached
A high-performance, distributed memory caching system used to speed up dynamic web applications.
- **Module Reference**: [Module 5: Caching](../05-caching/)
- **Related Terms**: Cache, Redis, In-Memory Database

### Message Queue
A component that facilitates asynchronous communication between services by storing messages until they can be processed.
- **Module Reference**: [Module 11: Message Queues](../11-message-queues/)
- **Related Terms**: Pub/Sub, Kafka, RabbitMQ, Asynchronous Processing

### Microservices
An architectural style where an application is composed of small, independent services that communicate over well-defined APIs.
- **Module Reference**: [Module 12: Microservices](../12-microservices/)
- **Related Terms**: Service-Oriented Architecture, API Gateway, Distributed System

### Monitoring
Continuously observing and collecting metrics about system health, performance, and behavior.
- **Module Reference**: [Module 13: Availability Patterns](../13-availability-patterns/)
- **Related Terms**: Observability, Logging, Alerting

### Multi-Leader Replication
A replication pattern where multiple nodes can accept writes and propagate changes to other nodes.
- **Module Reference**: [Module 8: Replication](../08-replication/)
- **Related Terms**: Active-Active, Conflict Resolution, Replication

## N

### Network Partition
A scenario where network failures divide a distributed system into isolated groups that cannot communicate.
- **Module Reference**: [Module 2: CAP Theorem](../02-cap-theorem/)
- **Related Terms**: Split Brain, Partition Tolerance, CAP Theorem

### NoSQL
A class of database management systems that don't use traditional relational database structures, often optimized for specific use cases.
- **Module Reference**: [Module 6: Database Design](../06-database-design/)
- **Related Terms**: Document Store, Key-Value Store, Wide-Column Store

### Normalization
The process of organizing database tables to reduce redundancy and improve data integrity.
- **Module Reference**: [Module 6: Database Design](../06-database-design/)
- **Related Terms**: Denormalization, Database Schema

## O

### Object Storage
A storage architecture that manages data as objects rather than files or blocks, typically used for unstructured data.
- **Module Reference**: [Module 6: Database Design](../06-database-design/)
- **Related Terms**: S3, Blob Storage, Cloud Storage

### Observability
The ability to understand the internal state of a system by examining its outputs (logs, metrics, traces).
- **Module Reference**: [Module 12: Microservices](../12-microservices/)
- **Related Terms**: Monitoring, Logging, Distributed Tracing

### Origin Server
The original source server that hosts the actual content, often protected by a CDN.
- **Module Reference**: [Module 14: CDN](../14-cdn/)
- **Related Terms**: CDN, Edge Server, Cache

## P

### Partition Tolerance
The ability of a distributed system to continue operating despite network partitions.
- **Module Reference**: [Module 2: CAP Theorem](../02-cap-theorem/)
- **Related Terms**: CAP Theorem, Network Partition, Distributed System

### Peer-to-Peer (P2P)
A distributed architecture where each node acts as both client and server, sharing resources directly with other nodes.
- **Module Reference**: [Module 8: Replication](../08-replication/)
- **Related Terms**: Decentralized, Distributed System

### Point of Presence (PoP)
A physical location where CDN or network providers have equipment to serve content to users.
- **Module Reference**: [Module 14: CDN](../14-cdn/)
- **Related Terms**: Edge Location, CDN

### Proxy
An intermediary server that sits between clients and servers, forwarding requests and responses.
- **Module Reference**: [Module 4: Load Balancing](../04-load-balancing/)
- **Related Terms**: Forward Proxy, Reverse Proxy, Load Balancer

### Pub/Sub (Publish-Subscribe)
A messaging pattern where publishers send messages to topics, and subscribers receive messages from topics they're interested in.
- **Module Reference**: [Module 11: Message Queues](../11-message-queues/)
- **Related Terms**: Message Queue, Event-Driven, Fan-Out

## Q

### QPS (Queries Per Second)
A measure of the number of requests or queries a system can handle per second.
- **Module Reference**: [Module 1: Back-of-Envelope Calculations](../01-back-of-envelope/)
- **Related Terms**: Throughput, TPS, RPS

### Quorum
The minimum number of nodes that must agree on a value before it's considered committed in a distributed system.
- **Module Reference**: [Module 9: Consistency Patterns](../09-consistency-patterns/)
- **Related Terms**: Consensus, Distributed Consensus, Replication

### Queue
A data structure that follows First-In-First-Out (FIFO) ordering, commonly used for task processing and message passing.
- **Module Reference**: [Module 11: Message Queues](../11-message-queues/)
- **Related Terms**: Message Queue, FIFO, Task Queue

## R

### RAID (Redundant Array of Independent Disks)
A technology that combines multiple disk drives for data redundancy and/or performance improvement.
- **Module Reference**: [Module 8: Replication](../08-replication/)
- **Related Terms**: Redundancy, Data Storage

### Rate Limiting
Controlling the number of requests a user or service can make in a given time period to prevent abuse and ensure fair resource allocation.
- **Module Reference**: [Module 10: API Design](../10-api-design/)
- **Related Terms**: Throttling, Token Bucket, API Gateway

### Read-Through Cache
A caching pattern where the cache sits between the application and database, handling cache misses by fetching from the database.
- **Module Reference**: [Module 5: Caching](../05-caching/)
- **Related Terms**: Cache-Aside, Write-Through Cache

### Redis
An in-memory data structure store used as a database, cache, and message broker.
- **Module Reference**: [Module 5: Caching](../05-caching/)
- **Related Terms**: Cache, In-Memory Database, Memcached

### Redundancy
Having backup components or data copies to improve system reliability and availability.
- **Module Reference**: [Module 13: Availability Patterns](../13-availability-patterns/)
- **Related Terms**: Replication, High Availability, Fault Tolerance

### Replica
A copy of data or a service instance that provides redundancy and improves availability.
- **Module Reference**: [Module 8: Replication](../08-replication/)
- **Related Terms**: Replication, Leader-Follower, Primary-Replica

### Replication
Creating and maintaining copies of data across multiple nodes for redundancy and performance.
- **Module Reference**: [Module 8: Replication](../08-replication/)
- **Related Terms**: Leader-Follower, Multi-Leader, Replica

### Replication Lag
The delay between when data is written to the primary node and when it appears on replica nodes.
- **Module Reference**: [Module 8: Replication](../08-replication/)
- **Related Terms**: Eventually Consistent, Asynchronous Replication

### REST (Representational State Transfer)
An architectural style for designing networked applications using HTTP methods and stateless communication.
- **Module Reference**: [Module 10: API Design](../10-api-design/)
- **Related Terms**: API, HTTP, Stateless

### Reverse Proxy
A server that sits in front of web servers, forwarding client requests to backend servers and returning responses to clients.
- **Module Reference**: [Module 4: Load Balancing](../04-load-balancing/)
- **Related Terms**: Load Balancer, Forward Proxy, Nginx

### RPC (Remote Procedure Call)
A protocol that allows a program to execute procedures on a remote server as if they were local calls.
- **Module Reference**: [Module 10: API Design](../10-api-design/)
- **Related Terms**: gRPC, API, Client-Server

## S

### Saga Pattern
A pattern for managing distributed transactions by breaking them into a sequence of local transactions with compensating actions.
- **Module Reference**: [Module 12: Microservices](../12-microservices/)
- **Related Terms**: Distributed Transactions, Choreography, Orchestration

### Scalability
The ability of a system to handle increased load by adding resources.
- **Module Reference**: [Module 3: Scalability](../03-scalability/)
- **Related Terms**: Horizontal Scaling, Vertical Scaling, Performance

### Service Discovery
A mechanism for services to find and communicate with each other in a dynamic, distributed environment.
- **Module Reference**: [Module 12: Microservices](../12-microservices/)
- **Related Terms**: Microservices, Service Registry, DNS

### Service Mesh
An infrastructure layer that handles service-to-service communication, providing features like load balancing, service discovery, and observability.
- **Module Reference**: [Module 12: Microservices](../12-microservices/)
- **Related Terms**: Istio, Microservices, Sidecar Pattern

### Session Store
A storage mechanism for maintaining user session data across multiple requests or servers.
- **Module Reference**: [Module 5: Caching](../05-caching/)
- **Related Terms**: Cache, Redis, Stateless

### Sharding
A type of horizontal partitioning that distributes data across multiple databases or tables.
- **Module Reference**: [Module 7: Data Partitioning](../07-data-partitioning/)
- **Related Terms**: Data Partitioning, Horizontal Scaling, Shard Key

### Shard Key
The attribute used to determine how data is distributed across shards in a partitioned database.
- **Module Reference**: [Module 7: Data Partitioning](../07-data-partitioning/)
- **Related Terms**: Sharding, Partition Key, Hash Key

### Single Point of Failure (SPOF)
A component whose failure would cause the entire system to fail.
- **Module Reference**: [Module 13: Availability Patterns](../13-availability-patterns/)
- **Related Terms**: High Availability, Redundancy, Fault Tolerance

### SLA (Service Level Agreement)
A commitment between a service provider and client defining expected service levels, including availability and performance.
- **Module Reference**: [Module 13: Availability Patterns](../13-availability-patterns/)
- **Related Terms**: SLO, SLI, Availability

### SLI (Service Level Indicator)
A quantitative measure of a service's behavior, such as latency, error rate, or availability.
- **Module Reference**: [Module 13: Availability Patterns](../13-availability-patterns/)
- **Related Terms**: SLO, SLA, Metrics

### SLO (Service Level Objective)
A target value or range for a service level indicator.
- **Module Reference**: [Module 13: Availability Patterns](../13-availability-patterns/)
- **Related Terms**: SLA, SLI, Target

### Split Brain
A condition in a distributed system where two or more parts believe they are the authoritative copy, potentially leading to data inconsistency.
- **Module Reference**: [Module 2: CAP Theorem](../02-cap-theorem/)
- **Related Terms**: Network Partition, Consensus, Quorum

### SQL (Structured Query Language)
A standard language for managing and querying relational databases.
- **Module Reference**: [Module 6: Database Design](../06-database-design/)
- **Related Terms**: Relational Database, RDBMS, Query

### Stateless
A design where each request contains all information needed to process it, with no stored session state on the server.
- **Module Reference**: [Module 10: API Design](../10-api-design/)
- **Related Terms**: REST, Horizontal Scaling, Session Store

### Sticky Session (Session Affinity)
A load balancing technique that routes requests from the same client to the same server.
- **Module Reference**: [Module 4: Load Balancing](../04-load-balancing/)
- **Related Terms**: Load Balancer, Session Management

### Strong Consistency
A consistency model where all reads receive the most recent write immediately.
- **Module Reference**: [Module 9: Consistency Patterns](../09-consistency-patterns/)
- **Related Terms**: ACID, Consistency, CAP Theorem

### Synchronous Communication
Communication where the caller waits for the response before proceeding with execution.
- **Module Reference**: [Module 11: Message Queues](../11-message-queues/)
- **Related Terms**: Asynchronous Communication, Blocking, RPC

## T

### Throughput
The number of operations or amount of data processed in a given time period.
- **Module Reference**: [Module 1: Back-of-Envelope Calculations](../01-back-of-envelope/)
- **Related Terms**: QPS, Bandwidth, Performance

### Throttling
Limiting the rate of operations to prevent system overload. Similar to rate limiting.
- **Module Reference**: [Module 10: API Design](../10-api-design/)
- **Related Terms**: Rate Limiting, Backpressure

### Time-To-Live (TTL)
The amount of time data should be cached or stored before being expired or refreshed.
- **Module Reference**: [Module 5: Caching](../05-caching/)
- **Related Terms**: Cache Expiration, DNS TTL

### Token Bucket
A rate limiting algorithm that allows bursts of traffic while maintaining an average rate limit.
- **Module Reference**: [Module 10: API Design](../10-api-design/)
- **Related Terms**: Rate Limiting, Throttling

### TPS (Transactions Per Second)
A measure of the number of transactions a system can process per second.
- **Module Reference**: [Module 1: Back-of-Envelope Calculations](../01-back-of-envelope/)
- **Related Terms**: QPS, Throughput, Performance

### Two-Phase Commit (2PC)
A distributed algorithm that ensures all nodes in a distributed transaction commit or abort atomically.
- **Module Reference**: [Module 9: Consistency Patterns](../09-consistency-patterns/)
- **Related Terms**: Distributed Transaction, ACID, Consistency

## U

### UUID (Universally Unique Identifier)
A 128-bit identifier that is globally unique, commonly used for distributed ID generation.
- **Module Reference**: [Module 7: Data Partitioning](../07-data-partitioning/)
- **Related Terms**: ID Generation, GUID

## V

### Vector Clock
A mechanism for capturing causality between events in a distributed system.
- **Module Reference**: [Module 9: Consistency Patterns](../09-consistency-patterns/)
- **Related Terms**: Eventual Consistency, Conflict Resolution

### Vertical Scaling (Scale-Up)
Increasing the capacity of a single machine by adding more CPU, RAM, or storage.
- **Module Reference**: [Module 3: Scalability](../03-scalability/)
- **Related Terms**: Horizontal Scaling, Scale-Out

## W

### WAL (Write-Ahead Log)
A logging mechanism where changes are written to a log before being applied to the database, ensuring durability.
- **Module Reference**: [Module 6: Database Design](../06-database-design/)
- **Related Terms**: Durability, Transaction Log, ACID

### WebSocket
A protocol providing full-duplex communication channels over a single TCP connection, enabling real-time bi-directional communication.
- **Module Reference**: [Module 11: Message Queues](../11-message-queues/)
- **Related Terms**: Real-Time Communication, Bi-directional, HTTP

### Write-Ahead Log
See **WAL (Write-Ahead Log)**

### Write-Through Cache
A caching pattern where writes go to both cache and database simultaneously before returning success.
- **Module Reference**: [Module 5: Caching](../05-caching/)
- **Related Terms**: Write-Behind Cache, Cache-Aside

## Z

### Zero Downtime Deployment
A deployment strategy that updates a system without causing service interruption to users.
- **Module Reference**: [Module 12: Microservices](../12-microservices/)
- **Related Terms**: Blue-Green Deployment, Rolling Deployment, Canary Deployment

### ZooKeeper
A distributed coordination service for maintaining configuration, naming, synchronization, and group services.
- **Module Reference**: [Module 12: Microservices](../12-microservices/)
- **Related Terms**: Coordination Service, Distributed Consensus, Service Discovery

---

## Quick Reference by Module

### Module 0: Foundations
DNS, Distributed System, Requirements Gathering

### Module 1: Back-of-Envelope Calculations
Back-of-Envelope Calculation, Latency, Throughput, QPS, TPS, Bandwidth

### Module 2: CAP Theorem
CAP Theorem, Consistency, Availability, Partition Tolerance, Eventually Consistent, Strong Consistency, Network Partition, Split Brain

### Module 3: Scalability
Scalability, Horizontal Scaling, Vertical Scaling, Elastic Scaling

### Module 4: Load Balancing
Load Balancer, Round Robin, Consistent Hashing, Reverse Proxy, Forward Proxy, Sticky Session, Health Check

### Module 5: Caching
Cache, Cache Hit, Cache Miss, Cache-Aside, Write-Through Cache, Read-Through Cache, LRU, TTL, Redis, Memcached, Bloom Filter

### Module 6: Database Design
SQL, NoSQL, Database Index, Normalization, Denormalization, CQRS, ETL, WAL, Object Storage, Federation

### Module 7: Data Partitioning
Data Partitioning, Sharding, Shard Key, Hash Function, Consistent Hashing, Hot Spot, UUID

### Module 8: Replication
Replication, Replica, Leader-Follower Replication, Multi-Leader Replication, Replication Lag, Gossip Protocol, RAID

### Module 9: Consistency Patterns
Consistency, Strong Consistency, Eventually Consistent, Quorum, Vector Clock, Two-Phase Commit, Event Sourcing

### Module 10: API Design
API, REST, GraphQL, gRPC, RPC, Rate Limiting, Throttling, Idempotency, Token Bucket, Stateless

### Module 11: Message Queues
Message Queue, Pub/Sub, Kafka, Asynchronous Processing, Event-Driven Architecture, Fan-Out, Back Pressure, Batch Processing, WebSocket

### Module 12: Microservices
Microservices, API Gateway, Service Discovery, Service Mesh, Docker, Kubernetes, Saga Pattern, Distributed Tracing, Observability, Zero Downtime Deployment

### Module 13: Availability Patterns
Availability, High Availability, Fault Tolerance, Circuit Breaker, Failover, Health Check, Heartbeat, SLA, SLO, SLI, Single Point of Failure, Redundancy, Load Shedding, Jitter

### Module 14: CDN
CDN, Edge Computing, Origin Server, Point of Presence

---

**Total Terms**: 145+

This glossary is a living document. As you progress through the course, refer back to these definitions to reinforce your understanding of system design concepts.
