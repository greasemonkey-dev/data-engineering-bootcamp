# Course Description: Data Engineering Bootcamp (Weeks 1-4)

## Overview

The **Data Engineering Bootcamp: Foundations & Data Storage** is an intensive course designed to transform software engineers into job-ready data engineers. This textbook covers the first 4 weeks of a comprehensive 12-week bootcamp program.

---

## Course Metadata

| Attribute | Details |
|-----------|---------|
| **Title** | Data Engineering Bootcamp: Foundations & Data Storage |
| **Duration** | 2 weeks (Weeks 1-4 of 12-week program) |
| **Time Commitment** | 80-100 hours total |
| **Format** | Self-paced interactive textbook |
| **Delivery** | Git-friendly markdown via GitHub Pages |
| **Target Audience** | Software engineers transitioning to data engineering |
| **Level** | College/Professional |
| **Prerequisites** | Python/programming proficiency, basic SQL, Git basics |

---

## Learning Outcomes

By completing this course, students will be able to:

### Week 1-2: Foundations & Environment Setup

**Python Proficiency:**
- **Write** efficient Python code using list comprehensions, generators, and context managers for data processing tasks
- **Implement** decorators and advanced Python features for production pipelines

**SQL Mastery:**
- **Construct** complex SQL queries using window functions, CTEs, and subqueries to analyze multi-table datasets
- **Optimize** query performance through execution plan analysis and indexing strategies

**DevOps Fundamentals:**
- **Execute** Linux command-line operations for file manipulation, process management, and system navigation
- **Implement** Git workflows including branching, merging, and pull requests for collaborative development
- **Build** containerized applications using Docker and Docker Compose for consistent development environments

**Conceptual Understanding:**
- **Explain** the differences between OLTP and OLAP systems and their appropriate use cases

### Week 3-4: Data Storage & Modeling

**Database Design:**
- **Design** normalized relational database schemas following 3NF principles for transactional systems
- **Design** dimensional models using star and snowflake schemas for analytical workloads
- **Implement** database indexes and query optimization techniques to improve query performance

**Storage Selection:**
- **Compare** relational databases vs NoSQL databases and select appropriate storage based on use case requirements
- **Evaluate** trade-offs between data normalization and denormalization in different contexts
- **Explain** data lake architecture and its role in modern data platforms

**Cloud Data Warehousing:**
- **Construct** BigQuery tables and queries for cloud-based data warehousing
- **Implement** partitioning and clustering strategies to optimize BigQuery performance and costs
- **Design** slowly changing dimension (SCD) implementations for historical tracking

---

## Bloom's Taxonomy Alignment

The course is designed with intentional cognitive progression:

| Week | Primary Bloom's Levels | Secondary Levels | Cognitive Demand |
|------|------------------------|------------------|------------------|
| 1-2  | Apply | Remember, Understand | Low-Medium |
| 3-4  | Apply, Analyze | Understand, Create | Medium |

### Cognitive Level Distribution

- **Remember (20%):** Define key terms, list components, identify concepts
- **Understand (30%):** Explain differences, describe processes, summarize approaches
- **Apply (33%):** Implement solutions, construct queries, execute workflows
- **Analyze (17%):** Compare approaches, evaluate trade-offs, debug issues

---

## Course Structure

### 4 Comprehensive Chapters

#### Chapter 1: Python & SQL Foundations (2500 words)
**Learning Focus:** Apply advanced Python features and SQL techniques

**Key Topics:**
- List comprehensions, generators, context managers, decorators
- SQL window functions for analytics (ROW_NUMBER, RANK, running totals)
- Common Table Expressions (CTEs) for complex multi-step queries
- Query optimization principles

**Hands-on Examples:**
- Inefficient CSV processing improved with generators
- E-commerce analytics with window functions
- Multi-step data transformations with CTEs

#### Chapter 2: DevOps Foundations - Git & Docker (2800 words)
**Learning Focus:** Apply collaborative development and containerization

**Key Topics:**
- Git branching strategies (feature branches, release branches)
- Merge vs rebase workflows and conflict resolution
- Docker fundamentals (images, containers, Dockerfiles)
- Docker Compose for multi-container applications

**Hands-on Examples:**
- Resolving merge conflicts in data pipeline code
- Containerizing Python ETL scripts
- Multi-container setup with application + PostgreSQL database

#### Chapter 3: Database Design & Modeling (3200 words)
**Learning Focus:** Analyze and design efficient database schemas

**Key Topics:**
- Database normalization (1NF, 2NF, 3NF, BCNF)
- Denormalization strategies for analytical workloads
- Index types (B-tree, hash, composite) and selection
- Query execution plans and optimization
- NoSQL databases (MongoDB, Redis) and CAP theorem

**Hands-on Examples:**
- Normalizing messy e-commerce spreadsheet step-by-step
- Identifying and fixing data anomalies
- Choosing indexes for common query patterns
- Deciding when denormalization is justified

#### Chapter 4: Data Warehousing & BigQuery (3400 words)
**Learning Focus:** Create dimensional models and optimize cloud warehouses

**Key Topics:**
- OLTP vs OLAP system design
- Dimensional modeling (fact tables, dimension tables)
- Star schema vs snowflake schema comparison
- Slowly Changing Dimensions (Types 1, 2, 3)
- BigQuery architecture and optimization
- Partitioning and clustering strategies

**Hands-on Examples:**
- Designing star schema for e-commerce analytics
- Implementing SCD Type 2 for customer dimension
- BigQuery partitioning cost comparison ($25 → $0.75 with optimization)
- Comparing snowflake vs star schema trade-offs

---

## Learning Resources

### Interactive Learning Graph (160 Concepts)
A visual knowledge map showing:
- **14 foundational concepts** (entry points with no prerequisites)
- **Dependency relationships** (what concepts build on others)
- **5 taxonomy categories** (FOUND, BASIC, INTER, ADV, APP)
- **Multiple learning pathways** for flexible progression

### Comprehensive Glossary (96 Terms)
ISO 11179-compliant definitions covering:
- Python features (comprehensions, generators, context managers, decorators)
- SQL concepts (window functions, CTEs, indexes, execution plans)
- Git operations (branch, merge, rebase, pull request)
- Docker terminology (container, image, Dockerfile, Compose)
- Database design (normalization, denormalization, ACID, CAP theorem)
- Data warehousing (fact table, dimension table, star schema, SCD)
- BigQuery optimization (partitioning, clustering)

Each term includes:
- Precise 20-50 word definition
- Concrete example from data engineering domain
- Alphabetical organization for quick reference

### Frequently Asked Questions (34 Questions)
Organized into 4 categories:
1. **Conceptual Clarifications (8 questions)** - "What's the difference between OLTP and OLAP?"
2. **Common Misconceptions (6 questions)** - "Do I always need to normalize to 3NF?"
3. **Practical Applications (10 questions)** - "When should I use MongoDB vs PostgreSQL?"
4. **Prerequisites & Next Steps (10 questions)** - "What Python skills do I need before starting?"

### Interactive MicroSims (6 Specifications)
Detailed specifications for interactive visualizations:

1. **SQL Query Execution Plan Visualizer** - See 766x performance improvement with proper indexing
2. **Git Merge vs Rebase Interactive** - Compare branching strategies side-by-side
3. **Star Schema vs Snowflake Comparison** - Visualize dimensional modeling trade-offs
4. **Database Normalization Journey** - Step through 1NF → 2NF → 3NF with real data
5. **BigQuery Partitioning Cost Calculator** - Calculate 97% cost reduction with partitioning
6. **Slowly Changing Dimension Timeline** - Compare SCD Types 1, 2, 3 with timeline visualization

### Chapter Quizzes (40 Questions Total)
- 10 questions per chapter
- Bloom's taxonomy alignment (20% Remember, 30% Understand, 32.5% Apply, 17.5% Analyze)
- Balanced answer distribution (A/B/C/D roughly equal)
- Detailed explanations for each answer
- Concept mapping to learning graph

---

## Assessment Framework

### Formative Assessment (Throughout Course)
- **"Try It" exercises** embedded in chapters
- **Reflection questions** prompting deeper thinking
- **Practice quizzes** with immediate feedback
- **Self-assessment** against learning objectives

### Summative Assessment (End of Weeks 1-4)
- **Chapter quizzes** (40 questions total)
- **Portfolio projects:**
  - Week 1-2: Build ETL pipeline with Python + SQL optimization
  - Week 3-4: Design dimensional model for e-commerce analytics
- **Code review** participation (for bootcamp cohorts)

### Assessment Rubrics

**Knowledge Assessment (Quizzes):**
- 70%+ correct: Proficient (ready for next weeks)
- 60-69%: Developing (review weak areas)
- <60%: Needs intervention (1-on-1 support recommended)

**Project Assessment:**
- Technical implementation (40%)
- Code quality and documentation (30%)
- Design decisions and justification (20%)
- Testing and error handling (10%)

---

## Teaching Methodology

### Evidence-Based Learning Principles

**1. Concrete Before Abstract**
- Start with messy real-world data before teaching theory
- Show production problems before explaining solutions
- Use e-commerce, retail, and streaming platform examples

**2. Intrinsic Motivation**
- Connect concepts to real costs: "Slow queries cost $thousands/month"
- Show production bugs caused by bad schema design
- Demonstrate career-relevant skills with immediate applicability

**3. Low Floor, High Ceiling**
- Accessible entry points for those with basic programming
- Unlimited depth through reflection questions and extensions
- Optional advanced topics for those who want more

**4. Socratic Coaching**
- Reflection questions guide discovery: "When would denormalization be justified?"
- No spoon-feeding of answers
- Encourage critical thinking about trade-offs

**5. Active Learning**
- "Try It" exercises for immediate practice
- Interactive MicroSims for exploration
- Real-world scenarios requiring decision-making

### Instructional Strategies by Bloom's Level

**Remember & Understand (Weeks 1-2 emphasis):**
- Clear explanations with examples
- Glossary for quick reference
- Concept mapping with learning graph

**Apply (Weeks 1-4 emphasis):**
- Step-by-step walkthroughs
- "Try It" exercises with immediate practice
- Code examples with detailed comments

**Analyze (Weeks 3-4 emphasis):**
- Comparative analysis (OLTP vs OLAP, star vs snowflake)
- Trade-off discussions
- Debugging scenarios and optimization challenges

---

## Technology Stack

### Programming Languages & Tools
- **Python 3.10+** - Primary programming language
- **SQL** - PostgreSQL, MySQL dialects
- **Bash** - Shell scripting and command-line tools

### Development Environment
- **Git & GitHub** - Version control and collaboration
- **Docker & Docker Compose** - Containerization
- **VS Code / PyCharm** - Recommended IDEs
- **Jupyter Notebooks** - Exploratory analysis

### Data Storage Systems
- **PostgreSQL** - Relational database (OLTP)
- **MongoDB** - Document database (NoSQL)
- **Redis** - Key-value store (caching)
- **Google BigQuery** - Cloud data warehouse (OLAP)

---

## Prerequisites

### Minimum Requirements
- **Programming:** Proficiency in at least one language (Python preferred)
- **SQL:** Basic queries (SELECT, JOIN, WHERE)
- **Command Line:** Navigate directories, run commands
- **Git:** Clone, commit, push operations
- **No formal degree required**

### Recommended Preparation
If you're rusty on any of these, review before starting:
- Python functions, loops, lists, dictionaries
- SQL joins and aggregations
- Basic Linux commands
- Git branching and merging

### Pre-Course Resources (Optional)
- **Python refresher:** Official Python tutorial
- **SQL practice:** SQLBolt or Mode Analytics SQL Tutorial
- **Docker tutorial:** Docker Getting Started Guide
- **Linux basics:** Linux Journey

---

## What Comes After This Course?

This course prepares you for:

### Weeks 5-6: Data Pipeline Fundamentals
- ETL vs ELT patterns
- Apache Airflow workflow orchestration
- Data quality validation with Great Expectations
- Error handling and monitoring

### Weeks 7-8: Big Data & Distributed Systems
- Apache Spark and PySpark
- Distributed computing concepts
- Kafka for streaming data
- Google Cloud Dataproc

### Weeks 9-10: Cloud Data Engineering
- GCP services (Dataflow, Composer, Pub/Sub)
- Infrastructure as Code with Terraform
- CI/CD for data pipelines
- Data governance and security

### Weeks 11-12: Team Capstone Project
- End-to-end data platform implementation
- Agile/Scrum methodology
- Code reviews and collaboration
- Technical presentations

---

## Success Metrics

Students who complete this course will be able to:
- ✅ Build production-quality Python scripts for data processing
- ✅ Design and optimize relational databases
- ✅ Create dimensional models for analytics
- ✅ Containerize data applications with Docker
- ✅ Collaborate effectively using Git workflows
- ✅ Choose appropriate storage solutions for different use cases
- ✅ Optimize cloud data warehouse queries and costs

---

## Course Credits

**Design:** Based on 12-week Data Engineering Bootcamp curriculum
**Format:** Interactive textbook with MkDocs Material
**Pedagogy:** Evidence-based learning science principles
**Version:** 1.0 (January 28, 2026)

---

## Getting Started

Ready to begin?

**[Start with Chapter 1: Python & SQL Foundations →](chapters/01-python-sql-foundations/)**

Or return to the [Course Home Page](index.md).
