# API Design

## What is an API?

**API (Application Programming Interface)** is a way for different software applications to communicate with each other. Think of it as a menu in a restaurant - it tells you what dishes (services) are available and how to order them.

**Example:**
- You use Instagram app → App calls Instagram API → API returns your feed
- Weather app → Calls weather service API → Gets current temperature
- Payment on website → Calls payment API → Processes your credit card

APIs allow applications to share data and functionality without knowing the internal implementation details.

## Why API Design Matters

Good API design is crucial because:
- **Easy to use**: Developers can integrate quickly
- **Reliable**: Predictable behavior
- **Maintainable**: Can evolve without breaking existing users
- **Efficient**: Fast response times
- **Secure**: Protects data and prevents abuse

Bad API design leads to:
- Frustrated developers
- Security vulnerabilities
- Performance issues
- Difficult to maintain and update

## Types of APIs

### 1. REST (Representational State Transfer)
**Most popular**: Simple, uses HTTP methods

**Key Principles:**
- Uses standard HTTP methods (GET, POST, PUT, DELETE)
- Resource-based URLs
- Stateless (each request independent)
- Returns data (usually JSON)

**Example:**
```
GET    /api/users          - Get all users
GET    /api/users/123      - Get user with ID 123
POST   /api/users          - Create new user
PUT    /api/users/123      - Update user 123
DELETE /api/users/123      - Delete user 123
```

**Request:**
```http
GET /api/users/123 HTTP/1.1
Host: api.example.com
Authorization: Bearer token123
```

**Response:**
```json
{
  "id": 123,
  "name": "Alice",
  "email": "alice@example.com",
  "created_at": "2024-01-15T10:30:00Z"
}
```

---

### 2. GraphQL
**Flexible**: Request exactly the data you need

**Example:**
```graphql
query {
  user(id: 123) {
    name
    email
    posts {
      title
      created_at
    }
  }
}
```

**Response:**
```json
{
  "data": {
    "user": {
      "name": "Alice",
      "email": "alice@example.com",
      "posts": [
        {"title": "My First Post", "created_at": "2024-01-15"}
      ]
    }
  }
}
```

**Pros:**
- Get exactly what you need (no over-fetching)
- Single endpoint
- Strongly typed

**Cons:**
- More complex than REST
- Harder to cache
- Learning curve

---

### 3. WebSocket
**Real-time**: Bidirectional communication

**Use cases**: Chat apps, live updates, gaming

**Example:**
```javascript
// Client connects
const socket = new WebSocket('ws://example.com');

// Receive messages
socket.onmessage = (event) => {
  console.log('New message:', event.data);
};

// Send messages
socket.send('Hello server!');
```

---

### 4. gRPC
**High performance**: Uses Protocol Buffers, HTTP/2

**Best for**: Microservices communication, not public APIs

---

## REST API Design Best Practices

### 1. Use Meaningful URLs (Resource-Based)

**Good:**
```
GET    /api/users              - List users
GET    /api/users/123          - Get specific user
GET    /api/users/123/posts    - Get posts by user 123
POST   /api/posts              - Create a post
```

**Bad:**
```
GET    /api/getAllUsers        - Verb in URL (use GET method instead)
GET    /api/user?id=123        - Use path parameter, not query
POST   /api/createPost         - Verb in URL
GET    /api/posts/delete/123   - Using GET for deletion
```

**Rules:**
- Use nouns, not verbs (verbs are HTTP methods)
- Use plural names (`/users`, not `/user`)
- Use path parameters for IDs (`/users/123`)
- Use query parameters for filtering (`/users?age=25`)

---

### 2. Use Appropriate HTTP Methods

| Method | Purpose | Example |
|--------|---------|---------|
| GET | Read/Retrieve | Get user profile |
| POST | Create | Create new user |
| PUT | Update (full replacement) | Update entire user object |
| PATCH | Partial update | Update only email |
| DELETE | Remove | Delete user account |

**Example:**
```http
# Get all posts
GET /api/posts

# Create new post
POST /api/posts
Content-Type: application/json
{
  "title": "My Post",
  "content": "Hello world"
}

# Update post
PUT /api/posts/123
{
  "title": "Updated Title",
  "content": "Updated content"
}

# Delete post
DELETE /api/posts/123
```

---

### 3. Use Proper HTTP Status Codes

**Success Codes:**
- `200 OK` - Request succeeded (GET, PUT, PATCH)
- `201 Created` - Resource created (POST)
- `204 No Content` - Success but no response body (DELETE)

**Client Error Codes:**
- `400 Bad Request` - Invalid data sent
- `401 Unauthorized` - Not authenticated
- `403 Forbidden` - Authenticated but not allowed
- `404 Not Found` - Resource doesn't exist
- `429 Too Many Requests` - Rate limit exceeded

**Server Error Codes:**
- `500 Internal Server Error` - Something went wrong
- `503 Service Unavailable` - Server temporarily down

**Example:**
```http
# Success
HTTP/1.1 200 OK
Content-Type: application/json
{
  "id": 123,
  "name": "Alice"
}

# Error
HTTP/1.1 404 Not Found
Content-Type: application/json
{
  "error": "User not found",
  "message": "No user exists with ID 123"
}
```

---

### 4. Versioning

Your API will evolve. Versioning prevents breaking existing clients.

**Methods:**

**A. URL Versioning (Common):**
```
GET /api/v1/users
GET /api/v2/users
```

**B. Header Versioning:**
```
GET /api/users
Accept: application/vnd.example.v1+json
```

**C. Query Parameter:**
```
GET /api/users?version=1
```

**Best Practice:**
- Start with v1
- Only increment for breaking changes
- Support old versions for some time (e.g., 6 months)

---

### 5. Pagination

Don't return all data at once!

**Limit-Offset:**
```
GET /api/users?limit=20&offset=40
```

**Page-Based:**
```
GET /api/users?page=2&per_page=20
```

**Cursor-Based (Best for large datasets):**
```
GET /api/users?cursor=abc123&limit=20
```

**Response:**
```json
{
  "data": [...],
  "pagination": {
    "total": 1000,
    "page": 2,
    "per_page": 20,
    "total_pages": 50,
    "next_cursor": "xyz789"
  }
}
```

---

### 6. Filtering, Sorting, and Searching

**Filtering:**
```
GET /api/users?age=25&country=USA
GET /api/posts?author=123&published=true
```

**Sorting:**
```
GET /api/users?sort=created_at&order=desc
GET /api/products?sort=price&order=asc
```

**Searching:**
```
GET /api/users?search=alice
GET /api/products?q=laptop
```

---

### 7. Error Handling

Always return clear error messages!

**Good Error Response:**
```json
{
  "error": {
    "code": "INVALID_EMAIL",
    "message": "The email address format is invalid",
    "field": "email",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

**Validation Errors:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "errors": [
      {
        "field": "email",
        "message": "Email is required"
      },
      {
        "field": "age",
        "message": "Age must be at least 18"
      }
    ]
  }
}
```

---

### 8. Authentication & Authorization

**Authentication**: Who are you?  
**Authorization**: What can you do?

**Common Methods:**

**A. API Keys** (Simple but less secure)
```http
GET /api/users
X-API-Key: abc123xyz789
```

**B. Bearer Token (OAuth, JWT)** (Most common)
```http
GET /api/users
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**C. Basic Auth** (Avoid for production)
```http
GET /api/users
Authorization: Basic base64(username:password)
```

**Best Practices:**
- Use HTTPS always (encrypt data in transit)
- Tokens should expire
- Use refresh tokens for long-lived sessions
- Implement rate limiting

---

### 9. Rate Limiting

Prevent abuse by limiting requests per user.

**Example:**
```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640000000
```

If limit exceeded:
```http
HTTP/1.1 429 Too Many Requests
Retry-After: 3600

{
  "error": "Rate limit exceeded. Try again in 1 hour."
}
```

**Common Limits:**
- Free tier: 100 requests/hour
- Paid tier: 10,000 requests/hour

---

### 10. Documentation

Good documentation is essential!

**Include:**
- Authentication requirements
- All endpoints
- Request/response examples
- Error codes
- Rate limits

**Tools:**
- Swagger/OpenAPI
- Postman
- API Blueprint

**Example (Swagger):**
```yaml
/users:
  get:
    summary: Get all users
    parameters:
      - name: limit
        in: query
        type: integer
    responses:
      200:
        description: Success
        schema:
          type: array
          items:
            $ref: '#/definitions/User'
```

---

## API Security Best Practices

### 1. Use HTTPS
Always encrypt data in transit.

### 2. Validate Input
Never trust user input. Validate everything.

```python
def create_user(data):
    # Validate
    if not is_valid_email(data['email']):
        return error(400, "Invalid email")
    
    if len(data['password']) < 8:
        return error(400, "Password too short")
    
    # Sanitize
    name = sanitize(data['name'])
    
    # Create user
    user = db.create_user(name, data['email'], hash_password(data['password']))
    return success(user)
```

### 3. Prevent SQL Injection
Use parameterized queries.

**Bad:**
```python
query = f"SELECT * FROM users WHERE email = '{email}'"  # Vulnerable!
```

**Good:**
```python
query = "SELECT * FROM users WHERE email = ?"
db.execute(query, [email])  # Safe
```

### 4. Implement Authorization
Check if user has permission for each action.

```python
def delete_post(post_id, user_id):
    post = db.get_post(post_id)
    
    # Check if user owns the post
    if post.author_id != user_id:
        return error(403, "You don't have permission to delete this post")
    
    db.delete_post(post_id)
    return success()
```

### 5. Rate Limiting
Prevent DDoS attacks and abuse.

### 6. CORS (Cross-Origin Resource Sharing)
Control which domains can access your API.

```python
# Allow specific domain
Access-Control-Allow-Origin: https://example.com

# Allow any domain (careful!)
Access-Control-Allow-Origin: *
```

---

## API Performance Optimization

### 1. Caching
```http
GET /api/users/123

# Response with cache headers
HTTP/1.1 200 OK
Cache-Control: max-age=3600
ETag: "abc123"

# Next request
GET /api/users/123
If-None-Match: "abc123"

# If not modified
HTTP/1.1 304 Not Modified
```

### 2. Compression
Compress responses to reduce bandwidth.

```http
GET /api/users
Accept-Encoding: gzip

HTTP/1.1 200 OK
Content-Encoding: gzip
```

### 3. Async Processing
For long operations, return immediately and process in background.

```http
POST /api/videos/process

HTTP/1.1 202 Accepted
Location: /api/jobs/abc123
{
  "job_id": "abc123",
  "status": "processing"
}

# Check status later
GET /api/jobs/abc123
{
  "job_id": "abc123",
  "status": "completed",
  "result_url": "/api/videos/123"
}
```

---

## Real-World Examples

### Twitter API
```
GET /api/v2/tweets/search/recent?query=system%20design
GET /api/v2/users/123/tweets
POST /api/v2/tweets
```

### Stripe Payment API
```
POST /api/v1/charges
Authorization: Bearer sk_test_...
{
  "amount": 2000,
  "currency": "usd",
  "source": "tok_visa"
}
```

### GitHub API
```
GET /api/repos/facebook/react
GET /api/repos/facebook/react/issues
POST /api/repos/myusername/myrepo/issues
```

---

## API Design Checklist

- [ ] Use RESTful resource-based URLs
- [ ] Use appropriate HTTP methods
- [ ] Return proper status codes
- [ ] Version your API
- [ ] Implement pagination
- [ ] Support filtering and sorting
- [ ] Provide clear error messages
- [ ] Require authentication
- [ ] Implement rate limiting
- [ ] Use HTTPS
- [ ] Validate and sanitize input
- [ ] Write comprehensive documentation
- [ ] Include examples
- [ ] Add caching headers
- [ ] Monitor and log requests

---

## Summary

- APIs enable communication between applications
- REST is the most common API style
- Use resource-based URLs and HTTP methods correctly
- Return appropriate status codes
- Version your API to allow evolution
- Implement pagination for large datasets
- Secure your API with authentication and validation
- Document everything clearly
- Performance matters: use caching and compression

---

## Next Steps

Practice API design with the challenges, then learn about **[Microservices](../06-microservices/)** to see how to build large systems from small, independent APIs!
