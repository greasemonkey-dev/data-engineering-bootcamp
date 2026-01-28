# MicroSim: Slowly Changing Dimension Timeline

## Overview
### Concept Visualized
- **Concept:** Slowly Changing Dimensions (slowly-changing-dims), SCD Type 1 (scd-type-1), SCD Type 2 (scd-type-2), SCD Type 3 (scd-type-3)
- **Learning Goal:** Students understand the differences between SCD Type 1, Type 2, and Type 3 by observing how each strategy handles dimensional attribute changes over time, demonstrating trade-offs between history preservation, storage, and query complexity
- **Difficulty:** Intermediate
- **Chapter:** Week 3-4 (Data Storage & Modeling)

### The "Aha" Moment
When students trigger a customer address change (John moves from California to Texas), they see three parallel timelines diverge: Type 1 overwrites the old value (losing history), Type 2 creates a new row with version dates (preserving full history), and Type 3 adds a "previous_address" column (limited history), demonstrating that SCD type choice determines what historical questions can be answered.

## Interface Design
### Layout
- Left Panel (60%): Three horizontal timeline panels showing SCD Type 1, 2, and 3 side-by-side
- Right Panel (40%): Dimension change simulator and historical query tester

### Controls (Right Panel)
| Control | Type | Range/Options | Default | Effect |
|---------|------|---------------|---------|--------|
| Select Customer | dropdown | "John Smith", "Jane Doe", "Bob Wilson" | "John Smith" | Chooses which customer to track |
| Change Type | dropdown | "Address Change", "Job Title Change", "Phone Number Change" | "Address Change" | Selects attribute to modify |
| Apply Change | button | N/A | N/A | Triggers dimension change event |
| Playback Speed | slider | 0.5x - 3x | 1x | Controls animation speed |
| Reset Timeline | button | N/A | N/A | Returns to initial state |
| Ask Historical Query | dropdown | Various questions | None | Tests what each SCD type can answer |

**Historical Query Examples:**
- "Where did John live on January 15, 2024?"
- "How many customers were in California in Q1 2024?"
- "What was John's previous address before current one?"
- "Show all addresses John has ever had"

**Query Result Panel:**
Shows whether each SCD type can answer the selected question:
```
Query: "Where did John live on January 15, 2024?"
├─ Type 1: ✗ Cannot answer (history overwritten)
├─ Type 2: ✓ Texas (has valid_from/valid_to dates)
└─ Type 3: ✗ Only knows current & previous (not specific dates)

Query: "What was John's address before current?"
├─ Type 1: ✗ Cannot answer
├─ Type 2: ✓ California (can traverse version history)
└─ Type 3: ✓ California (stored in previous_address column)
```

### Visualization (Left Panel)
**What is Displayed:**

**Three horizontal timeline panels stacked vertically:**

**Panel 1: SCD Type 1 (Overwrite)**
- Single row representing current dimension record
- Timeline shows point-in-time snapshots as discrete boxes
- When change occurs: Old value dissolves, new value fades in
- Table structure:
  ```
  | customer_id | name       | address      | phone      | updated_at |
  |-------------|------------|--------------|------------|------------|
  | 1001        | John Smith | California   | 555-0123   | 2024-01-10 |
  ```

**Panel 2: SCD Type 2 (Add Row)**
- Multiple rows stacked vertically showing version history
- Timeline shows valid date ranges as colored bars
- When change occurs: New row slides in below, old row's valid_to date fills in
- Table structure:
  ```
  | surrogate_key | customer_id | name       | address    | valid_from | valid_to   | is_current |
  |---------------|-------------|------------|------------|------------|------------|------------|
  | 1             | 1001        | John Smith | California | 2023-01-01 | 2024-02-15 | No         |
  | 2             | 1001        | John Smith | Texas      | 2024-02-15 | 9999-12-31 | Yes        |
  ```

**Panel 3: SCD Type 3 (Add Column)**
- Single row with additional "previous" columns
- Timeline shows current and previous values as connected boxes
- When change occurs: Current value shifts to "previous" column, new value enters "current"
- Table structure:
  ```
  | customer_id | name       | current_address | previous_address | address_changed_date |
  |-------------|------------|-----------------|------------------|----------------------|
  | 1001        | John Smith | Texas           | California       | 2024-02-15           |
  ```

**Visual Elements:**
- Timeline ruler at bottom showing dates (Jan 2023 → Dec 2024)
- Color coding:
  - Green: Current/active records
  - Blue: Historical records (Type 2)
  - Gray: Overwritten data (shown briefly in Type 1 before deletion)
- Animated transitions showing data movement

**How it Updates:**
- When "Apply Change" clicked:
  1. Timeline playhead animates to change date (1s)
  2. Three panels update simultaneously with different strategies:
     - Type 1: Old cell fades out (red flash), new value fades in (0.8s)
     - Type 2: New row slides in from bottom, old row's end date fills in, is_current flag changes (1.2s)
     - Type 3: Current value slides left to "previous" column, new value slides in from right (1s)
  3. Updated_at / changed_date timestamps update
  4. Query result panel updates showing impact on historical queries

## Educational Flow
### Step 1: Default State
Student sees initial state (January 1, 2024):
- Customer: John Smith
- Address: California
- All three SCD types show identical data
- Timeline playhead at January 1, 2024
- No changes applied yet
- Educational note: "All SCD types start the same. The difference is how they handle changes."

### Step 2: First Interaction - Address Change
Prompt: "John Smith moves from California to Texas on February 15, 2024. Click 'Apply Change' to see how each SCD type handles this."

Result - Synchronized animations across three panels:

**Type 1 Panel:**
- "California" cell flashes red and dissolves
- "Texas" fades in
- updated_at changes to "2024-02-15"
- No trace of California remains
- Icon appears: ⚠ "History Lost"

**Type 2 Panel:**
- Original row's valid_to changes from "9999-12-31" to "2024-02-15"
- is_current flag changes to "No"
- New row slides in below:
  - surrogate_key: 2 (new)
  - customer_id: 1001 (same business key)
  - address: Texas
  - valid_from: 2024-02-15
  - valid_to: 9999-12-31
  - is_current: Yes
- Timeline bars extend showing continuous coverage
- Icon appears: ✓ "Full History Preserved"

**Type 3 Panel:**
- "California" slides from "current_address" to "previous_address"
- "Texas" slides into "current_address"
- address_changed_date: "2024-02-15"
- Icon appears: ⓘ "Limited History (1 Previous Value)"

**Query Panel Updates:**
Query automatically runs: "Where did John live in January 2024?"
- Type 1: ✗ "Cannot answer (only knows Texas)"
- Type 2: ✓ "California (valid_from/valid_to covers Jan 2024)"
- Type 3: ✗ "Cannot answer (no date ranges, only current + previous)"

Students observe: Each strategy handles changes completely differently

### Step 3: Second Change - Phone Number Change
Prompt: "Apply another change: John changes phone number on June 1, 2024. Observe how Type 3 handles multiple changes."

Result:

**Type 1:**
- Phone number updates (overwrites)
- Previous phone lost

**Type 2:**
- Yet another new row created
- Now has 3 versions: Original, After address change, After phone change
- Full audit trail preserved

**Type 3:**
- Previous phone number overwrites previous address in "previous" column
- California address is now completely lost!
- Demonstrates: Type 3 only keeps ONE previous value

Educational callout appears: "Type 3 can only track one level of history. After the second change, California is lost!"

Students discover: Type 3's severe limitation

### Step 4: Historical Query Testing
Prompt: "Select different historical queries from the dropdown to see what each SCD type can answer"

Students try various queries:

**Query: "How many customers lived in California in Q1 2024?"**
- Type 1: ✗ Cannot aggregate historical state
- Type 2: ✓ Can filter by valid_from/valid_to date ranges
- Type 3: ✗ No date ranges for historical aggregation

**Query: "Show John's complete address history"**
- Type 1: ✗ Only current address
- Type 2: ✓ Full version history (California → Texas)
- Type 3: Partial (Current: Texas, Previous: California, older lost)

**Query: "What was John's address immediately before current?"**
- Type 1: ✗ Cannot answer
- Type 2: ✓ Query second-to-last record
- Type 3: ✓ Read previous_address column (simple!)

Pattern discovered:
- Type 1: Simplest schema, no history, cannot answer time-based questions
- Type 2: Most powerful, full history, complex queries with date joins
- Type 3: Middle ground, simple queries for immediate previous value only

### Step 5: Challenge - Trade-off Decision
Scenario presented:
"You're designing a Customer dimension for an e-commerce data warehouse. Requirements:
- 50 million customers
- Customer addresses change ~2% per month (1M updates/month)
- Business needs: 'What was revenue by customer state last quarter?'
- Typical query: JOIN fact_sales with dim_customer on current records

Which SCD type should you use?"

Expected learning:
**Should choose Type 2** because:
- Business needs historical state analysis ("by state last quarter")
- Type 1 cannot answer historical questions
- Type 3 doesn't support date-based queries
- Type 2 storage impact: ~1M new rows/month is acceptable for 50M base
- Query pattern: Most joins use is_current = TRUE (fast), historical analysis uses date ranges

**Trade-offs to consider:**
- Type 2 storage: +24M rows/year (manageable)
- Type 2 query complexity: Need to join on date ranges for historical analysis
- Alternative for current-only queries: Create view filtering is_current = TRUE

Advanced consideration: "For attributes that change frequently but history isn't needed (e.g., last_login_date), use Type 1 for those columns even in Type 2 dimension"

## Technical Specification
- **Library:** p5.js
- **Canvas:** Responsive, min 800px width × 600px height (200px per SCD panel)
- **Frame Rate:** 30fps
- **Data:** Static timeline with scripted changes
  ```javascript
  const timeline_events = [
    { date: "2024-02-15", customer: "John Smith", change: "address", from: "California", to: "Texas" },
    { date: "2024-06-01", customer: "John Smith", change: "phone", from: "555-0123", to: "555-9999" },
    { date: "2024-09-10", customer: "John Smith", change: "address", from: "Texas", to: "Florida" }
  ];

  const historical_queries = [
    { query: "Where did John live on 2024-01-15?", type1: false, type2: "California", type3: false },
    { query: "Show all addresses", type1: "Current only", type2: "Full history", type3: "Current + 1 previous" }
  ];
  ```

## Assessment Integration
After using this MicroSim, students should answer:

1. **Knowledge Check:** What is the main disadvantage of SCD Type 1?
   - a) It's the most complex to implement
   - b) It requires the most storage space
   - c) It cannot answer historical questions ✓
   - d) It's slower for current-state queries

2. **Application:** A dimension has 1 million rows and 5% change monthly. After 12 months with SCD Type 2, approximately how many total rows exist?
   - a) 1 million (same as start)
   - b) 1.6 million (1M + 12 months × 5% × 1M = 1.6M) ✓
   - c) 12 million (one copy per month)
   - d) 5 million

3. **Trade-off Analysis:** When is SCD Type 3 the best choice?
   - a) When you need complete audit history
   - b) When you need to track one previous value and changes are infrequent ✓
   - c) When storage is unlimited
   - d) SCD Type 3 is never the best choice

4. **Query Understanding:** With SCD Type 2, how do you query for current records only?
   - a) Filter WHERE is_current = TRUE ✓
   - b) Filter WHERE valid_to = '9999-12-31' (also correct, alternative approach)
   - c) Use MAX(valid_from)
   - d) Current records cannot be distinguished

## Extension Ideas
- Add fact table join visualization showing how historical analysis works
- Include storage calculator showing disk space impact over time
- Show SQL code snippets for inserting/updating each SCD type
- Add "Hybrid SCD" example (Type 1 for some attributes, Type 2 for others)
- Visualize SCD Type 4 (mini-dimension) and Type 6 (hybrid 1+2+3)
- Include performance comparison: query speed for current vs historical queries
- Add bi-temporal tracking (valid time vs transaction time)
- Show data warehouse ETL process applying SCD logic
- Include real-world examples: customer demographics, product prices, employee roles
