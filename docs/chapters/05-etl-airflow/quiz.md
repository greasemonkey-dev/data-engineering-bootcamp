# Chapter 5 Quiz: ETL/ELT & Apache Airflow

Test your understanding of data pipeline orchestration, ETL vs ELT patterns, idempotency, and monitoring.

---

#### 1. When should you prefer ELT over ETL?

<div class="upper-alpha" markdown>
1. When your source data is in JSON format and requires complex parsing logic
2. When your data warehouse is powerful enough to handle transformations efficiently and you want flexibility to re-run transforms
3. When you need to transform data using machine learning models that can't run in SQL
4. When you're loading data from multiple small sources and want to consolidate them first
</div>

??? question "Show Answer"
    The correct answer is **B**.

    ELT (Extract, Load, Transform) works best when your warehouse (like BigQuery or Snowflake) can transform data faster than you can process it externally. This approach loads raw data first, then transforms it in SQL, giving you flexibility to re-run transforms without re-extracting data.

    Option A suggests ETL (complex parsing is easier in Python). Option C suggests ETL (ML models require Python/Spark). Option D suggests ETL (consolidation before loading).

    **Concept:** ELT vs ETL pattern selection

---

#### 2. What makes an operation "idempotent"?

<div class="upper-alpha" markdown>
1. It completes in less than one second on average
2. It produces the same result no matter how many times you run it with the same input
3. It automatically retries on failure without human intervention
4. It processes data in parallel using multiple workers
</div>

??? question "Show Answer"
    The correct answer is **B**.

    An idempotent operation produces the same result whether you run it once or a hundred times. For example, `UPDATE users SET status='active' WHERE id=123` is idempotent (running it twice doesn't change the outcome), but `INSERT INTO users VALUES (123, 'active')` is not (running it twice creates duplicates or errors).

    Idempotency is crucial for data pipelines because retries, backfills, and accidental re-runs are common.

    **Concept:** Idempotency in data pipelines

---

#### 3. In Apache Airflow, what is a DAG?

<div class="upper-alpha" markdown>
1. A Python function that processes data in batches
2. A directed acyclic graph representing a workflow with tasks and dependencies
3. A database table that stores pipeline execution logs
4. A configuration file that defines retry and timeout settings
</div>

??? question "Show Answer"
    The correct answer is **B**.

    DAG stands for Directed Acyclic Graph. In Airflow, a DAG represents a workflow:
    - **Directed:** Tasks have a specific order (Task B runs after Task A)
    - **Acyclic:** No circular dependencies (Task A can't depend on Task B if B depends on A)
    - **Graph:** Visual representation of tasks and their relationships

    DAGs are defined in Python code and scheduled to run on specific intervals.

    **Concept:** Apache Airflow DAG

---

#### 4. You notice your nightly ETL pipeline has been failing for three days, but nobody was alerted. What monitoring level were you missing?

<div class="upper-alpha" markdown>
1. Airflow task failure alerts (email or Slack notifications)
2. Data quality validation checks
3. SLA (Service Level Agreement) monitoring
4. External heartbeat monitoring
</div>

??? question "Show Answer"
    The correct answer is **A**.

    If the pipeline failed and nobody was alerted, you're missing basic failure notifications. Airflow can send email or Slack alerts when tasks fail after retries are exhausted. This is the minimum monitoring level—without it, you only discover failures when users report stale data.

    Options B, C, and D are additional monitoring layers (data quality, latency, and external verification) but won't help if you're not even alerted to task failures.

    **Concept:** Monitoring and alerting

---

#### 5. Which SQL pattern ensures idempotent data loading?

<div class="upper-alpha" markdown>
1. `INSERT INTO sales VALUES (...)`
2. `INSERT INTO sales VALUES (...) ON CONFLICT (date, product_id) DO UPDATE SET ...`
3. `APPEND INTO sales VALUES (...)`
4. `COPY INTO sales FROM 's3://data.csv'`
</div>

??? question "Show Answer"
    The correct answer is **B**.

    The `INSERT ... ON CONFLICT ... DO UPDATE` pattern (PostgreSQL) or `MERGE` (BigQuery, Snowflake) ensures idempotency by:
    - Inserting new records if they don't exist
    - Updating existing records if they do exist (based on unique key)

    This prevents duplicates even if the operation runs multiple times. Option A creates duplicates. Options C and D don't guarantee idempotency without additional logic.

    **Concept:** Idempotent data loading patterns

---

#### 6. In an Airflow DAG, what does `depends_on_past=True` mean?

<div class="upper-alpha" markdown>
1. Today's pipeline run will wait until yesterday's run completes successfully
2. Tasks within the DAG must run in the order they were defined
3. The DAG will backfill all historical runs since the start date
4. Task failures will automatically trigger a rollback of previous tasks
</div>

??? question "Show Answer"
    The correct answer is **A**.

    `depends_on_past=True` means each DAG run waits for the previous run to succeed before starting. For example, if Monday's run fails, Tuesday's run won't start until Monday is fixed and succeeds.

    This setting is dangerous for most pipelines because one failure blocks all future runs. Use `depends_on_past=False` (the default) unless you have a specific reason (like each day depends on accumulating results from previous days).

    **Concept:** Airflow task dependencies

---

#### 7. Your Airflow pipeline downloads a 500GB file, transforms it in Pandas, and loads it to BigQuery. It keeps failing with "Out of Memory" errors. What's the best fix?

<div class="upper-alpha" markdown>
1. Increase the server RAM from 64GB to 1TB
2. Switch to ELT: load the raw file to BigQuery, then transform it with SQL
3. Use Pandas with larger chunk sizes to read the file in bigger batches
4. Add more retries to the Airflow task configuration
</div>

??? question "Show Answer"
    The correct answer is **B**.

    The problem is trying to load 500GB into memory. The solution is ELT:
    1. Load the raw file directly to BigQuery (no memory limits)
    2. Transform using SQL in BigQuery (handles terabytes easily)

    This is faster, cheaper, and more reliable than processing in Pandas. Option A is expensive and wasteful. Option C still hits memory limits. Option D doesn't solve the root cause.

    **Concept:** ETL vs ELT for large datasets

---

#### 8. What is the purpose of using a staging table before loading to production?

<div class="upper-alpha" markdown>
1. To speed up queries by pre-aggregating data
2. To validate data quality and structure before affecting production data
3. To reduce storage costs by compressing data before final load
4. To enable parallel processing by splitting data across multiple tables
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Staging tables allow you to:
    1. Load data into a temporary table
    2. Run validation checks (schema, row counts, nulls, anomalies)
    3. Only promote to production if validation passes

    This prevents bad data from reaching production. You can also use atomic swaps: delete production data and insert from staging in a single transaction, ensuring all-or-nothing loads.

    **Concept:** Staging table pattern

---

#### 9. Your pipeline succeeded, but loaded zero rows. None of your alerts fired. What monitoring was missing?

<div class="upper-alpha" markdown>
1. Airflow task failure alerts
2. Data quality validation checks (e.g., row count > 0)
3. Retry configuration with exponential backoff
4. External heartbeat monitoring
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Task failure alerts (A) only trigger if the task *fails*. If the task succeeds but produces wrong/empty data, you need data quality checks:

    ```python
    # After loading
    row_count = get_row_count(date)
    if row_count == 0:
        raise Exception(f"No data loaded for {date}!")
    ```

    This catches "silent failures" where the pipeline technically succeeds but produces bad results.

    **Concept:** Data quality monitoring

---

#### 10. In Airflow, what does `catchup=False` prevent?

<div class="upper-alpha" markdown>
1. Prevents the DAG from running if previous runs failed
2. Prevents Airflow from backfilling all runs between start_date and today when you first deploy the DAG
3. Prevents tasks from retrying automatically on failure
4. Prevents the DAG from running more than once per day
</div>

??? question "Show Answer"
    The correct answer is **B**.

    When you create a DAG with `start_date` in the past, Airflow wants to "catch up" by scheduling all the runs you missed. For example, if `start_date=2025-01-01` and you deploy on 2026-01-29, Airflow will schedule 394 backfill runs immediately.

    `catchup=False` disables this behavior—only future scheduled runs execute. Use this for most DAGs unless you specifically need historical backfills.

    **Concept:** Airflow catchup behavior

---

## Quiz Complete!

- **8-10 correct:** Excellent! You understand orchestration patterns, idempotency, and monitoring.
- **6-7 correct:** Good foundation. Review idempotency patterns and monitoring levels.
- **4-5 correct:** Revisit ETL vs ELT decision factors and Airflow DAG concepts.
- **0-3 correct:** Go back through the chapter, focusing on war stories and code examples.

**Next Chapter:** Data Quality & Error Handling with Great Expectations, circuit breakers, and observability patterns.
