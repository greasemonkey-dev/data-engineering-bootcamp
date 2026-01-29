# Chapter 11: Capstone Project - Design & Architecture

## Learning Objectives
By the end of this chapter, you will be able to:
1. **Design** an end-to-end data platform architecture incorporating all concepts from the bootcamp
2. **Evaluate** technology trade-offs and justify architectural decisions in technical design documents
3. **Plan** a data engineering project using Agile methodology with appropriate scope for a 2-week implementation
4. **Collaborate** in teams using GitHub workflows, code reviews, and technical communication

## Introduction

Welcome to the final stretch. You've spent 10 weeks learning Python, SQL, Git, Docker, databases, warehousing, ETL, Airflow, data quality, Spark, Kafka, cloud platforms, and infrastructure as code.

Now it's time to put it all together.

For the next two weeks, you'll build a real data platform from scratch. Not a tutorial. Not a guided exercise. A real system that ingests data, transforms it, stores it, monitors it, and serves it to end users.

This is your chance to make every mistake we've talked about—and fix them before you're doing it for a paycheck.

## The Capstone Project: Real-Time E-Commerce Analytics Platform

### Project Overview

Build a data platform that processes e-commerce events in real-time and powers business intelligence dashboards.

**Data Sources:**
- User clickstream events (Kafka topic, ~1000 events/second)
- Product catalog (PostgreSQL database, updated hourly)
- Order transactions (REST API, ~50 orders/minute)

**Requirements:**
1. **Streaming ingestion:** Consume Kafka clickstream events in real-time
2. **Batch integration:** Load product catalog and orders on schedule
3. **Data warehouse:** Store clean, modeled data in BigQuery
4. **Data quality:** Validate all incoming data, track data quality metrics
5. **Orchestration:** Use Airflow to manage all batch workflows
6. **Infrastructure:** All infrastructure defined in Terraform
7. **CI/CD:** Automated testing and deployment via GitHub Actions
8. **Monitoring:** Dashboards showing pipeline health and data quality

**Deliverables:**
- Working data platform (code in GitHub)
- Architecture diagram
- Technical design document
- 15-minute presentation
- Live demo

### Success Criteria

**Technical:**
- ✅ Ingests all three data sources successfully
- ✅ Processes streaming data with <5 second latency
- ✅ Maintains data quality >99%
- ✅ Infrastructure fully automated (one command to deploy)
- ✅ All tests passing in CI/CD
- ✅ Monitoring shows green health

**Professional:**
- ✅ Code review process followed
- ✅ Git commit messages are clear
- ✅ Documentation explains design decisions
- ✅ Presentation is polished

## Week 1: Design & Setup

### Day 1-2: Architecture Design

**Task 1: Draw the architecture**

Create a diagram showing:
- Data sources
- Ingestion layer (Kafka consumers, API clients)
- Processing layer (Dataflow, Spark)
- Storage layer (BigQuery, GCS)
- Orchestration (Airflow)
- Monitoring (Stackdriver, Great Expectations)

Tools: draw.io, Lucidchart, or pen and paper

**Task 2: Technology selection**

For each component, choose technologies and justify:

| Component | Options | Your Choice | Justification |
|-----------|---------|-------------|---------------|
| Streaming ingestion | Dataflow, Spark Streaming, Python consumer | ? | Why? |
| Batch orchestration | Airflow, Cloud Composer, Prefect | ? | Why? |
| Data warehouse | BigQuery, Snowflake, Redshift | ? | Why? |
| Infrastructure | Terraform, Pulumi, manual | ? | Why? |

**Task 3: Write design document**

Use this template:

```markdown
# E-Commerce Analytics Platform Design

## 1. Executive Summary
[2-3 sentences: What are you building and why?]

## 2. Requirements
- Streaming: [Details]
- Batch: [Details]
- Data quality: [Details]
- Scale: [Details]

## 3. Architecture
[Diagram + explanation]

## 4. Technology Choices
### Streaming
- **Choice:** [Technology]
- **Rationale:** [Why this vs alternatives]
- **Trade-offs:** [What you're giving up]

[Repeat for each component]

## 5. Data Model
[Schema diagrams for warehouse tables]

## 6. Data Quality
[What checks, where, how failures are handled]

## 7. Monitoring
[What metrics, what alerts]

## 8. Risks & Mitigation
[What could go wrong, how you'll handle it]

## 9. Timeline
[What gets built when over 2 weeks]
```

**Deliverable:** Design document in `docs/design.md`

### Day 3-4: Repository Setup & Infrastructure

**Task 1: GitHub repository structure**

```
ecommerce-data-platform/
├── .github/
│   └── workflows/
│       ├── test.yml
│       └── deploy.yml
├── terraform/
│   ├── environments/
│   │   ├── dev/
│   │   └── prod/
│   └── modules/
├── airflow/
│   ├── dags/
│   ├── plugins/
│   └── tests/
├── streaming/
│   ├── consumers/
│   ├── processors/
│   └── tests/
├── sql/
│   ├── schemas/
│   └── transformations/
├── tests/
│   ├── integration/
│   └── unit/
├── docs/
│   ├── design.md
│   ├── runbook.md
│   └── architecture.png
├── requirements.txt
├── Makefile
└── README.md
```

**Task 2: Terraform infrastructure**

Create base infrastructure:
- BigQuery datasets (raw, staging, prod)
- GCS buckets (data lake, logs)
- Cloud Composer environment (if using)
- Service accounts with minimal permissions
- Pub/Sub topics

```hcl
# terraform/environments/dev/main.tf
module "bigquery" {
  source = "../../modules/bigquery"

  dataset_id = "ecommerce_raw"
  location   = "us-central1"
}

module "storage" {
  source = "../../modules/storage"

  bucket_name = "ecommerce-data-lake-dev"
  location    = "us-central1"
}
```

Run: `terraform apply`

**Task 3: CI/CD pipeline**

```yaml
# .github/workflows/test.yml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Lint
        run: |
          pip install flake8
          flake8 airflow/ streaming/ --max-line-length=100

      - name: Unit tests
        run: pytest tests/unit/

      - name: DAG validation
        run: pytest tests/test_dags.py
```

**Deliverable:** Infrastructure deployed, CI passing

### Day 5: Data Modeling

**Task 1: Design warehouse schema**

Create dimensional model:

```sql
-- Fact table: user events
CREATE TABLE `ecommerce_prod.fact_events` (
    event_id STRING NOT NULL,
    event_timestamp TIMESTAMP NOT NULL,
    user_id STRING,
    session_id STRING,
    event_type STRING,
    page_url STRING,
    product_id STRING,
    -- Foreign keys
    date_key INT64,
    user_key INT64,
    product_key INT64
)
PARTITION BY DATE(event_timestamp)
CLUSTER BY user_id, event_type;

-- Dimension: users
CREATE TABLE `ecommerce_prod.dim_users` (
    user_key INT64 NOT NULL,
    user_id STRING NOT NULL,
    email STRING,
    signup_date DATE,
    country STRING,
    -- SCD Type 2
    valid_from TIMESTAMP,
    valid_to TIMESTAMP,
    is_current BOOLEAN
);

-- Dimension: products
CREATE TABLE `ecommerce_prod.dim_products` (
    product_key INT64 NOT NULL,
    product_id STRING NOT NULL,
    product_name STRING,
    category STRING,
    price FLOAT64,
    valid_from TIMESTAMP,
    valid_to TIMESTAMP,
    is_current BOOLEAN
);
```

**Task 2: Define data quality rules**

```python
# tests/data_quality/test_events.py
import great_expectations as gx

def test_fact_events_quality():
    """Validate fact_events table"""
    df = read_bigquery("SELECT * FROM ecommerce_prod.fact_events")
    expectations = gx.from_pandas(df)

    # Required columns not null
    expectations.expect_column_values_to_not_be_null('event_id')
    expectations.expect_column_values_to_not_be_null('event_timestamp')

    # Valid event types
    expectations.expect_column_values_to_be_in_set(
        'event_type',
        ['page_view', 'add_to_cart', 'purchase', 'search']
    )

    # Timestamps are recent (not in future, not too old)
    expectations.expect_column_values_to_be_between(
        'event_timestamp',
        min_value=datetime.now() - timedelta(days=7),
        max_value=datetime.now() + timedelta(hours=1)
    )
```

**Deliverable:** Schema DDL in `sql/schemas/`, data quality tests

## Week 2: Implementation & Presentation

### Day 6-8: Build Core Pipelines

**Priority order:**
1. Streaming ingestion (most critical)
2. Batch ETL for dimensions
3. Data quality monitoring
4. Dashboards

**Streaming pipeline (Day 6):**

```python
# streaming/consumers/clickstream_consumer.py
from kafka import KafkaConsumer
from google.cloud import bigquery
import json

def process_events():
    consumer = KafkaConsumer(
        'clickstream',
        bootstrap_servers=['kafka:9092'],
        value_deserializer=lambda m: json.loads(m.decode('utf-8'))
    )

    client = bigquery.Client()
    table_id = "ecommerce_prod.fact_events"
    errors = []

    for message in consumer:
        try:
            event = message.value
            validate_event(event)
            rows_to_insert = [transform_event(event)]

            errors = client.insert_rows_json(table_id, rows_to_insert)
            if errors:
                log_to_dlq(event, errors)

        except Exception as e:
            log_to_dlq(event, str(e))

def validate_event(event):
    """Raise ValueError if invalid"""
    required = ['event_id', 'event_timestamp', 'event_type']
    for field in required:
        if field not in event:
            raise ValueError(f"Missing required field: {field}")
```

**Batch ETL (Day 7):**

```python
# airflow/dags/product_catalog_etl.py
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def extract_products():
    """Extract from PostgreSQL source"""
    conn = get_postgres_connection()
    df = pd.read_sql("SELECT * FROM products", conn)
    df.to_parquet('/tmp/products.parquet')

def load_to_warehouse():
    """Load to BigQuery with SCD Type 2"""
    df = pd.read_parquet('/tmp/products.parquet')

    # Detect changes, update SCD
    update_dimension_table(
        df,
        table_id='ecommerce_prod.dim_products',
        key_column='product_id',
        scd_type=2
    )

with DAG('product_catalog_etl', schedule_interval='0 * * * *') as dag:
    extract = PythonOperator(task_id='extract', python_callable=extract_products)
    load = PythonOperator(task_id='load', python_callable=load_to_warehouse)

    extract >> load
```

**Deliverable:** Pipelines working, data flowing to warehouse

### Day 9-10: Testing & Monitoring

**Integration tests:**

```python
# tests/integration/test_end_to_end.py
def test_clickstream_to_warehouse():
    """Test event flows from Kafka to BigQuery"""
    # Publish test event to Kafka
    test_event = {
        'event_id': 'test-123',
        'event_timestamp': datetime.now().isoformat(),
        'event_type': 'page_view',
        'user_id': 'test-user'
    }
    publish_to_kafka('clickstream', test_event)

    # Wait for processing
    time.sleep(10)

    # Verify in BigQuery
    query = """
        SELECT * FROM ecommerce_prod.fact_events
        WHERE event_id = 'test-123'
    """
    results = bigquery_client.query(query).result()
    assert len(list(results)) == 1
```

**Monitoring dashboard:**

Create Looker/Data Studio dashboard showing:
- Events processed per minute
- Pipeline lag (streaming and batch)
- Data quality metrics (% passing validation)
- Error rates
- Cost per day

**Deliverable:** All tests passing, monitoring live

### Day 11-12: Documentation & Presentation

**Runbook (`docs/runbook.md`):**

```markdown
# Production Runbook

## Deployment
```bash
# Deploy infrastructure
cd terraform/environments/prod
terraform apply

# Deploy Airflow DAGs
gcloud composer environments storage dags import ...

# Start streaming consumer
docker-compose up -d clickstream-consumer
```

## Monitoring
- Dashboard: [URL]
- Alerts: Slack #data-alerts
- Logs: Stackdriver [URL]

## Common Issues

### Issue: Consumer lag growing
**Symptom:** Kafka consumer lag > 10,000
**Resolution:**
1. Check consumer logs
2. Scale consumers horizontally
3. Verify BigQuery isn't rate-limited

### Issue: Airflow DAG failing
**Symptom:** DAG red in Airflow UI
**Resolution:**
1. Check task logs
2. Verify source data availability
3. Check Great Expectations validation results
```

**Presentation (15 minutes):**

1. **Problem (2 min):** What business need does this solve?
2. **Architecture (3 min):** Show diagram, explain data flow
3. **Demo (5 min):** Live system, show data flowing
4. **Challenges (3 min):** What went wrong, how you fixed it
5. **Learnings (2 min):** What you'd do differently next time

**Deliverable:** Polished presentation, working demo

## Team Collaboration

### Pull Request Process

1. **Create feature branch:** `git checkout -b feature/streaming-consumer`
2. **Implement & test locally:** Make it work
3. **Open PR:** Title: "Add Kafka clickstream consumer"
4. **Code review:** At least 1 approval required
5. **Address feedback:** Make changes if requested
6. **Merge:** Squash and merge to main
7. **CI/CD deploys automatically**

### Code Review Checklist

**Reviewer checks:**
- ✅ Code is readable and documented
- ✅ Tests are included
- ✅ No secrets in code
- ✅ Error handling present
- ✅ Follows project conventions

**Author provides:**
- Description of changes
- Testing notes
- Screenshots/demo (if applicable)

## Reflection Questions

1. What's the riskiest part of your architecture? How are you mitigating that risk?

2. If your streaming consumer falls behind during a traffic spike, what happens?

3. How will you know if data quality degrades? What alerts will fire?

## Next Steps

Next chapter: Implementation best practices, code review process, and presentation prep.
