# Comprehensive Programming Cheat Sheet: Aggregate Window Functions

## Topic Overview

### Definitions

**Core Definition**  
Aggregate window functions are standard SQL aggregate functions (`SUM`, `AVG`, `COUNT`, `MIN`, `MAX`) that are applied over a window—a set of rows related to the current row—rather than over an entire `GROUP BY` group, returning one aggregated value per row.

**Technical Definition**  
An aggregate window function is an ordinary aggregate function followed by an `OVER` clause. When an `OVER` clause is present, the aggregate computes over the rows within the current row's window frame instead of collapsing all rows into a single group result. The window frame is defined by `PARTITION BY` (logical grouping), `ORDER BY` (row sequence), and an optional frame clause (`ROWS`, `RANGE`, or `GROUPS`). Without `ORDER BY`, the default frame is the entire partition; with `ORDER BY` and no explicit frame, the default is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, which produces running-aggregate behavior. Aggregate window functions preserve every input row, unlike `GROUP BY` aggregates that collapse rows.

**Beginner-Friendly Explanation**  
Normally, `SUM(sales)` with `GROUP BY` gives you one total per group, and you lose the individual rows. An aggregate window function lets you keep every row and add a column that shows a calculation across a group of related rows—for example, each day's sales alongside the running total of sales so far, or each employee's salary alongside the department average. It is like writing a summary in the margin next to every line of a detailed report.

### Key Characteristics

- **Row-preserving**: Aggregate window functions return one value per input row; they do not reduce the result set.
- **`OVER` clause required**: The presence of `OVER` is what distinguishes a window aggregate from a plain aggregate; without it, the function acts as a grouping aggregate.
- **Frame-dependent**: The set of rows aggregated for each row is determined by the window frame. The frame can span the entire partition, a fixed number of physical rows, a logical value range, or peer groups.
- **Default running behavior**: With `ORDER BY` and no explicit frame, the default `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` produces cumulative (running) aggregates.
- **Partition-resetting**: `PARTITION BY` resets the aggregation for each partition; without it, the entire result set is one window.
- **Supported by all major engines**: PostgreSQL, MySQL 8.0+, SQL Server, Oracle, and SQLite 3.25+ support aggregate window functions.
- **Frame units**: `ROWS` counts physical rows, `RANGE` counts logical values (requires a single `ORDER BY` column for offsets in most engines), and `GROUPS` counts peer groups.

### Prerequisites

- Understanding of basic aggregate functions (`SUM`, `AVG`, `COUNT`, `MIN`, `MAX`) and `GROUP BY`.
- Familiarity with the `SELECT` statement and logical processing order.
- Knowledge of window function fundamentals (`OVER`, `PARTITION BY`, `ORDER BY`).
- Awareness of frame clauses (`ROWS`, `RANGE`, `GROUPS`) and default framing behavior.

### Related Programming Areas

- Time-series analysis and period-over-period reporting.
- Running totals, cumulative sums, and moving averages.
- Financial dashboards and KPI tracking.
- Data warehousing and OLAP-style analytics.
- Sessionization and gap detection.

---

## Core Concepts and Features

### 1. `SUM` over Window

#### Definitions

**Core Definition**  
`SUM` over a window returns the sum of a numeric expression across the rows of the current row's window frame.

**Technical Definition**  
`SUM(expression) OVER (...)` computes the total of `expression` over the rows included in the window frame for each row. When used with `ORDER BY` and the default frame, it produces a cumulative (running) sum. When used with `PARTITION BY` and no `ORDER BY`, it returns the partition-wide total on every row. The function ignores `NULL` values unless all rows in the frame are `NULL` (in which case it returns `NULL`). Supported by PostgreSQL, MySQL 8.0+, SQL Server, Oracle, and SQLite.

**Beginner-Friendly Explanation**  
`SUM` over a window tells you the total of a column for a group of rows related to the current row—like a running total of sales, or the total salary of everyone in the same department shown next to each employee.

#### Purposes

- To compute running totals and cumulative sums across ordered rows.
- To calculate partition-wide totals while retaining individual row detail.
- To support financial balances, inventory accumulation, and progressive metrics.
- To compare individual values against group totals without self-joins.

#### Syntax Rules and Structure

**General syntax**:

```sql
SUM(expression) OVER (
    [ PARTITION BY expression [, ...] ]
    [ ORDER BY expression [ ASC | DESC ] [, ...] ]
    [ frame_clause ]
)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `SUM(expression)` | Sums the numeric expression. |
| `OVER` | Mandatory clause. |
| `PARTITION BY` | Optional; resets the sum per partition. |
| `ORDER BY` | Optional; activates the default cumulative frame. |
| `frame_clause` | Optional; defines the exact rows included in the sum. |

**Syntax rules**

- `SUM` over a window ignores `NULL` values.
- With `ORDER BY` and no frame, the default `RANGE` frame includes all rows from the partition start through the current row's last peer.
- To obtain a whole-partition total, omit `ORDER BY` or use `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`.
- To obtain a strict row-by-row running total (excluding peers beyond the current physical row), use `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`.

**Constraints and limitations**

- The expression must be numeric or convertible to a numeric type.
- `SUM` cannot be used in `WHERE` or `HAVING`; filter on the result in an outer query.
- In SQL Server, aggregate window functions with `ORDER BY` may require a `ROWS` frame for deterministic cumulative behavior.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Running total of sales

```sql
SELECT sale_date, amount,
       SUM(amount) OVER (ORDER BY sale_date) AS running_total
FROM sales
ORDER BY sale_date;
```

**Expected output** (illustrative):

```text
 sale_date  | amount | running_total
------------+--------+---------------
 2025-01-01 |    100 |           100
 2025-01-02 |    150 |           250
 2025-01-03 |    200 |           450
 2025-01-04 |    120 |           570
(4 rows)
```

**Why this output appears**: `ORDER BY sale_date` activates the default `RANGE` frame, which accumulates amounts from the first row through the current row (and any peers). Each row shows the total sales up to that date.

##### Example 2: MySQL 8.0 – Partition-wide total vs running total

```sql
-- Partition-wide total on every row.
SELECT department, employee, salary,
       SUM(salary) OVER (PARTITION BY department) AS dept_total
FROM employees;

-- Running total within each department.
SELECT department, employee, salary,
       SUM(salary) OVER (PARTITION BY department ORDER BY salary) AS running_dept_total
FROM employees;
```

**Expected output** (partition-wide total):

```text
+------------+----------+--------+------------+
| department | employee | salary | dept_total |
+------------+----------+--------+------------+
| Sales      | Alice    |   5000 |      15000 |
| Sales      | Bob      |   4000 |      15000 |
| Sales      | Carol    |   6000 |      15000 |
| IT         | Dave     |   7000 |      14000 |
| IT         | Eve      |   7000 |      14000 |
+------------+----------+--------+------------+
```

**Expected output** (running total):

```text
+------------+----------+--------+--------------------+
| department | employee | salary | running_dept_total |
+------------+----------+--------+--------------------+
| IT         | Dave     |   7000 |               7000 |
| IT         | Eve      |   7000 |              14000 |
| Sales      | Bob      |   4000 |               4000 |
| Sales      | Alice    |   5000 |               9000 |
| Sales      | Carol    |   6000 |              15000 |
+------------+----------+--------+--------------------+
```

**Why this output appears**: Without `ORDER BY`, every row in a department sees the full department total. With `ORDER BY salary`, the sum accumulates within each department. The two 7000 salaries in IT are peers under the default `RANGE` frame, so both show 14000.

#### Real-World Cases

- **Financial dashboards**: Running balance of an account ordered by transaction date.
- **Inventory tracking**: Cumulative stock level from daily movements.
- **Sales reporting**: Year-to-date revenue alongside each transaction.

---

### 2. `AVG` over Window

#### Definitions

**Core Definition**  
`AVG` over a window returns the arithmetic mean of a numeric expression across the rows of the current row's window frame.

**Technical Definition**  
`AVG(expression) OVER (...)` computes the average of `expression` over the window frame. It ignores `NULL` values; if all values in the frame are `NULL`, it returns `NULL`. When used with `ORDER BY` and the default frame, it produces a cumulative average. When combined with a `ROWS BETWEEN n PRECEDING AND CURRENT ROW` frame, it produces a moving (rolling) average. Supported by PostgreSQL, MySQL 8.0+, SQL Server, Oracle, and SQLite.

**Beginner-Friendly Explanation**  
`AVG` over a window gives you the average of a group of related rows for each row—like a 7-day moving average of sales, or the average salary of a department shown next to each employee.

#### Purposes

- To compute moving averages for smoothing time-series data.
- To calculate partition-wide averages while retaining row detail.
- To compare individual values against group averages.
- To support trend analysis and anomaly detection.

#### Syntax Rules and Structure

**General syntax**:

```sql
AVG(expression) OVER (
    [ PARTITION BY expression [, ...] ]
    [ ORDER BY expression [ ASC | DESC ] [, ...] ]
    [ frame_clause ]
)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `AVG(expression)` | Computes the mean of the expression. |
| `OVER` | Mandatory clause. |
| `PARTITION BY` | Optional; resets the average per partition. |
| `ORDER BY` | Optional; activates the default cumulative frame. |
| `frame_clause` | Optional; defines the rows included in the average. |

**Syntax rules**

- `AVG` ignores `NULL` values.
- For a moving average, use an explicit `ROWS` frame such as `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`.
- Without `ORDER BY`, the entire partition is the frame, and every row shows the partition average.
- With `ORDER BY` and no frame, the result is a cumulative average (default `RANGE` frame).

**Constraints and limitations**

- `AVG` returns a numeric type; for temporal values, convert to numeric units first.
- The moving average window size must be chosen carefully; a window larger than the available rows uses only the available rows.
- Some engines disallow `DISTINCT` with windowed `AVG`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – 3-day moving average

```sql
SELECT sale_date, amount,
       AVG(amount) OVER (
           ORDER BY sale_date
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
       ) AS moving_avg_3day
FROM sales
ORDER BY sale_date;
```

**Expected output** (illustrative):

```text
 sale_date  | amount | moving_avg_3day
------------+--------+-----------------
 2025-01-01 |    100 |          100.00
 2025-01-02 |    150 |          125.00
 2025-01-03 |    200 |          150.00
 2025-01-04 |    120 |          156.67
(4 rows)
```

**Why this output appears**: The frame includes the current row and the two preceding physical rows. The first row has no preceding rows, so the average uses only available rows; by the third row, all three rows contribute.

##### Example 2: SQL Server – Department average alongside each employee

```sql
SELECT Department, Employee, Salary,
       AVG(Salary) OVER (PARTITION BY Department) AS DeptAvg
FROM Employees;
```

**Expected output** (illustrative):

```text
Department Employee Salary DeptAvg
---------- -------- ------ -------
IT         Dave     7000   7000
IT         Eve      7000   7000
Sales      Alice    5000   5000
Sales      Bob      4000   5000
Sales      Carol    6000   5000
```

**Why this output appears**: `PARTITION BY Department` computes the average per department and attaches it to every employee row in that department.

#### Real-World Cases

- **Stock analysis**: 50-day and 200-day moving averages of closing prices.
- **Web analytics**: 7-day rolling average of daily active users.
- **Quality control**: Moving average of sensor readings to smooth noise.

---

### 3. `COUNT` over Window

#### Definitions

**Core Definition**  
`COUNT` over a window returns the number of rows (or non-`NULL` values) in the current row's window frame.

**Technical Definition**  
`COUNT(expression) OVER (...)` counts the non-`NULL` values of `expression` in the window frame; `COUNT(*) OVER (...)` counts all rows in the frame regardless of `NULL`s. When used with `ORDER BY` and the default frame, it produces a cumulative count. With `PARTITION BY` and no `ORDER BY`, it returns the partition size on every row. Supported by PostgreSQL, MySQL 8.0+, SQL Server, Oracle, and SQLite.

**Beginner-Friendly Explanation**  
`COUNT` over a window tells you how many rows are in the current window—like “how many orders has this customer placed so far?” or “how many employees are in this department?”

#### Purposes

- To compute running counts and cumulative frequencies.
- To determine partition sizes (e.g., number of rows per group).
- To support pagination and row numbering patterns.
- To compare individual rows against group counts.

#### Syntax Rules and Structure

**General syntax**:

```sql
COUNT(expression) OVER (...)
COUNT(*) OVER (...)
```

**Component breakdown

| Component | Description |
|-----------|-------------|
| `COUNT(expression)` | Counts non-`NULL` values of the expression. |
| `COUNT(*)` | Counts all rows in the frame. |
| `OVER` | Mandatory clause. |
| `PARTITION BY` | Optional; resets the count per partition. |
| `ORDER BY` | Optional; activates the default cumulative frame. |
| `frame_clause` | Optional; defines the rows included in the count. |

**Syntax rules**

- `COUNT(*)` counts rows; `COUNT(expression)` counts non-`NULL` values.
- With `ORDER BY` and no frame, the count is cumulative (default `RANGE` frame).
- Without `ORDER BY`, the count is the entire partition size on every row.
- `COUNT(DISTINCT expression) OVER (...)` is allowed in some engines but not all.

**Constraints and limitations**

- `COUNT` cannot be used in `WHERE` or `HAVING`; filter in an outer query.
- `COUNT(DISTINCT ...) OVER (...)` is not supported in all engines (e.g., SQL Server does not allow it with `OVER` in the same way).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Cumulative count of events

```sql
SELECT event_time, event_type,
       COUNT(*) OVER (ORDER BY event_time) AS cumulative_events
FROM events
ORDER BY event_time;
```

**Expected output** (illustrative):

```text
 event_time          | event_type | cumulative_events
---------------------+------------+-------------------
 2025-01-01 10:00:00 | login      |                 1
 2025-01-01 10:05:00 | click      |                 2
 2025-01-01 10:10:00 | logout     |                 3
(3 rows)
```

**Why this output appears**: `ORDER BY event_time` with the default frame counts all events from the first through the current row, producing a running count.

##### Example 2: MySQL – Partition size on each row

```sql
SELECT department, employee,
       COUNT(*) OVER (PARTITION BY department) AS dept_size
FROM employees;
```

**Expected output**:

```text
+------------+----------+-----------+
| department | employee | dept_size |
+------------+----------+-----------+
| IT         | Dave     |         2 |
| IT         | Eve      |         2 |
| Sales      | Alice    |         3 |
| Sales      | Bob      |         3 |
| Sales      | Carol    |         3 |
+------------+----------+-----------+
```

**Why this output appears**: `PARTITION BY department` counts the rows in each department and repeats the size on every row of that department.

#### Real-World Cases

- **Session analysis**: Cumulative count of events within a session.
- **Inventory**: Running count of items received.
- **HR reporting**: Number of employees per department.

---

### 4. `MIN` over Window

#### Definitions

**Core Definition**  
`MIN` over a window returns the smallest value of an expression across the rows of the current row's window frame.

**Technical Definition**  
`MIN(expression) OVER (...)` returns the minimum non-`NULL` value of `expression` in the window frame. When used with `ORDER BY` and the default frame, it returns the minimum value seen so far (a running minimum). With `PARTITION BY` and no `ORDER BY`, it returns the partition-wide minimum on every row. Supported by PostgreSQL, MySQL 8.0+, SQL Server, Oracle, and SQLite.

**Beginner-Friendly Explanation**  
`MIN` over a window tells you the smallest value seen so far or in the group—like the lowest price so far this month, or the minimum salary in the department.

#### Purposes

- To compute running minimums (e.g., lowest price to date).
- To find the partition-wide minimum alongside each row.
- To support drawdown analysis in finance.
- To identify the earliest or smallest value in an ordered sequence.

#### Syntax Rules and Structure

**General syntax**:

```sql
MIN(expression) OVER (
    [ PARTITION BY expression [, ...] ]
    [ ORDER BY expression [ ASC | DESC ] [, ...] ]
    [ frame_clause ]
)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `MIN(expression)` | Returns the minimum value. |
| `OVER` | Mandatory clause. |
| `PARTITION BY` | Optional; resets the minimum per partition. |
| `ORDER BY` | Optional; activates the default cumulative frame. |
| `frame_clause` | Optional; defines the rows included. |

**Syntax rules**

- `MIN` ignores `NULL` values.
- With `ORDER BY` and no frame, it produces a running minimum (default `RANGE` frame).
- Without `ORDER BY`, it returns the partition-wide minimum on every row.
- An explicit `ROWS` frame can restrict the minimum to a sliding window.

**Constraints and limitations**

- `MIN` cannot be used in `WHERE` or `HAVING`; use an outer query.
- For temporal or string data, `MIN` follows the type's ordering rules.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Running minimum of stock price

```sql
SELECT trade_date, price,
       MIN(price) OVER (ORDER BY trade_date) AS running_min
FROM stock_prices
ORDER BY trade_date;
```

**Expected output** (illustrative):

```text
 trade_date | price | running_min
------------+-------+-------------
 2025-01-01 |   150 |         150
 2025-01-02 |   145 |         145
 2025-01-03 |   148 |         145
 2025-01-04 |   140 |         140
(4 rows)
```

**Why this output appears**: The default frame accumulates from the first row; the running minimum updates only when a lower price appears.

##### Example 2: SQL Server – Department minimum salary

```sql
SELECT Department, Employee, Salary,
       MIN(Salary) OVER (PARTITION BY Department) AS MinSalary
FROM Employees;
```

**Expected output**:

```text
Department Employee Salary MinSalary
---------- -------- ------ ---------
IT         Dave     7000   7000
IT         Eve      7000   7000
Sales      Alice    5000   4000
Sales      Bob      4000   4000
Sales      Carol    6000   4000
```

**Why this output appears**: `PARTITION BY Department` computes the minimum salary per department and attaches it to every row.

#### Real-World Cases

- **Finance**: Running lowest price for drawdown calculation.
- **HR**: Minimum salary per department for equity analysis.
- **IoT**: Minimum temperature reading so far in a session.

---

### 5. `MAX` over Window

#### Definitions

**Core Definition**  
`MAX` over a window returns the largest value of an expression across the rows of the current row's window frame.

**Technical Definition**  
`MAX(expression) OVER (...)` returns the maximum non-`NULL` value of `expression` in the window frame. With `ORDER BY` and the default frame, it produces a running maximum. With `PARTITION BY` and no `ORDER BY`, it returns the partition-wide maximum on every row. Supported by PostgreSQL, MySQL 8.0+, SQL Server, Oracle, and SQLite.

**Beginner-Friendly Explanation**  
`MAX` over a window tells you the largest value seen so far or in the group—like the highest temperature so far today, or the maximum salary in the department.

#### Purposes

- To compute running maximums (e.g., highest price to date).
- To find the partition-wide maximum alongside each row.
- To support peak analysis and threshold detection.
- To identify the latest or largest value in an ordered sequence.

#### Syntax Rules and Structure

**General syntax**:

```sql
MAX(expression) OVER (
    [ PARTITION BY expression [, ...] ]
    [ ORDER BY expression [ ASC | DESC ] [, ...] ]
    [ frame_clause ]
)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `MAX(expression)` | Returns the maximum value. |
| `OVER` | Mandatory clause. |
| `PARTITION BY` | Optional; resets the maximum per partition. |
| `ORDER BY` | Optional; activates the default cumulative frame. |
| `frame_clause` | Optional; defines the rows included. |

**Syntax rules**

- `MAX` ignores `NULL` values.
- With `ORDER BY` and no frame, it produces a running maximum.
- Without `ORDER BY`, it returns the partition-wide maximum on every row.
- An explicit frame can restrict the maximum to a sliding window.

**Constraints and limitations**

- `MAX` cannot be used in `WHERE` or `HAVING`.
- For temporal or string data, `MAX` follows the type's ordering rules.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Running maximum temperature

```sql
SELECT reading_time, temperature,
       MAX(temperature) OVER (ORDER BY reading_time) AS max_so_far
FROM sensor_readings
ORDER BY reading_time;
```

**Expected output** (illustrative):

```text
 reading_time        | temperature | max_so_far
---------------------+-------------+------------
 2025-01-01 10:00:00 |        20.5 |       20.5
 2025-01-01 10:05:00 |        21.0 |       21.0
 2025-01-01 10:10:00 |        20.8 |       21.0
 2025-01-01 10:15:00 |        22.1 |       22.1
(4 rows)
```

**Why this output appears**: The running maximum updates only when a higher temperature is encountered.

##### Example 2: MySQL – Highest salary in each department

```sql
SELECT department, employee, salary,
       MAX(salary) OVER (PARTITION BY department) AS max_salary
FROM employees;
```

**Expected output**:

```text
+------------+----------+--------+------------+
| department | employee | salary | max_salary |
+------------+----------+--------+------------+
| IT         | Dave     |   7000 |       7000 |
| IT         | Eve      |   7000 |       7000 |
| Sales      | Alice    |   5000 |       6000 |
| Sales      | Bob      |   4000 |       6000 |
| Sales      | Carol    |   6000 |       6000 |
+------------+----------+--------+------------+
```

**Why this output appears**: `PARTITION BY department` computes the maximum salary per department and repeats it on each row.

#### Real-World Cases

- **Finance**: Running peak price for trailing-stop analysis.
- **Operations**: Maximum load observed so far in a system.
- **Sales**: Top sale amount per region attached to each sale row.

---

### 6. Running Totals

#### Definitions

**Core Definition**  
A running total is a cumulative aggregate (usually `SUM`) that adds values from the start of the partition up to the current row.

**Technical Definition**  
A running total is produced by an aggregate window function with `ORDER BY` and a frame that starts at `UNBOUNDED PRECEDING` and ends at `CURRENT ROW`. With the default `RANGE` frame, peers of the current row are included, so rows tied on the `ORDER BY` column see the same cumulative total. With an explicit `ROWS` frame, the accumulation is strictly row-by-row, producing different totals for tied rows.

**Beginner-Friendly Explanation**  
A running total is like a scoreboard that keeps adding points as the game progresses: after each event, you see the total score so far.

#### Purposes

- To accumulate values over time (sales, balances, inventory).
- To track progress against a target.
- To support financial statements and cumulative KPIs.
- To compute year-to-date or month-to-date metrics.

#### Syntax Rules and Structure

**Default running total** (includes peers):

```sql
SUM(amount) OVER (ORDER BY date_column)
```

**Strict row-by-row running total** (excludes peers beyond the current row):

```sql
SUM(amount) OVER (ORDER BY date_column ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `SUM(amount)` | The value being accumulated. |
| `ORDER BY date_column` | Defines the sequence. |
| Default frame | `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` (peers included). |
| `ROWS` frame | Strict physical row accumulation. |

**Syntax rules**

- `ORDER BY` is required for a running total.
- The default frame includes peers; use `ROWS` for strict row-by-row accumulation.
- `PARTITION BY` resets the running total per partition.
- To get the whole-partition total instead of a running total, omit `ORDER BY` or use `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`.

**Constraints and limitations**

- Peer inclusion under the default `RANGE` frame can be surprising when `ORDER BY` has duplicates.
- Running totals are computed within the window frame; changing the frame changes the result.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Running total with peer awareness

```sql
SELECT sale_date, amount,
       SUM(amount) OVER (ORDER BY sale_date) AS running_total
FROM sales
ORDER BY sale_date;
```

**Expected output**:

```text
 sale_date  | amount | running_total
------------+--------+---------------
 2025-01-01 |    100 |           100
 2025-01-02 |    150 |           250
 2025-01-02 |     50 |           300
 2025-01-03 |    200 |           500
(4 rows)
```

**Why this output appears**: The two rows on 2025-01-02 are peers under the default `RANGE` frame; both see the cumulative total including both amounts (300). The next row adds 200 to reach 500.

##### Example 2: MySQL – Strict row-by-row running total

```sql
SELECT sale_date, amount,
       SUM(amount) OVER (
           ORDER BY sale_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_total_strict
FROM sales
ORDER BY sale_date;
```

**Expected output**:

```text
+------------+--------+----------------------+
| sale_date  | amount | running_total_strict |
+------------+--------+----------------------+
| 2025-01-01 |    100 |                  100 |
| 2025-01-02 |    150 |                  250 |
| 2025-01-02 |     50 |                  300 |
| 2025-01-03 |    200 |                  500 |
+------------+--------+----------------------+
```

**Why this output appears**: The `ROWS` frame processes each physical row in order; the two rows on the same date accumulate sequentially, producing the same final totals here but different intermediate values if there were more peers.

#### Real-World Cases

- **Banking**: Running balance after each transaction.
- **Sales**: Year-to-date revenue at each sale.
- **Inventory**: Cumulative stock level after each receipt or shipment.

---

### 7. Moving Averages

#### Definitions

**Core Definition**  
A moving average is an average computed over a sliding window of a fixed number of rows or a value range, centered on or trailing the current row.

**Technical Definition**  
A moving average is produced by `AVG` over a window with an explicit `ROWS` frame such as `ROWS BETWEEN n PRECEDING AND CURRENT ROW` (trailing), `ROWS BETWEEN n PRECEDING AND n FOLLOWING` (centered), or a `RANGE` frame with a logical offset (e.g., `RANGE BETWEEN INTERVAL '7 days' PRECEDING AND CURRENT ROW`). The window slides with the current row, recomputing the average for each position.

**Beginner-Friendly Explanation**  
A moving average smooths out fluctuations by averaging a fixed number of nearby data points—like a 3-day moving average of temperature that changes each day as the window slides forward.

#### Purposes

- To smooth noisy time-series data and reveal trends.
- To compute rolling metrics such as 7-day active users.
- To support technical analysis (e.g., 50-day moving average of stock prices).
- To reduce the impact of outliers in sequential data.

#### Syntax Rules and Structure

**Trailing moving average**:

```sql
AVG(value) OVER (ORDER BY time ROWS BETWEEN n PRECEDING AND CURRENT ROW)
```

**Centered moving average**:

```sql
AVG(value) OVER (ORDER BY time ROWS BETWEEN n PRECEDING AND n FOLLOWING)
```

**Logical range moving average**:

```sql
AVG(value) OVER (ORDER BY time RANGE BETWEEN INTERVAL '7 days' PRECEDING AND CURRENT ROW)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `AVG(value)` | The value being averaged. |
| `ORDER BY time` | Defines the sequence. |
| `ROWS BETWEEN ...` | Physical row offset. |
| `RANGE BETWEEN ...` | Logical value offset (requires compatible `ORDER BY` type). |

**Syntax rules**

- An explicit frame is required to define the window size; the default `RANGE` frame produces a cumulative average, not a moving average.
- `ROWS` counts physical rows; `RANGE` counts value units (e.g., days, numeric difference).
- For `RANGE` with offsets, most engines require exactly one `ORDER BY` column.
- At the boundaries, the window uses only the available rows.

**Constraints and limitations**

- Choosing the window size affects responsiveness vs. smoothness.
- `RANGE` with intervals requires a temporal `ORDER BY` column.
- Some engines do not support `RANGE` with numeric offsets (e.g., SQL Server does not support numeric `RANGE` offsets in window frames).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL 8.0 – 3-day trailing moving average

```sql
SELECT sale_date, amount,
       AVG(amount) OVER (
           ORDER BY sale_date
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
       ) AS moving_avg_3day
FROM sales
ORDER BY sale_date;
```

**Expected output**:

```text
+------------+--------+-----------------+
| sale_date  | amount | moving_avg_3day |
+------------+--------+-----------------+
| 2025-01-01 |    100 |          100.00 |
| 2025-01-02 |    150 |          125.00 |
| 2025-01-03 |    200 |          150.00 |
| 2025-01-04 |    120 |          156.67 |
+------------+--------+-----------------+
```

**Why this output appears**: The frame includes the current row and the two preceding physical rows. The first row uses only itself; by the third row, three values are averaged.

##### Example 2: PostgreSQL – 7-day logical range moving average

```sql
SELECT sale_date, amount,
       AVG(amount) OVER (
           ORDER BY sale_date
           RANGE BETWEEN INTERVAL '6 days' PRECEDING AND CURRENT ROW
       ) AS moving_avg_7day
FROM sales
ORDER BY sale_date;
```

**Expected output** (illustrative):

```text
 sale_date  | amount | moving_avg_7day
------------+--------+-----------------
 2025-01-01 |    100 |          100.00
 2025-01-03 |    200 |          150.00
 2025-01-06 |    150 |          150.00
 2025-01-10 |    300 |          225.00
(4 rows)
```

**Why this output appears**: The `RANGE` frame includes all rows whose `sale_date` is within 6 days before the current row (plus the current row), producing a time-based moving average regardless of how many physical rows fall in that period.

#### Real-World Cases

- **Stock analysis**: 50-day moving average of closing prices.
- **Web analytics**: 7-day rolling average of daily signups.
- **Manufacturing**: Moving average of defect rates over the last 100 units.

---

### 8. Cumulative Sums

#### Definitions

**Core Definition**  
A cumulative sum is the total of a value from the beginning of the partition up to the current row, synonymous with a running total.

**Technical Definition**  
A cumulative sum is a `SUM` over a window with a frame that starts at the partition beginning (`UNBOUNDED PRECEDING`) and ends at the current row. It is the most common aggregate window function and is the default behavior of `SUM` with `ORDER BY` and no explicit frame. Unlike a moving average, the window grows as the current row advances, so every preceding row contributes.

**Beginner-Friendly Explanation**  
A cumulative sum is a running total: it adds every value from the start of the list up to the current item, so the total grows monotonically (assuming non-negative values).

#### Purposes

- To track accumulated totals over time.
- To compute year-to-date and month-to-date figures.
- To support financial balance calculations.
- To measure cumulative progress toward a goal.

#### Syntax Rules and Structure

**Syntax**:

```sql
SUM(expression) OVER (
    [ PARTITION BY ... ]
    ORDER BY ...
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `SUM(expression)` | The value being accumulated. |
| `ORDER BY` | Defines the sequence. |
| `UNBOUNDED PRECEDING` | Start at the partition beginning. |
| `CURRENT ROW` | End at the current row. |

**Syntax rules**

- The frame must start at `UNBOUNDED PRECEDING` to include all preceding rows.
- `ROWS` produces strict physical accumulation; `RANGE` includes peers.
- `PARTITION BY` resets the cumulative sum per group.
- The default frame with `ORDER BY` already produces a cumulative sum (with peer inclusion).

**Constraints and limitations**

- Cumulative sums can grow large; ensure the data type can hold the maximum total.
- For strict row-by-row accumulation, use `ROWS` rather than the default `RANGE`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Oracle – Cumulative sales by quarter

```sql
SELECT cust_id, calendar_quarter_desc,
       SUM(amount_sold) AS q_sales,
       SUM(SUM(amount_sold)) OVER (
           PARTITION BY cust_id
           ORDER BY calendar_quarter_desc
           ROWS UNBOUNDED PRECEDING
       ) AS cum_sales
FROM sales, times, customers
WHERE ...
GROUP BY cust_id, calendar_quarter_desc;
```

**Expected output** (illustrative):

```text
CUST_ID CALENDAR_QUARTER_DESC Q_SALES CUM_SALES
------- --------------------- ------- ---------
6380    1999-Q1               1000    1000
6380    1999-Q2               1500    2500
6380    1999-Q3               1200    3700
6380    1999-Q4               1800    5500
```

**Why this output appears**: The inner `SUM` aggregates quarterly sales; the outer `SUM(...) OVER (...)` accumulates the quarterly totals cumulatively per customer.

##### Example 2: SQL Server – Year-to-date sales

```sql
SELECT SalesPerson, SaleDate, Amount,
       SUM(Amount) OVER (
           PARTITION BY SalesPerson
           ORDER BY SaleDate
           ROWS UNBOUNDED PRECEDING
       ) AS YTD_Sales
FROM Sales;
```

**Expected output**:

```text
SalesPerson SaleDate   Amount YTD_Sales
----------- ---------- ------ ---------
Alice       2025-01-15 100    100
Alice       2025-02-10 150    250
Alice       2025-03-05 200    450
Bob         2025-01-20 50     50
Bob         2025-02-25 75     125
```

**Why this output appears**: The cumulative sum resets per salesperson (`PARTITION BY`) and accumulates in date order (`ORDER BY SaleDate`) across all rows from the partition start.

#### Real-World Cases

- **Finance**: Cumulative profit or loss over a fiscal year.
- **Operations**: Cumulative units produced on an assembly line.
- **Marketing**: Cumulative leads generated since campaign start.

---

### 9. Sliding Windows

#### Definitions

**Core Definition**  
A sliding window is a window frame that moves with the current row, including a fixed number of preceding and/or following rows or a fixed value range.

**Technical Definition**  
A sliding window is defined by a frame clause with offsets on both sides, such as `ROWS BETWEEN n PRECEDING AND m FOLLOWING` or `RANGE BETWEEN ... PRECEDING AND ... FOLLOWING`. Unlike a cumulative frame (`UNBOUNDED PRECEDING` to `CURRENT ROW`), a sliding window has a bounded size that remains constant (or nearly constant) as the current row advances. Sliding windows are used for moving averages, moving sums, and rolling statistics.

**Beginner-Friendly Explanation**  
A sliding window is like looking through a window that shows a fixed number of nearby rows—for example, the current row plus the two before and two after it. As you move to the next row, the window slides forward, always showing the same number of rows (except at the boundaries).

#### Purposes

- To compute moving averages and rolling statistics over a fixed window size.
- To smooth data and reduce noise.
- To compute trailing sums and centered aggregates.
- To analyse local trends rather than cumulative history.

#### Syntax Rules and Structure

**Trailing sliding window**:

```sql
ROWS BETWEEN n PRECEDING AND CURRENT ROW
```

**Centered sliding window**:

```sql
ROWS BETWEEN n PRECEDING AND n FOLLOWING
```

**Value-based sliding window**:

```sql
RANGE BETWEEN offset PRECEDING AND offset FOLLOWING
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `ROWS` | Physical row offsets. |
| `RANGE` | Logical value offsets (requires compatible `ORDER BY` type). |
| `n PRECEDING` | Rows/values before the current row. |
| `m FOLLOWING` | Rows/values after the current row. |

**Syntax rules**

- An explicit frame is required; the default frame is cumulative, not sliding.
- `ROWS` counts physical rows; `RANGE` counts value units.
- At partition boundaries, the window uses only the available rows.
- `PARTITION BY` resets the sliding window per group.

**Constraints and limitations**

- `RANGE` with offsets requires a single `ORDER BY` column in most engines.
- SQL Server does not support `RANGE` with numeric offsets in window frames.
- Choice of window size involves a trade-off between responsiveness and smoothness.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL 8.0 – Centered 3-row moving average

```sql
SELECT time, val,
       AVG(val) OVER (
           ORDER BY time
           ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
       ) AS centered_avg
FROM observations;
```

**Expected output** (excerpt):

```text
+----------+------+-------------+
| time     | val  | centered_avg|
+----------+------+-------------+
| 07:00:00 |   10 |      9.5000 |
| 07:15:00 |    9 |     14.6667 |
| 07:30:00 |   25 |     18.0000 |
| 07:45:00 |   20 |     22.5000 |
+----------+------+-------------+
```

**Why this output appears**: The frame includes the current row, one row before, and one row after. The first row has no preceding row, so the average uses only available rows.

##### Example 2: PostgreSQL – Sliding window with `RANGE` and an interval

```sql
SELECT sale_date, amount,
       SUM(amount) OVER (
           ORDER BY sale_date
           RANGE BETWEEN INTERVAL '3 days' PRECEDING AND INTERVAL '3 days' FOLLOWING
       ) AS sliding_7day_sum
FROM sales
ORDER BY sale_date;
```

**Expected output** (illustrative):

```text
 sale_date  | amount | sliding_7day_sum
------------+--------+------------------
 2025-01-01 |    100 |              300
 2025-01-03 |    200 |              600
 2025-01-06 |    150 |              450
 2025-01-10 |    300 |              300
```

**Why this output appears**: The `RANGE` frame includes all rows within 3 days before and after each row’s date, producing a time-based sliding sum.

#### Real-World Cases

- **Signal processing**: Moving average filter over sensor readings.
- **Finance**: Rolling volatility over a 30-day window.
- **E-commerce**: Rolling 7-day sum of orders for demand forecasting.

---

## References

- PostgreSQL Documentation – Window Functions (Reference) – https://www.postgresql.org/docs/current/functions-window.html
- PostgreSQL Documentation – Window Functions Tutorial – https://www.postgresql.org/docs/current/tutorial-window.html
- PostgreSQL Documentation – `SELECT` (Window Frame Syntax) – https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-WINDOW-FUNCTIONS
- MySQL 8.0 Reference Manual – Aggregate Function Descriptions – https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/aggregate-functions.html
- MySQL 8.0 Reference Manual – Window Function Frame Specification – https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/window-functions-frames.html
- MySQL 8.0 Reference Manual – Window Function Concepts and Syntax – https://dev.mysql.com/doc/refman/8.0/en/window-functions-usage.html
- Microsoft Learn – `OVER` Clause (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql
- Oracle Database Data Warehousing Guide – Windowing Aggregate Functions – https://docs.oracle.com/cd/A91034_01/DOC/server.901/a90237/analysis.htm
- Oracle Database SQL Language Reference – Analytic Functions – https://docs.oracle.com/en/database/oracle/oracle-database/23/sqlrf/Analytic-Functions.html
- SQLite Documentation – Window Functions – https://www.sqlite.org/windowfunctions.html
- SQL:2003 Standard – Window Functions (ISO/IEC 9075-2:2003)
- Ubuntu Manpages – `SELECT` Default Framing Option – https://manpages.ubuntu.com/manpages/noble/man7/SELECT.7.html
- LearnSQL – Window Functions: Aggregating Data – https://learnsql.com/blog/sql-window-functions/
- Use The Index, Luke – Window Functions and Indexing – https://use-the-index-luke.com/sql/partial-results/window-functions