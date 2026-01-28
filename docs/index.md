# Data Engineering Bootcamp: Foundations & Data Storage

Welcome to the **Data Engineering Bootcamp** - a comprehensive, hands-on course designed for software engineers transitioning into data engineering roles.

## Course Overview

This textbook covers **Weeks 1-4** of the full 12-week bootcamp, focusing on the essential foundations and data storage concepts that every data engineer must master.

### What You'll Learn

By completing this course, you will be able to:

- **Write production-quality Python** using advanced features like generators, comprehensions, and context managers
- **Master SQL** with window functions, CTEs, and query optimization techniques
- **Design robust databases** using normalization principles and appropriate indexing strategies
- **Build dimensional models** for data warehousing with star and snowflake schemas
- **Work effectively with Git** using branching, merging, and collaborative workflows
- **Containerize applications** with Docker and Docker Compose
- **Choose appropriate storage solutions** including relational, NoSQL, and cloud data warehouses
- **Optimize BigQuery queries** with partitioning and clustering strategies

### Course Structure

This course is divided into **4 comprehensive chapters**:

#### [Chapter 1: Python & SQL Foundations](chapters/01-python-sql-foundations/)
Master advanced Python features and SQL techniques essential for data engineering. Learn list comprehensions, generators, window functions, and CTEs through real-world examples.

#### [Chapter 2: DevOps Foundations - Git & Docker](chapters/02-git-docker/)
Build collaborative development skills with Git workflows and containerization with Docker. Understand branching strategies, merge vs rebase, and multi-container applications.

#### [Chapter 3: Database Design & Modeling](chapters/03-database-modeling/)
Design efficient database schemas using normalization principles. Learn when to normalize for transactional systems and when to denormalize for analytics.

#### [Chapter 4: Data Warehousing & BigQuery](chapters/04-data-warehousing/)
Build data warehouses with dimensional modeling. Compare OLTP vs OLAP systems, design star and snowflake schemas, and optimize BigQuery performance.

---

## Learning Resources

### Interactive Learning Tools

- **[Learning Graph](learning-graph/learning-graph.json)** - Visual map of 160 concepts showing dependencies and learning pathways
- **[Glossary](glossary.md)** - 96 technical terms with precise definitions and examples
- **[FAQ](faq.md)** - 34 frequently asked questions covering common misconceptions and practical guidance

### Interactive Visualizations (MicroSims)

Explore abstract concepts through interactive visualizations:

- **[SQL Query Execution Plan Visualizer](sims/sql-execution-plan/spec.md)** - See how indexes dramatically improve query performance
- **[Git Merge vs Rebase Interactive](sims/git-merge-rebase/spec.md)** - Compare branching strategies side-by-side
- **[Star Schema vs Snowflake Comparison](sims/star-vs-snowflake/spec.md)** - Understand dimensional modeling trade-offs
- **[Database Normalization Journey](sims/normalization-journey/spec.md)** - Step through normalization forms visually
- **[BigQuery Partitioning Cost Calculator](sims/bigquery-partitioning/spec.md)** - Optimize cloud data warehouse costs
- **[Slowly Changing Dimension Timeline](sims/scd-timeline/spec.md)** - Compare SCD Type 1, 2, and 3 strategies

### Chapter Quizzes

Test your knowledge with Bloom's taxonomy-aligned quizzes:

- [Chapter 1 Quiz: Python & SQL Foundations](chapters/01-python-sql-foundations/quiz.md) - 10 questions
- [Chapter 2 Quiz: Git & Docker](chapters/02-git-docker/quiz.md) - 10 questions
- [Chapter 3 Quiz: Database Modeling](chapters/03-database-modeling/quiz.md) - 10 questions
- [Chapter 4 Quiz: Data Warehousing](chapters/04-data-warehousing/quiz.md) - 10 questions

---

## Target Audience

This course is designed for:

- **Bootcamp graduates** looking to specialize in data engineering
- **Software engineers** wanting to transition to data-focused roles
- **Full-stack developers** curious about backend data infrastructure
- **Self-taught programmers** with solid coding fundamentals

### Prerequisites

Before starting this course, you should have:

- Proficiency in at least one programming language (Python preferred)
- Basic SQL knowledge (SELECT, JOIN, WHERE clauses)
- Comfort with command line basics
- Git fundamentals (clone, commit, push)

---

## Teaching Philosophy

This course follows evidence-based learning principles:

### Concrete Before Abstract
Every concept starts with real-world examples before diving into theory. You'll see messy denormalized spreadsheets before learning normalization rules, and watch slow queries before understanding indexes.

### Intrinsic Motivation
We connect every concept to real problems data engineers face: slow queries that cost money, bad schemas that cause production bugs, and inefficient pipelines that waste resources.

### Low Floor, High Ceiling
Accessible entry points for beginners with unlimited depth for those who want to dive deeper. Reflection questions prompt critical thinking about trade-offs and edge cases.

### Socratic Coaching
Rather than just delivering content, we ask questions that guide you to discover insights: "When would denormalization be justified?" "How would you decide between merge and rebase?"

---

## How to Use This Course

### For Self-Paced Learning

1. **Read chapters sequentially** - Each chapter builds on previous concepts
2. **Complete "Try It" exercises** as you encounter them
3. **Explore MicroSims** to visualize abstract concepts
4. **Answer reflection questions** before moving forward
5. **Take chapter quizzes** to test comprehension
6. **Reference glossary and FAQ** as needed

### For Bootcamp Instructors

This textbook complements live instruction:

- **Flip the classroom** - Students read content before class, use class time for labs
- **Reduce cognitive load** - Students aren't furiously taking notes during lectures
- **Support struggling students** - Async resource for catching up
- **Enable office hours** - Students can reference specific sections when asking questions

See the [Instructor Guide](chapters/INSTRUCTOR_GUIDE.md) for assessment strategies and implementation guidance.

---

## Technology Stack

This course covers:

**Programming & Development:**
- Python 3.10+
- SQL (PostgreSQL, MySQL)
- Git & GitHub
- Linux/Bash
- Docker & Docker Compose

**Data Storage:**
- PostgreSQL (relational)
- MongoDB (document NoSQL)
- Redis (key-value store)
- Google BigQuery (cloud data warehouse)

---

## What's Next?

After completing Weeks 1-4, you'll be ready for:

- **Weeks 5-6:** Data Pipeline Fundamentals (ETL/ELT, Apache Airflow, data quality)
- **Weeks 7-8:** Big Data & Distributed Systems (Apache Spark, Kafka, cloud processing)
- **Weeks 9-10:** Cloud Data Engineering (GCP deep dive, infrastructure as code, CI/CD)
- **Weeks 11-12:** Team Capstone Project (end-to-end data platform implementation)

---

## Course Information

**Duration:** 2 weeks (Weeks 1-4 of 12-week bootcamp)
**Time Commitment:** 80-100 hours total
**Format:** Self-paced textbook with interactive elements
**Level:** College/Professional
**Last Updated:** January 28, 2026

---

## Getting Started

Ready to begin your data engineering journey?

**[Start with Chapter 1: Python & SQL Foundations →](chapters/01-python-sql-foundations/)**

Or explore the [Course Description](course-description.md) for detailed learning objectives and assessment information.

---

*Built with evidence-based learning science principles for bootcamp graduates transitioning to data engineering.*
