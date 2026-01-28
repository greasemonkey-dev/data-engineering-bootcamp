# MicroSim: SQL Query Execution Plan Visualizer

## Overview

### Concept Visualized
- **Concept:** Query execution plans and database indexes (B-tree, hash, composite)
- **Learning Goal:** Students will understand how indexes dramatically affect query performance by adding/removing indexes and observing execution plan changes and timing differences
- **Difficulty:** Intermediate
- **Chapter:** Week 2 - SQL Mastery & Week 3 - Relational Databases

### The "Aha" Moment
When the student adds an index to a WHERE clause column, they see the execution plan switch from Sequential Scan (scanning all 100,000 rows) to Index Scan (scanning only matching rows), with query time dropping from 850ms to 12ms. This viscerally demonstrates why indexes matter in production databases.

## Interface Design

### Layout
- **Left Panel (60% width):** Visualization canvas showing execution plan tree and data scan animation
- **Right Panel (40% width):** Interactive controls, query editor, and performance metrics

### Controls (Right Panel)

| Control | Type | Range/Options | Default | Effect on Visualization |
|---------|------|---------------|---------|------------------------|
| Table Size | slider | 10K-1M rows | 100K | Affects scan time proportionally; visual shows more table blocks |
| Query Type | dropdown | ["Simple WHERE", "JOIN", "Aggregate with GROUP BY", "Complex Multi-Join"] | "Simple WHERE" | Changes execution plan structure and query text |
| Indexes | checkbox group | ["idx_customer_id", "idx_order_date", "idx_product_category", "idx_composite_date_customer"] | None selected | Adds/removes index; changes scan type in execution plan |
| Show Timing | toggle | on/off | on | Displays/hides millisecond timing for each node |
| Animate Scan | button | - | - | Triggers animation showing sequential vs index scan |
| Run Query | button | - | - | Executes query and displays results with timing |

**Additional UI Elements:**
- **Query Editor:** Read-only SQL text that updates based on Query Type selection
- **Metrics Panel:** Shows total rows scanned, execution time, cost estimate
- **Legend:** Color codes for scan types (Sequential=red, Index=green, Bitmap=yellow)

### Visualization Details (Left Panel)

**What is Displayed:**
- **Execution Plan Tree:** Hierarchical node structure showing operators (Scan, Join, Aggregate, Sort)
  - Nodes represented as rounded rectangles with operator name and row estimates
  - Color-coded by operation type:
    - Red: Sequential Scan (expensive)
    - Green: Index Scan (efficient)
    - Yellow: Bitmap Index Scan (medium efficiency)
    - Blue: Hash Join / Nested Loop
    - Purple: Aggregate / Sort
  - Arrow thickness represents data flow volume
  - Node size proportional to estimated cost

- **Table Visualization (below tree):**
  - Visual representation of table as grid of blocks
  - Animated scan shows row-by-row evaluation for Sequential Scan
  - Animated jump directly to matching rows for Index Scan
  - Progress bar showing percentage of table scanned

- **B-Tree Index Structure (appears when index selected):**
  - Simplified B-tree with root, internal nodes, leaf nodes
  - Highlights path from root to target data during Index Scan animation
  - Shows key comparisons at each level

**How it Updates:**
- When **indexes checkbox** changes:
  - Execution plan tree regenerates with different scan node types
  - Cost estimates recalculate (shown in nodes)
  - Timing updates (Sequential Scan: 850ms → Index Scan: 12ms for 100K rows)
  - Smooth 500ms transition animation

- When **table size slider** changes:
  - All timing metrics scale proportionally
  - Visual table grid expands/contracts
  - Cost numbers update in execution plan nodes

- When **query type dropdown** changes:
  - Entire execution plan tree rebuilds with different structure
  - Query text updates in editor
  - Appropriate indexes for that query become relevant

- When **Animate Scan button** clicked:
  - 3-second animation plays showing data access pattern
  - Sequential Scan: red highlight moves row-by-row through entire table
  - Index Scan: green highlight jumps through B-tree levels, then directly to matching rows

**Visual Feedback:**
- Hover states: Nodes show detailed cost breakdown tooltip (startup cost, total cost, rows)
- Active states: Currently executing node pulses with blue glow
- Performance indicator: Red/yellow/green badge on query timing (>500ms=red, 50-500ms=yellow, <50ms=green)

## Educational Flow

### Step 1: Default State
Student sees **Simple WHERE query** (`SELECT * FROM orders WHERE customer_id = 12345`) with execution plan showing **Sequential Scan** scanning all 100,000 rows. Timing shows 850ms. Table visualization shows red scanning animation going through every row.

This shows the baseline: without an index, database must check every single row.

### Step 2: First Interaction
**Prompt:** "Click 'Animate Scan' to see how the database finds your data. Notice it checks every single row—all 100,000 of them."

Student watches sequential scan animation (3 seconds of red highlighting moving through entire table).

**Prompt:** "Now check the box for idx_customer_id and click 'Run Query' again."

Student enables the index and clicks Run Query.

**Result:** Execution plan immediately changes to **Index Scan** with 12ms timing (70x faster!). Animation shows green path jumping directly through B-tree levels to the matching rows. Only 15 rows highlighted in table (the matches).

**What it teaches:** Indexes let the database jump directly to relevant data instead of checking everything. The B-tree visualization shows how this works—like using a book index instead of reading every page.

### Step 3: Exploration
**Prompt:** "Try different query types from the dropdown. Which queries benefit most from indexes?"

Student experiments:
- **Simple WHERE:** Sees index makes huge difference
- **JOIN:** Sees two index scans (one per table) instead of sequential scans + nested loop
- **Aggregate with GROUP BY:** Sees index scan + sort operation
- **Complex Multi-Join:** Sees execution plan with 4 tables, some using indexes, some not

**Key insight:** Indexes help filtering (WHERE) and joining, but don't eliminate all work—sorting and aggregation still happen. Student discovers that composite indexes help when query filters on multiple columns (idx_composite_date_customer helps `WHERE order_date = X AND customer_id = Y`).

### Step 4: Challenge
**Present scenario:** "Your production database has 5 million orders. A critical dashboard query takes 45 seconds and is timing out. The query filters by order_date and product_category. Which index(es) would you add?"

Student uses controls to:
1. Set table size to 1M rows (simulating large table)
2. Select "Complex Multi-Join" query type
3. Try different index combinations:
   - idx_order_date only: Improves but still 8 seconds
   - idx_product_category only: Improves but still 9 seconds
   - Both single indexes: Better, down to 3 seconds
   - idx_composite_date_category: Best! Down to 450ms

**Expected learning:**
- Composite indexes are more efficient for queries filtering on multiple columns
- Index order matters in composite indexes
- Even with indexes, large result sets take time to return
- Tradeoffs: Every index slows down INSERT/UPDATE operations (shown in warning message when too many indexes added)

## Technical Specification

### Technology
- **Library:** p5.js for canvas rendering and animations
- **Canvas:** Responsive, min 600px width × 500px height
- **Frame Rate:** 60fps for smooth animations
- **Data:** Static execution plan structures defined in JSON, timing calculations based on formulas

### Implementation Notes
- **Mobile:** Touch-friendly controls; stacked layout (visualization above controls) on screens < 768px
- **Accessibility:**
  - Keyboard navigation: Tab through controls, Space/Enter to activate
  - ARIA labels on all interactive elements
  - Text alternatives for execution plan tree (screen reader announces node traversal)
  - High contrast mode option for color-blind users
- **Performance:**
  - Limit animation complexity for tables > 500K rows (show representative subset)
  - Use canvas layering: static execution plan on bottom layer, animations on top layer
  - Debounce slider changes (300ms delay before recalculating)

### Data Requirements

**Execution Plan Templates (JSON format):**

```json
{
  "query_types": {
    "simple_where": {
      "sql": "SELECT * FROM orders WHERE customer_id = 12345",
      "without_index": {
        "node_type": "Sequential Scan",
        "table": "orders",
        "cost": "0.00..2512.00",
        "rows": 100000,
        "time_ms": 850
      },
      "with_index": {
        "node_type": "Index Scan",
        "index_name": "idx_customer_id",
        "table": "orders",
        "cost": "0.42..8.44",
        "rows": 15,
        "time_ms": 12
      }
    },
    "join": {
      "sql": "SELECT * FROM orders o JOIN customers c ON o.customer_id = c.id WHERE c.country = 'USA'",
      "execution_plan": {
        "node_type": "Hash Join",
        "cost": "5234.23..8945.67",
        "children": [
          {"node_type": "Sequential Scan", "table": "customers", "filter": "country = 'USA'"},
          {"node_type": "Sequential Scan", "table": "orders"}
        ]
      }
    }
  },
  "timing_formulas": {
    "sequential_scan_per_row": 0.0085,
    "index_scan_overhead": 8,
    "index_scan_per_match": 0.3
  }
}
```

**Sample Table Schema:**
```
orders (100K rows):
  - order_id (PK)
  - customer_id (FK)
  - order_date
  - product_category
  - amount

Possible indexes:
  - idx_customer_id (B-tree on customer_id)
  - idx_order_date (B-tree on order_date)
  - idx_product_category (Hash on product_category)
  - idx_composite_date_customer (B-tree on order_date, customer_id)
```

## Assessment Integration

After using this MicroSim, students should be able to answer:

1. **Quiz Question:** "You have a query with WHERE customer_id = 100 AND order_date > '2024-01-01'. Which index would be most efficient?"
   - A) Single index on customer_id
   - B) Single index on order_date
   - C) Composite index on (customer_id, order_date) ✓
   - D) No index needed

2. **Conceptual Question:** "Why does adding an index speed up SELECT queries but slow down INSERT operations?"
   - Expected answer: Index must be updated on every insert, requiring additional writes and B-tree rebalancing

3. **Trade-off Question:** "Your table has 10 million rows but only 5 distinct values for the 'status' column. Would an index on 'status' be effective? Why or why not?"
   - Expected answer: No—low cardinality means index scan returns huge result set, making sequential scan potentially faster due to better I/O patterns

## Extension Ideas (Optional)

- **Index Type Comparison:** Add visualization comparing B-tree vs Hash index behavior (hash for equality only, no range scans)
- **Covering Index Demo:** Show how including non-key columns in index eliminates need to access table (Index-Only Scan)
- **Join Strategy Comparison:** Visualize Hash Join vs Nested Loop Join vs Merge Join with different data sizes
- **Query Planner Simulation:** Let students see how database estimates row counts and chooses between multiple possible plans
- **Write Operation Impact:** Add "Run 1000 INSERTs" button that shows how execution time increases with each index added
- **Bitmap Index Scan:** Show intermediate strategy combining bitmap of matching rows from multiple indexes before table access
- **EXPLAIN ANALYZE Integration:** Advanced mode that shows actual vs estimated rows, highlighting where planner guessed wrong

---

**Target Learning Outcome:** Students understand that indexes are data structures (B-trees) that allow logarithmic-time lookups instead of linear scans, and can identify when indexes will/won't help query performance.
