# Data Engineering Bootcamp - Complete Course Design

## Executive Summary

**Duration:** 12 weeks (480-600 hours)
**Format:** Full-time intensive bootcamp, Sunday-Thursday (Israeli schedule)
**Target Audience:** Software engineers (including bootcamp graduates) transitioning to data engineering
**Outcome:** Job-ready Junior Data Engineers
**Key Approach:** 40% theory, 60% hands-on practice with team-based capstone project

---

## Course Structure Overview

### Week 1-2: Foundations & Environment Setup (80-100 hours)

**Core Topics:**
- Python for data engineering (review/advanced topics)
- SQL mastery (queries, optimization, window functions, CTEs)
- Linux/Bash and command-line proficiency
- Git/GitHub workflows and collaboration
- Docker fundamentals
- Setting up local development environment

**Learning Objectives (Bloom's Focus: Remember, Understand, Apply):**

By the end of Week 2, students will be able to:
1. **Write** (Apply) efficient Python code using list comprehensions, generators, and context managers for data processing tasks
2. **Construct** (Apply) complex SQL queries using window functions, CTEs, and subqueries to analyze multi-table datasets
3. **Execute** (Apply) Linux command-line operations for file manipulation, process management, and system navigation
4. **Implement** (Apply) Git workflows including branching, merging, and pull requests for collaborative development
5. **Build** (Apply) containerized applications using Docker and Docker Compose for consistent development environments
6. **Explain** (Understand) the differences between OLTP and OLAP systems and their appropriate use cases

**Hands-on:** Design and implement databases for sample business scenarios

---

### Week 3-4: Data Storage & Modeling (80-100 hours)

**Core Topics:**
- Relational databases (PostgreSQL deep dive)
- NoSQL databases (MongoDB, Redis)
- Data warehousing concepts (star schema, snowflake schema)
- Data lake architecture
- Introduction to BigQuery
- Data modeling best practices

**Learning Objectives (Bloom's Focus: Understand, Apply, Analyze):**

By the end of Week 4, students will be able to:
1. **Design** (Create) normalized relational database schemas following 3NF principles for transactional systems
2. **Design** (Create) dimensional models using star and snowflake schemas for analytical workloads
3. **Compare** (Analyze) relational databases vs NoSQL databases and select appropriate storage based on use case requirements
4. **Implement** (Apply) database indexes and query optimization techniques to improve query performance
5. **Construct** (Apply) BigQuery tables and queries for cloud-based data warehousing
6. **Evaluate** (Evaluate) trade-offs between data normalization and denormalization in different contexts
7. **Explain** (Understand) data lake architecture and its role in modern data platforms

**Hands-on:** Design and implement databases for sample business scenarios

---

### Week 5-6: Data Pipeline Fundamentals (80-100 hours)

**Core Topics:**
- ETL vs ELT patterns
- Batch processing with Python
- Workflow orchestration with Apache Airflow
- Data quality and validation
- Error handling and monitoring
- Logging and observability

**Learning Objectives (Bloom's Focus: Apply, Analyze):**

By the end of Week 6, students will be able to:
1. **Construct** (Apply) ETL pipelines using Python to extract, transform, and load data from multiple sources
2. **Differentiate** (Analyze) between ETL and ELT patterns and select the appropriate approach for specific scenarios
3. **Build** (Apply) production-ready Airflow DAGs with task dependencies, scheduling, and error handling
4. **Implement** (Apply) data quality validation rules using Great Expectations framework
5. **Design** (Create) error handling and retry logic for resilient data pipelines
6. **Configure** (Apply) logging and monitoring for observability in data workflows
7. **Debug** (Analyze) failing pipeline tasks using logs and Airflow UI

**Hands-on:** Build production-grade ETL pipelines with Airflow

**Week 6 Milestone:** Mid-course individual project (5 days) - Students will **create** (Create) an end-to-end data pipeline with ingestion, transformation, quality checks, and orchestration

---

### Week 7-8: Big Data & Distributed Systems (80-100 hours)

**Core Topics:**
- Introduction to distributed computing concepts
- Apache Spark fundamentals (PySpark)
- Data partitioning and parallelization strategies
- Working with large datasets efficiently
- Introduction to streaming concepts (Kafka basics)
- Cloud-native data processing (Dataflow/Dataproc on GCP)

**Learning Objectives (Bloom's Focus: Understand, Apply, Analyze):**

By the end of Week 8, students will be able to:
1. **Explain** (Understand) distributed computing concepts including partitioning, shuffling, and fault tolerance
2. **Write** (Apply) PySpark code using DataFrames API for large-scale data transformations
3. **Optimize** (Analyze) Spark jobs by analyzing execution plans and reducing shuffles
4. **Implement** (Apply) appropriate partitioning strategies to improve Spark job performance
5. **Compare** (Analyze) when to use Spark vs traditional Python/Pandas based on data volume and complexity
6. **Deploy** (Apply) Spark jobs to Google Cloud Dataproc clusters
7. **Explain** (Understand) basic streaming concepts and Kafka message queuing

**Key Insight:** Students will understand horizontal scaling when they see local jobs run 50-100x faster on distributed clusters

**Hands-on:** Build scalable batch processing jobs with Spark

---

### Week 9-10: Cloud Data Engineering & Modern Stack (80-100 hours)

**Core Topics:**
- Cloud-agnostic principles (storage, compute, networking)
- Google Cloud Platform deep dive:
  - BigQuery (data warehousing)
  - Cloud Storage (data lake)
  - Dataflow (streaming/batch)
  - Cloud Composer (managed Airflow)
  - Pub/Sub (messaging)
- Infrastructure as Code (Terraform basics)
- CI/CD for data pipelines
- Data governance and security fundamentals
- Cost optimization strategies

**Learning Objectives (Bloom's Focus: Apply, Analyze, Evaluate):**

By the end of Week 10, students will be able to:
1. **Explain** (Understand) cloud-agnostic data engineering principles for storage, compute, and networking
2. **Build** (Apply) data warehouses using BigQuery with appropriate partitioning and clustering strategies
3. **Implement** (Apply) cloud-native data pipelines using Google Cloud services (Dataflow, Composer, Pub/Sub)
4. **Write** (Apply) Infrastructure as Code using Terraform to provision cloud resources
5. **Design** (Create) CI/CD pipelines for automated testing and deployment of data workflows
6. **Evaluate** (Evaluate) cost vs performance trade-offs when selecting cloud resources and configurations
7. **Implement** (Apply) data governance controls including access management, encryption, and compliance requirements
8. **Analyze** (Analyze) cloud billing data to identify optimization opportunities

**Hands-on:** Migrate local pipelines to GCP, implement IaC

---

### Week 11-12: Capstone Team Project (80-100 hours)

**Learning Objectives (Bloom's Focus: Analyze, Evaluate, Create):**

By the end of Week 12, students will be able to:
1. **Design** (Create) complete end-to-end data platforms solving real-world business problems
2. **Collaborate** (Apply) in Agile teams using daily standups, sprint planning, and retrospectives
3. **Evaluate** (Evaluate) peer code through structured code reviews with constructive feedback
4. **Justify** (Evaluate) technical architecture decisions based on requirements, constraints, and trade-offs
5. **Create** (Create) comprehensive technical documentation including architecture diagrams and deployment guides
6. **Present** (Create) technical solutions to stakeholders with clear explanations of design choices
7. **Integrate** (Create) multiple technologies (orchestration, processing, storage, monitoring) into cohesive systems
8. **Debug** (Analyze) complex system issues across multiple components and services

**Structure:**
- Teams of 3-4 students tackle realistic business scenarios
- End-to-end data platform implementation
- Agile/Scrum methodology with daily standups
- Code reviews and pair programming
- Documentation and presentation
- Final presentations to instructors + industry guests

**Example Projects:**
- E-commerce analytics platform
- IoT sensor data pipeline
- Social media sentiment analysis system

**Final Deliverable:** Fully functional data platform with:
- End-to-end data pipelines
- Cloud deployment with IaC
- Monitoring and alerting
- CI/CD automation
- Comprehensive documentation
- 15-minute technical presentation

---

## Bloom's Taxonomy Framework & Assessment Alignment

### Cognitive Progression Across 12 Weeks

The bootcamp is designed with intentional cognitive progression using Bloom's Taxonomy:

| Week | Primary Bloom's Levels | Secondary Levels | Cognitive Demand | Assessment Type |
|------|------------------------|------------------|------------------|-----------------|
| 1-2  | Apply | Remember, Understand | Low-Medium | Coding exercises, SQL challenges |
| 3-4  | Apply, Analyze | Understand, Create | Medium | Database design projects |
| 5-6  | Apply, Analyze | Create | Medium-High | ETL pipeline + Individual project |
| 7-8  | Apply, Analyze | Understand | Medium-High | Spark optimization challenges |
| 9-10 | Apply, Evaluate | Analyze, Create | High | Cloud migration + cost analysis |
| 11-12 | Create, Evaluate | Analyze | Very High | Team capstone project |

### Bloom's Taxonomy Levels Explained

**Remember (Lowest):** Recall facts, terms, basic concepts
- Verbs: Define, List, Name, Identify
- Example: "List the components of an Airflow DAG"

**Understand:** Explain ideas or concepts
- Verbs: Explain, Describe, Summarize, Compare
- Example: "Explain the difference between ETL and ELT"

**Apply:** Use information in new situations
- Verbs: Implement, Execute, Build, Construct
- Example: "Build an Airflow DAG that runs daily at 2 AM"

**Analyze:** Draw connections among ideas
- Verbs: Analyze, Compare, Differentiate, Debug
- Example: "Analyze this slow Spark job and identify 3 optimization opportunities"

**Evaluate:** Justify a decision or course of action
- Verbs: Evaluate, Justify, Assess, Critique
- Example: "Evaluate whether to use BigQuery or Cloud SQL and justify your choice"

**Create (Highest):** Produce new or original work
- Verbs: Design, Develop, Create, Formulate
- Example: "Design a data platform for real-time fraud detection"

### Daily Learning Objective Structure

Each day includes objectives at multiple Bloom's levels:

**Morning Lecture Objectives (2-3 objectives):**
- Focus: Remember, Understand
- Example: "Explain the purpose of Airflow sensors and when to use them"

**Afternoon Practice Objectives (2-3 objectives):**
- Focus: Apply, Analyze
- Example: "Implement sensor-based task dependencies in an Airflow DAG"

**Exit Ticket (1 question, 5 minutes):**
- Quick formative assessment
- Example: "In your own words, when would you use a sensor vs. a regular task dependency?"

### Assessment Alignment Matrix

Every assessment directly maps to learning objectives:

| Assessment | Week | Bloom's Levels Tested | Alignment |
|------------|------|----------------------|-----------|
| Weekly Assignments | 1-10 | Apply, Analyze | Students practice skills from that week's objectives |
| Mid-Course Project | 6 | Apply, Analyze, Create | Tests cumulative objectives from Weeks 1-6 |
| Spark Optimization | 7-8 | Analyze | Direct assessment of "Optimize Spark jobs" objective |
| Cloud Migration | 9-10 | Apply, Evaluate | Tests IaC, deployment, and cost evaluation objectives |
| Team Capstone | 11-12 | Analyze, Evaluate, Create | Comprehensive assessment of all objectives |
| Code Reviews | 1-12 | Evaluate | Assesses ability to critique and justify decisions |

### Instructional Strategies by Bloom's Level

**For Remember & Understand (Weeks 1-4 emphasis):**
- Lectures with clear explanations
- Concept mapping exercises
- Reading technical documentation
- Demonstrations and walkthroughs

**For Apply (Weeks 1-10 emphasis):**
- Guided practice and code-alongs
- Pair programming exercises
- Structured labs with clear instructions
- Incremental skill building

**For Analyze (Weeks 5-12 emphasis):**
- Debugging challenges
- Performance optimization tasks
- Comparative analysis exercises
- Root cause analysis activities

**For Evaluate (Weeks 9-12 emphasis):**
- Code review sessions
- Architecture review discussions
- Trade-off analysis exercises
- Cost-benefit evaluations

**For Create (Weeks 5-12 emphasis):**
- Open-ended project work
- System design challenges
- Original implementations
- Integration of multiple concepts

### Formative Assessment Strategy

**Daily Exit Tickets (5 minutes):**
- Quick check-in question at end of day
- Identifies struggling students early
- Example: "What was the most confusing concept today?"

**Weekly Reflection (10 minutes, Thursdays):**
- Metacognitive prompts
- "What was hardest this week? Why?"
- "How would you explain [X] to a non-technical friend?"
- "What would you do differently next time?"

**Peer Code Reviews (Ongoing):**
- Students review each other's code
- Develops evaluation skills
- Builds collaborative culture

### Meta-Cognitive Development

Beyond technical skills, students develop:

**Self-Directed Learning:**
- Navigate documentation independently
- Debug using Stack Overflow, logs, error messages
- Identify knowledge gaps and seek resources

**Collaborative Problem-Solving:**
- Communicate technical concepts clearly
- Give and receive constructive feedback
- Share knowledge to support team success

**Professional Practice:**
- Follow coding standards and conventions
- Write clear documentation and commit messages
- Estimate effort and manage time

---

## Daily Schedule & Teaching Methodology

### Typical Daily Schedule (8-10 hours, Sunday-Thursday)

**Morning Block (9:00 AM - 12:30 PM):**
- 9:00-10:30: Live lecture/demonstration (new concepts, theory)
- 10:30-10:45: Break
- 10:45-12:30: Guided hands-on exercises (students code along, instructors circulate)

**Afternoon Block (1:30 PM - 6:00 PM):**
- 1:30-2:00: Daily standup (students share progress, blockers)
- 2:00-5:00: Independent/pair programming on assignments/projects
- 5:00-6:00: Code review sessions, Q&A, or guest speakers (industry practitioners)

**Evening (Optional 6:00-8:00 PM):**
- Office hours for struggling students
- Study groups and peer collaboration
- Catching up on assignments

**Weekend:**
- Friday-Saturday: Off

---

### Teaching Approach (40% theory, 60% hands-on)

- Lectures are focused and concise (90 min max)
- Every concept taught in morning is practiced in afternoon
- Weekly mini-projects (Thursdays) to consolidate learning
- Pair programming rotations to build collaboration skills
- Code reviews mandatory - students review each other's work
- Real-world guest speakers weekly (practicing data engineers)

---

### Support Structure

- Lead instructor + 2-3 teaching assistants (1:15-20 student ratio)
- Slack/Discord for async communication
- 1-on-1 check-ins biweekly

---

## Sample Week Breakdown (Israeli Schedule)

### Week 3: Data Warehousing & Modeling

**Sunday - Introduction to Data Warehousing:**
- Morning: Lecture on OLTP vs OLAP, data warehouse architecture, dimensional modeling concepts
- Afternoon: Hands-on - Analyze existing e-commerce database, identify fact vs dimension tables

**Monday - Star Schema & Snowflake Schema:**
- Morning: Deep dive into star/snowflake schema design, best practices, when to use each
- Afternoon: Design dimensional model for retail analytics use case (pair programming)

**Tuesday - BigQuery Introduction:**
- Morning: BigQuery architecture, pricing model, SQL extensions, best practices
- Afternoon: Load sample dataset into BigQuery, write analytical queries

**Wednesday - Slowly Changing Dimensions (SCD):**
- Morning: SCD Types 1-3, implementation strategies, trade-offs
- Afternoon: Implement SCD Type 2 for customer dimension table

**Thursday - Mini Project:**
- Full day: Build complete dimensional model for music streaming platform
- End of day: Code review sessions in pairs
- Assignment released: "Design and implement data warehouse for food delivery service" (due next Wednesday)

**Friday-Saturday:** Weekend off

---

### Week 7: Apache Spark Fundamentals

**Sunday - Distributed Computing Concepts:**
- Morning: MapReduce paradigm, partitioning, shuffling, distributed systems challenges
- Afternoon: Set up Spark locally, run first PySpark job, explore Spark UI

**Monday - RDDs and DataFrames:**
- Morning: RDD transformations/actions, DataFrame API, lazy evaluation, DAG optimization
- Afternoon: Convert batch processing scripts from Week 6 to PySpark

**Tuesday - Spark SQL and Performance:**
- Morning: Spark SQL, query optimization, partitioning strategies, broadcast joins
- Afternoon: Optimize slow Spark job (provided intentionally inefficient code)

**Wednesday - Data Processing Patterns:**
- Morning: Common patterns - aggregations, window functions, deduplication at scale
- Afternoon: Build aggregation pipeline for clickstream data (billions of records simulation)

**Thursday - Spark on GCP Dataproc:**
- Morning: Introduction to Dataproc, cluster management, job submission
- Afternoon: Deploy local Spark jobs to Dataproc, compare performance and costs
- Assignment released: "Build scalable product recommendation pipeline with Spark" (due next Wednesday)

**Friday-Saturday:** Weekend off

---

## Assessment & Evaluation Framework

### Grading Components

- Weekly assignments/projects: 30%
- Mid-course individual project (Week 6): 20%
- Final team capstone project: 30%
- Code reviews and collaboration: 10%
- Daily participation and attendance: 10%

---

### Weekly Assignments (Weeks 1-10)

- Released Thursday afternoon, due following Wednesday
- Progressively complex real-world scenarios
- Automated tests for validation (students must pass 80%+ tests)
- Peer code review required before submission

**Examples:**
- "Build an ETL pipeline for retail transactions"
- "Optimize slow SQL queries"
- "Deploy Airflow DAG to production"

---

### Mid-Course Project (Week 6)

- Individual project, 5 days to complete
- End-to-end pipeline: data ingestion → transformation → storage → visualization
- Must include: error handling, logging, testing, documentation
- Presentation to cohort (10 min demo + Q&A)
- Simulates technical interview take-home assignment

---

### Final Team Capstone (Weeks 11-12)

**Teams graded on:**
- Technical implementation
- Code quality
- Collaboration
- Documentation
- Presentation

**Must use:**
- Git workflow
- CI/CD
- IaC
- Monitoring
- Testing

**Evaluation:**
- Instructors + external industry reviewers
- Includes written technical design document

---

### Collaboration Metrics

- Quality of code reviews given to peers
- Git commit patterns and PR descriptions
- Participation in pair programming sessions
- Helpfulness in study groups and Slack

---

## Technology Stack & Tools

### Core Programming & Development

- Python 3.10+ (primary language)
- SQL (PostgreSQL, MySQL)
- Git & GitHub (version control, collaboration)
- Linux/Bash scripting
- VS Code / PyCharm (IDEs)
- Jupyter Notebooks (exploratory analysis)
- Docker & Docker Compose

---

### Data Storage & Databases

- PostgreSQL (relational)
- MongoDB (document NoSQL)
- Redis (caching/in-memory)
- Google BigQuery (cloud data warehouse)
- Cloud Storage / S3-compatible storage

---

### Data Processing & Orchestration

- Apache Airflow (workflow orchestration)
- Apache Spark / PySpark (distributed processing)
- Pandas & Polars (data manipulation)
- dbt (data transformation framework)
- Google Dataflow (streaming/batch on GCP)

---

### Cloud & Infrastructure

**Google Cloud Platform (primary):**
- BigQuery
- Cloud Storage
- Dataflow
- Composer
- Pub/Sub
- Cloud Functions

**Infrastructure:**
- Terraform (Infrastructure as Code)
- GitHub Actions (CI/CD)

---

### Data Quality & Testing

- Great Expectations (data validation)
- pytest (unit testing)
- SQL testing frameworks

---

### Monitoring & Observability

- Prometheus + Grafana (metrics)
- ELK Stack basics (logging)
- Cloud monitoring tools (GCP Logging/Monitoring)

---

### Collaboration & Documentation

- Slack/Discord (communication)
- Jira (project management during capstone)
- Confluence/Notion (documentation)
- draw.io / Lucidchart (architecture diagrams)

---

## Prerequisites & Admission Requirements

### Minimum Requirements

- Proficiency in at least one programming language (Python preferred, but Java/JavaScript/C++ acceptable)
- Basic SQL knowledge (SELECT, JOIN, WHERE clauses)
- Comfort with command line basics
- Git fundamentals (clone, commit, push)
- **No formal degree required**

---

### Ideal Candidate Profile

- Fullstack bootcamp graduates
- Self-taught developers with portfolio projects
- Career changers with strong technical aptitude
- Software engineers wanting to specialize in data
- Strong problem-solving and debugging skills
- Self-motivated learner
- Good communication and collaboration skills

---

### Pre-Course Preparation (2-4 weeks before start)

- Python refresher course (if rusty)
- SQL fundamentals online course
- Docker tutorial
- Linux command line basics
- Set up development environment (provided setup guide)

---

### Admissions Process

1. Application + portfolio/project review
2. Technical assessment (Python coding challenge + SQL problems, 90 minutes)
3. Live technical interview (45 min - assess coding ability and problem-solving)
4. Brief culture fit conversation (15 min - assess teamwork, motivation, grit)

---

## Learning Resources & Materials

### Course Materials Provided

- Custom curriculum and lecture slides
- Video recordings of all lectures (for review/catch-up)
- Curated reading lists and documentation guides
- Exercise notebooks and starter code repositories
- Solution guides (released after assignment deadlines)
- Architecture diagram templates
- Best practices checklists and style guides

---

### External Resources (Free/Open Source)

- Official documentation (Python, SQL, Airflow, Spark, GCP)
- Selected chapters from key books:
  - "Designing Data-Intensive Applications" by Martin Kleppmann
  - "Fundamentals of Data Engineering" by Joe Reis & Matt Housley
  - "The Data Warehouse Toolkit" by Ralph Kimball
- Online tutorials and blog posts (curated collection)
- YouTube channels and conference talks

---

### Hands-On Lab Environment

- GitHub organization with template repositories
- Google Cloud Platform credits ($300-500 per student)
- Docker images with pre-configured environments
- Shared PostgreSQL/MongoDB instances for practice
- Airflow sandbox environment
- Sample datasets (retail, IoT, logs, etc.)

---

### Community & Support Resources

- Private Slack workspace with channels per topic
- Shared knowledge base (FAQ, troubleshooting guides)
- Peer study groups (self-organized)
- Weekly "office hours" recordings library

---

### No Required Purchases

- All tools are free or open-source
- Cloud credits provided
- Optional: students can use personal laptops (specs provided) or access cloud-based development environments

---

## Instructor Team Requirements

### Lead Instructor (1)

- 5+ years data engineering experience
- Production experience with: Python, SQL, Airflow, Spark, cloud platforms
- Previous teaching/mentoring experience preferred
- Strong communication and presentation skills
- Can explain complex concepts simply

---

### Teaching Assistants (2-3 for cohort of 20-25 students)

- 2+ years data engineering experience
- Hands-on expertise with course tech stack
- Patient and supportive teaching style
- Available for office hours and 1-on-1 support
- Can debug and troubleshoot student issues quickly

---

### Guest Speakers (Weekly, Weeks 3-11)

- Practicing data engineers from various industries
- 1-hour sessions: career path, real-world challenges, Q&A
- Provides industry context and networking

---

## Potential Challenges & Mitigation Strategies

### Challenge 1: Varied Technical Backgrounds

**Issue:** Some students stronger in Python, others in SQL, pace varies

**Mitigation:**
- Pre-course assessment to identify gaps
- Leveling resources in Week 1
- Pair programming with mixed skill levels
- Office hours for catching up
- TAs provide targeted support

---

### Challenge 2: Cloud Costs

**Issue:** Students may exceed GCP credits if not careful

**Mitigation:**
- Budget alerts and monitoring training
- Strict resource cleanup protocols
- Shared environments for early exercises
- Cost optimization as explicit learning objective
- Emergency fund for exceptional cases

---

### Challenge 3: Team Dynamics in Capstone

**Issue:** Conflicts, unequal contribution, communication breakdowns

**Mitigation:**
- Team formation with personality/skill balance
- Agile ceremonies (standups, retros) built in
- Individual accountability metrics (commits, reviews)
- Instructors monitor team health weekly
- Conflict resolution protocols

---

### Challenge 4: Overwhelming Content Volume

**Issue:** 12 weeks is intense, student burnout risk

**Mitigation:**
- Clear weekly goals and priorities
- "Stretch" vs "core" material differentiation
- Mental health check-ins
- Built-in buffer days (no new content Thursdays for mini-projects)
- Encourage work-life balance

---

### Challenge 5: Keeping Up with Technology Changes

**Issue:** Data engineering tools evolve rapidly

**Mitigation:**
- Focus on fundamental concepts over tools
- Quarterly curriculum review and updates
- Guest speakers share emerging trends
- Teach "learning to learn" skills

---

## Key Design Decisions

1. **Cloud-agnostic + GCP implementation:** Teach portable concepts, use GCP for hands-on (better free tier, modern APIs)
2. **Team capstone project:** Builds critical collaboration skills needed in industry
3. **40/60 theory/practice split:** Balances understanding with practical competency
4. **No degree required:** Accessible to bootcamp graduates and self-taught developers
5. **Israeli work week (Sun-Thu):** Aligned with local business culture
6. **12-week duration:** Intensive enough for transformation, short enough to maintain momentum
7. **Practice-heavy approach:** Learn by doing, not just watching
8. **Modern tech stack:** Industry-relevant tools (Airflow, Spark, dbt, Terraform)

---

## Success Metrics

- Student completion rate: Target 85%+
- Portfolio quality: All graduates have 2-3 production-ready projects on GitHub
- Technical competency: 90%+ pass rate on final capstone project
- Student satisfaction: 4.5/5 average rating
- Employer feedback: Positive hiring outcomes for graduates

---

## Next Steps for Implementation

1. Finalize detailed curriculum for each week
2. Develop lecture materials and hands-on exercises
3. Set up lab environments and cloud infrastructure
4. Create assessment rubrics and automated tests
5. Recruit instructor team
6. Build marketing materials and launch admissions
7. Pilot with initial cohort of 10-15 students
8. Iterate based on feedback

---

**Document Created:** 2026-01-28
**Version:** 1.0
**Status:** Draft for Review
