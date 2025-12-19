# Design Exercise: System Design Foundations

Practice what you've learned by working through these hands-on design exercises!

## Exercise 1: Requirements Gathering Practice

### Scenario: School Library System

You've been asked to design a digital system for your school library.

**Your Task:**
Write down the requirements by answering these questions:

#### Functional Requirements
1. What can students do in this system?
2. What can librarians do in this system?
3. What can teachers do in this system?
4. What happens when a book is borrowed?
5. What happens when a book is returned late?

#### Non-Functional Requirements
1. How many students will use the system? (Scale)
2. How fast should search results appear? (Performance)
3. Can the system be unavailable at any time? (Availability)
4. What happens if the system crashes? Can we lose data? (Reliability)
5. How should student information be protected? (Security)

<details>
<summary>Sample Solution</summary>

**Functional Requirements:**
- Students can:
  - Search for books by title, author, or subject
  - Check availability of books
  - View their borrowed books
  - Reserve available books
  - See due dates and overdue notices
  
- Librarians can:
  - Add new books to the catalog
  - Mark books as borrowed/returned
  - Send overdue notifications
  - Generate reports on popular books
  - Manage student accounts
  
- Teachers can:
  - Reserve books for classes
  - View recommended books for subjects
  - Request new books

**Non-Functional Requirements:**
- **Scale**: 2,000 students, 20,000 books, 100 concurrent users
- **Performance**: Search results in < 2 seconds, borrowing process < 1 second
- **Availability**: 99% uptime (can be down briefly for maintenance)
- **Reliability**: Book records must never be lost, backup daily
- **Security**: Student data encrypted, authentication required, role-based access
- **Cost**: Must run on school's limited budget (cloud-friendly but cost-conscious)

</details>

---

## Exercise 2: Component Identification

### Scenario: Food Delivery App (like Uber Eats)

For a food delivery app, identify which components are needed and why.

**Components to Consider:**
- Client application
- Web server
- Database
- Cache
- Load balancer
- CDN
- Message queue
- External APIs

**Your Task:**
For each component, decide:
1. Do we need it? (Yes/No)
2. Why or why not?
3. What would it be used for?

<details>
<summary>Sample Solution</summary>

| Component | Needed? | Purpose |
|-----------|---------|---------|
| **Client Application** | ✅ Yes | Users need apps to browse restaurants, place orders |
| **Web Server** | ✅ Yes | Process orders, handle business logic |
| **Database** | ✅ Yes | Store restaurants, menus, orders, users |
| **Cache** | ✅ Yes | Cache restaurant lists, menus (don't change often) |
| **Load Balancer** | ✅ Yes | Distribute traffic during lunch/dinner rushes |
| **CDN** | ✅ Yes | Serve food images fast from locations near users |
| **Message Queue** | ✅ Yes | Handle order notifications to restaurants, delivery updates |
| **External APIs** | ✅ Yes | Payment processing (Stripe), Maps (Google Maps), SMS (Twilio) |

**Additional Components:**
- **Real-time tracking service**: Track delivery drivers
- **Notification service**: Push notifications for order updates
- **Search service**: Fast restaurant and food search

</details>

---

## Exercise 3: Architecture Design

### Scenario: Online Voting System

Design a simple architecture for a school club election voting system.

**Requirements:**
- 500 students can vote
- Each student votes once
- Results should be shown after voting closes
- Must be secure and reliable
- Voting period: 24 hours

**Your Task:**
1. Draw or describe the architecture
2. List all components
3. Explain how data flows from voter to database
4. Identify potential problems

<details>
<summary>Sample Solution</summary>

**Architecture:**

```
┌─────────────┐
│  Students   │
│  (Browser)  │
└──────┬──────┘
       │ HTTPS
┌──────▼──────┐
│   Web App   │
│  (React)    │
└──────┬──────┘
       │ API Calls
┌──────▼──────────┐
│  Web Server     │
│  (Node.js)      │
│  - Verify auth  │
│  - Check if     │
│    already voted│
│  - Record vote  │
└──────┬──────────┘
       │
┌──────▼──────────┐
│   Database      │
│  (PostgreSQL)   │
│                 │
│  Tables:        │
│  - students     │
│  - votes        │
│  - candidates   │
└─────────────────┘
```

**Components:**

1. **Frontend (React)**: 
   - Login with student ID
   - Display candidates
   - Submit vote button
   - Show confirmation

2. **Backend (Node.js)**:
   - Authentication (verify student ID)
   - Vote validation (one vote per student)
   - Store vote securely
   - Calculate results

3. **Database (PostgreSQL)**:
   - Students table (id, name, has_voted boolean)
   - Votes table (vote_id, candidate_id, timestamp)
   - Candidates table (id, name, position)

**Data Flow:**
1. Student logs in with school ID
2. Server verifies student identity
3. Server checks if student already voted
4. If not voted: Display ballot
5. Student selects candidate and submits
6. Server validates and records vote
7. Server marks student as "has_voted"
8. Show confirmation to student

**Security Measures:**
- HTTPS encryption
- Authentication required
- One vote per student (checked in database)
- Votes are anonymous (no link between student ID and vote)
- Voting only during specified time window

**Potential Problems & Solutions:**

| Problem | Solution |
|---------|----------|
| Student votes multiple times | Database constraint: check has_voted before allowing vote |
| Server crashes during voting | Use database transactions (all-or-nothing) |
| Results leaked early | Only show results after voting period ends |
| Database fails | Regular backups, database replication |
| Too many students vote at once | Load balancer with multiple servers |
| How to verify vote counted? | Give voter a unique receipt code |

</details>

---

## Exercise 4: Scaling Decisions

### Scenario: Growing Photo Sharing App

Your photo sharing app has grown:
- **Month 1**: 100 users, one server, works great
- **Month 6**: 10,000 users, server slowing down
- **Month 12**: 100,000 users, frequent crashes

**Your Task:**
For each stage, recommend changes to the system:

1. **Current Setup (Month 1):**
   - 1 web server
   - 1 database
   - Photos stored on server disk

2. **What to do at Month 6?**

3. **What to do at Month 12?**

<details>
<summary>Sample Solution</summary>

**Month 1 (100 users) - Current Setup:**
```
[Web Server + Database + File Storage]
```
✅ This is fine for 100 users. Keep it simple!

---

**Month 6 (10,000 users) - First Optimizations:**

**Changes Needed:**

1. **Move photos to cloud storage (AWS S3)**
   - Why: Server disk filling up
   - Why: Cheaper and more reliable than managing disk space
   
2. **Add caching layer (Redis)**
   - Cache frequently viewed photos metadata
   - Cache user profiles
   - Reduces database queries by 70%

3. **Vertical scaling**
   - Upgrade server: 2GB RAM → 8GB RAM
   - Still simple, but more capacity

**Updated Architecture:**
```
[Web Server (8GB)] → [Redis Cache]
         ↓
   [Database]
         ↓
   [Cloud Storage - S3]
```

---

**Month 12 (100,000 users) - Horizontal Scaling:**

**Changes Needed:**

1. **Load Balancer + Multiple Web Servers**
   - Add load balancer to distribute traffic
   - Run 3-5 web servers
   - If one crashes, others keep working

2. **Separate Database Server**
   - Move database to dedicated server
   - Better performance
   - Easier to scale independently

3. **CDN for Photos**
   - Serve photos from CDN (CloudFront)
   - Much faster for users worldwide
   - Reduces bandwidth costs

4. **Read Replicas for Database**
   - Create 2-3 read-only database copies
   - Direct read queries to replicas
   - Master handles writes only

**Final Architecture:**
```
       [CDN] (for photos)
         ↑
    [Load Balancer]
         ↓
[Server1] [Server2] [Server3]
         ↓
    [Redis Cache]
         ↓
    [Master DB]
         ↓
  [Read Replica 1] [Read Replica 2]
```

**Cost Consideration:**
- Month 1: ~$50/month (single server)
- Month 6: ~$200/month (upgraded server + cache + storage)
- Month 12: ~$800/month (multiple servers + database + CDN + cache)

**Next Steps (if reaching 1 million users):**
- Auto-scaling (add/remove servers automatically)
- Database sharding (split data across multiple databases)
- Microservices (separate upload, feed, search services)
- Multiple data centers for global reach

</details>

---

## Exercise 5: Trade-Offs Analysis

### Scenario: Video Chat Application

You're designing a video chat app. You need to make several design decisions.

**For each decision, analyze the trade-offs:**

### Decision 1: Where to process video?

**Option A: Client-Side Processing**
- Video encoding/decoding happens on user's device

**Option B: Server-Side Processing**
- Video sent to server, processed there, then sent to recipient

**Your Task:** List pros and cons of each, then choose one.

<details>
<summary>Analysis</summary>

| Aspect | Client-Side | Server-Side |
|--------|-------------|-------------|
| **Server Costs** | Low (server just routes) | High (processing requires powerful servers) |
| **User Device Requirements** | Must be powerful enough | Can work on low-end devices |
| **Latency** | Lower (direct peer-to-peer) | Higher (goes through server) |
| **Scalability** | Excellent (users do the work) | Limited (server can bottleneck) |
| **Features** | Limited (depends on device) | More features (server can modify stream) |
| **Privacy** | Better (encrypted end-to-end) | Lower (server sees video) |

**Recommendation:**
Use **Client-Side Processing** (Option A) for video chat because:
- Lower latency is critical for real-time video
- More cost-effective at scale
- Better privacy (end-to-end encryption)
- This is what Zoom, FaceTime, and WhatsApp do

**When to Use Server-Side:**
- Recording features
- Adding filters/effects
- Broadcasting to many viewers (streaming)

</details>

---

### Decision 2: How to store chat history?

**Option A: SQL Database (PostgreSQL)**
- Structured, relational data

**Option B: NoSQL Database (MongoDB)**
- Flexible schema, document-based

**Your Task:** Which would you choose and why?

<details>
<summary>Analysis</summary>

| Aspect | SQL (PostgreSQL) | NoSQL (MongoDB) |
|--------|------------------|-----------------|
| **Schema** | Fixed, must define upfront | Flexible, can change easily |
| **Queries** | Complex queries easy (SQL) | Simple queries fast, complex harder |
| **Scalability** | Vertical scaling easier | Horizontal scaling easier |
| **Consistency** | Strong (ACID guarantees) | Eventual (but configurable) |
| **Relationships** | Easy (foreign keys, joins) | Harder (manual references) |
| **Speed** | Good for complex queries | Faster for simple reads/writes |

**Recommendation:**
Use **SQL (PostgreSQL)** because:
- Chat messages have clear structure (id, sender, recipient, message, timestamp)
- Need to query conversations between users (relationships)
- Message order is critical (SQL handles well)
- Strong consistency important (messages shouldn't be lost or duplicated)

**When to Use NoSQL:**
- If storing varied message types (text, images, videos, polls)
- If planning to shard across many databases
- If need extremely high write throughput

</details>

---

## Exercise 6: Design from Scratch

### Scenario: Design a Ride-Sharing App (Mini Uber)

**Requirements:**
- Riders request rides
- Drivers accept ride requests
- Real-time location tracking
- Fare calculation
- Payment processing
- 10,000 daily active users (starting point)

**Your Task:**

1. **List all functional requirements**
2. **List non-functional requirements**
3. **Identify all major components**
4. **Draw/describe the architecture**
5. **Design the database schema**
6. **List API endpoints**
7. **Identify potential problems and solutions**

<details>
<summary>Sample Solution</summary>

**1. Functional Requirements:**

Riders can:
- Create account / login
- Request a ride (pickup and destination)
- See nearby drivers
- Track driver location in real-time
- See fare estimate
- Pay for ride
- Rate driver
- View ride history

Drivers can:
- Create account / login
- Go online/offline
- See ride requests
- Accept/decline rides
- Navigate to pickup and destination
- Mark ride as complete
- Rate rider
- View earnings

System should:
- Match riders with nearby drivers
- Calculate fares based on distance and time
- Process payments
- Send notifications
- Handle simultaneous requests

**2. Non-Functional Requirements:**

- **Performance**: Match driver within 30 seconds
- **Scalability**: 10,000 concurrent users
- **Availability**: 99.9% uptime
- **Reliability**: No lost ride requests or payments
- **Security**: Encrypted payments, user data protection
- **Real-time**: Location updates every 2-3 seconds

**3. Major Components:**

- **Rider Mobile App** (iOS/Android)
- **Driver Mobile App** (iOS/Android)
- **API Gateway** (routes requests)
- **Location Service** (tracks driver locations)
- **Matching Service** (pairs riders with drivers)
- **Fare Calculation Service** (estimates and calculates fares)
- **Payment Service** (processes payments)
- **Notification Service** (push notifications)
- **User Service** (manages accounts)
- **Trip Service** (manages ride lifecycle)
- **Database** (stores users, trips, payments)
- **Map Service** (Google Maps API)
- **Payment Gateway** (Stripe API)

**4. Architecture:**

```
┌─────────┐  ┌─────────┐
│ Rider   │  │ Driver  │
│  App    │  │  App    │
└────┬────┘  └────┬────┘
     │            │
     └──────┬─────┘
            │ HTTPS/WebSocket
     ┌──────▼──────┐
     │ API Gateway │
     └──────┬──────┘
            │
     ┌──────▼──────────────────────┐
     │      Microservices          │
     │                              │
     │  ┌──────────┐  ┌──────────┐│
     │  │Location  │  │ Matching ││
     │  │ Service  │  │ Service  ││
     │  └──────────┘  └──────────┘│
     │                              │
     │  ┌──────────┐  ┌──────────┐│
     │  │   Fare   │  │ Payment  ││
     │  │ Service  │  │ Service  ││
     │  └──────────┘  └──────────┘│
     │                              │
     │  ┌──────────┐  ┌──────────┐│
     │  │   User   │  │   Trip   ││
     │  │ Service  │  │ Service  ││
     │  └──────────┘  └──────────┘│
     └─────────┬────────────────────┘
               │
     ┌─────────▼─────────┐
     │    Databases      │
     │                   │
     │  ┌────┐  ┌────┐  │
     │  │User│  │Trip│  │
     │  │ DB │  │ DB │  │
     │  └────┘  └────┘  │
     │                   │
     │  ┌─────────────┐  │
     │  │  Location   │  │
     │  │Cache (Redis)│  │
     │  └─────────────┘  │
     └───────────────────┘
```

**5. Database Schema:**

```sql
-- Users Table
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  phone VARCHAR(20) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  user_type ENUM('rider', 'driver'),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Drivers Table (additional info for drivers)
CREATE TABLE drivers (
  user_id UUID PRIMARY KEY REFERENCES users(id),
  vehicle_type VARCHAR(50),
  license_plate VARCHAR(20),
  rating DECIMAL(2,1),
  total_trips INTEGER DEFAULT 0,
  is_online BOOLEAN DEFAULT FALSE,
  current_lat DECIMAL(10,8),
  current_lng DECIMAL(11,8),
  last_location_update TIMESTAMP
);

-- Trips Table
CREATE TABLE trips (
  id UUID PRIMARY KEY,
  rider_id UUID REFERENCES users(id),
  driver_id UUID REFERENCES users(id),
  status ENUM('requested', 'accepted', 'in_progress', 'completed', 'cancelled'),
  pickup_lat DECIMAL(10,8),
  pickup_lng DECIMAL(11,8),
  pickup_address TEXT,
  dropoff_lat DECIMAL(10,8),
  dropoff_lng DECIMAL(11,8),
  dropoff_address TEXT,
  fare_estimate DECIMAL(10,2),
  final_fare DECIMAL(10,2),
  distance_km DECIMAL(8,2),
  duration_minutes INTEGER,
  requested_at TIMESTAMP DEFAULT NOW(),
  accepted_at TIMESTAMP,
  started_at TIMESTAMP,
  completed_at TIMESTAMP
);

-- Payments Table
CREATE TABLE payments (
  id UUID PRIMARY KEY,
  trip_id UUID REFERENCES trips(id),
  amount DECIMAL(10,2),
  payment_method VARCHAR(50),
  status ENUM('pending', 'completed', 'failed'),
  stripe_payment_id VARCHAR(255),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Ratings Table
CREATE TABLE ratings (
  id UUID PRIMARY KEY,
  trip_id UUID REFERENCES trips(id),
  rater_id UUID REFERENCES users(id),
  ratee_id UUID REFERENCES users(id),
  rating INTEGER CHECK (rating BETWEEN 1 AND 5),
  comment TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);
```

**6. API Endpoints:**

**User Management:**
```
POST   /api/auth/register     - Create account
POST   /api/auth/login        - Login
GET    /api/users/profile     - Get user profile
PUT    /api/users/profile     - Update profile
```

**Rider Endpoints:**
```
POST   /api/rides/request     - Request a ride
GET    /api/rides/:id         - Get ride status
DELETE /api/rides/:id         - Cancel ride
GET    /api/rides/history     - Get ride history
POST   /api/rides/:id/rate    - Rate driver
```

**Driver Endpoints:**
```
PUT    /api/drivers/status    - Go online/offline
POST   /api/drivers/location  - Update location
GET    /api/rides/available   - Get available ride requests
POST   /api/rides/:id/accept  - Accept ride
POST   /api/rides/:id/start   - Start trip
POST   /api/rides/:id/complete - Complete trip
POST   /api/rides/:id/rate    - Rate rider
```

**Location:**
```
WebSocket /api/location/track  - Real-time location updates
GET    /api/drivers/nearby     - Get nearby drivers
```

**Payments:**
```
POST   /api/payments/methods  - Add payment method
GET    /api/payments/methods  - List payment methods
POST   /api/payments/process  - Process payment
```

**7. Potential Problems & Solutions:**

| Problem | Solution |
|---------|----------|
| **Multiple drivers accept same ride** | Use database transactions and locks; first to commit wins |
| **Driver location updates overwhelming system** | Batch updates, use Redis for temporary storage, update DB less frequently |
| **Matching algorithm slow** | Use geospatial indexing (PostGIS), search only nearby drivers |
| **Payment failures** | Retry mechanism, show clear error, allow alternative payment |
| **Network issues during ride** | Cache data locally, sync when connection restored |
| **Surge pricing complaints** | Clear communication, show price before confirmation |
| **Driver safety concerns** | SOS button, share trip with friends, driver verification |
| **Peak time scaling** | Auto-scaling for servers, prepare for 5x normal traffic |

**Additional Considerations:**
- **Maps**: Use Google Maps API for routing and ETA
- **Notifications**: Use FCM (Firebase) for push notifications
- **Analytics**: Track metrics (wait times, completion rates, earnings)
- **Fraud Detection**: Monitor for fake rides, suspicious patterns
- **Customer Support**: In-app help, live chat
- **Compliance**: Local taxi regulations, insurance

</details>

---

## Reflection Questions

After completing these exercises, reflect on:

1. **What was most challenging about gathering requirements?**
   - What questions did you forget to ask?

2. **How did you decide which components were necessary?**
   - Did you over-engineer or under-engineer?

3. **What trade-offs did you make in your designs?**
   - Would you make different choices for different scales?

4. **What real-world constraints affected your decisions?**
   - Cost, time, team size, existing infrastructure?

5. **How would you validate your design works?**
   - What tests or simulations would you run?

---

## Next Steps

Great job working through these exercises! You've practiced:
- ✅ Gathering requirements
- ✅ Identifying components
- ✅ Designing architectures
- ✅ Making trade-off decisions
- ✅ Thinking about scale

Continue to **[Trade-Offs](./trade-offs.md)** to learn more about making informed design decisions!
