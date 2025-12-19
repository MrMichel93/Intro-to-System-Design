# Case Study: Ride-Sharing App (Intermediate-Advanced)

## Problem Statement

Design a ride-sharing platform like Uber or Lyft that connects riders with drivers in real-time, handles location tracking, optimizes matching algorithms, processes payments, and scales to millions of rides per day across multiple cities globally.

## Requirements Analysis

### Functional Requirements

**Core Features:**
1. **Rider Features**:
   - Request a ride with pickup and destination
   - See nearby available drivers in real-time
   - Get fare estimate before booking
   - Track driver location during ride
   - Rate and review drivers
   - Payment processing

2. **Driver Features**:
   - Accept or decline ride requests
   - Navigate to pickup and destination
   - See rider rating before accepting
   - Track earnings
   - Go online/offline

3. **Matching System**:
   - Match riders with nearest available drivers
   - Optimize for pickup time and route efficiency
   - Handle concurrent requests
   - Support different vehicle types (economy, premium, XL)

4. **Pricing System**:
   - Dynamic pricing (surge pricing)
   - Fare calculation based on distance and time
   - Promotions and discounts
   - Split fare functionality

5. **Additional Features**:
   - Ride scheduling (advance booking)
   - Ride sharing (multiple passengers, different destinations)
   - Safety features (emergency contacts, trip sharing)
   - Driver background checks

### Non-Functional Requirements

**Scale Requirements:**
- **Users**: 100 million riders, 5 million drivers globally
- **Daily Active**: 10 million riders, 1 million drivers
- **Rides**: 20 million rides per day
  - ~230 rides/second average
  - ~700 rides/second peak (rush hours)
- **Location Updates**: 1 million concurrent active rides × 4 updates/min = 67K updates/sec
- **Cities**: 10,000 cities across 100 countries

**Performance Requirements:**
- **Matching Time**: < 5 seconds to find driver
- **Location Update Latency**: < 1 second
- **ETA Accuracy**: ±2 minutes
- **Map Rendering**: < 500ms
- **Availability**: 99.99% uptime (52 minutes downtime per year)
- **Payment Processing**: < 3 seconds

**Consistency & Reliability:**
- No double-booking (one driver, one ride at a time)
- Accurate location tracking (±10 meters)
- Reliable payment processing (no lost transactions)
- Fair matching (no favoritism)

## Capacity Estimation

### Storage Calculation

```
**Ride Data (per year):**
20M rides/day × 365 days = 7.3B rides/year

Per ride storage:
- Ride metadata: 1 KB (IDs, timestamps, status)
- Route data: 5 KB (GPS points, ~200 points × 25 bytes)
- Payment info: 500 bytes
- Total: ~6.5 KB per ride

Annual storage: 7.3B × 6.5 KB = 47.5 TB/year
5-year storage: 237 TB

**Location History:**
1M concurrent rides × 4 updates/min × 60 min = 240M updates/ride
20M rides/day × 240 updates = 4.8B location updates/day
4.8B × 25 bytes = 120 GB/day
Annual: 43.8 TB

**User Data:**
100M riders + 5M drivers = 105M users
105M × 2 KB per profile = 210 GB

**Total 5-year storage:** ~500 TB
```

### Traffic Estimation

```
**Location Updates:**
1M active rides × 4 updates/min = 4M updates/min = 67K/sec
Peak: 100K/sec

**Ride Requests:**
20M rides/day = 230 requests/sec average
Peak (rush hour): 700 requests/sec

**Matching Queries:**
For each request, query nearby drivers
700 requests/sec × 50 driver queries = 35K geospatial queries/sec

**API Requests:**
- Get nearby drivers: 100K/sec
- Update location: 100K/sec
- Fare estimates: 10K/sec
- Payment processing: 700/sec
- Total: ~250K API requests/sec
```

### Network Bandwidth

```
**Location Updates:**
100K updates/sec × 200 bytes = 20 MB/sec ingress

**Map Data:**
10M active users × 1 KB/sec map tiles = 10 GB/sec
Served mostly by CDN

**API Responses:**
250K requests/sec × 2 KB average = 500 MB/sec
```

## High-Level Architecture

```
┌────────────────────────────────────────────────────────────────────────┐
│                          CLIENT LAYER                                   │
│         [Rider Apps]              [Driver Apps]                        │
│    [iOS/Android/Web]          [iOS/Android]                           │
└────────────────────────────────┬───────────────────────────────────────┘
                                 │
┌────────────────────────────────▼───────────────────────────────────────┐
│                    API GATEWAY / LOAD BALANCER                          │
│    - Authentication, Rate limiting, Request routing                    │
│    - WebSocket connections for real-time updates                      │
└────────┬───────────────────────────────────────────────────────────────┘
         │
         ├─────────────────┬─────────────────┬──────────────────┬────────┐
         ▼                 ▼                 ▼                  ▼        │
┌────────────────┐ ┌────────────────┐ ┌────────────────┐ ┌─────────────▼──┐
│ LOCATION       │ │ MATCHING       │ │ PRICING        │ │ RIDE           │
│ SERVICE        │ │ SERVICE        │ │ SERVICE        │ │ SERVICE        │
│ - Track drivers│ │ - Find drivers │ │ - Calculate    │ │ - Manage rides │
│ - Update maps  │ │ - Assign rides │ │   fares        │ │ - State machine│
│ - Real-time    │ │ - Optimize     │ │ - Surge pricing│ │ - History      │
└────────┬───────┘ └────────┬───────┘ └────────┬───────┘ └────────┬───────┘
         │                  │                  │                  │
         ▼                  ▼                  ▼                  ▼
┌────────────────────────────────────────────────────────────────────────┐
│                    GEOSPATIAL DATABASE                                  │
│    Redis + Geohashing / Uber H3 / Google S2                           │
│    - Store driver locations with sub-second queries                   │
│    - Index: lat/long → driver_id                                      │
│    - TTL: 30 seconds (stale data removed)                             │
└────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│                    SUPPORTING SERVICES                                  │
├───────────────┬─────────────────┬─────────────────┬────────────────────┤
│ NOTIFICATION  │ PAYMENT         │ MAP SERVICE     │ ANALYTICS          │
│ - Push alerts │ - Payment       │ - Route calc    │ - Metrics          │
│ - SMS/Email   │   processing    │ - ETA estimates │ - ML models        │
│               │ - Stripe/Braintree│ - Google Maps │ - Surge prediction │
└───────────────┴─────────────────┴─────────────────┴────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│                    MESSAGE QUEUE (Kafka)                                │
│  [Location Updates] [Ride Events] [Payment Events] [Notifications]    │
└────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│                          DATA LAYER                                     │
├─────────────────┬─────────────────┬─────────────────┬──────────────────┤
│ RIDE DATA       │ USER DATA       │ PAYMENT DATA    │ CACHE            │
│ (Cassandra)     │ (PostgreSQL)    │ (PostgreSQL)    │ (Redis)          │
│ - Ride history  │ - User profiles │ - Transactions  │ - Driver status  │
│ - Route data    │ - Ratings       │ - Wallet        │ - Active rides   │
└─────────────────┴─────────────────┴─────────────────┴──────────────────┘
```

## Real-Time Location Tracking

### Location Update Flow

```python
class LocationService:
    def __init__(self):
        self.redis = Redis()
        self.kafka = KafkaProducer()
        self.geohash_precision = 6  # ~1.2 km precision
    
    async def update_driver_location(
        self,
        driver_id: str,
        lat: float,
        lon: float,
        heading: int,
        speed: float
    ):
        """Update driver location in real-time."""
        # 1. Calculate geohash for spatial indexing
        geohash = self.encode_geohash(lat, lon, self.geohash_precision)
        
        # 2. Store in Redis with TTL (auto-expire stale locations)
        location_key = f"driver:{driver_id}:location"
        await self.redis.geoadd(
            "drivers:active",
            lon, lat, driver_id
        )
        
        # 3. Store additional metadata
        await self.redis.hset(location_key, {
            "lat": lat,
            "lon": lon,
            "heading": heading,
            "speed": speed,
            "geohash": geohash,
            "timestamp": time.time()
        })
        await self.redis.expire(location_key, 30)  # 30 second TTL
        
        # 4. If driver is on active ride, update rider's view
        ride_id = await self.get_active_ride(driver_id)
        if ride_id:
            await self.push_location_to_rider(ride_id, lat, lon)
        
        # 5. Publish to Kafka for analytics/ML
        await self.kafka.produce("location_updates", {
            "driver_id": driver_id,
            "lat": lat,
            "lon": lon,
            "timestamp": time.time()
        })
    
    async def get_nearby_drivers(
        self,
        lat: float,
        lon: float,
        radius_km: float = 5.0,
        limit: int = 20
    ) -> list:
        """Find available drivers near a location."""
        # Use Redis GEORADIUS for efficient spatial query
        drivers = await self.redis.georadius(
            "drivers:active",
            lon, lat,
            radius_km,
            unit='km',
            withdist=True,
            sort='ASC',
            count=limit
        )
        
        # Filter only available drivers
        available = []
        for driver_id, distance in drivers:
            status = await self.redis.get(f"driver:{driver_id}:status")
            if status == "available":
                available.append({
                    "driver_id": driver_id,
                    "distance_km": distance
                })
        
        return available
```

### Geospatial Indexing Options

**Option 1: Redis Geospatial (Good for 1M+ drivers)**
```python
# Add driver location
GEOADD drivers:active {lon} {lat} {driver_id}

# Find nearby
GEORADIUS drivers:active {lon} {lat} 5 km WITHDIST COUNT 20

# Pros: Fast, simple, built-in
# Cons: Limited to Earth's surface, basic queries only
```

**Option 2: Uber H3 Hexagonal Hierarchical Spatial Index**
```python
import h3

# Convert lat/lon to H3 index (resolution 9 = ~175m cells)
h3_index = h3.geo_to_h3(lat, lon, resolution=9)

# Get neighboring cells
neighbors = h3.k_ring(h3_index, k=2)  # 2-ring = ~500m radius

# Query drivers in these cells
for cell in neighbors:
    drivers = redis.smembers(f"cell:{cell}:drivers")

# Pros: Uniform cell sizes, efficient nearest neighbor
# Cons: More complex, need to manage multiple cells
```

**Option 3: Google S2 Geometry (Used by Uber)**
```python
from s2sphere import CellId, LatLng

# Convert location to S2 cell
latlng = LatLng.from_degrees(lat, lon)
cell = CellId.from_lat_lng(latlng).parent(level=16)  # ~600m cells

# Get covering cells
covering = get_covering_cells(lat, lon, radius_km=5)

# Query all cells
drivers = []
for cell in covering:
    drivers.extend(redis.smembers(f"s2:{cell}:drivers"))

# Pros: Non-uniform cells (better for dense cities), precise
# Cons: Complex implementation
```

**Recommendation**: Redis Geospatial for MVP, migrate to S2/H3 for optimization at scale.

## Matching Algorithm

### Ride Matching Flow

```
┌─────────────────────────────────────────────────────────────┐
│ RIDER REQUESTS RIDE                                         │
│ - Pickup location: (37.7749, -122.4194)                    │
│ - Destination: (37.8044, -122.2712)                        │
│ - Vehicle type: Economy                                     │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│ STEP 1: FIND NEARBY AVAILABLE DRIVERS                      │
│ - Query drivers within 5 km radius                         │
│ - Filter by vehicle type                                   │
│ - Filter by availability status                            │
│ - Result: 50 candidate drivers                             │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│ STEP 2: CALCULATE SCORES FOR EACH DRIVER                   │
│ For each driver, compute:                                  │
│ - Distance to pickup (weight: 40%)                         │
│ - Estimated pickup time (weight: 30%)                      │
│ - Driver rating (weight: 15%)                              │
│ - Driver acceptance rate (weight: 10%)                     │
│ - Driver earnings (weight: 5%) - fairness                  │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│ STEP 3: RANK AND SELECT TOP DRIVERS                        │
│ - Sort by composite score                                  │
│ - Select top 5 drivers                                     │
│ - Check if drivers are truly available (race condition)    │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│ STEP 4: SEND REQUESTS IN SEQUENCE                          │
│ - Send to Driver #1 (30 second timeout)                   │
│ - If declined/timeout, send to Driver #2                  │
│ - Continue until match found or all decline                │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│ STEP 5: LOCK DRIVER AND CREATE RIDE                        │
│ - Use distributed lock (Redis) to claim driver            │
│ - Create ride record in database                           │
│ - Notify rider and driver                                  │
│ - Update driver status to "on_ride"                        │
└─────────────────────────────────────────────────────────────┘
```

### Matching Algorithm Implementation

```python
class MatchingService:
    def __init__(self):
        self.location_service = LocationService()
        self.map_service = MapService()
        self.redis = Redis()
    
    async def match_rider_with_driver(
        self,
        rider_id: str,
        pickup_lat: float,
        pickup_lon: float,
        vehicle_type: str
    ) -> dict:
        """Match rider with best available driver."""
        # 1. Find nearby drivers
        nearby = await self.location_service.get_nearby_drivers(
            pickup_lat, pickup_lon,
            radius_km=5.0,
            limit=50
        )
        
        if not nearby:
            raise NoDriversAvailableError("No drivers nearby")
        
        # 2. Score each driver
        scored_drivers = []
        for driver in nearby:
            score = await self.calculate_driver_score(
                driver,
                pickup_lat,
                pickup_lon
            )
            scored_drivers.append((score, driver))
        
        # 3. Sort by score (descending)
        scored_drivers.sort(reverse=True, key=lambda x: x[0])
        
        # 4. Try to assign in order
        for score, driver in scored_drivers[:5]:
            # Try to lock driver (prevent double-booking)
            locked = await self.try_lock_driver(driver['driver_id'])
            if locked:
                # Create ride
                ride = await self.create_ride(
                    rider_id,
                    driver['driver_id'],
                    pickup_lat,
                    pickup_lon
                )
                
                # Send request to driver
                accepted = await self.send_ride_request(
                    driver['driver_id'],
                    ride['ride_id'],
                    timeout=30
                )
                
                if accepted:
                    return ride
                else:
                    # Unlock and try next driver
                    await self.unlock_driver(driver['driver_id'])
        
        raise NoDriverAcceptedError("No driver accepted the ride")
    
    async def calculate_driver_score(
        self,
        driver: dict,
        pickup_lat: float,
        pickup_lon: float
    ) -> float:
        """Calculate composite score for driver selection."""
        # Get driver details
        driver_data = await self.get_driver_data(driver['driver_id'])
        
        # 1. Distance score (closer is better)
        distance_km = driver['distance_km']
        distance_score = max(0, 100 - (distance_km * 20))  # 0 at 5km
        
        # 2. ETA score (faster pickup is better)
        eta_minutes = await self.map_service.calculate_eta(
            driver['lat'], driver['lon'],
            pickup_lat, pickup_lon
        )
        eta_score = max(0, 100 - (eta_minutes * 10))  # 0 at 10 min
        
        # 3. Rating score
        rating_score = driver_data['rating'] * 20  # 5-star = 100
        
        # 4. Acceptance rate score
        acceptance_score = driver_data['acceptance_rate']
        
        # 5. Fairness score (prioritize lower earners slightly)
        avg_earnings = await self.get_average_earnings()
        if driver_data['daily_earnings'] < avg_earnings * 0.8:
            fairness_score = 110  # Slight boost
        else:
            fairness_score = 100
        
        # Weighted composite score
        total_score = (
            distance_score * 0.40 +
            eta_score * 0.30 +
            rating_score * 0.15 +
            acceptance_score * 0.10 +
            fairness_score * 0.05
        )
        
        return total_score
    
    async def try_lock_driver(self, driver_id: str) -> bool:
        """Attempt to lock driver (prevent double-booking)."""
        # Use Redis SET NX (set if not exists) for distributed lock
        lock_key = f"driver:{driver_id}:lock"
        locked = await self.redis.set(
            lock_key,
            "locked",
            nx=True,  # Only set if not exists
            ex=60     # Expire after 60 seconds
        )
        return locked
```

### Handling Race Conditions

```python
async def assign_ride_atomic(rider_id: str, driver_id: str):
    """Atomically assign ride to driver (prevent double-booking)."""
    # Use Redis transaction (MULTI/EXEC)
    pipe = redis.pipeline()
    
    # Check driver is available
    pipe.get(f"driver:{driver_id}:status")
    # Check driver isn't locked
    pipe.get(f"driver:{driver_id}:lock")
    
    # Execute check
    results = await pipe.execute()
    status, lock = results
    
    if status != "available" or lock is not None:
        return False  # Driver not available or already locked
    
    # Atomically update
    pipe = redis.pipeline()
    pipe.set(f"driver:{driver_id}:lock", "locked", ex=300)
    pipe.set(f"driver:{driver_id}:status", "assigned")
    pipe.set(f"driver:{driver_id}:ride", ride_id)
    await pipe.execute()
    
    return True
```

## Dynamic Pricing (Surge Pricing)

### Surge Pricing Algorithm

```python
class PricingService:
    async def calculate_fare(
        self,
        pickup_lat: float,
        pickup_lon: float,
        dest_lat: float,
        dest_lon: float,
        vehicle_type: str
    ) -> dict:
        """Calculate fare with surge pricing."""
        # 1. Calculate base fare
        distance_km = await self.calculate_distance(
            pickup_lat, pickup_lon,
            dest_lat, dest_lon
        )
        duration_min = await self.estimate_duration(
            pickup_lat, pickup_lon,
            dest_lat, dest_lon
        )
        
        base_fare = self.BASE_RATE[vehicle_type]
        distance_cost = distance_km * self.PER_KM_RATE[vehicle_type]
        time_cost = duration_min * self.PER_MIN_RATE[vehicle_type]
        
        fare = base_fare + distance_cost + time_cost
        
        # 2. Calculate surge multiplier
        surge = await self.calculate_surge_multiplier(
            pickup_lat,
            pickup_lon,
            vehicle_type
        )
        
        # 3. Apply surge
        final_fare = fare * surge
        
        return {
            "base_fare": fare,
            "surge_multiplier": surge,
            "final_fare": final_fare,
            "distance_km": distance_km,
            "duration_min": duration_min
        }
    
    async def calculate_surge_multiplier(
        self,
        lat: float,
        lon: float,
        vehicle_type: str
    ) -> float:
        """Calculate surge based on supply/demand."""
        # Get geohash cell for location
        cell = self.encode_geohash(lat, lon, precision=6)
        
        # Get supply (available drivers)
        supply = await self.redis.scard(f"cell:{cell}:available_drivers")
        
        # Get demand (pending ride requests)
        demand = await self.redis.scard(f"cell:{cell}:pending_rides")
        
        # Calculate ratio
        if supply == 0:
            return 3.0  # Max surge
        
        ratio = demand / supply
        
        # Apply surge curve
        if ratio < 0.5:
            return 1.0  # No surge
        elif ratio < 1.0:
            return 1.2  # Mild surge
        elif ratio < 2.0:
            return 1.5  # Moderate surge
        elif ratio < 3.0:
            return 2.0  # High surge
        else:
            return 3.0  # Maximum surge (3x)
    
    async def predict_surge(self, lat: float, lon: float):
        """Predict future surge using ML."""
        # Features for ML model
        features = {
            "hour": datetime.now().hour,
            "day_of_week": datetime.now().weekday(),
            "weather": await self.get_weather(lat, lon),
            "events": await self.get_nearby_events(lat, lon),
            "historical_demand": await self.get_historical_demand(lat, lon)
        }
        
        # Predict surge multiplier
        prediction = await self.ml_model.predict(features)
        return prediction
```

## API Design

### 1. Request Ride

```http
POST /api/v1/rides/request
Authorization: Bearer <token>
Content-Type: application/json

Request:
{
  "pickup": {
    "lat": 37.7749,
    "lon": -122.4194,
    "address": "123 Market St, San Francisco"
  },
  "destination": {
    "lat": 37.8044,
    "lon": -122.2712,
    "address": "Oakland Airport"
  },
  "vehicle_type": "economy",
  "payment_method": "card_123"
}

Response 201 Created:
{
  "ride_id": "ride_abc123",
  "status": "matching",
  "estimated_fare": {
    "min": 25.50,
    "max": 32.00,
    "surge_multiplier": 1.2
  },
  "estimated_duration": 25,  // minutes
  "estimated_distance": 18.5  // km
}
```

### 2. Update Driver Location (WebSocket)

```javascript
// Driver app sends location every 4 seconds
ws.send(JSON.stringify({
  "type": "location_update",
  "driver_id": "driver_xyz789",
  "lat": 37.7749,
  "lon": -122.4194,
  "heading": 45,
  "speed": 30,
  "timestamp": 1705320000
}));

// Server acknowledges
{
  "type": "ack",
  "timestamp": 1705320000
}

// If on active ride, rider gets updates
{
  "type": "driver_location",
  "ride_id": "ride_abc123",
  "driver": {
    "lat": 37.7749,
    "lon": -122.4194,
    "heading": 45,
    "eta_minutes": 3
  }
}
```

### 3. Get Nearby Drivers

```http
GET /api/v1/drivers/nearby?lat=37.7749&lon=-122.4194&radius=5
Authorization: Bearer <token>

Response 200 OK:
{
  "drivers": [
    {
      "driver_id": "driver_xyz789",
      "distance_km": 0.8,
      "eta_minutes": 3,
      "rating": 4.8,
      "vehicle": {
        "type": "economy",
        "make": "Toyota",
        "model": "Camry",
        "color": "Black",
        "plate": "ABC1234"
      }
    }
    // ... more drivers
  ],
  "count": 15
}
```

## Database Schema

### Rides Table (Cassandra)

```sql
CREATE TABLE rides (
    ride_id UUID PRIMARY KEY,
    rider_id UUID,
    driver_id UUID,
    pickup_location FROZEN<location>,  // {lat, lon, address}
    destination_location FROZEN<location>,
    requested_at TIMESTAMP,
    accepted_at TIMESTAMP,
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    status TEXT,  // requested, accepted, started, completed, cancelled
    fare DECIMAL,
    distance_km DECIMAL,
    duration_minutes INT,
    rating INT,
    vehicle_type TEXT
);

-- Index for rider's ride history
CREATE TABLE rides_by_rider (
    rider_id UUID,
    requested_at TIMESTAMP,
    ride_id UUID,
    PRIMARY KEY (rider_id, requested_at, ride_id)
) WITH CLUSTERING ORDER BY (requested_at DESC);

-- Index for driver's ride history
CREATE TABLE rides_by_driver (
    driver_id UUID,
    requested_at TIMESTAMP,
    ride_id UUID,
    PRIMARY KEY (driver_id, requested_at, ride_id)
) WITH CLUSTERING ORDER BY (requested_at DESC);
```

### Location History (Time-Series DB - InfluxDB)

```sql
-- Optimized for time-series data
MEASUREMENT location_updates
TAGS: driver_id, ride_id
FIELDS: lat, lon, speed, heading
TIMESTAMP: time

-- Query: Get driver's route for a ride
SELECT lat, lon, time
FROM location_updates
WHERE ride_id = 'ride_abc123'
  AND time >= '2024-01-15T10:00:00Z'
  AND time <= '2024-01-15T10:30:00Z'
ORDER BY time
```

### Users Table (PostgreSQL)

```sql
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20) UNIQUE NOT NULL,
    name VARCHAR(255),
    created_at TIMESTAMP DEFAULT NOW(),
    user_type VARCHAR(20),  -- rider, driver, both
    rating DECIMAL(3,2),
    total_rides INT DEFAULT 0,
    
    INDEX idx_email (email),
    INDEX idx_phone (phone)
);

CREATE TABLE drivers (
    driver_id UUID PRIMARY KEY REFERENCES users(user_id),
    license_number VARCHAR(50) UNIQUE,
    vehicle_id UUID,
    status VARCHAR(20),  -- online, offline, on_ride
    current_lat DECIMAL(10,8),
    current_lon DECIMAL(11,8),
    acceptance_rate DECIMAL(5,2),
    total_earnings DECIMAL(10,2),
    
    INDEX idx_status (status)
);
```

## Trade-Off Discussions

### 1. Matching: Greedy vs Optimal

**Greedy (Nearest Driver First):**
- ✅ Fast matching (< 1 second)
- ✅ Simple implementation
- ❌ Not globally optimal
- ❌ May leave some riders waiting longer

**Optimal (Global Assignment):**
- ✅ Minimizes total wait time
- ✅ Better utilization
- ❌ Slower (seconds to compute)
- ❌ Complex implementation

**Recommendation**: Greedy for real-time, run optimal offline for analytics and driver repositioning.

### 2. Location Updates: Push vs Pull

**Push (Clients send updates every 4 sec):**
- ✅ Real-time accuracy
- ✅ Server always has latest
- ❌ High network usage
- ❌ Battery drain on phones

**Pull (Server requests on demand):**
- ✅ Lower network/battery usage
- ❌ Potential staleness
- ❌ Not real-time

**Recommendation**: Push during active rides (4 sec), pull when idle (60 sec).

### 3. Payment: Pre-auth vs Post-ride

**Pre-authorization:**
- ✅ Guarantees payment
- ✅ Prevents fraud
- ❌ Temporary hold annoys users
- ❌ Complex with dynamic pricing

**Post-ride Charge:**
- ✅ Charge exact amount
- ✅ Better UX
- ❌ Risk of failed payment
- ❌ Need collections process

**Recommendation**: Pre-auth for new users/high fares, post-ride for trusted users.

### 4. Data Storage: SQL vs NoSQL

**SQL (PostgreSQL for users/payments):**
- ✅ ACID guarantees
- ✅ Complex queries
- ❌ Harder to scale horizontally

**NoSQL (Cassandra for rides/locations):**
- ✅ Horizontal scalability
- ✅ High write throughput
- ❌ Eventually consistent

**Recommendation**: Use both - SQL for critical data (payments), NoSQL for high-volume data (rides, locations).

## Scaling Strategies

### Phase 1: Single City (< 10K rides/day)

```
[Monolith App] → [PostgreSQL] → [Redis Cache]
                      ↓
                 [Stripe API]
```

- Single server
- Monolithic application
- Simple matching (nearest driver)
- Handles one city

### Phase 2: Multi-City (10K - 100K rides/day)

```
[API Gateway]
      ↓
[Microservices (Location, Matching, Ride, Payment)]
      ↓
[PostgreSQL Cluster] + [Redis Cluster]
      ↓
[Message Queue (Kafka)]
```

- Microservices architecture
- Database read replicas
- Redis for location data
- Handles 10 cities

### Phase 3: Regional (100K - 1M rides/day)

```
[Regional Load Balancer]
      ↓
[Microservices per Region (100+ instances)]
      ↓
[Cassandra (rides)] + [PostgreSQL (users)] + [Redis (locations)]
      ↓
[Kafka Cluster]
```

- Regional deployments
- Cassandra for ride data
- Geosharding (data partitioned by city)
- Handles 100 cities

### Phase 4: Global (1M+ rides/day)

```
[Global Load Balancer + CDN]
      ↓
[Multi-region Deployments (US, EU, Asia)]
      ↓
[1000+ Microservice Instances]
      ↓
[Cassandra (multi-DC)] + [PostgreSQL (sharded)] + [Redis (geo-distributed)]
      ↓
[Kafka (multi-region)] + [ML Pipeline]
```

- Multi-region architecture
- Advanced ML for matching/pricing
- Real-time analytics
- Handles 10,000+ cities globally

## Interview Talking Points

### Key Areas to Discuss

**1. Clarification Questions:**
- "Do we need to support ride scheduling or only immediate rides?"
- "What's the expected number of concurrent rides?"
- "Should we handle ride sharing (pool rides)?"
- "Do we need to support multiple stops?"

**2. Critical Bottlenecks:**
- "Location updates generate 100K/sec writes"
  - Solution: Redis for in-memory updates, batch to persistent storage
- "Matching needs sub-second geospatial queries"
  - Solution: Redis GEORADIUS or S2 indexing
- "Race conditions in driver assignment"
  - Solution: Distributed locks with Redis

**3. Optimizations:**
- "Pre-compute common routes during off-peak"
- "Use ML to predict demand and reposition drivers"
- "Cache fare estimates for popular routes"
- "Batch location updates every 4 seconds"

**4. Failure Scenarios:**
- "What if matching service fails?"
  - Fallback to simple nearest-driver algorithm
- "What if payment fails after ride?"
  - Queue for retry, allow rider to continue with liability
- "What if driver goes offline mid-ride?"
  - Reassign ride to nearby driver, compensate rider

**5. Advanced Features:**
- "How would you implement ride pooling?"
  - Match riders with similar routes, optimize pickup order
- "How would you detect fraud (fake GPS)?"
  - Validate location consistency, check speed limits
- "How would you implement driver earnings guarantees?"
  - Track hourly earnings, top up if below threshold

## Summary

Ride-sharing platform design demonstrates:

**Core Challenges:**
1. **Real-Time Location**: 100K updates/sec with <1 sec latency
2. **Efficient Matching**: Find best driver in <5 seconds
3. **Geospatial Queries**: Sub-second queries across millions of locations
4. **Race Conditions**: Prevent double-booking with distributed locks
5. **Dynamic Pricing**: Real-time surge calculation
6. **Global Scale**: 20M rides/day across 10K cities

**Key Solutions:**
- Redis GEORADIUS for spatial indexing
- WebSocket for real-time location updates
- Distributed locks for atomicity
- Hybrid matching algorithm (greedy + scoring)
- ML-based surge prediction
- Multi-region architecture for global scale

**Scale Achievements:**
- 100M users (10M daily active)
- 20M rides/day
- <5 second matching time
- <1 second location update latency
- 99.99% availability

This design mirrors real-world implementations from Uber, Lyft, and other ride-sharing platforms, proving that geospatial indexing and real-time processing are critical for location-based services at scale.
