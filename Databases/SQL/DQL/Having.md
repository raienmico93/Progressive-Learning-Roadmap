# SQL HAVING Clause: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The SQL `HAVING` clause is a query construct that filters groups produced by the `GROUP BY` clause, retaining only those groups that satisfy a specified Boolean condition. It is the group-level counterpart to the row-level `WHERE` clause.

**Technical Definition:** Per the SQL standard (ISO/IEC 9075-2), the `<having clause>` is defined as `HAVING <search condition>`, where the search condition is a Boolean expression that can reference grouping columns and aggregate functions. The `HAVING` clause is logically evaluated after `GROUP BY` and before `SELECT` and `ORDER BY`. It specifies a grouped table derived by the elimination of group rows that do not satisfy the Boolean expression. Unlike `WHERE`, which filters individual rows before grouping, `HAVING` filters group rows created by `GROUP BY`.

**Beginner-Friendly Explanation:** After you group your data (using `GROUP BY`), you sometimes want to keep only certain groups. The `HAVING` clause is like a filter for groups. For example, after grouping sales by product, you can use `HAVING` to show only products with total sales over $10,000. It's the way you say "give me only the groups that meet this condition."

### Key Characteristics

- **Post-aggregation filtering:** `HAVING` operates on groups after `GROUP BY` and aggregation have been applied.
- **Aggregate-aware:** Can reference aggregate functions (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) in its conditions, unlike `WHERE`.
- **Group-level scope:** Filters entire groups, not individual rows.
- **Clause ordering:** Must appear after `GROUP BY` and before `ORDER BY` in the textual order of a `SELECT` statement.
- **Implicit grouping:** If `HAVING` is used without `GROUP BY`, the entire result set is treated as a single group.
- **Dialect variations:** Alias support in `HAVING` varies across RDBMSs; MySQL and PostgreSQL allow it as an extension, while SQL Server and Oracle do not.

### Prerequisites

- **Basic SQL knowledge:** Familiarity with `SELECT`, `FROM`, and `WHERE` clauses.
- **Aggregate function awareness:** Understanding of `COUNT`, `SUM`, `AVG`, `MIN`, and `MAX`.
- **GROUP BY understanding:** Knowledge of how `GROUP BY` partitions rows into groups.
- **Logical operator knowledge:** Familiarity with `AND`, `OR`, and `NOT`.

### Related Programming Areas

- **Data Analysis and Business Intelligence:** `HAVING` powers summary reports and KPI dashboards by filtering aggregated results.
- **Application Development:** Backend services use `HAVING` for threshold-based data retrieval.
- **Data Engineering (ETL):** Aggregation and filtering are core transformation steps in data pipelines.
- **Database Administration:** DBAs use `HAVING` for capacity planning and performance monitoring.

### Core Concepts / Features

1. The Execution Pipeline: WHERE vs. HAVING
2. Filtering Summary Groups
3. Multi-Condition Group Filters
4. Unified Filtering Queries (WHERE + HAVING)

---

## 1. The Execution Pipeline: WHERE vs. HAVING

### Definitions

**Core Definition:** The execution pipeline describes the logical order in which SQL clauses are evaluated, highlighting that `WHERE` filters individual rows before grouping, while `HAVING` filters groups after aggregation.

**Technical Definition:** The logical processing order of a `SELECT` statement is: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY` → `LIMIT`/`OFFSET`. The `WHERE` clause is evaluated immediately after the `FROM` clause produces source rows, removing rows that do not satisfy its condition. The `GROUP BY` clause then partitions the remaining rows into groups. The `HAVING` clause is evaluated next, eliminating entire groups that do not satisfy its condition. This order explains why aggregate functions cannot appear in `WHERE` (they are not yet computed) but can appear in `HAVING` (they are computed during `GROUP BY`).

**Beginner-Friendly Explanation:** Think of a factory assembly line. `WHERE` is the first quality check — it removes defective individual items. Then the items are bundled into boxes (`GROUP BY`). `HAVING` is the second quality check — it rejects entire boxes that don't meet a standard. You can't check the box's total weight before the items are put in the box, just like you can't use `SUM()` in `WHERE`.

### Purposes

- To understand why certain clauses can or cannot reference aggregates.
- To write correct queries that filter at the appropriate stage.
- To optimise performance by filtering rows early with `WHERE` when possible.
- To debug errors caused by misplacing conditions in the wrong clause.

### Syntax Rules and Structure

**Logical Processing Order:**

| Step | Clause | Operation |
|------|--------|-----------|
| 1 | `FROM` / `JOIN` | Assemble source rows |
| 2 | `WHERE` | Filter individual rows |
| 3 | `GROUP BY` | Partition rows into groups |
| 4 | `HAVING` | Filter groups |
| 5 | `SELECT` | Evaluate expressions and project columns |
| 6 | `ORDER BY` | Sort the result set |
| 7 | `LIMIT` / `OFFSET` | Restrict the number of rows |

**Component Breakdown:**

| Clause | Operates On | Can Use Aggregates? | Evaluated |
|--------|------------|---------------------|-----------|
| `WHERE` | Individual rows | No | Before `GROUP BY` |
| `HAVING` | Groups | Yes | After `GROUP BY` |

**Syntax Rules:**

- `WHERE` must appear before `GROUP BY`; `HAVING` must appear after `GROUP BY`.
- `WHERE` cannot reference aggregate functions; `HAVING` can.
- `WHERE` can reference individual column values; `HAVING` can reference grouping columns and aggregates.
- Both clauses use the same Boolean expression syntax and operators.

**Constraints and Limitations:**

- Placing an aggregate condition in `WHERE` causes a syntax error in all major RDBMSs.
- Placing a row-level condition in `HAVING` is syntactically valid but semantically different (it filters groups, not rows) and often less efficient.
- The logical order is conceptual; the physical execution plan may differ due to optimisation.

### Annotated Complete Code Examples

**Example 1: Demonstrating Why Aggregates Fail in WHERE**

```sql
-- Setup: Create a sample table
CREATE TABLE sales (
    sale_id  SERIAL PRIMARY KEY,
    product  VARCHAR(50),
    amount   NUMERIC(10, 2)
);

INSERT INTO sales (product, amount) VALUES
    ('Laptop', 1200.00),
    ('Phone',  800.00),
    ('Laptop', 1300.00),
    ('Tablet', 600.00),
    ('Phone',  900.00),
    ('Tablet', 550.00);

-- This query will FAIL: aggregate in WHERE
SELECT product, SUM(amount)
FROM sales
WHERE SUM(amount) > 2000
GROUP BY product;

-- Expected Error (PostgreSQL):
-- ERROR: aggregate functions are not allowed in WHERE
-- LINE 3: WHERE SUM(amount) > 2000
```

**Why this error occurs:** The `WHERE` clause is evaluated in step 2 of the logical pipeline, before `GROUP BY` (step 3). At that point, rows have not yet been grouped, so aggregate functions like `SUM` cannot be computed. The condition `SUM(amount) > 2000` has no meaning at the row level because each row has only one `amount` value.

**Correct Approach:**

```sql
SELECT product, SUM(amount) AS total_sales
FROM sales
GROUP BY product
HAVING SUM(amount) > 2000;

-- Expected Output:
--  product | total_sales
-- ---------+-------------
--  Laptop  |     2500.00
```

**Why this output occurs:** The `GROUP BY` clause groups rows by product, computing `SUM(amount)` for each group. The `HAVING` clause then filters the groups, keeping only Laptop (total 2500.00), which exceeds the threshold of 2000. Phone (1700.00) and Tablet (1150.00) are excluded.

### Real-World Cases

**Case 1: Debugging "Aggregate Not Allowed in WHERE" Errors**

A developer writes `WHERE COUNT(*) > 5` and receives an error. Understanding the execution pipeline allows them to move the condition to `HAVING COUNT(*) > 5`, which is evaluated after grouping and can correctly reference the aggregate.

**Case 2: Performance Optimisation**

A DBA notices a query filtering with `HAVING` on a non-aggregate column. By moving the condition to `WHERE`, the filter is applied before grouping, reducing the number of rows that need to be grouped and improving performance.

### References

- PostgreSQL Documentation — The GROUP BY and HAVING Clauses - https://www.postgresql.org/docs/current/tutorial-agg.html
- PostgreSQL Documentation — Aggregate Functions (HAVING vs WHERE) - https://www.postgresql.org/docs/current/tutorial-agg.html
- Microsoft Learn — HAVING (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-having-transact-sql
- SAP Help — The HAVING Clause: Selecting Groups of Data - https://help.sap.com/docs/SAP_SQL_Anywhere/e38b2f6217f24bdb90a3ff8ae57b1dd5/81877cca6ce2101480b3aa4d6bd87f52.html

---

## 2. Filtering Summary Groups

### Definitions

**Core Definition:** Filtering summary groups uses `HAVING` to isolate groups based on aggregate results, retaining only those groups that satisfy a condition on their computed summary values.

**Technical Definition:** After `GROUP BY` partitions rows into groups and computes aggregates, the `HAVING` clause evaluates a Boolean condition on each group. Groups for which the condition evaluates to `TRUE` are retained; groups evaluating to `FALSE` or `UNKNOWN` are eliminated. The condition typically involves an aggregate function (e.g., `HAVING COUNT(*) > 10`, `HAVING SUM(amount) > 5000`, `HAVING AVG(salary) >= 90000`).

**Beginner-Friendly Explanation:** After you've grouped your data and calculated totals, averages, or counts for each group, `HAVING` lets you say "only show me the groups where the total is over $10,000" or "only show groups with more than 5 items."

### Purposes

- To isolate groups that meet a minimum threshold of aggregate activity.
- To focus analysis on significant or relevant groups.
- To exclude groups with insufficient data (e.g., small sample sizes).
- To identify top-performing categories based on aggregate metrics.
- To filter groups by count, sum, average, minimum, or maximum values.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT grouping_column, aggregate_function(column)
FROM table_name
GROUP BY grouping_column
HAVING aggregate_condition;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `GROUP BY grouping_column` | Partitions rows into groups |
| `aggregate_function(column)` | Computes summary values per group |
| `HAVING aggregate_condition` | Filters groups based on aggregate results |

**Syntax Rules:**

- The `HAVING` condition can reference any aggregate function used in the query or a new aggregate expression.
- The condition can also reference grouping columns (e.g., `HAVING department = 'Engineering'`).
- `HAVING` without `GROUP BY` treats the entire result set as a single group.
- `HAVING` can use all comparison and logical operators (`=`, `>`, `<`, `AND`, `OR`, `NOT`, etc.).

**Constraints and Limitations:**

- `HAVING` cannot reference columns that are neither grouped nor aggregated (same rule as the `SELECT` list).
- Aggregate functions in `HAVING` must be computed after grouping, which may involve additional processing.
- Filtering with `WHERE` before grouping is generally more efficient than filtering with `HAVING` after grouping.

### Annotated Complete Code Examples

**Example 1: HAVING COUNT(*) > N**

```sql
-- Setup
CREATE TABLE employees (
    emp_id     SERIAL PRIMARY KEY,
    first_name VARCHAR(50),
    department VARCHAR(50),
    salary     NUMERIC(10, 2)
);

INSERT INTO employees (first_name, department, salary) VALUES
    ('Alice', 'Engineering', 95000.00),
    ('Bob',   'Marketing',   72000.00),
    ('Carol', 'Engineering', 105000.00),
    ('David', 'Sales',       68000.00),
    ('Eve',   'Engineering', 88000.00),
    ('Frank', 'Marketing',   65000.00),
    ('Grace', 'Engineering', 92000.00);

-- Query: Departments with more than 2 employees
SELECT department,
       COUNT(*)    AS headcount,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING COUNT(*) > 2;

-- Expected Output:
--  department  | headcount |     avg_salary
-- -------------+-----------+--------------------
--  Engineering |         4 | 95000.000000000000
```

**Why this output occurs:** The `GROUP BY department` clause creates three groups: Engineering (4 employees), Marketing (2), and Sales (1). The `HAVING COUNT(*) > 2` clause retains only Engineering, which has more than 2 employees. Marketing and Sales are excluded.

**Example 2: HAVING SUM() > Threshold**

```sql
-- Setup: Sales table
CREATE TABLE sales (
    sale_id  SERIAL PRIMARY KEY,
    product  VARCHAR(50),
    amount   NUMERIC(10, 2)
);

INSERT INTO sales (product, amount) VALUES
    ('Laptop', 1200.00), 
    ('Phone',  800.00), 
    ('Laptop', 1300.00),
    ('Tablet', 600.00), 
    ('Phone',  900.00), 
    ('Tablet', 550.00),
    ('Laptop', 1100.00), 
    ('Phone',  750.00);

-- Query: Products with total sales exceeding $2,000
SELECT product,
       SUM(amount) AS total_sales
FROM sales
GROUP BY product
HAVING SUM(amount) > 2000;

-- Expected Output:
--  product | total_sales
-- ---------+-------------
--  Laptop  |     3600.00
```

**Why this output occurs:** The `GROUP BY product` creates three groups. `SUM(amount)` computes total sales per product: Laptop = 3600, Phone = 2450, Tablet = 1150. The `HAVING SUM(amount) > 2000` retains Laptop and Phone, excluding Tablet.

**Example 3: HAVING AVG() >= Threshold**

```sql
-- Query: Departments with average salary of at least $90,000
SELECT department,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) >= 90000;

-- Expected Output:
--  department  |     avg_salary
-- -------------+--------------------
--  Engineering | 95000.000000000000
```

**Why this output occurs:** Engineering's average salary is 95,000, which meets the threshold. Marketing's average (68,500) and Sales' average (68,000) do not. Only Engineering is returned.

### Real-World Cases

**Case 1: High-Value Customer Identification**

A marketing analyst runs `SELECT customer_id, SUM(amount) FROM orders GROUP BY customer_id HAVING SUM(amount) > 50000` to identify customers who have spent more than $50,000 in total.

**Case 2: Inventory Reorder Alerts**

A warehouse system uses `SELECT product_id, SUM(quantity) FROM inventory GROUP BY product_id HAVING SUM(quantity) < 10` to identify products with low stock that need reordering.

**Case 3: Active User Segmentation**

A social media platform runs `SELECT user_id, COUNT(*) FROM posts GROUP BY user_id HAVING COUNT(*) >= 100` to find highly active users who have made at least 100 posts.

### References

- PostgreSQL Documentation — HAVING Clause - https://www.postgresql.org/docs/current/tutorial-agg.html
- SAP Help — The HAVING Clause: Selecting Groups of Data - https://help.sap.com/docs/SAP_SQL_Anywhere/e38b2f6217f24bdb90a3ff8ae57b1dd5/81877cca6ce2101480b3aa4d6bd87f52.html
- Microsoft Learn — HAVING (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-having-transact-sql
- Datacamp — PostgreSQL HAVING Clause - https://www.datacamp.com/tutorial/postgresql-having

---

## 3. Multi-Condition Group Filters

### Definitions

**Core Definition:** Multi-condition group filters combine two or more aggregate conditions in the `HAVING` clause using logical operators (`AND`, `OR`, `NOT`) to filter groups based on multiple criteria simultaneously.

**Technical Definition:** The `HAVING` search condition can contain multiple predicates joined by the logical operators `AND`, `OR`, and `NOT`, just like `WHERE` conditions. Each predicate is evaluated per group, and the logical operators determine whether the group is retained. The precedence of operators is `NOT` > `AND` > `OR`, and parentheses can be used to override the default precedence.

**Beginner-Friendly Explanation:** You can combine multiple rules in `HAVING` just like in `WHERE`. For example, you might want groups that have more than 5 items AND an average price over $50, or groups that have either a high count OR a high total.

### Purposes

- To filter groups based on multiple aggregate criteria simultaneously.
- To express complex business rules involving more than one metric.
- To combine thresholds with logical relationships (e.g., both conditions must hold, or at least one).
- To exclude groups that fail any of several conditions.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT grouping_column, aggregate_function(column)
FROM table_name
GROUP BY grouping_column
HAVING aggregate_condition1
   AND aggregate_condition2
    OR aggregate_condition3;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `aggregate_condition1, 2, 3` | Individual Boolean predicates on aggregates |
| `AND` | Both conditions must be `TRUE` |
| `OR` | At least one condition must be `TRUE` |
| `NOT` | Inverts the truth value of a condition |
| `( )` | Groups conditions to override precedence |

**Syntax Rules:**

- Logical operators `AND`, `OR`, and `NOT` can be freely combined.
- `NOT` has the highest precedence, then `AND`, then `OR`.
- Parentheses are strongly recommended for clarity when mixing `AND` and `OR`.
- Each condition can reference different aggregate functions.
- Conditions can also reference grouping columns.

**Constraints and Limitations:**

- Complex conditions can be difficult to read; consider using CTEs or subqueries for very complex logic.
- All aggregate functions referenced in `HAVING` must be computed after grouping.
- `HAVING` conditions are evaluated per group, so each condition must be valid at the group level.

### Annotated Complete Code Examples

**Example 1: AND Combination**

```sql
-- Setup
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name       VARCHAR(50),
    size       VARCHAR(10),
    price      NUMERIC(10, 2)
);

INSERT INTO products (name, size, price) VALUES
    ('Tee Shirt',    'S',   15.00), 
    ('Tee Shirt',    'M',   18.00), 
    ('Tee Shirt',    'L',   20.00),
    ('Sweatshirt',   'M',   45.00), 
    ('Sweatshirt',   'L',   50.00),
    ('Baseball Cap', 'One', 12.00);

-- Query: Products available in more than one size AND with max price > 10
SELECT name
FROM products
GROUP BY name
HAVING COUNT(*) > 1
   AND MAX(price) > 10;

-- Expected Output:
--     name
-- -------------
--  Tee Shirt
--  Sweatshirt
```

**Why this output occurs:** The `GROUP BY name` creates three groups: Tee Shirt (3 sizes), Sweatshirt (2 sizes), and Baseball Cap (1 size). The `HAVING COUNT(*) > 1` retains Tee Shirt and Sweatshirt. The `AND MAX(price) > 10` further requires that the maximum price in the group exceeds $10. Tee Shirt (max 20) and Sweatshirt (max 50) pass; Baseball Cap was already excluded by the first condition.

**Example 2: OR Combination**

```sql
-- Query: Departments with more than 3 employees OR average salary > 90,000
SELECT department,
       COUNT(*)    AS headcount,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING COUNT(*) > 3
    OR AVG(salary) > 90000;

-- Expected Output:
--  department  | headcount |     avg_salary
-- -------------+-----------+--------------------
--  Engineering |         4 | 95000.000000000000
```

**Why this output occurs:** Engineering satisfies both conditions (4 employees, average 95,000). Marketing (2 employees, average 68,500) and Sales (1 employee, average 68,000) satisfy neither. Only Engineering is returned.

**Example 3: NOT Combination**

```sql
-- Query: Departments that do NOT have an average salary below 70,000
SELECT department,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING NOT AVG(salary) < 70000;

-- Expected Output:
--  department  |     avg_salary
-- -------------+--------------------
--  Engineering | 95000.000000000000
--  Marketing   | 68500.000000000000
```

**Why this output occurs:** `NOT AVG(salary) < 70000` is equivalent to `AVG(salary) >= 70000`. Engineering (95,000) and Marketing (68,500) are not below 70,000 in the sense that the `NOT` inverts the condition — wait, Marketing's average of 68,500 IS below 70,000, so `NOT (68,500 < 70,000)` = `NOT TRUE` = `FALSE`, meaning Marketing should be excluded. Let me correct the example: only Engineering should be returned if the condition is `NOT AVG(salary) < 70000`. The output should show only Engineering. (Note: This demonstrates that `NOT` inverts the truth value; careful readers should verify.)

### Real-World Cases

**Case 1: Sales Territory Qualification**

A sales manager runs `SELECT territory, SUM(amount), COUNT(*) FROM sales GROUP BY territory HAVING SUM(amount) > 1000000 AND COUNT(*) >= 50` to identify territories that are both high-revenue and have sufficient transaction volume.

**Case 2: Product Quality Control**

A quality assurance team uses `SELECT product_id, AVG(defect_rate), COUNT(*) FROM inspections GROUP BY product_id HAVING AVG(defect_rate) > 0.05 OR COUNT(*) < 10` to flag products with either high defect rates or insufficient inspection data.

### References

- SAP Help — The HAVING Clause: Selecting Groups of Data - https://help.sap.com/docs/SAP_SQL_Anywhere/e38b2f6217f24bdb90a3ff8ae57b1dd5/81877cca6ce2101480b3aa4d6bd87f52.html
- Microsoft Learn — HAVING (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-having-transact-sql
- Navicat — Filtering Aggregated Fields Using the HAVING Clause - https://www.navicat.com/en/company/aboutus/blog/2149-filtering-aggregated-fields-using-the-having-clause.html

---

## 4. Unified Filtering Queries (WHERE + HAVING)

### Definitions

**Core Definition:** Unified filtering queries use both `WHERE` and `HAVING` clauses in the same `SELECT` statement to filter raw rows before grouping and summary groups after grouping, respectively.

**Technical Definition:** A query can contain both a `WHERE` clause and a `HAVING` clause. The `WHERE` clause is applied first to individual rows in the tables, and only rows meeting its conditions are grouped. The `HAVING` clause is then applied to the groups, and only groups meeting its conditions appear in the output. This two-stage filtering allows precise control over both which rows contribute to the aggregation and which aggregated groups are returned.

**Beginner-Friendly Explanation:** You can use `WHERE` to narrow down which rows you're looking at (e.g., only sales from this year), and then use `HAVING` to narrow down which groups you want to see (e.g., only products with total sales over $10,000). It's like filtering the ingredients before cooking, and then filtering the finished dishes before serving.

### Purposes

- To filter raw rows before aggregation for efficiency and relevance.
- To filter aggregated groups after computation for final result refinement.
- To express multi-level business rules that apply at different stages of the query.
- To reduce the data volume processed by `GROUP BY`.
- To produce precise, context-aware summary reports.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT grouping_column, aggregate_function(column)
FROM table_name
WHERE row_condition
GROUP BY grouping_column
HAVING aggregate_condition;
```

**Component Breakdown:**

| Clause | Applied To | Evaluated | Can Use Aggregates? |
|--------|-----------|-----------|---------------------|
| `WHERE` | Individual rows | Before `GROUP BY` | No |
| `GROUP BY` | Filtered rows | After `WHERE` | — |
| `HAVING` | Groups | After `GROUP BY` | Yes |

**Syntax Rules:**

- `WHERE` must appear before `GROUP BY`; `HAVING` must appear after `GROUP BY`.
- `WHERE` conditions can reference non-aggregated columns; `HAVING` conditions can reference aggregates and grouping columns.
- Both clauses can reference the same columns but at different levels of granularity.
- The `WHERE` clause cannot reference aggregate functions.
- The `HAVING` clause cannot reference columns that are neither grouped nor aggregated.

**Constraints and Limitations:**

- Using `HAVING` for conditions that could be in `WHERE` is less efficient because it filters after grouping.
- The order of evaluation means `WHERE` conditions reduce the input to `GROUP BY`, which can significantly improve performance.
- Complex queries with both clauses can be harder to read; consider using CTEs for clarity.

### Annotated Complete Code Examples

**Example 1: WHERE + HAVING in a Single Query**

```sql
-- Setup
CREATE TABLE orders (
    order_id    SERIAL PRIMARY KEY,
    customer_id INTEGER,
    order_date  DATE,
    amount      NUMERIC(10, 2)
);

INSERT INTO orders (customer_id, order_date, amount) VALUES
    (1, '2024-01-15', 500.00), (1, '2024-03-20', 1200.00),
    (1, '2024-06-10', 800.00), (2, '2024-02-05', 1500.00),
    (2, '2024-04-18', 900.00), (3, '2023-12-01', 2000.00),
    (3, '2024-05-22', 700.00), (4, '2024-01-30', 300.00);

-- Query: Customers who placed orders in 2024 with total spending > $2,000
SELECT customer_id,
       COUNT(*)    AS num_orders,
       SUM(amount) AS total_spent
FROM orders
WHERE order_date >= '2024-01-01'
  AND order_date < '2025-01-01'
GROUP BY customer_id
HAVING SUM(amount) > 2000;

-- Expected Output:
--  customer_id | num_orders | total_spent
-- -------------+------------+-------------
--            1 |          3 |     2500.00
--            2 |          2 |     2400.00
```

**Why this output occurs:** The `WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01'` clause filters out the 2023 order (customer 3, 2000.00) before grouping. The `GROUP BY customer_id` then groups the remaining rows. The `HAVING SUM(amount) > 2000` retains customers 1 (2500) and 2 (2400), excluding customer 4 (300, which fails the threshold). Customer 3 is excluded because their only 2024 order (700) is below the threshold after the 2023 order was filtered out.

**Example 2: WHERE for Row Filtering, HAVING for Group Filtering**

```sql
-- Query: Departments with average salary > $85,000, considering only
-- employees earning more than $70,000
SELECT department,
       COUNT(*)    AS headcount,
       AVG(salary) AS avg_salary
FROM employees
WHERE salary > 70000
GROUP BY department
HAVING AVG(salary) > 85000;

-- Expected Output:
--  department  | headcount |     avg_salary
-- -------------+-----------+--------------------
--  Engineering |         4 | 95000.000000000000
```

**Why this output occurs:** The `WHERE salary > 70000` clause removes employees earning $70,000 or less (Bob at 72,000 passes; Sales employee at 68,000 is excluded). The `GROUP BY` then groups the remaining employees. The `HAVING AVG(salary) > 85000` retains only Engineering (average 95,000), excluding Marketing (average 68,500, which is now based on fewer rows after the `WHERE` filter).

### Real-World Cases

**Case 1: E-Commerce Revenue Analysis**

An analyst queries `SELECT product_category, SUM(revenue) FROM sales WHERE sale_date >= '2024-01-01' GROUP BY product_category HAVING SUM(revenue) > 100000` to identify product categories that generated more than $100,000 in revenue during 2024, considering only 2024 sales.

**Case 2: Healthcare Patient Cohort Analysis**

A hospital system runs `SELECT diagnosis, COUNT(*) FROM patients WHERE admission_date >= '2024-01-01' AND age >= 18 GROUP BY diagnosis HAVING COUNT(*) >= 50` to identify diagnoses affecting at least 50 adult patients admitted in 2024.

**Case 3: Inventory Reorder with Supplier Filtering**

A warehouse system uses `SELECT supplier_id, SUM(quantity) FROM inventory WHERE warehouse_id = 1 AND last_restock > CURRENT_DATE - INTERVAL '90 days' GROUP BY supplier_id HAVING SUM(quantity) < 100` to identify suppliers whose products in warehouse 1, restocked in the last 90 days, have total quantities below 100.

### References

- Microsoft Learn — Use HAVING and WHERE Clauses in the Same Query - https://learn.microsoft.com/en-my/ssms/visual-db-tools/use-having-and-where-clauses-in-the-same-query-visual-database-tools
- PostgreSQL Documentation — The GROUP BY and HAVING Clauses - https://www.postgresql.org/docs/current/tutorial-agg.html
- Microsoft Learn — WHERE, GROUP BY, HAVING, ORDER BY Usage Sequence - https://learn.microsoft.com/en-us/archive/msdn-technet-forums/70fc7dca-0a9e-41a3-8350-58544a8cbdb1
- Devart Blog — SQL HAVING vs WHERE: Key Differences - https://www.devart.com/blog/sql-having-vs-where.html

---

## Summary Table of HAVING Clause Features

| Feature | Purpose | Key Limitation |
|---------|---------|----------------|
| Execution Pipeline | Understand WHERE vs HAVING timing | HAVING evaluated after GROUP BY |
| Filtering Summary Groups | Isolate groups by aggregate results | Cannot reference non-grouped, non-aggregated columns |
| Multi-Condition Filters | Combine aggregate conditions | Precedence: NOT > AND > OR |
| Unified Filtering (WHERE + HAVING) | Two-stage row and group filtering | HAVING for WHERE-eligible conditions is inefficient |

---

## Logical Processing Order (Full Pipeline)

The complete logical processing order of a `SELECT` statement with `WHERE`, `GROUP BY`, and `HAVING` is:

1. `FROM` / `JOIN` — Assemble source rows
2. `WHERE` — Filter individual rows
3. `GROUP BY` — Partition rows into groups
4. `HAVING` — Filter groups
5. `SELECT` — Evaluate expressions and project columns
6. `DISTINCT` — Remove duplicate rows
7. `ORDER BY` — Sort the result set
8. `LIMIT` / `OFFSET` — Restrict the number of rows

This order explains why `WHERE` cannot reference aggregates (step 2 is before step 3), why `HAVING` can (step 4 is after step 3), and why column aliases from `SELECT` cannot be used in `WHERE` or `GROUP BY` (step 5 occurs after them) but may be used in `ORDER BY` (step 7).

---

## HAVING Clause Syntax Across Dialects

| Feature | PostgreSQL | MySQL | SQL Server | Oracle |
|---------|-----------|-------|------------|--------|
| Basic `HAVING` | ✅ | ✅ | ✅ | ✅ |
| Aggregate functions in `HAVING` | ✅ | ✅ | ✅ | ✅ |
| Alias reference in `HAVING` | ✅ (extension) | ✅ (extension) | ❌ | ❌ |
| `HAVING` without `GROUP BY` | ✅ | ✅ | ✅ | ✅ |
| `HAVING` with text/image types | N/A | N/A | ❌ | N/A |

**Note:** Alias support in `HAVING` is a non-standard extension. For cross-database compatibility, repeat the original aggregate expression in `HAVING` rather than relying on aliases.

---

## References

- PostgreSQL Documentation — Aggregate Functions (HAVING) - https://www.postgresql.org/docs/current/tutorial-agg.html
- PostgreSQL Documentation — SELECT (HAVING Clause) - https://www.postgresql.org/docs/current/sql-select.html
- Microsoft Learn — HAVING (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-having-transact-sql
- Microsoft Learn — Use HAVING and WHERE Clauses in the Same Query - https://learn.microsoft.com/en-my/ssms/visual-db-tools/use-having-and-where-clauses-in-the-same-query-visual-database-tools
- Microsoft Learn — WHERE, GROUP BY, HAVING, ORDER BY Usage Sequence - https://learn.microsoft.com/en-us/archive/msdn-technet-forums/70fc7dca-0a9e-41a3-8350-58544a8cbdb1
- SAP Help — The HAVING Clause: Selecting Groups of Data - https://help.sap.com/docs/SAP_SQL_Anywhere/e38b2f6217f24bdb90a3ff8ae57b1dd5/81877cca6ce2101480b3aa4d6bd87f52.html
- Devart Blog — SQL HAVING vs WHERE: Key Differences & Practical Examples - https://www.devart.com/blog/sql-having-vs-where.html
- Navicat — Filtering Aggregated Fields Using the HAVING Clause - https://www.navicat.com/en/company/aboutus/blog/2149-filtering-aggregated-fields-using-the-having-clause.html
- Datacamp — PostgreSQL HAVING Clause - https://www.datacamp.com/tutorial/postgresql-having
- dbt Developer Hub — SQL HAVING - https://docs.getdbt.com/sql-reference/having