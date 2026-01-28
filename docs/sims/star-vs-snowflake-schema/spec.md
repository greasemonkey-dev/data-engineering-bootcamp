# MicroSim: Star Schema vs Snowflake Schema Comparison

## Overview

### Concept Visualized
- **Concept:** Dimensional modeling—star schema vs snowflake schema design patterns
- **Learning Goal:** Students will understand the structural and performance trade-offs between star and snowflake schemas by interacting with side-by-side models and observing query path differences
- **Difficulty:** Intermediate
- **Chapter:** Week 3-4 - Data Storage & Modeling (Data Warehousing)

### The "Aha" Moment
When the student traces a query through both schemas, they see the star schema requires 2 joins (fact → dimension) while the snowflake schema requires 4 joins (fact → dimension → subdimension → subdimension). The star schema query completes in 85ms, the snowflake in 210ms. But then they notice the snowflake schema saves 35% storage space due to normalization. This viscerally demonstrates the classic **denormalization trade-off: query speed vs storage efficiency**.

## Interface Design

### Layout
- **Left Panel (50% width):** Star schema visualization with fact table and dimension tables
- **Right Panel (50% width):** Snowflake schema visualization with normalized dimension tables
- **Bottom Panel (full width, 20% height):** Interactive controls, query builder, and metrics comparison

### Controls (Bottom Panel)

| Control | Type | Range/Options | Default | Effect on Visualization |
|---------|------|---------------|---------|------------------------|
| Query Scenario | dropdown | ["Product Sales by Category", "Customer Demographics", "Time Period Analysis", "Multi-Dimension Drill-Down"] | "Product Sales by Category" | Changes highlighted query path and SQL |
| Run Query | button | - | - | Animates query execution through both schemas simultaneously |
| Show SQL | toggle | on/off | on | Displays SQL queries under each schema |
| Highlight Joins | toggle | on/off | on | Colors join paths as query animates |
| Data Volume | slider | 1K-10M rows | 100K | Updates metrics (storage, query time) |
| Show Storage | toggle | on/off | off | Displays table size metrics on each table |
| Toggle Normalization | button | - | - | Animates snowflake dimension collapsing/expanding |

**Additional UI Elements:**
- **Metrics Panel:** Side-by-side comparison table showing:
  - Total Joins Required
  - Query Execution Time
  - Storage Size
  - Maintenance Complexity
  - Update Anomaly Risk
- **SQL Display:** Shows queries for both schemas with JOIN clauses highlighted
- **Legend:** Explains table types (fact=orange, dimension=blue, subdimension=light blue)

### Visualization Details (Left & Right Panels)

**What is Displayed:**

**Star Schema (Left Panel):**
- **Fact Table (center):** Large orange rectangle labeled "FACT_SALES"
  - Columns shown: `sale_id, product_id, customer_id, date_id, store_id, quantity, revenue`
  - Primary key marked with key icon
  - Foreign keys marked with arrow icons
  - Row count badge: "2.5M rows"

- **Dimension Tables (surrounding fact table):** Blue rectangles connected to fact table with lines
  - `DIM_PRODUCT`: `product_id, product_name, category, subcategory, brand, price`
  - `DIM_CUSTOMER`: `customer_id, name, age, gender, city, state, country, segment`
  - `DIM_DATE`: `date_id, date, day_of_week, month, quarter, year, is_holiday`
  - `DIM_STORE`: `store_id, store_name, city, state, region, square_footage`

- **Relationship Lines:** Thick gray lines connecting fact table foreign keys to dimension table primary keys
- **Visual Layout:** Star pattern with fact in center, dimensions radiating outward

**Snowflake Schema (Right Panel):**
- **Fact Table (center):** Identical to star schema—`FACT_SALES` with same columns
- **Dimension Tables:** Same as star but with fewer columns (normalized)
  - `DIM_PRODUCT`: `product_id, product_name, subcategory_id, brand_id, price`
  - `DIM_CUSTOMER`: `customer_id, name, age, gender, city_id, segment_id`
  - `DIM_DATE`: `date_id, date, day_of_week, month_id, is_holiday`
  - `DIM_STORE`: `store_id, store_name, city_id, square_footage`

- **Subdimension Tables:** Light blue rectangles, smaller, connected to dimension tables
  - `DIM_PRODUCT_CATEGORY`: `subcategory_id, subcategory_name, category_id`
  - `DIM_CATEGORY`: `category_id, category_name`
  - `DIM_BRAND`: `brand_id, brand_name`
  - `DIM_CUSTOMER_SEGMENT`: `segment_id, segment_name, segment_description`
  - `DIM_GEO_CITY`: `city_id, city_name, state_id`
  - `DIM_GEO_STATE`: `state_id, state_name, country_id`
  - `DIM_GEO_COUNTRY`: `country_id, country_name, region`
  - `DIM_MONTH`: `month_id, month_name, quarter_id`
  - `DIM_QUARTER`: `quarter_id, quarter_name, year`

- **Relationship Lines:** Connected as tree structure (fact → dimension → subdimension → subdimension)
- **Visual Layout:** Snowflake pattern with branches extending from dimensions

**Color Coding:**
- Orange: Fact tables
- Dark Blue: First-level dimension tables
- Light Blue: Normalized subdimension tables
- Green: Active query path during animation
- Yellow: Tables currently being read during query animation

**How it Updates:**

**When "Run Query" button clicked:**
1. **Initial State (500ms):** Both fact tables pulse yellow (starting point)
2. **Star Schema Animation (left):**
   - Arrow animates from fact table to DIM_PRODUCT (300ms)
   - DIM_PRODUCT highlights yellow, join condition appears: "ON f.product_id = p.product_id"
   - Arrow animates from DIM_PRODUCT to DIM_CUSTOMER (300ms)
   - DIM_CUSTOMER highlights yellow, join condition appears
   - Green checkmark appears on star schema (query complete)
   - Timer shows: **85ms**

3. **Snowflake Schema Animation (right, simultaneous):**
   - Arrow animates from fact table to DIM_PRODUCT (300ms)
   - DIM_PRODUCT highlights yellow
   - Arrow animates from DIM_PRODUCT to DIM_PRODUCT_CATEGORY (300ms)
   - DIM_PRODUCT_CATEGORY highlights yellow
   - Arrow animates from DIM_PRODUCT_CATEGORY to DIM_CATEGORY (300ms)
   - DIM_CATEGORY highlights yellow
   - Arrow animates to DIM_CUSTOMER, then DIM_GEO_CITY, etc. (continues through normalized hierarchy)
   - Green checkmark appears on snowflake schema (query complete)
   - Timer shows: **210ms**

4. **Metrics Panel Updates:**
   - "Total Joins" counter increments during animation
   - "Query Time" counts up in milliseconds
   - "Storage Size" displays final numbers (Star: 3.2 GB, Snowflake: 2.1 GB)
   - Highlight differences in green (snowflake storage advantage) and red (snowflake query time disadvantage)

**When "Toggle Normalization" button clicked:**
- Snowflake schema animates: subdimension tables slide into parent dimensions (800ms)
- Columns merge and duplicate data appears (showing denormalization)
- Schema morphs into star schema structure
- Reverse animation shows normalization: duplicate data highlighted, then split into subdimensions

**When "Data Volume" slider changes:**
- Storage size metrics update proportionally
- Query time updates based on join cost formulas
- Row count badges on tables update

**Visual Feedback:**
- Hover states: Tables show detailed column list with sample values
- Join line hover: Shows join cardinality (1:many) and join condition
- Active query path: Animated data flow particles moving along join lines
- Performance indicator: Red/yellow/green badges on query times

## Educational Flow

### Step 1: Default State
Student sees both schemas side-by-side with **"Product Sales by Category"** query scenario loaded.

**SQL displayed under star schema:**
```sql
SELECT c.category_name, SUM(f.revenue)
FROM FACT_SALES f
JOIN DIM_PRODUCT p ON f.product_id = p.product_id
GROUP BY c.category_name;
```

**SQL displayed under snowflake schema:**
```sql
SELECT cat.category_name, SUM(f.revenue)
FROM FACT_SALES f
JOIN DIM_PRODUCT p ON f.product_id = p.product_id
JOIN DIM_PRODUCT_CATEGORY sc ON p.subcategory_id = sc.subcategory_id
JOIN DIM_CATEGORY cat ON sc.category_id = cat.category_id
GROUP BY cat.category_name;
```

**Info panel explains:** "Both schemas answer the same business question: 'What is revenue by product category?' But notice the star schema has category already denormalized in DIM_PRODUCT, while snowflake normalizes it into separate tables."

This shows the fundamental structural difference before any performance discussion.

### Step 2: First Interaction
**Prompt:** "Click 'Run Query' and watch how the query flows through each schema. Count the joins."

Student clicks Run Query and watches simultaneous animation.

**Result:**
- Star schema: 1 join (fact → product dimension), completes in 85ms
- Snowflake schema: 3 joins (fact → product → subcategory → category), completes in 210ms
- Metrics panel highlights: "Star schema: 2.5x faster for this query"

**Info panel explains:** "Star schema is faster because it denormalizes dimensions—category is already in the product dimension. Snowflake schema normalizes dimensions to eliminate redundancy, but requires more joins."

**Prompt:** "Now toggle 'Show Storage' to see the space trade-off."

Student enables storage display.

**Result:**
- Star schema: DIM_PRODUCT shows 1.2 GB (450K rows × 8 columns with repeated category/subcategory values)
- Snowflake schema: DIM_PRODUCT 850 MB + DIM_PRODUCT_CATEGORY 5 MB + DIM_CATEGORY 1 MB = 856 MB total
- Savings: 35% less storage with snowflake

**What it teaches:**
- Star schema trades storage for query speed (denormalization)
- Snowflake schema trades query speed for storage efficiency (normalization)
- The choice depends on priorities: query performance vs storage costs

### Step 3: Exploration
**Prompt:** "Try different query scenarios. Which queries benefit more from star schema? Do any queries perform similarly in both?"

Student selects **"Customer Demographics"** scenario:
- Query needs customer name, city, state, country
- Star schema: 1 join (fact → customer), 95ms
- Snowflake schema: 4 joins (fact → customer → city → state → country), 285ms
- Star schema advantage is even larger for deeply nested hierarchies

Student selects **"Time Period Analysis"**:
- Query needs just date and month
- Star schema: 1 join, 80ms
- Snowflake schema: 2 joins (fact → date → month), 120ms
- Smaller difference for shallow hierarchies

**Key insight:** The more levels of hierarchy (geography: city → state → country → region), the more snowflake schema suffers in query performance. Star schema flattens all hierarchy levels into one table.

**Prompt:** "What happens when you need to UPDATE a product's category in each schema?"

Student clicks on DIM_PRODUCT category field in star schema.

**Info panel shows:** "Star schema update: Must update category in 15,000 rows (every product in 'Electronics' category). Risk: Update anomaly if you miss some rows."

Student clicks on DIM_CATEGORY in snowflake schema.

**Info panel shows:** "Snowflake schema update: Update 1 row in DIM_CATEGORY. All products automatically reflect change through foreign key relationship."

**Key insight:** Snowflake schema is easier to maintain—updates happen in one place. Star schema has data redundancy, making updates error-prone.

### Step 4: Challenge
**Present scenario:** "You're designing a data warehouse for an e-commerce company. They have:
- 10 million orders (fact table)
- 500,000 products with 3-level category hierarchy (Electronics → Laptops → Gaming Laptops)
- 2 million customers across 50,000 cities in 200 countries
- Dashboard queries run every 5 minutes analyzing sales by category, geography, time
- Storage costs $0.02/GB/month, and you have budget constraints
- Marketing team occasionally updates product categories (10-20 times/month)

Should you use star schema or snowflake schema? Why?"

Student must:
1. Use Data Volume slider to set 10M rows
2. Run different query scenarios to measure performance
3. Check storage metrics
4. Consider maintenance complexity

**Expected reasoning:**
- **Query frequency:** Dashboard runs every 5 minutes → query performance is critical → **favors star schema**
- **Storage costs:** Budget constraints and large dimension tables → **favors snowflake schema**
- **Maintenance:** Frequent category updates → easier in snowflake → **favors snowflake schema**
- **Data volume:** 10M facts with 500K products → denormalizing products adds significant storage → **favors snowflake schema**

**Best answer:** Hybrid approach or snowflake schema with strategic denormalization:
- Use snowflake schema for geographic hierarchy (saves most space)
- Consider denormalizing product category into DIM_PRODUCT (most frequently queried)
- Or use materialized views to pre-join snowflake schema for dashboards
- Or use columnar storage (like BigQuery) where snowflake's extra joins matter less

**Advanced insight shown:** "In modern cloud data warehouses with columnar storage and massively parallel processing, snowflake schema join penalties are minimized. But for traditional row-based databases, star schema often wins."

## Technical Specification

### Technology
- **Library:** p5.js for schema rendering and animations
- **Canvas:** Responsive, min 1000px width × 600px height (side-by-side), stacks on mobile
- **Frame Rate:** 60fps for smooth query path animations
- **Data:** Schema metadata and query paths defined in JSON

### Implementation Notes

**Mobile Considerations:**
- Screens < 1024px: Stack schemas vertically instead of side-by-side
- Touch-friendly: Tap tables to expand column details
- Simplified animation on mobile (remove particle effects, instant transitions for slower devices)
- Pinch-to-zoom enabled for detailed table inspection

**Accessibility:**
- Keyboard navigation:
  - Tab through tables in reading order (fact → dimensions → subdimensions)
  - Arrow keys to navigate relationships
  - Enter to trigger query animation
  - Space to pause/resume animation
- Screen reader announces:
  - Table structure: "Fact table FACT_SALES with 7 columns, connected to 4 dimension tables"
  - Query progress: "Joining fact table to product dimension... now joining product to category..."
  - Metrics comparison: "Star schema completed in 85 milliseconds with 2 joins. Snowflake schema completed in 210 milliseconds with 5 joins."
- High contrast mode: Increase line thickness, remove subtle gradients
- Text alternatives for all animations (text-based query execution log)

**Performance:**
- Use CSS transforms for table positioning (hardware-accelerated)
- Canvas layering: Static schema on bottom layer, animations on top layer
- Debounce slider changes (400ms before recalculating metrics)
- Limit simultaneous animations to 2 paths (star + snowflake)

### Data Requirements

**Schema Definition (JSON format):**

```json
{
  "star_schema": {
    "tables": {
      "FACT_SALES": {
        "type": "fact",
        "columns": ["sale_id", "product_id", "customer_id", "date_id", "store_id", "quantity", "revenue"],
        "primary_key": "sale_id",
        "foreign_keys": ["product_id", "customer_id", "date_id", "store_id"],
        "row_count": 2500000,
        "size_mb": 850
      },
      "DIM_PRODUCT": {
        "type": "dimension",
        "columns": ["product_id", "product_name", "category", "subcategory", "brand", "price"],
        "primary_key": "product_id",
        "row_count": 450000,
        "size_mb": 1200,
        "sample_data": {
          "product_id": 101,
          "product_name": "Gaming Laptop X1",
          "category": "Electronics",
          "subcategory": "Laptops",
          "brand": "TechCorp",
          "price": 1299.99
        }
      }
    },
    "relationships": [
      {"from": "FACT_SALES.product_id", "to": "DIM_PRODUCT.product_id", "cardinality": "many-to-one"}
    ]
  },
  "snowflake_schema": {
    "tables": {
      "FACT_SALES": { "same": "as star schema" },
      "DIM_PRODUCT": {
        "type": "dimension",
        "columns": ["product_id", "product_name", "subcategory_id", "brand_id", "price"],
        "primary_key": "product_id",
        "foreign_keys": ["subcategory_id", "brand_id"],
        "row_count": 450000,
        "size_mb": 850
      },
      "DIM_PRODUCT_CATEGORY": {
        "type": "subdimension",
        "columns": ["subcategory_id", "subcategory_name", "category_id"],
        "primary_key": "subcategory_id",
        "foreign_keys": ["category_id"],
        "row_count": 500,
        "size_mb": 5
      },
      "DIM_CATEGORY": {
        "type": "subdimension",
        "columns": ["category_id", "category_name"],
        "primary_key": "category_id",
        "row_count": 50,
        "size_mb": 1
      }
    }
  },
  "query_scenarios": {
    "product_sales_by_category": {
      "description": "Analyze revenue by product category",
      "star_query_path": ["FACT_SALES", "DIM_PRODUCT"],
      "star_joins": 1,
      "star_time_ms": 85,
      "snowflake_query_path": ["FACT_SALES", "DIM_PRODUCT", "DIM_PRODUCT_CATEGORY", "DIM_CATEGORY"],
      "snowflake_joins": 3,
      "snowflake_time_ms": 210
    }
  }
}
```

**Performance Calculation Formulas:**
```javascript
// Query time = base_join_cost * num_joins * (row_count_factor)
star_time = 50 + (num_joins * 15) * (row_count / 100000)
snowflake_time = 50 + (num_joins * 25) * (row_count / 100000) // Higher cost per join due to normalization

// Storage calculation
star_storage = fact_table_size + sum(denormalized_dimension_sizes)
snowflake_storage = fact_table_size + sum(normalized_dimension_sizes) + sum(subdimension_sizes)
// Snowflake typically 20-40% smaller due to elimination of redundancy
```

## Assessment Integration

After using this MicroSim, students should be able to answer:

1. **Quiz Question:** "A star schema typically has _____ query performance but _____ storage requirements compared to snowflake schema."
   - A) Better, higher ✓
   - B) Worse, lower
   - C) Better, lower
   - D) Worse, higher

2. **Conceptual Question:** "Why does snowflake schema require more joins than star schema for the same analytical query?"
   - Expected answer: Snowflake schema normalizes dimensions into multiple related tables (e.g., splitting geography into city → state → country tables), so queries must join through hierarchy levels. Star schema denormalizes dimensions into flat tables, requiring only one join from fact to dimension.

3. **Trade-off Question:** "Your data warehouse has a product dimension with 1 million products across a 4-level category hierarchy. Dashboard queries frequently filter by top-level category. Should you use star or snowflake schema? What's a hybrid approach?"
   - Expected answer:
     - Pure star: Fast queries (1 join) but massive storage waste (category repeated 1M times)
     - Pure snowflake: Saves storage but requires 4 joins
     - **Hybrid (best):** Snowflake structure but denormalize top-level category into product dimension (most queried field), keep other hierarchy levels normalized
     - Or use materialized aggregate tables pre-joined at category level

4. **Scenario Question:** "A marketing team needs to rename a product category from 'Electronics' to 'Consumer Electronics'. How does the update differ between star and snowflake schema? Which is safer?"
   - Expected answer:
     - Star schema: Must UPDATE millions of rows in product dimension (WHERE category = 'Electronics'). Risk of inconsistency if update fails partway.
     - Snowflake schema: UPDATE 1 row in category table. All products automatically reflect change via foreign key.
     - Snowflake is safer for maintenance—single source of truth.

## Extension Ideas (Optional)

- **Hybrid Schema Builder:** Let students create custom hybrid approach, denormalizing selected columns while keeping others normalized
- **Query Optimizer Simulation:** Show how modern query optimizers handle snowflake joins (join elimination, predicate pushdown)
- **Real Database Comparison:** Load actual data into star/snowflake schemas in BigQuery, run real queries, compare actual execution times
- **Columnar Storage Impact:** Toggle between row-based and columnar storage, showing how columnar reduces snowflake join penalty
- **Aggregate Table Strategy:** Show how pre-aggregated fact tables at different grain levels can make snowflake queries fast
- **Slowly Changing Dimension Integration:** Demonstrate how SCD Type 2 complicates snowflake schema (more tables to historize)
- **Dimension Role-Playing:** Show how one dimension (like date) can be used multiple times in fact table (order_date, ship_date, delivery_date), and how this affects schema design
- **Fact Table Types:** Compare transaction fact tables vs snapshot fact tables vs accumulating snapshot fact tables in both schemas
- **Indexing Strategy:** Show how different index strategies (bitmap indexes on dimensions, partitioning on fact table) affect query performance in each schema

---

**Target Learning Outcome:** Students understand that star and snowflake schemas represent a trade-off between query performance and storage/maintenance efficiency, can identify when each is appropriate, and recognize that hybrid approaches often provide the best balance for real-world requirements.
