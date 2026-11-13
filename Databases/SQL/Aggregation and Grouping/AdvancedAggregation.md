# SQL Advanced Aggregation: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Advanced aggregation encompasses the SQL techniques and constructs that extend basic `GROUP BY` summarisation—including conditional, distinct, ordered-set, hypothetical-set, user-defined, filtered, and window-based aggregation—to express complex analytical logic in a single query.

**Technical Definition**
In SQL, advanced aggregation refers to the use of aggregate expressions beyond simple `SUM`/`COUNT`/`AVG`/`MIN`/`MAX` over grouped rows. It includes: conditional aggregation via `CASE` expressions inside aggregates; multiple aggregates computed over the same group; `DISTINCT` aggregation to eliminate duplicates before aggregation; ordered-set aggregates (`PERCENTILE_CONT`, `PERCENTILE_DISC`, `MODE`) that depend on the sort order of input rows; hypothetical-set aggregates (`RANK`, `DENSE_RANK`, `PERCENT_RANK`, `CUME_DIST`) that compute the rank a hypothetical row would receive; user-defined aggregates created with `CREATE AGGREGATE`; the `FILTER` clause for per-aggregate row filtering; and window aggregates (`OVER` clause) that compute aggregates across partitions while preserving individual rows. These constructs are standardised in part by SQL:2003/2008/2016, with significant vendor-specific extensions in PostgreSQL, MySQL, and SQL Server.

**Beginner-Friendly Explanation**
Basic aggregation gives you one summary per group. Advanced aggregation lets you ask much richer questions in a single query: “How many high-value and low-value orders were there per region?” (conditional aggregation), “What is the median and the 95th percentile response time?” (ordered-set aggregates), “What would this customer's rank be among all customers?” (hypothetical-set aggregates), or “What is the running total per customer?” (window aggregates). It is the difference between a simple report and a full analytical engine inside SQL.

---

### Key Characteristics

1. **Single-Pass Efficiency**: Many advanced aggregation techniques compute multiple metrics in a single pass over the data, avoiding repeated scans or self-joins.
2. **Order Sensitivity**: Ordered-set and hypothetical-set aggregates require the `WITHIN GROUP (ORDER BY ...)` syntax and produce results that depend on the sorted order of input rows.
3. **Filter Granularity**: The `FILTER` clause (PostgreSQL, SQL Server) restricts individual aggregates to subsets of rows within the same group, without affecting other aggregates in the same `SELECT` list.
4. **Row Preservation**: Window aggregates (`OVER` clause) preserve individual rows while adding aggregate values, unlike `GROUP BY` which collapses rows.
5. **Vendor Divergence**: PostgreSQL has the richest support (ordered-set, hypothetical-set, `FILTER`, user-defined aggregates, `ARRAY_AGG`, `JSON_AGG`). MySQL 8.0+ supports window functions and conditional aggregation but lacks ordered-set aggregates and `FILTER`. SQL Server supports `FILTER` (2022+), window functions, and `STRING_AGG` but lacks `ARRAY_AGG` and ordered-set aggregates in `WITHIN GROUP` form.
6. **Performance Trade-offs**: `DISTINCT` aggregation is generally more expensive than non-distinct aggregation because it requires deduplication (sorting or hashing). Window functions can be more efficient than self-joins for running totals and moving averages, but partitioning choices significantly affect performance.

---

### Prerequisites

- **Basic aggregation**: `GROUP BY`, `HAVING`, and core aggregate functions.
- **`CASE` expressions**: Conditional logic inside aggregate calls.
- **Window function concepts**: `OVER`, `PARTITION BY`, `ORDER BY`, and frame clauses.
- **Subqueries and CTEs**: For structuring advanced reporting queries.
- **NULL semantics**: Three-valued logic and how aggregates treat `NULL`.
- **Index and execution plan fundamentals**: To reason about performance.

---

### Related Programming Areas

| Area | Connection |
|------|------------|
| **Business Intelligence & Reporting** | Conditional aggregation pivots rows to columns; window functions compute running totals and rankings. |
| **Statistical Analysis** | Ordered-set aggregates provide medians and percentiles; hypothetical-set aggregates support rank-based analysis. |
| **Data Warehousing** | `GROUPING SETS`, `ROLLUP`, `CUBE`, and window aggregates power OLAP cubes and dashboards. |
| **Query Optimisation** | `DISTINCT` aggregation, window partitioning, and `FILTER` placement affect execution plans. |
| **Extensibility** | User-defined aggregates allow domain-specific summarisation (e.g., geometric, financial, or scientific aggregates). |

---

### Core Concepts / Features

1. **Conditional Aggregation**
2. **Multiple Aggregates**
3. **Distinct Aggregation**
4. **Ordered-Set Aggregates**
5. **Hypothetical-Set Aggregates**
6. **User-Defined Aggregates**
7. **Aggregate `FILTER` Clause**
8. **Window Aggregates**
9. **Aggregate over Partitions**
10. **Advanced Reporting Queries**

---

## 1. Conditional Aggregation

### Definitions

**Core Definition**
Conditional aggregation computes aggregate values over subsets of rows defined by a `CASE` expression inside the aggregate call, typically to pivot rows into columns.

**Technical Definition**
Conditional aggregation wraps a `CASE` expression inside an aggregate function: `SUM(CASE WHEN condition THEN value ELSE 0 END)` or `COUNT(CASE WHEN condition THEN 1 END)`. The `CASE` expression returns a value for rows matching the condition and `NULL` (or `0`) otherwise; the aggregate then operates only on the non-`NULL` (or non-zero) results. This allows multiple conditional metrics to be computed in a single `GROUP BY` query, effectively performing a pivot operation without `PIVOT` syntax.

**Beginner-Friendly Explanation**
Conditional aggregation lets you say “count only the orders where status = 'completed'” or “sum only the amounts for the Electronics category” while still grouping by something else. It is like having a separate filter for each column in your report.

---

### Purposes

- To pivot row-level categories into columns in a single query.
- To compute multiple conditional metrics without multiple scans or self-joins.
- To replace `PIVOT`/`UNPIVOT` operators where they are unavailable.
- To build cross-tabulation reports directly in SQL.

---

### Syntax Rules and Structure

```
aggregate_function ( CASE WHEN condition THEN value ELSE fallback END )
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `aggregate_function` | `SUM`, `COUNT`, `AVG`, `MIN`, `MAX`, etc. |
| `CASE WHEN condition` | Row-level predicate that determines whether the row contributes. |
| `THEN value` | Value passed to the aggregate when the condition is true. |
| `ELSE fallback` | Value when false; use `0` for `SUM`, `NULL` for `COUNT` (count ignores `NULL`). |

**Syntax Rules**

1. For `SUM`, use `ELSE 0` to avoid `NULL`s contributing nothing but still allowing the sum to be computed.
2. For `COUNT`, use `COUNT(CASE WHEN condition THEN 1 END)`—the implicit `ELSE NULL` is ignored by `COUNT`.
3. `CASE` expressions can be nested and combined.
4. PostgreSQL also supports the `FILTER` clause as a cleaner alternative (Section 7).

**Constraints and Limitations**

- MySQL 8.0+ supports window functions, but conditional aggregation with `CASE` is the portable approach across all three RDBMS.
- Conditional aggregation can increase CPU cost if many conditional columns are computed over large groups.

---

### Annotated Code Examples

**Example 1: Pivot Orders by Status**

```sql
SELECT
  customer_id,
  SUM(CASE WHEN status = 'completed' THEN amount ELSE 0 END) AS completed_total,
  SUM(CASE WHEN status = 'pending'   THEN amount ELSE 0 END) AS pending_total,
  COUNT(CASE WHEN status = 'completed' THEN 1 END) AS completed_count
FROM orders
GROUP BY customer_id;
```

**Expected Output (illustrative)**

| customer_id | completed_total | pending_total | completed_count |
|-------------|-----------------|---------------|-----------------|
| 101         | 500             | 200           | 3               |
| 102         | 0               | 750           | 0               |

**Why This Output Occurs**: The `CASE` expressions route each row's `amount` to the appropriate column based on `status`; the aggregates then sum or count the routed values per customer.

---

**Example 2: FILTER Alternative (PostgreSQL)**

```sql
SELECT
  customer_id,
  SUM(amount) FILTER (WHERE status = 'completed') AS completed_total,
  SUM(amount) FILTER (WHERE status = 'pending')   AS pending_total,
  COUNT(*)    FILTER (WHERE status = 'completed') AS completed_count
FROM orders
GROUP BY customer_id;
```

**Explanation**: The `FILTER` clause achieves the same result more concisely and is the preferred PostgreSQL idiom for conditional aggregation .

---

### Real-World Cases

- **Sales dashboards**: Revenue by region for each product category.
- **Financial reporting**: Debits and credits in separate columns.
- **Survey analysis**: Counts of responses by answer choice per question.

---

## 2. Multiple Aggregates

### Definitions

**Core Definition**
Multiple aggregates refer to the computation of two or more aggregate functions—possibly of different types and over different expressions—within the same `GROUP BY` query.

**Technical Definition**
A single `SELECT` statement with `GROUP BY` may contain any number of aggregate expressions in the `SELECT` list and `HAVING` clause. The query optimiser computes all aggregates in a single pass over the grouped data (when possible), avoiding repeated scans. Multiple aggregates can combine simple aggregates (`SUM`, `COUNT`, `AVG`), conditional aggregates, `DISTINCT` aggregates, and statistical aggregates in one query.

**Beginner-Friendly Explanation**
Instead of running separate queries for total sales, average order value, and number of orders, you can compute all of them at once in a single query.

---

### Purposes

- To compute a comprehensive set of metrics in a single data pass.
- To reduce I/O and query execution time compared to multiple separate queries.
- To support dashboards and reports that display many KPIs side by side.
- To combine different aggregation levels (e.g., `SUM` and `AVG`) for the same group.

---

### Syntax Rules and Structure

```
SELECT grouping_column,
       aggregate_1(expression_1),
       aggregate_2(expression_2),
       ...
FROM table
GROUP BY grouping_column;
```

**Syntax Rules**

1. Each aggregate is independent; they may reference different columns or expressions.
2. `COUNT(*)`, `COUNT(column)`, `COUNT(DISTINCT column)`, `SUM`, `AVG`, `MIN`, `MAX`, and statistical aggregates can all coexist.
3. The optimiser generally computes all aggregates in one pass, but `DISTINCT` aggregates may require additional processing (Section 3).

**Constraints and Limitations**

- Many aggregates, especially with `DISTINCT` or complex expressions, increase memory and CPU usage.
- SQL Server's `COUNT` returns `int`; use `COUNT_BIG` for large tables.

---

### Annotated Code Examples

**Example 1: Multiple Aggregates in One Query**

```sql
SELECT
  product_category,
  COUNT(*)              AS order_count,
  SUM(amount)           AS total_revenue,
  AVG(amount)           AS avg_order_value,
  MIN(amount)           AS smallest_order,
  MAX(amount)           AS largest_order,
  STDDEV(amount)        AS revenue_stddev
FROM orders
GROUP BY product_category;
```

**Expected Output (illustrative)**

| product_category | order_count | total_revenue | avg_order_value | smallest_order | largest_order | revenue_stddev |
|------------------|-------------|---------------|-----------------|----------------|---------------|----------------|
| Electronics      | 120         | 45000         | 375             | 50             | 2000          | 312.5          |
| Furniture        | 80          | 62000         | 775             | 200            | 3500          | 580.2          |

**Why This Output Occurs**: All seven aggregates are computed for each product category in a single pass over the grouped data.

---

### Real-World Cases

- **Executive dashboards**: Revenue, order count, average order value, and growth rate per region.
- **Operational monitoring**: Average, maximum, and standard deviation of response times per service.
- **Financial analysis**: Total, minimum, maximum, and variance of transactions per account.

---

## 3. Distinct Aggregation

### Definitions

**Core Definition**
Distinct aggregation computes an aggregate over the distinct values of an expression, eliminating duplicates before the aggregate is applied.

**Technical Definition**
`aggregate(DISTINCT expression)` first removes duplicate values of `expression` within each group, then applies the aggregate function to the remaining unique values. Common forms include `COUNT(DISTINCT column)`, `SUM(DISTINCT column)`, and `AVG(DISTINCT column)`. Distinct aggregation typically requires a sort or hash-based deduplication step, making it more expensive than non-distinct aggregation. `COUNT(DISTINCT)` can disable parallel aggregation in PostgreSQL because per-partition distinct counts cannot simply be summed.

**Beginner-Friendly Explanation**
Distinct aggregation answers questions like “how many unique customers placed an order?” rather than “how many orders were placed?” It counts each value only once.

---

### Purposes

- To count unique entities (unique visitors, distinct products sold).
- To compute averages or sums over unique values only.
- To measure cardinality within groups.
- To support deduplication-aware reporting.

---

### Syntax Rules and Structure

```
aggregate_function ( DISTINCT expression )
```

**Syntax Rules**

1. `DISTINCT` is placed immediately after the opening parenthesis of the aggregate.
2. `COUNT(DISTINCT expression)` ignores `NULL` values (as `COUNT(expression)` does).
3. `DISTINCT` is not supported for all aggregates; `COUNT`, `SUM`, and `AVG` are the most common.
4. `DISTINCT` cannot be combined with an internal `ORDER BY` clause that references expressions not in the `DISTINCT` list (PostgreSQL restriction).

**Constraints and Limitations**

- **Performance**: Distinct aggregation generally requires more work than non-distinct aggregation . `COUNT(DISTINCT)` can disable parallel aggregation in PostgreSQL.
- A hash-based rewrite for `COUNT(DISTINCT)` may slow down queries that also contain other aggregates because the hash distinct cannot be split into partial and merge phases .

---

### Annotated Code Examples

**Example 1: `COUNT(DISTINCT)`**

```sql
SELECT
  region,
  COUNT(*)                    AS total_orders,
  COUNT(DISTINCT customer_id) AS unique_customers
FROM orders
GROUP BY region;
```

**Expected Output (illustrative)**

| region | total_orders | unique_customers |
|--------|--------------|------------------|
| East   | 500          | 120              |
| West   | 350          | 95               |

**Why This Output Occurs**: `COUNT(*)` counts every order; `COUNT(DISTINCT customer_id)` counts each customer only once per region.

---

**Example 2: `SUM(DISTINCT)` and `AVG(DISTINCT)`**

```sql
SELECT
  SUM(DISTINCT amount) AS sum_unique_amounts,
  AVG(DISTINCT amount) AS avg_unique_amounts
FROM orders;
```

**Explanation**: Duplicate amounts are eliminated before summing or averaging, which may produce different results from `SUM(amount)` and `AVG(amount)`.

---

### Real-World Cases

- **Marketing**: Unique visitors per campaign.
- **Retail**: Distinct products sold per store.
- **Healthcare**: Unique patients per diagnosis.

---

## 4. Ordered-Set Aggregates

### Definitions

**Core Definition**
Ordered-set aggregates compute a result that depends on the sorted order of the input rows, using the `WITHIN GROUP (ORDER BY ...)` syntax.

**Technical Definition**
Ordered-set aggregate functions take a set of aggregated arguments specified in the `WITHIN GROUP (ORDER BY ...)` clause, rather than in the normal function argument list. PostgreSQL implements the SQL:2008 ordered-set aggregates, including `PERCENTILE_CONT`, `PERCENTILE_DISC`, and `MODE`. `PERCENTILE_CONT` interpolates between adjacent values; `PERCENTILE_DISC` returns an actual input value. These aggregates are not eligible for partial (parallel) mode.

**Beginner-Friendly Explanation**
Ordered-set aggregates are functions that need their input sorted before they can compute an answer—like finding the median (middle value) or the most frequent value.

---

### Purposes

- To compute medians, quartiles, and other percentiles.
- To find the most frequent value (`MODE`).
- To support distribution-aware reporting (e.g., 95th percentile latency).
- To replace complex self-joins or subqueries with built-in ordered-set functions.

---

### Syntax Rules and Structure

**General Syntax**
```
ordered_set_aggregate ( direct_args ) WITHIN GROUP ( ORDER BY sorted_args )
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `ordered_set_aggregate` | `PERCENTILE_CONT`, `PERCENTILE_DISC`, `MODE`, etc. |
| `direct_args` | Arguments evaluated once per group (e.g., the fraction for percentiles). |
| `WITHIN GROUP (ORDER BY ...)` | Defines the sorted input set for the aggregate. |

**Syntax Rules**

1. The `WITHIN GROUP` clause is required for ordered-set aggregates.
2. `PERCENTILE_CONT(fraction)` and `PERCENTILE_DISC(fraction)` require `fraction` between 0 and 1.
3. `MODE()` takes no direct arguments; its input is specified entirely in `WITHIN GROUP`.
4. Ordered-set aggregates are **not** eligible for partial mode (parallel aggregation) .

**Constraints and Limitations**

- PostgreSQL supports ordered-set aggregates; MySQL does not.
- SQL Server implements `PERCENTILE_CONT` and `PERCENTILE_DISC` as window functions (not `WITHIN GROUP` aggregates).
- Interpolation in `PERCENTILE_CONT` may produce values not present in the input.

---

### Annotated Code Examples

**Example 1: Median with `PERCENTILE_CONT`**

```sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY income) AS median_income
FROM households;
```

**Expected Output (illustrative)**

| median_income |
|---------------|
| 50489         |

**Why This Output Occurs**: `PERCENTILE_CONT(0.5)` computes the 50th percentile, interpolating between the two middle values if necessary .

---

**Example 2: 95th Percentile with `PERCENTILE_DISC`**

```sql
SELECT
  PERCENTILE_DISC(0.95) WITHIN GROUP (ORDER BY response_ms) AS p95_response
FROM api_requests;
```

**Expected Output (illustrative)**

| p95_response |
|--------------|
| 1250         |

**Why This Output Occurs**: `PERCENTILE_DISC` returns the first actual input value whose cumulative distribution is at least 95% .

---

**Example 3: Mode**

```sql
SELECT
  MODE() WITHIN GROUP (ORDER BY product_name) AS most_common_product
FROM orders;
```

**Expected Output (illustrative)**

| most_common_product |
|---------------------|
| Widget              |

**Why This Output Occurs**: `MODE` returns the most frequent value in the sorted input set .

---

### Real-World Cases

- **SLA monitoring**: 95th and 99th percentile latency.
- **Compensation analysis**: Median salary by department.
- **Quality control**: Median defect rate per production line.

---

## 5. Hypothetical-Set Aggregates

### Definitions

**Core Definition**
Hypothetical-set aggregates compute the rank or distribution that a hypothetical row would receive if it were added to the sorted input set.

**Technical Definition**
Hypothetical-set aggregates take direct arguments (the hypothetical row's values) and a `WITHIN GROUP (ORDER BY ...)` clause (the sorted input set). PostgreSQL implements `RANK`, `DENSE_RANK`, `PERCENT_RANK`, and `CUME_DIST` as hypothetical-set aggregates. The result is the value the corresponding window function would return for the hypothetical row. Unlike most built-in aggregates, these aggregates are **not strict**—they do not drop input rows containing `NULL`s.

**Beginner-Friendly Explanation**
Hypothetical-set aggregates answer questions like “if this new employee had salary X, what would their rank be among current employees?” without actually inserting the row.

---

### Purposes

- To determine the rank a hypothetical value would achieve.
- To compute percentile ranks and cumulative distributions for hypothetical values.
- To support “what-if” analysis directly in SQL.
- To avoid inserting temporary rows for rank comparisons.

---

### Syntax Rules and Structure

**General Syntax**
```
hypothetical_set_aggregate ( direct_args ) WITHIN GROUP ( ORDER BY sorted_args )
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `hypothetical_set_aggregate` | `RANK`, `DENSE_RANK`, `PERCENT_RANK`, `CUME_DIST`. |
| `direct_args` | Values of the hypothetical row. |
| `sorted_args` | The sorted input set to compare against. |

**Syntax Rules**

1. The direct arguments must match the number and types of the aggregated arguments in `WITHIN GROUP`.
2. These aggregates are not strict; `NULL`s are not automatically dropped.
3. `NULL` values sort according to the `ORDER BY` clause.

**Constraints and Limitations**

- PostgreSQL-specific (not available in MySQL or SQL Server).
- Not eligible for partial mode.

---

### Annotated Code Examples

**Example 1: Hypothetical Rank**

```sql
SELECT
  RANK(5000) WITHIN GROUP (ORDER BY salary) AS hypothetical_rank
FROM employees;
```

**Expected Output (illustrative)**

| hypothetical_rank |
|-------------------|
| 12                |

**Why This Output Occurs**: If a row with `salary = 5000` were added to the sorted salaries, it would rank 12th .

---

**Example 2: Hypothetical Percentile Rank**

```sql
SELECT
  PERCENT_RANK(75000) WITHIN GROUP (ORDER BY salary) AS pct_rank
FROM employees;
```

**Expected Output (illustrative)**

| pct_rank |
|----------|
| 0.85     |

**Why This Output Occurs**: The hypothetical salary of 75000 would fall at the 85th percentile of the existing salary distribution .

---

### Real-World Cases

- **Compensation benchmarking**: Where a proposed salary would rank.
- **Admissions**: How a hypothetical test score would rank among applicants.
- **Performance management**: Where a hypothetical sales figure would place a salesperson.

---

## 6. User-Defined Aggregates

### Definitions

**Core Definition**
User-defined aggregates are custom aggregate functions created by the user to perform domain-specific summarisation that is not covered by built-in aggregates.

**Technical Definition**
In PostgreSQL, aggregate functions are defined in terms of a **state value**, an **initial state**, a **state transition function** (`sfunc`), and an optional **final function**. `CREATE AGGREGATE name (input_type) ( sfunc = ..., stype = ..., initcond = ..., finalfunc = ... )` registers the aggregate. The state transition function takes the previous state and the current input row, returning a new state; the final function converts the ending state into the aggregate result. MySQL and SQL Server also support user-defined aggregates, but through different mechanisms (MySQL: `CREATE AGGREGATE FUNCTION` in C/C++; SQL Server: CLR aggregates).

**Beginner-Friendly Explanation**
If the built-in aggregates don't do exactly what you need—like computing a geometric mean, a custom string concatenation, or a domain-specific statistic—you can write your own aggregate function and use it just like `SUM` or `AVG`.

---

### Purposes

- To implement domain-specific aggregations not provided by the RDBMS.
- To encapsulate complex aggregation logic in a reusable database object.
- To optimise repeated custom aggregation patterns.
- To extend the database's analytical capabilities.

---

### Syntax Rules and Structure (PostgreSQL)

```
CREATE AGGREGATE aggregate_name ( input_type ) (
  sfunc = state_transition_function,
  stype = state_data_type,
  initcond = initial_state,          -- optional
  finalfunc = final_function         -- optional
);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `sfunc` | Function that updates the state for each input row. |
| `stype` | Data type of the internal state. |
| `initcond` | Initial value of the state (default `NULL`). |
| `finalfunc` | Function that converts the final state to the result. |

**Syntax Rules**

1. `sfunc` takes the previous state and the input value(s) and returns the new state.
2. If `initcond` is omitted, the initial state is `NULL`; a strict `sfunc` will not be called for `NULL` inputs.
3. The final function is optional; without it, the ending state is the result.
4. `CREATE OR REPLACE AGGREGATE` can redefine an existing aggregate.

**Constraints and Limitations**

- User-defined aggregates are **PostgreSQL-specific** (MySQL and SQL Server use different APIs).
- Performance depends on the efficiency of the transition function.
- Not all aggregate features (e.g., `DISTINCT`, `ORDER BY`) are automatically supported by custom aggregates.

---

### Annotated Code Examples

**Example 1: Custom `SUM` for Complex Numbers (PostgreSQL)**

```sql
-- State transition function: add two complex numbers
CREATE FUNCTION complex_add(complex, complex) RETURNS complex
AS 'SELECT $1 + $2' LANGUAGE SQL IMMUTABLE;

-- Define the aggregate
CREATE AGGREGATE sum (complex) (
  sfunc = complex_add,
  stype = complex,
  initcond = '(0,0)'
);

-- Use it
SELECT SUM(a) FROM test_complex;
```

**Expected Output (illustrative)**

| sum       |
|-----------|
| (34,53.9) |

**Why This Output Occurs**: The `complex_add` transition function accumulates the running sum; the initial state is `(0,0)`; the final state is the sum of all complex values .

---

### Real-World Cases

- **Financial modelling**: Custom weighted-average aggregates.
- **Geospatial analysis**: Aggregating bounding boxes or centroids.
- **Text processing**: Custom string concatenation with deduplication.

---

## 7. Aggregate `FILTER` Clause

### Definitions

**Core Definition**
The `FILTER` clause restricts an aggregate function to only those input rows for which a specified `WHERE` condition evaluates to true.

**Technical Definition**
`aggregate_function(...) FILTER (WHERE filter_clause)` evaluates the aggregate only over rows where `filter_clause` is true; rows where the condition is false or `NULL` are discarded. The `FILTER` clause is part of the SQL standard and is supported by PostgreSQL (9.4+) and SQL Server (2022+). It provides a cleaner, more efficient alternative to `CASE`-based conditional aggregation.

**Beginner-Friendly Explanation**
The `FILTER` clause lets you say “compute this aggregate only for rows that meet this condition” without wrapping everything in a `CASE` expression.

---

### Purposes

- To compute conditional aggregates concisely.
- To avoid repetitive `CASE` expressions in the `SELECT` list.
- To improve readability of pivot-style queries.
- To compute multiple filtered aggregates over the same group in one pass.

---

### Syntax Rules and Structure

**General Syntax**
```
aggregate_function ( expression ) FILTER ( WHERE filter_clause )
```

**Syntax Rules**

1. `FILTER` follows the aggregate call (after any internal `ORDER BY`).
2. Only rows where `filter_clause` evaluates to `true` are fed to the aggregate; `false` and `NULL` rows are discarded .
3. `FILTER` is supported for ordinary aggregates and aggregate window functions.
4. Filtered and unfiltered aggregates can appear side by side over the same groups.

**Constraints and Limitations**

- SQL Server 2022+ supports `FILTER`; earlier versions require `CASE`-based conditional aggregation.
- MySQL does not support the `FILTER` clause; use `CASE` expressions instead.

---

### Annotated Code Examples

**Example 1: Filtered Aggregates (PostgreSQL)**

```sql
SELECT
  customer_id,
  SUM(amount) FILTER (WHERE status = 'completed') AS completed_total,
  SUM(amount) FILTER (WHERE status = 'pending')   AS pending_total,
  COUNT(*)    FILTER (WHERE status = 'cancelled') AS cancelled_count
FROM orders
GROUP BY customer_id;
```

**Expected Output (illustrative)**

| customer_id | completed_total | pending_total | cancelled_count |
|-------------|-----------------|---------------|-----------------|
| 101         | 500             | 200           | 1               |
| 102         | 0               | 750           | 0               |

**Why This Output Occurs**: Each `FILTER` clause independently selects the rows relevant to that aggregate, all within the same group .

---

### Real-World Cases

- **Sales reporting**: Completed vs. pending vs. cancelled revenue.
- **Support analytics**: Count of resolved vs. open tickets.
- **Marketing**: Clicks vs. conversions per campaign.

---

## 8. Window Aggregates

### Definitions

**Core Definition**
Window aggregates compute an aggregate value across a set of rows related to the current row (the “window”) without collapsing the individual rows.

**Technical Definition**
A window aggregate is an ordinary aggregate function followed by an `OVER` clause: `aggregate_function(...) OVER ( [PARTITION BY ...] [ORDER BY ...] [frame_clause] )`. The `OVER` clause defines the window—a set of rows related to the current row—and the aggregate is computed for each row over its window. Unlike `GROUP BY`, window aggregates preserve the original rows and add the aggregate value as an additional column. PostgreSQL documents that any built-in or user-defined ordinary aggregate can be used as a window function when followed by an `OVER` clause .

**Beginner-Friendly Explanation**
Window aggregates let you add summary values to each row—like showing each employee's salary alongside the department average, without losing the individual employee rows.

---

### Purposes

- To compute running totals, moving averages, and cumulative sums.
- To compare each row to its group's aggregate (e.g., difference from group average).
- To rank rows within partitions.
- To avoid self-joins for per-row aggregate comparisons.

---

### Syntax Rules and Structure

**General Syntax**
```
aggregate_function ( expression ) OVER (
  [ PARTITION BY partition_expression [, ...] ]
  [ ORDER BY sort_expression [, ...] ]
  [ frame_clause ]
)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `PARTITION BY` | Divides rows into partitions; the aggregate is computed within each partition. |
| `ORDER BY` | Orders rows within each partition (required for running totals and ranking). |
| `frame_clause` | Defines the subset of the partition used for the aggregate (e.g., `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`). |

**Syntax Rules**

1. `OVER` is required; without it, the aggregate is a plain group aggregate.
2. `PARTITION BY` is optional; omitting it treats the entire result set as one partition.
3. `ORDER BY` is optional for simple aggregates but required for running totals and ranking.
4. Window aggregates do not reduce the number of rows; each input row produces one output row.

**Constraints and Limitations**

- Window functions are evaluated after `WHERE`, `GROUP BY`, and `HAVING`, but before `ORDER BY` and `LIMIT`.
- `DISTINCT` is not supported in window aggregate calls.
- Ordered-set and hypothetical-set aggregates are **not** supported as window functions in PostgreSQL.

---

### Annotated Code Examples

**Example 1: Department Average Alongside Each Row**

```sql
SELECT
  employee_id,
  department_id,
  salary,
  AVG(salary) OVER (PARTITION BY department_id) AS dept_avg_salary,
  salary - AVG(salary) OVER (PARTITION BY department_id) AS diff_from_avg
FROM employees;
```

**Expected Output (illustrative)**

| employee_id | department_id | salary | dept_avg_salary | diff_from_avg |
|-------------|---------------|--------|-----------------|---------------|
| 1           | 10            | 5000   | 5500            | -500          |
| 2           | 10            | 6000   | 5500            | 500           |
| 3           | 20            | 7000   | 7000            | 0             |

**Why This Output Occurs**: `AVG(salary) OVER (PARTITION BY department_id)` computes the department average for each row without collapsing the rows .

---

**Example 2: Running Total**

```sql
SELECT
  order_date,
  amount,
  SUM(amount) OVER (ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM orders;
```

**Expected Output (illustrative)**

| order_date | amount | running_total |
|------------|--------|---------------|
| 2026-01-01 | 100    | 100           |
| 2026-01-02 | 200    | 300           |
| 2026-01-03 | 150    | 450           |

**Why This Output Occurs**: The frame clause defines the window as all rows from the start up to the current row, producing a cumulative sum.

---

### Real-World Cases

- **Finance**: Running balance of an account.
- **Sales**: Moving average of daily sales over 7 days.
- **HR**: Each employee's salary compared to the department average.

---

## 9. Aggregate over Partitions

### Definitions

**Core Definition**
Aggregating over partitions means computing window aggregates within independently defined partitions of the result set.

**Technical Definition**
The `PARTITION BY` clause of the `OVER` specification divides the rows into independent groups (partitions) before the window aggregate is computed. Each partition is processed separately; the aggregate resets at each partition boundary. Partitioning can leverage table partitioning for performance if the `PARTITION BY` columns align with the table's physical partitioning key. Smaller partitions are more likely to fit in memory, reducing disk spills and improving performance.

**Beginner-Friendly Explanation**
Partitioning is like dividing a large spreadsheet into separate sheets by category, computing totals on each sheet independently, and then putting all the rows back together.

---

### Purposes

- To compute per-group aggregates while preserving all rows.
- To reset running totals or ranks at group boundaries.
- To improve performance by processing smaller, independent partitions.
- To align window computations with physical table partitions.

---

### Syntax Rules and Structure

```
aggregate_function ( expression ) OVER ( PARTITION BY column [, ...] [ORDER BY ...] [frame_clause] )
```

**Syntax Rules**

1. `PARTITION BY` can reference one or more columns or expressions.
2. The aggregate is computed independently within each partition.
3. `ORDER BY` within `OVER` orders rows **within each partition**, not across the entire result set.
4. To leverage physical table partitioning, align `PARTITION BY` with the table's partition key.

**Constraints and Limitations**

- A poorly chosen partition key can create very large partitions that spill to disk.
- PostgreSQL window functions do not automatically take advantage of existing table partitions; the optimiser must be able to prove that the partitioning is beneficial.

---

### Annotated Code Examples

**Example 1: Running Total per Customer**

```sql
SELECT
  customer_id,
  order_date,
  amount,
  SUM(amount) OVER (
    PARTITION BY customer_id
    ORDER BY order_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS customer_running_total
FROM orders;
```

**Expected Output (illustrative)**

| customer_id | order_date | amount | customer_running_total |
|-------------|------------|--------|------------------------|
| 101         | 2026-01-01 | 100    | 100                    |
| 101         | 2026-01-05 | 200    | 300                    |
| 102         | 2026-01-02 | 150    | 150                    |

**Why This Output Occurs**: The running total resets for each customer because `PARTITION BY customer_id` creates independent partitions.

---

### Real-World Cases

- **Banking**: Running balance per account.
- **E-commerce**: Cumulative spend per customer.
- **Operations**: Rolling average per machine per shift.

---

## 10. Advanced Reporting Queries

### Definitions

**Core Definition**
Advanced reporting queries combine grouping extensions, conditional aggregation, window functions, and ordered-set aggregates to produce multi-dimensional, analytical reports directly in SQL.

**Technical Definition**
Advanced reporting queries leverage `GROUPING SETS`, `ROLLUP`, and `CUBE` to compute subtotals and grand totals; `PIVOT`/`UNPIVOT` (or conditional aggregation) to reshape data; window functions for rankings, running totals, and moving averages; and CTEs for structuring complex logic. These techniques enable single-query solutions for business intelligence requirements that would otherwise require multiple queries, temporary tables, or application-side processing.

**Beginner-Friendly Explanation**
Advanced reporting queries are the “power tools” of SQL reporting—they let you build dashboards, cross-tabulations, and analytical summaries in a single query instead of combining many separate queries.

---

### Purposes

- To produce multi-level subtotals and grand totals in one result set.
- To pivot categorical data into columns for cross-tab reports.
- To compute rankings, running totals, and moving averages alongside detail rows.
- To replace multiple queries and application-side aggregation with a single, optimised SQL statement.

---

### Syntax Rules and Structure (Key Constructs)

1. **Grouping extensions**:
   ```
   GROUP BY GROUPING SETS ( (col1, col2), (col1), () )
   GROUP BY ROLLUP (col1, col2)
   GROUP BY CUBE (col1, col2)
   ```

2. **Conditional aggregation for pivoting**:
   ```
   SUM(CASE WHEN category = 'A' THEN amount ELSE 0 END) AS category_a
   ```

3. **Window functions for analytics**:
   ```
   SUM(amount) OVER (PARTITION BY region ORDER BY date) AS running_total
   RANK() OVER (PARTITION BY region ORDER BY amount DESC) AS rank_in_region
   ```

4. **CTEs for structuring**:
   ```
   WITH monthly AS (
     SELECT DATE_TRUNC('month', order_date) AS month, SUM(amount) AS total
     FROM orders GROUP BY month
   )
   SELECT * FROM monthly;
   ```

**Constraints and Limitations**

- `CUBE` with many columns produces an exponential number of grouping sets (2^n).
- Pivoting with conditional aggregation requires knowing the distinct categories in advance.
- Window functions are evaluated after `GROUP BY`, so they cannot directly reference grouped aliases in the same `SELECT` level (use a CTE or subquery).

---

### Annotated Code Examples

**Example 1: Multi-Level Report with ROLLUP and Window Functions**

```sql
WITH regional_sales AS (
  SELECT
    region,
    product_category,
    SUM(amount) AS total_sales
  FROM orders
  GROUP BY ROLLUP (region, product_category)
)
SELECT
  region,
  product_category,
  total_sales,
  SUM(total_sales) OVER (PARTITION BY region) AS region_subtotal,
  RANK() OVER (PARTITION BY region ORDER BY total_sales DESC) AS rank_in_region
FROM regional_sales
ORDER BY region, product_category;
```

**Explanation**: `ROLLUP` produces subtotals by region and grand total; window functions add region-level subtotals and rankings without additional queries.

---

**Example 2: Pivot Report with Conditional Aggregation**

```sql
SELECT
  DATE_TRUNC('month', order_date) AS month,
  SUM(CASE WHEN category = 'Electronics' THEN amount ELSE 0 END) AS electronics,
  SUM(CASE WHEN category = 'Clothing'    THEN amount ELSE 0 END) AS clothing,
  SUM(CASE WHEN category = 'Furniture'   THEN amount ELSE 0 END) AS furniture
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;
```

**Expected Output (illustrative)**

| month       | electronics | clothing | furniture |
|-------------|-------------|----------|-----------|
| 2026-01-01  | 15000       | 8000     | 12000     |
| 2026-02-01  | 18000       | 9500     | 11000     |

**Why This Output Occurs**: The `CASE` expressions route amounts into category-specific columns; the `GROUP BY` on the month expression produces one row per month .

---

### Real-World Cases

- **Executive dashboards**: Revenue by region, product, and quarter with subtotals.
- **Financial statements**: Monthly P&L with year-to-date running totals.
- **Sales analysis**: Rankings of salespeople within regions alongside running totals.

---

## References

1. **PostgreSQL 18 Documentation — Aggregate Functions** — https://www.postgresql.org/docs/current/functions-aggregate.html
2. **PostgreSQL 18 Documentation — Aggregate Expressions (FILTER, DISTINCT, ORDER BY)** — https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-AGGREGATES 
3. **PostgreSQL 18 Documentation — User-Defined Aggregates** — https://www.postgresql.org/docs/current/xaggr.html 
4. **PostgreSQL 18 Documentation — Window Functions** — https://www.postgresql.org/docs/current/functions-window.html 
5. **PostgreSQL Feature Matrix — FILTER Clause for Aggregate Functions** — https://www.postgresql.org/about/featurematrix/detail/filter-clause-for-aggregate-functions/ 
6. **PostgreSQL 18 Documentation — Ordered-Set and Hypothetical-Set Aggregates (Tables 9.64–9.66)** — https://www.postgresql.org/docs/current/functions-aggregate.html#FUNCTIONS-AGGREGATE-ORDEREDSET-TABLE 
7. **Microsoft SQL Server — Aggregate Functions (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/functions/aggregate-functions-transact-sql
8. **Microsoft SQL Server — OVER Clause (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql 
9. **MySQL 8.0 Reference Manual — Window Functions** — https://dev.mysql.com/doc/refman/8.0/en/window-functions.html
10. **MySQL 8.0 Reference Manual — Aggregate Function Descriptions** — https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html
11. **Apache AsterixDB — COUNT(DISTINCT) Hash Rewrite Performance Notes** — https://issues.apache.org/jira/browse/ASTERIXDB-3859 
12. **Microsoft Learn — Distinct Aggregation Considered Harmful** — https://learn.microsoft.com/en-us/archive/blogs/conor_cunningham_msft/distinct-aggregation-considered-harmful 
13. **SQL:2016 Standard — ISO/IEC 9075-2:2016 (Foundation)** — https://www.iso.org/standard/63555.html

---

This cheat sheet provides a comprehensive reference for advanced SQL aggregation across PostgreSQL, MySQL, and SQL Server, covering conditional, multiple, distinct, ordered-set, hypothetical-set, user-defined, filtered, and window-based aggregation, as well as advanced reporting patterns. Always consult your specific RDBMS documentation for version-specific feature availability and optimiser behaviour.