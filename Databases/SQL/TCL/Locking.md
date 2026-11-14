# Comprehensive Programming Cheat Sheet: SQL Locking

## Topic Overview

### Definitions

**Core Definition**  
SQL locking is the concurrency-control mechanism by which a relational database management system (RDBMS) manages simultaneous access to data, preventing conflicting operations from corrupting data while allowing safe concurrent reads and writes.

**Technical Definition**  
SQL locking encompasses the lock types (shared, exclusive, intent, update, schema, key-range, and vendor-specific variants), the granularity levels at which locks are acquired (row, page, table, database, and finer structures such as key ranges and index records), the explicit lock modes available in `SELECT` statements (`FOR UPDATE`, `FOR SHARE`, `FOR NO KEY UPDATE`, `FOR KEY SHARE`), and the mechanisms for escalation, timeout, deadlock detection, deadlock prevention, contention resolution, and lock monitoring. Locking behaviour is tightly coupled to the transaction isolation level and is implemented differently across PostgreSQL, SQL Server, MySQL (InnoDB), and Oracle.

**Beginner-Friendly Explanation**  
Imagine a shared document that multiple people want to edit. Locking is the set of rules that decides who can read, who can write, and who must wait, so that two people don’t overwrite each other’s changes. A shared lock is like “I’m reading, don’t change anything yet”; an exclusive lock is like “I’m editing, nobody else touch this.”

### Key Characteristics

- **Granularity matters**: Locks can be taken at row, page, table, or database level; finer granularity allows more concurrency but costs more bookkeeping.
- **Vendor-specific implementations**: PostgreSQL uses table-level lock modes with row-level flavours; SQL Server uses a multigranular locking architecture with intent locks; MySQL InnoDB uses record, gap, and next-key locks alongside intention locks.
- **Isolation-level dependence**: The locks acquired implicitly by a statement depend on the transaction isolation level.
- **Deadlock possibility**: Because locks are held until transaction end, circular wait conditions can arise and must be detected or prevented.
- **Escalation trade-off**: Some engines (notably SQL Server) escalate many fine-grained locks to a table lock to save memory, at the cost of concurrency.
- **Observability**: Every major RDBMS exposes lock metadata through system views or commands (`pg_locks`, `sys.dm_tran_locks`, `SHOW ENGINE INNODB STATUS`).

### Prerequisites

- A working knowledge of transactions (`BEGIN`, `COMMIT`, `ROLLBACK`) and ACID properties.
- Familiarity with `SELECT`, `INSERT`, `UPDATE`, and `DELETE`.
- Understanding of transaction isolation levels (Read Committed, Repeatable Read, Serializable).
- Access to a PostgreSQL, SQL Server, or MySQL instance for practicing the examples.

### Related Programming Areas

- Transaction processing and concurrency control.
- Database performance tuning and query optimisation.
- Application-level retry logic and idempotency.
- Distributed systems and distributed locking (advisory locks, application locks).
- Database administration and monitoring.

---

## Core Concepts and Features

### 1. Lock Types

#### 1.1 Shared Locks

##### Definitions

**Core Definition**  
A shared lock (S lock) allows a transaction to read a resource while preventing other transactions from modifying it.

**Technical Definition**  
A shared lock is a read lock. Multiple transactions can hold shared locks on the same resource simultaneously. A shared lock conflicts with an exclusive lock but is compatible with other shared locks. In PostgreSQL, the corresponding table-level mode is `ACCESS SHARE`; in SQL Server and MySQL InnoDB, it is the `S` lock.

**Beginner-Friendly Explanation**  
A shared lock means “I’m reading this; you can read it too, but nobody can change it while I’m reading.”

##### Purposes

- To prevent a resource from being modified while it is being read by another transaction.
- To allow multiple concurrent readers without blocking each other.
- To support consistent reads under isolation levels that require read stability.

##### Syntax Rules and Structure

Shared locks are usually acquired implicitly by `SELECT` statements (depending on isolation level) and explicitly in some vendor syntax (e.g., `SELECT … FOR SHARE` in PostgreSQL and MySQL, or `WITH (HOLDLOCK)` in SQL Server).

**General pattern (implicit)**:

```sql
SELECT columns FROM table WHERE condition;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `SELECT` | Acquires a shared lock (under the appropriate isolation level). |
| `FROM table` | The resource being read. |
| `WHERE condition` | Restricts the rows read and thus the scope of locks. |

**Syntax rules**

- In PostgreSQL, a plain `SELECT` acquires an `ACCESS SHARE` lock on the table, which conflicts only with `ACCESS EXCLUSIVE`.
- In MySQL InnoDB, a consistent nonlocking read under `READ COMMITTED` or `REPEATABLE READ` does **not** acquire shared locks; `SELECT … FOR SHARE` is needed for locking reads.
- In SQL Server, shared locks are acquired by read operations under the default `READ COMMITTED` isolation level.

**Constraints and limitations**

- Shared locks block writers but not other readers.
- Shared locks can lead to writer starvation if readers hold them for long periods.
- Under `SERIALIZABLE`, shared locks may be held until transaction end, increasing contention.

##### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: PostgreSQL – Implicit shared lock from `SELECT`

```sql
-- Step 1: Start a transaction.
BEGIN;

-- Step 2: Read a row; PostgreSQL acquires ACCESS SHARE on the table.
SELECT * FROM accounts WHERE id = 1;

-- Step 3: While this transaction is open, another session attempting
--         TRUNCATE or DROP TABLE would block because those need ACCESS EXCLUSIVE.
-- Step 4: Commit to release the shared lock.
COMMIT;
```

**Expected output** (psql):

```text
BEGIN
 id | balance
----+---------
  1 |  1000.00
(1 row)

COMMIT
```

**Why this output appears**: The `SELECT` succeeds and acquires a shared table lock. A concurrent `DROP TABLE` would wait because `ACCESS SHARE` conflicts with `ACCESS EXCLUSIVE`.

###### Example 2: MySQL InnoDB – Explicit shared lock with `FOR SHARE`

```sql
-- Step 1: Start a transaction.
START TRANSACTION;

-- Step 2: Acquire a shared lock on the selected rows.
SELECT * FROM products WHERE id = 10 FOR SHARE;

-- Step 3: Another transaction can also acquire FOR SHARE on the same row,
--         but cannot UPDATE or DELETE it until this transaction ends.
-- Step 4: Commit to release the locks.
COMMIT;
```

**Expected output**:

```text
+----+----------+-------+
| id | name     | price |
+----+----------+-------+
| 10 | Widget A | 19.99 |
+----+----------+-------+
1 row in set (0.00 sec)
```

**Why this output appears**: `FOR SHARE` acquires an `S` lock on the row and an `IS` intention lock on the table, allowing other readers but blocking writers.

##### Real-World Cases

- **Reporting queries**: A long-running report acquires shared locks to ensure the data it reads does not change underneath it.
- **Inventory checks**: A stock-count transaction reads current inventory with shared locks so that concurrent adjustments wait.

---

#### 1.2 Exclusive Locks

##### Definitions

**Core Definition**  
An exclusive lock (X lock) allows a transaction to modify a resource and prevents all other transactions from reading or modifying it.

**Technical Definition**  
An exclusive lock is the strongest lock mode. It conflicts with all other lock modes, including other exclusive locks and shared locks. In PostgreSQL, the table-level `ACCESS EXCLUSIVE` mode is the strongest; in SQL Server and MySQL InnoDB, the `X` lock is the exclusive mode.

**Beginner-Friendly Explanation**  
An exclusive lock means “I’m changing this; nobody else can read or write it until I’m done.”

##### Purposes

- To guarantee that only one transaction can modify a resource at a time.
- To prevent dirty reads and lost updates during data modification.
- To protect DDL operations (e.g., `DROP TABLE`, `TRUNCATE`) from concurrent access.

##### Syntax Rules and Structure

Exclusive locks are acquired implicitly by `INSERT`, `UPDATE`, `DELETE`, and by DDL statements such as `DROP TABLE`.

**General pattern (implicit)**:

```sql
UPDATE table SET column = value WHERE condition;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `UPDATE` / `DELETE` / `INSERT` | Acquires exclusive locks on affected rows and intent locks on higher levels. |
| `DROP TABLE` / `TRUNCATE` | Acquires the strongest table-level lock (`ACCESS EXCLUSIVE` in PostgreSQL). |

**Syntax rules**

- In PostgreSQL, `UPDATE`, `DELETE`, and `INSERT` acquire `ROW EXCLUSIVE` at the table level, plus row-level locks on affected rows.
- In SQL Server, data modification statements acquire intent exclusive (`IX`) locks at the table level and exclusive (`X`) locks on affected rows.
- In MySQL InnoDB, `UPDATE` and `DELETE` acquire `X` locks on affected index records and `IX` intention locks on the table.

**Constraints and limitations**

- Exclusive locks block all other access, so holding them for long durations severely reduces concurrency.
- In PostgreSQL, `ACCESS EXCLUSIVE` is the default lock for `LOCK TABLE` when no mode is specified.

##### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: PostgreSQL – Exclusive lock from `UPDATE`

```sql
-- Step 1: Start a transaction.
BEGIN;

-- Step 2: Update a row; PostgreSQL acquires ROW EXCLUSIVE on the table
--         and a row-level exclusive lock on the updated row.
UPDATE accounts SET balance = balance - 100 WHERE id = 1;

-- Step 3: Another transaction attempting to UPDATE the same row will block
--         until this transaction commits or rolls back.
-- Step 4: Commit.
COMMIT;
```

**Expected output**:

```text
BEGIN
UPDATE 1
COMMIT
```

**Why this output appears**: The `UPDATE` acquires an exclusive row lock; a concurrent `UPDATE` on the same row waits until the first transaction ends.

###### Example 2: SQL Server – Exclusive lock from `DELETE`

```sql
-- Step 1: Begin a transaction.
BEGIN TRANSACTION;

-- Step 2: Delete a row; SQL Server acquires an IX lock on the table
--         and an X lock on the deleted row.
DELETE FROM Orders WHERE OrderID = 1001;

-- Step 3: A concurrent SELECT under READ COMMITTED will block.
-- Step 4: Commit.
COMMIT TRANSACTION;
```

**Expected output**:

```text
(1 row affected)
```

**Why this output appears**: The `DELETE` holds exclusive locks until the transaction commits, blocking readers under locking isolation levels.

##### Real-World Cases

- **Financial transactions**: A debit/credit transaction holds exclusive locks on account rows to prevent concurrent balance changes.
- **Schema changes**: A `DROP TABLE` acquires an exclusive lock to ensure no concurrent queries access the table.

---

#### 1.3 Intent Locks

##### Definitions

**Core Definition**  
An intent lock is a table-level lock that signals a transaction’s intention to acquire a shared or exclusive lock on a lower-level resource (such as a row or page).

**Technical Definition**  
Intent locks establish a lock hierarchy. Before acquiring a shared or exclusive lock on a row or page, a transaction must first acquire an intent lock on the containing table (and page, if applicable). The intent lock prevents other transactions from acquiring an incompatible table-level lock. Intent lock variants include intent shared (`IS`), intent exclusive (`IX`), and shared with intent exclusive (`SIX`) in SQL Server, and intention shared (`IS`) and intention exclusive (`IX`) in MySQL InnoDB.

**Beginner-Friendly Explanation**  
An intent lock is like putting a sign on a filing cabinet that says “someone is working inside one of these folders.” It lets others know not to lock the whole cabinet while someone is working at the folder level.

##### Purposes

- To prevent a transaction from acquiring a conflicting table-level lock while another transaction holds row-level locks.
- To improve the efficiency of lock conflict detection at higher granularity levels.
- To support multigranularity locking without inspecting every row or page.

##### Syntax Rules and Structure

Intent locks are acquired automatically; they are not usually written explicitly.

**Component breakdown**

| Intent lock | Meaning |
|-------------|---------|
| `IS` (intent shared) | The transaction intends to set shared locks on lower-level resources. |
| `IX` (intent exclusive) | The transaction intends to set exclusive locks on lower-level resources. |
| `SIX` (shared with intent exclusive) | The transaction holds a shared lock on the table and intends to set exclusive locks on some rows. |

**Syntax rules**

- In SQL Server, a shared intent lock is requested at the table level before shared locks are requested on pages or rows.
- In MySQL InnoDB, before a transaction can acquire a shared lock on a row, it must first acquire an `IS` or stronger lock on the table; before an exclusive lock, it must acquire an `IX` lock.
- PostgreSQL uses table-level lock modes (e.g., `ROW SHARE`, `ROW EXCLUSIVE`) that serve a similar hierarchical purpose, though the terminology differs.

**Constraints and limitations**

- Intent locks do not block ordinary row-level access; they block incompatible table-level requests.
- A transaction can hold conflicting intent locks on the same table in some engines if the lower-level locks are on different resources.

##### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: SQL Server – Intent locks during an `UPDATE`

```sql
-- Step 1: Begin a transaction.
BEGIN TRANSACTION;

-- Step 2: Update a row. SQL Server acquires:
--   IX lock on the table,
--   IX lock on the page,
--   X lock on the row.
UPDATE Customers SET City = 'New York' WHERE CustomerID = 5;

-- Step 3: A concurrent LOCK TABLE ... IN EXCLUSIVE MODE would be blocked
--         because the IX lock conflicts with the table-level X request.
-- Step 4: Commit.
COMMIT TRANSACTION;
```

**Expected output**:

```text
(1 row affected)
```

**Why this output appears**: The `IX` intent lock on the table prevents another session from acquiring an exclusive table lock, while allowing other row-level operations on different rows.

###### Example 2: MySQL InnoDB – Intention locks during `SELECT … FOR UPDATE`

```sql
-- Step 1: Start a transaction.
START TRANSACTION;

-- Step 2: Acquire an exclusive row lock.
SELECT * FROM orders WHERE id = 1 FOR UPDATE;

-- Step 3: InnoDB has acquired an IX intention lock on the table
--         and an X record lock on the row.
-- Step 4: Commit.
COMMIT;
```

**Expected output**:

```text
+----+---------+--------+
| id | user_id | amount |
+----+---------+--------+
|  1 |     100 |  250.00|
+----+---------+--------+
1 row in set (0.00 sec)
```

**Why this output appears**: The `FOR UPDATE` clause sets an `IX` intention lock on the table and an exclusive lock on the selected row.

##### Real-World Cases

- **High-concurrency OLTP**: Intent locks allow many row-level operations on the same table without forcing table-level serialisation.
- **Lock escalation avoidance**: Intent locks help the engine detect when escalation to a table lock is safe or unsafe.

---

### 2. Lock Granularity

#### 2.1 Row Locks

##### Definitions

**Core Definition**  
A row lock is a lock placed on a single row (or record) of a table.

**Technical Definition**  
Row locks provide the finest level of locking granularity, allowing concurrent transactions to access different rows of the same table simultaneously. In MySQL InnoDB, row locks are implemented as record locks on index records; in SQL Server, row locks are placed on individual rows (or keys); in PostgreSQL, row-level locks are represented by row-level lock modes such as `FOR UPDATE`.

**Beginner-Friendly Explanation**  
A row lock means “I’m working on this specific row; other rows are still available to others.”

##### Purposes

- To maximise concurrency by allowing transactions to modify different rows simultaneously.
- To minimise the scope of blocking.
- To support fine-grained conflict resolution in OLTP workloads.

##### Syntax Rules and Structure

Row locks are acquired implicitly by `UPDATE`, `DELETE`, and by `SELECT … FOR UPDATE/SHARE`.

**Component breakdown**

| Statement | Row lock behaviour |
|-----------|-------------------|
| `UPDATE … WHERE id = 1` | Exclusive row lock on the matching row. |
| `DELETE … WHERE id = 1` | Exclusive row lock on the matching row. |
| `SELECT … FOR UPDATE` | Exclusive row lock on the selected rows. |
| `SELECT … FOR SHARE` | Shared row lock on the selected rows. |

**Constraints and limitations**

- Row locks consume more memory than coarser-grained locks.
- In MySQL InnoDB, row locks are always index-record locks; a query without an index may lock more rows than intended via a full scan.
- In PostgreSQL, row-level locks do not block plain `SELECT` queries; they block writers and `SELECT … FOR UPDATE/SHARE`.

##### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: MySQL InnoDB – Row lock from `UPDATE`

```sql
-- Step 1: Start a transaction.
START TRANSACTION;

-- Step 2: Update a single row by primary key.
UPDATE accounts SET balance = balance - 50 WHERE id = 7;

-- Step 3: Only the row with id = 7 is locked; other rows remain accessible.
-- Step 4: Commit.
COMMIT;
```

**Expected output**:

```text
Query OK, 1 row affected (0.00 sec)
```

**Why this output appears**: InnoDB places an `X` lock on the index record for `id = 7` and an `IX` intention lock on the table.

##### Real-World Cases

- **Order processing**: Different orders can be updated concurrently because each `UPDATE` locks only its own order row.
- **Seat reservation**: Each seat row is locked independently, allowing parallel bookings of different seats.

---

#### 2.2 Page Locks

##### Definitions

**Core Definition**  
A page lock is a lock placed on a page (a fixed-size unit of storage containing multiple rows) of a table or index.

**Technical Definition**  
Page-level locking is coarser than row-level locking. SQL Server supports page locks explicitly and may choose page-level granularity during escalation or based on query cost. InnoDB does not use page locks for ordinary row operations; its locks are record-based, though it does use page-level structures internally.

**Beginner-Friendly Explanation**  
A page lock means “I’m working on a whole page of rows at once; other pages are still free.”

##### Purposes

- To reduce the memory overhead of managing many row locks.
- To provide a middle ground between row-level concurrency and table-level simplicity.
- To support SQL Server’s dynamic lock escalation.

##### Syntax Rules and Structure

Page locks are usually chosen by the optimiser or acquired via hints.

**SQL Server example**:

```sql
SELECT * FROM Orders WITH (PAGLOCK) WHERE OrderDate = '2025-01-01';
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `PAGLOCK` | Hint requesting page-level locking. |
| `FROM Orders` | The table whose pages are locked. |

**Constraints and limitations**

- Page locks block access to all rows on the page, even if the transaction only needs one row.
- In SQL Server, page locks can escalate to table locks under memory pressure.
- MySQL InnoDB does not expose page locks to users; its record and gap locks operate on index records.

##### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: SQL Server – Page lock hint

```sql
-- Step 1: Begin a transaction.
BEGIN TRANSACTION;

-- Step 2: Request page-level locking.
SELECT * FROM Orders WITH (PAGLOCK) WHERE OrderDate = '2025-01-01';

-- Step 3: The entire page containing the matching rows is locked.
-- Step 4: Commit.
COMMIT TRANSACTION;
```

**Expected output** (illustrative):

```text
OrderID  CustomerID  OrderDate
-------  ----------  ----------
1001     5           2025-01-01
1002     7           2025-01-01
...
```

**Why this output appears**: The `PAGLOCK` hint instructs SQL Server to acquire page-level locks instead of row locks, blocking access to all rows on those pages.

##### Real-World Cases

- **Bulk updates**: A batch job updating many rows may benefit from page-level locking to reduce lock overhead.
- **Index maintenance**: Page-level locks can be appropriate when reorganising index pages.

---

#### 2.3 Table Locks

##### Definitions

**Core Definition**  
A table lock is a lock placed on an entire table, preventing concurrent access to any row in that table.

**Technical Definition**  
Table locks are the coarsest common granularity (short of database-level locks). They are used by DDL operations, by explicit `LOCK TABLE` statements, and by lock escalation. In PostgreSQL, all lock modes shown in `pg_locks` for ordinary DML are table-level modes with names that sometimes imply row-level semantics (e.g., `ROW EXCLUSIVE` is still a table-level lock). In MySQL, `LOCK TABLES … WRITE` acquires an exclusive table lock.

**Beginner-Friendly Explanation**  
A table lock means “I’m working on the whole table; nobody else can use it until I’m done.”

##### Purposes

- To protect DDL operations that must see a consistent table structure.
- To provide a simple, low-overhead locking strategy when row-level concurrency is not needed.
- To support bulk operations that affect most of a table.

##### Syntax Rules and Structure

**PostgreSQL explicit table lock**:

```sql
LOCK TABLE table_name IN lock_mode MODE;
```

**MySQL table lock**:

```sql
LOCK TABLES table_name WRITE;
-- ...
UNLOCK TABLES;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `LOCK TABLE` | Acquires a table-level lock. |
| `lock_mode` | e.g., `ACCESS EXCLUSIVE`, `SHARE`, `ROW EXCLUSIVE`. |
| `LOCK TABLES … WRITE` | MySQL statement for an exclusive table lock. |

**Constraints and limitations**

- Table locks severely reduce concurrency; they should be used sparingly.
- In PostgreSQL, `ACCESS EXCLUSIVE` blocks all other access, including `SELECT`.
- In MySQL, `LOCK TABLES` must be released with `UNLOCK TABLES` or at session end.

##### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: PostgreSQL – Explicit table lock

```sql
-- Step 1: Begin a transaction.
BEGIN;

-- Step 2: Acquire an exclusive table lock.
LOCK TABLE accounts IN ACCESS EXCLUSIVE MODE;

-- Step 3: Perform maintenance (e.g., add a column).
ALTER TABLE accounts ADD COLUMN last_updated timestamptz;

-- Step 4: Commit to release the lock.
COMMIT;
```

**Expected output**:

```text
BEGIN
LOCK TABLE
ALTER TABLE
COMMIT
```

**Why this output appears**: `ACCESS EXCLUSIVE` guarantees that no other session can read or write the table during the schema change.

##### Real-World Cases

- **Schema migrations**: DDL changes require an exclusive table lock to prevent concurrent access.
- **Bulk data loads**: A nightly batch load may lock the target table exclusively for speed.

---

#### 2.4 Database Locks

##### Definitions

**Core Definition**  
A database lock is a lock placed on an entire database, restricting access to all objects within it.

**Technical Definition**  
Database-level locks are uncommon in ordinary application workloads but exist in some engines for maintenance operations (e.g., `ALTER DATABASE … SET SINGLE_USER` in SQL Server) and for backup or restore operations. In PostgreSQL, a database-level lock can be acquired via `LOCK` on the database object, though this is rare.

**Beginner-Friendly Explanation**  
A database lock means “the whole database is off-limits while I do something important, like a backup or a major upgrade.”

##### Purposes

- To perform database-wide maintenance operations safely.
- To prevent connections during restore or upgrade.
- To enforce exclusive access for certain administrative tasks.

##### Syntax Rules and Structure

**SQL Server example**:

```sql
ALTER DATABASE SalesDB SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `ALTER DATABASE` | Changes database state. |
| `SET SINGLE_USER` | Restricts access to one connection. |
| `WITH ROLLBACK IMMEDIATE` | Rolls back active transactions and disconnects users. |

**Constraints and limitations**

- Database locks are disruptive and should be used only during maintenance windows.
- In SQL Server, `SINGLE_USER` with `ROLLBACK IMMEDIATE` forcibly terminates connections.
- In PostgreSQL, database-level locks are rarely used explicitly; `DROP DATABASE` requires no active connections.

##### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: SQL Server – Database lock for maintenance

```sql
-- Step 1: Restrict the database to a single user.
ALTER DATABASE SalesDB SET SINGLE_USER WITH ROLLBACK IMMEDIATE;

-- Step 2: Perform maintenance (e.g., restore a backup, shrink files).
-- ...

-- Step 3: Return the database to multi-user mode.
ALTER DATABASE SalesDB SET MULTI_USER;
```

**Expected output**:

```text
Commands completed successfully.
```

**Why this output appears**: `SINGLE_USER` acquires an effective database-level lock, preventing other connections while maintenance is performed.

##### Real-World Cases

- **Database upgrades**: A major version upgrade may require exclusive database access.
- **Backup and restore**: Some backup tools require a database lock to ensure consistency.

---

### 3. Lock Modes (PostgreSQL)

PostgreSQL provides explicit row-level lock modes through the `SELECT … FOR` clause. These modes are distinct from the table-level lock modes listed earlier and are used to control how selected rows are locked.

#### 3.1 `FOR UPDATE`

##### Definitions

**Core Definition**  
`FOR UPDATE` acquires an exclusive row-level lock on the rows returned by a `SELECT`, blocking other transactions from modifying or locking those rows.

**Technical Definition**  
`SELECT … FOR UPDATE` locks the selected rows as if they were being updated. It blocks other `UPDATE`, `DELETE`, `SELECT … FOR UPDATE`, `SELECT … FOR NO KEY UPDATE`, `SELECT … FOR SHARE`, and `SELECT … FOR KEY SHARE` operations on the same rows. It is the strongest of the four row-level modes.

**Beginner-Friendly Explanation**  
`FOR UPDATE` means “I’m going to change these rows, so lock them exclusively until I’m done.”

##### Purposes

- To prevent other transactions from modifying rows that the current transaction intends to update.
- To implement optimistic locking patterns in application code.
- To serialise access to specific rows in a controlled way.

##### Syntax Rules and Structure

```sql
SELECT columns
FROM table
WHERE condition
FOR UPDATE [OF table_name [, ...]] [NOWAIT | SKIP LOCKED];
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `FOR UPDATE` | Requests exclusive row-level locks. |
| `OF table_name` | Restricts the lock to the named table(s) in a join. |
| `NOWAIT` | Returns an error immediately if a lock cannot be acquired. |
| `SKIP LOCKED` | Skips rows that are already locked, returning only lockable rows. |

**Constraints and limitations**

- `FOR UPDATE` cannot be used with aggregate functions or `DISTINCT` in the same query block.
- Locks are held until the transaction ends.
- In PostgreSQL, `FOR UPDATE` on a view locks the underlying base-table rows.

##### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: PostgreSQL – `FOR UPDATE` with `NOWAIT`

```sql
-- Step 1: Start a transaction.
BEGIN;

-- Step 2: Lock a row exclusively; fail immediately if already locked.
SELECT * FROM accounts WHERE id = 1 FOR UPDATE NOWAIT;

-- Step 3: Update the row.
UPDATE accounts SET balance = balance - 100 WHERE id = 1;

-- Step 4: Commit.
COMMIT;
```

**Expected output** (successful case):

```text
 id | balance
----+---------
  1 | 1000.00
(1 row)

UPDATE 1
COMMIT
```

**Expected output** (if the row is already locked):

```text
ERROR:  could not obtain lock on row in relation "accounts"
```

**Why this output appears**: `NOWAIT` causes an immediate error instead of waiting, allowing the application to handle contention gracefully.

##### Real-World Cases

- **Seat booking**: Lock the selected seat row with `FOR UPDATE` to prevent double-booking.
- **Inventory reservation**: Lock the product row before decrementing stock.

---

#### 3.2 `FOR SHARE`

##### Definitions

**Core Definition**  
`FOR SHARE` acquires a shared row-level lock on the selected rows, allowing other transactions to read them but preventing modification.

**Technical Definition**  
`SELECT … FOR SHARE` acquires a shared lock on the selected rows. It blocks `UPDATE`, `DELETE`, `SELECT … FOR UPDATE`, and `SELECT … FOR NO KEY UPDATE` on the same rows, but allows other `SELECT … FOR SHARE` and plain `SELECT` statements.

**Beginner-Friendly Explanation**  
`FOR SHARE` means “I’m reading these rows and I want them to stay unchanged while I read, but other people can read them too.”

##### Purposes

- To ensure that rows read by a transaction are not modified by another transaction before the first transaction completes.
- To support consistent read-modify-write patterns without blocking other readers.

##### Syntax Rules and Structure

```sql
SELECT columns FROM table WHERE condition FOR SHARE [NOWAIT | SKIP LOCKED];
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `FOR SHARE` | Requests shared row-level locks. |
| `NOWAIT` / `SKIP LOCKED` | Same semantics as for `FOR UPDATE`. |

**Constraints and limitations**

- Shared locks can lead to deadlocks when two transactions each hold a shared lock and then try to upgrade to exclusive.
- In PostgreSQL, `FOR SHARE` is not compatible with `FOR UPDATE` on the same row.

##### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: PostgreSQL – `FOR SHARE` for a consistent read

```sql
-- Step 1: Start a transaction.
BEGIN;

-- Step 2: Acquire shared locks on rows that must not change.
SELECT * FROM products WHERE category = 'electronics' FOR SHARE;

-- Step 3: Perform calculations using the locked rows.
-- Step 4: Commit.
COMMIT;
```

**Expected output**:

```text
 id | name      | price
----+-----------+-------
 10 | Widget A  | 19.99
 11 | Widget B  | 29.99
(2 rows)
```

**Why this output appears**: `FOR SHARE` allows other sessions to read the same rows but blocks any `UPDATE` or `DELETE` until the transaction commits.

##### Real-World Cases

- **Price verification**: A transaction reads prices with shared locks to compute a quote while preventing concurrent price changes.
- **Report generation**: A report locks the rows it reads with shared locks for consistency.

---

#### 3.3 `FOR NO KEY UPDATE`

##### Definitions

**Core Definition**  
`FOR NO KEY UPDATE` acquires a row-level lock that is weaker than `FOR UPDATE` but still prevents other transactions from acquiring `FOR UPDATE`, `FOR SHARE`, or `FOR KEY SHARE` locks that would conflict.

**Technical Definition**  
`FOR NO KEY UPDATE` behaves similarly to `FOR UPDATE` but with a weaker lock. It does not block `SELECT … FOR KEY SHARE`, which allows foreign-key checks to proceed. It is used by `UPDATE` statements that do not modify key columns.

**Beginner-Friendly Explanation**  
`FOR NO KEY UPDATE` means “I’m updating this row, but not the key columns, so foreign-key checks can still proceed.”

##### Purposes

- To allow updates to non-key columns without blocking foreign-key validation.
- To reduce contention in schemas with many foreign-key relationships.

##### Syntax Rules and Structure

```sql
SELECT columns FROM table WHERE condition FOR NO KEY UPDATE;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `FOR NO KEY UPDATE` | Requests a lock weaker than `FOR UPDATE` but stronger than `FOR SHARE`. |

**Constraints and limitations**

- Cannot be used with `DISTINCT` or aggregate functions in the same query block.
- Still blocks `FOR UPDATE` and `FOR SHARE` requests.

##### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: PostgreSQL – `FOR NO KEY UPDATE` on a non-key column

```sql
-- Step 1: Begin a transaction.
BEGIN;

-- Step 2: Lock a row for a non-key update.
SELECT * FROM orders WHERE id = 100 FOR NO KEY UPDATE;

-- Step 3: Update a non-key column.
UPDATE orders SET status = 'shipped' WHERE id = 100;

-- Step 4: Commit.
COMMIT;
```

**Expected output**:

```text
 id  | status
-----+---------
 100 | pending
(1 row)

UPDATE 1
COMMIT
```

**Why this output appears**: `FOR NO KEY UPDATE` allows foreign-key checks (which use `FOR KEY SHARE`) to proceed while still preventing conflicting updates.

##### Real-World Cases

- **Order status updates**: Changing an order’s status (a non-key column) does not need to block foreign-key validation of related tables.
- **High-throughput updates**: Reducing lock strength on non-key updates improves concurrency.

---

#### 3.4 `FOR KEY SHARE`

##### Definitions

**Core Definition**  
`FOR KEY SHARE` acquires the weakest row-level lock, blocking only operations that would change the key values of the locked rows.

**Technical Definition**  
`SELECT … FOR KEY SHARE` acquires a shared lock that blocks `DELETE` and `UPDATE` statements that change key columns, but allows `FOR NO KEY UPDATE`, `FOR SHARE`, and plain `SELECT`. It is used internally by foreign-key checks.

**Beginner-Friendly Explanation**  
`FOR KEY SHARE` means “I’m checking this row’s key; don’t delete it or change its key, but you can change other columns.”

##### Purposes

- To allow foreign-key checks without blocking non-key updates.
- To minimise lock contention in schemas with referential integrity constraints.

##### Syntax Rules and Structure

```sql
SELECT columns FROM table WHERE condition FOR KEY SHARE;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `FOR KEY SHARE` | Requests the weakest row-level lock. |

**Constraints and limitations**

- Does not block `FOR NO KEY UPDATE`.
- Still blocks `DELETE` and key-changing `UPDATE`.

##### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: PostgreSQL – `FOR KEY SHARE` for a foreign-key check

```sql
-- Step 1: Begin a transaction.
BEGIN;

-- Step 2: Acquire a key-share lock on the referenced row.
SELECT * FROM customers WHERE id = 50 FOR KEY SHARE;

-- Step 3: Insert a child row referencing customer 50.
INSERT INTO orders (customer_id, amount) VALUES (50, 100.00);

-- Step 4: Commit.
COMMIT;
```

**Expected output**:

```text
 id | name
----+------
 50 | Acme
(1 row)

INSERT 0 1
COMMIT
```

**Why this output appears**: `FOR KEY SHARE` ensures the referenced customer row cannot be deleted or have its key changed, while still allowing updates to other columns.

##### Real-World Cases

- **Referential integrity**: Foreign-key checks use `FOR KEY SHARE` to protect referenced keys without blocking unrelated updates.
- **Parent-child inserts**: Inserting child rows acquires key-share locks on parent rows.

---

### 4. Lock Escalation

#### Definitions

**Core Definition**  
Lock escalation is the process by which an RDBMS converts many fine-grained locks (row or page locks) into a single coarser-grained lock (typically a table lock) to reduce lock-management overhead.

**Technical Definition**  
SQL Server performs lock escalation dynamically when the number of locks held by a single statement on a single table exceeds an internal threshold (often around 5,000 locks) or when lock memory exceeds a threshold. The escalated lock is a table-level shared (`S`) or exclusive (`X`) lock. PostgreSQL does **not** perform SQL Server–style lock escalation for ordinary row locks; its lock table can grow dynamically, constrained by `max_locks_per_transaction`.

**Beginner-Friendly Explanation**  
Imagine you’re putting sticky notes on individual pages of a book. If you use too many notes, you might replace them with one big note on the cover saying “the whole book is in use.” That’s lock escalation.

#### Purposes

- To reduce memory consumed by lock data structures.
- To improve performance when a statement legitimately needs to access most rows of a table.
- To simplify lock management under memory pressure.

#### Syntax Rules and Structure

Lock escalation is automatic; it can be influenced by hints and configuration.

**SQL Server example (hint)**:

```sql
SELECT * FROM Orders WITH (TABLOCK) WHERE OrderDate = '2025-01-01';
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `TABLOCK` | Hint that requests a table-level lock instead of row/page locks. |
| `ALTER TABLE … LOCK_ESCALATION` | Controls escalation behaviour (e.g., `AUTO`, `TABLE`, `DISABLE`). |
| Trace flag 1211 / 1224 | Disables or modifies lock escalation in SQL Server. |

**Constraints and limitations**

- Escalation can block many users unexpectedly, causing contention spikes.
- It cannot be completely disabled in all editions without trace flags or configuration.
- PostgreSQL does not escalate row locks to table locks in the same way; it relies on the lock table and `max_locks_per_transaction`.

#### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: SQL Server – Observing lock escalation

```sql
-- Step 1: Create a table with many rows.
-- (Assume a table "BigTable" with 1,000,000 rows.)

-- Step 2: Begin a transaction and update a large number of rows.
BEGIN TRANSACTION;
UPDATE BigTable SET status = 'processed' WHERE status = 'pending';
-- If the number of locks exceeds the threshold, SQL Server may escalate
-- to a table-level exclusive lock.

-- Step 3: Check sys.dm_tran_locks for a TAB-level X lock.
SELECT resource_type, request_mode, request_status
FROM sys.dm_tran_locks
WHERE resource_type = 'OBJECT';

-- Step 4: Commit.
COMMIT TRANSACTION;
```

**Expected output** (illustrative):

```text
resource_type request_mode request_status
------------- ------------ --------------
OBJECT        X            GRANT
```

**Why this output appears**: After escalation, a single table-level `X` lock appears in `sys.dm_tran_locks` instead of thousands of row locks.

###### Example 2: SQL Server – Preventing escalation by batching

```sql
-- Step 1: Delete large numbers of rows in small batches to avoid escalation.
DECLARE @done bit = 0;
WHILE @done = 0
BEGIN
    DELETE TOP (1000) FROM LogMessages WHERE LogDate < '2025-01-01';
    IF @@ROWCOUNT < 1000 SET @done = 1;
END;
```

**Expected output**:

```text
(1000 rows affected)
(1000 rows affected)
...
```

**Why this output appears**: Each batch acquires a small number of row locks, staying below the escalation threshold.

#### Real-World Cases

- **Batch jobs**: Large `UPDATE` or `DELETE` statements can trigger escalation and block OLTP traffic; batching avoids this.
- **Reporting queries**: A query that scans most of a table may escalate to a table lock, which is acceptable if the table is not concurrently modified.

---

### 5. Lock Timeouts

#### Definitions

**Core Definition**  
A lock timeout is a maximum time a statement or transaction will wait to acquire a lock before failing with an error.

**Technical Definition**  
Lock timeout is configurable at the session or statement level. In SQL Server, `SET LOCK_TIMEOUT` specifies the number of milliseconds a statement waits for a lock; the default is infinite (`-1`). In PostgreSQL, `lock_timeout` sets the maximum time to wait for a lock; `0` disables the timeout. In MySQL, `innodb_lock_wait_timeout` controls how long a transaction waits for a row lock.

**Beginner-Friendly Explanation**  
A lock timeout says “if I can’t get the lock within X seconds, give up and tell me there’s a problem” instead of waiting forever.

#### Purposes

- To prevent indefinite blocking when a lock cannot be acquired.
- To allow applications to handle contention gracefully (e.g., retry with backoff).
- To improve responsiveness in user-facing systems.

#### Syntax Rules and Structure

**SQL Server**:

```sql
SET LOCK_TIMEOUT 5000;  -- 5 seconds
```

**PostgreSQL**:

```sql
SET lock_timeout = '5s';
```

**MySQL**:

```sql
SET innodb_lock_wait_timeout = 10;  -- seconds
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `SET LOCK_TIMEOUT` | SQL Server session setting (milliseconds). |
| `lock_timeout` | PostgreSQL session/transaction setting. |
| `innodb_lock_wait_timeout` | MySQL InnoDB lock wait timeout (seconds). |

**Constraints and limitations**

- A lock timeout is not the same as a deadlock; a timeout occurs when a lock is simply not available within the specified period.
- In SQL Server, `LOCK_TIMEOUT` does not prevent deadlocks; deadlocks are detected separately.
- Timeout values should be tuned to application requirements; too short causes frequent failures, too long causes poor responsiveness.

#### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: SQL Server – Setting a lock timeout

```sql
-- Step 1: Set a 5-second lock timeout.
SET LOCK_TIMEOUT 5000;

-- Step 2: Attempt an update that may be blocked.
UPDATE accounts SET balance = balance - 100 WHERE id = 1;

-- Step 3: If the lock cannot be acquired within 5 seconds,
--         error 1222 is raised.
```

**Expected output** (timeout case):

```text
Msg 1222, Level 16, State 51, Line 1
Lock request time out period exceeded.
```

**Why this output appears**: The statement waited 5,000 milliseconds for the lock and then failed with error 1222.

###### Example 2: PostgreSQL – Setting `lock_timeout`

```sql
-- Step 1: Set a 2-second lock timeout.
SET lock_timeout = '2s';

-- Step 2: Attempt to lock a table.
LOCK TABLE accounts IN ACCESS EXCLUSIVE MODE;

-- Step 3: If the lock is not acquired within 2 seconds, an error is raised.
```

**Expected output** (timeout case):

```text
ERROR:  canceling statement due to lock timeout
```

**Why this output appears**: `lock_timeout` aborts the statement if the lock is not granted within the specified interval.

#### Real-World Cases

- **Web applications**: A short lock timeout prevents user requests from hanging indefinitely when a row is locked by a long-running batch job.
- **ETL pipelines**: Lock timeouts allow ETL jobs to fail fast and retry rather than waiting indefinitely for a table lock.

---

### 6. Deadlocks

#### Definitions

**Core Definition**  
A deadlock is a situation in which two or more transactions are each waiting for a lock held by the other, forming a circular wait that can never resolve without intervention.

**Technical Definition**  
A deadlock occurs when transaction A holds a lock on resource 1 and requests a lock on resource 2, while transaction B holds a lock on resource 2 and requests a lock on resource 1. Neither can proceed, and the RDBMS must detect the cycle and abort one transaction (the “victim”) to break the deadlock. Deadlocks are detected by the lock manager, typically after a timeout period (`deadlock_timeout` in PostgreSQL).

**Beginner-Friendly Explanation**  
Two people each hold one end of a rope and refuse to let go until the other lets go first. Neither can move, so one of them has to be forced to drop the rope.

#### Purposes

- (Deadlocks are not a feature to be used; they are a problem to be detected and prevented.)

#### Syntax Rules and Structure

Deadlocks are not created by explicit syntax; they arise from the interaction of locking statements.

**Canonical deadlock pattern**:

```sql
-- Transaction A
BEGIN;
UPDATE table1 SET ... WHERE id = 1;
UPDATE table2 SET ... WHERE id = 2;

-- Transaction B (concurrently)
BEGIN;
UPDATE table2 SET ... WHERE id = 2;
UPDATE table1 SET ... WHERE id = 1;
```

**Component breakdown**

| Step | Description |
|------|-------------|
| A locks row 1 of table1 | A holds a lock. |
| B locks row 2 of table2 | B holds a lock. |
| A requests row 2 of table2 | A waits for B. |
| B requests row 1 of table1 | B waits for A → deadlock. |

**Constraints and limitations**

- Deadlocks are inevitable in any system with locking; the goal is to minimise them and handle them gracefully.
- One transaction must be rolled back (the victim) to break the cycle.
- In PostgreSQL, the victim is chosen based on cost; in SQL Server, the transaction with the least work is typically chosen.

#### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: PostgreSQL – Observing a deadlock

```sql
-- Session 1:
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- (hold)

-- Session 2:
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 2;
-- (hold)

-- Session 1:
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
-- waits for Session 2

-- Session 2:
UPDATE accounts SET balance = balance + 100 WHERE id = 1;
-- deadlock detected; PostgreSQL aborts one transaction.
```

**Expected output** (one session):

```text
ERROR:  deadlock detected
DETAIL:  Process 12345 waits for ShareLock on transaction 67890; blocked by process 67890.
Process 67890 waits for ShareLock on transaction 12345; blocked by process 12345.
HINT:  See server log for query details.
```

**Why this output appears**: PostgreSQL detects the circular wait after `deadlock_timeout` and aborts one transaction with a deadlock error.

#### Real-World Cases

- **Bank transfers**: Two transfers moving money between the same two accounts in opposite directions can deadlock if rows are locked in different orders.
- **Order processing**: Two sessions updating order and order-detail tables in different sequences can deadlock.

---

### 7. Deadlock Detection

#### Definitions

**Core Definition**  
Deadlock detection is the mechanism by which an RDBMS identifies circular wait conditions among transactions and chooses a victim to abort.

**Technical Definition**  
Deadlock detection is performed by the lock manager. In PostgreSQL, detection runs after `deadlock_timeout` (default 1 second); the server builds a wait-for graph and aborts a transaction if a cycle is found. In SQL Server, a dedicated deadlock monitor thread checks for cycles every 5 seconds by default and kills the transaction with the least rollback cost. MySQL InnoDB performs deadlock detection immediately when a lock wait would create a cycle.

**Beginner-Friendly Explanation**  
The database periodically checks whether transactions are stuck in a loop of waiting and, if so, cancels one so the others can proceed.

#### Purposes

- To break deadlocks automatically without human intervention.
- To ensure that the database does not freeze due to circular waits.
- To log deadlock information for diagnosis.

#### Syntax Rules and Structure

Deadlock detection is automatic; configuration parameters control its timing.

**PostgreSQL parameters**:

- `deadlock_timeout` (default 1s) — time to wait before checking for deadlock.
- `log_lock_waits` — logs lock waits, useful for diagnosis.

**SQL Server**:

- Deadlock monitor runs automatically; deadlock graphs can be captured via Extended Events or trace flags.

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `deadlock_timeout` | PostgreSQL wait period before detection. |
| Wait-for graph | Internal structure representing transaction dependencies. |
| Victim selection | The transaction chosen for rollback. |

**Constraints and limitations**

- Detection is not instantaneous; there is a delay (especially in PostgreSQL due to `deadlock_timeout`).
- Deadlock detection itself consumes resources; very frequent detection may indicate a design problem.

#### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: PostgreSQL – Configuring `deadlock_timeout`

```sql
-- Step 1: Set a shorter deadlock detection delay for testing.
ALTER SYSTEM SET deadlock_timeout = '500ms';
SELECT pg_reload_conf();

-- Step 2: Create a deadlock as in the previous example.
-- The deadlock will be detected after approximately 500ms.
```

**Expected output**:

```text
ALTER SYSTEM
 pg_reload_conf
----------------
 t
(1 row)
```

**Why this output appears**: Reducing `deadlock_timeout` makes detection faster, at the cost of more frequent (and potentially unnecessary) checks.

#### Real-World Cases

- **Production monitoring**: Deadlock graphs are captured and analysed to identify problematic query patterns.
- **Application retries**: When a deadlock error occurs, the application retries the transaction.

---

### 8. Deadlock Prevention

#### Definitions

**Core Definition**  
Deadlock prevention is the set of design and coding practices that reduce or eliminate the occurrence of deadlocks.

**Technical Definition**  
Prevention techniques include consistent lock ordering (accessing resources in a fixed sequence), keeping transactions short, using lower isolation levels where appropriate, avoiding user interaction within transactions, and using lock hints such as `NOWAIT` or `SKIP LOCKED`. In PostgreSQL, consistent lock ordering is the primary prevention technique. In SQL Server, keeping transactions short and reducing lock footprints are recommended.

**Beginner-Friendly Explanation**  
If everyone agrees to always pick up the rope from the same end first, the rope-pulling deadlock can’t happen. Similarly, if all transactions access tables and rows in the same order, deadlocks are avoided.

#### Purposes

- To reduce the frequency of deadlocks.
- To improve application reliability and user experience.
- To avoid the overhead of repeated rollbacks and retries.

#### Syntax Rules and Structure

Prevention is a design discipline, not a single syntax.

**Consistent lock ordering example**:

```sql
-- Always update the lower ID first.
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

**Component breakdown**

| Technique | Description |
|-----------|-------------|
| Consistent ordering | Always lock resources in the same order. |
| Short transactions | Minimise the time locks are held. |
| `NOWAIT` / `SKIP LOCKED` | Avoid waiting for locks that may lead to deadlock. |
| Lower isolation | Reduce lock scope (e.g., `READ COMMITTED`). |

**Constraints and limitations**

- Prevention cannot eliminate all deadlocks; detection and retry logic are still required.
- Consistent ordering may be difficult in complex applications with multiple entry points.

#### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: PostgreSQL – Consistent lock ordering

```sql
-- Step 1: Always transfer from the lower account ID to the higher ID.
BEGIN;

-- Step 2: Lock the lower ID first.
UPDATE accounts SET balance = balance - 100 WHERE id = 1;

-- Step 3: Then lock the higher ID.
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

**Expected output**:

```text
BEGIN
UPDATE 1
UPDATE 1
COMMIT
```

**Why this output appears**: By acquiring locks in a consistent order, concurrent transfers cannot form a circular wait.

###### Example 2: SQL Server – Using `SET LOCK_TIMEOUT` and retry logic

```sql
-- Step 1: Set a short lock timeout.
SET LOCK_TIMEOUT 2000;

-- Step 2: Attempt the transaction.
BEGIN TRY
    BEGIN TRANSACTION;
    UPDATE TableA SET col = 1 WHERE id = 1;
    UPDATE TableB SET col = 2 WHERE id = 2;
    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    IF ERROR_NUMBER() = 1222
    BEGIN
        ROLLBACK TRANSACTION;
        -- Retry logic here.
    END
END CATCH;
```

**Expected output** (successful case):

```text
(1 row affected)
(1 row affected)
```

**Why this output appears**: The short timeout allows the application to detect contention quickly and retry, reducing the chance of a deadlock.

#### Real-World Cases

- **Banking systems**: Transfers always lock accounts in ascending account-number order.
- **E-commerce**: Order and inventory updates follow a fixed sequence (order first, then inventory).

---

### 9. Lock Contention

#### Definitions

**Core Definition**  
Lock contention is the situation in which multiple transactions compete for the same lock, causing some to wait.

**Technical Definition**  
Lock contention occurs when the lock request rate for a resource exceeds the rate at which locks are released. It manifests as lock waits, increased latency, and reduced throughput. High contention can be caused by long-running transactions, hot rows, missing indexes, or inappropriate isolation levels.

**Beginner-Friendly Explanation**  
Contention is a queue of people waiting to use the same bathroom; the longer each person takes, the longer the queue grows.

#### Purposes

- (Contention is a problem to be diagnosed and reduced, not a feature.)

#### Syntax Rules and Structure

Contention is observed through monitoring views and resolved through design changes.

**Common causes**:

- Long-running transactions holding locks.
- Missing indexes causing full scans and excessive row locks.
- Hot rows (e.g., a counter row updated by every transaction).
- Uncommitted transactions.

**Component breakdown**

| Cause | Mitigation |
|-------|------------|
| Long transactions | Shorten transactions; batch work. |
| Missing indexes | Add indexes to reduce scanned rows. |
| Hot rows | Partition or use queuing (e.g., `SKIP LOCKED`). |
| High isolation | Use lower isolation levels where possible. |

**Constraints and limitations**

- Some contention is inherent in high-concurrency systems.
- Reducing contention often requires trade-offs (e.g., lower isolation may allow anomalies).

#### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: PostgreSQL – Diagnosing contention with `pg_locks`

```sql
-- Step 1: Identify blocked and blocking sessions.
SELECT
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_statement,
    blocking_activity.query AS blocking_statement
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity
    ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
    AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
    AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity
    ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

**Expected output** (illustrative):

```text
 blocked_pid | blocked_user | blocking_pid | blocking_user | blocked_statement | blocking_statement
-------------+--------------+--------------+---------------+-------------------+--------------------
       12345 | app_user     |        67890 | batch_user    | UPDATE accounts … | UPDATE accounts …
```

**Why this output appears**: The query joins `pg_locks` with `pg_stat_activity` to show which session is blocking which, and the SQL statements involved.

#### Real-World Cases

- **Hot counter updates**: A `page_views` counter row is updated by every request, causing severe contention; a queue or sharded counter mitigates it.
- **Batch vs OLTP**: A nightly batch update holds locks that block daytime OLTP queries; scheduling the batch during off-peak hours reduces contention.

---

### 10. Lock Monitoring

#### Definitions

**Core Definition**  
Lock monitoring is the practice of inspecting the current lock state of a database to diagnose contention, deadlocks, and blocking chains.

**Technical Definition**  
Lock monitoring uses system views and commands to inspect granted and waiting locks, identify blocking sessions, and measure lock wait times. Key tools include `pg_locks` and `pg_stat_activity` in PostgreSQL, `sys.dm_tran_locks` in SQL Server, and `SHOW ENGINE INNODB STATUS` in MySQL.

**Beginner-Friendly Explanation**  
Lock monitoring is like looking at the queue at the bathroom to see who is inside, who is waiting, and why the line isn’t moving.

#### Purposes

- To identify blocking chains and their root causes.
- To measure lock wait times and contention severity.
- To diagnose deadlocks and near-deadlock situations.
- To validate the effectiveness of contention-reduction measures.

#### Syntax Rules and Structure

**PostgreSQL**:

```sql
SELECT * FROM pg_locks;
SELECT * FROM pg_stat_activity;
```

**SQL Server**:

```sql
SELECT * FROM sys.dm_tran_locks;
```

**MySQL**:

```sql
SHOW ENGINE INNODB STATUS\G
```

**Component breakdown**

| Tool | RDBMS | Purpose |
|------|-------|---------|
| `pg_locks` | PostgreSQL | One row per lockable object and lock mode. |
| `pg_stat_activity` | PostgreSQL | Session-level information, including blocked queries. |
| `sys.dm_tran_locks` | SQL Server | Active lock manager resources. |
| `SHOW ENGINE INNODB STATUS` | MySQL | InnoDB lock and transaction information. |

**Constraints and limitations**

- Monitoring views can be large and impact performance if queried frequently on busy systems.
- `pg_locks` does not directly show which session is blocking which; joins with `pg_stat_activity` and `pg_blocking_pids()` are needed.
- `SHOW ENGINE INNODB STATUS` output is limited to 1 MB.

#### Multiple Annotated Complete Step-by-Step Code Examples

###### Example 1: PostgreSQL – Simple lock monitor query

```sql
-- Step 1: List all granted locks with relation names.
SELECT
    l.locktype,
    l.mode,
    l.granted,
    c.relname AS relation
FROM pg_locks l
LEFT JOIN pg_class c ON c.oid = l.relation
WHERE l.granted
ORDER BY c.relname, l.mode;
```

**Expected output**:

```text
 locktype |     mode      | granted | relation
----------+---------------+---------+----------
 relation | AccessShareLock | t     | accounts
 relation | RowExclusiveLock| t     | accounts
 tuple    | ExclusiveLock   | t     | accounts
```

**Why this output appears**: `pg_locks` exposes one row per lock; joining `pg_class` resolves the relation OID to a table name.

###### Example 2: SQL Server – Lock monitor query

```sql
-- Step 1: List active locks with resource type and mode.
SELECT
    resource_type,
    request_mode,
    request_status,
    resource_database_id,
    resource_associated_entity_id
FROM sys.dm_tran_locks
WHERE request_status IN ('GRANT', 'WAIT');
```

**Expected output**:

```text
resource_type request_mode request_status resource_database_id ...
------------- ------------ -------------- -------------------- ...
OBJECT        IX           GRANT          5
KEY           X            GRANT          5
KEY           X            WAIT           5
```

**Why this output appears**: `sys.dm_tran_locks` returns one row per active lock request, including waiting requests.

###### Example 3: MySQL – InnoDB lock monitor output

```sql
-- Step 1: Show InnoDB engine status, including lock information.
SHOW ENGINE INNODB STATUS\G
```

**Expected output** (excerpt):

```text
------------------------
TRANSACTIONS
------------------------
Trx id counter 123456
...
---TRANSACTION 123450, ACTIVE 10 sec
2 lock struct(s), heap size 1136, 1 row lock(s)
MySQL thread id 12, OS thread handle 140..., query id 45 ...
```

**Why this output appears**: `SHOW ENGINE INNODB STATUS` includes a `TRANSACTIONS` section that lists active transactions and their locks.

#### Real-World Cases

- **Incident response**: A DBA uses lock monitoring to identify the head blocker during an outage.
- **Performance tuning**: Lock wait statistics guide index creation and transaction shortening.
- **Capacity planning**: Long-term lock monitoring reveals hot spots that require schema or application changes.

---

## References

- PostgreSQL Documentation – Explicit Locking – https://www.postgresql.org/docs/current/explicit-locking.html
- PostgreSQL Documentation – Lock Management – https://www.postgresql.org/docs/current/runtime-config-locks.html
- PostgreSQL Documentation – `pg_locks` – https://www.postgresql.org/docs/current/view-pg-locks.html
- PostgreSQL Wiki – Lock Monitoring – https://wiki.postgresql.org/wiki/Lock_Monitoring
- Microsoft Learn – SQL Server Transaction Locking and Row Versioning Guide – https://learn.microsoft.com/en-us/previous-versions/sql/sql-server-guides/jj856598(v=sql.110)
- Microsoft Learn – `sys.dm_tran_locks` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-locks-transact-sql
- Microsoft Learn – Resolve Blocking Problems Caused by Lock Escalation – https://learn.microsoft.com/en-us/troubleshoot/sql/database-engine/performance/resolve-blocking-problems-caused-lock-escalation
- Microsoft Learn – Deadlocks Guide – https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide
- MySQL 8.0 Reference Manual – InnoDB Locking – https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/innodb-locking.html
- MySQL 8.0 Reference Manual – InnoDB Standard Monitor and Lock Monitor Output – https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/innodb-standard-monitor.html
- MySQL 8.0 Reference Manual – `SHOW ENGINE` Statement – https://dev.mysql.com/doc/refman/8.0/en/show-engine.html
- Oracle Database – Instance Tuning Using Performance Views (Lock Contention) – https://docs.oracle.com/en/database/oracle/oracle-database/19/tgdba/instance-tuning-using-performance-views.html
- IBM Documentation – Diagnosing and Resolving Locking Problems – https://www.ibm.com/docs/en
- Redgate – LCK_M_U and Lock Escalation – https://documentation.red-gate.com
- Erik Darling – Using Lock Timeouts to Avoid Deadlocks in SQL Server – https://erikdarling.com