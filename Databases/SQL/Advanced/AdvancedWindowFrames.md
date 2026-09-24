# Advanced Window Frames & Boundaries: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A window frame is a subset of rows within the current partition that a window function uses to perform its calculation. It is defined by specifying a frame unit (`ROWS`, `RANGE`, or `GROUPS`) and boundaries relative to the current row.

**Technical Definition:** The frame clause in a window function's `OVER` specification determines which rows are included in the computation for each row. It can be written as either `{ RANGE | ROWS | GROUPS } frame_start` or `{ RANGE | ROWS | GROUPS } BETWEEN frame_start AND frame_end`. The frame unit determines whether boundaries are counted in physical rows, logical values, or peer groups .

**Beginner-Friendly Explanation:** Imagine a spreadsheet where you want to calculate a running total. The "frame" tells the database which rows to include in the calculation for each row. Do you want just the rows above? The current row and the one before it? All rows with the same date? The frame is the answer to "which neighbors count?"

### Key Characteristics

- **Three frame units:** `ROWS` (physical position), `RANGE` (logical value distance), and `GROUPS` (peer group count) .
- **Boundary keywords:** `UNBOUNDED PRECEDING`, `N PRECEDING`, `CURRENT ROW`, `N FOLLOWING`, `UNBOUNDED FOLLOWING` .
- **Default frame:** When `ORDER BY` is present but no frame is specified, the default is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` .
- **Peer awareness:** `RANGE` and `GROUPS` treat rows with equal `ORDER BY` values as "peers" and include them together .
- **Performance variance:** `ROWS` is typically faster than `RANGE` because it avoids value comparisons and can use in-memory worktables .

### Prerequisites

- **Window function fundamentals:** Understanding of `OVER`, `PARTITION BY`, and `ORDER BY`.
- **Aggregate function awareness:** Familiarity with `SUM`, `AVG`, `COUNT`, and ranking functions.
- **Set-based thinking:** Comfort with operating on subsets of rows.

### Related Programming Areas

- **Data Analysis & BI:** Running totals, moving averages, cumulative metrics.
- **Application Development:** Pagination, ranking, trend analysis.
- **Data Engineering:** Windowed aggregations in ETL pipelines.

### Core Concepts / Features

1. Frame Units: ROWS, RANGE, and GROUPS
2. Frame Boundaries
3. Dynamic Offsets
4. The Performance Trap: Default RANGE vs. ROWS

---

## 1. Frame Units: ROWS, RANGE, and GROUPS

### Definitions

**Core Definition:** Frame units determine how frame boundaries are measured: `ROWS` counts physical row positions, `RANGE` measures logical value distance, and `GROUPS` counts peer groups.

**Technical Definition:** `ROWS` defines boundaries by the number of rows before or after the current row. `RANGE` defines boundaries by the difference between the current row's `ORDER BY` value and surrounding row values. `GROUPS` defines boundaries by the number of peer groups before or after the current row's peer group .

**Beginner-Friendly Explanation:** `ROWS` is like counting seats in a row: "the 3 seats before me." `RANGE` is like measuring distance: "everyone within 10 meters of me." `GROUPS` is like counting tables: "the 2 tables before mine."

### Purposes

- To control precisely which rows contribute to each window calculation.
- To implement running totals, moving averages, and cumulative sums.
- To handle peer rows (ties) consistently and predictably.
- To optimize performance by choosing the appropriate frame unit.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
function_name ( expression ) OVER (
    [ PARTITION BY partition_expression ]
    [ ORDER BY order_expression ]
    { ROWS | RANGE | GROUPS } BETWEEN frame_start AND frame_end
)
```

**Component Breakdown:**

| Frame Unit | Boundary Measured By | Offset Type |
|------------|---------------------|-------------|
| `ROWS` | Physical row position | Integer (row count) |
| `RANGE` | Logical value distance | Numeric or interval |
| `GROUPS` | Peer group count | Integer (group count) |

**Syntax Rules:**

- `ROWS` supports integer offsets (e.g., `3 PRECEDING`) .
- `RANGE` requires exactly one `ORDER BY` column with a numeric, date, or interval type .
- `GROUPS` requires an `ORDER BY` clause; rows with equal `ORDER BY` values form a peer group .
- `GROUPS` is part of SQL:2011 (Feature T620) and is supported by PostgreSQL and SQLite, but **not** by SQL Server .

**Constraints and Limitations:**

- `RANGE` with numeric offsets cannot be used with descending `ORDER BY` in the same way as ascending .
- `GROUPS` is not universally supported; SQL Server does not implement it .
- The `ORDER BY` column for `RANGE` must be numeric or a type that supports subtraction .

### Annotated Complete Code Examples

**Example 1: ROWS vs. RANGE with Ties**

```sql
-- Setup: Sales with duplicate dates
CREATE TABLE sales (
    sale_date DATE,
    amount    NUMERIC(10, 2)
);

INSERT INTO sales VALUES
    ('2026-01-01', 100.00),
    ('2026-01-01', 200.00),  -- Same date (peer)
    ('2026-01-02', 150.00),
    ('2026-01-03', 300.00);

-- ROWS: Physical position (1 row before)
SELECT sale_date, amount,
       SUM(amount) OVER (ORDER BY sale_date ROWS 1 PRECEDING) AS rows_sum
FROM sales;

-- Expected Output:
--  sale_date  | amount  | rows_sum
-- ------------+---------+----------
--  2026-01-01 | 100.00  | 100.00
--  2026-01-01 | 200.00  | 300.00
--  2026-01-02 | 150.00  | 350.00
--  2026-01-03 | 300.00  | 450.00

-- RANGE: Logical value (1 day before)
SELECT sale_date, amount,
       SUM(amount) OVER (ORDER BY sale_date RANGE '1 day' PRECEDING) AS range_sum
FROM sales;

-- Expected Output:
--  sale_date  | amount  | range_sum
-- ------------+---------+----------
--  2026-01-01 | 100.00  | 100.00
--  2026-01-01 | 200.00  | 300.00
--  2026-01-02 | 150.00  | 450.00
--  2026-01-03 | 300.00  | 450.00
```

**Why the outputs differ:** With `ROWS 1 PRECEDING`, the second Jan 1 row includes the first Jan 1 row (1 physical row back). With `RANGE '1 day' PRECEDING`, the Jan 2 row includes both Jan 1 rows because they fall within the previous 1-day value range .

### Real-World Cases

**Case 1: Daily Sales with Multiple Transactions**

A retail analyst wants a running total by date. If multiple transactions share the same date, `RANGE` includes all same-day transactions in the cumulative sum, while `ROWS` includes only the physically previous row.

**Case 2: Sensor Data Smoothing**

An IoT system uses `RANGE INTERVAL '1 hour' PRECEDING` to compute a moving average over all readings within the last hour, regardless of how many readings occurred .

### References

- InfluxData Documentation — Window Frames (Frame Units) - https://test2.docs.influxdata.com/influxdb3/clustered/reference/sql/functions/window/
- PostgreSQL Mailing List — GROUPS option specification - https://www.postgresql.org/message-id/CAGMVOdtWkb9X7dUh7vjaCaiH34UGFg88unXYTEOub0Rk0swSXw%40mail.gmail.com
- Microsoft Learn — T620, WINDOW clause: GROUPS option - https://learn.microsoft.com/en-ie/openspecs/sql_standards/ms-tsqliso02/474febd5-3748-4ac0-9621-a4238b93668f

---

## 2. Frame Boundaries

### Definitions

**Core Definition:** Frame boundaries define the start and end points of the window frame relative to the current row, using keywords like `UNBOUNDED PRECEDING`, `CURRENT ROW`, and `UNBOUNDED FOLLOWING`.

**Technical Definition:** A frame boundary can be `UNBOUNDED PRECEDING` (start of partition), `N PRECEDING` (N units before current row), `CURRENT ROW` (the current row or peer group), `N FOLLOWING` (N units after current row), or `UNBOUNDED FOLLOWING` (end of partition). The start boundary must not be greater than the end boundary .

**Beginner-Friendly Explanation:** Boundaries are like bookends. `UNBOUNDED PRECEDING` is the first page of the chapter. `CURRENT ROW` is where you're reading now. `UNBOUNDED FOLLOWING` is the last page.

### Purposes

- To define exactly which rows are included in each window calculation.
- To create sliding windows of fixed size.
- To implement cumulative calculations from the partition start.
- To look ahead to future rows.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
{ ROWS | RANGE | GROUPS } BETWEEN frame_start AND frame_end
```

**Boundary Keywords:**

| Boundary | Meaning |
|----------|---------|
| `UNBOUNDED PRECEDING` | First row/range/group in partition  |
| `N PRECEDING` | N units before current row  |
| `CURRENT ROW` | Current row (ROWS) or current peer group (RANGE/GROUPS)  |
| `N FOLLOWING` | N units after current row  |
| `UNBOUNDED FOLLOWING` | Last row/range/group in partition  |

**Syntax Rules:**

- If `BETWEEN` is omitted, the frame is `BETWEEN frame_start AND CURRENT ROW` .
- The start boundary cannot be after the end boundary .
- `CURRENT ROW` in `ROWS` means the current physical row; in `RANGE`/`GROUPS` it means the current peer group .

**Constraints and Limitations:**

- `UNBOUNDED FOLLOWING` as a start boundary is invalid.
- `UNBOUNDED PRECEDING` as an end boundary is invalid.

### Annotated Complete Code Examples

**Example 1: Cumulative Sum with UNBOUNDED PRECEDING**

```sql
-- Running total from partition start
SELECT sale_date, amount,
       SUM(amount) OVER (ORDER BY sale_date ROWS UNBOUNDED PRECEDING) AS running_total
FROM sales;

-- Expected Output:
--  sale_date  | amount  | running_total
-- ------------+---------+---------------
--  2026-01-01 | 100.00  | 100.00
--  2026-01-01 | 200.00  | 300.00
--  2026-01-02 | 150.00  | 450.00
--  2026-01-03 | 300.00  | 750.00
```

**Why this works:** `ROWS UNBOUNDED PRECEDING` is shorthand for `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. Each row's sum includes all rows from the partition start through the current physical row .

### Real-World Cases

**Case 1: Financial Running Balance**

A bank statement shows a running balance using `SUM(amount) OVER (ORDER BY transaction_date ROWS UNBOUNDED PRECEDING)`.

**Case 2: Year-to-Date Sales**

A sales dashboard uses `PARTITION BY YEAR(sale_date)` with `ROWS UNBOUNDED PRECEDING` to compute YTD totals that reset each year.

### References

- NodeDB Documentation — Window Frames (Boundaries) - https://github.com/NodeDB-Lab/nodedb-docs/blob/main/docs/sql/window-frames.rdx
- InfluxData Documentation — Frame Boundaries - https://docs.influxdata.com/influxdb3/cloud-serverless/reference/sql/functions/window/#lag

---

## 3. Dynamic Offsets

### Definitions

**Core Definition:** Dynamic offsets use `N PRECEDING` and `N FOLLOWING` to create sliding windows of a specified size, where `N` can be a literal or a column value.

**Technical Definition:** The `<offset>` in a frame boundary specifies how many units before or after the current row the frame extends. For `ROWS`, the offset is an integer row count. For `RANGE`, it is a value of the same type as the `ORDER BY` column. For `GROUPS`, it is an integer group count .

**Beginner-Friendly Explanation:** Dynamic offsets let you say "include the last 7 rows" or "include everything within 3 days." It's how you build sliding windows that move with each row.

### Purposes

- To compute moving averages over a fixed number of rows.
- To create rolling sums for trend analysis.
- To implement sliding window calculations with date or numeric ranges.
- To support gap analysis and period-over-period comparisons.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
-- Fixed row window
ROWS BETWEEN 3 PRECEDING AND CURRENT ROW

-- Fixed value range window
RANGE BETWEEN INTERVAL '3 days' PRECEDING AND CURRENT ROW

-- Fixed peer group window
GROUPS BETWEEN 2 PRECEDING AND CURRENT ROW
```

**Component Breakdown:**

| Frame Unit | Offset Type | Example |
|------------|-------------|---------|
| `ROWS` | Integer | `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` |
| `RANGE` | Numeric/Interval | `RANGE BETWEEN INTERVAL '3 days' PRECEDING AND CURRENT ROW` |
| `GROUPS` | Integer | `GROUPS BETWEEN 2 PRECEDING AND CURRENT ROW` |

**Syntax Rules:**

- `ROWS` offsets are always integer row counts .
- `RANGE` offsets must match the data type of the `ORDER BY` column .
- `GROUPS` offsets are integer peer group counts .
- The window moves with each row; each row gets its own frame.

**Constraints and Limitations:**

- `RANGE` with interval offsets requires the `ORDER BY` column to be a date/time type.
- `RANGE` with numeric offsets requires the `ORDER BY` column to be numeric.
- `GROUPS` is not supported in SQL Server .

### Annotated Complete Code Examples

**Example 1: 3-Row Moving Average**

```sql
-- 3-row moving average (current + 2 preceding)
SELECT sale_date, amount,
       AVG(amount) OVER (ORDER BY sale_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg
FROM sales;

-- Expected Output:
--  sale_date  | amount  | moving_avg
-- ------------+---------+------------
--  2026-01-01 | 100.00  | 100.00
--  2026-01-01 | 200.00  | 150.00
--  2026-01-02 | 150.00  | 150.00
--  2026-01-03 | 300.00  | 216.67
```

**Why this works:** The window includes the current row and the 2 physical rows before it. For the first row, only 1 row is available. For the last row, the window includes 150, 300, and the previous 200 (the 2 rows before the current row) .

**Example 2: 3-Day Moving Average with RANGE**

```sql
-- 3-day moving average using date range
SELECT sale_date, amount,
       AVG(amount) OVER (
           ORDER BY sale_date
           RANGE BETWEEN INTERVAL '3 days' PRECEDING AND CURRENT ROW
       ) AS moving_avg_3d
FROM sales;
```

**Why this works:** The window includes all rows whose `sale_date` falls within 3 days before the current row's date. Unlike `ROWS`, this includes peers (same-date rows) and rows at the value boundary .

### Real-World Cases

**Case 1: Stock Price Moving Average**

A financial analyst uses `ROWS BETWEEN 19 PRECEDING AND CURRENT ROW` to compute a 20-day moving average of closing prices.

**Case 2: Website Traffic Rolling Sum**

An SRE uses `RANGE BETWEEN INTERVAL '5 minutes' PRECEDING AND CURRENT ROW` to compute a rolling 5-minute request count for real-time monitoring.

### References

- InfluxData Documentation — Dynamic Offsets - https://test2.docs.influxdata.com/influxdb3/clustered/reference/sql/functions/window/
- PostgreSQL Documentation — Frame Boundaries - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/15/postgresql-15-A4.pdf

---

## 4. The Performance Trap: Default RANGE vs. ROWS

### Definitions

**Core Definition:** The default window frame `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` causes performance degradation compared to explicitly specifying `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` because it requires value comparisons and often materializes worktables to disk.

**Technical Definition:** When `ORDER BY` is present without an explicit frame, the default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. `RANGE` mode must compare values to determine peers, which prevents certain in-memory optimizations. `ROWS` mode counts physical positions and can often use an in-memory worktable, avoiding tempdb I/O and locking .

**Beginner-Friendly Explanation:** If you don't specify the frame, the database defaults to a mode that's slower. By explicitly saying `ROWS`, you tell the database to use a faster method that doesn't need to compare values.

### Purposes (Why It Matters)

- To understand why seemingly identical queries have different performance.
- To improve running total and ranking query performance by 5-20% .
- To avoid tempdb worktable creation and contention .
- To write predictable, optimized window queries.

### Syntax Rules and Structure

**Default vs. Explicit Frame:**

```sql
-- Default frame (slower): RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
SELECT SUM(amount) OVER (ORDER BY sale_date) AS running_total FROM sales;

-- Explicit ROWS frame (faster): ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
SELECT SUM(amount) OVER (ORDER BY sale_date ROWS UNBOUNDED PRECEDING) AS running_total FROM sales;
```

**Component Breakdown:**

| Frame | Worktable Location | Performance |
|-------|-------------------|-------------|
| `RANGE` (default) | tempdb (disk) | Slower  |
| `ROWS` | In-memory | Faster  |

**Syntax Rules:**

- The default frame is `RANGE UNBOUNDED PRECEDING`, equivalent to `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` .
- `ROWS UNBOUNDED PRECEDING` is the explicit `ROWS` equivalent .
- For `row_number()` and similar ranking functions, `ROWS` is always the correct semantic choice .

**Constraints and Limitations:**

- The performance difference is most pronounced with index-only scans (~20%) and less without (~5-10%) .
- SQL Server 2019+ may optimize `RANGE` frames in some cases, but `ROWS` remains consistently faster .
- The graphical execution plan may not reveal the difference; check `STATISTICS IO` for worktable reads .

### Annotated Complete Code Examples

**Example 1: Performance Comparison**

```sql
-- Query 1: Default RANGE frame (creates tempdb worktable)
SET STATISTICS IO ON;
SELECT CustomerID, SalesOrderID, TotalDue,
       SUM(TotalDue) OVER (PARTITION BY CustomerID ORDER BY SalesOrderID) AS RunningTotal
FROM Sales.SalesOrderHeader;

-- Query 2: Explicit ROWS frame (in-memory worktable)
SELECT CustomerID, SalesOrderID, TotalDue,
       SUM(TotalDue) OVER (PARTITION BY CustomerID ORDER BY SalesOrderID
                           ROWS UNBOUNDED PRECEDING) AS RunningTotal
FROM Sales.SalesOrderHeader;

-- Result: Query 2 has 0 worktable reads; Query 1 has high logical reads 
```

**Why this matters:** The `RANGE` frame forces SQL Server to create a worktable in tempdb to track peer groups. The `ROWS` frame uses an in-memory worktable, eliminating I/O and locking overhead. Tests on 15 million rows showed `ROWS` completed in a fraction of the time .

**Example 2: row_number() with ROWS**

```sql
-- More efficient: explicit ROWS for row_number()
SELECT customer_id, timestamp
FROM (
    SELECT row_number() OVER (PARTITION BY customer_id ORDER BY timestamp
                              ROWS UNBOUNDED PRECEDING) AS rn
    FROM customer_orders
) ranked
WHERE rn <= 3;
```

**Why this works:** `row_number()` operates on physical rows, so `ROWS` mode is semantically correct and avoids the overhead of `RANGE` peer comparison .

### Real-World Cases

**Case 1: Large-Scale Running Totals**

A financial system computes running balances on 30 million transaction rows. Switching from the default `RANGE` frame to explicit `ROWS` reduced execution time from minutes to seconds .

**Case 2: Top-N Per Group**

A query using `row_number()` to find the top 3 orders per customer ran 20% faster when `ROWS UNBOUNDED PRECEDING` was specified instead of relying on the default .

### References

- SQL Authority — T-SQL Window Function Framing and Performance - https://blog.sqlauthority.com/2015/11/19/sql-server-t-sql-window-function-framing-and-performance-notes-from-the-field-103/
- Stack Overflow — ROWS vs RANGE performance details - https://stackoverflow.com/posts/72824267/revisions
- PostgreSQL Mailing List — Default framing behavior - https://www.postgresql.org/message-id/e08cc0400812290954ud2650e0sd45f9e566e9fbcfb%40mail.gmail.com

---

## Summary Table: Frame Units and Performance

| Frame Unit | Boundary Basis | Offset Type | Performance | Peer Handling |
|------------|---------------|-------------|-------------|---------------|
| `ROWS` | Physical row position | Integer | Fastest (in-memory)  | Ignores peers |
| `RANGE` | Logical value distance | Numeric/Interval | Slower (tempdb)  | Includes all peers  |
| `GROUPS` | Peer group count | Integer | Moderate | Includes all peers  |
| Default (no frame) | `RANGE UNBOUNDED PRECEDING` | N/A | Slower | Includes peers  |

---

## References

- InfluxData Documentation — SQL Window Functions - https://test2.docs.influxdata.com/influxdb3/clustered/reference/sql/functions/window/
- Stack Overflow — Revisions to Find first 3 orders for each customer (ROWS vs RANGE performance) - https://stackoverflow.com/posts/72824267/revisions
- PostgreSQL Mailing List — Window function with partition by and order by (Default frame) - https://www.postgresql.org/message-id/CAAo1mb%3D%2BSrSf9h7B55yGOu%2BQ70Suq7oQ0BQ3QuPJmooOM9MkZA%40mail.gmail.com
- PostgreSQL Documentation — Window Functions (Default framing) - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/10/postgresql-10-US.pdf
- PostgreSQL Mailing List — Add GROUPS option to the Window Functions - https://www.postgresql.org/message-id/CAGMVOdtWkb9X7dUh7vjaCaiH34UGFg88unXYTEOub0Rk0swSXw%40mail.gmail.com
- InfluxData Documentation — SQL Window Functions (lag) - https://docs.influxdata.com/influxdb3/cloud-serverless/reference/sql/functions/window/#lag
- Stack Overflow — Revision (ROWS vs RANGE performance) - https://stackoverflow.com/revisions/72824267/3
- PostgreSQL Mailing List — TODO items for window functions (RANGE vs ROWS peers) - https://www.postgresql.org/message-id/e08cc0400812290954ud2650e0sd45f9e566e9fbcfb%40mail.gmail.com
- PostgreSQL Documentation — Row Constructors - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/15/postgresql-15-A4.pdf
- Microsoft Learn — T620, WINDOW clause: GROUPS option - https://learn.microsoft.com/en-ie/openspecs/sql_standards/ms-tsqliso02/474febd5-3748-4ac0-9621-a4238b93668f
- Broadcom Tanzu Greenplum Documentation — RANGE and GROUPS example - https://techdocs.broadcom.com/content/dam/broadcom/techdocs/us/en/pdf/vmware-tanzu/data-solutions/tanzu-greenplum/7-6/vmware-tanzu-greenplum-7-6.pdf
- Stack Overflow — Revision (ROWS vs RANGE performance) - https://stackoverflow.com/revisions/538e14e2-18b2-406e-ba89-4553b467401b/view-source
- PostgreSQL Mailing List — Windowing Function Patch Review (Default frame) - https://www.postgresql.org/message-id/e08cc0400811042047i92ea376t2843e90e2f5a50e2%40mail.gmail.com
- JSR — SisalSchemaObjectSnapshot - https://jsr.io/@sisal/core/doc/schema/~/SisalSchemaObjectSnapshot
- jOOQ Manual — Dialect support for window functions - http://www.jooq.org/doc/3.16/manual-pdf/jOOQ-manual-3.16.pdf
- GitHub — SQL Mastery Notes: Window Framing (ROWS vs. RANGE) - https://github.com/Niranjan-Kumar-Singh/sql-mastery-notes/blob/main/Phase_13_Window_Functions/08_Window_Framing.md
- SAP IQ Documentation — RANGE frame specification - https://help.sap.com/doc/a89b3a2984f210158cd1a1d90160c895/16.1.2.0/zh-CN/SAP_IQ_Administration_Database_zh.pdf
- GitHub — SQLAlchemy Issue #12450 (GROUPS support) - https://github.com/sqlalchemy/sqlalchemy/issues/12450
- NodeDB Documentation — Window Frames - https://github.com/NodeDB-Lab/nodedb-docs/blob/main/docs/sql/window-frames.rdx
- SQL Authority — T-SQL Window Function Framing and Performance - https://blog.sqlauthority.com/2015/11/19/sql-server-t-sql-window-function-framing-and-performance-notes-from-the-field-103/