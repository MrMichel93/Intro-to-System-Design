# Design Exercise: E-Commerce Monolith to Microservices Migration

## 1. Design Problem

### Problem Statement
Design a strategy to break down a monolithic e-commerce application into microservices architecture. The system must be migrated gradually without downtime, with clear service boundaries, independent deployment capabilities, and improved scalability. The migration must address data management, inter-service communication, and operational complexity.

### Context and Constraints
- Existing monolith handles 50 million users
- Current system: Single codebase, single database, single deployment
- Business cannot afford downtime during migration
- Team of 50 engineers needs to work independently
- Some features are tightly coupled in current system
- Legacy database schema with complex relationships
- Must maintain backward compatibility with mobile apps and third-party integrations

### Requirements

#### Functional Requirements
- Identify service boundaries based on business capabilities
- Enable independent deployment of services
- Maintain data consistency across services
- Support inter-service communication
- Preserve existing functionality during migration
- Allow gradual rollout and rollback capability
- Support both synchronous and asynchronous communication

#### Non-Functional Requirements
- **Zero Downtime**: Migrate without service interruption
- **Performance**: No degradation from monolith
- **Scalability**: Individual services can scale independently
- **Maintainability**: Clear service ownership and boundaries
- **Observability**: Distributed tracing, logging, monitoring
- **Development Speed**: Faster feature delivery with independent teams

## 2. Guided Questions

### Understanding Microservices
1. **What are the benefits of microservices over monolith?**
   - Hint: Think about scalability, deployment, team organization
   - Consider: Independent scaling, technology flexibility, fault isolation

2. **What challenges come with microservices?**
   - Hint: Network calls, data consistency, complexity
   - Consider: Distributed transactions, monitoring, debugging

### Service Boundaries
3. **How do you identify service boundaries?**
   - Hint: Look at business capabilities, not technical layers
   - Consider: Domain-Driven Design, bounded contexts

4. **Should each service have its own database?**
   - Hint: Database per service pattern
   - Consider: Data duplication, eventual consistency, transactions

### Migration Strategy
5. **Do you migrate everything at once or gradually?**
   - Hint: Big bang vs. strangler fig pattern
   - Consider: Risk, complexity, business continuity

6. **How do you handle shared data between services?**
   - Hint: Orders need product information, users need address data
   - Consider: Data duplication, API calls, event sourcing

## 3. Step-by-Step Guidance

### Step 1: Analyze Current Monolith
Understand what you're breaking apart:

**Current Monolith Structure:**
```
┌─────────────────────────────────┐
│      E-Commerce Monolith        │
│                                 │
│  ┌────────────────────────────┐ │
│  │   User Management          │ │
│  │   - Authentication         │ │
│  │   - Profiles               │ │
│  │   - Address Management     │ │
│  ├────────────────────────────┤ │
│  │   Product Catalog          │ │
│  │   - Products               │ │
│  │   - Categories             │ │
│  │   - Inventory              │ │
│  ├────────────────────────────┤ │
│  │   Shopping Cart            │ │
│  │   - Add/Remove Items       │ │
│  │   - Save for Later         │ │
│  ├────────────────────────────┤ │
│  │   Order Management         │ │
│  │   - Checkout               │ │
│  │   - Order Processing       │ │
│  │   - Order History          │ │
│  ├────────────────────────────┤ │
│  │   Payment Processing       │ │
│  │   - Payment Methods        │ │
│  │   - Transactions           │ │
│  ├────────────────────────────┤ │
│  │   Shipping                 │ │
│  │   - Calculate Rates        │ │
│  │   - Track Shipments        │ │
│  └────────────────────────────┘ │
│                                 │
│      ┌───────────────┐          │
│      │   PostgreSQL  │          │
│      │   (Monolithic │          │
│      │    Database)  │          │
│      └───────────────┘          │
└─────────────────────────────────┘
```

### Step 2: Identify Microservices
Use Domain-Driven Design principles:

**Proposed Microservices:**

1. **User Service**
   - User registration and authentication
   - Profile management
   - Address management
   - User preferences

2. **Product Catalog Service**
   - Product information
   - Categories and search
   - Product images and descriptions
   - Inventory management

3. **Cart Service**
   - Shopping cart operations
   - Save for later
   - Cart persistence

4. **Order Service**
   - Order creation and processing
   - Order status tracking
   - Order history

5. **Payment Service**
   - Payment method management
   - Payment processing
   - Transaction records

6. **Shipping Service**
   - Shipping rate calculation
   - Carrier integration
   - Shipment tracking

7. **Notification Service**
   - Email notifications
   - SMS notifications
   - Push notifications

8. **Review Service**
   - Product reviews
   - Ratings and feedback

### Step 3: Choose Migration Strategy
**Strangler Fig Pattern** (Recommended):

```
Phase 1: Monolith + New Services
┌────────────┐     ┌────────────┐
│  Monolith  │────▶│   User     │
│            │     │  Service   │
│  (Most     │     └────────────┘
│   logic)   │
└────────────┘

Phase 2: Gradual Migration
┌────────────┐     ┌────────────┐
│  Monolith  │     │   User     │
│            │────▶│  Service   │
│  (Some     │     ├────────────┤
│   logic)   │     │  Product   │
│            │────▶│  Service   │
└────────────┘     └────────────┘

Phase 3: Complete Migration
┌────────────┐     ┌────────────┐
│   Legacy   │     │   User     │
│ (Retired)  │     │  Service   │
└────────────┘     ├────────────┤
                   │  Product   │
                   │  Service   │
                   ├────────────┤
                   │   Order    │
                   │  Service   │
                   ├────────────┤
                   │  Payment   │
                   │  Service   │
                   └────────────┘
```

### Step 4: Handle Data Migration
**Database per Service Pattern:**

```
Before (Monolith):
┌─────────────────────────────┐
│      PostgreSQL             │
│                             │
│  users, products, orders,   │
│  payments, shipping, etc.   │
└─────────────────────────────┘

After (Microservices):
┌──────────┐ ┌───────────┐ ┌──────────┐
│ User DB  │ │Product DB │ │ Order DB │
└──────────┘ └───────────┘ └──────────┘
┌──────────┐ ┌───────────┐
│Payment DB│ │Shipping DB│
└──────────┘ └───────────┘
```

**Data Synchronization:**
```python
# Keep monolith and new service in sync during migration
class DualWriteStrategy:
    def create_user(self, user_data):
        # Write to monolith database
        monolith_user_id = monolith_db.insert_user(user_data)
        
        # Also write to new User Service
        try:
            user_service.create_user(user_data)
        except Exception as e:
            logger.error(f"Failed to sync to User Service: {e}")
            # Monolith is source of truth during migration
        
        return monolith_user_id
```

### Step 5: Implement API Gateway
Centralized entry point:

```
┌─────────────┐
│   Clients   │
│ (Web/Mobile)│
└──────┬──────┘
       ↓
┌──────▼──────────┐
│   API Gateway   │
│                 │
│ - Routing       │
│ - Auth          │
│ - Rate Limiting │
│ - Caching       │
└────────┬────────┘
         │
  ┌──────┼─────┬──────┬──────┐
  ↓      ↓     ↓      ↓      ↓
[User] [Product] [Order] [Payment]
Service Service  Service  Service
```

**Routing Example:**
```
GET /api/users/*       → User Service
GET /api/products/*    → Product Service
POST /api/orders       → Order Service
POST /api/payments/*   → Payment Service
```

### Step 6: Service Communication
**Synchronous (REST/gRPC):**
```python
# Order Service needs product info
class OrderService:
    def create_order(self, cart_items):
        # Call Product Service
        products = product_service_client.get_products(
            product_ids=[item['product_id'] for item in cart_items]
        )
        
        # Calculate total
        total = sum(p['price'] * qty for p, qty in zip(products, cart_items))
        
        # Create order
        order = self.db.create_order(total=total, items=cart_items)
        return order
```

**Asynchronous (Event-Driven):**
```python
# Order created → Notify other services
class OrderService:
    def create_order(self, order_data):
        order = self.db.create_order(order_data)
        
        # Publish event
        event_bus.publish('order.created', {
            'order_id': order.id,
            'user_id': order.user_id,
            'total': order.total,
            'items': order.items
        })
        
        return order

# Other services subscribe
class InventoryService:
    @subscribe('order.created')
    def handle_order_created(self, event):
        # Reduce inventory
        self.reduce_stock(event['items'])

class NotificationService:
    @subscribe('order.created')
    def handle_order_created(self, event):
        # Send confirmation email
        self.send_order_confirmation(event['user_id'], event['order_id'])
```

### Step 7: Handle Distributed Transactions
**Saga Pattern:**

```python
# Two-Phase Saga for Order Creation
class OrderSaga:
    def create_order(self, cart_items):
        # Step 1: Reserve inventory
        reservation_id = inventory_service.reserve(cart_items)
        
        try:
            # Step 2: Process payment
            payment_id = payment_service.charge(amount)
            
            try:
                # Step 3: Create order
                order_id = order_service.create(cart_items)
                
                # Step 4: Confirm inventory
                inventory_service.confirm(reservation_id)
                
                return order_id
                
            except Exception:
                # Rollback payment
                payment_service.refund(payment_id)
                raise
        except Exception:
            # Rollback inventory
            inventory_service.release(reservation_id)
            raise
```

## 4. Sample Solution

### Complete Microservices Architecture

```
┌──────────────────────────────────────────────┐
│              Load Balancer                   │
└────────────────┬─────────────────────────────┘
                 ↓
┌────────────────▼─────────────────────────────┐
│             API Gateway                      │
│  - Authentication & Authorization            │
│  - Request Routing                           │
│  - Rate Limiting                             │
│  - Response Aggregation                      │
└───┬─────┬──────┬──────┬──────┬──────┬───────┘
    │     │      │      │      │      │
    ↓     ↓      ↓      ↓      ↓      ↓
┌───────────────────────────────────────────────┐
│           Microservices Layer                 │
│                                               │
│  ┌──────┐  ┌────────┐  ┌───────┐  ┌──────┐  │
│  │ User │  │Product │  │ Cart  │  │Order │  │
│  │Service  │Service │  │Service│  │Service  │
│  └───┬──┘  └───┬────┘  └───┬───┘  └───┬──┘  │
│      │         │           │          │     │
│  ┌───────┐  ┌────────┐  ┌──────────┐        │
│  │Payment│  │Shipping│  │Notification       │
│  │Service│  │Service │  │  Service │        │
│  └───┬───┘  └───┬────┘  └─────┬────┘        │
└──────┼──────────┼─────────────┼─────────────┘
       │          │             │
       ↓          ↓             ↓
┌────────────────────────────────────────┐
│        Event Bus (Kafka/RabbitMQ)      │
└────────────────────────────────────────┘
       │          │             │
       ↓          ↓             ↓
┌────────────────────────────────────────┐
│           Database Layer               │
│                                        │
│  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  │
│  │User │  │Prod │  │Order│  │Pay  │  │
│  │ DB  │  │ DB  │  │ DB  │  │ DB  │  │
│  └─────┘  └─────┘  └─────┘  └─────┘  │
└────────────────────────────────────────┘
```

### Migration Timeline

**Month 1-2: Preparation**
- Set up infrastructure (Kubernetes, service mesh)
- Implement API Gateway
- Set up monitoring and logging
- Design service boundaries
- Create migration plan

**Month 3-4: Extract First Service (User Service)**
- Create User Service with own database
- Implement dual-write (monolith + new service)
- Route read traffic to new service (canary 10%)
- Monitor and validate
- Gradually increase to 100%

**Month 5-6: Extract Product Catalog Service**
- Repeat process for Product Service
- Implement caching layer
- Test search and filtering
- Route traffic gradually

**Month 7-8: Extract Order Service**
- Most complex due to dependencies
- Implement Saga pattern for distributed transactions
- Careful testing of checkout flow
- Gradual rollout

**Month 9-10: Extract Remaining Services**
- Payment, Shipping, Notification services
- Less complex, follow established pattern

**Month 11-12: Optimize and Decommission Monolith**
- Remove monolith code
- Optimize inter-service communication
- Performance tuning
- Cost optimization

## 5. Extension Challenges

### Challenge 1: Service Discovery
**Requirement**: Services need to find each other dynamically

**How would you implement this?**
<details>
<summary>Solution</summary>

Use **Consul** or **Kubernetes Service Discovery**:

```python
# Service registration
class ProductService:
    def start(self):
        # Register with Consul
        consul.agent.service.register(
            name='product-service',
            service_id='product-service-1',
            address='10.0.1.5',
            port=8080,
            check={
                'http': 'http://10.0.1.5:8080/health',
                'interval': '10s'
            }
        )

# Service discovery
class OrderService:
    def get_product_info(self, product_id):
        # Discover Product Service
        services = consul.health.service('product-service', passing=True)
        product_service_url = f"http://{services[0]['Service']['Address']}:{services[0]['Service']['Port']}"
        
        # Make request
        response = requests.get(f"{product_service_url}/products/{product_id}")
        return response.json()
```

Alternative: **Kubernetes DNS**
```yaml
# Service automatically discoverable at:
# product-service.default.svc.cluster.local
apiVersion: v1
kind: Service
metadata:
  name: product-service
spec:
  selector:
    app: product-service
  ports:
    - port: 80
      targetPort: 8080
```

</details>

### Challenge 2: Distributed Tracing
**Requirement**: Track requests across multiple services

**How would you implement this?**
<details>
<summary>Solution</summary>

Use **OpenTelemetry** or **Jaeger**:

```python
from opentelemetry import trace
from opentelemetry.instrumentation.requests import RequestsInstrumentor

tracer = trace.get_tracer(__name__)

class OrderService:
    def create_order(self, cart_items):
        with tracer.start_as_current_span("create_order") as span:
            span.set_attribute("cart.items.count", len(cart_items))
            
            # Call Product Service (auto-instrumented)
            products = self.product_client.get_products(cart_items)
            
            # Call Payment Service
            with tracer.start_as_current_span("process_payment"):
                payment = self.payment_client.charge(total)
            
            # Create order
            order = self.db.create_order(cart_items, payment)
            
            span.set_attribute("order.id", order.id)
            return order

# Trace ID propagated across all services
# View entire request flow in Jaeger UI
```

Benefits:
- See full request path across services
- Identify bottlenecks
- Debug failures
- Monitor latency

</details>

### Challenge 3: Circuit Breaker Pattern
**Requirement**: Prevent cascade failures when a service is down

**How would you implement this?**
<details>
<summary>Solution</summary>

Use **Circuit Breaker** pattern:

```python
from circuitbreaker import circuit

class OrderService:
    @circuit(failure_threshold=5, recovery_timeout=60)
    def get_product_info(self, product_id):
        # Call Product Service
        response = requests.get(f"{PRODUCT_SERVICE_URL}/products/{product_id}")
        response.raise_for_status()
        return response.json()
    
    def create_order(self, cart_items):
        try:
            # Try to get product info
            products = [self.get_product_info(item['product_id']) 
                       for item in cart_items]
        except CircuitBreakerOpen:
            # Circuit open - use cached data
            products = [self.get_cached_product(item['product_id']) 
                       for item in cart_items]
        
        # Continue with order creation
        order = self.db.create_order(cart_items, products)
        return order
```

States:
- **Closed**: Normal operation
- **Open**: Too many failures, reject requests immediately
- **Half-Open**: Test if service recovered

</details>

---

## Summary

This design exercise demonstrates how to migrate from monolith to microservices by:
- Identifying service boundaries based on business capabilities
- Using Strangler Fig pattern for gradual migration
- Implementing database per service pattern
- Handling inter-service communication (sync and async)
- Managing distributed transactions with Saga pattern
- Using API Gateway for routing and cross-cutting concerns
- Ensuring observability with distributed tracing

**Key Takeaways**:
1. **Migrate gradually, not big bang**
2. **Service boundaries follow business domains**
3. **Each service owns its data**
4. **Use events for loose coupling**
5. **Implement patterns: Saga, Circuit Breaker, Service Discovery**
6. **Observability is critical in distributed systems**
