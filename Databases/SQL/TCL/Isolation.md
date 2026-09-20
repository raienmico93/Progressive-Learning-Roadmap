# SQL Transaction Isolation Levels: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Transaction isolation levels define the degree to which one transaction is isolated from the effects of other concurrent transactions, controlling which concurrency anomalies (dirty reads, non-repeatable reads, phantom reads) are permitted or prevented.

**Technical Definition:** Per the SQL standard (ISO/IEC 9075-2), four isolation levels are defined: `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`, and `SERIALIZABLE`. The SQL standard defines these levels in terms of phenomena — interactions between concurrent transactions — that must not occur at each level. The most strict is `SERIALIZABLE`, defined by the standard as guaranteeing that any concurrent execution of a set of transactions produces the same effect as running them one at a time in some order. The other three levels are defined in terms of prohibited phenomena.

**Beginner-Friendly Explanation:** Isolation levels are like privacy settings for database transactions. When multiple transactions run at the same time, isolation levels control how much one transaction can "see" the other's unfinished work. Higher isolation means more consistency but less concurrency. Lower isolation means more speed but a higher risk of reading inconsistent data.

### Key Characteristics

- **Concurrency vs. consistency trade-off:** Higher isolation provides stronger consistency guarantees but reduces concurrency and increases locking overhead.
- **Phenomena-based definition:** The SQL standard defines levels in terms of which anomalies (dirty reads, non-repeatable reads, phantom reads) are possible at each level.
- **Implementation-dependent:** Actual behaviour varies significantly across RDBMSs; PostgreSQL's `READ UNCOMMITTED` behaves like `READ COMMITTED`, and its `REPEATABLE READ` prevents phantom reads.
- **Default level varies:** PostgreSQL and Oracle default to `READ COMMITTED`; MySQL InnoDB defaults to `REPEATABLE READ`; SQL Server defaults to `READ COMMITTED`.
- **Snapshot-based isolation:** Some databases (SQL Server, PostgreSQL at `REPEATABLE READ`/`SERIALIZABLE`, Babelfish) use MVCC to provide snapshot consistency without read locks.
- **Serialization anomaly:** The SQL standard also defines "serialization anomaly" as a phenomenon possible at levels below `SERIALIZABLE`.

### Prerequisites

- **Basic SQL knowledge:** Familiarity with `SELECT`, `INSERT`, `UPDATE`, and `DELETE`.
- **Transaction fundamentals:** Understanding of `BEGIN`, `COMMIT`, `ROLLBACK`, and ACID properties.
- **Concurrency concepts:** Awareness of how multiple sessions can access the same data simultaneously.
- **NULL and data type awareness:** Understanding of how data types affect comparison and locking behaviour.

### Related Programming Areas

- **Database Administration:** DBAs monitor and tune isolation levels to balance performance and consistency.
- **Application Development:** Backend services choose isolation levels based on business requirements.
- **Financial Systems:** Banking and payment systems typically require high isolation (SERIALIZABLE or REPEATABLE READ).
- **Data Warehousing and BI:** Reporting systems often use lower isolation (READ COMMITTED) for concurrency.
- **Distributed Systems:** Isolation levels interact with distributed transaction protocols and consistency models.

### Core Concepts / Features

1. READ UNCOMMITTED
2. READ COMMITTED
3. REPEATABLE READ
4. SERIALIZABLE
5. Snapshot-Based Isolation
6. Anomalies vs. Levels Mapping

---

## 1. READ UNCOMMITTED

### Definitions

**Core Definition:** `READ UNCOMMITTED` is the lowest isolation level, allowing transactions to read data that has been modified by other transactions but not yet committed, thereby permitting dirty reads.

**Technical Definition:** At this level, transactions are isolated only enough to ensure that physically corrupt data is not read. In PostgreSQL, `READ UNCOMMITTED` is accepted syntactically but internally behaves like `READ COMMITTED` because MVCC cannot expose uncommitted data. In SQL Server, `READ UNCOMMITTED` does not issue shared locks, so transactions can read uncommitted modifications. Oracle provides this level only as a non-standard extension.

**Beginner-Friendly Explanation:** `READ UNCOMMITTED` is the "fastest but dirtiest" isolation level. It lets you see changes that other transactions haven't finished making yet. This is like reading someone's draft notes before they've finalised them — you might see things they later change or delete.

### Purposes

- To maximise concurrency by avoiding read locks entirely.
- To allow reading data that is in the process of being modified without waiting.
- To support approximate analytics where exact consistency is not required.
- To provide the highest possible read throughput for non-critical queries.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

**Dialect-Specific Syntax:**

| Dialect | Syntax | Default? |
|---------|--------|----------|
| PostgreSQL | `SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;` | No (behaves as READ COMMITTED) |
| MySQL | `SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;` | No |
| SQL Server | `SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;` | No |
| Oracle | `SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;` | No (non-standard) |

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `SET TRANSACTION` | Initiates the isolation level setting |
| `ISOLATION LEVEL` | Specifies the type of isolation |
| `READ UNCOMMITTED` | The specific level name |

**Syntax Rules:**

- Must be issued before any DML statements in the transaction.
- In SQL Server, can also be applied per-query using the `NOLOCK` table hint (though this is not recommended).
- In PostgreSQL, `READ UNCOMMITTED` is treated as `READ COMMITTED`.
- In MySQL, `READ UNCOMMITTED` uses the lowest locking strategy.

**Constraints and Limitations:**

- Permits dirty reads, non-repeatable reads, phantom reads, and serialization anomalies.
- Not available as a true isolation level in Oracle (only as a non-standard extension).
- Results are non-deterministic and unsuitable for financial or critical data operations.
- Dirty reads can expose data that is later rolled back, leading to incorrect business decisions.

### Annotated Complete Code Examples

**Example 1: READ UNCOMMITTED in SQL Server**

```sql
-- Session 1: Begin a transaction and modify data without committing
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 500 WHERE account_id = 1;
-- No COMMIT yet

-- Session 2: Set READ UNCOMMITTED and read the uncommitted data
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT * FROM accounts WHERE account_id = 1;

-- Expected Output (Session 2 sees the uncommitted change):
--  account_id | balance
-- ------------+---------
--           1 |  500.00   -- (was 1000.00, now shows the uncommitted -500)

-- Session 1: Now ROLLBACK
ROLLBACK;

-- The change is undone, but Session 2 already read a "dirty" value
```

**Why this output occurs:** Session 2 uses `READ UNCOMMITTED` and reads the modified but uncommitted balance from Session 1. If Session 1 then rolls back, Session 2's read was based on data that never officially existed — a dirty read.

### Real-World Cases

**Case 1: Approximate Analytics**

A data analyst runs a quick count of rows in a large table using `READ UNCOMMITTED` to avoid blocking write operations, accepting that the count may be slightly off due to in-flight transactions.

**Case 2: Monitoring and Diagnostics**

A DBA uses `READ UNCOMMITTED` to inspect current activity without being blocked by long-running write transactions.

### References

- Microsoft Learn — SQL Server Transaction Locking and Row Versioning Guide (READ UNCOMMITTED) - https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide
- PostgreSQL Documentation — Transaction Isolation (Table 13.1) - https://www.postgresql.org/docs/16/transaction-iso.html
- MySQL 8.0 Reference Manual — Transaction Isolation Levels - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/innodb-transaction-isolation-levels.html
- Oracle Database Concepts — Oracle Transaction Isolation Levels - https://docs.oracle.com/cd/F19136_01/cncpt.pdf

---

## 2. READ COMMITTED

### Definitions

**Core Definition:** `READ COMMITTED` prevents dirty reads by ensuring that a transaction only sees data committed before the query began, but it permits non-repeatable reads and phantom reads because successive queries can see different snapshots.

**Technical Definition:** At this level, a `SELECT` query sees only data committed before the query began; it never sees uncommitted data or changes committed by concurrent transactions during the query's execution. In effect, a `SELECT` sees a snapshot of the database as of the instant the query begins. However, two successive `SELECT` commands can see different data if other transactions commit changes between them. In SQL Server, the Database Engine keeps write locks until the end of the transaction but releases read locks as soon as the read operation completes.

**Beginner-Friendly Explanation:** `READ COMMITTED` is like reading a newspaper that gets updated between editions. You only read stories that have been finalised (committed), but if you pick up a new edition later, the stories may have changed. You won't read half-finished drafts (no dirty reads), but you might see different numbers if you read the same report twice.

### Purposes

- To prevent dirty reads while maintaining reasonable concurrency.
- To provide a balanced default isolation level suitable for most applications.
- To allow read operations to proceed without waiting for concurrent writers.
- To serve as the default isolation level in PostgreSQL, SQL Server, and Oracle.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

**Dialect-Specific Syntax:**

| Dialect | Syntax | Default? |
|---------|--------|----------|
| PostgreSQL | `SET TRANSACTION ISOLATION LEVEL READ COMMITTED;` | ✅ Yes |
| MySQL | `SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;` | No (default is REPEATABLE READ) |
| SQL Server | `SET TRANSACTION ISOLATION LEVEL READ COMMITTED;` | ✅ Yes |
| Oracle | `SET TRANSACTION ISOLATION LEVEL READ COMMITTED;` | ✅ Yes |

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `SET TRANSACTION` | Initiates the isolation level setting |
| `ISOLATION LEVEL` | Specifies the type of isolation |
| `READ COMMITTED` | The specific level name |

**Syntax Rules:**

- Must be issued before any DML statements in the transaction.
- In SQL Server, `READ_COMMITTED_SNAPSHOT` database option (when ON) makes `READ COMMITTED` use row versioning instead of shared locks.
- In PostgreSQL, `READ COMMITTED` is the default level and uses MVCC snapshot isolation per statement.

**Constraints and Limitations:**

- Permits non-repeatable reads: reading the same row twice can yield different values.
- Permits phantom reads: re-executing a query can return different sets of rows.
- Permits serialization anomalies.
- Does not prevent write skew (a serialization anomaly).

### Annotated Complete Code Examples

**Example 1: READ COMMITTED in PostgreSQL**

```sql
-- Session 1: Begin a transaction
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

SELECT balance FROM accounts WHERE account_id = 1;
-- Output: 1000.00

-- Session 2: Update and commit
UPDATE accounts SET balance = 900 WHERE account_id = 1;
COMMIT;

-- Session 1: Read again (same transaction)
SELECT balance FROM accounts WHERE account_id = 1;
-- Output: 900.00  -- Non-repeatable read!

COMMIT;
```

**Why this output occurs:** Session 1's second `SELECT` sees the committed change from Session 2 because `READ COMMITTED` takes a fresh snapshot for each statement. The value changed within the same transaction — a non-repeatable read.

### Real-World Cases

**Case 1: Web Application Default**

A web application uses `READ COMMITTED` as its default isolation level, allowing high concurrency for read-heavy workloads while preventing dirty reads.

**Case 2: Reporting with Acceptable Variability**

A reporting system uses `READ COMMITTED` for dashboards where minor variations between refreshes are acceptable.

### References

- PostgreSQL Documentation — Transaction Isolation (READ COMMITTED) - https://www.postgresql.org/docs/16/transaction-iso.html
- Microsoft Learn — SQL Server Transaction Locking and Row Versioning Guide (READ COMMITTED) - https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide
- MySQL 8.0 Reference Manual — Transaction Isolation Levels - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/innodb-transaction-isolation-levels.html
- Oracle Database Concepts — Oracle Transaction Isolation Levels - https://docs.oracle.com/cd/F19136_01/cncpt.pdf

---

## 3. REPEATABLE READ

### Definitions

**Core Definition:** `REPEATABLE READ` prevents dirty reads and non-repeatable reads by ensuring that any row read during a transaction can be read again with the same values, but it may still permit phantom reads (new rows matching a query condition).

**Technical Definition:** At this level, a transaction's `SELECT` queries see a consistent snapshot of the data as of the first read operation in the transaction. In PostgreSQL, `REPEATABLE READ` only sees data committed before the transaction began. In MySQL InnoDB (the default level), `REPEATABLE READ` uses consistent reads based on a snapshot established by the first read. In SQL Server, the Database Engine keeps shared locks on selected data until the end of the transaction, preventing other transactions from modifying the data, but phantom reads can still occur because range locks are not managed.

**Beginner-Friendly Explanation:** `REPEATABLE READ` is like taking a photograph of your data at the start of the transaction. Every time you look at the photo, it shows the same thing — you won't see any changes made by other transactions during your transaction. However, if you look at a different part of the photo (a range query), you might see new items that weren't there before (phantom reads).

### Purposes

- To prevent non-repeatable reads within a transaction.
- To provide a stable, consistent view of data across multiple queries in a transaction.
- To serve as the default isolation level in MySQL InnoDB.
- To balance consistency with concurrency better than `SERIALIZABLE`.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

**Dialect-Specific Syntax:**

| Dialect | Syntax | Default? |
|---------|--------|----------|
| PostgreSQL | `SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;` | No (default is READ COMMITTED) |
| MySQL | `SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;` | ✅ Yes (InnoDB default) |
| SQL Server | `SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;` | No |
| Oracle | Not supported as a separate level | N/A |

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `SET TRANSACTION` | Initiates the isolation level setting |
| `ISOLATION LEVEL` | Specifies the type of isolation |
| `REPEATABLE READ` | The specific level name |

**Syntax Rules:**

- Must be issued before any DML statements in the transaction.
- In PostgreSQL, `REPEATABLE READ` uses MVCC to provide a transaction-level snapshot.
- In MySQL InnoDB, `REPEATABLE READ` uses consistent reads and gap locks to prevent phantom reads in many cases.
- In SQL Server, `REPEATABLE READ` acquires shared locks on read data until the transaction ends, but does not use range locks.

**Constraints and Limitations:**

- Permits phantom reads (in most implementations; PostgreSQL's implementation prevents them).
- In SQL Server, `REPEATABLE READ` can cause blocking because shared locks are held.
- Permits serialization anomalies (write skew).
- Oracle does not provide this level as a separate option; `SERIALIZABLE` provides stronger guarantees.

### Annotated Complete Code Examples

**Example 1: REPEATABLE READ in MySQL InnoDB**

```sql
-- Session 1: Set REPEATABLE READ and begin
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN;

SELECT balance FROM accounts WHERE account_id = 1;
-- Output: 1000.00

-- Session 2: Update and commit
UPDATE accounts SET balance = 900 WHERE account_id = 1;
COMMIT;

-- Session 1: Read again
SELECT balance FROM accounts WHERE account_id = 1;
-- Output: 1000.00  -- Same value! (No non-repeatable read)

COMMIT;
```

**Why this output occurs:** Session 1's snapshot was established at the first `SELECT`. The committed change by Session 2 is not visible because `REPEATABLE READ` provides a consistent snapshot for the entire transaction.

### Real-World Cases

**Case 1: Financial Applications**

A banking application uses `REPEATABLE READ` to ensure that account balances read at the start of a transaction remain consistent throughout, preventing non-repeatable reads.

**Case 2: Data Validation Pipelines**

An ETL process uses `REPEATABLE READ` to read a consistent set of source data for transformation, ensuring that the data doesn't change mid-process.

### References

- MySQL 8.0 Reference Manual — Transaction Isolation Levels - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/innodb-transaction-isolation-levels.html
- PostgreSQL Documentation — Transaction Isolation (REPEATABLE READ) - https://www.postgresql.org/docs/16/transaction-iso.html
- Microsoft Learn — SQL Server Transaction Locking and Row Versioning Guide (REPEATABLE READ) - https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide
- Oracle Database Concepts — Oracle Transaction Isolation Levels - https://docs.oracle.com/cd/F19136_01/cncpt.pdf

---

## 4. SERIALIZABLE

### Definitions

**Core Definition:** `SERIALIZABLE` is the highest isolation level, guaranteeing that concurrent transactions produce the same effect as if they had been executed one after another in some sequential order.

**Technical Definition:** The SQL standard defines `SERIALIZABLE` by stating that any concurrent execution of a set of Serializable transactions is guaranteed to produce the same effect as running them one at a time in some order. At this level, none of the phenomena (dirty reads, non-repeatable reads, phantom reads, serialization anomalies) are possible. In SQL Server, `SERIALIZABLE` acquires range locks when a `SELECT` uses a `WHERE` clause range to prevent phantom reads. In PostgreSQL, `SERIALIZABLE` uses predicate locking or serializable snapshot isolation (SSI) to detect and prevent serialization anomalies.

**Beginner-Friendly Explanation:** `SERIALIZABLE` is the strictest isolation level. It makes concurrent transactions behave as if they were running one after another in a single file. No transaction can see another's unfinished work, and no two transactions can interfere with each other's data. This is the safest level but the slowest because transactions must wait for each other.

### Purposes

- To provide the strongest possible consistency guarantees.
- To prevent all concurrency anomalies: dirty reads, non-repeatable reads, phantom reads, and serialization anomalies.
- To satisfy the most stringent ACID requirements.
- To support critical financial, healthcare, and regulatory applications.
- To enable correct reasoning about concurrent behaviour.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

**Dialect-Specific Syntax:**

| Dialect | Syntax | Default? |
|---------|--------|----------|
| PostgreSQL | `SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;` | No |
| MySQL | `SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;` | No |
| SQL Server | `SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;` | No |
| Oracle | `SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;` | No |

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `SET TRANSACTION` | Initiates the isolation level setting |
| `ISOLATION LEVEL` | Specifies the type of isolation |
| `SERIALIZABLE` | The specific level name |

**Syntax Rules:**

- Must be issued before any DML statements in the transaction.
- In SQL Server, `SERIALIZABLE` acquires range locks for `SELECT` statements with range `WHERE` clauses.
- In PostgreSQL, `SERIALIZABLE` may raise a serialization failure error if a serialization anomaly is detected; applications must retry the transaction.
- In Oracle, `SERIALIZABLE` is implemented as a form of snapshot isolation with additional checks.

**Constraints and Limitations:**

- Lowest concurrency: transactions may block each other extensively.
- Highest risk of deadlocks and serialization failures.
- Requires application-level retry logic in PostgreSQL and other MVCC-based systems.
- Can significantly reduce throughput for write-heavy workloads.
- In SQL Server, `SERIALIZABLE` can cause DDL and replication operations to fail.

### Annotated Complete Code Examples

**Example 1: SERIALIZABLE in PostgreSQL with Retry Logic**

```sql
-- Application-level retry pattern
-- BEGIN;
-- SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- ... perform operations ...
-- COMMIT;
-- If a serialization failure occurs, ROLLBACK and retry the entire transaction.

-- PostgreSQL automatically detects serialization anomalies
BEGIN;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

COMMIT;
-- If another transaction conflicts, PostgreSQL raises:
-- ERROR: could not serialize access due to read/write dependencies among transactions
-- DETAIL: Reason code: Canceled on identification as a pivot, during commit attempt.
-- HINT: The transaction might succeed if retried.
```

**Why this works:** PostgreSQL's `SERIALIZABLE` uses Serializable Snapshot Isolation (SSI) to detect when concurrent transactions would produce a non-serializable outcome. It aborts one of the transactions with a serialization failure, and the application must retry.

### Real-World Cases

**Case 1: Banking Transfers**

A bank uses `SERIALIZABLE` for account transfers to ensure that the sum of balances is always consistent, even under concurrent transfers.

**Case 2: Inventory Management**

An e-commerce system uses `SERIALIZABLE` for inventory reservation to prevent overselling when multiple customers purchase the same item simultaneously.

### References

- PostgreSQL Documentation — Transaction Isolation (SERIALIZABLE) - https://www.postgresql.org/docs/16/transaction-iso.html
- Microsoft Learn — SQL Server Transaction Locking and Row Versioning Guide (SERIALIZABLE) - https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide
- MySQL 8.0 Reference Manual — Transaction Isolation Levels - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/innodb-transaction-isolation-levels.html
- Oracle Database Concepts — Oracle Transaction Isolation Levels - https://docs.oracle.com/cd/F19136_01/cncpt.pdf

---

## 5. Snapshot-Based Isolation

### Definitions

**Core Definition:** Snapshot isolation is a concurrency control mechanism that provides each transaction with a consistent snapshot of the database as of a specific point in time, without acquiring read locks, using Multi-Version Concurrency Control (MVCC).

**Technical Definition:** Snapshot isolation provides time-travel snapshot consistency without reading locks. Each transaction sees a snapshot of the database as of the time the transaction (or statement) begins. In SQL Server, `READ_COMMITTED_SNAPSHOT` (RCSI) uses row versioning to provide statement-level snapshot consistency for `READ COMMITTED`, while `ALLOW_SNAPSHOT_ISOLATION` enables the `SNAPSHOT` isolation level for transaction-level snapshot consistency. In PostgreSQL, `REPEATABLE READ` and `SERIALIZABLE` use MVCC to provide transaction-level snapshot isolation. In MySQL InnoDB, `REPEATABLE READ` uses consistent reads based on a snapshot established by the first read. Snapshot isolation prevents dirty reads, non-repeatable reads, and read skew, but permits write skew (a serialization anomaly) unless combined with serializable checks.

**Beginner-Friendly Explanation:** Snapshot isolation is like having a time machine for your database. When you start a transaction, you get a frozen picture of the data as it was at that moment. You can read the same data repeatedly and always see the same values, even if other transactions are making changes. You don't have to wait for anyone, and nobody has to wait for you. The catch is that two transactions might make conflicting changes based on the same snapshot, and neither might notice (write skew).

### Purposes

- To provide consistent reads without blocking writers or readers.
- To enable high concurrency for read-heavy workloads.
- To offer a middle ground between `READ COMMITTED` and `SERIALIZABLE`.
- To serve as the foundation for MVCC-based isolation levels.
- To support time-travel queries and historical reporting.

### Syntax Rules and Structure

**Complete General Syntax (SQL Server):**

```sql
-- Enable snapshot isolation at the database level
ALTER DATABASE database_name SET ALLOW_SNAPSHOT_ISOLATION ON;

-- Enable RCSI for READ COMMITTED
ALTER DATABASE database_name SET READ_COMMITTED_SNAPSHOT ON;

-- Use SNAPSHOT isolation
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
```

**Dialect-Specific Syntax:**

| Dialect | Snapshot Mechanism | Level Name |
|---------|-------------------|------------|
| SQL Server | Row versioning | `SNAPSHOT` (transaction-level), RCSI (`READ COMMITTED` statement-level) |
| PostgreSQL | MVCC | `REPEATABLE READ` (transaction-level snapshot) |
| MySQL InnoDB | MVCC | `REPEATABLE READ` (consistent reads) |
| Oracle | MVCC | `READ COMMITTED` (statement-level snapshot) |
| SAP ASE | MVCC | Mapped from ANSI levels to statement or transaction snapshot |

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `ALLOW_SNAPSHOT_ISOLATION` | Enables transaction-level snapshot isolation (SQL Server) |
| `READ_COMMITTED_SNAPSHOT` | Enables statement-level snapshot for READ COMMITTED (SQL Server) |
| `SET TRANSACTION ISOLATION LEVEL SNAPSHOT` | Activates snapshot isolation for the transaction |

**Syntax Rules:**

- In SQL Server, snapshot isolation must be enabled at the database level before use.
- RCSI changes the behaviour of `READ COMMITTED` to use row versioning instead of shared locks.
- In PostgreSQL, `REPEATABLE READ` automatically provides snapshot isolation without special configuration.
- In MySQL InnoDB, `REPEATABLE READ` provides snapshot isolation for consistent reads.

**Constraints and Limitations:**

- Snapshot isolation permits write skew (a serialization anomaly).
- Requires additional storage for row versions (tempdb in SQL Server, undo logs in MySQL).
- Long-running transactions can cause version store growth.
- Not all databases support explicit `SNAPSHOT` isolation (PostgreSQL uses `REPEATABLE READ` instead).

### Annotated Complete Code Examples

**Example 1: SQL Server SNAPSHOT Isolation**

```sql
-- Enable snapshot isolation on the database
ALTER DATABASE mydb SET ALLOW_SNAPSHOT_ISOLATION ON;

-- Session 1: Begin SNAPSHOT transaction
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
BEGIN TRANSACTION;

SELECT balance FROM accounts WHERE account_id = 1;
-- Output: 1000.00

-- Session 2: Update and commit (not blocked by Session 1)
UPDATE accounts SET balance = 900 WHERE account_id = 1;
COMMIT;

-- Session 1: Read again
SELECT balance FROM accounts WHERE account_id = 1;
-- Output: 1000.00  -- Snapshot consistency preserved

COMMIT;
```

**Why this output occurs:** Session 1 sees a transaction-level snapshot from the moment the transaction began. Session 2's committed change is not visible because the snapshot was taken before the update.

**Example 2: Write Skew Under Snapshot Isolation**

```sql
-- Classic write skew example: two transactions check a constraint and update disjoint rows
-- Constraint: sum of balances must be >= 0

-- Snapshot isolation allows both transactions to commit, violating the constraint

-- Session 1: Read balances and update one account
BEGIN ISOLATION LEVEL REPEATABLE READ; -- PostgreSQL snapshot isolation
SELECT SUM(balance) FROM accounts; -- Output: 1000.00
UPDATE accounts SET balance = balance - 600 WHERE account_id = 1;
COMMIT;

-- Session 2: Read balances and update another account
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT SUM(balance) FROM accounts; -- Output: 1000.00 (sees the same snapshot)
UPDATE accounts SET balance = balance - 600 WHERE account_id = 2;
COMMIT;

-- Result: Both transactions commit, but the sum is now -200.00 (constraint violated)
```

**Why this happens:** Both transactions read the same snapshot (sum = 1000) and each believed the sum would remain non-negative after their update. Since they updated disjoint rows, there was no write-write conflict, and snapshot isolation allowed both to commit — a write skew anomaly.

### Real-World Cases

**Case 1: High-Concurrency Web Applications**

A web application uses SQL Server's RCSI to enable `READ COMMITTED` with snapshot consistency, allowing readers and writers to proceed without blocking each other.

**Case 2: Reporting and Analytics**

A BI system uses snapshot isolation to run long-running analytical queries without blocking operational transactions, while still seeing a consistent view of the data.

### References

- Microsoft Learn — Row Versioning and Snapshot Isolation - https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide
- PostgreSQL Documentation — Transaction Isolation (MVCC) - https://www.postgresql.org/docs/16/transaction-iso.html
- SAP Help — Connection Support in Multiversion Concurrency Control - https://help.sap.com/docs/SAP_ASE_SDK
- ACM Digital Library — Reasoning about Weak Isolation Levels in Separation Logic - https://dl.acm.org/

---

## 6. Anomalies vs. Levels Mapping

### Definitions

**Core Definition:** The anomalies-to-isolation-levels mapping shows which concurrency anomalies are physically blocked or permitted at each isolation level, providing a precise reference for choosing the appropriate level.

**Technical Definition:** The SQL standard defines isolation levels in terms of phenomena that must not occur at each level. The standard phenomena are: dirty read, non-repeatable read, phantom read, and serialization anomaly. Additional anomalies such as dirty write, lost update, read skew, and write skew are also relevant and are addressed by specific implementations.

**Beginner-Friendly Explanation:** This mapping is a quick-reference table that tells you exactly what can go wrong at each isolation level. If you know your application cannot tolerate a specific anomaly, you can look up which isolation level prevents it.

### Purposes

- To provide a precise reference for selecting the appropriate isolation level.
- To understand the trade-offs between consistency and concurrency.
- To diagnose concurrency-related bugs by identifying which anomaly is occurring.
- To document isolation requirements for compliance and auditing.

### Anomalies Defined

| Anomaly | Description | Code |
|---------|-------------|------|
| **Dirty Write** | A transaction overwrites uncommitted data from another transaction | A1 |
| **Dirty Read** | A transaction reads uncommitted data from another transaction that later rolls back | A2 |
| **Lost Update** | Two transactions update the same data; one update overwrites the other | A3 |
| **Fuzzy Read (Non-Repeatable Read)** | A transaction reads the same row twice and gets different values | A4 |
| **Phantom Read** | A transaction re-executes a query and sees a different set of rows | A5 |
| **Read Skew** | A transaction reads related data at different times and sees inconsistent states | A6 |
| **Write Skew** | Two transactions read overlapping data, make disjoint updates, and both commit | A7 |
| **Serialization Anomaly** | Any anomaly that violates serializability (a superset of the above) | — |

### Complete Anomaly-to-Isolation Mapping

| Anomaly | READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SNAPSHOT | SERIALIZABLE |
|---------|:----------------:|:--------------:|:---------------:|:--------:|:------------:|
| Dirty Write | Prevented | Prevented | Prevented | Prevented | Prevented |
| Dirty Read | **Allowed** | Prevented | Prevented | Prevented | Prevented |
| Lost Update | **Allowed** | **Allowed** | Prevented¹ | Prevented¹ | Prevented |
| Fuzzy Read (Non-Repeatable Read) | **Allowed** | **Allowed** | Prevented | Prevented | Prevented |
| Phantom Read | **Allowed** | **Allowed** | Possible² | Possible² | Prevented |
| Read Skew | **Allowed** | **Allowed** | Prevented | Prevented | Prevented |
| Write Skew | **Allowed** | **Allowed** | **Allowed** | **Allowed** | Prevented |
| Serialization Anomaly | **Allowed** | **Allowed** | **Allowed** | **Allowed** | Prevented |

**Notes:**

¹ Lost update is prevented in REPEATABLE READ and SNAPSHOT in most implementations (PostgreSQL, MySQL InnoDB, SQL Server with row versioning) because write-write conflicts are detected. However, the SQL standard's REPEATABLE READ does not explicitly prevent lost updates.

² Phantom reads are possible under the SQL standard's REPEATABLE READ, but PostgreSQL's implementation (which uses snapshot isolation) prevents them. MySQL InnoDB's REPEATABLE READ uses gap locks to prevent phantom reads in many cases.

### Annotated Complete Code Examples

**Example 1: Demonstrating the Mapping**

```sql
-- The following table summarises the anomalies permitted at each level
-- based on the SQL standard and major RDBMS implementations.

-- At READ UNCOMMITTED, all three standard anomalies are possible:
-- - Dirty Read: Allowed
-- - Non-Repeatable Read: Allowed
-- - Phantom Read: Allowed

-- At READ COMMITTED:
-- - Dirty Read: Prevented
-- - Non-Repeatable Read: Possible
-- - Phantom Read: Possible

-- At REPEATABLE READ (SQL standard):
-- - Dirty Read: Prevented
-- - Non-Repeatable Read: Prevented
-- - Phantom Read: Possible

-- At SERIALIZABLE:
-- - Dirty Read: Prevented
-- - Non-Repeatable Read: Prevented
-- - Phantom Read: Prevented
-- - Serialization Anomaly: Prevented
```

### Real-World Cases

**Case 1: Choosing the Right Level**

A financial application cannot tolerate non-repeatable reads or phantom reads. The mapping shows that `REPEATABLE READ` prevents non-repeatable reads but not phantom reads, while `SERIALIZABLE` prevents both. The application chooses `SERIALIZABLE` for critical operations.

**Case 2: Performance vs. Consistency**

A high-traffic social media application can tolerate non-repeatable reads and phantom reads but not dirty reads. The mapping shows that `READ COMMITTED` prevents dirty reads while allowing higher concurrency.

### References

- PostgreSQL Documentation — Transaction Isolation (Table 13.1) - https://www.postgresql.org/docs/16/transaction-iso.html
- TU Delft Repository — Phenomena Possible in Different Isolation Levels - https://repository.tudelft.nl/
- GitHub — Anomaly-to-Isolation Mapping Matrix - https://github.com/bookforge-ai/bookforge-skills
- ACM Digital Library — A Critique of ANSI SQL Isolation Levels - https://dlnext.acm.org/

---

## Summary Table: Isolation Levels at a Glance

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Serialization Anomaly | Default In |
|-------|:----------:|:-------------------:|:------------:|:---------------------:|------------|
| READ UNCOMMITTED | Allowed | Allowed | Allowed | Allowed | — |
| READ COMMITTED | Prevented | Allowed | Allowed | Allowed | PostgreSQL, SQL Server, Oracle |
| REPEATABLE READ | Prevented | Prevented | Possible¹ | Allowed | MySQL InnoDB |
| SNAPSHOT | Prevented | Prevented | Possible | Allowed | SQL Server (opt-in) |
| SERIALIZABLE | Prevented | Prevented | Prevented | Prevented | — |

¹ PostgreSQL's `REPEATABLE READ` also prevents phantom reads due to its MVCC implementation.

---

## Dialect Comparison: Isolation Level Support

| Feature | PostgreSQL | MySQL | SQL Server | Oracle |
|---------|-----------|-------|------------|--------|
| READ UNCOMMITTED | Behaves as READ COMMITTED | ✅ | ✅ | Non-standard extension |
| READ COMMITTED | ✅ (default) | ✅ | ✅ (default) | ✅ (default) |
| REPEATABLE READ | ✅ (prevents phantom reads) | ✅ (default) | ✅ | ❌ (not separate) |
| SNAPSHOT | ❌ (use REPEATABLE READ) | ❌ | ✅ (opt-in) | ❌ |
| SERIALIZABLE | ✅ | ✅ | ✅ | ✅ |
| RCSI (READ COMMITTED SNAPSHOT) | ❌ | ❌ | ✅ | ❌ |
| MVCC | ✅ | ✅ | ✅ (row versioning) | ✅ |

---

## References

- PostgreSQL Documentation — Transaction Isolation - https://www.postgresql.org/docs/16/transaction-iso.html
- MySQL 8.0 Reference Manual — Transaction Isolation Levels - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/innodb-transaction-isolation-levels.html
- MySQL 8.4 Reference Manual — Transaction Isolation Levels - https://dev.mysql.com.tw/doc/refman/8.4/en/innodb-transaction-isolation-levels.html
- Microsoft Learn — SQL Server Transaction Locking and Row Versioning Guide - https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide
- Microsoft Learn — Isolation Levels in the Database Engine - https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide
- Microsoft Learn — SET TRANSACTION ISOLATION LEVEL (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/set-transaction-isolation-level-transact-sql
- Oracle Database Concepts — Oracle Transaction Isolation Levels - https://docs.oracle.com/cd/F19136_01/cncpt.pdf
- Oracle Database — ANSI/ISO SQL Transaction Isolation Levels - https://docs.oracle.com/en/database/oracle/oracle-database/
- SAP Help — Connection Support in Multiversion Concurrency Control - https://help.sap.com/docs/SAP_ASE_SDK
- TU Delft Repository — Phenomena Possible in Different Isolation Levels - https://repository.tudelft.nl/
- ACM Digital Library — A Critique of ANSI SQL Isolation Levels - https://dlnext.acm.org/
- ACM Digital Library — Reasoning about Weak Isolation Levels in Separation Logic - https://dl.acm.org/
- GitHub — Anomaly-to-Isolation Mapping Matrix - https://github.com/bookforge-ai/bookforge-skills
- ISO/IEC 9075-2 — SQL Standard (Transaction Isolation Levels) - https://jtc1sc32.org/doc/N0251-0300/32N0254.pdf