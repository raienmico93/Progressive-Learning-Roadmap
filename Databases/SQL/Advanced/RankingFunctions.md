# SQL Ranking & Distribution Functions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Ranking and distribution functions are specialized window functions that assign a rank, bucket, or statistical position to each row within a partition based on a specified ordering, without collapsing the rows.

**Technical Definition:** These functions operate within the `OVER` clause, using `PARTITION BY` to define groups and `ORDER BY` to establish sequence. Sequential ranking functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`) assign ordinal positions; quantile functions (`NTILE`) divide partitions into buckets; and statistical distribution functions (`PERCENT_RANK`, `CUME_DIST`, `PERCENTILE_CONT`, `PERCENTILE_DISC`) compute percentile positions or inverse distribution values.

**Beginner-Friendly Explanation:** Think of a class ranking. `ROW_NUMBER` gives every student a unique position (1, 2, 3, 4...). `RANK` gives tied students the same rank but skips the next number (1, 2, 2, 4...). `DENSE_RANK` gives tied students the same rank without skipping (1, 2, 2, 3...). `NTILE` divides students into equal-sized groups (top 25%, next 25%, etc.). `PERCENT_RANK` and `CUME_DIST` tell you what percentage of students you scored above. `PERCENTILE_CONT` and `PERCENTILE_DISC` find the value at a specific percentile.

### Key Characteristics

- **Require `ORDER BY`:** Ranking functions require an `ORDER BY` clause in the `OVER` specification .
- **Tie handling differs:** `ROW_NUMBER` ignores ties; `RANK` and `DENSE_RANK` acknowledge them differently .
- **Partition-relative:** All results are computed within each partition defined by `PARTITION BY` .
- **Non-deterministic with ties:** When ties exist, `ROW_NUMBER` and `NTILE` may assign values arbitrarily; use a unique key for deterministic results .

### Prerequisites

- **Window function fundamentals:** Understanding of `OVER`, `PARTITION BY`, and `ORDER BY`.
- **Aggregate awareness:** Familiarity with `COUNT`, `SUM`, and statistical concepts.
- **Set-based thinking:** Comfort with operating on groups of rows while retaining row detail.

### Related Programming Areas

- **Data Analysis & BI:** Rankings, percentiles, and distribution analysis.
- **Application Development:** Top-N per group, leaderboards, and bucketed reporting.
- **Data Engineering:** Quantile-based data splits and statistical profiling.
- **Reporting:** Median calculations, cumulative distributions, and percentile thresholds.

### Core Concepts / Features

1. Sequential Ranking (`ROW_NUMBER`, `RANK`, `DENSE_RANK`)
2. Quantile Distribution (`NTILE`)
3. Statistical Distribution (`PERCENT_RANK`, `CUME_DIST`)
4. Inverse Distribution (`PERCENTILE_CONT`, `PERCENTILE_DISC`)

---

## 1. Sequential Ranking: ROW_NUMBER vs. RANK vs. DENSE_RANK

### Definitions

**Core Definition:** Sequential ranking functions assign a numeric rank to each row within a partition based on an `ORDER BY` specification, differing in how they handle ties.

**Technical Definition:** `ROW_NUMBER()` assigns a unique sequential integer to each row, ignoring ties. `RANK()` assigns the same rank to tied rows and leaves gaps in the sequence. `DENSE_RANK()` assigns the same rank to tied rows but does not leave gaps . If no ties exist, all three functions return identical results .

**Beginner-Friendly Explanation:** Imagine a race with a photo finish. `ROW_NUMBER` is the order the judges list the runners (arbitrary for ties). `RANK` is the medal ceremony (two silvers mean no bronze). `DENSE_RANK` is the medal ceremony without skipping (two silvers means the next is bronze).

### Purposes

- To assign unique row numbers for pagination or deduplication (`ROW_NUMBER`).
- To rank items with ties while reflecting the number of preceding items (`RANK`).
- To rank distinct values without gaps (`DENSE_RANK`).
- To identify top-N items per group.

### Syntax Rules and Structure

```sql
ROW_NUMBER() OVER ( [PARTITION BY partition_expression] ORDER BY order_expression )
RANK()       OVER ( [PARTITION BY partition_expression] ORDER BY order_expression )
DENSE_RANK() OVER ( [PARTITION BY partition_expression] ORDER BY order_expression )
```

**Component Breakdown:**

| Function | Tie Behaviour | Sequence Example |
|----------|--------------|------------------|
| `ROW_NUMBER()` | Unique number per row | 1, 2, 3, 4, 5 |
| `RANK()` | Same rank for ties; gaps after ties | 1, 2, 2, 4, 5 |
| `DENSE_RANK()` | Same rank for ties; no gaps | 1, 2, 2, 3, 4 |

**Syntax Rules:**

- All three functions require `ORDER BY` in the `OVER` clause .
- `PARTITION BY` is optional; without it, the entire result set is one partition.
- `ROW_NUMBER` is non-deterministic when ties exist .

**Constraints and Limitations:**

- `ROW_NUMBER` gives arbitrary results for tied values unless the `ORDER BY` includes a unique tiebreaker .
- `RANK` leaves gaps that may not reflect logical position.
- `DENSE_RANK` maximum value equals the number of distinct values .

### Annotated Complete Code Examples

**Example 1: Comparing the Three Functions**

```sql
-- Setup
CREATE TABLE sales (
    salesperson VARCHAR(50),
    region      VARCHAR(50),
    sales_amt   NUMERIC(10, 2)
);

INSERT INTO sales VALUES
    ('Alice', 'North', 100.00),
    ('Bob',   'North', 200.00),
    ('Carol', 'North', 200.00),  -- Tie with Bob
    ('David', 'North', 300.00),
    ('Eve',   'North', 300.00),  -- Tie with David
    ('Frank', 'North', 500.00);

-- Query: All three ranking functions
SELECT salesperson, sales_amt,
       ROW_NUMBER() OVER (ORDER BY sales_amt) AS row_num,
       RANK()       OVER (ORDER BY sales_amt) AS rank,
       DENSE_RANK() OVER (ORDER BY sales_amt) AS dense_rank
FROM sales
ORDER BY sales_amt;

-- Expected Output:
--  salesperson | sales_amt | row_num | rank | dense_rank
-- -------------+-----------+---------+------+------------
--  Alice       | 100.00    |       1 |    1 |          1
--  Bob         | 200.00    |       2 |    2 |          2
--  Carol       | 200.00    |       3 |    2 |          2
--  David       | 300.00    |       4 |    4 |          3
--  Eve         | 300.00    |       5 |    4 |          3
--  Frank       | 500.00    |       6 |    6 |          4
```

**Why this output occurs:** Bob and Carol tie at 200. `ROW_NUMBER` assigns 2 and 3 arbitrarily. `RANK` assigns both rank 2, then skips to 4 for David. `DENSE_RANK` assigns both rank 2, then assigns 3 to David (no gap) .

**Example 2: Top-N Per Group**

```sql
-- Query: Top 2 sales per region (using ROW_NUMBER)
SELECT salesperson, region, sales_amt
FROM (
    SELECT salesperson, region, sales_amt,
           ROW_NUMBER() OVER (PARTITION BY region ORDER BY sales_amt DESC) AS rn
    FROM sales
) ranked
WHERE rn <= 2;

-- Expected Output (assuming single region for simplicity):
--  salesperson | region | sales_amt
-- -------------+--------+-----------
--  Frank       | North  | 500.00
--  David       | North  | 300.00
```

**Why this output occurs:** `ROW_NUMBER` assigns unique numbers within each region. The outer query filters to the top 2. If ties existed at the boundary, `ROW_NUMBER` would arbitrarily pick one .

### Real-World Cases

**Case 1: Pagination**

An application uses `ROW_NUMBER() OVER (ORDER BY created_at DESC)` to implement paginated API responses. The outer query filters for row numbers between page boundaries.

**Case 2: Deduplication**

A data engineer uses `ROW_NUMBER() OVER (PARTITION BY email ORDER BY updated_at DESC)` to identify the most recent record for each email address, then deletes duplicates.

### References

- Microsoft Q&A — What is the difference among Row_Number, Rank and Dense_Rank - https://learn.microsoft.com/en-us/answers/questions/211223/what-is-the-difference-among-row-number-rank-and-d 
- Learn Microsoft — Ranking Functions: RANK, DENSE_RANK, and NTILE - https://learn.microsoft.com/ko-kr/archive/blogs/craigfr/ranking-functions-rank-dense_rank-and-ntile 
- SQL Authority — What's the Difference between ROW_NUMBER, RANK, and DENSE_RANK? - https://blog.sqlauthority.com/2015/09/03/sql-server-whats-the-difference-between-row_number-rank-and-dense_rank-notes-from-the-field-096/ 
- Learn Microsoft — Use RANK, AGGREGATE, and OFFSET functions - https://learn.microsoft.com/et-ee/training/modules/write-queries-that-use-window-functions/4-use-rank-aggregate-offset-functions 

---

## 2. Quantile Distribution: NTILE

### Definitions

**Core Definition:** `NTILE` divides the rows of an ordered partition into a specified number of approximately equal groups (buckets), assigning a bucket number to each row.

**Technical Definition:** `NTILE(n)` distributes rows into `n` groups numbered from 1 to `n`. If the row count is not evenly divisible by `n`, the larger groups come first, with sizes differing by at most one row . For example, 53 rows into 5 buckets produces three buckets of 11 and two of 10 .

**Beginner-Friendly Explanation:** `NTILE` is like splitting a deck of cards into piles. If you have 53 cards and 5 players, three players get 11 cards and two get 10. The first players get the extra cards.

### Purposes

- To split data into quartiles, deciles, or other quantile buckets.
- To create equal-sized groups for comparative analysis.
- To implement percentile-based segmentation (top 25%, middle 50%, bottom 25%).
- To distribute processing work evenly across parallel workers.

### Syntax Rules and Structure

```sql
NTILE(integer_expression) OVER ( [PARTITION BY partition_expression] ORDER BY order_expression )
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `integer_expression` | Number of buckets (positive integer) |
| `ORDER BY` | Determines bucket assignment order |

**Syntax Rules:**

- `integer_expression` must be a positive integer constant .
- `ORDER BY` is required to determine the order of bucket assignment .
- If rows are evenly divisible by `n`, buckets are equal-sized .
- If not evenly divisible, larger buckets come first .

**Constraints and Limitations:**

- `NTILE` is non-deterministic when ties exist in the `ORDER BY` .
- Equal values can be split across adjacent buckets .
- Cannot reference columns in `integer_expression`; only constants are allowed .

### Annotated Complete Code Examples

**Example 1: NTILE with Uneven Buckets**

```sql
-- Using the sales table with 6 rows
SELECT salesperson, sales_amt,
       NTILE(4) OVER (ORDER BY sales_amt) AS quartile
FROM sales
ORDER BY sales_amt;

-- Expected Output:
--  salesperson | sales_amt | quartile
-- -------------+-----------+----------
--  Alice       | 100.00    |        1
--  Bob         | 200.00    |        1
--  Carol       | 200.00    |        2
--  David       | 300.00    |        2
--  Eve         | 300.00    |        3
--  Frank       | 500.00    |        4
```

**Why this output occurs:** 6 rows into 4 buckets: the first 2 buckets get 2 rows each, and the last 2 get 1 row each. The larger buckets come first . Note that Carol and Bob tie but are split across buckets 1 and 2 — this is the non-determinism .

**Example 2: NTILE with PARTITION BY**

```sql
-- Query: Quartiles within each region
SELECT salesperson, region, sales_amt,
       NTILE(2) OVER (PARTITION BY region ORDER BY sales_amt) AS half
FROM sales
ORDER BY region, sales_amt;
```

**Why this works:** `PARTITION BY region` creates separate buckets within each region. The `NTILE(2)` divides each region's rows into two halves.

### Real-World Cases

**Case 1: Customer Segmentation**

A marketing team uses `NTILE(4)` on customer lifetime value to segment customers into four tiers (platinum, gold, silver, bronze) of equal size.

**Case 2: A/B Test Assignment**

An experiment platform uses `NTILE(2)` to randomly split users into control and treatment groups of equal size.

### References

- Microsoft Learn — NTILE (Transact-SQL) - https://learn.microsoft.com/en-us/previous-versions/sql/sql-server-2005/ms175126(v=sql.90) 
- Microsoft Learn — NTILE (Transact-SQL, Portuguese) - https://learn.microsoft.com/pt-br/sql/t-sql/functions/ntile-transact-sql 
- Oracle Database Data Warehousing Guide - https://docs.oracle.com/en/database/oracle/oracle-database/26/dwhsg/database-data-warehousing-guide.pdf 

---

## 3. Statistical Distribution Functions: PERCENT_RANK and CUME_DIST

### Definitions

**Core Definition:** `PERCENT_RANK` and `CUME_DIST` are window functions that compute the relative position of a row within its partition as a percentage, using different formulas.

**Technical Definition:** `PERCENT_RANK()` returns the percentage of partition values less than the current row's value, excluding the highest value. The formula is `(rank - 1) / (rows - 1)`, where `rank` is the row's rank and `rows` is the partition row count . `CUME_DIST()` returns the cumulative distribution: the percentage of partition rows with values less than or equal to the current row's value .

**Beginner-Friendly Explanation:** If you scored 80 on a test, `PERCENT_RANK` tells you what fraction of the class you outperformed. `CUME_DIST` tells you what fraction scored the same as or lower than you.

### Purposes

- To compute percentile ranks (e.g., "top 10%").
- To calculate cumulative distribution for statistical analysis.
- To identify outliers or unusual values.
- To support percentile-based filtering.

### Syntax Rules and Structure

```sql
PERCENT_RANK() OVER ( [PARTITION BY partition_expression] ORDER BY order_expression )
CUME_DIST()    OVER ( [PARTITION BY partition_expression] ORDER BY order_expression )
```

**Component Breakdown:**

| Function | Formula | Range |
|----------|---------|-------|
| `PERCENT_RANK()` | `(rank - 1) / (rows - 1)` | 0 to 1  |
| `CUME_DIST()` | `rows ≤ current / total rows` | 0 to 1  |

**Syntax Rules:**

- Both functions require `ORDER BY` .
- Without `ORDER BY`, all rows are peers, and `CUME_DIST` returns 1 for all rows .
- `PERCENT_RANK` returns 0 for the first row (lowest value) .

**Constraints and Limitations:**

- `PERCENT_RANK` and `CUME_DIST` are non-deterministic with ties.
- The interpretation differs: `PERCENT_RANK` excludes the current row's peers from the "less than" count; `CUME_DIST` includes them.

### Annotated Complete Code Examples

**Example 1: PERCENT_RANK and CUME_DIST**

```sql
-- Query: Compare PERCENT_RANK and CUME_DIST
SELECT salesperson, sales_amt,
       PERCENT_RANK() OVER (ORDER BY sales_amt) AS pct_rank,
       CUME_DIST()    OVER (ORDER BY sales_amt) AS cume_dist
FROM sales
ORDER BY sales_amt;

-- Expected Output:
--  salesperson | sales_amt | pct_rank | cume_dist
-- -------------+-----------+----------+-----------
--  Alice       | 100.00    | 0.0000   | 0.1667
--  Bob         | 200.00    | 0.2000   | 0.5000
--  Carol       | 200.00    | 0.2000   | 0.5000
--  David       | 300.00    | 0.6000   | 0.8333
--  Eve         | 300.00    | 0.6000   | 0.8333
--  Frank       | 500.00    | 1.0000   | 1.0000
```

**Why this output occurs:** Alice has the lowest salary: `PERCENT_RANK = 0`, `CUME_DIST = 1/6 = 0.1667`. Bob and Carol tie: `PERCENT_RANK = (2-1)/(6-1) = 0.2`, `CUME_DIST = 3/6 = 0.5` (3 rows ≤ 200). Frank has the highest: both return 1 .

### Real-World Cases

**Case 1: Performance Benchmarking**

An analytics team uses `PERCENT_RANK` to identify which queries fall in the slowest 10% of execution times.

**Case 2: Customer Spend Percentile**

A marketing query uses `CUME_DIST` to find customers whose spend is at or below a certain threshold, enabling targeted promotions.

### References

- MySQL 8.0 Reference Manual — Window Function Descriptions (PERCENT_RANK, CUME_DIST) - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/window-function-descriptions.html 
- MySQL 8.4 Reference Manual — Window Function Descriptions (Chinese) - https://dev.mysql.com.tw/doc/refman/8.4/en/window-function-descriptions.html 

---

## 4. Inverse Distribution Functions: PERCENTILE_CONT and PERCENTILE_DISC

### Definitions

**Core Definition:** Inverse distribution functions return the value at a specified percentile within a group, either by interpolation (`PERCENTILE_CONT`) or by returning an actual value from the dataset (`PERCENTILE_DISC`).

**Technical Definition:** `PERCENTILE_CONT(x)` computes a value by linear interpolation between rows after ordering them. The row number is `RN = 1 + x * (n - 1)`; the result is interpolated between rows at `CEIL(RN)` and `FLOOR(RN)` . `PERCENTILE_DISC(x)` scans up the `CUME_DIST` values and returns the first value whose cumulative distribution is greater than or equal to `x` . `PERCENTILE_CONT` may return a value not present in the dataset; `PERCENTILE_DISC` always returns an actual value .

**Beginner-Friendly Explanation:** If you want the median salary, `PERCENTILE_CONT(0.5)` interpolates between the two middle salaries if there's an even number of rows. `PERCENTILE_DISC(0.5)` picks the actual salary value at the 50th percentile.

### Purposes

- To calculate medians and other percentiles.
- To find thresholds (e.g., 90th percentile response time).
- To compute interpolated values for continuous distributions.
- To return discrete values for discrete distributions.

### Syntax Rules and Structure

```sql
PERCENTILE_CONT(numeric_literal) WITHIN GROUP (ORDER BY order_by_expression) OVER ( [PARTITION BY partition_expression] )
PERCENTILE_DISC(numeric_literal) WITHIN GROUP (ORDER BY order_by_expression) OVER ( [PARTITION BY partition_expression] )
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `numeric_literal` | Percentile value (0.0 to 1.0)  |
| `WITHIN GROUP (ORDER BY ...)` | Sort specification for the distribution  |
| `OVER (PARTITION BY ...)` | Partition specification  |

**Syntax Rules:**

- `numeric_literal` must be between 0.0 and 1.0 .
- Only one `ORDER BY` expression is allowed .
- The expression must evaluate to a numeric type .
- `ORDER BY` and frame clauses cannot be specified in the `OVER` clause .
- `NULL` values are ignored .

**Constraints and Limitations:**

- `PERCENTILE_CONT` is non-deterministic .
- Both functions require the `WITHIN GROUP` syntax, which differs from standard window functions.
- SQL Server compatibility level must be 110 or higher for `WITHIN GROUP` .

### Annotated Complete Code Examples

**Example 1: Median Salary per Department**

```sql
-- Query: Median salary using both functions
SELECT DISTINCT department,
       PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) OVER (PARTITION BY department) AS median_cont,
       PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY salary) OVER (PARTITION BY department) AS median_disc
FROM employees;

-- Expected Output (example):
--  department  | median_cont | median_disc
-- -------------+-------------+-------------
--  Engineering | 34.375      | 32.6923
--  Marketing   | 17.427850   | 16.5865
```

**Why this output occurs:** `PERCENTILE_CONT(0.5)` interpolates the median value, which may not exist in the data. `PERCENTILE_DISC(0.5)` returns an actual salary value from the dataset .

**Example 2: 90th Percentile Response Time**

```sql
-- Query: 90th percentile response time per service
SELECT service_name,
       PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY response_ms) OVER (PARTITION BY service_name) AS p90
FROM api_logs;
```

**Why this works:** The function computes the 90th percentile response time for each service, useful for SLA monitoring.

### Real-World Cases

**Case 1: SLA Monitoring**

An operations team uses `PERCENTILE_CONT(0.95)` to monitor the 95th percentile API response time, ensuring 95% of requests complete within the target.

**Case 2: Salary Benchmarking**

An HR department uses `PERCENTILE_DISC(0.25)`, `(0.5)`, and `(0.75)` to establish salary quartiles for compensation planning.

### References

- Oracle Database Data Warehousing Guide — Inverse Percentile Example - https://docs.oracle.com/en/database/oracle/oracle-database/12.2/dwhsg/sql-analysis-reporting-data-warehouses.html 
- Microsoft Learn — PERCENTILE_CONT (Transact-SQL) - https://learn.microsoft.com/de-de/sql/t-sql/functions/percentile-cont-transact-sql 
- Microsoft Learn — PERCENTILE_CONT (Transact-SQL, Chinese) - https://learn.microsoft.com/zh-cn/sql/t-sql/functions/percentile-cont-transact-sql 
- SAP Help Portal — Inverse Distribution Functions - https://help.sap.com/docs/SAP_HANA_PLATFORM/4fe29514fd584807ac9f2a04f6754767/91e9b37c6bb1405e8c9f205fd19a3c9a.html 

---

## Summary Table of Ranking & Distribution Functions

| Function | Purpose | Tie Handling | Returns Actual Value? |
|----------|---------|--------------|----------------------|
| `ROW_NUMBER()` | Unique row number | Arbitrary | N/A |
| `RANK()` | Rank with gaps | Same rank, skips | N/A |
| `DENSE_RANK()` | Rank without gaps | Same rank, no skips | N/A |
| `NTILE(n)` | Bucket assignment | Splits across buckets | N/A |
| `PERCENT_RANK()` | Percentile rank | Same for ties | N/A |
| `CUME_DIST()` | Cumulative distribution | Same for ties | N/A |
| `PERCENTILE_CONT()` | Interpolated percentile | N/A | No (interpolated) |
| `PERCENTILE_DISC()` | Discrete percentile | N/A | Yes (actual value) |

---

## References

- Microsoft Q&A — What is the difference among Row_Number, Rank and Dense_Rank - https://learn.microsoft.com/en-us/answers/questions/211223/what-is-the-difference-among-row-number-rank-and-d 
- Oracle Database Data Warehousing Guide — NTILE and Ties - https://docs.oracle.com/en/database/oracle/oracle-database/26/dwhsg/database-data-warehousing-guide.pdf 
- MySQL 8.0 Reference Manual — Window Function Descriptions - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/window-function-descriptions.html 
- Oracle Database Data Warehousing Guide — Inverse Percentile Example - https://docs.oracle.com/en/database/oracle/oracle-database/12.2/dwhsg/sql-analysis-reporting-data-warehouses.html 
- Learn Microsoft — Use RANK, AGGREGATE, and OFFSET functions - https://learn.microsoft.com/et-ee/training/modules/write-queries-that-use-window-functions/4-use-rank-aggregate-offset-functions 
- Microsoft Learn — PERCENTILE_CONT (Transact-SQL) - https://learn.microsoft.com/de-de/sql/t-sql/functions/percentile-cont-transact-sql 
- PostgreSQL Documentation — Aggregate Functions - https://www.postgresql.org/docs/12/functions-aggregate.html 
- MySQL 8.0 Reference Manual — Window Function Concepts - https://dev.mysql.com/doc/refman/8.0/en/window-functions-usage.html 
- Learn Microsoft — Ranking Functions: RANK, DENSE_RANK, and NTILE - https://learn.microsoft.com/ko-kr/archive/blogs/craigfr/ranking-functions-rank-dense_rank-and-ntile 
- Microsoft Learn — NTILE (Transact-SQL) - https://learn.microsoft.com/en-us/previous-versions/sql/sql-server-2005/ms175126(v=sql.90) 
- MySQL 8.4 Reference Manual — Window Function Descriptions (Chinese) - https://dev.mysql.com.tw/doc/refman/8.4/en/window-function-descriptions.html 
- SAP Help Portal — Inverse Distribution Functions - https://help.sap.com/docs/SAP_HANA_PLATFORM/4fe29514fd584807ac9f2a04f6754767/91e9b37c6bb1405e8c9f205fd19a3c9a.html 
- Microsoft Learn — Ranking Functions (Transact-SQL) - https://learn.microsoft.com/th-th/sql/t-sql/functions/ranking-functions-transact-sql 
- Microsoft Learn — PERCENTILE_CONT (Transact-SQL, Chinese) - https://learn.microsoft.com/zh-cn/sql/t-sql/functions/percentile-cont-transact-sql 
- MySQL Developer Zone — MySQL 8.0.2: Introducing Window Functions - https://dev.mysql.com/blog-archive/mysql-8-0-2-introducing-window-functions/ 
- SQL Authority — What's the Difference between ROW_NUMBER, RANK, and DENSE_RANK? - https://blog.sqlauthority.com/2015/09/03/sql-server-whats-the-difference-between-row_number-rank-and-dense_rank-notes-from-the-field-096/ 
- Microsoft Learn — NTILE (Transact-SQL, Portuguese) - https://learn.microsoft.com/pt-br/sql/t-sql/functions/ntile-transact-sql 