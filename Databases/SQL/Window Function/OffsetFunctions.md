# Comprehensive Programming Cheat Sheet: Offset Functions

## Topic Overview

### Definitions

**Core Definition**  
Offset functions are a category of window functions that retrieve a value from a row at a specified physical or logical offset relative to the current row within its window partition.

**Technical Definition**  
Offset functions, also called positional or navigation functions, are window functions that access data from a row other than the current row without requiring a self-join. The standard offset functions are `LAG`, `LEAD`, `FIRST_VALUE`, `LAST_VALUE`, and `NTH_VALUE`. They are defined in the SQL standard (SQL:2003 and later) and are supported by PostgreSQL, MySQL 8.0+, SQL Server 2012+, Oracle, and other modern RDBMSs. `LAG` and `LEAD` operate on physical row offsets and do not respect the window frame; `FIRST_VALUE`, `LAST_VALUE`, and `NTH_VALUE` operate on the current window frame and therefore are sensitive to frame specification.

**Beginner-Friendly Explanation**  
Offset functions let you look at a different row in the same group without writing a complicated self-join. For example, `LAG` shows you the previous row's value, `LEAD` shows you the next row's value, and `FIRST_VALUE` shows you the very first row's value in the group. They are extremely useful for comparing a value to what came before it or after it, such as calculating month-over-month growth or finding the first and last event in a session.

### Key Characteristics

- **Row-preserving**: Offset functions return one value per input row, like all window functions; they do not collapse rows.
- **`ORDER BY` required**: All offset functions require an `ORDER BY` clause in the window specification to define the meaning of "previous," "next," "first," and "last".
- **`LAG` and `LEAD` ignore the frame**: These functions operate on physical row offsets and are not affected by `ROWS`, `RANGE`, or `GROUPS` frame clauses.
- **`FIRST_VALUE`, `LAST_VALUE`, and `NTH_VALUE` respect the frame**: These functions return a value from the first, last, or N-th row of the current window frame, which by default spans from the start of the partition through the current row's last peer. This default behavior often produces unhelpful results for `LAST_VALUE` and `NTH_VALUE` unless an explicit frame clause is added.
- **Default values**: `LAG` and `LEAD` accept an optional third parameter to return a substitute value when the offset goes beyond the partition boundary, avoiding `NULL` results.
- **Partition-resetting**: Offsets reset at partition boundaries; without `PARTITION BY`, the entire result set is one partition.
- **NULL treatment**: The SQL standard defines `RESPECT NULLS` and `IGNORE NULLS` options for these functions, but support varies by engine. PostgreSQL implements only `RESPECT NULLS` (the default); MySQL parses `IGNORE NULLS` but raises an error; SQL Server 2022+ supports both options.

### Prerequisites

- Understanding of window function fundamentals, including the `OVER` clause, `PARTITION BY`, and `ORDER BY`.
- Familiarity with window frames (`ROWS`, `RANGE`, `GROUPS`) and the concept of the default frame.
- Knowledge of `SELECT`, `WHERE`, and basic aggregate functions.
- Awareness of the target engine's support for offset functions and their optional parameters.

### Related Programming Areas

- Time-series analysis and period-over-period reporting.
- Sessionization and gap detection in event data.
- Running totals and moving averages.
- Churn analysis and customer behavior tracking.
- Data quality checks for sequential integrity.

---

## Core Concepts and Features

### 1. `LAG()`

#### Definitions

**Core Definition**  
`LAG()` returns the value of an expression evaluated on a row that is a specified number of rows **before** the current row within its partition.

**Technical Definition**  
`LAG(scalar_expression [, offset [, default]])` returns the value of `scalar_expression` from the row that is `offset` rows before the current row in the window’s `ORDER BY` sequence. If no such row exists, it returns `default` (or `NULL` if `default` is omitted). The `offset` defaults to 1. `LAG` does not respect the window frame; it always operates on physical row positions. Supported by PostgreSQL, MySQL 8.0+, SQL Server 2012+, Oracle, and SQLite 3.25+.

**Beginner-Friendly Explanation**  
`LAG()` shows you the value from the **previous** row. If you have daily sales, `LAG(sales)` gives you yesterday's sales on today's row.

#### Purposes

- To compare the current row's value with the immediately preceding row.
- To calculate period-over-period changes (e.g., day-over-day, month-over-month).
- To detect gaps or discontinuities in sequential data.
- To access prior values without a self-join.

#### Syntax Rules and Structure

**General syntax**:

```sql
LAG(scalar_expression [, offset [, default]]) OVER (
    [ PARTITION BY expression [, ...] ]
    ORDER BY expression [ ASC | DESC ] [, ...]
)
```

**PostgreSQL** (official) :

```sql
lag(value anyelement [, offset integer [, default anyelement ]]) OVER (...)
```

**SQL Server** (official) :

```sql
LAG ( scalar_expression [ , offset ] , [ default ] )
    [ IGNORE NULLS | RESPECT NULLS ]
    OVER ( [ partition_by_clause ] order_by_clause )
```

**MySQL 8.0+** (official) :

```sql
LAG(expr [, N [, default]]) OVER (...)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `scalar_expression` / `expr` / `value` | The column or expression whose value is retrieved. |
| `offset` | Number of rows back to look (default `1`). Must be a non-negative integer. |
| `default` | Value returned if the offset goes beyond the partition boundary (default `NULL`). Must be type-compatible with `scalar_expression`. |
| `OVER` | Mandatory clause. |
| `PARTITION BY` | Optional; resets the offset per partition. |
| `ORDER BY` | Required; defines row sequence. |

**Syntax rules**

- `offset` must be a non-negative integer; negative values are not allowed.
- `offset` cannot be a window function or subquery in most engines.
- `default` must be compatible with the type of `scalar_expression`.
- `LAG` ignores the window frame; it operates on physical row positions.
- `ORDER BY` is required to define a meaningful "previous" row.

**Constraints and limitations**

- Without `PARTITION BY`, the offset crosses the entire result set, which may cause the first row of one entity to compare against the last row of another.
- Without `ORDER BY`, the concept of "previous" is undefined and results are non-deterministic.
- `IGNORE NULLS` support is engine-specific: PostgreSQL does not implement it; MySQL parses but rejects it.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Month-over-month sales growth

```sql
WITH monthly_sales AS (
    SELECT DATE_TRUNC('month', sale_date) AS month, SUM(amount) AS revenue
    FROM sales
    GROUP BY DATE_TRUNC('month', sale_date)
)
SELECT month, revenue,
       LAG(revenue) OVER (ORDER BY month) AS prev_month_revenue,
       revenue - LAG(revenue) OVER (ORDER BY month) AS mom_change
FROM monthly_sales
ORDER BY month;
```

**Expected output** (illustrative):

```text
   month    | revenue | prev_month_revenue | mom_change
------------+---------+--------------------+------------
 2025-01-01 |   10000 |                    |           
 2025-02-01 |   12000 |              10000 |       2000
 2025-03-01 |   11000 |              12000 |      -1000
 2025-04-01 |   15000 |              11000 |       4000
(4 rows)
```

**Why this output appears**: `LAG(revenue)` retrieves the previous month's revenue; the first row has no previous month, so `prev_month_revenue` is `NULL` and `mom_change` is `NULL`. The subsequent rows compute the difference from the prior month.

##### Example 2: SQL Server – Using a default value

```sql
SELECT ProductID, OrderDate, TotalDue,
       LAG(TotalDue, 1, 0) OVER (PARTITION BY ProductID ORDER BY OrderDate) AS PrevTotal
FROM Sales.SalesOrderDetail;
```

**Expected output** (illustrative):

```text
ProductID OrderDate   TotalDue PrevTotal
--------- ----------- -------- ---------
1         2025-01-01  100.00   0.00
1         2025-02-01  150.00   100.00
1         2025-03-01  200.00   150.00
2         2025-01-15   50.00   0.00
2         2025-02-15   50.00   50.00
```

**Why this output appears**: `LAG(TotalDue, 1, 0)` looks one row back; when there is no previous row (the first order per product), it returns the default value `0` instead of `NULL`.

##### Example 3: MySQL – Detecting session gaps

```sql
SELECT user_id, event_time,
       LAG(event_time) OVER (PARTITION BY user_id ORDER BY event_time) AS prev_event,
       TIMESTAMPDIFF(SECOND, LAG(event_time) OVER (PARTITION BY user_id ORDER BY event_time), event_time) AS gap_seconds
FROM user_events;
```

**Expected output** (illustrative):

```text
+---------+---------------------+---------------------+-------------+
| user_id | event_time          | prev_event          | gap_seconds |
+---------+---------------------+---------------------+-------------+
| 1       | 2025-01-01 10:00:00 | NULL                |        NULL |
| 1       | 2025-01-01 10:05:00 | 2025-01-01 10:00:00 |         300 |
| 1       | 2025-01-01 11:30:00 | 2025-01-01 10:05:00 |        5100 |
| 2       | 2025-01-01 09:00:00 | NULL                |        NULL |
+---------+---------------------+---------------------+-------------+
```

**Why this output appears**: `LAG(event_time)` retrieves the previous event for the same user; `TIMESTAMPDIFF` computes the gap between events. The 5100-second gap (85 minutes) indicates a likely session break.

#### Real-World Cases

- **Financial reporting**: Month-over-month revenue growth, quarter-over-quarter comparisons.
- **Web analytics**: Sessionization by detecting inactivity gaps between events.
- **IoT telemetry**: Detecting equipment downtime by comparing consecutive sensor readings.
- **Inventory management**: Tracking stock level changes from the previous day.

---

### 2. `LEAD()`

#### Definitions

**Core Definition**  
`LEAD()` returns the value of an expression evaluated on a row that is a specified number of rows **after** the current row within its partition.

**Technical Definition**  
`LEAD(scalar_expression [, offset [, default]])` returns the value of `scalar_expression` from the row that is `offset` rows after the current row in the window’s `ORDER BY` sequence. If no such row exists, it returns `default` (or `NULL` if omitted). The `offset` defaults to 1. `LEAD` is the mirror image of `LAG` and does not respect the window frame. Supported by PostgreSQL, MySQL 8.0+, SQL Server 2012+, Oracle, and SQLite 3.25+.

**Beginner-Friendly Explanation**  
`LEAD()` shows you the value from the **next** row. If you have daily temperatures, `LEAD(temperature)` gives you tomorrow's temperature on today's row.

#### Purposes

- To compare the current row's value with the immediately following row.
- To calculate forward-looking changes and forecasts.
- To detect gaps by looking ahead (e.g., the next event's timestamp).
- To access subsequent values without a self-join.

#### Syntax Rules and Structure

**General syntax**:

```sql
LEAD(scalar_expression [, offset [, default]]) OVER (
    [ PARTITION BY expression [, ...] ]
    ORDER BY expression [ ASC | DESC ] [, ...]
)
```

**SQL Server** (official) :

```sql
LEAD ( scalar_expression [ , offset ] , [ default ] )
    [ IGNORE NULLS | RESPECT NULLS ]
    OVER ( [ partition_by_clause ] order_by_clause )
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `scalar_expression` | The column or expression whose value is retrieved. |
| `offset` | Number of rows forward to look (default `1`). |
| `default` | Value returned if the offset goes beyond the partition end (default `NULL`). |
| `OVER` | Mandatory clause. |
| `ORDER BY` | Required; defines row sequence. |

**Syntax rules**

- `offset` must be a non-negative integer.
- `default` must be type-compatible with `scalar_expression`.
- `LEAD` ignores the window frame.
- The last row of each partition returns `default` (or `NULL`).

**Constraints and limitations**

- Same partition and ordering requirements as `LAG`.
- Without `PARTITION BY`, the look-ahead crosses entity boundaries.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Next month's revenue for forecasting

```sql
WITH monthly_sales AS (
    SELECT DATE_TRUNC('month', sale_date) AS month, SUM(amount) AS revenue
    FROM sales
    GROUP BY DATE_TRUNC('month', sale_date)
)
SELECT month, revenue,
       LEAD(revenue) OVER (ORDER BY month) AS next_month_revenue
FROM monthly_sales
ORDER BY month;
```

**Expected output**:

```text
   month    | revenue | next_month_revenue
------------+---------+--------------------
 2025-01-01 |   10000 |              12000
 2025-02-01 |   12000 |              11000
 2025-03-01 |   11000 |              15000
 2025-04-01 |   15000 |                   
(4 rows)
```

**Why this output appears**: `LEAD(revenue)` retrieves the next month's revenue; the last row has no following month, so it returns `NULL`.

##### Example 2: SQL Server – Comparing current and next quota

```sql
SELECT EmployeeID, QuotaDate, SalesQuota,
       LEAD(SalesQuota, 1, 0) OVER (ORDER BY QuotaDate) AS NextQuota
FROM Sales.SalesPersonQuotaHistory;
```

**Expected output** (illustrative):

```text
EmployeeID QuotaDate   SalesQuota NextQuota
---------- ----------- ---------- ---------
1          2025-01-01  100000     120000
1          2025-04-01  120000     150000
1          2025-07-01  150000     0
```

**Why this output appears**: `LEAD` looks one row forward; the last row returns the default `0` because there is no next quota.

#### Real-World Cases

- **Forecasting**: Using the next period's known value for planning.
- **Event processing**: Checking whether the next event occurs within a timeout window.
- **Education analytics**: Comparing a student's score to the next assessment.
- **Logistics**: Determining the next scheduled delivery for a shipment.

---

### 3. `FIRST_VALUE()`

#### Definitions

**Core Definition**  
`FIRST_VALUE()` returns the value of an expression evaluated at the **first row** of the current window frame.

**Technical Definition**  
`FIRST_VALUE(expression)` returns the value of `expression` for the first row in the current window frame. Unlike `LAG` and `LEAD`, it respects the frame clause. With the default frame (`RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`), the first row is the first row of the partition. Supported by PostgreSQL, MySQL 8.0+, SQL Server 2012+, Oracle, and SQLite 3.25+.

**Beginner-Friendly Explanation**  
`FIRST_VALUE()` shows you the value from the **very first row** of the group (or the first row of the current frame if you specify one). It is like asking "what was the starting value?"

#### Purposes

- To retrieve the first value in a partition for comparison.
- To establish a baseline for cumulative calculations.
- To find the earliest event in a session or group.
- To carry forward an initial value across subsequent rows.

#### Syntax Rules and Structure

**General syntax**:

```sql
FIRST_VALUE(expression) OVER (
    [ PARTITION BY expression [, ...] ]
    ORDER BY expression [ ASC | DESC ] [, ...]
    [ frame_clause ]
)
```

**PostgreSQL** (official) :

```sql
first_value(value anyelement) OVER (...)
```

**Component breakdown

| Component | Description |
|-----------|-------------|
| `expression` | The column or expression whose value is retrieved. |
| `OVER` | Mandatory clause. |
| `PARTITION BY` | Optional; resets the "first" row per partition. |
| `ORDER BY` | Required to define which row is first. |
| `frame_clause` | Optional; defines the frame whose first row is used. |

**Syntax rules**

- `FIRST_VALUE` respects the window frame.
- Without `ORDER BY`, the "first" row is arbitrary.
- With the default frame, the first row is the partition's first row.
- `IGNORE NULLS` can skip `NULL` values where supported (Oracle, SQL Server 2022+; not PostgreSQL).

**Constraints and limitations**

- PostgreSQL does not implement `IGNORE NULLS` for this function.
- The default frame behavior may not be intuitive when `ORDER BY` is present; the first row is always the partition start under the default frame.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – First salary in each department

```sql
SELECT depname, empno, salary,
       FIRST_VALUE(salary) OVER (PARTITION BY depname ORDER BY salary) AS first_salary
FROM empsalary;
```

**Expected output**:

```text
 depname  | empno | salary | first_salary
----------+-------+--------+--------------
 develop  |     7 |   4200 |         4200
 develop  |     9 |   4500 |         4200
 develop  |    10 |   5200 |         4200
 develop  |    11 |   5200 |         4200
 develop  |     8 |   6000 |         4200
 personnel|     5 |   3500 |         3500
 personnel|     2 |   3900 |         3500
 sales    |     3 |   4800 |         4800
 sales    |     4 |   4800 |         4800
 sales    |     1 |   5000 |         4800
(10 rows)
```

**Why this output appears**: `FIRST_VALUE(salary)` returns the lowest salary in each department (the first row under `ORDER BY salary`), and that value is repeated on every row of the department.

##### Example 2: Oracle – First non-null value with `IGNORE NULLS`

```sql
SELECT prod_id, channel_id, quantity,
       FIRST_VALUE(quantity IGNORE NULLS) OVER (PARTITION BY prod_id ORDER BY channel_id) AS first_non_null
FROM sales_data;
```

**Expected output** (illustrative):

```text
PROD_ID CHANNEL_ID QUANTITY FIRST_NON_NULL
------- ---------- -------- --------------
18      1          100      100
18      2          NULL     100
18      3          200      100
```

**Why this output appears**: `IGNORE NULLS` skips the `NULL` quantity in channel 2; the first non-null value (100 from channel 1) is returned for all rows of the product.

#### Real-World Cases

- **Baseline comparison**: Comparing each month's revenue to January's revenue.
- **Session start time**: Attaching the first event time to all events in a session.
- **Initial inventory**: Showing the starting stock level alongside daily changes.

---

### 4. `LAST_VALUE()`

#### Definitions

**Core Definition**  
`LAST_VALUE()` returns the value of an expression evaluated at the **last row** of the current window frame.

**Technical Definition**  
`LAST_VALUE(expression)` returns the value of `expression` for the last row in the current window frame. Like `FIRST_VALUE`, it respects the frame clause. A critical caveat is that with the default frame (`RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`), the "last row" of the frame is the **current row itself** (or its last peer), not the last row of the partition. This often produces unhelpful results unless an explicit frame clause such as `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` is specified.

**Beginner-Friendly Explanation**  
`LAST_VALUE()` shows you the value from the **last row of the current window**. Be careful: by default, the "window" only goes up to the current row, so `LAST_VALUE` often just returns the current row's value. To get the actual last row of the whole group, you must explicitly tell it to look at the entire partition.

#### Purposes

- To retrieve the last value in a partition (with an explicit frame).
- To establish a closing value for comparison.
- To find the most recent event in a group.
- To compute totals or baselines at the end of a period.

#### Syntax Rules and Structure

**General syntax**:

```sql
LAST_VALUE(expression) OVER (
    [ PARTITION BY expression [, ...] ]
    ORDER BY expression [ ASC | DESC ] [, ...]
    [ frame_clause ]
)
```

**PostgreSQL** (official) :

```sql
last_value(value anyelement) OVER (...)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `expression` | The column or expression whose value is retrieved. |
| `OVER` | Mandatory clause. |
| `PARTITION BY` | Optional; resets the "last" row per partition. |
| `ORDER BY` | Required to define ordering. |
| `frame_clause` | **Critical**; must extend to the partition end to get the true last row. |

**Syntax rules**

- `LAST_VALUE` respects the frame.
- The default frame ends at the current row; use `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` to include the entire partition.
- `IGNORE NULLS` support varies by engine.

**Constraints and limitations**

- The default frame behavior is the most common pitfall with this function.
- Without an explicit frame, `LAST_VALUE` often returns the current row's value rather than the partition's last value.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Default frame pitfall vs explicit full-partition frame

```sql
-- Default frame: LAST_VALUE returns the current row's value.
SELECT salary,
       LAST_VALUE(salary) OVER (ORDER BY salary) AS default_last
FROM empsalary;

-- Explicit full-partition frame: LAST_VALUE returns the true last value.
SELECT salary,
       LAST_VALUE(salary) OVER (ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS true_last
FROM empsalary;
```

**Expected output** (default frame):

```text
 salary | default_last
--------+--------------
   3500 |         3500
   3900 |         3900
   4200 |         4200
   4500 |         4500
   4800 |         4800
   4800 |         4800
   5000 |         5000
   5200 |         5200
   5200 |         5200
   6000 |         6000
(10 rows)
```

**Expected output** (explicit frame):

```text
 salary | true_last
--------+-----------
   3500 |      6000
   3900 |      6000
   4200 |      6000
   4500 |      6000
   4800 |      6000
   4800 |      6000
   5000 |      6000
   5200 |      6000
   5200 |      6000
   6000 |      6000
(10 rows)
```

**Why this output appears**: With the default frame, the "last row" of the frame for each row is the current row itself, so `LAST_VALUE` returns the current salary. With the explicit full-partition frame, the last row is always 6000, so `LAST_VALUE` returns 6000 for every row.

##### Example 2: MySQL – True last value with explicit frame

```sql
SELECT depname, empno, salary,
       LAST_VALUE(salary) OVER (
           PARTITION BY depname
           ORDER BY salary
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS highest_salary
FROM empsalary;
```

**Expected output**:

```text
+----------+-------+--------+---------------+
| depname  | empno | salary | highest_salary|
+----------+-------+--------+---------------+
| develop  |     7 |   4200 |          6000 |
| develop  |     9 |   4500 |          6000 |
| develop  |    10 |   5200 |          6000 |
| develop  |    11 |   5200 |          6000 |
| develop  |     8 |   6000 |          6000 |
| personnel|     5 |   3500 |          3900 |
| personnel|     2 |   3900 |          3900 |
| sales    |     3 |   4800 |          5000 |
| sales    |     4 |   4800 |          5000 |
| sales    |     1 |   5000 |          5000 |
+----------+-------+--------+---------------+
```

**Why this output appears**: The explicit frame extends to the end of each partition; `LAST_VALUE` returns the highest salary per department (the last row under `ORDER BY salary`).

#### Real-World Cases

- **Closing balance**: Showing the final balance of a period alongside each transaction.
- **Most recent event**: Attaching the latest event time to all events in a session.
- **Target vs actual**: Comparing each period's value to the final period's value.

---

### 5. `NTH_VALUE()`

#### Definitions

**Core Definition**  
`NTH_VALUE()` returns the value of an expression evaluated at the **N-th row** of the current window frame.

**Technical Definition**  
`NTH_VALUE(expression, N)` returns the value of `expression` for the N-th row of the window frame, where N is a positive integer. The function respects the frame clause. With the default frame, the N-th row is counted from the start of the partition through the current row's last peer; if N exceeds the number of rows in the frame, the function returns `NULL`. Supported by PostgreSQL, MySQL 8.0+, SQL Server 2012+, Oracle, and SQLite 3.25+. PostgreSQL supports only `FROM FIRST` (the default); `FROM LAST` is not implemented.

**Beginner-Friendly Explanation**  
`NTH_VALUE(column, 3)` gives you the value from the **third row** of the group (or frame). It is like saying "give me the third item on the list."

#### Purposes

- To retrieve a specific row's value by position.
- To find the second, third, or N-th occurrence in a group.
- To compare against a median or a specific percentile position.
- To access values that are neither first nor last.

#### Syntax Rules and Structure

**General syntax**:

```sql
NTH_VALUE(expression, N) OVER (
    [ PARTITION BY expression [, ...] ]
    ORDER BY expression [ ASC | DESC ] [, ...]
    [ frame_clause ]
)
```

**PostgreSQL** (official) :

```sql
nth_value(value anyelement, n integer) OVER (...)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `expression` | The column or expression whose value is retrieved. |
| `N` | Positive integer specifying the row position within the frame. |
| `OVER` | Mandatory clause. |
| `PARTITION BY` | Optional; resets the row count per partition. |
| `ORDER BY` | Required to define row positions. |
| `frame_clause` | Optional; determines which rows are counted. |

**Syntax rules**

- `N` must be a positive integer.
- If `N` exceeds the frame size, `NULL` is returned.
- `NTH_VALUE` respects the frame.
- PostgreSQL supports only `FROM FIRST` (the default); `FROM LAST` is not implemented.

**Constraints and limitations**

- The default frame means the N-th row is counted from the partition start up to the current row's last peer; N may be unreachable for early rows.
- `IGNORE NULLS` support varies by engine.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Second-highest salary per department

```sql
SELECT depname, empno, salary,
       NTH_VALUE(salary, 2) OVER (
           PARTITION BY depname
           ORDER BY salary DESC
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS second_highest
FROM empsalary;
```

**Expected output**:

```text
 depname  | empno | salary | second_highest
----------+-------+--------+----------------
 develop  |     8 |   6000 |           5200
 develop  |    10 |   5200 |           5200
 develop  |    11 |   5200 |           5200
 develop  |     9 |   4500 |           5200
 develop  |     7 |   4200 |           5200
 personnel|     2 |   3900 |           3500
 personnel|     5 |   3500 |           3500
 sales    |     1 |   5000 |           4800
 sales    |     4 |   4800 |           4800
 sales    |     3 |   4800 |           4800
(10 rows)
```

**Why this output appears**: With the explicit full-partition frame ordered by salary descending, `NTH_VALUE(salary, 2)` returns the second-highest salary in each department, repeated on every row of that department.

##### Example 2: MySQL – Third sale amount per customer

```sql
SELECT customer_id, sale_date, amount,
       NTH_VALUE(amount, 3) OVER (
           PARTITION BY customer_id
           ORDER BY sale_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS third_sale_amount
FROM sales;
```

**Expected output** (illustrative):

```text
+-------------+------------+--------+-------------------+
| customer_id | sale_date  | amount | third_sale_amount |
+-------------+------------+--------+-------------------+
| 1           | 2025-01-01 | 100    | 300               |
| 1           | 2025-02-01 | 200    | 300               |
| 1           | 2025-03-01 | 300    | 300               |
| 1           | 2025-04-01 | 400    | 300               |
| 2           | 2025-01-15 | 50     | NULL              |
| 2           | 2025-02-15 | 60     | NULL              |
+-------------+------------+--------+-------------------+
```

**Why this output appears**: Customer 1 has at least three sales, so the third amount (300) is returned for all of customer 1's rows. Customer 2 has only two sales, so the function returns `NULL` for all of customer 2's rows.

#### Real-World Cases

- **Percentile analysis**: Retrieving the value at a specific rank (e.g., median is often the 50th percentile position).
- **Performance bands**: Finding the third-best score in a group.
- **Inventory milestones**: The amount of the third restock for a product.

---

### 6. Default Values

#### Definitions

**Core Definition**  
Default values in offset functions are the substitute values returned when the requested offset goes beyond the bounds of the partition.

**Technical Definition**  
`LAG` and `LEAD` accept an optional third parameter, `default`, which is returned when the offset points before the first row or after the last row of the partition. If omitted, the default is `NULL`. The default value must be type-compatible with the returned expression. `FIRST_VALUE`, `LAST_VALUE`, and `NTH_VALUE` do not have a default-value parameter; they return `NULL` when no row exists (or when `NTH_VALUE`'s N exceeds the frame size).

**Beginner-Friendly Explanation**  
A default value is what the function gives you when there is no previous or next row. Instead of getting `NULL` (which can cause problems in calculations), you can tell it to give you `0`, an empty string, or any other value.

#### Purposes

- To avoid `NULL` results on boundary rows.
- To make downstream calculations (like subtraction or division) safe.
- To provide meaningful placeholders for missing data.
- To ensure consistent output types.

#### Syntax Rules and Structure

**Syntax for `LAG` and `LEAD`**:

```sql
LAG(expression, offset, default) OVER (...)
LEAD(expression, offset, default) OVER (...)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `expression` | The value to retrieve. |
| `offset` | Number of rows back or forward. |
| `default` | Value to return if the row does not exist. |

**Syntax rules**

- `default` must be type-compatible with `expression`.
- `default` can be a literal, column, or expression.
- `default` cannot be a window function or aggregate.
- If omitted, `default` is `NULL`.

**Constraints and limitations**

- `FIRST_VALUE`, `LAST_VALUE`, and `NTH_VALUE` do not support a default parameter.
- Some engines require the `default` to be a constant expression.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Default `0` for missing previous values

```sql
SELECT month, revenue,
       revenue - LAG(revenue, 1, 0) OVER (ORDER BY month) AS change_from_zero
FROM monthly_sales;
```

**Expected output**:

```text
   month    | revenue | change_from_zero
------------+---------+------------------
 2025-01-01 |   10000 |            10000
 2025-02-01 |   12000 |             2000
 2025-03-01 |   11000 |            -1000
(3 rows)
```

**Why this output appears**: The first month has no previous row, so `LAG` returns the default `0`; `revenue - 0` equals the revenue itself. This avoids a `NULL` in the `change_from_zero` column.

##### Example 2: SQL Server – Default value for `LEAD`

```sql
SELECT OrderDate, TotalDue,
       LEAD(TotalDue, 1, 0) OVER (ORDER BY OrderDate) AS NextTotal
FROM Sales.SalesOrderHeader;
```

**Expected output**:

```text
OrderDate   TotalDue NextTotal
----------- -------- ---------
2025-01-01  100.00   150.00
2025-02-01  150.00   200.00
2025-03-01  200.00   0.00
```

**Why this output appears**: The last row has no following row, so `LEAD` returns `0` instead of `NULL`.

#### Real-World Cases

- **Financial calculations**: Using `0` as the default for missing prior balances.
- **Growth rates**: Using `1` or `0` as the default denominator to avoid division by zero.
- **Reporting**: Using empty strings or placeholders for missing prior values.

---

### 7. Use Cases

#### Definitions

**Core Definition**  
Use cases for offset functions are the practical analytical scenarios in which accessing a different row's value provides business insight.

**Technical Definition**  
Offset functions support period-over-period comparisons, gap detection, sessionization, baseline calculations, and positional retrieval. They eliminate self-joins and are fundamental to time-series analysis and sequential data processing.

**Beginner-Friendly Explanation**  
Offset functions are useful whenever you need to know "what came before," "what comes next," "what was the starting value," or "what is the third value."

#### Purposes

- To calculate growth rates and changes over time.
- To detect gaps, session breaks, and anomalies in sequential data.
- To establish baselines (first values) and endpoints (last values) for comparison.
- To retrieve specific positions (e.g., the second-best performer).

#### Syntax Rules and Structure

**Common patterns**:

```sql
-- Period-over-period change
value - LAG(value) OVER (PARTITION BY entity ORDER BY time)

-- Gap detection
time - LAG(time) OVER (PARTITION BY user ORDER BY time)

-- Baseline comparison
value - FIRST_VALUE(value) OVER (PARTITION BY group ORDER BY time)

-- Positional retrieval
NTH_VALUE(value, N) OVER (PARTITION BY group ORDER BY metric DESC)
```

**Component breakdown**

| Use case | Recommended function |
|----------|----------------------|
| Previous row comparison | `LAG()` |
| Next row comparison | `LEAD()` |
| First value baseline | `FIRST_VALUE()` |
| Last value (with explicit frame) | `LAST_VALUE()` |
| N-th position | `NTH_VALUE()` |
| Avoid `NULL` on boundaries | `LAG`/`LEAD` with `default` |

**Syntax rules**

- Always include `PARTITION BY` when comparing entities to avoid cross-entity comparisons.
- Always include `ORDER BY` to define the row sequence.
- Use `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` for true last-value semantics.
- Use a default value to avoid `NULL` in arithmetic.

**Constraints and limitations**

- Without `PARTITION BY`, boundary values bleed across entities.
- Without `ORDER BY`, results are non-deterministic.
- `LAST_VALUE` without an explicit frame often returns the current row's value.

---

### 8. Time-Series Analysis

#### Definitions

**Core Definition**  
Time-series analysis with offset functions involves comparing values across time-ordered rows to detect trends, changes, and patterns.

**Technical Definition**  
Offset functions are applied to time-ordered data to compute period-over-period deltas, moving averages, session gaps, and cumulative metrics. `LAG` and `LEAD` are the primary functions for point-to-point comparisons; `FIRST_VALUE` and `LAST_VALUE` establish period boundaries; `NTH_VALUE` accesses specific time points. When data is irregularly spaced, self-joins may be required for exact time offsets, but offset functions handle regular or row-based comparisons efficiently.

**Beginner-Friendly Explanation**  
Time-series analysis is about understanding how things change over time. Offset functions let you compare today's value to yesterday's, find gaps in a sequence of events, or see how the first day of the month compares to the last.

#### Purposes

- To compute growth rates (month-over-month, year-over-year).
- To detect gaps in event streams (sessionization).
- To calculate moving averages and rolling metrics.
- To compare current values against period baselines.
- To identify trends and anomalies.

#### Syntax Rules and Structure

**Common time-series patterns**:

```sql
-- Day-over-day change
value - LAG(value, 1) OVER (PARTITION BY sensor ORDER BY time)

-- Percentage change
(value - LAG(value) OVER (...)) / NULLIF(LAG(value) OVER (...), 0)

-- Session gap detection
time - LAG(time) OVER (PARTITION BY user ORDER BY time) > INTERVAL '30 minutes'

-- Year-over-year comparison
LAG(value, 12) OVER (PARTITION BY product ORDER BY month)
```

**Component breakdown**

| Pattern | Function | Frame |
|---------|----------|-------|
| Simple delta | `LAG`/`LEAD` | Ignored |
| Growth rate | `LAG`/`LEAD` + arithmetic | Ignored |
| Moving average | `AVG` with frame | `ROWS BETWEEN n PRECEDING AND CURRENT ROW` |
| Baseline comparison | `FIRST_VALUE` | Default (partition start) |
| Period-end comparison | `LAST_VALUE` | `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` |

**Syntax rules**

- Use `NULLIF` on the denominator when computing ratios to avoid division by zero.
- Use `PARTITION BY` entity ID to prevent cross-entity comparisons.
- Use `ORDER BY` time to ensure chronological sequence.
- For irregular time intervals, `LAG` with row offsets may not match exact time deltas; consider self-joins or `RANGE` frames with time intervals.

**Constraints and limitations**

- `LAG`/`LEAD` operate on row positions, not time values; irregular spacing requires careful offset selection.
- Moving averages require explicit `ROWS` frames; the default `RANGE` frame may include peers unintentionally.
- Sessionization thresholds require domain knowledge (e.g., 30-minute inactivity).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Month-over-month growth with percentage change

```sql
WITH monthly AS (
    SELECT DATE_TRUNC('month', sale_date) AS month, SUM(amount) AS revenue
    FROM sales
    GROUP BY DATE_TRUNC('month', sale_date)
)
SELECT month, revenue,
       LAG(revenue) OVER (ORDER BY month) AS prev_revenue,
       ROUND(
           (revenue - LAG(revenue) OVER (ORDER BY month)) * 100.0
           / NULLIF(LAG(revenue) OVER (ORDER BY month), 0),
           2
       ) AS mom_growth_pct
FROM monthly
ORDER BY month;
```

**Expected output** (illustrative):

```text
   month    | revenue | prev_revenue | mom_growth_pct
------------+---------+--------------+----------------
 2025-01-01 |   10000 |              |
 2025-02-01 |   12000 |        10000 |          20.00
 2025-03-01 |   11000 |        12000 |          -8.33
 2025-04-01 |   15000 |        11000 |          36.36
(4 rows)
```

**Why this output appears**: `LAG` retrieves the prior month's revenue; the percentage change is computed as `(current − previous) / previous × 100`. `NULLIF` prevents division by zero if previous revenue were `0`.

##### Example 2: InfluxDB – Detecting session gaps in IoT telemetry

```sql
SELECT sensor_id, reading_time, temperature,
       reading_time - LAG(reading_time) OVER (
           PARTITION BY sensor_id ORDER BY reading_time
       ) AS gap_interval
FROM sensor_readings;
```

**Expected output** (illustrative):

```text
sensor_id reading_time         temperature gap_interval
--------- ------------------- ----------- ------------
1         2025-01-01 10:00:00 20.5        
1         2025-01-01 10:01:00 20.7        00:01:00
1         2025-01-01 12:30:00 21.0        02:29:00
```

**Why this output appears**: `LAG(reading_time)` retrieves the previous reading's timestamp; subtracting gives the gap. A 2.5-hour gap indicates a likely outage or missing data.

##### Example 3: PostgreSQL – Year-over-year comparison with offset

```sql
SELECT product_id, month, revenue,
       LAG(revenue, 12) OVER (PARTITION BY product_id ORDER BY month) AS revenue_ly,
       revenue - LAG(revenue, 12) OVER (PARTITION BY product_id ORDER BY month) AS yoy_change
FROM monthly_product_revenue;
```

**Expected output** (illustrative):

```text
product_id month      revenue revenue_ly yoy_change
---------- ---------- ------- ---------- ----------
1          2025-01-01 1000               
1          2025-02-01 1100               
...
1          2026-01-01 1200    1000       200
1          2026-02-01 1300    1100       200
```

**Why this output appears**: `LAG(revenue, 12)` looks back 12 rows (months) to retrieve the same month last year; the year-over-year change is computed as the difference.

#### Real-World Cases

- **Financial dashboards**: Month-over-month and year-over-year revenue growth.
- **Web analytics**: Sessionization by detecting 30-minute inactivity gaps.
- **IoT monitoring**: Identifying sensor downtime or missing readings.
- **Supply chain**: Comparing current inventory to the same period last year.
- **Customer churn**: Detecting the last activity before a customer becomes inactive.

---

## References

- PostgreSQL Documentation – Window Functions (Offset Functions) – https://www.postgresql.org/docs/current/functions-window.html
- PostgreSQL Documentation – Window Functions Tutorial – https://www.postgresql.org/docs/current/tutorial-window.html
- MySQL 8.0 Reference Manual – Window Function Descriptions – https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/window-function-descriptions.html
- MySQL 8.0 Reference Manual – Window Function Concepts and Syntax – https://dev.mysql.com/doc/refman/8.0/en/window-functions-usage.html
- Microsoft Learn – LEAD (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/functions/lead-transact-sql
- Microsoft Learn – LAG (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/functions/lag-transact-sql
- Oracle Database SQL Language Reference – LAG/LEAD Functions – https://docs.oracle.com/en/database/oracle/oracle-database/23/sqlrf/LAG.html
- Oracle Data Warehousing Guide – FIRST_VALUE, LAST_VALUE, and NTH_VALUE – https://docs.oracle.com/en/database/oracle/oracle-database/23/dwhsg/sql-for-analysis-reporting.html
- Google Cloud BigQuery Documentation – Navigation Functions – https://cloud.google.com/bigquery/docs/reference/standard-sql/navigation_functions
- SQL Server Temporal Tables – LAG/LEAD Examples – https://learn.microsoft.com/en-us/sql/relational-databases/tables/temporal-table-usage-scenarios
- InfluxData Documentation – Compare Values Across Rows – https://docs.influxdata.com/influxdb3/cloud-serverless/query-data/sql/compare-values/
- SQL Server – IGNORE NULLS in LAG/LEAD – https://learn.microsoft.com/en-us/sql/t-sql/functions/lead-transact-sql#arguments
- SQLfmt – SQL LEAD and LAG Window Functions – https://www.sqlfmt.app/blog/sql-lead-and-lag-window-functions
- SQLite Documentation – Window Functions – https://www.sqlite.org/windowfunctions.html
- jOOQ – LAG Window Function – https://www.jooq.org/doc/latest/manual/sql-building/column-expressions/window-functions/lag-function/