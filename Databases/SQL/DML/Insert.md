# SQL INSERT Operations: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The SQL `INSERT` statement is the primary Data Manipulation Language (DML) command used to add new rows of data into a table, view, or other database object in a relational database management system (RDBMS).

**Technical Definition:** Per the ISO/IEC 9075 standard, an `<insert statement>` consists of `INSERT INTO <table name> [ <insert column list> ] <insert source>`, where the `<insert source>` is either a `<query expression>` (typically a `VALUES` clause or a `SELECT` statement) or `DEFAULT VALUES`. The statement adds one or more rows to the target table, with each row conforming to the table's column constraints and data types.

**Beginner-Friendly Explanation:** `INSERT` is the command you use to add new data to a database table. Think of a table as a spreadsheet; `INSERT` is how you add a new row to that spreadsheet. You tell the database which table to add to, which columns you're providing values for, and what those values are.

### Key Characteristics

- **Row-oriented:** Inserts one or more complete rows into a table.
- **Non-destructive to existing data:** Existing rows are not modified by a standard `INSERT` (unless an upsert clause is used).
- **Constraint-aware:** The database enforces all column constraints (NOT NULL, UNIQUE, CHECK, FOREIGN KEY) during insertion.
- **Transactional:** A single `INSERT` statement is atomic — it either inserts all specified rows or none at all.
- **Composable:** Can insert values from literals, expressions, or the result of a `SELECT` query.
- **Support for defaults:** Columns can be omitted, in which case default values (or `NULL`) are used.

### Prerequisites

- **Database access:** A connection to an RDBMS with `INSERT` privilege on the target table.
- **Table existence:** The target table must already exist with a defined schema.
- **Data type awareness:** Understanding of the table's column data types to provide compatible values.
- **Constraint knowledge:** Awareness of PRIMARY KEY, UNIQUE, FOREIGN KEY, CHECK, and NOT NULL constraints.
- **Transaction concepts:** Basic understanding of transactions and ACID properties for multi-statement operations.

### Related Programming Areas

- **Application Development:** Backend services insert user data, orders, logs, and records.
- **Data Engineering (ETL):** Extract-Transform-Load pipelines insert data into staging and warehouse tables.
- **Database Administration:** DBAs insert seed data, configuration values, and audit records.
- **Data Analysis:** Analysts insert computed results into summary tables.
- **Testing:** Test suites insert fixture data to set up known states.

### Core Concepts / Features

1. `INSERT INTO` Basic Syntax
2. Inserting a Single Row (`VALUES` Clause)
3. Inserting Multiple Rows in a Single Statement (Bulk Insertion)
4. Explicit Column Lists vs. Implicit Positional Inserts
5. Value Specifications
6. Inserting Default Values (`DEFAULT` Keyword or Omitting Columns)
7. Inserting `NULL` Values Explicitly
8. Copying Data from Another Table (`INSERT INTO ... SELECT`)
9. Handling Generated Identifiers (Auto-increment, Identity, `SERIAL`, UUID)
10. Retrieving Generated Keys (`RETURNING`, `OUTPUT`, `SCOPE_IDENTITY()`, `LAST_INSERT_ID()`)
11. Handling Constraint Violations (Foreign Key, Unique, Check)
12. Upsert Operations (`ON CONFLICT`, `ON DUPLICATE KEY UPDATE`, `MERGE`)
13. Transaction Safety During Failures (All-or-Nothing Atomicity)

---

## 1. INSERT INTO Basic Syntax

### Definitions

**Core Definition:** The `INSERT INTO` statement is the SQL command that adds one or more new rows to a table, specifying the target table, the columns to populate, and the values to insert.

**Technical Definition:** In standard SQL (ISO/IEC 9075-2), the `<insert statement>` is defined as `INSERT INTO <table name> [ ( <insert column list> ) ] <insert source>`, where `<insert source>` is either `<query expression>` or `DEFAULT VALUES`. The optional column list specifies which columns receive explicit values; columns not listed receive their default values or `NULL` if no default is defined.

**Beginner-Friendly Explanation:** `INSERT INTO` is the starting phrase of every insert command. It tells the database: "I want to put new data into this table." You then specify which columns you're filling and what values to use.

### Purposes

- To add new rows of data to an existing database table.
- To populate a newly created table with initial data.
- To store application-generated data (user registrations, orders, logs) in the database.
- To load data from external sources into the database.
- To create test fixtures and seed data for development and testing.

### Syntax Rules and Structure

**Complete General Syntax (PostgreSQL):**

```sql
[ WITH [ RECURSIVE ] with_query [, ...] ]
INSERT INTO table_name [ AS alias ]
    [ ( column_name [, ...] ) ]
    [ OVERRIDING { SYSTEM | USER } VALUE ]
    { DEFAULT VALUES
    | VALUES ( { expression | DEFAULT } [, ...] ) [, ...]
    | query }
    [ ON CONFLICT [ conflict_target ] conflict_action ]
    [ RETURNING [ WITH ( { OLD | NEW } AS output_alias [, ...] ) ]
                { * | output_expression [ [ AS ] output_name ] } [, ...] ]
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `INSERT INTO` | Initiates the insert operation; `INTO` is optional in some dialects |
| `table_name` | The target table receiving the rows |
| `( column_name, ... )` | Optional list of columns to populate |
| `VALUES (...)` | Explicit values for a single or multiple rows |
| `DEFAULT VALUES` | Inserts a row where every column uses its default |
| `query` | A `SELECT` statement whose results are inserted |
| `ON CONFLICT` | Alternative action when a constraint violation would occur |
| `RETURNING` | Returns values from the inserted rows |

**Syntax Rules:**

- The `INTO` keyword is optional in MySQL, SQL Server, and SQLite but is required by PostgreSQL and Oracle.
- If the column list is omitted, values must be provided for all columns in table-definition order, or the first N columns if fewer values are supplied (PostgreSQL extension).
- The number of values in the `VALUES` clause must match the number of columns in the explicit or implicit column list.
- The `DEFAULT VALUES` form cannot be combined with a column list.
- `RETURNING` is supported by PostgreSQL, Oracle (as `RETURNING INTO`), and SQL Server (as `OUTPUT`).

**Constraints and Limitations:**

- The target table must exist and the user must have `INSERT` privilege.
- Values must be compatible with the column data types; implicit type conversion may occur but is not guaranteed.
- `INSERT INTO ... SELECT` cannot be used to insert into a table that is also being selected from in some dialects (though PostgreSQL allows it).
- `DEFAULT VALUES` inserts exactly one row; it cannot be used for bulk insertion.
- The `OVERRIDING` clause is PostgreSQL-specific and controls identity column behaviour.

### Annotated Complete Code Examples

**Example 1: Basic INSERT with VALUES (PostgreSQL)**

```sql
-- Setup: Create a sample table
CREATE TABLE products (
    product_no  INTEGER PRIMARY KEY,
    name        TEXT NOT NULL,
    price       NUMERIC(10, 2) DEFAULT 0.00
);

-- Insert a single row using positional values
INSERT INTO products VALUES (1, 'Cheese', 9.99);

-- Expected Output:
-- INSERT 0 1
```

**Why this output occurs:** The `VALUES (1, 'Cheese', 9.99)` clause provides values for all three columns in the order they were defined in the `CREATE TABLE` statement. The database inserts one row and returns the command tag `INSERT 0 1`, indicating one row was inserted.

**Example 2: INSERT with Explicit Column List**

```sql
-- Insert using an explicit column list (recommended practice)
INSERT INTO products (product_no, name, price)
VALUES (2, 'Bread', 1.99);

-- Verify the insert
SELECT * FROM products;

-- Expected Output:
--  product_no |  name  | price
-- ------------+--------+-------
--           1 | Cheese |  9.99
--           2 | Bread  |  1.99
```

**Why this output occurs:** The column list `(product_no, name, price)` explicitly names the columns receiving values. This is equivalent to the positional insert but is more robust — if the table schema changes (e.g., a column is added), the query still works correctly because it does not rely on column order.

### Real-World Cases

**Case 1: User Registration**

A web application inserts a new user record when someone signs up: `INSERT INTO users (email, password_hash, created_at) VALUES ('user@example.com', '$2b$12$...', NOW())`. The `INSERT` statement stores the registration data.

**Case 2: Order Processing**

An e-commerce system inserts an order record and its line items into separate tables within a transaction. Each `INSERT` adds one row to the respective table.

### References

- PostgreSQL Documentation — INSERT - https://www.postgresql.org/docs/current/sql-insert.html
- MySQL 8.4 Reference Manual — INSERT Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/insert.html
- Microsoft SQL Server — INSERT (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/insert-transact-sql
- Oracle Database SQL Language Reference — INSERT - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/INSERT.html
- SQLite Documentation — INSERT - https://www2.sqlite.org/lang_insert.html

---

## 2. Inserting a Single Row (VALUES Clause)

### Definitions

**Core Definition:** Inserting a single row using the `VALUES` clause adds exactly one new row to a table, specifying a value for each column in the column list.

**Technical Definition:** The `<insert statement>` form `INSERT INTO table ( columns ) VALUES ( values )` creates one row. The `VALUES` clause contains a parenthesised list of expressions, each corresponding to a column in the explicit or implicit column list. The number of expressions must equal the number of columns.

**Beginner-Friendly Explanation:** This is the most basic way to add data: you write one set of values in parentheses, and the database adds one row.

### Purposes

- To add a single record to a table.
- To test queries with known data during development.
- To insert configuration or lookup values.
- To store individual user actions or events.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
INSERT INTO table_name [ ( column1 [, column2, ...] ) ]
VALUES ( value1 [, value2, ...] );
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `INSERT INTO table_name` | Target table |
| `( column1, column2, ... )` | Optional column list |
| `VALUES ( value1, value2, ... )` | Values for the corresponding columns |

**Syntax Rules:**

- Values are enclosed in parentheses and separated by commas.
- String values must be enclosed in single quotes.
- Numeric values are unquoted.
- `NULL` is written as the keyword `NULL` (unquoted).
- The `DEFAULT` keyword can be used to explicitly request a column's default value.
- Date and time values are typically written as strings in ISO format (`'YYYY-MM-DD'`).

**Constraints and Limitations:**

- The number of values must exactly match the number of columns in the column list (or all columns if the list is omitted).
- Values must be type-compatible with their target columns.
- Not all columns need to be specified; omitted columns receive defaults or `NULL`.
- Some RDBMSs require `INTO` (PostgreSQL, Oracle); others make it optional (MySQL, SQL Server, SQLite).

### Annotated Complete Code Examples

**Example 1: Single Row Insert**

```sql
-- Setup
CREATE TABLE employees (
    id          SERIAL PRIMARY KEY,
    first_name  VARCHAR(50) NOT NULL,
    last_name   VARCHAR(50) NOT NULL,
    department  VARCHAR(50) DEFAULT 'Unassigned',
    salary      NUMERIC(10, 2)
);

-- Insert a single row
INSERT INTO employees (first_name, last_name, department, salary)
VALUES ('Alice', 'Johnson', 'Engineering', 95000.00);

-- Verify
SELECT * FROM employees;

-- Expected Output:
--  id | first_name | last_name | department  |  salary
-- ----+------------+-----------+-------------+----------
--   1 | Alice      | Johnson   | Engineering | 95000.00
```

**Why this output occurs:** The `VALUES` clause provides four values corresponding to the four columns in the column list. The `id` column is not specified because it is a `SERIAL` column that auto-generates its value. The database assigns `id = 1` and inserts the row.

### Real-World Cases

**Case 1: Audit Logging**

An application inserts a single audit record for each user action: `INSERT INTO audit_log (user_id, action, timestamp) VALUES (42, 'LOGIN', NOW())`.

**Case 2: Configuration Management**

A deployment script inserts a configuration value: `INSERT INTO settings (key, value) VALUES ('max_connections', '100')`.

### References

- PostgreSQL Documentation — Inserting Data - https://www.postgresql.org/docs/current/dml-insert.html
- MySQL 8.4 Reference Manual — INSERT ... VALUES - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/insert.html

---

## 3. Inserting Multiple Rows in a Single Statement (Bulk Insertion)

### Definitions

**Core Definition:** Bulk insertion using a single `INSERT` statement adds multiple rows to a table by providing multiple parenthesised value lists in the `VALUES` clause, separated by commas.

**Technical Definition:** The `VALUES` clause can contain multiple `<row constructor>` elements: `VALUES ( ... ), ( ... ), ( ... )`. Each parenthesised list represents one row. The database inserts all rows as part of a single statement, which is atomic — either all rows are inserted or none are.

**Beginner-Friendly Explanation:** Instead of writing multiple `INSERT` statements, you can write one statement with multiple sets of values. This is faster and more efficient, especially when inserting many rows.

### Purposes

- To insert many rows efficiently with a single statement.
- To reduce network round-trips between the application and database.
- To ensure atomicity when inserting a batch of related rows.
- To load seed data or reference data in bulk.
- To improve performance compared to multiple single-row inserts.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
INSERT INTO table_name [ ( column1 [, column2, ...] ) ]
VALUES
    ( value1a [, value2a, ...] ),
    ( value1b [, value2b, ...] ),
    ( value1c [, value2c, ...] ),
    ...;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `VALUES` | Introduces one or more row constructors |
| `( ... ), ( ... )` | Each parenthesised group is one row |
| Commas between groups | Separate individual rows |

**Syntax Rules:**

- Each row constructor must have the same number of values as the column list (or all columns if omitted).
- All rows must have values of compatible types in the same positions.
- The entire statement is a single transaction unit (atomic).
- PostgreSQL, MySQL, SQL Server, Oracle, and SQLite all support multi-row `VALUES`.

**Constraints and Limitations:**

- There is an implementation-dependent limit on the number of rows per statement (e.g., 1,000 for SQL Server's `VALUES` clause, though this can be exceeded with `INSERT ... SELECT`).
- Very large multi-row inserts consume memory and can lock the table for the duration.
- For very large datasets, `COPY` (PostgreSQL) or `BULK INSERT` (SQL Server) are more efficient than multi-row `INSERT`.
- MySQL's `max_allowed_packet` limits the total size of the statement.

### Annotated Complete Code Examples

**Example 1: Multi-Row Insert**

```sql
-- Insert three rows in one statement
INSERT INTO employees (first_name, last_name, department, salary)
VALUES
    ('Bob',   'Smith',    'Marketing',   72000.00),
    ('Carol', 'Williams', 'Engineering', 105000.00),
    ('David', 'Brown',    'Sales',       68000.00);

-- Verify
SELECT * FROM employees ORDER BY id;

-- Expected Output:
--  id | first_name | last_name | department  |  salary
-- ----+------------+-----------+-------------+----------
--   1 | Alice      | Johnson   | Engineering | 95000.00
--   2 | Bob        | Smith     | Marketing   | 72000.00
--   3 | Carol      | Williams  | Engineering |105000.00
--   4 | David      | Brown     | Sales       | 68000.00
```

**Why this output occurs:** Three separate row constructors are provided in a single `VALUES` clause. The database inserts all three rows in one operation. Each row receives an auto-generated `id` (2, 3, 4) because the `SERIAL` column continues from its previous value.

### Real-World Cases

**Case 1: Importing CSV Data**

A data import tool reads a CSV file and constructs a single `INSERT` statement with hundreds of `VALUES` rows, reducing the number of database round-trips from hundreds to one.

**Case 2: Creating Test Fixtures**

A test suite inserts multiple rows of test data in a single statement to set up a known database state before running assertions.

### References

- PostgreSQL Documentation — Inserting Data (Multiple Rows) - https://www.postgresql.org/docs/current/dml-insert.html
- MySQL 8.4 Reference Manual — INSERT ... VALUES - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/insert.html
- Microsoft SQL Server — INSERT (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/insert-transact-sql

---

## 4. Explicit Column Lists vs. Implicit Positional Inserts

### Definitions

**Core Definition:** An explicit column list names the columns receiving values in the `INSERT` statement, while an implicit positional insert omits the column list and relies on the table's column-definition order.

**Technical Definition:** When the `<insert column list>` is omitted, the values in the `VALUES` clause are associated with the table's columns in their declared order (left-to-right). When the column list is provided, values are associated with the named columns regardless of their position in the table definition.

**Beginner-Friendly Explanation:** You can either tell the database exactly which columns you're filling (explicit column list) or let it figure it out based on the order the columns were defined (implicit). The explicit approach is safer and more readable.

### Purposes

- To make `INSERT` statements robust against schema changes (explicit).
- To reduce typing for quick ad-hoc inserts (implicit).
- To insert values for only some columns, leaving others to defaults (explicit).
- To control which columns receive values and in what order (explicit).

### Syntax Rules and Structure

**Explicit Column List:**

```sql
INSERT INTO table_name (column1, column2, column3)
VALUES (value1, value2, value3);
```

**Implicit Positional Insert:**

```sql
INSERT INTO table_name
VALUES (value1, value2, value3);
```

**Component Breakdown:**

| Form | Columns Specified | Value-to-Column Mapping |
|------|------------------|------------------------|
| Explicit | User-provided list | By name, in listed order |
| Implicit | All table columns | By position in table definition |

**Syntax Rules:**

- With an explicit column list, the order of columns in the list determines the mapping of values.
- With an implicit insert, the number of values must equal the total number of columns in the table (unless using a PostgreSQL extension that defaults remaining columns).
- The `DEFAULT` keyword can be used for any column in an explicit list to request the default value.
- Omitting a column from an explicit list causes it to receive its default value or `NULL`.

**Constraints and Limitations:**

- Implicit inserts break when columns are added, removed, or reordered in the table.
- Explicit column lists are strongly recommended for production code.
- PostgreSQL allows omitting trailing columns in an implicit insert (the remaining columns are defaulted); this is a PostgreSQL extension and not standard SQL.
- Oracle and SQL Server require all columns to be specified in an implicit insert (unless the omitted columns are nullable or have defaults, in which case the column list must be used).

### Annotated Complete Code Examples

**Example 1: Explicit Column List (Recommended)**

```sql
-- Explicit column list — robust against schema changes
INSERT INTO employees (first_name, last_name, salary)
VALUES ('Eve', 'Davis', 88000.00);

-- Verify
SELECT * FROM employees WHERE first_name = 'Eve';

-- Expected Output:
--  id | first_name | last_name | department  |  salary
-- ----+------------+-----------+-------------+----------
--   5 | Eve        | Davis     | Unassigned  | 88000.00
```

**Why this output occurs:** The column list explicitly names `first_name`, `last_name`, and `salary`. The `department` column is omitted, so it receives its default value `'Unassigned'`. The `id` column auto-generates its value. Even if the table's column order changes (e.g., `department` is moved before `salary`), this query still works correctly.

**Example 2: Implicit Positional Insert**

```sql
-- Implicit positional insert — must provide values for all columns
INSERT INTO employees
VALUES (6, 'Frank', 'Miller', 'HR', 75000.00);

-- Verify
SELECT * FROM employees WHERE first_name = 'Frank';

-- Expected Output:
--  id | first_name | last_name | department |  salary
-- ----+------------+-----------+------------+----------
--   6 | Frank      | Miller    | HR         | 75000.00
```

**Why this output occurs:** No column list is provided, so the five values correspond positionally to the five columns in the table definition order: `id`, `first_name`, `last_name`, `department`, `salary`. The `id` is explicitly provided as 6 rather than allowing the `SERIAL` to auto-generate it. This works but is fragile — if a column is added or reordered, the query breaks.

### Real-World Cases

**Case 1: Production Application Code**

A backend API always uses explicit column lists (`INSERT INTO orders (customer_id, total, status) VALUES (...)`) to remain resilient to schema migrations. When the `orders` table gains a new `discount` column, the existing code continues to work because it names only the columns it populates.

**Case 2: Quick Ad-Hoc Queries**

A developer testing a query in a SQL console uses `INSERT INTO logs VALUES (1, 'test', NOW())` for speed, knowing the table's exact column order.

### References

- PostgreSQL Documentation — Inserting Data (Column Lists) - https://www.postgresql.org/docs/current/dml-insert.html
- MySQL 8.4 Reference Manual — INSERT Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/insert.html

---

## 5. Value Specifications

### Definitions

**Core Definition:** Value specifications in an `INSERT` statement define the actual data values to be inserted into each column, which can be literals, expressions, function calls, or the `DEFAULT` keyword.

**Technical Definition:** In the `<insert source>` of an `INSERT` statement, each column value is specified as an `<expression>`, which can be a literal constant, a scalar expression (arithmetic, string concatenation), a function call, or the keyword `DEFAULT`. The expression is evaluated in the context of the current statement before being stored in the target column.

**Beginner-Friendly Explanation:** A value specification is simply the value you want to put into a column. It can be a number, a string, a date, a calculation, a function result, or the special word `DEFAULT` to let the database choose.

### Purposes

- To provide literal constant values for insertion.
- To compute values at insert time using expressions.
- To use database functions (e.g., `NOW()`, `UPPER()`, `gen_random_uuid()`) to generate values.
- To explicitly request column default values with the `DEFAULT` keyword.
- To insert `NULL` values where appropriate.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
INSERT INTO table_name (column1, column2, column3)
VALUES (
    literal_or_expression,
    expression,
    DEFAULT
);
```

**Value Types and Examples:**

| Value Type | Syntax | Example |
|-----------|--------|---------|
| String literal | `'text'` | `'Alice'` |
| Numeric literal | `123`, `3.14` | `95000.00` |
| Boolean literal | `TRUE`, `FALSE` | `TRUE` |
| NULL | `NULL` | `NULL` |
| DEFAULT | `DEFAULT` | `DEFAULT` |
| Expression | `expr op expr` | `100 * 1.1` |
| Function call | `func(args)` | `NOW()`, `UPPER('abc')` |
| Subquery | `(SELECT ...)` | `(SELECT MAX(id) FROM t)` |

**Syntax Rules:**

- String literals are enclosed in single quotes.
- Single quotes within a string are escaped by doubling them: `'It''s'`.
- Numeric literals are unquoted.
- `NULL` and `DEFAULT` are keywords, not strings.
- Expressions can reference the target table's columns (useful in `INSERT ... SELECT`).
- Subqueries must return exactly one value (scalar subquery) when used as a value.

**Constraints and Limitations:**

- The `DEFAULT` keyword cannot be used in an implicit positional insert for a column that has no default defined.
- Subqueries in `VALUES` cannot reference the target table of the `INSERT` (no correlated subqueries in `VALUES`).
- Function calls may have side effects (e.g., sequence advancement); use with caution.
- Type compatibility is enforced; implicit conversion may occur but can fail.

### Annotated Complete Code Examples

**Example 1: Literal, Expression, and Default Values**

```sql
-- Setup
CREATE TABLE orders (
    order_id    SERIAL PRIMARY KEY,
    customer    VARCHAR(50) NOT NULL,
    quantity    INTEGER NOT NULL DEFAULT 1,
    unit_price  NUMERIC(10, 2) NOT NULL,
    total       NUMERIC(12, 2) GENERATED ALWAYS AS (quantity * unit_price) STORED,
    order_date  DATE DEFAULT CURRENT_DATE
);

-- Insert using literals, expressions, and DEFAULT
INSERT INTO orders (customer, quantity, unit_price)
VALUES ('Alice', 3, 29.99);

-- Verify
SELECT * FROM orders;

-- Expected Output:
--  order_id | customer | quantity | unit_price | total  | order_date
-- ----------+----------+----------+------------+--------+------------
--         1 | Alice    |        3 |      29.99 |  89.97 | 2026-09-19
```

**Why this output occurs:** The `customer` column receives the literal `'Alice'`. The `quantity` column receives the literal `3`. The `unit_price` column receives the literal `29.99`. The `total` column is a generated column that automatically computes `3 * 29.99 = 89.97`. The `order_date` column uses its default `CURRENT_DATE`. The `order_id` auto-generates via `SERIAL`.

**Example 2: Using Functions in VALUES**

```sql
-- Insert using function calls
INSERT INTO employees (first_name, last_name, department, salary)
VALUES (UPPER('grace'), UPPER('hopper'), 'Engineering', 110000.00);

-- Verify
SELECT first_name, last_name FROM employees WHERE salary = 110000.00;

-- Expected Output:
--  first_name | last_name
-- ------------+-----------
--  GRACE      | HOPPER
```

**Why this output occurs:** The `UPPER()` function converts the string literals to uppercase before insertion. The database stores `'GRACE'` and `'HOPPER'` in the respective columns.

### Real-World Cases

**Case 1: Timestamp Generation**

An application inserts a log entry with `NOW()` as the timestamp value: `INSERT INTO logs (message, created_at) VALUES ('User login', NOW())`. The database function generates the current timestamp.

**Case 2: Computed Totals**

An e-commerce system inserts an order line with `quantity * unit_price` as the total: `INSERT INTO order_lines (order_id, product_id, quantity, unit_price, total) VALUES (1, 42, 2, 19.99, 2 * 19.99)`.

### References

- PostgreSQL Documentation — Value Expressions - https://www.postgresql.org/docs/current/sql-expressions.html
- MySQL 8.4 Reference Manual — INSERT ... VALUES - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/insert.html
- Oracle Database SQL Language Reference — INSERT - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/INSERT.html

---

## 6. Inserting Default Values (DEFAULT Keyword or Omitting Columns)

### Definitions

**Core Definition:** Default value insertion allows a column to receive its predefined default value — either by omitting the column from the `INSERT` statement's column list or by explicitly specifying the `DEFAULT` keyword in the `VALUES` clause.

**Technical Definition:** When a column is not present in the `<insert column list>`, it is assigned its declared default value (from `CREATE TABLE ... DEFAULT`) or `NULL` if no default is defined. The `DEFAULT` keyword can be used in the `VALUES` clause for any column to explicitly request the default. The `DEFAULT VALUES` form inserts a row where every column receives its default.

**Beginner-Friendly Explanation:** If you don't have a value for a column, you can skip it or write `DEFAULT`, and the database will use the column's default value (like an automatic timestamp or a status of 'active').

### Purposes

- To insert rows without specifying values for every column.
- To use database-defined defaults (e.g., timestamps, UUIDs, status flags).
- To explicitly indicate that a column should receive its default value.
- To insert a row where all columns use defaults with `DEFAULT VALUES`.
- To separate schema-defined behaviour from application logic.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
-- Omit columns (they receive defaults)
INSERT INTO table_name (col1, col2) VALUES (val1, val2);

-- Explicit DEFAULT keyword
INSERT INTO table_name (col1, col2, col3)
VALUES (val1, DEFAULT, val2);

-- All columns default
INSERT INTO table_name DEFAULT VALUES;
```

**Component Breakdown:**

| Form | Behaviour |
|------|-----------|
| Omitted column | Receives declared default or NULL |
| `DEFAULT` keyword | Explicitly requests the column's default value |
| `DEFAULT VALUES` | Inserts one row with all columns defaulted |

**Syntax Rules:**

- Columns omitted from the column list receive their defaults.
- The `DEFAULT` keyword can be used in any position in the `VALUES` clause.
- `DEFAULT VALUES` cannot be combined with a column list or a `VALUES` clause.
- If a column has no declared default and is omitted, it receives `NULL` (unless a `NOT NULL` constraint exists, in which case an error occurs).

**Constraints and Limitations:**

- `DEFAULT VALUES` inserts exactly one row; it cannot be used for multi-row insertion.
- If a column has a `NOT NULL` constraint and no default, omitting it causes an error.
- The `DEFAULT` keyword is not supported in all contexts in all dialects (e.g., older MySQL versions).
- Default expressions are evaluated at insert time; they can reference functions like `NOW()` or `CURRENT_TIMESTAMP`.

### Annotated Complete Code Examples

**Example 1: Omitting Columns to Use Defaults**

```sql
-- Setup: Table with defaults
CREATE TABLE tasks (
    task_id     SERIAL PRIMARY KEY,
    title       VARCHAR(100) NOT NULL,
    status      VARCHAR(20) DEFAULT 'pending',
    created_at  TIMESTAMP DEFAULT NOW(),
    priority    INTEGER DEFAULT 3
);

-- Insert omitting status, created_at, and priority
INSERT INTO tasks (title) VALUES ('Review PR #42');

-- Verify
SELECT * FROM tasks;

-- Expected Output:
--  task_id |     title      | status  |        created_at         | priority
-- ---------+----------------+---------+---------------------------+----------
--        1 | Review PR #42  | pending | 2026-09-19 10:30:00.12345 |        3
```

**Why this output occurs:** Only the `title` column is specified. The `status` column receives its default `'pending'`. The `created_at` column receives the current timestamp from `NOW()`. The `priority` column receives its default `3`. The `task_id` auto-generates via `SERIAL`.

**Example 2: Explicit DEFAULT Keyword**

```sql
-- Insert with explicit DEFAULT for a column
INSERT INTO tasks (title, status, priority)
VALUES ('Fix login bug', DEFAULT, 5);

-- Verify
SELECT title, status, priority FROM tasks WHERE title = 'Fix login bug';

-- Expected Output:
--      title      | status  | priority
-- ----------------+---------+----------
--  Fix login bug  | pending |        5
```

**Why this output occurs:** The `status` column is explicitly set to `DEFAULT`, which evaluates to the column's default value `'pending'`. The `priority` column receives the explicit value `5`.

### Real-World Cases

**Case 1: Timestamp Auditing**

Every table has `created_at TIMESTAMP DEFAULT NOW()`. Application code never specifies this column; the database automatically records the insertion time.

**Case 2: Status Workflow**

An order table has `status VARCHAR(20) DEFAULT 'draft'`. When an order is first created, the application omits the `status` column, and the database assigns `'draft'` automatically.

### References

- PostgreSQL Documentation — Default Values - https://www.postgresql.org/docs/current/ddl-default.html
- MySQL 8.4 Reference Manual — Data Type Default Values - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/data-type-defaults.html
- Microsoft SQL Server — Default Constraints - https://learn.microsoft.com/en-us/sql/relational-databases/tables/specify-default-values-for-columns

---

## 7. Inserting NULL Values Explicitly

### Definitions

**Core Definition:** Explicit `NULL` insertion places the `NULL` marker into a column, representing the absence of a value, rather than allowing the column to receive its default.

**Technical Definition:** `NULL` is a special marker in SQL indicating that a value is unknown or missing. It is distinct from zero, an empty string, or any other value. Explicitly inserting `NULL` into a column requires the column to be nullable (not declared `NOT NULL`). If a column is omitted from the column list, it receives `NULL` only if no default is defined.

**Beginner-Friendly Explanation:** `NULL` means "no value" or "unknown." You can explicitly put `NULL` into a column to indicate that the data is missing or not applicable.

### Purposes

- To represent missing or unknown data explicitly.
- To insert rows where certain fields are not applicable.
- To distinguish "no value" from "default value."
- To set columns to `NULL` in update-like scenarios (though `INSERT` only adds new rows).

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
INSERT INTO table_name (column1, column2, column3)
VALUES (value1, NULL, value2);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `NULL` | Keyword representing the absence of a value |
| Column position | Must correspond to a nullable column |

**Syntax Rules:**

- `NULL` is written as an unquoted keyword, not as a string `'NULL'`.
- The target column must allow `NULL` (i.e., not have a `NOT NULL` constraint).
- `NULL` can be used in any position in the `VALUES` clause.
- Omitting a column with no default is equivalent to inserting `NULL` (if the column is nullable).

**Constraints and Limitations:**

- Inserting `NULL` into a `NOT NULL` column causes a constraint violation error.
- `NULL` is not equal to itself; `NULL = NULL` yields `UNKNOWN` in SQL's three-valued logic.
- Aggregate functions generally ignore `NULL` values.
- Primary key columns cannot contain `NULL`.

### Annotated Complete Code Examples

**Example 1: Explicit NULL Insertion**

```sql
-- Setup: Table with nullable column
CREATE TABLE contacts (
    contact_id  SERIAL PRIMARY KEY,
    name        VARCHAR(50) NOT NULL,
    email       VARCHAR(100),
    phone       VARCHAR(20)
);

-- Insert with explicit NULL for phone
INSERT INTO contacts (name, email, phone)
VALUES ('Alice', 'alice@example.com', NULL);

-- Verify
SELECT * FROM contacts;

-- Expected Output:
--  contact_id | name  |       email        | phone
-- ------------+-------+--------------------+-------
--           1 | Alice | alice@example.com  | (null)
```

**Why this output occurs:** The `phone` column is explicitly set to `NULL`, indicating that no phone number is available. The `email` column receives the string value `'alice@example.com'`. The `name` column receives `'Alice'`.

**Example 2: NULL in a NOT NULL Column (Error)**

```sql
-- Attempt to insert NULL into a NOT NULL column
INSERT INTO contacts (name, email, phone)
VALUES (NULL, 'bob@example.com', '555-1234');

-- Expected Error (PostgreSQL):
-- ERROR: null value in column "name" violates not-null constraint
-- DETAIL: Failing row contains (2, null, bob@example.com, 555-1234).
```

**Why this error occurs:** The `name` column is defined as `NOT NULL`, but `NULL` is explicitly provided. The database enforces the constraint and rejects the insertion, preventing the row from being added.

### Real-World Cases

**Case 1: Optional Form Fields**

A user registration form has an optional "referral code" field. If the user does not provide a code, the application inserts `NULL`: `INSERT INTO users (username, email, referral_code) VALUES ('alice', 'alice@example.com', NULL)`.

**Case 2: Missing Data in ETL**

An ETL pipeline loading data from a legacy system encounters missing values. It explicitly inserts `NULL` into columns where the source data is absent, preserving the distinction between "no data" and "zero."

### References

- PostgreSQL Documentation — NULL Values - https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-NOT-NULL
- Oracle Database SQL Language Reference — NULLs - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Nulls.html
- Microsoft SQL Server — NULL and UNKNOWN - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/null-and-unknown-transact-sql

---

## 8. Copying Data from Another Table (INSERT INTO ... SELECT)

### Definitions

**Core Definition:** `INSERT INTO ... SELECT` copies rows from one or more source tables into a target table, using a `SELECT` query to determine the data to insert.

**Technical Definition:** The `<insert source>` in an `INSERT` statement can be a `<query expression>` (i.e., a `SELECT` statement). The result set of the `SELECT` determines the rows and values inserted. The number and types of columns in the `SELECT` result must be compatible with the target table's column list. This form supports `WHERE`, `JOIN`, `GROUP BY`, `ORDER BY`, and `LIMIT` in the source query.

**Beginner-Friendly Explanation:** Instead of typing out values manually, you can copy data from one table into another by writing a `SELECT` query that retrieves the rows you want. The database inserts all the rows returned by that query.

### Purposes

- To copy data between tables with compatible schemas.
- To populate a new table from an existing one.
- To insert aggregated or transformed data into a summary table.
- To load data from one database into another (via linked servers or `dblink`).
- To create archive or backup copies of data.
- To insert data that requires complex filtering or joining.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
INSERT INTO target_table [ ( column1, column2, ... ) ]
SELECT expression1, expression2, ...
FROM source_table
[ WHERE condition ]
[ GROUP BY ... ]
[ HAVING ... ]
[ ORDER BY ... ]
[ LIMIT ... ];
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `INSERT INTO target_table` | Target table for insertion |
| `( column1, column2, ... )` | Optional target column list |
| `SELECT` | Source query determining what to insert |
| `FROM source_table` | Source table(s) to read from |
| `WHERE` | Filters source rows |

**Syntax Rules:**

- The number of columns in the `SELECT` result must match the number of columns in the target column list (or the target table's columns if no list is given).
- The data types of corresponding columns must be compatible.
- The target table cannot be the same table being selected from in some dialects (PostgreSQL allows it, but it can cause infinite loops if not carefully written).
- `ORDER BY` in the `SELECT` does not guarantee insertion order; it only affects the order of rows returned by the `SELECT` (which may be used with `LIMIT`).

**Constraints and Limitations:**

- `INSERT INTO ... SELECT` is atomic: either all rows are inserted or none are.
- Large `SELECT` queries can consume significant resources and lock the target table for the duration.
- Some RDBMSs do not allow `INSERT INTO ... SELECT` on a table that is also referenced in the `SELECT` in certain join scenarios.
- Oracle's `INSERT ... SELECT` with `ORDER BY` requires the `ORDER BY` to be in a subquery.

### Annotated Complete Code Examples

**Example 1: Copying Data Between Tables**

```sql
-- Setup: Create source and target tables
CREATE TABLE employees (
    id          SERIAL PRIMARY KEY,
    first_name  VARCHAR(50),
    last_name   VARCHAR(50),
    department  VARCHAR(50),
    salary      NUMERIC(10, 2)
);

INSERT INTO employees (first_name, last_name, department, salary) VALUES
    ('Alice', 'Johnson', 'Engineering', 95000.00),
    ('Bob',   'Smith',   'Marketing',   72000.00),
    ('Carol', 'Williams','Engineering', 105000.00),
    ('David', 'Brown',   'Sales',       68000.00);

CREATE TABLE engineering_archive (
    id          INTEGER,
    full_name   VARCHAR(100),
    salary      NUMERIC(10, 2)
);

-- Copy Engineering employees into archive
INSERT INTO engineering_archive (id, full_name, salary)
SELECT id,
       first_name || ' ' || last_name,
       salary
FROM employees
WHERE department = 'Engineering';

-- Verify
SELECT * FROM engineering_archive;

-- Expected Output:
--  id |   full_name   |  salary
-- ----+---------------+----------
--   1 | Alice Johnson | 95000.00
--   3 | Carol Williams|105000.00
```

**Why this output occurs:** The `SELECT` query retrieves employees in the Engineering department and concatenates their first and last names into a `full_name` column. The target table `engineering_archive` has columns `id`, `full_name`, and `salary`. Two rows match the `WHERE` condition and are inserted.

**Example 2: INSERT INTO ... SELECT with Aggregation**

```sql
-- Setup: Create a summary table
CREATE TABLE dept_summary (
    department   VARCHAR(50),
    headcount    INTEGER,
    avg_salary   NUMERIC(10, 2)
);

-- Insert aggregated data
INSERT INTO dept_summary (department, headcount, avg_salary)
SELECT department,
       COUNT(*),
       AVG(salary)
FROM employees
GROUP BY department;

-- Verify
SELECT * FROM dept_summary;

-- Expected Output:
--  department  | headcount | avg_salary
-- -------------+-----------+------------
--  Engineering |         2 |  100000.00
--  Marketing   |         1 |   72000.00
--  Sales       |         1 |   68000.00
```

**Why this output occurs:** The `SELECT` query groups employees by department, counts the number of employees in each group, and computes the average salary. Each group becomes one row in the `dept_summary` table.

### Real-World Cases

**Case 1: Data Warehousing**

A nightly ETL job copies data from a transactional `orders` table into a data warehouse's `fact_orders` table using `INSERT INTO ... SELECT`, applying transformations and filtering to load only the previous day's orders.

**Case 2: Archiving Old Records**

An archive process moves records older than one year from an `active_logs` table to an `archive_logs` table: `INSERT INTO archive_logs SELECT * FROM active_logs WHERE created_at < NOW() - INTERVAL '1 year'`. After insertion, the old rows are deleted from the active table.

### References

- PostgreSQL Documentation — INSERT INTO ... SELECT - https://www.postgresql.org/docs/current/sql-insert.html
- MySQL 8.4 Reference Manual — INSERT ... SELECT Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/insert-select.html
- Microsoft SQL Server — INSERT ... SELECT - https://learn.microsoft.com/en-us/sql/t-sql/statements/insert-transact-sql
- Oracle Database SQL Language Reference — INSERT ... SELECT - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/INSERT.html

---

## 9. Handling Generated Identifiers (Auto-increment, Identity, SERIAL, UUID)

### Definitions

**Core Definition:** Generated identifiers are column values automatically produced by the database when a new row is inserted, commonly used for primary keys to ensure uniqueness without application intervention.

**Technical Definition:** Generated identifiers are implemented through mechanisms such as `SERIAL` (PostgreSQL), `IDENTITY` (SQL Server, Oracle), `AUTO_INCREMENT` (MySQL), and UUID functions (`gen_random_uuid()`). These mechanisms use sequences or algorithms to generate unique values, which are assigned to the column during insertion if no explicit value is provided.

**Beginner-Friendly Explanation:** When you add a new row, you often need a unique ID number for it. Instead of making up a number yourself, you can let the database generate one automatically — like a ticket dispenser that always gives the next number.

### Purposes

- To automatically generate unique primary key values for new rows.
- To eliminate the need for application-side ID management.
- To ensure uniqueness across concurrent insertions.
- To provide a consistent, database-enforced identity for every row.
- To support referential integrity by providing stable identifiers for foreign keys.

### Syntax Rules and Structure

**PostgreSQL SERIAL:**

```sql
CREATE TABLE users (
    id   SERIAL PRIMARY KEY,
    name TEXT
);
-- Equivalent to:
CREATE SEQUENCE users_id_seq;
CREATE TABLE users (
    id   INTEGER PRIMARY KEY DEFAULT nextval('users_id_seq'),
    name TEXT
);
ALTER SEQUENCE users_id_seq OWNED BY users.id;
```

**PostgreSQL UUID:**

```sql
CREATE TABLE users (
    id   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT
);
```

**SQL Server IDENTITY:**

```sql
CREATE TABLE users (
    id   INT IDENTITY(1,1) PRIMARY KEY,
    name NVARCHAR(100)
);
```

**MySQL AUTO_INCREMENT:**

```sql
CREATE TABLE users (
    id   INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100)
);
```

**Component Breakdown:**

| Mechanism | Dialect | Type | Notes |
|-----------|---------|------|-------|
| `SERIAL` | PostgreSQL | Integer | Shorthand for sequence + default |
| `IDENTITY` | SQL Server, Oracle | Integer | `IDENTITY(seed, increment)` |
| `AUTO_INCREMENT` | MySQL | Integer | Column attribute |
| `gen_random_uuid()` | PostgreSQL | UUID | Requires `pgcrypto` or built-in in PG 13+ |
| `NEWID()` | SQL Server | UUID | Function |
| `UUID()` | MySQL | UUID | Function |

**Syntax Rules:**

- `SERIAL` columns are integer types that auto-increment; they are not a true data type but a shorthand.
- `IDENTITY` columns are automatically `NOT NULL`.
- `AUTO_INCREMENT` columns must be indexed (usually primary key).
- UUID columns require a generation function; they are not auto-incremented.
- Generated identifier columns can be overridden with an explicit value (except `GENERATED ALWAYS AS IDENTITY` in PostgreSQL, which requires `OVERRIDING SYSTEM VALUE`).

**Constraints and Limitations:**

- Sequential IDs can be predictable, which may be a security concern for publicly exposed identifiers.
- Sequence values may have gaps due to rollbacks, failed transactions, or caching.
- `SERIAL` is not a true type; it is a macro that creates an integer column with a sequence default.
- UUIDs are larger (16 bytes) and may have performance implications for indexing.
- In PostgreSQL, `SERIAL` is discouraged in favour of `GENERATED ... AS IDENTITY` for SQL standard compliance.

### Annotated Complete Code Examples

**Example 1: PostgreSQL SERIAL**

```sql
-- Create table with SERIAL primary key
CREATE TABLE users (
    id   SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);

-- Insert without specifying id
INSERT INTO users (name) VALUES ('Alice');
INSERT INTO users (name) VALUES ('Bob');

-- Verify
SELECT * FROM users;

-- Expected Output:
--  id | name
-- ----+-------
--   1 | Alice
--   2 | Bob
```

**Why this output occurs:** The `id` column is defined as `SERIAL`, which creates an implicit sequence. When `id` is omitted from the `INSERT`, the database calls `nextval()` on the sequence, assigning `1` to Alice and `2` to Bob.

**Example 2: SQL Server IDENTITY**

```sql
-- Create table with IDENTITY
CREATE TABLE users (
    id   INT IDENTITY(1,1) PRIMARY KEY,
    name NVARCHAR(100) NOT NULL
);

-- Insert without specifying id
INSERT INTO users (name) VALUES ('Alice');
INSERT INTO users (name) VALUES ('Bob');

-- Verify
SELECT * FROM users;

-- Expected Output:
--  id | name
-- ----+-------
--   1 | Alice
--   2 | Bob
```

**Why this output occurs:** The `IDENTITY(1,1)` property tells SQL Server to start at 1 and increment by 1 for each new row. Omitting the `id` column triggers automatic generation.

### Real-World Cases

**Case 1: User Account Creation**

Every new user account receives an auto-generated ID from a `SERIAL` or `IDENTITY` column, which is then used as a foreign key reference in other tables (e.g., orders, posts).

**Case 2: Distributed Systems with UUIDs**

A microservices architecture uses UUID primary keys to avoid collisions when multiple services insert records independently. Each service generates a UUID via `gen_random_uuid()` or `UUID()`.

### References

- PostgreSQL Documentation — Serial Types - https://www.postgresql.org/docs/current/datatype-numeric.html#DATATYPE-SERIAL
- PostgreSQL Documentation — UUID Type - https://www.postgresql.org/docs/current/datatype-uuid.html
- Microsoft SQL Server — IDENTITY Property - https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql-identity-property
- MySQL 8.4 Reference Manual — AUTO_INCREMENT - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/example-auto-increment.html

---

## 10. Retrieving Generated Keys (RETURNING, OUTPUT, SCOPE_IDENTITY(), LAST_INSERT_ID())

### Definitions

**Core Definition:** Retrieving generated keys refers to obtaining the values automatically generated by the database (such as auto-increment IDs) immediately after an `INSERT` operation, without issuing a separate query.

**Technical Definition:** Different RDBMSs provide different mechanisms for returning generated values: PostgreSQL's `RETURNING` clause, SQL Server's `OUTPUT` clause and `SCOPE_IDENTITY()` function, MySQL's `LAST_INSERT_ID()` function, and Oracle's `RETURNING INTO` clause. These mechanisms return the values from the rows actually inserted, including values computed by defaults, triggers, or sequences.

**Beginner-Friendly Explanation:** When you insert a new row and the database generates an ID for it, you often need to know what that ID is (so you can use it in other tables). Instead of running a second query to find it, these features give you the generated value right away.

### Purposes

- To obtain the primary key value of a newly inserted row.
- To use the generated ID as a foreign key in related insertions.
- To avoid an additional `SELECT` query after `INSERT`.
- To retrieve values computed by triggers or default expressions.
- To support application logic that depends on the inserted row's identity.

### Syntax Rules and Structure

**PostgreSQL RETURNING:**

```sql
INSERT INTO table_name (columns)
VALUES (values)
RETURNING id;
```

**SQL Server OUTPUT:**

```sql
INSERT INTO table_name (columns)
OUTPUT inserted.id
VALUES (values);
```

**SQL Server SCOPE_IDENTITY():**

```sql
INSERT INTO table_name (columns) VALUES (values);
SELECT SCOPE_IDENTITY();
```

**MySQL LAST_INSERT_ID():**

```sql
INSERT INTO table_name (columns) VALUES (values);
SELECT LAST_INSERT_ID();
```

**Oracle RETURNING INTO:**

```sql
INSERT INTO table_name (columns)
VALUES (values)
RETURNING id INTO :variable;
```

**Component Breakdown:**

| Mechanism | Dialect | Returns | Scope |
|-----------|---------|---------|-------|
| `RETURNING` | PostgreSQL, Oracle | Any column/expression | Per-row |
| `OUTPUT` | SQL Server | Any column/expression | Per-row |
| `SCOPE_IDENTITY()` | SQL Server | Last identity in current scope | Session/scope |
| `LAST_INSERT_ID()` | MySQL | Last auto-increment ID | Per-connection |
| `RETURNING INTO` | Oracle | Into PL/SQL variables | PL/SQL context |

**Syntax Rules:**

- `RETURNING` and `OUTPUT` can return multiple columns and expressions.
- `RETURNING` can use `*` to return all columns of the target table.
- `SCOPE_IDENTITY()` returns the last identity value generated in the current scope (session and batch).
- `LAST_INSERT_ID()` returns the first auto-generated value from the most recent `INSERT` statement on the current connection.
- Oracle's `RETURNING INTO` requires a PL/SQL block or bind variables.

**Constraints and Limitations:**

- `LAST_INSERT_ID()` is connection-specific; it does not reflect inserts by other connections.
- `SCOPE_IDENTITY()` is affected by triggers: if a trigger inserts into another table, `SCOPE_IDENTITY()` returns the identity from the trigger's insert, not the original one.
- `OUTPUT` can be used with `INSERT`, `UPDATE`, `DELETE`, and `MERGE`.
- `RETURNING` requires `SELECT` privilege on the columns referenced.
- In SQL Server, `OUTPUT` cannot be used in a statement that also has a `RETURNING` clause (not applicable — SQL Server uses `OUTPUT` exclusively).

### Annotated Complete Code Examples

**Example 1: PostgreSQL RETURNING**

```sql
-- Insert and return the generated ID
INSERT INTO users (name)
VALUES ('Alice')
RETURNING id;

-- Expected Output:
--  id
-- ----
--   1
```

**Why this output occurs:** The `RETURNING id` clause tells PostgreSQL to return the value of the `id` column from the row that was just inserted. The database generates the ID via the `SERIAL` sequence and returns it directly.

**Example 2: SQL Server OUTPUT**

```sql
-- Insert and output the generated ID and name
INSERT INTO users (name)
OUTPUT inserted.id, inserted.name
VALUES ('Bob');

-- Expected Output:
--  id | name
-- ----+-------
--   2 | Bob
```

**Why this output occurs:** The `OUTPUT inserted.id, inserted.name` clause returns the inserted row's `id` and `name` columns. The `inserted` pseudo-table references the new row.

**Example 3: MySQL LAST_INSERT_ID()**

```sql
-- Insert and retrieve the auto-increment ID
INSERT INTO users (name) VALUES ('Carol');
SELECT LAST_INSERT_ID();

-- Expected Output:
--  LAST_INSERT_ID()
-- ------------------
--                 3
```

**Why this output occurs:** `LAST_INSERT_ID()` returns the first auto-generated value from the most recent `INSERT` on the current connection. Carol receives ID 3 (assuming Alice and Bob were 1 and 2).

### Real-World Cases

**Case 1: Order and Order Lines**

An e-commerce application inserts an order and needs the generated `order_id` to insert order lines: `INSERT INTO orders (...) RETURNING order_id` (PostgreSQL) or `SCOPE_IDENTITY()` (SQL Server). The returned ID is used as a foreign key in the `order_lines` table.

**Case 2: Audit Trail**

A trigger inserts an audit record and uses `OUTPUT` to capture the audit ID for logging. The application receives the audit ID as part of the insert operation.

### References

- PostgreSQL Documentation — Returning Data from Modified Rows - https://www.postgresql.org/docs/current/dml-returning.html
- Microsoft SQL Server — OUTPUT Clause - https://learn.microsoft.com/en-us/sql/t-sql/queries/output-clause-transact-sql
- MySQL 8.4 Reference Manual — LAST_INSERT_ID() - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/information-functions.html#function_last-insert-id
- Oracle Database SQL Language Reference — RETURNING INTO - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/INSERT.html

---

## 11. Handling Constraint Violations (Foreign Key, Unique, Check)

### Definitions

**Core Definition:** Constraint violations occur when an `INSERT` statement attempts to add data that violates a table's integrity constraints, such as UNIQUE, FOREIGN KEY, CHECK, or NOT NULL constraints.

**Technical Definition:** A constraint is a rule defined on a table that restricts the data that can be stored. During `INSERT`, the database evaluates each constraint. If any constraint is violated, the entire `INSERT` statement fails (unless the violation is handled by an `ON CONFLICT` or `IGNORE` clause), and no rows are inserted.

**Beginner-Friendly Explanation:** Constraints are rules that keep your data clean and consistent. If you try to insert data that breaks a rule — like using a duplicate ID or referencing a non-existent foreign key — the database rejects the insert and tells you what went wrong.

### Purposes

- To maintain data integrity and consistency.
- To prevent duplicate values in unique columns.
- To ensure foreign key references point to existing rows.
- To enforce business rules through CHECK constraints.
- To prevent NULL values in required columns.

### Syntax Rules and Structure

**Common Constraints:**

```sql
CREATE TABLE orders (
    order_id     SERIAL PRIMARY KEY,                          -- Unique, not null
    customer_id  INTEGER REFERENCES customers(customer_id),   -- Foreign key
    email        VARCHAR(100) UNIQUE,                         -- Unique
    quantity     INTEGER CHECK (quantity > 0),                -- Check
    status       VARCHAR(20) NOT NULL                         -- Not null
);
```

**Component Breakdown:**

| Constraint | Violation | Error Message (PostgreSQL) |
|-----------|-----------|---------------------------|
| UNIQUE | Duplicate value | `duplicate key value violates unique constraint` |
| FOREIGN KEY | Referenced row missing | `violates foreign key constraint` |
| CHECK | Condition false | `violates check constraint` |
| NOT NULL | NULL in non-nullable column | `null value in column violates not-null constraint` |
| PRIMARY KEY | Duplicate or NULL | `duplicate key value violates unique constraint` |

**Syntax Rules:**

- Constraints are checked at the end of the statement (for `INSERT`), or at the end of the transaction for deferred constraints.
- The `ON CONFLICT` clause can handle UNIQUE and EXCLUSION constraint violations.
- The `IGNORE` keyword (MySQL) converts errors to warnings and skips problematic rows.
- Foreign key violations occur when the inserted value does not exist in the referenced table.

**Constraints and Limitations:**

- Constraint violations abort the entire `INSERT` statement (except with `ON CONFLICT DO NOTHING`).
- Deferred constraints are checked at transaction commit, allowing temporary violations within a transaction.
- `ON CONFLICT` cannot handle CHECK or NOT NULL violations; it only handles unique/exclusion conflicts.
- MySQL's `IGNORE` can silently skip rows, which may lead to data loss if not monitored.

### Annotated Complete Code Examples

**Example 1: UNIQUE Constraint Violation**

```sql
-- Setup
CREATE TABLE users (
    id    SERIAL PRIMARY KEY,
    email VARCHAR(100) UNIQUE NOT NULL
);

INSERT INTO users (email) VALUES ('alice@example.com');

-- Attempt duplicate email
INSERT INTO users (email) VALUES ('alice@example.com');

-- Expected Error (PostgreSQL):
-- ERROR: duplicate key value violates unique constraint "users_email_key"
-- DETAIL: Key (email)=(alice@example.com) already exists.
```

**Why this error occurs:** The `email` column has a `UNIQUE` constraint. The second `INSERT` attempts to insert a duplicate email, violating the constraint. The database rejects the entire statement and no row is added.

**Example 2: FOREIGN KEY Constraint Violation**

```sql
-- Setup
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name        VARCHAR(50)
);

CREATE TABLE orders (
    order_id    SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(customer_id),
    total       NUMERIC(10, 2)
);

-- Attempt to insert an order with a non-existent customer
INSERT INTO orders (customer_id, total) VALUES (999, 100.00);

-- Expected Error (PostgreSQL):
-- ERROR: insert or update on table "orders" violates foreign key constraint "orders_customer_id_fkey"
-- DETAIL: Key (customer_id)=(999) is not present in table "customers".
```

**Why this error occurs:** The `customer_id` column has a foreign key reference to `customers(customer_id)`. The value `999` does not exist in the `customers` table, violating the foreign key constraint. The `INSERT` is rejected.

### Real-World Cases

**Case 1: User Registration with Duplicate Email**

An application attempts to register a new user with an email that already exists. The UNIQUE constraint on `email` prevents the duplicate, and the application catches the error and displays a message to the user.

**Case 2: Order with Invalid Customer**

A bug in an e-commerce application attempts to create an order for a customer that was deleted. The foreign key constraint rejects the insert, preventing orphaned orders.

### References

- PostgreSQL Documentation — Constraints - https://www.postgresql.org/docs/current/ddl-constraints.html
- MySQL 8.4 Reference Manual — Constraints - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/create-table-foreign-keys.html
- Microsoft SQL Server — Constraints - https://learn.microsoft.com/en-us/sql/relational-databases/tables/unique-constraints-and-check-constraints

---

## 12. Upsert Operations (ON CONFLICT, ON DUPLICATE KEY UPDATE, MERGE)

### Definitions

**Core Definition:** An upsert (a portmanteau of "update" and "insert") is a database operation that inserts a new row if it does not exist, or updates an existing row if a conflict (typically a unique constraint violation) is detected.

**Technical Definition:** PostgreSQL implements upsert via `INSERT ... ON CONFLICT DO NOTHING` or `ON CONFLICT DO UPDATE`. MySQL uses `INSERT ... ON DUPLICATE KEY UPDATE`. SQL Server and Oracle (and standard SQL) provide the `MERGE` statement, which performs insert, update, or delete operations based on a join between a source and target table. These operations guarantee atomic outcomes: either the insert or the update occurs, never both, and never a partial state.

**Beginner-Friendly Explanation:** An upsert is like saying "add this row if it's new, or update it if it already exists." It saves you from having to check first with a `SELECT` and then decide whether to `INSERT` or `UPDATE`.

### Purposes

- To synchronise data between systems without duplicate rows.
- To implement "insert or update" logic in a single statement.
- To handle concurrent inserts gracefully without race conditions.
- To merge data from staging tables into production tables.
- To implement idempotent data loading in ETL pipelines.

### Syntax Rules and Structure

**PostgreSQL ON CONFLICT:**

```sql
INSERT INTO table_name (columns)
VALUES (values)
ON CONFLICT (conflict_column)
DO UPDATE SET column = EXCLUDED.column;
```

**MySQL ON DUPLICATE KEY UPDATE:**

```sql
INSERT INTO table_name (columns)
VALUES (values)
ON DUPLICATE KEY UPDATE column = VALUES(column);
```

**SQL Server MERGE:**

```sql
MERGE INTO target_table AS target
USING source_table AS source
ON target.key = source.key
WHEN MATCHED THEN
    UPDATE SET target.column = source.column
WHEN NOT MATCHED THEN
    INSERT (columns) VALUES (source.values);
```

**Component Breakdown:**

| Clause | Dialect | Behaviour |
|--------|---------|-----------|
| `ON CONFLICT DO NOTHING` | PostgreSQL | Skips conflicting rows |
| `ON CONFLICT DO UPDATE` | PostgreSQL | Updates conflicting rows |
| `ON DUPLICATE KEY UPDATE` | MySQL | Updates conflicting rows |
| `MERGE` | SQL Server, Oracle | Full upsert + delete |

**Syntax Rules:**

- `ON CONFLICT` requires a conflict target (column or constraint name) or can use `DO NOTHING` without a target.
- The `EXCLUDED` pseudo-table in PostgreSQL references the row proposed for insertion.
- In MySQL, `VALUES(column)` references the value that would have been inserted.
- `MERGE` requires a `USING` clause and an `ON` join condition.
- `MERGE` in SQL Server requires a semicolon at the end of the statement.

**Constraints and Limitations:**

- `ON CONFLICT` cannot handle CHECK or NOT NULL violations; it only handles unique/exclusion conflicts.
- MySQL's `ON DUPLICATE KEY UPDATE` does not support a conflict target; it applies to any unique index.
- SQL Server's `MERGE` is known to have concurrency issues and is discouraged by some experts; it may be deprecated in future.
- Oracle's `MERGE` does not support `WHEN NOT MATCHED BY SOURCE` in all versions.
- Upsert operations may lock rows and can cause contention under high concurrency.

### Annotated Complete Code Examples

**Example 1: PostgreSQL ON CONFLICT DO UPDATE**

```sql
-- Setup
CREATE TABLE inventory (
    product_id  INTEGER PRIMARY KEY,
    product_name VARCHAR(100),
    quantity    INTEGER NOT NULL DEFAULT 0
);

INSERT INTO inventory (product_id, product_name, quantity)
VALUES (1, 'Cheese', 10);

-- Upsert: insert or update if product_id exists
INSERT INTO inventory (product_id, product_name, quantity)
VALUES (1, 'Cheese', 25)
ON CONFLICT (product_id)
DO UPDATE SET quantity = EXCLUDED.quantity;

-- Verify
SELECT * FROM inventory;

-- Expected Output:
--  product_id | product_name | quantity
-- ------------+--------------+----------
--           1 | Cheese       |       25
```

**Why this output occurs:** The initial insert creates a row with `quantity = 10`. The second statement attempts to insert `product_id = 1`, which conflicts with the existing primary key. The `ON CONFLICT (product_id) DO UPDATE` clause intercepts the conflict and updates the existing row, setting `quantity = 25` (the `EXCLUDED.quantity` value).

**Example 2: MySQL ON DUPLICATE KEY UPDATE**

```sql
-- Setup
CREATE TABLE inventory (
    product_id  INT PRIMARY KEY,
    product_name VARCHAR(100),
    quantity    INT NOT NULL DEFAULT 0
);

INSERT INTO inventory (product_id, product_name, quantity)
VALUES (1, 'Cheese', 10);

-- Upsert
INSERT INTO inventory (product_id, product_name, quantity)
VALUES (1, 'Cheese', 25)
ON DUPLICATE KEY UPDATE quantity = VALUES(quantity);

-- Verify
SELECT * FROM inventory;

-- Expected Output:
--  product_id | product_name | quantity
-- ------------+--------------+----------
--           1 | Cheese       |       25
```

**Why this output occurs:** The `ON DUPLICATE KEY UPDATE` clause detects the duplicate primary key and performs an update instead of an insert. `VALUES(quantity)` refers to the value that would have been inserted (`25`), which replaces the existing quantity.

**Example 3: SQL Server MERGE**

```sql
-- Setup
CREATE TABLE target_inventory (
    product_id  INT PRIMARY KEY,
    product_name NVARCHAR(100),
    quantity    INT NOT NULL DEFAULT 0
);

CREATE TABLE source_inventory (
    product_id  INT PRIMARY KEY,
    product_name NVARCHAR(100),
    quantity    INT NOT NULL DEFAULT 0
);

INSERT INTO target_inventory VALUES (1, 'Cheese', 10);
INSERT INTO source_inventory VALUES (1, 'Cheese', 25), (2, 'Bread', 50);

-- MERGE
MERGE INTO target_inventory AS target
USING source_inventory AS source
ON target.product_id = source.product_id
WHEN MATCHED THEN
    UPDATE SET target.quantity = source.quantity
WHEN NOT MATCHED THEN
    INSERT (product_id, product_name, quantity)
    VALUES (source.product_id, source.product_name, source.quantity);

-- Verify
SELECT * FROM target_inventory;

-- Expected Output:
--  product_id | product_name | quantity
-- ------------+--------------+----------
--           1 | Cheese       |       25
--           2 | Bread        |       50
```

**Why this output occurs:** The `MERGE` statement joins `target_inventory` with `source_inventory` on `product_id`. For product 1 (matched), the quantity is updated to 25. For product 2 (not matched), a new row is inserted with quantity 50.

### Real-World Cases

**Case 1: Daily Sales Sync**

An ETL job uses `ON CONFLICT DO UPDATE` to merge daily sales data into a summary table. If a product's sales record already exists for the day, it is updated; otherwise, a new row is inserted.

**Case 2: Inventory Management**

A warehouse system uses `MERGE` to synchronise inventory levels from a source system. Existing products are updated, new products are inserted, and (with `WHEN NOT MATCHED BY SOURCE`) discontinued products are deleted.

### References

- PostgreSQL Documentation — INSERT ... ON CONFLICT - https://www.postgresql.org/docs/current/sql-insert.html
- MySQL 8.4 Reference Manual — INSERT ... ON DUPLICATE KEY UPDATE - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/insert-on-duplicate.html
- Microsoft SQL Server — MERGE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql
- Oracle Database SQL Language Reference — MERGE - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/MERGE.html

---

## 13. Transaction Safety During Failures (All-or-Nothing Atomicity)

### Definitions

**Core Definition:** Transaction safety during failures refers to the guarantee that an `INSERT` operation (or a group of operations) is atomic — it either completes entirely or has no effect at all, even in the event of an error, system crash, or power failure.

**Technical Definition:** The ACID property of atomicity ensures that a transaction is treated as a single, indivisible unit of work. For a single `INSERT` statement, the database wraps the operation in an implicit transaction and ensures it is either fully committed or fully rolled back. For multi-statement transactions, explicit `BEGIN`/`COMMIT`/`ROLLBACK` blocks are used to group operations.

**Beginner-Friendly Explanation:** Atomicity means "all or nothing." If you're inserting data and something goes wrong halfway through, the database undoes everything, so you never end up with half-finished data.

### Purposes

- To guarantee that a failed `INSERT` leaves no partial data.
- To ensure data consistency across related tables.
- To recover gracefully from errors, constraint violations, and system crashes.
- To group multiple `INSERT` statements into a single atomic unit.
- To maintain database integrity in concurrent environments.

### Syntax Rules and Structure

**Implicit Transaction (Single Statement):**

```sql
-- Each statement is automatically wrapped in a transaction
INSERT INTO users (name) VALUES ('Alice');
-- Committed automatically if successful; rolled back if it fails.
```

**Explicit Transaction (Multiple Statements):**

```sql
BEGIN;  -- or START TRANSACTION in MySQL
INSERT INTO orders (customer_id, total) VALUES (1, 100.00);
INSERT INTO order_lines (order_id, product_id, quantity)
VALUES (LAST_INSERT_ID(), 42, 2);
COMMIT;  -- or ROLLBACK on error
```

**Component Breakdown:**

| Command | Purpose |
|---------|---------|
| `BEGIN` / `START TRANSACTION` | Starts an explicit transaction |
| `COMMIT` | Permanently saves all changes in the transaction |
| `ROLLBACK` | Undoes all changes in the transaction |
| `SAVEPOINT` | Creates a point within a transaction to roll back to |

**Syntax Rules:**

- A single `INSERT` statement is automatically atomic; no explicit transaction is needed.
- Multiple DML statements require an explicit transaction to ensure all-or-nothing behaviour.
- `ROLLBACK` undoes all changes made since the last `BEGIN` or `SAVEPOINT`.
- `SAVEPOINT` allows partial rollback within a transaction.
- The `SET XACT_ABORT ON` option in SQL Server automatically rolls back the entire transaction on any error.

**Constraints and Limitations:**

- Autocommit mode (the default in most RDBMSs) commits each statement immediately.
- DDL statements (e.g., `CREATE TABLE`) may cause implicit commits in some dialects (Oracle, MySQL), breaking transaction boundaries.
- Long-running transactions can hold locks and block other operations.
- Rollback operations consume resources and may take time for large transactions.
- Not all RDBMSs support transactional DDL (PostgreSQL does; Oracle and MySQL do not).

### Annotated Complete Code Examples

**Example 1: Single INSERT Atomicity**

```sql
-- Setup
CREATE TABLE accounts (
    id      SERIAL PRIMARY KEY,
    name    VARCHAR(50),
    balance NUMERIC(10, 2) CHECK (balance >= 0)
);

INSERT INTO accounts (name, balance) VALUES ('Alice', 100.00);

-- Attempt to insert a negative balance (violates CHECK)
INSERT INTO accounts (name, balance) VALUES ('Bob', -50.00);

-- Expected Error (PostgreSQL):
-- ERROR: new row for relation "accounts" violates check constraint "accounts_balance_check"

-- Verify: Bob's row was not inserted
SELECT * FROM accounts;

-- Expected Output:
--  id | name  | balance
-- ----+-------+---------
--   1 | Alice |  100.00
```

**Why this output occurs:** The second `INSERT` violates the CHECK constraint on `balance`. Because the statement is automatically wrapped in an implicit transaction, the database rolls back the insertion. Bob's row is not added.

**Example 2: Multi-Statement Transaction with Rollback**

```sql
-- Start an explicit transaction
BEGIN;

-- Insert an order
INSERT INTO orders (customer_id, total) VALUES (1, 100.00);

-- Insert order lines (this will fail if product_id 999 doesn't exist)
INSERT INTO order_lines (order_id, product_id, quantity)
VALUES (currval('orders_order_id_seq'), 999, 2);

-- If the second insert fails, roll back everything
-- (In practice, this would be handled by application error handling)

-- Rollback the transaction
ROLLBACK;

-- Verify: no order was created
SELECT * FROM orders;

-- Expected Output:
--  order_id | customer_id | total
-- ----------+-------------+-------
-- (0 rows)
```

**Why this output occurs:** The transaction begins with `BEGIN`. The first `INSERT` adds an order, but the second `INSERT` references a non-existent product (or would fail for some reason). The `ROLLBACK` command undoes both insertions, leaving the database in its original state. No order or order line is created.

### Real-World Cases

**Case 1: Financial Transfers**

A bank transfer involves debiting one account and crediting another. Both operations are wrapped in a transaction. If the credit fails (e.g., due to a constraint violation), the debit is rolled back, ensuring no money is lost.

**Case 2: Order Processing**

An e-commerce checkout inserts an order, order lines, and a payment record. If any of these operations fails (e.g., payment declined), the entire transaction is rolled back, preventing orphaned orders.

### References

- PostgreSQL Documentation — Transactions - https://www.postgresql.org/docs/current/tutorial-transactions.html
- MySQL 8.4 Reference Manual — Transactions - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/commit.html
- Microsoft SQL Server — Transactions - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/transactions-transact-sql
- Oracle Database — Transaction Management - https://docs.oracle.com/en/database/oracle/oracle-database/19/cncpt/transactions.html

---

## Summary Table of Key Features

| Feature | Purpose | Key Limitation |
|---------|---------|----------------|
| `INSERT INTO` | Add rows to a table | Requires INSERT privilege |
| Single Row (`VALUES`) | Insert one row | Must match column count |
| Multi-Row (`VALUES`) | Bulk insert efficiently | Statement size limits |
| Explicit Column List | Robust against schema changes | More typing |
| Implicit Positional | Concise for quick inserts | Breaks on schema changes |
| Value Specifications | Literals, expressions, functions | Type compatibility |
| Default Values | Use database-defined defaults | NOT NULL without default fails |
| Explicit `NULL` | Represent missing data | Cannot insert into NOT NULL |
| `INSERT INTO ... SELECT` | Copy data between tables | Column count and type match |
| Generated IDs (`SERIAL`, `IDENTITY`, UUID) | Auto-generate unique keys | Predictability, gaps |
| `RETURNING` / `OUTPUT` / `LAST_INSERT_ID()` | Retrieve generated keys | Dialect-specific |
| Constraint Violations | Enforce data integrity | Aborts entire statement |
| Upsert (`ON CONFLICT`, `MERGE`) | Insert or update atomically | Dialect-specific syntax |
| Transaction Atomicity | All-or-nothing guarantee | Long transactions lock resources |

---

## References

- PostgreSQL Documentation — INSERT - https://www.postgresql.org/docs/current/sql-insert.html
- PostgreSQL Documentation — Inserting Data - https://www.postgresql.org/docs/current/dml-insert.html
- PostgreSQL Documentation — Returning Data from Modified Rows - https://www.postgresql.org/docs/current/dml-returning.html
- PostgreSQL Documentation — Constraints - https://www.postgresql.org/docs/current/ddl-constraints.html
- PostgreSQL Documentation — Transactions - https://www.postgresql.org/docs/current/tutorial-transactions.html
- MySQL 8.4 Reference Manual — INSERT Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/insert.html
- MySQL 8.4 Reference Manual — INSERT ... ON DUPLICATE KEY UPDATE - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/insert-on-duplicate.html
- MySQL 8.4 Reference Manual — INSERT ... SELECT - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/insert-select.html
- MySQL 8.4 Reference Manual — LAST_INSERT_ID() - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/information-functions.html#function_last-insert-id
- MySQL 8.4 Reference Manual — AUTO_INCREMENT - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/example-auto-increment.html
- Microsoft SQL Server — INSERT (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/insert-transact-sql
- Microsoft SQL Server — MERGE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql
- Microsoft SQL Server — OUTPUT Clause - https://learn.microsoft.com/en-us/sql/t-sql/queries/output-clause-transact-sql
- Microsoft SQL Server — IDENTITY Property - https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql-identity-property
- Microsoft SQL Server — Transactions - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/transactions-transact-sql
- Oracle Database SQL Language Reference — INSERT - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/INSERT.html
- Oracle Database SQL Language Reference — MERGE - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/MERGE.html
- Oracle Database — NULLs - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Nulls.html
- SQLite Documentation — INSERT - https://www2.sqlite.org/lang_insert.html
- ISO/IEC 9075-2:2003 — SQL Standard (INSERT Statement) - https://jtc1sc32.org/doc/N0251-0300/32N0254.pdf