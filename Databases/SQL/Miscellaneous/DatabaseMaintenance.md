# SQL Database Maintenance: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**: Database maintenance is the set of recurring administrative operations that preserve database performance, reclaim storage, and ensure data integrity over time.

**Technical Definition**: Database maintenance encompasses statistics refresh (optimizer cardinality estimation), index defragmentation and rebuilding, table partitioning and compression, storage capacity monitoring, transaction log lifecycle management, dead-tuple reclamation (vacuuming), and health telemetry collection—each executed through engine-specific DDL statements, system stored procedures, or background daemon configuration.

**Beginner-Friendly Explanation**: Database maintenance is like servicing a car. Just as you change the oil, rotate the tires, and check fluid levels to keep a vehicle running smoothly, database maintenance involves updating the database's "maps" (statistics), tidying up its "filing system" (indexes), and clearing out "junk" (dead rows) so queries stay fast and storage doesn't run out.

### Key Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Frequency** | Ranges from continuous (autovacuum) to nightly/weekly (index rebuilds) to quarterly (archiving) |
| **Impact** | Some operations are online (non-blocking); others require locks or maintenance windows |
| **Engine Specificity** | Syntax, tooling, and best practices differ substantially between SQL Server, PostgreSQL, MySQL, and Oracle |
| **Automation** | Most operations can be scheduled via SQL Agent (SQL Server), cron/Patroni (PostgreSQL), or maintenance plans |
| **Observability** | Success requires monitoring fragmentation, bloat, log space, and query performance trends |

### Prerequisites

- **Sufficient Privileges**: `sysadmin` (SQL Server), superuser or table owner (PostgreSQL), or equivalent
- **Maintenance Window**: For operations that take locks (index rebuilds, `VACUUM FULL`)
- **Storage Headroom**: At least 25% free disk space to accommodate growth and temporary operations
- **Backup Before Destructive Operations**: Rebuilds and shrinks should be preceded by a verified backup
- **Monitoring Baseline**: Known-good values for fragmentation, bloat, query latency, and log growth

### Related Programming Areas

- **Database Administration (DBA)** : Scheduling, executing, and troubleshooting maintenance jobs
- **Performance Engineering**: Using statistics and index health to optimize query plans
- **Storage Engineering**: Capacity planning, I/O subsystem tuning, and file placement
- **Site Reliability Engineering (SRE)** : Automating maintenance pipelines and alerting on degradation
- **Data Governance**: Retention policies, archiving, and compliance-driven cleanup

### Core Concepts Overview

SQL database maintenance comprises seven complementary operational domains:

1. **Statistics Updates**: Refreshing optimizer cardinality estimates and histograms
2. **Index Maintenance**: Rebuilding, reorganizing, and pruning unused or duplicate indexes
3. **Table Maintenance**: Partitioning, compression, and data archiving
4. **Storage Monitoring**: Disk space alerts, growth forecasting, and I/O bottleneck analysis
5. **Log Management**: Transaction log truncation (SQL Server) and WAL archive cleanup (PostgreSQL)
6. **Bloat Management**: MVCC dead-tuple reclamation via vacuuming
7. **Health & Performance Telemetry**: Probes, log scraping, and alerting

---

## Core Concept 1: Statistics Updates

### Definitions

**Core Definition**: Statistics are metadata objects that describe the distribution of data values in table columns and indexes.

**Technical Definition**: Optimizer statistics consist of a histogram (frequency distribution of values across ranges) and density vectors (average number of rows per distinct value), stored in system catalogs (`sys.stats` in SQL Server, `pg_statistic` in PostgreSQL) and consumed by the query optimizer to estimate cardinality and select execution plans.

**Beginner-Friendly Explanation**: Statistics are like a map of your data. The database uses them to guess how many rows a query will find, and chooses the fastest route to get those rows. If the map is outdated, the database might take a slow route.

### Purposes

- **To** provide the optimizer with accurate cardinality estimates for query plan selection
- **To** detect and correct stale statistics that cause suboptimal plans
- **To** maintain histogram accuracy for columns with skewed or evolving data distributions
- **To** reduce query latency caused by underestimation or overestimation of row counts

### Syntax Rules and Structure

#### Complete General Syntax (SQL Server)

```sql
UPDATE STATISTICS table_or_indexed_view_name 
    [ ( statistics_name ) ]
    [ WITH 
        FULLSCAN | SAMPLE number PERCENT | RESAMPLE,
        NORECOMPUTE | RECOMPUTE,
        INCREMENTAL,
        MAXDOP = max_degree_of_parallelism
    ];
```

#### Complete General Syntax (PostgreSQL)

```sql
ANALYZE [ VERBOSE ] [ table_name [ ( column_name [, ...] ) ] ];
```

#### Component Breakdown (SQL Server)

| Component | Description |
|-----------|-------------|
| `FULLSCAN` | Scans all rows for maximum accuracy |
| `SAMPLE n PERCENT` | Samples n percent of rows |
| `RESAMPLE` | Uses the existing sampling rate |
| `NORECOMPUTE` | Disables automatic statistics updates for this statistic |
| `INCREMENTAL` | Updates partition-level statistics (partitioned tables) |

#### Component Breakdown (PostgreSQL)

| Component | Description |
|-----------|-------------|
| `VERBOSE` | Emits progress messages |
| `table_name` | Target table (defaults to all tables in current database) |
| `column_name` | Specific column to analyze |

#### Syntax Rules

- SQL Server: `UPDATE STATISTICS` requires the table owner or `db_owner`/`sysadmin` membership
- SQL Server: `AUTO_UPDATE_STATISTICS` should remain `ON` even when manual updates are scheduled
- PostgreSQL: `ANALYZE` requires only a read lock and can run concurrently with other activity
- PostgreSQL: `ANALYZE` takes a random sample for large tables; statistics are approximate and may vary slightly between runs

#### Constraints and Limitations

- SQL Server: Rebuilding an index with `ALTER INDEX REBUILD` does **not** update statistics for non-indexed columns
- SQL Server: Statistics on ascending columns (e.g., `IDENTITY`, timestamps) may require more frequent manual updates because auto-update thresholds are not reached quickly enough
- PostgreSQL: Autovacuum handles automatic analyzing when enabled; manual `ANALYZE` is a fallback strategy
- PostgreSQL: The `default_statistics_target` (default 100) controls histogram granularity; increasing it improves accuracy but slows `ANALYZE` and increases `pg_statistic` size

### Annotated Complete Step-by-Step Code Examples

#### Example 1: SQL Server Targeted Statistics Update

**Setup**: A database with an `Orders` table that has experienced significant inserts.

```sql
-- Step 1: Check when statistics were last updated
SELECT 
    name AS stats_name,
    STATS_DATE(object_id, stats_id) AS last_updated
FROM sys.stats
WHERE object_id = OBJECT_ID('dbo.Orders');
-- Expected: Shows last_updated dates; if stale, proceed.

-- Step 2: Update statistics with FULLSCAN for maximum accuracy
UPDATE STATISTICS dbo.Orders IX_Orders_OrderDate WITH FULLSCAN;
-- Expected: Commands completed successfully.

-- Step 3: Verify the update
SELECT STATS_DATE(OBJECT_ID('dbo.Orders'), stats_id) AS updated_time
FROM sys.stats
WHERE object_id = OBJECT_ID('dbo.Orders')
  AND name = 'IX_Orders_OrderDate';
-- Expected: Returns the current timestamp.
```

**Expected Output**:
```
Commands completed successfully.
```

**Why This Output Occurs**: `FULLSCAN` forces a complete read of the index, producing the most accurate histogram possible. This is recommended for critical indexes on ascending columns where auto-update may lag.

#### Example 2: PostgreSQL ANALYZE

**Setup**: A PostgreSQL database with a `products` table.

```sql
-- Step 1: Create sample table and insert data
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    price NUMERIC(10,2),
    category TEXT
);
INSERT INTO products (name, price, category)
SELECT 'Product ' || i, random() * 1000, 
       CASE WHEN i % 3 = 0 THEN 'Electronics' 
            WHEN i % 3 = 1 THEN 'Clothing' 
            ELSE 'Home' END
FROM generate_series(1, 10000) AS i;

-- Step 2: Run ANALYZE with verbose output
ANALYZE VERBOSE products;
```

**Expected Output**:
```
INFO:  analyzing "public.products"
INFO:  "products": scanned 3000 of 3000 pages, 
       containing 10000 live rows and 0 dead rows; 
       30000 rows in sample, 10000 estimated total rows
```

**Why This Output Occurs**: `ANALYZE VERBOSE` reports the number of pages scanned, live/dead rows, and sample size. The statistics are stored in `pg_statistic` for the planner to use.

```sql
-- Step 3: Verify statistics were collected
SELECT attname, n_distinct, most_common_vals
FROM pg_stats
WHERE tablename = 'products';
-- Expected: Shows distinct value counts and most common values for each column.
```

### Real-World Cases

**Case 1: Ascending-Key Query Regression**
A reporting query on a `Sales` table with an `IDENTITY` primary key suddenly runs slowly. The statistics on the primary key are stale because the auto-update threshold was not reached quickly enough. A targeted `UPDATE STATISTICS Sales PK_Sales WITH FULLSCAN` resolves the plan regression.

**Case 2: Partitioned Table Incremental Statistics**
A partitioned `Transactions` table has 24 monthly partitions. Using `UPDATE STATISTICS ... WITH INCREMENTAL` updates only changed partitions, reducing maintenance time by 80% compared to full-table statistics updates.

**Case 3: PostgreSQL Autovacuum Tuning**
A high-churn PostgreSQL database experiences planner misestimates because autovacuum's analyze threshold is too high. The DBA lowers `autovacuum_analyze_scale_factor` and `autovacuum_analyze_threshold` to trigger more frequent analysis.

---

## Core Concept 2: Index Maintenance

### Definitions

**Core Definition**: Index maintenance involves detecting and repairing fragmentation, reclaiming dead space, and removing redundant indexes.

**Technical Definition**: Index maintenance encompasses two degradation phenomena: fragmentation (logical page order diverging from physical order on disk) and bloat (dead entries from MVCC row versions occupying index pages), addressed through rebuild, reorganize, vacuum, or reindex operations depending on the engine.

**Beginner-Friendly Explanation**: Indexes are like the index at the back of a book. Over time, as you add and remove pages, the index gets messy—entries point to the wrong places. Index maintenance tidies it up so the database can find things quickly again.

### Purposes

- **To** reduce I/O cost caused by fragmented or bloated index pages
- **To** reclaim storage occupied by dead index entries
- **To** remove redundant or unused indexes that impose write overhead
- **To** maintain index page density for efficient range scans

### Sub-Concept 2.1: Reindexing and Defragmentation

#### Syntax Rules and Structure (SQL Server)

```sql
-- Reorganize (online, minimal locking)
ALTER INDEX index_name ON table_name REORGANIZE;

-- Rebuild (offline by default, or online with Enterprise)
ALTER INDEX index_name ON table_name REBUILD 
    WITH (ONLINE = ON, FILLFACTOR = 90);
```

#### Syntax Rules and Structure (PostgreSQL)

```sql
-- Rebuild a single index (blocking)
REINDEX INDEX index_name;

-- Rebuild without locking writes (PG 12+)
REINDEX INDEX CONCURRENTLY index_name;

-- Rebuild all indexes on a table
REINDEX TABLE table_name;
```

#### Syntax Rules and Structure (MySQL)

```sql
-- Rebuild table and indexes
OPTIMIZE TABLE table_name;
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `REORGANIZE` | Defragments leaf level; online operation |
| `REBUILD` | Drops and recreates the index; can be online |
| `CONCURRENTLY` | PostgreSQL: avoids blocking writes during reindex |
| `ONLINE = ON` | SQL Server: keeps table available during rebuild |

#### Constraints and Limitations

- SQL Server: Rebuilding indexes does **not** update statistics for non-indexed columns
- SQL Server: Rebuild thresholds: recommended when fragmentation > 30%; reorganize when 10–30%
- PostgreSQL: `REINDEX` without `CONCURRENTLY` takes an exclusive lock on the table, blocking writes
- MySQL: `OPTIMIZE TABLE` rebuilds the table and may be slow for large tables

### Sub-Concept 2.2: Unused and Duplicate Indexes

#### Definitions

**Unused Index**: An index that has not been used by the query optimizer since the last statistics reset.

**Duplicate Index**: An index whose leading key columns are identical to or a prefix of another index, making it redundant.

#### Syntax Rules and Structure (PostgreSQL)

```sql
-- Identify unused indexes
SELECT schemaname, relname, indexrelname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;

-- Identify duplicate indexes by leading column
SELECT 
    indrelid::regclass AS table_name,
    array_agg(indexrelid::regclass) AS indexes
FROM pg_index
GROUP BY indrelid, indkey
HAVING count(*) > 1;
```

#### Syntax Rules and Structure (SQL Server)

```sql
-- Identify unused indexes
SELECT 
    OBJECT_NAME(i.object_id) AS table_name,
    i.name AS index_name,
    us.user_seeks, us.user_scans, us.user_lookups, us.user_updates
FROM sys.indexes i
LEFT JOIN sys.dm_db_index_usage_stats us
    ON i.object_id = us.object_id AND i.index_id = us.index_id
WHERE us.user_seeks = 0 AND us.user_scans = 0
  AND us.user_lookups = 0
  AND i.type > 1;
```

#### Constraints and Limitations

- PostgreSQL: Index usage statistics reset on server restart unless `pg_stat_statements` tracks usage persistently
- SQL Server: Usage stats reset on instance restart or index rebuild
- Dropping an index is irreversible without recreation; verify usage over a full business cycle before dropping

### Annotated Complete Step-by-Step Code Examples

#### Example 1: SQL Server Index Fragmentation Detection and Repair

**Setup**: A database with a fragmented index.

```sql
-- Step 1: Detect fragmentation
SELECT 
    OBJECT_NAME(ips.object_id) AS table_name,
    i.name AS index_name,
    ips.avg_fragmentation_in_percent,
    ips.page_count
FROM sys.dm_db_index_physical_stats(
    DB_ID(), NULL, NULL, NULL, 'LIMITED') ips
JOIN sys.indexes i ON ips.object_id = i.object_id 
    AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 10
ORDER BY ips.avg_fragmentation_in_percent DESC;

-- Expected Output:
-- table_name | index_name | avg_fragmentation_in_percent | page_count
-- Orders     | IX_Orders_Date | 42.5                       | 1200
```

```sql
-- Step 2: Rebuild the fragmented index
ALTER INDEX IX_Orders_Date ON Orders 
REBUILD WITH (ONLINE = ON, FILLFACTOR = 90);
-- Expected: Commands completed successfully.

-- Step 3: Verify fragmentation is reduced
-- Re-run Step 1 query; fragmentation should be near 0%.
```

**Why This Output Occurs**: `sys.dm_db_index_physical_stats` reports fragmentation. `ONLINE = ON` allows concurrent DML during rebuild. `FILLFACTOR = 90` leaves 10% space per page for future inserts, reducing future fragmentation.

#### Example 2: PostgreSQL Unused Index Detection and Cleanup

```sql
-- Step 1: Find unused indexes (scanned 0 times)
SELECT 
    schemaname,
    relname AS table_name,
    indexrelname AS index_name,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;

-- Expected Output:
-- schemaname | table_name | index_name          | index_size
-- public     | orders     | idx_orders_old_date | 2048 kB
```

```sql
-- Step 2: Drop the unused index (after verification)
DROP INDEX CONCURRENTLY idx_orders_old_date;
-- Expected: DROP INDEX
```

**Why This Output Occurs**: `pg_stat_user_indexes` tracks `idx_scan` (number of index scans). An `idx_scan = 0` indicates the index has not been used since statistics reset. `CONCURRENTLY` drops without locking writes.

### Real-World Cases

**Case 1: Reducing Write Amplification from Redundant Indexes**
A table has three indexes with the same leading column (`account_id`). Dropping two redundant indexes reduces INSERT overhead by 40% and frees 15 GB of storage.

**Case 2: Online Index Rebuild in SQL Server Enterprise**
A 500 GB table requires index maintenance. Using `ALTER INDEX ... REBUILD WITH (ONLINE = ON)`, the rebuild completes without blocking application queries.

**Case 3: PostgreSQL Concurrent Reindex**
A production database cannot tolerate exclusive locks. `REINDEX INDEX CONCURRENTLY` rebuilds a bloated index while writes continue, completing in 45 minutes with zero downtime.

---

## Core Concept 3: Table Maintenance

### Definitions

**Core Definition**: Table maintenance encompasses partitioning, compression, and archiving strategies that manage data at the table level.

**Technical Definition**: Table maintenance includes partition management (switching, splitting, merging partitions), compression application (row, page, or columnstore), and data archival (moving cold data to secondary storage or archive tables) to control storage footprint and query performance.

**Beginner-Friendly Explanation**: Table maintenance is like organizing a warehouse. You put old items in labeled boxes (partitions), compress bulky items (compression), and move things you rarely use to a separate storage room (archiving).

### Purposes

- **To** improve query performance through partition elimination and compression
- **To** reduce storage costs by compressing or archiving cold data
- **To** enable efficient data lifecycle management (rolling windows, retention)
- **To** simplify maintenance by isolating active and inactive data

### Sub-Concept 3.1: Partitioning Maintenance

#### Syntax Rules and Structure (SQL Server)

```sql
-- Switch a partition out to an archive table
ALTER TABLE Orders 
SWITCH PARTITION 24 TO OrdersArchive;

-- Split a partition to add a new boundary
ALTER PARTITION FUNCTION PF_OrderDate() 
SPLIT RANGE ('2027-01-01');

-- Merge partitions to consolidate
ALTER PARTITION FUNCTION PF_OrderDate() 
MERGE RANGE ('2026-01-01');
```

#### Syntax Rules and Structure (PostgreSQL)

```sql
-- Attach a new partition
ALTER TABLE orders ATTACH PARTITION orders_2027
FOR VALUES FROM ('2027-01-01') TO ('2028-01-01');

-- Detach an old partition
ALTER TABLE orders DETACH PARTITION orders_2025;
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `SWITCH PARTITION` | Moves a partition to another table (SQL Server) |
| `SPLIT RANGE` | Adds a boundary to a partition function |
| `ATTACH PARTITION` | Adds an existing table as a partition (PostgreSQL) |
| `DETACH PARTITION` | Removes a partition from the partitioned table |

#### Constraints and Limitations

- PostgreSQL does not support `SPLIT` and `EXCHANGE` of partitions as SQL Server does
- Partition switching requires matching filegroup and schema (SQL Server)
- Partition functions must be altered during low-activity windows

### Sub-Concept 3.2: Compression

#### Syntax Rules and Structure (SQL Server)

```sql
-- Row compression
ALTER TABLE Orders REBUILD WITH (DATA_COMPRESSION = ROW);

-- Page compression
ALTER TABLE Orders REBUILD WITH (DATA_COMPRESSION = PAGE);

-- Columnstore compression
ALTER INDEX CCI_Orders ON Orders REBUILD 
WITH (DATA_COMPRESSION = COLUMNSTORE);
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `ROW` | Fixed-width field compression |
| `PAGE` | Row compression plus dictionary prefix/suffix compression |
| `COLUMNSTORE` | Column-wise compression for analytics |

#### Constraints and Limitations

- Page compression adds CPU overhead; high-churn OLTP tables may see 10% write throughput reduction
- PostgreSQL uses TOAST for automatic compression of wide rows; explicit table compression is not available for heap tables
- Compression benefits are workload-dependent; always test under peak load

### Sub-Concept 3.3: Data Archiving

#### Syntax Rules and Structure (SQL Server)

```sql
-- Move old data to an archive table
INSERT INTO OrdersArchive
SELECT * FROM Orders WHERE OrderDate < '2025-01-01';

DELETE FROM Orders WHERE OrderDate < '2025-01-01';
-- Then shrink or rebuild as needed
```

#### Syntax Rules and Structure (PostgreSQL)

```sql
-- Detach old partition and move to archive schema
ALTER TABLE orders DETACH PARTITION orders_2025;
ALTER TABLE orders_2025 SET SCHEMA archive;
```

#### Constraints and Limitations

- Archiving must preserve referential integrity or explicitly handle orphaned rows
- Archive storage may need separate backup and retention policies
- Deletion of large row sets can cause bloat (PostgreSQL) or long transaction logs (SQL Server)

### Annotated Complete Step-by-Step Code Examples

#### Example 1: SQL Server Partition Switching and Compression

**Setup**: A partitioned `Orders` table with monthly partitions.

```sql
-- Step 1: Compress the active partition
ALTER TABLE Orders REBUILD PARTITION = 12 
WITH (DATA_COMPRESSION = PAGE);
-- Expected: Commands completed successfully.

-- Step 2: Switch the oldest partition to an archive table
ALTER TABLE Orders SWITCH PARTITION 1 TO OrdersArchive;
-- Expected: Commands completed successfully.

-- Step 3: Truncate the archive table (after verification)
TRUNCATE TABLE OrdersArchive;
-- Expected: Commands completed successfully.
```

**Why This Output Occurs**: Partition switching moves the entire partition as a metadata operation, avoiding row-by-row movement. Compression reduces the active partition's storage footprint by 50–70% for read-heavy data.

#### Example 2: PostgreSQL Partition Detachment and Archiving

```sql
-- Step 1: Detach the oldest partition (no data movement)
ALTER TABLE orders DETACH PARTITION orders_2025;
-- Expected: ALTER TABLE

-- Step 2: Move the detached table to an archive schema
ALTER TABLE orders_2025 SET SCHEMA archive;
-- Expected: ALTER TABLE

-- Step 3: Verify the main table no longer includes the old data
SELECT count(*) FROM orders WHERE order_date < '2026-01-01';
-- Expected: 0 (or only rows from remaining partitions)
```

**Why This Output Occurs**: `DETACH PARTITION` converts a partition into a standalone table without copying data. Moving it to an archive schema isolates it from application queries while retaining it for compliance.

### Real-World Cases

**Case 1: Rolling Window Partition Management**
A logging system maintains 24 monthly partitions. Each month, a new partition is attached and the oldest is detached and archived to object storage, keeping the active table size constant.

**Case 2: Columnstore Compression for Analytics**
A data warehouse applies columnstore compression to fact tables, reducing storage by 80% and accelerating analytical scans by 40%.

**Case 3: Archive Schema for Regulatory Retention**
A financial database detaches old transaction partitions into an `archive` schema that remains queryable for audits but is excluded from daily backups.

---

## Core Concept 4: Storage Monitoring

### Definitions

**Core Definition**: Storage monitoring tracks disk space consumption, growth trends, and I/O performance to prevent outages.

**Technical Definition**: Storage monitoring involves collecting metrics on data and log file sizes, disk free space, disk queue length, and growth rates, using performance counters (Windows PerfMon, `iostat`, `pg_stat_bgwriter`) and system catalogs (`sys.dm_os_volume_stats`, `pg_database_size`).

**Beginner-Friendly Explanation**: Storage monitoring is like checking the fuel gauge in your car. You want to know how much space is left, how fast you're using it, and whether the engine (disk) is working too hard.

### Purposes

- **To** prevent disk-full errors that cause database outages
- **To** forecast when additional storage will be required
- **To** identify I/O bottlenecks before they degrade query performance
- **To** trigger alerts when free space falls below safe thresholds

### Sub-Concept 4.1: Disk Space Alerts

#### Syntax Rules and Structure (SQL Server)

```sql
SELECT 
    DB_NAME(database_id) AS db_name,
    name AS file_name,
    size * 8 / 1024 AS size_mb,
    physical_name
FROM sys.master_files;

SELECT 
    volume_mount_point,
    total_bytes / 1073741824 AS total_gb,
    available_bytes / 1073741824 AS free_gb,
    (available_bytes * 100.0 / total_bytes) AS pct_free
FROM sys.dm_os_volume_stats(NULL, NULL);
```

#### Syntax Rules and Structure (PostgreSQL)

```sql
SELECT 
    datname AS database_name,
    pg_size_pretty(pg_database_size(datname)) AS size
FROM pg_database
ORDER BY pg_database_size(datname) DESC;

SELECT 
    pg_size_pretty(sum(size)) AS total_wal_size
FROM pg_ls_waldir();
```

#### Constraints and Limitations

- Monitoring must account for autogrowth increments (SQL Server) or WAL segment recycling (PostgreSQL)
- Thresholds should be set based on growth rate and recovery time, not just percentage

### Sub-Concept 4.2: Growth Forecasting

#### Definitions

**Growth Rate**: The average increase in database size per day or week, used to predict future storage requirements.

#### Syntax Rules and Structure (SQL Server)

```sql
-- Capture baseline sizes weekly
SELECT 
    DB_NAME(database_id) AS db_name,
    SUM(size) * 8 / 1024 AS total_mb,
    GETDATE() AS captured_at
FROM sys.master_files
GROUP BY database_id;
```

#### Constraints and Limitations

- Growth rate is non-linear; seasonal spikes must be accounted for
- Archiving and compression change the growth trajectory

### Sub-Concept 4.3: I/O Bottleneck Analysis

#### Syntax Rules and Structure (Linux)

```bash
# Monitor disk utilization and queue length
iostat -x 5

# Key columns:
# %util: >80% indicates saturation
# await: average wait time (ms) — >20ms for SSDs is high
# aqu-sz: average queue size — >2 per disk may indicate bottleneck
```

#### Syntax Rules and Structure (SQL Server)

```sql
SELECT 
    DB_NAME(database_id) AS db_name,
    file_id,
    io_stall_read_ms / num_of_reads AS avg_read_stall_ms,
    io_stall_write_ms / num_of_writes AS avg_write_stall_ms
FROM sys.dm_io_virtual_file_stats(NULL, NULL)
WHERE num_of_reads > 0 OR num_of_writes > 0;
```

#### Constraints and Limitations

- I/O bottlenecks can be masked by caching; sustained high `%util` is the key signal
- Cloud storage (EBS, Azure Disk) has different baseline IOPS than local SSD

### Annotated Complete Step-by-Step Code Examples

#### Example 1: SQL Server Disk Space and Growth Monitoring

```sql
-- Step 1: Check current disk free space
SELECT 
    volume_mount_point,
    total_bytes / 1073741824 AS total_gb,
    available_bytes / 1073741824 AS free_gb,
    CAST(available_bytes * 100.0 / total_bytes AS DECIMAL(5,2)) AS pct_free
FROM sys.dm_os_volume_stats(NULL, NULL);

-- Expected Output:
-- volume_mount_point | total_gb | free_gb | pct_free
-- C:\                | 500      | 150     | 30.00
```

```sql
-- Step 2: Capture database sizes for growth tracking
SELECT 
    DB_NAME(database_id) AS db_name,
    SUM(size * 8.0 / 1024) AS total_mb,
    GETDATE() AS captured_at
FROM sys.master_files
GROUP BY database_id;
-- Store results in a monitoring table for trend analysis.
```

**Why This Output Occurs**: `sys.dm_os_volume_stats` reports volume-level free space. Storing periodic snapshots enables linear regression to forecast when free space will reach critical levels.

#### Example 2: PostgreSQL Database Size and WAL Monitoring

```sql
-- Step 1: Check database sizes
SELECT 
    datname,
    pg_size_pretty(pg_database_size(datname)) AS size
FROM pg_database
ORDER BY pg_database_size(datname) DESC;

-- Step 2: Check WAL directory size
SELECT 
    count(*) AS wal_file_count,
    pg_size_pretty(sum(size)) AS total_wal_size
FROM pg_ls_waldir();

-- Expected Output:
-- wal_file_count | total_wal_size
-- 48             | 768 MB
```

**Why This Output Occurs**: `pg_ls_waldir()` lists WAL segment files. Unusually high WAL count can indicate archiving failure or replication lag blocking WAL recycling.

### Real-World Cases

**Case 1: Preventing Disk-Full Outage**
A monitoring alert fires when free space drops below 15%. The DBA adds storage and adjusts autogrowth settings to prevent a production outage during peak hours.

**Case 2: Forecasting Storage Purchase**
Weekly database size snapshots over 6 months show a growth rate of 2 GB/day. The DBA forecasts that current storage will be exhausted in 45 days and initiates procurement.

**Case 3: I/O Bottleneck Diagnosis**
`iostat` shows `%util` consistently above 90% on the data disk. Investigation reveals missing indexes causing full table scans, which are resolved by adding an index and reducing I/O load.

---

## Core Concept 5: Log Management

### Definitions

**Core Definition**: Log management controls the growth, truncation, and retention of database transaction logs and error logs.

**Technical Definition**: Log management encompasses transaction log truncation (SQL Server, where the log is a circular file and truncation frees inactive VLFs) and WAL archive cleanup (PostgreSQL, where `pg_archivecleanup` removes archived segments preceding the oldest needed WAL file).

**Beginner-Friendly Explanation**: Logs are like a diary the database keeps of every change. If you never clean the diary, it fills up the shelf. Log management decides what to keep, what to archive, and what to recycle.

### Purposes

- **To** prevent transaction log or WAL directory from filling disk
- **To** maintain log truncation so space is reused within the log file
- **To** archive logs for point-in-time recovery and replication
- **To** rotate error logs for manageable troubleshooting

### Sub-Concept 5.1: SQL Server Transaction Log Truncation

#### Syntax Rules and Structure

```sql
-- Log truncation occurs automatically after:
-- 1. A checkpoint in SIMPLE recovery model
-- 2. A log backup in FULL/BULK_LOGGED recovery model

-- Manual log backup (triggers truncation)
BACKUP LOG database_name
TO DISK = 'path\backup.trn'
WITH INIT;

-- Shrink log file (use sparingly)
DBCC SHRINKFILE (database_name_log, target_size_mb);
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `BACKUP LOG` | Backs up active log; truncates inactive portion |
| `DBCC SHRINKFILE` | Reduces physical log file size |

#### Constraints and Limitations

- Log truncation does **not** reduce physical file size; it marks space as reusable
- A long-running transaction or replication delay can prevent truncation, causing log growth
- `DBCC SHRINKFILE` should not be routine; it causes fragmentation

### Sub-Concept 5.2: PostgreSQL WAL Archive Cleanup

#### Syntax Rules and Structure

```bash
# Standalone cleanup: remove WAL files older than a given segment
pg_archivecleanup /mnt/archive 000000010000003700000010

# As a standby cleanup command in postgresql.conf:
archive_cleanup_command = 'pg_archivecleanup /mnt/standby/archive %r 2>>cleanup.log'
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `archivelocation` | Directory containing archived WAL files |
| `oldestkeptwalfile` | WAL file to keep; all logically preceding files are removed |

#### Constraints and Limitations

- `pg_archivecleanup` removes files **logically preceding** the specified file; it preserves crash-restart capability
- Not appropriate when archive location serves multiple standbys or long-term retention

### Sub-Concept 5.3: Log Rotation (Error Logs)

#### Syntax Rules and Structure (SQL Server)

```sql
-- Cycle the SQL Server error log
EXEC sp_cycle_errorlog;

-- Cycle the SQL Server Agent error log
EXEC sp_cycle_agent_errorlog;
```

#### Syntax Rules and Structure (Linux)

```bash
# logrotate configuration for PostgreSQL
/var/log/postgresql/*.log {
    weekly
    rotate 8
    compress
    delaycompress
    missingok
    notifempty
    create 640 postgres postgres
}
```

#### Constraints and Limitations

- Error log cycling preserves the previous log as `.1`, `.2`, etc.; limit the number of retained files
- Log rotation must not interfere with monitoring agents that read logs

### Annotated Complete Step-by-Step Code Examples

#### Example 1: SQL Server Log Backup and Truncation

```sql
-- Step 1: Check log space usage
DBCC SQLPERF(LOGSPACE);
-- Expected Output shows database, log size, and % used.

-- Step 2: Perform a log backup (triggers truncation)
BACKUP LOG TestFullBackup
TO DISK = 'C:\Backups\TestFullBackup_Log.trn'
WITH INIT, STATS = 25;
-- Expected Output:
-- 25 percent processed.
-- 50 percent processed.
-- 75 percent processed.
-- 100 percent processed.
-- Processed 5 pages for database 'TestFullBackup', file 'TestFullBackup_log' on file 1.
-- BACKUP LOG successfully processed 5 pages in 0.031 seconds (1.250 MB/sec).

-- Step 3: Verify truncation
DBCC SQLPERF(LOGSPACE);
-- Log space used should decrease.
```

**Why This Output Occurs**: `BACKUP LOG` copies the active log to the backup file, then marks the inactive portion as reusable. `DBCC SQLPERF(LOGSPACE)` shows the percentage of log space used, which drops after truncation.

#### Example 2: PostgreSQL WAL Archive Cleanup

```bash
# Step 1: List archived WAL files
ls /mnt/archive/ | head -5
# Expected: 00000001000000370000000A through 000000010000003700000010

# Step 2: Remove files logically preceding 000000010000003700000010
pg_archivecleanup -d /mnt/archive 000000010000003700000010
# Expected Output:
# pg_archivecleanup: keep WAL file "/mnt/archive/000000010000003700000010" and later
# pg_archivecleanup: removing file "/mnt/archive/00000001000000370000000F"
# pg_archivecleanup: removing file "/mnt/archive/00000001000000370000000E"
# ...

# Step 3: Verify remaining files
ls /mnt/archive/ | head -3
# Expected: Starts from 000000010000003700000010
```

**Why This Output Occurs**: `pg_archivecleanup` parses the WAL file name to determine ordering and deletes all files that sort before the specified `oldestkeptwalfile`, preserving the specified file and all later segments.

### Real-World Cases

**Case 1: Log Growth from Long-Running Transaction**
A batch process runs for 8 hours without committing, preventing log truncation. The log grows to fill the disk. Resolving requires committing the transaction in batches or adding log space temporarily.

**Case 2: WAL Archive Cleanup on Standby**
A PostgreSQL standby accumulates WAL archives because cleanup was not configured. Configuring `archive_cleanup_command` with `pg_archivecleanup` removes obsolete segments automatically, reclaiming 200 GB.

**Case 3: Error Log Rotation for Compliance**
A regulated environment requires 90 days of error log retention. `sp_cycle_errorlog` is scheduled weekly, and log files are archived to compliance storage before deletion.

---

## Core Concept 6: Bloat Management

### Definitions

**Core Definition**: Bloat is the accumulation of dead row versions and empty space in PostgreSQL tables and indexes due to MVCC.

**Technical Definition**: In PostgreSQL's MVCC model, `UPDATE` and `DELETE` operations create new row versions and mark old versions as dead tuples; without vacuuming, these dead tuples accumulate as bloat, wasting storage and degrading query performance.

**Beginner-Friendly Explanation**: Imagine writing on a whiteboard and erasing. If you never clean the eraser residue, the board gets gray and hard to read. Bloat is that residue; vacuuming is cleaning the board.

### Purposes

- **To** reclaim storage occupied by dead tuples
- **To** prevent transaction ID wraparound (a critical failure condition)
- **To** maintain accurate statistics for the query planner
- **To** reduce I/O by compacting table pages

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL VACUUM)

```sql
VACUUM [ ( option [, ...] ) ] [ table_name [ ( column_name [, ...] ) ] ];

-- Options:
-- FULL, FREEZE, VERBOSE, ANALYZE, DISABLE_PAGE_SKIPPING
```

#### Complete General Syntax (Autovacuum Configuration)

```conf
# postgresql.conf
autovacuum = on
autovacuum_max_workers = 3
autovacuum_naptime = 1min
autovacuum_vacuum_threshold = 50
autovacuum_vacuum_scale_factor = 0.2
autovacuum_analyze_threshold = 50
autovacuum_analyze_scale_factor = 0.1
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `VACUUM` | Reclaims dead tuple space for reuse |
| `VACUUM FULL` | Rewrites table, returning space to OS (exclusive lock) |
| `ANALYZE` | Updates statistics |
| `autovacuum_vacuum_scale_factor` | Fraction of table size that triggers vacuum |

#### Syntax Rules

- `VACUUM` (without `FULL`) takes a `SHARE UPDATE EXCLUSIVE` lock and does not block reads or writes
- `VACUUM FULL` takes an `ACCESS EXCLUSIVE` lock and blocks all access
- Autovacuum runs automatically when the number of dead tuples exceeds `threshold + scale_factor * table_size`

#### Constraints and Limitations

- `VACUUM FULL` requires disk space equal to the table size and is not online
- Autovacuum may not keep up with very high churn; manual tuning or `pg_repack` may be needed
- Transaction ID wraparound requires freezing old tuples; autovacuum handles this but can be blocked by long-running transactions

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL Bloat Detection and VACUUM

**Setup**: A high-churn `orders` table.

```sql
-- Step 1: Detect bloat
SELECT 
    relname AS table_name,
    n_live_tup,
    n_dead_tup,
    ROUND(n_dead_tup * 100.0 / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_pct
FROM pg_stat_user_tables
WHERE n_dead_tup > 0
ORDER BY n_dead_tup DESC;

-- Expected Output:
-- table_name | n_live_tup | n_dead_tup | dead_pct
-- orders     | 5000000    | 1500000    | 23.08
```

```sql
-- Step 2: Run VACUUM ANALYZE
VACUUM (VERBOSE, ANALYZE) orders;
-- Expected Output:
-- INFO:  vacuuming "public.orders"
-- INFO:  "orders": removed 1500000 dead row versions in 25000 pages
-- INFO:  "orders": found 1500000 removable, 5000000 nonremovable row versions
-- INFO:  analyzing "public.orders"
```

**Why This Output Occurs**: `VACUUM` scans the table, removes dead tuples, and marks space as reusable. `VERBOSE` reports the number removed. `ANALYZE` updates statistics in the same pass.

#### Example 2: VACUUM FULL and pg_repack

```sql
-- Step 1: VACUUM FULL (requires exclusive lock)
VACUUM FULL orders;
-- Expected: VACUUM
-- This rewrites the table, returning space to the OS.
```

```bash
# Step 2: Alternative using pg_repack (online)
pg_repack -U postgres -d mydb -t orders
# Expected Output:
# INFO: repacking table "public.orders"
# INFO: repacked table "public.orders" in 45 seconds
```

**Why This Output Occurs**: `VACUUM FULL` rewrites the entire table into a new file, eliminating bloat but blocking all access. `pg_repack` performs the same reorganization online using triggers to capture changes during the rewrite.

### Real-World Cases

**Case 1: Autovacuum Tuning for High-Churn Tables**
A table with 50 million rows and 10% daily churn experiences bloat because the default `autovacuum_vacuum_scale_factor = 0.2` triggers too late. Lowering it to 0.05 for that table increases vacuum frequency and keeps bloat under 10%.

**Case 2: Transaction Wraparound Prevention**
A database approaches `autovacuum_freeze_max_age`. Autovacuum launches an aggressive freeze, preventing wraparound shutdown. Monitoring `age(datfrozenxid)` is critical.

**Case 3: pg_repack for Zero-Downtime Bloat Removal**
A 200 GB table needs bloat removal but cannot be locked. `pg_repack` reorganizes the table online, reducing size by 60% with no application downtime.

---

## Core Concept 7: Health & Performance Telemetry

### Definitions

**Core Definition**: Health telemetry is the collection of metrics, logs, and probes that indicate database operational state.

**Technical Definition**: Health telemetry encompasses liveness probes (is the process running?), readiness probes (is the database ready to accept connections?), error log scraping (parsing log files for critical events), and metrics export (Prometheus-style counters and gauges) for alerting and dashboards.

**Beginner-Friendly Explanation**: Health telemetry is like a hospital monitor for your database. It shows the heartbeat (liveness), whether the patient is ready for visitors (readiness), and alerts the nurse when something goes wrong (error log alerts).

### Purposes

- **To** detect database unavailability and trigger failover
- **To** alert on critical errors (corruption, disk full, replication lag)
- **To** provide dashboards for proactive performance management
- **To** satisfy compliance and audit requirements for operational monitoring

### Sub-Concept 7.1: Liveness and Readiness Probes

#### Syntax Rules and Structure (Kubernetes)

```yaml
livenessProbe:
  exec:
    command: ["pg_isready", "-h", "localhost", "-p", "5432"]
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  exec:
    command: ["psql", "-h", "localhost", "-U", "postgres", "-c", "SELECT 1"]
  initialDelaySeconds: 5
  periodSeconds: 5
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `pg_isready` | PostgreSQL utility that checks connection readiness |
| `SELECT 1` | Minimal query to verify database accepts queries |

#### Constraints and Limitations

- Liveness probes must be lightweight to avoid overloading the database
- Readiness probes should verify actual query capability, not just TCP connectivity

### Sub-Concept 7.2: Error Log Scraping

#### Syntax Rules and Structure (SQL Server)

```sql
-- Read the current error log
EXEC sp_readerrorlog;

-- Filter for severity 19+ errors
EXEC sp_readerrorlog 0, 1, N'Error: 823';
EXEC sp_readerrorlog 0, 1, N'Error: 824';
EXEC sp_readerrorlog 0, 1, N'Error: 825';
```

#### Syntax Rules and Structure (Linux)

```bash
# Tail PostgreSQL error log
tail -f /var/log/postgresql/postgresql-14-main.log

# Use journalctl for systemd-managed PostgreSQL
journalctl -u postgresql -f
```

#### Constraints and Limitations

- Log scraping must handle log rotation and avoid re-alerting on already-seen lines
- Regex-based parsing should be tested against sample log lines

### Sub-Concept 7.3: Alerting

#### Syntax Rules and Structure (Prometheus Alertmanager)

```yaml
groups:
- name: database
  rules:
  - alert: PostgreSQLDown
    expr: pg_up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "PostgreSQL instance {{ $labels.instance }} is down"

  - alert: HighDeadTuples
    expr: pg_stat_user_tables_n_dead_tup > 1000000
    for: 5m
    labels:
      severity: warning
```

#### Constraints and Limitations

- Alert thresholds must be tuned to avoid alert fatigue
- Alerts should include runbook links and severity levels

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL Monitoring with pg_stat_statements

```sql
-- Step 1: Enable pg_stat_statements
ALTER SYSTEM SET shared_preload_libraries = 'pg_stat_statements';
-- Restart required.
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Step 2: Query top queries by total execution time
SELECT 
    queryid,
    LEFT(query, 80) AS query_preview,
    calls,
    ROUND(total_exec_time::numeric / 1000, 2) AS total_seconds,
    ROUND(mean_exec_time::numeric, 2) AS mean_ms
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

**Expected Output**:
```
 queryid | query_preview                                    | calls | total_seconds | mean_ms
---------|--------------------------------------------------|-------|---------------|---------
 123456  | SELECT * FROM orders WHERE customer_id = $1      | 50000 | 1200.45       | 24.01
 789012  | UPDATE products SET price = $1 WHERE id = $2     | 20000 | 800.12        | 40.01
```

**Why This Output Occurs**: `pg_stat_statements` tracks execution statistics for all SQL statements. `total_exec_time` identifies the most resource-intensive queries for optimization.

#### Example 2: SQL Server Error Log Alerting

```sql
-- Step 1: Read error log for corruption indicators
EXEC sp_readerrorlog 0, 1, N'Error: 823';
-- Expected: Returns any "Error: 823" entries (I/O errors).

-- Step 2: Check suspect pages
SELECT * FROM msdb.dbo.suspect_pages;
-- Expected: Lists pages with checksum or torn-page errors.
```

**Expected Output** (if no errors):
```
(0 rows affected)
```

**Why This Output Occurs**: `sp_readerrorlog` filters the SQL Server error log for specific error numbers. `suspect_pages` tracks pages flagged during checksum validation. Regular monitoring of these sources enables early detection of storage corruption.

### Real-World Cases

**Case 1: Kubernetes Liveness Probe Prevents Stuck Pod**
A PostgreSQL pod becomes unresponsive due to a hung backend. The liveness probe (`pg_isready`) fails after 30 seconds, and Kubernetes restarts the pod automatically.

**Case 2: Error Log Alerting for Disk Failure**
A monitoring system scrapes the SQL Server error log and alerts on "Error: 825" (read retry succeeded). The DBA investigates and finds a failing disk before it causes data loss.

**Case 3: pg_stat_statements Identifies Regression**
After a deployment, `pg_stat_statements` shows a 10x increase in `mean_exec_time` for a critical query. The team rolls back the change and investigates the missing index.

---

## References

| Name | Link |
|------|------|
| Microsoft Learn — SQL Server Statistics | https://learn.microsoft.com/en-us/sql/relational-databases/statistics/statistics |
| PostgreSQL Documentation — ANALYZE | https://www.postgresql.org/docs/current/sql-analyze.html |
| AWS — SQL Server to Aurora PostgreSQL Index Maintenance | https://docs.aws.amazon.com/dms/latest/sql-server-to-aurora-postgresql-migration-playbook/ |
| SQL Engineering Handbook — Index Maintenance | https://github.com/theammarngp-makes/SQL-Engineering-Handbook/blob/main/15_INDEXES/10_INDEX_MAINTENANCE.md |
| AWS — Partitioning Databases | https://docs.aws.amazon.com/dms/latest/sql-server-to-aurora-postgresql-migration-playbook/chap-sql-server-aurora-pg.storage.partitioning.html |
| NovaDBA — Why Database Compression Can Improve Performance | https://novadba.com/why-database-compression-can-improve-performance-and-reduce-costs/ |
| Syteca — Database Management | https://www.syteca.com/docs/administration/deployment/database-management |
| Microsoft Learn — Storage and SQL Server Capacity Planning | https://learn.microsoft.com/en-us/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration |
| Microsoft Learn — SQL Server Transaction Log Architecture | https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-log-architecture-and-management-guide |
| PostgreSQL Documentation — pg_archivecleanup | https://www.postgresql.org/docs/current/pgarchivecleanup.html |
| AWS Prescriptive Guidance — PostgreSQL Maintenance on RDS and Aurora | https://docs.aws.amazon.com/prescriptive-guidance/latest/postgresql-maintenance-rds-aurora/ |
| Ultipa — Monitoring Operations | https://www.ultipa.com/docs/operations/monitoring |
| Pigsty — MySQL Monitoring | https://doc.pigsty.io/docs/mysql/monitor/ |
| IDERA — Configure Text and Expression Alerts | https://wiki.idera.com/spaces/SQLDM132/pages/12841976788/Configure+text+and+expression+alerts |
| AI-Agents Public — Database Monitoring and Alerting Patterns | https://github.com/vasilyu1983/AI-Agents-public/blob/main/frameworks/shared-skills/skills/data-sql-optimization/references/monitoring-alerting-patterns.md |