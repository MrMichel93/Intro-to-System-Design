# Design Challenge 2: Social Media News Feed

**Time Limit:** 45 minutes  
**Difficulty:** Intermediate  
**Topics Covered:** Fan-out patterns, caching, database design, real-time updates

---

## Problem Statement

Design a news feed system similar to Twitter or Facebook where users can post updates and see posts from people they follow in their personalized feed.

## Requirements

### Functional Requirements
1. Users can post status updates (text, max 280 characters)
2. Users can follow other users
3. Users can view their personalized news feed with posts from people they follow
4. Feed should show most recent posts first (reverse chronological)
5. Support 10,000 posts per user maximum
6. Posts are immutable (cannot be edited, only deleted)

### Non-Functional Requirements
1. System should support 500 million users
2. 50 million daily active users (DAU)
3. Each user follows 200 people on average
4. Feed generation should be fast (< 200ms)
5. New posts should appear in followers' feeds within 1 minute
6. System should handle celebrity users with millions of followers

### Out of Scope
- Likes, comments, shares
- Media uploads (photos, videos)
- Direct messaging
- Notifications
- Search functionality

---

## Your Design Task

### 1. Capacity Estimation (10 minutes)
Calculate:
- Daily active users' post generation rate
- Feed generation requests
- Storage requirements
- Database size estimation

**Your Answer:**
```
[Write your calculations here]
```

---

### 2. API Design (5 minutes)
Define the API endpoints for:
- Creating a post
- Following a user
- Fetching news feed

**Your Answer:**
```
[Write your API design here]
```

---

### 3. Database Schema (8 minutes)
Design schemas for:
- Users
- Posts
- Follow relationships
- News feeds (if needed)

**Your Answer:**
```
[Write your schema here]
```

---

### 4. High-Level Architecture (15 minutes)
Draw and describe your system architecture addressing:
- How posts are created
- How feeds are generated
- Fan-out strategy (push vs pull)
- Caching strategy

**Your Answer:**
```
[Draw/describe your architecture here]
```

---

### 5. Feed Generation Strategy (7 minutes)
Explain your approach for:
- Generating feeds efficiently
- Handling celebrity users (with millions of followers)
- Ensuring recent posts appear quickly

**Your Answer:**
```
[Write your strategy here]
```

---

## Self-Grading Rubric

Grade yourself on each criterion (0-5 points):

### Technical Correctness (35 points total)
- [ ] Capacity calculations are reasonable (5 points)
- [ ] API design is complete and RESTful (5 points)
- [ ] Database schema supports requirements (5 points)
- [ ] Architecture handles scale (5 points)
- [ ] Fan-out strategy is well-designed (5 points)
- [ ] Addresses celebrity user problem (5 points)
- [ ] Caching strategy is appropriate (5 points)

### System Design Principles (20 points total)
- [ ] Design balances read vs write optimization (5 points)
- [ ] Handles high throughput (5 points)
- [ ] Feed latency meets requirements (5 points)
- [ ] Considers consistency trade-offs (5 points)

### Communication and Clarity (15 points total)
- [ ] Assumptions clearly stated (5 points)
- [ ] Trade-offs explained (5 points)
- [ ] Diagrams are clear (5 points)

### Total: ____ / 70 points

---

## Sample Solution

See `challenge-02-solution.md` after completing your design.
