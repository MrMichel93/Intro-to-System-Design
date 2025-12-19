# CAP Theorem Design Exercises

Practice applying CAP theorem concepts to real-world scenarios!

## Exercise 1: Choose CAP for Different Systems

For each system below, determine whether it should be CP or AP and explain why.

### System A: Hotel Booking Platform
- Users book rooms
- Must prevent double-booking
- High season = lots of concurrent bookings
- International travelers across time zones

**Your Answer:**
- CAP Choice: __________
- Reasoning: __________

<details>
<summary>Solution</summary>

**CP (Consistency + Partition Tolerance)**

**Reasoning:**
- Double-booking is catastrophic (angry customers, reputation damage)
- Must guarantee no room is booked twice
- Brief unavailability during booking better than wrong data
- Financial and legal implications
- Users will wait a few extra seconds for confirmation

**Implementation:**
- Strong consistency for room availability
- Lock room during booking process
- Confirm only when all replicas agree
- Show "please wait" during confirmation

</details>

---

### System B: Twitter-like Social Network
- Users post short messages
- Users view feeds of people they follow
- Millions of posts per hour
- Global user base

**Your Answer:**
- CAP Choice: __________
- Reasoning: __________

<details>
<summary>Solution</summary>

**AP (Availability + Partition Tolerance)**

**Reasoning:**
- User frustration from downtime > seeing delayed posts
- Tweet delay of a few seconds is acceptable
- High volume requires high availability
- Eventual consistency is fine for social content
- Business model depends on uptime

**Implementation:**
- Async replication across datacenters
- Eventual consistency (seconds to minutes)
- Users might see different feeds temporarily
- Accept all writes immediately
- Sync in background

</details>

---

### System C: Stock Trading Platform
- Real-time stock prices
- Buy/sell orders
- High-frequency trading
- Money directly involved

**Your Answer:**
- CAP Choice: __________
- Reasoning: __________

<details>
<summary>Solution</summary>

**CP (Consistency + Partition Tolerance)**

**Reasoning:**
- Financial transactions must be consistent
- Wrong price = legal issues, lost money
- Order execution must be guaranteed correct
- Regulatory requirements for accuracy
- Traders accept brief unavailability over wrong data

**Implementation:**
- Synchronous replication for orders
- Strong consistency for account balances
- Reject orders during partitions if can't guarantee consistency
- Clear error messages
- Audit trails

</details>

---

## Exercise 2: Design for Network Partition

### Scenario: Global E-commerce Platform

You run an e-commerce site with datacenters in:
- US West Coast
- US East Coast  
- Europe
- Asia

A network partition occurs: Asia datacenter loses connection to others.

**Your Task:** Design how your system should behave for each type of data:

1. **Product Catalog** (descriptions, images, prices)
2. **Inventory Count** (items in stock)
3. **Shopping Carts** (items users added)
4. **Order Processing** (completed purchases)
5. **User Reviews** (ratings and comments)

For each, decide:
- Continue accepting writes? (Availability)
- Require full consistency? (Consistency)
- How to handle the partition?

<details>
<summary>Sample Solution</summary>

**1. Product Catalog → AP**
```
Behavior during partition:
- Asia continues showing products
- Allow slight staleness in prices/descriptions
- Sync when partition heals
Reasoning: Better to show slightly old product than error page
```

**2. Inventory Count → CP**
```
Behavior during partition:
- Asia cannot sell items (only datacenters with majority)
- Show "Currently unavailable" for purchases
- Display products but disable "Add to Cart"
Reasoning: Overselling is worse than temporary unavailability
```

**3. Shopping Carts → AP**
```
Behavior during partition:
- Continue accepting cart additions
- Store locally in Asia
- Sync when partition heals
- Worst case: Duplicate items (user can remove)
Reasoning: Don't lose potential sales
```

**4. Order Processing → CP**
```
Behavior during partition:
- Cannot process orders without full consistency
- Show "Payment system temporarily unavailable"
- Queue orders to process when partition heals
Reasoning: Financial transactions must be consistent
```

**5. User Reviews → AP**
```
Behavior during partition:
- Accept new reviews
- Show existing reviews (might be slightly stale)
- Sync when partition heals
Reasoning: Reviews aren't critical, eventual consistency fine
```

</details>

---

## Exercise 3: Trade-Off Analysis

### Scenario: Messaging App Design

You're designing a WhatsApp-like messaging app. Analyze the CAP trade-offs for different features:

**Features:**
1. Sending messages
2. Reading message history
3. Group chat
4. "Read receipts" (showing message was read)
5. Contact list sync

For each feature, choose CP or AP and explain trade-offs.

<details>
<summary>Solution</summary>

**1. Sending Messages → AP**
```
Choice: Availability
- Message accepted immediately
- Delivered asynchronously
- Eventual delivery to recipient
Trade-off: Might be delayed during partition, but user can keep sending
```

**2. Reading History → AP**
```
Choice: Availability
- Always show some messages (from local cache)
- Might not be complete during partition
- Sync when connection restored
Trade-off: Incomplete history better than no history
```

**3. Group Chat → AP**
```
Choice: Availability
- Messages delivered eventually to all members
- Members might see different message orders temporarily
- Converges to consistent state
Trade-off: Temporary inconsistency acceptable for groups
```

**4. Read Receipts → AP**
```
Choice: Availability
- Receipt might be delayed
- Not critical feature
- Eventually shows correct status
Trade-off: Delayed receipt better than disabling feature
```

**5. Contact List → AP with periodic sync**
```
Choice: Availability
- Local cache of contacts always available
- Sync changes when connected
- Offline mode works
Trade-off: Might show outdated status, but usable
```

**Overall Strategy**: Heavy AP bias because messaging must "just work" even with poor connectivity.

</details>

---

## Exercise 4: Hybrid Approach

### Scenario: Healthcare Patient Records System

Design a system that uses both CP and AP approaches appropriately.

**Requirements:**
- Patient demographics (name, DOB, insurance)
- Vital signs history (heart rate, blood pressure)
- Prescription orders (active medications)
- Lab results (blood tests, imaging)
- Appointment scheduling

**Your Task:** 
- Classify each data type as CP or AP
- Justify your decisions
- Design how they interact

<details>
<summary>Solution</summary>

**CP Data (Consistency Critical):**

**Prescription Orders:**
- Must be 100% accurate
- Wrong medication = patient harm
- Brief unavailability acceptable
- Implementation: Synchronous replication, all nodes must confirm

**Active Medications:**
- Critical for drug interactions
- Doctors need current list
- System waits for consensus
- Implementation: Majority quorum required

**AP Data (Availability Critical):**

**Vital Signs History:**
- Historical data (not life-critical)
- Can tolerate brief inconsistency
- Important to always access
- Implementation: Async replication, eventual consistency

**Lab Results:**
- Append-only (results don't change)
- Eventually consistent is fine
- Must be available for review
- Implementation: Write locally, sync background

**Hybrid Approach:**

**Appointment Scheduling:**
- Write: CP (prevent double-booking)
- Read: AP (view schedule even if slightly stale)
- Implementation: Strong consistency for booking, eventual for viewing

**Patient Demographics:**
- Updates: CP (must be correct across system)
- Reads: AP with cache (can show slightly stale)
- Implementation: Write to majority, read from cache

**System Architecture:**
```
Critical Path (CP):
[Prescriptions] → [Strong Consistency DB] → [All replicas confirm]

Non-Critical Path (AP):
[Vital Signs] → [Local DB] → [Async Sync] → [Other sites]

Hybrid:
[Demographics] → [Write: CP] → [Read: AP with cache]
```

**Emergency Mode:**
- During partition, critical data (prescriptions) unavailable
- Historical data (vitals) still accessible
- Clear UI indication of what's unavailable
- Queue critical updates for when partition heals

</details>

---

## Exercise 5: Cost-Benefit Analysis

### Scenario: Startup Budget Decisions

You're a startup with limited budget. Analyze the costs of CP vs. AP for your needs.

**Your Company:**
- Music streaming service
- 100K users (growing to 1M in year)
- Songs, playlists, user profiles
- Limited funding

**Decision Points:**

1. **User Playlists Storage**
   - CP: Strong consistency (expensive, complex)
   - AP: Eventual consistency (cheaper, simpler)

2. **Play Count/Statistics**
   - CP: Accurate real-time counts
   - AP: Eventually accurate counts

3. **User Authentication**
   - CP: Always consistent
   - AP: Eventual consistency

**Your Task:** Choose CP or AP for each, considering startup constraints.

<details>
<summary>Solution</summary>

**1. User Playlists → AP**
```
Decision: Eventual Consistency
Reasoning:
- Lower infrastructure costs ($1,000/mo vs $5,000/mo)
- Simpler to implement (faster time to market)
- User won't notice few-second delay in sync
- Can upgrade to CP later if needed
Savings: $48,000/year
```

**2. Play Count → AP**
```
Decision: Eventual Consistency
Reasoning:
- Real-time accuracy not critical for play counts
- Users don't notice if count off by few plays
- Significantly cheaper at scale
- Standard industry practice
Savings: $24,000/year on analytics infrastructure
```

**3. User Authentication → CP**
```
Decision: Strong Consistency
MUST HAVE: $5,000/year extra cost
Reasoning:
- Security cannot be compromised
- Wrong auth = major security breach
- Login inconsistency = bad user experience
- Worth the cost
Cost: Worth it for user trust
```

**Total Strategy:**
- CP for authentication only (critical)
- AP for everything else (cost-effective)
- Migrate to hybrid approach as you grow
- Initial savings: ~$70,000/year
- Use savings for marketing/growth

**Growth Plan:**
- Year 1: AP for most features (survive)
- Year 2: Hybrid approach (optimize)
- Year 3+: CP where valuable (mature)

</details>

---

## Reflection Questions

1. **When would you choose CP even if it costs more?**
   - Think about data types where correctness is critical

2. **Can you think of a system that should be CA (not distributed)?**
   - When is a single server acceptable?

3. **How would you explain CAP to a non-technical person?**
   - Practice explaining without jargon

4. **What metrics would you monitor to detect when CAP trade-offs affect users?**
   - Replication lag, consistency violations, availability metrics

---

## Summary

Key lessons from these exercises:

✅ **Context matters** - CP vs AP depends on your specific needs  
✅ **Hybrid approaches** are common - different data, different choices  
✅ **Business impact** - consider cost, revenue, user experience  
✅ **Start simple** - can always add complexity later  
✅ **Monitor and adapt** - real-world data guides decisions  

## Next Steps

Continue to [Trade-Offs](./trade-offs.md) to explore more nuanced decisions in CAP theorem!
