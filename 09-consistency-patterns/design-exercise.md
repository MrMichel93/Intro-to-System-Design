# Consistency Pattern Design Exercises

## Exercise 1: Choose Consistency Level

For each system, choose appropriate consistency:

**A. Stock trading platform**
<details>
<summary>Answer</summary>
Strong consistency (Linearizable) - Money and trades must be exact.
</details>

**B. YouTube view counter**
<details>
<summary>Answer</summary>
Eventual consistency - Approximate count is fine, prioritize performance.
</details>

**C. Flight booking system**
<details>
<summary>Answer</summary>
Strong consistency - Can't double-book seats.
</details>

## Exercise 2: Design with Quorum

Given: 5-node Cassandra cluster
Design W and R for:

**Scenario A: Important user data, balanced performance**
<details>
<summary>Answer</summary>
W=3, R=3 (Quorum) - Good balance, guaranteed consistency.
</details>

**Scenario B: Cache data, speed critical**
<details>
<summary>Answer</summary>
W=1, R=1 - Fast but eventual consistency, OK for cache.
</details>

More exercises in [design-exercise.md](./design-exercise.md).
