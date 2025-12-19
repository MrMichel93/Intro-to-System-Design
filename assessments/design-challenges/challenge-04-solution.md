# Design Challenge 4: Distributed Cache - Sample Solution

## 1. Capacity Estimation

### Assumptions:
- Average key size: 50 bytes
- Average value size: 1KB
- Average entry overhead: 100 bytes (metadata, pointers)
- Total per entry: ~1.15 KB

### Memory per Node:
```
Target: Store 100 million entries
With replication factor 3: Need 300 million entry-replicas total
Storage: 300M × 1.15KB = 345GB total across cluster

With 10 nodes: 345GB / 10 = ~35GB per node
Add 20% overhead: 42GB RAM per node
Recommendation: Use 64GB RAM nodes
```

### Operations:
```
Target: 1 million ops/sec
With replication reads: ~80% reads, 20% writes
Reads: 800K/sec, Writes: 200K/sec

Per node (10 nodes): 80K reads/sec, 20K writes/sec
This is well within modern server capabilities
```

---

## 2. API Design

### Client API:
```python
# Basic operations
cache.set(key, value, ttl=3600)  # TTL in seconds
value = cache.get(key)
cache.delete(key)

# Batch operations
cache.mget([key1, key2, key3])
cache.mset({key1: value1, key2: value2})
```

### Internal Protocol (Node-to-Node):
```
REPLICATE <key> <value> <ttl> <version>
HEARTBEAT <node_id> <timestamp>
GOSSIP <node_id> <status> <keys_count>
HANDOFF <key> <value> <target_node>
```

---

## 3. Data Distribution Strategy

### Consistent Hashing:

```
Hash Ring (0 to 2^32-1):

Node A: hash("node_a") → 1000
Node B: hash("node_b") → 5000
Node C: hash("node_c") → 9000

Key distribution:
key1: hash("key1") → 3000 → Assigned to Node B (next clockwise)
key2: hash("key2") → 7000 → Assigned to Node C
key3: hash("key3") → 500 → Assigned to Node A
```

**Virtual Nodes (for better distribution):**
- Each physical node has 150 virtual nodes on ring
- Better load balancing
- Smoother data distribution when adding/removing nodes

### Replication Strategy:

**Approach:** Chain replication with N=3
```
For key stored on Node A:
Primary: Node A
Replicas: Next 2 nodes clockwise (Node B, Node C)

Write path: A → B → C (synchronous or async)
Read path: Any of A, B, or C (load distribution)
```

**Consistency Level Options:**
- ONE: Read/write to any replica (fastest, least consistent)
- QUORUM: Read/write to majority (N/2 + 1 = 2 nodes)
- ALL: Read/write to all replicas (slowest, most consistent)

---

## 4. Architecture

```
┌──────────────────────────────────────────────────────┐
│                    Clients                            │
└───┬────────────────┬────────────────┬────────────────┘
    │                │                │
┌───▼─────┐      ┌───▼─────┐      ┌──▼──────┐
│ Cache   │      │ Cache   │      │ Cache   │
│ Node A  │◄────►│ Node B  │◄────►│ Node C  │
└────┬────┘      └────┬────┘      └────┬────┘
     │                │                │
     └────────────────┴────────────────┘
               Gossip Protocol
```

### Cache Node Components:

```
┌────────────────────────────────────┐
│         Cache Node                  │
├────────────────────────────────────┤
│  Request Handler (API)              │
├────────────────────────────────────┤
│  Routing Layer                      │
│  (Consistent Hashing)               │
├────────────────────────────────────┤
│  In-Memory Storage                  │
│  (Hash Table + LRU List)           │
├────────────────────────────────────┤
│  Replication Manager                │
├────────────────────────────────────┤
│  Cluster Manager                    │
│  (Gossip, Membership)               │
├────────────────────────────────────┤
│  TTL Manager (Background)           │
└────────────────────────────────────┘
```

### Data Structures:

**In-Memory Storage:**
```python
class CacheNode:
    def __init__(self):
        self.hash_map = {}  # key → CacheEntry
        self.lru_list = DoublyLinkedList()  # For eviction
        self.lock = ReadWriteLock()
    
class CacheEntry:
    def __init__(self, key, value, ttl):
        self.key = key
        self.value = value
        self.expiry = time.time() + ttl
        self.version = generate_version()
        self.lru_node = None  # Pointer to LRU list node
```

**Consistent Hash Ring:**
```python
class ConsistentHashRing:
    def __init__(self, nodes, virtual_nodes=150):
        self.ring = SortedDict()  # Position → node_id
        for node in nodes:
            self.add_node(node, virtual_nodes)
    
    def add_node(self, node_id, virtual_nodes):
        for i in range(virtual_nodes):
            position = hash(f"{node_id}:{i}")
            self.ring[position] = node_id
    
    def get_node(self, key):
        position = hash(key)
        # Find next node clockwise
        node = self.ring.get_next(position)
        return node
```

---

## 5. Failure Handling

### Node Failure Detection:

**Heartbeat Mechanism:**
```
Every 1 second:
  - Each node sends heartbeat to neighbors
  - If no heartbeat for 5 seconds → Mark as suspected
  - If no heartbeat for 10 seconds → Mark as dead
  - Gossip failure information to cluster
```

**Gossip Protocol:**
- Nodes exchange membership information periodically
- Failure information propagates through cluster
- Achieves eventual consistency on cluster state

### Data Recovery:

**When Node Fails:**
```
1. Detect failure (via heartbeat/gossip)
2. Identify keys owned by failed node
3. Promote replica nodes to primary
4. Hinted handoff: Store new writes for failed node
5. When node recovers: Replay hinted writes
```

**Read Repair:**
```
During read:
  1. Read from primary + replicas
  2. Compare versions
  3. If mismatch: Repair stale replicas
  4. Return latest version to client
```

### Split-Brain Prevention:

**Problem:** Network partition causes cluster split

**Solution: Quorum-based operations:**
```
For write to succeed:
  - Must reach W replicas (W = 2 for N=3)
  - Minority partition cannot accept writes
  - Prevents conflicting writes

For read:
  - Read from R replicas (R = 2 for N=3)
  - Return latest version by vector clock
```

**Vector Clocks for Conflict Detection:**
```
Entry version: {NodeA: 5, NodeB: 3, NodeC: 2}
Indicates: 
  - NodeA has applied 5 updates
  - NodeB has applied 3 updates
  - NodeC has applied 2 updates
```

---

## 6. Key Implementation Details

### LRU Eviction:

```python
def get(key):
    if key in hash_map:
        entry = hash_map[key]
        if entry.is_expired():
            delete(key)
            return None
        # Move to front of LRU list
        lru_list.move_to_front(entry.lru_node)
        return entry.value
    return None

def set(key, value, ttl):
    if memory_full():
        # Evict least recently used
        lru_entry = lru_list.remove_tail()
        del hash_map[lru_entry.key]
    
    entry = CacheEntry(key, value, ttl)
    hash_map[key] = entry
    entry.lru_node = lru_list.add_to_front(entry)
```

### TTL Management:

**Lazy Expiration:**
- Check expiry on access
- Low overhead

**Active Expiration:**
```python
# Background thread
def expire_keys():
    while True:
        # Sample random keys
        keys = random.sample(hash_map.keys(), 100)
        for key in keys:
            if hash_map[key].is_expired():
                delete(key)
        sleep(1)
```

### Dynamic Scaling:

**Adding Node:**
```
1. New node joins cluster
2. Update consistent hash ring
3. Calculate key ranges to transfer
4. Stream data from existing nodes
5. Update routing tables
6. New node ready for traffic

Data movement: Only 1/N keys move (N = node count)
```

**Removing Node:**
```
1. Mark node for decommission
2. Stream data to successor nodes
3. Update routing tables
4. Remove from cluster
```

---

## 7. Trade-offs

### CAP Theorem Choices:

**CP (Consistency + Partition Tolerance):**
- Quorum reads/writes
- Strong consistency
- Lower availability during partitions

**AP (Availability + Partition Tolerance):**
- Read from any replica
- Eventually consistent
- Always available

**Our Choice: Tunable Consistency**
- Client chooses consistency level
- ONE for high availability
- QUORUM for balanced
- ALL for strong consistency

### Performance Optimizations:

1. **Client-side routing:** Clients cache hash ring, route directly to correct node
2. **Zero-copy networking:** Use sendfile() for replication
3. **Lock-free reads:** Use atomic operations where possible
4. **Batching:** Group small operations into batches
5. **Compression:** Compress large values to save memory/bandwidth

---

## Summary

This design achieves:
- ✅ Sub-millisecond latency (< 1ms typical)
- ✅ 1M+ ops/sec throughput
- ✅ Graceful failure handling
- ✅ Minimal data movement on scaling
- ✅ 99.99% availability with proper replication

The consistent hashing + replication approach provides excellent scalability and fault tolerance while maintaining high performance.
