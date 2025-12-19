# Design Challenge 5: Distributed Web Crawler

**Time Limit:** 45 minutes  
**Difficulty:** Advanced  
**Topics Covered:** Distributed systems, queueing, politeness, deduplication, scalability

---

## Problem Statement

Design a distributed web crawler similar to what Google or Bing uses to index the web. The system should efficiently discover and download web pages while being polite to web servers.

## Requirements

### Functional Requirements
1. Crawl 1 billion web pages per month
2. Store HTML content and metadata (URL, timestamp, title)
3. Respect robots.txt and crawl-delay directives
4. Follow links to discover new pages
5. Support recrawling of pages (freshness)
6. Detect and handle duplicate URLs

### Non-Functional Requirements
1. Distributed: Scale horizontally across multiple machines
2. Politeness: Maximum 1 request per domain per second
3. Efficient: Minimize duplicate work
4. Resilient: Handle failures gracefully
5. Prioritization: Important pages crawled more frequently
6. Storage: Efficiently store crawled data

### Out of Scope
- Content parsing/analysis
- Rendering JavaScript
- Handling authentication
- Image/video downloading
- Search indexing

---

## Your Design Task

### 1. Capacity Estimation (8 minutes)
- Pages per second to crawl
- Storage requirements
- Bandwidth needs
- Number of crawler machines

### 2. URL Frontier Design (12 minutes)
- How URLs are queued for crawling
- Prioritization strategy
- Politeness enforcement

### 3. System Architecture (15 minutes)
- High-level components
- URL discovery and deduplication
- Distributed coordination
- Storage strategy

### 4. Handling Challenges (10 minutes)
- Duplicate URLs
- URL traps (infinite loops)
- Failed requests
- Load distribution

---

## Self-Grading Rubric (70 points total)

**Technical Correctness (40 points):**
- [ ] URL frontier properly designed (8 points)
- [ ] Politeness correctly enforced (8 points)
- [ ] Deduplication strategy is effective (8 points)
- [ ] Storage architecture is scalable (8 points)
- [ ] Distributed coordination handled (8 points)

**System Design (20 points):**
- [ ] Meets throughput requirements (5 points)
- [ ] Scalable architecture (5 points)
- [ ] Handles failures (5 points)
- [ ] Priority/freshness addressed (5 points)

**Communication (10 points):**
- [ ] Clear component descriptions (5 points)
- [ ] Trade-offs explained (5 points)

---

## Sample Solution

See `challenge-05-solution.md` after completing your design.
