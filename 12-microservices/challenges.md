# Microservices - Challenges & Questions

Test your understanding of microservices architecture!

## 🎯 Multiple Choice Questions

### Question 1
What is the main advantage of microservices over monolithic architecture?

A) Always faster performance  
B) Easier to develop initially  
C) Independent scaling and deployment  
D) Requires less infrastructure  

<details>
<summary>Answer</summary>

**C) Independent scaling and deployment**

Microservices allow each service to be scaled and deployed independently, which is the key advantage. However, they're not always faster or easier - they add complexity but provide flexibility.
</details>

---

### Question 2
In microservices, why should each service have its own database?

A) To save money on licenses  
B) To ensure independence and prevent tight coupling  
C) Databases are cheap  
D) It's a strict requirement  

<details>
<summary>Answer</summary>

**B) To ensure independence and prevent tight coupling**

Each service having its own database ensures services are truly independent. They can choose the best database for their needs and changes don't affect other services.
</details>

---

### Question 3
What is an API Gateway?

A) A type of database  
B) A single entry point for all client requests that routes to appropriate services  
C) A service that stores user sessions  
D) A backup service  

<details>
<summary>Answer</summary>

**B) A single entry point for all client requests that routes to appropriate services**

An API Gateway acts as a facade, providing a single entry point for clients. It handles routing, authentication, rate limiting, and can aggregate responses from multiple services.
</details>

---

### Question 4
What is the Circuit Breaker pattern used for?

A) Shutting down services during maintenance  
B) Preventing cascading failures by stopping calls to failing services  
C) Routing traffic between services  
D) Load balancing  

<details>
<summary>Answer</summary>

**B) Preventing cascading failures by stopping calls to failing services**

The Circuit Breaker pattern detects when a service is failing and stops sending requests to it temporarily, preventing cascading failures across the system.
</details>

---

### Question 5
When should you NOT use microservices?

A) When you have a large, complex application  
B) When you have multiple teams  
C) When you're a small startup with a simple application  
D) When you need to scale different parts independently  

<details>
<summary>Answer</summary>

**C) When you're a small startup with a simple application**

Microservices add complexity. For small teams or simple applications, a monolith is often better. Start with a monolith and migrate to microservices when you have clear pain points.
</details>

---

## 🧩 Scenario-Based Challenges

### Challenge 1: Breaking Up a Monolith

You have an e-commerce monolith. Divide it into microservices and identify dependencies.

<details>
<summary>Sample Solution</summary>

**Microservices:**
1. User Service - Authentication, profiles
2. Product Service - Catalog, search
3. Cart Service - Shopping cart
4. Order Service - Order management
5. Payment Service - Payment processing
6. Inventory Service - Stock management
7. Notification Service - Emails, SMS

Each service has its own database and communicates via APIs or events.
</details>

---

### Challenge 2: Handle Service Failure

Your Order Service needs to call User, Inventory, and Recommendation services. Recommendation is non-critical but currently down. How do you handle this gracefully?

<details>
<summary>Sample Solution</summary>

Implement Circuit Breaker for all services. For non-critical Recommendation service, use fallback to return cached popular items if the service is unavailable. Critical services (User, Inventory) should fail the request if unavailable after retries.
</details>

---

## 🤔 Critical Thinking Questions

1. **Why is it bad for microservices to share the same database?**
2. **How would you handle authentication across multiple microservices?**
3. **When would you choose synchronous vs asynchronous communication?**

---

## Next Steps

Excellent work! Now explore **[Message Queues](../07-message-queues/)** to learn more about asynchronous communication!
