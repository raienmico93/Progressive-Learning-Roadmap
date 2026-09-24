# SQL Value & Analytic Functions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Value and analytic functions are window functions that access values from other rows within the same result set without requiring a self-join, enabling comparisons across rows and cumulative calculations.

**Technical Definition:** These functions operate within the `OVER` clause, using `PARTITION BY` to define groups and `ORDER BY` to establish sequence. They fall into two categories: offset functions (`LAG`, `LEAD`) that retrieve values at a physical offset from the current row, and value functions (`FIRST_VALUE`, `LAST_VALUE`, `NTH_VALUE`) that retrieve values based on position or logical criteria. Unlike ranking functions, these functions return actual data values rather than rank numbers.

**Beginner-Friendly Explanation:** Imagine you have a list of daily sales. `LAG` lets you look at yesterday's sales, `LEAD` lets you peek at tomorrow's sales. `FIRST_VALUE` shows you the first sale in the period, and `LAST_VALUE` shows the last. These functions let you compare rows without writing complicated self-joins.

### Key Characteristics

- **Row-preserving:** All value and analytic functions return a value for every query row; they do not collapse rows like `GROUP BY`.
- **Offset-based access:** `LAG` and `LEAD` retrieve values at a fixed physical offset (previous or next row) .
- **Position-based access:** `FIRST_VALUE`, `LAST_VALUE`, and `NTH_VALUE` retrieve values based on position within the window frame .
- **Frame-dependent:** `FIRST_VALUE`, `LAST_VALUE`, and `NTH_VALUE` operate only on the rows within the current window frame .
- **Default frame caveat:** For `LAST_VALUE`, the default frame ends at the current row, giving potentially unhelpful results unless the frame is explicitly extended .

### Prerequisites

- **Window function fundamentals:** Understanding of `OVER`, `PARTITION BY`, `ORDER BY`, and frame clauses.
- **Aggregate awareness:** Familiarity with `SUM`, `AVG`, and cumulative calculations.
- **Set-based thinking:** Comfort with operating on sets of rows while retaining row detail.

### Related Programming Areas

- **Data Analysis & BI:** Period-over-period comparisons, running totals, moving averages.
- **Application Development:** Trend detection, delta calculations, leaderboards.
- **Data Engineering:** Cumulative metrics in ETL pipelines.
- **Reporting:** Year-over-year growth, rolling window statistics.

### Core Concepts / Features

1. `LAG`
2. `LEAD`
3. `FIRST_VALUE`
4. `LAST_VALUE`
5. `NTH_VALUE`
6. Common Analytic Patterns

---

## 1. LAG

### Definitions

**Core Definition:** `LAG` returns the value of an expression from a row at a specified physical offset *before* the current row within the partition.

**Technical Definition:** `LAG(value[, offset[, default]]) OVER ([partition_clause] order_by_clause)` returns the value evaluated at the row that is `offset` rows before the current row. If no such row exists, the `default` value is returned (or `NULL` if not specified). The default offset is 1 .

**Beginner-Friendly Explanation:** `LAG` lets you look at the previous row's value. It's like asking "What was the value one row ago?" — useful for comparing today vs. yesterday, or this month vs. last month.

### Purposes

- To compare the current row's value with the previous row's value.
- To calculate deltas (differences) between consecutive rows.
- To detect trends (increasing, decreasing, or stable).
- To compute period-over-period growth rates.
- To identify gaps or jumps in sequential data.

### Syntax Rules and Structure

```sql
LAG(value_expression [, offset [, default_value]])
OVER (
    [PARTITION BY partition_expression]
    ORDER BY order_expression
)
```

**Component Breakdown:**

| Component | Purpose | Default |
|-----------|---------|---------|
| `value_expression` | The column or expression to retrieve | Required |
| `offset` | Number of rows before the current row | 1 |
| `default_value` | Value to return if no row exists at offset | NULL |
| `PARTITION BY` | Divides rows into groups | Optional |
| `ORDER BY` | Determines row sequence | Required |

**Syntax Rules:**

- `ORDER BY` is required to establish which row is "previous."
- `offset` must be a non-negative integer; it can be a column, constant, or expression .
- `default_value` is returned when the offset row doesn't exist (e.g., first row with offset 1) .
- `PARTITION BY` resets the offset calculation for each group.

**Constraints and Limitations:**

- `LAG` operates on physical row positions, not logical values. If ties exist in the `ORDER BY`, results may be non-deterministic .
- `LAG` returns `NULL` (or the default) for the first row(s) in each partition.
- The SQL standard's `IGNORE NULLS` option is not implemented in PostgreSQL (behavior is always `RESPECT NULLS`) .

### Annotated Complete Code Examples

**Example 1: Day-over-Day Sales Comparison**

```sql
-- Setup: Daily sales data
CREATE TABLE daily_sales (
    sale_date DATE,
    amount    NUMERIC(10, 2)
);

INSERT INTO daily_sales VALUES
    ('2026-01-01', 1000.00),
    ('2026-01-02', 1500.00),
    ('2026-01-03', 1200.00),
    ('2026-01-04', 1800.00);

-- Query: Each day's sales and the previous day's sales
SELECT sale_date,
       amount,
       LAG(amount) OVER (ORDER BY sale_date) AS previous_day_sales,
       amount - LAG(amount) OVER (ORDER BY sale_date) AS day_over_day_change
FROM daily_sales
ORDER BY sale_date;

-- Expected Output:
--  sale_date  | amount  | previous_day_sales | day_over_day_change
-- ------------+---------+--------------------+---------------------
--  2026-01-01 | 1000.00 | (null)             | (null)
--  2026-01-02 | 1500.00 | 1000.00            | 500.00
--  2026-01-03 | 1200.00 | 1500.00            | -300.00
--  2026-01-04 | 1800.00 | 1200.00            | 600.00
```

**Why this output occurs:** The first row has no previous row, so `LAG` returns `NULL`. For each subsequent row, `LAG(amount)` retrieves the previous day's amount, and the difference is computed. This reveals that sales increased by 500 on Jan 2, decreased by 300 on Jan 3, and increased by 600 on Jan 4.

**Example 2: LAG with Offset and Default**

```sql
-- Query: Compare each day with the day two days ago
SELECT sale_date,
       amount,
       LAG(amount, 2, 0) OVER (ORDER BY sale_date) AS two_days_ago,
       amount - LAG(amount, 2, 0) OVER (ORDER BY sale_date) AS change
FROM daily_sales
ORDER BY sale_date;

-- Expected Output:
--  sale_date  | amount  | two_days_ago | change
-- ------------+---------+--------------+--------
--  2026-01-01 | 1000.00 | 0            | 1000.00
--  2026-01-02 | 1500.00 | 0            | 1500.00
--  2026-01-03 | 1200.00 | 1000.00      | 200.00
--  2026-01-04 | 1800.00 | 1500.00      | 300.00
```

**Why this output occurs:** With `offset = 2`, `LAG` looks two rows back. The first two rows have no row two positions back, so the default value `0` is returned. For Jan 3, it retrieves Jan 1's amount (1000).

### Real-World Cases

**Case 1: Stock Price Analysis**

A financial analyst uses `LAG` to compute the daily price change: `price - LAG(price) OVER (ORDER BY trade_date) AS daily_change`. This identifies days with significant price movements.

**Case 2: Customer Churn Detection**

A subscription business uses `LAG` on `last_login_date` to find users who haven't logged in for 30 days, flagging them for retention campaigns.

### References

- Oracle Data Flow Operators — LAG Function - https://docs.oracle.com/en-us/iaas/data-integration/using/using-operators.htm#19
- Microsoft Learn — [SQL 2012] Funções LEAD e LAG - https://learn.microsoft.com/pt-br/archive/technet-wiki/10272.sql-2012-funcoes-lead-e-lag

---

## 2. LEAD

### Definitions

**Core Definition:** `LEAD` returns the value of an expression from a row at a specified physical offset *after* the current row within the partition.

**Technical Definition:** `LEAD(value[, offset[, default]]) OVER ([partition_clause] order_by_clause)` returns the value evaluated at the row that is `offset` rows after the current row. If no such row exists, the `default` value is returned (or `NULL` if not specified). The default offset is 1 .

**Beginner-Friendly Explanation:** `LEAD` lets you look at the next row's value. It's like asking "What will the value be one row ahead?" — useful for forecasting or comparing today with tomorrow.

### Purposes

- To compare the current row's value with the next row's value.
- To compute forward-looking deltas.
- To detect upcoming changes or transitions.
- To calculate time-to-next-event.
- To support gap analysis (distance to the next occurrence).

### Syntax Rules and Structure

```sql
LEAD(value_expression [, offset [, default_value]])
OVER (
    [PARTITION BY partition_expression]
    ORDER BY order_expression
)
```

**Component Breakdown:**

| Component | Purpose | Default |
|-----------|---------|---------|
| `value_expression` | The column or expression to retrieve | Required |
| `offset` | Number of rows after the current row | 1 |
| `default_value` | Value to return if no row exists at offset | NULL |
| `PARTITION BY` | Divides rows into groups | Optional |
| `ORDER BY` | Determines row sequence | Required |

**Syntax Rules:**

- `LEAD` is the mirror image of `LAG`: it looks forward instead of backward .
- `ORDER BY` is required to establish which row is "next."
- `offset` must be a non-negative integer.
- `default_value` is returned when the offset row doesn't exist (e.g., last row with offset 1).

**Constraints and Limitations:**

- Same physical-position caveats as `LAG`: ties in `ORDER BY` may produce non-deterministic results.
- `LEAD` returns `NULL` (or the default) for the last row(s) in each partition.
- Cannot reference a column that isn't part of the window's `ORDER BY` or partition scope.

### Annotated Complete Code Examples

**Example 1: Time to Next Order**

```sql
-- Setup: Customer orders
CREATE TABLE orders (
    customer_id INTEGER,
    order_date  DATE,
    amount      NUMERIC(10, 2)
);

INSERT INTO orders VALUES
    (1, '2026-01-01', 100.00),
    (1, '2026-01-15', 200.00),
    (1, '2026-02-01', 150.00),
    (2, '2026-01-05', 300.00),
    (2, '2026-01-20', 250.00);

-- Query: Each order and the date of the next order for the same customer
SELECT customer_id,
       order_date,
       amount,
       LEAD(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS next_order_date,
       LEAD(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) - order_date AS days_to_next
FROM orders
ORDER BY customer_id, order_date;

-- Expected Output:
--  customer_id | order_date | amount  | next_order_date | days_to_next
-- -------------+------------+---------+-----------------+--------------
--            1 | 2026-01-01 | 100.00  | 2026-01-15      | 14
--            1 | 2026-01-15 | 200.00  | 2026-02-01      | 17
--            1 | 2026-02-01 | 150.00  | (null)          | (null)
--            2 | 2026-01-05 | 300.00  | 2026-01-20      | 15
--            2 | 2026-01-20 | 250.00  | (null)          | (null)
```

**Why this output occurs:** `PARTITION BY customer_id` ensures each customer's orders are processed separately. `LEAD(order_date)` retrieves the next order's date for the same customer. The last order for each customer has no next row, so `LEAD` returns `NULL`.

### Real-World Cases

**Case 1: Manufacturing Quality Control**

A factory uses `LEAD` on sensor readings to compare the current measurement with the next one, detecting drift or anomalies in real-time.

**Case 2: User Journey Analysis**

An analytics team uses `LEAD` on event timestamps to measure time between page views, identifying friction points in the user experience.

### References

- Microsoft Learn — [SQL 2012] Funções LEAD e LAG - https://learn.microsoft.com/pt-br/archive/technet-wiki/10272.sql-2012-funcoes-lead-e-lag
- Oracle Data Flow Operators — LAG/LEAD - https://docs.oracle.com/en-us/iaas/data-integration/using/using-operators.htm#19

---

## 3. FIRST_VALUE

### Definitions

**Core Definition:** `FIRST_VALUE` returns the value of an expression from the first row within the current window frame.

**Technical Definition:** `FIRST_VALUE(expr) OVER (window_spec)` returns the value evaluated at the first row of the window frame. The frame is defined by the `OVER` clause; if no frame is specified, the default frame spans from the partition start to the current row's last peer .

**Beginner-Friendly Explanation:** `FIRST_VALUE` tells you what the value was at the very beginning of the group or period. It's like asking "What was the starting value?"

### Purposes

- To establish a baseline for comparison (e.g., first day's sales, opening balance).
- To calculate growth relative to the starting point.
- To identify the earliest event in a sequence.
- To compute percentage change from the first value.

### Syntax Rules and Structure

```sql
FIRST_VALUE(value_expression)
OVER (
    [PARTITION BY partition_expression]
    ORDER BY order_expression
    [ROWS | RANGE frame_specification]
)
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `value_expression` | The column to retrieve from the first row |
| `PARTITION BY` | Divides rows into groups |
| `ORDER BY` | Determines row sequence |
| `frame_specification` | Limits which rows are considered (optional) |

**Syntax Rules:**

- `FIRST_VALUE` operates on the first row of the window frame, not necessarily the first row of the partition .
- Without an explicit frame, the default frame starts at the partition beginning .
- The frame can be extended to `UNBOUNDED FOLLOWING` to ensure the first row is always the partition's first row.

**Constraints and Limitations:**

- If the frame starts at `CURRENT ROW` (instead of `UNBOUNDED PRECEDING`), `FIRST_VALUE` returns the current row's value .
- `NULL` handling follows the standard's `RESPECT NULLS` behavior in PostgreSQL .

### Annotated Complete Code Examples

**Example 1: Sales Growth from First Day**

```sql
-- Query: Each day's sales compared to the first day's sales
SELECT sale_date,
       amount,
       FIRST_VALUE(amount) OVER (ORDER BY sale_date) AS first_day_sales,
       amount - FIRST_VALUE(amount) OVER (ORDER BY sale_date) AS growth_from_first
FROM daily_sales
ORDER BY sale_date;

-- Expected Output:
--  sale_date  | amount  | first_day_sales | growth_from_first
-- ------------+---------+-----------------+-------------------
--  2026-01-01 | 1000.00 | 1000.00         | 0.00
--  2026-01-02 | 1500.00 | 1000.00         | 500.00
--  2026-01-03 | 1200.00 | 1000.00         | 200.00
--  2026-01-04 | 1800.00 | 1000.00         | 800.00
```

**Why this output occurs:** `FIRST_VALUE(amount)` retrieves the first row's amount (1000) for every row. The difference shows how much each day has grown compared to the first day.

### Real-World Cases

**Case 1: Investment Performance**

A portfolio manager uses `FIRST_VALUE` to compare each day's portfolio value to the starting value, calculating cumulative return.

**Case 2: Patient Vital Signs**

A hospital system uses `FIRST_VALUE` on admission vitals to compare subsequent readings against the baseline.

### References

- PostgreSQL Documentation — Window Functions (first_value) - https://www.postgresql.org/docs/18/functions-window.html
- Oracle Data Warehousing Guide — FIRST_VALUE and LAST_VALUE - https://docs.oracle.com/cd/A84050_01/NT816CLI/DOC/server.816/a76994/analysis.htm

---

## 4. LAST_VALUE

### Definitions

**Core Definition:** `LAST_VALUE` returns the value of an expression from the last row within the current window frame.

**Technical Definition:** `LAST_VALUE(expr) OVER (window_spec)` returns the value evaluated at the last row of the window frame. Critically, the default frame for `LAST_VALUE` ends at the current row (plus peers), not at the partition end. This means `LAST_VALUE` without an explicit frame returns the *current* row's value, which is often unhelpful .

**Beginner-Friendly Explanation:** `LAST_VALUE` is supposed to tell you the ending value, but by default it only looks up to the current row. To get the actual last value of the partition, you must explicitly tell it to look at all rows.

### Purposes

- To identify the most recent or ending value in a sequence.
- To compare the current row against the final value.
- To calculate the range from current to end.
- To detect the last event in a group.

### Syntax Rules and Structure

```sql
LAST_VALUE(value_expression)
OVER (
    [PARTITION BY partition_expression]
    ORDER BY order_expression
    [ROWS | RANGE frame_specification]
)
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `value_expression` | The column to retrieve from the last row |
| `frame_specification` | **Must be explicitly set** to `UNBOUNDED FOLLOWING` for useful results |

**Syntax Rules:**

- The default frame for `LAST_VALUE` is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, which means the "last row" is the current row .
- To get the partition's actual last value, use `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` .
- Without the frame extension, `LAST_VALUE` often returns the same value as the current row.

**Constraints and Limitations:**

- The default behavior is a common source of confusion; always specify the frame explicitly when using `LAST_VALUE` .
- `NULL` handling follows `RESPECT NULLS` in PostgreSQL .

### Annotated Complete Code Examples

**Example 1: Default Frame Pitfall vs. Correct Usage**

```sql
-- Default frame: LAST_VALUE returns current row's value
SELECT sale_date,
       amount,
       LAST_VALUE(amount) OVER (ORDER BY sale_date) AS default_last_value
FROM daily_sales
ORDER BY sale_date;

-- Expected Output (default frame):
--  sale_date  | amount  | default_last_value
-- ------------+---------+--------------------
--  2026-01-01 | 1000.00 | 1000.00
--  2026-01-02 | 1500.00 | 1500.00
--  2026-01-03 | 1200.00 | 1200.00
--  2026-01-04 | 1800.00 | 1800.00

-- Correct frame: LAST_VALUE returns partition's last value
SELECT sale_date,
       amount,
       LAST_VALUE(amount) OVER (
           ORDER BY sale_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS actual_last_value
FROM daily_sales
ORDER BY sale_date;

-- Expected Output (correct frame):
--  sale_date  | amount  | actual_last_value
-- ------------+---------+-------------------
--  2026-01-01 | 1000.00 | 1800.00
--  2026-01-02 | 1500.00 | 1800.00
--  2026-01-03 | 1200.00 | 1800.00
--  2026-01-04 | 1800.00 | 1800.00
```

**Why this matters:** Without the frame extension, `LAST_VALUE` is useless for getting the partition's final value—it just echoes the current row. The explicit `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` tells it to consider all rows in the partition .

### Real-World Cases

**Case 1: Balance Reconciliation**

A banking query uses `LAST_VALUE` with an explicit frame to get the final balance for each account, then compares all transactions against that closing balance.

**Case 2: Project Status**

A project management tool uses `LAST_VALUE` on status updates to show the current status on every historical row.

### References

- PostgreSQL Documentation — Window Functions (last_value frame caveat) - https://www.postgresql.org/docs/18/functions-window.html
- Postgres Pro Standard — Window Functions - https://postgrespro.com/docs/postgrespro/17/functions-window.html

---

## 5. NTH_VALUE

### Definitions

**Core Definition:** `NTH_VALUE` returns the value of an expression from the *n*-th row within the current window frame.

**Technical Definition:** `NTH_VALUE(expr, n) OVER (window_spec)` returns the value evaluated at the *n*-th row of the window frame. If no such row exists, the return value is `NULL`. The `FROM FIRST` / `FROM LAST` option determines whether counting starts from the beginning or end of the frame (default is `FROM FIRST`) .

**Beginner-Friendly Explanation:** `NTH_VALUE` is like `FIRST_VALUE` and `LAST_VALUE` combined with a customizable position. You can ask "What was the 3rd value?" or "What was the 2nd-to-last value?"

### Purposes

- To retrieve a specific positioned value within a window (e.g., second-highest salary).
- To compare against a benchmark at a specific rank.
- To support "N-th element" analysis.
- To implement percentile-like logic without full percentile functions.

### Syntax Rules and Structure

```sql
NTH_VALUE(value_expression, n)
[FROM FIRST | FROM LAST]
[RESPECT NULLS | IGNORE NULLS]
OVER (
    [PARTITION BY partition_expression]
    ORDER BY order_expression
    [frame_specification]
)
```

**Component Breakdown:**

| Component | Purpose | Default |
|-----------|---------|---------|
| `value_expression` | The column to retrieve | Required |
| `n` | The position (1-based) | Required |
| `FROM FIRST` | Count from the start | Default |
| `FROM LAST` | Count from the end | Not implemented in PostgreSQL; MySQL parses but errors  |
| `RESPECT NULLS` | Include NULLs in counting | Default |
| `IGNORE NULLS` | Skip NULLs in counting | Oracle, not PostgreSQL  |

**Syntax Rules:**

- `n` must be a literal positive integer .
- `FROM LAST` is parsed but produces an error in MySQL; in PostgreSQL, only `FROM FIRST` is implemented .
- `IGNORE NULLS` is available in Oracle but not PostgreSQL .
- The frame determines which rows are eligible for the *n*-th position .

**Constraints and Limitations:**

- `FROM LAST` is not universally supported; reverse the `ORDER BY` to simulate it .
- `IGNORE NULLS` is not implemented in PostgreSQL .
- `NTH_VALUE` considers only rows within the window frame, not the entire partition, unless the frame is extended .

### Annotated Complete Code Examples

**Example 1: Second Sale Value**

```sql
-- Query: Each row's amount and the second row's amount
SELECT sale_date,
       amount,
       NTH_VALUE(amount, 2) OVER (
           ORDER BY sale_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS second_sale
FROM daily_sales
ORDER BY sale_date;

-- Expected Output:
--  sale_date  | amount  | second_sale
-- ------------+---------+-------------
--  2026-01-01 | 1000.00 | 1500.00
--  2026-01-02 | 1500.00 | 1500.00
--  2026-01-03 | 1200.00 | 1500.00
--  2026-01-04 | 1800.00 | 1500.00
```

**Why this output occurs:** `NTH_VALUE(amount, 2)` retrieves the second row's amount (1500) from the extended frame. Every row sees the same second value because the frame includes all rows.

**Example 2: NTH_VALUE with PARTITION BY**

```sql
-- Query: Second-highest salary per department
SELECT department, employee_name, salary,
       NTH_VALUE(salary, 2) OVER (
           PARTITION BY department
           ORDER BY salary DESC
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS second_highest
FROM employees;
```

**Why this works:** `PARTITION BY department` creates separate windows per department. `ORDER BY salary DESC` ranks highest first. `NTH_VALUE(salary, 2)` retrieves the second row's salary (second-highest) for each department.

### Real-World Cases

**Case 1: Second Best Performance**

A sales manager uses `NTH_VALUE` to find the second-best salesperson in each region, rewarding them separately from the top performer.

**Case 2: Benchmark Comparison**

An analyst uses `NTH_VALUE(revenue, 5)` to compare every product's revenue against the 5th-best product, identifying the top tier.

### References

- Oracle Data Warehousing Guide — NTH_VALUE - https://docs.oracle.com/en/database/oracle/oracle-database/21/dwhsg/sql-analysis-reporting-data-warehouses.html
- MySQL 8.0 Reference Manual — Window Function Descriptions (NTH_VALUE) - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/window-function-descriptions.html
- PostgreSQL Documentation — Window Functions (nth_value) - https://www.postgresql.org/docs/18/functions-window.html

---

## 6. Common Analytic Patterns

### Definitions

**Core Definition:** Common analytic patterns are recurring query structures that use value and analytic functions to solve standard business problems: running totals, moving averages, and year-to-date calculations.

**Technical Definition:** These patterns leverage window functions with `ORDER BY` and frame specifications to compute cumulative and rolling statistics. Running totals use `SUM` with `UNBOUNDED PRECEDING AND CURRENT ROW`; moving averages use `AVG` with a fixed `n PRECEDING` frame; YTD calculations use `SUM` with date-based `RANGE` frames.

**Beginner-Friendly Explanation:** These are recipes for common calculations: "total so far" (running total), "average of the last 7 days" (moving average), and "total since the start of the year" (YTD).

### Purposes

- To compute cumulative totals (running totals) for financial and operational reporting.
- To smooth data using moving averages for trend analysis.
- To calculate period-to-date metrics (YTD, MTD, QTD).
- To support comparative analysis across time periods.

### Syntax Rules and Structure

**Running Total:**

```sql
SUM(value) OVER (
    [PARTITION BY partition_expression]
    ORDER BY order_expression
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

**Moving Average (n-row):**

```sql
AVG(value) OVER (
    [PARTITION BY partition_expression]
    ORDER BY order_expression
    ROWS BETWEEN n-1 PRECEDING AND CURRENT ROW
)
```

**Year-to-Date (Date-based):**

```sql
SUM(value) OVER (
    PARTITION BY EXTRACT(YEAR FROM date_column)
    ORDER BY date_column
    RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

**Component Breakdown:**

| Pattern | Frame | Function |
|---------|-------|----------|
| Running Total | `UNBOUNDED PRECEDING AND CURRENT ROW` | `SUM` |
| Moving Average | `n PRECEDING AND CURRENT ROW` | `AVG` |
| YTD | `UNBOUNDED PRECEDING AND CURRENT ROW` (within year partition) | `SUM` |

### Annotated Complete Code Examples

**Example 1: Running Total**

```sql
-- Query: Running total of sales
SELECT sale_date,
       amount,
       SUM(amount) OVER (ORDER BY sale_date) AS running_total
FROM daily_sales
ORDER BY sale_date;

-- Expected Output:
--  sale_date  | amount  | running_total
-- ------------+---------+---------------
--  2026-01-01 | 1000.00 | 1000.00
--  2026-01-02 | 1500.00 | 2500.00
--  2026-01-03 | 1200.00 | 3700.00
--  2026-01-04 | 1800.00 | 5500.00
```

**Why this output occurs:** With `ORDER BY sale_date` and no explicit frame, the default frame `RANGE UNBOUNDED PRECEDING AND CURRENT ROW` creates a cumulative sum. Each row's total includes all previous rows .

**Example 2: Moving Average (3-row)**

```sql
-- Query: 3-day moving average
SELECT sale_date,
       amount,
       AVG(amount) OVER (
           ORDER BY sale_date
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
       ) AS moving_avg_3
FROM daily_sales
ORDER BY sale_date;

-- Expected Output:
--  sale_date  | amount  | moving_avg_3
-- ------------+---------+--------------
--  2026-01-01 | 1000.00 | 1000.00
--  2026-01-02 | 1500.00 | 1250.00
--  2026-01-03 | 1200.00 | 1233.33
--  2026-01-04 | 1800.00 | 1500.00
```

**Why this output occurs:** `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW` defines a 3-row window. For Jan 1, only 1 row is available, so the average is 1000. For Jan 2, the window includes Jan 1 and Jan 2, averaging 1250. For Jan 3, all 3 rows are averaged, yielding 1233.33 .

**Example 3: Year-to-Date (YTD)**

```sql
-- Setup: Multi-year sales
CREATE TABLE yearly_sales (
    sale_date DATE,
    amount    NUMERIC(10, 2)
);

INSERT INTO yearly_sales VALUES
    ('2025-12-01', 500.00),
    ('2026-01-01', 1000.00),
    ('2026-01-02', 1500.00),
    ('2026-02-01', 1200.00),
    ('2026-03-01', 1800.00);

-- Query: Year-to-date cumulative sales
SELECT sale_date,
       amount,
       SUM(amount) OVER (
           PARTITION BY EXTRACT(YEAR FROM sale_date)
           ORDER BY sale_date
       ) AS ytd_total
FROM yearly_sales
ORDER BY sale_date;

-- Expected Output:
--  sale_date  | amount  | ytd_total
-- ------------+---------+-----------
--  2025-12-01 | 500.00  | 500.00
--  2026-01-01 | 1000.00 | 1000.00
--  2026-01-02 | 1500.00 | 2500.00
--  2026-02-01 | 1200.00 | 3700.00
--  2026-03-01 | 1800.00 | 5500.00
```

**Why this output occurs:** `PARTITION BY EXTRACT(YEAR FROM sale_date)` resets the cumulative sum at each year boundary. The 2025 row accumulates to 500. The 2026 rows start fresh at 1000 and accumulate to 5500 by March .

### Real-World Cases

**Case 1: Financial Reporting**

A CFO uses YTD calculations to track revenue against annual targets, comparing cumulative performance month by month.

**Case 2: Operations Monitoring**

An SRE uses 7-day moving averages of response times to detect gradual performance degradation that daily snapshots might miss.

### References

- PostgreSQL Window Functions Guide — Running Totals and Moving Averages - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-25-postgresql-window-functions/README.md
- Microsoft Learn — Apply Window Functions for Analytics - https://learn.microsoft.com/et-ee/training/modules/write-advanced-sql-code/3-window-functions
- Crunchy Data — Window Functions for Data Analysis with Postgres - https://www.crunchydata.com/blog/window-functions-for-data-analysis-with-postgres

---

## Summary Table of Value & Analytic Functions

| Function | Purpose | Key Detail |
|----------|---------|------------|
| `LAG` | Previous row's value | Default offset 1; returns NULL/default at partition start  |
| `LEAD` | Next row's value | Default offset 1; returns NULL/default at partition end  |
| `FIRST_VALUE` | First row's value | Depends on frame; default frame starts at partition beginning  |
| `LAST_VALUE` | Last row's value | **Must extend frame** to `UNBOUNDED FOLLOWING` for useful results  |
| `NTH_VALUE` | *n*-th row's value | `n` must be literal; `FROM LAST` limited support  |

---

## References

- Oracle Data Flow Operators — LAG Function - https://docs.oracle.com/en-us/iaas/data-integration/using/using-operators.htm#19
- Microsoft Learn — [SQL 2012] Funções LEAD e LAG - https://learn.microsoft.com/pt-br/archive/technet-wiki/10272.sql-2012-funcoes-lead-e-lag
- PostgreSQL Documentation — Window Functions (first_value, last_value, nth_value) - https://www.postgresql.org/docs/18/functions-window.html
- Postgres Pro Standard — Window Functions - https://postgrespro.com/docs/postgrespro/17/functions-window.html
- Oracle Data Warehousing Guide — NTH_VALUE - https://docs.oracle.com/en/database/oracle/oracle-database/21/dwhsg/sql-analysis-reporting-data-warehouses.html
- MySQL 8.0 Reference Manual — Window Function Descriptions - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/window-function-descriptions.html
- Microsoft Learn — Apply Window Functions for Analytics - https://learn.microsoft.com/et-ee/training/modules/write-advanced-sql-code/3-window-functions
- PostgreSQL Window Functions Guide — Running Totals and Moving Averages - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-25-postgresql-window-functions/README.md
- Crunchy Data — Window Functions for Data Analysis with Postgres - https://www.crunchydata.com/blog/window-functions-for-data-analysis-with-postgres
- Oracle Data Warehousing Guide — FIRST_VALUE and LAST_VALUE - https://docs.oracle.com/cd/A84050_01/NT816CLI/DOC/server.816/a76994/analysis.htm
- Microsoft Learn — OVER Clause (Transact-SQL) - https://learn.microsoft.com/de-ch/sql/t-sql/queries/select-over-clause-transact-sql
- MySQL 8.0 Reference Manual — Window Function Frame Specification - https://docs.oracle.com/cd/E17952%5F01/mysql-8.0-en/window-functions-frames.html