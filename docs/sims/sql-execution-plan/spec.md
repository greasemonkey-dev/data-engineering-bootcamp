# MicroSim: SQL Query Execution Plan Visualizer

## Overview
### Concept Visualized
- **Concept:** Execution Plans (execution-plans), Database Indexes (database-indexes), Query Optimization (query-optimization)
- **Learning Goal:** Students understand how database indexes dramatically reduce the number of rows scanned during query execution by manipulating index configurations and observing execution plan changes
- **Difficulty:** Intermediate
- **Chapter:** Week 3-4 (Data Storage & Modeling)

### The "Aha" Moment
When students add a B-tree index on a WHERE clause column, they see the execution plan switch from a full table scan (examining 1,000,000 rows) to an index seek (examining only 150 rows), demonstrating how indexes transform query performance from minutes to milliseconds.

## Interface Design
### Layout
- Left Panel (60%): Visual execution plan tree with animated row flow
- Right Panel (40%): Index configuration controls and performance metrics

### Controls (Right Panel)
| Control | Type | Range/Options | Default | Effect |
|---------|------|---------------|---------|--------|
| Query Type | dropdown | "SELECT WHERE", "JOIN", "GROUP BY" | "SELECT WHERE" | Changes the SQL query being executed |
| Indexed Columns | multi-checkbox | "customer_id", "order_date", "status", "amount" | None checked | Adds/removes indexes on selected columns |
| Index Type | dropdown | "None", "B-Tree", "Hash" | "B-Tree" | Changes index algorithm |
| Table Size | slider | 1K - 10M rows | 1M rows | Scales the dataset size |
| Run Query | button | N/A | N/A | Animates query execution with current settings |

### Visualization (Left Panel)
**What is Displayed:**
- Execution plan tree (top-down flowchart: Query → Index Seek/Table Scan → Filter → Result)
- Each node shows: operation name, rows processed, estimated cost
- Animated data flow: colored particles flowing through nodes
- Color coding: Green (index used, fast), Red (table scan, slow), Yellow (intermediate operations)
- Bottom metrics bar: Total rows scanned, execution time estimate, cost score

**How it Updates:**
- When indexes are toggled, execution plan tree restructures in real-time (smooth 0.5s transition)
- "Table Scan" node transforms to "Index Seek" when appropriate index is added
- Row counts animate counting up/down to new values (1s duration)
- Particle flow speed increases dramatically when index is used
- Transitions: Smooth morph animations between plan structures

## Educational Flow
### Step 1: Default State
Student sees a simple query: `SELECT * FROM orders WHERE customer_id = 42;`
- Execution plan shows: Table Scan → Filter → Result
- Metrics: 1,000,000 rows scanned, 2.3s estimated time
- Slow red particles trickling through the plan

### Step 2: First Interaction
Prompt: "Check the checkbox to add an index on customer_id"
Result:
- Execution plan restructures to: Index Seek (customer_id) → Result
- Metrics update: 150 rows scanned, 0.003s estimated time
- Fast green particles rushing through the plan
- Students observe 766x improvement and understand index purpose

### Step 3: Exploration
Prompt: "Try different queries and see which columns benefit from indexing"
Pattern they should discover:
- Indexes help WHERE clause columns
- Indexes help JOIN columns
- Indexes on GROUP BY columns provide moderate benefit
- Hash indexes faster for equality checks, B-Tree for range queries
- Multiple indexes can compound benefits but have diminishing returns

### Step 4: Challenge
Scenario: "Your analytics query runs too slow: `SELECT SUM(amount) FROM orders WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31' GROUP BY status;` Which index(es) should you add to optimize this query?"

Expected learning:
- Students should index order_date (WHERE clause filter is primary bottleneck)
- Optionally index status (GROUP BY benefit)
- Understand that order_date needs B-Tree (range query), not Hash (equality only)
- Recognize trade-off: indexes speed reads but slow writes

## Technical Specification
- **Library:** p5.js
- **Canvas:** Responsive, min 600px width × 500px height
- **Frame Rate:** 30fps
- **Data:** Static configuration with formula-based performance calculations
  - `rows_scanned = has_index ? table_size * 0.00015 : table_size`
  - `execution_time_ms = rows_scanned * 0.0023`
  - Cost score based on rows scanned + sort operations

## Assessment Integration
After using this MicroSim, students should answer:

1. **Knowledge Check:** You have a query `SELECT * FROM users WHERE email = 'user@example.com'`. The users table has 5 million rows. Without an index, approximately how many rows would the database scan?
   - a) 1 row
   - b) 5,000 rows
   - c) 5,000,000 rows ✓
   - d) It depends on the query optimizer

2. **Application:** A query uses `WHERE created_at > '2024-01-01'`. Should you use a B-Tree index or Hash index on created_at, and why?
   - a) Hash index, because it's faster
   - b) B-Tree index, because it supports range queries ✓
   - c) No index needed for date columns
   - d) Either works equally well

3. **Trade-off Analysis:** Your table receives 10,000 INSERT operations per second but only 10 SELECT queries per hour. Should you add multiple indexes?
   - a) Yes, indexes always improve performance
   - b) No, indexes would slow down the frequent INSERT operations ✓
   - c) Only add Hash indexes
   - d) Indexes don't affect INSERT performance

## Extension Ideas
- Add composite index support showing index column order importance
- Include "EXPLAIN ANALYZE" output panel showing real PostgreSQL explain text
- Add cost-based optimizer simulation showing why optimizer chooses certain plans
- Include index fragmentation visualization over time with many writes
- Show covering index concept where query is satisfied entirely from index
- Add negative example: index on low-cardinality column (e.g., status with only 3 values)
