# Chapter 2: DevOps Foundations - Git & Docker

## Learning Objectives
By the end of this chapter, you will be able to:
1. **Implement** Git branching strategies and resolve merge conflicts safely in team environments
2. **Evaluate** when to use merge vs. rebase for integrating code changes
3. **Build** containerized applications using Docker with appropriate base images and multi-stage builds
4. **Create** multi-container environments with Docker Compose for local development and testing

## Introduction

It's Thursday afternoon. You and Sarah are both working on the same data pipeline. You're adding a new data source. She's fixing a bug in the transformation logic. You both modify `etl_pipeline.py`. You both push your changes.

```
! [rejected] main -> main (non-fast-forward)
error: failed to push some refs to 'origin'
```

Your heart sinks. What now? Do you force push and overwrite Sarah's work? Do you manually copy-paste your changes into her version? Do you give up and become a goat farmer?

Twenty minutes later, you've successfully merged your changes. But your pipeline that worked perfectly on your laptop throws errors in production: `ModuleNotFoundError: No module named 'pandas'`. You check—pandas is definitely in `requirements.txt`. Your DevOps engineer sighs and says the words every developer dreads: "Works on my machine."

These aren't just annoyances. They're **reliability risks**. A production data pipeline that breaks because of conflicting changes or environment differences can cost your company thousands of dollars per hour. This chapter is about making your code **collaborative** (Git) and **reproducible** (Docker).

By the end, you'll understand not just how to use these tools, but when and why different approaches matter. Let's start with Git—the tool that prevents you from accidentally destroying your teammate's work.

## Section 1: Git Branching - Safe Parallel Development

### Key Idea
Branches let multiple people work on the same codebase simultaneously without interfering with each other. They're essential for team collaboration and safe deployment workflows.

### Example: Feature Development in a Team Environment

You're on a data engineering team building an ETL pipeline. The current production pipeline is in the `main` branch. You need to add a new feature: loading data from Snowflake. Meanwhile, your teammate needs to fix a bug in the PostgreSQL connector.

**Without Branches (Dangerous):**
```bash
# Both of you work directly on main
# You modify db_connectors.py
git add db_connectors.py
git commit -m "Add Snowflake connector"
git push

# Your teammate tries to push their PostgreSQL fix
git push
# Error! Conflict! Now you need to coordinate who pushes first
```

**With Branches (Safe):**
```bash
# You create a feature branch
git checkout -b feature/snowflake-connector

# Make your changes
vim db_connectors.py

# Commit to YOUR branch
git add db_connectors.py
git commit -m "Add Snowflake connector with connection pooling"
git push -u origin feature/snowflake-connector

# Meanwhile, your teammate works on THEIR branch
# (on their machine)
git checkout -b fix/postgres-timeout
# ... make changes ...
git push -u origin fix/postgres-timeout
```

Now you both have isolated workspaces. Your teammate's changes don't affect yours until you explicitly merge them.

### Branching Strategy: Feature Branch Workflow

A common pattern in data engineering teams:

```
main (production-ready code)
├── develop (integration branch)
│   ├── feature/snowflake-connector (you)
│   ├── fix/postgres-timeout (teammate 1)
│   └── feature/data-quality-checks (teammate 2)
```

**The Workflow:**

```bash
# 1. Start from develop
git checkout develop
git pull origin develop

# 2. Create feature branch
git checkout -b feature/snowflake-connector

# 3. Work on your feature (many commits over several days)
git add src/connectors/snowflake.py
git commit -m "Add basic Snowflake connector"

git add tests/test_snowflake.py
git commit -m "Add tests for Snowflake connector"

git add src/connectors/snowflake.py
git commit -m "Add connection pooling to Snowflake"

# 4. Push your branch
git push -u origin feature/snowflake-connector

# 5. Open a Pull Request (PR) on GitHub/GitLab
# Team reviews your code

# 6. After approval, merge to develop
git checkout develop
git merge feature/snowflake-connector

# 7. Delete feature branch (clean up)
git branch -d feature/snowflake-connector
git push origin --delete feature/snowflake-connector
```

### Why This Matters

Branches enable:
- **Parallel work**: Multiple features in progress simultaneously
- **Code review**: PRs let teammates review before merging
- **Safe experimentation**: Try ideas without breaking production
- **Easy rollback**: If a feature breaks, just remove the branch

In data engineering, this is critical because:
- Pipelines run on schedules—you can't break production at 3 AM
- Data transformations are complex—code review catches logic errors
- Schema changes are risky—branches let you test migrations safely

### Try It

Create a simple project with a branch:

```bash
# Initialize a repo
mkdir my-pipeline && cd my-pipeline
git init

# Create initial file
echo "# ETL Pipeline" > README.md
git add README.md
git commit -m "Initial commit"

# Create a feature branch
git checkout -b feature/add-logging

# Make changes
echo "import logging" > pipeline.py
git add pipeline.py
git commit -m "Add logging module"

# Switch back to main
git checkout main

# Notice pipeline.py doesn't exist here
ls  # Only README.md

# Switch back to feature branch
git checkout feature/add-logging
ls  # Now pipeline.py exists
```

See how branches create isolated workspaces? Your changes in one branch don't affect other branches until you merge.

## Section 2: Merge vs. Rebase - Two Ways to Integrate Changes

### Key Idea
Both merge and rebase integrate changes from one branch to another, but they create different commit histories. Understanding when to use each is crucial for maintaining a clean, readable Git history.

### Example: Integrating Your Feature into Main

You've been working on `feature/snowflake-connector` for three days. Meanwhile, your team has merged five other PRs into `main`. Now you need to integrate your work.

**Your branch history:**
```
A---B---C---D  main
     \
      E---F---G  feature/snowflake-connector
```

**Option 1: Merge (Preserves History)**

```bash
git checkout main
git merge feature/snowflake-connector
```

**Result:**
```
A---B---C---D---H  main
     \         /
      E---F---G  feature/snowflake-connector
```

Creates a **merge commit** (H) that ties the histories together. The history shows exactly when and how branches were integrated.

**Option 2: Rebase (Linear History)**

```bash
git checkout feature/snowflake-connector
git rebase main
```

**Result:**
```
A---B---C---D---E'---F'---G'  feature/snowflake-connector
```

**Replays** your commits (E, F, G) on top of the latest main. The prime marks (E', F', G') indicate these are new commits with different hashes—they have the same changes but different parent commits.

### Visual Comparison with Real Code

Imagine this scenario:

**Initial state:**
```bash
# main branch: pipeline.py
def load_data():
    conn = connect_postgres()
    return conn.query("SELECT * FROM users")
```

**Your feature branch:**
```bash
# feature/snowflake-connector: pipeline.py
def load_data(source='postgres'):
    if source == 'postgres':
        conn = connect_postgres()
    elif source == 'snowflake':
        conn = connect_snowflake()  # Your addition
    return conn.query("SELECT * FROM users")
```

**Meanwhile, main branch evolved:**
```bash
# main branch: pipeline.py (someone added error handling)
def load_data():
    try:
        conn = connect_postgres()
        return conn.query("SELECT * FROM users")
    except ConnectionError as e:
        log_error(e)
        raise
```

**With merge:**
- Git creates a merge commit that combines both changes
- Both histories are preserved
- The merge commit shows the integration point

**With rebase:**
- Your commits are replayed on top of the new main
- Git tries to apply your changes to the new code
- The history looks like you made your changes AFTER the error handling was added

### When to Use Each

**Use Merge When:**
- Working on a team with many contributors
- You want to preserve the exact timeline of when changes were made
- Working on long-lived feature branches
- The branch has already been pushed and others might be using it

```bash
# Safe for shared branches
git checkout main
git merge feature/snowflake-connector
```

**Use Rebase When:**
- Updating your local feature branch with latest main before pushing
- You want a clean, linear history
- Working on a short-lived personal branch
- The branch has NOT been shared with others yet

```bash
# Good for local work-in-progress
git checkout feature/snowflake-connector
git rebase main
# Now your changes are on top of latest main
```

**Golden Rule of Rebasing:**
> **Never rebase commits that have been pushed to a shared branch and that others may have based work on.**

Why? Rebase rewrites history (creates new commits). If someone else has pulled your old commits and built on top of them, rebasing causes chaos.

### Handling Conflicts

Both merge and rebase can cause conflicts. Here's how to handle them:

**Merge Conflict:**
```bash
git checkout main
git merge feature/snowflake-connector

# Conflict!
Auto-merging pipeline.py
CONFLICT (content): Merge conflict in pipeline.py
Automatic merge failed; fix conflicts and then commit the result.

# Check conflicting file
cat pipeline.py
```

```python
def load_data():
<<<<<<< HEAD
    # Current main branch code
    try:
        conn = connect_postgres()
=======
    # Your feature branch code
    if source == 'postgres':
        conn = connect_postgres()
    elif source == 'snowflake':
        conn = connect_snowflake()
>>>>>>> feature/snowflake-connector
    return conn.query("SELECT * FROM users")
```

**Resolution:**
```python
# Edit file to combine both changes
def load_data(source='postgres'):
    try:
        if source == 'postgres':
            conn = connect_postgres()
        elif source == 'snowflake':
            conn = connect_snowflake()
        return conn.query("SELECT * FROM users")
    except ConnectionError as e:
        log_error(e)
        raise
```

```bash
# Mark as resolved
git add pipeline.py
git commit -m "Merge feature/snowflake-connector with error handling"
```

### Why This Matters

In data engineering:
- **Rebase** keeps your ETL pipeline's Git history clean and readable—important when debugging production issues
- **Merge** preserves the exact history of when data schema changes were integrated—crucial for audit trails
- **Conflicts** happen frequently with SQL query changes or schema migrations—knowing how to resolve them prevents deployment delays

### Try It

Create a conflict and resolve it:

```bash
# On main branch
echo "version = 1.0" > config.py
git add config.py
git commit -m "Set version 1.0"

# Create feature branch
git checkout -b feature/add-config
echo "version = 1.0\ndatabase = 'postgres'" > config.py
git add config.py
git commit -m "Add database config"

# Back to main, make conflicting change
git checkout main
echo "version = 1.1" > config.py
git add config.py
git commit -m "Bump version to 1.1"

# Now try to merge
git merge feature/add-config
# You'll get a conflict! Resolve it by combining both changes
```

## Section 3: Docker Basics - Reproducible Environments

### Key Idea
Containers package your application with all its dependencies, ensuring it runs the same way everywhere. This eliminates "works on my machine" problems and makes deployments predictable.

### Example: The "Works on My Machine" Problem

You've built a data pipeline:

```python
# pipeline.py
import pandas as pd
import psycopg2
from snowflake.connector import connect

def extract_data():
    # Extract from PostgreSQL
    conn = psycopg2.connect("postgresql://localhost/mydb")
    df = pd.read_sql("SELECT * FROM orders", conn)
    return df

def transform_data(df):
    # Data transformations
    return df.groupby('customer_id').agg({'amount': 'sum'})

def load_data(df):
    # Load to Snowflake
    sf_conn = connect(user='user', password='pass', account='account')
    # ...
```

**On your laptop:**
```bash
python pipeline.py
# Works perfectly!
```

**On the production server:**
```bash
python pipeline.py
# ImportError: No module named 'pandas'
# Oh, right, need to install dependencies

pip install pandas psycopg2 snowflake-connector-python
python pipeline.py
# ModuleNotFoundError: No module named 'psycopg2'
# Wait, psycopg2 needs PostgreSQL client libraries

apt-get install libpq-dev
pip install psycopg2
python pipeline.py
# ImportError: cannot import name 'connect' from 'snowflake.connector'
# Different version of snowflake-connector installed
```

This is a nightmare. What if you could package your entire environment—Python version, dependencies, system libraries—into a single deployable unit?

### Enter Docker: Your Environment in a Box

**Dockerfile** - A recipe for building your environment:

```dockerfile
# Start with Python 3.10 on Ubuntu
FROM python:3.10-slim

# Install system dependencies
RUN apt-get update && apt-get install -y \
    libpq-dev \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Copy requirements first (Docker caching optimization)
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY pipeline.py .
COPY config.py .

# Set environment variables
ENV PYTHONUNBUFFERED=1

# Run the pipeline
CMD ["python", "pipeline.py"]
```

**Build and run:**
```bash
# Build the container image
docker build -t my-pipeline:v1 .

# Run it
docker run my-pipeline:v1
# Runs exactly the same on any machine with Docker installed
```

### Understanding Docker Concepts

**Image vs. Container:**
- **Image**: A blueprint (like a class in OOP)—immutable, shareable
- **Container**: A running instance (like an object)—temporary, isolated

```bash
# Build image (once)
docker build -t my-pipeline:v1 .

# Run multiple containers from same image
docker run --name pipeline-run-1 my-pipeline:v1
docker run --name pipeline-run-2 my-pipeline:v1
docker run --name pipeline-run-3 my-pipeline:v1
```

**Layers and Caching:**

Each line in a Dockerfile creates a layer. Docker caches layers that haven't changed:

```dockerfile
FROM python:3.10-slim          # Layer 1 (cached if not changed)
RUN apt-get update ...         # Layer 2 (cached if not changed)
COPY requirements.txt .        # Layer 3 (cached if requirements.txt unchanged)
RUN pip install -r ...         # Layer 4 (cached if Layer 3 cached)
COPY pipeline.py .             # Layer 5 (rebuilt if pipeline.py changed)
```

This is why we copy `requirements.txt` before `pipeline.py`—if you only change your code, Docker doesn't need to reinstall dependencies.

### Multi-Stage Builds for Efficiency

Problem: Your Docker image is 2 GB because it includes build tools you don't need at runtime.

Solution: Multi-stage builds.

```dockerfile
# Stage 1: Build environment (includes compilers, build tools)
FROM python:3.10 AS builder

WORKDIR /app

COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Runtime environment (minimal)
FROM python:3.10-slim

# Copy only the installed packages from builder
COPY --from=builder /root/.local /root/.local

WORKDIR /app
COPY pipeline.py .

# Make sure scripts in .local are usable
ENV PATH=/root/.local/bin:$PATH

CMD ["python", "pipeline.py"]
```

**Result:** Image size reduced from 2 GB to 400 MB. Faster deployments, lower storage costs.

### Volumes: Persisting Data

Containers are **ephemeral**—when they stop, data inside them is lost. For data pipelines, you need persistent storage.

```bash
# Run pipeline with volume mount
docker run -v /local/data:/app/data my-pipeline:v1

# Now the container can read/write to /app/data
# And the data persists in /local/data on your host machine
```

**In Python:**
```python
# pipeline.py
def save_results(df):
    # This saves to /app/data inside container
    # Which maps to /local/data on host
    df.to_csv('/app/data/results.csv', index=False)
```

### Environment Variables for Configuration

Don't hardcode credentials in Dockerfiles:

```dockerfile
# Bad: Credentials in image (security risk!)
ENV DB_PASSWORD=secret123

# Good: Pass at runtime
ENV DB_PASSWORD=
```

```bash
# Pass secrets at runtime
docker run \
    -e DB_HOST=postgres.example.com \
    -e DB_PASSWORD=secret123 \
    my-pipeline:v1
```

**Or use environment file:**
```bash
# .env file
DB_HOST=postgres.example.com
DB_USER=data_engineer
DB_PASSWORD=secret123

# Run with env file
docker run --env-file .env my-pipeline:v1
```

### Why This Matters

For data engineers:
- **Reproducibility**: Your pipeline runs identically in dev, staging, and production
- **Isolation**: Different pipelines can use different Python versions without conflicts
- **Portability**: Ship your pipeline to any cloud (AWS, GCP, Azure) or on-prem
- **Scalability**: Orchestrators like Kubernetes run containers—containerizing is the first step to scaling

### Try It

Create a simple containerized ETL script:

```bash
# Create project directory
mkdir docker-pipeline && cd docker-pipeline

# Create a simple pipeline
cat > pipeline.py << 'EOF'
import pandas as pd
import sys

print(f"Python version: {sys.version}")
print(f"Pandas version: {pd.__version__}")

data = {'name': ['Alice', 'Bob'], 'age': [25, 30]}
df = pd.DataFrame(data)
print(df)
EOF

# Create requirements
echo "pandas==2.0.0" > requirements.txt

# Create Dockerfile
cat > Dockerfile << 'EOF'
FROM python:3.10-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY pipeline.py .
CMD ["python", "pipeline.py"]
EOF

# Build and run
docker build -t simple-pipeline .
docker run simple-pipeline
```

## Section 4: Docker Compose - Multi-Container Orchestration

### Key Idea
Real-world applications involve multiple services (database, app, cache). Docker Compose lets you define and run multi-container setups with a single command.

### Example: ETL Pipeline with PostgreSQL Database

Your pipeline needs a PostgreSQL database. Instead of installing PostgreSQL on your laptop, you can run it in a container alongside your pipeline.

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # PostgreSQL database
  postgres:
    image: postgres:14
    environment:
      POSTGRES_USER: dataeng
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: analytics
    volumes:
      # Persist data even when container stops
      - postgres_data:/var/lib/postgresql/data
      # Load initial schema
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U dataeng"]
      interval: 5s
      timeout: 5s
      retries: 5

  # ETL pipeline
  pipeline:
    build: .
    environment:
      DB_HOST: postgres  # Docker Compose creates network, use service name
      DB_USER: dataeng
      DB_PASSWORD: secret
      DB_NAME: analytics
    depends_on:
      postgres:
        condition: service_healthy
    volumes:
      - ./output:/app/output

volumes:
  postgres_data:  # Named volume for persistence
```

**pipeline.py (updated to use environment variables):**
```python
import os
import psycopg2
import pandas as pd

def get_db_connection():
    return psycopg2.connect(
        host=os.getenv('DB_HOST', 'localhost'),
        user=os.getenv('DB_USER', 'dataeng'),
        password=os.getenv('DB_PASSWORD'),
        database=os.getenv('DB_NAME', 'analytics')
    )

def extract_data():
    conn = get_db_connection()
    query = """
        SELECT
            customer_id,
            SUM(amount) as total_spent,
            COUNT(*) as order_count
        FROM orders
        WHERE order_date >= CURRENT_DATE - INTERVAL '30 days'
        GROUP BY customer_id
    """
    df = pd.read_sql(query, conn)
    conn.close()
    return df

def transform_data(df):
    # Add customer segment
    df['segment'] = pd.cut(
        df['total_spent'],
        bins=[0, 100, 500, float('inf')],
        labels=['low', 'medium', 'high']
    )
    return df

def load_data(df):
    # Save to CSV in mounted volume
    output_path = '/app/output/customer_segments.csv'
    df.to_csv(output_path, index=False)
    print(f"Results saved to {output_path}")

if __name__ == "__main__":
    print("Starting ETL pipeline...")
    raw_data = extract_data()
    transformed_data = transform_data(raw_data)
    load_data(transformed_data)
    print("Pipeline completed successfully!")
```

**init.sql (creates initial schema):**
```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    order_date DATE NOT NULL
);

-- Insert sample data
INSERT INTO orders (customer_id, amount, order_date) VALUES
(1, 150.00, CURRENT_DATE - INTERVAL '5 days'),
(1, 200.00, CURRENT_DATE - INTERVAL '3 days'),
(2, 50.00, CURRENT_DATE - INTERVAL '10 days'),
(2, 75.00, CURRENT_DATE - INTERVAL '2 days'),
(3, 1000.00, CURRENT_DATE - INTERVAL '1 day');
```

**Run everything:**
```bash
# Start all services
docker-compose up

# Run in background
docker-compose up -d

# View logs
docker-compose logs -f pipeline

# Stop everything
docker-compose down

# Stop and remove volumes (fresh start)
docker-compose down -v
```

### Docker Compose Features

**Networking:**
Services can communicate using service names:
```python
# In pipeline.py, use 'postgres' as hostname
conn = psycopg2.connect(host='postgres', ...)
```

Docker Compose creates a network where `postgres` resolves to the database container's IP.

**Dependency Management:**
```yaml
pipeline:
  depends_on:
    postgres:
      condition: service_healthy
```

This ensures the pipeline doesn't start until PostgreSQL is ready to accept connections.

**Environment Files:**
```yaml
services:
  pipeline:
    env_file:
      - .env.dev      # For development
      # - .env.prod   # For production
```

**Scaling:**
```bash
# Run 3 instances of pipeline
docker-compose up --scale pipeline=3
```

### Real-World Example: Complete Data Stack

A more complex setup with multiple data tools:

```yaml
version: '3.8'

services:
  # Source database (PostgreSQL)
  postgres:
    image: postgres:14
    environment:
      POSTGRES_DB: source_db
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres_data:/var/lib/postgresql/data

  # Data warehouse (PostgreSQL on different port)
  warehouse:
    image: postgres:14
    environment:
      POSTGRES_DB: warehouse
      POSTGRES_USER: analyst
      POSTGRES_PASSWORD: secret
    ports:
      - "5433:5432"
    volumes:
      - warehouse_data:/var/lib/postgresql/data

  # Redis cache
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  # ETL pipeline
  etl:
    build: .
    environment:
      SOURCE_DB_HOST: postgres
      WAREHOUSE_DB_HOST: warehouse
      REDIS_HOST: redis
    depends_on:
      - postgres
      - warehouse
      - redis

  # Jupyter for analysis
  jupyter:
    image: jupyter/datascience-notebook
    ports:
      - "8888:8888"
    environment:
      JUPYTER_ENABLE_LAB: "yes"
    volumes:
      - ./notebooks:/home/jovyan/work
    depends_on:
      - warehouse

volumes:
  postgres_data:
  warehouse_data:
```

**Start the entire data platform:**
```bash
docker-compose up -d
```

Now you have:
- Source database (localhost:5432)
- Data warehouse (localhost:5433)
- Redis cache (localhost:6379)
- Jupyter notebook (localhost:8888)
- ETL pipeline connecting them all

### Why This Matters

Docker Compose enables:
- **Local development that mirrors production**: Same services, same versions
- **Onboarding**: New team members run `docker-compose up` and have a working environment
- **Integration testing**: Test your pipeline against real databases, not mocks
- **Experimentation**: Try new tools (Redis, MongoDB) without installing them system-wide

For data engineers, this means:
- Test schema migrations locally before running in production
- Develop pipelines that interact with multiple systems (source DB, warehouse, cache)
- Share reproducible development environments with your team

### Try It

Set up a complete pipeline with database:

```bash
# Create project
mkdir compose-pipeline && cd compose-pipeline

# Create docker-compose.yml (use example above)
# Create pipeline.py (use example above)
# Create init.sql (use example above)
# Create Dockerfile

cat > Dockerfile << 'EOF'
FROM python:3.10-slim
RUN apt-get update && apt-get install -y libpq-dev gcc && rm -rf /var/lib/apt/lists/*
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY pipeline.py .
CMD ["python", "pipeline.py"]
EOF

# Create requirements.txt
cat > requirements.txt << 'EOF'
pandas==2.0.0
psycopg2-binary==2.9.6
EOF

# Create output directory
mkdir output

# Run everything
docker-compose up

# Check output
cat output/customer_segments.csv
```

## Common Pitfalls

### 1. Rebasing Shared Branches

**Problem:**
```bash
# On shared feature branch
git checkout feature/shared-work
git rebase main  # Rewrites history
git push --force  # Breaks everyone else's local copy
```

**Fix:** Only rebase local branches that haven't been pushed. For shared branches, use merge.

### 2. Forgetting .dockerignore

**Problem:**
```dockerfile
COPY . /app  # Copies EVERYTHING, including node_modules, .git, __pycache__
```

**Fix:** Create `.dockerignore`:
```
__pycache__/
*.pyc
.git/
.env
node_modules/
*.log
.DS_Store
```

### 3. Running Containers as Root

**Problem:**
```dockerfile
# Runs as root user (security risk)
CMD ["python", "pipeline.py"]
```

**Fix:** Create a non-root user:
```dockerfile
RUN useradd -m -u 1000 appuser
USER appuser
CMD ["python", "pipeline.py"]
```

### 4. Hardcoding Localhost in Database Connections

**Problem:**
```python
# Won't work inside Docker container
conn = psycopg2.connect(host='localhost', ...)
```

**Fix:** Use environment variables and service names:
```python
conn = psycopg2.connect(
    host=os.getenv('DB_HOST', 'localhost'),
    ...
)
```

## Reflection Questions

1. **When would merge be better than rebase?**

   Consider:
   - You're working on a shared feature branch with 3 other engineers
   - You want to preserve the exact timeline of when a critical bug fix was integrated
   - You need to demonstrate to auditors exactly when certain data transformations were added

   What's the cost of having a non-linear history? What do you gain from preserving it?

2. **Your Docker image is 3 GB and takes 10 minutes to build. How would you optimize it?**

   Think about:
   - Are you using the smallest appropriate base image? (alpine vs slim vs full)
   - Are you leveraging layer caching properly?
   - Could you use multi-stage builds?
   - Are you installing development dependencies in production images?

3. **You have a merge conflict in a SQL migration file that alters a production table schema. How do you resolve it safely?**

   Considerations:
   - Both changes might be valid but incompatible
   - The order of migrations matters
   - Testing the merged result is critical
   - You might need to coordinate with the other developer

4. **When would you use Docker Compose vs. Kubernetes?**

   Docker Compose is great for:
   - Local development
   - Small deployments (single server)
   - Simple multi-container apps

   Kubernetes is better for:
   - Production at scale
   - Auto-scaling based on load
   - High availability across multiple servers
   - Complex orchestration requirements

   Where does your data pipeline fall on this spectrum?

## Summary

- **Git branches enable safe parallel development**, allowing multiple team members to work on features simultaneously without interfering with each other or breaking production code.

- **Merge preserves history** and is safe for shared branches, while **rebase creates linear history** and is best for local cleanup before pushing. Never rebase shared branches.

- **Merge conflicts are inevitable in team environments**. Understanding how to resolve them—especially in data pipeline code and SQL migrations—is a critical skill.

- **Docker containers package your application with all dependencies**, eliminating "works on my machine" problems and ensuring your pipeline runs identically everywhere.

- **Docker Compose orchestrates multi-container environments**, making it easy to run complex data platforms (databases, caches, pipelines) with a single command.

- **Dockerfile best practices**—layer caching, multi-stage builds, .dockerignore, non-root users—reduce image size, improve security, and speed up deployments.

## Next Steps

You now have the tools for collaborative development (Git) and reproducible environments (Docker). In Chapter 3, we'll dive into database design and modeling. You'll learn:

- How to normalize data to prevent anomalies and redundancy
- When to denormalize for performance
- Index selection strategies for query optimization
- Designing schemas that scale from thousands to millions of rows

The Git and Docker skills you learned here will be essential as you work with database migrations, test schema changes locally with Docker Compose, and collaborate with teammates on data model designs. Your ability to safely experiment (branches) and reproduce environments (containers) will make you a more effective database designer.
