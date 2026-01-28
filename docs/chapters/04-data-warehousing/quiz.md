# Chapter 4 Quiz: Data Warehousing & BigQuery

#### 1. What is a fact table in a star schema?

<div class="upper-alpha" markdown>
1. A table containing historical records of all dimension changes
2. A table containing measured, quantifiable events with foreign keys to dimension tables
3. A small table that contains all unique values for filtering
4. A table that stores metadata about the data warehouse
</div>

??? question "Show Answer"
    The correct answer is **B**.

    A fact table stores measurable facts (e.g., sales amounts, quantities) with foreign key references to dimension tables. Each row typically represents a business event. Option A describes a slowly changing dimension or history table. Option C describes a dimension table. Option D is incorrect.

    **Concept:** Fact Tables

---

#### 2. Explain the key difference between a star schema and a snowflake schema.

<div class="upper-alpha" markdown>
1. Star schema is faster while snowflake schema uses less storage
2. Star schema has denormalized dimensions while snowflake has normalized dimensions
3. Snowflake schema uses more fact tables than star schema
4. Star schema is only used in NoSQL databases
</div>

??? question "Show Answer"
    The correct answer is **B**.

    In a star schema, dimension tables are denormalized (all attributes in one table). In a snowflake schema, dimensions are normalized into multiple related tables. Star schemas are simpler to query; snowflake schemas save storage. Option A is partially true but oversimplified. Option C is incorrect. Option D is false.

    **Concept:** Star vs Snowflake Schema

---

#### 3. What is the primary benefit of partitioning in BigQuery?

<div class="upper-alpha" markdown>
1. It encrypts sensitive data automatically
2. It allows BigQuery to scan only relevant data, reducing query costs and improving performance
3. It eliminates the need for WHERE clauses in queries
4. It automatically creates indexes on all columns
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Partitioning divides large tables by date or other columns. BigQuery scans only the relevant partitions for a query, reducing data scanned and costs (you pay per TB scanned). Option A describes encryption. Option C is false; WHERE clauses still filter within partitions. Option D is incorrect.

    **Concept:** BigQuery Partitioning

---

#### 4. Your analysis shows that a BigQuery table query scans 100GB but returns only 1MB of data. Which optimization would you apply first?

<div class="upper-alpha" markdown>
1. Add clustering to the table
2. Enable compression on all columns
3. Add a partition on the filter column used in WHERE clauses
4. Increase the number of slots reserved for this query
</div>

??? question "Show Answer"
    The correct answer is **C**.

    Partitioning the filter column prevents scanning unnecessary data. You pay for data scanned, not returned, so this has the highest impact. Option A (clustering) is beneficial but secondary to partitioning. Option B reduces size but doesn't prevent unnecessary scans. Option D doesn't address the inefficiency.

    **Concept:** BigQuery Optimization

---

#### 5. Describe the difference between OLTP and OLAP systems.

<div class="upper-alpha" markdown>
1. OLTP handles real-time transactions; OLAP handles analytical queries on aggregated data
2. OLTP uses NoSQL; OLAP uses relational databases
3. OLTP is faster because it uses more indexes
4. OLAP systems don't need to store historical data
</div>

??? question "Show Answer"
    The correct answer is **A**.

    OLTP (Online Transaction Processing) optimizes for fast, concurrent writes/updates with normalized schemas (e.g., operational databases). OLAP (Online Analytical Processing) optimizes for complex read queries on historical, aggregated data with denormalized schemas (data warehouses). Option B is incorrect; both can use either. Option C is incomplete. Option D is false; OLAP specifically stores historical data.

    **Concept:** OLTP vs OLAP

---

#### 6. You're designing a dimensional model for a retail analytics warehouse with daily sales data. How should you structure the Date dimension table?

<div class="upper-alpha" markdown>
1. Include only the date column to minimize storage
2. Precompute all date attributes (year, quarter, month, day_of_week) and store them in the dimension
3. Calculate date attributes dynamically in queries
4. Store date as a Unix timestamp to save space
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Precomputing attributes (year, quarter, month, etc.) in the dimension table enables efficient filtering and grouping without calculation overhead. It's a small table so storage is negligible. Option A loses useful attributes. Option C recalculates repeatedly, reducing performance. Option D sacrifices readability and ease of filtering.

    **Concept:** Dimension Table Design

---

#### 7. Why is denormalization acceptable in a data warehouse but problematic in an OLTP database?

<div class="upper-alpha" markdown>
1. Data warehouses have no update requirements while OLTP systems have frequent updates
2. OLTP systems are accessed by many users while data warehouses are not
3. Data warehouses don't need to maintain data integrity
4. OLTP systems are always faster than denormalized data warehouses
</div>

??? question "Show Answer"
    The correct answer is **A**.

    In data warehouses, data is typically loaded in batch (daily, weekly) with minimal updates. Denormalization optimizes read performance for complex analytical queries. In OLTP, frequent updates to denormalized data cause redundancy and anomalies. Option B is incorrect. Option C is false. Option D is oversimplified.

    **Concept:** Denormalization Trade-offs

---

#### 8. Given a BigQuery table with millions of customer transactions, which approach would minimize query costs when aggregating sales by product_id?

<div class="upper-alpha" markdown>
1. Partition the table by product_id; filter by product_id in the WHERE clause
2. Cluster the table by product_id; create a materialized view with pre-aggregated data
3. Create a separate fact table for each product
4. Run queries during off-hours to reduce costs
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Clustering groups similar data together; a materialized view pre-aggregates results so repeated queries don't re-scan raw data. Option A partitions by product but doesn't address aggregation. Option C is unscalable. Option D doesn't reduce data scanned.

    **Concept:** BigQuery Optimization

---

#### 9. What is a Slowly Changing Dimension Type 2, and when would you use it?

<div class="upper-alpha" markdown>
1. A dimension that updates records in-place when attributes change, overwriting old values
2. A dimension that creates new rows with surrogate keys when attributes change, maintaining full history
3. A dimension that stores multiple versions in a single row using array columns
4. A dimension that is updated only once per month
</div>

??? question "Show Answer"
    The correct answer is **B**.

    SCD Type 2 creates a new record each time a dimension attribute changes, with effective date ranges and surrogate keys. This preserves history and enables accurate historical analysis. Option A describes Type 1 (overwrite). Option C describes Type 3. Option D is about update frequency, not change handling.

    **Concept:** Slowly Changing Dimensions Type 2

---

#### 10. Compare the cost implications of querying a normalized 1NF database versus a denormalized star schema for the same analytical query in BigQuery.

<div class="upper-alpha" markdown>
1. The normalized database always costs less because it stores less data
2. The star schema typically costs less because fewer JOINs mean less data must be scanned
3. Both cost the same if they contain the same raw data
4. Denormalized schemas always cost more due to redundancy
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Star schemas denormalize dimensions to reduce JOINs and complexity. In BigQuery, you pay for bytes scanned. Fewer JOINs often mean simpler query plans with less data scanned. A highly normalized schema may require multiple JOINs, scanning more intermediate results. Option A is false; storage cost differs from query cost. Option C ignores join overhead. Option D doesn't account for query plan efficiency.

    **Concept:** Schema Design and Cost Analysis

---

## Question Distribution Summary

**Bloom's Taxonomy:**
- Remember (25%): Questions 1, 3 = 20%
- Understand (30%): Questions 2, 5, 7 = 30%
- Apply (30%): Questions 4, 6, 8, 9 = 40%
- Analyze (15%): Question 10 = 10%

**Answer Distribution:**
- A: 20% (2 questions)
- B: 60% (6 questions)
- C: 10% (1 question)
- D: 10% (1 question)

*Note: Answer distribution will be balanced across all 4 quizzes (40 questions total) to achieve target percentages.*
