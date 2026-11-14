# Comprehensive Programming Cheat Sheet: Window Function Fundamentals

## Topic Overview

### Definitions

**Core Definition**  
A window function is an SQL function that computes a value for each row of a query result by examining a set of rows related to that row, called a window, without collapsing those rows into a single output row.

**Technical Definition**  
A window function is defined by the SQL standard (SQL:2003 and later) as a function whose input is a “window” of one or more rows from the result set of a `SELECT` statement, and whose output is one value per input row. Window functions are syntactically distinguished by an `OVER` clause, which specifies partitioning, ordering, and framing of the window. They operate on the virtual table produced after `FROM`, `WHERE`, `GROUP BY`, and `HAVING` processing, and they are permitted only in the `SELECT` list and the `ORDER BY` clause of a query.

**Beginner-Friendly Explanation**  
A window function is like looking at each row through a special lens that can also see a group of related rows—such as “all rows in the same department” or “all rows up to the current date.” For each row, the function computes something using that group, but the row itself remains in the output. This is different from a regular aggregate, which would collapse the group into one row.

### Key Characteristics

- **Row-preserving**: Unlike aggregate functions, window functions do not reduce the number of rows in the result set.
- **`OVER` clause required**: Every window function call must be followed by an `OVER` clause; without it, an aggregate function behaves as a plain aggregate.
- **Three components**: The `OVER` clause can contain `PARTITION BY` (grouping), `ORDER BY` (ordering within partitions), and a frame specification (`ROWS`, `RANGE`, or `GROUPS`).
- **Post-aggregation evaluation**: Window functions are evaluated after `FROM`, `WHERE`, `GROUP BY`, and `HAVING`, and before `SELECT DISTINCT` and the final `ORDER BY`.
- **Permitted clauses**: Window functions may appear only in the `SELECT` list and the `ORDER BY` clause; they are forbidden in `WHERE`, `GROUP BY`, and `HAVING`.
- **Vendor support**: Supported by PostgreSQL, MySQL 8.0+, SQL Server, Oracle, SQLite 3.25+, and other modern RDBMSs.

### Prerequisites

- Solid understanding of `SELECT`, `WHERE`, `GROUP BY`, and `HAVING`.
- Familiarity with aggregate functions (`SUM`, `AVG`, `COUNT`, `MIN`, `MAX`).
- Knowledge of ordering (`ORDER BY`) and aliasing.
- Awareness of the logical processing order of SQL clauses.

### Related Programming Areas

- Analytical query processing and reporting.
- Data warehousing and OLAP.
- Ranking, running totals, and moving averages.
- Time-series analysis and gap filling.
- Query optimisation and execution plans.

---

## Core Concepts and Features

### 1. What Window Functions Are

#### Definitions

**Core Definition**  
A window function performs a calculation across a set of rows related to the current row, returning a value for each row without grouping the rows into a single output row.

**Technical Definition**  
A window function is an SQL function whose evaluation depends on a window specification. The window is the set of rows from the query’s virtual result set that are related to the current row according to the `OVER` clause. The function computes one result per row. Window functions include ranking functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`), value functions (`LAG`, `LEAD`, `FIRST_VALUE`, `LAST_VALUE`, `NTH_VALUE`), and aggregate functions used with an `OVER` clause (`SUM`, `AVG`, `COUNT`, `MIN`, `MAX`).

**Beginner-Friendly Explanation**  
Imagine a spreadsheet where each row can see a “window” of other rows—perhaps all rows above it, or all rows in the same category—and compute something using that window. The row itself stays in the output, and the computed value is added as a new column.

#### Purposes

- To compute rankings and percentiles without losing row detail.
- To calculate running totals, moving averages, and cumulative sums.
- To compare each row with values from preceding or following rows (`LAG`, `LEAD`).
- To compute per‑group aggregates while retaining individual rows.
- To perform analytical calculations that would otherwise require self‑joins or subqueries.

#### Syntax Rules and Structure

**General syntax**:

```sql
window_function_name ( [expression] ) OVER (
    [ PARTITION BY partition_expression [, ...] ]
    [ ORDER BY sort_expression [ ASC | DESC ] [, ...] ]
    [ frame_clause ]
)
```

**PostgreSQL** (official):

```sql
function_name ( * ) OVER [ window_name ]
function_name ( expression [, ...] ) OVER [ window_name ]
function_name ( expression [, ...] ) OVER ( [ PARTITION BY ... ] [ ORDER BY ... ] [ frame_clause ] )
function_name ( * ) OVER ( [ PARTITION BY ... ] [ ORDER BY ... ] [ frame_clause ] )
```

**SQL Server** (official):

```sql
function_name ( expression ) OVER (
    [ PARTITION BY value_expression [, ...] ]
    [ ORDER BY order_by_expression [ ASC | DESC ] [ , ... ] ]
    [ ROWS | RANGE frame_extent ]
)
```

**MySQL 8.0+** (official):

```sql
function_name ( expression ) OVER (
    [ PARTITION BY expr [, ...] ]
    [ ORDER BY expr [ ASC | DESC ] [, ...] ]
    [ frame_clause ]
)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `function_name` | Ranking, value, or aggregate function. |
| `expression` | Argument to the function (may be omitted for `ROW_NUMBER`, `RANK`, etc.). |
| `OVER` | Mandatory clause introducing the window specification. |
| `PARTITION BY` | Divides rows into groups; the function is applied per partition. |
| `ORDER BY` | Defines row order within each partition. |
| `frame_clause` | Narrows the window to a subset of the partition (`ROWS`, `RANGE`, or `GROUPS`). |

**Syntax rules**

- The `OVER` clause is mandatory for a function to be treated as a window function.
- `PARTITION BY` is optional; without it, the entire result set is one partition.
- `ORDER BY` is optional for some functions (e.g., `SUM` over the whole partition) but required for ranking functions.
- Frame clauses are optional; the default frame depends on the presence of `ORDER BY`.
- Window functions are allowed only in `SELECT` and `ORDER BY`.

**Constraints and limitations**

- Window functions cannot appear in `WHERE`, `GROUP BY`, or `HAVING`; use a subquery or CTE to filter on window results.
- Nested window functions are not allowed in MySQL and are restricted in other engines.
- Some functions (e.g., `ROW_NUMBER`) require `ORDER BY` within `OVER`.
- `DISTINCT` is not allowed in window function arguments in most engines.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Ranking employees within departments

```sql
-- Step 1: Define the query with a window function.
SELECT depname, empno, salary,
       rank() OVER (PARTITION BY depname ORDER BY salary DESC) AS rank
FROM empsalary;
```

**Expected output**:

```text
 depname  | empno | salary | rank
----------+-------+--------+------
 develop  |     8 |   6000 |    1
 develop  |    10 |   5200 |    2
 develop  |    11 |   5200 |    2
 develop  |     9 |   4500 |    4
 develop  |     7 |   4200 |    5
 personnel|     2 |   3900 |    1
 personnel|     5 |   3500 |    2
 sales    |     1 |   5000 |    1
 sales    |     4 |   4800 |    2
 sales    |     3 |   4800 |    2
(10 rows)
```

**Why this output appears**: `PARTITION BY depname` groups rows by department, and `ORDER BY salary DESC` ranks salaries within each department. All original rows remain in the output, each annotated with its rank.

##### Example 2: MySQL 8.0 – Running total with `SUM` as a window function

```sql
SELECT year, country, product, profit,
       SUM(profit) OVER (PARTITION BY country ORDER BY year) AS running_profit
FROM sales;
```

**Expected output** (illustrative):

```text
+------+---------+------------+--------+----------------+
| year | country | product    | profit | running_profit |
+------+---------+------------+--------+----------------+
| 2000 | Finland | Computer   |   1500 |           1500 |
| 2000 | Finland | Phone      |    100 |           1600 |
| 2001 | Finland | Phone      |     10 |           1610 |
| 2000 | India   | Calculator |     75 |             75 |
| 2000 | India   | Calculator |     75 |            150 |
| 2000 | India   | Computer   |   1200 |           1350 |
+------+---------+------------+--------+----------------+
```

**Why this output appears**: `PARTITION BY country` resets the running sum for each country, and `ORDER BY year` accumulates profit year by year. The rows are not collapsed; each retains its individual values alongside the running total.

##### Example 3: SQL Server – `LAG` to compare with the previous row

```sql
SELECT CustomerID, OrderDate, TotalDue,
       LAG(TotalDue, 1) OVER (PARTITION BY CustomerID ORDER BY OrderDate) AS PreviousTotal
FROM Sales.SalesOrderHeader;
```

**Expected output** (illustrative):

```text
CustomerID OrderDate   TotalDue PreviousTotal
---------- ----------- -------- -------------
1          2025-01-01  100.00   NULL
1          2025-02-01  150.00   100.00
1          2025-03-01  200.00   150.00
2          2025-01-15   50.00   NULL
```

**Why this output appears**: `LAG(TotalDue, 1)` retrieves the `TotalDue` value from the previous row within the same `CustomerID` partition, ordered by `OrderDate`. The first row of each partition has no previous row, so it returns `NULL`.

#### Real-World Cases

- **Sales dashboards**: Ranking products by revenue within each region while keeping every product row.
- **Financial reporting**: Computing running balances per account.
- **HR analytics**: Comparing each employee’s salary to the departmental average without collapsing the department.

---

### 2. Window vs Aggregate Functions

#### Definitions

**Core Definition**  
An aggregate function collapses a group of rows into a single result row, while a window function computes a value for each row using a window of related rows and preserves all rows.

**Technical Definition**  
Aggregate functions (`SUM`, `AVG`, `COUNT`, `MIN`, `MAX`) without an `OVER` clause group rows according to `GROUP BY` and produce one row per group. When an aggregate function is followed by an `OVER` clause, it becomes a window function: it computes the aggregate over the window frame for each row and returns one value per row, without reducing the row count. Window functions execute after non‑window aggregates, so aggregate results can be passed as arguments to window functions, but not vice versa.

**Beginner-Friendly Explanation**  
A regular aggregate is like summarising a whole class into one average grade—you lose the individual students. A window function is like writing each student’s grade next to the class average—every student remains, and the summary is attached to each row.

#### Purposes

- To choose between summarised output (aggregates) and row‑level enriched output (window functions).
- To compute both detail and summary in one query without self‑joins.
- To understand when an aggregate function can be repurposed as a window function via `OVER`.
- To avoid accidental row loss when analytical detail is required.

#### Syntax Rules and Structure

**Aggregate function** (without `OVER`):

```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

**Aggregate function as window function** (with `OVER`):

```sql
SELECT department, employee, salary,
       AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM employees;
```

**Component breakdown**

| Form | Row count | Syntax |
|------|-----------|--------|
| Aggregate | One row per group | `function(...)` with `GROUP BY` |
| Window | Same as input rows | `function(...) OVER (...)` |

**Syntax rules**

- An aggregate without `OVER` collapses rows; with `OVER` it does not.
- `GROUP BY` is not used with window functions in the same query block unless the query also performs grouping.
- Window functions execute after `GROUP BY` and `HAVING`, so they can reference aggregate results.
- To filter on a window function result, wrap the query in a subquery or CTE.

**Constraints and limitations**

- Mixing grouped aggregates and window functions in one `SELECT` requires care: the window function operates on the grouped result, not the original rows.
- `DISTINCT` is not allowed in window function arguments in many engines.
- Window functions cannot be used in `WHERE` or `HAVING`; filter after the window computation.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Aggregate vs window side by side

```sql
-- Aggregate: one row per department.
SELECT depname, AVG(salary) AS avg_salary
FROM empsalary
GROUP BY depname;

-- Window: one row per employee, with department average attached.
SELECT depname, empno, salary,
       AVG(salary) OVER (PARTITION BY depname) AS dept_avg
FROM empsalary;
```

**Expected output** (aggregate):

```text
 depname  |      avg_salary
----------+---------------------
 develop  | 5020.0000000000000000
 personnel| 3700.0000000000000000
 sales    | 4866.6666666666666667
(3 rows)
```

**Expected output** (window):

```text
 depname  | empno | salary |      dept_avg
----------+-------+--------+---------------------
 develop  |    11 |   5200 | 5020.0000000000000000
 develop  |     7 |   4200 | 5020.0000000000000000
 develop  |     9 |   4500 | 5020.0000000000000000
 develop  |     8 |   6000 | 5020.0000000000000000
 develop  |    10 |   5200 | 5020.0000000000000000
 personnel|     5 |   3500 | 3700.0000000000000000
 personnel|     2 |   3900 | 3700.0000000000000000
 sales    |     3 |   4800 | 4866.6666666666666667
 sales    |     1 |   5000 | 4866.6666666666666667
 sales    |     4 |   4800 | 4866.6666666666666667
(10 rows)
```

**Why this output appears**: The aggregate query collapses each department into one row; the window query preserves every employee row and attaches the department average to each.

##### Example 2: MySQL – `SUM` as aggregate and as window

```sql
-- Aggregate: total profit for all sales.
SELECT SUM(profit) AS total_profit FROM sales;

-- Window: per-row profit plus country-level total.
SELECT year, country, product, profit,
       SUM(profit) OVER (PARTITION BY country) AS country_profit
FROM sales;
```

**Expected output** (aggregate):

```text
+--------------+
| total_profit |
+--------------+
|         7535 |
+--------------+
```

**Expected output** (window):

```text
+------+---------+------------+--------+----------------+
| year | country | product    | profit | country_profit |
+------+---------+------------+--------+----------------+
| 2000 | Finland | Computer   |   1500 |           1610 |
| 2000 | Finland | Phone      |    100 |           1610 |
| 2001 | Finland | Phone      |     10 |           1610 |
| 2000 | India   | Calculator |     75 |           1350 |
| 2000 | India   | Calculator |     75 |           1350 |
| 2000 | India   | Computer   |   1200 |           1350 |
+------+---------+------------+--------+----------------+
```

**Why this output appears**: The aggregate returns a single global sum; the window function returns the country sum on every row of that country.

#### Real-World Cases

- **Dashboards**: Showing each transaction alongside the account total.
- **Comparative analysis**: Comparing each row to a group average without losing detail.
- **Data quality checks**: Flagging outliers by comparing individual values to group statistics.

---

### 3. `OVER` Clause

#### Definitions

**Core Definition**  
The `OVER` clause defines the window—partitioning, ordering, and framing—over which a window function operates.

**Technical Definition**  
The `OVER` clause follows the window function call and contains an optional `PARTITION BY` list, an optional `ORDER BY` list, and an optional frame clause. It determines how the rows of the query’s virtual result set are grouped into partitions, ordered within partitions, and bounded into frames. The `OVER` clause is the syntactic marker that distinguishes a window function call from an ordinary function or aggregate call.

**Beginner-Friendly Explanation**  
The `OVER` clause is the instruction manual for the window: it says which rows belong together (`PARTITION BY`), how they are sorted (`ORDER BY`), and which subset of rows the function should actually look at (the frame).

#### Purposes

- To specify how rows are grouped for window computation.
- To control the order in which rows are processed within each partition.
- To narrow the window to a frame for cumulative or moving calculations.
- To reuse a named window specification across multiple functions via the `WINDOW` clause.

#### Syntax Rules and Structure

**General syntax**:

```sql
function_name(...) OVER (
    [ PARTITION BY expression [, ...] ]
    [ ORDER BY expression [ ASC | DESC ] [, ...] ]
    [ { ROWS | RANGE | GROUPS } frame_clause ]
)
```

**Named window**:

```sql
SELECT sum(salary) OVER w, avg(salary) OVER w
FROM empsalary
WINDOW w AS (PARTITION BY depname ORDER BY salary DESC);
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `PARTITION BY` | Divides rows into partitions; function resets per partition. |
| `ORDER BY` | Orders rows within each partition. |
| Frame clause | Narrows the window to a subset of partition rows. |
| `WINDOW` clause | Defines a reusable named window specification. |

**Syntax rules**

- `OVER` is mandatory for window functions.
- `PARTITION BY` and `ORDER BY` are optional, but ranking functions require `ORDER BY`.
- The `WINDOW` clause allows naming a specification and referencing it in multiple `OVER` clauses.
- `OVER wname` and `OVER (wname)` are not always equivalent; the latter copies and may modify the specification.

**Constraints and limitations**

- The `OVER` clause cannot contain subqueries in most engines.
- `ORDER BY` in `OVER` is independent of the query’s final `ORDER BY`.
- Frame clauses are not allowed with all functions (e.g., `ROW_NUMBER` does not use a frame).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – `OVER` with partitioning and ordering

```sql
SELECT depname, empno, salary,
       rank() OVER (PARTITION BY depname ORDER BY salary DESC) AS rank
FROM empsalary;
```

**Expected output**:

```text
 depname  | empno | salary | rank
----------+-------+--------+------
 develop  |     8 |   6000 |    1
 develop  |    10 |   5200 |    2
 develop  |    11 |   5200 |    2
 develop  |     9 |   4500 |    4
 develop  |     7 |   4200 |    5
 personnel|     2 |   3900 |    1
 personnel|     5 |   3500 |    2
 sales    |     1 |   5000 |    1
 sales    |     4 |   4800 |    2
 sales    |     3 |   4800 |    2
(10 rows)
```

**Why this output appears**: `PARTITION BY depname` groups employees by department, and `ORDER BY salary DESC` ranks them within each department.

##### Example 2: MySQL – `OVER` with only `PARTITION BY`

```sql
SELECT year, country, product, profit,
       SUM(profit) OVER (PARTITION BY country) AS country_profit
FROM sales;
```

**Expected output**:

```text
+------+---------+------------+--------+----------------+
| year | country | product    | profit | country_profit |
+------+---------+------------+--------+----------------+
| 2000 | Finland | Computer   |   1500 |           1610 |
| 2000 | Finland | Phone      |    100 |           1610 |
| 2001 | Finland | Phone      |     10 |           1610 |
| 2000 | India   | Calculator |     75 |           1350 |
| 2000 | India   | Calculator |     75 |           1350 |
| 2000 | India   | Computer   |   1200 |           1350 |
+------+---------+------------+--------+----------------+
```

**Why this output appears**: Without `ORDER BY`, the window frame defaults to the entire partition, so the country total is repeated on every row of that country.

##### Example 3: PostgreSQL – Named `WINDOW` clause

```sql
SELECT sum(salary) OVER w AS sum_salary,
       avg(salary) OVER w AS avg_salary
FROM empsalary
WINDOW w AS (PARTITION BY depname ORDER BY salary DESC);
```

**Expected output** (illustrative):

```text
 sum_salary | avg_salary
------------+------------
       6000 |    6000.00
      11200 |    5600.00
      16400 |    5466.67
      20900 |    5225.00
      25100 |    5020.00
       3900 |    3900.00
       7400 |    3700.00
       5000 |    5000.00
       9800 |    4900.00
      14600 |    4866.67
(10 rows)
```

**Why this output appears**: The named window `w` is reused by both `SUM` and `AVG`, avoiding duplication and ensuring identical partitioning and ordering.

#### Real-World Cases

- **Reusable specifications**: Defining one window for multiple analytical columns.
- **Per‑group calculations**: Partitioning by region, department, or customer.
- **Ordered analytics**: Ranking and cumulative calculations require `ORDER BY`.

---

### 4. Window Partitioning

#### Definitions

**Core Definition**  
Window partitioning divides the query’s result set into independent groups (partitions) within which the window function is evaluated separately.

**Technical Definition**  
`PARTITION BY` specifies one or more expressions; rows with equal values for all expressions form a partition. The window function resets its computation for each partition. Without `PARTITION BY`, the entire result set is treated as a single partition. Partitioning is analogous to `GROUP BY` but does not collapse rows; it only defines the scope for window evaluation.

**Beginner-Friendly Explanation**  
Partitioning is like splitting a class into groups by team; calculations like “team average” are done per team, and the results are attached to each student in that team.

#### Purposes

- To compute per‑group metrics while retaining individual rows.
- To reset running totals or rankings for each group.
- To isolate analytical calculations by category, region, or time period.
- To avoid self‑joins when group‑level context is needed per row.

#### Syntax Rules and Structure

**Syntax**:

```sql
function_name(...) OVER (PARTITION BY expression [, ...])
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `PARTITION BY` | Keyword introducing partition expressions. |
| `expression` | Column or expression defining partition membership. |

**Syntax rules**

- Multiple expressions create finer partitions.
- `PARTITION BY` is optional; omitting it means one global partition.
- Partition expressions are evaluated on the virtual table after `WHERE` and `GROUP BY`.
- `PARTITION BY` cannot reference output column aliases in most engines.

**Constraints and limitations**

- Large numbers of partitions can increase memory and sort overhead.
- Partitioning does not guarantee output order; a separate `ORDER BY` is needed for presentation.
- Some engines restrict `PARTITION BY` expressions in certain contexts.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Partitioning by department

```sql
SELECT depname, empno, salary,
       AVG(salary) OVER (PARTITION BY depname) AS dept_avg
FROM empsalary;
```

**Expected output**:

```text
 depname  | empno | salary |      dept_avg
----------+-------+--------+---------------------
 develop  |    11 |   5200 | 5020.0000000000000000
 develop  |     7 |   4200 | 5020.0000000000000000
 develop  |     9 |   4500 | 5020.0000000000000000
 develop  |     8 |   6000 | 5020.0000000000000000
 develop  |    10 |   5200 | 5020.0000000000000000
 personnel|     5 |   3500 | 3700.0000000000000000
 personnel|     2 |   3900 | 3700.0000000000000000
 sales    |     3 |   4800 | 4866.6666666666666667
 sales    |     1 |   5000 | 4866.6666666666666667
 sales    |     4 |   4800 | 4866.6666666666666667
(10 rows)
```

**Why this output appears**: Each department forms a partition; `AVG(salary)` is computed per department and attached to every employee in that department.

##### Example 2: MySQL – Partitioning by country

```sql
SELECT year, country, product, profit,
       SUM(profit) OVER (PARTITION BY country) AS country_profit
FROM sales;
```

**Expected output**:

```text
+------+---------+------------+--------+----------------+
| year | country | product    | profit | country_profit |
+------+---------+------------+--------+----------------+
| 2000 | Finland | Computer   |   1500 |           1610 |
| 2000 | Finland | Phone      |    100 |           1610 |
| 2001 | Finland | Phone      |     10 |           1610 |
| 2000 | India   | Calculator |     75 |           1350 |
| 2000 | India   | Calculator |     75 |           1350 |
| 2000 | India   | Computer   |   1200 |           1350 |
+------+---------+------------+--------+----------------+
```

**Why this output appears**: The `country` column partitions the data; the sum resets for each country.

##### Example 3: SQL Server – No partitioning (global window)

```sql
SELECT CustomerID, OrderDate, TotalDue,
       SUM(TotalDue) OVER () AS GrandTotal
FROM Sales.SalesOrderHeader;
```

**Expected output** (illustrative):

```text
CustomerID OrderDate   TotalDue GrandTotal
---------- ----------- -------- ----------
1          2025-01-01  100.00   350.00
1          2025-02-01  150.00   350.00
2          2025-01-15   50.00   350.00
2          2025-02-15   50.00   350.00
```

**Why this output appears**: Without `PARTITION BY`, the entire result set is one partition, so `GrandTotal` is the same on every row.

#### Real-World Cases

- **Departmental averages**: Attaching department averages to each employee.
- **Regional sales totals**: Showing each sale alongside its region’s total.
- **Account balances**: Resetting running balances per account.

---

### 5. Window Ordering

#### Definitions

**Core Definition**  
Window ordering defines the sequence in which rows within each partition are processed by the window function.

**Technical Definition**  
`ORDER BY` inside the `OVER` clause specifies the logical order of rows within each partition. It is independent of the query’s final `ORDER BY`. Ordering is required for ranking functions and for cumulative calculations; it also determines the default frame. Rows that are equal according to the `ORDER BY` expressions are called peers and are treated as equivalent by ranking functions.

**Beginner-Friendly Explanation**  
Ordering tells the window function which row comes “first,” “next,” and “last” within each group, which matters for things like running totals and rankings.

#### Purposes

- To define the sequence for ranking and cumulative calculations.
- To ensure deterministic results when ties exist.
- To control the default frame when no explicit frame is given.
- To order rows for functions like `LAG`, `LEAD`, `FIRST_VALUE`, and `LAST_VALUE`.

#### Syntax Rules and Structure

**Syntax**:

```sql
function_name(...) OVER (PARTITION BY ... ORDER BY expression [ ASC | DESC ] [, ...])
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `ORDER BY` | Keyword introducing sort expressions within the window. |
| `expression` | Column or expression to sort by. |
| `ASC` / `DESC` | Sort direction (default `ASC`). |

**Syntax rules**

- `ORDER BY` inside `OVER` is independent of the query’s outer `ORDER BY`.
- Ranking functions (`RANK`, `DENSE_RANK`, `ROW_NUMBER`) require `ORDER BY`.
- Without `ORDER BY`, rows are processed in an unspecified order.
- Ties (peers) affect ranking functions: `RANK` skips ranks after ties, `DENSE_RANK` does not.
- The default frame with `ORDER BY` is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`.

**Constraints and limitations**

- Non‑deterministic ordering (ties without a tiebreaker) can produce unstable results.
- `ORDER BY` in `OVER` does not guarantee final output order.
- Some engines restrict the use of certain expressions in window `ORDER BY`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Ordering for ranking

```sql
SELECT depname, empno, salary,
       rank() OVER (PARTITION BY depname ORDER BY salary DESC) AS rank
FROM empsalary;
```

**Expected output**:

```text
 depname  | empno | salary | rank
----------+-------+--------+------
 develop  |     8 |   6000 |    1
 develop  |    10 |   5200 |    2
 develop  |    11 |   5200 |    2
 develop  |     9 |   4500 |    4
 develop  |     7 |   4200 |    5
 personnel|     2 |   3900 |    1
 personnel|     5 |   3500 |    2
 sales    |     1 |   5000 |    1
 sales    |     4 |   4800 |    2
 sales    |     3 |   4800 |    2
(10 rows)
```

**Why this output appears**: `ORDER BY salary DESC` orders employees within each department; `RANK` assigns 1 to the highest salary, with ties sharing the same rank and the next rank skipped.

##### Example 2: PostgreSQL – Running total with `ORDER BY`

```sql
SELECT salary,
       sum(salary) OVER (ORDER BY salary) AS running_sum
FROM empsalary;
```

**Expected output**:

```text
 salary | running_sum
--------+-------------
   3500 |        3500
   3900 |        7400
   4200 |       11600
   4500 |       16100
   4800 |       25700
   4800 |       25700
   5000 |       30700
   5200 |       41100
   5200 |       41100
   6000 |       47100
(10 rows)
```

**Why this output appears**: Without partitioning, the entire table is one partition; `ORDER BY salary` creates a cumulative sum from the lowest salary through the current row, including peers.

##### Example 3: MySQL – `ORDER BY` with `LAG`

```sql
SELECT year, country, profit,
       LAG(profit) OVER (PARTITION BY country ORDER BY year) AS prev_profit
FROM sales;
```

**Expected output** (illustrative):

```text
+------+---------+--------+-------------+
| year | country | profit | prev_profit |
+------+---------+--------+-------------+
| 2000 | Finland |   1500 |        NULL |
| 2000 | Finland |    100 |        1500 |
| 2001 | Finland |     10 |         100 |
| 2000 | India   |     75 |        NULL |
| 2000 | India   |     75 |          75 |
| 2000 | India   |   1200 |          75 |
+------+---------+--------+-------------+
```

**Why this output appears**: `ORDER BY year` within each country determines which row is “previous”; `LAG` retrieves the prior profit value.

#### Real-World Cases

- **Rankings**: Leaderboards with ties.
- **Cumulative metrics**: Running totals ordered by date.
- **Comparisons**: Previous/next values for trend analysis.

---

### 6. Window Frames

#### Definitions

**Core Definition**  
A window frame is the subset of rows within a partition that a window function actually uses for a given row.

**Technical Definition**  
The frame clause specifies the start and end of the frame relative to the current row. It can use `ROWS` (physical offset in rows), `RANGE` (logical offset based on `ORDER BY` values), or `GROUPS` (peer groups). The default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` when `ORDER BY` is present, and the entire partition when `ORDER BY` is absent. Frame clauses are supported by PostgreSQL, SQL Server, Oracle, MySQL 8.0+, and SQLite.

**Beginner-Friendly Explanation**  
A frame is the actual slice of rows the function looks at for the current row—for example, “all rows from the start up to this row” (running total) or “this row and the two before it” (moving average).

#### Purposes

- To compute running totals and cumulative aggregates.
- To compute moving averages over a fixed number of rows.
- To restrict calculations to a logical range of ordered values.
- To include or exclude peers and boundary rows.

#### Syntax Rules and Structure

**General syntax**:

```sql
{ ROWS | RANGE | GROUPS } BETWEEN frame_start AND frame_end
```

**Frame bounds**:

```text
UNBOUNDED PRECEDING
n PRECEDING
CURRENT ROW
n FOLLOWING
UNBOUNDED FOLLOWING
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `ROWS` | Physical row offset. |
| `RANGE` | Logical offset based on `ORDER BY` values. |
| `GROUPS` | Offset in peer groups. |
| `BETWEEN ... AND ...` | Defines start and end bounds. |
| `UNBOUNDED PRECEDING` | Start of partition. |
| `UNBOUNDED FOLLOWING` | End of partition. |
| `CURRENT ROW` | Current row (or current peer group in `RANGE`). |

**Syntax rules**

- Frame clauses are optional; the default depends on `ORDER BY`.
- `ROWS` uses a fixed number of physical rows; `RANGE` uses value‑based offsets.
- `GROUPS` counts peer groups rather than rows.
- `EXCLUDE` clauses (`CURRENT ROW`, `GROUP`, `TIES`, `NO OTHERS`) are supported in PostgreSQL 11+, Oracle, and some other engines.
- `frame_start` cannot be `UNBOUNDED FOLLOWING`; `frame_end` cannot be `UNBOUNDED PRECEDING`.

**Constraints and limitations**

- `RANGE` with numeric offsets requires exactly one `ORDER BY` column in many engines.
- `GROUPS` is not universally supported (e.g., older MySQL versions).
- Frame clauses are ignored by functions that do not use frames (e.g., `ROW_NUMBER`).
- The default frame with `ORDER BY` includes peers, which can surprise users expecting strict row‑by‑row running totals; use `ROWS` to avoid peer inclusion.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Default frame vs explicit `ROWS`

```sql
-- Default frame with ORDER BY: includes peers.
SELECT salary,
       sum(salary) OVER (ORDER BY salary) AS default_sum
FROM empsalary;

-- Explicit ROWS frame: strictly row-by-row.
SELECT salary,
       sum(salary) OVER (ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS rows_sum
FROM empsalary;
```

**Expected output** (default frame):

```text
 salary | default_sum
--------+-------------
   3500 |        3500
   3900 |        7400
   4200 |       11600
   4500 |       16100
   4800 |       25700
   4800 |       25700
   5000 |       30700
   5200 |       41100
   5200 |       41100
   6000 |       47100
(10 rows)
```

**Expected output** (`ROWS` frame):

```text
 salary | rows_sum
--------+----------
   3500 |     3500
   3900 |     7400
   4200 |    11600
   4500 |    16100
   4800 |    20900
   4800 |    25700
   5000 |    30700
   5200 |    35900
   5200 |    41100
   6000 |    47100
(10 rows)
```

**Why this output appears**: The default `RANGE` frame includes all peers with the same salary; the `ROWS` frame includes exactly the rows up to the current physical row, so the two 4800 rows produce different running totals.

##### Example 2: MySQL – Moving average with `ROWS BETWEEN`

```sql
SELECT year, profit,
       AVG(profit) OVER (ORDER BY year ROWS BETWEEN 1 PRECEDING AND CURRENT ROW) AS moving_avg
FROM sales;
```

**Expected output** (illustrative):

```text
+------+--------+------------+
| year | profit | moving_avg |
+------+--------+------------+
| 2000 |   1500 |    1500.00 |
| 2000 |    100 |     800.00 |
| 2001 |     10 |      55.00 |
+------+--------+------------+
```

**Why this output appears**: The frame includes the current row and the one preceding it, so the average is computed over at most two rows.

##### Example 3: PostgreSQL – `RANGE` frame with logical offset

```sql
SELECT salary,
       sum(salary) OVER (ORDER BY salary RANGE BETWEEN 500 PRECEDING AND CURRENT ROW) AS range_sum
FROM empsalary;
```

**Expected output** (illustrative):

```text
 salary | range_sum
--------+-----------
   3500 |      3500
   3900 |      7400
   4200 |     11600
   4500 |     16100
   4800 |     20900
   4800 |     20900
   5000 |     30700
   5200 |     35900
   5200 |     35900
   6000 |     41100
```

**Why this output appears**: The `RANGE` frame includes rows whose `ORDER BY` value is within 500 of the current row’s value, so it operates on value proximity rather than physical row counts.

#### Real-World Cases

- **Running totals**: Cumulative sums with `ROWS` for strict row order.
- **Moving averages**: Sliding windows over time or sequence.
- **Peer‑aware calculations**: `RANGE` frames that include all rows with the same value.

---

### 7. Window Function Execution

#### Definitions

**Core Definition**  
Window function execution is the phase in the logical processing of a query where window functions are evaluated, after grouping and aggregation but before final projection, distinct, and ordering.

**Technical Definition**  
Logically, a `SELECT` statement processes `FROM`, `WHERE`, `GROUP BY`, and `HAVING` first, producing a virtual table of rows. Window functions are then evaluated over this virtual table, using the window specifications in their `OVER` clauses. After window evaluation, the `SELECT` list is computed, `DISTINCT` is applied, and finally `ORDER BY` and `LIMIT` are processed. Window functions execute after non‑window aggregates, so aggregate results may appear as arguments to window functions, but not vice versa.

**Beginner-Friendly Explanation**  
The database first filters and groups the data, then it calculates all the window function results, and only then does it produce the final columns and sort the output. This is why you cannot use a window function directly in `WHERE`.

#### Purposes

- To understand why window functions cannot appear in `WHERE` or `HAVING`.
- To know when to wrap a window function in a subquery or CTE for filtering.
- To reason about the interaction between `GROUP BY`, window functions, and `DISTINCT`.
- To predict the order of evaluation in complex queries.

#### Syntax Rules and Structure

**Logical order** (conceptual):

```text
FROM → WHERE → GROUP BY → HAVING → WINDOW → SELECT → DISTINCT → ORDER BY → LIMIT
```

**Component breakdown**

| Phase | What happens |
|-------|--------------|
| `FROM` | Tables are joined and the virtual table is formed. |
| `WHERE` | Rows are filtered. |
| `GROUP BY` | Rows are grouped. |
| `HAVING` | Groups are filtered. |
| `WINDOW` | Window functions are evaluated. |
| `SELECT` | Projection expressions are computed. |
| `DISTINCT` | Duplicate rows are removed. |
| `ORDER BY` | Final ordering. |
| `LIMIT` | Row count limited. |

**Syntax rules**

- Window functions are allowed only in `SELECT` and `ORDER BY` because those are evaluated after the window phase.
- To filter on a window result, use a subquery or CTE: compute the window function in the inner query, filter in the outer query.
- Window functions execute after aggregates, so `SUM(amount) OVER (...)` can use an aggregate result, but `SUM(SUM(amount)) OVER (...)` requires the inner aggregate to be in a subquery.

**Constraints and limitations**

- The actual execution plan may reorder operations for performance, but the logical semantics follow the order above.
- `QUALIFY` (supported in some engines like Snowflake, Teradata, and BigQuery) allows filtering on window functions after their evaluation, but is not standard in PostgreSQL, MySQL, or SQL Server.
- Window functions in `ORDER BY` are evaluated after the `SELECT` list in some engines, which can affect aliasing.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Filtering on a window function via a subquery

```sql
-- Direct filtering is not allowed:
-- SELECT depname, rank() OVER (...) FROM empsalary WHERE rank() < 3;  -- ERROR

-- Correct approach: subquery or CTE.
SELECT depname, empno, salary, pos
FROM (
    SELECT depname, empno, salary,
           rank() OVER (PARTITION BY depname ORDER BY salary DESC) AS pos
    FROM empsalary
) AS ss
WHERE pos < 3;
```

**Expected output**:

```text
 depname  | empno | salary | pos
----------+-------+--------+-----
 develop  |     8 |   6000 |   1
 develop  |    10 |   5200 |   2
 develop  |    11 |   5200 |   2
 personnel|     2 |   3900 |   1
 personnel|     5 |   3500 |   2
 sales    |     1 |   5000 |   1
 sales    |     4 |   4800 |   2
 sales    |     3 |   4800 |   2
(8 rows)
```

**Why this output appears**: The inner query computes the rank; the outer query filters on the computed `pos` column, which is legal because the window function has already been evaluated.

##### Example 2: MySQL – Window function after `GROUP BY`

```sql
SELECT country, SUM(profit) AS total_profit,
       RANK() OVER (ORDER BY SUM(profit) DESC) AS profit_rank
FROM sales
GROUP BY country;
```

**Expected output** (illustrative):

```text
+---------+--------------+-------------+
| country | total_profit | profit_rank |
+---------+--------------+-------------+
| USA     |         4575 |           1 |
| Finland |         1610 |           2 |
| India   |         1350 |           3 |
+---------+--------------+-------------+
```

**Why this output appears**: `GROUP BY` first aggregates profit per country; the window function `RANK` then ranks the aggregated results. The window function sees the grouped result, not the original rows.

##### Example 3: SQL Server – Window function and `DISTINCT`

```sql
SELECT DISTINCT CustomerID,
       SUM(TotalDue) OVER (PARTITION BY CustomerID) AS CustomerTotal
FROM Sales.SalesOrderHeader;
```

**Expected output** (illustrative):

```text
CustomerID CustomerTotal
---------- -------------
1          450.00
2          100.00
```

**Why this output appears**: The window function is evaluated before `DISTINCT`, so the `CustomerTotal` is computed on all rows; `DISTINCT` then removes duplicate `(CustomerID, CustomerTotal)` pairs.

#### Real-World Cases

- **Top‑N per group**: Computing ranks in a subquery and filtering in the outer query.
- **Grouped analytics**: Applying window functions to aggregated results.
- **Deduplication**: Understanding how `DISTINCT` interacts with window results.

---

### 8. Window Function Performance

#### Definitions

**Core Definition**  
Window function performance is the cost of evaluating window functions, influenced by partitioning, ordering, frame size, indexing, and memory usage.

**Technical Definition**  
Window function evaluation typically requires sorting the data by the `PARTITION BY` and `ORDER BY` columns, maintaining a working set of rows, and computing the function per row. Performance depends on the availability of supporting indexes that match the partitioning and ordering, the size of partitions, the frame type (`ROWS` is often cheaper than `RANGE`), and the engine’s execution mode (row mode vs batch mode). Optimisations include composite indexes on `(PARTITION BY columns, ORDER BY columns)`, avoiding unnecessary sorting, and using `ROWS` frames when peer handling is not needed.

**Beginner-Friendly Explanation**  
Window functions can be slow if the database has to sort a lot of data or keep large windows in memory. Indexes that match the way you partition and order the data can make them much faster.

#### Purposes

- To reduce sorting and memory overhead in window queries.
- To choose indexes that support partitioning and ordering.
- To select frame types that minimise computational cost.
- To avoid performance pitfalls such as unnecessary `RANGE` frames or missing indexes.

#### Syntax Rules and Structure

**Index pattern** (SQL Server example):

```sql
CREATE INDEX IX_SalesOrderHeader_Customer_OrderDate
ON Sales.SalesOrderHeader (CustomerID, OrderDate)
INCLUDE (TotalDue);
```

**Query pattern**:

```sql
SELECT CustomerID, OrderDate, SUM(TotalDue) OVER (
    PARTITION BY CustomerID ORDER BY OrderDate
) AS RunningTotal
FROM Sales.SalesOrderHeader;
```

**Component breakdown**

| Factor | Impact |
|--------|--------|
| Composite index on `(PARTITION BY, ORDER BY)` | Avoids sorting and enables streaming. |
| Covering index with included columns | Avoids key lookups. |
| `ROWS` frame instead of `RANGE` | Avoids peer processing and can be faster. |
| Small partitions | Reduce memory per partition. |
| Batch mode execution (SQL Server) | Can speed up window aggregates. |

**Syntax rules**

- Index key order should match `PARTITION BY` columns followed by `ORDER BY` columns.
- Include columns used in the window function to make the index covering.
- Use `ROWS` frames when peer inclusion is not required; PostgreSQL has shown significant speedups with `ROWS` over `RANGE`.
- Avoid selecting unnecessary columns that increase sort width.

**Constraints and limitations**

- Indexes cannot always eliminate sorting if the query’s `ORDER BY` differs from the window’s `ORDER BY`.
- Large partitions may spill to disk.
- Batch mode is not available in all editions or configurations.
- Too many indexes can slow down writes.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Supporting index for a running total

```sql
-- Step 1: Create a supporting index.
CREATE INDEX IX_SalesOrderHeader_Customer_OrderDate
ON Sales.SalesOrderHeader (CustomerID, OrderDate)
INCLUDE (TotalDue);

-- Step 2: Query with a window function that matches the index.
SELECT CustomerID, OrderDate, SUM(TotalDue) OVER (
    PARTITION BY CustomerID ORDER BY OrderDate
) AS RunningTotal
FROM Sales.SalesOrderHeader;
```

**Expected output** (illustrative):

```text
CustomerID OrderDate   RunningTotal
---------- ----------- ------------
1          2025-01-01  100.00
1          2025-02-01  250.00
1          2025-03-01  450.00
2          2025-01-15   50.00
2          2025-02-15  100.00
```

**Why this output appears**: The index `(CustomerID, OrderDate)` matches the `PARTITION BY CustomerID ORDER BY OrderDate` specification, allowing the engine to scan the index in order and compute the running total without a separate sort.

##### Example 2: PostgreSQL – `ROWS` vs `RANGE` frame performance

```sql
-- RANGE frame (default with ORDER BY): includes peers.
EXPLAIN (ANALYZE, COSTS OFF)
SELECT salary, sum(salary) OVER (ORDER BY salary) AS running_sum
FROM empsalary;

-- ROWS frame: strictly row-by-row, often faster.
EXPLAIN (ANALYZE, COSTS OFF)
SELECT salary, sum(salary) OVER (ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_sum
FROM empsalary;
```

**Expected output** (plans, illustrative):

```text
-- RANGE: WindowAgg with peer checking
 WindowAgg (actual time=0.050..0.120 rows=10 loops=1)
   ->  Sort (actual time=0.030..0.040 rows=10 loops=1)
         Sort Key: salary

-- ROWS: WindowAgg without peer checking
 WindowAgg (actual time=0.030..0.080 rows=10 loops=1)
   ->  Sort (actual time=0.020..0.030 rows=10 loops=1)
         Sort Key: salary
```

**Why this output appears**: `ROWS` avoids peer detection and can be faster; PostgreSQL discussions report up to a 344% improvement using `ROWS` over `RANGE` in some workloads.

##### Example 3: MySQL – Index for `PARTITION BY` and `ORDER BY`

```sql
CREATE INDEX idx_sales_country_year ON sales (country, year);

SELECT year, country, profit,
       SUM(profit) OVER (PARTITION BY country ORDER BY year) AS running_profit
FROM sales;
```

**Expected output**:

```text
+------+---------+--------+----------------+
| year | country | profit | running_profit |
+------+---------+--------+----------------+
| 2000 | Finland |   1500 |           1500 |
| 2000 | Finland |    100 |           1600 |
| 2001 | Finland |     10 |           1610 |
| 2000 | India   |     75 |             75 |
| 2000 | India   |     75 |            150 |
| 2000 | India   |   1200 |           1350 |
+------+---------+--------+----------------+
```

**Why this output appears**: The index `(country, year)` matches the window specification, reducing the need for a separate sort.

#### Real-World Cases

- **Large fact tables**: Composite indexes on partitioning and ordering columns.
- **Time‑series analytics**: `ROWS` frames for moving windows over timestamps.
- **Batch reporting**: Using batch mode in SQL Server for window aggregates.

---

## References

- PostgreSQL Documentation – Window Functions (Tutorial) – https://www.postgresql.org/docs/current/tutorial-window.html
- PostgreSQL Documentation – Window Functions (Reference) – https://www.postgresql.org/docs/current/functions-window.html
- PostgreSQL Documentation – `SELECT` (Window Function Syntax) – https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-WINDOW-FUNCTIONS
- MySQL 8.4 Reference Manual – Window Function Concepts and Syntax – https://dev.mysql.com/doc/refman/8.4/en/window-functions-usage.html
- MySQL 8.4 Reference Manual – Window Function Frame Specification – https://dev.mysql.com/doc/refman/8.4/en/window-functions-frames.html
- Microsoft Learn – `OVER` Clause (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql
- Oracle Database Documentation – Window Function Concepts and Syntax – https://docs.oracle.com/en/database/oracle/oracle-database/23/sqlrf/SELECT.html
- SQLite Documentation – Window Functions – https://www.sqlite.org/windowfunctions.html
- SQL:2003 Standard – Window Functions (ISO/IEC 9075-2:2003)
- Use The Index, Luke – Window Functions and Indexing – https://use-the-index-luke.com/sql/partial-results/window-functions
- Erik Darling Data – Indexing SQL Server Queries For Performance: Windowing Functions – https://erikdarling.com/indexing-sql-server-queries-for-performance-windowing-functions/
- jOOQ – Lexical and Logical SELECT Clause Order – https://www.jooq.org/doc/latest/manual/sql-building/sql-statements/select-statement/select-lexical-vs-logical-order/