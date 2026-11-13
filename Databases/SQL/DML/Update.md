# Comprehensive Programming Cheat Sheet: SQL UPDATE Operations

## Topic Overview

### Definitions

**Core Definition**
UPDATE operations are the Data Manipulation Language (DML) statements used to modify existing rows in a table by assigning new values to one or more columns, optionally filtered by a condition.

**Technical Definition**
`UPDATE` changes the values of the specified columns in all rows that satisfy the `WHERE` condition; only the columns to be modified need be mentioned in the `SET` clause, and columns not explicitly modified retain their previous values. The statement supports single-column and multi-column assignments, expression-based values (`column = column + 1`), `DEFAULT` assignment, subquery-derived values, and—in vendor-specific extensions—`FROM`/multi-table joins, CTEs, `RETURNING` (PostgreSQL) or `OUTPUT` (SQL Server), `ORDER BY ... LIMIT` (MySQL single-table), and `TOP` (SQL Server). UPDATE is a write operation that acquires row-level locks (and in some systems, page or table locks) and writes to the transaction log; its performance is dominated by index maintenance, lock contention, and the number of rows affected per transaction.

**Beginner-Friendly Explanation**
UPDATE is how you change data that is already in a table. You tell the database which table to modify, which columns get new values, and—critically—which rows should be changed. Without a `WHERE` clause, every row in the table is updated, which is rarely what you want. UPDATE is powerful but also risky: a missing or wrong `WHERE` clause can overwrite an entire table.

### Key Characteristics

- **`WHERE` is the safety net**: Omitting `WHERE` updates every row. Production guidelines universally require a `WHERE` clause, and some systems enforce this with `sql_safe_updates` (MySQL) or `pg-safeupdate` (PostgreSQL).
- **Set-based semantics**: `UPDATE` operates on sets of rows; the `SET` expressions can reference the old values of the row (`SET counter = counter + 1`), enabling atomic increments without a read-modify-write cycle.
- **Vendor divergence in multi-table updates**: PostgreSQL uses `UPDATE ... FROM`; MySQL uses `UPDATE ... JOIN` (or comma-separated table references); SQL Server uses `UPDATE ... FROM` with a join, and also supports CTE-based updates. The semantics of which row gets updated when multiple source rows match are vendor-dependent.
- **Returning modified data**: PostgreSQL's `RETURNING` and SQL Server's `OUTPUT` return the old and/or new values of updated rows in the same statement, eliminating a follow-up `SELECT`.
- **Performance is write-bound**: Every UPDATE touches indexes, writes to WAL/redo, and creates row versions (in MVCC systems) or acquires exclusive locks. Batch updates are the standard technique for safely modifying large tables without long lock holds or replication lag.

### Prerequisites

- A running RDBMS instance (PostgreSQL, MySQL, or SQL Server) and a client connection.
- `UPDATE` privilege on the target table (and `SELECT` privilege on any columns read in the `SET` or `WHERE` expressions).
- For multi-table updates: `UPDATE` privilege on every table being modified and `SELECT` privilege on the source tables.
- A `WHERE` clause that identifies the target rows using an indexed column, to avoid full-table scans and excessive locking.

### Related Programming Areas

- Data Manipulation Language (DML) – `INSERT`, `DELETE`, `MERGE`.
- Data Definition Language (DDL) – schema changes that affect which columns can be updated.
- Transaction management – `COMMIT`, `ROLLBACK`, isolation levels, and lock timeouts.
- Concurrency control – row-level vs. page-level vs. table-level locking; MVCC vs. lock-based reads.
- Replication and CDC – large UPDATEs generate large amounts of WAL/binlog, causing replication lag.

---

## Core Concepts and Features

### 1. `UPDATE`

#### Definitions

**Core Definition**
`UPDATE` is the SQL statement that modifies existing rows in a table by assigning new values to specified columns.

**Technical Definition**
The PostgreSQL synopsis is `UPDATE [ ONLY ] table_name [ * ] [ [ AS ] alias ] SET { column = expression | DEFAULT | (column, ...) = (sub-SELECT) } [, ...] [ FROM from_item [, ...] ] [ WHERE condition | WHERE CURRENT OF cursor ] [ RETURNING ... ]`. MySQL's single-table syntax is `UPDATE [LOW_PRIORITY] [IGNORE] table_reference SET assignment_list [WHERE where_condition] [ORDER BY ...] [LIMIT row_count]`, and its multiple-table syntax is `UPDATE [LOW_PRIORITY] [IGNORE] table_references SET assignment_list [WHERE where_condition]`. SQL Server's syntax includes `[ WITH <common_table_expression> ] UPDATE [ TOP ( expression ) [ PERCENT ] ] { table_alias | object | @table_variable } SET ... [ OUTPUT Clause ] [ FROM { <table_source> } ] [ WHERE ... ]`.

**Beginner-Friendly Explanation**
`UPDATE` is the master command for changing existing data. You name the table, write `SET` followed by the column(s) and new value(s), and add a `WHERE` clause to say which rows should change.

#### Purposes

- To modify existing records in response to user actions or business processes.
- To correct data errors or apply bulk data fixes.
- To increment counters, update timestamps, or change status values atomically.
- To synchronise data between tables using values from related tables.

#### Syntax Rules and Structure

**General syntax (PostgreSQL):**

```sql
UPDATE table_name [ AS alias ]
SET column_name = { expression | DEFAULT | sub-SELECT } [, ...]
[ FROM from_item [, ...] ]
[ WHERE condition ];
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `table_name` | The table to update; can be schema-qualified. |
| `SET` | Introduces the column assignments. |
| `column_name = expression` | Assigns a new value; the expression can reference old column values. |
| `DEFAULT` | Sets the column to its declared default (or `NULL`). |
| `FROM` (PostgreSQL, SQL Server) | Additional tables providing values for the update. |
| `WHERE` | Filters which rows are updated; without it, all rows are updated. |

**Syntax rules:**

- The `SET` clause must contain at least one assignment.
- The `WHERE` clause is technically optional, but omitting it updates every row.
- In PostgreSQL, the target column must not be qualified with the table name (`SET table_name.col = 1` is invalid).

**Constraints and limitations:**

- Updating a column that is part of a primary key or unique constraint may violate the constraint if the new value duplicates an existing value.
- In MySQL, the multiple-table syntax cannot use `ORDER BY` or `LIMIT`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Basic UPDATE with a WHERE clause

```sql
-- Step 1: Create a table and insert sample data.
CREATE TABLE employees (
    id     serial PRIMARY KEY,
    name   text NOT NULL,
    salary numeric(10,2),
    active boolean DEFAULT true
);

INSERT INTO employees (name, salary) VALUES
    ('Alice', 50000), ('Bob', 60000), ('Carol', 70000);

-- Step 2: Give Alice a raise.
UPDATE employees
SET salary = 55000
WHERE name = 'Alice';

-- Step 3: Verify.
SELECT id, name, salary FROM employees ORDER BY id;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 3
UPDATE 1
 id | name  | salary
----+-------+--------
  1 | Alice |  55000
  2 | Bob   |  60000
  3 | Carol |  70000
(3 rows)
```

**Why this output appears**: The `WHERE name = 'Alice'` clause restricts the update to one row; `UPDATE 1` confirms that exactly one row was modified.

#### Real-World Cases

- **Profile updates**: A user changes their email address; an `UPDATE` modifies the `email` column for that user's row.
- **Price adjustments**: A product's price is increased by 5%; `UPDATE products SET price = price * 1.05 WHERE category = 'electronics'` applies the change to a category.

---

### 2. Single-Row Update

#### Definitions

**Core Definition**
A single-row update modifies exactly one row, typically identified by a primary key or a unique constraint.

**Technical Definition**
A single-row update uses a `WHERE` clause that matches at most one row (e.g., `WHERE id = 42`). The database acquires an exclusive row lock on that row, writes the new values, updates any affected indexes, and writes a WAL/redo record. Single-row updates are the dominant OLTP pattern and are efficient when the `WHERE` clause uses an indexed column.

**Beginner-Friendly Explanation**
A single-row update changes one specific record—like updating one user's profile or one order's status. You use the row's unique ID in the `WHERE` clause to make sure only that record changes.

#### Purposes

- To update a specific record identified by its primary key or another unique identifier.
- To perform user-facing operations where the target is known precisely.
- To avoid unintended widespread changes.

#### Syntax Rules and Structure

**General syntax:**

```sql
UPDATE table_name
SET column1 = value1 [, column2 = value2 ...]
WHERE primary_key_column = value;
```


**Syntax rules:**

- The `WHERE` clause should reference an indexed column (ideally the primary key) to ensure efficient row lookup.
- The statement should affect exactly one row if the key is unique.

**Constraints and limitations:**

- If the `WHERE` clause matches zero rows, the update succeeds but affects no rows.
- If the `WHERE` clause somehow matches multiple rows (e.g., a non-unique column), all matching rows are updated.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Single-row update by primary key

```sql
-- Step 1: Create and populate a table.
CREATE TABLE accounts (
    id      INT PRIMARY KEY,
    balance DECIMAL(10,2)
);
INSERT INTO accounts VALUES (1, 1000.00), (2, 500.00);

-- Step 2: Update one account.
UPDATE accounts
SET balance = balance + 250.00
WHERE id = 1;

-- Step 3: Verify.
SELECT * FROM accounts;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
+----+---------+
| id | balance |
+----+---------+
|  1 | 1250.00 |
|  2 |  500.00 |
+----+---------+
```

**Why this output appears**: The `WHERE id = 1` clause targets exactly one row; `Rows matched: 1` confirms the single-row scope.

#### Real-World Cases

- **User settings**: A user toggles a preference; `UPDATE users SET theme = 'dark' WHERE user_id = 42`.
- **Inventory adjustment**: A single product's stock is decremented after a sale.

---

### 3. Multi-Row Update

#### Definitions

**Core Definition**
A multi-row update modifies two or more rows in a single `UPDATE` statement, typically using a `WHERE` clause that matches multiple rows.

**Technical Definition**
A multi-row update's `WHERE` clause returns a set of rows rather than a single row. The database locks all matching rows and applies the `SET` assignments to each. Multi-row updates can affect large portions of a table and are the primary mechanism for bulk data corrections. Their performance and locking impact depend on the number of rows, the indexes used in the `WHERE` clause, and whether the update causes page splits, lock escalation, or replication lag.

**Beginner-Friendly Explanation**
A multi-row update changes many records at once. For example, you might give every employee in a department a raise, or mark all orders older than a year as archived.

#### Purposes

- To apply a uniform change to a defined set of rows (e.g., a category-wide price increase).
- To correct data across many rows in a single operation.
- To batch state transitions (e.g., marking all pending orders as processed).

#### Syntax Rules and Structure

**General syntax:**

```sql
UPDATE table_name
SET column1 = value1 [, ...]
WHERE condition_that_matches_multiple_rows;
```


**Syntax rules:**

- The `WHERE` clause should be designed to use indexes where possible; a non-indexed `WHERE` clause causes a full-table scan and locks many rows.
- In MySQL, `LIMIT row_count` can cap the number of rows updated in a single statement, enabling controlled batching.

**Constraints and limitations:**

- Updating millions of rows in one transaction can cause lock contention, long-running transactions, and replication lag.
- In PostgreSQL, a large `UPDATE` creates new row versions and requires `VACUUM` to reclaim dead tuples.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Multi-row update by category

```sql
-- Step 1: Create and populate a table.
CREATE TABLE products (id int PRIMARY KEY, category text, price numeric);
INSERT INTO products VALUES (1, 'A', 10), (2, 'A', 20), (3, 'B', 30);

-- Step 2: Increase prices for category A.
UPDATE products SET price = price * 1.10 WHERE category = 'A';

-- Step 3: Verify.
SELECT * FROM products ORDER BY id;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 3
UPDATE 2
 id | category | price
----+----------+-------
  1 | A        |    11
  2 | A        |    22
  3 | B        |    30
(3 rows)
```

**Why this output appears**: The `WHERE category = 'A'` clause matches two rows; both receive a 10% price increase, and `UPDATE 2` confirms the row count.

#### Real-World Cases

- **Archival**: `UPDATE orders SET status = 'archived' WHERE created_at < '2024-01-01'` moves old orders to an archived state.
- **Bulk correction**: A data quality issue requires updating all rows where a `country` column has a legacy code.

---

### 4. Single-Column Update

#### Definitions

**Core Definition**
A single-column update modifies exactly one column in the target row(s).

**Technical Definition**
A single-column update's `SET` clause contains one assignment: `SET column = expression`. The database writes the new value to that column's storage and updates any indexes that include the column. Single-column updates are the most common form and are generally cheaper than multi-column updates because fewer indexes need maintenance.

**Beginner-Friendly Explanation**
A single-column update changes just one field—like changing a person's phone number without touching their name or address.

#### Purposes

- To modify a single attribute of a row without affecting other columns.
- To minimise index maintenance overhead when only one column needs to change.
- To perform targeted corrections or state transitions.

#### Syntax Rules and Structure

**General syntax:**

```sql
UPDATE table_name SET column = expression WHERE condition;
```


**Syntax rules:**

- The `SET` clause contains exactly one assignment.
- The expression can reference the column's old value (e.g., `SET counter = counter + 1`).

**Constraints and limitations:**

- If the column is part of a unique constraint, the new value must not duplicate an existing value.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Single-column update

```sql
-- Step 1: Create and populate a table.
CREATE TABLE users (id INT PRIMARY KEY, email VARCHAR(255), active BOOLEAN);
INSERT INTO users VALUES (1, 'a@example.com', TRUE);

-- Step 2: Deactivate one user.
UPDATE users SET active = FALSE WHERE id = 1;

-- Step 3: Verify.
SELECT * FROM users;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 1 row affected (0.01 sec)
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
+----+----------------+--------+
| id | email          | active |
+----+----------------+--------+
|  1 | a@example.com  |      0 |
+----+----------------+--------+
```

**Why this output appears**: Only the `active` column changes; `email` retains its previous value.

#### Real-World Cases

- **Soft delete**: `UPDATE users SET deleted_at = NOW() WHERE id = 42` marks a row as deleted without physically removing it.
- **Status transition**: `UPDATE jobs SET status = 'running' WHERE id = 100` changes a job's state.

---

### 5. Multi-Column Update

#### Definitions

**Core Definition**
A multi-column update modifies two or more columns in the target row(s) within a single `UPDATE` statement.

**Technical Definition**
A multi-column update's `SET` clause contains multiple comma-separated assignments: `SET col1 = expr1, col2 = expr2, ...`. In PostgreSQL, a row-valued assignment can also be used: `SET (col1, col2) = (expr1, expr2)` or `SET (col1, col2) = (sub-SELECT)`. All assignments are evaluated using the old row values unless a subquery is involved.

**Beginner-Friendly Explanation**
A multi-column update changes several fields at once—like updating a user's name, email, and phone number in one operation.

#### Purposes

- To update related fields atomically, ensuring consistency across columns.
- To reduce the number of statements when multiple columns need new values.
- To use row-valued subqueries that compute multiple column values together.

#### Syntax Rules and Structure

**General syntax:**

```sql
UPDATE table_name
SET column1 = expression1, column2 = expression2 [, ...]
WHERE condition;
```


**Row-valued form (PostgreSQL):**

```sql
UPDATE table_name
SET (column1, column2) = (SELECT expr1, expr2 FROM ...)
WHERE condition;
```


**Syntax rules:**

- All assignments in a single `UPDATE` see the old row values (simultaneous assignment semantics).
- The row-valued form requires the subquery to return exactly one row; if it returns zero rows, all target columns are set to `NULL`.

**Constraints and limitations:**

- If any assignment violates a constraint, the entire statement fails (unless `ON CONFLICT` or `IGNORE` is used).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Multi-column update with simultaneous assignment

```sql
-- Step 1: Create a table.
CREATE TABLE stats (id int PRIMARY KEY, wins int, losses int);
INSERT INTO stats VALUES (1, 5, 3);

-- Step 2: Swap wins and losses in one statement.
UPDATE stats SET wins = losses, losses = wins WHERE id = 1;

-- Step 3: Verify.
SELECT * FROM stats;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
UPDATE 1
 id | wins | losses
----+------+--------
  1 |    3 |      5
(1 row)
```

**Why this output appears**: Both assignments are evaluated against the old row values (`wins = 5`, `losses = 3`), so the swap works correctly. If assignments were applied sequentially, the second assignment would see the already-updated `wins` value and produce the wrong result.

#### Real-World Cases

- **Profile update**: A user changes their name and email in one form submission; a multi-column update applies both changes atomically.
- **State transition**: A job's status and `completed_at` timestamp are updated together.

---

### 6. Conditional Updates

#### Definitions

**Core Definition**
A conditional update applies different values to different rows based on conditions evaluated within the `SET` clause, rather than applying a single uniform value to all matching rows.

**Technical Definition**
Conditional updates typically use a `CASE` expression in the `SET` clause: `SET column = CASE WHEN condition1 THEN value1 WHEN condition2 THEN value2 ELSE value3 END`. This allows a single `UPDATE` statement to apply row-specific logic. The `WHERE` clause still determines which rows are candidates; the `CASE` expression determines the value each candidate receives.

**Beginner-Friendly Explanation**
A conditional update lets you set different values for different rows in one statement. For example, you could give a 10% raise to employees in one department and a 5% raise to everyone else, all in a single `UPDATE`.

#### Purposes

- To apply tiered or category-specific changes in one statement without multiple round-trips.
- To implement business rules that depend on the row's current values.
- To avoid the overhead of separate `UPDATE` statements for each condition.

#### Syntax Rules and Structure

**General syntax:**

```sql
UPDATE table_name
SET column = CASE
    WHEN condition1 THEN value1
    WHEN condition2 THEN value2
    ELSE default_value
END
WHERE some_condition;
```


**Syntax rules:**

- The `CASE` expression must return a value compatible with the column's type.
- The `ELSE` clause provides a value for rows that match none of the `WHEN` conditions; if omitted, those rows receive `NULL`.

**Constraints and limitations:**

- Complex `CASE` expressions can make the statement harder to read and optimise.
- MySQL also supports `IF(condition, then_value, else_value)` as a simpler conditional function.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – CASE-based conditional update

```sql
-- Step 1: Create a table.
CREATE TABLE employees (id int PRIMARY KEY, dept text, salary numeric);
INSERT INTO employees VALUES (1, 'eng', 100), (2, 'eng', 100), (3, 'sales', 100);

-- Step 2: Give engineering a 20% raise and sales a 10% raise.
UPDATE employees
SET salary = CASE
    WHEN dept = 'eng'   THEN salary * 1.20
    WHEN dept = 'sales' THEN salary * 1.10
    ELSE salary
END;

-- Step 3: Verify.
SELECT * FROM employees ORDER BY id;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 3
UPDATE 3
 id | dept  | salary
----+-------+--------
  1 | eng   |    120
  2 | eng   |    120
  3 | sales |    110
(3 rows)
```

**Why this output appears**: The `CASE` expression applies different multipliers based on the `dept` column, allowing a single statement to implement tiered raises.

#### Real-World Cases

- **Tiered pricing**: Different customer tiers receive different discount percentages in one `UPDATE`.
- **Priority-based routing**: A support ticket's priority is set based on its category using a `CASE` expression.

---

### 7. Updating Using Expressions

#### Definitions

**Core Definition**
Updating using expressions assigns new values computed from the old values of the row (or from functions and arithmetic) rather than literal constants.

**Technical Definition**
The `SET` clause accepts arbitrary expressions, including arithmetic (`price * 1.10`), string concatenation, function calls (`now()`, `upper(name)`), and references to other columns in the same row. The expression is evaluated for each row using the old values before the update. This enables atomic increments, computed timestamps, and derived values without a separate read-modify-write cycle.

**Beginner-Friendly Explanation**
Updating using expressions means the new value is calculated rather than typed in. For example, “add 10% to the current price” or “set the updated_at column to the current time.”

#### Purposes

- To atomically increment or decrement counters (`SET counter = counter + 1`).
- To apply percentage-based or formula-based changes.
- To derive new values from existing columns or functions.

#### Syntax Rules and Structure

**General syntax:**

```sql
UPDATE table_name
SET column = expression_referencing_old_values
WHERE condition;
```


**Syntax rules:**

- The expression can reference any column of the same row; the old value is used.
- Functions such as `now()`, `upper()`, `coalesce()` can be used.

**Constraints and limitations:**

- If the expression can produce a value that violates a constraint (e.g., division by zero, overflow), the statement fails.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Expression-based increment and timestamp update

```sql
-- Step 1: Create a table.
CREATE TABLE counters (
    id         INT PRIMARY KEY,
    hits       INT NOT NULL DEFAULT 0,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
INSERT INTO counters (id, hits) VALUES (1, 10);

-- Step 2: Increment hits and let updated_at refresh automatically.
UPDATE counters SET hits = hits + 1 WHERE id = 1;

-- Step 3: Verify.
SELECT id, hits, updated_at FROM counters;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 1 row affected (0.01 sec)
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
+----+------+---------------------+
| id | hits | updated_at          |
+----+------+---------------------+
|  1 |   11 | 2026-11-13 10:00:00 |
+----+------+---------------------+
```

**Why this output appears**: `hits = hits + 1` reads the old value (10) and stores 11. The `ON UPDATE CURRENT_TIMESTAMP` clause automatically refreshes `updated_at` when the row changes.

#### Real-World Cases

- **Counters**: Incrementing page views, login counts, or stock levels without a read-modify-write cycle.
- **Audit timestamps**: `UPDATE ... SET updated_at = now()` on every modification.

---

### 8. Update with `FROM` (PostgreSQL)

#### Definitions

**Core Definition**
`UPDATE ... FROM` is a PostgreSQL extension that allows the `SET` expressions to reference columns from other tables in the same database, joining the target table to a source table.

**Technical Definition**
The `FROM` clause in PostgreSQL's `UPDATE` specifies additional tables that can be referenced in the `SET` expressions and the `WHERE` condition. The target table is implicitly joined to the `FROM` tables; if multiple source rows match a target row, only one of them is used (which one is unpredictable). This is the standard PostgreSQL technique for updating a table based on values in another table.

**Beginner-Friendly Explanation**
`UPDATE ... FROM` lets you update one table using data from another table. For example, you can set each product's price to the latest price in a price-history table.

#### Purposes

- To update a table using values from a related table (e.g., synchronising denormalised data).
- To avoid correlated subqueries when the update depends on a join.
- To perform set-based updates that would otherwise require procedural code.

#### Syntax Rules and Structure

**General syntax (PostgreSQL):**

```sql
UPDATE target_table
SET column = source_table.expression
FROM source_table
WHERE target_table.join_key = source_table.join_key
  AND condition;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `FROM` | Introduces additional tables for the update. |
| `SET column = source.expression` | Assigns a value derived from the source table. |
| `WHERE target.key = source.key` | Joins the target and source tables. |

**Syntax rules:**

- The `FROM` clause can list multiple tables, effectively performing a join.
- If multiple source rows match a target row, only one is used (unpredictable); use a subquery with aggregation or `DISTINCT ON` to control which row is chosen.

**Constraints and limitations:**

- PostgreSQL-specific; MySQL and SQL Server use different syntax (JOIN and `FROM` with aliases, respectively).
- The target table cannot appear in the `FROM` clause in some configurations (use a subquery or CTE instead).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – UPDATE ... FROM with a price-history table

```sql
-- Step 1: Create target and source tables.
CREATE TABLE products (id int PRIMARY KEY, price numeric);
CREATE TABLE price_updates (product_id int, new_price numeric);

INSERT INTO products VALUES (1, 10), (2, 20);
INSERT INTO price_updates VALUES (1, 15), (2, 25);

-- Step 2: Update prices from the price_updates table.
UPDATE products AS p
SET price = u.new_price
FROM price_updates AS u
WHERE p.id = u.product_id;

-- Step 3: Verify.
SELECT * FROM products ORDER BY id;
```

**Expected output:**

```text
CREATE TABLE
CREATE TABLE
INSERT 0 2
INSERT 0 2
UPDATE 2
 id | price
----+-------
  1 |    15
  2 |    25
(2 rows)
```

**Why this output appears**: The `FROM price_updates` clause provides the new prices, and the `WHERE` join condition matches each product to its corresponding update row.

#### Real-World Cases

- **Denormalisation sync**: A `orders` table's `customer_name` column is updated from the `customers` table after a customer rename.
- **Inventory reconciliation**: Stock levels are updated from a warehouse-count staging table.

---

### 9. Update with Subqueries

#### Definitions

**Core Definition**
An update with subqueries uses a scalar subquery or a correlated subquery in the `SET` clause to compute the new value for each row.

**Technical Definition**
A scalar subquery in `SET` must return at most one row and one column. A correlated subquery references the target table's columns, so it is evaluated once per row. PostgreSQL also supports row-valued subqueries: `SET (col1, col2) = (SELECT expr1, expr2 FROM ...)`. Subquery-based updates are portable across RDBMSs but can be slower than join-based updates because the subquery may execute once per row.

**Beginner-Friendly Explanation**
A subquery update calculates the new value by running a query inside the `UPDATE`. For example, “set each employee's salary to the average salary of their department” uses a subquery to compute the average.

#### Purposes

- To update a column with an aggregate or derived value from another table.
- To perform row-specific lookups when a join is not convenient.
- To use portable SQL that works across RDBMSs without vendor-specific `FROM` or `JOIN` syntax.

#### Syntax Rules and Structure

**Scalar subquery:**

```sql
UPDATE table_name
SET column = (SELECT expression FROM other_table WHERE condition)
WHERE condition;
```


**Row-valued subquery (PostgreSQL):**

```sql
UPDATE table_name
SET (col1, col2) = (SELECT expr1, expr2 FROM other_table WHERE condition)
WHERE condition;
```


**Syntax rules:**

- The subquery must return no more than one row; if it returns zero rows, the target columns are set to `NULL`.
- Correlated subqueries reference the outer `UPDATE`'s table.

**Constraints and limitations:**

- MySQL does not allow the target table to appear in a subquery in the `FROM` clause (error 1093); use a `JOIN`-based update instead.
- Subqueries can be slower than joins on large datasets.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Correlated subquery update

```sql
-- Step 1: Create tables.
CREATE TABLE employees (id int PRIMARY KEY, dept_id int, salary numeric);
CREATE TABLE departments (id int PRIMARY KEY, avg_salary numeric);

INSERT INTO employees VALUES (1, 1, 100), (2, 1, 120), (3, 2, 80);
INSERT INTO departments VALUES (1, 110), (2, 80);

-- Step 2: Set each employee's salary to their department's average.
UPDATE employees AS e
SET salary = (SELECT d.avg_salary FROM departments AS d WHERE d.id = e.dept_id);

-- Step 3: Verify.
SELECT * FROM employees ORDER BY id;
```

**Expected output:**

```text
CREATE TABLE
CREATE TABLE
INSERT 0 3
INSERT 0 2
UPDATE 3
 id | dept_id | salary
----+---------+--------
  1 |       1 |    110
  2 |       1 |    110
  3 |       2 |     80
(3 rows)
```

**Why this output appears**: The correlated subquery looks up `avg_salary` for each employee's `dept_id`, applying the department-specific average to each row.

#### Real-World Cases

- **Normalisation**: A denormalised `total` column is recomputed from a sum over a related `line_items` table.
- **Ranking or bucketing**: Each row is assigned a category based on an aggregate computed by a subquery.

---

### 10. Update with CTEs

#### Definitions

**Core Definition**
An update with CTEs uses a `WITH` clause to define one or more common table expressions (CTEs) that can be referenced in the `UPDATE` statement, simplifying complex logic.

**Technical Definition**
All three major RDBMSs support `WITH ... UPDATE`. In PostgreSQL, the `WITH` clause precedes `UPDATE` and defines named subqueries that can be referenced in the `SET` or `WHERE` clauses. In MySQL, `UPDATE` can start with a `WITH` clause to define CTEs accessible within the update. In SQL Server, the `WITH <common_table_expression>` clause specifies a temporary named result set that the `UPDATE` references. CTEs are especially useful for computing aggregates or deduplicating source data before applying the update.

**Beginner-Friendly Explanation**
A CTE is a temporary named query that you define before the `UPDATE`. You can use it to pre-compute the values you need, making the update easier to read and sometimes faster.

#### Purposes

- To pre-compute aggregates or joins once and reference them multiple times.
- To simplify complex update logic by breaking it into named steps.
- To deduplicate source rows before updating, avoiding unpredictable results from multiple matches.

#### Syntax Rules and Structure

**General syntax (PostgreSQL):**

```sql
WITH cte_name AS (
    SELECT ...
)
UPDATE target_table
SET column = cte_name.expression
FROM cte_name
WHERE target_table.key = cte_name.key;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `WITH cte_name AS (...)` | Defines a named temporary result set. |
| `UPDATE ... FROM cte_name` | References the CTE as a source table. |

**Syntax rules:**

- The CTE is visible only within the `UPDATE` statement.
- Multiple CTEs can be defined, separated by commas.
- In SQL Server, the CTE can be referenced in the `FROM` clause of the `UPDATE`.

**Constraints and limitations:**

- MySQL's CTE support for `UPDATE` may still trigger error 1093 in some patterns where the CTE references the target table.
- CTEs are not always materialised; the optimiser may inline them, so performance varies.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – UPDATE with a CTE

```sql
-- Step 1: Create tables.
CREATE TABLE orders (id int PRIMARY KEY, customer_id int, total numeric);
CREATE TABLE order_totals (customer_id int, grand_total numeric);

INSERT INTO orders VALUES (1, 100, 50), (2, 100, 70), (3, 200, 30);

-- Step 2: Use a CTE to compute totals and update a summary table.
WITH totals AS (
    SELECT customer_id, SUM(total) AS grand_total
    FROM orders
    GROUP BY customer_id
)
UPDATE order_totals
SET grand_total = t.grand_total
FROM totals AS t
WHERE order_totals.customer_id = t.customer_id;
```

**Expected output:**

```text
Commands completed successfully.
```

**Why this output appears**: The CTE `totals` computes each customer's total order amount once; the `UPDATE` then applies those totals to the summary table. (If `order_totals` is empty, the update affects zero rows; the CTE itself succeeds.)

#### Real-World Cases

- **Summary table refresh**: A CTE aggregates line items, and the `UPDATE` applies the aggregates to a denormalised summary table.
- **Deduplication before update**: A CTE uses `ROW_NUMBER()` to select one row per key, preventing unpredictable multi-match behaviour in the update.

---

### 11. Updating from Related Data

#### Definitions

**Core Definition**
Updating from related data is the general practice of modifying a table using values derived from other tables, whether through `FROM` (PostgreSQL), `JOIN` (MySQL), `FROM` with joins (SQL Server), subqueries, or CTEs.

**Technical Definition**
Different RDBMSs expose different syntax for updating from related data. **PostgreSQL** uses `UPDATE ... FROM`. **MySQL** uses `UPDATE table1 JOIN table2 ON ... SET table1.col = table2.col` (multiple-table syntax); MySQL explicitly notes that for single-table `UPDATE` with a subquery, the optimiser does not use semijoin or materialization optimisations, and recommends rewriting as a multi-table update using a join instead. **SQL Server** uses `UPDATE target SET ... FROM target JOIN source ON ...`; the `FROM` clause specifies the rows to update.

**Beginner-Friendly Explanation**
Updating from related data means using information from another table to decide what to change. For example, you might update each product's price based on the latest entry in a price-history table, or copy a customer's name into an orders table.

#### Purposes

- To synchronise denormalised or cached data with its source.
- To apply values from a staging table to a production table.
- To perform set-based updates that depend on joins across multiple tables.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
UPDATE target SET col = source.col
FROM source
WHERE target.key = source.key;
```


**MySQL:**

```sql
UPDATE target
JOIN source ON target.key = source.key
SET target.col = source.col;
```


**SQL Server:**

```sql
UPDATE target
SET col = source.col
FROM target
JOIN source ON target.key = source.key;
```


**Syntax rules:**

- Ensure the join is deterministic (a 1:1 or 1:N relationship where only one source row matches each target row); otherwise, the updated value is unpredictable. SQL Server guidance recommends using a CTE with `ROW_NUMBER()` to resolve duplicates before updating.
- In MySQL, the multiple-table syntax cannot use `ORDER BY` or `LIMIT`.

**Constraints and limitations:**

- Non-deterministic joins can produce arbitrary results.
- MySQL's single-table `UPDATE` with a subquery on the same table is restricted (error 1093).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – UPDATE ... JOIN

```sql
-- Step 1: Create target and source tables.
CREATE TABLE products (id INT PRIMARY KEY, price DECIMAL(10,2));
CREATE TABLE price_updates (product_id INT, new_price DECIMAL(10,2));

INSERT INTO products VALUES (1, 10.00), (2, 20.00);
INSERT INTO price_updates VALUES (1, 12.50), (2, 25.00);

-- Step 2: Update prices using a JOIN.
UPDATE products AS p
JOIN price_updates AS u ON p.id = u.product_id
SET p.price = u.new_price;

-- Step 3: Verify.
SELECT * FROM products ORDER BY id;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 0 rows affected (0.02 sec)
Query OK, 2 rows affected (0.01 sec)
Rows matched: 2  Changed: 2  Warnings: 0
+----+-------+
| id | price |
+----+-------+
|  1 | 12.50 |
|  2 | 25.00 |
+----+-------+
```

**Why this output appears**: The `JOIN` matches each product to its price update, and the `SET` clause applies the new price from the source table.

#### Real-World Cases

- **Price synchronisation**: A retailer's product prices are updated from a supplier's feed table using a join.
- **Customer name denormalisation**: An `orders` table's `customer_name` is updated from the `customers` table after a bulk rename.

---

### 12. `RETURNING` (PostgreSQL)

#### Definitions

**Core Definition**
`RETURNING` is a PostgreSQL clause that returns data from the rows updated by an `UPDATE` statement, computed over the actually updated rows.

**Technical Definition**
The optional `RETURNING` clause causes `UPDATE` to compute and return value(s) based on each row actually updated. Any expression using the table's columns, and/or columns of other tables mentioned in `FROM`, can be computed. By default, the new (post-update) values are used, but the old (pre-update) values can be requested with `RETURNING WITH (OLD AS ...)`. The syntax of the `RETURNING` list is identical to the output list of `SELECT`. `RETURNING` is a PostgreSQL extension (also supported in some other systems with different syntax, such as SQL Server's `OUTPUT`).

**Beginner-Friendly Explanation**
`RETURNING` lets you get data back from the rows you just updated, in the same statement. This is useful when you need to know what changed without running a separate `SELECT`.

#### Purposes

- To return the new values of updated rows for logging or application use.
- To return the old values for audit trails.
- To retrieve generated or computed values after an update without a separate query.

#### Syntax Rules and Structure

**General syntax:**

```sql
UPDATE table_name
SET column = expression
WHERE condition
RETURNING * | output_expression [ [ AS ] output_name ] [, ...];
```


**Old values (PostgreSQL 18+):**

```sql
UPDATE table_name SET column = expression WHERE condition
RETURNING WITH (OLD AS o) o.column, NEW.column;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `RETURNING` | The clause keyword. |
| `output_expression` | A column name or expression using the updated row's columns. |
| `WITH (OLD AS ...)` | Optional alias for the pre-update values. |

**Syntax rules:**

- The `RETURNING` list can include `*` to return all columns.
- The data returned reflects the row as it exists after the update (including trigger modifications).

**Constraints and limitations:**

- `RETURNING` is PostgreSQL-specific; MySQL has no direct equivalent, and SQL Server uses `OUTPUT`.
- The user must have `SELECT` privilege on the returned columns.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – UPDATE with RETURNING

```sql
-- Step 1: Create a table.
CREATE TABLE accounts (id int PRIMARY KEY, balance numeric);
INSERT INTO accounts VALUES (1, 100), (2, 200);

-- Step 2: Update one account and return the new balance.
UPDATE accounts
SET balance = balance + 50
WHERE id = 1
RETURNING id, balance;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 2
UPDATE 1
 id | balance
----+---------
  1 |     150
(1 row)
```

**Why this output appears**: `RETURNING id, balance` returns the updated row's `id` and the new `balance` (150), computed after the update.

#### Real-World Cases

- **Audit logging**: After updating a row, `RETURNING` provides the old and new values for an audit table.
- **API responses**: A PUT endpoint updates a resource and returns the updated representation in one round-trip.

---

### 13. Update Performance

#### Definitions

**Core Definition**
Update performance is the measure of how efficiently an `UPDATE` statement executes, influenced by the number of rows, index maintenance, lock contention, logging overhead, and the batching strategy.

**Technical Definition**
UPDATE performance is dominated by several factors: (1) **rows affected**—updating many rows in one transaction increases lock hold time and log volume; (2) **index maintenance**—every index on a modified column must be updated; (3) **locking**—row-level locks in MVCC systems (PostgreSQL, MySQL InnoDB) or exclusive locks in SQL Server can block concurrent writes; (4) **WAL/redo volume**—large updates generate large amounts of write-ahead log, causing replication lag and I/O pressure; and (5) **dead tuple accumulation**—PostgreSQL creates new row versions, requiring `VACUUM` to reclaim space. Batching is the universal technique for improving update performance and safety on large tables: MySQL recommends batching large updates into small chunks to reduce lock hold time and redo log pressure; SQL Server uses `UPDATE TOP (N)` in a `WHILE` loop to break large updates into smaller autocommitted transactions; PostgreSQL practitioners recommend small chunks with a low `lock_timeout` to avoid long lock waits.

**Beginner-Friendly Explanation**
Updating a million rows in one statement is slow and risky: it locks the table for a long time, fills the transaction log, and can cause replication to fall behind. Breaking the update into batches of 1,000–10,000 rows, committing each batch, and pausing briefly between batches keeps the database responsive and safe.

#### Purposes

- To apply large data changes without blocking concurrent queries or causing replication lag.
- To reduce transaction log growth and avoid long-running transactions.
- To maintain predictable performance for production workloads.

#### Syntax Rules and Structure

**MySQL batch update:**

```sql
UPDATE table_name
SET column = value
WHERE condition
LIMIT 1000;  -- repeat until affected rows = 0
```


**SQL Server batch update:**

```sql
WHILE 1 = 1
BEGIN
    UPDATE TOP (5000) table_name
    SET column = value
    WHERE condition;
    IF @@ROWCOUNT < 5000 BREAK;
END
```


**PostgreSQL batch update (application-side loop):**

```sql
UPDATE table_name
SET column = value
WHERE id IN (
    SELECT id FROM table_name WHERE condition LIMIT 5000
);
-- commit; repeat
```


**Component breakdown:**

| Technique | RDBMS | Purpose |
|-----------|-------|---------|
| `LIMIT` | MySQL | Caps rows per statement |
| `TOP (N)` | SQL Server | Caps rows per statement |
| `LIMIT` subquery | PostgreSQL | Caps rows via a subquery |
| Index on `WHERE` | All | Avoids full-table scans |

**Syntax rules:**

- Each batch should be its own transaction (or autocommitted) to release locks promptly.
- A brief sleep (50–200 ms) between batches reduces I/O pressure and allows autovacuum or other maintenance to run.
- The `WHERE` clause should use an index to avoid scanning the entire table for each batch; using `LIMIT` without an index can force a full-table scan and still lock the table.

**Constraints and limitations:**

- `LIMIT` without an indexed `WHERE` clause can cause full-table scans and defeat the purpose of batching.
- In SQL Server, `TOP` rows are not ordered; without a deterministic `ORDER BY` (or a key-based `WHERE`), successive batches may miss rows or revisit the same rows.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Batched update with LIMIT

```sql
-- Step 1: Create a table with many rows.
CREATE TABLE logs (id INT PRIMARY KEY, processed BOOLEAN DEFAULT FALSE);
INSERT INTO logs (id) SELECT seq FROM seq_1_to_10000;

-- Step 2: Update in batches of 1000 (conceptual loop; run repeatedly).
UPDATE logs SET processed = TRUE WHERE processed = FALSE LIMIT 1000;

-- Step 3: Verify progress.
SELECT COUNT(*) AS remaining FROM logs WHERE processed = FALSE;
```

**Expected output (after repeated batches):**

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

**Why this output appears**: Each `UPDATE ... LIMIT 1000` processes 1,000 unprocessed rows, commits, and releases locks. Repeating the statement (or wrapping it in an application loop) eventually processes all 10,000 rows without a single long-running transaction.

#### Real-World Cases

- **Data backfill**: A new column is populated in batches to avoid locking a large table for an extended period.
- **Status cleanup**: Millions of old rows are marked as archived in batches of 5,000 with a short sleep between batches to keep replication lag manageable.

---

### 14. Locking Considerations

#### Definitions

**Core Definition**
Locking considerations for `UPDATE` refer to the lock modes acquired by the statement and their impact on concurrent reads and writes.

**Technical Definition**
Locking behaviour varies significantly by RDBMS. **PostgreSQL** uses MVCC for reads: ordinary `SELECT` statements do not block writers, and writers do not block readers. `UPDATE` acquires row-level exclusive locks on the updated rows; these locks are held until the transaction commits or rolls back. PostgreSQL has no lock escalation. **MySQL/InnoDB** uses MVCC for consistent reads and row-level locks for writes; ordinary `SELECT` reads a consistent snapshot without locking, while `UPDATE` locks the rows it examines. MySQL has a default lock wait timeout of 50 seconds (`innodb_lock_wait_timeout`). **SQL Server** (under the default pessimistic `READ COMMITTED` isolation) takes shared locks on rows read by `SELECT` and update locks on rows examined by DML; these can block readers and writers. With `READ_COMMITTED_SNAPSHOT` (RCSI), readers use row versions and do not block writers. SQL Server can escalate row locks to page or table locks if too many locks are held on a single object.

**Beginner-Friendly Explanation**
Locking is the traffic control system for updates. An `UPDATE` locks the rows it is changing so that two users cannot modify the same row at the same time. In PostgreSQL and MySQL, readers usually are not blocked by writers. In SQL Server without RCSI, readers can be blocked. Long-running updates that hold locks for a long time can block other users, which is why batching is important.

#### Purposes

- To understand why an `UPDATE` might block or be blocked by other sessions.
- To choose isolation levels and batching strategies that minimise blocking.
- To set appropriate lock timeouts to avoid indefinite waits.

#### Syntax Rules and Structure

Locking is not directly controlled by `UPDATE` syntax (except through hints in some systems), but by isolation level and session settings:

- **MySQL**: `SET SESSION innodb_lock_wait_timeout = 10;` to limit lock waits.
- **PostgreSQL**: `SET lock_timeout = '3s';` to limit lock waits.
- **SQL Server**: Use `READ_COMMITTED_SNAPSHOT` or `SNAPSHOT` isolation to reduce reader-writer blocking; use query hints (`UPDLOCK`, `ROWLOCK`) when necessary.

**Constraints and limitations:**

- Lock timeouts cause the statement to fail with an error; the application must handle the error and retry if appropriate.
- In SQL Server, lock escalation can escalate row locks to a table lock, blocking all access; `ALTER TABLE ... SET (LOCK_ESCALATION = AUTO | TABLE | DISABLE)` can control this.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Setting lock_timeout before an UPDATE

```sql
-- Step 1: Set a short lock timeout.
SET lock_timeout = '200ms';

-- Step 2: Attempt an UPDATE that might be blocked.
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
```

**Expected output** (if blocked):

```text
SET
ERROR:  canceling statement due to lock timeout
```

**Why this output appears**: If another transaction holds a conflicting lock on row `id = 1`, PostgreSQL waits up to 200ms, then fails with a lock timeout error rather than blocking indefinitely.

#### Real-World Cases

- **High-concurrency OLTP**: An application sets a short lock timeout on updates and retries on timeout, preventing a single slow transaction from stalling the system.
- **Reporting vs. writes**: SQL Server databases with heavy reporting use `READ_COMMITTED_SNAPSHOT` so that long-running reports do not block `UPDATE` statements.

---

### 15. Safe Update Practices

#### Definitions

**Core Definition**
Safe update practices are the guidelines and safeguards used to prevent accidental or destructive `UPDATE` statements in production environments, including mandatory `WHERE` clauses, batching, pre-checks, and database-level safety modes.

**Technical Definition**
Safe update practices include: (1) **always include a `WHERE` clause**—unconditional `UPDATE` is prohibited in production guidelines; (2) **use `sql_safe_updates`** (MySQL) or `pg-safeupdate` (PostgreSQL) to enforce that `UPDATE` and `DELETE` use a key in the `WHERE` clause or a `LIMIT`; (3) **check before you act**—run a `SELECT` with the same `WHERE` clause to verify the affected row count before executing the update; (4) **batch large updates** to limit lock hold time and replication impact; (5) **update the audit timestamp** (`update_time`) alongside the data change; (6) **use transactions** and `ROLLBACK` to test an update before committing; and (7) **back up critical data** or use soft deletes instead of physical deletes where appropriate.

**Beginner-Friendly Explanation**
Safe update practices are the habits that prevent you from accidentally overwriting an entire table. Always use a `WHERE` clause, check how many rows your update will affect before running it, and break large changes into small batches.

#### Purposes

- To prevent accidental full-table updates that destroy or corrupt data.
- To limit the blast radius of a mistake.
- To keep production systems responsive during data changes.
- To maintain an audit trail of changes.

#### Syntax Rules and Structure

**MySQL `sql_safe_updates`:**

```sql
SET SESSION sql_safe_updates = 1;
-- Now UPDATE without a WHERE clause or LIMIT fails.
```


**PostgreSQL `pg-safeupdate`:**

```sql
LOAD 'pg_safeupdate';
-- Now UPDATE without a WHERE clause fails.
```


**Pre-check pattern:**

```sql
-- Step 1: Check the affected rows.
SELECT COUNT(*) FROM table_name WHERE condition;

-- Step 2: Execute the update.
UPDATE table_name SET column = value WHERE condition;
```


**Component breakdown:**

| Practice | Implementation |
|----------|----------------|
| Mandatory `WHERE` | Application code review, `sql_safe_updates`, `pg-safeupdate` |
| Pre-check row count | `SELECT COUNT(*)` with the same `WHERE` clause |
| Batching | `LIMIT` / `TOP` / subquery batches |
| Audit timestamp | `SET update_time = NOW()` |
| Soft delete | `UPDATE ... SET deleted_at = NOW()` |

**Syntax rules:**

- `sql_safe_updates` requires a `WHERE` clause that uses a key column or a `LIMIT` clause.
- `pg-safeupdate` prevents `UPDATE` and `DELETE` without a `WHERE` clause.

**Constraints and limitations:**

- `sql_safe_updates` can be disabled per session; it is not a hard security boundary.
- Pre-checks are point-in-time; the actual update may affect a different number of rows if the data changes between the check and the update (use a transaction with appropriate isolation to avoid this).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Pre-check and safe update

```sql
-- Step 1: Enable safe updates for the session.
SET SESSION sql_safe_updates = 1;

-- Step 2: Pre-check the affected rows.
SELECT COUNT(*) AS will_update FROM users WHERE status = 1 AND created_at < '2023-01-01';

-- Step 3: Execute the update with a LIMIT.
UPDATE users
SET status = 2, update_time = NOW()
WHERE status = 1 AND created_at < '2023-01-01'
LIMIT 100;

-- Step 4: Verify.
SELECT COUNT(*) AS still_pending FROM users WHERE status = 1 AND created_at < '2023-01-01';
```

**Expected output:**

```text
Query OK, 0 rows affected (0.00 sec)
+-------------+
| will_update |
+-------------+
|        5000 |
+-------------+
Query OK, 100 rows affected (0.01 sec)
Rows matched: 100  Changed: 100  Warnings: 0
+---------------+
| still_pending |
+---------------+
|          4900 |
+---------------+
```

**Why this output appears**: The pre-check confirms 5,000 rows are eligible. The `UPDATE ... LIMIT 100` updates only 100 of them, leaving 4,900 for subsequent batches. `sql_safe_updates = 1` would have rejected an unconditional `UPDATE`.

#### Real-World Cases

- **Production data fix**: A DBA runs a `SELECT COUNT(*)` with the intended `WHERE` clause, confirms the count matches expectations, then executes the batched `UPDATE` with a `LIMIT`.
- **Compliance**: An organisation enforces `sql_safe_updates` on all MySQL sessions used for ad-hoc queries, preventing developers from accidentally running `UPDATE users SET role = 'admin'` without a `WHERE` clause.

---

## References

- PostgreSQL Documentation – UPDATE – https://www.postgresql.org/docs/18/sql-update.html
- PostgreSQL Documentation – UPDATE (man page, PostgreSQL 18.6) – https://manpages.debian.org/testing/postgresql-client-18/UPDATE.7.en.html
- MySQL 8.0 Reference Manual – UPDATE Statement – https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/update.html#1
- Microsoft Learn – UPDATE (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/queries/update-transact-sql
- Microsoft Learn – OUTPUT Clause (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/queries/output-clause-transact-sql
- MySQL 8.0 Reference Manual – Optimizing Subqueries, Derived Tables, View References, and Common Table Expressions – https://dev.mysql.com/doc/refman/8.0/en/subquery-optimization.html
- MySQL DML 数据变更指南 (ByteDance AgentKit Samples) – https://github.com/bytedance/agentkit-samples/blob/9fc58da287b9b2d5910047ca8519efea597189cb/skills/database-skill/references/develop/mysql/dml-guide.md
- OneUptime Blog – MySQL UPDATE with LIMIT – https://github.com/OneUptime/blog/blob/master/posts/2026-03-31-mysql-update-with-limit.md
- Microsoft Docs – Performing batched updates – https://learn.microsoft.com/en-us/archive/blogs/sqlserverfaq/performing-batched-updates
- PostgreSQL Mailing List – Bulk DML performance (batch commit guidance) – https://www.postgresql.org/message-id/flat/...
- Stack Overflow – Does a transaction lock linked tables? (lock escalation discussion) – https://stackoverflow.com/questions/…
- Bytebase – How to Handle Ad-hoc Database Data Changes? – https://www.bytebase.com/blog/how-to-handle-ad-hoc-database-data-changes/
- Cybrosys – How to Prevent Accidental Full Table Updates in PostgreSQL 18 Using pg-safeupdate – https://www.cybrosys.com/blog/how-to-prevent-accidental-full-table-updates-in-postgresql-18-using-pg-safeupdate
- GitHub – Safe DML Patterns for Agents – https://github.com/oracle/skills/blob/main/skills/db/agent/safe-dml-patterns.md