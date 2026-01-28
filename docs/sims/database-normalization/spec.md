# MicroSim: Database Normalization Journey

## Overview

### Concept Visualized
- **Concept:** Database normalization (1NF, 2NF, 3NF) and update/insert/delete anomalies
- **Learning Goal:** Students will understand why normalization matters by starting with a messy denormalized e-commerce table, observing data anomalies, and stepping through normalization forms to see problems eliminated
- **Difficulty:** Intermediate
- **Chapter:** Week 3-4 - Data Storage & Modeling (Relational Databases)

### The "Aha" Moment
When the student tries to update a customer's email address in the denormalized table, they see they must update 15 different order rows. They miss one row and the system now has inconsistent data (update anomaly). Then they normalize to 3NF and try the same update—this time it's a single row change, and all orders automatically reflect the new email. This demonstrates: **normalization isn't just theory, it prevents real bugs**.

## Interface Design

### Layout
- **Top Panel (70% height):** Table visualization showing current schema state with sample data
- **Right Sidebar (20% width):** Controls and normalization progress tracker
- **Bottom Panel (30% height):** Anomaly detection panel and interactive challenges

### Controls (Right Sidebar)

| Control | Type | Range/Options | Default | Effect on Visualization |
|---------|------|---------------|---------|------------------------|
| Normalization Level | step progress | [0NF, 1NF, 2NF, 3NF] | 0NF | Advances through normalization forms, splitting tables |
| Show Data Issues | toggle | on/off | on | Highlights anomalies (duplicate data, inconsistencies) |
| Data Operation | button group | ["Insert Order", "Update Customer", "Delete Product"] | - | Triggers interactive scenarios demonstrating anomalies |
| Next Step | button | - | - | Advances to next normal form with explanation |
| Auto-Play | toggle | on/off | off | Automatically steps through normalization with 3-second pauses |
| Highlight Keys | toggle | on/off | on | Shows primary keys (gold) and foreign keys (blue) |
| Show Dependencies | toggle | on/off | off | Displays functional dependencies with arrows |

**Additional UI Elements:**
- **Normalization Progress Bar:** Visual showing 0NF → 1NF → 2NF → 3NF with checkmarks for completed steps
- **Anomaly Counter:** Badges showing count of each anomaly type (update, insert, delete)
- **Info Box:** Explains current normal form definition and what problem it solves
- **Checklist:** Shows normalization criteria being checked (e.g., "✓ Atomic values", "✗ No partial dependencies")

### Visualization Details (Top Panel)

**What is Displayed:**

**0NF (Unnormalized) State:**
- **Single Table:** `ORDERS` with repeating groups and denormalized data
- **Columns:** `order_id, order_date, customer_name, customer_email, customer_phone, products, quantities, prices, total, shipping_address`
- **Sample Row Data (visible grid):**
  ```
  | order_id | order_date | customer_name | customer_email | products | quantities | prices |
  |----------|------------|---------------|----------------|----------|------------|---------|
  | 1001     | 2024-01-15 | Alice Smith   | alice@email.com | Laptop,Mouse,Keyboard | 1,2,1 | 999.99,25.00,75.00 |
  ```
- **Visual Indicators:**
  - Red wavy underlines on multi-valued cells (products, quantities, prices)
  - Yellow highlight on duplicate data (customer details repeated across multiple orders)
  - Icon badges: "Update Anomaly ×3", "Insert Anomaly ×2", "Delete Anomaly ×1"

**1NF (First Normal Form) State:**
- **Table splits horizontally:** Each product becomes separate row
- **New Schema:** `ORDERS` with columns: `order_id, order_line_id, order_date, customer_name, customer_email, customer_phone, product_name, quantity, price, total, shipping_address`
- **Sample Data:**
  ```
  | order_id | order_line_id | customer_name | customer_email | product_name | quantity | price |
  |----------|---------------|---------------|----------------|--------------|----------|-------|
  | 1001     | 1             | Alice Smith   | alice@email.com | Laptop       | 1        | 999.99 |
  | 1001     | 2             | Alice Smith   | alice@email.com | Mouse        | 2        | 25.00  |
  | 1001     | 3             | Alice Smith   | alice@email.com | Keyboard     | 1        | 75.00  |
  ```
- **Visual Changes:**
  - Red underlines removed (atomic values achieved)
  - Yellow highlights remain (customer data still duplicated)
  - Checkmark on "Atomic Values" criterion
  - Anomaly counter: Update anomalies persist (still 3)

**2NF (Second Normal Form) State:**
- **Tables split by subject:** Order header separated from order lines, customer separated
- **New Schema:**
  - `ORDERS`: `order_id, customer_id, order_date, shipping_address, total`
  - `ORDER_LINES`: `order_line_id, order_id, product_name, quantity, price`
  - `CUSTOMERS`: `customer_id, customer_name, customer_email, customer_phone`
- **Visual Layout:** Three tables side-by-side with arrow connections showing foreign keys
- **Sample Data:**
  ```
  CUSTOMERS:
  | customer_id | customer_name | customer_email |
  |-------------|---------------|----------------|
  | 101         | Alice Smith   | alice@email.com |

  ORDERS:
  | order_id | customer_id | order_date | total |
  |----------|-------------|------------|-------|
  | 1001     | 101         | 2024-01-15 | 1099.99 |

  ORDER_LINES:
  | order_line_id | order_id | product_name | quantity | price |
  |---------------|----------|--------------|----------|-------|
  | 1             | 1001     | Laptop       | 1        | 999.99 |
  ```
- **Visual Changes:**
  - Yellow highlights removed from customer data (no more duplication)
  - Arrows connecting tables: ORDERS.customer_id → CUSTOMERS.customer_id
  - Checkmark on "No Partial Dependencies" criterion
  - Anomaly counter: Update anomalies reduced to 1 (product data still duplicated)

**3NF (Third Normal Form) State:**
- **Further decomposition:** Products separated into own table
- **New Schema:**
  - `ORDERS`: `order_id, customer_id, order_date, shipping_address, total`
  - `ORDER_LINES`: `order_line_id, order_id, product_id, quantity, price_at_purchase`
  - `CUSTOMERS`: `customer_id, customer_name, customer_email, customer_phone`
  - `PRODUCTS`: `product_id, product_name, category, current_price`
- **Visual Layout:** Four tables with relationship lines
- **Sample Data:**
  ```
  PRODUCTS:
  | product_id | product_name | category | current_price |
  |------------|--------------|----------|---------------|
  | 501        | Laptop       | Electronics | 999.99     |

  ORDER_LINES:
  | order_line_id | order_id | product_id | quantity | price_at_purchase |
  |---------------|----------|------------|----------|-------------------|
  | 1             | 1001     | 501        | 1        | 999.99            |
  ```
- **Visual Changes:**
  - All yellow highlights removed (full normalization)
  - Checkmark on "No Transitive Dependencies" criterion
  - Anomaly counter: All anomalies eliminated (green checkmarks)
  - Foreign key arrows: ORDER_LINES.product_id → PRODUCTS.product_id

**Color Coding:**
- Gold: Primary keys (key icon)
- Blue: Foreign keys (arrow icon)
- Red: Violations of current normal form (wavy underlines)
- Yellow: Data redundancy (highlighting duplicate values)
- Green: Correctly normalized data

**Animation Behaviors:**
- **Normalization Transition:** Smooth 1.5-second animation showing:
  1. Highlight columns being moved (fade to yellow)
  2. Table splits apart (columns slide into new table)
  3. Foreign key column appears in source table (fades in)
  4. Arrow animates connecting tables
  5. Sample data repopulates in both tables
- **Anomaly Demonstration:** When user triggers data operation:
  - Affected rows pulse red
  - Update operation: All duplicate rows highlight, then one-by-one update (showing manual effort)
  - Insert anomaly: Red X appears showing operation blocked
  - Delete anomaly: Row deletion causes unintended data loss (highlighted in red)

**Visual Feedback:**
- Hover states: Columns show functional dependency tooltip: "customer_email → customer_id (dependent)"
- Click on cell: Shows all duplicate instances of that value across table (yellow glow)
- Anomaly icon hover: Explains specific anomaly with example

## Educational Flow

### Step 1: Default State (0NF - Unnormalized)
Student sees messy `ORDERS` table with:
- 8 rows of sample e-commerce orders
- Multi-valued cells: `products = "Laptop,Mouse,Keyboard"`
- Duplicate customer data: Alice Smith's email appears in 3 different order rows
- Products listed with denormalized attributes

**Info panel explains:** "This is how many spreadsheets and early databases look. It works for small data, but causes serious problems at scale. Let's see what breaks."

**Anomaly indicators show:**
- Update Anomaly ×3 (customer data duplicated)
- Insert Anomaly ×1 (can't add customer without order)
- Delete Anomaly ×1 (deleting last order loses customer info)

This shows the baseline: functional but problematic structure.

### Step 2: First Interaction - Experience Update Anomaly
**Prompt:** "Alice Smith changed her email to alice.smith@newemail.com. Click 'Update Customer' to change it."

Student clicks "Update Customer" button.

**Interactive Challenge Appears:**
- "Select all rows that need to be updated"
- Student must click each row containing Alice's email
- 3 rows highlight as student clicks
- One row is scrolled below visible area (trap!)
- Student clicks "Apply Update"

**Result:**
- 2 rows update successfully (email changes)
- 1 row remains with old email (the one they missed)
- Red alert appears: "⚠️ UPDATE ANOMALY: Inconsistent data! Alice now has 2 different emails in the system."
- Visual shows: Two rows with alice@email.com, one with alice.smith@newemail.com
- Data corruption highlighted in red

**Info panel explains:** "This is an update anomaly. Because customer data is duplicated across multiple rows, updates are error-prone. Miss one row and you have inconsistent data—different orders show different contact info for the same customer!"

**Prompt:** "Click 'Next Step' to normalize to 1NF and see if it helps."

**What it teaches:** Denormalization forces redundant updates that are error-prone. Students feel the pain of maintaining duplicate data.

### Step 3: First Normal Form (1NF)
Student clicks "Next Step". Animation plays:
- Multi-valued cells (products, quantities, prices) split into separate rows
- Table goes from 8 rows to 23 rows (each product gets own row)
- Atomic values achieved (checkmark)

**Info panel explains:** "First Normal Form (1NF) requires:
1. Atomic values (no lists in cells) ✓
2. Each row must be unique ✓

We've eliminated multi-valued attributes. But notice customer data is still duplicated—update anomalies persist."

**Prompt:** "Try the update challenge again in 1NF."

Student clicks "Update Customer" in 1NF state.

**Result:** Same problem—now customer email appears in even MORE rows (23 instead of 8) because of the split. Anomaly is worse!

**Key insight:** 1NF is necessary but not sufficient. Merely making data atomic doesn't solve redundancy.

### Step 4: Second Normal Form (2NF)
**Prompt:** "Click 'Next Step' to normalize to 2NF. We'll separate customers from orders."

Student advances to 2NF. Animation shows:
- Customer columns (name, email, phone) slide out into new CUSTOMERS table
- customer_id column appears in ORDERS table
- Foreign key arrow connects them
- Customer data now appears once per customer

**Info panel explains:** "Second Normal Form (2NF) requires:
1. Must be in 1NF ✓
2. No partial dependencies (non-key attributes fully depend on entire primary key) ✓

We've separated customers from orders. Each customer's data exists in exactly one place."

**Prompt:** "Now try updating Alice's email again."

Student clicks "Update Customer" in 2NF state.

**Interactive Challenge:**
- "Update Alice's email in the CUSTOMERS table"
- Student clicks single row in CUSTOMERS table
- Clicks "Apply Update"

**Result:**
- Email updates in 1 row in CUSTOMERS table
- Visual shows: All 3 related ORDERS automatically reflect new email (animated foreign key link glows)
- Green checkmark: "✓ UPDATE SUCCESS: Changed 1 row, all orders updated automatically via relationship"

**Info panel explains:** "By normalizing to 2NF, customer data exists in one place. Update once, and all references are automatically correct. No more update anomalies for customer data!"

**Anomaly counter updates:** Update Anomaly ×1 (reduced from 3), Insert Anomaly ×0, Delete Anomaly ×0

**What it teaches:** Proper normalization eliminates update anomalies through single source of truth. Foreign keys maintain relationships without duplication.

### Step 5: Third Normal Form (3NF)
**Info panel:** "One problem remains: Product information (name, category, price) is duplicated in ORDER_LINES. If we change a product's category, we'd have the same update anomaly."

**Prompt:** "Click 'Next Step' for full normalization to 3NF."

Student advances to 3NF. Animation shows:
- Product details slide out into PRODUCTS table
- product_id replaces product_name in ORDER_LINES
- Note: price_at_purchase stays in ORDER_LINES (explained as historical data)

**Info panel explains:** "Third Normal Form (3NF) requires:
1. Must be in 2NF ✓
2. No transitive dependencies (non-key attributes depend only on primary key, not on other non-key attributes) ✓

Products are now separate. Historical price (price_at_purchase) stays in ORDER_LINES because it represents the price at that specific time, not current price."

**Prompt:** "Try all three data operations now that we're in 3NF."

Student tests:
1. **Update Customer Email:** Single row change ✓
2. **Insert New Customer:** Can add customer without requiring an order ✓
3. **Delete Product:** Can delete product without losing order history (product_id preserved in ORDER_LINES) ✓

All anomalies eliminated!

### Step 6: Challenge
**Present scenario:** "Your startup's MVP uses a single denormalized user_activities table with 50,000 rows:
- `user_id, user_name, user_email, user_subscription_tier, activity_type, activity_timestamp, device_name, device_os, device_browser`

You've noticed:
- When users change email, customer support must update 100+ rows per user
- 20 users have inconsistent emails in the system from incomplete updates
- You can't add new users until they perform an activity
- Your database has grown to 2GB despite only 5,000 users

Normalize this to 3NF. How many tables would you create? What are they?"

**Student interaction:** Blank workspace appears with denormalized table. Student must:
1. Identify entities: Users, Activities, Devices
2. Drag columns into appropriate tables
3. Define primary keys
4. Connect tables with foreign keys

**Expected solution:**
```
USERS (user_id PK, user_name, user_email, subscription_tier)
DEVICES (device_id PK, device_name, device_os, device_browser)
ACTIVITIES (activity_id PK, user_id FK, device_id FK, activity_type, activity_timestamp)
```

**Validation:**
- Check that each table has no redundant data
- Check that all functional dependencies are proper
- Show metrics: "Original: 1 table, 50K rows, 2GB. Normalized: 3 tables, 55K rows total, 1.2GB (40% savings)"

**Expected learning:** Students can identify entities, eliminate redundancy, and apply normalization principles to real-world scenarios.

## Technical Specification

### Technology
- **Library:** p5.js for table rendering and animations
- **Canvas:** Responsive, min 800px width × 600px height
- **Frame Rate:** 60fps for smooth table transformations
- **Data:** Sample database records in JSON, normalization rules defined declaratively

### Implementation Notes

**Mobile Considerations:**
- Screens < 768px: Tables stack vertically, foreign key arrows become horizontal
- Touch gestures: Swipe between normalization levels
- Simplified animations on mobile (instant table splits instead of slide animations)
- Accordion view for multiple tables (expand/collapse)

**Accessibility:**
- Keyboard navigation:
  - Tab through cells in table
  - Arrow keys to navigate within table
  - Spacebar to trigger data operations
  - N key to advance to Next normalization level
- Screen reader announces:
  - Table structure: "Orders table with 8 rows, 12 columns. Contains duplicate customer data in 3 rows."
  - Normalization step: "Advancing to Second Normal Form. Separating customers into new table. Customer data duplicated 0 times."
  - Anomaly detection: "Update anomaly detected: Customer email exists in 3 rows. Must update all to maintain consistency."
- High contrast mode: Bold borders, remove background colors
- Text alternatives for all visual indicators (anomaly badges described in text)

**Performance:**
- Render only visible rows (virtual scrolling for tables > 50 rows)
- Use CSS Grid for table layout (more performant than canvas for large tables)
- Debounce interactive operations (prevent double-clicks)
- Preload animation states for smooth transitions

### Data Requirements

**Sample Dataset (JSON format):**

```json
{
  "unnormalized_data": [
    {
      "order_id": 1001,
      "order_date": "2024-01-15",
      "customer_name": "Alice Smith",
      "customer_email": "alice@email.com",
      "customer_phone": "555-0101",
      "products": "Laptop,Mouse,Keyboard",
      "quantities": "1,2,1",
      "prices": "999.99,25.00,75.00",
      "total": 1099.99,
      "shipping_address": "123 Main St, Boston, MA"
    },
    {
      "order_id": 1002,
      "order_date": "2024-01-16",
      "customer_name": "Alice Smith",
      "customer_email": "alice@email.com",
      "customer_phone": "555-0101",
      "products": "Monitor",
      "quantities": "1",
      "prices": "299.99",
      "total": 299.99,
      "shipping_address": "123 Main St, Boston, MA"
    },
    {
      "order_id": 1003,
      "order_date": "2024-01-17",
      "customer_name": "Bob Johnson",
      "customer_email": "bob@email.com",
      "customer_phone": "555-0102",
      "products": "Keyboard,Mouse",
      "quantities": "1,1",
      "prices": "75.00,25.00",
      "total": 100.00,
      "shipping_address": "456 Oak Ave, Seattle, WA"
    }
  ],
  "anomalies": [
    {
      "type": "update",
      "description": "Customer email duplicated across multiple orders",
      "affected_rows": [1001, 1002],
      "entity": "customer",
      "solution_normal_form": "2NF"
    },
    {
      "type": "insert",
      "description": "Cannot add customer without placing an order",
      "entity": "customer",
      "solution_normal_form": "2NF"
    },
    {
      "type": "delete",
      "description": "Deleting last order removes customer information",
      "affected_entities": ["customer", "order"],
      "solution_normal_form": "2NF"
    },
    {
      "type": "update",
      "description": "Product details duplicated across order lines",
      "affected_rows": "multiple",
      "entity": "product",
      "solution_normal_form": "3NF"
    }
  ],
  "normalization_rules": {
    "1NF": {
      "definition": "Each cell contains atomic (indivisible) values, and each row is unique",
      "transformations": [
        "Split multi-valued attributes into separate rows",
        "Create composite primary key (order_id, order_line_id)"
      ]
    },
    "2NF": {
      "definition": "In 1NF and all non-key attributes fully depend on the entire primary key",
      "transformations": [
        "Extract customers into CUSTOMERS table",
        "Add customer_id foreign key to ORDERS table",
        "Remove partial dependencies"
      ]
    },
    "3NF": {
      "definition": "In 2NF and no transitive dependencies (non-key attributes depend only on primary key)",
      "transformations": [
        "Extract products into PRODUCTS table",
        "Add product_id foreign key to ORDER_LINES table",
        "Keep price_at_purchase in ORDER_LINES (represents historical fact, not product attribute)"
      ]
    }
  }
}
```

**Functional Dependencies:**
```
0NF state:
  order_id → order_date, customer_name, customer_email, total, shipping_address, products
  customer_email → customer_name, customer_phone (transitive)
  products → prices (multi-valued dependency)

2NF state:
  order_id → customer_id, order_date, total, shipping_address
  customer_id → customer_name, customer_email, customer_phone
  (order_id, order_line_id) → product_name, quantity, price

3NF state:
  order_id → customer_id, order_date, total, shipping_address
  customer_id → customer_name, customer_email, customer_phone
  (order_id, order_line_id) → product_id, quantity, price_at_purchase
  product_id → product_name, category, current_price
```

## Assessment Integration

After using this MicroSim, students should be able to answer:

1. **Quiz Question:** "A table is in 2NF if:"
   - A) It has atomic values
   - B) It is in 1NF and has no partial dependencies ✓
   - C) It has no transitive dependencies
   - D) It has a primary key

2. **Conceptual Question:** "Explain why denormalized databases suffer from update anomalies. Give a concrete example."
   - Expected answer: Update anomalies occur when the same fact is stored in multiple places. If customer email is duplicated across 10 order rows, updating it requires changing 10 rows. Missing one row creates inconsistent data. Normalization eliminates this by storing each fact once.

3. **Design Question:** "You have a table: `EMPLOYEES (emp_id, emp_name, dept_id, dept_name, dept_location)`. What normal form is this? What's the problem? How would you normalize it?"
   - Expected answer:
     - Violates 3NF (transitive dependency: emp_id → dept_id → dept_name, dept_location)
     - Problem: Department info duplicated for every employee. Changing dept_name requires updating many employee rows.
     - Solution: Split into EMPLOYEES (emp_id, emp_name, dept_id) and DEPARTMENTS (dept_id, dept_name, dept_location)

4. **Trade-off Question:** "When might you intentionally denormalize a properly normalized database?"
   - Expected answer:
     - Data warehousing (star schema for query performance)
     - Read-heavy applications where query speed matters more than update efficiency
     - Caching frequently joined data
     - But: Must accept update anomaly risks and implement careful update procedures

## Extension Ideas (Optional)

- **Boyce-Codd Normal Form (BCNF):** Show edge cases where 3NF isn't sufficient (overlapping candidate keys)
- **Fourth Normal Form (4NF):** Demonstrate multi-valued dependencies
- **Denormalization Calculator:** Show read vs write operation costs, let students decide when denormalization is justified
- **Real Database Connection:** Load actual database schema, detect normalization violations
- **Performance Comparison:** Show query execution times for normalized vs denormalized schemas under different workloads
- **Migration Tool:** Generate ALTER TABLE scripts to normalize existing database
- **Anomaly Sandbox:** Free-form mode where students create any schema and system detects all possible anomalies
- **Historical Perspective:** Show how old databases evolved (hierarchical → network → relational) and why normalization matters
- **NoSQL Connection:** Explain why document databases (MongoDB) intentionally denormalize, and when that's appropriate

---

**Target Learning Outcome:** Students understand that normalization eliminates data anomalies by removing redundancy, can identify which normal form a table satisfies, can normalize tables to 3NF, and recognize when denormalization is a justified trade-off.
