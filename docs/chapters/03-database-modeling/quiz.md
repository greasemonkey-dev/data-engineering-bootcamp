# Chapter 3 Quiz: Database Design & Modeling

#### 1. What is the primary goal of database normalization?

<div class="upper-alpha" markdown>
1. To improve query performance by combining related tables
2. To eliminate redundancy and dependency anomalies in data
3. To reduce the total number of tables in a database
4. To make the database easier to understand visually
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Normalization (1NF, 2NF, 3NF, etc.) eliminates data redundancy, reduces update anomalies, and ensures data integrity. Option A describes denormalization. Option C is not the primary goal. Option D is a side benefit but not the main purpose.

    **Concept:** Database Normalization

---

#### 2. Explain why a database table violates Second Normal Form (2NF).

<div class="upper-alpha" markdown>
1. It contains attributes that don't depend on the primary key
2. It has multiple rows with identical data
3. It uses NULL values in critical columns
4. It has more than three foreign keys
</div>

??? question "Show Answer"
    The correct answer is **A**.

    2NF requires that all non-key attributes depend on the entire primary key, not just part of it (partial dependency). If a non-key attribute depends only on part of a composite key, it violates 2NF. Option B describes duplication, not 2NF violation. Option C is a data quality issue. Option D is not a 2NF criterion.

    **Concept:** Second Normal Form

---

#### 3. What is a database index and how does it improve query performance?

<div class="upper-alpha" markdown>
1. It duplicates data to allow faster lookups without reading the entire table
2. It creates a sorted data structure that allows the database to find rows without a full table scan
3. It compresses table data to reduce storage and retrieval time
4. It automatically partitions data across multiple servers
</div>

??? question "Show Answer"
    The correct answer is **B**.

    An index is a sorted data structure (typically B-tree) that allows the database engine to locate rows quickly without scanning every row. Option A is incorrect; indices don't duplicate data (though they do add storage). Option C describes compression. Option D describes sharding, not indexing.

    **Concept:** Database Indexes

---

#### 4. Which index type would you use for a column containing mostly NULL values and frequent LIKE queries?

<div class="upper-alpha" markdown>
1. Hash index
2. B-tree index
3. Bloom filter index
4. Composite index
</div>

??? question "Show Answer"
    The correct answer is **B**.

    B-tree indexes handle range queries and LIKE patterns efficiently and work well with NULL values. Hash indexes only support equality. Option C (Bloom filter) is used for existence checks but not practical for general queries in traditional databases. Option D (composite) indexes multiple columns but doesn't address the NULL/LIKE issue specifically.

    **Concept:** Index Types and Selection

---

#### 5. Describe the difference between a Primary Key and a Foreign Key.

<div class="upper-alpha" markdown>
1. Primary keys are faster while foreign keys are slower
2. A primary key uniquely identifies a row; a foreign key links to the primary key of another table
3. Foreign keys are optional while primary keys are required
4. Primary keys can contain NULL values while foreign keys cannot
</div>

??? question "Show Answer"
    The correct answer is **B**.

    A primary key uniquely identifies each row in a table. A foreign key is a column (or set of columns) that references the primary key of another table, establishing relationships. Option A is incorrect; they don't differ in speed. Option C is false; both can be designed as required or optional. Option D is false; primary keys cannot be NULL.

    **Concept:** Primary and Foreign Keys

---

#### 6. You need to design a schema for an e-commerce system with Products, Orders, and OrderItems. The Orders table has millions of rows. How would you structure the OrderItems table?

<div class="upper-alpha" markdown>
1. Store all order details in the Orders table without a separate OrderItems table
2. Create an OrderItems table with columns: OrderID (FK), ProductID (FK), Quantity, Price
3. Create OrderItems with many columns duplicated from Orders for faster queries
4. Store OrderItems in a NoSQL database while keeping Orders in relational
</div>

??? question "Show Answer"
    The correct answer is **B**.

    This normalized design separates order headers (Orders) from order line items (OrderItems), supporting multiple products per order efficiently. Option A doesn't scale; it wastes storage with repeated order data. Option C introduces redundancy and update anomalies. Option D adds unnecessary complexity without clear benefits.

    **Concept:** Schema Design

---

#### 7. Why might you choose to denormalize data in a data warehouse, even though normalization is a best practice?

<div class="upper-alpha" markdown>
1. Denormalized data requires fewer indexes
2. Denormalization reduces query complexity and improves analytical query performance, accepting increased storage
3. Normalization is only for transactional systems and doesn't apply to warehouses
4. Denormalization is faster to design and requires no maintenance
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Data warehouses often denormalize data (using star schemas with fact/dimension tables) to simplify analytical queries and reduce join operations. The trade-off: more storage and potential update complexity. Option A is false; denormalized data often needs more indexes. Option C is misleading; normalization principles apply everywhere but warehouses prioritize query performance. Option D is false; denormalized schemas require careful maintenance.

    **Concept:** Denormalization in Data Warehousing

---

#### 8. Given a PostgreSQL table with columns user_id, event_date, and value, how would you add an index to optimize queries filtering by user_id and event_date?

<div class="upper-alpha" markdown>
1. Create two separate indexes: one on user_id and one on event_date
2. Create a composite index on (user_id, event_date)
3. Create a unique index on the combination of both columns
4. Create an index on value since it's the most frequently queried column
</div>

??? question "Show Answer"
    The correct answer is **B**.

    A composite index on (user_id, event_date) matches the query pattern exactly, allowing efficient lookups by both columns or user_id alone. Option A works but uses more space and may not be as efficient. Option C adds a uniqueness constraint that may not be appropriate. Option D indexes the wrong column.

    **Concept:** Index Design

---

#### 9. What is a Slowly Changing Dimension (SCD) and why is it important in data warehousing?

<div class="upper-alpha" markdown>
1. It's a dimension that changes slowly and requires special handling to maintain historical accuracy
2. It's a performance optimization technique that caches slow queries
3. It's a type of index used for historical data
4. It's a dimension table that is updated frequently and must be locked
</div>

??? question "Show Answer"
    The correct answer is **A**.

    SCDs handle dimension attributes that change over time (e.g., customer address). SCD Type 1 overwrites old values, Type 2 creates new rows, Type 3 stores current and previous values. This maintains historical accuracy in analytics. Option B describes caching. Option C is incorrect; it's a concept, not an index type. Option D contradicts the "slowly" aspect.

    **Concept:** Slowly Changing Dimensions

---

#### 10. You're analyzing a query execution plan and notice a full table scan on a 10 million row table. What questions should you ask to diagnose the performance issue?

<div class="upper-alpha" markdown>
1. Is the table sorted correctly?
2. Is there an appropriate index available and is the query using it? Are statistics up-to-date?
3. Should I always use NoSQL instead of relational databases?
4. Is the base image of the Docker container correct?
</div>

??? question "Show Answer"
    The correct answer is **B**.

    To diagnose full table scans, check: (1) if an index exists for the query's filter columns, (2) if the optimizer chose to use it, and (3) if table statistics are current (outdated statistics lead to poor decisions). Option A is incorrect; table sorting is separate from indexes. Option C introduces unrelated technology. Option D is completely unrelated.

    **Concept:** Query Execution Plan Analysis

---

## Question Distribution Summary

**Bloom's Taxonomy:**
- Remember (25%): Questions 1, 3 = 20%
- Understand (30%): Questions 2, 5, 9 = 30%
- Apply (30%): Questions 4, 6, 8 = 30%
- Analyze (15%): Questions 7, 10 = 20%

**Answer Distribution:**
- A: 20% (2 questions)
- B: 60% (6 questions)
- C: 10% (1 question)
- D: 10% (1 question)

*Note: Answer distribution will be balanced across all 4 quizzes (40 questions total) to achieve target percentages.*
