# Comprehensive Programming Cheat Sheet: Window Partitioning

## Topic Overview

### Definitions

**Core Definition**  
Window partitioning is the division of a query’s result set into independent groups, called partitions, within which a window function is evaluated separately for each row.

**Technical Definition**  
Window partitioning is specified by the `PARTITION BY` clause inside the `OVER` clause of a window function. It groups rows that share the same values for the specified partitioning expressions. Each partition is processed independently: the window function’s computation resets at partition boundaries, and frames are defined relative to the current row within its own partition. Partitioning is defined by the SQL standard as part of the window specification and is supported by PostgreSQL, MySQL 8.0+, SQL Server, Oracle, and SQLite.

**Beginner-Friendly Explanation**  
Partitioning is like splitting a large spreadsheet into smaller sheets by a category—one sheet per department, one sheet per country, or one sheet per year. A window function looks only at the rows on the current sheet when computing a value for a row, so “department average” is computed department by department rather than across the whole company.

### Key Characteristics

- **Row-preserving**: Partitioning does not reduce the number of rows in the result set; it only defines the scope for window function evaluation.
- **Resets computation**: Aggregates, rankings, and running totals restart at each partition boundary.
- **Optional clause**: If `PARTITION BY` is omitted, the entire result set is treated as a single partition.
- **Independent of final ordering**: The partition order is not the output order; a separate query-level `ORDER BY` controls presentation.
- **Compatible with grouping**: `PARTITION BY` may appear alongside `GROUP BY`, but it operates on the grouped result in that case.
- **Multiple expressions allowed**: Partitions can be defined by a single column or a combination of columns and expressions.
- **Engine support**: Available in PostgreSQL 8.4+, MySQL 8.0+, SQL Server 2012+, Oracle, and SQLite 3.25+.

### Prerequisites

- Understanding of window function fundamentals and the `OVER` clause.
- Familiarity with `SELECT`, `WHERE`, `GROUP BY`, and `HAVING`.
- Knowledge of aggregate functions and ranking functions.
- Awareness of the logical processing order of SQL clauses.

### Related Programming Areas

- Analytical query processing and OLAP.
- Ranking, running totals, and moving averages.
- Per-group reporting and comparative analysis.
- Query optimisation and index design.
- Data warehousing and business intelligence.

---

## Core Concepts and Features

### 1. `PARTITION BY`

#### Definitions

**Core Definition**  
`PARTITION BY` is the clause within `OVER` that divides the result set into groups for window function evaluation.

**Technical Definition**  
`PARTITION BY` accepts one or more expressions. Rows with equal values for all partitioning expressions are placed in the same partition. The window function is computed independently for each partition. The partitioning expressions are evaluated on the virtual table produced after `FROM`, `WHERE`, `GROUP BY`, and `HAVING`, and they cannot reference output column aliases in most engines. If `PARTITION BY` is omitted, the entire result set forms a single partition.

**Beginner-Friendly Explanation**  
`PARTITION BY` is how you tell the database “group the rows by this column before doing the window calculation,” without collapsing the rows into one per group.

#### Purposes

- To compute per-group metrics while retaining every individual row.
- To reset running totals and rankings for each group.
- To isolate window calculations by category, region, or time period.
- To avoid self-joins when group-level context is needed per row.

#### Syntax Rules and Structure

**General syntax**:

```sql
window_function(...) OVER (
    PARTITION BY expression [, expression ...]
    [ ORDER BY ... ]
    [ frame_clause ]
)
```

**PostgreSQL** (official):

```sql
function_name(...) OVER (PARTITION BY expression [, ...] [ ORDER BY ... ] [ frame_clause ])
```

**MySQL 8.0+** (official):

```sql
function_name(...) OVER (PARTITION BY expr [, ...] [ ORDER BY ... ] [ frame_clause ])
```

**SQL Server** (official):

```sql
function_name(...) OVER (
    [ PARTITION BY value_expression [, ...] ]
    [ ORDER BY ... ]
    [ ROWS | RANGE frame_extent ]
)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `PARTITION BY` | Keyword introducing the partitioning expressions. |
| `expression` | Column, expression, or `CASE` expression defining partition membership. |
| `,` | Separates multiple partitioning expressions. |

**Syntax rules**

- `PARTITION BY` is optional; omitting it creates one global partition.
- Partitioning expressions are evaluated before window functions and cannot reference `SELECT` aliases in most engines.
- Multiple expressions create finer partitions based on unique combinations.
- `PARTITION BY` may appear with or without `ORDER BY`; the combination determines default framing.

**Constraints and limitations**

- Large numbers of small partitions can increase overhead.
- Partitioning does not guarantee output order; use query-level `ORDER BY` for presentation.
- Some engines restrict expressions in `PARTITION BY` (e.g., MySQL standard SQL permits column references but not arbitrary functions in all contexts).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Partitioning by department for an average

```sql
-- Step 1: Define the query with PARTITION BY.
SELECT depname, empno, salary,
       avg(salary) OVER (PARTITION BY depname) AS dept_avg
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

**Why this output appears**: `PARTITION BY depname` groups employees by department; `avg(salary)` is computed within each department and attached to every employee row in that department.

##### Example 2: MySQL 8.0 – Partitioning by country for a group total

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
| 2000 | USA     | Calculator |     75 |           4575 |
| 2000 | USA     | Computer   |   1500 |           4575 |
| 2001 | USA     | Calculator |     50 |           4575 |
| 2001 | USA     | Computer   |   1500 |           4575 |
| 2001 | USA     | Computer   |   1200 |           4575 |
| 2001 | USA     | TV         |    150 |           4575 |
| 2001 | USA     | TV         |    100 |           4575 |
+------+---------+------------+--------+----------------+
```

**Why this output appears**: The `country` column partitions the data; the sum resets for each country and is repeated on every row of that country.

##### Example 3: SQL Server – Partitioning for ranking

```sql
SELECT CustomerID, OrderDate, TotalDue,
       ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY OrderDate) AS RowNum
FROM Sales.SalesOrderHeader;
```

**Expected output** (illustrative):

```text
CustomerID OrderDate   TotalDue RowNum
---------- ----------- -------- ------
1          2025-01-01  100.00   1
1          2025-02-01  150.00   2
1          2025-03-01  200.00   3
2          2025-01-15   50.00   1
2          2025-02-15   50.00   2
```

**Why this output appears**: `PARTITION BY CustomerID` creates a separate ranking sequence for each customer; `ROW_NUMBER` restarts at 1 for each customer’s first order.

#### Real-World Cases

- **Departmental averages**: Attaching each employee’s department average to their row.
- **Regional sales totals**: Showing each sale alongside its region’s total.
- **Per-customer running balances**: Resetting cumulative balances per account.

---

### 2. Multiple Partition Columns

#### Definitions

**Core Definition**  
Multiple partition columns define partitions by the unique combination of values across two or more expressions, creating finer-grained groups.

**Technical Definition**  
`PARTITION BY expr1, expr2, ..., exprN` forms partitions for every distinct tuple `(expr1, expr2, ..., exprN)`. Each combination of values creates an independent partition. This is semantically equivalent to partitioning by a composite key; the window function is evaluated separately for each unique combination.

**Beginner-Friendly Explanation**  
If partitioning by department gives one group per department, partitioning by department and year gives one group per department per year—like having a separate sheet for each department–year combination.

#### Purposes

- To create finer partitions when a single column is insufficient.
- To reset computations for each combination of categories.
- To handle hierarchical grouping (e.g., country and year).
- To support multi-dimensional analytical calculations.

#### Syntax Rules and Structure

**Syntax**:

```sql
window_function(...) OVER (PARTITION BY expr1, expr2 [, ...])
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `expr1, expr2, ...` | Comma-separated partitioning expressions. |
| Combination | Each unique tuple forms one partition. |

**Syntax rules**

- Expressions may be columns, `CASE` expressions, or scalar functions.
- The order of expressions in `PARTITION BY` does not affect the partition definition (unlike `ORDER BY`).
- Each expression is evaluated independently; `NULL` values are treated as equal for partitioning purposes in most engines.
- Multiple columns can come from the same table or from joined tables.

**Constraints and limitations**

- More expressions generally mean more, smaller partitions; extremely fine partitions can increase overhead.
- Not all expressions are portable across engines (e.g., function support varies).
- `PARTITION BY` expressions cannot reference `SELECT` aliases in most engines.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Partitioning by department and year

```sql
SELECT depname, year, empno, salary,
       AVG(salary) OVER (PARTITION BY depname, year) AS dept_year_avg
FROM empsalary_yearly;
```

**Expected output** (illustrative):

```text
 depname  | year | empno | salary |   dept_year_avg
----------+------+-------+--------+--------------------
 develop  | 2024 |     8 |   6000 | 5500.000000000000
 develop  | 2024 |    10 |   5000 | 5500.000000000000
 develop  | 2025 |     9 |   4500 | 4500.000000000000
 sales    | 2024 |     1 |   5000 | 5000.000000000000
 sales    | 2025 |     4 |   4800 | 4800.000000000000
(5 rows)
```

**Why this output appears**: Each unique `(depname, year)` combination forms its own partition; the average resets for each department–year pair.

##### Example 2: MySQL 8.0 – Partitioning by category and extracted year

```sql
SELECT product_id, category_id, sale_date, sale_amount,
       AVG(sale_amount) OVER (
           PARTITION BY category_id, EXTRACT(YEAR FROM sale_date)
       ) AS yearly_category_avg
FROM sales;
```

**Expected output** (illustrative):

```text
+------------+-------------+------------+-------------+---------------------+
| product_id | category_id | sale_date  | sale_amount | yearly_category_avg |
+------------+-------------+------------+-------------+---------------------+
|        101 |           5 | 2024-03-01 |      100.00 |          150.000000 |
|        102 |           5 | 2024-07-15 |      200.00 |          150.000000 |
|        103 |           5 | 2025-01-10 |      300.00 |          300.000000 |
|        201 |           9 | 2024-05-20 |       50.00 |           50.000000 |
+------------+-------------+------------+-------------+---------------------+
```

**Why this output appears**: The partition is defined by `(category_id, year(sale_date))`; the average resets for each category and year combination.

##### Example 3: SQL Server – Partitioning by region and product category

```sql
SELECT Region, Category, Product, Sales,
       SUM(Sales) OVER (PARTITION BY Region, Category) AS RegionCategoryTotal
FROM SalesData;
```

**Expected output** (illustrative):

```text
Region Category Product Sales RegionCategoryTotal
------ -------- ------- ----- -------------------
East   A        P1      100   300
East   A        P2      200   300
East   B        P3      150   150
West   A        P4      250   400
West   A        P5      150   400
```

**Why this output appears**: Each `(Region, Category)` pair forms a partition; the sum is computed per combination and repeated on each row of that combination.

#### Real-World Cases

- **Multi-dimensional reporting**: Sales by region and year, headcount by department and location.
- **Cohort analysis**: Grouping by signup month and plan type.
- **Inventory analysis**: Partitioning by warehouse and product category.

---

### 3. Partition vs `GROUP BY`

#### Definitions

**Core Definition**  
`GROUP BY` collapses rows into one row per group, whereas `PARTITION BY` divides rows into groups for window function evaluation without collapsing them.

**Technical Definition**  
`GROUP BY` is a query-level clause that changes the entire query’s output granularity: one output row per group, with non-grouped columns only accessible through aggregates. `PARTITION BY` is a window-level clause that defines partitions for window functions but leaves the query’s row count unchanged; every input row produces an output row. `GROUP BY` is used with non-window aggregates; `PARTITION BY` is used inside `OVER` with window functions (including aggregate functions used as window functions).

**Beginner-Friendly Explanation**  
`GROUP BY` is like rolling up a detailed report into a summary with one line per category—you lose the individual lines. `PARTITION BY` is like keeping every detailed line and adding a summary column next to it, computed per category.

#### Purposes

- To choose between summarised output (`GROUP BY`) and row-level enriched output (window functions).
- To understand when a query requires collapsing versus when it must retain detail.
- To combine both clauses appropriately when per-group aggregates must feed window calculations.
- To avoid the common mistake of trying to use `GROUP BY` when row-level detail is required.

#### Syntax Rules and Structure

**`GROUP BY` example**:

```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

**`PARTITION BY` example**:

```sql
SELECT department, employee, salary,
       AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM employees;
```

**Component breakdown**

| Aspect | `GROUP BY` | `PARTITION BY` |
|--------|------------|----------------|
| Clause level | Query level | Window level (inside `OVER`) |
| Row count | One row per group | Same as input rows |
| Output detail | Aggregated only | Detail plus window result |
| Typical use | Summaries | Per-row context |
| Can appear with window functions? | Yes (window functions run after grouping) | Yes (partitions operate on grouped result) |

**Syntax rules**

- `GROUP BY` changes the meaning of the entire `SELECT`; all non-aggregated columns must be grouped.
- `PARTITION BY` does not change the query’s row count.
- Window functions execute after `GROUP BY`, so `SUM(amount) OVER (PARTITION BY ...)` can reference grouped aggregates.
- `PARTITION BY` expressions cannot reference output aliases in most engines, similar to `GROUP BY`.

**Constraints and limitations**

- `GROUP BY` cannot retain row-level detail in the same `SELECT` without window functions.
- `PARTITION BY` cannot reduce rows; if deduplication is needed, use `DISTINCT` or `GROUP BY` in an outer query.
- Mixing `GROUP BY` and window functions requires careful reasoning about what the window operates on (the grouped result).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Side-by-side comparison

```sql
-- GROUP BY: one row per department.
SELECT depname, AVG(salary) AS avg_salary
FROM empsalary
GROUP BY depname;

-- PARTITION BY: one row per employee, with department average.
SELECT depname, empno, salary,
       AVG(salary) OVER (PARTITION BY depname) AS dept_avg
FROM empsalary;
```

**Expected output** (GROUP BY):

```text
 depname  |      avg_salary
----------+---------------------
 develop  | 5020.0000000000000000
 personnel| 3700.0000000000000000
 sales    | 4866.6666666666666667
(3 rows)
```

**Expected output** (PARTITION BY):

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

**Why this output appears**: `GROUP BY` reduces 10 rows to 3; `PARTITION BY` retains all 10 rows and adds the department average as a new column.

##### Example 2: MySQL – When to use each

```sql
-- Use GROUP BY for a high-level summary.
SELECT country, SUM(profit) AS total_profit
FROM sales
GROUP BY country;

-- Use PARTITION BY for detail plus context.
SELECT year, country, product, profit,
       SUM(profit) OVER (PARTITION BY country) AS country_profit
FROM sales;
```

**Expected output** (GROUP BY):

```text
+---------+--------------+
| country | total_profit |
+---------+--------------+
| Finland |         1610 |
| India   |         1350 |
| USA     |         4575 |
+---------+--------------+
```

**Expected output** (PARTITION BY):

```text
+------+---------+------------+--------+----------------+
| year | country | product    | profit | country_profit |
+------+---------+------------+--------+----------------+
| 2000 | Finland | Computer   |   1500 |           1610 |
| 2000 | Finland | Phone      |    100 |           1610 |
| 2001 | Finland | Phone      |     10 |           1610 |
| 2000 | India   | Calculator |     75 |           1350 |
| ...  | ...     | ...        | ...    | ...            |
+------+---------+------------+--------+----------------+
```

**Why this output appears**: `GROUP BY` collapses to one row per country; `PARTITION BY` keeps every sale and attaches the country total.

##### Example 3: SQL Server – Performance observation

```sql
-- GROUP BY often produces a smaller, cheaper plan.
SELECT CustomerID, SUM(TotalDue) AS TotalSpent
FROM Sales.SalesOrderHeader
GROUP BY CustomerID;

-- PARTITION BY retains all rows and may require a larger window buffer.
SELECT CustomerID, OrderDate, TotalDue,
       SUM(TotalDue) OVER (PARTITION BY CustomerID) AS CustomerTotal
FROM Sales.SalesOrderHeader;
```

**Expected output** (GROUP BY, illustrative):

```text
CustomerID TotalSpent
---------- ----------
1          450.00
2          100.00
```

**Expected output** (PARTITION BY, illustrative):

```text
CustomerID OrderDate   TotalDue CustomerTotal
---------- ----------- -------- -------------
1          2025-01-01  100.00   450.00
1          2025-02-01  150.00   450.00
1          2025-03-01  200.00   450.00
2          2025-01-15   50.00   100.00
2          2025-02-15   50.00   100.00
```

**Why this output appears**: `GROUP BY` returns two rows for two customers; `PARTITION BY` returns five rows but adds the customer total to each row. Performance benchmarks often show `GROUP BY` faster when only summary output is needed, because it can use hash or sort aggregation and does not maintain a window buffer for every row.

#### Real-World Cases

- **Executive dashboards**: `GROUP BY` for summary KPIs; `PARTITION BY` for detail tables with per-group context.
- **Data quality checks**: `GROUP BY` to find duplicates; `PARTITION BY` with `ROW_NUMBER` to flag them while keeping rows.
- **ETL validation**: `GROUP BY` for reconciliation totals; `PARTITION BY` for row-level audit trails.

---

### 4. Partition Ordering

#### Definitions

**Core Definition**  
Partition ordering is the specification of the order of rows within each partition, using `ORDER BY` inside the `OVER` clause, which determines the sequence for window function evaluation.

**Technical Definition**  
`ORDER BY` within `OVER` defines the logical row order within each partition. It is independent of the query’s final `ORDER BY`. Ordering is required for ranking functions (`RANK`, `DENSE_RANK`, `ROW_NUMBER`) and determines the default frame when no explicit frame is given (`RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`). Rows that are equal under the `ORDER BY` expressions are peers; ranking functions and `RANGE` frames treat peers specially.

**Beginner-Friendly Explanation**  
Ordering inside a partition tells the window function which row comes “first,” “next,” and “last” within each group—essential for rankings and running totals.

#### Purposes

- To define the sequence for ranking and cumulative calculations.
- To ensure deterministic results when ties exist.
- To control the default frame behavior.
- To support functions like `LAG`, `LEAD`, `FIRST_VALUE`, and `LAST_VALUE`.

#### Syntax Rules and Structure

**Syntax**:

```sql
window_function(...) OVER (
    PARTITION BY expr
    ORDER BY sort_expr [ ASC | DESC ] [, ...]
)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `ORDER BY` | Keyword introducing sort expressions within the partition. |
| `sort_expr` | Column or expression to sort by. |
| `ASC` / `DESC` | Sort direction (default `ASC`). |

**Syntax rules**

- `ORDER BY` inside `OVER` is independent of the outer query’s `ORDER BY`.
- Ranking functions require `ORDER BY` within `OVER`.
- Without `ORDER BY`, rows are processed in an unspecified order, and the entire partition is the default frame.
- Ties (peers) affect `RANK` (skips ranks) and `DENSE_RANK` (does not skip).
- `NULL` ordering is engine-specific but generally follows the engine’s sort rules.

**Constraints and limitations**

- Non-deterministic ordering from ties can produce unstable results unless a tiebreaker is added.
- `ORDER BY` in `OVER` does not guarantee the final output order.
- Some engines restrict expressions in window `ORDER BY`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Ranking with partition ordering

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

##### Example 2: MySQL – Running total with partition ordering

```sql
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

**Why this output appears**: `ORDER BY year` orders rows within each country; the running sum accumulates by year and resets at each new country.

##### Example 3: SQL Server – `LAG` with partition ordering

```sql
SELECT CustomerID, OrderDate, TotalDue,
       LAG(TotalDue) OVER (PARTITION BY CustomerID ORDER BY OrderDate) AS PrevTotal
FROM Sales.SalesOrderHeader;
```

**Expected output** (illustrative):

```text
CustomerID OrderDate   TotalDue PrevTotal
---------- ----------- -------- ---------
1          2025-01-01  100.00   NULL
1          2025-02-01  150.00   100.00
1          2025-03-01  200.00   150.00
2          2025-01-15   50.00   NULL
2          2025-02-15   50.00    50.00
```

**Why this output appears**: `ORDER BY OrderDate` determines the previous row within each customer; `LAG` retrieves the prior `TotalDue`.

#### Real-World Cases

- **Leaderboards**: Ranking within groups with ties.
- **Cumulative metrics**: Running totals ordered by date.
- **Trend analysis**: Previous/next values for comparison.

---

### 5. Partition Scope

#### Definitions

**Core Definition**  
Partition scope is the set of rows included in a partition, as defined by the `PARTITION BY` expressions, and the frame’s subset within that partition.

**Technical Definition**  
The partition scope is determined by the unique combination of `PARTITION BY` values: all rows sharing those values belong to the same partition. Within a partition, the frame clause further narrows the rows used for a given row’s calculation. If no `PARTITION BY` is specified, the scope is the entire result set. If no frame clause is specified, the scope depends on `ORDER BY`: without `ORDER BY`, the entire partition; with `ORDER BY`, the default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`.

**Beginner-Friendly Explanation**  
Partition scope is “which rows count as being in the same group.” If you partition by country, all rows for Finland are in one scope, and all rows for India are in another. The frame then decides whether the function looks at all rows in the scope or only a slice of them.

#### Purposes

- To understand which rows influence a given row’s window result.
- To reason about default framing when `ORDER BY` is present or absent.
- To control whether calculations use the entire partition or a moving subset.
- To avoid incorrect assumptions about peer inclusion.

#### Syntax Rules and Structure

**Scope definition**:

```sql
PARTITION BY expr   -- defines the scope boundary
ORDER BY ...        -- influences default frame
frame_clause        -- optional explicit narrowing
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `PARTITION BY` | Defines the partition scope. |
| `ORDER BY` | Activates the default `RANGE` frame if no frame is specified. |
| Frame clause | Explicitly sets the start and end of the scope within the partition. |

**Syntax rules**

- Without `PARTITION BY`, the entire result set is one scope.
- Without `ORDER BY` and without a frame, the entire partition is the scope for every row.
- With `ORDER BY` and no frame, the default scope is the partition start through the current row’s last peer.
- Frame clauses (`ROWS`, `RANGE`, `GROUPS`) explicitly narrow the scope.

**Constraints and limitations**

- The default `RANGE` frame includes peers, which can surprise users expecting strict row-by-row behavior.
- `ROWS` frames operate on physical row positions; `RANGE` frames operate on `ORDER BY` values.
- Frame clauses are ignored by functions that do not use frames (e.g., `ROW_NUMBER`).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Scope with and without `ORDER BY`

```sql
-- Without ORDER BY: entire partition is the scope.
SELECT salary,
       sum(salary) OVER (PARTITION BY depname) AS partition_sum
FROM empsalary;

-- With ORDER BY: default RANGE frame, scope from start to current peer.
SELECT salary,
       sum(salary) OVER (PARTITION BY depname ORDER BY salary) AS running_sum
FROM empsalary;
```

**Expected output** (without `ORDER BY`):

```text
 salary | partition_sum
--------+--------------
   5200 |        25100
   4200 |        25100
   4500 |        25100
   6000 |        25100
   5200 |        25100
   3500 |         7400
   3900 |         7400
   4800 |        14600
   5000 |        14600
   4800 |        14600
(10 rows)
```

**Expected output** (with `ORDER BY`, default `RANGE`):

```text
 salary | running_sum
--------+-------------
   4200 |        4200
   4500 |        8700
   5200 |       19100
   5200 |       19100
   6000 |       25100
   3500 |        3500
   3900 |        7400
   4800 |        9600
   4800 |       14400
   5000 |       14600
(10 rows)
```

**Why this output appears**: Without `ORDER BY`, every row in a department sees the full department total. With `ORDER BY`, the default `RANGE` frame includes all rows from the start of the partition up to the current row’s last peer, so the two 5200 rows both see the same cumulative sum.

##### Example 2: MySQL – Explicit `ROWS` frame narrows scope

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

**Why this output appears**: The `ROWS BETWEEN 1 PRECEDING AND CURRENT ROW` frame restricts the scope to the current row and the one physically preceding it, producing a moving average over at most two rows.

##### Example 3: SQL Server – `RANGE` frame with logical scope

```sql
SELECT Salary,
       SUM(Salary) OVER (ORDER BY Salary RANGE BETWEEN 500 PRECEDING AND CURRENT ROW) AS RangeSum
FROM Employees;
```

**Expected output** (illustrative):

```text
Salary RangeSum
------ --------
3500   3500
3900   7400
4200   11600
4500   16100
4800   20900
4800   20900
5000   30700
```

**Why this output appears**: The `RANGE` frame includes rows whose `ORDER BY` value is within 500 of the current row’s value, creating a value-based scope rather than a physical row count.

#### Real-World Cases

- **Moving averages**: Restricting scope with `ROWS` or `RANGE` for trend smoothing.
- **Peer-aware totals**: Using default `RANGE` to include all rows with the same rank.
- **Cumulative metrics**: Using `ROWS` for strict row-by-row running totals.

---

## References

- PostgreSQL Documentation – Window Functions (Tutorial) – https://www.postgresql.org/docs/current/tutorial-window.html
- PostgreSQL Documentation – Window Functions (Reference) – https://www.postgresql.org/docs/current/functions-window.html
- PostgreSQL Documentation – `SELECT` (Window Function Syntax) – https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-WINDOW-FUNCTIONS
- MySQL 8.4 Reference Manual – Window Function Concepts and Syntax – https://dev.mysql.com/doc/refman/8.4/en/window-functions-usage.html
- MySQL 8.4 Reference Manual – Window Function Frame Specification – https://dev.mysql.com/doc/refman/8.4/en/window-functions-frames.html
- Microsoft Learn – `OVER` Clause (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql
- Microsoft Learn – Use the OVER Clause (Training) – https://learn.microsoft.com/en-us/training/modules/write-queries-that-use-window-functions/3-use-over-clause
- SQL:2003 Standard – Window Functions (ISO/IEC 9075-2:2003) – https://jtc1sc32.org/doc/N1401-1450/32N1408T%20text%20CD%209075-2%20SQL_Foundation.pdf
- DbVisualizer – SQL Window Functions: Running Totals, Rankings, and Deduplication – https://www.dbvis.com/thetable/sql-window-functions-running-totals-rankings-and-deduplication/
- DbVisualizer – SQL PARTITION BY: A Guide to Window Functions and Data Segmentation – https://www.dbvis.com/thetable/sql-partition-by-in-postgresql-a-guide-to-window-functions-and-data-segmentation/
- Baeldung – Understanding the SQL OVER() Clause – https://www.baeldung.com/sql/over
- Stack Overflow – What is the difference between PARTITION BY and GROUP BY – https://stackoverflow.com/questions/2404565/what-is-the-difference-between-partition-by-and-group-by
- Erik Darling Data – Indexing SQL Server Queries For Performance: Windowing Functions – https://erikdarling.com/indexing-sql-server-queries-for-performance-windowing-functions/
- LearnSQL – How to Use the PARTITION BY Clause in SQL – https://learnsql.com/blog/partition-by-clause-in-sql/
- DataCamp – MySQL PARTITION BY Clauses: Usage & Examples – https://www.datacamp.com/tutorial/mysql-partition-by