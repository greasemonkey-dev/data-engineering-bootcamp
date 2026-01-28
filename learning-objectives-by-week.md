# Learning Objectives by Week - Data Engineering Bootcamp

## Week 1-2: Foundations & Environment Setup

**Bloom's Focus:** Remember, Understand, Apply

By the end of Week 2, students will be able to:

1. **Write** (Apply) efficient Python code using list comprehensions, generators, and context managers for data processing tasks
2. **Construct** (Apply) complex SQL queries using window functions, CTEs, and subqueries to analyze multi-table datasets
3. **Execute** (Apply) Linux command-line operations for file manipulation, process management, and system navigation
4. **Implement** (Apply) Git workflows including branching, merging, and pull requests for collaborative development
5. **Build** (Apply) containerized applications using Docker and Docker Compose for consistent development environments
6. **Explain** (Understand) the differences between OLTP and OLAP systems and their appropriate use cases

---

## Week 3-4: Data Storage & Modeling

**Bloom's Focus:** Understand, Apply, Analyze

By the end of Week 4, students will be able to:

1. **Design** (Create) normalized relational database schemas following 3NF principles for transactional systems
2. **Design** (Create) dimensional models using star and snowflake schemas for analytical workloads
3. **Compare** (Analyze) relational databases vs NoSQL databases and select appropriate storage based on use case requirements
4. **Implement** (Apply) database indexes and query optimization techniques to improve query performance
5. **Construct** (Apply) BigQuery tables and queries for cloud-based data warehousing
6. **Evaluate** (Evaluate) trade-offs between data normalization and denormalization in different contexts
7. **Explain** (Understand) data lake architecture and its role in modern data platforms

**Key Assessment:** Design and implement a complete dimensional model for an e-commerce analytics platform

---

## Week 5-6: Data Pipeline Fundamentals

**Bloom's Focus:** Apply, Analyze

By the end of Week 6, students will be able to:

1. **Construct** (Apply) ETL pipelines using Python to extract, transform, and load data from multiple sources
2. **Differentiate** (Analyze) between ETL and ELT patterns and select the appropriate approach for specific scenarios
3. **Build** (Apply) production-ready Airflow DAGs with task dependencies, scheduling, and error handling
4. **Implement** (Apply) data quality validation rules using Great Expectations framework
5. **Design** (Create) error handling and retry logic for resilient data pipelines
6. **Configure** (Apply) logging and monitoring for observability in data workflows
7. **Debug** (Analyze) failing pipeline tasks using logs and Airflow UI

**Mid-Course Project (Week 6):** Students will **create** (Create) an end-to-end data pipeline with ingestion, transformation, quality checks, and orchestration

---

## Week 7-8: Big Data & Distributed Systems

**Bloom's Focus:** Understand, Apply, Analyze

By the end of Week 8, students will be able to:

1. **Explain** (Understand) distributed computing concepts including partitioning, shuffling, and fault tolerance
2. **Write** (Apply) PySpark code using DataFrames API for large-scale data transformations
3. **Optimize** (Analyze) Spark jobs by analyzing execution plans and reducing shuffles
4. **Implement** (Apply) appropriate partitioning strategies to improve Spark job performance
5. **Compare** (Analyze) when to use Spark vs traditional Python/Pandas based on data volume and complexity
6. **Deploy** (Apply) Spark jobs to Google Cloud Dataproc clusters
7. **Explain** (Understand) basic streaming concepts and Kafka message queuing

**Key Insight:** Students will understand horizontal scaling when they see local jobs run 50-100x faster on distributed clusters

---

## Week 9-10: Cloud Data Engineering & Modern Stack

**Bloom's Focus:** Apply, Analyze, Evaluate

By the end of Week 10, students will be able to:

1. **Explain** (Understand) cloud-agnostic data engineering principles for storage, compute, and networking
2. **Build** (Apply) data warehouses using BigQuery with appropriate partitioning and clustering strategies
3. **Implement** (Apply) cloud-native data pipelines using Google Cloud services (Dataflow, Composer, Pub/Sub)
4. **Write** (Apply) Infrastructure as Code using Terraform to provision cloud resources
5. **Design** (Create) CI/CD pipelines for automated testing and deployment of data workflows
6. **Evaluate** (Evaluate) cost vs performance trade-offs when selecting cloud resources and configurations
7. **Implement** (Apply) data governance controls including access management, encryption, and compliance requirements
8. **Analyze** (Analyze) cloud billing data to identify optimization opportunities

**Key Assessment:** Migrate local pipelines to GCP with full IaC, monitoring, and cost optimization

---

## Week 11-12: Capstone Team Project

**Bloom's Focus:** Analyze, Evaluate, Create

By the end of Week 12, students will be able to:

1. **Design** (Create) complete end-to-end data platforms solving real-world business problems
2. **Collaborate** (Apply) in Agile teams using daily standups, sprint planning, and retrospectives
3. **Evaluate** (Evaluate) peer code through structured code reviews with constructive feedback
4. **Justify** (Evaluate) technical architecture decisions based on requirements, constraints, and trade-offs
5. **Create** (Create) comprehensive technical documentation including architecture diagrams and deployment guides
6. **Present** (Create) technical solutions to stakeholders with clear explanations of design choices
7. **Integrate** (Create) multiple technologies (orchestration, processing, storage, monitoring) into cohesive systems
8. **Debug** (Analyze) complex system issues across multiple components and services

**Final Deliverable:** Fully functional data platform with:
- End-to-end data pipelines
- Cloud deployment with IaC
- Monitoring and alerting
- CI/CD automation
- Comprehensive documentation
- 15-minute technical presentation

---

## Bloom's Taxonomy Progression Across Bootcamp

| Week | Primary Levels | Secondary Levels | Cognitive Demand |
|------|----------------|------------------|------------------|
| 1-2  | Apply | Remember, Understand | Low-Medium |
| 3-4  | Apply, Analyze | Understand, Create | Medium |
| 5-6  | Apply, Analyze | Create | Medium-High |
| 7-8  | Apply, Analyze | Understand | Medium-High |
| 9-10 | Apply, Evaluate | Analyze, Create | High |
| 11-12 | Create, Evaluate | Analyze | Very High |

---

## Assessment Alignment Matrix

| Week | Learning Objective Level | Assessment Type | Bloom's Level Assessed |
|------|-------------------------|-----------------|------------------------|
| 1-2  | Apply | Coding exercises, SQL challenges | Remember, Understand, Apply |
| 3-4  | Apply, Analyze | Database design project | Apply, Analyze |
| 5-6  | Apply, Analyze, Create | ETL pipeline + Mid-course project | Apply, Analyze, Create |
| 7-8  | Apply, Analyze | Spark optimization challenges | Apply, Analyze |
| 9-10 | Apply, Evaluate | Cloud migration + cost analysis | Apply, Analyze, Evaluate |
| 11-12 | Create, Evaluate | Team capstone project | Analyze, Evaluate, Create |

---

## Daily Learning Objective Framework

Each day should include:

**Morning Lecture Objectives (2-3 objectives):**
- Lower Bloom's levels: Remember, Understand
- Example: "Explain the purpose of Airflow sensors"

**Afternoon Practice Objectives (2-3 objectives):**
- Higher Bloom's levels: Apply, Analyze
- Example: "Implement sensor-based task dependencies in an Airflow DAG"

**Exit Ticket Question (1 question):**
- Quick check for understanding
- Example: "In your own words, when would you use a sensor vs. a regular task dependency?"

---

## Instructional Strategies by Bloom's Level

### Remember & Understand (Weeks 1-4 focus)
- **Strategies:** Lectures, reading, demos, concept maps
- **Assessment:** Definitions, explanations, quizzes
- **Example:** "Explain the difference between star and snowflake schemas"

### Apply (Weeks 1-10 focus)
- **Strategies:** Guided practice, code-alongs, pair programming
- **Assessment:** Coding exercises, project implementation
- **Example:** "Build an Airflow DAG that runs daily at 2 AM"

### Analyze (Weeks 5-12 focus)
- **Strategies:** Debugging challenges, performance optimization, comparative analysis
- **Assessment:** Optimization tasks, trade-off analysis
- **Example:** "Analyze this slow Spark job and identify 3 optimization opportunities"

### Evaluate (Weeks 9-12 focus)
- **Strategies:** Code reviews, architecture reviews, cost analysis
- **Assessment:** Justified recommendations, critical reviews
- **Example:** "Evaluate whether to use BigQuery or Cloud SQL for this use case and justify your choice"

### Create (Weeks 5-12 focus)
- **Strategies:** Project work, system design, open-ended problems
- **Assessment:** Original implementations, design documents
- **Example:** "Design a data platform for real-time fraud detection"

---

## Meta-Cognitive Learning Objectives

Students will also develop these transferable skills:

1. **Self-Directed Learning**
   - Navigate technical documentation independently
   - Debug issues using Stack Overflow, GitHub issues, and error messages
   - Identify knowledge gaps and seek resources

2. **Collaborative Problem-Solving**
   - Communicate technical concepts to diverse audiences
   - Give and receive constructive code review feedback
   - Contribute to team success through knowledge sharing

3. **Professional Practice**
   - Follow coding standards and best practices
   - Write clear documentation and commit messages
   - Estimate effort and manage time effectively

---

## Validation Criteria

Learning objectives are considered well-formed if they:

✅ Use measurable Bloom's taxonomy verbs (avoid "know", "understand" without context)
✅ Specify observable outcomes (can be assessed)
✅ Align with assessments (what you teach = what you test)
✅ Progress from lower to higher cognitive levels across weeks
✅ Connect to real-world data engineering tasks

❌ Avoid vague verbs: "be familiar with", "learn about", "gain exposure to"
❌ Avoid unmeasurable outcomes: "appreciate", "believe in"

---

**Document Version:** 1.0
**Created:** 2026-01-28
**Aligned with:** Data Engineering Bootcamp Course Design v1.0
