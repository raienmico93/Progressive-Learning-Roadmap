# Comprehensive Programming Cheat Sheet: SQL Table Constraints

## Topic Overview

### Definitions

**Core Definition**
Table constraints are declarative rules attached to a table or its columns that restrict the values that can be stored, thereby enforcing data integrity at the database level.

**Technical Definition**
A constraint is a rule enforced by the database engine that restricts the data that can be inserted, updated, or deleted from a table. Constraints are declared as part of the table definition (`CREATE TABLE`) or added later (`ALTER TABLE`), and they are stored in the system catalogs as first-class objects with names, dependencies, and enforcement semantics. The principal constraint types are `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, `DEFAULT`, and—in PostgreSQL—`EXCLUDE`. Constraints can be column-level (attached to a single column) or table-level (spanning multiple columns), and their enforcement can be immediate or deferred (for certain constraint types in PostgreSQL).

**Beginner-Friendly Explanation**
Constraints are the rules of the game for your database. They stop bad data from getting in—like a missing name, a duplicate email address, or a salary that is less than zero. You declare these rules once, and the database enforces them automatically for every insert, update, and delete.

### Key Characteristics

- **Declarative integrity**: Constraints are declared as part of the schema, not enforced by application code, ensuring that all applications respect the same rules.
- **Vendor divergence**: Core constraint types (`NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, `DEFAULT`) are broadly standardised, but `EXCLUDE` is PostgreSQL-specific, and deferrability, validation options, and enforcement toggles vary by RDBMS.
- **Performance cost**: Constraints—especially `UNIQUE`, `PRIMARY KEY`, and `FOREIGN KEY`—incur overhead on write operations because they require index lookups and updates. They generally do not penalise reads.
- **Dependency-aware**: Constraints participate in the dependency graph; a `PRIMARY KEY` or `UNIQUE` constraint referenced by a `FOREIGN KEY` cannot be dropped without cascading or first removing the dependent foreign key.
- **Timing flexibility (PostgreSQL)**: `UNIQUE`, `PRIMARY KEY`, `EXCLUDE`, and `FOREIGN KEY` constraints can be declared `DEFERRABLE`, allowing their checks to be postponed until transaction commit.

### Prerequisites

- A running RDBMS instance (PostgreSQL, MySQL, or SQL Server) and a client connection.
- `CREATE TABLE` or `ALTER TABLE` privilege on the target table.
- For `FOREIGN KEY` constraints: the referenced table and columns must exist and be `PRIMARY KEY` or `UNIQUE`.
- Understanding of the application's data integrity requirements and the performance implications of constraint enforcement.

### Related Programming Areas

- Data Definition Language (DDL) – `CREATE TABLE`, `ALTER TABLE`.
- Data Manipulation Language (DML) – constraints are enforced during `INSERT`, `UPDATE`, and `DELETE`.
- Indexing and query performance – `UNIQUE`, `PRIMARY KEY`, and `EXCLUDE` constraints automatically create indexes.
- Transaction management – deferrable constraints interact with transaction boundaries.
- Database migration and schema versioning – constraints are added, modified, and dropped as part of schema evolution.

---

## Core Concepts and Features

### 1. `NOT NULL`

#### Definitions

**Core Definition**
The `NOT NULL` constraint requires that a column cannot contain `NULL` values.

**Technical Definition**
`NOT NULL` is a column-level constraint that rejects any `INSERT` or `UPDATE` that would set the column to `NULL`. In PostgreSQL, it is implemented as a column attribute (`attnotnull` in `pg_attribute`) and is always checked immediately; it cannot be deferred. In SQL Server, `NOT NULL` is a column property rather than a separately named constraint. In MySQL, `NOT NULL` is specified as part of the column definition.

**Beginner-Friendly Explanation**
`NOT NULL` means “this field is required.” You cannot leave it blank; every row must have a value for this column.

#### Purposes

- To ensure that critical columns always have a value (e.g., a user's email address or an order's total).
- To prevent `NULL` from propagating into calculations and comparisons, which can cause subtle bugs.
- To enforce application-level requirements at the database level.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name data_type NOT NULL
```

**PostgreSQL:** `column_name data_type NOT NULL` (column constraint).

**MySQL:** `col_name data_type [NOT NULL | NULL]`.

**SQL Server:** `column_name data_type NOT NULL`.

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `NOT NULL` | The constraint keyword; rejects `NULL` values. |

**Syntax rules:**

- `NOT NULL` is a column constraint and cannot be declared at the table level.
- A `PRIMARY KEY` implies `NOT NULL`; specifying both is redundant but allowed.
- In PostgreSQL, `NOT NULL` is not deferrable.

**Constraints and limitations:**

- Adding `NOT NULL` to a column that already contains `NULL` values requires first updating or deleting those rows.
- `NOT NULL` does not prevent empty strings or zero values; it only rejects `NULL`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – NOT NULL on a column

```sql
-- Step 1: Create a table with a NOT NULL column.
CREATE TABLE users (
    user_id serial PRIMARY KEY,
    email   text NOT NULL
);

-- Step 2: Attempt to insert a NULL email.
INSERT INTO users (email) VALUES (NULL);
```

**Expected output:**

```text
CREATE TABLE
ERROR:  null value in column "email" of relation "users" violates not-null constraint
DETAIL:  Failing row contains (1, null).
```

**Why this output appears**: The `NOT NULL` constraint rejects the `NULL` value, and PostgreSQL raises an error with the constraint name and the failing row.

#### Real-World Cases

- **User registration**: An `email` column is `NOT NULL` because every account must have a contact address.
- **Financial records**: An `amount` column in a transactions table is `NOT NULL` to prevent incomplete financial entries.

---

### 2. `UNIQUE`

#### Definitions

**Core Definition**
The `UNIQUE` constraint ensures that all values in a column or combination of columns are distinct.

**Technical Definition**
`UNIQUE` can be a column constraint or a table constraint. It creates a unique B-tree index (in PostgreSQL) or a unique index (in MySQL and SQL Server) to enforce the rule. In PostgreSQL, null values are not considered equal for the purpose of a unique constraint, so multiple rows with `NULL` in a unique column are permitted. Each unique table constraint must name a set of columns different from any other unique or primary key constraint on the same table.

**Beginner-Friendly Explanation**
`UNIQUE` means “no duplicates allowed.” If a column is `UNIQUE`, you cannot have two rows with the same value in that column.

#### Purposes

- To enforce natural keys such as email addresses, usernames, or product SKUs.
- To prevent duplicate records in a table.
- To provide a target for foreign-key references (a `UNIQUE` constraint can be referenced by a `FOREIGN KEY`).

#### Syntax Rules and Structure

**Column-level:**

```sql
column_name data_type UNIQUE
```

**Table-level:**

```sql
UNIQUE ( column_name [, ...] )
```

**PostgreSQL:** `UNIQUE [ NULLS [ NOT ] DISTINCT ]` (PostgreSQL 15+).

**MySQL:** `[CONSTRAINT [symbol]] UNIQUE [INDEX | KEY] [index_name] (key_part,...)`.

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UNIQUE` | The constraint keyword. |
| `(column, ...)` | The column or columns that must be unique. |
| `NULLS NOT DISTINCT` | Treats `NULL` values as equal (PostgreSQL 15+). |

**Syntax rules:**

- A `UNIQUE` constraint automatically creates a unique index.
- In PostgreSQL, `NULL` values are distinct by default; multiple `NULL`s are allowed.
- In MySQL and SQL Server, `NULL` handling varies (SQL Server allows only one `NULL` per unique column).

**Constraints and limitations:**

- `UNIQUE` constraints incur write overhead because the index must be updated on every insert, update, and delete.
- Dropping a `UNIQUE` constraint referenced by a `FOREIGN KEY` requires `CASCADE` or dropping the foreign key first.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – UNIQUE constraint on email

```sql
-- Step 1: Create a table with a UNIQUE column.
CREATE TABLE accounts (
    account_id serial PRIMARY KEY,
    email      text UNIQUE NOT NULL
);

-- Step 2: Insert two rows with the same email.
INSERT INTO accounts (email) VALUES ('a@example.com');
INSERT INTO accounts (email) VALUES ('a@example.com');
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
ERROR:  duplicate key value violates unique constraint "accounts_email_key"
DETAIL:  Key (email)=(a@example.com) already exists.
```

**Why this output appears**: The `UNIQUE` constraint creates a unique index on `email`; the second insert violates the uniqueness rule.

#### Real-World Cases

- **Username uniqueness**: A `users` table has a `UNIQUE` constraint on `username` to prevent two accounts with the same name.
- **Product SKUs**: A `products` table has a `UNIQUE` constraint on `sku` to ensure each product has a distinct identifier.

---

### 3. `PRIMARY KEY`

#### Definitions

**Core Definition**
The `PRIMARY KEY` constraint uniquely identifies each row in a table and implies both `UNIQUE` and `NOT NULL`.

**Technical Definition**
`PRIMARY KEY` is a combination of `UNIQUE` and `NOT NULL` that serves as the table's primary row identifier. Only one primary key can be defined per table, whether as a column constraint or a table constraint. In PostgreSQL, it creates a unique B-tree index; in MySQL and SQL Server, it creates a unique clustered or nonclustered index. A primary key can be composite (spanning multiple columns).

**Beginner-Friendly Explanation**
`PRIMARY KEY` is the “official ID” for each row. It must be unique and cannot be blank, and every table can have only one.

#### Purposes

- To provide a stable, unique identifier for each row.
- To serve as the target for foreign-key references from other tables.
- To enforce entity integrity by rejecting duplicate or `NULL` key values.

#### Syntax Rules and Structure

**Column-level:**

```sql
column_name data_type PRIMARY KEY
```

**Table-level:**

```sql
PRIMARY KEY ( column_name [, ...] )
```

**PostgreSQL:** `PRIMARY KEY (column_name [, ...])` (table constraint).

**MySQL:** `[CONSTRAINT [symbol]] PRIMARY KEY [index_type] (key_part,...)`.

**SQL Server:** `[CONSTRAINT constraint_name] PRIMARY KEY [CLUSTERED | NONCLUSTERED] (column [,...])`.

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `PRIMARY KEY` | The constraint keyword. |
| `(column, ...)` | One or more columns that form the key. |

**Syntax rules:**

- A table can have only one primary key.
- `PRIMARY KEY` implies `NOT NULL` and `UNIQUE`.
- A composite primary key allows duplicate values in individual columns but requires the combination to be unique.

**Constraints and limitations:**

- Changing a primary key requires dropping and recreating it; this can be expensive on large tables.
- A primary key referenced by a foreign key cannot be dropped without `CASCADE` or dropping the foreign key first.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Composite primary key

```sql
-- Step 1: Create a table with a composite primary key.
CREATE TABLE order_items (
    order_id   int,
    line_no    int,
    product_id int,
    PRIMARY KEY (order_id, line_no)
);

-- Step 2: Insert duplicate combinations.
INSERT INTO order_items VALUES (1, 1, 100);
INSERT INTO order_items VALUES (1, 1, 200);
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
ERROR:  duplicate key value violates unique constraint "order_items_pkey"
DETAIL:  Key (order_id, line_no)=(1, 1) already exists.
```

**Why this output appears**: The composite primary key requires the combination of `order_id` and `line_no` to be unique.

#### Real-World Cases

- **Order line items**: A composite primary key of `(order_id, line_no)` ensures each line is unique within an order.
- **Junction tables**: A many-to-many relationship table uses a composite primary key of the two foreign keys.

---

### 4. `FOREIGN KEY`

#### Definitions

**Core Definition**
A `FOREIGN KEY` constraint ensures that values in a column (or set of columns) match values in a referenced table's primary key or unique key, enforcing referential integrity.

**Technical Definition**
`FOREIGN KEY` is a constraint that creates a dependency between a child table and a parent table. The child table's foreign-key columns must contain values that exist in the parent table's referenced columns (or be `NULL`, if allowed). The constraint supports `ON DELETE` and `ON UPDATE` actions: `NO ACTION`, `RESTRICT`, `CASCADE`, `SET NULL`, and `SET DEFAULT`. In PostgreSQL, foreign-key constraints are implemented internally as triggers.

**Beginner-Friendly Explanation**
A `FOREIGN KEY` is a link between two tables. It says: “The value in this column must exist in that other table.” For example, an order's `customer_id` must match a real customer in the `customers` table.

#### Purposes

- To enforce referential integrity between related tables.
- To prevent orphaned records (e.g., an order referencing a non-existent customer).
- To define cascading actions that automatically propagate deletes or updates.

#### Syntax Rules and Structure

**Column-level:**

```sql
column_name data_type REFERENCES reftable [(refcolumn)] [ON DELETE action] [ON UPDATE action]
```

**Table-level:**

```sql
FOREIGN KEY (column [, ...]) REFERENCES reftable [(refcolumn [, ...])] [ON DELETE action] [ON UPDATE action]
```

**PostgreSQL:** Supports `MATCH FULL | MATCH PARTIAL | MATCH SIMPLE` and `DEFERRABLE`.

**MySQL:** `[CONSTRAINT [symbol]] FOREIGN KEY [index_name] (col_name,...) reference_definition`.

**SQL Server:** `[FOREIGN KEY] REFERENCES [schema_name.] referenced_table_name [(ref_column)] [ON DELETE ...] [ON UPDATE ...]`.

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `FOREIGN KEY (columns)` | The child columns. |
| `REFERENCES table (columns)` | The parent table and referenced columns. |
| `ON DELETE action` | Action when a parent row is deleted. |
| `ON UPDATE action` | Action when a parent key is updated. |

**Syntax rules:**

- The referenced columns must be `PRIMARY KEY` or `UNIQUE` in the parent table.
- A foreign key can reference a table in the same database only (cross-database foreign keys are not supported in MySQL/PostgreSQL; SQL Server supports cross-database with limitations).
- `ON DELETE SET NULL` requires the child columns to be nullable.

**Constraints and limitations:**

- Foreign-key constraints incur overhead on inserts and updates because they require lookups against the parent table.
- Dropping a parent table referenced by a foreign key requires `CASCADE` (PostgreSQL) or dropping the foreign key first.
- MySQL allows disabling foreign-key checks with `SET FOREIGN_KEY_CHECKS = 0` (session-level).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Foreign key with ON DELETE CASCADE

```sql
-- Step 1: Create parent and child tables.
CREATE TABLE customers (id int PRIMARY KEY, name text);
CREATE TABLE orders (
    id int PRIMARY KEY,
    customer_id int REFERENCES customers(id) ON DELETE CASCADE
);

-- Step 2: Insert data.
INSERT INTO customers VALUES (1, 'Acme');
INSERT INTO orders VALUES (100, 1);

-- Step 3: Delete the parent; the child is cascaded.
DELETE FROM customers WHERE id = 1;
SELECT * FROM orders;
```

**Expected output:**

```text
CREATE TABLE
CREATE TABLE
INSERT 0 1
INSERT 0 1
DELETE 1
 id | customer_id
----+-------------
(0 rows)
```

**Why this output appears**: `ON DELETE CASCADE` automatically deletes the child row when the parent is deleted.

#### Real-World Cases

- **E-commerce**: `order_items.product_id` references `products.id`; deleting a product with existing orders is restricted.
- **Blog**: `comments.post_id` references `posts.id` with `ON DELETE CASCADE`, so deleting a post removes its comments.

---

### 5. `CHECK`

#### Definitions

**Core Definition**
The `CHECK` constraint requires that a Boolean expression evaluates to `TRUE` or `UNKNOWN` (i.e., not `FALSE`) for every row.

**Technical Definition**
`CHECK` is a column or table constraint that specifies a Boolean expression. New or updated rows must satisfy the expression for the operation to succeed. A column-level `CHECK` should reference only that column's value; a table-level `CHECK` may reference multiple columns. `CHECK` expressions cannot contain subqueries or reference variables other than columns of the current row. In PostgreSQL, `CHECK` constraints can be added as `NOT VALID` to skip validation of existing rows.

**Beginner-Friendly Explanation**
`CHECK` is a custom rule you write for a column or table. For example, “salary must be greater than zero” or “end_date must be after start_date.”

#### Purposes

- To enforce domain-specific business rules that cannot be expressed with standard constraints.
- To validate that values fall within a permitted range or pattern.
- To enforce multi-column consistency rules (e.g., `end_date > start_date`).

#### Syntax Rules and Structure

**Column-level:**

```sql
column_name data_type CHECK (expression)
```

**Table-level:**

```sql
CHECK (expression)
```

**PostgreSQL:** `CHECK (expression) [NO INHERIT]`; supports `NOT VALID`.

**MySQL:** `[CONSTRAINT [symbol]] CHECK (expr) [[NOT] ENFORCED]`.

**SQL Server:** `CHECK [NOT FOR REPLICATION] (logical_expression)`.

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `CHECK (expression)` | The Boolean expression that must not evaluate to `FALSE`. |
| `NOT VALID` | Skips validation of existing rows (PostgreSQL). |
| `NOT ENFORCED` | Creates the constraint but does not enforce it (MySQL). |

**Syntax rules:**

- `CHECK` expressions cannot contain subqueries.
- A `CHECK` constraint is satisfied if the expression evaluates to `TRUE` or `UNKNOWN` (`NULL`).
- In PostgreSQL, `NOT VALID` constraints are enforced for new rows but not retroactively for existing rows until `VALIDATE CONSTRAINT` is run.

**Constraints and limitations:**

- `CHECK` constraints are not deferrable in PostgreSQL.
- In MySQL, `CHECK` constraints were ignored before version 8.0.16.
- `CHECK` constraints incur a small per-row evaluation cost.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – CHECK constraint with NOT VALID and VALIDATE

```sql
-- Step 1: Create a table with a CHECK constraint as NOT VALID.
CREATE TABLE accounts (id serial PRIMARY KEY, balance numeric);
INSERT INTO accounts (balance) VALUES (100), (-50);

ALTER TABLE accounts ADD CONSTRAINT chk_positive CHECK (balance >= 0) NOT VALID;

-- Step 2: Validate the constraint.
ALTER TABLE accounts VALIDATE CONSTRAINT chk_positive;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 2
ALTER TABLE
ERROR:  check constraint "chk_positive" of relation "accounts" is violated by some row
```

**Why this output appears**: The `NOT VALID` constraint is added without checking existing rows. `VALIDATE CONSTRAINT` then scans the table and finds the negative balance, causing the validation to fail.

#### Real-World Cases

- **Date ranges**: `CHECK (end_date > start_date)` ensures valid periods in a bookings table.
- **Salary bounds**: `CHECK (salary > 0)` prevents negative salaries.
- **Status values**: `CHECK (status IN ('active', 'closed', 'pending'))` restricts a status column to valid values.

---

### 6. `DEFAULT`

#### Definitions

**Core Definition**
The `DEFAULT` constraint specifies a value that the database automatically assigns to a column when no value is provided during an `INSERT`.

**Technical Definition**
`DEFAULT` is a column attribute that provides a literal value, expression, or function call evaluated at insert time when the column is omitted or explicitly set to `DEFAULT`. In SQL Server, `DEFAULT` is implemented as a named constraint (`DEFAULT constant_expression FOR column`). In PostgreSQL and MySQL, it is a column attribute.

**Beginner-Friendly Explanation**
`DEFAULT` is the automatic fill-in value. If you don't provide a value for a column, the database uses the default you specified.

#### Purposes

- To supply sensible values for columns that are often left blank (e.g., a `status` column defaulting to `'pending'`).
- To automatically record timestamps (`DEFAULT now()` or `DEFAULT CURRENT_TIMESTAMP`).
- To ensure that columns always have a value even when the application omits them.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name data_type DEFAULT default_expression
```

**PostgreSQL:** `DEFAULT default_expr` (column constraint).

**MySQL:** `DEFAULT {literal | (expr)}`.

**SQL Server:** `DEFAULT constant_expression FOR column` (table-level) or as part of column definition.

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `DEFAULT expression` | The value or expression to use when no value is supplied. |

**Syntax rules:**

- The default expression must be compatible with the column's data type.
- In PostgreSQL, `DEFAULT` can reference a sequence via `nextval()`.
- In SQL Server, `DEFAULT` is a named constraint and can be dropped with `DROP CONSTRAINT`.

**Constraints and limitations:**

- Changing a default does not affect existing rows.
- `DEFAULT` is not a true constraint in PostgreSQL and MySQL; it is a column attribute. In SQL Server, it is a constraint object.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – DEFAULT with now()

```sql
-- Step 1: Create a table with defaults.
CREATE TABLE tasks (
    task_id    serial PRIMARY KEY,
    status     text NOT NULL DEFAULT 'pending',
    created_at timestamptz DEFAULT now()
);

-- Step 2: Insert without specifying status or created_at.
INSERT INTO tasks (task_id) VALUES (DEFAULT);
SELECT * FROM tasks;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
 task_id | status  |          created_at
---------+---------+-------------------------------
       1 | pending | 2026-11-13 10:00:00.123+00
(1 row)
```

**Why this output appears**: The `status` column defaults to `'pending'` and `created_at` defaults to `now()` when omitted.

#### Real-World Cases

- **Audit timestamps**: `created_at DEFAULT now()` automatically records when a row was inserted.
- **Workflow status**: A `status` column defaults to `'pending'` for new records.

---

### 7. `EXCLUDE` (PostgreSQL)

#### Definitions

**Core Definition**
The `EXCLUDE` constraint ensures that no two rows satisfy a specified operator-based comparison, typically to prevent overlapping ranges or values.

**Technical Definition**
`EXCLUDE` is a PostgreSQL-specific table constraint that generalises `UNIQUE`. It specifies one or more *exclusion elements*, each pairing a column or expression with a comparison operator (e.g., `=` for equality, `&&` for overlap). The constraint rejects any insert or update that would cause the specified comparison to be true for any pair of rows. `EXCLUDE` is often used with range types and GiST indexes to prevent overlapping bookings or time ranges.

**Beginner-Friendly Explanation**
`EXCLUDE` is like `UNIQUE`, but more powerful. Instead of just preventing exact duplicates, it can prevent overlaps—for example, ensuring that no two bookings for the same room overlap in time.

#### Purposes

- To prevent overlapping ranges (e.g., double-booked meeting rooms or hotel reservations).
- To enforce complex uniqueness rules that cannot be expressed with `UNIQUE`.
- To support temporal and spatial integrity constraints.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
EXCLUDE [ USING index_method ] ( exclude_element WITH operator [, ...] )
    [ INCLUDE ( column [, ...] ) ]
    [ WHERE ( predicate ) ]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `USING index_method` | The index method (typically `gist`). |
| `exclude_element` | A column or expression. |
| `WITH operator` | The comparison operator (e.g., `=`, `&&`). |
| `WHERE predicate` | Optional partial constraint (PostgreSQL). |

**Syntax rules:**

- `EXCLUDE` requires an appropriate index method; `gist` is commonly used with range types.
- The `btree_gist` extension is required for equality comparisons on scalar types in a GiST index.
- `EXCLUDE` automatically creates an index to enforce the constraint.

**Constraints and limitations:**

- `EXCLUDE` is PostgreSQL-specific and not available in MySQL or SQL Server.
- Performance depends on the index method and the complexity of the exclusion elements.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – EXCLUDE to prevent overlapping bookings

```sql
-- Step 1: Enable the btree_gist extension.
CREATE EXTENSION IF NOT EXISTS btree_gist;

-- Step 2: Create a table with an EXCLUDE constraint.
CREATE TABLE bookings (
    room_id   int,
    during    tsrange,
    EXCLUDE USING gist (room_id WITH =, during WITH &&)
);

-- Step 3: Insert overlapping bookings.
INSERT INTO bookings VALUES (1, '[2025-01-01 09:00, 2025-01-01 10:00)');
INSERT INTO bookings VALUES (1, '[2025-01-01 09:30, 2025-01-01 10:30)');
```

**Expected output:**

```text
CREATE EXTENSION
CREATE TABLE
INSERT 0 1
ERROR:  conflicting key value violates exclusion constraint "bookings_room_id_during_excl"
DETAIL:  Key (room_id, during)=(1, ["2025-01-01 09:30:00","2025-01-01 10:30:00")) conflicts with existing key (room_id, during)=(1, ["2025-01-01 09:00:00","2025-01-01 10:00:00")).
```

**Why this output appears**: The `EXCLUDE` constraint uses `&&` (overlap) to reject a booking that overlaps with an existing booking for the same room.

#### Real-World Cases

- **Hotel reservations**: An `EXCLUDE` constraint on `(room_id, stay_period)` prevents double-booking of the same room.
- **Conference scheduling**: An `EXCLUDE` constraint on `(speaker_id, time_slot)` prevents a speaker from being scheduled in two sessions at the same time.

---

### 8. Composite Constraints

#### Definitions

**Core Definition**
A composite constraint is a constraint that spans multiple columns rather than a single column.

**Technical Definition**
Composite constraints are table-level constraints that reference two or more columns. They include composite `PRIMARY KEY`, composite `UNIQUE`, composite `FOREIGN KEY`, and multi-column `CHECK`. A composite primary key allows duplicate values in individual columns but requires the combination of all key columns to be unique. A composite foreign key must reference a composite primary key or unique key in the parent table.

**Beginner-Friendly Explanation**
A composite constraint is a rule that involves more than one column. For example, “the combination of `first_name` and `last_name` must be unique” is a composite constraint.

#### Purposes

- To enforce uniqueness across a combination of columns (e.g., a composite natural key).
- To model many-to-many relationships with a junction table that has a composite primary key.
- To enforce referential integrity for relationships that use composite keys.

#### Syntax Rules and Structure

**Composite PRIMARY KEY:**

```sql
PRIMARY KEY ( column1, column2 [, ...] )
```

**Composite UNIQUE:**

```sql
UNIQUE ( column1, column2 [, ...] )
```

**Composite FOREIGN KEY:**

```sql
FOREIGN KEY ( column1, column2 [, ...] )
    REFERENCES parent_table ( parent_column1, parent_column2 [, ...] )
```

**Composite CHECK:**

```sql
CHECK ( column1 > column2 )
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `(column1, column2, ...)` | The columns that form the composite constraint. |

**Syntax rules:**

- The column order in a composite constraint matters for index performance and uniqueness semantics.
- A composite foreign key must reference the same number of columns in the same order as the parent key.

**Constraints and limitations:**

- Composite primary keys create composite indexes; the column order affects query performance.
- A composite foreign key cannot reference a parent key with a different number of columns.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Composite unique constraint

```sql
-- Step 1: Create a table with a composite unique constraint.
CREATE TABLE product_variants (
    product_id int,
    color      text,
    size       text,
    UNIQUE (product_id, color, size)
);

-- Step 2: Insert duplicate combinations.
INSERT INTO product_variants VALUES (1, 'red', 'M');
INSERT INTO product_variants VALUES (1, 'red', 'M');
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
ERROR:  duplicate key value violates unique constraint "product_variants_product_id_color_size_key"
DETAIL:  Key (product_id, color, size)=(1, red, M) already exists.
```

**Why this output appears**: The composite unique constraint ensures that no two rows have the same combination of `product_id`, `color`, and `size`.

#### Real-World Cases

- **Inventory**: A `product_variants` table uses a composite unique key of `(product_id, color, size)` to prevent duplicate variants.
- **Junction tables**: A `post_tags` table uses a composite primary key of `(post_id, tag_id)` to prevent duplicate tag assignments.

---

### 9. Constraint Naming

#### Definitions

**Core Definition**
Constraint naming is the practice of assigning explicit, meaningful names to constraints instead of relying on system-generated names.

**Technical Definition**
In PostgreSQL, MySQL, and SQL Server, constraints can be named using the `CONSTRAINT constraint_name` clause. If a name is not specified, the system generates one (e.g., `table_name_column_name_key` in PostgreSQL, or an auto-generated name in SQL Server). Named constraints appear in error messages, making debugging easier, and can be referenced in `ALTER TABLE ... DROP CONSTRAINT` statements.

**Beginner-Friendly Explanation**
Naming a constraint is like giving it a label so you can find it easily later. If you don't name it, the database makes up a name that may be hard to remember.

#### Purposes

- To make error messages more readable and informative for developers and DBAs.
- To enable reliable `DROP CONSTRAINT` and `ALTER CONSTRAINT` operations by name.
- To enforce naming conventions across a team or organisation.

#### Syntax Rules and Structure

**General syntax:**

```sql
CONSTRAINT constraint_name constraint_definition
```

**PostgreSQL:** `[ CONSTRAINT constraint_name ] { NOT NULL | UNIQUE | PRIMARY KEY | CHECK ... }`.

**MySQL:** `[CONSTRAINT [symbol]] PRIMARY KEY | UNIQUE | FOREIGN KEY | CHECK`.

**SQL Server:** `[CONSTRAINT constraint_name] PRIMARY KEY | UNIQUE | FOREIGN KEY | CHECK | DEFAULT`.

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `CONSTRAINT constraint_name` | The optional name for the constraint. |

**Syntax rules:**

- Constraint names must follow the RDBMS's identifier rules (length, allowed characters).
- In SQL Server, constraint names must be unique within the schema; in PostgreSQL, within the table.
- Named constraints appear in error messages, which is invaluable for debugging.

**Constraints and limitations:**

- System-generated names can vary across RDBMS versions and installations, making migrations less portable.
- Renaming a constraint requires `ALTER TABLE ... RENAME CONSTRAINT` in PostgreSQL or `sp_rename` in SQL Server.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Named CHECK constraint

```sql
-- Step 1: Create a table with a named CHECK constraint.
CREATE TABLE employees (
    id     serial PRIMARY KEY,
    salary numeric CONSTRAINT chk_salary_positive CHECK (salary > 0)
);

-- Step 2: Violate the constraint.
INSERT INTO employees (salary) VALUES (-100);
```

**Expected output:**

```text
CREATE TABLE
ERROR:  new row for relation "employees" violates check constraint "chk_salary_positive"
DETAIL:  Failing row contains (1, -100).
```

**Why this output appears**: The explicit constraint name `chk_salary_positive` appears in the error message, making it clear which rule was violated.

#### Real-World Cases

- **Debugging**: A named constraint `chk_order_total_positive` immediately tells a developer what went wrong.
- **Migrations**: A migration script drops a constraint by name, which is only possible if the name was explicitly assigned.

---

### 10. Constraint Enforcement

#### Definitions

**Core Definition**
Constraint enforcement is the mechanism by which the database engine checks and applies constraints during data modification operations.

**Technical Definition**
Constraints are enforced by the database engine either declaratively (via built-in mechanisms such as unique indexes and foreign-key triggers) or procedurally (via user-defined triggers). Declarative constraints are preferred for standard integrity rules because they are optimised and less error-prone. Triggers are used for complex business rules that cannot be expressed declaratively. In PostgreSQL, foreign-key constraints are implemented internally as triggers; in MySQL and SQL Server, they are enforced by the storage engine or query processor. Constraint enforcement can be toggled in some systems: MySQL uses `SET FOREIGN_KEY_CHECKS = 0`; SQL Server uses `WITH NOCHECK`; PostgreSQL uses `NOT VALID` for `CHECK` and `FOREIGN KEY` constraints.

**Beginner-Friendly Explanation**
Constraint enforcement is how the database actually makes sure the rules are followed. Some rules are built into the database engine (like `UNIQUE` and `FOREIGN KEY`), while others require triggers—special programs that run automatically when data changes.

#### Purposes

- To ensure that all data modifications respect the declared integrity rules.
- To choose between declarative enforcement (built-in, optimised) and procedural enforcement (triggers, for complex rules).
- To temporarily disable enforcement during bulk loads or migrations (with caution).

#### Syntax Rules and Structure

**Disabling enforcement (examples):**

**MySQL:**

```sql
SET FOREIGN_KEY_CHECKS = 0;
-- perform operations
SET FOREIGN_KEY_CHECKS = 1;
```


**SQL Server:**

```sql
ALTER TABLE table_name NOCHECK CONSTRAINT constraint_name;
-- or
ALTER TABLE table_name WITH NOCHECK ADD CONSTRAINT constraint_name ...;
```


**PostgreSQL:**

```sql
ALTER TABLE table_name ADD CONSTRAINT constraint_name ... NOT VALID;
-- later
ALTER TABLE table_name VALIDATE CONSTRAINT constraint_name;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `FOREIGN_KEY_CHECKS` (MySQL) | Session variable that toggles foreign-key enforcement. |
| `WITH NOCHECK` (SQL Server) | Adds or re-enables a constraint without validating existing data. |
| `NOT VALID` (PostgreSQL) | Adds a constraint without validating existing rows. |

**Syntax rules:**

- Disabling enforcement is a session-level or table-level operation and should be used only for controlled scenarios (e.g., data import, migration).
- `WITH NOCHECK` in SQL Server means the query optimizer may ignore the constraint, potentially leading to poor plans.
- MySQL's `FOREIGN_KEY_CHECKS` does not affect `UNIQUE` or `CHECK` constraints; it only affects foreign keys.

**Constraints and limitations:**

- Disabling enforcement can allow invalid data to enter the database, which may cause errors when enforcement is re-enabled.
- Re-enabling enforcement without validating existing data leaves the constraint in a `NOT VALID` or `NOCHECK` state, and the optimizer may not trust it.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Disabling foreign-key checks during import

```sql
-- Step 1: Disable foreign-key checks.
SET FOREIGN_KEY_CHECKS = 0;

-- Step 2: Insert data that would normally violate foreign keys.
INSERT INTO child (parent_id) VALUES (999);

-- Step 3: Re-enable checks.
SET FOREIGN_KEY_CHECKS = 1;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.00 sec)
Query OK, 1 row affected (0.01 sec)
Query OK, 0 rows affected (0.00 sec)
```

**Why this output appears**: With `FOREIGN_KEY_CHECKS = 0`, MySQL does not validate the foreign-key relationship, allowing the insert to succeed. After re-enabling, future operations are checked, but the existing invalid row remains.

#### Real-World Cases

- **Data migration**: A migration script disables foreign-key checks to load tables in an order that would otherwise violate dependencies, then re-enables checks after the load.
- **Bulk import**: A data import tool uses `WITH NOCHECK` (SQL Server) or `NOT VALID` (PostgreSQL) to avoid scanning large tables during constraint creation, then validates separately.

---

### 11. Constraint Dependency

#### Definitions

**Core Definition**
Constraint dependency refers to the relationships between constraints and other database objects, such that dropping or altering one object may be blocked or may cascade to dependent objects.

**Technical Definition**
In PostgreSQL, the system catalog `pg_depend` tracks dependencies between objects. A `FOREIGN KEY` constraint depends on the referenced `PRIMARY KEY` or `UNIQUE` constraint; a view depends on the tables it references. `DROP TABLE` or `DROP CONSTRAINT` fails if dependent objects exist, unless `CASCADE` is specified. In SQL Server and MySQL, foreign-key dependencies similarly prevent dropping a referenced key. When `CASCADE` is used in PostgreSQL to drop a table referenced by a foreign key, only the foreign-key constraint is removed—not the referencing table itself.

**Beginner-Friendly Explanation**
Constraint dependency means that constraints are connected to each other. If you try to delete a primary key that a foreign key depends on, the database stops you unless you say “CASCADE” to remove the dependent foreign key too.

#### Purposes

- To understand why a `DROP TABLE` or `DROP CONSTRAINT` might fail.
- To safely resolve dependencies before performing schema changes.
- To use `CASCADE` judiciously to remove dependent objects when appropriate.

#### Syntax Rules and Structure

**Dropping with CASCADE (PostgreSQL):**

```sql
ALTER TABLE table_name DROP CONSTRAINT constraint_name CASCADE;
DROP TABLE table_name CASCADE;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `CASCADE` | Automatically drops dependent objects. |
| `RESTRICT` | Refuses to drop if dependencies exist (default). |

**Syntax rules:**

- In PostgreSQL, `CASCADE` on a `DROP TABLE` removes dependent foreign-key constraints but does not drop the referencing table itself.
- `CASCADE` is recursive: it drops objects that depend on the dropped object, and objects that depend on those objects, and so on.

**Constraints and limitations:**

- `CASCADE` can have far-reaching and unintended consequences; it should be used only after inspecting dependencies.
- In MySQL and SQL Server, `CASCADE` is not available for `DROP TABLE`; dependencies must be resolved manually.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Dependency error and CASCADE resolution

```sql
-- Step 1: Create a parent table with a primary key and a child table with a foreign key.
CREATE TABLE parent (id int PRIMARY KEY);
CREATE TABLE child (id int PRIMARY KEY, parent_id int REFERENCES parent(id));

-- Step 2: Attempt to drop the parent table without CASCADE.
DROP TABLE parent;
```

**Expected output:**

```text
CREATE TABLE
CREATE TABLE
ERROR:  cannot drop table parent because other objects depend on it
DETAIL:  constraint child_parent_id_fkey on table child depends on table parent
HINT:  Use DROP ... CASCADE to drop the dependent objects too.
```

```sql
-- Step 3: Drop with CASCADE.
DROP TABLE parent CASCADE;
```

**Expected output:**

```text
NOTICE:  drop cascades to constraint child_parent_id_fkey on table child
DROP TABLE
```

**Why this output appears**: The foreign-key constraint depends on the parent table. `CASCADE` removes the constraint, but not the `child` table.

#### Real-World Cases

- **Schema refactoring**: Dropping a legacy primary key requires first dropping the foreign keys that reference it, or using `CASCADE`.
- **Migration rollback**: A migration that adds a foreign key must be reversed by dropping the foreign key before dropping the referenced unique constraint.

---

### 12. Deferrable Constraints (PostgreSQL)

#### Definitions

**Core Definition**
A deferrable constraint is a constraint whose checking can be postponed until the end of the transaction rather than being checked immediately after each statement.

**Technical Definition**
In PostgreSQL, `UNIQUE`, `PRIMARY KEY`, `EXCLUDE`, and `FOREIGN KEY` constraints can be declared `DEFERRABLE`. A `DEFERRABLE` constraint can be either `INITIALLY IMMEDIATE` (checked after each statement, the default) or `INITIALLY DEFERRED` (checked only at transaction commit). The `SET CONSTRAINTS` command can change the timing within a transaction. `NOT NULL` and `CHECK` constraints are **not** deferrable.

**Beginner-Friendly Explanation**
A deferrable constraint is like a homework assignment that can be checked at the end of the day instead of after every problem. This is useful when you need to temporarily violate a rule during a transaction and fix it before committing.

#### Purposes

- To allow temporary violations of a constraint during a multi-statement transaction, as long as the final state is valid.
- To support circular foreign-key references that would otherwise be impossible to insert.
- To reduce the overhead of immediate constraint checking during complex transactions.

#### Syntax Rules and Structure

**General syntax:**

```sql
constraint_definition [ DEFERRABLE | NOT DEFERRABLE ] [ INITIALLY IMMEDIATE | INITIALLY DEFERRED ]
```

**PostgreSQL:** Supports `DEFERRABLE` for `UNIQUE`, `PRIMARY KEY`, `EXCLUDE`, and `REFERENCES` (foreign key).

**SET CONSTRAINTS:**

```sql
SET CONSTRAINTS { ALL | constraint_name [, ...] } { DEFERRED | IMMEDIATE };
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `DEFERRABLE` | Allows the constraint to be deferred. |
| `INITIALLY DEFERRED` | Checks the constraint only at transaction commit. |
| `SET CONSTRAINTS` | Changes the timing within a transaction. |

**Syntax rules:**

- `NOT DEFERRABLE` is the default; such constraints are checked immediately after every command.
- `DEFERRABLE INITIALLY DEFERRED` constraints are checked only at transaction commit.
- Deferrable constraints cannot be used as conflict arbitrators in `INSERT ... ON CONFLICT DO UPDATE`.

**Constraints and limitations:**

- Only `UNIQUE`, `PRIMARY KEY`, `EXCLUDE`, and `FOREIGN KEY` constraints can be deferrable in PostgreSQL.
- Deferred constraint checks happen at commit, which can make debugging harder because the error appears late in the transaction.
- MySQL and SQL Server do not support deferrable constraints in the same way.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Deferrable foreign key for circular references

```sql
-- Step 1: Create two tables with circular foreign keys.
CREATE TABLE departments (
    dept_id int PRIMARY KEY,
    manager_id int
);

CREATE TABLE employees (
    emp_id int PRIMARY KEY,
    dept_id int REFERENCES departments(dept_id) DEFERRABLE INITIALLY DEFERRED
);

ALTER TABLE departments
    ADD CONSTRAINT fk_manager
    FOREIGN KEY (manager_id) REFERENCES employees(emp_id)
    DEFERRABLE INITIALLY DEFERRED;

-- Step 2: Insert in a transaction; constraints are checked at commit.
BEGIN;
INSERT INTO departments (dept_id, manager_id) VALUES (1, 100);
INSERT INTO employees (emp_id, dept_id) VALUES (100, 1);
COMMIT;
```

**Expected output:**

```text
CREATE TABLE
CREATE TABLE
ALTER TABLE
BEGIN
INSERT 0 1
INSERT 0 1
COMMIT
```

**Why this output appears**: The circular foreign keys are declared `DEFERRABLE INITIALLY DEFERRED`, so the intermediate violation (departments references employees before the employee exists) is allowed until the transaction commits, at which point both rows exist and the constraint is satisfied.

#### Real-World Cases

- **Circular references**: A `departments` table and an `employees` table reference each other; deferrable constraints allow both to be inserted in a single transaction.
- **Bulk data loading**: Deferring foreign-key checks until commit reduces the number of individual checks during a large transaction.

---

### 13. Constraint Timing

#### Definitions

**Core Definition**
Constraint timing refers to when a constraint is checked: immediately after each statement or deferred until transaction commit.

**Technical Definition**
In PostgreSQL, each constraint has one of three timing characteristics: `NOT DEFERRABLE` (checked immediately after every command), `DEFERRABLE INITIALLY IMMEDIATE` (checked after each statement but can be deferred), or `DEFERRABLE INITIALLY DEFERRED` (checked only at transaction commit). The `SET CONSTRAINTS` command can switch a deferrable constraint between immediate and deferred mode within a transaction.

**Beginner-Friendly Explanation**
Constraint timing is about **when** the database checks the rule. Immediate means “check right now, after every statement.” Deferred means “check at the end of the transaction.”

#### Purposes

- To control the trade-off between early error detection (immediate) and flexibility for multi-statement transactions (deferred).
- To allow temporary violations during a transaction that will be resolved before commit.
- To optimise performance by reducing the number of constraint checks during bulk operations.

#### Syntax Rules and Structure

**PostgreSQL:** Timing is specified at constraint creation:

```sql
[ DEFERRABLE | NOT DEFERRABLE ] [ INITIALLY IMMEDIATE | INITIALLY DEFERRED ]
```

**SET CONSTRAINTS:**

```sql
SET CONSTRAINTS { ALL | constraint_name [, ...] } { DEFERRED | IMMEDIATE };
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `INITIALLY IMMEDIATE` | Check after each statement (default for deferrable constraints). |
| `INITIALLY DEFERRED` | Check only at transaction commit. |
| `SET CONSTRAINTS ... DEFERRED` | Switch to deferred mode within a transaction. |

**Syntax rules:**

- `NOT DEFERRABLE` constraints are always immediate.
- `SET CONSTRAINTS` affects only the current transaction.
- `SET CONSTRAINTS ALL DEFERRED` defers all deferrable constraints in the transaction.

**Constraints and limitations:**

- Immediate constraints provide faster feedback and are easier to debug.
- Deferred constraints can mask errors until commit, making debugging harder.
- MySQL and SQL Server do not support `SET CONSTRAINTS`; their constraint timing is generally immediate.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – SET CONSTRAINTS to defer a unique constraint

```sql
-- Step 1: Create a deferrable unique constraint.
CREATE TABLE t (id int, UNIQUE (id) DEFERRABLE INITIALLY IMMEDIATE);

-- Step 2: Begin a transaction and defer the constraint.
BEGIN;
SET CONSTRAINTS ALL DEFERRED;

-- Step 3: Insert duplicate values.
INSERT INTO t VALUES (1);
INSERT INTO t VALUES (1);

-- Step 4: Commit; the constraint fails at commit.
COMMIT;
```

**Expected output:**

```text
CREATE TABLE
BEGIN
SET CONSTRAINTS
INSERT 0 1
INSERT 0 1
ERROR:  duplicate key value violates unique constraint "t_id_key"
```

**Why this output appears**: The constraint is deferred within the transaction, so the duplicate inserts are allowed. At `COMMIT`, the constraint is checked and the transaction fails.

#### Real-World Cases

- **Multi-step data fixes**: A data correction requires temporarily violating a unique constraint; the constraint is deferred until the end of the transaction.
- **Bulk loading**: Deferring foreign-key checks during a large import reduces the number of individual lookups.

---

### 14. Constraint Validation

#### Definitions

**Core Definition**
Constraint validation is the process of checking that existing data satisfies a newly added or re-enabled constraint.

**Technical Definition**
In PostgreSQL, `ALTER TABLE ... ADD CONSTRAINT ... NOT VALID` adds a `CHECK` or `FOREIGN KEY` constraint without scanning existing rows. The constraint is enforced for new rows immediately, but existing rows are not validated until `ALTER TABLE ... VALIDATE CONSTRAINT` is run. `VALIDATE CONSTRAINT` takes a `SHARE UPDATE EXCLUSIVE` lock, which does not block reads or writes. In SQL Server, `WITH CHECK` (the default for new constraints) validates existing data; `WITH NOCHECK` skips validation. Re-enabling a constraint with `WITH CHECK CHECK CONSTRAINT` validates existing data.

**Beginner-Friendly Explanation**
Constraint validation is how you check that the data already in the table follows a new rule. If you add a rule without validation (`NOT VALID` or `WITH NOCHECK`), the rule applies to new data only until you run validation.

#### Purposes

- To add constraints to large tables without holding a long lock that blocks reads and writes.
- To validate existing data after a constraint has been added as `NOT VALID`.
- To ensure that the query optimizer can trust the constraint for better execution plans.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
ALTER TABLE table_name ADD CONSTRAINT constraint_name ... NOT VALID;
ALTER TABLE table_name VALIDATE CONSTRAINT constraint_name;
```


**SQL Server:**

```sql
ALTER TABLE table_name WITH CHECK ADD CONSTRAINT constraint_name ...;
-- or
ALTER TABLE table_name WITH NOCHECK ADD CONSTRAINT constraint_name ...;
-- later
ALTER TABLE table_name WITH CHECK CHECK CONSTRAINT constraint_name;
```


**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `NOT VALID` (PostgreSQL) | Skips validation of existing rows. |
| `VALIDATE CONSTRAINT` (PostgreSQL) | Validates a `NOT VALID` constraint. |
| `WITH CHECK` (SQL Server) | Validates existing data. |
| `WITH NOCHECK` (SQL Server) | Skips validation of existing data. |

**Syntax rules:**

- `NOT VALID` is allowed only for `CHECK` and `FOREIGN KEY` constraints in PostgreSQL.
- `VALIDATE CONSTRAINT` should be run in a separate transaction from the `ADD CONSTRAINT ... NOT VALID` for maximum benefit.
- The query optimizer does not assume a `NOT VALID` constraint holds for all rows, so it may miss optimisations until validation is complete.

**Constraints and limitations:**

- Validation scans the entire table and can be slow on large tables, but it does not block reads or writes (PostgreSQL).
- In SQL Server, `WITH NOCHECK` constraints are ignored by the query optimizer until re-enabled with `WITH CHECK CHECK CONSTRAINT`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – NOT VALID then VALIDATE

```sql
-- Step 1: Add a CHECK constraint as NOT VALID.
ALTER TABLE orders ADD CONSTRAINT chk_amount_positive CHECK (amount > 0) NOT VALID;

-- Step 2: Validate the constraint.
ALTER TABLE orders VALIDATE CONSTRAINT chk_amount_positive;
```

**Expected output:**

```text
ALTER TABLE
ALTER TABLE
```

**Why this output appears**: The `NOT VALID` constraint is added without scanning existing rows. `VALIDATE CONSTRAINT` then scans the table to verify that all existing rows satisfy the constraint.

#### Real-World Cases

- **Production constraint addition**: A DBA adds a `CHECK` constraint as `NOT VALID` during business hours, then validates it during a low-traffic period.
- **Data migration**: After importing data, foreign-key constraints are added as `NOT VALID` to avoid blocking the import, then validated separately.

---

### 15. Constraint Performance

#### Definitions

**Core Definition**
Constraint performance refers to the overhead that constraints impose on database write operations and the impact they have on query optimisation.

**Technical Definition**
Constraints incur performance costs primarily on `INSERT`, `UPDATE`, and `DELETE` operations. `UNIQUE` and `PRIMARY KEY` constraints create unique B-tree indexes; every write must update these indexes and perform an index lookup to check for duplicates. `FOREIGN KEY` constraints require lookups against the parent table on insert and update of the child table, and may require cascading actions on delete or update. `CHECK` constraints evaluate a Boolean expression for every affected row. `NOT NULL` and `DEFAULT` constraints have minimal overhead. Constraints generally do not penalise read queries, and the query optimizer can use constraints to produce better execution plans (e.g., eliminating joins or scans).

**Beginner-Friendly Explanation**
Constraints make writes slower because the database has to do extra work to check the rules. `UNIQUE` and `PRIMARY KEY` are the most expensive because they maintain indexes; `FOREIGN KEY` also adds lookup costs. Reads are not slowed down by constraints, and constraints can actually help the optimizer choose faster plans.

#### Purposes

- To understand the trade-off between data integrity and write performance.
- To choose which constraints are necessary and which can be omitted or deferred.
- To design indexes and constraints that support both integrity and query performance.

#### Syntax Rules and Structure

Performance is not controlled by syntax but by design decisions:

- **Minimise redundant indexes**: A `UNIQUE` constraint and a separate unique index on the same columns are redundant; use one or the other.
- **Index foreign-key columns**: While PostgreSQL does not automatically index foreign-key columns, doing so speeds up cascading deletes and joins.
- **Use `NOT VALID` for large tables**: Adding constraints as `NOT VALID` and validating separately avoids long locks and allows the operation to complete without blocking.
- **Consider deferrable constraints for bulk operations**: Deferring checks until commit can reduce per-statement overhead during large transactions.

**Constraints and limitations:**

- Every index, including those created by `UNIQUE` and `PRIMARY KEY` constraints, incurs a write penalty.
- `FOREIGN KEY` and `UNIQUE` constraints are the most expensive because they require reads against external state on every insert.
- High-contention environments may see disproportionate overhead from unique constraint checks.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Measuring write overhead with and without a unique constraint

```sql
-- Step 1: Create two tables, one with a unique constraint and one without.
CREATE TABLE with_unique (id int UNIQUE, data text);
CREATE TABLE without_unique (id int, data text);

-- Step 2: Insert 100,000 rows into each and compare timing.
\timing on
INSERT INTO with_unique SELECT g, md5(random()::text) FROM generate_series(1, 100000) AS g;
INSERT INTO without_unique SELECT g, md5(random()::text) FROM generate_series(1, 100000) AS g;
```

**Expected output** (illustrative; actual timings vary):

```text
CREATE TABLE
CREATE TABLE
Timing is on.
INSERT 0 100000
Time: 245.123 ms
INSERT 0 100000
Time: 120.456 ms
```

**Why this output appears**: The table with the unique constraint performs an index lookup and index update for every row, roughly doubling the insert time compared to the table without the constraint.

#### Real-World Cases

- **High-throughput ingestion**: A logging table may omit foreign keys and unique constraints to maximise insert throughput, relying on application-level validation instead.
- **OLTP systems**: Primary keys and foreign keys are essential for integrity and join performance, even though they add write overhead; the trade-off is almost always worth it for transactional systems.

---

## References

- PostgreSQL Documentation – Constraints – https://www.postgresql.org/docs/current/ddl-constraints.html
- PostgreSQL Documentation – `CREATE TABLE` – https://www.postgresql.org/docs/current/sql-createtable.html
- PostgreSQL Documentation – Dependency Tracking – https://www.postgresql.org/docs/current/ddl-depend.html
- PostgreSQL Documentation – `SET CONSTRAINTS` – https://www.postgresql.org/docs/current/sql-set-constraints.html
- PostgreSQL Documentation – Deferrable Constraints (mailing list discussion) – https://www.postgresql.org/message-id/0317c0c5-a7e5-9073-01b9-80391491108b%40aklaver.com
- MySQL 8.0 Reference Manual – `CREATE TABLE` – https://dev.mysql.com/doc/refman/8.0/en/create-table.html
- MySQL 8.0 Reference Manual – Server SQL Modes (foreign_key_checks) – https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html
- Microsoft Learn – `column_constraint` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/column-constraint-transact-sql
- Microsoft Learn – `table_constraint` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/table-constraint-transact-sql
- Microsoft Learn – `ALTER TABLE ... WITH CHECK | WITH NOCHECK` – https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-table-transact-sql
- Microsoft Learn – Best Practice: Naming Constraints – https://learn.microsoft.com/en-us/archive/blogs/sqlserverfaq/best-practice-naming-constraints
- Stack Overflow – How does unique constraint affect write performance in Postgres DB – https://stackoverflow.com/questions/13190682/how-does-unique-constraint-affect-write-performance-in-postgres-db
- PostgreSQL Wiki – Constraints Are Slow! – https://wiki.postgresql.org/wiki/Constraints_Are_Slow
- Tiger Data – The Hidden Cost of Postgres Constraints at Scale – https://www.tigerdata.com/blog/the-hidden-cost-of-postgres-constraints-at-scale
- PostgreSQL Documentation – `NOT VALID` / `VALIDATE CONSTRAINT` – https://www.postgresql.org/docs/current/sql-altertable.html
- Oracle Database SQL Language Reference – `DROP CONSTRAINT ... CASCADE` – https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/constraint.html