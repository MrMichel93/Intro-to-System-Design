# Microservices

## What are Microservices?

**Microservices** is an architectural style where an application is built as a collection of small, independent services that work together. Each service focuses on doing one thing well and can be developed, deployed, and scaled independently.

**Analogy**: Think of a restaurant:
- **Monolith**: One person does everything (cooking, serving, cleaning, billing)
- **Microservices**: Specialized team (chef, waiter, cleaner, cashier) - each focused on their task

## Monolith vs Microservices

### Monolithic Architecture
All code in one large application.

```
┌─────────────────────────────┐
│     Single Application      │
│                             │
│  ┌────────────────────────┐ │
│  │   User Management      │ │
│  ├────────────────────────┤ │
│  │   Product Catalog      │ │
│  ├────────────────────────┤ │
│  │   Shopping Cart        │ │
│  ├────────────────────────┤ │
│  │   Payment Processing   │ │
│  ├────────────────────────┤ │
│  │   Order Management     │ │
│  └────────────────────────┘ │
│                             │
│      Single Database        │
└─────────────────────────────┘
```

**Pros:**
- Simple to develop initially
- Easy to test (everything together)
- Simple deployment (one unit)

**Cons:**
- Hard to scale (must scale everything)
- Tight coupling (changes affect everything)
- Slow deployment (must deploy entire app)
- Technology lock-in (one language/framework)
- If one part crashes, everything crashes

---

### Microservices Architecture
Application split into independent services.

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│  User    │  │ Product  │  │   Cart   │
│ Service  │  │ Service  │  │ Service  │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │             │
┌────▼────┐   ┌────▼────┐   ┌────▼────┐
│ User DB │   │Prod DB  │   │Cart DB  │
└─────────┘   └─────────┘   └─────────┘

┌──────────┐  ┌──────────┐
│ Payment  │  │  Order   │
│ Service  │  │ Service  │
└────┬─────┘  └────┬─────┘
     │             │
┌────▼────┐   ┌────▼────┐
│ Pay DB  │   │Order DB │
└─────────┘   └─────────┘
```

**Pros:**
- Independent scaling (scale only what needs it)
- Independent deployment (update one service)
- Technology flexibility (each service can use different tech)
- Fault isolation (one service fails, others continue)
- Easier to understand (smaller codebases)
- Better for large teams

**Cons:**
- More complex to design and manage
- Network latency (services communicate over network)
- Distributed system challenges
- More difficult testing (integration tests)
- Need for orchestration (Docker, Kubernetes)

---

## When to Use Microservices

✅ **Good reasons:**
- Large, complex application
- Multiple teams working on different features
- Need to scale different parts independently
- Want flexibility in technology choices
- Rapid development and deployment needed

❌ **Not good reasons:**
- Small application with few users
- Limited team size (< 5 people)
- Startup in early stage (keep it simple)
- When you don't have DevOps expertise

**Rule of thumb**: Start with a monolith, migrate to microservices when you have clear pain points.

---

## Key Principles of Microservices

### 1. Single Responsibility
Each service does ONE thing well.

**Example:**
- ✅ User Service: Handle authentication, user profiles
- ✅ Payment Service: Process payments, refunds
- ❌ Everything Service: Does users, payments, products, orders (too broad!)

---

### 2. Independence
Services can be developed, deployed, and scaled independently.

```
Deploy Payment Service → Only Payment Service restarts
Deploy User Service → Only User Service restarts
```

No need to redeploy everything!

---

### 3. Decentralized Data
Each service has its own database.

**Why?**
- Services don't depend on each other's data structure
- Can choose best database for each service
- Changes don't ripple across services

**Challenge**: How to get data from multiple services?
- **Solution 1**: API calls between services
- **Solution 2**: Event-driven architecture (more on this later)

---

### 4. Communication via APIs
Services communicate using well-defined APIs (typically REST or gRPC).

**Example:**
```
Order Service needs user info
→ Calls User Service API: GET /api/users/123
→ Gets user data
→ Processes order
```

---

## Communication Patterns

### 1. Synchronous (Request-Response)

**REST API Example:**
```
Order Service → HTTP GET /api/users/123 → User Service
Order Service ← JSON {user data} ← User Service
```

**Pros:**
- Simple and direct
- Immediate response
- Easy to understand

**Cons:**
- Tight coupling (Order Service must wait)
- If User Service is down, Order Service affected
- Network latency

---

### 2. Asynchronous (Event-Driven)

**Message Queue Example:**
```
Order Service → Publishes "Order Created" event → Message Queue
                        ↓
            Multiple services subscribe:
            - Email Service (send confirmation)
            - Inventory Service (reduce stock)
            - Analytics Service (track metrics)
```

**Pros:**
- Loose coupling (services don't wait for each other)
- Fault tolerant (messages queued if service is down)
- Scalable

**Cons:**
- More complex
- Eventual consistency
- Harder to debug

---

## Service Discovery

How do services find each other in a dynamic environment?

**Problem:**
```
Order Service needs Payment Service
But Payment Service IP changes when it restarts!
How does Order Service find it?
```

**Solution: Service Registry**
```
1. Payment Service starts → Registers itself: "I'm at 192.168.1.5:8080"
2. Order Service needs Payment Service → Asks registry: "Where is Payment Service?"
3. Registry responds: "192.168.1.5:8080"
4. Order Service calls Payment Service
```

**Tools:**
- **Consul**: Service registry and health checking
- **Eureka**: Netflix's service discovery
- **Kubernetes**: Built-in service discovery

---

## API Gateway

**Problem**: Frontend shouldn't call 20 different microservices directly.

**Solution**: API Gateway - single entry point for all clients.

```
Mobile App ─┐
Web App ────┼─→ [API Gateway] ─┬→ User Service
Browser ────┘                   ├→ Product Service
                                ├→ Order Service
                                └→ Payment Service
```

**API Gateway Responsibilities:**
1. **Routing**: Send requests to correct service
2. **Authentication**: Verify tokens
3. **Rate Limiting**: Prevent abuse
4. **Load Balancing**: Distribute traffic
5. **Caching**: Cache frequent responses
6. **Aggregation**: Combine multiple service calls

**Example:**
```
Client request: GET /api/dashboard

API Gateway:
1. Calls User Service: GET /users/123
2. Calls Order Service: GET /orders?user=123
3. Calls Payment Service: GET /payments?user=123
4. Combines responses and returns to client

Client receives complete dashboard data in one request!
```

**Tools**: Kong, AWS API Gateway, Azure API Management, NGINX

---

## Data Management in Microservices

### Challenge: Each service has its own database
How do you query data across services?

### Pattern 1: API Composition
```python
# Order Service needs to show order with user details
def get_order_with_user(order_id):
    # Get order from own database
    order = db.query("SELECT * FROM orders WHERE id = ?", order_id)
    
    # Call User Service for user details
    user = http_client.get(f"http://user-service/api/users/{order.user_id}")
    
    # Combine and return
    return {
        "order": order,
        "user": user
    }
```

**Pros**: Simple  
**Cons**: Multiple network calls, slower

---

### Pattern 2: Database per Service with Events
```python
# When user updates profile in User Service
def update_user(user_id, new_email):
    # Update User Service database
    db.update_user(user_id, new_email)
    
    # Publish event
    event_bus.publish("UserUpdated", {
        "user_id": user_id,
        "email": new_email
    })

# Order Service subscribes to UserUpdated events
def handle_user_updated(event):
    # Update local copy of user data
    db.update_user_cache(event.user_id, event.email)
```

**Pros**: Fast reads (data local)  
**Cons**: Eventual consistency, more complex

---

### Pattern 3: CQRS (Command Query Responsibility Segregation)
Separate read and write models.

```
Write Model (Commands):
- Create Order → Order Service writes to Order DB

Read Model (Queries):
- Get Order with User → Read from combined view/materialized view
```

---

## Handling Failures

Microservices must handle failures gracefully!

### 1. Circuit Breaker Pattern
Prevent cascading failures.

```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5):
        self.failure_count = 0
        self.state = "CLOSED"  # CLOSED, OPEN, HALF_OPEN
        self.failure_threshold = failure_threshold
    
    def call_service(self, service_call):
        if self.state == "OPEN":
            # Circuit is open - don't even try
            raise ServiceUnavailableError("Circuit breaker is OPEN")
        
        try:
            result = service_call()
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise
    
    def on_failure(self):
        self.failure_count += 1
        if self.failure_count >= self.failure_threshold:
            self.state = "OPEN"
            # Set timer to try again later (HALF_OPEN)
    
    def on_success(self):
        self.failure_count = 0
        self.state = "CLOSED"
```

**States:**
- **CLOSED**: Normal operation, requests go through
- **OPEN**: Too many failures, reject requests immediately
- **HALF_OPEN**: Try one request, if succeeds → CLOSED, if fails → OPEN

---

### 2. Retry with Exponential Backoff
```python
def call_service_with_retry(service_call, max_retries=3):
    for attempt in range(max_retries):
        try:
            return service_call()
        except TemporaryError:
            if attempt == max_retries - 1:
                raise
            wait_time = 2 ** attempt  # 1s, 2s, 4s
            time.sleep(wait_time)
```

---

### 3. Fallback
```python
def get_user_recommendations(user_id):
    try:
        # Try personalized recommendations
        return recommendation_service.get_personalized(user_id)
    except ServiceUnavailableError:
        # Fallback to popular items
        return recommendation_service.get_popular()
```

---

## Real-World Examples

### Netflix
- 500+ microservices
- Each service responsible for specific feature
- Can deploy 1000s of times per day
- Services: User service, Recommendation service, Video streaming, Billing, etc.

### Amazon
- Two-pizza team rule (team should be feed-able by two pizzas)
- Each team owns their microservices
- Deployed independently
- Services: Product catalog, Cart, Payment, Shipping, etc.

### Uber
- Microservices for different functions
- Services: Rider service, Driver service, Trip service, Payment, Maps, etc.
- Each can scale independently based on demand

---

## Best Practices

1. **Start Simple**: Don't over-engineer early
2. **Define Clear Boundaries**: Each service should have clear responsibility
3. **Design for Failure**: Assume services will fail
4. **Monitor Everything**: Logs, metrics, traces
5. **Automate Deployment**: Use CI/CD pipelines
6. **API Versioning**: Don't break existing clients
7. **Security**: Secure service-to-service communication
8. **Documentation**: Document APIs and dependencies

---

## Common Pitfalls

1. **Too Many Services**: Don't create microservice for everything
2. **Distributed Monolith**: Services tightly coupled (defeats purpose)
3. **No API Gateway**: Clients calling services directly
4. **Shared Database**: Services sharing same database (not independent)
5. **Ignoring Network**: Network calls are slow and can fail
6. **Poor Monitoring**: Can't debug distributed system without good monitoring

---

## Tools & Technologies

**Container Orchestration:**
- Docker (containers)
- Kubernetes (orchestration)

**Service Mesh:**
- Istio (traffic management, security)
- Linkerd

**API Gateway:**
- Kong, NGINX, AWS API Gateway

**Service Discovery:**
- Consul, Eureka

**Monitoring:**
- Prometheus, Grafana
- ELK Stack (Elasticsearch, Logstash, Kibana)
- Distributed Tracing: Jaeger, Zipkin

---

## Summary

- Microservices split applications into small, independent services
- Each service has one responsibility and own database
- Services communicate via APIs or events
- Benefits: Independent scaling, deployment, technology choice
- Challenges: Complexity, network issues, distributed system problems
- Use API Gateway as single entry point
- Design for failure with circuit breakers and retries
- Start with monolith, migrate to microservices when needed

---

## Next Steps

Complete the challenges, then learn about **[Message Queues](../07-message-queues/)** - a key technology for microservices communication!
