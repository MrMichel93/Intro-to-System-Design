# API Design - Challenges & Questions

Test your understanding of API design principles!

## 🎯 Multiple Choice Questions

### Question 1
Which URL is designed following REST best practices?

A) `GET /api/getUserById?id=123`  
B) `POST /api/deleteUser/123`  
C) `GET /api/users/123`  
D) `GET /api/users/get/123`  

<details>
<summary>Answer</summary>

**C) `GET /api/users/123`**

REST APIs should use resource-based URLs (nouns, not verbs). The HTTP method (GET) indicates the action. Option A uses a verb in the URL, B uses GET for deletion, and D has unnecessary "get" in the path.
</details>

---

### Question 2
What HTTP status code should you return when a user tries to access a resource they don't have permission to view?

A) 401 Unauthorized  
B) 403 Forbidden  
C) 404 Not Found  
D) 400 Bad Request  

<details>
<summary>Answer</summary>

**B) 403 Forbidden**

- **401 Unauthorized**: User is not authenticated (not logged in)
- **403 Forbidden**: User is authenticated but doesn't have permission
- **404 Not Found**: Resource doesn't exist
- **400 Bad Request**: Invalid request format/data
</details>

---

### Question 3
Which HTTP method should be used to partially update a user's profile?

A) GET  
B) POST  
C) PUT  
D) PATCH  

<details>
<summary>Answer</summary>

**D) PATCH**

- **PATCH**: Partial update (update only specified fields)
- **PUT**: Full replacement (must send entire object)
- **POST**: Create new resource
- **GET**: Retrieve resource
</details>

---

### Question 4
What's the best way to version an API?

A) Never version, just update existing endpoints  
B) Include version in URL: `/api/v1/users`  
C) Create entirely new domain for each version  
D) Change version every month  

<details>
<summary>Answer</summary>

**B) Include version in URL: `/api/v1/users`**

URL versioning is the most common and clear approach. It makes it obvious which version is being used and allows supporting multiple versions simultaneously.
</details>

---

### Question 5
Your API endpoint returns 10,000 user records. What should you do?

A) Return all 10,000 records  
B) Implement pagination  
C) Return error message  
D) Randomly pick 100 records  

<details>
<summary>Answer</summary>

**B) Implement pagination**

Returning large amounts of data is slow and wasteful. Pagination allows clients to fetch data in manageable chunks:
```
GET /api/users?page=1&per_page=50
```
</details>

---

### Question 6
Which authentication method is most secure for modern web APIs?

A) Passing password in URL  
B) Basic Auth over HTTP  
C) Bearer Token (JWT) over HTTPS  
D) No authentication  

<details>
<summary>Answer</summary>

**C) Bearer Token (JWT) over HTTPS**

JWT tokens over HTTPS provide good security:
- Tokens can expire
- HTTPS encrypts data in transit
- Tokens can include claims (permissions)
- No need to send password with each request

Never send passwords in URLs or use HTTP without encryption!
</details>

---

## 🧩 Scenario-Based Challenges

### Challenge 1: Design a Blog API

Design a RESTful API for a blog platform:

**Features:**
- Users can create, read, update, delete posts
- Users can comment on posts
- Users can like posts
- Get posts by author
- Search posts by keyword

**Your Task:** Define the endpoints, HTTP methods, and request/response formats.

<details>
<summary>Sample Solution</summary>

**Blog API Design:**

```
Authentication:
All requests require: Authorization: Bearer {token}

Base URL: https://api.blog.com/v1

=== Posts ===

1. Get all posts
GET /posts?page=1&per_page=20&sort=created_at&order=desc
Response: 200 OK
{
  "data": [
    {
      "id": 1,
      "title": "My First Post",
      "content": "...",
      "author": {
        "id": 123,
        "name": "Alice"
      },
      "likes_count": 42,
      "comments_count": 5,
      "created_at": "2024-01-15T10:30:00Z"
    }
  ],
  "pagination": {
    "total": 100,
    "page": 1,
    "per_page": 20
  }
}

2. Get specific post
GET /posts/1
Response: 200 OK
{
  "id": 1,
  "title": "My First Post",
  "content": "Full content here...",
  "author": {...},
  "likes_count": 42,
  "created_at": "2024-01-15T10:30:00Z"
}

3. Create post
POST /posts
Content-Type: application/json
{
  "title": "New Post Title",
  "content": "Post content here...",
  "tags": ["tech", "tutorial"]
}
Response: 201 Created
Location: /posts/123
{
  "id": 123,
  "title": "New Post Title",
  "content": "Post content here...",
  "author": {...},
  "created_at": "2024-01-15T10:35:00Z"
}

4. Update post
PUT /posts/123
{
  "title": "Updated Title",
  "content": "Updated content"
}
Response: 200 OK

5. Delete post
DELETE /posts/123
Response: 204 No Content

6. Get posts by author
GET /users/123/posts
Response: 200 OK
{
  "data": [...],
  "pagination": {...}
}

7. Search posts
GET /posts/search?q=system+design&page=1
Response: 200 OK
{
  "data": [...],
  "pagination": {...}
}

=== Comments ===

8. Get comments for a post
GET /posts/123/comments?page=1&per_page=10
Response: 200 OK
{
  "data": [
    {
      "id": 1,
      "content": "Great post!",
      "author": {
        "id": 456,
        "name": "Bob"
      },
      "created_at": "2024-01-15T11:00:00Z"
    }
  ]
}

9. Add comment
POST /posts/123/comments
{
  "content": "Nice article!"
}
Response: 201 Created

10. Update comment
PATCH /comments/1
{
  "content": "Updated comment"
}
Response: 200 OK

11. Delete comment
DELETE /comments/1
Response: 204 No Content

=== Likes ===

12. Like a post
POST /posts/123/likes
Response: 201 Created
{
  "post_id": 123,
  "user_id": 789,
  "created_at": "2024-01-15T11:05:00Z"
}

13. Unlike a post
DELETE /posts/123/likes
Response: 204 No Content

14. Get users who liked a post
GET /posts/123/likes?page=1
Response: 200 OK
{
  "data": [
    {
      "user_id": 456,
      "name": "Bob",
      "liked_at": "2024-01-15T11:00:00Z"
    }
  ]
}

=== Error Responses ===

Invalid request:
400 Bad Request
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Title is required",
    "field": "title"
  }
}

Unauthorized:
401 Unauthorized
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Authentication required"
  }
}

Forbidden:
403 Forbidden
{
  "error": {
    "code": "FORBIDDEN",
    "message": "You don't have permission to delete this post"
  }
}

Not found:
404 Not Found
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Post not found"
  }
}

Rate limit exceeded:
429 Too Many Requests
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Try again in 1 hour."
  }
}
```

**API Best Practices Applied:**
✅ RESTful resource-based URLs
✅ Proper HTTP methods
✅ Appropriate status codes
✅ Pagination
✅ Filtering and sorting
✅ Nested resources (/posts/123/comments)
✅ Clear error messages
✅ Consistent response format
</details>

---

### Challenge 2: Handle Authentication

Design the authentication flow for your API:

**Requirements:**
- User registration
- User login
- Token-based authentication
- Token refresh
- Logout

**Your Task:** Define endpoints and the auth flow.

<details>
<summary>Sample Solution</summary>

**Authentication API Design:**

```
=== Registration ===

POST /auth/register
Content-Type: application/json
{
  "name": "Alice",
  "email": "alice@example.com",
  "password": "SecurePass123!"
}

Response: 201 Created
{
  "user": {
    "id": 123,
    "name": "Alice",
    "email": "alice@example.com",
    "created_at": "2024-01-15T10:00:00Z"
  },
  "tokens": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "def50200e3...",
    "expires_in": 3600,
    "token_type": "Bearer"
  }
}

=== Login ===

POST /auth/login
Content-Type: application/json
{
  "email": "alice@example.com",
  "password": "SecurePass123!"
}

Response: 200 OK
{
  "user": {
    "id": 123,
    "name": "Alice",
    "email": "alice@example.com"
  },
  "tokens": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "def50200e3...",
    "expires_in": 3600,
    "token_type": "Bearer"
  }
}

Error (invalid credentials):
401 Unauthorized
{
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Email or password is incorrect"
  }
}

=== Using Access Token ===

GET /api/posts
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

Response: 200 OK
{...}

=== Token Expiration ===

When access token expires:
GET /api/posts
Authorization: Bearer expired_token

Response: 401 Unauthorized
{
  "error": {
    "code": "TOKEN_EXPIRED",
    "message": "Access token has expired. Use refresh token to get new access token."
  }
}

=== Refresh Token ===

POST /auth/refresh
Content-Type: application/json
{
  "refresh_token": "def50200e3..."
}

Response: 200 OK
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in": 3600,
  "token_type": "Bearer"
}

=== Logout ===

POST /auth/logout
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

Response: 200 OK
{
  "message": "Successfully logged out"
}

(Server invalidates the refresh token)

=== Get Current User ===

GET /auth/me
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

Response: 200 OK
{
  "id": 123,
  "name": "Alice",
  "email": "alice@example.com"
}
```

**Token Strategy:**

**Access Token (JWT):**
- Short-lived (15 minutes - 1 hour)
- Includes user ID and permissions
- Verified on each request
- Stored in memory (not localStorage for security)

**Refresh Token:**
- Long-lived (7-30 days)
- Stored securely in database
- Used only to get new access tokens
- Can be revoked

**Security Measures:**
1. Hash passwords (bcrypt)
2. Use HTTPS only
3. Validate email format
4. Enforce strong passwords
5. Rate limit login attempts
6. Implement account lockout after failed attempts
7. Add CSRF protection
8. Use secure cookies for refresh tokens (httpOnly, secure, sameSite)

**Flow Diagram:**
```
1. User registers/logs in
   ↓
2. Server returns access token + refresh token
   ↓
3. Client uses access token for API requests
   ↓
4. Access token expires
   ↓
5. Client uses refresh token to get new access token
   ↓
6. Repeat step 3-5
   ↓
7. User logs out → Refresh token invalidated
```
</details>

---

### Challenge 3: Rate Limiting Design

Implement rate limiting for your API:

**Requirements:**
- Free users: 100 requests/hour
- Premium users: 1000 requests/hour
- Return rate limit info in headers
- Return clear error when limit exceeded

**Your Task:** Design the rate limiting mechanism.

<details>
<summary>Sample Solution</summary>

**Rate Limiting Implementation:**

**Response Headers (on every request):**
```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
X-RateLimit-Used: 5
```

**When Limit Exceeded:**
```http
HTTP/1.1 429 Too Many Requests
Retry-After: 3600
Content-Type: application/json

{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "You have exceeded your rate limit of 100 requests per hour",
    "retry_after": 3600,
    "limit": 100,
    "reset_at": "2024-01-15T12:00:00Z"
  }
}
```

**Algorithm: Token Bucket**

```python
import time
import redis

class RateLimiter:
    def __init__(self, redis_client):
        self.redis = redis_client
    
    def check_rate_limit(self, user_id, limit=100, window=3600):
        """
        Check if user is within rate limit.
        
        Args:
            user_id: User identifier
            limit: Max requests per window
            window: Time window in seconds (default 1 hour)
        
        Returns:
            dict: Rate limit status
        """
        key = f"rate_limit:{user_id}"
        current_time = int(time.time())
        window_start = current_time - window
        
        # Remove old requests outside window
        self.redis.zremrangebyscore(key, 0, window_start)
        
        # Count requests in current window
        request_count = self.redis.zcard(key)
        
        # Calculate reset time
        reset_time = current_time + window
        
        if request_count >= limit:
            # Rate limit exceeded
            return {
                "allowed": False,
                "limit": limit,
                "remaining": 0,
                "used": request_count,
                "reset": reset_time,
                "retry_after": window
            }
        
        # Add current request
        self.redis.zadd(key, {current_time: current_time})
        self.redis.expire(key, window)
        
        return {
            "allowed": True,
            "limit": limit,
            "remaining": limit - request_count - 1,
            "used": request_count + 1,
            "reset": reset_time
        }

# Usage in API endpoint
def api_endpoint(request):
    user_id = get_user_id(request)
    user_tier = get_user_tier(user_id)
    
    # Different limits for different tiers
    limit = 1000 if user_tier == "premium" else 100
    
    rate_limit = rate_limiter.check_rate_limit(user_id, limit=limit)
    
    # Add headers
    response_headers = {
        "X-RateLimit-Limit": rate_limit["limit"],
        "X-RateLimit-Remaining": rate_limit["remaining"],
        "X-RateLimit-Reset": rate_limit["reset"],
        "X-RateLimit-Used": rate_limit["used"]
    }
    
    if not rate_limit["allowed"]:
        return error_response(
            status=429,
            error_code="RATE_LIMIT_EXCEEDED",
            message=f"Rate limit of {limit} requests per hour exceeded",
            headers=response_headers
        )
    
    # Process request normally
    return success_response(data, headers=response_headers)
```

**Alternative: Fixed Window**
```python
def check_rate_limit_fixed_window(user_id, limit=100):
    """Simpler but has edge case issues"""
    current_hour = int(time.time() / 3600)
    key = f"rate_limit:{user_id}:{current_hour}"
    
    count = redis.incr(key)
    
    if count == 1:
        redis.expire(key, 3600)
    
    if count > limit:
        return {"allowed": False, "remaining": 0}
    
    return {"allowed": True, "remaining": limit - count}
```

**Advanced: Distributed Rate Limiting**

For multiple API servers:
```python
# Use Redis for shared state across servers
# Lua script for atomic operations
lua_script = """
local key = KEYS[1]
local limit = tonumber(ARGV[1])
local window = tonumber(ARGV[2])
local current = tonumber(ARGV[3])

redis.call('zremrangebyscore', key, 0, current - window)
local count = redis.call('zcard', key)

if count < limit then
    redis.call('zadd', key, current, current)
    redis.call('expire', key, window)
    return {1, limit - count - 1}
else
    return {0, 0}
end
"""

# Execute atomically
result = redis.eval(lua_script, 1, key, limit, window, current_time)
```

**Bypass Rate Limits (for testing):**
```python
# Add header to bypass (internal/admin only)
if request.headers.get('X-Bypass-Rate-Limit') == secret_token:
    # Skip rate limiting
    pass
```

**Monitoring:**
- Track rate limit hits per user
- Alert if many users hitting limits (may need to increase)
- Monitor for abuse patterns
</details>

---

### Challenge 4: API Error Handling

Design comprehensive error handling for your API.

**Create error responses for:**
1. Validation errors (multiple fields invalid)
2. Authentication errors
3. Authorization errors
4. Not found errors
5. Server errors
6. External service failures

<details>
<summary>Sample Solution</summary>

**Comprehensive Error Handling:**

**1. Validation Errors (400):**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "timestamp": "2024-01-15T10:30:00Z",
    "path": "/api/users",
    "errors": [
      {
        "field": "email",
        "message": "Email format is invalid",
        "code": "INVALID_FORMAT",
        "value": "not-an-email"
      },
      {
        "field": "password",
        "message": "Password must be at least 8 characters",
        "code": "TOO_SHORT",
        "constraint": {
          "min_length": 8
        }
      },
      {
        "field": "age",
        "message": "Age must be between 13 and 120",
        "code": "OUT_OF_RANGE",
        "constraint": {
          "min": 13,
          "max": 120
        }
      }
    ]
  }
}
```

**2. Authentication Errors (401):**

Missing token:
```json
{
  "error": {
    "code": "AUTHENTICATION_REQUIRED",
    "message": "Authentication token is required",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

Invalid token:
```json
{
  "error": {
    "code": "INVALID_TOKEN",
    "message": "Authentication token is invalid or malformed",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

Expired token:
```json
{
  "error": {
    "code": "TOKEN_EXPIRED",
    "message": "Authentication token has expired",
    "expired_at": "2024-01-15T09:30:00Z",
    "instructions": "Use refresh token to obtain new access token"
  }
}
```

**3. Authorization Errors (403):**
```json
{
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "You don't have permission to perform this action",
    "required_permission": "posts:delete",
    "your_permissions": ["posts:read", "posts:create"],
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

**4. Not Found Errors (404):**
```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "The requested resource was not found",
    "resource_type": "User",
    "resource_id": "123",
    "path": "/api/users/123",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

**5. Server Errors (500):**
```json
{
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "An unexpected error occurred. Please try again later.",
    "error_id": "err_abc123xyz",
    "timestamp": "2024-01-15T10:30:00Z",
    "support": "Contact support@example.com with error_id if problem persists"
  }
}
```

**6. External Service Failures (503):**
```json
{
  "error": {
    "code": "SERVICE_UNAVAILABLE",
    "message": "External payment service is temporarily unavailable",
    "service": "payment_gateway",
    "retry_after": 60,
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

**7. Rate Limiting (429):**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "You have exceeded your request quota",
    "limit": 100,
    "window": "1 hour",
    "retry_after": 1847,
    "reset_at": "2024-01-15T11:00:00Z",
    "upgrade_url": "https://example.com/upgrade"
  }
}
```

**Error Handling Implementation:**

```python
class APIError(Exception):
    """Base API error"""
    def __init__(self, code, message, status=400, **kwargs):
        self.code = code
        self.message = message
        self.status = status
        self.extra = kwargs

class ValidationError(APIError):
    """Validation failed"""
    def __init__(self, errors):
        super().__init__(
            code="VALIDATION_ERROR",
            message="Request validation failed",
            status=400,
            errors=errors
        )

class NotFoundError(APIError):
    """Resource not found"""
    def __init__(self, resource_type, resource_id):
        super().__init__(
            code="RESOURCE_NOT_FOUND",
            message=f"{resource_type} not found",
            status=404,
            resource_type=resource_type,
            resource_id=resource_id
        )

# Error handler middleware
def error_handler(error):
    if isinstance(error, APIError):
        response = {
            "error": {
                "code": error.code,
                "message": error.message,
                "timestamp": datetime.utcnow().isoformat() + "Z",
                **error.extra
            }
        }
        return json_response(response, status=error.status)
    
    # Unexpected error - log and return generic message
    logger.error(f"Unexpected error: {error}", exc_info=True)
    error_id = generate_error_id()
    
    response = {
        "error": {
            "code": "INTERNAL_SERVER_ERROR",
            "message": "An unexpected error occurred",
            "error_id": error_id,
            "timestamp": datetime.utcnow().isoformat() + "Z"
        }
    }
    return json_response(response, status=500)
```

**Best Practices:**
1. **Consistent format**: All errors follow same structure
2. **Actionable messages**: Tell users what went wrong and how to fix
3. **Error codes**: Machine-readable codes for programmatic handling
4. **No sensitive info**: Don't expose stack traces or internal details
5. **Logging**: Log full error details server-side with error_id
6. **Help users**: Provide links to docs or support
</details>

---

## 🤔 Critical Thinking Questions

1. **Why is versioning important, and when should you create a new version?**

2. **What are the trade-offs between REST and GraphQL?**

3. **How would you design an API that needs to support both web and mobile clients with different needs?**

4. **Why should you avoid returning database IDs directly in URLs (security perspective)?**

---

## Next Steps

Great job mastering API design! Now explore **[Microservices](../06-microservices/)** to learn how to build scalable systems from multiple small APIs!
