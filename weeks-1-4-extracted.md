# Data Engineering Bootcamp - Weeks 1-4 Content Extraction

## Course Overview
- **Title:** Data Engineering Bootcamp
- **Target Audience:** Software engineers (including bootcamp graduates) transitioning to data engineering
- **Level:** College/Professional
- **Format:** Full-time intensive bootcamp, Sunday-Thursday
- **Weeks 1-4 Focus:** Foundations & Data Storage/Modeling

---

## Week 1-2: Foundations & Environment Setup

### Core Topics
- Python for data engineering (advanced topics)
- SQL mastery (queries, optimization, window functions, CTEs)
- Linux/Bash and command-line proficiency
- Git/GitHub workflows and collaboration
- Docker fundamentals
- Setting up local development environment

### Learning Objectives (Bloom's Focus: Remember, Understand, Apply)

By the end of Week 2, students will be able to:

1. **Write** (Apply) efficient Python code using list comprehensions, generators, and context managers for data processing tasks
2. **Construct** (Apply) complex SQL queries using window functions, CTEs, and subqueries to analyze multi-table datasets
3. **Execute** (Apply) Linux command-line operations for file manipulation, process management, and system navigation
4. **Implement** (Apply) Git workflows including branching, merging, and pull requests for collaborative development
5. **Build** (Apply) containerized applications using Docker and Docker Compose for consistent development environments
6. **Explain** (Understand) the differences between OLTP and OLAP systems and their appropriate use cases

### Key Concepts to Cover
- List comprehensions
- Generators and iterators
- Context managers
- Decorators
- SQL window functions
- Common Table Expressions (CTEs)
- Subqueries
- Query optimization
- OLTP vs OLAP
- Bash scripting
- Git branching strategies
- Git merge vs rebase
- Docker containers
- Docker Compose
- Dockerfile creation
- Container orchestration basics

---

## Week 3-4: Data Storage & Modeling

### Core Topics
- Relational databases (PostgreSQL deep dive)
- NoSQL databases (MongoDB, Redis)
- Data warehousing concepts (star schema, snowflake schema)
- Data lake architecture
- Introduction to BigQuery
- Data modeling best practices

### Learning Objectives (Bloom's Focus: Understand, Apply, Analyze)

By the end of Week 4, students will be able to:

1. **Design** (Create) normalized relational database schemas following 3NF principles for transactional systems
2. **Design** (Create) dimensional models using star and snowflake schemas for analytical workloads
3. **Compare** (Analyze) relational databases vs NoSQL databases and select appropriate storage based on use case requirements
4. **Implement** (Apply) database indexes and query optimization techniques to improve query performance
5. **Construct** (Apply) BigQuery tables and queries for cloud-based data warehousing
6. **Evaluate** (Evaluate) trade-offs between data normalization and denormalization in different contexts
7. **Explain** (Understand) data lake architecture and its role in modern data platforms

### Key Concepts to Cover
- Database normalization (1NF, 2NF, 3NF)
- Denormalization
- Database indexes (B-tree, hash, composite)
- Query execution plans
- PostgreSQL-specific features
- Document databases (MongoDB)
- Key-value stores (Redis)
- CAP theorem
- Eventual consistency
- Data warehousing
- Fact tables
- Dimension tables
- Star schema
- Snowflake schema
- Slowly Changing Dimensions (SCD)
- Data lakes vs data warehouses
- BigQuery architecture
- Partitioning in BigQuery
- Clustering in BigQuery
- BigQuery pricing model

---

## Sample Week Breakdown - Week 3

### Sunday - Introduction to Data Warehousing
- Morning: Lecture on OLTP vs OLAP, data warehouse architecture, dimensional modeling concepts
- Afternoon: Hands-on - Analyze existing e-commerce database, identify fact vs dimension tables

### Monday - Star Schema & Snowflake Schema
- Morning: Deep dive into star/snowflake schema design, best practices, when to use each
- Afternoon: Design dimensional model for retail analytics use case (pair programming)

### Tuesday - BigQuery Introduction
- Morning: BigQuery architecture, pricing model, SQL extensions, best practices
- Afternoon: Load sample dataset into BigQuery, write analytical queries

### Wednesday - Slowly Changing Dimensions (SCD)
- Morning: SCD Types 1-3, implementation strategies, trade-offs
- Afternoon: Implement SCD Type 2 for customer dimension table

### Thursday - Mini Project
- Full day: Build complete dimensional model for music streaming platform
- End of day: Code review sessions in pairs
- Assignment released: "Design and implement data warehouse for food delivery service"

---

## Assessment for Weeks 1-4

### Weekly Assignments
- Week 1-2: "Build an ETL pipeline for retail transactions" + "Optimize slow SQL queries"
- Week 3-4: "Design and implement data warehouse for e-commerce analytics platform"

### Learning Outcomes
- Students can write production-quality Python code
- Students can design and query complex SQL databases
- Students can work with Git in team environments
- Students can containerize applications with Docker
- Students can design dimensional models for analytics
- Students can choose appropriate storage solutions for different use cases

---

## Technology Stack (Weeks 1-4)

### Programming & Development
- Python 3.10+
- SQL (PostgreSQL, MySQL)
- Git & GitHub
- Linux/Bash
- VS Code / PyCharm
- Jupyter Notebooks
- Docker & Docker Compose

### Data Storage
- PostgreSQL (relational)
- MongoDB (document NoSQL)
- Redis (caching/in-memory)
- Google BigQuery (cloud data warehouse)

---

## Hands-on Projects (Weeks 1-4)

1. **Week 1:** Build a data processing script using advanced Python features (generators, context managers)
2. **Week 2:** Optimize a slow SQL database (add indexes, rewrite queries, analyze execution plans)
3. **Week 3:** Design dimensional model for e-commerce analytics (star schema with fact and dimension tables)
4. **Week 4:** Build complete data warehouse in BigQuery with SCD implementation

---

## Common Real-World Scenarios

- E-commerce analytics (orders, customers, products)
- Retail transactions (point-of-sale data)
- Music streaming platform (users, songs, plays)
- Food delivery service (restaurants, orders, drivers)

---

## Prerequisites Assumed
- Proficiency in at least one programming language
- Basic SQL knowledge (SELECT, JOIN, WHERE)
- Comfort with command line basics
- Git fundamentals (clone, commit, push)

---

## MicroSim Opportunities (Weeks 1-4)

Concepts that would benefit from interactive visualization:

1. **SQL Query Execution Plans** - Visualize how different indexes affect query performance
2. **Git Branching Strategies** - Interactive diagram showing merge vs rebase workflows
3. **Docker Container Lifecycle** - Visualize container creation, execution, and networking
4. **Star Schema vs Snowflake Schema** - Interactive comparison of dimensional modeling approaches
5. **Database Normalization** - Show denormalized table being normalized to 3NF step-by-step
6. **BigQuery Partitioning** - Visualize how partitioning affects query cost and performance
7. **Slowly Changing Dimensions** - Demonstrate SCD Type 1 vs Type 2 vs Type 3 with timeline
8. **OLTP vs OLAP** - Compare transaction processing vs analytical query patterns

---

## Key Pedagogical Principles

- **Concrete before abstract:** Show real e-commerce schemas before teaching normalization theory
- **Intrinsic motivation:** Connect to real problems (slow queries cost money, bad schemas cause bugs)
- **Low floor, high ceiling:** Start with simple SELECT queries, progress to complex window functions
- **Socratic coaching:** Include reflection questions like "When would denormalization be justified?"

---

**Extracted:** 2026-01-28
**Source:** data-engineering-bootcamp-design.md
**Scope:** Weeks 1-4 (Foundations & Data Storage/Modeling)
