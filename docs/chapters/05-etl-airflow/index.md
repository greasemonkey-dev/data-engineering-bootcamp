# Chapter 5: ETL/ELT & Apache Airflow

## Learning Objectives
By the end of this chapter, you will be able to:
1. **Compare** ETL and ELT patterns and select the appropriate approach based on data volume, transformation complexity, and infrastructure constraints
2. **Design** Apache Airflow DAGs with proper task dependencies, error handling, and retry logic for production data pipelines
3. **Implement** idempotent data transformations that can safely re-run without corrupting data or producing duplicates
4. **Configure** monitoring and alerting for data pipelines to detect failures before users notice them

## Introduction

It's Tuesday, 11:47 PM. I'm at a bar with friends—actual non-engineer friends who think I have a "normal" job—when my Slack goes off. Then my phone. Then my personal email.

The data science team's dashboard is blank. Not showing old data. Not showing errors. Just... blank. Like someone erased the entire company's metrics.

I excuse myself, pull out my laptop (yes, I'm that person), and SSH into our data warehouse. The tables are there. The data is there. Yesterday's data. From 24 hours ago.

Our overnight ETL pipeline—the one that runs every night at 2 AM to refresh the dashboards—never ran. It just... didn't wake up. Like it hit snooze and went back to sleep.

I check our cron server. The cron job is there. I check the logs. Empty. No execution. No error. No trace that anything even tried to run.

My "normal" Tuesday evening just became an archaeology expedition through six months of half-documented bash scripts, trying to figure out which server is supposed to be running what, and why none of it is actually running.

This isn't a story about what went wrong. This is a story about how we were doing everything wrong from the start.

Here's what nobody tells you when you're learning data engineering: The transform part? That's the easy part. The hard part is making sure your transformations actually *run* every day, in the right order, handling failures gracefully, and alerting you when things go sideways *before* the CEO asks why the dashboard is blank.

Welcome to orchestration. Welcome to Apache Airflow. Welcome to the realization that "works on my laptop" is not the same as "runs reliably in production every single day for the next three years."

## Section 1: ETL vs ELT - The Great Debate I Learned the Hard Way

### The Day I Processed 500GB Of Data In Python (I Shouldn't Have)

Let me paint you a picture of hubris.

Six months into my second data engineering job, I get assigned "the integration." We're bringing in data from a new vendor—daily dumps of their entire transaction database. About 500GB per day, compressed. My job: extract it, clean it up, and load it into our warehouse.

I'd just read a blog post about ETL. Extract, Transform, Load. Made perfect sense. Extract the data, transform it in Python (because I'm a Python wizard, obviously), then load the cleaned data into the warehouse.

So I built this beautiful pipeline:
1. Download the compressed file from their S3 bucket
2. Decompress it (1.8TB uncompressed, but who's counting)
3. Read it into Pandas (because Pandas is awesome)
4. Clean it up: parse dates, fix data types, handle nulls
5. Load it into PostgreSQL

Tested it on a sample file: 50MB. Worked beautifully. Deployment day came. I was confident.

Sunday morning, 4 AM. My monitoring alert wakes me up: "ETL process killed by OOM (Out of Memory)."

OOM? Out of memory? That's impossible. Our server has 64GB of RAM. Surely that's enough for...

Oh no.

I'd tried to load 1.8TB into Pandas. On a machine with 64GB of RAM. Pandas tried valiantly to comply, filled up all the RAM, started swapping to disk, thrashing so hard the server became unresponsive, and eventually the kernel just mercy-killed the process.

The data science team woke up Monday to blank dashboards. Again.

### Here's What I Wish I'd Known: Transform Where the Data Lives

There are two philosophies in data engineering:

**ETL (Extract, Transform, Load)** - Transform data *before* loading it into the warehouse
- Extract data from source systems
- Transform it in intermediate processing (Python, Spark, etc.)
- Load the cleaned data into the warehouse

**ELT (Extract, Load, Transform)** - Transform data *after* loading it into the warehouse
- Extract data from source systems
- Load it into the warehouse as-is (raw)
- Transform it using SQL in the warehouse itself

Neither is universally better. The right choice depends on your constraints.

#### When ETL Makes Sense

Use ETL when:
- **Your source data is complex/semi-structured** (JSON, XML, logs) and needs parsing
- **Your transformations are computationally heavy** (ML models, complex parsing)
- **Your warehouse charges by the query** (BigQuery, Snowflake with per-query pricing)
- **You're loading from many small sources** and want to consolidate first

Example: Processing server logs before loading:

```python
# ETL: Transform BEFORE loading
import gzip
import json
from datetime import datetime

def parse_log_line(line):
    """Transform raw logs into structured data"""
    log = json.loads(line)
    return {
        'timestamp': datetime.fromisoformat(log['ts']),
        'user_id': log['user'],
        'event_type': log['event'],
        'page_url': log.get('page', 'unknown'),
        'duration_ms': int(log.get('duration', 0))
    }

def extract_and_transform(log_file):
    """Generator that yields transformed records"""
    with gzip.open(log_file, 'rt') as f:
        for line in f:
            try:
                yield parse_log_line(line)
            except (json.JSONDecodeError, KeyError, ValueError) as e:
                # Log errors but don't stop the pipeline
                logger.warning(f"Skipped malformed log: {e}")
                continue

# Only load clean, structured data
cleaned_logs = list(extract_and_transform('server_logs_2026-01-29.gz'))
load_to_warehouse(cleaned_logs)
```

**Why this works:** JSON parsing in Python is easier than SQL. Errors are handled gracefully. Only valid data reaches the warehouse.

#### When ELT Makes Sense

Use ELT when:
- **Your source data is already structured** (CSV, Parquet, database dumps)
- **Your warehouse is powerful** (BigQuery, Snowflake, Redshift with enough capacity)
- **You want transformation flexibility** (business logic changes frequently)
- **You need reproducibility** (can re-run transforms without re-extracting)

Example: Loading vendor data then transforming in SQL:

```sql
-- ELT: Load THEN transform

-- Step 1: Load raw data (no transformation)
CREATE TABLE raw_transactions (
    transaction_id STRING,
    customer_id STRING,
    transaction_date STRING,  -- Still a string!
    amount STRING,             -- Still a string!
    currency STRING
);

-- Load everything as-is (fast, simple)
LOAD DATA INTO raw_transactions
FROM 'gs://vendor-data/transactions_2026-01-29.csv';

-- Step 2: Transform in SQL
CREATE OR REPLACE TABLE clean_transactions AS
SELECT
    transaction_id,
    customer_id,
    PARSE_DATE('%Y-%m-%d', transaction_date) AS transaction_date,
    CAST(amount AS FLOAT64) AS amount,
    currency,
    -- Apply business logic
    CASE
        WHEN amount > 10000 THEN 'high_value'
        WHEN amount > 1000 THEN 'medium_value'
        ELSE 'low_value'
    END AS transaction_tier
FROM raw_transactions
WHERE transaction_date IS NOT NULL  -- Filter bad data
    AND amount IS NOT NULL;
```

**Why this works:** BigQuery can scan and transform terabytes in seconds. If business logic changes, just re-run the transform (no re-download). Failed transforms don't affect raw data.

### My Hard-Learned Rule

After that 4 AM disaster, here's the rule I follow:

> If your warehouse can handle the transformation faster than you can download the data, use ELT.

In my case:
- **Download + decompress:** 45 minutes for 500GB
- **BigQuery scan + transform:** 2 minutes for 1.8TB

The math was obvious once I stopped being stubborn about Python.

### Scars I've Earned: ETL/ELT Edition

**Scar #1: Tried to transform 1.8TB in Pandas**
```python
# DON'T
df = pd.read_csv('huge_file.csv')  # OOM killer has entered the chat
```
**What it cost me:** A Sunday morning, confused data scientists, and a lecture about resource limits

**Scar #2: Loaded raw data without checking first**
```sql
-- DON'T
LOAD DATA INTO production_table
FROM 'gs://untrusted-vendor/data.csv';  -- Hope and pray
```
**What it cost me:** 47 million rows of garbage data in production, a 3-hour rollback, and zero trust from my team

**Scar #3: Hardcoded credentials in the ETL script**
```python
# DON'T
conn = psycopg2.connect(
    "host=prod-db.company.com user=admin password=hunter2"  # In git history forever
)
```
**What it cost me:** A security audit finding, mandatory security training, and explaining to my manager why I committed passwords to GitHub

## Section 2: Apache Airflow - The Orchestrator I Didn't Know I Needed

### The Cron Job That Became Sentient (And Not In A Good Way)

Remember that Tuesday night at the bar? The ETL pipeline that just... didn't run? Let me tell you how I got into that mess.

When I first started building data pipelines, I used cron. Everyone uses cron, right? It's simple:

```bash
# Run ETL every day at 2 AM
0 2 * * * /home/me/etl/run_pipeline.sh
```

Clean. Simple. One line.

Then the requirements evolved:
- "Can we run this twice a day?"
- "Can we wait for the vendor file before starting?"
- "Can we retry if it fails?"
- "Can we run step 2 only if step 1 succeeds?"
- "Can we send a Slack alert if it fails?"
- "Can we see a history of what ran when?"

Six months later, I had 47 cron jobs across 3 servers, a tangle of bash scripts checking for lock files, and absolutely zero visibility into what was running, what failed, and what was just... stuck.

That Tuesday night disaster? One of our three cron servers had been rebooted for maintenance. The cron jobs didn't start. Nobody noticed for 24 hours because there was no monitoring.

My senior engineer sat me down Wednesday morning: "Have you heard of Apache Airflow?"

I hadn't.

"Let me show you something," she said.

### Here's What I Wish I'd Known: Orchestrators Are Your New Best Friend

Apache Airflow is a platform to programmatically author, schedule, and monitor workflows. Think of it as "cron on steroids with a web UI and actual error handling."

The core concepts:

**DAG (Directed Acyclic Graph):** A workflow—a collection of tasks with dependencies
- "Directed" = tasks have a specific order
- "Acyclic" = no circular dependencies (task A can't depend on task B if B depends on A)
- "Graph" = visual representation of tasks and their relationships

**Task:** A single unit of work (run a SQL query, call an API, execute Python)

**Operator:** A template for a task (PythonOperator, BashOperator, SQLOperator)

**Dependencies:** Relationships between tasks (task B runs after task A completes)

### Example: Converting Cron Chaos to Airflow Clarity

Here's what my cron disaster looked like:

```bash
# run_etl.sh - good luck maintaining this
#!/bin/bash
set -e

# Step 1: Check if vendor file exists
while true; do
    if aws s3 ls s3://vendor/data_$(date +%Y%m%d).csv; then
        break
    fi
    sleep 300  # Wait 5 minutes
done

# Step 2: Download it
aws s3 cp s3://vendor/data_$(date +%Y%m%d).csv /tmp/

# Step 3: Run transform
python /home/me/etl/transform.py /tmp/data_$(date +%Y%m%d).csv

# Step 4: Load to warehouse
python /home/me/etl/load.py /tmp/data_$(date +%Y%m%d).csv

# If we got here, send success notification
curl -X POST -H 'Content-type: application/json' \
    --data '{"text":"ETL completed"}' \
    https://hooks.slack.com/services/XXX

# If anything failed, cron will email me. Maybe. If email is configured. Is it?
```

Here's the same workflow in Airflow:

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor
from airflow.providers.postgres.operators.postgres import PostgresOperator
from airflow.operators.slack import SlackAPIPostOperator
from datetime import datetime, timedelta

# Default settings for all tasks
default_args = {
    'owner': 'data-eng-team',
    'depends_on_past': False,  # Don't wait for previous runs
    'start_date': datetime(2026, 1, 1),
    'email': ['alerts@company.com'],
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 3,  # Retry up to 3 times
    'retry_delay': timedelta(minutes=5),  # Wait 5 min between retries
}

# Define the workflow
with DAG(
    'vendor_etl_pipeline',
    default_args=default_args,
    description='Daily vendor data ETL',
    schedule_interval='0 2 * * *',  # 2 AM daily
    catchup=False,  # Don't backfill old runs
    tags=['etl', 'vendor', 'production'],
) as dag:

    # Task 1: Wait for vendor file
    wait_for_file = S3KeySensor(
        task_id='wait_for_vendor_file',
        bucket_name='vendor-data',
        bucket_key='data_{{ ds_nodash }}.csv',  # ds_nodash = YYYYMMDD
        poke_interval=300,  # Check every 5 minutes
        timeout=7200,  # Give up after 2 hours
        mode='poke',
    )

    # Task 2: Download and transform
    def extract_and_transform(**context):
        import boto3
        import pandas as pd
        from etl.transform import clean_data

        # Get execution date from Airflow context
        execution_date = context['ds_nodash']

        # Download
        s3 = boto3.client('s3')
        local_file = f'/tmp/data_{execution_date}.csv'
        s3.download_file('vendor-data', f'data_{execution_date}.csv', local_file)

        # Transform
        df = pd.read_csv(local_file, chunksize=100000)  # Process in chunks!
        cleaned = clean_data(df)
        cleaned.to_parquet(f'/tmp/cleaned_{execution_date}.parquet')

        return f'/tmp/cleaned_{execution_date}.parquet'

    transform = PythonOperator(
        task_id='extract_and_transform',
        python_callable=extract_and_transform,
        provide_context=True,
    )

    # Task 3: Load to warehouse
    load = PostgresOperator(
        task_id='load_to_warehouse',
        postgres_conn_id='prod_warehouse',  # Connection defined in Airflow UI
        sql='''
            COPY transactions
            FROM '{{ ti.xcom_pull(task_ids="extract_and_transform") }}'
            WITH (FORMAT parquet);
        ''',
    )

    # Task 4: Notify success
    notify = SlackAPIPostOperator(
        task_id='notify_success',
        slack_conn_id='slack_data_alerts',
        channel='#data-pipeline-status',
        text='✅ Vendor ETL completed successfully for {{ ds }}',
    )

    # Define dependencies: wait → transform → load → notify
    wait_for_file >> transform >> load >> notify
```

### Why This Is Better (So Much Better)

**1. Visual Dependency Graph**

Airflow renders this as a nice graph showing what runs when:

```
[wait_for_vendor_file] → [extract_and_transform] → [load_to_warehouse] → [notify_success]
```

Click on any task to see logs, duration, retry attempts, and whether it succeeded or failed.

**2. Automatic Retries**

```python
'retries': 3,
'retry_delay': timedelta(minutes=5),
```

Transient failures (network blip, database deadlock) get automatically retried. No more 3 AM wake-ups for issues that would've fixed themselves.

**3. Built-in Monitoring**

- Web UI shows all your DAGs
- Click into any DAG to see run history
- Green = success, Red = failed, Yellow = running
- Get alerts only when retries are exhausted

**4. Idempotency Awareness**

Notice this in the DAG definition:

```python
'depends_on_past': False,
```

This means "today's run doesn't depend on yesterday's success." If yesterday failed, today still runs. This is huge for recovering from failures.

**5. Templating with Jinja**

```python
bucket_key='data_{{ ds_nodash }}.csv'  # Airflow replaces with execution date
```

Airflow provides variables:
- `{{ ds }}` = execution date (YYYY-MM-DD)
- `{{ ds_nodash }}` = execution date (YYYYMMDD)
- `{{ ti }}` = task instance (for passing data between tasks)

### Scars I've Earned: Airflow Edition

**Scar #1: Set `depends_on_past=True` globally**
```python
# DON'T
default_args = {
    'depends_on_past': True,  # One failure blocks all future runs
}
```
**What it cost me:** A Monday pipeline failure that prevented all runs for the rest of the week until I manually cleared the failed task

**Scar #2: Used huge retries without exponential backoff**
```python
# DON'T
'retries': 100,
'retry_delay': timedelta(seconds=1),  # Hammer the API 100 times
```
**What it cost me:** Got our IP address blocked by the vendor's API for "abuse," had to call them and apologize

**Scar #3: Forgot `catchup=False` on a new DAG**
```python
# DON'T
with DAG(
    'new_pipeline',
    start_date=datetime(2025, 1, 1),  # 1 year ago
    schedule_interval='@daily',
    # catchup=True is the default!
)
```
**What it cost me:** Airflow immediately scheduled 365 backfill runs, overwhelmed our database, and I had to manually delete all of them

## Section 3: Idempotency - The Property That Saves Careers

### The Append That Appended Twice (Million Times)

It's Wednesday. Our daily sales report is due to the executive team by 9 AM. The report comes from a simple pipeline that runs every night:

```python
# Extract yesterday's sales
sales = extract_sales_data(yesterday)

# Load into the reporting table
append_to_table('daily_sales_report', sales)
```

Clean. Simple. Worked perfectly for six months.

Then one Tuesday night, our database had a brief network hiccup. The pipeline's `append_to_table` call failed with a timeout. Airflow's retry mechanism kicked in (good Airflow!). The insert succeeded on retry.

Wednesday morning, 7:45 AM. I'm having coffee when Slack lights up:

"Why does yesterday's revenue show $4.8M instead of $2.4M?"

Oh no.

The first attempt actually *did* succeed—the network timeout happened after the commit but before the acknowledgment reached our script. So when Airflow retried, it inserted the same data again.

Every row. Twice.

I spent the next 75 minutes trying to de-duplicate the table before the executive meeting, praying I could figure out which rows were the real ones and which were duplicates.

I couldn't. We went into the meeting with wrong numbers. The CFO made decisions based on revenue that was literally double the truth.

That afternoon, my tech lead introduced me to a word I'd never forget: "idempotent."

### Here's What I Wish I'd Known: Idempotency Is Non-Negotiable

**Idempotent** (adjective): An operation that produces the same result no matter how many times you run it.

Examples:
- **Idempotent:** `UPDATE users SET status = 'active' WHERE user_id = 123`
  - Run it once: user 123 is active
  - Run it 100 times: user 123 is still active (same result)

- **NOT idempotent:** `INSERT INTO users (user_id, status) VALUES (123, 'active')`
  - Run it once: one row inserted
  - Run it twice: two rows inserted (or duplicate key error)

In data pipelines, idempotency means:

> If the same pipeline runs twice with the same input, it produces the same output.

This is crucial because:
- Networks fail mid-operation
- Retries happen automatically
- Humans accidentally click "run" twice
- Backfills re-process old data

### Patterns for Idempotent Pipelines

#### Pattern 1: DELETE + INSERT (Not Great, But Simple)

```python
def load_sales_data_v1(date):
    """Delete existing data for this date, then insert"""
    # Delete any existing data
    execute_sql(f"DELETE FROM daily_sales_report WHERE date = '{date}'")

    # Insert new data
    sales = extract_sales_data(date)
    insert_into_table('daily_sales_report', sales)
```

**Pros:** Simple, guarantees no duplicates
**Cons:** Not atomic (delete and insert aren't in a transaction), lose data if insert fails

#### Pattern 2: MERGE / UPSERT (Better)

```sql
-- PostgreSQL MERGE (INSERT ... ON CONFLICT UPDATE)
INSERT INTO daily_sales_report (date, product_id, revenue, units_sold)
VALUES ('2026-01-29', 'PROD-123', 1500.00, 42)
ON CONFLICT (date, product_id)  -- If this combination exists
DO UPDATE SET
    revenue = EXCLUDED.revenue,
    units_sold = EXCLUDED.units_sold,
    updated_at = NOW();
```

```python
# BigQuery MERGE
MERGE INTO daily_sales_report AS target
USING new_sales_data AS source
ON target.date = source.date AND target.product_id = source.product_id
WHEN MATCHED THEN
    UPDATE SET revenue = source.revenue, units_sold = source.units_sold
WHEN NOT MATCHED THEN
    INSERT (date, product_id, revenue, units_sold)
    VALUES (source.date, source.product_id, source.revenue, source.units_sold);
```

**Pros:** Atomic, handles both inserts and updates
**Cons:** Requires unique key constraint

#### Pattern 3: Staging Table + Swap (Best for Large Loads)

```python
def load_sales_data_v3(date):
    """Load into staging, validate, then swap"""
    # Load into temporary staging table
    staging_table = f"daily_sales_staging_{date.replace('-', '')}"
    create_table(staging_table)
    insert_into_table(staging_table, extract_sales_data(date))

    # Validate data
    if not validate_sales_data(staging_table):
        raise ValueError("Data validation failed!")

    # Atomic swap
    execute_sql(f"""
        BEGIN;
        DELETE FROM daily_sales_report WHERE date = '{date}';
        INSERT INTO daily_sales_report SELECT * FROM {staging_table};
        DROP TABLE {staging_table};
        COMMIT;
    """)
```

**Pros:** Validates before affecting production, atomic transaction
**Cons:** More complex, requires extra storage for staging

#### Pattern 4: Partition Overwrite (Best for Time-Series Data)

```sql
-- BigQuery partitioned table
CREATE TABLE daily_sales_report (
    date DATE,
    product_id STRING,
    revenue FLOAT64,
    units_sold INT64
)
PARTITION BY date;

-- Idempotent load: replace only today's partition
MERGE INTO daily_sales_report AS target
USING new_sales_data AS source
ON target.date = source.date AND target.product_id = source.product_id
WHEN MATCHED THEN UPDATE SET ...
WHEN NOT MATCHED THEN INSERT ...;
```

For large datasets, BigQuery can atomically replace a single partition:

```python
# Overwrite only the partition for this date
job_config = bigquery.QueryJobConfig(
    write_disposition='WRITE_TRUNCATE',
    time_partitioning=bigquery.TimePartitioning(field='date'),
)
```

**Pros:** Efficient for time-series data, no impact on other dates
**Cons:** Requires partitioned tables

### My Idempotency Checklist

Before deploying any pipeline, I ask:

1. **Can I run this twice safely?** If not, it's not ready.
2. **What's my unique key?** (date, transaction_id, user_id + date, etc.)
3. **Am I using INSERT or MERGE?** (MERGE is almost always better)
4. **What happens if this fails halfway?** (Staging tables save lives)

### Scars I've Earned: Idempotency Edition

**Scar #1: Used APPEND mode on a retry-enabled pipeline**
```python
# DON'T
df.to_gbq('dataset.table', if_exists='append')  # With automatic retries
```
**What it cost me:** Duplicate data in production, a panicked morning de-duplication session, and executive decisions made on wrong numbers

**Scar #2: DELETE + INSERT without a transaction**
```python
# DON'T
execute_sql("DELETE FROM sales WHERE date = '2026-01-29'")
# Pipeline crashes here, data is gone forever
execute_sql("INSERT INTO sales ...")
```
**What it cost me:** Lost data when the insert failed, had to restore from backup, explaining to my manager what a transaction is

**Scar #3: Unique key wasn't actually unique**
```sql
-- DON'T
CREATE TABLE events (
    user_id INT,
    event_type STRING,
    created_at TIMESTAMP
);
-- Thought (user_id, event_type) was unique. It wasn't.
```
**What it cost me:** MERGE statement failed with "duplicate key" error, traced back to bad assumptions about data uniqueness

## Section 4: Monitoring & Alerting - Because Hope Is Not A Strategy

### The Pipeline That Failed For Three Days (Before Anyone Noticed)

Let me tell you about the most embarrassing moment of my career.

It's Thursday morning. I'm in a planning meeting. My phone buzzes. The product manager: "Hey, our user growth dashboard hasn't updated since Monday. Is something broken?"

My stomach drops.

I excuse myself, run back to my desk, check Airflow. Sure enough: our user analytics pipeline has been failing since Monday afternoon. Three days. Three. Days.

How did nobody notice for three days?

Because I didn't set up monitoring.

The pipeline ran every night. It updated a user growth dashboard that the product team checked weekly. When it failed, it just... quietly failed. No alerts. No emails. No Slack messages. The scheduler kept trying every night, failing every night, and I had no idea.

The failure? A column the vendor added to their data format that broke our transform logic. A simple `KeyError` that could've been fixed in 5 minutes if I'd known about it on Monday.

Instead, I spent Thursday afternoon:
1. Fixing the bug (5 minutes)
2. Backfilling three days of data (45 minutes)
3. Explaining to three different teams why their dashboard was stale (90 minutes)
4. Writing a post-mortem doc (60 minutes)
5. Sitting through a "let's talk about incident response" meeting (30 minutes)

That's 3.5 hours of cleanup for a problem that would've taken 5 minutes if I'd noticed it Monday night.

My manager's question: "Why didn't we get an alert?"

My answer: "Because I didn't set one up."

His response: "That changes today."

### Here's What I Wish I'd Known: Monitor Everything That Matters

Monitoring isn't optional. It's part of the pipeline. Your pipeline isn't "done" until it can tell you when it's broken.

#### Level 1: Airflow Alerts (The Bare Minimum)

Built into Airflow:

```python
default_args = {
    'email': ['data-alerts@company.com'],
    'email_on_failure': True,  # Send email if task fails
    'email_on_retry': False,   # Don't spam on retries
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}
```

This catches:
- Tasks that fail after retries exhausted
- Tasks that timeout
- DAGs that don't start on schedule

What it doesn't catch:
- Data quality issues (wrong schema, missing rows)
- Silent failures (task succeeds but produces bad data)
- Downstream dependencies breaking

#### Level 2: Slack Notifications (Better)

```python
from airflow.operators.slack import SlackWebhookOperator

# Task that notifies on failure
def notify_failure(context):
    """Called when any task in the DAG fails"""
    task_instance = context['task_instance']
    exception = context.get('exception')

    slack_msg = f"""
    :red_circle: *Pipeline Failed*
    • DAG: {task_instance.dag_id}
    • Task: {task_instance.task_id}
    • Execution Date: {context['ds']}
    • Error: {exception}
    • Logs: {task_instance.log_url}
    """

    SlackWebhookOperator(
        task_id='slack_alert',
        http_conn_id='slack_webhook',
        message=slack_msg,
        username='Airflow Bot',
    ).execute(context=context)

# Add to DAG
with DAG(
    'critical_pipeline',
    default_args=default_args,
    on_failure_callback=notify_failure,  # Called if ANY task fails
) as dag:
    # Your tasks here
    pass
```

Now failures show up in Slack where your team actually sees them.

#### Level 3: Data Quality Checks (Essential)

The pipeline succeeded. But did it produce *correct* data?

```python
from airflow.operators.python import BranchPythonOperator
from airflow.operators.dummy import DummyOperator
from airflow.exceptions import AirflowException

def validate_sales_data(**context):
    """Check data quality after load"""
    date = context['ds']

    # Check 1: Row count isn't zero
    row_count = execute_query(f"""
        SELECT COUNT(*) FROM daily_sales_report WHERE date = '{date}'
    """)[0][0]

    if row_count == 0:
        raise AirflowException(f"No data loaded for {date}!")

    # Check 2: Revenue isn't suspiciously different from yesterday
    revenue_change = execute_query(f"""
        SELECT
            (SELECT SUM(revenue) FROM daily_sales_report WHERE date = '{date}') /
            (SELECT SUM(revenue) FROM daily_sales_report WHERE date = DATE_SUB('{date}', INTERVAL 1 DAY))
            AS revenue_ratio
    """)[0][0]

    if revenue_change > 2.0 or revenue_change < 0.5:
        # Revenue doubled or halved? Probably a data issue
        raise AirflowException(
            f"Revenue changed by {revenue_change:.1%} from yesterday. "
            f"Investigate before proceeding."
        )

    # Check 3: No null values in critical columns
    null_check = execute_query(f"""
        SELECT COUNT(*) FROM daily_sales_report
        WHERE date = '{date}' AND (product_id IS NULL OR revenue IS NULL)
    """)[0][0]

    if null_check > 0:
        raise AirflowException(f"Found {null_check} rows with null values!")

    # All checks passed
    return 'send_success_notification'

# In your DAG
with DAG('sales_pipeline', ...) as dag:
    load = PythonOperator(task_id='load_sales_data', ...)

    validate = BranchPythonOperator(
        task_id='validate_data',
        python_callable=validate_sales_data,
        provide_context=True,
    )

    success = SlackWebhookOperator(
        task_id='send_success_notification',
        message='✅ Sales pipeline completed and validated',
    )

    # Only send success notification if validation passes
    load >> validate >> success
```

This catches:
- Empty loads (source file missing)
- Schema changes (unexpected nulls)
- Data anomalies (revenue suddenly doubled)

#### Level 4: SLAs (Service Level Agreements)

Define acceptable latency:

```python
with DAG(
    'sales_pipeline',
    default_args=default_args,
    sla_miss_callback=notify_sla_miss,  # Called if SLA is breached
) as dag:

    load = PythonOperator(
        task_id='load_sales',
        python_callable=load_sales_data,
        sla=timedelta(hours=1),  # Must complete within 1 hour
    )
```

If the task takes longer than 1 hour, you get notified even if it eventually succeeds.

#### Level 5: External Monitoring (Paranoid, But Good)

Use an external service (Datadog, PagerDuty, Dead Man's Snitch) to verify pipelines ran:

```python
import requests

def heartbeat_ping(**context):
    """Ping external monitor to prove pipeline ran"""
    requests.get('https://deadmanssnitch.com/your-unique-url')

# Last task in DAG
with DAG('critical_pipeline', ...) as dag:
    # ... all your tasks ...

    heartbeat = PythonOperator(
        task_id='heartbeat',
        python_callable=heartbeat_ping,
    )

    # Final task
    final_task >> heartbeat
```

If the pipeline doesn't run at all (Airflow crashes, server reboots, etc.), the external service doesn't receive the heartbeat and alerts you.

### My Monitoring Pyramid

```
                        [External Heartbeat]          ← Most reliable
                    [SLAs & Latency Monitoring]
                [Data Quality Validation]
            [Slack Notifications]
        [Email Alerts]
    [Airflow Logs]
```

Start at the bottom, work your way up.

### Scars I've Earned: Monitoring Edition

**Scar #1: Only monitored pipeline failures, not data quality**
```python
# DON'T
# Pipeline succeeds but loads zero rows
load_data()  # Success!
# Nobody notices for three days
```
**What it cost me:** Three days of stale dashboards, product decisions made on old data, and a very awkward meeting

**Scar #2: Alert fatigue from over-monitoring**
```python
# DON'T
'email_on_retry': True,  # Spams you on every retry
'retries': 10,
# You wake up to 10 emails at 3 AM
```
**What it cost me:** Started ignoring emails, then missed a real alert in the noise, causing a different 3-day outage

**Scar #3: No escalation for critical pipelines**
```python
# DON'T
'email': ['me@company.com'],  # Only alerts me
# I'm on vacation, pipeline breaks, nobody else knows
```
**What it cost me:** CFO asked about dashboard data while I was on a beach in Tel Aviv, team couldn't fix it without me, I spent vacation debugging remotely

## Summary

Building reliable data pipelines is about more than just transformations:

- **ETL vs ELT:** Transform where it makes sense—in Python/Spark for complex logic, in SQL for structured data
- **Apache Airflow:** Orchestration beats cron every time—retries, monitoring, and dependency management built-in
- **Idempotency:** Design pipelines to run safely multiple times—use MERGE/UPSERT, staging tables, and partitions
- **Monitoring:** Hope is not a strategy—alert on failures, validate data quality, and set up heartbeats

The difference between a data engineer and a senior data engineer? The senior engineer designs for failure from day one.

## Reflection Questions

1. When was the last time you re-ran a pipeline and weren't 100% sure it wouldn't create duplicates? How would you fix that?

2. If your most critical pipeline failed right now, how long would it take for someone to notice? Who would notice first—you or your users?

3. Think about a pipeline you've built. If it silently started loading zero rows every day, what would break? When would you find out?

## Next Steps

In the next chapter, we'll dive deeper into data quality and error handling. We'll explore:
- Great Expectations for systematic data validation
- Circuit breakers for cascading failures
- Dead letter queues for unprocessable records
- Observability patterns that give you confidence your pipelines are healthy

Because pipelines that run are good. Pipelines that run *correctly* are better.
