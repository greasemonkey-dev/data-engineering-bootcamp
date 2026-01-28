# Data Engineering Bootcamp: Foundations & Data Storage

A comprehensive, interactive textbook for software engineers transitioning into data engineering roles. This course covers **Weeks 1-4** of a full 12-week bootcamp program, focusing on essential foundations and data storage concepts.

## 🎯 What You'll Learn

- **Python for Data Engineering** - Advanced features like generators, comprehensions, context managers
- **SQL Mastery** - Window functions, CTEs, query optimization
- **Git & Docker** - Collaborative workflows and containerization
- **Database Design** - Normalization, indexing, schema optimization
- **Data Warehousing** - Dimensional modeling with star/snowflake schemas
- **Cloud Data Warehouses** - BigQuery optimization and cost management

## 📚 Course Structure

### 4 Comprehensive Chapters

| Chapter | Topics | Words | Quiz |
|---------|--------|-------|------|
| [Chapter 1: Python & SQL Foundations](docs/chapters/01-python-sql-foundations/) | List comprehensions, generators, window functions, CTEs | 2,500 | 10 questions |
| [Chapter 2: Git & Docker](docs/chapters/02-git-docker/) | Branching, merge vs rebase, containerization, Docker Compose | 2,800 | 10 questions |
| [Chapter 3: Database Modeling](docs/chapters/03-database-modeling/) | Normalization, indexing, query optimization, NoSQL | 3,200 | 10 questions |
| [Chapter 4: Data Warehousing](docs/chapters/04-data-warehousing/) | OLTP vs OLAP, star/snowflake schemas, BigQuery, SCD | 3,400 | 10 questions |

## 🎮 Interactive Learning Resources

### Learning Graph (160 Concepts)
Visual knowledge map showing concept dependencies and learning pathways
- [View Learning Graph](docs/learning-graph/learning-graph.json)

### Glossary (96 Terms)
ISO 11179-compliant definitions with real-world examples
- [View Glossary](docs/glossary.md)

### FAQ (34 Questions)
Common questions, misconceptions, and practical guidance
- [View FAQ](docs/faq.md)

### Interactive MicroSims (6 Visualizations)
Specifications for interactive concept visualizations:
- SQL Query Execution Plan Visualizer
- Git Merge vs Rebase Interactive
- Star Schema vs Snowflake Comparison
- Database Normalization Journey
- BigQuery Partitioning Cost Calculator
- Slowly Changing Dimension Timeline

## 🚀 Quick Start

### Option 1: View Online (GitHub Pages)

Visit the live site: **[https://yourusername.github.io/data-engineering-course/](https://yourusername.github.io/data-engineering-course/)**

### Option 2: Run Locally

```bash
# Clone the repository
git clone https://github.com/yourusername/data-engineering-course.git
cd data-engineering-course

# Install MkDocs and Material theme
pip install mkdocs-material
pip install mkdocs-minify-plugin

# Serve locally
mkdocs serve

# Open browser to http://127.0.0.1:8000
```

### Option 3: Build Static Site

```bash
# Build static HTML files
mkdocs build

# Output will be in site/ directory
# Deploy to any static hosting (GitHub Pages, Netlify, Vercel, etc.)
```

## 📋 Prerequisites

- **Programming:** Proficiency in Python or another programming language
- **SQL:** Basic queries (SELECT, JOIN, WHERE)
- **Command Line:** Comfortable with terminal/shell
- **Git:** Clone, commit, push basics
- **No formal degree required**

## 🎓 Target Audience

- Bootcamp graduates specializing in data engineering
- Software engineers transitioning to data roles
- Full-stack developers exploring backend data infrastructure
- Self-taught programmers with solid fundamentals

## 🏗️ Project Structure

```
data-engineering-course/
├── docs/                           # Course content
│   ├── index.md                    # Course home page
│   ├── course-description.md       # Detailed course info
│   ├── glossary.md                 # Technical terms (96 entries)
│   ├── faq.md                      # FAQ (34 questions)
│   ├── chapters/                   # Chapter content and quizzes
│   │   ├── 01-python-sql-foundations/
│   │   ├── 02-git-docker/
│   │   ├── 03-database-modeling/
│   │   └── 04-data-warehousing/
│   ├── sims/                       # MicroSim specifications
│   │   ├── sql-execution-plan/
│   │   ├── git-merge-rebase/
│   │   ├── star-vs-snowflake/
│   │   ├── normalization-journey/
│   │   ├── bigquery-partitioning/
│   │   └── scd-timeline/
│   └── learning-graph/             # Concept dependency graph
├── mkdocs.yml                      # MkDocs configuration
├── README.md                       # This file
├── PROGRESS-TODO.md                # Generation progress tracker
└── learning-graph.json             # Source learning graph
```

## 🔧 Technology Stack

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

## 📊 Content Metrics

- **Total Concepts:** 160 (in learning graph)
- **Total Words:** ~12,000 (across 4 chapters)
- **Quiz Questions:** 40 (10 per chapter)
- **Glossary Terms:** 96
- **FAQ Entries:** 34
- **MicroSim Specs:** 6

## 🎨 Teaching Philosophy

### Evidence-Based Learning Principles

**Concrete Before Abstract** - Real-world examples before theory

**Intrinsic Motivation** - Connect concepts to real costs and production problems

**Low Floor, High Ceiling** - Accessible entry points with unlimited depth

**Socratic Coaching** - Questions guide discovery rather than spoon-feeding answers

**Active Learning** - "Try It" exercises and interactive explorations

## 📝 Assessment Framework

### Formative Assessment
- "Try It" exercises embedded in chapters
- Reflection questions prompting deeper thinking
- Practice quizzes with immediate feedback

### Summative Assessment
- 40 quiz questions aligned with Bloom's taxonomy
- Portfolio projects (ETL pipeline, dimensional model design)
- Code review participation (for bootcamp cohorts)

## 🚀 Deployment to GitHub Pages

### Automatic Deployment (Recommended)

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy MkDocs

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: 3.x
      - run: pip install mkdocs-material mkdocs-minify-plugin
      - run: mkdocs gh-deploy --force
```

### Manual Deployment

```bash
# Build and deploy to gh-pages branch
mkdocs gh-deploy

# GitHub Pages will automatically serve from gh-pages branch
```

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Contribution Guidelines

- Follow existing markdown formatting
- Add examples to glossary entries
- Ensure quiz questions have balanced answer distribution
- Test locally with `mkdocs serve` before submitting PR

## 📄 License

This course content is licensed under the [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/).

You are free to:
- **Share** — copy and redistribute the material
- **Adapt** — remix, transform, and build upon the material

Under the following terms:
- **Attribution** — Give appropriate credit
- **ShareAlike** — Distribute adaptations under the same license

## 📬 Contact & Support

- **Issues:** [GitHub Issues](https://github.com/yourusername/data-engineering-course/issues)
- **Discussions:** [GitHub Discussions](https://github.com/yourusername/data-engineering-course/discussions)
- **Email:** your.email@example.com

## 🌟 Acknowledgments

- **Design:** Based on 12-week Data Engineering Bootcamp curriculum
- **Pedagogy:** Evidence-based learning science principles
- **Format:** Built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/)

## 🗺️ What's Next?

After completing Weeks 1-4, continue with:

- **Weeks 5-6:** Data Pipeline Fundamentals (ETL, Apache Airflow, data quality)
- **Weeks 7-8:** Big Data & Distributed Systems (Spark, Kafka, cloud processing)
- **Weeks 9-10:** Cloud Data Engineering (GCP, IaC, CI/CD)
- **Weeks 11-12:** Team Capstone Project (end-to-end platform)

---

**Version:** 1.0.0
**Last Updated:** January 28, 2026
**Status:** Production Ready ✅

**[Start Learning Now →](https://yourusername.github.io/data-engineering-course/)**
