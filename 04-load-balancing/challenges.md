# Load Balancing - Challenges & Questions

Test your understanding of load balancing concepts!

## 🎯 Multiple Choice Questions

### Question 1
What is the main purpose of a load balancer?

A) To make servers run faster  
B) To distribute traffic across multiple servers  
C) To store user data  
D) To encrypt network traffic  

<details>
<summary>Answer</summary>

**B) To distribute traffic across multiple servers**

A load balancer's primary job is to distribute incoming requests evenly across multiple servers to prevent any single server from being overwhelmed.
</details>

---

### Question 2
Which load balancing algorithm would be BEST for a chat application with long-running connections?

A) Round Robin  
B) Random  
C) Least Connections  
D) IP Hash  

<details>
<summary>Answer</summary>

**C) Least Connections**

Chat applications have connections that last a long time. Least Connections sends new connections to the server with the fewest active connections, preventing any server from having too many long-running connections.
</details>

---

### Question 3
You have 3 servers: Server A (32GB RAM), Server B (16GB RAM), Server C (16GB RAM). Which algorithm should you use?

A) Round Robin  
B) Weighted Round Robin  
C) Random  
D) IP Hash  

<details>
<summary>Answer</summary>

**B) Weighted Round Robin**

Server A is twice as powerful, so it should handle more requests. Weighted Round Robin lets you assign higher weight to Server A (e.g., weight of 2) while Servers B and C get weight of 1.
</details>

---

### Question 4
What happens when a server fails its health check?

A) The entire system shuts down  
B) The load balancer stops sending traffic to that server  
C) The server is deleted permanently  
D) All users are disconnected  

<details>
<summary>Answer</summary>

**B) The load balancer stops sending traffic to that server**

Health checks detect failing servers, and the load balancer automatically stops routing traffic to them until they become healthy again. Other servers continue handling requests.
</details>

---

### Question 5
What is the difference between Layer 4 and Layer 7 load balancing?

A) Layer 4 is faster, Layer 7 is slower  
B) Layer 4 routes by IP/port, Layer 7 routes by content (URL, headers)  
C) Layer 7 is hardware, Layer 4 is software  
D) There is no difference  

<details>
<summary>Answer</summary>

**B) Layer 4 routes by IP/port, Layer 7 routes by content (URL, headers)**

Layer 4 works at the transport layer (TCP/UDP) and makes routing decisions based on IP addresses and ports. Layer 7 works at the application layer and can inspect HTTP content to make smarter routing decisions.
</details>

---

### Question 6
Why should you avoid using sticky sessions when possible?

A) They're too expensive  
B) They reduce the benefits of load balancing  
C) They make the application slower  
D) They don't work with HTTPS  

<details>
<summary>Answer</summary>

**B) They reduce the benefits of load balancing**

Sticky sessions tie users to specific servers, which means if that server gets overloaded or fails, those users have problems. Better to use shared session storage (like Redis) so any server can handle any request.
</details>

---

## 🧩 Scenario-Based Challenges

### Challenge 1: Algorithm Selection

For each scenario, choose the BEST load balancing algorithm and explain why:

**Scenario A**: An e-commerce site with 5 identical servers  
**Your Answer**: _______________  
**Why**: _______________

**Scenario B**: A video streaming service with servers of different capacities (some powerful, some basic)  
**Your Answer**: _______________  
**Why**: _______________

**Scenario C**: A real-time multiplayer game where connections last 30+ minutes  
**Your Answer**: _______________  
**Why**: _______________

<details>
<summary>Sample Answers</summary>

**Scenario A: Round Robin**
- All servers are identical, so simple round-robin distribution works perfectly
- Easy to implement and fair distribution
- Good for typical web requests that complete quickly

**Scenario B: Weighted Round Robin**
- Need to account for different server capacities
- Assign higher weights to powerful servers
- Example: Powerful server (weight: 3), Basic servers (weight: 1)

**Scenario C: Least Connections**
- Game connections are long-running
- Need to prevent any server from having too many active games
- Least Connections ensures even distribution of concurrent connections
</details>

---

### Challenge 2: Troubleshooting

Your load balancer setup has issues. Diagnose the problem:

**Problem 1:**
- Setup: 3 web servers behind a load balancer
- Issue: Users keep getting logged out randomly
- Load Balancer: Using Round Robin
- Sessions: Stored locally on each server

**What's wrong?**: _______________  
**Solution**: _______________

<details>
<summary>Answer</summary>

**Problem**: Users get logged out because their requests are being sent to different servers, and each server doesn't know about sessions from other servers.

**Solutions:**
1. **Best**: Use shared session storage (Redis, database)
2. **Okay**: Enable sticky sessions (but not ideal)
3. **Better Long-term**: Use token-based authentication (JWT)
</details>

---

**Problem 2:**
- Setup: Load balancer with 4 servers
- Issue: Server 1 handles 70% of traffic, others handle 10% each
- Algorithm: IP Hash

**What's wrong?**: _______________  
**Solution**: _______________

<details>
<summary>Answer</summary>

**Problem**: IP Hash distributes based on user IP addresses. If most users come from a similar network (e.g., same ISP, same office), they may hash to the same server, causing uneven distribution.

**Solutions:**
1. Switch to Round Robin or Least Connections
2. If you need session affinity, use cookie-based sticky sessions
3. Add more variety to the hash algorithm (include port, user-agent)
</details>

---

### Challenge 3: Design Exercise

Design a load balancing architecture for a social media application:

**Requirements:**
- 1 million daily active users
- Features: Posts, comments, messaging, photo uploads
- Must handle 10,000 requests per second
- 99.9% uptime required

**Your Design Should Include:**
1. How many load balancers?
2. How many tiers of load balancing?
3. Which algorithms for different services?
4. How to handle server failures?
5. Geographic considerations?

<details>
<summary>Sample Solution</summary>

**Architecture:**

```
[Global DNS Load Balancer]
         |
    ┌────┴────┐
    ↓         ↓
[US Region]  [EU Region]
    |          |
[Primary LB]  [Primary LB]
[Backup LB]   [Backup LB]
    |          |
    ├─── [Web Tier] (Round Robin) ───┐
    |                                  |
    ├─── [API Tier] (Least Connections) ─┐
    |                                      |
    └─── [Media Processing] (Weighted RR) ┘
```

**Details:**

1. **Geographic Load Balancing**
   - DNS-based routing to nearest region
   - Reduces latency for users
   - US and EU data centers

2. **Regional Load Balancers**
   - Primary + Backup for redundancy
   - Auto-failover if primary fails
   - Layer 7 load balancing

3. **Service-Specific Distribution**
   - **Web Tier**: Round Robin (simple requests)
   - **API Tier**: Least Connections (handles various request lengths)
   - **Media Processing**: Weighted Round Robin (different server sizes)

4. **Health Checks**
   - HTTP health check every 15 seconds
   - 3 failures = remove from pool
   - 2 successes = add back to pool

5. **Scaling Plan**
   - 10-20 web servers per region
   - 15-30 API servers per region
   - 5-10 media processing servers (powerful machines)
   - Auto-scaling based on CPU/request rate

**Failover Strategy:**
- Backup LB activates within 30 seconds
- Failed servers automatically removed
- Alert team for manual inspection
- Keep 20% extra capacity for spikes
</details>

---

### Challenge 4: SSL/TLS Termination

Your company wants to implement HTTPS for their web application.

**Current Setup:**
- 5 web servers behind a load balancer
- Each server has its own SSL certificate
- Performance is slow due to SSL processing

**Questions:**
1. What is SSL/TLS termination?
2. Should you terminate SSL at the load balancer or servers?
3. What are the pros and cons?
4. Draw the architecture

<details>
<summary>Sample Answer</summary>

**1. SSL/TLS Termination:**
SSL/TLS termination is when encryption/decryption happens at the load balancer instead of the application servers. The load balancer handles HTTPS from users and sends plain HTTP to backend servers.

**2. Recommendation:**
Terminate SSL at the load balancer for most use cases.

**3. Pros and Cons:**

**Pros of LB Termination:**
- Reduces CPU load on application servers (SSL is computationally expensive)
- Centralized certificate management (one place to update certs)
- Easier to enforce security policies
- Servers can focus on application logic
- Cost savings (fewer SSL certs needed)

**Cons of LB Termination:**
- Traffic between LB and servers is unencrypted (mitigate with private network)
- Load balancer becomes more critical (needs to be powerful)
- May not meet compliance for ultra-sensitive data

**4. Architecture:**

```
Internet (HTTPS)
       ↓
[Load Balancer] ← SSL Certificate installed here
  (Terminates SSL)
       ↓
Internal Network (HTTP)
       ↓
   ┌───┼───┐
   ↓   ↓   ↓
[Web Server 1] [Web Server 2] [Web Server 3]
(No SSL needed)
```

**Best Practice:**
- Use SSL termination at LB
- Ensure LB to server traffic is on private network
- For ultra-sensitive data, use end-to-end encryption (SSL to servers too)
</details>

---

## 🔧 Hands-On Exercise

### Exercise: Configure a Simple Load Balancer

If you have Docker installed, try this exercise:

**Step 1**: Create 3 simple web servers

Create `server.py`:
```python
from flask import Flask
import os

app = Flask(__name__)
server_id = os.environ.get('SERVER_ID', '1')

@app.route('/')
def hello():
    return f"Hello from Server {server_id}!"

@app.route('/health')
def health():
    return "OK", 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Step 2**: Research how to:
1. Run 3 instances on different ports (5001, 5002, 5003)
2. Set up Nginx as a load balancer
3. Configure Round Robin algorithm
4. Test health checks

**Challenge Questions:**
1. What happens when you stop one server?
2. Can you configure Weighted Round Robin?
3. How would you add a 4th server without downtime?

---

## 📊 Comparison Exercise

Create a comparison table for different load balancing approaches:

| Aspect | Hardware LB | Software LB | Cloud LB |
|--------|-------------|-------------|----------|
| Cost | | | |
| Flexibility | | | |
| Scalability | | | |
| Ease of Setup | | | |
| Performance | | | |
| Best Use Case | | | |

<details>
<summary>Sample Answer</summary>

| Aspect | Hardware LB | Software LB | Cloud LB |
|--------|-------------|-------------|----------|
| Cost | Very High ($10k-$100k+) | Low (free or cheap) | Pay per use (moderate) |
| Flexibility | Low (fixed features) | High (configurable) | Medium (provider limits) |
| Scalability | Limited by device | Limited by server | Auto-scales easily |
| Ease of Setup | Complex | Moderate | Very Easy |
| Performance | Excellent | Good | Excellent |
| Best Use Case | Large enterprises with budget | Startups, custom needs | Most modern applications |
</details>

---

## 🤔 Critical Thinking Questions

1. **Why can't you just use DNS to distribute traffic instead of a load balancer?**
   (Hint: Think about failure detection and real-time updates)

2. **A load balancer is itself a single point of failure. How do large companies solve this?**
   (Think about redundancy)

3. **When might you NOT want to use a load balancer?**
   (Consider small applications, specific use cases)

4. **How would you handle load balancing for WebSocket connections that need to stay open?**
   (Long-lived connections are tricky!)

---

## Next Steps

Great job! You now understand how to distribute traffic effectively. Next, learn about **[Caching](../03-caching/)** to make your applications lightning fast!
