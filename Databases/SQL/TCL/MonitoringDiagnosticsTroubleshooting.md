# SQL Monitoring, Diagnostics & Troubleshooting: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL monitoring, diagnostics, and troubleshooting is the practice of observing database activity, identifying performance bottlenecks and concurrency conflicts, and resolving issues that degrade system performance or availability.

**Technical Definition:** Database monitoring encompasses the collection and analysis of runtime metrics (locks, waits, transactions, resource usage) through system views, dynamic management views (DMVs), performance schemas, and logging facilities. Diagnostics involves correlating these metrics to identify root causes—such as blocking chains, deadlocks, and resource contention—while troubleshooting applies corrective actions (query tuning, lock management, configuration changes) to restore optimal performance .

**Beginner-Friendly Explanation:** Think of your database like a busy highway. Monitoring is watching the traffic cameras. Diagnostics is figuring out why there's a traffic jam—is it an accident, roadwork, or too many cars? Troubleshooting is clearing the jam. This cheat sheet shows you how to see what's happening, understand why, and fix it.

### Key Characteristics

- **Multi-layered observability:** Different RDBMSs expose diagnostics through system catalogs, DMVs, performance schemas, and log files.
- **Blocking is normal, deadlocks are not:** Blocking occurs when one transaction waits for another; deadlocks occur when waits become circular and must be resolved by aborting a transaction .
- **Lock escalation can cause unexpected blocking:** When a transaction acquires too many fine-grained locks, the engine may escalate to a table lock, blocking all concurrent access .
- **Deadlock information is ephemeral:** Most engines capture only the latest deadlock in memory; persistent logging requires explicit configuration .
- **Resource overhead matters:** Monitoring tools themselves consume resources; tuning thresholds (e.g., `deadlock_timeout`, `log_min_duration_statement`) is essential .

### Prerequisites

- **Transaction and locking fundamentals:** Understanding of lock modes, isolation levels, and blocking.
- **System view familiarity:** Knowledge of querying system catalogs and DMVs.
- **Database permissions:** Access to system views and server-level DMVs (often requires elevated privileges).

### Related Programming Areas

- **Database Administration:** Daily monitoring, performance tuning, and incident response.
- **Site Reliability Engineering:** Deadlock analysis, retry logic design, and capacity planning.
- **Application Development:** Understanding how application queries contribute to blocking.

### Core Concepts / Features

1. Identifying Blockers
2. Analyzing Block Chains
3. Deadlock Logging & Analysis
4. Under-the-Hood Sizing

---

## 1. Identifying Blockers

### Definitions

**Core Definition:** Identifying blockers involves querying system views to discover which sessions are holding locks and which sessions are waiting for those locks.

**Technical Definition:** Blocking occurs when one transaction holds a lock on a resource that another transaction needs. The waiting transaction is "blocked" by the holding transaction. System views such as `pg_locks` (PostgreSQL), `sys.dm_tran_locks` (SQL Server), and `INFORMATION_SCHEMA.INNODB_LOCKS` (MySQL) expose the current lock state, allowing DBAs to identify blocked and blocking sessions .

**Beginner-Friendly Explanation:** Blocking is like two cars trying to merge into the same lane. One has to wait for the other. To find out who's blocking whom, you check the database's "traffic report"—the system views that list who holds locks and who's waiting.

### Purposes

- To discover which sessions are currently holding locks.
- To identify which sessions are waiting for locks and why.
- To locate the "lead blocker" at the head of a blocking chain.
- To gather evidence for terminating blocking sessions or tuning queries.

### Syntax Rules and Structure

**PostgreSQL Blocking Query:**

```sql
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_statement,
    blocking_activity.query AS blocking_statement,
    now() - blocked_activity.query_start AS blocked_duration
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity 
    ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
JOIN pg_catalog.pg_stat_activity blocking_activity 
    ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

**SQL Server Blocking Query:**

```sql
SELECT dtl.request_session_id AS waitSID,
       der.blocking_session_id AS blockSID,
       dowt.resource_description,
       der.wait_type,
       dowt.wait_duration_ms,
       DB_NAME(dtl.resource_database_id) AS DB,
       dtl.resource_associated_entity_id AS waitingAssociatedEntity,
       dest.[text] AS waitSQL,
       dest1.[text] AS blockingSQL
FROM sys.dm_tran_locks dtl
JOIN sys.dm_os_waiting_tasks dowt 
    ON dowt.resource_address = dtl.lock_owner_address
JOIN sys.dm_exec_requests der 
    ON der.session_id = dtl.request_session_id
CROSS APPLY sys.dm_exec_sql_text(der.sql_handle) dest
LEFT JOIN sys.dm_exec_requests der1 
    ON der.session_id = dowt.blocking_session_id
OUTER APPLY sys.dm_exec_sql_text(der1.sql_handle) dest1
LEFT JOIN sys.dm_tran_locks dtl1 
    ON dtl1.request_session_id = der1.session_id;
```

**MySQL Lock Wait Query:**

```sql
SELECT 
    r.trx_id AS waiting_trx,
    r.trx_mysql_thread_id AS waiting_thread,
    r.trx_query AS waiting_query,
    b.trx_id AS blocking_trx,
    b.trx_mysql_thread_id AS blocking_thread,
    b.trx_query AS blocking_query
FROM information_schema.innodb_lock_waits w
JOIN information_schema.innodb_trx r ON r.trx_id = w.requesting_trx_id
JOIN information_schema.innodb_trx b ON b.trx_id = w.blocking_trx_id;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `pg_locks` / `sys.dm_tran_locks` / `innodb_locks` | System view showing current locks |
| `pg_stat_activity` / `sys.dm_exec_requests` | Session information and active queries |
| `blocked_locks.granted = FALSE` | Identifies waiting (ungranted) locks |
| `blocking_session_id` | Session ID of the blocking transaction |

**Syntax Rules:**

- In PostgreSQL, `pg_locks` shows one row per lock held or awaited .
- In SQL Server, `sys.dm_tran_locks` requires `VIEW SERVER STATE` permission.
- In MySQL, `INNODB_LOCKS` and `INNODB_LOCK_WAITS` tables are deprecated in MySQL 8.0 in favor of `performance_schema.data_locks` and `data_lock_waits`.

**Constraints and Limitations:**

- System views show current state; blocking may resolve before you act.
- Requires elevated permissions to view all sessions.
- High-frequency polling of system views can add overhead.

### Annotated Complete Code Examples

**Example 1: PostgreSQL Blocking Detection**

```sql
-- Query to find blocked and blocking sessions
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_statement,
    blocking_activity.query AS blocking_statement,
    now() - blocked_activity.query_start AS blocked_duration
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity 
    ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
JOIN pg_catalog.pg_stat_activity blocking_activity 
    ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;

-- Expected Output:
--  blocked_pid | blocked_user | blocking_pid | blocking_user | blocked_statement           | blocking_statement          | blocked_duration
-- -------------+--------------+--------------+---------------+-----------------------------+-----------------------------+------------------
--          123 | app_user     |          456 | batch_user    | UPDATE accounts SET...      | UPDATE accounts SET...      | 00:02:15.123
```

**Why this output occurs:** The `blocked_locks.granted = FALSE` condition identifies sessions waiting for locks. The join with `pg_stat_activity` provides user and query context. The blocking session (456) holds a lock that the blocked session (123) needs .

### Real-World Cases

**Case 1: Application Timeout**

An application reports query timeouts. The DBA runs the blocking query and discovers that a long-running batch job is holding a table lock, blocking all application writes. The batch job is killed or rescheduled.

**Case 2: Lock Escalation Investigation**

A query that normally runs in milliseconds suddenly takes seconds. The DBA checks `sys.dm_tran_locks` and discovers that a bulk update escalated to a table lock, blocking all reads .

### References

- AWS re:Post — Improve long-running queries in Amazon RDS PostgreSQL - https://repost.aws/knowledge-center/improve-long-running-postgresql-queries
- Alibaba Cloud — 如何快速解决RDS SQL Server的阻塞问题 - https://www.alibabacloud.com/help/zh/rds/apsaradb-rds-for-sql-server/resolve-the-blocking-issues-on-an-apsaradb-rds-for-sql-server-instance
- GitHub — How to Analyze Deadlocks with SHOW ENGINE INNODB STATUS in MySQL - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-how-to-analyze-deadlocks-with-show-engine-innodb-status-in-m/README.md
- Supabase Docs — How to check if my queries are being blocked by other queries? - https://supabase.com/docs/guides/troubleshooting/how-to-check-if-my-queries-are-being-blocked-by-other-queries-NSKtR1
- Microsoft Learn — Descripción y resolución de problemas de bloqueo - https://learn.microsoft.com/es-es/troubleshoot/sql/database-engine/performance/understand-resolve-blocking

---

## 2. Analyzing Block Chains

### Definitions

**Core Definition:** A block chain is a sequence of sessions where each session is blocked by the previous one, forming a dependency chain that ultimately traces back to a "lead blocker" at the head.

**Technical Definition:** When multiple transactions compete for locks, blocking can cascade: Session A blocks Session B, which blocks Session C, and so on. The lead blocker is the session at the head of the chain that is not blocked by anyone else. Identifying the lead blocker is critical because terminating or tuning that session resolves the entire chain .

**Beginner-Friendly Explanation:** Imagine a line of people waiting for a restroom. The person at the front is taking forever, and everyone behind them is stuck. The "lead blocker" is the person at the front—fixing their problem clears the whole line.

### Purposes

- To identify the root cause of widespread blocking.
- To determine which session to terminate or tune to resolve cascading waits.
- To understand the dependency structure of concurrent transactions.
- To quantify the impact of blocking (how many sessions are affected).

### Syntax Rules and Structure

**PostgreSQL Blocking Chain with `pg_blocking_pids()`:**

```sql
SELECT 
    p.pid,
    p.usename,
    p.state,
    p.wait_event_type || ': ' || p.wait_event AS wait_event,
    current_timestamp - p.state_change AS time_in_state,
    current_timestamp - p.xact_start AS time_in_xact,
    l.relation::regclass AS relname,
    l.locktype,
    l.mode,
    pg_blocking_pids(l.pid) AS blocking_pids
FROM pg_stat_activity p
LEFT OUTER JOIN pg_locks l ON (p.pid = l.pid AND NOT l.granted)
WHERE p.pid IN (
    SELECT unnest(pg_blocking_pids(pid)) FROM pg_stat_activity
)
ORDER BY time_in_xact DESC;
```

**SQL Server Block Chain Analysis:**

```sql
SELECT 
    er.session_id AS blocked_session,
    er.blocking_session_id AS blocking_session,
    er.wait_type,
    er.wait_time,
    er.wait_resource,
    es.status,
    es.open_transaction_count,
    dest.text AS blocked_query
FROM sys.dm_exec_requests er
JOIN sys.dm_exec_sessions es ON er.session_id = es.session_id
CROSS APPLY sys.dm_exec_sql_text(er.sql_handle) dest
WHERE er.blocking_session_id <> 0
ORDER BY er.wait_time DESC;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `pg_blocking_pids(pid)` | Returns array of PIDs blocking the given PID |
| `blocking_session_id` | SQL Server column identifying the blocker |
| `wait_time` / `wait_duration_ms` | How long the session has been waiting |
| `open_transaction_count` | Indicates whether the blocker has an open transaction |

**Syntax Rules:**

- In PostgreSQL, `pg_blocking_pids()` returns an array; the last element is the "lead blocker" .
- In SQL Server, `blocking_session_id = 0` means the session is not blocked.
- A session with `open_transaction_count > 0` but status "sleeping" may be holding locks from a previous transaction .

**Constraints and Limitations:**

- Block chains can be deep; recursive queries may be needed to trace fully.
- The lead blocker may not be the session holding the most locks.
- Terminating the lead blocker may cause its transaction to roll back, which can be expensive.

### Annotated Complete Code Examples

**Example 1: SQL Server Block Chain with Lead Blocker Identification**

```sql
-- Identify all blocked sessions and their blockers
SELECT 
    blocked.session_id AS blocked_session,
    blocked.blocking_session_id AS blocking_session,
    blocked.wait_type,
    blocked.wait_time,
    blocker.status AS blocker_status,
    blocker.open_transaction_count AS blocker_open_tran,
    blocker_text.text AS blocker_query
FROM sys.dm_exec_requests blocked
JOIN sys.dm_exec_sessions blocker 
    ON blocked.blocking_session_id = blocker.session_id
CROSS APPLY sys.dm_exec_sql_text(blocker.most_recent_sql_handle) blocker_text
WHERE blocked.blocking_session_id <> 0;

-- Expected Output:
--  blocked_session | blocking_session | wait_type | wait_time | blocker_status | blocker_open_tran | blocker_query
-- -----------------+------------------+-----------+-----------+----------------+-------------------+---------------
--              120 |              115 | LCK_M_X   |      5000 | sleeping       |                 1 | BEGIN TRAN...
--              125 |              120 | LCK_M_S   |      3000 | suspended      |                 0 | SELECT...
```

**Why this output occurs:** Session 120 is blocked by 115, and session 125 is blocked by 120. Session 115 is the lead blocker (status "sleeping" with an open transaction, indicating it's holding locks but not actively working). The chain is 115 → 120 → 125 .

### Real-World Cases

**Case 1: Cascading Application Timeouts**

Multiple application threads time out simultaneously. Block chain analysis reveals a single "sleeping" session with an open transaction is the lead blocker. Killing that session resolves all timeouts.

**Case 2: Batch Job Interference**

A nightly batch job holds locks longer than expected, blocking report generation the next morning. The block chain shows the batch job at the head.

### References

- AWS re:Post — Improve long-running queries in Amazon RDS PostgreSQL - https://repost.aws/knowledge-center/improve-long-running-postgresql-queries
- Microsoft Learn — Descripción y resolución de problemas de bloqueo - https://learn.microsoft.com/es-es/troubleshoot/sql/database-engine/performance/understand-resolve-blocking
- Stack Overflow — Revision: Blocking session query - https://stackoverflow.com/revisions/79049077/1

---

## 3. Deadlock Logging & Analysis

### Definitions

**Core Definition:** Deadlock logging is the capture and storage of deadlock events, including the transactions, locks, and resources involved, for subsequent analysis and prevention.

**Technical Definition:** A deadlock occurs when two or more transactions form a circular wait, each holding a lock the other needs. Database engines detect deadlocks and abort one transaction (the "victim") to break the cycle. Deadlock logs (graphs) capture the process list, resource list, and victim selection, enabling DBAs to understand the root cause and prevent recurrence .

**Beginner-Friendly Explanation:** A deadlock is like two people at a four-way stop, each waiting for the other to go first. Neither moves, and eventually one has to back up. The deadlock log is the "police report" that tells you who was involved, what they were doing, and who had to back up.

### Purposes

- To capture deadlock events for post-incident analysis.
- To identify the SQL statements and resources involved in deadlocks.
- To determine which transaction was chosen as the victim and why.
- To implement preventive measures (lock ordering, index tuning, retry logic).

### Syntax Rules and Structure

**SQL Server: Capturing Deadlock Graphs via Extended Events**

```sql
-- Query system_health ring buffer for deadlock events
SELECT 
    xdr.value('@timestamp', 'datetime') AS deadlock_time,
    xdr.query('.') AS event_data
FROM (
    SELECT CAST([target_data] AS XML) AS target_data
    FROM sys.dm_xe_session_targets AS xt
    INNER JOIN sys.dm_xe_sessions AS xs 
        ON xs.address = xt.event_session_address
    WHERE xs.name = N'system_health'
      AND xt.target_name = N'ring_buffer'
) AS XML_Data
CROSS APPLY target_data.nodes('RingBufferTarget/event[@name="xml_deadlock_report"]') AS XEventData(xdr)
ORDER BY deadlock_time DESC;
```

**MySQL: SHOW ENGINE INNODB STATUS**

```sql
SHOW ENGINE INNODB STATUS\G
-- Look for the "LATEST DETECTED DEADLOCK" section
```

**PostgreSQL: Deadlock Logging Configuration**

```sql
-- Enable logging of lock waits (set in postgresql.conf)
deadlock_timeout = '1s'
log_lock_waits = on
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `xml_deadlock_report` | SQL Server Extended Event containing deadlock graph |
| `system_health` session | Built-in session capturing deadlocks by default |
| `LATEST DETECTED DEADLOCK` | MySQL InnoDB status section |
| `deadlock_timeout` | PostgreSQL parameter controlling deadlock detection frequency |
| `log_lock_waits` | PostgreSQL parameter to log lock waits |

**Syntax Rules:**

- SQL Server's `system_health` session captures deadlocks automatically since 2008 .
- MySQL's `SHOW ENGINE INNODB STATUS` shows only the most recent deadlock; enable `innodb_print_all_deadlocks` for persistent logging .
- PostgreSQL logs lock waits when `log_lock_waits = on` and waits exceed `deadlock_timeout` .

**Constraints and Limitations:**

- Deadlock graphs can be large and complex to parse manually.
- MySQL's in-memory deadlock buffer is overwritten by new deadlocks.
- PostgreSQL's deadlock detection is periodic (default 1s), so short deadlocks may resolve before detection .

### Annotated Complete Code Examples

**Example 1: MySQL Deadlock Analysis**

```sql
-- Show the latest deadlock
SHOW ENGINE INNODB STATUS\G

-- Relevant output section:
-- ------------------------
-- LATEST DETECTED DEADLOCK
-- ------------------------
-- 2024-05-10 14:32:11
-- *** (1) TRANSACTION:
-- TRANSACTION 421938, ACTIVE 2 sec
-- UPDATE orders SET status = 'shipped' WHERE id = 101
-- *** (1) HOLDS THE LOCK(S):
-- RECORD LOCKS ... table orders lock_mode X
-- *** (1) WAITING FOR THIS LOCK TO BE GRANTED:
-- RECORD LOCKS ... table inventory lock_mode X waiting
-- *** (2) TRANSACTION:
-- TRANSACTION 421939, ACTIVE 3 sec
-- UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 55
-- *** (2) HOLDS THE LOCK(S):
-- RECORD LOCKS ... table inventory lock_mode X
-- *** (2) WAITING FOR THIS LOCK TO BE GRANTED:
-- RECORD LOCKS ... table orders lock_mode X waiting
-- *** WE ROLL BACK TRANSACTION (2)
```

**Why this output occurs:** Transaction 1 holds a lock on `orders` and waits for `inventory`. Transaction 2 holds a lock on `inventory` and waits for `orders`. Circular wait detected. Transaction 2 (smaller) is rolled back to break the deadlock .

### Real-World Cases

**Case 1: Application Deadlock Retry**

An application encounters frequent deadlocks (error 1213 in MySQL). The DBA analyzes the deadlock log and discovers that transactions are accessing tables in inconsistent order. The application is modified to access tables in a consistent order.

**Case 2: SQL Server Deadlock Graph Analysis**

A deadlock occurs in SQL Server. The DBA queries the `system_health` session, extracts the deadlock graph XML, and uses Management Studio's graphical view to see that two queries are updating the same table in different orders.

### References

- GitHub — How to Analyze Deadlocks with SHOW ENGINE INNODB STATUS in MySQL - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-how-to-analyze-deadlocks-with-show-engine-innodb-status-in-m/README.md
- Microsoft Learn — Guia de Impasses - SQL Server - https://learn.microsoft.com/pt-pt/sql/relational-databases/sql-server-deadlocks-guide
- Microsoft Learn — Analýza a prevence zablokování - Azure SQL Database - https://learn.microsoft.com/cs-cz/azure/azure-sql/database/analyze-prevent-deadlocks
- Oracle Help Center — 15.7.5.2 デッドロック検出 - https://docs.oracle.com/cd/E17952_01/mysql-8.0-ja/innodb-deadlock-detection.html
- GitHub — postgres logging - https://github.com/ghdrako/doc_snipets/wiki/postgres-logging
- DataNumen — SQL Server Розширені події - https://www.datanumen.com/uk/sql-recovery/guides/sql-server-extended-events/

---

## 4. Under-the-Hood Sizing

### Definitions

**Core Definition:** Under-the-hood sizing involves monitoring and managing the internal storage structures that support transaction processing, including undo logs, write-ahead logs (WAL), redo logs, and MVCC garbage collection overhead.

**Technical Definition:** Database engines use internal storage structures to maintain ACID properties and support MVCC (Multi-Version Concurrency Control). Undo logs (Oracle, MySQL) store pre-modification values for rollback and read consistency. WAL (PostgreSQL) and redo logs (Oracle, MySQL) ensure durability. MVCC garbage collection (PostgreSQL VACUUM, MySQL purge) reclaims space from dead row versions. Monitoring these structures is critical for preventing bloat, transaction ID wraparound, and disk space exhaustion .

**Beginner-Friendly Explanation:** Think of your database like a warehouse. The "undo log" is the receipt you keep in case you need to return something. The "WAL" is the shipping manifest that proves what was sent. "MVCC garbage collection" is the janitor who cleans up returned items. If the receipts pile up too high (undo retention), the manifests get too large (WAL growth), or the janitor doesn't clean (VACUUM), the warehouse becomes unusable.

### Purposes

- To prevent disk space exhaustion from undo, redo, or WAL growth.
- To ensure MVCC garbage collection keeps up with transaction load.
- To monitor transaction ID wraparound risk (PostgreSQL).
- To tune undo retention for long-running queries (Oracle).

### Syntax Rules and Structure

**PostgreSQL: VACUUM and Bloat Monitoring**

```sql
-- Check table bloat
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS total_size,
    pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) AS table_size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Manual VACUUM (removes dead tuples)
VACUUM (VERBOSE, ANALYZE) table_name;

-- Check transaction ID age
SELECT datname, age(datfrozenxid) AS xid_age
FROM pg_database
ORDER BY xid_age DESC;
```

**PostgreSQL: WAL Monitoring**

```sql
-- Check WAL directory size (requires OS access)
-- SELECT pg_size_pretty(sum(size)) FROM pg_ls_waldir();

-- Check replication slots (may retain WAL)
SELECT slot_name, active, 
       pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) AS retained_wal
FROM pg_replication_slots;
```

**MySQL: Purge Lag Monitoring**

```sql
-- Check history list length (purge lag)
SHOW ENGINE INNODB STATUS\G
-- Look for "History list length" in the TRANSACTIONS section

-- Or query performance_schema
SELECT * FROM information_schema.innodb_metrics 
WHERE name LIKE '%purge%';
```

**Oracle: Undo Retention**

```sql
-- Check undo tablespace usage
SELECT tablespace_name, 
       ROUND(SUM(bytes)/1024/1024, 2) AS mb_used
FROM dba_undo_extents
GROUP BY tablespace_name;

-- Check undo retention setting
SHOW PARAMETER undo_retention;
```

**Component Breakdown:**

| Component | Purpose | Risk If Unmanaged |
|-----------|---------|-------------------|
| Undo Log (Oracle/MySQL) | Rollback and read consistency | ORA-01555 snapshot too old; disk exhaustion |
| WAL (PostgreSQL) | Durability and replication | Disk exhaustion; replication lag |
| Redo Log (Oracle/MySQL) | Crash recovery | Checkpoint delays; instance hangs |
| VACUUM (PostgreSQL) | Reclaim dead tuples | Table bloat; XID wraparound |
| Purge (MySQL) | Reclaim old versions | History list growth; disk bloat |

**Syntax Rules:**

- PostgreSQL VACUUM can be automatic (`autovacuum`) or manual. Monitor `pg_stat_user_tables` for dead tuple counts.
- MySQL's `innodb_purge_threads` controls purge parallelism.
- Oracle's `undo_retention` specifies the minimum time undo data is retained.
- WAL growth is controlled by `max_wal_size` and `min_wal_size` (PostgreSQL).

**Constraints and Limitations:**

- VACUUM can be I/O-intensive and may need to be scheduled during off-peak hours.
- Undo retention that is too long can cause undo tablespace exhaustion.
- Purge lag can cause read performance degradation in MySQL.

### Annotated Complete Code Examples

**Example 1: PostgreSQL VACUUM Monitoring**

```sql
-- Check dead tuple counts and last vacuum
SELECT 
    schemaname,
    relname AS tablename,
    n_live_tup AS live_tuples,
    n_dead_tup AS dead_tuples,
    last_vacuum,
    last_autovacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY n_dead_tup DESC;

-- Expected Output:
--  schemaname | tablename | live_tuples | dead_tuples | last_vacuum | last_autovacuum
-- ------------+-----------+-------------+-------------+-------------+-----------------
--  public     | orders    |      500000 |       50000 |             | 2024-05-10 03:00
--  public     | audit_log |     2000000 |      200000 |             | 2024-05-09 02:00
```

**Why this output occurs:** The `orders` table has 50,000 dead tuples that have not been reclaimed. The last autovacuum was on May 10. Without VACUUM, the table will continue to bloat, consuming disk space and degrading query performance .

### Real-World Cases

**Case 1: PostgreSQL Bloat from Long-Running Transactions**

A reporting query runs for hours, preventing VACUUM from reclaiming dead tuples because the snapshot is still needed. The DBA monitors `n_dead_tup` and schedules VACUUM after the report completes.

**Case 2: MySQL Purge Lag**

A write-heavy application causes the InnoDB history list to grow. The DBA monitors `History list length` and increases `innodb_purge_threads` to improve purge throughput.

### References

- GitHub — postgres logging (log_min_duration_statement) - https://github.com/ghdrako/doc_snipets/wiki/postgres-logging
- AWS re:Post — Improve long-running queries in Amazon RDS PostgreSQL - https://repost.aws/knowledge-center/improve-long-running-postgresql-queries

---

## Summary Table: Monitoring Tools by RDBMS

| RDBMS | Blocking View | Deadlock Capture | Internal Sizing |
|-------|--------------|------------------|-----------------|
| PostgreSQL | `pg_locks`, `pg_stat_activity`, `pg_blocking_pids()` | `log_lock_waits`, deadlock logs | `VACUUM`, `pg_stat_user_tables`, `pg_ls_waldir()` |
| SQL Server | `sys.dm_tran_locks`, `sys.dm_exec_requests` | `system_health` session (`xml_deadlock_report`) | Lock memory, tempdb usage |
| MySQL | `information_schema.innodb_lock_waits`, `SHOW PROCESSLIST` | `SHOW ENGINE INNODB STATUS`, `innodb_print_all_deadlocks` | `History list length`, purge threads |
| Oracle | `V$LOCK`, `V$SESSION`, `V$LOCKED_OBJECT` | Alert log, `DBA_BLOCKERS` | `DBA_UNDO_EXTENTS`, `undo_retention` |

---

## Dialect Comparison: Diagnostic Configuration

| Setting | PostgreSQL | MySQL | SQL Server | Oracle |
|---------|-----------|-------|------------|--------|
| Lock wait timeout | `lock_timeout` | `innodb_lock_wait_timeout` | `SET LOCK_TIMEOUT` | `DISTRIBUTED_LOCK_TIMEOUT` |
| Deadlock detection | `deadlock_timeout` (1s default) | `innodb_deadlock_detect` (ON) | Automatic | Automatic |
| Deadlock logging | `log_lock_waits = on` | `innodb_print_all_deadlocks = ON` | `system_health` session | Alert log |
| Slow query logging | `log_min_duration_statement` | `slow_query_log` | Extended Events | `SQL_TRACE` / AWR |

---

## References

- AWS re:Post — Improve long-running queries in Amazon RDS PostgreSQL - https://repost.aws/knowledge-center/improve-long-running-postgresql-queries
- Alibaba Cloud — 如何快速解决RDS SQL Server的阻塞问题 - https://www.alibabacloud.com/help/zh/rds/apsaradb-rds-for-sql-server/resolve-the-blocking-issues-on-an-apsaradb-rds-for-sql-server-instance
- Microsoft — SQL Server Locking Granularity and Escalation - https://download.microsoft.com/download/d/b/d/dbde7972-1eb9-470a-ba18-58849db3eb3b/tshootperfprobs2008.docx
- GitHub — How to Analyze Deadlocks with SHOW ENGINE INNODB STATUS in MySQL - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-how-to-analyze-deadlocks-with-show-engine-innodb-status-in-m/README.md
- Microsoft Learn — Analýza a prevence zablokování - Azure SQL Database - https://learn.microsoft.com/cs-cz/azure/azure-sql/database/analyze-prevent-deadlocks
- Microsoft Learn — Guia de Impasses - SQL Server - https://learn.microsoft.com/pt-pt/sql/relational-databases/sql-server-deadlocks-guide
- GitHub — postgres logging - https://github.com/ghdrako/doc_snipets/wiki/postgres-logging
- Supabase Docs — How to check if my queries are being blocked by other queries? - https://supabase.com/docs/guides/troubleshooting/how-to-check-if-my-queries-are-being-blocked-by-other-queries-NSKtR1
- Microsoft Learn — Descripción y resolución de problemas de bloqueo - https://learn.microsoft.com/es-es/troubleshoot/sql/database-engine/performance/understand-resolve-blocking
- Tencent Cloud — MySQL故障排查全攻略 - https://cloud.tencent.cn/developer/article/2595342
- Microsoft Learn — Guide des interblocages - SQL Server - https://learn.microsoft.com/fr-fr/sql/relational-databases/sql-server-deadlocks-guide
- GitHub — postgres logging (log_min_duration_statement) - https://github.com/ghdrako/doc_snipets/wiki/postgres-logging
- Alibaba Cloud — Diagnose and handle lock waits - https://www.alibabacloud.com/help/en/analyticdb/analyticdb-for-postgresql/user-guide/how-do-i-diagnose-and-handle-locks
- Stack Overflow — Revision: Blocking session query - https://stackoverflow.com/revisions/79049077/1
- Microsoft — Oracle to SQL Server Migration Handbook - https://download.microsoft.com/download/8/D/B/8DBE04B2-9047-477D-B716-AB89BE97B70C/Oracle%20to%20SQL%20Server%20Migration%20Handbook.pdf
- Oracle Help Center — 15.7.5.2 デッドロック検出 - https://docs.oracle.com/cd/E17952_01/mysql-8.0-ja/innodb-deadlock-detection.html
- DataNumen — SQL Server Розширені події - https://www.datanumen.com/uk/sql-recovery/guides/sql-server-extended-events/
- Microsoft Learn — Panduan kebuntuan - SQL Server - https://learn.microsoft.com/id-id/sql/relational-databases/sql-server-deadlocks-guide