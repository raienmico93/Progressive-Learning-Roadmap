# Comprehensive Programming Cheat Sheet: Transaction Isolation

## Topic Overview

### Definitions

**Core Definition**  
Transaction isolation is the "I" in ACID—the property that determines how and when the changes made by one transaction become visible to other concurrent transactions, and what anomalies (dirty reads, non-repeatable reads, phantoms, lost updates, write skew) are permitted or prevented.

**Technical Definition**  
Transaction isolation is a concurrency-control setting defined by the SQL standard (SQL:1992) in terms of **phenomena** that must not occur at each level. The standard defines four levels: READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, and SERIALIZABLE. Engines implement these levels using locking, multiversion concurrency control (MVCC), or Serializable Snapshot Isolation (SSI). PostgreSQL maps READ UNCOMMITTED to READ COMMITTED and implements REPEATABLE READ as Snapshot Isolation (which prevents phantom reads, exceeding the standard's minimum). MySQL InnoDB supports all four levels with REPEATABLE READ as the default. SQL Server adds SNAPSHOT and READ_COMMITTED_SNAPSHOT as row-versioning-based levels. Oracle offers READ COMMITTED (default), SERIALIZABLE, and a READ-ONLY mode not in SQL92. Isolation level is set per transaction (`SET TRANSACTION ISOLATION LEVEL`) or per session, and must be chosen before the transaction begins.

**Beginner-Friendly Explanation**  
Isolation is about how much one transaction can "see" of another transaction's incomplete work. At the lowest level, you might see uncommitted changes (dirty reads). At the highest level, the database makes concurrent transactions behave as if they ran one after another. The trade-off is always the same: stronger isolation means more correctness guarantees but less concurrency and more retries. Choosing the right level is about matching the level of protection to what your application actually needs.

### Key Characteristics

- **Phenomenon-driven**: Each level is defined by which anomalies it prevents. The standard specifies minimum guarantees; engines may provide stronger ones.
- **Vendor-divergent**: The same level name can behave differently across engines (e.g., PostgreSQL's REPEATABLE READ prevents phantoms; MySQL's REPEATABLE READ uses next-key locks; SQL Server's READ COMMITTED uses locks unless `READ_COMMITTED_SNAPSHOT` is ON).
- **Performance-sensitive**: Higher isolation increases lock contention, abort rates, and retry overhead, reducing throughput under contention.
- **MVCC vs. locking**: PostgreSQL and Oracle use MVCC (readers don't block writers); MySQL InnoDB uses MVCC plus next-key locks; SQL Server defaults to locking-based READ COMMITTED unless row versioning is enabled.
- **Application responsibility**: SERIALIZABLE and REPEATABLE READ may abort transactions with serialization failures; the application must be prepared to retry.
- **Snapshot isolation is not serializable**: Snapshot isolation prevents dirty reads, non-repeatable reads, and phantoms, but permits **write skew**, which only SERIALIZABLE prevents.

### Prerequisites

- Understanding of `BEGIN`, `COMMIT`, `ROLLBACK`, and transaction boundaries.
- Familiarity with `SELECT`, `UPDATE`, `DELETE`, and locking reads (`SELECT ... FOR UPDATE`).
- Awareness of MVCC and how row versions are stored (e.g., PostgreSQL dead tuples, MySQL undo logs, SQL Server `tempdb` row versions).
- Knowledge of the application's consistency requirements and retry capabilities.

### Related Programming Areas

- Concurrency control and locking (row locks, gap locks, predicate locks).
- MVCC implementation and vacuum/cleanup (PostgreSQL `VACUUM`, MySQL purge).
- Serializable Snapshot Isolation (SSI) and serialization failure handling.
- Application transaction retry design.
- Distributed transactions and two-phase commit isolation.

---

## Core Concepts and Features

### 1. Read Uncommitted

#### Definitions

**Core Definition**  
READ UNCOMMITTED is the lowest isolation level: a transaction may see data written by other uncommitted transactions (dirty reads).

**Technical Definition**  
At READ UNCOMMITTED, the engine does not acquire shared locks for reads and does not enforce snapshot consistency. A `SELECT` can read rows that another transaction has modified but not yet committed. If the writer rolls back, the reader has seen data that never existed in a committed state. PostgreSQL does not implement true READ UNCOMMITTED; it silently promotes the request to READ COMMITTED because its MVCC architecture cannot produce dirty reads. MySQL InnoDB and SQL Server support it with locking hints or isolation-level settings.

**Beginner-Friendly Explanation**  
READ UNCOMMITTED means "show me everything, even work in progress that might be undone." It is fast because readers never wait, but you can see data that ultimately gets rolled back—like reading a draft that gets deleted.

#### Purposes

- To allow non-blocking reads in reporting or monitoring queries where approximate results are acceptable.
- To avoid reader-writer blocking when the application can tolerate dirty reads.
- To provide a baseline for understanding why higher levels are necessary.

#### Syntax Rules and Structure

**Setting the level**:

```sql
-- Standard / MySQL / SQL Server
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

-- SQL Server table hint
SELECT * FROM t WITH (NOLOCK);
```

**Component breakdown**:

| Component | Description |
|-----------|-------------|
| `SET TRANSACTION ISOLATION LEVEL` | Sets the level for the next transaction. |
| `READ UNCOMMITTED` | The level keyword. |
| `WITH (NOLOCK)` (SQL Server) | Table hint equivalent to READ UNCOMMITTED for that table. |

**Syntax rules**:

- PostgreSQL accepts the syntax but behaves as READ COMMITTED.
- MySQL supports it; InnoDB reads the latest row version without locking.
- SQL Server supports it; `NOLOCK` is a common but risky alternative.

**Constraints and limitations**:

- Dirty reads can produce results that were never committed, leading to incorrect application decisions.
- `NOLOCK` can also produce missing or duplicate rows due to page splits and allocation scans.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Dirty read with `NOLOCK`

```sql
-- Session A
BEGIN TRANSACTION;
UPDATE accounts SET balance = 0 WHERE id = 1;
-- Not yet committed.

-- Session B
SELECT balance FROM accounts WITH (NOLOCK) WHERE id = 1;
-- Returns 0 (dirty read).

-- Session A
ROLLBACK;
```

**Expected output** (Session B):

```text
balance
-------
      0
```

**Why this output appears**: Session B reads the uncommitted value `0`. When Session A rolls back, the true balance is restored, but Session B already saw a value that never existed permanently.

#### Real-World Cases

- **Monitoring dashboards**: Approximate counts where a few dirty rows are harmless.
- **Legacy reports**: Performance-critical queries that cannot tolerate reader blocking.

---

### 2. Read Committed

#### Definitions

**Core Definition**  
READ COMMITTED guarantees that a transaction sees only data committed before the **statement** (or, in Oracle, the **query**) began; it never sees uncommitted data, but different statements in the same transaction can see different committed snapshots.

**Technical Definition**  
READ COMMITTED is the default in PostgreSQL, Oracle, and SQL Server (locking-based unless `READ_COMMITTED_SNAPSHOT` is ON). In PostgreSQL, each `SELECT` sees a snapshot as of the instant the query begins; `UPDATE`/`DELETE` re-evaluate the `WHERE` clause against the latest committed version of the row if a concurrent update occurred. In Oracle, each query sees data committed before the query began, and readers never block writers. In MySQL, READ COMMITTED uses MVCC without gap locks, allowing phantoms. Non-repeatable reads and phantoms are possible at this level.

**Beginner-Friendly Explanation**  
READ COMMITTED means "you only see committed data, but if you look twice, you might see different things because someone committed a change between your two looks." It is the pragmatic default: safe enough for most OLTP work, but not stable enough for multi-query reports.

#### Purposes

- To provide a safe default that prevents dirty reads without the overhead of stronger isolation.
- To allow readers to see the latest committed data at statement start, which is often what OLTP operations want.
- To balance correctness and concurrency for high-throughput workloads.

#### Syntax Rules and Structure

**Setting the level**:

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- or simply BEGIN; (default in PostgreSQL, Oracle, SQL Server)
```

**Component breakdown**:

| Engine | Default? | Read behavior |
|--------|----------|---------------|
| PostgreSQL | Yes | Snapshot per statement |
| Oracle | Yes | Snapshot per query |
| SQL Server | Yes | Shared locks per statement (unless RCSI) |
| MySQL InnoDB | No (default RR) | Snapshot per statement |

**Syntax rules**:

- Must be set before the transaction begins (or as part of `BEGIN`).
- In PostgreSQL, a concurrent update causes the statement to re-evaluate the `WHERE` clause against the updated row, sometimes producing surprising results in complex predicates.
- In SQL Server, `READ_COMMITTED_SNAPSHOT ON` changes READ COMMITTED to use row versions instead of shared locks, eliminating reader-writer blocking but still allowing non-repeatable reads and phantoms.

**Constraints and limitations**:

- Non-repeatable reads and phantoms are permitted; the same query can return different results within one transaction.
- Complex search conditions may see inconsistent snapshots in PostgreSQL because updated rows are re-evaluated against the latest version.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Non-repeatable read at READ COMMITTED

```sql
-- Session A
BEGIN ISOLATION LEVEL READ COMMITTED;
SELECT balance FROM accounts WHERE id = 1;  -- returns 1000

-- Session B
UPDATE accounts SET balance = 900 WHERE id = 1;
COMMIT;

-- Session A re-reads:
SELECT balance FROM accounts WHERE id = 1;  -- returns 900
COMMIT;
```

**Expected output** (Session A):

```text
 balance
---------
    1000
(1 row)

 balance
---------
     900
(1 row)
COMMIT
```

**Why this output appears**: At READ COMMITTED, each statement sees a fresh snapshot. The second `SELECT` sees Session B's committed update, producing a non-repeatable read.

#### Real-World Cases

- **OLTP operations**: Individual row lookups and updates where each statement is independent.
- **Read-committed snapshot (SQL Server)**: Reporting queries that cannot tolerate blocking but can tolerate non-repeatable reads.
- **Default application workloads**: The pragmatic choice for most web applications.

---

### 3. Repeatable Read

#### Definitions

**Core Definition**  
REPEATABLE READ guarantees that all reads within a transaction see a stable snapshot taken at the start of the transaction (or the first non-transaction-control statement), preventing non-repeatable reads; phantom behavior depends on the engine.

**Technical Definition**  
In PostgreSQL, REPEATABLE READ is implemented as **Snapshot Isolation**: the transaction sees a snapshot as of the start of the first non-transaction-control statement, and concurrent updates to rows read by the transaction cause a serialization failure (`40001`) if the transaction attempts to modify them. PostgreSQL's REPEATABLE READ also prevents phantom reads (stronger than the SQL standard requires). In MySQL InnoDB, REPEATABLE READ is the default; consistent reads use the snapshot established by the first read, and locking reads use next-key locks to prevent phantoms in many cases. SQL Server's REPEATABLE READ uses shared locks held until transaction end, preventing non-repeatable reads but allowing phantoms.

**Beginner-Friendly Explanation**  
REPEATABLE READ means "once I look at something, it stays the same for the rest of my transaction." If someone else changes it, I either don't see the change or my transaction fails and retries. It is like taking a frozen snapshot of the data you read.

#### Purposes

- To provide a stable view for multi-statement reads (reports, analytics).
- To prevent non-repeatable reads without requiring full SERIALIZABLE locking.
- To allow long-running read-only transactions to see a consistent snapshot without blocking writers (in MVCC engines).

#### Syntax Rules and Structure

**Setting the level**:

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

**Component breakdown**:

| Engine | Implementation | Phantom prevention? |
|--------|----------------|---------------------|
| PostgreSQL | Snapshot Isolation | Yes (stronger than standard) |
| MySQL InnoDB | MVCC + next-key locks | Yes (for indexed range scans) |
| SQL Server | Shared locks held to end | No (phantoms allowed) |
| Oracle | Not a separate level (uses SERIALIZABLE) | N/A |

**Syntax rules**:

- Must be set before the transaction begins.
- In PostgreSQL and MySQL, the snapshot is taken at the first read, not at `BEGIN`.
- Applications must be prepared to retry on serialization failures (PostgreSQL `40001`, MySQL deadlock/lock wait timeout).

**Constraints and limitations**:

- Write skew is possible at this level (it is not serializable).
- SQL Server's REPEATABLE READ holds shared locks for the transaction duration, blocking writers.
- PostgreSQL's REPEATABLE READ can abort update transactions that conflict with concurrent commits.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Repeatable read prevents non-repeatable reads

```sql
-- Session A
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT balance FROM accounts WHERE id = 1;  -- returns 1000

-- Session B
UPDATE accounts SET balance = 900 WHERE id = 1;
COMMIT;

-- Session A re-reads:
SELECT balance FROM accounts WHERE id = 1;  -- still returns 1000
COMMIT;
```

**Expected output** (Session A):

```text
 balance
---------
    1000
(1 row)

 balance
---------
    1000
(1 row)
COMMIT
```

**Why this output appears**: Session A's snapshot is fixed at transaction start. Session B's update is not visible, so the second read returns the same value.

#### Real-World Cases

- **Report generation**: Multiple queries that must see a consistent snapshot of the data.
- **Analytics**: Long-running read-only transactions that cannot tolerate changing data.
- **MySQL default**: Most InnoDB workloads run at REPEATABLE READ by default.

---

### 4. Serializable

#### Definitions

**Core Definition**  
SERIALIZABLE guarantees that concurrent transactions produce the same effect as if they had executed one at a time in some order, preventing all anomalies including write skew.

**Technical Definition**  
PostgreSQL implements SERIALIZABLE using **Serializable Snapshot Isolation (SSI)** : it builds on Snapshot Isolation by tracking read-write dependencies and aborting transactions that would create a serialization anomaly. MySQL implements SERIALIZABLE by implicitly converting plain `SELECT` to `SELECT ... LOCK IN SHARE MODE` and using next-key locks, effectively locking reads. SQL Server uses locking-based serializable with range locks. Oracle's SERIALIZABLE uses snapshot-based isolation with rollback on serialization conflict; reads never block writes. Applications must be prepared to retry aborted transactions.

**Beginner-Friendly Explanation**  
SERIALIZABLE means "the database makes it look like transactions ran one after another, even if they actually overlapped." It is the strongest guarantee and the hardest to implement efficiently. If the database detects that two transactions would produce an inconsistent result, it aborts one and expects the application to retry.

#### Purposes

- To guarantee correctness for business rules that span multiple rows or tables (write skew prevention).
- To prevent all isolation anomalies without application-level locking.
- To support critical financial or inventory operations where any anomaly is unacceptable.

#### Syntax Rules and Structure

**Setting the level**:

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

**Component breakdown**:

| Engine | Implementation | Read blocking? | Retry on conflict? |
|--------|----------------|----------------|---------------------|
| PostgreSQL (SSI) | Predicate locks / SIREAD | No | Yes (`40001`) |
| MySQL InnoDB | Shared next-key locks | Yes | Deadlock/lock timeout |
| SQL Server | Range locks | Yes | Deadlock victim |
| Oracle | Snapshot + conflict detection | No | Yes ("Cannot serialize access") |

**Syntax rules**:

- Must be set before the transaction begins.
- PostgreSQL SSI may abort transactions even when they touch different rows, due to page-level predicate locks.
- Retry logic must wrap the entire transaction, not just the failed statement.

**Constraints and limitations**:

- Higher abort rates under contention; retries consume resources.
- Locking implementations (MySQL, SQL Server) block readers and writers, reducing concurrency.
- PostgreSQL SSI is not truly "no blocking" for writes; it detects conflicts at commit time.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Serializable prevents write skew

```sql
-- Setup: two doctors, each on call. Rule: at least one must remain on call.
-- Session 1
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT COUNT(*) FROM doctors WHERE on_call = true;  -- returns 2
UPDATE doctors SET on_call = false WHERE name = 'Alice';
-- Not yet committed.

-- Session 2
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT COUNT(*) FROM doctors WHERE on_call = true;  -- returns 2
UPDATE doctors SET on_call = false WHERE name = 'Bob';
COMMIT;  -- succeeds

-- Session 1
COMMIT;  -- fails with serialization error
```

**Expected output** (Session 1):

```text
ERROR:  could not serialize access due to read/write dependencies among transactions
DETAIL:  Reason code: Canceled on identification as a pivot, during commit attempt.
HINT:  The transaction might succeed if retried.
```

**Why this output appears**: Both transactions read `COUNT(*) = 2` and each updates a different row, so neither write conflicts. Snapshot Isolation would allow both to commit, leaving zero doctors on call (write skew). PostgreSQL's SSI detects the dangerous read-write dependency and aborts one transaction at commit time.

#### Real-World Cases

- **Financial ledgers**: Debits and credits that must balance across rows.
- **Inventory reservations**: Preventing overselling when multiple transactions check stock.
- **On-call scheduling**: Ensuring constraints that span multiple rows.

---

### 5. Snapshot Isolation

#### Definitions

**Core Definition**  
Snapshot Isolation (SI) is a concurrency-control model in which each transaction reads from a consistent snapshot of the database taken at the start of the transaction; writes are checked for conflicts at commit using a **first-committer-wins** rule.

**Technical Definition**  
SI is not one of the four SQL standard levels but is the implementation technique behind PostgreSQL's REPEATABLE READ and SQL Server's SNAPSHOT isolation. Each transaction reads the latest committed version of each row as of its snapshot timestamp. At commit, if another transaction has modified any row the current transaction also modified (write-write conflict), the current transaction aborts (first-committer-wins). SI prevents dirty reads, non-repeatable reads, lost updates, and phantoms, but permits **write skew** because it does not detect read-write conflicts. PostgreSQL's SERIALIZABLE extends SI with SSI to detect and abort write-skew scenarios.

**Beginner-Friendly Explanation**  
Snapshot Isolation is like giving each transaction its own private photocopy of the database at the moment it starts. Reads never block, and writes only fail if two transactions try to change the same row. But if two transactions each read one row and write a different row, both can succeed—even if the combined result violates a rule. That is write skew, and it is why SI alone is not serializable.

#### Purposes

- To provide non-blocking reads with a stable snapshot (no reader-writer blocking).
- To prevent lost updates via first-committer-wins.
- To serve as the foundation for Serializable Snapshot Isolation (SSI).
- To enable high-concurrency read-heavy workloads without locking overhead.

#### Syntax Rules and Structure

**Enabling SI**:

```sql
-- PostgreSQL: REPEATABLE READ is SI.
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- SQL Server: enable and use SNAPSHOT.
ALTER DATABASE db SET ALLOW_SNAPSHOT_ISOLATION ON;
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
```

**Component breakdown**:

| Feature | SI behavior |
|---------|-------------|
| Read consistency | Snapshot at transaction start |
| Write conflict | First-committer-wins abort |
| Read-write conflict | Not detected (write skew possible) |
| Phantom prevention | Yes (snapshot-based) |

**Syntax rules**:

- SI requires row versioning (PostgreSQL MVCC, SQL Server `tempdb` row versions, Oracle undo).
- `SELECT ... FOR UPDATE` acquires locks and can block, even under SI.
- Long-running SI transactions can prevent cleanup of old row versions (PostgreSQL `VACUUM`, Oracle undo retention).

**Constraints and limitations**:

- Write skew is permitted; applications must use SERIALIZABLE or explicit locks if write skew is unacceptable.
- First-committer-wins can cause aborts under write contention.
- SQL Server SNAPSHOT isolation requires `ALLOW_SNAPSHOT_ISOLATION ON` and adds `tempdb` overhead.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Snapshot Isolation allows write skew

```sql
-- Enable SI
ALTER DATABASE TestDB SET ALLOW_SNAPSHOT_ISOLATION ON;

-- Session 1
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
BEGIN TRANSACTION;
SELECT COUNT(*) FROM doctors WHERE on_call = 1;  -- returns 2
UPDATE doctors SET on_call = 0 WHERE name = 'Alice';

-- Session 2
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
BEGIN TRANSACTION;
SELECT COUNT(*) FROM doctors WHERE on_call = 1;  -- returns 2
UPDATE doctors SET on_call = 0 WHERE name = 'Bob';
COMMIT;  -- succeeds

-- Session 1
COMMIT;  -- also succeeds (write skew)
```

**Expected output**: Both transactions commit; the final state has zero doctors on call, violating the business rule.

**Why this output appears**: SI checks only write-write conflicts. The two transactions wrote different rows, so neither aborts. The read-write dependency (each read the other's row) is not detected, producing write skew.

#### Real-World Cases

- **Read-heavy reporting**: Non-blocking reads with a consistent snapshot.
- **High-concurrency OLTP**: Avoiding reader-writer blocking while preventing lost updates.
- **Foundation for SSI**: PostgreSQL SERIALIZABLE builds on SI to detect write skew.

---

### 6. Isolation Anomalies

#### 6.1 Dirty Reads

**Core Definition**: A transaction reads data written by another transaction that has not yet committed.

**Technical Definition**: Dirty reads occur at READ UNCOMMITTED when the engine does not enforce snapshot consistency or locking for reads. The reader can see uncommitted values that may later be rolled back. PostgreSQL does not permit dirty reads at any level because its MVCC architecture only exposes committed row versions.

**Beginner-Friendly Explanation**: Reading someone else's unsaved work.

**Syntax Rules**: Allowed only at READ UNCOMMITTED (and not in PostgreSQL). Prevented at READ COMMITTED and above.

**Code Example** (SQL Server):

```sql
-- Session A: UPDATE t SET x = 0 WHERE id = 1; (not committed)
-- Session B: SELECT x FROM t WITH (NOLOCK) WHERE id = 1; -- returns 0
-- Session A: ROLLBACK;
```

**Expected Output**: Session B reads `0` even though the final committed value is different.

**Why This Output Appears**: `NOLOCK` bypasses shared locks, allowing the read of an uncommitted row version.

**Real-World Cases**: Monitoring queries that tolerate approximate data.

---

#### 6.2 Non-Repeatable Reads

**Core Definition**: A transaction re-reads a row it previously read and finds that another transaction has modified or deleted it and committed.

**Technical Definition**: Non-repeatable reads are possible at READ COMMITTED because each statement sees a fresh snapshot. They are prevented at REPEATABLE READ and above, where the transaction sees a stable snapshot.

**Beginner-Friendly Explanation**: You look at a row, someone changes it and commits, and when you look again, it has changed.

**Syntax Rules**: Possible at READ COMMITTED; prevented at REPEATABLE READ, SNAPSHOT, and SERIALIZABLE.

**Code Example** (PostgreSQL – as shown in READ COMMITTED example).

**Expected Output**: The second `SELECT` returns a different value from the first.

**Why This Output Appears**: The second statement sees a newer snapshot that includes the concurrent commit.

**Real-World Cases**: Multi-step reports that need stable reads; use REPEATABLE READ.

---

#### 6.3 Phantom Reads

**Core Definition**: A transaction re-executes a query returning a set of rows that satisfy a search condition and finds that another transaction has inserted or deleted rows that satisfy the condition, changing the set.

**Technical Definition**: Phantom reads are possible at READ COMMITTED and (in SQL Server) at REPEATABLE READ. PostgreSQL's REPEATABLE READ (SI) prevents phantoms because the snapshot is fixed. MySQL InnoDB's REPEATABLE READ uses next-key locks for locking reads and a consistent snapshot for plain reads, preventing phantoms in many cases.

**Beginner-Friendly Explanation**: You count the rows matching a condition, someone inserts a new matching row and commits, and your next count is different.

**Syntax Rules**: Possible at READ COMMITTED; prevented at SNAPSHOT, PostgreSQL REPEATABLE READ, and SERIALIZABLE. SQL Server REPEATABLE READ allows phantoms.

**Code Example** (SQL Server – REPEATABLE READ allows phantoms):

```sql
-- Session A: SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
-- SELECT COUNT(*) FROM orders WHERE total > 100; -- returns 10
-- Session B: INSERT INTO orders (total) VALUES (200); COMMIT;
-- Session A: SELECT COUNT(*) FROM orders WHERE total > 100; -- returns 11
```

**Expected Output**: The count changes from 10 to 11.

**Why This Output Appears**: Shared locks prevent updates to existing rows but do not prevent inserts of new rows that satisfy the predicate.

**Real-World Cases**: Reports that must see a stable set of rows; use SNAPSHOT or SERIALIZABLE.

---

#### 6.4 Lost Updates

**Core Definition**: Two transactions read the same row, both modify it based on the read value, and one transaction's update overwrites the other's, causing the first update to be lost.

**Technical Definition**: Lost updates occur when a transaction reads a value, another transaction reads the same value, and both write back modifications without detecting the conflict. At READ COMMITTED and above, row-level locking for `UPDATE` prevents lost updates in most engines because the second `UPDATE` waits for the first to commit and then re-evaluates. Snapshot Isolation's first-committer-wins rule also prevents lost updates (if both write the same row, one aborts).

**Beginner-Friendly Explanation**: Two people read the same bank balance, both add a deposit, and one deposit overwrites the other—money disappears.

**Syntax Rules**: Prevented at READ COMMITTED and above for simple `UPDATE` statements (locking or first-committer-wins). Application-level read-modify-write without locking can still lose updates under SI if it reads outside the transaction or uses stale data.

**Code Example** (application-level lost update prevented by `UPDATE ... SET x = x + 1`):

```sql
-- Anti-pattern: read then write
SELECT balance FROM accounts WHERE id = 1; -- 1000
-- application computes 1000 + 100
UPDATE accounts SET balance = 1100 WHERE id = 1;

-- Preferred: atomic increment
UPDATE accounts SET balance = balance + 100 WHERE id = 1;
```

**Expected Output**: The atomic increment applies both updates correctly; the read-then-write pattern can lose one.

**Why This Output Appears**: `UPDATE ... SET balance = balance + 100` reads the current committed value under lock, while the application-level pattern uses a stale read.

**Real-World Cases**: Counter increments, inventory decrements.

---

#### 6.5 Write Skew

**Core Definition**: Write skew occurs when two transactions each read a set of rows, make disjoint writes (write to different rows), and both commit, producing a result that violates a constraint that would have been preserved by serial execution.

**Technical Definition**: Write skew is permitted under Snapshot Isolation because SI checks only write-write conflicts, not read-write dependencies. Each transaction reads data that the other writes, but they write different rows, so first-committer-wins does not fire. Only SERIALIZABLE (via SSI or locking) detects and prevents write skew.

**Beginner-Friendly Explanation**: Two doctors are on call. Each checks that the other is still on call, then goes off call. Both commit because they updated different rows—and now nobody is on call. The individual transactions were "correct" in isolation, but together they break the rule.

**Syntax Rules**: Allowed under Snapshot Isolation (PostgreSQL REPEATABLE READ, SQL Server SNAPSHOT). Prevented under SERIALIZABLE. `SELECT ... FOR UPDATE` can prevent it manually by locking the rows read.

**Code Example** (PostgreSQL – as shown in SERIALIZABLE example; under REPEATABLE READ both commits succeed, violating the rule).

**Expected Output**: Under REPEATABLE READ, both transactions commit; under SERIALIZABLE, one aborts with `40001`.

**Why This Output Appears**: SI does not track read-write dependencies; SSI does.

**Real-World Cases**: On-call scheduling, inventory reservation, financial constraints spanning rows.

---

### 7. Isolation Level Selection

#### Definitions

**Core Definition**: Isolation level selection is the process of choosing the weakest isolation level that still satisfies the application's consistency requirements, balancing correctness against concurrency and retry overhead.

**Technical Definition**: The selection depends on the anomalies the application can tolerate, the engine's implementation of each level, and the workload's contention profile. READ COMMITTED is the pragmatic default for OLTP; REPEATABLE READ/SNAPSHOT for multi-query stability; SERIALIZABLE for write-skew-sensitive business rules. Oracle's SERIALIZABLE is non-blocking for reads and may offer higher throughput under read/write contention than locking implementations. SQL Server's READ_COMMITTED_SNAPSHOT provides non-blocking reads at READ COMMITTED semantics.

**Beginner-Friendly Explanation**: Pick the weakest level that still prevents the anomalies that would break your application. If you can tolerate non-repeatable reads, use READ COMMITTED. If you need stable multi-query reads, use REPEATABLE READ or SNAPSHOT. If you have multi-row business rules, use SERIALIZABLE—and build retry logic.

#### Purposes

- To match isolation guarantees to actual application requirements.
- To avoid unnecessary locking and retry overhead from stronger levels.
- To ensure that critical invariants are protected by SERIALIZABLE while routine operations use weaker levels.
- To account for vendor-specific behavior (e.g., PostgreSQL's REPEATABLE READ prevents phantoms; SQL Server's does not).

#### Syntax Rules and Structure

**Decision matrix**:

| Requirement | Recommended level |
|-------------|-------------------|
| Single-row OLTP, can tolerate non-repeatable reads | READ COMMITTED |
| Stable multi-query snapshot, no phantoms | REPEATABLE READ / SNAPSHOT |
| Non-blocking reads, can tolerate write skew | SNAPSHOT |
| Multi-row business rules, no anomalies | SERIALIZABLE |

**Syntax rules**:

- Set the level before the transaction begins.
- Use the weakest level that meets requirements.
- For SERIALIZABLE, always implement retry logic for `40001` / deadlock errors.

**Constraints and limitations**:

- Vendor differences mean the same level name does not guarantee identical behavior.
- READ UNCOMMITTED is rarely appropriate for application logic; use only for monitoring.

---

### 8. Performance Implications

#### Definitions

**Core Definition**: Performance implications of isolation are the concurrency, latency, and resource costs associated with each level, including lock contention, abort/retry rates, row-version storage, and cleanup overhead.

**Technical Definition**: Stronger isolation increases the chance of conflicts, leading to blocking (locking implementations) or serialization aborts (MVCC/SSI implementations). Oracle notes that SERIALIZABLE requires rolling back and retrying aborted transactions, which can consume significant resources in high-contention environments. PostgreSQL applications using REPEATABLE READ or SERIALIZABLE must be prepared to retry on serialization failures. SI requires storing old row versions, increasing storage and cleanup load (PostgreSQL dead tuples, SQL Server `tempdb` row versions, Oracle undo). Locking-based levels (MySQL SERIALIZABLE, SQL Server REPEATABLE READ) block readers and writers, reducing concurrency.

**Beginner-Friendly Explanation**: Stronger isolation is safer but slower under contention. Transactions may wait for locks or be aborted and retried, and the database must store extra row versions for snapshot-based levels. The performance cost is not fixed—it depends on how much contention your workload has.

#### Purposes

- To understand the throughput/latency trade-off of isolation levels.
- To design retry logic that handles serialization failures gracefully.
- To choose between locking-based and MVCC-based implementations for a workload.
- To tune row-version cleanup and avoid bloat.

#### Syntax Rules and Structure

**Performance factors**:

| Factor | Lower isolation | Higher isolation |
|--------|-----------------|------------------|
| Lock contention | Low | High (locking) |
| Abort/retry rate | Low | High (SSI/serializable) |
| Row-version storage | Minimal | Significant (SI) |
| Read blocking | None (MVCC) | Possible (locking) |

**Syntax rules**:

- Retry loops should use exponential backoff with jitter.
- Keep transactions short to reduce conflict windows.
- Monitor abort rates and lock waits; adjust isolation or application logic.

**Constraints and limitations**:

- The cost of retries can exceed the cost of stronger isolation if retries are frequent.
- Row-version cleanup (vacuum/purge) must keep up with transaction rate.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Retry on serialization failure

```sql
-- Application retry loop (pseudo-code)
for attempt in 1..3:
    BEGIN;
    try:
        -- business logic at SERIALIZABLE
        COMMIT;
        break;
    except serialization_failure (40001):
        ROLLBACK;
        sleep(backoff with jitter);
        continue;
```

**Expected output**: The transaction succeeds on retry with a fresh snapshot, avoiding the conflict.

**Why this output appears**: PostgreSQL's SSI aborts one transaction with `40001`; the retry sees the committed change and can proceed without conflict.

#### Real-World Cases

- **High-contention OLTP**: READ COMMITTED with retry only on deadlocks.
- **Financial batch jobs**: SERIALIZABLE with robust retry and short transactions.
- **Reporting**: SNAPSHOT or REPEATABLE READ for stable reads without blocking writers.

---

## Summary: Isolation Level Matrix

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Write Skew | Lost Update |
|-------|------------|---------------------|--------------|------------|-------------|
| READ UNCOMMITTED | Possible | Possible | Possible | Possible | Possible |
| READ COMMITTED | No | Possible | Possible | Possible | No (locking) |
| REPEATABLE READ (SQL Server) | No | No | Possible | Possible | No |
| REPEATABLE READ (PostgreSQL/MySQL) | No | No | No (PG) / No (InnoDB) | Possible | No |
| SNAPSHOT | No | No | No | Possible | No |
| SERIALIZABLE | No | No | No | No | No |

---

## References

- PostgreSQL Documentation – Transaction Isolation – https://www.postgresql.org/docs/current/transaction-iso.html 
- PostgreSQL Documentation – `SET TRANSACTION` – https://www.postgresql.org/docs/current/sql-set-transaction.html
- PostgreSQL Wiki – Serializable Snapshot Isolation (SSI) – https://wiki.postgresql.org/wiki/SSI 
- MySQL 8.0 Reference Manual – Transaction Isolation Levels – https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html 
- Microsoft Learn – Understanding Transaction Isolation Levels – https://learn.microsoft.com/en-us/previous-versions/tn-archive/cc546518(v=technet.10) 
- Microsoft Learn – Snapshot Isolation – https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide
- Oracle Database Concepts – Data Concurrency and Consistency – https://docs.oracle.com/en/database/oracle/oracle-database/23/cncpt/data-concurrency-and-consistency.html 
- Oracle Database – Serializable Isolation and Retry – https://docs.oracle.com/cd/A87862_01/NT817CLI/server.817/a76965/c23cnsis.htm 
- Martin Kleppmann, Designing Data-Intensive Applications – Chapter 7: Transactions – https://dataintensive.net/
- Berenson et al., "A Critique of ANSI SQL Isolation Levels" – https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-95-51.pdf
- Cahill, Röhm, Fekete – "Serializable Snapshot Isolation" – https://arxiv.org/abs/cs/0703110
- PostgreSQL Wiki – Write Skew and SSI – https://wiki.postgresql.org/wiki/SSI#Write_Skew