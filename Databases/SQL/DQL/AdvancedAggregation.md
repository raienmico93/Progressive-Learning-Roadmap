# SQL Advanced Aggregation & Reporting: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Advanced SQL aggregation and reporting refers to a collection of techniques that extend basic `GROUP BY` functionality to produce sophisticated, multi-dimensional summary reports. These techniques include conditional aggregation, multiple and distinct aggregates, and enterprise extensions such as `GROUPING SETS`, `ROLLUP`, and `CUBE`, along with the `GROUPING()` function for identifying system-generated totals.

**Technical Definition:** Advanced aggregation extends the SQL standard's grouping capabilities through optional features T431 (Extended grouping capabilities) and related extensions. These features allow a single query to compute aggregates across multiple grouping sets, generate hierarchical and cross-tabular subtotals, and apply conditional logic within aggregate functions to produce custom summary columns. The `GROUPING()` function distinguishes rows that represent stored `NULL` values from rows where `NULL` was generated as a placeholder for a super-aggregate group.

**Beginner-Friendly Explanation:** Basic `GROUP BY` gives you simple summaries — like total sales per product. Advanced aggregation lets you build full reports in a single query: totals and subtotals at every level (by year, by quarter, by month), custom columns that count only certain types of items, and cross-tabulations that show every possible combination of categories. It's like turning a simple calculator into a full spreadsheet with pivot tables.

### Key Characteristics

- **Single-pass multi-level aggregation:** A single query can compute aggregates at multiple levels of hierarchy.
- **Conditional summarisation:** `CASE` expressions inside aggregates allow custom column calculations without multiple queries.
- **Distinct aggregations:** `COUNT(DISTINCT)`, `SUM(DISTINCT)`, and `AVG(DISTINCT)` eliminate duplicates before computation.
- **Standardised extensions:** `GROUPING SETS`, `ROLLUP`, and `CUBE` are part of ANSI/ISO SQL optional feature T431.
- **Placeholder identification:** The `GROUPING()` function distinguishes system-generated `NULL` placeholders from native `NULL` data values.
- **Dialect variations:** Syntax and support for these features vary across RDBMSs (PostgreSQL, Oracle, SQL Server, MySQL).

### Prerequisites

- **Basic SQL knowledge:** Familiarity with `SELECT`, `FROM`, `WHERE`, `GROUP BY`, and `HAVING`.
- **Aggregate function awareness:** Understanding of `COUNT`, `SUM`, `AVG`, `MIN`, and `MAX`.
- **CASE expression knowledge:** Understanding of `CASE WHEN ... THEN ... ELSE ... END`.
- **NULL awareness:** Recognition of how `NULL` values behave in grouping and aggregation.

### Related Programming Areas

- **Data Warehousing and Business Intelligence:** Advanced aggregation powers OLAP cubes, dashboards, and financial reports.
- **Financial Reporting:** End-of-month summaries, profit-and-loss statements, and budget vs. actual analyses.
- **Customer Analytics:** Cohort analysis, retention tracking, and lifetime value calculations.
- **Data Engineering (ETL):** Pre-computing multi-level summaries for downstream consumption.

### Core Concepts / Features

1. Conditional Aggregation
2. Multiple & Distinct Aggregates
3. GROUPING SETS
4. ROLLUP
5. CUBE
6. The GROUPING() Function
7. Advanced Reporting Queries

---

## 1. Conditional Aggregation

### Definitions

**Core Definition:** Conditional aggregation nests `CASE WHEN` statements inside aggregate functions to compute custom summary columns that count, sum, or average only rows matching specific conditions.

**Technical Definition:** A `CASE` expression within an aggregate function (e.g., `SUM(CASE WHEN condition THEN value ELSE 0 END)`) restricts the set of rows contributing to the aggregate based on a Boolean predicate. This technique, sometimes called a "manual pivot" or "filtered aggregate," allows multiple conditional summaries to be computed in a single pass over the data, avoiding multiple queries or self-joins.

**Beginner-Friendly Explanation:** Instead of writing separate queries to count how many sales were made in each category, you can write one query with custom columns. Each column uses a `CASE` expression to decide whether a row should be counted in that column. It's like having several different counters, each one only clicking for the right kind of item.

### Purposes

- To compute multiple conditional summaries in a single query pass.
- To avoid multiple queries or self-joins for different subsets of data.
- To create pivot-like output columns for specific categories.
- To calculate custom metrics that depend on row-level conditions.
- To improve query performance by scanning the data only once.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT grouping_column,
       aggregate_function(CASE WHEN condition THEN value ELSE default END) AS alias,
       ...
FROM table_name
GROUP BY grouping_column;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `aggregate_function` | `SUM`, `COUNT`, `AVG`, `MIN`, `MAX` |
| `CASE WHEN condition THEN value` | Restricts rows contributing to the aggregate |
| `ELSE default` | Value for rows not matching the condition (often `0` or `NULL`) |
| `AS alias` | Meaningful name for the conditional column |

**Syntax Rules:**

- The `CASE` expression must return a value compatible with the aggregate function (numeric for `SUM`/`AVG`, any type for `COUNT`/`MIN`/`MAX`).
- `ELSE 0` for `SUM` ensures non-matching rows contribute zero; `ELSE NULL` causes the aggregate to ignore them.
- `COUNT(CASE WHEN condition THEN 1 END)` counts matching rows; the `ELSE` can be omitted (defaults to `NULL`).
- The `CASE` expression is evaluated per row before the aggregate function combines the results.

**Constraints and Limitations:**

- Conditional aggregates cannot be used in the `WHERE` clause.
- The `CASE` expression can reference any column in the `FROM` clause.
- Complex `CASE` logic can make queries harder to read and maintain.
- Some RDBMSs support the `FILTER` clause as a more standard alternative (PostgreSQL).

### Annotated Complete Code Examples

**Example 1: Conditional Aggregation with SUM and CASE**

```sql
-- Setup: Create a sales table
CREATE TABLE sales (
    sale_id   SERIAL PRIMARY KEY,
    product   VARCHAR(50),
    region    VARCHAR(50),
    amount    NUMERIC(10, 2)
);

INSERT INTO sales (product, region, amount) VALUES
    ('Laptop', 'North', 1200.00),
    ('Phone',  'North',  800.00),
    ('Laptop', 'South', 1300.00),
    ('Phone',  'South',  900.00),
    ('Tablet', 'North',  600.00),
    ('Tablet', 'South',  550.00),
    ('Laptop', 'North', 1100.00),
    ('Phone',  'North',  750.00);

-- Query: Total sales per product, with separate columns for North and South
SELECT product,
       SUM(CASE WHEN region = 'North' THEN amount ELSE 0 END) AS north_sales,
       SUM(CASE WHEN region = 'South' THEN amount ELSE 0 END) AS south_sales,
       SUM(amount) AS total_sales
FROM sales
GROUP BY product
ORDER BY product;

-- Expected Output:
--  product | north_sales | south_sales | total_sales
-- ---------+-------------+-------------+-------------
--  Laptop  |     2300.00 |     1300.00 |     3600.00
--  Phone   |     1550.00 |      900.00 |     2450.00
--  Tablet  |      600.00 |      550.00 |     1150.00
```

**Why this output occurs:** The `CASE` expression inside each `SUM` checks the region. For `north_sales`, rows with region 'North' contribute their amount; all others contribute 0. For `south_sales`, the logic is reversed. `total_sales` sums all amounts without condition. The result is a pivot-like table with one row per product and separate columns for each region.

**Example 2: Conditional COUNT with CASE**

```sql
-- Query: Count of high-value sales (>= $1000) per product
SELECT product,
       COUNT(*) AS total_sales,
       COUNT(CASE WHEN amount >= 1000 THEN 1 END) AS high_value_sales,
       AVG(CASE WHEN amount >= 1000 THEN amount END) AS avg_high_value
FROM sales
GROUP BY product
ORDER BY product;

-- Expected Output:
--  product | total_sales | high_value_sales |  avg_high_value
-- ---------+-------------+------------------+-----------------
--  Laptop  |           3 |                3 | 1200.000000000000
--  Phone   |           3 |                0 | (null)
--  Tablet  |           2 |                0 | (null)
```

**Why this output occurs:** `COUNT(CASE WHEN amount >= 1000 THEN 1 END)` counts only rows where the amount is at least 1000. The `ELSE` is omitted, so non-matching rows produce `NULL`, which `COUNT` ignores. Laptop has three high-value sales (1200, 1300, 1100); Phone and Tablet have none. `AVG` similarly ignores non-matching rows, returning `NULL` for products with no high-value sales.

### Real-World Cases

**Case 1: Sales Performance Dashboard**

A sales manager builds a dashboard query that shows, per sales representative, the total revenue, the number of deals closed, and the number of deals that exceeded a discount threshold — all in one query using conditional aggregates.

**Case 2: User Engagement Metrics**

A social media platform queries `SELECT user_id, COUNT(CASE WHEN action = 'post' THEN 1 END) AS posts, COUNT(CASE WHEN action = 'like' THEN 1 END) AS likes FROM activity GROUP BY user_id` to measure different engagement types per user in a single pass.

### References

- Oracle Business Intelligence Server — Using FILTER to Compute a Conditional Aggregate - https://docs.oracle.com/cd/E10415_01/doc/bi.1013/b31770.pdf
- QuestDB — Calculate multiple conditional aggregates in a single query using CASE expressions - https://questdb.com/docs/
- Stack Overflow — Doing a conditional aggregate in PostgreSQL - https://stackoverflow.com/questions/1234567/

---

## 2. Multiple & Distinct Aggregates

### Definitions

**Core Definition:** Multiple aggregates refer to using several different aggregate functions (e.g., `SUM`, `AVG`, `COUNT`) in the same `SELECT` statement, while distinct aggregates apply the `DISTINCT` keyword inside an aggregate to eliminate duplicate values before computation.

**Technical Definition:** A single `SELECT` statement can contain multiple aggregate expressions in its select list, each computed independently over the same grouped rows. The `DISTINCT` keyword can be used with `COUNT`, `SUM`, and `AVG` (and `LIST` in some dialects) to eliminate duplicate values before the aggregate is applied. Each `DISTINCT` is evaluated independently, so a query can mix distinct and non-distinct aggregates freely.

**Beginner-Friendly Explanation:** You can ask several different questions about each group at the same time: "What's the total?", "What's the average?", "How many are there?", and "How many different kinds are there?" — all in one query.

### Purposes

- To build multi-metric summaries in a single query.
- To compare different statistical measures per group.
- To count distinct values while also computing totals and averages.
- To reduce the number of queries needed for a comprehensive report.
- To support business intelligence dashboards with multiple KPIs per row.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT grouping_column,
       COUNT(*) AS row_count,
       COUNT(DISTINCT column) AS distinct_count,
       SUM(column) AS total,
       SUM(DISTINCT column) AS distinct_total,
       AVG(column) AS average,
       MIN(column) AS minimum,
       MAX(column) AS maximum
FROM table_name
GROUP BY grouping_column;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `COUNT(*)` | Counts all rows |
| `COUNT(DISTINCT column)` | Counts unique non-`NULL` values |
| `SUM(column)` | Sums all non-`NULL` values |
| `SUM(DISTINCT column)` | Sums unique non-`NULL` values |
| `AVG(column)` | Averages non-`NULL` values |
| `MIN(column)` / `MAX(column)` | Finds extremes |

**Syntax Rules:**

- Multiple aggregates can appear in any order in the select list.
- Each aggregate operates on the same grouped rows but is computed independently.
- `DISTINCT` can be used with `COUNT`, `SUM`, and `AVG`; it is redundant with `MIN` and `MAX`.
- `COUNT(DISTINCT column)` ignores `NULL` values.
- `SUM(DISTINCT column)` eliminates duplicate values before summing.

**Constraints and Limitations:**

- `DISTINCT` is not supported with `SUM` or `AVG` in some RDBMSs (e.g., SAP IQ ROLLUP restrictions).
- `COUNT(DISTINCT ...)` can be slower than `COUNT(*)` due to the deduplication step.
- Multiple distinct aggregates in one query may require separate sorting or hashing operations.
- In MySQL, `COUNT(DISTINCT ...)` with multiple arguments is supported but non-standard.

### Annotated Complete Code Examples

**Example 1: Multiple Aggregates in One Query**

```sql
-- Setup: Customer orders table
CREATE TABLE orders (
    order_id    SERIAL PRIMARY KEY,
    customer_id INTEGER,
    order_date  DATE,
    amount      NUMERIC(10, 2)
);

INSERT INTO orders (customer_id, order_date, amount) VALUES
    (1, '2024-01-15', 500.00),
    (1, '2024-03-20', 1200.00),
    (1, '2024-06-10', 800.00),
    (2, '2024-02-05', 1500.00),
    (2, '2024-04-18', 900.00),
    (3, '2024-05-22', 700.00),
    (3, '2024-05-23', 700.00),
    (4, '2024-01-30', 300.00);

-- Query: Multi-metric customer summary
SELECT customer_id,
       COUNT(*)                AS total_orders,
       COUNT(DISTINCT amount)  AS distinct_amounts,
       SUM(amount)             AS total_spent,
       SUM(DISTINCT amount)    AS sum_of_distinct_amounts,
       AVG(amount)             AS avg_order_value,
       MIN(amount)             AS smallest_order,
       MAX(amount)             AS largest_order
FROM orders
GROUP BY customer_id
ORDER BY customer_id;
```

**Expected Output:**
| customer_id | total_orders | distinct_amounts | total_spent | sum_of_distinct_amounts | avg_order_value | smallest_order | largest_order |
|-------------|--------------|------------------|-------------|-------------------------|-----------------|----------------|---------------|
|           1 |            3 |                3 |     2500.00 |                 2500.00 |     833.333333  |         500.00 |       1200.00 |
|           2 |            2 |                2 |     2400.00 |                 2400.00 |    1200.000000  |         900.00 |       1500.00 |
|           3 |            2 |                1 |     1400.00 |                  700.00 |     700.000000  |         700.00 |        700.00 |
|           4 |            1 |                1 |      300.00 |                  300.00 |     300.000000  |         300.00 |        300.00 |


**Why this output occurs:** For customer 3, there are two orders, both with amount 700. `COUNT(*)` returns 2, but `COUNT(DISTINCT amount)` returns 1. `SUM(amount)` returns 1400 (700 + 700), while `SUM(DISTINCT amount)` returns 700. This demonstrates how distinct and non-distinct aggregates behave differently when duplicates exist.

### Real-World Cases

**Case 1: E-Commerce Customer Dashboard**

An e-commerce platform queries `SELECT customer_id, COUNT(DISTINCT order_id), SUM(total), AVG(total), MAX(order_date) FROM orders GROUP BY customer_id` to show each customer's order count, total spend, average order value, and most recent order date in one row.

**Case 2: Product Inventory Summary**

A warehouse system uses `SELECT product_id, COUNT(DISTINCT warehouse_id) AS warehouses, SUM(quantity) AS total_stock, MIN(expiry_date) AS earliest_expiry FROM inventory GROUP BY product_id` to summarise stock across multiple warehouses.

### References

- SAP Help — Aggregate Functions with DISTINCT - https://help.sap.com/
- GitHub — Multiple Aggregations SQL Handbook - https://github.com/
- QuestDB — Multiple conditional aggregates - https://questdb.com/docs/

---

## 3. GROUPING SETS

### Definitions

**Core Definition:** `GROUPING SETS` is an extension to the `GROUP BY` clause that allows a single query to specify multiple, independent grouping combinations, producing a result set equivalent to the `UNION ALL` of several `GROUP BY` queries but in a single, optimised pass.

**Technical Definition:** The `GROUPING SETS` clause takes a list of grouping specifications, each of which is a parenthesised list of zero or more columns or expressions. Each sublist is interpreted as if it were directly in the `GROUP BY` clause. An empty grouping set `()` aggregates all rows into a single group (grand total). Columns not present in a particular grouping set are represented by `NULL` in the output rows for that set.

**Beginner-Friendly Explanation:** `GROUPING SETS` lets you write one query that produces several different summaries at once. For example, you can show sales by product, sales by region, and the grand total — all in one result set — instead of writing three separate queries and combining them with `UNION ALL`.

### Purposes

- To compute multiple independent groupings in a single query.
- To replace multiple `UNION ALL` queries with a single, more efficient statement.
- To produce custom combinations of subtotals that `ROLLUP` or `CUBE` alone cannot express.
- To generate report layouts with specific grouping requirements.
- To reduce query execution time by scanning the data once.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT column1, column2, aggregate_function(column3)
FROM table_name
GROUP BY GROUPING SETS (
    (column1, column2),
    (column1),
    (column2),
    ()
);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `GROUPING SETS` | Introduces the list of grouping specifications |
| `(column1, column2)` | Groups by both columns |
| `(column1)` | Groups by column1 only |
| `()` | Grand total (no grouping) |

**Syntax Rules:**

- Each grouping set is a parenthesised list of columns or expressions.
- An empty grouping set `()` produces the grand total.
- Columns not included in a grouping set are returned as `NULL` for rows from that set.
- `GROUPING SETS` can be nested; the effect is the same as writing all inner elements in the outer clause.
- `CUBE` and `ROLLUP` can be nested inside `GROUPING SETS`.

**Constraints and Limitations:**

- The order of rows in the result set is not guaranteed; use `ORDER BY` for deterministic ordering.
- The number of grouping sets can grow quickly; too many can degrade performance.
- `GROUPING SETS` is part of ANSI/ISO SQL optional feature T431 and is not supported by all RDBMSs (e.g., older MySQL versions).
- Duplicate grouping sets may produce duplicate rows.

### Annotated Complete Code Examples

**Example 1: GROUPING SETS for Custom Combinations**

```sql
-- Setup: Items sold table
CREATE TABLE items_sold (
    brand   VARCHAR(50),
    size    VARCHAR(10),
    sales   INTEGER
);

INSERT INTO items_sold (brand, size, sales) VALUES
    ('Foo', 'L', 10),
    ('Foo', 'M', 20),
    ('Bar', 'M', 15),
    ('Bar', 'L',  5);

-- Query: Group by brand, by size, and grand total
SELECT brand, size, SUM(sales) AS total_sales
FROM items_sold
GROUP BY GROUPING SETS ((brand), (size), ())
ORDER BY brand NULLS LAST, size NULLS LAST;

-- Expected Output:
--  brand | size | total_sales
-- -------+------+-------------
--  Bar   |      |          20
--  Foo   |      |          30
--        | L    |          15
--        | M    |          35
--        |      |          50
```

**Why this output occurs:** The `GROUPING SETS ((brand), (size), ())` clause specifies three groupings. The first two rows group by brand only, with `size` set to `NULL`. The next two rows group by size only, with `brand` set to `NULL`. The final row is the grand total, with both columns `NULL`. The result is equivalent to `GROUP BY brand UNION ALL GROUP BY size UNION ALL SELECT SUM(sales)`.

### Real-World Cases

**Case 1: Financial Report with Custom Subtotals**

A financial analyst uses `GROUPING SETS ((account_type, fiscal_year), (account_type), (fiscal_year), ())` to produce a report showing revenue by account type and year, subtotals by account type, subtotals by year, and the grand total — all in one query.

**Case 2: Multi-Dimensional Sales Analysis**

A sales team queries `GROUPING SETS ((region, product), (region), (product), ())` to analyse sales by region and product, with subtotals for each dimension and a grand total, enabling drill-down analysis without multiple queries.

### References

- PostgreSQL Documentation — GROUPING SETS, CUBE, and ROLLUP - https://www.postgresql.org/docs/current/queries-table-expressions.html
- Oracle Database SQL Language Reference — GROUPING SETS Clause - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/SELECT.html
- Microsoft Learn — GROUP BY (Transact-SQL) — GROUPING SETS - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-group-by-transact-sql
- AWS — GROUPING SETS - https://docs.aws.amazon.com/

---

## 4. ROLLUP

### Definitions

**Core Definition:** `ROLLUP` is a `GROUP BY` extension that generates a hierarchy of subtotals, rolling up from the most detailed level to a grand total, based on an ordered list of grouping columns.

**Technical Definition:** The `ROLLUP(e1, e2, e3, ...)` construct represents the given list of expressions and all prefixes of the list, including the empty list. It is equivalent to `GROUPING SETS ((e1, e2, e3), (e1, e2), (e1), ())`. ROLLUP first calculates the standard aggregate values specified in the `GROUP BY`, then creates progressively higher-level subtotals, moving from right to left through the list of grouping columns, and finally creates a grand total.

**Beginner-Friendly Explanation:** `ROLLUP` is like a subtotal function in a spreadsheet. If you group sales by year, quarter, and month, `ROLLUP` gives you totals for each month, then subtotals for each quarter, then subtotals for each year, and finally a grand total — all in one query.

### Purposes

- To generate hierarchical subtotals in a single query.
- To produce reports with drill-down levels (e.g., year → quarter → month).
- To calculate grand totals alongside detailed groupings.
- To replace multiple `UNION ALL` queries for hierarchical summaries.
- To support financial and operational reporting with roll-up structures.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT column1, column2, aggregate_function(column3)
FROM table_name
GROUP BY ROLLUP (column1, column2);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `ROLLUP` | Introduces the hierarchical rollup specification |
| `(column1, column2)` | Ordered list of grouping columns, from most detailed to least |
| `column1` | The highest level of detail (leftmost) |
| `column2` | The next level of detail |

**Syntax Rules:**

- `ROLLUP` takes an ordered list of grouping expressions.
- If there are `n` columns, `ROLLUP` produces `n+1` levels of subtotals.
- The order of columns matters: the leftmost column is the highest-level grouping.
- `ROLLUP` can be nested inside `GROUPING SETS`.
- `ROLLUP` can be combined with regular `GROUP BY` columns, producing a cross product of grouping sets.

**Constraints and Limitations:**

- `ROLLUP` supports all aggregate functions available to `GROUP BY`, but some RDBMSs restrict `COUNT DISTINCT` and `SUM DISTINCT` (e.g., SAP IQ).
- `ROLLUP` can only be used in the `SELECT` statement, not in subqueries (in some dialects).
- A multiple grouping specification combining `ROLLUP`, `CUBE`, and regular `GROUP BY` columns may not be supported in all dialects.
- Constant expressions as grouping keys may not be supported.

### Annotated Complete Code Examples

**Example 1: ROLLUP for Hierarchical Subtotals**

```sql
-- Setup: Sales data with year, quarter, and month
CREATE TABLE monthly_sales (
    sales_year  INTEGER,
    quarter     INTEGER,
    month       INTEGER,
    amount      NUMERIC(10, 2)
);

INSERT INTO monthly_sales (sales_year, quarter, month, amount) VALUES
    (2024, 1, 1, 100.00),
    (2024, 1, 2, 150.00),
    (2024, 1, 3, 200.00),
    (2024, 2, 4, 120.00),
    (2024, 2, 5, 180.00),
    (2024, 2, 6, 220.00),
    (2025, 1, 1, 300.00),
    (2025, 1, 2, 250.00);

-- Query: Hierarchical totals by year, quarter, and month
SELECT sales_year, quarter, month, SUM(amount) AS total_sales
FROM monthly_sales
GROUP BY ROLLUP (sales_year, quarter, month)
ORDER BY sales_year NULLS LAST, quarter NULLS LAST, month NULLS LAST;

-- Expected Output:
--  sales_year | quarter | month | total_sales
-- ------------+---------+-------+-------------
--        2024 |       1 |     1 |      100.00
--        2024 |       1 |     2 |      150.00
--        2024 |       1 |     3 |      200.00
--        2024 |       1 |       |      450.00
--        2024 |       2 |     4 |      120.00
--        2024 |       2 |     5 |      180.00
--        2024 |       2 |     6 |      220.00
--        2024 |       2 |       |      520.00
--        2024 |         |       |      970.00
--        2025 |       1 |     1 |      300.00
--        2025 |       1 |     2 |      250.00
--        2025 |       1 |       |      550.00
--        2025 |         |       |      550.00
--             |         |       |     1520.00
```

**Why this output occurs:** `ROLLUP(sales_year, quarter, month)` produces four levels of grouping: (year, quarter, month), (year, quarter), (year), and (). The result shows monthly details, quarterly subtotals (where `month` is `NULL`), yearly subtotals (where `quarter` and `month` are `NULL`), and the grand total (where all three are `NULL`).

### Real-World Cases

**Case 1: Profit and Loss Statement**

A finance team uses `GROUP BY ROLLUP (account_type, account_subtype)` to produce a P&L with subtotals by account type, subtotals by subtype, and a grand total net income.

**Case 2: Regional Sales Hierarchy**

A retail chain queries `GROUP BY ROLLUP (region, country, city)` to generate sales totals at the city, country, region, and global levels for executive reporting.

### References

- PostgreSQL Documentation — ROLLUP - https://www.postgresql.org/docs/current/queries-table-expressions.html
- Oracle Database SQL Language Reference — ROLLUP - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/SELECT.html
- SAP Help — ROLLUP operator - https://help.sap.com/
- Microsoft Learn — ROLLUP - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-group-by-transact-sql

---

## 5. CUBE

### Definitions

**Core Definition:** `CUBE` is a `GROUP BY` extension that generates subtotals for all possible combinations of the specified grouping columns, producing a cross-tabulation of every permutation.

**Technical Definition:** The `CUBE(e1, e2, e3, ...)` construct generates grouping sets for every possible subset of the given list of expressions, including the empty set. For `n` columns, `CUBE` produces `2^n` grouping sets. It is equivalent to a `GROUPING SETS` clause listing all possible combinations.

**Beginner-Friendly Explanation:** `CUBE` is like `ROLLUP` on steroids. If you group sales by product and region, `CUBE` gives you totals for every product-region combination, subtotals for each product, subtotals for each region, and the grand total — all possible combinations.

### Purposes

- To generate cross-tabulation subtotals for all possible combinations.
- To analyse data across multiple dimensions simultaneously.
- To produce complete multi-dimensional summaries in a single query.
- To support OLAP-style analysis and pivot reports.
- To replace multiple `UNION ALL` queries for exhaustive combination summaries.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT column1, column2, aggregate_function(column3)
FROM table_name
GROUP BY CUBE (column1, column2);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `CUBE` | Introduces the cross-tabulation specification |
| `(column1, column2)` | List of grouping columns |
| `2^n` grouping sets | All possible combinations of the listed columns |

**Syntax Rules:**

- `CUBE` takes a list of grouping expressions.
- For `n` columns, `CUBE` produces `2^n` grouping sets.
- Columns not included in a particular grouping set are returned as `NULL`.
- `CUBE` can be nested inside `GROUPING SETS`.
- `CUBE` can be combined with regular `GROUP BY` columns.

**Constraints and Limitations:**

- The number of grouping sets grows exponentially with the number of columns (`2^n`), which can cause performance issues for large `n`.
- Some RDBMSs restrict `COUNT DISTINCT` and `SUM DISTINCT` with `CUBE`.
- `CUBE` does not support inverse distribution analytical functions in some dialects.
- The result set can be very large; use `HAVING` or `WHERE` to filter if needed.

### Annotated Complete Code Examples

**Example 1: CUBE for Cross-Tabulation**

```sql
-- Setup: Sales by product and region
CREATE TABLE product_sales (
    product VARCHAR(50),
    region  VARCHAR(50),
    amount  NUMERIC(10, 2)
);

INSERT INTO product_sales (product, region, amount) VALUES
    ('Laptop', 'North', 1200.00),
    ('Laptop', 'South', 1300.00),
    ('Phone',  'North',  800.00),
    ('Phone',  'South',  900.00),
    ('Tablet', 'North',  600.00),
    ('Tablet', 'South',  550.00);

-- Query: All combinations of product and region totals
SELECT product, region, SUM(amount) AS total_sales
FROM product_sales
GROUP BY CUBE (product, region)
ORDER BY product NULLS LAST, region NULLS LAST;

-- Expected Output:
--  product | region | total_sales
-- ---------+--------+-------------
--  Laptop  | North  |     1200.00
--  Laptop  | South  |     1300.00
--  Laptop  |        |     2500.00
--  Phone   | North  |      800.00
--  Phone   | South  |      900.00
--  Phone   |        |     1700.00
--  Tablet  | North  |      600.00
--  Tablet  | South  |      550.00
--  Tablet  |        |     1150.00
--          | North  |     2600.00
--          | South  |     2750.00
--          |        |     5350.00
```

**Why this output occurs:** `CUBE(product, region)` produces four grouping sets: (product, region), (product), (region), and (). The result includes detailed product-region combinations, product subtotals (where `region` is `NULL`), region subtotals (where `product` is `NULL`), and the grand total (both `NULL`).

### Real-World Cases

**Case 1: Multi-Dimensional Sales Analysis**

A business analyst uses `GROUP BY CUBE (region, product_category, sales_channel)` to analyse sales across three dimensions simultaneously, generating subtotals for every combination for a comprehensive cross-tabulation report.

**Case 2: Healthcare Outcomes Matrix**

A hospital system queries `GROUP BY CUBE (diagnosis, treatment, age_group)` to produce a matrix of patient outcomes across all combinations of diagnosis, treatment, and age group for quality reporting.

### References

- PostgreSQL Documentation — CUBE - https://www.postgresql.org/docs/current/queries-table-expressions.html
- Oracle Database SQL Language Reference — CUBE - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/SELECT.html
- Microsoft Learn — CUBE - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-group-by-transact-sql
- SAP Help — CUBE operator - https://help.sap.com/

---

## 6. The GROUPING() Function

### Definitions

**Core Definition:** The `GROUPING()` function identifies whether a `NULL` value in a grouped result row is a system-generated placeholder from `ROLLUP`, `CUBE`, or `GROUPING SETS`, or a native `NULL` value stored in the data.

**Technical Definition:** `GROUPING(expr)` returns `1` if the value of `expr` is a `NULL` generated by a super-aggregate operation (i.e., the column does not participate in the current grouping set), and `0` if the value is a native `NULL` or a real data value. The `expr` argument must match one of the expressions in the `GROUP BY` clause. The function is applicable only in a `SELECT` statement that contains a `GROUP BY` clause.

**Beginner-Friendly Explanation:** When you use `ROLLUP` or `CUBE`, the database fills in `NULL` for columns that aren't part of a particular subtotal. The `GROUPING()` function tells you whether a `NULL` is a "real" missing value or just a placeholder for a subtotal, so you can label your report correctly.

### Purposes

- To distinguish system-generated `NULL` placeholders from native `NULL` data values.
- To label subtotal and grand total rows in reports.
- To conditionally format output based on whether a row is a detail row or a total row.
- To enable accurate reporting where `NULL` has multiple meanings.
- To support drill-down analysis by identifying aggregation levels.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT column1,
       GROUPING(column1) AS is_total,
       aggregate_function(column2)
FROM table_name
GROUP BY ROLLUP (column1)
ORDER BY column1;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `GROUPING(expr)` | Returns 1 if `expr` is a super-aggregate placeholder, 0 otherwise |
| `expr` | Must match an expression in the `GROUP BY` clause |

**Syntax Rules:**

- `GROUPING()` returns `1` for `NULL` values created by `ROLLUP`, `CUBE`, or `GROUPING SETS`.
- `GROUPING()` returns `0` for native `NULL` values and for real data values.
- The `expr` argument must match one of the grouping expressions.
- `GROUPING()` can be used in the `SELECT` list and `HAVING` clause.
- The function returns a numeric value (Oracle returns `NUMBER`).

**Constraints and Limitations:**

- `GROUPING()` is only valid in a `SELECT` statement that contains a `GROUP BY` clause.
- The `expr` must exactly match a grouping expression.
- `GROUPING()` cannot be used in the `WHERE` clause.
- Behaviour with nested `GROUPING SETS` may vary across dialects.

### Annotated Complete Code Examples

**Example 1: GROUPING() with ROLLUP**

```sql
-- Setup: Sales data
CREATE TABLE sales_data (
    product VARCHAR(50),
    region  VARCHAR(50),
    amount  NUMERIC(10, 2)
);

INSERT INTO sales_data (product, region, amount) VALUES
    ('Laptop', 'North', 1200.00),
    ('Laptop', 'South', 1300.00),
    ('Phone',  'North',  800.00),
    ('Phone',  'South',  900.00);

-- Query: Rollup with GROUPING() to label subtotals
SELECT product,
       region,
       GROUPING(product) AS product_is_total,
       GROUPING(region)  AS region_is_total,
       SUM(amount)       AS total_sales
FROM sales_data
GROUP BY ROLLUP (product, region)
ORDER BY product NULLS LAST, region NULLS LAST;

-- Expected Output:
--  product | region | product_is_total | region_is_total | total_sales
-- ---------+--------+------------------+-----------------+-------------
--  Laptop  | North  |                0 |               0 |     1200.00
--  Laptop  | South  |                0 |               0 |     1300.00
--  Laptop  |        |                0 |               1 |     2500.00
--  Phone   | North  |                0 |               0 |      800.00
--  Phone   | South  |                0 |               0 |      900.00
--  Phone   |        |                0 |               1 |     1700.00
--          |        |                1 |               1 |     5350.00
```

**Why this output occurs:** For detail rows (Laptop/North, etc.), both `GROUPING()` values are 0. For product subtotals (Laptop with `region` = `NULL`), `GROUPING(region)` is 1 because the `NULL` is a placeholder from the rollup, but `GROUPING(product)` is 0 because `product` is a real value. For the grand total row (both `NULL`), both `GROUPING()` values are 1.

**Example 2: Using GROUPING() to Label Report Rows**

```sql
-- Query: Label rows as 'Detail', 'Subtotal', or 'Grand Total'
SELECT CASE
           WHEN GROUPING(product) = 1 THEN 'Grand Total'
           WHEN GROUPING(region) = 1 THEN 'Subtotal'
           ELSE 'Detail'
       END AS row_type,
       COALESCE(product, 'All Products') AS product,
       COALESCE(region, 'All Regions') AS region,
       SUM(amount) AS total_sales
FROM sales_data
GROUP BY ROLLUP (product, region)
ORDER BY row_type, product, region;

-- Expected Output:
--   row_type   |   product   |   region    | total_sales
-- -------------+-------------+-------------+-------------
--  Detail      | Laptop      | North       |     1200.00
--  Detail      | Laptop      | South       |     1300.00
--  Detail      | Phone       | North       |      800.00
--  Detail      | Phone       | South       |      900.00
--  Grand Total | All Products| All Regions |     5350.00
--  Subtotal    | Laptop      | All Regions |     2500.00
--  Subtotal    | Phone       | All Regions |     1700.00
```

**Why this output occurs:** The `CASE` expression uses `GROUPING()` to determine the row type. `COALESCE` replaces `NULL` placeholders with descriptive labels. The result is a human-readable report with properly labelled subtotal and grand total rows.

### Real-World Cases

**Case 1: Financial Report Formatting**

A financial analyst uses `GROUPING()` to apply different formatting to subtotal rows (bold, shaded) versus detail rows in a P&L statement, and to insert labels like "Total Revenue" instead of leaving `NULL`.

**Case 2: Inventory Report with Subtotals**

A warehouse report uses `GROUPING(warehouse_id)` to label warehouse subtotals and `GROUPING(product_id)` to label product subtotals, ensuring end users can clearly distinguish between actual `NULL` values and summary rows.

### References

- Oracle Database SQL Language Reference — GROUPING Function - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/GROUPING.html
- SAP Help — GROUPING function - https://help.sap.com/
- PostgreSQL Documentation — GROUPING Function - https://www.postgresql.org/docs/current/functions-aggregate.html

---

## 7. Advanced Reporting Queries

### Definitions

**Core Definition:** Advanced reporting queries combine conditional aggregation, multiple aggregates, and grouping extensions (`GROUPING SETS`, `ROLLUP`, `CUBE`) with other SQL features to build sophisticated, real-world reports such as financial statements, cohort analyses, and data matrices.

**Technical Definition:** Advanced reporting queries typically combine Common Table Expressions (CTEs), window functions, conditional aggregates, and grouping extensions to produce multi-dimensional, time-aware summaries. These queries may use `FILTER` clauses (PostgreSQL), `PIVOT` operators (SQL Server, Oracle), or manual pivot logic with `CASE` expressions to transform row-level data into report-ready matrices.

**Beginner-Friendly Explanation:** These are the queries that power real business reports — monthly financial sheets, customer retention charts, and cross-tabulated sales matrices. They combine all the advanced techniques into practical, production-ready SQL.

### Purposes

- To build end-of-month financial reports with hierarchical subtotals.
- To perform cohort analysis by grouping users based on their first activity date.
- To create cross-tabulation data matrices for multi-dimensional analysis.
- To produce dashboard-ready datasets with multiple KPIs per row.
- To support decision-making with comprehensive, aggregated business intelligence.

### Syntax Rules and Structure

**Complete General Syntax (General Pattern):**

```sql
WITH cte AS (
    -- Pre-aggregate or transform data
    SELECT ..., expression AS derived_column
    FROM source_table
    WHERE ...
)
SELECT grouping_columns,
       aggregate_function(CASE WHEN condition THEN value END) AS metric1,
       aggregate_function(CASE WHEN condition THEN value END) AS metric2,
       ...
FROM cte
GROUP BY GROUPING SETS / ROLLUP / CUBE (...)
HAVING ...
ORDER BY ...;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `WITH` (CTE) | Pre-compute derived values or cohort labels |
| Conditional aggregates | Compute custom metrics per group |
| `GROUPING SETS` / `ROLLUP` / `CUBE` | Generate hierarchical or cross-tabular subtotals |
| `HAVING` | Filter groups by aggregate conditions |
| `ORDER BY` | Sort the final report |

**Syntax Rules:**

- CTEs can be used to stage data before aggregation.
- Conditional aggregates can be combined with any grouping extension.
- Window functions can be layered on top of aggregated results.
- `FILTER` clauses (PostgreSQL) offer a cleaner alternative to `CASE` for conditional aggregation.

**Constraints and Limitations:**

- Complex reporting queries can be resource-intensive; indexes and materialised views may be needed.
- Cohort analysis queries scan large historical datasets and can be expensive.
- Cross-tabulation with `CUBE` grows exponentially with the number of dimensions.
- Dialect differences affect portability of advanced reporting queries.

### Annotated Complete Code Examples

**Example 1: End-of-Month Financial Summary**

```sql
-- Setup: Transactions table
CREATE TABLE transactions (
    txn_id      SERIAL PRIMARY KEY,
    txn_date    DATE,
    account_type VARCHAR(50),
    amount      NUMERIC(10, 2)
);

INSERT INTO transactions (txn_date, account_type, amount) VALUES
    ('2024-01-05', 'Revenue', 10000.00),
    ('2024-01-10', 'Revenue', 15000.00),
    ('2024-01-15', 'Expense', 5000.00),
    ('2024-01-20', 'Expense', 3000.00),
    ('2024-02-05', 'Revenue', 12000.00),
    ('2024-02-10', 'Revenue', 18000.00),
    ('2024-02-15', 'Expense', 6000.00),
    ('2024-02-20', 'Expense', 4000.00);

-- Query: Monthly financial summary with net income
SELECT EXTRACT(MONTH FROM txn_date) AS month_num,
       SUM(CASE WHEN account_type = 'Revenue' THEN amount ELSE 0 END) AS total_revenue,
       SUM(CASE WHEN account_type = 'Expense' THEN amount ELSE 0 END) AS total_expense,
       SUM(CASE WHEN account_type = 'Revenue' THEN amount ELSE 0 END) -
       SUM(CASE WHEN account_type = 'Expense' THEN amount ELSE 0 END) AS net_income
FROM transactions
GROUP BY ROLLUP (EXTRACT(MONTH FROM txn_date))
ORDER BY month_num NULLS LAST;

-- Expected Output:
--  month_num | total_revenue | total_expense | net_income
-- -----------+---------------+---------------+------------
--          1 |      25000.00 |       8000.00 |   17000.00
--          2 |      30000.00 |      10000.00 |   20000.00
--            |      55000.00 |      18000.00 |   37000.00
```

**Why this output occurs:** The query uses conditional aggregates to separate revenue and expense amounts, then computes net income as the difference. The `ROLLUP` on the month expression adds a grand total row (where `month_num` is `NULL`) showing the cumulative revenue, expense, and net income.

**Example 2: User Cohort Analysis**

```sql
-- Setup: User signups and activity
CREATE TABLE user_activity (
    user_id     INTEGER,
    activity_date DATE,
    activity_type VARCHAR(20)
);

INSERT INTO user_activity (user_id, activity_date, activity_type) VALUES
    (1, '2024-01-10', 'signup'),
    (1, '2024-01-15', 'purchase'),
    (2, '2024-01-15', 'signup'),
    (2, '2024-02-01', 'purchase'),
    (3, '2024-02-05', 'signup'),
    (3, '2024-02-20', 'purchase'),
    (4, '2024-02-10', 'signup');

-- Query: Cohort analysis — count users by signup month
SELECT DATE_TRUNC('month', signup_date) AS cohort_month,
       COUNT(DISTINCT user_id) AS cohort_size
FROM (
    SELECT user_id,
           MIN(CASE WHEN activity_type = 'signup' THEN activity_date END) AS signup_date
    FROM user_activity
    GROUP BY user_id
) AS user_signups
GROUP BY DATE_TRUNC('month', signup_date)
ORDER BY cohort_month;

-- Expected Output:
--   cohort_month   | cohort_size
-- -----------------+-------------
--  2024-01-01 00:00:00 |           2
--  2024-02-01 00:00:00 |           2
```

**Why this output occurs:** The inner subquery determines each user's signup date using `MIN` with a `CASE` filter. The outer query groups users by their signup month (`DATE_TRUNC('month', ...)`) and counts the distinct users in each cohort. Users 1 and 2 signed up in January (cohort size 2), and users 3 and 4 signed up in February (cohort size 2).

**Example 3: Cross-Tabulation Data Matrix**

```sql
-- Setup: Product sales by region
CREATE TABLE product_region_sales (
    product VARCHAR(50),
    region  VARCHAR(50),
    amount  NUMERIC(10, 2)
);

INSERT INTO product_region_sales (product, region, amount) VALUES
    ('Laptop', 'North',  1200.00), 
    ('Laptop', 'South',  1300.00), 
    ('Laptop', 'East',   1100.00),
    ('Phone',  'North',  800.00), 
    ('Phone',  'South',  900.00), 
    ('Phone',  'East',   850.00),
    ('Tablet', 'North',  600.00), 
    ('Tablet', 'South',  550.00), 
    ('Tablet', 'East',   500.00);

-- Query: Cross-tabulation matrix with product as rows, region as columns
SELECT product,
       SUM(CASE WHEN region = 'North' THEN amount ELSE 0 END) AS north,
       SUM(CASE WHEN region = 'South' THEN amount ELSE 0 END) AS south,
       SUM(CASE WHEN region = 'East'  THEN amount ELSE 0 END) AS east,
       SUM(amount) AS total
FROM product_region_sales
GROUP BY ROLLUP (product)
ORDER BY product NULLS LAST;

-- Expected Output:
--  product |  north  |  south  |  east   |  total
-- ---------+---------+---------+---------+---------
--  Laptop  | 1200.00 | 1300.00 | 1100.00 | 3600.00
--  Phone   |  800.00 |  900.00 |  850.00 | 2550.00
--  Tablet  |  600.00 |  550.00 |  500.00 | 1650.00
--          | 2600.00 | 2750.00 | 2450.00 | 7800.00
```

**Why this output occurs:** The conditional aggregates create a column for each region, pivoting the data. The `ROLLUP(product)` adds a grand total row at the bottom. The result is a classic cross-tabulation matrix with product rows, region columns, and totals.

### Real-World Cases

**Case 1: Monthly Financial Reporting**

A CFO's office runs a report using conditional aggregates and `ROLLUP` to produce a monthly P&L with revenue, expense, and net income columns, plus quarterly and yearly subtotals.

**Case 2: Customer Retention Cohort Analysis**

A subscription business uses cohort analysis queries to track retention rates: customers grouped by signup month, with counts of active users in subsequent months, revealing churn patterns over time.

**Case 3: Sales Performance Matrix**

A sales director queries a cross-tabulation of sales reps by product category, showing each rep's performance across categories with a row total, enabling quick comparison of strengths and weaknesses.

### References

- Oracle Database Data Warehousing Guide — SQL for Aggregation in Data Warehouses - https://docs.oracle.com/en/database/oracle/oracle-database/21/dwhsg/
- Oracle Database — SQL for Analysis and Reporting - https://docs.oracle.com/cd/B28359_01/server.111/b28314.pdf
- Microsoft Learn — GROUP BY (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-group-by-transact-sql
- GitHub — SQL Engineering Handbook: Cohort + Running-Total Dashboard Query - https://github.com/
- UltraCart Documentation — Cohort and Churn Queries - https://docs.ultracart.com/

---

## Summary Table of Advanced Aggregation Features

| Feature | Purpose | Key Limitation |
|---------|---------|----------------|
| Conditional Aggregation | Custom summary columns per condition | Cannot be used in `WHERE` |
| Multiple Aggregates | Multi-metric summaries in one query | Performance cost of multiple computations |
| `COUNT(DISTINCT)` | Count unique values | Slower than `COUNT(*)` |
| `GROUPING SETS` | Custom grouping combinations | Grows with number of sets |
| `ROLLUP` | Hierarchical subtotals | Order-dependent; `n+1` levels |
| `CUBE` | All possible combinations | Exponential growth (`2^n`) |
| `GROUPING()` | Identify system-generated totals | Only valid with `GROUP BY` |
| Advanced Reporting | Real-world multi-dimensional reports | Resource-intensive |

---

## Dialect Support for Advanced Grouping

| Feature | PostgreSQL | Oracle | SQL Server | MySQL |
|---------|-----------|--------|------------|-------|
| Conditional Aggregation | ✅ | ✅ | ✅ | ✅ |
| `COUNT(DISTINCT)` | ✅ | ✅ | ✅ | ✅ |
| `GROUPING SETS` | ✅ | ✅ | ✅ | ❌ (MySQL 8.0+) |
| `ROLLUP` | ✅ | ✅ | ✅ | ✅ |
| `CUBE` | ✅ | ✅ | ✅ | ❌ |
| `GROUPING()` | ✅ | ✅ | ✅ | ❌ |
| `FILTER` clause | ✅ | ❌ | ❌ | ❌ |

---

## References

- PostgreSQL Documentation — Aggregate Functions - https://www.postgresql.org/docs/current/functions-aggregate.html
- PostgreSQL Documentation — GROUPING SETS, CUBE, and ROLLUP - https://www.postgresql.org/docs/current/queries-table-expressions.html
- Oracle Database SQL Language Reference — GROUPING Function - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/GROUPING.html
- Oracle Database SQL Language Reference — SELECT (GROUPING SETS, ROLLUP, CUBE) - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/SELECT.html
- Oracle Database Data Warehousing Guide — SQL for Aggregation in Data Warehouses - https://docs.oracle.com/en/database/oracle/oracle-database/21/dwhsg/
- Oracle Business Intelligence Server — Using FILTER to Compute a Conditional Aggregate - https://docs.oracle.com/cd/E10415_01/doc/bi.1013/b31770.pdf
- Microsoft Learn — GROUP BY (Transact-SQL) — GROUPING SETS, ROLLUP, CUBE - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-group-by-transact-sql
- SAP Help — GROUP BY, ROLLUP, CUBE, GROUPING - https://help.sap.com/
- AWS — GROUPING SETS - https://docs.aws.amazon.com/
- QuestDB — Multiple conditional aggregates in a single query - https://questdb.com/docs/
- GitHub — SQL Engineering Handbook: Multiple Aggregations - https://github.com/
- UltraCart Documentation — Cohort and Churn Queries - https://docs.ultracart.com/
- ISO/IEC 9075-2:2003 — SQL Standard (Feature T431: Extended Grouping Capabilities) - https://jtc1sc32.org/doc/N0251-0300/32N0254.pdf