# SQL Concurrency Phenomena & Anomalies: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Concurrency phenomena and anomalies are the various ways in which concurrent database transactions can interfere with each other, producing results that would be impossible if the transactions had run one after another in isolation.

**Technical Definition:** The SQL standard (ISO/IEC 9075-2) defines isolation levels in terms of "phenomena" — interactions between concurrent transactions that must not occur at each level. These phenomena include dirty read, non-repeatable read, phantom read, and serialization anomaly. The standard also defines additional anomalies such as dirty write, lost update, read skew, and write skew. The isolation level determines which of these anomalies are physically blocked and which are permitted .

**Beginner-Friendly Explanation:** When multiple people or programs use a database at the same time, they can accidentally interfere with each other. A concurrency anomaly is what happens when that interference causes someone to see wrong data or make a decision based on outdated information. Understanding these anomalies helps you choose the right isolation level to prevent the ones that matter for your application.

### Key Characteristics

- **Interleaving-dependent:** Anomalies arise from the specific timing of overlapping read and write operations across transactions.
- **Phenomena-based definition:** The SQL standard defines isolation levels by which anomalies they permit or prevent.
- **Implementation-dependent:** Actual behaviour varies significantly across RDBMSs; PostgreSQL's `REPEATABLE READ` prevents phantom reads, while the SQL standard does not require this .
- **Trade-off with concurrency:** Preventing more anomalies generally reduces concurrency and increases blocking or aborts.
- **Detectable or preventable:** Some anomalies are prevented by locking; others (like write skew) are only detected and cause transaction aborts under `SERIALIZABLE` .

### Prerequisites

- **Basic SQL knowledge:** Familiarity with `SELECT`, `INSERT`, `UPDATE`, `DELETE`.
- **Transaction fundamentals:** Understanding of `BEGIN`, `COMMIT`, `ROLLBACK`, and ACID properties.
- **Isolation level awareness:** Knowledge of `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`.
- **Concurrency concepts:** Awareness of how multiple sessions access the same data.

### Related Programming Areas

- **Database Administration:** Tuning isolation levels to balance performance and consistency.
- **Application Development:** Choosing isolation levels based on business requirements.
- **Financial Systems:** Preventing lost updates and write skew in banking applications.
- **Data Warehousing:** Managing read skew and phantom reads in reporting workloads.
- **Distributed Systems:** Understanding how anomalies manifest across nodes.

### Core Concepts / Features

1. Dirty Reads
2. Non-Repeatable Reads / Fuzzy Reads
3. Phantom Reads
4. Lost Updates
5. Write Conflicts / First-Committer-Wins Rule
6. Read Skew
7. Write Skew

---

## 1. Dirty Reads (Reading Uncommitted Data)

### Definitions

**Core Definition:** A dirty read occurs when one transaction reads data that has been modified by another transaction but not yet committed, and that other transaction subsequently rolls back.

**Technical Definition:** In the ANSI SQL standard, a dirty read (also called a "dirty read" or "uncommitted dependency") occurs when transaction T1 modifies a row, and transaction T2 reads that modified row before T1 commits or rolls back. If T1 then rolls back, T2 has read a value that never officially existed. Dirty reads are permitted at the `READ UNCOMMITTED` isolation level and prevented at `READ COMMITTED` and above .

**Beginner-Friendly Explanation:** A dirty read is like reading a note someone wrote in pencil, then watching them erase it. You saw the note, but it was never final. In database terms, you read a value that another transaction was still deciding whether to keep.

### Purposes (Why It Matters)

- To understand why `READ UNCOMMITTED` is dangerous for any application requiring accurate data.
- To recognise when approximate or "good enough" reads are acceptable (e.g., monitoring dashboards).
- To diagnose data inconsistencies caused by reading in-flight modifications.
- To justify the use of `READ COMMITTED` as a minimum for most applications.

### Syntax Rules and Structure

**Demonstrating Dirty Reads:**

```sql
-- Session 1: Begin transaction and modify data (no COMMIT)
BEGIN;
UPDATE accounts SET balance = balance - 500 WHERE account_id = 1;

-- Session 2: Set READ UNCOMMITTED and read
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT balance FROM accounts WHERE account_id = 1;
-- Session 2 sees the uncommitted balance

-- Session 1: Roll back
ROLLBACK;
-- Session 2's read was "dirty" — the value never persisted
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| Session 1 | Modifies data without committing |
| Session 2 | Reads the uncommitted modification |
| `ROLLBACK` | Undoes Session 1's change, invalidating Session 2's read |

**Syntax Rules:**

- Dirty reads are only possible at `READ UNCOMMITTED`.
- In PostgreSQL, `READ UNCOMMITTED` behaves like `READ COMMITTED` due to MVCC; dirty reads are not possible .
- In SQL Server, `READ UNCOMMITTED` uses no shared locks, allowing dirty reads .
- In MySQL InnoDB, `READ UNCOMMITTED` allows dirty reads .

**Constraints and Limitations:**

- Dirty reads can lead to incorrect business decisions based on data that is later rolled back.
- Some applications (e.g., approximate dashboards) may tolerate dirty reads for performance.
- Dirty reads are never acceptable for financial or transactional integrity.

### Annotated Complete Code Examples

**Example 1: Dirty Read Demonstration (SQL Server)**

```sql
-- Session 1: Begin transaction and update without committing
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 500 WHERE account_id = 1;
-- No COMMIT

-- Session 2: READ UNCOMMITTED reads the uncommitted value
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT balance FROM accounts WHERE account_id = 1;
-- Output: 500.00 (uncommitted)

-- Session 1: Rollback
ROLLBACK;
-- Session 2 read a "dirty" value that never persisted
```

**Why this output occurs:** Session 2 reads the uncommitted modification because `READ UNCOMMITTED` does not acquire shared locks and does not wait for Session 1 to commit. When Session 1 rolls back, Session 2's read is invalid.

### Real-World Cases

**Case 1: Approximate Analytics**

A dashboard showing "approximate active users" uses `READ UNCOMMITTED` to avoid blocking writes, accepting that the count may include users whose sessions are later rolled back.

**Case 2: Monitoring Queries**

A DBA uses `READ UNCOMMITTED` to inspect current activity without waiting for long-running write transactions to complete.

### References

- PostgreSQL Documentation — Transaction Isolation (Table 13.1) - https://www.postgresql.org/docs/16/transaction-iso.html 
- Microsoft Learn — Transaction Locking and Row Versioning Guide - https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide 
- MySQL 8.0 Reference Manual — Transaction Isolation Levels - https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html 
- Oracle Database Concepts — Data Concurrency and Consistency - https://docs.oracle.com/cd/B14117_01/server.101/b10743/consist.htm 

---

## 2. Non-Repeatable Reads / Fuzzy Reads

### Definitions

**Core Definition:** A non-repeatable read (also called a fuzzy read) occurs when a transaction reads the same row twice and gets different values because another transaction committed a modification between the two reads.

**Technical Definition:** A non-repeatable read occurs when transaction T1 reads a row, transaction T2 modifies and commits that row, and T1 reads the same row again and sees the new (changed) value. This means T1's view of the row is not "repeatable" within its own transaction. Non-repeatable reads are permitted at `READ UNCOMMITTED` and `READ COMMITTED`, and prevented at `REPEATABLE READ` and above .

**Beginner-Friendly Explanation:** A non-repeatable read is like reading a document, then someone changes it while you're still working, and you read it again and see different content. Within the same transaction, you expected consistency but got a changing view.

### Purposes (Why It Matters)

- To understand why `READ COMMITTED` does not guarantee stable reads within a transaction.
- To recognise when applications perform multiple reads of the same row and assume consistency.
- To justify using `REPEATABLE READ` for transactions that need stable views.
- To diagnose bugs caused by values changing mid-transaction.

### Syntax Rules and Structure

**Demonstrating Non-Repeatable Reads:**

```sql
-- Session 1: Begin transaction and read a row
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT balance FROM accounts WHERE account_id = 1;
-- Output: 1000.00

-- Session 2: Update and commit
UPDATE accounts SET balance = 900 WHERE account_id = 1;
COMMIT;

-- Session 1: Read again (same transaction)
SELECT balance FROM accounts WHERE account_id = 1;
-- Output: 900.00 — Non-repeatable read!

COMMIT;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| Session 1 first read | Sees the original value |
| Session 2 commit | Changes the value |
| Session 1 second read | Sees the new value within the same transaction |

**Syntax Rules:**

- Non-repeatable reads occur at `READ COMMITTED` because each statement takes a fresh snapshot .
- At `REPEATABLE READ`, the transaction sees a snapshot from the first read, preventing non-repeatable reads .
- At `SERIALIZABLE`, non-repeatable reads are also prevented .

**Constraints and Limitations:**

- Non-repeatable reads can cause inconsistent calculations if a transaction reads the same value multiple times.
- They are less severe than dirty reads because only committed data is read.
- Applications that read once and then decide are less affected than those that read repeatedly.

### Annotated Complete Code Examples

**Example 1: Non-Repeatable Read in PostgreSQL**

```sql
-- Session 1: READ COMMITTED transaction
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT balance FROM accounts WHERE account_id = 1;
-- Output: 1000.00

-- Session 2: Update and commit
UPDATE accounts SET balance = 900 WHERE account_id = 1;
COMMIT;

-- Session 1: Read again
SELECT balance FROM accounts WHERE account_id = 1;
-- Output: 900.00 — Non-repeatable read

COMMIT;
```

**Why this output occurs:** At `READ COMMITTED`, each `SELECT` takes a fresh snapshot. Session 1's second read sees Session 2's committed change, producing a different value within the same transaction.

### Real-World Cases

**Case 1: Report Generation**

A report queries a balance, performs calculations, then queries the balance again. If the balance changed between reads, the report is inconsistent. `REPEATABLE READ` prevents this.

**Case 2: Inventory Check**

An inventory system reads a stock count, checks a business rule, then reads the count again. A non-repeatable read could cause the system to make an incorrect decision.

### References

- PostgreSQL Documentation — Transaction Isolation (Repeatable Read) - https://www.postgresql.org/docs/16/transaction-iso.html 
- MySQL 8.0 Reference Manual — Transaction Isolation Levels - https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html 
- Microsoft Learn — Transaction Isolation Levels Training - https://learn.microsoft.com/en-us/training/modules/optimize-database-performance/03-preserve-data-integrity-isolation-levels 

---

## 3. Phantom Reads (Rows Appear or Disappear Mid-Transaction)

### Definitions

**Core Definition:** A phantom read occurs when a transaction re-executes a query and sees a different set of rows because another transaction inserted, updated, or deleted rows that match the query's search condition.

**Technical Definition:** A phantom read occurs when transaction T1 executes a query that returns a set of rows, transaction T2 inserts (or deletes) rows that satisfy T1's query condition and commits, and T1 re-executes the same query and sees the new (or missing) rows. The "phantom" rows are those that appear or disappear between executions. Phantom reads are permitted at `READ UNCOMMITTED`, `READ COMMITTED`, and (according to the SQL standard) `REPEATABLE READ`, and prevented at `SERIALIZABLE` .

**Beginner-Friendly Explanation:** A phantom read is like counting the people in a room, then someone walks in while you're not looking, and you count again and get a different number. The new person is a "phantom" — they appeared between your two counts.

### Purposes (Why It Matters)

- To understand why `REPEATABLE READ` does not guarantee a stable set of rows across queries.
- To recognise when range queries (e.g., `WHERE date BETWEEN ...`) are vulnerable to phantom reads.
- To justify using `SERIALIZABLE` for operations requiring absolute consistency across multiple queries.
- To diagnose bugs caused by new rows appearing mid-transaction.

### Syntax Rules and Structure

**Demonstrating Phantom Reads:**

```sql
-- Session 1: Begin transaction and count rows
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT COUNT(*) FROM orders WHERE order_date = '2024-01-15';
-- Output: 10

-- Session 2: Insert a new order and commit
INSERT INTO orders (order_date, amount) VALUES ('2024-01-15', 100.00);
COMMIT;

-- Session 1: Count again (same transaction)
SELECT COUNT(*) FROM orders WHERE order_date = '2024-01-15';
-- Output: 11 — Phantom read! (If the implementation permits it)

COMMIT;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| Session 1 first count | Sees 10 rows |
| Session 2 insert | Adds a new matching row |
| Session 1 second count | Sees 11 rows (phantom) |

**Syntax Rules:**

- Phantom reads are prevented by `SERIALIZABLE` through range locks (SQL Server) or predicate locks (PostgreSQL) .
- PostgreSQL's `REPEATABLE READ` (Snapshot Isolation) also prevents phantom reads because it uses a transaction-level snapshot .
- MySQL InnoDB's `REPEATABLE READ` uses gap locks to prevent phantom reads in many cases .

**Constraints and Limitations:**

- Phantom reads are particularly problematic for aggregate queries (COUNT, SUM) that must be consistent.
- Not all `REPEATABLE READ` implementations prevent phantom reads; the SQL standard permits them at this level.
- `SERIALIZABLE` prevents phantom reads but at significant concurrency cost.

### Annotated Complete Code Examples

**Example 1: Phantom Read in SQL Server REPEATABLE READ**

```sql
-- Session 1: REPEATABLE READ transaction
BEGIN TRANSACTION;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT COUNT(*) FROM orders WHERE order_date = '2024-01-15';
-- Output: 10

-- Session 2: Insert and commit
INSERT INTO orders (order_date, amount) VALUES ('2024-01-15', 100.00);
COMMIT;

-- Session 1: Count again
SELECT COUNT(*) FROM orders WHERE order_date = '2024-01-15';
-- Output: 11 — Phantom read! (SQL Server REPEATABLE READ does not prevent phantoms)

COMMIT;
```

**Why this output occurs:** SQL Server's `REPEATABLE READ` acquires shared locks on rows read, but does not acquire range locks. Session 2 can insert a new row that matches the query condition, and Session 1 sees it on re-execution .

### Real-World Cases

**Case 1: Financial Reconciliation**

A reconciliation process counts transactions in a period, then re-counts to verify. A phantom read could cause the verification to fail or produce incorrect totals.

**Case 2: Inventory Reservation**

An inventory system checks stock levels, then re-checks before committing. A phantom read (new stock arriving) could cause the system to make an incorrect reservation decision.

### References

- PostgreSQL Documentation — Transaction Isolation (Phantom Reads) - https://www.postgresql.org/docs/16/transaction-iso.html 
- Microsoft Learn — Transaction Locking and Row Versioning Guide - https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide 
- MySQL 8.0 Reference Manual — Transaction Isolation Levels - https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html 

---

## 4. Lost Updates (Blind Overwrites of Concurrent Changes)

### Definitions

**Core Definition:** A lost update occurs when two transactions read the same row, both modify it, and one transaction's modification overwrites the other's, effectively discarding one of the updates.

**Technical Definition:** A lost update (also called a "lost update" or "dirty write" variant) occurs when transaction T1 reads a value, transaction T2 reads the same value, T1 writes a new value based on its read, T2 writes a new value based on its (now stale) read, and T2's write overwrites T1's. The result is that T1's update is lost. Lost updates are prevented by `REPEATABLE READ` in most MVCC implementations (through write-write conflict detection) and by `SERIALIZABLE` .

**Beginner-Friendly Explanation:** A lost update is like two people editing the same spreadsheet cell. Both see "10", both change it to "15" and "20" respectively, and the last save wins. The first person's edit is lost.

### Purposes (Why It Matters)

- To understand why concurrent updates to the same row require isolation or explicit locking.
- To recognise the most common concurrency bug in application code (read-modify-write races).
- To justify using `REPEATABLE READ` or `SELECT FOR UPDATE` for update operations.
- To implement optimistic concurrency control (version numbers) as an alternative to isolation levels.

### Syntax Rules and Structure

**Demonstrating Lost Updates:**

```sql
-- Setup
CREATE TABLE counter (id INT PRIMARY KEY, value INT);
INSERT INTO counter VALUES (1, 10);

-- Session 1: Read and modify (no lock)
BEGIN;
SELECT value FROM counter WHERE id = 1;  -- Reads 10
-- Application calculates: 10 + 5 = 15

-- Session 2: Read and modify (no lock)
BEGIN;
SELECT value FROM counter WHERE id = 1;  -- Reads 10
-- Application calculates: 10 + 3 = 13

-- Session 1: Update
UPDATE counter SET value = 15 WHERE id = 1;
COMMIT;

-- Session 2: Update (overwrites Session 1's change)
UPDATE counter SET value = 13 WHERE id = 1;
COMMIT;

-- Final value: 13 (Session 1's update is lost)
SELECT value FROM counter WHERE id = 1;
-- Output: 13
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| Both sessions read | See the same initial value (10) |
| Both sessions calculate | Based on stale reads |
| Second writer | Overwrites the first writer's update |

**Syntax Rules:**

- Lost updates are prevented by `REPEATABLE READ` in PostgreSQL, which detects write-write conflicts and aborts one transaction .
- MySQL InnoDB's `REPEATABLE READ` also prevents lost updates through locking .
- `SERIALIZABLE` prevents lost updates.
- `SELECT FOR UPDATE` prevents lost updates by acquiring exclusive locks.

**Constraints and Limitations:**

- Lost updates are the most common concurrency bug in application code.
- They can be prevented by isolation levels, explicit locks, or optimistic concurrency control.
- Not all `REPEATABLE READ` implementations prevent lost updates; the SQL standard does not require it.

### Annotated Complete Code Examples

**Example 1: Lost Update Under READ COMMITTED (PostgreSQL)**

```sql
-- Session 1: READ COMMITTED
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT value FROM counter WHERE id = 1;  -- Reads 10
-- (Application logic)

-- Session 2: READ COMMITTED
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT value FROM counter WHERE id = 1;  -- Reads 10
UPDATE counter SET value = 13 WHERE id = 1;
COMMIT;

-- Session 1: Update
UPDATE counter SET value = 15 WHERE id = 1;
COMMIT;
-- Final value: 15 (Session 2's update is lost)
```

**Why this output occurs:** At `READ COMMITTED`, Session 1's `UPDATE` overwrites Session 2's committed change because Session 1 read the old value (10) and calculated 15, not knowing about Session 2's update.

### Real-World Cases

**Case 1: Counter Increment**

A website counter increments a view count. Without isolation or locking, concurrent increments can be lost, undercounting views.

**Case 2: Balance Transfer**

Two concurrent transfers from the same account can both read the same balance and both proceed, potentially overdrawing the account.

### References

- VLDB — First Committer Wins Rule - https://www.vldb.org/pvldb/vol15/p2402-nam.pdf 
- PostgreSQL Documentation — Transaction Isolation (Repeatable Read) - https://www.postgresql.org/docs/16/transaction-iso.html 
- Microsoft Learn — Transaction Isolation Levels Training - https://learn.microsoft.com/en-us/training/modules/optimize-database-performance/03-preserve-data-integrity-isolation-levels 

---

## 5. Write Conflicts / First-Committer-Wins Rule

### Definitions

**Core Definition:** The First-Committer-Wins (FCW) rule is a concurrency control policy where, if two concurrent transactions attempt to modify the same data, the transaction that commits first wins, and the other transaction is aborted or must retry.

**Technical Definition:** To resolve write-write conflicts, various database systems including Oracle, MySQL, and PostgreSQL have implemented First Committer Wins (FCW) or its variant First Updater Wins (FUW) rule. If a write transaction makes an update that conflicts with another concurrent write transaction, FCW and FUW commit the first committer and first writer, respectively, and the other transactions are aborted . In snapshot isolation, FCW is applied at commit time: if a transaction's write set conflicts with any transaction that committed after its snapshot was taken, it is aborted .

**Beginner-Friendly Explanation:** The First-Committer-Wins rule is like a race where the first person to cross the finish line wins. If two transactions try to change the same data, the one that commits first is kept, and the other one has to try again (or is aborted).

### Purposes (Why It Matters)

- To understand how databases resolve concurrent write conflicts without deadlocks.
- To design applications that handle serialization failures and retry logic.
- To choose between pessimistic locking (wait) and optimistic concurrency (abort/retry).
- To diagnose "could not serialize access" errors.

### Syntax Rules and Structure

**FCW in Action (PostgreSQL REPEATABLE READ):**

```sql
-- Session 1: Begin REPEATABLE READ
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;

-- Session 2: Begin REPEATABLE READ and attempt to update the same row
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
UPDATE accounts SET balance = balance - 200 WHERE account_id = 1;
-- Session 2 blocks waiting for Session 1

-- Session 1: Commit (wins)
COMMIT;

-- Session 2: Unblocked, but detects conflict
-- ERROR: could not serialize access due to concurrent update
-- Session 2 must ROLLBACK and retry
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| Session 1 | First updater; commits successfully |
| Session 2 | Second updater; blocked, then aborted due to conflict |
| `ROLLBACK` / retry | Session 2 must retry with fresh snapshot |

**Syntax Rules:**

- FCW is used in snapshot isolation (PostgreSQL REPEATABLE READ, SQL Server SNAPSHOT).
- The first committer's changes are preserved; the second transaction is aborted .
- Applications must be prepared to retry transactions that fail with serialization errors .
- Read-only transactions never have serialization conflicts .

**Constraints and Limitations:**

- FCW requires application-level retry logic.
- It can cause transaction aborts under high contention.
- It differs from First-Updater-Wins (FUW), which grants priority to the first writer even before commit.

### Annotated Complete Code Examples

**Example 1: FCW in PostgreSQL REPEATABLE READ**

```sql
-- Session 1
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;

-- Session 2
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
UPDATE accounts SET balance = balance - 200 WHERE account_id = 1;
-- Blocks waiting for Session 1

-- Session 1
COMMIT;  -- First committer wins

-- Session 2
-- ERROR: could not serialize access due to concurrent update
ROLLBACK;
-- Retry with a fresh transaction
```

**Why this output occurs:** PostgreSQL's `REPEATABLE READ` uses snapshot isolation with FCW. Session 2's update targets a row modified after its snapshot. When Session 1 commits first, Session 2 is aborted with a serialization error and must retry .

### Real-World Cases

**Case 1: Inventory Reservation**

Two customers try to buy the last item. FCW ensures one transaction succeeds and the other is aborted, preventing overselling.

**Case 2: Banking Transfer**

Two concurrent transfers from the same account. FCW aborts one transfer, requiring the application to retry or inform the user.

### References

- VLDB — First Committer Wins Rule - https://www.vldb.org/pvldb/vol15/p2402-nam.pdf 
- PostgreSQL Documentation — Transaction Isolation (Serialization Failures) - https://www.postgresql.org/docs/16/transaction-iso.html 
- Docs.rs — validate_first_committer_wins - https://docs.rs/fsqlite-mvcc/latest/fsqlite_mvcc/begin_concurrent/fn.validate_first_committer_wins.html 

---

## 6. Read Skew: Interleaved Reads Yielding Inconsistent States Across Tables

### Definitions

**Core Definition:** Read skew occurs when a transaction reads data from multiple tables or rows at different points in time, and the data is inconsistent because another transaction committed changes between the reads.

**Technical Definition:** Read skew (also called "read skew" or "inconsistent analysis") occurs when a transaction reads a set of related data, but the reads are not atomic with respect to other transactions. For example, T1 reads row A (value 100), T2 updates A and B and commits, T1 reads row B (value 200 based on the new state), and T1's view is inconsistent because A reflects the old state and B reflects the new state. Read skew is prevented by `REPEATABLE READ` (snapshot isolation) and above .

**Beginner-Friendly Explanation:** Read skew is like reading a news article that references two facts, but someone updates one fact while you're reading. You end up with an article that mixes old and new information, which doesn't make sense as a whole.

### Purposes (Why It Matters)

- To understand why multi-table reads can be inconsistent even without phantom reads.
- To recognise when applications read related data across multiple queries.
- To justify using `REPEATABLE READ` for transactions that read multiple related rows.
- To diagnose inconsistencies in reports that read from multiple tables.

### Syntax Rules and Structure

**Demonstrating Read Skew:**

```sql
-- Setup: Two related tables
CREATE TABLE accounts (id INT PRIMARY KEY, balance INT);
INSERT INTO accounts VALUES (1, 100), (2, 200);

-- Session 1: Read balance of account 1
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT balance FROM accounts WHERE id = 1;
-- Output: 100

-- Session 2: Transfer 50 from account 1 to account 2 and commit
UPDATE accounts SET balance = balance - 50 WHERE id = 1;
UPDATE accounts SET balance = balance + 50 WHERE id = 2;
COMMIT;

-- Session 1: Read balance of account 2
SELECT balance FROM accounts WHERE id = 2;
-- Output: 250 (new value)

-- Session 1 has an inconsistent view: account 1 = 100 (old), account 2 = 250 (new)
-- Total should be 300, but Session 1 sees 350
COMMIT;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| Session 1 first read | Sees old value for account 1 |
| Session 2 commit | Changes both accounts atomically |
| Session 1 second read | Sees new value for account 2 |
| Result | Inconsistent snapshot across tables |

**Syntax Rules:**

- Read skew occurs at `READ COMMITTED` because each statement takes a fresh snapshot .
- `REPEATABLE READ` prevents read skew by providing a transaction-level snapshot .
- `SERIALIZABLE` also prevents read skew.
- Oracle's `READ COMMITTED` provides statement-level consistency but not transaction-level consistency .

**Constraints and Limitations:**

- Read skew is subtle because each individual read is valid, but the combination is inconsistent.
- It is particularly problematic for financial calculations that span multiple accounts.
- Applications that read related data across multiple queries are vulnerable.

### Annotated Complete Code Examples

**Example 1: Read Skew in PostgreSQL READ COMMITTED**

```sql
-- Session 1: READ COMMITTED
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT balance FROM accounts WHERE id = 1;
-- Output: 100

-- Session 2: Transfer and commit
UPDATE accounts SET balance = balance - 50 WHERE id = 1;
UPDATE accounts SET balance = balance + 50 WHERE id = 2;
COMMIT;

-- Session 1: Read account 2
SELECT balance FROM accounts WHERE id = 2;
-- Output: 250

-- Session 1's view: account 1 = 100, account 2 = 250
-- Total = 350 (inconsistent; should be 300)
COMMIT;
```

**Why this output occurs:** At `READ COMMITTED`, each `SELECT` takes a fresh snapshot. Session 1's first read sees the old balance of account 1, while its second read sees the new balance of account 2. The result is an inconsistent view of the two accounts.

### Real-World Cases

**Case 1: Financial Reporting**

A report calculates total assets by reading multiple accounts. Read skew could cause the total to be incorrect if transfers occur during the report.

**Case 2: Inventory Valuation**

An inventory system calculates total value by reading multiple product tables. Read skew could cause double-counting or missing inventory.

### References

- PostgreSQL Documentation — Transaction Isolation (Repeatable Read) - https://www.postgresql.org/docs/16/transaction-iso.html 
- Oracle Database Concepts — Data Concurrency and Consistency - https://docs.oracle.com/cd/B14117_01/server.101/b10743/consist.htm 
- MySQL 8.0 Reference Manual — Transaction Isolation Levels - https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html 

---

## 7. Write Skew: Concurrent Transactions Evaluating Overlapping Premises and Breaking Constraints

### Definitions

**Core Definition:** Write skew is a concurrency anomaly where two transactions read overlapping data, make disjoint updates based on their reads, and both commit, violating a constraint that neither transaction violated individually.

**Technical Definition:** Write skew (also called "write skew" or "write skew anomaly") occurs when transaction T1 reads a set of rows, transaction T2 reads the same set of rows, T1 updates some rows, T2 updates different rows, and both commit. The updates are disjoint (no write-write conflict), so the database allows both to commit. However, the combined effect violates a constraint that each transaction believed was maintained. Write skew is permitted under snapshot isolation (PostgreSQL `REPEATABLE READ`, SQL Server `SNAPSHOT`) and is only prevented by `SERIALIZABLE` .

**Beginner-Friendly Explanation:** Write skew is like two doctors on call. Both check that the other is available, then both decide to go off-duty. Each thinks the other is covering, but now no one is on call. Neither doctor did anything wrong individually, but together they broke the rule.

### Purposes (Why It Matters)

- To understand the fundamental limitation of snapshot isolation.
- To recognise when applications enforce constraints through reads and writes.
- To justify using `SERIALIZABLE` for applications with cross-row constraints.
- To implement application-level checks or explicit locks when write skew is unacceptable.

### Syntax Rules and Structure

**Demonstrating Write Skew:**

```sql
-- Setup: On-call schedule
CREATE TABLE doctors (id INT PRIMARY KEY, name TEXT, on_call BOOLEAN);
INSERT INTO doctors VALUES (1, 'Alice', true), (2, 'Bob', true);

-- Constraint: At least one doctor must be on call

-- Session 1: Check if another doctor is on call, then go off call
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT COUNT(*) FROM doctors WHERE on_call = true;
-- Output: 2 (both are on call; safe to go off)
UPDATE doctors SET on_call = false WHERE id = 1;

-- Session 2: Same check, different doctor
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT COUNT(*) FROM doctors WHERE on_call = true;
-- Output: 2 (sees the same snapshot; safe to go off)
UPDATE doctors SET on_call = false WHERE id = 2;

-- Both commit
COMMIT;  -- Session 1
COMMIT;  -- Session 2

-- Result: No doctor on call (constraint violated)
SELECT COUNT(*) FROM doctors WHERE on_call = true;
-- Output: 0
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| Both sessions read | See 2 doctors on call (same snapshot) |
| Both sessions update | Disjoint rows (no write-write conflict) |
| Both commit | FCW does not detect a conflict |
| Result | Constraint violated |

**Syntax Rules:**

- Write skew is permitted under snapshot isolation (PostgreSQL `REPEATABLE READ`, SQL Server `SNAPSHOT`).
- `SERIALIZABLE` prevents write skew by detecting read/write dependencies .
- The SQL standard defines write skew as a "serialization anomaly" prevented only by `SERIALIZABLE`.
- Applications can prevent write skew with `SELECT FOR UPDATE` or explicit table locks.

**Constraints and Limitations:**

- Write skew is subtle because each transaction's individual logic is correct.
- It is not prevented by `REPEATABLE READ` in most implementations.
- `SERIALIZABLE` may abort one transaction, requiring retry.
- Not all applications need write skew prevention; only those with cross-row constraints.

### Annotated Complete Code Examples

**Example 1: Write Skew in PostgreSQL REPEATABLE READ**

```sql
-- Session 1: REPEATABLE READ
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT COUNT(*) FROM doctors WHERE on_call = true;
-- Output: 2
UPDATE doctors SET on_call = false WHERE id = 1;

-- Session 2: REPEATABLE READ
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT COUNT(*) FROM doctors WHERE on_call = true;
-- Output: 2 (same snapshot)
UPDATE doctors SET on_call = false WHERE id = 2;

-- Both commitCOMMIT;  -- Session 1
COMMIT;  -- Session 2

-- Constraint violated
SELECT COUNT(*) FROM doctors WHERE on_call = true;
-- Output: 0
```

**Why this output occurs:** Both transactions read the same snapshot (2 doctors on call) and updated disjoint rows (different doctor IDs). Since there was no write-write conflict, both committed. The combined effect violates the constraint.

### Real-World Cases

**Case 1: On-Call Scheduling**

Two doctors both check that the other is on call and both go off call, leaving no one on call.

**Case 2: Inventory Allocation**

Two orders each check that enough stock exists for their individual order, but together they exceed available stock, causing overselling.

**Case 3: Financial Constraint**

Two transactions each check that a sum remains positive, but their combined withdrawals make it negative.

### References

- PostgreSQL Documentation — Transaction Isolation (Serializable) - https://www.postgresql.org/docs/16/transaction-iso.html 
- PostgreSQL Documentation — Serializable Isolation Example - https://www.postgresql.org/docs/9.5/transaction-iso.html 
- ISO/IEC 9075-2 — SQL Standard (Serialization Anomalies) - https://jtc1sc32.org/doc/N0251-0300/32N0254.pdf

---

## Summary Table: Anomalies vs. Isolation Levels

| Anomaly | READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SNAPSHOT | SERIALIZABLE |
|---------|:----------------:|:--------------:|:---------------:|:--------:|:------------:|
| Dirty Read | Allowed | Prevented | Prevented | Prevented | Prevented |
| Non-Repeatable Read | Allowed | Allowed | Prevented | Prevented | Prevented |
| Phantom Read | Allowed | Allowed | Possible¹ | Possible¹ | Prevented |
| Lost Update | Allowed | Allowed | Prevented² | Prevented² | Prevented |
| Read Skew | Allowed | Allowed | Prevented | Prevented | Prevented |
| Write Skew | Allowed | Allowed | **Allowed** | **Allowed** | Prevented |
| Serialization Anomaly | Allowed | Allowed | Allowed | Allowed | Prevented |

¹ PostgreSQL's `REPEATABLE READ` also prevents phantom reads due to MVCC snapshot isolation.

² Lost update prevention varies by implementation; PostgreSQL and MySQL InnoDB prevent it at `REPEATABLE READ`.

---

## References

- PostgreSQL Documentation — Transaction Isolation - https://www.postgresql.org/docs/16/transaction-iso.html 
- PostgreSQL Documentation — Serializable Isolation Example - https://www.postgresql.org/docs/9.5/transaction-iso.html 
- MySQL 8.0 Reference Manual — Transaction Isolation Levels - https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html 
- MySQL 8.4 Reference Manual — SET TRANSACTION Statement - https://dev.mysql.com/doc/refman/8.4/en/set-transaction.html 
- Oracle Database Concepts — Data Concurrency and Consistency - https://docs.oracle.com/cd/B14117_01/server.101/b10743/consist.htm 
- Oracle Database Development Guide — Transaction Isolation Levels - https://docs.oracle.com/en/database/oracle/oracle-database/19/adfns/database-development-guide.pdf 
- Microsoft Learn — Transaction Locking and Row Versioning Guide - https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide 
- Microsoft Learn — Transaction Isolation Levels Training - https://learn.microsoft.com/en-us/training/modules/optimize-database-performance/03-preserve-data-integrity-isolation-levels 
- Microsoft Learn — Adjusting Transaction Isolation Levels - https://learn.microsoft.com/hi-in/previous-versions/sql/sql-server-2008-r2/ms189542(v=sql.105) 
- VLDB — First Committer Wins Rule - https://www.vldb.org/pvldb/vol15/p2402-nam.pdf 
- Docs.rs — validate_first_committer_wins - https://docs.rs/fsqlite-mvcc/latest/fsqlite_mvcc/begin_concurrent/fn.validate_first_committer_wins.html 
- Google Patents — First-Committer-Wins Rule - https://patents.google.com/patent/JP7398066B2/en 
- Dart Packages — IsolationLevel enum - https://pub.dev/documentation/entidb/latest/entidb/IsolationLevel.html 
- ISO/IEC 9075-2 — SQL Standard (Transaction Isolation) - https://jtc1sc32.org/doc/N0251-0300/32N0254.pdf