# Comprehensive Programming Cheat Sheet: Query Performance Tuning

## Topic Overview

### Definitions

**Core Definition**: Query performance tuning is the systematic process of identifying, diagnosing, and resolving slow or resource-intensive SQL statements to improve application responsiveness and database throughput.

**Technical Definition**: Query performance tuning encompasses the observability mechanisms (slow query logs, statistics views, profiling tools), the diagnostic metrics (execution time, I/O, CPU, memory, lock waits), and the remediation controls (query cancellation, timeouts, plan optimization) used to ensure that SQL statements execute within acceptable latency and resource bounds. It spans multiple layers: the database engine's instrumentation (e.g., PostgreSQL `pg_stat_statements`, MySQL `performance_schema`, SQL Server Query Store, Oracle AWR), the operating system's resource counters, and the application's query patterns. Effective tuning requires correlating observed latency with resource consumption (I/O, CPU, memory), concurrency contention (locks, blocking), and plan quality.

**Beginner-Friendly Explanation**: Query performance tuning is detective work for databases. When a page on your application is slow, the database might be the culprit. Tuning is how you find the specific query that is causing the problem—whether it is reading too much data from disk, burning too much CPU, waiting for a lock, or simply taking too long—and then fix it so your users stop waiting.

### Key Characteristics

- **Observability-first**: You cannot tune what you cannot measure. Every major RDBMS provides slow query logs, real-time activity views, and historical statistics.
- **Multi-dimensional**: Slow queries can be caused by I/O bottlenecks, CPU saturation, memory pressure, lock contention, or poor execution plans. Tuning requires distinguishing among these causes.
- **Vendor-specific instrumentation**: PostgreSQL, MySQL, SQL Server, and Oracle each have distinct tools, catalog views, and configuration parameters for performance diagnosis.
- **Prevention over intervention**: Query timeouts and cancellation are safety nets, but the root cause (missing index, bad plan, lock contention) must be fixed to prevent recurrence.
- **Continuous monitoring**: Performance regressions occur after schema changes, data growth, or deployment updates; historical baselines are essential for detection.

### Prerequisites

- Understanding of SQL `SELECT`, `JOIN`, `WHERE`, and execution plans (`EXPLAIN`).
- Familiarity with indexes, statistics, and the query optimizer.
- Access to database administrative privileges (to read statistics views, set timeouts, cancel queries).
- Knowledge of the target engine's configuration files and runtime parameters.

### Related Programming Areas

- Database administration (DBA) and site reliability engineering (SRE).
- Application performance monitoring (APM) and observability.
- Query optimization and execution plan analysis.
- Capacity planning and resource management.
- Incident response and root cause analysis.

---

## Core Concepts and Features

### 1. Identifying Slow Queries

#### Definitions

**Core Definition**: Identifying slow queries is the process of locating SQL statements that consume disproportionate time or resources relative to the workload, using logs, statistics views, or profiling tools.

**Technical Definition**: Identification relies on three complementary sources: (1) **historical logs** (slow query logs, `log_min_duration_statement`), which capture statements exceeding a configured threshold; (2) **aggregate statistics views** (`pg_stat_statements`, `performance_schema.events_statements_summary_by_digest`, `sys.dm_exec_query_stats`, Oracle `V$SQL`), which rank statements by total or average elapsed time, CPU time, or I/O; and (3) **real-time activity views** (`pg_stat_activity`, `SHOW PROCESSLIST`, `sys.dm_exec_requests`, `V$SESSION`), which show currently running statements and their wait states.

**Beginner-Friendly Explanation**: Finding slow queries is like finding the slowest cars on a highway. You can look at past trip logs (slow query logs), average speeds by route (statistics views), or watch the traffic cameras right now (real-time activity views).

#### Purposes

- To locate the specific SQL statements responsible for latency or resource consumption.
- To prioritize tuning efforts by total impact (total time, total CPU) rather than isolated worst-case duration.
- To distinguish between queries that are slow because they are expensive and queries that are slow because they are blocked.
- To establish a baseline for detecting regressions after changes.

#### Syntax Rules and Structure

**PostgreSQL** (top queries by total time from `pg_stat_statements`):

```sql
SELECT query, calls, total_exec_time, mean_exec_time, rows
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

**MySQL** (top queries by total time from `performance_schema`):

```sql
SELECT digest_text, count_star,
       ROUND(SUM_TIMER_WAIT/1e12, 2) AS total_seconds
FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 10;
```

**SQL Server** (top queries by total CPU from `sys.dm_exec_query_stats`):

```sql
SELECT TOP 10
    qs.total_worker_time/1000 AS total_cpu_ms,
    qs.execution_count,
    SUBSTRING(st.text, (qs.statement_start_offset/2)+1,
        ((CASE qs.statement_end_offset
            WHEN -1 THEN DATALENGTH(st.text)
            ELSE qs.statement_end_offset END
            - qs.statement_start_offset)/2)+1) AS query_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) st
ORDER BY qs.total_worker_time DESC;
```

**Component breakdown**:

| Engine | Primary identification view | Key metric |
|--------|------------------------------|------------|
| PostgreSQL | `pg_stat_statements` | `total_exec_time`, `mean_exec_time` |
| MySQL | `events_statements_summary_by_digest` | `SUM_TIMER_WAIT` |
| SQL Server | `sys.dm_exec_query_stats` | `total_worker_time` (CPU) |
| Oracle | `V$SQLAREA` / `DBA_HIST_SQLSTAT` | `ELAPSED_TIME`, `CPU_TIME` |

**Syntax rules**:

- Aggregate statistics views group statements by normalized text (digest), so parameter differences are collapsed.
- Total time (`total_exec_time`) is often a better prioritization metric than mean time, because a query running thousands of times at 100 ms each can consume more total database time than a single 10-second report.
- Real-time views must be filtered to exclude background workers and idle sessions.

**Constraints and limitations**:

- Statistics views have retention limits (e.g., `pg_stat_statements.max`, SQL Server plan cache eviction).
- Normalized statements may group queries that have different optimal plans (e.g., different parameter values).
- Real-time views show only current activity; historical slow queries require logs or persistent statistics stores.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Ranking queries by total execution time

```sql
-- Step 1: Ensure pg_stat_statements is loaded.
-- shared_preload_libraries = 'pg_stat_statements'

-- Step 2: Query top 5 by total time.
SELECT queryid, query, calls,
       ROUND(total_exec_time::numeric, 2) AS total_ms,
       ROUND(mean_exec_time::numeric, 2) AS mean_ms
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 5;
```

**Expected output**:

```text
 queryid | query                                  | calls  | total_ms | mean_ms
---------+----------------------------------------+--------+----------+---------
  123456 | SELECT * FROM orders WHERE customer_id = $1 | 500000 | 250000.00 | 0.50
  789012 | UPDATE inventory SET qty = qty - $1      | 100000 | 180000.00 | 1.80
(2 rows)
```

**Why this output appears**: The first query has a low mean time (0.50 ms) but a high call count (500,000), making it the largest total consumer. The second query has a higher mean time but fewer calls. Prioritizing by total time reveals that the first query is the biggest overall bottleneck.

##### Example 2: SQL Server – Finding top CPU consumers

```sql
SELECT TOP 5
    qs.total_worker_time/1000 AS total_cpu_ms,
    qs.execution_count,
    qs.total_worker_time/qs.execution_count/1000 AS avg_cpu_ms,
    SUBSTRING(st.text, (qs.statement_start_offset/2)+1,
        ((CASE qs.statement_end_offset
            WHEN -1 THEN DATALENGTH(st.text)
            ELSE qs.statement_end_offset END
            - qs.statement_start_offset)/2)+1) AS query_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) st
ORDER BY qs.total_worker_time DESC;
```

**Expected output**:

```text
total_cpu_ms execution_count avg_cpu_ms query_text
------------ --------------- ---------- ------------------------------------------
450000       150000          3          SELECT * FROM Orders WHERE Status = @p0
320000       80000           4          UPDATE Inventory SET Qty = Qty - @p0
```

**Why this output appears**: The first query consumes 450 seconds of total CPU across 150,000 executions, averaging 3 ms per execution. The second query has a higher average CPU but lower total, indicating the first query is the larger overall CPU consumer.

#### Real-World Cases

- **ORM-generated queries**: Object-relational mappers often generate hundreds of similar statements; aggregating by digest reveals the dominant pattern.
- **Reporting vs. OLTP**: A long-running report may be slow per execution but run rarely; an OLTP query running thousands of times per second may consume more total resources despite being individually fast.
- **Post-deployment regression**: Comparing top-query rankings before and after a deployment identifies newly introduced bottlenecks.

---

### 2. Slow Query Logs

#### Definitions

**Core Definition**: A slow query log is a file or table that records SQL statements whose execution time exceeds a configured threshold, providing a historical record for after-the-fact analysis.

**Technical Definition**: Slow query logs are enabled by configuration parameters that specify the time threshold and output destination. In PostgreSQL, `log_min_duration_statement` (in milliseconds) logs statements exceeding the threshold to the server log; `log_statement` controls which statement types are logged. In MySQL, `slow_query_log = ON` and `long_query_time` (seconds) enable logging to a file or table, with `log_queries_not_using_indexes` optionally capturing index-less queries regardless of duration. SQL Server does not have a traditional slow query log but provides Query Store and Extended Events for similar purposes.

**Beginner-Friendly Explanation**: A slow query log is a diary of every query that took longer than a threshold you set. If a query takes more than, say, 1 second, it gets written to the diary. Later, you can read the diary to see which queries were slow and when they happened.

#### Purposes

- To capture slow statements for retrospective analysis when real-time monitoring is insufficient.
- To establish a historical record of query latency for trend analysis.
- To identify queries that exceed application SLA thresholds.
- To feed log analysis tools (`pgBadger`, `pt-query-digest`) that aggregate and rank slow queries.

#### Syntax Rules and Structure

**PostgreSQL** (configuration in `postgresql.conf`):

```ini
logging_collector = on
log_directory = 'log'
log_filename = 'postgresql-%a.log'
log_min_duration_statement = 1000   -- log queries > 1 second
log_line_prefix = '%m [%p] %q%u@%d '
```

**MySQL** (configuration in `my.cnf` or runtime):

```sql
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow.log';
SET GLOBAL long_query_time = 1;          -- 1 second
SET GLOBAL log_queries_not_using_indexes = 'ON';
```

**Component breakdown**:

| Engine | Parameter | Meaning |
|--------|-----------|---------|
| PostgreSQL | `log_min_duration_statement` | Milliseconds; -1 disables, 0 logs all |
| MySQL | `long_query_time` | Seconds; default 10 |
| MySQL | `log_queries_not_using_indexes` | Log index-less queries regardless of time |

**Syntax rules**:

- PostgreSQL: `log_min_duration_statement = 0` logs every statement (high overhead); use a threshold appropriate to the workload.
- MySQL: `long_query_time` can be fractional (e.g., `0.2`) for latency-sensitive applications.
- Logs can expose sensitive data; consider hashing or redacting parameters.
- Log rotation must be configured to prevent disk exhaustion.

**Constraints and limitations**:

- Logging every slow query at a low threshold can impact performance in high-volume environments.
- Logs capture statement text, which may include sensitive parameters.
- SQL Server lacks a direct equivalent; Query Store or Extended Events must be used.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Enabling and analyzing slow query logging

```sql
-- Step 1: Set the threshold (requires superuser or appropriate role).
ALTER SYSTEM SET log_min_duration_statement = 1000;  -- 1 second
SELECT pg_reload_conf();

-- Step 2: Find the log directory.
SHOW log_directory;

-- Step 3: After some time, use pgBadger to analyze.
-- $ pgbadger /var/log/postgresql/postgresql-*.log -o report.html
```

**Expected output**: A log entry like:

```text
2026-11-14 10:15:30.123 UTC [12345] user@db LOG:  duration: 1523.456 ms  statement: SELECT * FROM orders WHERE status = 'pending';
```

**Why this output appears**: The `duration` field records the actual execution time, and the `statement` field contains the query text. pgBadger parses these entries and produces a report ranking queries by frequency, duration, and time of day.

##### Example 2: MySQL – Runtime slow query log configuration

```sql
-- Step 1: Enable the slow query log dynamically (MySQL 8.0).
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 0.5;  -- 500 ms
SET GLOBAL log_queries_not_using_indexes = 'ON';

-- Step 2: Verify.
SHOW VARIABLES LIKE 'slow_query%';
SHOW VARIABLES LIKE 'long_query_time';

-- Step 3: Analyze with pt-query-digest.
-- $ pt-query-digest /var/log/mysql/slow.log
```

**Expected output** (pt-query-digest excerpt):

```text
# Rank Query ID           Response time Calls R/Call  V/M   Item
# ==== ================== ============= ===== ======= ===== ==============
#    1 0xABC123DEF4567890 120.5000 50.2% 5000  0.0241  0.01 SELECT orders
#    2 0xDEF4567890ABC123  80.3000 33.5% 2000  0.0401  0.02 UPDATE inventory
```

**Why this output appears**: `pt-query-digest` aggregates the slow log by normalized query fingerprint, showing total response time and call counts. The first query consumes 50.2% of total slow-query time across 5,000 calls.

#### Real-World Cases

- **Post-incident forensics**: After a performance incident, slow query logs show which statements were running slowly during the incident window.
- **Capacity planning**: Long-term slow query logs reveal growing latency trends as data volumes increase.
- **Application SLA monitoring**: Logs feed alerts when queries exceed defined thresholds.

---

### 3. Query Profiling

#### Definitions

**Core Definition**: Query profiling is the detailed measurement of an individual query's execution, including per-node timing, row counts, and resource consumption, typically through `EXPLAIN ANALYZE` or engine-specific profiling tools.

**Technical Definition**: Profiling goes beyond aggregate statistics to reveal *where* within a query the time is spent. PostgreSQL's `EXPLAIN (ANALYZE, BUFFERS)` executes the query and reports actual timing and buffer usage per plan node. MySQL's `EXPLAIN ANALYZE` (8.0.18+) provides tree-formatted per-node timing. SQL Server uses `SET STATISTICS IO, TIME ON` for statement-level metrics and actual execution plans for per-operator details. Oracle uses `DBMS_XPLAN.DISPLAY_CURSOR` with `ALLSTATS LAST` for runtime plan statistics.

**Beginner-Friendly Explanation**: Profiling is like using a stopwatch for every step of a recipe. Instead of just knowing the whole meal took 30 minutes, profiling tells you chopping took 5 minutes, sautéing took 10 minutes, and baking took 15 minutes—so you know where to optimize.

#### Purposes

- To identify the specific plan node (scan, join, sort, aggregate) responsible for the majority of runtime.
- To compare estimated row counts with actual row counts and detect cardinality estimation errors.
- To measure buffer cache hits and disk reads per node.
- To validate that an index is actually being used and reducing work.

#### Syntax Rules and Structure

**PostgreSQL**:

```sql
EXPLAIN (ANALYZE, BUFFERS, VERBOSE)
SELECT ...;
```

**MySQL**:

```sql
EXPLAIN ANALYZE
SELECT ...;
```

**SQL Server**:

```sql
SET STATISTICS IO ON;
SET STATISTICS TIME ON;
SELECT ...;
```

**Component breakdown**:

| Field | Meaning |
|-------|---------|
| `actual time=start..end` | Time to first row and time to last row (per loop, ms) |
| `rows=N` (actual) | Actual rows produced (per loop) |
| `loops=N` | Number of executions; total cost = `actual time × loops` |
| `Buffers: shared hit/read` | Cache hits vs. disk reads (PostgreSQL) |
| `Batches > 1` | Hash join spilled to disk (PostgreSQL) |

**Syntax rules**:

- `ANALYZE` executes the statement; wrap DML in a transaction and roll back if needed.
- `loops` multiplies everything: always compute total contribution as `actual time × loops`.
- `Buffers` reveals I/O pressure; high `shared read` indicates cache misses.

**Constraints and limitations**:

- Profiling a single query in isolation may not reflect production conditions (parameter values, concurrency, cache state).
- `EXPLAIN ANALYZE` adds measurement overhead.
- MySQL `EXPLAIN ANALYZE` supports only `SELECT` statements.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Profiling a nested loop join

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT o.id, c.email
FROM orders o
JOIN customers c ON c.id = o.customer_id
WHERE o.status = 'pending';
```

**Expected output** (excerpt):

```text
Nested Loop  (actual time=0.050..2500.000 rows=10000 loops=1)
  ->  Seq Scan on orders o  (actual time=0.010..100.000 rows=10000 loops=1)
        Filter: (status = 'pending')
        Buffers: shared hit=500 read=100
  ->  Index Scan using customers_pkey on customers c
        (actual time=0.001..0.001 rows=1 loops=10000)
        Index Cond: (id = o.customer_id)
        Buffers: shared hit=30000
```

**Why this output appears**: The inner index scan runs 10,000 times (once per pending order), with 0.001 ms per loop. The total inner contribution is approximately 10,000 × 0.001 ms = 10 ms, but the buffer statistics show 30,000 cache hits, indicating repeated lookups. The outer sequential scan took 100 ms. The total query time is dominated by the repeated inner executions and their cumulative overhead.

#### Real-World Cases

- **Bad plan diagnosis**: Profiling reveals a nested loop with a high `loops` count where a hash join would be faster.
- **Spill detection**: `Batches > 1` or `external merge Disk` indicates memory pressure.
- **Estimation error detection**: A node with estimated `rows=10` but actual `rows=100000` signals stale statistics.

---

### 4. Query Timing

#### Definitions

**Core Definition**: Query timing is the measurement of how long a query takes to execute, typically divided into planning time and execution time.

**Technical Definition**: SQL engines report two timing phases: **planning time** (parsing, rewriting, and optimizing the query) and **execution time** (running the chosen plan). In PostgreSQL, `EXPLAIN ANALYZE` reports both at the top of the plan. MySQL's `performance_schema` records `TIMER_WAIT` (total elapsed) and separates CPU time from wait time. SQL Server's `SET STATISTICS TIME` reports parse/compile time and execution time. Oracle's `V$SQL` stores `PARSE_TIME`, `PLAN_TIME`, and `EXECUTION_TIME` separately.

**Beginner-Friendly Explanation**: Query timing tells you whether a query is slow because the database is *thinking* too long (planning) or *working* too long (execution). Planning problems usually mean complex queries or too many joins; execution problems mean scanning too much data or waiting on locks.

#### Purposes

- To determine whether optimization should focus on query complexity (planning) or access paths (execution).
- To measure the impact of index changes on execution time.
- To correlate application-level latency with database-level timing.
- To establish performance baselines for regression detection.

#### Syntax Rules and Structure

**PostgreSQL** (`EXPLAIN ANALYZE` reports both):

```text
Planning Time: 0.150 ms
Execution Time: 1523.456 ms
```

**SQL Server** (`STATISTICS TIME`):

```text
SQL Server parse and compile time:
   CPU time = 15 ms, elapsed time = 20 ms.
SQL Server Execution Times:
   CPU time = 1500 ms,  elapsed time = 2500 ms.
```

**Component breakdown**:

| Engine | Planning metric | Execution metric |
|--------|-----------------|------------------|
| PostgreSQL | `Planning Time` | `Execution Time` |
| MySQL | `TIMER_WAIT` (total) | `CPU_TIME` / wait breakdown |
| SQL Server | Parse/compile time | Execution CPU and elapsed |
| Oracle | `PARSE_TIME`, `PLAN_TIME` | `EXECUTION_TIME` |

**Syntax rules**:

- Planning time is usually small but can dominate for complex queries with many joins or partitions.
- Execution time includes time waiting for locks, I/O, and CPU.
- High `loops` in execution plans means execution time is cumulative.

**Constraints and limitations**:

- Planning time depends on statistics freshness and query complexity.
- Execution time measured on a cold cache differs from warm-cache timing.
- Concurrent workload affects execution time even for the same query.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Distinguishing planning from execution time

```sql
EXPLAIN ANALYZE
SELECT ...
FROM large_fact f
JOIN dim1 d1 ON d1.id = f.d1_id
JOIN dim2 d2 ON d2.id = f.d2_id
JOIN dim3 d3 ON d3.id = f.d3_id
WHERE f.date >= '2025-01-01';
```

**Expected output** (excerpt):

```text
Planning Time: 45.200 ms
Execution Time: 2500.000 ms
```

**Why this output appears**: Planning took 45 ms (significant for a complex join query), but execution dominated at 2.5 seconds. Optimizing indexes or statistics would address the execution time; simplifying the query or increasing `geqo_threshold` might reduce planning time.

##### Example 2: SQL Server – CPU vs. elapsed time

```sql
SET STATISTICS TIME ON;
SELECT * FROM Orders WHERE Status = 'pending';
```

**Expected output**:

```text
SQL Server Execution Times:
   CPU time = 1500 ms,  elapsed time = 2500 ms.
```

**Why this output appears**: CPU time (1500 ms) is significantly less than elapsed time (2500 ms), indicating that the query spent about 1 second waiting (likely on I/O or locks). A query where CPU time ≈ elapsed time is CPU-bound; one where elapsed ≫ CPU is wait-bound.

#### Real-World Cases

- **Planning-dominated queries**: Dynamic SQL with many joins or partitioned tables may spend more time planning than executing; plan caching or simplified queries help.
- **Wait-bound queries**: Elapsed time much greater than CPU time indicates I/O or lock waits; the fix is indexing or lock reduction.
- **Cold vs. warm cache**: The first execution of a query reads from disk; subsequent executions may be much faster.

---

### 5. Resource Consumption

#### Definitions

**Core Definition**: Resource consumption is the amount of I/O, CPU, memory, and lock time a query uses during execution, measured through engine-specific statistics views and system counters.

**Technical Definition**: Resource consumption metrics include **logical reads** (pages accessed from cache), **physical reads** (pages read from disk), **CPU time** (processor cycles), **memory usage** (sort/hash work areas, buffer cache), and **wait time** (time spent waiting for locks, I/O, or latches). Each RDBMS exposes these metrics through catalog views: PostgreSQL `pg_stat_statements` (I/O timing), `pg_stat_database` (cache hit ratio, temp files), and `pg_stat_bgwriter` (background writer I/O); MySQL `performance_schema` (file I/O, wait events, memory); SQL Server `sys.dm_exec_query_stats` (CPU, reads, writes, memory grants); Oracle `V$SQL` (buffer gets, disk reads, CPU time).

**Beginner-Friendly Explanation**: Resource consumption tells you *what* your query is spending. Is it burning CPU doing calculations? Is it reading millions of pages from disk? Is it waiting for memory? Is it holding locks that block other queries? Each resource has a different fix.

#### Purposes

- To determine whether a query is I/O-bound, CPU-bound, memory-bound, or wait-bound.
- To prioritize tuning by total resource impact across the workload.
- To detect memory spills (hash joins, sorts) that degrade performance.
- To monitor cache hit ratios and disk I/O trends.

#### Syntax Rules and Structure

**PostgreSQL** (I/O and cache metrics):

```sql
-- Database-level cache hit ratio.
SELECT datname,
       blks_hit, blks_read,
       ROUND(blks_hit * 100.0 / NULLIF(blks_hit + blks_read, 0), 2) AS cache_hit_pct
FROM pg_stat_database;

-- Per-query I/O.
SELECT query, shared_blks_hit, shared_blks_read,
       shared_blks_dirtied, temp_blks_read, temp_blks_written
FROM pg_stat_statements
ORDER BY shared_blks_read DESC
LIMIT 10;
```

**MySQL** (wait events and memory):

```sql
SELECT EVENT_NAME, COUNT_STAR, SUM_TIMER_WAIT
FROM performance_schema.events_waits_summary_global_by_event_name
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 10;
```

**SQL Server** (CPU, reads, memory grants):

```sql
SELECT TOP 10
    total_worker_time/1000 AS cpu_ms,
    total_logical_reads,
    total_physical_reads,
    total_grant_kb
FROM sys.dm_exec_query_stats
ORDER BY total_worker_time DESC;
```

**Component breakdown**:

| Resource | PostgreSQL metric | MySQL metric | SQL Server metric |
|----------|-------------------|--------------|-------------------|
| I/O reads | `shared_blks_read` | `events_waits` (file I/O) | `total_physical_reads` |
| Cache hits | `shared_blks_hit` | Buffer pool stats | `total_logical_reads` |
| CPU | CPU time (OS) | `CPU_TIME` | `total_worker_time` |
| Memory | `work_mem` spill (`temp_blks_*`) | `memory_summary` | `total_grant_kb` |

**Syntax rules**:

- Cache hit ratio near 100% indicates good memory utilization; low ratios indicate disk I/O pressure.
- `temp_blks_written` in PostgreSQL indicates spill from sorts or hash joins.
- SQL Server memory grants should match actual usage; large discrepancies indicate estimation errors.

**Constraints and limitations**:

- Logical reads count cache accesses, not disk accesses; high logical reads still consume CPU and memory bandwidth.
- CPU time may include time spent on other processes on a busy system.
- Memory metrics are engine-specific and not directly comparable across vendors.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Detecting cache misses and spill

```sql
SELECT query,
       calls,
       shared_blks_hit,
       shared_blks_read,
       temp_blks_written
FROM pg_stat_statements
WHERE temp_blks_written > 0
ORDER BY temp_blks_written DESC
LIMIT 5;
```

**Expected output**:

```text
 query                                  | calls | shared_blks_hit | shared_blks_read | temp_blks_written
----------------------------------------+-------+-----------------+------------------+-------------------
 SELECT * FROM orders ORDER BY total DESC | 1000 | 200000          | 50000            | 12800
```

**Why this output appears**: The query reads 200,000 pages from cache and 50,000 from disk, but also writes 12,800 temporary blocks—a clear sign that the sort spilled to disk. Increasing `work_mem` or adding an index on `total` would eliminate the spill.

#### Real-World Cases

- **I/O-bound reporting**: Large fact-table scans with low cache hit ratios benefit from indexes or materialized views.
- **CPU-bound aggregation**: Hash aggregation on high-cardinality data consumes CPU; reducing the number of groups or using pre-aggregation helps.
- **Memory spills**: Hash joins or sorts that exceed `work_mem` spill to disk; increasing memory or restructuring the query helps.

---

### 6. I/O Statistics

#### Definitions

**Core Definition**: I/O statistics measure the volume and pattern of disk reads and writes performed by queries, distinguishing between logical reads (cache) and physical reads (disk).

**Technical Definition**: I/O statistics include **shared buffer hits** (pages found in PostgreSQL's shared buffer cache), **shared reads** (pages read from disk into cache), **temp reads/writes** (temporary files for sorts, hashes, and materialized views), and **background writer statistics** (`pg_stat_bgwriter`: `buffers_clean`, `maxwritten_clean`, `buffers_alloc`). MySQL exposes file I/O through `performance_schema.file_summary_by_event_name`. SQL Server reports `logical_reads`, `physical_reads`, `read_ahead_reads`, and `lob_logical_reads` in `SET STATISTICS IO`. Oracle reports `PHYSICAL_READS`, `DBLOCK_GETS`, and `CONSISTENT_GETS` in `V$SQL`.

**Beginner-Friendly Explanation**: I/O statistics tell you how much work the disk is doing. If a query reads mostly from cache, it is fast. If it reads mostly from disk, it is slow and might benefit from an index or more memory.

#### Purposes

- To distinguish between cache-served and disk-served I/O.
- To identify queries that cause excessive temporary file usage (spill).
- To monitor the effectiveness of the buffer cache and background writer.
- To correlate I/O spikes with query execution.

#### Syntax Rules and Structure

**PostgreSQL** (`EXPLAIN (ANALYZE, BUFFERS)`):

```text
Buffers: shared hit=1000 read=5000, temp read=1024 written=512
```

**PostgreSQL** (`pg_stat_bgwriter`):

```sql
SELECT buffers_clean, maxwritten_clean, buffers_alloc
FROM pg_stat_bgwriter;
```

**MySQL** (`performance_schema`):

```sql
SELECT FILE_NAME, COUNT_READ, COUNT_WRITE, SUM_TIMER_READ
FROM performance_schema.file_summary_by_event_name
ORDER BY SUM_TIMER_READ DESC
LIMIT 10;
```

**SQL Server** (`SET STATISTICS IO ON`):

```text
Table 'Orders'. Scan count 1, logical reads 5000, physical reads 100, read-ahead reads 200.
```

**Component breakdown**:

| Metric | Meaning |
|--------|---------|
| `shared hit` | Page found in PostgreSQL shared buffers |
| `shared read` | Page read from disk |
| `temp read/written` | Temporary file I/O (spill) |
| `logical reads` | Pages accessed from SQL Server buffer pool |
| `physical reads` | Pages read from disk |

**Syntax rules**:

- High `shared read` relative to `shared hit` indicates cache pressure.
- `temp read/written` > 0 means a sort or hash spilled to disk.
- SQL Server `read-ahead reads` indicate prefetching; high values can be efficient for sequential scans.

**Constraints and limitations**:

- Buffer cache state affects read counts; warm caches show fewer physical reads.
- Temporary file I/O may be unavoidable for very large sorts; tune memory or restructure the query.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Identifying temp file spill

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM orders ORDER BY total DESC;
```

**Expected output** (excerpt):

```text
Sort  (actual time=5000.000..5500.000 rows=1000000 loops=1)
  Sort Key: total DESC
  Sort Method: external merge  Disk: 102400kB
  Buffers: shared hit=20000 read=1000, temp read=12800 written=12800
```

**Why this output appears**: The `temp read=12800` and `written=12800` indicate that the sort spilled 12,800 pages to disk because it exceeded `work_mem`. The `external merge Disk: 102400kB` confirms the spill volume.

#### Real-World Cases

- **Buffer cache sizing**: Low cache hit ratios across the workload suggest increasing `shared_buffers`.
- **Spill elimination**: Increasing `work_mem` for specific queries eliminates temp I/O.
- **Background writer tuning**: High `maxwritten_clean` may indicate the background writer is stopping too frequently.

---

### 7. CPU Statistics

#### Definitions

**Core Definition**: CPU statistics measure the processor time consumed by query execution, distinguishing between user CPU time (query processing) and system CPU time (kernel operations).

**Technical Definition**: CPU statistics include **worker time** (SQL Server `total_worker_time`), **CPU time** per statement (MySQL `CPU_TIME`, Oracle `CPU_TIME`), and operating-system-level `%user`/`%system` utilization. High CPU consumption relative to elapsed time indicates a CPU-bound query (e.g., complex calculations, large aggregations, inefficient joins). High elapsed time relative to CPU indicates wait-bound execution (I/O, locks).

**Beginner-Friendly Explanation**: CPU statistics tell you how hard the processor is working on your query. If CPU time is close to total elapsed time, the query is CPU-bound—it is doing a lot of calculations. If CPU time is much lower than elapsed time, the query is waiting for something (disk or locks).

#### Purposes

- To distinguish CPU-bound queries from wait-bound queries.
- To identify queries that consume disproportionate processor time.
- To detect parallel query execution overhead (SQL Server `CXPACKET` waits).
- To prioritize indexing or query simplification for CPU-heavy operations.

#### Syntax Rules and Structure

**SQL Server**:

```sql
SELECT TOP 10
    total_worker_time/1000 AS total_cpu_ms,
    execution_count,
    total_worker_time/execution_count/1000 AS avg_cpu_ms
FROM sys.dm_exec_query_stats
ORDER BY total_worker_time DESC;
```

**MySQL**:

```sql
SELECT digest_text, COUNT_STAR,
       ROUND(SUM_CPU_TIME/1e12, 2) AS total_cpu_seconds
FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_CPU_TIME DESC
LIMIT 10;
```

**Component breakdown**:

| Metric | Meaning |
|--------|---------|
| `total_worker_time` | Total CPU time (SQL Server) |
| `CPU_TIME` | CPU time per statement (MySQL/Oracle) |
| `%user` / `%system` | OS-level CPU utilization |

**Syntax rules**:

- Compare CPU time to elapsed time to determine if a query is CPU-bound or wait-bound.
- High CPU with low elapsed time indicates efficient CPU utilization; high CPU with high elapsed time indicates CPU saturation.
- SQL Server `CXPACKET` waits indicate parallelism coordination overhead, often a sign of imbalanced parallel query distribution.

**Constraints and limitations**:

- CPU time includes time spent on other processes on a shared server.
- Parallel queries may use multiple CPUs, inflating CPU time relative to elapsed time.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – CPU vs. elapsed analysis

```sql
SET STATISTICS TIME ON;
SELECT CustomerID, SUM(TotalDue)
FROM Sales.SalesOrderHeader
GROUP BY CustomerID;
```

**Expected output**:

```text
SQL Server Execution Times:
   CPU time = 2500 ms,  elapsed time = 2600 ms.
```

**Why this output appears**: CPU time (2500 ms) is nearly equal to elapsed time (2600 ms), indicating the query is CPU-bound (the aggregation consumes processor time). If elapsed were 10,000 ms with CPU at 2500 ms, the query would be wait-bound (I/O or locks).

#### Real-World Cases

- **CPU-bound aggregation**: High CPU from hash aggregation; reducing group cardinality or pre-aggregating helps.
- **Parallelism overhead**: High `CXPACKET` waits with high CPU; adjusting `MAXDOP` or `cost threshold for parallelism` may help.
- **Implicit conversions**: Queries that convert types per row consume CPU; fixing data types eliminates the conversion.

---

### 8. Memory Statistics

#### Definitions

**Core Definition**: Memory statistics measure the working memory allocated to query operators (sorts, hash joins, aggregations) and the buffer cache, revealing whether queries are operating within memory or spilling to disk.

**Technical Definition**: PostgreSQL uses `work_mem` per operation; `EXPLAIN (ANALYZE, BUFFERS)` reports `Memory Usage` for hash nodes and `Sort Method: external merge Disk` for spills. MySQL's `performance_schema.memory_summary_global_by_event_name` reports memory allocation per instrument. SQL Server uses **memory grants**: `sys.dm_exec_query_memory_grants` shows `requested_memory_kb`, `granted_memory_kb`, and `used_memory_kb`; `sys.dm_exec_query_stats.total_grant_kb` shows the grant per cached plan. Oracle uses the PGA (Program Global Area) and `V$PGASTAT`.

**Beginner-Friendly Explanation**: Memory statistics tell you whether your query has enough desk space to work. If a sort or hash join runs out of memory, it spills to disk, which is much slower. Memory statistics show whether spill is happening and how much memory the query is using.

#### Purposes

- To detect hash joins and sorts that spill to disk due to insufficient memory.
- To evaluate whether memory grants in SQL Server are appropriate for actual usage.
- To tune `work_mem` (PostgreSQL) or memory grant settings (SQL Server).
- To identify queries that request excessive memory and starve other queries.

#### Syntax Rules and Structure

**PostgreSQL** (spill indicators):

```text
Hash
  Buckets: 65536  Batches: 8  Memory Usage: 4096kB
Sort
  Sort Method: external merge  Disk: 102400kB
```

**SQL Server** (memory grants):

```sql
SELECT session_id, requested_memory_kb, granted_memory_kb, used_memory_kb, wait_time_ms
FROM sys.dm_exec_query_memory_grants;
```

**Component breakdown**:

| Metric | Meaning |
|--------|---------|
| `Batches > 1` | Hash join spilled to disk (PostgreSQL) |
| `external merge Disk` | Sort spilled to disk |
| `granted_memory_kb` | Memory actually allocated (SQL Server) |
| `used_memory_kb` | Memory actually used |
| `wait_time_ms` | Time spent waiting for a memory grant |

**Syntax rules**:

- `Batches = 1` in a hash join indicates the hash table fits in memory.
- SQL Server `granted_memory_kb` much larger than `used_memory_kb` indicates an overestimate; much smaller indicates an underestimate and potential spill.
- Memory grants are calculated from estimated cardinality; bad statistics cause bad grants.

**Constraints and limitations**:

- Increasing `work_mem` globally affects all concurrent queries; use per-session settings for heavy queries.
- SQL Server memory grants are limited by `max server memory` and Resource Governor.
- Memory statistics do not directly measure the buffer cache; use buffer statistics for that.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Detecting memory grant waits

```sql
SELECT session_id, requested_memory_kb, granted_memory_kb,
       used_memory_kb, wait_time_ms
FROM sys.dm_exec_query_memory_grants
WHERE wait_time_ms > 0
ORDER BY wait_time_ms DESC;
```

**Expected output**:

```text
session_id requested_memory_kb granted_memory_kb used_memory_kb wait_time_ms
---------- ------------------- ------------------ -------------- -------------
55         1024000             1024000            900000         1500
```

**Why this output appears**: Session 55 waited 1,500 ms for a 1 GB memory grant. The grant was fully allocated and mostly used (900 MB), suggesting the estimate was reasonable but memory was temporarily unavailable due to concurrent queries.

#### Real-World Cases

- **Hash join spill**: `Batches > 1` in PostgreSQL or `granted_memory_kb < used_memory_kb` in SQL Server; increasing memory or fixing statistics helps.
- **Memory grant waits**: Queries queue for memory grants; reducing grant size or concurrency helps.
- **Sort spill**: `external merge Disk` indicates `work_mem` is too low for the sort volume.

---

### 9. Lock Waits

#### Definitions

**Core Definition**: A lock wait occurs when a query cannot acquire a lock on a resource because another session holds an incompatible lock, causing the query to pause until the lock is released or a timeout occurs.

**Technical Definition**: Lock waits are measured through **wait events** and **lock views**. PostgreSQL: `pg_stat_activity.wait_event_type = 'Lock'`, `pg_locks`, and `pg_blocking_pids()`. MySQL: `sys.innodb_lock_waits` (formatted view of `performance_schema.data_lock_waits`), `SHOW ENGINE INNODB STATUS`. SQL Server: `sys.dm_os_waiting_tasks` (`wait_type = 'LCK_M_*'`), `sys.dm_tran_locks`. Oracle: `V$SESSION.blocking_session`, `V$LOCK`. Lock waits are a normal part of concurrency control; persistent waits indicate blocking chains that require intervention.

**Beginner-Friendly Explanation**: A lock wait is like waiting for a fitting room. Someone else is inside (holding the lock), and you cannot enter until they leave. If they take too long, you either wait (potentially forever) or give up (timeout).

#### Purposes

- To identify sessions that are blocked and the sessions blocking them.
- To distinguish lock waits from I/O or CPU waits as the cause of slow queries.
- To resolve blocking chains by terminating or rescheduling the blocking session.
- To set appropriate lock timeout thresholds.

#### Syntax Rules and Structure

**PostgreSQL** (find blocked sessions and blockers):

```sql
SELECT pid, usename, pg_blocking_pids(pid) AS blocked_by,
       wait_event_type, wait_event, query
FROM pg_stat_activity
WHERE CARDINALITY(pg_blocking_pids(pid)) > 0;
```

**MySQL** (current lock waits):

```sql
SELECT * FROM sys.innodb_lock_waits;
```

**SQL Server** (waiting tasks):

```sql
SELECT session_id, wait_type, wait_duration_ms, blocking_session_id, resource_description
FROM sys.dm_os_waiting_tasks
WHERE wait_type LIKE 'LCK_M_%';
```

**Oracle** (blocked sessions):

```sql
SELECT sid, serial#, blocking_session, seconds_in_wait, wait_class
FROM v$session
WHERE blocking_session IS NOT NULL;
```

**Component breakdown**:

| Engine | View | Key columns |
|--------|------|-------------|
| PostgreSQL | `pg_stat_activity` + `pg_blocking_pids()` | `blocked_by`, `wait_event` |
| MySQL | `sys.innodb_lock_waits` | `waiting_pid`, `blocking_pid` |
| SQL Server | `sys.dm_os_waiting_tasks` | `blocking_session_id`, `wait_duration_ms` |
| Oracle | `v$session` | `blocking_session`, `seconds_in_wait` |

**Syntax rules**:

- Always identify the **head blocker**: the session at the root of the blocking chain that is not itself blocked.
- Distinguish between lock waits (`wait_event_type = 'Lock'`) and other waits (I/O, client).
- Lock waits can be caused by long transactions, uncommitted changes, or explicit `LOCK TABLE` statements.

**Constraints and limitations**:

- Lock waits are normal in small durations; only persistent waits (seconds to minutes) require intervention.
- Terminating the head blocker may roll back a large transaction, causing its own performance impact.
- Some engines (MySQL) have separate metadata lock waits (`Waiting for table metadata lock`).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Finding the head blocker

```sql
SELECT pid, usename, state,
       wait_event_type, wait_event,
       pg_blocking_pids(pid) AS blocked_by,
       LEFT(query, 80) AS query
FROM pg_stat_activity
WHERE CARDINALITY(pg_blocking_pids(pid)) > 0
   OR pid IN (
       SELECT unnest(pg_blocking_pids(pid))
       FROM pg_stat_activity
       WHERE CARDINALITY(pg_blocking_pids(pid)) > 0
   )
ORDER BY pid;
```

**Expected output**:

```text
 pid  | usename | state  | wait_event_type | wait_event | blocked_by | query
------+---------+--------+-----------------+------------+------------+------------------
 100  | app     | active | Lock            | transactionid | {200}   | UPDATE orders SET ...
 200  | etl     | idle in transaction | | | {} | 
```

**Why this output appears**: PID 100 is blocked by PID 200 on a transaction ID lock. PID 200 is the head blocker: it is `idle in transaction`, meaning it holds locks but is not executing a statement. The fix is to end PID 200's transaction (commit or rollback) or terminate the session.

#### Real-World Cases

- **Long transactions**: A batch job opens a transaction and holds locks for minutes; all OLTP queries on the same rows wait.
- **Idle-in-transaction**: A connection pool holds an open transaction while idle, blocking other sessions; `idle_in_transaction_session_timeout` prevents this.
- **Metadata locks**: `ALTER TABLE` waits for all open transactions on the table; MySQL `Waiting for table metadata lock` is a common symptom.

---

### 10. Blocking Queries

#### Definitions

**Core Definition**: A blocking query is a query (or transaction) that holds a lock preventing other queries from proceeding; the chain of blocked queries waiting on it is called a blocking chain.

**Technical Definition**: Blocking is an inherent property of lock-based concurrency control. A session that holds an exclusive lock on a row or table blocks other sessions that request a conflicting lock. The **head blocker** is the session at the root of the chain that is not itself blocked by another session. Blocking becomes problematic when the head blocker holds locks for an extended period (long transaction, uncommitted changes, or idle-in-transaction state). Detection uses the same views as lock waits; resolution involves committing/rolling back the blocking transaction, terminating the session, or adjusting isolation levels.

**Beginner-Friendly Explanation**: A blocking query is the one holding up the line. Everyone else is waiting for it to finish so they can get their turn. The goal is to find the query at the front of the line and figure out why it is taking so long.

#### Purposes

- To identify the root cause of a blocking chain.
- To terminate or reschedule the blocking session to restore throughput.
- To design applications that avoid long-held locks (short transactions, appropriate isolation levels).
- To set lock timeout thresholds that fail fast rather than waiting indefinitely.

#### Syntax Rules and Structure

**PostgreSQL** (recursive blocking tree):

```sql
WITH RECURSIVE blocking_tree AS (
    SELECT pid, usename, query, pg_blocking_pids(pid) AS blocked_by, 0 AS depth
    FROM pg_stat_activity
    WHERE CARDINALITY(pg_blocking_pids(pid)) = 0
      AND EXISTS (SELECT 1 FROM pg_stat_activity a2 WHERE pid = ANY(pg_blocking_pids(a2.pid)))
    UNION ALL
    SELECT a.pid, a.usename, a.query, pg_blocking_pids(a.pid), bt.depth + 1
    FROM pg_stat_activity a
    JOIN blocking_tree bt ON a.pid = ANY(bt.blocked_by)
)
SELECT * FROM blocking_tree ORDER BY depth;
```

**SQL Server** (`sp_who2` or DMV):

```sql
EXEC sp_who2;
-- Or:
SELECT blocking_session_id, session_id, wait_type, wait_time, status, command
FROM sys.dm_exec_requests
WHERE blocking_session_id <> 0;
```

**MySQL** (blocking transactions):

```sql
SELECT * FROM sys.innodb_lock_waits;
-- Or:
SELECT waiting_pid, blocking_pid, locked_table, locked_index
FROM performance_schema.data_lock_waits;
```

**Component breakdown**:

| Engine | Blocking detection | Head blocker resolution |
|--------|-------------------|-------------------------|
| PostgreSQL | `pg_blocking_pids()` | `pg_cancel_backend()` / `pg_terminate_backend()` |
| MySQL | `sys.innodb_lock_waits` | `KILL QUERY` / `KILL CONNECTION` |
| SQL Server | `sys.dm_exec_requests.blocking_session_id` | `KILL <spid>` |
| Oracle | `v$session.blocking_session` | `ALTER SYSTEM KILL SESSION` |

**Syntax rules**:

- Blocking is not always a problem; short-duration blocking is normal. Only persistent blocking requires intervention.
- The head blocker is the session with no `blocking_session_id` (SQL Server) or no `blocking_session` (Oracle) that is blocking others.
- Terminating the head blocker rolls back its transaction; ensure this is safe before proceeding.

**Constraints and limitations**:

- Killing a session may leave the database in a transaction rollback state, consuming resources.
- Preventing blocking requires application changes (shorter transactions, row-level locking, optimistic concurrency).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Identifying and resolving a blocking chain

```sql
-- Step 1: Find the head blocker.
SELECT blocking_session_id, session_id, wait_type, wait_time, command
FROM sys.dm_exec_requests
WHERE blocking_session_id <> 0;

-- Step 2: Terminate the head blocker (if appropriate).
KILL 55;
```

**Expected output** (Step 1):

```text
blocking_session_id session_id wait_type wait_time command
------------------- ---------- --------- --------- -----------
55                  60         LCK_M_X   15000     UPDATE
55                  72         LCK_M_S   12000     SELECT
```

**Why this output appears**: Sessions 60 and 72 are blocked by session 55 on exclusive and shared locks, respectively. Session 55 is the head blocker. `KILL 55` rolls back its transaction and releases the locks, allowing the blocked sessions to proceed.

#### Real-World Cases

- **Batch updates vs. OLTP**: A nightly batch update holds locks on large row sets, blocking daytime OLTP queries; scheduling the batch during off-hours or using smaller batches helps.
- **Idle-in-transaction**: A connection pool leaves a transaction open, blocking `ALTER TABLE`; `idle_in_transaction_session_timeout` resolves it.
- **Deadlocks**: Cyclic blocking where each session waits for the other; the engine detects and kills one victim.

---

### 11. Query Cancellation

#### Definitions

**Core Definition**: Query cancellation is the act of terminating a running SQL statement (or its entire session) before it completes, typically because it is consuming excessive resources or blocking other work.

**Technical Definition**: Cancellation mechanisms differ by engine: PostgreSQL `pg_cancel_backend(pid)` sends `SIGINT` to cancel the current statement; `pg_terminate_backend(pid)` sends `SIGTERM` to end the session. MySQL `KILL QUERY id` terminates the statement but leaves the connection open; `KILL CONNECTION id` (or `KILL id`) ends the session. SQL Server `KILL spid` terminates the session; there is no built-in statement-level cancellation from another session. Oracle `ALTER SYSTEM KILL SESSION 'sid,serial#'` terminates the session. Application-level cancellation uses statement timeouts.

**Beginner-Friendly Explanation**: Cancellation is the emergency brake. When a query has been running for too long and is causing problems, you can stop it—either just the query (letting the connection continue) or the whole connection.

#### Purposes

- To stop a runaway query that is consuming excessive CPU, I/O, or memory.
- To unblock other queries by terminating the head blocker.
- To free resources during an incident.
- To enforce application-level timeouts that prevent indefinite waiting.

#### Syntax Rules and Structure

**PostgreSQL**:

```sql
SELECT pg_cancel_backend(12345);   -- cancel statement
SELECT pg_terminate_backend(12345); -- terminate session
```

**MySQL**:

```sql
KILL QUERY 12345;      -- cancel statement
KILL CONNECTION 12345; -- terminate connection
KILL 12345;            -- same as KILL CONNECTION
```

**SQL Server**:

```sql
KILL 55;  -- terminates the session
```

**Oracle**:

```sql
ALTER SYSTEM KILL SESSION '123,456' IMMEDIATE;
```

**Component breakdown**:

| Engine | Statement-level cancellation | Session termination |
|--------|------------------------------|---------------------|
| PostgreSQL | `pg_cancel_backend()` | `pg_terminate_backend()` |
| MySQL | `KILL QUERY` | `KILL CONNECTION` |
| SQL Server | Not available from another session | `KILL` |
| Oracle | Not available directly | `ALTER SYSTEM KILL SESSION` |

**Syntax rules**:

- Cancellation requires appropriate privileges (superuser in PostgreSQL, `CONNECTION_ADMIN` or `SUPER` in MySQL, `ALTER ANY CONNECTION` in SQL Server).
- Statement-level cancellation leaves the connection open; session termination closes it.
- Cancellation may roll back the statement's changes; ensure transactional integrity.

**Constraints and limitations**:

- SQL Server does not support cancelling a single statement from another session; `KILL` terminates the session.
- Cancellation may take time if the statement is in a non-interruptible phase (e.g., certain I/O operations).
- Repeated cancellation without root-cause analysis leads to recurring incidents.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Cancelling a runaway query

```sql
-- Step 1: Find the PID of the runaway query.
SELECT pid, now() - query_start AS duration, query
FROM pg_stat_activity
WHERE state = 'active'
  AND now() - query_start > interval '5 minutes';

-- Step 2: Cancel the statement.
SELECT pg_cancel_backend(12345);

-- Step 3: If cancellation is insufficient, terminate the session.
SELECT pg_terminate_backend(12345);
```

**Expected output**:

```text
 pg_cancel_backend
-------------------
 t
(1 row)
```

**Why this output appears**: `pg_cancel_backend` returns `t` (true) if the signal was successfully sent. The query receives `SIGINT` and aborts with a "canceling statement due to user request" error.

#### Real-World Cases

- **Runaway reporting query**: A query without a `LIMIT` scans a billion-row table; cancellation stops it before it exhausts resources.
- **Blocking chain resolution**: Terminating the head blocker releases locks for waiting sessions.
- **Application timeouts**: Setting `statement_timeout` or `max_execution_time` prevents queries from running indefinitely.

---

### 12. Query Timeouts

#### Definitions

**Core Definition**: A query timeout is a configured limit on how long a query (or lock wait) may run before the database automatically cancels it and returns an error.

**Technical Definition**: Timeouts are enforced by engine-level parameters or session-level settings. PostgreSQL provides `statement_timeout` (per statement), `lock_timeout` (per lock wait), `idle_in_transaction_session_timeout` (idle transactions), and `transaction_timeout` (PostgreSQL 17+). MySQL provides `max_execution_time` (SELECT only, milliseconds) and `innodb_lock_wait_timeout` (seconds). SQL Server uses `LOCK_TIMEOUT` (milliseconds for lock waits) and application-level command timeouts. Oracle uses `RESOURCE_LIMIT` and profiles with `IDLE_TIME` and `CONNECT_TIME`.

**Beginner-Friendly Explanation**: A timeout is an alarm clock for your query. If the query runs longer than the alarm allows, the database stops it and tells the application "this took too long." It prevents a single slow query from hogging resources forever.

#### Purposes

- To prevent runaway queries from consuming resources indefinitely.
- To fail fast when locks cannot be acquired within a reasonable time.
- To enforce application SLA expectations at the database level.
- To protect the database from connection pool exhaustion caused by long-running queries.

#### Syntax Rules and Structure

**PostgreSQL**:

```sql
-- Session-level.
SET statement_timeout = '30s';
SET lock_timeout = '5s';
SET idle_in_transaction_session_timeout = '60s';

-- Per-role.
ALTER ROLE app_user SET statement_timeout = '10s';
```

**MySQL**:

```sql
SET SESSION max_execution_time = 5000;  -- milliseconds (SELECT only)
SET SESSION innodb_lock_wait_timeout = 10;  -- seconds
```

**SQL Server**:

```sql
SET LOCK_TIMEOUT 5000;  -- milliseconds; -1 = infinite
```

**Component breakdown**:

| Engine | Parameter | Scope | Unit |
|--------|-----------|-------|------|
| PostgreSQL | `statement_timeout` | Session/role/database | ms (0 = disabled) |
| PostgreSQL | `lock_timeout` | Session | ms |
| MySQL | `max_execution_time` | Session | ms (SELECT only) |
| MySQL | `innodb_lock_wait_timeout` | Session | seconds |
| SQL Server | `LOCK_TIMEOUT` | Session | ms (-1 = infinite) |

**Syntax rules**:

- `statement_timeout` cancels the entire statement after the specified duration.
- `lock_timeout` cancels only if the statement is waiting for a lock; it does not limit execution time.
- MySQL `max_execution_time` applies only to `SELECT` statements; it does not limit `INSERT`, `UPDATE`, or `DELETE`.
- Timeouts return specific error codes (PostgreSQL `57014`, MySQL `3024`, SQL Server `1222`).

**Constraints and limitations**:

- Timeouts are a safety net, not a substitute for query optimization.
- Setting timeouts too low may cancel legitimate long-running batch jobs.
- Application-level timeouts (e.g., HTTP timeouts, ORM timeouts) may fire before database-level timeouts.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Setting statement and lock timeouts

```sql
-- Step 1: Set a 5-second statement timeout for an interactive session.
SET statement_timeout = '5s';

-- Step 2: Run a potentially slow query.
SELECT * FROM large_table ORDER BY random();

-- Step 3: If the query exceeds 5 seconds, it is cancelled.
```

**Expected output**:

```text
ERROR:  canceling statement due to statement timeout
```

**Why this output appears**: The `statement_timeout` parameter triggers cancellation after 5 seconds, returning error code `57014`. The application receives the error and can retry, show a message, or log the incident.

##### Example 2: MySQL – Limiting lock wait time

```sql
SET SESSION innodb_lock_wait_timeout = 5;

-- Attempt to update a row locked by another transaction.
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
```

**Expected output** (after 5 seconds):

```text
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```

**Why this output appears**: The `innodb_lock_wait_timeout` limits how long the `UPDATE` waits for the row lock. After 5 seconds, the statement fails with error 1205 rather than waiting indefinitely.

#### Real-World Cases

- **OLTP applications**: `statement_timeout = '30s'` prevents a single query from holding a connection for minutes.
- **Batch jobs**: Longer timeouts (or disabled timeouts) for ETL processes that legitimately run for hours.
- **Lock contention**: `lock_timeout` prevents sessions from queuing indefinitely behind a blocking transaction.

---

## Summary: Tuning Decision Matrix

| Symptom | Primary diagnostic | Likely cause | Tuning action |
|---------|--------------------|--------------|---------------|
| Query slow but not blocked | `EXPLAIN ANALYZE` | Bad plan, missing index, spill | Add index, update statistics, increase `work_mem` |
| Query waiting on lock | `pg_blocking_pids`, `sys.innodb_lock_waits` | Blocking transaction | Commit/rollback blocker, adjust isolation |
| High CPU, low elapsed | `STATISTICS TIME`, `total_worker_time` | CPU-bound aggregation or calculation | Simplify query, reduce cardinality |
| High elapsed, low CPU | `STATISTICS TIME`, `wait_event` | I/O or lock wait | Index, increase cache, resolve blocking |
| Temp file writes | `temp_blks_written`, `Sort Method: external merge` | Memory spill | Increase `work_mem`, reduce data size |
| Cache miss ratio high | `blks_read`, `physical_reads` | Insufficient buffer cache | Increase `shared_buffers`, add index |
| Memory grant wait | `sys.dm_exec_query_memory_grants` | Insufficient memory | Fix statistics, reduce grant, increase memory |
| Repeated blocking | `blocking_session_id` | Long transaction, idle-in-transaction | Shorten transactions, set `idle_in_transaction_session_timeout` |

---

## References

- PostgreSQL Documentation – Monitoring Database Activity – https://www.postgresql.org/docs/current/monitoring.html
- PostgreSQL Documentation – `pg_stat_statements` – https://www.postgresql.org/docs/current/pgstatstatements.html
- PostgreSQL Documentation – `EXPLAIN` – https://www.postgresql.org/docs/current/sql-explain.html
- PostgreSQL Documentation – `statement_timeout` – https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-STATEMENT-TIMEOUT
- MySQL 8.0 Reference Manual – Slow Query Log – https://dev.mysql.com/doc/refman/8.0/en/slow-query-log.html
- MySQL 8.0 Reference Manual – Performance Schema – https://dev.mysql.com/doc/refman/8.0/en/performance-schema.html
- MySQL 8.0 Reference Manual – `KILL` Statement – https://dev.mysql.com/doc/refman/8.0/en/kill.html
- Microsoft Learn – Query Store – https://learn.microsoft.com/en-us/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store
- Microsoft Learn – `sys.dm_exec_query_stats` – https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-stats-transact-sql
- Microsoft Learn – Understand and Resolve SQL Server Blocking Problems – https://learn.microsoft.com/en-us/troubleshoot/sql/performance/understand-resolve-blocking
- Microsoft Learn – `SET LOCK_TIMEOUT` – https://learn.microsoft.com/en-us/sql/t-sql/statements/set-lock-timeout-transact-sql
- Oracle Database SQL Tuning Guide – Automatic Workload Repository – https://docs.oracle.com/en/database/oracle/oracle-database/23/tgsql/automatic-workload-repository.html
- Oracle Database Reference – `V$SESSION` – https://docs.oracle.com/en/database/oracle/oracle-database/23/refrn/V-SESSION.html
- Oracle Database Reference – `V$SQL` – https://docs.oracle.com/en/database/oracle/oracle-database/23/refrn/V-SQL.html
- Amazon CloudWatch – Monitoring Self-Managed PostgreSQL – https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Database-Insights-Self-Managed-PostgreSQL.html
- MyDBA – How Do I Find Long-Running Queries in PostgreSQL? – https://mydba.dev/blog/postgres-long-running-queries
- IBM Cloud Docs – PostgreSQL Queries – https://ondeck.console.cloud.ibm.com/docs/databases-for-postgresql
- Guance – How to Monitor MySQL Slow Queries – https://www.guance.com/learn/articles/how-to-monitor-mysql-slow-queries
- OneUptime – How to Kill a Long-Running Query in MySQL – https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-kill-long-running-query/README.md
- Microsoft Learn – Monitor and Tune Queries With Query Store – https://learn.microsoft.com/en-us/training/modules/optimize-database-performance/05-monitor-tune-queries-query-store
- Microsoft Support – Understand and Resolve SQL Server Blocking Problems – https://github.com/microsoftdocs/supportarticles-docs/blob/f35d5396cb94e2ff813f0e643d6b227d885aa8df/support/sql/database-engine/performance/understand-resolve-blocking.md