# Chapter 3: Database Design & Modeling

## Learning Objectives
By the end of this chapter, you will be able to:
1. **Design** normalized relational database schemas following Third Normal Form (3NF) principles to eliminate data anomalies
2. **Evaluate** the trade-offs between normalization and denormalization in different contexts
3. **Implement** appropriate indexes to optimize query performance for specific workload patterns
4. **Analyze** query execution plans to identify and resolve performance bottlenecks

## Introduction

Your e-commerce startup just hit product-market fit. Orders are flooding in. Your database, which worked fine for the first 1,000 customers, is now grinding to a halt. Queries that took 100ms now take 30 seconds. Worse, you're finding duplicate data everywhere—the same customer email spelled three different ways, product prices that don't match between tables, and orders that reference products that don't exist.

Last week, a developer tried to update a customer's shipping address and accidentally changed it for 500 orders instead of just future ones. Yesterday, the marketing team tried to analyze "top products by category" and the query ran for six hours before you killed it.

These aren't random bugs. They're symptoms of **poor database design**. The difference between a well-designed schema and a poorly designed one is the difference between a database that scales gracefully and one that becomes a bottleneck the moment you get traction.

This chapter is about making intentional design decisions. You'll learn when to normalize (split data across tables) and when to denormalize (combine data for performance). You'll understand why some queries are fast and others are slow, and how to fix the slow ones. And you'll see real examples of how design choices made in week one affect your ability to scale in year one.

Let's start with the foundation: normalization. We'll begin with a messy real-world dataset and progressively clean it up.

## Section 1: The Problem with Denormalized Data

### Key Idea
Denormalized data—storing everything in one big table—leads to redundancy, update anomalies, and data integrity issues. Normalization solves these problems by organizing data into related tables.

### Example: A Messy E-Commerce Spreadsheet

Your startup began with a spreadsheet to track orders. As you grew, you turned it into a database table. Here's what it looks like:

**orders table:**
```
order_id | customer_name | customer_email     | customer_phone  | product_name    | product_category | product_price | quantity | order_date
---------|---------------|--------------------|-----------------|-----------------|-----------------|--------------|---------|-----------
1        | Alice Smith   | alice@email.com    | 555-0101        | Python Book     | Books           | 29.99        | 2       | 2024-01-15
2        | Bob Jones     | bob@email.com      | 555-0102        | SQL Course      | Courses         | 199.00       | 1       | 2024-01-16
3        | Alice Smith   | alice@email.com    | 555-0101        | Docker Guide    | Books           | 39.99        | 1       | 2024-01-17
4        | Alice Smith   | asmith@email.com   | 555-0101        | Python Book     | Books           | 29.99        | 1       | 2024-01-18
5        | Bob Jones     | bob@email.com      | 555-0102        | Python Book     | Books           | 34.99        | 3       | 2024-01-19
```

Looks reasonable at first glance. But notice the problems:

**Problem 1: Data Redundancy**
- Alice Smith's information is repeated in rows 1, 3, and 4
- Python Book's details appear in rows 1, 4, and 5
- Every order for the same customer duplicates their contact info

**Problem 2: Update Anomalies**
```sql
-- Alice changes her email. Do we update all three rows?
UPDATE orders SET customer_email = 'alice.smith@newdomain.com'
WHERE customer_name = 'Alice Smith';

-- But wait, row 4 has 'asmith@email.com' - is that the same Alice?
-- Now Alice has two different emails in the database!
```

**Problem 3: Inconsistent Data**
- Row 1: Python Book costs $29.99
- Row 5: Python Book costs $34.99
- Which is correct? Did the price change? Is it an error?

**Problem 4: Insertion Anomaly**
```sql
-- You want to add a new product to the catalog
-- But you can't insert a product without an order!
INSERT INTO orders (product_name, product_category, product_price)
VALUES ('New Book', 'Books', 49.99);
-- Error: order_id, customer_name, etc. are required
```

**Problem 5: Deletion Anomaly**
```sql
-- Bob cancels order #2 (SQL Course)
DELETE FROM orders WHERE order_id = 2;
-- Now you've lost the only record that SQL Course exists!
```

### Why This Matters

In production systems:
- **Redundancy wastes storage** and makes data inconsistent
- **Update anomalies cause bugs** that affect real customers
- **Insertion/deletion anomalies limit** what your application can do
- **Inconsistent data breaks analytics** and reporting

These problems are solvable. The solution is **normalization**: systematically organizing data to eliminate redundancy and anomalies.

### Try It

Look at this denormalized table. How many anomalies can you spot?

```
employee_id | employee_name | department_name | department_location | manager_name  | project_name  | project_budget
------------|---------------|-----------------|---------------------|---------------|---------------|---------------
1           | Alice         | Engineering     | Building A          | Bob           | Data Pipeline | 100000
1           | Alice         | Engineering     | Building A          | Bob           | ML Platform   | 150000
2           | Carol         | Marketing       | Building B          | Dave          | Campaign 2024 | 50000
3           | Eve           | Engineering     | Building C          | Bob           | Data Pipeline | 100000
```

Problems to consider:
- What happens if Engineering moves to Building C?
- What if Bob is replaced as manager?
- Can you add a new department without adding an employee?

## Section 2: Normalization to Third Normal Form (3NF)

### Key Idea
Normalization is a step-by-step process of decomposing tables to eliminate redundancy. Third Normal Form (3NF) eliminates most practical issues while remaining intuitive and efficient.

### The Normalization Process

Let's fix the messy orders table step by step.

**First Normal Form (1NF): Atomic Values**

Rule: Each column must contain atomic (indivisible) values. No arrays, lists, or multi-valued attributes.

Our orders table already satisfies 1NF (each cell contains a single value). But imagine if we had:

```
order_id | customer_name | products
---------|---------------|---------------------------
1        | Alice Smith   | Python Book, Docker Guide
```

This violates 1NF because `products` contains multiple values. Fix:

```
order_id | customer_name | product
---------|---------------|-------------
1        | Alice Smith   | Python Book
1        | Alice Smith   | Docker Guide
```

**Second Normal Form (2NF): No Partial Dependencies**

Rule: All non-key columns must depend on the entire primary key, not just part of it.

If our primary key were `(order_id, product_name)`, then `customer_name` depends only on `order_id`, not the full key. This is a **partial dependency**.

Fix: Split into separate tables:

```sql
-- orders table (one row per order)
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    customer_phone VARCHAR(20),
    order_date DATE
);

-- order_items table (one row per product in an order)
CREATE TABLE order_items (
    order_id INT,
    product_name VARCHAR(100),
    quantity INT,
    PRIMARY KEY (order_id, product_name),
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);
```

**Third Normal Form (3NF): No Transitive Dependencies**

Rule: Non-key columns must not depend on other non-key columns.

In our `orders` table, `customer_name`, `customer_email`, and `customer_phone` all describe the customer, not the order. They're **transitively dependent** on `order_id` through `customer_id`.

In `order_items`, `product_category` and `product_price` depend on `product_name`, not the order.

**Final Normalized Schema (3NF):**

```sql
-- Customers table
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    customer_email VARCHAR(100) UNIQUE NOT NULL,
    customer_phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Products table
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    product_category VARCHAR(50) NOT NULL,
    product_price DECIMAL(10, 2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Orders table
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    order_total DECIMAL(10, 2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- Order items table
CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL CHECK (quantity > 0),
    price_at_purchase DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

### Before and After: Problem Resolution

**Original Issue: Update Alice's email**

Before (denormalized):
```sql
-- Have to update multiple rows, might miss some
UPDATE orders SET customer_email = 'alice.new@email.com'
WHERE customer_name = 'Alice Smith';  -- What if name is misspelled?
```

After (normalized):
```sql
-- Update once, affects all orders automatically
UPDATE customers SET customer_email = 'alice.new@email.com'
WHERE customer_id = 123;
```

**Original Issue: Change product price**

Before (denormalized):
```sql
-- Do we update all past orders? Just future ones?
UPDATE orders SET product_price = 34.99
WHERE product_name = 'Python Book';  -- This changes historical data!
```

After (normalized):
```sql
-- Update product catalog
UPDATE products SET product_price = 34.99
WHERE product_id = 5;

-- Past orders are unaffected (they store price_at_purchase)
-- Future orders get the new price automatically
```

**Original Issue: Add product without an order**

Before (denormalized):
```sql
-- Impossible!
```

After (normalized):
```sql
-- Easy
INSERT INTO products (product_name, product_category, product_price)
VALUES ('New Book', 'Books', 49.99);
```

### Understanding Foreign Keys

Foreign keys enforce **referential integrity**:

```sql
-- Try to create an order for a non-existent customer
INSERT INTO orders (customer_id, order_date)
VALUES (999999, CURRENT_DATE);
-- Error: foreign key constraint violated

-- Try to delete a customer who has orders
DELETE FROM customers WHERE customer_id = 123;
-- Error: violates foreign key constraint

-- Cascade option: delete orders when customer is deleted
ALTER TABLE orders
ADD CONSTRAINT fk_customer
FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
ON DELETE CASCADE;  -- Be careful with this!
```

### Why This Matters

Normalized schemas:
- **Prevent data anomalies**: One place to update each fact
- **Enforce data integrity**: Foreign keys prevent orphaned records
- **Reduce storage**: No redundant data
- **Enable flexible queries**: Join tables as needed for different analyses

In data engineering:
- **ETL pipelines benefit**: Extract from normalized sources without deduplication logic
- **Data quality improves**: Constraints catch errors at insert time
- **Migrations are safer**: Changing one table doesn't require updating redundant data

### Try It

Normalize this denormalized table to 3NF:

```
invoice_id | customer_name | customer_city | product_name | product_vendor | vendor_country | quantity
-----------|---------------|---------------|--------------|----------------|----------------|----------
1          | Alice         | New York      | Widget A     | Acme Corp      | USA            | 10
2          | Bob           | London        | Widget A     | Acme Corp      | USA            | 5
3          | Alice         | New York      | Widget B     | Beta Inc       | Canada         | 3
```

<details>
<summary>Solution</summary>

```sql
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100),
    customer_city VARCHAR(100)
);

CREATE TABLE vendors (
    vendor_id SERIAL PRIMARY KEY,
    vendor_name VARCHAR(100),
    vendor_country VARCHAR(100)
);

CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100),
    vendor_id INT,
    FOREIGN KEY (vendor_id) REFERENCES vendors(vendor_id)
);

CREATE TABLE invoices (
    invoice_id SERIAL PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE invoice_items (
    invoice_item_id SERIAL PRIMARY KEY,
    invoice_id INT,
    product_id INT,
    quantity INT,
    FOREIGN KEY (invoice_id) REFERENCES invoices(invoice_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```
</details>

## Section 3: Strategic Denormalization for Performance

### Key Idea
While normalization prevents anomalies, it requires joins for queries. In some cases, intentional denormalization improves read performance at the cost of write complexity and storage.

### Example: The Performance Problem with Joins

Your normalized e-commerce schema works great. But your most common query—showing orders with customer and product details—is getting slow:

```sql
-- This query runs on every page load
SELECT
    o.order_id,
    o.order_date,
    c.customer_name,
    c.customer_email,
    p.product_name,
    p.product_category,
    oi.quantity,
    oi.price_at_purchase,
    oi.quantity * oi.price_at_purchase AS line_total
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE o.order_date >= CURRENT_DATE - INTERVAL '7 days'
ORDER BY o.order_date DESC;
```

At 1,000 orders/day, this query takes 50ms. At 100,000 orders/day, it takes 5 seconds. The problem? Four joins across large tables.

### When to Denormalize

**Denormalize when:**
1. **Read-heavy workload**: 99% reads, 1% writes
2. **Predictable access patterns**: Same queries run repeatedly
3. **Performance is critical**: Page load times matter
4. **Storage is cheap, compute is expensive**: Cloud costs favor less computation

**Don't denormalize when:**
1. **Write-heavy workload**: Frequent updates
2. **Data changes frequently**: Keeping denormalized data in sync is hard
3. **Data integrity is paramount**: Normalization prevents errors
4. **Storage is expensive**: Redundant data costs real money

### Denormalization Strategy 1: Add Computed Columns

Instead of joining to calculate totals:

```sql
-- Add denormalized column
ALTER TABLE orders ADD COLUMN order_total DECIMAL(10, 2);

-- Update with trigger or application code
CREATE OR REPLACE FUNCTION update_order_total()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE orders
    SET order_total = (
        SELECT SUM(quantity * price_at_purchase)
        FROM order_items
        WHERE order_id = NEW.order_id
    )
    WHERE order_id = NEW.order_id;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER order_item_changes
AFTER INSERT OR UPDATE OR DELETE ON order_items
FOR EACH ROW EXECUTE FUNCTION update_order_total();
```

**Trade-off:**
- **Gain**: Fast reads (no aggregation needed)
- **Cost**: Slower writes (trigger overhead), more storage

### Denormalization Strategy 2: Materialized Views

A materialized view is a pre-computed query result stored as a table:

```sql
-- Create materialized view with pre-joined data
CREATE MATERIALIZED VIEW order_details_mv AS
SELECT
    o.order_id,
    o.order_date,
    c.customer_name,
    c.customer_email,
    p.product_name,
    p.product_category,
    oi.quantity,
    oi.price_at_purchase,
    oi.quantity * oi.price_at_purchase AS line_total
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id;

-- Create index for fast queries
CREATE INDEX idx_order_details_date ON order_details_mv(order_date);

-- Query the materialized view (fast!)
SELECT * FROM order_details_mv
WHERE order_date >= CURRENT_DATE - INTERVAL '7 days'
ORDER BY order_date DESC;
```

**Refresh strategy:**

```sql
-- Refresh manually when data changes
REFRESH MATERIALIZED VIEW order_details_mv;

-- Or schedule automatic refresh
-- (via cron, Airflow, or database scheduler)
```

**Trade-off:**
- **Gain**: Extremely fast reads (data is pre-joined)
- **Cost**: Stale data between refreshes, storage for redundant data

### Denormalization Strategy 3: Wide Tables for Analytics

For analytics workloads (data warehouses), fully denormalized "wide tables" are common:

```sql
-- Denormalized analytics table
CREATE TABLE orders_analytics (
    order_id INT,
    order_date DATE,
    customer_id INT,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    customer_signup_date DATE,
    customer_segment VARCHAR(50),
    product_id INT,
    product_name VARCHAR(100),
    product_category VARCHAR(50),
    product_price DECIMAL(10, 2),
    quantity INT,
    price_at_purchase DECIMAL(10, 2),
    line_total DECIMAL(10, 2),
    order_total DECIMAL(10, 2)
);
```

This table duplicates everything but enables lightning-fast analytics:

```sql
-- No joins needed!
SELECT
    product_category,
    SUM(line_total) AS revenue,
    COUNT(DISTINCT customer_id) AS unique_customers
FROM orders_analytics
WHERE order_date >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY product_category
ORDER BY revenue DESC;
```

**Trade-off:**
- **Gain**: Fast analytics, simple queries
- **Cost**: Huge storage, complex ETL to keep in sync

### Real-World Example: Netflix

Netflix uses both normalized and denormalized data:

**Normalized (transactional system):**
- User accounts
- Subscription plans
- Billing records
- Content metadata

**Denormalized (analytics/recommendations):**
- Wide tables with viewing history + user demographics + content attributes
- Enables fast ML model training and recommendation queries
- Updated periodically via ETL pipelines

### Why This Matters

In data engineering:
- **OLTP systems** (transactional) use normalized schemas
- **OLAP systems** (analytics) use denormalized schemas
- Your job is often moving data from normalized to denormalized (ETL)

Understanding when to denormalize:
- Saves infrastructure costs (less compute for queries)
- Improves user experience (faster dashboards)
- Enables real-time analytics (pre-aggregated data)

But over-denormalization causes:
- Data staleness issues
- Complex update logic
- Storage bloat

### Try It

You have a normalized schema for a blog:

```sql
CREATE TABLE users (user_id, username, email);
CREATE TABLE posts (post_id, user_id, title, content, created_at);
CREATE TABLE comments (comment_id, post_id, user_id, text, created_at);
```

Your most common query:
```sql
SELECT p.title, u.username, COUNT(c.comment_id) AS comment_count
FROM posts p
JOIN users u ON p.user_id = u.user_id
LEFT JOIN comments c ON p.post_id = c.post_id
GROUP BY p.post_id, p.title, u.username;
```

Design a denormalized solution. What column(s) would you add? What trigger(s) would keep it updated?

<details>
<summary>Solution</summary>

```sql
-- Add denormalized columns
ALTER TABLE posts
ADD COLUMN author_username VARCHAR(100),
ADD COLUMN comment_count INT DEFAULT 0;

-- Trigger to update comment count
CREATE OR REPLACE FUNCTION update_post_comment_count()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        UPDATE posts SET comment_count = comment_count + 1 WHERE post_id = NEW.post_id;
    ELSIF TG_OP = 'DELETE' THEN
        UPDATE posts SET comment_count = comment_count - 1 WHERE post_id = OLD.post_id;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER comment_count_trigger
AFTER INSERT OR DELETE ON comments
FOR EACH ROW EXECUTE FUNCTION update_post_comment_count();

-- Now the query is simple
SELECT title, author_username, comment_count FROM posts;
```

Trade-offs:
- Faster reads (no joins, no COUNT aggregation)
- Slower writes (trigger overhead)
- Risk: Username change requires updating all posts
</details>

## Section 4: Indexes - The Secret to Fast Queries

### Key Idea
Indexes are data structures that make lookups fast by avoiding full table scans. Choosing the right indexes is crucial for performance, but too many indexes slow down writes.

### Example: The Slow Query Problem

You have a table with 10 million orders:

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    status VARCHAR(20),
    order_total DECIMAL(10, 2)
);
```

This query is painfully slow:

```sql
EXPLAIN ANALYZE
SELECT * FROM orders
WHERE customer_id = 12345
AND order_date >= '2024-01-01';
```

**Output:**
```
Seq Scan on orders (cost=0.00..200000.00 rows=150 width=50) (actual time=1850.234..1850.456 rows=150 loops=1)
  Filter: (customer_id = 12345 AND order_date >= '2024-01-01')
  Rows Removed by Filter: 9999850
Planning Time: 0.123 ms
Execution Time: 1850.567 ms
```

**Seq Scan** means the database scanned all 10 million rows. For 150 matching rows, it checked 9,999,850 unnecessary rows. This doesn't scale.

### How Indexes Work

An index is like a book's index: instead of reading every page to find a topic, you look it up in the index and jump directly to the right page.

**B-tree index** (default in PostgreSQL):

```sql
-- Create index on customer_id
CREATE INDEX idx_orders_customer ON orders(customer_id);
```

Now the query uses the index:

```sql
EXPLAIN ANALYZE
SELECT * FROM orders
WHERE customer_id = 12345;
```

**Output:**
```
Index Scan using idx_orders_customer on orders (cost=0.43..850.21 rows=500 width=50) (actual time=0.045..1.234 rows=500 loops=1)
  Index Cond: (customer_id = 12345)
Planning Time: 0.145 ms
Execution Time: 1.456 ms
```

**Result**: 1850ms → 1.5ms (1,233x faster!)

### Composite Indexes

What if you frequently query by both `customer_id` and `order_date`?

```sql
-- Composite index on multiple columns
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
```

Now this query is fast:

```sql
SELECT * FROM orders
WHERE customer_id = 12345
AND order_date >= '2024-01-01';
```

**Column order matters!**

```sql
-- Works: Query filters by customer_id (first column in index)
WHERE customer_id = 12345

-- Works: Query filters by customer_id and order_date
WHERE customer_id = 12345 AND order_date >= '2024-01-01'

-- DOESN'T use index: Query only filters by order_date (second column)
WHERE order_date >= '2024-01-01'
```

**Rule**: Composite index `(A, B, C)` can be used for queries filtering on:
- `A`
- `A, B`
- `A, B, C`

But NOT for:
- `B`
- `C`
- `B, C`

### Covering Indexes

A **covering index** includes all columns needed by a query, so the database doesn't need to look at the table:

```sql
-- Query that only needs customer_id, order_date, order_total
SELECT customer_id, order_date, order_total
FROM orders
WHERE customer_id = 12345;

-- Create covering index (includes extra column)
CREATE INDEX idx_orders_customer_covering
ON orders(customer_id) INCLUDE (order_date, order_total);
```

The database can satisfy the entire query from the index without touching the table (**index-only scan**).

### Partial Indexes

Index only the rows you care about:

```sql
-- 95% of orders are 'completed', but you only query 'pending' orders
CREATE INDEX idx_orders_pending
ON orders(customer_id, order_date)
WHERE status = 'pending';
```

**Benefits:**
- Smaller index (faster to scan, less storage)
- Faster for queries that match the WHERE condition

```sql
-- Uses partial index
SELECT * FROM orders
WHERE customer_id = 12345 AND status = 'pending';
```

### Expression Indexes

Index a computed value:

```sql
-- Query on lowercase email
SELECT * FROM customers WHERE LOWER(email) = 'alice@example.com';

-- Create index on expression
CREATE INDEX idx_customers_email_lower ON customers(LOWER(email));
```

### Index Trade-offs

**Benefits:**
- Dramatically faster reads (1000x+ speedups possible)
- Enable queries that would otherwise timeout

**Costs:**
- Slower writes (every INSERT/UPDATE/DELETE must update indexes)
- Storage overhead (indexes can be as large as the table)
- Maintenance overhead (VACUUM, REINDEX)

**Rule of thumb:**
- Small tables (<10,000 rows): Indexes rarely help
- Medium tables (10,000–1,000,000 rows): Index wisely
- Large tables (>1,000,000 rows): Indexes are essential

### Index Selection Strategy

1. **Identify slow queries** (use query logs, monitoring tools)
2. **Analyze execution plans** (`EXPLAIN ANALYZE`)
3. **Look for Seq Scans** on large tables
4. **Create indexes on filter columns** (WHERE, JOIN, ORDER BY)
5. **Test query performance** before and after
6. **Monitor write performance** (indexes slow down INSERTs)

### Real-World Example

Startup with 100M order records:

**Before indexes:**
- Dashboard load: 30 seconds
- API endpoint timeout: 50% of requests

**After strategic indexing:**
```sql
CREATE INDEX idx_orders_customer ON orders(customer_id);
CREATE INDEX idx_orders_date ON orders(order_date);
CREATE INDEX idx_orders_status_date ON orders(status, order_date);
```

**Results:**
- Dashboard load: 500ms
- API timeout rate: 0%
- Write throughput: 95% of before (acceptable trade-off)

### Why This Matters

In data engineering:
- **ETL queries need indexes**: Extracting data from source systems
- **Incremental loads depend on indexes**: "Get records since last run"
- **Join performance matters**: Multi-table analytics queries
- **Data warehouse queries**: Columnar indexes, partitioning strategies

Without proper indexes:
- Queries timeout, pipelines fail
- API endpoints slow down, users complain
- Cloud costs spike (more CPU time for queries)

### Try It

Given this schema:

```sql
CREATE TABLE events (
    event_id BIGSERIAL PRIMARY KEY,
    user_id INT,
    event_type VARCHAR(50),
    event_timestamp TIMESTAMP,
    event_data JSONB
);
```

You frequently run:
```sql
-- Query 1: User activity
SELECT * FROM events
WHERE user_id = 123
AND event_timestamp >= CURRENT_DATE - INTERVAL '7 days';

-- Query 2: Event type analysis
SELECT event_type, COUNT(*)
FROM events
WHERE event_timestamp >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY event_type;

-- Query 3: Recent logins
SELECT * FROM events
WHERE event_type = 'login'
AND event_timestamp >= CURRENT_DATE - INTERVAL '1 hour';
```

What indexes would you create?

<details>
<summary>Solution</summary>

```sql
-- For Query 1: Composite index on user_id and timestamp
CREATE INDEX idx_events_user_timestamp ON events(user_id, event_timestamp);

-- For Query 2: Index on timestamp (covers date filtering)
CREATE INDEX idx_events_timestamp ON events(event_timestamp);

-- For Query 3: Partial index on recent logins
CREATE INDEX idx_events_recent_logins
ON events(event_timestamp)
WHERE event_type = 'login';

-- Alternative for Query 3: Composite if you filter by type frequently
CREATE INDEX idx_events_type_timestamp ON events(event_type, event_timestamp);
```

Trade-offs:
- `idx_events_user_timestamp` helps Query 1 but not Query 2 or 3
- `idx_events_timestamp` helps all queries but less optimized than specific indexes
- Partial index for logins is smaller and faster for Query 3
- Creating all indexes slows down INSERTs—monitor write performance
</details>

## Common Pitfalls

### 1. Over-Normalizing OLAP Systems

**Problem:**
```sql
-- Normalized schema for analytics
-- Every query needs 5+ joins
SELECT ...
FROM fact_sales
JOIN dim_customer ON ...
JOIN dim_product ON ...
JOIN dim_date ON ...
JOIN dim_store ON ...
JOIN dim_promotion ON ...;
```

**Fix:** For analytics (OLAP), denormalized schemas are expected. Use star schemas, not 3NF.

### 2. Creating Too Many Indexes

**Problem:**
```sql
-- 20 indexes on a table with frequent writes
-- Every INSERT updates 20 indexes!
```

**Fix:** Only index columns you actually query. Use `pg_stat_user_indexes` to find unused indexes.

```sql
-- Find unused indexes
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0
AND indexname NOT LIKE '%_pkey';
```

### 3. Wrong Column Order in Composite Indexes

**Problem:**
```sql
CREATE INDEX idx_wrong ON orders(order_date, customer_id);

-- This query won't use the index efficiently
SELECT * FROM orders WHERE customer_id = 123;
```

**Fix:** Put the most selective column first (the one that filters out the most rows).

### 4. Denormalizing Without a Sync Strategy

**Problem:**
```sql
-- Added denormalized column but no trigger/process to keep it updated
ALTER TABLE orders ADD COLUMN customer_name VARCHAR(100);
-- Now customer_name is often NULL or outdated
```

**Fix:** Always implement a sync mechanism (triggers, application code, ETL job) when denormalizing.

## Reflection Questions

1. **How would you decide if denormalization is worth it?**

   Consider:
   - What's the read/write ratio? (99:1 vs. 50:50)
   - How often does the denormalized data change?
   - What's the cost of staleness? (e-commerce prices vs. historical analytics)
   - Can you tolerate eventual consistency?

2. **You have a slow query. Your colleague suggests adding an index. How do you verify this will help?**

   Steps:
   - Run `EXPLAIN ANALYZE` to see current execution plan
   - Check if it's doing a Seq Scan on a large table
   - Create index in a test environment
   - Run `EXPLAIN ANALYZE` again to confirm it uses the index
   - Measure actual query time improvement
   - Check impact on write performance

3. **Your database has 50 tables. How do you decide which ones to normalize vs. denormalize?**

   Framework:
   - Transactional tables (orders, payments): Normalize
   - Reporting tables (dashboards, BI): Denormalize
   - Frequently joined tables: Consider materialized views
   - Rarely queried tables: Don't over-optimize

4. **When would you choose a materialized view over a denormalized table?**

   Materialized view advantages:
   - Automatically refreshable
   - Still maintains normalized sources
   - Can be recreated easily if corrupted

   Denormalized table advantages:
   - More control over sync logic
   - Can have different schema than source
   - Can add additional indexes

## Summary

- **Normalization to 3NF eliminates data anomalies** by organizing data into related tables with foreign key relationships, preventing redundancy and ensuring data integrity.

- **Denormalization is a strategic choice** for read-heavy workloads where performance is critical. Use computed columns, materialized views, or wide tables depending on staleness tolerance and query patterns.

- **Indexes are essential for query performance** on large tables. Create indexes on columns used in WHERE, JOIN, and ORDER BY clauses, but balance read performance against write overhead.

- **OLTP systems use normalized schemas** for transactional consistency, while **OLAP systems use denormalized schemas** for analytical performance. Data engineers bridge these worlds with ETL.

- **Query execution plans (EXPLAIN ANALYZE)** reveal whether queries use indexes effectively. Monitor for Seq Scans on large tables as a sign of missing indexes.

## Next Steps

You now understand how to design efficient database schemas for transactional systems. In Chapter 4, we'll shift to analytical systems—data warehouses. You'll learn:

- The difference between OLTP and OLAP architectures
- Designing star and snowflake schemas for analytics
- BigQuery-specific optimizations (partitioning, clustering)
- Slowly Changing Dimensions (SCD) for tracking historical data

The normalization principles you learned here apply to source systems (OLTP). In the next chapter, you'll intentionally denormalize data for analytics (OLAP). Understanding both paradigms is what makes you an effective data engineer—knowing when to split tables apart and when to join them together.
