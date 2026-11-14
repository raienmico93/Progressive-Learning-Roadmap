# SQL Enterprise Analytical Patterns & Problem Solving: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Enterprise analytical patterns are reusable SQL query structures that solve recurring business problems involving ranking, sequence analysis, event grouping, time-series comparisons, and customer lifecycle metrics.

**Technical Definition:** These patterns leverage window functions, common table expressions (CTEs), and advanced joins to solve problems that would otherwise require procedural code or multiple query passes. They include top-N-per-group ranking, gaps-and-islands detection, sessionization, period-over-period analysis, and cohort-based retention/LTV modeling .

**Beginner-Friendly Explanation:** Think of these as "recipes" for common business questions. "Who are the top 3 salespeople per region?" "When did users stop logging in for a while?" "How many customers from January are still buying in June?" Each recipe uses the same building blocks — window functions, CTEs, and date logic — arranged in a proven pattern.

### Key Characteristics

- **Window-function-centric:** Most patterns rely on `ROW_NUMBER`, `RANK`, `LAG`, `LEAD`, and running `SUM` .
- **CTE-driven:** Multi-step logic is decomposed into readable CTEs that build on each other .
- **Set-based:** Avoids cursors and procedural loops; solves problems in a single query pass where possible.
- **Composable:** Patterns can be combined (e.g., sessionization + aggregation + cohort analysis).
- **Dialect-aware:** Core concepts are standard, but date arithmetic and frame syntax vary by RDBMS .

### Prerequisites

- **Window function fundamentals:** `OVER`, `PARTITION BY`, `ORDER BY`, frames.
- **CTE syntax:** `WITH name AS (...) SELECT ...`.
- **Date/time functions:** `DATE_TRUNC`, `DATEDIFF`, `INTERVAL`, `EXTRACT`.
- **Join and aggregation basics:** `INNER JOIN`, `LEFT JOIN`, `GROUP BY`.

### Related Programming Areas

- **Product Analytics:** Sessionization, user journeys, engagement metrics.
- **Growth/Marketing Analytics:** Cohort retention, churn, LTV .
- **Financial Analytics:** Period-over-period comparisons, running totals.
- **Operations Analytics:** Gap detection, sequence validation, SLA monitoring.

### Core Concepts / Features

1. Top-N-per-Group Problems
2. Gaps and Islands
3. Sessionization
4. Time-Series Analysis
5. User Lifecycle Metrics

---

## 1. Top-N-per-Group Problems

### Definitions

**Core Definition:** The top-N-per-group pattern retrieves the highest, lowest, or most recent N rows within each category, using ranking functions to assign a position and then filtering on that position.

**Technical Definition:** A `ROW_NUMBER()` (or `RANK()`/`DENSE_RANK()`) window function assigns a sequential number to each row within a partition defined by `PARTITION BY`. An outer query or CTE filters for `rn <= N`. The choice between `ROW_NUMBER`, `RANK`, and `DENSE_RANK` determines tie behavior .

**Beginner-Friendly Explanation:** Imagine a leaderboard for each department. You want the top 3 performers per department. You rank everyone within their department, then keep only ranks 1, 2, and 3.

### Purposes

- To find the top N products by revenue per category .
- To retrieve the most recent N orders per customer .
- To identify the highest-paid employees per department.
- To support "best in class" reporting and leaderboards.

### Syntax Rules and Structure

```sql
WITH ranked AS (
    SELECT
        group_column,
        value_column,
        ROW_NUMBER() OVER (
            PARTITION BY group_column
            ORDER BY value_column DESC
        ) AS rn
    FROM source_table
)
SELECT * FROM ranked WHERE rn <= N;
```

**Component Breakdown:**

| Function | Tie Behavior | Use When |
|----------|-------------|----------|
| `ROW_NUMBER()` | Arbitrary tiebreak | Exactly N rows needed  |
| `RANK()` | Same rank, gaps | Ties should share rank |
| `DENSE_RANK()` | Same rank, no gaps | Ties should share rank, no gaps |

**Syntax Rules:**

- `PARTITION BY` defines the group.
- `ORDER BY` defines the ranking criterion.
- Filter on the rank in an outer query; window functions cannot be filtered directly in `WHERE` .
- Add secondary sort keys to `ORDER BY` for deterministic tie-breaking .

**Constraints and Limitations:**

- `ROW_NUMBER` gives non-deterministic results when ties exist unless the `ORDER BY` includes a unique tiebreaker.
- `RANK` and `DENSE_RANK` may return more than N rows when ties exist .

### Annotated Complete Code Examples

**Example 1: Top 3 Products per Category**

```sql
-- Setup
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    category   VARCHAR(50),
    product_name VARCHAR(100),
    revenue    DECIMAL(12, 2)
);

INSERT INTO products VALUES
    (1, 'Electronics', 'Laptop',   120000),
    (2, 'Electronics', 'Phone',     95000),
    (3, 'Electronics', 'Tablet',    60000),
    (4, 'Clothing',    'Jacket',    30000),
    (5, 'Clothing',    'Jeans',     22000),
    (6, 'Books',       'Novel A',    5000),
    (7, 'Books',       'Novel B',    4200);

-- Query: Top 2 products per category by revenue
WITH ranked AS (
    SELECT
        product_id,
        category,
        product_name,
        revenue,
        ROW_NUMBER() OVER (
            PARTITION BY category
            ORDER BY revenue DESC
        ) AS rn
    FROM products
)
SELECT product_id, category, product_name, revenue
FROM ranked
WHERE rn <= 2
ORDER BY category, rn;

-- Expected Output:
--  product_id | category    | product_name | revenue
-- ------------+-------------+--------------+---------
--           1 | Electronics | Laptop       | 120000
--           2 | Electronics | Phone        |  95000
--           4 | Clothing    | Jacket       |  30000
--           5 | Clothing    | Jeans        |  22000
--           6 | Books       | Novel A      |   5000
--           7 | Books       | Novel B      |   4200
```

**Why this output occurs:** `ROW_NUMBER` assigns rank 1 to the highest-revenue product in each category, rank 2 to the second, and so on. The outer query filters for ranks 1 and 2, returning exactly two products per category .

### Real-World Cases

**Case 1: Sales Leaderboard**

A sales manager uses `ROW_NUMBER() OVER (PARTITION BY region ORDER BY revenue DESC)` to identify the top 3 salespeople per region for quarterly bonuses.

**Case 2: Most Recent Orders**

An e-commerce support team retrieves the 5 most recent orders per customer to investigate complaints: `ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC)` .

### References

- MySQL Top N per Group with Window Functions 
- How to Get the Top N Records per Group in MySQL 

---

## 2. Gaps and Islands

### Definitions

**Core Definition:** Gaps-and-islands is a pattern that identifies contiguous runs of sequential values (islands) and the missing values between them (gaps) in a dataset.

**Technical Definition:** The island pattern uses the difference between a sequence column and `ROW_NUMBER()` to create a constant group key for each consecutive run. The gap pattern uses `LEAD()` or `LAG()` to compare each row with the next, detecting jumps greater than the expected interval .

**Beginner-Friendly Explanation:** Imagine a list of login dates: Jan 1, 2, 3, 6, 7, 10. The "islands" are Jan 1-3, Jan 6-7, and Jan 10. The "gaps" are Jan 4-5 and Jan 8-9. This pattern finds both.

### Purposes

- To detect missing invoice numbers or dropped connections .
- To identify consecutive login streaks or attendance runs.
- To find holes in sequences for data quality auditing.
- To validate that no records are missing from a batch.

### Syntax Rules and Structure

**Islands (Consecutive Runs):**

```sql
WITH numbered AS (
    SELECT
        sequence_column,
        sequence_column - ROW_NUMBER() OVER (ORDER BY sequence_column) AS grp
    FROM table_name
)
SELECT
    MIN(sequence_column) AS run_start,
    MAX(sequence_column) AS run_end,
    COUNT(*) AS run_length
FROM numbered
GROUP BY grp
ORDER BY run_start;
```

**Gaps (Missing Values):**

```sql
SELECT
    sequence_column + 1 AS gap_start,
    next_value - 1 AS gap_end
FROM (
    SELECT
        sequence_column,
        LEAD(sequence_column) OVER (ORDER BY sequence_column) AS next_value
    FROM table_name
) t
WHERE next_value - sequence_column > 1;
```

**Component Breakdown:**

| Pattern | Technique | Key Insight |
|---------|-----------|-------------|
| Island | `value - ROW_NUMBER()` | Constant within consecutive runs  |
| Gap | `LEAD()` comparison | Jump > expected interval indicates gap  |

**Syntax Rules:**

- For dates, subtract `ROW_NUMBER()` days from the date: `login_date - ROW_NUMBER() OVER (ORDER BY login_date)::int` .
- In SQLite, use `julianday(date) - ROW_NUMBER()` .
- Add `PARTITION BY user_id` for per-user analysis .

**Constraints and Limitations:**

- The `value - ROW_NUMBER()` trick requires an ordered, numeric sequence.
- For dates, the subtraction result is a date; group by that date.
- Gaps may be at the beginning or end of the sequence and won't be detected by `LEAD`/`LAG` alone.

### Annotated Complete Code Examples

**Example 1: Consecutive Login Streaks**

```sql
-- Setup: Login dates
CREATE TABLE logins (login_day INTEGER);
INSERT INTO logins VALUES (1), (2), (3), (6), (7), (10);

-- Query: Find consecutive runs
WITH numbered AS (
    SELECT
        login_day,
        login_day - ROW_NUMBER() OVER (ORDER BY login_day) AS grp
    FROM logins
)
SELECT
    MIN(login_day) AS run_start,
    MAX(login_day) AS run_end,
    COUNT(*) AS run_length
FROM numbered
GROUP BY grp
ORDER BY run_start;

-- Expected Output:
--  run_start | run_end | run_length
-- -----------+---------+------------
--          1 |       3 |          3
--          6 |       7 |          2
--         10 |      10 |          1
```

**Why this output occurs:** The `login_day - ROW_NUMBER()` value is constant within each consecutive run: for days 1,2,3 it's 0; for 6,7 it's 2; for 10 it's 4. Grouping by this value collects each run together .

**Example 2: Finding Gaps**

```sql
-- Query: Find missing days
SELECT
    login_day + 1 AS gap_start,
    next_day - 1 AS gap_end
FROM (
    SELECT
        login_day,
        LEAD(login_day) OVER (ORDER BY login_day) AS next_day
    FROM logins
) t
WHERE next_day - login_day > 1;

-- Expected Output:
--  gap_start | gap_end
-- -----------+---------
--          4 |       5
--          8 |       9
```

**Why this output occurs:** `LEAD` retrieves the next day's value. When the difference is greater than 1, there's a gap. Day 3 jumps to day 6 (gap 4-5); day 7 jumps to day 10 (gap 8-9) .

### Real-World Cases

**Case 1: Invoice Sequence Validation**

An accounting system uses gaps-and-islands to verify that invoice numbers are sequential and detect missing invoices, flagging them for investigation .

**Case 2: User Engagement Streaks**

A fitness app uses island detection to count consecutive days a user logged a workout, awarding streak badges.

### References

- Advanced Window Functions — Gaps and Islands 
- Advanced Patterns: Gaps-and-Islands, Sessionization & Pivot 
- Gaps-and-Islands Lecture Notes 

---

## 3. Sessionization

### Definitions

**Core Definition:** Sessionization is the process of grouping a stream of user events into discrete sessions, where a session ends after a period of inactivity exceeding a threshold.

**Technical Definition:** Sessionization uses `LAG()` to measure the time gap between consecutive events for each user. A new session flag is set when the gap exceeds the threshold or when the event is the first for the user. A running `SUM()` over these flags assigns a session ID. Grouping by user and session ID produces session-level metrics .

**Beginner-Friendly Explanation:** Imagine someone clicking around a website. They click, click, click, then go to lunch for an hour, then come back and click more. The clicks before lunch are one session; the clicks after are a new session.

### Purposes

- To measure engagement: session duration, pages per session, bounce rate .
- To analyze user journeys and drop-off points.
- To attribute conversions to sessions.
- To understand how users navigate through a product .

### Syntax Rules and Structure

```sql
WITH gaps AS (
    SELECT
        user_id,
        event_time,
        event_time - LAG(event_time) OVER (
            PARTITION BY user_id ORDER BY event_time
        ) AS since_prev
    FROM events
),
flagged AS (
    SELECT
        user_id,
        event_time,
        CASE
            WHEN since_prev IS NULL THEN 1
            WHEN since_prev > INTERVAL '30 minutes' THEN 1
            ELSE 0
        END AS is_new_session
    FROM gaps
),
sessionized AS (
    SELECT
        user_id,
        event_time,
        SUM(is_new_session) OVER (
            PARTITION BY user_id ORDER BY event_time
        ) AS session_id
    FROM flagged
)
SELECT
    user_id,
    session_id,
    MIN(event_time) AS session_start,
    MAX(event_time) AS session_end,
    COUNT(*) AS events_in_session
FROM sessionized
GROUP BY user_id, session_id
ORDER BY user_id, session_start;
```

**Component Breakdown:**

| Step | Function | Purpose |
|------|----------|---------|
| 1 | `LAG()` | Measure gap to previous event  |
| 2 | `CASE` | Flag session starts (gap > threshold) |
| 3 | `SUM()` | Assign cumulative session ID |
| 4 | `GROUP BY` | Aggregate per session  |

**Syntax Rules:**

- `PARTITION BY user_id` ensures sessions are per-user.
- The threshold (e.g., 30 minutes) is configurable .
- The first event for each user is always a session start (`since_prev IS NULL`).

**Constraints and Limitations:**

- Requires event data with timestamps and user identifiers.
- Threshold choice affects session granularity; 30 minutes is a common web analytics default .
- Sessions spanning midnight are not split unless explicitly handled.

### Annotated Complete Code Examples

**Example 1: Sessionizing Clickstream Events**

```sql
-- Setup
CREATE TABLE events (
    id          INTEGER PRIMARY KEY,
    user_id     INTEGER,
    event_time  TIMESTAMP
);

INSERT INTO events VALUES
    (1, 1, '2026-06-01 10:00'),
    (2, 1, '2026-06-01 10:10'),
    (3, 1, '2026-06-01 10:20'),
    (4, 1, '2026-06-01 11:05'),   -- 45-min gap → new session
    (5, 1, '2026-06-01 11:15');

-- Query: Sessionize with 30-min threshold
WITH gaps AS (
    SELECT
        user_id,
        event_time,
        event_time - LAG(event_time) OVER (
            PARTITION BY user_id ORDER BY event_time
        ) AS since_prev
    FROM events
),
flagged AS (
    SELECT
        user_id,
        event_time,
        CASE
            WHEN since_prev IS NULL THEN 1
            WHEN since_prev > INTERVAL '30 minutes' THEN 1
            ELSE 0
        END AS is_new_session
    FROM gaps
),
sessionized AS (
    SELECT
        user_id,
        event_time,
        SUM(is_new_session) OVER (
            PARTITION BY user_id ORDER BY event_time
        ) AS session_id
    FROM flagged
)
SELECT
    user_id,
    session_id,
    MIN(event_time) AS session_start,
    MAX(event_time) AS session_end,
    COUNT(*) AS n_events
FROM sessionized
GROUP BY user_id, session_id
ORDER BY user_id, session_id;

-- Expected Output:
--  user_id | session_id | session_start       | session_end         | n_events
-- ---------+------------+---------------------+---------------------+----------
--        1 |          1 | 2026-06-01 10:00:00 | 2026-06-01 10:20:00 |        3
--        1 |          2 | 2026-06-01 11:05:00 | 2026-06-01 11:15:00 |        2
```

**Why this output occurs:** The first event has `since_prev = NULL`, so it starts session 1. Events 2 and 3 have gaps under 30 minutes, so they stay in session 1. Event 4 has a 45-minute gap, so `is_new_session = 1`, incrementing the session ID to 2. Event 5 stays in session 2 .

### Real-World Cases

**Case 1: Product Analytics**

A product team uses sessionization to compute daily active sessions, average session duration, and bounce rate (sessions with one event) .

**Case 2: Phone Call Grouping**

A telecom company groups dropped calls into a single phone session if the redial gap is within 60 seconds, using Oracle's `MATCH_RECOGNIZE` .

### References

- Advanced Window Functions — Sessionization 
- SQL Window Functions — Sessionization 
- Sessionize an Event Stream Challenge 
- ClickHouse Session Analytics 
- Oracle Data Warehousing Guide — Sessionization 

---

## 4. Time-Series Analysis

### Definitions

**Core Definition:** Time-series analysis patterns compute period-over-period comparisons (month-over-month, year-over-year), rolling aggregates, and trend metrics from time-stamped data.

**Technical Definition:** `LAG()` retrieves the value from a previous period for MoM/YoY calculations. `AVG() OVER (ORDER BY date RANGE BETWEEN INTERVAL '6 days' PRECEDING AND CURRENT ROW)` computes rolling averages. `SUM() OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)` computes rolling sums .

**Beginner-Friendly Explanation:** "Is this month's revenue better than last month's?" "What's the 7-day average of daily sales?" "Is this year's sales up compared to the same month last year?" These patterns answer those questions.

### Purposes

- To calculate month-over-month (MoM) and year-over-year (YoY) growth rates .
- To smooth noisy data with rolling averages .
- To compute year-to-date (YTD) cumulative metrics .
- To detect trends and seasonality.

### Syntax Rules and Structure

**Period-over-Period (LAG):**

```sql
SELECT
    month,
    revenue,
    LAG(revenue, 1) OVER (ORDER BY month) AS prev_month_revenue,
    LAG(revenue, 12) OVER (ORDER BY month) AS prev_year_revenue,
    (revenue - LAG(revenue, 1) OVER (ORDER BY month))
        / NULLIF(LAG(revenue, 1) OVER (ORDER BY month), 0) AS mom_growth
FROM monthly_sales;
```

**Rolling Average (RANGE):**

```sql
SELECT
    day,
    revenue,
    AVG(revenue) OVER (
        ORDER BY day
        RANGE BETWEEN INTERVAL '6 days' PRECEDING AND CURRENT ROW
    ) AS rolling_7d_avg
FROM daily_sales;
```

**Component Breakdown:**

| Pattern | Function | Key Detail |
|---------|----------|------------|
| MoM/YoY | `LAG(value, N)` | N=1 for MoM, N=12 for YoY  |
| Rolling Avg | `AVG() OVER RANGE` | RANGE handles non-consecutive dates  |
| Rolling Sum | `SUM() OVER ROWS` | ROWS for physical row count  |
| YTD | `SUM() OVER (PARTITION BY year ORDER BY date)` | Resets each year  |

**Syntax Rules:**

- `LAG(value, 1)` gets the previous row; `LAG(value, 12)` gets the same month last year (if monthly data) .
- `RANGE BETWEEN INTERVAL '6 days' PRECEDING AND CURRENT ROW` includes all rows within 6 days (7 days total) .
- `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` includes exactly 7 physical rows.
- Use a date spine to fill missing periods, or `LAG` will compare wrong periods .

**Constraints and Limitations:**

- `LAG` is row-based, not period-based. Missing periods cause incorrect comparisons .
- `RANGE` with intervals requires a date/time `ORDER BY` column.
- Rolling windows can be computationally expensive on large datasets.

### Annotated Complete Code Examples

**Example 1: Month-over-Month Growth**

```sql
-- Setup
CREATE TABLE monthly_sales (
    month   VARCHAR(7),
    revenue NUMERIC(10, 2)
);

INSERT INTO monthly_sales VALUES
    ('2026-01', 1000.00),
    ('2026-02', 1500.00),
    ('2026-03', 1200.00);

-- Query: MoM growth
SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_month,
    revenue - LAG(revenue) OVER (ORDER BY month) AS mom_change,
    ROUND((revenue - LAG(revenue) OVER (ORDER BY month))
        / NULLIF(LAG(revenue) OVER (ORDER BY month), 0) * 100, 2) AS mom_pct
FROM monthly_sales
ORDER BY month;

-- Expected Output:
--   month   | revenue | prev_month | mom_change | mom_pct
-- ----------+---------+------------+------------+---------
--  2026-01 | 1000.00 | (null)     | (null)     | (null)
--  2026-02 | 1500.00 | 1000.00    | 500.00     | 50.00
--  2026-03 | 1200.00 | 1500.00    | -300.00    | -20.00
```

**Why this output occurs:** `LAG(revenue)` retrieves the previous month's value. The difference and percentage change are computed. January has no previous month, so it returns NULL .

### Real-World Cases

**Case 1: Financial Reporting**

A CFO dashboard shows revenue with MoM growth percentage and 3-month rolling average to smooth seasonal spikes .

**Case 2: Operations Monitoring**

An SRE monitors API error rates with a 7-day rolling average using `RANGE BETWEEN INTERVAL '6 days' PRECEDING AND CURRENT ROW` to detect gradual degradation .

### References

- Lakehouse Time Series Analysis Guide 
- Analytical SQL — Period-over-Period 
- LinkedIn Post on Period-over-Period Analysis 

---

## 5. User Lifecycle Metrics

### Definitions

**Core Definition:** User lifecycle metrics patterns compute cohort retention, churn rates, and customer lifetime value (LTV) by grouping users based on their acquisition period and tracking their behavior over time.

**Technical Definition:** Cohort analysis assigns each user to a cohort based on their first purchase month, then tracks retention (whether they purchased in subsequent months) and cumulative revenue (LTV) per cohort. Churn is the inverse of retention. LTV is computed as cumulative revenue per cohort divided by cohort size .

**Beginner-Friendly Explanation:** Imagine customers who joined in January. How many of them bought again in February? March? This pattern answers that. "Cohort" is just a fancy word for "group of customers who started at the same time."

### Purposes

- To measure customer retention over time .
- To calculate customer lifetime value (LTV) .
- To identify churn patterns and at-risk customers .
- To compare acquisition channels by retention quality .
- To support growth and marketing decisions.

### Syntax Rules and Structure

**Cohort Assignment:**

```sql
WITH customer_cohorts AS (
    SELECT
        customer_id,
        DATE_TRUNC('month', MIN(order_date)) AS cohort_month
    FROM orders
    GROUP BY customer_id
)
```

**Retention Calculation:**

```sql
SELECT
    c.cohort_month,
    DATE_TRUNC('month', o.order_date) AS activity_month,
    EXTRACT(YEAR FROM AGE(DATE_TRUNC('month', o.order_date), c.cohort_month)) * 12
        + EXTRACT(MONTH FROM AGE(DATE_TRUNC('month', o.order_date), c.cohort_month)) AS month_number,
    COUNT(DISTINCT o.customer_id) AS active_customers
FROM customer_cohorts c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.cohort_month, activity_month, month_number
```

**LTV Calculation:**

```sql
SELECT
    cohort_month,
    month_number,
    SUM(revenue) AS cohort_revenue,
    SUM(revenue) / COUNT(DISTINCT customer_id) AS arpu,
    SUM(SUM(revenue) / COUNT(DISTINCT customer_id))
        OVER (PARTITION BY cohort_month ORDER BY month_number) AS cumulative_ltv
FROM cohort_data
GROUP BY cohort_month, month_number
```

**Component Breakdown:**

| Metric | Definition | Formula |
|--------|-----------|---------|
| Retention | % of cohort active in month N | `active_in_month_N / cohort_size`  |
| Churn | % of cohort lost by month N | `1 - retention` |
| ARPU | Average revenue per user | `revenue / active_users`  |
| LTV | Cumulative revenue per user | `SUM(ARPU) OVER (PARTITION BY cohort ORDER BY month)`  |

**Syntax Rules:**

- Cohort is defined by first purchase month.
- `month_number` = months since first purchase (0 = acquisition month).
- Retention is computed per cohort per month.
- LTV is cumulative ARPU over months .

**Constraints and Limitations:**

- Requires historical transaction data with customer IDs .
- Small cohorts produce unreliable retention curves .
- LTV projections require assumptions about future retention .

### Annotated Complete Code Examples

**Example 1: Cohort Retention Table**

```sql
-- Setup
CREATE TABLE orders (
    customer_id INTEGER,
    order_date  DATE,
    amount      NUMERIC(10, 2)
);

INSERT INTO orders VALUES
    (1, '2026-01-15', 100.00),
    (1, '2026-02-20', 150.00),
    (2, '2026-01-20', 200.00),
    (2, '2026-03-10', 100.00),
    (3, '2026-02-05', 300.00),
    (3, '2026-03-15', 250.00);

-- Query: Cohort retention
WITH cohorts AS (
    SELECT
        customer_id,
        DATE_TRUNC('month', MIN(order_date))::date AS cohort_month
    FROM orders
    GROUP BY customer_id
),
activity AS (
    SELECT
        c.cohort_month,
        DATE_TRUNC('month', o.order_date)::date AS activity_month,
        (EXTRACT(YEAR FROM AGE(DATE_TRUNC('month', o.order_date)::date, c.cohort_month)) * 12
         + EXTRACT(MONTH FROM AGE(DATE_TRUNC('month', o.order_date)::date, c.cohort_month)))::int AS month_number,
        o.customer_id
    FROM cohorts c
    JOIN orders o ON c.customer_id = o.customer_id
)
SELECT
    cohort_month,
    month_number,
    COUNT(DISTINCT customer_id) AS active_customers
FROM activity
GROUP BY cohort_month, month_number
ORDER BY cohort_month, month_number;

-- Expected Output:
--  cohort_month | month_number | active_customers
-- --------------+--------------+------------------
--  2026-01-01  |            0 |                2
--  2026-01-01  |            1 |                1
--  2026-01-01  |            2 |                1
--  2026-02-01  |            0 |                1
--  2026-02-01  |            1 |                1
```

**Why this output occurs:** Customers 1 and 2 first purchased in January (cohort 2026-01). In month 0 (January), both are active (2 customers). In month 1 (February), only customer 1 purchases (1 customer). In month 2 (March), customers 1 and 2 purchase (2 customers — but wait, customer 2 purchased in March, so 2 active in month 2). Customer 3 is in the February cohort .

### Real-World Cases

**Case 1: SaaS Retention Analysis**

A SaaS company tracks monthly cohorts to measure product-market fit. High retention in months 1-3 indicates strong value delivery .

**Case 2: E-Commerce LTV by Channel**

An e-commerce retailer compares LTV across acquisition channels (paid search vs. organic) to optimize marketing spend .

### References

- Customer Cohort & Retention Analysis 
- Compute LTV from Cohort Retention Exercise 
- Customer Segmentation RFM Analysis 

---

## Summary Table of Enterprise Analytical Patterns

| Pattern | Core Technique | Key Function | Typical Use Case |
|---------|---------------|--------------|------------------|
| Top-N per Group | `ROW_NUMBER` + filter | `ROW_NUMBER() OVER (PARTITION BY ...)`  | Top products per category |
| Gaps & Islands | `value - ROW_NUMBER()` | `ROW_NUMBER()`, `LEAD()`  | Missing invoice detection |
| Sessionization | `LAG` → flag → `SUM` | `LAG()`, `SUM() OVER`  | Clickstream sessions |
| Time-Series | `LAG`, rolling `AVG` | `LAG()`, `AVG() OVER RANGE`  | MoM/YoY growth |
| Cohort/Retention | `MIN` date + join | `MIN()`, `COUNT(DISTINCT)`  | Customer retention |

---

## References

- MySQL Top N per Group with Window Functions 
- Advanced Window Functions — Gaps and Islands, Sessionization 
- Oracle Data Warehousing Guide — Sessionization with MATCH_RECOGNIZE 
- Lakehouse Time Series Analysis Guide 
- Customer Cohort & Retention Analysis with SQL 
- How to Get the Top N Records per Group in MySQL 
- Advanced Patterns: Gaps-and-Islands, Sessionization & Pivot 
- Sessionize an Event Stream Challenge 
- Compute LTV from Cohort Retention Exercise 
- SQL Window Functions — ROW_NUMBER, RANK, DENSE_RANK 
- Advanced Window Functions Reference 
- ClickHouse Session Analytics 
- Analytical SQL — Period-over-Period 
- Customer Segmentation RFM Analysis 
- SQL Window Functions Reference 
- Gaps-and-Islands Lecture Notes 
- Oracle Sessionization Examples 
- Period-over-Period Analysis Discussion 