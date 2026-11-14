# Comprehensive Programming Cheat Sheet: Window Ordering

## Topic Overview

### Definitions

**Core Definition**  
Window ordering is the specification of the sequence in which rows within each partition are processed by a window function, defined by the `ORDER BY` clause inside the `OVER` clause.

**Technical Definition**  
Window ordering is the `ORDER BY` sub-clause of a window specification. It defines a logical sort order for rows within each partition. This order determines the sequence for ranking functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`), the direction for value functions (`LAG`, `LEAD`, `FIRST_VALUE`, `LAST_VALUE`, `NTH_VALUE`), and the default frame for aggregate window functions. The ordering is independent of the query-level `ORDER BY` that controls final output presentation. Rows that are not distinct under the window `ORDER BY` expressions are called *peers*. The SQL standard defines window ordering as part of the window specification, supported by PostgreSQL, MySQL 8.0+, SQL Server, Oracle, and SQLite.

**Beginner-Friendly Explanation**  
Window ordering is how you tell the database which row comes "first," "second," and "last" inside each group before it does the window calculation. It is like sorting a deck of cards within each suit before dealing them out—the suit groups the cards (partitioning), and the sort order within each suit tells the dealer which card is on top.

### Key Characteristics

- **Scoped to partitions**: Ordering is applied independently within each partition; it does not reorder across partitions.
- **Independent of final output**: The window `ORDER BY` is separate from the query-level `ORDER BY` that determines the final result order.
- **Required for ranking**: Ranking functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`) require an `ORDER BY` in the window specification.
- **Determines default framing**: When `ORDER BY` is present and no explicit frame is given, the default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`.
- **Peer-sensitive**: Rows with equal values under the window `ORDER BY` are peers; ranking functions and `RANGE` frames treat peers specially.
- **Supports multiple keys and directions**: Multiple sort expressions, `ASC`/`DESC`, and engine-specific `NULLS FIRST`/`NULLS LAST` options are available.

### Prerequisites

- Understanding of window function fundamentals and the `OVER` clause.
- Knowledge of `PARTITION BY` and window frames.
- Familiarity with `SELECT`, `WHERE`, `GROUP BY`, and `HAVING`.
- Awareness of the target engine’s NULL ordering defaults and syntax.

### Related Programming Areas

- Ranking and leaderboard analytics.
- Running totals and moving averages.
- Time-series analysis and trend detection.
- Query optimisation and index design.
- Data warehousing and OLAP.

---

## Core Concepts and Features

### 1. `ORDER BY` in Window

#### Definitions

**Core Definition**  
`ORDER BY` in a window is the clause inside `OVER` that establishes the logical row sequence within each partition for window function evaluation.

**Technical Definition**  
The `ORDER BY` sub-clause of a window specification accepts one or more sort expressions. Each expression may specify `ASC` (default) or `DESC`, and engines may support `NULLS FIRST` or `NULLS LAST`. The ordering is applied after partitioning and before frame determination. It defines the meaning of "preceding," "following," and "current row" for frame clauses and value functions. Without an `ORDER BY`, the window contains all rows of the partition and the concept of a current row's position within an order is undefined.

**Beginner-Friendly Explanation**  
`ORDER BY` inside the window tells the database, "Within each group, line up the rows in this order before you compute the window function." Without it, the database does not know which row comes before or after which.

#### Purposes

- To define the sequence for ranking functions such as `ROW_NUMBER` and `RANK`.
- To establish the direction for `LAG`, `LEAD`, `FIRST_VALUE`, and `LAST_VALUE`.
- To activate the default `RANGE` frame for cumulative aggregate calculations.
- To ensure deterministic results by providing an explicit sort order.

#### Syntax Rules and Structure

**General syntax**:

```sql
window_function(...) OVER (
    [ PARTITION BY expr [, ...] ]
    ORDER BY sort_expression [ ASC | DESC ] [, ...]
    [ frame_clause ]
)
```

**PostgreSQL** (official) :

```sql
[ ORDER BY expression [ ASC | DESC | USING operator ] [ NULLS { FIRST | LAST } ] [, ...] ]
```

**MySQL 8.0+** (official) :

```sql
[ ORDER BY expr [ ASC | DESC ] [, ...] ]
```

**SQL Server** (official) :

```sql
[ ORDER BY order_by_expression [ ASC | DESC ] [ , ... ] ]
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `ORDER BY` | Keyword introducing sort expressions within the window. |
| `sort_expression` | Column, expression, or alias to sort by. |
| `ASC` / `DESC` | Sort direction (default `ASC`). |
| `NULLS FIRST` / `NULLS LAST` | Optional NULL placement (engine-dependent). |
| `frame_clause` | Optional explicit frame; default depends on `ORDER BY`. |

**Syntax rules**

- `ORDER BY` is optional for aggregate window functions but required for ranking functions.
- The window `ORDER BY` is independent of the query-level `ORDER BY`; they serve different purposes.
- Multiple sort expressions create a lexicographic order.
- Without `ORDER BY` and without a frame, the window is the entire partition.
- With `ORDER BY` and no explicit frame, the default is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, which includes all rows from the partition start through the current row's last peer.

**Constraints and limitations**

- SQL Server does not support `NULLS FIRST`/`NULLS LAST` in window `ORDER BY`; NULLs are treated as the lowest possible values.
- Ranking functions ignore frame clauses; they operate on the entire partition ordered by `ORDER BY`.
- Some engines disallow constants or certain expressions in window `ORDER BY`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Ordering for ranking

```sql
-- Step 1: Rank employees by salary within each department.
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

**Why this output appears**: `ORDER BY salary DESC` sorts employees within each department from highest to lowest salary; `RANK` assigns 1 to the highest, with ties sharing the same rank and the next rank skipped.

##### Example 2: MySQL 8.0 – Ordering for a running total

```sql
SELECT time, subject, val,
       SUM(val) OVER (PARTITION BY subject ORDER BY time ROWS UNBOUNDED PRECEDING) AS running_total
FROM observations;
```

**Expected output** (illustrative):

```text
+----------+---------+------+---------------+
| time     | subject | val  | running_total |
+----------+---------+------+---------------+
| 07:00:00 | st113   |   10 |            10 |
| 07:15:00 | st113   |    9 |            19 |
| 07:30:00 | st113   |   25 |            44 |
| 07:45:00 | st113   |   20 |            64 |
| 07:00:00 | xh458   |    0 |             0 |
| 07:15:00 | xh458   |   10 |            10 |
| 07:30:00 | xh458   |    5 |            15 |
| 07:45:00 | xh458   |   30 |            45 |
| 08:00:00 | xh458   |   25 |            70 |
+----------+---------+------+---------------+
```

**Why this output appears**: `ORDER BY time` sorts observations within each subject chronologically; the `ROWS UNBOUNDED PRECEDING` frame accumulates values from the start of the partition up to the current row.

##### Example 3: SQL Server – Ordering for `LAG`

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

**Why this output appears**: `ORDER BY OrderDate` determines which row is "previous" within each customer; `LAG` retrieves the prior `TotalDue`, returning `NULL` for the first row of each partition.

#### Real-World Cases

- **Leaderboards**: Ranking players by score within each region.
- **Financial data**: Calculating running balances ordered by transaction date.
- **Time-series**: Computing moving averages over chronologically ordered observations.

---

### 2. Ascending and Descending

#### Definitions

**Core Definition**  
Ascending (`ASC`) and descending (`DESC`) specify the direction of the window sort order.

**Technical Definition**  
`ASC` sorts from smallest to largest (the default); `DESC` sorts from largest to smallest. The direction affects the meaning of "preceding" and "following" in frame clauses and value functions: in ascending order, preceding rows have smaller values; in descending order, preceding rows have larger values. Direction is specified per sort expression and may be mixed across multiple keys.

**Beginner-Friendly Explanation**  
Ascending means "smallest first"; descending means "largest first." The direction changes which row is considered "before" the current row, which matters for running totals and previous/next comparisons.

#### Purposes

- To control whether rankings go from highest to lowest or lowest to highest.
- To define the direction of cumulative calculations.
- To determine which row is "previous" or "next" for `LAG` and `LEAD`.
- To match business semantics (e.g., top scores first vs. earliest dates first).

#### Syntax Rules and Structure

**Syntax**:

```sql
ORDER BY expression [ ASC | DESC ]
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `expression` | Column or expression to sort by. |
| `ASC` | Ascending order (default if omitted). |
| `DESC` | Descending order. |

**Syntax rules**

- `ASC` is the default and may be omitted.
- Direction applies only to the immediately preceding expression; it does not cascade to other keys.
- `ORDER BY x, y DESC` means `x ASC, y DESC`, not `x DESC, y DESC`.
- Mixing directions across keys is allowed and common (e.g., `ORDER BY category ASC, sales DESC`).

**Constraints and limitations**

- Direction affects frame semantics; a `ROWS` frame counts physical rows in the specified direction.
- `DESC` with `NULLS FIRST` is the default NULL placement in PostgreSQL; `ASC` defaults to `NULLS LAST`.
- SQL Server treats NULLs as lowest regardless of direction.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Ascending vs descending ranking

```sql
-- Ascending: lowest salary gets rank 1.
SELECT empno, salary,
       rank() OVER (ORDER BY salary ASC) AS rank_asc
FROM empsalary;

-- Descending: highest salary gets rank 1.
SELECT empno, salary,
       rank() OVER (ORDER BY salary DESC) AS rank_desc
FROM empsalary;
```

**Expected output** (ascending):

```text
 empno | salary | rank_asc
-------+--------+----------
     5 |   3500 |        1
     2 |   3900 |        2
     7 |   4200 |        3
     9 |   4500 |        4
     3 |   4800 |        5
     4 |   4800 |        5
     1 |   5000 |        7
    10 |   5200 |        8
    11 |   5200 |        8
     8 |   6000 |       10
(10 rows)
```

**Expected output** (descending):

```text
 empno | salary | rank_desc
-------+--------+-----------
     8 |   6000 |         1
    10 |   5200 |         2
    11 |   5200 |         2
     1 |   5000 |         4
     3 |   4800 |         5
     4 |   4800 |         5
     9 |   4500 |         7
     7 |   4200 |         8
     2 |   3900 |         9
     5 |   3500 |        10
(10 rows)
```

**Why this output appears**: `ASC` ranks the lowest salary first; `DESC` ranks the highest first. Ties share ranks and create gaps in `RANK`.

##### Example 2: MySQL – Descending running total

```sql
SELECT product, profit,
       SUM(profit) OVER (ORDER BY profit DESC ROWS UNBOUNDED PRECEDING) AS cumulative_profit
FROM sales;
```

**Expected output** (illustrative):

```text
+---------+--------+-------------------+
| product | profit | cumulative_profit |
+---------+--------+-------------------+
| TV      |   1500 |              1500 |
| Computer|   1200 |              2700 |
| Phone   |   1000 |              3700 |
| Tablet  |    500 |              4200 |
| Cable   |    100 |              4300 |
+---------+--------+-------------------+
```

**Why this output appears**: `ORDER BY profit DESC` sorts highest profit first; the cumulative sum accumulates from the highest profit downward.

#### Real-World Cases

- **Top-N reports**: `DESC` to rank highest values first.
- **Chronological analysis**: `ASC` to process events in time order.
- **Mixed-direction dashboards**: `ORDER BY region ASC, sales DESC` to show top sellers per region.

---

### 3. Multiple Sort Keys

#### Definitions

**Core Definition**  
Multiple sort keys define a lexicographic window ordering using two or more expressions, creating a deterministic sequence when a single key has ties.

**Technical Definition**  
`ORDER BY expr1 [ASC|DESC], expr2 [ASC|DESC], ...` sorts rows by `expr1` first; rows with equal `expr1` values are sorted by `expr2`, and so on. This creates a composite ordering that reduces or eliminates peer groups. Multiple keys are essential for deterministic ranking when the primary key has duplicates and for complex analytical orderings such as category-then-date.

**Beginner-Friendly Explanation**  
Multiple sort keys are like sorting a list of people by last name, then first name, then middle name—the second key only matters when the first key is the same.

#### Purposes

- To break ties deterministically when a single key has duplicates.
- To create complex orderings such as category followed by date.
- To reduce peer groups for ranking functions where each row should have a unique rank.
- To match business ordering requirements (e.g., region, then sales, then product).

#### Syntax Rules and Structure

**Syntax**:

```sql
ORDER BY expr1 [ASC|DESC], expr2 [ASC|DESC] [, ...]
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `expr1` | Primary sort key. |
| `expr2` | Secondary sort key, applied when `expr1` values are equal. |
| Direction | Each key may have its own `ASC` or `DESC`. |

**Syntax rules**

- Keys are evaluated left to right.
- Direction applies only to its own expression.
- Any number of keys may be specified, subject to engine limits.
- Multiple keys can reference columns from the same or joined tables.

**Constraints and limitations**

- More keys increase sort cost.
- SQL Server does not support `NULLS FIRST`/`NULLS LAST` per key; NULLs sort as lowest overall.
- The combination of keys determines the peer group: two rows are peers only if they are equal on all keys.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Deterministic ranking with a tiebreaker

```sql
SELECT depname, empno, salary,
       row_number() OVER (PARTITION BY depname ORDER BY salary DESC, empno ASC) AS rn
FROM empsalary;
```

**Expected output**:

```text
 depname  | empno | salary | rn
----------+-------+--------+----
 develop  |     8 |   6000 |  1
 develop  |    10 |   5200 |  2
 develop  |    11 |   5200 |  3
 develop  |     9 |   4500 |  4
 develop  |     7 |   4200 |  5
 personnel|     2 |   3900 |  1
 personnel|     5 |   3500 |  2
 sales    |     1 |   5000 |  1
 sales    |     3 |   4800 |  2
 sales    |     4 |   4800 |  3
(10 rows)
```

**Why this output appears**: `salary DESC` is the primary key; `empno ASC` breaks ties between the two 5200 salaries in `develop` and the two 4800 salaries in `sales`, producing unique row numbers.

##### Example 2: MySQL – Mixed directions with multiple keys

```sql
SELECT year, country, product, profit,
       RANK() OVER (ORDER BY country ASC, profit DESC) AS rank
FROM sales;
```

**Expected output** (illustrative):

```text
+------+---------+------------+--------+------+
| year | country | product    | profit | rank |
+------+---------+------------+--------+------+
| 2000 | Finland | Computer   |   1500 |    1 |
| 2000 | Finland | Phone      |    100 |    2 |
| 2001 | Finland | Phone      |     10 |    3 |
| 2000 | India   | Computer   |   1200 |    4 |
| 2000 | India   | Calculator |     75 |    5 |
| 2000 | India   | Calculator |     75 |    5 |
+------+---------+------------+--------+------+
```

**Why this output appears**: `country ASC` groups countries alphabetically; within each country, `profit DESC` ranks highest profit first; ties in India share rank 5.

##### Example 3: SQL Server – Three sort keys

```sql
SELECT Region, Category, Product, Sales,
       ROW_NUMBER() OVER (ORDER BY Region ASC, Category ASC, Sales DESC) AS rn
FROM SalesData;
```

**Expected output** (illustrative):

```text
Region Category Product Sales rn
------ -------- ------- ----- --
East   A        P2      200   1
East   A        P1      100   2
East   B        P3      150   3
West   A        P4      250   4
West   A        P5      150   5
```

**Why this output appears**: Rows are ordered by `Region` alphabetically, then `Category`, then `Sales` descending within each region–category group.

#### Real-World Cases

- **Leaderboards with tiebreakers**: `ORDER BY score DESC, time ASC` to rank fastest among tied scores.
- **Report ordering**: `ORDER BY region ASC, sales DESC, product ASC` for multi-level reports.
- **Deterministic testing**: Adding a unique tiebreaker ensures reproducible row numbers.

---

### 4. NULL Ordering

#### Definitions

**Core Definition**  
NULL ordering determines whether `NULL` values appear before or after non-`NULL` values in the window sort order.

**Technical Definition**  
SQL engines differ in how they treat `NULL` in `ORDER BY`. The SQL standard and PostgreSQL support explicit `NULLS FIRST` and `NULLS LAST` options. PostgreSQL’s default is `NULLS LAST` for `ASC` and `NULLS FIRST` for `DESC` (treating NULLs as larger than non-NULLs). MySQL historically treats NULLs as lowest; SQL Server treats NULLs as the lowest possible values in all cases. The choice of NULL ordering affects ranking positions and frame boundaries, especially when `NULL` values are present in the `ORDER BY` key.

**Beginner-Friendly Explanation**  
NULL ordering decides whether missing values go at the top or bottom of the sorted list. Different databases make different choices by default, so explicit control is important when NULLs are present.

#### Purposes

- To control where missing values appear in rankings and cumulative calculations.
- To ensure consistent results across engines with different defaults.
- To exclude or include NULLs in the window scope as required by business rules.
- To avoid misleading rankings when NULLs represent unknown or invalid data.

#### Syntax Rules and Structure

**PostgreSQL** (official) :

```sql
ORDER BY expression [ ASC | DESC ] [ NULLS { FIRST | LAST } ]
```

**MySQL / SQL Server** (no per-key `NULLS FIRST`/`NULLS LAST` syntax):

```sql
ORDER BY expression [ ASC | DESC ]
-- Engine-specific defaults apply.
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `NULLS FIRST` | NULLs sort before all non-NULL values. |
| `NULLS LAST` | NULLs sort after all non-NULL values. |
| Default | Engine-specific: PostgreSQL `NULLS LAST` for `ASC`, `NULLS FIRST` for `DESC`; SQL Server NULLs lowest; MySQL NULLs lowest. |

**Syntax rules**

- `NULLS FIRST`/`NULLS LAST` are supported in PostgreSQL, Oracle, and SQLite.
- SQL Server does not support these options in window `ORDER BY`; NULLs are treated as the lowest values.
- MySQL 8.0 does not support `NULLS FIRST`/`NULLS LAST`; use expressions like `ISNULL(col)` or `COALESCE` to emulate.
- NULL ordering affects peer grouping: all NULLs are peers with each other.

**Constraints and limitations**

- Portability: queries using `NULLS FIRST`/`NULLS LAST` may fail on SQL Server or MySQL.
- Default behavior may differ from business expectations; explicit handling is recommended.
- In SQL Server, a constant or `NULL` expression in window `ORDER BY` may cause errors in some contexts.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Explicit `NULLS LAST`

```sql
SELECT name, salary,
       ROW_NUMBER() OVER (ORDER BY salary DESC NULLS LAST) AS rn
FROM employees;
```

**Expected output** (illustrative):

```text
 name  | salary | rn
-------+--------+----
 Alice |   6000 |  1
 Bob   |   5200 |  2
 Carol |   4800 |  3
 Dave  |   NULL |  4
 Eve   |   NULL |  5
```

**Why this output appears**: `NULLS LAST` places `NULL` salaries after all non-`NULL` values, so employees with unknown salaries receive the highest row numbers.

##### Example 2: MySQL – Emulating `NULLS LAST` with a helper expression

```sql
SELECT name, salary,
       ROW_NUMBER() OVER (ORDER BY salary IS NULL, salary DESC) AS rn
FROM employees;
```

**Expected output**:

```text
+-------+--------+----+
| name  | salary | rn |
+-------+--------+----+
| Alice |   6000 |  1 |
| Bob   |   5200 |  2 |
| Carol |   4800 |  3 |
| Dave  |   NULL |  4 |
| Eve   |   NULL |  5 |
+-------+--------+----+
```

**Why this output appears**: `salary IS NULL` evaluates to `0` for non-NULL and `1` for NULL, so non-NULL rows sort first; `salary DESC` then orders the non-NULL rows by salary.

##### Example 3: SQL Server – NULLs sort as lowest

```sql
SELECT name, salary,
       ROW_NUMBER() OVER (ORDER BY salary ASC) AS rn
FROM Employees;
```

**Expected output**:

```text
name  salary rn
----- ------ --
Dave  NULL   1
Eve   NULL   2
Carol 4800   3
Bob   5200   4
Alice 6000   5
```

**Why this output appears**: SQL Server treats `NULL` as the lowest possible value; `ASC` places `NULL` rows first, regardless of whether the business considers them valid.

#### Real-World Cases

- **Data quality dashboards**: Explicitly placing NULLs last to highlight missing values.
- **Financial reports**: Ranking accounts with known balances above those with unknown balances.
- **Cross-engine portability**: Writing engine-specific NULL handling to achieve consistent results.

---

### 5. Ordering vs Partitioning

#### Definitions

**Core Definition**  
Partitioning divides rows into independent groups for window calculation, while ordering defines the sequence of rows within each group; they are complementary components of a window specification.

**Technical Definition**  
`PARTITION BY` creates disjoint subsets of the result set; the window function resets for each partition. `ORDER BY` defines a logical sort within each partition; it determines the sequence for ranking and cumulative calculations and activates the default frame. Partitioning is optional and supported by all window functions; ordering is optional for aggregates but required for ranking functions. Together, they define the scope and sequence of the window computation.

**Beginner-Friendly Explanation**  
Partitioning is like dividing a class into teams; ordering is like lining up each team by height. The partition decides who is in your group; the order decides who stands before and after you within that group.

#### Purposes

- To distinguish between grouping for scope (`PARTITION BY`) and sequencing for calculation (`ORDER BY`).
- To choose the right combination for ranking, running totals, or whole-partition aggregates.
- To avoid confusion between window partitioning and `GROUP BY`.
- To design indexes that support both partitioning and ordering.

#### Syntax Rules and Structure

**Combined syntax**:

```sql
window_function(...) OVER (
    PARTITION BY expr [, ...]
    ORDER BY expr [ ASC | DESC ] [, ...]
)
```

**Component breakdown**

| Component | Role | Resets? | Required? |
|-----------|------|---------|-----------|
| `PARTITION BY` | Divides rows into groups | Yes, per partition | Optional |
| `ORDER BY` | Sequences rows within each partition | No, but determines frame | Required for ranking; optional for aggregates |

**Syntax rules**

- `PARTITION BY` is applied first, then `ORDER BY` within each partition.
- Without `PARTITION BY`, the entire result set is one partition; `ORDER BY` sequences all rows.
- Without `ORDER BY`, the entire partition is the window; ranking functions are invalid.
- The combination determines the default frame: with `ORDER BY`, default is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`; without it, the entire partition.

**Constraints and limitations**

- Partitioning is not the same as `GROUP BY`; it does not collapse rows.
- Ordering does not affect partition boundaries; a new partition resets the order.
- Index design should generally lead with `PARTITION BY` columns followed by `ORDER BY` columns for optimal performance.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Partitioning and ordering together

```sql
SELECT depname, empno, salary,
       SUM(salary) OVER (PARTITION BY depname ORDER BY salary DESC) AS running_total
FROM empsalary;
```

**Expected output**:

```text
 depname  | empno | salary | running_total
----------+-------+--------+---------------
 develop  |     8 |   6000 |          6000
 develop  |    10 |   5200 |         11200
 develop  |    11 |   5200 |         11200
 develop  |     9 |   4500 |         15700
 develop  |     7 |   4200 |         19900
 personnel|     2 |   3900 |          3900
 personnel|     5 |   3500 |          7400
 sales    |     1 |   5000 |          5000
 sales    |     4 |   4800 |          9800
 sales    |     3 |   4800 |          9800
(10 rows)
```

**Why this output appears**: `PARTITION BY depname` resets the sum for each department; `ORDER BY salary DESC` sequences rows within each department; the default `RANGE` frame includes peers, so the two 5200 rows both show 11200.

##### Example 2: MySQL – Partitioning without ordering vs ordering without partitioning

```sql
-- Partitioning only: entire partition per row.
SELECT country, profit,
       SUM(profit) OVER (PARTITION BY country) AS country_total
FROM sales;

-- Ordering only: global running total.
SELECT year, profit,
       SUM(profit) OVER (ORDER BY year ROWS UNBOUNDED PRECEDING) AS global_running
FROM sales;
```

**Expected output** (partitioning only):

```text
+---------+--------+---------------+
| country | profit | country_total |
+---------+--------+---------------+
| Finland |   1500 |          1610 |
| Finland |    100 |          1610 |
| Finland |     10 |          1610 |
| India   |     75 |          1350 |
| India   |     75 |          1350 |
| India   |   1200 |          1350 |
+---------+--------+---------------+
```

**Expected output** (ordering only):

```text
+------+--------+---------------+
| year | profit | global_running|
+------+--------+---------------+
| 2000 |   1500 |          1500 |
| 2000 |    100 |          1600 |
| 2000 |     75 |          1675 |
| 2000 |     75 |          1750 |
| 2000 |   1200 |          2950 |
| 2001 |     10 |          2960 |
+------+--------+---------------+
```

**Why this output appears**: Without `ORDER BY`, every row in a country sees the full country total. Without `PARTITION BY`, the running total spans the entire result set ordered by year.

##### Example 3: SQL Server – Partitioning and ordering for ranking

```sql
SELECT CustomerID, OrderDate, TotalDue,
       ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY OrderDate) AS OrderSeq
FROM Sales.SalesOrderHeader;
```

**Expected output** (illustrative):

```text
CustomerID OrderDate   TotalDue OrderSeq
---------- ----------- -------- --------
1          2025-01-01  100.00   1
1          2025-02-01  150.00   2
1          2025-03-01  200.00   3
2          2025-01-15   50.00   1
2          2025-02-15   50.00   2
```

**Why this output appears**: `PARTITION BY CustomerID` restarts the sequence for each customer; `ORDER BY OrderDate` sequences orders chronologically within each customer.

#### Real-World Cases

- **Per-customer sequencing**: Partitioning by customer and ordering by order date to number orders.
- **Global rankings**: Ordering without partitioning to rank all rows across the entire dataset.
- **Partition-wide totals**: Partitioning without ordering to attach group totals to every row.

---

## References

- PostgreSQL Documentation – Window Functions (Tutorial) – https://www.postgresql.org/docs/current/tutorial-window.html
- PostgreSQL Documentation – Window Functions (Reference) – https://www.postgresql.org/docs/current/functions-window.html
- PostgreSQL Documentation – `SELECT` (`ORDER BY` and NULL ordering) – https://www.postgresql.org/docs/current/sql-select.html
- MySQL 8.4 Reference Manual – Window Function Frame Specification – https://dev.mysql.com/doc/refman/8.4/en/window-functions-frames.html
- MySQL 8.4 Reference Manual – Window Function Concepts and Syntax – https://dev.mysql.com/doc/refman/8.4/en/window-functions-usage.html
- Microsoft Learn – `OVER` Clause (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql
- Microsoft Learn – Ranking Functions (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/functions/ranking-functions-transact-sql
- Oracle Database Documentation – Analytic Functions – https://docs.oracle.com/en/database/oracle/oracle-database/23/sqlrf/Analytic-Functions.html
- SQLite Documentation – Window Functions – https://www.sqlite.org/windowfunctions.html
- SQL:2003 Standard – Window Functions (ISO/IEC 9075-2:2003)
- SQL Authority – What is the OVER Clause? – https://blog.sqlauthority.com/2015/11/04/sql-server-what-is-the-over-clause-notes-from-the-field-101/
- LearnSQL – Window Functions in SQL – https://learnsql.com/blog/sql-window-functions/
- Use The Index, Luke – Window Functions and Indexing – https://use-the-index-luke.com/sql/partial-results/window-functions