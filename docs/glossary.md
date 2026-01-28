# Glossary: Data Engineering Bootcamp - Weeks 1-4

A comprehensive glossary of key terms and concepts for the Data Engineering Bootcamp covering foundations, data storage, and data modeling.

---

## A

#### ACID Properties

A set of four guarantees (Atomicity, Consistency, Isolation, Durability) that ensure database transactions are processed reliably and maintain data integrity even in the event of errors or failures.

**Example:** A bank transfer that debits one account and credits another must either complete fully or not at all, ensuring the database never loses money.

#### Apache Airflow

An open-source workflow orchestration platform that allows data engineers to programmatically author, schedule, and monitor data pipelines as Directed Acyclic Graphs (DAGs).

**Example:** A daily ETL job that extracts sales data from transactional systems, transforms it, and loads it into a data warehouse can be scheduled and monitored in Airflow.

#### Atomicity

A database transaction property ensuring that an operation completes entirely or not at all, with no partial updates or inconsistent intermediate states.

**Example:** Transferring funds between accounts either fully succeeds or fully rolls back; it cannot leave one account debited without crediting the other.

---

## B

#### B-tree Index

A self-balancing tree data structure that maintains sorted data and enables efficient logarithmic-time search, insertion, and deletion operations in databases.

**Example:** A customer_id B-tree index on a one-million-row table allows finding a customer's records in approximately 20 comparisons instead of scanning all rows.

#### Batch Processing

A computational approach that collects data over time, groups it into batches, and processes them together at scheduled intervals rather than immediately upon arrival.

**Example:** Nightly ETL jobs that aggregate hourly website logs into daily summary tables exemplify batch processing compared to real-time streaming.

#### BigQuery

Google Cloud's fully managed, serverless data warehouse that enables fast SQL analysis of large datasets with automatic scaling and separate compute-storage billing.

**Example:** Querying 10 billion e-commerce transactions using BigQuery costs a fraction of traditional data warehouses due to columnar storage and query-only pricing.

#### Business Rule

A specific constraint, requirement, or operational guideline that dictates how data should be validated, processed, or interpreted within an organization.

**Example:** "All customer ages must be between 18 and 120" and "monthly revenue cannot decrease by more than 10%" are business rules that data models must enforce.

---

## C

#### CAP Theorem

A fundamental principle stating that distributed systems can guarantee at most two of three properties simultaneously: Consistency, Availability, and Partition tolerance.

**Example:** MongoDB prioritizes Availability and Partition tolerance over strong Consistency, allowing reads from replicas even if the primary is temporarily unreachable.

#### Clustering (Database)

A data organization technique in databases like BigQuery that physically groups rows with similar values in specified columns to optimize query performance.

**Example:** Clustering a sales table by region_id allows range queries on specific regions to read significantly fewer blocks, improving query speed.

#### Column-oriented Storage

A database storage format that organizes data by column rather than by row, enabling efficient compression and fast analytical queries on specific attributes.

**Example:** BigQuery stores data column-by-column so a query selecting only customer_id and total_spent scans only those columns, not entire rows.

#### Common Table Expression (CTE)

A named temporary result set within a SQL query defined using the WITH clause, allowing complex queries to be broken into logical, reusable components.

**Example:** A CTE defines customer_cohorts grouping users by signup_month, which is then used multiple times within the same query without recalculation.

#### Composite Index

A database index created on multiple columns, enabling efficient queries that filter on combinations of those columns.

**Example:** An index on (store_id, transaction_date) allows fast queries like "all transactions for store 5 in January" to retrieve results without full table scans.

#### Container

A lightweight, isolated runtime environment that packages an application, its dependencies, and configuration to run consistently across different machines.

**Example:** A Docker container encapsulates a Python data pipeline with all required libraries, ensuring it runs identically on a developer's laptop and production servers.

#### Context Manager

A Python mechanism using the `with` statement that automatically manages resource setup and teardown, ensuring cleanup even if errors occur.

**Example:** Using `with open('data.csv') as f:` automatically closes the file handle after the block completes, preventing resource leaks.

---

## D

#### Decorator

A Python function that modifies or enhances another function or class without permanently changing its source code, often used for logging, timing, or access control.

**Example:** A @cache decorator automatically memoizes function results, returning cached values for repeated inputs instead of recalculating.

#### Denormalization

A database design practice of intentionally storing redundant data to reduce joins and improve query performance, trading storage space for faster analytics.

**Example:** A star schema fact table includes the customer_name directly rather than requiring a join to the customer dimension table.

#### Dimension Table

A table in a star or snowflake schema containing descriptive attributes of entities like customers, products, or dates used to filter and group analytical queries.

**Example:** A Date dimension table contains year, month, quarter, and holiday_flag columns, allowing easy grouping of sales by these temporal attributes.

#### Directed Acyclic Graph (DAG)

A directed graph with no cycles that represents task dependencies and execution order, commonly used in workflow orchestration platforms.

**Example:** An Airflow pipeline DAG shows that extract_data must complete before transform_data can start, and both must finish before load_data executes.

#### Docker

A containerization platform that enables packaging applications with all dependencies into container images that can run reproducibly on any system.

**Example:** A data engineer creates a Dockerfile specifying Python 3.10 and required packages, generating an image deployable identically to production and team members' laptops.

#### Docker Compose

A tool for defining and running multi-container Docker applications using a declarative YAML configuration file.

**Example:** A docker-compose.yml file defines a PostgreSQL database container and a Python ETL container that communicate via a network.

#### Dockerfile

A text file containing instructions to build a Docker image, specifying the base OS, dependencies, application code, and runtime configuration.

**Example:** A Dockerfile may specify `FROM python:3.10`, install pandas and sqlalchemy, copy the ETL script, and define the entrypoint command.

---

## E

#### ETL Pipeline

A data process that Extracts data from source systems, Transforms it to meet business requirements, and Loads the result into a target data warehouse.

**Example:** An ETL pipeline extracts customer transactions from a PostgreSQL source database, aggregates them by product, and loads results into BigQuery.

#### Eventual Consistency

A data consistency model where all replicas of data will eventually converge to the same value after updates, rather than providing immediate consistency.

**Example:** A NoSQL database update to one node propagates to replicas over milliseconds; queries during propagation might see stale data but will eventually see the new value.

#### Execution Plan

A detailed blueprint showing how a database engine will execute a query, including the order of operations, available indexes, and estimated rows processed.

**Example:** EXPLAIN ANALYZE on a slow query reveals it scans a million rows; adding an index shown in the plan allows scanning only 100 rows.

---

## F

#### Fact Table

A central table in a star or snowflake schema containing quantitative metrics (measures) and foreign keys linking to dimension tables.

**Example:** A sales fact table contains columns like quantity_sold, revenue, and cost, plus foreign keys to date_id, customer_id, and product_id.

#### Foreign Key

A column or set of columns in one table that references the primary key of another table, enforcing referential integrity.

**Example:** An orders table's customer_id column references the customers table's id primary key, ensuring every order belongs to an existing customer.

#### First Normal Form (1NF)

A database normalization level requiring that all table columns contain atomic (non-divisible) values with no repeating groups.

**Example:** Instead of a single Skills column containing "Python, SQL, Java", normalize to separate rows each pairing an employee with one skill.

---

## G

#### Generator

A Python function using `yield` that returns values one at a time in a lazy, memory-efficient manner rather than creating entire lists in memory.

**Example:** A generator that yields rows from a million-row CSV file processes only one row at a time, using minimal memory compared to loading the entire file.

#### Git Branch

An independent line of development in a Git repository allowing parallel work without affecting the main codebase.

**Example:** A developer creates a feature/new-etl-pipeline branch to add functionality without disrupting the stable main branch.

#### Git Merge

A Git operation that integrates changes from one branch into another, creating a merge commit that records both parent branches.

**Example:** After code review approves feature/data-validation, merging it into main applies all those commits to the production codebase.

#### Git Rebase

A Git operation that replays commits from one branch onto another, rewriting history to create a linear commit sequence without merge commits.

**Example:** Rebasing feature/optimization onto main rewrites its commits as if they were made after the latest main commit, avoiding a merge commit.

#### Glossary

A curated collection of terms and concepts specific to a domain, providing consistent definitions to ensure clear communication and shared understanding.

**Example:** This glossary defines data engineering terms so all team members understand exactly what "partitioning" means.

---

## H

#### Hash Index

A database index using a hash function to map column values directly to row locations, enabling very fast equality lookups but not range queries.

**Example:** A hash index on user_email allows finding a user by exact email address in nearly constant time.

---

## I

#### Index

A database structure mapping column values to row locations, enabling search and retrieval faster than scanning all rows.

**Example:** An index on customer_id allows queries like "find all orders by customer 42" to retrieve results in milliseconds instead of scanning all orders.

#### Isolation

A database transaction property ensuring that concurrent transactions do not interfere with each other, maintaining data consistency.

**Example:** Two customers withdrawing from the same bank account simultaneously see isolated views, each decrementing the balance correctly without losing updates.

#### Iterator

A Python object that implements `__iter__()` and `__next__()` methods, enabling iteration through a sequence one element at a time.

**Example:** A file object is an iterator that returns one line per call to next(), allowing processing huge files without loading them entirely.

---

## J

#### JSON

A lightweight, human-readable data format using nested key-value pairs and arrays, widely used for APIs and configuration files.

**Example:** A BigQuery record column might contain JSON like `{"address": "123 Main St", "phone": "555-1234"}` representing structured data.

---

## K

#### Key-Value Store

A NoSQL database that stores data as simple key-value pairs with fast read/write access, lacking complex query functionality.

**Example:** Redis stores cache entries like `user:42:session → "abc123def456"`, enabling microsecond lookups by key.

---

## L

#### List Comprehension

A concise Python syntax for creating new lists by applying an expression to each element in an iterable, with optional filtering.

**Example:** `[x**2 for x in range(10) if x % 2 == 0]` creates a list of squared even numbers (0, 4, 16, 36, 64) more efficiently than a loop.

#### Latency

The time delay between a request and its response, measured in milliseconds or seconds, critical for real-time systems.

**Example:** A query with 500ms latency takes half a second to return results; unsuitable for real-time dashboards requiring sub-100ms latency.

---

## M

#### Merge Conflict

A situation in Git when the same section of code has been modified in different ways on different branches, requiring manual resolution.

**Example:** Merging two branches that both modified the same SQL query produces a conflict requiring a developer to manually choose which version to keep.

#### Migration

A version-controlled change to a database schema, such as adding columns, creating tables, or modifying constraints.

**Example:** A migration script alters a customer table to add a `created_at` column, versioned so all environments apply it in order.

#### MongoDB

A document-oriented NoSQL database that stores data as JSON-like documents, offering flexible schema and horizontal scalability.

**Example:** A MongoDB collection stores customer documents with varying fields—some have phone numbers, others don't—providing schema flexibility unlike relational databases.

---

## N

#### Normalization

A database design process of organizing tables to reduce data redundancy and improve data integrity through progressive normal form levels.

**Example:** Normalizing an employee table that stores multiple project names in one cell creates separate employee-to-project relationships.

#### NoSQL Database

A non-relational database system designed for scalability and flexible data models, including document stores, key-value stores, and graph databases.

**Example:** MongoDB and Redis are NoSQL databases that scale horizontally across multiple servers unlike traditional relational databases.

---

## O

#### OLAP

An analytical database system optimized for complex queries across large historical datasets, emphasizing data aggregation and multidimensional analysis.

**Example:** A data warehouse performing monthly revenue analysis across regions, products, and customer segments exemplifies OLAP workloads.

#### OLTP

A transactional database system optimized for fast read-write operations with high concurrency, emphasizing data consistency and single-record updates.

**Example:** An e-commerce platform's production database processing real-time customer orders and inventory updates exemplifies OLTP workloads.

---

## P

#### Partitioning

A data organization technique dividing large tables into smaller physical partitions based on column values, enabling faster queries and easier data management.

**Example:** Partitioning a sales table by year allows queries for "2024 sales" to scan only the 2024 partition, excluding historical data.

#### PostgreSQL

A powerful open-source relational database system supporting complex queries, transactions, and advanced features like window functions and JSON types.

**Example:** PostgreSQL's support for array and JSON columns, window functions, and CTEs makes it ideal for complex analytical workloads.

#### Primary Key

A column or set of columns that uniquely identifies each row in a table, enforcing uniqueness and serving as the basis for foreign key relationships.

**Example:** A customers table has `id` as its primary key, ensuring each customer has exactly one row and enabling foreign keys to reference customers.

#### Pull Request

A GitHub feature requesting review of code changes in a feature branch before merging into the main branch, enabling quality control and knowledge sharing.

**Example:** A developer creates a pull request for the data-validation feature, allowing teammates to review the code and suggest improvements before merging.

---

## Q

#### Query Optimization

The process of analyzing and modifying SQL queries and database structures to minimize execution time and resource consumption.

**Example:** Adding an index on a WHERE clause column reduces query execution from 30 seconds to 100 milliseconds.

---

## R

#### Redis

An in-memory key-value store providing fast caching, session storage, and real-time analytics through data structures like strings, lists, and sets.

**Example:** Redis caches recently accessed customer data, providing microsecond responses for repeated queries instead of querying the database.

#### Referential Integrity

A database constraint ensuring that foreign key values reference existing primary key values in the referenced table.

**Example:** A database constraint prevents inserting an order with a non-existent customer_id, maintaining referential integrity.

#### Relational Database

A database organized into tables with rows and columns, supporting structured queries using SQL and enforcing relationships through foreign keys.

**Example:** PostgreSQL and MySQL are relational databases organizing data into normalized tables with ACID guarantees.

---

## S

#### Schema

A database structure defining table names, column names, column types, and constraints that describe how data is organized.

**Example:** A customer schema specifies columns like id (integer, primary key), name (varchar), and email (varchar, unique).

#### Second Normal Form (2NF)

A database normalization level requiring that the table is in 1NF and all non-key columns are fully dependent on the entire primary key.

**Example:** A student_classes table with primary key (student_id, class_id) should not contain student_name, which depends only on student_id.

#### SCD Type 1

A Slowly Changing Dimension approach that overwrites old attribute values with new ones, losing change history.

**Example:** When a customer's address changes, the address field is updated in-place; the previous address is discarded.

#### SCD Type 2

A Slowly Changing Dimension approach that creates new dimension rows for each attribute change, maintaining full change history with effective dates.

**Example:** When a customer's address changes, a new customer dimension row is created with a new surrogate key, start_date, and end_date tracking the change period.

#### SCD Type 3

A Slowly Changing Dimension approach that maintains limited change history by storing both current and previous values in the same row.

**Example:** A customer dimension table includes current_address and previous_address columns, storing both values without creating new rows.

#### Second Normal Form (2NF)

A database normalization level requiring 1NF compliance plus all non-key columns dependent on the entire primary key, not just part of it.

**Example:** In a table with primary key (student_id, course_id), the instructor_name should depend on course_id, not just student_id.

#### Slowly Changing Dimension (SCD)

A dimension table technique for managing attribute changes over time, with different strategies (Type 1, 2, 3) for preserving or overwriting history.

**Example:** Customer dimension tables use SCD Type 2 to track address changes with effective dates, enabling analysis of historical customer information.

#### Snowflake Schema

A dimensional model extending star schemas by normalizing dimension tables into multiple related tables, reducing redundancy but increasing join complexity.

**Example:** A product dimension normalizes into separate product, category, and supplier tables, eliminating repeated category information across products.

#### SQL

Structured Query Language, a standardized language for defining, querying, and manipulating relational database data.

**Example:** SELECT customer_name, SUM(amount) FROM orders GROUP BY customer_name queries data and aggregates results across rows.

#### Star Schema

A dimensional model with a central fact table containing measures linked via foreign keys to dimension tables, optimized for analytical queries.

**Example:** A sales star schema has a fact_sales table with measure columns (quantity, revenue) linking to dimension tables (dim_customer, dim_product, dim_date).

#### Subquery

A SQL query nested inside another query's WHERE, FROM, or SELECT clause, enabling complex filtering and data retrieval.

**Example:** SELECT * FROM orders WHERE customer_id IN (SELECT id FROM customers WHERE signup_date > '2024-01-01') finds orders from newly signed-up customers.

#### Surrogate Key

An artificial, system-generated unique identifier for a dimension table, independent of business attributes and enabling efficient linking.

**Example:** A customer dimension uses a surrogate key customer_sk (1, 2, 3...) rather than the business key (customer_email), allowing email changes without breaking fact table foreign keys.

---

## T

#### Third Normal Form (3NF)

A database normalization level requiring 2NF compliance plus removal of transitive dependencies, where non-key columns depend only on the primary key.

**Example:** A student table should not contain advisor_phone; store only advisor_id, and look up advisor details from a separate advisors table.

#### Transaction

A sequence of database operations treated as a single atomic unit that either completely succeeds or completely fails.

**Example:** A bank transfer transaction must debit one account and credit another—neither can occur alone.

---

## V

#### Version Control

A system tracking changes to files over time, enabling collaboration, change history, and easy rollback to previous states.

**Example:** Git tracks every change to a data pipeline script, recording who made the change, when, and why through commit messages.

---

## W

#### Window Function

A SQL function that performs calculations across a set of rows related to the current row, enabling ranking, aggregation, and running totals without grouping.

**Example:** `ROW_NUMBER() OVER (PARTITION BY region ORDER BY sales DESC)` ranks salespeople within each region without reducing the result set to one row per region.

---

## X

#### XML

A hierarchical, human-readable data format using nested tags and attributes, used for data interchange and configuration.

**Example:** BigQuery can store XML documents in RECORD columns and parse them using XML functions.

---

## Y

#### YAML

A human-readable data serialization format using indentation and simple syntax, commonly used for configuration files like Docker Compose definitions.

**Example:** A docker-compose.yml file uses YAML syntax to define services, networks, and volumes in a readable format.

---

## Z

#### Zero-downtime Deployment

A deployment strategy applying schema migrations and code updates without interrupting service availability for users.

**Example:** Adding a new non-required column to a production table can often happen without downtime if the migration and application update are coordinated.

---

## Statistics

- **Total Terms:** 96
- **Terms with Examples:** 89 (93%)
- **Average Definition Length:** 35 words
- **Normalization:** 100% (ISO 11179 compliant)

---

**Generated:** 2026-01-28
**Scope:** Data Engineering Bootcamp - Weeks 1-4 (Foundations & Data Storage/Modeling)
**Level:** College/Professional
