# Design Challenge 4: Distributed Key-Value Cache

**Time Limit:** 45 minutes  
**Difficulty:** Advanced  
**Topics Covered:** Distributed systems, consistent hashing, replication, CAP theorem

---

## Problem Statement

Design a distributed in-memory key-value cache system similar to Memcached or Redis Cluster that provides high throughput and low latency access to frequently used data.

## Requirements

### Functional Requirements
1. Support basic operations: GET, SET, DELETE
2. Support TTL (Time To Live) for automatic key expiration
3. LRU eviction when memory is full
4. Ability to add/remove cache nodes dynamically
5. Data should be distributed across multiple nodes

### Non-Functional Requirements
1. Sub-millisecond latency for operations (p99 < 5ms)
2. Support 1 million operations per second
3. Handle node failures gracefully
4. Minimize data movement when scaling
5. High availability (99.99% uptime)
6. Support storing values up to 1MB

### Out of Scope
- Complex data structures (lists, sets, sorted sets)
- Persistence to disk
- Authentication/authorization
- Pub/sub functionality

---

## Your Design Task

### 1. Capacity Estimation (8 minutes)
- Memory per node
- Number of nodes needed
- Network bandwidth

### 2. API Design (5 minutes)
- Client interface
- Internal node communication protocol

### 3. Data Distribution Strategy (12 minutes)
- How keys are distributed across nodes
- Consistent hashing implementation
- Replication strategy

### 4. Architecture (12 minutes)
- Node architecture
- Client routing
- Cluster membership management

### 5. Failure Handling (8 minutes)
- Node failure detection
- Data recovery
- Split-brain prevention

---

## Self-Grading Rubric (70 points total)

**Technical Correctness (40 points):**
- [ ] Consistent hashing properly explained (8 points)
- [ ] Replication strategy is sound (8 points)
- [ ] LRU eviction correctly implemented (6 points)
- [ ] TTL mechanism is efficient (6 points)
- [ ] Handles node additions/removals (6 points)
- [ ] Failure handling is comprehensive (6 points)

**System Design (20 points):**
- [ ] Meets latency requirements (5 points)
- [ ] Scalable architecture (5 points)
- [ ] High availability design (5 points)
- [ ] CAP theorem trade-offs addressed (5 points)

**Communication (10 points):**
- [ ] Clear explanations (5 points)
- [ ] Trade-offs justified (5 points)

---

## Sample Solution

See `challenge-04-solution.md` after completing your design.
