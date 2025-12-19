# Consistency Pattern Examples

## Example 1: Banking System (Strong Consistency)

**Pattern:** Linearizable
**Why:** Money must be accurate
**Implementation:** Synchronous replication, all nodes confirm

## Example 2: Twitter Feed (Eventual Consistency)

**Pattern:** Eventual consistency
**Why:** Brief delay acceptable, high availability needed
**Implementation:** Async replication, cache aggressively

## Example 3: Google Docs (Causal Consistency)

**Pattern:** Causal + Operational Transform
**Why:** Preserve edit causality, real-time collaboration
**Implementation:** Track operations, transform concurrent edits

See [README](./README.md) for more examples.
