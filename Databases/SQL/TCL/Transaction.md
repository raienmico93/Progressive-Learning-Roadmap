# SQL Transaction Fundamentals: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A SQL transaction is a logical unit of work that groups one or more SQL statements into an all-or-nothing operation, ensuring that either all changes are applied to the database or none are.

**Technical Definition:** Per the SQL standard (ISO/IEC 9075-2), a transaction is a sequence of operations performed as a single logical unit of work. The transaction boundaries are defined by `BEGIN` (or `START TRANSACTION`), `COMMIT`, and `ROLLBACK`. Transactions must comply with the ACID properties: **Atomicity** (all tasks performed or none), **Consistency** (database moves from one valid state to another), **Isolation** (concurrent transactions do not interfere), and **Durability** (committed changes persist despite system failures).

**Beginner-Friendly Explanation:** A transaction is like a safety wrapper around a group of database changes. If everything succeeds, you save all the changes together. If something fails, you undo everything as if nothing happened. It's the database's way of ensuring that your data never ends up in a half-finished state.

### Key Characteristics

- **All-or-nothing boundary:** A transaction groups SQL statements so they are either all committed or all rolled back.
- **ACID compliance:** Transactions must exhibit Atomicity, Consistency, Isolation, and Durability.
- **Session-scoped:** A transaction is associated with a single database connection (session).
- **Isolation levels:** Different levels (READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE) control how concurrent transactions interact.
- **Savepoints:** Intermediate markers within a transaction allow partial rollbacks without discarding the entire transaction.
- **Dialect variations:** Syntax for starting transactions, autocommit defaults, and DDL behaviour varies across RDBMSs.

### Prerequisites

- **Basic SQL knowledge:** Familiarity with `SELECT`, `INSERT`, `UPDATE`, and `DELETE`.
- **Connection concepts:** Understanding of database sessions and connections.
- **ACID awareness:** Basic understanding of the ACID properties.
- **Application driver knowledge:** Familiarity with how database drivers handle transactions.

### Related Programming Areas

- **Application Development:** Backend services use transactions to ensure data integrity in multi-step operations.
- **Financial Systems:** Banking transfers, payments, and order processing rely on transactional atomicity.
- **Data Engineering (ETL):** Transactions ensure that batch loads are applied atomically.
- **Database Administration:** DBAs monitor and tune transaction isolation levels and lock contention.
- **Distributed Systems:** Distributed transactions coordinate across multiple databases.

### Core Concepts / Features

1. The Transactional Unit: Defining an All-or-Nothing Boundary
2. BEGIN TRANSACTION / START TRANSACTION
3. COMMIT
4. ROLLBACK
5. Savepoints & Partial Rollbacks
6. Implicit vs. Explicit Transactions & Autocommit
7. Transaction Isolation Levels (ACID "I")

---

## 1. The Transactional Unit: Defining an All-or-Nothing Logical Boundary

### Definitions

**Core Definition:** A transactional unit is a logical boundary that groups a set of database mutations into a single, indivisible operation, ensuring that the database either reflects all changes or none.

**Technical Definition:** The transaction is the unit of recovery. If anything fails partway through, the engine can return the data to its pre-transaction state without manual cleanup. The transaction lifecycle has three points of control: `BEGIN` → `EXECUTE STATEMENTS` → `COMMIT` (persist) or `ROLLBACK` (discard).

**Beginner-Friendly Explanation:** A transactional unit is a "safety box" around your changes. You put all your changes inside the box, and the database guarantees that the box is either fully applied or fully discarded. It's like writing a document in a text editor: either you save the whole document, or you discard all your edits.

### Purposes

- To ensure data integrity by preventing partial updates.
- To provide recovery from logical or physical system failures while keeping the database in a consistent state.
- To provide isolation between concurrent users and programs.
- To define a clear boundary for committing or rolling back changes.
- To satisfy the Atomicity and Durability requirements of ACID.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
BEGIN [TRANSACTION | WORK];
    -- SQL statements (DML operations)
COMMIT | ROLLBACK;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `BEGIN` | Initiates the transaction boundary |
| SQL statements | The operations to be executed atomically |
| `COMMIT` | Permanently applies all changes |
| `ROLLBACK` | Discards all changes |

**Syntax Rules:**

- The transaction boundary begins with `BEGIN` and ends with `COMMIT` or `ROLLBACK`.
- If no explicit transaction is started, each statement may be treated as its own transaction (autocommit).
- DDL statements (e.g., `CREATE`, `ALTER`, `DROP`) may cause implicit commits in some RDBMSs (MySQL, Oracle), breaking transaction boundaries.
- Identity and sequence values do not roll back on any major engine; gaps in auto-increment columns are normal after rollbacks.

**Constraints and Limitations:**

- Not all SQL statements can be executed inside a transaction (e.g., some DDL statements in certain dialects).
- Long-running transactions can hold locks and block other operations.
- Distributed transactions across multiple databases require special protocols (e.g., two-phase commit).

### Annotated Complete Code Examples

**Example 1: Basic Transaction Boundary**

```sql
-- Setup: Create accounts table
CREATE TABLE accounts (
    account_id INTEGER PRIMARY KEY,
    balance    NUMERIC(10, 2)
);

INSERT INTO accounts VALUES (1, 1000.00), (2, 500.00);

-- Begin a transaction
BEGIN;

-- Transfer $200 from account 1 to account 2
UPDATE accounts SET balance = balance - 200 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 200 WHERE account_id = 2;

-- Commit the transaction
COMMIT;

-- Verify the result
SELECT * FROM accounts;

-- Expected Output:
--  account_id | balance
-- ------------+---------
--           1 |  800.00
--           2 |  700.00
```

**Why this output occurs:** Both `UPDATE` statements are executed inside the transaction. The `COMMIT` permanently applies both changes. If an error had occurred after the first `UPDATE`, a `ROLLBACK` would have restored both balances to their original values.

**Example 2: Transaction with Error and Rollback**

```sql
-- Begin a transaction
BEGIN;

-- Attempt to transfer more than the available balance
UPDATE accounts SET balance = balance - 1500 WHERE account_id = 1;

-- Check for a business rule violation (insufficient funds)
-- If the balance would go negative, roll back
ROLLBACK;

-- Verify: balances are unchanged
SELECT * FROM accounts;

-- Expected Output:
--  account_id | balance
-- ------------+---------
--           1 |  800.00
--           2 |  700.00
```

**Why this output occurs:** The `ROLLBACK` undoes the `UPDATE`, restoring the balance to 800.00. Without the rollback, the balance would have become negative, violating the business rule.

### Real-World Cases

**Case 1: Bank Transfer**

A bank transfer involves debiting one account and crediting another. Both operations are wrapped in a transaction. If the credit fails (e.g., due to a constraint violation), the debit is rolled back, ensuring no money is lost.

**Case 2: Order Processing**

An e-commerce checkout inserts an order, order lines, and a payment record. If any of these operations fails, the entire transaction is rolled back, preventing orphaned orders.

### References

- AWS — Transactions for T-SQL - https://docs.aws.amazon.com/dms/latest/sql-server-to-aurora-postgresql-migration-playbook/chap-sql-server-aurora-pg.tsql.transactions.html
- Microsoft Learn — SQL Server: Concurrency Control Models, ACID Properties and Transaction Isolation Levels - https://learn.microsoft.com/en-us/archive/technet-wiki/51484.sql-server-concurrency-control-models-acid-properties-and-transaction-isolation-levels
- DigitalOcean — SQL Commit And Rollback - https://www.digitalocean.com/community/tutorials/sql-commit-sql-rollback

---

## 2. BEGIN TRANSACTION / START TRANSACTION

### Definitions

**Core Definition:** `BEGIN TRANSACTION` (or `START TRANSACTION`) is the SQL statement that initiates a new transaction, marking the beginning of a context window in which all subsequent SQL statements are part of a single atomic unit.

**Technical Definition:** `BEGIN` switches the session into an explicit transaction mode until `COMMIT` or `ROLLBACK` is issued. The ANSI SQL standard defines `START TRANSACTION` as the statement for starting new transactions. In MySQL, `BEGIN` is an alias for `START TRANSACTION`. In SQL Server, `BEGIN TRANSACTION` (or `BEGIN TRAN`) is required. In Oracle, the first DML statement starts the transaction implicitly; there is no explicit `BEGIN` statement.

**Beginner-Friendly Explanation:** `BEGIN` is the command that tells the database "I'm about to make several changes, and I want them all treated as one unit." It opens the transaction window. Everything you do after `BEGIN` is part of the same transaction until you `COMMIT` or `ROLLBACK`.

### Purposes

- To explicitly start a transaction and define its beginning boundary.
- To group multiple SQL statements into a single atomic unit.
- To gain control over when changes become permanent.
- To enable rollback of a set of changes if an error occurs.
- To set transaction attributes (e.g., isolation level) before executing statements.

### Syntax Rules and Structure

**Complete General Syntax (Standard SQL):**

```sql
START TRANSACTION [transaction_mode [, ...]];
```

**Dialect-Specific Syntax:**

| Dialect | Syntax | Notes |
|---------|--------|-------|
| PostgreSQL | `BEGIN;` or `START TRANSACTION;` | `BEGIN` is the traditional form |
| MySQL | `START TRANSACTION;` or `BEGIN;` | `BEGIN` is an alias |
| SQL Server | `BEGIN TRANSACTION;` or `BEGIN TRAN;` | Requires `TRANSACTION` keyword |
| Oracle | Implicit (first DML statement) | No explicit `BEGIN` statement |

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `BEGIN` / `START` | Initiates the transaction |
| `TRANSACTION` | Optional keyword (required in SQL Server) |
| `WORK` | Optional synonym for `TRANSACTION` |

**Syntax Rules:**

- `BEGIN` is a keyword that must appear at the start of the transaction.
- In PostgreSQL, `BEGIN` switches the session into an explicit transaction until `COMMIT` or `ROLLBACK`.
- In MySQL, `BEGIN` and `START TRANSACTION` are equivalent.
- In SQL Server, `BEGIN TRANSACTION` is required; `BEGIN` alone is a different statement (begin/end block).
- In Oracle, transactions start implicitly with the first DML statement; `SET TRANSACTION` only sets attributes and does not start a transaction.

**Constraints and Limitations:**

- Oracle does not support an explicit `BEGIN` transaction statement.
- In autocommit mode, a `BEGIN` statement may be required to disable autocommit for the duration of the transaction.
- Some drivers automatically issue `BEGIN` when a transaction is started programmatically.

### Annotated Complete Code Examples

**Example 1: BEGIN in PostgreSQL**

```sql
-- PostgreSQL: Begin an explicit transaction
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

COMMIT;

-- Verify
SELECT * FROM accounts;
```

**Why this works:** `BEGIN` starts the explicit transaction. Both `UPDATE` statements are part of the transaction. `COMMIT` makes the changes permanent.

**Example 2: BEGIN TRANSACTION in SQL Server**

```sql
-- SQL Server: Begin an explicit transaction
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

COMMIT TRANSACTION;
```

**Why this works:** SQL Server requires `BEGIN TRANSACTION` (or `BEGIN TRAN`). The transaction is committed with `COMMIT TRANSACTION` (or `COMMIT`).

**Example 3: Implicit Start in Oracle**

```sql
-- Oracle: No BEGIN statement; the first DML starts the transaction
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

COMMIT;
```

**Why this works:** In Oracle, the first `UPDATE` implicitly starts the transaction. The `COMMIT` permanently saves the changes.

### Real-World Cases

**Case 1: Multi-Step Data Entry**

A data entry application uses `BEGIN` to start a transaction before inserting a customer record and its associated addresses. If any insert fails, a `ROLLBACK` discards all changes.

**Case 2: Batch Processing**

An ETL pipeline uses `BEGIN` to start a transaction before loading a batch of records. If the batch fails, the entire batch is rolled back, ensuring data consistency.

### References

- DigitalOcean — COMMIT and ROLLBACK Syntax Across Database Platforms - https://www.digitalocean.com/rss/community/tags/sql.atom
- O'Reilly — SQL in a Nutshell (Transaction Start Syntax) - https://www.oreilly.com/library/view/sql-in-a/1565927443/re40.html
- AWS — Transactions for T-SQL - https://docs.aws.amazon.com/dms/latest/sql-server-to-aurora-postgresql-migration-playbook/chap-sql-server-aurora-pg.tsql.transactions.html

---

## 3. COMMIT

### Definitions

**Core Definition:** `COMMIT` is the SQL statement that permanently saves all changes made during the current transaction, making them visible to other sessions and durable across system failures.

**Technical Definition:** `COMMIT` explicitly ends an open transaction and makes the changes permanent in the database. Once `COMMIT` returns, standard transaction control cannot undo the changes, and recovery requires a backup or engine-specific point-in-time recovery. The `COMMIT` keyword is functionally equivalent to `COMMIT WORK AND NO CHAIN`.

**Beginner-Friendly Explanation:** `COMMIT` is like clicking "Save" after editing a document. It writes all your changes to the database permanently. Once you commit, you can't undo the changes with a simple rollback — they're saved for good.

### Purposes

- To permanently apply all changes made during the transaction.
- To release locks held by the transaction.
- To make changes visible to other database sessions.
- To satisfy the Durability requirement of ACID.
- To end the transaction and return the session to autocommit mode.

### Syntax Rules and Structure

**Complete General Syntax (Standard SQL):**

```sql
COMMIT [WORK] [AND [NO] CHAIN];
```

**Dialect-Specific Syntax:**

| Dialect | Syntax | Notes |
|---------|--------|-------|
| PostgreSQL | `COMMIT;` or `END;` | `END` is a synonym for `COMMIT` |
| MySQL | `COMMIT;` | Autocommit is on by default |
| SQL Server | `COMMIT;` or `COMMIT TRANSACTION;` | Autocommit is the default |
| Oracle | `COMMIT;` | SQL*Plus does not autocommit |

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `COMMIT` | Permanently saves the transaction |
| `WORK` | Optional keyword (no functional difference) |
| `AND CHAIN` | Starts a new transaction with the same attributes immediately |
| `AND NO CHAIN` | Ends the transaction (default behaviour) |

**Syntax Rules:**

- `COMMIT` ends the current transaction and makes all changes permanent.
- `COMMIT WORK` is functionally equivalent to `COMMIT`.
- `COMMIT AND CHAIN` immediately starts a new transaction with the same isolation level.
- After `COMMIT`, the session returns to autocommit (or implicit-transaction) mode.
- If no transaction is active: MySQL and Oracle silently succeed; PostgreSQL emits a warning; SQL Server raises an error.

**Constraints and Limitations:**

- Once `COMMIT` is executed, the changes cannot be undone by `ROLLBACK`.
- DDL statements in MySQL and Oracle cause implicit commits, committing any pending DML before the DDL runs.
- Identity and sequence values do not roll back, so gaps in auto-increment columns persist after rollbacks.

### Annotated Complete Code Examples

**Example 1: COMMIT After Multiple Updates**

```sql
BEGIN;

UPDATE accounts SET balance = balance - 200 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 200 WHERE account_id = 2;

COMMIT;

-- Verify
SELECT * FROM accounts;

-- Expected Output:
--  account_id | balance
-- ------------+---------
--           1 |  600.00
--           2 |  900.00
```

**Why this output occurs:** Both updates are applied and then permanently saved with `COMMIT`. The balances reflect the transfer.

**Example 2: COMMIT with AND CHAIN (PostgreSQL)**

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;

COMMIT AND CHAIN;

-- A new transaction has started automatically
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

COMMIT;
```

**Why this works:** `COMMIT AND CHAIN` commits the first transaction and immediately starts a new one with the same attributes. The second `UPDATE` is part of the new transaction, which is then committed separately.

### Real-World Cases

**Case 1: E-Commerce Order Placement**

An e-commerce system commits an order transaction after inserting the order, order lines, and payment record. The `COMMIT` ensures all records are saved together.

**Case 2: Batch Data Load**

An ETL process commits after loading each batch of records. The `COMMIT` makes the batch durable and releases locks for the next batch.

### References

- DigitalOcean — SQL Commit And Rollback - https://www.digitalocean.com/community/tutorials/sql-commit-sql-rollback
- O'Reilly — SQL in a Nutshell (COMMIT Syntax) - https://www.oreilly.com/library/view/sql-in-a/1565927443/re40.html
- DigitalOcean — COMMIT Syntax Across Database Platforms - https://www.digitalocean.com/rss/community/tags/sql.atom

---

## 4. ROLLBACK

### Definitions

**Core Definition:** `ROLLBACK` is the SQL statement that undoes all changes made during the current transaction, reverting the database to its state before the transaction began.

**Technical Definition:** The `ROLLBACK` statement undoes a transaction to its beginning or to a previously declared `SAVEPOINT`. It closes open cursors and releases locks in the same way as `COMMIT`. After `ROLLBACK`, the transaction ends on every engine and the session returns to autocommit (or implicit-transaction) mode.

**Beginner-Friendly Explanation:** `ROLLBACK` is like clicking "Undo" on everything you did since you started the transaction. It takes the database back to the state it was in before you began, as if none of your changes ever happened.

### Purposes

- To undo all changes made during the transaction.
- To recover from errors, constraint violations, or deadlocks.
- To release locks held by the transaction.
- To return the session to a clean state for retry.
- To maintain data integrity when a business rule violation occurs.

### Syntax Rules and Structure

**Complete General Syntax (Standard SQL):**

```sql
ROLLBACK [WORK] [AND [NO] CHAIN] [TO SAVEPOINT savepoint_name];
```

**Dialect-Specific Syntax:**

| Dialect | Syntax | Notes |
|---------|--------|-------|
| PostgreSQL | `ROLLBACK;` or `ROLLBACK TO SAVEPOINT name;` | Aborted state requires `ROLLBACK` |
| MySQL | `ROLLBACK;` | Autocommit on by default |
| SQL Server | `ROLLBACK;` or `ROLLBACK TRANSACTION;` | Uses `ROLLBACK TRAN savepoint_name` |
| Oracle | `ROLLBACK;` | SQL*Plus does not autocommit |

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `ROLLBACK` | Undoes the transaction |
| `WORK` | Optional keyword |
| `TO SAVEPOINT name` | Rolls back to a specific savepoint |
| `AND CHAIN` | Starts a new transaction with the same attributes |

**Syntax Rules:**

- `ROLLBACK` undoes all changes made since `BEGIN` or the last `SAVEPOINT`.
- `ROLLBACK TO SAVEPOINT name` undoes changes back to the named savepoint but does not end the transaction.
- After `ROLLBACK`, the session returns to autocommit mode.
- In PostgreSQL, if a statement raises an error and is not caught, the transaction enters an aborted state and refuses all commands except `ROLLBACK`.

**Constraints and Limitations:**

- `ROLLBACK` cannot undo changes that have already been committed.
- Rolling back a large transaction can be time-consuming.
- Identity and sequence values do not roll back; gaps persist.
- `ROLLBACK` normally frees locks, but does not free locks when rolling back to a savepoint.

### Annotated Complete Code Examples

**Example 1: ROLLBACK After Error**

```sql
BEGIN;

UPDATE accounts SET balance = balance - 1500 WHERE account_id = 1;

-- Business rule violation detected: balance would go negative
ROLLBACK;

-- Verify: balance is unchanged
SELECT * FROM accounts WHERE account_id = 1;

-- Expected Output:
--  account_id | balance
-- ------------+---------
--           1 |  600.00
```

**Why this output occurs:** The `ROLLBACK` undoes the `UPDATE`, restoring the balance to its pre-transaction value. The negative balance never persists.

**Example 2: ROLLBACK in PostgreSQL After Error**

```sql
BEGIN;

-- This will cause an error (e.g., division by zero)
UPDATE accounts SET balance = balance / 0 WHERE account_id = 1;

-- PostgreSQL enters an aborted state; only ROLLBACK is allowed
ROLLBACK;

-- The session is now clean and ready for new commands
SELECT * FROM accounts;
```

**Why this works:** In PostgreSQL, an uncaught error puts the transaction in an aborted state. The `ROLLBACK` clears the aborted state and returns the session to normal operation.

### Real-World Cases

**Case 1: Failed Payment Processing**

An e-commerce system rolls back an order transaction if the payment gateway returns a decline. The `ROLLBACK` discards the order and order lines, preventing an unpaid order from being created.

**Case 2: Constraint Violation Recovery**

A data import process rolls back a transaction when a foreign key constraint violation occurs, preventing partially loaded data from corrupting the database.

### References

- O'Reilly — SQL in a Nutshell (ROLLBACK Syntax) - https://www.oreilly.com/library/view/sql-in-a/1565927443/re40.html
- DigitalOcean — SQL Commit And Rollback - https://www.digitalocean.com/community/tutorials/sql-commit-sql-rollback
- DigitalOcean — COMMIT and ROLLBACK Syntax Across Database Platforms - https://www.digitalocean.com/rss/community/tags/sql.atom

---

## 5. Savepoints & Partial Rollbacks

### Definitions

**Core Definition:** A `SAVEPOINT` is a named marker within a transaction that allows partial rollback to that point without discarding the entire transaction.

**Technical Definition:** The `SAVEPOINT` command defines a marker state within a transaction. Transactions may be partially rolled back to a unique savepoint marker using the `ROLLBACK TO SAVEPOINT` command. Different RDBMSs use incompatible syntax for nested savepoints: SQL-92 uses `SAVEPOINT name` / `ROLLBACK TO SAVEPOINT name` / `RELEASE SAVEPOINT name` (PostgreSQL, MySQL, MariaDB, Oracle, SQLite, Db2, Snowflake); SQL Server uses `SAVE TRANSACTION name` / `ROLLBACK TRANSACTION name` (no `RELEASE` — savepoints are released automatically on commit/rollback).

**Beginner-Friendly Explanation:** A savepoint is like a bookmark in a book. You can mark a point in your transaction, continue making changes, and if something goes wrong later, you can roll back to the bookmark instead of rolling back the entire transaction. It saves you from having to redo all the work you did before the bookmark.

### Purposes

- To allow partial rollback within a transaction without discarding all changes.
- To handle errors at a granular level (e.g., roll back only the failed step).
- To implement nested transaction logic in databases that do not support true nested transactions.
- To provide step-by-step logical retreats during complex data operations.
- To enable retry logic for specific portions of a transaction.

### Syntax Rules and Structure

**Complete General Syntax (SQL-92):**

```sql
SAVEPOINT savepoint_name;
-- ... SQL statements ...
ROLLBACK TO SAVEPOINT savepoint_name;
-- ... or ...
RELEASE SAVEPOINT savepoint_name;
```

**SQL Server Syntax:**

```sql
SAVE TRANSACTION savepoint_name;
-- ... SQL statements ...
ROLLBACK TRANSACTION savepoint_name;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `SAVEPOINT name` | Creates a named marker within the transaction |
| `ROLLBACK TO SAVEPOINT name` | Rolls back to the marker, undoing changes after it |
| `RELEASE SAVEPOINT name` | Removes the savepoint (but does not commit) |

**Syntax Rules:**

- Savepoints are created within an open transaction; they do not start a new transaction.
- `ROLLBACK TO SAVEPOINT` undoes changes made after the savepoint but keeps the transaction open.
- `RELEASE SAVEPOINT` removes the savepoint; the transaction remains active.
- Savepoints are released automatically when the transaction is committed or rolled back.
- Reusing savepoint names is engine-specific; generate unique names per checkpoint.
- SQL Server uses `SAVE TRANSACTION` and `ROLLBACK TRANSACTION` instead of `SAVEPOINT` and `ROLLBACK TO SAVEPOINT`.

**Constraints and Limitations:**

- Savepoints do not free locks when rolling back to them.
- In PostgreSQL, savepoints can be used to recover from errors within a transaction.
- MySQL's `ROLLBACK TO SAVEPOINT` does not free row locks held after the savepoint.
- Savepoints add overhead; use them judiciously in high-volume transactions.

### Annotated Complete Code Examples

**Example 1: Savepoint and Partial Rollback (PostgreSQL/MySQL/Oracle)**

```sql
BEGIN;

-- Step 1: Insert a new account
INSERT INTO accounts VALUES (3, 300.00);

-- Create a savepoint after step 1
SAVEPOINT after_insert;

-- Step 2: Attempt to transfer from the new account (may fail)
UPDATE accounts SET balance = balance - 500 WHERE account_id = 3;

-- Business rule violation: balance would go negative
ROLLBACK TO SAVEPOINT after_insert;

-- Step 2 is undone, but Step 1 (INSERT) is preserved
UPDATE accounts SET balance = balance + 500 WHERE account_id = 2;

COMMIT;

-- Verify: account 3 exists, account 2 was credited
SELECT * FROM accounts;

-- Expected Output:
--  account_id | balance
-- ------------+---------
--           1 |  600.00
--           2 | 1400.00
--           3 |  300.00
```

**Why this output occurs:** The `SAVEPOINT after_insert` marks the state after the `INSERT`. When the `UPDATE` for account 3 is rolled back to the savepoint, the `INSERT` (step 1) remains part of the transaction. The subsequent `UPDATE` for account 2 is also part of the transaction. The final `COMMIT` saves the insert and the credit to account 2.

**Example 2: SQL Server Savepoint**

```sql
BEGIN TRANSACTION;

INSERT INTO accounts VALUES (4, 400.00);

SAVE TRANSACTION after_insert;

UPDATE accounts SET balance = balance - 500 WHERE account_id = 4;

-- Business rule violation: roll back to savepoint
ROLLBACK TRANSACTION after_insert;

UPDATE accounts SET balance = balance + 500 WHERE account_id = 2;

COMMIT TRANSACTION;

-- Verify
SELECT * FROM accounts;
```

**Why this works:** SQL Server uses `SAVE TRANSACTION` and `ROLLBACK TRANSACTION savepoint_name`. The insert is preserved, the failed update is rolled back, and the successful update is committed.

### Real-World Cases

**Case 1: Multi-Step Order Processing**

An order processing system uses savepoints to handle partial failures. If the inventory reservation fails after the order header is inserted, the system rolls back to the savepoint and retries with a different warehouse, without losing the order header.

**Case 2: Data Migration with Error Recovery**

A data migration script uses savepoints at each major step. If a step fails (e.g., due to a constraint violation), the script rolls back to the savepoint and logs the error, continuing with the next step without discarding the entire migration.

### References

- Pub.dev — SavepointDialect enum (SQL-92 and SQL Server syntax) - https://pub.dev/documentation/odbc_fast/4.5.0/domain_entities_savepoint_dialect/SavepointDialect.html
- O'Reilly — SQL in a Nutshell (SAVEPOINT) - https://www.oreilly.com/library/view/sql-in-a/1565927443/re40.html
- DigitalOcean — SQL Commit And Rollback (Savepoint Usage) - https://www.digitalocean.com/community/tutorials/sql-commit-sql-rollback

---

## 6. Implicit vs. Explicit Transactions & Autocommit

### Definitions

**Core Definition:** Implicit transactions are automatically started by the database engine without an explicit `BEGIN` statement, while explicit transactions are started manually with `BEGIN`/`START TRANSACTION`. Autocommit mode controls whether each statement is automatically committed or held until an explicit `COMMIT`.

**Technical Definition:** A connection operates in autocommit mode whenever the default mode has not been overridden by either explicit or implicit transactions. When the explicit transaction is committed or rolled back, or when implicit transaction mode is turned off, the connection returns to autocommit mode. In autocommit mode, each individual SQL statement is treated as its own transaction and committed immediately upon successful execution, without requiring an explicit `COMMIT`. The `AutoCommit` property specifies whether to commit each SQL statement implicitly or only until an explicit commit occurs. By default, this property is set to `true`, which commits each SQL statement implicitly.

**Beginner-Friendly Explanation:** Autocommit is like a "save as you go" mode. Every statement you run is automatically saved. Explicit transactions are like "save all at once" mode — you make all your changes, then save them together with `COMMIT`. Implicit transactions are when the database automatically starts a transaction for you, but you still have to `COMMIT` to save.

### Purposes

- To control when changes become permanent in the database.
- To balance performance (autocommit is faster for single statements) with consistency (explicit transactions ensure atomicity).
- To manage transaction boundaries in application code and database drivers.
- To avoid accidental partial commits in multi-step operations.
- To comply with application framework and driver defaults.

### Syntax Rules and Structure

**Autocommit Mode:**

| Dialect | Default Autocommit | Notes |
|---------|-------------------|-------|
| MySQL | ON | Each statement commits immediately |
| SQL Server | ON | Autocommit is the default |
| PostgreSQL | ON (client-side) | No server-level autocommit; client controls it |
| Oracle | OFF in SQL*Plus | SQL*Plus does not autocommit; `SET AUTOCOMMIT ON` changes this |

**Implicit Transaction Modes:**

| Dialect | Setting | Behaviour |
|---------|---------|-----------|
| SQL Server | `SET IMPLICIT_TRANSACTIONS ON` | DML statements automatically start a transaction without `BEGIN` |
| MySQL | N/A | Uses autocommit or explicit `START TRANSACTION` |
| PostgreSQL | N/A | Uses autocommit or explicit `BEGIN` |
| Oracle | N/A | First DML starts transaction implicitly |

**Syntax Rules (SQL Server Implicit Transactions):**

```sql
SET IMPLICIT_TRANSACTIONS ON;
-- DML statements automatically start a transaction
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
-- Must explicitly COMMIT or ROLLBACK
COMMIT;
```

**Component Breakdown:**

| Mode | Transaction Start | Transaction End |
|------|------------------|-----------------|
| Autocommit | Each statement | Automatic after each statement |
| Explicit | `BEGIN` / `START TRANSACTION` | `COMMIT` / `ROLLBACK` |
| Implicit | Automatically by DML | `COMMIT` / `ROLLBACK` |

**Syntax Rules:**

- Autocommit mode is the default in most RDBMSs; each statement is its own transaction.
- Explicit transactions require `BEGIN` (or equivalent) to start and `COMMIT`/`ROLLBACK` to end.
- Implicit transactions (SQL Server) start automatically on DML statements but require explicit `COMMIT`/`ROLLBACK`.
- Application drivers (JDBC, ODBC, ODP.NET) control autocommit programmatically.
- In Oracle, DDL statements cause implicit commits, committing any pending DML before the DDL runs.

**Constraints and Limitations:**

- Autocommit mode does not allow multi-statement rollback; each statement is committed independently.
- Implicit transactions can leave transactions open if `COMMIT`/`ROLLBACK` is forgotten, causing locks to be held.
- DDL statements in MySQL and Oracle cause implicit commits, breaking transaction boundaries.
- Application driver autocommit settings may override database defaults.

### Annotated Complete Code Examples

**Example 1: Explicit Transaction (Autocommit Off)**

```sql
-- Disable autocommit for the session (MySQL)
SET autocommit = 0;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

-- Changes are not visible to other sessions until COMMIT
COMMIT;

-- Re-enable autocommit
SET autocommit = 1;
```

**Why this works:** With autocommit off, both updates are part of an explicit transaction. `COMMIT` makes them permanent.

**Example 2: SQL Server Implicit Transactions**

```sql
SET IMPLICIT_TRANSACTIONS ON;

-- DML statement automatically starts a transaction
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;

-- The transaction is still open; must explicitly commit
COMMIT;

SET IMPLICIT_TRANSACTIONS OFF;
```

**Why this works:** `SET IMPLICIT_TRANSACTIONS ON` causes the `UPDATE` to automatically start a transaction. The `COMMIT` must be issued explicitly to save the change.

### Real-World Cases

**Case 1: Application Driver Configuration**

A Java application using JDBC sets `connection.setAutoCommit(false)` to enable explicit transaction control. The application issues `connection.commit()` or `connection.rollback()` based on business logic.

**Case 2: SQL Server Implicit Transactions**

A legacy application uses `SET IMPLICIT_TRANSACTIONS ON` to automatically start transactions for each DML statement, ensuring that changes are not committed until explicitly saved.

### References

- Oracle — ODP.NET AutoCommit Property - https://docs.oracle.com/en/database/oracle/oracle-database/26/odpnt/ConnectionIsInAutoCommit.html
- Microsoft Learn — Autocommit Transactions - https://learn.microsoft.com/en-us/previous-versions/
- DigitalOcean — COMMIT and ROLLBACK Syntax Across Database Platforms - https://www.digitalocean.com/rss/community/tags/sql.atom

---

## 7. Transaction Isolation Levels (ACID "I")

### Definitions

**Core Definition:** Transaction isolation levels define the degree to which one transaction is isolated from the effects of other concurrent transactions, balancing concurrency against consistency.

**Technical Definition:** The SQL standard defines four isolation levels: `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`, and `SERIALIZABLE`. Isolation means that a transaction is isolated from other concurrent processes. Different isolation levels permit different phenomena: dirty reads (reading uncommitted data), non-repeatable reads (reading the same row twice and getting different values), and phantom reads (reading a range of rows twice and getting different rows).

**Beginner-Friendly Explanation:** Isolation levels control how much your transaction can "see" other transactions that are running at the same time. Higher isolation means more consistency but less concurrency. Lower isolation means more concurrency but a higher risk of seeing inconsistent data.

### Purposes

- To balance performance (concurrency) against consistency (isolation).
- To control which phenomena (dirty reads, non-repeatable reads, phantoms) are permitted.
- To prevent data corruption from concurrent access.
- To satisfy the Isolation requirement of ACID.
- To allow applications to choose the appropriate trade-off for their use case.

### Syntax Rules and Structure

**Isolation Levels:**

| Isolation Level | Dirty Reads | Non-Repeatable Reads | Phantom Reads |
|-----------------|-------------|----------------------|---------------|
| READ UNCOMMITTED | Possible | Possible | Possible |
| READ COMMITTED | Impossible | Possible | Possible |
| REPEATABLE READ | Impossible | Impossible | Possible |
| SERIALIZABLE | Impossible | Impossible | Impossible |

**Syntax Rules (Setting Isolation Level):**

```sql
-- Standard SQL
SET TRANSACTION ISOLATION LEVEL isolation_level;

-- SQL Server
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- PostgreSQL
BEGIN;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

**Component Breakdown:**

| Isolation Level | Description |
|-----------------|-------------|
| `READ UNCOMMITTED` | Allows reading uncommitted data from other transactions |
| `READ COMMITTED` | Only reads committed data; default in many systems |
| `REPEATABLE READ` | Prevents non-repeatable reads; default in MySQL InnoDB |
| `SERIALIZABLE` | Highest isolation; transactions appear to run sequentially |

**Syntax Rules:**

- Isolation levels are set per transaction, not per statement.
- `SET TRANSACTION ISOLATION LEVEL` must be issued before any DML statements in the transaction.
- The default isolation level varies by RDBMS: READ COMMITTED in PostgreSQL, Oracle, and SQL Server; REPEATABLE READ in MySQL InnoDB.
- Lower isolation levels improve concurrency but increase the risk of data anomalies.

**Constraints and Limitations:**

- `SERIALIZABLE` can significantly reduce concurrency and increase deadlocks.
- `READ UNCOMMITTED` can cause dirty reads, leading to incorrect results.
- Isolation level semantics vary across RDBMSs; "REPEATABLE READ" means something different in each.
- Applications should choose the lowest isolation level that meets their consistency requirements.

### Annotated Complete Code Examples

**Example 1: Setting Isolation Level (SQL Server)**

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

BEGIN TRANSACTION;

SELECT * FROM accounts WHERE account_id = 1;
-- Other transactions cannot modify this row until COMMIT

COMMIT;
```

**Why this works:** `SERIALIZABLE` isolation prevents other transactions from modifying the selected row until the transaction completes, ensuring maximum consistency.

**Example 2: Setting Isolation Level (PostgreSQL)**

```sql
BEGIN;

SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

SELECT * FROM accounts WHERE account_id = 1;

COMMIT;
```

**Why this works:** `READ COMMITTED` is the default in PostgreSQL. Setting it explicitly ensures that the transaction only sees committed data.

### Real-World Cases

**Case 1: Financial Reporting**

A financial report uses `SERIALIZABLE` isolation to ensure that the numbers are consistent and reflect a single point in time.

**Case 2: High-Concurrency Web Application**

A web application uses `READ COMMITTED` isolation to allow high concurrency while preventing dirty reads.

### References

- Microsoft Learn — SQL Server: Concurrency Control Models, ACID Properties and Transaction Isolation Levels - https://learn.microsoft.com/en-us/archive/technet-wiki/51484.sql-server-concurrency-control-models-acid-properties-and-transaction-isolation-levels
- Martin Kleppmann — Hermitage: Testing the "I" in ACID - https://martin.kleppmann.com/2014/11/25/hermitage-testing-the-i-in-acid.html
- AWS — Transactions for T-SQL (Isolation Levels) - https://docs.aws.amazon.com/dms/latest/sql-server-to-aurora-postgresql-migration-playbook/chap-sql-server-aurora-pg.tsql.transactions.html

---

## Summary Table of Transaction Fundamentals

| Concept | Purpose | Key Limitation |
|---------|---------|----------------|
| Transactional Unit | All-or-nothing boundary | DDL causes implicit commits in MySQL/Oracle |
| BEGIN / START TRANSACTION | Initiate transaction context | Oracle uses implicit start |
| COMMIT | Permanently save changes | Cannot be undone |
| ROLLBACK | Discard all changes | Cannot undo committed changes |
| SAVEPOINT | Partial rollback marker | Does not free locks on rollback |
| Implicit vs. Explicit | Control transaction boundaries | Driver defaults vary |
| Isolation Levels | Balance concurrency and consistency | Semantics vary across RDBMSs |

---

## Dialect Comparison: Transaction Syntax

| Feature | PostgreSQL | MySQL | SQL Server | Oracle |
|---------|-----------|-------|------------|--------|
| Start Transaction | `BEGIN;` | `START TRANSACTION;` | `BEGIN TRANSACTION;` | Implicit (first DML) |
| Commit | `COMMIT;` | `COMMIT;` | `COMMIT;` | `COMMIT;` |
| Rollback | `ROLLBACK;` | `ROLLBACK;` | `ROLLBACK;` | `ROLLBACK;` |
| Savepoint | `SAVEPOINT name;` | `SAVEPOINT name;` | `SAVE TRANSACTION name;` | `SAVEPOINT name;` |
| Partial Rollback | `ROLLBACK TO SAVEPOINT name;` | `ROLLBACK TO SAVEPOINT name;` | `ROLLBACK TRANSACTION name;` | `ROLLBACK TO SAVEPOINT name;` |
| Release Savepoint | `RELEASE SAVEPOINT name;` | `RELEASE SAVEPOINT name;` | Automatic | `RELEASE SAVEPOINT name;` |
| Default Autocommit | Client-side | ON | ON | OFF (SQL*Plus) |
| DDL Implicit Commit | No | Yes | No | Yes |

---

## References

- AWS — Transactions for T-SQL - https://docs.aws.amazon.com/dms/latest/sql-server-to-aurora-postgresql-migration-playbook/chap-sql-server-aurora-pg.tsql.transactions.html
- DigitalOcean — SQL Commit And Rollback - https://www.digitalocean.com/community/tutorials/sql-commit-sql-rollback
- DigitalOcean — COMMIT and ROLLBACK Syntax Across Database Platforms - https://www.digitalocean.com/rss/community/tags/sql.atom
- O'Reilly — SQL in a Nutshell (ROLLBACK, COMMIT) - https://www.oreilly.com/library/view/sql-in-a/1565927443/re40.html
- Microsoft Learn — SQL Server: Concurrency Control Models, ACID Properties and Transaction Isolation Levels - https://learn.microsoft.com/en-us/archive/technet-wiki/51484.sql-server-concurrency-control-models-acid-properties-and-transaction-isolation-levels
- Microsoft Learn — Efficient Transactions with Code - https://learn.microsoft.com/id-id/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide
- Oracle — ODP.NET AutoCommit Property - https://docs.oracle.com/en/database/oracle/oracle-database/26/odpnt/ConnectionIsInAutoCommit.html
- Pub.dev — SavepointDialect enum (SQL-92 and SQL Server syntax) - https://pub.dev/documentation/odbc_fast/4.5.0/domain_entities_savepoint_dialect/SavepointDialect.html
- Martin Kleppmann — Hermitage: Testing the "I" in ACID - https://martin.kleppmann.com/2014/11/25/hermitage-testing-the-i-in-acid.html
- ISO/IEC 9075-2 — SQL Standard (Transactions) - https://jtc1sc32.org/doc/N0251-0300/32N0254.pdf