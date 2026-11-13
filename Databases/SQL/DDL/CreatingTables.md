# Comprehensive Programming Cheat Sheet: SQL Table Creation Fundamentals

## Topic Overview

### Definitions

**Core Definition**
Creating tables is the Data Definition Language (DDL) process of defining a new relation in a database by specifying its name, columns, data types, constraints, default values, and storage or persistence characteristics.

**Technical Definition**
`CREATE TABLE` allocates a new relation in the system catalogs, registers column attributes (name, data type, collation, nullability, default), defines column‑level and table‑level constraints (primary key, unique, foreign key, check, exclusion), and optionally specifies generated or identity columns, storage parameters, partitioning strategy, and table persistence (`TEMPORARY`, `UNLOGGED`). The core statement is standardised, but syntax and available features vary across PostgreSQL, MySQL, and SQL Server.

**Beginner-Friendly Explanation**
Creating a table is like designing a spreadsheet. You decide what columns (categories of information) the spreadsheet will have, what type of data each column holds (numbers, text, dates), and what rules apply—such as “this column cannot be empty” or “this column must be unique.” You can also choose special kinds of spreadsheets that behave differently, like ones that disappear when you log out or ones that are split into smaller pieces for speed.

### Key Characteristics

- **Structure before data**: `CREATE TABLE` defines the container; data is added later with `INSERT`.
- **Constraint‑driven integrity**: Primary keys, foreign keys, `NOT NULL`, `UNIQUE`, and `CHECK` constraints enforce data quality at the table level.
- **Vendor divergence**: While `CREATE TABLE` is broadly standardised, features such as `UNLOGGED` (PostgreSQL), `AUTO_INCREMENT` (MySQL), `IDENTITY` (SQL Server), generated columns, and partitioning syntax are implementation‑specific.
- **Persistence options**: Tables can be permanent, temporary (session‑scoped), or unlogged (non‑crash‑safe in PostgreSQL).
- **Catalog visibility**: Every created table is recorded in system catalogs (`information_schema.tables`, `pg_catalog.pg_class`, `sys.tables`), enabling introspection and automation.

### Prerequisites

- A running RDBMS instance (PostgreSQL, MySQL, or SQL Server) and a client connection.
- An account with `CREATE` privilege on the target schema or database.
- Basic familiarity with SQL data types and constraint concepts.
- For identity columns and sequences: knowledge of numeric data types and auto‑generation behaviour.
- For partitioned tables: understanding of partitioning keys and range/list/hash strategies.

### Related Programming Areas

- Data Definition Language (DDL) – the umbrella category for `CREATE`, `ALTER`, and `DROP`.
- Data Manipulation Language (DML) – `INSERT`, `SELECT`, `UPDATE`, `DELETE` operate on the created tables.
- Query optimisation and indexing – table structure and constraints directly affect performance.
- Schema versioning and migration – table creation is the primary operation tracked by Flyway and Liquibase.
- Data modelling and normalisation – table design is the physical realisation of the logical data model.

---

## Core Concepts and Features

### 1. `CREATE TABLE`

#### Definitions

**Core Definition**
`CREATE TABLE` is the SQL statement that defines a new table in the current database.

**Technical Definition**
`CREATE TABLE` registers a new relation in the system catalogs, specifying its name, columns, data types, constraints, and optional storage or persistence attributes. In PostgreSQL, the statement supports `TEMPORARY`, `UNLOGGED`, `PARTITION BY`, `INHERITS`, and `LIKE` clauses. In MySQL, it supports `TEMPORARY`, `IF NOT EXISTS`, table options such as `ENGINE` and `CHARSET`, and partition options. In SQL Server, it supports `IDENTITY`, computed columns, filegroup placement, and partitioning via a partition scheme.

**Beginner-Friendly Explanation**
This is the command that creates a brand‑new spreadsheet. You tell the database what to call it, what columns it should have, what type of data each column holds, and any rules (like “this column cannot be empty” or “this column must be unique”).

#### Purposes

- To define the structure of a new relation before inserting data.
- To enforce data integrity through column and table constraints.
- To specify storage characteristics such as tablespace, filegroup, or partitioning strategy.
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
CREATE TABLE [ database_name.[schema_name]. | schema_name. ] table_name
    ( { <column_definition> | <computed_column_definition> } [ ,...n ] )
    [ ON { partition_scheme_name ( partition_column_name ) | filegroup | "default" } ]
    [ WITH ( <table_option> [ ,...n ] ) ];
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `table_name` | Identifier for the new table; can be schema‑qualified. |
| `column_name data_type` | Defines a column and its data type. |
| `column_constraint` | `NOT NULL`, `DEFAULT`, `CHECK`, `UNIQUE`, `PRIMARY KEY`, `REFERENCES`, identity/generated clauses. |
| `table_constraint` | Constraints spanning multiple columns (composite primary key, foreign key, exclusion). |
| `TEMPORARY` / `TEMP` | Creates a session‑scoped temporary table. |
| `UNLOGGED` (PostgreSQL) | Creates a table that bypasses WAL logging (non‑crash‑safe). |
| `PARTITION BY` | Declares the table as partitioned. |
| `ON COMMIT` (temporary tables) | Controls behaviour at transaction end. |
| `TABLESPACE` / `ON filegroup` | Specifies physical storage location. |

**Syntax rules:**

- A table must have at least one column; column names must be unique within the table.
- `PRIMARY KEY` implies `NOT NULL` and `UNIQUE`; only one primary key is allowed per table.
- In PostgreSQL, `CREATE TABLE` cannot run inside a transaction block if it includes certain object definitions that require their own transaction.
- In MySQL, `CREATE` privilege on the table is required.

**Constraints and limitations:**

- Table names must be unique within a schema (and within a database in MySQL, where schema and database are synonymous).
- Maximum column counts vary: PostgreSQL 1600, MySQL 4096, SQL Server 1024.
- `UNLOGGED` tables are PostgreSQL‑specific and are not crash‑safe.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Create a table with constraints and a default

```sql
-- Step 1: Create a table with a primary key, NOT NULL,
--         a DEFAULT, and a CHECK constraint.
CREATE TABLE employees (
    employee_id serial PRIMARY KEY,
    first_name  text NOT NULL,
    last_name   text NOT NULL,
    email       text UNIQUE,
    salary      numeric(10,2) CHECK (salary > 0),
    hired_at    timestamptz DEFAULT now()
);
```

**Expected output:**

```text
CREATE TABLE
```

**Why this output appears**: PostgreSQL creates the table and returns the command tag `CREATE TABLE`. `serial` creates an implicit sequence and default; `PRIMARY KEY` implies `NOT NULL` and `UNIQUE`; the `CHECK` constraint ensures salary is positive; `DEFAULT now()` sets the hiring timestamp automatically.

##### Example 2: MySQL – Create a table with an auto‑increment primary key

```sql
-- Step 1: Create a products table with AUTO_INCREMENT and a unique key.
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

##### Example 3: SQL Server – Create a table with an IDENTITY column

```sql
-- Step 1: Create a table with an IDENTITY column and a primary key.
CREATE TABLE customers (
    customer_id INT IDENTITY(1,1) PRIMARY KEY,
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

**Why this output appears**: SQL Server creates the table with an `IDENTITY(1,1)` column that auto‑generates values starting at 1 with an increment of 1. The `PRIMARY KEY` and `UNIQUE` constraints are enforced.

#### Real-World Cases

- **Application schema initialisation**: Migration scripts use `CREATE TABLE` to define the initial schema for a new application.
- **Data warehousing staging**: A staging table is created with the same structure as the target table to load and validate data before merging.
- **Audit logging**: An append‑only audit table is created with a `DEFAULT now()` timestamp and restricted permissions for application roles.

---

### 2. Column Definitions

#### Definitions

**Core Definition**
A column definition specifies the name, data type, and optional constraints and attributes of a single column within a table.

**Technical Definition**
A column definition consists of a column name, a data type, and zero or more column constraints (`NOT NULL`, `DEFAULT`, `CHECK`, `UNIQUE`, `PRIMARY KEY`, `REFERENCES`) and attributes (collation, identity, generated). The definition determines what values the column can hold and how those values are validated or generated.

**Beginner-Friendly Explanation**
A column definition is the blueprint for one column: it says “this column is called `email`, it holds text, it cannot be empty, and every value must be unique.”

#### Purposes

- To assign a name and data type to each attribute of a table.
- To enforce column‑level integrity through constraints.
- To specify default values and generation behaviour for columns.
- To control collation and storage attributes at the column level.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name data_type
    [ COLLATE collation ]
    [ column_constraint [ ... ] ]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `column_name` | Unique identifier within the table. |
| `data_type` | The type of values the column can hold (e.g., `integer`, `varchar(100)`, `timestamptz`). |
| `COLLATE` | Specifies the collation for character data (PostgreSQL, SQL Server). |
| `column_constraint` | `NOT NULL`, `DEFAULT`, `CHECK`, `UNIQUE`, `PRIMARY KEY`, `REFERENCES`, identity/generated clauses. |

**Syntax rules:**

- A column must have a data type; the type determines the valid range of values and the operations that can be performed.
- `NOT NULL` and `DEFAULT` are among the most common column attributes.
- In PostgreSQL, `COLLATE` can be specified at the column level to override the database default.

**Constraints and limitations:**

- The maximum column name length varies by vendor (PostgreSQL: 63 bytes; MySQL: 64 characters; SQL Server: 128 characters).
- Some data types are vendor‑specific (e.g., `SERIAL` in PostgreSQL, `AUTO_INCREMENT` in MySQL, `IDENTITY` in SQL Server).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Column with COLLATE and DEFAULT

```sql
-- Step 1: Create a table with a case-insensitive collation on a column.
CREATE TABLE users (
    user_id   serial PRIMARY KEY,
    username  text COLLATE "en_US.utf8" NOT NULL,
    created_at timestamptz DEFAULT now()
);
```

**Expected output:**

```text
CREATE TABLE
```

**Why this output appears**: The `COLLATE "en_US.utf8"` clause sets the collation for `username`, affecting sorting and comparison behaviour. `DEFAULT now()` provides a timestamp for new rows.

#### Real-World Cases

- **Internationalisation**: A column storing names in multiple languages uses a specific collation to ensure correct sorting.
- **Audit fields**: `created_at` and `updated_at` columns use `DEFAULT now()` to automatically record timestamps.

---

### 3. Data Types

#### Definitions

**Core Definition**
A data type defines the kind of values a column can hold, such as numbers, text, dates, or binary data.

**Technical Definition**
Data types determine the storage format, valid range of values, and permissible operations for a column. Common categories include numeric (`integer`, `numeric`, `decimal`), character (`char`, `varchar`, `text`), temporal (`date`, `time`, `timestamp`, `timestamptz`), boolean, binary (`bytea`, `varbinary`), JSON (`json`, `jsonb`), and UUID. PostgreSQL, MySQL, and SQL Server each provide a rich set of built‑in types, with some vendor‑specific extensions.

**Beginner-Friendly Explanation**
A data type is the “kind of thing” a column holds. A column for ages uses a number type; a column for names uses a text type; a column for dates uses a date type.

#### Purposes

- To constrain the values that can be stored in a column to a valid set.
- To determine the storage size and performance characteristics of a column.
- To enable type‑specific operations such as arithmetic, string concatenation, and date arithmetic.
- To ensure data consistency across rows and tables.

#### Syntax Rules and Structure

**Common data types:**

| Category | PostgreSQL | MySQL | SQL Server |
|----------|------------|-------|------------|
| Integer | `integer`, `bigint`, `smallint` | `INT`, `BIGINT`, `SMALLINT` | `INT`, `BIGINT`, `SMALLINT` |
| Decimal | `numeric(p,s)`, `decimal(p,s)` | `DECIMAL(p,s)` | `DECIMAL(p,s)` |
| Text | `text`, `varchar(n)`, `char(n)` | `VARCHAR(n)`, `TEXT`, `CHAR(n)` | `NVARCHAR(n)`, `VARCHAR(n)`, `CHAR(n)` |
| Date/Time | `date`, `time`, `timestamptz` | `DATE`, `TIME`, `DATETIME` | `DATE`, `TIME`, `DATETIME2` |
| Boolean | `boolean` | `BOOLEAN` (synonym `TINYINT(1)`) | `BIT` |
| JSON | `json`, `jsonb` | `JSON` | `NVARCHAR` with `ISJSON` check |
| UUID | `uuid` | `CHAR(36)` | `UNIQUEIDENTIFIER` |

**Syntax rules:**

- The data type must be specified for every column.
- Length or precision modifiers (e.g., `varchar(100)`, `numeric(10,2)`) constrain the size or precision.
- Some types have aliases (e.g., `int` = `integer`, `varchar` = `character varying`).

**Constraints and limitations:**

- Not all types are available in all RDBMSs (e.g., PostgreSQL `jsonb` has no direct MySQL or SQL Server equivalent).
- Using an inappropriate type can lead to data truncation or loss of precision.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Table using numeric, text, and temporal types

```sql
-- Step 1: Create a table with a variety of data types.
CREATE TABLE orders (
    order_id    serial PRIMARY KEY,
    customer_id integer NOT NULL,
    amount      numeric(10,2) NOT NULL,
    order_date  date NOT NULL DEFAULT CURRENT_DATE,
    notes       text,
    metadata    jsonb
);
```

**Expected output:**

```text
CREATE TABLE
```

**Why this output appears**: Each column is assigned an appropriate type: `integer` for IDs, `numeric(10,2)` for monetary amounts, `date` for dates, `text` for variable‑length strings, and `jsonb` for semi‑structured data.

#### Real-World Cases

- **Financial systems**: `numeric` or `decimal` is used for monetary values to avoid floating‑point rounding errors.
- **Content management**: `text` or `varchar` stores article bodies, titles, and metadata.
- **IoT telemetry**: `jsonb` (PostgreSQL) stores flexible sensor payloads that do not fit a fixed schema.

---

### 4. Column Constraints

#### Definitions

**Core Definition**
A column constraint is a rule attached to a single column that restricts the values that column can hold.

**Technical Definition**
Column constraints are declarative rules enforced by the database engine. The principal types are `NOT NULL` (the column cannot contain `NULL`), `DEFAULT` (a value is supplied if none is given), `CHECK` (values must satisfy a Boolean expression), `UNIQUE` (no duplicate values), `PRIMARY KEY` (unique row identifier, implying `NOT NULL` and `UNIQUE`), and `REFERENCES` (foreign key to another table). Constraints can be named for easier management.

**Beginner-Friendly Explanation**
A column constraint is a rule for one column. For example, “this column cannot be empty” (`NOT NULL`), “this column must be unique” (`UNIQUE`), or “this column must be greater than zero” (`CHECK`).

#### Purposes

- To enforce entity integrity by ensuring every row is uniquely identifiable.
- To enforce domain integrity by restricting column values to a valid range or pattern.
- To enforce referential integrity by linking columns to values in other tables.
- To provide a declarative, centralised definition of business rules.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name data_type [ CONSTRAINT constraint_name ] {
    NOT NULL
  | NULL
  | DEFAULT default_expr
  | CHECK ( expression )
  | UNIQUE
  | PRIMARY KEY
  | REFERENCES reftable [ ( refcolumn ) ]
}
```


**Component breakdown:**

| Constraint | Description |
|------------|-------------|
| `NOT NULL` | The column cannot contain `NULL`. |
| `DEFAULT` | Supplies a value when none is provided. |
| `CHECK` | Values must satisfy a Boolean expression. |
| `UNIQUE` | No two rows may have the same value. |
| `PRIMARY KEY` | Uniquely identifies each row; implies `NOT NULL` and `UNIQUE`. |
| `REFERENCES` | Values must exist in a referenced table’s primary or unique key. |

**Syntax rules:**

- `PRIMARY KEY` and `UNIQUE` automatically create indexes.
- `CHECK` constraints cannot reference other tables or use subqueries.
- Foreign‑key constraints require the referenced column to be `PRIMARY KEY` or `UNIQUE`.

**Constraints and limitations:**

- Adding a constraint to a table with existing data fails if the data violates the constraint.
- In MySQL, `CHECK` constraints were ignored before version 8.0.16.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Column constraints in action

```sql
-- Step 1: Create a table with NOT NULL, UNIQUE, CHECK, and DEFAULT constraints.
CREATE TABLE accounts (
    account_id serial PRIMARY KEY,
    email      text NOT NULL UNIQUE,
    balance    numeric(12,2) NOT NULL DEFAULT 0.00 CHECK (balance >= 0),
    status     text NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'closed'))
);
```

**Expected output:**

```text
CREATE TABLE
```

**Why this output appears**: `email` must be non‑null and unique; `balance` defaults to zero and cannot be negative; `status` defaults to `'active'` and is restricted to two values. All constraints are registered in the system catalogs.

##### Example 2: MySQL – Foreign‑key constraint violation

```sql
-- Step 1: Create parent and child tables.
CREATE TABLE parent (id INT PRIMARY KEY);
CREATE TABLE child (
    id INT PRIMARY KEY,
    parent_id INT,
    FOREIGN KEY (parent_id) REFERENCES parent(id)
);

-- Step 2: Attempt to insert a child with a non‑existent parent.
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
- **Order processing**: A `FOREIGN KEY` from `order_items.product_id` to `products.id` ensures orders cannot reference non‑existent products.
- **Financial data**: A `CHECK` constraint on `amount > 0` prevents negative transactions.

---

### 5. Table Constraints

#### Definitions

**Core Definition**
A table constraint is a rule that applies to one or more columns of a table and is defined at the table level rather than on individual columns.

**Technical Definition**
Table constraints are declared after all column definitions and can span multiple columns. They include composite `PRIMARY KEY`, composite `UNIQUE`, `FOREIGN KEY` (which must be a table constraint when referencing multiple columns), and `CHECK` constraints that reference multiple columns. `EXCLUDE` constraints (PostgreSQL) are also table‑level constraints that prevent overlapping values.

**Beginner-Friendly Explanation**
A table constraint is a rule for the table as a whole. For example, “the combination of `first_name` and `last_name` must be unique” is a table constraint because it involves two columns together.

#### Purposes

- To enforce uniqueness across a combination of columns.
- To define foreign‑key relationships that span multiple columns.
- To express check conditions that reference more than one column.
- To prevent overlapping ranges or values using exclusion constraints (PostgreSQL).

#### Syntax Rules and Structure

**General syntax:**

```sql
CREATE TABLE table_name (
    ...,
    [ CONSTRAINT constraint_name ] {
        PRIMARY KEY ( column_name [, ...] )
      | UNIQUE ( column_name [, ...] )
      | CHECK ( expression )
      | FOREIGN KEY ( column_name [, ...] ) REFERENCES reftable [ ( refcolumn [, ...] ) ]
      | EXCLUDE [ USING index_method ] ( exclude_element WITH operator [, ...] )
    }
);
```


**Component breakdown:**

| Constraint | Description |
|------------|-------------|
| `PRIMARY KEY (columns)` | Composite primary key spanning multiple columns. |
| `UNIQUE (columns)` | No two rows may have the same combination of values. |
| `CHECK (expression)` | Values must satisfy a Boolean expression that may reference multiple columns. |
| `FOREIGN KEY (columns)` | Values must exist in the referenced table’s key. |
| `EXCLUDE` | Prevents overlapping values (PostgreSQL). |

**Syntax rules:**

- A table can have only one primary key, but it may span multiple columns.
- Foreign keys referencing composite keys must also be composite.
- `CHECK` constraints can reference any column in the same row.

**Constraints and limitations:**

- Composite primary keys create composite indexes; the column order matters for index performance.
- `EXCLUDE` constraints are PostgreSQL‑specific and require the `btree_gist` or similar extension.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Composite primary key and multi‑column check

```sql
-- Step 1: Create a table with a composite primary key and a table-level check.
CREATE TABLE order_items (
    order_id   int,
    line_no    int,
    product_id int,
    quantity   int NOT NULL,
    unit_price numeric(10,2) NOT NULL,
    PRIMARY KEY (order_id, line_no),
    CHECK (quantity > 0 AND unit_price > 0)
);
```

**Expected output:**

```text
CREATE TABLE
```

**Why this output appears**: The primary key is the combination of `order_id` and `line_no`, ensuring that each line in an order is unique. The `CHECK` constraint references both `quantity` and `unit_price` and ensures both are positive.

##### Example 2: PostgreSQL – Foreign key spanning multiple columns

```sql
-- Step 1: Create a parent table with a composite unique key.
CREATE TABLE warehouses (
    company_id int,
    warehouse_id int,
    location text,
    UNIQUE (company_id, warehouse_id)
);

-- Step 2: Create a child table with a composite foreign key.
CREATE TABLE inventory (
    company_id int,
    warehouse_id int,
    product_id int,
    quantity int,
    FOREIGN KEY (company_id, warehouse_id)
        REFERENCES warehouses (company_id, warehouse_id)
);
```

**Expected output:**

```text
CREATE TABLE
CREATE TABLE
```

**Why this output appears**: The child table’s foreign key references the composite unique key in the parent table, ensuring that every inventory row belongs to a valid warehouse.

#### Real-World Cases

- **Order line items**: A composite primary key of `(order_id, line_no)` ensures each line is unique within an order.
- **Warehouse inventory**: A composite foreign key links inventory to a specific warehouse within a company.
- **Temporal data**: A table constraint ensures `end_date > start_date` for validity periods.

---

### 6. Default Values

#### Definitions

**Core Definition**
A default value is the value automatically assigned to a column when no value is provided during an `INSERT`.

**Technical Definition**
`DEFAULT` is a column attribute that specifies a literal value, expression, or function call evaluated at insert time when the column is omitted from the `INSERT` statement or explicitly set to `DEFAULT`. Defaults can reference functions (`now()`, `CURRENT_DATE`, `nextval()`), expressions, or constants. In PostgreSQL, the default expression is stored in the catalog and evaluated at insert time.

**Beginner-Friendly Explanation**
A default value is the “automatic fill‑in” for a column. If you do not provide a value, the database uses the default you specified—like automatically stamping today’s date on a new record.

#### Purposes

- To supply sensible values for columns that are often left blank.
- To automatically record timestamps for auditing or tracking.
- To generate identifiers using sequences or identity behaviour.
- To ensure that columns always have a value even when the application omits them.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name data_type DEFAULT default_expr
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `default_expr` | A literal, expression, or function call (e.g., `0`, `'active'`, `now()`, `nextval('seq')`). |

**Syntax rules:**

- The default expression must be compatible with the column’s data type.
- In PostgreSQL, `DEFAULT` can reference a sequence via `nextval()`; in modern versions, `GENERATED ... AS IDENTITY` is preferred for identity columns.
- In MySQL, `DEFAULT` cannot be used with `TEXT` or `BLOB` columns in some versions.

**Constraints and limitations:**

- Defaults are evaluated at insert time; they are not stored as part of the row.
- Changing a column’s default does not affect existing rows.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Default values with functions

```sql
-- Step 1: Create a table with defaults for timestamps and status.
CREATE TABLE tasks (
    task_id    serial PRIMARY KEY,
    title      text NOT NULL,
    status     text NOT NULL DEFAULT 'pending',
    created_at timestamptz DEFAULT now(),
    due_date   date DEFAULT CURRENT_DATE + 7
);
```

**Expected output:**

```text
CREATE TABLE
```

**Why this output appears**: When a row is inserted without specifying `status`, `created_at`, or `due_date`, the database automatically uses `'pending'`, the current timestamp, and the current date plus 7 days, respectively.

#### Real-World Cases

- **Audit trails**: `created_at` and `updated_at` columns use `DEFAULT now()` to automatically timestamp inserts and updates.
- **Workflow systems**: A `status` column defaults to `'pending'` so new records start in the correct state.
- **Subscription management**: A `trial_end_date` column defaults to `CURRENT_DATE + 30` to give every new subscriber a 30‑day trial.

---

### 7. Generated Columns

#### Definitions

**Core Definition**
A generated column is a column whose value is automatically computed from an expression involving other columns in the same row.

**Technical Definition**
Generated columns are defined with `GENERATED ALWAYS AS (expression)` and can be either `VIRTUAL` (computed on read, not stored) or `STORED` (computed on write and stored on disk). PostgreSQL supports only `STORED` generated columns. MySQL supports both `VIRTUAL` and `STORED`, with `VIRTUAL` as the default. SQL Server uses computed columns (see section 8) rather than the `GENERATED` syntax.

**Beginner-Friendly Explanation**
A generated column is a column that calculates its own value from other columns. For example, a `total` column could automatically multiply `quantity` by `unit_price` for every row.

#### Purposes

- To avoid storing redundant data that can be derived from other columns.
- To simplify queries by pre‑computing common expressions.
- To ensure consistency by centralising the computation in the table definition.
- To enable indexing on computed values (MySQL `STORED` generated columns).

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
column_name data_type GENERATED ALWAYS AS ( generation_expr ) STORED
```


**MySQL:**

```sql
column_name data_type [GENERATED ALWAYS] AS (expr) [VIRTUAL | STORED]
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `GENERATED ALWAYS AS` | Declares the column as generated. |
| `expr` | The expression that computes the column value. |
| `STORED` | The value is computed on write and stored on disk (PostgreSQL and MySQL). |
| `VIRTUAL` | The value is computed on read and not stored (MySQL default). |

**Syntax rules:**

- The expression must reference only columns in the same row.
- The expression must be deterministic (no `now()`, `random()`, or user‑defined functions that depend on session state).
- Generated columns cannot be inserted into or updated directly.

**Constraints and limitations:**

- PostgreSQL supports only `STORED` generated columns.
- MySQL `VIRTUAL` columns take no storage but are computed on every read; `STORED` columns require storage but can be indexed.
- SQL Server does not use the `GENERATED` syntax; it uses computed columns instead.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Stored generated column

```sql
-- Step 1: Create a table with a stored generated column.
CREATE TABLE invoices (
    invoice_id serial PRIMARY KEY,
    subtotal   numeric(10,2) NOT NULL,
    tax_rate   numeric(5,4) NOT NULL DEFAULT 0.20,
    total      numeric(10,2) GENERATED ALWAYS AS (subtotal * (1 + tax_rate)) STORED
);

-- Step 2: Insert a row; total is computed automatically.
INSERT INTO invoices (subtotal, tax_rate) VALUES (100.00, 0.20);
SELECT * FROM invoices;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
 invoice_id | subtotal | tax_rate | total
------------+----------+----------+-------
          1 |   100.00 |   0.2000 | 120.00
(1 row)
```

**Why this output appears**: The `total` column is computed as `subtotal * (1 + tax_rate)` when the row is inserted and stored on disk. Querying the table returns the stored computed value.

##### Example 2: MySQL – Virtual and stored generated columns

```sql
-- Step 1: Create a table with both VIRTUAL and STORED generated columns.
CREATE TABLE rectangles (
    id     INT PRIMARY KEY AUTO_INCREMENT,
    width  DECIMAL(10,2) NOT NULL,
    height DECIMAL(10,2) NOT NULL,
    area   DECIMAL(10,2) AS (width * height) VIRTUAL,
    perimeter DECIMAL(10,2) AS (2 * (width + height)) STORED
);

-- Step 2: Insert a row.
INSERT INTO rectangles (width, height) VALUES (5.00, 3.00);
SELECT * FROM rectangles;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 1 row affected (0.01 sec)

+----+-------+--------+-------+-----------+
| id | width | height | area  | perimeter |
+----+-------+--------+-------+-----------+
|  1 |  5.00 |   3.00 | 15.00 |     16.00 |
+----+-------+--------+-------+-----------+
```

**Why this output appears**: `area` is a virtual column computed on read (not stored); `perimeter` is a stored column computed on write. Both values are returned in the result set.

#### Real-World Cases

- **E‑commerce**: A `total_price` generated column computes `quantity * unit_price` automatically for each order line.
- **Payroll**: A `gross_pay` generated column computes `hours * rate` from stored columns.
- **Data warehousing**: Pre‑computed aggregations stored in generated columns reduce query‑time computation.

---

### 8. Computed Columns

#### Definitions

**Core Definition**
A computed column (SQL Server terminology) is a column whose value is derived from an expression involving other columns in the same table, similar to a generated column.

**Technical Definition**
In SQL Server, computed columns are defined with `column_name AS computed_column_expression`. They are not stored by default (virtual), but can be marked `PERSISTED` to store the computed value on disk. Computed columns can be used in indexes if the expression is deterministic and precise. SQL Server does not use the `GENERATED ALWAYS AS` syntax used by PostgreSQL and MySQL.

**Beginner-Friendly Explanation**
A computed column is SQL Server’s version of a calculated column. It automatically calculates its value from other columns, such as `total AS price * quantity`.

#### Purposes

- To avoid storing redundant data that can be derived from other columns.
- To simplify queries by pre‑computing common expressions.
- To enable indexing on computed values using `PERSISTED`.
- To provide a consistent, centralised definition of calculated fields.

#### Syntax Rules and Structure

**SQL Server:**

```sql
column_name AS computed_column_expression [ PERSISTED ]
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `column_name AS` | Declares the column as computed. |
| `computed_column_expression` | The expression that computes the value (e.g., `price * quantity`). |
| `PERSISTED` | Stores the computed value on disk (required for indexing in some cases). |

**Syntax rules:**

- The expression must reference only columns in the same table.
- The expression must be deterministic for indexing; expressions using `GETDATE()` or other non‑deterministic functions cannot be indexed.
- Computed columns cannot be inserted into or updated directly.

**Constraints and limitations:**

- SQL Server computed columns are different from PostgreSQL generated columns in syntax and feature set.
- `PERSISTED` computed columns require storage and are updated when the underlying columns change.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Computed column for line total

```sql
-- Step 1: Create a table with a computed column.
CREATE TABLE order_lines (
    order_id   INT,
    line_no    INT,
    quantity   INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    line_total AS (quantity * unit_price) PERSISTED
);
GO

-- Step 2: Insert a row.
INSERT INTO order_lines (order_id, line_no, quantity, unit_price)
VALUES (1, 1, 3, 25.00);
SELECT * FROM order_lines;
```

**Expected output:**

```text
Commands completed successfully.

(1 row affected)

order_id  line_no  quantity  unit_price  line_total
--------  -------  --------  ----------  ----------
1         1        3         25.00       75.00
```

**Why this output appears**: `line_total` is computed as `quantity * unit_price` and stored on disk because of the `PERSISTED` keyword. When the row is inserted, the computed value is calculated and stored.

#### Real-World Cases

- **Sales reporting**: A computed column calculates `revenue AS quantity * unit_price` for each sale.
- **Inventory management**: A computed column calculates `stock_value AS quantity_on_hand * unit_cost`.
- **Financial systems**: A computed column calculates `net_amount AS gross_amount - tax_amount`.

---

### 9. Identity Columns

#### Definitions

**Core Definition**
An identity column is a column whose values are automatically generated by the database using an implicit sequence, typically for primary‑key purposes.

**Technical Definition**
Identity columns are defined with `GENERATED ALWAYS AS IDENTITY` or `GENERATED BY DEFAULT AS IDENTITY` in PostgreSQL, `IDENTITY(seed, increment)` in SQL Server, and `AUTO_INCREMENT` in MySQL. PostgreSQL’s identity columns are the SQL‑standard replacement for the older `serial` pseudo‑type. SQL Server’s identity property can be applied to numeric columns and supports seed and increment values. MySQL’s `AUTO_INCREMENT` attribute is functionally equivalent.

**Beginner-Friendly Explanation**
An identity column is a column that numbers its rows automatically. You do not need to provide a value; the database assigns the next number in sequence for every new row.

#### Purposes

- To generate unique numeric identifiers for primary keys.
- To eliminate the need for application‑side ID generation.
- To provide a standard, portable alternative to vendor‑specific auto‑increment mechanisms.
- To support `GENERATED ALWAYS` (no manual override) or `BY DEFAULT` (manual override allowed) behaviour.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
column_name data_type GENERATED { ALWAYS | BY DEFAULT } AS IDENTITY [ ( sequence_options ) ]
```


**SQL Server:**

```sql
column_name data_type IDENTITY [ ( seed, increment ) ] [ NOT FOR REPLICATION ]
```


**MySQL:**

```sql
column_name data_type AUTO_INCREMENT
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `GENERATED ALWAYS AS IDENTITY` | PostgreSQL: values are always generated; manual inserts are rejected. |
| `GENERATED BY DEFAULT AS IDENTITY` | PostgreSQL: values are generated unless explicitly provided. |
| `IDENTITY(seed, increment)` | SQL Server: starting value and step size. |
| `AUTO_INCREMENT` | MySQL: automatically generates sequential integer values. |

**Syntax rules:**

- Only one identity column is allowed per table in SQL Server and MySQL.
- PostgreSQL allows multiple identity columns in the same table.
- In PostgreSQL, `GENERATED ALWAYS` prevents manual inserts into the identity column; `BY DEFAULT` allows them.

**Constraints and limitations:**

- Identity columns must be of an integer type (smallint, integer, bigint).
- In MySQL, `AUTO_INCREMENT` columns must be indexed.
- `GENERATED ALWAYS` in PostgreSQL will reject an `INSERT` that explicitly supplies a value for the identity column.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – GENERATED ALWAYS vs BY DEFAULT

```sql
-- Step 1: Create a table with GENERATED ALWAYS.
CREATE TABLE people (
    id   bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name text NOT NULL
);

-- Step 2: Insert without specifying id.
INSERT INTO people (name) VALUES ('Alice');
SELECT * FROM people;

-- Step 3: Attempt to insert an explicit id (fails).
INSERT INTO people (id, name) VALUES (100, 'Bob');
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
 id | name
----+-------
  1 | Alice
(1 row)

ERROR:  cannot insert a non-DEFAULT value into column "id"
DETAIL:  Column "id" is an identity column defined as GENERATED ALWAYS.
```

**Why this output appears**: `GENERATED ALWAYS` instructs PostgreSQL to reject any manual insert into the identity column. The first insert succeeds because the value is omitted; the second insert fails because it supplies an explicit value.

##### Example 2: SQL Server – IDENTITY column

```sql
-- Step 1: Create a table with an IDENTITY column starting at 100.
CREATE TABLE invoices (
    invoice_id INT IDENTITY(100, 1) PRIMARY KEY,
    amount     DECIMAL(10,2) NOT NULL
);
GO

-- Step 2: Insert rows.
INSERT INTO invoices (amount) VALUES (250.00), (175.00);
SELECT * FROM invoices;
```

**Expected output:**

```text
Commands completed successfully.

(2 rows affected)

invoice_id  amount
----------  -------
100         250.00
101         175.00
```

**Why this output appears**: The `IDENTITY(100, 1)` property starts generating values at 100 and increments by 1 for each new row.

#### Real-World Cases

- **User accounts**: An identity column generates unique user IDs without application involvement.
- **Order numbers**: An identity column provides sequential order numbers for tracking.
- **Audit records**: An identity column provides a unique, ever‑increasing identifier for log entries.

---

### 10. Sequences

#### Definitions

**Core Definition**
A sequence is a database object that generates a sequence of numeric values according to specified rules (start, increment, minimum, maximum, cycle).

**Technical Definition**
Sequences are independent objects that can be shared across tables and sessions. PostgreSQL supports `CREATE SEQUENCE` with full control over start, increment, min/max, cycle, and cache. SQL Server supports `CREATE SEQUENCE` and uses `NEXT VALUE FOR` to retrieve values. MySQL does not support standalone sequences; `AUTO_INCREMENT` is the functional equivalent, and sequences can be simulated with tables and triggers.

**Beginner-Friendly Explanation**
A sequence is like a ticket dispenser: every time you pull a ticket, it hands you the next number. You can set the starting number, the step size, and whether it should wrap around.

#### Purposes

- To generate unique numeric identifiers for primary keys.
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
    [ AS [ built_in_integer_type ] ]
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
| `INCREMENT BY` | Value added to the current value to produce the next value. |
| `MINVALUE` / `MAXVALUE` | Bounds for the sequence. |
| `CYCLE` | Restarts from `MINVALUE` after reaching `MAXVALUE`. |
| `CACHE` | Pre‑allocates a block of values for performance. |
| `OWNED BY` (PostgreSQL) | Associates the sequence with a table column. |

**Syntax rules:**

- The sequence name must be unique within its schema.
- `START WITH` must be within the bounds defined by `MINVALUE` and `MAXVALUE`.
- In PostgreSQL, `nextval()`, `currval()`, and `setval()` operate on sequences.

**Constraints and limitations:**

- MySQL does not support `CREATE SEQUENCE`; `AUTO_INCREMENT` is tied to a single table column.
- Sequences do not guarantee gap‑free numbering; rollbacks and caching can create gaps.
- `CYCLE` can cause duplicate values if the sequence wraps around.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Create and use a sequence

```sql
-- Step 1: Create a sequence starting at 100, incrementing by 10.
CREATE SEQUENCE order_seq START WITH 100 INCREMENT BY 10;

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
CREATE SEQUENCE dbo.OrderNumbers AS INT START WITH 1 INCREMENT BY 1;

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

**Why this output appears**: `NEXT VALUE FOR` retrieves the next sequence number and uses it directly in the `INSERT` statement, decoupling number generation from the table’s identity property.

#### Real-World Cases

- **Invoice numbering**: A sequence generates sequential invoice numbers with a specific prefix and increment.
- **Shared counters**: SQL Server sequences can be referenced by multiple tables, ensuring a single monotonic counter across the database.
- **Multi‑table ID generation**: PostgreSQL sequences can be shared across multiple tables that need a common numbering scheme.

---

### 11. Temporary Tables

#### Definitions

**Core Definition**
A temporary table is a table whose data and definition are visible only to the current session (or transaction) and are automatically dropped when the session ends or the transaction completes.

**Technical Definition**
Temporary tables are created with `CREATE TEMPORARY TABLE` (or `CREATE TEMP TABLE`). In PostgreSQL, they are stored in a session‑specific schema (`pg_temp_N`) and can be configured with `ON COMMIT PRESERVE ROWS`, `ON COMMIT DELETE ROWS`, or `ON COMMIT DROP`. In MySQL, temporary tables are session‑scoped and can use the `MEMORY` storage engine for in‑memory performance. In SQL Server, temporary tables are either local (`#temp`) or global (`##temp`); local temporary tables are visible only in the current session, while global temporary tables are visible to all sessions.

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
| `ON COMMIT DELETE ROWS` | Rows are deleted at each commit. |
| `ON COMMIT DROP` | Table is dropped at the end of the transaction. |
| `#` / `##` (SQL Server) | Prefix for local and global temporary tables. |

**Syntax rules:**

- Temporary tables are session‑scoped; each session sees its own instance.
- In PostgreSQL, temporary tables are stored in the `pg_temp` schema, which is implicitly first in the `search_path`.
- In MySQL, temporary tables are not visible to other sessions and are dropped when the session ends.

**Constraints and limitations:**

- Temporary tables in PostgreSQL are not WAL‑logged if configured on a temporary tablespace.
- In MySQL, temporary tables cannot be partitioned.
- SQL Server global temporary tables are dropped when the last session referencing them disconnects.

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

**Why this output appears**: The `#` prefix creates a session‑scoped temporary table. It is automatically dropped when the session ends or the connection is closed.

#### Real-World Cases

- **ETL staging**: A temporary table holds extracted data before transformation and loading into the target warehouse.
- **Report generation**: A complex report materialises intermediate results in a temporary table to avoid repeating expensive subqueries.
- **Session scratchpad**: An application stores user‑specific calculations in a temporary table during a long‑running session.

---

### 12. Unlogged Tables

#### Definitions

**Core Definition**
An unlogged table is a PostgreSQL table whose data is not written to the write‑ahead log (WAL), trading durability for write performance.

**Technical Definition**
`CREATE UNLOGGED TABLE` creates a table that bypasses WAL logging for `INSERT`, `UPDATE`, and `DELETE` operations. This makes writes significantly faster than regular logged tables, but the table is not crash‑safe: if the server crashes or shuts down uncleanly, the table is automatically truncated. Unlogged tables are not replicated to standby servers and are not included in WAL‑based backups.

**Beginner-Friendly Explanation**
An unlogged table is a fast but risky table. It writes data much faster because it skips the database’s safety journal, but if the server crashes, all its data disappears. It is useful for temporary, reproducible data that you can afford to lose.

#### Purposes

- To accelerate write‑heavy workloads where data can be regenerated if lost.
- To store transient or cache‑like data that does not require crash durability.
- To reduce WAL volume in high‑throughput staging or benchmarking scenarios.
- To improve performance for bulk loads where the data will later be copied to a logged table.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
CREATE UNLOGGED TABLE [ IF NOT EXISTS ] table_name ( ... );
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UNLOGGED` | Keyword that creates a non‑WAL‑logged table. |

**Syntax rules:**

- The `UNLOGGED` keyword is PostgreSQL‑specific and is not part of the SQL standard.
- Unlogged tables can have indexes, constraints, and triggers, but those objects are also unlogged.

**Constraints and limitations:**

- Unlogged tables are not crash‑safe: their contents are automatically truncated after a crash or unclean shutdown.
- Their contents are not replicated to standby servers.
- They are not included in WAL‑based backups.

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

**Why this output appears**: The `INSERT` is not WAL‑logged, so it completes faster than it would on a logged table. The data is fully readable during normal operation; the lack of durability only matters if the server crashes.

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

**Why this output appears**: PostgreSQL automatically truncates unlogged tables during crash recovery because their contents were never WAL‑logged. The table definition survives, but all rows are lost.

#### Real-World Cases

- **Benchmarking**: Performance tests use unlogged tables to measure raw write throughput without WAL overhead.
- **Session caches**: Applications store session or cache data in unlogged tables where loss after a crash is acceptable.
- **Bulk staging**: A high‑volume ETL pipeline loads data into an unlogged staging table for transformation, then writes the final result to a logged table.

---

### 13. Partitioned Tables

#### Definitions

**Core Definition**
A partitioned table is a logical table that is physically divided into smaller, independently managed pieces called partitions, based on a partitioning key.

**Technical Definition**
PostgreSQL declarative partitioning (available since version 10) allows a table to be defined with `PARTITION BY { RANGE | LIST | HASH } (column)`. The parent table holds no data itself; rows are routed to child partitions based on the partitioning key. MySQL supports `PARTITION BY RANGE | LIST | HASH` with restrictions (e.g., temporary tables cannot be partitioned). SQL Server uses partition functions and partition schemes, and `CREATE TABLE` places the table on a partition scheme.

**Beginner-Friendly Explanation**
A partitioned table is like a filing cabinet that is split into several drawers. You tell the database which drawer to use based on a rule—for example, “put all 2024 orders in the 2024 drawer and all 2025 orders in the 2025 drawer.” Queries that only need 2024 data can skip the 2025 drawer entirely.

#### Purposes

- To improve query performance on very large tables through partition pruning.
- To simplify data lifecycle management (e.g., dropping an old partition instead of deleting rows).
- To enable parallel maintenance operations on individual partitions.
- To distribute data across different tablespaces or storage tiers.

#### Syntax Rules and Structure

**PostgreSQL:**

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
```


**MySQL:**

```sql
CREATE TABLE sales (
    sale_id INT,
    sale_date DATE
)
PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026)
);
```


**SQL Server:**

```sql
-- Requires a partition function and partition scheme first
CREATE PARTITION FUNCTION pfYearly (date) AS RANGE RIGHT FOR VALUES ('2024-01-01', '2025-01-01');
CREATE PARTITION SCHEME psYearly AS PARTITION pfYearly TO ([PRIMARY], [PRIMARY], [PRIMARY]);
CREATE TABLE sales (
    sale_id INT,
    sale_date DATE
) ON psYearly (sale_date);
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `PARTITION BY RANGE` | Partitions based on a continuous range of values. |
| `PARTITION BY LIST` | Partitions based on a discrete list of values. |
| `PARTITION BY HASH` | Partitions based on a hash of the column value. |
| `PARTITION OF parent_table` | Creates a child partition of an existing partitioned table. |
| `FOR VALUES ...` | Defines the range or list of values for the partition. |

**Syntax rules:**

- The partitioning column must be part of any unique constraint or primary key on the partitioned table.
- Partitions can themselves be partitioned (sub‑partitioning).
- In MySQL, temporary tables cannot be partitioned.

**Constraints and limitations:**

- Declarative partitioning is PostgreSQL‑specific (version 10+); MySQL and SQL Server use different syntax and features.
- Unique constraints on a partitioned table must include the partition key.
- `CREATE TABLE ... PARTITION OF` takes an `ACCESS EXCLUSIVE` lock on the parent table.

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

##### Example 2: SQL Server – Partitioned table using a partition scheme

```sql
-- Step 1: Create a partition function and scheme.
CREATE PARTITION FUNCTION pfYearly (date) AS RANGE RIGHT FOR VALUES ('2024-01-01', '2025-01-01');
CREATE PARTITION SCHEME psYearly AS PARTITION pfYearly TO ([PRIMARY], [PRIMARY], [PRIMARY]);
GO

-- Step 2: Create a table on the partition scheme.
CREATE TABLE partitioned_sales (
    sale_id INT IDENTITY(1,1),
    sale_date DATE NOT NULL,
    amount DECIMAL(10,2)
) ON psYearly (sale_date);
GO
```

**Expected output:**

```text
Commands completed successfully.
```

**Why this output appears**: The table is physically partitioned according to the `pfYearly` function and placed on the `psYearly` scheme. Rows are distributed across partitions based on the `sale_date` values.

#### Real-World Cases

- **Time‑series data**: IoT sensor readings or application logs are range‑partitioned by month or day, enabling efficient queries and easy archival by dropping old partitions.
- **Multi‑region SaaS**: Customer data is list‑partitioned by region, allowing region‑specific maintenance and compliance.
- **Large fact tables**: A data warehouse fact table is hash‑partitioned to distribute I/O evenly across storage devices.

---

## References

- PostgreSQL Documentation – `CREATE TABLE` – https://www.postgresql.org/docs/current/sql-createtable.html
- PostgreSQL Documentation – Identity Columns – https://www.postgresql.org/docs/current/ddl-identity-columns.html
- PostgreSQL Documentation – Generated Columns – https://www.postgresql.org/docs/current/ddl-generated-columns.html
- PostgreSQL Documentation – Table Partitioning – https://www.postgresql.org/docs/current/ddl-partitioning.html
- PostgreSQL Documentation – Unlogged Tables – https://www.postgresql.org/docs/current/non-durability.html
- PostgreSQL Documentation – `CREATE SEQUENCE` – https://www.postgresql.org/docs/current/sql-createsequence.html
- MySQL 8.0 Reference Manual – `CREATE TABLE` – https://dev.mysql.com/doc/refman/8.0/en/create-table.html
- MySQL 8.0 Reference Manual – Generated Columns – https://dev.mysql.com/doc/refman/8.0/en/create-table-generated-columns.html
- MySQL 8.0 Reference Manual – `AUTO_INCREMENT` – https://dev.mysql.com/doc/refman/8.0/en/example-auto-increment.html
- MySQL 8.0 Reference Manual – Partitioning – https://dev.mysql.com/doc/refman/8.0/en/partitioning.html
- Microsoft Learn – `CREATE TABLE` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql
- Microsoft Learn – Computed Columns – https://learn.microsoft.com/en-us/sql/relational-databases/tables/specify-computed-columns-in-a-table
- Microsoft Learn – `IDENTITY` Property – https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql#identity
- Microsoft Learn – Sequence Numbers – https://learn.microsoft.com/en-us/sql/relational-databases/sequence-numbers/sequence-numbers
- Microsoft Learn – Partitioned Tables and Indexes – https://learn.microsoft.com/en-us/sql/relational-databases/partitions/partitioned-tables-and-indexes