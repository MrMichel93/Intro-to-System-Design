# Design Challenge 1: URL Shortener Service

**Time Limit:** 45 minutes  
**Difficulty:** Beginner-Intermediate  
**Topics Covered:** Database design, hashing, caching, scalability basics

---

## Problem Statement

Design a URL shortening service like bit.ly or TinyURL that converts long URLs into short, shareable links.

## Requirements

### Functional Requirements
1. Users can submit a long URL and receive a shortened URL
2. When users access the shortened URL, they are redirected to the original URL
3. Shortened URLs should be as short as possible (6-8 characters)
4. Custom short URLs (aliases) should be supported if available
5. Track basic analytics (click count per short URL)

### Non-Functional Requirements
1. System should handle 100 million URL shortenings per month
2. Read/write ratio is 100:1 (reads are far more common)
3. Short URLs should not be predictable
4. Service should be highly available (99.9% uptime)
5. Low latency for redirects (< 100ms)

### Out of Scope
- User authentication/accounts
- Link expiration
- Advanced analytics (geographic location, device types)
- Link editing or deletion

---

## Your Design Task

Complete the following sections:

### 1. Capacity Estimation (10 minutes)
Calculate:
- Queries per second (QPS) for writes
- Queries per second (QPS) for reads
- Storage required for 5 years
- Bandwidth requirements

**Your Answer:**
```
[Write your calculations here]
```

---

### 2. API Design (5 minutes)
Define the API endpoints for:
- Creating a short URL
- Redirecting to the original URL

**Your Answer:**
```
[Write your API design here]
```

---

### 3. Database Schema (5 minutes)
Design the database schema for storing URLs.

**Your Answer:**
```
[Write your schema here]
```

---

### 4. High-Level Architecture (15 minutes)
Draw and describe your system architecture including:
- Client
- Application servers
- Database
- Cache
- Any other components

**Your Answer:**
```
[Draw/describe your architecture here]
```

---

### 5. Algorithm for Generating Short URLs (5 minutes)
Explain your approach for generating unique short URLs.

**Your Answer:**
```
[Write your algorithm here]
```

---

### 6. Trade-offs and Optimizations (5 minutes)
Discuss:
- Key design decisions and why you made them
- Potential bottlenecks
- How you would scale this system

**Your Answer:**
```
[Write your analysis here]
```

---

## Self-Grading Rubric

Grade yourself on each criterion (0-5 points):

### Technical Correctness (30 points total)
- [ ] Capacity calculations are accurate and well-explained (5 points)
- [ ] API design is RESTful and complete (5 points)
- [ ] Database schema supports all requirements (5 points)
- [ ] Architecture includes necessary components (5 points)
- [ ] Short URL generation algorithm is sound (5 points)
- [ ] Identified major bottlenecks correctly (5 points)

### System Design Principles (25 points total)
- [ ] Design addresses scalability requirements (5 points)
- [ ] Caching strategy is appropriate (5 points)
- [ ] Database choice is justified (5 points)
- [ ] Handles high read/write ratio efficiently (5 points)
- [ ] Considers availability and fault tolerance (5 points)

### Communication and Clarity (15 points total)
- [ ] Assumptions are clearly stated (5 points)
- [ ] Diagrams are clear and labeled (5 points)
- [ ] Trade-offs are well-articulated (5 points)

### Total: ____ / 70 points

**Grading Scale:**
- 60-70: Excellent - Strong system design skills
- 50-59: Good - Solid understanding with minor gaps
- 40-49: Fair - Basic understanding, needs improvement
- Below 40: Needs significant improvement

---

## Sample Solution

See `challenge-01-solution.md` for a detailed sample solution after completing your design.
