# Introduction to System Design

## 1. Introduction

### What is System Design?

System design is the art and science of architecting software systems that work at scale. Think of it as being an architect for buildings, but instead of designing physical structures, you're designing how millions of users can simultaneously use an application like Instagram, stream videos on Netflix, or order rides on Uber without the system crashing.

At its core, system design answers questions like:
- How do we store billions of photos efficiently?
- How can we deliver videos to users around the world instantly?
- How do we ensure your messages are delivered even when servers fail?
- How do we handle millions of requests per second?

System design is about understanding the **big picture** — connecting databases, servers, caching layers, load balancers, and other components to create a cohesive, functioning system.

### Why Does System Design Matter?

**Career Impact**: System design is one of the most valuable skills in tech. Companies like Google, Meta, Amazon, and Netflix dedicate entire teams to designing systems that serve billions of users. Understanding system design makes you a more valuable engineer.

**Interview Relevance**: System design interviews are a critical part of senior engineering roles. Companies want to see how you think about scaling problems, make trade-offs, and design robust systems. This course prepares you for those conversations.

**Real-World Impact**: The systems you'll learn to design power the applications you use every day. Understanding how they work gives you insight into:
- Why some apps load instantly while others lag
- How services stay online during massive traffic spikes
- Why certain features are built one way and not another
- How to build applications that scale from 100 to 100 million users

### What Makes a System "Good"?

There's no such thing as a perfect system — only systems that are **appropriate for their requirements**. A "good" system depends entirely on what you're trying to achieve.

Key characteristics to consider:
- **Scalability**: Can it handle growth from 1,000 to 1 million users?
- **Reliability**: Does it work consistently, even when things fail?
- **Performance**: Is it fast enough for users?
- **Maintainability**: Can developers easily update and fix it?
- **Cost-Efficiency**: Does it use resources wisely?

**The Trade-off Mindset**: Every design decision involves trade-offs. You can't have everything. Want faster reads? You might sacrifice consistency. Need strong consistency? You might give up some availability. Understanding these trade-offs is the heart of system design.

## 2. Who This Course Is For

### Prerequisites

Before starting this course, you should be comfortable with:

**Programming Fundamentals:**
- Variables, functions, loops, and conditionals
- Basic data structures (arrays, hash maps, lists)
- Understanding of how programs execute

**Basic Web Knowledge:**
- What happens when you visit a website
- Concept of client-server architecture
- Basic HTTP (GET/POST requests)
- What databases do (even if you haven't used one deeply)

**Not Required But Helpful:**
- Experience with any programming language (Python, JavaScript, Java, etc.)
- Basic understanding of databases
- General curiosity about how large applications work

### What You'll Be Able to Do After This Course

By the end of this course, you will:

✅ **Design systems from scratch** - Take requirements and create architectural diagrams for real-world applications

✅ **Make informed trade-off decisions** - Understand when to use SQL vs NoSQL, caching vs database queries, synchronous vs asynchronous processing

✅ **Estimate system capacity** - Calculate storage needs, bandwidth requirements, and server capacity for millions of users

✅ **Communicate technical decisions** - Explain your design choices clearly in interviews and team discussions

✅ **Recognize patterns** - Identify common system design patterns in apps you use daily

✅ **Ace system design interviews** - Confidently approach design questions at top tech companies

### Self-Assessment Quiz: Are You Ready?

Answer these questions honestly to check your readiness:

1. **Can you write a simple program** (in any language) that reads user input and processes it?
   - Yes → You're good! · No → Brush up on basic programming first

2. **Do you understand what a database is** and why applications use them?
   - Yes → Great! · No → Read about databases basics, then come back

3. **Have you used web applications** like social media, streaming services, or e-commerce?
   - Yes → Perfect! · No → Start exploring popular apps

4. **Are you comfortable with basic math** (multiplication, division, working with large numbers)?
   - Yes → Excellent! · No → Practice basic calculations — you'll need them

5. **Are you curious about how things work** at scale?
   - Yes → You'll love this course! · No → Consider if this is the right time

**If you answered "Yes" to questions 1, 2, and 4**, you're ready to start!

## 3. Course Philosophy

### Learning by Designing Real Systems

You won't just read about concepts — you'll **design actual systems**. Each module includes hands-on exercises where you design systems like:
- A URL shortener (like bit.ly)
- A social media feed (like Twitter)
- A video streaming platform (like Netflix)
- A ride-sharing service (like Uber)

By designing these systems yourself, you'll develop intuition for how components fit together.

### Understanding Trade-offs Over Memorizing Solutions

**There are no "right answers" in system design** — only trade-offs. This course teaches you to:
- Identify different approaches to the same problem
- Understand the pros and cons of each approach
- Choose the approach that best fits your requirements
- Articulate why you made specific decisions

We emphasize **thinking over memorization**. You'll learn frameworks for thinking through problems, not templates to copy.

### Thinking at Scale

System design is fundamentally about scale:
- What works for 100 users often breaks for 100,000 users
- What works for 100,000 users requires rethinking for 100 million users

You'll develop the skill of asking: "What happens when this scales 10x? 100x? 1000x?"

### No "Perfect" Design — Only Appropriate Designs

A perfect design for Instagram wouldn't work for WhatsApp, even though both are social applications. Context matters:
- User requirements vary
- Budget constraints differ  
- Scale needs change over time
- Teams have different expertise

You'll learn to design systems that are **appropriate** for their specific context, requirements, and constraints.

## 4. Course Structure

The course flows from foundations to advanced topics to practical application:

```
┌─────────────────────────────────────────────────────────────────────┐
│                         SYSTEM DESIGN JOURNEY                        │
└─────────────────────────────────────────────────────────────────────┘

    FOUNDATIONS                    Build Your Base
    ┌──────────┐
    │  Core    │ → What is a system? Requirements gathering
    │ Concepts │ → Capacity estimation & calculations
    │  (0-2)   │ → CAP Theorem & distributed system basics
    └────┬─────┘
         │
         ▼
    THEORY & CONCEPTS             Understand the Building Blocks
    ┌──────────┐
    │ Scaling  │ → Scalability principles
    │   &      │ → Load balancing strategies
    │ Patterns │ → Caching mechanisms
    │  (3-6)   │ → Database design fundamentals
    └────┬─────┘
         │
         ▼
    COMPONENTS                    Master Individual Parts
    ┌──────────┐
    │Advanced  │ → Data partitioning & sharding
    │Components│ → Replication strategies
    │   &      │ → Consistency patterns
    │ Systems  │ → API design principles
    │ (7-10)   │
    └────┬─────┘
         │
         ▼
    ARCHITECTURE                  Assemble Complete Systems
    ┌──────────┐
    │ System   │ → Message queues
    │ Patterns │ → Microservices architecture
    │    &     │ → Availability patterns
    │ Services │ → CDN & content delivery
    │ (11-14)  │
    └────┬─────┘
         │
         ▼
    PRACTICE                      Apply Everything
    ┌──────────┐
    │  Case    │ → Real-world systems (Twitter, Netflix, etc.)
    │ Studies  │ → Design templates & patterns
    │    &     │ → Interview preparation
    │Templates │ → Build your own designs
    └──────────┘
```

**Module Organization**: Each module contains:
- **README.md**: Core concepts with clear explanations
- **examples.md**: Real-world examples and case studies
- **design-exercise.md**: Hands-on practice problems
- **trade-offs.md**: Deep dive into design decisions

## 5. Learning Paths

Choose the path that fits your goals. All paths cover the same core material but with different focus and pacing.

### Path 1: Complete Beginner Path
**Goal**: Build comprehensive understanding from scratch  
**Duration**: 8-10 weeks (4-5 hours/week)  
**Recommended for**: Students new to system design, those with time to learn deeply

**Week 1-2: Foundations**
- Module 0: System Design Foundations
- Module 1: Back-of-Envelope Calculations
- Module 2: CAP Theorem

**Week 3-4: Core Concepts**
- Module 3: Scalability
- Module 4: Load Balancing
- Module 5: Caching
- Module 6: Database Design

**Week 5-6: Advanced Components**
- Module 7: Data Partitioning
- Module 8: Replication
- Module 9: Consistency Patterns
- Module 10: API Design

**Week 7-8: System Architecture**
- Module 11: Message Queues
- Module 12: Microservices
- Module 13: Availability Patterns
- Module 14: CDN

**Week 9-10: Practice & Mastery**
- Work through all case studies
- Complete design exercises
- Use design templates for practice systems

### Path 2: Interview Prep Path
**Goal**: Prepare for system design interviews efficiently  
**Duration**: 4-6 weeks (6-8 hours/week)  
**Recommended for**: Those preparing for interviews, experienced developers

**Week 1: Essential Foundations**
- Module 0: System Design Foundations (focus on requirements gathering)
- Module 1: Back-of-Envelope Calculations (critical for interviews!)
- Module 2: CAP Theorem (know the trade-offs)

**Week 2: Must-Know Components**
- Module 3: Scalability
- Module 4: Load Balancing
- Module 5: Caching
- Module 6: Database Design

**Week 3: Advanced Topics (Most Asked)**
- Module 7: Data Partitioning
- Module 10: API Design
- Module 11: Message Queues
- Module 13: Availability Patterns

**Week 4-6: Practice & Case Studies**
- Twitter case study
- Instagram case study
- Uber case study
- Netflix case study
- Interview prep module
- Mock design exercises

**Interview Focus Areas**: Capacity estimation, scalability patterns, database choices, caching strategies, handling failures.

### Path 3: Deep Dive Path  
**Goal**: Become a system design expert  
**Duration**: 12-15 weeks (5-7 hours/week)  
**Recommended for**: Those pursuing careers in distributed systems, architects-in-training

Follow the Complete Beginner Path, but add:
- Deep study of all trade-offs documents
- Complete all design exercises (not just reading solutions)
- Build real implementations of simple systems
- Research additional resources for each topic
- Study all 6+ case studies in depth
- Design 3-5 systems entirely on your own
- Compare your designs with provided solutions

**Time Commitment Summary**:
- Complete Beginner: 32-50 hours total over 8-10 weeks
- Interview Prep: 24-48 hours total over 4-6 weeks  
- Deep Dive: 60-105 hours total over 12-15 weeks

## 6. How to Use This Course

Follow this process for each module to maximize learning:

### Step 1: Read the Lesson
- Start with the module's README.md
- Take notes on key concepts
- Don't worry if you don't understand everything immediately
- Focus on understanding the "why" behind each concept

### Step 2: Study Real-World Examples
- Read through examples.md
- See how the concept applies to real applications
- Think about apps you use — do you recognize these patterns?

### Step 3: Draw Diagrams
**This is crucial!** Drawing forces you to think clearly.

- Sketch the system architecture by hand or use digital tools
- Label all components (databases, servers, caches, etc.)
- Show data flow with arrows
- Don't aim for perfection — aim for understanding

**Recommended approach**: 
- First attempt: Draw without looking at solutions
- Second attempt: Compare with examples and redraw
- Third attempt: Explain your diagram out loud

### Step 4: Complete Design Exercises
- Attempt the design-exercise.md problems yourself first
- Spend at least 30 minutes on your own solution
- Write down your assumptions and requirements
- Sketch your architecture

### Step 5: Compare with Sample Solutions
- After attempting on your own, review provided solutions
- Note differences between your approach and the sample
- Understand why different choices were made
- There's no single "right" answer — focus on trade-offs

### Step 6: Reflect on Trade-offs
- Read the trade-offs.md document
- For each decision in the design, ask:
  - What are we optimizing for?
  - What are we sacrificing?
  - When would we choose differently?
- Write down insights in your own words

**Pro Tip**: Keep a design journal where you note key insights, common patterns you notice, and questions that arise.

## 7. What You'll Build

Throughout this course, you'll design systems of increasing complexity. Here's your progression:

### Level 1: Foundation Projects (Weeks 1-3)
**1. URL Shortener (like bit.ly)**
- Learn: Basic database design, hashing, simple scaling
- Complexity: Level 1/5 (Beginner)
- Key Concepts: ID generation, redirects, analytics

**2. Pastebin (code snippet sharing)**
- Learn: Text storage, caching, expiration policies
- Complexity: Level 1/5 (Beginner)
- Key Concepts: TTL, content delivery, basic security

### Level 2: Intermediate Projects (Weeks 4-6)
**3. Twitter/X (social media timeline)**
- Learn: Feed generation, fan-out patterns, real-time updates
- Complexity: Level 3/5 (Intermediate)
- Key Concepts: Write patterns, read patterns, graph data

**4. Instagram (photo sharing platform)**
- Learn: Media storage, CDN usage, feed ranking
- Complexity: Level 3/5 (Intermediate)
- Key Concepts: Object storage, image optimization, social graphs

**5. Messaging System (like WhatsApp)**
- Learn: Real-time communication, delivery guarantees, encryption
- Complexity: Level 4/5 (Advanced)
- Key Concepts: WebSockets, message queues, presence systems

### Level 3: Advanced Projects (Weeks 7-10)
**6. Video Streaming Platform (like Netflix)**
- Learn: Video encoding, adaptive bitrate, global distribution
- Complexity: Level 4/5 (Advanced)
- Key Concepts: CDN strategy, transcoding pipelines, recommendations

**7. Ride Sharing Service (like Uber)**
- Learn: Geospatial indexing, real-time matching, surge pricing
- Complexity: Level 5/5 (Expert)
- Key Concepts: Location services, distributed matching, payment systems

**8. E-commerce Platform (like Amazon)**
- Learn: Inventory management, order processing, payment flows
- Complexity: Level 5/5 (Expert)
- Key Concepts: Transactions, consistency, search systems

### Progression Strategy
- **Weeks 1-3**: Master simple systems with 1-2 components
- **Weeks 4-6**: Handle moderate complexity with multiple interacting services
- **Weeks 7-10**: Design complex, multi-region, highly-available systems

Each project builds on previous knowledge, gradually increasing sophistication.

## 8. Tools You'll Need

### Diagramming Tools (Choose One or More)

**Free Online Tools:**
- **[Excalidraw](https://excalidraw.com/)** (Recommended for beginners)
  - Hand-drawn style, simple interface
  - No account needed, works in browser
  - Great for quick sketches

- **[draw.io / diagrams.net](https://draw.io/)** (Recommended for detailed diagrams)
  - Professional-looking diagrams
  - Extensive shape libraries
  - Free and open source

- **[LucidChart](https://www.lucidchart.com/)** (Free tier available)
  - Collaborative features
  - Templates for system design
  - Clean, professional output

**Offline Options:**
- **Paper and Pencil** (Seriously!)
  - Best for learning and interviews
  - Forces you to think before drawing
  - No learning curve

- **Whiteboard**
  - Great for practice sessions
  - Mimics interview conditions
  - Easy to iterate

**ASCII Diagrams** (for documentation):
- Many examples in this course use ASCII art
- Great for README files and documentation
- Works everywhere, no special tools needed

### Calculator for Capacity Estimation

You'll need basic calculation abilities for:
- Storage requirements (TB, PB)
- Bandwidth estimates (Mbps, Gbps)
- QPS (Queries Per Second)
- User capacity planning

**Options:**
- Computer/phone calculator app
- Spreadsheet software (Excel, Google Sheets) for complex calculations
- Python REPL for quick calculations

### Note-Taking System

**Organized notes are essential for system design learning.**

Choose a system that works for you:

**Digital Options:**
- **Notion** - Great for organizing modules and concepts
- **Obsidian** - Excellent for linking related concepts
- **Google Docs** - Simple and accessible
- **Markdown files** - Version-controllable, portable

**Physical Options:**
- **Notebook** - Write by hand to reinforce learning
- **Index cards** - Great for reviewing key concepts

**What to capture:**
- Key concepts and definitions
- Trade-off decisions and reasoning
- Common patterns you notice
- Questions to research further
- Your design attempts and what you learned

## 9. Success Metrics

### How to Know You're Making Progress

System design learning is gradual. Here are signs you're on the right track:

**Week 2-3 Indicators:**
- ✅ You can explain what scalability means to a friend
- ✅ You understand the difference between vertical and horizontal scaling
- ✅ You can estimate basic storage needs for a small application
- ✅ You're starting to notice design patterns in apps you use

**Week 4-6 Indicators:**
- ✅ You can sketch a basic architecture diagram with 3-5 components
- ✅ You understand when to use caching vs direct database queries
- ✅ You can articulate at least one trade-off for any design decision
- ✅ You're thinking "how would this scale?" when using applications

**Week 7-10 Indicators:**
- ✅ You can design a complete system from requirements to architecture
- ✅ You can compare two different approaches and explain trade-offs
- ✅ You can estimate capacity needs (storage, bandwidth, QPS) confidently
- ✅ You can explain your designs clearly to others
- ✅ You recognize common patterns across different systems

### Self-Assessment Checkpoints

**After Module 6 (Week 4):**
Try designing a URL shortener without looking at solutions. Can you:
- [ ] Gather requirements?
- [ ] Estimate capacity?
- [ ] Draw the architecture?
- [ ] Choose database schema?
- [ ] Explain trade-offs?

If yes to 4+, you're progressing well!

**After Module 10 (Week 6):**
Try designing Twitter's timeline feature. Can you:
- [ ] Design for both read-heavy and write-heavy patterns?
- [ ] Explain caching strategy?
- [ ] Handle high traffic?
- [ ] Discuss consistency trade-offs?
- [ ] Consider failure scenarios?

If yes to 4+, you're ready for advanced topics!

**After Module 14 (Week 8):**
Design a complete system (Instagram or Uber) from scratch. Can you:
- [ ] Create comprehensive requirements?
- [ ] Calculate all capacity estimates?
- [ ] Design full architecture with 8+ components?
- [ ] Explain every component choice?
- [ ] Identify 3+ critical trade-offs?
- [ ] Discuss failure handling?
- [ ] Consider monitoring and operations?

If yes to 6+, you've mastered the fundamentals!

**Red Flags** (If these happen, review earlier modules):
- ❌ You can't explain why you chose a particular component
- ❌ You're memorizing architectures without understanding trade-offs
- ❌ You can't estimate basic capacity needs
- ❌ You skip drawing diagrams
- ❌ You can't identify what breaks when scaling 10x

### Continuous Improvement

**Weekly Practice:**
- Design one small system (30-60 minutes)
- Review one case study in depth
- Explain one concept to someone else (or write it down)

**Monthly Check:**
- Redesign a system you worked on 4 weeks ago — how has your approach improved?
- Compare your early designs to current ones — what's different?

**Remember**: System design expertise develops over months and years. Consistent practice matters more than speed.

---

## 🗂️ Course Modules

### Foundations
0. **[System Design Foundations](./00-foundations/)** - Core concepts and requirements gathering
1. **[Back-of-Envelope Calculations](./01-back-of-envelope/)** - Capacity estimation and planning
2. **[CAP Theorem](./02-cap-theorem/)** - Understanding distributed system trade-offs

### Core Concepts  
3. **[Scalability](./03-scalability/)** - Designing systems that grow
4. **[Load Balancing](./04-load-balancing/)** - Distributing traffic effectively
5. **[Caching](./05-caching/)** - Optimizing performance with caching strategies
6. **[Database Design](./06-database-design/)** - Structuring and scaling data storage

### Advanced Components
7. **[Data Partitioning](./07-data-partitioning/)** - Sharding and horizontal partitioning
8. **[Replication](./08-replication/)** - Data redundancy and reliability
9. **[Consistency Patterns](./09-consistency-patterns/)** - Managing data consistency
10. **[API Design](./10-api-design/)** - Building clean, maintainable interfaces

### System Architecture
11. **[Message Queues](./11-message-queues/)** - Asynchronous communication patterns
12. **[Microservices](./12-microservices/)** - Service-oriented architecture
13. **[Availability Patterns](./13-availability-patterns/)** - High availability and failover
14. **[CDN](./14-cdn/)** - Global content delivery

### Practice & Application
- **[Case Studies](./case-studies/)** - Real-world system designs (Twitter, Netflix, Uber, WhatsApp, Instagram, YouTube)
- **[Design Templates](./design-templates/)** - Reusable patterns and frameworks
- **[Interview Prep](./interview-prep/)** - Interview strategies and practice
- **[Resources](./resources/)** - Additional learning materials

---

## 🚀 Ready to Begin?

Start your system design journey with **[Module 0: System Design Foundations](./00-foundations/)**

Remember: System design is a skill that develops with practice. Be patient with yourself, focus on understanding trade-offs, and enjoy the process of learning how the world's most interesting systems work!

---

## 📝 License

This project is open source and available for educational purposes.

## 🤝 Contributing

Found an issue or have suggestions? Contributions are welcome! Please open an issue or submit a pull request.
