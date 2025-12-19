# Design Exercise: E-Commerce Database Schema

## 1. Design Problem

### Problem Statement
Design a relational database schema for a full-featured e-commerce platform like Amazon or eBay. The system must efficiently handle products, users, orders, inventory, payments, reviews, and support complex queries while maintaining data integrity and performance at scale.

### Context and Constraints
- Multiple product categories with different attributes (clothing has sizes, electronics have specs)
- Products can have multiple images, variants (color, size), and pricing tiers
- Users can be buyers, sellers, or both
- Shopping cart must persist across sessions
- Orders go through multiple states (pending, paid, shipped, delivered, returned)
- Inventory tracking across multiple warehouses
- Payment information must be secure and PCI compliant
- Product search and filtering must be fast
- Reviews and ratings affect product rankings
- Promotional codes and discounts

### Requirements

#### Functional Requirements
- User management (registration, profiles, addresses, payment methods)
- Product catalog (categories, products, variants, pricing)
- Shopping cart (add/remove items, persist cart)
- Order management (checkout, payment, order tracking)
- Inventory management (stock levels, warehouse locations)
- Review and rating system
- Seller management (vendor accounts, products, sales analytics)
- Promotional codes and discount system
- Order history and reporting
- Search and filtering (by category, price, ratings, etc.)

#### Non-Functional Requirements
- **Scale**: 10 million users, 1 million products, 100K orders per day
- **Performance**: Product page load < 200ms, search < 500ms, checkout < 2 seconds
- **Consistency**: Strong consistency for inventory and payments (no overselling, no double charging)
- **Availability**: 99.9% uptime
- **Data Integrity**: ACID transactions for critical operations
- **Security**: Encrypted sensitive data, PCI DSS compliance for payments

## 2. Guided Questions

### Understanding Database Requirements
1. **What entities (tables) do we need?**
   - Hint: Think about nouns in the requirements
   - Consider: Users, products, orders, payments, reviews, etc.

2. **What are the relationships between entities?**
   - Hint: One-to-one, one-to-many, or many-to-many?
   - Consider: A user can have many orders, an order can have many products

3. **Which fields need to be indexed?**
   - Hint: What will users search or filter by?
   - Consider: Product name, category, price, user email

### Schema Design Decisions
4. **Should product attributes be in the products table or separate?**
   - Hint: Different products have different attributes
   - Consider: Electronics need "processor", clothing needs "size"

5. **How do we handle product variants (size, color)?**
   - Hint: Same product, different SKU
   - Consider: T-shirt in red/blue, sizes S/M/L

6. **How do we prevent overselling (inventory management)?**
   - Hint: Multiple users might order last item simultaneously
   - Consider: Database transactions, locks

### Performance Considerations
7. **How do we make product search fast?**
   - Hint: Millions of products to search through
   - Consider: Full-text search, separate search engine, denormalization

8. **Should we normalize or denormalize order data?**
   - Hint: Product price might change after order is placed
   - Consider: Historical data, query performance

## 3. Step-by-Step Guidance

### Step 1: Identify Core Entities
Start with main business objects:

**Core Entities:**
- Users (buyers and sellers)
- Products (items for sale)
- Categories (product organization)
- Orders (purchase records)
- OrderItems (products in an order)
- Cart (temporary shopping basket)
- Reviews (product feedback)
- Payments (transaction records)
- Addresses (shipping/billing)
- Inventory (stock tracking)

### Step 2: Define Relationships
Determine how entities connect:

**Key Relationships:**
- User → Orders (one-to-many: user can have multiple orders)
- Order → OrderItems (one-to-many: order contains multiple products)
- Product → OrderItems (one-to-many: product appears in multiple orders)
- User → Reviews (one-to-many: user can write multiple reviews)
- Product → Reviews (one-to-many: product can have multiple reviews)
- Product → Category (many-to-one: products belong to categories)
- Product → ProductVariants (one-to-many: product has multiple variants)
- User → Addresses (one-to-many: user can have multiple addresses)
- Order → Payment (one-to-one: order has one payment)

### Step 3: Design Schema with Normalization
Start with normalized design (reduce redundancy):

```sql
-- Users table
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    phone VARCHAR(20),
    is_seller BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Categories table (hierarchical)
CREATE TABLE categories (
    category_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    parent_category_id INTEGER REFERENCES categories(category_id),
    slug VARCHAR(100) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Products table
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    seller_id INTEGER REFERENCES users(user_id),
    category_id INTEGER REFERENCES categories(category_id),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    base_price DECIMAL(10, 2) NOT NULL,
    brand VARCHAR(100),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Product variants (color, size, etc.)
CREATE TABLE product_variants (
    variant_id SERIAL PRIMARY KEY,
    product_id INTEGER REFERENCES products(product_id),
    sku VARCHAR(100) UNIQUE NOT NULL,
    name VARCHAR(100), -- "Red, Size M"
    price_adjustment DECIMAL(10, 2) DEFAULT 0,
    attributes JSONB, -- {"color": "red", "size": "M"}
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Inventory table
CREATE TABLE inventory (
    inventory_id SERIAL PRIMARY KEY,
    variant_id INTEGER REFERENCES product_variants(variant_id),
    warehouse_id INTEGER REFERENCES warehouses(warehouse_id),
    quantity INTEGER DEFAULT 0,
    reserved_quantity INTEGER DEFAULT 0,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Shopping cart
CREATE TABLE cart_items (
    cart_item_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id),
    variant_id INTEGER REFERENCES product_variants(variant_id),
    quantity INTEGER DEFAULT 1,
    added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Orders table
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id),
    status VARCHAR(50), -- pending, paid, shipped, delivered, cancelled, returned
    total_amount DECIMAL(10, 2) NOT NULL,
    discount_amount DECIMAL(10, 2) DEFAULT 0,
    shipping_address_id INTEGER REFERENCES addresses(address_id),
    billing_address_id INTEGER REFERENCES addresses(address_id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Order items (denormalized for historical data)
CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(order_id),
    product_id INTEGER REFERENCES products(product_id),
    variant_id INTEGER REFERENCES product_variants(variant_id),
    product_name VARCHAR(255), -- Snapshot at time of order
    variant_name VARCHAR(100),
    price DECIMAL(10, 2), -- Price at time of purchase
    quantity INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Step 4: Handle Product Variants
Two approaches for product variations:

**Approach 1: Variant Table** (Recommended)
```sql
-- Products: Base product info
-- ProductVariants: Specific SKUs (color, size combinations)
-- Inventory: Stock for each variant

Product: "Classic T-Shirt"
  Variant 1: "Red, Size S" (SKU: TSHIRT-RED-S)
  Variant 2: "Red, Size M" (SKU: TSHIRT-RED-M)
  Variant 3: "Blue, Size S" (SKU: TSHIRT-BLUE-S)
```

**Approach 2: EAV (Entity-Attribute-Value)**
```sql
CREATE TABLE product_attributes (
    product_id INTEGER,
    attribute_name VARCHAR(50),
    attribute_value VARCHAR(255)
);

-- Flexible but harder to query
```

### Step 5: Prevent Inventory Overselling
Use database transactions and row locking:

```sql
-- Reserve inventory when adding to cart (pessimistic locking)
BEGIN TRANSACTION;

-- Lock the inventory row
SELECT quantity, reserved_quantity
FROM inventory
WHERE variant_id = 123
FOR UPDATE;

-- Check if available
IF quantity - reserved_quantity >= requested_quantity THEN
    UPDATE inventory
    SET reserved_quantity = reserved_quantity + requested_quantity
    WHERE variant_id = 123;
    
    -- Add to cart
    INSERT INTO cart_items (user_id, variant_id, quantity)
    VALUES (456, 123, requested_quantity);
    
    COMMIT;
ELSE
    ROLLBACK;
    RAISE EXCEPTION 'Insufficient inventory';
END IF;
```

### Step 6: Optimize for Search
Product search requires special handling:

**Option 1: Database Full-Text Search**
```sql
-- Add full-text search index
CREATE INDEX idx_product_search ON products 
USING GIN(to_tsvector('english', name || ' ' || description));

-- Search query
SELECT * FROM products
WHERE to_tsvector('english', name || ' ' || description) @@ 
      to_tsquery('english', 'laptop & gaming');
```

**Option 2: Dedicated Search Engine (Elasticsearch)**
- Index products in Elasticsearch
- Fast full-text search, filtering, aggregations
- Sync with database via events/queue

### Step 7: Handle Reviews and Ratings
```sql
CREATE TABLE reviews (
    review_id SERIAL PRIMARY KEY,
    product_id INTEGER REFERENCES products(product_id),
    user_id INTEGER REFERENCES users(user_id),
    order_id INTEGER REFERENCES orders(order_id), -- Only verified purchases
    rating INTEGER CHECK (rating BETWEEN 1 AND 5),
    title VARCHAR(200),
    content TEXT,
    is_verified_purchase BOOLEAN DEFAULT FALSE,
    helpful_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Denormalized rating summary for performance
CREATE TABLE product_ratings (
    product_id INTEGER PRIMARY KEY REFERENCES products(product_id),
    average_rating DECIMAL(3, 2),
    total_reviews INTEGER DEFAULT 0,
    rating_5_count INTEGER DEFAULT 0,
    rating_4_count INTEGER DEFAULT 0,
    rating_3_count INTEGER DEFAULT 0,
    rating_2_count INTEGER DEFAULT 0,
    rating_1_count INTEGER DEFAULT 0,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 4. Sample Solution

### Complete Database Schema

```sql
-- ============================================
-- USERS AND AUTHENTICATION
-- ============================================

CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    phone VARCHAR(20),
    is_seller BOOLEAN DEFAULT FALSE,
    email_verified BOOLEAN DEFAULT FALSE,
    status VARCHAR(20) DEFAULT 'active', -- active, suspended, deleted
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_status ON users(status) WHERE status = 'active';

-- ============================================
-- ADDRESSES
-- ============================================

CREATE TABLE addresses (
    address_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id) ON DELETE CASCADE,
    address_type VARCHAR(20), -- shipping, billing
    full_name VARCHAR(200),
    address_line1 VARCHAR(255) NOT NULL,
    address_line2 VARCHAR(255),
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100),
    postal_code VARCHAR(20) NOT NULL,
    country VARCHAR(100) NOT NULL,
    phone VARCHAR(20),
    is_default BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_addresses_user ON addresses(user_id);

-- ============================================
-- PRODUCT CATALOG
-- ============================================

CREATE TABLE categories (
    category_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    parent_category_id INTEGER REFERENCES categories(category_id),
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    image_url VARCHAR(500),
    sort_order INTEGER DEFAULT 0,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_categories_parent ON categories(parent_category_id);
CREATE INDEX idx_categories_slug ON categories(slug);

CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    seller_id INTEGER REFERENCES users(user_id),
    category_id INTEGER REFERENCES categories(category_id),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    brand VARCHAR(100),
    base_price DECIMAL(10, 2) NOT NULL CHECK (base_price >= 0),
    is_active BOOLEAN DEFAULT TRUE,
    featured BOOLEAN DEFAULT FALSE,
    search_vector tsvector, -- For full-text search
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_products_seller ON products(seller_id);
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_products_name ON products(name);
CREATE INDEX idx_products_search ON products USING GIN(search_vector);
CREATE INDEX idx_products_active ON products(is_active) WHERE is_active = TRUE;
CREATE INDEX idx_products_featured ON products(featured) WHERE featured = TRUE;

-- Auto-update search vector
CREATE TRIGGER products_search_vector_update
BEFORE INSERT OR UPDATE ON products
FOR EACH ROW EXECUTE FUNCTION
  tsvector_update_trigger(search_vector, 'pg_catalog.english', name, description, brand);

CREATE TABLE product_images (
    image_id SERIAL PRIMARY KEY,
    product_id INTEGER REFERENCES products(product_id) ON DELETE CASCADE,
    image_url VARCHAR(500) NOT NULL,
    alt_text VARCHAR(255),
    sort_order INTEGER DEFAULT 0,
    is_primary BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_product_images_product ON product_images(product_id);

CREATE TABLE product_variants (
    variant_id SERIAL PRIMARY KEY,
    product_id INTEGER REFERENCES products(product_id) ON DELETE CASCADE,
    sku VARCHAR(100) UNIQUE NOT NULL,
    name VARCHAR(100),
    attributes JSONB, -- {"color": "red", "size": "M"}
    price_adjustment DECIMAL(10, 2) DEFAULT 0,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_product_variants_product ON product_variants(product_id);
CREATE INDEX idx_product_variants_sku ON product_variants(sku);
CREATE INDEX idx_product_variants_attributes ON product_variants USING GIN(attributes);

-- ============================================
-- INVENTORY MANAGEMENT
-- ============================================

CREATE TABLE warehouses (
    warehouse_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    address VARCHAR(500),
    city VARCHAR(100),
    country VARCHAR(100),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE inventory (
    inventory_id SERIAL PRIMARY KEY,
    variant_id INTEGER REFERENCES product_variants(variant_id) ON DELETE CASCADE,
    warehouse_id INTEGER REFERENCES warehouses(warehouse_id),
    quantity INTEGER DEFAULT 0 CHECK (quantity >= 0),
    reserved_quantity INTEGER DEFAULT 0 CHECK (reserved_quantity >= 0),
    reorder_level INTEGER DEFAULT 10,
    reorder_quantity INTEGER DEFAULT 50,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(variant_id, warehouse_id)
);

CREATE INDEX idx_inventory_variant ON inventory(variant_id);
CREATE INDEX idx_inventory_warehouse ON inventory(warehouse_id);
CREATE INDEX idx_inventory_low_stock ON inventory(quantity) 
    WHERE quantity <= reorder_level;

-- ============================================
-- SHOPPING CART
-- ============================================

CREATE TABLE cart_items (
    cart_item_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id) ON DELETE CASCADE,
    variant_id INTEGER REFERENCES product_variants(variant_id) ON DELETE CASCADE,
    quantity INTEGER DEFAULT 1 CHECK (quantity > 0),
    added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, variant_id)
);

CREATE INDEX idx_cart_items_user ON cart_items(user_id);

-- ============================================
-- ORDERS
-- ============================================

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id),
    order_number VARCHAR(50) UNIQUE NOT NULL,
    status VARCHAR(50) NOT NULL, -- pending, paid, processing, shipped, delivered, cancelled, returned
    subtotal DECIMAL(10, 2) NOT NULL,
    discount_amount DECIMAL(10, 2) DEFAULT 0,
    tax_amount DECIMAL(10, 2) DEFAULT 0,
    shipping_cost DECIMAL(10, 2) DEFAULT 0,
    total_amount DECIMAL(10, 2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    shipping_address_id INTEGER REFERENCES addresses(address_id),
    billing_address_id INTEGER REFERENCES addresses(address_id),
    promo_code_id INTEGER REFERENCES promo_codes(promo_code_id),
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_number ON orders(order_number);
CREATE INDEX idx_orders_created ON orders(created_at DESC);

CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(order_id) ON DELETE CASCADE,
    product_id INTEGER REFERENCES products(product_id),
    variant_id INTEGER REFERENCES product_variants(variant_id),
    -- Snapshot data at time of order (denormalized for history)
    product_name VARCHAR(255) NOT NULL,
    variant_name VARCHAR(100),
    sku VARCHAR(100),
    price DECIMAL(10, 2) NOT NULL,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    subtotal DECIMAL(10, 2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_order_items_order ON order_items(order_id);
CREATE INDEX idx_order_items_product ON order_items(product_id);

CREATE TABLE order_status_history (
    history_id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(order_id) ON DELETE CASCADE,
    status VARCHAR(50) NOT NULL,
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by INTEGER REFERENCES users(user_id)
);

CREATE INDEX idx_order_status_history_order ON order_status_history(order_id);

-- ============================================
-- PAYMENTS
-- ============================================

CREATE TABLE payment_methods (
    payment_method_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id) ON DELETE CASCADE,
    type VARCHAR(50), -- credit_card, debit_card, paypal, etc.
    card_last4 VARCHAR(4),
    card_brand VARCHAR(50),
    expiry_month INTEGER,
    expiry_year INTEGER,
    is_default BOOLEAN DEFAULT FALSE,
    -- Store tokenized payment data, never raw card numbers
    payment_token VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_payment_methods_user ON payment_methods(user_id);

CREATE TABLE payments (
    payment_id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(order_id),
    payment_method_id INTEGER REFERENCES payment_methods(payment_method_id),
    amount DECIMAL(10, 2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    status VARCHAR(50), -- pending, completed, failed, refunded
    transaction_id VARCHAR(255), -- External payment gateway reference
    payment_gateway VARCHAR(50), -- stripe, paypal, etc.
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP
);

CREATE INDEX idx_payments_order ON payments(order_id);
CREATE INDEX idx_payments_status ON payments(status);
CREATE INDEX idx_payments_transaction ON payments(transaction_id);

-- ============================================
-- REVIEWS AND RATINGS
-- ============================================

CREATE TABLE reviews (
    review_id SERIAL PRIMARY KEY,
    product_id INTEGER REFERENCES products(product_id) ON DELETE CASCADE,
    user_id INTEGER REFERENCES users(user_id) ON DELETE CASCADE,
    order_id INTEGER REFERENCES orders(order_id),
    rating INTEGER NOT NULL CHECK (rating BETWEEN 1 AND 5),
    title VARCHAR(200),
    content TEXT,
    is_verified_purchase BOOLEAN DEFAULT FALSE,
    helpful_count INTEGER DEFAULT 0,
    status VARCHAR(20) DEFAULT 'approved', -- pending, approved, rejected
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, product_id, order_id)
);

CREATE INDEX idx_reviews_product ON reviews(product_id);
CREATE INDEX idx_reviews_user ON reviews(user_id);
CREATE INDEX idx_reviews_rating ON reviews(rating);
CREATE INDEX idx_reviews_status ON reviews(status) WHERE status = 'approved';

-- Denormalized ratings for performance
CREATE TABLE product_ratings (
    product_id INTEGER PRIMARY KEY REFERENCES products(product_id) ON DELETE CASCADE,
    average_rating DECIMAL(3, 2) DEFAULT 0,
    total_reviews INTEGER DEFAULT 0,
    rating_5_count INTEGER DEFAULT 0,
    rating_4_count INTEGER DEFAULT 0,
    rating_3_count INTEGER DEFAULT 0,
    rating_2_count INTEGER DEFAULT 0,
    rating_1_count INTEGER DEFAULT 0,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- ============================================
-- PROMOTIONS AND DISCOUNTS
-- ============================================

CREATE TABLE promo_codes (
    promo_code_id SERIAL PRIMARY KEY,
    code VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    discount_type VARCHAR(20), -- percentage, fixed_amount
    discount_value DECIMAL(10, 2) NOT NULL,
    min_purchase_amount DECIMAL(10, 2),
    max_discount_amount DECIMAL(10, 2),
    usage_limit INTEGER,
    used_count INTEGER DEFAULT 0,
    valid_from TIMESTAMP,
    valid_until TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_promo_codes_code ON promo_codes(code);
CREATE INDEX idx_promo_codes_active ON promo_codes(is_active, valid_from, valid_until);

-- ============================================
-- WISHLIST
-- ============================================

CREATE TABLE wishlist_items (
    wishlist_item_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id) ON DELETE CASCADE,
    product_id INTEGER REFERENCES products(product_id) ON DELETE CASCADE,
    added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, product_id)
);

CREATE INDEX idx_wishlist_items_user ON wishlist_items(user_id);
```

### Key Design Decisions Explained

#### 1. Denormalized Order Items
**Decision**: Store product name, price, etc. in order_items

**Why**:
- Product details might change after order placed
- Historical accuracy (price at time of purchase)
- Faster order queries (no joins needed)

**Trade-off**: Data duplication vs. query performance and accuracy

#### 2. Separate Inventory Table
**Decision**: Track inventory per variant per warehouse

**Why**:
- Support multiple warehouses
- Track reserved quantities (items in carts)
- Enable warehouse-specific allocation

#### 3. Product Ratings Denormalization
**Decision**: Separate table with aggregated ratings

**Why**:
- Avoid computing average on every product query
- Faster product listing pages
- Updated via triggers or background jobs

#### 4. JSONB for Variant Attributes
**Decision**: Store variant attributes as JSONB

**Why**:
- Different products have different attributes
- Flexible schema (add new attributes easily)
- PostgreSQL JSONB has good query performance

**Alternative**: EAV model (more normalized, harder to query)

### Sample Queries

#### Product Search with Filters
```sql
SELECT p.*, pr.average_rating, pr.total_reviews
FROM products p
LEFT JOIN product_ratings pr ON p.product_id = pr.product_id
WHERE p.is_active = TRUE
  AND p.category_id IN (SELECT category_id FROM categories WHERE slug = 'electronics')
  AND p.base_price BETWEEN 100 AND 500
  AND pr.average_rating >= 4.0
  AND p.search_vector @@ to_tsquery('english', 'laptop')
ORDER BY pr.average_rating DESC, p.created_at DESC
LIMIT 20;
```

#### Create Order (Transaction)
```sql
BEGIN TRANSACTION;

-- Create order
INSERT INTO orders (user_id, order_number, status, subtotal, total_amount, shipping_address_id)
VALUES (123, 'ORD-2024-12345', 'pending', 150.00, 165.00, 456)
RETURNING order_id INTO v_order_id;

-- Move cart items to order
INSERT INTO order_items (order_id, product_id, variant_id, product_name, variant_name, price, quantity, subtotal)
SELECT v_order_id, p.product_id, pv.variant_id, p.name, pv.name, 
       p.base_price + pv.price_adjustment, ci.quantity,
       (p.base_price + pv.price_adjustment) * ci.quantity
FROM cart_items ci
JOIN product_variants pv ON ci.variant_id = pv.variant_id
JOIN products p ON pv.product_id = p.product_id
WHERE ci.user_id = 123;

-- Reduce inventory (move reserved to sold)
UPDATE inventory i
SET quantity = quantity - ci.quantity,
    reserved_quantity = reserved_quantity - ci.quantity
FROM cart_items ci
WHERE i.variant_id = ci.variant_id
  AND ci.user_id = 123;

-- Clear cart
DELETE FROM cart_items WHERE user_id = 123;

COMMIT;
```

#### Update Product Ratings (Trigger)
```sql
CREATE OR REPLACE FUNCTION update_product_ratings()
RETURNS TRIGGER AS $$
BEGIN
    -- Recalculate ratings when review added/updated
    INSERT INTO product_ratings (product_id, average_rating, total_reviews, 
                                  rating_5_count, rating_4_count, rating_3_count, 
                                  rating_2_count, rating_1_count)
    SELECT 
        product_id,
        AVG(rating)::DECIMAL(3,2),
        COUNT(*),
        COUNT(*) FILTER (WHERE rating = 5),
        COUNT(*) FILTER (WHERE rating = 4),
        COUNT(*) FILTER (WHERE rating = 3),
        COUNT(*) FILTER (WHERE rating = 2),
        COUNT(*) FILTER (WHERE rating = 1)
    FROM reviews
    WHERE product_id = NEW.product_id
      AND status = 'approved'
    GROUP BY product_id
    ON CONFLICT (product_id) 
    DO UPDATE SET
        average_rating = EXCLUDED.average_rating,
        total_reviews = EXCLUDED.total_reviews,
        rating_5_count = EXCLUDED.rating_5_count,
        rating_4_count = EXCLUDED.rating_4_count,
        rating_3_count = EXCLUDED.rating_3_count,
        rating_2_count = EXCLUDED.rating_2_count,
        rating_1_count = EXCLUDED.rating_1_count,
        updated_at = CURRENT_TIMESTAMP;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER review_update_ratings
AFTER INSERT OR UPDATE ON reviews
FOR EACH ROW
EXECUTE FUNCTION update_product_ratings();
```

## 5. Extension Challenges

### Challenge 1: Multi-Currency Support
**Requirement**: Support multiple currencies with real-time conversion

**How would you modify the schema?**
<details>
<summary>Solution</summary>

```sql
-- Add currency tables
CREATE TABLE currencies (
    currency_code VARCHAR(3) PRIMARY KEY,
    name VARCHAR(100),
    symbol VARCHAR(10),
    is_active BOOLEAN DEFAULT TRUE
);

CREATE TABLE exchange_rates (
    rate_id SERIAL PRIMARY KEY,
    from_currency VARCHAR(3) REFERENCES currencies(currency_code),
    to_currency VARCHAR(3) REFERENCES currencies(currency_code),
    rate DECIMAL(12, 6) NOT NULL,
    effective_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(from_currency, to_currency, effective_date)
);

-- Modify products table
ALTER TABLE products 
ADD COLUMN currency VARCHAR(3) DEFAULT 'USD' REFERENCES currencies(currency_code);

-- Store prices in multiple currencies
CREATE TABLE product_prices (
    price_id SERIAL PRIMARY KEY,
    product_id INTEGER REFERENCES products(product_id),
    currency VARCHAR(3) REFERENCES currencies(currency_code),
    price DECIMAL(10, 2) NOT NULL,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(product_id, currency)
);

-- Display price in user's currency
CREATE FUNCTION get_price_in_currency(p_product_id INTEGER, p_currency VARCHAR(3))
RETURNS DECIMAL(10, 2) AS $$
DECLARE
    v_price DECIMAL(10, 2);
    v_base_currency VARCHAR(3);
    v_base_price DECIMAL(10, 2);
    v_rate DECIMAL(12, 6);
BEGIN
    -- Try to get direct price
    SELECT price INTO v_price
    FROM product_prices
    WHERE product_id = p_product_id AND currency = p_currency;
    
    IF FOUND THEN
        RETURN v_price;
    END IF;
    
    -- Convert from base currency
    SELECT base_price, currency INTO v_base_price, v_base_currency
    FROM products
    WHERE product_id = p_product_id;
    
    SELECT rate INTO v_rate
    FROM exchange_rates
    WHERE from_currency = v_base_currency
      AND to_currency = p_currency
    ORDER BY effective_date DESC
    LIMIT 1;
    
    RETURN v_base_price * v_rate;
END;
$$ LANGUAGE plpgsql;
```
</details>

### Challenge 2: Auction/Bidding System
**Requirement**: Add auction functionality for some products

**How would you extend the schema?**
<details>
<summary>Solution</summary>

```sql
-- Auctions table
CREATE TABLE auctions (
    auction_id SERIAL PRIMARY KEY,
    product_id INTEGER REFERENCES products(product_id),
    seller_id INTEGER REFERENCES users(user_id),
    starting_price DECIMAL(10, 2) NOT NULL,
    reserve_price DECIMAL(10, 2), -- Minimum price to sell
    current_bid DECIMAL(10, 2),
    current_winner_id INTEGER REFERENCES users(user_id),
    bid_increment DECIMAL(10, 2) DEFAULT 1.00,
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP NOT NULL,
    status VARCHAR(20), -- active, ended, cancelled
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE bids (
    bid_id SERIAL PRIMARY KEY,
    auction_id INTEGER REFERENCES auctions(auction_id),
    bidder_id INTEGER REFERENCES users(user_id),
    bid_amount DECIMAL(10, 2) NOT NULL,
    is_auto_bid BOOLEAN DEFAULT FALSE,
    max_bid_amount DECIMAL(10, 2), -- For auto-bidding
    status VARCHAR(20), -- active, outbid, winning, won, lost
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_auctions_status ON auctions(status, end_time);
CREATE INDEX idx_bids_auction ON bids(auction_id, created_at DESC);

-- Place bid function
CREATE FUNCTION place_bid(p_auction_id INTEGER, p_bidder_id INTEGER, p_amount DECIMAL(10, 2))
RETURNS BOOLEAN AS $$
DECLARE
    v_current_bid DECIMAL(10, 2);
    v_min_bid DECIMAL(10, 2);
    v_increment DECIMAL(10, 2);
BEGIN
    -- Lock auction row
    SELECT current_bid, bid_increment INTO v_current_bid, v_increment
    FROM auctions
    WHERE auction_id = p_auction_id
      AND status = 'active'
      AND end_time > NOW()
    FOR UPDATE;
    
    IF NOT FOUND THEN
        RAISE EXCEPTION 'Auction not active';
    END IF;
    
    -- Calculate minimum bid
    v_min_bid := COALESCE(v_current_bid, 0) + v_increment;
    
    IF p_amount < v_min_bid THEN
        RAISE EXCEPTION 'Bid too low. Minimum: %', v_min_bid;
    END IF;
    
    -- Insert bid
    INSERT INTO bids (auction_id, bidder_id, bid_amount, status)
    VALUES (p_auction_id, p_bidder_id, p_amount, 'winning');
    
    -- Update auction
    UPDATE auctions
    SET current_bid = p_amount,
        current_winner_id = p_bidder_id
    WHERE auction_id = p_auction_id;
    
    -- Mark previous bids as outbid
    UPDATE bids
    SET status = 'outbid'
    WHERE auction_id = p_auction_id
      AND bidder_id != p_bidder_id
      AND status = 'winning';
    
    RETURN TRUE;
END;
$$ LANGUAGE plpgsql;
```
</details>

### Challenge 3: Subscription Products
**Requirement**: Support recurring subscription products (monthly, yearly)

**How would you modify the schema?**
<details>
<summary>Solution</summary>

```sql
-- Subscription plans
CREATE TABLE subscription_plans (
    plan_id SERIAL PRIMARY KEY,
    product_id INTEGER REFERENCES products(product_id),
    name VARCHAR(100) NOT NULL,
    billing_interval VARCHAR(20), -- monthly, yearly, weekly
    billing_interval_count INTEGER DEFAULT 1,
    price DECIMAL(10, 2) NOT NULL,
    trial_period_days INTEGER DEFAULT 0,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Customer subscriptions
CREATE TABLE subscriptions (
    subscription_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id),
    plan_id INTEGER REFERENCES subscription_plans(plan_id),
    status VARCHAR(20), -- active, cancelled, past_due, paused
    current_period_start TIMESTAMP,
    current_period_end TIMESTAMP,
    cancel_at_period_end BOOLEAN DEFAULT FALSE,
    cancelled_at TIMESTAMP,
    trial_end TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Subscription invoices
CREATE TABLE subscription_invoices (
    invoice_id SERIAL PRIMARY KEY,
    subscription_id INTEGER REFERENCES subscriptions(subscription_id),
    amount DECIMAL(10, 2) NOT NULL,
    status VARCHAR(20), -- draft, open, paid, void, uncollectible
    period_start TIMESTAMP,
    period_end TIMESTAMP,
    due_date TIMESTAMP,
    paid_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_subscriptions_user ON subscriptions(user_id);
CREATE INDEX idx_subscriptions_status ON subscriptions(status);
CREATE INDEX idx_subscription_invoices_subscription ON subscription_invoices(subscription_id);
CREATE INDEX idx_subscription_invoices_due ON subscription_invoices(status, due_date);
```
</details>

### Challenge 4: Performance at 100x Scale
**Scenario**: System grows to 1 billion users, 100 million products

**How would you optimize the database?**
<details>
<summary>Solution</summary>

**1. Partitioning**:
```sql
-- Partition orders by date (monthly)
CREATE TABLE orders (
    order_id BIGSERIAL,
    created_at TIMESTAMP NOT NULL,
    ...
) PARTITION BY RANGE (created_at);

CREATE TABLE orders_2024_01 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE orders_2024_02 PARTITION OF orders
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
```

**2. Sharding** (application-level):
```python
# Shard by user_id
def get_shard(user_id):
    return user_id % NUM_SHARDS

# Route queries to appropriate shard
shard_id = get_shard(user_id)
connection = shard_connections[shard_id]
```

**3. Read Replicas**:
```
Primary DB (writes)
    ↓ replication
Read Replica 1 (reads - product catalog)
Read Replica 2 (reads - order history)
Read Replica 3 (reads - search)
```

**4. Caching Strategy**:
```python
# Cache hot data
- Product details: 1 hour
- Category tree: 1 hour
- User session: 24 hours
- Shopping cart: 1 hour
- Product ratings: 30 minutes

# Cache at multiple levels
1. Application cache (Redis)
2. Query result cache
3. CDN (for static content)
```

**5. Denormalization for Performance**:
```sql
-- Materialized view for product listing
CREATE MATERIALIZED VIEW product_listing_cache AS
SELECT 
    p.product_id,
    p.name,
    p.base_price,
    c.name as category_name,
    pr.average_rating,
    pr.total_reviews,
    MIN(pi.image_url) as primary_image
FROM products p
JOIN categories c ON p.category_id = c.category_id
LEFT JOIN product_ratings pr ON p.product_id = pr.product_id
LEFT JOIN product_images pi ON p.product_id = pi.product_id
WHERE p.is_active = TRUE
GROUP BY p.product_id, p.name, p.base_price, c.name, pr.average_rating, pr.total_reviews;

CREATE UNIQUE INDEX ON product_listing_cache(product_id);

-- Refresh periodically (every 5 minutes)
REFRESH MATERIALIZED VIEW CONCURRENTLY product_listing_cache;
```

**6. Move to Specialized Databases**:
- **Product Search**: Elasticsearch
- **User Sessions**: Redis
- **Analytics**: ClickHouse/BigQuery
- **Images**: S3/CDN
- **Transactional Data**: PostgreSQL (sharded)

**7. Archival Strategy**:
```sql
-- Move old orders to archive
CREATE TABLE orders_archive (LIKE orders INCLUDING ALL);

-- Move orders older than 2 years
INSERT INTO orders_archive
SELECT * FROM orders WHERE created_at < NOW() - INTERVAL '2 years';

DELETE FROM orders WHERE created_at < NOW() - INTERVAL '2 years';
```
</details>

### Challenge 5: GDPR Compliance
**Requirement**: Support user data export and deletion (GDPR right to be forgotten)

**How would you implement this?**
<details>
<summary>Solution</summary>

```sql
-- Data export function
CREATE FUNCTION export_user_data(p_user_id INTEGER)
RETURNS JSON AS $$
DECLARE
    v_data JSON;
BEGIN
    SELECT json_build_object(
        'user', (SELECT row_to_json(u) FROM users u WHERE user_id = p_user_id),
        'addresses', (SELECT json_agg(row_to_json(a)) FROM addresses a WHERE user_id = p_user_id),
        'orders', (SELECT json_agg(row_to_json(o)) FROM orders o WHERE user_id = p_user_id),
        'reviews', (SELECT json_agg(row_to_json(r)) FROM reviews r WHERE user_id = p_user_id),
        'cart', (SELECT json_agg(row_to_json(c)) FROM cart_items c WHERE user_id = p_user_id),
        'wishlist', (SELECT json_agg(row_to_json(w)) FROM wishlist_items w WHERE user_id = p_user_id)
    ) INTO v_data;
    
    RETURN v_data;
END;
$$ LANGUAGE plpgsql;

-- Anonymize user data (soft delete, keep for business records)
CREATE FUNCTION anonymize_user(p_user_id INTEGER)
RETURNS BOOLEAN AS $$
BEGIN
    -- Update user record
    UPDATE users
    SET email = 'deleted-' || user_id || '@example.com',
        first_name = 'Deleted',
        last_name = 'User',
        phone = NULL,
        status = 'deleted'
    WHERE user_id = p_user_id;
    
    -- Delete addresses
    DELETE FROM addresses WHERE user_id = p_user_id;
    
    -- Delete payment methods
    DELETE FROM payment_methods WHERE user_id = p_user_id;
    
    -- Anonymize reviews (keep for product ratings)
    UPDATE reviews
    SET user_id = NULL
    WHERE user_id = p_user_id;
    
    -- Delete cart and wishlist
    DELETE FROM cart_items WHERE user_id = p_user_id;
    DELETE FROM wishlist_items WHERE user_id = p_user_id;
    
    -- Keep orders but anonymize (legal requirement for financial records)
    -- Orders remain linked to anonymized user
    
    RETURN TRUE;
END;
$$ LANGUAGE plpgsql;

-- Data retention policy table
CREATE TABLE data_deletion_requests (
    request_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id),
    requested_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    scheduled_deletion_date TIMESTAMP, -- 30 days after request
    status VARCHAR(20), -- pending, processing, completed
    completed_at TIMESTAMP
);

-- Background job processes deletion requests after grace period
```
</details>

---

## Summary

This design exercise demonstrates how to design a production-ready e-commerce database by:
- Identifying core entities and relationships
- Choosing appropriate normalization levels
- Handling complex business logic (inventory, variants, orders)
- Optimizing for performance (indexes, denormalization)
- Planning for scale (partitioning, sharding, read replicas)
- Ensuring data integrity (transactions, constraints)

**Key Takeaways**:
1. **Start normalized, denormalize for performance**: Begin with proper normalization, then strategically denormalize
2. **Index strategically**: Index columns used in WHERE, JOIN, ORDER BY
3. **Use transactions for critical operations**: Inventory management, order creation
4. **Plan for scale from the start**: Partitioning, sharding strategies
5. **Separate read and write workloads**: Use read replicas
6. **Cache aggressively**: Most e-commerce is read-heavy
