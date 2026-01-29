# Chapter 6: Data Quality & Error Handling

## Learning Objectives
By the end of this chapter, you will be able to:
1. **Implement** data quality validation checks using Great Expectations to catch data anomalies before they reach production
2. **Design** error handling strategies including dead letter queues, circuit breakers, and retry patterns for resilient pipelines
3. **Construct** observability patterns with structured logging, metrics, and traces to debug production issues efficiently
4. **Evaluate** data quality trade-offs between strict validation (rejecting bad data) and permissive handling (accepting and flagging)

## Introduction

It's Sunday, 8:23 PM. I'm watching a movie with my family when my phone starts vibrating. Not one alert. Not two. Seventeen alerts in three minutes.

Every single one says the same thing: "Data Quality Alert: NULL values detected in critical fields."

I open my laptop. The customer_revenue table—the one feeding our entire BI dashboard, the one the board looks at every Monday morning—has 47,000 rows. All of them have NULL in the `revenue` column.

Not zero. NULL.

Which means tomorrow morning, when the board opens the dashboard, they'll see... nothing. No revenue data. Just blank cells where numbers should be.

The pipeline "succeeded." Airflow shows all green. No errors. No exceptions. The data loaded perfectly.

It was just completely, utterly useless.

Here's what happened: Our vendor changed their CSV format. They renamed `total_amount` to `transaction_amount`. My pipeline still ran—it just couldn't find the column it expected, so it filled everything with NULL. And because I only validated that *some* data loaded (row count > 0), not that it was *correct* data, the pipeline happily marked itself successful and went to sleep.

I spent the next three hours:
1. Rolling back the load
2. Fixing the column mapping
3. Re-running the pipeline
4. Writing a post-mortem explaining how 47,000 NULL values made it to production

That Sunday night taught me something I should've learned years earlier:

**Your pipeline succeeding doesn't mean your data is correct.**

This chapter is about the difference between pipelines that run and pipelines you can *trust*. It's about catching bad data before it poisons your warehouse, building systems that fail gracefully instead of silently, and designing observability so you know what's happening in production without guessing.

## Section 1: Great Expectations - The Test Suite Your Data Needs

### The Day I Shipped A Decimal Point Bug (That Cost $2.3 Million)

Let me tell you about the worst bug I've ever shipped.

We had a pricing pipeline. Every night, it calculated recommended prices for our marketplace products based on competitor pricing, demand, and inventory. The prices fed directly into the website. Millions of dollars in transactions every day.

I made a change. Simple refactoring. Extracted some logic into a function. Tested it on my laptop with sample data. Looked good. Deployed Thursday evening.

Sunday afternoon, my manager calls me. Not Slack. A phone call. Never a good sign.

"Did you touch the pricing pipeline this week?"

My stomach drops. "...yes?"

"We just sold 47 MacBook Pros for $12.99 each. They retail for $1,299."

Oh no. Oh no no no.

I'd divided by 100 instead of multiplying. A single typo. The prices weren't in dollars—they were in cents.

The pipeline ran successfully. No errors. No warnings. It just quietly generated prices that were 100x too low.

Our finance team noticed when Sunday's revenue was mysteriously $2.3 million short. By then, we'd sold thousands of products at 1% of their actual price. We couldn't claw back the orders—they'd already shipped.

That mistake cost the company $2.3 million. It cost me three months of working weekends to implement proper data quality checks on every pipeline I owned.

### Here's What I Wish I'd Known: Test Your Data Like You Test Your Code

You write unit tests for your code. Why don't you write tests for your data?

**Great Expectations** is a Python library for data validation. Think of it as pytest for your data pipelines.

Instead of hoping your data is correct, you **assert** properties about it:
- "Revenue should never be negative"
- "Email addresses should match this regex pattern"
- "Product prices should be between $1 and $10,000"
- "This column should never be NULL"

If any assertion fails, the pipeline stops. Loudly. Before bad data reaches production.

### Example: Preventing My $2.3M Decimal Bug

Here's the pricing pipeline that cost me $2.3 million:

```python
def calculate_prices(products_df):
    """Calculate recommended prices"""
    # OOPS: divided instead of multiplied
    products_df['recommended_price'] = products_df['base_price'] / 100
    return products_df

# Load and process
products = load_products()
prices = calculate_prices(products)

# Load to warehouse - no validation!
load_to_warehouse('product_prices', prices)
```

Here's what it should have looked like with Great Expectations:

```python
import great_expectations as gx

def calculate_prices(products_df):
    """Calculate recommended prices with validation"""
    # Calculate prices
    products_df['recommended_price'] = products_df['base_price'] / 100  # Still has bug!

    # Create expectations
    expectations = gx.from_pandas(products_df)

    # Assertion 1: Prices should be between $1 and $10,000
    result = expectations.expect_column_values_to_be_between(
        column='recommended_price',
        min_value=1.0,
        max_value=10000.0
    )

    if not result.success:
        raise ValueError(
            f"Price validation failed! "
            f"Found {result.result['unexpected_count']} prices outside valid range. "
            f"Examples: {result.result['partial_unexpected_list']}"
        )

    # Assertion 2: No NULL prices
    result = expectations.expect_column_values_to_not_be_null(
        column='recommended_price'
    )

    if not result.success:
        raise ValueError(f"Found {result.result['unexpected_count']} NULL prices!")

    # Assertion 3: Prices shouldn't change more than 20% from yesterday
    yesterday_prices = load_yesterday_prices()
    merged = products_df.merge(yesterday_prices, on='product_id')
    merged['price_change_pct'] = (
        (merged['recommended_price'] - merged['yesterday_price']) /
        merged['yesterday_price']
    ).abs()

    outliers = merged[merged['price_change_pct'] > 0.20]
    if len(outliers) > 100:  # Some changes are expected, but not thousands
        raise ValueError(
            f"Price validation failed! {len(outliers)} products changed >20%. "
            f"This suggests a systemic issue, not normal price fluctuations."
        )

    return products_df

# Load and process
products = load_products()
prices = calculate_prices(products)  # Would've caught the bug here!
load_to_warehouse('product_prices', prices)
```

With these checks, my decimal bug would've failed validation immediately:
- **All 50,000 products** would've had prices under $130 (way below $1-$10k range)
- **100% of prices** would've changed by 99% from yesterday

The pipeline would've failed loudly on my laptop *before* I deployed it. No $2.3 million mistake.

### Great Expectations Patterns

#### Pattern 1: Schema Validation

```python
# Expect specific columns to exist
expectations.expect_table_columns_to_match_ordered_list(
    column_list=['product_id', 'product_name', 'category', 'price', 'inventory']
)

# Expect specific data types
expectations.expect_column_values_to_be_of_type(
    column='product_id',
    type_='int64'
)
```

**Catches:** Schema changes from vendors, missing columns, type mismatches

#### Pattern 2: Value Range Validation

```python
# Prices between $0.01 and $100,000
expectations.expect_column_values_to_be_between(
    column='price',
    min_value=0.01,
    max_value=100000.0
)

# Dates within reasonable range (not in the future, not before company existed)
expectations.expect_column_values_to_be_between(
    column='order_date',
    min_value=datetime(2020, 1, 1),
    max_value=datetime.now() + timedelta(days=1)  # Allow tomorrow for timezones
)
```

**Catches:** Data entry errors, unit conversion bugs, time zone issues

#### Pattern 3: Uniqueness Constraints

```python
# Transaction IDs must be unique
expectations.expect_column_values_to_be_unique(
    column='transaction_id'
)

# Email addresses must be unique
expectations.expect_column_values_to_be_unique(
    column='email'
)
```

**Catches:** Duplicate loads, incorrect join logic, primary key violations

#### Pattern 4: NULL/Missing Value Checks

```python
# Critical fields must never be NULL
for column in ['customer_id', 'order_date', 'total_amount']:
    expectations.expect_column_values_to_not_be_null(column=column)

# Optional fields can be NULL, but track the percentage
result = expectations.expect_column_values_to_not_be_null(
    column='customer_phone',
    mostly=0.80  # At least 80% should have phone numbers
)
```

**Catches:** Missing required data, incomplete records, upstream failures

#### Pattern 5: Regex Pattern Matching

```python
# Email format
expectations.expect_column_values_to_match_regex(
    column='email',
    regex=r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
)

# US phone numbers
expectations.expect_column_values_to_match_regex(
    column='phone',
    regex=r'^\+?1?\d{10,}$'
)

# SKU format (e.g., "PROD-12345")
expectations.expect_column_values_to_match_regex(
    column='sku',
    regex=r'^PROD-\d{5}$'
)
```

**Catches:** Format violations, invalid data entry, encoding issues

#### Pattern 6: Statistical Anomaly Detection

```python
# Revenue shouldn't be more than 3 standard deviations from mean
expectations.expect_column_values_to_be_between(
    column='daily_revenue',
    min_value=historical_mean - (3 * historical_std),
    max_value=historical_mean + (3 * historical_std)
)

# Row count should be within 20% of yesterday
expectations.expect_table_row_count_to_be_between(
    min_value=int(yesterday_count * 0.80),
    max_value=int(yesterday_count * 1.20)
)
```

**Catches:** Systemic issues, data pipeline breaks, unusual business events

### My Data Quality Checklist

For every pipeline, I validate:

1. **Schema:** Expected columns exist with correct types
2. **Completeness:** Required fields are not NULL
3. **Accuracy:** Values are in valid ranges
4. **Consistency:** Relationships between fields make sense
5. **Timeliness:** Data is fresh (not stale)
6. **Uniqueness:** Keys are unique where required

### Scars I've Earned: Data Quality Edition

**Scar #1: Validated only row count, not content**
```python
# DON'T
if row_count > 0:
    print("Success!")
# All 47,000 rows had NULL revenue. Still "success."
```
**What it cost me:** Board meeting with blank dashboards, explaining how "technically the pipeline worked"

**Scar #2: Hardcoded validation thresholds that became stale**
```python
# DON'T
if row_count < 10000:  # Based on January data
    raise ValueError("Too few rows")
# In December, we had 50,000 rows. Validation blocked legitimate data.
```
**What it cost me:** Pipeline failures during peak season, manual overrides, lost trust from stakeholders

**Scar #3: Didn't validate data *before* expensive operations**
```python
# DON'T
load_to_bigquery(data)  # $500 to scan
validate_data(data)  # Validation fails, but already paid for load
```
**What it cost me:** Thousands in wasted BigQuery costs loading data we immediately had to delete

## Section 2: Error Handling - Fail Fast, Fail Loud, Fail Gracefully

### The Silent Failure That Lasted Six Weeks

Tuesday morning stand-up. Product manager mentions casually: "Has anyone noticed the customer churn dashboard hasn't changed in like... a month?"

We all look at each other. Nobody had noticed.

I check the pipeline. It's running. Every day. Airflow shows all green. No errors. No alerts.

I check the logs. There's my answer:

```
2026-01-29: Processing 50,000 records
2026-01-29: ERROR: Database connection timeout
2026-01-29: Continuing...
2026-01-29: Loaded 0 records
2026-01-29: Pipeline completed successfully
```

For six weeks, our customer churn pipeline had been failing to connect to the database, catching the exception, logging an error (that nobody read), and marking itself successful.

The churn dashboard was showing six-week-old data. Nobody noticed because churn changes slowly.

The bug? A try-catch block that was too permissive:

```python
try:
    records = fetch_from_database()
    load_to_warehouse(records)
    print("Pipeline completed successfully")
except Exception as e:
    print(f"ERROR: {e}")
    print("Continuing...")  # WHY?!
```

That `except Exception` caught everything—connection timeouts, query errors, even `KeyboardInterrupt`. And then it just... continued. Marked itself successful. Went home early.

My tech lead's feedback: "Fail fast, fail loud, fail gracefully. Pick two out of three. Never pick 'continue like nothing happened.'"

### Here's What I Wish I'd Known: Errors Are Information, Not Embarrassment

Good error handling isn't about *hiding* errors. It's about:
1. **Failing fast** when something is wrong (don't process bad data)
2. **Failing loud** so you know immediately (not six weeks later)
3. **Failing gracefully** so you can recover (not corrupting data)

#### Pattern 1: Specific Exception Handling

```python
# DON'T: Catch everything
try:
    data = fetch_from_api()
    process(data)
except Exception as e:  # Too broad!
    log.error(f"Error: {e}")
    # Now what? Continue? Retry? Die?
```

```python
# DO: Catch specific, recoverable errors
from requests.exceptions import Timeout, ConnectionError

try:
    response = requests.get(api_url, timeout=30)
    response.raise_for_status()
    data = response.json()

except Timeout:
    # Timeout is retriable - maybe the server is just slow
    log.warning("API timeout, retrying in 60 seconds")
    time.sleep(60)
    raise  # Re-raise to trigger Airflow retry

except ConnectionError as e:
    # Connection error might be transient
    log.error(f"Connection failed: {e}, will retry")
    raise

except ValueError as e:
    # JSON parse error is NOT retriable - data is corrupt
    log.error(f"Invalid JSON response: {e}")
    send_alert("API returned invalid data")
    raise  # Fail the pipeline

except Exception as e:
    # Truly unexpected errors
    log.error(f"Unexpected error: {e}", exc_info=True)
    send_alert(f"Unknown error in pipeline: {e}")
    raise
```

**Key principle:** Only catch exceptions you know how to handle. Everything else should fail the pipeline.

#### Pattern 2: Dead Letter Queues

When processing a batch of records, some might be bad. Don't let one bad record kill the entire batch.

```python
def process_transactions(transactions):
    """Process transactions with dead letter queue for bad records"""
    successful = []
    dead_letter_queue = []

    for txn in transactions:
        try:
            # Validate
            if txn['amount'] <= 0:
                raise ValueError("Amount must be positive")
            if not txn.get('customer_id'):
                raise ValueError("Missing customer_id")

            # Transform
            transformed = transform_transaction(txn)

            # Append to success
            successful.append(transformed)

        except Exception as e:
            # Don't let one bad record kill the batch
            dead_letter_queue.append({
                'original_record': txn,
                'error': str(e),
                'timestamp': datetime.now().isoformat(),
                'pipeline': 'transaction_processing'
            })

    # Load successful records
    if successful:
        load_to_warehouse('transactions', successful)
        log.info(f"Loaded {len(successful)} transactions")

    # Save failed records for investigation
    if dead_letter_queue:
        load_to_warehouse('transactions_dlq', dead_letter_queue)
        log.warning(f"Sent {len(dead_letter_queue)} records to DLQ")

        # Alert if too many failures
        failure_rate = len(dead_letter_queue) / len(transactions)
        if failure_rate > 0.10:  # More than 10% failed
            send_alert(
                f"High failure rate in transaction processing: "
                f"{failure_rate:.1%} ({len(dead_letter_queue)}/{len(transactions)})"
            )

    return {
        'successful': len(successful),
        'failed': len(dead_letter_queue)
    }
```

**Benefits:**
- Bad records don't block good records
- You can investigate failures separately
- Alerts fire when failure rate is abnormal

#### Pattern 3: Circuit Breakers

Don't hammer a failing service. If it's down, stop trying.

```python
class CircuitBreaker:
    """Stop calling a service if it's consistently failing"""
    def __init__(self, failure_threshold=5, timeout_seconds=60):
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.timeout_seconds = timeout_seconds
        self.circuit_open_until = None

    def call(self, func, *args, **kwargs):
        # Check if circuit is open
        if self.circuit_open_until:
            if datetime.now() < self.circuit_open_until:
                raise Exception(
                    f"Circuit breaker is OPEN. "
                    f"Retry after {self.circuit_open_until.isoformat()}"
                )
            else:
                # Timeout elapsed, try again
                log.info("Circuit breaker attempting to close")
                self.circuit_open_until = None
                self.failure_count = 0

        # Try the operation
        try:
            result = func(*args, **kwargs)
            self.failure_count = 0  # Reset on success
            return result

        except Exception as e:
            self.failure_count += 1
            log.warning(f"Circuit breaker failure {self.failure_count}/{self.failure_threshold}")

            if self.failure_count >= self.failure_threshold:
                # Open the circuit
                self.circuit_open_until = datetime.now() + timedelta(seconds=self.timeout_seconds)
                log.error(f"Circuit breaker OPENED until {self.circuit_open_until.isoformat()}")
                send_alert("Circuit breaker opened for API calls")

            raise

# Usage
api_circuit = CircuitBreaker(failure_threshold=3, timeout_seconds=300)

for record in records:
    try:
        result = api_circuit.call(enrich_with_api_data, record)
        processed.append(result)
    except Exception:
        # API is down, skip enrichment for now
        log.warning(f"Skipping API enrichment for record {record['id']}")
        processed.append(record)  # Process without enrichment
```

**Benefits:**
- Stops hammering a down service
- Allows service to recover
- Degrades gracefully (process without enrichment rather than failing entirely)

### Scars I've Earned: Error Handling Edition

**Scar #1: Caught and ignored exceptions**
```python
# DON'T
try:
    critical_operation()
except Exception:
    pass  # The "I hope it's fine" strategy
```
**What it cost me:** Six weeks of stale data, lost trust, and a stern talking-to about production systems

**Scar #2: Retried non-idempotent operations**
```python
# DON'T
for attempt in range(10):
    try:
        append_to_table(data)  # Not idempotent!
        break
    except:
        time.sleep(5)
# If it succeeded on attempt 3 but acknowledgment failed, attempts 4-10 create duplicates
```
**What it cost me:** Duplicate records in production, hours spent de-duplicating

**Scar #3: No dead letter queue**
```python
# DON'T
for record in million_records:
    process(record)  # One bad record kills the entire batch
```
**What it cost me:** Spent four hours finding the one malformed record in a million that was crashing the pipeline

## Summary

Building trustworthy data pipelines requires:

- **Great Expectations:** Validate data like you test code—assert properties, catch anomalies, fail before bad data reaches production
- **Specific error handling:** Catch only errors you can handle, fail fast on truly unexpected issues
- **Dead letter queues:** Don't let one bad record block a million good ones
- **Circuit breakers:** Stop hammering failing services, allow graceful degradation

The difference between junior and senior data engineers? Seniors design for bad data from day one.

## Reflection Questions

1. What would happen if your most critical data source started sending NULL values in all fields tomorrow? Would you catch it before users noticed?

2. Think about your error handling. If you catch an exception, what do you do with it? Continue? Retry? Alert? Why?

3. When was the last time you checked your "dead letter queue" or error table? Do you even have one?

## Next Steps

In the next chapter, we'll explore Apache Spark and distributed computing—because sometimes your data is too big for a single machine, no matter how carefully you handle errors.
