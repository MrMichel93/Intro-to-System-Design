# Case Study: E-commerce Platform (Advanced)

## Problem Statement

Design a large-scale e-commerce platform like Amazon or eBay that supports millions of products, handles high-volume transactions, manages complex inventory across multiple warehouses, processes payments securely, and provides a seamless shopping experience for millions of concurrent users during peak events like Black Friday.

## Requirements Analysis

### Functional Requirements

**Customer Features:**
1. **Product Discovery**:
   - Search products by keywords, filters, categories
   - Browse recommendations
   - View product details, images, reviews
   - Compare products

2. **Shopping Experience**:
   - Add items to cart
   - Apply coupons and discounts
   - Save items for later (wishlist)
   - Guest checkout or account checkout

3. **Order Management**:
   - Place orders
   - Track order status and shipments
   - Order history
   - Returns and refunds
   - Cancel orders (before shipment)

4. **Reviews & Ratings**:
   - Write product reviews
   - Rate products
   - Upload photos with reviews
   - Mark reviews as helpful

**Seller Features:**
1. **Product Management**:
   - List new products
   - Update prices and inventory
   - Manage product variations (size, color)
   - Bulk operations

2. **Order Fulfillment**:
   - Receive and process orders
   - Update shipping status
   - Handle returns
   - View analytics and reports

**Platform Features:**
1. **Inventory Management**:
   - Track stock across multiple warehouses
   - Reserve inventory during checkout
   - Handle overselling prevention
   - Automatic reordering

2. **Payment Processing**:
   - Multiple payment methods
   - Secure payment gateway integration
   - Split payments
   - Fraud detection
   - Refund processing

3. **Shipping & Logistics**:
   - Multiple shipping options
   - Real-time shipping cost calculation
   - Integration with carriers (FedEx, UPS, USPS)
   - Track shipments

### Non-Functional Requirements

**Scale Requirements:**
- **Users**: 500 million registered users
  - 50 million daily active users (DAU)
  - 5 million concurrent users during peak
- **Products**: 100 million active listings
- **Orders**: 10 million orders per day
  - ~120 orders/second average
  - ~2,000 orders/second during peak sales (Black Friday)
- **Transactions**: $50 billion annual GMV (Gross Merchandise Value)

**Performance Requirements:**
- **Page Load Time**: < 2 seconds for product pages
- **Search Response**: < 100ms
- **Checkout Flow**: < 5 seconds total
- **Payment Processing**: < 3 seconds
- **Inventory Updates**: Real-time (<1 second)
- **Availability**: 99.99% uptime (52 minutes downtime per year)

**Data Consistency:**
- **Inventory**: Strong consistency (no overselling)
- **Payments**: ACID compliance (no lost money)
- **Product catalog**: Eventual consistency acceptable
- **User data**: Strong consistency

**Security:**
- PCI DSS compliance for payment processing
- Encryption at rest and in transit
- Fraud detection and prevention
- DDoS protection

## Capacity Estimation

### Storage Calculation

```
**Product Catalog:**
100M products × 5 KB metadata = 500 GB
100M products × 10 images × 200 KB = 200 TB (images)
Total: ~200 TB

**User Data:**
500M users × 2 KB profile = 1 TB
500M users × 20 orders avg × 2 KB = 20 TB (order history)
Total: ~21 TB

**Orders (cumulative 5 years):**
Year 1: 3.65B orders × 5 KB = 18.25 TB
Year 2: 3.65B orders × 5 KB = 18.25 TB (+ Year 1)
Year 3-5: Same pattern
Total 5 years: 3.65B × 5 years = 18.25B orders × 5 KB = 91.25 TB
(All orders retained for history, not replaced)

**Reviews:**
100M products × 50 reviews avg × 1 KB = 5 TB

**Transaction Logs:**
10M transactions/day × 2 KB = 20 GB/day
Annual: 7.3 TB

**Total Storage (5 years):** ~350 TB + 200 TB images = 550 TB
```

### Traffic Estimation

```
**Product Page Views:**
50M DAU × 20 pages/day = 1B page views/day
= 11,600 requests/second average
Peak: 50,000 requests/second

**Search Queries:**
50M DAU × 5 searches/day = 250M searches/day
= 2,900 searches/second average
Peak: 10,000 searches/second

**Add to Cart:**
10M orders/day × 1.5 items = 15M cart additions/day
= 174 cart operations/second

**Checkout:**
10M checkouts/day = 120/second average
Peak: 2,000/second

**API Calls:**
Total: ~100,000 API requests/second average
Peak: 300,000 requests/second
```

### Bandwidth Requirements

```
**Ingress:**
- Product images upload: 100 GB/day
- Order data: 60 GB/day
Total: ~160 GB/day

**Egress:**
- Product pages (HTML/JSON): 50K req/sec × 50 KB = 2.5 GB/sec
- Product images (CDN): 100K images/sec × 200 KB = 20 GB/sec
- Search results: 10K req/sec × 10 KB = 100 MB/sec
Total: ~23 GB/sec peak (mostly CDN for images)
```

## High-Level Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                          CLIENT LAYER                                 │
│  [Web Storefront] [Mobile Apps] [Third-party Apps] [APIs]           │
└────────────────────────────────┬─────────────────────────────────────┘
                                 │
┌────────────────────────────────▼─────────────────────────────────────┐
│                    CDN (Static Assets, Product Images)                │
│                    [CloudFront / Cloudflare]                          │
└────────────────────────────────┬─────────────────────────────────────┘
                                 │
┌────────────────────────────────▼─────────────────────────────────────┐
│                    API GATEWAY / LOAD BALANCER                        │
│              Authentication, Rate limiting, Request routing           │
└──────┬──────────────┬──────────────┬──────────────┬───────────────────┘
       │              │              │              │
       ▼              ▼              ▼              ▼
┌─────────────┐ ┌──────────┐ ┌──────────────┐ ┌─────────────┐
│  PRODUCT    │ │  CART    │ │    ORDER     │ │   PAYMENT   │
│  SERVICE    │ │  SERVICE │ │   SERVICE    │ │   SERVICE   │
│ - Catalog   │ │ - Add    │ │ - Checkout   │ │ - Process   │
│ - Search    │ │ - Update │ │ - Tracking   │ │ - Refunds   │
│ - Reviews   │ │ - Persist│ │ - History    │ │ - Fraud     │
└─────┬───────┘ └────┬─────┘ └──────┬───────┘ └──────┬──────┘
      │              │              │               │
      ▼              ▼              ▼               ▼
┌──────────────────────────────────────────────────────────────────────┐
│                    SUPPORTING SERVICES                                │
├─────────────┬─────────────┬──────────────┬──────────────┬────────────┤
│ INVENTORY   │ USER        │ NOTIFICATION │ SEARCH       │ ANALYTICS  │
│ - Stock mgmt│ - Profiles  │ - Email/SMS  │ - Elastic-   │ - Metrics  │
│ - Reserve   │ - Auth      │ - Push       │   search     │ - Reports  │
│ - Warehouses│ - Addresses │              │              │ - ML       │
└─────┬───────┴─────┬───────┴──────┬───────┴──────┬───────┴─────┬──────┘
      │             │              │              │             │
┌─────▼─────────────▼──────────────▼──────────────▼─────────────▼──────┐
│                      MESSAGE QUEUE (Kafka)                             │
│  [Order Events] [Payment Events] [Inventory Events] [Email Queue]    │
└────────────────────────────────┬───────────────────────────────────────┘
                                 │
┌────────────────────────────────▼───────────────────────────────────────┐
│                          DATA LAYER                                     │
├──────────────┬──────────────┬──────────────┬──────────────┬────────────┤
│ PRODUCT DB   │ ORDER DB     │ USER DB      │ INVENTORY DB │ CACHE      │
│ (Elasticsearch│ (PostgreSQL)│ (PostgreSQL) │ (PostgreSQL) │ (Redis)    │
│ + PostgreSQL)│ - ACID       │ - ACID       │ - ACID       │ - Sessions │
│              │ - Sharded    │              │ - Strong     │ - Cart     │
│              │              │              │   consistency│ - Products │
└──────────────┴──────────────┴──────────────┴──────────────┴────────────┘

┌───────────────────────────────────────────────────────────────────────┐
│                    STORAGE & CDN                                       │
│   [S3 - Product Images] → [CloudFront CDN]                           │
└───────────────────────────────────────────────────────────────────────┘
```

## Inventory Management System

### The Inventory Challenge

**Problem**: Multiple users trying to buy the last item in stock simultaneously.

**Requirements:**
- No overselling (sell only what's in stock)
- High throughput (thousands of checkouts/second)
- Low latency (<1 second to reserve)
- Handle distributed warehouses

### Solution 1: Optimistic Locking with Version Numbers

```python
class InventoryService:
    async def reserve_item(
        self,
        product_id: str,
        quantity: int,
        order_id: str
    ) -> bool:
        """Reserve inventory using optimistic locking."""
        max_retries = 3
        
        for attempt in range(max_retries):
            # 1. Read current inventory with version
            inventory = await self.db.query(
                """
                SELECT quantity, version, warehouse_id
                FROM inventory
                WHERE product_id = ?
                  AND quantity >= ?
                ORDER BY quantity DESC
                LIMIT 1
                """,
                product_id, quantity
            )
            
            if not inventory:
                return False  # Out of stock
            
            # 2. Try to update with version check (optimistic lock)
            updated = await self.db.execute(
                """
                UPDATE inventory
                SET quantity = quantity - ?,
                    version = version + 1,
                    updated_at = NOW()
                WHERE product_id = ?
                  AND warehouse_id = ?
                  AND version = ?
                  AND quantity >= ?
                """,
                quantity,
                product_id,
                inventory['warehouse_id'],
                inventory['version'],
                quantity
            )
            
            if updated:
                # Success! Create reservation record
                await self.create_reservation(
                    order_id,
                    product_id,
                    quantity,
                    inventory['warehouse_id']
                )
                return True
            
            # Version conflict - someone else updated, retry
            await asyncio.sleep(0.01 * (2 ** attempt))  # Exponential backoff
        
        return False  # Failed after retries
```

### Solution 2: Distributed Locking with Redis

```python
class InventoryServiceWithLocks:
    async def reserve_item_with_lock(
        self,
        product_id: str,
        quantity: int,
        order_id: str
    ) -> bool:
        """Reserve inventory using distributed locks."""
        lock_key = f"inventory:lock:{product_id}"
        lock_id = str(uuid.uuid4())
        
        # 1. Acquire distributed lock
        locked = await self.redis.set(
            lock_key,
            lock_id,
            nx=True,  # Only set if not exists
            ex=10     # Expire after 10 seconds
        )
        
        if not locked:
            return False  # Lock held by another process
        
        try:
            # 2. Check and update inventory (now protected by lock)
            inventory = await self.db.query(
                """
                SELECT quantity, warehouse_id
                FROM inventory
                WHERE product_id = ?
                  AND quantity >= ?
                LIMIT 1
                """,
                product_id, quantity
            )
            
            if not inventory:
                return False
            
            # 3. Update inventory
            await self.db.execute(
                """
                UPDATE inventory
                SET quantity = quantity - ?
                WHERE product_id = ?
                  AND warehouse_id = ?
                """,
                quantity,
                product_id,
                inventory['warehouse_id']
            )
            
            # 4. Create reservation
            await self.create_reservation(
                order_id,
                product_id,
                quantity,
                inventory['warehouse_id']
            )
            
            return True
            
        finally:
            # 5. Release lock
            await self.release_lock(lock_key, lock_id)
    
    async def release_lock(self, lock_key: str, lock_id: str):
        """Release lock only if we own it (Lua script for atomicity)."""
        lua_script = """
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
        """
        await self.redis.eval(lua_script, 1, lock_key, lock_id)
```

### Solution 3: Reservation System with TTL

```python
class ReservationBasedInventory:
    RESERVATION_TTL = 600  # 10 minutes to complete checkout
    
    async def reserve_with_ttl(
        self,
        product_id: str,
        quantity: int,
        order_id: str
    ) -> bool:
        """Reserve inventory with automatic expiration."""
        # 1. Create temporary reservation
        reservation_id = str(uuid.uuid4())
        
        # 2. Atomic decrement in Redis
        available = await self.redis.decrby(
            f"inventory:{product_id}:available",
            quantity
        )
        
        if available < 0:
            # Rollback - not enough inventory
            await self.redis.incrby(
                f"inventory:{product_id}:available",
                quantity
            )
            return False
        
        # 3. Store reservation with TTL
        await self.redis.setex(
            f"reservation:{reservation_id}",
            self.RESERVATION_TTL,
            json.dumps({
                "product_id": product_id,
                "quantity": quantity,
                "order_id": order_id,
                "expires_at": time.time() + self.RESERVATION_TTL
            })
        )
        
        # 4. Background job releases expired reservations
        return True
    
    async def confirm_reservation(self, order_id: str):
        """Confirm reservation after successful payment."""
        reservation = await self.get_reservation(order_id)
        
        # Update persistent inventory
        await self.db.execute(
            """
            UPDATE inventory
            SET quantity = quantity - ?,
                reserved = reserved + ?
            WHERE product_id = ?
            """,
            reservation['quantity'],
            reservation['quantity'],
            reservation['product_id']
        )
        
        # Delete reservation
        await self.redis.delete(f"reservation:{reservation['id']}")
    
    async def release_expired_reservations(self):
        """Background job to release expired reservations."""
        expired = await self.get_expired_reservations()
        
        for reservation in expired:
            # Return inventory to available pool
            await self.redis.incrby(
                f"inventory:{reservation['product_id']}:available",
                reservation['quantity']
            )
            
            # Delete reservation
            await self.redis.delete(f"reservation:{reservation['id']}")
```

### Multi-Warehouse Inventory

```python
class MultiWarehouseInventory:
    async def find_optimal_warehouse(
        self,
        product_id: str,
        quantity: int,
        shipping_address: dict
    ) -> str:
        """Select warehouse that minimizes shipping cost/time."""
        # 1. Get all warehouses with sufficient stock
        warehouses = await self.db.query(
            """
            SELECT warehouse_id, quantity, lat, lon
            FROM inventory
            WHERE product_id = ?
              AND quantity >= ?
            """,
            product_id, quantity
        )
        
        if not warehouses:
            return None
        
        # 2. Calculate shipping cost/time for each warehouse
        scored = []
        for wh in warehouses:
            distance = self.calculate_distance(
                wh['lat'], wh['lon'],
                shipping_address['lat'], shipping_address['lon']
            )
            
            shipping_time = self.estimate_shipping_time(distance)
            shipping_cost = self.estimate_shipping_cost(distance, quantity)
            
            # Composite score (prioritize time over cost)
            score = shipping_time * 0.7 + (shipping_cost / 10) * 0.3
            
            scored.append((score, wh['warehouse_id']))
        
        # 3. Select warehouse with best score
        scored.sort()
        return scored[0][1]
```

## Payment Processing System

### Two-Phase Commit for Payment

```python
class PaymentService:
    async def process_payment(
        self,
        order_id: str,
        amount: Decimal,
        payment_method: str
    ) -> dict:
        """Process payment with two-phase commit."""
        transaction_id = str(uuid.uuid4())
        
        try:
            # PHASE 1: PREPARE
            # 1a. Authorize payment (hold funds)
            auth_result = await self.payment_gateway.authorize(
                amount=amount,
                payment_method=payment_method,
                transaction_id=transaction_id
            )
            
            if not auth_result['success']:
                raise PaymentAuthorizationError(auth_result['error'])
            
            # 1b. Reserve inventory (already done in cart)
            # 1c. Create order record (pending state)
            await self.create_order(order_id, 'pending', transaction_id)
            
            # PHASE 2: COMMIT
            # 2a. Capture payment (actually charge)
            capture_result = await self.payment_gateway.capture(
                transaction_id=transaction_id,
                amount=amount
            )
            
            if not capture_result['success']:
                # Capture failed - release authorization
                await self.payment_gateway.void(transaction_id)
                raise PaymentCaptureError(capture_result['error'])
            
            # 2b. Confirm inventory reservation
            await self.inventory.confirm_reservation(order_id)
            
            # 2c. Update order status to confirmed
            await self.update_order_status(order_id, 'confirmed')
            
            # 2d. Publish order confirmed event
            await self.kafka.produce('order_confirmed', {
                'order_id': order_id,
                'transaction_id': transaction_id,
                'amount': float(amount)
            })
            
            return {
                'success': True,
                'transaction_id': transaction_id,
                'order_id': order_id
            }
            
        except Exception as e:
            # ROLLBACK: Compensating transactions
            await self.rollback_payment(transaction_id, order_id)
            raise
    
    async def rollback_payment(self, transaction_id: str, order_id: str):
        """Compensating transaction to rollback failed payment."""
        # 1. Void/refund payment
        try:
            await self.payment_gateway.void(transaction_id)
        except:
            # If void fails, issue refund
            await self.payment_gateway.refund(transaction_id)
        
        # 2. Release inventory reservation
        await self.inventory.release_reservation(order_id)
        
        # 3. Mark order as failed
        await self.update_order_status(order_id, 'failed')
        
        # 4. Notify user
        await self.notification.send(
            order_id,
            "Payment failed. Please try again."
        )
```

### Fraud Detection

```python
class FraudDetectionService:
    async def check_order_fraud(self, order: dict) -> dict:
        """Multi-layer fraud detection."""
        risk_score = 0
        flags = []
        
        # 1. Velocity checks
        recent_orders = await self.get_recent_orders(
            order['user_id'],
            minutes=60
        )
        
        if len(recent_orders) > 5:
            risk_score += 30
            flags.append('high_velocity')
        
        # 2. Payment method checks
        if order['payment_method'] == 'new_card':
            risk_score += 15
            flags.append('new_payment_method')
        
        # 3. Shipping address checks
        if order['shipping_address'] != order['billing_address']:
            risk_score += 10
            flags.append('address_mismatch')
        
        # 4. Order value checks
        avg_order_value = await self.get_avg_order_value(order['user_id'])
        if order['total'] > avg_order_value * 3:
            risk_score += 20
            flags.append('unusually_high_value')
        
        # 5. Geographic checks
        user_location = await self.get_user_location(order['user_id'])
        order_location = order['shipping_address']['country']
        if user_location != order_location:
            risk_score += 15
            flags.append('international_order')
        
        # 6. ML model prediction
        ml_score = await self.ml_model.predict_fraud(order)
        risk_score += ml_score * 50
        
        # Determine action
        if risk_score < 30:
            action = 'approve'
        elif risk_score < 60:
            action = 'review'
        else:
            action = 'decline'
        
        return {
            'risk_score': risk_score,
            'action': action,
            'flags': flags
        }
```

## API Design

### 1. Search Products

```http
GET /api/v1/products/search?q=laptop&category=electronics&min_price=500&max_price=2000&page=1&limit=20
Authorization: Bearer <token>

Response 200 OK:
{
  "results": [
    {
      "product_id": "prod_123",
      "name": "Dell XPS 13 Laptop",
      "price": 1299.99,
      "currency": "USD",
      "rating": 4.5,
      "review_count": 1234,
      "thumbnail": "https://cdn.example.com/prod_123_thumb.jpg",
      "in_stock": true,
      "seller": {
        "seller_id": "seller_456",
        "name": "Dell Official Store",
        "rating": 4.8
      }
    }
    // ... more products
  ],
  "total_results": 1523,
  "page": 1,
  "total_pages": 77,
  "facets": {
    "brands": ["Dell", "HP", "Lenovo", "Apple"],
    "price_ranges": ["$500-$1000", "$1000-$1500", "$1500+"]
  }
}
```

### 2. Add to Cart

```http
POST /api/v1/cart/items
Authorization: Bearer <token>
Content-Type: application/json

Request:
{
  "product_id": "prod_123",
  "quantity": 1,
  "options": {
    "color": "silver",
    "storage": "512GB"
  }
}

Response 200 OK:
{
  "cart_id": "cart_abc123",
  "items": [
    {
      "product_id": "prod_123",
      "name": "Dell XPS 13 Laptop",
      "quantity": 1,
      "price": 1299.99,
      "subtotal": 1299.99
    }
  ],
  "total": 1299.99,
  "item_count": 1
}
```

### 3. Checkout

```http
POST /api/v1/orders/checkout
Authorization: Bearer <token>
Content-Type: application/json

Request:
{
  "cart_id": "cart_abc123",
  "shipping_address": {
    "name": "John Doe",
    "street": "123 Main St",
    "city": "San Francisco",
    "state": "CA",
    "zip": "94102",
    "country": "US"
  },
  "payment_method": {
    "type": "card",
    "card_id": "card_xyz789"
  },
  "shipping_method": "standard"
}

Response 201 Created:
{
  "order_id": "order_def456",
  "status": "confirmed",
  "total": 1299.99,
  "tax": 104.00,
  "shipping": 9.99,
  "grand_total": 1413.98,
  "estimated_delivery": "2024-01-20",
  "tracking_url": "https://track.example.com/order_def456"
}
```

### 4. Track Order

```http
GET /api/v1/orders/{order_id}
Authorization: Bearer <token>

Response 200 OK:
{
  "order_id": "order_def456",
  "status": "shipped",
  "placed_at": "2024-01-15T10:30:00Z",
  "shipped_at": "2024-01-16T14:00:00Z",
  "estimated_delivery": "2024-01-20",
  "items": [
    {
      "product_id": "prod_123",
      "name": "Dell XPS 13 Laptop",
      "quantity": 1,
      "price": 1299.99
    }
  ],
  "shipment": {
    "carrier": "FedEx",
    "tracking_number": "123456789",
    "current_location": "Oakland, CA",
    "tracking_url": "https://fedex.com/track/123456789"
  },
  "timeline": [
    {"status": "placed", "timestamp": "2024-01-15T10:30:00Z"},
    {"status": "confirmed", "timestamp": "2024-01-15T10:31:00Z"},
    {"status": "processing", "timestamp": "2024-01-15T15:00:00Z"},
    {"status": "shipped", "timestamp": "2024-01-16T14:00:00Z"}
  ]
}
```

## Database Schema

### Products Table (PostgreSQL)

```sql
CREATE TABLE products (
    product_id UUID PRIMARY KEY,
    seller_id UUID NOT NULL,
    name VARCHAR(500) NOT NULL,
    description TEXT,
    category_id INT,
    price DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    status VARCHAR(20) DEFAULT 'active',
    
    INDEX idx_seller (seller_id),
    INDEX idx_category (category_id),
    INDEX idx_price (price),
    FULLTEXT INDEX idx_search (name, description)
);

CREATE TABLE product_variants (
    variant_id UUID PRIMARY KEY,
    product_id UUID REFERENCES products(product_id),
    sku VARCHAR(100) UNIQUE,
    attributes JSONB,  -- {color: "silver", size: "M"}
    price_adjustment DECIMAL(10,2) DEFAULT 0,
    
    INDEX idx_product (product_id),
    INDEX idx_sku (sku)
);
```

### Inventory Table (PostgreSQL - Strong Consistency)

```sql
CREATE TABLE inventory (
    inventory_id UUID PRIMARY KEY,
    product_id UUID NOT NULL,
    variant_id UUID,
    warehouse_id UUID NOT NULL,
    quantity INT NOT NULL CHECK (quantity >= 0),
    reserved INT DEFAULT 0,
    version INT DEFAULT 0,  -- For optimistic locking
    updated_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE (product_id, variant_id, warehouse_id),
    INDEX idx_product (product_id),
    INDEX idx_warehouse (warehouse_id)
);

CREATE TABLE inventory_reservations (
    reservation_id UUID PRIMARY KEY,
    order_id UUID NOT NULL,
    product_id UUID NOT NULL,
    variant_id UUID,
    quantity INT NOT NULL,
    warehouse_id UUID NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    status VARCHAR(20) DEFAULT 'active',
    
    INDEX idx_order (order_id),
    INDEX idx_expires (expires_at),
    INDEX idx_status (status)
);
```

### Orders Table (PostgreSQL - ACID)

```sql
CREATE TABLE orders (
    order_id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    status VARCHAR(50) NOT NULL,
    subtotal DECIMAL(10,2) NOT NULL,
    tax DECIMAL(10,2) DEFAULT 0,
    shipping DECIMAL(10,2) DEFAULT 0,
    discount DECIMAL(10,2) DEFAULT 0,
    total DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    payment_method VARCHAR(50),
    shipping_address JSONB,
    billing_address JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    INDEX idx_user (user_id),
    INDEX idx_status (status),
    INDEX idx_created (created_at)
);

CREATE TABLE order_items (
    item_id UUID PRIMARY KEY,
    order_id UUID REFERENCES orders(order_id),
    product_id UUID NOT NULL,
    variant_id UUID,
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    subtotal DECIMAL(10,2) NOT NULL,
    warehouse_id UUID,
    
    INDEX idx_order (order_id),
    INDEX idx_product (product_id)
);
```

### Payments Table (PostgreSQL - ACID & PCI Compliant)

```sql
CREATE TABLE payments (
    payment_id UUID PRIMARY KEY,
    order_id UUID REFERENCES orders(order_id),
    transaction_id VARCHAR(255) UNIQUE,
    amount DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    status VARCHAR(50) NOT NULL,
    payment_method VARCHAR(50),
    gateway VARCHAR(50),  -- stripe, braintree, etc.
    gateway_response JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    
    INDEX idx_order (order_id),
    INDEX idx_transaction (transaction_id),
    INDEX idx_status (status)
);

-- Never store raw card data - use tokenization
CREATE TABLE payment_methods (
    method_id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    type VARCHAR(20),  -- card, paypal, etc.
    token VARCHAR(255) NOT NULL,  -- Gateway token (Stripe, Braintree)
    last_four VARCHAR(4),
    expiry_month INT,
    expiry_year INT,
    is_default BOOLEAN DEFAULT FALSE,
    
    INDEX idx_user (user_id)
);
```

## Trade-Off Discussions

### 1. Inventory Consistency: Strong vs Eventual

**Strong Consistency (PostgreSQL with locks):**
- ✅ Never oversell
- ✅ Accurate stock counts
- ❌ Slower checkout
- ❌ Single point of contention

**Eventual Consistency (Distributed with compensation):**
- ✅ Fast checkout
- ✅ High throughput
- ❌ May occasionally oversell
- ❌ Need compensation logic

**Recommendation**: Strong consistency for inventory (acceptable slower checkout), eventual for product catalog.

### 2. Search: Database vs Elasticsearch

**Database Full-Text Search:**
- ✅ Simple, no extra service
- ✅ Always consistent
- ❌ Slower for complex queries
- ❌ Limited features

**Elasticsearch:**
- ✅ Fast, powerful search
- ✅ Facets, relevance ranking
- ❌ Eventually consistent
- ❌ Extra complexity

**Recommendation**: Elasticsearch for product search, database as source of truth.

### 3. Payment: Synchronous vs Asynchronous

**Synchronous (Block until payment completes):**
- ✅ Immediate confirmation
- ✅ Simpler logic
- ❌ Slower checkout
- ❌ User waits for payment gateway

**Asynchronous (Process payment in background):**
- ✅ Fast checkout
- ✅ Better UX
- ❌ Complex state management
- ❌ Need webhooks

**Recommendation**: Synchronous for credit cards (3 sec is acceptable), async for bank transfers.

### 4. Cart Storage: Session vs Database

**Session/Cookie:**
- ✅ No database writes
- ✅ Very fast
- ❌ Lost if cookies cleared
- ❌ Not shareable across devices

**Database:**
- ✅ Persistent
- ✅ Multi-device
- ❌ Extra database load
- ❌ Need cleanup for abandoned carts

**Recommendation**: Redis cache (30-day TTL) + periodic sync to database for logged-in users.

## Scaling Strategies

### Phase 1: Startup (< 10K orders/day)

```
[Monolith App] → [PostgreSQL] → [Redis Cache] → [Stripe API]
                      ↓
                 [S3 + CloudFront]
```

- Monolithic application
- Single PostgreSQL database
- Simple payment integration
- Handles small catalog

### Phase 2: Growth (10K - 100K orders/day)

```
[Load Balancer]
      ↓
[Microservices (Product, Cart, Order, Payment)]
      ↓
[PostgreSQL Primary + Read Replicas] + [Redis Cluster]
      ↓
[Elasticsearch for Search]
```

- Separate services
- Database read replicas
- Elasticsearch for search
- Handles medium catalog

### Phase 3: Large Scale (100K - 1M orders/day)

```
[API Gateway + CDN]
      ↓
[Microservices (50+ instances)]
      ↓
[PostgreSQL Sharded] + [Redis Cluster] + [Elasticsearch Cluster]
      ↓
[Kafka Event Stream]
```

- Database sharding (by user_id)
- Event-driven architecture
- Multiple data centers
- Handles large catalog

### Phase 4: Massive Scale (1M+ orders/day)

```
[Global CDN + Edge Computing]
      ↓
[Multi-region API Gateways]
      ↓
[1000+ Microservice Instances]
      ↓
[Sharded Databases per Region]
      ↓
[Multi-region Kafka + Data Replication]
```

- Multi-region deployment
- Regional data residency
- Advanced ML recommendations
- Handles massive catalog (100M+ products)

## Interview Talking Points

### Key Discussion Areas

**1. Critical Questions:**
- "How do we prevent overselling during flash sales?"
  - Strong consistency with optimistic locking
- "How do we handle payment failures?"
  - Two-phase commit with compensation
- "How do we scale inventory across warehouses?"
  - Select optimal warehouse per order

**2. System Bottlenecks:**
- "Inventory updates are the main bottleneck"
  - Solution: Optimistic locking + Redis for reservation
- "Payment processing can timeout"
  - Solution: Async processing with status polling
- "Search becomes slow with large catalog"
  - Solution: Elasticsearch with proper indexing

**3. Complex Scenarios:**
- "What if payment succeeds but order creation fails?"
  - Use distributed transactions (saga pattern)
- "What if user adds item to cart but it sells out?"
  - Reservation system with TTL
- "How to handle returns and refunds?"
  - Reverse transactions, return inventory to pool

**4. Advanced Features:**
- "How would you implement real-time price updates?"
  - WebSocket connections + cache invalidation
- "How would you detect fraudulent orders?"
  - ML model + rule-based checks + manual review queue
- "How would you handle international orders?"
  - Multi-currency, tax calculation per country, regional warehouses

## Summary

E-commerce platform design demonstrates:

**Core Challenges:**
1. **Inventory Consistency**: Prevent overselling at scale
2. **Payment Security**: PCI compliance, fraud detection
3. **High Availability**: 99.99% uptime during peak sales
4. **Complex Transactions**: Multi-step checkout with rollback
5. **Search Performance**: Fast search across 100M+ products

**Key Solutions:**
- Strong consistency for inventory (PostgreSQL + optimistic locking)
- Two-phase commit for payments
- Elasticsearch for product search
- Redis for cart and session management
- Event-driven architecture (Kafka) for async processing
- Multi-warehouse optimization

**Scale Achievements:**
- 10M orders/day
- 2,000 orders/second during peak
- 100M products in catalog
- 50M daily active users
- 99.99% availability
- <5 second checkout

This design mirrors real-world implementations from Amazon, eBay, Shopify, proving that careful attention to consistency, transactions, and inventory management are critical for e-commerce at scale.
