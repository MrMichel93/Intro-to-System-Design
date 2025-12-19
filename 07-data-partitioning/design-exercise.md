# Data Partitioning Design Exercises

## Exercise 1: Choose Partition Strategy

You're designing a Twitter-like system. Choose the best partition key:
- Option A: tweet_id
- Option B: user_id
- Option C: timestamp

<details>
<summary>Solution</summary>

**Best: user_id (Option B)**

**Why:**
- Most queries: "Get user's tweets"
- Keeps user's data together
- Even distribution of users

Tweet_id would scatter user's tweets across shards.
Timestamp would create hot shards (everyone tweets now!).

</details>

## Exercise 2: Design for Growth

Current: 10M users, 4 shards
Future: 1B users

How do you design partitioning to handle growth?

<details>
<summary>Solution</summary>

Use consistent hashing or plan for resharding:

**Option 1:** Start with 16-32 shards (room to grow)
**Option 2:** Use consistent hashing (easy to add shards)
**Option 3:** Plan resharding windows (e.g., double shards yearly)

Trade-off: More shards = more complexity now vs easier later.

</details>

More exercises in [design-exercise.md](./design-exercise.md).
