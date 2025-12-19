# Replication Design Exercises

## Exercise 1: Choose Replication Strategy

**Scenario:** Global social media platform

Requirements:
- 1 billion users
- Users in US, Europe, Asia
- High availability needed
- Occasional stale data OK

Which replication strategy?

<details>
<summary>Solution</summary>

**Multi-Leader or Leaderless**

**Reasoning:**
- Geographic distribution (low latency everywhere)
- High availability critical
- Eventual consistency acceptable for social media
- Handle network partitions between regions

**Implementation:**
- Leader in each region (US, EU, Asia)
- Async replication between regions
- Users write to nearest leader
- Conflict resolution: Last-write-wins or version vectors

</details>

## Exercise 2: Handle Replication Lag

User posts a comment and immediately refreshes. How do you ensure they see their own comment?

<details>
<summary>Solution</summary>

**Read-After-Write Consistency**

Options:
1. Read user's own writes from leader
2. Track write timestamp, wait for replication
3. Sticky sessions (same replica)

**Best:** Read user's own content from leader, others from replicas.

</details>

More exercises in [design-exercise.md](./design-exercise.md).
