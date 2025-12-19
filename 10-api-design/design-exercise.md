# Design Exercise: Ride-Sharing App REST API

## 1. Design Problem

### Problem Statement
Design a complete REST API for a ride-sharing platform like Uber or Lyft. The API must support riders requesting rides, drivers accepting and completing rides, real-time location tracking, payments, ratings, and ride history. The design must be RESTful, secure, performant, and easy for mobile app developers to integrate.

### Context and Constraints
- Mobile apps (iOS/Android) are primary clients
- Real-time location updates are critical
- High concurrency during peak hours (morning/evening commutes)
- Geographically distributed users
- Must work offline gracefully (queue operations)
- Security is critical (user data, payments, locations)
- Need backward compatibility as API evolves
- Third-party integrations (maps, payments, SMS)

### Requirements

#### Functional Requirements
- User authentication and authorization (riders and drivers)
- Request and match rides
- Real-time location tracking
- Fare estimation and calculation
- Payment processing
- Driver and rider ratings
- Ride history
- Driver earnings and analytics
- Surge pricing
- Ride sharing (multiple passengers)
- Schedule rides in advance
- Promo codes and discounts

#### Non-Functional Requirements
- **Performance**: API response < 200ms (p95), location updates < 500ms
- **Scale**: 10 million users, 1 million daily rides, 100K concurrent connections
- **Availability**: 99.99% uptime
- **Security**: OAuth 2.0, HTTPS only, encrypted sensitive data
- **Rate Limiting**: Prevent abuse (100 requests/minute per user)
- **Consistency**: Strong consistency for payments, eventual consistency for location
- **Documentation**: Complete API documentation with examples

## 2. Guided Questions

### API Design Basics
1. **What are the main resources (nouns) in the system?**
   - Hint: Think about entities users interact with
   - Consider: Users, rides, drivers, payments, locations

2. **What HTTP methods should each endpoint use?**
   - Hint: GET for reading, POST for creating, PUT for updating, DELETE for removing
   - Consider: REST conventions, idempotency

3. **How should we version the API?**
   - Hint: APIs evolve, clients update slowly
   - Consider: URL versioning (/v1/), header versioning

### Real-Time Requirements
4. **How do we handle real-time location updates?**
   - Hint: REST is request-response, but we need streaming
   - Consider: WebSockets, Server-Sent Events, polling

5. **What happens when a client loses connection during a ride?**
   - Hint: Need to recover state when reconnecting
   - Consider: Idempotent operations, state reconciliation

### Security and Authorization
6. **How do we secure the API?**
   - Hint: Not all endpoints should be publicly accessible
   - Consider: OAuth 2.0, JWT tokens, API keys

7. **Should riders see driver personal information?**
   - Hint: Privacy vs. safety
   - Consider: Data minimization, what's necessary for service

### Error Handling
8. **What happens when payment fails mid-ride?**
   - Hint: Ride is complete, but payment failed
   - Consider: Retry logic, alternative payment methods, grace period

9. **How do we communicate errors to clients?**
   - Hint: Clients need to handle errors gracefully
   - Consider: HTTP status codes, error response format

## 3. Step-by-Step Guidance

### Step 1: Identify API Resources
Core resources in the ride-sharing domain:

**Resources:**
- `/users` - User accounts (riders and drivers)
- `/rides` - Ride requests and trips
- `/drivers` - Driver-specific information
- `/locations` - Real-time location data
- `/payments` - Payment methods and transactions
- `/ratings` - Driver and rider ratings
- `/promos` - Promotional codes

### Step 2: Define RESTful Endpoints
Follow REST conventions:

```
Authentication & Users:
POST   /api/v1/auth/register          - Register new user
POST   /api/v1/auth/login             - Login (get token)
POST   /api/v1/auth/refresh           - Refresh token
GET    /api/v1/users/me               - Get current user profile
PUT    /api/v1/users/me               - Update profile
POST   /api/v1/users/me/phone/verify  - Verify phone number

Rides (Rider):
POST   /api/v1/rides                  - Request a ride
GET    /api/v1/rides/:id              - Get ride details
DELETE /api/v1/rides/:id              - Cancel ride
GET    /api/v1/rides                  - Get ride history
POST   /api/v1/rides/:id/rating       - Rate driver

Rides (Driver):
GET    /api/v1/drivers/rides/available - Get available ride requests nearby
POST   /api/v1/rides/:id/accept       - Accept ride
POST   /api/v1/rides/:id/arrive       - Notify arrived at pickup
POST   /api/v1/rides/:id/start        - Start trip
POST   /api/v1/rides/:id/complete     - Complete trip
POST   /api/v1/rides/:id/cancel       - Cancel accepted ride

Driver Status:
PUT    /api/v1/drivers/me/status      - Go online/offline
POST   /api/v1/drivers/me/location    - Update location
GET    /api/v1/drivers/me/earnings    - Get earnings summary

Location:
WebSocket ws://api.example.com/v1/locations/track - Real-time tracking

Payments:
GET    /api/v1/payments/methods       - List payment methods
POST   /api/v1/payments/methods       - Add payment method
DELETE /api/v1/payments/methods/:id   - Remove payment method
POST   /api/v1/payments/rides/:id     - Process payment for ride
GET    /api/v1/payments/history       - Payment history

Fare:
POST   /api/v1/fare/estimate          - Estimate ride fare
```

### Step 3: Design Request/Response Formats
Use consistent, clear JSON structures:

**POST /api/v1/rides (Request a Ride)**
```json
Request:
{
  "pickup": {
    "latitude": 37.7749,
    "longitude": -122.4194,
    "address": "123 Market St, San Francisco, CA"
  },
  "dropoff": {
    "latitude": 37.8049,
    "longitude": -122.4294,
    "address": "456 Mission St, San Francisco, CA"
  },
  "ride_type": "uberx",
  "payment_method_id": "pm_12345",
  "passengers": 1,
  "notes": "Please call when you arrive"
}

Response (201 Created):
{
  "ride_id": "ride_abc123",
  "status": "searching",
  "estimated_fare": {
    "min": 12.50,
    "max": 15.75,
    "currency": "USD"
  },
  "estimated_arrival_time": "2024-12-19T10:45:00Z",
  "pickup": { ... },
  "dropoff": { ... },
  "created_at": "2024-12-19T10:40:00Z"
}
```

**GET /api/v1/rides/:id (Get Ride Status)**
```json
Response (200 OK):
{
  "ride_id": "ride_abc123",
  "status": "driver_assigned",
  "driver": {
    "driver_id": "drv_xyz789",
    "name": "John D.",
    "rating": 4.8,
    "total_trips": 1250,
    "vehicle": {
      "make": "Toyota",
      "model": "Camry",
      "color": "Silver",
      "license_plate": "ABC123"
    },
    "phone": "+1-555-0100",
    "photo_url": "https://cdn.example.com/drivers/xyz789.jpg",
    "location": {
      "latitude": 37.7749,
      "longitude": -122.4194
    }
  },
  "estimated_arrival": "2024-12-19T10:43:00Z",
  "estimated_duration": 180,
  "fare_estimate": {
    "amount": 14.25,
    "currency": "USD"
  },
  "pickup": { ... },
  "dropoff": { ... }
}
```

### Step 4: Handle Authentication & Authorization
Use OAuth 2.0 with JWT tokens:

**Login Flow:**
```
1. Client: POST /api/v1/auth/login
   Body: {"phone": "+1-555-0100", "password": "..."}

2. Server Response:
   {
     "access_token": "eyJhbGc...",
     "refresh_token": "xyz...",
     "token_type": "Bearer",
     "expires_in": 3600,
     "user": {
       "user_id": "usr_123",
       "name": "Alice Smith",
       "role": "rider"
     }
   }

3. Client stores tokens

4. Subsequent requests:
   Header: Authorization: Bearer eyJhbGc...
```

**Role-Based Access Control:**
```python
# Endpoint permissions
/api/v1/rides (POST)           -> Riders only
/api/v1/drivers/rides/available -> Drivers only
/api/v1/rides/:id (GET)        -> Rider or Driver of that ride
```

### Step 5: Implement Real-Time Location Tracking
Use WebSockets for bidirectional communication:

**WebSocket Protocol:**
```javascript
// Client connects
const ws = new WebSocket('wss://api.example.com/v1/locations/track');

// Authenticate
ws.send(JSON.stringify({
  type: 'auth',
  token: 'Bearer eyJhbGc...'
}));

// Driver sends location updates
ws.send(JSON.stringify({
  type: 'location_update',
  ride_id: 'ride_abc123',
  latitude: 37.7749,
  longitude: -122.4194,
  heading: 90,
  speed: 35.5,
  timestamp: '2024-12-19T10:42:00Z'
}));

// Rider receives driver location
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  if (data.type === 'driver_location') {
    updateMapWithDriverLocation(data);
  }
};
```

**Alternative: Polling (Fallback)**
```
GET /api/v1/rides/:id/location
Response:
{
  "driver_location": {
    "latitude": 37.7749,
    "longitude": -122.4194,
    "heading": 90,
    "updated_at": "2024-12-19T10:42:30Z"
  },
  "estimated_arrival": "2024-12-19T10:45:00Z"
}
```

### Step 6: Design Error Responses
Consistent error format across all endpoints:

```json
{
  "error": {
    "code": "PAYMENT_FAILED",
    "message": "Payment method declined",
    "details": "The card was declined by your bank. Please try another payment method.",
    "status": 402,
    "timestamp": "2024-12-19T10:42:00Z",
    "request_id": "req_abc123"
  }
}
```

**HTTP Status Codes:**
- 200 OK - Success
- 201 Created - Resource created
- 400 Bad Request - Invalid input
- 401 Unauthorized - Missing/invalid auth
- 403 Forbidden - Not allowed
- 404 Not Found - Resource doesn't exist
- 409 Conflict - State conflict (e.g., ride already accepted)
- 422 Unprocessable Entity - Validation failed
- 429 Too Many Requests - Rate limit exceeded
- 500 Internal Server Error - Server error
- 503 Service Unavailable - Temporary outage

### Step 7: Implement Rate Limiting
Prevent API abuse:

```
Response Headers:
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000

When exceeded (429 Too Many Requests):
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "details": "Rate limit of 100 requests per minute exceeded. Try again in 30 seconds.",
    "retry_after": 30
  }
}
```

**Rate Limit Tiers:**
- Standard users: 100 requests/minute
- Premium users: 500 requests/minute
- Partner apps: 1000 requests/minute
- Location updates: 1 per second per driver

## 4. Sample Solution

### Complete API Specification

#### Authentication
```
POST /api/v1/auth/register
POST /api/v1/auth/login
POST /api/v1/auth/logout
POST /api/v1/auth/refresh
POST /api/v1/auth/forgot-password
POST /api/v1/auth/reset-password
POST /api/v1/auth/verify-phone
```

#### User Management
```
GET    /api/v1/users/me
PUT    /api/v1/users/me
POST   /api/v1/users/me/avatar
DELETE /api/v1/users/me
GET    /api/v1/users/me/addresses
POST   /api/v1/users/me/addresses
PUT    /api/v1/users/me/addresses/:id
DELETE /api/v1/users/me/addresses/:id
```

#### Rides - Rider
```
POST   /api/v1/rides              - Request ride
GET    /api/v1/rides/:id          - Get ride details
DELETE /api/v1/rides/:id          - Cancel ride
GET    /api/v1/rides              - List ride history
POST   /api/v1/rides/:id/rating   - Rate driver
GET    /api/v1/rides/:id/receipt  - Get ride receipt
POST   /api/v1/rides/:id/share    - Share ride with friend
```

#### Rides - Driver
```
GET    /api/v1/drivers/rides/available - Get available rides
POST   /api/v1/rides/:id/accept        - Accept ride
POST   /api/v1/rides/:id/arrive        - Arrived at pickup
POST   /api/v1/rides/:id/start         - Start trip
POST   /api/v1/rides/:id/complete      - Complete trip
POST   /api/v1/rides/:id/cancel        - Cancel ride
POST   /api/v1/rides/:id/rating        - Rate rider
```

#### Driver Management
```
PUT    /api/v1/drivers/me/status       - Online/offline
POST   /api/v1/drivers/me/location     - Update location
GET    /api/v1/drivers/me/earnings     - Get earnings
GET    /api/v1/drivers/me/stats        - Get statistics
POST   /api/v1/drivers/me/vehicle      - Update vehicle info
POST   /api/v1/drivers/me/documents    - Upload documents
```

#### Payments
```
GET    /api/v1/payments/methods        - List payment methods
POST   /api/v1/payments/methods        - Add payment method
PUT    /api/v1/payments/methods/:id    - Update payment method
DELETE /api/v1/payments/methods/:id    - Remove payment method
GET    /api/v1/payments/history        - Payment history
POST   /api/v1/payments/rides/:id      - Pay for ride
```

#### Fare & Pricing
```
POST   /api/v1/fare/estimate           - Estimate fare
GET    /api/v1/pricing/surge           - Get surge pricing info
```

#### Promotions
```
GET    /api/v1/promos                  - Get available promos
POST   /api/v1/promos/apply            - Apply promo code
GET    /api/v1/promos/:code            - Validate promo code
```

### API Design Decisions Explained

#### 1. RESTful Resource Naming
**Decision**: Use plural nouns (`/rides`, `/drivers`, `/payments`)

**Why**:
- Industry standard (REST conventions)
- Consistent and predictable
- Clear collection vs. individual resource

#### 2. Versioning in URL
**Decision**: `/api/v1/` in URL path

**Why**:
- Explicit and visible
- Easy to support multiple versions
- Can deprecate old versions gradually

**Alternative**: Header versioning (more "pure" REST, but less visible)

#### 3. WebSocket for Location Tracking
**Decision**: Use WebSocket for real-time updates

**Why**:
- Bidirectional communication
- Low latency (push updates)
- Efficient (persistent connection)

**Trade-off**: More complex than polling, but much better performance

#### 4. Nested Resources
**Decision**: `/rides/:id/rating` vs. `/ratings`

**Why**:
- Shows relationship (rating belongs to ride)
- Intuitive for developers
- Prevents orphaned ratings

#### 5. Idempotency Keys
**Decision**: Require idempotency keys for critical operations

```
POST /api/v1/rides
Headers:
  Idempotency-Key: unique-uuid-per-request
```

**Why**:
- Prevent duplicate ride requests
- Network issues cause retries
- Safe to retry without side effects

### Sample Request/Response Examples

#### Request Ride (Full Example)
```http
POST /api/v1/rides HTTP/1.1
Host: api.rideapp.com
Authorization: Bearer eyJhbGc...
Content-Type: application/json
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000

{
  "pickup": {
    "latitude": 37.7749,
    "longitude": -122.4194,
    "address": "123 Market St, San Francisco, CA 94102"
  },
  "dropoff": {
    "latitude": 37.8049,
    "longitude": -122.4294,
    "address": "456 Mission St, San Francisco, CA 94105"
  },
  "ride_type": "standard",
  "payment_method_id": "pm_1234567890",
  "passengers": 2,
  "promo_code": "SAVE20",
  "scheduled_time": null,
  "notes": "Please use south entrance"
}

HTTP/1.1 201 Created
Content-Type: application/json
X-Request-ID: req_abc123
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 99

{
  "ride_id": "ride_7y8z9a0b1c2d",
  "status": "searching_driver",
  "rider": {
    "user_id": "usr_123",
    "name": "Alice Smith",
    "phone": "+1-555-0100",
    "rating": 4.9
  },
  "pickup": {
    "latitude": 37.7749,
    "longitude": -122.4194,
    "address": "123 Market St, San Francisco, CA 94102"
  },
  "dropoff": {
    "latitude": 37.8049,
    "longitude": -122.4294,
    "address": "456 Mission St, San Francisco, CA 94105"
  },
  "ride_type": "standard",
  "passengers": 2,
  "fare_estimate": {
    "min": 15.00,
    "max": 18.50,
    "currency": "USD",
    "breakdown": {
      "base_fare": 3.00,
      "distance": 8.50,
      "time": 4.00,
      "surge_multiplier": 1.0,
      "discount": -2.00,
      "taxes": 1.50
    }
  },
  "estimated_duration": 12,
  "estimated_arrival": "2024-12-19T11:00:00Z",
  "created_at": "2024-12-19T10:45:00Z",
  "expires_at": "2024-12-19T10:50:00Z"
}
```

## 5. Extension Challenges

### Challenge 1: API Documentation
**Requirement**: Create interactive API documentation

**How would you implement this?**
<details>
<summary>Solution</summary>

Use OpenAPI (Swagger) specification:

```yaml
openapi: 3.0.0
info:
  title: Ride Sharing API
  version: 1.0.0
  description: API for ride-sharing platform

paths:
  /api/v1/rides:
    post:
      summary: Request a ride
      security:
        - BearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/RideRequest'
      responses:
        '201':
          description: Ride created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Ride'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'

components:
  schemas:
    RideRequest:
      type: object
      required:
        - pickup
        - dropoff
        - ride_type
      properties:
        pickup:
          $ref: '#/components/schemas/Location'
        dropoff:
          $ref: '#/components/schemas/Location'
        ride_type:
          type: string
          enum: [standard, premium, xl]
    
    Location:
      type: object
      required:
        - latitude
        - longitude
      properties:
        latitude:
          type: number
          format: double
          example: 37.7749
        longitude:
          type: number
          format: double
          example: -122.4194
        address:
          type: string
          example: "123 Market St, San Francisco, CA"
```

Tools:
- Swagger UI for interactive documentation
- Postman collections
- Code generation for client SDKs

</details>

### Challenge 2: API Versioning Strategy
**Requirement**: Support v1 and v2 simultaneously during transition

**How would you handle this?**
<details>
<summary>Solution</summary>

**Option 1: URL Path Versioning** (Recommended)
```
/api/v1/rides - Old version
/api/v2/rides - New version
```

**Option 2: Header Versioning**
```
Header: API-Version: 1
Header: API-Version: 2
```

**Implementation Strategy:**
```python
# Route to different controllers based on version
@app.route('/api/<version>/rides', methods=['POST'])
def create_ride(version):
    if version == 'v1':
        return v1_controllers.create_ride()
    elif version == 'v2':
        return v2_controllers.create_ride()
    else:
        return {'error': 'Unsupported API version'}, 400

# Deprecation headers
response.headers['Deprecated'] = 'true'
response.headers['Sunset'] = '2025-06-01'
response.headers['Link'] = '</api/v2/rides>; rel="successor-version"'
```

**Migration Plan:**
1. Announce v2 with 6-month migration window
2. Run v1 and v2 in parallel
3. Add deprecation warnings to v1 responses
4. Monitor v1 usage (analytics)
5. Sunset v1 after migration window
6. Keep v1 documentation accessible (archived)

</details>

### Challenge 3: Offline Support
**Requirement**: App should work when network is poor

**How would you design the API?**
<details>
<summary>Solution</summary>

**Offline Queue Pattern:**
```javascript
class OfflineQueue {
  constructor() {
    this.queue = [];
  }
  
  // Queue operation when offline
  async executeOrQueue(operation) {
    if (navigator.onLine) {
      try {
        return await operation();
      } catch (error) {
        if (error.code === 'NETWORK_ERROR') {
          this.queue.push(operation);
        }
        throw error;
      }
    } else {
      this.queue.push(operation);
      return { queued: true };
    }
  }
  
  // Sync when back online
  async sync() {
    while (this.queue.length > 0) {
      const operation = this.queue[0];
      try {
        await operation();
        this.queue.shift();
      } catch (error) {
        // Retry later
        break;
      }
    }
  }
}

// Usage
offlineQueue.executeOrQueue(async () => {
  await api.post('/api/v1/drivers/me/location', {
    latitude: 37.7749,
    longitude: -122.4194
  });
});

// When network restored
window.addEventListener('online', () => {
  offlineQueue.sync();
});
```

**API Design for Offline:**
- Idempotency keys (prevent duplicates when syncing)
- Batch operations (sync multiple at once)
- Conflict resolution (server wins vs. client wins)
- Optimistic UI updates

</details>

### Challenge 4: GraphQL Alternative
**Requirement**: Consider GraphQL instead of REST

**What are the trade-offs?**
<details>
<summary>Analysis</summary>

**GraphQL Advantages:**
- Clients request exactly what they need
- Single endpoint for all queries
- Strong typing and introspection
- No over-fetching or under-fetching

**GraphQL Example:**
```graphql
query {
  ride(id: "ride_123") {
    status
    driver {
      name
      rating
      vehicle {
        make
        model
      }
    }
    fare {
      amount
      currency
    }
  }
}
```

**REST Advantages:**
- Simpler to implement
- Better caching (HTTP caching)
- Easier monitoring and debugging
- Better for operations (POST, PUT, DELETE)

**Recommendation for Ride-Sharing:**
Use **REST** because:
- Operations-heavy (request ride, accept ride, complete ride)
- Real-time location better suited for WebSockets
- HTTP caching important (driver profiles, vehicle info)
- Simpler for mobile developers
- Better error handling for state transitions

GraphQL better for:
- Content-heavy apps (social media, news)
- Complex data relationships
- Desktop/web apps with varied data needs

</details>

### Challenge 5: Security Hardening
**Requirement**: Prevent common API security vulnerabilities

**What would you implement?**
<details>
<summary>Solution</summary>

**1. Authentication & Authorization:**
```python
# JWT token with short expiration
access_token_expires = timedelta(minutes=15)
refresh_token_expires = timedelta(days=30)

# Require HTTPS only
@app.before_request
def require_https():
    if not request.is_secure:
        return {'error': 'HTTPS required'}, 403

# Role-based access control
@require_role(['driver'])
def get_available_rides():
    ...
```

**2. Rate Limiting:**
```python
from flask_limiter import Limiter

limiter = Limiter(
    app,
    key_func=get_user_id,
    default_limits=["100 per minute"]
)

@limiter.limit("10 per minute")
@app.route('/api/v1/rides', methods=['POST'])
def create_ride():
    ...
```

**3. Input Validation:**
```python
from marshmallow import Schema, fields, validate

class RideRequestSchema(Schema):
    pickup = fields.Nested(LocationSchema, required=True)
    dropoff = fields.Nested(LocationSchema, required=True)
    ride_type = fields.String(
        required=True,
        validate=validate.OneOf(['standard', 'premium', 'xl'])
    )
    passengers = fields.Integer(
        validate=validate.Range(min=1, max=6)
    )

# Validate all input
schema = RideRequestSchema()
errors = schema.validate(request.json)
if errors:
    return {'error': errors}, 400
```

**4. SQL Injection Prevention:**
```python
# Always use parameterized queries
cursor.execute(
    "SELECT * FROM rides WHERE ride_id = %s",
    (ride_id,)
)

# Never string concatenation
# BAD: f"SELECT * FROM rides WHERE ride_id = '{ride_id}'"
```

**5. Sensitive Data Handling:**
```python
# Never log sensitive data
logger.info(f"Processing payment for ride {ride_id}")
# NOT: logger.info(f"Card number: {card_number}")

# Mask sensitive fields in responses
{
  "payment_method": {
    "type": "card",
    "last_4": "4242",
    "brand": "visa"
    # NOT: "full_number": "4242424242424242"
  }
}
```

**6. CORS Configuration:**
```python
from flask_cors import CORS

# Restrict origins
CORS(app, origins=[
    "https://rideapp.com",
    "https://mobile.rideapp.com"
])
```

**7. Security Headers:**
```python
@app.after_request
def add_security_headers(response):
    response.headers['X-Content-Type-Options'] = 'nosniff'
    response.headers['X-Frame-Options'] = 'DENY'
    response.headers['X-XSS-Protection'] = '1; mode=block'
    response.headers['Strict-Transport-Security'] = 'max-age=31536000'
    return response
```

</details>

---

## Summary

This design exercise demonstrates how to design a production-ready REST API by:
- Following REST conventions and best practices
- Handling real-time requirements with WebSockets
- Implementing robust authentication and authorization
- Designing clear, consistent request/response formats
- Planning for errors and edge cases
- Versioning for evolution
- Optimizing for mobile clients

**Key Takeaways**:
1. **RESTful design**: Use standard HTTP methods and status codes
2. **Consistency is critical**: Predictable patterns across all endpoints
3. **Security first**: Authentication, authorization, rate limiting, validation
4. **Think mobile**: Bandwidth, offline support, battery life
5. **Document thoroughly**: Good docs = happy developers
6. **Version from day 1**: Easier to add than retrofit
