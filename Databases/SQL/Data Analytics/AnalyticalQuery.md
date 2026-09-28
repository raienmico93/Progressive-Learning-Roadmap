# SQL Analytical Query Design: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL analytical query design is the practice of structuring SQL queries to explore, summarize, segment, and identify trends within data, using declarative set-based operations rather than procedural row-by-row logic.

**Technical Definition:** Analytical query design encompasses the use of SQL constructs—`SELECT` with filtering and sampling, aggregate functions for statistical summarization, `GROUP BY` with advanced grouping extensions (`GROUPING SETS`, `CUBE`, `ROLLUP`), conditional expressions (`CASE`, `COALESCE`, `NULLIF`) for segmentation, and window functions with ordering for trend analysis—to transform raw data into actionable insights. These techniques form the foundation of business intelligence, data warehousing, and exploratory data analysis.

**Beginner-Friendly Explanation:** Analytical query design is like being a detective examining a crime scene. You start by looking around broadly (exploratory queries) to see what is there. Then you count and measure things (descriptive statistics). You group similar things together (aggregation). You label different types of things (segmentation). And finally, you look for patterns over time (trend analysis). SQL is the language you use to do all of this with data.

### Key Characteristics

- **Exploratory-first:** Analytical workflows begin with broad scanning to understand data shape and content before drilling into specifics.
- **Set-based:** Analytical queries operate on entire datasets, leveraging the database engine's optimizations rather than application loops.
- **Aggregation-centric:** Reduction of detail rows into summary metrics is the core operation of analytical SQL.
- **Multi-dimensional:** Modern SQL supports grouping across multiple dimensions simultaneously through `GROUPING SETS`, `CUBE`, and `ROLLUP`.
- **Order-aware:** Trend analysis depends on sequential ordering using window functions and `ORDER BY`.
- **Composable:** Analytical queries are built by composing smaller query blocks (CTEs, subqueries, joins) into layered pipelines.

### Prerequisites

- Proficiency with basic SQL `SELECT`, `WHERE`, `JOIN`, and `GROUP BY` syntax.
- Understanding of `NULL` semantics and three-valued logic.
- Familiarity with aggregate functions (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`).
- Awareness of window functions and the `OVER` clause.
- Knowledge of data types and casting behavior.

### Related Programming Areas

- Business intelligence and reporting.
- Data warehousing and dimensional modeling.
- Exploratory data analysis (EDA) and data science.
- ETL/ELT pipeline design.
- Performance tuning and query optimization.

### Core Concepts / Features

1. **Exploratory Queries** (broad scanning, structural inspection, sampling)
2. **Descriptive Statistics** (central tendency, dispersion, count-based evaluations)
3. **Aggregation** (multi-dimensional reduction, advanced grouping extensions)
4. **Segmentation** (categorical logic, binning, string parsing)
5. **Trend Analysis** (sequential ordering, window functions, directional patterns)


## Core Concept 1: Exploratory Queries

### Definitions

**Core Definition:** Exploratory queries are broad, unfiltered or lightly filtered SQL statements used to inspect the structure, content, and shape of a dataset before applying detailed analysis.

**Technical Definition:** Exploratory queries employ `SELECT *`, `LIMIT`/`OFFSET` for pagination and sampling, `TABLESAMPLE` for statistical row sampling, and `INFORMATION_SCHEMA` queries for structural inspection of tables, columns, and constraints. These queries prioritize breadth of coverage over precision, allowing analysts to understand data volume, null distribution, cardinality, and value ranges before designing more targeted queries.

**Beginner-Friendly Explanation:** Exploratory queries are like browsing a new library. You walk through the aisles to see what sections exist, pick up a few random books to see what they look like, and check the card catalog to understand what is available. You are not reading every book cover to cover—you are getting the lay of the land.

### Purposes

- To inspect table structures and column metadata using `INFORMATION_SCHEMA`.
- To view sample rows from a table without retrieving the entire dataset using `LIMIT` and `OFFSET`.
- To extract statistically representative samples using `TABLESAMPLE`.
- To identify value ranges, null distributions, and cardinality before designing targeted queries.
- To verify data types and formats before applying transformations.

### Syntax Rules and Structure

#### Complete General Syntax (LIMIT and OFFSET)

```sql
SELECT column_list
FROM table_name
[WHERE condition]
[ORDER BY column_list]
LIMIT row_count [OFFSET offset_count];

-- PostgreSQL and MySQL compatible syntax:
SELECT column_list
FROM table_name
LIMIT row_count OFFSET offset_count;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `LIMIT row_count` | Maximum number of rows to return. `LIMIT row_count` is equivalent to `LIMIT 0, row_count`. |
| `OFFSET offset_count` | Number of rows to skip before starting to return rows. |
| `ORDER BY` | Determines which rows are considered "first" for the limit; essential for deterministic pagination. |

#### Complete General Syntax (TABLESAMPLE — PostgreSQL)

```sql
SELECT * FROM table_name TABLESAMPLE sampling_method (argument [, ...]) [ REPEATABLE (seed) ];
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `sampling_method` | `BERNOULLI` (row-level sampling) or `SYSTEM` (page-level sampling). |
| `argument` | Percentage of the table to sample (0–100). |
| `REPEATABLE (seed)` | Optional seed for reproducible sampling. |
| `SYSTEM_ROWS` | Extension: samples an exact number of rows. |
| `SYSTEM_TIME` | Extension: samples for a time limit. |

#### Complete General Syntax (INFORMATION_SCHEMA)

```sql
SELECT table_schema, table_name, column_name, data_type, is_nullable
FROM INFORMATION_SCHEMA.COLUMNS
WHERE table_name = 'table_name'
ORDER BY ordinal_position;
```

#### Syntax Rules

- **LIMIT/OFFSET:** For compatibility with PostgreSQL, MySQL also supports the `LIMIT row_count OFFSET offset` syntax. If `LIMIT` occurs within a subquery and also is applied in the outer query, the outermost `LIMIT` takes precedence.
- **TABLESAMPLE:** A `TABLESAMPLE` clause after a `table_name` indicates that the specified sampling method should be used to retrieve a subset of the rows in that table. This sampling precedes the application of any other filters such as `WHERE` clauses.
- **TABLESAMPLE SYSTEM:** Uses pages; 50 means every page of the table has a 50% chance of being drafted. You need to know the total row count and adjust the percentage to arrive at a specific sample size.
- **TABLESAMPLE BERNOULLI:** Uses records; with 50, every record of every page has a 50% chance. Again, needs to be combined with total row count and trimmed with `LIMIT` to arrive at a specific sample size.
- **INFORMATION_SCHEMA:** A SQL standard schema that provides metadata about objects across all catalogs in the metastore.

#### Constraints and Limitations

- **TABLESAMPLE without ORDER BY:** Sampling returns rows in an unspecified order; without `ORDER BY`, results are non-deterministic.
- **TABLESAMPLE SYSTEM row-count variance:** The number of live records on a page is not constant, so the actual sample size varies.
- **LIMIT without ORDER BY:** Without `ORDER BY`, the rows returned by `LIMIT` are unpredictable and may change between executions.
- **MySQL `LIMIT` in subqueries:** If `LIMIT` occurs within a parenthesized query expression and also is applied in the outer query, the results are undefined and may change in future MySQL versions.
- **SQL Server:** Does not support `LIMIT`/`OFFSET` directly; use `TOP` or `OFFSET ... FETCH NEXT`.

### Annotated Code Examples

#### Example 1: PostgreSQL — Sampling with TABLESAMPLE

```sql
-- Create a sample table
CREATE TABLE population (
    id SERIAL PRIMARY KEY,
    name TEXT,
    age INT,
    city TEXT
);

-- Insert 10,000 rows
INSERT INTO population (name, age, city)
SELECT
    'Person_' || i,
    (random() * 80 + 18)::INT,
    CASE (i % 5)
        WHEN 0 THEN 'New York'
        WHEN 1 THEN 'Los Angeles'
        WHEN 2 THEN 'Chicago'
        WHEN 3 THEN 'Houston'
        ELSE 'Phoenix'
    END
FROM generate_series(1, 10000) AS i;

-- SYSTEM sampling: ~10% of pages
SELECT * FROM population TABLESAMPLE SYSTEM(10) LIMIT 5;

-- BERNOULLI sampling: ~10% of rows
SELECT * FROM population TABLESAMPLE BERNOULLI(10) REPEATABLE(42) LIMIT 5;

-- SYSTEM_ROWS: exact sample size (requires tsm_system_rows extension)
CREATE EXTENSION IF NOT EXISTS tsm_system_rows;
SELECT * FROM population TABLESAMPLE SYSTEM_ROWS(100);
```

**Expected Output (SYSTEM, partial):**

```
 id  |    name    | age |    city
-----+------------+-----+------------
 1   | Person_1   |  45 | New York
 2   | Person_2   |  32 | Los Angeles
 3   | Person_3   |  67 | Chicago
...
```

**Why This Works:** `TABLESAMPLE SYSTEM(10)` selects approximately 10% of the table's pages, returning all rows on those pages. `TABLESAMPLE BERNOULLI(10)` selects each row with 10% probability. `SYSTEM_ROWS(100)` returns exactly 100 rows. `REPEATABLE(42)` ensures the same sample is returned for a given seed.

#### Example 2: SQL Server — Structural Inspection with INFORMATION_SCHEMA

```sql
-- List all tables and their columns
SELECT
    t.TABLE_SCHEMA,
    t.TABLE_NAME,
    c.COLUMN_NAME,
    c.DATA_TYPE,
    c.CHARACTER_MAXIMUM_LENGTH,
    c.IS_NULLABLE
FROM INFORMATION_SCHEMA.TABLES t
JOIN INFORMATION_SCHEMA.COLUMNS c
    ON t.TABLE_SCHEMA = c.TABLE_SCHEMA
    AND t.TABLE_NAME = c.TABLE_NAME
WHERE t.TABLE_TYPE = 'BASE TABLE'
ORDER BY t.TABLE_SCHEMA, t.TABLE_NAME, c.ORDINAL_POSITION;
```

**Expected Output:**

```
TABLE_SCHEMA | TABLE_NAME | COLUMN_NAME | DATA_TYPE | CHARACTER_MAXIMUM_LENGTH | IS_NULLABLE
-------------+------------+-------------+-----------+--------------------------+------------
dbo          | Customers  | CustomerID  | int       | NULL                     | NO
dbo          | Customers  | Name        | nvarchar  | 100                      | YES
dbo          | Orders     | OrderID     | int       | NULL                     | NO
dbo          | Orders     | CustomerID  | int       | NULL                     | YES
...
```

**Why This Works:** The `INFORMATION_SCHEMA.TABLES` and `INFORMATION_SCHEMA.COLUMNS` views provide SQL-standard metadata about tables and columns. Joining them gives a complete picture of the database schema.

### Real-World Cases

- **Data profiling:** Inspecting column types, nullability, and value ranges before ETL development.
- **Schema discovery:** Understanding an unfamiliar database's structure before writing queries.
- **Sampling for development:** Extracting a representative subset of production data for testing.
- **Pagination:** Implementing `LIMIT`/`OFFSET` for web application pagination.

### References

- PostgreSQL: SELECT (LIMIT and OFFSET) — https://www.postgresql.org/docs/current/sql-select.html
- PostgreSQL: TABLESAMPLE — https://www.postgresql.org/docs/current/sql-select.html
- PostgreSQL: tsm_system_rows — https://www.postgresql.org/docs/current/tsm-system-rows.html
- MySQL: SELECT (LIMIT) — https://dev.mysql.com/doc/refman/8.0/en/select.html
- SQL Server: Information Schema Views — https://learn.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/system-information-schema-views-transact-sql
- Databricks: Information Schema — https://learn.microsoft.com/en-us/azure/databricks/sql/language-manual/sql-ref-information-schema


## Core Concept 2: Descriptive Statistics

### Definitions

**Core Definition:** Descriptive statistics are aggregate calculations that summarize the central tendency, dispersion, and distribution of a dataset using SQL aggregate functions.

**Technical Definition:** Descriptive statistics in SQL use aggregate functions—`MIN`, `MAX`, `AVG`, `VARIANCE`, `STDDEV`, `COUNT`, and `COUNT(DISTINCT)`—to compute summary metrics across a set of rows. `AVG` computes the arithmetic mean; `VARIANCE` and `STDDEV` measure dispersion; `MIN` and `MAX` provide the range; `COUNT` and `COUNT(DISTINCT)` quantify volume and cardinality. These functions ignore NULL values by default, making them robust to missing data.

**Beginner-Friendly Explanation:** Descriptive statistics are like taking a snapshot of your data to understand what it looks like. The average tells you what is typical. The standard deviation tells you how spread out the values are. The minimum and maximum tell you the range. And the count tells you how many data points you have.

### Purposes

- To compute the average, sum, minimum, and maximum of numeric columns.
- To measure the dispersion of values using variance and standard deviation.
- To count rows and distinct values for cardinality assessment.
- To summarize data distributions before applying more complex analysis.
- To establish baselines and detect anomalies.

### Syntax Rules and Structure

#### Complete General Syntax (Aggregate Functions)

```sql
SELECT
    aggregate_function(expression) [FILTER (WHERE condition)]
FROM table_name
[WHERE condition]
[GROUP BY column_list];
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `AVG(expression)` | Arithmetic mean of the input values. |
| `MIN(expression)` | Minimum value of the input values. |
| `MAX(expression)` | Maximum value of the input values. |
| `COUNT(*)` | Number of input rows. |
| `COUNT(expression)` | Number of input rows for which the value of expression is not null. |
| `COUNT(DISTINCT expression)` | Number of distinct non-null values. |
| `STDDEV(expression)` | Historical alias for `stddev_samp` (sample standard deviation). |
| `STDDEV_POP(expression)` | Population standard deviation. |
| `VARIANCE(expression)` | Historical alias for `var_samp` (sample variance). |
| `VAR_POP(expression)` | Population variance. |

#### Syntax Rules

- **NULL handling:** Most aggregate functions ignore NULL values. `COUNT(*)` counts all rows including those with NULLs. `COUNT(expression)` counts only non-null values.
- **FILTER clause:** A `FILTER` clause can be attached to an aggregate function to process only the rows matching a boolean condition.
- **DISTINCT:** `AVG(DISTINCT <column_name>)`, `COUNT(DISTINCT <column_name>)`, and `SUM(DISTINCT <column_name>)` work with `ROLLUP`, `CUBE`, and `GROUPING SETS`.
- **Sample vs. population:** `STDDEV` and `VARIANCE` are sample statistics (divide by N-1); `STDDEV_POP` and `VAR_POP` are population statistics (divide by N).

#### Constraints and Limitations

- **Type restrictions:** `AVG`, `STDDEV`, and `VARIANCE` require numeric or interval input types.
- **NULL propagation:** If all input values are NULL, aggregate functions return NULL (except `COUNT(*)`, which returns 0).
- **Precision loss:** `AVG` on integers may return a numeric with more decimal places than expected; cast explicitly for control.

### Annotated Code Examples

#### Example 1: PostgreSQL — Descriptive Statistics

```sql
-- Create a table with numeric values
CREATE TABLE sensor_readings (
    reading_id SERIAL PRIMARY KEY,
    sensor_id INT,
    value NUMERIC(10,2)
);

INSERT INTO sensor_readings (sensor_id, value) VALUES
(1, 22.5), (1, 23.1), (1, 22.8), (1, 21.9), (1, 24.0),
(2, 19.8), (2, 20.2), (2, 19.5), (2, 20.1), (2, NULL);

-- Compute descriptive statistics per sensor
SELECT
    sensor_id,
    COUNT(*) AS row_count,
    COUNT(value) AS non_null_count,
    ROUND(AVG(value), 2) AS avg_value,
    MIN(value) AS min_value,
    MAX(value) AS max_value,
    ROUND(STDDEV(value), 3) AS stddev_value,
    ROUND(VARIANCE(value), 3) AS variance_value
FROM sensor_readings
GROUP BY sensor_id
ORDER BY sensor_id;
```

**Expected Output:**

```
 sensor_id | row_count | non_null_count | avg_value | min_value | max_value | stddev_value | variance_value
-----------+-----------+----------------+-----------+-----------+-----------+--------------+----------------
         1 |         5 |              5 |     22.86 |     21.90 |     24.00 |        0.807 |          0.651
         2 |         5 |              4 |     19.90 |     19.50 |     20.20 |        0.294 |          0.086
```

**Why This Works:** Sensor 1 has 5 non-null readings; sensor 2 has 4 non-null readings (one NULL value is ignored by `AVG`, `STDDEV`, and `VARIANCE`). `STDDEV` and `VARIANCE` measure the spread of values around the mean.

#### Example 2: PostgreSQL — COUNT(DISTINCT) for Cardinality

```sql
-- Count distinct cities and the distribution
SELECT
    COUNT(DISTINCT city) AS distinct_cities,
    COUNT(*) AS total_rows,
    COUNT(DISTINCT city) * 100.0 / COUNT(*) AS city_cardinality_pct
FROM population;
```

**Expected Output:**

```
 distinct_cities | total_rows | city_cardinality_pct
-----------------+------------+----------------------
               5 |      10000 |                 0.05
```

**Why This Works:** `COUNT(DISTINCT city)` returns the number of unique city values (5). The cardinality percentage shows that there are 5 distinct cities across 10,000 rows, indicating high repetition.

### Real-World Cases

- **Quality control:** Computing mean and standard deviation of manufacturing measurements to detect outliers.
- **Financial analysis:** Calculating average transaction value and variance by customer segment.
- **Sensor monitoring:** Tracking min, max, and average readings for anomaly detection.
- **Data profiling:** Assessing column cardinality and null rates before ETL design.

### References

- PostgreSQL: Aggregate Functions — https://www.postgresql.org/docs/current/functions-aggregate.html
- PostgreSQL: Aggregate Expressions — https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-AGGREGATES
- MySQL: Aggregate Function Descriptions — https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html
- SQL Server: Aggregate Functions — https://learn.microsoft.com/en-us/sql/t-sql/functions/aggregate-functions-transact-sql
- Oracle: Aggregate Functions — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/Aggregate-Functions.html


## Core Concept 3: Aggregation

### Definitions

**Core Definition:** Aggregation is the process of reducing a set of rows to summary rows by grouping them based on specified columns and computing aggregate values for each group.

**Technical Definition:** The `GROUP BY` clause groups rows based on a set of specified grouping expressions and computes aggregations on the group of rows based on one or more specified aggregate functions. Advanced grouping extensions—`GROUPING SETS`, `CUBE`, and `ROLLUP`—allow multiple aggregations to be produced in a single query, computing subtotals and grand totals across multiple dimensions without `UNION ALL` of separate queries.

**Beginner-Friendly Explanation:** Aggregation is like summarizing a spreadsheet. Instead of looking at every single row, you group rows by a category (like "region" or "product type") and compute totals, averages, and counts for each group. Advanced grouping extensions let you compute subtotals and grand totals at the same time—like a pivot table that shows totals by region, by product, and a grand total all in one result.

### Purposes

- To reduce large datasets into summary metrics for reporting and analysis.
- To compute subtotals and grand totals across multiple dimensions using `ROLLUP` and `CUBE`.
- To produce multiple aggregation levels in a single query using `GROUPING SETS`.
- To filter aggregated results using the `HAVING` clause.
- To avoid multiple `UNION ALL` queries for multi-level aggregations.

### Syntax Rules and Structure

#### Complete General Syntax (GROUP BY with Extensions)

```sql
SELECT column_list, aggregate_function(expression)
FROM table_name
[WHERE condition]
GROUP BY
    group_expression [, ...]
  | ROLLUP (group_expression [, ...])
  | CUBE (group_expression [, ...])
  | GROUPING SETS (grouping_set [, ...])
[HAVING condition];
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `group_expression` | Column name, column position, or expression to group by. |
| `ROLLUP` | Computes aggregations for a hierarchy: `ROLLUP(warehouse, product)` is equivalent to `GROUPING SETS((warehouse, product), (warehouse), ())`. |
| `CUBE` | Computes aggregations for all combinations: `CUBE(warehouse, product)` is equivalent to `GROUPING SETS((warehouse, product), (warehouse), (product), ())`. |
| `GROUPING SETS` | Explicitly specifies each grouping set. `GROUPING SETS ((a), (b))` is equivalent to `GROUPING SETS (a, b)`. |
| `GROUP BY ()` | Specifies an empty group that generates a grand total. |
| `HAVING` | Filters grouped results based on aggregate conditions. |

#### Syntax Rules

- **ROLLUP hierarchy:** The N elements of a `ROLLUP` specification results in N+1 `GROUPING SETS`. For example, `GROUP BY ROLLUP(warehouse, product)` is equivalent to `GROUP BY GROUPING SETS((warehouse, product), (warehouse), ())`.
- **CUBE combinations:** The N elements of a `CUBE` specification results in 2^N grouping sets. `GROUP BY CUBE(warehouse, product)` is equivalent to `GROUP BY GROUPING SETS((warehouse, product), (warehouse), (product), ())`.
- **Nested grouping:** `GROUPING SETS` can specify groupings equivalent to those returned by `ROLLUP` or `CUBE`, and can be nested within each other.
- **NULL placeholders:** In the output of `ROLLUP`, `CUBE`, and `GROUPING SETS`, NULL values in the grouping columns indicate subtotal or grand total rows. Use the `GROUPING()` function to distinguish NULL placeholders from actual NULL data.

#### Constraints and Limitations

- **SQL Server legacy syntax:** `WITH CUBE` and `WITH ROLLUP` are legacy syntax for backward compatibility only. Do not use them in new development; use `GROUPING SETS`, `CUBE`, or `ROLLUP` instead.
- **Materialized views (Oracle):** The `ROLLUP`, `CUBE`, and `GROUPING SETS` clauses are not supported in a materialized view definition.
- **Nested `GROUPING SETS` (SQL Server):** `GROUP BY GROUPING SETS (A1, A2, ...An, GROUPING SETS (C1, C2, ...Cn))` is allowed in the SQL-2006 standard but not in Transact-SQL.
- **Performance:** Aggregation across many dimensions can be computationally expensive; `GROUPING SETS` is more efficient than multiple `UNION ALL` queries but still requires careful index design.

### Annotated Code Examples

#### Example 1: PostgreSQL — ROLLUP for Hierarchical Subtotals

```sql
-- Create a sales table
CREATE TABLE sales (
    sale_id SERIAL PRIMARY KEY,
    region TEXT,
    product TEXT,
    amount NUMERIC(10,2)
);

INSERT INTO sales (region, product, amount) VALUES
('North', 'Widget', 100),
('North', 'Gadget', 200),
('South', 'Widget', 150),
('South', 'Gadget', 250),
('East', 'Widget', 120),
('East', 'Gadget', 180);

-- ROLLUP: subtotals by region and grand total
SELECT
    COALESCE(region, 'ALL REGIONS') AS region,
    product,
    SUM(amount) AS total_sales
FROM sales
GROUP BY ROLLUP(region, product)
ORDER BY region, product;
```

**Expected Output:**

```
  region    | product | total_sales
------------+---------+-------------
 ALL REGIONS|         |        1000
 East       | Gadget  |         180
 East       | Widget  |         120
 East       |         |         300
 North      | Gadget  |         200
 North      | Widget  |         100
 North      |         |         300
 South      | Gadget  |         250
 South      | Widget  |         150
 South      |         |         400
```

**Why This Works:** `ROLLUP(region, product)` produces grouping sets for `(region, product)`, `(region)`, and `()`. The rows with NULL `product` are region subtotals; the row with NULL `region` is the grand total. `COALESCE` makes the grand total row's region label explicit.

#### Example 2: SQL Server — CUBE for All Combinations

```sql
-- CUBE: all combinations of region and product
SELECT
    ISNULL(region, 'ALL REGIONS') AS region,
    ISNULL(product, 'ALL PRODUCTS') AS product,
    SUM(amount) AS total_sales
FROM sales
GROUP BY CUBE(region, product)
ORDER BY region, product;
```

**Expected Output:**

```
  region    |   product   | total_sales
------------+-------------+-------------
 ALL REGIONS| ALL PRODUCTS|        1000
 ALL REGIONS| Gadget      |         630
 ALL REGIONS| Widget      |         370
 East       | ALL PRODUCTS|         300
 East       | Gadget      |         180
 East       | Widget      |         120
 North      | ALL PRODUCTS|         300
 North      | Gadget      |         200
 North      | Widget      |         100
 South      | ALL PRODUCTS|         400
 South      | Gadget      |         250
 South      | Widget      |         150
```

**Why This Works:** `CUBE(region, product)` produces grouping sets for all combinations: `(region, product)`, `(region)`, `(product)`, and `()`. This shows totals by region, by product, and the grand total—all in one query. `ISNULL` replaces NULL placeholders with readable labels.

#### Example 3: MySQL — GROUPING SETS

```sql
-- GROUPING SETS: explicit control over which groupings to compute
SELECT
    IFNULL(region, 'ALL REGIONS') AS region,
    IFNULL(product, 'ALL PRODUCTS') AS product,
    SUM(amount) AS total_sales
FROM sales
GROUP BY GROUPING SETS (
    (region, product),
    (region),
    ()
)
ORDER BY region, product;
```

**Expected Output:**

```
  region    |   product   | total_sales
------------+-------------+-------------
 ALL REGIONS| ALL PRODUCTS|        1000
 East       |             |         300
 North      |             |         300
 South      |             |         400
 East       | Gadget      |         180
 East       | Widget      |         120
 North      | Gadget      |         200
 North      | Widget      |         100
 South      | Gadget      |         250
 South      | Widget      |         150
```

**Why This Works:** `GROUPING SETS ((region, product), (region), ())` computes exactly three grouping levels: full detail by region and product, subtotals by region, and a grand total. This is equivalent to `ROLLUP(region, product)` but with explicit control over which groupings are included.

### Real-World Cases

- **Financial reporting:** Producing income statements with subtotals by department and grand totals.
- **Sales analytics:** Generating reports with totals by region, by product, and by region-product combination.
- **Inventory management:** Computing stock levels at warehouse, product, and warehouse-product levels.
- **Web analytics:** Aggregating page views by device, by country, and by device-country combination.

### References

- PostgreSQL: GROUP BY and Grouping Sets — https://www.postgresql.org/docs/current/queries-table-expressions.html
- MySQL: GROUP BY Modifiers — https://dev.mysql.com/doc/refman/8.0/en/group-by-modifiers.html
- SQL Server: GROUP BY (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/queries/select-group-by-transact-sql
- Oracle: GROUPING SETS, ROLLUP, and CUBE — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/SELECT.html
- AWS: GROUP BY Clause — https://docs.aws.amazon.com/opensearch-service/latest/developerguide/opensearch-service-dg.pdf


## Core Concept 4: Segmentation

### Definitions

**Core Definition:** Segmentation is the practice of classifying data rows into meaningful categories or bins using conditional logic, null-handling functions, and string-parsing operations.

**Technical Definition:** Segmentation uses `CASE WHEN` expressions to map continuous or coded values into discrete categories; `COALESCE` and `NULLIF` to handle nulls and sentinel values; data binning (e.g., `NTILE`, `WIDTH_BUCKET`, or arithmetic ranges) to group continuous values into buckets; and string functions (`SUBSTRING`, `REGEXP`, `SPLIT_PART`) to extract categorical information from unstructured text.

**Beginner-Friendly Explanation:** Segmentation is like sorting a pile of mail into different trays. You decide the rules: "If it is from a customer, put it in the 'Customer' tray; if it is a bill, put it in the 'Bill' tray." In SQL, you use `CASE` to write those rules and create new category columns.

### Purposes

- To classify rows into business-relevant categories using `CASE WHEN`.
- To replace NULL values with meaningful defaults using `COALESCE`.
- To convert sentinel values (e.g., `'N/A'`, `0`) into proper NULLs using `NULLIF`.
- To bin continuous values into discrete ranges for histogram-style analysis.
- To extract categorical information from unstructured text using string functions.

### Syntax Rules and Structure

#### Complete General Syntax (CASE)

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    [ELSE default_result]
END
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `WHEN condition THEN result` | A condition-result pair; evaluated in order. |
| `ELSE default_result` | Optional fallback returned when no condition is TRUE. |
| `END` | Terminates the `CASE` expression. |

#### Complete General Syntax (COALESCE and NULLIF)

```sql
COALESCE(value1, value2, ..., valueN)  -- Returns the first non-NULL value
NULLIF(value1, value2)                  -- Returns NULL if value1 = value2, else value1
```

#### Complete General Syntax (Data Binning with NTILE)

```sql
NTILE(number_of_buckets) OVER (ORDER BY column)
```

#### Syntax Rules

- **CASE evaluation order:** Conditions are evaluated sequentially; the first TRUE condition determines the result.
- **COALESCE type compatibility:** All arguments must be of compatible types; the return type is the common type of all arguments.
- **NULLIF equivalence:** `NULLIF(v1, v2)` is equivalent to `CASE WHEN v1 = v2 THEN NULL ELSE v1 END`.
- **NTILE:** Divides rows into a specified number of buckets based on the `ORDER BY` clause; buckets are as equal in size as possible.

#### Constraints and Limitations

- **CASE type compatibility:** All `THEN` and `ELSE` results must share a compatible data type.
- **COALESCE with NULL:** `COALESCE(NULL, NULL, 'default')` returns `'default'`; `COALESCE('value', 'default')` returns `'value'`.
- **NULLIF with NULL:** `NULLIF(NULL, value)` returns NULL; `NULLIF(value, NULL)` returns `value`.
- **NTILE with ties:** If the number of rows is not divisible by the number of buckets, the remaining rows are distributed one per bucket, starting with the first bucket.

### Annotated Code Examples

#### Example 1: PostgreSQL — Segmentation with CASE and COALESCE

```sql
-- Create a customer table
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name TEXT,
    lifetime_value NUMERIC(10,2),
    last_order_date DATE
);

INSERT INTO customers (customer_name, lifetime_value, last_order_date) VALUES
('Alice', 2500.00, '2026-09-01'),
('Bob', 1200.00, '2026-03-15'),
('Carol', 450.00, '2026-08-20'),
('Dave', NULL, NULL),
('Eve', 5000.00, '2026-09-25');

-- Segment customers based on lifetime value and recency
SELECT
    customer_name,
    COALESCE(lifetime_value, 0) AS lifetime_value,
    CASE
        WHEN lifetime_value IS NULL THEN 'Prospect'
        WHEN lifetime_value > 2000 AND last_order_date > CURRENT_DATE - INTERVAL '30 days' THEN 'VIP Active'
        WHEN lifetime_value > 2000 THEN 'VIP Dormant'
        WHEN last_order_date > CURRENT_DATE - INTERVAL '90 days' THEN 'Active'
        ELSE 'At Risk'
    END AS customer_segment
FROM customers
ORDER BY lifetime_value DESC;
```

**Expected Output:**

```
 customer_name | lifetime_value | customer_segment
---------------+----------------+------------------
 Eve           |        5000.00 | VIP Active
 Alice         |        2500.00 | VIP Active
 Bob           |        1200.00 | At Risk
 Carol         |         450.00 | Active
 Dave          |           0.00 | Prospect
```

**Why This Works:** The `CASE` expression evaluates conditions in order. Eve and Alice are VIP Active because their lifetime value exceeds 2000 and their last order is within 30 days. Bob is At Risk because his lifetime value is below 2000 and his last order is older than 90 days. Dave is a Prospect because his lifetime value is NULL. `COALESCE` displays NULL as 0 for readability.

#### Example 2: PostgreSQL — Data Binning with NTILE

```sql
-- Bin customers into quartiles by lifetime value
SELECT
    customer_name,
    lifetime_value,
    NTILE(4) OVER (ORDER BY lifetime_value DESC NULLS LAST) AS value_quartile
FROM customers
WHERE lifetime_value IS NOT NULL;
```

**Expected Output:**

```
 customer_name | lifetime_value | value_quartile
---------------+----------------+----------------
 Eve           |        5000.00 |              1
 Alice         |        2500.00 |              1
 Bob           |        1200.00 |              2
 Carol         |         450.00 |              3
```

**Why This Works:** `NTILE(4)` divides the rows into four quartiles based on descending lifetime value. Eve and Alice are in quartile 1 (highest value), Bob in quartile 2, and Carol in quartile 3. NULL values are excluded by the `WHERE` clause.

### Real-World Cases

- **Customer segmentation:** Classifying customers into VIP, Active, At Risk, and Prospect segments.
- **Risk assessment:** Binning credit scores into risk categories.
- **Product categorization:** Grouping products into price bands (Budget, Mid-range, Premium).
- **Data cleansing:** Using `NULLIF` to convert sentinel values like `'N/A'` into proper NULLs.

### References

- PostgreSQL: Conditional Expressions — https://www.postgresql.org/docs/current/functions-conditional.html
- PostgreSQL: Window Functions (NTILE) — https://www.postgresql.org/docs/current/functions-window.html
- MySQL: CASE Operator — https://dev.mysql.com/doc/refman/8.0/en/flow-control-functions.html
- SQL Server: CASE (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/language-elements/case-transact-sql


## Core Concept 5: Trend Analysis

### Definitions

**Core Definition:** Trend analysis is the practice of identifying directional patterns and changes over sequential data using ordered window functions, lag/lead comparisons, and running aggregates.

**Technical Definition:** Trend analysis uses window functions—`LAG`, `LEAD`, `ROW_NUMBER`, `RANK`, `SUM() OVER`, `AVG() OVER`—combined with `ORDER BY` to compute metrics that compare rows to their neighbors in a sequence. These functions perform calculations across sets of rows related to the current query row, enabling running totals, moving averages, period-over-period comparisons, and directional change detection without self-joins or application loops.

**Beginner-Friendly Explanation:** Trend analysis is like tracking the stock market. You want to know: Is the price going up or down compared to yesterday? What is the average over the last 7 days? SQL window functions let you compare each row to the previous row, calculate running totals, and compute moving averages—all in a single query.

### Purposes

- To compare each row to its previous or next row using `LAG` and `LEAD`.
- To compute running totals and cumulative sums using `SUM() OVER`.
- To calculate moving averages for smoothing time-series data.
- To rank rows within partitions using `ROW_NUMBER`, `RANK`, and `DENSE_RANK`.
- To detect directional changes (increase/decrease) by comparing consecutive values.

### Syntax Rules and Structure

#### Complete General Syntax (Window Functions)

```sql
function_name(arguments) OVER (
    [PARTITION BY partition_expression]
    [ORDER BY sort_expression [ASC | DESC]]
    [frame_clause]
)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `PARTITION BY` | Divides rows into groups for the window calculation. |
| `ORDER BY` | Determines the order of rows within each partition. |
| `frame_clause` | Defines the set of rows in the window: `ROWS BETWEEN ... AND ...` or `RANGE BETWEEN ... AND ...`. |
| `LAG(column, offset)` | Accesses the value from the previous row (offset rows back). |
| `LEAD(column, offset)` | Accesses the value from the next row (offset rows forward). |
| `ROW_NUMBER()` | Unique sequential number within the partition. |
| `RANK()` | Rank with gaps for ties. |
| `SUM() OVER (...)` | Running or windowed sum. |
| `AVG() OVER (...)` | Running or windowed average. |

#### Syntax Rules

- **ORDER BY is mandatory for ranking and lag/lead:** Window functions that depend on ordering (e.g., `LAG`, `LEAD`, `ROW_NUMBER`) require an `ORDER BY` clause in the `OVER` specification.
- **Default frame:** When an `ORDER BY` is specified, the default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. When no `ORDER BY` is specified, the frame is the entire partition.
- **ROWS vs. RANGE:** `ROWS` counts physical rows; `RANGE` includes peer rows (rows with the same `ORDER BY` value).
- **Time-based windows:** `RANGE BETWEEN INTERVAL '7 days' PRECEDING AND CURRENT ROW` includes all rows where the timestamp falls within the trailing 7-day window, regardless of how many rows there are.
- **Window function placement:** Window functions can only appear in the `SELECT` list and `ORDER BY` clause; they cannot appear in `WHERE`, `GROUP BY`, or `HAVING`.

#### Constraints and Limitations

- **Cannot use in WHERE:** To filter on a window function result, wrap it in a CTE or subquery and filter in the outer query.
- **Performance:** Window functions require sorting; large partitions can consume significant memory.
- **SQL standard `IGNORE NULLS`:** The SQL standard defines `RESPECT NULLS` or `IGNORE NULLS` for `lead`, `lag`, `first_value`, `last_value`, and `nth_value`, but support varies by database.
- **MySQL:** Window functions require MySQL 8.0+.

### Annotated Code Examples

#### Example 1: PostgreSQL — Period-over-Period Comparison with LAG

```sql
-- Create a daily sales table
CREATE TABLE daily_sales (
    sale_date DATE PRIMARY KEY,
    amount NUMERIC(10,2)
);

INSERT INTO daily_sales VALUES
('2026-09-01', 1000),
('2026-09-02', 1200),
('2026-09-03', 900),
('2026-09-04', 1500),
('2026-09-05', 1400),
('2026-09-06', 1600),
('2026-09-07', 1100);

-- Compute day-over-day change and 3-day moving average
SELECT
    sale_date,
    amount,
    LAG(amount, 1) OVER (ORDER BY sale_date) AS prev_day_amount,
    amount - LAG(amount, 1) OVER (ORDER BY sale_date) AS day_over_day_change,
    ROUND(AVG(amount) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ), 2) AS moving_avg_3day
FROM daily_sales
ORDER BY sale_date;
```

**Expected Output:**

```
 sale_date  | amount | prev_day_amount | day_over_day_change | moving_avg_3day
------------+--------+-----------------+---------------------+-----------------
 2026-09-01 |   1000 |          (null) |              (null) |         1000.00
 2026-09-02 |   1200 |            1000 |                 200 |         1100.00
 2026-09-03 |    900 |            1200 |                -300 |         1033.33
 2026-09-04 |   1500 |             900 |                 600 |         1200.00
 2026-09-05 |   1400 |            1500 |                -100 |         1266.67
 2026-09-06 |   1600 |            1400 |                 200 |         1500.00
 2026-09-07 |   1100 |            1600 |                -500 |         1366.67
```

**Why This Works:** `LAG(amount, 1)` accesses the previous day's amount. The day-over-day change is computed by subtracting the previous amount from the current amount. The 3-day moving average uses `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW` to include the current row and the two preceding rows. `NULL` is returned for the first row because there is no previous row.

#### Example 2: PostgreSQL — Running Total and Cumulative Sum

```sql
-- Compute running total of sales
SELECT
    sale_date,
    amount,
    SUM(amount) OVER (ORDER BY sale_date) AS running_total,
    ROUND(
        amount * 100.0 / SUM(amount) OVER (),
        2
    ) AS pct_of_total
FROM daily_sales
ORDER BY sale_date;
```

**Expected Output:**

```
 sale_date  | amount | running_total | pct_of_total
------------+--------+---------------+--------------
 2026-09-01 |   1000 |          1000 |         12.66
 2026-09-02 |   1200 |          2200 |         15.19
 2026-09-03 |    900 |          3100 |         11.39
 2026-09-04 |   1500 |          4600 |         18.99
 2026-09-05 |   1400 |          6000 |         17.72
 2026-09-06 |   1600 |          7600 |         20.25
 2026-09-07 |   1100 |          8700 |         13.92
```

**Why This Works:** `SUM(amount) OVER (ORDER BY sale_date)` computes a running total (cumulative sum) that increases with each row. `SUM(amount) OVER ()` (without `ORDER BY`) computes the grand total across all rows (8700). Dividing each row's amount by the grand total gives the percentage contribution.

### Real-World Cases

- **Financial analysis:** Computing day-over-day stock price changes and moving averages.
- **Sales reporting:** Tracking running totals and month-over-month growth.
- **Web analytics:** Analyzing daily active users and period-over-period retention.
- **Operations monitoring:** Detecting anomalies by comparing current metrics to moving averages.

### References

- PostgreSQL: Window Functions — https://www.postgresql.org/docs/current/functions-window.html
- PostgreSQL: Window Function Calls — https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-WINDOW-FUNCTIONS
- MySQL: Window Functions — https://dev.mysql.com/doc/refman/8.0/en/window-functions.html
- SQL Server: Window Functions — https://learn.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql
- Oracle: Analytic Functions — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/Analytic-Functions.html


## Summary Table: Analytical Query Design Techniques

| Technique | Primary Constructs | Typical Use Case |
|-----------|-------------------|------------------|
| **Exploratory Queries** | `SELECT *`, `LIMIT`, `OFFSET`, `TABLESAMPLE`, `INFORMATION_SCHEMA` | Data profiling, schema discovery, sampling |
| **Descriptive Statistics** | `MIN`, `MAX`, `AVG`, `VARIANCE`, `STDDEV`, `COUNT(DISTINCT)` | Summarizing distributions, detecting outliers |
| **Aggregation** | `GROUP BY`, `HAVING`, `GROUPING SETS`, `CUBE`, `ROLLUP` | Multi-dimensional reporting, subtotals, grand totals |
| **Segmentation** | `CASE WHEN`, `COALESCE`, `NULLIF`, `NTILE`, `SUBSTRING` | Customer segmentation, risk classification, binning |
| **Trend Analysis** | `LAG`, `LEAD`, `ROW_NUMBER`, `SUM() OVER`, `AVG() OVER` | Running totals, moving averages, period-over-period comparison |


## Final Notes on Deprecated and Unsafe Features

- **SQL Server `WITH CUBE` and `WITH ROLLUP`:** Legacy syntax for backward compatibility only. Do not use in new development; use `GROUPING SETS`, `CUBE`, or `ROLLUP` instead.
- **MySQL `LIMIT` in subqueries:** If `LIMIT` occurs within a parenthesized query expression and also is applied in the outer query, the results are undefined and may change in future MySQL versions.
- **`SELECT *` in production queries:** Avoid using `SELECT *` in production code; explicitly list columns to prevent breakage when schema changes and to enable covering index optimization.
- **Window functions in `WHERE`:** Window functions cannot be used in `WHERE`, `GROUP BY`, or `HAVING` clauses. Wrap the query in a CTE or subquery to filter on window function results.
- **`TABLESAMPLE` without `ORDER BY`:** Sampling returns rows in an unspecified order; without `ORDER BY`, results are non-deterministic and may change between executions.
- **`NTILE` with ties:** If rows have tied values in the `ORDER BY`, the assignment to buckets is non-deterministic. Add a tie-breaking column to the `ORDER BY` for reproducibility.
- **Version-specific:** MySQL window functions require MySQL 8.0+; MySQL `INTERSECT` and `EXCEPT` require MySQL 8.0.31+; PostgreSQL `TABLESAMPLE SYSTEM_ROWS` requires the `tsm_system_rows` extension; SQL Server `GROUPING SETS` requires SQL Server 2008+.