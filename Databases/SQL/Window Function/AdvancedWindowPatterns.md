# Comprehensive Programming Cheat Sheet: Advanced Window Patterns

## Topic Overview

### Definitions

**Core Definition**  
Advanced window patterns are reusable analytical query techniques that combine window functions with CTEs, `LAG`/`LEAD`, ranking functions, and frame clauses to solve complex problems such as identifying consecutive ranges, grouping events into sessions, measuring retention, and comparing periods.

**Technical Definition**  
Advanced window patterns are query compositions that leverage the SQL window function mechanism—`OVER`, `PARTITION BY`, `ORDER BY`, and frame clauses—in combination with ranking functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`), offset functions (`LAG`, `LEAD`), and aggregate window functions. These patterns often use the row-number-difference technique for gaps and islands, `LAG` for sessionization and period comparisons, conditional aggregation for cohort and funnel analysis, and the `WINDOW` clause for reusable window specifications. They are supported across PostgreSQL, MySQL 8.0+, SQL Server, Oracle, and other modern analytical engines, with vendor-specific optimizations such as ClickHouse's `windowFunnel` and `retention` functions.

**Beginner-Friendly Explanation**  
Advanced window patterns are “recipes” for solving common analytical puzzles using window functions. For example, if you want to know which consecutive dates a user logged in, or how many users returned after signing up, or what percentage of visitors completed each step of a purchase funnel, these patterns give you a structured SQL approach instead of a complicated tangle of self-joins and subqueries.

### Key Characteristics

- **Composable**: Patterns combine multiple window functions, CTEs, and sometimes joins to express multi-step logic in one query.
- **Row-preserving**: Window functions retain every input row, which is essential for detail-level analytics such as session events or funnel steps.
- **Order-sensitive**: Most advanced patterns depend critically on `ORDER BY` within the window to define “previous,” “next,” “first,” and “consecutive.”
- **Partition-aware**: `PARTITION BY` resets calculations per entity (user, product, account), which is fundamental to sessionization, cohort, and funnel analysis.
- **Vendor-extensible**: While the core patterns use standard SQL window functions, some engines provide specialized functions (e.g., ClickHouse `windowFunnel`, `retention`; Databricks `MATCH_RECOGNIZE`) for sequential pattern matching.
- **Performance-sensitive**: Patterns that require multiple sorts or large partitions benefit from indexes on `(PARTITION BY columns, ORDER BY columns)`.

### Prerequisites

- Solid understanding of window functions: `OVER`, `PARTITION BY`, `ORDER BY`, frame clauses (`ROWS`, `RANGE`, `GROUPS`).
- Familiarity with ranking functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`) and offset functions (`LAG`, `LEAD`).
- Knowledge of CTEs (`WITH` clauses) for staging intermediate results.
- Awareness of the target engine’s support for specialized analytical functions.

### Related Programming Areas

- Product analytics: sessionization, funnels, retention.
- Financial reporting: period-over-period comparisons, cohort revenue.
- Data quality: gaps and islands, sequence validation.
- Event stream processing: sequential pattern detection.
- Data warehousing and OLAP.

---

## Core Concepts and Features

### 1. Named Windows

#### Definitions

**Core Definition**  
A named window is a window specification defined once in a `WINDOW` clause and referenced by name in multiple `OVER` clauses, reducing repetition and ensuring consistency.

**Technical Definition**  
The `WINDOW` clause appears after `HAVING` and before `ORDER BY` in a `SELECT` statement. It defines one or more window names, each with a complete window specification (`PARTITION BY`, `ORDER BY`, and frame clause). These names can then be referenced in `OVER` clauses as `OVER window_name` or extended with additional clauses as `OVER (window_name ...)`. The `WINDOW` clause is supported by PostgreSQL, MySQL 8.0+, and SQL Server (where it is not a separate clause but can be emulated through repeated specifications).

**Beginner-Friendly Explanation**  
If you use the same window definition in several places, you can give it a name once and refer to that name instead of copying the whole `OVER (...)` definition repeatedly. It is like defining a variable for a window.

#### Purposes

- To avoid duplicating complex window specifications across multiple function calls.
- To ensure that several window functions use exactly the same partitioning and ordering.
- To make queries more readable and maintainable.
- To support inheritance and customization of window definitions (see next section).

#### Syntax Rules and Structure

**PostgreSQL** (official) :

```sql
SELECT ...
FROM ...
WINDOW window_name AS ( window_definition ) [, ...]
ORDER BY ...;
```

**MySQL 8.0+**:

```sql
SELECT ...
FROM ...
WINDOW window_name AS ( window_definition ) [, ...]
ORDER BY ...;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `WINDOW` | Keyword introducing the named window clause. |
| `window_name` | Identifier for the window. |
| `AS` | Separator. |
| `window_definition` | Parenthesized specification: `[PARTITION BY ...] [ORDER BY ...] [frame_clause]`. |

**Syntax rules**

- The `WINDOW` clause appears after `HAVING` and before `ORDER BY`.
- A named window can be referenced in `OVER window_name` or extended in `OVER (window_name ...)`.
- Window names must be unique within the query.
- Named windows can reference earlier named windows (inheritance, see next section).

**Constraints and limitations**

- SQL Server does not have a `WINDOW` clause; the same effect requires repeating the window specification or using a CTE.
- The `WINDOW` clause cannot be used in subqueries in some engines.
- Referencing a window name in `OVER` without parentheses (`OVER w`) is usually equivalent to `OVER (w)`, but the latter allows extension.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Reusing a named window

```sql
SELECT depname, empno, salary,
       rank() OVER w AS rank,
       avg(salary) OVER w AS dept_avg
FROM empsalary
WINDOW w AS (PARTITION BY depname ORDER BY salary DESC);
```

**Expected output** (illustrative):

```text
 depname  | empno | salary | rank |      dept_avg
----------+-------+--------+------+---------------------
 develop  |     8 |   6000 |    1 | 5020.0000000000000000
 develop  |    10 |   5200 |    2 | 5020.0000000000000000
 develop  |    11 |   5200 |    2 | 5020.0000000000000000
 develop  |     9 |   4500 |    4 | 5020.0000000000000000
 develop  |     7 |   4200 |    5 | 5020.0000000000000000
 personnel|     2 |   3900 |    1 | 3700.0000000000000000
 personnel|     5 |   3500 |    2 | 3700.0000000000000000
 sales    |     1 |   5000 |    1 | 4866.6666666666666667
 sales    |     4 |   4800 |    2 | 4866.6666666666666667
 sales    |     3 |   4800 |    2 | 4866.6666666666666667
(10 rows)
```

**Why this output appears**: Both `rank()` and `avg(salary)` reference the same named window `w`, guaranteeing identical partitioning and ordering. The named window avoids repeating `(PARTITION BY depname ORDER BY salary DESC)` twice.

#### Real-World Cases

- **Consistent analytics**: Multiple metrics (rank, average, running total) over the same partition.
- **Maintainability**: Changing the window definition in one place updates all references.
- **Code review**: Named windows make the query’s window logic explicit and easy to audit.

---

### 2. Window Inheritance

#### Definitions

**Core Definition**  
Window inheritance is the ability of a named window definition to reference an earlier named window and copy its partitioning and ordering, optionally adding or overriding clauses.

**Technical Definition**  
When a `window_definition` begins with an `existing_window_name`, the new window copies the partitioning clause from the referenced window and its ordering clause if the referenced window has one. The new window cannot specify its own `PARTITION BY` if it inherits partitioning, and it can specify `ORDER BY` only if the inherited window does not have one. The new window always uses its own frame clause; the inherited window must not specify a frame clause. This feature is part of the SQL standard and is implemented in PostgreSQL and MySQL.

**Beginner-Friendly Explanation**  
You can define a base window (e.g., partitioned by department) and then create a refined window that inherits the partitioning but adds an ordering or a different frame. It is like inheriting properties from a parent window and customizing only what you need.

#### Purposes

- To build window specifications incrementally without repetition.
- To share partitioning across multiple windows while varying ordering or framing.
- To express hierarchical relationships between window definitions.
- To reduce errors by ensuring consistent partitioning.

#### Syntax Rules and Structure

**Syntax**:

```sql
WINDOW base AS (PARTITION BY ...),
       derived AS (base ORDER BY ...)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `base` | Parent window with partitioning (and possibly ordering). |
| `derived` | Child window referencing `base` and adding ordering or frame. |

**Syntax rules**

- The referenced window must be defined earlier in the `WINDOW` list.
- A derived window copies partitioning from the base.
- A derived window can add `ORDER BY` only if the base does not already have one.
- A derived window always uses its own frame clause; the base must not specify a frame if it is to be inherited.
- A derived window cannot specify its own `PARTITION BY` when inheriting.

**Constraints and limitations**

- Inheritance is not supported in SQL Server’s `OVER` clause (no `WINDOW` clause).
- Frame clauses cannot be inherited.
- The rules are strict; violating them causes syntax errors.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Inheriting partitioning and adding ordering

```sql
SELECT depname, empno, salary,
       sum(salary) OVER base AS total_salary,
       rank() OVER derived AS salary_rank
FROM empsalary
WINDOW base AS (PARTITION BY depname),
       derived AS (base ORDER BY salary DESC);
```

**Expected output** (illustrative):

```text
 depname  | empno | salary | total_salary | salary_rank
----------+-------+--------+--------------+-------------
 develop  |     8 |   6000 |        25100 |           1
 develop  |    10 |   5200 |        25100 |           2
 develop  |    11 |   5200 |        25100 |           2
 develop  |     9 |   4500 |        25100 |           4
 develop  |     7 |   4200 |        25100 |           5
 personnel|     2 |   3900 |         7400 |           1
 personnel|     5 |   3500 |         7400 |           2
 sales    |     1 |   5000 |        14600 |           1
 sales    |     4 |   4800 |        14600 |           2
 sales    |     3 |   4800 |        14600 |           2
(10 rows)
```

**Why this output appears**: `base` partitions by department but has no ordering, so `sum(salary)` computes the department total on every row. `derived` inherits the partitioning and adds `ORDER BY salary DESC`, so `rank()` ranks within each department.

#### Real-World Cases

- **Layered analytics**: A base window for partitioning and derived windows for different orderings (e.g., by date, by amount).
- **Consistency**: Ensuring that multiple rankings use the same partition but different sort keys.
- **Maintainability**: Centralizing partitioning logic in a base window.

---

### 3. Window Function Chaining

#### Definitions

**Core Definition**  
Window function chaining is the practice of using multiple window functions—or multiple layers of window functions—in the same query, often with different window specifications, to compute several analytical results simultaneously.

**Technical Definition**  
Window function chaining can mean (a) multiple window functions in the same `SELECT` list, each with its own `OVER` clause, or (b) successive layers of window functions where the output of one window function is used as input to another, typically through a CTE or subquery. Because window functions are evaluated after `WHERE`, `GROUP BY`, and `HAVING`, they cannot be nested directly; chaining requires a CTE or subquery to stage the intermediate result.

**Beginner-Friendly Explanation**  
Chaining means computing one window result, then using that result to compute another. For example, first rank employees by salary within each department, then compute the running total of those ranks. You cannot do it in one `SELECT` because window functions cannot be nested, so you use a CTE to pass the first result to the second step.

#### Purposes

- To compute multi-step analytics in a readable, modular way.
- To apply window functions to the results of other window functions.
- To avoid deeply nested subqueries.
- To optimize performance by reducing the dataset at each stage.

#### Syntax Rules and Structure

**Pattern**:

```sql
WITH stage1 AS (
    SELECT ...,
           ROW_NUMBER() OVER (...) AS rn
    FROM ...
)
SELECT ...,
       SUM(rn) OVER (...) AS running_rn
FROM stage1;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `stage1` | CTE computing the first window result. |
| `rn` | Output of the first window function. |
| Final `SELECT` | Applies a second window function to `rn`. |

**Syntax rules**

- Window functions cannot be nested directly; use a CTE or subquery.
- Each `OVER` clause can have its own `PARTITION BY`, `ORDER BY`, and frame.
- Multiple window functions in the same `SELECT` list are allowed if they are independent.
- Performance may degrade with many different window specifications; reusing named windows helps.

**Constraints and limitations**

- Deep chaining may require materializing intermediate results, which can be expensive.
- Some engines optimize multiple window functions with the same partitioning into a single sort, but different orderings may require multiple sorts.
- Chaining through CTEs may prevent predicate pushdown in some engines.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Two-stage chaining with a CTE

```sql
WITH ranked AS (
    SELECT depname, empno, salary,
           ROW_NUMBER() OVER (PARTITION BY depname ORDER BY salary DESC) AS rn
    FROM empsalary
)
SELECT depname, empno, salary, rn,
       SUM(rn) OVER (PARTITION BY depname ORDER BY rn) AS cumulative_rn
FROM ranked;
```

**Expected output** (illustrative):

```text
 depname  | empno | salary | rn | cumulative_rn
----------+-------+--------+----+--------------
 develop  |     8 |   6000 |  1 |             1
 develop  |    10 |   5200 |  2 |             3
 develop  |    11 |   5200 |  3 |             6
 develop  |     9 |   4500 |  4 |            10
 develop  |     7 |   4200 |  5 |            15
 personnel|     2 |   3900 |  1 |             1
 personnel|     5 |   3500 |  2 |             3
 sales    |     1 |   5000 |  1 |             1
 sales    |     4 |   4800 |  2 |             3
 sales    |     3 |   4800 |  3 |             6
(10 rows)
```

**Why this output appears**: The CTE computes `rn` (row number within department); the outer query computes a cumulative sum of `rn` within the same department. This two-stage approach is necessary because `SUM(ROW_NUMBER() OVER (...)) OVER (...)` would be invalid (window functions cannot be nested directly).

#### Real-World Cases

- **Multi-step ranking**: Rank by one metric, then rank the ranks.
- **Layered running totals**: Compute a running total, then a running total of that running total.
- **Data quality**: Flag rows based on one window result, then aggregate the flags with another window.

---

### 4. Gaps and Islands

#### Definitions

**Core Definition**  
Gaps and islands is a pattern that identifies consecutive ranges of values (“islands”) and the missing values between them (“gaps”).

**Technical Definition**  
The gaps-and-islands pattern typically uses the row-number difference technique: for a sequence of consecutive values, the difference between the value and `ROW_NUMBER()` is constant, forming a group identifier. Grouping by this difference collapses consecutive values into islands. Gaps are then the ranges between islands, often detected using `LEAD` to compare the end of one island with the start of the next. The pattern is used for consecutive date ranges, sequential numbers, and any ordered data with missing or repeated segments.

**Beginner-Friendly Explanation**  
Imagine a list of check-in dates for a user. Some dates are consecutive (an “island”), and some dates are missing (a “gap”). This pattern groups the consecutive dates together and highlights the missing ones.

#### Purposes

- To find consecutive runs of values (e.g., consecutive login days).
- To identify missing values or gaps in sequences.
- To collapse consecutive ranges into summary rows.
- To validate sequential integrity in data.

#### Syntax Rules and Structure

**Row-number difference pattern**:

```sql
WITH numbered AS (
    SELECT *,
           value - ROW_NUMBER() OVER (ORDER BY value) AS grp
    FROM sequence
)
SELECT MIN(value) AS island_start, MAX(value) AS island_end, COUNT(*) AS length
FROM numbered
GROUP BY grp
ORDER BY island_start;
```

**Gap detection with `LEAD`**:

```sql
SELECT value AS gap_start, next_value AS gap_end
FROM (
    SELECT value, LEAD(value) OVER (ORDER BY value) AS next_value
    FROM sequence
) t
WHERE next_value - value > 1;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `value - ROW_NUMBER()` | Produces a constant group key for consecutive values. |
| `GROUP BY grp` | Collapses each island. |
| `LEAD(value)` | Retrieves the next value to detect gaps. |

**Syntax rules**

- The sequence must be ordered by the value or date.
- `ROW_NUMBER()` must be ordered by the same column as the sequence.
- For date sequences, subtract an integer row number from the date (using date arithmetic).
- Gaps are detected when the difference between consecutive values exceeds the expected step.

**Constraints and limitations**

- Duplicate values require deduplication first.
- Non-numeric sequences (e.g., dates) require compatible arithmetic.
- The technique assumes a fixed step size (e.g., 1 day).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Finding consecutive voucher numbers

```sql
;WITH cte AS (
    SELECT *,
           CAST(SUBSTRING(VoucherNo, 2, 3) AS INT)
           - ROW_NUMBER() OVER (ORDER BY VoucherNo) AS Grp
    FROM T1
)
SELECT TransNo,
       MIN(VoucherNo) AS FirstVoucherNo,
       MAX(VoucherNo) AS LastVoucherNo,
       COUNT(*) AS Quantity
FROM cte
GROUP BY TransNo, Grp;
```

**Expected output**:

```text
TransNo FirstVoucherNo LastVoucherNo Quantity
------- -------------- ------------- --------
Trns1   V100           V104          5
Trns1   V106           V106          1
```

**Why this output appears**: The difference between the numeric part of the voucher number and `ROW_NUMBER()` is constant for consecutive vouchers. Grouping by this difference collapses V100–V104 into one island (quantity 5) and V106 into another (quantity 1), revealing the gap at V105.

##### Example 2: PostgreSQL – Consecutive login dates

```sql
WITH numbered AS (
    SELECT user_id, login_date,
           login_date - ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date)::int AS grp
    FROM logins
)
SELECT user_id, MIN(login_date) AS streak_start, MAX(login_date) AS streak_end, COUNT(*) AS streak_length
FROM numbered
GROUP BY user_id, grp
ORDER BY user_id, streak_start;
```

**Expected output** (illustrative):

```text
 user_id | streak_start | streak_end | streak_length
---------+--------------+------------+---------------
       1 | 2025-01-01   | 2025-01-05 |             5
       1 | 2025-01-08   | 2025-01-10 |             3
       2 | 2025-02-01   | 2025-02-02 |             2
(3 rows)
```

**Why this output appears**: The difference between `login_date` and `ROW_NUMBER()` is constant for consecutive dates. Grouping by this difference produces streaks; the gap between 2025-01-05 and 2025-01-08 is visible as separate islands.

#### Real-World Cases

- **User streaks**: Consecutive days of app usage.
- **Inventory**: Consecutive serial numbers in stock.
- **Log analysis**: Missing sequence numbers in event logs.
- **Financial data**: Consecutive trading days with prices.

---

### 5. Sessionization

#### Definitions

**Core Definition**  
Sessionization is the process of grouping a stream of events into sessions based on inactivity gaps, where a new session begins when the time since the previous event exceeds a threshold.

**Technical Definition**  
Sessionization uses `LAG` to compute the gap between each event and the previous event for the same entity (e.g., user). A new session is flagged when the gap exceeds the timeout (commonly 30 minutes). A cumulative sum of the new-session flags produces a session identifier. Grouping by this identifier yields session-level metrics such as start time, end time, duration, and event count.

**Beginner-Friendly Explanation**  
If a user clicks around a website, pauses for more than 30 minutes, then clicks again, the pause marks the start of a new session. Sessionization automatically assigns a session ID to each group of clicks separated by less than 30 minutes.

#### Purposes

- To measure session duration, page views per session, and bounce rates.
- To attribute conversions to sessions.
- To analyze user engagement patterns.
- To feed session-level data into downstream analytics.

#### Syntax Rules and Structure

**Pattern**:

```sql
WITH gaps AS (
    SELECT user_id, event_time,
           event_time - LAG(event_time) OVER (
               PARTITION BY user_id ORDER BY event_time
           ) AS gap
    FROM events
),
flags AS (
    SELECT *,
           CASE WHEN gap IS NULL OR gap > INTERVAL '30 minutes' THEN 1 ELSE 0 END AS new_session
    FROM gaps
),
sessions AS (
    SELECT *,
           SUM(new_session) OVER (
               PARTITION BY user_id ORDER BY event_time
           ) AS session_id
    FROM flags
)
SELECT user_id, session_id, MIN(event_time) AS session_start, MAX(event_time) AS session_end, COUNT(*) AS event_count
FROM sessions
GROUP BY user_id, session_id;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `LAG(event_time)` | Previous event time per user. |
| `gap` | Time since previous event. |
| `new_session` | Flag: 1 if gap > timeout or first event. |
| `SUM(new_session) OVER (...)` | Cumulative sum assigns session IDs. |

**Syntax rules**

- `LAG` must be partitioned by the entity (user) and ordered by event time.
- The timeout threshold is domain-specific (30 minutes is common for web analytics).
- The cumulative sum of the flag creates a monotonically increasing session ID.
- First event of each user has `gap = NULL`, which must be treated as a new session.

**Constraints and limitations**

- Requires ordered event data.
- Timeout choice affects session granularity.
- `LAG` with a `NULL` gap for the first event must be handled explicitly.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – 30-minute sessionization

```sql
WITH gaps AS (
    SELECT user_id, event_time,
           event_time - LAG(event_time) OVER (
               PARTITION BY user_id ORDER BY event_time
           ) AS gap
    FROM events
),
flags AS (
    SELECT *,
           CASE WHEN gap IS NULL OR gap > INTERVAL '30 minutes' THEN 1 ELSE 0 END AS is_new
    FROM gaps
),
sessions AS (
    SELECT *,
           SUM(is_new) OVER (PARTITION BY user_id ORDER BY event_time) AS session_id
    FROM flags
)
SELECT user_id, session_id, MIN(event_time) AS session_start, MAX(event_time) AS session_end, COUNT(*) AS events
FROM sessions
GROUP BY user_id, session_id
ORDER BY user_id, session_start;
```

**Expected output** (illustrative):

```text
 user_id | session_id |    session_start    |     session_end     | events
---------+------------+---------------------+---------------------+--------
       1 |          1 | 2025-01-01 10:00:00 | 2025-01-01 10:25:00 |      3
       1 |          2 | 2025-01-01 11:30:00 | 2025-01-01 11:45:00 |      2
       2 |          1 | 2025-01-01 09:00:00 | 2025-01-01 09:05:00 |      1
(3 rows)
```

**Why this output appears**: The 65-minute gap between 10:25 and 11:30 exceeds the 30-minute threshold, so a new session (session_id 2) starts. The cumulative sum of `is_new` assigns session IDs incrementally.

##### Example 2: ClickHouse – Sessionization with `LAG`

```sql
SELECT user_id,
       sum(is_new_session) OVER (PARTITION BY user_id ORDER BY event_time) AS session_id,
       min(event_time) AS session_start,
       max(event_time) AS session_end,
       count() AS events
FROM (
    SELECT user_id, event_time,
           CASE WHEN event_time - lagInFrame(event_time) OVER (
               PARTITION BY user_id ORDER BY event_time
           ) > 1800 THEN 1 ELSE 0 END AS is_new_session
    FROM events
)
GROUP BY user_id, session_id;
```

**Expected output**: Similar session-level aggregation.

**Why this output appears**: ClickHouse uses `lagInFrame` (the window `LAG` equivalent); the 1800-second threshold (30 minutes) flags new sessions, and the cumulative sum creates session IDs.

#### Real-World Cases

- **Web analytics**: Session duration and pages per session.
- **Mobile apps**: Session counts and engagement time.
- **IoT**: Grouping sensor readings into activity sessions.
- **Security**: Detecting anomalous activity bursts separated by inactivity.

---

### 6. Cohort Analysis

#### Definitions

**Core Definition**  
Cohort analysis groups users by a shared characteristic (typically signup period) and tracks their behavior over time to measure retention, revenue, or engagement.

**Technical Definition**  
Cohort analysis assigns each user to a cohort based on their first activity (e.g., signup month). It then joins subsequent activity to the cohort, computing the time offset (day, week, month) since cohort entry. Window functions are used to carry cohort size and to mark consecutive activity ranges. The result is a cohort matrix showing metrics (retention rate, revenue) by cohort and period.

**Beginner-Friendly Explanation**  
Cohort analysis answers questions like “Of the users who signed up in January, how many were still active in February, March, and April?” It groups users by when they started and tracks them forward.

#### Purposes

- To measure retention and churn by cohort.
- To compare the long-term value of different cohorts.
- To identify whether product changes improve retention.
- To build cohort triangles for revenue and engagement.

#### Syntax Rules and Structure

**Pattern**:

```sql
WITH cohorts AS (
    SELECT user_id, DATE_TRUNC('month', signup_date) AS cohort_month
    FROM users
),
activity AS (
    SELECT user_id, DATE_TRUNC('month', activity_date) AS activity_month
    FROM events
),
cohort_activity AS (
    SELECT c.cohort_month, a.activity_month,
           DATEDIFF('month', c.cohort_month, a.activity_month) AS month_offset,
           COUNT(DISTINCT a.user_id) AS active_users
    FROM cohorts c
    JOIN activity a ON c.user_id = a.user_id
    GROUP BY 1, 2, 3
),
cohort_sizes AS (
    SELECT cohort_month, COUNT(DISTINCT user_id) AS cohort_size
    FROM cohorts
    GROUP BY 1
)
SELECT ca.cohort_month, ca.month_offset,
       ca.active_users, cs.cohort_size,
       ROUND(ca.active_users * 100.0 / cs.cohort_size, 1) AS retention_pct
FROM cohort_activity ca
JOIN cohort_sizes cs ON ca.cohort_month = cs.cohort_month
ORDER BY ca.cohort_month, ca.month_offset;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `cohorts` | Assigns each user to a cohort month. |
| `activity` | Activity months per user. |
| `cohort_activity` | Joins cohorts to activity and computes month offset. |
| `cohort_sizes` | Number of users in each cohort. |
| Final `SELECT` | Joins sizes and computes retention percentage. |

**Syntax rules**

- Cohort assignment is typically based on the first event (signup, first purchase).
- Activity is grouped by cohort and period offset.
- `DATEDIFF` or date arithmetic computes the offset.
- Window functions can carry cohort size or mark consecutive activity streaks.

**Constraints and limitations**

- Right-censoring: recent cohorts have fewer observed periods; immature cohorts should be filtered or marked.
- Cohort definitions affect results; be explicit about the cohorting event.
- Large event tables may require pre-aggregation for performance.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Monthly retention cohorts

```sql
WITH cohorts AS (
    SELECT user_id, DATE_TRUNC('month', signup_date) AS cohort_month
    FROM users
),
activity AS (
    SELECT user_id, DATE_TRUNC('month', activity_date) AS activity_month
    FROM events
),
retention AS (
    SELECT c.cohort_month,
           (DATE_PART('year', a.activity_month) - DATE_PART('year', c.cohort_month)) * 12
           + (DATE_PART('month', a.activity_month) - DATE_PART('month', c.cohort_month)) AS month_offset,
           COUNT(DISTINCT a.user_id) AS active_users
    FROM cohorts c
    JOIN activity a ON c.user_id = a.user_id
    GROUP BY 1, 2
),
sizes AS (
    SELECT cohort_month, COUNT(DISTINCT user_id) AS cohort_size
    FROM cohorts
    GROUP BY 1
)
SELECT r.cohort_month, r.month_offset,
       r.active_users, s.cohort_size,
       ROUND(r.active_users * 100.0 / s.cohort_size, 1) AS retention_pct
FROM retention r
JOIN sizes s ON r.cohort_month = s.cohort_month
ORDER BY r.cohort_month, r.month_offset;
```

**Expected output** (illustrative):

```text
 cohort_month | month_offset | active_users | cohort_size | retention_pct
--------------+--------------+--------------+-------------+---------------
 2025-01-01   |            0 |          100 |         100 |         100.0
 2025-01-01   |            1 |           60 |         100 |          60.0
 2025-01-01   |            2 |           40 |         100 |          40.0
 2025-02-01   |            0 |          120 |         120 |         100.0
 2025-02-01   |            1 |           72 |         120 |          60.0
(5 rows)
```

**Why this output appears**: Each user is assigned to the month of their signup; activity months are compared to the cohort month to compute the offset. Retention percentage is active users divided by cohort size.

#### Real-World Cases

- **SaaS**: Monthly retention cohorts by signup plan.
- **E-commerce**: Repeat purchase rates by acquisition cohort.
- **Mobile apps**: Day-N retention by install cohort.

---

### 7. Funnel Analysis

#### Definitions

**Core Definition**  
Funnel analysis measures how many users progress through a defined sequence of steps (e.g., view → add to cart → checkout → purchase) and identifies drop-off at each step.

**Technical Definition**  
Funnel analysis assigns each user a step level based on the furthest step completed within a time window. Window functions or aggregate `CASE` expressions count users at each step. Conversion rates are computed as the ratio of users reaching a step to users reaching the previous step. Some engines provide specialized functions (e.g., ClickHouse `windowFunnel`, Apache Doris `WINDOW_FUNNEL`) that compute the maximum completed step per user in a single call.

**Beginner-Friendly Explanation**  
A funnel shows where users drop off: 1000 people viewed a product, 300 added it to the cart, 100 checked out, and 50 purchased. Funnel analysis calculates these counts and percentages.

#### Purposes

- To identify bottlenecks in user journeys.
- To measure the effectiveness of each step.
- To compare funnel performance across segments or time periods.
- To optimize conversion rates.

#### Syntax Rules and Structure

**Standard SQL pattern**:

```sql
WITH step_flags AS (
    SELECT user_id,
           MAX(CASE WHEN event = 'view' THEN 1 ELSE 0 END) AS viewed,
           MAX(CASE WHEN event = 'add_to_cart' THEN 1 ELSE 0 END) AS carted,
           MAX(CASE WHEN event = 'checkout' THEN 1 ELSE 0 END) AS checked_out,
           MAX(CASE WHEN event = 'purchase' THEN 1 ELSE 0 END) AS purchased
    FROM events
    WHERE event_time >= ...
    GROUP BY user_id
)
SELECT COUNT(CASE WHEN viewed = 1 THEN 1 END) AS step1,
       COUNT(CASE WHEN carted = 1 THEN 1 END) AS step2,
       COUNT(CASE WHEN checked_out = 1 THEN 1 END) AS step3,
       COUNT(CASE WHEN purchased = 1 THEN 1 END) AS step4,
       ROUND(COUNT(CASE WHEN carted = 1 THEN 1 END) * 100.0
             / NULLIF(COUNT(CASE WHEN viewed = 1 THEN 1 END), 0), 2) AS view_to_cart_pct,
       ...
FROM step_flags;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `MAX(CASE WHEN event = ...)` | Flags whether the user reached each step. |
| `COUNT(CASE WHEN flag = 1)` | Counts users at each step. |
| Conversion ratio | Users at step N / users at step N-1. |

**Syntax rules**

- Steps must be defined in order.
- A user is counted at a step if they performed the event at any time (or within a window).
- Conversion rates use `NULLIF` to avoid division by zero.
- Specialized functions (e.g., ClickHouse `windowFunnel`) compute the maximum step level per user.

**Constraints and limitations**

- Strict vs. non-strict funnels: strict funnels require steps in order; non-strict funnels only require presence.
- Time windows affect results.
- Users can enter the funnel at different points.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: ClickHouse – Multi-step funnel with `windowFunnel`

```sql
SELECT level, count() AS users,
       round(count() * 100.0 / max(count()) OVER (), 2) AS pct_of_top
FROM (
    SELECT user_id,
           windowFunnel(86400)( -- 24-hour window
               ts,
               event = 'page_view',
               event = 'add_to_cart',
               event = 'checkout',
               event = 'purchase'
           ) AS level
    FROM events
    WHERE ts >= today() - 30
    GROUP BY user_id
)
GROUP BY level
ORDER BY level DESC;
```

**Expected output** (illustrative):

```text
level users pct_of_top
----- ----- ----------
4     50    100.00
3     100   50.00
2     300   33.33
1     1000  10.00
```

**Why this output appears**: `windowFunnel` returns the maximum step level reached by each user within a 24-hour window. Counting users at each level and dividing by the top level gives the conversion percentages.

##### Example 2: Standard SQL – Step-by-step drop-off

```sql
WITH funnel AS (
    SELECT user_id,
           COUNT(CASE WHEN event = 'view' THEN 1 END) AS viewed,
           COUNT(CASE WHEN event = 'add_to_cart' THEN 1 END) AS carted,
           COUNT(CASE WHEN event = 'checkout' THEN 1 END) AS checked_out,
           COUNT(CASE WHEN event = 'purchase' THEN 1 END) AS purchased
    FROM events
    GROUP BY user_id
)
SELECT COUNT(CASE WHEN viewed > 0 THEN 1 END) AS step1_views,
       COUNT(CASE WHEN carted > 0 THEN 1 END) AS step2_cart,
       COUNT(CASE WHEN checked_out > 0 THEN 1 END) AS step3_checkout,
       COUNT(CASE WHEN purchased > 0 THEN 1 END) AS step4_purchase,
       ROUND(COUNT(CASE WHEN carted > 0 THEN 1 END) * 100.0
             / NULLIF(COUNT(CASE WHEN viewed > 0 THEN 1 END), 0), 2) AS view_to_cart_pct
FROM funnel;
```

**Expected output**:

```text
step1_views step2_cart step3_checkout step4_purchase view_to_cart_pct
----------- ---------- -------------- -------------- ----------------
1000        300        100            50             30.00
```

**Why this output appears**: The CTE flags each user’s step completion; the outer query counts users at each step and computes the view-to-cart conversion rate.

#### Real-World Cases

- **E-commerce**: Purchase funnel optimization.
- **SaaS**: Signup → activation → upgrade funnel.
- **Marketing**: Ad impression → click → landing page → conversion.

---

### 8. Retention Analysis

#### Definitions

**Core Definition**  
Retention analysis measures the proportion of users from a cohort who remain active in subsequent periods.

**Technical Definition**  
Retention analysis builds on cohort analysis by defining “active” for each period (e.g., any event in a month) and computing the retention rate as active users in period N divided by cohort size. Window functions are used to carry cohort size, mark consecutive activity, and compute period offsets. Retention curves and triangles visualize how retention decays over time.

**Beginner-Friendly Explanation**  
Retention analysis tells you “of the users who signed up in January, what percentage came back in February, March, and so on?” It is a key metric for product health.

#### Purposes

- To measure product stickiness and engagement.
- To compare retention across cohorts and acquisition channels.
- To identify the “retention curve” and find a stable long-term rate.
- To evaluate the impact of product changes on retention.

#### Syntax Rules and Structure

**Pattern**:

```sql
WITH cohort_sizes AS (
    SELECT DATE_TRUNC('month', signup_date) AS cohort, COUNT(DISTINCT user_id) AS size
    FROM users GROUP BY 1
),
activity AS (
    SELECT user_id, DATE_TRUNC('month', activity_date) AS period
    FROM events GROUP BY 1, 2
),
retention AS (
    SELECT c.cohort,
           DATEDIFF('month', c.cohort, a.period) AS period_offset,
           COUNT(DISTINCT a.user_id) AS retained
    FROM users u
    JOIN cohort_sizes c ON DATE_TRUNC('month', u.signup_date) = c.cohort
    JOIN activity a ON u.user_id = a.user_id
    GROUP BY 1, 2
)
SELECT r.cohort, r.period_offset,
       r.retained, cs.size,
       ROUND(r.retained * 100.0 / cs.size, 2) AS retention_rate
FROM retention r
JOIN cohort_sizes cs ON r.cohort = cs.cohort
ORDER BY r.cohort, r.period_offset;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `cohort_sizes` | Size of each signup cohort. |
| `activity` | Periods in which users were active. |
| `retention` | Retained users by cohort and period offset. |
| Final `SELECT` | Computes retention rate. |

**Syntax rules**

- “Active” must be defined explicitly (e.g., any event in a period).
- Period offset is computed from the cohort start.
- Retention rate = retained users / cohort size.
- Right-censoring: immature cohorts should be filtered or flagged.

**Constraints and limitations**

- Retention rates for recent periods may be incomplete.
- Cohort definitions and activity definitions strongly affect results.
- Large event data may require pre-aggregation.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Day-N retention

```sql
WITH retention_base AS (
    SELECT u.user_id, u.signup_date, a.activity_date,
           a.activity_date - u.signup_date AS days_since_signup
    FROM users u
    JOIN activity a ON u.user_id = a.user_id
),
cohort_size AS (
    SELECT DATE_TRUNC('month', signup_date) AS cohort_month,
           COUNT(DISTINCT user_id) AS cohort_users
    FROM users
    GROUP BY 1
),
retention_by_day AS (
    SELECT DATE_TRUNC('month', signup_date) AS cohort_month,
           days_since_signup,
           COUNT(DISTINCT user_id) AS retained_users
    FROM retention_base
    GROUP BY 1, 2
)
SELECT r.cohort_month, r.days_since_signup,
       r.retained_users, c.cohort_users,
       ROUND(r.retained_users * 100.0 / c.cohort_users, 1) AS retention_pct
FROM retention_by_day r
JOIN cohort_size c ON r.cohort_month = c.cohort_month
WHERE r.days_since_signup IN (1, 3, 7, 14, 30)
ORDER BY r.cohort_month, r.days_since_signup;
```

**Expected output** (illustrative):

```text
 cohort_month | days_since_signup | retained_users | cohort_users | retention_pct
--------------+-------------------+----------------+--------------+---------------
 2025-01-01   |                 1 |             80 |          100 |          80.0
 2025-01-01   |                 3 |             65 |          100 |          65.0
 2025-01-01   |                 7 |             50 |          100 |          50.0
 2025-01-01   |                14 |             40 |          100 |          40.0
 2025-01-01   |                30 |             30 |          100 |          30.0
(5 rows)
```

**Why this output appears**: Users are grouped by signup month; activity dates are compared to signup dates to compute day offsets. Retention percentage is retained users at day N divided by cohort size.

#### Real-World Cases

- **Subscription services**: Monthly renewal rates.
- **Mobile apps**: Day-1, Day-7, Day-30 retention.
- **E-commerce**: Repeat purchase rates by cohort.

---

### 9. Period-over-Period Comparisons

#### Definitions

**Core Definition**  
Period-over-period comparisons measure the change in a metric between two corresponding time periods, such as month-over-month or year-over-year.

**Technical Definition**  
Period-over-period comparisons use `LAG` (or `LEAD`) with an appropriate offset to retrieve the value from a prior period. For month-over-month, the offset is 1 row; for year-over-year on monthly data, the offset is 12 rows. The comparison metric is typically the difference or percentage change. `PARTITION BY` is used to compare within the same entity (e.g., product, region), and the `ORDER BY` must be on the time column.

**Beginner-Friendly Explanation**  
Period-over-period comparison tells you “how much did sales grow compared to last month?” or “how does this quarter compare to the same quarter last year?” It uses `LAG` to fetch the previous period’s value and computes the change.

#### Purposes

- To measure growth trends (MoM, QoQ, YoY).
- To compare performance across time periods.
- To identify seasonality and anomalies.
- To support financial and KPI reporting.

#### Syntax Rules and Structure

**Pattern**:

```sql
SELECT period, value,
       LAG(value, offset) OVER (
           PARTITION BY entity ORDER BY period
       ) AS prior_value,
       value - LAG(value, offset) OVER (
           PARTITION BY entity ORDER BY period
       ) AS change
FROM metrics;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `LAG(value, offset)` | Retrieves the prior period’s value. |
| `offset` | Number of periods back (1 for MoM, 12 for YoY on monthly data). |
| `PARTITION BY entity` | Compares within the same entity. |
| `ORDER BY period` | Ensures chronological order. |

**Syntax rules**

- `ORDER BY` must be on the time column.
- `LAG` counts rows, not calendar periods; missing periods shift the comparison.
- For irregular time series, use a date spine or `RANGE` frame.
- Percentage change = `(value - prior_value) / NULLIF(prior_value, 0)`.

**Constraints and limitations**

- Missing periods cause incorrect comparisons if `LAG` is used with row offsets.
- `LAG` with offset 12 on monthly data assumes every month is present.
- Calendar effects (e.g., 4-4-5 retail calendars) may require special handling.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Oracle – Year-over-year comparison with `LAG`

```sql
SELECT product, week, weekly_ytd_sales,
       LAG(weekly_ytd_sales, 1) OVER (
           PARTITION BY product, week
           ORDER BY yr
       ) AS prior_year_weekly_ytd_sales
FROM year_over_year_sales
WHERE yr = 2001;
```

**Expected output** (illustrative):

```text
PRODUCT WEEK WEEKLY_YTD_SALES PRIOR_YEAR_WEEKLY_YTD_SALES
------- ---- --------------- ---------------------------
Y Box   37   24616.85        95433.09
Y Box   38   37739.65        107726.96
```

**Why this output appears**: `LAG(..., 1) OVER (PARTITION BY product, week ORDER BY yr)` pairs the same week across years; the prior year’s weekly year-to-date sales is returned for comparison.

##### Example 2: PostgreSQL – Month-over-month growth

```sql
WITH monthly AS (
    SELECT DATE_TRUNC('month', sale_date) AS month, SUM(amount) AS revenue
    FROM sales
    GROUP BY 1
)
SELECT month, revenue,
       LAG(revenue) OVER (ORDER BY month) AS prev_revenue,
       revenue - LAG(revenue) OVER (ORDER BY month) AS mom_change,
       ROUND((revenue - LAG(revenue) OVER (ORDER BY month)) * 100.0
             / NULLIF(LAG(revenue) OVER (ORDER BY month), 0), 2) AS mom_pct
FROM monthly
ORDER BY month;
```

**Expected output**:

```text
   month    | revenue | prev_revenue | mom_change | mom_pct
------------+---------+--------------+------------+---------
 2025-01-01 |   10000 |              |            |
 2025-02-01 |   12000 |        10000 |       2000 |   20.00
 2025-03-01 |   11000 |        12000 |      -1000 |   -8.33
(3 rows)
```

**Why this output appears**: `LAG(revenue)` retrieves the previous month’s revenue; the change and percentage change are computed. The first month has no prior value.

#### Real-World Cases

- **Financial reporting**: YoY revenue growth.
- **Marketing**: MoM lead generation.
- **Operations**: Week-over-week production output.

---

### 10. Sequential Event Analysis

#### Definitions

**Core Definition**  
Sequential event analysis detects patterns in ordered event streams, such as consecutive occurrences, specific sequences, or time-ordered relationships between events.

**Technical Definition**  
Sequential event analysis uses window functions (`LAG`, `LEAD`, `ROW_NUMBER`) to compare an event with adjacent events and to assign sequence numbers within sessions or groups. For complex pattern matching, engines may provide `MATCH_RECOGNIZE` (SQL:2016) or similar functions that match regular-expression-like patterns across ordered rows. Common patterns include detecting runs, V-shapes, and multi-step sequences.

**Beginner-Friendly Explanation**  
Sequential event analysis looks for patterns in the order of events—like “did the user view the page, then add to cart, then purchase?” or “are there three consecutive failed login attempts?”

#### Purposes

- To detect event sequences that indicate a business outcome.
- To identify runs of consecutive values.
- To validate that events occur in the correct order.
- To support fraud detection and user behavior analysis.

#### Syntax Rules and Structure

**Standard SQL pattern**:

```sql
SELECT user_id, event_time, event_type,
       LAG(event_type) OVER (
           PARTITION BY user_id ORDER BY event_time
       ) AS prev_event,
       LEAD(event_type) OVER (
           PARTITION BY user_id ORDER BY event_time
       ) AS next_event
FROM events;
```

**`MATCH_RECOGNIZE` pattern** (Databricks, Oracle, etc.):

```sql
SELECT *
FROM events
MATCH_RECOGNIZE (
    PARTITION BY user_id
    ORDER BY event_time
    PATTERN (A B C)
    DEFINE A AS event_type = 'view',
           B AS event_type = 'add_to_cart',
           C AS event_type = 'purchase'
);
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `LAG`/`LEAD` | Compare adjacent events. |
| `ROW_NUMBER` | Assign sequence numbers within a session. |
| `MATCH_RECOGNIZE` | Match row patterns (vendor-specific). |

**Syntax rules**

- `ORDER BY` must be on the event time to define sequence.
- `PARTITION BY` resets the sequence per entity.
- `MATCH_RECOGNIZE` requires a `PATTERN` and `DEFINE` clause.
- Standard SQL lacks a universal pattern-matching clause; vendors implement their own.

**Constraints and limitations**

- `MATCH_RECOGNIZE` is not available in all engines.
- Complex patterns may require multiple CTEs in engines without `MATCH_RECOGNIZE`.
- `LAG`/`LEAD` only compare adjacent rows; non-adjacent patterns require additional logic.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Detecting view → cart → purchase sequence

```sql
WITH ordered AS (
    SELECT user_id, event_time, event_type,
           LAG(event_type, 1) OVER (PARTITION BY user_id ORDER BY event_time) AS prev1,
           LAG(event_type, 2) OVER (PARTITION BY user_id ORDER BY event_time) AS prev2
    FROM events
)
SELECT user_id, event_time
FROM ordered
WHERE event_type = 'purchase'
  AND prev1 = 'add_to_cart'
  AND prev2 = 'view';
```

**Expected output** (illustrative):

```text
 user_id |     event_time
---------+---------------------
       1 | 2025-01-01 10:30:00
       2 | 2025-01-01 11:15:00
```

**Why this output appears**: `LAG` retrieves the two preceding events for each purchase. Rows where the sequence is exactly view → add_to_cart → purchase are returned.

##### Example 2: Databricks – `MATCH_RECOGNIZE` for V-shape detection

```sql
SELECT *
FROM stock_prices
MATCH_RECOGNIZE (
    PARTITION BY symbol
    ORDER BY trade_date
    PATTERN (A B+ C)
    DEFINE A AS price > LAG(price),
           B AS price < LAG(price),
           C AS price > LAG(price)
);
```

**Expected output** (illustrative): Rows forming a V-shaped price movement.

**Why this output appears**: The pattern `(A B+ C)` matches a price increase (A), followed by one or more decreases (B+), followed by an increase (C), which is a V-shape.

#### Real-World Cases

- **E-commerce**: Detecting the exact sequence of views, carts, and purchases.
- **Security**: Three consecutive failed logins followed by a successful login.
- **Finance**: V-shaped or W-shaped price patterns.

---

## References

- PostgreSQL Documentation – `SELECT` (WINDOW Clause) – https://www.postgresql.org/docs/current/sql-select.html
- PostgreSQL Documentation – Window Functions – https://www.postgresql.org/docs/current/functions-window.html
- MySQL 8.0 Reference Manual – Window Function Concepts and Syntax – https://dev.mysql.com/doc/refman/8.0/en/window-functions-usage.html
- Microsoft Learn – Gaps and Islands Problem – https://learn.microsoft.com/en-us/archive/technet-wiki/18399.t-sql-gaps-and-islands-problem
- Microsoft Learn – `MATCH_RECOGNIZE` Clause (Databricks SQL) – https://learn.microsoft.com/en-us/azure/databricks/sql/language-manual/sql-ref-syntax-qry-select-match-recognize
- Oracle Data Warehousing Guide – Period-to-Period Comparison – https://docs.oracle.com/en/database/oracle/oracle-database/21/dwhsg/sql-analysis-reporting-data-warehouses.html
- ClickHouse Documentation – `windowFunnel` – https://clickhouse.com/docs/en/sql-reference/aggregate-functions/parametric-functions#windowfunnel
- ClickHouse Documentation – `retention` – https://clickhouse.com/docs/en/sql-reference/aggregate-functions/parametric-functions#retention
- Singdata – Retention and Cohort Analysis Guide – http://www.singdata.com/documents/sql_retention_cohort_guide
- GitHub – Sessionization with Window Functions – https://github.com/sergiorgiraldo/Python-lang/blob/master/data-engineering-interview-patterns/sql/01_window_functions/de_scenarios/sessionization.sql
- GitHub – Gaps and Islands: Detect Consecutive Ranges – https://raw.githubusercontent.com/ (search result snippet)
- SQL:2016 Standard – `MATCH_RECOGNIZE` (ISO/IEC 9075-2:2016)
- Use The Index, Luke – Window Functions and Indexing – https://use-the-index-luke.com/sql/partial-results/window-functions