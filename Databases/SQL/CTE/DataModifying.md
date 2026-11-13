# Comprehensive Programming Cheat Sheet: Data-Modifying CTEs

## Topic Overview

### Definitions

**Core Definition**  
A Data-Modifying CTE is a Common Table Expression whose body contains an `INSERT`, `UPDATE`, `DELETE`, or `MERGE` statement, allowing data changes to be expressed as part of a `WITH` clause and, optionally, referenced by the main query.

**Technical Definition**  
Data-modifying CTEs are a PostgreSQL and SQL Server extension (SQL Server supports them only through `OUTPUT` with `INSERT/UPDATE/DELETE` in `WITH`, and only for `INSERT … OUTPUT` chains in specific forms; PostgreSQL supports full DML CTEs in `WITH`). In PostgreSQL, a CTE may contain `INSERT`, `UPDATE`, or `DELETE` with a `RETURNING` clause, and the main statement can read the returned rows. Execution is guaranteed to run each data-modifying CTE exactly once, even if the main query does not reference it, and all such CTEs execute to completion within a single snapshot. MySQL 8.0 does not support data-modifying CTEs; it supports `WITH` only before `SELECT`, `UPDATE`, and `DELETE` (as the outer statement, not inside the CTE body).

**Beginner-Friendly Explanation**  
A data-modifying CTE lets you put a data change (insert, update, or delete) inside the `WITH` clause, name its result, and then use the changed rows in the rest of the query—for example, “insert this row, give me back the new ID, and then use that ID to insert a related row.”

### Key Characteristics

- **Side-effecting CTEs**: Unlike ordinary CTEs, these change data.
- **Exactly-once semantics**: In PostgreSQL, each data-modifying CTE executes exactly once, regardless of whether the main query reads it.
- **Single snapshot**: All CTEs and the main query see the same snapshot of the database; modifications from earlier CTEs are visible to later CTEs via `RETURNING` data, but not via re-reading the base tables.
- **`RETURNING` required for reference**: A data-modifying CTE can be referenced by the main query only if it has a `RETURNING` clause.
- **No `SELECT`-only recursion**: Data-modifying CTEs cannot be recursive.
- **Engine-specific**: PostgreSQL is the primary engine with full support; SQL Server has limited support; MySQL 8.4 does not support DML inside CTEs.
- **Chaining**: Multiple DML CTEs can be chained, with later CTEs consuming `RETURNING` output from earlier ones.

### Prerequisites

- Understanding of ordinary CTEs and their scope.
- Knowledge of `INSERT`, `UPDATE`, `DELETE` syntax and `RETURNING`/`OUTPUT`.
- Familiarity with transaction semantics and isolation levels.
- Awareness of the target engine’s support matrix (PostgreSQL 9.1+ for DML CTEs; SQL Server for `OUTPUT`-based patterns).

### Related Programming Areas

- ETL and data migration.
- Upsert patterns (`INSERT … ON CONFLICT`, `MERGE`).
- Audit logging and change data capture.
- Multi-table transactional writes.
- Application data access layers and stored procedures.

---

## Core Concepts and Features

### 1. `INSERT` in CTE

#### Definitions

**Core Definition**  
An `INSERT` in a CTE is an `INSERT` statement placed inside a `WITH` clause, optionally with a `RETURNING` clause whose output the main query can read.

**Technical Definition**  
In PostgreSQL, `WITH ins AS (INSERT INTO … RETURNING …) SELECT … FROM ins …` executes the insert exactly once, and the main query reads the returned rows. The insert is visible to the CTE’s `RETURNING` output and to later CTEs via that output, but the base table changes are not visible to other parts of the same statement’s snapshot. The `INSERT` must include `RETURNING` if the main query references the CTE; if the CTE is not referenced, it still executes (for side effects), and `RETURNING` is optional.

**Beginner-Friendly Explanation**  
You can write “insert this data, and hand me back what was inserted” as a helper query, and then use those returned rows in the rest of the statement.

#### Purposes

- To combine an insert and a dependent read in one statement.
- To feed generated keys (e.g., `SERIAL`, `IDENTITY`) into subsequent inserts or queries.
- To ensure an insert happens exactly once as part of a larger statement.
- To reduce round trips between the application and the database.

#### Syntax Rules and Structure

**PostgreSQL**:

```sql
WITH ins AS (
    INSERT INTO target_table (col1, col2, ...)
    VALUES (...), (...)
    RETURNING id, col1, ...
)
SELECT ... FROM ins ...;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `WITH ins AS (...)` | CTE name and body. |
| `INSERT INTO target_table (...)` | Target table and columns. |
| `VALUES (...)` or `SELECT ...` | Rows to insert. |
| `RETURNING ...` | Columns returned to the outer query. |
| `SELECT ... FROM ins` | Main query consuming the inserted rows. |

**Syntax rules**

- `RETURNING` is required if the main query references the CTE.
- The CTE executes exactly once, even if the main query does not reference it.
- Data-modifying CTEs cannot be recursive.
- In PostgreSQL, all data-modifying CTEs and the main query share one snapshot; changes from one CTE are visible to later CTEs only via `RETURNING`.
- The `INSERT` must be a top-level statement inside the parentheses; nested `WITH` inside a DML CTE is allowed in recent PostgreSQL versions.

**Constraints and limitations**

- MySQL 8.4 does not support `INSERT` inside a CTE body.
- SQL Server supports `INSERT` only via `OUTPUT` in specific `WITH` patterns, not full DML CTEs.
- `RETURNING` output cannot be used in a `WHERE` clause of the same CTE; it is available to the outer statement and later CTEs.
- Triggers on the target table still fire normally.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Insert and read back the inserted rows

```sql
-- Step 1: Create a target table.
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE
);

-- Step 2: Insert via a CTE and read the returned rows.
WITH ins AS (
    INSERT INTO customers (name, email)
    VALUES ('Alice', 'alice@example.com'),
           ('Bob',   'bob@example.com')
    RETURNING id, name, email
)
SELECT id, name, email FROM ins ORDER BY id;
```

**Expected output**:

```text
 id | name  |      email
----+-------+------------------
  1 | Alice | alice@example.com
  2 | Bob   | bob@example.com
(2 rows)
```

**Why this output appears**: The `INSERT` executes once and returns the generated IDs and inserted values via `RETURNING`; the main query reads them from the CTE.

##### Example 2: PostgreSQL – Insert and immediately aggregate

```sql
WITH ins AS (
    INSERT INTO orders (customer_id, amount)
    VALUES (1, 100), (1, 200), (2, 50)
    RETURNING customer_id, amount
)
SELECT customer_id, COUNT(*) AS order_count, SUM(amount) AS total
FROM ins
GROUP BY customer_id
ORDER BY customer_id;
```

**Expected output**:

```text
 customer_id | order_count | total
-------------+-------------+-------
           1 |           2 |   300
           2 |           1 |    50
(2 rows)
```

**Why this output appears**: The CTE returns the inserted rows, and the main query aggregates them without re-reading the `orders` table.

##### Example 3: PostgreSQL – Insert without referencing the CTE (side effect only)

```sql
-- The insert runs even though the main query does not reference the CTE.
WITH ins AS (
    INSERT INTO audit_log (event) VALUES ('startup')
    RETURNING id
)
SELECT 1 AS ok;
```

**Expected output**:

```text
 ok
----
  1
(1 row)
```

**Why this output appears**: PostgreSQL executes data-modifying CTEs to completion even if unreferenced; the `INSERT` still runs, and the main query returns `1`.

#### Real-World Cases

- **Feeding generated keys**: Insert a parent row and use its `RETURNING id` to insert children.
- **Bulk staging**: Insert into a staging table and aggregate immediately.
- **Audit logging**: Insert an event and return its ID for the main query.

---

### 2. `UPDATE` in CTE

#### Definitions

**Core Definition**  
An `UPDATE` in a CTE is an `UPDATE` statement placed inside a `WITH` clause, optionally returning the updated rows for use by the main query.

**Technical Definition**  
In PostgreSQL, `WITH upd AS (UPDATE … RETURNING …) SELECT … FROM upd …` executes the update exactly once. The `RETURNING` clause exposes the post-update row values (unless `RETURNING` refers to `OLD`, which PostgreSQL does not support in the same way as some engines; use `RETURNING` for new values). The update is visible to the CTE’s output and to later CTEs via that output, but not to other parts of the same statement via re-reads.

**Beginner-Friendly Explanation**  
You can update rows as a helper query, get the updated rows back, and then use them in the rest of the statement.

#### Purposes

- To update rows and immediately use the updated values.
- To perform a read-modify-write in one statement.
- To feed updated keys or statuses into subsequent inserts or deletes.
- To reduce application round trips.

#### Syntax Rules and Structure

**PostgreSQL**:

```sql
WITH upd AS (
    UPDATE target_table
    SET col1 = expr1, col2 = expr2
    WHERE condition
    RETURNING id, col1, ...
)
SELECT ... FROM upd ...;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `WITH upd AS (...)` | CTE name and body. |
| `UPDATE target_table` | Target table. |
| `SET col = expr` | Assignments. |
| `WHERE condition` | Rows to update. |
| `RETURNING ...` | Updated values returned to the outer query. |
| `SELECT ... FROM upd` | Main query consuming updated rows. |

**Syntax rules**

- `RETURNING` is required if the main query references the CTE.
- The CTE executes exactly once.
- Data-modifying CTEs cannot be recursive.
- In PostgreSQL, `RETURNING` shows the new values; to see old values, use a separate CTE that captures them before the update (if the engine supports it).
- Update triggers fire normally.

**Constraints and limitations**

- MySQL 8.4 does not support `UPDATE` inside a CTE body.
- SQL Server supports `UPDATE … OUTPUT` in limited `WITH` patterns.
- A CTE cannot update a table and then read the same table’s pre-update state in the same snapshot.
- Concurrency: `UPDATE` CTEs take row locks; long-running CTEs hold locks for the duration of the statement.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Update and return updated rows

```sql
-- Step 1: Create a table.
CREATE TABLE accounts (id INT PRIMARY KEY, balance NUMERIC);

INSERT INTO accounts VALUES (1, 100), (2, 200);

-- Step 2: Update balances and return the new values.
WITH upd AS (
    UPDATE accounts
    SET balance = balance + 50
    WHERE id IN (1, 2)
    RETURNING id, balance
)
SELECT id, balance FROM upd ORDER BY id;
```

**Expected output**:

```text
 id | balance
----+---------
  1 |     150
  2 |     250
(2 rows)
```

**Why this output appears**: The `UPDATE` adds 50 to each balance and returns the new values; the main query reads them.

##### Example 2: PostgreSQL – Read-modify-write in one statement

```sql
WITH upd AS (
    UPDATE inventory
    SET quantity = quantity - 1
    WHERE product_id = 42 AND quantity > 0
    RETURNING product_id, quantity
)
SELECT product_id, quantity FROM upd;
```

**Expected output**:

```text
 product_id | quantity
------------+----------
         42 |        9
(1 row)
```

**Why this output appears**: The `UPDATE` decrements the quantity only if it is positive, and `RETURNING` exposes the new quantity.

##### Example 3: PostgreSQL – Update and feed results into an insert

```sql
WITH upd AS (
    UPDATE orders
    SET status = 'shipped'
    WHERE status = 'pending'
    RETURNING order_id, customer_id
),
ins AS (
    INSERT INTO shipment_log (order_id, shipped_at)
    SELECT order_id, now() FROM upd
    RETURNING order_id
)
SELECT COUNT(*) AS shipped_count FROM ins;
```

**Expected output**:

```text
 shipped_count
---------------
             3
(1 row)
```

**Why this output appears**: The first CTE updates orders and returns their IDs; the second CTE inserts shipment log entries for those orders; the main query counts them.

#### Real-World Cases

- **Inventory decrement**: Update stock and return the new level for confirmation.
- **Order state transitions**: Mark orders shipped and log the shipment in the same statement.
- **Balance adjustments**: Update account balances and return the new values for reporting.

---

### 3. `DELETE` in CTE

#### Definitions

**Core Definition**  
A `DELETE` in a CTE is a `DELETE` statement placed inside a `WITH` clause, optionally returning the deleted rows for use by the main query.

**Technical Definition**  
In PostgreSQL, `WITH del AS (DELETE … RETURNING …) SELECT … FROM del …` executes the delete exactly once. `RETURNING` exposes the deleted rows (their values before deletion). Deleted rows are visible to the CTE’s output and to later CTEs via that output, but not to re-reads of the base table within the same statement snapshot.

**Beginner-Friendly Explanation**  
You can delete rows as a helper query, get the deleted rows back, and then use them—for example, archive them before they disappear.

#### Purposes

- To archive deleted rows in the same statement.
- To delete and immediately report or log what was removed.
- To cascade deletes through dependent tables using returned keys.
- To reduce application round trips for delete‑and‑archive workflows.

#### Syntax Rules and Structure

**PostgreSQL**:

```sql
WITH del AS (
    DELETE FROM target_table
    WHERE condition
    RETURNING id, col1, ...
)
SELECT ... FROM del ...;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `WITH del AS (...)` | CTE name and body. |
| `DELETE FROM target_table` | Target table. |
| `WHERE condition` | Rows to delete. |
| `RETURNING ...` | Deleted row values. |
| `SELECT ... FROM del` | Main query consuming deleted rows. |

**Syntax rules**

- `RETURNING` is required if the main query references the CTE.
- The CTE executes exactly once.
- Data-modifying CTEs cannot be recursive.
- `RETURNING` shows the deleted rows’ values (pre-delete).
- Delete triggers fire normally.

**Constraints and limitations**

- MySQL 8.4 does not support `DELETE` inside a CTE body.
- SQL Server supports `DELETE … OUTPUT` in limited `WITH` patterns.
- Once deleted, rows cannot be re-read from the base table in the same statement snapshot; only the `RETURNING` output is available.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Delete and archive in one statement

```sql
-- Step 1: Create tables.
CREATE TABLE sessions (id INT PRIMARY KEY, user_id INT, created_at TIMESTAMP);
CREATE TABLE session_archive (id INT, user_id INT, created_at TIMESTAMP);

INSERT INTO sessions VALUES (1, 10, now() - interval '30 days'),
                             (2, 11, now() - interval '10 days');

-- Step 2: Delete old sessions and archive them.
WITH del AS (
    DELETE FROM sessions
    WHERE created_at < now() - interval '20 days'
    RETURNING id, user_id, created_at
),
ins AS (
    INSERT INTO session_archive (id, user_id, created_at)
    SELECT id, user_id, created_at FROM del
    RETURNING id
)
SELECT COUNT(*) AS archived_count FROM ins;
```

**Expected output**:

```text
 archived_count
----------------
              1
(1 row)
```

**Why this output appears**: The `DELETE` removes the old session and returns it; the second CTE inserts it into the archive; the main query counts archived rows.

##### Example 2: PostgreSQL – Delete and report deleted rows

```sql
WITH del AS (
    DELETE FROM orders
    WHERE status = 'cancelled'
    RETURNING order_id, customer_id, amount
)
SELECT customer_id, COUNT(*) AS cancelled_count, SUM(amount) AS total_cancelled
FROM del
GROUP BY customer_id;
```

**Expected output**:

```text
 customer_id | cancelled_count | total_cancelled
-------------+-----------------+-----------------
         104 |               2 |          150.00
         112 |               1 |           75.00
(2 rows)
```

**Why this output appears**: The `DELETE` returns the cancelled orders, and the main query aggregates them by customer.

##### Example 3: PostgreSQL – Delete without referencing the CTE

```sql
WITH del AS (
    DELETE FROM temp_data WHERE created_at < now() - interval '7 days'
    RETURNING id
)
SELECT 1 AS ok;
```

**Expected output**:

```text
 ok
----
  1
(1 row)
```

**Why this output appears**: The `DELETE` executes exactly once even though the main query does not reference the CTE.

#### Real-World Cases

- **Session cleanup**: Delete expired sessions and archive them.
- **Order cancellation**: Delete cancelled orders and report totals.
- **Data retention**: Purge old records and log the purge.

---

### 4. `RETURNING` Clause

#### Definitions

**Core Definition**  
The `RETURNING` clause specifies which columns a data-modifying statement returns to the caller, making the modified rows available to the outer query or application.

**Technical Definition**  
`RETURNING` is a PostgreSQL extension (also supported by SQLite 3.35+, and by SQL Server as `OUTPUT`) that appends a projection to `INSERT`, `UPDATE`, `DELETE`, or `MERGE`. In a data-modifying CTE, `RETURNING` defines the CTE’s output columns. Without `RETURNING`, the CTE produces no rows and cannot be referenced by the main query (though it still executes).

**Beginner-Friendly Explanation**  
`RETURNING` is how you say “after changing the data, give me back these columns” so the rest of the query can use them.

#### Purposes

- To expose modified rows to the outer query.
- To obtain generated keys from `INSERT`.
- To capture pre-delete or post-update values.
- To chain data modifications by feeding returned values into later CTEs.

#### Syntax Rules and Structure

**Syntax**:

```sql
INSERT INTO t (...) VALUES (...) RETURNING col1, col2, ...;
UPDATE t SET ... WHERE ... RETURNING col1, col2, ...;
DELETE FROM t WHERE ... RETURNING col1, col2, ...;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `RETURNING` | Keyword introducing the projection. |
| `col1, col2, ...` | Columns or expressions to return. |
| `*` | All columns (use with care). |

**Syntax rules**

- `RETURNING` may use column names, expressions, and aliases.
- In PostgreSQL, `RETURNING` on `UPDATE` shows new values; on `DELETE` shows deleted rows.
- `RETURNING` output can be referenced by the main query and later CTEs.
- `RETURNING` is not allowed on `SELECT` (it belongs to DML statements).

**Constraints and limitations**

- `RETURNING` on `UPDATE` cannot reference the old row values directly in PostgreSQL without workarounds (e.g., a pre‑update CTE).
- Large `RETURNING` result sets consume memory in the CTE.
- MySQL 8.4 does not support `RETURNING` for `INSERT`/`UPDATE`/`DELETE` in the same way (it has `RETURNING` only for some statements in MariaDB; MySQL itself lacks it for DML).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – `RETURNING` with expressions

```sql
WITH ins AS (
    INSERT INTO products (name, price)
    VALUES ('Widget', 19.99)
    RETURNING id, name, price, price * 1.1 AS price_with_tax
)
SELECT * FROM ins;
```

**Expected output**:

```text
 id | name   | price | price_with_tax
----+--------+-------+----------------
  1 | Widget | 19.99 |          21.99
(1 row)
```

**Why this output appears**: `RETURNING` allows expressions; the main query reads the computed `price_with_tax` alongside the inserted values.

##### Example 2: PostgreSQL – `RETURNING` on `UPDATE`

```sql
WITH upd AS (
    UPDATE accounts
    SET balance = balance * 1.05
    WHERE id = 1
    RETURNING id, balance
)
SELECT * FROM upd;
```

**Expected output**:

```text
 id | balance
----+---------
  1 |  105.00
(1 row)
```

**Why this output appears**: `RETURNING` shows the post-update balance.

##### Example 3: SQL Server – `OUTPUT` as the equivalent of `RETURNING`

```sql
INSERT INTO Customers (Name, Email)
OUTPUT inserted.Id, inserted.Name, inserted.Email
VALUES ('Alice', 'alice@example.com');
```

**Expected output**:

```text
Id Name  Email
-- ----- ------------------
1  Alice alice@example.com
```

**Why this output appears**: SQL Server uses `OUTPUT inserted.*` to return inserted values, analogous to PostgreSQL `RETURNING`.

#### Real-World Cases

- **Generated keys**: Return `SERIAL`/`IDENTITY` values after insert.
- **Change capture**: Return updated rows for logging.
- **Chaining**: Feed returned keys into dependent inserts.

---

### 5. Chained Data Modifications

#### Definitions

**Core Definition**  
Chained data modifications are multiple data-modifying CTEs in one `WITH` clause, where later CTEs consume the `RETURNING` output of earlier ones.

**Technical Definition**  
In PostgreSQL, multiple DML CTEs execute within the same statement and snapshot. Each CTE’s `RETURNING` output is available to later CTEs and the main query. Execution order is not guaranteed except that a CTE cannot reference a later CTE; the engine ensures that referenced outputs are available. All CTEs execute to completion, and the statement commits atomically.

**Beginner-Friendly Explanation**  
You can chain changes: insert a row, use its new ID to insert a related row, then use that to update a third table—all in one statement.

#### Purposes

- To perform multi-table writes atomically in one statement.
- To propagate generated keys through dependent tables.
- To implement archive-then-delete, insert-then-log, and similar workflows.
- To reduce application round trips.

#### Syntax Rules and Structure

**Pattern**:

```sql
WITH
  step1 AS (INSERT ... RETURNING id, ...),
  step2 AS (INSERT ... SELECT ... FROM step1 RETURNING ...),
  step3 AS (UPDATE ... FROM step2 RETURNING ...)
SELECT ... FROM step3;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `step1` | First modification with `RETURNING`. |
| `step2` | Consumes `step1`’s output. |
| `step3` | Consumes `step2`’s output. |
| `SELECT` | Final read of the chain. |

**Syntax rules**

- Each CTE must have `RETURNING` if referenced by a later CTE or the main query.
- Declaration order defines dependency order.
- All CTEs share one snapshot; they cannot see each other’s changes via base‑table reads, only via `RETURNING`.
- The statement is atomic: either all CTEs’ changes commit or none.

**Constraints and limitations**

- Execution order among independent CTEs is unspecified.
- A CTE cannot see changes made by a sibling CTE via base‑table re-reads.
- Long chains increase lock duration.
- MySQL 8.4 does not support this pattern.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Insert parent, insert child, update audit

```sql
WITH
  new_customer AS (
      INSERT INTO customers (name) VALUES ('Carol')
      RETURNING id
  ),
  new_order AS (
      INSERT INTO orders (customer_id, amount)
      SELECT id, 99.99 FROM new_customer
      RETURNING order_id, customer_id
  ),
  audit AS (
      UPDATE customer_stats
      SET order_count = order_count + 1
      WHERE customer_id IN (SELECT customer_id FROM new_order)
      RETURNING customer_id
  )
SELECT * FROM new_order;
```

**Expected output**:

```text
 order_id | customer_id
----------+-------------
        1 |           1
(1 row)
```

**Why this output appears**: `new_customer` returns the generated customer ID; `new_order` uses it; `audit` updates stats for the same customer. The main query returns the new order.

##### Example 2: PostgreSQL – Archive then delete chain

```sql
WITH
  del AS (
      DELETE FROM sessions
      WHERE created_at < now() - interval '30 days'
      RETURNING id, user_id, created_at
  ),
  archive AS (
      INSERT INTO session_archive (id, user_id, created_at)
      SELECT id, user_id, created_at FROM del
      RETURNING id
  )
SELECT COUNT(*) AS archived FROM archive;
```

**Expected output**:

```text
 archived
----------
        5
(1 row)
```

**Why this output appears**: The delete returns deleted rows; the insert archives them; the main query counts archived rows.

##### Example 3: PostgreSQL – Chain with update and insert

```sql
WITH
  upd AS (
      UPDATE inventory
      SET quantity = quantity - 1
      WHERE product_id = 42 AND quantity > 0
      RETURNING product_id, quantity
  ),
  log AS (
      INSERT INTO inventory_log (product_id, new_quantity)
      SELECT product_id, quantity FROM upd
      RETURNING product_id
  )
SELECT * FROM log;
```

**Expected output**:

```text
 product_id
------------
         42
(1 row)
```

**Why this output appears**: The update decrements stock and returns the new quantity; the log insert consumes it; the main query returns the logged product.

#### Real-World Cases

- **Order processing**: Insert order, insert order lines, update customer stats.
- **Data retention**: Delete old rows, archive them, log the purge.
- **Inventory**: Decrement stock, log the change, update reorder flags.

---

### 6. Use Cases

#### Definitions

**Core Definition**  
Use cases for data-modifying CTEs are the practical scenarios where combining data changes with reads in one statement is beneficial.

**Technical Definition**  
Data-modifying CTEs are used for atomic multi-step writes, generated-key propagation, read-modify-write patterns, archive-then-delete, upsert-like workflows, and audit logging—where the statement must be atomic and the application benefits from reduced round trips.

**Beginner-Friendly Explanation**  
They are useful whenever you need to change data and immediately use the result of that change in the same statement.

#### Purposes

- To implement atomic multi-table writes.
- To propagate generated keys to dependent tables.
- To implement archive-and-delete in one statement.
- To implement read-modify-write with immediate feedback.
- To reduce application round trips and race conditions.

#### Syntax Rules and Structure

**Use-case patterns**:

```sql
-- Generated-key propagation
WITH p AS (INSERT INTO parent ... RETURNING id)
INSERT INTO child (parent_id) SELECT id FROM p;

-- Archive then delete
WITH d AS (DELETE FROM t WHERE ... RETURNING *)
INSERT INTO archive SELECT * FROM d;

-- Read-modify-write
WITH u AS (UPDATE t SET x = x + 1 WHERE ... RETURNING *)
SELECT * FROM u;
```

**Component breakdown**

| Use case | Pattern |
|----------|---------|
| Key propagation | Parent insert returning ID → child insert. |
| Archive/delete | Delete returning rows → archive insert. |
| Read-modify-write | Update returning rows → main query. |
| Audit logging | Insert/update/delete returning keys → audit insert. |

**Syntax rules**

- Each modification must use `RETURNING` if its output is consumed.
- Chained CTEs must respect declaration order.
- All modifications occur in one transaction/snapshot.

**Constraints and limitations**

- Not supported in MySQL 8.4.
- SQL Server supports limited patterns via `OUTPUT`.
- Triggers and constraints still apply.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Upsert-like pattern

```sql
WITH ins AS (
    INSERT INTO settings (key, value)
    VALUES ('theme', 'dark')
    ON CONFLICT (key) DO UPDATE SET value = EXCLUDED.value
    RETURNING key, value, (xmax = 0) AS inserted
)
SELECT key, value, inserted FROM ins;
```

**Expected output**:

```text
  key  | value | inserted
-------+-------+----------
 theme | dark  | t
(1 row)
```

**Why this output appears**: `ON CONFLICT DO UPDATE` performs an upsert; `xmax = 0` distinguishes insert from update.

##### Example 2: PostgreSQL – Audit logging

```sql
WITH del AS (
    DELETE FROM orders WHERE status = 'cancelled' RETURNING order_id, customer_id
),
audit AS (
    INSERT INTO order_audit (order_id, action, at)
    SELECT order_id, 'deleted', now() FROM del
    RETURNING order_id
)
SELECT COUNT(*) AS audited FROM audit;
```

**Expected output**:

```text
 audited
---------
       3
(1 row)
```

**Why this output appears**: Deleted orders are logged in the same statement.

##### Example 3: PostgreSQL – Multi-tenant data move

```sql
WITH moved AS (
    DELETE FROM tenant_a.items WHERE category = 'archive' RETURNING *
)
INSERT INTO tenant_b.items SELECT * FROM moved;
```

**Expected output**:

```text
INSERT 0 12
```

**Why this output appears**: Rows are moved from one tenant table to another in one statement.

#### Real-World Cases

- **Order processing**: Atomic order + lines + stats update.
- **Data retention**: Purge and archive in one statement.
- **Audit trails**: Log every change in the same statement.

---

### 7. Pitfalls

#### Definitions

**Core Definition**  
Pitfalls of data-modifying CTEs are common mistakes and hazards that lead to unexpected results, performance problems, or semantic errors.

**Technical Definition**  
Pitfalls include assuming execution order among independent CTEs, expecting base‑table re‑reads to see sibling CTE changes, forgetting that unreferenced DML CTEs still execute, misusing `RETURNING` semantics (new vs old values), overlooking lock duration, and relying on features unsupported by the target engine.

**Beginner-Friendly Explanation**  
Common mistakes include thinking the database runs your helper changes in a particular order, or assuming that reading a table again will show changes made earlier in the same statement.

#### Purposes

- To avoid incorrect assumptions about execution order.
- To avoid surprising side effects from unreferenced CTEs.
- To avoid locking and performance problems.
- To write portable SQL across engines.

#### Syntax Rules and Structure

**Hazard patterns**:

```sql
-- Hazard: assumes sibling order
WITH a AS (UPDATE t SET x = 1 RETURNING *),
     b AS (SELECT * FROM t WHERE x = 1)   -- does NOT see a's change
SELECT * FROM b;

-- Hazard: unreferenced DML still runs
WITH d AS (DELETE FROM t WHERE id = 1 RETURNING *)
SELECT 1;

-- Hazard: RETURNING on UPDATE shows new values, not old
WITH u AS (UPDATE t SET x = x + 1 RETURNING x) SELECT * FROM u;
```

**Component breakdown**

| Pitfall | Description |
|---------|-------------|
| Sibling order | Independent CTEs have no guaranteed execution order. |
| Snapshot isolation | Base‑table reads do not see sibling CTE changes. |
| Unreferenced DML | Runs anyway; surprising side effects. |
| `RETURNING` semantics | Shows new values on `UPDATE`, deleted rows on `DELETE`. |
| Lock duration | Long statements hold locks. |
| Engine portability | MySQL 8.4 lacks DML CTEs. |

**Syntax rules**

- Use `RETURNING` output to pass data between CTEs, not base‑table re‑reads.
- Reference or explicitly document unreferenced DML CTEs.
- Keep statements short to reduce lock duration.
- Check engine support before relying on DML CTEs.

**Constraints and limitations**

- Execution order of independent CTEs is unspecified.
- Snapshot semantics mean changes are not visible via re‑reads.
- Some engines disallow DML in CTEs entirely.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Snapshot isolation pitfall

```sql
WITH a AS (
    UPDATE items SET price = price * 2 WHERE id = 1 RETURNING *
),
b AS (
    SELECT price FROM items WHERE id = 1   -- sees the OLD price
)
SELECT a.price AS new_price, b.price AS old_price
FROM a CROSS JOIN b;
```

**Expected output**:

```text
 new_price | old_price
-----------+-----------
     20.00 |     10.00
```

**Why this output appears**: `b` reads the base table in the same snapshot, so it sees the pre‑update price; only `a`’s `RETURNING` shows the new price. To pass new values, reference `a` directly.

##### Example 2: PostgreSQL – Unreferenced DML still executes

```sql
WITH d AS (
    DELETE FROM sessions WHERE id = 1 RETURNING *
)
SELECT 1 AS ok;
```

**Expected output**:

```text
 ok
----
  1
(1 row)
```

**Why this output appears**: The delete executes even though the main query does not reference `d`; this is a common surprise when the DML CTE was intended as a dry run.

##### Example 3: MySQL 8.0 – DML CTE not supported

```sql
-- This is NOT supported in MySQL 8.0/8.4.
WITH ins AS (
    INSERT INTO t (x) VALUES (1) RETURNING id
)
SELECT * FROM ins;
```

**Expected output**:

```text
ERROR 1064 (42000): You have an error in your SQL syntax ...
```

**Why this output appears**: MySQL does not support data-modifying statements inside a CTE body; use a transaction with separate statements or a stored procedure instead.

#### Real-World Cases

- **Incorrect data propagation**: Teams mistakenly re-read tables instead of using `RETURNING`.
- **Unintended deletes**: A DML CTE runs even when “testing” the outer query.
- **Portability failures**: Scripts written for PostgreSQL fail on MySQL.

---

## References

- PostgreSQL Documentation – `WITH` Queries (Data-Modifying Statements in `WITH`) – https://www.postgresql.org/docs/current/queries-with.html#QUERIES-WITH-MODIFYING
- PostgreSQL Documentation – `INSERT` (`RETURNING`) – https://www.postgresql.org/docs/current/sql-insert.html
- PostgreSQL Documentation – `UPDATE` (`RETURNING`) – https://www.postgresql.org/docs/current/sql-update.html
- PostgreSQL Documentation – `DELETE` (`RETURNING`) – https://www.postgresql.org/docs/current/sql-delete.html
- PostgreSQL Documentation – `MERGE` – https://www.postgresql.org/docs/current/sql-merge.html
- Microsoft Learn – `OUTPUT` Clause (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/queries/output-clause-transact-sql
- Microsoft Learn – `WITH common_table_expression` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql
- MySQL 8.4 Reference Manual – `WITH` (Common Table Expressions) – https://dev.mysql.com/doc/refman/8.4/en/with.html
- SQLite Documentation – `RETURNING` – https://www.sqlite.org/lang_returning.html
- SQL:1999 Standard – Common Table Expressions (ISO/IEC 9075-2:1999)
- Modern SQL – Data-Modifying CTEs – https://modern-sql.com/feature/with
- Use The Index, Luke – Data-Modifying CTEs – https://use-the-index-luke.com/sql/with