# SQL Time-Series Analysis: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL time-series analysis is the practice of grouping, aggregating, and comparing data points indexed by time—using date truncation functions, window functions, and sequential ordering—to identify trends, patterns, and period-over-period changes.

**Technical Definition:** Time-series analysis in SQL uses date/time functions (`DATE_TRUNC`, `EXTRACT`, `DATEPART`, `DATE_FORMAT`) to bucket timestamps into uniform intervals (day, week, month, year), aggregate functions (`SUM`, `AVG`, `COUNT`) to summarize each bucket, and window functions (`LAG`, `LEAD`, `AVG() OVER`, `SUM() OVER`) with frame specifications (`ROWS BETWEEN`, `RANGE BETWEEN`) to compute rolling averages, cumulative totals, and period-over-period comparisons. The key technical considerations include: time zone handling, ISO week alignment, partition-aware window calculations, and the distinction between `ROWS` (physical row offsets) and `RANGE` (value-based or interval-based offsets).

**Beginner-Friendly Explanation:** Time-series analysis is like looking at a graph of your data over time. You want to know: How much did we sell each day, week, or month? Is this month better than last month? Is this year better than last year? What is the average of the last 7 days? SQL gives you tools to group your data by time periods, compare one period to another, and calculate running totals and moving averages—all with a few queries.

### Key Characteristics

- **Time-indexed:** The data is ordered by a date or timestamp column, which determines the sequence of analysis.
- **Bucketed:** Time-series analysis requires grouping timestamps into uniform intervals (day, week, month, quarter, year).
- **Sequential:** Unlike other analytical queries, time-series analysis depends on the order of rows; `ORDER BY` is mandatory for window functions.
- **Non-additive:** Rolling averages, cumulative totals, and period-over-period comparisons are non-additive across time periods; they must be recomputed from base data.
- **Frame-aware:** The window frame (`ROWS` vs. `RANGE`, `PRECEDING` vs. `FOLLOWING`) determines which rows are included in the calculation.
- **Vendor-varied:** Date truncation syntax differs significantly across PostgreSQL (`DATE_TRUNC`), SQL Server (`DATETRUNC`), MySQL (`DATE_FORMAT`), and Oracle (`TRUNC`).

### Prerequisites

- Proficiency with SQL `SELECT`, `WHERE`, `GROUP BY`, and `ORDER BY`.
- Understanding of aggregate functions (`SUM`, `AVG`, `COUNT`, `MIN`, `MAX`).
- Familiarity with window functions and the `OVER` clause (`PARTITION BY`, `ORDER BY`).
- Knowledge of date/time data types and time zone concepts.
- Awareness of `NULL` semantics in window function calculations.

### Related Programming Areas

- Business intelligence and executive dashboards.
- Financial analysis and revenue reporting.
- DevOps and infrastructure monitoring.
- IoT and sensor data analytics.
- Data warehousing and ETL pipeline design.

### Core Concepts / Features

1. **Daily Aggregation** (`DATE()`, `TRUNC()`, `DATE_TRUNC('day', timestamp)`)
2. **Weekly Aggregation** (`EXTRACT(WEEK)`, `DATE_TRUNC('week')`, ISO-week rules)
3. **Monthly Aggregation** (`DATE_TRUNC('month')`, `TO_CHAR`, `DATE_FORMAT`)
4. **Year-over-Year (YoY) Comparison** (`LAG(expression, offset)`, self-joins)
5. **Month-over-Month (MoM) Comparison** (`LAG()`, `LEAD()`)
6. **Rolling Averages** (`AVG() OVER (ORDER BY date ROWS BETWEEN n PRECEDING AND CURRENT ROW)`)
7. **Cumulative Totals** (`SUM() OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)`)


## Core Concept 1: Daily Aggregation

### Definitions

**Core Definition:** Daily aggregation is the process of grouping time-series data by individual calendar days, using date truncation functions to reduce timestamps to their date component and computing aggregate metrics per day.

**Technical Definition:** Daily aggregation uses `DATE_TRUNC('day', timestamp)` (PostgreSQL, Amazon Redshift), `TRUNC(date)` (Oracle), `DATE(timestamp)` (MySQL), or `CAST(timestamp AS DATE)` (SQL Server) to strip the time component from a timestamp, producing a date value that can be used in `GROUP BY`. The function returns the timestamp truncated to the specified precision; for `day`, it returns midnight (00:00:00) of that day.

**Beginner-Friendly Explanation:** Daily aggregation is like summarizing a day's worth of activity into a single row. Instead of looking at 10,000 individual transactions, you group them by day and see "Monday: $5,000, Tuesday: $4,200" and so on.

### Purposes

- To reduce high-frequency timestamp data into daily summaries for reporting.
- To identify day-of-week patterns and daily trends.
- To provide the base granularity for weekly and monthly aggregation.
- To detect daily anomalies and outliers.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
SELECT
    DATE_TRUNC('day', timestamp_column) AS day,
    aggregate_function(metric_column) AS metric_value
FROM table_name
[WHERE date_range_condition]
GROUP BY DATE_TRUNC('day', timestamp_column)
ORDER BY day;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `DATE_TRUNC('day', timestamp_column)` | Truncates the timestamp to the start of the day (midnight). |
| `aggregate_function(metric_column)` | The metric to aggregate (e.g., `SUM`, `AVG`, `COUNT`). |
| `GROUP BY DATE_TRUNC(...)` | Groups rows by day. |

#### Complete General Syntax (SQL Server, MySQL, Oracle)

```sql
-- SQL Server: CAST to DATE
SELECT CAST(timestamp_column AS DATE) AS day, SUM(amount)
FROM table_name
GROUP BY CAST(timestamp_column AS DATE);

-- MySQL: DATE()
SELECT DATE(timestamp_column) AS day, SUM(amount)
FROM table_name
GROUP BY DATE(timestamp_column);

-- Oracle: TRUNC()
SELECT TRUNC(timestamp_column) AS day, SUM(amount)
FROM table_name
GROUP BY TRUNC(timestamp_column);
```

#### Syntax Rules

- **`DATE_TRUNC` vs. `CAST`:** `DATE_TRUNC('day', ...)` returns a `timestamp` at midnight; `CAST(... AS DATE)` returns a `date` type. Both work for grouping.
- **Time zone:** `DATE_TRUNC` on a `TIMESTAMPTZ` uses the session time zone. For consistent results, convert to a specific time zone first using `AT TIME ZONE`.
- **Index usage:** Applying a function to a column in `WHERE` or `GROUP BY` prevents index usage. Consider a functional index or a pre-computed date column.

#### Constraints and Limitations

- **SQL Server:** `DATE_TRUNC` does not exist; use `DATETRUNC` (SQL Server 2022+) or `CAST(... AS DATE)`.
- **MySQL:** `DATE_TRUNC` does not exist; use `DATE()`.
- **Oracle:** Use `TRUNC(date_column)`; `TRUNC` without a format model truncates to the nearest day.
- **Time zone sensitivity:** `DATE_TRUNC('day', timestamptz '2001-02-16 20:38:40+00', 'Australia/Sydney')` returns `2001-02-16 13:00:00+00`, which is midnight in Sydney. Always specify the time zone for consistent daily boundaries.

### Annotated Code Examples

#### Example 1: PostgreSQL — Daily Revenue Aggregation

```sql
-- Create a sample transactions table
CREATE TABLE transactions (
    transaction_id SERIAL PRIMARY KEY,
    transaction_at TIMESTAMPTZ,
    amount NUMERIC(10,2)
);

INSERT INTO transactions (transaction_at, amount) VALUES
('2026-09-01 08:15:00+00', 100.00),
('2026-09-01 14:30:00+00', 200.00),
('2026-09-02 09:00:00+00', 150.00),
('2026-09-02 18:45:00+00', 300.00),
('2026-09-03 11:30:00+00', 250.00);

-- Daily revenue aggregation
SELECT
    DATE_TRUNC('day', transaction_at) AS day,
    SUM(amount) AS daily_revenue,
    COUNT(*) AS transaction_count
FROM transactions
GROUP BY DATE_TRUNC('day', transaction_at)
ORDER BY day;
```

**Expected Output:**

```
         day         | daily_revenue | transaction_count
---------------------+---------------+-------------------
 2026-09-01 00:00:00 |        300.00 |                 2
 2026-09-02 00:00:00 |        450.00 |                 2
 2026-09-03 00:00:00 |        250.00 |                 1
```

**Why This Works:** `DATE_TRUNC('day', transaction_at)` strips the time component, returning midnight of each day. The `GROUP BY` clause groups all transactions on the same day. `SUM` and `COUNT` aggregate the metrics.

#### Example 2: MySQL — Daily Aggregation with DATE()

```sql
-- MySQL daily aggregation
SELECT
    DATE(transaction_at) AS day,
    SUM(amount) AS daily_revenue,
    COUNT(*) AS transaction_count
FROM transactions
GROUP BY DATE(transaction_at)
ORDER BY day;
```

**Expected Output:**

```
    day       | daily_revenue | transaction_count
--------------+---------------+-------------------
 2026-09-01   |        300.00 |                 2
 2026-09-02   |        450.00 |                 2
 2026-09-03   |        250.00 |                 1
```

**Why This Works:** MySQL's `DATE()` function extracts the date portion of a datetime or timestamp value, returning a `DATE` type that can be used in `GROUP BY`.

### Real-World Cases

- **E-commerce:** Daily sales reporting and order volume tracking.
- **IoT:** Daily average sensor readings and anomaly detection.
- **DevOps:** Daily error rates and request volumes from application logs.
- **Finance:** Daily transaction volumes and account balances.

### References

- PostgreSQL: date_trunc — https://www.postgresql.org/docs/current/functions-datetime.html
- MySQL: DATE() — https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html
- SQL Server: DATETRUNC — https://learn.microsoft.com/en-us/sql/t-sql/functions/datetrunc-transact-sql
- Oracle: TRUNC (date) — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/TRUNC-date.html


## Core Concept 2: Weekly Aggregation

### Definitions

**Core Definition:** Weekly aggregation is the process of grouping time-series data by week, using week-numbering or week-truncation functions to assign each row to a calendar week.

**Technical Definition:** Weekly aggregation uses `DATE_TRUNC('week', date)` (PostgreSQL, which returns the Monday of the ISO week), `DATETRUNC(week, date)` or `DATETRUNC(iso_week, date)` (SQL Server 2022+), `EXTRACT(WEEK FROM date)` (PostgreSQL, returns the ISO week number), `DATEPART(ISO_WEEK, date)` (SQL Server), or `WEEK(date)` (MySQL). ISO 8601 defines weeks as starting on Monday, with the first week of the year containing January 4. In T-SQL, the first day of the week is defined by the `@@DATEFIRST` setting; for a U.S. English environment, `@@DATEFIRST` defaults to 7 (Sunday). The `iso_week` datepart always starts on Monday.

**Beginner-Friendly Explanation:** Weekly aggregation is like summarizing a week's worth of data into one row. You need to be careful about two things: What day does the week start on? (Monday is the ISO standard, but some systems start on Sunday.) And what happens at the end of the year? ISO weeks can cross year boundaries, so the last days of December might belong to week 1 of the next year.

### Purposes

- To reduce daily data into weekly summaries for trend analysis.
- To align reporting with business weeks (which may start on Sunday or Monday).
- To compute week-over-week comparisons.
- To smooth out daily noise and highlight weekly patterns.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
-- DATE_TRUNC('week') returns Monday of the week (ISO 8601)
SELECT
    DATE_TRUNC('week', date_column) AS week_start,
    SUM(metric) AS weekly_total
FROM table_name
GROUP BY DATE_TRUNC('week', date_column)
ORDER BY week_start;

-- EXTRACT(WEEK FROM date) returns the ISO week number
SELECT
    EXTRACT(ISOYEAR FROM date_column) AS iso_year,
    EXTRACT(WEEK FROM date_column) AS iso_week,
    SUM(metric) AS weekly_total
FROM table_name
GROUP BY EXTRACT(ISOYEAR FROM date_column), EXTRACT(WEEK FROM date_column)
ORDER BY iso_year, iso_week;
```

#### Complete General Syntax (SQL Server)

```sql
-- DATETRUNC with week (respects @@DATEFIRST)
SELECT
    DATETRUNC(week, date_column) AS week_start,
    SUM(metric) AS weekly_total
FROM table_name
GROUP BY DATETRUNC(week, date_column)
ORDER BY week_start;

-- DATETRUNC with iso_week (always Monday)
SELECT
    DATETRUNC(iso_week, date_column) AS iso_week_start,
    SUM(metric) AS weekly_total
FROM table_name
GROUP BY DATETRUNC(iso_week, date_column)
ORDER BY iso_week_start;
```

#### Syntax Rules

- **ISO week rules:** ISO weeks start on Monday, and the first week of a year contains January 4. In other words, the first Thursday of a year is in week 1 of that year.
- **`iso_week` vs. `week`:** In SQL Server, `DATETRUNC(week, ...)` respects the `@@DATEFIRST` setting; `DATETRUNC(iso_week, ...)` always starts on Monday. The `EXTRACT(WEEK FROM ...)` in PostgreSQL uses ISO numbering by default.
- **Year boundary:** When using `EXTRACT(WEEK)`, always pair it with `EXTRACT(ISOYEAR)` to avoid ambiguity at year boundaries.
- **MySQL:** `WEEK(date, mode)` allows specifying the week mode (0–7); `WEEK(date, 3)` follows ISO 8601 (Monday start, week 1 contains January 4).

#### Constraints and Limitations

- **@@DATEFIRST dependency (SQL Server):** `DATEPART(week, ...)` depends on the session's `@@DATEFIRST` setting, which can produce different results across sessions. Use `DATEPART(ISO_WEEK, ...)` for consistency.
- **Week 53:** Some years have 53 ISO weeks; ensure your grouping handles this correctly.
- **PostgreSQL DATE_TRUNC('week'):** Returns the Monday of the ISO week. If you need a different start day, you must adjust manually.
- **Year boundary ambiguity:** A date like `2026-01-01` may belong to ISO week 53 of 2025, not week 1 of 2026. Use `ISOYEAR` alongside `WEEK`.

### Annotated Code Examples

#### Example 1: PostgreSQL — Weekly Aggregation with ISO Week

```sql
-- Weekly revenue aggregation (ISO weeks, Monday start)
SELECT
    DATE_TRUNC('week', transaction_at) AS week_start,
    SUM(amount) AS weekly_revenue,
    COUNT(*) AS transaction_count
FROM transactions
GROUP BY DATE_TRUNC('week', transaction_at)
ORDER BY week_start;
```

**Expected Output:**

```
     week_start      | weekly_revenue | transaction_count
---------------------+----------------+-------------------
 2026-08-31 00:00:00 |         300.00 |                 2
 2026-09-07 00:00:00 |         700.00 |                 3
```

**Why This Works:** `DATE_TRUNC('week', ...)` returns the Monday of the ISO week. September 1, 2026 is a Tuesday, so it falls in the week starting Monday, August 31. September 2 and 3 fall in the week starting September 7. The aggregation groups by week start.

#### Example 2: SQL Server — ISO Week vs. Calendar Week

```sql
-- Compare DATETRUNC week (respects @@DATEFIRST) vs. iso_week (always Monday)
SELECT
    DATETRUNC(week, transaction_at) AS week_start,
    DATETRUNC(iso_week, transaction_at) AS iso_week_start,
    SUM(amount) AS weekly_revenue
FROM transactions
GROUP BY DATETRUNC(week, transaction_at), DATETRUNC(iso_week, transaction_at)
ORDER BY iso_week_start;
```

**Expected Output:**

```
 week_start            | iso_week_start        | weekly_revenue
-----------------------+-----------------------+----------------
 2026-08-30 00:00:00   | 2026-08-31 00:00:00   |         300.00
 2026-09-06 00:00:00   | 2026-09-07 00:00:00   |         700.00
```

**Why This Works:** With `@@DATEFIRST = 7` (Sunday), `DATETRUNC(week, ...)` starts weeks on Sunday, while `DATETRUNC(iso_week, ...)` starts on Monday. The difference is visible in the output.

### Real-World Cases

- **Retail:** Weekly sales reporting and week-over-week comparison.
- **SaaS:** Weekly active user counts and weekly recurring revenue.
- **HR:** Weekly payroll summaries and timesheet aggregation.
- **DevOps:** Weekly deployment counts and incident summaries.

### References

- PostgreSQL: date_trunc — https://www.postgresql.org/docs/current/functions-datetime.html
- PostgreSQL: EXTRACT (ISO Week) — https://www.postgresql.org/docs/current/functions-datetime.html
- SQL Server: DATETRUNC — https://learn.microsoft.com/en-us/sql/t-sql/functions/datetrunc-transact-sql
- MySQL: WEEK() — https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html


## Core Concept 3: Monthly Aggregation

### Definitions

**Core Definition:** Monthly aggregation is the process of grouping time-series data by calendar month, using month-truncation or formatting functions to reduce dates to their month component and computing aggregate metrics per month.

**Technical Definition:** Monthly aggregation uses `DATE_TRUNC('month', date)` (PostgreSQL), `DATETRUNC(month, date)` (SQL Server 2022+), `DATE_FORMAT(date, '%Y-%m')` (MySQL), or `TRUNC(date, 'MM')` (Oracle) to group data by month. The `DATE_FORMAT` function can also be used to produce formatted month labels like `'2026-09'` or `'Sep 2026'` for reporting.

**Beginner-Friendly Explanation:** Monthly aggregation is like summarizing a month's worth of data into one row. It is the most common granularity for business reporting—monthly revenue, monthly active users, monthly expenses. You group all the days of a month together and compute totals, averages, or counts.

### Purposes

- To produce monthly reports for business review.
- To track month-over-month performance and growth.
- To align with financial reporting cycles (monthly close).
- To reduce data volume for dashboard visualization.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
SELECT
    DATE_TRUNC('month', date_column) AS month_start,
    SUM(metric) AS monthly_total
FROM table_name
GROUP BY DATE_TRUNC('month', date_column)
ORDER BY month_start;
```

#### Complete General Syntax (MySQL)

```sql
SELECT
    DATE_FORMAT(date_column, '%Y-%m') AS month_label,
    SUM(metric) AS monthly_total
FROM table_name
GROUP BY DATE_FORMAT(date_column, '%Y-%m')
ORDER BY month_label;
```

#### Complete General Syntax (SQL Server, Oracle)

```sql
-- SQL Server
SELECT DATETRUNC(month, date_column) AS month_start, SUM(metric)
FROM table_name
GROUP BY DATETRUNC(month, date_column);

-- Oracle
SELECT TRUNC(date_column, 'MM') AS month_start, SUM(metric)
FROM table_name
GROUP BY TRUNC(date_column, 'MM');
```

#### Syntax Rules

- **Month start vs. month label:** `DATE_TRUNC('month', ...)` returns the first day of the month (e.g., `2026-09-01`); `DATE_FORMAT(..., '%Y-%m')` returns a string label (e.g., `'2026-09'`).
- **MySQL `DATE_FORMAT`:** The `%Y-%m` format produces a sortable month label. Other formats include `%b %Y` for `'Sep 2026'` and `%M %Y` for `'September 2026'`.
- **Oracle `TRUNC`:** `TRUNC(date, 'MM')` returns the first day of the month. `TO_CHAR(date, 'YYYY-MM')` produces a string label.

#### Constraints and Limitations

- **String sorting:** Month labels like `'Sep 2026'` do not sort chronologically; use `'2026-09'` for sortable labels.
- **Missing months:** If a month has no data, it will not appear in the result. Use a calendar table or `generate_series` to fill gaps.
- **Time zone:** Ensure month boundaries align with the business time zone.

### Annotated Code Examples

#### Example 1: PostgreSQL — Monthly Revenue with DATE_TRUNC

```sql
-- Monthly revenue aggregation
SELECT
    DATE_TRUNC('month', transaction_at) AS month_start,
    SUM(amount) AS monthly_revenue,
    COUNT(*) AS transaction_count,
    ROUND(AVG(amount), 2) AS avg_transaction
FROM transactions
GROUP BY DATE_TRUNC('month', transaction_at)
ORDER BY month_start;
```

**Expected Output:**

```
     month_start      | monthly_revenue | transaction_count | avg_transaction
----------------------+-----------------+-------------------+-----------------
 2026-09-01 00:00:00  |         1000.00 |                 5 |          200.00
```

**Why This Works:** All five transactions fall in September 2026. `DATE_TRUNC('month', ...)` groups them into a single month row. `SUM`, `COUNT`, and `AVG` compute the monthly metrics.

#### Example 2: MySQL — Monthly Report with DATE_FORMAT

```sql
-- MySQL monthly report with formatted label
SELECT
    DATE_FORMAT(transaction_at, '%Y-%m') AS month_label,
    DATE_FORMAT(transaction_at, '%b %Y') AS display_label,
    SUM(amount) AS monthly_revenue,
    COUNT(*) AS transaction_count
FROM transactions
GROUP BY DATE_FORMAT(transaction_at, '%Y-%m'), DATE_FORMAT(transaction_at, '%b %Y')
ORDER BY month_label;
```

**Expected Output:**

```
 month_label | display_label | monthly_revenue | transaction_count
-------------+---------------+-----------------+-------------------
 2026-09     | Sep 2026      |         1000.00 |                 5
```

**Why This Works:** `DATE_FORMAT(..., '%Y-%m')` produces a sortable label, while `DATE_FORMAT(..., '%b %Y')` produces a human-readable label. Both are included in the `GROUP BY` to satisfy MySQL's grouping rules.

### Real-World Cases

- **Finance:** Monthly profit and loss statements, monthly revenue recognition.
- **SaaS:** Monthly recurring revenue (MRR) and monthly churn.
- **E-commerce:** Monthly sales reports and seasonal trend analysis.
- **Marketing:** Monthly campaign performance and lead generation.

### References

- PostgreSQL: date_trunc — https://www.postgresql.org/docs/current/functions-datetime.html
- MySQL: DATE_FORMAT — https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html
- SQL Server: DATETRUNC — https://learn.microsoft.com/en-us/sql/t-sql/functions/datetrunc-transact-sql
- Oracle: TRUNC (date) — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/TRUNC-date.html


## Core Concept 4: Year-over-Year (YoY) Comparison

### Definitions

**Core Definition:** Year-over-Year (YoY) comparison is the practice of comparing a metric in one period to the same period in the previous year, using window functions or self-joins to align annual data.

**Technical Definition:** YoY comparison uses the `LAG()` window function with an offset of 12 (for monthly data) or 4 (for quarterly data) to access the value from the prior year. The `LAG` function provides access to a row at a given physical offset prior to that position, without using a self-join. Alternatively, a self-join on `date + INTERVAL '1 year'` can be used. The YoY growth rate is `((current - previous) / previous) * 100`.

**Beginner-Friendly Explanation:** Year-over-Year comparison answers the question: "Is this year better than last year?" Instead of comparing January to December (which may be affected by seasonality), you compare January 2026 to January 2025, February 2026 to February 2025, and so on. This eliminates seasonal distortion.

### Purposes

- To measure annual growth without seasonal distortion.
- To evaluate business performance against the same period in prior years.
- To identify long-term trends that are not visible in month-over-month comparisons.
- To provide context for quarterly and annual earnings reports.

### Syntax Rules and Structure

#### Complete General Syntax (LAG with Offset 12)

```sql
WITH monthly_metrics AS (
    SELECT
        DATE_TRUNC('month', date_column) AS month,
        SUM(metric) AS metric_value
    FROM table_name
    GROUP BY DATE_TRUNC('month', date_column)
)
SELECT
    month,
    metric_value AS current_value,
    LAG(metric_value, 12) OVER (ORDER BY month) AS prior_year_value,
    ROUND(
        (metric_value - LAG(metric_value, 12) OVER (ORDER BY month)) * 100.0
        / NULLIF(LAG(metric_value, 12) OVER (ORDER BY month), 0),
        2
    ) AS yoy_growth_pct
FROM monthly_metrics
ORDER BY month;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `LAG(metric_value, 12)` | Accesses the value from 12 months ago (the same month in the prior year). |
| `NULLIF(..., 0)` | Prevents division by zero when the prior year value is 0. |
| `ORDER BY month` | Ensures the rows are in chronological order. |

#### Complete General Syntax (Self-Join on Date + INTERVAL)

```sql
SELECT
    c.month,
    c.metric_value AS current_value,
    p.metric_value AS prior_year_value,
    ROUND(
        (c.metric_value - p.metric_value) * 100.0 / NULLIF(p.metric_value, 0),
        2
    ) AS yoy_growth_pct
FROM monthly_metrics c
LEFT JOIN monthly_metrics p ON c.month = p.month + INTERVAL '1 year'
ORDER BY c.month;
```

#### Syntax Rules

- **Offset of 12:** For monthly data, the offset is 12. For quarterly data, the offset is 4.
- **Partition awareness:** If the data has multiple dimensions (e.g., multiple products), use `PARTITION BY` to compute YoY within each dimension.
- **Missing months:** `LAG(..., 12)` assumes 12 consecutive months of data. If a month is missing, the 12th row back will not correspond to the same month in the prior year.
- **NULL handling:** The first 12 rows will have `NULL` for `prior_year_value` because there is no row 12 positions back.

#### Constraints and Limitations

- **Data completeness:** If the dataset does not contain a full 12 months of history, YoY comparisons for early months will be `NULL`.
- **Leap years:** February has 29 days in leap years; ensure date arithmetic handles this correctly.
- **Partitioned data:** When partitioning by a dimension (e.g., product), ensure each partition has a complete 12-month history.
- **Version-specific:** `LAG()` requires MySQL 8.0+, PostgreSQL 8.4+, SQL Server 2012+, Oracle 8i+.

### Annotated Code Examples

#### Example 1: PostgreSQL — Monthly YoY Revenue Comparison

```sql
-- Create a table with 2 years of monthly data
CREATE TABLE monthly_revenue (
    month DATE PRIMARY KEY,
    revenue NUMERIC(10,2)
);

INSERT INTO monthly_revenue VALUES
('2025-01-01', 10000), ('2025-02-01', 12000), ('2025-03-01', 11000),
('2025-04-01', 13000), ('2025-05-01', 14000), ('2025-06-01', 15000),
('2025-07-01', 16000), ('2025-08-01', 15500), ('2025-09-01', 16500),
('2025-10-01', 17000), ('2025-11-01', 18000), ('2025-12-01', 20000),
('2026-01-01', 11000), ('2026-02-01', 13500), ('2026-03-01', 12500),
('2026-04-01', 15000), ('2026-05-01', 15500), ('2026-06-01', 17000),
('2026-07-01', 18000), ('2026-08-01', 17500), ('2026-09-01', 19000);

-- YoY comparison
WITH monthly AS (
    SELECT month, revenue FROM monthly_revenue
)
SELECT
    month,
    revenue AS current_revenue,
    LAG(revenue, 12) OVER (ORDER BY month) AS prior_year_revenue,
    ROUND(
        (revenue - LAG(revenue, 12) OVER (ORDER BY month)) * 100.0
        / NULLIF(LAG(revenue, 12) OVER (ORDER BY month), 0),
        2
    ) AS yoy_growth_pct
FROM monthly
WHERE month >= '2026-01-01'
ORDER BY month;
```

**Expected Output:**

```
    month    | current_revenue | prior_year_revenue | yoy_growth_pct
-------------+-----------------+--------------------+----------------
 2026-01-01  |        11000.00 |           10000.00 |          10.00
 2026-02-01  |        13500.00 |           12000.00 |          12.50
 2026-03-01  |        12500.00 |           11000.00 |          13.64
 2026-04-01  |        15000.00 |           13000.00 |          15.38
 2026-05-01  |        15500.00 |           14000.00 |          10.71
 2026-06-01  |        17000.00 |           15000.00 |          13.33
 2026-07-01  |        18000.00 |           16000.00 |          12.50
 2026-08-01  |        17500.00 |           15500.00 |          12.90
 2026-09-01  |        19000.00 |           16500.00 |          15.15
```

**Why This Works:** `LAG(revenue, 12)` accesses the revenue from 12 months prior. The `WHERE month >= '2026-01-01'` filter limits the output to 2026, and the YoY growth percentage shows how much each month grew compared to the same month in 2025.

#### Example 2: SQL Server — YoY with Self-Join

```sql
-- SQL Server YoY with self-join
SELECT
    c.month,
    c.revenue AS current_revenue,
    p.revenue AS prior_year_revenue,
    ROUND(
        (c.revenue - p.revenue) * 100.0 / NULLIF(p.revenue, 0),
        2
    ) AS yoy_growth_pct
FROM monthly_revenue c
LEFT JOIN monthly_revenue p
    ON c.month = DATEADD(year, 1, p.month)
WHERE c.month >= '2026-01-01'
ORDER BY c.month;
```

**Expected Output:**

```
 month       | current_revenue | prior_year_revenue | yoy_growth_pct
-------------+-----------------+--------------------+----------------
 2026-01-01  |        11000.00 |           10000.00 |          10.00
 2026-02-01  |        13500.00 |           12000.00 |          12.50
 ...
```

**Why This Works:** The self-join matches each 2026 month with its 2025 counterpart using `DATEADD(year, 1, p.month)`. This is more portable than `LAG()` for databases that do not support window functions.

### Real-World Cases

- **Retail:** Year-over-year sales comparison for same-store sales analysis.
- **SaaS:** Year-over-year revenue growth for investor reporting.
- **Finance:** Year-over-year expense growth for budget planning.
- **Healthcare:** Year-over-year patient volume and outcome comparisons.

### References

- PostgreSQL: Window Functions (LAG) — https://www.postgresql.org/docs/current/functions-window.html
- SQL Server: LAG (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/functions/lag-transact-sql
- Oracle: LAG — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/LAG.html
- MySQL: LAG — https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html


## Core Concept 5: Month-over-Month (MoM) Comparison

### Definitions

**Core Definition:** Month-over-Month (MoM) comparison is the practice of comparing a metric in one month to the immediately preceding month, using the `LAG()` window function with an offset of 1.

**Technical Definition:** MoM comparison uses `LAG(metric, 1) OVER (ORDER BY month)` to access the prior month's value. The `LAG` function provides access to a row at a given physical offset prior to that position. The MoM change is `current - previous` (absolute) or `((current - previous) / previous) * 100` (percentage). `LEAD()` can be used for forward-looking comparisons.

**Beginner-Friendly Explanation:** Month-over-Month comparison answers the question: "Is this month better than last month?" It measures the sequential change in a metric from one month to the next. Unlike YoY, it does not eliminate seasonality—it reflects both growth and seasonal effects.

### Purposes

- To track sequential monthly performance changes.
- To identify short-term trends and inflection points.
- To monitor the impact of recent changes (pricing, marketing, product launches).
- To provide early signals of acceleration or deceleration.

### Syntax Rules and Structure

#### Complete General Syntax (LAG with Offset 1)

```sql
WITH monthly_metrics AS (
    SELECT
        DATE_TRUNC('month', date_column) AS month,
        SUM(metric) AS metric_value
    FROM table_name
    GROUP BY DATE_TRUNC('month', date_column)
)
SELECT
    month,
    metric_value AS current_value,
    LAG(metric_value, 1) OVER (ORDER BY month) AS previous_month_value,
    metric_value - LAG(metric_value, 1) OVER (ORDER BY month) AS mom_change,
    ROUND(
        (metric_value - LAG(metric_value, 1) OVER (ORDER BY month)) * 100.0
        / NULLIF(LAG(metric_value, 1) OVER (ORDER BY month), 0),
        2
    ) AS mom_growth_pct
FROM monthly_metrics
ORDER BY month;
```

#### Complete General Syntax (LEAD for Forward Comparison)

```sql
SELECT
    month,
    metric_value AS current_value,
    LEAD(metric_value, 1) OVER (ORDER BY month) AS next_month_value,
    LEAD(metric_value, 1) OVER (ORDER BY month) - metric_value AS mom_change
FROM monthly_metrics
ORDER BY month;
```

#### Syntax Rules

- **Offset of 1:** For monthly data, the offset is 1. For weekly data, the offset is 1. For daily data, the offset is 1.
- **PARTITION BY:** If the data has multiple dimensions, use `PARTITION BY` to compute MoM within each dimension.
- **NULL handling:** The first row has `NULL` for `previous_month_value`. Use `COALESCE` to replace it with a default (e.g., 0) if needed.
- **ORDER BY:** The `ORDER BY` clause in the `OVER` specification is mandatory for `LAG` and `LEAD`.

#### Constraints and Limitations

- **Missing months:** If a month has no data, it will not appear in the result, and the `LAG` offset will skip over it. Use a calendar table to fill gaps.
- **Seasonality:** MoM comparisons are affected by seasonality; use YoY for seasonally adjusted comparisons.
- **Negative values:** Growth rates from negative to positive values (or vice versa) are mathematically problematic.
- **Base effect:** A small base month produces large percentage changes.

### Annotated Code Examples

#### Example 1: PostgreSQL — MoM Revenue Comparison

```sql
-- MoM comparison for 2026 monthly revenue
SELECT
    month,
    revenue AS current_revenue,
    LAG(revenue, 1) OVER (ORDER BY month) AS prev_month_revenue,
    ROUND(
        (revenue - LAG(revenue, 1) OVER (ORDER BY month)) * 100.0
        / NULLIF(LAG(revenue, 1) OVER (ORDER BY month), 0),
        2
    ) AS mom_growth_pct
FROM monthly_revenue
WHERE month >= '2026-01-01'
ORDER BY month;
```

**Expected Output:**

```
    month    | current_revenue | prev_month_revenue | mom_growth_pct
-------------+-----------------+--------------------+----------------
 2026-01-01  |        11000.00 |           20000.00 |         -45.00
 2026-02-01  |        13500.00 |           11000.00 |          22.73
 2026-03-01  |        12500.00 |           13500.00 |          -7.41
 2026-04-01  |        15000.00 |           12500.00 |          20.00
 2026-05-01  |        15500.00 |           15000.00 |           3.33
 2026-06-01  |        17000.00 |           15500.00 |           9.68
 2026-07-01  |        18000.00 |           17000.00 |           5.88
 2026-08-01  |        17500.00 |           18000.00 |          -2.78
 2026-09-01  |        19000.00 |           17500.00 |           8.57
```

**Why This Works:** `LAG(revenue, 1)` accesses the previous month's revenue. The `WHERE` filter limits the output to 2026, but the `LAG` function still sees the December 2025 value (20000) for the January 2026 calculation. The MoM growth percentage shows the sequential change.

#### Example 2: MySQL — MoM with CTE

```sql
-- MySQL MoM comparison
WITH monthly AS (
    SELECT
        DATE_FORMAT(transaction_at, '%Y-%m') AS month,
        SUM(amount) AS revenue
    FROM transactions
    GROUP BY DATE_FORMAT(transaction_at, '%Y-%m')
)
SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_month_revenue,
    ROUND(
        (revenue - LAG(revenue) OVER (ORDER BY month)) * 100.0
        / NULLIF(LAG(revenue) OVER (ORDER BY month), 0),
        2
    ) AS mom_growth_pct
FROM monthly
ORDER BY month;
```

**Expected Output:**

```
 month   | revenue | prev_month_revenue | mom_growth_pct
---------+---------+--------------------+----------------
 2026-09 | 1000.00 |             (null) |         (null)
```

**Why This Works:** The CTE aggregates revenue by month. `LAG(revenue)` (with default offset 1) accesses the previous month. Since there is only one month of data, the previous month value is `NULL`.

### Real-World Cases

- **SaaS:** Monthly recurring revenue (MRR) growth rate.
- **E-commerce:** Month-over-month sales growth.
- **Finance:** Monthly expense variance analysis.
- **Marketing:** Monthly lead generation and conversion trends.

### References

- PostgreSQL: Window Functions (LAG) — https://www.postgresql.org/docs/current/functions-window.html
- MySQL: Window Function Descriptions (LAG) — https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html
- SQL Server: LAG (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/functions/lag-transact-sql
- Oracle: LAG — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/LAG.html


## Core Concept 6: Rolling Averages

### Definitions

**Core Definition:** A rolling average (moving average) is the average of a fixed number of consecutive data points, computed by defining a window frame that slides over the data as each row is processed.

**Technical Definition:** Rolling averages use `AVG(metric) OVER (ORDER BY date ROWS BETWEEN n PRECEDING AND CURRENT ROW)` to compute the average of the current row and the `n` preceding rows. The frame clause (`ROWS BETWEEN ... AND ...`) defines the subset of the partition used for the calculation. By defining a frame as extending N rows on either side of the current row, you can compute rolling averages. `RANGE BETWEEN` can be used instead of `ROWS` for value-based or interval-based windows (e.g., a true 7-day trailing window regardless of row count).

**Beginner-Friendly Explanation:** A rolling average is like a moving average in stock trading. Instead of looking at just today's value (which might be noisy), you look at the average of the last 7 days. Each day, the window slides forward, dropping the oldest day and adding the newest. This smooths out short-term fluctuations and reveals underlying trends.

### Purposes

- To smooth out noise and short-term fluctuations in time-series data.
- To identify underlying trends that are obscured by daily volatility.
- To detect anomalies by comparing current values to the rolling average.
- To provide a more stable metric for forecasting and decision-making.

### Syntax Rules and Structure

#### Complete General Syntax

```sql
AVG(metric_column) OVER (
    [PARTITION BY partition_column]
    ORDER BY date_column
    ROWS BETWEEN n PRECEDING AND CURRENT ROW
) AS rolling_avg
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `AVG(metric_column)` | The aggregate function to apply. |
| `PARTITION BY` | Optional; divides rows into groups. |
| `ORDER BY date_column` | Determines the order of rows in the window. |
| `ROWS BETWEEN n PRECEDING AND CURRENT ROW` | Includes the current row and the `n` preceding rows. |
| `ROWS BETWEEN n PRECEDING AND n FOLLOWING` | Centered rolling average (includes `n` rows before and after). |

#### Complete General Syntax (RANGE for Interval-Based Windows)

```sql
AVG(metric_column) OVER (
    ORDER BY date_column
    RANGE BETWEEN INTERVAL '7 days' PRECEDING AND CURRENT ROW
) AS rolling_7day_avg
```

#### Syntax Rules

- **ROWS vs. RANGE:** `ROWS` uses physical row offsets; `RANGE` uses value-based or interval-based offsets. Use `RANGE BETWEEN INTERVAL` for a true trailing-time-window average (all readings in the past 7 days regardless of row count).
- **Default frame:** When an `ORDER BY` is specified, the default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. For `AVG`, this produces a cumulative average, not a rolling average. Always specify the frame explicitly.
- **Centered average:** `ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING` includes the current row and one row before and after it.
- **Trailing average:** `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` includes the current row and the 6 preceding rows (a 7-row window).

#### Constraints and Limitations

- **Boundary behavior:** For the first `n` rows, the window does not have `n` preceding rows; the average is computed over the available rows. This can produce misleading values at the start of the series.
- **Missing data:** `ROWS` skips missing rows; `RANGE` includes rows based on value, so missing dates may produce different results.
- **Performance:** Rolling averages require sorting and window processing; large datasets may benefit from pre-aggregated data.

### Annotated Code Examples

#### Example 1: PostgreSQL — 7-Day Rolling Average

```sql
-- Create a daily metrics table
CREATE TABLE daily_metrics (
    metric_date DATE PRIMARY KEY,
    metric_value NUMERIC(10,2)
);

INSERT INTO daily_metrics VALUES
('2026-09-01', 100), ('2026-09-02', 110), ('2026-09-03', 105),
('2026-09-04', 120), ('2026-09-05', 115), ('2026-09-06', 130),
('2026-09-07', 125), ('2026-09-08', 140), ('2026-09-09', 135),
('2026-09-10', 150);

-- 3-day and 7-day rolling averages
SELECT
    metric_date,
    metric_value,
    ROUND(AVG(metric_value) OVER (
        ORDER BY metric_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ), 2) AS rolling_3day_avg,
    ROUND(AVG(metric_value) OVER (
        ORDER BY metric_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ), 2) AS rolling_7day_avg
FROM daily_metrics
ORDER BY metric_date;
```

**Expected Output:**

```
 metric_date | metric_value | rolling_3day_avg | rolling_7day_avg
-------------+--------------+------------------+------------------
 2026-09-01  |       100.00 |           100.00 |           100.00
 2026-09-02  |       110.00 |           105.00 |           105.00
 2026-09-03  |       105.00 |           105.00 |           105.00
 2026-09-04  |       120.00 |           111.67 |           108.75
 2026-09-05  |       115.00 |           113.33 |           110.00
 2026-09-06  |       130.00 |           121.67 |           113.33
 2026-09-07  |       125.00 |           123.33 |           115.00
 2026-09-08  |       140.00 |           131.67 |           119.29
 2026-09-09  |       135.00 |           133.33 |           122.86
 2026-09-10  |       150.00 |           141.67 |           130.71
```

**Why This Works:** The 3-day rolling average includes the current row and two preceding rows. The 7-day rolling average includes the current row and six preceding rows. For the first two rows, the 3-day average is computed over fewer rows (1 and 2 rows, respectively).

#### Example 2: PostgreSQL — Centered Rolling Average

```sql
-- Centered 3-day rolling average
SELECT
    metric_date,
    metric_value,
    ROUND(AVG(metric_value) OVER (
        ORDER BY metric_date
        ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
    ), 2) AS centered_3day_avg
FROM daily_metrics
ORDER BY metric_date;
```

**Expected Output:**

```
 metric_date | metric_value | centered_3day_avg
-------------+--------------+-------------------
 2026-09-01  |       100.00 |            105.00
 2026-09-02  |       110.00 |            105.00
 2026-09-03  |       105.00 |            111.67
 2026-09-04  |       120.00 |            113.33
 2026-09-05  |       115.00 |            121.67
 2026-09-06  |       130.00 |            123.33
 2026-09-07  |       125.00 |            131.67
 2026-09-08  |       140.00 |            133.33
 2026-09-09  |       135.00 |            141.67
 2026-09-10  |       150.00 |            142.50
```

**Why This Works:** `ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING` includes the current row plus one row before and after. For the first and last rows, the window is truncated (one side is missing), so the average is computed over two rows instead of three.

### Real-World Cases

- **Finance:** 50-day and 200-day moving averages for stock prices.
- **IoT:** 7-day rolling average of sensor readings for anomaly detection.
- **DevOps:** 7-day rolling average of error rates to smooth out daily spikes.
- **E-commerce:** 30-day rolling average of daily sales to identify trends.

### References

- PostgreSQL: Window Function Calls (Frame Specification) — https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-WINDOW-FUNCTIONS
- MySQL: Window Function Frame Specification — https://dev.mysql.com/doc/refman/8.0/en/window-functions-frames.html
- SQL Server: OVER Clause (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql
- Oracle: Analytic Functions — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/Analytic-Functions.html


## Core Concept 7: Cumulative Totals

### Definitions

**Core Definition:** A cumulative total (running total, year-to-date sum) is the sum of all values from the start of a partition up to the current row, computed using a window frame that starts at the beginning of the partition and ends at the current row.

**Technical Definition:** Cumulative totals use `SUM(metric) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)`. The `UNBOUNDED PRECEDING` keyword specifies that the window starts at the first row of the partition. When an `ORDER BY` is specified and no frame is specified, the default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, which produces a cumulative total. The result is a running sum that increases with each row.

**Beginner-Friendly Explanation:** A cumulative total is like a running tally. If you are tracking daily sales, the cumulative total tells you how much you have sold so far this month or this year. Each day, the total increases by that day's sales.

### Purposes

- To compute year-to-date (YTD), quarter-to-date (QTD), or month-to-date (MTD) totals.
- To track running balances (e.g., bank account balances, inventory levels).
- To compute cumulative growth or cumulative returns.
- To provide context for period-specific metrics (e.g., "we have sold $500K YTD").

### Syntax Rules and Structure

#### Complete General Syntax

```sql
SUM(metric_column) OVER (
    [PARTITION BY partition_column]
    ORDER BY date_column
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
) AS cumulative_total
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `SUM(metric_column)` | The aggregate function to apply. |
| `PARTITION BY` | Optional; resets the cumulative total for each partition (e.g., each year). |
| `ORDER BY date_column` | Determines the order in which rows are added. |
| `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` | Includes all rows from the start of the partition to the current row. |
| `UNBOUNDED PRECEDING` | The window starts at the first row of the partition. |

#### Complete General Syntax (Year-to-Date with Partition)

```sql
SUM(metric_column) OVER (
    PARTITION BY EXTRACT(YEAR FROM date_column)
    ORDER BY date_column
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
) AS ytd_total
```

#### Syntax Rules

- **Default frame:** When `ORDER BY` is specified and no frame is specified, the default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, which produces a cumulative total. However, it is best practice to specify the frame explicitly.
- **PARTITION BY for reset:** Use `PARTITION BY` to reset the cumulative total for each group (e.g., each year, each customer, each product).
- **Unbounded preceding:** `UNBOUNDED PRECEDING` means the window starts at the first row of the partition, regardless of how many rows there are.
- **Cumulative average:** Replace `SUM` with `AVG` to compute a cumulative average (running average of all rows up to the current row).

#### Constraints and Limitations

- **Performance:** Cumulative totals require sorting the entire partition; large datasets may benefit from pre-aggregated data.
- **Missing rows:** If a date is missing, the cumulative total skips it; the next row continues from the previous total.
- **NULL values:** `SUM` ignores NULL values; if all values are NULL, the cumulative total is NULL.
- **Partition size:** Very large partitions can consume significant memory during sorting.

### Annotated Code Examples

#### Example 1: PostgreSQL — Cumulative Sum (Running Total)

```sql
-- Cumulative sum of daily metrics
SELECT
    metric_date,
    metric_value,
    SUM(metric_value) OVER (
        ORDER BY metric_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM daily_metrics
ORDER BY metric_date;
```

**Expected Output:**

```
 metric_date | metric_value | running_total
-------------+--------------+---------------
 2026-09-01  |       100.00 |        100.00
 2026-09-02  |       110.00 |        210.00
 2026-09-03  |       105.00 |        315.00
 2026-09-04  |       120.00 |        435.00
 2026-09-05  |       115.00 |        550.00
 2026-09-06  |       130.00 |        680.00
 2026-09-07  |       125.00 |        805.00
 2026-09-08  |       140.00 |        945.00
 2026-09-09  |       135.00 |       1080.00
 2026-09-10  |       150.00 |       1230.00
```

**Why This Works:** `SUM(metric_value) OVER (ORDER BY metric_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)` computes the cumulative sum from the first row to the current row. Each row's `running_total` is the sum of all values up to and including that row.

#### Example 2: PostgreSQL — Year-to-Date with Partition

```sql
-- Year-to-date cumulative sum
CREATE TABLE yearly_metrics (
    metric_date DATE,
    metric_value NUMERIC(10,2)
);

INSERT INTO yearly_metrics VALUES
('2025-12-01', 100), ('2025-12-02', 110),
('2026-01-01', 200), ('2026-01-02', 210), ('2026-01-03', 220),
('2026-02-01', 300), ('2026-02-02', 310);

-- YTD cumulative sum (resets each year)
SELECT
    metric_date,
    metric_value,
    SUM(metric_value) OVER (
        PARTITION BY EXTRACT(YEAR FROM metric_date)
        ORDER BY metric_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS ytd_total
FROM yearly_metrics
ORDER BY metric_date;
```

**Expected Output:**

```
 metric_date | metric_value | ytd_total
-------------+--------------+-----------
 2025-12-01  |       100.00 |    100.00
 2025-12-02  |       110.00 |    210.00
 2026-01-01  |       200.00 |    200.00
 2026-01-02  |       210.00 |    410.00
 2026-01-03  |       220.00 |    630.00
 2026-02-01  |       300.00 |    930.00
 2026-02-02  |       310.00 |   1240.00
```

**Why This Works:** The `PARTITION BY EXTRACT(YEAR FROM metric_date)` clause resets the cumulative sum at the start of each year. The 2025 rows accumulate separately from the 2026 rows. The 2026 cumulative total starts at 200 on January 1 and reaches 1240 by February 2.

### Real-World Cases

- **Finance:** Year-to-date revenue, expense, and profit tracking.
- **Banking:** Running account balances.
- **Inventory:** Cumulative inventory levels (receipts minus shipments).
- **Sales:** Month-to-date and quarter-to-date sales targets.

### References

- PostgreSQL: Window Function Calls (Frame Specification) — https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-WINDOW-FUNCTIONS
- MySQL: Window Function Frame Specification — https://dev.mysql.com/doc/refman/8.0/en/window-functions-frames.html
- SQL Server: OVER Clause (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql
- Oracle: Analytic Functions — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/Analytic-Functions.html


## Summary Table: Time-Series Analysis Techniques

| Technique | Primary Constructs | Key Considerations |
|-----------|-------------------|-------------------|
| **Daily Aggregation** | `DATE_TRUNC('day', ts)`, `DATE(ts)`, `CAST(ts AS DATE)`, `TRUNC(ts)` | Time zone boundaries |
| **Weekly Aggregation** | `DATE_TRUNC('week')`, `DATETRUNC(iso_week)`, `EXTRACT(WEEK)`, `DATEPART(ISO_WEEK)` | ISO vs. calendar weeks; `@@DATEFIRST` |
| **Monthly Aggregation** | `DATE_TRUNC('month')`, `DATE_FORMAT(ts, '%Y-%m')`, `TRUNC(dt, 'MM')` | Month label sorting |
| **Year-over-Year** | `LAG(metric, 12)` or self-join on `date + INTERVAL '1 year'` | Data completeness; leap years |
| **Month-over-Month** | `LAG(metric, 1)` or `LEAD(metric, 1)` | Seasonality; base effect |
| **Rolling Averages** | `AVG() OVER (ROWS BETWEEN n PRECEDING AND CURRENT ROW)` | `ROWS` vs. `RANGE`; boundary behavior |
| **Cumulative Totals** | `SUM() OVER (ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)` | `PARTITION BY` for reset |


## Final Notes on Deprecated and Unsafe Features

- **`DATE_TRUNC` in SQL Server:** `DATE_TRUNC` does not exist in SQL Server; use `DATETRUNC` (SQL Server 2022+) or `CAST(... AS DATE)`. `DATETRUNC` is not available in earlier versions.
- **`DATEPART(week, ...)` in SQL Server:** Depends on the `@@DATEFIRST` setting, which can vary by session. Use `DATEPART(ISO_WEEK, ...)` for consistent Monday-start weeks.
- **`EXTRACT(WEEK)` without `ISOYEAR`:** Using `EXTRACT(WEEK)` alone can produce ambiguous results at year boundaries (e.g., January 1 might be week 53 of the prior year). Always pair with `EXTRACT(ISOYEAR)`.
- **Default window frame:** When `ORDER BY` is specified and no frame is specified, the default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. For `AVG`, this produces a cumulative average, not a rolling average. Always specify the frame explicitly for rolling calculations.
- **`ROWS` vs. `RANGE`:** `ROWS` uses physical row offsets; `RANGE` uses value-based offsets. For time-series data with missing dates, `RANGE BETWEEN INTERVAL` produces a true trailing-time-window average, while `ROWS` produces a fixed-row-count window.
- **`LAG` offset assumptions:** `LAG(metric, 12)` assumes 12 consecutive months of data. If months are missing, the offset will not correspond to the same month in the prior year.
- **Version-specific:** `LAG()` requires MySQL 8.0+, PostgreSQL 8.4+, SQL Server 2012+, Oracle 8i+. `DATETRUNC` requires SQL Server 2022+. `DATE_TRUNC` requires PostgreSQL 8.0+.
- **Time zone alignment:** Always ensure timestamps are in a consistent time zone before performing time-series aggregation. Use `AT TIME ZONE` (PostgreSQL, SQL Server) to convert to a standard zone.