# Chapter 7: Apache Spark & PySpark

## Learning Objectives
By the end of this chapter, you will be able to:
1. **Explain** why distributed computing is necessary when single-machine processing fails and identify scenarios requiring Spark
2. **Implement** PySpark transformations and actions to process large datasets across multiple nodes efficiently
3. **Optimize** Spark jobs by managing partitions, avoiding shuffles, and tuning executor configurations
4. **Debug** common Spark failures including out-of-memory errors, data skew, and serialization issues

## Introduction

It's Wednesday, 2:47 PM. My Spark job has been running for 6 hours. It's processing 500GB of data. It's 94% complete. I can see the finish line.

Then my screen fills with red text:

```
WARN TaskSetManager: Lost task 2847.3 in stage 12.0
ERROR Executor: Exception in task 2847.3
java.lang.OutOfMemoryError: Java heap space
```

Then more errors. Task 2848 fails. Task 2849 fails. Tasks are failing faster than Spark can restart them.

Then the entire job dies. Six hours of processing. Gone. I'll have to start over.

And here's the thing that makes it worse: this is the **third time** I've run this job today. Same failure. Same 94% mark. Like the universe is trolling me.

I check the Spark UI. One partition is 45GB. Every other partition is 2GB. That one massive partition keeps overwhelming whatever executor gets unlucky enough to process it.

This is called "data skew." And it's the most common reason Spark jobs fail at 94% complete, wasting your entire afternoon.

Here's what nobody tells you when learning Spark: Writing code that works on your laptop is easy. Writing code that works on a 100-node cluster processing terabytes? That's the hard part.

Welcome to distributed computing. Where everything that could go wrong across 100 machines will go wrong, usually at 94% complete.

## Section 1: Why Spark? - The Day Pandas Couldn't Save Me

### When 64GB of RAM Wasn't Enough

Remember the ETL disaster in Chapter 5? The one where I tried to load 1.8TB into Pandas? Let me tell you what happened after I learned my lesson.

I switched to ELT. Load raw data to BigQuery, transform in SQL. Worked beautifully. For six months.

Then the data science team came to me with a request: "We need to run this machine learning model on all our historical transaction data. Can you prepare the training data?"

Sure, I thought. How hard can it be?

Turns out: very hard.

The requirements:
- 3 years of transaction data (2.4TB uncompressed)
- Join with customer data (500GB)
- Feature engineering (create 147 derived columns)
- Aggregate by customer (87 million unique customers)
- Output training dataset for ML model

I tried BigQuery. The SQL query worked, but:
- Cost: $1,200 per run (querying petabytes)
- No support for their custom feature engineering logic (needed Python UDFs)
- Output was 890GB (exceeds BigQuery export limits)

I tried Pandas on a huge EC2 instance (r5.24xlarge, 768GB RAM):
- Cost: $5.50/hour
- Ran for 14 hours before OOM error
- Turns out joins explode data size (2.4TB + 500GB → 8.9TB after join)

I tried processing in chunks:
- Wrote a convoluted chunking logic
- Took 37 hours to complete
- Code was unmaintainable nightmare

My tech lead: "Have you considered Spark?"

I hadn't.

"Let me show you something," she said.

### Here's What I Wish I'd Known: Some Jobs Need Distribution

**Apache Spark** is a distributed computing framework. Instead of one big machine processing all your data, Spark splits the work across many machines (a cluster).

Key concepts:

**Cluster:** A group of machines working together
- **Driver:** The boss (runs your code, coordinates workers)
- **Executors:** The workers (process data in parallel)

**RDD (Resilient Distributed Dataset):** Spark's way of representing data spread across machines

**DataFrame:** Higher-level API (like Pandas, but distributed)

**Transformations:** Operations that create new DataFrames (lazy, not executed immediately)
- `filter()`, `select()`, `groupBy()`, `join()`

**Actions:** Operations that trigger actual computation
- `count()`, `collect()`, `write()`

The magic: Spark automatically distributes your data and computations across all available machines.

### Example: The ML Training Data Pipeline in PySpark

Here's what took 37 hours in chunked Pandas, rewritten in Spark:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, sum, avg, count, datediff, when

# Initialize Spark
spark = SparkSession.builder \
    .appName("ML Training Data") \
    .config("spark.executor.memory", "16g") \
    .config("spark.executor.cores", "4") \
    .getOrCreate()

# Read data (Spark reads in parallel across cluster)
transactions = spark.read.parquet("s3://data/transactions/")
customers = spark.read.parquet("s3://data/customers/")

# Join (distributed across executors)
data = transactions.join(customers, on="customer_id", how="inner")

# Feature engineering (applied in parallel)
features = data.select(
    col("customer_id"),
    col("transaction_amount"),
    col("transaction_date"),
    col("customer_age"),
    col("customer_segment"),

    # Derived features
    when(col("transaction_amount") > 100, 1).otherwise(0).alias("is_high_value"),
    datediff("transaction_date", "customer_signup_date").alias("days_since_signup"),
    (col("transaction_amount") / col("customer_age")).alias("amount_per_age"),
)

# Aggregate by customer (shuffles data, but Spark handles it)
customer_features = features.groupBy("customer_id").agg(
    count("*").alias("transaction_count"),
    sum("transaction_amount").alias("total_spent"),
    avg("transaction_amount").alias("avg_transaction"),
    sum("is_high_value").alias("high_value_count"),
    avg("days_since_signup").alias("avg_days_since_signup")
)

# Write output (distributed write)
customer_features.write \
    .mode("overwrite") \
    .parquet("s3://output/ml_training_data/")

spark.stop()
```

**Result:**
- Runtime: 47 minutes (on a 20-node cluster)
- Cost: ~$80 (cheaper than BigQuery, way cheaper than 37 hours of my time)
- Code: Clean, readable, maintainable

That's the power of Spark. What took 37 hours on one machine took 47 minutes on 20 machines.

### When to Use Spark

**Use Spark when:**
- Data doesn't fit in memory on a single machine (>100GB)
- Complex transformations that BigQuery can't express (custom Python logic)
- Iterative processing (machine learning, graph algorithms)
- Need to process files directly (Parquet, JSON, CSV on S3/GCS)

**Don't use Spark when:**
- Data fits in Pandas comfortably (<10GB)
- Simple SQL transforms (use your warehouse instead)
- Real-time streaming with millisecond latency (Kafka Streams is better)
- You're adding complexity for no reason

## Section 2: PySpark Basics - The 80/20 You Need

### The Job That Took 8 Hours (Then I Learned About Lazy Evaluation)

My first Spark job was adorable in hindsight.

I wanted to count transactions by country. Simple, right?

```python
# Read data
transactions = spark.read.parquet("s3://data/transactions/")

# Filter for 2025
filtered = transactions.filter(col("year") == 2025)
print(f"Loaded transactions: {filtered.count()}")

# Group by country
grouped = filtered.groupBy("country")
print(f"Grouped by country: {grouped.count()}")

# Count
result = grouped.agg(count("*").alias("txn_count"))
print(f"Final result: {result.count()}")

# Save
result.write.parquet("s3://output/country_counts/")
```

This job took **8 hours**.

My senior engineer looked at my code and laughed. "You counted the same data three times."

"What? No, I..." Then I understood.

Every `.count()` triggers a full scan of the data. I was:
1. Scanning 2TB to count filtered transactions
2. Scanning 2TB again to count grouped data
3. Scanning 2TB again to count final result
4. Scanning 2TB one more time to write the output

**That's 8TB scanned for a job that should've scanned 2TB once.**

### Here's What I Wish I'd Known: Transformations Are Lazy, Actions Are Expensive

Spark operations come in two types:

**Transformations (lazy):** Build an execution plan, but don't execute
- `filter()`, `select()`, `groupBy()`, `join()`, `withColumn()`
- Cost: Free (just planning)

**Actions (eager):** Actually execute the plan
- `count()`, `collect()`, `show()`, `write()`
- Cost: Scans your data

My fixed code:

```python
# Build the plan (no execution yet)
result = spark.read.parquet("s3://data/transactions/") \
    .filter(col("year") == 2025) \
    .groupBy("country") \
    .agg(count("*").alias("txn_count"))

# Execute ONCE
result.write.parquet("s3://output/country_counts/")

# If you need to see the results:
result.show(10)  # Second execution (unavoidable if you want preview)
```

**Runtime: 22 minutes** (instead of 8 hours)

### Essential PySpark Patterns

#### Pattern 1: Filter Early, Filter Often

```python
# BAD: Read everything, then filter
df = spark.read.parquet("s3://data/transactions/")  # 2TB
filtered = df.filter(col("year") == 2025)  # Still processed 2TB

# GOOD: Filter during read (partition pruning)
df = spark.read.parquet("s3://data/transactions/") \
    .filter(col("year") == 2025)  # Spark only reads 2025 partitions
```

If your data is partitioned by year, Spark skips reading other years entirely.

#### Pattern 2: Select Only Columns You Need

```python
# BAD: Read all 50 columns, use 3
df = spark.read.parquet("s3://data/transactions/")
result = df.select("customer_id", "amount", "date")

# GOOD: Select early
df = spark.read.parquet("s3://data/transactions/") \
    .select("customer_id", "amount", "date")
# Spark only reads these 3 columns (columnar format like Parquet)
```

#### Pattern 3: Avoid `collect()` on Large Data

```python
# BAD: Brings all data to driver
df = spark.read.parquet("s3://huge_file/")
data = df.collect()  # OOM on driver! All data to one machine
for row in data:
    process(row)

# GOOD: Process distributed
df = spark.read.parquet("s3://huge_file/")
df.foreach(lambda row: process(row))  # Runs on executors, in parallel
```

`collect()` brings all data to the driver. If your data is 500GB, your driver needs 500GB+ RAM. Use it only for small result sets (<1GB).

#### Pattern 4: Repartition for Parallelism

```python
# BAD: File has 1 partition, Spark uses 1 executor
df = spark.read.csv("s3://data/giant_file.csv")  # 1 file = 1 partition
df.write.parquet("output/")  # Slow, only 1 executor working

# GOOD: Repartition to use all executors
df = spark.read.csv("s3://data/giant_file.csv") \
    .repartition(200)  # Split into 200 partitions
df.write.parquet("output/")  # Fast, 200 executors working in parallel
```

### Scars I've Earned: PySpark Edition

**Scar #1: Called `.count()` in every step for debugging**
```python
# DON'T
df = spark.read.parquet("data/")
print(f"Rows: {df.count()}")  # Scan 1
filtered = df.filter(...)
print(f"After filter: {filtered.count()}")  # Scan 2
result = filtered.groupBy(...)
print(f"Result: {result.count()}")  # Scan 3
```
**What it cost me:** 8 hours for a job that should've taken 20 minutes

**Scar #2: Used `.collect()` on 200GB of data**
```python
# DON'T
results = huge_df.collect()  # Driver crashes
```
**What it cost me:** Driver node OOM, lost 3 hours of processing

**Scar #3: Didn't partition before writing**
```python
# DON'T
df.write.parquet("output/")  # Creates 10,000 tiny files
```
**What it cost me:** "Small files problem" - reading 10,000 tiny files is slower than reading 200 medium files

## Section 3: Spark Performance - The 94% Problem

### Data Skew: The Silent Job Killer

Remember that job that kept dying at 94%? Let me show you what was happening.

I was processing clickstream data. Millions of users, billions of events. Needed to aggregate by user_id.

```python
# Group by user
user_stats = events.groupBy("user_id").agg(
    count("*").alias("event_count"),
    sum("duration").alias("total_duration")
)
```

Simple query. Ran for 6 hours. Failed at 94%.

The Spark UI showed the problem:

```
Executor 1: Processing 2GB (Task 1/200)
Executor 2: Processing 2GB (Task 2/200)
Executor 3: Processing 2GB (Task 3/200)
...
Executor 47: Processing 45GB (Task 189/200) <- STILL RUNNING
```

One executor was processing 22x more data than others. That executor ran out of memory.

Why? **Data skew.** One user_id had 10 million events (a bot). Every other user had ~500 events.

When Spark groups by `user_id`, it sends all records for each user to the same partition. That bot's 10 million events went to one partition, overwhelming it.

### Here's What I Wish I'd Known: Skew Breaks Everything

**Data skew:** When data isn't evenly distributed across partitions.

Causes:
- One customer has 10x more orders than others
- One product has 100x more reviews
- NULL values all hash to the same partition
- Bots generating millions of events

Symptoms:
- Jobs fail at 90%+
- One task takes 10x longer than others
- Out of memory on specific executors

**Solution 1: Salt the Key**

Add randomness to break up hot keys:

```python
from pyspark.sql.functions import rand, concat, lit

# Add random salt to spread data
salted = events.withColumn("salted_user",
    concat(col("user_id"), lit("_"), (rand() * 10).cast("int"))
)

# Group by salted key
user_stats = salted.groupBy("salted_user").agg(...)

# Remove salt and re-aggregate
final = user_stats.withColumn("user_id",
    split(col("salted_user"), "_")[0]
).groupBy("user_id").agg(
    sum("event_count").alias("event_count"),
    sum("total_duration").alias("total_duration")
)
```

Now the bot's 10 million events are split across 10 partitions instead of overwhelming one.

**Solution 2: Broadcast Joins for Small Tables**

```python
# BAD: Regular join shuffles both sides
big_df.join(small_df, on="key")  # Shuffles 500GB

# GOOD: Broadcast small table to all executors
from pyspark.sql.functions import broadcast

big_df.join(broadcast(small_df), on="key")  # No shuffle! Small table copied to all nodes
```

Use broadcast joins when one side is <100MB.

**Solution 3: Increase Partition Count**

```python
# More partitions = smaller partitions = less likely to OOM
spark.conf.set("spark.sql.shuffle.partitions", "1000")  # Default is 200
```

### My Spark Performance Checklist

Before running any Spark job:

1. **Filter early:** Reduce data size ASAP
2. **Select only needed columns:** Don't read what you won't use
3. **Check for skew:** Look at partition sizes in Spark UI
4. **Avoid unnecessary shuffles:** Use broadcast joins when possible
5. **Partition appropriately:** Not too few (underutilized cluster), not too many (overhead)
6. **Cache strategically:** `.cache()` DataFrames you'll reuse

## Summary

Apache Spark distributes computation across many machines to handle data that doesn't fit on one:

- **Lazy evaluation:** Transformations build a plan; actions execute it
- **Data skew:** Uneven data distribution kills jobs at 94% - use salting
- **Broadcast joins:** For small tables, avoid shuffling by copying to all nodes
- **Partitioning:** Balance parallelism (more partitions) vs overhead (too many partitions)

The difference between working Spark code and fast Spark code? Understanding what happens when Spark distributes your data.

## Reflection Questions

1. Your Spark job processes 1TB but takes 8 hours. How would you diagnose whether it's data skew, too few partitions, or unnecessary shuffles?

2. When should you use `.collect()` vs `.write()`? What's the difference in where data ends up?

3. Think about your largest dataset. Is it big enough to need Spark? Or could Pandas + chunking work?

## Next Steps

In the next chapter, we'll explore Kafka and streaming data—because batch processing is great, but sometimes you need results in seconds, not hours.
