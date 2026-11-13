# Comprehensive Programming Cheat Sheet: SQL Altering Tables

## Topic Overview

### Definitions

**Core Definition**
Altering tables is the Data Definition Language (DDL) process of modifying an existing table's structure using the `ALTER TABLE` statement, including adding, dropping, or renaming columns, changing data types, adding or dropping constraints, renaming or moving the table, and managing default values.

**Technical Definition**
`ALTER TABLE` modifies the definition of an existing relation by applying one or more *actions*—each action being a column operation (`ADD COLUMN`, `DROP COLUMN`, `ALTER COLUMN`), a constraint operation (`ADD CONSTRAINT`, `DROP CONSTRAINT`), or a table-level operation (`RENAME TO`, `SET SCHEMA`). In PostgreSQL, `ALTER TABLE` acquires an `ACCESS EXCLUSIVE` lock for most actions, blocking all concurrent reads and writes for the duration of the operation. In MySQL, the `ALGORITHM` and `LOCK` clauses control whether an alteration is performed instantly, in-place, or via a full table copy, and whether concurrent DML is permitted. In SQL Server, `ALTER TABLE` acquires a schema-modification (Sch-M) lock that is incompatible with all other locks, including plain reads.

**Beginner-Friendly Explanation**
Altering a table is like renovating a building while people are still inside. You might add a new room (column), knock down a wall (drop a column), change the plumbing (alter a data type), or put up a new safety rail (add a constraint). The challenge is doing this without forcing everyone to evacuate—and different database systems give you different tools to manage that disruption.

### Key Characteristics

- **Locking is the central concern**: `ALTER TABLE` operations typically acquire strong locks that block reads and writes; the degree and duration of blocking vary by RDBMS and by the specific action.
- **Table rewrites vs. metadata-only changes**: Some actions (e.g., adding a nullable column, renaming) are metadata-only and are nearly instantaneous; others (e.g., changing a column type, adding a constraint that validates existing data) may rewrite the entire table and its indexes.
- **Vendor-specific online DDL capabilities**: MySQL offers `ALGORITHM=INSTANT`, `INPLACE`, and `COPY` with fine-grained `LOCK` control. PostgreSQL 11+ makes `ADD COLUMN` with a constant default metadata-only, but most other `ALTER TABLE` forms still take `ACCESS EXCLUSIVE` locks. SQL Server Enterprise Edition supports `ONLINE = ON` for some operations, but many `ALTER TABLE` actions still require a Sch-M lock.
- **Irreversibility**: Most `ALTER TABLE` operations are not automatically reversible; dropping a column or constraint permanently removes data or integrity rules.
- **Production risk**: An `ALTER TABLE` that takes minutes on a small table can take hours on a large production table, causing an outage if the lock blocks application queries.

### Prerequisites

- A running RDBMS instance (PostgreSQL, MySQL, or SQL Server) and a client connection.
- Ownership of the table or `ALTER` privilege on it.
- For `ALTER TABLE ... ADD CONSTRAINT ... FOREIGN KEY`: `REFERENCES` privilege on the referenced table.
- For `ALTER TABLE ... SET SCHEMA` (PostgreSQL): `CREATE` privilege on the target schema.
- Awareness of the locking behaviour and online DDL capabilities of your specific RDBMS version.

### Related Programming Areas

- Data Definition Language (DDL) – `CREATE`, `ALTER`, `DROP`.
- Schema versioning and migration – `ALTER TABLE` is the primary vehicle for schema evolution.
- Query performance tuning – adding indexes and changing column types affect execution plans.
- Production operations – zero-downtime deployment and expand-contract migration patterns.
- Locking and concurrency – understanding lock modes is essential for safe schema changes.

---

## Core Concepts and Features

### 1. `ALTER TABLE`

#### Definitions

**Core Definition**
`ALTER TABLE` is the SQL statement used to change the definition of an existing table.

**Technical Definition**
`ALTER TABLE` accepts a table name and one or more *actions* separated by commas. Actions include column operations (`ADD COLUMN`, `DROP COLUMN`, `ALTER COLUMN`, `RENAME COLUMN`), constraint operations (`ADD CONSTRAINT`, `DROP CONSTRAINT`, `VALIDATE CONSTRAINT`), and table-level operations (`RENAME TO`, `SET SCHEMA`, `SET TABLESPACE`). PostgreSQL's `ALTER TABLE` synopsis lists the full grammar, including `RENAME`, `SET SCHEMA`, `ATTACH PARTITION`, and column-specific `ALTER` forms. MySQL's `ALTER TABLE` supports `ADD`, `DROP`, `CHANGE`, `MODIFY`, `RENAME`, and the `ALGORITHM` and `LOCK` clauses. SQL Server's `ALTER TABLE` supports `ALTER COLUMN`, `ADD`, `DROP`, and `WITH (ONLINE = ON | OFF)` for some operations.

**Beginner-Friendly Explanation**
`ALTER TABLE` is the master command for changing a table after it has been created. You use it to add or remove columns, change their types, rename things, and add or remove rules.

#### Purposes

- To evolve a table's structure as application requirements change.
- To add, remove, or modify columns without dropping and recreating the table.
- To add or remove constraints that enforce data integrity.
- To rename or relocate a table without losing its data.
- To manage default values for columns.

#### Syntax Rules and Structure

**PostgreSQL (general outline):**

```sql
ALTER TABLE [ IF EXISTS ] [ ONLY ] name [ * ] action [, ... ]
-- where action includes:
--   ADD [ COLUMN ] column_name data_type [ column_constraint [ ... ] ]
--   DROP [ COLUMN ] column_name [ RESTRICT | CASCADE ]
--   ALTER [ COLUMN ] column_name [ SET DATA ] TYPE data_type [ USING expression ]
--   ALTER [ COLUMN ] column_name SET DEFAULT expression
--   ALTER [ COLUMN ] column_name DROP DEFAULT
--   RENAME [ COLUMN ] column_name TO new_column_name
--   ADD table_constraint
--   DROP CONSTRAINT constraint_name [ RESTRICT | CASCADE ]
--   RENAME TO new_name
--   SET SCHEMA new_schema
```


**MySQL (general outline):**

```sql
ALTER TABLE tbl_name
    [alter_option [, alter_option] ...]
-- where alter_option includes:
--   ADD [COLUMN] col_name column_definition
--   DROP [COLUMN] col_name
--   CHANGE [COLUMN] old_col_name new_col_name column_definition
--   MODIFY [COLUMN] col_name column_definition
--   ALTER [COLUMN] col_name { SET DEFAULT literal | DROP DEFAULT }
--   RENAME [TO | AS] new_tbl_name
--   ADD [CONSTRAINT [symbol]] PRIMARY KEY / UNIQUE / FOREIGN KEY / CHECK
--   DROP {INDEX | KEY | PRIMARY KEY | FOREIGN KEY | CHECK} ...
--   ALGORITHM [=] {DEFAULT | INSTANT | INPLACE | COPY}
--   LOCK [=] {DEFAULT | NONE | SHARED | EXCLUSIVE}
```


**SQL Server (general outline):**

```sql
ALTER TABLE table_name
{
    ALTER COLUMN column_name data_type [ NULL | NOT NULL ]
  | ADD column_definition | table_constraint
  | DROP { COLUMN column_name | CONSTRAINT constraint_name }
  | WITH { CHECK | NOCHECK } ...
  | WITH ( ONLINE = ON | OFF )
}
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `table_name` | The table to be altered; can be schema-qualified. |
| `action` / `alter_option` | One or more modifications to apply. |
| `IF EXISTS` (PostgreSQL) | Suppresses the error if the table does not exist. |
| `ONLY` (PostgreSQL) | Restricts the operation to the named table, excluding inheritance descendants. |
| `ALGORITHM` / `LOCK` (MySQL) | Controls the DDL execution strategy and concurrency level. |
| `ONLINE = ON` (SQL Server) | Requests an online operation where supported. |

**Syntax rules:**

- Multiple actions can be combined in a single `ALTER TABLE` statement in PostgreSQL and MySQL; SQL Server generally processes one action per statement.
- In PostgreSQL, `RENAME` and `SET SCHEMA` cannot be combined with other actions in the same statement.
- MySQL's `ALGORITHM` and `LOCK` clauses are placed at the end of the statement, separated by commas.

**Constraints and limitations:**

- `ALTER TABLE` acquires locks that may block concurrent reads and writes; the lock mode depends on the RDBMS and the specific action.
- Some actions require table rewrites, which can be slow and I/O-intensive on large tables.
- Dropping a column with `CASCADE` can remove dependent objects (views, foreign keys) in other tables.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Multiple actions in one statement

```sql
-- Step 1: Create a table.
CREATE TABLE employees (
    employee_id serial PRIMARY KEY,
    first_name  text NOT NULL,
    last_name   text NOT NULL
);

-- Step 2: Add a column, set a default, and add a constraint in one ALTER TABLE.
ALTER TABLE employees
    ADD COLUMN email text,
    ALTER COLUMN email SET DEFAULT 'unknown@example.com',
    ADD CONSTRAINT uq_employee_email UNIQUE (email);
```

**Expected output:**

```text
CREATE TABLE
ALTER TABLE
```

**Why this output appears**: PostgreSQL applies all three actions atomically within a single `ALTER TABLE` statement. The new column is added, the default is registered, and the unique constraint is created.

##### Example 2: MySQL – ALTER TABLE with ALGORITHM and LOCK

```sql
-- Step 1: Create a table.
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name       VARCHAR(255) NOT NULL
);

-- Step 2: Add an index using INPLACE algorithm and permit concurrent DML.
ALTER TABLE products
    ADD INDEX idx_name (name),
    ALGORITHM=INPLACE,
    LOCK=NONE;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

**Why this output appears**: The `ALGORITHM=INPLACE` clause requests an in-place index build, and `LOCK=NONE` permits concurrent reads and writes during the operation. MySQL uses the requested algorithm because it is supported for adding an index.

#### Real-World Cases

- **Feature development**: A new feature requires an additional `phone_number` column; `ALTER TABLE ... ADD COLUMN` adds it without downtime if the column is nullable.
- **Compliance**: A `CHECK` constraint is added to enforce that `salary > 0`; the constraint is added as `NOT VALID` first, then validated separately to avoid a long lock.
- **Refactoring**: A column is renamed as part of a domain model change; the old name is kept as a view or generated column during a transition period.

---

### 2. Adding Columns

#### Definitions

**Core Definition**
Adding a column extends a table with a new attribute, initially populated with `NULL` or a specified default value for all existing rows.

**Technical Definition**
`ALTER TABLE ... ADD COLUMN column_name data_type [ column_constraint ]` adds a new column to the table. In PostgreSQL 11 and later, adding a column with a *non-volatile* constant default is a metadata-only operation: the default is stored in the table's metadata and applied lazily when rows are read, avoiding a full table rewrite. Adding a column with a *volatile* default (e.g., `clock_timestamp()`) or with `NOT NULL` without a default requires a table rewrite. In MySQL, adding a column can be `INSTANT` (metadata-only, MySQL 8.0.12+ for columns added at the end), `INPLACE`, or `COPY`, depending on the column position and attributes. In SQL Server, adding a nullable column or a `NOT NULL` column with a constant default on Enterprise Edition is a metadata-only operation.

**Beginner-Friendly Explanation**
Adding a column is like adding a new field to a form. Existing records get a blank or default value for that field; you do not need to fill it in manually for every record.

#### Purposes

- To extend a table with new data that was not anticipated in the original design.
- To add derived or computed columns for reporting or application convenience.
- To add audit or tracking columns such as `created_at` or `updated_by`.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
ALTER TABLE table_name ADD [ COLUMN ] [ IF NOT EXISTS ] column_name data_type
    [ COLLATE collation ] [ column_constraint [ ... ] ];
```


**MySQL:**

```sql
ALTER TABLE tbl_name ADD [COLUMN] col_name column_definition [FIRST | AFTER col_name];
```


**SQL Server:**

```sql
ALTER TABLE table_name ADD column_name data_type [ NULL | NOT NULL ] [ DEFAULT constant ];
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `column_name` | Name of the new column. |
| `data_type` | Data type of the new column. |
| `IF NOT EXISTS` (PostgreSQL) | Suppresses the error if the column already exists. |
| `FIRST` / `AFTER` (MySQL) | Position of the new column; affects whether the operation can be INSTANT. |
| `DEFAULT` | Default value for existing and future rows. |

**Syntax rules:**

- In PostgreSQL, adding a column with a non-volatile default is metadata-only (PG 11+); volatile defaults require a table rewrite.
- In MySQL, adding a column at the end of the table can use `ALGORITHM=INSTANT` (8.0.12+); adding a column in the middle requires `INPLACE` or `COPY`.
- In SQL Server, adding a `NOT NULL` column without a default fails if the table contains rows.

**Constraints and limitations:**

- Adding a column with `NOT NULL` and no default fails if the table is not empty (unless the RDBMS supports a metadata-only default).
- Adding a column with a volatile default (e.g., `now()`, `random()`) requires a table rewrite in PostgreSQL.
- In MySQL, adding a column in a specific position (`FIRST` or `AFTER`) prevents `INSTANT` algorithm and may require `INPLACE` or `COPY`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Metadata-only ADD COLUMN with default

```sql
-- Step 1: Create a table with existing rows.
CREATE TABLE measurements (id serial PRIMARY KEY, value numeric);
INSERT INTO measurements (value) SELECT random() FROM generate_series(1, 1000000);

-- Step 2: Add a column with a constant default (metadata-only in PG 11+).
ALTER TABLE measurements ADD COLUMN unit text DEFAULT 'metric';
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1000000
ALTER TABLE
```

**Why this output appears**: Because the default `'metric'` is a non-volatile constant, PostgreSQL 11+ stores it as metadata without rewriting the table. The operation completes quickly even on a table with one million rows.

##### Example 2: MySQL – Instant ADD COLUMN at the end

```sql
-- Step 1: Create a table.
CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(100));

-- Step 2: Add a column at the end with ALGORITHM=INSTANT.
ALTER TABLE users ADD COLUMN email VARCHAR(255), ALGORITHM=INSTANT;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

**Why this output appears**: MySQL 8.0.12+ supports `ALGORITHM=INSTANT` for adding a column at the end of the table. The operation modifies only the data dictionary, making it nearly instantaneous regardless of table size.

#### Real-World Cases

- **Feature flag rollout**: A `beta_enabled` boolean column is added with a default of `false`; the metadata-only operation allows the migration to complete in milliseconds.
- **Audit fields**: `created_at timestamptz DEFAULT now()` is added to an existing table; because `now()` is stable (not volatile) in PostgreSQL, the column can be added without a rewrite.

---

### 3. Dropping Columns

#### Definitions

**Core Definition**
Dropping a column removes an attribute and all its data from a table permanently.

**Technical Definition**
`ALTER TABLE ... DROP COLUMN column_name [ RESTRICT | CASCADE ]` removes the column from the table's definition and marks its storage as free. In PostgreSQL, the column is not physically removed from existing rows until the table is rewritten (e.g., by `VACUUM FULL`); the `attisdropped` flag is set in `pg_attribute`. `CASCADE` drops dependent objects such as views, foreign-key constraints, and indexes that reference the column. MySQL's `DROP COLUMN` removes the column and its data; it is an in-place operation for InnoDB but may rebuild the table. SQL Server's `DROP COLUMN` fails if the column is referenced by a constraint, index, or computed column.

**Beginner-Friendly Explanation**
Dropping a column is like deleting a field from a form. All the data that was stored in that field is gone, and anything that depended on it (like a report that used the field) may break.

#### Purposes

- To remove obsolete or unused columns after a refactoring.
- To reclaim storage occupied by large, unused columns.
- To simplify a table by removing redundant attributes.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
ALTER TABLE table_name DROP [ COLUMN ] [ IF EXISTS ] column_name [ RESTRICT | CASCADE ];
```


**MySQL:**

```sql
ALTER TABLE tbl_name DROP [COLUMN] col_name;
```


**SQL Server:**

```sql
ALTER TABLE table_name DROP COLUMN column_name;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IF EXISTS` (PostgreSQL) | Suppresses the error if the column does not exist. |
| `RESTRICT` | Refuses to drop if dependent objects exist (default in PostgreSQL). |
| `CASCADE` | Drops dependent objects (views, constraints) automatically. |

**Syntax rules:**

- `CASCADE` can remove objects in other tables (e.g., foreign-key constraints in referencing tables).
- In PostgreSQL, dropping a column is metadata-only (the data remains until the table is rewritten).
- In SQL Server, the column must not be referenced by any constraint or index.

**Constraints and limitations:**

- `DROP COLUMN ... CASCADE` can have far-reaching effects; always inspect dependencies first.
- Dropping a column does not immediately reclaim disk space in PostgreSQL; a `VACUUM FULL` or table rewrite is required.
- In MySQL, dropping a column may rebuild the table if `ALGORITHM=COPY` is used.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Drop a column with CASCADE

```sql
-- Step 1: Create a table and a view that references it.
CREATE TABLE orders (order_id int PRIMARY KEY, legacy_note text);
CREATE VIEW order_notes AS SELECT order_id, legacy_note FROM orders;

-- Step 2: Drop the column with CASCADE.
ALTER TABLE orders DROP COLUMN legacy_note CASCADE;
```

**Expected output:**

```text
CREATE TABLE
CREATE VIEW
NOTICE:  drop cascades to view order_notes
ALTER TABLE
```

**Why this output appears**: `CASCADE` instructs PostgreSQL to drop the dependent view `order_notes` before dropping the column. Without `CASCADE`, the statement would fail because the view depends on the column.

#### Real-World Cases

- **Deprecation**: A `legacy_status` column is dropped after the application has fully migrated to a new `status_id` column.
- **Storage recovery**: A large `blob_data` column is dropped to free storage; a subsequent `VACUUM FULL` reclaims the space.

---

### 4. Renaming Columns

#### Definitions

**Core Definition**
Renaming a column changes its identifier without affecting its data type, constraints, or data.

**Technical Definition**
`ALTER TABLE ... RENAME COLUMN old_name TO new_name` updates the column's name in the system catalogs. In PostgreSQL, this is a metadata-only operation that takes an `ACCESS EXCLUSIVE` lock but does not rewrite the table. MySQL uses `ALTER TABLE ... CHANGE old_name new_name column_definition` or `RENAME COLUMN old_name TO new_name` (MySQL 8.0+). SQL Server uses `EXEC sp_rename 'table.old_name', 'new_name', 'COLUMN'`.

**Beginner-Friendly Explanation**
Renaming a column is like changing the label on a form field. The data inside stays the same; only the name changes.

#### Purposes

- To correct a misspelled or misleading column name.
- To align column names with new naming conventions or domain terminology.
- To make a column name more descriptive or consistent with other tables.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
ALTER TABLE table_name RENAME [ COLUMN ] column_name TO new_column_name;
```


**MySQL:**

```sql
ALTER TABLE tbl_name RENAME COLUMN old_col_name TO new_col_name;
-- or
ALTER TABLE tbl_name CHANGE old_col_name new_col_name column_definition;
```


**SQL Server:**

```sql
EXEC sp_rename 'table_name.old_column_name', 'new_column_name', 'COLUMN';
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `column_name` / `old_col_name` | Existing column name. |
| `new_column_name` / `new_col_name` | Desired new name. |
| `column_definition` (MySQL `CHANGE`) | The full definition of the column; `CHANGE` can also change the type. |

**Syntax rules:**

- The new name must be unique within the table.
- In PostgreSQL, `RENAME COLUMN` is an `ACCESS EXCLUSIVE` operation but does not rewrite the table.
- MySQL's `RENAME COLUMN` (8.0+) is metadata-only; `CHANGE` may rebuild the table if the definition changes.

**Constraints and limitations:**

- Renaming a column does not update references in views, functions, or application code; dependent objects may break.
- In SQL Server, `sp_rename` is not transaction-safe in all cases and may leave objects in an inconsistent state if interrupted.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Rename a column

```sql
-- Step 1: Create a table.
CREATE TABLE customers (id int PRIMARY KEY, fullname text);

-- Step 2: Rename the column.
ALTER TABLE customers RENAME COLUMN fullname TO full_name;

-- Step 3: Verify.
SELECT column_name FROM information_schema.columns
WHERE table_name = 'customers' ORDER BY ordinal_position;
```

**Expected output:**

```text
CREATE TABLE
ALTER TABLE
 column_name
-------------
 id
 full_name
(2 rows)
```

**Why this output appears**: The rename updates `pg_attribute.attname`; the column's data and constraints are unaffected. The metadata query confirms the new name.

#### Real-World Cases

- **Naming convention migration**: `tbl_cust_nm` is renamed to `customer_name` to follow snake_case conventions.
- **Domain refactoring**: `product_no` is renamed to `product_id` after the domain model standardises on `_id` suffixes.

---

### 5. Changing Data Types

#### Definitions

**Core Definition**
Changing a data type alters the type of values a column can hold, potentially converting existing data to the new type.

**Technical Definition**
`ALTER TABLE ... ALTER COLUMN column_name [ SET DATA ] TYPE new_data_type [ USING expression ]` changes the column's type. PostgreSQL requires an explicit `USING` clause when there is no implicit or assignment cast from the old type to the new type; the `USING` expression specifies how to compute the new value from the old. In MySQL, `ALTER TABLE ... MODIFY COLUMN column_name new_data_type` or `CHANGE` changes the type. In SQL Server, `ALTER TABLE ... ALTER COLUMN column_name new_data_type` changes the type; the column must not have dependencies that prevent the change.

**Beginner-Friendly Explanation**
Changing a data type is like converting a form field from “text” to “number.” Existing values must be convertible; otherwise, the change fails.

#### Purposes

- To accommodate larger values (e.g., `varchar(100)` to `varchar(255)`).
- To change the precision of numeric columns (e.g., `numeric(10,2)` to `numeric(12,4)`).
- To migrate from one type to another as part of a schema refactoring (e.g., `integer` to `bigint`).
- To enforce stricter type constraints.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
ALTER TABLE table_name ALTER [ COLUMN ] column_name [ SET DATA ] TYPE new_data_type
    [ COLLATE collation ] [ USING expression ];
```


**MySQL:**

```sql
ALTER TABLE tbl_name MODIFY [COLUMN] col_name new_data_type;
-- or
ALTER TABLE tbl_name CHANGE [COLUMN] old_col_name new_col_name new_data_type;
```


**SQL Server:**

```sql
ALTER TABLE table_name ALTER COLUMN column_name new_data_type [ NULL | NOT NULL ];
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `TYPE` / `MODIFY` / `ALTER COLUMN` | The clause that initiates the type change. |
| `new_data_type` | The target data type. |
| `USING expression` (PostgreSQL) | An expression that converts old values to the new type. |
| `COLLATE` (PostgreSQL) | Optional collation for character types. |

**Syntax rules:**

- In PostgreSQL, if no implicit or assignment cast exists, a `USING` clause is mandatory.
- In MySQL, narrowing a type (e.g., `varchar(255)` to `varchar(100)`) truncates data if values exceed the new length.
- In SQL Server, `ALTER COLUMN` cannot change a column with a `PRIMARY KEY` or `UNIQUE` constraint without dropping the constraint first.

**Constraints and limitations:**

- Changing a column's type generally requires a table rewrite (except for certain metadata-only changes such as widening `varchar` in some systems).
- In PostgreSQL, a type change takes an `ACCESS EXCLUSIVE` lock and rewrites the table.
- Data may be truncated or lost if the new type cannot accommodate existing values.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Change type with a USING clause

```sql
-- Step 1: Create a table with a text column.
CREATE TABLE products (id serial PRIMARY KEY, price text);
INSERT INTO products (price) VALUES ('19.99'), ('29.95');

-- Step 2: Change the column type from text to numeric with an explicit USING.
ALTER TABLE products ALTER COLUMN price TYPE numeric(10,2) USING price::numeric;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 2
ALTER TABLE
```

**Why this output appears**: The `USING price::numeric` clause explicitly converts each text value to a numeric value. Without the `USING` clause, PostgreSQL would attempt an assignment cast, which may not exist for `text` to `numeric`.

#### Real-World Cases

- **Scaling identifiers**: An `integer` primary key is changed to `bigint` as the table approaches the integer limit.
- **Precision increase**: A `numeric(10,2)` column is widened to `numeric(12,4)` to accommodate larger transactions with more decimal places.

---

### 6. Adding Constraints

#### Definitions

**Core Definition**
Adding a constraint attaches a new integrity rule to a table, such as a primary key, foreign key, unique constraint, or check constraint.

**Technical Definition**
`ALTER TABLE ... ADD CONSTRAINT constraint_name constraint_type ...` adds a constraint to an existing table. In PostgreSQL, constraints can be added as `NOT VALID`, which skips validation of existing rows and takes only a brief `ACCESS EXCLUSIVE` lock; the constraint can later be validated with `VALIDATE CONSTRAINT`, which takes a `SHARE UPDATE EXCLUSIVE` lock and does not block reads or writes. MySQL's `ADD CONSTRAINT` supports `PRIMARY KEY`, `UNIQUE`, `FOREIGN KEY`, and `CHECK`. SQL Server's `ADD CONSTRAINT` supports the same constraint types plus `DEFAULT`.

**Beginner-Friendly Explanation**
Adding a constraint is like installing a safety rail. It makes sure that future data follows a rule; adding it as `NOT VALID` first lets you install the rail without blocking everyone while you check the existing data.

#### Purposes

- To enforce data integrity rules that were not present when the table was created.
- To add referential integrity after a data import or migration.
- To add uniqueness constraints for new business rules.
- To add `CHECK` constraints that validate domain-specific conditions.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
ALTER TABLE table_name ADD [ CONSTRAINT constraint_name ]
    { PRIMARY KEY ( columns ) | UNIQUE ( columns )
    | CHECK ( expression ) [ NOT VALID ]
    | FOREIGN KEY ( columns ) REFERENCES reftable ( refcolumns ) [ NOT VALID ] };
```


**MySQL:**

```sql
ALTER TABLE tbl_name ADD [CONSTRAINT [symbol]]
    { PRIMARY KEY (key_part,...) | UNIQUE (key_part,...)
    | FOREIGN KEY (col_name,...) reference_definition
    | CHECK (expr) };
```


**SQL Server:**

```sql
ALTER TABLE table_name ADD [ CONSTRAINT constraint_name ]
    { PRIMARY KEY | UNIQUE | FOREIGN KEY | CHECK | DEFAULT } ...;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `constraint_name` | Optional name for the constraint; useful for later `DROP CONSTRAINT`. |
| `NOT VALID` (PostgreSQL) | Skips validation of existing rows; the constraint applies to new rows only. |
| `VALIDATE CONSTRAINT` | Validates a `NOT VALID` constraint without blocking reads or writes. |

**Syntax rules:**

- In PostgreSQL, adding a `CHECK` or `FOREIGN KEY` constraint without `NOT VALID` scans the entire table and holds an `ACCESS EXCLUSIVE` lock.
- `NOT VALID` is PostgreSQL-specific and is the recommended approach for large tables.
- Adding a `PRIMARY KEY` requires that the columns are `NOT NULL` and unique.

**Constraints and limitations:**

- Adding a constraint that existing data violates fails unless `NOT VALID` is used (PostgreSQL) or the data is corrected first.
- `NOT VALID` constraints are enforced for new rows but not retroactively validated for existing rows until `VALIDATE CONSTRAINT` is run.
- In MySQL and SQL Server, adding a foreign key constraint validates existing data by default and may be slow on large tables.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Add a CHECK constraint as NOT VALID, then validate

```sql
-- Step 1: Create a table with existing data.
CREATE TABLE accounts (id serial PRIMARY KEY, balance numeric);
INSERT INTO accounts (balance) VALUES (100), (200), (-50);

-- Step 2: Add a CHECK constraint as NOT VALID (brief lock).
ALTER TABLE accounts ADD CONSTRAINT chk_balance_positive CHECK (balance >= 0) NOT VALID;

-- Step 3: Validate the constraint (does not block reads/writes).
ALTER TABLE accounts VALIDATE CONSTRAINT chk_balance_positive;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 3
ALTER TABLE
ERROR:  check constraint "chk_balance_positive" of relation "accounts" is violated by some row
```

**Why this output appears**: The `NOT VALID` constraint is added successfully because existing rows are not checked. However, `VALIDATE CONSTRAINT` scans the table and finds the row with `balance = -50`, which violates the constraint. The validation fails, but the brief `NOT VALID` step did not block the table for an extended period.

#### Real-World Cases

- **Data import validation**: After importing legacy data, foreign-key constraints are added as `NOT VALID` to avoid blocking the import, then validated during a maintenance window.
- **Post-migration integrity**: A `CHECK` constraint is added to enforce a new business rule; the `NOT VALID` approach keeps the application online while the constraint is validated in the background.

---

### 7. Dropping Constraints

#### Definitions

**Core Definition**
Dropping a constraint removes an integrity rule from a table, allowing data that would previously have been rejected.

**Technical Definition**
`ALTER TABLE ... DROP CONSTRAINT constraint_name [ RESTRICT | CASCADE ]` removes the named constraint. In PostgreSQL, `CASCADE` drops dependent objects (e.g., a foreign-key constraint in another table that depends on a unique constraint being dropped). MySQL uses `DROP {PRIMARY KEY | INDEX | FOREIGN KEY | CHECK} constraint_name`. SQL Server uses `DROP CONSTRAINT constraint_name`.

**Beginner-Friendly Explanation**
Dropping a constraint is like removing a safety rail. After it is gone, data that would previously have been rejected can be inserted.

#### Purposes

- To remove obsolete integrity rules that no longer match business requirements.
- To facilitate data migration or refactoring that temporarily violates an existing constraint.
- To drop a constraint before changing the underlying column type.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
ALTER TABLE table_name DROP CONSTRAINT [ IF EXISTS ] constraint_name [ RESTRICT | CASCADE ];
```


**MySQL:**

```sql
ALTER TABLE tbl_name DROP {PRIMARY KEY | INDEX index_name | FOREIGN KEY fk_symbol | CHECK constraint_name};
```


**SQL Server:**

```sql
ALTER TABLE table_name DROP CONSTRAINT constraint_name;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `constraint_name` | The name of the constraint to drop. |
| `IF EXISTS` (PostgreSQL) | Suppresses the error if the constraint does not exist. |
| `CASCADE` (PostgreSQL) | Drops dependent objects automatically. |

**Syntax rules:**

- In MySQL, dropping a primary key requires `DROP PRIMARY KEY` (no name); dropping a foreign key requires the foreign-key symbol.
- In SQL Server, the constraint name must be known; query `sys.objects` or `INFORMATION_SCHEMA.TABLE_CONSTRAINTS` to find it.

**Constraints and limitations:**

- Dropping a primary key or unique constraint may require dropping dependent foreign keys first (unless `CASCADE` is used in PostgreSQL).
- `DROP CONSTRAINT ... CASCADE` can remove constraints in other tables, so it should be used with caution.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Drop a foreign key constraint

```sql
-- Step 1: Create parent and child tables with a foreign key.
CREATE TABLE parent (id int PRIMARY KEY);
CREATE TABLE child (id int PRIMARY KEY, parent_id int REFERENCES parent(id));

-- Step 2: Drop the foreign key constraint.
ALTER TABLE child DROP CONSTRAINT child_parent_id_fkey;
```

**Expected output:**

```text
CREATE TABLE
CREATE TABLE
ALTER TABLE
```

**Why this output appears**: The constraint is dropped, and the child table no longer enforces referential integrity for `parent_id`. Existing data is not validated after the drop.

#### Real-World Cases

- **Refactoring**: A foreign-key constraint is dropped temporarily during a data migration that reorders parent rows, then recreated afterwards.
- **Deprecation**: A `UNIQUE` constraint is dropped when the business rule that required it is removed.

---

### 8. Renaming Tables

#### Definitions

**Core Definition**
Renaming a table changes its identifier without affecting its data, constraints, indexes, or triggers.

**Technical Definition**
PostgreSQL uses `ALTER TABLE old_name RENAME TO new_name`. MySQL uses `ALTER TABLE old_name RENAME new_name` or `RENAME TABLE old_name TO new_name` (the latter can rename multiple tables atomically). SQL Server uses `EXEC sp_rename 'schema.old_name', 'new_name'`.

**Beginner-Friendly Explanation**
Renaming a table is like changing the label on a folder. Everything inside stays the same; only the folder name changes.

#### Purposes

- To correct a misspelled or misleading table name.
- To align table names with new naming conventions.
- To reflect a change in the table's purpose or domain.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
ALTER TABLE [ IF EXISTS ] old_name RENAME TO new_name;
```


**MySQL:**

```sql
ALTER TABLE old_name RENAME [TO | AS] new_name;
-- or
RENAME TABLE old_name TO new_name [, old_name2 TO new_name2 ...];
```


**SQL Server:**

```sql
EXEC sp_rename 'schema.old_name', 'new_name';
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `old_name` | Existing table name. |
| `new_name` | Desired new name. |
| `sp_rename` (SQL Server) | System stored procedure for renaming objects. |

**Syntax rules:**

- The new name must be unique within the schema.
- In PostgreSQL, `RENAME TO` cannot be combined with other actions in the same `ALTER TABLE` statement.
- In SQL Server, the schema name is included in the first argument but not in the second.

**Constraints and limitations:**

- Renaming a table does not update references in views, functions, or application code; dependent objects may break.
- SQL Server's `sp_rename` is not transaction-safe in all cases.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Rename a table

```sql
-- Step 1: Create a table.
CREATE TABLE old_products (id int PRIMARY KEY, name text);

-- Step 2: Rename it.
ALTER TABLE old_products RENAME TO products;
```

**Expected output:**

```text
CREATE TABLE
ALTER TABLE
```

**Why this output appears**: The table's `relname` in `pg_class` is updated; the table's data, indexes, and constraints remain intact, though automatically generated index names may still reference the old table name.

#### Real-World Cases

- **Naming convention migration**: `tbl_cust` is renamed to `customers` to follow the organisation's naming standard.
- **Domain refactoring**: `items` is renamed to `products` after the application's domain model is updated.

---

### 9. Moving Tables

#### Definitions

**Core Definition**
Moving a table relocates it from one schema to another without changing its structure or data.

**Technical Definition**
PostgreSQL uses `ALTER TABLE name SET SCHEMA new_schema` to move a table (and its associated indexes, constraints, and sequences owned by the table) to a different schema. MySQL does not have a separate schema layer; moving a table between databases requires `RENAME TABLE old_db.table TO new_db.table`. SQL Server uses `ALTER SCHEMA new_schema TRANSFER old_schema.table_name` to move a table between schemas.

**Beginner-Friendly Explanation**
Moving a table is like moving a folder from one department's filing cabinet to another's. The contents stay the same, but the table now lives in a different organisational unit.

#### Purposes

- To reorganise database objects into a more appropriate schema.
- To isolate legacy or third-party tables into a dedicated schema.
- To support multi-schema application design and tenant isolation.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
ALTER TABLE [ IF EXISTS ] name SET SCHEMA new_schema;
```


**MySQL:**

```sql
RENAME TABLE old_db.table_name TO new_db.table_name;
```


**SQL Server:**

```sql
ALTER SCHEMA new_schema TRANSFER old_schema.table_name;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `SET SCHEMA` (PostgreSQL) | Moves the table to the specified schema. |
| `RENAME TABLE` (MySQL) | Can move a table between databases (schemas). |
| `ALTER SCHEMA ... TRANSFER` (SQL Server) | Moves the table to the target schema. |

**Syntax rules:**

- In PostgreSQL, `SET SCHEMA` moves associated indexes and sequences owned by the table automatically.
- In MySQL, the user must have `ALTER` and `DROP` on the old table and `CREATE` and `INSERT` on the new table.
- In SQL Server, the user must have `ALTER` permission on the schema and `CONTROL` on the table.

**Constraints and limitations:**

- Moving a table does not update references in views or functions that use the old schema-qualified name.
- In PostgreSQL, `SET SCHEMA` cannot be combined with other `ALTER TABLE` actions in the same statement.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Move a table to a new schema

```sql
-- Step 1: Create a schema and a table.
CREATE SCHEMA archive;
CREATE TABLE public.old_orders (id int PRIMARY KEY, data text);

-- Step 2: Move the table to the archive schema.
ALTER TABLE public.old_orders SET SCHEMA archive;
```

**Expected output:**

```text
CREATE SCHEMA
CREATE TABLE
ALTER TABLE
```

**Why this output appears**: The table's namespace (`relnamespace`) is updated to the OID of the `archive` schema. Associated indexes and owned sequences move with the table.

#### Real-World Cases

- **Legacy isolation**: A legacy application's tables are moved to a `legacy` schema while new development proceeds in `app`.
- **Multi-tenant design**: Tables are moved into per-tenant schemas (`tenant_acme`, `tenant_beta`) as part of a tenant isolation strategy.

---

### 10. Adding Defaults

#### Definitions

**Core Definition**
Adding a default value specifies a value that the database automatically assigns to a column when no value is provided during an `INSERT`.

**Technical Definition**
`ALTER TABLE ... ALTER COLUMN column_name SET DEFAULT expression` sets the default for a column. The default applies only to subsequent `INSERT` and `UPDATE` commands; existing rows are not changed. In PostgreSQL, setting a default is a metadata-only operation. In MySQL, `ALTER TABLE ... ALTER COLUMN column_name SET DEFAULT literal` sets the default. In SQL Server, a default is implemented as a `DEFAULT` constraint added with `ALTER TABLE ... ADD CONSTRAINT ... DEFAULT ... FOR column_name`.

**Beginner-Friendly Explanation**
Adding a default is like telling the database, “If someone leaves this field blank, fill in this value automatically.”

#### Purposes

- To ensure that columns always have a sensible value when the application omits them.
- To automatically record timestamps or status values.
- To simplify application code by centralising default logic in the database.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
ALTER TABLE table_name ALTER [ COLUMN ] column_name SET DEFAULT expression;
```


**MySQL:**

```sql
ALTER TABLE tbl_name ALTER [COLUMN] col_name SET DEFAULT {literal | (expr)};
```


**SQL Server:**

```sql
ALTER TABLE table_name ADD CONSTRAINT constraint_name DEFAULT constant_expression FOR column_name;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `SET DEFAULT` | The clause that assigns a default. |
| `expression` / `literal` | The value or expression to use as the default. |
| `DEFAULT constraint` (SQL Server) | SQL Server implements defaults as named constraints. |

**Syntax rules:**

- The default expression must be compatible with the column's data type.
- In PostgreSQL, the default applies only to subsequent inserts and updates; existing rows are unaffected.
- In SQL Server, the default constraint must be named for later removal.

**Constraints and limitations:**

- Changing a default does not retroactively update existing rows.
- In SQL Server, dropping a default requires dropping the named constraint.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Set a column default

```sql
-- Step 1: Create a table without a default.
CREATE TABLE tasks (id serial PRIMARY KEY, status text);

-- Step 2: Set a default for the status column.
ALTER TABLE tasks ALTER COLUMN status SET DEFAULT 'pending';

-- Step 3: Insert a row without specifying status.
INSERT INTO tasks (id) VALUES (DEFAULT);
SELECT * FROM tasks;
```

**Expected output:**

```text
CREATE TABLE
ALTER TABLE
INSERT 0 1
 id | status
----+---------
  1 | pending
(1 row)
```

**Why this output appears**: The `SET DEFAULT` clause registers `'pending'` as the default for `status`. When the `INSERT` omits `status`, the default is used.

#### Real-World Cases

- **Workflow systems**: A `status` column defaults to `'pending'` so new tasks start in the correct state.
- **Audit fields**: `created_at` defaults to `now()` to automatically timestamp new records.

---

### 11. Removing Defaults

#### Definitions

**Core Definition**
Removing a default drops the automatically assigned value from a column, so future inserts that omit the column receive `NULL` (or fail if the column is `NOT NULL`).

**Technical Definition**
PostgreSQL uses `ALTER TABLE ... ALTER COLUMN column_name DROP DEFAULT`. MySQL uses `ALTER TABLE ... ALTER COLUMN column_name DROP DEFAULT`. SQL Server requires dropping the named `DEFAULT` constraint with `ALTER TABLE ... DROP CONSTRAINT constraint_name`.

**Beginner-Friendly Explanation**
Removing a default is like removing the automatic fill-in rule. After it is gone, leaving the field blank results in `NULL` (or an error if the column cannot be empty).

#### Purposes

- To remove an obsolete default that no longer reflects the desired behaviour.
- To enforce that applications must explicitly supply a value.
- To prepare a column for a type change that is incompatible with the current default.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
ALTER TABLE table_name ALTER [ COLUMN ] column_name DROP DEFAULT;
```


**MySQL:**

```sql
ALTER TABLE tbl_name ALTER [COLUMN] col_name DROP DEFAULT;
```


**SQL Server:**

```sql
ALTER TABLE table_name DROP CONSTRAINT constraint_name;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `DROP DEFAULT` (PostgreSQL, MySQL) | Removes the default directly. |
| `DROP CONSTRAINT` (SQL Server) | Removes the named default constraint. |

**Syntax rules:**

- In PostgreSQL, `DROP DEFAULT` is equivalent to setting the default to `NULL`.
- In SQL Server, the constraint name must be known; query `sys.default_constraints` to find it.

**Constraints and limitations:**

- Removing a default from a `NOT NULL` column means future inserts that omit the column will fail.
- Existing rows are unaffected by removing a default.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Remove a default

```sql
-- Step 1: Create a table with a default.
CREATE TABLE accounts (id serial PRIMARY KEY, balance numeric DEFAULT 0);

-- Step 2: Remove the default.
ALTER TABLE accounts ALTER COLUMN balance DROP DEFAULT;
```

**Expected output:**

```text
CREATE TABLE
ALTER TABLE
```

**Why this output appears**: The default is removed from the column's metadata. Future inserts that omit `balance` will receive `NULL` instead of `0`.

#### Real-World Cases

- **Business rule change**: A `discount` column previously defaulted to `0`; after a policy change, the default is removed so that discounts must be explicitly set.
- **Schema cleanup**: A legacy default is removed before the column is dropped entirely.

---

### 12. Alter Performance

#### Definitions

**Core Definition**
Alter performance refers to the speed and resource consumption of `ALTER TABLE` operations, which depend on whether the operation requires a table rewrite, how much data must be processed, and what locks are held.

**Technical Definition**
`ALTER TABLE` performance is determined by three factors: (1) whether the operation is metadata-only (fast) or requires a table rewrite (slow); (2) the size of the table and its indexes; and (3) the lock mode acquired, which determines whether concurrent queries are blocked. In PostgreSQL, adding a nullable column or a column with a constant default (PG 11+) is metadata-only; changing a column type or adding a validated `CHECK` constraint rewrites the table and takes an `ACCESS EXCLUSIVE` lock. In MySQL, `ALGORITHM=INSTANT` operations are metadata-only, `INPLACE` operations rebuild the table without copying, and `COPY` operations create a new table and copy all rows. In SQL Server, adding a nullable column or a `NOT NULL` column with a constant default on Enterprise Edition is metadata-only; widening a `varchar` is metadata-only, but narrowing it scans the table.

**Beginner-Friendly Explanation**
Some table alterations are like changing a sign on a door—instant and invisible to users. Others are like rebuilding the entire room while everyone waits outside. Knowing which is which helps you avoid accidentally locking a busy production table for hours.

#### Purposes

- To choose the fastest and least disruptive way to apply a schema change.
- To avoid table rewrites on large tables when a metadata-only alternative exists.
- To plan maintenance windows for operations that inherently require a rewrite.

#### Syntax Rules and Structure

Performance characteristics are not controlled by syntax but by the choice of operation and, in MySQL, by the `ALGORITHM` clause. Key principles:

- Prefer metadata-only operations: adding nullable columns, renaming columns, setting/dropping defaults (in PostgreSQL).
- In MySQL, specify `ALGORITHM=INSTANT` or `ALGORITHM=INPLACE` and `LOCK=NONE` to ensure the operation does not block DML.
- In PostgreSQL, use `NOT VALID` constraints and `VALIDATE CONSTRAINT` to avoid long `ACCESS EXCLUSIVE` locks when adding constraints.

**Constraints and limitations:**

- Not all operations support metadata-only execution; changing a column's type or adding a primary key generally requires a rewrite.
- `ALGORITHM=INSTANT` in MySQL has limitations (e.g., adding a column at the end only, no `FULLTEXT` indexes).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Metadata-only vs. rewrite operations

```sql
-- Step 1: Create a table with 1 million rows.
CREATE TABLE big_table (id serial PRIMARY KEY, data text);
INSERT INTO big_table (data) SELECT md5(random()::text) FROM generate_series(1, 1000000);

-- Step 2: Metadata-only: add a nullable column (fast).
ALTER TABLE big_table ADD COLUMN note text;

-- Step 3: Rewrite: change a column type (slow).
ALTER TABLE big_table ALTER COLUMN data TYPE varchar(64);
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1000000
ALTER TABLE
ALTER TABLE
```

**Why this output appears**: Both statements succeed, but the first is metadata-only and returns almost instantly. The second rewrites all one million rows and takes significantly longer, holding an `ACCESS EXCLUSIVE` lock for the duration.

#### Real-World Cases

- **Large-scale migration**: A team adds a `tenant_id` column to a billion-row table using `ADD COLUMN` with a constant default, avoiding a full rewrite.
- **Index maintenance**: An index is added with `CREATE INDEX CONCURRENTLY` (PostgreSQL) or `ALGORITHM=INPLACE, LOCK=NONE` (MySQL) to avoid blocking writes.

---

### 13. Online DDL

#### Definitions

**Core Definition**
Online DDL is a set of techniques that allow table alterations to proceed while concurrent data manipulation (DML) operations continue, minimising downtime.

**Technical Definition**
MySQL's online DDL feature supports `ALGORITHM=INSTANT`, `ALGORITHM=INPLACE`, and `ALGORITHM=COPY`. `INSTANT` operations modify only metadata; `INPLACE` operations rebuild the table without copying it and may permit concurrent DML; `COPY` operations create a new table and copy all rows, blocking DML. The `LOCK` clause controls the level of concurrent access: `LOCK=NONE` permits reads and writes, `LOCK=SHARED` permits reads only, and `LOCK=EXCLUSIVE` blocks all access. PostgreSQL does not have a built-in online DDL mechanism for most `ALTER TABLE` operations; tools such as `pg_repack` and `pg_squeeze` provide online table restructuring, and `CREATE INDEX CONCURRENTLY` provides online index builds. SQL Server provides `ONLINE = ON` for some index and table operations on Enterprise Edition.

**Beginner-Friendly Explanation**
Online DDL is like renovating a shop while it stays open. Customers can still come in and buy things while the work happens in the background.

#### Purposes

- To apply schema changes to high-traffic tables without causing downtime.
- To maintain application availability during migrations.
- To reduce the risk of lock contention and blocked queries.

#### Syntax Rules and Structure

**MySQL:**

```sql
ALTER TABLE tbl_name
    alter_option,
    ALGORITHM [=] {DEFAULT | INSTANT | INPLACE | COPY},
    LOCK [=] {DEFAULT | NONE | SHARED | EXCLUSIVE};
```


**SQL Server (where supported):**

```sql
ALTER TABLE table_name
    operation
    WITH (ONLINE = ON);
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ALGORITHM=INSTANT` (MySQL) | Metadata-only; fastest. |
| `ALGORITHM=INPLACE` (MySQL) | Rebuilds without copying; may allow concurrent DML. |
| `ALGORITHM=COPY` (MySQL) | Creates a new table and copies rows; blocks DML. |
| `LOCK=NONE` (MySQL) | Permits concurrent reads and writes. |
| `ONLINE = ON` (SQL Server) | Requests an online operation where supported. |

**Syntax rules:**

- In MySQL, if the requested algorithm cannot be used, the statement fails immediately rather than falling back to a slower algorithm.
- `LOCK=NONE` halts the operation if the requested concurrency level is not available.

**Constraints and limitations:**

- MySQL's online DDL does not cover all operations; some still require `ALGORITHM=COPY`.
- PostgreSQL lacks a native online DDL mechanism for most `ALTER TABLE` forms; external tools or careful sequencing are required.
- SQL Server's `ONLINE = ON` is available only for certain operations and editions (typically Enterprise).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Online index creation

```sql
-- Step 1: Create a table with data.
CREATE TABLE orders (id INT PRIMARY KEY, customer_id INT, amount DECIMAL(10,2));
INSERT INTO orders SELECT g, g % 1000, random() * 1000 FROM generate_series(1, 100000) AS g;

-- Step 2: Add an index online, permitting concurrent DML.
ALTER TABLE orders
    ADD INDEX idx_customer (customer_id),
    ALGORITHM=INPLACE,
    LOCK=NONE;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.15 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

**Why this output appears**: The index is built in-place while concurrent reads and writes continue. The `LOCK=NONE` clause ensures that the operation does not block DML; if it could not proceed without blocking, it would fail rather than proceed.

#### Real-World Cases

- **High-traffic e-commerce**: An index is added to the `orders` table during business hours using `ALGORITHM=INPLACE, LOCK=NONE`, avoiding a maintenance window.
- **Continuous deployment**: A CI/CD pipeline runs `ALTER TABLE ... ALGORITHM=INSTANT` for schema changes that support it, ensuring zero downtime.

---

### 14. Locking Considerations

#### Definitions

**Core Definition**
Locking considerations refer to the lock modes acquired by `ALTER TABLE` and their impact on concurrent reads and writes.

**Technical Definition**
PostgreSQL's `ALTER TABLE` typically acquires an `ACCESS EXCLUSIVE` lock, which conflicts with every other lock mode, including `ACCESS SHARE` (held by plain `SELECT` statements). This means that an `ALTER TABLE` blocks all reads and writes on the table until the operation completes and the transaction ends. MySQL's online DDL uses metadata locks; an `ALTER TABLE` requests an exclusive metadata lock, and concurrent DML may be blocked depending on the `ALGORITHM` and `LOCK` clauses. SQL Server's `ALTER TABLE` acquires a schema-modification (Sch-M) lock, which is incompatible with all other locks, including plain reads. Lock escalation (SQL Server) can further expand the scope of locking.

**Beginner-Friendly Explanation**
Locking is the traffic control system for schema changes. A strong lock is like closing the entire road; a weaker lock is like allowing one lane to stay open. Different databases use different traffic-control rules, and understanding them is essential to avoid blocking users.

#### Purposes

- To understand why an `ALTER TABLE` might block application queries.
- To choose operations and approaches that minimise lock duration and scope.
- To set appropriate timeouts and use online DDL techniques.

#### Syntax Rules and Structure

Lock modes are not directly specified in SQL (except through MySQL's `LOCK` clause and SQL Server's `ONLINE` option), but they are determined by the operation and the RDBMS. Key lock modes:

| RDBMS | Lock Mode | Blocks |
|-------|-----------|--------|
| PostgreSQL | `ACCESS EXCLUSIVE` | All reads and writes |
| MySQL | Exclusive metadata lock | May block DML depending on `ALGORITHM`/`LOCK` |
| SQL Server | Schema-modification (Sch-M) | All reads and writes |

**Constraints and limitations:**

- In PostgreSQL, the lock is held until the end of the transaction, not just for the duration of the statement.
- The wait for a lock is unbounded by default in PostgreSQL; setting `lock_timeout` prevents indefinite blocking.
- In SQL Server, `ALTER TABLE` cannot use `LOCK_ESCALATION` to avoid the Sch-M lock; the Sch-M lock is inherent to the operation.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Set lock_timeout before DDL

```sql
-- Step 1: Set a short lock timeout for the session.
SET lock_timeout = '100ms';

-- Step 2: Attempt an ALTER TABLE.
ALTER TABLE orders ADD COLUMN currency text NOT NULL DEFAULT 'EUR';
```

**Expected output** (if the lock is not acquired within 100ms):

```text
SET
ERROR:  canceling statement due to lock timeout
```

**Why this output appears**: If another transaction holds a conflicting lock on `orders`, the `ALTER TABLE` waits up to 100ms, then fails with a lock timeout error rather than blocking indefinitely. The deployment runner can catch this error and retry later.

#### Real-World Cases

- **Production safety**: A DBA sets `lock_timeout = '5s'` before running migrations, ensuring that a blocked `ALTER TABLE` fails quickly rather than queueing behind a long-running report query.
- **Lock monitoring**: A monitoring system queries `pg_locks` (PostgreSQL) or `sys.dm_tran_locks` (SQL Server) to detect long-held locks caused by pending `ALTER TABLE` statements.

---

### 15. Schema Modifications in Production

#### Definitions

**Core Definition**
Schema modifications in production are the practices and patterns used to apply `ALTER TABLE` changes safely to live, high-traffic databases without causing outages or data loss.

**Technical Definition**
Production schema modification encompasses the **expand-contract pattern** (also called **parallel change**): a breaking schema change is split into phases—(1) *expand*: add new columns or tables without removing old ones; (2) *migrate*: backfill data and dual-write to both old and new structures; (3) *contract*: remove the old structures after all application instances use the new ones. This pattern ensures that the schema remains compatible with both old and new application versions during a rolling deployment. Other practices include using online DDL where available, adding constraints as `NOT VALID` (PostgreSQL), backfilling in batches to avoid lock escalation, and using tools such as Flyway or Liquibase to version and track migrations.

**Beginner-Friendly Explanation**
Changing a production database is like changing the tyres on a moving car. You cannot stop the car, so you add a spare tyre first (expand), switch to it gradually (migrate), and only then remove the old one (contract). This way, the car never stops moving.

#### Purposes

- To apply schema changes without downtime or data loss.
- To maintain compatibility between old and new application versions during rolling deployments.
- To reduce the risk of lock contention and blocked queries on high-traffic tables.
- To make schema changes reviewable, repeatable, and auditable.

#### Syntax Rules and Structure

The expand-contract pattern is a process, not a single SQL statement. Its phases are:

1. **Expand**: `ALTER TABLE ... ADD COLUMN new_col ...` (without dropping `old_col`).
2. **Migrate**: Backfill `new_col` from `old_col` in batches; application dual-writes to both.
3. **Contract**: `ALTER TABLE ... DROP COLUMN old_col` after all instances use `new_col`.

**Component breakdown:**

| Phase | Action | Example |
|-------|--------|---------|
| Expand | Add new column or table | `ALTER TABLE users ADD COLUMN email_new text;` |
| Migrate | Backfill in batches; dual-write | `UPDATE users SET email_new = email WHERE ...` |
| Contract | Drop old column or table | `ALTER TABLE users DROP COLUMN email;` |

**Syntax rules:**

- Each phase should be a separate migration script, reviewed and deployed independently.
- Backfills should be batched (e.g., 5,000 rows at a time) to avoid long transactions and lock escalation.
- Constraints should be added as `NOT VALID` and validated separately to avoid long `ACCESS EXCLUSIVE` locks.

**Constraints and limitations:**

- The expand-contract pattern increases the total number of migrations and temporarily increases storage usage (both old and new columns exist during the transition).
- Application code must be written to tolerate both the old and new schemas during the transition period.
- Some changes (e.g., changing a primary key type) require more complex multi-step procedures.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Expand-contract pattern for renaming a column

```sql
-- Phase 1: EXPAND — add a new column.
ALTER TABLE customers ADD COLUMN full_name text;

-- Phase 2: MIGRATE — backfill in batches; application dual-writes.
UPDATE customers SET full_name = old_name WHERE full_name IS NULL AND id BETWEEN 1 AND 5000;
-- (repeat for subsequent batches)

-- Phase 3: CONTRACT — after all application instances use full_name, drop old_name.
ALTER TABLE customers DROP COLUMN old_name;
```

**Expected output:**

```text
ALTER TABLE
UPDATE 5000
...
ALTER TABLE
```

**Why this output appears**: Each phase is a separate, low-risk operation. The expand phase is metadata-only; the migrate phase is batched to avoid long locks; the contract phase is a metadata-only drop. No phase blocks reads or writes for an extended period.

#### Real-World Cases

- **Renaming a column without downtime**: A `name` column is renamed to `full_name` using expand-contract, avoiding the instant breakage that a direct `RENAME COLUMN` would cause for old application instances.
- **Changing a column type safely**: A `varchar(100)` column is migrated to `text` by adding a new `text` column, backfilling it, dual-writing, and finally dropping the old column.
- **Adding a NOT NULL column to a large table**: The column is added as nullable, backfilled in batches, then altered to `NOT NULL` with a `CHECK` constraint added as `NOT VALID` and validated separately.

---

## References

- PostgreSQL Documentation – `ALTER TABLE` – https://www.postgresql.org/docs/current/sql-altertable.html
- PostgreSQL Documentation – Modifying Tables – https://www.postgresql.org/docs/current/ddl-alter.html
- PostgreSQL Documentation – Explicit Locking – https://www.postgresql.org/docs/current/explicit-locking.html
- PostgreSQL Documentation – `lock_timeout` – https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-LOCK-TIMEOUT
- PostgreSQL Wiki – Lock Monitoring – https://wiki.postgresql.org/wiki/Lock_Monitoring
- MySQL 8.0 Reference Manual – `ALTER TABLE` Statement – https://dev.mysql.com/doc/refman/8.0/en/alter-table.html
- MySQL 8.0 Reference Manual – InnoDB and Online DDL – https://dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl.html
- MySQL 8.0 Reference Manual – Online DDL Performance and Concurrency – https://dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-performance.html
- Microsoft Learn – `ALTER TABLE` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-table-transact-sql
- Microsoft Learn – `sp_rename` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-rename-transact-sql
- Microsoft Learn – Transaction Locking and Row Versioning Guide – https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide
- Bytebase – SQL Server Schema Migration and Change Management – https://www.bytebase.com/blog/sql-server-schema-migration-guide/
- DbSchema – PostgreSQL Schema Migration: Deploying Changes Safely – https://dbschema.com/blog/postgresql/deploying-postgresql-schema-changes-safely/
- PgFence – ADD COLUMN with a DEFAULT: Sometimes Instant, Sometimes Catastrophic – https://pgfence.com/blog/2026/05/19/add-column-with-a-default-sometimes-instant-sometimes-catastrophic/
- Martin Fowler – Parallel Change (Expand-Contract) – https://martinfowler.com/bliki/ParallelChange.html