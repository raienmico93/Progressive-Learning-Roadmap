# Creating Database Objects: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Creating database objects is the process of defining and instantiating the structural components of a relational database—tables, schemas, views, indexes, sequences, constraints, triggers, stored procedures, and functions—using Data Definition Language (DDL) statements.

**Technical Definition:** Database object creation encompasses the DDL statements that define the schema of a relational database system. Each object type serves a distinct architectural purpose: tables store data in rows and columns; schemas provide logical namespaces; views define virtual tables derived from queries; indexes accelerate data retrieval; sequences generate monotonic numeric values; constraints enforce data integrity rules; triggers execute procedural code in response to data modification events; and stored procedures and functions encapsulate reusable procedural logic. These objects form the foundation of the relational model and are governed by the SQL standard (SQL-92 onward) with significant vendor-specific extensions.

**Beginner-Friendly Explanation:** Creating database objects is like building the structure of a library. Tables are the bookshelves that hold the books (data). Schemas are the different sections (Fiction, Non-Fiction, Reference). Views are like windows that let you see specific parts of the collection. Indexes are the card catalog that helps you find things quickly. Sequences are like the automatic numbering machine for new books. Constraints are the rules about what can go on each shelf. Triggers are like alarms that go off when something changes. Stored procedures and functions are like the library staff who perform specific tasks.

### Key Characteristics

- **Declarative DDL:** Objects are defined using `CREATE` statements that describe what the object should look like, not how to build it.
- **Vendor-specific syntax:** While the SQL standard defines basic `CREATE` syntax, each RDBMS (PostgreSQL, MySQL, SQL Server, Oracle) implements significant extensions.
- **Metadata-driven:** The database stores object definitions in system catalogs (e.g., `information_schema`, `pg_catalog`, `sys.objects`).
- **Dependency-aware:** Objects can depend on one another (e.g., a view depends on a table); dependencies affect drop and alter operations.
- **Privilege-controlled:** Creating objects requires specific permissions (e.g., `CREATE TABLE`, `CREATE VIEW`, `CREATE PROCEDURE`).
- **Persistence:** Objects persist until explicitly dropped or the database is destroyed (with the exception of temporary objects).

### Prerequisites

- Understanding of the relational model (tables, rows, columns, keys).
- Familiarity with SQL data types and expressions.
- Knowledge of database connection and authentication mechanisms.
- Awareness of the target database's specific syntax and limitations.
- Basic understanding of transactions and locking.

### Related Programming Areas

- Database schema design and normalization.
- Application development and ORM mapping.
- Data warehousing and ETL pipelines.
- Database administration and performance tuning.
- DevOps and schema migration automation.

### Core Concepts / Features

1. **Tables** (permanent, temporary, CTEs vs. materialized views)
2. **Schemas** (creation, authorization, ownership, search paths)
3. **Views** (standard, updatable, materialized)
4. **Indexes** (B-Tree, Hash, GIST/GIN, clustered vs. non-clustered, partial/filtered)
5. **Sequences** (creation, cycle options, IDENTITY integration)
6. **Constraints** (inline vs. out-of-line)
7. **Triggers** (DDL vs. DML, BEFORE/AFTER/INSTEAD OF, row vs. statement)
8. **Stored Procedures** (parameters, execution contexts, transaction control)
9. **Functions** (scalar, table-valued, deterministic vs. non-deterministic)


## Core Concept 1: Tables

### Definitions

**Core Definition:** A table is the fundamental database object that stores data in a structured format of rows (records) and columns (fields).

**Technical Definition:** A table in a relational database is a two-dimensional structure consisting of a fixed number of columns, each with a declared data type, and a variable number of rows containing the actual data values. Tables are created with the `CREATE TABLE` statement, which specifies the table name, column names, data types, and optional constraints. Tables can be permanent (persisting until dropped), temporary (existing only for a session or transaction), or unlogged (PostgreSQL-specific, not written to the write-ahead log).

**Beginner-Friendly Explanation:** A table is like a spreadsheet. It has columns (like "Name", "Age", "Email") and rows (each row is one record, like a person's information). You create the table structure first, then fill it with data.

### Purposes

- To store persistent data in a structured, queryable format.
- To provide a target for data manipulation operations (INSERT, UPDATE, DELETE, SELECT).
- To enforce data integrity through column data types and constraints.
- To serve as the foundation for views, indexes, and other dependent objects.

### Syntax Rules and Structure

#### Complete General Syntax (SQL Standard / PostgreSQL)

```sql
CREATE [ [ GLOBAL | LOCAL ] { TEMPORARY | TEMP } | UNLOGGED ] TABLE [ IF NOT EXISTS ] table_name (
    column_name data_type [ COLLATE collation ] [ column_constraint [ ... ] ]
    [, ...]
    [, table_constraint ]
) [ INHERITS ( parent_table ) ] [ WITH ( storage_parameter ) ];
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `CREATE TABLE` | Keyword beginning the table creation statement. |
| `table_name` | The name of the table (optionally schema-qualified). |
| `column_name data_type` | Column definition with name and data type. |
| `column_constraint` | Optional constraints on the column (NOT NULL, DEFAULT, CHECK, etc.). |
| `table_constraint` | Optional constraints on the table (PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK). |
| `TEMPORARY` / `TEMP` | Creates a session-scoped temporary table. |
| `UNLOGGED` | Creates a table not written to the WAL (PostgreSQL-specific). |
| `IF NOT EXISTS` | Suppresses error if the table already exists. |

#### Complete General Syntax (MySQL)

```sql
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name (
    col_name column_definition
    [, ...]
    [, {INDEX | KEY} [index_name] [index_type] (key_part,...)]
    [, [CONSTRAINT [symbol]] PRIMARY KEY [index_type] (key_part,...)]
    [, [CONSTRAINT [symbol]] UNIQUE [INDEX | KEY] [index_name] [index_type] (key_part,...)]
    [, [CONSTRAINT [symbol]] FOREIGN KEY [index_name] (col_name,...) reference_definition]
    [, [CONSTRAINT [symbol]] CHECK (expr) [[NOT] ENFORCED]]
) [table_options] [partition_options];
```

#### Syntax Rules

- **Permanent tables:** Persist until explicitly dropped with `DROP TABLE`.
- **Temporary tables:** Exist only for the duration of a session (or transaction in some databases); automatically dropped when the session ends. `TEMPORARY` tables are visible only to the session that created them.
- **Unlogged tables (PostgreSQL):** Not written to the WAL; faster writes but not crash-safe.
- **Column order:** The order of columns in the definition determines their default display order; new columns are appended at the end (except MySQL, which supports `FIRST`/`AFTER`).
- **Table inheritance (PostgreSQL):** `INHERITS` allows a table to inherit columns from a parent table.

#### Constraints and Limitations

- **Temporary tables:** Cannot be partitioned, cannot have foreign keys referencing them, and are not visible across sessions.
- **Unlogged tables:** Not replicated and not recoverable after a crash.
- **Column limits:** PostgreSQL allows up to 1,600 columns per table; SQL Server allows 1,024 (non-sparse) or 30,000 (sparse); MySQL allows up to 4,096 columns.
- **Tablespace:** In PostgreSQL and Oracle, tables can be placed in specific tablespaces for storage management.
- **Version-specific:** `UNLOGGED` is PostgreSQL-specific; `TEMPORARY` behavior varies across databases.

### Annotated Code Examples

#### Example 1: PostgreSQL — Permanent and Temporary Tables

```sql
-- Create a permanent table
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    hire_date DATE DEFAULT CURRENT_DATE,
    salary NUMERIC(10,2) CHECK (salary > 0)
);

-- Create a temporary table (exists only for this session)
CREATE TEMPORARY TABLE temp_employee_data (
    employee_id INTEGER,
    bonus_amount NUMERIC(10,2)
);

-- Insert into permanent table
INSERT INTO employees (first_name, last_name, salary) VALUES ('Alice', 'Smith', 75000);

-- Query
SELECT * FROM employees;
```

**Expected Output:**

```
 employee_id | first_name | last_name | hire_date  |  salary
-------------+------------+-----------+------------+---------
           1 | Alice      | Smith     | 2026-09-26 | 75000.00
```

**Why This Works:** The `employees` table is permanent and will persist across sessions. The `temp_employee_data` table exists only in the current session and is automatically dropped when the session ends. The `SERIAL` type auto-generates the `employee_id`. The `CHECK` constraint enforces that `salary` is positive.

#### Example 2: MySQL — Temporary Table with LIKE

```sql
-- Create a temporary table with the same structure as an existing table
CREATE TEMPORARY TABLE temp_orders LIKE orders;

-- Insert filtered data into the temporary table
INSERT INTO temp_orders
SELECT * FROM orders WHERE order_date >= '2026-01-01';

-- Query the temporary table
SELECT COUNT(*) AS recent_orders FROM temp_orders;
```

**Expected Output:**

```
recent_orders
-------------
        1500
```

**Why This Works:** The `LIKE` clause copies the structure (columns, data types, indexes) of the `orders` table without copying data. The temporary table is session-scoped and automatically dropped when the connection closes.

### Real-World Cases

- **OLTP systems:** Permanent tables for customers, orders, products, and transactions.
- **ETL staging:** Temporary tables for holding intermediate data during transformation.
- **Session state:** Temporary tables for storing user-specific session data (shopping carts, preferences).
- **Data warehousing:** Unlogged tables for staging areas where crash safety is not critical.

### References

- PostgreSQL Documentation: CREATE TABLE — https://www.postgresql.org/docs/current/sql-createtable.html
- MySQL Reference Manual: CREATE TABLE — https://dev.mysql.com/doc/refman/8.0/en/create-table.html
- SQL Server: CREATE TABLE — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql
- Oracle Database SQL Language Reference: CREATE TABLE — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CREATE-TABLE.html


## Core Concept 2: Schemas

### Definitions

**Core Definition:** A schema is a named collection of database objects—tables, views, indexes, sequences, functions, and procedures—that provides a logical namespace for organizing and securing database objects.

**Technical Definition:** A schema is a container for database objects that allows objects with the same name to coexist in different namespaces. Schemas are created with the `CREATE SCHEMA` statement, which can also include the definitions of objects within the schema. The `AUTHORIZATION` clause specifies the owner of the schema. The search path is an ordered list of schemas that the database engine searches when resolving unqualified object names.

**Beginner-Friendly Explanation:** A schema is like a folder on your computer. You can have two files named "report.txt" in different folders, and they do not conflict. Schemas let you organize database objects into logical groups, so different applications or users can have their own objects without name conflicts.

### Purposes

- To organize database objects into logical groups (e.g., by application, department, or purpose).
- To allow multiple objects with the same name to coexist in different namespaces.
- To provide a unit of privilege management (granting access to a schema grants access to its objects).
- To control which objects are visible to which users through the search path.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
CREATE SCHEMA schema_name [ AUTHORIZATION role_specification ]
    [ schema_element [ ... ] ];
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `schema_name` | The name of the schema (optional if `AUTHORIZATION` is specified). |
| `AUTHORIZATION role_specification` | The owner of the schema. If omitted, the current user is the owner. |
| `schema_element` | Optional `CREATE TABLE`, `CREATE VIEW`, `GRANT`, etc. statements included in the schema. |

#### Complete General Syntax (SQL Server)

```sql
CREATE SCHEMA schema_name [ AUTHORIZATION owner_name ]
    [ schema_element [ ... ] ];
```

#### Syntax Rules

- **Schema ownership:** The owner of the schema is the user specified in `AUTHORIZATION`. PostgreSQL allows schemas to contain objects owned by users other than the schema owner.
- **Search path (PostgreSQL):** `search_path` is a comma-separated list of schema names. The first schema in the list is the "current schema" and is used for creating new objects with unqualified names. Default is `"$user", public`.
- **Public schema:** All databases include a `public` schema by default. Objects created without a schema qualifier are placed in the current schema (typically `public`).
- **System schemas:** Schemas starting with `pg_` are reserved for system use and cannot be created by users.

#### Constraints and Limitations

- **Schema naming:** Schema names cannot start with `pg_` (PostgreSQL).
- **Authorization:** The `AUTHORIZATION` clause must specify a role that exists; the role must have `CREATE` privilege in the database.
- **Search path security:** Adding a schema to `search_path` trusts all users with `CREATE` privilege in that schema, which can be a security risk.
- **Cross-database:** Schemas are database-scoped; they cannot span multiple databases.

### Annotated Code Examples

#### Example 1: PostgreSQL — Creating a Schema and Setting Search Path

```sql
-- Create a schema owned by the current user
CREATE SCHEMA sales;

-- Create a table in the new schema
CREATE TABLE sales.orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    order_total NUMERIC(10,2)
);

-- Add the schema to the search path
SET search_path TO sales, public;

-- Now we can reference the table without the schema qualifier
SELECT * FROM orders;
```

**Expected Output:**

```
 order_id | customer_id | order_total
----------+-------------+------------
(0 rows)
```

**Why This Works:** The `CREATE SCHEMA sales` statement creates a new schema. The `sales.orders` table is created within that schema. `SET search_path TO sales, public` makes `sales` the current schema, so unqualified references to `orders` resolve to `sales.orders`.

#### Example 2: SQL Server — Creating a Schema with Authorization

```sql
-- Create a schema owned by a specific user
CREATE SCHEMA HumanResources AUTHORIZATION dbo;
GO

-- Create a table in the schema
CREATE TABLE HumanResources.Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50)
);
GO
```

**Expected Output:**

```
Commands completed successfully.
```

**Why This Works:** The schema `HumanResources` is created and owned by the `dbo` user. The `HumanResources.Employees` table is then created within that schema. SQL Server requires the `GO` batch separator after `CREATE SCHEMA`.

### Real-World Cases

- **Multi-tenant applications:** Each tenant gets its own schema, isolating their data from other tenants.
- **Departmental organization:** `sales`, `hr`, `finance`, `inventory` schemas for different business functions.
- **Version management:** `v1`, `v2` schemas for different API versions.
- **Security isolation:** Granting access to a schema rather than individual tables.

### References

- PostgreSQL Documentation: CREATE SCHEMA — https://www.postgresql.org/docs/current/sql-createschema.html
- PostgreSQL Documentation: Schemas — https://www.postgresql.org/docs/current/ddl-schemas.html
- SQL Server: CREATE SCHEMA — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-schema-transact-sql
- Oracle Database: CREATE SCHEMA — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CREATE-SCHEMA.html


## Core Concept 3: Views

### Definitions

**Core Definition:** A view is a named query stored in the database that acts as a virtual table, presenting the results of a `SELECT` statement as if it were a physical table.

**Technical Definition:** A view is a stored query definition that does not physically store data (unlike a materialized view). When a view is queried, the database engine executes the underlying `SELECT` statement and returns the results. Views can be updatable (if the underlying query meets certain criteria) and can include a `WITH CHECK OPTION` clause that prevents inserts or updates through the view from creating rows that would not be visible through the view. Materialized views physically store the query results and must be refreshed to reflect changes in the underlying tables.

**Beginner-Friendly Explanation:** A view is like a saved search. Instead of writing the same complex query over and over, you save it as a view and query the view like a table. It does not store data itself—it just runs the query each time you use it. A materialized view is like a snapshot that does store data, but you have to refresh it to see changes.

### Purposes

- To simplify complex queries by encapsulating them in a named object.
- To provide a security mechanism by granting access to specific columns or rows without granting access to base tables.
- To present a consistent interface to data even as the underlying schema changes.
- To enable updatable views that allow data modification through the view.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
CREATE [ OR REPLACE ] [ TEMP | TEMPORARY ] VIEW view_name [ ( column_name [, ...] ) ]
    [ WITH ( view_option_name [= view_option_value] [, ... ] ) ]
    AS query
    [ WITH [ CASCADED | LOCAL ] CHECK OPTION ];
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `OR REPLACE` | Replaces an existing view if it exists. |
| `view_name` | The name of the view. |
| `( column_name, ... )` | Optional list of column names for the view. |
| `AS query` | The `SELECT` statement defining the view. |
| `WITH CHECK OPTION` | Prevents inserts/updates that would not be visible through the view. `CASCADED` (default) checks all underlying views; `LOCAL` checks only the current view. |

#### Complete General Syntax (MySQL)

```sql
CREATE [OR REPLACE]
    [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
    [DEFINER = user]
    [SQL SECURITY { DEFINER | INVOKER }]
    VIEW view_name [(column_list)]
    AS select_statement
    [WITH [CASCADED | LOCAL] CHECK OPTION];
```

#### Complete General Syntax (SQL Server)

```sql
CREATE [ OR ALTER ] VIEW [ schema_name . ] view_name [ (column [ ,...n ] ) ]
    [ WITH <view_attribute> [ ,...n ] ]
    AS select_statement
    [ WITH CHECK OPTION ];
```

#### Syntax Rules

- **Updatable views:** A view is updatable if it references exactly one table (or another updatable view) and does not contain `DISTINCT`, `GROUP BY`, `HAVING`, `UNION`, or aggregate functions.
- **WITH CHECK OPTION:** Ensures that any row inserted or updated through the view satisfies the view's `WHERE` clause, preventing the creation of rows invisible to the view.
- **CASCADED vs. LOCAL:** `CASCADED` (default) applies the check to the view and all underlying views; `LOCAL` applies only to the view itself.
- **Materialized views (PostgreSQL):** Created with `CREATE MATERIALIZED VIEW`; physically stores data; must be refreshed with `REFRESH MATERIALIZED VIEW`.
- **Materialized views (MySQL):** Only supported on MySQL HeatWave.

#### Constraints and Limitations

- **Materialized view refresh:** PostgreSQL materialized views are not automatically refreshed; use `REFRESH MATERIALIZED VIEW [CONCURRENTLY]`.
- **CONCURRENTLY:** For materialized views, `REFRESH MATERIALIZED VIEW CONCURRENTLY` requires a unique index on the view and does not block reads.
- **SQL Server indexed views:** SQL Server does not have materialized views but supports indexed views (views with a unique clustered index).
- **MySQL:** Standard views are not materialized; `ALGORITHM=MERGE` merges the view query into the outer query, while `ALGORITHM=TEMPTABLE` creates a temporary table.

### Annotated Code Examples

#### Example 1: PostgreSQL — Creating and Using a View

```sql
-- Create base tables
CREATE TABLE departments (
    dept_id SERIAL PRIMARY KEY,
    dept_name TEXT NOT NULL
);

CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    emp_name TEXT NOT NULL,
    dept_id INT REFERENCES departments(dept_id),
    salary NUMERIC(10,2)
);

-- Insert sample data
INSERT INTO departments (dept_name) VALUES ('Engineering'), ('Sales');
INSERT INTO employees (emp_name, dept_id, salary) VALUES
('Alice', 1, 90000), ('Bob', 1, 85000), ('Carol', 2, 75000);

-- Create a view
CREATE VIEW engineering_employees AS
SELECT e.emp_name, e.salary, d.dept_name
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id
WHERE d.dept_name = 'Engineering';

-- Query the view
SELECT * FROM engineering_employees;
```

**Expected Output:**

```
 emp_name | salary  |  dept_name
----------+---------+-------------
 Alice    | 90000.00 | Engineering
 Bob      | 85000.00 | Engineering
```

**Why This Works:** The view `engineering_employees` encapsulates a join between `employees` and `departments`, filtering for the Engineering department. Users can query the view without knowing the underlying join logic.

#### Example 2: PostgreSQL — Updatable View with WITH CHECK OPTION

```sql
-- Create an updatable view with CHECK OPTION
CREATE VIEW high_earners AS
SELECT emp_id, emp_name, salary
FROM employees
WHERE salary > 80000
WITH CHECK OPTION;

-- This insert succeeds (salary > 80000)
INSERT INTO high_earners (emp_id, emp_name, salary) VALUES (100, 'Dave', 95000);

-- This insert fails (salary <= 80000, violates CHECK OPTION)
INSERT INTO high_earners (emp_id, emp_name, salary) VALUES (101, 'Eve', 70000);
```

**Expected Output (for the failing insert):**

```
ERROR:  new row violates check option for view "high_earners"
DETAIL:  Failing row contains (101, Eve, 70000).
```

**Why This Works:** The `WITH CHECK OPTION` prevents inserts or updates that would create rows not visible through the view. Since the view only shows employees with salary > 80000, inserting an employee with salary 70000 is rejected.

#### Example 3: PostgreSQL — Materialized View with Refresh

```sql
-- Create a materialized view
CREATE MATERIALIZED VIEW dept_salary_stats AS
SELECT d.dept_name, COUNT(*) AS emp_count, AVG(e.salary) AS avg_salary
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id
GROUP BY d.dept_name;

-- Query the materialized view
SELECT * FROM dept_salary_stats;
```

**Expected Output:**

```
 dept_name   | emp_count |      avg_salary
-------------+-----------+---------------------
 Engineering |         2 | 87500.000000000000
 Sales       |         1 | 75000.000000000000
```

**Why This Works:** The materialized view physically stores the aggregated results. Unlike a regular view, querying it does not re-execute the underlying query. To update the data after changes to the base tables, run `REFRESH MATERIALIZED VIEW dept_salary_stats`.

### Real-World Cases

- **Security:** A view that exposes only non-sensitive columns (e.g., `employee_public` view without `salary`).
- **Simplification:** A view that joins five tables into a single, easy-to-query object.
- **Backward compatibility:** A view that presents an old schema even after the underlying tables have changed.
- **Reporting:** Materialized views for pre-aggregated dashboard metrics.

### References

- PostgreSQL Documentation: CREATE VIEW — https://www.postgresql.org/docs/current/sql-createview.html
- PostgreSQL Documentation: CREATE MATERIALIZED VIEW — https://www.postgresql.org/docs/current/sql-creatematerializedview.html
- MySQL Reference Manual: CREATE VIEW — https://dev.mysql.com/doc/refman/8.0/en/create-view.html
- SQL Server: CREATE VIEW — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-view-transact-sql
- Oracle Database: CREATE VIEW — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CREATE-VIEW.html


## Core Concept 4: Indexes

### Definitions

**Core Definition:** An index is a database object that accelerates data retrieval operations by providing a fast lookup structure for one or more columns of a table.

**Technical Definition:** An index is a data structure (typically a B-tree, hash table, or specialized structure) that maps column values to the physical locations of rows, allowing the database engine to locate rows without scanning the entire table. PostgreSQL provides multiple index methods: B-tree (default, for equality and range queries), Hash (for equality only), GiST (for geometric and full-text data), SP-GiST (for partitioned search trees), GIN (for arrays, JSONB, and full-text search), and BRIN (for large, naturally ordered tables). SQL Server distinguishes between clustered indexes (which determine the physical order of rows) and non-clustered indexes (separate structures with pointers to rows). Partial indexes (PostgreSQL) and filtered indexes (SQL Server) index only a subset of rows.

**Beginner-Friendly Explanation:** An index is like the index at the back of a textbook. Instead of reading every page to find a topic, you look it up in the index, which tells you exactly which page to go to. Database indexes work the same way—they help the database find rows quickly without scanning the whole table.

### Purposes

- To dramatically speed up `SELECT` queries with `WHERE`, `JOIN`, and `ORDER BY` clauses.
- To enforce uniqueness constraints (unique indexes).
- To support specialized queries (full-text search, spatial queries, JSON queries).
- To reduce I/O and CPU usage for data retrieval.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
CREATE [ UNIQUE ] INDEX [ CONCURRENTLY ] [ [ IF NOT EXISTS ] name ]
    ON table_name [ USING method ]
    ( { column_name | ( expression ) } [ COLLATE collation ] [ opclass ]
      [ ASC | DESC ] [ NULLS { FIRST | LAST } ] [, ...] )
    [ WITH ( storage_parameter [= value] [, ... ] ) ]
    [ TABLESPACE tablespace_name ]
    [ WHERE predicate ];
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `UNIQUE` | Creates a unique index. |
| `CONCURRENTLY` | Builds the index without locking out writes. |
| `USING method` | Index method: `btree`, `hash`, `gist`, `spgist`, `gin`, `brin`. |
| `( expression )` | Index on an expression (e.g., `upper(col)`). |
| `WHERE predicate` | Creates a partial index containing only rows matching the predicate. |

#### Complete General Syntax (SQL Server)

```sql
CREATE [ UNIQUE ] [ CLUSTERED | NONCLUSTERED ] INDEX index_name
    ON <object> ( column [ ASC | DESC ] [ ,...n ] )
    [ INCLUDE ( column_name [ ,...n ] ) ]
    [ WHERE <filter_predicate> ]
    [ WITH ( <index_option> [ ,...n ] ) ]
    [ ON { partition_scheme_name ( column_name ) | filegroup_name | "default" } ];
```

#### Syntax Rules

- **B-tree (default):** Best for equality and range queries (`=`, `<`, `>`, `BETWEEN`, `IN`). Only B-tree, GiST, and GIN support multicolumn indexes in PostgreSQL.
- **Hash:** Best for equality comparisons only. Not crash-safe before PostgreSQL 10.
- **GIN:** Best for arrays, JSONB, and full-text search.
- **GiST:** Best for geometric data and full-text search with ranking.
- **Partial/filtered indexes:** Index only rows matching a `WHERE` clause; smaller and faster than full indexes.
- **Clustered index (SQL Server):** Determines the physical order of rows in the table; only one per table.
- **Non-clustered index (SQL Server):** Separate structure with pointers to rows; can include non-key columns with `INCLUDE`.
- **CONCURRENTLY (PostgreSQL):** Builds the index without blocking writes, but takes longer and cannot be used in a transaction.

#### Constraints and Limitations

- **Index overhead:** Indexes consume storage space and slow down `INSERT`, `UPDATE`, and `DELETE` operations.
- **Index selection:** The query optimizer decides whether to use an index; inappropriate indexes can degrade performance.
- **Expression indexes:** Functions used in index expressions must be `IMMUTABLE` (their results must not change for the same inputs).
- **SQL Server filtered indexes:** Cannot be created on computed columns, UDT columns, spatial columns, or hierarchy ID columns; cannot use `NULL` comparisons in the filter predicate.
- **Version-specific:** `CONCURRENTLY` is PostgreSQL-specific; `INCLUDE` is SQL Server-specific; Hash indexes were not crash-safe before PostgreSQL 10.

### Annotated Code Examples

#### Example 1: PostgreSQL — B-tree and Partial Indexes

```sql
-- Create a table
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    status TEXT DEFAULT 'pending',
    total NUMERIC(10,2)
);

-- Create a B-tree index on customer_id
CREATE INDEX idx_orders_customer ON orders (customer_id);

-- Create a partial index for pending orders only
CREATE INDEX idx_orders_pending ON orders (order_date)
WHERE status = 'pending';

-- Create an expression index on upper(customer_name) — but no customer_name column here,
-- so let's use a computed index on order_date year
CREATE INDEX idx_orders_year ON orders (EXTRACT(YEAR FROM order_date));
```

**Expected Output:**

```
CREATE INDEX
```

**Why This Works:** The B-tree index on `customer_id` speeds up joins and lookups by customer. The partial index on `order_date WHERE status = 'pending'` is smaller and faster because it only includes pending orders. The expression index on `EXTRACT(YEAR FROM order_date)` allows fast queries by year.

#### Example 2: SQL Server — Clustered and Non-Clustered Indexes

```sql
-- Create a clustered index (determines physical row order)
CREATE CLUSTERED INDEX IX_Orders_OrderID ON dbo.Orders (OrderID);

-- Create a non-clustered index with included columns
CREATE NONCLUSTERED INDEX IX_Orders_CustomerDate
ON dbo.Orders (CustomerID, OrderDate)
INCLUDE (Total, Status);

-- Create a filtered index for active orders only
CREATE NONCLUSTERED INDEX IX_Orders_Active
ON dbo.Orders (OrderDate)
WHERE Status = 'Active';
```

**Expected Output:**

```
Commands completed successfully.
```

**Why This Works:** The clustered index physically orders rows by `OrderID`. The non-clustered index on `(CustomerID, OrderDate)` includes `Total` and `Status` as non-key columns, allowing the query to be satisfied entirely from the index (a covering index). The filtered index only indexes active orders, making it smaller and faster.

### Real-World Cases

- **OLTP systems:** B-tree indexes on primary keys and foreign keys for fast lookups.
- **Full-text search:** GIN indexes on `tsvector` columns in PostgreSQL.
- **JSON queries:** GIN indexes on `jsonb` columns for containment and existence queries.
- **Data warehousing:** BRIN indexes on large, naturally ordered tables (e.g., time-series data).
- **Partial indexes:** Indexing only active or recent records to reduce index size.

### References

- PostgreSQL Documentation: CREATE INDEX — https://www.postgresql.org/docs/current/sql-createindex.html
- PostgreSQL Documentation: Index Types — https://www.postgresql.org/docs/current/indexes-types.html
- MySQL Reference Manual: CREATE INDEX — https://dev.mysql.com/doc/refman/8.0/en/create-index.html
- SQL Server: CREATE INDEX — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-index-transact-sql
- Oracle Database: CREATE INDEX — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CREATE-INDEX.html


## Core Concept 5: Sequences

### Definitions

**Core Definition:** A sequence is a database object that generates a sequence of unique numeric values according to a specified increment, typically used to generate primary key values.

**Technical Definition:** A sequence is an independent database object that generates monotonically increasing (or decreasing) numeric values. Unlike `IDENTITY` columns (which are table-scoped), sequences are schema-scoped and can be shared across multiple tables. PostgreSQL sequences are manipulated with `nextval()`, `currval()`, and `setval()`. SQL Server introduced sequences in SQL Server 2012. MySQL uses the `AUTO_INCREMENT` attribute (not a separate object), while Oracle has native `CREATE SEQUENCE` support.

**Beginner-Friendly Explanation:** A sequence is like a ticket dispenser at a deli counter. Each time someone takes a ticket, the number increments by one. You can use the ticket number as a unique ID for your order.

### Purposes

- To generate unique numeric identifiers for primary key columns.
- To provide a shared counter that can be used across multiple tables.
- To control the start value, increment, and cycling behavior of generated numbers.
- To support distributed systems where IDs must be unique across multiple nodes.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
CREATE [ TEMPORARY | TEMP ] SEQUENCE [ IF NOT EXISTS ] name
    [ AS data_type ]
    [ INCREMENT [ BY ] increment ]
    [ MINVALUE minvalue | NO MINVALUE ]
    [ MAXVALUE maxvalue | NO MAXVALUE ]
    [ START [ WITH ] start ]
    [ CACHE cache ]
    [ [ NO ] CYCLE ]
    [ OWNED BY { table_name.column_name | NONE } ];
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `INCREMENT BY` | Value added to the current sequence value (default 1). |
| `MINVALUE` / `MAXVALUE` | Minimum and maximum values the sequence can generate. |
| `START WITH` | Starting value (default: `minvalue` for ascending, `maxvalue` for descending). |
| `CACHE` | Number of values to preallocate for performance. |
| `CYCLE` | Restart from `minvalue` after reaching `maxvalue`; `NO CYCLE` (default) raises an error. |
| `OWNED BY` | Associates the sequence with a table column; dropped when the column is dropped. |

#### Complete General Syntax (SQL Server)

```sql
CREATE SEQUENCE [schema_name.] sequence_name
    [ AS [ built_in_integer_type | user-defined_integer_type ] ]
    [ START WITH <constant> ]
    [ INCREMENT BY <constant> ]
    [ { MINVALUE [ <constant> ] } | { NO MINVALUE } ]
    [ { MAXVALUE [ <constant> ] } | { NO MAXVALUE } ]
    [ CYCLE | { NO CYCLE } ]
    [ { CACHE [ <constant> ] } | { NO CACHE } ];
```

#### Syntax Rules

- **PostgreSQL:** `nextval()` advances the sequence and returns the next value; `currval()` returns the current value for the session; `setval()` sets the sequence to a specific value.
- **PostgreSQL `SERIAL`:** A shorthand for creating an integer column with an owned sequence and a default value of `nextval()`.
- **SQL Server:** Sequences are independent objects; use `NEXT VALUE FOR sequence_name` to retrieve the next value.
- **MySQL:** Uses `AUTO_INCREMENT` column attribute; not a separate object. `LAST_INSERT_ID()` retrieves the most recent auto-generated value.
- **Oracle:** `CREATE SEQUENCE` is the native mechanism; `sequence_name.NEXTVAL` and `sequence_name.CURRVAL` are used in SQL.

#### Constraints and Limitations

- **Sequence gaps:** Sequences can have gaps due to transaction rollbacks, caching, or concurrent access.
- **CYCLE risk:** Cycling sequences can generate duplicate values, violating primary key constraints.
- **CACHE trade-off:** Larger cache sizes improve performance but may cause larger gaps after a crash.
- **MySQL:** `AUTO_INCREMENT` columns must be indexed and can have only one per table.
- **Version-specific:** SQL Server sequences require SQL Server 2012+; PostgreSQL `AS data_type` requires PostgreSQL 10+.

### Annotated Code Examples

#### Example 1: PostgreSQL — Creating and Using a Sequence

```sql
-- Create a sequence
CREATE SEQUENCE order_id_seq
    START WITH 1000
    INCREMENT BY 1
    NO MAXVALUE
    CACHE 10;

-- Use the sequence in an INSERT
INSERT INTO orders (order_id, customer_id, order_date, total)
VALUES (nextval('order_id_seq'), 101, CURRENT_DATE, 150.00);

-- Check the current value
SELECT currval('order_id_seq') AS current_order_id;
```

**Expected Output:**

```
 current_order_id
------------------
             1000
```

**Why This Works:** The sequence starts at 1000 and increments by 1. `nextval()` returns the next value (1000) and advances the sequence. `currval()` returns the value most recently obtained by `nextval()` in the current session.

#### Example 2: SQL Server — Creating and Using a Sequence

```sql
-- Create a sequence
CREATE SEQUENCE dbo.OrderIDSequence
    AS INT
    START WITH 1000
    INCREMENT BY 1
    NO CYCLE
    CACHE 50;

-- Use the sequence in an INSERT
INSERT INTO dbo.Orders (OrderID, CustomerID, OrderDate, Total)
VALUES (NEXT VALUE FOR dbo.OrderIDSequence, 101, GETDATE(), 150.00);
```

**Expected Output:**

```
(1 row affected)
```

**Why This Works:** SQL Server sequences are independent objects. `NEXT VALUE FOR` retrieves the next value from the sequence. The `CACHE 50` option preallocates 50 values for performance.

#### Example 3: MySQL — AUTO_INCREMENT

```sql
-- Create a table with AUTO_INCREMENT
CREATE TABLE animals (
    id MEDIUMINT NOT NULL AUTO_INCREMENT,
    name CHAR(30) NOT NULL,
    PRIMARY KEY (id)
) ENGINE=InnoDB;

-- Insert rows; id is auto-generated
INSERT INTO animals (name) VALUES ('dog'), ('cat'), ('penguin');

-- Retrieve the last inserted ID
SELECT LAST_INSERT_ID() AS last_id;
```

**Expected Output:**

```
last_id
-------
      3
```

**Why This Works:** The `AUTO_INCREMENT` attribute automatically generates unique values for the `id` column. `LAST_INSERT_ID()` returns the first auto-generated value from the most recent `INSERT` statement.

### Real-World Cases

- **Primary key generation:** Sequences provide unique IDs for `INSERT` statements.
- **Shared counters:** Multiple tables can draw from the same sequence for globally unique IDs.
- **Multi-tenant systems:** Each tenant gets its own sequence with a distinct range.
- **Distributed systems:** Sequences with different increment values can generate non-overlapping ID ranges across nodes.

### References

- PostgreSQL Documentation: CREATE SEQUENCE — https://www.postgresql.org/docs/current/sql-createsequence.html
- SQL Server: CREATE SEQUENCE — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-sequence-transact-sql
- MySQL Reference Manual: AUTO_INCREMENT — https://dev.mysql.com/doc/refman/8.0/en/example-auto-increment.html
- Oracle Database: CREATE SEQUENCE — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CREATE-SEQUENCE.html


## Core Concept 6: Constraints

### Definitions

**Core Definition:** A constraint is a rule enforced by the database that restricts the values that can be stored in a table, ensuring data integrity and consistency.

**Technical Definition:** Constraints are declarative rules attached to table columns or the table as a whole. They are defined either inline (as part of a column definition) or out-of-line (as a separate table constraint). The five main types are: `NOT NULL` (column cannot be NULL), `UNIQUE` (values must be unique), `PRIMARY KEY` (unique and not null), `FOREIGN KEY` (values must match a parent table), and `CHECK` (values must satisfy a boolean condition). Constraints can be named (for easier reference in error messages and ALTER statements) or unnamed.

**Beginner-Friendly Explanation:** Constraints are like the rules on a form. "This field is required" (NOT NULL), "This must be a valid email address" (CHECK), "This ID must be unique" (UNIQUE), "This department must exist" (FOREIGN KEY). They keep your data clean and consistent.

### Purposes

- To enforce data integrity rules at the database level, independent of application code.
- To prevent invalid data from entering the table.
- To establish relationships between tables (foreign keys).
- To provide meaningful error messages when data violates a rule.

### Syntax Rules and Structure

#### Complete General Syntax (Inline Column Constraint)

```sql
CREATE TABLE table_name (
    column_name data_type
        [ CONSTRAINT constraint_name ]
        { NOT NULL | NULL | UNIQUE | PRIMARY KEY | CHECK ( condition ) | REFERENCES parent_table ( parent_column ) }
        [ ... ]
);
```

#### Complete General Syntax (Out-of-Line Table Constraint)

```sql
CREATE TABLE table_name (
    column1 data_type,
    column2 data_type,
    ...
    [ CONSTRAINT constraint_name ]
    { PRIMARY KEY ( column1, ... ) |
      UNIQUE ( column1, ... ) |
      FOREIGN KEY ( column1, ... ) REFERENCES parent_table ( parent_column, ... ) |
      CHECK ( condition ) }
);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `CONSTRAINT constraint_name` | Optional name for the constraint. |
| `NOT NULL` | Column must contain a value (cannot be NULL). |
| `UNIQUE` | All values in the column (or combination of columns) must be distinct. |
| `PRIMARY KEY` | Combination of `UNIQUE` and `NOT NULL`; uniquely identifies rows. |
| `FOREIGN KEY` | Values must match values in a parent table's key. |
| `CHECK ( condition )` | Values must satisfy a boolean condition. |

#### Syntax Rules

- **Inline vs. out-of-line:** Inline constraints are defined as part of a column definition; out-of-line constraints are defined after all columns. `NOT NULL` is always inline; `CHECK` can be either; `PRIMARY KEY`, `UNIQUE`, and `FOREIGN KEY` can be either.
- **Named constraints:** Use `CONSTRAINT name` to give the constraint a meaningful name; otherwise the database generates one (e.g., `employees_salary_check`).
- **CHECK constraints (MySQL):** Enforced only in MySQL 8.0.16+; earlier versions parsed but ignored them.
- **FOREIGN KEY:** Requires the parent table's referenced column to have a `PRIMARY KEY` or `UNIQUE` constraint.
- **DEFERRABLE:** In PostgreSQL, constraints can be declared `DEFERRABLE INITIALLY DEFERRED` to postpone checking until the end of the transaction.

#### Constraints and Limitations

- **NOT NULL is not a CHECK:** `NULL` values evaluate to `UNKNOWN`, not `FALSE`, so a `CHECK` constraint cannot enforce non-nullness.
- **MySQL CHECK:** `CHECK` constraints are not enforced for `INSERT IGNORE` or `UPDATE IGNORE` statements.
- **SQL Server:** A table can have a maximum of 249 non-clustered indexes and constraints that create indexes.
- **Foreign key cascading:** `ON DELETE CASCADE` and `ON UPDATE CASCADE` can propagate changes to child tables; use with caution.

### Annotated Code Examples

#### Example 1: PostgreSQL — Inline and Out-of-Line Constraints

```sql
-- Create a table with inline and out-of-line constraints
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,                          -- Inline PRIMARY KEY
    product_name VARCHAR(100) NOT NULL,                     -- Inline NOT NULL
    sku VARCHAR(20) UNIQUE,                                 -- Inline UNIQUE
    price NUMERIC(10,2) CHECK (price > 0),                  -- Inline CHECK
    category_id INT,
    CONSTRAINT fk_category FOREIGN KEY (category_id)
        REFERENCES categories (category_id)                 -- Out-of-line FOREIGN KEY
        ON DELETE SET NULL,
    CONSTRAINT chk_sku_format CHECK (sku ~ '^[A-Z]{2}-[0-9]{4}$')  -- Out-of-line CHECK
);
```

**Expected Output:**

```
CREATE TABLE
```

**Why This Works:** The inline constraints are defined directly on columns. The out-of-line constraints (`fk_category`, `chk_sku_format`) are defined after all columns. The `ON DELETE SET NULL` action sets the `category_id` to NULL if the referenced category is deleted.

#### Example 2: MySQL — CHECK Constraint

```sql
-- Create a table with CHECK constraints
CREATE TABLE employees (
    emp_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    salary DECIMAL(10,2) CHECK (salary > 0),
    hire_date DATE,
    termination_date DATE,
    CONSTRAINT chk_dates CHECK (termination_date IS NULL OR termination_date > hire_date)
) ENGINE=InnoDB;

-- Valid insert
INSERT INTO employees (first_name, last_name, salary, hire_date)
VALUES ('Alice', 'Smith', 75000, '2026-01-15');

-- Invalid insert (negative salary)
INSERT INTO employees (first_name, last_name, salary)
VALUES ('Bob', 'Jones', -5000);
```

**Expected Output (for the invalid insert):**

```
ERROR 3819 (HY000): Check constraint 'employees_chk_1' is violated.
```

**Why This Works:** MySQL 8.0.16+ enforces `CHECK` constraints. The `chk_dates` constraint ensures that `termination_date` is either NULL or after `hire_date`. The `salary > 0` check prevents negative salaries.

### Real-World Cases

- **User registration:** `UNIQUE` constraint on email addresses; `NOT NULL` on username and password.
- **E-commerce:** `FOREIGN KEY` from orders to customers; `CHECK` on product prices.
- **Financial systems:** `CHECK` on account balances; `FOREIGN KEY` from transactions to accounts.
- **HR systems:** `CHECK` on hire dates; `UNIQUE` on employee IDs.

### References

- PostgreSQL Documentation: Constraints — https://www.postgresql.org/docs/current/ddl-constraints.html
- MySQL Reference Manual: CHECK Constraints — https://dev.mysql.com/doc/refman/8.0/en/create-table-check-constraints.html
- SQL Server: Constraints — https://learn.microsoft.com/en-us/sql/relational-databases/tables/unique-constraints-and-check-constraints
- Oracle Database: Constraints — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/constraint.html


## Core Concept 7: Triggers

### Definitions

**Core Definition:** A trigger is a database object that automatically executes a specified function or procedure in response to a data modification event (INSERT, UPDATE, DELETE) or a DDL event on a table, view, or schema.

**Technical Definition:** Triggers are event-driven procedural constructs. DML triggers fire on `INSERT`, `UPDATE`, or `DELETE` operations on tables or views. DDL triggers fire on `CREATE`, `ALTER`, or `DROP` statements. Triggers can fire `BEFORE` the event (allowing modification of the row), `AFTER` the event (for auditing or cascading actions), or `INSTEAD OF` the event (for making views updatable). Row-level triggers execute once per affected row; statement-level triggers execute once per statement regardless of the number of rows affected.

**Beginner-Friendly Explanation:** A trigger is like a motion sensor alarm. When something happens (someone inserts, updates, or deletes data), the alarm goes off and a predefined action is executed—like logging the change, updating another table, or validating the data.

### Purposes

- To enforce complex business rules that cannot be expressed with simple constraints.
- To maintain audit trails (recording who changed what and when).
- To automatically update derived or denormalized data.
- To make views updatable through `INSTEAD OF` triggers.
- To enforce DDL policies (preventing unauthorized schema changes).

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
CREATE [ CONSTRAINT ] TRIGGER name
    { BEFORE | AFTER | INSTEAD OF } { event [ OR ... ] }
    ON table_name
    [ FROM referenced_table_name ]
    [ NOT DEFERRABLE | [ DEFERRABLE ] [ INITIALLY IMMEDIATE | INITIALLY DEFERRED ] ]
    [ REFERENCING { { OLD | NEW } TABLE [ AS ] transition_relation_name } [ ... ] ]
    [ FOR [ EACH ] { ROW | STATEMENT } ]
    [ WHEN ( condition ) ]
    EXECUTE { FUNCTION | PROCEDURE } function_name ( arguments );
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `BEFORE` / `AFTER` / `INSTEAD OF` | When the trigger fires relative to the event. |
| `event` | `INSERT`, `UPDATE [OF column]`, `DELETE`, or `TRUNCATE`. |
| `ON table_name` | The table or view the trigger is attached to. |
| `FOR EACH ROW` / `FOR EACH STATEMENT` | Row-level or statement-level execution. |
| `WHEN ( condition )` | Optional boolean condition that must be true for the trigger to fire. |
| `EXECUTE FUNCTION` | The trigger function to execute. |

#### Complete General Syntax (MySQL)

```sql
CREATE [DEFINER = user] TRIGGER trigger_name
    trigger_time trigger_event
    ON tbl_name FOR EACH ROW
    [trigger_order]
    trigger_body;

trigger_time: { BEFORE | AFTER }
trigger_event: { INSERT | UPDATE | DELETE }
trigger_order: { FOLLOWS | PRECEDES } other_trigger_name
```

#### Complete General Syntax (SQL Server)

```sql
CREATE [ OR ALTER ] TRIGGER [ schema_name . ] trigger_name
    ON { table | view }
    [ WITH <dml_trigger_option> [ , ...n ] ]
    { FOR | AFTER | INSTEAD OF }
    { [ INSERT ] [ , ] [ UPDATE ] [ , ] [ DELETE ] }
    [ NOT FOR REPLICATION ]
    AS { sql_statement [ ; ] [ , ...n ] | EXTERNAL NAME <method_specifier> };
```

#### Syntax Rules

- **PostgreSQL:** Trigger functions must be defined before the trigger itself can be created; the function must return type `trigger`.
- **MySQL:** Triggers are defined directly in the `CREATE TRIGGER` statement; there is no separate function. `BEFORE` triggers can modify the `NEW` row; `AFTER` triggers cannot.
- **SQL Server:** `INSTEAD OF` triggers are used on views to make them updatable. `AFTER` triggers fire after the statement completes. `FOR` is synonymous with `AFTER`.
- **Row-level vs. statement-level:** A row-level trigger fires once per affected row; a statement-level trigger fires once per SQL statement, regardless of the number of rows.
- **INSTEAD OF:** Must be row-level and can only be defined on views (PostgreSQL).

#### Constraints and Limitations

- **Trigger cascades:** Triggers can fire other triggers, leading to cascading effects; use `pg_trigger_depth()` (PostgreSQL) to detect recursion.
- **Performance:** Row-level triggers on large tables can significantly slow down bulk operations.
- **MySQL:** Cannot create triggers on temporary tables or views.
- **SQL Server:** `INSTEAD OF` triggers cannot be defined on tables with cascading referential integrity.
- **Ordering:** Multiple triggers on the same event fire in alphabetical order by name (PostgreSQL) or can be ordered with `FOLLOWS`/`PRECEDES` (MySQL).

### Annotated Code Examples

#### Example 1: PostgreSQL — Audit Trigger

```sql
-- Create an audit log table
CREATE TABLE employee_audit (
    audit_id SERIAL PRIMARY KEY,
    emp_id INT,
    action TEXT,
    old_salary NUMERIC(10,2),
    new_salary NUMERIC(10,2),
    changed_at TIMESTAMP DEFAULT NOW()
);

-- Create a trigger function
CREATE OR REPLACE FUNCTION log_salary_change()
RETURNS TRIGGER AS $$
BEGIN
    IF OLD.salary IS DISTINCT FROM NEW.salary THEN
        INSERT INTO employee_audit (emp_id, action, old_salary, new_salary)
        VALUES (NEW.emp_id, 'SALARY_CHANGE', OLD.salary, NEW.salary);
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Attach the trigger
CREATE TRIGGER trg_salary_change
    AFTER UPDATE ON employees
    FOR EACH ROW
    EXECUTE FUNCTION log_salary_change();

-- Test: Update a salary
UPDATE employees SET salary = 95000 WHERE emp_id = 1;

-- Check audit log
SELECT * FROM employee_audit;
```

**Expected Output:**

```
 audit_id | emp_id |    action     | old_salary | new_salary |         changed_at
----------+--------+---------------+------------+------------+----------------------------
        1 |      1 | SALARY_CHANGE |   90000.00 |   95000.00 | 2026-09-26 10:30:00.123456
```

**Why This Works:** The `AFTER UPDATE` trigger fires once for each updated row. The trigger function compares the old and new salary values and inserts an audit record if they differ. The `IS DISTINCT FROM` operator handles NULL comparisons correctly.

#### Example 2: MySQL — BEFORE INSERT Trigger

```sql
-- Create a trigger that normalizes email before insert
DELIMITER //
CREATE TRIGGER normalize_email
BEFORE INSERT ON users
FOR EACH ROW
BEGIN
    SET NEW.email = LOWER(TRIM(NEW.email));
END;
//
DELIMITER ;

-- Insert a row with unnormalized email
INSERT INTO users (name, email) VALUES ('Alice', '  ALICE@EXAMPLE.COM  ');

-- Check the result
SELECT name, email FROM users;
```

**Expected Output:**

```
name  | email
------+--------------------
Alice | alice@example.com
```

**Why This Works:** The `BEFORE INSERT` trigger fires before the row is inserted, allowing modification of the `NEW` row. The trigger normalizes the email by trimming whitespace and converting to lowercase. This pattern is commonly used for data cleansing.

### Real-World Cases

- **Audit trails:** Logging all changes to sensitive tables (salary, permissions).
- **Data validation:** Complex validation that cannot be expressed with `CHECK` constraints (e.g., cross-table rules).
- **Denormalization:** Updating summary tables or counters when base data changes.
- **Updatable views:** `INSTEAD OF` triggers on views that join multiple tables.

### References

- PostgreSQL Documentation: CREATE TRIGGER — https://www.postgresql.org/docs/current/sql-createtrigger.html
- MySQL Reference Manual: CREATE TRIGGER — https://dev.mysql.com/doc/refman/8.0/en/create-trigger.html
- SQL Server: CREATE TRIGGER — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-trigger-transact-sql
- Oracle Database: CREATE TRIGGER — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CREATE-TRIGGER.html


## Core Concept 8: Stored Procedures

### Definitions

**Core Definition:** A stored procedure is a precompiled, named collection of SQL statements and procedural logic stored in the database that can be executed on demand with parameters.

**Technical Definition:** A stored procedure is a database object that encapsulates a set of SQL statements and procedural logic (such as control flow, error handling, and variable declarations) into a single callable unit. Procedures support input parameters (`IN`), output parameters (`OUT`), and input-output parameters (`INOUT`). They execute with either `DEFINER` rights (the privileges of the procedure owner) or `INVOKER` rights (the privileges of the calling user). Procedures can contain transaction control statements (`COMMIT`, `ROLLBACK`) in some databases.

**Beginner-Friendly Explanation:** A stored procedure is like a recipe saved in the database. Instead of writing the same steps every time you need to perform a task, you save the recipe once and just say "make this recipe" (call the procedure). You can pass ingredients (parameters) to customize the result.

### Purposes

- To encapsulate complex business logic in a reusable, callable unit.
- To reduce network traffic by executing multiple statements on the server.
- To improve performance through precompilation and caching of execution plans.
- To enforce security by granting `EXECUTE` permission on the procedure without granting direct table access.
- To support transaction control within procedural logic.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
CREATE [ OR REPLACE ] PROCEDURE name ( [ [ argmode ] [ argname ] argtype
    [ { DEFAULT | = } default_expr ] [, ...] ] )
    { LANGUAGE lang_name
    | TRANSFORM { FOR TYPE type_name } [, ... ]
    | [ EXTERNAL ] SECURITY INVOKER | [ EXTERNAL ] SECURITY DEFINER
    | SET configuration_parameter { TO value | = value | FROM CURRENT }
    | AS 'definition'
    | AS 'obj_file', 'link_symbol'
    | sql_body
    } ...
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `argmode` | `IN`, `OUT`, `INOUT`, or `VARIADIC` (default: `IN`). |
| `argname` | Parameter name. |
| `argtype` | Parameter data type. |
| `SECURITY DEFINER` | Executes with the privileges of the procedure owner. |
| `SECURITY INVOKER` | Executes with the privileges of the calling user (default). |
| `LANGUAGE` | Procedural language (e.g., `plpgsql`, `sql`). |

#### Complete General Syntax (MySQL)

```sql
CREATE [DEFINER = user] PROCEDURE sp_name ([proc_parameter[,...]])
    [characteristic ...]
    routine_body;

proc_parameter: [ IN | OUT | INOUT ] param_name type
characteristic: {
    COMMENT 'string'
  | LANGUAGE SQL
  | [NOT] DETERMINISTIC
  | { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
  | SQL SECURITY { DEFINER | INVOKER }
}
```

#### Complete General Syntax (SQL Server)

```sql
CREATE [ OR ALTER ] { PROC | PROCEDURE } [schema_name.] procedure_name
    [ { @parameter_name [ type_schema_name. ] data_type }
        [ VARYING ] [ NULL ] [ = default ] [ OUT | OUTPUT | [READONLY] ]
    ] [ ,...n ]
    [ WITH <procedure_option> [ ,...n ] ]
    [ FOR REPLICATION ]
    AS { [ BEGIN ] sql_statement [;] [ ...n ] [ END ] }
```

#### Syntax Rules

- **PostgreSQL:** `CALL procedure_name(...)` executes a procedure. Procedures can contain transaction control (`COMMIT`, `ROLLBACK`) if called outside a transaction block.
- **MySQL:** `CALL procedure_name(...)` executes a procedure. Procedures can contain DDL statements. `DELIMITER` is typically changed before defining the procedure body.
- **SQL Server:** `EXEC procedure_name` or `EXECUTE procedure_name` executes a procedure. Output parameters are specified with `OUTPUT`.
- **DEFINER vs. INVOKER:** `DEFINER` procedures execute with the owner's privileges, useful for granting controlled access. `INVOKER` procedures execute with the caller's privileges, respecting the caller's security context.

#### Constraints and Limitations

- **Transaction control:** PostgreSQL procedures can only contain transaction control statements when called from a `CALL` statement outside an explicit transaction block.
- **MySQL:** Stored procedures cannot return a result set directly (unlike functions); they can return multiple result sets.
- **SQL Server:** Stored procedures can return a status code (integer) indicating success or failure.
- **Overloading (PostgreSQL):** Procedures with the same name but different argument types can coexist (overloading).

### Annotated Code Examples

#### Example 1: PostgreSQL — Procedure with IN and OUT Parameters

```sql
-- Create a procedure that calculates department statistics
CREATE OR REPLACE PROCEDURE get_dept_stats(
    IN dept_name_param TEXT,
    OUT emp_count INT,
    OUT avg_salary NUMERIC
)
LANGUAGE plpgsql
AS $$
BEGIN
    SELECT COUNT(*), AVG(salary)
    INTO emp_count, avg_salary
    FROM employees e
    JOIN departments d ON e.dept_id = d.dept_id
    WHERE d.dept_name = dept_name_param;
END;
$$;

-- Call the procedure
CALL get_dept_stats('Engineering', NULL, NULL);
```

**Expected Output:**

```
 emp_count | avg_salary
-----------+------------
         2 | 87500.00
```

**Why This Works:** The procedure accepts an `IN` parameter (`dept_name_param`) and returns two `OUT` parameters (`emp_count`, `avg_salary`). The `INTO` clause assigns the query results to the output parameters. The `CALL` statement executes the procedure.

#### Example 2: MySQL — Procedure with Transaction Control

```sql
DELIMITER //
CREATE PROCEDURE transfer_funds(
    IN from_account INT,
    IN to_account INT,
    IN amount DECIMAL(10,2)
)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Transfer failed';
    END;

    START TRANSACTION;
    UPDATE accounts SET balance = balance - amount WHERE account_id = from_account;
    UPDATE accounts SET balance = balance + amount WHERE account_id = to_account;
    COMMIT;
END;
//
DELIMITER ;

-- Execute the procedure
CALL transfer_funds(101, 102, 500.00);
```

**Expected Output:**

```
Query OK, 0 rows affected
```

**Why This Works:** The procedure uses `START TRANSACTION` and `COMMIT` to ensure that both updates succeed or both fail. The `EXIT HANDLER` catches any SQL exception, rolls back the transaction, and signals a custom error. This is a classic banking transfer pattern.

### Real-World Cases

- **Banking:** Funds transfer procedures with transaction control and error handling.
- **Order processing:** Procedures that validate inventory, create orders, and update stock atomically.
- **Batch operations:** Procedures that perform nightly cleanup or aggregation tasks.
- **Security:** Procedures that grant controlled access to sensitive data without exposing base tables.

### References

- PostgreSQL Documentation: CREATE PROCEDURE — https://www.postgresql.org/docs/current/sql-createprocedure.html
- MySQL Reference Manual: CREATE PROCEDURE — https://dev.mysql.com/doc/refman/8.0/en/create-procedure.html
- SQL Server: CREATE PROCEDURE — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-procedure-transact-sql
- Oracle Database: CREATE PROCEDURE — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CREATE-PROCEDURE.html


## Core Concept 9: Functions

### Definitions

**Core Definition:** A function is a named, reusable database object that accepts zero or more input parameters, performs a computation or query, and returns a single value (scalar function) or a set of rows (table-valued function).

**Technical Definition:** A function is a database object that encapsulates logic and returns a value. Scalar functions return a single value (e.g., an integer, string, or date). Table-valued functions (TVFs) return a set of rows that can be used in the `FROM` clause like a table. Functions are characterized as deterministic (always return the same result for the same inputs) or non-deterministic (may return different results for the same inputs, e.g., functions that read the current time). In PostgreSQL, functions can be written in SQL, PL/pgSQL, C, or other languages. MySQL distinguishes between stored functions (defined with `CREATE FUNCTION` and stored in the database) and user-defined functions (UDFs, implemented in C/C++ and loaded as shared libraries).

**Beginner-Friendly Explanation:** A function is like a calculator that takes input and produces output. A scalar function is like a simple calculator: you give it a number, it gives you back a result. A table-valued function is like a search engine: you give it a keyword, and it gives you back a list of results (rows).

### Purposes

- To encapsulate reusable computation logic that can be called from SQL statements.
- To extend the database with custom scalar or table-valued logic.
- To simplify complex queries by encapsulating subqueries in table-valued functions.
- To support parameterized views through table-valued functions.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
CREATE [ OR REPLACE ] FUNCTION name ( [ [ argmode ] [ argname ] argtype
    [ { DEFAULT | = } default_expr ] [, ...] ] )
    [ RETURNS rettype
      | RETURNS TABLE ( column_name column_type [, ...] ) ]
    { LANGUAGE lang_name
    | TRANSFORM { FOR TYPE type_name } [, ... ]
    | IMMUTABLE | STABLE | VOLATILE
    | [ NOT ] LEAKPROOF
    | CALLED ON NULL INPUT | RETURNS NULL ON NULL INPUT | STRICT
    | [ EXTERNAL ] SECURITY INVOKER | [ EXTERNAL ] SECURITY DEFINER
    | PARALLEL { UNSAFE | RESTRICTED | SAFE }
    | COST execution_cost
    | ROWS result_rows
    | SUPPORT support_function
    | SET configuration_parameter { TO value | = value | FROM CURRENT }
    | AS 'definition'
    | AS 'obj_file', 'link_symbol'
    | sql_body
    } ...
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `RETURNS rettype` | The return type for scalar functions. |
| `RETURNS TABLE (...)` | Defines a table-valued function returning a set of rows. |
| `IMMUTABLE` | Function cannot modify the database and always returns the same result for the same arguments. |
| `STABLE` | Function cannot modify the database and returns the same result for the same arguments within a single table scan. |
| `VOLATILE` | Function may modify the database and return different results on successive calls. |
| `STRICT` | Function returns NULL if any argument is NULL. |

#### Complete General Syntax (MySQL)

```sql
CREATE [DEFINER = user] FUNCTION sp_name ([func_parameter[,...]])
    RETURNS type
    [characteristic ...]
    routine_body;

func_parameter: param_name type
characteristic: {
    COMMENT 'string'
  | LANGUAGE SQL
  | [NOT] DETERMINISTIC
  | { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
  | SQL SECURITY { DEFINER | INVOKER }
}
```

#### Syntax Rules

- **Scalar vs. table-valued:** Scalar functions return a single value and can be used in `SELECT`, `WHERE`, and `ORDER BY`. Table-valued functions return a set of rows and are used in the `FROM` clause.
- **Deterministic vs. non-deterministic:** In MySQL, `DETERMINISTIC` means the function always returns the same result for the same inputs; `NOT DETERMINISTIC` means it may return different results. This affects binary logging and replication.
- **Volatility (PostgreSQL):** `IMMUTABLE` functions can be used in index expressions; `STABLE` functions are evaluated once per query; `VOLATILE` functions are evaluated once per row.
- **MySQL UDFs:** User-defined functions are created with `CREATE FUNCTION ... SONAME` and are implemented in C/C++; they are distinct from stored functions.

#### Constraints and Limitations

- **Function overloading (PostgreSQL):** Functions with the same name but different argument types can coexist. Functions with the same name and input argument types cannot coexist, even if output types differ.
- **SQL Server:** Scalar functions can be used in `SELECT`, `WHERE`, and `ORDER BY`; table-valued functions are used in the `FROM` clause.
- **MySQL:** Stored functions cannot return result sets; they return a single scalar value. UDFs return `STRING`, `INTEGER`, `REAL`, or `DECIMAL`.
- **Version-specific:** PostgreSQL 11+ supports `CREATE FUNCTION ... sql_body` with `BEGIN ATOMIC` for SQL-standard function bodies.

### Annotated Code Examples

#### Example 1: PostgreSQL — Scalar Function

```sql
-- Create a scalar function that calculates annual salary
CREATE OR REPLACE FUNCTION annual_salary(monthly_salary NUMERIC)
RETURNS NUMERIC
LANGUAGE SQL
IMMUTABLE
AS $$
    SELECT monthly_salary * 12;
$$;

-- Use the function in a query
SELECT emp_name, salary, annual_salary(salary) AS annual
FROM employees;
```

**Expected Output:**

```
 emp_name |  salary  |   annual
----------+----------+------------
 Alice    | 90000.00 | 1080000.00
 Bob      | 85000.00 | 1020000.00
 Carol    | 75000.00 |  900000.00
```

**Why This Works:** The function `annual_salary` takes a numeric input and returns the input multiplied by 12. The `IMMUTABLE` keyword tells PostgreSQL that the function always returns the same result for the same input, allowing it to be used in indexes. The function is called in the `SELECT` list.

#### Example 2: PostgreSQL — Table-Valued Function

```sql
-- Create a table-valued function that returns employees by department
CREATE OR REPLACE FUNCTION employees_in_dept(dept_name_param TEXT)
RETURNS TABLE (emp_name TEXT, salary NUMERIC)
LANGUAGE SQL
STABLE
AS $$
    SELECT e.emp_name, e.salary
    FROM employees e
    JOIN departments d ON e.dept_id = d.dept_id
    WHERE d.dept_name = dept_name_param;
$$;

-- Use the function in the FROM clause
SELECT * FROM employees_in_dept('Engineering');
```

**Expected Output:**

```
 emp_name |  salary
----------+----------
 Alice    | 90000.00
 Bob      | 85000.00
```

**Why This Works:** The function returns a table with two columns (`emp_name`, `salary`). The `RETURNS TABLE` clause defines the output schema. The function is used in the `FROM` clause like a regular table. `STABLE` indicates that the function does not modify the database and returns consistent results within a single query.

### Real-World Cases

- **Data transformation:** Scalar functions for formatting dates, calculating ages, or parsing strings.
- **Parameterized views:** Table-valued functions that accept parameters and return filtered result sets.
- **Business calculations:** Functions for computing taxes, discounts, or commissions.
- **JSON processing:** Functions that extract or transform JSON data.

### References

- PostgreSQL Documentation: CREATE FUNCTION — https://www.postgresql.org/docs/current/sql-createfunction.html
- MySQL Reference Manual: CREATE FUNCTION — https://dev.mysql.com/doc/refman/8.0/en/create-function.html
- SQL Server: CREATE FUNCTION — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-function-transact-sql
- Oracle Database: CREATE FUNCTION — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CREATE-FUNCTION.html


## Summary Table: Database Object Creation Across DBMS

| Object | PostgreSQL | MySQL | SQL Server | Oracle |
|--------|-----------|-------|------------|--------|
| **Table** | `CREATE TABLE` | `CREATE TABLE` | `CREATE TABLE` | `CREATE TABLE` |
| **Temporary Table** | `CREATE TEMP TABLE` | `CREATE TEMPORARY TABLE` | `CREATE TABLE #temp` | `CREATE GLOBAL TEMPORARY TABLE` |
| **Schema** | `CREATE SCHEMA` | `CREATE DATABASE` (schema ≈ database) | `CREATE SCHEMA` | `CREATE USER` (schema ≈ user) |
| **View** | `CREATE VIEW` | `CREATE VIEW` | `CREATE VIEW` | `CREATE VIEW` |
| **Materialized View** | `CREATE MATERIALIZED VIEW` | HeatWave only | Indexed view | `CREATE MATERIALIZED VIEW` |
| **Index** | `CREATE INDEX` | `CREATE INDEX` | `CREATE INDEX` | `CREATE INDEX` |
| **Sequence** | `CREATE SEQUENCE` | `AUTO_INCREMENT` | `CREATE SEQUENCE` | `CREATE SEQUENCE` |
| **Constraint** | Inline / Out-of-line | Inline / Out-of-line | Inline / Out-of-line | Inline / Out-of-line |
| **Trigger** | `CREATE TRIGGER` | `CREATE TRIGGER` | `CREATE TRIGGER` | `CREATE TRIGGER` |
| **Procedure** | `CREATE PROCEDURE` | `CREATE PROCEDURE` | `CREATE PROCEDURE` | `CREATE PROCEDURE` |
| **Function** | `CREATE FUNCTION` | `CREATE FUNCTION` | `CREATE FUNCTION` | `CREATE FUNCTION` |


## Final Notes on Deprecated and Unsafe Features

- **PostgreSQL `UNLOGGED` tables:** Not crash-safe; data is lost after a crash. Use only for regenerable data (staging areas, caches).
- **PostgreSQL `SERIAL`:** Deprecated in favor of `GENERATED ... AS IDENTITY` (SQL standard). `SERIAL` creates an owned sequence and sets the default to `nextval()`.
- **MySQL `CHECK` before 8.0.16:** Parsed but ignored. Do not rely on `CHECK` constraints for data integrity in MySQL versions before 8.0.16.
- **MySQL `AUTO_INCREMENT` gaps:** Auto-increment values are not guaranteed to be sequential; gaps occur due to rollbacks and concurrent inserts.
- **SQL Server `WITH NOCHECK`:** Constraints added with `WITH NOCHECK` are not trusted by the query optimizer; re-enable with `WITH CHECK CHECK CONSTRAINT` for optimal plans.
- **PostgreSQL function volatility:** Incorrect volatility marking (e.g., marking a volatile function as `IMMUTABLE`) can cause index corruption.
- **SQL Server filtered indexes:** Cannot be created on computed columns, UDT columns, spatial columns, or hierarchy ID columns; cannot use `NULL` comparisons in the filter predicate.
- **Version-specific:** SQL Server sequences (2012+), PostgreSQL `CREATE PROCEDURE` (11+), PostgreSQL `GENERATED ... AS IDENTITY` (10+), MySQL `CHECK` enforcement (8.0.16+), MySQL `RENAME COLUMN` (8.0+).