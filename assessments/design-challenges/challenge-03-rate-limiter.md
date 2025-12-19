# Design Challenge 3: Distributed Rate Limiter

**Time Limit:** 45 minutes  
**Difficulty:** Intermediate-Advanced  
**Topics Covered:** Rate limiting algorithms, distributed systems, caching, API design

---

## Problem Statement

Design a distributed rate limiting system that can be used to protect APIs from abuse and ensure fair resource usage across multiple servers.

## Requirements

### Functional Requirements
1. Support multiple rate limiting rules:
   - Limit requests per user per minute/hour/day
   - Limit requests per IP address
   - Limit requests per API endpoint
2. Support different rate limits for different user tiers (free, premium, enterprise)
3. Return clear error messages when limits are exceeded
4. Provide real-time visibility into current usage
5. Allow dynamic configuration of rate limits without system restart

### Non-Functional Requirements
1. Low latency (< 10ms overhead per request)
2. Highly available (99.99% uptime)
3. Distributed: Work across multiple application servers
4. Scalable: Handle 100,000 requests per second
5. Accurate: Rate limiting should be consistent across all servers
6. No single point of failure

### Out of Scope
- User authentication system
- API gateway implementation
- Billing/payment system
- Detailed analytics and reporting

---

## Your Design Task

### 1. Algorithm Selection (10 minutes)
Compare and choose a rate limiting algorithm:
- Token Bucket
- Leaky Bucket
- Fixed Window Counter
- Sliding Window Log
- Sliding Window Counter

Explain your choice and trade-offs.

**Your Answer:**
```
[Write your analysis here]
```

---

### 2. API Design (5 minutes)
Define:
- How rate limiter is invoked
- Response format when limit is exceeded
- Headers to include in responses

**Your Answer:**
```
[Write your API design here]
```

---

### 3. Data Model (8 minutes)
Design the data structure for storing:
- Rate limit rules
- Current usage counters
- User tier configurations

**Your Answer:**
```
[Write your data model here]
```

---

### 4. High-Level Architecture (12 minutes)
Draw and describe your distributed rate limiting architecture including:
- Where rate limiting logic runs
- How state is shared across servers
- Storage layer for counters
- Configuration management

**Your Answer:**
```
[Draw/describe your architecture here]
```

---

### 5. Handling Edge Cases (10 minutes)
Explain how you handle:
- Clock synchronization issues across servers
- Race conditions in distributed environment
- Cache failures
- Sudden traffic spikes

**Your Answer:**
```
[Write your solutions here]
```

---

## Self-Grading Rubric

Grade yourself on each criterion (0-5 points):

### Technical Correctness (35 points total)
- [ ] Algorithm choice is well-justified (5 points)
- [ ] API design is clear and complete (5 points)
- [ ] Data model supports all requirements (5 points)
- [ ] Architecture is truly distributed (5 points)
- [ ] Handles race conditions properly (5 points)
- [ ] Addresses consistency challenges (5 points)
- [ ] Performance requirements are met (5 points)

### System Design Principles (20 points total)
- [ ] Low latency design (5 points)
- [ ] High availability approach (5 points)
- [ ] Scalability considerations (5 points)
- [ ] No single point of failure (5 points)

### Communication and Clarity (15 points total)
- [ ] Trade-offs clearly explained (5 points)
- [ ] Edge cases addressed (5 points)
- [ ] Diagrams are clear (5 points)

### Total: ____ / 70 points

---

## Sample Solution

See `challenge-03-solution.md` after completing your design.
