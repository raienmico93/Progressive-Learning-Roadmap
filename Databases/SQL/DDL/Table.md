# SQL Table Fundamentals: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL table fundamentals encompass the essential operations and concepts for creating, viewing, renaming, truncating, dropping, copying, and organizing tables—the primary structures that store data in a relational database.

**Technical Definition:** A table is a two-dimensional data structure consisting of rows (records) and columns (fields), each with a declared data type. Table fundamentals cover the Data Definition Language (DDL) statements (`CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`, `TRUNCATE TABLE`) and the metadata queries (`INFORMATION_SCHEMA`, `pg_catalog`, `sys.tables`) that define, inspect, and manage tables. This includes specialized table types (temporary, derived, partitioned) and storage organizations (heap-organized vs. index-organized), as well as copy semantics (shallow vs. deep) and lifecycle operations.

**Beginner-Friendly Explanation:** A table is like a spreadsheet. It has columns (like "Name", "Age", "Email") and rows (each row is one record, like a person's information). Table fundamentals are the basic operations you need to know: how to create a table, look at its structure, rename it, empty it, delete it, copy it, and organize it for performance.

### Key Characteristics

- **DDL-driven:** Tables are created, altered, and dropped using Data Definition Language statements.
- **Metadata-rich:** Every table has associated metadata (column definitions, constraints, indexes, storage parameters) stored in system catalogs.
- **Vendor-specific syntax:** While the SQL standard defines basic `CREATE TABLE`, each RDBMS (PostgreSQL, MySQL, SQL Server, Oracle) implements significant variations.
- **Lifecycle-managed:** Tables can be permanent, temporary, or transient; they can be renamed, truncated, or dropped.
- **Storage-aware:** Tables can be heap-organized (unordered) or index-organized (B-tree sorted), and can be partitioned for performance.

### Prerequisites

- Basic understanding of the relational model (rows, columns, keys).
- Familiarity with SQL data types (INTEGER, VARCHAR, DATE, BOOLEAN, etc.).
- Knowledge of primary keys, foreign keys, and constraints.
- Awareness of the target DBMS's syntax and limitations.

### Related Programming Areas

- Database schema design and normalization.
- ETL/ELT pipelines and data warehousing.
- Application development and ORM mapping.
- Database administration and performance tuning.
- Multi-tenant SaaS architecture.

### Core Concepts / Features

1. **Creating Tables** (basic syntax, CTAS vs. SELECT INTO)
2. **Viewing Table Definitions** (system catalogs, SHOW CREATE TABLE, sp_help)
3. **Listing Tables** (metadata views, cross-schema visibility)
4. **Renaming Tables** (ALTER TABLE ... RENAME TO, dependent objects)
5. **Truncating Tables** (TRUNCATE vs. DELETE, log space optimization)
6. **Dropping Tables** (DROP TABLE, CASCADE, recycle bin)
7. **Temporary Tables** (global vs. local, session vs. transaction scope)
8. **Table Cloning** (deep vs. shallow copy)
9. **Derived Tables** (inline views, execution scopes)
10. **Table Metadata** (data dictionaries, row counts, storage)
11. **Table Partitioning & Organization** (range, list, hash; heap vs. index-organized)


## Core Concept 1: Creating Tables

### Definitions

**Core Definition:** Creating a table is the operation of defining a new table structure—including its columns, data types, and constraints—using the `CREATE TABLE` statement.

**Technical Definition:** `CREATE TABLE` is a DDL statement that defines a new table in the database. The statement specifies the table name, column names and data types, and optional constraints (PRIMARY KEY, NOT NULL, UNIQUE, CHECK, FOREIGN KEY). The `CREATE TABLE AS SELECT` (CTAS) variant creates a table and populates it with the results of a query in a single step. SQL Server uses `SELECT ... INTO` instead of CTAS for the same purpose. Implementations vary wildly: MySQL uses `AUTO_INCREMENT` for auto-generated keys, PostgreSQL uses `SERIAL`, SQL Server uses `IDENTITY`, and Oracle uses `SEQUENCE` or `GENERATED ALWAYS AS IDENTITY`.

**Beginner-Friendly Explanation:** Creating a table is like designing a form. You decide what fields (columns) the form will have, what type of data goes in each field (text, number, date), and what rules apply (this field is required, this ID must be unique). You can also create a table by copying the results of a query—like making a new spreadsheet from an existing report.

### Purposes

- **To define the structure** for storing persistent data in a relational database.
- **To enforce data integrity** through column data types and constraints.
- **To create a table from an existing query** (CTAS/SELECT INTO) for backups, reports, or staging.
- **To provide a target** for data manipulation operations (INSERT, UPDATE, DELETE, SELECT).

### Syntax Rules and Structure

#### Complete General Syntax (Basic CREATE TABLE)

```sql
CREATE TABLE [IF NOT EXISTS] table_name (
    column_name data_type [column_constraint [ ... ]]
    [, ...]
    [, table_constraint ]
);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `CREATE TABLE` | Keyword beginning the table creation statement. |
| `table_name` | The name of the table (optionally schema-qualified). |
| `column_name data_type` | Column definition with name and data type. |
| `column_constraint` | Optional constraints on the column (NOT NULL, DEFAULT, CHECK, etc.). |
| `table_constraint` | Optional constraints on the table (PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK). |
| `IF NOT EXISTS` | Suppresses error if the table already exists (MySQL, PostgreSQL, SQLite). |

#### Complete General Syntax (CTAS / SELECT INTO)

```sql
-- PostgreSQL, MySQL, Oracle, SQLite
CREATE TABLE new_table AS
SELECT column1, column2, ...
FROM source_table
WHERE condition;

-- SQL Server
SELECT column1, column2, ...
INTO new_table
FROM source_table
WHERE condition;
```

#### Syntax Rules

- **Basic CREATE TABLE:** Column definitions are separated by commas. Constraints can be inline (column-level) or out-of-line (table-level).
- **CTAS:** Creates a new table and populates it with the results of the `SELECT` statement. Column names and data types are inferred from the query result.
- **SELECT INTO (SQL Server):** Creates a new table in the default filegroup and inserts the resulting rows from the query into it.
- **CTAS vs. SELECT INTO:** `CREATE TABLE AS` is functionally similar to `SELECT INTO`, but it is preferred since it is less likely to be confused with other uses of the `SELECT INTO` syntax. Furthermore, `CREATE TABLE AS` offers a superset of the functionality offered by `SELECT INTO`.

#### Constraints and Limitations

- **CTAS does not copy constraints:** CTAS copies column names and data types only; primary keys, indexes, and constraints are not copied.
- **CTAS does not copy identity properties:** Auto-increment/identity columns are not carried over to the new table.
- **SELECT INTO is SQL Server-specific:** Not available in PostgreSQL, MySQL, or Oracle (use CTAS instead).
- **MySQL:** `CREATE TABLE ... AS SELECT` does not preserve indexes.
- **Column order:** In CTAS, the new table's column order matches the `SELECT` list, not the source table.

### Annotated Code Examples

#### Example 1: PostgreSQL — Basic CREATE TABLE with Constraints

```sql
-- Create a table with various constraints
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    salary NUMERIC(10,2) CHECK (salary > 0),
    hire_date DATE DEFAULT CURRENT_DATE,
    dept_id INT REFERENCES departments(dept_id)
);
```

**Expected Output:**

```
CREATE TABLE
```

**Why This Works:** The `SERIAL` type auto-generates the `employee_id`. `NOT NULL` ensures names are always provided. `UNIQUE` prevents duplicate emails. `CHECK (salary > 0)` enforces positive salaries. `DEFAULT CURRENT_DATE` sets the hire date automatically. `REFERENCES` creates a foreign key to the `departments` table.

#### Example 2: PostgreSQL — CREATE TABLE AS SELECT (CTAS)

```sql
-- Create a table from a query
CREATE TABLE high_earners AS
SELECT employee_id, first_name, last_name, salary
FROM employees
WHERE salary > 100000;
```

**Expected Output:**

```
SELECT 15
```

**Why This Works:** The CTAS statement creates a new table `high_earners` with columns inferred from the `SELECT` list and populates it with rows where `salary > 100000`. Note that no primary key, indexes, or constraints are copied.

#### Example 3: SQL Server — SELECT INTO

```sql
-- Create a copy of a table using SELECT INTO
SELECT *
INTO EmployeesBackup
FROM Employees;
```

**Expected Output:**

```
(1500 rows affected)
```

**Why This Works:** `SELECT ... INTO` creates a new table `EmployeesBackup` in the default filegroup and inserts all rows from `Employees`. The new table has the same column names and data types, but no indexes or constraints.

### Real-World Cases

- **Application provisioning:** Creating core tables (users, orders, products) during initial setup.
- **Backup tables:** Using CTAS/SELECT INTO to create quick backups before major changes.
- **ETL staging:** Creating staging tables from source queries for transformation.
- **Reporting:** Creating summary tables from complex analytical queries.

### References

- PostgreSQL: CREATE TABLE — https://www.postgresql.org/docs/current/sql-createtable.html
- PostgreSQL: CREATE TABLE AS — https://www.postgresql.org/docs/current/sql-createtableas.html
- MySQL: CREATE TABLE — https://dev.mysql.com/doc/refman/8.0/en/create-table.html
- SQL Server: CREATE TABLE — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql
- SQL Server: SELECT - INTO Clause — https://learn.microsoft.com/en-us/sql/t-sql/queries/select-into-clause-transact-sql
- Oracle: CREATE TABLE — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CREATE-TABLE.html


## Core Concept 2: Viewing Table Definitions

### Definitions

**Core Definition:** Viewing a table definition is the operation of retrieving the metadata that describes a table's structure—including column names, data types, constraints, indexes, and storage parameters—using system catalogs or vendor-specific commands.

**Technical Definition:** Every RDBMS provides system catalogs and commands for inspecting table definitions. PostgreSQL offers `information_schema` (SQL-standard views) and `pg_catalog` (native system tables such as `pg_tables` and `pg_class`), plus the `\d` meta-command in `psql`. MySQL provides `SHOW CREATE TABLE` (which displays the exact DDL that recreates the table) and `SHOW COLUMNS`, plus `information_schema.COLUMNS` and `information_schema.TABLES`. SQL Server uses `sp_help` (a system stored procedure returning column information, constraints, and indexes) and system catalog views like `sys.columns` and `INFORMATION_SCHEMA.COLUMNS`. Oracle uses `DESCRIBE` (SQL*Plus command) and data dictionary views like `USER_TAB_COLUMNS` and `USER_TABLES`.

**Beginner-Friendly Explanation:** Viewing a table definition is like looking at the blueprint of a building. You can see all the rooms (columns), what they are used for (data types), and the rules (constraints). Different database systems have different commands to show you this blueprint—some use a simple command, others require querying system tables.

### Purposes

- **To understand an existing table's structure** before writing queries or making modifications.
- **To generate DDL scripts** for migration or documentation.
- **To audit schema consistency** across environments.
- **To debug data issues** by verifying column types and constraints.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL — \d and information_schema)

```sql
-- psql meta-command (detailed)
\d table_name

-- Query information_schema
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_name = 'employees';

-- Query pg_catalog
SELECT schemaname, tablename, tableowner, tablespace, hasindexes
FROM pg_tables
WHERE tablename = 'employees';
```

#### Complete General Syntax (MySQL — SHOW CREATE TABLE)

```sql
SHOW CREATE TABLE table_name;
SHOW CREATE TABLE table_name\G  -- Vertical format
```

#### Complete General Syntax (SQL Server — sp_help)

```sql
EXEC sp_help 'dbo.mytable';
-- Or query INFORMATION_SCHEMA
SELECT COLUMN_NAME, DATA_TYPE, CHARACTER_MAXIMUM_LENGTH, IS_NULLABLE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'mytable';
```

#### Complete General Syntax (Oracle — DESCRIBE)

```sql
DESCRIBE table_name;

-- Query data dictionary
SELECT column_name, data_type, data_length, nullable, data_default
FROM USER_TAB_COLUMNS
WHERE table_name = 'EMPLOYEES';
```

#### Syntax Rules

- **PostgreSQL:** The `\d` meta-command is the fastest way to view a table definition in `psql`. The `information_schema.columns` view provides a standardized, SQL-compliant view of column metadata.
- **MySQL:** `SHOW CREATE TABLE` displays the exact `CREATE TABLE` DDL statement that MySQL would use to recreate the table, including all column definitions, data types, constraints, indexes, partitioning clauses, storage engine settings, character set, collation, and auto_increment value.
- **SQL Server:** `sp_help` returns several result sets describing the table, its columns, and constraints. The `Alt+F1` keyboard shortcut in SSMS executes `sp_help` for the highlighted object.
- **Oracle:** `DESCRIBE` is a SQL*Plus command that displays column names, data types, and nullability.

#### Constraints and Limitations

- **PostgreSQL:** `information_schema` views are standardized but may not show all PostgreSQL-specific features (e.g., storage parameters, tablespaces).
- **MySQL:** `SHOW CREATE TABLE` requires some privilege on the table; `information_schema` is accessible to all users but shows only accessible tables.
- **SQL Server:** `sp_help` output is formatted for human reading; for programmatic access, query system catalog views directly.
- **Oracle:** `DESCRIBE` is not a SQL statement; it is a SQL*Plus command. From other clients, query `USER_TAB_COLUMNS`.

### Annotated Code Examples

#### Example 1: PostgreSQL — Viewing a Table Definition

```sql
-- Using psql meta-command
\d employees
```

**Expected Output:**

```
                                    Table "public.employees"
   Column    |          Type          | Collation | Nullable |           Default
-------------+------------------------+-----------+----------+-----------------------------
 employee_id | integer                |           | not null | nextval('employees_employee_id_seq'::regclass)
 first_name  | character varying(50)  |           | not null |
 last_name   | character varying(50)  |           | not null |
 email       | character varying(100) |           |          |
 salary      | numeric(10,2)          |           |          |
 hire_date   | date                   |           |          | CURRENT_DATE
 dept_id     | integer                |           |          |
Indexes:
    "employees_pkey" PRIMARY KEY, btree (employee_id)
    "employees_email_key" UNIQUE CONSTRAINT, btree (email)
Foreign-key constraints:
    "employees_dept_id_fkey" FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
```

**Why This Works:** The `\d` meta-command displays the complete table definition, including columns, data types, defaults, indexes, and foreign key constraints in a human-readable format.

#### Example 2: MySQL — SHOW CREATE TABLE

```sql
SHOW CREATE TABLE orders\G
```

**Expected Output:**

```
*************************** 1. row ***************************
       Table: orders
Create Table: CREATE TABLE `orders` (
  `id` int NOT NULL AUTO_INCREMENT,
  `customer_id` int NOT NULL,
  `status` varchar(20) DEFAULT 'pending',
  `total` decimal(10,2) NOT NULL DEFAULT '0.00',
  `created_at` datetime NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_customer_id` (`customer_id`),
  CONSTRAINT `fk_orders_customer` FOREIGN KEY (`customer_id`) REFERENCES `customers` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=10001 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
```

**Why This Works:** `SHOW CREATE TABLE` displays the exact DDL statement, including all column definitions, indexes, foreign keys, and storage engine settings. The `\G` modifier formats the output vertically for readability.

### Real-World Cases

- **Documentation:** Generating schema documentation from existing tables.
- **Migration:** Scripting table definitions for migration to another environment.
- **Debugging:** Verifying column types and constraints when queries behave unexpectedly.
- **Onboarding:** Helping new developers understand the database structure.

### References

- PostgreSQL: System Catalogs — https://www.postgresql.org/docs/current/catalogs.html
- PostgreSQL: information_schema — https://www.postgresql.org/docs/current/information-schema.html
- MySQL: SHOW CREATE TABLE — https://dev.mysql.com/doc/refman/8.0/en/show-create-table.html
- SQL Server: sp_help — https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-help-transact-sql
- SQL Server: View the Table Definition — https://learn.microsoft.com/en-us/sql/relational-databases/tables/view-the-table-definition
- Oracle: DESCRIBE — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/DESCRIBE.html


## Core Concept 3: Listing Tables

### Definitions

**Core Definition:** Listing tables is the operation of retrieving the names and metadata of all tables available in a database or schema, using system catalog views or vendor-specific commands.

**Technical Definition:** Each RDBMS provides system catalog views and commands for enumerating tables. The SQL standard defines `INFORMATION_SCHEMA.TABLES`, which is supported by PostgreSQL, MySQL, and SQL Server. Oracle provides `SYS.ALL_TABLES`, `USER_TABLES`, and `DBA_TABLES`. PostgreSQL additionally provides `pg_catalog.pg_tables` and the `\dt` meta-command in `psql`. MySQL provides `SHOW TABLES` and `information_schema.TABLES`. SQL Server provides `sys.tables` and `INFORMATION_SCHEMA.TABLES`.

**Beginner-Friendly Explanation:** Listing tables is like looking at a directory of all the tables in your database. Different database systems have different ways to show you this list—some use a simple command, others require querying system tables.

### Purposes

- **To discover available tables** in a database or schema.
- **To audit table inventory** for compliance and documentation.
- **To programmatically check** for the existence of a table before creating or dropping it.
- **To handle cross-schema visibility** by filtering tables by schema.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL — \dt and pg_tables)

```sql
-- psql meta-command
\dt
\dt schema_name.*

-- Query pg_catalog
SELECT schemaname, tablename, tableowner, tablespace, hasindexes
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema');

-- Query information_schema
SELECT table_schema, table_name, table_type
FROM information_schema.tables
WHERE table_schema NOT IN ('pg_catalog', 'information_schema');
```

#### Complete General Syntax (MySQL — SHOW TABLES)

```sql
SHOW TABLES;

-- Query information_schema
SELECT table_schema, table_name, table_type, engine, table_rows
FROM information_schema.tables
WHERE table_schema = 'mydb';
```

#### Complete General Syntax (SQL Server — sys.tables)

```sql
SELECT name, schema_id, create_date, modify_date
FROM sys.tables;

-- Or query INFORMATION_SCHEMA
SELECT TABLE_SCHEMA, TABLE_NAME, TABLE_TYPE
FROM INFORMATION_SCHEMA.TABLES;
```

#### Complete General Syntax (Oracle — ALL_TABLES)

```sql
SELECT table_name, tablespace_name, num_rows, last_analyzed
FROM user_tables;

-- Or for tables accessible to the current user:
SELECT owner, table_name, tablespace_name, num_rows
FROM all_tables;
```

#### Syntax Rules

- **PostgreSQL:** `pg_tables` provides access to useful information about each table in the database.
- **MySQL:** `SHOW TABLES` lists tables in the current database. `information_schema.TABLES` provides additional metadata (engine, row count estimate, collation).
- **SQL Server:** `sys.tables` is the system catalog view for tables; `INFORMATION_SCHEMA.TABLES` is the SQL-standard equivalent.
- **Oracle:** `USER_TABLES` shows tables owned by the current user; `ALL_TABLES` shows tables accessible to the current user; `DBA_TABLES` shows all tables (requires DBA privileges).

#### Constraints and Limitations

- **Cross-schema visibility:** In PostgreSQL and SQL Server, `information_schema.TABLES` shows only tables the current user has privileges on. Filter by `table_schema` to list tables in a specific schema.
- **MySQL:** `SHOW TABLES` returns only tables in the currently selected database; use `SHOW TABLES FROM database_name` for another database.
- **Oracle:** `USER_TABLES` includes only tables owned by the current schema; use `ALL_TABLES` for cross-schema visibility.
- **Row counts:** `num_rows` in Oracle's `USER_TABLES` and `TABLE_ROWS` in MySQL's `information_schema.TABLES` are estimates, not exact counts.

### Annotated Code Examples

#### Example 1: PostgreSQL — Listing Tables in a Schema

```sql
-- List all tables in the 'sales' schema
SELECT schemaname, tablename, tableowner, hasindexes
FROM pg_tables
WHERE schemaname = 'sales'
ORDER BY tablename;
```

**Expected Output:**

```
 schemaname |   tablename    | tableowner | hasindexes
------------+----------------+------------+------------
 sales      | customers      | sales_admin| t
 sales      | order_items    | sales_admin| t
 sales      | orders         | sales_admin| t
 sales      | products       | sales_admin| t
```

**Why This Works:** The `pg_tables` view contains one row per table in the database. Filtering by `schemaname = 'sales'` restricts the output to tables in the sales schema.

#### Example 2: SQL Server — Listing Tables with Row Counts

```sql
SELECT
    s.name AS schema_name,
    t.name AS table_name,
    p.rows AS row_count
FROM sys.tables t
JOIN sys.schemas s ON t.schema_id = s.schema_id
JOIN sys.partitions p ON t.object_id = p.object_id AND p.index_id IN (0, 1)
WHERE s.name NOT IN ('sys', 'INFORMATION_SCHEMA')
ORDER BY s.name, t.name;
```

**Expected Output:**

```
schema_name | table_name  | row_count
------------+-------------+-----------
dbo         | Customers   |      1500
dbo         | Orders      |     15000
dbo         | Products    |       250
sales       | OrderItems  |     45000
```

**Why This Works:** The query joins `sys.tables` with `sys.schemas` and `sys.partitions` to retrieve table names, schema names, and approximate row counts. Filtering `index_id IN (0, 1)` selects the heap or clustered index partition.

### Real-World Cases

- **Inventory auditing:** Generating a report of all tables in a database for compliance.
- **Automated provisioning:** Scripts checking `information_schema.TABLES` before creating a new table.
- **Cross-schema queries:** Identifying tables across multiple schemas for migration or documentation.
- **Development onboarding:** Helping new developers discover available tables.

### References

- PostgreSQL: pg_tables — https://www.postgresql.org/docs/current/view-pg-tables.html
- PostgreSQL: information_schema.tables — https://www.postgresql.org/docs/current/infoschema-tables.html
- MySQL: SHOW TABLES — https://dev.mysql.com/doc/refman/8.0/en/show-tables.html
- SQL Server: sys.tables — https://learn.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-tables-transact-sql
- Oracle: USER_TABLES — https://docs.oracle.com/en/database/oracle/oracle-database/21/refrn/USER_TABLES.html


## Core Concept 4: Renaming Tables

### Definitions

**Core Definition:** Renaming a table is the operation of changing a table's name without altering its structure or contents, using `ALTER TABLE ... RENAME TO` (PostgreSQL, MySQL, Oracle) or `sp_rename` (SQL Server).

**Technical Definition:** `ALTER TABLE ... RENAME TO` is a DDL statement that changes the name of a table in the system catalog. The table's data, indexes, constraints, and triggers remain intact. In SQL Server, the `sp_rename` system stored procedure is used instead. Dependent objects—views, stored procedures, and foreign key constraints—may or may not be automatically updated depending on the RDBMS. In PostgreSQL, Oracle, and MySQL, dependent objects are automatically updated. In SQL Server, `sp_rename` does not automatically update references in stored procedures or views.

**Beginner-Friendly Explanation:** Renaming a table is like changing the label on a filing cabinet. The contents inside do not change—only the name on the outside. However, if other documents (views, procedures) refer to the old name, they may need to be updated.

### Purposes

- **To align table names** with updated naming conventions or business terminology.
- **To correct naming errors** made during initial creation.
- **To prepare a table for migration** or handover with a more descriptive name.
- **To support versioning** (e.g., renaming `orders` to `orders_archive` before creating a new `orders` table).

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL, MySQL, Oracle)

```sql
ALTER TABLE current_table_name RENAME TO new_table_name;
```

#### Complete General Syntax (MySQL — RENAME TABLE)

```sql
RENAME TABLE current_table_name TO new_table_name;
-- Can rename multiple tables in one statement:
RENAME TABLE t1 TO t1_new, t2 TO t2_new;
```

#### Complete General Syntax (SQL Server)

```sql
EXEC sp_rename 'current_table_name', 'new_table_name';
```

#### Syntax Rules

- **PostgreSQL:** `ALTER TABLE ... RENAME TO` automatically updates references in views, foreign key constraints, and indexes that depend on the table.
- **MySQL:** Both `RENAME TABLE` and `ALTER TABLE ... RENAME TO` are supported. `RENAME TABLE` can rename multiple tables in a single statement and is atomic.
- **Oracle:** `ALTER TABLE ... RENAME TO` or the standalone `RENAME old_name TO new_name` statement.
- **SQL Server:** `sp_rename` requires the table name (optionally schema-qualified) and the new name. The procedure warns that changing any part of an object name could break scripts and stored procedures.

#### Constraints and Limitations

- **SQL Server:** `sp_rename` does not automatically update references in stored procedures, views, or functions; these must be manually updated.
- **MySQL:** `RENAME TABLE` fails if any of the tables are locked or have active transactions.
- **PostgreSQL:** Renaming a table does not rename the associated sequence (for `SERIAL` columns); the sequence retains its original name.
- **Oracle:** Renaming a table invalidates dependent views and procedures; they must be recompiled.

### Annotated Code Examples

#### Example 1: PostgreSQL — Renaming a Table

```sql
-- Create a table
CREATE TABLE customers_old (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

-- Rename the table
ALTER TABLE customers_old RENAME TO customers;

-- Verify
SELECT tablename FROM pg_tables WHERE tablename = 'customers';
```

**Expected Output:**

```
 tablename
-----------
 customers
```

**Why This Works:** The `ALTER TABLE ... RENAME TO` statement changes the table name from `customers_old` to `customers`. Dependent objects (indexes, constraints) are automatically updated.

#### Example 2: SQL Server — Using sp_rename

```sql
-- Rename a table using sp_rename
EXEC sp_rename 'dbo.employee_data', 'employees';
```

**Expected Output:**

```
Caution: Changing any part of an object name could break scripts and stored procedures.
```

**Why This Works:** `sp_rename` changes the table name. The warning reminds you that dependent objects may need manual updates.

### Real-World Cases

- **Naming convention alignment:** Renaming `tbl_orders` to `orders` for consistency.
- **Archive and reset:** Renaming `orders` to `orders_2025` before creating a fresh `orders` table for the new year.
- **Migration preparation:** Renaming tables during a database migration.
- **Typo correction:** Fixing misspelled table names.

### References

- PostgreSQL: ALTER TABLE ... RENAME TO — https://www.postgresql.org/docs/current/sql-altertable.html
- MySQL: RENAME TABLE — https://dev.mysql.com/doc/refman/8.0/en/rename-table.html
- SQL Server: sp_rename — https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-rename-transact-sql
- Oracle: ALTER TABLE — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/ALTER-TABLE.html


## Core Concept 5: Truncating Tables

### Definitions

**Core Definition:** Truncating a table is the operation of rapidly removing all rows from a table using `TRUNCATE TABLE`, which deallocates data pages rather than deleting rows individually.

**Technical Definition:** `TRUNCATE TABLE` removes all rows from a table without logging the individual row deletions. It is similar to the `DELETE` statement with no `WHERE` clause; however, `TRUNCATE TABLE` is faster and uses fewer system and transaction log resources. The `DELETE` statement removes rows one at a time and records an entry in the transaction log for each deleted row. `TRUNCATE TABLE` removes the data by deallocating the data pages used to store the table data and records only the page deallocations in the transaction log. The table structure, columns, constraints, indexes, and triggers remain intact.

**Beginner-Friendly Explanation:** Truncating a table is like emptying a filing cabinet by removing the entire drawer instead of taking out each folder one by one. It is much faster, but you lose all the data. The cabinet itself (table structure) stays the same.

### Purposes

- **To rapidly empty a table** while preserving its structure for reuse.
- **To reset a table** in test or development environments.
- **To clear staging tables** after ETL processing.
- **To reclaim storage space** more efficiently than `DELETE`.

### Syntax Rules and Structure

#### Complete General Syntax

```sql
TRUNCATE TABLE [database_name.[schema_name]. | schema_name.]table_name;
```

#### Syntax Rules

- **TRUNCATE vs. DELETE:** `TRUNCATE TABLE` deallocates data pages; `DELETE` removes rows one at a time. `TRUNCATE` is faster and uses less transaction log space.
- **Identity reset:** If the table contains an identity column, the counter for that column is reset to the seed value defined for the column. If no seed was defined, the default value 1 is used. To retain the identity counter, use `DELETE` instead.
- **Triggers:** `TRUNCATE TABLE` cannot activate a trigger because the operation does not log individual row deletions.
- **Foreign keys:** You cannot use `TRUNCATE TABLE` on tables that are referenced by a `FOREIGN KEY` constraint. (You can truncate a table that has a foreign key that references itself.)
- **Indexed views:** Cannot truncate tables that participate in an indexed view.
- **Replication:** Cannot truncate tables published by transactional or merge replication.

#### Constraints and Limitations

- **Irreversibility:** `TRUNCATE TABLE` is not logged per-row; it cannot be rolled back in some databases (though it is transactional in PostgreSQL and SQL Server).
- **Foreign key dependencies:** Tables referenced by foreign keys cannot be truncated without dropping the foreign key first.
- **Triggers:** `TRUNCATE` does not fire `DELETE` triggers.
- **Permissions:** Requires `ALTER` permission on the table (SQL Server) or `TRUNCATE` privilege (PostgreSQL).

### Annotated Code Examples

#### Example 1: PostgreSQL — Truncating a Table

```sql
-- Create and populate a table
CREATE TABLE temp_data (id SERIAL PRIMARY KEY, value TEXT);
INSERT INTO temp_data (value) VALUES ('a'), ('b'), ('c');

-- Truncate the table
TRUNCATE TABLE temp_data;

-- Verify (identity sequence is NOT reset in PostgreSQL by default)
SELECT * FROM temp_data;
SELECT currval('temp_data_id_seq');
```

**Expected Output:**

```
 id | value
----+-------
(0 rows)

 currval
---------
       3
```

**Why This Works:** `TRUNCATE TABLE` removes all rows by deallocating pages. In PostgreSQL, the identity sequence is not reset by default (use `TRUNCATE ... RESTART IDENTITY` to reset). The table structure remains intact.

#### Example 2: SQL Server — Truncating a Table

```sql
-- Truncate a table (identity is reset to 1)
TRUNCATE TABLE dbo.StagingOrders;

-- Verify
SELECT COUNT(*) FROM dbo.StagingOrders;
```

**Expected Output:**

```
-----------
          0
```

**Why This Works:** SQL Server's `TRUNCATE TABLE` resets the identity counter to the seed value (default 1). The table is now empty but the structure remains.

### Real-World Cases

- **ETL staging:** Truncating staging tables before loading new data.
- **Test environments:** Resetting tables to a clean state between test runs.
- **Log cleanup:** Rapidly removing all rows from a log table before archiving.
- **Session data:** Clearing temporary session data after a user logs out.

### References

- PostgreSQL: TRUNCATE — https://www.postgresql.org/docs/current/sql-truncate.html
- MySQL: TRUNCATE TABLE — https://dev.mysql.com/doc/refman/8.0/en/truncate-table.html
- SQL Server: TRUNCATE TABLE — https://learn.microsoft.com/en-us/sql/t-sql/statements/truncate-table-transact-sql
- Oracle: TRUNCATE TABLE — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/TRUNCATE-TABLE.html


## Core Concept 6: Dropping Tables

### Definitions

**Core Definition:** Dropping a table is the operation of permanently removing a table, including its data, structure, indexes, triggers, and constraints, from the database using the `DROP TABLE` statement.

**Technical Definition:** `DROP TABLE` is a DDL statement that removes a table from the database. All views and PL/SQL program units dependent on a dropped table remain, yet become invalid (not usable). All synonyms for a dropped table remain, but return an error when used. All extents allocated for a table that is dropped are returned to the free space of the tablespace and can be used by any other object requiring new extents or new objects. In Oracle, when you drop a table, normally the database does not immediately release the space associated with the table. Rather, the database renames the table and places it in a recycle bin, where it can later be recovered with the `FLASHBACK TABLE` statement. The `PURGE` clause releases the space immediately.

**Beginner-Friendly Explanation:** Dropping a table is like demolishing a building and removing all its contents. Everything is gone—the structure, the data, the rules. In Oracle, it is like moving the building to a temporary storage area (recycle bin) where you can bring it back if you made a mistake.

### Purposes

- **To remove obsolete tables** that are no longer needed.
- **To clean up test or temporary tables** after a project is complete.
- **To reset a database** by dropping and recreating tables.
- **To free storage space** occupied by unused tables.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL, MySQL, SQL Server)

```sql
DROP TABLE [IF EXISTS] table_name [, ...] [CASCADE | RESTRICT];
```

#### Complete General Syntax (Oracle)

```sql
DROP TABLE table_name [CASCADE CONSTRAINTS] [PURGE];
```

#### Syntax Rules

- **CASCADE:** Automatically drops objects that depend on the table (e.g., views, foreign key constraints).
- **RESTRICT:** Refuses to drop the table if any objects depend on it (default in PostgreSQL).
- **IF EXISTS:** Suppresses error if the table does not exist.
- **Oracle CASCADE CONSTRAINTS:** Drops all referential integrity constraints that refer to primary and unique keys in the dropped table.
- **Oracle PURGE:** Drops the table and releases the space associated with it in a single step. If `PURGE` is specified, the database does not place the table and its dependent objects into the recycle bin.

#### Constraints and Limitations

- **Irreversibility:** In most databases, `DROP TABLE` is irreversible unless the table is in the Oracle recycle bin.
- **Dependent objects:** Views, procedures, and functions that reference the table become invalid.
- **Foreign keys:** Tables referenced by foreign keys cannot be dropped without `CASCADE` or dropping the foreign key first.
- **Recycle bin (Oracle):** Dropped tables remain in the recycle bin until purged, consuming storage space.

### Annotated Code Examples

#### Example 1: PostgreSQL — Dropping a Table with CASCADE

```sql
-- Drop a table and all dependent objects
DROP TABLE orders CASCADE;
```

**Expected Output:**

```
NOTICE:  drop cascades to view order_summary
NOTICE:  drop cascades to constraint fk_order_customer on table order_items
DROP TABLE
```

**Why This Works:** `CASCADE` drops the `orders` table and all objects that depend on it, including the `order_summary` view and the foreign key constraint on `order_items`.

#### Example 2: Oracle — Dropping a Table with PURGE

```sql
-- Drop a table and immediately release space
DROP TABLE hr.admin_emp PURGE;
```

**Expected Output:**

```
Table dropped.
```

**Why This Works:** The `PURGE` clause drops the table and releases the space immediately, bypassing the recycle bin. Without `PURGE`, the table would be placed in the recycle bin and could be recovered with `FLASHBACK TABLE`.

### Real-World Cases

- **Environment cleanup:** Dropping temporary tables after automated testing.
- **Data retention:** Dropping tables that contain data past its retention period.
- **Schema refactoring:** Dropping obsolete tables during a major version upgrade.
- **Disk space reclamation:** Dropping unused tables to free storage.

### References

- PostgreSQL: DROP TABLE — https://www.postgresql.org/docs/current/sql-droptable.html
- MySQL: DROP TABLE — https://dev.mysql.com/doc/refman/8.0/en/drop-table.html
- SQL Server: DROP TABLE — https://learn.microsoft.com/en-us/sql/t-sql/statements/drop-table-transact-sql
- Oracle: DROP TABLE — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/DROP-TABLE.html


## Core Concept 7: Temporary Tables

### Definitions

**Core Definition:** A temporary table is a special table that stores data temporarily, either for the duration of a session (session-scoped) or a transaction (transaction-scoped), and is automatically dropped when the scope ends.

**Technical Definition:** Temporary tables are database objects whose data is visible only within a specific scope. In PostgreSQL, `CREATE TEMPORARY TABLE` creates a session-scoped table that persists until the session ends; the `ON COMMIT` clause can be set to `PRESERVE ROWS` (default), `DELETE ROWS`, or `DROP`. In SQL Server, local temporary tables (prefixed with `#`) are visible only to the current session, while global temporary tables (prefixed with `##`) are visible to all sessions and are dropped when the creating session ends. In Oracle, `CREATE GLOBAL TEMPORARY TABLE` creates a table whose structure persists but whose data is private to each session; the `ON COMMIT` clause defaults to `DELETE ROWS` (transaction-scoped) or can be `PRESERVE ROWS` (session-scoped). MySQL's `CREATE TEMPORARY TABLE` creates a session-scoped table.

**Beginner-Friendly Explanation:** A temporary table is like a scratchpad. You use it to hold intermediate results while you work, and when you are done (or when you close the workbook), it is automatically thrown away. Some scratchpads are private to you (local temp tables), while others can be shared (global temp tables).

### Purposes

- **To store intermediate results** for complex queries without polluting the permanent schema.
- **To improve query performance** by materializing expensive subqueries.
- **To enable batch processing** where multiple statements need to share intermediate data.
- **To isolate session-specific data** in multi-user environments.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
CREATE [TEMPORARY | TEMP] TABLE table_name (
    column definitions
) ON COMMIT { PRESERVE ROWS | DELETE ROWS | DROP };
```

#### Complete General Syntax (SQL Server)

```sql
-- Local temporary table
CREATE TABLE #temp_table (column definitions);

-- Global temporary table
CREATE TABLE ##global_temp_table (column definitions);
```

#### Complete General Syntax (Oracle)

```sql
CREATE GLOBAL TEMPORARY TABLE table_name (
    column definitions
) ON COMMIT { DELETE ROWS | PRESERVE ROWS };
```

#### Complete General Syntax (MySQL)

```sql
CREATE TEMPORARY TABLE table_name (
    column definitions
);
```

#### Syntax Rules

- **PostgreSQL:** `ON COMMIT PRESERVE ROWS` (default) keeps data after commit; `ON COMMIT DELETE ROWS` deletes data after commit; `ON COMMIT DROP` drops the table after commit.
- **SQL Server:** Local temp tables (`#`) are visible only in the current session; global temp tables (`##`) are visible to all sessions and are deleted when the creating session ends.
- **Oracle:** `ON COMMIT DELETE ROWS` (default) makes the table transaction-scoped; `ON COMMIT PRESERVE ROWS` makes it session-scoped. The table structure persists after the session ends.
- **MySQL:** Temporary tables are session-scoped and automatically dropped when the session ends.

#### Constraints and Limitations

- **SQL Server:** Global temp tables (`##`) are visible to all sessions but are dropped when the creating session ends. They can cause conflicts if multiple sessions use the same name.
- **Oracle:** Global temporary tables have a permanent structure but temporary data. The data is private to each session.
- **PostgreSQL:** Temporary tables are placed in a special schema (`pg_temp_nnn`) and are searched first by the search path.
- **Indexes:** Indexes can be created on temporary tables, but they are also temporary.
- **Constraints:** Foreign keys cannot reference temporary tables.

### Annotated Code Examples

#### Example 1: PostgreSQL — Temporary Table with ON COMMIT DELETE ROWS

```sql
-- Create a transaction-scoped temp table
CREATE TEMPORARY TABLE temp_orders (
    order_id INT,
    customer_id INT,
    amount NUMERIC(10,2)
) ON COMMIT DELETE ROWS;

-- Begin transaction
BEGIN;
INSERT INTO temp_orders VALUES (1, 101, 150.00);
SELECT COUNT(*) FROM temp_orders;  -- Returns 1
COMMIT;

-- After commit, data is deleted
SELECT COUNT(*) FROM temp_orders;  -- Returns 0
```

**Expected Output:**

```
 count
-------
     1
COMMIT
 count
-------
     0
```

**Why This Works:** The `ON COMMIT DELETE ROWS` clause makes the temporary table transaction-scoped. Data is automatically deleted after each commit.

#### Example 2: SQL Server — Local Temporary Table

```sql
-- Create a local temp table
CREATE TABLE #TempEmployees (
    emp_id INT,
    emp_name NVARCHAR(100)
);

-- Insert data
INSERT INTO #TempEmployees VALUES (1, 'Alice'), (2, 'Bob');

-- Query the temp table
SELECT * FROM #TempEmployees;
```

**Expected Output:**

```
emp_id | emp_name
-------+---------
1      | Alice
2      | Bob
```

**Why This Works:** The `#` prefix creates a local temporary table that is visible only to the current session and is automatically dropped when the session ends.

### Real-World Cases

- **ETL pipelines:** Staging intermediate transformation results in temporary tables.
- **Report generation:** Materializing complex query results for pagination.
- **Batch processing:** Sharing data across multiple statements in a batch.
- **Session state:** Storing user-specific session data temporarily.

### References

- PostgreSQL: CREATE TABLE — https://www.postgresql.org/docs/current/sql-createtable.html
- SQL Server: Temporary Tables — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql
- Oracle: CREATE GLOBAL TEMPORARY TABLE — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CREATE-TABLE.html
- MySQL: CREATE TEMPORARY TABLE — https://dev.mysql.com/doc/refman/8.0/en/create-temporary-table.html


## Core Concept 8: Table Cloning

### Definitions

**Core Definition:** Table cloning is the operation of creating a copy of an existing table, either as a shallow copy (structure only) or a deep copy (structure + data + indexes + constraints).

**Technical Definition:** A shallow copy duplicates the table's column definitions and data types but not its data, indexes, or constraints. A deep copy duplicates the table's structure, data, indexes, and constraints. The method used determines what is copied:

- **Shallow copy (structure only):** `CREATE TABLE new_table LIKE original_table` (PostgreSQL, MySQL) or `CREATE TABLE new_table AS SELECT * FROM original_table WHERE 1=0`.
- **Deep copy (structure + data):** `CREATE TABLE new_table AS SELECT * FROM original_table` (CTAS) or `SELECT * INTO new_table FROM original_table` (SQL Server).
- **Deep copy with indexes and constraints:** Requires scripting the original table's DDL or using a database-specific method. In SQL Server, you can script the table (including indexes and constraints) or use SMO (SQL Server Management Objects).

**Beginner-Friendly Explanation:** Cloning a table is like copying a document. A shallow copy is like copying just the blank form (structure only)—no data, no rules. A deep copy is like copying the form and all its filled-in data. A complete deep copy also copies the rules (constraints) and the index (the table of contents).

### Purposes

- **To create backups** before making significant changes.
- **To create development or test environments** with production-like data.
- **To archive historical data** while maintaining the original table structure.
- **To create template tables** for repeated use.

### Syntax Rules and Structure

#### Complete General Syntax (Shallow Copy — Structure Only)

```sql
-- PostgreSQL, MySQL
CREATE TABLE new_table LIKE original_table;

-- PostgreSQL, MySQL, Oracle, SQLite
CREATE TABLE new_table AS SELECT * FROM original_table WHERE 1=0;

-- SQL Server
SELECT * INTO new_table FROM original_table WHERE 1=0;
```

#### Complete General Syntax (Deep Copy — Structure + Data)

```sql
-- PostgreSQL, MySQL, Oracle, SQLite
CREATE TABLE new_table AS SELECT * FROM original_table;

-- SQL Server
SELECT * INTO new_table FROM original_table;
```

#### Complete General Syntax (Deep Copy — Structure + Data + Indexes + Constraints)

```sql
-- PostgreSQL: Step 1: Create structure with indexes and constraints
CREATE TABLE new_table (LIKE original_table INCLUDING ALL);

-- Step 2: Insert data
INSERT INTO new_table SELECT * FROM original_table;
```

#### Syntax Rules

- **Shallow copy:** `CREATE TABLE ... LIKE` copies column names, data types, primary keys, indexes, and constraints (but not data).
- **CTAS deep copy:** Copies column names and data types only; does not copy primary keys, indexes, or constraints.
- **SELECT INTO deep copy (SQL Server):** Copies basic table structure and data; does not copy indexes, constraints, or triggers.
- **Complete deep copy:** Requires scripting the original table's DDL (including indexes and constraints) and then inserting data.

#### Constraints and Limitations

- **CTAS/SELECT INTO:** Does not copy indexes, constraints, or identity properties.
- **CREATE TABLE ... LIKE:** Copies structure including indexes and constraints, but not data.
- **Complete deep copy:** Requires manual scripting in most databases; no single-statement solution exists.
- **Foreign keys:** Cloning tables with foreign keys requires careful handling to avoid constraint violations.

### Annotated Code Examples

#### Example 1: PostgreSQL — Shallow Copy (Structure Only)

```sql
-- Create a shallow copy (structure including indexes and constraints)
CREATE TABLE employees_backup (LIKE employees INCLUDING ALL);

-- Verify structure
\d employees_backup
```

**Expected Output:**

```
                                    Table "public.employees_backup"
   Column    |          Type          | Collation | Nullable |           Default
-------------+------------------------+-----------+----------+-----------------------------
 employee_id | integer                |           | not null | nextval('employees_employee_id_seq'::regclass)
 first_name  | character varying(50)  |           | not null |
...
Indexes:
    "employees_backup_pkey" PRIMARY KEY, btree (employee_id)
```

**Why This Works:** `CREATE TABLE ... (LIKE ... INCLUDING ALL)` creates a shallow copy that includes column definitions, primary keys, indexes, and constraints—but not data.

#### Example 2: SQL Server — Deep Copy (Structure + Data)

```sql
-- Create a deep copy (structure + data, no indexes/constraints)
SELECT *
INTO EmployeesBackup
FROM Employees;

-- Verify row count
SELECT COUNT(*) AS row_count FROM EmployeesBackup;
```

**Expected Output:**

```
row_count
---------
     1500
```

**Why This Works:** `SELECT ... INTO` creates a new table with the same column structure and copies all data. No indexes or constraints are copied.

### Real-World Cases

- **Backup before migration:** Deep copying a table before a major schema change.
- **Development seeding:** Creating a test table with production-like data.
- **Archive tables:** Creating a shallow copy for an archive table with the same structure.
- **Template tables:** Creating a shallow copy as a template for repeated use.

### References

- PostgreSQL: CREATE TABLE ... LIKE — https://www.postgresql.org/docs/current/sql-createtable.html
- PostgreSQL: CREATE TABLE AS — https://www.postgresql.org/docs/current/sql-createtableas.html
- MySQL: CREATE TABLE ... LIKE — https://dev.mysql.com/doc/refman/8.0/en/create-table-like.html
- SQL Server: SELECT ... INTO — https://learn.microsoft.com/en-us/sql/t-sql/queries/select-into-clause-transact-sql


## Core Concept 9: Derived Tables

### Definitions

**Core Definition:** A derived table is an inner subquery placed within the `FROM` or `JOIN` clause of a SQL statement, acting as a transient, virtual table for the scope of the parent query.

**Technical Definition:** A derived table (also called an inline view or subquery in the `FROM` clause) is an inner subquery block placed within the `FROM` or `JOIN` clauses of a SQL statement. In relational algebra, a derived table creates a transient, virtual relation that can be queried, joined, aggregated, and filtered like a base disk table for the scope of the parent query execution context. Derived tables require mandatory table alias assignment in ANSI SQL (`FROM (...) AS derived_alias`). This enforces explicit scope boundaries for column projections.

**Beginner-Friendly Explanation:** A derived table is like a temporary view that exists only for the duration of a single query. You write a subquery, give it a name (alias), and then use that name in the outer query as if it were a real table. It is useful for pre-aggregating data before joining it to other tables.

### Purposes

- **To pre-aggregate data** before joining it to other tables.
- **To simplify complex queries** by breaking them into logical layers.
- **To avoid creating permanent views** for one-time queries.
- **To enable filtering on aggregated results** that cannot be done in a `WHERE` clause directly.

### Syntax Rules and Structure

#### Complete General Syntax

```sql
SELECT outer_columns
FROM (
    SELECT inner_columns
    FROM source_table
    WHERE inner_conditions
    GROUP BY grouping_columns
) AS derived_alias
WHERE outer_conditions;
```

#### Syntax Rules

- **Mandatory alias:** Every derived table must have its own alias. In MySQL, the `[AS] tbl_name` clause is mandatory because every table in a `FROM` clause must have a name.
- **Column aliases:** Column aliases can be specified after the derived table alias: `FROM (...) AS derived_alias (col1, col2, ...)`.
- **Scoping:** Columns from the derived table are visible only within the outer query. The derived table cannot reference columns from other tables in the outer `FROM` clause (unless using `LATERAL`).
- **Execution:** The optimizer may flatten (merge) the derived table into the outer query (subquery pull-up) if the derived table is simple (no `GROUP BY`, `DISTINCT`, `LIMIT`). Otherwise, it is materialized as a temporary structure.

#### Constraints and Limitations

- **Mandatory alias:** Omitting the alias causes a syntax error in most databases (`ERROR 1248: Every derived table must have its own alias`).
- **No correlation:** A derived table cannot reference columns from other tables in the same `FROM` clause (unless `LATERAL` is used).
- **Performance:** Derived tables with aggregation are materialized, which can consume memory. Simple derived tables are flattened by the optimizer.
- **Readability:** Deeply nested derived tables can be difficult to read; CTEs are often preferred for complex logic.

### Annotated Code Examples

#### Example 1: PostgreSQL — Derived Table for Pre-Aggregation

```sql
-- Pre-aggregate employee counts per department, then join
SELECT d.dept_name, agg.total_employees
FROM departments d
JOIN (
    SELECT dept_id, COUNT(*) AS total_employees
    FROM employees
    GROUP BY dept_id
) AS agg ON d.dept_id = agg.dept_id
WHERE agg.total_employees > 2;
```

**Expected Output:**

```
 dept_name   | total_employees
-------------+-----------------
 Engineering |               5
 Sales       |               3
```

**Why This Works:** The derived table `agg` pre-aggregates the employee count per department. The outer query joins this virtual table with `departments` and filters for departments with more than 2 employees.

#### Example 2: MySQL — Derived Table with Column Aliases

```sql
-- Derived table with explicit column aliases
SELECT t.category, t.avg_price
FROM (
    SELECT category, AVG(price) AS avg_price
    FROM products
    GROUP BY category
) AS t
WHERE t.avg_price > 100;
```

**Expected Output:**

```
category  | avg_price
----------+-----------
Electronics |    250.00
```

**Why This Works:** The derived table `t` computes the average price per category. The outer query filters for categories with an average price greater than 100.

### Real-World Cases

- **ETL data aggregation:** Pre-aggregating raw transactional detail rows before joining against dimension tables.
- **Financial reporting:** Calculating multi-tier tax or commission rollups before applying executive filters.
- **Customer cohort analysis:** Computing per-customer first-purchase timestamps in a derived layer to segment cohort retention metrics.
- **Top-N queries:** Ranking rows within a derived table and filtering for the top N in the outer query.

### References

- PostgreSQL: Subquery Expressions — https://www.postgresql.org/docs/current/functions-subquery.html
- PostgreSQL: FROM Clause — https://www.postgresql.org/docs/current/sql-select.html
- MySQL: Derived Tables — https://dev.mysql.com/doc/refman/8.0/en/derived-tables.html
- SQL Server: Derived Tables — https://learn.microsoft.com/en-us/sql/t-sql/queries/from-transact-sql


## Core Concept 10: Table Metadata

### Definitions

**Core Definition:** Table metadata is the set of descriptive information about a table—including row counts, physical storage size, partitioning information, and data distribution statistics—stored in system catalogs and data dictionary views.

**Technical Definition:** Table metadata encompasses the quantitative and structural information about a table's physical and logical characteristics. This includes the number of rows (exact or estimated), the disk space consumed by data and indexes, the partitioning scheme and partition boundaries, and statistical distributions used by the query optimizer. Each RDBMS provides system functions and catalog views for querying this metadata: PostgreSQL uses `pg_class.reltuples`, `pg_relation_size()`, and `pg_total_relation_size()`; SQL Server uses `sys.dm_db_partition_stats`, `sys.partitions`, and `sys.allocation_units`; Oracle uses `USER_TABLES.num_rows`, `USER_SEGMENTS.bytes`, and `USER_TAB_PARTITIONS`; MySQL uses `information_schema.TABLES.TABLE_ROWS` and `information_schema.PARTITIONS`.

**Beginner-Friendly Explanation:** Table metadata is like the information card in a library's card catalog. It tells you how many books are on the shelf (row count), how much space the shelf takes up (storage size), and how the books are organized (partitioning). It does not contain the actual books (data)—just information about them.

### Purposes

- **To monitor database growth** and plan capacity.
- **To identify large tables** that may benefit from partitioning or archiving.
- **To verify statistics freshness** for query optimization.
- **To audit storage consumption** by table, schema, or tablespace.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL — Row Count and Size)

```sql
-- Estimated row count (from pg_class)
SELECT relname, reltuples::bigint AS estimated_rows
FROM pg_class
WHERE relname = 'employees';

-- Total table size (including indexes and TOAST)
SELECT pg_size_pretty(pg_total_relation_size('employees')) AS total_size;

-- Table-only size (excluding indexes)
SELECT pg_size_pretty(pg_table_size('employees')) AS table_size;

-- Index size
SELECT pg_size_pretty(pg_indexes_size('employees')) AS index_size;
```

#### Complete General Syntax (SQL Server — Row Count and Storage)

```sql
-- Accurate row count
SELECT SUM(row_count) AS total_rows
FROM sys.dm_db_partition_stats
WHERE object_id = OBJECT_ID('dbo.Employees')
  AND (index_id = 0 OR index_id = 1);

-- Storage size
SELECT
    SUM(reserved_page_count) * 8.0 / 1024 AS reserved_mb
FROM sys.dm_db_partition_stats
WHERE object_id = OBJECT_ID('dbo.Employees');
```

#### Complete General Syntax (Oracle — Row Count and Size)

```sql
-- Estimated row count (from statistics)
SELECT table_name, num_rows, last_analyzed
FROM user_tables
WHERE table_name = 'EMPLOYEES';

-- Storage size
SELECT segment_name, bytes / 1024 / 1024 AS size_mb
FROM user_segments
WHERE segment_name = 'EMPLOYEES';
```

#### Complete General Syntax (MySQL — Row Count and Size)

```sql
-- Estimated row count
SELECT table_name, table_rows, data_length, index_length
FROM information_schema.TABLES
WHERE table_schema = 'mydb' AND table_name = 'employees';
```

#### Syntax Rules

- **PostgreSQL `reltuples`:** This is an estimate updated by `VACUUM`, `ANALYZE`, and some DDL commands. If the table has never been vacuumed or analyzed, `reltuples` contains -1 indicating that the row count is unknown.
- **PostgreSQL `pg_total_relation_size()`:** Returns the total disk space used by the table, including all associated indexes. `pg_table_size()` returns table-only size; `pg_indexes_size()` returns index-only size.
- **SQL Server `sys.dm_db_partition_stats`:** The `row_count` column is the approximate number of rows in the partition. Using `index_id = 0 OR index_id = 1` selects the heap or clustered index.
- **Oracle `num_rows`:** The `num_rows` column in `USER_TABLES` is a statistical estimate generated by `ANALYZE` or `DBMS_STATS`. It is used by the optimizer to compute query costs.
- **MySQL `TABLE_ROWS`:** For InnoDB tables, the row count is only a rough estimate used in SQL optimization. For accurate counts, use `SELECT COUNT(*)` or `information_schema.INNODB_SYS_TABLESTATS.NUM_ROWS`.

#### Constraints and Limitations

- **Estimates vs. exact counts:** Most metadata row counts are estimates. Use `SELECT COUNT(*)` for exact counts (slower on large tables).
- **Statistics freshness:** Estimated row counts depend on statistics being up-to-date. Run `ANALYZE` (PostgreSQL), `UPDATE STATISTICS` (SQL Server), or `DBMS_STATS.GATHER_TABLE_STATS` (Oracle) to refresh.
- **Size calculations:** Size functions report allocated space, which may include free space within pages.

### Annotated Code Examples

#### Example 1: PostgreSQL — Table Size and Row Count

```sql
-- Get estimated row count and storage sizes
SELECT
    relname AS table_name,
    reltuples::bigint AS estimated_rows,
    pg_size_pretty(pg_table_size(oid)) AS table_size,
    pg_size_pretty(pg_indexes_size(oid)) AS index_size,
    pg_size_pretty(pg_total_relation_size(oid)) AS total_size
FROM pg_class
WHERE relname = 'employees';
```

**Expected Output:**

```
 table_name | estimated_rows | table_size | index_size | total_size
------------+----------------+------------+------------+------------
 employees  |           1500 | 1024 kB    | 512 kB     | 1536 kB
```

**Why This Works:** `pg_class.reltuples` provides the estimated row count (updated by `ANALYZE`). `pg_table_size`, `pg_indexes_size`, and `pg_total_relation_size` return the storage sizes of the table, its indexes, and the total, respectively.

#### Example 2: SQL Server — Row Count and Partition Information

```sql
-- Get accurate row count and partition details
SELECT
    p.partition_number,
    p.rows AS row_count,
    fg.name AS filegroup_name
FROM sys.partitions p
JOIN sys.allocation_units au ON p.partition_id = au.container_id
JOIN sys.filegroups fg ON au.data_space_id = fg.data_space_id
WHERE p.object_id = OBJECT_ID('dbo.Orders')
  AND p.index_id IN (0, 1);
```

**Expected Output:**

```
partition_number | row_count | filegroup_name
-----------------+-----------+----------------
1                |     50000 | PRIMARY
2                |     45000 | PRIMARY
3                |     48000 | PRIMARY
```

**Why This Works:** `sys.partitions` contains one row per partition. `p.rows` gives the approximate row count per partition. Joining with `sys.allocation_units` and `sys.filegroups` identifies the filegroup where each partition is stored.

### Real-World Cases

- **Capacity planning:** Monitoring table growth to predict storage needs.
- **Performance tuning:** Identifying large tables that may benefit from partitioning.
- **Statistics management:** Checking `last_analyzed` to determine if statistics need refreshing.
- **Storage auditing:** Reporting on disk usage by table for cost allocation.

### References

- PostgreSQL: pg_class — https://www.postgresql.org/docs/current/catalog-pg-class.html
- PostgreSQL: Database Object Size Functions — https://www.postgresql.org/docs/current/functions-admin.html
- SQL Server: sys.dm_db_partition_stats — https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-db-partition-stats-transact-sql
- Oracle: USER_TABLES — https://docs.oracle.com/en/database/oracle/oracle-database/21/refrn/USER_TABLES.html
- MySQL: information_schema.TABLES — https://dev.mysql.com/doc/refman/8.0/en/information-schema-tables-table.html


## Core Concept 11: Table Partitioning & Organization

### Definitions

**Core Definition:** Table partitioning is the division of a large table into smaller, more manageable pieces called partitions, while maintaining a single logical table for queries. Table organization refers to the physical storage structure of a table—heap-organized (unordered) or index-organized (B-tree sorted).

**Technical Definition:** Partitioning splits a table into multiple physical segments based on a partitioning key. The three main strategies are: **Range partitioning** (divides data based on continuous value ranges, e.g., dates or numbers), **List partitioning** (uses predefined discrete values, e.g., region codes), and **Hash partitioning** (employs a hash function for even distribution across partitions to balance load). In Oracle, a **heap-organized table** stores data as an unordered collection (heap); data is inserted where it fits. An **index-organized table** (IOT) stores data in a B-tree index structure in a primary key sorted manner. Each leaf block in the index structure stores both the key and nonkey columns. IOTs provide fast primary key access, fast range access, lower storage requirements, and are ideal for OLTP applications.

**Beginner-Friendly Explanation:** Partitioning a table is like organizing a huge filing cabinet into separate drawers by date or category. You still have one filing cabinet (one logical table), but the data is physically stored in separate drawers (partitions), making it faster to find and manage. Index-organized tables are like a dictionary—the data is stored in alphabetical order (by primary key), so looking up a word is very fast.

### Purposes

- **To improve query performance** by enabling partition pruning (scanning only relevant partitions).
- **To simplify data management** by allowing partition-level operations (truncate, drop, archive).
- **To distribute I/O** across multiple storage devices.
- **To optimize primary key access** with index-organized tables.
- **To reduce storage** by avoiding duplicate storage of primary keys (IOTs).

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL — Range Partitioning)

```sql
CREATE TABLE sales (
    sale_id BIGSERIAL,
    sale_date DATE NOT NULL,
    amount NUMERIC(10,2)
) PARTITION BY RANGE (sale_date);

CREATE TABLE sales_2024 PARTITION OF sales
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

CREATE TABLE sales_2025 PARTITION OF sales
    FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');
```

#### Complete General Syntax (MySQL — List Partitioning)

```sql
CREATE TABLE customers (
    customer_id INT NOT NULL,
    region VARCHAR(20)
) PARTITION BY LIST COLUMNS(region) (
    PARTITION p_north VALUES IN ('North', 'Northeast'),
    PARTITION p_south VALUES IN ('South', 'Southeast'),
    PARTITION p_west VALUES IN ('West', 'Northwest')
);
```

#### Complete General Syntax (Oracle — Hash Partitioning)

```sql
CREATE TABLE dept (
    deptno NUMBER,
    deptname VARCHAR2(32)
) PARTITION BY HASH(deptno) PARTITIONS 16;
```

#### Complete General Syntax (Oracle — Index-Organized Table)

```sql
CREATE TABLE admin_docindex (
    token CHAR(20),
    doc_id NUMBER,
    token_frequency NUMBER,
    token_offsets VARCHAR2(2000),
    CONSTRAINT pk_admin_docindex PRIMARY KEY (token, doc_id)
) ORGANIZATION INDEX
TABLESPACE admin_tbs
PCTTHRESHOLD 20
OVERFLOW TABLESPACE admin_tbs2;
```

#### Syntax Rules

- **Range partitioning:** Best for time-series data or retention windows. Partitions are defined by `FOR VALUES FROM ... TO ...` (PostgreSQL) or `VALUES LESS THAN` (MySQL, Oracle).
- **List partitioning:** Best for discrete categorical values (regions, status codes). Partitions are defined by explicit value lists.
- **Hash partitioning:** Best for even distribution when no natural range or list exists. The number of partitions is specified in the DDL.
- **Index-organized tables (Oracle):** Created with `ORGANIZATION INDEX`. The primary key columns are stored in the B-tree index; non-key columns are stored in leaf blocks. The `OVERFLOW` clause stores non-key columns in a separate segment.

#### Constraints and Limitations

- **Partition key:** Must be included in the primary key of the table.
- **Partition pruning:** The query optimizer can skip partitions only if the `WHERE` clause includes the partition key.
- **Index-organized tables (Oracle):** Not suitable for tables with frequent full-table scans or large rows that cause row overflow.
- **Hash partitioning:** Does not support partition pruning for range queries.
- **Version-specific:** PostgreSQL declarative partitioning (10+); MySQL partitioning (5.1+); Oracle partitioning (8.0+); SQL Server partitioning (2005+).

### Annotated Code Examples

#### Example 1: PostgreSQL — Range Partitioning by Date

```sql
-- Create a partitioned table
CREATE TABLE orders (
    order_id BIGSERIAL,
    order_date DATE NOT NULL,
    customer_id BIGINT,
    amount NUMERIC(12,2),
    PRIMARY KEY (order_date, order_id)
) PARTITION BY RANGE (order_date);

-- Create partitions
CREATE TABLE orders_2024 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
CREATE TABLE orders_2025 PARTITION OF orders
    FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');

-- Insert data (automatically routed to the correct partition)
INSERT INTO orders (order_date, customer_id, amount)
VALUES ('2024-06-15', 101, 150.00), ('2025-03-20', 102, 200.00);

-- Query with partition pruning
SELECT * FROM orders WHERE order_date >= '2025-01-01';
```

**Expected Output:**

```
 order_id | order_date | customer_id | amount
----------+------------+-------------+--------
        2 | 2025-03-20 |         102 | 200.00
```

**Why This Works:** The `PARTITION BY RANGE (order_date)` clause partitions the table by year. Rows are automatically routed to the correct partition based on `order_date`. The query with `WHERE order_date >= '2025-01-01'` only scans the `orders_2025` partition (partition pruning).

#### Example 2: Oracle — Index-Organized Table

```sql
-- Create an index-organized table
CREATE TABLE admin_docindex (
    token CHAR(20),
    doc_id NUMBER,
    token_frequency NUMBER,
    token_offsets VARCHAR2(2000),
    CONSTRAINT pk_admin_docindex PRIMARY KEY (token, doc_id)
) ORGANIZATION INDEX
TABLESPACE admin_tbs
PCTTHRESHOLD 20
OVERFLOW TABLESPACE admin_tbs2;
```

**Expected Output:**

```
Table created.
```

**Why This Works:** The `ORGANIZATION INDEX` clause creates an index-organized table. The primary key `(token, doc_id)` determines the B-tree structure. The `PCTTHRESHOLD 20` and `OVERFLOW` clauses store rows exceeding 20% of the block size in a separate overflow segment.

### Real-World Cases

- **Time-series data:** Range partitioning by date for logs, events, and transactions.
- **Multi-region applications:** List partitioning by region or country code.
- **Even distribution:** Hash partitioning for high-volume OLTP tables.
- **Reference data:** Index-organized tables for lookup tables with primary key access (e.g., product catalogs, employee directories).

### References

- PostgreSQL: Table Partitioning — https://www.postgresql.org/docs/current/ddl-partitioning.html
- MySQL: Partitioning — https://dev.mysql.com/doc/refman/8.0/en/partitioning.html
- Oracle: Partitioned Tables and Indexes — https://docs.oracle.com/en/database/oracle/oracle-database/21/vldbg/partition-concepts.html
- Oracle: Index-Organized Tables — https://docs.oracle.com/en/database/oracle/oracle-database/21/admin/managing-tables.html
- SQL Server: Partitioned Tables and Indexes — https://learn.microsoft.com/en-us/sql/relational-databases/partitions/partitioned-tables-and-indexes


## Summary Table: Table Operations Across DBMS

| Operation | PostgreSQL | MySQL | SQL Server | Oracle |
|-----------|-----------|-------|------------|--------|
| **CREATE TABLE** | `CREATE TABLE ...` | `CREATE TABLE ...` | `CREATE TABLE ...` | `CREATE TABLE ...` |
| **CTAS** | `CREATE TABLE AS` | `CREATE TABLE AS` | `SELECT ... INTO` | `CREATE TABLE AS` |
| **View Definition** | `\d`, `information_schema` | `SHOW CREATE TABLE` | `sp_help`, `sys.columns` | `DESCRIBE`, `USER_TAB_COLUMNS` |
| **List Tables** | `\dt`, `pg_tables` | `SHOW TABLES` | `sys.tables` | `USER_TABLES` |
| **Rename Table** | `ALTER TABLE ... RENAME TO` | `RENAME TABLE`, `ALTER TABLE ... RENAME` | `sp_rename` | `ALTER TABLE ... RENAME TO` |
| **Truncate** | `TRUNCATE TABLE` | `TRUNCATE TABLE` | `TRUNCATE TABLE` | `TRUNCATE TABLE` |
| **Drop Table** | `DROP TABLE [CASCADE]` | `DROP TABLE` | `DROP TABLE` | `DROP TABLE [PURGE]` |
| **Temporary Table** | `CREATE TEMP TABLE` | `CREATE TEMPORARY TABLE` | `#local`, `##global` | `CREATE GLOBAL TEMPORARY TABLE` |
| **Shallow Copy** | `CREATE TABLE ... LIKE` | `CREATE TABLE ... LIKE` | Script DDL | `CREATE TABLE ... AS WHERE 1=0` |
| **Deep Copy** | `CREATE TABLE AS` | `CREATE TABLE AS` | `SELECT ... INTO` | `CREATE TABLE AS` |
| **Partitioning** | Range, List, Hash | Range, List, Hash, Key | Range | Range, List, Hash, Composite |
| **IOT** | N/A (cluster) | N/A | N/A | `ORGANIZATION INDEX` |


## Final Notes on Deprecated and Unsafe Features

- **PostgreSQL `SELECT INTO`:** Deprecated in favor of `CREATE TABLE AS` because `SELECT INTO` is interpreted differently in ECPG and PL/pgSQL. Use `CREATE TABLE AS` instead.
- **SQL Server `SELECT INTO`:** Does not copy indexes, constraints, or triggers. For a complete deep copy, script the table DDL or use SMO (SQL Server Management Objects).
- **MySQL `CREATE TABLE ... AS SELECT`:** Does not preserve indexes. For a shallow copy that preserves indexes and constraints, use `CREATE TABLE ... LIKE`.
- **TRUNCATE TABLE restrictions:** Cannot be used on tables referenced by foreign keys, tables participating in indexed views, or tables published by replication. Use `DELETE` instead.
- **Oracle recycle bin:** Dropped tables remain in the recycle bin until purged, consuming storage space. Use `PURGE` to release space immediately.
- **Row count estimates:** `pg_class.reltuples`, `USER_TABLES.num_rows`, and `information_schema.TABLES.TABLE_ROWS` are estimates, not exact counts. Use `SELECT COUNT(*)` for exact counts.
- **CTE vs. temp table materialization:** In SQL Server, CTEs are not materialized and can lead to repeated re-execution of the underlying syntax. PostgreSQL 12+ can inline CTEs when they are not recursive and are referenced only once. PostgreSQL < 12 and MySQL always materialize CTEs.
- **Derived table alias:** Every derived table must have its own alias. Omitting the alias causes a syntax error in most databases.