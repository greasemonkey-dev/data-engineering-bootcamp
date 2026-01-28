# Frequently Asked Questions - Data Engineering Bootcamp Weeks 1-4

## Conceptual Clarifications

### What is the difference between OLTP and OLAP systems?

OLTP (Online Transaction Processing) systems are optimized for frequent, fast write and update operations like processing customer orders or updating inventory in real-time. OLAP (Online Analytical Processing) systems are designed for complex, read-heavy analytical queries across large datasets, prioritizing query speed over write performance. Data engineers typically extract data from OLTP systems (e.g., PostgreSQL for an e-commerce platform) and load it into OLAP systems (e.g., BigQuery data warehouse) for analytics. See Week 3 for detailed comparison and use cases.

### How do star schemas differ from snowflake schemas?

Star schemas have a single fact table at the center connected directly to denormalized dimension tables, creating a simple, flat structure that speeds up queries and reduces joins. Snowflake schemas normalize dimension tables into multiple related tables, consuming more storage and requiring additional joins but reducing data redundancy. Use star schemas when query performance is critical and storage costs are low (typical in data warehouses); use snowflake schemas when storage is constrained or dimension tables are shared across multiple fact tables. See Week 3 for architectural examples.

### What's the difference between normalization and denormalization?

Normalization organizes data to eliminate redundancy and ensure data consistency, critical for transactional systems (OLTP) where multiple writes could otherwise create conflicts. Denormalization intentionally adds redundancy by combining related data into single tables or repeating columns, optimizing for query performance in analytical systems (OLAP). A normalized e-commerce database stores customer info once; a denormalized warehouse might repeat customer city in every order row to avoid joins. See Week 4 for normalization principles (1NF, 2NF, 3NF) and when to apply each approach.

### What's the difference between list comprehensions and generators in Python?

List comprehensions create the entire list in memory immediately, using [x for x in range(1000000)], making them fast for small datasets but memory-intensive for large ones. Generators produce values one at a time on demand using (x for x in range(1000000)), consuming minimal memory but slightly slower per-iteration. For data engineering, use generators when processing massive files or streaming data; use list comprehensions when you need immediate access to all values. See Week 1 for implementation examples in data processing scripts.

### How do indexes improve database query performance?

An index is a separate data structure (typically B-tree) that maps column values to row locations, allowing the database to find matching rows without scanning every row. Without an index on a "customer_id" column in a table with 100 million rows, the database must check all 100 million rows (slow); with an index, it jumps directly to matching rows (fast). Indexes trade write speed and storage for read speed, ideal for columns frequently used in WHERE clauses or JOINs. See Week 4 for composite indexes and execution plan analysis.

### What's the difference between git merge and git rebase?

Git merge combines two branches by creating a new "merge commit" that points to both parent commits, preserving complete history and making it clear when branches diverged. Git rebase replays one branch's commits on top of another, creating a linear history without merge commits, but losing visibility of when branches were created. Use merge in shared repositories (safer, easier to understand conflicts) and rebase in feature branches (cleaner history). See Week 2 for Git workflows and team collaboration best practices.

### What are context managers and why are they important for data engineering?

Context managers (using the `with` statement) ensure resources like file handles, database connections, or locks are properly acquired and released, even if errors occur. Instead of manually opening and closing a file, `with open('file.csv') as f: process(f)` guarantees the file closes automatically. Data engineering scripts often process large files or query massive databases, making context managers essential for preventing resource leaks. See Week 1 for context manager patterns in ETL scripts.

### How does Docker relate to virtual machines?

Docker containers are lightweight, isolated environments that share the host OS kernel, while virtual machines include an entire OS, making them larger and slower. Docker is ideal for data engineering pipelines because you can package Python, PostgreSQL, and your code together, ensuring it runs identically on your laptop, a colleague's machine, and production servers. Use containers to version entire development environments. See Week 2 for Docker fundamentals and practical container examples.

## Common Misconceptions

### Do I always need to normalize my database to 3NF?

No—normalization is a tool designed for transactional consistency, not a universal requirement. OLTP systems (banks, inventory) must normalize to prevent data anomalies when multiple concurrent writes happen, while OLAP systems (data warehouses) intentionally denormalize to reduce joins and speed up analytical queries. A data warehouse fact table might contain customer city repeated in millions of rows; this is a design choice, not a mistake. See Week 4 for normalization principles and when to denormalize intentionally.

### Is a NoSQL database always faster than a relational database?

No—speed depends entirely on your access patterns. NoSQL databases (MongoDB, Redis) excel at horizontal scaling and flexible schemas, but relational databases (PostgreSQL) are optimized for complex multi-table queries and strong consistency. A MongoDB query that scans millions of documents can be slower than a PostgreSQL query using indexes, and PostgreSQL can handle billions of rows efficiently. Choose based on your data shape and query patterns, not blanket assumptions. See Week 3 for comparison frameworks.

### Should I always use git rebase instead of merge?

No—while rebase creates cleaner linear history, merge is safer for shared branches where multiple people commit simultaneously. Rebasing public branches can cause conflicts for teammates; merging is more collaborative. Use rebase on feature branches you own, and merge when integrating into shared branches like main. See Week 2 for team Git workflows that mix both strategies.

### Do I need to denormalize immediately to improve query performance?

Not necessarily—before denormalizing, first optimize with indexes, better SQL queries (window functions, CTEs), and query execution plan analysis. Many "slow" queries are actually slow queries, not slow schemas. Denormalization is a last resort after exhausting indexing and query optimization. See Week 4 for the optimization process: 1) Analyze execution plans, 2) Add indexes, 3) Rewrite queries, 4) Then denormalize if needed.

### Is it true that generators always consume less memory than list comprehensions?

Generators do consume less memory for large datasets, but they have trade-offs: you can only iterate once, they're slower per-item, and some operations (like sorting) force them into memory anyway. Generators are perfect for streaming data through a pipeline; list comprehensions are better when you need to access elements multiple times. See Week 1 for performance comparisons in realistic data processing scenarios.

### Will Docker containers always make my application portable across different systems?

Docker containers are portable across systems running Docker, but not without limitations: Windows container images won't run natively on Linux (though WSL 2 helps), ARM containers won't run on x86 hardware, and GPU support requires nvidia-docker. Docker solves the "works on my machine" problem for development environments and Linux servers, the most common data engineering scenario. See Week 2 for real-world container deployment considerations.

## Practical Applications

### When should I use PostgreSQL vs MongoDB for a data engineering project?

Use PostgreSQL when you have well-defined schemas, need complex multi-table queries (JOINs), and require strong consistency (financial transactions, inventory systems). Use MongoDB when you have flexible, evolving schemas, deeply nested data (e.g., user profiles with varying attributes), or need easy horizontal scaling for write-heavy workloads. For most data engineering pipelines at Weeks 1-4 level, PostgreSQL is the default choice due to its mature ecosystem and SQL compatibility. See Week 3 for detailed comparison and real-world examples (e-commerce vs user-generated content platforms).

### How do I decide whether to partition my BigQuery tables?

Partition BigQuery tables when they're larger than 1 GB and commonly filtered by specific columns (date, region, customer_id), as partitioning drastically reduces data scanned and costs. Partition by ingestion time (default) for time-series data, by a date column for historical snapshots, or by categorical columns for logical groupings. Avoid partitioning small tables or columns with very high cardinality (millions of unique values). See Week 3 for BigQuery architecture and cost optimization examples.

### When should I use a context manager in Python for data processing?

Use context managers whenever you work with resources that must be cleaned up: opening files (`with open()`), connecting to databases (`with connection()`), acquiring locks, or managing temporary resources. In data engineering, context managers prevent file handle leaks in long-running ETL jobs and ensure database connections close even if exceptions occur. See Week 1 for patterns in production data pipelines.

### How do I choose between star schema and snowflake schema for my data warehouse?

Choose star schema if your dimensions are small (under 10 million rows), storage costs aren't critical, and queries must be extremely fast (typical analytical queries). Choose snowflake schema if dimensions are very large, shared across multiple fact tables, or storage costs matter (cost-conscious data lakes). For an e-commerce data warehouse, star schema works well; for an enterprise data warehouse shared across dozens of teams, snowflake schema reduces redundancy. See Week 3 for decision frameworks and real-world examples.

### How do I identify which columns need indexes?

Prioritize indexes on columns in WHERE clauses, JOIN conditions, and ORDER BY statements that filter large tables. Create composite indexes for frequently used multi-column filters. Check query execution plans to see which columns cause full table scans. In an e-commerce database, index "order_date" and "customer_id" on orders table, but not on columns with few unique values (gender) or rarely filtered columns. See Week 4 for execution plan analysis and indexing strategies.

### What Python features are most important for writing efficient data processing scripts?

Master list comprehensions (faster than loops), generators (memory-efficient for streaming), and context managers (resource safety)—these three features distinguish efficient data engineering code from amateur scripts. Decorators are useful for logging and validation. Avoid storing entire datasets in memory when possible; process in chunks. See Week 1 for practical examples in ETL pipelines.

### How do I handle schema evolution when dimensional tables change over time?

Implement Slowly Changing Dimension (SCD) Type 2 for time-tracking changes: add "start_date" and "end_date" columns, keeping historical records with version flags. Use SCD Type 1 (overwrite) only when history doesn't matter. An e-commerce product dimension might track when prices change; SCD Type 2 keeps all price versions. See Week 3 for SCD implementation strategies and when to use each type.

### Should I load data directly into a production data warehouse or use a staging area?

Always use a staging area for data validation and transformation before loading production warehouses. Extract raw data → stage → validate → transform → load (ELTL pattern). This prevents corrupted or incomplete data from affecting analytics and allows you to debug pipeline failures without touching production. See Week 2-3 for ETL pipeline patterns and error handling.

### How do I optimize slow SQL queries in a data warehouse?

Follow this sequence: 1) Use EXPLAIN ANALYZE to find the bottleneck, 2) Add indexes on filtered/joined columns, 3) Rewrite using window functions or CTEs instead of self-joins, 4) Denormalize if optimization can't solve it. Most slow queries are actually slow SQL, not slow schemas. A query joining six tables might be 100x faster with proper indexing. See Week 2 for SQL optimization techniques and Week 4 for query execution plans.

### What's the best way to handle large file imports in Python without running out of memory?

Use chunking strategies: read files in batches (pandas.read_csv with chunksize), use generators to process rows one at a time, or use streaming database inserts instead of loading entire files. For a 10 GB CSV file, chunking into 100 MB chunks lets you process with minimal memory. See Week 1 for generator patterns and context managers in file processing pipelines.

### How do I decide between a data lake and a data warehouse?

Data lakes store all raw data in any format (structured, unstructured) for flexibility and scalability, but require significant governance to remain useful. Data warehouses store cleaned, modeled data optimized for specific queries, providing reliability and performance but less flexibility. Modern architectures use both: data lake for raw storage, data warehouse for analytics. See Week 3 for architectural comparison and enterprise data platform patterns.

## Prerequisites & Next Steps

### What Python skills do I need before starting this bootcamp?

You should be comfortable with basic Python (variables, loops, functions, lists, dictionaries) and have written at least several hundred lines of code in a project. This bootcamp assumes programming maturity and teaches advanced Python for data engineering: generators, list comprehensions, context managers, and decorators. If you're rusty, review functions, data structures, and exception handling before Week 1. See the Prerequisites section in course introduction.

### What SQL knowledge is required to start?

You should know basic SELECT queries, JOINs (INNER, LEFT, RIGHT), WHERE conditions, and simple aggregations (GROUP BY, COUNT, SUM). Weeks 1-2 assume this foundation and immediately jump to advanced SQL: window functions, CTEs, subqueries, and optimization. If you haven't written a LEFT JOIN or GROUP BY query, spend a few hours on SQL basics first. See Week 1-2 curriculum for topics covered.

### What comes after learning database modeling in Week 4?

Weeks 5-8 (not covered here) typically cover data pipelines, orchestration (Airflow), real-time streaming (Kafka), and building production systems. You'll apply database modeling knowledge to design schemas that data pipelines can feed. The modeling skills from Weeks 1-4 form the foundation for understanding how to extract, transform, and load data into warehouses. See course roadmap for full curriculum.

### Do I need cloud experience before learning BigQuery?

No—Week 3 teaches BigQuery as a cloud data warehouse without assuming prior cloud experience. You'll learn BigQuery-specific concepts (partitioning, clustering, pricing model) from scratch. However, basic Unix/Linux command-line comfort helps with cloud environment navigation. If you've never used a terminal, review Linux basics from Week 2. See Week 3 for BigQuery introduction.

### How do these Weeks 1-4 foundations connect to real data engineering jobs?

These weeks cover the core skills used daily in data engineering roles: writing Python for ETL (Week 1-2), designing databases and warehouses (Week 3-4), and using Git/Docker for production code. You'll understand OLTP vs OLAP systems, design schemas, optimize queries, and work with both relational and NoSQL databases. Weeks 1-4 provide the foundation for building actual data pipelines, orchestrating workflows, and building production systems in later weeks. See course learning outcomes and job market alignment documentation.

### What resources help me practice these skills after the bootcamp?

Practice SQL on LeetCode or HackerRank (SQL problems), build mini ETL projects locally with PostgreSQL and Python, contribute to open-source data engineering projects on GitHub, and replicate tutorial data warehouse designs in BigQuery's free tier. The best practice is building small projects like modeling an e-commerce database, writing a Python script to populate it, then querying it with complex SQL. See course project assignments for starter ideas.

### What developer tools should I have installed before starting Week 1?

You'll need Python 3.10+, PostgreSQL or MySQL, git, Docker, a code editor (VS Code, PyCharm), and a terminal. The course provides setup guides. If starting on macOS/Linux, most tools are easy to install; Windows users should use WSL 2 (Windows Subsystem for Linux). Don't worry about BigQuery setup until Week 3. See Week 2 for detailed environment setup guide.

### How do git and Docker skills from Weeks 1-2 apply to later data engineering work?

Git skills ensure you can collaborate on data pipelines in team environments and track changes to SQL schemas and Python code. Docker containerizes entire environments (Python, PostgreSQL, libraries) so pipelines run identically on laptops and production servers. Later weeks build on these foundations by using Docker to containerize data pipelines and Git to version data models. These aren't optional; they're essential for professional work. See Week 1-2 assessments.

### Will I be ready for a data engineering internship or job after Week 4?

Week 4 completion means you understand foundational concepts (OLTP vs OLAP, schemas, normalization, basic optimization) and can work with relational databases and cloud warehouses. You'll be competitive for junior data engineer roles focusing on data warehouse design and analytics, but typical roles also require pipeline orchestration and real-time systems (Weeks 5-8+). After Week 4, you have strong fundamentals; after completing the full bootcamp, you're job-ready. See course completion and career outcomes documentation.
