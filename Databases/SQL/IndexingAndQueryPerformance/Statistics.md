# Comprehensive Programming Cheat Sheet: Statistics

## Topic Overview

### Definitions

**Core Definition**
Statistics in a relational database are metadata objects that describe the distribution of data in tables and columns, used by the query optimizer to estimate the cost and cardinality of alternative execution plans.

**Technical Definition**
Optimizer statistics are sampled summaries of data distribution stored in system catalogs (e.g., PostgreSQL `pg_statistic`, MySQL `column_statistics`, SQL Server statistics BLOBs). They include table-level metrics (row counts, page counts) and column-level metrics (null fraction, average width, number of distinct values, most common values with frequencies, histogram bounds, and physical-logical correlation). The optimizer uses these summaries to estimate selectivity of predicates, join cardinality, and the cost of access methods, join algorithms, and sort operations. Statistics are collected by `ANALYZE` (PostgreSQL), `ANALYZE TABLE` (MySQL), or `UPDATE STATISTICS`/auto-statistics (SQL Server).

**Beginner-Friendly Explanation**
Statistics are the database's cheat sheet about what its data looks like. They tell the optimizer things like "this column has about 500 distinct values," "10% of rows have status = 'pending'," or "the rows are stored in roughly the same order as this column." Without accurate statistics, the optimizer is guessing—and bad guesses lead to slow queries. Statistics are refreshed by commands like `ANALYZE` and `VACUUM ANALYZE`.

### Key Characteristics

- **Sampled, not exhaustive**: Statistics are derived from a sample of table rows, not the full table, so they are approximate.
- **Per-column and per-table**: Table statistics include row counts and page counts; column statistics describe value distribution.
- **Used for selectivity estimation**: The optimizer combines statistics with predicate operators to estimate how many rows match a condition.
- **Cost-model input**: Cardinality estimates feed join-order selection, algorithm choice (nested loop vs. hash join), and memory allocation.
- **Staleness is the primary failure mode**: Outdated statistics cause bad plans; manual `ANALYZE` is often required after bulk loads.
- **Vendor-specific storage and commands**: PostgreSQL uses `pg_statistic`/`pg_stats` and `ANALYZE`; MySQL uses `column_statistics` and `ANALYZE TABLE ... UPDATE HISTOGRAM`; SQL Server stores statistics BLOBs and supports auto-statistics.

### Prerequisites

- Understanding of SQL queries, predicates (`WHERE`, `JOIN` conditions), and indexes.
- Familiarity with `EXPLAIN` / execution plans and cardinality estimates.
- Awareness of database maintenance commands (`ANALYZE`, `VACUUM`, `UPDATE STATISTICS`).
- Knowledge of the target engine’s statistics catalog views.

### Related Programming Areas

- Query optimization and execution-plan analysis.
- Index design and validation.
- Database maintenance and autovacuum configuration.
- Data warehousing and ETL tuning.
- Cardinality estimation research and extended statistics.

---

## Core Concepts and Features

### 1. Table Statistics

#### Definitions

**Core Definition**
Table statistics are metadata values describing the overall size and shape of a table, such as the number of rows (tuples) and the number of pages (blocks) it occupies.

**Technical Definition**
In PostgreSQL, table-level statistics are stored in `pg_class` (`reltuples`, `relpages`) and updated by `ANALYZE` and `VACUUM`. `reltuples` is an estimate of live rows; `relpages` is the number of 8 KB pages. The optimizer uses these values to estimate the cost of sequential scans, index scans, and joins. MySQL’s `InnoDB` maintains table statistics in `mysql.innodb_table_stats` (row counts, clustered index size). SQL Server stores row counts and page counts in `sys.partitions` and statistics objects.

**Beginner-Friendly Explanation**
Table statistics tell the optimizer how big a table is—roughly how many rows and how many disk pages. A table with 1,000 rows is treated very differently from a table with 100 million rows.

#### Purposes

- To provide the optimizer with the base cardinality for cost estimation.
- To determine whether a sequential scan or an index scan is cheaper.
- To estimate the build-side size for hash joins and the outer-side size for nested loops.
- To support autovacuum/auto-analyze threshold calculations.

#### Syntax Rules and Structure

**PostgreSQL** (query table statistics):

```sql
SELECT relname, reltuples, relpages
FROM pg_class
WHERE relname = 'orders';
```

**MySQL** (query InnoDB table statistics):

```sql
SELECT table_name, num_rows, clustered_index_size
FROM mysql.innodb_table_stats
WHERE table_name = 'orders';
```

**SQL Server** (query row counts):

```sql
SELECT OBJECT_NAME(p.object_id) AS table_name,
       SUM(p.rows) AS row_count
FROM sys.partitions p
WHERE p.index_id IN (0, 1)
GROUP BY p.object_id;
```

**Component breakdown**

| Field | Meaning |
|-------|---------|
| `reltuples` | Estimated number of live rows (PostgreSQL). |
| `relpages` | Number of disk pages (PostgreSQL). |
| `num_rows` | Estimated row count (MySQL InnoDB). |
| `rows` | Row count per partition (SQL Server). |

**Syntax rules**

- Table statistics are updated by `ANALYZE` (PostgreSQL), `ANALYZE TABLE` (MySQL), or auto-statistics (SQL Server).
- `reltuples` can be negative in PostgreSQL for unanalyzed tables; run `ANALYZE` to populate.
- Row counts are estimates; they can drift between `ANALYZE` runs.

**Constraints and limitations**

- Table statistics alone are not enough for selectivity estimation; column statistics are required.
- Row-count estimates become stale after large data changes, leading to bad plans.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Inspecting table statistics

```sql
-- Step 1: Create a table and populate it.
CREATE TABLE orders (id serial PRIMARY KEY, status text, total numeric);
INSERT INTO orders (status, total)
SELECT CASE WHEN random() < 0.1 THEN 'pending' ELSE 'completed' END,
       random() * 100
FROM generate_series(1, 100000);

-- Step 2: Analyze the table.
ANALYZE orders;

-- Step 3: Query table statistics.
SELECT relname, reltuples, relpages
FROM pg_class
WHERE relname = 'orders';
```

**Expected output**:

```text
 relname | reltuples | relpages
---------+-----------+----------
 orders  |    100000 |      884
(1 row)
```

**Why this output appears**: `ANALYZE` sampled the table and updated `reltuples` to approximately 100,000 and `relpages` to the number of pages occupied. The optimizer uses these values as the base row count for cost estimation.

#### Real-World Cases

- **Bulk loads**: After loading millions of rows, table statistics are stale until `ANALYZE` runs, causing the optimizer to underestimate cardinality.
- **Autovacuum thresholds**: PostgreSQL’s auto-analyze triggers when the number of changes exceeds `autovacuum_analyze_threshold + autovacuum_analyze_scale_factor * n_live_tup`.

---

### 2. Column Statistics

#### Definitions

**Core Definition**
Column statistics describe the distribution of values within a single column, including null fraction, average width, number of distinct values, most common values, histogram bounds, and physical-logical correlation.

**Technical Definition**
PostgreSQL stores column statistics in `pg_statistic`, exposed through the human-readable `pg_stats` view. The key fields are `null_frac`, `avg_width`, `n_distinct`, `most_common_vals`, `most_common_freqs`, `histogram_bounds`, and `correlation`. MySQL stores column histograms in `column_statistics` as JSON, accessible via `INFORMATION_SCHEMA.COLUMN_STATISTICS`. SQL Server stores histograms and density vectors in statistics BLOBs, visible through `DBCC SHOW_STATISTICS`.

**Beginner-Friendly Explanation**
Column statistics tell the optimizer what the values in a column look like: how many are NULL, how many distinct values there are, which values appear most often, and how the values are spread out. These details let the optimizer estimate how many rows a `WHERE` clause will match.

#### Purposes

- To estimate the selectivity of equality predicates using most common values.
- To estimate range predicate selectivity using histograms.
- To estimate join cardinality using distinct-value counts.
- To decide whether an index scan will be efficient based on correlation.

#### Syntax Rules and Structure

**PostgreSQL** (query column statistics):

```sql
SELECT attname, null_frac, avg_width, n_distinct,
       most_common_vals, most_common_freqs,
       histogram_bounds, correlation
FROM pg_stats
WHERE tablename = 'orders' AND attname = 'status';
```

**MySQL** (query histogram):

```sql
SELECT COLUMN_NAME, HISTOGRAM
FROM INFORMATION_SCHEMA.COLUMN_STATISTICS
WHERE TABLE_NAME = 'orders' AND COLUMN_NAME = 'status';
```

**SQL Server** (show statistics):

```sql
DBCC SHOW_STATISTICS ('orders', 'stat_status');
```

**Component breakdown**

| Field | Meaning |
|-------|---------|
| `null_frac` | Fraction of NULL values. |
| `avg_width` | Average bytes per value. |
| `n_distinct` | Estimated distinct values. |
| `most_common_vals` / `most_common_freqs` | MCVs and their frequencies. |
| `histogram_bounds` | Bucket boundaries for non-MCV values. |
| `correlation` | Physical vs. logical ordering (-1 to 1). |

**Syntax rules**

- Column statistics are created by `ANALYZE` (PostgreSQL), `ANALYZE TABLE ... UPDATE HISTOGRAM` (MySQL), or auto-statistics (SQL Server).
- `n_distinct` can be negative in PostgreSQL, meaning the absolute value is the distinct count divided by the row count.
- `correlation` close to ±1 means the column is physically ordered; close to 0 means random ordering.

**Constraints and limitations**

- Statistics are sampled; rare values may be missed.
- Correlated columns are assumed independent by default; extended statistics are needed for multi-column correlation.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Inspecting column statistics

```sql
SELECT attname, null_frac, n_distinct,
       most_common_vals, most_common_freqs,
       correlation
FROM pg_stats
WHERE tablename = 'orders' AND attname = 'status';
```

**Expected output** (illustrative):

```text
 attname | null_frac | n_distinct | most_common_vals | most_common_freqs | correlation
---------+-----------+------------+------------------+-------------------+-------------
 status  |         0 |          2 | {completed,pending} | {0.9,0.1}      | 0.02
(1 row)
```

**Why this output appears**: The `status` column has two distinct values. 90% of rows are `completed` and 10% are `pending`, as shown by `most_common_freqs`. The low correlation (`0.02`) indicates the physical order is unrelated to the status value.

#### Real-World Cases

- **Skewed distributions**: A column where 80% of values are `'CLICK'` requires a higher statistics target or extended statistics to estimate other values accurately.
- **Join cardinality**: `n_distinct` on join keys determines the estimated size of hash joins and merge joins.

---

### 3. Histograms

#### Definitions

**Core Definition**
A histogram is a column statistic that summarizes the distribution of non-MCV values by dividing the value range into buckets and recording bucket boundaries.

**Technical Definition**
PostgreSQL histograms store `histogram_bounds` as an array of boundary values; each bucket contains approximately the same number of rows, and values within a bucket are assumed uniformly distributed. Most common values are excluded from the histogram to avoid double-counting. MySQL supports two histogram types: **singleton** (one value per bucket) and **equi-height** (range buckets with cumulative frequency). SQL Server histograms contain up to 200 steps, each with an upper bound, equal rows, range rows, and distinct range rows.

**Beginner-Friendly Explanation**
A histogram is a bar chart of value ranges. If a column has values from 1 to 1000, the histogram might divide that range into 10 buckets, each containing about 10% of the rows. The optimizer uses the bucket boundaries to estimate how many rows fall between two values.

#### Purposes

- To estimate the selectivity of range predicates (`BETWEEN`, `>`, `<`).
- To refine cardinality estimates when the data distribution is not uniform.
- To complement most common values for the non-MCV portion of the data.

#### Syntax Rules and Structure

**PostgreSQL** (histogram bounds):

```sql
SELECT histogram_bounds
FROM pg_stats
WHERE tablename = 'orders' AND attname = 'total';
```

**MySQL** (equi-height histogram JSON excerpt):

```json
{
  "buckets": [[1, 10, 0.25, 5], [10, 50, 0.75, 20], ...],
  "histogram-type": "equi-height",
  "number-of-buckets-specified": 128
}
```

**Component breakdown**

| Component | Meaning |
|-----------|---------|
| `histogram_bounds` | Boundary values between buckets (PostgreSQL). |
| Bucket lower/upper | Range covered by the bucket (MySQL). |
| Cumulative frequency | Fraction of rows ≤ upper bound (MySQL). |
| Equal rows | Rows exactly equal to the upper bound (SQL Server). |
| Range rows | Rows in the range excluding the upper bound (SQL Server). |

**Syntax rules**

- Histograms exclude most common values; the remaining values are divided into buckets.
- The number of buckets is controlled by the statistics target (`default_statistics_target` in PostgreSQL, `WITH N BUCKETS` in MySQL, up to 200 in SQL Server).
- Values within a bucket are assumed uniformly distributed.

**Constraints and limitations**

- Histograms cannot capture extreme skew if the skewed values are not MCVs.
- SQL Server histograms only cover the first column of a multi-column statistics object.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Using histogram bounds for range estimation

```sql
SELECT histogram_bounds
FROM pg_stats
WHERE tablename = 'city' AND attname = 'population';
```

**Expected output**:

```text
 {42,107800,137893,200453,380755,10500000}
```

**Why this output appears**: The histogram has six boundary values, dividing the population range into five buckets. Each bucket contains roughly 20% of the non-MCV rows. The optimizer uses these bounds to estimate how many cities fall between, for example, 107,800 and 137,893.

#### Real-World Cases

- **Range queries**: Estimating rows for `WHERE price BETWEEN 100 AND 200`.
- **Data skew**: Histograms help when data is not uniformly distributed but most common values do not capture the skew.

---

### 4. Most Common Values (MCV)

#### Definitions

**Core Definition**
Most common values (MCVs) are the top N most frequently occurring non-null values in a column, stored together with their frequencies to improve equality predicate selectivity estimates.

**Technical Definition**
PostgreSQL stores `most_common_vals` and `most_common_freqs` arrays in `pg_statistic`. The number of MCVs is controlled by the statistics target. When a query uses an equality predicate on a value that appears in the MCV list, the optimizer uses the stored frequency directly instead of assuming uniform distribution. SQL Server’s histogram implicitly captures frequent values as steps with high `equal_rows`. MySQL singletons and equi-height histograms capture frequent values in the bucket structure.

**Beginner-Friendly Explanation**
MCVs are the “popular” values. If 10% of orders have status `'pending'`, then `'pending'` is likely an MCV with frequency 0.1. When you query `WHERE status = 'pending'`, the optimizer knows to expect about 10% of the table, not 50%.

#### Purposes

- To provide accurate selectivity estimates for equality predicates on skewed data.
- To avoid the uniform-distribution assumption, which fails badly for skewed columns.
- To improve join cardinality estimates when the join key has a skewed distribution.

#### Syntax Rules and Structure

**PostgreSQL** (query MCVs):

```sql
SELECT most_common_vals, most_common_freqs
FROM pg_stats
WHERE tablename = 'city' AND attname = 'countrycode';
```

**Component breakdown**

| Field | Meaning |
|-------|---------|
| `most_common_vals` | Array of the N most frequent values. |
| `most_common_freqs` | Array of their frequencies (fractions of total rows). |

**Syntax rules**

- The number of MCVs is limited by the statistics target.
- MCVs are excluded from the histogram to avoid double-counting.
- If a column has no MCVs (e.g., all values are unique), the arrays are `NULL`.

**Constraints and limitations**

- MCVs are sampled; rare frequent values may be missed if the sample is small.
- For very high-cardinality columns, MCVs may not be useful; histograms or distinct-value estimates are better.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – MCV-driven selectivity estimate

```sql
SELECT most_common_vals, most_common_freqs
FROM pg_stats
WHERE tablename = 'city' AND attname = 'countrycode';
```

**Expected output**:

```text
 most_common_vals | most_common_freqs
------------------+-------------------------------------------
 {IND,CHN,USA,BRA,JPN} | {0.0953333,0.084,0.072,0.0706667,0.0613333}
```

**Why this output appears**: `IND` appears in 9.53% of rows, `CHN` in 8.4%, and so on. When the optimizer plans `WHERE countrycode = 'IND'`, it estimates approximately 9.53% of the table, rather than assuming 1/232 (uniform distribution).

#### Real-World Cases

- **Skewed status columns**: `'pending'` being 10% of orders while `'completed'` is 90%.
- **Country codes**: A few countries dominate the row count, and MCVs capture their frequencies.

---

### 5. Distinct Values (n_distinct)

#### Definitions

**Core Definition**
`n_distinct` is a statistic estimating the number of distinct non-null values in a column.

**Technical Definition**
In PostgreSQL, `n_distinct` is stored in `pg_stats`. A positive value is the absolute number of distinct values; a negative value means the distinct count is `abs(n_distinct) × row_count` (useful for columns that scale with table size, such as primary keys). MySQL stores distinct-value estimates in histograms and optimizer statistics. SQL Server stores density (`1 / distinct_values`) in statistics objects. `n_distinct` is critical for estimating join cardinality: the estimated number of join output rows is roughly `outer_rows × inner_rows / max(distinct_outer, distinct_inner)`.

**Beginner-Friendly Explanation**
`n_distinct` tells you how many different values a column has. A `status` column might have 5 distinct values; a `customer_id` column might have 100,000. The optimizer uses this to estimate how many rows will match a join or a `GROUP BY`.

#### Purposes

- To estimate join cardinality and `GROUP BY` group counts.
- To estimate `DISTINCT` query costs.
- To decide whether a hash aggregate or sort-based aggregate is cheaper.

#### Syntax Rules and Structure

**PostgreSQL**:

```sql
SELECT attname, n_distinct
FROM pg_stats
WHERE tablename = 'orders' AND attname = 'customer_id';
```

**Component breakdown**

| Value | Meaning |
|-------|---------|
| Positive integer | Absolute distinct count. |
| Negative fraction | Distinct count = `abs(n_distinct) × row_count`. |
| `-1` | All values are distinct (e.g., primary key). |

**Syntax rules**

- `n_distinct` is an estimate from a sample; it may be inaccurate for very large tables.
- It can be set manually with `ALTER TABLE ... ALTER COLUMN ... SET (n_distinct = ...)` if the estimate is known to be wrong.

**Constraints and limitations**

- Sampling error can cause significant underestimation for high-cardinality columns.
- Multi-column distinct counts are not captured by single-column `n_distinct`; extended statistics are needed.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Interpreting negative `n_distinct`

```sql
SELECT attname, n_distinct
FROM pg_stats
WHERE tablename = 'orders' AND attname IN ('id', 'customer_id', 'status');
```

**Expected output**:

```text
 attname     | n_distinct
-------------+------------
 id          |         -1
 customer_id |       -0.05
 status      |          2
```

**Why this output appears**: `id` is unique (`-1` means all rows distinct). `customer_id` has about 5% as many distinct values as rows (e.g., 5,000 distinct customers for 100,000 orders). `status` has 2 distinct values.

#### Real-World Cases

- **Join keys**: Primary key–foreign key joins rely on `n_distinct` to estimate output size.
- **`GROUP BY` cardinality**: High `n_distinct` means more groups and potentially more hash table memory.

---

### 6. Correlation

#### Definitions

**Core Definition**
Correlation measures how closely the physical order of rows in a table matches the logical order of a column’s values, on a scale from -1 to +1.

**Technical Definition**
PostgreSQL’s `correlation` statistic in `pg_stats` is calculated as the statistical correlation between the physical row position and the column value order. A value close to +1 means values increase with physical position; close to -1 means they decrease; close to 0 means no correlation. The optimizer uses correlation to decide whether an index scan will cause random I/O or sequential I/O: high absolute correlation means index-ordered heap access is likely to hit pages in physical order, making index scans cheaper.

**Beginner-Friendly Explanation**
Correlation tells you whether the table is stored in the same order as the column’s values. If a table is physically sorted by `created_at`, then correlation is near 1, and an index on `created_at` is very efficient because reading the index also reads the table in order. If the table is randomly ordered, correlation is near 0, and index scans may jump around the disk.

#### Purposes

- To decide whether an index scan or a sequential scan is cheaper.
- To estimate the random I/O cost of index-ordered heap access.
- To inform index design decisions (e.g., clustering a table on a frequently range-filtered column).

#### Syntax Rules and Structure

**PostgreSQL**:

```sql
SELECT attname, correlation
FROM pg_stats
WHERE tablename = 'orders' AND attname = 'created_at';
```

**Component breakdown**

| Value | Interpretation |
|-------|----------------|
| +1 | Perfectly ordered ascending. |
| -1 | Perfectly ordered descending. |
| 0 | Random physical order relative to values. |

**Syntax rules**

- Correlation is computed on the sampled rows used for statistics.
- It is specific to the physical storage at the time of `ANALYZE`; table rewrites or clustering change it.

**Constraints and limitations**

- Correlation does not capture partial ordering or correlation with multiple columns.
- It is engine-specific; MySQL and SQL Server do not expose an identical statistic.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Correlation and index scan choice

```sql
-- Table physically ordered by id.
SELECT attname, correlation
FROM pg_stats
WHERE tablename = 'orders' AND attname = 'id';
```

**Expected output**:

```text
 attname | correlation
---------+-------------
 id      |           1
```

**Why this output appears**: The `id` column is a serial primary key and rows are inserted in increasing order, so physical order matches logical order. The optimizer knows that an index scan on `id` will read the heap in sequential order, making it efficient.

#### Real-World Cases

- **Time-series tables**: Tables appended in `created_at` order have high correlation on `created_at`; range scans on that column are efficient.
- **Random inserts**: UUID primary keys have low correlation with physical order, making index scans more expensive.

---

### 7. Statistics Targets

#### Definitions

**Core Definition**
A statistics target is a per-column or global setting that controls how many most common values and histogram buckets the database collects when analyzing a column.

**Technical Definition**
In PostgreSQL, `default_statistics_target` (default 100) sets the global target; `ALTER TABLE ... ALTER COLUMN ... SET STATISTICS n` overrides it per column. The target determines the maximum number of MCVs and histogram buckets stored in `pg_statistic`. Higher targets improve estimate accuracy for skewed or high-cardinality columns, at the cost of longer `ANALYZE` runs and larger catalogs. MySQL controls histogram bucket count with `WITH N BUCKETS` (1–1024) in `ANALYZE TABLE ... UPDATE HISTOGRAM`. SQL Server histograms are capped at 200 steps; the sampling rate can be controlled with `FULLSCAN` or `SAMPLE`.

**Beginner-Friendly Explanation**
The statistics target is how much detail the database collects about a column. A target of 100 means it tracks about 100 “most common values” and 100 histogram buckets. If a column has a complex distribution, increasing the target gives the optimizer a more accurate picture—but it takes longer to analyze.

#### Purposes

- To improve estimate accuracy for columns with skewed or high-cardinality distributions.
- To balance `ANALYZE` cost against plan quality.
- To selectively increase detail on join keys and frequently filtered columns.

#### Syntax Rules and Structure

**PostgreSQL** (per-column target):

```sql
ALTER TABLE orders ALTER COLUMN status SET STATISTICS 500;
ANALYZE orders;
```

**MySQL** (bucket count):

```sql
ANALYZE TABLE orders UPDATE HISTOGRAM ON status WITH 256 BUCKETS;
```

**Component breakdown**

| Engine | Control | Range |
|--------|---------|-------|
| PostgreSQL | `default_statistics_target` / `SET STATISTICS` | 0–10000 |
| MySQL | `WITH N BUCKETS` | 1–1024 |
| SQL Server | Sampling rate / `FULLSCAN` | Up to 200 steps |

**Syntax rules**

- PostgreSQL: `SET STATISTICS` requires table ownership; the value 0 disables statistics collection for that column.
- Higher targets increase `ANALYZE` time and catalog size.
- SQL Server’s histogram step count is fixed at a maximum of 200; the sampling rate is the main tunable.

**Constraints and limitations**

- Increasing the target does not fix fundamentally unestimatable patterns (e.g., extreme skew beyond the MCV list).
- Extended statistics are needed for multi-column correlation.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Increasing the target for a skewed column

```sql
-- Step 1: Increase the statistics target.
ALTER TABLE orders ALTER COLUMN status SET STATISTICS 500;

-- Step 2: Re-analyze.
ANALYZE orders;

-- Step 3: Verify the target.
SELECT attname, attstattarget
FROM pg_attribute
WHERE attrelid = 'orders'::regclass AND attname = 'status';
```

**Expected output**:

```text
 attname | attstattarget
---------+---------------
 status  |           500
```

**Why this output appears**: The statistics target for `status` is now 500, allowing up to 500 MCVs and histogram buckets. The next `ANALYZE` will collect more detailed distribution information for that column.

#### Real-World Cases

- **Skewed event types**: Raising the target from 100 to 1000 reduced estimation error 4000× in a real-world case, enabling the optimizer to choose the correct hash join.
- **Join keys**: High-cardinality foreign keys benefit from targets of 500–1000.

---

### 8. `ANALYZE`

#### Definitions

**Core Definition**
`ANALYZE` is the command that collects statistics about table contents and stores them in the system catalog for use by the query optimizer.

**Technical Definition**
In PostgreSQL, `ANALYZE` samples table rows (or performs a full scan if `FULL` is specified) and updates `pg_statistic` and `pg_class`. It acquires a `SHARE UPDATE EXCLUSIVE` lock, allowing reads and writes to continue. It can be run per table or for the entire database. MySQL’s `ANALYZE TABLE` updates key distribution statistics and, with `UPDATE HISTOGRAM`, column histograms. SQL Server updates statistics automatically or via `UPDATE STATISTICS`.

**Beginner-Friendly Explanation**
`ANALYZE` is the “refresh the cheat sheet” command. After you load or change a lot of data, the old statistics become stale, and `ANALYZE` takes a fresh look at the data and updates the optimizer’s notes.

#### Purposes

- To refresh stale statistics after bulk data changes.
- To populate statistics for newly created tables or columns.
- To improve cardinality estimates and plan quality.
- To update the visibility map and related metadata (when combined with `VACUUM`).

#### Syntax Rules and Structure

**PostgreSQL**:

```sql
ANALYZE [ ( option [, ...] ) ] [ table_and_columns [, ...] ]

-- Examples:
ANALYZE orders;
ANALYZE orders (status, customer_id);
ANALYZE VERBOSE orders;
```

**MySQL**:

```sql
ANALYZE TABLE orders;
ANALYZE TABLE orders UPDATE HISTOGRAM ON status WITH 128 BUCKETS;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `ANALYZE` | Command keyword. |
| `table_and_columns` | Optional target table and column list. |
| `VERBOSE` | Reports progress (PostgreSQL). |
| `UPDATE HISTOGRAM` | Manages column histograms (MySQL). |

**Syntax rules**

- `ANALYZE` without a table name analyzes the entire database (PostgreSQL).
- `ANALYZE` uses sampling by default; `FULL` forces a full scan (PostgreSQL).
- It can be run inside a transaction but is often run outside for maintenance.
- `ANALYZE` is distinct from `VACUUM`; `VACUUM ANALYZE` combines both.

**Constraints and limitations**

- Sampling can miss rare values; increasing the statistics target or using `FULL` mitigates this.
- `ANALYZE` does not update statistics for foreign tables unless explicitly requested.
- On very large tables, `ANALYZE FULL` can be slow and I/O-intensive.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Analyzing a single table after a bulk load

```sql
-- Step 1: Bulk load.
INSERT INTO orders (status, total)
SELECT 'completed', random() * 100 FROM generate_series(1, 1000000);

-- Step 2: Analyze the table.
ANALYZE orders;

-- Step 3: Verify updated statistics.
SELECT reltuples FROM pg_class WHERE relname = 'orders';
```

**Expected output**:

```text
 reltuples
-----------
   1000000
(1 row)
```

**Why this output appears**: `ANALYZE` sampled the newly inserted rows and updated `reltuples` to approximately 1,000,000. Without this step, the optimizer would still think the table has its pre-load row count.

#### Real-World Cases

- **Post-ETL maintenance**: Running `ANALYZE` after nightly loads to prevent bad plans during business hours.
- **Autovacuum tuning**: Auto-analyze may not trigger quickly enough for large tables; manual `ANALYZE` is often required.

---

### 9. `VACUUM ANALYZE`

#### Definitions

**Core Definition**
`VACUUM ANALYZE` is a PostgreSQL command that combines vacuuming (reclaiming dead row space and updating the visibility map) with analyzing (collecting fresh optimizer statistics) in a single pass.

**Technical Definition**
`VACUUM` removes dead tuples left by `UPDATE` and `DELETE` operations and updates the visibility map, which enables index-only scans. `ANALYZE` collects statistics. `VACUUM ANALYZE` performs both operations, reducing the number of table scans compared to running them separately. It is commonly used in autovacuum and manual maintenance. In MySQL, `ANALYZE TABLE` updates statistics but does not reclaim space; `OPTIMIZE TABLE` performs a rebuild and statistics update. SQL Server’s `UPDATE STATISTICS` updates statistics only.

**Beginner-Friendly Explanation**
`VACUUM ANALYZE` is a two-for-one maintenance command: it cleans up dead rows and refreshes the statistics at the same time. It is the standard way to keep a PostgreSQL table healthy after heavy updates or deletes.

#### Purposes

- To reclaim disk space occupied by dead tuples.
- To update the visibility map for index-only scans.
- To refresh optimizer statistics in the same pass.
- To reduce the total I/O compared to separate `VACUUM` and `ANALYZE` runs.

#### Syntax Rules and Structure

**PostgreSQL**:

```sql
VACUUM ANALYZE [ ( option [, ...] ) ] [ table_and_columns [, ...] ]

-- Examples:
VACUUM ANALYZE orders;
VACUUM (ANALYZE, VERBOSE) orders;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `VACUUM` | Reclaims dead tuple space and updates the visibility map. |
| `ANALYZE` | Collects optimizer statistics. |
| `VERBOSE` | Reports progress. |

**Syntax rules**

- `VACUUM ANALYZE` cannot run inside a transaction block (PostgreSQL).
- It acquires a `SHARE UPDATE EXCLUSIVE` lock, allowing reads and writes.
- `VACUUM FULL` is a different, more invasive operation that rewrites the table.

**Constraints and limitations**

- `VACUUM ANALYZE` still uses sampling for statistics; `ANALYZE FULL` is not combined with `VACUUM` in the same command.
- It does not update statistics for columns that have `attstattarget = 0`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Combining maintenance operations

```sql
-- After many updates and deletes:
VACUUM ANALYZE orders;

-- Verify dead tuples are reduced and statistics are updated.
SELECT relname, n_live_tup, n_dead_tup, last_analyze
FROM pg_stat_user_tables
WHERE relname = 'orders';
```

**Expected output** (illustrative):

```text
 relname | n_live_tup | n_dead_tup |         last_analyze
---------+------------+------------+------------------------------
 orders  |     950000 |          0 | 2026-11-14 10:00:00.000000+00
```

**Why this output appears**: `VACUUM ANALYZE` removed dead tuples (`n_dead_tup = 0`) and refreshed statistics (`last_analyze` is current). The live tuple count reflects the current table size.

#### Real-World Cases

- **High-churn tables**: Tables with frequent updates and deletes accumulate dead tuples; `VACUUM ANALYZE` keeps them healthy.
- **Post-maintenance**: Running `VACUUM ANALYZE` after a large purge to reclaim space and refresh statistics.

---

### 10. Statistics Staleness

#### Definitions

**Core Definition**
Statistics staleness is the condition in which the stored optimizer statistics no longer reflect the current data distribution, typically because the table has changed since the last `ANALYZE`.

**Technical Definition**
Statistics become stale when rows are inserted, updated, or deleted after the last statistics collection. The optimizer continues to use the old statistics until `ANALYZE` (or auto-analyze) runs. Stale statistics cause cardinality estimation errors, which in turn cause suboptimal plan choices—nested loops where hash joins would be better, missing index usage, or insufficient memory grants. PostgreSQL’s auto-analyze triggers based on `autovacuum_analyze_threshold + autovacuum_analyze_scale_factor * n_live_tup`; for very large tables, the threshold can be so high that auto-analyze does not fire after a bulk load, requiring manual `ANALYZE`.

**Beginner-Friendly Explanation**
Statistics are like a map of the data. If the data changes but the map is not updated, the optimizer follows the old map and gets lost. Stale statistics are one of the most common causes of suddenly slow queries after a data load.

#### Purposes

- To recognize when plan regressions are caused by outdated statistics rather than query or schema changes.
- To decide when manual `ANALYZE` is required despite autovacuum.
- To design monitoring that alerts on statistics staleness.

#### Syntax Rules and Structure

**PostgreSQL** (identify stale statistics):

```sql
SELECT schemaname, tablename,
       last_analyze, last_autoanalyze,
       n_live_tup, n_mod_since_analyze
FROM pg_stat_user_tables
WHERE n_mod_since_analyze > 1000
ORDER BY n_mod_since_analyze DESC;
```

**Component breakdown**

| Field | Meaning |
|-------|---------|
| `last_analyze` | Last manual `ANALYZE` time. |
| `last_autoanalyze` | Last auto-analyze time. |
| `n_mod_since_analyze` | Rows modified since the last analyze. |

**Syntax rules**

- High `n_mod_since_analyze` relative to table size indicates staleness.
- `last_analyze` older than the last bulk load suggests stale statistics.
- Auto-analyze thresholds are configurable per table.

**Constraints and limitations**

- Staleness is not always visible from timestamps alone; a table can be modified frequently without triggering auto-analyze.
- Some changes (e.g., updates that do not change indexed columns) may not affect statistics much.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Detecting stale statistics after a bulk load

```sql
-- After loading 50 million rows into orders:
SELECT schemaname, tablename,
       last_analyze, last_autoanalyze,
       n_live_tup, n_mod_since_analyze
FROM pg_stat_user_tables
WHERE tablename = 'orders';
```

**Expected output** (illustrative):

```text
 schemaname | tablename | last_analyze | last_autoanalyze | n_live_tup | n_mod_since_analyze
------------+-----------+--------------+------------------+------------+---------------------
 public     | orders    | 2026-11-01   |                  |   50000000 |            50000000
```

**Why this output appears**: The table has 50 million live rows, but `last_analyze` is two weeks old and `n_mod_since_analyze` equals the entire table size. The optimizer’s statistics still describe the pre-load state (e.g., 1 million rows), causing severe underestimation.

#### Real-World Cases

- **Bulk ETL loads**: Auto-analyze may not trigger until 10% of a 50-million-row table changes; manual `ANALYZE` is needed after the load.
- **Plan regression after data growth**: A query that used a hash join suddenly switches to a nested loop because the optimizer still thinks the table is small.

---

### 11. Statistics Impact on Plans

#### Definitions

**Core Definition**
Statistics impact on plans describes how changes in statistics accuracy or staleness affect the optimizer’s choice of access methods, join algorithms, join order, and memory allocation.

**Technical Definition**
The optimizer uses statistics to estimate selectivity, cardinality, and cost. When statistics are accurate, the optimizer can choose between a nested loop with an index seek (good for small outer sets) and a hash join (good for large unsorted inputs). When statistics are stale or inaccurate, the optimizer may choose a nested loop for a large outer input (causing millions of index lookups) or a hash join with insufficient memory (causing spills). The most common symptoms are large discrepancies between `estimated rows` and `actual rows` in `EXPLAIN ANALYZE`, along with plan changes after `ANALYZE`.

**Beginner-Friendly Explanation**
Statistics are the inputs to the optimizer’s decision-making. Good statistics lead to good plans; bad statistics lead to bad plans. If you see a query suddenly using a nested loop and running for hours instead of seconds, stale statistics are a prime suspect.

#### Purposes

- To connect statistics quality to concrete plan choices and performance outcomes.
- To prioritize statistics maintenance for high-impact tables.
- To diagnose plan regressions by comparing plans before and after `ANALYZE`.

#### Syntax Rules and Structure

**Diagnosing statistics impact**:

```sql
EXPLAIN ANALYZE
SELECT ...
-- Compare "rows=estimated" with "rows=actual"
-- Large differences suggest statistics problems.
```

**Component breakdown**

| Symptom | Likely statistics cause | Fix |
|---------|-------------------------|-----|
| Nested loop with high `loops` on inner side | Underestimated outer cardinality | `ANALYZE` or increase target |
| Hash join with `Batches > 1` | Underestimated build side | `ANALYZE` or increase `work_mem` |
| Sort spilling to disk | Underestimated row count | `ANALYZE` or increase `work_mem` |
| Missing index usage | Overestimated selectivity | `ANALYZE` or check correlation |

**Syntax rules**

- Statistics affect all plan choices; no single rule covers every case.
- Extended statistics are needed for correlated columns; single-column statistics assume independence.

**Constraints and limitations**

- Some estimation errors are inherent (e.g., correlated predicates, expressions, functions) and cannot be fixed by `ANALYZE` alone.
- Plan guides or hints may be needed when statistics cannot capture the true distribution.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Plan changes after `ANALYZE`

```sql
-- Before ANALYZE (stale statistics):
EXPLAIN
SELECT * FROM orders WHERE customer_id = 42;
-- Plan: Seq Scan on orders (cost=0.00..1834.00 rows=10 width=72)

-- After ANALYZE:
ANALYZE orders;
EXPLAIN
SELECT * FROM orders WHERE customer_id = 42;
-- Plan: Index Scan using idx_orders_customer on orders (cost=0.43..8.45 rows=3 width=72)
```

**Expected output** (illustrative plans):

```text
-- Before:
 Seq Scan on orders  (cost=0.00..1834.00 rows=10 width=72)
   Filter: (customer_id = 42)

-- After:
 Index Scan using idx_orders_customer on orders
   (cost=0.43..8.45 rows=3 width=72)
   Index Cond: (customer_id = 42)
```

**Why this output appears**: Before `ANALYZE`, stale statistics led the optimizer to believe only 10 rows match, making a sequential scan look cheap. After `ANALYZE`, the optimizer knows the predicate is selective (3 rows) and switches to an index scan, dramatically reducing cost.

##### Example 2: SQL Server – Memory grant and spill from bad estimates

```sql
-- Actual execution plan shows:
-- Sort operator: Estimated Rows = 100, Actual Rows = 2,000,000
-- Warning: Sort spilled to tempdb (granted memory too small)
```

**Why this output appears**: SQL Server allocated memory based on an estimate of 100 rows, but the sort processed 2 million rows. The memory grant was insufficient, causing a spill to `tempdb`. Updating statistics or rewriting the query improves the estimate and the memory grant.

#### Real-World Cases

- **ETL-induced plan regression**: After a nightly load, queries switch from hash joins to nested loops until `ANALYZE` runs; automating `ANALYZE` after ETL fixes the issue.
- **Skewed distributions**: A column where one value dominates causes the optimizer to choose a plan that is optimal for the average but disastrous for the dominant value; extended statistics or plan guides help.

---

## References

- PostgreSQL Documentation – Statistics Used by the Planner – https://www.postgresql.org/docs/current/planner-statistics.html
- PostgreSQL Documentation – `pg_stats` View – https://www.postgresql.org/docs/current/view-pg-stats.html
- PostgreSQL Documentation – `ANALYZE` – https://www.postgresql.org/docs/current/sql-analyze.html
- PostgreSQL Documentation – `VACUUM` – https://www.postgresql.org/docs/current/sql-vacuum.html
- PostgreSQL Wiki – Planner Statistics – https://wiki.postgresql.org/wiki/Planner_Statistics
- EnterpriseDB – Statistics in PostgreSQL (Heikki Linnakangas) – https://wiki.postgresql.org/images/3/34/PGDay2010-Statistics.pdf
- MySQL 8.0 Reference Manual – Optimizer Statistics – https://dev.mysql.com/doc/refman/8.0/en/optimizer-statistics.html
- MySQL 8.0 Reference Manual – `ANALYZE TABLE` – https://dev.mysql.com/doc/refman/8.0/en/analyze-table.html
- Microsoft Learn – Statistics (SQL Server) – https://learn.microsoft.com/en-us/sql/relational-databases/statistics/statistics
- Microsoft Learn – `DBCC SHOW_STATISTICS` – https://learn.microsoft.com/en-us/sql/t-sql/database-console-commands/dbcc-show-statistics-transact-sql
- PostgreSQL Deep Dive – Statistics Catalog Lesson – https://github.com/Iamanshuaditya/postgresql-deep-dive/blob/main/lessons/module_09_catalogs/lesson_24_statistics.md
- PostgreSQL Documentation – `ALTER TABLE ... SET STATISTICS` – https://www.postgresql.org/docs/current/sql-altertable.html
- PostgreSQL Documentation – Extended Statistics – https://www.postgresql.org/docs/current/sql-createstatistics.html
- SQL Server – CREATE STATISTICS (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/create-statistics-transact-sql