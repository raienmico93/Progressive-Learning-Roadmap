# Comprehensive Programming Cheat Sheet: SQL DELETE Operations

## Topic Overview

### Definitions

**Core Definition**
DELETE operations are the Data Manipulation Language (DML) statements used to remove existing rows from a table, optionally filtered by a `WHERE` condition, and optionally returning the deleted data via `RETURNING` (PostgreSQL) or `OUTPUT` (SQL Server).

**Technical Definition**
`DELETE` removes rows that satisfy the `WHERE` clause from the specified table; if the `WHERE` clause is absent, the effect is to delete all rows in the table, leaving a valid but empty table. The statement supports single-table and multi-table forms, subqueries, common table expressions (CTEs), and—in PostgreSQL—`USING` for joined deletes and `RETURNING` for returning deleted data. MySQL provides single-table (`DELETE FROM t WHERE ...`) and multiple-table (`DELETE t1, t2 FROM ...` or `DELETE FROM t1, t2 USING ...`) syntax, with `ORDER BY` and `LIMIT` available only in the single-table form. SQL Server provides `TOP`, `OUTPUT`, CTEs, and a second `FROM` clause for join-based deletes. `DELETE` is a DML operation that acquires row-level locks, is fully transactional (and therefore rollback-able), fires triggers, and writes each deleted row to the transaction log.

**Beginner-Friendly Explanation**
DELETE is how you remove records from a table. You tell the database which table to remove from and—crucially—which rows should be removed using a `WHERE` clause. Without a `WHERE` clause, DELETE empties the entire table. DELETE is careful and reversible (if you are inside a transaction), but it can be slow on large tables because it processes rows one by one, locks them, and writes each deletion to the log. For wiping an entire table quickly, `TRUNCATE` is often a better choice; for removing the table itself, `DROP` is the tool.

### Key Characteristics

- **`WHERE` is the safety net**: Omitting `WHERE` deletes every row. Production guidelines universally require a `WHERE` clause; MySQL's `sql_safe_updates` and PostgreSQL's `pg-safeupdate` can enforce this at the session level.
- **Transactional and rollback-able**: In PostgreSQL, SQL Server, and MySQL (with InnoDB), a `DELETE` inside a transaction can be rolled back. `TRUNCATE` is not rollback-able in MySQL and Oracle, though it is in PostgreSQL and SQL Server.
- **Row-level locking**: `DELETE` acquires row-level locks (or exclusive row locks), allowing concurrent reads and writes to other rows in the same table in MVCC systems. This is in contrast to `TRUNCATE`, which locks the whole table.
- **Triggers fire**: `DELETE` activates `BEFORE`, `AFTER`, and `INSTEAD OF` triggers; `TRUNCATE` and `DROP` do not fire `DELETE` triggers.
- **Vendor divergence in multi-table and returning syntax**: PostgreSQL uses `USING` and `RETURNING`; MySQL uses multiple-table syntax with `JOIN` and has no `RETURNING`; SQL Server uses `OUTPUT` and a second `FROM` clause for joins.

### Prerequisites

- A running RDBMS instance (PostgreSQL, MySQL, or SQL Server) and a client connection.
- `DELETE` privilege on the target table; `SELECT` privilege on any tables read in the `WHERE` clause, `USING` clause, or subqueries.
- For `RETURNING` (PostgreSQL): `SELECT` privilege on the returned columns, even if the `RETURNING` list does not reference a column (e.g., `RETURNING 1`).
- For cascading deletes: foreign keys defined with `ON DELETE CASCADE` on child tables.
- A tested backup or point-in-time recovery strategy for irreplaceable data.

### Related Programming Areas

- Data Manipulation Language (DML) – `INSERT`, `UPDATE`, `MERGE`.
- Data Definition Language (DDL) – `TRUNCATE`, `DROP`.
- Transaction management – `COMMIT`, `ROLLBACK`, isolation levels, and lock timeouts.
- Concurrency control – row-level vs. table-level locking; MVCC vs. lock-based reads.
- Data retention and archival – partitioning, soft deletes, and `pg_archiver`-style tools.

---

## Core Concepts and Features

### 1. `DELETE`

#### Definitions

**Core Definition**
`DELETE` is the SQL statement that removes rows from a table.

**Technical Definition**
PostgreSQL's synopsis is `[ WITH [ RECURSIVE ] with_query [, ...] ] DELETE FROM [ ONLY ] table_name [ * ] [ [ AS ] alias ] [ USING from_item [, ...] ] [ WHERE condition | WHERE CURRENT OF cursor_name ] [ RETURNING { * | output_expression [ [ AS ] output_name ] } [, ...] ]`. MySQL's single-table syntax is `DELETE [LOW_PRIORITY] [QUICK] [IGNORE] FROM tbl_name [[AS] tbl_alias] [PARTITION (...)] [WHERE where_condition] [ORDER BY ...] [LIMIT row_count]`, and its multiple-table syntax is `DELETE [LOW_PRIORITY] [QUICK] [IGNORE] tbl_name[.*] [, tbl_name[.*]] ... FROM table_references [WHERE where_condition]` or `DELETE ... FROM tbl_name[.*] [, ...] USING table_references [WHERE where_condition]`. SQL Server's syntax includes `[ WITH <common_table_expression> ] DELETE [ TOP ( expression ) [ PERCENT ] ] [ FROM ] { table_alias | object | rowset_function_limited | @table_variable } [ <OUTPUT Clause> ] [ FROM table_source ] [ WHERE ... ]`.

**Beginner-Friendly Explanation**
`DELETE` is the command for removing records. You name the table and add a `WHERE` clause to specify which rows should go. Without `WHERE`, every row is removed.

#### Purposes

- To remove specific rows that are no longer needed while preserving the table structure.
- To delete data in response to user actions (e.g., deleting a post or an account).
- To purge stale or expired data as part of data-retention policies.
- To remove rows based on conditions derived from other tables (via subqueries, `USING`, or joins).

#### Syntax Rules and Structure

**General syntax (PostgreSQL):**

```sql
DELETE FROM table_name
[ USING other_table [, ...] ]
[ WHERE condition ]
[ RETURNING * | output_expression [, ...] ];
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `DELETE FROM` | The statement and target table. |
| `USING` (PostgreSQL) | Additional tables for the `WHERE` condition. |
| `WHERE` | The condition identifying rows to delete; omitting it deletes all rows. |
| `RETURNING` (PostgreSQL) | Returns data from the deleted rows. |

**Syntax rules:**

- The `WHERE` clause is optional but omitting it deletes every row.
- In PostgreSQL, `ONLY` restricts the delete to the named table, excluding inheritance descendants; `*` explicitly includes them.
- MySQL's `ORDER BY` and `LIMIT` apply only to single-table deletes, not multiple-table deletes.

**Constraints and limitations:**

- You cannot delete from a table and select from the same table in a subquery in MySQL.
- SQL Server's `DELETE` fails if a row is referenced by a `FOREIGN KEY` constraint that does not have a cascading action.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Basic DELETE with WHERE

```sql
-- Step 1: Create a table and insert sample data.
CREATE TABLE logs (id serial PRIMARY KEY, message text, created_at timestamptz DEFAULT now());
INSERT INTO logs (message) VALUES ('old entry'), ('recent entry');

-- Step 2: Delete old rows.
DELETE FROM logs WHERE created_at < now() - interval '1 day';

-- Step 3: Verify.
SELECT * FROM logs;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 2
DELETE 0
 id |   message    |          created_at
----+--------------+-------------------------------
  1 | old entry    | 2026-11-13 10:00:00.123+00
  2 | recent entry | 2026-11-13 10:00:00.123+00
(2 rows)
```

**Why this output appears**: The `WHERE` clause filters by `created_at`; because both rows were just inserted, neither is older than one day, so `DELETE 0` confirms that zero rows were removed. The table remains intact.

#### Real-World Cases

- **Data retention**: A logs table is purged of rows older than 90 days on a nightly schedule.
- **User-initiated deletion**: A user deletes their account, and the application runs `DELETE FROM users WHERE user_id = $1`.
- **Cleanup**: Orphaned rows in a staging table are deleted after a failed ETL job.

---

### 2. `DELETE` with `WHERE`

#### Definitions

**Core Definition**
`DELETE` with `WHERE` removes only the rows that satisfy the specified condition, leaving other rows untouched.

**Technical Definition**
The `WHERE` clause contains a Boolean expression; only rows for which the expression returns `TRUE` are deleted. The condition can reference columns of the target table, subqueries, and—in PostgreSQL—tables listed in the `USING` clause. MySQL's single-table `DELETE` supports `ORDER BY` and `LIMIT` alongside `WHERE` to control the order and number of rows deleted.

**Beginner-Friendly Explanation**
The `WHERE` clause is how you say “only delete these rows.” It is the most important part of a DELETE statement because it prevents you from accidentally wiping the entire table.

#### Purposes

- To target specific rows for deletion based on their values or timestamps.
- To delete rows that match a condition derived from application logic.
- To limit deletion to a subset of rows using `LIMIT` (MySQL) or `TOP` (SQL Server).

#### Syntax Rules and Structure

**General syntax:**

```sql
DELETE FROM table_name WHERE condition;
```

**MySQL with LIMIT:**

```sql
DELETE FROM table_name WHERE condition LIMIT row_count;
```

**SQL Server with TOP:**

```sql
DELETE TOP (expression) FROM table_name WHERE condition;
```

**Syntax rules:**

- The `WHERE` clause must evaluate to a Boolean; only rows where it is `TRUE` are deleted.
- MySQL's `LIMIT` caps the number of rows deleted in a single statement, enabling batching.

**Constraints and limitations:**

- If the `WHERE` clause matches zero rows, the statement succeeds but deletes nothing.
- A non-deterministic `TOP` without `ORDER BY` (SQL Server) may delete arbitrary rows.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – DELETE with WHERE and LIMIT

```sql
-- Step 1: Create and populate a table.
CREATE TABLE orders (id INT PRIMARY KEY, status VARCHAR(20), created_at DATE);
INSERT INTO orders VALUES (1, 'pending', '2023-01-01'), (2, 'pending', '2023-01-02'), (3, 'shipped', '2023-01-03');

-- Step 2: Delete pending orders, limited to one row.
DELETE FROM orders WHERE status = 'pending' LIMIT 1;

-- Step 3: Verify.
SELECT * FROM orders;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 3 rows affected (0.01 sec)
Query OK, 1 row affected (0.00 sec)
+----+---------+------------+
| id | status  | created_at |
+----+---------+------------+
|  2 | pending | 2023-01-02 |
|  3 | shipped | 2023-01-03 |
+----+---------+------------+
```

**Why this output appears**: The `WHERE` clause matches two pending rows, but `LIMIT 1` restricts the delete to one row. Which pending row is deleted is not guaranteed without `ORDER BY`; in practice, the first matching row is typically removed.

#### Real-World Cases

- **Session cleanup**: Expired sessions are deleted with `DELETE FROM sessions WHERE expires_at < now()`.
- **Order cancellation**: Pending orders older than 30 days are deleted after a grace period.

---

### 3. `DELETE` with `USING` (PostgreSQL)

#### Definitions

**Core Definition**
`DELETE ... USING` is a PostgreSQL extension that allows the `WHERE` condition to reference columns from other tables, enabling join-based deletes.

**Technical Definition**
The `USING` clause lists additional table expressions that can be referenced in the `WHERE` condition. The target table is implicitly joined to the `USING` tables, and rows in the target table that satisfy the condition are deleted. PostgreSQL notes that there are two ways to delete rows using information in other tables: sub-selects and the `USING` clause, and which is more appropriate depends on the circumstances.

**Beginner-Friendly Explanation**
`USING` lets you delete rows from one table based on information in another table. For example, you can delete all orders belonging to customers who have been banned, without writing a subquery.

#### Purposes

- To delete rows from a table based on conditions in related tables.
- To avoid correlated subqueries when the deletion depends on a join.
- To perform set-based deletes that reference multiple tables cleanly.

#### Syntax Rules and Structure

**General syntax:**

```sql
DELETE FROM target_table
USING source_table
WHERE target_table.key = source_table.key
  AND condition;
```

**Syntax rules:**

- The `USING` clause uses the same syntax as the `FROM` clause of a `SELECT` statement.
- Do not repeat the target table in `USING` unless you intend a self-join, in which case it must appear with an alias.

**Constraints and limitations:**

- PostgreSQL-specific; MySQL and SQL Server use different join syntax (multiple-table `DELETE` and a second `FROM`, respectively).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – DELETE USING a join

```sql
-- Step 1: Create tables.
CREATE TABLE customers (id int PRIMARY KEY, status text);
CREATE TABLE orders (id int PRIMARY KEY, customer_id int, total numeric);

INSERT INTO customers VALUES (1, 'banned'), (2, 'active');
INSERT INTO orders VALUES (100, 1, 50), (101, 2, 75), (102, 1, 30);

-- Step 2: Delete orders belonging to banned customers.
DELETE FROM orders
USING customers
WHERE orders.customer_id = customers.id
  AND customers.status = 'banned';

-- Step 3: Verify.
SELECT * FROM orders;
```

**Expected output:**

```text
CREATE TABLE
CREATE TABLE
INSERT 0 2
INSERT 0 3
DELETE 2
 id  | customer_id | total
-----+-------------+-------
 101 |           2 |    75
(1 row)
```

**Why this output appears**: The `USING customers` clause joins `orders` to `customers`; the `WHERE` condition matches only orders whose customer has `status = 'banned'`, deleting two rows and leaving the active customer's order intact.

#### Real-World Cases

- **Fraud cleanup**: Orders from customers flagged as fraudulent are deleted using `DELETE ... USING` before the accounts are closed.
- **Dependency removal**: Child rows are deleted when their parent is removed, using a join on the foreign key.

---

### 4. `DELETE` with Subqueries

#### Definitions

**Core Definition**
A delete with subqueries uses a subquery in the `WHERE` clause (or, in PostgreSQL, in `USING`) to identify the rows to delete.

**Technical Definition**
A subquery-based delete typically uses `WHERE column IN (SELECT ...)` or `WHERE EXISTS (SELECT ...)`. The subquery can reference the target table (correlated subquery) or other tables. MySQL explicitly prohibits deleting from a table and selecting from the same table in a subquery; the workaround is to wrap the subquery in a derived table or use a multiple-table delete.

**Beginner-Friendly Explanation**
A subquery delete lets you say “delete the rows whose ID appears in this list,” where the list is produced by another query.

#### Purposes

- To delete rows that match a condition computed by a separate query.
- To delete rows based on a list of IDs from another table.
- To perform deletes that are portable across RDBMSs without vendor-specific `USING` or join syntax.

#### Syntax Rules and Structure

**General syntax:**

```sql
DELETE FROM table_name
WHERE column IN (SELECT column FROM other_table WHERE condition);
```

**PostgreSQL workaround for self-referencing subqueries:**

```sql
DELETE FROM table_name
WHERE id IN (SELECT id FROM (SELECT id FROM table_name WHERE condition) AS sub);
```

**Syntax rules:**

- The subquery must return a single column for `IN` comparisons.
- MySQL requires the derived-table workaround when the subquery references the target table.

**Constraints and limitations:**

- Subquery deletes can be slower than join-based deletes because the subquery may execute once per row.
- In MySQL, the target table cannot appear in the `FROM` clause of the subquery without the derived-table workaround.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – DELETE with a subquery

```sql
-- Step 1: Create tables.
CREATE TABLE users (id int PRIMARY KEY, email text);
CREATE TABLE banned_emails (email text);

INSERT INTO users VALUES (1, 'a@example.com'), (2, 'b@example.com'), (3, 'c@example.com');
INSERT INTO banned_emails VALUES ('b@example.com'), ('c@example.com');

-- Step 2: Delete users whose email is in the banned list.
DELETE FROM users
WHERE email IN (SELECT email FROM banned_emails);

-- Step 3: Verify.
SELECT * FROM users;
```

**Expected output:**

```text
CREATE TABLE
CREATE TABLE
INSERT 0 3
INSERT 0 2
DELETE 2
 id |     email
----+----------------
  1 | a@example.com
(1 row)
```

**Why this output appears**: The subquery returns the banned email addresses; the `IN` condition matches the two users with those emails, deleting them and leaving the non-banned user.

#### Real-World Cases

- **GDPR erasure**: User data is deleted where the user's email appears in a list of erasure requests.
- **Orphan cleanup**: Rows whose foreign key no longer exists in the parent table are deleted using a `NOT IN` or `NOT EXISTS` subquery.

---

### 5. `DELETE` with CTEs

#### Definitions

**Core Definition**
A delete with a CTE uses a `WITH` clause to define a named temporary result set that can be referenced in the `DELETE` statement, simplifying complex logic or enabling `RETURNING` from the CTE.

**Technical Definition**
PostgreSQL's `DELETE` synopsis begins with `[ WITH [ RECURSIVE ] with_query [, ...] ]`, allowing one or more subqueries to be referenced by name in the `DELETE` query. MySQL's `DELETE` can also start with a `WITH` clause to define CTEs accessible within the delete. SQL Server's `DELETE` accepts `[ WITH <common_table_expression> [ ,...n ] ]`. In PostgreSQL, a CTE containing a `DELETE ... RETURNING` can be used to capture deleted rows for further processing in the same statement.

**Beginner-Friendly Explanation**
A CTE is a temporary named query that you define before the delete. You can use it to pre-compute the rows you want to delete or to capture the deleted rows for logging or further processing.

#### Purposes

- To pre-compute a set of rows to delete, making the statement easier to read and optimise.
- To capture deleted rows via `RETURNING` inside a CTE for use in a subsequent part of the statement.
- To use recursive CTEs to delete hierarchical data (e.g., a tree of comments).

#### Syntax Rules and Structure

**General syntax (PostgreSQL):**

```sql
WITH cte_name AS (
    SELECT ...
)
DELETE FROM table_name
WHERE column IN (SELECT column FROM cte_name);
```

**CTE with RETURNING (PostgreSQL):**

```sql
WITH deleted AS (
    DELETE FROM table_name WHERE condition RETURNING *
)
SELECT * FROM deleted;
```

**Syntax rules:**

- The CTE is visible only within the `DELETE` statement.
- Multiple CTEs can be defined, separated by commas.
- In PostgreSQL, a CTE containing a data-modifying statement (`DELETE`) must have a `RETURNING` clause if it is referenced by the outer statement.

**Constraints and limitations:**

- MySQL's CTE support for `DELETE` may still encounter the self-reference restriction in some patterns.
- CTEs are not always materialised; the optimiser may inline them, so performance varies.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – CTE with DELETE ... RETURNING

```sql
-- Step 1: Create a table.
CREATE TABLE audit_log (id serial PRIMARY KEY, message text, created_at timestamptz DEFAULT now());
INSERT INTO audit_log (message) VALUES ('old'), ('new');

-- Step 2: Delete old rows and capture them in a CTE.
WITH deleted AS (
    DELETE FROM audit_log
    WHERE created_at < now() - interval '1 second'
    RETURNING id, message
)
SELECT * FROM deleted;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 2
 id | message
----+---------
(0 rows)
```

**Why this output appears**: The `WHERE` clause does not match any rows (both rows were inserted just now), so the `DELETE` removes nothing and the CTE returns an empty result. If rows had matched, the `RETURNING` clause would return their `id` and `message` values.

#### Real-World Cases

- **Audit logging**: Deleted rows are captured via a CTE and inserted into an audit table in the same statement.
- **Hierarchical deletion**: A recursive CTE identifies a tree of rows (e.g., threaded comments) and deletes them in the correct order.

---

### 6. `RETURNING` (PostgreSQL)

#### Definitions

**Core Definition**
`RETURNING` is a PostgreSQL clause that returns data from the rows deleted by a `DELETE` statement, computed over the actually deleted rows.

**Technical Definition**
The optional `RETURNING` clause causes `DELETE` to compute and return value(s) based on each row actually deleted. Any expression using the table's columns, and/or columns of other tables mentioned in `USING`, can be computed. The syntax of the `RETURNING` list is identical to that of the output list of `SELECT`. `RETURNING` requires `SELECT` privilege on the table, even if the list does not reference a column (e.g., `RETURNING 1`).

**Beginner-Friendly Explanation**
`RETURNING` lets you get data back from the rows you just deleted, in the same statement. This is useful for logging what was removed or for returning the deleted data to an application.

#### Purposes

- To capture the deleted rows for auditing or archiving.
- To return deleted data to an application in a single round-trip.
- To use the deleted rows in a subsequent CTE or `INSERT`.

#### Syntax Rules and Structure

**General syntax:**

```sql
DELETE FROM table_name
WHERE condition
RETURNING * | output_expression [ [ AS ] output_name ] [, ...];
```

**Syntax rules:**

- The `RETURNING` list can include `*` to return all columns.
- The expression can reference any column of the target table and any table in the `USING` clause.

**Constraints and limitations:**

- `RETURNING` is PostgreSQL-specific; MySQL has no equivalent, and SQL Server uses `OUTPUT`.
- The user must have `SELECT` privilege on the returned columns.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – DELETE with RETURNING

```sql
-- Step 1: Create a table.
CREATE TABLE sessions (id int PRIMARY KEY, user_id int, expires_at timestamptz);
INSERT INTO sessions VALUES (1, 42, now() - interval '1 day'), (2, 43, now() + interval '1 day');

-- Step 2: Delete expired sessions and return them.
DELETE FROM sessions
WHERE expires_at < now()
RETURNING id, user_id;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 2
DELETE 1
 id | user_id
----+---------
  1 |      42
(1 row)
```

**Why this output appears**: The `RETURNING id, user_id` clause returns the `id` and `user_id` of the deleted row, confirming which session was removed.

#### Real-World Cases

- **Session cleanup**: Expired sessions are deleted and the deleted IDs are returned to the application for cache invalidation.
- **Audit trail**: Deleted rows are returned and inserted into an audit table in the same transaction.

---

### 7. Deleting All Rows

#### Definitions

**Core Definition**
Deleting all rows removes every row from a table without removing the table structure, indexes, or constraints.

**Technical Definition**
`DELETE FROM table_name` without a `WHERE` clause removes all rows. This is functionally equivalent to `TRUNCATE TABLE` in terms of the end state (an empty table), but `DELETE` is slower because it scans and locks each row, fires triggers, and writes each deletion to the transaction log. MySQL explicitly notes that `TRUNCATE TABLE` is a faster way to empty a table than a `DELETE` with no `WHERE` clause.

**Beginner-Friendly Explanation**
Deleting all rows is like emptying a spreadsheet completely, but keeping the column headers and rules. `DELETE FROM table_name` does this row by row; `TRUNCATE TABLE` does it in one quick sweep.

#### Purposes

- To completely empty a table while preserving its structure and constraints.
- To reset a table for a fresh load or a new test run.
- To remove all data when the table's schema must remain intact.

#### Syntax Rules and Structure

**DELETE all rows:**

```sql
DELETE FROM table_name;
```

**TRUNCATE (faster alternative):**

```sql
TRUNCATE TABLE table_name;
```

**Syntax rules:**

- `DELETE FROM table_name` without `WHERE` deletes all rows but does not reset auto-increment counters (in MySQL, it does not reset `AUTO_INCREMENT`; in PostgreSQL, it does not reset sequences unless `RESTART IDENTITY` is used).
- `TRUNCATE` cannot be used with a `WHERE` clause.

**Constraints and limitations:**

- `DELETE` all rows can be very slow on large tables and holds row locks for the duration.
- `TRUNCATE` locks the entire table, blocking concurrent reads and writes.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – DELETE all rows vs. TRUNCATE

```sql
-- Step 1: Create and populate a table.
CREATE TABLE staging (id serial PRIMARY KEY, data text);
INSERT INTO staging (data) SELECT md5(random()::text) FROM generate_series(1, 10000);

-- Step 2: Delete all rows with DELETE.
DELETE FROM staging;

-- Step 3: Verify the count.
SELECT count(*) FROM staging;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 10000
DELETE 10000
 count
-------
     0
(1 row)
```

**Why this output appears**: `DELETE FROM staging` removes all 10,000 rows, one by one, and `DELETE 10000` confirms the count. The table structure remains intact.

#### Real-World Cases

- **Test reset**: A test suite runs `DELETE FROM table_name` or `TRUNCATE TABLE` between tests to ensure a clean state.
- **Staging refresh**: A staging table is emptied before loading a fresh batch of data from production.

---

### 8. `TRUNCATE`

#### Definitions

**Core Definition**
`TRUNCATE TABLE` is a DDL statement that quickly removes all rows from a table, optionally resetting identity sequences, while preserving the table's structure.

**Technical Definition**
`TRUNCATE` is a DDL command that deallocates the data pages of a table without scanning individual rows, making it significantly faster than `DELETE` for emptying a table. It locks the entire table, does not fire `DELETE` triggers, and does not return the number of rows deleted. In MySQL, `TRUNCATE` cannot be rolled back; in PostgreSQL and SQL Server, it can be rolled back within a transaction. `TRUNCATE` resets `AUTO_INCREMENT` in MySQL and identity columns in SQL Server; in PostgreSQL, `RESTART IDENTITY` must be specified to reset sequences.

**Beginner-Friendly Explanation**
`TRUNCATE` is like using a giant eraser to wipe a table clean instantly. It does not bother to remove rows one by one; it simply marks the table's storage as empty. It is much faster than `DELETE` for wiping a whole table, but it locks the table while it works and cannot be used with a `WHERE` clause.

#### Purposes

- To quickly empty a table and reset its auto-increment counter.
- To reclaim storage from a table without dropping it.
- To reset a table between test runs or ETL batches.

#### Syntax Rules and Structure

**General syntax:**

```sql
TRUNCATE TABLE table_name;
```

**PostgreSQL with identity reset:**

```sql
TRUNCATE TABLE table_name RESTART IDENTITY;
```

**Syntax rules:**

- `TRUNCATE` cannot be used with a `WHERE` clause; it always removes all rows.
- In MySQL, `TRUNCATE` causes an implicit commit and cannot be rolled back.
- In PostgreSQL, `TRUNCATE` can be rolled back if executed inside a transaction.

**Constraints and limitations:**

- `TRUNCATE` requires the `TRUNCATE` privilege (PostgreSQL) or `DROP` privilege (MySQL) or `ALTER` permission (SQL Server).
- `TRUNCATE` cannot be used on a table that is referenced by a foreign key from another table unless the referencing table is also truncated or the foreign key is dropped.
- `TRUNCATE` does not fire `DELETE` triggers.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – TRUNCATE resets AUTO_INCREMENT

```sql
-- Step 1: Create a table and insert rows.
CREATE TABLE items (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(100));
INSERT INTO items (name) VALUES ('a'), ('b'), ('c');

-- Step 2: Truncate the table.
TRUNCATE TABLE items;

-- Step 3: Insert a new row; the ID restarts at 1.
INSERT INTO items (name) VALUES ('new');
SELECT * FROM items;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 3 rows affected (0.01 sec)
Query OK, 0 rows affected (0.01 sec)
Query OK, 1 row affected (0.00 sec)
+----+------+
| id | name |
+----+------+
|  1 | new  |
+----+------+
```

**Why this output appears**: `TRUNCATE` removes all rows and resets the `AUTO_INCREMENT` counter to its starting value (1). The next insert receives `id = 1`.

#### Real-World Cases

- **Nightly ETL**: A staging table is truncated each night before loading the day's data.
- **Test isolation**: A test framework truncates tables between tests to guarantee a clean slate.

---

### 9. `TRUNCATE` vs `DELETE`

#### Definitions

**Core Definition**
`TRUNCATE` and `DELETE` both remove rows from a table, but they differ in speed, locking, transactionality, trigger behaviour, and side effects.

**Technical Definition**
`DELETE` is a DML operation that removes rows one by one, fires triggers, is transactional (rollback-able in all major RDBMSs except where DDL is implicitly committed), and supports `WHERE`. `TRUNCATE` is a DDL operation that removes all rows by deallocating pages, does not fire `DELETE` triggers, locks the whole table, and is not rollback-able in MySQL and Oracle but is in PostgreSQL and SQL Server. `TRUNCATE` is faster and uses fewer transaction log resources than `DELETE`.

**Beginner-Friendly Explanation**
`DELETE` is precise and careful: it removes the rows you specify, one by one, and can be undone. `TRUNCATE` is fast and blunt: it wipes the whole table at once, usually cannot be undone, and does not bother with triggers. Use `DELETE` when you need to remove specific rows; use `TRUNCATE` when you want to empty the whole table quickly.

#### Purposes

- To choose the right tool for removing data based on speed, safety, and side effects.
- To understand the trade-offs between row-level precision and table-level speed.
- To avoid accidentally using `TRUNCATE` when rollback or triggers are required.

#### Syntax Rules and Structure

| Aspect | `DELETE` | `TRUNCATE` |
|--------|----------|------------|
| Type | DML | DDL |
| `WHERE` clause | Yes | No |
| Speed | Slower (row by row) | Faster (page deallocation) |
| Locking | Row-level locks | Table-level lock |
| Rollback | Yes (all major RDBMSs) | MySQL: No; PostgreSQL/SQL Server: Yes |
| Triggers | Fires `DELETE` triggers | Does not fire `DELETE` triggers |
| Auto-increment | Not reset | Reset (MySQL, SQL Server); PostgreSQL: optional |
| Transaction log | Logs each row | Minimal logging |

**Syntax rules:**

- `DELETE` supports `WHERE`, `ORDER BY`, and `LIMIT` (MySQL single-table); `TRUNCATE` supports none of these.
- `TRUNCATE` requires a higher privilege level than `DELETE` (e.g., `DROP` in MySQL).

**Constraints and limitations:**

- `TRUNCATE` cannot be used on a table referenced by a foreign key from another table unless the referencing table is also truncated or the foreign key is dropped.
- `DELETE` without `WHERE` is functionally equivalent to `TRUNCATE` in end state but is much slower and fires triggers.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – DELETE vs. TRUNCATE rollback behaviour

```sql
-- Step 1: Create a table and insert rows.
CREATE TABLE test_delete (id int, val text);
INSERT INTO test_delete VALUES (1, 'a'), (2, 'b');

-- Step 2: DELETE inside a transaction and roll back.
BEGIN;
DELETE FROM test_delete;
ROLLBACK;
SELECT count(*) FROM test_delete;

-- Step 3: TRUNCATE inside a transaction and roll back.
BEGIN;
TRUNCATE TABLE test_delete;
ROLLBACK;
SELECT count(*) FROM test_delete;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 2
BEGIN
DELETE 2
ROLLBACK
 count
-------
     2
(1 row)
BEGIN
TRUNCATE TABLE
ROLLBACK
 count
-------
     2
(1 row)
```

**Why this output appears**: In PostgreSQL, both `DELETE` and `TRUNCATE` are transactional; rolling back restores the rows in both cases. In MySQL, the `TRUNCATE` rollback would fail because `TRUNCATE` causes an implicit commit.

#### Real-World Cases

- **Audit-sensitive tables**: A table with an `AFTER DELETE` trigger that logs deletions must use `DELETE`, not `TRUNCATE`, because `TRUNCATE` bypasses triggers.
- **Large staging tables**: A nightly ETL job uses `TRUNCATE` to quickly empty a staging table, accepting that the operation cannot be rolled back.

---

### 10. `DROP` vs `TRUNCATE` vs `DELETE`

#### Definitions

**Core Definition**
`DROP`, `TRUNCATE`, and `DELETE` are three distinct operations for removing data or structures, ranging from removing specific rows (`DELETE`) to emptying a table (`TRUNCATE`) to removing the table entirely (`DROP`).

**Technical Definition**
`DELETE` removes rows (DML, row-level locks, rollback-able, fires triggers). `TRUNCATE` removes all rows and preserves the table structure (DDL, table lock, faster, may not be rollback-able). `DROP TABLE` removes the table definition, data, indexes, constraints, triggers, and privileges from the database (DDL, table lock, not rollback-able in MySQL, rollback-able in PostgreSQL and SQL Server). The choice depends on whether you need to remove specific rows, empty the table, or eliminate the table entirely.

**Beginner-Friendly Explanation**
Think of a table as a filing cabinet. `DELETE` removes selected files. `TRUNCATE` empties the entire cabinet but keeps the cabinet itself. `DROP` removes the cabinet and everything in it from the room.

#### Purposes

- To choose the correct operation for the desired scope of removal.
- To avoid accidentally dropping a table when only the data should be removed.
- To understand the reversibility and side effects of each operation.

#### Syntax Rules and Structure

| Operation | Removes | Structure preserved? | Rollback? | Triggers? |
|-----------|---------|----------------------|-----------|-----------|
| `DELETE` | Specific rows | Yes | Yes | Yes |
| `TRUNCATE` | All rows | Yes | Depends on RDBMS | No |
| `DROP TABLE` | Table + data + structure | No | Depends on RDBMS | No |

**Syntax rules:**

- `DELETE FROM t WHERE ...` removes selected rows.
- `TRUNCATE TABLE t` removes all rows.
- `DROP TABLE t` removes the table entirely.

**Constraints and limitations:**

- `DROP TABLE` fails if the table is referenced by a foreign key from another table (unless `CASCADE` is used in PostgreSQL).
- `TRUNCATE` cannot be used with `WHERE`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – DELETE, TRUNCATE, and DROP in sequence

```sql
-- Step 1: Create a table.
CREATE TABLE demo (id int PRIMARY KEY, val text);
INSERT INTO demo VALUES (1, 'a'), (2, 'b');

-- Step 2: DELETE one row.
DELETE FROM demo WHERE id = 1;

-- Step 3: TRUNCATE the table.
TRUNCATE TABLE demo;

-- Step 4: DROP the table.
DROP TABLE demo;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 2
DELETE 1
TRUNCATE TABLE
DROP TABLE
```

**Why this output appears**: `DELETE` removes one row, `TRUNCATE` removes all remaining rows and preserves the table, and `DROP` removes the table entirely. After `DROP`, the table no longer exists.

#### Real-World Cases

- **Retention policy**: Old rows are removed with `DELETE`; if the entire dataset is obsolete, `TRUNCATE` is used; if the table itself is no longer needed, `DROP` is used.
- **Decommissioning**: A legacy table is `DROP`ped after its data is archived, while a staging table is `TRUNCATE`d nightly.

---

### 11. Cascading Deletes

#### Definitions

**Core Definition**
A cascading delete automatically removes child rows when their parent row is deleted, as specified by the `ON DELETE CASCADE` action on a foreign-key constraint.

**Technical Definition**
When a foreign key is defined with `ON DELETE CASCADE`, deleting a row in the parent table automatically deletes all rows in the child table that reference it, maintaining referential integrity. Without the cascade action, a `DELETE` that would orphan child rows fails with a foreign-key violation (SQL Server rolls back the entire statement). Cascading deletes can be recursive if multiple levels of foreign keys have `ON DELETE CASCADE`.

**Beginner-Friendly Explanation**
A cascading delete is like a chain reaction. If you delete a customer, all of that customer's orders are automatically deleted too, so you never have orders pointing to a customer that no longer exists.

#### Purposes

- To maintain referential integrity by automatically removing orphaned child rows.
- To simplify application logic by letting the database handle dependency cleanup.
- To ensure that deleting a parent does not leave dangling references.

#### Syntax Rules and Structure

**Foreign key with cascade:**

```sql
CREATE TABLE child (
    ...
    parent_id int REFERENCES parent(id) ON DELETE CASCADE
);
```

**Syntax rules:**

- `ON DELETE CASCADE` is specified on the foreign-key constraint in the child table.
- Other `ON DELETE` actions include `NO ACTION` (default, error if child rows exist), `RESTRICT` (similar to `NO ACTION` but not deferrable in some systems), `SET NULL`, and `SET DEFAULT`.

**Constraints and limitations:**

- Cascading deletes can be slow and lock-intensive on large graphs of related tables.
- Multiple levels of `ON DELETE CASCADE` can delete far more data than intended; use with caution.
- In SQL Server, cascading deletes may conflict with other cascading paths, causing an error.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Cascading delete

```sql
-- Step 1: Create parent and child tables with ON DELETE CASCADE.
CREATE TABLE customers (id int PRIMARY KEY, name text);
CREATE TABLE orders (
    id int PRIMARY KEY,
    customer_id int REFERENCES customers(id) ON DELETE CASCADE
);

INSERT INTO customers VALUES (1, 'Acme');
INSERT INTO orders VALUES (100, 1), (101, 1);

-- Step 2: Delete the customer; orders are cascaded.
DELETE FROM customers WHERE id = 1;

-- Step 3: Verify that the orders are gone.
SELECT * FROM orders;
```

**Expected output:**

```text
CREATE TABLE
CREATE TABLE
INSERT 0 1
INSERT 0 2
DELETE 1
 id | customer_id
----+-------------
(0 rows)
```

**Why this output appears**: The `ON DELETE CASCADE` clause automatically deletes the two `orders` rows when the `customers` row is deleted, leaving the `orders` table empty.

#### Real-World Cases

- **User account deletion**: Deleting a user cascades to their posts, comments, and sessions, ensuring no orphaned data remains.
- **Order cancellation**: Deleting an order cascades to its line items, keeping the order history consistent.

---

### 12. Delete Performance

#### Definitions

**Core Definition**
Delete performance is the measure of how efficiently a `DELETE` statement executes, influenced by the number of rows, locking, transaction log volume, index maintenance, and batching strategy.

**Technical Definition**
`DELETE` performance is dominated by several factors: (1) **rows affected**—deleting many rows in one transaction holds locks for a long time and generates large transaction logs; (2) **locking**—`DELETE` acquires row-level locks, which can block concurrent writes and, in some systems, escalate to table locks; (3) **triggers**—every deleted row fires applicable triggers, adding overhead; (4) **index maintenance**—each deletion updates all indexes on the table; and (5) **WAL/redo volume**—large deletes generate significant write-ahead log traffic, causing replication lag and I/O pressure. The universal mitigation is **batching**: deleting rows in small chunks (typically 1,000–10,000 rows) with a commit after each batch, optionally with a short sleep between batches to reduce I/O demand. MySQL provides `LIMIT row_count` for single-table deletes to cap rows per statement; SQL Server uses `DELETE TOP (N)` in a `WHILE` loop; PostgreSQL uses a subquery with `LIMIT` or a CTE to select the batch of IDs to delete.

**Beginner-Friendly Explanation**
Deleting a million rows in one statement is slow and risky: it locks the table for a long time, fills the transaction log, and can cause replication to fall behind. Breaking the delete into batches of 1,000–10,000 rows, committing each batch, and pausing briefly between batches keeps the database responsive and safe.

#### Purposes

- To delete large volumes of data without causing lock contention or replication lag.
- To reduce transaction log growth and avoid long-running transactions.
- To maintain predictable performance for production workloads.

#### Syntax Rules and Structure

**PostgreSQL batch delete:**

```sql
DELETE FROM logs
WHERE id IN (
    SELECT id FROM logs
    WHERE created_at < now() - interval '90 days'
    LIMIT 5000
);
-- commit; repeat
```

**MySQL batch delete:**

```sql
DELETE FROM logs
WHERE created_at < NOW() - INTERVAL 90 DAY
LIMIT 1000;
-- repeat until affected rows = 0
```

**SQL Server batch delete:**

```sql
WHILE 1 = 1
BEGIN
    DELETE TOP (5000) FROM logs
    WHERE created_at < DATEADD(DAY, -90, GETDATE());
    IF @@ROWCOUNT < 5000 BREAK;
END
```

**Syntax rules:**

- Each batch should be its own transaction (or autocommitted) to release locks promptly.
- A brief sleep (50–200 ms) between batches reduces I/O pressure and allows autovacuum or other maintenance to run.
- The `WHERE` clause should use an indexed column to avoid scanning the entire table for each batch.

**Constraints and limitations:**

- `LIMIT` without an indexed `WHERE` clause can cause full-table scans and defeat the purpose of batching.
- In SQL Server, `TOP` rows are not ordered; without a key-based `WHERE` or `ORDER BY`, successive batches may miss rows or revisit the same rows.
- Dropping indexes before a large delete and recreating them afterwards can be faster but requires exclusive access to the table.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Batched delete with LIMIT

```sql
-- Step 1: Create a table with many rows.
CREATE TABLE logs (id INT PRIMARY KEY, created_at DATE);
INSERT INTO logs SELECT seq, '2023-01-01' FROM seq_1_to_10000;

-- Step 2: Delete in batches of 1000 (conceptual loop; run repeatedly).
DELETE FROM logs WHERE created_at < '2024-01-01' LIMIT 1000;

-- Step 3: Verify progress.
SELECT COUNT(*) AS remaining FROM logs WHERE created_at < '2024-01-01';
```

**Expected output (after one batch):**

```text
Query OK, 10000 rows affected (0.05 sec)
Query OK, 1000 rows affected (0.01 sec)
Rows matched: 1000  Changed: 1000  Warnings: 0
+-----------+
| remaining |
+-----------+
|      9000 |
+-----------+
```

**Why this output appears**: Each `DELETE ... LIMIT 1000` removes 1,000 rows and commits, releasing locks. Repeating the statement eventually removes all 10,000 rows without a single long-running transaction.

#### Real-World Cases

- **Time-series purge**: Old log rows are deleted in daily batches of 5,000 with a short sleep between batches to keep replication lag under control.
- **Data archival**: Rows older than the retention period are archived to a cold-storage table and then deleted in batches from the hot table.

---

### 13. Safe Deletion Strategies

#### Definitions

**Core Definition**
Safe deletion strategies are the practices and patterns used to prevent accidental or destructive `DELETE` operations, including mandatory `WHERE` clauses, pre-checks, batching, soft deletes, and database-level safeguards.

**Technical Definition**
Safe deletion strategies include: (1) **mandatory `WHERE` clause**—unconditional `DELETE` is prohibited in production guidelines; MySQL's `sql_safe_updates` and PostgreSQL's `pg-safeupdate` enforce this at the session level; (2) **pre-check the affected rows**—run a `SELECT COUNT(*)` with the same `WHERE` clause before executing the delete; (3) **batching**—limit the number of rows deleted per statement to control locking and replication impact; (4) **soft deletes**—add a `deleted_at` timestamp column and mark rows as deleted instead of physically removing them, allowing recovery and preserving foreign-key references; (5) **use transactions**—wrap the delete in a transaction and `ROLLBACK` to test before committing; (6) **set lock timeouts**—`lock_timeout` (PostgreSQL) or `innodb_lock_wait_timeout` (MySQL) to prevent indefinite blocking; and (7) **back up critical data** or use partitioning to make large deletes reversible by dropping partitions.

**Beginner-Friendly Explanation**
Safe deletion means never running a `DELETE` without a `WHERE` clause, checking how many rows will be affected before you commit, breaking large deletes into batches, and considering a “soft delete” that just marks rows as hidden instead of actually removing them.

#### Purposes

- To prevent accidental full-table deletes that destroy data.
- To limit the blast radius of a mistake.
- To keep production systems responsive during data removal.
- To retain the ability to recover deleted data.

#### Syntax Rules and Structure

**Pre-check pattern:**

```sql
-- Step 1: Check the affected rows.
SELECT COUNT(*) FROM table_name WHERE condition;

-- Step 2: Execute the delete.
DELETE FROM table_name WHERE condition;
```

**Soft delete pattern (PostgreSQL):**

```sql
ALTER TABLE orders ADD COLUMN deleted_at timestamptz;

-- "Delete" a row:
UPDATE orders SET deleted_at = now() WHERE id = 42;

-- Exclude soft-deleted rows:
CREATE VIEW active_orders AS SELECT * FROM orders WHERE deleted_at IS NULL;
```

**MySQL `sql_safe_updates`:**

```sql
SET SESSION sql_safe_updates = 1;
-- Now DELETE without a WHERE clause or LIMIT fails.
```

**Component breakdown:**

| Practice | Implementation |
|----------|----------------|
| Mandatory `WHERE` | `sql_safe_updates`, `pg-safeupdate`, code review |
| Pre-check row count | `SELECT COUNT(*)` with the same `WHERE` |
| Batching | `LIMIT` / `TOP` / subquery batches |
| Soft delete | `deleted_at` timestamp + views |
| Lock timeout | `SET lock_timeout = '3s'` (PostgreSQL) |
| Backup / PITR | Regular backups and point-in-time recovery |

**Syntax rules:**

- `sql_safe_updates` requires a `WHERE` clause that uses a key column or a `LIMIT` clause.
- `pg-safeupdate` prevents `DELETE` and `UPDATE` without a `WHERE` clause.
- Soft-deleted rows must be excluded from queries, typically via a view or a `WHERE deleted_at IS NULL` predicate.

**Constraints and limitations:**

- `sql_safe_updates` can be disabled per session; it is not a hard security boundary.
- Pre-checks are point-in-time; the actual delete may affect a different number of rows if the data changes between the check and the delete.
- Soft deletes increase storage usage and require a scheduled hard-delete job to purge old soft-deleted rows.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Soft delete and active view

```sql
-- Step 1: Add a deleted_at column to an existing table.
ALTER TABLE orders ADD COLUMN deleted_at timestamptz;

-- Step 2: "Delete" a row by setting deleted_at.
UPDATE orders SET deleted_at = now() WHERE id = 42;

-- Step 3: Create a view that excludes soft-deleted rows.
CREATE VIEW active_orders AS
SELECT * FROM orders WHERE deleted_at IS NULL;

-- Step 4: Verify that the soft-deleted row is hidden from the view.
SELECT COUNT(*) FROM active_orders;
SELECT COUNT(*) FROM orders WHERE deleted_at IS NOT NULL;
```

**Expected output:**

```text
ALTER TABLE
UPDATE 1
CREATE VIEW
 count
-------
   999
(1 row)
 count
-------
     1
(1 row)
```

**Why this output appears**: The soft-deleted row (`id = 42`) is excluded from `active_orders` but remains physically present in the `orders` table, allowing recovery or audit access.

#### Real-World Cases

- **Regulatory compliance**: Financial records are soft-deleted to retain an audit trail while hiding them from normal application views.
- **User-facing deletes**: A user “deletes” a post, but the row is retained with a `deleted_at` timestamp for moderation and recovery.
- **Large-scale cleanup**: Old data is soft-deleted first, then a scheduled job hard-deletes it in batches during low-traffic periods.

---

## References

- PostgreSQL Documentation – DELETE – https://www.postgresql.org/docs/17/sql-delete.html
- MySQL 8.0 Reference Manual – DELETE Statement – https://dev.mysql.com/doc/refman/26.7/en/delete.html
- Microsoft Learn – DELETE (Transact-SQL) – https://github.com/microsoftdocs/sql-docs/blob/4c7374e97347c3723cd42ea99fb540a4d09e1ae6/docs/t-sql/statements/delete-transact-sql.md
- LearnSQL – Comparing TRUNCATE TABLE, DELETE, and DROP TABLE in SQL – https://learnsql.com/blog/difference-between-truncate-delete-and-drop-table-in-sql/
- Supabase Docs – Deleting data and dropping objects safely – https://supabase.com/docs/guides/database/postgres/data-deletion
- Redgate – How to Monitor and Optimize Batched Deletion Processes in SQL Server – https://www.red-gate.com/blog/how-to-monitor-and-optimize-batched-deletion-processes-in-sql-server
- Infobip Developers Hub – Delete without DELETE: Smarter strategies for removing high-volume, short-lived data – https://www.infobip.com/developers/blog/delete-without-delete-smarter-strategies-for-removing-high-volume-short-lived-data
- PostgreSQL Mailing List – Need an idea to operate massive delete operation on big size table – https://www.postgresql.org/message-id/...
- GitHub – SQL Server batch delete patterns – https://github.com/...
- Oracle Documentation – ON DELETE CASCADE – https://docs.oracle.com/...
- Microsoft Learn – Cascading Referential Integrity Constraints – https://learn.microsoft.com/...