# SQL Performance Debugging: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**: SQL performance debugging is the systematic process of identifying, diagnosing, and resolving queries and database configurations that cause excessive latency, resource consumption, or throughput degradation.

**Technical Definition**: Performance debugging encompasses query-level analysis (identifying slow queries, full table scans, missing indexes, poor cardinality estimates, expensive joins, excessive sorting, and inefficient pagination) and execution plan interpretation (reading EXPLAIN, EXPLAIN ANALYZE, and SHOW PROFILE outputs to understand the optimizer's chosen plan and the executor's actual runtime behavior). The goal is to align the optimizer's plan with the physical data distribution and access patterns through indexing, query rewriting, and statistics maintenance.

**Beginner-Friendly Explanation**: Performance debugging is like being a detective for slow queries. You look at clues (execution plans, wait statistics, row counts) to figure out why the database is taking so long, then fix the root cause—usually by adding an index, rewriting the query, or updating statistics.

### Key Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Plan-Dependent** | Performance is determined by the execution plan chosen by the optimizer |
| **Data-Dependent** | The same query may be fast on small data and slow on large data |
| **Cardinality-Sensitive** | Poor row estimates lead to bad plan choices |
| **Index-Driven** | Most performance improvements come from proper indexing |
| **Measurable** | EXPLAIN ANALYZE provides actual runtime metrics, not just estimates |

### Prerequisites

- **Query Access**: Ability to run EXPLAIN / EXPLAIN ANALYZE on the target queries
- **Schema Knowledge**: Understanding of table sizes, column distributions, and existing indexes
- **Statistics Access**: Ability to view and update optimizer statistics
- **Monitoring Tools**: `pg_stat_statements`, slow query log, Performance Schema, or equivalent
- **Baseline Metrics**: Known-good execution times for comparison

### Related Programming Areas

- **Index Design**: B-Tree, Hash, Composite, Partial, and Covering indexes
- **Query Optimization**: Rewriting queries to enable better plans
- **Statistics Maintenance**: Updating statistics to improve cardinality estimates
- **Schema Design**: Normalization, partitioning, and data type selection
- **Application Architecture**: Caching, pagination strategy, connection management

### Core Concepts Overview

SQL performance debugging comprises eight complementary categories:

1. **Slow Queries**: Identifying bottlenecks and tracking long-running queries
2. **Full Table Scans**: Spotting ALL or SEQ SCAN in execution plans
3. **Missing Indexes**: Detecting queries missing B-Tree, Hash, or Composite indexes
4. **Poor Cardinality Estimates**: Diagnosing stale statistics and wrong plan paths
5. **Expensive Joins**: Resolving nested loops and hash joins on unindexed columns
6. **Excessive Sorting**: Eliminating high overhead from in-memory or disk sorting
7. **Inefficient Pagination**: Fixing large OFFSET values that degrade performance
8. **Analyzing Execution Plans**: Interpreting EXPLAIN, EXPLAIN ANALYZE, and SHOW PROFILE

---

## Core Concept 1: Slow Queries

### Definitions

**Core Definition**: Slow queries are SQL statements whose execution time exceeds acceptable thresholds, consuming disproportionate CPU, I/O, or memory resources.

**Technical Definition**: Slow query identification involves capturing query execution metrics (total time, mean time, calls, rows) through monitoring tools like PostgreSQL's `pg_stat_statements`, MySQL's slow query log or Performance Schema, and SQL Server's Query Store. The focus is on queries with high total execution time (frequency × mean time) and high individual latency.

**Beginner-Friendly Explanation**: Slow queries are like slow cashiers at a supermarket. One slow transaction isn't a problem, but if that cashier is slow for thousands of customers, the whole store backs up. You find the slowest cashiers (queries) and figure out why they're slow.

### Purposes

- **To** identify the queries consuming the most total database time
- **To** distinguish between infrequent slow queries and frequent moderately slow queries
- **To** track query performance trends over time
- **To** prioritize optimization efforts based on total impact

### Syntax Rules and Structure

#### PostgreSQL: pg_stat_statements

```sql
-- Enable extension (requires shared_preload_libraries)
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Top queries by total execution time
SELECT 
    queryid,
    LEFT(query, 80) AS query_preview,
    calls,
    ROUND(total_exec_time::numeric, 2) AS total_ms,
    ROUND(mean_exec_time::numeric, 2) AS mean_ms,
    ROUND(100 * total_exec_time / SUM(total_exec_time) OVER (), 2) AS pct_total,
    rows
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

#### MySQL: Performance Schema

```sql
-- Top queries by total latency
SELECT 
    DIGEST_TEXT AS query_preview,
    COUNT_STAR AS calls,
    ROUND(SUM_TIMER_WAIT / 1000000000, 2) AS total_ms,
    ROUND(AVG_TIMER_WAIT / 1000000000, 2) AS mean_ms,
    SUM_ROWS_EXAMINED AS rows_examined
FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 10;
```

#### MySQL: Slow Query Log

```sql
-- Enable slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;  -- Log queries slower than 1 second
SET GLOBAL log_queries_not_using_indexes = 'ON';
```

#### Component Breakdown

| Metric | Description | Priority |
|--------|-------------|----------|
| `total_exec_time` / `SUM_TIMER_WAIT` | Cumulative time across all calls | High total = optimization target |
| `mean_exec_time` / `AVG_TIMER_WAIT` | Average time per call | High mean = slow individual query |
| `calls` / `COUNT_STAR` | Number of executions | High calls × high mean = critical |
| `rows` / `SUM_ROWS_EXAMINED` | Rows returned vs. examined | High rows examined = missing index |

#### Syntax Rules

- `pg_stat_statements` requires `shared_preload_libraries = 'pg_stat_statements'` and a restart.
- Statistics reset on server restart unless persisted.
- Slow query log thresholds should be tuned to the workload (1 second is a common default).
- `log_queries_not_using_indexes` can flood the log; use with caution.

#### Constraints and Limitations

- `pg_stat_statements` normalizes queries (replaces literals with `$1`), grouping similar queries.
- Slow query logs may miss queries that are fast individually but slow in aggregate.
- Monitoring overhead is minimal but non-zero.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Identifying Top Resource-Consuming Queries (PostgreSQL)

```sql
-- Step 1: Enable pg_stat_statements
-- postgresql.conf:
-- shared_preload_libraries = 'pg_stat_statements'
-- Restart PostgreSQL, then:
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Step 2: Find the top 5 queries by total execution time
SELECT 
    LEFT(query, 60) AS query_preview,
    calls,
    ROUND(total_exec_time::numeric / 1000, 2) AS total_seconds,
    ROUND(mean_exec_time::numeric, 2) AS mean_ms,
    ROUND(100 * total_exec_time / SUM(total_exec_time) OVER (), 2) AS pct
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 5;
```

**Expected Output**:
```
 query_preview                                    | calls  | total_seconds | mean_ms | pct  
--------------------------------------------------+--------+---------------+---------+------
 SELECT * FROM orders WHERE customer_id = $1      | 500000 |       12500.45 |   25.00 | 45.2
 SELECT * FROM products WHERE category = $1       | 200000 |        8000.12 |   40.00 | 28.9
 UPDATE inventory SET quantity = $1 WHERE id = $2 | 100000 |        3500.67 |   35.00 | 12.6
 SELECT COUNT(*) FROM events WHERE created_at > $1|   5000 |        2000.33 |  400.07 |  7.2
 SELECT * FROM users WHERE email = $1             | 300000 |        1500.89 |    5.00 |  5.4
```

**Why This Output Occurs**: The first query has the highest total time (45.2%) despite a low mean time (25ms) because it's called 500,000 times. The fourth query has a high mean time (400ms) but few calls. Both are optimization targets: the first needs an index to reduce mean time, the fourth needs query rewriting or indexing.

### Real-World Cases

**Case 1: E-Commerce Order Lookup**: A `SELECT * FROM orders WHERE customer_id = ?` query consumes 45% of total database time due to a missing index. Adding an index on `customer_id` reduces mean time from 25ms to 0.5ms.

**Case 2: Analytics Dashboard**: A dashboard query scans 10 million rows every 5 minutes. The mean time is 400ms, but total time is high due to frequent execution. Materializing the result or adding a covering index resolves the issue.

**Case 3: ORM N+1 Query**: An ORM generates 1,000 individual `SELECT` queries instead of one `JOIN`. `pg_stat_statements` shows 1,000 calls with 5ms each (5 seconds total). The fix is eager loading or batch fetching.

---

## Core Concept 2: Full Table Scans

### Definitions

**Core Definition**: A full table scan reads every row in a table to satisfy a query, bypassing indexes entirely.

**Technical Definition**: A sequential scan (SEQ SCAN in PostgreSQL, type=ALL in MySQL, Table Scan in SQL Server) occurs when the optimizer determines that reading the entire table is cheaper than using an index. This is often correct for small tables or queries returning a large percentage of rows, but it is a performance problem when the table is large and the query is selective.

**Beginner-Friendly Explanation**: A full table scan is like reading an entire book to find one sentence. If the book is small (100 pages), it's fine. If the book is 10,000 pages, you should use the index at the back to find the sentence directly.

### Purposes

- **To** detect full table scans in execution plans (SEQ SCAN, type=ALL, Table Scan)
- **To** distinguish between necessary scans (small tables, high selectivity) and unnecessary scans (missing index)
- **To** use EXPLAIN to verify whether an index is being used
- **To** quantify the cost of full scans (rows examined vs. rows returned)

### Syntax Rules and Structure

#### PostgreSQL: EXPLAIN with SEQ SCAN

```sql
EXPLAIN (ANALYZE, BUFFERS, COSTS OFF)
SELECT * FROM orders WHERE customer_id = 12345;
```

**Expected Output (SEQ SCAN)**:
```
 Seq Scan on orders  (actual time=0.500..2500.000 rows=1 loops=1)
   Filter: (customer_id = 12345)
   Rows Removed by Filter: 999999
   Buffers: shared hit=10000 read=5000
 Planning Time: 0.200 ms
 Execution Time: 2500.500 ms
```

#### MySQL: EXPLAIN with type=ALL

```sql
EXPLAIN SELECT * FROM orders WHERE customer_id = 12345;
```

**Expected Output (type=ALL)**:
```
+----+-------------+--------+------+---------------+------+---------+------+---------+-------------+
| id | select_type | table  | type | possible_keys | key  | key_len | ref  | rows    | Extra       |
+----+-------------+--------+------+---------------+------+---------+------+---------+-------------+
|  1 | SIMPLE      | orders | ALL  | NULL          | NULL | NULL    | NULL | 1000000 | Using where |
+----+-------------+--------+------+---------------+------+---------+------+---------+-------------+
```

#### Component Breakdown

| Indicator | Meaning | Action |
|-----------|---------|--------|
| `Seq Scan` (PostgreSQL) | Full table scan | Add index if selective |
| `type=ALL` (MySQL) | Full table scan | Add index if selective |
| `Rows Removed by Filter` | Rows read but not returned | High = missing index |
| `rows` (MySQL) | Estimated rows examined | High = missing index |
| `possible_keys = NULL` | No usable index | Create an index |

#### Syntax Rules

- `EXPLAIN ANALYZE` actually executes the query and shows real rows and timing.
- `EXPLAIN` (without ANALYZE) shows estimates only.
- `BUFFERS` in PostgreSQL shows shared hits vs. disk reads.
- MySQL's `EXPLAIN ANALYZE` (8.0.18+) shows actual execution metrics.

#### Constraints and Limitations

- Full table scans are sometimes optimal (small tables, low-selectivity queries).
- The optimizer may choose a scan even when an index exists if it estimates a scan is cheaper.
- Index-only scans are even faster than index scans; consider covering indexes.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Detecting and Fixing a Full Table Scan

```sql
-- Step 1: Create a table with 1 million rows and no index on customer_id
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total NUMERIC(10,2)
);
INSERT INTO orders (customer_id, order_date, total)
SELECT (random() * 10000)::int, '2025-01-01'::date + (random() * 365)::int,
       random() * 1000
FROM generate_series(1, 1000000);

-- Step 2: Run EXPLAIN ANALYZE on a selective query
EXPLAIN (ANALYZE, BUFFERS, COSTS OFF)
SELECT * FROM orders WHERE customer_id = 12345;
```

**Expected Output (Before Index)**:
```
 Seq Scan on orders  (actual time=0.500..2500.000 rows=100 loops=1)
   Filter: (customer_id = 12345)
   Rows Removed by Filter: 999900
   Buffers: shared hit=10000 read=5000
 Execution Time: 2500.500 ms
```

```sql
-- Step 3: Create an index
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

-- Step 4: Re-run EXPLAIN ANALYZE
EXPLAIN (ANALYZE, BUFFERS, COSTS OFF)
SELECT * FROM orders WHERE customer_id = 12345;
```

**Expected Output (After Index)**:
```
 Index Scan using idx_orders_customer_id on orders
   (actual time=0.050..0.500 rows=100 loops=1)
   Index Cond: (customer_id = 12345)
   Buffers: shared hit=10 read=2
 Execution Time: 0.550 ms
```

**Why This Output Occurs**: Without the index, PostgreSQL reads all 1 million rows to find the 100 matching `customer_id = 12345`. With the index, it jumps directly to the matching rows, reducing execution time from 2,500ms to 0.55ms (4,500x faster).

#### Example 2: MySQL EXPLAIN Showing type=ALL

```sql
-- Step 1: Create table without index
CREATE TABLE products (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    product_code VARCHAR(50),
    category VARCHAR(50),
    price DECIMAL(10,2)
);
-- Insert 500,000 rows...

-- Step 2: EXPLAIN a selective query
EXPLAIN SELECT * FROM products WHERE product_code = 'ABC123';
```

**Expected Output**:
```
+----+-------------+----------+------+---------------+------+---------+------+--------+-------------+
| id | select_type | table    | type | possible_keys | key  | key_len | ref  | rows   | Extra       |
+----+-------------+----------+------+---------------+------+---------+------+--------+-------------+
|  1 | SIMPLE      | products | ALL  | NULL          | NULL | NULL    | NULL | 500000 | Using where |
+----+-------------+----------+------+---------------+------+---------+------+--------+-------------+
```

```sql
-- Step 3: Add an index
CREATE INDEX idx_products_code ON products(product_code);

-- Step 4: Re-run EXPLAIN
EXPLAIN SELECT * FROM products WHERE product_code = 'ABC123';
```

**Expected Output**:
```
+----+-------------+----------+------+------------------+------------------+---------+-------+------+-------------+
| id | select_type | table    | type | possible_keys    | key              | key_len | ref   | rows | Extra       |
+----+-------------+----------+------+------------------+------------------+---------+-------+------+-------------+
|  1 | SIMPLE      | products | ref  | idx_products_code| idx_products_code| 53      | const |    1 | Using index |
+----+-------------+----------+------+------------------+------------------+---------+-------+------+-------------+
```

**Why This Output Occurs**: `type=ALL` with `key=NULL` indicates a full table scan. After adding the index, `type=ref` and `key=idx_products_code` confirm index usage. `rows` drops from 500,000 to 1.

### Real-World Cases

**Case 1: Customer Lookup by Email**: A query `SELECT * FROM customers WHERE email = ?` performs a full table scan on 5 million rows. Adding an index on `email` reduces lookup time from 3 seconds to 5ms.

**Case 2: Date Range Report**: A monthly report `SELECT * FROM orders WHERE order_date BETWEEN ? AND ?` scans the entire table. A composite index on `(order_date, customer_id)` enables an index range scan.

**Case 3: Small Table Exception**: A lookup table with 50 rows always uses a sequential scan — this is optimal and requires no index.

---

## Core Concept 3: Missing Indexes

### Definitions

**Core Definition**: A missing index is an index that would significantly improve query performance but does not exist, forcing the database to scan more rows than necessary.

**Technical Definition**: Index types include B-Tree (default, supports equality and range), Hash (equality only), Composite (multi-column B-Tree), Partial (filtered index), and Covering (includes all columns needed by a query). Missing indexes are detected when EXPLAIN shows a sequential scan with a high `Rows Removed by Filter` (PostgreSQL) or high `rows` with `type=ALL` (MySQL) on a selective predicate.

**Beginner-Friendly Explanation**: An index is like the index at the back of a textbook. Without it, you'd read every page to find a topic. With it, you jump directly to the right page. A missing index means the database is reading every page.

### Purposes

- **To** detect queries that would benefit from an index
- **To** choose the right index type (B-Tree, Hash, Composite, Partial, Covering)
- **To** understand index column order and its impact on query performance
- **To** balance read performance against write overhead and storage

### Syntax Rules and Structure

#### B-Tree Index (Default)

```sql
-- PostgreSQL
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

-- MySQL
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
```

#### Composite Index

```sql
-- Composite index on (customer_id, order_date)
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- Supports: WHERE customer_id = ? AND order_date > ?
-- Supports: WHERE customer_id = ?
-- Does NOT support: WHERE order_date > ? (leading column not used)
```

#### Covering Index (INCLUDE)

```sql
-- PostgreSQL: INCLUDE adds non-key columns to the index
CREATE INDEX idx_orders_customer_covering 
ON orders(customer_id) INCLUDE (order_date, total);
-- Query can be satisfied entirely from the index (index-only scan)
```

#### Partial Index

```sql
-- PostgreSQL: index only active orders
CREATE INDEX idx_orders_active 
ON orders(customer_id) WHERE status = 'ACTIVE';
-- Smaller index, faster for queries on active orders
```

#### Hash Index

```sql
-- PostgreSQL: hash index for equality-only lookups
CREATE INDEX idx_orders_hash ON orders USING HASH (customer_id);
-- Only supports = comparisons, not ranges
```

#### Component Breakdown

| Index Type | Use Case | Supports Ranges? | Size |
|------------|----------|------------------|------|
| B-Tree | General purpose | Yes | Medium |
| Hash | Equality only | No | Small |
| Composite | Multi-column filters | Yes (leftmost prefix) | Large |
| Partial | Filtered subsets | Yes | Small |
| Covering | Index-only scans | Yes | Large |

#### Syntax Rules

- Composite index column order matters: the leftmost column must be used for the index to be effective.
- Covering indexes (INCLUDE) add columns to the leaf level without affecting the index key.
- Partial indexes reduce size and write overhead by indexing only relevant rows.
- Hash indexes are faster than B-Tree for equality but do not support range queries.

#### Constraints and Limitations

- Every index adds write overhead (INSERT, UPDATE, DELETE must maintain the index).
- Too many indexes slow down writes and consume storage.
- Indexes on low-cardinality columns (e.g., `status` with 3 values) are often ineffective.
- PostgreSQL's `INCLUDE` requires version 11+.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Composite Index for Multi-Column Filter

```sql
-- Step 1: Query with two filter columns
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM orders 
WHERE customer_id = 12345 AND order_date > '2025-06-01';
```

**Expected Output (Before Index)**:
```
 Seq Scan on orders  (actual time=0.500..2500.000 rows=50 loops=1)
   Filter: ((customer_id = 12345) AND (order_date > '2025-06-01'::date))
   Rows Removed by Filter: 999950
 Execution Time: 2500.500 ms
```

```sql
-- Step 2: Create composite index (customer_id first, then order_date)
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- Step 3: Re-run EXPLAIN
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM orders 
WHERE customer_id = 12345 AND order_date > '2025-06-01';
```

**Expected Output (After Index)**:
```
 Index Scan using idx_orders_customer_date on orders
   (actual time=0.050..0.200 rows=50 loops=1)
   Index Cond: ((customer_id = 12345) AND (order_date > '2025-06-01'::date))
 Execution Time: 0.250 ms
```

**Why This Output Occurs**: The composite index on `(customer_id, order_date)` allows the database to jump directly to the matching `customer_id` and then filter by `order_date` within the index. The leftmost column (`customer_id`) is used for the equality predicate, and the second column (`order_date`) is used for the range predicate.

#### Example 2: Covering Index for Index-Only Scan

```sql
-- Step 1: Query needs customer_id, order_date, total
EXPLAIN (ANALYZE, COSTS OFF)
SELECT customer_id, order_date, total 
FROM orders WHERE customer_id = 12345;
```

```sql
-- Step 2: Create covering index
CREATE INDEX idx_orders_covering 
ON orders(customer_id) INCLUDE (order_date, total);

-- Step 3: Re-run EXPLAIN
EXPLAIN (ANALYZE, COSTS OFF)
SELECT customer_id, order_date, total 
FROM orders WHERE customer_id = 12345;
```

**Expected Output**:
```
 Index Only Scan using idx_orders_covering on orders
   (actual time=0.030..0.100 rows=100 loops=1)
   Index Cond: (customer_id = 12345)
   Heap Fetches: 0
 Execution Time: 0.120 ms
```

**Why This Output Occurs**: The covering index includes all columns needed by the query (`customer_id`, `order_date`, `total`). The executor satisfies the query entirely from the index without accessing the heap (`Heap Fetches: 0`), making it faster than a regular index scan.

### Real-World Cases

**Case 1: E-Commerce Order History**: A query `WHERE customer_id = ? AND order_date > ?` uses a composite index on `(customer_id, order_date)`, reducing query time from 2 seconds to 5ms.

**Case 2: Product Search**: A query `WHERE category = ? AND price < ?` uses a composite index on `(category, price)`. A B-Tree index on `category` alone is insufficient because price filtering happens after the index lookup.

**Case 3: Active Orders Only**: A partial index `WHERE status = 'ACTIVE'` reduces index size by 90% and improves write performance while still accelerating active-order queries.

---

## Core Concept 4: Poor Cardinality Estimates

### Definitions

**Core Definition**: Poor cardinality estimates occur when the query optimizer's row-count estimates diverge significantly from actual row counts, leading it to choose a suboptimal execution plan.

**Technical Definition**: The optimizer uses statistics (histograms, distinct value counts, null fractions) to estimate the number of rows that will satisfy each predicate. When statistics are stale, incomplete, or unrepresentative, the optimizer may underestimate or overestimate cardinality, choosing a nested loop when a hash join would be faster, or vice versa. Common causes include stale statistics, skewed data distributions, correlated columns, and non-uniform data growth.

**Beginner-Friendly Explanation**: The optimizer is like a GPS that estimates traffic. If the traffic data is outdated, the GPS might route you through a jam. Poor cardinality estimates mean the optimizer's "traffic data" (statistics) is wrong, so it picks a slow route.

### Purposes

- **To** detect when estimated rows differ significantly from actual rows in EXPLAIN ANALYZE
- **To** identify stale statistics that cause wrong plan choices
- **To** recognize correlated columns that violate the optimizer's independence assumption
- **To** update statistics or create extended statistics to improve estimates

### Syntax Rules and Structure

#### PostgreSQL: EXPLAIN ANALYZE Showing Estimate Mismatch

```sql
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM orders WHERE customer_id = 12345 AND order_date > '2025-06-01';
```

**Expected Output (Poor Estimate)**:
```
 Index Scan using idx_orders_customer_date on orders
   (actual time=0.050..2500.000 rows=50 loops=1)
   Index Cond: ((customer_id = 12345) AND (order_date > '2025-06-01'::date))
   Rows Removed by Index Recheck: 999950
   Planning Time: 0.200 ms
   Execution Time: 2500.500 ms
```

**Note**: The optimizer estimated 50 rows but actually had to recheck 999,950 rows. This indicates a cardinality misestimate.

#### Updating Statistics

```sql
-- PostgreSQL: update statistics for a table
ANALYZE orders;

-- PostgreSQL: increase statistics target for a column
ALTER TABLE orders ALTER COLUMN customer_id SET STATISTICS 500;
ANALYZE orders;

-- MySQL: update statistics
ANALYZE TABLE orders;

-- SQL Server: update statistics
UPDATE STATISTICS orders WITH FULLSCAN;
```

#### Extended Statistics for Correlated Columns (PostgreSQL)

```sql
-- Create extended statistics for correlated columns
CREATE STATISTICS orders_customer_date_stats (dependencies)
ON customer_id, order_date FROM orders;

ANALYZE orders;
```

#### Component Breakdown

| Metric | Description | Good Value |
|--------|-------------|------------|
| `rows` (estimated) | Optimizer's row estimate | Close to actual |
| `actual rows` | Real rows returned | — |
| `Rows Removed by Filter` | Rows read but filtered out | Low |
| `n_distinct` | Distinct value count | Accurate |
| `most_common_vals` | Histogram of frequent values | Representative |

#### Syntax Rules

- `ANALYZE` updates statistics; run after significant data changes.
- `SET STATISTICS` increases histogram granularity for a column (default 100).
- Extended statistics capture correlations between columns.
- MySQL's `ANALYZE TABLE` updates index statistics.

#### Constraints and Limitations

- Statistics are samples; they never perfectly represent the data.
- Correlated columns violate the optimizer's independence assumption.
- Skewed data (e.g., one customer with 90% of orders) is hard to estimate.
- Statistics maintenance adds overhead; balance frequency against freshness.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Cardinality Misestimate and Fix

```sql
-- Step 1: Check statistics freshness
SELECT 
    schemaname, 
    relname, 
    last_analyze, 
    last_autoanalyze,
    n_live_tup,
    n_dead_tup
FROM pg_stat_user_tables
WHERE relname = 'orders';
```

**Expected Output**:
```
 schemaname | relname | last_analyze | last_autoanalyze | n_live_tup | n_dead_tup 
------------+---------+--------------+------------------+------------+------------
 public     | orders  | 2025-01-01   | 2025-01-01       |    1000000 |      50000
```

```sql
-- Step 2: Run EXPLAIN ANALYZE to see estimate vs. actual
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM orders WHERE customer_id = 12345 AND order_date > '2025-06-01';
```

**Expected Output (Before ANALYZE)**:
```
 Index Scan using idx_orders_customer_date on orders
   (actual time=0.050..2500.000 rows=50 loops=1)
   Index Cond: ((customer_id = 12345) AND (order_date > '2025-06-01'::date))
   Rows Removed by Index Recheck: 999950
 Execution Time: 2500.500 ms
```

```sql
-- Step 3: Update statistics
ANALYZE orders;

-- Step 4: Re-run EXPLAIN ANALYZE
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM orders WHERE customer_id = 12345 AND order_date > '2025-06-01';
```

**Expected Output (After ANALYZE)**:
```
 Index Scan using idx_orders_customer_date on orders
   (actual time=0.050..0.500 rows=50 loops=1)
   Index Cond: ((customer_id = 12345) AND (order_date > '2025-06-01'::date))
 Execution Time: 0.550 ms
```

**Why This Output Occurs**: Before `ANALYZE`, the optimizer had stale statistics and overestimated the number of rows, causing it to recheck many rows. After `ANALYZE`, the statistics reflect the actual data distribution, and the optimizer chooses a better plan.

#### Example 2: Extended Statistics for Correlated Columns

```sql
-- Step 1: Create extended statistics
CREATE STATISTICS orders_customer_date_stats (dependencies)
ON customer_id, order_date FROM orders;

-- Step 2: Analyze to collect extended statistics
ANALYZE orders;

-- Step 3: Check the extended statistics
SELECT 
    stxname,
    stxkeys,
    stxddependencies
FROM pg_statistic_ext
WHERE stxname = 'orders_customer_date_stats';
```

**Expected Output**:
```
 stxname                       | stxkeys | stxddependencies 
-------------------------------+---------+------------------
 orders_customer_date_stats    | {1,2}   | {"1 => 2": 0.95}
```

**Why This Output Occurs**: The extended statistics capture the functional dependency between `customer_id` and `order_date` (95% of the time, knowing `customer_id` determines `order_date`). This allows the optimizer to make better estimates when both columns are used in the same query.

### Real-World Cases

**Case 1: Skewed Customer Data**: One customer has 90% of all orders. The optimizer assumes uniformity and underestimates the rows for that customer. Increasing the statistics target and running `ANALYZE` improves the estimate.

**Case 2: Correlated Columns**: `city` and `postal_code` are highly correlated. The optimizer assumes independence and underestimates rows when both are in the WHERE clause. Extended statistics resolve the issue.

**Case 3: Stale Statistics After Bulk Load**: A nightly ETL job loads 1 million rows. Statistics become stale, causing poor plans the next morning. Running `ANALYZE` after the ETL job fixes the estimates.

---

## Core Concept 5: Expensive Joins

### Definitions

**Core Definition**: Expensive joins are join operations that consume disproportionate CPU, memory, or I/O due to large row counts, missing indexes, or suboptimal join strategies.

**Technical Definition**: Join algorithms include nested loop (efficient for small outer sets with indexed inner), hash join (efficient for large equijoins), and merge join (efficient for pre-sorted inputs). Expensive joins occur when the optimizer chooses a nested loop without an index (forcing a full scan per outer row), a hash join with insufficient memory (spilling to disk), or a merge join with expensive sorts.

**Beginner-Friendly Explanation**: A nested loop join is like finding a book by checking every shelf for each request. A hash join is like building a catalog first, then looking up books quickly. Expensive joins happen when the database picks the wrong strategy or lacks the index to make the right strategy efficient.

### Purposes

- **To** identify whether the optimizer chose nested loop, hash join, or merge join
- **To** detect nested loops without indexes (the most common expensive join)
- **To** recognize hash joins spilling to disk due to insufficient `work_mem`
- **To** optimize joins through indexing, query rewriting, or `work_mem` tuning

### Syntax Rules and Structure

#### PostgreSQL: EXPLAIN ANALYZE Showing Nested Loop

```sql
EXPLAIN (ANALYZE, BUFFERS, COSTS OFF)
SELECT o.order_id, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date > '2025-06-01';
```

**Expected Output (Expensive Nested Loop)**:
```
 Nested Loop  (actual time=0.500..5000.000 rows=10000 loops=1)
   ->  Seq Scan on orders o  (actual time=0.200..500.000 rows=10000 loops=1)
         Filter: (order_date > '2025-06-01'::date)
   ->  Seq Scan on customers c  (actual time=0.300..0.400 rows=1 loops=10000)
         Filter: (customer_id = o.customer_id)
         Rows Removed by Filter: 999
 Execution Time: 5000.500 ms
```

**Problem**: The inner `Seq Scan on customers` executes 10,000 times (once per outer row). Each execution scans 1,000 rows to find 1 match.

#### Fix: Add Index on Join Column

```sql
CREATE INDEX idx_customers_customer_id ON customers(customer_id);
```

**Expected Output (After Index)**:
```
 Nested Loop  (actual time=0.500..50.000 rows=10000 loops=1)
   ->  Seq Scan on orders o  (actual time=0.200..500.000 rows=10000 loops=1)
         Filter: (order_date > '2025-06-01'::date)
   ->  Index Scan using idx_customers_customer_id on customers c
         (actual time=0.005..0.005 rows=1 loops=10000)
         Index Cond: (customer_id = o.customer_id)
 Execution Time: 50.500 ms
```

**Why This Output Occurs**: Without the index, each of the 10,000 outer rows triggers a full scan of `customers` (10,000 × 1,000 = 10,000,000 rows read). With the index, each lookup is O(log n), reducing execution time from 5,000ms to 50ms (100x faster).

#### Hash Join with Insufficient Memory

```sql
EXPLAIN (ANALYZE, BUFFERS, COSTS OFF)
SELECT o.order_id, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;
```

**Expected Output (Hash Join Spilling to Disk)**:
```
 Hash Join  (actual time=500.000..8000.000 rows=1000000 loops=1)
   Hash Cond: (o.customer_id = c.customer_id)
   ->  Seq Scan on orders o  (actual time=0.200..2000.000 rows=1000000 loops=1)
   ->  Hash  (actual time=1000.000..1000.000 rows=1000000 loops=1)
         Buckets: 131072  Batches: 32  Memory Usage: 4096 kB
         -- Batches > 1 indicates spilling to disk
 Execution Time: 8000.500 ms
```

**Fix: Increase work_mem**

```sql
SET work_mem = '256MB';
-- Re-run query; Batches should drop to 1
```

#### Component Breakdown

| Join Type | Best For | Index Requirement |
|-----------|----------|-------------------|
| Nested Loop | Small outer, indexed inner | Index on inner join column |
| Hash Join | Large equijoins | None, but needs memory |
| Merge Join | Pre-sorted inputs | Indexes on both join columns |

#### Syntax Rules

- Nested loops are efficient when the inner side has an index and the outer side is small.
- Hash joins are efficient for large equijoins but require sufficient `work_mem` to avoid disk spills.
- Merge joins require both inputs sorted; indexes can provide pre-sorted data.
- The optimizer chooses the join type based on table sizes, indexes, and statistics.

#### Constraints and Limitations

- `work_mem` is per-operation, not per-query; a query with multiple hash joins may use multiple allocations.
- Nested loops with large outer sets are almost always slow without indexes.
- Hash joins cannot be used for non-equality joins (e.g., `<`, `>`).

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Nested Loop Without Index

```sql
-- Step 1: Create tables without index on join column
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100)
);
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,  -- No index!
    order_date DATE,
    total NUMERIC(10,2)
);
-- Insert 1,000 customers, 1,000,000 orders

-- Step 2: EXPLAIN ANALYZE the join
EXPLAIN (ANALYZE, BUFFERS, COSTS OFF)
SELECT o.order_id, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date > '2025-06-01';
```

**Expected Output (Before Index)**:
```
 Nested Loop  (actual time=0.500..5000.000 rows=10000 loops=1)
   ->  Seq Scan on orders o  (actual time=0.200..500.000 rows=10000 loops=1)
         Filter: (order_date > '2025-06-01'::date)
   ->  Seq Scan on customers c  (actual time=0.300..0.400 rows=1 loops=10000)
         Filter: (customer_id = o.customer_id)
         Rows Removed by Filter: 999
 Execution Time: 5000.500 ms
```

```sql
-- Step 3: Create index on join column
CREATE INDEX idx_customers_customer_id ON customers(customer_id);

-- Step 4: Re-run EXPLAIN ANALYZE
EXPLAIN (ANALYZE, BUFFERS, COSTS OFF)
SELECT o.order_id, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date > '2025-06-01';
```

**Expected Output (After Index)**:
```
 Nested Loop  (actual time=0.500..50.000 rows=10000 loops=1)
   ->  Seq Scan on orders o  (actual time=0.200..500.000 rows=10000 loops=1)
         Filter: (order_date > '2025-06-01'::date)
   ->  Index Scan using idx_customers_customer_id on customers c
         (actual time=0.005..0.005 rows=1 loops=10000)
         Index Cond: (customer_id = o.customer_id)
 Execution Time: 50.500 ms
```

**Why This Output Occurs**: The inner side of the nested loop executes once per outer row. Without an index, each execution scans the entire `customers` table. With the index, each execution is O(log n), reducing total execution time by 100x.

### Real-World Cases

**Case 1: E-Commerce Order-Customer Join**: A join between `orders` (1M rows) and `customers` (100K rows) uses a nested loop without an index on `customers.customer_id`, executing 1M full scans of `customers`. Adding the index reduces query time from 5 seconds to 50ms.

**Case 2: Analytics Hash Join Spill**: A hash join between two large tables spills to disk because `work_mem` is too low (4MB default). Increasing `work_mem` to 256MB eliminates the spill and reduces query time from 8 seconds to 2 seconds.

**Case 3: Merge Join with Sorted Inputs**: A query joins two tables on a date column, both indexed. The optimizer chooses a merge join, which is efficient because both inputs are already sorted by the index.

---

## Core Concept 6: Excessive Sorting

### Definitions

**Core Definition**: Excessive sorting occurs when a query performs large sort operations that consume significant memory or spill to disk, degrading performance.

**Technical Definition**: Sorting is required for `ORDER BY`, `GROUP BY`, `DISTINCT`, and merge joins. When the sort operation's data exceeds `work_mem` (PostgreSQL) or `sort_buffer_size` (MySQL), the database spills to temporary disk files, dramatically slowing execution. Sort operations are shown in EXPLAIN as `Sort` (PostgreSQL) or `Using filesort` (MySQL).

**Beginner-Friendly Explanation**: Sorting is like arranging a deck of cards. If the deck is small, you can do it on the table (memory). If the deck is huge, you need to spread cards on the floor (disk), which is much slower.

### Purposes

- **To** detect sort operations in execution plans (`Sort`, `Using filesort`)
- **To** determine whether sorting is in memory or on disk (`Sort Method: quicksort` vs. `external merge`)
- **To** eliminate unnecessary sorts through indexes or query rewriting
- **To** tune `work_mem` or `sort_buffer_size` to keep sorts in memory

### Syntax Rules and Structure

#### PostgreSQL: EXPLAIN ANALYZE Showing Sort

```sql
EXPLAIN (ANALYZE, BUFFERS, COSTS OFF)
SELECT * FROM orders ORDER BY order_date DESC LIMIT 100;
```

**Expected Output (In-Memory Sort)**:
```
 Limit  (actual time=500.000..500.100 rows=100 loops=1)
   ->  Sort  (actual time=500.000..500.050 rows=100 loops=1)
         Sort Key: order_date DESC
         Sort Method: top-N heapsort  Memory: 25kB
         ->  Seq Scan on orders  (actual time=0.200..400.000 rows=1000000 loops=1)
 Execution Time: 500.200 ms
```

**Expected Output (Disk Sort)**:
```
 Sort  (actual time=5000.000..8000.000 rows=1000000 loops=1)
   Sort Key: order_date DESC
   Sort Method: external merge  Disk: 50000kB
   ->  Seq Scan on orders  (actual time=0.200..2000.000 rows=1000000 loops=1)
 Execution Time: 8000.500 ms
```

#### MySQL: EXPLAIN Showing Using filesort

```sql
EXPLAIN SELECT * FROM orders ORDER BY order_date DESC LIMIT 100;
```

**Expected Output**:
```
+----+-------------+--------+------+---------------+------+---------+------+---------+----------------+
| id | select_type | table  | type | possible_keys | key  | key_len | ref  | rows    | Extra          |
+----+-------------+--------+------+---------------+------+---------+------+---------+----------------+
|  1 | SIMPLE      | orders | ALL  | NULL          | NULL | NULL    | NULL | 1000000 | Using filesort |
+----+-------------+--------+------+---------------+------+---------+------+---------+----------------+
```

#### Fix: Index on Sort Column

```sql
CREATE INDEX idx_orders_order_date ON orders(order_date);

-- Re-run EXPLAIN
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM orders ORDER BY order_date DESC LIMIT 100;
```

**Expected Output (After Index)**:
```
 Limit  (actual time=0.050..0.100 rows=100 loops=1)
   ->  Index Scan Backward using idx_orders_order_date on orders
         (actual time=0.040..0.080 rows=100 loops=1)
 Execution Time: 0.120 ms
```

**Why This Output Occurs**: With the index, the database reads the first 100 rows in sorted order without a sort operation. Without the index, it reads all 1M rows, sorts them, and returns the top 100.

#### Component Breakdown

| Sort Method | Location | Performance |
|-------------|----------|-------------|
| `quicksort` | Memory | Fast |
| `top-N heapsort` | Memory (for LIMIT) | Fast |
| `external merge` | Disk | Slow |
| `Using filesort` (MySQL) | Memory or disk | Depends on size |

#### Syntax Rules

- Indexes on sort columns can eliminate sorts entirely.
- `ORDER BY` with `LIMIT` can use a top-N heapsort, which is more memory-efficient.
- `work_mem` (PostgreSQL) and `sort_buffer_size` (MySQL) control sort memory.
- `GROUP BY` and `DISTINCT` also require sorting unless an index provides the order.

#### Constraints and Limitations

- Increasing `work_mem` affects all sort operations in the session.
- Indexes on sort columns add write overhead.
- `ORDER BY` on expressions (e.g., `ORDER BY LOWER(name)`) cannot use a regular index.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Eliminating a Disk Sort with an Index

```sql
-- Step 1: Query with ORDER BY on an unindexed column
EXPLAIN (ANALYZE, BUFFERS, COSTS OFF)
SELECT * FROM orders ORDER BY order_date DESC LIMIT 100;
```

**Expected Output (Before Index)**:
```
 Limit  (actual time=5000.000..5000.100 rows=100 loops=1)
   ->  Sort  (actual time=5000.000..5000.050 rows=100 loops=1)
         Sort Key: order_date DESC
         Sort Method: external merge  Disk: 50000kB
         ->  Seq Scan on orders  (actual time=0.200..2000.000 rows=1000000 loops=1)
 Execution Time: 5000.200 ms
```

```sql
-- Step 2: Create index on sort column
CREATE INDEX idx_orders_order_date ON orders(order_date);

-- Step 3: Re-run EXPLAIN
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM orders ORDER BY order_date DESC LIMIT 100;
```

**Expected Output (After Index)**:
```
 Limit  (actual time=0.050..0.100 rows=100 loops=1)
   ->  Index Scan Backward using idx_orders_order_date on orders
         (actual time=0.040..0.080 rows=100 loops=1)
 Execution Time: 0.120 ms
```

**Why This Output Occurs**: The index on `order_date` stores rows in sorted order. The `LIMIT 100` query reads only the first 100 rows from the index (backward scan for DESC) without sorting the entire table. Execution time drops from 5,000ms to 0.12ms.

### Real-World Cases

**Case 1: Top-N Product Report**: A `SELECT * FROM products ORDER BY price DESC LIMIT 10` query sorts 1M rows to return 10. Adding an index on `price` enables a top-N index scan, reducing query time from 5 seconds to 1ms.

**Case 2: Group By with Sort**: A `GROUP BY` query sorts 1M rows to compute aggregates. Adding an index on the GROUP BY column eliminates the sort, improving performance by 10x.

**Case 3: Disk Sort from Low work_mem**: A sort operation spills to disk because `work_mem` is 4MB. Increasing it to 64MB keeps the sort in memory, reducing query time from 8 seconds to 2 seconds.

---

## Core Concept 7: Inefficient Pagination

### Definitions

**Core Definition**: Inefficient pagination occurs when a query uses a large `OFFSET` value, causing the database to scan and discard many rows before returning the requested page.

**Technical Definition**: `LIMIT n OFFSET m` requires the database to generate and discard the first `m` rows before returning `n` rows. As `m` grows, performance degrades linearly because the database must scan `m + n` rows, even though only `n` are returned. The solution is keyset pagination (also called cursor-based pagination), which uses a `WHERE` clause based on the last seen value (e.g., `WHERE id > :last_id ORDER BY id LIMIT n`), allowing the database to jump directly to the next page.

**Beginner-Friendly Explanation**: Offset pagination is like reading a book from the beginning to find page 500 — you flip through 499 pages first. Keyset pagination is like using a bookmark — you start exactly where you left off.

### Purposes

- **To** recognize the linear performance degradation of large OFFSET values
- **To** replace OFFSET pagination with keyset (cursor) pagination
- **To** maintain stable pagination results when data changes between pages
- **To** optimize deep pagination for large datasets

### Syntax Rules and Structure

#### Inefficient OFFSET Pagination (Wrong)

```sql
-- WRONG: large OFFSET scans and discards 500,000 rows
SELECT * FROM orders
ORDER BY order_id
LIMIT 100 OFFSET 500000;
```

**Execution Plan**:
```
 Limit  (actual time=5000.000..5000.100 rows=100 loops=1)
   ->  Index Scan using orders_pkey on orders
         (actual time=0.050..4500.000 rows=500100 loops=1)
 Execution Time: 5000.200 ms
```

#### Efficient Keyset Pagination (Correct)

```sql
-- CORRECT: keyset pagination using the last seen ID
SELECT * FROM orders
WHERE order_id > 500000  -- Last ID from previous page
ORDER BY order_id
LIMIT 100;
```

**Execution Plan**:
```
 Limit  (actual time=0.050..0.100 rows=100 loops=1)
   ->  Index Scan using orders_pkey on orders
         (actual time=0.040..0.080 rows=100 loops=1)
         Index Cond: (order_id > 500000)
 Execution Time: 0.120 ms
```

#### Component Breakdown

| Pagination Method | Query Pattern | Performance at Deep Pages |
|-------------------|---------------|---------------------------|
| OFFSET | `LIMIT n OFFSET m` | O(m + n) — linear degradation |
| Keyset | `WHERE id > last_id LIMIT n` | O(n) — constant time |
| Cursor | `WHERE (date, id) > (:last_date, :last_id)` | O(n) — constant time |

#### Syntax Rules

- Keyset pagination requires a unique, sortable column (e.g., primary key, timestamp).
- The `WHERE` clause must use the same ordering as the `ORDER BY`.
- For multi-column sorts, use tuple comparison: `WHERE (date, id) > (:last_date, :last_id)`.
- Keyset pagination cannot jump to arbitrary pages (no "page 500" link).

#### Constraints and Limitations

- Keyset pagination does not support random page access (e.g., "go to page 500").
- Requires a stable, unique sort key; ties must be broken by a secondary key.
- Applications must track the last-seen value, not just the page number.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: OFFSET vs. Keyset Pagination

```sql
-- Step 1: Create a table with 1 million rows
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total NUMERIC(10,2)
);
INSERT INTO orders (customer_id, order_date, total)
SELECT (random() * 10000)::int, '2025-01-01'::date + (random() * 365)::int,
       random() * 1000
FROM generate_series(1, 1000000);

-- Step 2: WRONG — OFFSET pagination at page 5000
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM orders
ORDER BY order_id
LIMIT 100 OFFSET 500000;
```

**Expected Output (OFFSET)**:
```
 Limit  (actual time=5000.000..5000.100 rows=100 loops=1)
   ->  Index Scan using orders_pkey on orders
         (actual time=0.050..4500.000 rows=500100 loops=1)
 Execution Time: 5000.200 ms
```

```sql
-- Step 3: CORRECT — keyset pagination
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM orders
WHERE order_id > 500000
ORDER BY order_id
LIMIT 100;
```

**Expected Output (Keyset)**:
```
 Limit  (actual time=0.050..0.100 rows=100 loops=1)
   ->  Index Scan using orders_pkey on orders
         (actual time=0.040..0.080 rows=100 loops=1)
         Index Cond: (order_id > 500000)
 Execution Time: 0.120 ms
```

**Why This Output Occurs**: OFFSET 500000 forces the database to scan and discard 500,000 rows before returning 100. Keyset pagination uses the index to jump directly to `order_id > 500000`, reading only 100 rows. Execution time drops from 5,000ms to 0.12ms (40,000x faster).

#### Example 2: Keyset Pagination with Multiple Sort Columns

```sql
-- Keyset pagination with (order_date, order_id) as the sort key
SELECT * FROM orders
WHERE (order_date, order_id) > ('2025-06-01', 500000)
ORDER BY order_date, order_id
LIMIT 100;
```

**Why This Output Occurs**: Tuple comparison `(order_date, order_id) > ('2025-06-01', 500000)` correctly handles ties in `order_date` by falling back to `order_id`. This requires a composite index on `(order_date, order_id)`.

### Real-World Cases

**Case 1: Infinite Scroll API**: A social media API uses keyset pagination with `WHERE created_at < :last_created_at` to support infinite scroll without performance degradation at deep pages.

**Case 2: Admin Dashboard Page 1000**: An admin dashboard uses OFFSET pagination and becomes unusably slow at page 1000. Switching to keyset pagination with `WHERE id > :last_id` restores sub-100ms response times.

**Case 3: Batch Export**: A batch export process reads 100,000 rows in pages of 1,000. Using keyset pagination ensures each page takes constant time, completing the export in minutes instead of hours.

---

## Core Concept 8: Analyzing Execution Plans

### Definitions

**Core Definition**: Execution plan analysis is the process of interpreting the output of EXPLAIN, EXPLAIN ANALYZE, or SHOW PROFILE to understand how the database executes a query and identify performance bottlenecks.

**Technical Definition**: An execution plan is a tree of plan nodes (scan, join, sort, aggregate, limit) that describes the order and method of data access. EXPLAIN shows the optimizer's estimated plan; EXPLAIN ANALYZE executes the query and shows actual row counts, timing, and buffer usage. Key metrics include estimated vs. actual rows, startup vs. total cost, loops, and sort method.

**Beginner-Friendly Explanation**: An execution plan is like a recipe the database follows to answer your query. EXPLAIN shows the recipe the database plans to use; EXPLAIN ANALYZE shows the recipe it actually used and how long each step took. Reading the plan tells you which step is the bottleneck.

### Purposes

- **To** read execution plans from top to bottom (or bottom to top, depending on format)
- **To** identify the most expensive node (highest actual time, most rows, disk sort)
- **To** compare estimated vs. actual rows to detect cardinality misestimates
- **To** verify that indexes are being used and joins are efficient
- **To** use EXPLAIN options (ANALYZE, BUFFERS, VERBOSE, FORMAT JSON) for detailed analysis

### Syntax Rules and Structure

#### PostgreSQL: EXPLAIN Options

```sql
-- Basic estimated plan
EXPLAIN SELECT * FROM orders WHERE customer_id = 12345;

-- Actual execution with timing and buffers
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM orders WHERE customer_id = 12345;

-- JSON format for programmatic analysis
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) 
SELECT * FROM orders WHERE customer_id = 12345;

-- Verbose output with output columns
EXPLAIN (ANALYZE, VERBOSE, COSTS OFF) 
SELECT * FROM orders WHERE customer_id = 12345;
```

#### MySQL: EXPLAIN Formats

```sql
-- Traditional tabular format
EXPLAIN SELECT * FROM orders WHERE customer_id = 12345;

-- JSON format with cost estimates
EXPLAIN FORMAT=JSON SELECT * FROM orders WHERE customer_id = 12345;

-- Tree format (MySQL 8.0.16+)
EXPLAIN FORMAT=TREE SELECT * FROM orders WHERE customer_id = 12345;

-- Actual execution (MySQL 8.0.18+)
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 12345;
```

#### PostgreSQL: SHOW PROFILE Alternative

PostgreSQL does not have `SHOW PROFILE`; it uses `EXPLAIN ANALYZE` and `pg_stat_statements`. MySQL's `SHOW PROFILE` is deprecated in favor of `Performance Schema`.

```sql
-- MySQL: SHOW PROFILE (deprecated)
SET profiling = 1;
SELECT * FROM orders WHERE customer_id = 12345;
SHOW PROFILE FOR QUERY 1;
```

#### Component Breakdown (PostgreSQL EXPLAIN ANALYZE)

| Field | Meaning | Good Value |
|-------|---------|------------|
| `actual time=start..total` | Time to first row and total time (ms) | Low total |
| `rows=N` | Actual rows returned | Close to estimate |
| `loops=N` | Number of times node executed | 1 for most nodes |
| `Buffers: shared hit=X read=Y` | Memory hits vs. disk reads | High hit ratio |
| `Sort Method: quicksort` | In-memory sort | Fast |
| `Sort Method: external merge` | Disk sort | Slow |
| `Rows Removed by Filter` | Rows read but filtered | Low |

#### Syntax Rules

- `EXPLAIN` (without ANALYZE) does not execute the query; it shows estimates only.
- `EXPLAIN ANALYZE` executes the query and shows actual metrics.
- Read plans from the innermost (most indented) node outward; the innermost node is executed first.
- Compare `rows` (estimated) with `actual rows` to detect cardinality issues.
- `BUFFERS` shows shared buffer hits and disk reads; high `read` values indicate missing indexes.

#### Constraints and Limitations

- `EXPLAIN ANALYZE` executes the query, which may have side effects (e.g., `INSERT ... RETURNING`).
- Plan output can be large for complex queries; use `FORMAT JSON` for programmatic parsing.
- The optimizer may choose different plans on different runs due to statistics changes.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Reading a Complex Execution Plan

```sql
-- Query: find top 10 customers by total order value in 2025
EXPLAIN (ANALYZE, BUFFERS, COSTS OFF)
SELECT 
    c.customer_name,
    SUM(o.total) AS total_spent
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date >= '2025-01-01'
GROUP BY c.customer_name
ORDER BY total_spent DESC
LIMIT 10;
```

**Expected Output**:
```
 Limit  (actual time=500.000..500.100 rows=10 loops=1)
   ->  Sort  (actual time=500.000..500.050 rows=10 loops=1)
         Sort Key: (sum(o.total)) DESC
         Sort Method: top-N heapsort  Memory: 25kB
         ->  HashAggregate  (actual time=400.000..450.000 rows=1000 loops=1)
               Group Key: c.customer_name
               ->  Hash Join  (actual time=50.000..300.000 rows=100000 loops=1)
                     Hash Cond: (o.customer_id = c.customer_id)
                     ->  Seq Scan on orders o  (actual time=0.200..100.000 rows=100000 loops=1)
                           Filter: (order_date >= '2025-01-01'::date)
                           Rows Removed by Filter: 900000
                     ->  Hash  (actual time=10.000..10.000 rows=1000 loops=1)
                           Buckets: 1024  Batches: 1  Memory Usage: 50kB
                           ->  Seq Scan on customers c  (actual time=0.100..5.000 rows=1000 loops=1)
 Execution Time: 500.200 ms
```

**Analysis**:
1. **Limit** (top): Returns 10 rows; fast.
2. **Sort**: `top-N heapsort` in memory (25kB); fast.
3. **HashAggregate**: Groups 100,000 rows into 1,000 groups; moderate.
4. **Hash Join**: Joins 100,000 orders with 1,000 customers; the hash table is small (50kB, 1 batch).
5. **Seq Scan on orders**: Reads 1,000,000 rows, filters to 100,000 (900,000 removed). **This is the bottleneck.** An index on `order_date` would reduce the scan.

**Optimization**: Add an index on `orders(order_date)`:

```sql
CREATE INDEX idx_orders_order_date ON orders(order_date);
```

**Expected Output (After Index)**:
```
 Limit  (actual time=50.000..50.100 rows=10 loops=1)
   ->  Sort  (actual time=50.000..50.050 rows=10 loops=1)
         Sort Key: (sum(o.total)) DESC
         Sort Method: top-N heapsort  Memory: 25kB
         ->  HashAggregate  (actual time=40.000..45.000 rows=1000 loops=1)
               Group Key: c.customer_name
               ->  Hash Join  (actual time=10.000..30.000 rows=100000 loops=1)
                     Hash Cond: (o.customer_id = c.customer_id)
                     ->  Index Scan using idx_orders_order_date on orders o
                           (actual time=0.050..15.000 rows=100000 loops=1)
                           Index Cond: (order_date >= '2025-01-01'::date)
                     ->  Hash  (actual time=5.000..5.000 rows=1000 loops=1)
 Execution Time: 50.200 ms
```

**Why This Output Occurs**: The index on `order_date` allows the database to read only the 100,000 rows matching the date filter, instead of scanning all 1,000,000 rows. Execution time drops from 500ms to 50ms (10x faster).

#### Example 2: MySQL EXPLAIN FORMAT=TREE

```sql
EXPLAIN FORMAT=TREE
SELECT c.customer_name, COUNT(*) AS order_count
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date >= '2025-01-01'
GROUP BY c.customer_name;
```

**Expected Output**:
```
-> Table scan on c  (cost=100.00 rows=1000)
    -> Hash join (cost=500.00 rows=100000)
        -> Table scan on o  (cost=200.00 rows=100000)
        -> Hash
            -> Table scan on c  (cost=100.00 rows=1000)
```

**Why This Output Occurs**: The tree format shows the join order and access methods. `Table scan on o` indicates a full table scan on `orders`, which could be optimized with an index on `order_date`.

### Real-World Cases

**Case 1: Dashboard Query Optimization**: A dashboard query takes 5 seconds. EXPLAIN ANALYZE reveals a sequential scan on a 10M-row table with 9M rows removed by filter. Adding an index reduces execution to 50ms.

**Case 2: Hash Join Spill Diagnosis**: A join query takes 8 seconds. EXPLAIN ANALYZE shows `Batches: 32` in the Hash node, indicating disk spill. Increasing `work_mem` to 256MB reduces `Batches` to 1 and execution time to 2 seconds.

**Case 3: Cardinality Misestimate Detection**: EXPLAIN ANALYZE shows `rows=100` (estimated) vs. `actual rows=100000`. Running `ANALYZE` on the table updates statistics and improves the estimate, leading to a better plan.

---

## References

| Name | Link |
|------|------|
| PostgreSQL Documentation — Using EXPLAIN | https://www.postgresql.org/docs/current/using-explain.html |
| PostgreSQL Documentation — pg_stat_statements | https://www.postgresql.org/docs/current/pgstatstatements.html |
| PostgreSQL Documentation — Statistics Used by the Planner | https://www.postgresql.org/docs/current/planner-statistics.html |
| MySQL 8.0 Reference Manual — EXPLAIN Output Format | https://dev.mysql.com/doc/refman/8.0/en/explain-output.html |
| MySQL 8.0 Reference Manual — Performance Schema Statement Digests | https://dev.mysql.com/doc/refman/8.0/en/performance-schema-statement-digests.html |
| MySQL 8.0 Reference Manual — Slow Query Log | https://dev.mysql.com/doc/refman/8.0/en/slow-query-log.html |
| Microsoft Learn — Query Store | https://learn.microsoft.com/en-us/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store |
| Microsoft Learn — Execution Plans | https://learn.microsoft.com/en-us/sql/relational-databases/performance/execution-plans |
| Use The Index, Luke — SQL Indexing and Tuning | https://use-the-index-luke.com/ |
| Brent Ozar — How to Think Like the SQL Server Engine | https://www.brentozar.com/archive/2019/10/how-to-think-like-the-sql-server-engine/ |
| PostgreSQL Wiki — Performance Optimization | https://wiki.postgresql.org/wiki/Performance_Optimization |
| Percona — MySQL Query Optimization | https://www.percona.com/blog/ |
| AWS — Amazon RDS Performance Insights | https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PerfInsights.html |
| PostgreSQL — Index-Only Scans and Covering Indexes | https://www.postgresql.org/docs/current/indexes-index-only-scans.html |
| PostgreSQL — Extended Statistics | https://www.postgresql.org/docs/current/planner-statistics.html#PLANNER-STATS-EXTENDED |
| MySQL — Optimizing LIMIT and OFFSET | https://dev.mysql.com/doc/refman/8.0/en/limit-optimization.html |
| Markus Winand — Pagination Done the Right Way | https://use-the-index-luke.com/sql/partial-results/fetch-next-page |