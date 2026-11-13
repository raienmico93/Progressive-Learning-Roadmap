# SQL Aggregate Functions: COUNT(), SUM(), AVG(), MIN(), MAX() — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL aggregate functions are built-in functions that perform a calculation on a set of rows and return a single scalar value, summarising data across multiple records.

**Technical Definition:** Per the SQL standard (ISO/IEC 9075-2), an aggregate function is a function that operates on a collection of values (typically a column across multiple rows) and returns a single value. Aggregate functions are permitted in the `SELECT` list, `ORDER BY`, and `HAVING` clauses, and are commonly used with `GROUP BY` to compute summaries per group. They are not permitted in the `WHERE` clause because `WHERE` is evaluated before aggregation.

**Beginner-Friendly Explanation:** Aggregate functions are like a calculator for your data. Instead of looking at every row individually, they let you ask questions like "How many rows are there?", "What's the total?", "What's the average?", "What's the smallest value?", or "What's the largest value?" — and they give you one answer.

### Key Characteristics

- **Multi-row to single-value:** Each aggregate function takes many input values and returns one result.
- **NULL-ignoring (except COUNT(*)):** Most aggregate functions ignore `NULL` values in their calculations; `COUNT(*)` counts all rows including those with `NULL`.
- **GROUP BY compatible:** Aggregates can be computed per group when combined with `GROUP BY`.
- **HAVING filtering:** Aggregate results can be filtered using the `HAVING` clause, which is evaluated after grouping.
- **DISTINCT support:** Most aggregate functions accept a `DISTINCT` keyword to eliminate duplicate values before computation.
- **Dialect variations:** Return types, NULL handling edge cases, and temporal value support vary across RDBMSs.

### Prerequisites

- **Basic SQL knowledge:** Familiarity with `SELECT`, `FROM`, and `WHERE` clauses.
- **Relational concepts:** Understanding of tables, rows, and columns.
- **NULL awareness:** Recognition that `NULL` represents an unknown value and is handled specially by aggregates.
- **Data type awareness:** Understanding numeric, string, and date/time types, as different aggregates apply to different types.

### Related Programming Areas

- **Data Analysis and Business Intelligence:** Aggregates power dashboards, KPIs, and summary reports.
- **Application Development:** Backend services use aggregates for counts, totals, and averages.
- **Data Engineering (ETL):** Aggregation is a core transformation step in data pipelines.
- **Database Administration:** DBAs use aggregates for capacity planning and performance monitoring.
- **Machine Learning:** Feature engineering often begins with aggregate summaries of raw data.

### Core Concepts / Features

1. `COUNT()`
2. `SUM()`
3. `AVG()`
4. `MIN()`
5. `MAX()`

---

## 1. COUNT()

### Definitions

**Core Definition:** `COUNT()` returns the number of rows or non-NULL values in a set of rows.

**Technical Definition:** `COUNT(*)` counts all rows in the result set, including rows containing `NULL` values. `COUNT(expression)` counts only rows where the expression evaluates to a non-`NULL` value. `COUNT(DISTINCT expression)` counts the number of distinct non-`NULL` values. `COUNT` never returns `NULL`; it returns `0` when no rows match.

**Beginner-Friendly Explanation:** `COUNT` answers the question "How many?" Use `COUNT(*)` to count all rows, or `COUNT(column)` to count only rows where that column has a value.

### Purposes

- To count the total number of rows in a table or result set.
- To count non-`NULL` values in a specific column.
- To count distinct values in a column.
- To measure group sizes when used with `GROUP BY`.
- To validate data completeness by comparing `COUNT(*)` with `COUNT(column)`.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
COUNT ( { [ [ ALL | DISTINCT ] expression ] | * } )
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `*` | Counts all rows, including those with `NULL` values |
| `ALL` | Counts all non-`NULL` values (default) |
| `DISTINCT` | Counts only distinct non-`NULL` values |
| `expression` | A column name or expression to evaluate |

**Syntax Rules:**

- `COUNT(*)` ignores no rows; it counts every row in the set.
- `COUNT(column)` ignores rows where `column` is `NULL`.
- `COUNT(DISTINCT column)` ignores `NULL` values and counts only unique non-`NULL` values.
- `ALL` is the default and is rarely written explicitly.
- `COUNT` can be used with `GROUP BY`, `HAVING`, and `ORDER BY`.

**Constraints and Limitations:**

- `COUNT(*)` cannot be combined with `DISTINCT`.
- `COUNT(DISTINCT expression)` evaluates the expression for each row, which can be slower on large datasets.
- In MySQL, `COUNT(DISTINCT ...)` with multiple arguments is supported, but this is not standard SQL.
- `COUNT` always returns an integer; the maximum value depends on the RDBMS (e.g., `INT` in SQL Server, `BIGINT` in PostgreSQL).

### Annotated Complete Code Examples

**Example 1: COUNT(*) and COUNT(column)**

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
    ('David', 'Sales',       NULL),
    ('Eve',   NULL,          88000.00);

-- Query: Count all rows vs. count non-NULL departments
SELECT
    COUNT(*) AS total_rows,
    COUNT(department) AS departments_with_values,
    COUNT(salary) AS salaries_with_values
FROM employees;

-- Expected Output:
--  total_rows | departments_with_values | salaries_with_values
-- ------------+-------------------------+----------------------
--           5 |                       4 |                    4
```

**Why this output occurs:** There are five rows in the table. `COUNT(*)` returns `5`. The `department` column has one `NULL` value (Eve), so `COUNT(department)` returns `4`. The `salary` column has one `NULL` value (David), so `COUNT(salary)` returns `4`. This demonstrates that `COUNT(*)` counts all rows, while `COUNT(column)` counts only non-`NULL` values.

**Example 2: COUNT(DISTINCT)**

```sql
-- Query: Count distinct departments
SELECT COUNT(DISTINCT department) AS unique_departments
FROM employees;

-- Expected Output:
--  unique_departments
-- --------------------
--                   3
```

**Why this output occurs:** The `department` column contains `'Engineering'`, `'Marketing'`, `'Sales'`, and `NULL`. `DISTINCT` removes duplicates and `NULL` values, leaving three unique departments: Engineering, Marketing, and Sales.

### Real-World Cases

**Case 1: User Activity Metrics**

A social media platform counts daily active users with `SELECT COUNT(DISTINCT user_id) FROM activity WHERE activity_date = CURRENT_DATE`. The `DISTINCT` ensures a user who performs multiple actions in a day is counted once.

**Case 2: Data Quality Audit**

A data engineer checks for missing values with `SELECT COUNT(*) - COUNT(email) AS missing_emails FROM users`. The difference between total rows and non-`NULL` emails reveals the number of records missing an email address.

### References

- PostgreSQL Documentation — Aggregate Functions (COUNT) - https://www.postgresql.org/docs/current/functions-aggregate.html
- MySQL 8.4 Reference Manual — COUNT() - https://dev.mysql.com/doc/refman/8.4/en/aggregate-functions.html#function_count
- Microsoft SQL Server — COUNT (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/count-transact-sql
- Oracle Database SQL Language Reference — COUNT - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/COUNT.html

---

## 2. SUM()

### Definitions

**Core Definition:** `SUM()` returns the total (arithmetic sum) of all non-`NULL` numeric values in a column or expression.

**Technical Definition:** `SUM([ALL | DISTINCT] expression)` computes the sum of all values in the expression for each group. `ALL` (the default) includes all non-`NULL` values; `DISTINCT` eliminates duplicate values before summing. If no rows match, or all values are `NULL`, `SUM` returns `NULL`. `SUM` can be applied only to numeric and interval expressions.

**Beginner-Friendly Explanation:** `SUM` adds up all the numbers in a column. It ignores empty values (NULLs) and gives you the total.

### Purposes

- To calculate totals for numeric columns (e.g., revenue, quantity, salary).
- To aggregate financial data for reporting.
- To compute cumulative values across groups.
- To support weighted calculations when combined with other aggregates.
- To measure overall magnitude of a dataset.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SUM ( [ ALL | DISTINCT ] expression )
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `ALL` | Includes all non-`NULL` values (default) |
| `DISTINCT` | Eliminates duplicate values before summing |
| `expression` | A numeric column or expression |

**Syntax Rules:**

- `SUM` ignores `NULL` values.
- `SUM(DISTINCT column)` sums only unique non-`NULL` values.
- The return type depends on the input: integer inputs may produce a larger integer type; `DECIMAL` inputs produce `DECIMAL`; `FLOAT`/`DOUBLE` inputs produce `DOUBLE`.
- `SUM` cannot be used with temporal (date/time) values directly in MySQL; they must be converted to numeric units first.

**Constraints and Limitations:**

- `SUM` applied to an empty set or all-`NULL` column returns `NULL`, not `0`.
- `SUM` cannot be used in the `WHERE` clause.
- In SQLite, `SUM()` returns `NULL` if all values are `NULL`; `TOTAL()` is an alternative that returns `0.0` in that case.
- Overflow is possible with very large sums; the RDBMS may raise an error or wrap around depending on the type.

### Annotated Complete Code Examples

**Example 1: Basic SUM**

```sql
-- Query: Total salary expenditure
SELECT SUM(salary) AS total_salary
FROM employees;

-- Expected Output:
--  total_salary
-- --------------
--      360000.00
```

**Why this output occurs:** The `salary` column contains 95,000, 72,000, 105,000, `NULL` (David), and 88,000. `SUM` ignores the `NULL` value and adds the remaining four: 95,000 + 72,000 + 105,000 + 88,000 = 360,000.

**Example 2: SUM with GROUP BY**

```sql
-- Query: Total salary per department
SELECT department, SUM(salary) AS dept_total
FROM employees
WHERE department IS NOT NULL
GROUP BY department
ORDER BY dept_total DESC;

-- Expected Output:
--  department  | dept_total
-- -------------+------------
--  Engineering |  200000.00
--  Marketing   |   72000.00
--  Sales       |       0.00
```

**Why this output occurs:** The `WHERE` clause excludes the row with `NULL` department. `GROUP BY` groups the remaining rows by department. Engineering has two employees (95,000 + 105,000 = 200,000), Marketing has one (72,000), and Sales has one with `NULL` salary, which `SUM` ignores, resulting in `NULL` displayed as `0.00` in some clients.

### Real-World Cases

**Case 1: Financial Reporting**

A finance team runs `SELECT SUM(amount) FROM transactions WHERE transaction_date BETWEEN '2024-01-01' AND '2024-03-31'` to calculate quarterly revenue.

**Case 2: Inventory Valuation**

A warehouse system computes `SELECT SUM(quantity * unit_cost) AS total_value FROM inventory` to determine the total value of all stock on hand.

### References

- PostgreSQL Documentation — Aggregate Functions (SUM) - https://www.postgresql.org/docs/current/functions-aggregate.html
- MySQL 8.4 Reference Manual — SUM() - https://dev.mysql.com/doc/refman/8.4/en/aggregate-functions.html#function_sum
- Microsoft SQL Server — SUM (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/sum-transact-sql
- Oracle Database SQL Language Reference — SUM - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SUM.html

---

## 3. AVG()

### Definitions

**Core Definition:** `AVG()` returns the arithmetic mean (average) of all non-`NULL` numeric values in a column or expression.

**Technical Definition:** `AVG([ALL | DISTINCT] expression)` computes the sum of all non-`NULL` values divided by the count of those values. `ALL` (default) includes all values; `DISTINCT` averages only unique values. If the set is empty or all values are `NULL`, `AVG` returns `NULL`. The return type follows the same rules as `SUM` for numeric types.

**Beginner-Friendly Explanation:** `AVG` calculates the average — it adds up all the numbers and divides by how many there are, ignoring empty values.

### Purposes

- To compute average values for numeric columns (e.g., average salary, average order value).
- To measure central tendency in data analysis.
- To benchmark performance against averages.
- To support statistical reporting and KPIs.
- To compare group averages when used with `GROUP BY`.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
AVG ( [ ALL | DISTINCT ] expression )
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `ALL` | Averages all non-`NULL` values (default) |
| `DISTINCT` | Averages only unique non-`NULL` values |
| `expression` | A numeric column or expression |

**Syntax Rules:**

- `AVG` ignores `NULL` values in both the numerator and denominator.
- `AVG(DISTINCT column)` eliminates duplicate values before averaging.
- The denominator is the count of non-`NULL` values, not the total row count.
- Like `SUM`, `AVG` cannot operate on temporal values directly in MySQL without conversion.

**Constraints and Limitations:**

- `AVG` returns `NULL` for empty sets or all-`NULL` columns.
- The result type may differ from the input type (e.g., integer input may produce a numeric/decimal result).
- `AVG` cannot be used in the `WHERE` clause.
- Precision may be lost with floating-point inputs.

### Annotated Complete Code Examples

**Example 1: Basic AVG**

```sql
-- Query: Average salary (ignoring NULL)
SELECT AVG(salary) AS average_salary
FROM employees;

-- Expected Output:
--   average_salary
-- -----------------
--       90000.000000
```

**Why this output occurs:** The `salary` values are 95,000, 72,000, 105,000, `NULL`, and 88,000. `AVG` ignores the `NULL` and computes (95,000 + 72,000 + 105,000 + 88,000) / 4 = 360,000 / 4 = 90,000.

**Example 2: AVG with GROUP BY**

```sql
-- Query: Average salary per department
SELECT department, AVG(salary) AS avg_salary
FROM employees
WHERE department IS NOT NULL
GROUP BY department
ORDER BY avg_salary DESC;

-- Expected Output:
--  department  |     avg_salary
-- -------------+--------------------
--  Engineering | 100000.000000000000
--  Marketing   |  72000.000000000000
--  Sales       |                  0
```

**Why this output occurs:** Engineering averages (95,000 + 105,000) / 2 = 100,000. Marketing averages 72,000 / 1 = 72,000. Sales has one employee with `NULL` salary, so `AVG` returns `NULL`, displayed as `0` in some clients.

### Real-World Cases

**Case 1: Performance Metrics**

A call centre manager queries `SELECT AVG(call_duration) FROM calls WHERE call_date = CURRENT_DATE` to measure the average call length for the day.

**Case 2: Student Grades**

An educational platform computes `SELECT student_id, AVG(score) FROM exam_results GROUP BY student_id` to determine each student's average exam score.

### References

- PostgreSQL Documentation — Aggregate Functions (AVG) - https://www.postgresql.org/docs/current/functions-aggregate.html
- MySQL 8.4 Reference Manual — AVG() - https://dev.mysql.com/doc/refman/8.4/en/aggregate-functions.html#function_avg
- Microsoft SQL Server — AVG (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/avg-transact-sql
- Oracle Database SQL Language Reference — AVG - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/AVG.html

---

## 4. MIN()

### Definitions

**Core Definition:** `MIN()` returns the smallest (minimum) value in a column or expression across a set of rows.

**Technical Definition:** `MIN([ALL | DISTINCT] expression)` returns the minimum non-`NULL` value. `DISTINCT` has no practical effect on `MIN` because the minimum value is the same whether duplicates are present or not. `MIN` can be applied to numeric, string, and date/time types. For strings, the minimum is the first value in the collation order; for dates, it is the earliest date.

**Beginner-Friendly Explanation:** `MIN` finds the smallest value — the lowest number, the earliest date, or the first string alphabetically.

### Purposes

- To find the lowest numeric value (e.g., minimum price, lowest salary).
- To find the earliest date or timestamp.
- To find the first string in alphabetical order.
- To identify boundary values for range analysis.
- To compare against averages and maxima for distribution insight.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
MIN ( [ ALL | DISTINCT ] expression )
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `ALL` | Considers all non-`NULL` values (default) |
| `DISTINCT` | Has no practical effect for `MIN` |
| `expression` | A column or expression of a comparable type |

**Syntax Rules:**

- `MIN` ignores `NULL` values.
- `MIN` works with numeric, string, and date/time types.
- `DISTINCT` is syntactically valid but semantically redundant for `MIN`.
- For strings, `MIN` returns the value that sorts first according to the column's collation.

**Constraints and Limitations:**

- `MIN` cannot be used with `LOB` or `LONG` data types in Oracle.
- `MIN` returns `NULL` if the set is empty or all values are `NULL`.
- The result for strings depends on collation settings, which can vary by database and locale.

### Annotated Complete Code Examples

**Example 1: MIN on Numeric Column**

```sql
-- Query: Lowest salary
SELECT MIN(salary) AS lowest_salary
FROM employees;

-- Expected Output:
--  lowest_salary
-- ---------------
--       72000.00
```

**Why this output occurs:** The `salary` values are 95,000, 72,000, 105,000, `NULL`, and 88,000. `MIN` ignores the `NULL` and returns the smallest value: 72,000.

**Example 2: MIN on Date Column**

```sql
-- Setup: Add a hire_date column
ALTER TABLE employees ADD COLUMN hire_date DATE;
UPDATE employees SET hire_date = '2020-03-15' WHERE first_name = 'Alice';
UPDATE employees SET hire_date = '2019-07-01' WHERE first_name = 'Bob';
UPDATE employees SET hire_date = '2021-01-10' WHERE first_name = 'Carol';
UPDATE employees SET hire_date = '2022-11-20' WHERE first_name = 'David';
UPDATE employees SET hire_date = '2018-05-05' WHERE first_name = 'Eve';

-- Query: Earliest hire date
SELECT MIN(hire_date) AS earliest_hire
FROM employees;

-- Expected Output:
--  earliest_hire
-- ---------------
--  2018-05-05
```

**Why this output occurs:** `MIN` compares the date values and returns the earliest one, which is Eve's hire date of 2018-05-05.

### Real-World Cases

**Case 1: Pricing Analysis**

An e-commerce analyst queries `SELECT MIN(price) FROM products WHERE category = 'Electronics'` to find the lowest-priced electronic item.

**Case 2: Employee Tenure**

An HR system uses `SELECT MIN(hire_date) FROM employees` to identify the longest-serving employee's start date.

### References

- PostgreSQL Documentation — Aggregate Functions (MIN) - https://www.postgresql.org/docs/current/functions-aggregate.html
- MySQL 8.4 Reference Manual — MIN() - https://dev.mysql.com/doc/refman/8.4/en/aggregate-functions.html#function_min
- Microsoft SQL Server — MIN (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/min-transact-sql
- Oracle Database SQL Language Reference — MIN - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/MIN.html

---

## 5. MAX()

### Definitions

**Core Definition:** `MAX()` returns the largest (maximum) value in a column or expression across a set of rows.

**Technical Definition:** `MAX([ALL | DISTINCT] expression)` returns the maximum non-`NULL` value. Like `MIN`, `DISTINCT` has no practical effect on `MAX`. `MAX` works with numeric, string, and date/time types. For strings, the maximum is the last value in collation order; for dates, it is the latest date.

**Beginner-Friendly Explanation:** `MAX` finds the largest value — the highest number, the latest date, or the last string alphabetically.

### Purposes

- To find the highest numeric value (e.g., maximum price, highest salary).
- To find the latest date or timestamp.
- To find the last string in alphabetical order.
- To identify upper boundary values for range analysis.
- To compare against averages and minima for distribution insight.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
MAX ( [ ALL | DISTINCT ] expression )
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `ALL` | Considers all non-`NULL` values (default) |
| `DISTINCT` | Has no practical effect for `MAX` |
| `expression` | A column or expression of a comparable type |

**Syntax Rules:**

- `MAX` ignores `NULL` values.
- `MAX` works with numeric, string, and date/time types.
- `DISTINCT` is syntactically valid but semantically redundant for `MAX`.
- For strings, `MAX` returns the value that sorts last according to the column's collation.

**Constraints and Limitations:**

- `MAX` cannot be used with `LOB` or `LONG` data types in Oracle.
- `MAX` returns `NULL` if the set is empty or all values are `NULL`.
- String comparison depends on collation settings, which can vary by database and locale.

### Annotated Complete Code Examples

**Example 1: MAX on Numeric Column**

```sql
-- Query: Highest salary
SELECT MAX(salary) AS highest_salary
FROM employees;

-- Expected Output:
--  highest_salary
-- ----------------
--       105000.00
```

**Why this output occurs:** The `salary` values are 95,000, 72,000, 105,000, `NULL`, and 88,000. `MAX` ignores the `NULL` and returns the largest value: 105,000.

**Example 2: MAX on Date Column**

```sql
-- Query: Latest hire date
SELECT MAX(hire_date) AS latest_hire
FROM employees;

-- Expected Output:
--  latest_hire
-- -------------
--  2022-11-20
```

**Why this output occurs:** `MAX` compares the date values and returns the latest one, which is David's hire date of 2022-11-20.

**Example 3: Using MIN, MAX, AVG, and COUNT Together**

```sql
-- Query: Salary statistics
SELECT
    COUNT(salary) AS employee_count,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary,
    AVG(salary) AS avg_salary
FROM employees;

-- Expected Output:
--  employee_count | min_salary | max_salary |     avg_salary
-- ----------------+------------+------------+--------------------
--               4 |   72000.00 |  105000.00 | 90000.000000000000
```

**Why this output occurs:** All four aggregates operate on the same non-`NULL` salary values: 95,000, 72,000, 105,000, and 88,000. `COUNT` returns 4, `MIN` returns 72,000, `MAX` returns 105,000, and `AVG` returns 90,000. This demonstrates how multiple aggregates can be combined in a single query for a comprehensive statistical summary.

### Real-World Cases

**Case 1: Sales Performance**

A sales manager queries `SELECT MAX(amount) FROM orders WHERE order_date >= '2024-01-01'` to find the largest single order of the year.

**Case 2: System Monitoring**

An operations team uses `SELECT MAX(response_time) FROM api_logs WHERE log_date = CURRENT_DATE` to identify the slowest API response of the day.

### References

- PostgreSQL Documentation — Aggregate Functions (MAX) - https://www.postgresql.org/docs/current/functions-aggregate.html
- MySQL 8.4 Reference Manual — MAX() - https://dev.mysql.com/doc/refman/8.4/en/aggregate-functions.html#function_max
- Microsoft SQL Server — MAX (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/max-transact-sql
- Oracle Database SQL Language Reference — MAX - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/MAX.html

---

## Aggregate Functions: Interaction with GROUP BY and HAVING

Aggregate functions are most powerful when combined with `GROUP BY` and `HAVING`. The `GROUP BY` clause divides rows into groups, and aggregates are computed per group. The `HAVING` clause then filters groups based on aggregate results.

**Complete General Syntax:**

```sql
SELECT grouping_column, aggregate_function(column)
FROM table_name
WHERE row_condition
GROUP BY grouping_column
HAVING aggregate_condition
ORDER BY ...;
```

**Key Rules:**

- `WHERE` filters rows before grouping; `HAVING` filters groups after aggregation.
- Aggregate functions cannot appear in the `WHERE` clause.
- Columns in the `SELECT` list must either be aggregate functions or appear in the `GROUP BY` clause (standard SQL).

**Example:**

```sql
SELECT department,
       COUNT(*) AS headcount,
       AVG(salary) AS avg_salary
FROM employees
WHERE salary IS NOT NULL
GROUP BY department
HAVING AVG(salary) > 80000
ORDER BY avg_salary DESC;

-- Expected Output:
--  department  | headcount |     avg_salary
-- -------------+-----------+--------------------
--  Engineering |         2 | 100000.000000000000
```

**Why this output occurs:** `WHERE` excludes rows with `NULL` salary. `GROUP BY` groups remaining rows by department. `HAVING` keeps only groups where the average salary exceeds 80,000. Only Engineering qualifies.

---

## Summary Table of Aggregate Functions

| Function | Return Type | Ignores NULL? | DISTINCT Support | Works with Strings? | Works with Dates? |
|----------|------------|---------------|------------------|--------------------|--------------------|
| `COUNT()` | Integer | `COUNT(*)` no; `COUNT(expr)` yes | Yes | Yes (counts) | Yes (counts) |
| `SUM()` | Numeric | Yes | Yes | No | No (MySQL) |
| `AVG()` | Numeric | Yes | Yes | No | No (MySQL) |
| `MIN()` | Same as input | Yes | Redundant | Yes | Yes |
| `MAX()` | Same as input | Yes | Redundant | Yes | Yes |

---

## NULL Handling Summary

- All aggregate functions except `COUNT(*)` ignore `NULL` values.
- `COUNT(*)` counts all rows, including those with `NULL` values.
- `COUNT(column)` counts only rows where `column` is not `NULL`.
- `SUM`, `AVG`, `MIN`, and `MAX` return `NULL` if the set is empty or all values are `NULL`.
- To include `NULL` values in computations, use `COALESCE(column, value)` to substitute a default.

---

## DISTINCT in Aggregate Functions

`DISTINCT` eliminates duplicate values before the aggregate is computed. It is supported by `COUNT`, `SUM`, and `AVG`; it is syntactically valid but redundant for `MIN` and `MAX`.

**Example:**

```sql
-- Without DISTINCT: average of all values
SELECT AVG(salary) FROM employees;  -- 90000

-- With DISTINCT: average of unique values
SELECT AVG(DISTINCT salary) FROM employees;  -- 90000 (if all unique)
```

**Example with duplicates:**

```sql
-- Suppose salaries are: 50000, 50000, 100000
SELECT AVG(salary) FROM employees;          -- 66666.67
SELECT AVG(DISTINCT salary) FROM employees; -- 75000
```

---

## References

- PostgreSQL Documentation — Aggregate Functions - https://www.postgresql.org/docs/current/functions-aggregate.html
- PostgreSQL Tutorial — Aggregate Functions - https://www.postgresql.org/docs/current/tutorial-agg.html
- MySQL 8.4 Reference Manual — Aggregate Function Descriptions - https://dev.mysql.com/doc/refman/8.4/en/aggregate-functions.html
- Microsoft SQL Server — Aggregate Functions (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/aggregate-functions-transact-sql
- Oracle Database SQL Language Reference — Aggregate Functions - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Aggregate-Functions.html
- SQLite Documentation — Aggregate Functions - https://www.sqlite.org/lang_aggfunc.html
- SAP SQL Anywhere — Aggregate Functions and NULL - https://help.sap.com/docs/SAP_SQL_Anywhere/e38b2f6217f24bdb90a3ff8ae57b1dd5/81818d496ce21014a9b59da6ac36d8fd.html
- IBM Db2 — Aggregate Functions - https://www.ibm.com/docs/en/db2/11.5?topic=functions-aggregate