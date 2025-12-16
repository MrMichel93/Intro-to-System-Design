# Scalability - Challenges & Questions

Test your understanding of scalability concepts with these challenges!

## 🎯 Multiple Choice Questions

### Question 1
You have a web application running on a single server with 4GB RAM. The application is getting slower as more users join. What is the FIRST step you should take?

A) Immediately add 10 more servers  
B) Monitor metrics to identify the bottleneck  
C) Rewrite the entire application  
D) Delete user accounts to reduce load  

<details>
<summary>Answer</summary>

**B) Monitor metrics to identify the bottleneck**

Before making changes, you need to understand what's causing the slowdown. It could be CPU, memory, disk I/O, or database queries. Monitoring helps you make informed decisions.
</details>

---

### Question 2
Which statement is TRUE about vertical scaling?

A) It requires load balancers  
B) It has unlimited scaling potential  
C) It's simpler to implement than horizontal scaling  
D) It prevents single points of failure  

<details>
<summary>Answer</summary>

**C) It's simpler to implement than horizontal scaling**

Vertical scaling just means upgrading hardware, which is straightforward but has physical limits. Horizontal scaling is more complex but offers better long-term scalability.
</details>

---

### Question 3
Your gaming app has 1,000 users during weekdays but 50,000 users on weekends. What's the best scaling strategy?

A) Keep servers for 50,000 users running 24/7  
B) Use auto-scaling to add servers on weekends  
C) Only support 1,000 users  
D) Vertical scaling only  

<details>
<summary>Answer</summary>

**B) Use auto-scaling to add servers on weekends**

Auto-scaling lets you add servers when demand increases and remove them when it decreases, saving costs while maintaining performance.
</details>

---

### Question 4
What does "stateless application" mean in the context of scalability?

A) The application has no databases  
B) User session data is stored on a specific server  
C) Any server can handle any request without relying on local data  
D) The application doesn't track user activity  

<details>
<summary>Answer</summary>

**C) Any server can handle any request without relying on local data**

Stateless applications don't store user session data locally on servers. This allows any server to handle any request, making horizontal scaling much easier.
</details>

---

### Question 5
Which is NOT a sign that your system needs to scale?

A) Response times increasing  
B) CPU usage consistently at 85%  
C) One user reporting a bug  
D) Growing number of timeout errors  

<details>
<summary>Answer</summary>

**C) One user reporting a bug**

A single bug report isn't a scaling issue—it's a code issue. Scaling is needed when performance degrades due to increased load.
</details>

---

## 🧩 Scenario-Based Challenges

### Challenge 1: Design Decision
You're building a photo-sharing app for your school (500 students). Currently, you have one server. What scaling strategy should you plan for the next 2 years?

**Consider:**
- Expected growth to 5,000 students across multiple schools
- Photo uploads and storage
- Budget constraints

**Your Answer:**
```
Write your scaling strategy here, including:
1. Current setup
2. When to scale
3. Which type of scaling (vertical/horizontal)
4. What metrics to monitor
```

<details>
<summary>Sample Answer</summary>

**Current Setup:**
- Single server adequate for 500 students
- Monitor response times and storage usage

**Short-term (0-6 months):**
- Vertical scaling if needed (upgrade to 16GB RAM)
- Implement basic monitoring (CPU, memory, disk)
- Use a CDN for serving photos (faster, reduces server load)

**Medium-term (6-12 months, ~2,000 students):**
- Add horizontal scaling with 2-3 servers
- Implement load balancer
- Use separate database server
- Move photo storage to cloud storage (S3, etc.)

**Long-term (1-2 years, ~5,000 students):**
- Auto-scaling group (3-10 servers based on demand)
- Database read replicas
- Cache layer (Redis) for frequently accessed data
- Monitoring alerts for proactive scaling

**Key Metrics:**
- Response time (keep under 200ms)
- Server CPU/memory (scale at 70% usage)
- Database query performance
- Storage growth rate
</details>

---

### Challenge 2: Trade-offs Analysis
Compare vertical and horizontal scaling for a chat application:

| Aspect | Vertical Scaling | Horizontal Scaling |
|--------|------------------|-------------------|
| Implementation Complexity | | |
| Cost at Small Scale | | |
| Cost at Large Scale | | |
| Fault Tolerance | | |
| Maximum Capacity | | |

Fill in the table and explain which you would choose and why.

<details>
<summary>Sample Answer</summary>

| Aspect | Vertical Scaling | Horizontal Scaling |
|--------|------------------|-------------------|
| Implementation Complexity | Low - just upgrade hardware | High - need load balancing, data distribution |
| Cost at Small Scale | Lower - one server is cheaper | Higher - multiple servers + infrastructure |
| Cost at Large Scale | Very High - enterprise hardware expensive | Moderate - use many cheap servers |
| Fault Tolerance | Poor - single point of failure | Good - if one server fails, others continue |
| Maximum Capacity | Limited by hardware | Nearly unlimited |

**Recommendation for Chat App:**
Start with vertical scaling (simpler for MVP), but design the application to be stateless from day one. Once you hit ~5,000 concurrent users or your server maxes out, switch to horizontal scaling. For a chat app, horizontal scaling is essential long-term because:
1. Real-time messaging requires high availability
2. Users expect 24/7 uptime
3. Chat apps can grow viral quickly
</details>

---

### Challenge 3: Real-World Application
Explain how you would scale these applications:

1. **Video Streaming Platform** (like YouTube)
   - What needs to scale?
   - Biggest challenges?
   - Recommended approach?

2. **E-commerce Site** (like Amazon)
   - What parts scale differently?
   - How to handle flash sales?
   - Database strategy?

<details>
<summary>Sample Answer</summary>

**1. Video Streaming Platform:**

*What needs to scale:*
- Video storage (petabytes of data)
- Video streaming servers
- Video transcoding (converting to different qualities)
- User authentication and recommendations

*Biggest challenges:*
- Bandwidth costs (streaming video uses lots of data)
- Different video qualities for different internet speeds
- Global users need low latency

*Recommended approach:*
- Use CDN for distributing video files worldwide
- Horizontal scaling for web servers
- Separate transcoding service (process videos asynchronously)
- Cache popular videos at edge locations
- Database sharding by user ID

**2. E-commerce Site:**

*What parts scale differently:*
- Product catalog (read-heavy, can cache heavily)
- Shopping cart (needs to be fast and consistent)
- Payment processing (needs to be reliable, can be slower)
- Search functionality (needs special optimization)

*How to handle flash sales:*
- Pre-scale servers before the sale
- Use queue system for checkout (prevent overselling)
- Cache product pages aggressively
- Rate limiting per user (prevent bots)
- Separate database for inventory (handle writes carefully)

*Database strategy:*
- Read replicas for product browsing
- Master database for orders and inventory
- Cache layer (Redis) for product details
- Separate analytics database (doesn't affect user experience)
</details>

---

## 🚀 Practical Exercise

### Exercise: Design a Scalable System

Design a scalable architecture for a school assignment submission system:

**Requirements:**
- 10,000 students can submit assignments
- Teachers can view and grade submissions
- File uploads (PDFs, images)
- Deadline reminders via email
- Grade analytics dashboard

**Your Task:**
Draw or describe:
1. The architecture (servers, databases, services)
2. Which components need to scale
3. How you would handle 10,000 students submitting at the last minute before deadline
4. Cost-saving strategies

**Hints:**
- Think about read vs. write operations
- Consider asynchronous processing
- Where would caching help?
- What can be done in the background?

---

## 📝 Reflection Questions

1. **Why might a company choose NOT to scale horizontally even if it's technically better?**
   (Think about complexity, team size, costs)

2. **Can you think of an application where vertical scaling is actually the best long-term choice?**
   (Hint: Consider applications with specific hardware requirements)

3. **How would you explain scalability to a friend who doesn't know programming?**
   (Practice explaining in simple terms!)

---

## Next Steps

Once you've completed these challenges, move on to **[Load Balancing](../02-load-balancing/)** to learn how to distribute traffic across your scaled servers!
