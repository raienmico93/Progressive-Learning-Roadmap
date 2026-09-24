# SQL Window Function Fundamentals & Core Architecture: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A window function performs a calculation across a set of table rows that are somehow related to the current row, returning a value for *each* row rather than collapsing them into a single output row.

**Technical Definition:** A window function operates on a "window" — a set of rows defined by the `OVER` clause — and produces a result for each query row. Unlike aggregate functions with `GROUP BY`, which group rows into a single result row, window functions do not collapse groups of query rows to a single output row. Instead, they produce a result for each row . The window is defined by the `OVER` clause, which can include `PARTITION BY` (dividing rows into groups), `ORDER BY` (defining logical order within partitions), and frame clauses (`ROWS` or `RANGE`) that further limit the rows considered for each calculation .

**Beginner-Friendly Explanation:** Imagine you have a table of employee salaries. A regular aggregate with `GROUP BY` would give you one row per department showing the average salary. A window function gives you the average salary *on every single employee row*, so you can compare each person's salary to their department's average without losing any detail.

### Key Characteristics

- **Row-preserving:** Window functions return a value for every query row; they do not collapse rows like `GROUP BY` does .
- **Post-processing:** Window functions logically execute *after* `FROM`, `WHERE`, `GROUP BY`, and `HAVING` processing, but *before* `ORDER BY` .
- **Clause placement:** Window functions are permitted only in the `SELECT` list and `ORDER BY` clause — not in `WHERE`, `GROUP BY`, or `HAVING` .
- **Flexible partitioning:** A query can contain multiple window functions with different `OVER` clauses, each slicing the data differently .
- **Named windows:** A `WINDOW` clause can define a named window specification that multiple `OVER` clauses can reference, reducing duplication .
- **Platform consistency:** The core concepts are standard across MySQL, PostgreSQL, SQL Server, BigQuery, and SAP HANA, with minor syntactic variations .

### Prerequisites

- **Basic SQL proficiency:** `SELECT`, `FROM`, `WHERE`, `GROUP BY`, `ORDER BY`.
- **Aggregate function knowledge:** `SUM`, `AVG`, `COUNT`, `MIN`, `MAX`.
- **Subquery awareness:** Understanding of how queries are structured.
- **Set-based thinking:** Comfort with the idea of operating on sets of rows.

### Related Programming Areas

- **Data Analysis & BI:** Running totals, moving averages, rankings, percentile calculations.
- **Application Development:** Per-group top-N queries, comparative analytics.
- **Data Engineering:** Windowed aggregations in ETL pipelines.
- **Reporting:** Year-over-year comparisons, cohort analysis.

### Core Concepts / Features

1. The Windowing Concept
2. The `OVER` Clause
3. `PARTITION BY` Mechanics
4. `ORDER BY` in Windowing

---

## 1. The Windowing Concept

### Definitions

**Core Definition:** Windowing is a querying technique where a function operates on a set of rows related to the current row (the "window") while still returning a result for each individual row.

**Technical Definition:** A window function performs an aggregate-like operation on a set of query rows, but whereas an aggregate operation groups query rows into a single result row, a window function produces a result for *each* query row . The row for which function evaluation occurs is called the *current row*. The query rows related to the current row over which function evaluation occurs comprise the *window* for the current row . This allows calculations that retain row-level detail while incorporating group-level context.

**Beginner-Friendly Explanation:** Think of a classroom. `GROUP BY` is like the teacher writing one grade per group on the board — you lose individual student names. Windowing is like writing the group average next to *each* student's name, so every student can see how they compare to the group.

### Purposes

- To compute aggregate values without collapsing rows, preserving individual detail.
- To compare each row against a group-level metric (e.g., department average).
- To calculate running totals, moving averages, and cumulative sums.
- To rank rows within categories without losing other columns.
- To perform top-N per group queries elegantly.

### Syntax Rules and Structure

**Aggregate vs. Window Function Comparison:**

```sql
-- Aggregate: one row per group
SELECT country, SUM(sales) AS total_sales
FROM sales_data
GROUP BY country;

-- Window: one row per query row, with group total attached
SELECT country, sales,
       SUM(sales) OVER (PARTITION BY country) AS country_total
FROM sales_data;
```

**Component Breakdown:**

| Concept | Aggregate (`GROUP BY`) | Window Function |
|---------|----------------------|-----------------|
| Output rows | One per group | One per query row  |
| Row detail | Lost | Preserved  |
| Execution order | Before window functions | After `GROUP BY` / `HAVING`  |
| Clause placement | In `SELECT` with `GROUP BY` | In `SELECT` list or `ORDER BY` |

**Syntax Rules:**

- Window functions are permitted only in the `SELECT` list and the `ORDER BY` clause; they are forbidden in `GROUP BY`, `HAVING`, and `WHERE` .
- Window functions execute after non-window aggregate functions; it is valid to include an aggregate function call in the arguments of a window function, but not vice versa .
- If filtering or grouping is needed after window calculations, wrap the query in a subquery .

**Constraints and Limitations:**

- Window functions cannot appear in `WHERE` because they logically execute after `WHERE` processing .
- To filter based on a window function result (e.g., top-3 per group), you must use a subquery or CTE .

### Annotated Complete Code Examples

**Example 1: Aggregate vs. Window Comparison**

```sql
-- Setup: Sales data table
CREATE TABLE sales_data (
    country VARCHAR(50),
    sales   NUMERIC(10, 2)
);

INSERT INTO sales_data VALUES
    ('USA', 100.00),
    ('USA', 200.00),
    ('Canada', 150.00),
    ('Canada', 250.00);

-- Aggregate query: one row per country
SELECT country, SUM(sales) AS total_sales
FROM sales_data
GROUP BY country;

-- Expected Output:
--  country | total_sales
-- ---------+-------------
--  USA     |      300.00
--  Canada  |      400.00

-- Window function query: one row per original row
SELECT country, sales,
       SUM(sales) OVER (PARTITION BY country) AS country_total
FROM sales_data;

-- Expected Output:
--  country | sales  | country_total
-- ---------+--------+---------------
--  USA     | 100.00 |        300.00
--  USA     | 200.00 |        300.00
--  Canada  | 150.00 |        400.00
--  Canada  | 250.00 |        400.00
```

**Why this output occurs:** The aggregate query collapses each country's rows into one. The window function retains all four rows while attaching the country-level total to each row, enabling per-row comparison with the group total .

### Real-World Cases

**Case 1: Salary Benchmarking**

An HR analyst wants each employee's salary alongside their department's average. A window function `AVG(salary) OVER (PARTITION BY department)` attaches the department average to each employee row, enabling immediate comparison.

**Case 2: Sales Contribution Analysis**

A sales manager needs each transaction's contribution to the regional total. `SUM(amount) OVER (PARTITION BY region)` provides the regional total on every transaction row.

### References

- MySQL 8.0 Reference Manual — Window Function Concepts and Syntax - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/window-functions-usage.html
- PostgreSQL Documentation — Window Functions - https://git.postgresql.org/cgit/pgrpms.git/plain/rpm/redhat/10/postgresql/master/postgresql-10-A4.pdf

---

## 2. The OVER Clause

### Definitions

**Core Definition:** The `OVER` clause defines the window — the set of rows — that a window function operates on, specifying how rows are partitioned, ordered, and framed.

**Technical Definition:** The `OVER` clause determines the partitioning and ordering of a rowset before the associated window function is applied . It can be empty (`OVER ()`), which treats the entire result set as a single window . The clause accepts three optional components: `PARTITION BY`, `ORDER BY`, and a frame specification (`ROWS` or `RANGE`) .

**Beginner-Friendly Explanation:** The `OVER` clause is like the instruction sheet for the window function. It tells the database: "Split the data by these columns, sort it this way, and only look at this range of rows when calculating the result."

### Purposes

- To define which rows are included in the window calculation.
- To partition data into independent groups for window function processing.
- To establish row ordering for ranking and frame-based functions.
- To limit the window to a subset of rows (e.g., a moving average window).

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
function_name ( expression ) OVER (
    [ PARTITION BY partition_expression, ... ]
    [ ORDER BY order_expression [ ASC | DESC ], ... ]
    [ ROWS | RANGE frame_specification ]
)
```

**Empty OVER Clause:**

```sql
SUM(sales) OVER () AS global_total
```

**Named Window Reference:**

```sql
SELECT SUM(sales) OVER w, AVG(sales) OVER w
FROM sales_data
WINDOW w AS (PARTITION BY country ORDER BY sales DESC);
```

**Component Breakdown:**

| Component | Purpose | Required? |
|-----------|---------|-----------|
| `PARTITION BY` | Divides result set into groups | Optional  |
| `ORDER BY` | Defines logical order within partitions | Optional  |
| `ROWS` / `RANGE` | Limits rows within partition | Optional; requires `ORDER BY`  |
| Named window | Reusable window definition | Optional  |

**Syntax Rules:**

- `OVER ()` with no arguments means the entire query result set is a single window .
- `PARTITION BY` divides rows into independent groups; the function is applied to each partition separately .
- If `PARTITION BY` is omitted, there is a single partition containing all rows .
- `ORDER BY` defines the logical order in which the window function calculation is performed .
- `ROWS` or `RANGE` requires `ORDER BY` and further limits the rows within the partition .
- A `WINDOW` clause can define named windows for reuse, reducing duplication when multiple functions share the same windowing behavior .

**Constraints and Limitations:**

- `OVER` cannot be used with the `CHECKSUM` aggregate function .
- `RANGE` cannot be used with numeric `PRECEDING` or `FOLLOWING` specifications (only `ROWS` supports numeric offsets) .
- If `ORDER BY` is specified but `ROWS`/`RANGE` is not, the default frame is `RANGE UNBOUNDED PRECEDING AND CURRENT ROW` for functions that accept frames .

### Annotated Complete Code Examples

**Example 1: Empty OVER vs. PARTITION BY**

```sql
-- Using the sales_data table from above

-- Empty OVER: global total on every row
SELECT country, sales,
       SUM(sales) OVER () AS global_total
FROM sales_data;

-- Expected Output:
--  country | sales  | global_total
-- ---------+--------+-------------
--  USA     | 100.00 |       700.00
--  USA     | 200.00 |       700.00
--  Canada  | 150.00 |       700.00
--  Canada  | 250.00 |       700.00

-- PARTITION BY: per-country total on every row
SELECT country, sales,
       SUM(sales) OVER (PARTITION BY country) AS country_total
FROM sales_data;

-- Expected Output:
--  country | sales  | country_total
-- ---------+--------+---------------
--  USA     | 100.00 |        300.00
--  USA     | 200.00 |        300.00
--  Canada  | 150.00 |        400.00
--  Canada  | 250.00 |        400.00
```

**Why this output occurs:** `OVER ()` treats all rows as one window, so the global total (700.00) appears on every row. `OVER (PARTITION BY country)` creates separate windows for USA and Canada, so the total differs per partition .

### Real-World Cases

**Case 1: Top-N Per Group**

A query needs the top 3 products by sales in each category. A window function `ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC)` assigns a rank within each category; an outer query filters for `rank <= 3` .

**Case 2: Moving Average**

A financial analyst computes a 7-day moving average of stock prices. `AVG(price) OVER (ORDER BY trade_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)` defines a sliding window of 7 rows .

### References

- Microsoft Learn — OVER Clause (Transact-SQL) - https://learn.microsoft.com/ga-ie/sql/t-sql/queries/select-over-clause-transact-sql
- PostgreSQL Documentation — Window Functions - https://git.postgresql.org/cgit/pgrpms.git/plain/rpm/redhat/10/postgresql/master/postgresql-10-A4.pdf
- MySQL 8.0 Reference Manual — Window Function Concepts and Syntax - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/window-functions-usage.html

---

## 3. PARTITION BY Mechanics

### Definitions

**Core Definition:** `PARTITION BY` divides the query's result set into separate groups (partitions) based on the unique values of the specified expression(s), and the window function is applied independently within each partition.

**Technical Definition:** The `PARTITION BY` clause organizes the result set into logical groups based on the unique values of the specified expression . When used with window functions, the functions are applied to each partition independently. If `PARTITION BY` is omitted, there is a single partition consisting of all query rows .

**Beginner-Friendly Explanation:** `PARTITION BY` is like dividing a large spreadsheet into separate sheets, one for each category. The window function does its calculation on each sheet separately, so each category gets its own results.

### Purposes

- To calculate group-level metrics (e.g., department average) while retaining all rows.
- To enable per-category rankings and running totals.
- To isolate calculations so they don't bleed across category boundaries.
- To support top-N-per-group queries.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
function_name ( expression ) OVER (
    PARTITION BY partition_expression [, partition_expression, ... ]
)
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `PARTITION BY` | Divides rows into independent groups |
| `partition_expression` | Column(s) or expression(s) used for grouping |
| Multiple expressions | Creates partitions for each unique combination |

**Syntax Rules:**

- `PARTITION BY` creates one partition per distinct value of the expression .
- If multiple expressions are specified, partitions are formed for each unique combination .
- The `PARTITION BY` expression cannot be an integer literal .
- If `PARTITION BY` is omitted, the entire result set is treated as a single partition .
- `PARTITION BY` does not reduce the number of output rows .

**NULL Handling in Partitions:**

- `NULL` values are treated as equivalent to each other for partitioning purposes.
- All rows with `NULL` in the partition column form a single partition.
- This behavior is consistent with `GROUP BY` NULL handling.

**Constraints and Limitations:**

- `PARTITION BY` cannot reference column aliases from the `SELECT` list.
- The expression must be made available by the `FROM` clause .

### Annotated Complete Code Examples

**Example 1: Partitioning by Department**

```sql
-- Setup: Employee salaries by department
CREATE TABLE employees (
    emp_name   VARCHAR(50),
    department VARCHAR(50),
    salary     NUMERIC(10, 2)
);

INSERT INTO employees VALUES
    ('Alice', 'Engineering', 95000.00),
    ('Bob',   'Engineering', 105000.00),
    ('Carol', 'Marketing',   72000.00),
    ('David', 'Marketing',   88000.00),
    ('Eve',   'Sales',       68000.00);

-- Query: Each employee's salary and their department's average
SELECT emp_name, department, salary,
       AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM employees
ORDER BY department, salary;

-- Expected Output:
--  emp_name | department  |  salary  |     dept_avg
-- ----------+-------------+----------+--------------------
--  Alice    | Engineering | 95000.00 | 100000.000000000000
--  Bob      | Engineering |105000.00 | 100000.000000000000
--  Carol    | Marketing   | 72000.00 |  80000.000000000000
--  David    | Marketing   | 88000.00 |  80000.000000000000
--  Eve      | Sales       | 68000.00 |  68000.000000000000
```

**Why this output occurs:** `PARTITION BY department` creates three independent windows. The `AVG` is computed within each department separately. Every row retains its detail while showing the department average .

**Example 2: Partitioning by Multiple Columns**

```sql
-- Setup: Sales by region and year
CREATE TABLE regional_sales (
    region VARCHAR(50),
    year   INTEGER,
    sales  NUMERIC(10, 2)
);

INSERT INTO regional_sales VALUES
    ('North', 2024, 100.00),
    ('North', 2024, 200.00),
    ('North', 2025, 150.00),
    ('South', 2024, 120.00),
    ('South', 2025, 180.00);

-- Query: Total sales per region per year
SELECT region, year, sales,
       SUM(sales) OVER (PARTITION BY region, year) AS region_year_total
FROM regional_sales
ORDER BY region, year;

-- Expected Output:
--  region | year | sales  | region_year_total
-- --------+------+--------+-------------------
--  North  | 2024 | 100.00 |            300.00
--  North  | 2024 | 200.00 |            300.00
--  North  | 2025 | 150.00 |            150.00
--  South  | 2024 | 120.00 |            120.00
--  South  | 2025 | 180.00 |            180.00
```

**Why this output occurs:** `PARTITION BY region, year` creates four partitions: (North, 2024), (North, 2025), (South, 2024), (South, 2025). The sum is computed independently within each unique combination .

### Real-World Cases

**Case 1: Departmental Salary Comparison**

An HR dashboard shows each employee's salary alongside their department's average, enabling managers to identify outliers.

**Case 2: Monthly Sales by Region**

A sales report partitions by region and month to show each transaction's contribution to its region-month total.

### References

- SAP HANA Cloud — PARTITION BY clause - https://help.sap.com/docs/SAP_HANA_DATA_LAKE_CN/73a183f2600c4b7890db9ffe2d09570e/a588f918af8b441a939c39e987597230.html
- MySQL 8.0 Reference Manual — Window Function Concepts and Syntax - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/window-functions-usage.html
- Microsoft Learn — Use the OVER clause - https://learn.microsoft.com/lb-lu/training/modules/write-queries-that-use-window-functions/3-use-over-clause

---

## 4. ORDER BY in Windowing

### Definitions

**Core Definition:** `ORDER BY` within the `OVER` clause defines the logical order of rows within each partition, controlling how ranking functions rank rows and how frame-based functions (running totals, moving averages) process rows sequentially.

**Technical Definition:** The `ORDER BY` clause defines the logical order of the rows within each partition of the result set, specifying the logical order in which the window function calculation is performed . If `ORDER BY` is specified and a `ROWS` or `RANGE` clause is not specified, the default window frame is `RANGE UNBOUNDED PRECEDING AND CURRENT ROW` for functions that accept frame specifications .

**Beginner-Friendly Explanation:** `ORDER BY` in the `OVER` clause is like telling the database: "Process these rows in this order." It matters for ranking (who's first?) and for running calculations (cumulative sum up to this row).

### Purposes

- To establish sequence for ranking functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`).
- To define the direction of cumulative calculations (running totals, moving averages).
- To determine the default frame boundaries when no explicit frame is given.
- To control which rows are included in frame-based calculations.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
function_name ( expression ) OVER (
    [ PARTITION BY partition_expression ]
    ORDER BY order_expression [ ASC | DESC ] [, ...]
    [ ROWS | RANGE frame_specification ]
)
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `ORDER BY` | Defines logical order within partitions |
| `ASC` / `DESC` | Sort direction (ASC is default) |
| `COLLATE` | Collation for string sorting  |
| `ROWS` / `RANGE` | Frame boundaries (depends on `ORDER BY`) |

**Default Frame Boundaries:**

| Scenario | Default Frame |
|----------|--------------|
| No `ORDER BY` | Entire partition  |
| `ORDER BY` without frame | `RANGE UNBOUNDED PRECEDING AND CURRENT ROW`  |
| Ranking functions | Ignore frame clause entirely |

**Syntax Rules:**

- `ORDER BY` within `OVER` can only refer to columns made available by the `FROM` clause; integer column positions are not allowed .
- `ASC` is the default sort order; `NULL` values are treated as the lowest possible values .
- If `ORDER BY` is omitted, the default frame is the entire partition .
- `ROWS` or `RANGE` requires `ORDER BY` .
- When `ORDER BY` is specified without `ROWS`/`RANGE`, the default frame for functions like `SUM` and `AVG` is from the start of the partition to the current row .

**Performance Considerations:**

- Window functions may require sorting the entire dataset, which can be expensive on large tables .
- Using a shared sort order for multiple window functions avoids repeated sorts .
- For large datasets with high cardinality, window functions are recommended over `GROUP BY` + `JOIN` .

**Constraints and Limitations:**

- Ranking functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`) require `ORDER BY` .
- `RANGE` cannot be used with numeric `PRECEDING` or `FOLLOWING` specifications .
- The default frame `RANGE UNBOUNDED PRECEDING AND CURRENT ROW` includes all rows with the same `ORDER BY` value as the current row .

### Annotated Complete Code Examples

**Example 1: ORDER BY Without Frame (Cumulative Sum)**

```sql
-- Using the sales_data table from Example 1
SELECT country, sales,
       SUM(sales) OVER (ORDER BY sales) AS running_total
FROM sales_data
ORDER BY sales;

-- Expected Output:
--  country | sales  | running_total
-- ---------+--------+---------------
--  USA     | 100.00 |        100.00
--  Canada  | 150.00 |        250.00
--  USA     | 200.00 |        450.00
--  Canada  | 250.00 |        700.00
```

**Why this output occurs:** With `ORDER BY sales` and no explicit frame, the default frame is `RANGE UNBOUNDED PRECEDING AND CURRENT ROW`. The sum accumulates from the first row (lowest sales) through the current row .

**Example 2: Explicit ROWS Frame for Moving Average**

```sql
-- Query: 2-row moving average (current and previous row)
SELECT country, sales,
       AVG(sales) OVER (ORDER BY sales ROWS BETWEEN 1 PRECEDING AND CURRENT ROW) AS moving_avg
FROM sales_data
ORDER BY sales;

-- Expected Output:
--  country | sales  |     moving_avg
-- ---------+--------+--------------------
--  USA     | 100.00 | 100.000000000000000
--  Canada  | 150.00 | 125.000000000000000
--  USA     | 200.00 | 175.000000000000000
--  Canada  | 250.00 | 225.000000000000000
```

**Why this output occurs:** `ROWS BETWEEN 1 PRECEDING AND CURRENT ROW` defines a window of exactly 2 rows (previous + current). The first row has no preceding row, so its average is just itself. The second row averages 100 and 150, and so on .

### Real-World Cases

**Case 1: Running Total Report**

A financial report shows cumulative revenue by day. `SUM(revenue) OVER (ORDER BY sale_date)` produces a running total that accumulates from the first day .

**Case 2: Ranking Employees by Salary**

An HR query uses `RANK() OVER (PARTITION BY department ORDER BY salary DESC)` to rank employees within each department, identifying top performers .

### References

- Microsoft Learn — OVER Clause (Transact-SQL) - https://learn.microsoft.com/ga-ie/sql/t-sql/queries/select-over-clause-transact-sql
- PostgreSQL Documentation — Window Functions - https://git.postgresql.org/cgit/pgrpms.git/plain/rpm/redhat/10/postgresql/master/postgresql-10-A4.pdf
- MySQL 8.0 Reference Manual — Window Function Concepts and Syntax - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/window-functions-usage.html
- Microsoft Learn — Use the OVER clause - https://learn.microsoft.com/en-gb/training/modules/write-queries-that-use-window-functions/3-use-over-clause
- AWS — Using SQL Window Functions Instead of Join and GroupBy - https://docs.aws.amazon.com/es_es/prescriptive-guidance/latest/spark-tuning-glue-emr/using-sql-window-functions.html

---

## Summary Table: Window Function Core Components

| Component | Purpose | Key Detail |
|-----------|---------|------------|
| Window Function | Calculate across row sets, return per-row | Preserves row detail unlike `GROUP BY`  |
| `OVER` Clause | Define the window | Empty `OVER ()` = entire result set  |
| `PARTITION BY` | Divide into independent groups | Omit = single partition  |
| `ORDER BY` | Define logical order | Affects ranking and default frames  |
| `ROWS` / `RANGE` | Limit frame within partition | Requires `ORDER BY`  |
| Named Window | Reuse window specification | Defined in `WINDOW` clause  |

---

## References

- MySQL 8.0 Reference Manual — Window Function Concepts and Syntax - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/window-functions-usage.html
- Microsoft Learn — OVER Clause (Transact-SQL) - https://learn.microsoft.com/ga-ie/sql/t-sql/queries/select-over-clause-transact-sql
- PostgreSQL Documentation — Window Functions - https://git.postgresql.org/cgit/pgrpms.git/plain/rpm/redhat/10/postgresql/master/postgresql-10-A4.pdf
- Microsoft Learn — Use the OVER clause - https://learn.microsoft.com/lb-lu/training/modules/write-queries-that-use-window-functions/3-use-over-clause
- SAP HANA Cloud — PARTITION BY clause - https://help.sap.com/docs/SAP_HANA_DATA_LAKE_CN/73a183f2600c4b7890db9ffe2d09570e/a588f918af8b441a939c39e987597230.html
- Google Cloud — Window Function Calls (BigQuery) - https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/window-function-calls
- AWS — Using SQL Window Functions Instead of Join and GroupBy - https://docs.aws.amazon.com/es_es/prescriptive-guidance/latest/spark-tuning-glue-emr/using-sql-window-functions.html
- PostgreSQL Documentation — Window Functions (9.1) - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.1/postgresql-9.1-A4.pdf
- Microsoft Learn — OVER Clause (Transact-SQL, SQL Server 2005) - https://learn.microsoft.com/fr-fr/previous-versions/sql/sql-server-2005/ms189461(v=sql.90)
- Microsoft SQL Server 2012 Transact-SQL DML Reference - https://download.microsoft.com/download/0/F/B/0FBFAA46-2BFD-478F-8E56-7BF3C672DF9D/SQL%20Server%202012%20Transact-SQL%20DML%20Reference.pdf
- PostgreSQL Documentation — Window Functions (9.2) - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.2/postgresql-9.2-A4.pdf
- Patent US11455303 — Window Function Rewrite - https://patentimages.storage.googleapis.com/74/d1/46/e40723b600706f/US11455303.pdf
- Patent US20140214754A1 — Reporting Window Functions - http://patentimages.storage.googleapis.com/a7/66/62/77e78470390b3b/US20140214754A1.pdf