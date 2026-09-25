# SQL GROUP BY Clause: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The SQL `GROUP BY` clause is a query construct that combines rows with identical values in specified columns (or expressions) into single summary rows, enabling aggregate calculations to be performed on each group independently.

**Technical Definition:** Per the SQL standard (ISO/IEC 9075-2), the `<group by clause>` is defined as `GROUP BY <grouping element> [, ...]`, where each grouping element is an expression, a column reference, or an extension such as `ROLLUP`, `CUBE`, or `GROUPING SETS`. The clause partitions the result of the `FROM` and `WHERE` clauses into groups of rows that share the same values for the grouping expressions. Aggregate functions in the `SELECT` list are then computed across each group, producing one summary row per group. `NULL` values are considered equivalent for grouping purposes.

**Beginner-Friendly Explanation:** Imagine you have a spreadsheet of sales records, each row showing a product, a city, and a sale amount. The `GROUP BY` clause is like telling the spreadsheet: "Combine all rows that have the same product and city, and give me one summary row for each combination." Instead of seeing every individual sale, you see totals, averages, or counts for each group.

### Key Characteristics

- **Row-to-group collapse:** Multiple individual rows are condensed into a single summary row per distinct combination of grouping values.
- **Aggregate compatibility:** Works with aggregate functions (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) to compute summary statistics per group.
- **NULL equivalence:** `NULL` values are treated as equal to each other for grouping purposes.
- **Clause ordering:** Must appear after `WHERE` and before `HAVING` and `ORDER BY` in the textual order of a `SELECT` statement.
- **Dialect variations:** Functional dependency detection, `ROLLUP`, `CUBE`, and `GROUPING SETS` support vary across RDBMSs.
- **The SELECT rule:** Non-aggregated columns in the `SELECT` list must appear in the `GROUP BY` clause (or be functionally dependent on grouping columns).

### Prerequisites

- **Basic SQL knowledge:** Familiarity with `SELECT`, `FROM`, and `WHERE` clauses.
- **Aggregate function awareness:** Understanding of `COUNT`, `SUM`, `AVG`, `MIN`, and `MAX`.
- **Relational concepts:** Understanding of tables, rows, columns, and data types.
- **NULL awareness:** Recognition that `NULL` represents an unknown value and is grouped as equivalent to other `NULL` values.

### Related Programming Areas

- **Data Analysis and Business Intelligence:** `GROUP BY` powers summary reports, dashboards, and KPIs.
- **Application Development:** Backend services use `GROUP BY` for aggregated data retrieval.
- **Data Engineering (ETL):** Aggregation is a core transformation step in data pipelines.
- **Database Administration:** DBAs use `GROUP BY` for capacity planning and performance monitoring.
- **Machine Learning:** Feature engineering often begins with grouped aggregate summaries.

### Core Concepts / Features

1. Grouping Records
2. Single-Column Grouping
3. Multi-Column Grouping
4. Grouping Expressions
5. Aggregate Calculations
6. The "SELECT" Rule
7. `HAVING` Clause (Filtering Groups)
8. Performance and Indexing Considerations

---

## 1. Grouping Records

### Definitions

**Core Definition:** Grouping records is the process of combining individual rows that share identical values in specified columns into single summary rows, collapsing detail to reveal high-level trends.

**Technical Definition:** The `GROUP BY` clause partitions the result set of the `FROM` and `WHERE` clauses into subsets (groups) based on the values of one or more grouping expressions. Each group is then collapsed into a single row in the final result set. Aggregate functions in the `SELECT` list are evaluated per group rather than across the entire result set. Without a `GROUP BY` clause, aggregate functions report a single value for the entire result set; with `GROUP BY`, they report one value per group.

**Beginner-Friendly Explanation:** Think of a class attendance sheet with every student's name and their grade on every exam. Grouping records would collapse all of Alice's exam scores into one row showing her average, all of Bob's into another row, and so on. You lose the individual exam details but gain a clear summary.

### Purposes

- To collapse detailed rows into summary rows for high-level analysis.
- To reveal trends and patterns that are obscured by individual row details.
- To compute aggregate statistics (totals, averages, counts) per category.
- To reduce the volume of data returned by a query.
- To prepare data for reporting, dashboards, and decision-making.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT grouping_column, aggregate_function(column)
FROM table_name
[WHERE row_condition]
GROUP BY grouping_column
[HAVING aggregate_condition]
[ORDER BY ...];
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `GROUP BY` | Keyword introducing the grouping specification |
| `grouping_column` | Column(s) or expression(s) that define the groups |
| `aggregate_function(column)` | Aggregate computed per group |
| `HAVING` | Filters groups after aggregation |
| `ORDER BY` | Sorts the final grouped result |

**Syntax Rules:**

- `GROUP BY` must appear after `WHERE` (if present) and before `HAVING` and `ORDER BY`.
- The `GROUP BY` clause can reference input column names, output column names (aliases), or arbitrary expressions formed from input columns.
- `NULL` values are treated as equivalent for grouping purposes; all `NULL`s in a grouping column form one group.
- Aggregate functions cannot appear in the `GROUP BY` clause itself.

**Constraints and Limitations:**

- `GROUP BY` cannot reference `LOB` columns, nested tables, or varrays in Oracle.
- Scalar subquery expressions are not permitted in `GROUP BY` in Oracle.
- The `ALL` keyword in `GROUP BY` is deprecated in SQL Server and should be avoided.
- MySQL's `ONLY_FULL_GROUP_BY` mode (enabled by default) rejects queries where non-aggregated columns are not named in `GROUP BY` or functionally dependent on grouping columns.

### Annotated Complete Code Examples

**Example 1: Collapsing Sales Records**

```sql
-- Setup: Create a sample sales table
CREATE TABLE sales (
    sale_id    SERIAL PRIMARY KEY,
    product    VARCHAR(50),
    city       VARCHAR(50),
    amount     NUMERIC(10, 2)
);

INSERT INTO sales (product, city, amount) VALUES
    ('Laptop', 'Manila',   1200.00),
    ('Laptop', 'Cebu',     1100.00),
    ('Phone',  'Manila',    800.00),
    ('Phone',  'Cebu',      750.00),
    ('Laptop', 'Manila',   1300.00),
    ('Phone',  'Manila',    900.00);
```
```sql
-- Query: Collapse individual sales into product-city summaries
SELECT product,
       city,
       COUNT(*)    AS num_sales,
       SUM(amount) AS total_sales
FROM sales
GROUP BY product, city
ORDER BY product, city;
```

Expected Output:
 product | city   | num_sales | total_sales
---------|--------|-----------|-------------
 Laptop  | Cebu   |         1 |     1100.00
 Laptop  | Manila |         2 |     2500.00
 Phone   | Cebu   |         1 |      750.00
 Phone   | Manila |         2 |     1700.00


**Why this output occurs:** 
- The `GROUP BY product, city` clause partitions the six individual sales rows into four groups based on unique product-city combinations. 
- Each group is collapsed into one summary row. The `COUNT(*)` counts the number of sales in each group, and `SUM(amount)` totals the sales amount per group. The `ORDER BY` sorts the summary rows alphabetically.

### Real-World Cases

**Case 1: Monthly Sales Reporting**

A retail company runs `SELECT YEAR(order_date), MONTH(order_date), SUM(total) FROM orders GROUP BY YEAR(order_date), MONTH(order_date)` to produce monthly sales totals. The grouping collapses thousands of individual orders into twelve summary rows per year.

**Case 2: Website Analytics**

A web analytics platform uses `SELECT page_url, COUNT(*) AS visits, AVG(time_on_page) FROM page_views GROUP BY page_url` to summarise traffic and engagement per page.

### References

- PostgreSQL Documentation — GROUP BY Clause - https://www.postgresql.org/docs/current/sql-select.html
- MySQL 8.4 Reference Manual — MySQL Handling of GROUP BY - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/group-by-handling.html
- Microsoft SQL Server — GROUP BY (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-group-by-transact-sql
- Oracle Database SQL Language Reference — GROUP BY Clause - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SELECT.html

---

## 2. Single-Column Grouping

### Definitions

**Core Definition:** Single-column grouping uses exactly one column (or expression) in the `GROUP BY` clause, creating one group for each distinct value in that column.

**Technical Definition:** When `GROUP BY` contains a single grouping expression, the result set is partitioned into as many groups as there are distinct (non-`NULL` and `NULL` combined) values in that expression. Each group produces one summary row. All `NULL` values in the grouping column are treated as a single group.

**Beginner-Friendly Explanation:** Grouping by one column is like sorting a stack of invoices by customer name. All invoices for the same customer end up in the same pile, and you can then calculate a total for each customer.

### Purposes

- To summarise data by a single categorical dimension (e.g., country, department, category).
- To compute aggregates per distinct value of one column.
- To find the number of unique values in a column.
- To build simple summary reports.
- To introduce grouping concepts before moving to multi-column grouping.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT grouping_column, aggregate_function(column)
FROM table_name
GROUP BY grouping_column;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `grouping_column` | The single column whose distinct values define the groups |
| `aggregate_function(column)` | Aggregate computed per distinct value |

**Syntax Rules:**

- The grouping column must be a column from the `FROM` clause or an expression derived from source columns.
- The grouping column can be referenced by name, ordinal position, or alias (dialect-dependent).
- `NULL` values form a single group.
- All non-aggregated columns in the `SELECT` list must appear in the `GROUP BY` clause.

**Constraints and Limitations:**

- The grouping column should have a manageable number of distinct values; grouping by a high-cardinality column (e.g., primary key) produces one group per row, defeating the purpose of grouping.
- Performance can degrade if the grouping column is not indexed.
- In MySQL's `ONLY_FULL_GROUP_BY` mode, any non-aggregated column in `SELECT` must be functionally dependent on the grouping column.

### Annotated Complete Code Examples

**Example 1: Grouping by Department**

```sql
-- Setup
CREATE TABLE employees (
    id          SERIAL PRIMARY KEY,
    first_name  VARCHAR(50),
    department  VARCHAR(50),
    salary      NUMERIC(10, 2)
);

INSERT INTO employees (first_name, department, salary) VALUES
    ('Alice', 'Engineering', 95000.00),
    ('Bob',   'Marketing',   72000.00),
    ('Carol', 'Engineering', 105000.00),
    ('David', 'Sales',       68000.00),
    ('Eve',   'Engineering', 88000.00);
```

```sql
-- Query: Summary statistics per department
SELECT department,
       COUNT(*)    AS headcount,
       AVG(salary) AS avg_salary,
       SUM(salary) AS total_salary
FROM employees
GROUP BY department
ORDER BY department;
```

Expected Output:
 department  | headcount |     avg_salary     | total_salary
-------------|-----------|--------------------|--------------
 Engineering |         3 | 96000.000000000000 |    288000.00
 Marketing   |         1 | 72000.000000000000 |     72000.00
 Sales       |         1 | 68000.000000000000 |     68000.00


**Why this output occurs:** The `GROUP BY department` clause creates three groups: Engineering, Marketing, and Sales. For each group, `COUNT(*)` returns the number of employees, `AVG(salary)` computes the average salary, and `SUM(salary)` totals the salaries. Engineering has three employees; Marketing and Sales each have one.

**Example 2: Grouping with NULL Values**

```sql
-- Insert a row with NULL department
INSERT INTO employees (first_name, department, salary)
VALUES ('Frank', NULL, 75000.00);
```

```sql
-- Query: Group by department including NULL
SELECT department,
       COUNT(*) AS headcount
FROM employees
GROUP BY department
ORDER BY department NULLS LAST;

-- Expected Output:
--  department  | headcount
-- -------------+-----------
--  Engineering |         3
--  Marketing   |         1
--  Sales       |         1
--  (null)      |         1
```

**Why this output occurs:** The `NULL` department forms its own group, as `NULL` values are considered equivalent for grouping purposes. Frank's row is counted in the `(null)` group.

### Real-World Cases

**Case 1: Customer Country Distribution**

A global e-commerce platform runs `SELECT country, COUNT(*) AS customers FROM users GROUP BY country` to see how many customers are in each country. Each distinct country forms one group.

**Case 2: Product Category Revenue**

A retail analyst queries `SELECT category, SUM(revenue) FROM sales GROUP BY category` to determine total revenue per product category.

### References

- PostgreSQL Documentation — GROUP BY Clause - https://www.postgresql.org/docs/current/sql-select.html
- MySQL 8.4 Reference Manual — GROUP BY Modifiers - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/group-by-modifiers.html
- Microsoft SQL Server — GROUP BY (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-group-by-transact-sql

---

## 3. Multi-Column Grouping

### Definitions

**Core Definition:** Multi-column grouping uses two or more columns (or expressions) in the `GROUP BY` clause, creating a separate group for every unique combination of values across those columns.

**Technical Definition:** When `GROUP BY` contains multiple grouping expressions, the result set is partitioned into groups based on the distinct combinations (Cartesian product) of values from all grouping expressions. Each unique combination forms one group and produces one summary row. The order of columns in the `GROUP BY` clause affects the order of grouping but not the resulting groups themselves (though it may affect index usage and sort order).

**Beginner-Friendly Explanation:** Multi-column grouping is like organising a warehouse by both aisle and shelf. Each unique aisle-shelf combination is a separate location, and you can count how many items are in each combination.

### Purposes

- To summarise data by multiple dimensions simultaneously.
- To create nested or hierarchical summaries (e.g., country then city).
- To compute aggregates for each unique combination of categories.
- To analyse relationships between multiple categorical variables.
- To prepare data for cross-tabulation and pivot-style reports.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT column1, column2, aggregate_function(column3)
FROM table_name
GROUP BY column1, column2;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `column1, column2` | Multiple grouping columns defining the combinations |
| `aggregate_function(column3)` | Aggregate computed per unique combination |

**Syntax Rules:**

- Grouping columns are separated by commas.
- Each unique combination of values in the grouping columns forms one group.
- `NULL` values in any grouping column are treated as equivalent to other `NULL`s in that column.
- The order of columns in `GROUP BY` determines the nesting order of groups (for ordered output, use `ORDER BY`).
- All non-aggregated columns in `SELECT` must appear in `GROUP BY`.

**Constraints and Limitations:**

- The number of groups grows multiplicatively with the number of distinct values in each grouping column.
- Grouping by many columns with high cardinality can produce a very large result set.
- Multi-column grouping may require more memory and sort operations than single-column grouping.
- In some RDBMSs, composite columns can be grouped as a unit by enclosing them in parentheses.

### Annotated Complete Code Examples

**Example 1: Grouping by Product and City**

```sql
-- Setup
CREATE TABLE sales (
    sale_id  SERIAL PRIMARY KEY,
    product  VARCHAR(50),
    city     VARCHAR(50),
    amount   NUMERIC(10, 2)
);

INSERT INTO sales (product, city, amount) VALUES
    ('Laptop', 'Manila',   1200.00),
    ('Laptop', 'Cebu',     1100.00),
    ('Phone',  'Manila',    800.00),
    ('Phone',  'Cebu',      750.00),
    ('Laptop', 'Manila',   1300.00),
    ('Phone',  'Manila',    900.00),
    ('Tablet', 'Manila',    600.00),
    ('Tablet', 'Cebu',      550.00);
```

```sql
-- Query: Total sales per product per city
SELECT product,
       city,
       COUNT(*)    AS num_sales,
       SUM(amount) AS total_sales
FROM sales
GROUP BY product, city
ORDER BY product, city;

-- Expected Output:
--  product | city   | num_sales | total_sales
-- ---------+--------+-----------+-------------
--  Laptop  | Cebu   |         1 |     1100.00
--  Laptop  | Manila |         2 |     2500.00
--  Phone   | Cebu   |         1 |      750.00
--  Phone   | Manila |         2 |     1700.00
--  Tablet  | Cebu   |         1 |      550.00
--  Tablet  | Manila |         1 |      600.00
```

**Why this output occurs:** The `GROUP BY product, city` clause creates one group for each unique product-city combination. There are six such combinations (3 products × 2 cities), each producing one summary row. The aggregates are computed per combination.

**Example 2: Three-Column Grouping**

```sql
-- Add a sales_rep column for three-dimensional grouping
ALTER TABLE sales ADD COLUMN sales_rep VARCHAR(50);
UPDATE sales SET sales_rep = 'Alice' WHERE product = 'Laptop' AND city = 'Manila';
UPDATE sales SET sales_rep = 'Bob'   WHERE product = 'Laptop' AND city = 'Cebu';
UPDATE sales SET sales_rep = 'Carol' WHERE product = 'Phone';
UPDATE sales SET sales_rep = 'David' WHERE product = 'Tablet';
```

```sql
-- Query: Group by product, city, and sales_rep
SELECT product, city, sales_rep,
       SUM(amount) AS total_sales
FROM sales
GROUP BY product, city, sales_rep
ORDER BY product, city, sales_rep;

-- Expected Output:
--  product | city   | sales_rep | total_sales
-- ---------+--------+-----------+-------------
--  Laptop  | Cebu   | Bob       |     1100.00
--  Laptop  | Manila | Alice     |     2500.00
--  Phone   | Cebu   | Carol     |      750.00
--  Phone   | Manila | Carol     |     1700.00
--  Tablet  | Cebu   | David     |      550.00
--  Tablet  | Manila | David     |      600.00
```

**Why this output occurs:** Adding `sales_rep` to the `GROUP BY` clause creates finer-grained groups. The Laptop-Manila combination was previously one group; now it is split by sales representative (though in this example there is only one rep per combination). The result set still has six rows because the rep assignment aligns with the product-city combinations.

### Real-World Cases

**Case 1: Regional Product Performance**

A retail chain analyses sales by `region` and `product_category` to determine which products perform best in which regions. The multi-column grouping reveals insights that single-column grouping cannot.

**Case 2: Employee Salary Analysis**

An HR system groups employees by `department` and `job_title` to compare salary ranges for the same role across different departments.

### References

- Microsoft Learn — GROUP BY Components (Multi-Column Grouping) - https://learn.microsoft.com/en-us/previous-versions/sql/sql-server-2005/ms190186(v=sql.90)
- PostgreSQL Documentation — GROUP BY Clause - https://www.postgresql.org/docs/current/sql-select.html
- Oracle Database SQL Language Reference — GROUP BY Clause - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SELECT.html

---

## 4. Grouping Expressions

### Definitions

**Core Definition:** Grouping by an expression uses a calculation, function, or transformation of one or more columns as the grouping criterion, rather than a raw column value.

**Technical Definition:** A `GROUP BY` item can be an arbitrary expression formed from input-column values, provided it does not include aggregate functions. For example, `GROUP BY YEAR(order_date)` groups rows by the year extracted from a date column. The expression is evaluated for each row, and rows with the same result are grouped together.

**Beginner-Friendly Explanation:** Instead of grouping by a column's exact value, you can group by a computed value. For example, instead of grouping by exact dates, you can group by the year part of each date.

### Purposes

- To group data by a derived attribute (e.g., year from a date, first letter of a name).
- To normalise data before grouping (e.g., uppercase a string before grouping).
- To group by calculated ranges or buckets.
- To analyse temporal patterns (e.g., monthly, quarterly, yearly).
- To avoid adding redundant columns to the table schema.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT expression AS alias, aggregate_function(column)
FROM table_name
GROUP BY expression;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `expression` | A calculation, function call, or transformation |
| `alias` | Output column name for the grouped expression |

**Syntax Rules:**

- The expression cannot contain aggregate functions.
- The expression can reference one or more columns from the `FROM` clause.
- The expression must be deterministic (return the same result for the same input).
- The output alias can be referenced in `ORDER BY` but generally not in `GROUP BY` (unless the dialect allows it).
- Functions like `YEAR()`, `MONTH()`, `UPPER()`, `LOWER()`, `DATE_TRUNC()`, and `SUBSTRING()` are commonly used in grouping expressions.

**Constraints and Limitations:**

- Not all functions are valid in `GROUP BY`; scalar subqueries are not permitted in Oracle.
- Grouping by a non-deterministic function (e.g., `RANDOM()`) produces unpredictable results.
- Expression-based grouping may prevent index usage, as the index is on the raw column, not the computed expression.
- The expression must return at most one atomic value per row.

### Annotated Complete Code Examples

**Example 1: Grouping by Year from a Date**

```sql
-- Setup: Create an orders table with dates
CREATE TABLE orders (
    order_id   SERIAL PRIMARY KEY,
    order_date DATE,
    total      NUMERIC(10, 2)
);

INSERT INTO orders (order_date, total) VALUES
    ('2024-01-15', 100.00),
    ('2024-03-22', 200.00),
    ('2024-06-10', 150.00),
    ('2025-02-05', 300.00),
    ('2025-04-18', 250.00),
    ('2025-07-30', 400.00);
```

```sql
-- Query: Total sales per year
SELECT EXTRACT(YEAR FROM order_date) AS order_year,
       COUNT(*)    AS num_orders,
       SUM(total)  AS total_sales
FROM orders
GROUP BY EXTRACT(YEAR FROM order_date)
ORDER BY order_year;

-- Expected Output:
--  order_year | num_orders | total_sales
-- ------------+------------+-------------
--        2024 |          3 |      450.00
--        2025 |          3 |      950.00
```

**Why this output occurs:** The expression `EXTRACT(YEAR FROM order_date)` is evaluated for each row, producing either 2024 or 2025. Rows with the same year are grouped together, and the aggregates are computed per year. The `ORDER BY order_year` sorts the summary rows chronologically.

**Example 2: Grouping by a Computed Category**

```sql
-- Setup: Employees with salaries
CREATE TABLE employees (
    emp_id     SERIAL PRIMARY KEY,
    first_name VARCHAR(50),
    salary     NUMERIC(10, 2)
);

INSERT INTO employees (first_name, salary) VALUES
    ('Alice', 95000.00),
    ('Bob', 72000.00),
    ('Carol', 105000.00),
    ('David', 68000.00),
    ('Eve', 88000.00),
    ('Frank', 55000.00);
```

```sql
-- Query: Group by salary band (using a CASE expression)
SELECT CASE
           WHEN salary >= 100000 THEN 'High'
           WHEN salary >= 80000  THEN 'Medium'
           ELSE 'Low'
       END AS salary_band,
       COUNT(*)    AS headcount,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY CASE
             WHEN salary >= 100000 THEN 'High'
             WHEN salary >= 80000  THEN 'Medium'
             ELSE 'Low'
         END
ORDER BY salary_band;

-- Expected Output:
--  salary_band | headcount |     avg_salary
-- -------------+-----------+--------------------
--  High        |         1 | 105000.000000000000
--  Low         |         2 |  61500.000000000000
--  Medium      |         3 |  85000.000000000000
```

**Why this output occurs:** The `CASE` expression evaluates each row's salary and assigns a band (`High`, `Medium`, or `Low`). The `GROUP BY` clause groups rows by this computed band. The aggregates are computed per band. The `ORDER BY salary_band` sorts the result alphabetically.

### Real-World Cases

**Case 1: Financial Year Reporting**

A finance team groups transactions by fiscal year using `GROUP BY EXTRACT(YEAR FROM transaction_date) - CASE WHEN EXTRACT(MONTH FROM transaction_date) < 4 THEN 1 ELSE 0 END` to align with a non-calendar fiscal year.

**Case 2: Data Normalisation**

A marketing team groups customer emails by domain using `GROUP BY SUBSTRING(email FROM POSITION('@' IN email) + 1)` to analyse which email providers are most common.

### References

- Microsoft Learn — Grouping by Expressions - https://learn.microsoft.com/en-us/previous-versions/sql/sql-server-2005/ms190186(v=sql.90)
- PostgreSQL Documentation — Value Expressions - https://www.postgresql.org/docs/current/sql-expressions.html
- Oracle Database SQL Language Reference — GROUP BY Clause - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SELECT.html

---

## 5. Aggregate Calculations

### Definitions

**Core Definition:** Aggregate calculations are mathematical functions (such as `COUNT`, `SUM`, `AVG`, `MIN`, and `MAX`) that operate on a set of rows within each group defined by `GROUP BY`, returning a single summary value per group.

**Technical Definition:** Aggregate functions are computed across all rows making up each group, producing a separate value for each group (whereas without `GROUP BY`, an aggregate produces a single value computed across all selected rows). The aggregate function is applied to the non-grouped columns or expressions.

**Beginner-Friendly Explanation:** Aggregate calculations are like having a calculator for each group. After you've grouped your data, you can ask "How many rows are in this group?", "What's the total?", "What's the average?", etc., and the database computes one answer per group.

### Purposes

- To compute summary statistics (counts, totals, averages, extremes) per group.
- To quantify group characteristics for analysis.
- To compare groups against each other.
- To support decision-making with aggregated metrics.
- To produce reports with key performance indicators (KPIs).

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT grouping_column, aggregate_function(column)
FROM table_name
GROUP BY grouping_column;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `aggregate_function` | `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` |
| `column` | The column or expression to aggregate |

**Syntax Rules:**

- Aggregate functions ignore `NULL` values (except `COUNT(*)`).
- Aggregates can be nested (e.g., `MAX(AVG(salary))` in some dialects).
- Aggregates can appear in the `SELECT` list, `HAVING` clause, and `ORDER BY` clause.
- Aggregates cannot appear in the `WHERE` clause or in the `GROUP BY` clause itself.

**Constraints and Limitations:**

- `SUM` and `AVG` require numeric inputs.
- `MIN` and `MAX` work with numeric, string, and date/time types.
- `COUNT(*)` counts all rows; `COUNT(column)` counts non-`NULL` values.
- Aggregate functions return `NULL` for empty sets (except `COUNT`, which returns `0`).

### Annotated Complete Code Examples

**Example 1: Comprehensive Aggregate Summary per Group**

```sql
-- Setup
CREATE TABLE products (
    product_id   SERIAL PRIMARY KEY,
    category     VARCHAR(50),
    price        NUMERIC(10, 2),
    stock        INTEGER
);

INSERT INTO products (category, price, stock) VALUES
    ('Electronics', 1200.00, 10),
    ('Electronics',  800.00, 25),
    ('Electronics', 1500.00,  5),
    ('Clothing',     50.00, 100),
    ('Clothing',     75.00,  80),
    ('Clothing',     60.00, 120),
    ('Books',        20.00, 200),
    ('Books',        35.00, 150);

-- Query: Aggregate statistics per category
SELECT category,
       COUNT(*)        AS num_products,
       SUM(price)      AS total_price,
       AVG(price)      AS avg_price,
       MIN(price)      AS min_price,
       MAX(price)      AS max_price,
       SUM(price * stock) AS inventory_value
FROM products
GROUP BY category
ORDER BY category;

-- Expected Output:
--   category    | num_products | total_price |     avg_price     | min_price | max_price | inventory_value
-- --------------+--------------+-------------+-------------------+-----------+-----------+-----------------
--  Books        |            2 |       55.00 | 27.500000000000000 |     20.00 |     35.00 |         7250.00
--  Clothing     |            3 |      185.00 | 61.666666666666667 |     50.00 |     75.00 |        15000.00
--  Electronics  |            3 |     3500.00 |1166.666666666666667 |    800.00 |   1500.00 |         14500.00
```

**Why this output occurs:** The `GROUP BY category` clause creates three groups. For each group, six different aggregates are computed: count of products, sum of prices, average price, minimum price, maximum price, and total inventory value (`price * stock` summed). The result provides a complete statistical profile of each category.

**Example 2: COUNT(*) vs COUNT(column)**

```sql
-- Add a row with NULL price
INSERT INTO products (category, price, stock)
VALUES ('Electronics', NULL, 15);

-- Query: Demonstrate COUNT(*) vs COUNT(price)
SELECT category,
       COUNT(*)     AS total_rows,
       COUNT(price) AS rows_with_price,
       AVG(price)   AS avg_price
FROM products
GROUP BY category
ORDER BY category;

-- Expected Output:
--   category    | total_rows | rows_with_price |     avg_price
-- --------------+------------+-----------------+-------------------
--  Books        |          2 |               2 | 27.500000000000000
--  Clothing     |          3 |               3 | 61.666666666666667
--  Electronics  |          4 |               3 |1166.666666666666667
```

**Why this output occurs:** For Electronics, there are now four rows, but one has a `NULL` price. `COUNT(*)` returns 4 (all rows), while `COUNT(price)` returns 3 (rows where price is not `NULL`). `AVG(price)` divides the sum of non-`NULL` prices by 3 (the count of non-`NULL` values), not by 4. This demonstrates that `COUNT(*)` and `COUNT(column)` behave differently with `NULL` values.

### Real-World Cases

**Case 1: Financial KPIs**

A CFO's dashboard uses `GROUP BY` with multiple aggregates to display revenue, cost, profit, and profit margin per business unit. Each unit is a group, and the aggregates provide the KPIs.

**Case 2: Inventory Management**

A warehouse system runs `SELECT warehouse_id, SUM(quantity), AVG(unit_cost), MAX(last_restock) FROM inventory GROUP BY warehouse_id` to summarise stock levels per warehouse.

### References

- PostgreSQL Documentation — Aggregate Functions - https://www.postgresql.org/docs/current/functions-aggregate.html
- MySQL 8.4 Reference Manual — Aggregate Functions - https://dev.mysql.com/doc/refman/8.4/en/aggregate-functions.html
- Microsoft SQL Server — Aggregate Functions - https://learn.microsoft.com/en-us/sql/t-sql/functions/aggregate-functions-transact-sql
- Oracle Database SQL Language Reference — Aggregate Functions - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Aggregate-Functions.html

---

## 6. The "SELECT" Rule

### Definitions

**Core Definition:** The "SELECT rule" (also known as the functional dependency rule) requires that any column in the `SELECT` list that is not part of an aggregate function must either appear in the `GROUP BY` clause or be functionally dependent on the grouping columns.

**Technical Definition:** When `GROUP BY` is present, it is not valid for `SELECT` list expressions to refer to ungrouped columns except within aggregate functions or when the ungrouped column is functionally dependent on the grouped columns (i.e., the grouping columns uniquely determine the ungrouped column's value). This rule ensures that each output column has exactly one well-defined value per group.

**Beginner-Friendly Explanation:** The rule means: "If you're grouping your data, every column you show must either be a group label or a calculated summary. You can't show a column that varies within a group, because the database wouldn't know which value to pick."

### Purposes

- To ensure query results are deterministic and unambiguous.
- To prevent errors where the database arbitrarily picks a value from a group.
- To enforce the logical integrity of grouped queries.
- To comply with the SQL standard and avoid dialect-specific exceptions.
- To guide query design toward meaningful summaries.

### Syntax Rules and Structure

**Correct Form:**

```sql
SELECT grouping_column, aggregate_function(column)
FROM table_name
GROUP BY grouping_column;
```

**Incorrect Form (Violates the Rule):**

```sql
SELECT grouping_column, non_grouped_column, aggregate_function(column)
FROM table_name
GROUP BY grouping_column;
-- non_grouped_column is neither in GROUP BY nor aggregated → ERROR
```

**Component Breakdown:**

| Rule Component | Requirement |
|---------------|-------------|
| Grouped columns | Must appear in `GROUP BY` |
| Aggregated columns | Must be wrapped in an aggregate function |
| Functionally dependent columns | May appear if determined by grouping columns |

**Syntax Rules:**

- Every non-aggregated column in `SELECT` must be in `GROUP BY` or functionally dependent on a grouping column.
- This rule applies to the `SELECT` list, `HAVING` condition, and `ORDER BY` list.
- Functional dependency means: if column A is grouped, and column B is uniquely determined by A (e.g., A is a primary key), then B can appear in `SELECT` without being in `GROUP BY`.

**Constraints and Limitations:**

- MySQL's `ONLY_FULL_GROUP_BY` mode enforces this rule strictly by default.
- PostgreSQL enforces the rule by default.
- SQL Server requires all non-aggregated columns to be in `GROUP BY`.
- Oracle (prior to 23ai) requires all non-aggregated columns to be in `GROUP BY`.
- Older MySQL versions (before 5.7) did not enforce this rule, leading to unpredictable results.
- Some dialects allow functional dependency detection, which can relax the rule in specific cases.

### Annotated Complete Code Examples

**Example 1: Correct Query (Rule Satisfied)**

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
    ('Eve',   'Engineering', 88000.00);

-- Correct: department is in GROUP BY, salary is aggregated
SELECT department,
       COUNT(*)    AS headcount,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY department;

-- Expected Output:
--  department  | headcount |     avg_salary
-- -------------+-----------+--------------------
--  Engineering |         3 | 96000.000000000000
--  Marketing   |         1 | 72000.000000000000
--  Sales       |         1 | 68000.000000000000
```

**Why this output occurs:** The `SELECT` list contains `department` (which is in `GROUP BY`), `COUNT(*)` (an aggregate), and `AVG(salary)` (an aggregate). Every non-aggregated column is a grouping column, satisfying the rule.

**Example 2: Incorrect Query (Rule Violated)**

```sql
-- Incorrect: first_name is not in GROUP BY and not aggregated
SELECT department,
       first_name,   -- VIOLATION: not in GROUP BY, not aggregated
       AVG(salary) AS avg_salary
FROM employees
GROUP BY department;

-- Expected Error (PostgreSQL / MySQL with ONLY_FULL_GROUP_BY):
-- ERROR: column "employees.first_name" must appear in the GROUP BY clause
--        or be used in an aggregate function
```

**Why this error occurs:** The `first_name` column is neither in the `GROUP BY` clause nor wrapped in an aggregate function. Since each department has multiple employees with different first names, the database cannot determine which first name to return for the group. The rule prevents this ambiguity.

**Correct Approach:**

```sql
-- Option 1: Add first_name to GROUP BY (creates more groups)
SELECT department, first_name, AVG(salary) AS avg_salary
FROM employees
GROUP BY department, first_name;

-- Option 2: Use an aggregate on first_name
SELECT department,
       MIN(first_name) AS example_name,
       AVG(salary)     AS avg_salary
FROM employees
GROUP BY department;
```

### Real-World Cases

**Case 1: Preventing Ambiguous Reports**

A report developer writes `SELECT department, employee_name, AVG(salary) FROM employees GROUP BY department`. The query fails because `employee_name` is not grouped or aggregated. The developer must either group by `employee_name` (producing more rows) or remove it from the `SELECT` list.

**Case 2: Functional Dependency Exception**

In a query joining `orders` and `customers` where `customer_id` is the primary key of `customers`, grouping by `customer_id` makes `customer_name` functionally dependent on `customer_id`. Some databases allow `SELECT customer_id, customer_name, SUM(amount) FROM orders JOIN customers USING (customer_id) GROUP BY customer_id` because `customer_name` is uniquely determined by the grouped `customer_id`.

### References

- PostgreSQL Documentation — GROUP BY Clause (Select Rule) - https://www.postgresql.org/docs/current/sql-select.html
- MySQL 8.4 Reference Manual — MySQL Handling of GROUP BY - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/group-by-handling.html
- Microsoft Learn — GROUP BY Components (SELECT Rule) - https://learn.microsoft.com/en-us/previous-versions/sql/sql-server-2005/ms190186(v=sql.90)
- Amazon Redshift — GROUP BY Clause (SELECT Rule) - https://docs.aws.amazon.com/redshift/latest/dg/r_GROUP_BY_clause.html

---

## 7. HAVING Clause (Filtering Groups)

### Definitions

**Core Definition:** The `HAVING` clause filters groups produced by `GROUP BY` based on aggregate conditions, keeping only groups that satisfy the specified condition.

**Technical Definition:** `HAVING` specifies a grouped table derived by the elimination of group rows that do not satisfy a Boolean expression. It is evaluated after `GROUP BY` and before `SELECT` and `ORDER BY`. Unlike `WHERE`, which filters individual rows before grouping, `HAVING` filters groups after aggregation and can reference aggregate functions.

**Beginner-Friendly Explanation:** `WHERE` filters rows before they are grouped; `HAVING` filters groups after they have been formed. Use `HAVING` when you want to keep only groups that meet a certain aggregate condition, such as "departments with more than 5 employees."

### Purposes

- To filter groups based on aggregate results.
- To keep only groups that meet a minimum threshold.
- To exclude groups with insufficient data.
- To focus analysis on significant or relevant groups.
- To combine row-level and group-level filtering in a single query.

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
| `HAVING` | Keyword introducing the group filter |
| `aggregate_condition` | Boolean expression involving aggregates |

**Syntax Rules:**

- `HAVING` must appear after `GROUP BY` and before `ORDER BY`.
- `HAVING` can reference aggregate functions (unlike `WHERE`).
- `HAVING` can reference grouping columns.
- `HAVING` cannot reference columns that are neither grouped nor aggregated (same rule as `SELECT`).
- `WHERE` and `HAVING` can be used together in the same query.

**Constraints and Limitations:**

- `HAVING` without `GROUP BY` is allowed in some dialects but is non-standard and behaves inconsistently.
- `HAVING` conditions are evaluated after aggregation, so they cannot use indexes on individual rows.
- Filtering with `WHERE` before `GROUP BY` is generally more efficient than filtering with `HAVING` after `GROUP BY`.

### Annotated Complete Code Examples

**Example 1: HAVING with Aggregate Condition**

```sql
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
--  Engineering |         3 | 96000.000000000000
```

**Why this output occurs:** The `GROUP BY department` creates three groups. The `HAVING COUNT(*) > 2` clause keeps only groups with more than 2 rows. Engineering has 3 employees and passes; Marketing (1) and Sales (1) fail and are excluded.

**Example 2: WHERE and HAVING Together**

```sql
-- Query: Average salary per department, considering only employees
-- earning more than 70,000, keeping only departments with avg > 90,000
SELECT department,
       COUNT(*)    AS headcount,
       AVG(salary) AS avg_salary
FROM employees
WHERE salary > 70000
GROUP BY department
HAVING AVG(salary) > 90000;

-- Expected Output:
--  department  | headcount |     avg_salary
-- -------------+-----------+--------------------
--  Engineering |         3 | 96000.000000000000
```

**Why this output occurs:** The `WHERE salary > 70000` clause filters out David (68,000) before grouping. The `GROUP BY` then groups the remaining rows. The `HAVING AVG(salary) > 90000` keeps only Engineering (average 96,000), excluding Marketing (72,000) and Sales (none remaining).

### Real-World Cases

**Case 1: Sales Territory Performance**

A sales manager runs `SELECT territory, SUM(amount) FROM sales GROUP BY territory HAVING SUM(amount) > 1000000` to identify only high-performing territories that exceeded $1 million in sales.

**Case 2: Customer Segmentation**

A marketing analyst uses `SELECT country, COUNT(*) FROM customers GROUP BY country HAVING COUNT(*) >= 100` to focus on countries with at least 100 customers, excluding small markets.

### References

- PostgreSQL Documentation — HAVING Clause - https://www.postgresql.org/docs/current/sql-select.html
- MySQL 8.4 Reference Manual — HAVING Clause - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/select.html
- Microsoft SQL Server — HAVING (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-having-transact-sql
- Oracle Database SQL Language Reference — HAVING Clause - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SELECT.html

---

## 8. Performance and Indexing Considerations

### Definitions

**Core Definition:** Query performance in the context of `GROUP BY` refers to how efficiently the database engine can partition, aggregate, and return grouped results, heavily influenced by the availability and use of indexes.

**Technical Definition:** The database may use several strategies to execute `GROUP BY`: sorting the rows and then scanning sequentially (sort-based grouping), using a hash table to accumulate groups (hash-based grouping), or using an index that already provides the rows in grouping order (index-based grouping). The chosen strategy depends on the query, available indexes, table statistics, and the RDBMS optimizer.

**Beginner-Friendly Explanation:** An index on the grouping column is like having a pre-sorted phone book — it helps the database find and group rows much faster. Without an index, the database may have to sort the entire table before grouping.

### Purposes

- To minimise the time required to produce grouped results.
- To reduce CPU, memory, and I/O consumption during grouping.
- To enable scaling as data volume grows.
- To improve application responsiveness for reports and dashboards.
- To inform index design decisions.

### Syntax Rules and Structure

**Creating an Index for Grouping:**

```sql
CREATE INDEX idx_table_grouping_column ON table_name (grouping_column);
```

**Multi-Column Grouping Index:**

```sql
CREATE INDEX idx_table_multi_group ON table_name (column1, column2);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `CREATE INDEX` | Creates an index to speed up queries |
| `(grouping_column)` | The column(s) used in `GROUP BY` |
| `EXPLAIN` | Shows the execution plan |

**Syntax Rules:**

- An index on the grouping column(s) can allow the optimizer to read rows in grouping order, avoiding a sort.
- Composite indexes can support multi-column grouping if the leading columns match the `GROUP BY` columns.
- `EXPLAIN` (PostgreSQL, MySQL) or `EXPLAIN PLAN` (Oracle, SQL Server) shows whether an index is used.
- Covering indexes (indexes that include all columns referenced by the query) can further improve performance.

**Constraints and Limitations:**

- Indexes consume disk space and slow down write operations.
- Grouping by expressions (e.g., `EXTRACT(YEAR FROM date)`) cannot use a standard index on the raw column.
- Hash-based grouping may be faster than sort-based grouping for large datasets with many groups.
- The optimizer's choice depends on statistics; outdated statistics can lead to poor plans.
- Too many indexes can degrade write performance and increase maintenance overhead.

### Annotated Complete Code Examples

**Example 1: Index Usage for GROUP BY**

```sql
-- Setup: Create a table with an index on the grouping column
CREATE TABLE sales (
    sale_id  SERIAL PRIMARY KEY,
    product  VARCHAR(50),
    amount   NUMERIC(10, 2)
);

CREATE INDEX idx_sales_product ON sales (product);

-- Insert sample data
INSERT INTO sales (product, amount) VALUES
    ('Laptop', 1200.00), ('Phone', 800.00), ('Laptop', 1300.00),
    ('Tablet', 600.00), ('Phone', 900.00), ('Tablet', 550.00);

-- Query with EXPLAIN
EXPLAIN SELECT product, SUM(amount)
FROM sales
GROUP BY product;

-- Expected Output (PostgreSQL):
--  QUERY PLAN
-- ----------------------------------------------------------------------
--  GroupAggregate  (cost=0.15..1.45 rows=3 width=36)
--    Group Key: product
--    ->  Index Scan using idx_sales_product on sales
--          (cost=0.15..1.30 rows=6 width=36)
```

**Why this plan occurs:** The index `idx_sales_product` provides rows in `product` order. The `GroupAggregate` operation can group adjacent rows without a separate sort step. The `Index Scan` reads the indexed rows efficiently.

**Example 2: Expression Grouping Without Index**

```sql
-- Query: Group by a computed expression (no index on expression)
EXPLAIN SELECT EXTRACT(YEAR FROM order_date), SUM(total)
FROM orders
GROUP BY EXTRACT(YEAR FROM order_date);

-- Expected Output (PostgreSQL):
--  QUERY PLAN
-- ----------------------------------------------------------------------
--  HashAggregate  (cost=1.10..1.12 rows=2 width=40)
--    Group Key: EXTRACT(year FROM order_date)
--    ->  Seq Scan on orders  (cost=0.00..1.07 rows=6 width=36)
```

**Why this plan occurs:** The expression `EXTRACT(YEAR FROM order_date)` is computed for each row during a sequential scan. A `HashAggregate` operation accumulates groups in memory. No index is used because there is no index on the expression. This plan is efficient for small datasets but may degrade for large ones.

### Real-World Cases

**Case 1: Data Warehouse Aggregation**

A data warehouse creates composite indexes on `(date_key, product_key)` to accelerate `GROUP BY date_key, product_key` queries used in daily sales reports.

**Case 2: Dashboard Performance**

A dashboard query groups millions of rows by `region`. Adding an index on `region` reduces the query time from seconds to milliseconds because the optimizer can use an index scan with grouping instead of a full table sort.

### References

- PostgreSQL Documentation — Indexes and GROUP BY - https://www.postgresql.org/docs/current/indexes.html
- MySQL 8.4 Reference Manual — GROUP BY Optimization - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/group-by-optimization.html
- Microsoft SQL Server — Query Tuning with Indexes - https://learn.microsoft.com/en-us/sql/relational-databases/indexes/
- Oracle Database SQL Tuning Guide — GROUP BY Optimization - https://docs.oracle.com/en/database/oracle/oracle-database/19/tgsql/

---

## Summary Table of GROUP BY Features

| Feature | Purpose | Key Limitation |
|---------|---------|----------------|
| Grouping Records | Collapse rows into summaries | Loses individual row detail |
| Single-Column Grouping | Summarise by one dimension | Limited analytical depth |
| Multi-Column Grouping | Summarise by multiple dimensions | Result set grows multiplicatively |
| Grouping Expressions | Group by derived values | Cannot use indexes on raw columns |
| Aggregate Calculations | Compute summary statistics | `SUM`/`AVG` require numeric inputs |
| SELECT Rule | Ensure deterministic results | Strict in most dialects |
| HAVING Clause | Filter groups by aggregate conditions | Evaluated after aggregation (less efficient than WHERE) |
| Performance & Indexing | Speed up grouping operations | Indexes slow writes; expression grouping prevents index use |

---

## GROUP BY Execution Order

The logical processing order of a `SELECT` statement with `GROUP BY` is:

1. `FROM` / `JOIN` — Assemble source rows
2. `WHERE` — Filter individual rows
3. `GROUP BY` — Partition rows into groups
4. `HAVING` — Filter groups
5. `SELECT` — Evaluate expressions and project columns
6. `DISTINCT` — Remove duplicate rows
7. `ORDER BY` — Sort the result set
8. `LIMIT` / `OFFSET` — Restrict the number of rows

This order explains why `WHERE` cannot reference aggregates (they are computed in step 3), why `HAVING` can reference aggregates (step 4 is after step 3), and why column aliases from `SELECT` cannot be used in `WHERE` or `GROUP BY` (the `SELECT` step occurs after them).

---

## References

- PostgreSQL Documentation — SELECT (GROUP BY Clause) - https://www.postgresql.org/docs/current/sql-select.html
- PostgreSQL Documentation — Aggregate Functions - https://www.postgresql.org/docs/current/functions-aggregate.html
- MySQL 8.4 Reference Manual — MySQL Handling of GROUP BY - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/group-by-handling.html
- MySQL 8.4 Reference Manual — GROUP BY Modifiers - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/group-by-modifiers.html
- Microsoft SQL Server — GROUP BY (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-group-by-transact-sql
- Microsoft Learn — GROUP BY Components (Single/Multi-Column, Expressions, SELECT Rule) - https://learn.microsoft.com/en-us/previous-versions/sql/sql-server-2005/ms190186(v=sql.90)
- Oracle Database SQL Language Reference — GROUP BY Clause - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SELECT.html
- Oracle Database SQL Language Reference — Aggregate Functions - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Aggregate-Functions.html
- Amazon Redshift — GROUP BY Clause - https://docs.aws.amazon.com/redshift/latest/dg/r_GROUP_BY_clause.html
- ISO/IEC 9075-2:2003 — SQL Standard (GROUP BY Specification) - https://jtc1sc32.org/doc/N0251-0300/32N0254.pdf