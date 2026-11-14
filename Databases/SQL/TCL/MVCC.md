# Comprehensive Programming Cheat Sheet: MVCC (Multi-Version Concurrency Control)

## Topic Overview

### Definitions

**Core Definition**  
Multi-Version Concurrency Control (MVCC) is a concurrency-control architecture in which the database maintains multiple versions of each data item so that read operations can access a consistent snapshot of the data without blocking concurrent write operations, and writes do not block reads.

**Technical Definition**  
MVCC is a family of concurrency-control protocols in which each write creates a new version of a data item rather than overwriting the previous version in place. Each version carries metadata—typically transaction identifiers, timestamps, or version pointers—that the visibility subsystem uses to decide which version a given transaction is permitted to see. Read operations select the appropriate version according to a snapshot or visibility rule; write operations create new versions and mark older versions as obsolete when they are no longer visible to any active transaction. The technique was formalised for the database community by Bernstein and Goodman (1983), building on Reed's earlier multiversion timestamp ordering. MVCC is the dominant concurrency-control mechanism in PostgreSQL, MySQL InnoDB, Oracle, and SQL Server (via row versioning under snapshot isolation).

**Beginner-Friendly Explanation**  
Imagine a shared document where every edit creates a new copy instead of overwriting the old one. Readers keep reading their own copy, unaware that someone else has written a newer version. MVCC works the same way: when you `UPDATE` a row, the database does not destroy the old row immediately—it writes a new version and marks the old one as historical. Readers who started earlier continue to see the old version, while new readers see the new version. This is why, in MVCC databases, “reading never blocks writing and writing never blocks reading.”

### Key Characteristics

- **Non-blocking reads**: Plain `SELECT` statements do not acquire locks that conflict with writers; readers and writers operate on different versions.
- **Snapshot-based visibility**: Each statement or transaction sees a consistent snapshot of the database, determined by transaction identifiers or timestamps.
- **Version storage**: Old versions are kept in the table itself (PostgreSQL) or in a separate undo/version store (MySQL InnoDB, SQL Server, Oracle).
- **Garbage collection requirement**: Obsolete versions must be reclaimed by a background or manual process (`VACUUM` in PostgreSQL, `purge` in InnoDB, version-store cleanup in SQL Server).
- **Transaction-ID dependence**: Visibility decisions rely on comparing transaction IDs; in PostgreSQL, this introduces the wraparound problem because transaction IDs are finite.
- **Isolation-level mapping**: MVCC underpins `READ COMMITTED`, `REPEATABLE READ` (which maps to snapshot isolation in PostgreSQL and InnoDB), and `SERIALIZABLE` (via Serializable Snapshot Isolation in PostgreSQL).

### Prerequisites

- Familiarity with transactions (`BEGIN`, `COMMIT`, `ROLLBACK`) and ACID properties.
- Understanding of isolation levels and their anomalies.
- Basic knowledge of how `UPDATE` and `DELETE` affect rows.
- For hands-on examples: access to PostgreSQL 13+, MySQL 8.0+ (InnoDB), or SQL Server with snapshot isolation enabled.

### Related Programming Areas

- Transaction processing and isolation-level design.
- Database administration (vacuuming, bloat monitoring, transaction-ID maintenance).
- Application-level retry logic for serialization failures.
- Data warehousing and long-running analytical queries (where snapshots matter).
- Replication and point-in-time recovery (which interact with version retention).

---

## Core Concepts and Features

### 1. Multi-Version Concurrency Control (MVCC)

#### Definitions

**Core Definition**  
MVCC is the database architecture that preserves multiple versions of rows so that concurrent readers and writers can operate without blocking each other.

**Technical Definition**  
MVCC maintains version chains (or version stores) in which each write produces a new version tagged with the writing transaction’s identifier or timestamp. A read operation evaluates a visibility rule—such as “the version was created by a transaction that committed before my snapshot and was not deleted by a transaction visible to my snapshot”—to choose the correct version. In PostgreSQL, versions (tuples) are stored directly in the heap and carry `xmin` (creating transaction) and `xmax` (deleting transaction) system columns. In MySQL InnoDB, versions are stored in the undo log and chained via `DB_ROLL_PTR`; visibility is determined by a `ReadView`. In SQL Server, row versions are stored in `tempdb` and identified by sequence numbers when row-versioning isolation levels are enabled.

**Beginner-Friendly Explanation**  
MVCC is the database’s way of letting everyone read the book while someone else is writing the next edition. You get your own copy of the page as it was when you started reading, and the writer works on a new page.

#### Purposes

- To allow concurrent reads and writes without mutual blocking, maximising throughput in multiuser environments.
- To provide each transaction with a consistent view of the data for the duration of its snapshot.
- To avoid the reader-writer blocking inherent in two-phase locking (2PL) systems.
- To support statement-level and transaction-level read consistency.
- To enable non-blocking `SELECT` queries even while rows are being updated or deleted.

#### Syntax Rules and Structure

MVCC is not a single SQL statement; it is a storage and concurrency architecture. The “syntax” consists of the statements that interact with MVCC and the configuration parameters that control it.

**Core interacting statements**:

```sql
-- Reading under MVCC: a plain SELECT sees a snapshot.
SELECT * FROM table WHERE condition;

-- Writing under MVCC: UPDATE creates a new version.
UPDATE table SET column = value WHERE condition;

-- Deleting under MVCC: DELETE marks the old version as dead.
DELETE FROM table WHERE condition;

-- Transaction boundaries control snapshot duration.
BEGIN;
-- ...
COMMIT;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `SELECT` | Reads the version visible to the statement’s or transaction’s snapshot; does not block writers. |
| `UPDATE` | Creates a new row version; the previous version remains for older snapshots. |
| `DELETE` | Marks the previous version as deleted for newer snapshots; old snapshots may still see it. |
| `BEGIN` / `COMMIT` | Define the transaction whose snapshot governs visibility. |

**Syntax rules (vendor-specific)**

- **PostgreSQL**: Every `SELECT` sees a snapshot; under `READ COMMITTED`, the snapshot is taken per statement; under `REPEATABLE READ` and `SERIALIZABLE`, it is taken at the first statement of the transaction.
- **MySQL InnoDB**: Consistent nonlocking reads under `READ COMMITTED` and `REPEATABLE READ` use MVCC; `SELECT … FOR UPDATE` and `SELECT … FOR SHARE` are locking reads that bypass MVCC and read the latest committed version.
- **SQL Server**: MVCC-based isolation is opt-in via `ALLOW_SNAPSHOT_ISOLATION` and `READ_COMMITTED_SNAPSHOT`; without these options, SQL Server uses locking-based isolation.

**Constraints and limitations**

- MVCC requires storage for old versions; without garbage collection, tables and indexes bloat.
- Long-running transactions can prevent old versions from being reclaimed.
- In PostgreSQL, transaction-ID wraparound is a direct consequence of the 32-bit XID space used by MVCC.
- MVCC does not eliminate all conflicts: write-write conflicts still require locking or conflict detection (e.g., `UPDATE` on the same row).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – MVCC in action with two concurrent transactions

```sql
-- Session 1: Start a transaction and read a row.
BEGIN;
SELECT balance FROM accounts WHERE id = 1;
-- Output: 1000.00

-- Session 2 (concurrently): Update the same row and commit.
BEGIN;
UPDATE accounts SET balance = 900.00 WHERE id = 1;
COMMIT;

-- Session 1: Re-read the row within the same transaction.
SELECT balance FROM accounts WHERE id = 1;
-- Output: 1000.00 (still the old version, because Session 1's snapshot
-- was taken at the first SELECT under REPEATABLE READ/SERIALIZABLE,
-- or at statement start under READ COMMITTED for the first statement).

COMMIT;
```

**Expected output (Session 1, second SELECT)**:

```text
 balance
---------
 1000.00
(1 row)
```

**Why this output appears**: Under PostgreSQL’s default `READ COMMITTED`, each statement takes a new snapshot; the second `SELECT` would see the new value (900.00). Under `REPEATABLE READ` or `SERIALIZABLE`, the transaction’s snapshot is fixed at the first statement, so the second `SELECT` still sees 1000.00. The key point is that Session 2’s `UPDATE` was **not blocked** by Session 1’s read; MVCC allowed both to proceed.

##### Example 2: MySQL InnoDB – Consistent read without blocking

```sql
-- Session 1: Start a transaction.
START TRANSACTION;

-- Session 1: Consistent read of a row.
SELECT balance FROM accounts WHERE id = 1;
-- Output: 1000.00

-- Session 2: Update the same row (not blocked by Session 1's read).
START TRANSACTION;
UPDATE accounts SET balance = 900.00 WHERE id = 1;
COMMIT;

-- Session 1: Re-read the row under REPEATABLE READ.
SELECT balance FROM accounts WHERE id = 1;
-- Output: 1000.00 (the old version from the undo log).

COMMIT;
```

**Expected output (Session 1, second SELECT)**:

```text
+---------+
| balance |
+---------+
| 1000.00 |
+---------+
1 row in set (0.00 sec)
```

**Why this output appears**: InnoDB uses the undo log to reconstruct the version visible to Session 1’s `ReadView`. Session 2’s update created a new version, but Session 1’s snapshot still points to the old version.

#### Real-World Cases

- **High-concurrency OLTP**: MVCC allows order-processing systems to read inventory levels without blocking concurrent updates, improving throughput.
- **Reporting on live data**: Analysts run long `SELECT` queries against a transactional database without blocking application writes.
- **Mixed workloads**: MVCC decouples read and write concurrency, which is essential for HTAP (hybrid transactional/analytical processing) systems.

---

### 2. Snapshots

#### Definitions

**Core Definition**  
A snapshot is a consistent view of the database at a specific point in time, defined by the set of transactions that are visible to a reader.

**Technical Definition**  
A snapshot is a data structure that records the transaction identifiers (or timestamps) that are considered “in progress” or “too new” at the moment the snapshot is taken. In PostgreSQL, an MVCC snapshot contains `xmin` (the lowest XID still active), `xmax` (the first XID not yet assigned), and `xip` (the list of in-progress XIDs between `xmin` and `xmax`). In MySQL InnoDB, a `ReadView` contains `m_ids` (active transaction IDs), `min_trx_id`, `max_trx_id`, and `creator_trx_id`. In SQL Server, snapshot isolation uses a transaction sequence number to identify the version visible to the transaction.

**Beginner-Friendly Explanation**  
A snapshot is a frozen picture of the database that says “these transactions are done, these are still running, and these haven’t started yet.” When you read, the database checks this picture to decide which version of each row you are allowed to see.

#### Purposes

- To provide a consistent point-in-time view of the data for a transaction or statement.
- To define the visibility boundary for MVCC reads.
- To support repeatable reads and snapshot isolation without locking.
- To enable long-running queries to see stable data even while writes continue.

#### Syntax Rules and Structure

Snapshots are created implicitly; some RDBMSs expose functions to inspect them.

**PostgreSQL**:

```sql
-- Inspect the current snapshot.
SELECT pg_current_snapshot();
-- Example output: 100:105:100,102,104
-- Format: xmin:xmax:xip_list
```

**MySQL InnoDB**:

```sql
-- Start a consistent snapshot explicitly (optional).
START TRANSACTION WITH CONSISTENT SNAPSHOT;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `pg_current_snapshot()` | Returns the current transaction’s snapshot as `xmin:xmax:xip_list`. |
| `START TRANSACTION WITH CONSISTENT SNAPSHOT` | InnoDB syntax to establish a snapshot immediately. |

**Syntax rules**

- PostgreSQL: Under `READ COMMITTED`, a new snapshot is taken for each statement; under `REPEATABLE READ`/`SERIALIZABLE`, one snapshot is taken at the first statement and held for the transaction.
- MySQL InnoDB: Under `REPEATABLE READ`, the `ReadView` is created at the first consistent read and reused; under `READ COMMITTED`, a new `ReadView` is created for each statement.
- SQL Server: A snapshot transaction’s view is fixed at transaction start when `ALLOW_SNAPSHOT_ISOLATION` is on.

**Constraints and limitations**

- Snapshots consume memory (for the in-progress XID list) and must be managed.
- A snapshot does not prevent write-write conflicts; two transactions can still try to update the same row, and one will fail or wait.
- The lifetime of a snapshot is bounded by the transaction duration; long-lived snapshots prevent version cleanup.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Inspecting a snapshot

```sql
-- Step 1: Start a transaction.
BEGIN;

-- Step 2: Inspect the snapshot.
SELECT pg_current_snapshot();
-- Example output: 750:755:752,753

-- Step 3: Commit.
COMMIT;
```

**Expected output**:

```text
 pg_current_snapshot
---------------------
 750:755:752,753
(1 row)
```

**Why this output appears**: `xmin = 750` means transactions below 750 are visible (committed before the snapshot). `xmax = 755` means transactions at or above 755 are invisible. The XIDs `752` and `753` are in progress and therefore invisible. This snapshot governs which row versions the transaction can see.

#### Real-World Cases

- **Long-running reports**: A report takes a snapshot at start and reads consistent data for its entire duration, unaffected by concurrent writes.
- **Backup operations**: A backup may use a snapshot to ensure a consistent point-in-time view.
- **Replication**: Logical replication uses snapshots to establish a consistent starting point.

---

### 3. Visibility Rules

#### Definitions

**Core Definition**  
Visibility rules are the algorithms that decide whether a particular row version is visible to a given snapshot or transaction.

**Technical Definition**  
In PostgreSQL, a tuple (row version) is visible to a snapshot if its `xmin` is committed and before the snapshot’s `xmax`, and its `xmax` is either invalid (not deleted) or belongs to a transaction that is not visible to the snapshot. In MySQL InnoDB, the `ReadView` compares the row’s `DB_TRX_ID` against the view’s `m_ids`, `min_trx_id`, and `max_trx_id` to determine visibility; if the row is not visible, the undo log is traversed via `DB_ROLL_PTR`. In SQL Server snapshot isolation, a row version is visible if its sequence number is less than the snapshot’s sequence number and it was not created by an uncommitted transaction.

**Beginner-Friendly Explanation**  
Visibility rules are the “whose turn is it to be seen” logic. They look at the transaction IDs attached to each row version and the snapshot’s list of active transactions, and decide: “Can I see this version, or do I need to look at an older one?”

#### Purposes

- To determine which version of a row a reader should see.
- To enforce isolation guarantees (read committed, repeatable read, snapshot isolation).
- To prevent readers from seeing uncommitted or future data.
- To support consistent reads across multiple statements within a transaction.

#### Syntax Rules and Structure

Visibility rules are implemented internally; they are exposed through system columns and functions.

**PostgreSQL system columns**:

```sql
SELECT xmin, xmax, ctid, * FROM table WHERE condition;
```

**Component breakdown**

| Column | Description |
|--------|-------------|
| `xmin` | The transaction ID that created this row version. |
| `xmax` | The transaction ID that deleted or superseded this row version (0 if not deleted). |
| `ctid` | The physical location of the tuple. |

**MySQL InnoDB hidden columns**:

- `DB_TRX_ID`: Transaction ID of the last transaction that inserted or updated the row.
- `DB_ROLL_PTR`: Pointer to the undo log record for the previous version.

**Syntax rules**

- PostgreSQL: A version is visible if `xmin` is committed and `xmin < snapshot.xmax` and `xmin` is not in `snapshot.xip`; and `xmax` is either 0 or not visible to the snapshot.
- MySQL InnoDB: The `ReadView` visibility check determines whether to follow the undo chain.
- SQL Server: Snapshot isolation compares the row’s version sequence number with the transaction’s snapshot sequence number.

**Constraints and limitations**

- Visibility rules are engine-internal and cannot be overridden by users.
- Misunderstanding visibility can lead to surprising query results, especially under `READ COMMITTED` vs `REPEATABLE READ`.
- System columns like `xmin`/`xmax` are PostgreSQL-specific and not part of the SQL standard.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Inspecting `xmin` and `xmax`

```sql
-- Step 1: Create a test table.
CREATE TABLE visibility_demo (id int PRIMARY KEY, val text);

-- Step 2: Insert a row and inspect xmin.
INSERT INTO visibility_demo VALUES (1, 'initial');
SELECT xmin, xmax, * FROM visibility_demo;
-- Output: xmin = some XID, xmax = 0

-- Step 3: Update the row in a new transaction and inspect again.
UPDATE visibility_demo SET val = 'updated' WHERE id = 1;
SELECT xmin, xmax, * FROM visibility_demo;
-- Output: xmin = new XID, xmax = 0 (the old version now has xmax set)
```

**Expected output** (illustrative):

```text
 xmin | xmax | id |   val
------+------+----+---------
  800 |    0 |  1 | initial
(1 row)

 xmin | xmax | id |   val
------+------+----+---------
  801 |    0 |  1 | updated
(1 row)
```

**Why this output appears**: The `UPDATE` created a new tuple with `xmin = 801`; the old tuple (with `xmin = 800`) now has `xmax = 801` and is invisible to snapshots that see the new version. The `SELECT` returns only the visible version.

#### Real-World Cases

- **Debugging stale reads**: Understanding visibility helps explain why a session sees old data after another session has committed an update.
- **Audit and forensic analysis**: System columns can reveal when a row version was created or superseded.
- **Replication conflict resolution**: Visibility rules determine which version wins in logical replication.

---

### 4. Vacuum

#### Definitions

**Core Definition**  
`VACUUM` is the PostgreSQL utility that reclaims storage occupied by dead row versions and updates statistics, preventing table and index bloat.

**Technical Definition**  
`VACUUM` scans tables and indexes, identifies tuples whose `xmax` indicates deletion and whose deleting transaction is older than the oldest snapshot in the system (`OldestXmin`), and marks their space as reusable. It does **not** return space to the operating system in the standard form (unlike `VACUUM FULL`, which rewrites the table). `VACUUM` also updates the visibility map and freezes old transaction IDs when necessary. The `VACUUM` command has options for `FULL`, `FREEZE`, `ANALYZE`, and `VERBOSE`.

**Beginner-Friendly Explanation**  
Vacuum is the database’s cleanup crew. When you update or delete a row, the old version stays on disk until vacuum removes it. Without vacuum, the table becomes bloated with dead rows that take up space but are no longer visible.

#### Purposes

- To reclaim storage occupied by dead tuples for reuse by future rows.
- To prevent table and index bloat.
- To update planner statistics when combined with `ANALYZE`.
- To freeze old transaction IDs and prevent wraparound failures.
- To maintain the visibility map for index-only scans.

#### Syntax Rules and Structure

**PostgreSQL official syntax**:

```sql
VACUUM [ ( option [, ...] ) ] [ table_and_columns [, ...] ]

where option can be:
    FULL [ boolean ]
    FREEZE [ boolean ]
    VERBOSE [ boolean ]
    ANALYZE [ boolean ]
    DISABLE_PAGE_SKIPPING [ boolean ]
    SKIP_LOCKED [ boolean ]
    INDEX_CLEANUP { AUTO | ON | OFF }
    PROCESS_MAIN [ boolean ]
    PROCESS_TOAST [ boolean ]
    TRUNCATE [ boolean ]
    PARALLEL integer
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `VACUUM` | The base command. |
| `FULL` | Rewrites the table to compact it and return space to the OS (locks the table). |
| `FREEZE` | Freezes old tuples aggressively. |
| `VERBOSE` | Prints detailed progress. |
| `ANALYZE` | Updates statistics after vacuuming. |
| `table_and_columns` | Optional target; if omitted, all tables in the current database are vacuumed. |

**Syntax rules**

- `VACUUM` cannot run inside a transaction block.
- `VACUUM FULL` acquires an `ACCESS EXCLUSIVE` lock and rewrites the table; it should be used sparingly.
- Plain `VACUUM` can run concurrently with reads and writes.
- `VACUUM` skips pages that are not visible to any snapshot.

**Constraints and limitations**

- Standard `VACUUM` does not return space to the operating system; it only makes space reusable within the table.
- `VACUUM FULL` is much slower and blocks all access.
- Vacuum cannot remove tuples that are still visible to an active snapshot (long-running transactions block cleanup).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Manual `VACUUM` with `ANALYZE`

```sql
-- Step 1: Create a table and insert/delete rows.
CREATE TABLE vacuum_demo (id serial PRIMARY KEY, payload text);
INSERT INTO vacuum_demo (payload)
SELECT 'row ' || g FROM generate_series(1, 10000) g;
DELETE FROM vacuum_demo WHERE id <= 5000;

-- Step 2: Run VACUUM ANALYZE to reclaim dead tuples and update statistics.
VACUUM (ANALYZE, VERBOSE) vacuum_demo;
```

**Expected output** (excerpt from `VERBOSE`):

```text
INFO:  vacuuming "public.vacuum_demo"
INFO:  "vacuum_demo": found 5000 removable, 5000 nonremovable row versions in 45 out of 45 pages
DETAIL:  0 dead row versions cannot be removed yet.
...
INFO:  analyzing "public.vacuum_demo"
INFO:  "vacuum_demo": scanned 45 of 45 pages, containing 5000 live rows and 0 dead rows
```

**Why this output appears**: `VACUUM` identifies 5,000 removable dead tuples (the deleted rows) and marks their space for reuse. The `ANALYZE` component updates the planner statistics. The table’s physical size may not shrink, but the space is reusable.

##### Example 2: PostgreSQL – `VACUUM FULL` to reclaim disk space

```sql
-- Step 1: Check the table size before.
SELECT pg_size_pretty(pg_total_relation_size('vacuum_demo'));
-- Output: 1024 kB

-- Step 2: Run VACUUM FULL to rewrite and compact the table.
VACUUM FULL vacuum_demo;

-- Step 3: Check the size after.
SELECT pg_size_pretty(pg_total_relation_size('vacuum_demo'));
-- Output: 512 kB (approximately)
```

**Expected output**:

```text
 pg_size_pretty
----------------
 1024 kB
(1 row)

VACUUM

 pg_size_pretty
----------------
 512 kB
(1 row)
```

**Why this output appears**: `VACUUM FULL` rewrites the table into a new file, removing all dead space and returning it to the operating system. The table is much smaller but the operation required an exclusive lock.

#### Real-World Cases

- **Bulk delete cleanup**: After archiving and deleting millions of old rows, `VACUUM` reclaims the space for new data.
- **Post-migration maintenance**: After a large data migration, `VACUUM ANALYZE` refreshes statistics and cleans up dead tuples.
- **Transaction-ID freeze**: Regular `VACUUM FREEZE` prevents wraparound failures in long-running clusters.

---

### 5. Autovacuum

#### Definitions

**Core Definition**  
Autovacuum is the PostgreSQL background daemon that automatically runs `VACUUM` and `ANALYZE` on tables when they accumulate enough dead tuples or modifications.

**Technical Definition**  
The autovacuum launcher (`autovacuum launcher`) periodically wakes up (controlled by `autovacuum_naptime`, default 1 minute) and starts worker processes for databases and tables that meet their thresholds. Thresholds are defined by `autovacuum_vacuum_threshold` (default 50 dead tuples) plus `autovacuum_vacuum_scale_factor` (default 0.2, i.e., 20% of the table), and similarly for `ANALYZE` with `autovacuum_analyze_threshold` and `autovacuum_analyze_scale_factor`. Autovacuum also performs freezing to prevent transaction-ID wraparound.

**Beginner-Friendly Explanation**  
Autovacuum is a robot that automatically cleans up the database when it notices too many dead rows, so you don’t have to remember to run `VACUUM` yourself.

#### Purposes

- To automate the cleanup of dead tuples without manual intervention.
- To keep table and index bloat under control.
- To update statistics automatically for the query planner.
- To freeze old transaction IDs and prevent wraparound.
- To reduce administrative overhead for routine maintenance.

#### Syntax Rules and Structure

Autovacuum is configured via server parameters and per-table storage parameters.

**Key configuration parameters**:

```text
autovacuum = on                          -- Enable/disable autovacuum
autovacuum_naptime = '1min'              -- How often the launcher wakes up
autovacuum_vacuum_threshold = 50         -- Minimum dead tuples before vacuum
autovacuum_vacuum_scale_factor = 0.2     -- Fraction of table size added to threshold
autovacuum_analyze_threshold = 50        -- Minimum changes before analyze
autovacuum_analyze_scale_factor = 0.1    -- Fraction of table size for analyze
autovacuum_freeze_max_age = 200000000    -- Max XID age before forced freeze
autovacuum_max_workers = 3               -- Number of worker processes
```

**Per-table overrides**:

```sql
ALTER TABLE my_table SET (
    autovacuum_vacuum_scale_factor = 0.05,
    autovacuum_vacuum_threshold = 100
);
```

**Component breakdown**

| Parameter | Description |
|-----------|-------------|
| `autovacuum` | Global on/off switch. |
| `autovacuum_naptime` | Sleep interval between launcher cycles. |
| `autovacuum_vacuum_threshold` | Absolute minimum dead tuples. |
| `autovacuum_vacuum_scale_factor` | Proportional component based on table size. |
| `autovacuum_freeze_max_age` | XID age at which a table is force-vacuumed for freezing. |

**Syntax rules**

- Autovacuum must be enabled at the server level (`autovacuum = on`); it is on by default in modern PostgreSQL.
- Per-table settings override global settings.
- Autovacuum workers are subject to `autovacuum_max_workers`; tables queue for workers.
- Autovacuum cannot run on a table that is in the middle of a `VACUUM FULL`.

**Constraints and limitations**

- Autovacuum can be blocked by long-running transactions that hold old snapshots.
- Aggressive settings can cause I/O spikes; tuning is required on write-heavy systems.
- Autovacuum does not shrink tables; it only makes space reusable.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Configuring per-table autovacuum settings

```sql
-- Step 1: Create a high-churn table.
CREATE TABLE high_churn (id serial PRIMARY KEY, data text);

-- Step 2: Set aggressive autovacuum thresholds for this table.
ALTER TABLE high_churn SET (
    autovacuum_vacuum_scale_factor = 0.05,
    autovacuum_vacuum_threshold = 100,
    autovacuum_analyze_scale_factor = 0.02
);

-- Step 3: Verify the settings.
SELECT relname, reloptions
FROM pg_class
WHERE relname = 'high_churn';
```

**Expected output**:

```text
  relname   |                                      reloptions
------------+---------------------------------------------------------------------------------------
 high_churn | {autovacuum_vacuum_scale_factor=0.05,autovacuum_vacuum_threshold=100,
             autovacuum_analyze_scale_factor=0.02}
(1 row)
```

**Why this output appears**: The per-table settings override the global defaults, causing autovacuum to trigger more frequently for this table.

##### Example 2: PostgreSQL – Monitoring autovacuum activity

```sql
-- Step 1: Check autovacuum progress.
SELECT
    p.pid,
    a.query,
    p.phase,
    p.heap_blks_total,
    p.heap_blks_scanned,
    p.heap_blks_vacuumed
FROM pg_stat_progress_vacuum p
JOIN pg_stat_activity a ON a.pid = p.pid;
```

**Expected output** (illustrative):

```text
  pid  |              query              |     phase      | heap_blks_total | ...
-------+---------------------------------+----------------+-----------------+...
 12345 | autovacuum: VACUUM public.big_table | scanning heap |          100000 | ...
```

**Why this output appears**: `pg_stat_progress_vacuum` shows the progress of active autovacuum workers, including the phase and block counts.

#### Real-World Cases

- **Write-heavy OLTP**: Autovacuum keeps dead tuples under control without manual intervention.
- **Post-bulk-load maintenance**: After a large data load, autovacuum analyzes new data and updates statistics for the planner.
- **Wraparound prevention**: Autovacuum’s freeze operations run automatically, preventing transaction-ID wraparound.

---

### 6. Bloat

#### Definitions

**Core Definition**  
Bloat is the accumulation of dead row versions and unused space in tables and indexes, caused by MVCC’s retention of old versions that have not yet been cleaned up.

**Technical Definition**  
Table bloat is the difference between the physical size of a table and the size required to store its live rows. It arises when `UPDATE` and `DELETE` create dead tuples that `VACUUM` has not yet reclaimed, or when free space cannot be reused because of fill-factor settings or page fragmentation. Index bloat is the analogous accumulation of dead index entries. Bloat increases I/O, reduces cache efficiency, and can degrade query performance.

**Beginner-Friendly Explanation**  
Bloat is like a warehouse that keeps empty boxes from old shipments instead of throwing them away. The warehouse takes up more space than necessary, and finding the current stock becomes slower.

#### Purposes

- (Bloat is a problem to be monitored and mitigated, not a feature.)

#### Syntax Rules and Structure

Bloat is detected through queries on system catalogs and addressed with `VACUUM`, `VACUUM FULL`, or `pg_repack`.

**Common bloat-detection query (PostgreSQL)**:

```sql
SELECT
    relname,
    pg_size_pretty(pg_total_relation_size(oid)) AS total_size,
    n_dead_tup,
    n_live_tup,
    round(n_dead_tup * 100.0 / nullif(n_live_tup + n_dead_tup, 0), 2) AS dead_pct
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;
```

**Component breakdown**

| Column | Description |
|--------|-------------|
| `relname` | Table name. |
| `pg_total_relation_size` | Total disk size including indexes and TOAST. |
| `n_dead_tup` | Number of dead tuples (estimate). |
| `n_live_tup` | Number of live tuples (estimate). |
| `dead_pct` | Percentage of tuples that are dead. |

**Syntax rules**

- Bloat is not directly measured by a single function; `pg_stat_user_tables` provides estimates.
- `pgstattuple` extension offers more accurate measurements (`pgstattuple(regclass)`).
- `VACUUM FULL` rewrites the table and eliminates bloat but locks it.
- `pg_repack` rewrites tables online with minimal locking.

**Constraints and limitations**

- `n_dead_tup` is an estimate maintained by the statistics collector; it may be stale.
- `VACUUM` does not shrink the table; it only makes space reusable. To return space to the OS, `VACUUM FULL` or `pg_repack` is required.
- Index bloat requires `REINDEX` or `pg_repack` to fully resolve.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Detecting table bloat

```sql
-- Step 1: Query pg_stat_user_tables for dead-tuple estimates.
SELECT
    relname,
    n_live_tup,
    n_dead_tup,
    pg_size_pretty(pg_total_relation_size(relid)) AS total_size
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY n_dead_tup DESC;
```

**Expected output** (illustrative):

```text
 relname  | n_live_tup | n_dead_tup | total_size
----------+------------+------------+------------
 orders   |     500000 |     250000 | 2048 MB
 sessions |      20000 |      15000 | 128 MB
```

**Why this output appears**: The `orders` table has 250,000 dead tuples, indicating significant bloat. The `total_size` shows the physical disk footprint.

##### Example 2: PostgreSQL – Using `pgstattuple` for accurate bloat measurement

```sql
-- Step 1: Create the extension (if not already installed).
CREATE EXTENSION IF NOT EXISTS pgstattuple;

-- Step 2: Measure bloat in a table.
SELECT * FROM pgstattuple('orders');
```

**Expected output**:

```text
 table_len | tuple_count | tuple_len | tuple_percent | dead_tuple_count | dead_tuple_len | ...
-----------+-------------+-----------+---------------+------------------+----------------+...
 2147483648|      500000 | 800000000 |         37.25 |           250000 |     400000000 | ...
```

**Why this output appears**: `pgstattuple` scans the table and reports exact counts of live and dead tuples, providing a more reliable bloat assessment than the statistics collector estimates.

#### Real-World Cases

- **Performance degradation**: A heavily updated table becomes bloated, causing full scans to read more pages than necessary.
- **Disk-space pressure**: Bloat consumes disk space that could be used for other data; `VACUUM FULL` or `pg_repack` reclaims it.
- **Index-only scan inefficiency**: Index bloat increases the cost of index-only scans and can mislead the planner.

---

### 7. Transaction IDs

#### Definitions

**Core Definition**  
A transaction ID (XID in PostgreSQL, transaction ID in other systems) is a unique identifier assigned to each transaction, used by MVCC to determine version visibility and ordering.

**Technical Definition**  
In PostgreSQL, a transaction ID is a 32-bit unsigned integer assigned sequentially to each transaction that writes data. The XID space is circular: after approximately 4 billion transactions, the counter wraps around. PostgreSQL uses special frozen XIDs (`FrozenTransactionId`, `xid = 2`) to mark tuples that are visible to all transactions, preventing comparisons from becoming ambiguous after wraparound. In MySQL InnoDB, transaction IDs are 48-bit integers stored in the `DB_TRX_ID` hidden column. In SQL Server, snapshot isolation uses transaction sequence numbers rather than 32-bit XIDs.

**Beginner-Friendly Explanation**  
A transaction ID is like a ticket number given to every transaction. The database uses these numbers to figure out which version of a row is newer and who should see what. In PostgreSQL, the ticket numbers eventually run out and start over, which is why the database has to “freeze” old tickets.

#### Purposes

- To uniquely identify each transaction for visibility and ordering decisions.
- To tag row versions with the transaction that created or deleted them.
- To support snapshot comparisons and conflict detection.
- To enable freezing and wraparound prevention in PostgreSQL.

#### Syntax Rules and Structure

Transaction IDs are assigned automatically; they can be inspected through system functions.

**PostgreSQL**:

```sql
-- Current transaction ID.
SELECT txid_current();

-- XID age of a table (how close to wraparound).
SELECT relname, age(relfrozenxid) AS xid_age
FROM pg_class
WHERE relkind = 'r'
ORDER BY xid_age DESC;
```

**Component breakdown**

| Function/Column | Description |
|-----------------|-------------|
| `txid_current()` | Returns the current transaction’s XID (assigning one if needed). |
| `age(relfrozenxid)` | Number of transactions since the table was last frozen. |
| `relfrozenxid` | The oldest XID that has not been frozen for the table. |

**Syntax rules**

- XIDs are assigned only to transactions that write data; read-only transactions may not consume an XID in some versions.
- `txid_current()` forces the assignment of an XID.
- `age(relfrozenxid)` is a critical monitoring metric; high values indicate wraparound risk.

**Constraints and limitations**

- The 32-bit XID space is finite; wraparound is inevitable without freezing.
- XIDs are not unique over the long term; comparisons must account for wraparound.
- In PostgreSQL, `VACUUM FREEZE` and autovacuum’s freeze operations are essential for XID hygiene.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Inspecting XID age

```sql
-- Step 1: Check the XID age of user tables.
SELECT
    relname,
    age(relfrozenxid) AS xid_age,
    pg_size_pretty(pg_total_relation_size(oid)) AS size
FROM pg_class
WHERE relkind = 'r'
  AND relnamespace = 'public'::regnamespace
ORDER BY xid_age DESC;
```

**Expected output**:

```text
 relname  |  xid_age  |  size
----------+-----------+---------
 orders   | 150000000 | 2048 MB
 customers|  50000000 | 512 MB
```

**Why this output appears**: `orders` has an XID age of 150 million, approaching the autovacuum freeze threshold (default 200 million). Monitoring this value helps prevent wraparound.

#### Real-World Cases

- **Wraparound prevention**: DBAs monitor XID age and ensure autovacuum freezes tables before the age approaches 2 billion.
- **Transaction auditing**: XIDs can be used to correlate row versions with specific transactions for forensic analysis.
- **Replication**: Logical replication uses XIDs to track which transactions have been applied.

---

### 8. Wraparound

#### Definitions

**Core Definition**  
Transaction ID wraparound is the condition in PostgreSQL where the 32-bit XID counter wraps around to zero, causing old transactions to appear “in the future” and making their data invisible.

**Technical Definition**  
PostgreSQL’s XID space is a 32-bit circular counter. Comparisons between XIDs use modular arithmetic with a 2³¹ boundary: an XID is considered “older” if its signed 32-bit difference from another XID is negative. When the counter advances past 2³¹, the interpretation of “old” and “new” flips. To prevent this, PostgreSQL freezes old tuples by replacing their `xmin` with `FrozenTransactionId` (XID 2), which is treated as older than all normal XIDs. If freezing falls behind and the XID age approaches 2 billion, PostgreSQL refuses new write transactions to avoid catastrophic data loss.

**Beginner-Friendly Explanation**  
Imagine a clock that goes up to 12 and then starts over at 1. If you’re not careful, “11 o’clock” might look like it comes after “1 o’clock.” PostgreSQL’s transaction numbers have the same problem, and freezing is how the database avoids confusing old and new data.

#### Purposes

- (Wraparound is a failure mode to prevent, not a feature.)

#### Syntax Rules and Structure

Wraparound prevention is managed through `VACUUM FREEZE`, autovacuum, and monitoring.

**Key operations**:

```sql
-- Manually freeze a table.
VACUUM FREEZE table_name;

-- Check the database-wide XID age.
SELECT datname, age(datfrozenxid) AS xid_age
FROM pg_database
ORDER BY xid_age DESC;
```

**Component breakdown**

| Operation | Description |
|-----------|-------------|
| `VACUUM FREEZE` | Freezes all eligible tuples in a table, replacing their XIDs with the frozen XID. |
| `age(datfrozenxid)` | Database-level XID age; when it reaches ~2 billion, wraparound protection activates. |
| `autovacuum_freeze_max_age` | Threshold (default 200 million) at which autovacuum forces a freeze. |

**Syntax rules**

- `VACUUM FREEZE` is aggressive and may be slow; it is normally handled by autovacuum.
- `age(datfrozenxid)` should be monitored; values above 1.5 billion warrant immediate action.
- PostgreSQL will emit warnings when the age approaches the limit and refuse writes at the hard limit.

**Constraints and limitations**

- Long-running transactions prevent freezing because they may still need to see old tuples.
- `VACUUM FREEZE` cannot freeze tuples that are still visible to any active snapshot.
- Once wraparound occurs, data may become inaccessible and recovery may require single-user mode and manual intervention.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Monitoring database-level XID age

```sql
-- Step 1: Check the XID age of each database.
SELECT
    datname,
    age(datfrozenxid) AS xid_age,
    pg_size_pretty(pg_database_size(datname)) AS size
FROM pg_database
ORDER BY xid_age DESC;
```

**Expected output**:

```text
  datname   |  xid_age   |  size
------------+------------+---------
 production | 1800000000 | 50 GB
 analytics  |  500000000 | 20 GB
 postgres   |   10000000 | 8 MB
```

**Why this output appears**: The `production` database has an XID age of 1.8 billion, dangerously close to the 2 billion limit. Immediate vacuuming and freezing are required to prevent wraparound.

##### Example 2: PostgreSQL – Forcing a freeze

```sql
-- Step 1: Freeze a critical table manually.
VACUUM FREEZE VERBOSE production.orders;

-- Step 2: Re-check the database age.
SELECT datname, age(datfrozenxid) FROM pg_database WHERE datname = 'production';
```

**Expected output**:

```text
INFO:  aggressively vacuuming "production.orders"
INFO:  "orders": found 0 removable, 1000000 nonremovable row versions in 50000 pages
...
VACUUM

 datname    |  xid_age
------------+-----------
 production | 100000000
(1 row)
```

**Why this output appears**: `VACUUM FREEZE` replaced the old XIDs with the frozen XID, dramatically reducing the database’s XID age and averting wraparound risk.

#### Real-World Cases

- **Long-running clusters**: A PostgreSQL cluster that has processed billions of transactions must rely on autovacuum freezing to avoid wraparound.
- **Post-incident recovery**: If wraparound warnings are ignored, the database may stop accepting writes; recovery requires emergency vacuuming in single-user mode.
- **Cloud managed databases**: Providers like Azure and AWS enforce wraparound protection and may restrict certain operations to prevent catastrophic failures.

---

### 9. Long-Running Transactions

#### Definitions

**Core Definition**  
A long-running transaction is a transaction that remains open for an extended period, holding a snapshot that prevents vacuum from reclaiming old row versions.

**Technical Definition**  
In MVCC systems, a transaction’s snapshot defines the oldest XID that must remain visible. Until the transaction completes, vacuum (or purge) cannot remove any tuple that might still be visible to that snapshot. This causes dead tuples to accumulate, leading to bloat and, in PostgreSQL, preventing XID freezing and increasing wraparound risk. Long-running transactions also hold locks and can cause lock contention.

**Beginner-Friendly Explanation**  
A long-running transaction is like a reader who refuses to close an old copy of a book. The library can’t throw away that old copy until the reader is done, so old copies pile up.

#### Purposes

- (Long-running transactions are a problem to detect and mitigate, not a feature.)

#### Syntax Rules and Structure

Long-running transactions are detected through system views.

**PostgreSQL**:

```sql
SELECT pid, now() - xact_start AS duration, state, query
FROM pg_stat_activity
WHERE xact_start IS NOT NULL
ORDER BY duration DESC;
```

**MySQL InnoDB**:

```sql
SELECT trx_id, trx_started, TIMESTAMPDIFF(SECOND, trx_started, NOW()) AS age_seconds
FROM information_schema.innodb_trx
ORDER BY age_seconds DESC;
```

**Component breakdown**

| View | Description |
|------|-------------|
| `pg_stat_activity` | Shows session-level transaction start times and states. |
| `information_schema.innodb_trx` | Shows InnoDB transaction details, including start time. |

**Syntax rules**

- In PostgreSQL, `xact_start` is non-null for sessions with an open transaction.
- A transaction in `idle in transaction` state is especially problematic: it holds a snapshot but is not executing.
- `pg_terminate_backend(pid)` can be used to terminate a blocking transaction (with caution).

**Constraints and limitations**

- Terminating a transaction may cause application errors; coordinate with the application owner.
- `idle_in_transaction_session_timeout` can automatically terminate idle transactions.
- Long-running read-only transactions also prevent cleanup, even if they do not hold locks.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Identifying long-running transactions

```sql
-- Step 1: Find transactions older than 5 minutes.
SELECT
    pid,
    now() - xact_start AS duration,
    state,
    left(query, 80) AS query_snippet
FROM pg_stat_activity
WHERE xact_start IS NOT NULL
  AND now() - xact_start > interval '5 minutes'
ORDER BY duration DESC;
```

**Expected output**:

```text
  pid  |    duration    |        state        |          query_snippet
-------+----------------+---------------------+----------------------------------
 12345 | 00:15:30.123   | idle in transaction | SELECT * FROM orders WHERE ...
 67890 | 00:08:12.456   | active              | UPDATE accounts SET ...
```

**Why this output appears**: Both sessions have open transactions older than 5 minutes. The `idle in transaction` session is particularly dangerous because it holds a snapshot without doing useful work.

##### Example 2: PostgreSQL – Terminating a blocking transaction

```sql
-- Step 1: Terminate the offending session (use with caution).
SELECT pg_terminate_backend(12345);

-- Step 2: Verify it is gone.
SELECT pid, state FROM pg_stat_activity WHERE pid = 12345;
```

**Expected output**:

```text
 pg_terminate_backend
----------------------
 t
(1 row)

 pid | state
-----+-------
(0 rows)
```

**Why this output appears**: `pg_terminate_backend` forcibly ends the session, releasing its snapshot and allowing vacuum to proceed.

#### Real-World Cases

- **Connection-pool leaks**: An application that opens a transaction and forgets to commit or roll back leaves an `idle in transaction` session, blocking vacuum.
- **Analytical queries**: A reporting query that runs for hours holds a snapshot, preventing cleanup of dead tuples generated by OLTP traffic.
- **Batch jobs**: A batch process that updates millions of rows in a single transaction creates a long-running transaction and massive bloat.

---

### 10. Snapshot Isolation

#### Definitions

**Core Definition**  
Snapshot isolation is an isolation level in which a transaction reads from a consistent snapshot of the database taken at the start of the transaction, and write conflicts are detected at commit time.

**Technical Definition**  
Snapshot isolation (SI) guarantees that all reads in a transaction see the database as of the transaction’s start time. Write-write conflicts are prevented by first-committer-wins: if two concurrent transactions attempt to update the same row, the second transaction to commit is aborted. SI prevents dirty reads, non-repeatable reads, and phantom reads, but it allows write skew—a serialization anomaly that requires `SERIALIZABLE` (or Serializable Snapshot Isolation, SSI, in PostgreSQL) to prevent. SQL Server implements SI via row versioning in `tempdb` when `ALLOW_SNAPSHOT_ISOLATION` is enabled; PostgreSQL’s `REPEATABLE READ` is implemented as snapshot isolation; MySQL InnoDB’s `REPEATABLE READ` uses a consistent snapshot for reads but differs in locking behaviour for writes.

**Beginner-Friendly Explanation**  
Snapshot isolation means “I see the database exactly as it was when I started, and if someone else changes something I’m also changing, one of us will be told to try again.”

#### Purposes

- To provide repeatable reads without blocking writers.
- To eliminate reader-writer blocking in OLTP workloads.
- To prevent common anomalies (dirty reads, non-repeatable reads, phantoms) without the overhead of serializable locking.
- To support long-running reads with a stable view.

#### Syntax Rules and Structure

**PostgreSQL** (REPEATABLE READ is snapshot isolation):

```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
-- reads see a fixed snapshot
COMMIT;
```

**SQL Server**:

```sql
-- Enable snapshot isolation on the database.
ALTER DATABASE mydb SET ALLOW_SNAPSHOT_ISOLATION ON;

-- Use snapshot isolation in a transaction.
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
BEGIN TRANSACTION;
-- ...
COMMIT;
```

**MySQL InnoDB**:

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;
-- consistent reads use the same ReadView
COMMIT;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `ISOLATION LEVEL REPEATABLE READ` | PostgreSQL: maps to snapshot isolation. |
| `ALLOW_SNAPSHOT_ISOLATION ON` | SQL Server database option enabling SI. |
| `SET TRANSACTION ISOLATION LEVEL SNAPSHOT` | SQL Server session setting. |

**Syntax rules**

- PostgreSQL: `REPEATABLE READ` and `SERIALIZABLE` both use snapshots; `SERIALIZABLE` adds SSI conflict detection.
- SQL Server: SI must be enabled at the database level before it can be used.
- MySQL InnoDB: `REPEATABLE READ` uses a consistent snapshot for nonlocking reads, but locking reads (`FOR UPDATE`) read the latest committed version.

**Constraints and limitations**

- SI allows write skew; use `SERIALIZABLE` if write skew is unacceptable.
- SQL Server SI stores row versions in `tempdb`, which can grow large; monitor `tempdb` space.
- Long-running SI transactions prevent version cleanup, causing bloat in the version store.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Snapshot isolation under `REPEATABLE READ`

```sql
-- Session 1: Start a repeatable-read transaction.
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Session 1: Read a row.
SELECT balance FROM accounts WHERE id = 1;
-- Output: 1000.00

-- Session 2: Update the same row and commit.
BEGIN;
UPDATE accounts SET balance = 900.00 WHERE id = 1;
COMMIT;

-- Session 1: Re-read the row.
SELECT balance FROM accounts WHERE id = 1;
-- Output: 1000.00 (snapshot isolation holds)

COMMIT;
```

**Expected output (Session 1, second SELECT)**:

```text
 balance
---------
 1000.00
(1 row)
```

**Why this output appears**: Under `REPEATABLE READ`, Session 1’s snapshot was fixed at the first `SELECT`. Session 2’s committed update is invisible to that snapshot, so the second `SELECT` still returns 1000.00.

##### Example 2: SQL Server – Enabling and using snapshot isolation

```sql
-- Step 1: Enable snapshot isolation on the database.
ALTER DATABASE AdventureWorks2022 SET ALLOW_SNAPSHOT_ISOLATION ON;
GO

-- Step 2: Start a snapshot transaction.
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
GO
BEGIN TRANSACTION;
GO

-- Step 3: Read a row.
SELECT VacationHours FROM HumanResources.Employee WHERE BusinessEntityID = 4;
-- Output: 48

-- Step 4: Another session updates the row (not blocked).
-- (In another session: UPDATE ... SET VacationHours = 40 ...)

-- Step 5: Re-read within the snapshot transaction.
SELECT VacationHours FROM HumanResources.Employee WHERE BusinessEntityID = 4;
-- Output: 48 (still the old version)

COMMIT;
GO
```

**Expected output**:

```text
VacationHours
-------------
48

VacationHours
-------------
48
```

**Why this output appears**: SQL Server’s snapshot isolation reads the version stored in `tempdb` that was visible at transaction start; the concurrent update created a new version, but the snapshot transaction continues to see the old one.

#### Real-World Cases

- **OLTP with reporting**: Snapshot isolation allows reports to read consistent data without blocking order processing.
- **Web applications**: Snapshot isolation reduces blocking and deadlocks in high-concurrency web workloads.
- **Financial systems**: Snapshot isolation provides stable reads for calculations while writes proceed concurrently, though write skew must be carefully analysed.

---

## References

- PostgreSQL Documentation – Introduction to MVCC – https://www.postgresql.org/docs/current/mvcc-intro.html
- PostgreSQL Documentation – Explicit Locking – https://www.postgresql.org/docs/current/explicit-locking.html
- PostgreSQL Documentation – `VACUUM` – https://www.postgresql.org/docs/current/sql-vacuum.html
- PostgreSQL Documentation – Routine Vacuuming – https://www.postgresql.org/docs/current/routine-vacuuming.html
- PostgreSQL Documentation – Preventing Transaction ID Wraparound Failures – https://www.postgresql.org/docs/current/routine-vacuuming.html#VACUUM-FOR-WRAPAROUND
- PostgreSQL Documentation – `pg_stat_activity` – https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-ACTIVITY-VIEW
- PostgreSQL Wiki – Lock Monitoring – https://wiki.postgresql.org/wiki/Lock_Monitoring
- MySQL 8.4 Reference Manual – InnoDB Multi-Versioning – https://dev.mysql.com/doc/refman/8.4/en/innodb-multi-versioning.html
- MySQL 8.4 Reference Manual – InnoDB Undo Logs – https://dev.mysql.com/doc/refman/8.4/en/innodb-undo-logs.html
- Microsoft Learn – Transaction Locking and Row Versioning Guide – https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide
- Microsoft Learn – Row Versioning-based Isolation Levels – https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide#row_versioning
- Microsoft Learn – Snapshot Isolation in SQL Server – https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/sql/snapshot-isolation-in-sql-server
- Bernstein, P. A., & Goodman, N. (1983). Multiversion Concurrency Control—Theory and Algorithms. *ACM Transactions on Database Systems*, 8(4), 465–483. https://dl.acm.org/doi/10.1145/319996.319998
- Berenson, H., Bernstein, P., Gray, J., Melton, J., O’Neil, E., & O’Neil, P. (1995). A Critique of ANSI SQL Isolation Levels. *Proceedings of the 1995 ACM SIGMOD International Conference on Management of Data*, 1–10. https://doi.org/10.1145/223784.223785
- Ports, D. R. K., & Grittner, K. (2012). Serializable Snapshot Isolation in PostgreSQL. *Proceedings of the VLDB Endowment*, 5(12), 1850–1861. https://doi.org/10.14778/2367502.2367523
- Fekete, A., Liarokapis, D., O’Neil, E., O’Neil, P., & Shasha, D. (2004). Making Snapshot Isolation Serializable. *ACM Transactions on Database Systems*, 30(2), 492–528. https://doi.org/10.1145/1071610.1071615
- Oracle Database Concepts – Data Concurrency and Consistency – https://docs.oracle.com/en/database/oracle/oracle-database/23/cncpt/data-concurrency-and-consistency.html
- AWS – Reclaiming Storage Space by Vacuuming (Aurora PostgreSQL) – https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/limitless-vacuum.html