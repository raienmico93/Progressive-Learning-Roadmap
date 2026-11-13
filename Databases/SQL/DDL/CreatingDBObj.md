# Comprehensive Programming Cheat Sheet: Creating Database Objects

## Topic Overview

### Definitions

**Core Definition**
Creating database objects is the set of Data Definition Language (DDL) statements used to define and instantiate the structural components of a relational database—tables, schemas, views, indexes, sequences, constraints, triggers, stored procedures, and functions—that together form the database's architecture.

**Technical Definition**
Database objects are the named, persistent entities registered in a database's system catalogs. Each object type has a distinct role: tables store rows; schemas provide namespaces; views expose stored queries; indexes accelerate data access; sequences generate numeric values; constraints enforce data integrity; triggers execute logic in response to data events; stored procedures encapsulate procedural logic callable via `CALL`; and functions return values within SQL expressions. Creation statements vary across PostgreSQL, MySQL, and SQL Server, though core DDL concepts are standardised.

**Beginner-Friendly Explanation**
A database is like a city. Tables are the buildings where data lives. Schemas are the districts that organise those buildings. Views are the display windows that let you see selected parts of the data. Indexes are the street signs that help you find things quickly. Sequences are the ticket dispensers that hand out unique numbers. Constraints are the building codes that keep everything safe and consistent. Triggers are the automatic doors that open when someone walks in. Stored procedures and functions are the service counters where you submit a request and get a result. Creating database objects is how you build this city, piece by piece.

### Key Characteristics

- **Interdependence**: Tables depend on schemas for their namespace; constraints and indexes attach to tables; triggers reference functions; views depend on tables. Creation order matters.
- **Vendor-specific extensions**: Core DDL is standardised, but features such as PostgreSQL's `UNLOGGED` tables, MySQL's `AUTO_INCREMENT`, and SQL Server's `IDENTITY` are implementation-defined.
- **Privilege-gated**: Each object type requires specific privileges—`CREATE` on the schema for tables, `USAGE` on the language for functions, `TRIGGER` privilege for triggers.
- **Catalog visibility**: Every created object is recorded in `information_schema` and vendor-specific system catalogs, enabling introspection and automation.
- **Reversibility**: Objects can be altered and dropped with `ALTER` and `DROP`, but `DROP` is typically irreversible and may cascade to dependent objects.

### Prerequisites

- A running RDBMS instance (PostgreSQL, MySQL, or SQL Server) and a client connection.
- `CREATE` privilege on the target schema or database.
- Basic familiarity with SQL data types and the `SELECT` statement.
- For functions and procedures: `USAGE` privilege on the procedural language (e.g., `plpgsql` in PostgreSQL).
- For triggers: `TRIGGER` privilege on the target table.

### Related Programming Areas

- Data Definition Language (DDL) – the umbrella category for object creation.
- Data Manipulation Language (DML) – operates on the data stored in tables and views.
- Data Control Language (DCL) – `GRANT` and `REVOKE` govern access to created objects.
- Query optimisation – indexes and views directly influence execution plans.
- Application architecture – schemas and stored procedures shape how applications interact with the database.
- Schema versioning – migration tools such as Flyway and Liquibase track the creation and evolution of database objects.

---

## Core Concepts and Features

### 1. Tables

#### Definitions

**Core Definition**
A table is the primary database object that stores data as rows (records) and columns (attributes).

**Technical Definition**
A table is a relation with a fixed set of columns, each with a defined data type, and zero or more rows. It is created with `CREATE TABLE`, which specifies column definitions, constraints, storage parameters, and optional persistence characteristics. Tables are owned by a schema and a database user, and their structure is recorded in system catalogs such as `information_schema.tables` and `pg_catalog.pg_class`.

**Beginner-Friendly Explanation**
A table is the spreadsheet inside your database—columns define what kind of information you store, and rows hold the actual records.

#### Purposes

- To provide the primary persistent storage structure for relational data.
- To enforce column-level and table-level data integrity through constraints.
- To serve as the foundation for views, indexes, triggers, and foreign-key relationships.
- To allow data to be queried, joined, and aggregated using SQL.

#### Syntax Rules and Structure

**General syntax (PostgreSQL, simplified):**

```sql
CREATE TABLE [ IF NOT EXISTS ] table_name (
    column_name data_type [ column_constraint [ ... ] ]
    [, ...]
    [, table_constraint ]
);
```


**MySQL (simplified):**

```sql
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name (
    column_name data_type [column_constraint],
    ...
) [table_options];
```


**SQL Server (simplified):**

```sql
CREATE TABLE [ database_name.[schema_name]. | schema_name. ] table_name (
    column_name data_type [column_constraint],
    ...
);
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `table_name` | Unique identifier within the schema. |
| `column_name data_type` | Each column's name and type (e.g., `integer`, `varchar(100)`, `timestamptz`). |
| `column_constraint` | `NOT NULL`, `DEFAULT`, `PRIMARY KEY`, `UNIQUE`, `CHECK`, `REFERENCES`. |
| `table_constraint` | Composite primary keys, foreign keys, unique constraints spanning multiple columns. |
| `IF NOT EXISTS` | Suppresses the error if the table already exists (PostgreSQL, MySQL). |

**Syntax rules:**

- A table must have at least one column; column names must be unique within the table.
- `PRIMARY KEY` implies `NOT NULL` and `UNIQUE`; only one primary key is allowed per table.
- In MySQL, `AUTO_INCREMENT` columns must be indexed; in SQL Server, `IDENTITY` columns serve a similar purpose.

**Constraints and limitations:**

- Table names must be unique within a schema; in MySQL, schema and database are synonymous.
- `CREATE TABLE` in PostgreSQL cannot run inside a transaction block if it includes certain object definitions that require their own transaction.
- The maximum number of columns varies by vendor (PostgreSQL: 1600; MySQL: 4096; SQL Server: 1024).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Create a table with a primary key, default, and check constraint

```sql
-- Step 1: Create a table for employee records.
CREATE TABLE employees (
    employee_id serial PRIMARY KEY,
    first_name  text NOT NULL,
    last_name   text NOT NULL,
    salary      numeric(10,2) CHECK (salary > 0),
    hired_at    timestamptz DEFAULT now()
);
```

**Expected output:**

```text
CREATE TABLE
```

**Why this output appears**: PostgreSQL creates the table and returns the command tag `CREATE TABLE`. `serial` creates an implicit sequence and default; `PRIMARY KEY` enforces uniqueness and not-null; `CHECK` ensures salary is positive; `DEFAULT now()` sets the hiring timestamp automatically.

##### Example 2: MySQL – Create a table with an auto-increment primary key

```sql
-- Step 1: Create a products table.
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    sku        VARCHAR(64) NOT NULL UNIQUE,
    name       VARCHAR(255) NOT NULL,
    price      DECIMAL(10,2) NOT NULL DEFAULT 0.00
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
```

**Why this output appears**: MySQL creates the table with the InnoDB engine and `utf8mb4` character set. `AUTO_INCREMENT` generates sequential integers; `UNIQUE` on `sku` prevents duplicates.

#### Real-World Cases

- **Application schema initialisation**: Migration scripts use `CREATE TABLE` to define the initial schema for a new application.
- **Staging tables**: ETL pipelines create staging tables with the same structure as target tables to load and validate data before merging.
- **Audit logging**: An append-only audit table is created with a `DEFAULT now()` timestamp and restricted permissions for application roles.

---

### 2. Schemas

#### Definitions

**Core Definition**
A schema is a named namespace that groups related database objects such as tables, views, indexes, and functions.

**Technical Definition**
A schema is a persistent, named descriptor that owns zero or more schema objects and is itself owned by an authorization identifier. Within a schema, object names must be unique; across schemas, identical names are permitted. In PostgreSQL, `CREATE SCHEMA` can optionally include `CREATE TABLE`, `CREATE VIEW`, `CREATE INDEX`, `CREATE SEQUENCE`, `CREATE TRIGGER`, and `GRANT` statements, creating the schema and its initial objects atomically. In MySQL, `CREATE SCHEMA` is a synonym for `CREATE DATABASE`; in SQL Server, `CREATE SCHEMA` creates a schema within the current database and can also create tables and views and grant permissions in the same batch.

**Beginner-Friendly Explanation**
A schema is like a folder inside a database. You put related tables and other objects in the same folder so they stay organised and do not clash with objects in other folders.

#### Purposes

- To organise database objects into logical groups for manageability.
- To prevent naming collisions when different modules use similar object names.
- To provide a granularity for privilege management below the database level.
- To support multi-schema application design and tenant isolation.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
CREATE SCHEMA schema_name [ AUTHORIZATION role_specification ]
    [ schema_element [ ... ] ];
```


**MySQL:**

```sql
CREATE SCHEMA [IF NOT EXISTS] schema_name
    [CHARACTER SET charset_name]
    [COLLATE collation_name];
```


**SQL Server:**

```sql
CREATE SCHEMA schema_name_clause [ <schema_element> [ ...n ] ]
-- where schema_name_clause can be schema_name, AUTHORIZATION owner_name,
-- or schema_name AUTHORIZATION owner_name
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `schema_name` | Unique identifier within the database; cannot begin with `pg_` in PostgreSQL. |
| `AUTHORIZATION` | Specifies the owning role or principal. |
| `schema_element` | Optional `CREATE TABLE`, `CREATE VIEW`, `GRANT`, etc., executed within the new schema. |
| `CHARACTER SET`, `COLLATE` (MySQL) | Default character set and collation for the schema. |

**Syntax rules:**

- In PostgreSQL, `CREATE SCHEMA` must be the first statement in a transaction if it includes schema elements.
- In SQL Server, `CREATE SCHEMA` must be its own batch.
- `IF NOT EXISTS` prevents errors when the schema already exists.

**Constraints and limitations:**

- Schemas cannot be nested.
- System schemas (`pg_catalog`, `information_schema`) are reserved and cannot be created by users.
- In MySQL, `CREATE SCHEMA` and `CREATE DATABASE` are identical; there is no separate schema layer.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Create a schema with an initial table

```sql
-- Step 1: Create a schema "app" and a table within it in one statement.
CREATE SCHEMA app
    AUTHORIZATION app_owner
    CREATE TABLE users (
        id serial PRIMARY KEY,
        username text NOT NULL UNIQUE,
        created_at timestamptz DEFAULT now()
    );
```

**Expected output:**

```text
CREATE SCHEMA
```

**Why this output appears**: PostgreSQL creates the schema, assigns ownership to `app_owner`, and creates the `users` table inside the new schema atomically. The command tag is `CREATE SCHEMA` regardless of the number of elements created.

##### Example 2: MySQL – Create a schema (database) with a specific character set

```sql
-- Step 1: Create a schema using utf8mb4.
CREATE SCHEMA IF NOT EXISTS analytics
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

-- Step 2: Verify.
SHOW DATABASES LIKE 'analytics';
```

**Expected output:**

```text
Query OK, 1 row affected (0.01 sec)

+----------------------+
| Database (analytics) |
+----------------------+
| analytics            |
+----------------------+
1 row in set (0.00 sec)
```

**Why this output appears**: In MySQL, `CREATE SCHEMA` is a synonym for `CREATE DATABASE`; the schema is implemented as a directory containing table files. `SHOW DATABASES` confirms its creation.

#### Real-World Cases

- **Modular application**: A monolith is decomposed into schemas such as `auth`, `billing`, and `reporting`.
- **Third-party integration**: A vendor's tables are placed in a dedicated schema (`vendor_acme`) to avoid name collisions.
- **Multi-tenant provisioning**: Each tenant receives a schema created with `AUTHORIZATION tenant_role` and a templated set of tables.

---

### 3. Views

#### Definitions

**Core Definition**
A view is a named, stored query that behaves like a virtual table.

**Technical Definition**
A view is defined by a `SELECT` (or `VALUES`) statement and does not store data itself; the query is executed each time the view is referenced. In PostgreSQL, `CREATE VIEW` defines a view of a query; `CREATE OR REPLACE VIEW` replaces an existing view provided the new query generates the same columns with the same names and data types. MySQL views support `ALGORITHM` (merge or temptable), `DEFINER`, and `SQL SECURITY` clauses. SQL Server views can be schema-bound (`SCHEMABINDING`) and can be encrypted (`ENCRYPTION`).

**Beginner-Friendly Explanation**
A view is like a saved search. You define what you want to see once, and every time you open it, the database runs the search again and shows you the current results.

#### Purposes

- To simplify complex queries by encapsulating joins and filters behind a single name.
- To provide a security layer by exposing only selected columns or rows to certain users.
- To present a stable, backward-compatible interface even when underlying table structures change.
- To enable reusable, named query definitions across applications.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
CREATE [ OR REPLACE ] [ TEMP | TEMPORARY ] VIEW name [ ( column_name [, ...] ) ]
    AS query
    [ WITH [ CASCADED | LOCAL ] CHECK OPTION ];
```


**MySQL:**

```sql
CREATE [OR REPLACE]
    [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
    [DEFINER = user]
    [SQL SECURITY { DEFINER | INVOKER }]
    VIEW view_name [(column_list)]
    AS select_statement
    [WITH [CASCADED | LOCAL] CHECK OPTION];
```


**SQL Server:**

```sql
CREATE [ OR ALTER ] VIEW [ schema_name . ] view_name [ (column [ ,...n ] ) ]
    [ WITH <view_attribute> [ ,...n ] ]
    AS select_statement
    [ WITH CHECK OPTION ];
-- <view_attribute> can include ENCRYPTION, SCHEMABINDING, VIEW_METADATA
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `view_name` | Unique identifier within the schema. |
| `(column_name, ...)` | Optional explicit column names for the view. |
| `AS query` | The `SELECT` statement that defines the view. |
| `OR REPLACE` / `OR ALTER` | Replaces an existing view definition. |
| `WITH CHECK OPTION` | Constrains inserts and updates to rows that satisfy the view's defining condition. |

**Syntax rules:**

- The view name must be distinct from all other relations in the same schema.
- The new query in `CREATE OR REPLACE VIEW` must generate the same columns (names, order, types) as the existing view; it may add new columns only at the end.
- In MySQL, `DEFINER` and `SQL SECURITY` control the privilege context used when the view is accessed.

**Constraints and limitations:**

- Views are not physically materialised (unless materialised views are used); performance depends on the underlying query.
- In MySQL, the view definition is "frozen" at creation time; adding columns to base tables later does not affect the view.
- Some views are not updatable (e.g., those with `DISTINCT`, aggregates, or `UNION`).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Create a simple view

```sql
-- Step 1: Create a base table.
CREATE TABLE sales (region text, amount numeric);

-- Step 2: Insert data.
INSERT INTO sales VALUES ('North', 100), ('South', 150);

-- Step 3: Create a view that filters high-value sales.
CREATE VIEW high_value_sales AS
SELECT region, amount
FROM sales
WHERE amount > 120;

-- Step 4: Query the view.
SELECT * FROM high_value_sales;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 2
CREATE VIEW
 region | amount
--------+--------
 South  |    150
(1 row)
```

**Why this output appears**: The view stores the query, not the data. When queried, PostgreSQL executes the `SELECT` against the current contents of `sales`, returning only rows with `amount > 120`.

##### Example 2: MySQL – Create a view with CHECK OPTION

```sql
-- Step 1: Create a table and a view.
CREATE TABLE t (qty INT, price INT);
INSERT INTO t VALUES (10, 100), (5, 200);

CREATE VIEW v AS
SELECT qty, price, qty * price AS value
FROM t
WHERE qty > 0
WITH CHECK OPTION;

-- Step 2: Query the view.
SELECT * FROM v;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 2 rows affected (0.01 sec)

+------+-------+-------+
| qty  | price | value |
+------+-------+-------+
|   10 |   100 |  1000 |
|    5 |   200 |  1000 |
+------+-------+-------+
```

**Why this output appears**: The view computes `value` as `qty * price`. `WITH CHECK OPTION` ensures that any `INSERT` or `UPDATE` through the view must satisfy `qty > 0`, preventing rows from being added that would not appear in the view.

#### Real-World Cases

- **Reporting**: A view joins `orders`, `customers`, and `products` into a single denormalised reporting table for analysts.
- **Security**: A view exposes only non-sensitive columns (e.g., employee name and department but not salary) to a restricted role.
- **Backward compatibility**: A view presents an old table structure while the underlying tables are refactored into a new normalised form.

---

### 4. Indexes

#### Definitions

**Core Definition**
An index is a database object that accelerates the retrieval of rows from a table based on the values of one or more columns.

**Technical Definition**
An index is a auxiliary data structure—typically a B-tree, hash, GiST, GIN, or BRIN structure—that stores a sorted or hashed representation of key values and pointers to the corresponding table rows. PostgreSQL supports B-tree, hash, GiST, SP-GiST, GIN, and BRIN index methods. MySQL supports B-tree and hash indexes, with `UNIQUE`, `FULLTEXT`, and `SPATIAL` variants. SQL Server supports clustered and nonclustered rowstore indexes, as well as columnstore indexes.

**Beginner-Friendly Explanation**
An index is like the index at the back of a book. Instead of reading every page to find a topic, you look up the topic in the index and jump straight to the right page.

#### Purposes

- To speed up `SELECT` queries that filter or sort on indexed columns.
- To enforce uniqueness through unique indexes and primary-key indexes.
- To support efficient join operations by indexing foreign-key columns.
- To enable specialised query patterns such as full-text search (MySQL `FULLTEXT`) or spatial queries (`SPATIAL`).

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
CREATE [ UNIQUE ] INDEX [ CONCURRENTLY ] [ IF NOT EXISTS ] index_name
    ON table_name [ USING method ]
    ( { column_name | ( expression ) } [ ASC | DESC ] [ NULLS { FIRST | LAST } ] [, ...] )
    [ WHERE predicate ];
```


**MySQL:**

```sql
CREATE [UNIQUE | FULLTEXT | SPATIAL] INDEX index_name
    [index_type]
    ON tbl_name (key_part,...)
    [index_option]
    [algorithm_option | lock_option];
-- key_part: {col_name [(length)] | (expr)} [ASC | DESC]
```


**SQL Server:**

```sql
CREATE [ UNIQUE ] [ CLUSTERED | NONCLUSTERED ] INDEX index_name
    ON <object> ( column [ ASC | DESC ] [ ,...n ] )
    [ INCLUDE ( column_name [ ,...n ] ) ]
    [ WHERE <filter_predicate> ]
    [ WITH ( <relational_index_option> [ ,...n ] ) ];
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `index_name` | Unique identifier for the index. |
| `ON table_name (columns)` | The table and columns to index. |
| `USING method` | Index method: `btree` (default), `hash`, `gist`, `gin`, `brin` (PostgreSQL). |
| `UNIQUE` | Enforces uniqueness of the indexed values. |
| `CLUSTERED` / `NONCLUSTERED` | SQL Server: clustered indexes determine physical row order; nonclustered indexes are separate structures. |
| `CONCURRENTLY` (PostgreSQL) | Builds the index without blocking writes. |

**Syntax rules:**

- The columns referenced in an index must exist in the table.
- In PostgreSQL, `CREATE INDEX CONCURRENTLY` cannot run inside a transaction block.
- MySQL maps `CREATE INDEX` to an `ALTER TABLE` operation internally.

**Constraints and limitations:**

- Indexes consume disk space and slow down `INSERT`, `UPDATE`, and `DELETE` operations.
- Too many indexes on a table can degrade write performance.
- `UNIQUE` indexes fail if duplicate values already exist in the data.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Create a B-tree index for query acceleration

```sql
-- Step 1: Create a table and insert data.
CREATE TABLE orders (
    order_id serial PRIMARY KEY,
    customer_id int NOT NULL,
    order_date date NOT NULL
);

-- Step 2: Create an index on customer_id.
CREATE INDEX idx_orders_customer ON orders (customer_id);

-- Step 3: Query using the indexed column; the planner uses the index.
EXPLAIN (COSTS OFF)
SELECT * FROM orders WHERE customer_id = 42;
```

**Expected output:**

```text
CREATE TABLE
CREATE INDEX
                 QUERY PLAN
--------------------------------------------
 Index Scan using idx_orders_customer on orders
   Index Cond: (customer_id = 42)
(2 rows)
```

**Why this output appears**: The planner recognises that the `WHERE` clause filters on `customer_id`, which is indexed, and chooses an index scan instead of a sequential scan.

##### Example 2: MySQL – Create a unique index

```sql
-- Step 1: Create a table.
CREATE TABLE users (id INT, email VARCHAR(255));

-- Step 2: Create a unique index on email.
CREATE UNIQUE INDEX idx_email ON users (email);

-- Step 3: Attempt to insert duplicate emails.
INSERT INTO users VALUES (1, 'a@example.com');
INSERT INTO users VALUES (2, 'a@example.com');
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 0 rows affected (0.01 sec)
ERROR 1062 (23000): Duplicate entry 'a@example.com' for key 'idx_email'
```

**Why this output appears**: The unique index enforces that no two rows can share the same email. The second insert violates the constraint and raises an error.

#### Real-World Cases

- **Primary-key lookups**: Every `PRIMARY KEY` automatically creates an index, enabling fast row retrieval by ID.
- **Foreign-key joins**: Indexing foreign-key columns speeds up join operations between parent and child tables.
- **Reporting filters**: An index on `order_date` accelerates queries that filter by date range in a data warehouse.

---

### 5. Sequences

#### Definitions

**Core Definition**
A sequence is a database object that generates a sequence of numeric values according to specified rules (start, increment, minimum, maximum, cycle).

**Technical Definition**
`CREATE SEQUENCE` creates a special single-row table that acts as a number generator. In PostgreSQL, sequences are independent objects that can be owned by a table column (via `OWNED BY`) or used with `serial`/`identity` columns. SQL Server sequences are schema-bound objects that generate numbers independently of any table and are accessed via `NEXT VALUE FOR`. MySQL does not support standalone sequences; instead, it uses the `AUTO_INCREMENT` column property to generate sequential values, and sequences can be simulated with stored procedures or triggers.

**Beginner-Friendly Explanation**
A sequence is like a ticket dispenser at a deli: every time you pull a ticket, it hands you the next number in line. You can set it to start at a certain number, skip numbers, or wrap around when it reaches the end.

#### Purposes

- To generate unique numeric identifiers for primary-key columns.
- To provide a shared number generator across multiple tables or sessions.
- To control the start, increment, and cycling behaviour of generated values.
- To decouple number generation from row insertion (SQL Server `NEXT VALUE FOR`).

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
CREATE [ { TEMPORARY | TEMP } | UNLOGGED ] SEQUENCE [ IF NOT EXISTS ] name
    [ AS data_type ]
    [ INCREMENT [ BY ] increment ]
    [ MINVALUE minvalue | NO MINVALUE ]
    [ MAXVALUE maxvalue | NO MAXVALUE ]
    [ [ NO ] CYCLE ]
    [ START [ WITH ] start ]
    [ CACHE cache ]
    [ OWNED BY { table_name.column_name | NONE } ];
```


**SQL Server:**

```sql
CREATE SEQUENCE [ schema_name . ] sequence_name
    [ AS [ built_in_integer_type | user-defined_integer_type ] ]
    [ START WITH <constant> ]
    [ INCREMENT BY <constant> ]
    [ { MINVALUE [ <constant> ] } | { NO MINVALUE } ]
    [ { MAXVALUE [ <constant> ] } | { NO MAXVALUE } ]
    [ CYCLE | { NO CYCLE } ]
    [ { CACHE [ <constant> ] } | { NO CACHE } ];
```


**MySQL (equivalent via AUTO_INCREMENT):**

```sql
CREATE TABLE table_name (
    id INT AUTO_INCREMENT PRIMARY KEY,
    ...
);
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `INCREMENT BY` | Value added to the current sequence value to produce the next value. |
| `MINVALUE` / `MAXVALUE` | Bounds for the sequence; `NO MINVALUE` / `NO MAXVALUE` use type defaults. |
| `CYCLE` | Restarts from `MINVALUE` after reaching `MAXVALUE` (or vice versa). |
| `CACHE` | Pre-allocates a block of sequence numbers for performance. |
| `OWNED BY` (PostgreSQL) | Associates the sequence with a table column; dropping the column drops the sequence. |

**Syntax rules:**

- The sequence name must be unique within its schema.
- `START WITH` must be within the bounds defined by `MINVALUE` and `MAXVALUE`.
- In PostgreSQL, `nextval()`, `currval()`, and `setval()` operate on sequences.

**Constraints and limitations:**

- MySQL does not support `CREATE SEQUENCE`; `AUTO_INCREMENT` is tied to a single table column and cannot be shared across tables.
- Sequences do not guarantee gap-free numbering; rollbacks and caching can create gaps.
- `CYCLE` can cause duplicate values if the sequence wraps around; use with caution for primary keys.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Create a sequence and use it

```sql
-- Step 1: Create a sequence starting at 100, incrementing by 10.
CREATE SEQUENCE order_seq
    START WITH 100
    INCREMENT BY 10;

-- Step 2: Retrieve the next values.
SELECT nextval('order_seq'), nextval('order_seq');
```

**Expected output:**

```text
CREATE SEQUENCE
 nextval | nextval
---------+---------
     100 |     110
(1 row)
```

**Why this output appears**: `nextval()` advances the sequence and returns the next value. The first call returns 100 (the start value), and the second call adds the increment (10), returning 110.

##### Example 2: SQL Server – Create a sequence and use NEXT VALUE FOR

```sql
-- Step 1: Create a sequence.
CREATE SEQUENCE dbo.OrderNumbers
    AS INT
    START WITH 1
    INCREMENT BY 1;

-- Step 2: Use the sequence in an INSERT.
CREATE TABLE Orders (OrderID INT, OrderDate DATE);
INSERT INTO Orders (OrderID, OrderDate)
VALUES (NEXT VALUE FOR dbo.OrderNumbers, GETDATE());
```

**Expected output:**

```text
Commands completed successfully.

(1 row affected)
```

**Why this output appears**: `NEXT VALUE FOR` retrieves the next sequence number and uses it directly in the `INSERT` statement, decoupling number generation from the table's identity property.

#### Real-World Cases

- **Primary-key generation**: A sequence supplies unique `order_id` values across multiple tables or sessions.
- **Invoice numbering**: A sequence generates sequential invoice numbers with a specific prefix and increment.
- **Shared counters**: SQL Server sequences can be referenced by multiple tables, ensuring a single monotonic counter across the database.

---

### 6. Constraints

#### Definitions

**Core Definition**
A constraint is a rule attached to a table or column that restricts the values that can be stored, ensuring data integrity.

**Technical Definition**
Constraints are declarative rules enforced by the database engine. The principal types are `NOT NULL` (column must have a value), `PRIMARY KEY` (unique row identifier, implying `NOT NULL` and `UNIQUE`), `UNIQUE` (no duplicate values in the specified columns), `FOREIGN KEY` (values must exist in a referenced table's primary or unique key), and `CHECK` (values must satisfy a Boolean expression). Constraints can be defined inline with column definitions or as table-level constraints spanning multiple columns.

**Beginner-Friendly Explanation**
Constraints are the rules of the game. They stop you from entering invalid data—like a missing name, a duplicate email address, or a salary that is less than zero.

#### Purposes

- To enforce entity integrity by ensuring every row is uniquely identifiable.
- To enforce referential integrity by preventing orphaned foreign-key values.
- To enforce domain integrity by restricting column values to a valid range or pattern.
- To provide a declarative, centralised definition of business rules that all applications must respect.

#### Syntax Rules and Structure

Constraints can be defined at the column level or the table level:

```sql
-- Column-level constraints
CREATE TABLE table_name (
    column_name data_type [ CONSTRAINT constraint_name ] { NOT NULL | UNIQUE | PRIMARY KEY | CHECK (condition) | REFERENCES other_table(column) }
);

-- Table-level constraints
CREATE TABLE table_name (
    ...,
    [ CONSTRAINT constraint_name ] { PRIMARY KEY (column, ...) | UNIQUE (column, ...) | FOREIGN KEY (column) REFERENCES other_table(column) | CHECK (condition) }
);
```


**Component breakdown:**

| Constraint | Description |
|------------|-------------|
| `NOT NULL` | The column cannot contain `NULL`. |
| `PRIMARY KEY` | Uniquely identifies each row; implies `NOT NULL` and `UNIQUE`. |
| `UNIQUE` | No two rows may have the same value(s) in the specified column(s). |
| `FOREIGN KEY` | Values must exist in the referenced table's primary or unique key. |
| `CHECK` | Values must satisfy a Boolean expression. |

**Syntax rules:**

- A table can have only one `PRIMARY KEY`; it may span multiple columns (composite key).
- `FOREIGN KEY` constraints can reference only columns that are `PRIMARY KEY` or `UNIQUE` in the referenced table.
- `CHECK` constraints cannot reference other tables or use subqueries.

**Constraints and limitations:**

- Adding a constraint to a table with existing data fails if the data violates the constraint.
- `FOREIGN KEY` constraints can cause insert/update failures if the referenced row does not exist; `ON DELETE CASCADE` and `ON UPDATE CASCADE` modify this behaviour.
- MySQL and SQL Server support `CHECK` constraints; older MySQL versions ignored them.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Table with primary key, foreign key, and check constraints

```sql
-- Step 1: Create a parent table.
CREATE TABLE departments (
    dept_id serial PRIMARY KEY,
    dept_name text NOT NULL UNIQUE
);

-- Step 2: Create a child table with a foreign key and check constraint.
CREATE TABLE employees (
    emp_id serial PRIMARY KEY,
    emp_name text NOT NULL,
    salary numeric CHECK (salary > 0),
    dept_id int REFERENCES departments(dept_id)
);
```

**Expected output:**

```text
CREATE TABLE
CREATE TABLE
```

**Why this output appears**: The `employees` table has a primary key (`emp_id`), a `NOT NULL` constraint on `emp_name`, a `CHECK` constraint ensuring positive salaries, and a `FOREIGN KEY` referencing `departments(dept_id)`. All constraints are registered in the system catalogs.

##### Example 2: MySQL – Demonstrate a foreign-key violation

```sql
-- Step 1: Create tables with a foreign key.
CREATE TABLE parent (id INT PRIMARY KEY);
CREATE TABLE child (id INT PRIMARY KEY, parent_id INT, FOREIGN KEY (parent_id) REFERENCES parent(id));

-- Step 2: Attempt to insert a child with a non-existent parent.
INSERT INTO child VALUES (1, 999);
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 0 rows affected (0.02 sec)
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails
```

**Why this output appears**: The `FOREIGN KEY` constraint requires that `parent_id` values exist in `parent(id)`. Since `999` does not exist, MySQL rejects the insert.

#### Real-World Cases

- **User registration**: A `UNIQUE` constraint on `email` prevents duplicate accounts.
- **Order processing**: A `FOREIGN KEY` from `order_items.product_id` to `products.id` ensures that orders cannot reference non-existent products.
- **Financial data**: A `CHECK` constraint on `amount > 0` prevents negative transactions.

---

### 7. Triggers

#### Definitions

**Core Definition**
A trigger is a database object that automatically executes a specified function or procedure in response to certain events on a table or view.

**Technical Definition**
A trigger is associated with a table, view, or foreign table and fires `BEFORE`, `AFTER`, or `INSTEAD OF` an `INSERT`, `UPDATE`, `DELETE`, or `TRUNCATE` event. It can execute once per row (`FOR EACH ROW`) or once per statement (`FOR EACH STATEMENT`). In PostgreSQL, the trigger executes a user-defined function that returns `trigger`. MySQL triggers are `BEFORE` or `AFTER` `INSERT`, `UPDATE`, or `DELETE` on a table, with `FOR EACH ROW` semantics. SQL Server supports DML triggers, DDL triggers, and logon triggers, each with its own syntax and scope.

**Beginner-Friendly Explanation**
A trigger is an automatic action that happens when something changes in a table. For example, every time a new employee is hired, a trigger could automatically add a record to an audit log.

#### Purposes

- To enforce complex business rules that cannot be expressed with standard constraints.
- To maintain audit trails by logging changes to sensitive tables.
- To automatically update derived or denormalised data when base data changes.
- To validate or transform data before it is written (in `BEFORE` triggers).

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
CREATE [ OR REPLACE ] [ CONSTRAINT ] TRIGGER name
    { BEFORE | AFTER | INSTEAD OF } { event [ OR ... ] }
    ON table_name
    [ FOR [ EACH ] { ROW | STATEMENT } ]
    [ WHEN ( condition ) ]
    EXECUTE { FUNCTION | PROCEDURE } function_name ( arguments );
```


**MySQL:**

```sql
CREATE [DEFINER = user] TRIGGER [IF NOT EXISTS] trigger_name
    trigger_time trigger_event
    ON tbl_name
    FOR EACH ROW
    [trigger_order]
    trigger_body;
-- trigger_time: { BEFORE | AFTER }
-- trigger_event: { INSERT | UPDATE | DELETE }
```


**SQL Server (DML trigger):**

```sql
CREATE [ OR ALTER ] TRIGGER [ schema_name . ] trigger_name
    ON { table | view }
    { FOR | AFTER | INSTEAD OF }
    { [ INSERT ] [ , ] [ UPDATE ] [ , ] [ DELETE ] }
    AS { sql_statement [ ; ] [ , ...n ] };
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `BEFORE` / `AFTER` / `INSTEAD OF` | Timing of trigger execution relative to the event. |
| `event` | `INSERT`, `UPDATE`, `DELETE`, or `TRUNCATE`. |
| `FOR EACH ROW` / `FOR EACH STATEMENT` | Whether the trigger fires per row or per statement. |
| `EXECUTE FUNCTION` (PostgreSQL) | The function that implements the trigger logic. |
| `trigger_body` (MySQL) | The SQL statements executed by the trigger. |

**Syntax rules:**

- In PostgreSQL, the trigger function must be created before the trigger and must return `trigger`.
- MySQL triggers cannot be associated with temporary tables or views.
- SQL Server `INSTEAD OF` triggers are commonly used on views to make them updatable.

**Constraints and limitations:**

- Triggers add overhead to every affected data-modification operation.
- Recursive or cascading triggers can cause unexpected behaviour or infinite loops.
- Trigger logic is invisible to application developers unless documented; this can lead to maintainability challenges.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Audit trigger

```sql
-- Step 1: Create an audit table.
CREATE TABLE audit_log (id serial, table_name text, operation text, logged_at timestamptz DEFAULT now());

-- Step 2: Create a trigger function.
CREATE FUNCTION log_audit() RETURNS trigger AS $$
BEGIN
    INSERT INTO audit_log (table_name, operation)
    VALUES (TG_TABLE_NAME, TG_OP);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Step 3: Attach the trigger to a table.
CREATE TABLE customers (id int PRIMARY KEY, name text);
CREATE TRIGGER customers_audit
    AFTER INSERT OR UPDATE OR DELETE ON customers
    FOR EACH ROW EXECUTE FUNCTION log_audit();

-- Step 4: Insert a row and verify the audit log.
INSERT INTO customers VALUES (1, 'Acme');
SELECT * FROM audit_log;
```

**Expected output:**

```text
CREATE TABLE
CREATE FUNCTION
CREATE TABLE
CREATE TRIGGER
INSERT 0 1
 id | table_name | operation |         logged_at
----+------------+-----------+----------------------------
  1 | customers  | INSERT    | 2026-11-13 10:00:00.123+00
(1 row)
```

**Why this output appears**: The `AFTER INSERT` trigger fires once for the inserted row, executing `log_audit()`, which inserts a record into `audit_log` with the table name and operation.

##### Example 2: MySQL – BEFORE INSERT trigger

```sql
-- Step 1: Create a table and a trigger that normalises email to lowercase.
CREATE TABLE users (id INT PRIMARY KEY, email VARCHAR(255));

CREATE TRIGGER normalize_email
BEFORE INSERT ON users
FOR EACH ROW
SET NEW.email = LOWER(NEW.email);

-- Step 2: Insert a row with mixed-case email.
INSERT INTO users VALUES (1, 'User@Example.COM');
SELECT * FROM users;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 0 rows affected (0.01 sec)
Query OK, 1 row affected (0.00 sec)

+----+------------------+
| id | email            |
+----+------------------+
|  1 | user@example.com |
+----+------------------+
```

**Why this output appears**: The `BEFORE INSERT` trigger modifies `NEW.email` to lowercase before the row is written, ensuring consistent email formatting regardless of how the application supplies the value.

#### Real-World Cases

- **Audit trails**: Triggers on `UPDATE` and `DELETE` log old and new values to an audit table for compliance.
- **Data synchronisation**: A trigger updates a denormalised summary table whenever a detail row changes.
- **Validation**: A `BEFORE INSERT` trigger rejects rows that violate complex business rules not expressible as `CHECK` constraints.

---

### 8. Stored Procedures

#### Definitions

**Core Definition**
A stored procedure is a named, precompiled collection of SQL and procedural statements stored in the database and executed via `CALL`.

**Technical Definition**
`CREATE PROCEDURE` defines a routine that can accept `IN`, `OUT`, and `INOUT` parameters, contain control-flow logic, transaction control, and exception handling, and be invoked with the `CALL` statement. In PostgreSQL, procedures can be written in `plpgsql`, `sql`, `c`, or other languages. MySQL procedures support `IN`, `OUT`, and `INOUT` parameters and are created with `CREATE PROCEDURE`; they are invoked with `CALL`. SQL Server procedures are created with `CREATE PROCEDURE` (or `CREATE PROC`) and can return output parameters and a status value.

**Beginner-Friendly Explanation**
A stored procedure is like a recipe saved in the database. Instead of writing out all the steps every time you want to cook a dish, you just say “make dish X” and the database follows the saved recipe.

#### Purposes

- To encapsulate multi-step business logic in a single, reusable database object.
- To reduce network traffic by executing multiple statements server-side.
- To enforce consistent data-modification rules across all applications.
- To provide a controlled interface for sensitive operations (e.g., financial transactions).

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
CREATE [ OR REPLACE ] PROCEDURE name (
    [ [ argmode ] [ argname ] argtype [ { DEFAULT | = } default_expr ] [, ...] ]
)
{ LANGUAGE lang_name
  | TRANSFORM { FOR TYPE type_name } [, ...]
  | [ EXTERNAL ] SECURITY INVOKER | [ EXTERNAL ] SECURITY DEFINER
  | SET configuration_parameter { TO value | = value | FROM CURRENT }
  | AS 'definition'
  | sql_body
} ...
```


**MySQL:**

```sql
CREATE [DEFINER = user] PROCEDURE [IF NOT EXISTS] sp_name (
    [ IN | OUT | INOUT ] param_name type [, ...]
)
[characteristic ...]
routine_body;
```


**SQL Server:**

```sql
CREATE [ OR ALTER ] { PROC | PROCEDURE } [schema_name.] procedure_name
    [ { @parameter_name [ type_schema_name. ] data_type }
      [ VARYING ] [ NULL ] [ = default ] [ OUT | OUTPUT | [READONLY] ] [ ,...n ] ]
    [ WITH <procedure_option> [ ,...n ] ]
    [ FOR REPLICATION ]
    AS { [ BEGIN ] sql_statement [;] [ ...n ] [ END ] };
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `name` / `sp_name` | The procedure's name. |
| `argmode` / `IN` / `OUT` / `INOUT` | Parameter mode: input, output, or both. |
| `LANGUAGE` (PostgreSQL) | The procedural language (e.g., `plpgsql`, `sql`). |
| `routine_body` (MySQL) | The SQL statements and procedural logic. |
| `AS ...` (SQL Server) | The body of the procedure. |

**Syntax rules:**

- In PostgreSQL, procedures are invoked with `CALL`; functions are invoked within expressions.
- MySQL procedures can contain `BEGIN ... END` blocks, `DECLARE` variables, and exception handlers.
- SQL Server procedures can return a status value (0 for success) and output parameters.

**Constraints and limitations:**

- Procedures cannot be used directly in `SELECT` expressions; use functions for that purpose.
- Debugging stored procedures can be more difficult than debugging application code.
- Procedures are database-specific and not easily portable across RDBMSs.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Procedure with IN and INOUT parameters

```sql
-- Step 1: Create a procedure that inserts data and returns a count.
CREATE PROCEDURE insert_and_count(a integer, b integer, INOUT total bigint DEFAULT 0)
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO tbl VALUES (a);
    INSERT INTO tbl VALUES (b);
    SELECT count(*) INTO total FROM tbl;
END;
$$;

-- Step 2: Call the procedure.
CALL insert_and_count(1, 2, NULL);
```

**Expected output:**

```text
CREATE PROCEDURE
 total
-------
     2
(1 row)
```

**Why this output appears**: The procedure inserts two rows into `tbl`, then counts all rows and assigns the count to the `INOUT` parameter `total`, which is returned to the caller.

##### Example 2: MySQL – Procedure with an OUT parameter

```sql
-- Step 1: Create a procedure.
DELIMITER //
CREATE PROCEDURE citycount (IN country CHAR(3), OUT cities INT)
BEGIN
    SELECT COUNT(*) INTO cities
    FROM city
    WHERE country_code = country;
END //
DELIMITER ;

-- Step 2: Call the procedure.
CALL citycount('USA', @city_count);
SELECT @city_count;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 0 rows affected (0.01 sec)

+------------+
| @city_count |
+------------+
|          42 |
+------------+
```

**Why this output appears**: The `OUT` parameter `cities` receives the result of the `COUNT(*)` query, and the caller retrieves it through the session variable `@city_count`.

#### Real-World Cases

- **Batch processing**: A nightly procedure archives old records, updates summary tables, and logs the operation.
- **Data migration**: A procedure transforms and copies data from a staging table to a production table with validation and error handling.
- **API backend**: An application calls a stored procedure to perform a multi-step transaction (e.g., transfer funds between accounts) atomically.

---

### 9. Functions

#### Definitions

**Core Definition**
A function is a named routine that accepts parameters, performs a calculation or query, and returns a value (scalar or table).

**Technical Definition**
`CREATE FUNCTION` defines a user-defined function (UDF) that can be invoked within SQL expressions, `SELECT` statements, and other routines. PostgreSQL functions can return scalar values, composite types, or tables (`RETURNS TABLE`), and can be written in SQL, `plpgsql`, C, or other languages. MySQL functions return a single scalar value and are created with `CREATE FUNCTION ... RETURNS type`; they are invoked directly in expressions. SQL Server functions can be scalar, inline table-valued, or multi-statement table-valued, and are commonly used in `SELECT` lists and `WHERE` clauses.

**Beginner-Friendly Explanation**
A function is like a calculator: you give it some inputs, it performs a calculation, and it gives you back a result. You can use functions inside queries wherever a value is expected.

#### Purposes

- To encapsulate reusable calculations that can be used in multiple queries.
- To extend SQL with custom logic that operates on data within a query context.
- To simplify complex expressions and improve query readability.
- To provide a controlled interface for data access in security-sensitive contexts.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
CREATE [ OR REPLACE ] FUNCTION name (
    [ [ argmode ] [ argname ] argtype [ { DEFAULT | = } default_expr ] [, ...] ]
)
[ RETURNS rettype | RETURNS TABLE ( column_name column_type [, ...] ) ]
{ LANGUAGE lang_name
  | TRANSFORM { FOR TYPE type_name } [, ...]
  | WINDOW
  | { IMMUTABLE | STABLE | VOLATILE }
  | [ NOT ] LEAKPROOF
  | { CALLED ON NULL INPUT | RETURNS NULL ON NULL INPUT | STRICT }
  | { [ EXTERNAL ] SECURITY INVOKER | [ EXTERNAL ] SECURITY DEFINER }
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


**MySQL:**

```sql
CREATE [DEFINER = user] FUNCTION [IF NOT EXISTS] sp_name (
    [func_parameter[,...]]
)
RETURNS type
[characteristic ...]
routine_body;
```


**SQL Server:**

```sql
-- Scalar function
CREATE [ OR ALTER ] FUNCTION [ schema_name. ] function_name (
    [ { @parameter_name [ AS ] [ type_schema_name. ] parameter_data_type
        [ NULL ] [ = default ] [ READONLY ] } [ ,...n ] ]
)
RETURNS return_data_type
[ WITH <function_option> [ ,...n ] ]
[ AS ]
BEGIN
    function_body
    RETURN scalar_expression
END;

-- Inline table-valued function
CREATE [ OR ALTER ] FUNCTION [ schema_name. ] function_name (
    [ { @parameter_name [ AS ] [ type_schema_name. ] parameter_data_type
        [ NULL ] [ = default ] [ READONLY ] } [ ,...n ] ]
)
RETURNS TABLE
[ WITH <function_option> [ ,...n ] ]
[ AS ]
RETURN [ ( ] select_stmt [ ) ];
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `RETURNS rettype` | The data type of the scalar value returned. |
| `RETURNS TABLE (...)` | Defines a table-returning function. |
| `LANGUAGE` | The implementation language (e.g., `plpgsql`, `sql`). |
| `IMMUTABLE` / `STABLE` / `VOLATILE` | Volatility category, affecting optimisation and indexing (PostgreSQL). |
| `RETURNS type` (MySQL) | The scalar return type. |
| `RETURNS TABLE` (SQL Server) | Inline table-valued function return type. |

**Syntax rules:**

- Functions must return a value; procedures use `OUT`/`INOUT` parameters instead.
- In PostgreSQL, `CREATE OR REPLACE FUNCTION` cannot change the return type; the function must be dropped and recreated.
- SQL Server scalar functions are invoked with `schema.function_name(arguments)`.

**Constraints and limitations:**

- Functions used in `WHERE` clauses or `SELECT` lists may be called once per row, which can be inefficient for large datasets.
- Functions that modify data (`VOLATILE` in PostgreSQL) cannot be used in certain optimisation contexts.
- MySQL stored functions and loadable functions share the same namespace.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Scalar function

```sql
-- Step 1: Create a function that calculates a discounted price.
CREATE FUNCTION discounted_price(price numeric, discount numeric)
RETURNS numeric
LANGUAGE sql
IMMUTABLE
AS $$
    SELECT price * (1 - discount);
$$;

-- Step 2: Use the function in a query.
SELECT discounted_price(100, 0.20) AS final_price;
```

**Expected output:**

```text
CREATE FUNCTION
 final_price
-------------
       80.00
(1 row)
```

**Why this output appears**: The function performs the arithmetic `100 * (1 - 0.20)` and returns `80.00`. The `IMMUTABLE` marker tells PostgreSQL that the function's result depends only on its arguments, enabling constant folding.

##### Example 2: SQL Server – Inline table-valued function

```sql
-- Step 1: Create an inline table-valued function.
CREATE FUNCTION dbo.GetEmployeesByDept (@DeptID INT)
RETURNS TABLE
AS
RETURN
(
    SELECT EmployeeID, FirstName, LastName
    FROM Employees
    WHERE DepartmentID = @DeptID
);

-- Step 2: Use the function in a query.
SELECT * FROM dbo.GetEmployeesByDept(10);
```

**Expected output:**

```text
(3 rows affected)

EmployeeID  FirstName  LastName
----------  ---------  --------
1           Alice      Smith
3           Bob        Jones
5           Carol      White
```

**Why this output appears**: The inline table-valued function returns the result of the embedded `SELECT` as a table, which can be queried with `SELECT * FROM function_name(arguments)`.

#### Real-World Cases

- **Calculated columns**: A function computes tax, shipping, or discounts and is used in `SELECT` lists across multiple queries.
- **Data validation**: A scalar function checks whether a value matches a complex pattern and is used in `CHECK` constraints or `WHERE` clauses.
- **Parameterised views**: A table-valued function accepts parameters and returns a filtered result set, serving as a parameterised alternative to views.

---

## References

- PostgreSQL Documentation – `CREATE TABLE` – https://www.postgresql.org/docs/current/sql-createtable.html
- PostgreSQL Documentation – `CREATE SCHEMA` – https://www.postgresql.org/docs/current/sql-createschema.html
- PostgreSQL Documentation – `CREATE VIEW` – https://www.postgresql.org/docs/current/sql-createview.html
- PostgreSQL Documentation – `CREATE INDEX` – https://www.postgresql.org/docs/current/sql-createindex.html
- PostgreSQL Documentation – `CREATE SEQUENCE` – https://www.postgresql.org/docs/current/sql-createsequence.html
- PostgreSQL Documentation – Constraints – https://www.postgresql.org/docs/current/ddl-constraints.html
- PostgreSQL Documentation – `CREATE TRIGGER` – https://www.postgresql.org/docs/current/sql-createtrigger.html
- PostgreSQL Documentation – `CREATE PROCEDURE` – https://www.postgresql.org/docs/current/sql-createprocedure.html
- PostgreSQL Documentation – `CREATE FUNCTION` – https://www.postgresql.org/docs/current/sql-createfunction.html
- MySQL 8.0 Reference Manual – `CREATE TABLE` – https://dev.mysql.com/doc/refman/8.0/en/create-table.html
- MySQL 8.0 Reference Manual – `CREATE SCHEMA` (synonym for `CREATE DATABASE`) – https://dev.mysql.com/doc/refman/8.0/en/create-database.html
- MySQL 8.0 Reference Manual – `CREATE VIEW` – https://dev.mysql.com/doc/refman/8.0/en/create-view.html
- MySQL 8.0 Reference Manual – `CREATE INDEX` – https://dev.mysql.com/doc/refman/8.0/en/create-index.html
- MySQL 8.0 Reference Manual – `CREATE TRIGGER` – https://dev.mysql.com/doc/refman/8.0/en/create-trigger.html
- MySQL 8.0 Reference Manual – `CREATE PROCEDURE` and `CREATE FUNCTION` – https://dev.mysql.com/doc/refman/8.0/en/create-procedure.html
- Microsoft Learn – `CREATE TABLE` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql
- Microsoft Learn – `CREATE SCHEMA` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/create-schema-transact-sql
- Microsoft Learn – `CREATE VIEW` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/create-view-transact-sql
- Microsoft Learn – `CREATE INDEX` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/create-index-transact-sql
- Microsoft Learn – `CREATE SEQUENCE` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/create-sequence-transact-sql
- Microsoft Learn – `CREATE TRIGGER` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/create-trigger-transact-sql
- Microsoft Learn – `CREATE PROCEDURE` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/create-procedure-transact-sql
- Microsoft Learn – `CREATE FUNCTION` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/create-function-transact-sql
- Oracle Database SQL Language Reference – `CREATE TABLE` – https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/CREATE-TABLE.html
- Oracle Database SQL Language Reference – Constraints – https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/constraint.html