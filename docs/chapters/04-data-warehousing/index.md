# Chapter 4: Data Warehousing & BigQuery

## Learning Objectives
By the end of this chapter, you will be able to:
1. **Design** dimensional models using star and snowflake schemas for analytical workloads
2. **Evaluate** when to use star vs. snowflake schema based on query patterns and data characteristics
3. **Implement** partitioning and clustering strategies in BigQuery to optimize query performance and costs
4. **Create** Slowly Changing Dimension (SCD) Type 2 implementations to track historical data changes

## Introduction

Your VP of Product walks into your office on Monday morning: "I need a report by Wednesday. Show me our top-selling products by category for Q4, broken down by customer segment and region. Oh, and I need to see how those numbers compare to Q3 and last year."

You open your production PostgreSQL database—the one that powers your e-commerce platform. You write a query. It's running. Still running. Five minutes later, it's still running. You check the database metrics: CPU at 95%, query queue backing up, API response times spiking. Your operations team is paging you about slow checkout times.

You just learned a painful lesson: **operational databases aren't built for analytics**. Your OLTP (Online Transaction Processing) database is optimized for fast INSERTs, UPDATEs, and simple lookups. But analytical queries—the ones that aggregate millions of rows, join across many tables, and compare historical periods—bring it to its knees.

This is why data warehouses exist. They're purpose-built for analytics: denormalized schemas, columnar storage, massive parallelism, and optimizations for read-heavy workloads. In this chapter, you'll learn how to design schemas specifically for analytics, understand the trade-offs between different modeling approaches, and leverage BigQuery's unique features to build cost-effective, high-performance data warehouses.

Let's start with the fundamental question: what makes a warehouse different from a database?

## Section 1: OLTP vs OLAP - Two Different Worlds

### Key Idea
OLTP systems optimize for transactional consistency and fast writes, while OLAP systems optimize for analytical queries and fast reads. The schema design, storage format, and query patterns are fundamentally different.

### Example: The Same Question, Two Systems

**Business Question:** "What were our total sales last month by product category?"

**OLTP Database (PostgreSQL - Normalized Schema):**

```sql
-- Schema: Normalized for transactional integrity
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    status VARCHAR(20)
);

CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT,
    price DECIMAL(10,2)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category_id INT
);

CREATE TABLE categories (
    category_id INT PRIMARY KEY,
    category_name VARCHAR(50)
);

-- Query: Multiple joins, slow on large tables
SELECT
    c.category_name,
    SUM(oi.quantity * oi.price) AS total_sales
FROM order_items oi
JOIN orders o ON oi.order_id = o.order_id
JOIN products p ON oi.product_id = p.product_id
JOIN categories c ON p.category_id = c.category_id
WHERE o.order_date >= '2024-01-01'
    AND o.order_date < '2024-02-01'
    AND o.status = 'completed'
GROUP BY c.category_name;

-- Performance: 15 seconds on 100M rows
-- Impact: Locks tables, slows down production transactions
```

**OLAP Data Warehouse (BigQuery - Star Schema):**

```sql
-- Schema: Denormalized for analytical performance
CREATE TABLE fact_sales (
    sale_id INT64,
    order_date DATE,
    customer_id INT64,
    product_id INT64,
    category_id INT64,
    -- Denormalized attributes for fast filtering
    category_name STRING,
    product_name STRING,
    customer_segment STRING,
    region STRING,
    -- Metrics
    quantity INT64,
    unit_price NUMERIC,
    sales_amount NUMERIC,
    cost_amount NUMERIC,
    profit_amount NUMERIC
)
PARTITION BY order_date
CLUSTER BY category_name, region;

-- Query: No joins needed, blazing fast
SELECT
    category_name,
    SUM(sales_amount) AS total_sales
FROM fact_sales
WHERE order_date >= '2024-01-01'
    AND order_date < '2024-02-01'
GROUP BY category_name;

-- Performance: 800ms on 100M rows
-- Impact: No impact on production system (separate database)
```

### Key Differences

| Aspect | OLTP (Database) | OLAP (Warehouse) |
|--------|----------------|------------------|
| **Purpose** | Run the business | Analyze the business |
| **Workload** | Many small transactions | Few large queries |
| **Schema** | Normalized (3NF) | Denormalized (star/snowflake) |
| **Writes** | Frequent INSERTs/UPDATEs | Batch loads (ETL) |
| **Reads** | Simple lookups (by ID) | Complex aggregations |
| **Storage** | Row-oriented | Column-oriented |
| **Query Pattern** | "Get order #12345" | "Sum all sales by category" |
| **Typical Size** | GB to low TB | TB to PB |
| **Users** | Thousands (customers, app) | Dozens (analysts, executives) |

### Why Column-Oriented Storage Matters

**Row-oriented (OLTP):**
```
Row 1: [order_id=1, customer_id=100, order_date='2024-01-15', amount=50.00]
Row 2: [order_id=2, customer_id=101, order_date='2024-01-16', amount=75.00]
Row 3: [order_id=3, customer_id=100, order_date='2024-01-17', amount=120.00]
```
Reading all `amount` values requires reading all rows entirely.

**Column-oriented (OLAP):**
```
order_id:     [1, 2, 3, ...]
customer_id:  [100, 101, 100, ...]
order_date:   ['2024-01-15', '2024-01-16', '2024-01-17', ...]
amount:       [50.00, 75.00, 120.00, ...]
```
Reading all `amount` values only requires reading the `amount` column.

**Result:** For analytical queries that aggregate columns, columnar storage is 10-100x faster.

### Why This Matters

In data engineering:
- **Don't run analytics on production databases** - it slows down your application
- **Separate OLTP and OLAP systems** - ETL pipelines move data from one to the other
- **Schema design depends on use case** - normalize for transactions, denormalize for analytics

Real-world architecture:
```
OLTP (PostgreSQL)  →  ETL Pipeline (Airflow)  →  OLAP (BigQuery)
   ↓                         ↓                          ↓
Application logic      Transform & load          Analytics queries
Fast writes            Scheduled nightly         Fast aggregations
Normalized schema      Data quality checks       Denormalized schema
```

### Try It

Think about these queries. Which should run on OLTP? Which on OLAP?

1. "Show me order #54321 with all its items"
2. "What's the average order value by day for the past year?"
3. "Update customer #123's email address"
4. "Which products have the highest profit margin by region and quarter?"
5. "Check if product #789 is in stock"

<details>
<summary>Answers</summary>

1. **OLTP** - Simple lookup by primary key
2. **OLAP** - Aggregation over large time range
3. **OLTP** - Transactional update
4. **OLAP** - Complex analytical aggregation
5. **OLTP** - Simple lookup for application logic
</details>

## Section 2: Star Schema - The Foundation of Data Warehousing

### Key Idea
Star schema organizes data into fact tables (metrics/events) and dimension tables (descriptive attributes). This structure optimizes for analytical queries while remaining intuitive and maintainable.

### Example: E-Commerce Analytics Warehouse

**Business Requirement:** "Analyze sales performance across products, customers, time, and locations."

**Star Schema Design:**

```sql
-- FACT TABLE: The center of the star
-- Contains metrics and foreign keys to dimensions
CREATE TABLE fact_sales (
    sale_id INT64,  -- Surrogate key for the fact
    -- Foreign keys to dimensions
    date_key INT64,
    customer_key INT64,
    product_key INT64,
    store_key INT64,
    -- Degenerate dimensions (non-foreign key attributes)
    order_id STRING,
    invoice_number STRING,
    -- Metrics (what we're measuring)
    quantity INT64,
    unit_price NUMERIC,
    discount_amount NUMERIC,
    sales_amount NUMERIC,  -- Extended price
    cost_amount NUMERIC,
    profit_amount NUMERIC
)
PARTITION BY DATE_TRUNC(date_key, MONTH)
CLUSTER BY product_key, store_key;

-- DIMENSION: Time/Date
-- Pre-built calendar with useful attributes
CREATE TABLE dim_date (
    date_key INT64 PRIMARY KEY,  -- 20240115 for Jan 15, 2024
    full_date DATE,
    day_of_week STRING,  -- 'Monday'
    day_of_week_num INT64,  -- 1
    day_of_month INT64,  -- 15
    day_of_year INT64,  -- 15
    week_of_year INT64,  -- 3
    month_num INT64,  -- 1
    month_name STRING,  -- 'January'
    month_abbr STRING,  -- 'Jan'
    quarter INT64,  -- 1
    year INT64,  -- 2024
    is_weekend BOOL,  -- false
    is_holiday BOOL,  -- false
    holiday_name STRING,  -- null
    fiscal_year INT64,  -- 2024
    fiscal_quarter INT64  -- 1
);

-- DIMENSION: Customer
CREATE TABLE dim_customer (
    customer_key INT64 PRIMARY KEY,  -- Surrogate key
    customer_id STRING,  -- Natural key from source system
    customer_name STRING,
    email STRING,
    phone STRING,
    -- Segmentation attributes
    customer_segment STRING,  -- 'Premium', 'Standard', 'Basic'
    customer_type STRING,  -- 'Individual', 'Business'
    signup_date DATE,
    -- SCD Type 2 columns (more on this later)
    effective_date DATE,
    expiration_date DATE,
    is_current BOOL
);

-- DIMENSION: Product
CREATE TABLE dim_product (
    product_key INT64 PRIMARY KEY,
    product_id STRING,
    product_name STRING,
    product_description STRING,
    -- Hierarchy attributes
    category STRING,  -- 'Electronics'
    subcategory STRING,  -- 'Laptops'
    brand STRING,  -- 'Apple'
    -- Product attributes
    color STRING,
    size STRING,
    weight NUMERIC,
    unit_cost NUMERIC,
    unit_price NUMERIC,
    is_active BOOL
);

-- DIMENSION: Store/Location
CREATE TABLE dim_store (
    store_key INT64 PRIMARY KEY,
    store_id STRING,
    store_name STRING,
    -- Geographic hierarchy
    address STRING,
    city STRING,
    state STRING,
    country STRING,
    region STRING,  -- 'West Coast', 'Northeast'
    -- Store attributes
    store_type STRING,  -- 'Flagship', 'Outlet', 'Online'
    square_footage INT64,
    manager_name STRING,
    open_date DATE
);
```

**Why This Design?**

1. **Fact table is long and narrow**: Many rows, focused columns
2. **Dimension tables are short and wide**: Fewer rows, many descriptive attributes
3. **Star shape**: Fact table in the center, dimensions around it (no joins between dimensions)
4. **Queries are simple**: Join fact to dimensions, filter/group by dimension attributes

### Example Queries

**Query 1: Sales by Product Category**
```sql
SELECT
    p.category,
    SUM(f.sales_amount) AS total_sales,
    SUM(f.quantity) AS total_quantity,
    COUNT(DISTINCT f.customer_key) AS unique_customers
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
WHERE f.date_key >= 20240101  -- Jan 1, 2024
    AND f.date_key < 20240201  -- Feb 1, 2024
GROUP BY p.category
ORDER BY total_sales DESC;
```

**Query 2: Sales Trends by Month**
```sql
SELECT
    d.year,
    d.month_name,
    SUM(f.sales_amount) AS monthly_sales,
    AVG(f.profit_amount) AS avg_profit_per_sale
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
GROUP BY d.year, d.month_num, d.month_name
ORDER BY d.year, d.month_num;
```

**Query 3: Top Customers by Region**
```sql
SELECT
    s.region,
    c.customer_name,
    SUM(f.sales_amount) AS total_spent,
    COUNT(DISTINCT f.sale_id) AS order_count
FROM fact_sales f
JOIN dim_customer c ON f.customer_key = c.customer_key
JOIN dim_store s ON f.store_key = s.store_key
WHERE c.customer_segment = 'Premium'
    AND f.date_key >= 20240101
GROUP BY s.region, c.customer_key, c.customer_name
ORDER BY s.region, total_spent DESC;
```

**Query 4: Weekend vs. Weekday Sales**
```sql
SELECT
    d.is_weekend,
    AVG(f.sales_amount) AS avg_transaction_value,
    COUNT(*) AS transaction_count
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
GROUP BY d.is_weekend;
```

### Design Principles

**1. Surrogate Keys**

Use artificial keys instead of natural keys:
```sql
-- Bad: Use source system ID as primary key
CREATE TABLE dim_customer (
    customer_id STRING PRIMARY KEY,  -- What if ID changes in source?
    ...
);

-- Good: Use surrogate key
CREATE TABLE dim_customer (
    customer_key INT64 PRIMARY KEY,  -- Generated in warehouse
    customer_id STRING,  -- Natural key from source
    ...
);
```

**Benefits:**
- Handles source system ID changes
- Supports Slowly Changing Dimensions (SCD)
- Improves join performance (INT vs. STRING)
- Decouples warehouse from source systems

**2. Conformed Dimensions**

Shared dimensions across multiple fact tables:
```sql
-- Same dim_customer used by multiple facts
fact_sales → dim_customer
fact_returns → dim_customer
fact_support_tickets → dim_customer
```

This enables cross-functional analysis: "Which customers buy a lot but also have many support tickets?"

**3. Grain Declaration**

The **grain** is the level of detail in the fact table. Be explicit!

```sql
-- Grain: One row per line item on an order
-- (Order #123 with 3 products = 3 fact rows)
fact_sales (sale_id, date_key, customer_key, product_key, ...)

-- Different grain: One row per order
-- (Order #123 with 3 products = 1 fact row)
fact_orders (order_id, date_key, customer_key, order_total, ...)
```

**Rule:** All facts and dimensions must match the declared grain.

### Why This Matters

Star schema provides:
- **Query performance**: Few joins, columnar scans
- **Simplicity**: Business users can understand the model
- **Flexibility**: Add new dimensions without restructuring facts
- **Consistency**: Conformed dimensions ensure consistent reporting

In data engineering:
- **ETL complexity**: You build pipelines to populate these tables
- **Data quality**: Dimension lookups must succeed (no orphaned facts)
- **Historical tracking**: Dimensions change over time (next section!)

### Try It

Design a star schema for a music streaming service. Requirements:

- Analyze song plays by user, song, time, and device
- Track metrics: play count, duration, skip rate
- Dimensions: user demographics, song attributes, artist info, device type

Sketch out:
1. Fact table name and columns
2. At least 4 dimension tables with key attributes

<details>
<summary>Solution</summary>

```sql
-- FACT: Song plays
CREATE TABLE fact_plays (
    play_id INT64,
    date_key INT64,
    user_key INT64,
    song_key INT64,
    artist_key INT64,
    device_key INT64,
    -- Metrics
    play_duration_seconds INT64,
    was_skipped BOOL,
    was_completed BOOL,
    play_timestamp TIMESTAMP
);

-- DIMENSION: Date (same as before)

-- DIMENSION: User
CREATE TABLE dim_user (
    user_key INT64 PRIMARY KEY,
    user_id STRING,
    username STRING,
    country STRING,
    subscription_type STRING,  -- 'Free', 'Premium'
    signup_date DATE,
    age_group STRING,
    gender STRING
);

-- DIMENSION: Song
CREATE TABLE dim_song (
    song_key INT64 PRIMARY KEY,
    song_id STRING,
    song_title STRING,
    duration_seconds INT64,
    genre STRING,
    subgenre STRING,
    release_date DATE,
    explicit BOOL,
    popularity_score INT64
);

-- DIMENSION: Artist
CREATE TABLE dim_artist (
    artist_key INT64 PRIMARY KEY,
    artist_id STRING,
    artist_name STRING,
    country STRING,
    genre STRING,
    follower_count INT64
);

-- DIMENSION: Device
CREATE TABLE dim_device (
    device_key INT64 PRIMARY KEY,
    device_type STRING,  -- 'Mobile', 'Desktop', 'Smart Speaker'
    os STRING,
    app_version STRING
);
```
</details>

## Section 3: Snowflake Schema vs. Star Schema

### Key Idea
Snowflake schema normalizes dimension tables by splitting them into sub-dimensions. This reduces redundancy but increases query complexity. Choose based on your priorities: storage vs. simplicity.

### Example: Product Dimension Normalization

**Star Schema (Denormalized):**
```sql
CREATE TABLE dim_product (
    product_key INT64 PRIMARY KEY,
    product_id STRING,
    product_name STRING,
    -- Category attributes (repeated for each product)
    category STRING,
    category_description STRING,
    -- Subcategory attributes (repeated)
    subcategory STRING,
    subcategory_description STRING,
    -- Brand attributes (repeated)
    brand STRING,
    brand_country STRING,
    brand_founded_year INT64
);

-- Sample data shows redundancy:
-- product_key | product_name   | category | category_description  | brand | brand_country
-- 1           | iPhone 15      | Electronics | Electronic devices   | Apple | USA
-- 2           | MacBook Pro    | Electronics | Electronic devices   | Apple | USA
-- 3           | AirPods        | Electronics | Electronic devices   | Apple | USA
```

Notice: "Electronics", "Electronic devices", "Apple", "USA" are repeated for every Apple electronics product.

**Snowflake Schema (Normalized):**
```sql
CREATE TABLE dim_product (
    product_key INT64 PRIMARY KEY,
    product_id STRING,
    product_name STRING,
    subcategory_key INT64,  -- Foreign key
    brand_key INT64         -- Foreign key
);

CREATE TABLE dim_subcategory (
    subcategory_key INT64 PRIMARY KEY,
    subcategory STRING,
    subcategory_description STRING,
    category_key INT64  -- Foreign key
);

CREATE TABLE dim_category (
    category_key INT64 PRIMARY KEY,
    category STRING,
    category_description STRING
);

CREATE TABLE dim_brand (
    brand_key INT64 PRIMARY KEY,
    brand STRING,
    brand_country STRING,
    brand_founded_year INT64
);
```

**Visual Comparison:**

Star:
```
fact_sales → dim_product (contains all hierarchy levels)
```

Snowflake:
```
fact_sales → dim_product → dim_subcategory → dim_category
                         ↘ dim_brand
```

### Query Comparison

**Star Schema Query:**
```sql
-- Simple: One join
SELECT
    p.category,
    p.brand,
    SUM(f.sales_amount) AS total_sales
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
GROUP BY p.category, p.brand;
```

**Snowflake Schema Query:**
```sql
-- Complex: Three joins
SELECT
    c.category,
    b.brand,
    SUM(f.sales_amount) AS total_sales
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_subcategory sc ON p.subcategory_key = sc.subcategory_key
JOIN dim_category c ON sc.category_key = c.category_key
JOIN dim_brand b ON p.brand_key = b.brand_key
GROUP BY c.category, b.brand;
```

### Trade-offs

| Aspect | Star Schema | Snowflake Schema |
|--------|-------------|------------------|
| **Query Simplicity** | Simple (few joins) | Complex (many joins) |
| **Query Performance** | Faster (fewer joins) | Slower (more joins) |
| **Storage** | More (redundancy) | Less (normalized) |
| **Data Quality** | Risk of inconsistency | Enforced consistency |
| **ETL Complexity** | Simpler loads | Complex with lookups |
| **User Understanding** | Easier (flatter) | Harder (hierarchical) |

### When to Use Snowflake Schema

**Choose Snowflake when:**
1. **Storage costs are high** and data volume is massive
2. **Dimension hierarchies change frequently** (e.g., org charts)
3. **Strong referential integrity is required**
4. **Multiple fact tables share sub-dimensions**

**Example: Retail Chain with Complex Hierarchies**
```sql
-- Store dimension with geographic hierarchy
fact_sales → dim_store → dim_city → dim_state → dim_country → dim_region

-- Product dimension with category hierarchy
fact_sales → dim_product → dim_subcategory → dim_category → dim_department
```

This prevents updating "California is in USA" across 500 store records.

### Hybrid Approach

In practice, many warehouses use a **hybrid**:

```sql
-- Star for stable dimensions
CREATE TABLE dim_customer (
    customer_key INT64,
    customer_name STRING,
    customer_segment STRING  -- Denormalized
);

-- Snowflake for complex, changing hierarchies
CREATE TABLE dim_product (
    product_key INT64,
    product_name STRING,
    category_key INT64  -- Normalized
);

CREATE TABLE dim_category (
    category_key INT64,
    category_name STRING,
    parent_category_key INT64  -- Self-referencing hierarchy
);
```

### Why This Matters

In data engineering:
- **Star is the default** for most use cases
- **Snowflake for specific needs** (deep hierarchies, storage constraints)
- **Performance trumps theory**: Test with your data and queries

Real-world example: Amazon
- **Star for most dimensions** (customer, date)
- **Snowflake for product categories** (millions of products, complex taxonomies)
- **Performance optimization**: Materialized joins for frequently queried snowflake paths

### Try It

Convert this denormalized dimension to a snowflake schema:

```sql
CREATE TABLE dim_employee (
    employee_key INT64,
    employee_name STRING,
    job_title STRING,
    department STRING,
    department_location STRING,
    division STRING,
    division_head STRING,
    division_budget NUMERIC
);
```

<details>
<summary>Solution</summary>

```sql
CREATE TABLE dim_employee (
    employee_key INT64 PRIMARY KEY,
    employee_name STRING,
    job_title_key INT64,
    department_key INT64
);

CREATE TABLE dim_job_title (
    job_title_key INT64 PRIMARY KEY,
    job_title STRING,
    job_level INT64,
    job_family STRING
);

CREATE TABLE dim_department (
    department_key INT64 PRIMARY KEY,
    department STRING,
    department_location STRING,
    division_key INT64
);

CREATE TABLE dim_division (
    division_key INT64 PRIMARY KEY,
    division STRING,
    division_head STRING,
    division_budget NUMERIC
);
```

Trade-offs:
- **Pros**: No redundancy, update division budget once
- **Cons**: Every employee query needs 3 joins
</details>

## Section 4: BigQuery Optimization - Partitioning and Clustering

### Key Idea
BigQuery's serverless architecture requires different optimization strategies than traditional databases. Partitioning limits the data scanned (reduces cost), while clustering optimizes how data is organized (improves performance).

### Example: The Cost Problem

You have a table with 2 years of daily sales data (700+ days, 100M+ rows):

```sql
CREATE TABLE fact_sales (
    sale_date DATE,
    customer_id INT64,
    product_id INT64,
    store_id INT64,
    sales_amount NUMERIC
);

-- Typical query: Last 7 days of sales
SELECT
    sale_date,
    SUM(sales_amount) AS daily_sales
FROM fact_sales
WHERE sale_date >= CURRENT_DATE() - 7
GROUP BY sale_date;
```

**Without partitioning:**
- BigQuery scans ALL 100M rows (entire table)
- Cost: ~$0.50 per query (at $5/TB)
- Performance: 3-5 seconds

**With date partitioning:**
```sql
CREATE TABLE fact_sales (
    sale_date DATE,
    customer_id INT64,
    product_id INT64,
    store_id INT64,
    sales_amount NUMERIC
)
PARTITION BY sale_date;
```

- BigQuery scans only 7 days of data (~1M rows)
- Cost: ~$0.005 per query (100x cheaper!)
- Performance: 200-500ms (10x faster)

### Partitioning Strategies

**Date/Timestamp Partitioning (Most Common):**
```sql
-- Daily partitions
PARTITION BY sale_date

-- Monthly partitions (better for historical data)
PARTITION BY DATE_TRUNC(sale_date, MONTH)

-- Timestamp partitioning (for event data)
PARTITION BY DATE(event_timestamp)
```

**Integer Range Partitioning:**
```sql
-- Partition by customer ID ranges
PARTITION BY RANGE_BUCKET(customer_id, GENERATE_ARRAY(0, 100000, 1000))
-- Creates partitions: [0-1000), [1000-2000), ...
```

**Ingestion-Time Partitioning:**
```sql
-- Automatic partition based on when data is loaded
PARTITION BY _PARTITIONDATE
```

### Clustering: Optimizing Within Partitions

After partitioning by date, you might still scan millions of rows within a partition. Clustering sorts data within partitions for faster filtering.

```sql
CREATE TABLE fact_sales (
    sale_date DATE,
    customer_id INT64,
    product_id INT64,
    region STRING,
    sales_amount NUMERIC
)
PARTITION BY sale_date
CLUSTER BY region, product_id;
```

**How clustering helps:**

Query filtering by region:
```sql
SELECT SUM(sales_amount)
FROM fact_sales
WHERE sale_date = '2024-01-15'
    AND region = 'West';
```

- **Partitioning** limits scan to one day's data
- **Clustering** within that day, data is sorted by region
- BigQuery skips blocks that don't contain 'West'
- Result: 10-100x less data scanned within the partition

### Clustering Best Practices

**1. Choose high-cardinality columns:**
```sql
-- Good: Many distinct values
CLUSTER BY customer_id, product_id  -- Thousands of each

-- Bad: Few distinct values
CLUSTER BY is_active  -- Only true/false
```

**2. Order by query filter frequency:**
```sql
-- If you filter by region more often than product:
CLUSTER BY region, product_id  -- region first

-- Not: CLUSTER BY product_id, region
```

**3. Limit to 4 columns:**
```sql
-- Good: 2-4 columns
CLUSTER BY region, product_id, store_id

-- Diminishing returns after 4
```

**4. Match common WHERE clauses:**
```sql
-- Common query:
WHERE region = 'West' AND product_category = 'Electronics'

-- Matching clustering:
CLUSTER BY region, product_category
```

### Cost Analysis Example

**Table:** 1 TB of sales data (2 years, 500M rows)

**Query 1: Last 30 days, no filters**
```sql
SELECT * FROM fact_sales
WHERE sale_date >= CURRENT_DATE() - 30;
```

| Configuration | Data Scanned | Cost | Performance |
|---------------|--------------|------|-------------|
| No optimization | 1 TB | $5.00 | 15s |
| Partitioned by date | 41 GB | $0.21 | 3s |
| + Clustered | 41 GB | $0.21 | 3s |

*Clustering doesn't help here (no additional filters).*

**Query 2: Last 30 days, West region**
```sql
SELECT * FROM fact_sales
WHERE sale_date >= CURRENT_DATE() - 30
    AND region = 'West';
```

| Configuration | Data Scanned | Cost | Performance |
|---------------|--------------|------|-------------|
| No optimization | 1 TB | $5.00 | 18s |
| Partitioned by date | 41 GB | $0.21 | 4s |
| + Clustered by region | 8 GB | $0.04 | 800ms |

*Clustering provides 5x cost savings and 5x speed improvement!*

### Partition Expiration

Automatically delete old data to save costs:

```sql
CREATE TABLE fact_sales (
    sale_date DATE,
    ...
)
PARTITION BY sale_date
OPTIONS (
    partition_expiration_days = 730  -- Delete partitions older than 2 years
);
```

### Why This Matters

BigQuery pricing model:
- **$5 per TB scanned** (on-demand)
- Partitioning and clustering directly reduce costs
- A poorly designed table can cost 100x more to query

In data engineering:
- **Design tables for common queries**: Partition and cluster based on WHERE clauses
- **Monitor query costs**: Identify expensive queries and optimize
- **Balance flexibility vs. cost**: More partitions = lower cost but less flexible queries

Real-world example:
- Company with 10 PB warehouse
- Implemented partitioning + clustering
- Reduced monthly query costs from $50k to $5k

### Try It

You have event logs (10 TB, 1 billion rows) with this schema:

```sql
CREATE TABLE event_logs (
    event_timestamp TIMESTAMP,
    user_id INT64,
    event_type STRING,  -- 50 distinct values
    device_type STRING,  -- 10 distinct values
    country STRING,  -- 200 distinct values
    event_data JSON
);
```

Common queries:
1. "Events from last 7 days"
2. "Login events from last 30 days"
3. "All events for user #12345 in the last year"
4. "Events by country for last month"

Design partition and cluster strategy.

<details>
<summary>Solution</summary>

```sql
CREATE TABLE event_logs (
    event_timestamp TIMESTAMP,
    user_id INT64,
    event_type STRING,
    device_type STRING,
    country STRING,
    event_data JSON
)
PARTITION BY DATE(event_timestamp)
CLUSTER BY event_type, country, user_id;
```

**Reasoning:**
- **Partition by date**: All queries filter by time range (7 days, 30 days, 1 year)
- **Cluster by event_type first**: Query 2 filters by event type, most selective
- **Cluster by country second**: Query 4 filters by country
- **Cluster by user_id third**: Query 3 filters by user (though high cardinality makes this less effective)

**Trade-offs:**
- This optimizes for queries 1, 2, and 4
- Query 3 (user-specific) will still be expensive (need to scan many partitions)
- Consider separate table or partition by user_id if query 3 is critical
</details>

## Section 5: Slowly Changing Dimensions (SCD) Type 2

### Key Idea
Dimension attributes change over time. SCD Type 2 tracks historical changes by creating new rows with effective dates, enabling time-travel queries and historical accuracy.

### Example: Customer Address Changes

Your customer moves. How do you handle this in your warehouse?

**Option 1: Overwrite (SCD Type 1) - No History**
```sql
UPDATE dim_customer
SET address = '456 New St',
    city = 'Seattle',
    state = 'WA'
WHERE customer_id = 'C123';
```

**Problem:** Historical sales now show the customer always lived in Seattle, even for orders placed when they lived in New York. Your "sales by region" report is now wrong for historical data.

**Option 2: Add New Row (SCD Type 2) - Full History**
```sql
-- Original row (now expired)
customer_key | customer_id | name  | address      | city     | state | effective_date | expiration_date | is_current
1001         | C123        | Alice | 123 Old St   | New York | NY    | 2020-01-01     | 2024-01-15      | false

-- New row (current)
1005         | C123        | Alice | 456 New St   | Seattle  | WA    | 2024-01-16     | 9999-12-31      | true
```

Now historical sales (before 2024-01-16) correctly show New York, while new sales show Seattle.

### Implementing SCD Type 2

**Schema Design:**
```sql
CREATE TABLE dim_customer (
    customer_key INT64 PRIMARY KEY,  -- Surrogate key (changes for each version)
    customer_id STRING,  -- Natural key (same across versions)
    customer_name STRING,
    email STRING,
    address STRING,
    city STRING,
    state STRING,
    customer_segment STRING,
    -- SCD Type 2 columns
    effective_date DATE,  -- When this version became active
    expiration_date DATE,  -- When this version expired (9999-12-31 for current)
    is_current BOOL  -- TRUE for current version, FALSE for historical
);
```

**Initial Load:**
```sql
INSERT INTO dim_customer VALUES
(1, 'C123', 'Alice', 'alice@email.com', '123 Old St', 'New York', 'NY', 'Premium',
 '2020-01-01', '9999-12-31', TRUE);
```

**Update Process (Customer Moves):**
```sql
-- Step 1: Expire the old row
UPDATE dim_customer
SET expiration_date = '2024-01-15',
    is_current = FALSE
WHERE customer_id = 'C123'
    AND is_current = TRUE;

-- Step 2: Insert new row
INSERT INTO dim_customer VALUES
(2, 'C123', 'Alice', 'alice@email.com', '456 New St', 'Seattle', 'WA', 'Premium',
 '2024-01-16', '9999-12-31', TRUE);
```

### Querying SCD Type 2 Tables

**Current State (Most Common):**
```sql
-- Get current customer info
SELECT *
FROM dim_customer
WHERE customer_id = 'C123'
    AND is_current = TRUE;
```

**Historical State (Time Travel):**
```sql
-- What was Alice's address on Jan 10, 2024?
SELECT *
FROM dim_customer
WHERE customer_id = 'C123'
    AND '2024-01-10' BETWEEN effective_date AND expiration_date;
-- Returns: 123 Old St, New York, NY

-- What was Alice's address on Jan 20, 2024?
SELECT *
FROM dim_customer
WHERE customer_id = 'C123'
    AND '2024-01-20' BETWEEN effective_date AND expiration_date;
-- Returns: 456 New St, Seattle, WA
```

**Historical Fact Joins:**
```sql
-- Sales report with correct historical customer data
SELECT
    f.sale_date,
    c.customer_name,
    c.city AS customer_city_at_time_of_sale,  -- Correct historical city
    f.sales_amount
FROM fact_sales f
JOIN dim_customer c
    ON f.customer_key = c.customer_key  -- Join on surrogate key
WHERE f.sale_date BETWEEN '2024-01-01' AND '2024-01-31';
```

**Note:** The fact table stores `customer_key` (surrogate key), not `customer_id` (natural key). This is crucial for SCD Type 2 to work!

### Which Attributes Should Be SCD Type 2?

**Good candidates:**
- Address, city, state (affects regional analysis)
- Customer segment (affects segmentation analysis)
- Product category (affects category trends)
- Price (affects historical revenue accuracy)

**Bad candidates:**
- Last login date (changes too frequently)
- Transaction count (updated constantly)
- Profile photo URL (not used in analytics)

**Mixed approach:**
```sql
CREATE TABLE dim_customer (
    customer_key INT64,
    customer_id STRING,
    -- SCD Type 2 attributes
    address STRING,
    city STRING,
    state STRING,
    customer_segment STRING,
    effective_date DATE,
    expiration_date DATE,
    is_current BOOL,
    -- SCD Type 1 attributes (just overwrite)
    email STRING,  -- Changes infrequently, history not needed
    phone STRING,
    last_login_date DATE  -- Too frequent to track
);
```

### ETL Pipeline for SCD Type 2

```python
def update_customer_dimension(new_data):
    """
    ETL logic for SCD Type 2 customer dimension.
    """
    for customer in new_data:
        # Get current record
        current = get_current_record(customer['customer_id'])

        if current is None:
            # New customer - insert
            insert_customer(customer, is_current=True)

        else:
            # Existing customer - check if attributes changed
            if customer_changed(current, customer):
                # Expire old record
                expire_record(current['customer_key'])

                # Insert new record
                insert_customer(customer, is_current=True)

            else:
                # No change - optionally update Type 1 attributes
                update_non_tracked_attributes(current['customer_key'], customer)

def customer_changed(current, new):
    """
    Check if Type 2 attributes have changed.
    """
    type2_attributes = ['address', 'city', 'state', 'customer_segment']
    for attr in type2_attributes:
        if current[attr] != new[attr]:
            return True
    return False
```

### Why This Matters

SCD Type 2 enables:
- **Historical accuracy**: Reports reflect data as it was at the time
- **Trend analysis**: Track how customers/products change over time
- **Compliance**: Audit trails for regulatory requirements
- **Point-in-time queries**: "What did our customer base look like last year?"

In data engineering:
- **ETL complexity increases**: Need to detect changes and manage versions
- **Storage increases**: Multiple versions of same entity
- **Query patterns change**: Must join on surrogate keys, not natural keys

Real-world example: E-commerce
- Product price changes (SCD Type 2): Historical orders show correct price
- Customer upgrades to Premium (SCD Type 2): Analyze premium customers over time
- Product name typo fix (SCD Type 1): Just overwrite, history doesn't matter

### Try It

You have a product dimension. Products can change category and price. Design an SCD Type 2 solution.

Requirements:
- Track category changes (important for trend analysis)
- Track price changes (important for revenue accuracy)
- Don't track description changes (just overwrite)

<details>
<summary>Solution</summary>

```sql
CREATE TABLE dim_product (
    product_key INT64 PRIMARY KEY,
    product_id STRING,
    product_name STRING,
    -- SCD Type 2 tracked attributes
    category STRING,
    price NUMERIC,
    effective_date DATE,
    expiration_date DATE,
    is_current BOOL,
    -- SCD Type 1 attributes (overwrite)
    description STRING,
    image_url STRING,
    last_updated TIMESTAMP
);

-- Example: Price change from $29.99 to $34.99 on Jan 15, 2024

-- Before (current row):
-- product_key | product_id | name         | category | price | effective  | expiration | is_current
-- 101         | P001       | Python Book  | Books    | 29.99 | 2023-01-01 | 9999-12-31 | true

-- After (two rows):
-- 101         | P001       | Python Book  | Books    | 29.99 | 2023-01-01 | 2024-01-14 | false
-- 105         | P001       | Python Book  | Books    | 34.99 | 2024-01-15 | 9999-12-31 | true

-- Historical query: Sales before price change
SELECT SUM(f.sales_amount)
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
WHERE p.product_id = 'P001'
    AND f.sale_date < '2024-01-15';
-- Uses product_key 101 (old price $29.99)

-- Historical query: Sales after price change
SELECT SUM(f.sales_amount)
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
WHERE p.product_id = 'P001'
    AND f.sale_date >= '2024-01-15';
-- Uses product_key 105 (new price $34.99)
```
</details>

## Common Pitfalls

### 1. Querying Fact with Natural Key Instead of Surrogate Key

**Problem:**
```sql
-- This loses historical accuracy!
SELECT SUM(sales_amount)
FROM fact_sales f
JOIN dim_customer c ON f.customer_id = c.customer_id  -- Natural key
WHERE c.is_current = TRUE;
```

**Fix:** Always join on surrogate keys in fact tables:
```sql
SELECT SUM(sales_amount)
FROM fact_sales f
JOIN dim_customer c ON f.customer_key = c.customer_key;  -- Surrogate key
```

### 2. Not Choosing Partition Key Based on Query Patterns

**Problem:**
```sql
-- Partitioned by customer_id, but you query by date
PARTITION BY customer_id

SELECT * FROM fact_sales
WHERE sale_date >= '2024-01-01';  -- Full scan!
```

**Fix:** Partition by the column you filter most often (usually date).

### 3. Over-Clustering

**Problem:**
```sql
-- Too many clustering columns (diminishing returns)
CLUSTER BY col1, col2, col3, col4, col5, col6, col7
```

**Fix:** Limit to 2-4 high-cardinality columns that match WHERE clauses.

### 4. Not Handling Dimension Load Order

**Problem:**
```sql
-- Load facts before dimensions
INSERT INTO fact_sales (..., product_key = 999);
-- Product 999 doesn't exist in dim_product yet!
```

**Fix:** Always load dimensions before facts in your ETL pipeline.

## Reflection Questions

1. **When would you choose snowflake over star schema?**

   Consider:
   - How complex are your dimension hierarchies?
   - How often do hierarchy definitions change (e.g., org restructuring)?
   - Are storage costs a major concern?
   - How comfortable are your analysts with complex SQL?

2. **A BigQuery table costs $100/day to query. How would you optimize it?**

   Steps:
   - Analyze query logs to find expensive queries
   - Check if table is partitioned (add partition if not)
   - Check if queries filter on high-cardinality columns (add clustering)
   - Consider materialized views for repeated aggregations
   - Educate users on query best practices (avoid SELECT *)

3. **You need to track product price changes historically. Would you use SCD Type 1, Type 2, or a separate price history table?**

   SCD Type 1: If you only care about current price, no historical analysis
   SCD Type 2: If you need correct historical revenue in reports
   Separate table: If prices change very frequently (e.g., dynamic pricing)

4. **Your star schema has 20 dimensions. Is this a problem?**

   Consider:
   - How many dimensions does a typical query join? (If 3-4, it's fine)
   - Are some dimensions rarely used? (Consider removing or separating)
   - Are some dimensions really attributes of other dimensions? (Snowflake candidate)
   - Does query performance meet requirements? (If yes, don't over-optimize)

## Summary

- **OLTP and OLAP serve different purposes**: Transactional systems use normalized schemas for data integrity, while analytical systems use denormalized schemas for query performance.

- **Star schema is the foundation** of dimensional modeling, with a central fact table surrounded by dimension tables. It provides simplicity, performance, and flexibility for analytics.

- **Snowflake schema normalizes dimensions**, reducing redundancy at the cost of query complexity. Use it for complex hierarchies or storage constraints, but default to star schema for most cases.

- **BigQuery partitioning reduces costs** by limiting data scanned, while **clustering improves performance** by organizing data within partitions. Design based on common query patterns.

- **SCD Type 2 tracks historical changes** by creating new dimension rows with effective dates. Essential for accurate historical reporting but increases ETL complexity and storage.

- **Surrogate keys enable SCD Type 2** and decouple the warehouse from source system changes. Always use surrogate keys in dimension tables.

## Next Steps

Congratulations! You've completed the foundations of data engineering (Weeks 1-4). You now understand:

- Python and SQL for data processing
- Git and Docker for collaboration and deployment
- Database design and normalization
- Data warehouse modeling and optimization

In Weeks 5-8, you'll build on these foundations with:
- **ETL pipeline development** with Airflow
- **Data quality frameworks** and testing
- **Streaming data** with Kafka
- **Cloud data platforms** (Snowflake, Databricks)

The concepts from these first four chapters—scalable code patterns, collaborative workflows, intentional schema design, and optimization strategies—will be essential as you tackle more complex data engineering challenges. You're ready to build production pipelines that scale!
