# Comprehensive Programming Cheat Sheet: SQL INSERT Operations

## Topic Overview

### Definitions

**Core Definition**
INSERT operations are the Data Manipulation Language (DML) statements used to add new rows of data into a table, either by supplying explicit values, deriving them from a query, or relying on column defaults and generated expressions.

**Technical Definition**
`INSERT` adds one or more rows to a table, view, or—in some systems—an updatable CTE. The statement specifies the target table, an optional column list, and a source of row values: a `VALUES` clause (one or many rows), a `DEFAULT VALUES` clause, or a query (`INSERT ... SELECT`). All RDBMSs support the core `INSERT ... VALUES` and `INSERT ... SELECT` forms; extensions diverge significantly: PostgreSQL provides `ON CONFLICT` for upserts, `RETURNING` for returning inserted data, and `OVERRIDING` for identity columns; MySQL provides `ON DUPLICATE KEY UPDATE` for upserts, `INSERT IGNORE` for constraint violation suppression, and (since 8.0.19) a limited `RETURNING` clause; SQL Server provides `MERGE` for upserts, `OUTPUT` for returning inserted data, and `BULK INSERT`/`bcp` for high-volume loading.

**Beginner-Friendly Explanation**
Inserting data is how you put new records into a table. You tell the database which table to add to, which columns you are supplying, and what the values are. You can insert one row at a time, many rows at once, or copy rows from another table. The database checks that the data follows the rules (constraints) and then stores it permanently.

### Key Characteristics

- **Declarative value specification**: The `VALUES` clause supplies literal or computed values; `DEFAULT` can be used to explicitly request a column's default value, and `DEFAULT VALUES` inserts a row with all defaults.
- **Set-based insertion**: `INSERT ... SELECT` inserts zero or more rows derived from a query, enabling server-side data transformation and bulk transfer without round-tripping data to the client.
- **Vendor-specific upsert mechanisms**: PostgreSQL `ON CONFLICT`, MySQL `ON DUPLICATE KEY UPDATE`, and SQL Server `MERGE` all solve the "insert or update" problem but with different syntax, locking semantics, and edge cases.
- **Returning inserted data**: PostgreSQL's `RETURNING` clause and SQL Server's `OUTPUT` clause return data from the inserted rows in the same statement, eliminating the need for a separate `SELECT`; MySQL's `RETURNING` is limited to certain versions and contexts.
- **Performance is dominated by round-trips and WAL**: Single-row inserts are dominated by network round-trips and per-statement overhead; multi-row `VALUES`, `INSERT ... SELECT`, and dedicated bulk-loading tools (`COPY`, `LOAD DATA`, `bcp`) amortise this overhead and can be orders of magnitude faster.

### Prerequisites

- A running RDBMS instance (PostgreSQL, MySQL, or SQL Server) and a client connection.
- `INSERT` privilege on the target table; `UPDATE` and `SELECT` privileges are additionally required for `ON CONFLICT DO UPDATE`, `ON DUPLICATE KEY UPDATE`, and `RETURNING`/`OUTPUT`.
- The target table must exist with a compatible column structure (or the statement must supply a column list that matches a subset of columns).
- For `INSERT ... SELECT`: `SELECT` privilege on the source tables.

### Related Programming Areas

- Data Manipulation Language (DML) – `UPDATE`, `DELETE`, `MERGE`.
- Data Definition Language (DDL) – table and column definitions that determine the valid insert shape.
- Transaction management – `COMMIT`, `ROLLBACK`, and statement-level atomicity.
- Constraint enforcement – `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`.
- Bulk loading and ETL – `COPY`, `LOAD DATA`, `bcp`, `BULK INSERT`.

---

## Core Concepts and Features

### 1. `INSERT INTO`

#### Definitions

**Core Definition**
`INSERT INTO` is the SQL statement that adds one or more new rows to a table.

**Technical Definition**
The `INSERT INTO` statement specifies a target table and a source of rows. The source can be a `VALUES` list, a `DEFAULT VALUES` clause, or a query. The optional column list determines which columns receive explicit values; columns not listed receive their default values (or `NULL` if no default exists). In PostgreSQL, the general synopsis is `INSERT INTO table_name [ AS alias ] [ ( column_name [, ...] ) ] { DEFAULT VALUES | VALUES ( ... ) | query } [ ON CONFLICT ... ] [ RETURNING ... ]`. SQL Server's synopsis includes `[ TOP ( expression ) [ PERCENT ] ]`, `OUTPUT`, and support for `derived_table`, `execute_statement`, and `DEFAULT VALUES`.

**Beginner-Friendly Explanation**
`INSERT INTO` is the basic command for adding a row to a table. You name the table, optionally list the columns, and then provide the values.

#### Purposes

- To add new records to a table as part of normal application operation.
- To seed reference or lookup tables with initial data.
- To transfer data between tables or from external sources.

#### Syntax Rules and Structure

**General syntax (PostgreSQL):**

```sql
INSERT INTO table_name [ AS alias ]
    [ ( column_name [, ...] ) ]
    { DEFAULT VALUES | VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
    [ ON CONFLICT [ conflict_target ] conflict_action ]
    [ RETURNING * | output_expression [ [ AS ] output_name ] [, ...] ];
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `table_name` | The target table; can be schema-qualified. |
| `(column_name, ...)` | Optional list of columns to populate; unlisted columns receive defaults. |
| `DEFAULT VALUES` | Inserts a row with all columns set to their default values. |
| `VALUES (...)` | One or more rows of explicit values. |
| `query` | A `SELECT` (or `VALUES`) query that produces rows to insert. |

**Syntax rules:**

- If the column list is omitted, values are matched to columns in their declared order.
- If fewer values than columns are supplied without a column list, the first N columns receive values and the rest receive defaults.
- Explicitly listing columns is considered better style because it is resilient to column reordering.

**Constraints and limitations:**

- The number of values must match the number of listed columns (or the number of table columns if no list is given).
- Data types must be compatible; implicit or assignment casts may be attempted, but incompatible values are rejected.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Basic INSERT with explicit column list

```sql
-- Step 1: Create a table.
CREATE TABLE products (
    product_no integer PRIMARY KEY,
    name       text NOT NULL,
    price      numeric(10,2)
);

-- Step 2: Insert a row with an explicit column list.
INSERT INTO products (product_no, name, price)
VALUES (1, 'Cheese', 9.99);

-- Step 3: Verify.
SELECT * FROM products;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
 product_no |  name  | price
------------+--------+-------
          1 | Cheese |  9.99
(1 row)
```

**Why this output appears**: The `INSERT` statement supplies a value for each of the three listed columns. The row is stored, and `SELECT` retrieves it.

#### Real-World Cases

- **User registration**: A new row is inserted into `users` when someone signs up.
- **Order creation**: An `orders` row is inserted when a customer checks out.
- **Reference data**: A `countries` table is seeded with a fixed set of rows during database setup.

---

### 2. Single-Row Insert

#### Definitions

**Core Definition**
A single-row insert adds exactly one row to a table using a `VALUES` clause with one row of values.

**Technical Definition**
`INSERT INTO table_name (columns) VALUES (values)` inserts one row. Each value is an expression, a literal, or the keyword `DEFAULT`. The statement is atomic: either the row is inserted or the statement fails. Single-row inserts are the most common form in OLTP applications, but they are also the least efficient for bulk loading because each statement incurs parsing, planning, and network round-trip overhead.

**Beginner-Friendly Explanation**
A single-row insert adds one record at a time. It is simple and is what most applications use when a user submits a form.

#### Purposes

- To add one record in response to a user action (e.g., submitting a form).
- To insert a row where the values are computed or parameterised individually.
- To use `RETURNING` or `OUTPUT` to retrieve generated identifiers immediately.

#### Syntax Rules and Structure

**General syntax:**

```sql
INSERT INTO table_name ( column1, column2, ... )
VALUES ( value1, value2, ... );
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `( column1, column2, ... )` | The columns to populate. |
| `VALUES ( value1, value2, ... )` | One row of values corresponding to the listed columns. |

**Syntax rules:**

- The number of values must equal the number of listed columns.
- `DEFAULT` can be used as a value to request the column's default.

**Constraints and limitations:**

- Single-row inserts are inefficient for loading many rows; use multi-row `VALUES` or `INSERT ... SELECT` for bulk operations.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Single-row INSERT with RETURNING

```sql
-- Step 1: Create a table with a serial primary key.
CREATE TABLE users (
    id   serial PRIMARY KEY,
    name text NOT NULL
);

-- Step 2: Insert one row and return the generated id.
INSERT INTO users (name) VALUES ('Alice') RETURNING id;
```

**Expected output:**

```text
CREATE TABLE
 id
----
  1
(1 row)
```

**Why this output appears**: The `RETURNING id` clause returns the value generated by the `serial` column in the same statement, avoiding a separate query.

#### Real-World Cases

- **Form submission**: Each web form submission results in a single-row insert.
- **Audit event**: An application logs a single event row per action.

---

### 3. Multi-Row Insert

#### Definitions

**Core Definition**
A multi-row insert adds two or more rows in a single `INSERT` statement using a `VALUES` clause with multiple parenthesised row expressions.

**Technical Definition**
`INSERT INTO table_name (columns) VALUES (row1), (row2), ...` inserts multiple rows atomically. The rows are inserted in the order listed, and the entire statement either succeeds or fails as a unit. Multi-row `VALUES` amortises statement parsing and network overhead across all rows and is significantly faster than repeated single-row inserts for moderate row counts. SQL Server has a limit of 1,000 rows per `INSERT ... VALUES` statement and a 2,100-parameter limit; PostgreSQL has no hard row limit but a 65,535-parameter protocol limit.

**Beginner-Friendly Explanation**
A multi-row insert adds several records at once. Instead of sending one `INSERT` per row, you send one statement with many rows, which is much faster.

#### Purposes

- To load a moderate batch of rows efficiently in a single round-trip.
- To insert related rows that should be atomic (all or nothing).
- To reduce statement parsing and network overhead compared to repeated single-row inserts.

#### Syntax Rules and Structure

**General syntax:**

```sql
INSERT INTO table_name ( column1, column2, ... )
VALUES
    ( value1a, value2a, ... ),
    ( value1b, value2b, ... ),
    ...;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `(row1), (row2), ...` | Multiple parenthesised rows of values. |

**Syntax rules:**

- Each parenthesised group must have the same number of values as the column list.
- The rows are inserted in the order specified.

**Constraints and limitations:**

- SQL Server caps `INSERT ... VALUES` at 1,000 rows per statement.
- Very large `VALUES` lists can hit parameter limits; batch into chunks of 500–1,000 rows.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Multi-row INSERT

```sql
-- Step 1: Create a table.
CREATE TABLE contacts (
    id        INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name  VARCHAR(50)
);

-- Step 2: Insert three rows in one statement.
INSERT INTO contacts (id, first_name, last_name)
VALUES
    (1, 'John', 'Doe'),
    (2, 'Jane', 'Doe'),
    (3, 'Jack', 'Doe');
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

**Why this output appears**: All three rows are inserted atomically; `Records: 3` confirms the row count.

#### Real-World Cases

- **Batch import**: A CSV file is parsed and inserted in batches of 1,000 rows.
- **Seeding data**: A migration script inserts multiple reference rows in one statement.

---

### 4. Explicit Column Lists

#### Definitions

**Core Definition**
An explicit column list in an `INSERT` statement specifies the names of the columns being populated, rather than relying on the table's declared column order.

**Technical Definition**
The optional `( column_name [, ...] )` clause after the table name determines which columns receive the values supplied by `VALUES` or the query. Columns not listed are set to their default values (or `NULL` if no default exists). Explicit column lists are considered best practice because they decouple the `INSERT` statement from the physical column order and make the statement resilient to future column additions.

**Beginner-Friendly Explanation**
Instead of providing values for every column in the table's exact order, you list the columns you want to fill and provide values for just those columns. The database fills the rest with defaults.

#### Purposes

- To insert data into a subset of columns without supplying values for every column.
- To make `INSERT` statements robust against column reordering or addition.
- To explicitly control which columns receive defaults.

#### Syntax Rules and Structure

**General syntax:**

```sql
INSERT INTO table_name ( column1, column2, ... )
VALUES ( value1, value2, ... );
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `( column1, column2, ... )` | The columns to populate, in any order. |

**Syntax rules:**

- The order of values in `VALUES` corresponds to the order of columns in the list, not the table's declared order.
- Columns not listed receive their default values or `NULL`.

**Constraints and limitations:**

- If a column not listed is `NOT NULL` and has no default, the `INSERT` fails.
- Omitting the column list requires supplying values for all columns in declared order.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Explicit column list omitting a column

```sql
-- Step 1: Create a table with three columns.
CREATE TABLE weather (
    city    varchar(80),
    temp_lo int,
    temp_hi int,
    prcp    real,
    date    date
);

-- Step 2: Insert a row with an explicit column list, omitting prcp.
INSERT INTO weather (city, temp_lo, temp_hi, date)
VALUES ('San Francisco', 43, 57, '1994-11-29');

-- Step 3: Verify; prcp is NULL because it was not supplied and has no default.
SELECT * FROM weather;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
      city      | temp_lo | temp_hi | prcp |    date
----------------+---------+---------+------+------------
 San Francisco  |      43 |      57 |      | 1994-11-29
(1 row)
```

**Why this output appears**: The `prcp` column is omitted from the column list, so it receives its default value—`NULL`, because no default was declared.

#### Real-World Cases

- **Partial updates**: An application inserts a row with only the columns it knows about, letting the database fill audit timestamps and status defaults.
- **Schema evolution**: A migration adds a new column with a default; existing `INSERT` statements that use explicit column lists continue to work unchanged.

---

### 5. `INSERT ... VALUES`

#### Definitions

**Core Definition**
`INSERT ... VALUES` is the form of `INSERT` that supplies explicit values for one or more rows using a `VALUES` clause.

**Technical Definition**
The `VALUES` clause specifies one or more rows of expressions, where each expression can be a literal, a parameter, a computed expression, or the `DEFAULT` keyword. The `VALUES` clause can also be used as a standalone query (e.g., `SELECT * FROM (VALUES (...), (...)) AS t(a,b)`), but in the context of `INSERT` it is the row source. PostgreSQL's synopsis places `DEFAULT VALUES` and `VALUES (...)` as alternatives; SQL Server's synopsis includes `VALUES ( { DEFAULT | NULL | expression } [ ,...n ] )`.

**Beginner-Friendly Explanation**
`INSERT ... VALUES` is the standard way to insert explicit data. You write `VALUES` and then list the row(s) you want to insert.

#### Purposes

- To insert explicit, known values into a table.
- To use parameters or expressions to compute values at insert time.
- To request default values for specific columns using the `DEFAULT` keyword.

#### Syntax Rules and Structure

**General syntax:**

```sql
INSERT INTO table_name ( column1, column2, ... )
VALUES ( expression1, expression2, ... );
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `VALUES` | The clause that introduces explicit row values. |
| `expression` | A literal, parameter, computation, or `DEFAULT`. |

**Syntax rules:**

- `DEFAULT` can be used in place of any value to request the column's default.
- The number of expressions must match the number of listed columns.

**Constraints and limitations:**

- `VALUES` cannot contain subqueries; use `INSERT ... SELECT` for query-derived rows.
- SQL Server has a 1,000-row limit per `VALUES` clause.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – VALUES with DEFAULT for a specific column

```sql
-- Step 1: Create a table with a default.
CREATE TABLE products (
    product_no integer PRIMARY KEY,
    name       text NOT NULL,
    price      numeric(10,2) DEFAULT 9.99
);

-- Step 2: Insert a row, explicitly requesting the default for price.
INSERT INTO products (product_no, name, price)
VALUES (1, 'Cheese', DEFAULT);

-- Step 3: Verify.
SELECT * FROM products;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
 product_no |  name  | price
------------+--------+-------
          1 | Cheese |  9.99
(1 row)
```

**Why this output appears**: The `DEFAULT` keyword in the `VALUES` list instructs PostgreSQL to use the column's declared default (`9.99`) instead of supplying an explicit value.

#### Real-World Cases

- **Parameterised inserts**: An application uses a prepared statement with `VALUES ($1, $2, $3)` to insert user-supplied data safely.
- **Conditional defaults**: A row is inserted with `DEFAULT` for a `created_at` column so the database sets the timestamp.

---

### 6. `INSERT ... SELECT`

#### Definitions

**Core Definition**
`INSERT ... SELECT` inserts zero or more rows into a target table, where the rows are produced by a `SELECT` query rather than a `VALUES` list.

**Technical Definition**
`INSERT INTO target (columns) SELECT ... FROM source ...` executes the query and inserts each row it returns into the target table. The query can involve joins, aggregates, filters, and expressions. This is the standard set-based way to copy or transform data within the database, avoiding the round-trip of exporting and re-importing rows. In PostgreSQL, the query can be a `VALUES` list, a `SELECT`, or a `WITH` query. SQL Server supports `INSERT ... SELECT` as an alternative to `VALUES` in the same statement position.

**Beginner-Friendly Explanation**
`INSERT ... SELECT` copies rows from one place to another. Instead of writing out values by hand, you write a query that produces the rows, and those rows are inserted into the target table.

#### Purposes

- To copy rows from one table to another (e.g., archiving, staging, or migration).
- To transform data during insertion (e.g., aggregating, joining, or computing new values).
- To load a table from a query result without transferring data to the client.

#### Syntax Rules and Structure

**General syntax:**

```sql
INSERT INTO target_table ( column1, column2, ... )
SELECT expression1, expression2, ...
FROM source_table
WHERE ...;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `target_table` | The table receiving the rows. |
| `SELECT ...` | The query that produces the rows to insert. |

**Syntax rules:**

- The number and types of columns produced by the `SELECT` must match the target column list.
- The `SELECT` can reference any tables, use joins, aggregates, and CTEs.

**Constraints and limitations:**

- The target table cannot appear in the `FROM` clause of the `SELECT` in some RDBMSs (or requires special handling).
- Constraint violations in any row cause the entire statement to fail (unless `ON CONFLICT` or `IGNORE` is used).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – INSERT ... SELECT from another table

```sql
-- Step 1: Create source and target tables.
CREATE TABLE source_data (id int, value text);
INSERT INTO source_data VALUES (1, 'a'), (2, 'b'), (3, 'c');

CREATE TABLE target_data (id int, value text);

-- Step 2: Copy all rows from source to target.
INSERT INTO target_data (id, value)
SELECT id, value FROM source_data;

-- Step 3: Verify.
SELECT * FROM target_data;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 3
CREATE TABLE
INSERT 0 3
 id | value
----+-------
  1 | a
  2 | b
  3 | c
(3 rows)
```

**Why this output appears**: The `SELECT` returns three rows, and `INSERT ... SELECT` inserts all three into `target_data` in a single statement.

#### Real-World Cases

- **Archiving**: Rows older than a threshold are moved from a live table to an archive table using `INSERT ... SELECT` followed by `DELETE`.
- **Data warehouse loading**: A fact table is populated by aggregating and joining staging tables.

---

### 7. Inserting Default Values

#### Definitions

**Core Definition**
Inserting default values means creating a row where every column receives its declared default value (or `NULL` if no default exists), without supplying any explicit values.

**Technical Definition**
PostgreSQL provides the `DEFAULT VALUES` clause: `INSERT INTO table_name DEFAULT VALUES` inserts a row with all columns set to their default values, as if `DEFAULT` were specified for each column. Individual columns can also receive defaults by omitting them from the column list or by using the `DEFAULT` keyword in the `VALUES` list. SQL Server supports `DEFAULT VALUES` as well. MySQL does not have a standalone `DEFAULT VALUES` clause in the same way, but `INSERT INTO t () VALUES ()` achieves a similar result.

**Beginner-Friendly Explanation**
Sometimes you want to insert a row without specifying any values—just let the database fill in every column with its default. `DEFAULT VALUES` does exactly that.

#### Purposes

- To create a row that is entirely composed of defaults (e.g., a placeholder row in a queue table).
- To rely on the database to generate all values, including identity columns, timestamps, and status defaults.
- To explicitly request defaults for specific columns using the `DEFAULT` keyword.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
INSERT INTO table_name DEFAULT VALUES;
```


**SQL Server:**

```sql
INSERT INTO table_name DEFAULT VALUES;
```


**MySQL (equivalent):**

```sql
INSERT INTO table_name () VALUES ();
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `DEFAULT VALUES` | Inserts one row with all columns set to their defaults. |

**Syntax rules:**

- `DEFAULT VALUES` inserts exactly one row.
- Every column must have a default or be nullable; otherwise, the statement fails.

**Constraints and limitations:**

- MySQL does not support `DEFAULT VALUES`; use `INSERT INTO t () VALUES ()` instead.
- `DEFAULT VALUES` cannot be combined with a column list.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – DEFAULT VALUES with serial and default timestamp

```sql
-- Step 1: Create a table where every column has a default.
CREATE TABLE event_log (
    id         serial PRIMARY KEY,
    message    text DEFAULT 'no message',
    created_at timestamptz DEFAULT now()
);

-- Step 2: Insert a row using DEFAULT VALUES.
INSERT INTO event_log DEFAULT VALUES;

-- Step 3: Verify.
SELECT * FROM event_log;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
 id |  message   |          created_at
----+------------+-------------------------------
  1 | no message | 2026-11-13 10:00:00.123+00
(1 row)
```

**Why this output appears**: `DEFAULT VALUES` supplies no explicit values; the `serial` column generates `1`, `message` uses `'no message'`, and `created_at` uses the current timestamp.

#### Real-World Cases

- **Queue placeholder**: A job queue table inserts a row with all defaults to claim the next sequence number.
- **Audit sentinel**: A row is inserted with only defaults to mark the start of a batch process.

---

### 8. Handling Generated Identifiers

#### Definitions

**Core Definition**
Handling generated identifiers is the process of retrieving the values that the database automatically generated for identity, serial, or auto-increment columns during an `INSERT`.

**Technical Definition**
Different RDBMSs provide different mechanisms for retrieving generated identifiers. **PostgreSQL** uses the `RETURNING` clause, which returns the inserted row's columns (including generated identifiers) as part of the `INSERT` statement. **SQL Server** uses the `OUTPUT` clause, which returns data from the inserted rows, including identity values, either to the client or into a table variable. **MySQL** provides `LAST_INSERT_ID()`, which returns the first auto-increment value generated by the most recent `INSERT`; for multi-row inserts, it returns the first generated ID, not all of them. MySQL 8.0.19+ also supports a limited `RETURNING` clause in some contexts.

**Beginner-Friendly Explanation**
When you insert a row into a table with an automatic ID column, the database assigns an ID. Handling generated identifiers means getting that ID back so your application can use it—for example, to link a new order to its order items.

#### Purposes

- To obtain the primary key of a newly inserted row for use in related inserts.
- To avoid a separate `SELECT` query after `INSERT`, reducing round-trips.
- To retrieve multiple generated identifiers when inserting multiple rows.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
INSERT INTO table_name (columns) VALUES (values) RETURNING id;
```


**SQL Server:**

```sql
INSERT INTO table_name (columns)
OUTPUT INSERTED.id
VALUES (values);
```


**MySQL:**

```sql
INSERT INTO table_name (columns) VALUES (values);
SELECT LAST_INSERT_ID();
```


**Component breakdown:**

| RDBMS | Mechanism | Returns |
|-------|-----------|---------|
| PostgreSQL | `RETURNING` | The inserted row's columns |
| SQL Server | `OUTPUT INSERTED.*` | The inserted row's columns |
| MySQL | `LAST_INSERT_ID()` | The first generated ID |

**Syntax rules:**

- PostgreSQL's `RETURNING` can return any expression using the inserted row's columns.
- SQL Server's `OUTPUT` can output to the client or into a table variable for further processing.
- MySQL's `LAST_INSERT_ID()` is session-scoped and returns the first generated ID from the most recent insert.

**Constraints and limitations:**

- MySQL's `LAST_INSERT_ID()` returns only the first ID of a multi-row insert, not all of them.
- MySQL's `RETURNING` support is version- and context-dependent; it is not available in all MySQL versions or for all statement forms.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – RETURNING generated ID

```sql
-- Step 1: Create a table with a serial primary key.
CREATE TABLE users (id serial PRIMARY KEY, name text);

-- Step 2: Insert a row and return the generated id.
INSERT INTO users (name) VALUES ('Alice') RETURNING id;
```

**Expected output:**

```text
CREATE TABLE
 id
----
  1
(1 row)
```

**Why this output appears**: The `RETURNING id` clause returns the value generated by the `serial` column in the same statement.

##### Example 2: SQL Server – OUTPUT inserted identity

```sql
-- Step 1: Create a table with an identity column.
CREATE TABLE users (id int IDENTITY(1,1) PRIMARY KEY, name nvarchar(100));

-- Step 2: Insert a row and output the generated id.
INSERT INTO users (name)
OUTPUT INSERTED.id
VALUES (N'Alice');
```

**Expected output:**

```text
Commands completed successfully.
id
-----------
1
```

**Why this output appears**: The `OUTPUT INSERTED.id` clause returns the identity value generated for the inserted row.

#### Real-World Cases

- **Parent-child inserts**: An order is inserted, its generated `order_id` is retrieved, and that ID is used to insert the order's line items.
- **API response**: A REST API returns the newly created resource's ID by using `RETURNING` or `OUTPUT`.

---

### 9. Handling Constraint Violations

#### Definitions

**Core Definition**
Handling constraint violations is the practice of anticipating and managing the errors or alternative behaviours that occur when an `INSERT` would violate a table constraint such as `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, or `NOT NULL`.

**Technical Definition**
By default, a constraint violation aborts the entire `INSERT` statement and raises an error. RDBMSs provide several mechanisms to handle violations gracefully: **PostgreSQL** offers `ON CONFLICT DO NOTHING` (skip conflicting rows) and `ON CONFLICT DO UPDATE` (upsert). **MySQL** offers `INSERT IGNORE` (convert errors to warnings and skip conflicting rows) and `ON DUPLICATE KEY UPDATE` (upsert). **SQL Server** offers `MERGE` for upserts and `TRY...CATCH` for procedural error handling; it does not have a direct `INSERT IGNORE` equivalent. Application-level pre-checks or `MERGE` are the typical approaches.

**Beginner-Friendly Explanation**
When you try to insert data that breaks a rule—like a duplicate ID or a missing required field—the database normally stops and shows an error. Handling constraint violations means deciding what should happen instead: skip the bad row, update the existing row, or handle the error in your code.

#### Purposes

- To insert data idempotently, so repeated runs do not fail on existing rows.
- To implement upsert behaviour: insert new rows and update existing ones.
- To skip individual bad rows during a bulk load without aborting the entire operation.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
INSERT INTO table_name (columns) VALUES (values)
ON CONFLICT (conflict_target) DO NOTHING;

INSERT INTO table_name (columns) VALUES (values)
ON CONFLICT (conflict_target) DO UPDATE SET column = EXCLUDED.column;
```


**MySQL:**

```sql
INSERT IGNORE INTO table_name (columns) VALUES (values);

INSERT INTO table_name (columns) VALUES (values)
ON DUPLICATE KEY UPDATE column = VALUES(column);
```


**SQL Server:**

```sql
BEGIN TRY
    INSERT INTO table_name (columns) VALUES (values);
END TRY
BEGIN CATCH
    -- handle error
END CATCH;
```


**Component breakdown:**

| Mechanism | RDBMS | Behaviour |
|-----------|-------|-----------|
| `ON CONFLICT DO NOTHING` | PostgreSQL | Skips rows that conflict on the specified target |
| `ON CONFLICT DO UPDATE` | PostgreSQL | Updates the existing row with new values |
| `INSERT IGNORE` | MySQL | Converts errors to warnings and skips conflicting rows |
| `ON DUPLICATE KEY UPDATE` | MySQL | Updates the existing row |
| `MERGE` | SQL Server | Inserts, updates, or deletes based on a match condition |
| `TRY...CATCH` | SQL Server | Catches and handles errors procedurally |

**Syntax rules:**

- `ON CONFLICT` requires a conflict target (a unique index or constraint) for `DO UPDATE`; `DO NOTHING` can omit it.
- `INSERT IGNORE` also ignores `NOT NULL` violations by inserting implicit defaults, which may mask data issues.
- `ON DUPLICATE KEY UPDATE` uses the `VALUES()` function (or `new.` alias) to refer to the values that would have been inserted.

**Constraints and limitations:**

- `INSERT IGNORE` can suppress errors that you did not intend to suppress (e.g., data truncation, invalid dates).
- MySQL's `ON DUPLICATE KEY UPDATE` with multiple unique indexes may update an arbitrary matching row, leading to non-deterministic behaviour.
- SQL Server's `MERGE` has known concurrency and correctness caveats; test thoroughly and consider `HOLDLOCK`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – INSERT IGNORE skips duplicate rows

```sql
-- Step 1: Create a table with a unique key.
CREATE TABLE users (id INT PRIMARY KEY, email VARCHAR(255) UNIQUE);

-- Step 2: Insert a row.
INSERT INTO users VALUES (1, 'a@example.com');

-- Step 3: Attempt to insert a duplicate using INSERT IGNORE.
INSERT IGNORE INTO users VALUES (1, 'b@example.com');
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 1 row affected (0.01 sec)
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

**Why this output appears**: `INSERT IGNORE` suppresses the duplicate-key error, converts it to a warning, and skips the row without aborting the statement.

#### Real-World Cases

- **Idempotent migrations**: A seed script uses `INSERT ... ON CONFLICT DO NOTHING` so it can be run repeatedly without failing.
- **Data ingestion**: A pipeline uses `INSERT IGNORE` to skip rows that violate uniqueness, logging warnings for later review.

---

### 10. `DEFAULT` Values

#### Definitions

**Core Definition**
`DEFAULT` values are the values that the database automatically assigns to a column when no explicit value is provided during an `INSERT`.

**Technical Definition**
A column's default is specified in its `CREATE TABLE` definition using the `DEFAULT` clause. During an `INSERT`, if the column is omitted from the column list, or if the `DEFAULT` keyword is supplied as its value, the database evaluates and uses the default expression. In PostgreSQL, the default is stored in `pg_attrdef` and evaluated at insert time. The `DEFAULT` keyword can also be used explicitly in a `VALUES` list to request the default for a specific column.

**Beginner-Friendly Explanation**
A `DEFAULT` value is the automatic fill-in for a column. If you do not provide a value, the database uses the default you defined.

#### Purposes

- To supply sensible values for columns that are often left blank.
- To automatically generate timestamps, identifiers, or status values.
- To decouple the application from the details of default values.

#### Syntax Rules and Structure

**Default in table definition:**

```sql
CREATE TABLE table_name (
    column_name data_type DEFAULT default_expression
);
```


**Explicit DEFAULT in INSERT:**

```sql
INSERT INTO table_name (column_name) VALUES (DEFAULT);
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `DEFAULT default_expression` | The default value or expression in the column definition. |
| `DEFAULT` in `VALUES` | Explicitly requests the column's default. |

**Syntax rules:**

- The default expression must be compatible with the column's data type.
- `DEFAULT` can be used in place of any value in a `VALUES` list.

**Constraints and limitations:**

- A `NOT NULL` column with no default will cause an `INSERT` to fail if no value is supplied.
- Volatile default expressions (e.g., `now()`, `random()`) are evaluated once per row.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Omitting a column to use its default

```sql
-- Step 1: Create a table with a default.
CREATE TABLE accounts (
    id      serial PRIMARY KEY,
    balance numeric DEFAULT 0
);

-- Step 2: Insert a row without specifying balance.
INSERT INTO accounts (id) VALUES (DEFAULT);

-- Step 3: Verify; balance is 0.
SELECT * FROM accounts;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
 id | balance
----+---------
  1 |       0
(1 row)
```

**Why this output appears**: The `balance` column is omitted from the column list, so it receives its default value of `0`.

#### Real-World Cases

- **Audit timestamps**: `created_at DEFAULT now()` automatically records row creation time.
- **Status defaults**: A `status` column defaults to `'pending'` for new records.

---

### 11. `RETURNING` (PostgreSQL)

#### Definitions

**Core Definition**
`RETURNING` is a PostgreSQL clause that returns data from the rows inserted, updated, or deleted by a DML statement.

**Technical Definition**
The `RETURNING` clause can appear on `INSERT`, `UPDATE`, `DELETE`, and `MERGE` statements. Its output list is identical to a `SELECT` output list: it can contain column names, expressions, and `*`. For `INSERT`, the default data available to `RETURNING` is the row as inserted; for `UPDATE`, it is the new content; for `DELETE`, it is the deleted row; and for `MERGE`, it is the source row plus the target row. `RETURNING` requires `SELECT` privilege on the columns mentioned.

**Beginner-Friendly Explanation**
`RETURNING` lets you get data back from the rows you just inserted, without running a separate query. It is especially useful for retrieving automatically generated IDs and default values.

#### Purposes

- To retrieve generated identifiers (e.g., serial or identity values) in the same statement.
- To return default values or computed columns from inserted rows.
- To avoid an extra `SELECT` round-trip after an `INSERT`.

#### Syntax Rules and Structure

**General syntax:**

```sql
INSERT INTO table_name (columns) VALUES (values)
RETURNING * | output_expression [ [ AS ] output_name ] [, ...];
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `RETURNING` | The clause keyword. |
| `output_expression` | A column name or expression using the inserted row's columns. |
| `output_name` | An optional alias for the output column. |

**Syntax rules:**

- The `RETURNING` list can include `*` to return all columns.
- The data returned is the row as it exists after insertion (including defaults and trigger modifications).
- `RETURNING` can be used with `ON CONFLICT DO UPDATE` to return the updated row.

**Constraints and limitations:**

- `RETURNING` is PostgreSQL-specific; MySQL and SQL Server have different mechanisms (`LAST_INSERT_ID()` and `OUTPUT`).
- The user must have `SELECT` privilege on the returned columns.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – RETURNING multiple columns

```sql
-- Step 1: Create a table with a serial ID and a default timestamp.
CREATE TABLE posts (
    id         serial PRIMARY KEY,
    title      text NOT NULL,
    created_at timestamptz DEFAULT now()
);

-- Step 2: Insert and return the generated id and timestamp.
INSERT INTO posts (title) VALUES ('Hello World')
RETURNING id, created_at;
```

**Expected output:**

```text
CREATE TABLE
 id |          created_at
----+-------------------------------
  1 | 2026-11-13 10:00:00.123+00
(1 row)
```

**Why this output appears**: `RETURNING id, created_at` returns the generated serial value and the default timestamp for the inserted row.

#### Real-World Cases

- **API responses**: A POST endpoint inserts a row and returns the generated resource ID in the same database round-trip.
- **Audit logging**: After inserting a row, `RETURNING` provides the values needed to write an audit entry.

---

### 12. `ON CONFLICT` (PostgreSQL)

#### Definitions

**Core Definition**
`ON CONFLICT` is a PostgreSQL clause that specifies an alternative action when an `INSERT` would violate a unique or exclusion constraint: either do nothing or update the existing row.

**Technical Definition**
`ON CONFLICT` has two forms: `DO NOTHING` (skip the conflicting row) and `DO UPDATE SET ...` (update the existing row with new values). The conflict is identified by a conflict target: a column list with an associated unique index, or `ON CONSTRAINT constraint_name`. In the `DO UPDATE` clause, the `EXCLUDED` pseudo-table refers to the row that would have been inserted, allowing the update to reference the proposed new values. `ON CONFLICT DO UPDATE` is the standard PostgreSQL upsert mechanism.

**Beginner-Friendly Explanation**
`ON CONFLICT` tells PostgreSQL what to do when an insert would create a duplicate: either skip it (`DO NOTHING`) or update the existing row (`DO UPDATE`). It is how you do “insert or update” in PostgreSQL.

#### Purposes

- To implement idempotent inserts that do not fail on duplicate keys.
- To perform upserts: insert new rows and update existing ones in a single statement.
- To handle unique constraint violations gracefully during bulk loading.

#### Syntax Rules and Structure

**General syntax:**

```sql
INSERT INTO table_name (columns) VALUES (values)
ON CONFLICT [ conflict_target ]
    { DO NOTHING | DO UPDATE SET column = expression [, ...] [ WHERE condition ] };
```


**Conflict target:**

```sql
( { index_column_name | ( index_expression ) } [ COLLATE collation ] [ opclass ] [, ...] )
    [ WHERE index_predicate ]
| ON CONSTRAINT constraint_name
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `conflict_target` | The unique index or constraint that triggers the conflict. |
| `DO NOTHING` | Skips the conflicting row without raising an error. |
| `DO UPDATE SET` | Updates the existing row with new values. |
| `EXCLUDED` | Pseudo-table referencing the proposed new row. |

**Syntax rules:**

- `DO UPDATE` requires a conflict target; `DO NOTHING` can omit it.
- `EXCLUDED.column` refers to the value that would have been inserted.
- A `WHERE` clause can be added to `DO UPDATE` to conditionally update.

**Constraints and limitations:**

- `ON CONFLICT` only handles unique and exclusion constraint violations; other constraint violations still raise errors.
- `ON CONFLICT DO UPDATE` requires `UPDATE` privilege on the table and `SELECT` privilege on any columns read in the update expressions.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Upsert with ON CONFLICT DO UPDATE

```sql
-- Step 1: Create a table with a unique constraint.
CREATE TABLE counters (
    key   text PRIMARY KEY,
    value int NOT NULL DEFAULT 0
);

-- Step 2: Insert a row.
INSERT INTO counters (key, value) VALUES ('hits', 1);

-- Step 3: Upsert: increment the value if the key exists.
INSERT INTO counters (key, value) VALUES ('hits', 1)
ON CONFLICT (key) DO UPDATE
SET value = counters.value + EXCLUDED.value;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
INSERT 0 1
```

**Why this output appears**: The second `INSERT` conflicts on the primary key `key`. The `DO UPDATE` clause sets `value` to the existing value plus the `EXCLUDED.value` (the proposed new value), effectively incrementing the counter.

#### Real-World Cases

- **Counter increments**: A page-view counter uses `ON CONFLICT DO UPDATE` to atomically increment the count.
- **Idempotent API**: A sync endpoint uses `ON CONFLICT DO NOTHING` to avoid duplicating records when the same payload is received twice.

---

### 13. `ON DUPLICATE KEY` (MySQL)

#### Definitions

**Core Definition**
`ON DUPLICATE KEY UPDATE` is a MySQL clause that performs an `UPDATE` on an existing row when an `INSERT` would cause a duplicate value in a `UNIQUE` index or `PRIMARY KEY`.

**Technical Definition**
When an `INSERT ... ON DUPLICATE KEY UPDATE` encounters a duplicate key, MySQL updates the existing row with the specified assignments instead of inserting a new one. The `VALUES(column)` function (or the `new.` alias in newer versions) refers to the value that would have been inserted. If the row is inserted as new, the affected-rows value is 1; if an existing row is updated, it is 2; if the existing row is set to its current values, it is 0. The `LAST_INSERT_ID()` function returns the `AUTO_INCREMENT` value of the inserted or updated row.

**Beginner-Friendly Explanation**
`ON DUPLICATE KEY UPDATE` is MySQL's way of saying: “If this row already exists, update it instead of inserting a new one.” It is how you do “insert or update” in MySQL.

#### Purposes

- To implement upserts in MySQL.
- To update a row's values when a duplicate key is encountered.
- To atomically increment counters or update timestamps on conflict.

#### Syntax Rules and Structure

**General syntax:**

```sql
INSERT INTO table_name (columns) VALUES (values)
ON DUPLICATE KEY UPDATE column = expression [, ...];
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ON DUPLICATE KEY UPDATE` | The clause that specifies the update action. |
| `column = expression` | The assignment(s) to perform on the existing row. |
| `VALUES(column)` | Refers to the value that would have been inserted. |

**Syntax rules:**

- `VALUES(column)` (or `new.column`) retrieves the proposed value.
- Multiple assignments can be separated by commas.

**Constraints and limitations:**

- The statement is not standard SQL; it is MySQL-specific.
- With multiple unique indexes, MySQL may update an arbitrary matching row, so avoid using this clause on tables with multiple unique indexes.
- `INSERT IGNORE` can be combined with `ON DUPLICATE KEY UPDATE`, but the behaviour may be unexpected when multiple unique keys are involved.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Upsert with ON DUPLICATE KEY UPDATE

```sql
-- Step 1: Create a table with a unique key.
CREATE TABLE counters (
    id    INT PRIMARY KEY,
    value INT NOT NULL DEFAULT 0
);

-- Step 2: Insert a row.
INSERT INTO counters (id, value) VALUES (1, 1);

-- Step 3: Upsert: increment the value if the id exists.
INSERT INTO counters (id, value) VALUES (1, 1)
ON DUPLICATE KEY UPDATE value = value + VALUES(value);
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 1 row affected (0.01 sec)
Query OK, 2 rows affected (0.00 sec)
```

**Why this output appears**: The second `INSERT` conflicts on the primary key `id = 1`. The `ON DUPLICATE KEY UPDATE` clause updates the existing row, adding `VALUES(value)` (which is `1`) to the existing `value`, resulting in `2`. The affected-rows value is `2`, indicating an update.

#### Real-World Cases

- **Page-view counters**: `INSERT ... ON DUPLICATE KEY UPDATE` increments a counter for a given page or entity.
- **User settings**: A settings row is inserted if absent, or updated if the user already has settings.

---

### 14. `MERGE` (SQL Server)

#### Definitions

**Core Definition**
`MERGE` is a SQL Server statement that performs `INSERT`, `UPDATE`, or `DELETE` operations on a target table based on the results of a join with a source table.

**Technical Definition**
`MERGE INTO target USING source ON join_condition` evaluates each source row against the target. Depending on whether a match is found, it executes the corresponding `WHEN MATCHED THEN UPDATE`, `WHEN NOT MATCHED THEN INSERT`, or `WHEN MATCHED THEN DELETE` action. `MERGE` is SQL Server's primary upsert mechanism, though it is also used for synchronisation tasks. Microsoft recommends specifying only the join columns in the `ON` clause and using `WHEN` clauses for additional filtering. For performance, ensure indexes exist on the join columns; the optimizer chooses among nested loops, merge join, and hash join based on table sizes and indexes.

**Beginner-Friendly Explanation**
`MERGE` compares two tables row by row and decides what to do for each row: if it matches, update it; if it doesn't match, insert it. It is SQL Server's way of doing “insert or update” (and sometimes delete) in one statement.

#### Purposes

- To implement upserts: insert new rows and update existing ones.
- To synchronise a target table with a source table.
- To perform conditional inserts, updates, and deletes in a single set-based statement.

#### Syntax Rules and Structure

**General syntax (SQL Server):**

```sql
MERGE [ TOP ( expression ) [ PERCENT ] ] [ INTO ] target_table
USING source_table
ON join_condition
[ WHEN MATCHED [ AND condition ] THEN { UPDATE SET ... | DELETE } ]
[ WHEN NOT MATCHED [ BY TARGET ] [ AND condition ] THEN INSERT ... ]
[ WHEN NOT MATCHED BY SOURCE [ AND condition ] THEN { UPDATE SET ... | DELETE } ];
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `target_table` | The table being modified. |
| `source_table` | The table providing the rows. |
| `ON join_condition` | The match condition between source and target. |
| `WHEN MATCHED THEN UPDATE` | Updates existing rows. |
| `WHEN NOT MATCHED THEN INSERT` | Inserts new rows. |
| `WHEN MATCHED THEN DELETE` | Deletes matching rows (if specified). |

**Syntax rules:**

- Only the join columns should be in the `ON` clause; additional filtering should use `WHEN ... AND condition`.
- `MERGE` must be terminated with a semicolon.

**Constraints and limitations:**

- `MERGE` is not parameterised automatically, so literal values can lead to recompilation and suboptimal plans; use parameters where possible.
- `MERGE` has known concurrency caveats; use `HOLDLOCK` on the target to avoid race conditions.
- Benchmark `MERGE` against separate `UPDATE` and `INSERT` statements; sometimes the separate statements perform better.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – MERGE for upsert

```sql
-- Step 1: Create target and source tables.
CREATE TABLE target (id int PRIMARY KEY, value int);
CREATE TABLE source (id int, value int);

INSERT INTO target VALUES (1, 100);
INSERT INTO source VALUES (1, 200), (2, 300);

-- Step 2: MERGE source into target.
MERGE INTO target AS t
USING source AS s
ON t.id = s.id
WHEN MATCHED THEN
    UPDATE SET value = s.value
WHEN NOT MATCHED THEN
    INSERT (id, value) VALUES (s.id, s.value);
```

**Expected output:**

```text
Commands completed successfully.
```

**Why this output appears**: For `id = 1`, a match is found, so the `value` is updated to `200`. For `id = 2`, no match is found, so a new row is inserted. The `MERGE` statement handles both cases in one pass.

#### Real-World Cases

- **Data synchronisation**: A nightly job merges a staging table into a production dimension table, updating existing rows and inserting new ones.
- **Slowly changing dimensions**: `MERGE` is used to apply type-1 and type-2 changes to a dimension table.

---

### 15. Upsert Patterns

#### Definitions

**Core Definition**
An upsert pattern is a technique that inserts a row if it does not exist and updates it if it does exist, in a single atomic operation or a controlled sequence.

**Technical Definition**
Upsert is a portmanteau of “update” and “insert.” Different RDBMSs implement it differently: **PostgreSQL** uses `INSERT ... ON CONFLICT DO UPDATE`; **MySQL** uses `INSERT ... ON DUPLICATE KEY UPDATE`; **SQL Server** uses `MERGE`. A portable fallback pattern—often called “update-then-insert” or “select-then-act”—involves first attempting an `UPDATE`, checking the row count, and then performing an `INSERT` if no rows were updated. This pattern is less efficient than native upserts but is portable across RDBMSs.

**Beginner-Friendly Explanation**
An upsert is a “save or update” operation. If the record is new, insert it; if it already exists, update it. Each database has its own way of doing this.

#### Purposes

- To implement “save” semantics where the caller does not know whether the record already exists.
- To synchronise data from external sources without duplicating rows.
- To atomically update counters, settings, or state.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
INSERT INTO table (id, value) VALUES ($1, $2)
ON CONFLICT (id) DO UPDATE SET value = EXCLUDED.value;
```


**MySQL:**

```sql
INSERT INTO table (id, value) VALUES ($1, $2)
ON DUPLICATE KEY UPDATE value = VALUES(value);
```


**SQL Server:**

```sql
MERGE INTO table AS t
USING (VALUES ($1, $2)) AS s(id, value)
ON t.id = s.id
WHEN MATCHED THEN UPDATE SET value = s.value
WHEN NOT MATCHED THEN INSERT (id, value) VALUES (s.id, s.value);
```


**Portable update-then-insert:**

```sql
UPDATE table SET value = $2 WHERE id = $1;
-- if row_count = 0:
INSERT INTO table (id, value) VALUES ($1, $2);
```


**Component breakdown:**

| Pattern | RDBMS | Atomicity |
|---------|-------|-----------|
| `ON CONFLICT` | PostgreSQL | Single statement, atomic |
| `ON DUPLICATE KEY UPDATE` | MySQL | Single statement, atomic |
| `MERGE` | SQL Server | Single statement, atomic (with caveats) |
| Update-then-insert | Portable | Not atomic without transaction/locking |

**Syntax rules:**

- Native upserts are atomic and do not require explicit transactions.
- The portable pattern requires a transaction and careful locking to avoid race conditions.

**Constraints and limitations:**

- Native upsert syntax is not portable across RDBMSs.
- `MERGE` has concurrency caveats; use `HOLDLOCK` when necessary.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – ON CONFLICT upsert

```sql
-- Step 1: Create a table.
CREATE TABLE settings (
    user_id int PRIMARY KEY,
    theme   text NOT NULL
);

-- Step 2: Upsert a setting.
INSERT INTO settings (user_id, theme) VALUES (1, 'dark')
ON CONFLICT (user_id) DO UPDATE SET theme = EXCLUDED.theme;

-- Step 3: Upsert again with a different theme.
INSERT INTO settings (user_id, theme) VALUES (1, 'light')
ON CONFLICT (user_id) DO UPDATE SET theme = EXCLUDED.theme;

-- Step 4: Verify.
SELECT * FROM settings;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
INSERT 0 1
 user_id | theme
---------+-------
       1 | light
(1 row)
```

**Why this output appears**: The first insert creates the row. The second insert conflicts on `user_id` and updates the `theme` to `'light'` via `EXCLUDED.theme`.

#### Real-World Cases

- **User preferences**: A settings table uses upsert so that saving preferences works whether or not the user has existing settings.
- **Inventory counts**: A stock table uses upsert to add received quantities to existing counts.

---

### 16. Bulk Inserts

#### Definitions

**Core Definition**
Bulk inserts are techniques for loading large volumes of rows into a table efficiently, using multi-row statements, query-based inserts, or dedicated bulk-loading utilities.

**Technical Definition**
Bulk inserts amortise per-statement overhead (parsing, planning, network round-trips) across many rows. The primary SQL-level techniques are multi-row `VALUES` (batched to respect RDBMS limits) and `INSERT ... SELECT` from a staging table or query. For very large volumes, dedicated utilities are faster: **PostgreSQL** provides `COPY` (and the binary `COPY` protocol), which writes directly to heap pages and is significantly faster than repeated `INSERT` statements; **MySQL** provides `LOAD DATA INFILE`, which can be an order of magnitude faster than `INSERT`; **SQL Server** provides `BULK INSERT` (T-SQL) and `bcp` (command-line), as well as `SqlBulkCopy` from application code. Benchmarks show that bulk utilities can be 3–7× faster than multi-row inserts and 50–130× faster than single-row inserts, depending on the RDBMS and workload.

**Beginner-Friendly Explanation**
Bulk inserts are how you load a lot of data quickly. Instead of inserting one row at a time, you use a special tool or a multi-row statement that the database can process much more efficiently.

#### Purposes

- To load large datasets (e.g., ETL, data migration, initial seeding) efficiently.
- To minimise the time and resource cost of inserting millions of rows.
- To reduce network round-trips and statement overhead.

#### Syntax Rules and Structure

**PostgreSQL COPY:**

```sql
COPY table_name (column1, column2, ...) FROM '/path/to/file.csv' WITH (FORMAT csv);
```


**MySQL LOAD DATA:**

```sql
LOAD DATA INFILE '/path/to/file.csv'
INTO TABLE table_name
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n';
```


**SQL Server BULK INSERT:**

```sql
BULK INSERT table_name
FROM 'C:\path\to\file.csv'
WITH (FIELDTERMINATOR = ',', ROWTERMINATOR = '\n');
```


**Multi-row INSERT (batched):**

```sql
INSERT INTO table_name (column1, column2) VALUES
    (1, 'a'), (2, 'b'), ..., (1000, 'z');
-- Repeat in batches of 500–1,000.
```


**Component breakdown:**

| Technique | RDBMS | Speed relative to single-row INSERT |
|-----------|-------|-------------------------------------|
| Multi-row `VALUES` | All | ~10–50× faster |
| `INSERT ... SELECT` | All | Fast for server-side sources |
| `COPY` | PostgreSQL | Fastest; writes directly to heap |
| `LOAD DATA INFILE` | MySQL | Very fast; optimised for bulk loading |
| `BULK INSERT` / `bcp` | SQL Server | Very fast; native bulk-copy API |
| `SqlBulkCopy` | SQL Server | Fast from application code |

**Syntax rules:**

- Batch multi-row `VALUES` to 500–1,000 rows to avoid parameter limits and memory pressure.
- `COPY` and `LOAD DATA` typically require file access on the server or a client-side streaming mechanism.

**Constraints and limitations:**

- `COPY` and `LOAD DATA` are not standard SQL and are RDBMS-specific.
- Bulk utilities may bypass some constraint checks or trigger firing by default; configure options explicitly.
- SQL Server's `INSERT ... VALUES` has a 1,000-row limit per statement.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – COPY from CSV

```sql
-- Step 1: Create a table.
CREATE TABLE employees (id int, name text, salary numeric);

-- Step 2: Load data from a CSV file on the server.
COPY employees (id, name, salary)
FROM '/tmp/employees.csv'
WITH (FORMAT csv, HEADER true);

-- Step 3: Verify the row count.
SELECT count(*) FROM employees;
```

**Expected output:**

```text
CREATE TABLE
COPY 100000
 count
--------
 100000
(1 row)
```

**Why this output appears**: `COPY` reads the CSV file directly on the server and inserts all rows in a single operation, bypassing the per-row overhead of `INSERT`.

#### Real-World Cases

- **Data warehouse ETL**: Nightly jobs use `COPY` or `LOAD DATA` to load millions of fact rows into the warehouse.
- **Initial migration**: A one-time migration uses `bcp` or `COPY` to move data from a legacy system.

---

### 17. Insert Performance

#### Definitions

**Core Definition**
Insert performance is the measure of how quickly and efficiently an `INSERT` operation completes, influenced by round-trip overhead, statement complexity, indexing, constraints, and the chosen insertion technique.

**Technical Definition**
Insert performance is dominated by several factors: (1) **round-trip overhead**—each statement incurs network and parsing costs, so batching rows reduces this per-row; (2) **index maintenance**—every `INSERT` updates all indexes on the table, so tables with many indexes are slower to insert into; (3) **constraint checking**—`UNIQUE`, `PRIMARY KEY`, and `FOREIGN KEY` constraints require index lookups; (4) **WAL/redo logging**—durable databases write to a write-ahead log, so `synchronous_commit` and `fsync` settings affect throughput; and (5) **lock contention**—concurrent inserts into the same table or index can block. Benchmarks show single-row inserts are 50–130× slower than bulk inserts for 1,000 rows, and multi-row inserts are 1.3–57× slower than the fastest bulk utility depending on the RDBMS.

**Beginner-Friendly Explanation**
Insert performance is about how fast you can add data. Inserting one row at a time is slow because of all the overhead. Inserting many rows at once, or using a special bulk-loading tool, is much faster. Indexes and constraints also slow down inserts because the database has to do extra work to maintain them.

#### Purposes

- To choose the right insertion technique for the data volume and latency requirements.
- To identify and remove bottlenecks in insert-heavy workloads.
- To balance data integrity (constraints, indexes) against write throughput.

#### Syntax Rules and Structure

Performance is not controlled by a single syntax, but by technique selection and configuration:

- **Batch multi-row `VALUES`** to 500–1,000 rows per statement.
- **Use `INSERT ... SELECT`** when the source data is already in the database.
- **Use `COPY` / `LOAD DATA` / `bcp`** for large external datasets.
- **Minimise indexes** on insert-heavy tables; add indexes only when read performance requires them.
- **Consider `UNLOGGED` tables** (PostgreSQL) or `TABLOCK` hints (SQL Server) for disposable data.
- **Disable constraint checks** during bulk loads where appropriate (e.g., `SET FOREIGN_KEY_CHECKS = 0` in MySQL).

**Constraints and limitations:**

- Dropping indexes during bulk load and recreating them afterwards can be faster than maintaining them during the load, but it requires exclusive access to the table.
- `synchronous_commit = off` (PostgreSQL) or `innodb_flush_log_at_trx_commit = 0` (MySQL) increases insert throughput at the cost of durability.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Comparing single-row vs. multi-row insert

```sql
-- Step 1: Create a table.
CREATE TABLE bench (id serial PRIMARY KEY, data text);

-- Step 2: Insert 10,000 rows one at a time (slow).
\timing on
INSERT INTO bench (data) SELECT md5(random()::text) FROM generate_series(1, 10000);
-- (This is one INSERT ... SELECT, but the contrast is with 10,000 separate INSERTs.)

-- Step 3: Insert 10,000 rows via COPY (fast).
-- (Assume /tmp/bench.csv contains 10,000 lines.)
COPY bench (data) FROM '/tmp/bench.csv';
```

**Expected output** (illustrative):

```text
CREATE TABLE
Time: 120.456 ms
COPY 10000
Time: 35.123 ms
```

**Why this output appears**: The `COPY` operation bypasses per-row parsing, planning, and network overhead, making it significantly faster than row-by-row insertion.

#### Real-World Cases

- **High-throughput ingestion**: A logging pipeline uses `COPY` or `LOAD DATA` to sustain thousands of inserts per second.
- **OLTP application**: An application uses parameterised single-row inserts for user-facing operations but batches background jobs for efficiency.

---

## References

- PostgreSQL Documentation – INSERT – https://www.postgresql.org/docs/current/sql-insert.html
- PostgreSQL Documentation – Populating a Table With Rows – https://www.postgresql.org/docs/current/tutorial-populate.html
- PostgreSQL Documentation – Returning Data from Modified Rows – https://www.postgresql.org/docs/current/dml-returning.html
- PostgreSQL Documentation – COPY – https://www.postgresql.org/docs/current/sql-copy.html
- MySQL 8.0 Reference Manual – INSERT Statement – https://dev.mysql.com/doc/refman/8.0/en/insert.html
- MySQL 8.0 Reference Manual – INSERT ... ON DUPLICATE KEY UPDATE – https://dev.mysql.com/doc/refman/8.0/en/insert-on-duplicate.html
- MySQL 8.0 Reference Manual – LOAD DATA INFILE – https://dev.mysql.com/doc/refman/8.0/en/load-data.html
- Microsoft Learn – INSERT (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/insert-transact-sql
- Microsoft Learn – MERGE (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql
- Microsoft Learn – BULK INSERT (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/bulk-insert-transact-sql
- Microsoft Learn – OUTPUT Clause – https://learn.microsoft.com/en-us/sql/t-sql/queries/output-clause-transact-sql
- ServiceStack – Bulk Inserts – https://docs.servicestack.net/ormlite/bulk-inserts
- ServiceStack – Bulk Insert Performance Benchmarks – https://servicestack.net/posts/bulk-insert-performance
- Bytebase – Upsert in SQL: INSERT ON DUPLICATE KEY vs ON CONFLICT Explained – https://www.bytebase.com/blog/upsert-in-sql/
- SQLiz – SQL Bulk and Batch Insert by Database: Syntax, Limits & Performance – https://www.sqliz.com/bulk-insert/
- MySQL 8.0 Reference Manual – PRIMARY KEY and UNIQUE Index Constraints (INSERT IGNORE) – https://dev.mysql.com/doc/refman/8.0/en/create-table.html