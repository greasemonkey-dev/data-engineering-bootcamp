# MicroSim: Star Schema vs Snowflake Schema Comparison

## Overview
### Concept Visualized
- **Concept:** Star Schema (star-schema), Snowflake Schema (snowflake-schema), Schema Design Tradeoffs (schema-comparison)
- **Learning Goal:** Students understand the structural and performance tradeoffs between star and snowflake schemas by toggling normalization levels and observing changes in join complexity, storage, and query patterns
- **Difficulty:** Intermediate
- **Chapter:** Week 3-4 (Data Storage & Modeling)

### The "Aha" Moment
When students click "Normalize Customer Dimension," they see a single Customer dimension table split into Customer → City → State → Country, transforming the simple 2-table star schema join into a complex 5-table snowflake join, demonstrating the tradeoff between storage savings (reduced redundancy) and query complexity (more joins).

## Interface Design
### Layout
- Left Panel (60%): Side-by-side schema diagrams (Star Schema | Snowflake Schema)
- Right Panel (40%): Dimension normalization controls and metrics comparison

### Controls (Right Panel)
| Control | Type | Range/Options | Default | Effect |
|---------|------|---------------|---------|--------|
| Normalize Product | toggle | On/Off | Off | Splits Product into Product → Category → Department |
| Normalize Customer | toggle | On/Off | Off | Splits Customer into Customer → City → State → Country |
| Normalize Date | toggle | On/Off | Off | Splits Date into Date → Month → Quarter → Year |
| Show Sample Query | dropdown | "Monthly Sales by State", "Product Category Revenue", "Customer Lifetime Value" | None | Displays SQL for selected analytical query |
| Highlight Join Path | button | N/A | N/A | Animates join path through schema for active query |
| Storage Calculator | info panel | N/A | N/A | Shows estimated storage with/without normalization |

**Metrics Comparison Panel:**
| Metric | Star Schema | Snowflake Schema |
|--------|-------------|------------------|
| Tables Count | 5 | Dynamic (5-11) |
| Avg Joins per Query | 2-3 | 4-7 |
| Storage (GB) | Higher | Lower |
| Query Complexity | Simple | Complex |
| Query Performance | Faster | Slower |
| Redundant Data | Yes | Minimal |

### Visualization (Left Panel)
**What is Displayed:**
- **Center:** Fact table (Sales_Fact) shown as yellow rectangle with columns:
  - sale_id, date_key, product_key, customer_key, store_key, quantity, amount
- **Surrounding:** Dimension tables shown as blue rectangles radiating from fact table:
  - Dim_Date, Dim_Product, Dim_Customer, Dim_Store
- **Relationships:** Lines connecting fact table foreign keys to dimension primary keys
- **Table details:** Each table shows key columns, row count estimate, storage size

**How it Updates:**
- When normalization toggle is activated:
  1. Dimension table "splits" with animation (0.8s)
  2. Sub-tables slide out from parent (1s, cascade effect)
  3. New relationship lines draw from parent to child tables (0.5s)
  4. Star schema side stays static, snowflake side updates
  5. Metrics panel updates with new counts

- **Example - Normalize Customer:**
  - Original: Dim_Customer [customer_key, name, address, city, state, country, phone]
  - Becomes:
    - Dim_Customer [customer_key, name, city_key, phone]
    - Dim_City [city_key, city_name, state_key]
    - Dim_State [state_key, state_name, country_key]
    - Dim_Country [country_key, country_name]

- Color coding: Yellow (Fact), Blue (Dimensions), Purple (Normalized Sub-dimensions)
- Visual feedback: Redundant data cells pulse red in star schema, green checkmarks in snowflake for eliminated redundancy

## Educational Flow
### Step 1: Default State
Student sees simple e-commerce star schema:
- 1 Fact table (Sales_Fact) in center: 10M rows, 850 MB
- 4 Dimension tables around it: Date, Product, Customer, Store
- Each dimension shows sample data with visible redundancy (e.g., "California" repeated 50K times in Customer table)
- Metrics show: 5 tables, 2-3 joins average, 1.2 GB total storage

### Step 2: First Interaction
Prompt: "Enable 'Normalize Customer' to see how snowflake schema reduces redundancy"

Result:
- Star schema side: Unchanged
- Snowflake schema side: Customer dimension splits into 4 tables
- Storage metrics update:
  - Star: Dim_Customer = 200 MB (redundant city/state/country)
  - Snowflake: Combined = 145 MB (55 MB saved, 27.5% reduction)
- Metrics panel shows joins increased from 2 to 4 for customer-related queries

Students observe: Storage savings come at cost of query complexity

### Step 3: Exploration
Prompt: "Try different queries using 'Show Sample Query' dropdown and observe join differences"

Pattern they should discover:
- **Star Schema advantages:**
  - Simple queries: `SELECT SUM(amount) FROM sales_fact JOIN dim_customer ON customer_key WHERE state = 'CA'`
  - Only 2 tables involved
  - Faster execution (fewer joins)
  - Easier for business users to understand

- **Snowflake Schema advantages:**
  - Storage efficiency: Eliminates redundancy (state name stored once, not 50K times)
  - Data integrity: State name changes update in one place
  - Better for slowly changing dimensions (covered in SCD MicroSim)

- **Trade-off discovery:**
  - Snowflake saves storage but adds join overhead
  - Star is denormalized for query performance
  - Choice depends on: query patterns, storage costs, update frequency

### Step 4: Challenge
Scenario: "Your data warehouse has 500M fact records and dimensions that rarely change. Queries are run thousands of times per day by business analysts using BI tools. Storage costs $0.02/GB/month. Which schema should you choose?"

Expected learning:
- Should choose STAR SCHEMA
- Reasoning:
  - Query performance is critical (thousands of queries/day)
  - Dimensions rarely change (denormalization doesn't cause update anomalies)
  - Storage cost is minimal compared to query compute costs
  - Business users need simple queries
- Extra credit: Recognize dimensional models are typically denormalized by design for analytics

## Technical Specification
- **Library:** p5.js
- **Canvas:** Responsive, min 800px width × 600px height
- **Frame Rate:** 30fps
- **Data:** Static schema definitions with normalization rules
  ```javascript
  const schemas = {
    star: {
      fact: { name: "Sales_Fact", rows: 10000000, columns: [...] },
      dimensions: [
        { name: "Dim_Customer", rows: 50000, columns: [...] }
      ]
    },
    normalization_rules: {
      customer: {
        splits_into: ["Dim_Customer", "Dim_City", "Dim_State", "Dim_Country"],
        storage_reduction: 0.275
      }
    }
  }
  ```

## Assessment Integration
After using this MicroSim, students should answer:

1. **Knowledge Check:** What is the primary difference between star and snowflake schemas?
   - a) Star schema has more tables
   - b) Snowflake schema normalizes dimension tables ✓
   - c) Star schema cannot handle large datasets
   - d) Snowflake schema is always faster

2. **Application:** A dimension table has 1M rows with city/state/country columns where "California, USA" appears 200K times. Normalizing would split this into 3 tables: Cities (5000 rows), States (50 rows), Countries (3 rows). Approximately how much storage is saved?
   - a) No savings, more tables means more storage
   - b) Minimal savings (< 5%)
   - c) Significant savings (> 20%) ✓
   - d) It depends on the fact table size

3. **Trade-off Analysis:** When is snowflake schema preferred over star schema?
   - a) When query performance is the top priority
   - b) When storage costs are high and dimensions change frequently ✓
   - c) When business users write their own SQL queries
   - d) Snowflake is never preferred, star is always better

## Extension Ideas
- Add query execution time simulator showing performance impact of join count
- Include actual SQL query builder that generates JOIN statements for both schemas
- Show index impact: star needs fewer indexes, snowflake benefits from more indexes
- Add "Hybrid Schema" option showing selective normalization
- Include realistic data samples showing redundancy visually (scrollable data preview)
- Add BI tool perspective showing how tools auto-generate queries differently
- Visualize incremental dimension updates showing update complexity differences
- Include cost calculator: storage cost vs compute cost for queries
