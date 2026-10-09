# SQL Transaction Debugging: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**: SQL transaction debugging is the systematic process of identifying, diagnosing, and resolving concurrency problems that arise when multiple transactions compete for shared resources, hold locks too long, or leave connections in an inconsistent state.

**Technical Definition**: Transaction debugging encompasses lock contention analysis (identifying row-level and table-level locks that block concurrent queries), deadlock diagnosis (reading engine deadlock logs and wait-for graphs to determine why two or more transactions are mutually blocked), long-running transaction tracking (finding sessions that hold open transactions and prevent MVCC cleanup or log truncation), isolation anomaly detection (dirty reads, non-repeatable reads, phantom reads, and serialization failures), uncommitted transaction recovery (connections idle in transaction without COMMIT or ROLLBACK), and connection pool starvation troubleshooting (all available connections exhausted by leaked or stuck sessions).

**Beginner-Friendly Explanation**: Transaction debugging is like investigating a traffic jam. Cars (transactions) want to use the same intersection (row or table). If two cars each wait for the other to move, nobody goes anywhere (deadlock). If one car parks in the intersection and never leaves (long-running transaction), everyone else is blocked. And if all the parking spaces (connections) are taken by cars that aren't going anywhere, new cars can't even enter the lot (pool starvation).

### Key Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Blocking vs. Deadlock** | Blocking is one-way waiting; deadlock is circular waiting |
| **Lock Granularity** | Row-level (fine), page-level, table-level (coarse) |
| **Isolation Level** | Determines which anomalies are possible and which locks are taken |
| **Timeout Behavior** | Some waits time out; others wait indefinitely |
| **MVCC Impact** | Long transactions prevent vacuum/cleanup in PostgreSQL, bloating tables |
| **Observability** | Each engine exposes different system views and logs |

### Prerequisites

- **Privileged Access**: Ability to query system views (pg_stat_activity, performance_schema, sys.dm_tran_locks)
- **Engine Log Access**: PostgreSQL logs, MySQL InnoDB status, SQL Server error log and deadlock graphs
- **Isolation Level Knowledge**: Understanding of READ UNCOMMITTED through SERIALIZABLE
- **Monitoring Baseline**: Known normal connection counts, transaction durations, and lock wait times
- **Timeout Configuration**: `lock_timeout`, `deadlock_timeout`, `innodb_lock_wait_timeout`, `LOCK_TIMEOUT`

### Related Programming Areas

- **Concurrency Control**: Lock managers, MVCC, two-phase locking
- **Application Design**: Transaction scoping, connection lifecycle, retry logic
- **Database Administration**: Vacuum tuning, connection pool sizing, monitoring
- **Performance Engineering**: Lock wait analysis, throughput optimization
- **Site Reliability Engineering**: Incident response, deadlock alerting, pool exhaustion runbooks

### Core Concepts Overview

SQL transaction debugging comprises six complementary categories:

1. **Lock Contention**: Identifying blocking queries due to row-level or table-level locks
2. **Deadlocks**: Diagnosing circular waits through engine status logs
3. **Long-Running Transactions**: Tracking processes that hold locks and block cleanup
4. **Isolation-Related Anomalies**: Debugging dirty reads, non-repeatable reads, phantom reads
5. **Uncommitted Transactions**: Recovering connections idle in transaction
6. **Connection Pool Starvation**: Troubleshooting application freezes from exhausted connections

---

## Core Concept 1: Lock Contention

### Definitions

**Core Definition**: Lock contention occurs when one transaction holds a lock on a resource that another transaction needs, forcing the second transaction to wait.

**Technical Definition**: Lock contention arises from competing lock requests at the same granularity. Row-level locks allow concurrent access to different rows but block conflicting access to the same row. Table-level locks (e.g., `ACCESS EXCLUSIVE` in PostgreSQL, `LOCK TABLES` in MySQL) block all concurrent access. SQL Server uses lock modes including Shared (S), Update (U), Exclusive (X), Intent (I), and Schema (Sch), each with different compatibility matrices. Contention is measured by lock wait time; prolonged waits indicate serialization bottlenecks.

**Beginner-Friendly Explanation**: Lock contention is like two people trying to edit the same document at the same time. The first person locks the paragraph they're editing. The second person wants to edit the same paragraph but must wait until the first person finishes. If the first person takes too long, the second person waits a long time.

### Purposes

- **To** identify which transaction is blocking others and on what resource
- **To** distinguish row-level contention (fine-grained, short waits) from table-level contention (coarse-grained, long waits)
- **To** determine the root blocker in a chain of waiting transactions
- **To** resolve contention by committing/rolling back the blocker, killing it, or adding indexes to reduce lock scope

### Syntax Rules and Structure

#### PostgreSQL: Find Blocking Queries

```sql
SELECT
    blocked.pid AS blocked_pid,
    blocked.query AS blocked_query,
    blocking.pid AS blocking_pid,
    blocking.query AS blocking_query,
    blocked.wait_event_type,
    blocked.wait_event,
    now() - blocked.query_start AS blocked_duration
FROM pg_stat_activity blocked
JOIN pg_stat_activity blocking
    ON blocking.pid = ANY(pg_blocking_pids(blocked.pid))
WHERE blocked.wait_event_type = 'Lock';
```

#### PostgreSQL: Find Locks Held

```sql
SELECT
    l.pid,
    l.locktype,
    l.relation::regclass AS table_name,
    l.mode,
    l.granted,
    a.query,
    now() - a.query_start AS duration
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE NOT l.granted  -- Waiting locks
   OR l.mode IN ('AccessExclusiveLock', 'ExclusiveLock')
ORDER BY l.pid;
```

#### MySQL: Find Blocking Transactions

```sql
SELECT
    waiting_trx_id,
    waiting_thread,
    waiting_query,
    blocking_trx_id,
    blocking_thread,
    blocking_query
FROM sys.innodb_lock_waits;
```

#### MySQL: InnoDB Lock Waits

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

#### SQL Server: Find Blocking Sessions

```sql
SELECT
    blocked.session_id AS blocked_session,
    blocked.wait_type,
    blocked.wait_time,
    blocked.wait_resource,
    blocking.session_id AS blocking_session,
    blocking_text.text AS blocking_query
FROM sys.dm_exec_requests blocked
JOIN sys.dm_exec_requests blocking
    ON blocked.blocking_session_id = blocking.session_id
CROSS APPLY sys.dm_exec_sql_text(blocking.sql_handle) AS blocking_text
WHERE blocked.blocking_session_id <> 0;
```

#### Component Breakdown

| Field | Description |
|-------|-------------|
| `blocked_pid` / `waiting_thread` | Session waiting for a lock |
| `blocking_pid` / `blocking_thread` | Session holding the lock |
| `locktype` / `wait_type` | Type of lock (relation, tuple, transaction) |
| `mode` | Lock mode (AccessShareLock, RowExclusiveLock, etc.) |
| `granted` | TRUE if lock acquired, FALSE if waiting |
| `wait_duration` | How long the wait has persisted |

#### Syntax Rules

- `pg_blocking_pids()` returns the PIDs of sessions blocking a given PID.
- `sys.innodb_lock_waits` (MySQL) provides a ready-made view of blocking relationships.
- SQL Server's `blocking_session_id` column is 0 when the session is not blocked.
- Filter for `wait_event_type = 'Lock'` (PostgreSQL) to isolate lock waits from I/O waits.

#### Constraints and Limitations

- Lock contention is normal under concurrent load; only prolonged waits are problems.
- Killing the blocker rolls back its transaction, which may take time for large transactions.
- Table-level locks (e.g., `ALTER TABLE`) block all concurrent access and should be scheduled during maintenance windows.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL Lock Contention Diagnosis

```sql
-- Step 1: Session A begins a transaction and updates a row (holds lock)
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
-- Lock held; do not commit yet
```

```sql
-- Step 2: Session B tries to update the same row (waits)
BEGIN;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 1;
-- Blocks, waiting for Session A
```

```sql
-- Step 3: In a third session, find the blocker
SELECT
    blocked.pid AS blocked_pid,
    blocked.query AS blocked_query,
    blocking.pid AS blocking_pid,
    blocking.query AS blocking_query,
    blocked.wait_event_type,
    blocked.wait_event,
    now() - blocked.query_start AS blocked_duration
FROM pg_stat_activity blocked
JOIN pg_stat_activity blocking
    ON blocking.pid = ANY(pg_blocking_pids(blocked.pid))
WHERE blocked.wait_event_type = 'Lock';
```

**Expected Output**:
```
 blocked_pid | blocked_query                                          | blocking_pid | blocking_query                                          | wait_event_type | wait_event      | blocked_duration
-------------+--------------------------------------------------------+--------------+---------------------------------------------------------+-----------------+-----------------+------------------
       12346 | UPDATE accounts SET balance = balance + 100 WHERE ...  |        12345 | UPDATE accounts SET balance = balance - 100 WHERE ...   | Lock            | transactionid   | 00:00:15.234
```

```sql
-- Step 4: Resolve by committing or rolling back the blocker (Session A)
COMMIT;  -- or ROLLBACK
-- Session B's lock is granted, and its UPDATE proceeds.
```

**Why This Output Occurs**: Session A holds an exclusive row lock from the `UPDATE`. Session B requests the same lock and waits. The query joins `pg_stat_activity` to `pg_blocking_pids()` to reveal the blocking relationship. Committing Session A releases the lock, unblocking Session B.

#### Example 2: MySQL InnoDB Lock Wait Diagnosis

```sql
-- Step 1: Create a table and start a transaction
CREATE TABLE accounts (account_id INT PRIMARY KEY, balance DECIMAL(10,2));
INSERT INTO accounts VALUES (1, 1000), (2, 1000);

-- Session A
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
-- Lock held

-- Session B
START TRANSACTION;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 1;
-- Waits

-- Step 2: In a third session, check lock waits
SELECT
    waiting_trx_id,
    waiting_thread,
    waiting_query,
    blocking_trx_id,
    blocking_thread,
    blocking_query
FROM sys.innodb_lock_waits;
```

**Expected Output**:
```
+----------------+----------------+-----------------------------------------+-----------------+-----------------+-----------------------------------------+
| waiting_trx_id | waiting_thread | waiting_query                           | blocking_trx_id | blocking_thread | blocking_query                          |
+----------------+----------------+-----------------------------------------+-----------------+-----------------+-----------------------------------------+
| 123456         | 42             | UPDATE accounts SET balance = balance...| 123455          | 41              | UPDATE accounts SET balance = balance...|
+----------------+----------------+-----------------------------------------+-----------------+-----------------+-----------------------------------------+
```

```sql
-- Step 3: Check InnoDB status for more detail
SHOW ENGINE INNODB STATUS\G
-- Look for "TRANSACTIONS" section showing lock waits
```

**Expected Output (excerpt)**:
```
---TRANSACTION 123456, ACTIVE 15 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 2 lock struct(s), heap size 1136, 1 row lock(s)
MySQL thread id 42, ... UPDATE accounts SET balance = balance + 100 WHERE account_id = 1
------- TRX HAS BEEN WAITING 15 SEC FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 2 page no 4 n bits 72 index PRIMARY of table `test`.`accounts`
trx id 123456 lock_mode X locks rec but not gap waiting
```

**Why This Output Occurs**: `sys.innodb_lock_waits` provides a summary of blocking relationships. `SHOW ENGINE INNODB STATUS` provides detailed lock information, including the lock mode (`X` = exclusive), the index, and the wait duration. Committing Session A releases the lock.

### Real-World Cases

**Case 1: E-Commerce Inventory Contention**: Multiple orders attempt to decrement the same product's inventory simultaneously. Row-level locks serialize the updates, but the wait time is short. Monitoring shows occasional 5-second waits during flash sales. The fix is to use `SELECT ... FOR UPDATE` with `NOWAIT` or application-level queueing.

**Case 2: Schema Change Blocks All Queries**: An `ALTER TABLE` acquires an `ACCESS EXCLUSIVE` lock, blocking all reads and writes. The fix is to use `ALTER TABLE ... ADD COLUMN` (which is metadata-only in PostgreSQL 11+) or schedule DDL during maintenance windows.

**Case 3: Missing Index Causes Table Lock**: A `DELETE` without an index on the filter column scans and locks every row. Adding the index reduces lock scope to matching rows only.

---

## Core Concept 2: Deadlocks

### Definitions

**Core Definition**: A deadlock occurs when two or more transactions are mutually waiting for locks held by each other, creating a cycle that cannot be resolved without intervention.

**Technical Definition**: A deadlock arises from circular lock dependencies: Transaction A holds lock on resource 1 and waits for resource 2; Transaction B holds lock on resource 2 and waits for resource 1. Databases detect deadlocks by building a wait-for graph and searching for cycles. When a cycle is found, the engine chooses a victim (usually the transaction with the least work done) and rolls it back with an error (SQLSTATE 40P01 in PostgreSQL, ERROR 1213 in MySQL, error 1205 in SQL Server).

**Beginner-Friendly Explanation**: A deadlock is like two cars at a four-way stop, each waiting for the other to go first. Neither moves, and the only solution is for one car to back up. The database "backs up" one transaction by rolling it back, allowing the other to proceed.

### Purposes

- **To** read engine deadlock logs to identify the transactions and resources involved
- **To** determine the victim and survivor of a deadlock
- **To** identify the root cause (inconsistent lock ordering, missing indexes, long transactions)
- **To** prevent deadlocks through consistent lock ordering, shorter transactions, and retry logic

### Syntax Rules and Structure

#### PostgreSQL: Deadlock Detection and Logging

```conf
# postgresql.conf
deadlock_timeout = 1s        # Time before checking for deadlock
log_lock_waits = on           # Log waits longer than deadlock_timeout
log_min_messages = warning    # Ensure deadlock messages are logged
```

```sql
-- Check recent deadlocks in logs
-- Look for: "ERROR: deadlock detected"
-- Log includes: "Process X waits for ... blocked by process Y"
```

#### MySQL: InnoDB Deadlock Detection

```sql
-- Enable deadlock logging
SET GLOBAL innodb_print_all_deadlocks = ON;

-- Check the latest deadlock
SHOW ENGINE INNODB STATUS\G
-- Look for "LATEST DETECTED DEADLOCK" section
```

#### SQL Server: Deadlock Graph

```sql
-- Enable trace flag to capture deadlock graphs
DBCC TRACEON (1222, -1);

-- Query system health session for deadlock graphs
SELECT 
    XEvent.query('(event/data/value/deadlock)[1]') AS deadlock_graph
FROM (
    SELECT CAST(target_data AS XML) AS TargetData
    FROM sys.dm_xe_session_targets st
    JOIN sys.dm_xe_sessions s ON s.address = st.event_session_address
    WHERE s.name = 'system_health'
) AS Data
CROSS APPLY TargetData.nodes('//RingBufferTarget/event') AS XEventData(XEvent)
WHERE XEvent.query('(event/@name)[1]').value('.', 'varchar(50)') = 'xml_deadlock_report';
```

#### Component Breakdown

| Field | Description |
|-------|-------------|
| `deadlock_timeout` | Time before PostgreSQL checks for deadlock cycles |
| `innodb_print_all_deadlocks` | Logs all deadlocks to MySQL error log |
| `LATEST DETECTED DEADLOCK` | MySQL InnoDB status section showing the last deadlock |
| `xml_deadlock_report` | SQL Server deadlock graph in XML format |
| `deadlock detected` | PostgreSQL error message (SQLSTATE 40P01) |

#### Syntax Rules

- Deadlocks are detected automatically by the engine; no user action is required for detection.
- The victim transaction is rolled back; the application should catch the error and retry.
- `deadlock_timeout` should be tuned: too low causes frequent checks, too high delays detection.
- Deadlock logs include the SQL statements and lock modes involved.

#### Constraints and Limitations

- Deadlocks are unavoidable in systems with concurrent writes; the goal is to minimize them.
- The victim selection is not always the "less important" transaction; applications should retry.
- Deadlock logs may be verbose; use structured logging or monitoring tools to aggregate.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL Deadlock Diagnosis

```sql
-- Step 1: Session A — lock row 1, then try to lock row 2
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
-- (pause)
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
```

```sql
-- Step 2: Session B — lock row 2, then try to lock row 1 (opposite order)
BEGIN;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
-- (pause)
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
```

**Expected Error (one session)**:
```
ERROR:  deadlock detected
DETAIL:  Process 12346 waits for ShareLock on transaction 12345; blocked by process 12345.
Process 12345 waits for ShareLock on transaction 12346; blocked by process 12346.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,1) in relation "accounts"
SQLSTATE: 40P01
```

```sql
-- Step 3: Check the PostgreSQL log for the full deadlock report
-- Log shows:
-- ERROR:  deadlock detected
-- DETAIL:  Process 12346 waits for ShareLock on transaction 12345; blocked by process 12345.
--          Process 12345 waits for ShareLock on transaction 12346; blocked by process 12346.
--          Process 12346: UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
--          Process 12345: UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
```

**Why This Output Occurs**: Session A locks row 1, Session B locks row 2. Session A then tries to lock row 2 (held by B), and Session B tries to lock row 1 (held by A). The wait-for graph has a cycle, and PostgreSQL detects the deadlock after `deadlock_timeout`. One session is chosen as the victim and rolled back with SQLSTATE 40P01.

#### Example 2: MySQL InnoDB Deadlock Report

```sql
-- Step 1: Enable deadlock logging
SET GLOBAL innodb_print_all_deadlocks = ON;

-- Step 2: Create the deadlock scenario (same as PostgreSQL example)
-- Session A and Session B execute opposite-order updates.

-- Step 3: Check SHOW ENGINE INNODB STATUS
SHOW ENGINE INNODB STATUS\G
```

**Expected Output (excerpt)**:
```
------------------------
LATEST DETECTED DEADLOCK
------------------------
2026-10-09 10:15:30 0x7f8b8c0a1700
*** (1) TRANSACTION:
TRANSACTION 12345, ACTIVE 5 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 3 lock struct(s), heap size 1136, 2 row lock(s)
MySQL thread id 41, ... UPDATE accounts SET balance = balance + 100 WHERE account_id = 2
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 2 page no 4 n bits 72 index PRIMARY of table `test`.`accounts`
trx id 12345 lock_mode X locks rec but not gap waiting
*** (2) TRANSACTION:
TRANSACTION 12346, ACTIVE 5 sec starting index read
mysql tables in use 1, locked 1
3 lock struct(s), heap size 1136, 2 row lock(s)
MySQL thread id 42, ... UPDATE accounts SET balance = balance - 100 WHERE account_id = 1
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 2 page no 4 n bits 72 index PRIMARY of table `test`.`accounts`
trx id 12346 lock_mode X locks rec but not gap
*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 2 page no 4 n bits 72 index PRIMARY of table `test`.`accounts`
trx id 12346 lock_mode X locks rec but not gap waiting
*** WE ROLL BACK TRANSACTION (1)
```

**Why This Output Occurs**: MySQL logs both transactions, showing what each holds and what each waits for. Transaction 1 is chosen as the victim (`WE ROLL BACK TRANSACTION (1)`) and rolled back. Transaction 2 proceeds.

### Real-World Cases

**Case 1: Inconsistent Lock Ordering**: Two application flows update `accounts` and `audit_log` in different orders, causing deadlocks. The fix is to always acquire locks in the same order (e.g., by primary key ascending).

**Case 2: Missing Index Causes Deadlock**: A `DELETE` without an index locks more rows than necessary, increasing the chance of deadlock. Adding the index reduces lock scope and eliminates deadlocks.

**Case 3: Deadlock Retry Logic**: An application catches SQLSTATE 40P01 and retries the transaction with exponential backoff. Deadlocks become transparent to users.

---

## Core Concept 3: Long-Running Transactions

### Definitions

**Core Definition**: A long-running transaction is a transaction that remains open for an extended period, holding locks and preventing cleanup operations.

**Technical Definition**: In PostgreSQL, long-running transactions hold a snapshot that prevents `VACUUM` from removing dead tuples created after the transaction started, causing table bloat. They also prevent `VACUUM` from advancing the `xmin` horizon, blocking transaction ID wraparound protection. In MySQL, long transactions increase undo log size, hold row locks, and delay purge of old row versions. In SQL Server, they hold locks, grow the transaction log, and block log truncation.

**Beginner-Friendly Explanation**: A long-running transaction is like a customer who sits at a table in a restaurant for hours without ordering. The table can't be cleaned, other customers can't sit there, and the restaurant's turnover drops. The database version: old row versions can't be cleaned up, locks are held, and logs grow.

### Purposes

- **To** identify transactions that have been open for an unusually long time
- **To** understand the impact on MVCC cleanup (PostgreSQL vacuum, MySQL purge)
- **To** track down the application code or idle connection responsible
- **To** terminate or allow the transaction to complete to restore normal cleanup

### Syntax Rules and Structure

#### PostgreSQL: Find Long-Running Transactions

```sql
SELECT
    pid,
    usename,
    application_name,
    client_addr,
    state,
    now() - xact_start AS transaction_duration,
    now() - query_start AS query_duration,
    wait_event_type,
    wait_event,
    LEFT(query, 100) AS query_preview
FROM pg_stat_activity
WHERE xact_start IS NOT NULL
  AND now() - xact_start > interval '5 minutes'
ORDER BY xact_start;
```

#### PostgreSQL: Check Transaction ID Age (Wraparound Risk)

```sql
SELECT
    datname,
    age(datfrozenxid) AS xid_age,
    current_setting('autovacuum_freeze_max_age')::int AS max_age,
    ROUND(100.0 * age(datfrozenxid) / 
          current_setting('autovacuum_freeze_max_age')::int, 2) AS pct_toward_wraparound
FROM pg_database
ORDER BY age(datfrozenxid) DESC;
```

#### MySQL: Find Long-Running Transactions

```sql
SELECT
    trx_id,
    trx_state,
    trx_started,
    TIMESTAMPDIFF(SECOND, trx_started, NOW()) AS duration_seconds,
    trx_mysql_thread_id,
    trx_query
FROM information_schema.innodb_trx
WHERE TIMESTAMPDIFF(SECOND, trx_started, NOW()) > 300
ORDER BY trx_started;
```

#### SQL Server: Find Long-Running Transactions

```sql
SELECT
    s.session_id,
    s.login_name,
    s.host_name,
    s.program_name,
    t.transaction_begin_time,
    DATEDIFF(SECOND, t.transaction_begin_time, GETDATE()) AS duration_seconds,
    s.status,
    r.command_text
FROM sys.dm_tran_active_transactions t
JOIN sys.dm_tran_session_transactions st ON t.transaction_id = st.transaction_id
JOIN sys.dm_exec_sessions s ON st.session_id = s.session_id
LEFT JOIN sys.dm_exec_requests r ON s.session_id = r.session_id
WHERE DATEDIFF(SECOND, t.transaction_begin_time, GETDATE()) > 300
ORDER BY t.transaction_begin_time;
```

#### Component Breakdown

| Field | Description |
|-------|-------------|
| `xact_start` | When the transaction began (PostgreSQL) |
| `trx_started` | When the transaction began (MySQL) |
| `transaction_begin_time` | When the transaction began (SQL Server) |
| `xid_age` | Age of the oldest transaction ID (PostgreSQL wraparound risk) |
| `trx_state` | Transaction state (RUNNING, LOCK WAIT, etc.) |
| `wait_event` | What the transaction is waiting for |

#### Syntax Rules

- `xact_start` is NULL for sessions not in a transaction.
- `age(datfrozenxid)` shows how close the database is to transaction ID wraparound.
- MySQL's `innodb_trx` shows only InnoDB transactions.
- SQL Server's `dm_tran_active_transactions` shows all active transactions.

#### Constraints and Limitations

- Long-running transactions may be legitimate (large batch operations); investigate before killing.
- Killing a transaction rolls back all its work, which may take as long as the transaction ran.
- Idle-in-transaction sessions are the most common cause; set `idle_in_transaction_session_timeout`.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL Long-Running Transaction Detection

```sql
-- Step 1: Find transactions running longer than 5 minutes
SELECT
    pid,
    usename,
    state,
    now() - xact_start AS transaction_duration,
    now() - query_start AS query_duration,
    LEFT(query, 80) AS query_preview
FROM pg_stat_activity
WHERE xact_start IS NOT NULL
  AND now() - xact_start > interval '5 minutes'
ORDER BY xact_start;
```

**Expected Output**:
```
 pid  | usename | state  | transaction_duration | query_duration | query_preview
------+---------+--------+----------------------+----------------+-------------------------------
 1234 | app     | idle   | 02:15:30             | 02:15:00       | SELECT * FROM orders WHERE ...
 5678 | app     | active | 00:10:00             | 00:10:00       | UPDATE inventory SET ...
```

```sql
-- Step 2: Check impact on vacuum
SELECT
    schemaname,
    relname,
    n_dead_tup,
    last_vacuum,
    last_autovacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 10000
ORDER BY n_dead_tup DESC;
```

**Expected Output**:
```
 schemaname | relname  | n_dead_tup | last_vacuum | last_autovacuum
------------+----------+------------+-------------+-----------------
 public     | orders   |    2500000 | 2026-10-08  | 2026-10-08
 public     | products |     500000 | 2026-10-09  | 2026-10-09
```

```sql
-- Step 3: Terminate the idle-in-transaction session
SELECT pg_terminate_backend(1234);
-- Expected: t (true)
```

**Why This Output Occurs**: The `idle in transaction` session (PID 1234) has been open for 2 hours and 15 minutes, holding a snapshot that prevents `VACUUM` from removing dead tuples. The `orders` table has 2.5 million dead tuples that cannot be cleaned. Terminating the session allows `VACUUM` to proceed.

#### Example 2: Preventing Idle-in-Transaction Sessions

```sql
-- Set a timeout for idle-in-transaction sessions
ALTER SYSTEM SET idle_in_transaction_session_timeout = '5min';
SELECT pg_reload_conf();

-- Verify
SHOW idle_in_transaction_session_timeout;
```

**Expected Output**:
```
 idle_in_transaction_session_timeout 
-------------------------------------
 5min
```

**Why This Output Occurs**: The `idle_in_transaction_session_timeout` setting automatically terminates sessions that remain idle in a transaction for more than 5 minutes. This prevents long-running idle transactions from blocking vacuum and holding locks.

### Real-World Cases

**Case 1: ORM Connection Leak**: An ORM opens a transaction and fails to commit/rollback on an error path, leaving the connection idle in transaction. The fix adds `idle_in_transaction_session_timeout` and fixes the ORM error handling.

**Case 2: Batch Job Holds Snapshot**: A nightly batch job runs for 6 hours in a single transaction. During that time, `VACUUM` cannot remove dead tuples, and the table bloats by 50%. The fix is to commit periodically or use `VACUUM` with `FREEZE` after the batch.

**Case 3: Transaction ID Wraparound Risk**: A database's `xid_age` reaches 80% of `autovacuum_freeze_max_age` because a long-running transaction blocks freezing. The fix is to terminate the transaction and allow autovacuum to freeze old tuples.

---

## Core Concept 4: Isolation-Related Anomalies

### Definitions

**Core Definition**: Isolation anomalies are incorrect results that occur when concurrent transactions interfere with each other under weaker isolation levels.

**Technical Definition**: The SQL standard defines four isolation levels (READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE) and three anomalies they prevent: dirty reads (reading uncommitted data), non-repeatable reads (reading the same row twice and getting different values), and phantom reads (running the same query twice and getting different row sets). PostgreSQL's REPEATABLE READ prevents phantom reads but may throw serialization errors; MySQL's REPEATABLE READ uses next-key locking to prevent phantoms; SQL Server's SNAPSHOT isolation uses row versioning.

**Beginner-Friendly Explanation**: Isolation anomalies are like reading a book while someone else is editing it. A dirty read is reading a sentence that the author later deletes. A non-repeatable read is reading page 10, then reading it again and finding different text. A phantom read is counting 5 chapters, then counting again and finding 6 because a new chapter was inserted.

### Purposes

- **To** identify which isolation anomalies are possible under the current isolation level
- **To** reproduce anomalies with controlled concurrent transactions
- **To** choose the appropriate isolation level for the application's consistency requirements
- **To** handle serialization failures gracefully in SERIALIZABLE transactions

### Syntax Rules and Structure

#### Isolation Level Matrix

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Serialization Anomaly |
|-----------------|------------|---------------------|--------------|----------------------|
| READ UNCOMMITTED | Possible | Possible | Possible | Possible |
| READ COMMITTED | Not possible | Possible | Possible | Possible |
| REPEATABLE READ | Not possible | Not possible | Possible (standard) | Possible |
| SERIALIZABLE | Not possible | Not possible | Not possible | Not possible |

#### PostgreSQL: Set Isolation Level

```sql
-- Per-transaction
BEGIN ISOLATION LEVEL REPEATABLE READ;
-- or
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

#### MySQL: Set Isolation Level

```sql
-- Per-session
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Per-transaction
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
START TRANSACTION;
```

#### SQL Server: Set Isolation Level

```sql
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
BEGIN TRANSACTION;
```

#### Component Breakdown

| Anomaly | Definition | Example |
|---------|------------|---------|
| Dirty Read | Reading uncommitted data from another transaction | T1 updates balance, T2 reads it, T1 rolls back |
| Non-Repeatable Read | Re-reading a row returns different values | T1 reads balance, T2 updates it, T1 reads again |
| Phantom Read | Re-running a query returns different rows | T1 counts orders, T2 inserts an order, T1 counts again |
| Serialization Anomaly | Result inconsistent with any serial order | Write skew in SERIALIZABLE |

#### Syntax Rules

- PostgreSQL's default is READ COMMITTED; MySQL's default is REPEATABLE READ; SQL Server's default is READ COMMITTED.
- SERIALIZABLE in PostgreSQL uses SSI (Serializable Snapshot Isolation) and may throw SQLSTATE 40001.
- MySQL's REPEATABLE READ uses next-key locking to prevent phantoms.
- SQL Server's SNAPSHOT isolation uses row versioning in tempdb.

#### Constraints and Limitations

- Stronger isolation reduces concurrency and increases lock waits or serialization failures.
- SERIALIZABLE transactions must be prepared to retry on SQLSTATE 40001.
- Snapshot isolation (PostgreSQL REPEATABLE READ, SQL Server SNAPSHOT) can produce write skew.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Non-Repeatable Read Under READ COMMITTED

```sql
-- Step 1: Session A reads a row
BEGIN ISOLATION LEVEL READ COMMITTED;
SELECT balance FROM accounts WHERE account_id = 1;
-- Result: 1000
```

```sql
-- Step 2: Session B updates the row and commits
BEGIN;
UPDATE accounts SET balance = 500 WHERE account_id = 1;
COMMIT;
```

```sql
-- Step 3: Session A reads the same row again
SELECT balance FROM accounts WHERE account_id = 1;
-- Result: 500 (non-repeatable read!)
COMMIT;
```

**Expected Output (Session A)**:
```
 balance 
---------
    1000
(1 row)

 balance 
---------
     500
(1 row)
```

**Why This Output Occurs**: Under READ COMMITTED, each statement sees the latest committed data. Session A's first read sees 1000; Session B commits 500; Session A's second read sees 500. The same query returns different values within the same transaction — a non-repeatable read.

#### Example 2: Preventing Non-Repeatable Read with REPEATABLE READ

```sql
-- Step 1: Session A begins REPEATABLE READ
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT balance FROM accounts WHERE account_id = 1;
-- Result: 1000
```

```sql
-- Step 2: Session B updates and commits
BEGIN;
UPDATE accounts SET balance = 500 WHERE account_id = 1;
COMMIT;
```

```sql
-- Step 3: Session A reads again
SELECT balance FROM accounts WHERE account_id = 1;
-- Result: 1000 (snapshot from transaction start)
COMMIT;
```

**Expected Output (Session A)**:
```
 balance 
---------
    1000
(1 row)

 balance 
---------
    1000
(1 row)
```

**Why This Output Occurs**: Under REPEATABLE READ, Session A sees a consistent snapshot from the start of the transaction. Session B's committed change is not visible to Session A until it starts a new transaction. The non-repeatable read is prevented.

#### Example 3: Phantom Read Under READ COMMITTED

```sql
-- Step 1: Session A counts orders
BEGIN ISOLATION LEVEL READ COMMITTED;
SELECT COUNT(*) FROM orders WHERE customer_id = 1;
-- Result: 5
```

```sql
-- Step 2: Session B inserts a new order and commits
BEGIN;
INSERT INTO orders (customer_id, total) VALUES (1, 100);
COMMIT;
```

```sql
-- Step 3: Session A counts again
SELECT COUNT(*) FROM orders WHERE customer_id = 1;
-- Result: 6 (phantom read!)
COMMIT;
```

**Expected Output (Session A)**:
```
 count 
-------
     5
(1 row)

 count 
-------
     6
(1 row)
```

**Why This Output Occurs**: Under READ COMMITTED, Session A's second count sees the newly committed order, returning 6 instead of 5. This is a phantom read — a new row appeared in the result set.

#### Example 4: Serialization Failure Under SERIALIZABLE

```sql
-- Step 1: Session A begins SERIALIZABLE
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT SUM(balance) FROM accounts WHERE account_id IN (1, 2);
-- Result: 2000
```

```sql
-- Step 2: Session B begins SERIALIZABLE and transfers
BEGIN ISOLATION LEVEL SERIALIZABLE;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
COMMIT;
```

```sql
-- Step 3: Session A tries to commit after reading stale data
COMMIT;
```

**Expected Error (Session A)**:
```
ERROR:  could not serialize access due to read/write dependencies among transactions
DETAIL:  Reason code: Canceled on identification as a pivot, during commit attempt.
HINT:  The transaction might succeed if retried.
SQLSTATE: 40001
```

**Why This Output Occurs**: PostgreSQL's SSI detects that Session A's read of `SUM(balance)` and Session B's write create a serialization conflict. Session A is rolled back with SQLSTATE 40001, and the application should retry.

### Real-World Cases

**Case 1: Reporting Under READ COMMITTED**: A report that runs multiple queries gets inconsistent results because other transactions commit between queries. The fix is to use REPEATABLE READ for the report.

**Case 2: Write Skew in Snapshot Isolation**: Two transactions each check a condition, then update different rows based on the check, producing an inconsistent state. The fix is SERIALIZABLE isolation with retry logic.

**Case 3: Serialization Failure Retry**: An application using SERIALIZABLE catches SQLSTATE 40001 and retries the transaction with exponential backoff, achieving correctness without user-visible errors.

---

## Core Concept 5: Uncommitted Transactions (Idle in Transaction)

### Definitions

**Core Definition**: An idle-in-transaction session is a database connection that has begun a transaction but has not issued COMMIT or ROLLBACK, and is not currently executing a query.

**Technical Definition**: In PostgreSQL, a session in `idle in transaction` state holds a snapshot and may hold locks acquired before going idle. In MySQL, a session with an open transaction holds undo log space and row locks. In SQL Server, the session holds locks and prevents log truncation. Idle-in-transaction sessions are the most common cause of long-running transaction problems because they are invisible to the application (no query is running) but hold resources indefinitely.

**Beginner-Friendly Explanation**: An idle-in-transaction session is like a customer who opened a fitting room, tried on clothes, and then walked away without returning the clothes or closing the door. The room is blocked, and other customers can't use it.

### Purposes

- **To** detect sessions that are idle in transaction for extended periods
- **To** understand the difference between idle (no transaction) and idle in transaction (open transaction)
- **To** configure timeouts that automatically terminate idle-in-transaction sessions
- **To** fix application code that leaves transactions open on error paths

### Syntax Rules and Structure

#### PostgreSQL: Identify Idle-in-Transaction Sessions

```sql
SELECT
    pid,
    usename,
    application_name,
    client_addr,
    state,
    now() - state_change AS idle_duration,
    now() - xact_start AS transaction_duration,
    LEFT(query, 80) AS last_query
FROM pg_stat_activity
WHERE state = 'idle in transaction'
ORDER BY xact_start;
```

#### PostgreSQL: Set Idle-in-Transaction Timeout

```sql
-- Per-session
SET idle_in_transaction_session_timeout = '5min';

-- Global (postgresql.conf)
ALTER SYSTEM SET idle_in_transaction_session_timeout = '5min';
SELECT pg_reload_conf();
```

#### MySQL: Identify Idle-in-Transaction Sessions

```sql
SELECT
    trx_id,
    trx_state,
    trx_started,
    TIMESTAMPDIFF(SECOND, trx_started, NOW()) AS duration_seconds,
    trx_mysql_thread_id,
    trx_query
FROM information_schema.innodb_trx
WHERE trx_state = 'RUNNING'
  AND trx_query IS NULL
ORDER BY trx_started;
```

#### Component Breakdown

| State | Meaning | Risk |
|-------|---------|------|
| `active` | Executing a query | Normal |
| `idle` | No transaction, no query | Normal |
| `idle in transaction` | Open transaction, no query | High — holds snapshot/locks |
| `idle in transaction (aborted)` | Transaction failed, not rolled back | High — holds resources |

#### Syntax Rules

- `idle_in_transaction_session_timeout` applies to sessions that are idle in transaction.
- The timeout does not apply to sessions in `idle` state (no transaction).
- `state_change` shows when the session entered its current state.
- `query_start` shows when the current/last query started.

#### Constraints and Limitations

- Setting the timeout too low may terminate legitimate long-running batch operations.
- The timeout applies per-session; it can be overridden with `SET`.
- Applications must handle connection termination gracefully (reconnect and retry).

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Detecting and Fixing Idle-in-Transaction

```sql
-- Step 1: Find idle-in-transaction sessions
SELECT
    pid,
    usename,
    state,
    now() - state_change AS idle_duration,
    now() - xact_start AS transaction_duration,
    LEFT(query, 60) AS last_query
FROM pg_stat_activity
WHERE state = 'idle in transaction'
ORDER BY xact_start;
```

**Expected Output**:
```
 pid  | usename | state                | idle_duration | transaction_duration | last_query
------+---------+----------------------+---------------+----------------------+----------------------------
 1234 | app     | idle in transaction  | 00:45:00      | 01:30:00             | UPDATE accounts SET ...
 5678 | app     | idle in transaction  | 00:10:00      | 00:15:00             | SELECT * FROM orders ...
```

```sql
-- Step 2: Terminate the oldest idle-in-transaction session
SELECT pg_terminate_backend(1234);
-- Expected: t
```

```sql
-- Step 3: Configure the timeout to prevent recurrence
ALTER SYSTEM SET idle_in_transaction_session_timeout = '5min';
SELECT pg_reload_conf();

-- Step 4: Verify
SHOW idle_in_transaction_session_timeout;
```

**Expected Output**:
```
 idle_in_transaction_session_timeout 
-------------------------------------
 5min
```

**Why This Output Occurs**: PID 1234 has been idle in transaction for 45 minutes, holding a snapshot that blocks vacuum. Terminating it releases the snapshot. The timeout setting ensures future sessions are automatically terminated after 5 minutes of idle-in-transaction.

#### Example 2: Application-Level Fix for Idle-in-Transaction

```java
// WRONG: transaction left open on error path
public void transferFunds(Long from, Long to, BigDecimal amount) {
    Connection conn = dataSource.getConnection();
    conn.setAutoCommit(false);
    
    try {
        debit(conn, from, amount);
        credit(conn, to, amount);
        conn.commit();
    } catch (SQLException e) {
        // MISSING: conn.rollback() — connection remains in transaction!
        throw new RuntimeException(e);
    } finally {
        conn.close();  // Returns connection to pool with open transaction
    }
}

// CORRECT: rollback on error
public void transferFunds(Long from, Long to, BigDecimal amount) {
    Connection conn = dataSource.getConnection();
    conn.setAutoCommit(false);
    
    try {
        debit(conn, from, amount);
        credit(conn, to, amount);
        conn.commit();
    } catch (SQLException e) {
        conn.rollback();  // Explicit rollback
        throw new RuntimeException(e);
    } finally {
        conn.close();
    }
}
```

**Why This Output Occurs**: The wrong version leaves the transaction open when an exception occurs, and `conn.close()` returns the connection to the pool with the transaction still active. The correct version explicitly rolls back before closing, ensuring the connection is clean when returned to the pool.

### Real-World Cases

**Case 1: ORM Error Path Leak**: An ORM's `@Transactional` annotation rolls back on exceptions by default, but a developer uses programmatic transactions and forgets `rollback()` on one error path. The fix adds `rollback()` and configures `idle_in_transaction_session_timeout` as a safety net.

**Case 2: Connection Pool Pollution**: A connection returned to the pool with an open transaction causes the next user of that connection to see stale data. The fix ensures all connections are committed or rolled back before returning to the pool.

**Case 3: Batch Job Leaves Transaction Open**: A batch job opens a transaction, processes 1 million rows, and then waits for user input. The fix commits periodically and closes the transaction before waiting.

---

## Core Concept 6: Connection Pool Starvation

### Definitions

**Core Definition**: Connection pool starvation occurs when all available database connections are in use (or leaked), causing new requests to wait indefinitely or fail.

**Technical Definition**: A connection pool has a fixed maximum size (`maximumPoolSize`). When all connections are checked out and not returned, new requests wait in a queue until a connection becomes available or the `connectionTimeout` expires. Starvation causes include connection leaks (connections not returned on error paths), long-running transactions holding connections, and pool sizing too small for the workload. Symptoms include application freezes, timeout exceptions, and `max_connections` errors at the database.

**Beginner-Friendly Explanation**: Connection pool starvation is like a parking lot with 20 spaces and 100 cars. Once all 20 spaces are taken, new cars wait at the entrance. If some cars never leave (connection leaks), the lot stays full and nobody can park.

### Purposes

- **To** detect when all pool connections are checked out and requests are waiting
- **To** identify connection leaks (connections not returned to the pool)
- **To** distinguish pool starvation from database-side `max_connections` exhaustion
- **To** size the pool correctly for the application's concurrency and the database's capacity

### Syntax Rules and Structure

#### HikariCP: Pool Metrics

```java
// Access pool metrics
HikariPoolMXBean poolMXBean = dataSource.getHikariPoolMXBean();

System.out.println("Active connections: " + poolMXBean.getActiveConnections());
System.out.println("Idle connections: " + poolMXBean.getIdleConnections());
System.out.println("Total connections: " + poolMXBean.getTotalConnections());
System.out.println("Threads awaiting connection: " + poolMXBean.getThreadsAwaitingConnection());
```

#### PostgreSQL: Connection Count by Application

```sql
SELECT
    application_name,
    state,
    COUNT(*) AS connection_count
FROM pg_stat_activity
WHERE backend_type = 'client backend'
GROUP BY application_name, state
ORDER BY connection_count DESC;
```

#### PostgreSQL: Check max_connections

```sql
SHOW max_connections;

SELECT
    COUNT(*) AS total_connections,
    COUNT(*) FILTER (WHERE state = 'active') AS active,
    COUNT(*) FILTER (WHERE state = 'idle') AS idle,
    COUNT(*) FILTER (WHERE state = 'idle in transaction') AS idle_in_transaction
FROM pg_stat_activity
WHERE backend_type = 'client backend';
```

#### MySQL: Connection Count

```sql
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Threads_running';
SHOW VARIABLES LIKE 'max_connections';
```

#### Component Breakdown

| Metric | Description | Warning Threshold |
|--------|-------------|-------------------|
| `Active connections` | Connections currently in use | = maxPoolSize |
| `Idle connections` | Connections available in pool | = 0 |
| `Threads awaiting connection` | Threads blocked waiting for a connection | > 0 |
| `Total connections` | Active + idle | = maxPoolSize |
| `max_connections` | Database server limit | Approaching |

#### Syntax Rules

- Pool size should be sized to `(CPU cores × 2) + effective_spindle_count` for optimal throughput.
- Total connections across all application instances must not exceed `max_connections`.
- `connectionTimeout` should be set to fail fast rather than wait indefinitely.
- `leakDetectionThreshold` helps identify connections held too long.

#### Constraints and Limitations

- More connections do not improve throughput beyond the optimal pool size; they cause contention.
- Connection leaks are the most common cause of starvation; monitor `Active connections` trends.
- Database-side `max_connections` exhaustion differs from pool starvation; check both.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Diagnosing Connection Pool Starvation

```java
// Step 1: Configure HikariCP with leak detection
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:postgresql://localhost:5432/inventory");
config.setMaximumPoolSize(20);
config.setMinimumIdle(5);
config.setConnectionTimeout(5000);       // 5 seconds
config.setLeakDetectionThreshold(30000); // 30 seconds

HikariDataSource dataSource = new HikariDataSource(config);

// Step 2: Monitor pool metrics
HikariPoolMXBean pool = dataSource.getHikariPoolMXBean();
System.out.println("Active: " + pool.getActiveConnections());
System.out.println("Idle: " + pool.getIdleConnections());
System.out.println("Awaiting: " + pool.getThreadsAwaitingConnection());
```

**Expected Output (Normal)**:
```
Active: 3
Idle: 17
Awaiting: 0
```

**Expected Output (Starvation)**:
```
Active: 20
Idle: 0
Awaiting: 45
```

**Why This Output Occurs**: In the starvation scenario, all 20 connections are active, none are idle, and 45 threads are waiting for a connection. This indicates a connection leak or long-running transactions holding connections. HikariCP's leak detection logs a warning for any connection held longer than 30 seconds.

#### Example 2: Database-Side Connection Exhaustion

```sql
-- Step 1: Check database connection limits
SHOW max_connections;
-- Expected: 100

-- Step 2: Count current connections by application
SELECT
    application_name,
    state,
    COUNT(*) AS connection_count
FROM pg_stat_activity
WHERE backend_type = 'client backend'
GROUP BY application_name, state
ORDER BY connection_count DESC;
```

**Expected Output**:
```
 application_name | state                | connection_count
------------------+----------------------+------------------
 app-service      | idle                 |               80
 app-service      | active               |               15
 app-service      | idle in transaction  |                5
```

```sql
-- Step 3: Check for idle-in-transaction connections holding slots
SELECT
    pid,
    application_name,
    state,
    now() - state_change AS idle_duration
FROM pg_stat_activity
WHERE state = 'idle in transaction'
ORDER BY state_change;
```

**Expected Output**:
```
 pid  | application_name | state                | idle_duration
------+------------------+----------------------+---------------
 1234 | app-service      | idle in transaction  | 00:45:00
 5678 | app-service      | idle in transaction  | 00:15:00
```

**Why This Output Occurs**: The application is using 100 connections (80 idle, 15 active, 5 idle-in-transaction), approaching the `max_connections` limit of 100. The 5 idle-in-transaction connections are holding snapshots and slots. The fix is to terminate the idle-in-transaction sessions, set `idle_in_transaction_session_timeout`, and reduce the pool size per application instance.

### Real-World Cases

**Case 1: Serverless Connection Storm**: A Lambda function opens a new connection per invocation, exhausting `max_connections`. The fix is to use RDS Proxy or PgBouncer for connection multiplexing.

**Case 2: Connection Leak in Error Path**: An application forgets to close connections in a `catch` block, leaking connections until the pool is exhausted. The fix adds try-with-resources or `finally` blocks.

**Case 3: Pool Sizing Too Small**: An application with 100 concurrent requests uses a pool of 10 connections, causing 90 requests to wait. The fix increases the pool size to match concurrency and database capacity.

---

## References

| Name | Link |
|------|------|
| PostgreSQL Documentation — Explicit Locking | https://www.postgresql.org/docs/current/explicit-locking.html |
| PostgreSQL Documentation — Monitoring Locks | https://www.postgresql.org/docs/current/monitoring-locks.html |
| PostgreSQL Documentation — Transaction Isolation | https://www.postgresql.org/docs/current/transaction-iso.html |
| PostgreSQL Documentation — Routine Vacuuming | https://www.postgresql.org/docs/current/routine-vacuuming.html |
| MySQL 8.0 Reference Manual — InnoDB Locking | https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html |
| MySQL 8.0 Reference Manual — Deadlocks | https://dev.mysql.com/doc/refman/8.0/en/innodb-deadlocks.html |
| MySQL 8.0 Reference Manual — Transaction Isolation Levels | https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html |
| Microsoft Learn — Transaction Locking and Row Versioning Guide | https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide |
| Microsoft Learn — Deadlocks | https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide#deadlocks |
| Microsoft Learn — sys.dm_tran_locks | https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-locks-transact-sql |
| HikariCP — Configuration | https://github.com/brettwooldridge/HikariCP |
| PgBouncer Documentation | https://www.pgbouncer.org/config.html |
| PostgreSQL Wiki — Lock Monitoring | https://wiki.postgresql.org/wiki/Lock_Monitoring |
| Percona — How to Diagnose and Resolve MySQL Deadlocks | https://www.percona.com/blog/ |
| Redgate — SQL Server Deadlock Troubleshooting | https://www.red-gate.com/simple-talk/databases/sql-server/ |
| AWS — Amazon RDS Connection Management | https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_BestPractices.html |