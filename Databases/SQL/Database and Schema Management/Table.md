# Comprehensive Programming Cheat Sheet: SQL Table Fundamentals

## Topic Overview

### Definitions

**Core Definition**
Table fundamentals in SQL encompass the Data Definition Language (DDL) statements and catalog queries used to create, inspect, list, rename, truncate, drop, and manage the lifecycle of tables—the primary structures that store relational data.

**Technical Definition**
A table is a named relation consisting of an ordered set of columns (attributes) and an unordered set of rows (tuples). Table fundamentals cover `CREATE TABLE` (with column definitions, data types, constraints, and storage options), catalog introspection via `information_schema` and `pg_catalog`, `ALTER TABLE ... RENAME`, `TRUNCATE TABLE`, `DROP TABLE`, and specialised table types including temporary, unlogged, partitioned, cloned, and derived tables. Syntax and feature availability vary across PostgreSQL, MySQL, and SQL Server, though the core DDL statements are broadly aligned with the SQL standard.

**Beginner-Friendly Explanation**
A table is like a spreadsheet inside a database: it has columns (the categories of information) and rows (the individual records). Table fundamentals are all the commands you need to create that spreadsheet, look at its structure, give it a new name, empty it out, delete it entirely, or create special kinds of spreadsheets that behave differently—like ones that disappear when you log out or ones that are split into smaller pieces for speed.

### Key Characteristics

- **Structure first, data second**: Table DDL defines the container; data manipulation (`INSERT`, `UPDATE`, `DELETE`) operates within that container.
- **Constraint-driven integrity**: Primary keys, foreign keys, `NOT NULL`, `CHECK`, and `UNIQUE` constraints enforce data quality at the table level.
- **Vendor-specific extensions**: While `CREATE TABLE` is standard, features such as `UNLOGGED` (PostgreSQL), `TEMPORARY` semantics, partitioning syntax, and storage engines (MySQL) are implementation-defined.
- **Catalog visibility**: Every table is recorded in system catalogs (`information_schema.tables`, `pg_catalog.pg_tables`, `sys.tables`), enabling introspection and automation.
- **Lifecycle operations are privilege-gated**: Creating, altering, truncating, and dropping tables require specific privileges (ownership, `CREATE`, `TRUNCATE`, `DROP`).

### Prerequisites

- A running RDBMS instance (PostgreSQL, MySQL, or SQL Server) and a client connection.
- An account with `CREATE` privilege on the target schema or database.
- Basic familiarity with SQL data types and constraints.
- For `TRUNCATE` and `DROP`: ownership of the table or elevated privileges (`TRUNCATE` privilege, `DROP` privilege).

### Related Programming Areas

- Data Definition Language (DDL) – `CREATE`, `ALTER`, `DROP`, `TRUNCATE`.
- Data Manipulation Language (DML) – `INSERT`, `SELECT`, `UPDATE`, `DELETE`.
- Query optimisation and indexing – table structure directly affects performance.
- Database migration and schema versioning – tables are the primary objects versioned by Flyway, Liquibase, etc.
- Data modelling and normalisation – table design is the physical realisation of the logical data model.

---

## Core Concepts and Features

### 1. Creating Tables (`CREATE TABLE`)

#### Definitions

**Core Definition**
`CREATE TABLE` defines a new table in the current database by specifying its name, columns, data types, and optional constraints.

**Technical Definition**
`CREATE TABLE` allocates a new relation in the system catalogs, defines its column attributes (name, data type, collation, default, nullability), registers constraints (primary key, foreign key, unique, check, exclusion), and optionally specifies storage parameters, tablespace, partitioning strategy, and table persistence (`TEMPORARY`, `UNLOGGED`). The statement is part of the SQL standard, with vendor-specific extensions.

**Beginner-Friendly Explanation**
This is the command that creates a new spreadsheet. You tell the database what to call it, what columns it should have, what type of data each column holds, and any rules (like “this column cannot be empty” or “this column must be unique”).

#### Purposes

- To define the structure of a new relation before inserting data.
- To enforce data integrity through column and table constraints.
- To specify storage characteristics such as tablespace, fillfactor, and persistence.
- To create specialised table types (temporary, unlogged, partitioned) by including the appropriate clauses.

#### Syntax Rules and Structure

**General syntax (PostgreSQL, simplified):**

```sql
CREATE [ [ GLOBAL | LOCAL ] { TEMPORARY | TEMP } | UNLOGGED ] TABLE [ IF NOT EXISTS ]
    table_name (
        { column_name data_type [ column_constraint [ ... ] ]
        | table_constraint
        | LIKE source_table [ like_option ... ] } [, ... ]
    )
    [ INHERITS ( parent_table [, ... ] ) ]
    [ PARTITION BY { RANGE | LIST | HASH } ( ... ) ]
    [ WITH ( storage_parameter [= value] [, ... ] ) ]
    [ ON COMMIT { PRESERVE ROWS | DELETE ROWS | DROP } ]
    [ TABLESPACE tablespace_name ];
```


**MySQL (simplified):**

```sql
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
    (create_definition,...)
    [table_options]
    [partition_options];
```


**SQL Server (simplified):**

```sql
CREATE TABLE { database_name.schema_name.table_name | schema_name.table_name | table_name }
    ( { <column_definition> } [ ,...n ] )
    [ ON { partition_scheme_name ( partition_column_name ) | filegroup | "default" } ]
    [ WITH ( <table_option> [ ,...n ] ) ];
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `table_name` | Identifier for the new table. Can be schema-qualified. |
| `column_name data_type` | Defines a column and its data type (e.g., `integer`, `varchar(100)`, `timestamptz`). |
| `column_constraint` | `NOT NULL`, `DEFAULT`, `CHECK`, `UNIQUE`, `PRIMARY KEY`, `REFERENCES`. |
| `table_constraint` | Constraints spanning multiple columns (composite primary key, foreign key, exclusion). |
| `LIKE source_table` | Copies column definitions (and optionally indexes, constraints) from an existing table. |
| `TEMPORARY` / `TEMP` | Creates a session-scoped temporary table. |
| `UNLOGGED` (PostgreSQL) | Creates a table that bypasses the write-ahead log (non-crash-safe). |
| `PARTITION BY` | Declares the table as partitioned (PostgreSQL declarative partitioning). |
| `ON COMMIT` (temporary tables) | Controls behaviour at transaction end: `PRESERVE ROWS`, `DELETE ROWS`, or `DROP`. |
| `TABLESPACE` | Specifies the physical storage location. |

**Syntax rules:**

- Column names must be unique within the table.
- A table must have at least one column.
- `PRIMARY KEY` implies `NOT NULL` and `UNIQUE`.
- In PostgreSQL, `CREATE TABLE` cannot be executed inside a transaction block if it includes certain object definitions that require their own transaction.
- In MySQL, the `CREATE` privilege on the table is required.

**Constraints and limitations:**

- Table names must be unique within a schema (and within a database in MySQL, where schema and database are synonymous).
- Some data types and features are vendor-specific (e.g., `SERIAL` in PostgreSQL, `AUTO_INCREMENT` in MySQL, `IDENTITY` in SQL Server).
- `UNLOGGED` tables are PostgreSQL-specific and are not crash-safe.
- Partitioning syntax differs significantly between PostgreSQL, MySQL, and SQL Server.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Create a table with constraints

```sql
-- Step 1: Create a simple table with a primary key, NOT NULL,
--         DEFAULT, and a CHECK constraint.
CREATE TABLE employees (
    employee_id serial PRIMARY KEY,
    first_name  text NOT NULL,
    last_name   text NOT NULL,
    email       text UNIQUE,
    salary      numeric(10,2) CHECK (salary > 0),
    hired_at    timestamptz DEFAULT now(),
    department  text
);
```

**Expected output:**

```text
CREATE TABLE
```

**Why this output appears**: PostgreSQL creates the table and returns the command tag `CREATE TABLE`. The `serial` pseudo-type creates an implicit sequence and default; `PRIMARY KEY` implies `NOT NULL` and `UNIQUE`; the `CHECK` constraint enforces that `salary` is positive.

##### Example 2: MySQL – Create a table with an auto-increment primary key

```sql
-- Step 1: Create a table with AUTO_INCREMENT and a composite unique key.
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    sku        VARCHAR(64) NOT NULL,
    name       VARCHAR(255) NOT NULL,
    price      DECIMAL(10,2) NOT NULL DEFAULT 0.00,
    UNIQUE KEY uq_sku (sku)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
```

**Why this output appears**: MySQL creates the table with the InnoDB storage engine and the specified character set. `AUTO_INCREMENT` generates sequential integer values; the unique key on `sku` prevents duplicates.

##### Example 3: SQL Server – Create a table with a clustered primary key

```sql
-- Step 1: Create a table with an IDENTITY column and a clustered primary key.
CREATE TABLE customers (
    customer_id INT IDENTITY(1,1) PRIMARY KEY CLUSTERED,
    full_name   NVARCHAR(200) NOT NULL,
    email       NVARCHAR(320) NOT NULL UNIQUE,
    created_at  DATETIME2 DEFAULT SYSUTCDATETIME()
);
GO
```

**Expected output:**

```text
Commands completed successfully.
```

**Why this output appears**: SQL Server creates the table with a clustered primary key on `customer_id` and a unique constraint on `email`. `IDENTITY(1,1)` auto-generates values starting at 1 with an increment of 1.

#### Real-World Cases

- **Application schema initialisation**: Migration scripts use `CREATE TABLE` to define the initial schema for a new application.
- **Data warehousing staging**: A staging table is created with the same structure as the target table to load and validate data before merging.
- **Audit logging**: An append-only audit table is created with a `DEFAULT now()` timestamp and no `UPDATE`/`DELETE` permissions for application roles.

---

### 2. Viewing Table Definitions

#### Definitions

**Core Definition**
Viewing a table definition retrieves the structure of an existing table, including column names, data types, constraints, indexes, and storage attributes.

**Technical Definition**
Table definitions can be inspected through client meta-commands (`\d table_name` in psql, `DESCRIBE table_name` in MySQL, `sp_help table_name` in SQL Server) or by querying system catalogs (`information_schema.columns`, `pg_catalog.pg_attribute`, `sys.columns`). The output includes column ordinals, data types, nullability, defaults, and constraint information.

**Beginner-Friendly Explanation**
This is how you look at the blueprint of an existing table—what columns it has, what type of data each column holds, and what rules are attached—without opening the data itself.

#### Purposes

- To understand the structure of an existing table before writing queries or migrations.
- To verify that a table was created correctly after DDL execution.
- To document the schema for data dictionaries and onboarding materials.
- To compare table structures across environments for schema drift detection.

#### Syntax Rules and Structure

**PostgreSQL (psql meta-command):**

```text
\d table_name
\d+ table_name
```


**PostgreSQL (SQL via information_schema):**

```sql
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_schema = 'public'
  AND table_name = 'employees'
ORDER BY ordinal_position;
```


**MySQL:**

```sql
DESCRIBE table_name;
SHOW CREATE TABLE table_name;
```


**SQL Server:**

```sql
EXEC sp_help 'table_name';
-- or
SELECT column_name, data_type, is_nullable
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'customers';
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `\d` | psql meta-command that describes a table, view, index, or sequence. |
| `\d+` | Verbose variant that includes comments, storage, and statistics. |
| `information_schema.columns` | Standard view containing column metadata. |
| `DESCRIBE` | MySQL and Oracle command for table structure. |
| `sp_help` | SQL Server stored procedure for object metadata. |

**Syntax rules:**

- `\d` in psql accepts a table name or pattern; without a name it lists all relations.
- `DESCRIBE` in MySQL requires the table to exist and the user to have some privilege on it.
- `information_schema.columns` returns only columns the current user has access to.

**Constraints and limitations:**

- Meta-commands are client-specific and not part of SQL.
- `information_schema` may not expose vendor-specific attributes such as storage parameters or partitioning details; use `pg_catalog` (PostgreSQL) or `sys` views (SQL Server) for those.
- Column ordering in `information_schema.columns` is by `ordinal_position`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Describe a table with psql

```text
-- Step 1: Connect to the database with psql.
$ psql -U postgres -d app

-- Step 2: Describe the "employees" table.
app=# \d employees
```

**Expected output:**

```text
                                        Table "public.employees"
   Column    |           Type           | Collation | Nullable |           Default
-------------+--------------------------+-----------+----------+------------------------------
 employee_id | integer                  |           | not null | nextval('employees_employee_id_seq'::regclass)
 first_name  | text                     |           | not null |
 last_name   | text                     |           | not null |
 email       | text                     |           |          |
 salary      | numeric(10,2)            |           |          |
 hired_at    | timestamp with time zone |           |          | now()
 department  | text                     |           |          |
Indexes:
    "employees_pkey" PRIMARY KEY, btree (employee_id)
    "employees_email_key" UNIQUE CONSTRAINT, btree (email)
Check constraints:
    "employees_salary_check" CHECK (salary > 0::numeric)
```

**Why this output appears**: `\d` queries `pg_catalog` and formats the result as a human-readable table. It shows column definitions, indexes, and check constraints associated with the table.

##### Example 2: MySQL – View the full table definition

```sql
-- Step 1: Use the database.
USE school_db;

-- Step 2: Show the CREATE TABLE statement.
SHOW CREATE TABLE students;
```

**Expected output:**

```text
+----------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table    | Create Table                                                                                                                                                                                                                                                  |
+----------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| students | CREATE TABLE `students` (
  `student_id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(100) NOT NULL,
  `enrolled_at` datetime DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`student_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci |
+----------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

**Why this output appears**: `SHOW CREATE TABLE` reconstructs the original `CREATE TABLE` statement from the data dictionary, including storage engine, character set, and collation.

#### Real-World Cases

- **Onboarding**: A new developer uses `\d` or `DESCRIBE` to understand an unfamiliar table before writing a query.
- **Documentation generation**: A script queries `information_schema.columns` to generate a data dictionary.
- **Schema comparison**: A DBA compares `SHOW CREATE TABLE` output between staging and production to detect drift.

---

### 3. Listing Tables

#### Definitions

**Core Definition**
Listing tables retrieves the names of all tables (and optionally views) in a database or schema.

**Technical Definition**
Tables can be listed via client meta-commands (`\dt` in psql, `SHOW TABLES` in MySQL) or by querying system catalogs (`information_schema.tables`, `pg_catalog.pg_tables`, `sys.tables`). Filtering by schema, table type, and ownership is possible through catalog queries.

**Beginner-Friendly Explanation**
This is how you get a list of all the spreadsheets (tables) that exist in a database or schema.

#### Purposes

- To discover what tables exist in a database or schema.
- To verify that a table was created or dropped successfully.
- To audit table inventory for compliance or cleanup.
- To iterate over tables in automation scripts (backup, maintenance, migration).

#### Syntax Rules and Structure

**PostgreSQL (psql):**

```text
\dt
\dt schema_name.*
```


**PostgreSQL (SQL):**

```sql
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_type = 'BASE TABLE'
  AND table_schema NOT IN ('pg_catalog', 'information_schema')
ORDER BY table_schema, table_name;
```


**MySQL:**

```sql
SHOW TABLES;
SHOW TABLES FROM database_name;
```


**SQL Server:**

```sql
SELECT schema_name(schema_id) AS schema_name, name AS table_name
FROM sys.tables
ORDER BY schema_name, table_name;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `\dt` | psql meta-command to list tables. |
| `information_schema.tables` | Standard view listing tables and views. |
| `SHOW TABLES` | MySQL command listing tables in the current or specified database. |
| `sys.tables` | SQL Server system catalog view. |

**Syntax rules:**

- `\dt` lists tables in the `search_path` schemas.
- `SHOW TABLES` in MySQL returns only tables and views for which the user has some privilege.
- `sys.tables` includes user tables; system tables are in `sys.system_objects`.

**Constraints and limitations:**

- `information_schema.tables` includes views; filter by `table_type = 'BASE TABLE'` for tables only.
- Partitioned tables may appear differently depending on the RDBMS (PostgreSQL lists the parent and partitions).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – List tables with schema and owner

```sql
-- Step 1: Query pg_catalog.pg_tables for table names, schema, and owner.
SELECT schemaname, tablename, tableowner
FROM pg_catalog.pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY schemaname, tablename;
```

**Expected output:**

```text
 schemaname | tablename | tableowner
------------+-----------+------------
 public     | employees | postgres
 public     | products  | postgres
(2 rows)
```

**Why this output appears**: `pg_tables` is a view over `pg_class` and `pg_namespace` that exposes table names, schemas, and owners. Filtering out system schemas leaves user tables.

##### Example 2: MySQL – List tables in a database

```sql
-- Step 1: Select the database.
USE school_db;

-- Step 2: List tables.
SHOW TABLES;
```

**Expected output:**

```text
+---------------------+
| Tables_in_school_db |
+---------------------+
| courses             |
| enrollments         |
| students            |
+---------------------+
3 rows in set (0.00 sec)
```

**Why this output appears**: MySQL returns one row per table in the selected database, using the column name `Tables_in_<database>`.

#### Real-World Cases

- **Inventory audit**: A DBA lists all tables to identify unused or orphaned tables.
- **Backup automation**: A script lists tables and iterates over them to perform per-table backups.
- **Development onboarding**: A new team member runs `\dt` or `SHOW TABLES` to see what data structures exist.

---

### 4. Renaming Tables

#### Definitions

**Core Definition**
Renaming a table changes its identifier from an old name to a new name without altering its structure or data.

**Technical Definition**
In PostgreSQL, `ALTER TABLE old_name RENAME TO new_name` updates the `relname` in `pg_class`. In MySQL, `RENAME TABLE old_name TO new_name` performs an atomic rename and can rename multiple tables in one statement. In SQL Server, `sp_rename 'old_name', 'new_name'` changes the table name, but renaming is not transactional and requires careful handling of dependencies.

**Beginner-Friendly Explanation**
This is like changing the label on a spreadsheet tab without changing anything inside the spreadsheet.

#### Purposes

- To align table names with new naming conventions.
- To correct a misspelled or outdated table name.
- To reflect a change in the table’s purpose or domain.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
ALTER TABLE [ IF EXISTS ] old_name RENAME TO new_name;
```


**MySQL:**

```sql
RENAME TABLE old_name TO new_name [, old_name2 TO new_name2 ...];
```


**SQL Server:**

```sql
EXEC sp_rename 'schema.old_name', 'new_name';
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ALTER TABLE ... RENAME TO` | PostgreSQL clause for renaming. |
| `RENAME TABLE ... TO` | MySQL statement for atomic renaming. |
| `sp_rename` | SQL Server system stored procedure for renaming objects. |

**Syntax rules:**

- The new name must be unique within the schema.
- In PostgreSQL, the user must own the table.
- In MySQL, `RENAME TABLE` requires `ALTER` and `DROP` privileges on the old table and `CREATE` and `INSERT` on the new table.
- In SQL Server, `sp_rename` requires `ALTER` permission on the object.

**Constraints and limitations:**

- Renaming does not update references in views, functions, or application code automatically; dependent objects may break.
- SQL Server’s `sp_rename` is not transaction-safe in all cases and can leave objects in an inconsistent state if interrupted.
- Renaming a table does not rename associated indexes, sequences, or constraints in most systems.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Rename a table

```sql
-- Step 1: Create a table.
CREATE TABLE old_products (id int PRIMARY KEY, name text);

-- Step 2: Rename it.
ALTER TABLE old_products RENAME TO products;

-- Step 3: Verify.
\d products
```

**Expected output:**

```text
CREATE TABLE
ALTER TABLE
                                        Table "public.products"
 Column |  Type   | Collation | Nullable | Default
--------+---------+-----------+----------+---------
 id     | integer |           | not null |
 name   | text    |           |          |
Indexes:
    "old_products_pkey" PRIMARY KEY, btree (id)
```

**Why this output appears**: The table is renamed, but the automatically generated primary key index retains its original name (`old_products_pkey`). This illustrates that renaming a table does not rename dependent objects.

##### Example 2: MySQL – Rename multiple tables atomically

```sql
-- Step 1: Rename two tables in a single statement.
RENAME TABLE students TO learners,
             courses  TO subjects;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.01 sec)
```

**Why this output appears**: MySQL performs the renames atomically; either both succeed or neither does. This is useful for migration scripts that must keep related tables consistent.

#### Real-World Cases

- **Naming convention migration**: A legacy table named `tbl_cust` is renamed to `customers` to match the organisation’s naming standard.
- **Domain refactoring**: A table named `items` is renamed to `products` after the application’s domain model is updated.
- **Staging promotion**: A table named `orders_staging` is renamed to `orders` after data validation.

---

### 5. Truncating Tables (`TRUNCATE TABLE`)

#### Definitions

**Core Definition**
`TRUNCATE TABLE` quickly removes all rows from a table while preserving the table’s structure, indexes, and constraints.

**Technical Definition**
`TRUNCATE` deallocates the data pages of a table rather than scanning and deleting individual rows, making it significantly faster than `DELETE` for large tables. In PostgreSQL, `TRUNCATE` acquires an `ACCESS EXCLUSIVE` lock, is transaction-safe with respect to data, and can optionally restart identity sequences (`RESTART IDENTITY`) or cascade to tables with foreign-key references (`CASCADE`). In MySQL, `TRUNCATE` implicitly commits and cannot be rolled back.

**Beginner-Friendly Explanation**
This empties the spreadsheet—removes every row—but keeps the column headers and any rules intact, ready for new data.

#### Purposes

- To quickly empty a table before loading fresh data.
- To reset a staging or temporary table between test runs.
- To reclaim disk space occupied by a table’s rows without dropping the table.
- To restart identity sequences after clearing a table (PostgreSQL `RESTART IDENTITY`).

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
TRUNCATE [ TABLE ] [ ONLY ] name [ * ] [, ... ]
    [ RESTART IDENTITY | CONTINUE IDENTITY ]
    [ CASCADE | RESTRICT ];
```


**MySQL:**

```sql
TRUNCATE [TABLE] table_name;
```


**SQL Server:**

```sql
TRUNCATE TABLE table_name;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ONLY` | Truncate only the named table, not inheritance descendants. |
| `RESTART IDENTITY` | Restart sequences owned by the table’s columns. |
| `CONTINUE IDENTITY` | Do not change sequence values (default). |
| `CASCADE` | Truncate tables with foreign-key references to the named tables. |
| `RESTRICT` | Refuse to truncate if foreign-key references exist (default). |

**Syntax rules:**

- The user must have the `TRUNCATE` privilege on the table.
- `TRUNCATE` acquires an `ACCESS EXCLUSIVE` lock in PostgreSQL, blocking all concurrent access.
- In MySQL, `TRUNCATE` causes an implicit commit and cannot be rolled back.
- `TRUNCATE` does not fire `ON DELETE` triggers in PostgreSQL, but it does fire `ON TRUNCATE` triggers.

**Constraints and limitations:**

- `TRUNCATE` cannot be used on a table that has foreign-key references from other tables unless `CASCADE` is used or all referencing tables are truncated in the same command.
- `TRUNCATE` is not MVCC-safe in PostgreSQL: concurrent transactions using an older snapshot will still see the old rows.
- In MySQL, `TRUNCATE` resets `AUTO_INCREMENT` counters.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Truncate and restart identity

```sql
-- Step 1: Create a table with a serial column.
CREATE TABLE logs (id serial PRIMARY KEY, message text);

-- Step 2: Insert a few rows.
INSERT INTO logs (message) VALUES ('first'), ('second'), ('third');

-- Step 3: Truncate and restart the identity sequence.
TRUNCATE logs RESTART IDENTITY;

-- Step 4: Verify the sequence restarted.
INSERT INTO logs (message) VALUES ('new first') RETURNING id;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 3
TRUNCATE TABLE
 id
----
  1
(1 row)
```

**Why this output appears**: `RESTART IDENTITY` resets the sequence owned by `id` to its starting value. The next `INSERT` receives `id = 1` again.

##### Example 2: PostgreSQL – Truncate with foreign-key cascade

```sql
-- Step 1: Create parent and child tables.
CREATE TABLE parent (id int PRIMARY KEY);
CREATE TABLE child (id int PRIMARY KEY, parent_id int REFERENCES parent(id));

-- Step 2: Insert data.
INSERT INTO parent VALUES (1);
INSERT INTO child VALUES (10, 1);

-- Step 3: Truncate the parent and cascade to the child.
TRUNCATE parent CASCADE;
```

**Expected output:**

```text
CREATE TABLE
CREATE TABLE
INSERT 0 1
INSERT 0 1
NOTICE:  truncate cascades to table "child"
TRUNCATE TABLE
```

**Why this output appears**: `CASCADE` automatically truncates the `child` table because it has a foreign-key reference to `parent`. Without `CASCADE`, the truncate would fail due to the foreign-key dependency.

#### Real-World Cases

- **Test isolation**: A test suite truncates all tables in a `beforeEach` hook to ensure a clean state.
- **Staging refresh**: A nightly job truncates staging tables and reloads them from production extracts.
- **Sequence reset**: After a data cleanup, `TRUNCATE ... RESTART IDENTITY` resets auto-increment counters so new rows start from 1.

---

### 6. Dropping Tables (`DROP TABLE`)

#### Definitions

**Core Definition**
`DROP TABLE` permanently removes a table and all its data, indexes, constraints, triggers, and associated objects from the database.

**Technical Definition**
`DROP TABLE` deletes the table’s entry from the system catalogs and removes its physical files (or marks them for reuse). By default it uses `RESTRICT`, refusing to drop a table that is referenced by a view or foreign-key constraint; `CASCADE` drops dependent objects as well. The statement is irreversible without a backup.

**Beginner-Friendly Explanation**
This deletes the entire spreadsheet—including its structure and all its data—permanently. There is no recycle bin for tables, so backups are essential.

#### Purposes

- To remove obsolete or temporary tables after their purpose is served.
- To clean up test or staging tables in a development environment.
- To decommission a table as part of a schema refactoring.
- To free disk space occupied by large, unused tables.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
DROP TABLE [ IF EXISTS ] name [, ...] [ CASCADE | RESTRICT ];
```


**MySQL:**

```sql
DROP [TEMPORARY] TABLE [IF EXISTS] tbl_name [, tbl_name] ...;
```


**SQL Server:**

```sql
DROP TABLE [IF EXISTS] table_name;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IF EXISTS` | Suppresses the error if the table does not exist. |
| `name` | Table to drop; multiple tables can be dropped in one statement (PostgreSQL, MySQL). |
| `CASCADE` | Automatically drop objects that depend on the table (views, foreign-key constraints). |
| `RESTRICT` | Refuse to drop if dependent objects exist (default). |

**Syntax rules:**

- Only the table owner, schema owner, or a superuser can drop a table in PostgreSQL.
- In MySQL, the `DROP` privilege on the table is required.
- `DROP TABLE` always removes indexes, rules, triggers, and constraints that exist for the target table.

**Constraints and limitations:**

- `DROP TABLE ... CASCADE` can have far-reaching effects; always verify dependencies before executing.
- In PostgreSQL, `CASCADE` will remove a dependent view entirely, but for a foreign-key constraint it removes only the constraint, not the referencing table.
- `DROP TABLE` cannot be rolled back outside a transaction in MySQL (DDL causes an implicit commit).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Drop a table safely

```sql
-- Step 1: Drop the table if it exists.
DROP TABLE IF EXISTS staging_data;
```

**Expected output:**

```text
NOTICE:  table "staging_data" does not exist, skipping
DROP TABLE
```

**Why this output appears**: `IF EXISTS` prevents an error when the table is absent, issuing a notice instead. The statement succeeds either way.

##### Example 2: PostgreSQL – Drop a table with dependent objects

```sql
-- Step 1: Create a table and a view that depends on it.
CREATE TABLE base_table (id int PRIMARY KEY, value text);
CREATE VIEW base_view AS SELECT * FROM base_table;

-- Step 2: Drop the table with CASCADE.
DROP TABLE base_table CASCADE;
```

**Expected output:**

```text
CREATE TABLE
CREATE VIEW
NOTICE:  drop cascades to view base_view
DROP TABLE
```

**Why this output appears**: `CASCADE` drops the dependent view `base_view` before dropping `base_table`. Without `CASCADE`, the statement would fail because the view depends on the table.

#### Real-World Cases

- **Environment teardown**: A CI/CD pipeline drops temporary tables after each test run.
- **Schema refactoring**: A table that has been replaced by a new structure is dropped after data migration.
- **Disk-space recovery**: A DBA drops obsolete large tables to free storage on a constrained server.

---

### 7. Temporary Tables

#### Definitions

**Core Definition**
A temporary table is a table whose data and definition are visible only to the current session (or transaction) and are automatically dropped when the session ends or the transaction completes.

**Technical Definition**
Temporary tables are created with `CREATE TEMPORARY TABLE` (or `CREATE TEMP TABLE`). In PostgreSQL, they are stored in a session-specific schema (`pg_temp_N`) and can be configured with `ON COMMIT PRESERVE ROWS` (default), `ON COMMIT DELETE ROWS`, or `ON COMMIT DROP`. In MySQL, temporary tables are session-scoped and can use the `MEMORY` storage engine for in-memory performance. In SQL Server, temporary tables are either local (`#temp`) or global (`##temp`).

**Beginner-Friendly Explanation**
A temporary table is like a scratchpad that exists only for your current session. When you disconnect, the scratchpad disappears automatically, leaving no trace.

#### Purposes

- To store intermediate results during complex queries or ETL processes.
- To isolate data for a single session without affecting other users.
- To stage data before merging it into permanent tables.
- To avoid repeated computation by materialising a subquery result once and reusing it.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
CREATE [ [ GLOBAL | LOCAL ] { TEMPORARY | TEMP } ] TABLE [ IF NOT EXISTS ]
    table_name ( ... )
    [ ON COMMIT { PRESERVE ROWS | DELETE ROWS | DROP } ];
```


**MySQL:**

```sql
CREATE TEMPORARY TABLE [IF NOT EXISTS] tbl_name ( ... );
```


**SQL Server:**

```sql
CREATE TABLE #local_temp ( ... );    -- session-scoped
CREATE TABLE ##global_temp ( ... );  -- visible to all sessions
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `TEMPORARY` / `TEMP` | Keyword that creates a temporary table. |
| `ON COMMIT PRESERVE ROWS` | Rows persist after commit (PostgreSQL default). |
| `ON COMMIT DELETE ROWS` | Rows are deleted at each commit (automatic truncate). |
| `ON COMMIT DROP` | Table is dropped at the end of the transaction. |
| `#` / `##` (SQL Server) | Prefix for local and global temporary tables. |

**Syntax rules:**

- Temporary tables are session-scoped; each session sees its own instance.
- In PostgreSQL, temporary tables are stored in the `pg_temp` schema, which is implicitly first in the `search_path`.
- In MySQL, `TEMPORARY` tables are not visible to other sessions and are dropped when the session ends.

**Constraints and limitations:**

- Temporary tables in PostgreSQL are not WAL-logged if `temp_tablespaces` is configured on a temporary tablespace.
- In MySQL, temporary tables cannot be partitioned and have restrictions on storage engine choice.
- SQL Server global temporary tables (`##`) are visible to all sessions and are dropped when the last session referencing them disconnects.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Temporary table with ON COMMIT DROP

```sql
-- Step 1: Begin a transaction.
BEGIN;

-- Step 2: Create a temporary table that is dropped at commit.
CREATE TEMP TABLE temp_results (id int, score numeric)
    ON COMMIT DROP;

-- Step 3: Insert data.
INSERT INTO temp_results VALUES (1, 95.5), (2, 87.0);

-- Step 4: Query the temporary table.
SELECT * FROM temp_results;

-- Step 5: Commit; the table is dropped automatically.
COMMIT;

-- Step 6: Attempting to query the table now fails.
SELECT * FROM temp_results;
```

**Expected output:**

```text
BEGIN
CREATE TABLE
INSERT 0 2
 id | score
----+-------
  1 |  95.5
  2 |  87.0
(2 rows)
COMMIT
ERROR:  relation "temp_results" does not exist
```

**Why this output appears**: `ON COMMIT DROP` instructs PostgreSQL to drop the temporary table when the transaction commits. After `COMMIT`, the table no longer exists, so the subsequent `SELECT` raises an error.

##### Example 2: SQL Server – Local temporary table

```sql
-- Step 1: Create a local temporary table.
CREATE TABLE #TempOrders (
    order_id int,
    amount   decimal(10,2)
);

-- Step 2: Insert data.
INSERT INTO #TempOrders VALUES (1001, 250.00), (1002, 175.50);

-- Step 3: Query it.
SELECT * FROM #TempOrders;
```

**Expected output:**

```text
order_id  amount
--------- -------
1001      250.00
1002      175.50
```

**Why this output appears**: The `#` prefix creates a session-scoped temporary table. It is automatically dropped when the session ends or the connection is closed.

#### Real-World Cases

- **ETL staging**: A temporary table holds extracted data before transformation and loading into the target warehouse.
- **Report generation**: A complex report materialises intermediate results in a temporary table to avoid repeating expensive subqueries.
- **Session scratchpad**: An application stores user-specific calculations in a temporary table during a long-running session.

---

### 8. Unlogged Tables

#### Definitions

**Core Definition**
An unlogged table is a PostgreSQL table whose data is not written to the write-ahead log (WAL), trading durability for write performance.

**Technical Definition**
`CREATE UNLOGGED TABLE` creates a table that bypasses WAL logging for `INSERT`, `UPDATE`, and `DELETE` operations. This makes writes significantly faster than regular logged tables, but the table is not crash-safe: if the server crashes or shuts down uncleanly, the table is automatically truncated. Unlogged tables are not replicated to standby servers and are not included in WAL-based backups.

**Beginner-Friendly Explanation**
An unlogged table is a fast but risky table. It writes data much faster because it skips the database’s safety journal, but if the server crashes, all its data disappears. It is useful for temporary, reproducible data that you can afford to lose.

#### Purposes

- To accelerate write-heavy workloads where data can be regenerated if lost.
- To store transient or cache-like data that does not require crash durability.
- To reduce WAL volume in high-throughput staging or benchmarking scenarios.
- To improve performance for bulk loads where the data will later be copied to a logged table.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
CREATE UNLOGGED TABLE [ IF NOT EXISTS ] table_name ( ... );
```

The `UNLOGGED` keyword appears in the same position as `TEMPORARY` in the `CREATE TABLE` syntax.


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UNLOGGED` | Keyword that creates a non-WAL-logged table. |

**Syntax rules:**

- The `UNLOGGED` keyword is PostgreSQL-specific and is not part of the SQL standard.
- Unlogged tables can have indexes, constraints, and triggers, but those objects are also unlogged.
- `TRUNCATE` on an unlogged table is WAL-logged, but the table’s contents are not.

**Constraints and limitations:**

- Unlogged tables are **not crash-safe**: their contents are automatically truncated after a crash or unclean shutdown.
- Their contents are **not replicated** to standby servers.
- They are **not included in WAL-based backups**; a `pg_dump` will include the table definition but not the data.
- Sequence values associated with unlogged tables may still be logged, depending on the sequence’s own logging status.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Create and use an unlogged table

```sql
-- Step 1: Create an unlogged table for benchmark data.
CREATE UNLOGGED TABLE benchmark_results (
    run_id     int,
    metric     text,
    value      numeric,
    recorded_at timestamptz DEFAULT now()
);

-- Step 2: Insert a large number of rows quickly.
INSERT INTO benchmark_results (run_id, metric, value)
SELECT g, 'throughput', random() * 1000
FROM generate_series(1, 100000) AS g;

-- Step 3: Query the table.
SELECT count(*) FROM benchmark_results;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 100000
 count
--------
 100000
(1 row)
```

**Why this output appears**: The `INSERT` is not WAL-logged, so it completes faster than it would on a logged table. The data is fully readable during normal operation; the lack of durability only matters if the server crashes.

##### Example 2: PostgreSQL – Observe truncation after crash (conceptual)

```sql
-- Step 1: Create an unlogged table and insert data.
CREATE UNLOGGED TABLE crash_test (id int, note text);
INSERT INTO crash_test VALUES (1, 'survives only until crash');

-- Step 2: Simulate an unclean shutdown (e.g., kill -9 the postmaster).
-- After restart, query the table.
SELECT count(*) FROM crash_test;
```

**Expected output** (after crash recovery):

```text
 count
-------
     0
(1 row)
```

**Why this output appears**: PostgreSQL automatically truncates unlogged tables during crash recovery because their contents were never WAL-logged. The table definition survives, but all rows are lost.

#### Real-World Cases

- **Benchmarking**: Performance tests use unlogged tables to measure raw write throughput without WAL overhead.
- **Session caches**: Applications store session or cache data in unlogged tables where loss after a crash is acceptable.
- **Bulk staging**: A high-volume ETL pipeline loads data into an unlogged staging table for transformation, then writes the final result to a logged table.

---

### 9. Partitioned Tables

#### Definitions

**Core Definition**
A partitioned table is a logical table that is physically divided into smaller, independently managed pieces called partitions, based on a partitioning key.

**Technical Definition**
PostgreSQL declarative partitioning (available since version 10) allows a table to be defined with `PARTITION BY { RANGE | LIST | HASH } (column)`. The parent table holds no data itself; rows are routed to child partitions based on the partitioning key. Each partition is a regular table with its own storage, indexes, and constraints. Partition pruning allows the query planner to skip partitions that cannot contain matching rows, improving performance on large datasets.

**Beginner-Friendly Explanation**
A partitioned table is like a filing cabinet that is split into several drawers. You tell the database which drawer to use based on a rule—for example, “put all 2024 orders in the 2024 drawer and all 2025 orders in the 2025 drawer.” Queries that only need 2024 data can skip the 2025 drawer entirely.

#### Purposes

- To improve query performance on very large tables through partition pruning.
- To simplify data lifecycle management (e.g., dropping an old partition instead of deleting rows).
- To enable parallel maintenance operations on individual partitions.
- To distribute data across different tablespaces or storage tiers.

#### Syntax Rules and Structure

**PostgreSQL (declarative partitioning):**

```sql
-- Parent partitioned table
CREATE TABLE measurement (
    logdate   date NOT NULL,
    peaktemp  int,
    unitsales int
) PARTITION BY RANGE (logdate);

-- Child partitions
CREATE TABLE measurement_y2024 PARTITION OF measurement
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

CREATE TABLE measurement_y2025 PARTITION OF measurement
    FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `PARTITION BY RANGE (column)` | Partitions based on a continuous range of values. |
| `PARTITION BY LIST (column)` | Partitions based on a discrete list of values. |
| `PARTITION BY HASH (column)` | Partitions based on a hash of the column value. |
| `PARTITION OF parent_table` | Creates a child partition of an existing partitioned table. |
| `FOR VALUES ...` | Defines the range or list of values for the partition. |
| `DEFAULT` | Creates a default partition for rows that do not match any other partition. |

**Syntax rules:**

- The partitioning column must be part of any unique constraint or primary key on the partitioned table.
- Partitions can themselves be partitioned (sub-partitioning).
- A default partition is optional; if present, it catches rows that do not match any other partition.
- The `ONLY` keyword in queries excludes child partitions; omitting `ONLY` includes them.

**Constraints and limitations:**

- Declarative partitioning is PostgreSQL-specific (version 10+); MySQL and SQL Server use different partitioning syntax and features.
- Unique constraints on a partitioned table must include the partition key.
- `CREATE TABLE ... PARTITION OF` takes an `ACCESS EXCLUSIVE` lock on the parent table.
- Partition pruning works best with `RANGE` and `LIST` partitioning; `HASH` partitioning provides even distribution but less pruning opportunity.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Range partitioning by date

```sql
-- Step 1: Create a range-partitioned table for sales data.
CREATE TABLE sales (
    sale_id   serial,
    sale_date date NOT NULL,
    amount    numeric(10,2) NOT NULL
) PARTITION BY RANGE (sale_date);

-- Step 2: Create partitions for each year.
CREATE TABLE sales_2024 PARTITION OF sales
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

CREATE TABLE sales_2025 PARTITION OF sales
    FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');

-- Step 3: Insert data; rows are routed automatically.
INSERT INTO sales (sale_date, amount) VALUES
    ('2024-06-15', 100.00),
    ('2025-02-20', 250.00);

-- Step 4: Query the parent table; partition pruning is applied.
EXPLAIN (COSTS OFF)
SELECT * FROM sales WHERE sale_date >= '2025-01-01';
```

**Expected output:**

```text
CREATE TABLE
CREATE TABLE
CREATE TABLE
INSERT 0 2
              QUERY PLAN
--------------------------------------
 Seq Scan on sales_2025
   Filter: (sale_date >= '2025-01-01'::date)
(2 rows)
```

**Why this output appears**: The planner recognises that the `WHERE` clause can only match rows in `sales_2025` and prunes `sales_2024` from the plan. This is the core performance benefit of partitioning.

##### Example 2: PostgreSQL – List partitioning by region

```sql
-- Step 1: Create a list-partitioned table.
CREATE TABLE customers (
    customer_id int,
    region      text NOT NULL
) PARTITION BY LIST (region);

-- Step 2: Create partitions for specific regions.
CREATE TABLE customers_us PARTITION OF customers FOR VALUES IN ('US');
CREATE TABLE customers_eu PARTITION OF customers FOR VALUES IN ('EU');
CREATE TABLE customers_other PARTITION OF customers DEFAULT;

-- Step 3: Insert data.
INSERT INTO customers VALUES (1, 'US'), (2, 'EU'), (3, 'APAC');

-- Step 4: Verify routing.
SELECT tableoid::regclass AS partition, * FROM customers;
```

**Expected output:**

```text
  partition   | customer_id | region
--------------+-------------+--------
 customers_us |           1 | US
 customers_eu |           2 | EU
 customers_other |        3 | APAC
```

**Why this output appears**: `tableoid::regclass` reveals which partition each row resides in. `APAC` does not match `US` or `EU`, so it falls into the default partition.

#### Real-World Cases

- **Time-series data**: IoT sensor readings or application logs are range-partitioned by month or day, enabling efficient queries and easy archival by dropping old partitions.
- **Multi-region SaaS**: Customer data is list-partitioned by region, allowing region-specific maintenance and compliance.
- **Large fact tables**: A data warehouse fact table is hash-partitioned to distribute I/O evenly across storage devices.

---

### 10. Table Cloning

#### Definitions

**Core Definition**
Table cloning creates a new table that duplicates the structure (and optionally the data) of an existing table.

**Technical Definition**
Cloning can be performed in two ways: (1) **Structure-only cloning** using `CREATE TABLE new_table (LIKE source_table INCLUDING ALL)`, which copies column definitions, constraints, indexes, and defaults without copying rows; (2) **Data-inclusive cloning** using `CREATE TABLE new_table AS SELECT * FROM source_table`, which creates a new table with columns and data derived from the query but does not copy indexes, constraints, or defaults. MySQL supports `CREATE TABLE new_tbl LIKE old_tbl` for structure-only cloning and `CREATE TABLE new_tbl AS SELECT ...` for data-inclusive cloning. SQL Server uses `SELECT ... INTO new_table FROM source_table` for data-inclusive cloning.

**Beginner-Friendly Explanation**
Cloning a table means making a copy. You can copy just the structure (the column headers and rules) or the structure plus all the data. The two approaches have different trade-offs: one preserves indexes and constraints but no data; the other preserves data but not indexes or constraints.

#### Purposes

- To create a backup or snapshot of a table before making structural changes.
- To create a template table for a new tenant or environment.
- To materialise a subset of data for testing or analysis without affecting the original.
- To experiment with schema changes on a copy before applying them to the production table.

#### Syntax Rules and Structure

**PostgreSQL (structure-only):**

```sql
CREATE TABLE new_table (LIKE source_table INCLUDING ALL);
```


**PostgreSQL (data-inclusive):**

```sql
CREATE TABLE new_table AS SELECT * FROM source_table [WHERE ...];
```


**MySQL (structure-only):**

```sql
CREATE TABLE new_tbl LIKE old_tbl;
```


**MySQL (data-inclusive):**

```sql
CREATE TABLE new_tbl AS SELECT * FROM old_tbl [WHERE ...];
```


**SQL Server (data-inclusive):**

```sql
SELECT * INTO new_table FROM source_table [WHERE ...];
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `LIKE source_table` | Copies column names, data types, and `NOT NULL` constraints. |
| `INCLUDING ALL` | Copies comments, constraints, defaults, identity, indexes, statistics, and storage. |
| `CREATE TABLE AS SELECT` | Creates a table from a query result; does not copy indexes or constraints. |
| `SELECT ... INTO` | SQL Server equivalent of `CREATE TABLE AS SELECT`. |

**Syntax rules:**

- `LIKE` cloning does not copy data; `CREATE TABLE AS SELECT` copies data but not indexes or constraints.
- `INCLUDING ALL` is an abbreviation for `INCLUDING COMMENTS INCLUDING CONSTRAINTS INCLUDING DEFAULTS INCLUDING IDENTITY INCLUDING INDEXES INCLUDING STATISTICS INCLUDING STORAGE`.
- The new table is completely decoupled from the original after creation; changes to one do not affect the other.

**Constraints and limitations:**

- `CREATE TABLE AS SELECT` does not copy foreign keys, check constraints, or indexes; they must be added manually if needed.
- `LIKE` cloning in PostgreSQL does not copy foreign keys unless `INCLUDING ALL` is used, and even then only certain constraint types are copied.
- Cloning a very large table can be I/O-intensive and may require significant disk space.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Clone structure with indexes and constraints

```sql
-- Step 1: Create the source table.
CREATE TABLE original (
    id serial PRIMARY KEY,
    name text NOT NULL UNIQUE,
    created_at timestamptz DEFAULT now()
);

-- Step 2: Clone the structure, including indexes and constraints.
CREATE TABLE clone (LIKE original INCLUDING ALL);

-- Step 3: Verify the clone has the same structure.
\d clone
```

**Expected output:**

```text
CREATE TABLE
CREATE TABLE
                                      Table "public.clone"
   Column   |           Type           | Collation | Nullable |           Default
------------+--------------------------+-----------+----------+------------------------------
 id         | integer                  |           | not null | nextval('original_id_seq'::regclass)
 name       | text                     |           | not null |
 created_at | timestamp with time zone |           |          | now()
Indexes:
    "clone_pkey" PRIMARY KEY, btree (id)
    "clone_name_key" UNIQUE CONSTRAINT, btree (name)
```

**Why this output appears**: `INCLUDING ALL` copies the primary key, unique constraint, default, and `NOT NULL` constraints. The clone has its own indexes with automatically generated names.

##### Example 2: MySQL – Clone data with CREATE TABLE AS SELECT

```sql
-- Step 1: Create and populate the source table.
CREATE TABLE source_data (id INT PRIMARY KEY, value VARCHAR(100));
INSERT INTO source_data VALUES (1, 'alpha'), (2, 'beta'), (3, 'gamma');

-- Step 2: Clone the data into a new table.
CREATE TABLE cloned_data AS SELECT * FROM source_data WHERE id <= 2;

-- Step 3: Verify.
SELECT * FROM cloned_data;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0

Query OK, 2 rows affected (0.02 sec)
Records: 2  Duplicates: 0  Warnings: 0

+----+-------+
| id | value |
+----+-------+
|  1 | alpha |
|  2 | beta  |
+----+-------+
```

**Why this output appears**: `CREATE TABLE AS SELECT` creates a new table with the columns and data from the query. The primary key from `source_data` is not copied to `cloned_data`; the clone has no indexes or constraints.

#### Real-World Cases

- **Pre-migration backup**: A DBA clones a table before applying a risky `ALTER TABLE` so that the original data is preserved.
- **Tenant provisioning**: A template table is cloned (structure-only) to create per-tenant tables with identical schemas.
- **Testing with realistic data**: A subset of production data is cloned into a test environment using `CREATE TABLE AS SELECT` with a `WHERE` clause.

---

### 11. Derived Tables

#### Definitions

**Core Definition**
A derived table is a subquery used in the `FROM` clause of a `SELECT` statement, producing a temporary result set that the outer query can treat as a table.

**Technical Definition**
A derived table is an expression that generates a table within the scope of a query’s `FROM` clause. It must have an alias (in MySQL, the `AS` clause is mandatory), and its columns must have unique names. Derived tables are not persisted; they exist only for the duration of the query. They are distinct from temporary tables, which are explicitly created and persisted for a session.

**Beginner-Friendly Explanation**
A derived table is like a temporary spreadsheet that exists only while a query is running. You write a subquery, give it a name, and then query that named result as if it were a real table.

#### Purposes

- To break complex queries into manageable, readable steps.
- To perform aggregations or transformations that cannot be expressed directly in a single `SELECT`.
- To reuse the same subquery result multiple times in the outer query.
- To work around limitations on referencing aliases in `WHERE` or `HAVING` clauses.

#### Syntax Rules and Structure

**General syntax:**

```sql
SELECT outer_columns
FROM (
    SELECT inner_columns
    FROM inner_table
    WHERE ...
) AS derived_alias;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `(subquery)` | The inner `SELECT` statement that generates the derived table. |
| `AS derived_alias` | Mandatory alias for the derived table; required in MySQL and PostgreSQL. |
| `(col_list)` (optional) | Explicit column names for the derived table, if the inner query’s column names are not suitable. |

**Syntax rules:**

- The derived table must have an alias.
- Column names within the derived table must be unique.
- A derived table cannot contain references to other tables of the same `SELECT` (i.e., no correlated references to the outer query) unless using `LATERAL`.
- Derived tables can be nested.

**Constraints and limitations:**

- Derived tables are not indexed; performance depends on the inner query’s plan.
- In MySQL, derived tables may be materialised or merged into the outer query depending on the optimizer.
- Derived tables cannot be updated directly; they are read-only result sets.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Aggregate a derived table

```sql
-- Step 1: Create a table and insert data.
CREATE TABLE t1 (s1 INT, s2 CHAR(5), s3 FLOAT);
INSERT INTO t1 VALUES (1, '1', 1.0), (2, '2', 2.0);

-- Step 2: Use a derived table to compute a transformed result.
SELECT sb1, sb2, sb3
FROM (
    SELECT s1 AS sb1, s2 AS sb2, s3 * 2 AS sb3
    FROM t1
) AS sb
WHERE sb1 > 1;
```

**Expected output:**

```text
+------+------+------+
| sb1  | sb2  | sb3  |
+------+------+------+
|    2 | 2    |    4 |
+------+------+------+
```

**Why this output appears**: The inner query renames columns and doubles `s3`. The outer query filters on the derived column `sb1`, which would not be possible without the derived table because `WHERE` cannot reference `SELECT` aliases directly.

##### Example 2: PostgreSQL – Aggregate an aggregate using a derived table

```sql
-- Step 1: Create a sales table.
CREATE TABLE sales (region text, amount numeric);
INSERT INTO sales VALUES ('North', 100), ('North', 200), ('South', 150);

-- Step 2: Compute the average of regional totals using a derived table.
SELECT AVG(region_total) AS avg_regional_total
FROM (
    SELECT region, SUM(amount) AS region_total
    FROM sales
    GROUP BY region
) AS regional_sums;
```

**Expected output:**

```text
 avg_regional_total
--------------------
              225.0
(1 row)
```

**Why this output appears**: The inner query computes the total per region (`North` = 300, `South` = 150). The outer query averages those totals: (300 + 150) / 2 = 225. This two-step aggregation is not possible with a single `GROUP BY`.

#### Real-World Cases

- **Reporting**: A report query uses a derived table to calculate subtotals per category, then aggregates those subtotals into a grand total.
- **Data cleaning**: A derived table filters out invalid rows and normalises values before the outer query joins the cleaned result with other tables.
- **Pagination**: A derived table applies `LIMIT` and `OFFSET` to a sorted result, and the outer query joins the paginated rows with lookup tables.

---

### 12. Table Metadata

#### Definitions

**Core Definition**
Table metadata is the structured information about a table’s definition, columns, constraints, indexes, storage, and statistics, stored in the database’s system catalogs.

**Technical Definition**
Metadata is exposed through standard views (`information_schema.tables`, `information_schema.columns`, `information_schema.table_constraints`) and vendor-specific catalogs (`pg_catalog.pg_class`, `pg_catalog.pg_attribute`, `sys.tables`, `sys.columns`). Metadata queries enable introspection, schema comparison, documentation generation, and dynamic SQL generation.

**Beginner-Friendly Explanation**
Table metadata is the “information about the information”—the database’s own record of what tables exist, what columns they have, what types those columns are, and what rules apply. It is like the table of contents and index of a book, rather than the book’s contents.

#### Purposes

- To introspect the database schema programmatically without relying on client meta-commands.
- To generate documentation, data dictionaries, or entity-relationship diagrams.
- To detect schema drift between environments.
- To build dynamic SQL that adapts to the current schema.

#### Syntax Rules and Structure

**PostgreSQL (information_schema.columns):**

```sql
SELECT table_schema, table_name, column_name, data_type,
       is_nullable, column_default
FROM information_schema.columns
WHERE table_name = 'employees'
ORDER BY ordinal_position;
```


**PostgreSQL (pg_catalog.pg_attribute):**

```sql
SELECT a.attname AS column_name,
       pg_catalog.format_type(a.atttypid, a.atttypmod) AS data_type,
       a.attnotnull AS not_null
FROM pg_catalog.pg_attribute a
JOIN pg_catalog.pg_class c ON c.oid = a.attrelid
JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
WHERE c.relname = 'employees'
  AND n.nspname = 'public'
  AND a.attnum > 0
ORDER BY a.attnum;
```


**MySQL (information_schema.columns):**

```sql
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_schema = 'school_db'
  AND table_name = 'students'
ORDER BY ordinal_position;
```


**SQL Server (INFORMATION_SCHEMA.COLUMNS):**

```sql
SELECT column_name, data_type, is_nullable
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'dbo'
  AND TABLE_NAME = 'customers'
ORDER BY ORDINAL_POSITION;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `information_schema.columns` | Standard view with column name, data type, nullability, default, and ordinal position. |
| `pg_catalog.pg_attribute` | PostgreSQL low-level catalog with detailed type and storage information. |
| `sys.columns` (SQL Server) | System catalog view with column metadata. |
| `table_catalog` / `table_schema` / `table_name` | Qualifiers for filtering metadata to a specific table. |

**Syntax rules:**

- `information_schema` views expose only objects the current user has access to.
- `pg_catalog` is always present and accessible; its contents are not filtered by privilege.
- Column names in `information_schema` are standardised; vendor-specific attributes require vendor catalogs.

**Constraints and limitations:**

- `information_schema` does not expose storage parameters, partitioning details, or index metadata in full; use vendor catalogs for those.
- Metadata queries can be slower on databases with very large numbers of objects.
- Some metadata (e.g., statistics) is updated asynchronously and may be stale.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Retrieve column metadata

```sql
-- Step 1: Create a table.
CREATE TABLE items (
    id serial PRIMARY KEY,
    name text NOT NULL,
    price numeric(10,2) DEFAULT 0.00,
    created_at timestamptz DEFAULT now()
);

-- Step 2: Query metadata for the table's columns.
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_name = 'items'
ORDER BY ordinal_position;
```

**Expected output:**

```text
 column_name |        data_type         | is_nullable |  column_default
-------------+--------------------------+-------------+------------------
 id          | integer                  | NO          | nextval('items_id_seq'::regclass)
 name        | text                     | NO          |
 price       | numeric                  | YES         | 0.00
 created_at  | timestamp with time zone | YES         | now()
(4 rows)
```

**Why this output appears**: `information_schema.columns` returns one row per column, ordered by `ordinal_position`. The `data_type` for `id` is `integer` (the underlying type of `serial`), and `column_default` shows the sequence-generated default.

##### Example 2: PostgreSQL – List all constraints on a table

```sql
-- Step 1: Query table constraints and key column usage.
SELECT tc.constraint_name, tc.constraint_type, kcu.column_name
FROM information_schema.table_constraints tc
JOIN information_schema.key_column_usage kcu
  ON tc.constraint_name = kcu.constraint_name
 AND tc.table_schema = kcu.table_schema
WHERE tc.table_name = 'items'
ORDER BY tc.constraint_type, tc.constraint_name;
```

**Expected output:**

```text
 constraint_name | constraint_type | column_name
-----------------+-----------------+-------------
 items_pkey      | PRIMARY KEY     | id
```

**Why this output appears**: The join between `table_constraints` and `key_column_usage` maps each constraint to its constituent columns. The `items` table has only a primary key constraint on `id`.

#### Real-World Cases

- **Data dictionary generation**: A script queries `information_schema.columns` for all tables and outputs a Markdown or HTML data dictionary.
- **Schema drift detection**: A CI/CD pipeline compares metadata between staging and production and flags differences.
- **Dynamic SQL**: A stored procedure queries metadata to discover columns and builds an `INSERT` or `SELECT` statement dynamically.

---

## References

- PostgreSQL Documentation – `CREATE TABLE` – https://www.postgresql.org/docs/current/sql-createtable.html
- PostgreSQL Documentation – `TRUNCATE` – https://www.postgresql.org/docs/current/sql-truncate.html
- PostgreSQL Documentation – `DROP TABLE` – https://www.postgresql.org/docs/current/sql-droptable.html
- PostgreSQL Documentation – Temporary Tables – https://www.postgresql.org/docs/current/sql-createtable.html (ON COMMIT clause)
- PostgreSQL Documentation – Unlogged Tables – https://www.postgresql.org/docs/current/sql-createtable.html (UNLOGGED keyword) and https://www.postgresql.org/docs/current/non-durability.html
- PostgreSQL Documentation – Table Partitioning – https://www.postgresql.org/docs/current/ddl-partitioning.html
- PostgreSQL Documentation – `CREATE TABLE AS` – https://www.postgresql.org/docs/current/sql-createtableas.html
- PostgreSQL Documentation – `information_schema.columns` – https://www.postgresql.org/docs/current/infoschema-columns.html
- PostgreSQL Documentation – `pg_catalog.pg_attribute` – https://www.postgresql.org/docs/current/catalog-pg-attribute.html
- PostgreSQL Documentation – `ALTER TABLE ... RENAME` – https://www.postgresql.org/docs/current/sql-altertable.html
- MySQL 8.0 Reference Manual – `CREATE TABLE` – https://dev.mysql.com/doc/refman/8.0/en/create-table.html
- MySQL 8.0 Reference Manual – `RENAME TABLE` – https://dev.mysql.com/doc/refman/8.0/en/rename-table.html
- MySQL 8.0 Reference Manual – `TRUNCATE TABLE` – https://dev.mysql.com/doc/refman/8.0/en/truncate-table.html
- MySQL 8.0 Reference Manual – `DROP TABLE` – https://dev.mysql.com/doc/refman/8.0/en/drop-table.html
- MySQL 8.0 Reference Manual – Derived Tables – https://dev.mysql.com/doc/refman/8.0/en/derived-tables.html
- MySQL 8.0 Reference Manual – `CREATE TABLE ... SELECT` – https://dev.mysql.com/doc/refman/8.0/en/create-table-select.html
- Microsoft Learn – `CREATE TABLE` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql
- Microsoft Learn – `sp_rename` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-rename-transact-sql
- Microsoft Learn – `TRUNCATE TABLE` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/truncate-table-transact-sql
- Microsoft Learn – `DROP TABLE` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/drop-table-transact-sql
- Microsoft Learn – Temporary Tables – https://learn.microsoft.com/en-us/sql/t-sql/language-reference/temporary-tables
- Microsoft Learn – `SELECT ... INTO` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/queries/select-into-clause-transact-sql
- AWS – Working with Unlogged Tables in Aurora PostgreSQL – https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.BestPractices.html
- Crunchy Data – PostgreSQL Unlogged Tables: Look Ma, No WAL – https://www.crunchydata.com/blog/postgresql-unlogged-tables-look-ma-no-wal