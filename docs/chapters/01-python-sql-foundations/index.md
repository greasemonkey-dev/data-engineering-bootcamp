# Chapter 1: Python & SQL Foundations

## Learning Objectives
By the end of this chapter, you will be able to:
1. **Write** efficient Python code using list comprehensions and generators to process large datasets with minimal memory overhead
2. **Construct** complex SQL queries using window functions and CTEs to perform advanced analytical operations
3. **Analyze** the performance characteristics of different Python data structures and choose appropriate ones for data engineering tasks
4. **Evaluate** query performance and apply optimization techniques to improve SQL execution time

## Introduction

It's 3 AM. Your phone buzzes. The data pipeline you deployed yesterday is crawling to a halt, processing only 100 rows per minute instead of the expected 10,000. Your manager is awake. Your customers are complaining about stale dashboards. And you're frantically SSHing into a server, watching memory usage spike to 99%.

What went wrong? A single line of code: `data = list(read_csv('transactions.csv'))`. That innocent-looking line just loaded 50 million rows into memory. All at once.

This chapter is about the difference between code that works and code that works *at scale*. As a data engineer, you're not just writing scripts—you're building pipelines that process millions of rows, run for hours, and need to recover gracefully from failures. The difference between a junior engineer and a senior one often comes down to understanding a handful of fundamental concepts: when to use a generator instead of a list, how to write SQL that leverages indexes, and why your query that works on 1,000 rows brings the database to its knees at 1,000,000.

Let's start with Python, then move to SQL. By the end of this chapter, you'll understand not just *what* these tools do, but *when* and *why* to use them.

## Section 1: Python Generators - Processing Data Without Breaking the Bank

### Key Idea
Generators allow you to process data one item at a time, rather than loading everything into memory at once. They're the difference between a pipeline that scales and one that crashes.

### Example: Processing a Large CSV File

Imagine you're building a data pipeline that processes daily transaction logs. Each file contains millions of rows. Let's look at two approaches:

**The Memory-Hungry Approach:**

```python
def process_transactions_bad(filename):
    # Load ALL transactions into memory at once
    transactions = []
    with open(filename, 'r') as f:
        for line in f:
            transactions.append(parse_transaction(line))

    # Filter for high-value transactions
    high_value = [t for t in transactions if t['amount'] > 1000]

    # Calculate total
    total = sum(t['amount'] for t in high_value)
    return total

# With 10 million rows, this might use 5+ GB of RAM
result = process_transactions_bad('transactions.csv')
```

**The Scalable Approach Using Generators:**

```python
def read_transactions(filename):
    """Generator that yields one transaction at a time"""
    with open(filename, 'r') as f:
        for line in f:
            yield parse_transaction(line)

def process_transactions_good(filename):
    # No data loaded yet - just created a generator object
    transactions = read_transactions(filename)

    # Filter using a generator expression (not a list comprehension!)
    high_value = (t for t in transactions if t['amount'] > 1000)

    # Calculate total - data is processed one row at a time
    total = sum(t['amount'] for t in high_value)
    return total

# Memory usage stays constant regardless of file size
result = process_transactions_good('transactions.csv')
```

Notice the subtle difference: `(t for t in ...)` uses parentheses (generator expression), while `[t for t in ...]` uses brackets (list comprehension). This small syntax change has huge implications.

Let's measure the difference:

```python
import tracemalloc

# Test with the list approach
tracemalloc.start()
data_list = [i**2 for i in range(10_000_000)]
list_memory = tracemalloc.get_traced_memory()[1]  # Peak memory
tracemalloc.stop()

# Test with the generator approach
tracemalloc.start()
data_gen = (i**2 for i in range(10_000_000))
# Process one at a time
for _ in data_gen:
    pass
gen_memory = tracemalloc.get_traced_memory()[1]  # Peak memory
tracemalloc.stop()

print(f"List approach: {list_memory / 1024 / 1024:.1f} MB")
print(f"Generator approach: {gen_memory / 1024 / 1024:.1f} MB")

# Output:
# List approach: 381.5 MB
# Generator approach: 0.1 MB
```

### Why This Matters

In data engineering, you rarely work with data that fits comfortably in memory. Consider:

- **Log files**: A single day of application logs can be 100+ GB
- **Database exports**: Exporting a production table might yield billions of rows
- **Stream processing**: Data arrives continuously; you can't "load it all" because there is no "all"

Generators enable **streaming data processing**—the cornerstone of scalable data pipelines. They let you build systems that process terabytes of data on machines with gigabytes of RAM.

### Try It

Open a Python REPL and try this:

```python
# This will crash on most laptops
numbers = [i for i in range(1_000_000_000)]

# This works fine
numbers = (i for i in range(1_000_000_000))
first_ten = [next(numbers) for _ in range(10)]
print(first_ten)  # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

Now imagine you're reading from a database instead of generating numbers. Can you sketch out how you'd write a generator function that yields rows from a SQL query one at a time?

## Section 2: List Comprehensions and Generator Expressions

### Key Idea
List comprehensions and generator expressions provide concise, readable syntax for transforming and filtering data. Choose list comprehensions when you need to reuse the data; choose generator expressions when you're processing it once.

### Example: Data Transformation Pipeline

You're building an ETL pipeline that needs to:
1. Read customer records from a CSV
2. Parse and validate each record
3. Normalize phone numbers
4. Filter out invalid emails
5. Output to a database

**Readable but Inefficient:**

```python
def process_customers_verbose(filename):
    # Step 1: Read all data
    with open(filename) as f:
        lines = f.readlines()

    # Step 2: Parse
    records = []
    for line in lines:
        records.append(parse_customer(line))

    # Step 3: Normalize phones
    normalized = []
    for record in records:
        record['phone'] = normalize_phone(record['phone'])
        normalized.append(record)

    # Step 4: Filter
    valid = []
    for record in normalized:
        if is_valid_email(record['email']):
            valid.append(record)

    return valid
```

**Pythonic and Efficient:**

```python
def process_customers_pythonic(filename):
    with open(filename) as f:
        # Chained generator expressions - each step processes one item at a time
        records = (parse_customer(line) for line in f)
        normalized = (
            {**record, 'phone': normalize_phone(record['phone'])}
            for record in records
        )
        valid = (
            record for record in normalized
            if is_valid_email(record['email'])
        )

        # Only when we iterate over 'valid' does any processing happen
        for record in valid:
            insert_to_database(record)
```

Notice how the second version:
- Uses **less memory** (no intermediate lists)
- Is **more readable** (each transformation is one line)
- Is **more composable** (easy to add/remove steps)

### When to Use Each

**Use list comprehensions when:**
- You need to iterate over the data multiple times
- The dataset is small enough to fit in memory
- You need to check the length or access by index

```python
# Good use of list comprehension
user_ids = [row['user_id'] for row in users]
print(f"Processing {len(user_ids)} users")  # Need length
for user_id in user_ids:
    process(user_id)
for user_id in user_ids:  # Iterate again
    cleanup(user_id)
```

**Use generator expressions when:**
- You process each item exactly once
- The dataset is large
- You're feeding the data into another function (like `sum()`, `max()`, or database insert)

```python
# Good use of generator expression
total_revenue = sum(
    order['amount']
    for order in read_orders('orders.csv')
    if order['status'] == 'completed'
)
```

### Try It

Let's say you have a list of URLs and you want to extract the domain names. Try writing both versions:

```python
urls = [
    'https://example.com/page1',
    'https://test.org/page2',
    'https://example.com/page3'
]

# Using list comprehension
domains_list = [url.split('/')[2] for url in urls]

# Using generator expression
domains_gen = (url.split('/')[2] for url in urls)

# What happens if you print them?
print(domains_list)  # ['example.com', 'test.org', 'example.com']
print(domains_gen)   # <generator object at 0x...>

# To see generator values, convert to list or iterate
print(list(domains_gen))  # Now you see the values
```

Which one would you use if you had 10 million URLs and just wanted to count unique domains?

## Section 3: SQL Window Functions - Analytics Without Self-Joins

### Key Idea
Window functions let you perform calculations across related rows without grouping them together. They're essential for analytics queries like running totals, rankings, and period-over-period comparisons.

### Example: E-Commerce Analytics Dashboard

Your product manager asks: "For each customer, show their order history with a running total of spending and a rank for each order by amount."

**The Hard Way (Multiple Queries or Self-Joins):**

```sql
-- First, get orders with rankings (requires complex subquery)
SELECT
    o1.customer_id,
    o1.order_id,
    o1.order_date,
    o1.amount,
    COUNT(o2.order_id) as rank_by_amount
FROM orders o1
LEFT JOIN orders o2
    ON o1.customer_id = o2.customer_id
    AND o2.amount >= o1.amount
GROUP BY o1.customer_id, o1.order_id, o1.order_date, o1.amount
ORDER BY o1.customer_id, rank_by_amount;

-- Then you'd need another query for running totals...
```

This query is hard to read, hard to maintain, and slow (multiple passes over the data).

**The Window Function Way:**

```sql
SELECT
    customer_id,
    order_id,
    order_date,
    amount,
    -- Running total of spending per customer
    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
    ) as running_total,
    -- Rank orders by amount within each customer
    RANK() OVER (
        PARTITION BY customer_id
        ORDER BY amount DESC
    ) as amount_rank,
    -- Compare to previous order
    LAG(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
    ) as previous_order_amount
FROM orders
ORDER BY customer_id, order_date;
```

**Sample Output:**

```
customer_id | order_id | order_date | amount | running_total | amount_rank | previous_order_amount
------------|----------|------------|--------|---------------|-------------|----------------------
1001        | 5001     | 2024-01-15 | 50.00  | 50.00         | 3           | NULL
1001        | 5002     | 2024-02-20 | 120.00 | 170.00        | 1           | 50.00
1001        | 5003     | 2024-03-10 | 75.00  | 245.00        | 2           | 120.00
1002        | 5004     | 2024-01-18 | 200.00 | 200.00        | 1           | NULL
1002        | 5005     | 2024-02-22 | 150.00 | 350.00        | 2           | 200.00
```

### Understanding the OVER Clause

The `OVER` clause defines the "window" of rows for the calculation:

- **PARTITION BY**: Divides rows into groups (like GROUP BY, but without collapsing rows)
- **ORDER BY**: Defines the order within each partition
- **Frame specification** (optional): Defines exactly which rows to include (e.g., "last 7 days")

```sql
-- Running total for last 7 days
SUM(amount) OVER (
    PARTITION BY customer_id
    ORDER BY order_date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
)
```

### Common Window Functions

**Ranking Functions:**
```sql
-- No gaps in ranking (1, 2, 3, 4)
ROW_NUMBER() OVER (ORDER BY amount DESC)

-- Gaps when ties exist (1, 2, 2, 4)
RANK() OVER (ORDER BY amount DESC)

-- No gaps even with ties (1, 2, 2, 3)
DENSE_RANK() OVER (ORDER BY amount DESC)
```

**Offset Functions:**
```sql
-- Previous row value
LAG(amount, 1) OVER (ORDER BY order_date)

-- Next row value
LEAD(amount, 1) OVER (ORDER BY order_date)

-- First value in partition
FIRST_VALUE(amount) OVER (PARTITION BY customer_id ORDER BY order_date)
```

**Aggregate Functions as Window Functions:**
```sql
-- Average of partition
AVG(amount) OVER (PARTITION BY customer_id)

-- Count within window
COUNT(*) OVER (PARTITION BY customer_id ORDER BY order_date)
```

### Why This Matters

Window functions are crucial for:
- **Real-time dashboards**: Calculate metrics without pre-aggregation
- **Time-series analysis**: Period-over-period comparisons, moving averages
- **Ranking and percentiles**: Top N per category, quartile analysis
- **Deduplication**: Use `ROW_NUMBER()` to remove duplicates

Most importantly, they're **much faster** than self-joins or multiple queries because the database engine makes a single pass over the data.

### Try It

Given this products table:

```sql
CREATE TABLE products (
    category VARCHAR(50),
    product_name VARCHAR(100),
    price DECIMAL(10,2),
    sales_count INT
);
```

Write a query that shows:
1. Each product's rank by price within its category
2. The price difference from the most expensive product in the category
3. What percentage of total category sales this product represents

<details>
<summary>Solution</summary>

```sql
SELECT
    category,
    product_name,
    price,
    sales_count,
    RANK() OVER (PARTITION BY category ORDER BY price DESC) as price_rank,
    FIRST_VALUE(price) OVER (PARTITION BY category ORDER BY price DESC) - price as price_gap,
    ROUND(100.0 * sales_count / SUM(sales_count) OVER (PARTITION BY category), 2) as pct_category_sales
FROM products
ORDER BY category, price DESC;
```
</details>

## Section 4: Common Table Expressions (CTEs) - Writing Readable SQL

### Key Idea
CTEs let you break complex queries into logical, named steps—like writing functions in SQL. They make your queries easier to understand, debug, and maintain.

### Example: Multi-Step Business Logic

Your analytics team needs a report showing:
1. Customers who placed orders in the last 30 days
2. Their total spending in that period
3. Only customers who spent more than the average
4. Ranked by total spending

**The Nested Subquery Nightmare:**

```sql
SELECT
    customer_id,
    total_spent,
    RANK() OVER (ORDER BY total_spent DESC) as spending_rank
FROM (
    SELECT
        customer_id,
        SUM(amount) as total_spent
    FROM orders
    WHERE order_date >= CURRENT_DATE - INTERVAL '30 days'
    GROUP BY customer_id
    HAVING SUM(amount) > (
        SELECT AVG(customer_total)
        FROM (
            SELECT SUM(amount) as customer_total
            FROM orders
            WHERE order_date >= CURRENT_DATE - INTERVAL '30 days'
            GROUP BY customer_id
        ) as avg_calc
    )
) as high_spenders
ORDER BY spending_rank;
```

This query works, but it's a mess. Try debugging it. Try explaining it to a colleague. Try modifying it six months from now.

**The CTE Way:**

```sql
WITH recent_orders AS (
    -- Step 1: Get recent orders
    SELECT
        customer_id,
        amount,
        order_date
    FROM orders
    WHERE order_date >= CURRENT_DATE - INTERVAL '30 days'
),
customer_totals AS (
    -- Step 2: Calculate total spending per customer
    SELECT
        customer_id,
        SUM(amount) as total_spent,
        COUNT(*) as order_count
    FROM recent_orders
    GROUP BY customer_id
),
avg_spending AS (
    -- Step 3: Calculate average spending
    SELECT AVG(total_spent) as avg_total
    FROM customer_totals
),
high_spenders AS (
    -- Step 4: Filter for above-average spenders
    SELECT
        ct.customer_id,
        ct.total_spent,
        ct.order_count
    FROM customer_totals ct
    CROSS JOIN avg_spending avg
    WHERE ct.total_spent > avg.avg_total
)
-- Final query
SELECT
    customer_id,
    total_spent,
    order_count,
    RANK() OVER (ORDER BY total_spent DESC) as spending_rank
FROM high_spenders
ORDER BY spending_rank;
```

Each CTE is a named, reusable query. You can:
- Reference it multiple times (like a variable)
- Read the logic step by step (like functions)
- Test each step independently (copy the CTE and add a SELECT)

### Recursive CTEs: Handling Hierarchical Data

CTEs can be recursive, which is powerful for hierarchical data like org charts, category trees, or graph traversals.

**Example: Employee Hierarchy**

```sql
WITH RECURSIVE employee_hierarchy AS (
    -- Base case: start with CEO (no manager)
    SELECT
        employee_id,
        employee_name,
        manager_id,
        1 as level,
        employee_name as path
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive case: find direct reports
    SELECT
        e.employee_id,
        e.employee_name,
        e.manager_id,
        eh.level + 1,
        eh.path || ' > ' || e.employee_name
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT
    employee_id,
    REPEAT('  ', level - 1) || employee_name as org_chart,
    level,
    path
FROM employee_hierarchy
ORDER BY path;
```

**Output:**
```
employee_id | org_chart           | level | path
------------|---------------------|-------|---------------------------
1           | Alice (CEO)         | 1     | Alice (CEO)
2           |   Bob (VP)          | 2     | Alice (CEO) > Bob (VP)
4           |     Dana (Manager)  | 3     | Alice (CEO) > Bob (VP) > Dana (Manager)
7           |       Frank (IC)    | 4     | Alice (CEO) > Bob (VP) > Dana (Manager) > Frank (IC)
```

### Why This Matters

In data engineering, you often work with complex business logic:
- Multi-stage data transformations
- Incremental updates (e.g., "process only new records since last run")
- Data quality checks at each step
- Historical vs. current data comparisons

CTEs help you:
- **Debug faster**: Test each step independently
- **Optimize selectively**: Profile which CTE is slow, optimize that one
- **Collaborate better**: Colleagues can understand your queries
- **Refactor safely**: Change one CTE without breaking others

### Try It

You have these tables:

```sql
-- users: user_id, signup_date, country
-- orders: order_id, user_id, order_date, amount
-- products: product_id, product_name, category
-- order_items: order_id, product_id, quantity
```

Write a CTE-based query to find:
1. Users who signed up in the last 90 days
2. Made at least 3 orders
3. Purchased from at least 2 different product categories
4. Show their total spending and favorite category (by $ spent)

<details>
<summary>Solution Framework</summary>

```sql
WITH recent_users AS (
    -- Step 1: Get users who signed up in last 90 days
    SELECT user_id, signup_date, country
    FROM users
    WHERE signup_date >= CURRENT_DATE - INTERVAL '90 days'
),
user_orders AS (
    -- Step 2: Get their orders
    SELECT ru.user_id, o.order_id, o.amount
    FROM recent_users ru
    JOIN orders o ON ru.user_id = o.user_id
),
user_categories AS (
    -- Step 3: Find categories purchased
    SELECT
        uo.user_id,
        p.category,
        SUM(uo.amount) as category_spending
    FROM user_orders uo
    JOIN order_items oi ON uo.order_id = oi.order_id
    JOIN products p ON oi.product_id = p.product_id
    GROUP BY uo.user_id, p.category
),
qualified_users AS (
    -- Step 4: Filter for users meeting criteria
    SELECT
        user_id,
        COUNT(DISTINCT category) as categories_purchased,
        SUM(category_spending) as total_spending
    FROM user_categories
    GROUP BY user_id
    HAVING
        COUNT(DISTINCT category) >= 2
        AND (SELECT COUNT(*) FROM user_orders WHERE user_id = user_categories.user_id) >= 3
)
-- Continue from here...
SELECT * FROM qualified_users;
```
</details>

## Common Pitfalls

### 1. Using List Comprehensions When You Need Generators

**Problem:**
```python
# Loading 1 billion records into memory
data = [row for row in read_database('SELECT * FROM huge_table')]
```

**Fix:**
```python
# Processing one row at a time
data = (row for row in read_database('SELECT * FROM huge_table'))
for row in data:
    process(row)
```

**When to worry:** Any time your data source is larger than available RAM, or when you're processing data exactly once.

### 2. Forgetting PARTITION BY in Window Functions

**Problem:**
```sql
-- This calculates running total across ALL customers
SELECT
    customer_id,
    SUM(amount) OVER (ORDER BY order_date) as running_total
FROM orders;
```

**Fix:**
```sql
-- Running total per customer
SELECT
    customer_id,
    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
    ) as running_total
FROM orders;
```

### 3. Over-Nesting CTEs

**Problem:**
```sql
WITH step1 AS (...),
     step2 AS (SELECT * FROM step1),
     step3 AS (SELECT * FROM step2),
     step4 AS (SELECT * FROM step3)
     -- 10 more steps...
```

**Fix:** If you have more than 5-6 CTEs, consider breaking the query into multiple steps or creating a temporary table for intermediate results. CTEs are great for readability, but too many can hurt performance and become hard to follow.

### 4. Not Understanding Generator Exhaustion

**Problem:**
```python
data = (x for x in range(1000))
print(sum(data))  # 499500
print(sum(data))  # 0 (generator is exhausted!)
```

**Fix:** Either recreate the generator, or use a list if you need multiple iterations:
```python
# Option 1: Recreate
data = (x for x in range(1000))
print(sum(data))
data = (x for x in range(1000))  # Recreate
print(sum(data))

# Option 2: Use list if data is small
data = list(range(1000))
print(sum(data))
print(sum(data))  # Works fine
```

## Reflection Questions

1. **When would you NOT use a generator?**

   Think about these scenarios:
   - You need to access the length of the data
   - You need to iterate over the data multiple times
   - You need random access (indexing)
   - The dataset is small and you want faster access

   What's the trade-off you're making in each case?

2. **Your query with window functions is slow. What would you check first?**

   Hint: Window functions require sorting. What does sorting require? What if your PARTITION BY column isn't indexed?

3. **You have a 10-step data transformation pipeline. When should you use CTEs vs. temporary tables vs. intermediate files?**

   Consider:
   - How long does each step take?
   - Do you need to inspect intermediate results?
   - Are you running this once or repeatedly?
   - What if step 7 fails—do you want to recompute steps 1-6?

4. **A colleague writes a query with five self-joins to calculate ranks and running totals. You suggest window functions. They say, "But my way works." How do you convince them?**

   This is about more than performance—it's about maintainability, readability, and what happens when the data grows 10x next year.

## Summary

- **Generators enable streaming data processing**, allowing you to work with datasets larger than memory by processing one item at a time. Use generator expressions `(x for x in ...)` when you only need to iterate once.

- **List comprehensions are for reusable data** that fits in memory. Use `[x for x in ...]` when you need to iterate multiple times, check length, or use indexing.

- **Window functions eliminate self-joins** and enable advanced analytics like running totals, rankings, and period-over-period comparisons in a single query pass.

- **CTEs break complex queries into logical steps**, making SQL more readable, debuggable, and maintainable. Use them to structure multi-step business logic.

- **Performance matters at scale**: Code that works on 1,000 rows may fail at 1,000,000. Always consider memory usage, query execution plans, and whether your solution will scale.

## Next Steps

Now that you have Python and SQL foundations, it's time to think about collaboration and deployment. In Chapter 2, we'll explore Git workflows for team development and Docker for creating reproducible, portable environments. You'll learn how to:

- Manage merge conflicts when multiple engineers work on the same pipeline
- Use branching strategies for safe feature development
- Containerize your data pipelines for consistent execution across environments
- Orchestrate multi-container systems with Docker Compose

The patterns you learned here—streaming processing, query optimization, readable code structure—will carry forward. But now we need to ensure your carefully crafted pipelines work the same way on your laptop, your colleague's machine, and in production.
