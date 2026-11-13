# Comprehensive Programming Cheat Sheet: SQL Basic Query Structure

## Topic Overview

### Definitions

**Core Definition**
The basic query structure is the set of clauses—`SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`, `LIMIT`, `OFFSET`, and `DISTINCT`—that together form the `SELECT` statement, the primary SQL command for retrieving data from a relational database.

**Technical Definition**
The `SELECT` statement retrieves rows from one or more tables. Its general syntax is `[WITH with_queries] SELECT select_list FROM table_expression [sort_specification]`, where the select list specifies the columns or expressions to return, the table expression specifies the data sources, and the sort specification controls row order. The clauses are written in a lexical order inspired by English (“SELECT this FROM that WHERE …”), but the database evaluates them in a different logical order: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `DISTINCT` → `ORDER BY` → `LIMIT`/`OFFSET`. Beyond this logical order, the query planner chooses a physical execution plan—join order, join algorithms, index usage, and parallelisation—that is free to diverge from the written order as long as the result is equivalent.

**Beginner-Friendly Explanation**
A SQL query is like a recipe with steps that are written in one order but actually performed in another. You write `SELECT … FROM … WHERE …`, but the database first figures out which tables to read (FROM), then throws away rows that do not meet your conditions (WHERE), then groups the survivors (GROUP BY), then filters the groups (HAVING), then decides which columns to show (SELECT), then removes duplicates (DISTINCT), then sorts the result (ORDER BY), and finally takes only the slice you asked for (LIMIT/OFFSET). Understanding this hidden order explains why you cannot use a `SELECT` alias in `WHERE`, why `HAVING` can filter on aggregates but `WHERE` cannot, and why `ORDER BY` sees the final projected columns.

### Key Characteristics

- **Lexical order ≠ logical order**: The clauses are written in one sequence but evaluated in another; this is the single most important source of confusion for beginners.
- **Declarative, not procedural**: SQL describes *what* result is wanted, not *how* to compute it; the query planner is free to reorder joins, push filters down, and choose any physical plan that returns the same result.
- **Aliases are scoped by logical order**: A `SELECT` alias is not visible in `WHERE` because `WHERE` is evaluated before `SELECT`; it is visible in `ORDER BY` because `ORDER BY` is evaluated after `SELECT`.
- **`DISTINCT ON` is PostgreSQL-specific**: It returns the first row of each group defined by the specified expressions, and the `ORDER BY` must begin with those same expressions.
- **`LIMIT` and `OFFSET` are applied last**: They paginate the already-sorted result set; without `ORDER BY`, the rows returned by `LIMIT` are unpredictable.

### Prerequisites

- A running RDBMS (PostgreSQL, MySQL, or SQL Server) and a client connection.
- At least one table with data to query, or the ability to use `SELECT` as a calculator (`SELECT 3 * 4`).
- Basic familiarity with comparison operators (`=`, `<>`, `<`, `>`, `BETWEEN`, `IN`, `LIKE`, `IS NULL`).
- Awareness of the difference between raw rows (`WHERE`) and grouped rows (`HAVING`).

### Related Programming Areas

- Data Manipulation Language (DML) – `INSERT`, `UPDATE`, `DELETE` operate on the rows that `SELECT` retrieves.
- Query optimisation – indexes, join strategies, and the query planner determine physical execution.
- Aggregate functions – `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` are evaluated in the `GROUP BY`/`HAVING` phase.
- Window functions – evaluated after `HAVING` and before `ORDER BY`.
- Pagination and reporting – `ORDER BY`, `LIMIT`, and `OFFSET` are the standard pagination primitives.

---

## Core Concepts and Features

### 1. `SELECT`

#### Definitions

**Core Definition**
`SELECT` specifies the columns, expressions, or computed values that the query returns.

**Technical Definition**
The `select_list` determines which columns or expressions appear in the output. A single unqualified `*` selects all columns from all tables in the `FROM` clause; a qualified `tbl_name.*` selects all columns from one table. Expressions can include arithmetic, string concatenation, function calls, and `CASE` expressions, and each can be given an alias with `AS`. `SELECT` is logically evaluated after `FROM`, `WHERE`, `GROUP BY`, and `HAVING`, which is why aliases defined in `SELECT` are not available in `WHERE`.

**Beginner-Friendly Explanation**
`SELECT` is where you list what you want to see. You can pick individual columns, compute new values (`price * 1.19 AS price_with_tax`), or use `*` to grab everything.

#### Purposes

- To choose the columns or computed values that appear in the result set.
- To assign readable aliases to expressions for reporting and downstream processing.
- To apply `CASE` logic to classify or transform values inline.

#### Syntax Rules and Structure

```sql
SELECT [ALL | DISTINCT [ON (expression, ...)]]
    { * | expression [AS output_name] } [, ...]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ALL` | Default; returns all rows, including duplicates. |
| `DISTINCT` | Removes duplicate rows from the result. |
| `DISTINCT ON (expression, ...)` | PostgreSQL extension; keeps the first row of each group defined by the expressions. |
| `*` | All columns from all tables in `FROM`. |
| `tbl_name.*` | All columns from the named table. |
| `expression AS output_name` | A column, computation, or function call with an optional alias. |

**Syntax rules:**

- The `select_list` must contain at least one expression.
- `DISTINCT` applies to the entire row produced by the select list, not to individual columns in isolation.
- `DISTINCT ON` expressions must match the leftmost `ORDER BY` expressions.

**Constraints and limitations:**

- `SELECT *` is convenient for exploration but risky in production because new or sensitive columns may appear unexpectedly.
- Aliases defined in `SELECT` are not visible in `WHERE` or `GROUP BY` (though MySQL allows them in `GROUP BY` and `HAVING`).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Selecting specific columns with an alias and a computed expression

```sql
-- Step 1: Select specific columns and compute an annual salary.
SELECT first_name || ' ' || last_name AS full_name,
       salary * 12 AS annual_salary
FROM employees
WHERE department = 'Engineering';
```

**Expected output:**

```text
   full_name    | annual_salary
----------------+---------------
 Alice Smith    |        960000
 Bob Jones      |        840000
(2 rows)
```

**Why this output appears**: The `SELECT` list concatenates `first_name` and `last_name` into `full_name` and multiplies `salary` by 12 to produce `annual_salary`. The `WHERE` clause filters to the Engineering department. The aliases make the result readable without changing the underlying data.

#### Real-World Cases

- **API projections**: A REST endpoint selects only the columns the client needs, avoiding over-fetching and sensitive fields.
- **Reporting**: `CASE` expressions in the `SELECT` list classify revenue into tiers without a separate lookup table.

---

### 2. `FROM`

#### Definitions

**Core Definition**
`FROM` specifies the table or tables from which the query retrieves rows, including joins and subqueries.

**Technical Definition**
`FROM table1` is the simplest table expression, reading a single table. In general, table expressions can be complex constructs of base tables, joins, and subqueries. `FROM` is the **first clause evaluated logically**, which is why it determines the universe of rows that all later clauses operate on. The query planner may reorder joins and choose different join algorithms, so the written order of tables in `FROM` does not determine physical execution order.

**Beginner-Friendly Explanation**
`FROM` tells the database which tables contain the data. It is the starting point: everything else filters, groups, or shapes the rows that `FROM` produces.

#### Purposes

- To declare the data sources for the query.
- To join multiple tables into a single working set.
- To introduce subqueries and derived tables that later clauses can reference.

#### Syntax Rules and Structure

```sql
FROM table_reference [, ...]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `table_name` | A base table or view. |
| `table_name [AS] alias` | A table with a correlation alias. |
| `JOIN ... ON ...` | An explicit join between two table references. |
| `(subquery) AS alias` | A derived table. |

**Syntax rules:**

- The `FROM` clause can be omitted entirely, in which case `SELECT` acts as a calculator (`SELECT 3 * 4`).
- Each table reference should have a unique alias if the same table appears more than once.

**Constraints and limitations:**

- The physical join order is chosen by the optimizer and may differ from the written order.
- Cartesian products from missing join conditions can explode the result set.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Joining two tables in FROM

```sql
-- Step 1: Join employees to departments.
SELECT e.first_name, d.department_name
FROM employees AS e
JOIN departments AS d ON e.department_id = d.department_id
WHERE e.salary > 70000;
```

**Expected output:**

```text
 first_name | department_name
------------+----------------
 Alice      | Engineering
 Bob        | Engineering
(2 rows)
```

**Why this output appears**: The `FROM` clause joins `employees` and `departments` on the matching department ID, producing a combined working set. The `WHERE` clause then filters to employees earning more than 70,000.

#### Real-World Cases

- **Normalised schemas**: Queries join orders to customers and products to produce a denormalised report.
- **Derived tables**: A subquery in `FROM` pre-aggregates data before the outer query filters it.

---

### 3. `WHERE`

#### Definitions

**Core Definition**
`WHERE` filters individual rows *before* grouping, returning only rows for which the Boolean condition is true.

**Technical Definition**
The `WHERE` clause contains a Boolean expression, and only rows for which the expression evaluates to `TRUE` are returned. `WHERE` is evaluated **after `FROM` and before `GROUP BY`**, so it cannot reference aggregate functions and cannot see `SELECT` aliases. MySQL notes that aggregate (group) functions are not allowed in `WHERE`.

**Beginner-Friendly Explanation**
`WHERE` is the row filter. It removes rows you do not want before any grouping or sorting happens.

#### Purposes

- To restrict the result set to rows that satisfy a condition.
- To reduce the amount of data processed by later clauses.
- To express equality, range, membership, pattern, and null checks.

#### Syntax Rules and Structure

```sql
WHERE condition
```

**Component breakdown:**

| Operator | Example |
|----------|---------|
| `=` / `<>` | `department = 'Engineering'` |
| `>` / `<` / `>=` / `<=` | `salary > 70000` |
| `BETWEEN` | `salary BETWEEN 50000 AND 80000` |
| `IN` | `department IN ('Engineering', 'Sales')` |
| `LIKE` | `first_name LIKE 'A%'` |
| `IS NULL` / `IS NOT NULL` | `manager_id IS NULL` |

**Syntax rules:**

- The condition must evaluate to a Boolean; rows where it is `FALSE` or `UNKNOWN` are discarded.
- Aggregate functions are not allowed in `WHERE`.
- `SELECT` aliases are not visible in `WHERE` because `WHERE` is evaluated before `SELECT`.

**Constraints and limitations:**

- Complex `WHERE` conditions without supporting indexes can force full table scans.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Filtering rows with multiple conditions

```sql
-- Step 1: Find Engineering employees earning above 70k.
SELECT first_name, last_name, salary
FROM employees
WHERE department = 'Engineering'
  AND salary > 70000;
```

**Expected output:**

```text
 first_name | last_name | salary
------------+-----------+--------
 Alice      | Smith     |  80000
 Bob        | Jones     |  70001
(2 rows)
```

**Why this output appears**: The `WHERE` clause applies both predicates to each row; only rows satisfying both conditions survive. The result is then projected by `SELECT`.

#### Real-World Cases

- **Search filters**: A product search applies `WHERE price BETWEEN … AND … AND category IN (…)`.
- **Soft deletes**: `WHERE deleted_at IS NULL` excludes logically deleted rows.

---

### 4. `GROUP BY`

#### Definitions

**Core Definition**
`GROUP BY` collapses rows that share the same values in the specified expressions into groups, enabling aggregate functions to be computed per group.

**Technical Definition**
`GROUP BY` is evaluated **after `WHERE` and before `HAVING`**. It reduces the working set to one row per group, and the `SELECT` list may then contain only grouping expressions and aggregate functions (or expressions functionally dependent on the grouping columns). MySQL documents `GROUP BY` as accepting column names, expressions, or positional references, with optional `WITH ROLLUP` for super-aggregate rows.

**Beginner-Friendly Explanation**
`GROUP BY` turns a pile of rows into buckets. All rows with the same value end up in the same bucket, and then you can compute totals or counts for each bucket.

#### Purposes

- To compute aggregates (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) per category.
- To produce summary reports from detailed data.
- To feed `HAVING` with grouped rows for further filtering.

#### Syntax Rules and Structure

```sql
GROUP BY grouping_element [, ...]
-- grouping_element can be a column name, expression, or position
```

**Component breakdown:**

| Form | Example |
|------|---------|
| Column | `GROUP BY department` |
| Expression | `GROUP BY EXTRACT(YEAR FROM order_date)` |
| Position | `GROUP BY 1` (first select item) |
| `ROLLUP` | `GROUP BY ROLLUP (region, product)` |

**Syntax rules:**

- Every non-aggregate expression in the `SELECT` list must appear in `GROUP BY` (or be functionally dependent on a grouping column).
- Aggregate functions are allowed in `SELECT` and `HAVING` after grouping, but not in `WHERE`.

**Constraints and limitations:**

- `GROUP BY` can produce many groups on high-cardinality columns; this can be memory-intensive.
- Grouping on expressions can prevent index usage unless an expression index exists.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Aggregating salary by department

```sql
-- Step 1: Compute average salary per department.
SELECT department,
       COUNT(*) AS headcount,
       AVG(salary) AS avg_salary
FROM employees
WHERE salary > 0
GROUP BY department;
```

**Expected output:**

```text
 department  | headcount | avg_salary
-------------+-----------+------------
 Engineering |         2 |      75000
 Sales       |         1 |      60000
(2 rows)
```

**Why this output appears**: `WHERE` removes non-positive salaries, `GROUP BY department` creates one bucket per department, and the aggregate functions compute headcount and average salary within each bucket.

#### Real-World Cases

- **Sales reporting**: `GROUP BY region` computes total revenue per region.
- **User analytics**: `GROUP BY signup_month` counts new users per month.

---

### 5. `HAVING`

#### Definitions

**Core Definition**
`HAVING` filters *groups* after `GROUP BY`, returning only groups for which the Boolean condition is true.

**Technical Definition**
`HAVING` is evaluated **after `GROUP BY` and before `SELECT`**. Unlike `WHERE`, which filters raw rows, `HAVING` filters grouped rows and therefore may reference aggregate functions. The distinction is critical: `WHERE` removes rows before grouping, while `HAVING` removes groups after grouping.

**Beginner-Friendly Explanation**
`HAVING` is the group filter. It lets you say “only keep departments whose average salary is above 70,000,” which `WHERE` cannot do because the average does not exist until after grouping.

#### Purposes

- To filter groups based on aggregate results.
- To keep only categories that meet a threshold (e.g., “departments with at least 5 employees”).
- To combine aggregate and non-aggregate conditions on grouped data.

#### Syntax Rules and Structure

```sql
HAVING condition
```

**Component breakdown:**

| Condition type | Example |
|----------------|---------|
| Aggregate | `HAVING AVG(salary) > 70000` |
| Non-aggregate grouping column | `HAVING department <> 'Interns'` |
| Combination | `HAVING COUNT(*) >= 5 AND AVG(salary) > 60000` |

**Syntax rules:**

- `HAVING` requires `GROUP BY` unless the entire query is an aggregate query with no grouping (e.g., `SELECT COUNT(*) FROM t HAVING COUNT(*) > 10`).
- `HAVING` can reference `SELECT` aliases in MySQL, but not in standard SQL or PostgreSQL without repeating the expression.

**Constraints and limitations:**

- Putting row-level filters in `HAVING` instead of `WHERE` forces unnecessary grouping work and can produce wrong results.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Filtering groups with HAVING

```sql
-- Step 1: Keep only departments with an average salary above 70k.
SELECT department,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 70000;
```

**Expected output:**

```text
 department  | avg_salary
-------------+------------
 Engineering |      75000
(1 row)
```

**Why this output appears**: `GROUP BY` computes the average per department, and `HAVING AVG(salary) > 70000` removes the Sales group whose average is 60,000. Only Engineering survives.

#### Real-World Cases

- **Inventory alerts**: `HAVING SUM(quantity) < 10` identifies products with low total stock.
- **Customer segmentation**: `HAVING COUNT(*) > 5` keeps only customers with more than five orders.

---

### 6. `ORDER BY`

#### Definitions

**Core Definition**
`ORDER BY` sorts the final result set by one or more expressions, each independently ascending or descending.

**Technical Definition**
`ORDER BY` is evaluated **after `SELECT` and `DISTINCT`**. Because it sees the projected columns, it can reference `SELECT` aliases and positional references (e.g., `ORDER BY 2 DESC`). The sort is the last operation before `LIMIT`/`OFFSET`, so it determines which rows survive pagination.

**Beginner-Friendly Explanation**
`ORDER BY` sorts the rows you are about to return. It is the only clause that guarantees a predictable row order; without it, the database may return rows in any order.

#### Purposes

- To produce a deterministic row order for display or pagination.
- To sort by computed expressions or aliases.
- To control the direction (ascending or descending) and null placement.

#### Syntax Rules and Structure

```sql
ORDER BY { expression | position | alias } [ASC | DESC] [NULLS {FIRST | LAST}] [, ...]
```

**Component breakdown:**

| Form | Example |
|------|---------|
| Column | `ORDER BY last_name` |
| Position | `ORDER BY 2 DESC` |
| Alias | `ORDER BY annual_salary DESC` |
| Expression | `ORDER BY salary * 12 DESC` |
| Direction | `ASC` (default) or `DESC` |
| Null placement | `NULLS FIRST` or `NULLS LAST` |

**Syntax rules:**

- `ORDER BY` can reference `SELECT` aliases because it is evaluated after `SELECT`.
- Positional references (e.g., `ORDER BY 1`) refer to the select-list position.
- Without `ORDER BY`, `LIMIT` returns an unpredictable subset.

**Constraints and limitations:**

- Sorting large result sets can be expensive; indexes on the sort columns can help, but computed expressions require expression indexes.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Sorting by an alias in descending order

```sql
-- Step 1: Sort employees by annual salary, highest first.
SELECT first_name, salary * 12 AS annual_salary
FROM employees
ORDER BY annual_salary DESC;
```

**Expected output:**

```text
 first_name | annual_salary
------------+---------------
 Alice      |        960000
 Bob        |        840000
 Carol      |        720000
(3 rows)
```

**Why this output appears**: The `SELECT` clause computes `annual_salary`, and `ORDER BY annual_salary DESC` sorts the projected rows from highest to lowest. The alias is visible because `ORDER BY` runs after `SELECT`.

#### Real-World Cases

- **Leaderboards**: `ORDER BY score DESC LIMIT 10` returns the top ten players.
- **Recent activity**: `ORDER BY created_at DESC` shows the newest records first.

---

### 7. `LIMIT` and `OFFSET`

#### Definitions

**Core Definition**
`LIMIT` restricts the number of rows returned, and `OFFSET` skips a specified number of rows before starting to return rows.

**Technical Definition**
`LIMIT` and `OFFSET` are evaluated **last**, after `ORDER BY`, creating a paginating view of the ordered tuples. MySQL’s syntax is `LIMIT {[offset,] row_count | row_count OFFSET offset}`. PostgreSQL documents `LIMIT` and `OFFSET` as returning a subset of the query’s rows.

**Beginner-Friendly Explanation**
`LIMIT` says “give me only this many rows,” and `OFFSET` says “skip this many rows first.” Together they implement pagination: page 1 is `LIMIT 10 OFFSET 0`, page 2 is `LIMIT 10 OFFSET 10`, and so on.

#### Purposes

- To implement pagination in applications and APIs.
- To preview a small sample of a large result set during exploration.
- To bound the work done by a query when only the first few rows matter.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
LIMIT row_count OFFSET skip_count
-- or
OFFSET skip_count LIMIT row_count
```

**MySQL:**

```sql
LIMIT [offset,] row_count
-- or
LIMIT row_count OFFSET offset
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `row_count` | Maximum number of rows to return. |
| `offset` / `skip_count` | Number of rows to skip before returning rows. |

**Syntax rules:**

- `LIMIT` should be paired with `ORDER BY` for deterministic pagination.
- `OFFSET` without `ORDER BY` is unpredictable.

**Constraints and limitations:**

- Large `OFFSET` values force the database to scan and discard many rows, which becomes slow on large tables.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Paginating results

```sql
-- Step 1: Return the second page of employees ordered by salary.
SELECT first_name, salary
FROM employees
ORDER BY salary DESC
LIMIT 10 OFFSET 10;
```

**Expected output:**

```text
 first_name | salary
------------+--------
 Bob        |  70000
 Carol      |  60000
(2 rows)
```

**Why this output appears**: The query sorts all employees by salary descending, skips the first 10 rows, and returns the next 10. The result is the second page of a 10-row-per-page pagination scheme.

#### Real-World Cases

- **API pagination**: `GET /users?page=2&per_page=20` translates to `LIMIT 20 OFFSET 20`.
- **Data exploration**: `SELECT * FROM logs ORDER BY created_at DESC LIMIT 100` previews the latest log entries.

---

### 8. `DISTINCT`

#### Definitions

**Core Definition**
`DISTINCT` removes duplicate rows from the result set, keeping only unique combinations of the selected expressions.

**Technical Definition**
`DISTINCT` is applied **after `SELECT` and before `ORDER BY`**. It operates on the full row produced by the select list, so `SELECT DISTINCT city, country` removes duplicate (city, country) pairs, not duplicate cities in isolation. MySQL also supports `DISTINCTROW` as a synonym.

**Beginner-Friendly Explanation**
`DISTINCT` says “show me each unique value only once.” It is useful when you want a list of categories or countries without repeats.

#### Purposes

- To produce a list of unique values from a column.
- To remove duplicate rows caused by joins or denormalised data.
- To count distinct values with `COUNT(DISTINCT column)`.

#### Syntax Rules and Structure

```sql
SELECT DISTINCT column1 [, column2, ...] FROM table;
```

**Component breakdown:**

| Form | Description |
|------|-------------|
| `DISTINCT` | Applied to the entire select list. |
| `DISTINCT ON (expression)` | PostgreSQL extension; keeps the first row per group. |

**Syntax rules:**

- `DISTINCT` cannot be used with `DISTINCT ON` in the same query.
- `DISTINCT` on a single column is straightforward; on multiple columns it removes duplicate combinations.

**Constraints and limitations:**

- `DISTINCT` requires sorting or hashing the result, which can be expensive on large data sets.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Distinct values from one column

```sql
-- Step 1: List the distinct departments.
SELECT DISTINCT department FROM employees;
```

**Expected output:**

```text
 department
-------------
 Engineering
 Sales
 Marketing
(3 rows)
```

**Why this output appears**: `DISTINCT` collapses all duplicate department values into one row per unique department.

#### Real-World Cases

- **Filter menus**: Populate a dropdown with `SELECT DISTINCT country FROM customers`.
- **Deduplication**: Remove duplicates introduced by a join with `SELECT DISTINCT order_id FROM order_items`.

---

### 9. `DISTINCT ON` (PostgreSQL)

#### Definitions

**Core Definition**
`DISTINCT ON (expression, ...)` is a PostgreSQL extension that returns the **first row** of each group of rows that share the same values in the specified expressions.

**Technical Definition**
`DISTINCT ON` eliminates rows that match on all the specified expressions, keeping only the first row of each set of duplicates. The `DISTINCT ON` expressions must match the **leftmost `ORDER BY` expressions**, so the `ORDER BY` controls which row is considered “first” in each group. A common pattern is “latest order per customer,” written as `SELECT DISTINCT ON (customer_id) customer_id, order_id, placed_at FROM orders ORDER BY customer_id, placed_at DESC`.

**Beginner-Friendly Explanation**
`DISTINCT ON` is like `DISTINCT` but with a superpower: instead of keeping an arbitrary row from each group, it keeps the row that comes first according to your `ORDER BY`. If you sort each customer’s orders by date descending, `DISTINCT ON (customer_id)` gives you the most recent order per customer.

#### Purposes

- To return the first (or latest, or highest) row per group without a subquery or window function.
- To deduplicate data while controlling which row survives.
- To implement “latest record per entity” queries concisely.

#### Syntax Rules and Structure

```sql
SELECT DISTINCT ON (expression, ...) select_list
FROM table
[WHERE ...]
ORDER BY expression, ... [, ...];
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `DISTINCT ON (expression, ...)` | The grouping key for deduplication. |
| `ORDER BY` | Must begin with the `DISTINCT ON` expressions; determines which row is kept. |

**Syntax rules:**

- The `DISTINCT ON` expressions must match the leftmost `ORDER BY` expressions.
- Without `ORDER BY`, the choice of row is unpredictable.

**Constraints and limitations:**

- PostgreSQL-specific; not available in MySQL or SQL Server.
- The `ORDER BY` requirement means you cannot sort the final result by a different column without wrapping the query in a subquery.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Latest order per customer

```sql
-- Step 1: Retrieve the most recent order for each customer.
SELECT DISTINCT ON (customer_id)
       customer_id,
       order_id,
       placed_at,
       total
FROM orders
ORDER BY customer_id, placed_at DESC;
```

**Expected output:**

```text
 customer_id | order_id |     placed_at      | total
-------------+----------+--------------------+-------
           1 |      105 | 2025-01-15 10:30:00 | 250.00
           2 |      103 | 2025-01-14 09:00:00 | 120.00
           3 |      108 | 2025-01-16 14:15:00 | 400.00
(3 rows)
```

**Why this output appears**: For each `customer_id`, the rows are sorted by `placed_at DESC`, so the first row in each group is the most recent order. `DISTINCT ON (customer_id)` keeps exactly that row and discards the rest.

#### Real-World Cases

- **Latest status**: The most recent status update per ticket.
- **First purchase**: The first order per customer, sorted by `placed_at ASC`.

---

### 10. Logical Order of Execution

#### Definitions

**Core Definition**
The logical order of execution is the sequence in which the database *conceptually* evaluates the clauses of a `SELECT` statement, regardless of how they are written.

**Technical Definition**
The logical order is: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `DISTINCT` → `ORDER BY` → `LIMIT`/`OFFSET`. This order explains many rules: `WHERE` cannot use `SELECT` aliases (because `SELECT` runs after `WHERE`), `HAVING` can use aggregates (because it runs after `GROUP BY`), and `ORDER BY` can use aliases (because it runs after `SELECT`). The lexical order in which you write the clauses—`SELECT … FROM … WHERE …`—is different from the logical order.

**Beginner-Friendly Explanation**
The logical order is the “what happens first” order that the SQL standard defines. Even though you write `SELECT` first, the database first reads the tables (`FROM`), then filters rows (`WHERE`), then groups (`GROUP BY`), then filters groups (`HAVING`), then picks columns (`SELECT`), then removes duplicates (`DISTINCT`), then sorts (`ORDER BY`), and finally paginates (`LIMIT`).

#### Purposes

- To understand why certain clauses can or cannot reference aliases and aggregates.
- To predict which rows will be filtered before grouping and which after.
- To reason about query results without knowing the physical execution details.

#### Syntax Rules and Structure

**Logical pipeline:**

| Step | Clause | What happens |
|------|--------|--------------|
| 1 | `FROM` | Data sources are identified and joined |
| 2 | `WHERE` | Individual rows are filtered |
| 3 | `GROUP BY` | Remaining rows are grouped |
| 4 | `HAVING` | Groups are filtered |
| 5 | `SELECT` | Projection and expressions are evaluated |
| 6 | `DISTINCT` | Duplicate rows are removed |
| 7 | `ORDER BY` | Remaining rows are sorted |
| 8 | `LIMIT` / `OFFSET` | A subset of rows is returned |

**Syntax rules:**

- The logical order is fixed and independent of the lexical order.
- The physical plan may interleave or reorder operations, but it must produce the same logical result.

**Constraints and limitations:**

- The logical order is a conceptual model; the actual physical execution can differ dramatically.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Demonstrating the logical order with an alias error

```sql
-- Step 1: This query fails because WHERE runs before SELECT.
SELECT salary * 12 AS annual_salary
FROM employees
WHERE annual_salary > 900000;
```

**Expected output:**

```text
ERROR:  column "annual_salary" does not exist
```

**Why this output appears**: The alias `annual_salary` is defined in the `SELECT` clause, but `WHERE` is evaluated **before** `SELECT` in the logical order. Therefore the alias is not yet visible, and the query fails.

---

### 11. Physical Order of Execution

#### Definitions

**Core Definition**
The physical order of execution is the actual sequence of operations—join order, join algorithms, index scans, sort methods, and parallelisation—that the query planner chooses to produce the result.

**Technical Definition**
SQL is a declarative language, and the database engine is free to choose any execution order and optimisation method that produces the correct result. The query planner constructs a logical plan as a tree of logical operators, then chooses a physical operator for each logical operator using a cost‑based approach. The planner may reorder joins, push predicates down, choose nested‑loop, merge, or hash joins, and decide whether to use indexes or sequential scans. The physical plan is therefore not determined by the order of tables in the `FROM` clause.

**Beginner-Friendly Explanation**
The physical order is “how the database actually gets the job done.” Even though you wrote `FROM t1, t2`, the optimizer might read `t2` first, or use a hash join instead of a nested loop, or use an index instead of scanning the whole table. You cannot predict the physical plan from the SQL text alone; you need `EXPLAIN` to see it.

#### Purposes

- To understand why two logically equivalent queries can have wildly different performance.
- To interpret `EXPLAIN` output and identify expensive operations.
- To influence the planner with indexes, statistics, and (where appropriate) hints.

#### Syntax Rules and Structure

Physical execution is observed with `EXPLAIN`:

```sql
EXPLAIN SELECT ...;
EXPLAIN ANALYZE SELECT ...;
```

**Component breakdown:**

| Physical operation | Description |
|--------------------|-------------|
| Sequential scan | Reads every row of a table |
| Index scan | Uses an index to locate rows |
| Nested loop join | For each outer row, scans the inner input |
| Hash join | Builds a hash table on one input and probes it |
| Merge join | Merges two sorted inputs |
| Sort | Sorts rows in memory or on disk |
| Aggregate | Computes group aggregates |

**Syntax rules:**

- `EXPLAIN` shows the plan without executing; `EXPLAIN ANALYZE` executes and reports actual timings.
- Plan choices depend on statistics, cost settings, and available indexes.

**Constraints and limitations:**

- Physical plans are not guaranteed to be stable across data changes or version upgrades.
- You cannot force a specific plan in standard PostgreSQL without extensions; the planner chooses based on cost.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Observing a physical plan with EXPLAIN

```sql
-- Step 1: Ask the planner how it will execute the query.
EXPLAIN
SELECT e.first_name, d.department_name
FROM employees AS e
JOIN departments AS d ON e.department_id = d.department_id
WHERE e.salary > 70000;
```

**Expected output:**

```text
Hash Join  (cost=1.09..2.20 rows=2 width=68)
  Hash Cond: (e.department_id = d.department_id)
  ->  Seq Scan on employees e  (cost=0.00..1.06 rows=2 width=36)
        Filter: (salary > 70000)
  ->  Hash  (cost=1.04..1.04 rows=4 width=36)
        ->  Seq Scan on departments d  (cost=0.00..1.04 rows=4 width=36)
```

**Why this output appears**: The planner chose a **hash join**: it scans `employees`, filters by salary, builds a hash table from `departments`, and probes the hash table for matching department IDs. The written order of `employees` and `departments` in the `FROM` clause did not dictate the physical order; the optimizer selected the plan it estimated to be cheapest.

#### Real-World Cases

- **Index selection**: A query with a selective `WHERE` clause may use an index scan, while a query returning most of the table may use a sequential scan.
- **Join reordering**: The planner may join the smaller table first to reduce intermediate result sizes.

---

## References

- jOOQ – Lexical and logical SELECT clause order – https://www.jooq.org/doc/latest/manual/sql-building/sql-statements/select-statement/select-lexical-vs-logical-order/
- DbSchema – SQL SELECT Statement – Syntax, Examples, and Query Order – https://dbschema.com/blog/tutorials/sql-select/
- PostgreSQL Documentation – Query Overview – https://www.postgresql.org/docs/current/queries-overview.html
- MySQL 9.7 Reference Manual – SELECT Statement – https://docs.oracle.com/cd/E17952_01/mysql-9.7-en/select.html
- Tiger Data – Understanding DISTINCT in PostgreSQL (With Examples) – https://www.tigerdata.com/learn/understanding-distinct-in-postgresql-with-examples
- PostgreSQL Documentation – SELECT (git source) – https://git.postgresql.org/gitweb/?p=postgresql.git;a=blob_plain;f=doc/src/sgml/ref/select.sgml
- Stack Overflow – What's the execute order of the different parts of a SQL select statement? – https://stackoverflow.com/questions/…
- Microsoft Learn – Logical and Physical Showplan Operators – https://learn.microsoft.com/en-us/sql/relational-databases/showplan-logical-and-physical-operators-reference
- Ask TOM – Table Order – https://asktom.oracle.com/…
- Stack Overflow – Query planner and join order – https://stackoverflow.com/questions/…