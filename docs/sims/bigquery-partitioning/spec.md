# MicroSim: BigQuery Partitioning Cost Calculator

## Overview
### Concept Visualized
- **Concept:** BigQuery Partitioning (bigquery-partitioning), BigQuery Pricing Model (bigquery-pricing), BigQuery Optimization (bigquery-optimization)
- **Learning Goal:** Students understand how table partitioning in BigQuery dramatically reduces query costs by manipulating partition keys and date ranges, observing the reduction in data scanned and associated costs
- **Difficulty:** Intermediate
- **Chapter:** Week 3-4 (Data Storage & Modeling)

### The "Aha" Moment
When students add a partition on the date column and run a query filtering by last 30 days, they see data scanned drop from 5TB (entire table, $25.00) to 150GB (one partition, $0.75)—a 97% cost reduction—demonstrating why partitioning is essential for large-scale analytics.

## Interface Design
### Layout
- Left Panel (60%): Visual representation of table storage with partition visualization
- Right Panel (40%): Query controls, partition configuration, and cost analysis

### Controls (Right Panel)
| Control | Type | Range/Options | Default | Effect |
|---------|------|---------------|---------|--------|
| Partition Strategy | dropdown | "None", "Daily", "Monthly", "Integer Range" | "None" | Sets table partitioning method |
| Partition Column | dropdown | "created_date", "event_timestamp", "customer_id" | None | Selects column to partition on |
| Query Type | dropdown | "Last 7 Days", "Last 30 Days", "Last Year", "Specific Date", "All Time" | "Last 30 Days" | Determines query date range |
| Table Size | slider | 100GB - 50TB | 5TB | Sets total table size |
| Run Query | button | N/A | N/A | Executes query and shows cost calculation |
| Show Clustering | toggle | On/Off | Off | Adds clustering visualization (advanced) |

**Cost Analysis Panel:**
Displays before/after comparison:
```
Without Partitioning:
├─ Data Scanned: 5,000 GB
├─ Query Cost: $25.00
└─ Partitions Touched: N/A

With Daily Partitioning (last 30 days):
├─ Data Scanned: 150 GB
├─ Query Cost: $0.75
├─ Partitions Touched: 30 of 1,825
├─ Cost Savings: 97%
└─ Monthly Savings (100 queries): $2,425
```

### Visualization (Left Panel)
**What is Displayed:**
- **Unpartitioned Table View:**
  - Single large blue rectangle representing entire table
  - Size label: "5 TB"
  - When query runs, entire rectangle highlights red (all data scanned)

- **Partitioned Table View:**
  - Grid of smaller rectangles, each representing one partition
  - Daily partitions: 1825 squares (5 years of data)
  - Color gradient: Recent partitions (darker blue) → Older partitions (lighter blue, faded)
  - Each partition labeled with date and size (e.g., "2024-01-15, 2.7GB")

**Interactive Visualization:**
- When "Run Query" clicked:
  1. Partitions matching query filter highlight in green (data to scan)
  2. Other partitions fade to gray (skipped via partition pruning)
  3. "Data scanner" animation sweeps across highlighted partitions (1s)
  4. Cost meter fills proportionally to data scanned
  5. Tooltip on hover shows partition metadata: date range, row count, size

**How it Updates:**
- Partition strategy change: Table morphs from single block to partitioned grid (1s animation)
- Query execution: Color wave highlights relevant partitions (0.5s)
- Size slider: Partition sizes scale proportionally
- Clustering toggle: Adds second dimension visualization showing clustered blocks within partitions

## Educational Flow
### Step 1: Default State
Student sees unpartitioned 5TB analytics table:
- Table: "website_events" with 5TB data spanning 5 years (2020-2024)
- Query configured: "Last 30 Days" (typical analytics query)
- Visualization: Single massive blue block
- Cost panel shows: Data Scanned = 5,000 GB, Cost = $25.00 per query
- Note displayed: "This query scans the entire table even though you only need 30 days of data"

### Step 2: First Interaction
Prompt: "Change Partition Strategy to 'Daily' and Partition Column to 'event_timestamp', then click 'Run Query'"

Result - Animated transformation:
1. Table explodes into 1,825 small squares arranged in grid (1.5s animation)
2. Each square represents one day of data
3. Query execution highlights only last 30 partitions in green
4. Cost panel updates:
   - Data Scanned: 5,000 GB → 150 GB (97% reduction)
   - Query Cost: $25.00 → $0.75 (97% savings)
   - Partitions Touched: 30 of 1,825
5. Educational callout appears: "BigQuery only scans partitions matching your WHERE clause filter!"

Students observe: Dramatic cost reduction by avoiding unnecessary data scanning

### Step 3: Exploration
Prompt: "Try different query date ranges and observe how many partitions are scanned"

Pattern they should discover:
- **Query "Last 7 Days":**
  - Scans: 7 partitions, ~35 GB, $0.18
  - More specific = cheaper

- **Query "Last Year":**
  - Scans: 365 partitions, ~1,825 GB, $9.13
  - Still 64% savings vs full scan

- **Query "All Time":**
  - Scans: All 1,825 partitions, 5,000 GB, $25.00
  - No savings (but no penalty either)

- **Key insight:** Partitioning helps most when queries filter on partition key
- **Anti-pattern:** Query without date filter = full table scan despite partitioning

### Step 4: Wrong Partition Key
Prompt: "Change Partition Column to 'customer_id' (integer range partitioning) and run the same 'Last 30 Days' query"

Result:
- Table reorganizes into customer_id ranges: [0-100K], [100K-200K], etc.
- Query still filters on event_timestamp (date)
- All partitions highlight (cannot prune by date anymore)
- Cost panel: Data Scanned = 5,000 GB, Cost = $25.00
- Warning appears: "⚠ Partition key doesn't match your query filter! No cost savings."

Students observe: Partition key must match query patterns

### Step 5: Challenge
Scenario: "Your analytics team runs 100 queries per day, all filtering on event_timestamp. Table grows by 10GB daily. Should you partition by timestamp, and should you use daily or monthly partitions?"

Expected learning:
- **Should partition:** Yes, 97% cost savings × 100 queries = $2,425/month saved
- **Daily vs Monthly:**
  - Daily: Better granularity, most queries need 1-30 days
  - Monthly: Fewer partitions (60 instead of 1,825), but less precise
  - **Recommendation:** Daily partitioning for this access pattern
- **Bonus insight:** With 10GB daily growth, table will be 3.65TB/year—partitioning also helps with partition expiration (auto-delete old data)

Advanced question: "What if queries filter on both timestamp AND customer_id?"
Answer: Use timestamp partitioning + customer_id clustering for best performance

## Technical Specification
- **Library:** p5.js
- **Canvas:** Responsive, min 700px width × 500px height
- **Frame Rate:** 30fps
- **Data:** Formula-based calculations
  ```javascript
  // BigQuery pricing: $5 per TB scanned (as of 2024)
  const PRICE_PER_TB = 5.00;

  function calculateCost(partitionStrategy, queryDateRange, tableSize) {
    if (partitionStrategy === "None") {
      return tableSize * PRICE_PER_TB;
    }

    const dataScanned = calculatePartitionsScanned(queryDateRange) * avgPartitionSize;
    return dataScanned * PRICE_PER_TB;
  }

  // Daily partitions: ~2.7GB average (5TB / 1825 days)
  // Monthly partitions: ~83GB average (5TB / 60 months)
  ```

## Assessment Integration
After using this MicroSim, students should answer:

1. **Knowledge Check:** BigQuery charges based on:
   - a) Number of queries executed
   - b) Amount of data scanned by queries ✓
   - c) Number of tables in your dataset
   - d) Storage space used

2. **Application:** A partitioned table has daily partitions for 3 years (1,095 partitions, 10TB total). A query filters: `WHERE event_date >= '2024-01-01'`. Approximately how much data is scanned?
   - a) 10 TB (entire table)
   - b) ~27 GB (28 days of January 2024)
   - c) ~1 GB (one partition)
   - d) None, the query is free

   **Answer:** b) ~27 GB (queries scan all partitions matching the filter: 28 days in January)

3. **Cost Optimization:** Your queries always filter on user_country. Should you partition the table by user_country?
   - a) Yes, always partition on query filters
   - b) No, BigQuery doesn't support country-based partitioning ✓
   - c) Yes, but only if there are fewer than 4,000 countries
   - d) No, use clustering instead (partitioning limited to date/timestamp/integer; use clustering for strings)

   **Better answer:** d) No, partition on date/timestamp if available, use clustering on user_country

## Extension Ideas
- Add clustering visualization showing how clustering works within partitions
- Include partition expiration simulation (auto-delete old partitions)
- Show ingestion-time vs column-based partitioning differences
- Add "Query Cost History" graph showing monthly costs with/without partitioning
- Include partition size limits (4,000 partition max in BigQuery)
- Show require_partition_filter setting to prevent accidental full scans
- Add real BigQuery SQL examples showing partition metadata queries
- Include clustered vs non-clustered performance comparison
- Visualize columnar storage format and how it enables column pruning
- Add multi-statement transaction cost calculation
