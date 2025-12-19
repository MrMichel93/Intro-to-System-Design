# Problem Analysis Template

Use this template to systematically break down any system design problem before jumping into solutions.

## 1. Problem Statement

**Clearly define what you're building:**

Write a 2-3 sentence description of the system.

```
Example: Design a URL shortening service that converts long URLs into short, 
shareable links and redirects users from short URLs to the original destinations.
```

---

## 2. Clarifying Questions

**Ask these questions to understand the scope before designing:**

### Functional Requirements
- [ ] What are the core features users need?
- [ ] Are there any optional/nice-to-have features?
- [ ] Who are the users (consumers, businesses, internal teams)?
- [ ] What actions can users perform?
- [ ] Are there different user roles/permissions?
- [ ] Should the system support mobile, web, or both?
- [ ] Are there any integrations with external systems?

### Non-Functional Requirements
- [ ] What is the expected scale (users, requests, data)?
- [ ] What are the performance requirements (latency, throughput)?
- [ ] What is the availability requirement (99%, 99.9%, 99.99%)?
- [ ] What are the consistency requirements (strong, eventual)?
- [ ] Are there geographic constraints (single region, multi-region)?
- [ ] What is the data retention policy?
- [ ] Are there security/compliance requirements?

### Constraints & Assumptions
- [ ] What is the budget/cost constraint?
- [ ] What is the timeline for deployment?
- [ ] Are there technology preferences or restrictions?
- [ ] What is the team's expertise level?
- [ ] Are there existing systems to integrate with?
- [ ] Can we use third-party services?

---

## 3. Requirements Definition

### Functional Requirements

List concrete, measurable features:

**Example:**
1. Users can submit a long URL and receive a shortened URL
2. Users can access a short URL and be redirected to the original URL
3. Short URLs expire after 5 years of inactivity
4. Users can view click statistics for their URLs

### Non-Functional Requirements

**Scale:**
- Number of users: ___
- Requests per second: ___
- Data volume: ___
- Growth rate: ___

**Performance:**
- Read latency: ___ ms (e.g., < 100ms)
- Write latency: ___ ms (e.g., < 200ms)
- Throughput: ___ requests/sec

**Availability:**
- Target uptime: ___% (e.g., 99.9%)
- Maximum acceptable downtime: ___

**Consistency:**
- [ ] Strong consistency (all reads see latest write)
- [ ] Eventual consistency (reads may see stale data temporarily)
- [ ] Per-feature consistency requirements

**Data:**
- Data durability: ___ (e.g., no data loss)
- Retention period: ___
- Backup requirements: ___

---

## 4. Capacity Estimation

### Traffic Estimates

**Writes (Creating/Updating):**
```
Daily active users (DAU): ___
Actions per user per day: ___
Total writes per day: ___ × ___ = ___
Writes per second (avg): ___ / 86,400 = ___
Peak writes per second (3x): ___ × 3 = ___
```

**Reads (Viewing/Retrieving):**
```
Read-to-write ratio: ___ : 1
Total reads per day: ___ × ___ = ___
Reads per second (avg): ___ / 86,400 = ___
Peak reads per second (3x): ___ × 3 = ___
```

### Storage Estimates

**Data per record:**
```
Record fields:
- Field 1: ___ bytes
- Field 2: ___ bytes
- Field 3: ___ bytes
- Metadata: ___ bytes
Total per record: ___ bytes
```

**Total storage:**
```
Records per day: ___
Records per year: ___ × 365 = ___
Years to store: ___
Total records: ___
Storage required: ___ records × ___ bytes = ___ TB/PB
```

**Storage with replication:**
```
Replication factor: ___ (e.g., 3x)
Total storage: ___ TB × ___ = ___ TB
```

### Bandwidth Estimates

**Incoming (Writes):**
```
Write size: ___ bytes
Writes per second: ___
Bandwidth: ___ bytes/sec × ___ = ___ MB/sec
```

**Outgoing (Reads):**
```
Read size: ___ bytes
Reads per second: ___
Bandwidth: ___ bytes/sec × ___ = ___ MB/sec
```

### Memory/Cache Estimates

**Caching hot data (80/20 rule):**
```
Percentage to cache: 20%
Records to cache: ___ × 0.2 = ___
Size per cached record: ___ bytes
Cache size needed: ___ records × ___ bytes = ___ GB
```

---

## 5. Core Use Cases & Flows

**List 3-5 primary use cases with detailed flows:**

### Use Case 1: [Name]

**Actor:** [Who performs this action]

**Preconditions:** [What must be true before this use case]

**Flow:**
1. Step 1
2. Step 2
3. Step 3
4. ...

**Postconditions:** [What is true after this use case]

**Error Scenarios:**
- What if step X fails?
- What if data is invalid?

---

### Use Case 2: [Name]

[Repeat format above]

---

## 6. Core Entities & Data Model

**Identify main entities and their relationships:**

### Entity 1: [Name]

**Attributes:**
- `id`: Unique identifier
- `attribute1`: Description
- `attribute2`: Description
- `created_at`: Timestamp
- `updated_at`: Timestamp

**Relationships:**
- Relates to Entity2 (one-to-many)
- Relates to Entity3 (many-to-many)

---

## 7. Success Metrics

**How will you measure if the system is working well?**

### Performance Metrics
- [ ] Average response time: ___ ms
- [ ] P95 response time: ___ ms
- [ ] P99 response time: ___ ms
- [ ] Throughput: ___ requests/sec
- [ ] Error rate: < ___%

### Business Metrics
- [ ] User engagement: ___
- [ ] Conversion rate: ___
- [ ] Feature adoption: ___
- [ ] Cost per request: ___

### Reliability Metrics
- [ ] Uptime: ___%
- [ ] Mean time between failures (MTBF): ___
- [ ] Mean time to recovery (MTTR): ___
- [ ] Data durability: ___%

---

## 8. Constraints & Bottlenecks

**Identify potential limitations:**

### Technical Constraints
- [ ] Database size limits
- [ ] Network bandwidth limits
- [ ] Single point of failure concerns
- [ ] Technology stack limitations

### Business Constraints
- [ ] Budget limitations
- [ ] Time to market
- [ ] Team expertise
- [ ] Regulatory requirements

### Potential Bottlenecks
- [ ] Database reads/writes
- [ ] Network latency
- [ ] CPU intensive operations
- [ ] Memory usage
- [ ] Disk I/O

---

## 9. Prioritization

**Rank requirements by importance:**

### Must Have (P0) - Launch Blockers
1. 
2. 
3. 

### Should Have (P1) - Important for Success
1. 
2. 
3. 

### Nice to Have (P2) - Future Enhancements
1. 
2. 
3. 

---

## 10. Out of Scope

**Explicitly state what you're NOT building:**

- 
- 
- 

---

## Example: URL Shortener Problem Analysis

### 1. Problem Statement
Design a URL shortening service like bit.ly that converts long URLs into short, shareable links and tracks basic analytics.

### 2. Clarifying Questions Asked
- **Scale**: How many URLs shortened per month? → 100 million
- **Reads vs Writes**: What's the ratio? → 100:1 (read-heavy)
- **Custom URLs**: Can users choose custom aliases? → Yes, optional
- **Expiration**: Do URLs expire? → Yes, after 10 years
- **Analytics**: What level of tracking? → Basic click counts

### 3. Requirements

**Functional:**
1. Shorten long URL → short URL
2. Redirect short URL → original URL
3. Optional custom aliases
4. Track click counts
5. URLs expire after 10 years

**Non-Functional:**
- Scale: 100M URLs/month, 10B redirects/month
- Performance: < 50ms redirect latency
- Availability: 99.9%
- Consistency: Eventual consistency acceptable

### 4. Capacity Estimation

**Traffic:**
- Writes: 100M/month = 40/sec (avg), 120/sec (peak)
- Reads: 10B/month = 4,000/sec (avg), 12,000/sec (peak)

**Storage:**
- Per URL: ~300 bytes (original URL, short code, metadata)
- 10 years: 12B URLs × 300 bytes = 3.6 TB ✓

**Bandwidth:**
- Incoming: 40/sec × 300 bytes = 12 KB/sec
- Outgoing: 4,000/sec × 300 bytes = 1.2 MB/sec

**Cache:**
- Cache 20% hot URLs: 2.4B URLs × 300 bytes = 720 GB

### 5. Core Use Cases

**Use Case 1: Shorten URL**
1. User submits long URL
2. System validates URL format
3. System generates unique short code
4. System stores mapping in database
5. System returns short URL to user

**Use Case 2: Redirect**
1. User visits short URL
2. System extracts short code
3. System checks cache for mapping
4. If miss, query database
5. System redirects to original URL
6. (Async) Increment click counter

### 6. Core Entities

**URL Mapping:**
- `short_code` (string, unique, indexed)
- `long_url` (string)
- `user_id` (integer, optional)
- `created_at` (timestamp)
- `expires_at` (timestamp)
- `clicks` (integer)

### 7. Success Metrics
- Average redirect latency < 50ms
- Cache hit rate > 80%
- 99.9% uptime
- Zero data loss

### 8. Constraints & Bottlenecks
- High read traffic → Need caching
- Database read scalability → Need replicas
- Short code generation → Need collision handling

### 9. Prioritization

**P0:**
- Shorten URL
- Redirect URL
- Basic storage

**P1:**
- Click tracking
- Custom aliases

**P2:**
- Advanced analytics
- User accounts

### 10. Out of Scope
- QR code generation
- Link editing after creation
- Advanced analytics dashboard
- A/B testing features

---

## Usage Tips

1. **Start with clarifying questions** - Don't assume requirements
2. **Write down assumptions** - Make them explicit
3. **Use real numbers** - Estimates help guide design decisions
4. **Be specific** - "Fast" is vague; "< 100ms" is measurable
5. **Identify trade-offs early** - Consistency vs availability, etc.
6. **Keep it realistic** - Consider budget, time, and team constraints
7. **Review with stakeholders** - Confirm understanding before designing

---

## Next Steps

After completing this analysis:
1. Move to [Architecture Diagram Guide](./architecture-diagram-guide.md) to design the system
2. Use [Component Selection Matrix](./component-selection-matrix.md) to choose technologies
3. Apply [Trade-off Analysis Framework](./trade-off-analysis-framework.md) for design decisions
