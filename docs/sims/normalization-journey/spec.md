# MicroSim: Database Normalization Journey

## Overview
### Concept Visualized
- **Concept:** First Normal Form (first-normal-form), Second Normal Form (second-normal-form), Third Normal Form (third-normal-form), Normalization (normalization)
- **Learning Goal:** Students understand database normalization by transforming a denormalized table step-by-step through 1NF, 2NF, and 3NF, observing how data anomalies are eliminated at each stage
- **Difficulty:** Intermediate
- **Chapter:** Week 3-4 (Data Storage & Modeling)

### The "Aha" Moment
When students progress from 2NF to 3NF, they see a seemingly normalized Customer table split further because ZIP code determines city/state (transitive dependency), demonstrating that normalization isn't just about removing duplication—it's about removing dependencies that cause update anomalies.

## Interface Design
### Layout
- Left Panel (60%): Visual table representation showing current normalization form
- Right Panel (40%): Step controls, anomaly detector, and educational explanations

### Controls (Right Panel)
| Control | Type | Range/Options | Default | Effect |
|---------|------|---------------|---------|--------|
| Normalization Stage | stepper | "Unnormalized" → "1NF" → "2NF" → "3NF" | Unnormalized | Advances through normalization stages |
| Show Next Step | button | N/A | N/A | Advances to next normal form with animation |
| Reset Journey | button | N/A | N/A | Returns to unnormalized starting state |
| Highlight Anomalies | toggle | On/Off | On | Pulses cells with update/insert/delete anomalies |
| Show Dependencies | toggle | On/Off | Off | Draws arrows showing functional dependencies |
| Speed | slider | 0.5x - 2x | 1x | Controls animation speed |

**Anomaly Detector Panel:**
- Shows active anomalies in current form:
  - Insert Anomaly: Can't add X without Y
  - Update Anomaly: Changing X requires updating N rows
  - Delete Anomaly: Deleting X loses information about Y
- Count decreases as normalization progresses

**Educational Explanation Panel:**
- Displays current rule being applied:
  - **1NF:** "Eliminate repeating groups, ensure atomic values"
  - **2NF:** "Remove partial dependencies (non-key attributes depending on part of composite key)"
  - **3NF:** "Remove transitive dependencies (non-key attributes depending on other non-key attributes)"

### Visualization (Left Panel)
**What is Displayed:**
- Tables rendered as spreadsheet-style grids with column headers and sample rows
- Primary keys highlighted in yellow
- Foreign keys highlighted in blue
- Cells with anomalies pulse with red border when "Highlight Anomalies" is on
- Dependency arrows (when enabled) show functional dependencies: ColumnA → ColumnB

**Initial Unnormalized State (Example: Order Management):**
```
Orders Table (5 rows shown):
| OrderID | OrderDate | CustomerName | CustomerAddress | Products | Quantities | Prices |
|---------|-----------|--------------|-----------------|----------|------------|--------|
| 101     | 2024-01-15| John Smith   | 123 Main, CA    | Mouse,Keyboard | 2,1    | 25,80  |
| 102     | 2024-01-16| Jane Doe     | 456 Oak, NY     | Monitor  | 1          | 300    |
| 103     | 2024-01-16| John Smith   | 123 Main, CA    | Mouse    | 5          | 25     |
```

Problems visible:
- Multi-valued attributes: Products, Quantities, Prices (comma-separated)
- Redundant data: CustomerAddress repeated for John Smith
- Update anomaly: If John Smith moves, must update multiple rows

## Educational Flow
### Step 1: Default State (Unnormalized)
Student sees the messy unnormalized Orders table with visible problems:
- Anomaly Detector shows: 3 insert anomalies, 5 update anomalies, 2 delete anomalies
- Multi-valued fields highlighted: "Products" column contains "Mouse,Keyboard"
- Redundant data pulsing: John Smith's address appears twice
- Instructions: "Click 'Show Next Step' to achieve First Normal Form"

### Step 2: First Interaction - Transition to 1NF
Prompt: "Click 'Show Next Step' to eliminate repeating groups and multi-valued attributes"

Result - Animated transformation:
1. Multi-valued Product cells "explode" into separate rows (1s animation)
2. Table expands from 3 rows to 5 rows:
```
Orders Table (1NF):
| OrderID | OrderDate | CustomerName | CustomerAddress | Product  | Quantity | Price |
|---------|-----------|--------------|-----------------|----------|----------|-------|
| 101     | 2024-01-15| John Smith   | 123 Main, CA    | Mouse    | 2        | 25    |
| 101     | 2024-01-15| John Smith   | 123 Main, CA    | Keyboard | 1        | 80    |
| 102     | 2024-01-16| Jane Doe     | 456 Oak, NY     | Monitor  | 1        | 300   |
| 103     | 2024-01-16| John Smith   | 123 Main, CA    | Mouse    | 5        | 25    |
```
3. Composite primary key appears: (OrderID, Product) highlighted in yellow
4. Anomaly count updates: Insert anomalies eliminated, but update/delete anomalies persist
5. Explanation updates: "✓ Atomic values achieved. But notice: customer info still repeats..."

Students observe: Table is cleaner but still has redundancy

### Step 3: Transition to 2NF
Prompt: "Click 'Show Next Step' to remove partial dependencies"

Result - Animated transformation:
1. Table "splits" vertically into two tables (1.5s animation)
2. Customer columns slide out forming new table
3. Two tables emerge:

```
Orders Table (2NF):
| OrderID | OrderDate | CustomerID |
|---------|-----------|------------|
| 101     | 2024-01-15| 1001       |
| 102     | 2024-01-16| 1002       |
| 103     | 2024-01-16| 1001       |

Order_Items Table (2NF):
| OrderID | Product  | Quantity | Price |
|---------|----------|----------|-------|
| 101     | Mouse    | 2        | 25    |
| 101     | Keyboard | 1        | 80    |
| 102     | Monitor  | 1        | 300   |
| 103     | Mouse    | 5        | 25    |

Customers Table (2NF):
| CustomerID | CustomerName | CustomerAddress | ZIP   | City | State |
|------------|--------------|-----------------|-------|------|-------|
| 1001       | John Smith   | 123 Main St     | 90210 | LA   | CA    |
| 1002       | Jane Doe     | 456 Oak Ave     | 10001 | NYC  | NY    |
```

4. Foreign key CustomerID highlighted in blue with arrow to Customers table
5. Anomaly count: Update anomalies reduced (customer info no longer duplicated)
6. Explanation: "✓ Non-key attributes now depend on entire key. But look at Customers table..."

Students observe: Better, but Customers table still has redundancy (ZIP → City, State)

### Step 4: Transition to 3NF
Prompt: "Enable 'Show Dependencies' and click 'Show Next Step' to remove transitive dependencies"

Result - Animated transformation:
1. Dependency arrows appear in Customers table: ZIP → City, ZIP → State (transitive)
2. Customers table splits (1.5s animation):

```
Customers Table (3NF):
| CustomerID | CustomerName | CustomerAddress | ZIP   |
|------------|--------------|-----------------|-------|
| 1001       | John Smith   | 123 Main St     | 90210 |
| 1002       | Jane Doe     | 456 Oak Ave     | 10001 |

ZIP_Codes Table (3NF):
| ZIP   | City | State |
|-------|------|-------|
| 90210 | LA   | CA    |
| 10001 | NYC  | NY    |
```

3. Foreign key relationship drawn: Customers.ZIP → ZIP_Codes.ZIP
4. Anomaly count: 0 update anomalies, 0 insert anomalies, 0 delete anomalies
5. Explanation: "✓ Third Normal Form achieved! Each non-key attribute depends only on the primary key."

Students observe: Completely normalized, no redundancy, no anomalies

### Step 5: Challenge
Scenario presented: "A developer suggests denormalizing back to 2NF, keeping City and State in Customers table because 'it makes queries simpler.' What are the consequences?"

Expected learning:
- Students should identify:
  - Update anomaly returns: If NYC changes to "New York City", must update thousands of rows
  - Data integrity risk: Inconsistent city names (NYC vs New York vs Manhattan)
  - Storage waste: Repeated city/state strings
- Understand trade-off: Query simplicity vs data integrity
- Recognize when denormalization is acceptable (read-heavy OLAP) vs problematic (OLTP)

## Technical Specification
- **Library:** p5.js
- **Canvas:** Responsive, min 700px width × 600px height
- **Frame Rate:** 30fps
- **Data:** Static sample dataset with predefined transformations
  ```javascript
  const normalization_stages = {
    unnormalized: { tables: [...], anomalies: [...] },
    first_nf: { tables: [...], anomalies: [...], transformation: {...} },
    second_nf: { tables: [...], anomalies: [...], transformation: {...} },
    third_nf: { tables: [...], anomalies: [...], transformation: {...} }
  }
  ```

## Assessment Integration
After using this MicroSim, students should answer:

1. **Knowledge Check:** What does First Normal Form (1NF) require?
   - a) All non-key attributes depend on the primary key
   - b) Each cell contains atomic (indivisible) values ✓
   - c) No transitive dependencies exist
   - d) Foreign keys are properly defined

2. **Application:** You have a table where ZIP code determines City and State. The primary key is CustomerID. Which normal form is violated?
   - a) 1NF
   - b) 2NF
   - c) 3NF ✓ (transitive dependency: CustomerID → ZIP → City/State)
   - d) The table is fully normalized

3. **Scenario-Based:** A Student table has columns: StudentID (PK), Name, CourseID, CourseName, InstructorName. StudentID + CourseID form a composite key for enrollment. What's the problem?
   - a) Violates 1NF (multi-valued attributes)
   - b) Violates 2NF (CourseName and InstructorName depend only on CourseID, partial dependency) ✓
   - c) Violates 3NF (transitive dependency)
   - d) No problem, it's normalized

## Extension Ideas
- Add "Update Anomaly Simulator" showing what happens when you update denormalized data
- Include BCNF (Boyce-Codd Normal Form) as optional advanced step
- Add "Denormalization Decision" mode showing when to intentionally denormalize (data warehouses)
- Show storage calculation: bytes saved through normalization
- Include real-world examples from e-commerce, healthcare, finance
- Add interactive "Find the Dependency" game where students identify functional dependencies
- Show SQL DDL code generation for each normal form
- Include performance testing showing query differences between normalized/denormalized
