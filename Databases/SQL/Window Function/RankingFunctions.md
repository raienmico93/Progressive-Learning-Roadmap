# Comprehensive Programming Cheat Sheet: Ranking Functions

## Topic Overview

### Definitions

**Core Definition**  
Ranking functions are a category of window functions that assign a rank, row number, percentile, or bucket to each row based on its position within an ordered partition of the result set.

**Technical Definition**  
Ranking functions are non-aggregate window functions defined by the SQL standard (SQL:2003 and later) that compute a numeric value for each row based on the row's position within its window partition, as determined by the `ORDER BY` clause of the window specification. The six standard ranking functions are `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `PERCENT_RANK()`, `CUME_DIST()`, and `NTILE()`. They are supported by PostgreSQL, MySQL 8.0+, SQL Server, Oracle, and other modern RDBMSs. Ranking functions ignore window frame clauses; they always operate on the entire partition ordered by the window `ORDER BY`.

**Beginner-Friendly Explanation**  
Ranking functions are like giving each row a "position" or "score" based on how it compares to other rows in its group. For example, you can rank employees by salary within each department, number rows sequentially, divide rows into equal-sized buckets, or calculate what percentage of rows are below a given value. Unlike `GROUP BY`, ranking functions keep every row in the output and add a new column with the ranking information.

### Key Characteristics

- **Row-preserving**: Ranking functions do not collapse rows; every input row produces one output row with an additional ranking column.
- **Require `ORDER BY`**: All ranking functions require an `ORDER BY` clause in the window specification to define the ranking order; `ROW_NUMBER` and `NTILE` specifically require it in most engines, while `RANK`, `DENSE_RANK`, `PERCENT_RANK`, and `CUME_DIST` are meaningless without it.
- **Tie handling varies**: `ROW_NUMBER` ignores ties (assigns unique numbers), `RANK` and `DENSE_RANK` handle ties differently (gaps vs. no gaps), and `PERCENT_RANK`/`CUME_DIST` account for ties in their formulas.
- **Frame-independent**: Ranking functions do not use the window frame clause; they always evaluate over the entire partition.
- **Partition-relative**: Rankings reset for each partition defined by `PARTITION BY`.
- **Return types**: `ROW_NUMBER`, `RANK`, and `DENSE_RANK` return integers (typically `bigint`); `PERCENT_RANK` and `CUME_DIST` return floating-point values between 0 and 1; `NTILE` returns an integer bucket number.

### Prerequisites

- Understanding of window function fundamentals, including the `OVER` clause, `PARTITION BY`, and `ORDER BY`.
- Familiarity with `SELECT`, `WHERE`, `GROUP BY`, and `HAVING`.
- Knowledge of subqueries and CTEs for filtering on ranking results.
- Awareness of the target engine's support for ranking functions (MySQL 8.0+, PostgreSQL 8.4+, SQL Server 2005+, Oracle).

### Related Programming Areas

- Top-N and bottom-N reporting.
- Deduplication and data quality.
- Leaderboards and competitive rankings.
- Percentile and distribution analysis.
- Pagination and row numbering.
- Data warehousing and OLAP.

---

## Core Concepts and Features

### 1. `ROW_NUMBER()`

#### Definitions

**Core Definition**  
`ROW_NUMBER()` assigns a unique, sequential integer to each row within its partition, starting at 1, based on the window `ORDER BY`.

**Technical Definition**  
`ROW_NUMBER()` is a ranking function that returns the sequential position of the current row within its partition, counting from 1. It does not handle ties: even if two rows have identical `ORDER BY` values, they receive different row numbers. The assignment of numbers to tied rows is non-deterministic unless the `ORDER BY` includes a unique tiebreaker. `ROW_NUMBER()` is supported by PostgreSQL, MySQL 8.0+, SQL Server, Oracle, and SQLite 3.25+.

**Beginner-Friendly Explanation**  
`ROW_NUMBER()` simply numbers the rows in each group from 1 upward, like numbering items on a list. Even if two rows look identical, they get different numbers.

#### Purposes

- To assign unique sequential identifiers to rows within a partition.
- To implement top-N, bottom-N, and inner-N reporting by filtering on row numbers.
- To deduplicate rows by keeping only the first row (`rn = 1`) per group.
- To support pagination by selecting a range of row numbers.

#### Syntax Rules and Structure

**General syntax**:

```sql
ROW_NUMBER() OVER (
    [ PARTITION BY expression [, ...] ]
    ORDER BY expression [ ASC | DESC ] [, ...]
)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `ROW_NUMBER()` | Function name; takes no arguments. |
| `OVER` | Mandatory clause introducing the window. |
| `PARTITION BY` | Optional; resets numbering per partition. |
| `ORDER BY` | Required; defines the sequence for numbering. |

**Syntax rules**

- `ROW_NUMBER()` takes no arguments.
- `ORDER BY` is required in most engines; without it, numbering is arbitrary.
- Ties are broken arbitrarily unless the `ORDER BY` includes a unique column.
- The function ignores the window frame clause; it always numbers the entire partition.

**Constraints and limitations**

- Non-deterministic ordering for ties can make results unstable across executions.
- `ROW_NUMBER()` cannot be used directly in `WHERE` or `HAVING`; use a subquery or CTE to filter on the row number.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Numbering employees within departments

```sql
SELECT depname, empno, salary,
       ROW_NUMBER() OVER (PARTITION BY depname ORDER BY salary DESC) AS row_num
FROM empsalary;
```

**Expected output**:

```text
 depname  | empno | salary | row_num
----------+-------+--------+---------
 develop  |     8 |   6000 |       1
 develop  |    10 |   5200 |       2
 develop  |    11 |   5200 |       3
 develop  |     9 |   4500 |       4
 develop  |     7 |   4200 |       5
 personnel|     2 |   3900 |       1
 personnel|     5 |   3500 |       2
 sales    |     1 |   5000 |       1
 sales    |     4 |   4800 |       2
 sales    |     3 |   4800 |       3
(10 rows)
```

**Why this output appears**: `PARTITION BY depname` resets the numbering for each department; `ORDER BY salary DESC` numbers employees from highest to lowest salary. The two 5200 salaries in `develop` receive distinct row numbers (2 and 3) because `ROW_NUMBER()` ignores ties.

##### Example 2: MySQL 8.0 – Using `ROW_NUMBER()` for top-N per group

```sql
WITH ranked AS (
    SELECT product_id, category, product_name, revenue,
           ROW_NUMBER() OVER (PARTITION BY category ORDER BY revenue DESC) AS rn
    FROM products
)
SELECT category, product_name, revenue
FROM ranked
WHERE rn <= 3
ORDER BY category, revenue DESC;
```

**Expected output** (illustrative):

```text
+----------+--------------+---------+
| category | product_name | revenue |
+----------+--------------+---------+
| Electronics | Laptop    | 5000.00 |
| Electronics | Tablet    | 3000.00 |
| Electronics | Phone     | 2000.00 |
| Clothing    | Jacket    | 800.00  |
| Clothing    | Shirt     | 500.00  |
| Clothing    | Pants     | 400.00  |
+----------+--------------+---------+
```

**Why this output appears**: The CTE computes row numbers per category ordered by revenue descending; the outer query filters `rn <= 3`, returning the top three products per category.

##### Example 3: SQL Server – Deduplication with `ROW_NUMBER()`

```sql
;WITH CTE AS (
    SELECT Id, SecondIdentifier, Year, ItemBoxId,
           ROW_NUMBER() OVER (PARTITION BY ItemBoxId ORDER BY Id, SecondIdentifier) AS RN
    FROM YourTable
)
DELETE FROM CTE WHERE RN > 1;
```

**Expected output** (no result set; rows deleted):

```text
(1 row affected)
```

**Why this output appears**: The CTE numbers rows within each `ItemBoxId` group; `DELETE FROM CTE WHERE RN > 1` removes all duplicate rows, keeping the first row per group.

#### Real-World Cases

- **Top-N reporting**: Finding the highest-paid employees per department.
- **Deduplication**: Removing duplicate records while preserving one canonical row.
- **Pagination**: Using row numbers to implement `OFFSET`/`FETCH` equivalents.
- **Data quality**: Assigning unique identifiers to rows in a result set.

---

### 2. `RANK()`

#### Definitions

**Core Definition**  
`RANK()` assigns a rank to each row within its partition, with identical values receiving the same rank and subsequent ranks skipped (creating gaps).

**Technical Definition**  
`RANK()` returns the rank of the current row within its partition, with gaps. Rows with equal values for the `ORDER BY` expressions are peers and receive the same rank; the next rank is the count of rows preceding the current peer group plus one. For example, if two rows tie for rank 1, the next row receives rank 3. `RANK()` is supported by PostgreSQL, MySQL 8.0+, SQL Server, Oracle, and SQLite.

**Beginner-Friendly Explanation**  
`RANK()` is like Olympic ranking: if two athletes tie for first place, they both get a gold medal (rank 1), and the next athlete gets bronze (rank 3)—the silver rank is skipped.

#### Purposes

- To rank rows with gaps after ties, reflecting competitive standings.
- To implement leaderboards where ties share a rank and subsequent ranks are skipped.
- To compare performance against a standard where gaps matter (e.g., competition results).
- To combine with `PARTITION BY` for per-group rankings.

#### Syntax Rules and Structure

**General syntax**:

```sql
RANK() OVER (
    [ PARTITION BY expression [, ...] ]
    ORDER BY expression [ ASC | DESC ] [, ...]
)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `RANK()` | Function name; takes no arguments. |
| `OVER` | Mandatory clause. |
| `PARTITION BY` | Optional; resets ranking per partition. |
| `ORDER BY` | Required; defines the ranking order. |

**Syntax rules**

- `RANK()` takes no arguments.
- `ORDER BY` is required.
- Ties receive the same rank; the next rank skips the number of tied rows minus one.
- `RANK()` ignores the window frame; it always ranks the entire partition.

**Constraints and limitations**

- Gaps in rank numbers may be undesirable when dense ranking is needed; use `DENSE_RANK()` instead.
- Non-deterministic ordering of ties can produce inconsistent rank assignments across executions if no unique tiebreaker is present.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Ranking with gaps

```sql
SELECT depname, empno, salary,
       RANK() OVER (PARTITION BY depname ORDER BY salary DESC) AS rank
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

**Why this output appears**: The two 5200 salaries in `develop` tie at rank 2, and the next salary (4500) receives rank 4, skipping rank 3. `RANK()` produces gaps after ties.

##### Example 2: MySQL – `RANK()` for competition standings

```sql
SELECT player_name, score,
       RANK() OVER (ORDER BY score DESC) AS player_rank
FROM player_scores;
```

**Expected output** (illustrative):

```text
+-------------+-------+-------------+
| player_name | score | player_rank |
+-------------+-------+-------------+
| Alice       |  100  |           1 |
| Bob         |   95  |           2 |
| Carol       |   95  |           2 |
| Dave        |   90  |           4 |
+-------------+-------+-------------+
```

**Why this output appears**: Bob and Carol tie at rank 2; Dave receives rank 4, skipping rank 3. This reflects standard competition ranking.

#### Real-World Cases

- **Competitive rankings**: Sports standings, exam results, and sales contests where ties share rank and gaps are acceptable.
- **Performance comparisons**: Identifying the top performers with clear gaps between tied and non-tied positions.
- **Audit reporting**: Ranking records where gaps highlight tied groups.

---

### 3. `DENSE_RANK()`

#### Definitions

**Core Definition**  
`DENSE_RANK()` assigns a rank to each row within its partition, with identical values receiving the same rank and no gaps in the ranking sequence.

**Technical Definition**  
`DENSE_RANK()` returns the rank of the current row within its partition, without gaps. Rows with equal values are peers and receive the same rank; the next distinct value receives the next consecutive rank. For example, if two rows tie for rank 1, the next distinct row receives rank 2 (not 3). `DENSE_RANK()` is supported by PostgreSQL, MySQL 8.0+, SQL Server, Oracle, and SQLite.

**Beginner-Friendly Explanation**  
`DENSE_RANK()` is like ranking distinct values: if two students tie for first place, they both get rank 1, and the next student gets rank 2—no ranks are skipped.

#### Purposes

- To rank distinct values without gaps after ties.
- To assign sequential tier numbers when ties should share a tier but subsequent tiers remain consecutive.
- To avoid gaps in ranking-based filtering (e.g., selecting the top 3 distinct salary levels).
- To provide a compact ranking for reporting.

#### Syntax Rules and Structure

**General syntax**:

```sql
DENSE_RANK() OVER (
    [ PARTITION BY expression [, ...] ]
    ORDER BY expression [ ASC | DESC ] [, ...]
)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `DENSE_RANK()` | Function name; takes no arguments. |
| `OVER` | Mandatory clause. |
| `PARTITION BY` | Optional; resets ranking per partition. |
| `ORDER BY` | Required; defines the ranking order. |

**Syntax rules**

- `DENSE_RANK()` takes no arguments.
- `ORDER BY` is required.
- Ties receive the same rank; the next distinct value receives the next consecutive rank.
- `DENSE_RANK()` ignores the window frame.

**Constraints and limitations**

- No gaps may be undesirable when competitive gaps are expected; use `RANK()` instead.
- Non-deterministic tie ordering can affect which rows share a rank if no unique tiebreaker is present.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Dense ranking without gaps

```sql
SELECT depname, empno, salary,
       DENSE_RANK() OVER (PARTITION BY depname ORDER BY salary DESC) AS dense_rank
FROM empsalary;
```

**Expected output**:

```text
 depname  | empno | salary | dense_rank
----------+-------+--------+------------
 develop  |     8 |   6000 |          1
 develop  |    10 |   5200 |          2
 develop  |    11 |   5200 |          2
 develop  |     9 |   4500 |          3
 develop  |     7 |   4200 |          4
 personnel|     2 |   3900 |          1
 personnel|     5 |   3500 |          2
 sales    |     1 |   5000 |          1
 sales    |     4 |   4800 |          2
 sales    |     3 |   4800 |          2
(10 rows)
```

**Why this output appears**: The two 5200 salaries share rank 2; the next salary (4500) receives rank 3, not 4. `DENSE_RANK()` does not skip ranks after ties.

##### Example 2: SQL Server – Comparing `RANK` and `DENSE_RANK`

```sql
SELECT Salary,
       RANK() OVER (ORDER BY Salary DESC) AS RankWithGaps,
       DENSE_RANK() OVER (ORDER BY Salary DESC) AS DenseRank
FROM Employees;
```

**Expected output** (illustrative):

```text
Salary RankWithGaps DenseRank
------ ------------ ---------
6000   1            1
5200   2            2
5200   2            2
4500   4            3
4200   5            4
```

**Why this output appears**: `RANK()` skips rank 3 after the tie, while `DENSE_RANK()` assigns rank 3 to the next distinct salary.

#### Real-World Cases

- **Tier assignment**: Assigning customers to loyalty tiers without gaps.
- **Top-N distinct values**: Selecting the top 3 distinct salary levels.
- **Compact reporting**: Rankings where consecutive numbers are preferred over gaps.

---

### 4. `PERCENT_RANK()`

#### Definitions

**Core Definition**  
`PERCENT_RANK()` returns the relative rank of the current row as a value between 0 and 1, calculated as `(rank - 1) / (total rows - 1)`.

**Technical Definition**  
`PERCENT_RANK()` computes the percentage rank of a row within its partition using the formula `(rank - 1) / (total_rows - 1)`, where `rank` is the `RANK()` of the current row (with gaps) and `total_rows` is the number of rows in the partition. The result ranges from 0 (for the first row) to 1 (for the last row). If the partition has only one row, the result is 0. `PERCENT_RANK()` is supported by PostgreSQL, MySQL 8.0+, SQL Server, Oracle, and other modern engines.

**Beginner-Friendly Explanation**  
`PERCENT_RANK()` tells you what fraction of rows are strictly below the current row, on a scale from 0 to 1. A value of 0 means you are at the very bottom; 1 means you are at the very top.

#### Purposes

- To express a row's relative position as a percentile-like value.
- To compare performance across partitions of different sizes.
- To identify top or bottom percentiles for filtering.
- To support statistical analysis and distribution reporting.

#### Syntax Rules and Structure

**General syntax**:

```sql
PERCENT_RANK() OVER (
    [ PARTITION BY expression [, ...] ]
    ORDER BY expression [ ASC | DESC ] [, ...]
)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `PERCENT_RANK()` | Function name; takes no arguments. |
| `OVER` | Mandatory clause. |
| `PARTITION BY` | Optional; resets calculation per partition. |
| `ORDER BY` | Required; defines the ranking order. |

**Syntax rules**

- `PERCENT_RANK()` takes no arguments.
- `ORDER BY` is required.
- Returns a `double precision` (or equivalent) value between 0 and 1.
- The formula is `(rank - 1) / (total_rows - 1)` where `rank` is the `RANK()` value.
- For a partition with one row, the result is 0.

**Constraints and limitations**

- Sensitive to ties: tied rows share the same `PERCENT_RANK` because they share the same `RANK` value.
- The denominator is `total_rows - 1`, so a partition with one row returns 0.
- Interpretation as a true percentile may differ from other percentile definitions.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Percent rank within a partition

```sql
SELECT depname, empno, salary,
       PERCENT_RANK() OVER (PARTITION BY depname ORDER BY salary DESC) AS pct_rank
FROM empsalary;
```

**Expected output** (illustrative):

```text
 depname  | empno | salary |       pct_rank
----------+-------+--------+---------------------
 develop  |     8 |   6000 |                   0
 develop  |    10 |   5200 | 0.25
 develop  |    11 |   5200 | 0.25
 develop  |     9 |   4500 | 0.75
 develop  |     7 |   4200 | 1
 personnel|     2 |   3900 |                   0
 personnel|     5 |   3500 |                   1
 sales    |     1 |   5000 |                   0
 sales    |     4 |   4800 | 0.5
 sales    |     3 |   4800 | 0.5
(10 rows)
```

**Why this output appears**: In `develop`, the highest salary has `PERCENT_RANK = 0`; the two 5200 salaries share rank 2, so `(2-1)/(5-1) = 0.25`; the lowest salary has rank 5, so `(5-1)/(5-1) = 1`.

##### Example 2: MySQL – `PERCENT_RANK` vs `CUME_DIST`

```sql
SELECT val,
       ROW_NUMBER() OVER w AS 'row_number',
       CUME_DIST() OVER w AS 'cume_dist',
       PERCENT_RANK() OVER w AS 'percent_rank'
FROM numbers
WINDOW w AS (ORDER BY val);
```

**Expected output** (excerpt):

```text
+------+------------+--------------------+--------------+
| val  | row_number | cume_dist          | percent_rank |
+------+------------+--------------------+--------------+
| 1    | 1          | 0.2222222222222222 | 0            |
| 1    | 2          | 0.2222222222222222 | 0            |
| 2    | 3          | 0.3333333333333333 | 0.25         |
| 3    | 4          | 0.6666666666666666 | 0.375        |
| 3    | 5          | 0.6666666666666666 | 0.375        |
| 3    | 6          | 0.6666666666666666 | 0.375        |
| 4    | 7          | 0.8888888888888888 | 0.75         |
| 4    | 8          | 0.8888888888888888 | 0.75         |
| 5    | 9          | 1                  | 1            |
+------+------------+--------------------+--------------+
```

**Why this output appears**: `PERCENT_RANK` uses `(rank - 1) / (total_rows - 1)`, so for `val = 3` (rank 4, total 9), `(4-1)/(9-1) = 3/8 = 0.375`. `CUME_DIST` includes ties differently, yielding 6/9 = 0.666... for the same rows.

#### Real-World Cases

- **Percentile filtering**: Selecting rows above the 90th percentile.
- **Performance comparison**: Comparing sales performance across regions of different sizes.
- **Statistical analysis**: Reporting relative standings in a normalized 0–1 scale.

---

### 5. `CUME_DIST()`

#### Definitions

**Core Definition**  
`CUME_DIST()` returns the cumulative distribution of a value within a partition—the proportion of rows with values less than or equal to the current row's value.

**Technical Definition**  
`CUME_DIST()` computes the number of rows preceding or peer with the current row in the window ordering, divided by the total number of rows in the partition. The result is a value between 0 and 1 (exclusive of 0 in most cases) representing the cumulative distribution. Unlike `PERCENT_RANK()`, `CUME_DIST()` includes the current row and its peers in the numerator. `CUME_DIST()` is supported by PostgreSQL, MySQL 8.0+, SQL Server, Oracle, and other modern engines.

**Beginner-Friendly Explanation**  
`CUME_DIST()` tells you what fraction of rows have values less than or equal to the current row—including the current row itself. It is like asking “what percentage of the group scored at or below this value?”

#### Purposes

- To compute the cumulative distribution of values within a partition.
- To identify the proportion of rows at or below a given value.
- To support percentile-like analysis that includes ties.
- To compare with `PERCENT_RANK()` for statistical reporting.

#### Syntax Rules and Structure

**General syntax**:

```sql
CUME_DIST() OVER (
    [ PARTITION BY expression [, ...] ]
    ORDER BY expression [ ASC | DESC ] [, ...]
)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `CUME_DIST()` | Function name; takes no arguments. |
| `OVER` | Mandatory clause. |
| `PARTITION BY` | Optional; resets calculation per partition. |
| `ORDER BY` | Required; defines the ordering. |

**Syntax rules**

- `CUME_DIST()` takes no arguments.
- `ORDER BY` is required.
- Returns a value between 0 and 1 (typically greater than 0 and up to 1).
- Numerator: number of rows preceding or peer with the current row; denominator: total rows in the partition.
- Without `ORDER BY`, all rows are peers and the result is 1 for every row.

**Constraints and limitations**

- Includes the current row and its peers, unlike `PERCENT_RANK()`.
- Tied rows share the same `CUME_DIST()` value.
- The interpretation as a cumulative distribution may differ from other statistical definitions.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Cumulative distribution of salaries

```sql
SELECT depname, empno, salary,
       CUME_DIST() OVER (PARTITION BY depname ORDER BY salary) AS cume_dist
FROM empsalary;
```

**Expected output** (illustrative):

```text
 depname  | empno | salary |      cume_dist
----------+-------+--------+---------------------
 develop  |     7 |   4200 | 0.2
 develop  |     9 |   4500 | 0.4
 develop  |    10 |   5200 | 0.8
 develop  |    11 |   5200 | 0.8
 develop  |     8 |   6000 | 1
 personnel|     5 |   3500 | 0.5
 personnel|     2 |   3900 | 1
 sales    |     3 |   4800 | 0.6666666666666666
 sales    |     4 |   4800 | 0.6666666666666666
 sales    |     1 |   5000 | 1
(10 rows)
```

**Why this output appears**: In `develop`, the 4200 salary is the lowest (1 of 5 rows = 0.2); the two 5200 salaries have 4 rows at or below them (4/5 = 0.8); the 6000 salary includes all 5 rows (1.0).

##### Example 2: BigQuery – `CUME_DIST` for finish times

```sql
SELECT name, finish_time,
       CUME_DIST() OVER (PARTITION BY division ORDER BY finish_time ASC) AS finish_rank
FROM finishers;
```

**Expected output** (illustrative):

```text
+-------------+-------------+-------------+
| name        | finish_time | finish_rank |
+-------------+-------------+-------------+
| Sophia Liu  | 02:10:00    | 0.5         |
| John Doe    | 02:10:00    | 0.5         |
| Jane Smith  | 02:15:00    | 1.0         |
+-------------+-------------+-------------+
```

**Why this output appears**: Tied finish times share the same `CUME_DIST` value; the last finisher includes all rows in the cumulative count.

#### Real-World Cases

- **Distribution analysis**: Understanding what proportion of customers spend at or below a given amount.
- **Percentile reporting**: Computing cumulative percentiles that include ties.
- **Ranking with ties**: Comparing `CUME_DIST` with `PERCENT_RANK` to understand tie effects.

---

### 6. `NTILE()`

#### Definitions

**Core Definition**  
`NTILE(n)` divides the ordered rows of a partition into `n` approximately equal-sized buckets and assigns each row its bucket number.

**Technical Definition**  
`NTILE(n)` partitions the rows of each window partition into `n` buckets as evenly as possible, assigning bucket numbers from 1 to `n`. If the number of rows is not divisible by `n`, the larger buckets come first. `NTILE()` requires an `ORDER BY` in the window specification; without it, bucket assignment is arbitrary. `NTILE()` is supported by PostgreSQL, MySQL 8.0+, SQL Server, Oracle, and other modern engines.

**Beginner-Friendly Explanation**  
`NTILE(4)` splits the rows into four roughly equal groups (quartiles) and tells you which group each row belongs to. For example, the top 25% of scores get bucket 1, the next 25% get bucket 2, and so on.

#### Purposes

- To divide rows into equal-sized buckets for quartile, decile, or percentile analysis.
- To assign rows to tiers or groups of approximately equal size.
- To support stratified sampling and distribution analysis.
- To create balanced groups for A/B testing or resource allocation.

#### Syntax Rules and Structure

**General syntax**:

```sql
NTILE(num_buckets) OVER (
    [ PARTITION BY expression [, ...] ]
    ORDER BY expression [ ASC | DESC ] [, ...]
)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `NTILE(num_buckets)` | Function name with integer argument. |
| `num_buckets` | Number of buckets to divide the partition into. |
| `OVER` | Mandatory clause. |
| `PARTITION BY` | Optional; resets bucketing per partition. |
| `ORDER BY` | Required; defines the order for bucketing. |

**Syntax rules**

- `num_buckets` must be a positive integer (constant or expression).
- Buckets are numbered from 1 to `num_buckets`.
- If rows are not evenly divisible, larger buckets come first.
- `NTILE()` requires `ORDER BY` to define a meaningful bucket assignment.
- `NTILE()` ignores the window frame.

**Constraints and limitations**

- The bucket sizes may differ by at most one row.
- Without `ORDER BY`, bucket assignment is non-deterministic.
- `num_buckets` must be a positive integer; zero or negative values cause errors.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Dividing salaries into quartiles

```sql
SELECT depname, empno, salary,
       NTILE(4) OVER (ORDER BY salary) AS quartile
FROM empsalary;
```

**Expected output** (illustrative):

```text
 depname  | empno | salary | quartile
----------+-------+--------+----------
 personnel|     5 |   3500 |        1
 personnel|     2 |   3900 |        1
 develop  |     7 |   4200 |        1
 develop  |     9 |   4500 |        2
 sales    |     3 |   4800 |        2
 sales    |     4 |   4800 |        3
 sales    |     1 |   5000 |        3
 develop  |    10 |   5200 |        4
 develop  |    11 |   5200 |        4
 develop  |     8 |   6000 |        4
(10 rows)
```

**Why this output appears**: The 10 rows are divided into 4 buckets; the first two buckets receive 3 rows each, and the last two receive 2 rows each, following the rule that larger buckets come first.

##### Example 2: SQL Server – `NTILE(4)` for price quartiles

```sql
SELECT ProductID, ListPrice,
       NTILE(4) OVER (ORDER BY ListPrice) AS PriceQuartile
FROM Production.Product;
```

**Expected output** (illustrative):

```text
ProductID ListPrice PriceQuartile
--------- --------- -------------
1         0.00      1
2         5.00      1
3         10.00     2
4         20.00     2
5         50.00     3
6         100.00    3
7         200.00    4
8         500.00    4
```

**Why this output appears**: Products are ordered by list price and divided into four quartiles; lower-priced products receive lower quartile numbers.

#### Real-World Cases

- **Quartile analysis**: Dividing customers into spend quartiles.
- **Stratified sampling**: Selecting equal-sized groups for testing.
- **Performance tiers**: Assigning employees to performance bands.

---

### 7. Ranking Use Cases

#### Definitions

**Core Definition**  
Ranking use cases are the practical scenarios in which ranking functions are applied to solve analytical, reporting, and data-quality problems.

**Technical Definition**  
Ranking functions support top-N reporting, deduplication, pagination, percentile analysis, bucket assignment, and competitive ranking. They are frequently combined with CTEs or subqueries to filter on computed rank values. The choice of ranking function depends on the required tie handling: `ROW_NUMBER` for unique numbering, `RANK` for gaps after ties, `DENSE_RANK` for no gaps, `PERCENT_RANK` and `CUME_DIST` for relative standing, and `NTILE` for bucketing.

**Beginner-Friendly Explanation**  
Ranking functions are useful whenever you need to know “who is first,” “which rows are in the top 10%,” or “which duplicate should I keep.”

#### Purposes

- To identify top or bottom performers within groups.
- To remove duplicate records while preserving one canonical row.
- To paginate results without relying on `OFFSET`.
- To analyse distributions and percentiles.
- To assign rows to equal-sized buckets or tiers.

#### Syntax Rules and Structure

**Common patterns**:

```sql
-- Top-N per group
WITH ranked AS (
    SELECT ..., ROW_NUMBER() OVER (PARTITION BY grp ORDER BY metric DESC) AS rn
    FROM t
)
SELECT * FROM ranked WHERE rn <= N;

-- Deduplication
WITH numbered AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY key ORDER BY tiebreaker) AS rn
    FROM t
)
DELETE FROM numbered WHERE rn > 1;
```

**Component breakdown**

| Use case | Recommended function |
|----------|----------------------|
| Unique sequential numbers | `ROW_NUMBER()` |
| Competitive ranking with gaps | `RANK()` |
| Tier ranking without gaps | `DENSE_RANK()` |
| Relative standing (0–1) | `PERCENT_RANK()` |
| Cumulative distribution | `CUME_DIST()` |
| Bucket assignment | `NTILE(n)` |
| Deduplication | `ROW_NUMBER()` with `rn = 1` |

**Syntax rules**

- Ranking functions cannot be used directly in `WHERE` or `HAVING`; use a CTE or subquery.
- `ORDER BY` should be deterministic; add a unique tiebreaker to avoid arbitrary results.
- `PARTITION BY` resets rankings for each group.

**Constraints and limitations**

- Non-deterministic tie handling can cause inconsistent results across executions.
- Large result sets may require significant sort memory.
- Not all ranking functions are supported in older MySQL versions (pre-8.0).

---

### 8. Top-N per Group

#### Definitions

**Core Definition**  
Top-N per group is the retrieval of the N highest-ranked rows within each partition, based on a ranking function.

**Technical Definition**  
Top-N per group is implemented by computing a ranking function (typically `ROW_NUMBER()`, `RANK()`, or `DENSE_RANK()`) over a partition in a subquery or CTE, then filtering the outer query to `rank <= N`. `ROW_NUMBER()` returns exactly N rows per group (ties broken arbitrarily); `RANK()` may return more than N rows when ties exist at the boundary; `DENSE_RANK()` returns N distinct rank levels.

**Beginner-Friendly Explanation**  
Top-N per group means “give me the top 3 products in each category” or “the 5 highest-paid employees in each department.”

#### Purposes

- To retrieve the highest-performing items in each category.
- To implement per-group leaderboards and dashboards.
- To limit results per partition without losing group context.
- To support efficient “top N” reporting in applications.

#### Syntax Rules and Structure

**Pattern**:

```sql
WITH ranked AS (
    SELECT columns,
           ROW_NUMBER() OVER (PARTITION BY group_column ORDER BY metric DESC) AS rn
    FROM table
)
SELECT columns
FROM ranked
WHERE rn <= N;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `PARTITION BY group_column` | Defines the groups. |
| `ORDER BY metric DESC` | Defines the ranking criterion. |
| `ROW_NUMBER()`, `RANK()`, or `DENSE_RANK()` | Ranking function. |
| `WHERE rn <= N` | Filters the top N per group. |

**Syntax rules**

- The ranking function must be computed in a subquery or CTE; it cannot be used directly in `WHERE`.
- `ROW_NUMBER()` guarantees exactly N rows per group.
- `RANK()` may return more than N rows if ties occur at the boundary.
- `DENSE_RANK()` returns N distinct rank levels, which may correspond to more than N rows.

**Constraints and limitations**

- Performance depends on sorting each partition; indexes on `(group_column, metric)` can help.
- Without a tiebreaker, `ROW_NUMBER()` may assign the N-th position arbitrarily among tied rows.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Oracle – Top 3 employees per department

```sql
SELECT department_id, first_name, last_name, salary
FROM (
    SELECT department_id, first_name, last_name, salary,
           ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rn
    FROM employees
)
WHERE rn <= 3
ORDER BY department_id, salary DESC;
```

**Expected output** (illustrative):

```text
DEPARTMENT_ID FIRST_NAME LAST_NAME  SALARY
------------- ---------- ---------  ------
10            Jennifer   Whalen     4400
20            Michael    Hartstein  13000
20            Pat        Fay        6000
20            Den        Raphaely   11000
30            Den        Raphaely   11000
...
```

**Why this output appears**: The inner query ranks employees by salary within each department; the outer query filters `rn <= 3`, returning the top three per department.

##### Example 2: MySQL – Top 2 products per category

```sql
WITH ranked AS (
    SELECT product_id, category, product_name, revenue,
           ROW_NUMBER() OVER (PARTITION BY category ORDER BY revenue DESC) AS rn
    FROM products
)
SELECT category, product_name, revenue
FROM ranked
WHERE rn <= 2
ORDER BY category, revenue DESC;
```

**Expected output**:

```text
+----------+--------------+---------+
| category | product_name | revenue |
+----------+--------------+---------+
| Electronics | Laptop    | 5000.00 |
| Electronics | Tablet    | 3000.00 |
| Clothing    | Jacket    | 800.00  |
| Clothing    | Shirt     | 500.00  |
+----------+--------------+---------+
```

**Why this output appears**: The CTE computes row numbers per category; the outer query selects the top two per category.

#### Real-World Cases

- **Sales reporting**: Top 5 products per region.
- **HR analytics**: Top 3 highest-paid employees per department.
- **E-commerce**: Best-selling items per category.

---

### 9. Deduplication

#### Definitions

**Core Definition**  
Deduplication is the process of identifying and removing duplicate rows from a table, typically keeping one canonical row per duplicate group.

**Technical Definition**  
Deduplication with ranking functions uses `ROW_NUMBER()` (or occasionally `RANK()` or `DENSE_RANK()`) partitioned by the columns that define duplication, ordered by a tiebreaker (e.g., the most recent date or lowest ID). Rows with row number 1 are kept; rows with row number greater than 1 are deleted. This technique works in PostgreSQL, SQL Server, MySQL 8.0+, and Oracle.

**Beginner-Friendly Explanation**  
Deduplication means “find rows that are duplicates and keep only one of each.” Ranking functions help by numbering the duplicates so you can delete all but the first.

#### Purposes

- To remove duplicate records while preserving a canonical copy.
- To clean data before loading into a warehouse.
- To enforce uniqueness when constraints are missing.
- To keep the most recent or highest-priority record per duplicate group.

#### Syntax Rules and Structure

**Pattern**:

```sql
WITH numbered AS (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY duplicate_columns ORDER BY tiebreaker) AS rn
    FROM table
)
DELETE FROM numbered WHERE rn > 1;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `PARTITION BY duplicate_columns` | Columns that define duplicate groups. |
| `ORDER BY tiebreaker` | Determines which row to keep (e.g., `id ASC`, `date DESC`). |
| `ROW_NUMBER()` | Assigns 1 to the canonical row. |
| `DELETE ... WHERE rn > 1` | Removes duplicates. |

**Syntax rules**

- The CTE computes row numbers; the `DELETE` targets rows with `rn > 1`.
- The `ORDER BY` determines which row is kept; choose it carefully (e.g., newest, oldest, smallest ID).
- In SQL Server, `DELETE FROM CTE` works directly; in PostgreSQL, use `DELETE FROM table WHERE ctid IN (SELECT ctid FROM ...)`.
- MySQL 8.0 supports `DELETE ... JOIN` with a derived table containing row numbers.

**Constraints and limitations**

- Deleting large numbers of rows can be slow and lock-intensive; batch deletes may be necessary.
- Always test on a subset or backup before deleting.
- The tiebreaker must be deterministic to ensure consistent results.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Delete duplicates keeping the first by ID

```sql
;WITH CTE AS (
    SELECT Id, SecondIdentifier, Year, ItemBoxId,
           ROW_NUMBER() OVER (PARTITION BY ItemBoxId ORDER BY Id, SecondIdentifier) AS RN
    FROM YourTable
)
DELETE FROM CTE WHERE RN > 1;
```

**Expected output**:

```text
(1 row affected)
```

**Why this output appears**: The CTE numbers rows within each `ItemBoxId` group, ordered by `Id` and `SecondIdentifier`; `DELETE FROM CTE WHERE RN > 1` removes all but the first row per group.

##### Example 2: MySQL 8.0 – Deduplication with `ROW_NUMBER()`

```sql
WITH ranked AS (
    SELECT id,
           ROW_NUMBER() OVER (PARTITION BY email ORDER BY created_at DESC) AS rn
    FROM users
)
DELETE u
FROM users u
JOIN ranked r ON u.id = r.id
WHERE r.rn > 1;
```

**Expected output**:

```text
Query OK, 3 rows affected (0.02 sec)
```

**Why this output appears**: The CTE ranks users by `created_at` descending within each email; the `DELETE ... JOIN` removes all but the most recent user per email.

#### Real-World Cases

- **Customer data cleanup**: Removing duplicate customer records.
- **Log deduplication**: Keeping the latest event per unique key.
- **Data migration**: Ensuring uniqueness before loading into a new system.

---

## References

- MySQL 8.0 Reference Manual – Window Function Descriptions – https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html
- MySQL 8.0 Reference Manual – Window Function Concepts and Syntax – https://dev.mysql.com/doc/refman/8.0/en/window-functions-usage.html
- PostgreSQL Documentation – Window Functions – https://www.postgresql.org/docs/current/functions-window.html
- PostgreSQL Documentation – Window Functions Tutorial – https://www.postgresql.org/docs/current/tutorial-window.html
- Oracle Database SQL Language Reference – ROW_NUMBER – https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/ROW_NUMBER.html
- Oracle Database SQL Language Reference – Ranking Functions – https://docs.oracle.com/en/database/oracle/oracle-database/23/sqlrf/RANK.html
- Microsoft Learn – Ranking Functions (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/functions/ranking-functions-transact-sql
- Microsoft Learn – Delete Duplicated Rows – https://learn.microsoft.com/en-us/answers/questions/96141/delete-duplicated-rows
- Google Cloud BigQuery Documentation – Numbering Functions – https://cloud.google.com/bigquery/docs/reference/standard-sql/numbering_functions
- SAP HANA SQL Reference – Window Functions – https://help.sap.com/docs/SAP_HANA_PLATFORM/4fe29514fd584807ac9f2a04f6754767/20a566dc75191014b5d26f5b154467d4.html
- SQL Authority – Difference Between ROW_NUMBER, RANK, and DENSE_RANK – https://blog.sqlauthority.com/2015/09/03/sql-server-difference-between-row_number-rank-and-dense_rank-notes-from-the-field-096/
- Database.Guide – A Comparison of 6 SQL Rank Functions – https://database.guide/a-comparison-of-6-sql-rank-functions/
- LearnSQL – Window Functions in MySQL – https://learnsql.com/blog/window-functions-in-mysql/