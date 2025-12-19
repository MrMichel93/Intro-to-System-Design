# System Design Interview Preparation

Complete guide to acing system design interviews.

## Interview Structure

### Typical 45-60 Minute Interview

**Minutes 0-5: Problem Clarification**
- Ask clarifying questions
- Understand requirements
- Define scope

**Minutes 5-10: Requirements & Estimation**
- Functional requirements
- Non-functional requirements
- Back-of-envelope calculations

**Minutes 10-20: High-Level Design**
- Draw architecture diagram
- Identify major components
- Explain data flow

**Minutes 20-40: Deep Dive**
- Detailed component design
- Database schema
- API design
- Handle interviewer's questions

**Minutes 40-45: Trade-offs & Scaling**
- Discuss bottlenecks
- Explain trade-offs
- Scaling strategies

**Minutes 45-60: Q&A**
- Answer follow-up questions
- Discuss alternatives

## Common Interview Questions

### Easy
1. Design a URL Shortener (bit.ly)
2. Design a Pastebin
3. Design a Key-Value Store
4. Design a Rate Limiter

### Medium
5. Design Twitter
6. Design Instagram
7. Design Netflix
8. Design Uber
9. Design WhatsApp
10. Design YouTube

### Hard
11. Design Google Search
12. Design Gmail
13. Design Amazon
14. Design Facebook News Feed

## Interview Framework (RADIO)

### R - Requirements
**Functional:** What should the system do?
**Non-functional:** Scale, performance, availability

### A - Architecture
**High-level design:** Components and their interactions

### D - Data Model
**Database schema:** How data is structured and stored

### I - Interface
**API design:** How clients interact with the system

### O - Optimizations
**Trade-offs:** Bottlenecks, scaling, improvements

## Key Questions to Ask

### Scope & Scale
- How many users?
- How many requests per second?
- Read-to-write ratio?
- Geographic distribution?
- Data retention period?

### Features
- Core features vs nice-to-have?
- Real-time requirements?
- Consistency requirements?
- Analytics needed?

### Constraints
- Latency requirements?
- Budget constraints?
- Technology preferences?
- Team size and expertise?

## Common Mistakes to Avoid

❌ **Jumping to solution** - Clarify requirements first  
❌ **Ignoring scale** - Always consider numbers  
❌ **Over-engineering** - Start simple, then optimize  
❌ **Ignoring trade-offs** - Everything has pros/cons  
❌ **Not asking questions** - Show you think critically  
❌ **Poor communication** - Explain your thinking  
❌ **Focusing only on happy path** - Consider failures  
❌ **Not considering costs** - Resources aren't free  

## What Interviewers Look For

✅ **Problem-solving** - Can you break down complex problems?  
✅ **Trade-off analysis** - Do you understand pros/cons?  
✅ **Communication** - Can you explain clearly?  
✅ **Practical knowledge** - Have you built systems?  
✅ **Scalability thinking** - Can you handle growth?  
✅ **Depth** - Can you go deep when asked?  
✅ **Breadth** - Do you know many technologies?  

## Practice Strategy

### Week 1-2: Fundamentals
- Study all course modules
- Focus on trade-offs
- Practice calculations

### Week 3-4: Easy Problems
- URL Shortener
- Pastebin
- Key-Value Store

### Week 5-6: Medium Problems
- Twitter, Instagram
- Netflix, YouTube
- Uber, WhatsApp

### Week 7-8: Mock Interviews
- Practice with peers
- Record yourself
- Get feedback

## Sample Problem Walkthrough

See [Sample Interview](./sample-interview.md) for a complete walkthrough of designing Twitter.

## Resources

- Practice problems: [Detailed scenarios](./practice-problems.md)
- Cheat sheets: [Quick reference guide](../resources/cheat-sheets.md)
- Common patterns: [Reusable solutions](../design-templates/)

## Tips for Success

1. **Think out loud** - Interviewers want to understand your thought process
2. **Start high-level** - Don't dive into details immediately
3. **Use diagrams** - Draw boxes and arrows
4. **Manage time** - Don't spend 40 minutes on requirements
5. **Be flexible** - Adapt when interviewer steers you
6. **Know trade-offs** - Every decision has pros/cons
7. **Practice** - Do many mock interviews

Good luck with your interviews!
