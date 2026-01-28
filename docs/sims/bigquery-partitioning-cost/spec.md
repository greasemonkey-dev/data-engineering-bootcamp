# MicroSim: BigQuery Partitioning Cost Calculator

## Overview

### Concept Visualized
- **Concept:** BigQuery table partitioning strategies and pricing model (data scanned = cost)
- **Learning Goal:** Students will understand how partitioning dramatically reduces query costs in BigQuery by comparing data scanned across different partition strategies for the same analytical queries
- **Difficulty:** Intermediate
- **Chapter:** Week 3-4 - Data Storage & Modeling (BigQuery Introduction)

### The "Aha" Moment
When the student runs the query "Get sales for last 7 days" on an unpartitioned 5TB table, they see BigQuery scans all 5TB costing $25. Then they apply daily partitioning and run the same query—BigQuery scans only 35GB (7 days of data) costing $0.175. That's a **143x cost reduction** for the exact same result. This demonstrates: **in cloud data warehouses, schema design directly impacts your bill**.

## Interface Design

### Layout
- **Left Panel (55% width):** Table visualization showing data blocks colored by partition/date
- **Right Panel (45% width):** Controls, query builder, and cost metrics
- **Bottom Status Bar (full width):** Real-time cost calculator and data scanned indicator

### Controls (Right Panel)

| Control | Type | Range/Options | Default | Effect on Visualization |
|---------|------|---------------|---------|------------------------|
| Table Size | slider | 100GB-10TB | 5TB | Scales table visualization and costs proportionally |
| Partition Strategy | dropdown | ["None (Unpartitioned)", "Daily", "Monthly", "Yearly", "Custom Range"] | "None" | Changes how data blocks are organized/colored |
| Partition Column | dropdown | ["date", "timestamp", "ingestion_time"] | "date" | Determines partition key (only if partitioned) |
| Clustering | checkbox | - | off | Adds clustering within partitions (shows additional optimization) |
| Query Preset | dropdown | ["Last 7 Days", "Last Month", "Specific Date", "Year-to-Date", "Full Table Scan"] | "Last 7 Days" | Loads common query patterns |
| Date Range | date picker | 2020-2024 | last 7 days | Filters data range for query |
| Run Query | button | - | - | Executes query, animates data scan, shows cost |
| Compare Strategies | button | - | - | Runs same query across all partition strategies side-by-side |

**Additional UI Elements:**
- **Query Editor:** Editable SQL text showing current query with WHERE clause
- **Cost Breakdown Panel:** Itemized costs (data scanned, on-demand vs flat-rate pricing)
- **Best Practice Tips:** Context-sensitive recommendations based on current query
- **Historical Queries Log:** Shows past queries with costs (builds over session)

### Visualization Details (Left Panel)

**What is Displayed:**

**Table Representation:**
- **Visual Metaphor:** Table shown as grid of "data blocks" (rectangles)
  - Each block represents a chunk of data (e.g., 1GB)
  - 5TB table = 5,000 blocks
  - Arranged in timeline from left (older) to right (newer)

**Color Coding by Partition Strategy:**

**Unpartitioned (None):**
- All blocks same gray color (no organization)
- Data appears as monolithic mass
- Label: "5TB unpartitioned data"

**Daily Partitioning:**
- Blocks colored by date (gradient from blue to red across time)
- Vertical dividers separate days
- Labels: "2024-01-15", "2024-01-16", "2024-01-17"...
- Each day's data visually grouped

**Monthly Partitioning:**
- Blocks colored by month (12 distinct colors)
- Thicker vertical dividers separate months
- Labels: "Jan 2024", "Feb 2024"...

**Yearly Partitioning:**
- Blocks colored by year (4-5 distinct colors for multi-year data)
- Very thick dividers separate years
- Labels: "2020", "2021", "2022"...

**Query Execution Visualization:**
- When query runs, animation highlights which blocks are scanned
- **Unpartitioned:** All blocks highlight yellow (full table scan)
- **Partitioned:** Only relevant partition blocks highlight green (pruned scan)
- Non-scanned blocks fade to 20% opacity (showing pruning effectiveness)
- Progress bar shows percentage of table scanned

**Additional Visual Elements:**
- **Partition Pruning Indicator:** Scissors icon appears when partitions are eliminated
- **Data Scanned Bar:** Horizontal bar below table showing proportion: "35GB / 5TB (0.7%)"
- **Cost Meter:** Thermometer visualization filling from green ($0) to red ($50+)

**How it Updates:**

**When "Partition Strategy" dropdown changes:**
1. Table blocks animate rearranging (500ms)
2. Colors transition to new partition scheme (800ms)
3. Partition dividers appear/disappear
4. Labels update to show partition boundaries
5. Info panel explains chosen strategy: "Daily partitioning creates 365 partitions per year. Queries filtering by date scan only relevant days."

**When "Run Query" button clicked:**
1. Query text highlights WHERE clause (300ms)
2. System calculates matching partitions (shown in right panel):
   - "Date filter: 2024-01-15 to 2024-01-21"
   - "Partitions matched: 7"
   - "Partitions scanned: 7 / 1,825 (0.4%)"
3. Table visualization animation (2 seconds):
   - Matching partition blocks pulse yellow
   - Non-matching blocks fade to 20% opacity
   - Data flow animation: selected blocks → query processor icon
   - Progress counter increments: "Scanning 35GB..."
4. Results panel appears:
   - "Data scanned: 35GB"
   - "Cost: $0.175 (on-demand pricing)"
   - "Execution time: 3.2s"
   - Comparison badge: "143x cheaper than unpartitioned!"

**When "Compare Strategies" button clicked:**
- Screen splits into 4 quadrants showing same query with:
  1. No partitioning: Scans 5TB, costs $25
  2. Daily partitioning: Scans 35GB, costs $0.175
  3. Monthly partitioning: Scans 155GB (entire January), costs $0.775
  4. Yearly partitioning: Scans 1.8TB (entire 2024), costs $9
- Bar chart appears comparing costs
- Winner highlighted in green with crown icon

**Visual Feedback:**
- Hover over data blocks: Tooltip shows partition details ("Partition: 2024-01-15, Size: 5GB, Rows: 2.1M")
- Hover over query: Highlights partition column in WHERE clause
- Cost meter changes color: <$1 green, $1-$10 yellow, >$10 red
- Optimization suggestions appear: "💡 Daily partitioning recommended for queries filtering by date"

## Educational Flow

### Step 1: Default State
Student sees **unpartitioned 5TB sales table** with sample schema:
```sql
CREATE TABLE sales (
  order_id INT64,
  customer_id INT64,
  product_id INT64,
  order_date DATE,
  order_timestamp TIMESTAMP,
  amount FLOAT64
)
```

Query preset: **"Last 7 Days"**
```sql
SELECT *
FROM sales
WHERE order_date >= '2024-01-15' AND order_date <= '2024-01-21'
```

Info panel explains: "This table contains 3 years of sales data (5TB). Let's see how much it costs to query one week of data."

This shows the baseline: no optimization.

### Step 2: First Interaction - Feel the Cost
**Prompt:** "Click 'Run Query' to see how BigQuery processes this query."

Student clicks Run Query.

**Result:**
- Animation shows ALL data blocks highlighting yellow (full table scan)
- Progress bar: "Scanning 5,000GB... 10%... 50%... 100%"
- Results panel:
  - **Data scanned: 5TB**
  - **Cost: $25.00** (at $5 per TB)
  - **Execution time: 45 seconds**
  - ⚠️ "Warning: This query scanned 5TB to return 7 days of data!"

**Info panel explains:** "Without partitioning, BigQuery must scan every row to find matching dates. Even though you only want 7 days, it reads 3 years. You pay for all 5TB scanned."

**Prompt:** "That's $25 for one query! If your dashboard runs this every 5 minutes (288 times/day), that's $7,200/day = $216,000/month. Let's fix this with partitioning."

**What it teaches:** Cloud data warehouse costs are based on data scanned, not returned. Poor schema design = expensive queries.

### Step 3: Apply Partitioning
**Prompt:** "Change 'Partition Strategy' to 'Daily' and run the same query again."

Student selects "Daily" from dropdown.

**Visual change:**
- Table blocks rearrange into daily partitions (animation)
- Color gradient appears (blue = older, red = newer)
- Partition labels appear: "2021-01-01", "2021-01-02"... "2024-01-21"
- Partition count badge: "1,095 partitions"

**Info panel explains:** "Daily partitioning creates one partition per day. BigQuery stores partition metadata and uses it to eliminate partitions during queries. Your WHERE clause `order_date >= '2024-01-15'` allows BigQuery to skip 1,088 partitions!"

**Prompt:** "Now click 'Run Query' again."

Student clicks Run Query.

**Result:**
- Animation shows ONLY 7 daily partition blocks highlighting green (2024-01-15 through 2024-01-21)
- Other 1,088 partitions fade to 20% opacity with "✂️ Pruned" label
- Progress bar: "Scanning 35GB... (partitions pruned!)"
- Results panel:
  - **Data scanned: 35GB** (7 days × 5GB per day)
  - **Cost: $0.175**
  - **Execution time: 3.2 seconds**
  - ✅ "Optimized: Partition pruning saved $24.82 (99.3% reduction)!"
  - Badge: "143x cheaper than unpartitioned"

**Cost comparison tooltip:** "$25.00 → $0.175 saved $24.82 per query. At 288 queries/day: saved $7,148/day = $214,440/month!"

**What it teaches:** Partitioning uses metadata to skip irrelevant data, dramatically reducing costs. Proper partitioning can save 99%+ on costs for filtered queries.

### Step 4: Exploration - Partition Granularity Matters
**Prompt:** "Try different partition strategies (Monthly, Yearly) and compare costs. Which is best?"

**Student experiments:**

**Monthly Partitioning:**
- Query for last 7 days (Jan 15-21) must scan entire January partition
- Data scanned: 155GB (31 days)
- Cost: $0.775
- Lesson: Too coarse—scans 4.4x more data than necessary

**Yearly Partitioning:**
- Query for last 7 days must scan entire 2024 partition
- Data scanned: 1.8TB (365 days so far)
- Cost: $9.00
- Lesson: Way too coarse—scans 51x more data than necessary

**Daily Partitioning (revisited):**
- Scans exactly 7 days needed
- Cost: $0.175
- Lesson: Optimal granularity for this query pattern

**Key insight:** Partition granularity should match query filter granularity. If queries typically filter by day/week, use daily partitioning. If queries filter by month, monthly partitioning is appropriate.

**Info panel explains:** "Daily partitioning has 365x more partitions than yearly, but each partition is 365x smaller. BigQuery partition metadata is extremely lightweight (free to store), so finer partitioning is almost always better for reducing costs."

### Step 5: Compare Different Query Patterns
**Prompt:** "Switch query preset to 'Full Table Scan' (e.g., calculating lifetime totals). Does partitioning still help?"

Student selects "Full Table Scan" preset:
```sql
SELECT SUM(amount) AS total_revenue
FROM sales
-- No WHERE clause
```

Student runs query with daily partitioning.

**Result:**
- All 1,095 partitions highlight (no pruning possible)
- Data scanned: 5TB (entire table)
- Cost: $25.00
- Info: "Partitioning doesn't help queries without partition column filters"

**Key insight:** Partitioning only helps queries that filter on partition column. For full table scans, partition strategy doesn't matter. Must design partitions based on common query patterns.

### Step 6: Challenge
**Present scenario:** "You're designing a BigQuery table for IoT sensor data:
- 1 billion events per day
- 500GB uncompressed data per day
- Table will contain 2 years of data = 365TB total
- Common queries:
  - Dashboard: Show last 24 hours of data (refreshes every 5 minutes)
  - Weekly reports: Analyze last 7 days by sensor type
  - Monthly aggregations: Calculate monthly totals per device
  - Ad-hoc analyses: Engineers query arbitrary date ranges

Your company has $5,000/month BigQuery budget. Design the optimal partitioning strategy."

**Interactive Challenge:**
Student must:
1. Choose partition strategy
2. Choose partition column (ingestion_time vs event_timestamp)
3. Decide whether to add clustering
4. Test with sample queries

**Simulator shows:**
- Dashboard query (last 24h) runs 8,640 times/month:
  - Unpartitioned: 365TB × 8,640 = **$15.8M/month** 🔴
  - Daily partitioned: 0.5TB × 8,640 = **$21,600/month** 🟡
  - Hourly partitioned: 21GB × 8,640 = **$907/month** ✅

**Expected solution:**
- **Partition:** Daily or hourly on ingestion_time (ingestion_time is better than event_timestamp due to late-arriving data)
- **Clustering:** Add clustering on sensor_id and sensor_type (helps "by sensor type" queries)
- **Result:** Total cost ~$2,800/month (well under budget)

**Advanced considerations shown:**
- "Hourly partitioning creates 17,520 partitions/year. BigQuery has 4,000 partition limit per table. Use partition expiration or switch to daily + clustering."
- "Consider BigQuery flat-rate pricing ($20,000/month) if your workload exceeds $6,000/month on-demand."

**Expected learning:** Students understand how to calculate costs, choose partition granularity based on query patterns, and recognize partition limits and alternative pricing models.

## Technical Specification

### Technology
- **Library:** p5.js for table block visualization and animations
- **Canvas:** Responsive, min 700px width × 500px height
- **Frame Rate:** 60fps for smooth scanning animations
- **Data:** Partition metadata and cost formulas defined in JSON

### Implementation Notes

**Mobile Considerations:**
- Screens < 768px: Stack left/right panels vertically
- Table blocks: Show representative sample (e.g., 100 blocks) instead of 5,000 on small screens
- Touch-friendly: Large buttons, swipeable query presets
- Simplified animations (instant pruning highlight instead of gradual fade)

**Accessibility:**
- Keyboard navigation:
  - Tab through controls
  - Arrow keys to adjust date range
  - Enter to run query
  - Escape to clear results
- Screen reader announces:
  - Partition strategy: "Daily partitioning selected. 1,095 partitions created based on order_date column."
  - Query execution: "Query started. Scanning 35 gigabytes across 7 partitions. 1,088 partitions pruned. Cost: 17 cents. Query completed."
  - Cost comparison: "143 times cheaper than unpartitioned. Savings: $24.82 per query."
- High contrast mode: Bold partition boundaries, distinct colors for scanned vs pruned
- Text alternatives: Cost table showing all metrics in accessible format

**Performance:**
- For large tables (>1TB), show representative visualization (e.g., 1 block = 10GB)
- Use canvas instancing for rendering thousands of identical blocks
- Debounce slider changes (500ms before recalculating)
- Precompute partition metadata for instant lookups during query execution

### Data Requirements

**Table Metadata (JSON format):**

```json
{
  "table": {
    "name": "sales",
    "size_bytes": 5497558138880,
    "size_human": "5TB",
    "row_count": 2190000000,
    "date_range": {
      "start": "2021-01-01",
      "end": "2024-01-21"
    },
    "schema": [
      {"name": "order_id", "type": "INT64"},
      {"name": "customer_id", "type": "INT64"},
      {"name": "product_id", "type": "INT64"},
      {"name": "order_date", "type": "DATE"},
      {"name": "order_timestamp", "type": "TIMESTAMP"},
      {"name": "amount", "type": "FLOAT64"}
    ]
  },
  "partition_strategies": {
    "none": {
      "partitions": 1,
      "partition_size_avg_gb": 5000,
      "metadata_cost": 0
    },
    "daily": {
      "partitions": 1095,
      "partition_size_avg_gb": 4.57,
      "metadata_cost": 0.00
    },
    "monthly": {
      "partitions": 36,
      "partition_size_avg_gb": 138.89,
      "metadata_cost": 0
    },
    "yearly": {
      "partitions": 3,
      "partition_size_avg_gb": 1666.67,
      "metadata_cost": 0
    },
    "hourly": {
      "partitions": 26280,
      "partition_size_avg_gb": 0.19,
      "metadata_cost": 0,
      "note": "Exceeds 4,000 partition limit. Requires partition expiration."
    }
  },
  "pricing": {
    "on_demand_per_tb": 5.00,
    "flat_rate_monthly": 20000,
    "storage_per_gb_month": 0.02,
    "free_tier_monthly_gb": 10240
  }
}
```

**Query Presets:**
```json
{
  "query_presets": {
    "last_7_days": {
      "name": "Last 7 Days",
      "sql": "SELECT * FROM sales WHERE order_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)",
      "date_filter": {"days": 7},
      "description": "Common dashboard query"
    },
    "last_month": {
      "name": "Last Month",
      "sql": "SELECT * FROM sales WHERE order_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 1 MONTH)",
      "date_filter": {"days": 30},
      "description": "Monthly reporting"
    },
    "specific_date": {
      "name": "Specific Date",
      "sql": "SELECT * FROM sales WHERE order_date = '2024-01-15'",
      "date_filter": {"days": 1},
      "description": "Single day analysis"
    },
    "year_to_date": {
      "name": "Year-to-Date",
      "sql": "SELECT * FROM sales WHERE order_date >= '2024-01-01'",
      "date_filter": {"days": 21},
      "description": "YTD aggregations"
    },
    "full_table": {
      "name": "Full Table Scan",
      "sql": "SELECT SUM(amount) FROM sales",
      "date_filter": null,
      "description": "Lifetime calculations"
    }
  }
}
```

**Cost Calculation Logic:**
```javascript
function calculateQueryCost(tableSizeGB, partitionStrategy, queryDateFilter) {
  const onDemandRate = 5.00; // $ per TB

  if (!queryDateFilter) {
    // Full table scan
    return {
      dataScannedGB: tableSizeGB,
      cost: (tableSizeGB / 1024) * onDemandRate,
      partitionsPruned: 0
    };
  }

  // Calculate partitions scanned based on strategy
  const daysQueried = queryDateFilter.days;
  let dataScannedGB;

  switch(partitionStrategy) {
    case 'daily':
      dataScannedGB = (tableSizeGB / 1095) * daysQueried;
      break;
    case 'monthly':
      const monthsQueried = Math.ceil(daysQueried / 30);
      dataScannedGB = (tableSizeGB / 36) * monthsQueried;
      break;
    case 'yearly':
      const yearsQueried = Math.ceil(daysQueried / 365);
      dataScannedGB = (tableSizeGB / 3) * yearsQueried;
      break;
    default: // unpartitioned
      dataScannedGB = tableSizeGB;
  }

  return {
    dataScannedGB: dataScannedGB,
    cost: (dataScannedGB / 1024) * onDemandRate,
    partitionsPruned: calculatePartitionsPruned(partitionStrategy, daysQueried)
  };
}
```

## Assessment Integration

After using this MicroSim, students should be able to answer:

1. **Quiz Question:** "BigQuery charges $5 per TB of data scanned. A query scans 250GB. What is the cost?"
   - A) $0.25
   - B) $1.25 ✓
   - C) $5.00
   - D) $25.00

   (Calculation: 250GB / 1024GB per TB × $5 = $1.22, rounded to $1.25)

2. **Conceptual Question:** "Why does partitioning reduce query costs in BigQuery, but not in traditional databases like PostgreSQL?"
   - Expected answer: BigQuery uses columnar storage and charges based on data scanned. Partitioning allows partition pruning—skipping entire partitions based on metadata, reducing data scanned and thus cost. Traditional databases charge based on compute time or don't separate storage/compute costs, so partitioning helps performance but doesn't directly reduce costs.

3. **Design Question:** "You have a 10TB table with queries that filter by user_id (10M unique users). Would partitioning on user_id reduce costs? Why or why not?"
   - Expected answer: No. BigQuery only supports partitioning on DATE, TIMESTAMP, DATETIME, or INTEGER (limited range). user_id with 10M values would create too many partitions (4,000 limit). Instead, use **clustering** on user_id, which sorts data without partition metadata and still improves filtering performance.

4. **Scenario Question:** "Your daily dashboard query scans 2TB and costs $10. It runs every 5 minutes (288 times/day = $2,880/day). You have a $10,000/month budget. What are your options?"
   - Expected answer:
     - **Option 1:** Optimize with partitioning/clustering to reduce data scanned (best)
     - **Option 2:** Switch to BigQuery flat-rate pricing ($20,000/month for unlimited queries, not viable with $10K budget)
     - **Option 3:** Cache results (BI Engine or materialized views) to avoid repeated scans
     - **Option 4:** Reduce query frequency (dashboard refreshes every 30 minutes instead of 5)
     - **Option 5:** Pre-aggregate data in smaller summary tables

## Extension Ideas (Optional)

- **Clustering Visualization:** Show how clustering orders data within partitions, further reducing scan cost
- **Partition Expiration:** Demonstrate auto-deletion of old partitions to manage storage costs
- **Materialized Views:** Show how pre-aggregating data into smaller materialized view reduces query costs
- **BI Engine Caching:** Visualize how frequently-run queries get cached, eliminating scan costs for repeated queries
- **Time-Unit Column Partitioning:** Compare partition by DATE column vs partition by ingestion time (_PARTITIONTIME)
- **Integer Range Partitioning:** Show partitioning by integer columns (e.g., customer_id ranges 0-999, 1000-1999) for non-temporal data
- **Partition Decorator Queries:** Demonstrate querying specific partitions: `SELECT * FROM table$20240115`
- **Flat-Rate vs On-Demand Calculator:** Let students input workload characteristics, calculate which pricing model is cheaper
- **Real-World Case Studies:** Show actual company examples (Spotify saved 80% costs with partitioning, etc.)
- **Cross-Region Queries:** Visualize data transfer costs when querying tables in different regions

---

**Target Learning Outcome:** Students understand that BigQuery costs are determined by data scanned, not returned, and that partitioning reduces costs by enabling partition pruning. Students can choose appropriate partition granularity based on query patterns and calculate cost savings for real-world scenarios.
