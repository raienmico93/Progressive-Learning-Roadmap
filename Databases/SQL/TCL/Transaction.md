# Comprehensive Programming Cheat Sheet: Transaction Fundamentals

## Topic Overview

### Definitions

**Core Definition**
A transaction is a sequence of one or more SQL statements that the database treats as a single, indivisible unit of work: either all of its changes are applied permanently, or none of them are.

**Technical Definition**
A transaction is the atomic unit of recovery and concurrency control in a relational database management system (RDBMS). It groups statements into a single logical operation whose effects become visible to other sessions only at commit time, and whose changes are guaranteed to be durable once committed. Transactions are bounded by explicit control statements (`BEGIN`/`START TRANSACTION`, `COMMIT`, `ROLLBACK`, and savepoint variants) or, under autocommit, by the boundaries of a single statement. Transaction semantics are governed by the ACID properties (Atomicity, Consistency, Isolation, Durability), and isolation behavior is defined by the SQL standard's isolation levels (READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE). Nested transactions are not supported as first-class objects in most engines; savepoints provide partial rollback within a single transaction.

**Beginner-Friendly Explanation**
A transaction is like a bundle of changes that succeed or fail together. Think of transferring money between two bank accounts: the money must leave one account and arrive in the other as one all-or-nothing operation. If the debit succeeds but the credit fails, the transaction rolls back and the debit is undone. Transactions are how the database guarantees that your data stays consistent even when errors, crashes, or concurrent users get in the way.

### Key Characteristics

- **All-or-nothing semantics**: A transaction's changes are either all committed or all rolled back; there is no partial application at the transaction boundary.
- **Scope-limited**: Transactions begin and end at explicit boundaries (`BEGIN`, `COMMIT`, `ROLLBACK`) or, under autocommit, at statement boundaries.
- **Concurrency-aware**: Isolation levels control how concurrent transactions see each other's changes, trading consistency for concurrency.
- **Durable once committed**: A committed transaction survives crashes, power loss, and process termination (subject to the engine's `fsync`/WAL configuration).
- **Savepoints enable partial rollback**: Within a transaction, `SAVEPOINT` marks intermediate points that can be rolled back to without aborting the entire transaction.
- **Autocommit is the default in most engines**: Each statement runs in its own implicit transaction unless explicitly grouped.
- **Nested transactions are emulated, not native**: Savepoints provide nested-like behavior; true nested transactions (with independent commit/rollback) are not supported in mainstream RDBMSs.

### Prerequisites

- Understanding of SQL `SELECT`, `INSERT`, `UPDATE`, `DELETE`, and `WHERE` clauses.
- Familiarity with table constraints (`PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `CHECK`).
- Awareness of concurrent sessions and the concept of isolation.
- Access to a database client that supports transaction control statements.

### Related Programming Areas

- Concurrency control and isolation-level design.
- Database recovery and write-ahead logging (WAL).
- Application transaction management (connection pools, ORMs, frameworks).
- Distributed transactions and two-phase commit.
- Financial systems, inventory management, and other consistency-critical domains.

---

## Core Concepts and Features

### 1. What Transactions Are

#### Definitions

**Core Definition**
A transaction is a logical unit of database work that groups multiple statements so they take effect together or not at all.

**Technical Definition**
A transaction is an execution context in which a sequence of statements operates on a database snapshot, acquires locks, and buffers changes. The transaction ends with either a `COMMIT` (making changes permanent and visible) or a `ROLLBACK` (discarding changes and releasing locks). The RDBMS maintains transaction state in memory and in the transaction log (WAL in PostgreSQL, redo log in MySQL, transaction log in SQL Server, redo in Oracle) to guarantee atomicity and durability. Transactions are the fundamental mechanism for enforcing consistency under concurrency.

**Beginner-Friendly Explanation**
A transaction is a wrapper around a group of changes. You open the wrapper with `BEGIN`, make your changes, and then either seal the wrapper with `COMMIT` (saving everything) or throw it away with `ROLLBACK` (saving nothing).

#### Purposes

- To group related data modifications into an all-or-nothing operation.
- To ensure that intermediate inconsistent states are never visible to other sessions.
- To provide a rollback path when an error occurs mid-way through a multi-step operation.
- To define the boundary at which changes become durable and visible.

#### Syntax Rules and Structure

**General lifecycle**:

```sql
BEGIN;          -- or START TRANSACTION
-- one or more DML statements
COMMIT;         -- or ROLLBACK
```

**Component breakdown**:

| Phase | Description |
|-------|-------------|
| Begin | Opens a transaction; subsequent statements are part of it. |
| Body | DML statements execute within the transaction's snapshot and lock context. |
| Commit / Rollback | Ends the transaction, either applying or discarding changes. |

**Syntax rules**:

- Only one transaction can be active per session at a time (savepoints create nested *scopes*, not nested transactions).
- DML statements inside a transaction are not visible to other sessions until `COMMIT`.
- DDL behavior varies: PostgreSQL supports transactional DDL; MySQL causes an implicit commit for most DDL; SQL Server supports transactional DDL; Oracle issues implicit commits around DDL.

**Constraints and limitations**

- Long-running transactions hold locks and prevent vacuum/cleanup, causing bloat and contention.
- Some statements cannot run inside a transaction block (e.g., `CREATE INDEX CONCURRENTLY` in PostgreSQL, `VACUUM`).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – A simple transaction

```sql
-- Step 1: Open a transaction.
BEGIN;

-- Step 2: Perform multiple related changes.
INSERT INTO accounts (id, balance) VALUES (1, 1000) ON CONFLICT (id) DO NOTHING;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Step 3: Commit the transaction.
COMMIT;
```

**Expected output**:

```text
BEGIN
INSERT 0 0
UPDATE 1
UPDATE 1
COMMIT
```

**Why this output appears**: `BEGIN` opens the transaction. Each DML statement reports the number of rows affected. `COMMIT` makes all changes permanent and visible to other sessions. If any statement had failed, the entire transaction could have been rolled back, undoing the earlier changes.

##### Example 2: MySQL – Transaction with rollback on error

```sql
START TRANSACTION;

INSERT INTO inventory (product_id, qty) VALUES (10, 100);
UPDATE inventory SET qty = qty - 5 WHERE product_id = 10;
-- Simulate an error: insufficient stock.
UPDATE inventory SET qty = qty - 1000 WHERE product_id = 10 AND qty >= 1000;

-- The second UPDATE affected 0 rows; roll back the whole transaction.
ROLLBACK;
```

**Expected output**:

```text
Query OK, 0 rows affected
Query OK, 1 row affected
Query OK, 0 rows affected
Query OK, 0 rows affected
```

**Why this output appears**: The rollback discards the insert and the decrement, leaving `inventory` unchanged. The application can then retry or report an error without leaving the database in a partial state.

#### Real-World Cases

- **Bank transfers**: Debit and credit must succeed together.
- **Order placement**: Reserve inventory, create order, charge payment—all in one transaction.
- **Batch imports**: A failed import should not leave half the rows loaded.

---

### 2. Atomic Unit of Work

#### Definitions

**Core Definition**
The atomic unit of work is the all-or-nothing boundary of a transaction: every statement inside the transaction succeeds and commits together, or the entire unit is rolled back.

**Technical Definition**
Atomicity is implemented through the transaction log and undo/rollback mechanisms. Before modifying data pages, the RDBMS writes log records (undo and redo) describing the change. On commit, the log is flushed to durable storage; on rollback or crash recovery, the log is used to undo uncommitted changes and redo committed ones. The atomic unit ensures that no partial state is ever visible, even if the system crashes mid-transaction.

**Beginner-Friendly Explanation**
Atomic means "cannot be split." A transaction is one atomic unit of work—like a single keystroke that either registers completely or not at all.

#### Purposes

- To guarantee that multi-statement operations never leave partial results.
- To enable safe recovery after crashes or errors.
- To provide a rollback path that restores the pre-transaction state.
- To define the smallest unit that is visible to other sessions.

#### Syntax Rules and Structure

**Atomic boundary pattern**:

```sql
BEGIN;
  statement_1;
  statement_2;
  statement_3;
COMMIT;   -- all three succeed together
-- or
ROLLBACK; -- all three are discarded together
```

**Component breakdown**:

| Element | Role |
|---------|------|
| `BEGIN` | Marks the start of the atomic unit. |
| Body statements | Execute and buffer changes. |
| `COMMIT` / `ROLLBACK` | Applies or discards the entire unit. |

**Syntax rules**

- If any statement in the unit fails, the application should issue `ROLLBACK` (or let the engine abort the transaction, as PostgreSQL does).
- PostgreSQL: an error inside a transaction aborts the transaction; subsequent statements are rejected until `ROLLBACK` or `ROLLBACK TO SAVEPOINT`.
- MySQL: an error does not automatically roll back the transaction; the application must decide.
- SQL Server: `SET XACT_ABORT ON` causes automatic rollback on runtime errors.

**Constraints and limitations**

- Atomicity covers data changes, not external side effects (email sent, file written).
- Some engines auto-commit DDL, breaking atomicity across schema and data changes.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Atomic multi-table insert

```sql
BEGIN;

INSERT INTO orders (order_id, customer_id, total) VALUES (1001, 42, 250.00);
INSERT INTO order_items (order_id, product_id, qty) VALUES (1001, 7, 2);
INSERT INTO order_items (order_id, product_id, qty) VALUES (1001, 9, 1);

-- If any insert fails, the whole transaction rolls back.
COMMIT;
```

**Expected output**:

```text
BEGIN
INSERT 0 1
INSERT 0 1
INSERT 0 1
COMMIT
```

**Why this output appears**: All three inserts commit together. If the second `INSERT` violated a foreign key, PostgreSQL would abort the transaction and the first insert would be undone.

#### Real-World Cases

- **Multi-table writes**: Order header plus order lines.
- **Data migration**: Copying rows across tables in one atomic step.
- **Payment processing**: Ledger entries that must balance.

---

### 3. ACID Properties

#### 3.1 Atomicity

##### Definitions

**Core Definition**
Atomicity guarantees that a transaction is treated as a single indivisible unit: either all its changes are applied, or none are.

**Technical Definition**
Atomicity is enforced by the transaction log's undo mechanism. Uncommitted changes are never made visible to other transactions; if a transaction aborts, the RDBMS uses undo records to reverse its changes. On crash recovery, uncommitted transactions are rolled back using the log. In PostgreSQL, MVCC (Multi-Version Concurrency Control) keeps uncommitted row versions invisible; on abort, they are marked dead and later vacuumed. In MySQL InnoDB, undo logs and the rollback segment reverse uncommitted changes.

**Beginner-Friendly Explanation**
Atomicity means "all or nothing." A transaction that inserts five rows and then fails on the sixth leaves zero rows, not five.

##### Purposes

- To prevent partial application of multi-step operations.
- To guarantee that aborted transactions leave no trace.
- To support crash recovery by cleanly identifying committed vs. uncommitted work.
- To provide a reliable rollback path for application error handling.

##### Syntax Rules and Structure

**Atomicity in action**:

```sql
BEGIN;
INSERT INTO t VALUES (1);
INSERT INTO t VALUES (1);  -- violates primary key; transaction aborts
ROLLBACK;                  -- no rows persisted
```

**Component breakdown**:

| Mechanism | Engine | Role |
|-----------|--------|------|
| Undo log | MySQL InnoDB | Reverses uncommitted changes |
| MVCC dead tuples | PostgreSQL | Hides and discards uncommitted versions |
| Transaction log | SQL Server, Oracle | Supports rollback and recovery |

**Syntax rules**

- An explicit `ROLLBACK` is required in most engines after an error (except PostgreSQL, which aborts automatically).
- Savepoints allow partial rollback while keeping the transaction open.

**Constraints and limitations**

- External side effects (emails, files) are not rolled back automatically.
- Some DDL statements implicitly commit and break atomicity across schema changes.

##### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: PostgreSQL – Atomicity enforced on primary-key violation

```sql
CREATE TABLE t (id INT PRIMARY KEY);

BEGIN;
INSERT INTO t VALUES (1);
INSERT INTO t VALUES (1);  -- duplicate key error; transaction aborts
-- Subsequent statements are rejected until rollback.
ROLLBACK;

SELECT COUNT(*) FROM t;  -- returns 0
```

**Expected output**:

```text
CREATE TABLE
BEGIN
INSERT 0 1
ERROR:  duplicate key value violates unique constraint "t_pkey"
ROLLBACK
 count
-------
     0
(1 row)
```

**Why this output appears**: The duplicate-key error aborts the transaction. The rollback discards the first insert, and `COUNT(*)` confirms the table is empty.

##### Real-World Cases

- **Inventory reservation**: All or nothing to avoid overselling.
- **Financial ledgers**: Debits and credits must balance.

---

#### 3.2 Consistency

##### Definitions

**Core Definition**
Consistency guarantees that a transaction moves the database from one valid state to another, preserving all declared constraints and business rules.

**Technical Definition**
Consistency is enforced through declarative constraints (`PRIMARY KEY`, `UNIQUE`, `FOREIGN KEY`, `CHECK`, `NOT NULL`), triggers, and application logic. The database ensures that no transaction commits a state that violates a constraint; if a constraint is violated, the statement (or transaction) fails. Unlike atomicity, isolation, and durability—which are database-enforced—consistency also depends on the application writing correct logic (e.g., debits equaling credits).

**Beginner-Friendly Explanation**
Consistency means the database rules are never broken. If every order must have a customer, the database will reject an order without one.

##### Purposes

- To prevent invalid data from entering the database.
- To enforce business rules declaratively rather than relying solely on application code.
- To preserve invariants across multi-step transactions.
- To ensure that committed state satisfies all constraints.

##### Syntax Rules and Structure

**Constraint enforcement**:

```sql
BEGIN;
INSERT INTO order_items (order_id, product_id, qty) VALUES (1001, 7, 2);
-- If order_id 1001 does not exist in orders, foreign key fails.
COMMIT;
```

**Component breakdown**:

| Constraint | Effect |
|------------|--------|
| `PRIMARY KEY` | Uniqueness and non-null |
| `UNIQUE` | Uniqueness |
| `FOREIGN KEY` | Referential integrity |
| `CHECK` | Domain and business rules |
| `NOT NULL` | Required values |

**Syntax rules**

- Constraints can be deferred (e.g., `DEFERRABLE INITIALLY DEFERRED` in PostgreSQL) so they are checked at commit rather than at statement time.
- Triggers can enforce complex consistency rules that constraints cannot express.

**Constraints and limitations**

- Consistency of application-level invariants (e.g., "sum of debits equals sum of credits") is not automatically enforced by constraints in most engines.
- Deferred constraints can allow temporary inconsistency within a transaction, checked at commit.

##### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: PostgreSQL – Foreign-key consistency

```sql
BEGIN;
INSERT INTO order_items (order_id, product_id, qty) VALUES (9999, 7, 2);
-- order_id 9999 does not exist in orders; foreign key fails.
ROLLBACK;
```

**Expected output**:

```text
BEGIN
ERROR:  insert or update on table "order_items" violates foreign key constraint
DETAIL:  Key (order_id)=(9999) is not present in table "orders".
ROLLBACK
```

**Why this output appears**: The foreign-key constraint prevents the insert, ensuring referential integrity. Rolling back leaves the database in a consistent state.

##### Real-World Cases

- **Referential integrity**: Orders cannot reference non-existent customers.
- **Check constraints**: Balances cannot go negative if a `CHECK (balance >= 0)` exists.
- **Deferred constraints**: Cyclic foreign keys loaded within one transaction.

---

#### 3.3 Isolation

##### Definitions

**Core Definition**
Isolation controls how concurrent transactions see each other's uncommitted and committed changes, preventing interference that would produce incorrect results.

**Technical Definition**
The SQL standard defines four isolation levels: **READ UNCOMMITTED** (dirty reads possible), **READ COMMITTED** (no dirty reads; non-repeatable reads possible), **REPEATABLE READ** (no non-repeatable reads; phantom reads possible in theory), and **SERIALIZABLE** (transactions appear to execute serially). Engines implement these levels with locking, MVCC, or serializable snapshot isolation (SSI). PostgreSQL implements READ UNCOMMITTED as READ COMMITTED and uses SSI for SERIALIZABLE. MySQL InnoDB implements MVCC with REPEATABLE READ as default and uses next-key locking for SERIALIZABLE. SQL Server uses locking with READ COMMITTED as default and supports snapshot isolation as an opt-in.

**Beginner-Friendly Explanation**
Isolation is about not tripping over other transactions. If two people are booking the last seat at the same time, isolation rules decide who wins and how each sees the other's booking.

##### Purposes

- To prevent dirty reads, non-repeatable reads, and phantom reads.
- To balance consistency guarantees against concurrency and throughput.
- To allow applications to choose the appropriate trade-off per workload.
- To enable serializable execution where correctness demands it.

##### Syntax Rules and Structure

**Setting isolation level**:

```sql
-- PostgreSQL / standard
BEGIN ISOLATION LEVEL SERIALIZABLE;
-- or
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- MySQL
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;

-- SQL Server
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
BEGIN TRANSACTION;
```

**Isolation-level matrix**:

| Level | Dirty reads | Non-repeatable reads | Phantoms |
|-------|-------------|----------------------|----------|
| READ UNCOMMITTED | Possible | Possible | Possible |
| READ COMMITTED | Prevented | Possible | Possible |
| REPEATABLE READ | Prevented | Prevented | Possible (standard); prevented in some engines |
| SERIALIZABLE | Prevented | Prevented | Prevented |

**Syntax rules**

- Isolation level must be set before the transaction begins (or as part of `BEGIN`).
- Higher isolation reduces concurrency and may cause serialization failures requiring retry.
- Some engines (e.g., PostgreSQL) implement SERIALIZABLE via SSI, which may abort transactions with serialization errors (`40001`).

**Constraints and limitations**

- Isolation cannot be changed mid-transaction in most engines.
- SERIALIZABLE can cause frequent retries in high-contention workloads.
- READ UNCOMMITTED is not truly implemented in some engines (PostgreSQL maps it to READ COMMITTED).

##### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: PostgreSQL – Non-repeatable read at READ COMMITTED

```sql
-- Session A
BEGIN ISOLATION LEVEL READ COMMITTED;
SELECT balance FROM accounts WHERE id = 1;  -- returns 1000

-- Session B commits a change
-- Session A re-reads:
SELECT balance FROM accounts WHERE id = 1;  -- returns 900 (non-repeatable read)
COMMIT;
```

**Expected output** (Session A):

```text
BEGIN
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

**Why this output appears**: At READ COMMITTED, each statement sees the latest committed snapshot, so the second read observes Session B's committed update. At REPEATABLE READ, the second read would still return 1000.

##### Real-World Cases

- **Booking systems**: SERIALIZABLE or `SELECT ... FOR UPDATE` to prevent double-booking.
- **Reporting**: REPEATABLE READ for stable multi-query snapshots.
- **High-throughput OLTP**: READ COMMITTED as the pragmatic default.

---

#### 3.4 Durability

##### Definitions

**Core Definition**
Durability guarantees that once a transaction commits, its changes survive crashes, power loss, and restarts.

**Technical Definition**
Durability is implemented by writing the transaction log to non-volatile storage before the commit is acknowledged. PostgreSQL uses WAL with `fsync`; MySQL InnoDB uses the redo log with `innodb_flush_log_at_trx_commit`; SQL Server uses the transaction log with write-ahead logging; Oracle uses redo logs and `COMMIT WRITE WAIT`. Durability can be relaxed for performance (e.g., `synchronous_commit = off` in PostgreSQL) at the cost of possibly losing recent commits on crash.

**Beginner-Friendly Explanation**
Durability means that once the database says "committed," the data is safe, even if the server loses power the next second.

##### Purposes

- To guarantee that committed data survives hardware failure and crashes.
- To support recovery after unexpected shutdowns.
- To provide a firm boundary between "saved" and "not saved" for application logic.
- To meet regulatory and business requirements for data persistence.

##### Syntax Rules and Structure

**Durability controls**:

```sql
-- PostgreSQL: relax durability for performance
SET synchronous_commit = off;

-- MySQL: control log flush at commit
SET GLOBAL innodb_flush_log_at_trx_commit = 1;  -- full durability
```

**Component breakdown**:

| Parameter | Effect |
|-----------|--------|
| `synchronous_commit` (PG) | `on` (default) = flush WAL before commit ack |
| `innodb_flush_log_at_trx_commit` (MySQL) | `1` = flush to disk; `2` = to OS cache; `0` = once per second |
| `COMMIT WRITE WAIT` (Oracle) | Wait for redo write before ack |

**Syntax rules**

- Full durability requires flushing the log to stable storage before acknowledging commit.
- Relaxing durability improves throughput but risks losing recent commits on crash.
- Replicas may acknowledge commits asynchronously depending on `synchronous_standby_names` (PG) or `synchronous_commit` settings.

**Constraints and limitations**

- Durability depends on hardware honoring flush semantics (battery-backed write cache, SSD flush behavior).
- Asynchronous replication can lose committed data on failover.

##### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: PostgreSQL – Durability with `synchronous_commit`

```sql
-- Default: durable
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;  -- waits for WAL flush

-- Relaxed: faster commit, possible loss on crash
SET synchronous_commit = off;
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;  -- may return before WAL flush
```

**Expected output**: Both commits return `COMMIT`, but only the first is guaranteed durable at the moment of acknowledgment.

**Why this output appears**: With `synchronous_commit = off`, the commit does not wait for WAL to reach disk, trading a small durability window for lower latency.

##### Real-World Cases

- **Financial systems**: Full durability is mandatory.
- **Analytics ingestion**: Relaxed durability acceptable for throughput.
- **Async replication failover**: Understanding potential commit loss.

---

### 4. Transaction Boundaries

#### Definitions

**Core Definition**
Transaction boundaries are the points at which a transaction begins and ends, defining which statements are grouped into a single atomic unit.

**Technical Definition**
Boundaries are established by `BEGIN`/`START TRANSACTION`, `COMMIT`, `ROLLBACK`, and—implicitly—by autocommit behavior, DDL statements that force commits, and session termination. Correct boundary placement is critical: too-short boundaries lose atomicity; too-long boundaries hold locks and bloat, reducing concurrency and causing contention.

**Beginner-Friendly Explanation**
Transaction boundaries are where you open and close the transaction. Everything between `BEGIN` and `COMMIT` is inside; everything after is outside.

#### Purposes

- To define the exact set of statements that commit or roll back together.
- To minimize lock duration by keeping transactions short.
- To ensure consistency by including all related changes in one unit.
- To align application logic with database atomicity.

#### Syntax Rules and Structure

**Explicit boundaries**:

```sql
BEGIN;
-- statements
COMMIT;
```

**Autocommit boundaries** (each statement is its own transaction):

```sql
INSERT INTO t VALUES (1);  -- committed immediately
INSERT INTO t VALUES (2);  -- committed immediately
```

**Component breakdown**:

| Boundary | Effect |
|----------|--------|
| `BEGIN` | Starts a transaction |
| `COMMIT` | Ends and applies |
| `ROLLBACK` | Ends and discards |
| Statement end (autocommit) | Implicit commit per statement |
| Session disconnect | Implicit rollback of open transaction |

**Syntax rules**

- Only one transaction per session; nesting requires savepoints.
- DDL may force an implicit commit (MySQL, Oracle).
- Session termination rolls back any open transaction.

**Constraints and limitations**

- Application frameworks may open transactions automatically; know the framework's boundaries.
- Connection pools may hold transactions open if not properly committed.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Boundary placement affecting atomicity

```sql
-- Anti-pattern: separate statements in autocommit mode
INSERT INTO orders (id, customer_id) VALUES (1001, 42);
INSERT INTO order_items (order_id, product_id) VALUES (1001, 7);
-- If the second insert fails, the first remains committed.

-- Correct: explicit transaction
START TRANSACTION;
INSERT INTO orders (id, customer_id) VALUES (1001, 42);
INSERT INTO order_items (order_id, product_id) VALUES (1001, 7);
COMMIT;
```

**Expected output**: In the autocommit case, the first insert persists even if the second fails. In the explicit case, both persist or neither does.

**Why this output appears**: Autocommit commits each statement independently, so the boundary is per-statement. Explicit `START TRANSACTION` groups both statements into one atomic unit.

#### Real-World Cases

- **Order creation**: Header and lines must share a boundary.
- **API endpoints**: One transaction per request, committed before responding.
- **Batch jobs**: Larger transactions reduce commit overhead but hold locks longer.

---

### 5. `BEGIN`

#### Definitions

**Core Definition**
`BEGIN` (or `START TRANSACTION`) is the statement that opens a transaction, marking the start of a group of statements that will commit or roll back together.

**Technical Definition**
`BEGIN` transitions the session from autocommit mode into an explicit transaction block. In PostgreSQL, `BEGIN` starts a transaction with the default or specified isolation level; `START TRANSACTION` is synonymous. In MySQL, `START TRANSACTION` (and `BEGIN` as an alias) starts an explicit transaction. SQL Server uses `BEGIN TRANSACTION` (with optional name). Oracle has no `BEGIN` statement for transactions; transactions start implicitly with the first DML statement and end with `COMMIT` or `ROLLBACK`.

**Beginner-Friendly Explanation**
`BEGIN` is the "start here" marker. Everything after it is part of the transaction until you say `COMMIT` or `ROLLBACK`.

#### Purposes

- To explicitly start a transaction with a known isolation level.
- To group subsequent statements into a single atomic unit.
- To disable autocommit behavior for the duration of the transaction.
- To give the application an explicit boundary for error handling.

#### Syntax Rules and Structure

**PostgreSQL**:

```sql
BEGIN [ WORK | TRANSACTION ] [ transaction_mode [, ...] ]
-- or
START TRANSACTION [ transaction_mode [, ...] ]

-- transaction_mode: ISOLATION LEVEL ... | READ WRITE | READ ONLY | [ NOT ] DEFERRABLE
```

**MySQL**:

```sql
START TRANSACTION [ transaction_characteristic [, ...] ]
-- or
BEGIN [ WORK ]

-- transaction_characteristic: WITH CONSISTENT SNAPSHOT | READ WRITE | READ ONLY
```

**SQL Server**:

```sql
BEGIN { TRAN | TRANSACTION } [ transaction_name ]
```

**Component breakdown**:

| Component | Description |
|-----------|-------------|
| `BEGIN` / `START TRANSACTION` | Opens the transaction. |
| `transaction_mode` | Isolation level and read/write characteristics. |
| `transaction_name` (SQL Server) | Optional name for nested-tracking. |

**Syntax rules**

- `BEGIN` is a no-op if a transaction is already open (PostgreSQL warns).
- Isolation level can be set in `BEGIN` or via `SET TRANSACTION` before any DML.
- SQL Server requires `BEGIN TRANSACTION` before `SAVE TRANSACTION`.

**Constraints and limitations**

- PostgreSQL `BEGIN` inside an existing transaction issues a warning and does not nest.
- Oracle does not use `BEGIN` for transactions.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – `BEGIN` with isolation level

```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT balance FROM accounts WHERE id = 1;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;
```

**Expected output**:

```text
BEGIN
 balance
---------
    1000
(1 row)
UPDATE 1
COMMIT
```

**Why this output appears**: `BEGIN ISOLATION LEVEL REPEATABLE READ` opens a transaction where all reads see the same snapshot. The update commits at `COMMIT`.

#### Real-World Cases

- **Explicit transaction control**: Wrapping multi-step operations.
- **Isolation tuning**: Per-transaction isolation for reporting vs. OLTP.
- **Stored procedures**: Explicit boundaries for error handling.

---

### 6. `COMMIT`

#### Definitions

**Core Definition**
`COMMIT` ends a transaction and makes all its changes permanent and visible to other sessions.

**Technical Definition**
`COMMIT` writes the transaction's log records to durable storage (per the durability configuration), marks the transaction as committed in the transaction log, releases locks, and makes the changes visible to subsequent snapshots. After `COMMIT`, the transaction cannot be rolled back. In PostgreSQL, `COMMIT` is a no-op outside a transaction (with a warning); in MySQL, `COMMIT` outside a transaction also has no effect.

**Beginner-Friendly Explanation**
`COMMIT` is the "save" button. It seals the transaction and makes all the changes real.

#### Purposes

- To finalize a transaction and make changes durable.
- To release locks held by the transaction.
- To make changes visible to other sessions.
- To mark a clean boundary for subsequent operations.

#### Syntax Rules and Structure

**Standard**:

```sql
COMMIT [ WORK | TRANSACTION ] [ AND [ NO ] CHAIN ]
```

**Component breakdown**:

| Component | Description |
|-----------|-------------|
| `COMMIT` | Ends and applies the transaction. |
| `AND CHAIN` (PG) | Immediately starts a new transaction with the same characteristics. |
| `AND NO CHAIN` | Default; no new transaction starts. |

**Syntax rules**

- `COMMIT` outside a transaction is a no-op or warning in most engines.
- After `COMMIT`, all locks acquired by the transaction are released.
- If durability is relaxed, commit may return before disk flush completes.

**Constraints and limitations**

- Cannot be undone once acknowledged.
- External side effects are not coordinated automatically.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – `COMMIT` releasing locks

```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;

-- Another session can now see the new balance.
```

**Expected output**:

```text
Query OK, 0 rows affected
Query OK, 1 row affected
Query OK, 0 rows affected
```

**Why this output appears**: The update is applied and locks are released at `COMMIT`, allowing other sessions to see the change.

#### Real-World Cases

- **API writes**: Commit before returning success to the client.
- **ETL batches**: Commit per batch to limit lock duration.
- **Long-running reports**: Avoid long transactions by committing intermediate work (where atomicity is not required).

---

### 7. `ROLLBACK`

#### Definitions

**Core Definition**
`ROLLBACK` ends a transaction and discards all its changes, restoring the database to its pre-transaction state.

**Technical Definition**
`ROLLBACK` uses the undo log (or MVCC dead-tuple marking) to reverse all changes made since `BEGIN` or since the last savepoint. It releases all locks acquired by the transaction. In PostgreSQL, after an error the transaction is in an aborted state; `ROLLBACK` is required to clear it. In MySQL, `ROLLBACK` rolls back the transaction but leaves the session usable.

**Beginner-Friendly Explanation**
`ROLLBACK` is the "undo" button. It throws away everything the transaction did.

#### Purposes

- To undo a failed or partially completed transaction.
- To clear an aborted transaction state (PostgreSQL).
- To release locks after deciding not to commit.
- To implement retry logic after serialization failures.

#### Syntax Rules and Structure

**Standard**:

```sql
ROLLBACK [ WORK | TRANSACTION ] [ AND [ NO ] CHAIN ]
-- or
ROLLBACK [ WORK ] TO [ SAVEPOINT ] savepoint_name
```

**Component breakdown**:

| Component | Description |
|-----------|-------------|
| `ROLLBACK` | Discards the entire transaction. |
| `ROLLBACK TO SAVEPOINT name` | Discards changes after the savepoint, keeping earlier changes. |

**Syntax rules**

- `ROLLBACK` outside a transaction is a no-op or warning.
- After `ROLLBACK`, the session returns to autocommit mode (unless `AND CHAIN`).
- PostgreSQL requires `ROLLBACK` after an error to clear the aborted state.

**Constraints and limitations**

- Cannot undo committed changes.
- Some DDL statements may have already committed implicitly.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – `ROLLBACK` after error

```sql
BEGIN;
INSERT INTO t VALUES (1);
INSERT INTO t VALUES (1);  -- error
ROLLBACK;

SELECT COUNT(*) FROM t;
```

**Expected output**:

```text
BEGIN
INSERT 0 1
ERROR:  duplicate key value violates unique constraint
ROLLBACK
 count
-------
     0
(1 row)
```

**Why this output appears**: The rollback discards the first insert, leaving the table empty. `ROLLBACK` also clears the aborted transaction state so the session can continue.

#### Real-World Cases

- **Error handling**: Application rolls back on exception.
- **Validation failures**: Discard changes when business rules are not met.
- **Serialization retries**: Roll back and retry on `40001` errors.

---

### 8. `SAVEPOINT`

#### Definitions

**Core Definition**
`SAVEPOINT` marks an intermediate point within a transaction that can be rolled back to without aborting the entire transaction.

**Technical Definition**
A savepoint creates a named marker in the transaction's change stream. Subsequent changes can be rolled back to the savepoint using `ROLLBACK TO SAVEPOINT`, discarding only the work done after the marker. Savepoints are the standard mechanism for partial rollback and emulate nested transactions within a single transaction. In PostgreSQL, after an error, the transaction is aborted, but `ROLLBACK TO SAVEPOINT` restores it to a usable state.

**Beginner-Friendly Explanation**
A savepoint is a checkpoint inside a transaction. If something goes wrong later, you can rewind to the checkpoint instead of undoing the whole transaction.

#### Purposes

- To allow partial rollback of a transaction without abandoning earlier work.
- To emulate nested transactions.
- To recover from errors in a sub-operation while keeping the outer transaction alive.
- To implement retry logic for individual steps within a larger transaction.

#### Syntax Rules and Structure

**Standard**:

```sql
SAVEPOINT savepoint_name;
```

**Component breakdown**:

| Component | Description |
|-----------|-------------|
| `SAVEPOINT` | Keyword. |
| `savepoint_name` | Identifier for the savepoint. |

**Syntax rules**

- Savepoints are scoped to the transaction; they are released at `COMMIT`/`ROLLBACK`.
- Multiple savepoints can be active; later savepoints with the same name shadow earlier ones.
- SQL Server uses `SAVE TRANSACTION name` (no `SAVEPOINT` keyword).
- Oracle uses `SAVEPOINT name`.

**Constraints and limitations**

- Savepoints add overhead; too many can impact performance.
- SQL Server's `SAVE TRANSACTION` does not support `RELEASE` and rolls back to the savepoint but does not decrement `@@TRANCOUNT`.
- Savepoints do not protect against errors that abort the entire transaction in some engines (e.g., certain deadlock situations).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Savepoint for partial rollback

```sql
BEGIN;
INSERT INTO orders (order_id, customer_id) VALUES (1001, 42);
SAVEPOINT after_order;

INSERT INTO order_items (order_id, product_id) VALUES (1001, 7);
-- Something goes wrong; roll back only the items.
ROLLBACK TO SAVEPOINT after_order;

-- The order still exists; add items again correctly.
INSERT INTO order_items (order_id, product_id) VALUES (1001, 8);
COMMIT;
```

**Expected output**:

```text
BEGIN
INSERT 0 1
SAVEPOINT
INSERT 0 1
ROLLBACK
INSERT 0 1
COMMIT
```

**Why this output appears**: `ROLLBACK TO SAVEPOINT after_order` discards the failed item insert but keeps the order insert. The transaction continues and commits with the corrected item.

#### Real-World Cases

- **Batch processing**: Savepoint per row; roll back only failing rows.
- **Multi-step workflows**: Savepoint after each milestone.
- **Retry logic**: Savepoint before a risky operation, roll back and retry on failure.

---

### 9. `RELEASE SAVEPOINT`

#### Definitions

**Core Definition**
`RELEASE SAVEPOINT` removes a savepoint, destroying the marker and making its position unavailable for future rollback.

**Technical Definition**
`RELEASE SAVEPOINT` removes the named savepoint and all savepoints established after it (in PostgreSQL, all savepoints created after the released one are also released). The transaction's changes up to that point remain part of the transaction and can no longer be rolled back via that savepoint. `RELEASE SAVEPOINT` does not commit the transaction; it only removes the rollback marker. SQL Server does not have a `RELEASE` equivalent; savepoints are implicitly removed at transaction end.

**Beginner-Friendly Explanation**
`RELEASE SAVEPOINT` removes a checkpoint you no longer need. The work up to that point becomes part of the transaction and cannot be rolled back to that checkpoint anymore.

#### Purposes

- To free savepoint resources once they are no longer needed.
- To prevent accidental rollback to an obsolete savepoint.
- To simplify transaction state before `COMMIT`.
- To manage nested savepoint stacks.

#### Syntax Rules and Structure

**PostgreSQL / MySQL / Oracle**:

```sql
RELEASE SAVEPOINT savepoint_name;
-- or in PostgreSQL: RELEASE savepoint_name;
```

**Component breakdown**:

| Component | Description |
|-----------|-------------|
| `RELEASE SAVEPOINT` | Removes the savepoint. |
| `savepoint_name` | The savepoint to release. |

**Syntax rules**

- The savepoint must exist; releasing a non-existent savepoint raises an error.
- Releasing a savepoint also releases all savepoints established after it (PostgreSQL).
- `RELEASE` does not commit the transaction.

**Constraints and limitations**

- SQL Server has no `RELEASE SAVEPOINT`; savepoints are cleared at transaction end.
- Releasing a savepoint does not undo or commit any changes; it only removes the rollback marker.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Releasing a savepoint

```sql
BEGIN;
SAVEPOINT sp1;
INSERT INTO t VALUES (1);
RELEASE SAVEPOINT sp1;

-- ROLLBACK TO SAVEPOINT sp1 is no longer possible.
COMMIT;
```

**Expected output**:

```text
BEGIN
SAVEPOINT
INSERT 0 1
RELEASE
COMMIT
```

**Why this output appears**: Releasing `sp1` removes the savepoint. The insert remains part of the transaction and commits. Attempting `ROLLBACK TO SAVEPOINT sp1` afterwards would raise an error.

#### Real-World Cases

- **Long transactions with milestones**: Release savepoints after confirming a step succeeded.
- **Savepoint stack management**: Avoid accumulating unnecessary savepoints.

---

### 10. `ROLLBACK TO SAVEPOINT`

#### Definitions

**Core Definition**
`ROLLBACK TO SAVEPOINT` undoes all changes made after the named savepoint while keeping the transaction open and preserving changes made before the savepoint.

**Technical Definition**
`ROLLBACK TO SAVEPOINT name` restores the transaction to the state it had at the savepoint: changes after the savepoint are undone, locks acquired after the savepoint are released (except those still needed by earlier work), and the transaction remains active. In PostgreSQL, this also clears an aborted transaction state, making the transaction usable again. The savepoint itself remains defined and can be rolled back to again.

**Beginner-Friendly Explanation**
`ROLLBACK TO SAVEPOINT` rewinds the transaction to the checkpoint but does not end the transaction. You can keep working after rewinding.

#### Purposes

- To discard a failed sub-operation without losing earlier work.
- To recover from errors inside a transaction (especially in PostgreSQL).
- To implement retry loops within a single transaction.
- To maintain transaction continuity across recoverable failures.

#### Syntax Rules and Structure

**Standard**:

```sql
ROLLBACK [ WORK ] TO [ SAVEPOINT ] savepoint_name;
```

**Component breakdown**:

| Component | Description |
|-----------|-------------|
| `ROLLBACK TO SAVEPOINT` | Rewinds to the savepoint. |
| `savepoint_name` | Target savepoint. |

**Syntax rules**

- The savepoint must exist.
- `ROLLBACK TO SAVEPOINT` does not end the transaction; `COMMIT` or a full `ROLLBACK` is still needed.
- In PostgreSQL, after an error, `ROLLBACK TO SAVEPOINT` restores the transaction to a usable state.
- In SQL Server, `ROLLBACK TRANSACTION savepoint_name` rolls back to the savepoint but does not decrement `@@TRANCOUNT`.

**Constraints and limitations**

- Cannot roll back to a savepoint that was released.
- Some changes (e.g., sequences in PostgreSQL, `INSERT` into an `IDENTITY` column in SQL Server) are not rolled back.
- Savepoints do not protect against certain errors (e.g., deadlocks) that abort the entire transaction.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Recovering from an error with `ROLLBACK TO SAVEPOINT`

```sql
BEGIN;
INSERT INTO orders (order_id, customer_id) VALUES (1001, 42);
SAVEPOINT sp_items;

INSERT INTO order_items (order_id, product_id) VALUES (1001, 7);
INSERT INTO order_items (order_id, product_id) VALUES (9999, 8);  -- FK error

ROLLBACK TO SAVEPOINT sp_items;  -- discard both item inserts, keep order

INSERT INTO order_items (order_id, product_id) VALUES (1001, 8);
COMMIT;
```

**Expected output**:

```text
BEGIN
INSERT 0 1
SAVEPOINT
INSERT 0 1
ERROR:  insert or update on table "order_items" violates foreign key constraint
ROLLBACK
INSERT 0 1
COMMIT
```

**Why this output appears**: The FK error aborts the transaction. `ROLLBACK TO SAVEPOINT sp_items` discards the failed item inserts but keeps the order insert. The transaction becomes usable again, and the corrected item is inserted and committed.

#### Real-World Cases

- **Batch imports**: Savepoint per row; roll back only failing rows.
- **Application retries**: Savepoint before a risky operation; roll back and retry.
- **Multi-step workflows**: Preserve earlier steps when a later step fails.

---

### 11. Implicit Transactions

#### Definitions

**Core Definition**
An implicit transaction is a transaction automatically opened and committed by the RDBMS around a single statement when no explicit transaction is active (autocommit mode).

**Technical Definition**
Under autocommit, each statement runs in its own transaction: the engine implicitly begins a transaction before the statement and commits it after successful completion. In SQL Server, `SET IMPLICIT_TRANSACTIONS ON` changes this behavior: the engine opens a transaction automatically before the first DML statement, but the transaction remains open until explicitly committed or rolled back—this is the "implicit transactions mode" that some client tools and ODBC drivers enable. MySQL's `autocommit = 1` (default) is statement-level autocommit; `SET autocommit = 0` is equivalent to explicit transactions. PostgreSQL always runs in autocommit unless `BEGIN` is issued; there is no "implicit transaction mode" that leaves transactions open.

**Beginner-Friendly Explanation**
An implicit transaction is a transaction the database creates for you without you asking. In autocommit mode, every single statement is its own mini-transaction that commits immediately. SQL Server also has a mode where it opens a transaction for you and leaves it open until you commit—a common source of confusion.

#### Purposes

- To provide atomicity for single statements without explicit transaction control.
- To reduce boilerplate for simple operations.
- To enable SQL Server compatibility with certain client tools that expect open transactions.
- To clarify the difference between statement-level autocommit and open implicit transactions.

#### Syntax Rules and Structure

**PostgreSQL** (statement-level autocommit):

```sql
-- Each statement commits immediately.
INSERT INTO t VALUES (1);
INSERT INTO t VALUES (2);
```

**MySQL**:

```sql
SET autocommit = 1;  -- statement-level autocommit (default)
SET autocommit = 0;  -- explicit transaction mode
```

**SQL Server** (implicit transactions mode):

```sql
SET IMPLICIT_TRANSACTIONS ON;
-- Now DML opens a transaction that stays open until COMMIT/ROLLBACK.
INSERT INTO t VALUES (1);
COMMIT;
```

**Component breakdown**:

| Mode | Behavior | Engine |
|------|----------|--------|
| Autocommit | Each statement commits | PostgreSQL, MySQL, SQL Server |
| Implicit transactions | DML opens a transaction, stays open | SQL Server |
| `autocommit = 0` | Like explicit transactions | MySQL |

**Syntax rules**

- In PostgreSQL, `autocommit` is a client setting (psql `\set AUTOCOMMIT on`), not a server variable; the server is always statement-level unless `BEGIN` is issued.
- SQL Server's implicit transactions mode is enabled by `SET IMPLICIT_TRANSACTIONS ON` or by certain client APIs (ODBC, OLE DB).
- MySQL's `autocommit` variable controls statement-level vs. explicit transactions.

**Constraints and limitations**

- Forgetting to commit in implicit transactions mode leaves transactions open, holding locks and blocking vacuum/cleanup.
- Autocommit provides no atomicity across multiple statements.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Implicit transactions mode

```sql
SET IMPLICIT_TRANSACTIONS ON;
-- @@TRANCOUNT is now 0.

INSERT INTO t VALUES (1);
-- A transaction is automatically opened; @@TRANCOUNT = 1.

INSERT INTO t VALUES (2);
-- Still in the same transaction; @@TRANCOUNT = 1.

COMMIT;
-- Transaction committed; @@TRANCOUNT = 0.
```

**Expected output**: Both inserts commit together at `COMMIT`. If `ROLLBACK` had been issued instead, both would be undone.

**Why this output appears**: `SET IMPLICIT_TRANSACTIONS ON` makes SQL Server open a transaction on the first DML statement and leave it open until explicit commit/rollback.

#### Real-World Cases

- **ODBC/OLE DB clients**: May enable implicit transactions automatically.
- **Simple CRUD**: Autocommit is sufficient for single-statement operations.
- **Migrating from autocommit to explicit**: Understanding the behavior change.

---

### 12. Autocommit

#### Definitions

**Core Definition**
Autocommit is a mode in which each SQL statement is automatically wrapped in its own transaction and committed immediately upon successful completion.

**Technical Definition**
When autocommit is on (the default in most engines), the RDBMS begins an implicit transaction before each statement and commits it after successful execution. A failed statement is rolled back without affecting prior or subsequent statements. Autocommit can be disabled at the session level (`SET autocommit = 0` in MySQL, `\set AUTOCOMMIT off` in psql, `SET IMPLICIT_TRANSACTIONS ON` in SQL Server) to require explicit `COMMIT`/`ROLLBACK`.

**Beginner-Friendly Explanation**
Autocommit means "save after every statement." You do not need to type `COMMIT` for simple single-statement operations; the database does it for you.

#### Purposes

- To simplify single-statement operations without explicit transaction control.
- To ensure that each statement is atomic on its own.
- To provide a safe default for interactive and ad-hoc SQL.
- To allow explicit transaction control to be layered on top when needed.

#### Syntax Rules and Structure

**PostgreSQL** (client-level):

```text
\set AUTOCOMMIT on    -- default in psql
\set AUTOCOMMIT off   -- requires explicit COMMIT
```

**MySQL**:

```sql
SET autocommit = 1;  -- default
SET autocommit = 0;  -- explicit transactions required
```

**SQL Server**:

```sql
-- Default is autocommit (no implicit transactions).
-- SET IMPLICIT_TRANSACTIONS OFF (default)
```

**Component breakdown**:

| Setting | Effect |
|---------|--------|
| Autocommit ON | Each statement commits immediately |
| Autocommit OFF | Statements accumulate until `COMMIT`/`ROLLBACK` |

**Syntax rules**

- Autocommit is a session setting, not a server-wide one.
- Turning autocommit off means you must explicitly commit; otherwise, the transaction stays open.
- Some client libraries always run in autocommit mode unless configured otherwise.

**Constraints and limitations**

- Autocommit provides no multi-statement atomicity.
- Open transactions from autocommit-off sessions can hold locks indefinitely if not committed.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Comparing autocommit modes

```sql
-- Autocommit ON (default)
SET autocommit = 1;
INSERT INTO t VALUES (1);  -- committed immediately
INSERT INTO t VALUES (2);  -- committed immediately

-- Autocommit OFF
SET autocommit = 0;
INSERT INTO t VALUES (3);
INSERT INTO t VALUES (4);
ROLLBACK;  -- both 3 and 4 discarded
```

**Expected output**: Rows 1 and 2 persist; rows 3 and 4 are rolled back.

**Why this output appears**: With autocommit on, each insert commits independently. With autocommit off, the inserts accumulate in one transaction until `ROLLBACK`.

#### Real-World Cases

- **Interactive SQL**: Autocommit for ad-hoc queries.
- **Application code**: Explicit transactions for multi-step operations.
- **Connection pools**: Understand whether the pool sets autocommit.

---

### 13. Nested Transactions

#### Definitions

**Core Definition**
Nested transactions are transactions within transactions, where an inner transaction can commit or roll back independently of the outer transaction—a feature that mainstream RDBMSs do not support natively; savepoints provide partial nesting.

**Technical Definition**
True nested transactions (with independent commit and rollback semantics) are part of some theoretical models and are supported by a few systems, but standard SQL and mainstream RDBMSs (PostgreSQL, MySQL, SQL Server, Oracle) do not implement them. Instead, they provide **savepoints**, which allow partial rollback within a single transaction but do not allow an inner transaction to commit independently. SQL Server's `BEGIN TRANSACTION` inside an existing transaction increments `@@TRANCOUNT`; a nested `COMMIT` decrements the count but does not actually commit until the outermost transaction commits. This is sometimes called "nested transactions" but is not the same as independent nested transactions.

**Beginner-Friendly Explanation**
Real nested transactions would let you commit an inner transaction while keeping the outer one open, or roll back the inner without affecting the outer. Mainstream databases do not do this. Instead, they offer savepoints (partial rollback) and, in SQL Server, a counter-based fake nesting.

#### Purposes

- To understand the limitations of nesting in mainstream RDBMSs.
- To use savepoints for partial rollback in place of true nesting.
- To recognize SQL Server's `@@TRANCOUNT` behavior and its pitfalls.
- To design application transaction management that does not rely on unsupported nesting.

#### Syntax Rules and Structure

**SQL Server fake nesting**:

```sql
BEGIN TRANSACTION;          -- @@TRANCOUNT = 1
  BEGIN TRANSACTION;        -- @@TRANCOUNT = 2
  COMMIT;                   -- @@TRANCOUNT = 1 (not actually committed)
COMMIT;                     -- @@TRANCOUNT = 0 (actually committed)
```

**Savepoint-based nesting (PostgreSQL/MySQL/Oracle)**:

```sql
BEGIN;
  SAVEPOINT outer;
  -- work
  SAVEPOINT inner;
  -- work
  ROLLBACK TO SAVEPOINT inner;  -- partial rollback
  RELEASE SAVEPOINT outer;
COMMIT;
```

**Component breakdown**:

| Mechanism | Independent commit? | Partial rollback? | Engine |
|-----------|---------------------|-------------------|--------|
| Savepoint | No | Yes | PG, MySQL, Oracle |
| SQL Server `BEGIN TRAN` nesting | No (`@@TRANCOUNT` only) | Via `SAVE TRAN` | SQL Server |
| True nested transactions | Yes | Yes | Not in mainstream RDBMSs |

**Syntax rules**

- SQL Server: `COMMIT` on an inner transaction only decrements `@@TRANCOUNT`; the outer transaction commits all work.
- SQL Server: `ROLLBACK` on an inner transaction rolls back the entire transaction and sets `@@TRANCOUNT = 0` (unless `SAVE TRANSACTION` was used).
- Savepoints provide partial rollback but are not independent transactions.

**Constraints and limitations**

- SQL Server's fake nesting can cause subtle bugs: an inner `ROLLBACK` destroys the outer transaction.
- Savepoints do not allow committing inner work independently.
- Application-level "nested transaction" abstractions (e.g., in ORMs) often use savepoints or counter-based emulation.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Fake nesting with `@@TRANCOUNT`

```sql
BEGIN TRANSACTION;              -- @@TRANCOUNT = 1
  INSERT INTO t VALUES (1);
  BEGIN TRANSACTION;            -- @@TRANCOUNT = 2
    INSERT INTO t VALUES (2);
  COMMIT;                       -- @@TRANCOUNT = 1, not committed yet
COMMIT;                         -- @@TRANCOUNT = 0, all rows committed

SELECT COUNT(*) FROM t;         -- 2
```

**Expected output**:

```text
(1 row affected)
(1 row affected)
(1 row affected)
 count
-------
     2
(1 row)
```

**Why this output appears**: The inner `COMMIT` only decrements `@@TRANCOUNT`; the actual commit happens at the outer `COMMIT`, at which point both inserts become permanent.

##### Example 2: PostgreSQL – Savepoint-based partial nesting

```sql
BEGIN;
INSERT INTO t VALUES (1);
SAVEPOINT inner;
INSERT INTO t VALUES (2);
ROLLBACK TO SAVEPOINT inner;  -- discards row 2
INSERT INTO t VALUES (3);
COMMIT;

SELECT * FROM t;  -- rows 1 and 3
```

**Expected output**:

```text
 id
----
  1
  3
(2 rows)
```

**Why this output appears**: The savepoint allows partial rollback of row 2 while keeping row 1. Row 3 is inserted after the rollback, and the transaction commits rows 1 and 3.

#### Real-World Cases

- **ORMs**: Frameworks often emulate nested transactions with savepoints.
- **Stored procedures**: Called procedures may start "transactions" that only increment `@@TRANCOUNT` in SQL Server.
- **Error isolation**: Savepoints isolate failures in a sub-operation without aborting the whole transaction.

---

## Summary: Transaction Control Matrix

| Statement | Purpose | PostgreSQL | MySQL | SQL Server | Oracle |
|-----------|---------|------------|-------|------------|--------|
| `BEGIN` / `START TRANSACTION` | Open transaction | `BEGIN` / `START TRANSACTION` | `START TRANSACTION` / `BEGIN` | `BEGIN TRANSACTION` | Implicit (first DML) |
| `COMMIT` | Save changes | `COMMIT` | `COMMIT` | `COMMIT TRANSACTION` | `COMMIT` |
| `ROLLBACK` | Discard changes | `ROLLBACK` | `ROLLBACK` | `ROLLBACK TRANSACTION` | `ROLLBACK` |
| `SAVEPOINT` | Mark partial rollback point | `SAVEPOINT name` | `SAVEPOINT name` | `SAVE TRANSACTION name` | `SAVEPOINT name` |
| `RELEASE SAVEPOINT` | Remove savepoint | `RELEASE SAVEPOINT name` | `RELEASE SAVEPOINT name` | Not supported | Not supported |
| `ROLLBACK TO SAVEPOINT` | Partial rollback | `ROLLBACK TO SAVEPOINT name` | `ROLLBACK TO SAVEPOINT name` | `ROLLBACK TRANSACTION name` | `ROLLBACK TO SAVEPOINT name` |
| Autocommit | Per-statement commit | Client setting (`\set AUTOCOMMIT`) | `SET autocommit` | Default; `IMPLICIT_TRANSACTIONS` for open | Implicit per statement |
| True nested transactions | Independent commit/rollback | Not supported | Not supported | Fake via `@@TRANCOUNT` | Not supported |

---

## References

- PostgreSQL Documentation – Transactions – https://www.postgresql.org/docs/current/tutorial-transactions.html
- PostgreSQL Documentation – `BEGIN` – https://www.postgresql.org/docs/current/sql-begin.html
- PostgreSQL Documentation – `COMMIT` – https://www.postgresql.org/docs/current/sql-commit.html
- PostgreSQL Documentation – `ROLLBACK` – https://www.postgresql.org/docs/current/sql-rollback.html
- PostgreSQL Documentation – `SAVEPOINT` – https://www.postgresql.org/docs/current/sql-savepoint.html
- PostgreSQL Documentation – `RELEASE SAVEPOINT` – https://www.postgresql.org/docs/current/sql-release-savepoint.html
- PostgreSQL Documentation – `ROLLBACK TO SAVEPOINT` – https://www.postgresql.org/docs/current/sql-rollback-to.html
- PostgreSQL Documentation – Transaction Isolation – https://www.postgresql.org/docs/current/transaction-iso.html
- MySQL 8.0 Reference Manual – Transaction and Atomic Operation Statements – https://dev.mysql.com/doc/refman/8.0/en/commit.html
- MySQL 8.0 Reference Manual – `START TRANSACTION` – https://dev.mysql.com/doc/refman/8.0/en/commit.html
- MySQL 8.0 Reference Manual – `SAVEPOINT` – https://dev.mysql.com/doc/refman/8.0/en/savepoint.html
- MySQL 8.0 Reference Manual – InnoDB Transaction Model – https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-model.html
- Microsoft Learn – Transactions (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/language-elements/transactions-transact-sql
- Microsoft Learn – `BEGIN TRANSACTION` – https://learn.microsoft.com/en-us/sql/t-sql/language-elements/begin-transaction-transact-sql
- Microsoft Learn – `SAVE TRANSACTION` – https://learn.microsoft.com/en-us/sql/t-sql/language-elements/save-transaction-transact-sql
- Microsoft Learn – `SET IMPLICIT_TRANSACTIONS` – https://learn.microsoft.com/en-us/sql/t-sql/statements/set-implicit-transactions-transact-sql
- Oracle Database SQL Language Reference – `COMMIT` – https://docs.oracle.com/en/database/oracle/oracle-database/23/sqlrf/COMMIT.html
- Oracle Database SQL Language Reference – `SAVEPOINT` – https://docs.oracle.com/en/database/oracle/oracle-database/23/sqlrf/SAVEPOINT.html
- Oracle Database Concepts – Transactions – https://docs.oracle.com/en/database/oracle/oracle-database/23/cncpt/transactions.html
- SQL:1992 Standard – Transaction Management (ISO/IEC 9075:1992)
- Martin Kleppmann, Designing Data-Intensive Applications, Chapter 7: Transactions – https://dataintensive.net/
- PostgreSQL Wiki – Transaction Isolation – https://wiki.postgresql.org/wiki/Transaction_Isolation