# SQL Aggregate Functions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
SQL aggregate functions are built-in routines that compute a single result from a set of input rows, summarizing or transforming data across multiple rows into a scalar value, an array, a string, or a JSON document.

**Technical Definition**
In SQL, aggregate functions operate on a set of values (typically a column across multiple rows) and return a single value per group. They are defined in part by SQL:2003 and SQL:2016, but each RDBMS—PostgreSQL, MySQL, and SQL Server—implements a distinct set with vendor-specific names, return types, and NULL-handling semantics. Aggregate functions are used with `GROUP BY` to produce one result per group, or without `GROUP BY` to aggregate over all rows. Ordered-set aggregates (`PERCENTILE_CONT`, `PERCENTILE_DISC`, `MODE`) use a special `WITHIN GROUP (ORDER BY ...)` syntax that reflects their dependence on sort order. Aggregate functions support partial mode for parallel aggregation in PostgreSQL, and their performance is heavily influenced by index design, NULL distribution, and the presence of `DISTINCT` or `FILTER` clauses .

**Beginner-Friendly Explanation**
Aggregate functions let you summarize lots of data into a single answer. Instead of looking at every row one by one, you can ask questions like “how many rows are there?”, “what is the total?”, “what is the average?”, or “what is the most common value?”—and get one result. You can do this for the whole table or for each group (e.g., per country, per category).

---

### Key Characteristics

1. **NULL Ignoring by Default**: Nearly all aggregate functions ignore `NULL` values, with the major exception of `COUNT(*)`, which counts rows regardless of `NULL`s. `SUM`, `AVG`, `MIN`, and `MAX` skip `NULL`s; `ARRAY_AGG` and `JSON_AGG` include `NULL`s by default in PostgreSQL but have strict variants that skip them .
2. **Empty-Set Behaviour**: Except for `COUNT`, aggregate functions return `NULL` when no rows are selected. `SUM` of no rows is `NULL`, not `0`; `ARRAY_AGG` of no rows is `NULL`, not an empty array. Use `COALESCE` to substitute defaults .
3. **Order Sensitivity**: String, array, and JSON aggregates produce different results depending on input order. Use an internal `ORDER BY` clause to control ordering deterministically .
4. **Vendor-Specific Naming**: `STRING_AGG` (PostgreSQL, SQL Server) vs. `GROUP_CONCAT` (MySQL); `ARRAY_AGG` and `JSON_AGG` are PostgreSQL-specific; SQL Server uses `STRING_AGG` and lacks native array aggregation .
5. **Statistical Function Divergence**: PostgreSQL provides the richest set of statistical aggregates (`STDDEV_SAMP`, `STDDEV_POP`, `VAR_SAMP`, `VAR_POP`, correlation, regression). MySQL provides `STDDEV`, `STDDEV_POP`, `STDDEV_SAMP`, `VARIANCE`, `VAR_POP`, `VAR_SAMP`. SQL Server provides `STDEV`, `STDEVP`, `VAR`, `VARP`. `PERCENTILE_CONT` and `PERCENTILE_DISC` are available in PostgreSQL and SQL Server (as window functions in SQL Server), but not in MySQL .
6. **Ordered-Set Aggregates**: `PERCENTILE_CONT`, `PERCENTILE_DISC`, and `MODE` require the `WITHIN GROUP (ORDER BY ...)` syntax, which separates them from ordinary aggregates .
7. **Performance Sensitivity**: `COUNT(*)` on large tables requires a full scan or full index scan in PostgreSQL. `GROUP BY` can use index-based Loose Index Scan in MySQL when conditions are met. `DISTINCT` aggregates can disable parallel aggregation. `FILTER` clauses may interact with index usage .

---

### Prerequisites

- **Basic SQL querying**: `SELECT`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`.
- **NULL semantics**: Three-valued logic and how `NULL` propagates through aggregates.
- **Data types**: Numeric, string, boolean, array, JSON, and temporal types.
- **Index fundamentals**: B-tree indexes and how they support grouping and aggregation.
- **Window function concepts**: For understanding the relationship between aggregates and window functions.

---

### Related Programming Areas

| Area | Connection |
|------|------------|
| **Reporting & Dashboards** | Aggregates produce summary statistics for KPIs and business metrics. |
| **Data Warehousing** | `GROUP BY` with aggregates is the foundation of OLAP and star-schema queries. |
| **Statistics & Data Science** | `STDDEV`, `VARIANCE`, and percentiles support statistical analysis directly in SQL. |
| **String/JSON APIs** | `STRING_AGG`, `ARRAY_AGG`, and `JSON_AGG` denormalise one-to-many relationships for APIs. |
| **Query Optimisation** | Index design, `FILTER`, and parallel aggregation are critical for performance. |
| **Data Quality** | `BOOL_AND`/`BOOL_OR` and `COUNT` variants detect data integrity issues. |

---

### Core Concepts / Features

1. **Basic Aggregations** — `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`
2. **Collection & String Aggregations** — `STRING_AGG`, `ARRAY_AGG`, `JSON_AGG`
3. **Boolean Aggregations** — `BOOL_AND`, `BOOL_OR`
4. **Statistical & Distribution Functions** — `STDDEV`, `VARIANCE`, `PERCENTILE_CONT`, `PERCENTILE_DISC`, `MODE`
5. **Aggregate Behavior & Optimization** — Aggregate NULL Handling, Aggregate Performance

---

## 1. Basic Aggregations

### 1.1 `COUNT`

#### Definitions

**Core Definition**
`COUNT` returns the number of rows or non-`NULL` values in a set.

**Technical Definition**
`COUNT(*)` returns the total number of input rows, including rows where all columns are `NULL`. `COUNT(expression)` returns the number of rows for which `expression` is not `NULL`. `COUNT(DISTINCT expression)` returns the number of distinct non-`NULL` values. In PostgreSQL, `COUNT` returns `bigint`; in MySQL, it returns `BIGINT`; in SQL Server, `COUNT` returns `int` and `COUNT_BIG` returns `bigint` .

**Beginner-Friendly Explanation**
`COUNT` answers “how many?” `COUNT(*)` counts every row; `COUNT(column)` counts only rows where that column has a value.

#### Purposes

- To count total rows in a table or group.
- To count non-`NULL` values in a column.
- To count distinct values (e.g., unique visitors).
- To validate data completeness.

#### Syntax Rules and Structure

**General Syntax**
```
COUNT ( * )
COUNT ( expression )
COUNT ( DISTINCT expression )
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `*` | Count all rows. |
| `expression` | Count non-`NULL` values of the expression. |
| `DISTINCT` | Count distinct non-`NULL` values. |

**Syntax Rules**

1. `COUNT(*)` never returns `NULL`; it returns `0` for an empty set.
2. `COUNT(expression)` ignores `NULL` values.
3. `COUNT(DISTINCT ...)` eliminates duplicates before counting.

**Constraints and Limitations**

- `COUNT(*)` on large tables requires a full scan or full index scan in PostgreSQL .
- `COUNT(DISTINCT ...)` can disable parallel aggregation in PostgreSQL .
- SQL Server's `COUNT` returns `int`; use `COUNT_BIG` to avoid overflow on large tables .

#### Annotated Code Examples

**Example 1: `COUNT(*)` vs. `COUNT(column)`**

```sql
-- Setup: table weather(city, temp_lo)
SELECT
  COUNT(*) AS total_rows,
  COUNT(temp_lo) AS non_null_temps
FROM weather;
```

**Expected Output (illustrative)**

| total_rows | non_null_temps |
|------------|----------------|
| 10         | 8              |

**Why This Output Occurs**: `COUNT(*)` counts all 10 rows; `COUNT(temp_lo)` counts only the 8 rows where `temp_lo` is not `NULL` .

**Example 2: `COUNT(DISTINCT ...)`**

```sql
SELECT COUNT(DISTINCT city) AS distinct_cities
FROM weather;
```

**Expected Output**

| distinct_cities |
|-----------------|
| 3               |

#### Real-World Cases

- **User metrics**: `COUNT(DISTINCT user_id)` for unique visitors.
- **Data completeness**: Comparing `COUNT(*)` to `COUNT(column)` to measure null rate.
- **Pagination**: Counting total rows for page calculation.

---

### 1.2 `SUM`

#### Definitions

**Core Definition**
`SUM` returns the total of all non-`NULL` numeric values in a set.

**Technical Definition**
`SUM(expression)` computes the sum of all non-`NULL` input values. In PostgreSQL, the return type is `bigint` for `smallint`/`integer` inputs, `numeric` for `bigint`/`numeric`, and the same floating-point type for floating-point inputs . In MySQL, `SUM` returns `DECIMAL` for exact-value arguments and `DOUBLE` for approximate-value arguments . In SQL Server, `SUM` returns the same type as the input for exact numeric types, with potential overflow if the sum exceeds the type's range.

**Beginner-Friendly Explanation**
`SUM` adds up all the numbers in a column, ignoring missing values.

#### Purposes

- To compute total revenue, quantity, or other additive metrics.
- To calculate running totals when used with window functions.
- To validate that component values sum to a known total.
- To aggregate numeric data for reporting.

#### Syntax Rules and Structure

**General Syntax**
```
SUM ( expression )
SUM ( DISTINCT expression )
```

**Syntax Rules**

1. `SUM` ignores `NULL` values.
2. `SUM` of an empty set returns `NULL`, not `0` .
3. `SUM(DISTINCT ...)` adds each distinct value only once.

**Constraints and Limitations**

- `SUM` of `integer` columns can overflow `integer` range in SQL Server; consider `CAST` to `bigint` or `numeric` .
- MySQL's `SUM` does not work with temporal values directly; convert to numeric units first .

#### Annotated Code Examples

**Example 1: Basic `SUM`**

```sql
SELECT SUM(amount) AS total_revenue
FROM orders;
```

**Expected Output (illustrative)**

| total_revenue |
|---------------|
| 125430.50     |

**Example 2: `SUM` with `DISTINCT`**

```sql
SELECT SUM(DISTINCT amount) AS sum_distinct
FROM orders;
```

**Explanation**: Duplicate amounts are counted only once.

#### Real-World Cases

- **Financial totals**: Summing invoice amounts or transaction values.
- **Inventory**: Summing stock quantities across warehouses.
- **Time tracking**: Summing hours worked per project.

---

### 1.3 `AVG`

#### Definitions

**Core Definition**
`AVG` returns the arithmetic mean of all non-`NULL` numeric values in a set.

**Technical Definition**
`AVG(expression)` computes the sum of non-`NULL` values divided by the count of non-`NULL` values. In PostgreSQL, `AVG(integer)` returns `numeric`, `AVG(bigint)` returns `numeric`, and `AVG(interval)` returns `interval` . In MySQL, `AVG` returns `DECIMAL` for exact-value arguments and `DOUBLE` for approximate-value arguments .

**Beginner-Friendly Explanation**
`AVG` calculates the average—add up all the values and divide by how many there are, ignoring missing values.

#### Purposes

- To compute average order value, average rating, or average temperature.
- To normalise totals by count.
- To detect outliers by comparing individual values to the average.
- To summarise continuous data.

#### Syntax Rules and Structure

**General Syntax**
```
AVG ( expression )
AVG ( DISTINCT expression )
```

**Syntax Rules**

1. `AVG` ignores `NULL` values; the denominator is the count of non-`NULL` values, not the total row count.
2. `AVG` of an empty set returns `NULL` .
3. `AVG(DISTINCT ...)` averages only distinct values.

**Constraints and Limitations**

- `AVG(integer)` may perform integer division in some systems; cast to `NUMERIC` to avoid truncation.
- MySQL's `AVG` does not work with temporal values directly.

#### Annotated Code Examples

**Example 1: `AVG` of a Column**

```sql
SELECT AVG(temp_lo) AS avg_low_temp
FROM weather;
```

**Expected Output (illustrative)**

| avg_low_temp |
|--------------|
| 42.5         |

**Example 2: `AVG` with Integer Division Pitfall**

```sql
-- Risk of integer division in some systems
SELECT AVG(rating) AS avg_rating
FROM reviews;
-- Safer:
SELECT AVG(rating::NUMERIC) AS avg_rating
FROM reviews;
```

#### Real-World Cases

- **E-commerce**: Average order value (AOV).
- **Education**: Average test scores.
- **Operations**: Average response time.

---

### 1.4 `MIN`

#### Definitions

**Core Definition**
`MIN` returns the smallest non-`NULL` value in a set.

**Technical Definition**
`MIN(expression)` returns the minimum value of the non-`NULL` input values. It is available for any sortable data type: numeric, string, date/time, enum, and certain other types in PostgreSQL . In SQL Server, `MIN` returns the minimum numeric, earliest date/time, or first string according to collation .

**Beginner-Friendly Explanation**
`MIN` finds the smallest value—the lowest number, the earliest date, or the alphabetically first string.

#### Purposes

- To find the earliest date or lowest numeric value.
- To identify the smallest element in a group.
- To support range queries and outlier detection.
- To determine the lower bound of a data distribution.

#### Syntax Rules and Structure

**General Syntax**
```
MIN ( expression )
```

**Syntax Rules**

1. `MIN` ignores `NULL` values.
2. `MIN` of an empty set returns `NULL`.
3. Return type is the same as the input type.

**Constraints and Limitations**

- String comparison depends on collation.
- `MIN`/`MAX` can use index optimisations in MySQL under specific conditions (Loose Index Scan) .

#### Annotated Code Examples

**Example 1: `MIN` of a Numeric Column**

```sql
SELECT MIN(temp_lo) AS lowest_temp
FROM weather;
```

**Expected Output**

| lowest_temp |
|-------------|
| 35          |

**Example 2: `MIN` of a Date Column**

```sql
SELECT MIN(order_date) AS first_order_date
FROM orders;
```

#### Real-World Cases

- **HR**: Earliest hire date in a department.
- **Sales**: Lowest sale amount in a period.
- **Inventory**: Oldest stock item.

---

### 1.5 `MAX`

#### Definitions

**Core Definition**
`MAX` returns the largest non-`NULL` value in a set.

**Technical Definition**
`MAX(expression)` returns the maximum value of the non-`NULL` input values, available for any sortable type. In SQL Server, `MAX` returns the maximum numeric, latest date/time, or last string according to collation .

**Beginner-Friendly Explanation**
`MAX` finds the largest value—the highest number, the latest date, or the alphabetically last string.

#### Purposes

- To find the latest date or highest numeric value.
- To identify the largest element in a group.
- To support ranking and top-N queries.
- To determine the upper bound of a data distribution.

#### Syntax Rules and Structure

**General Syntax**
```
MAX ( expression )
```

**Syntax Rules**

1. `MAX` ignores `NULL` values.
2. `MAX` of an empty set returns `NULL`.
3. Return type is the same as the input type.

**Constraints and Limitations**

- Same collation and index considerations as `MIN`.

#### Annotated Code Examples

**Example 1: `MAX` of a Numeric Column**

```sql
SELECT MAX(temp_lo) AS highest_temp
FROM weather;
```

**Expected Output**

| highest_temp |
|--------------|
| 46           |

**Example 2: `MAX` with `GROUP BY`**

```sql
SELECT city, MAX(temp_lo) AS max_temp
FROM weather
GROUP BY city;
```

**Expected Output (illustrative)**

| city          | max_temp |
|---------------|----------|
| Hayward       | 37       |
| San Francisco | 46       |

**Why This Output Occurs**: `MAX` is computed per city group .

#### Real-World Cases

- **Finance**: Highest transaction amount in a day.
- **Support**: Latest ticket update time.
- **IoT**: Peak sensor reading.

---

## 2. Collection & String Aggregations

### 2.1 `STRING_AGG` / `GROUP_CONCAT`

#### Definitions

**Core Definition**
`STRING_AGG` (PostgreSQL, SQL Server) or `GROUP_CONCAT` (MySQL) concatenates non-`NULL` string values from multiple rows into a single string, separated by a delimiter.

**Technical Definition**
`STRING_AGG(expression, delimiter)` in PostgreSQL and SQL Server concatenates the non-`NULL` input values into a `text` or `nvarchar` string, with `delimiter` inserted between values. MySQL's equivalent is `GROUP_CONCAT([DISTINCT] expression [ORDER BY ...] [SEPARATOR delimiter])`. Ordering is unspecified by default and must be controlled with an internal `ORDER BY` clause to produce deterministic results .

**Beginner-Friendly Explanation**
`STRING_AGG`/`GROUP_CONCAT` turns many rows into one comma-separated (or custom-separated) string—useful for displaying lists like “Alice, Bob, Carol” on a single line.

#### Purposes

- To produce comma-separated lists for reports and APIs.
- To denormalise one-to-many relationships into a single field.
- To aggregate tags, categories, or labels.
- To generate CSV-like output from query results.

#### Syntax Rules and Structure

**PostgreSQL**
```
STRING_AGG ( expression, delimiter [ ORDER BY ... ] )
```
**SQL Server**
```
STRING_AGG ( expression, separator ) [ WITHIN GROUP ( ORDER BY ... ) ]
```
**MySQL**
```
GROUP_CONCAT ( [DISTINCT] expression [ ORDER BY ... ] [ SEPARATOR delimiter ] )
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `expression` | Column or expression to concatenate. |
| `delimiter` / `separator` | String inserted between values. |
| `ORDER BY` | Optional ordering of values before concatenation. |
| `DISTINCT` (MySQL) | Eliminate duplicate values. |
| `SEPARATOR` (MySQL) | Custom delimiter (default `,`). |

**Syntax Rules**

1. `STRING_AGG` in PostgreSQL requires two arguments (expression and delimiter); delimiter cannot be omitted .
2. SQL Server's `STRING_AGG` uses `WITHIN GROUP (ORDER BY ...)` for ordering .
3. MySQL's `GROUP_CONCAT` allows `DISTINCT`, `ORDER BY`, and `SEPARATOR` .
4. `NULL` values are skipped; the delimiter is not inserted for skipped `NULL`s.

**Constraints and Limitations**

- Ordering is not guaranteed without explicit `ORDER BY` .
- Result size is limited by `max_allowed_packet` in MySQL and similar limits in other systems.
- Parallel aggregation support varies: PostgreSQL supports `STRING_AGG` in partial mode .

#### Annotated Code Examples

**Example 1: PostgreSQL `STRING_AGG` with `ORDER BY`**

```sql
SELECT
  city,
  STRING_AGG(name, ', ' ORDER BY name) AS residents
FROM residents
GROUP BY city;
```

**Expected Output (illustrative)**

| city          | residents        |
|---------------|------------------|
| San Francisco | Alice, Bob, Carol |

**Why This Output Occurs**: Names are concatenated per city, ordered alphabetically, with `', '` between values .

**Example 2: MySQL `GROUP_CONCAT` with `DISTINCT` and `SEPARATOR`**

```sql
SELECT
  category,
  GROUP_CONCAT(DISTINCT product ORDER BY product SEPARATOR ' | ') AS products
FROM catalog
GROUP BY category;
```

**Expected Output (illustrative)**

| category | products         |
|----------|------------------|
| Books    | SQL Guide \| Python 101 |

#### Real-World Cases

- **E-commerce**: Listing product names in an order.
- **CRM**: Aggregating phone numbers or emails per contact.
- **Content management**: Joining tags per article.

---

### 2.2 `ARRAY_AGG`

#### Definitions

**Core Definition**
`ARRAY_AGG` collects values from multiple rows into a single array.

**Technical Definition**
`ARRAY_AGG(expression [ ORDER BY ... ])` is a PostgreSQL aggregate that collects all input values—including `NULL`s—into an array. When applied to array inputs, it concatenates them into an array of one higher dimension . `ARRAY_AGG` returns `NULL` rather than an empty array when there are no input rows .

**Beginner-Friendly Explanation**
`ARRAY_AGG` turns a column of values into a single array—like turning rows of tags into a list you can pass to an application.

#### Purposes

- To pass a list of values to an application in a single database call.
- To avoid N+1 query problems in APIs.
- To collect related values for further array processing.
- To aggregate values into a structured type.

#### Syntax Rules and Structure

**PostgreSQL**
```
ARRAY_AGG ( expression [ ORDER BY ... ] ) → anyarray
```

**Syntax Rules**

1. `ARRAY_AGG` includes `NULL` values by default; use `FILTER (WHERE expression IS NOT NULL)` to exclude them.
2. Ordering is unspecified without `ORDER BY`.
3. Returns `NULL` for an empty input set .

**Constraints and Limitations**

- **PostgreSQL-specific**; MySQL and SQL Server do not have `ARRAY_AGG`.
- Large arrays may consume significant memory.
- Not all array elements need be of the same type, but the result array must be homogeneous.

#### Annotated Code Examples

**Example 1: Basic `ARRAY_AGG` with Ordering**

```sql
SELECT ARRAY_AGG(v ORDER BY v DESC) AS sorted_values
FROM (VALUES (1),(3),(4),(3),(2)) AS vals(v);
```

**Expected Output**

| sorted_values |
|---------------|
| {4,3,3,2,1}   |

**Why This Output Occurs**: Values are collected into an array in descending order .

**Example 2: `ARRAY_AGG` with `DISTINCT`**

```sql
SELECT ARRAY_AGG(DISTINCT v ORDER BY v DESC) AS distinct_sorted
FROM (VALUES (1),(3),(4),(3),(2)) AS vals(v);
```

**Expected Output**

| distinct_sorted |
|-----------------|
| {4,3,2,1}       |

#### Real-World Cases

- **API responses**: Returning a list of associated IDs in one row.
- **Graph queries**: Collecting neighbouring node IDs.
- **Sessionisation**: Aggregating event sequences into arrays.

---

### 2.3 `JSON_AGG`

#### Definitions

**Core Definition**
`JSON_AGG` collects values from multiple rows into a single JSON array.

**Technical Definition**
`JSON_AGG(expression [ ORDER BY ... ])` is a PostgreSQL aggregate that aggregates values, including `NULL`s, into a JSON array. Values are converted to JSON according to `to_json` or `to_jsonb`. `JSONB_AGG` is the binary JSON variant. Strict variants (`JSON_AGG_STRICT`, `JSONB_AGG_STRICT`) skip `NULL` values . `JSON_OBJECT_AGG` collects key/value pairs into a JSON object .

**Beginner-Friendly Explanation**
`JSON_AGG` turns rows into a JSON array—perfect for building JSON responses directly in SQL without application-side serialisation.

#### Purposes

- To generate JSON payloads directly from SQL for APIs.
- To aggregate structured data into a single document.
- To avoid multiple round trips between application and database.
- To support document-oriented access patterns on relational data.

#### Syntax Rules and Structure

**PostgreSQL**
```
JSON_AGG ( expression [ ORDER BY ... ] ) → json
JSONB_AGG ( expression [ ORDER BY ... ] ) → jsonb
JSON_AGG_STRICT ( expression ) → json
JSON_OBJECT_AGG ( key, value [ ORDER BY ... ] ) → json
```

**Syntax Rules**

1. `JSON_AGG` includes `NULL`s; `JSON_AGG_STRICT` skips them .
2. Ordering is unspecified without `ORDER BY`.
3. `JSON_OBJECT_AGG` requires two arguments (key and value); keys cannot be `NULL` .

**Constraints and Limitations**

- **PostgreSQL-specific** (MySQL has `JSON_ARRAYAGG` and `JSON_OBJECTAGG`).
- `JSON_AGG` is not eligible for partial mode (parallel aggregation) .
- Large JSON documents may exceed memory or packet limits.

#### Annotated Code Examples

**Example 1: Basic `JSON_AGG`**

```sql
SELECT JSON_AGG(t) AS json_array
FROM (SELECT id, name FROM users WHERE active = true) t;
```

**Expected Output (illustrative)**

| json_array                                                                 |
|----------------------------------------------------------------------------|
| [{"id":1,"name":"Alice"},{"id":2,"name":"Bob"}]                           |

**Example 2: `JSON_OBJECT_AGG`**

```sql
SELECT JSON_OBJECT_AGG(name, email) AS user_emails
FROM users;
```

**Expected Output (illustrative)**

| user_emails                                      |
|--------------------------------------------------|
| {"Alice":"alice@example.com","Bob":"bob@example.com"} |

#### Real-World Cases

- **REST APIs**: Building nested JSON responses in a single query.
- **Data export**: Generating JSON files from relational data.
- **Configuration storage**: Aggregating key/value settings into a JSON object.

---

## 3. Boolean Aggregations

### 3.1 `BOOL_AND` / `EVERY`

#### Definitions

**Core Definition**
`BOOL_AND` returns `true` if all non-`NULL` input boolean values are `true`; otherwise it returns `false`.

**Technical Definition**
`BOOL_AND(expression)` is a PostgreSQL boolean aggregate that returns `true` if every non-`NULL` input value is `true`, and `false` otherwise. It is equivalent to the SQL standard `EVERY` aggregate, which PostgreSQL also supports . MySQL and SQL Server do not have a native `BOOL_AND`; equivalent logic requires `MIN(boolean_column)` or `CASE` expressions.

**Beginner-Friendly Explanation**
`BOOL_AND` answers “are all of these true?”—useful for checking that every condition in a group is satisfied.

#### Purposes

- To verify that all rows in a group satisfy a condition.
- To detect data quality issues (e.g., all records must have a flag set).
- To implement universal quantification in SQL.
- To simplify `HAVING` clauses compared to `COUNT`-based logic.

#### Syntax Rules and Structure

**PostgreSQL**
```
BOOL_AND ( boolean_expression ) → boolean
EVERY ( boolean_expression ) → boolean   -- SQL standard synonym
```

**Syntax Rules**

1. `BOOL_AND` ignores `NULL` values; it returns `true` only if all non-`NULL` values are `true`.
2. If all values are `NULL`, `BOOL_AND` returns `NULL` (not `false`).
3. `EVERY` is a standard SQL synonym.

**Constraints and Limitations**

- **PostgreSQL-specific** (though `EVERY` is standard).
- MySQL and SQL Server require workarounds (e.g., `MIN(flag) = 1` if `flag` is 0/1).

#### Annotated Code Examples

**Example 1: Checking That All Flags Are True**

```sql
SELECT BOOL_AND(is_verified) AS all_verified
FROM user_profiles
WHERE user_id = 42;
```

**Expected Output**

| all_verified |
|--------------|
| t            |

**Why This Output Occurs**: All non-`NULL` `is_verified` values are `true` .

#### Real-World Cases

- **Compliance checks**: Ensuring all required documents are uploaded.
- **Test results**: Verifying that all test cases passed.
- **Data integrity**: Confirming all related records have a mandatory flag.

---

### 3.2 `BOOL_OR`

#### Definitions

**Core Definition**
`BOOL_OR` returns `true` if at least one non-`NULL` input boolean value is `true`; otherwise it returns `false`.

**Technical Definition**
`BOOL_OR(expression)` is a PostgreSQL boolean aggregate that returns `true` if any non-`NULL` input value is `true`, and `false` otherwise. It corresponds to the SQL standard `ANY` or `SOME` aggregate, though PostgreSQL does not use those names due to syntactic ambiguity with `ANY` subqueries .

**Beginner-Friendly Explanation**
`BOOL_OR` answers “is at least one of these true?”—useful for checking whether any row in a group satisfies a condition.

#### Purposes

- To detect whether any row in a group meets a condition.
- To implement existential quantification in SQL.
- To flag groups that contain at least one problematic record.
- To simplify `HAVING` clauses.

#### Syntax Rules and Structure

**PostgreSQL**
```
BOOL_OR ( boolean_expression ) → boolean
```

**Syntax Rules**

1. `BOOL_OR` ignores `NULL` values; it returns `false` only if all non-`NULL` values are `false`.
2. If all values are `NULL`, `BOOL_OR` returns `NULL`.
3. No standard SQL synonym is available in PostgreSQL due to `ANY` ambiguity.

**Constraints and Limitations**

- **PostgreSQL-specific**.
- MySQL and SQL Server require workarounds (e.g., `MAX(flag) = 1`).

#### Annotated Code Examples

**Example 1: Checking for Any Failure**

```sql
SELECT BOOL_OR(status = 'failed') AS any_failed
FROM test_runs
WHERE suite_id = 7;
```

**Expected Output**

| any_failed |
|------------|
| t          |

**Why This Output Occurs**: At least one test run in suite 7 has `status = 'failed'` .

#### Real-World Cases

- **Alerting**: Flagging any server with high error rate.
- **Fraud detection**: Identifying accounts with at least one suspicious transaction.
- **Quality assurance**: Checking whether any product in a batch failed inspection.

---

## 4. Statistical & Distribution Functions

### 4.1 `STDDEV`

#### Definitions

**Core Definition**
`STDDEV` returns the standard deviation of a set of numeric values.

**Technical Definition**
Standard deviation measures the dispersion of values around the mean. PostgreSQL provides `STDDEV` as a historical alias for `STDDEV_SAMP` (sample standard deviation), along with `STDDEV_POP` (population standard deviation) and `STDDEV_SAMP`. Return type is `double precision` for floating-point arguments and `numeric` otherwise . MySQL provides `STDDEV`/`STDDEV_POP` (population) and `STDDEV_SAMP` (sample), returning `DOUBLE` . SQL Server provides `STDEV` (sample) and `STDEVP` (population) .

**Beginner-Friendly Explanation**
Standard deviation tells you how spread out the values are—a small value means values are clustered around the average; a large value means they are widely spread.

#### Purposes

- To measure variability or volatility in data.
- To detect outliers and anomalies.
- To compare consistency across groups.
- To support statistical process control.

#### Syntax Rules and Structure

**PostgreSQL**
```
STDDEV ( expression ) → numeric/double precision   -- alias for STDDEV_SAMP
STDDEV_POP ( expression ) → numeric/double precision
STDDEV_SAMP ( expression ) → numeric/double precision
```
**MySQL**
```
STDDEV ( expression ) → double   -- population
STDDEV_POP ( expression ) → double
STDDEV_SAMP ( expression ) → double
```
**SQL Server**
```
STDEV ( expression ) → float     -- sample
STDEVP ( expression ) → float    -- population
```

**Syntax Rules**

1. PostgreSQL's `STDDEV` is a sample standard deviation (denominator `N-1`) .
2. MySQL's `STDDEV` is a population standard deviation (denominator `N`) .
3. SQL Server's `STDEV` is sample; `STDEVP` is population .
4. `NULL` values are ignored.

**Constraints and Limitations**

- **Semantic divergence**: `STDDEV` means different things in PostgreSQL (sample) vs. MySQL (population).
- Small sample sizes may produce `NULL` or misleading results.

#### Annotated Code Examples

**Example 1: PostgreSQL `STDDEV_SAMP`**

```sql
SELECT STDDEV_SAMP(temp_lo) AS sample_stddev
FROM weather;
```

**Expected Output (illustrative)**

| sample_stddev |
|---------------|
| 4.7258        |

**Example 2: MySQL Population Standard Deviation**

```sql
SELECT STDDEV(temp_lo) AS population_stddev
FROM weather;
```

#### Real-World Cases

- **Finance**: Volatility of stock returns.
- **Manufacturing**: Consistency of product dimensions.
- **Education**: Spread of test scores.

---

### 4.2 `VARIANCE`

#### Definitions

**Core Definition**
`VARIANCE` returns the variance of a set of numeric values—the square of the standard deviation.

**Technical Definition**
PostgreSQL provides `VARIANCE` as a historical alias for `VAR_SAMP` (sample variance), along with `VAR_POP` (population variance) and `VAR_SAMP` . MySQL provides `VARIANCE`/`VAR_POP` (population) and `VAR_SAMP` (sample) . SQL Server provides `VAR` (sample) and `VARP` (population).

**Beginner-Friendly Explanation**
Variance is another measure of spread—it is the average of the squared differences from the mean. It is the square of standard deviation.

#### Purposes

- To measure dispersion in squared units.
- To support ANOVA and other statistical tests.
- To compare variability across datasets.
- To derive standard deviation (`SQRT(VARIANCE)`).

#### Syntax Rules and Structure

**PostgreSQL**
```
VARIANCE ( expression ) → numeric/double precision   -- alias for VAR_SAMP
VAR_POP ( expression ) → numeric/double precision
VAR_SAMP ( expression ) → numeric/double precision
```
**MySQL**
```
VARIANCE ( expression ) → double   -- population
VAR_POP ( expression ) → double
VAR_SAMP ( expression ) → double
```
**SQL Server**
```
VAR ( expression ) → float     -- sample
VARP ( expression ) → float    -- population
```

**Syntax Rules**

1. PostgreSQL's `VARIANCE` is sample variance .
2. MySQL's `VARIANCE` is population variance .
3. SQL Server's `VAR` is sample; `VARP` is population.

**Constraints and Limitations**

- Same semantic divergence as `STDDEV`.
- Squared units make interpretation less intuitive than standard deviation.

#### Annotated Code Examples

**Example 1: PostgreSQL Sample Variance**

```sql
SELECT VAR_SAMP(temp_lo) AS sample_variance
FROM weather;
```

**Expected Output (illustrative)**

| sample_variance |
|-----------------|
| 22.3333         |

#### Real-World Cases

- **Risk management**: Portfolio variance.
- **Quality control**: Process variance.
- **Psychology**: Variability of test responses.

---

### 4.3 `PERCENTILE_CONT`

#### Definitions

**Core Definition**
`PERCENTILE_CONT` returns a value corresponding to a specified percentile in a continuous distribution, interpolating between adjacent values if necessary.

**Technical Definition**
`PERCENTILE_CONT(fraction) WITHIN GROUP (ORDER BY sort_expression)` is an ordered-set aggregate that computes a continuous percentile. The `fraction` must be between 0 and 1. If the exact percentile falls between two input values, the function interpolates linearly between them . PostgreSQL supports both single-fraction and array-of-fractions forms. SQL Server implements `PERCENTILE_CONT` as a window function (not a `WITHIN GROUP` aggregate) .

**Beginner-Friendly Explanation**
`PERCENTILE_CONT` finds the value below which a given percentage of data falls, smoothly interpolating between data points—like finding the median (50th percentile).

#### Purposes

- To compute medians and other percentiles for continuous data.
- To analyse distributions without assuming normality.
- To support SLA reporting (e.g., 95th percentile response time).
- To compare distributions across groups.

#### Syntax Rules and Structure

**PostgreSQL**
```
PERCENTILE_CONT ( fraction ) WITHIN GROUP ( ORDER BY sort_expression ) → double precision or interval
PERCENTILE_CONT ( fractions[] ) WITHIN GROUP ( ORDER BY sort_expression ) → double precision[] or interval[]
```
**SQL Server**
```
PERCENTILE_CONT ( numeric_literal ) WITHIN GROUP ( ORDER BY order_by_expression ) OVER ( [ partition_by_clause ] )
```

**Syntax Rules**

1. `fraction` must be between 0 and 1; `NULL` fraction returns `NULL` .
2. The `WITHIN GROUP (ORDER BY ...)` clause is required in PostgreSQL.
3. SQL Server uses the `OVER` clause instead of `WITHIN GROUP` .
4. `NULL` values in the sorted input are ignored.

**Constraints and Limitations**

- **Not available in MySQL**.
- SQL Server syntax differs (window function vs. ordered-set aggregate).
- Interpolation may produce values not present in the input data.

#### Annotated Code Examples

**Example 1: PostgreSQL Median (50th Percentile)**

```sql
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY income) AS median_income
FROM households;
```

**Expected Output (illustrative)**

| median_income |
|---------------|
| 50489         |

**Why This Output Occurs**: The median is interpolated between the two middle income values .

**Example 2: 95th Percentile Response Time**

```sql
SELECT PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY response_ms) AS p95
FROM api_requests;
```

#### Real-World Cases

- **SLA monitoring**: 95th percentile latency.
- **Compensation analysis**: Median salary.
- **Real estate**: Median home price.

---

### 4.4 `PERCENTILE_DISC`

#### Definitions

**Core Definition**
`PERCENTILE_DISC` returns the first value in the sorted input whose cumulative distribution is greater than or equal to the specified percentile.

**Technical Definition**
`PERCENTILE_DISC(fraction) WITHIN GROUP (ORDER BY sort_expression)` is an ordered-set aggregate that returns a discrete value from the input set—the smallest value whose cumulative distribution is ≥ `fraction`. Unlike `PERCENTILE_CONT`, it does not interpolate; it returns an actual input value . SQL Server implements it as a window function.

**Beginner-Friendly Explanation**
`PERCENTILE_DISC` finds the actual data value at a given percentile, without averaging between values—like finding the median but always returning one of the real data points.

#### Purposes

- To find an actual value at a percentile (not an interpolated one).
- To support percentile-based thresholds where interpolation is undesirable.
- To analyse discrete or categorical distributions.
- To compute percentiles that must be valid input values.

#### Syntax Rules and Structure

**PostgreSQL**
```
PERCENTILE_DISC ( fraction ) WITHIN GROUP ( ORDER BY sort_expression ) → same as sort expression
```
**SQL Server**
```
PERCENTILE_DISC ( numeric_literal ) WITHIN GROUP ( ORDER BY order_by_expression ) OVER ( ... )
```

**Syntax Rules**

1. `fraction` must be between 0 and 1.
2. Returns the first value whose cumulative distribution ≥ `fraction`.
3. Does not interpolate; result is always an input value.

**Constraints and Limitations**

- **Not available in MySQL**.
- SQL Server uses window-function syntax.

#### Annotated Code Examples

**Example 1: Discrete Median**

```sql
SELECT PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY rating) AS median_rating
FROM reviews;
```

**Expected Output (illustrative)**

| median_rating |
|---------------|
| 4             |

**Why This Output Occurs**: The function returns an actual rating value, not an interpolated one.

#### Real-World Cases

- **Survey analysis**: Median response category.
- **Inventory**: Reorder point based on actual demand values.
- **Performance grading**: Percentile-based grade boundaries using real scores.

---

### 4.5 `MODE`

#### Definitions

**Core Definition**
`MODE` returns the most frequent value in a set.

**Technical Definition**
`MODE() WITHIN GROUP (ORDER BY sort_expression)` is an ordered-set aggregate that returns the most frequent input value. If multiple values are equally frequent, one is chosen arbitrarily . MySQL provides a `MODE()` function that is not an aggregate in the same sense; SQL Server does not have a native `MODE` aggregate .

**Beginner-Friendly Explanation**
`MODE` finds the most common value—like finding the most frequently purchased product.

#### Purposes

- To identify the most common category or value.
- To detect the dominant pattern in data.
- To support market basket and frequency analysis.
- To find the “typical” value in categorical data.

#### Syntax Rules and Structure

**PostgreSQL**
```
MODE ( ) WITHIN GROUP ( ORDER BY sort_expression ) → same as sort expression
```

**Syntax Rules**

1. `MODE` takes no direct arguments; the aggregated input is specified entirely in `WITHIN GROUP (ORDER BY ...)`.
2. If there are multiple equally frequent values, the result is arbitrary .
3. `NULL` values are ignored.

**Constraints and Limitations**

- **PostgreSQL-specific** (MySQL's `MODE` is not the same aggregate; SQL Server lacks it).
- Arbitrary tie-breaking may not be suitable for deterministic reporting.
- Not eligible for partial mode (parallel aggregation) .

#### Annotated Code Examples

**Example 1: Most Frequent Value**

```sql
SELECT MODE() WITHIN GROUP (ORDER BY product_name) AS most_common_product
FROM orders;
```

**Expected Output (illustrative)**

| most_common_product |
|---------------------|
| Widget              |

**Why This Output Occurs**: “Widget” appears more often than any other product .

#### Real-World Cases

- **Retail**: Most popular product or size.
- **Demographics**: Most common age group.
- **IT**: Most frequent error code.

---

## 5. Aggregate Behavior & Optimization

### 5.1 Aggregate NULL Handling

#### Definitions

**Core Definition**
Aggregate NULL handling refers to the rules that determine how aggregate functions treat `NULL` values in their input.

**Technical Definition**
Most aggregate functions ignore `NULL` values entirely: `SUM`, `AVG`, `MIN`, `MAX`, `STDDEV`, `VARIANCE`, `STRING_AGG`, and `BOOL_AND`/`BOOL_OR` all skip `NULL`s. `COUNT(*)` counts all rows regardless of `NULL`s; `COUNT(expression)` counts only non-`NULL` values. `ARRAY_AGG` and `JSON_AGG` include `NULL`s by default in PostgreSQL, but strict variants skip them. Empty-set behaviour also differs: except for `COUNT`, aggregates return `NULL` when no rows are selected .

**Beginner-Friendly Explanation**
`NULL` means “unknown” or “missing.” Most aggregates simply ignore missing values, but `COUNT(*)` counts all rows even if some values are missing. This can lead to surprising results if you are not careful.

#### Purposes (of Understanding NULL Handling)

- To avoid incorrect results when data contains missing values.
- To choose the right aggregate variant (e.g., `COUNT(*)` vs. `COUNT(column)`).
- To use `COALESCE` appropriately when `NULL` results are undesirable.
- To write accurate `HAVING` clauses.

#### Syntax Rules and Structure (Key Behaviours)

| Aggregate | NULL Handling | Empty Set |
|-----------|---------------|-----------|
| `COUNT(*)` | Counts all rows | `0` |
| `COUNT(expr)` | Ignores `NULL` | `0` |
| `SUM` | Ignores `NULL` | `NULL` |
| `AVG` | Ignores `NULL` | `NULL` |
| `MIN`/`MAX` | Ignores `NULL` | `NULL` |
| `STRING_AGG` | Skips `NULL` | `NULL` |
| `ARRAY_AGG` | Includes `NULL` | `NULL` |
| `JSON_AGG` | Includes `NULL` | `NULL` |
| `BOOL_AND`/`BOOL_OR` | Ignores `NULL` | `NULL` (if all `NULL`) |

**Constraints and Limitations**

- `SUM` of no rows returns `NULL`, not `0`; use `COALESCE(SUM(x), 0)` .
- `AVG` denominator is the count of non-`NULL` values, which may differ from the total row count.
- `BOOL_AND`/`BOOL_OR` return `NULL` if all inputs are `NULL`.

#### Annotated Code Examples

**Example 1: `SUM` of No Rows Returns `NULL`**

```sql
SELECT SUM(amount) AS total
FROM orders
WHERE 1 = 0;  -- no rows
```

**Expected Output**

| total |
|-------|
| NULL  |

**Fix**:

```sql
SELECT COALESCE(SUM(amount), 0) AS total
FROM orders
WHERE 1 = 0;
```

#### Real-World Cases

- **Financial reports**: Ensuring totals show `0` instead of `NULL` for empty periods.
- **Data quality dashboards**: Distinguishing between “no data” and “zero.”
- **API responses**: Avoiding `null` values in JSON output.

---

### 5.2 Aggregate Performance

#### Definitions

**Core Definition**
Aggregate performance refers to the efficiency with which the database engine computes aggregate results, influenced by indexing, query structure, parallelisation, and data distribution.

**Technical Definition**
Aggregate performance is determined by the execution plan chosen by the query optimiser. Key factors include: index availability for `GROUP BY` and `MIN`/`MAX` (Loose Index Scan in MySQL); parallel aggregation support (partial mode in PostgreSQL); the presence of `DISTINCT` or `FILTER` clauses; and the cost of sorting for ordered-set aggregates. `COUNT(*)` on large tables requires a full scan or full index scan in PostgreSQL .

**Beginner-Friendly Explanation**
Aggregate performance is about how fast the database can calculate summaries. Some operations are fast if the right indexes exist; others require scanning the whole table. Understanding this helps you write queries that finish quickly.

#### Purposes (of Optimising Aggregate Performance)

- To reduce query execution time for large datasets.
- To avoid full table scans where index access is possible.
- To enable parallel aggregation for CPU-bound workloads.
- To design indexes that support `GROUP BY` and `MIN`/`MAX`.

#### Syntax Rules and Structure (Optimisation Techniques)

1. **Indexing for `GROUP BY`**: Ensure `GROUP BY` columns form a leftmost prefix of a B-tree index .
2. **Loose Index Scan (MySQL)**: When `GROUP BY` uses indexed columns and the only aggregates are `MIN`/`MAX` on the same indexed column, MySQL can use Loose Index Scan .
3. **Parallel Aggregation (PostgreSQL)**: Aggregates that support partial mode (e.g., `SUM`, `AVG`, `COUNT`, `STRING_AGG`, `ARRAY_AGG`) can participate in parallel plans .
4. **Avoiding `DISTINCT` Where Possible**: `COUNT(DISTINCT ...)` disables parallel aggregation in PostgreSQL .
5. **`FILTER` Clause**: Can improve performance by reducing the number of rows aggregated, but may interact with index usage .
6. **Covering Indexes**: Indexes that include all columns needed by the aggregate can avoid table access.

**Constraints and Limitations**

- Optimisation techniques are vendor-specific.
- `COUNT(*)` on large tables is inherently expensive in PostgreSQL .
- Parallel aggregation requires appropriate configuration (`max_parallel_workers_per_gather`, etc.).

#### Annotated Code Examples

**Example 1: Index Supporting `GROUP BY` (MySQL)**

```sql
-- Index: idx(c1, c2, c3)
SELECT c1, c2, MIN(c3), MAX(c3)
FROM t1
GROUP BY c1, c2;
```

**Explanation**: The `GROUP BY` columns form a leftmost prefix of the index, and the only aggregates are `MIN`/`MAX` on the next indexed column, enabling Loose Index Scan .

**Example 2: Parallel Aggregation (PostgreSQL)**

```sql
SET max_parallel_workers_per_gather = 4;
SELECT SUM(amount) FROM large_sales_table;
```

**Explanation**: `SUM` supports partial mode; with parallel workers enabled, PostgreSQL can compute partial aggregates in parallel and combine them .

#### Real-World Cases

- **Data warehousing**: Pre-aggregated summary tables and materialised views for expensive aggregates.
- **Real-time dashboards**: Using covering indexes and `FILTER` to speed up aggregate queries.
- **Large-scale analytics**: Leveraging parallel aggregation for `SUM`/`AVG` over billions of rows.

---

## References

1. **PostgreSQL 16 Documentation — Aggregate Functions** — https://www.postgresql.org/docs/16/functions-aggregate.html
2. **PostgreSQL 16 Tutorial — Aggregate Functions** — https://www.postgresql.org/docs/16/tutorial-agg.html
3. **PostgreSQL 16 Documentation — SQL Expressions (Aggregate Syntax)** — https://www.postgresql.org/docs/16/sql-expressions.html#SYNTAX-AGGREGATES
4. **MySQL 8.0 Reference Manual — Aggregate Function Descriptions** — https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html
5. **MySQL 8.0 Reference Manual — GROUP BY Optimization** — https://dev.mysql.com/doc/refman/8.0/en/group-by-optimization.html
6. **Microsoft SQL Server — Aggregate Functions (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/functions/aggregate-functions-transact-sql
7. **Microsoft SQL Server — STRING_AGG (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/functions/string-agg-transact-sql
8. **Microsoft SQL Server — PERCENTILE_CONT (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/functions/percentile-cont-transact-sql
9. **Microsoft SQL Server — PERCENTILE_DISC (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/functions/percentile-disc-transact-sql
10. **PostgreSQL Documentation — Parallel Query** — https://www.postgresql.org/docs/current/parallel-query.html
11. **PostgreSQL Wiki — Aggregate Functions** — https://wiki.postgresql.org/wiki/Aggregate_Functions
12. **SQL:2016 Standard — ISO/IEC 9075-2:2016 (Foundation)** — https://www.iso.org/standard/63555.html

---

This cheat sheet provides a comprehensive reference for SQL aggregate functions across PostgreSQL, MySQL, and SQL Server, with emphasis on vendor-specific behaviour, NULL handling, and performance optimisation. Always consult your specific RDBMS documentation for version-specific features and limitations.