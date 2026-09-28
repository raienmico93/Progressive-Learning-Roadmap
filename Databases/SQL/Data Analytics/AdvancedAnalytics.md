# SQL Advanced Analytics: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL advanced analytics is the application of sophisticated SQL constructs—window functions, statistical aggregates, cohort and funnel modeling, and distribution analysis—to extract deep behavioral, statistical, and temporal insights from large datasets.

**Technical Definition:** SQL advanced analytics encompasses a set of analytical techniques that go beyond basic aggregation: cohort analysis (grouping users by acquisition period and tracking subsequent activity), funnel analysis (measuring step-by-step conversion through ordered event sequences), customer segmentation (RFM scoring via `NTILE`), ranking analysis (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `QUALIFY`), statistical aggregation (`CORR`, `REGR_SLOPE`, `COVAR_POP`), percentile and median calculations (`PERCENTILE_CONT`, `PERCENTILE_DISC`, `MEDIAN`), and distribution analysis (`WIDTH_BUCKET`, `CUME_DIST`, histograms). These techniques are built on window functions, aggregate functions, and Common Table Expressions (CTEs).

**Beginner-Friendly Explanation:** SQL advanced analytics is like being a detective who not only counts things but also understands patterns. You can track how many customers who joined in January are still active in June (cohort analysis). You can see where people drop off in a signup process (funnel analysis). You can rank customers by how much they spend (ranking). You can calculate the median salary instead of just the average (median). And you can group values into ranges to draw a histogram (distribution). SQL gives you the tools to do all of this with data.

### Key Characteristics

- **Window-function-driven:** Most advanced analytics rely on window functions (`OVER`, `PARTITION BY`, `ORDER BY`, `ROWS/RANGE`).
- **Multi-layered:** Advanced analytics queries are typically composed of CTEs that build on each other.
- **Behavioral:** Focuses on user behavior over time (cohorts, funnels, retention).
- **Statistical:** Includes statistical functions (correlation, regression, percentiles) beyond simple aggregates.
- **Distribution-aware:** Analyzes how values are distributed (histograms, cumulative distributions, percentile boundaries).
- **Vendor-varied:** Support for advanced functions varies across PostgreSQL, SQL Server, MySQL, Oracle, and cloud warehouses (BigQuery, Snowflake, Databricks).

### Prerequisites

- Proficiency with SQL `SELECT`, `WHERE`, `JOIN`, `GROUP BY`, and `HAVING`.
- Understanding of aggregate functions (`SUM`, `COUNT`, `AVG`, `MIN`, `MAX`).
- Familiarity with window functions (`ROW_NUMBER`, `RANK`, `LAG`, `LEAD`, `SUM() OVER`).
- Knowledge of CTEs (`WITH` clause) for multi-step query composition.
- Awareness of `NULL` semantics and three-valued logic.

### Related Programming Areas

- Product analytics and growth engineering.
- Business intelligence and executive dashboards.
- Data science and statistical modeling.
- Marketing attribution and customer lifetime value (LTV).
- Data warehousing and ETL pipeline design.

### Core Concepts / Features

1. **Cohort Analysis** (lifecycle matrices, retention tracking)
2. **Funnel Analysis** (step-based drop-off, sessionized user paths)
3. **Customer Segmentation** (RFM modeling, `NTILE` scoring)
4. **Ranking Analysis** (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `QUALIFY`)
5. **Statistical Aggregation** (`CORR`, `REGR_SLOPE`, `REGR_INTERCEPT`, covariance)
6. **Percentiles** (`PERCENTILE_CONT`, `PERCENTILE_DISC`)
7. **Median Calculations** (`MEDIAN`, `PERCENTILE_CONT(0.5)`)
8. **Distribution Analysis** (frequency histograms, `WIDTH_BUCKET`, `CUME_DIST`)


## Core Concept 1: Cohort Analysis

### Definitions

**Core Definition:** Cohort analysis is the practice of grouping users by a shared characteristic (typically their first activity date) and tracking their behavior over subsequent time periods to measure retention, engagement, or conversion.

**Technical Definition:** Cohort analysis identifies each user's cohort date—the date of their first activity, computed as `MIN(activity_date) OVER (PARTITION BY user_id)` or `DATE_TRUNC('month', MIN(activity_date))`—and then joins subsequent activity to compute period offsets. The result is a lifecycle matrix (retention matrix) where each cell represents the count or percentage of users from a cohort who were active in a given period offset. The typical SQL pattern involves: (1) defining the cohort date per user, (2) calculating the period index (e.g., months since first activity), (3) aggregating distinct user counts by cohort and period, and (4) optionally pivoting the result into a wide retention matrix.

**Beginner-Friendly Explanation:** Cohort analysis is like tracking a group of students who started school in September. You want to know: How many of them are still attending in October? In November? In December? By grouping students by their start month, you can compare retention across different cohorts and see if newer cohorts are doing better or worse.

### Purposes

- To measure user retention and product stickiness over time.
- To compare engagement across different acquisition cohorts (e.g., users acquired in January vs. February).
- To identify the long-term impact of onboarding changes or product launches.
- To compute customer lifetime value (LTV) by cohort.

### Syntax Rules and Structure

#### Complete General Syntax (Cohort Definition)

```sql
WITH user_cohorts AS (
    SELECT
        user_id,
        DATE_TRUNC('month', MIN(activity_date)) AS cohort_month
    FROM events
    GROUP BY user_id
),
cohort_activity AS (
    SELECT
        c.cohort_month,
        DATE_TRUNC('month', e.activity_date) AS activity_month,
        COUNT(DISTINCT e.user_id) AS active_users
    FROM user_cohorts c
    JOIN events e ON c.user_id = e.user_id
    GROUP BY c.cohort_month, DATE_TRUNC('month', e.activity_date)
)
SELECT
    cohort_month,
    activity_month,
    active_users,
    -- Period offset in months
    (EXTRACT(YEAR FROM activity_month) - EXTRACT(YEAR FROM cohort_month)) * 12
    + (EXTRACT(MONTH FROM activity_month) - EXTRACT(MONTH FROM cohort_month)) AS period_offset
FROM cohort_activity
ORDER BY cohort_month, period_offset;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `user_cohorts` CTE | Assigns each user to their first activity month (`cohort_month`). |
| `cohort_activity` CTE | Joins users back to their activity, grouping by cohort and activity month. |
| `period_offset` | Computes the number of months between the activity month and the cohort month. |
| `COUNT(DISTINCT user_id)` | Counts unique users active in each cohort-period combination. |

#### Syntax Rules

- **Cohort date:** Use `MIN(activity_date)` per user to define the cohort. The granularity (day, week, month) depends on the analysis.
- **Period offset:** Compute the offset in the same units as the cohort granularity (e.g., months since signup for monthly cohorts).
- **Distinct users:** Always use `COUNT(DISTINCT user_id)` to avoid counting the same user multiple times within a period.
- **Pivot:** To create a wide retention matrix (cohort × period offset), use conditional aggregation or a pivot operation.

#### Constraints and Limitations

- **Cohort size:** Small cohorts produce volatile retention rates.
- **Incomplete cohorts:** Recent cohorts have fewer periods of data; their retention curves are incomplete.
- **Activity definition:** "Active" must be defined consistently (e.g., logged in, made a purchase, opened an app).
- **Time zone:** Ensure all activity dates are in a consistent time zone.

### Annotated Code Examples

#### Example 1: PostgreSQL — Monthly Cohort Retention Matrix

```sql
-- Create an events table
CREATE TABLE events (
    event_id SERIAL PRIMARY KEY,
    user_id INT,
    activity_date DATE
);

INSERT INTO events (user_id, activity_date) VALUES
(1, '2026-01-05'), (1, '2026-02-10'), (1, '2026-03-15'),
(2, '2026-01-10'), (2, '2026-02-20'),
(3, '2026-01-15'),
(4, '2026-02-01'), (4, '2026-03-05'),
(5, '2026-02-15'),
(6, '2026-03-01');

-- Cohort retention matrix
WITH user_cohorts AS (
    SELECT user_id, DATE_TRUNC('month', MIN(activity_date)) AS cohort_month
    FROM events
    GROUP BY user_id
),
cohort_activity AS (
    SELECT
        c.cohort_month,
        DATE_TRUNC('month', e.activity_date) AS activity_month,
        COUNT(DISTINCT e.user_id) AS active_users
    FROM user_cohorts c
    JOIN events e ON c.user_id = e.user_id
    GROUP BY c.cohort_month, DATE_TRUNC('month', e.activity_date)
)
SELECT
    cohort_month,
    activity_month,
    active_users,
    (EXTRACT(YEAR FROM activity_month) - EXTRACT(YEAR FROM cohort_month)) * 12
    + (EXTRACT(MONTH FROM activity_month) - EXTRACT(MONTH FROM cohort_month)) AS period_offset
FROM cohort_activity
ORDER BY cohort_month, period_offset;
```

**Expected Output:**

```
 cohort_month      | activity_month    | active_users | period_offset
-------------------+-------------------+--------------+---------------
 2026-01-01 00:00:00| 2026-01-01 00:00:00|            3 |             0
 2026-01-01 00:00:00| 2026-02-01 00:00:00|            2 |             1
 2026-01-01 00:00:00| 2026-03-01 00:00:00|            1 |             2
 2026-02-01 00:00:00| 2026-02-01 00:00:00|            2 |             0
 2026-02-01 00:00:00| 2026-03-01 00:00:00|            1 |             1
 2026-03-01 00:00:00| 2026-03-01 00:00:00|            1 |             0
```

**Why This Works:** The `user_cohorts` CTE assigns each user to their first activity month. The `cohort_activity` CTE joins users back to all their activity, grouping by cohort and activity month. The January cohort had 3 users at period 0, 2 at period 1, and 1 at period 2. This is the retention curve for the January cohort.

### Real-World Cases

- **SaaS:** Monthly cohort retention to measure product stickiness and inform LTV.
- **E-commerce:** Repeat purchase rate by acquisition cohort.
- **Mobile apps:** Day 1, Day 7, and Day 30 retention rates by install cohort.
- **Marketing:** Comparing retention of users acquired through different channels.

### References

- PostgreSQL: Window Functions Tutorial — https://www.postgresql.org/docs/current/tutorial-window.html
- PostgreSQL: WITH Queries (CTEs) — https://www.postgresql.org/docs/current/queries-with.html
- Microsoft: Cohort Analysis in SQL — https://learn.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql


## Core Concept 2: Funnel Analysis

### Definitions

**Core Definition:** Funnel analysis is the practice of measuring how many users progress through a defined sequence of steps (e.g., visit → add to cart → checkout → purchase), identifying where drop-off occurs.

**Technical Definition:** Funnel analysis uses sequential conditional aggregation to count distinct users at each step of a predefined funnel. The SQL pattern computes `COUNT(DISTINCT CASE WHEN step_condition THEN user_id END)` for each step, often with temporal ordering enforced via `ROW_NUMBER()`, `LAG()`, or self-joins to ensure steps occur in the correct sequence. Ordered funnels require steps to occur in chronological order, filtering out-of-order behavior.

**Beginner-Friendly Explanation:** Funnel analysis is like tracking how many people walk into a store, how many pick up a product, how many go to the checkout, and how many actually buy. Each step has fewer people than the one before—the "funnel" narrows. SQL helps you count how many people reach each step.

### Purposes

- To measure conversion rates at each stage of a multi-step process.
- To identify the step with the largest drop-off (the biggest opportunity for improvement).
- To compare funnel performance across segments, devices, or time periods.
- To evaluate the impact of UX changes or marketing campaigns on conversion.

### Syntax Rules and Structure

#### Complete General Syntax (Basic Funnel)

```sql
SELECT
    COUNT(DISTINCT CASE WHEN event_type = 'step1' THEN user_id END) AS step1_users,
    COUNT(DISTINCT CASE WHEN event_type = 'step2' THEN user_id END) AS step2_users,
    COUNT(DISTINCT CASE WHEN event_type = 'step3' THEN user_id END) AS step3_users,
    ROUND(
        COUNT(DISTINCT CASE WHEN event_type = 'step3' THEN user_id END) * 100.0
        / NULLIF(COUNT(DISTINCT CASE WHEN event_type = 'step1' THEN user_id END), 0),
        2
    ) AS step1_to_step3_pct
FROM events
WHERE event_date >= start_date AND event_date < end_date;
```

#### Complete General Syntax (Ordered Funnel with Window Functions)

```sql
WITH ordered_events AS (
    SELECT
        user_id,
        event_type,
        event_time,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY event_time) AS event_order
    FROM events
    WHERE event_type IN ('step1', 'step2', 'step3')
)
SELECT
    COUNT(DISTINCT CASE WHEN event_type = 'step1' THEN user_id END) AS step1_users,
    COUNT(DISTINCT CASE WHEN event_type = 'step2' THEN user_id END) AS step2_users,
    COUNT(DISTINCT CASE WHEN event_type = 'step3' THEN user_id END) AS step3_users
FROM ordered_events;
-- Note: For strict ordering, use LAG/LEAD or self-joins to verify sequence.
```

#### Syntax Rules

- **Distinct users:** Use `COUNT(DISTINCT user_id)` to count each user once per step.
- **Ordering:** For ordered funnels, ensure steps occur in chronological order. Use `ROW_NUMBER()` or `LAG()` to verify sequence.
- **Time window:** Define the conversion window (e.g., same day, 7 days, 30 days) consistently.
- **Step conditions:** Each step condition should be mutually exclusive (a user is counted in the first step they satisfy).

#### Constraints and Limitations

- **Order enforcement:** Basic `CASE` aggregation counts users who performed the action at any time, not necessarily in order. Use window functions or self-joins for strict ordering.
- **Guest users:** If users are not logged in, use session IDs or device IDs.
- **Bot traffic:** Filter out bot traffic to avoid inflated conversion rates.
- **Selection bias:** The denominator must include all users who entered the funnel, not just those who reached a later step.

### Annotated Code Examples

#### Example 1: PostgreSQL — Basic Funnel Analysis

```sql
-- Compute funnel conversion rates
SELECT
    COUNT(DISTINCT CASE WHEN event_type = 'visit' THEN user_id END) AS visitors,
    COUNT(DISTINCT CASE WHEN event_type = 'add_to_cart' THEN user_id END) AS cart_adders,
    COUNT(DISTINCT CASE WHEN event_type = 'purchase' THEN user_id END) AS purchasers,
    ROUND(
        COUNT(DISTINCT CASE WHEN event_type = 'add_to_cart' THEN user_id END) * 100.0
        / NULLIF(COUNT(DISTINCT CASE WHEN event_type = 'visit' THEN user_id END), 0),
        2
    ) AS visit_to_cart_pct,
    ROUND(
        COUNT(DISTINCT CASE WHEN event_type = 'purchase' THEN user_id END) * 100.0
        / NULLIF(COUNT(DISTINCT CASE WHEN event_type = 'visit' THEN user_id END), 0),
        2
    ) AS visit_to_purchase_pct
FROM events
WHERE event_date = '2026-09-15';
```

**Expected Output:**

```
 visitors | cart_adders | purchasers | visit_to_cart_pct | visit_to_purchase_pct
----------+-------------+------------+-------------------+-----------------------
        7 |           4 |          4 |             57.14 |                 57.14
```

**Why This Works:** The `CASE` expressions count distinct users at each funnel step. The percentage calculations divide the step count by the first step count, showing the conversion rate from visit to cart and from visit to purchase.

#### Example 2: PostgreSQL — Ordered Funnel with LAG

```sql
-- Ordered funnel: ensure steps occur in sequence
WITH user_steps AS (
    SELECT
        user_id,
        MAX(CASE WHEN event_type = 'visit' THEN event_time END) AS visit_time,
        MAX(CASE WHEN event_type = 'add_to_cart' THEN event_time END) AS cart_time,
        MAX(CASE WHEN event_type = 'purchase' THEN event_time END) AS purchase_time
    FROM events
    GROUP BY user_id
)
SELECT
    COUNT(DISTINCT CASE WHEN visit_time IS NOT NULL THEN user_id END) AS visitors,
    COUNT(DISTINCT CASE
        WHEN cart_time IS NOT NULL AND cart_time >= visit_time THEN user_id
    END) AS ordered_cart_adders,
    COUNT(DISTINCT CASE
        WHEN purchase_time IS NOT NULL AND purchase_time >= cart_time THEN user_id
    END) AS ordered_purchasers
FROM user_steps;
```

**Expected Output:**

```
 visitors | ordered_cart_adders | ordered_purchasers
----------+---------------------+--------------------
        7 |                   4 |                  3
```

**Why This Works:** The `user_steps` CTE pivots the event times for each user. The outer query counts users who performed each step in the correct order (cart after visit, purchase after cart). This filters out users who performed steps out of order.

### Real-World Cases

- **E-commerce:** Checkout funnel (product view → add to cart → checkout → purchase).
- **SaaS:** Signup funnel (landing page → signup form → email verification → first login).
- **Marketing:** Lead generation funnel (ad click → landing page → form fill → demo request).
- **Mobile apps:** Onboarding funnel (install → open → tutorial → first action).

### References

- PostgreSQL: Window Functions — https://www.postgresql.org/docs/current/functions-window.html
- PostgreSQL: Aggregate Expressions (FILTER) — https://www.postgresql.org/docs/current/sql-expressions.html
- Alibaba Cloud: Funnel Analysis Functions — https://www.alibabacloud.com/help/en/analyticdb-for-postgresql/latest/funnel-analysis-functions


## Core Concept 3: Customer Segmentation (RFM)

### Definitions

**Core Definition:** RFM (Recency, Frequency, Monetary) segmentation is a behavioral customer segmentation technique that scores customers on how recently they purchased (R), how often they purchase (F), and how much they spend (M), then groups them into segments based on these scores.

**Technical Definition:** RFM analysis computes three metrics per customer: Recency (days since last purchase), Frequency (total number of orders), and Monetary (total spend). Each metric is scored into quantile bins (typically 1–5) using `NTILE(5)`, with the recency score inverted (shorter days = higher score). The three scores are concatenated into an RFM segment code (e.g., "555" for best customers), and customers are classified into named segments (Champions, Loyal, At Risk, etc.) based on their score combination.

**Beginner-Friendly Explanation:** RFM is like grading your customers on a report card. How recently did they buy? (A for recent, F for long ago.) How often do they buy? How much do they spend? By combining these grades, you can identify your best customers ("Champions"), customers you are about to lose ("At Risk"), and new customers who might become loyal.

### Purposes

- To identify high-value customers for retention campaigns.
- To detect at-risk customers before they churn.
- To personalize marketing messages based on customer value.
- To allocate marketing budget efficiently by targeting the right segments.

### Syntax Rules and Structure

#### Complete General Syntax (RFM Scoring)

```sql
WITH rfm_base AS (
    SELECT
        customer_id,
        CURRENT_DATE - MAX(order_date) AS recency_days,
        COUNT(DISTINCT order_id) AS frequency,
        SUM(order_total) AS monetary
    FROM orders
    WHERE status <> 'cancelled'
    GROUP BY customer_id
),
rfm_scores AS (
    SELECT
        customer_id,
        recency_days,
        frequency,
        monetary,
        NTILE(5) OVER (ORDER BY recency_days ASC) AS r_score,
        NTILE(5) OVER (ORDER BY frequency DESC) AS f_score,
        NTILE(5) OVER (ORDER BY monetary DESC) AS m_score
    FROM rfm_base
)
SELECT
    customer_id,
    r_score,
    f_score,
    m_score,
    r_score || f_score || m_score AS rfm_segment,
    CASE
        WHEN r_score = 5 AND f_score >= 4 AND m_score >= 4 THEN 'Champions'
        WHEN r_score >= 4 AND f_score >= 3 THEN 'Loyal'
        WHEN r_score >= 4 AND f_score <= 2 THEN 'New Customers'
        WHEN r_score <= 2 AND f_score >= 3 THEN 'At Risk'
        WHEN r_score <= 2 AND f_score <= 2 THEN 'Hibernating'
        ELSE 'Potential Loyalist'
    END AS customer_segment
FROM rfm_scores
ORDER BY rfm_segment DESC;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `rfm_base` CTE | Computes recency (days), frequency (order count), and monetary (total spend) per customer. |
| `NTILE(5)` | Assigns quintile scores (1–5) for each metric. Recency is ordered ascending (shorter = better). |
| `r_score \|\| f_score \|\| m_score` | Concatenates the three scores into an RFM code. |
| `CASE` | Maps the RFM code to a named segment. |

#### Syntax Rules

- **Recency inversion:** `NTILE(5) OVER (ORDER BY recency_days ASC)` gives score 5 to the most recent customers.
- **Frequency and Monetary:** `NTILE(5) OVER (ORDER BY frequency DESC)` gives score 5 to the most frequent/highest-spending customers.
- **Segment rules:** Define segment rules based on business context; the rules above are illustrative.
- **NTILE determinism:** `NTILE` can be non-deterministic when there are ties in the `ORDER BY` column. Use additional tie-breaking columns for reproducible results.

#### Constraints and Limitations

- **NTILE tie behavior:** `NTILE` may distribute equal values across adjacent buckets, producing non-deterministic results. Add tie-breaking columns to `ORDER BY`.
- **Small customer bases:** `NTILE(5)` requires at least 5 customers; with fewer, some buckets may be empty.
- **Recency definition:** Use the most recent purchase date per customer; if a customer has no purchases, they are excluded or assigned to a separate segment.
- **Monetary currency:** Ensure all monetary values are in the same currency.

### Annotated Code Examples

#### Example 1: PostgreSQL — RFM Scoring and Segmentation

```sql
-- Create orders table
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    order_total NUMERIC(10,2),
    status TEXT DEFAULT 'completed'
);

INSERT INTO orders (customer_id, order_date, order_total) VALUES
(1, '2026-09-01', 500), (1, '2026-09-10', 300), (1, '2026-09-20', 200),
(2, '2026-08-15', 1000), (2, '2026-09-05', 800),
(3, '2026-09-25', 150),
(4, '2026-06-01', 2000), (4, '2026-07-01', 1500),
(5, '2026-09-28', 50);

-- RFM segmentation
WITH rfm_base AS (
    SELECT
        customer_id,
        CURRENT_DATE - MAX(order_date) AS recency_days,
        COUNT(DISTINCT order_id) AS frequency,
        SUM(order_total) AS monetary
    FROM orders
    WHERE status <> 'cancelled'
    GROUP BY customer_id
),
rfm_scores AS (
    SELECT
        customer_id,
        recency_days,
        frequency,
        monetary,
        NTILE(5) OVER (ORDER BY recency_days ASC) AS r_score,
        NTILE(5) OVER (ORDER BY frequency DESC) AS f_score,
        NTILE(5) OVER (ORDER BY monetary DESC) AS m_score
    FROM rfm_base
)
SELECT
    customer_id,
    recency_days,
    frequency,
    monetary,
    r_score,
    f_score,
    m_score,
    CASE
        WHEN r_score >= 4 AND f_score >= 4 AND m_score >= 4 THEN 'Champions'
        WHEN r_score >= 4 AND f_score >= 3 THEN 'Loyal'
        WHEN r_score >= 4 AND f_score <= 2 THEN 'New'
        WHEN r_score <= 2 AND f_score >= 3 THEN 'At Risk'
        ELSE 'Hibernating'
    END AS segment
FROM rfm_scores
ORDER BY customer_id;
```

**Expected Output:**

```
 customer_id | recency_days | frequency | monetary | r_score | f_score | m_score | segment
-------------+--------------+-----------+----------+---------+---------+---------+-----------
           1 |            8 |         3 |   1000.00|       5 |       5 |       3 | Champions
           2 |           23 |         2 |   1800.00|       3 |       3 |       5 | Loyal
           3 |            3 |         1 |    150.00|       5 |       1 |       1 | New
           4 |          103 |         2 |   3500.00|       1 |       3 |       5 | At Risk
           5 |            0 |         1 |     50.00|       5 |       1 |       1 | New
```

**Why This Works:** Each customer is scored on recency (days since last order), frequency (order count), and monetary (total spend). `NTILE(5)` assigns quintile scores. Customer 1 has the highest recency score (most recent) and frequency score (most orders), making them a Champion. Customer 4 has a low recency score (long ago) but high monetary, making them At Risk.

### Real-World Cases

- **E-commerce:** Identifying Champions for VIP programs and At Risk customers for win-back campaigns.
- **Retail:** Allocating marketing budget to high-value segments.
- **SaaS:** Scoring customers for upsell and cross-sell opportunities.
- **Financial services:** Segmenting customers for personalized product recommendations.

### References

- Microsoft: NTILE (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/functions/ntile-transact-sql
- Oracle: NTILE — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/NTILE.html
- Oracle: NTILE (Analytic Function) — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/NTILE.html


## Core Concept 4: Ranking Analysis

### Definitions

**Core Definition:** Ranking analysis is the practice of assigning a positional rank to each row within a partition based on an ordering criterion, using `ROW_NUMBER()`, `RANK()`, or `DENSE_RANK()`, and filtering the results with the `QUALIFY` clause or a subquery.

**Technical Definition:** Ranking functions differ in how they handle ties: `ROW_NUMBER()` returns a unique, sequential number for each row (no ties); `RANK()` assigns the same rank to tied rows and leaves gaps (e.g., 1, 1, 3); `DENSE_RANK()` assigns the same rank to tied rows without gaps (e.g., 1, 1, 2). The `QUALIFY` clause filters rows based on the result of a window function without requiring a subquery or CTE; it is to window functions what `HAVING` is to aggregates. `QUALIFY` requires at least one window function in the `SELECT` list or the `QUALIFY` clause itself.

**Beginner-Friendly Explanation:** Ranking analysis is like ranking sports teams. `ROW_NUMBER()` gives each team a unique position (1st, 2nd, 3rd) even if two teams have the same score. `RANK()` gives tied teams the same rank but skips the next rank (1st, 1st, 3rd). `DENSE_RANK()` gives tied teams the same rank without skipping (1st, 1st, 2nd). `QUALIFY` lets you say "show me only the top-ranked team" without writing a nested query.

### Purposes

- To identify the top N rows within each group (e.g., top 3 products per category).
- To assign unique row numbers for pagination or deduplication.
- To rank customers, products, or events by a metric.
- To filter window function results without subqueries using `QUALIFY`.

### Syntax Rules and Structure

#### Complete General Syntax (Ranking Functions)

```sql
ROW_NUMBER() OVER (
    [PARTITION BY partition_expression]
    ORDER BY sort_expression [ASC | DESC]
) AS row_num

RANK() OVER (
    [PARTITION BY partition_expression]
    ORDER BY sort_expression [ASC | DESC]
) AS rank_num

DENSE_RANK() OVER (
    [PARTITION BY partition_expression]
    ORDER BY sort_expression [ASC | DESC]
) AS dense_rank_num
```

#### Complete General Syntax (QUALIFY)

```sql
SELECT column_list, RANK() OVER (...) AS rnk
FROM table_name
QUALIFY rnk <= 3;
```

#### Syntax Rules

- **ROW_NUMBER:** Always returns a unique number; ties are broken arbitrarily (or by additional `ORDER BY` columns).
- **RANK:** Ties share the same rank; the next rank is skipped (1, 1, 3).
- **DENSE_RANK:** Ties share the same rank; no gaps (1, 1, 2).
- **QUALIFY:** Filters rows after window function evaluation, analogous to how `HAVING` filters after `GROUP BY`. The expressions in `QUALIFY` cannot contain aggregate functions.
- **QUALIFY availability:** Supported in Databricks SQL, Amazon Redshift, Snowflake, BigQuery, Teradata, and Oracle 26ai. Not supported in PostgreSQL, MySQL, or SQL Server (use a subquery or CTE instead).

#### Constraints and Limitations

- **QUALIFY cannot contain aggregates:** Only window functions are allowed.
- **QUALIFY requires a window function:** At least one window function must be present in the `SELECT` list or the `QUALIFY` clause.
- **Tie-breaking:** `ROW_NUMBER` with ties is non-deterministic; add a unique column to `ORDER BY` for reproducibility.
- **Version-specific:** `QUALIFY` requires Databricks Runtime 10.4+, Snowflake, BigQuery, Redshift, Teradata, or Oracle 26ai.

### Annotated Code Examples

#### Example 1: PostgreSQL — Ranking with ROW_NUMBER and RANK

```sql
-- Create a products table
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    category TEXT,
    product_name TEXT,
    revenue NUMERIC(10,2)
);

INSERT INTO products (category, product_name, revenue) VALUES
('Electronics', 'Laptop', 5000),
('Electronics', 'Phone', 5000),
('Electronics', 'Tablet', 3000),
('Clothing', 'Shirt', 1000),
('Clothing', 'Jacket', 2000),
('Clothing', 'Shoes', 2000);

-- Rank products by revenue within each category
SELECT
    category,
    product_name,
    revenue,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY revenue DESC) AS row_num,
    RANK() OVER (PARTITION BY category ORDER BY revenue DESC) AS rank_num,
    DENSE_RANK() OVER (PARTITION BY category ORDER BY revenue DESC) AS dense_rank_num
FROM products
ORDER BY category, revenue DESC;
```

**Expected Output:**

```
 category     | product_name | revenue | row_num | rank_num | dense_rank_num
--------------+--------------+---------+---------+----------+----------------
 Clothing     | Jacket       | 2000.00 |       1 |        1 |              1
 Clothing     | Shoes        | 2000.00 |       2 |        1 |              1
 Clothing     | Shirt        | 1000.00 |       3 |        3 |              2
 Electronics  | Laptop       | 5000.00 |       1 |        1 |              1
 Electronics  | Phone        | 5000.00 |       2 |        1 |              1
 Electronics  | Tablet       | 3000.00 |       3 |        3 |              2
```

**Why This Works:** Jacket and Shoes have the same revenue (2000). `ROW_NUMBER` assigns them 1 and 2 (arbitrary tie-break). `RANK` assigns both rank 1, and Shirt gets rank 3 (gap). `DENSE_RANK` assigns both rank 1, and Shirt gets rank 2 (no gap).

#### Example 2: Databricks SQL — QUALIFY for Top-N Filtering

```sql
-- Create a dealer table
CREATE TABLE dealer (id INT, city STRING, car_model STRING, quantity INT);
INSERT INTO dealer VALUES
(100, 'Fremont', 'Honda Civic', 10),
(100, 'Fremont', 'Honda Accord', 15),
(100, 'Fremont', 'Honda CRV', 7),
(200, 'Dublin', 'Honda Civic', 20),
(200, 'Dublin', 'Honda Accord', 10),
(200, 'Dublin', 'Honda CRV', 3),
(300, 'San Jose', 'Honda Civic', 5),
(300, 'San Jose', 'Honda Accord', 8);

-- Top-ranked car model by quantity per city using QUALIFY
SELECT city, car_model, quantity
FROM dealer
QUALIFY RANK() OVER (PARTITION BY city ORDER BY quantity DESC) = 1;
```

**Expected Output:**

```
 city     | car_model    | quantity
----------+--------------+---------
 Fremont  | Honda Accord |       15
 Dublin   | Honda Civic  |       20
 San Jose | Honda Accord |        8
```

**Why This Works:** The `QUALIFY` clause filters rows where `RANK() OVER (PARTITION BY city ORDER BY quantity DESC) = 1`. This returns the top-selling car model in each city without needing a subquery or CTE.

### Real-World Cases

- **Sales:** Top 10 products by revenue per region.
- **HR:** Highest-paid employee per department.
- **Web analytics:** Most-viewed page per session.
- **Finance:** Top-performing portfolio per fund manager.

### References

- Microsoft: NTILE (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/functions/ntile-transact-sql
- Databricks: QUALIFY Clause — https://learn.microsoft.com/en-us/azure/databricks/sql/language-manual/sql-ref-syntax-qry-select-qualify
- Oracle: QUALIFY Clause (Oracle 26ai) — https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/QUALIFY-clause.html
- Amazon Redshift: QUALIFY Clause — https://docs.aws.amazon.com/redshift/latest/dg/r_QUALIFY_clause.html


## Core Concept 5: Statistical Aggregation

### Definitions

**Core Definition:** Statistical aggregation is the use of SQL aggregate functions to compute statistical measures—correlation, covariance, linear regression slope and intercept—that describe relationships between variables.

**Technical Definition:** SQL provides statistical aggregate functions defined in the SQL:2003 standard: `CORR(Y, X)` returns the correlation coefficient; `COVAR_POP(Y, X)` and `COVAR_SAMP(Y, X)` return population and sample covariance; `REGR_SLOPE(Y, X)` returns the slope of the least-squares-fit linear equation; `REGR_INTERCEPT(Y, X)` returns the y-intercept; `REGR_R2(Y, X)` returns the square of the correlation coefficient. These functions operate on pairs of numeric expressions and ignore rows where either expression is NULL.

**Beginner-Friendly Explanation:** Statistical aggregation helps you understand relationships between two variables. `CORR` tells you how strongly two things are related (from -1 to 1). `REGR_SLOPE` tells you: for every 1-unit increase in X, how much does Y change? These functions are like the "trendline" in a spreadsheet chart, but computed directly in SQL.

### Purposes

- To measure the strength and direction of the relationship between two variables.
- To compute linear regression coefficients for predictive modeling.
- To analyze covariance and correlation for portfolio and risk analysis.
- To support data science feature engineering and exploratory analysis.

### Syntax Rules and Structure

#### Complete General Syntax (Statistical Aggregates)

```sql
SELECT
    CORR(Y, X) AS correlation,
    COVAR_POP(Y, X) AS population_covariance,
    COVAR_SAMP(Y, X) AS sample_covariance,
    REGR_SLOPE(Y, X) AS slope,
    REGR_INTERCEPT(Y, X) AS intercept,
    REGR_R2(Y, X) AS r_squared
FROM table_name;
```

**Component Breakdown:**

| Function | Description |
|----------|-------------|
| `CORR(Y, X)` | Correlation coefficient between Y and X. Range: -1 to 1. |
| `COVAR_POP(Y, X)` | Population covariance. |
| `COVAR_SAMP(Y, X)` | Sample covariance. |
| `REGR_SLOPE(Y, X)` | Slope of the least-squares-fit linear equation determined by the (X, Y) pairs. |
| `REGR_INTERCEPT(Y, X)` | Y-intercept of the least-squares-fit linear equation. |
| `REGR_R2(Y, X)` | Square of the correlation coefficient. |

#### Syntax Rules

- **Argument order:** `CORR(Y, X)` and `REGR_SLOPE(Y, X)` take the dependent variable (Y) first, then the independent variable (X).
- **NULL handling:** Rows where either Y or X is NULL are ignored.
- **Availability:** Supported in PostgreSQL 8.4+, Oracle, MariaDB 10.2+, SAP HANA, Snowflake, BigQuery, and Databricks. Not supported in MySQL or SQL Server.
- **Aggregate vs. window:** These functions can be used as aggregates (with `GROUP BY`) or as window functions (with `OVER`).

#### Constraints and Limitations

- **No MySQL/SQL Server support:** MySQL and SQL Server do not have built-in `CORR` or `REGR_*` functions; use manual formulas or application-level computation.
- **Non-deterministic with NULLs:** Results depend on the non-NULL pairs; document how NULLs are handled.
- **Outliers:** Correlation and regression are sensitive to outliers; consider robust alternatives.
- **Linear assumption:** `REGR_SLOPE` assumes a linear relationship; it may not capture non-linear patterns.

### Annotated Code Examples

#### Example 1: PostgreSQL — Correlation and Regression

```sql
-- Create a table with paired data
CREATE TABLE marketing_spend (
    spend NUMERIC(10,2),
    revenue NUMERIC(10,2)
);

INSERT INTO marketing_spend VALUES
(100, 500),
(200, 900),
(300, 1200),
(400, 1600),
(500, 2000);

-- Statistical aggregation
SELECT
    ROUND(CORR(revenue, spend)::NUMERIC, 4) AS correlation,
    ROUND(REGR_SLOPE(revenue, spend)::NUMERIC, 4) AS slope,
    ROUND(REGR_INTERCEPT(revenue, spend)::NUMERIC, 4) AS intercept,
    ROUND(REGR_R2(revenue, spend)::NUMERIC, 4) AS r_squared
FROM marketing_spend;
```

**Expected Output:**

```
 correlation |  slope  | intercept | r_squared
-------------+---------+-----------+-----------
      0.9965 |  3.8000 |   96.0000 |    0.9930
```

**Why This Works:** The correlation is 0.9965, indicating a very strong positive relationship between marketing spend and revenue. The slope is 3.8, meaning for every $1 increase in spend, revenue increases by $3.80. The R-squared is 0.9930, meaning 99.3% of the variance in revenue is explained by spend.

### Real-World Cases

- **Marketing:** Measuring the relationship between ad spend and conversions.
- **Finance:** Computing beta (slope) and correlation for stock analysis.
- **Operations:** Analyzing the relationship between temperature and energy consumption.
- **Healthcare:** Correlating patient metrics (e.g., BMI and blood pressure).

### References

- PostgreSQL: Aggregate Functions (Statistics) — https://www.postgresql.org/docs/current/functions-aggregate.html
- MariaDB: CORR and REGR_SLOPE — https://mariadb.com/kb/en/corr/
- SAP HANA: Time Series Window Aggregate Functions — https://help.sap.com/docs/SAP_HANA_PLATFORM
- Oracle: REGR_SLOPE — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/REGR_SLOPE.html


## Core Concept 6: Percentiles

### Definitions

**Core Definition:** Percentiles are inverse distribution functions that return the value below which a given percentage of values in a group fall, using either linear interpolation (`PERCENTILE_CONT`) or discrete selection (`PERCENTILE_DISC`).

**Technical Definition:** `PERCENTILE_CONT(percentile) WITHIN GROUP (ORDER BY expression)` computes a percentile by linear interpolation between adjacent values after ordering them. `PERCENTILE_DISC(percentile) WITHIN GROUP (ORDER BY expression)` returns the first value in the ordered set whose cumulative distribution is greater than or equal to the specified percentile. `PERCENTILE_CONT` returns a computed result (may not exist in the dataset); `PERCENTILE_DISC` simply returns a value from the set of values that are aggregated over. The data type of the result for `PERCENTILE_DISC` is the same as the data type of its `ORDER BY` item; for `PERCENTILE_CONT`, the result is a numeric or datetime type.

**Beginner-Friendly Explanation:** Percentiles divide a dataset into 100 equal parts. The 50th percentile is the median (the middle value). The 90th percentile is the value below which 90% of the data falls. `PERCENTILE_CONT` interpolates between values (like calculating an average of two middle values), while `PERCENTILE_DISC` picks an actual value from the dataset.

### Purposes

- To identify boundary values for SLA thresholds (e.g., 95th percentile response time).
- To analyze income, test scores, or performance distributions.
- To compute medians and quartiles.
- To compare distributions across groups.

### Syntax Rules and Structure

#### Complete General Syntax (PERCENTILE_CONT)

```sql
PERCENTILE_CONT(percentile) WITHIN GROUP (ORDER BY expression)
```

#### Complete General Syntax (PERCENTILE_DISC)

```sql
PERCENTILE_DISC(percentile) WITHIN GROUP (ORDER BY expression)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `percentile` | A value between 0 and 1 (e.g., 0.5 for median, 0.9 for 90th percentile). |
| `WITHIN GROUP (ORDER BY expression)` | Specifies the column to order by before computing the percentile. |
| `PERCENTILE_CONT` | Returns an interpolated value; the result may not exist in the dataset. |
| `PERCENTILE_DISC` | Returns a discrete value from the dataset. |

#### Syntax Rules

- **Percentile argument:** Must be between 0 and 1.
- **NULL handling:** NULLs are ignored in the `ORDER BY` expression.
- **Aggregate vs. window:** These functions can be used as aggregates (with `GROUP BY`) or as window functions (with `OVER`).
- **Type of result:** `PERCENTILE_DISC` returns the same data type as the `ORDER BY` expression; `PERCENTILE_CONT` returns a numeric or datetime type.

#### Constraints and Limitations

- **MySQL/SQL Server:** MySQL does not support `PERCENTILE_CONT`/`PERCENTILE_DISC`; SQL Server supports them from SQL Server 2012+.
- **Performance:** Percentile calculations require sorting; large datasets may benefit from approximate percentile functions (e.g., Oracle's `APPROX_PERCENTILE`).
- **Interpolation semantics:** `PERCENTILE_CONT` and `PERCENTILE_DISC` may return different results for the same percentile.

### Annotated Code Examples

#### Example 1: Oracle — PERCENTILE_CONT vs. PERCENTILE_DISC

```sql
-- Compute median salary using both functions
SELECT
    department_id,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary DESC) AS median_cont,
    PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY salary DESC) AS median_disc
FROM employees
GROUP BY department_id
ORDER BY department_id;
```

**Expected Output (partial):**

```
DEPARTMENT_ID | MEDIAN_CONT | MEDIAN_DISC
--------------+-------------+------------
           10 |        4400 |        4400
           20 |        9500 |       13000
           30 |        2850 |        2900
```

**Why This Works:** For department 20, `PERCENTILE_CONT(0.5)` returns 9500 (the interpolated median), while `PERCENTILE_DISC(0.5)` returns 13000 (the first value whose cumulative distribution is ≥ 0.5). The difference arises because `PERCENTILE_CONT` interpolates between adjacent values, while `PERCENTILE_DISC` selects an actual value from the dataset.

### Real-World Cases

- **DevOps:** 95th and 99th percentile response times for SLA monitoring.
- **Finance:** Portfolio return percentiles for risk analysis.
- **HR:** Salary percentiles for compensation benchmarking.
- **Education:** Test score percentiles for student ranking.

### References

- Oracle: PERCENTILE_CONT — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/PERCENTILE_CONT.html
- Oracle: PERCENTILE_DISC — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/PERCENTILE_DISC.html
- SQL Server: PERCENTILE_CONT — https://learn.microsoft.com/en-us/sql/t-sql/functions/percentile-cont-transact-sql
- SAP HANA: Distribution Functions — https://help.sap.com/docs/SAP_HANA_PLATFORM


## Core Concept 7: Median Calculations

### Definitions

**Core Definition:** The median is the middle value of a sorted dataset, or the interpolated value between the two middle values when the dataset has an even number of elements.

**Technical Definition:** `MEDIAN(expression)` is an inverse distribution function that assumes a continuous distribution model. It takes a numeric or datetime value and returns the middle value or an interpolated value that would be the middle value once the values are sorted. Nulls are ignored in the calculation. The result is computed by first ordering the rows: using N as the number of rows in the group, the row number (RN) of interest is `RN = (1 + (0.5*(N-1))`. The final result is computed by linear interpolation between the values from rows at `CRN = CEILING(RN)` and `FRN = FLOOR(RN)`. `MEDIAN` is the specific case of `PERCENTILE_CONT` where the percentile value defaults to 0.5.

**Beginner-Friendly Explanation:** The median is the middle number. If you sort all the values from smallest to largest, the median is the one in the middle. If there are two middle numbers, the median is the average of those two. Unlike the average, the median is not affected by extreme outliers.

### Purposes

- To find the typical value in a dataset that may have outliers.
- To compute median income, median salary, or median response time.
- To compare distributions across groups when the mean is misleading.
- To provide a robust measure of central tendency.

### Syntax Rules and Structure

#### Complete General Syntax (MEDIAN)

```sql
MEDIAN(expression) [OVER (query_partition_clause)]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `expression` | A numeric or datetime value. |
| `OVER (query_partition_clause)` | Optional; partitions the result set for window-based median calculation. |

#### Complete General Syntax (PERCENTILE_CONT for Median)

```sql
PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY expression)
```

#### Syntax Rules

- **MEDIAN availability:** Supported in Oracle, SAP HANA, Amazon Redshift, Snowflake, BigQuery, and Databricks. Not supported in PostgreSQL, MySQL, or SQL Server (use `PERCENTILE_CONT(0.5)` in PostgreSQL 9.4+ or a subquery).
- **NULL handling:** NULLs are ignored.
- **Interpolation:** For even-sized groups, `MEDIAN` returns the average of the two middle values.
- **Window usage:** `MEDIAN` can be used as a window function with `OVER (PARTITION BY ...)`.

#### Constraints and Limitations

- **PostgreSQL:** Does not have a native `MEDIAN` function; use `PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY ...)`.
- **MySQL:** Does not have `MEDIAN` or `PERCENTILE_CONT`; use a subquery with `ROW_NUMBER` or `LIMIT`.
- **SQL Server:** Does not have `MEDIAN`; use `PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY ...)` (SQL Server 2012+).
- **Performance:** Median calculation requires sorting; large datasets may benefit from approximate median functions.

### Annotated Code Examples

#### Example 1: Oracle — MEDIAN Salary by Department

```sql
-- Compute median salary by department
SELECT
    department_id,
    MEDIAN(salary) AS median_salary,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
ORDER BY department_id;
```

**Expected Output (partial):**

```
DEPARTMENT_ID | MEDIAN_SALARY | AVG_SALARY
--------------+---------------+-----------
           10 |          4400 |       4400
           20 |          9500 |       9500
           30 |          2850 |       4150
```

**Why This Works:** For department 30, the median salary is 2850, while the average is 4150. The average is higher because a few high salaries pull it up, while the median is not affected by outliers.

#### Example 2: PostgreSQL — Median with PERCENTILE_CONT

```sql
-- PostgreSQL median using PERCENTILE_CONT
SELECT
    department_id,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) AS median_salary
FROM employees
GROUP BY department_id
ORDER BY department_id;
```

**Expected Output:**

```
 department_id | median_salary
---------------+---------------
            10 |          4400
            20 |          9500
            30 |          2850
```

**Why This Works:** `PERCENTILE_CONT(0.5)` computes the median by linear interpolation. In PostgreSQL, this is the standard way to compute the median because there is no native `MEDIAN` function.

### Real-World Cases

- **HR:** Median salary by department for compensation analysis.
- **Real estate:** Median home price by neighborhood.
- **Healthcare:** Median patient wait time by hospital.
- **Finance:** Median transaction value for fraud detection.

### References

- Oracle: MEDIAN — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/MEDIAN.html
- SAP HANA: MEDIAN Function — https://help.sap.com/docs/SAP_HANA_PLATFORM
- Amazon Redshift: MEDIAN Function — https://docs.aws.amazon.com/redshift/latest/dg/r_MEDIAN.html


## Core Concept 8: Distribution Analysis

### Definitions

**Core Definition:** Distribution analysis is the practice of analyzing how values are distributed across a dataset, using frequency histograms, equiwidth bucketing (`WIDTH_BUCKET`), and cumulative distribution functions (`CUME_DIST`).

**Technical Definition:** `WIDTH_BUCKET(expression, hist_min, hist_max, bucket_count)` constructs equiwidth histograms, dividing the range into intervals (buckets) of identical sizes. Values below the low bucket return 0, and values above the high bucket return `bucket_count + 1`. `CUME_DIST()` calculates the cumulative distribution of a value in a group of values; the range of values returned is >0 to <=1. Tie values always evaluate to the same cumulative distribution value. `PERCENT_RANK()` is similar but computes the relative rank of a value. Frequency distributions can also be constructed using `FLOOR(expression / bucket_width) * bucket_width` in databases that lack `WIDTH_BUCKET`.

**Beginner-Friendly Explanation:** Distribution analysis is like creating a bar chart of your data. `WIDTH_BUCKET` groups values into equal-width ranges (e.g., ages 0–20, 20–40, 40–60) and tells you which bucket each value falls into. `CUME_DIST` tells you what percentage of values are less than or equal to a given value. This helps you understand the shape of your data—is it concentrated in the middle? Skewed to one side?

### Purposes

- To visualize the distribution of a continuous variable.
- To identify skewness, outliers, and concentration.
- To compute cumulative distributions for percentile analysis.
- To create histograms for dashboards and reports.

### Syntax Rules and Structure

#### Complete General Syntax (WIDTH_BUCKET)

```sql
WIDTH_BUCKET(expression, hist_min, hist_max, bucket_count)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `expression` | The value to bucket (numeric or datetime). |
| `hist_min` | The low boundary of bucket 1. |
| `hist_max` | The high boundary of bucket `bucket_count`. |
| `bucket_count` | The number of buckets (positive integer). |
| Returns | Bucket number (0 for underflow, `bucket_count + 1` for overflow). |

#### Complete General Syntax (CUME_DIST)

```sql
CUME_DIST() OVER (
    [PARTITION BY partition_expression]
    ORDER BY sort_expression
)
```

#### Complete General Syntax (Histogram with FLOOR)

```sql
SELECT
    FLOOR(expression / bucket_width) * bucket_width AS bucket_start,
    COUNT(*) AS frequency
FROM table_name
GROUP BY FLOOR(expression / bucket_width) * bucket_width
ORDER BY bucket_start;
```

#### Syntax Rules

- **WIDTH_BUCKET:** Divides the range [hist_min, hist_max] into `bucket_count` equal-width buckets. Values less than `hist_min` go to bucket 0; values greater than or equal to `hist_max` go to bucket `bucket_count + 1`.
- **CUME_DIST:** Returns a value >0 and ≤1. Tied values receive the same cumulative distribution.
- **CUME_DIST with NULLs:** By default, `CUME_DIST` includes NULL values and treats them as the lowest values.
- **FLOOR histogram:** `FLOOR(expression / bucket_width) * bucket_width` creates a lower bound for each bucket. Add `bucket_width` to get the upper bound.

#### Constraints and Limitations

- **WIDTH_BUCKET availability:** Supported in PostgreSQL, Oracle, Vertica, Snowflake, and Databricks. Not supported in MySQL or SQL Server.
- **CUME_DIST availability:** Supported in SQL Server 2012+, Oracle, PostgreSQL, MySQL 8.0+, and Snowflake.
- **Bucket boundary semantics:** Buckets are [closed, open), meaning the lower bound is included and the upper bound is excluded.
- **Equal-width limitation:** `WIDTH_BUCKET` produces equal-width buckets, which may not be optimal for skewed distributions.

### Annotated Code Examples

#### Example 1: PostgreSQL — Histogram with WIDTH_BUCKET

```sql
-- Create a table with numeric values
CREATE TABLE customer_ages (
    customer_id SERIAL PRIMARY KEY,
    age INT
);

INSERT INTO customer_ages (age) VALUES
(18), (22), (25), (28), (30), (35), (38), (42), (45), (50),
(55), (60), (65), (70), (22), (28), (35), (45), (55), (30);

-- Create a 5-bucket histogram (ages 18-70)
SELECT
    WIDTH_BUCKET(age, 18, 70, 5) AS bucket,
    MIN(age) AS min_age,
    MAX(age) AS max_age,
    COUNT(*) AS frequency
FROM customer_ages
GROUP BY WIDTH_BUCKET(age, 18, 70, 5)
ORDER BY bucket;
```

**Expected Output:**

```
 bucket | min_age | max_age | frequency
--------+---------+---------+-----------
      1 |      18 |      28 |         5
      2 |      30 |      38 |         5
      3 |      42 |      50 |         4
      4 |      55 |      60 |         3
      5 |      65 |      70 |         3
```

**Why This Works:** `WIDTH_BUCKET(age, 18, 70, 5)` divides the range 18–70 into 5 equal-width buckets of width 10.4. Bucket 1 covers ages 18–28.4, bucket 2 covers 28.4–38.8, and so on. The `COUNT(*)` shows the frequency in each bucket, creating a histogram of age distribution.

#### Example 2: SQL Server — CUME_DIST for Salary Distribution

```sql
-- Compute cumulative distribution of salaries
SELECT
    Department,
    LastName,
    Rate,
    CUME_DIST() OVER (PARTITION BY Department ORDER BY Rate) AS cume_dist,
    PERCENT_RANK() OVER (PARTITION BY Department ORDER BY Rate) AS pct_rank
FROM HumanResources.vEmployeeDepartmentHistory AS edh
INNER JOIN HumanResources.EmployeePayHistory AS e
    ON e.BusinessEntityID = edh.BusinessEntityID
WHERE Department IN (N'Information Services', N'Document Control')
ORDER BY Department, Rate DESC;
```

**Expected Output (partial):**

```
Department          | LastName      | Rate    | cume_dist | pct_rank
--------------------+---------------+---------+-----------+----------
Document Control    | Arifin        | 17.7885 |         1 |        1
Document Control    | Norred        | 16.8269 |       0.8 |      0.5
Document Control    | Kharatishvili | 16.8269 |       0.8 |      0.5
Document Control    | Chai          | 10.25   |       0.4 |        0
Document Control    | Berge         | 10.25   |       0.4 |        0
```

**Why This Works:** `CUME_DIST` returns the percentage of employees whose salary is less than or equal to the current employee's salary. Arifin has the highest salary, so `cume_dist = 1` (100% of employees earn ≤ their salary). Norred and Kharatishvili have the same salary, so they share the same `cume_dist = 0.8`.

### Real-World Cases

- **Marketing:** Age distribution of customers for targeting.
- **Finance:** Income distribution for wealth management.
- **DevOps:** Response time distribution for SLA analysis.
- **Education:** Test score distribution for grading curves.

### References

- Vertica: WIDTH_BUCKET — https://www.vertica.com/docs/11.1.x/PDF/Vertica_11.1.x_Complete_Documentation.pdf
- Oracle: WIDTH_BUCKET — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/WIDTH_BUCKET.html
- SQL Server: CUME_DIST — https://learn.microsoft.com/en-us/sql/t-sql/functions/cume-dist-transact-sql
- Oracle: CUME_DIST — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CUME_DIST.html
- PostgreSQL: WIDTH_BUCKET — https://www.postgresql.org/docs/current/functions-math.html


## Summary Table: Advanced Analytics Techniques

| Technique | Primary Constructs | Key Consideration |
|-----------|-------------------|-------------------|
| **Cohort Analysis** | `MIN() OVER`, CTEs, `DATE_TRUNC`, self-joins | Cohort definition and period offset |
| **Funnel Analysis** | `COUNT(DISTINCT CASE)`, `ROW_NUMBER()`, `LAG()` | Step ordering and conversion window |
| **RFM Segmentation** | `NTILE(5)`, CTEs, `CASE` | Recency inversion; tie-breaking |
| **Ranking Analysis** | `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `QUALIFY` | Tie handling; QUALIFY availability |
| **Statistical Aggregation** | `CORR()`, `REGR_SLOPE()`, `REGR_INTERCEPT()`, `COVAR_POP()` | Argument order (Y, X); linearity assumption |
| **Percentiles** | `PERCENTILE_CONT()`, `PERCENTILE_DISC()` | Interpolation vs. discrete selection |
| **Median** | `MEDIAN()`, `PERCENTILE_CONT(0.5)` | Availability varies by database |
| **Distribution Analysis** | `WIDTH_BUCKET()`, `CUME_DIST()`, `FLOOR()` | Bucket boundary semantics; NULL handling |


## Final Notes on Deprecated and Unsafe Features

- **QUALIFY availability:** Supported in Databricks SQL, Amazon Redshift, Snowflake, BigQuery, Teradata, and Oracle 26ai. Not supported in PostgreSQL, MySQL, or SQL Server. Use a subquery or CTE for those databases.
- **WIDTH_BUCKET availability:** Supported in PostgreSQL, Oracle, Vertica, Snowflake, and Databricks. Not supported in MySQL or SQL Server. Use `FLOOR(expression / bucket_width) * bucket_width` as a portable alternative.
- **MEDIAN availability:** Supported in Oracle, SAP HANA, Amazon Redshift, Snowflake, and BigQuery. Not supported in PostgreSQL, MySQL, or SQL Server. Use `PERCENTILE_CONT(0.5)` in PostgreSQL (9.4+) and SQL Server (2012+).
- **CORR and REGR_SLOPE availability:** Supported in PostgreSQL 8.4+, Oracle, MariaDB 10.2+, SAP HANA, Snowflake, BigQuery, and Databricks. Not supported in MySQL or SQL Server.
- **NTILE non-determinism:** `NTILE` may distribute equal values across adjacent buckets, producing non-deterministic results. Add tie-breaking columns to `ORDER BY` for reproducible results.
- **CUME_DIST and NULLs:** `CUME_DIST` includes NULL values and treats them as the lowest values. This may produce unexpected results if NULLs are present.
- **Window function frame defaults:** When `ORDER BY` is specified and no frame is specified, the default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. For cumulative calculations, this is correct; for rolling calculations, always specify the frame explicitly.
- **Version-specific:** `QUALIFY` requires Databricks Runtime 10.4+, Snowflake, BigQuery, Redshift, Teradata, or Oracle 26ai. `PERCENTILE_CONT` requires SQL Server 2012+ and PostgreSQL 9.4+. `CUME_DIST` requires SQL Server 2012+ and MySQL 8.0+.