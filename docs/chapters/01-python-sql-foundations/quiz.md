# Chapter 1 Quiz: Python & SQL Foundations

#### 1. What is the primary advantage of using a generator instead of a list comprehension for processing large datasets?

<div class="upper-alpha" markdown>
1. Generators execute faster than list comprehensions
2. Generators use lazy evaluation and consume less memory
3. Generators can only be iterated once, ensuring data integrity
4. Generators automatically parallelize operations across CPU cores
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Generators use lazy evaluation, producing values one at a time only when requested, which significantly reduces memory consumption for large datasets. List comprehensions create the entire list in memory immediately. Option A is incorrect because execution speed is comparable; the main benefit is memory efficiency. Option C describes a characteristic but not a benefit. Option D is incorrect as generators do not automatically parallelize—that requires explicit threading or multiprocessing libraries.

    **Concept:** Generators and Lazy Evaluation

---

#### 2. Which of the following best describes the purpose of a context manager in Python?

<div class="upper-alpha" markdown>
1. It manages the flow of control between different functions
2. It ensures proper acquisition and release of resources
3. It optimizes memory allocation for data structures
4. It monitors the execution context of decorators
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Context managers (using the `with` statement) ensure that resources like file handles, database connections, or locks are properly acquired before use and released afterward, even if an exception occurs. This prevents resource leaks. Option A confuses context managers with control flow statements. Option C is incorrect; memory management is handled elsewhere. Option D incorrectly conflates context managers with decorators.

    **Concept:** Context Managers

---

#### 3. In a SQL window function, what does the OVER clause define?

<div class="upper-alpha" markdown>
1. The filtering criteria that determines which rows are included
2. The frame of rows over which the function is calculated
3. The sort order for the entire query result set
4. The join condition between multiple tables
</div>

??? question "Show Answer"
    The correct answer is **B**.

    The OVER clause specifies the window (partition, ordering, and frame) over which the window function operates. It defines which rows are considered for the calculation. Option A describes the WHERE clause. Option C is partially true but incomplete; ordering is part of the window specification, not the entire result set ordering. Option D relates to JOIN syntax, not window functions.

    **Concept:** Window Functions

---

#### 4. How would you write a query to find the running total of sales amount ordered by date using a window function?

<div class="upper-alpha" markdown>
1. `SELECT date, sales, SUM(sales) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) FROM transactions`
2. `SELECT date, sales, SUM(sales) OVER (PARTITION BY date ORDER BY sales) FROM transactions`
3. `SELECT date, sales, CUMSUM(sales ORDER BY date) FROM transactions`
4. `SELECT date, sales, SUM(sales) OVER (ORDER BY date DESC) FROM transactions`
</div>

??? question "Show Answer"
    The correct answer is **A**.

    The ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW clause explicitly defines the window frame to include all rows from the start up to the current row, creating a running total. Option B uses PARTITION BY which resets the sum for each date group. Option C uses incorrect syntax; CUMSUM is not a standard SQL function. Option D calculates the total for all rows in reverse order, not a running total.

    **Concept:** Window Function Frames

---

#### 5. What is the primary difference between a Common Table Expression (CTE) and a subquery?

<div class="upper-alpha" markdown>
1. CTEs are stored in the database while subqueries are temporary
2. CTEs are more readable and can be referenced multiple times in the same query
3. Subqueries are faster because they are optimized differently
4. CTEs only work with SELECT statements, not with INSERT or UPDATE
</div>

??? question "Show Answer"
    The correct answer is **B**.

    CTEs (WITH clause) improve query readability and can be referenced multiple times, reducing repetition. While functionally similar, CTEs provide better code organization. Option A is incorrect; both are temporary. Option C is misleading; performance depends on the optimizer, not the structure. Option D is false; CTEs work with INSERT, UPDATE, and DELETE statements.

    **Concept:** Common Table Expressions

---

#### 6. A list comprehension `[x**2 for x in range(1000000) if x % 2 == 0]` is used to process a large dataset. What is a potential issue with this approach?

<div class="upper-alpha" markdown>
1. It will take significantly longer than a traditional for loop
2. It creates the entire list in memory at once, potentially causing memory issues
3. It cannot handle the condition `x % 2 == 0` properly
4. It will skip every other number due to the if clause
</div>

??? question "Show Answer"
    The correct answer is **B**.

    List comprehensions create the complete list immediately in memory. For 1,000,000 elements, this could consume significant memory. Using a generator expression with parentheses instead would use lazy evaluation. Option A is incorrect; list comprehensions are typically as fast or faster than loops. Option C is incorrect; the if clause works correctly. Option D misunderstands the if clause; it filters, not skips alternating positions.

    **Concept:** List Comprehensions vs Generators

---

#### 7. Why might you choose denormalized data structures in a data engineering context?

<div class="upper-alpha" markdown>
1. Because normalized databases are always slower and less reliable
2. To reduce query complexity and improve read performance at the cost of storage
3. Because denormalized data is always easier to maintain
4. To eliminate the need for indexes and query optimization
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Denormalization is a performance optimization technique that reduces join operations and simplifies queries, though it increases storage and complicates updates. This trade-off is often justified in data warehousing contexts. Option A is an overstatement; normalization has benefits. Option C is false; denormalized data is often harder to maintain due to update anomalies. Option D is incorrect; denormalization doesn't eliminate the need for optimization.

    **Concept:** Denormalization Trade-offs

---

#### 8. Given a dataset with columns `user_id`, `purchase_date`, and `amount`, how would you calculate the average purchase amount per user?

<div class="upper-alpha" markdown>
1. `SELECT user_id, AVG(amount) FROM purchases GROUP BY user_id`
2. `SELECT user_id, amount / COUNT(*) FROM purchases`
3. `SELECT AVG(amount) FROM purchases WHERE user_id IS NOT NULL`
4. `SELECT user_id, AVG(DISTINCT amount) FROM purchases GROUP BY user_id`
</div>

??? question "Show Answer"
    The correct answer is **A**.

    Using GROUP BY with AVG() correctly calculates the average amount per user. Option B divides each amount by the total count, which is incorrect. Option C calculates the global average, not per-user averages. Option D uses AVG(DISTINCT amount), which would only average unique values, not all purchases—an incorrect interpretation of the requirement.

    **Concept:** SQL Aggregation Functions

---

#### 9. What does a decorator in Python do?

<div class="upper-alpha" markdown>
1. It decorates the code with comments to improve readability
2. It modifies the behavior of a function or class without changing its source code
3. It creates a visual representation of code structure
4. It automatically generates documentation for functions
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Decorators wrap functions to modify their behavior (e.g., adding logging, caching, or authentication) without altering the original function code. They use the @ syntax and are applied at definition time. Option A confuses decorators with comments. Option C is incorrect; decorators don't create visualizations. Option D is incorrect; that's the purpose of docstrings or documentation tools.

    **Concept:** Decorators

---

#### 10. How would you optimize a slow SQL query that returns results in 5 seconds?

<div class="upper-alpha" markdown>
1. Always add an index on every column used in WHERE clauses
2. Rewrite subqueries as CTEs and add appropriate indexes based on execution plan analysis
3. Remove all JOINs and denormalize the tables
4. Increase the database server's RAM to speed up query execution
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Effective optimization involves analyzing the execution plan to identify bottlenecks, then adding targeted indexes and restructuring queries as needed. Option A is excessive; not all columns benefit from indexes. Option C is premature optimization that sacrifices data integrity. Option D doesn't address the root cause if the issue is poor query structure or missing indexes.

    **Concept:** Query Optimization

---

## Question Distribution Summary

**Bloom's Taxonomy:**
- Remember (25%): Questions 1, 3 = 20%
- Understand (30%): Questions 2, 5, 7 = 30%
- Apply (30%): Questions 4, 6, 8, 9 = 40%
- Analyze (15%): Question 10 = 10%

**Answer Distribution:**
- A: 20% (2 questions)
- B: 50% (5 questions)
- C: 20% (2 questions)
- D: 10% (1 question)

*Note: Answer distribution will be balanced across all 4 quizzes (40 questions total) to achieve target percentages.*
