# SQL Table Constraints: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL table constraints are declarative rules enforced by the database engine that restrict the data that can be stored in a table, ensuring data integrity, accuracy, and reliability.

**Technical Definition:** Per the ANSI/ISO SQL Standard (ISO/IEC 9075), a constraint is an assertion about the data in a database that must always be true. When a constraint is enabled, the assertion is guaranteed to hold. Constraints are defined as part of the Data Definition Language (DDL) in `CREATE TABLE` or `ALTER TABLE` statements and are checked during Data Manipulation Language (DML) operations. The primary constraint types are NOT NULL, CHECK, UNIQUE, PRIMARY KEY, and FOREIGN KEY.

**Beginner-Friendly Explanation:** Think of a database table as a form with rules written on it. "This field must be filled in" (NOT NULL). "No two people can have the same ID number" (UNIQUE/PRIMARY KEY). "The age must be between 0 and 150" (CHECK). "This department code must exist in the departments table" (FOREIGN KEY). These rules are constraints, and the database automatically enforces them every time you try to insert, update, or delete data.

### Key Characteristics

- **Declarative:** Constraints are defined once in the schema and automatically enforced by the database engine for all future operations.
- **Standardized:** The core constraint types are defined by the ANSI/ISO SQL Standard, ensuring portability across database systems.
- **Scoped:** Constraints apply at the column level (single column) or the table level (one or more columns, including composite keys).
- **Named:** Constraints can be explicitly named for manageability, or the database assigns a system-generated name if omitted.
- **Deferrable:** Some constraints can be deferred until the end of a transaction, allowing temporary violations during complex multi-statement operations.
- **Enforcement-aware:** Constraints can be validated against existing data (`WITH CHECK`) or added without validation (`WITH NOCHECK`), affecting query optimization.

### Prerequisites

- Basic SQL syntax (CREATE TABLE, ALTER TABLE, INSERT, UPDATE, DELETE)
- Understanding of data types
- Familiarity with NULL handling and three-valued logic
- Basic knowledge of indexes (PRIMARY KEY and UNIQUE constraints automatically create indexes)

### Related Programming Areas

- **Database Design and Normalization:** Constraints are fundamental to implementing a correct logical data model.
- **Data Integrity and Validation:** Constraints enforce domain, entity, and referential integrity at the database level.
- **Application Development:** Constraints provide a safety net that catches invalid data before it corrupts the database.
- **Database Migration:** Constraint names and behaviors are a key consideration when moving schemas between database systems.
- **Performance Tuning:** Constraint definitions influence the query optimizer's ability to generate efficient execution plans.

---

## Core Concepts / Key Features

### 1. Core Constraint Types

#### 1.1 NOT NULL (Preventing Missing Values at the Column Level)

**Core Definitions:**

- **Core Definition:** The NOT NULL constraint specifies that a column does not accept NULL values.
- **Technical Definition:** NOT NULL is a column-level constraint that enforces domain integrity by requiring every row to have a non-NULL value in the specified column. Unlike other constraints, NOT NULL cannot be deferred and cannot be defined at the table level.
- **Beginner-Friendly Explanation:** NOT NULL is like a required field on a form. You must fill it in—you can't leave it blank.

**Purposes:**

- To ensure critical columns always have a value
- To prevent missing data in essential fields (names, IDs, dates)
- To simplify application logic by guaranteeing a value is present
- To support PRIMARY KEY columns, which implicitly require NOT NULL

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- Column-level definition
column_name data_type NOT NULL

-- Add to existing table
ALTER TABLE table_name ALTER COLUMN column_name data_type NOT NULL;
```

**Component Breakdown:**

- `column_name`: The column to which the constraint applies
- `NOT NULL`: The keyword phrase indicating the constraint
- Cannot be defined as a table-level constraint

**Syntax Rules:**

- NOT NULL is the only constraint that cannot be defined at the table level.
- NOT NULL constraints cannot be deferred.
- A column with NOT NULL can still have a DEFAULT value, which is used when no value is supplied.

**Constraints and Limitations:**

- Adding NOT NULL to a column with existing NULLs will fail unless the NULLs are updated first.
- NOT NULL is not named (it does not appear in the information schema as a named constraint in most databases).

**Annotated Code Examples:**

```sql
-- Setup: Create a table with NOT NULL
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    email VARCHAR(200),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Valid insert (email can be NULL, but name and created_at cannot)
INSERT INTO customers (customer_id, customer_name, email)
VALUES (1, 'Alice', 'alice@example.com');

-- Invalid insert (customer_name cannot be NULL)
INSERT INTO customers (customer_id, customer_name)
VALUES (2, NULL);
-- Error: null value in column "customer_name" violates not-null constraint
```

**Expected Output (valid insert):**

| customer_id | customer_name | email              | created_at          |
|-------------|---------------|--------------------|---------------------|
| 1           | Alice         | alice@example.com  | 2026-09-19 10:30:00 |

**Explanation:** The NOT NULL constraint on `customer_name` prevents the second insert. The `created_at` column has a DEFAULT, so it is automatically populated.

**Real-World Cases:**

- **User registration:** Usernames and email addresses must be provided.
- **Order processing:** Order dates and customer IDs are mandatory.
- **Financial systems:** Transaction amounts cannot be NULL.

**References:**

- Microsoft Learn: Constraints (NOT NULL) - https://learn.microsoft.com/en-us/sql/relational-databases/tables/unique-constraints-and-check-constraints
- PostgreSQL: Constraints - https://www.postgresql.org/docs/current/ddl-constraints.html


#### 1.2 UNIQUE (Ensuring Distinct Values Across a Column or Column Group)

**Core Definitions:**

- **Core Definition:** The UNIQUE constraint ensures that all values in a column, or a group of columns, are distinct across all rows in the table.
- **Technical Definition:** A UNIQUE constraint enforces entity integrity by preventing duplicate values. Unlike PRIMARY KEY, UNIQUE constraints allow NULL values (though the number of NULLs allowed depends on the database dialect). A table can have multiple UNIQUE constraints, and they can be defined at the column level or table level.
- **Beginner-Friendly Explanation:** UNIQUE means "no duplicates allowed." For example, no two customers can have the same email address.

**Purposes:**

- To prevent duplicate values in a column or column group
- To enforce natural keys (e.g., email addresses, social security numbers)
- To support FOREIGN KEY references (the referenced columns must be UNIQUE or PRIMARY KEY)
- To allow multiple NULLs while still preventing duplicate non-NULL values

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- Column-level
column_name data_type UNIQUE

-- Table-level
CONSTRAINT constraint_name UNIQUE (column_name1, column_name2, ...)

-- Add to existing table
ALTER TABLE table_name ADD CONSTRAINT constraint_name UNIQUE (column_list);
```

**Component Breakdown:**

- `column_list`: One or more columns that together must be unique
- `CONSTRAINT constraint_name`: Optional explicit name for the constraint

**Syntax Rules:**

- A table can have multiple UNIQUE constraints.
- UNIQUE constraints automatically create a unique index (in most databases).
- UNIQUE constraints allow NULL values, but the handling of multiple NULLs varies:
  - **PostgreSQL, MySQL, SQLite:** Multiple NULLs are allowed (NULLs are considered distinct).
  - **SQL Server:** Only one NULL is allowed per column (treated as a value for uniqueness).
  - **Oracle:** Multiple NULLs are allowed.
- UNIQUE constraints can be deferred in PostgreSQL and Oracle.

**Constraints and Limitations:**

- Composite UNIQUE constraints enforce uniqueness across the combination of columns, not each column individually.
- UNIQUE constraints on nullable columns may behave unexpectedly with NULLs (see Section 5.2).

**Annotated Code Examples:**

```sql
-- Setup: Create a table with UNIQUE constraints
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(200) CONSTRAINT uq_users_email UNIQUE,
    phone VARCHAR(20)
);

-- Valid inserts
INSERT INTO users VALUES (1, 'alice', 'alice@example.com', '555-0100');
INSERT INTO users VALUES (2, 'bob', NULL, '555-0200');
INSERT INTO users VALUES (3, 'charlie', NULL, '555-0300');
-- Note: Multiple NULLs allowed in email (PostgreSQL behavior)

-- Invalid insert (duplicate username)
INSERT INTO users VALUES (4, 'alice', 'alice2@example.com', '555-0400');
-- Error: duplicate key value violates unique constraint "users_username_key"
```

**Expected Output (valid inserts):**

| user_id | username | email              | phone    |
|---------|----------|--------------------|----------|
| 1       | alice    | alice@example.com  | 555-0100 |
| 2       | bob      | NULL               | 555-0200 |
| 3       | charlie  | NULL               | 555-0300 |

**Explanation:** The UNIQUE constraint on `username` prevents the fourth insert. The UNIQUE constraint on `email` allows multiple NULLs in PostgreSQL.

```sql
-- Composite UNIQUE constraint
CREATE TABLE product_variants (
    product_id INT,
    variant_name VARCHAR(50),
    color VARCHAR(30),
    size VARCHAR(10),
    CONSTRAINT uq_product_variant UNIQUE (product_id, color, size)
);
-- No two rows can have the same combination of product_id, color, and size
```

**Real-World Cases:**

- **User management:** Unique usernames and email addresses.
- **Inventory:** Unique product SKUs or barcodes.
- **HR:** Unique employee IDs or social security numbers.

**References:**

- Microsoft Learn: UNIQUE Constraints - https://learn.microsoft.com/en-us/sql/relational-databases/tables/unique-constraints-and-check-constraints
- PostgreSQL: UNIQUE Constraints - https://www.postgresql.org/docs/current/ddl-constraints.html
- MySQL: UNIQUE Constraints - https://dev.mysql.com/doc/refman/8.0/en/create-table.html


#### 1.3 PRIMARY KEY (Uniquely Identifying Rows)

**Core Definitions:**

- **Core Definition:** The PRIMARY KEY constraint uniquely identifies each row in a table and is an implicit combination of NOT NULL and UNIQUE.
- **Technical Definition:** A PRIMARY KEY is a column or set of columns that uniquely identifies each row. It enforces entity integrity by requiring uniqueness and disallowing NULL values. A table can have only one PRIMARY KEY. When defined, it automatically creates a unique index (in most databases).
- **Beginner-Friendly Explanation:** The PRIMARY KEY is like a social security number for each row—it's unique, never missing, and identifies exactly one record.

**Purposes:**

- To provide a stable, unique identifier for each row
- To enforce entity integrity (no duplicates, no NULLs)
- To serve as the target for FOREIGN KEY references
- To optimize query performance through the automatically created index

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- Column-level (single column)
column_name data_type PRIMARY KEY

-- Table-level (single or composite)
CONSTRAINT constraint_name PRIMARY KEY (column_name1, column_name2, ...)

-- Add to existing table
ALTER TABLE table_name ADD CONSTRAINT constraint_name PRIMARY KEY (column_list);
```

**Component Breakdown:**

- `column_list`: One or more columns that form the primary key
- A composite primary key is defined at the table level (cannot be defined at the column level)

**Syntax Rules:**

- A table can have only one PRIMARY KEY.
- PRIMARY KEY columns cannot contain NULL values.
- PRIMARY KEY automatically creates a unique index (clustered in SQL Server by default, non-clustered in PostgreSQL/MySQL).
- Composite primary keys are always table-level constraints.

**Constraints and Limitations:**

- Cannot be defined on a nullable column.
- Cannot be deferred (in most databases).
- Changing a PRIMARY KEY requires dropping and recreating it.

**Annotated Code Examples:**

```sql
-- Single-column PRIMARY KEY
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(100) NOT NULL
);

-- Composite PRIMARY KEY (table-level)
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT NOT NULL,
    PRIMARY KEY (order_id, product_id)
);
-- No two rows can have the same order_id + product_id combination
```

**Expected Output (for composite PK):** Attempting to insert a duplicate combination fails.

**Real-World Cases:**

- **Order systems:** Order ID as PRIMARY KEY.
- **Junction tables:** Composite PRIMARY KEY of two foreign keys.
- **User management:** User ID as PRIMARY KEY.

**References:**

- Microsoft Learn: PRIMARY KEY Constraints - https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-primary-keys
- PostgreSQL: PRIMARY KEY - https://www.postgresql.org/docs/current/ddl-constraints.html
- Oracle: PRIMARY KEY Constraint - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/constraint.html


#### 1.4 FOREIGN KEY (Enforcing Referential Integrity Between Tables)

**Core Definitions:**

- **Core Definition:** A FOREIGN KEY constraint establishes a link between two tables, ensuring that values in the child table match values in the parent table's PRIMARY KEY or UNIQUE column.
- **Technical Definition:** A FOREIGN KEY constraint enforces referential integrity by preventing actions that would destroy the link between related tables. It is defined on the child table and references a candidate key (PRIMARY KEY or UNIQUE) in the parent table. The constraint can specify actions for `ON DELETE` and `ON UPDATE` to handle changes in the parent table.
- **Beginner-Friendly Explanation:** A FOREIGN KEY is like a reference in a book. If a chapter references page 42, page 42 must exist. Similarly, if an order references customer 5, customer 5 must exist in the customers table.

**Purposes:**

- To maintain referential integrity between related tables
- To prevent orphaned child records
- To enforce business relationships at the database level
- To enable cascading actions that automatically handle related data

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- Column-level
column_name data_type REFERENCES parent_table(parent_column)
    [ON DELETE action] [ON UPDATE action]

-- Table-level
CONSTRAINT constraint_name FOREIGN KEY (column_list)
    REFERENCES parent_table(parent_column_list)
    [ON DELETE action] [ON UPDATE action]

-- Reference options:
--   RESTRICT | CASCADE | SET NULL | SET DEFAULT | NO ACTION
```

**Component Breakdown:**

- `column_list`: The child table columns that reference the parent
- `parent_table(parent_column_list)`: The parent table and referenced columns
- `ON DELETE action`: What to do when a parent row is deleted
- `ON UPDATE action`: What to do when a parent key value is updated

**Syntax Rules:**

- The referenced columns must be PRIMARY KEY or UNIQUE in the parent table.
- The child and parent columns must have compatible data types.
- `SET NULL` requires the foreign key columns to be nullable.
- `SET DEFAULT` requires default values for the foreign key columns.

**Constraints and Limitations:**

- FOREIGN KEY constraints can cause performance overhead during DML operations.
- Circular references can complicate schema design and require deferrable constraints.
- Some databases do not support all reference options (e.g., Oracle does not support `ON UPDATE`).

**Annotated Code Examples:**

```sql
-- Setup: Parent and child tables with FOREIGN KEY
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(100) NOT NULL
);

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100) NOT NULL,
    dept_id INT,
    CONSTRAINT fk_emp_dept FOREIGN KEY (dept_id)
        REFERENCES departments(dept_id)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);

INSERT INTO departments VALUES (1, 'Engineering');
INSERT INTO departments VALUES (2, 'Marketing');

INSERT INTO employees VALUES (1, 'Alice', 1);
INSERT INTO employees VALUES (2, 'Bob', 2);

-- Valid insert
INSERT INTO employees VALUES (3, 'Charlie', 1);

-- Invalid insert (dept_id 99 does not exist)
INSERT INTO employees VALUES (4, 'Diana', 99);
-- Error: insert or update on table "employees" violates foreign key constraint
```

**Expected Output (valid inserts):**

| emp_id | emp_name | dept_id |
|--------|----------|---------|
| 1      | Alice    | 1       |
| 2      | Bob      | 2       |
| 3      | Charlie  | 1       |

**Explanation:** The FOREIGN KEY prevents insertion of a row with a non-existent department ID.

```sql
-- ON DELETE CASCADE demonstration
-- Deleting department 1 will delete all employees in that department
DELETE FROM departments WHERE dept_id = 1;
-- Alice and Charlie are automatically deleted
```

**Real-World Cases:**

- **E-commerce:** Orders reference customers; order items reference products.
- **HR systems:** Employees reference departments.
- **Content management:** Comments reference articles.

**References:**

- Microsoft Learn: FOREIGN KEY Constraints - https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-foreign-key-relationships
- MySQL: FOREIGN KEY Constraints - https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html
- PostgreSQL: Foreign Keys - https://www.postgresql.org/docs/current/ddl-constraints.html


#### 1.5 CHECK (Validating Custom Boolean Conditions)

**Core Definitions:**

- **Core Definition:** The CHECK constraint validates that column values satisfy a Boolean condition, enforcing domain integrity.
- **Technical Definition:** A CHECK constraint specifies a Boolean search condition that must evaluate to TRUE or UNKNOWN for each row. Values that evaluate to FALSE are rejected. Multiple CHECK constraints can be defined per column. CHECK constraints can be defined at the column level or table level.
- **Beginner-Friendly Explanation:** CHECK is like a validation rule on a form. For example, "age must be greater than 0" or "end date must be after start date."

**Purposes:**

- To enforce domain integrity (valid ranges, patterns, formats)
- To validate cross-column relationships (e.g., start_date < end_date)
- To implement business rules at the database level
- To prevent invalid data from entering the database

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- Column-level
column_name data_type CHECK (condition)

-- Table-level
CONSTRAINT constraint_name CHECK (condition)

-- Add to existing table
ALTER TABLE table_name ADD CONSTRAINT constraint_name CHECK (condition);
```

**Component Breakdown:**

- `condition`: A Boolean expression that must not evaluate to FALSE

**Syntax Rules:**

- CHECK constraints cannot reference columns in other tables.
- CHECK constraints cannot contain subqueries.
- CHECK constraints are evaluated for every INSERT and UPDATE.
- NULL values bypass CHECK constraints (because NULL comparisons evaluate to UNKNOWN, not FALSE).

**Constraints and Limitations:**

- CHECK constraints are not enforced for NULL values (UNKNOWN passes).
- Some databases limit the complexity of CHECK conditions.
- CHECK constraints cannot be deferred.

**Annotated Code Examples:**

```sql
-- Setup: Table with CHECK constraints
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2) CHECK (price > 0),
    discount DECIMAL(5,2) CHECK (discount >= 0 AND discount <= 100),
    category VARCHAR(50),
    CONSTRAINT chk_price_discount CHECK (price > discount)
);

-- Valid insert
INSERT INTO products VALUES (1, 'Widget', 19.99, 10.00, 'Hardware');

-- Invalid insert (price must be > 0)
INSERT INTO products VALUES (2, 'Free Item', 0, 0, 'Misc');
-- Error: new row for relation "products" violates check constraint "products_price_check"

-- Invalid insert (price must be > discount)
INSERT INTO products VALUES (3, 'Discounted', 5.00, 10.00, 'Hardware');
-- Error: new row for relation "products" violates check constraint "chk_price_discount"
```

**Expected Output (valid insert):**

| product_id | product_name | price | discount | category |
|------------|--------------|-------|----------|----------|
| 1          | Widget       | 19.99 | 10.00    | Hardware |

**Explanation:** The CHECK constraints enforce that price is positive and greater than the discount.

**Real-World Cases:**

- **HR:** Salary must be positive; hire date cannot be in the future.
- **E-commerce:** Product price must be greater than cost.
- **Finance:** Account balance must not go below zero for certain account types.

**References:**

- Microsoft Learn: CHECK Constraints - https://learn.microsoft.com/en-us/sql/relational-databases/tables/unique-constraints-and-check-constraints
- MySQL: CHECK Constraints - https://dev.mysql.com/doc/refman/8.0/en/create-table-check-constraints.html
- PostgreSQL: CHECK Constraints - https://www.postgresql.org/docs/current/ddl-constraints.html


### 2. Default Values & Data Generation

#### 2.1 DEFAULT Constraints (Assigning Fallback Values)

**Core Definitions:**

- **Core Definition:** A DEFAULT constraint assigns a value to a column when no value is explicitly provided during an INSERT.
- **Technical Definition:** DEFAULT is a column-level specification that provides a fallback value. It can be a literal, a function call (e.g., `CURRENT_TIMESTAMP`), or an expression. The DEFAULT is applied only when the column is omitted from the INSERT statement or when the DEFAULT keyword is explicitly used.
- **Beginner-Friendly Explanation:** DEFAULT is like a pre-filled form field. If you don't write anything, the form already has a value filled in.

**Purposes:**

- To provide sensible default values for optional columns
- To automatically populate audit columns (timestamps, user IDs)
- To reduce the amount of data the application must supply
- To ensure columns have a value even when the application omits them

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- Column-level
column_name data_type DEFAULT default_value

-- Named DEFAULT constraint
column_name data_type CONSTRAINT constraint_name DEFAULT default_value
```

**Component Breakdown:**

- `default_value`: A literal, function, or expression

**Syntax Rules:**

- DEFAULT is applied only when the column is not included in the INSERT column list.
- DEFAULT can be used with `INSERT ... VALUES (DEFAULT, ...)`.
- DEFAULT constraints can be named using the CONSTRAINT keyword.
- DEFAULT does not apply to UPDATE statements (unless explicitly set to DEFAULT).

**Annotated Code Examples:**

```sql
-- Setup: Table with DEFAULT values
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'Pending',
    priority INT DEFAULT 1
);

-- Insert without specifying defaults
INSERT INTO orders (order_id) VALUES (1);
-- order_date, status, and priority get their defaults

-- Insert with explicit values
INSERT INTO orders (order_id, status) VALUES (2, 'Shipped');
-- order_date and priority get defaults, status is 'Shipped'
```

**Expected Output:**

| order_id | order_date          | status  | priority |
|----------|---------------------|---------|----------|
| 1        | 2026-09-19 10:30:00 | Pending | 1        |
| 2        | 2026-09-19 10:30:00 | Shipped | 1        |

**Real-World Cases:**

- **Audit columns:** `created_at DEFAULT CURRENT_TIMESTAMP`.
- **Status fields:** `status DEFAULT 'Active'`.
- **Counters:** `priority DEFAULT 1`.

**References:**

- Microsoft Learn: DEFAULT Constraints - https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-default-constraints
- PostgreSQL: DEFAULT - https://www.postgresql.org/docs/current/ddl-default.html
- MySQL: DEFAULT - https://dev.mysql.com/doc/refman/8.0/en/data-type-defaults.html


#### 2.2 Auto-incrementing / Identity Constraints (SERIAL, GENERATED ALWAYS AS IDENTITY)

**Core Definitions:**

- **Core Definition:** Auto-incrementing constraints automatically generate sequential numeric values for a column, typically used for primary keys.
- **Technical Definition:** Different databases implement auto-increment differently: PostgreSQL uses `SERIAL` (legacy) and `GENERATED ... AS IDENTITY` (standard), MySQL uses `AUTO_INCREMENT`, SQL Server uses `IDENTITY`, and Oracle uses `GENERATED ... AS IDENTITY`. The SQL standard identity column supports two modes: `GENERATED ALWAYS` (cannot insert explicit values) and `GENERATED BY DEFAULT` (allows explicit values, falling back to generation when omitted).
- **Beginner-Friendly Explanation:** Auto-increment is like a numbered ticket dispenser. Each time you insert a row, the database automatically assigns the next number. You don't have to specify the ID.

**Purposes:**

- To automatically generate unique primary key values
- To eliminate the need for application-managed sequences
- To ensure unique identifiers without manual intervention
- To support `GENERATED ALWAYS` for strict sequence management or `GENERATED BY DEFAULT` for flexibility

**Syntax Structures and Rules:**

**Complete General Syntax (ANSI SQL:2003 and later):**

```sql
-- GENERATED ALWAYS (strict)
column_name data_type GENERATED ALWAYS AS IDENTITY
    (START WITH n INCREMENT BY m)

-- GENERATED BY DEFAULT (flexible)
column_name data_type GENERATED BY DEFAULT AS IDENTITY
    (START WITH n INCREMENT BY m)
```

**Vendor-Specific Syntax:**

| Database | Syntax |
|----------|--------|
| PostgreSQL | `SERIAL` (legacy) or `GENERATED ... AS IDENTITY` |
| MySQL | `AUTO_INCREMENT` |
| SQL Server | `IDENTITY(seed, increment)` |
| Oracle | `GENERATED ... AS IDENTITY` |

**Syntax Rules:**

- `GENERATED ALWAYS` columns reject explicit values (must use DEFAULT or omit the column).
- `GENERATED BY DEFAULT` columns accept explicit values; the sequence is used when no value is supplied.
- Auto-increment columns are typically integers (INT, BIGINT).
- The sequence does not check for duplicates in `GENERATED BY DEFAULT` mode.

**Annotated Code Examples:**

```sql
-- PostgreSQL: GENERATED ALWAYS AS IDENTITY
CREATE TABLE employees (
    emp_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    emp_name VARCHAR(100) NOT NULL
);

INSERT INTO employees (emp_name) VALUES ('Alice');
-- emp_id is automatically 1

INSERT INTO employees (emp_id, emp_name) VALUES (100, 'Bob');
-- Error: cannot insert into GENERATED ALWAYS identity column

-- PostgreSQL: GENERATED BY DEFAULT
CREATE TABLE departments (
    dept_id INT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    dept_name VARCHAR(100)
);

INSERT INTO departments (dept_name) VALUES ('Engineering');
-- dept_id is automatically 1

INSERT INTO departments (dept_id, dept_name) VALUES (100, 'Special');
-- Explicit value 100 is accepted
```

**Expected Output (GENERATED ALWAYS):**

| emp_id | emp_name |
|--------|----------|
| 1      | Alice    |

**Expected Output (GENERATED BY DEFAULT):**

| dept_id | dept_name   |
|---------|-------------|
| 1       | Engineering |
| 100     | Special     |

**Real-World Cases:**

- **All tables:** Auto-incrementing primary keys are the standard for surrogate keys.
- **Order numbers:** Sequential order IDs.

**References:**

- PostgreSQL: Identity Columns - https://www.postgresql.org/docs/current/ddl-identity-columns.html
- Microsoft Learn: IDENTITY - https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql


### 3. Syntax & Structural Scoping

#### 3.1 Column-Level vs. Table-Level Constraint Definitions

**Core Definitions:**

- **Core Definition:** Column-level constraints are defined as part of the column definition; table-level constraints are defined after all column definitions.
- **Technical Definition:** Column-level constraints apply to a single column and are declared immediately after the column's data type. Table-level constraints can apply to one or more columns and are declared after all columns are defined. A key constraint involves comparing different rows within the same table and is therefore always a table-level constraint.
- **Beginner-Friendly Explanation:** Column-level is like writing the rule next to the column it applies to. Table-level is like writing all the rules at the bottom of the form.

**Purposes:**

- To choose the appropriate syntax based on constraint scope
- To define composite constraints that span multiple columns
- To explicitly name constraints for manageability

**Syntax Structures and Rules:**

**Column-Level Syntax:**

```sql
CREATE TABLE table_name (
    column1 data_type [CONSTRAINT name] constraint_type,
    column2 data_type [CONSTRAINT name] constraint_type
);
```

**Table-Level Syntax:**

```sql
CREATE TABLE table_name (
    column1 data_type,
    column2 data_type,
    [CONSTRAINT name] constraint_type (column1, column2, ...)
);
```

**Syntax Rules:**

- NOT NULL can only be defined at the column level.
- Composite PRIMARY KEY and UNIQUE constraints must be defined at the table level.
- CHECK constraints can be defined at either level.
- FOREIGN KEY constraints can be defined at either level.
- DEFAULT can be defined at either level (though column-level is more common).

**Annotated Code Examples:**

```sql
-- Column-level constraints
CREATE TABLE employees (
    emp_id INT CONSTRAINT pk_emp PRIMARY KEY,
    emp_name VARCHAR(100) CONSTRAINT nn_emp_name NOT NULL,
    email VARCHAR(200) CONSTRAINT uq_emp_email UNIQUE,
    salary DECIMAL(10,2) CONSTRAINT ck_salary CHECK (salary > 0)
);

-- Table-level constraints (equivalent)
CREATE TABLE employees (
    emp_id INT,
    emp_name VARCHAR(100),
    email VARCHAR(200),
    salary DECIMAL(10,2),
    CONSTRAINT pk_emp PRIMARY KEY (emp_id),
    CONSTRAINT nn_emp_name NOT NULL (emp_name),  -- Not allowed! NOT NULL is column-level only
    CONSTRAINT uq_emp_email UNIQUE (email),
    CONSTRAINT ck_salary CHECK (salary > 0)
);

-- Composite PRIMARY KEY (must be table-level)
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT NOT NULL,
    PRIMARY KEY (order_id, product_id)
);
```

**Real-World Cases:**

- **Junction tables:** Composite PRIMARY KEY or UNIQUE constraints.
- **Single-column keys:** Column-level PRIMARY KEY for simplicity.
- **Cross-column validation:** Table-level CHECK constraints.

**References:**

- Stack Overflow: Column level vs table level constraints - https://stackoverflow.com/questions/9494129/column-level-vs-table-level-constraints-in-sql-server
- PostgreSQL: Table Constraints - https://www.postgresql.org/docs/current/ddl-constraints.html


#### 3.2 Single-Column Constraints vs. Composite (Multi-column) Constraints

**Core Definitions:**

- **Core Definition:** A single-column constraint applies to one column; a composite constraint applies to a combination of two or more columns.
- **Technical Definition:** Composite constraints (PRIMARY KEY, UNIQUE, FOREIGN KEY) enforce uniqueness or referential integrity across the combination of specified columns. For UNIQUE, the combination must be unique; individual columns may contain duplicates. For FOREIGN KEY, the combination of child columns must match a combination of parent columns.
- **Beginner-Friendly Explanation:** A composite key is like a combination lock—you need all the parts to match. Two rows can have the same first name and the same last name, but not the same first name AND last name.

**Purposes:**

- To enforce uniqueness across a natural key composed of multiple attributes
- To model many-to-many relationships with junction tables
- To support complex referential integrity requirements

**Syntax Structures and Rules:**

```sql
-- Composite PRIMARY KEY
PRIMARY KEY (column1, column2)

-- Composite UNIQUE
UNIQUE (column1, column2)

-- Composite FOREIGN KEY
FOREIGN KEY (column1, column2) REFERENCES parent(col1, col2)
```

**Syntax Rules:**

- Composite constraints must be defined at the table level.
- The order of columns in the constraint definition matters for FOREIGN KEY references.
- For UNIQUE, the combination of columns must be unique, but each individual column can have duplicates.

**Annotated Code Examples:**

```sql
-- Composite UNIQUE constraint
CREATE TABLE course_enrollments (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    grade CHAR(2),
    UNIQUE (student_id, course_id)
);
-- A student can enroll in a course only once, but can enroll in multiple courses
-- and a course can have multiple students

-- Composite FOREIGN KEY
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

**Expected Output (Composite UNIQUE):** Attempting to enroll the same student in the same course twice fails.

**Real-World Cases:**

- **Junction tables:** Composite PRIMARY KEY on (order_id, product_id).
- **Enrollment systems:** UNIQUE on (student_id, course_id).
- **Multi-tenant systems:** UNIQUE on (tenant_id, username).

**References:**

- Microsoft Learn: Composite Keys - https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-primary-keys
- PostgreSQL: Composite Constraints - https://www.postgresql.org/docs/current/ddl-constraints.html


### 4. Naming & Metadata Management

#### 4.1 Explicit Constraint Naming Conventions

**Core Definitions:**

- **Core Definition:** Explicit constraint naming assigns a user-defined name to a constraint using the `CONSTRAINT constraint_name` syntax.
- **Technical Definition:** Constraint names must follow identifier rules and be unique within the schema. Explicit names make constraints easier to reference in DDL statements (e.g., `ALTER TABLE ... DROP CONSTRAINT name`) and appear in error messages and the information schema. Common naming conventions use prefixes: `PK_` for primary key, `FK_` for foreign key, `UQ_` for unique, `CK_` for check, `DF_` for default.
- **Beginner-Friendly Explanation:** Naming your constraints is like labeling your folders. Instead of searching through system-generated names, you can find exactly what you need.

**Purposes:**

- To make constraint management easier (dropping, disabling, modifying)
- To improve error message clarity
- To enable consistent naming across development, staging, and production
- To support migration scripts that reference constraints by name

**Syntax Structures and Rules:**

```sql
-- Named constraint in CREATE TABLE
CREATE TABLE table_name (
    column1 data_type CONSTRAINT constraint_name constraint_type,
    ...
    CONSTRAINT table_constraint_name constraint_type (column_list)
);

-- Named constraint in ALTER TABLE
ALTER TABLE table_name ADD CONSTRAINT constraint_name constraint_type (...);
```

**Naming Convention Examples:**

| Constraint Type | Prefix | Example |
|----------------|--------|---------|
| PRIMARY KEY | `PK_` | `PK_Employees` |
| FOREIGN KEY | `FK_` | `FK_Orders_Customers` |
| UNIQUE | `UQ_` | `UQ_Users_Email` |
| CHECK | `CK_` | `CK_Products_Price` |
| DEFAULT | `DF_` | `DF_Orders_Status` |

**Annotated Code Examples:**

```sql
-- System-generated name (avoid)
CREATE TABLE SystemNamed (
    column1 INT DEFAULT 42
);
-- Constraint name might be: DF_SystemNam_colum_07020F21 (unfriendly)

-- Explicit name (recommended)
CREATE TABLE ProgrammerNamed (
    column1 INT CONSTRAINT DF_column1 DEFAULT 42
);
-- Constraint name: DF_column1 (clear and manageable)

-- Named foreign key
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    CONSTRAINT FK_Orders_Customers
        FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

**Real-World Cases:**

- **Migration scripts:** Referencing constraints by name in ALTER TABLE statements.
- **Troubleshooting:** Identifying which constraint was violated from error messages.
- **Code reviews:** Consistent naming makes schema definitions easier to review.

**References:**

- Microsoft Learn: Best Practice: Naming Constraints - https://learn.microsoft.com/en-us/archive/blogs/dtjones/best-practice-naming-constraints
- PostgreSQL: Constraint Naming - https://www.postgresql.org/docs/current/ddl-constraints.html


#### 4.2 System-Generated (Implicit) Names and Migration Challenges

**Core Definitions:**

- **Core Definition:** When a constraint is created without an explicit name, the database assigns a system-generated name.
- **Technical Definition:** System-generated names vary by database: SQL Server uses formats like `DF_SystemNam_colum_07020F21`, PostgreSQL uses `tablename_columnname_key`, MySQL uses `tablename_ibfk_N`. These names are non-intuitive and can differ across environments, making migrations and schema comparisons difficult.
- **Beginner-Friendly Explanation:** System-generated names are like automatically generated filenames—they work, but they're hard to remember and change from system to system.

**Purposes:**

- To understand why explicit naming is a best practice
- To troubleshoot migration failures caused by name mismatches
- To write database-agnostic migration scripts

**Annotated Code Examples:**

```sql
-- System-generated names (example from SQL Server)
CREATE TABLE dbo.Example (
    column1 INT DEFAULT 42
);
-- The constraint name might be: DF_Example_colum_07020F21

-- Querying system-generated names (SQL Server)
SELECT name, type_desc
FROM sys.objects
WHERE parent_object_id = OBJECT_ID('dbo.Example');
```

**Real-World Cases:**

- **Schema comparison tools:** System-generated names cause false differences between environments.
- **CI/CD pipelines:** Migration scripts fail when constraint names differ.

**References:**

- Devart: DF038: The constraint name is not explicitly specified - https://docs.devart.com
- SQL Enlight: SA0075B: Avoid adding constraints with default system generated name - https://docs.sqlenlight.com


#### 4.3 Querying Database Information Schemas to Inspect Constraints

**Core Definitions:**

- **Core Definition:** The INFORMATION_SCHEMA is a set of views (standardized in SQL:2003) that provide metadata about database objects, including constraints.
- **Technical Definition:** The `INFORMATION_SCHEMA.TABLE_CONSTRAINTS` view lists all constraints, while `INFORMATION_SCHEMA.CHECK_CONSTRAINTS`, `INFORMATION_SCHEMA.KEY_COLUMN_USAGE`, and `INFORMATION_SCHEMA.REFERENTIAL_CONSTRAINTS` provide additional details. Vendor-specific catalog views (e.g., `sys.objects` in SQL Server, `pg_constraint` in PostgreSQL) offer more detailed information.
- **Beginner-Friendly Explanation:** The information schema is like a card catalog in a library—it tells you what constraints exist and where they are, without having to look at the tables themselves.

**Syntax Structures and Rules:**

```sql
-- List all constraints for a table (standard)
SELECT constraint_name, constraint_type
FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS
WHERE table_name = 'employees';

-- List check constraints
SELECT constraint_name, check_clause
FROM INFORMATION_SCHEMA.CHECK_CONSTRAINTS
WHERE constraint_schema = 'public';

-- List foreign key details
SELECT
    tc.constraint_name,
    kcu.column_name,
    ccu.table_name AS referenced_table,
    ccu.column_name AS referenced_column
FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS tc
JOIN INFORMATION_SCHEMA.KEY_COLUMN_USAGE kcu
    ON tc.constraint_name = kcu.constraint_name
JOIN INFORMATION_SCHEMA.CONSTRAINT_COLUMN_USAGE ccu
    ON tc.constraint_name = ccu.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY';
```

**Annotated Code Examples:**

```sql
-- PostgreSQL: List all constraints on a table
SELECT conname, contype, pg_get_constraintdef(oid) AS definition
FROM pg_constraint
WHERE conrelid = 'employees'::regclass;

-- SQL Server: List all constraints on a table
SELECT name, type_desc, OBJECT_DEFINITION(object_id) AS definition
FROM sys.objects
WHERE parent_object_id = OBJECT_ID('dbo.Employees')
  AND type IN ('PK', 'FK', 'UQ', 'C', 'D');
```

**Expected Output (PostgreSQL):**

| conname | contype | definition |
|---------|---------|------------|
| employees_pkey | p | PRIMARY KEY (emp_id) |
| employees_email_key | u | UNIQUE (email) |
| employees_salary_check | c | CHECK (salary > 0) |

**Real-World Cases:**

- **Documentation:** Generating schema documentation automatically.
- **Impact analysis:** Understanding what constraints exist before making schema changes.
- **Auditing:** Verifying that constraints are correctly defined in production.

**References:**

- Oracle: INFORMATION_SCHEMA TABLE_CONSTRAINTS - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/INFORMATION_SCHEMA-TABLE_CONSTRAINTS.html
- Microsoft Learn: TABLE_CONSTRAINTS - https://learn.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/table-constraints-transact-sql


### 5. Enforcement Mechanics & Lifecycle

#### 5.1 Constraint Enforcement Timing (Immediate vs. Deferred)

**Core Definitions:**

- **Core Definition:** Constraint enforcement timing determines when constraints are checked: immediately after each statement (default) or deferred until the end of the transaction.
- **Technical Definition:** Constraints can be `DEFERRABLE` or `NOT DEFERRABLE`. A `DEFERRABLE` constraint can be set to `INITIALLY IMMEDIATE` (checked after each statement, the default) or `INITIALLY DEFERRED` (checked only at transaction commit). The timing can be changed within a transaction using `SET CONSTRAINTS`. Only UNIQUE, PRIMARY KEY, EXCLUDE, and FOREIGN KEY constraints can be deferred; NOT NULL and CHECK constraints cannot.
- **Beginner-Friendly Explanation:** Normally, the database checks the rules after every single change. With deferred constraints, you can temporarily break the rules as long as everything is correct by the time you save your work.

**Purposes:**

- To allow temporary constraint violations during complex multi-statement operations
- To support circular foreign key references
- To enable bulk data loading with temporary inconsistencies
- To control when constraint checks occur for performance reasons

**Syntax Structures and Rules:**

```sql
-- In CREATE TABLE or ALTER TABLE
CONSTRAINT constraint_name constraint_type
    [DEFERRABLE | NOT DEFERRABLE]
    [INITIALLY IMMEDIATE | INITIALLY DEFERRED]

-- Change timing within a transaction
SET CONSTRAINTS { ALL | constraint_name [, ...] } { DEFERRED | IMMEDIATE };
```

**Syntax Rules:**

- `NOT DEFERRABLE` is the default for all constraints.
- `INITIALLY IMMEDIATE` is the default for deferrable constraints.
- Deferred constraints are checked at transaction commit.
- `SET CONSTRAINTS` can change the timing within a transaction.
- Not all databases support deferrable constraints (e.g., MySQL does not support DEFERRABLE).

**Annotated Code Examples:**

```sql
-- PostgreSQL: Deferrable FOREIGN KEY
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100),
    dept_id INT,
    CONSTRAINT fk_emp_dept FOREIGN KEY (dept_id)
        REFERENCES departments(dept_id)
        DEFERRABLE INITIALLY DEFERRED
);

BEGIN;
-- Insert a department and an employee in any order
INSERT INTO employees VALUES (1, 'Alice', 10);  -- dept 10 doesn't exist yet
INSERT INTO departments VALUES (10, 'Engineering');  -- now it exists
COMMIT;  -- constraint checked here, passes

-- Change constraint timing within a transaction
BEGIN;
SET CONSTRAINTS fk_emp_dept IMMEDIATE;
-- Now the constraint is checked after each statement
```

**Real-World Cases:**

- **Circular references:** Two tables referencing each other.
- **Bulk loading:** Loading data in a non-optimal order.
- **Data migration:** Temporarily allowing inconsistencies during transformation.

**References:**

- PostgreSQL: Deferrable Constraints - https://www.postgresql.org/docs/current/sql-createtable.html
- Oracle: Deferrable Constraints - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/constraint.html


#### 5.2 Handling Old Data When Adding New Constraints (WITH CHECK vs. WITH NOCHECK)

**Core Definitions:**

- **Core Definition:** When adding a constraint to an existing table with data, you can validate existing rows (`WITH CHECK`) or skip validation (`WITH NOCHECK`).
- **Technical Definition:** `WITH CHECK` (the default for new constraints) validates that all existing rows satisfy the constraint before adding it. `WITH NOCHECK` adds the constraint without validating existing data. Constraints added with `WITH NOCHECK` are not considered by the query optimizer until re-enabled with `WITH CHECK CHECK CONSTRAINT`.
- **Beginner-Friendly Explanation:** WITH CHECK means "make sure all the old data is also valid." WITH NOCHECK means "I promise the old data is fine, just add the rule for future data."

**Purposes:**

- To add constraints to tables with existing data
- To control the performance impact of constraint validation
- To temporarily add constraints without full validation
- To document intended constraints even when data doesn't yet comply

**Syntax Structures and Rules:**

```sql
-- SQL Server
ALTER TABLE table_name
WITH CHECK | WITH NOCHECK
ADD CONSTRAINT constraint_name constraint_type (...);

-- Re-enable a constraint with validation
ALTER TABLE table_name
WITH CHECK CHECK CONSTRAINT constraint_name;

-- PostgreSQL
ALTER TABLE table_name
ADD CONSTRAINT constraint_name constraint_type (...)
NOT VALID;

-- Validate later
ALTER TABLE table_name
VALIDATE CONSTRAINT constraint_name;
```

**Syntax Rules:**

- `WITH CHECK` is the default for new constraints.
- `WITH NOCHECK` is not recommended except in rare cases.
- Constraints added with `WITH NOCHECK` are not used by the query optimizer.
- Re-enabling with `WITH CHECK CHECK CONSTRAINT` validates existing data.

**Annotated Code Examples:**

```sql
-- SQL Server: Add constraint with validation
ALTER TABLE employees
WITH CHECK
ADD CONSTRAINT ck_salary CHECK (salary > 0);
-- Fails if any existing row has salary <= 0

-- SQL Server: Add constraint without validation
ALTER TABLE employees
WITH NOCHECK
ADD CONSTRAINT ck_salary CHECK (salary > 0);
-- Succeeds even if some rows violate the constraint

-- Validate the constraint later
ALTER TABLE employees
WITH CHECK CHECK CONSTRAINT ck_salary;
-- Now validates existing data; fails if violations exist
```

```sql
-- PostgreSQL: Add NOT VALID constraint
ALTER TABLE employees
ADD CONSTRAINT ck_salary CHECK (salary > 0) NOT VALID;

-- Validate later
ALTER TABLE employees
VALIDATE CONSTRAINT ck_salary;
```

**Real-World Cases:**

- **Legacy data migration:** Adding constraints to tables with historical data.
- **Performance:** Avoiding long validation scans on large tables.
- **Incremental validation:** Validating constraints during maintenance windows.

**References:**

- Microsoft Learn: WITH CHECK / WITH NOCHECK - https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-table-transact-sql
- PostgreSQL: NOT VALID - https://www.postgresql.org/docs/current/sql-altertable.html


#### 5.3 Dropping, Disabling, or Enabling Constraints

**Core Definitions:**

- **Core Definition:** Constraints can be dropped (removed permanently), disabled (temporarily not enforced), or enabled (re-activated).
- **Technical Definition:** `ALTER TABLE ... DROP CONSTRAINT` removes the constraint permanently. `ALTER TABLE ... DISABLE CONSTRAINT` (Oracle) or `ALTER TABLE ... NOCHECK CONSTRAINT` (SQL Server) disables enforcement. PostgreSQL uses `ALTER TABLE ... DROP CONSTRAINT` followed by re-adding. Disabling constraints can be useful for bulk operations.
- **Beginner-Friendly Explanation:** Dropping a constraint is like removing a rule from the rulebook. Disabling it is like temporarily suspending the rule.

**Purposes:**

- To remove obsolete constraints
- To temporarily disable constraints for bulk data loading
- To re-enable constraints after maintenance
- To modify constraint definitions (drop and recreate)

**Syntax Structures and Rules:**

```sql
-- Drop constraint
ALTER TABLE table_name DROP CONSTRAINT constraint_name;

-- Disable constraint (Oracle)
ALTER TABLE table_name DISABLE CONSTRAINT constraint_name;

-- Disable constraint (SQL Server)
ALTER TABLE table_name NOCHECK CONSTRAINT constraint_name;

-- Enable constraint (Oracle)
ALTER TABLE table_name ENABLE CONSTRAINT constraint_name;

-- Enable constraint (SQL Server)
ALTER TABLE table_name CHECK CONSTRAINT constraint_name;
```

**Syntax Rules:**

- Dropping a PRIMARY KEY constraint drops the associated index.
- Dropping a FOREIGN KEY constraint removes the referential integrity enforcement.
- Disabling a constraint does not remove it; it can be re-enabled.
- In PostgreSQL, disabling a constraint requires dropping it (or using `ALTER TABLE ... DROP CONSTRAINT`).

**Annotated Code Examples:**

```sql
-- Drop a constraint
ALTER TABLE employees DROP CONSTRAINT ck_salary;

-- SQL Server: Disable a constraint
ALTER TABLE employees NOCHECK CONSTRAINT ck_salary;

-- SQL Server: Re-enable with validation
ALTER TABLE employees CHECK CONSTRAINT ck_salary;

-- Oracle: Disable and enable
ALTER TABLE employees DISABLE CONSTRAINT ck_salary;
ALTER TABLE employees ENABLE CONSTRAINT ck_salary;

-- PostgreSQL: Drop and re-add (PostgreSQL doesn't support disabling)
ALTER TABLE employees DROP CONSTRAINT ck_salary;
ALTER TABLE employees ADD CONSTRAINT ck_salary CHECK (salary > 0);
```

**Real-World Cases:**

- **Bulk loading:** Disabling constraints before a large load, then re-enabling.
- **Schema evolution:** Dropping and recreating constraints with new definitions.
- **Troubleshooting:** Temporarily disabling a constraint to isolate an issue.

**References:**

- Oracle: Modifying, Renaming, or Dropping Existing Integrity Constraints - https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/managing-integrity-constraints.html
- Microsoft Learn: Disable/Enable Constraints - https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-table-transact-sql


### 6. Interactions & Dependencies

#### 6.1 Referential Integrity Actions for Foreign Keys (ON DELETE / ON UPDATE)

**Core Definitions:**

- **Core Definition:** ON DELETE and ON UPDATE clauses specify what action the database takes when a parent row is deleted or its key is updated.
- **Technical Definition:** The SQL standard defines five referential actions: `NO ACTION` (default), `RESTRICT`, `CASCADE`, `SET NULL`, and `SET DEFAULT`. `CASCADE` deletes or updates child rows; `SET NULL` sets foreign key columns to NULL; `SET DEFAULT` sets them to their default values; `RESTRICT` and `NO ACTION` prevent the operation.
- **Beginner-Friendly Explanation:** These actions are like instructions for what to do with the children when the parent is deleted. "Delete them too" (CASCADE), "just unlink them" (SET NULL), or "don't let me delete the parent" (RESTRICT).

**Purposes:**

- To automate the handling of related records
- To maintain referential integrity during parent modifications
- To implement business rules for data cleanup
- To prevent orphaned records

**Syntax Structures and Rules:**

```sql
FOREIGN KEY (column_list)
REFERENCES parent_table(parent_column_list)
[ON DELETE { NO ACTION | RESTRICT | CASCADE | SET NULL | SET DEFAULT }]
[ON UPDATE { NO ACTION | RESTRICT | CASCADE | SET NULL | SET DEFAULT }]
```

**Action Definitions:**

| Action | ON DELETE Behavior | ON UPDATE Behavior |
|--------|-------------------|-------------------|
| NO ACTION | Error, rollback | Error, rollback |
| RESTRICT | Error, rollback | Error, rollback |
| CASCADE | Delete child rows | Update child FK values |
| SET NULL | Set child FK to NULL | Set child FK to NULL |
| SET DEFAULT | Set child FK to default | Set child FK to default |

**Syntax Rules:**

- `SET NULL` requires the FK columns to be nullable.
- `SET DEFAULT` requires default values defined for the FK columns.
- `RESTRICT` and `NO ACTION` are functionally similar (both prevent the operation).
- If no action is specified, the default is `NO ACTION`.

**Annotated Code Examples:**

```sql
-- Setup: Parent and child tables with CASCADE
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(100)
);

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100),
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

INSERT INTO departments VALUES (1, 'Engineering');
INSERT INTO employees VALUES (1, 'Alice', 1);
INSERT INTO employees VALUES (2, 'Bob', 1);

-- Deleting department 1 cascades to employees
DELETE FROM departments WHERE dept_id = 1;
-- Alice and Bob are automatically deleted
```

**Expected Output (after CASCADE delete):**

| emp_id  | emp_name |
|---------|----------|
| (empty) | (empty)  |

```sql
-- SET NULL example
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100),
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
        ON DELETE SET NULL
);

DELETE FROM departments WHERE dept_id = 1;
-- Employees in dept 1 now have dept_id = NULL
```

**Real-World Cases:**

- **E-commerce:** Deleting a customer cascades to their orders (CASCADE) or sets order customer to NULL (SET NULL).
- **HR:** Deleting a department cascades to employees (CASCADE) or sets their department to NULL (SET NULL).
- **CMS:** Deleting a category prevents deletion if articles exist (RESTRICT).

**References:**

- MySQL: FOREIGN KEY Constraints (Reference Options) - https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html
- Microsoft Learn: ON DELETE/ON UPDATE - https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql


#### 6.2 The NULL Behavior Trap: How NULL Values Bypass Constraints

**Core Definitions:**

- **Core Definition:** NULL values bypass CHECK constraints and have special behavior in UNIQUE constraints, because NULL comparisons evaluate to UNKNOWN, not FALSE.
- **Technical Definition:** For CHECK constraints, a condition that evaluates to UNKNOWN passes the constraint. For UNIQUE constraints, the treatment of multiple NULLs varies: PostgreSQL, MySQL, and Oracle allow multiple NULLs, while SQL Server allows only one NULL per column. This is because the SQL standard generally treats NULLs as an "escape hatch" for constraints.
- **Beginner-Friendly Explanation:** NULL means "unknown," so the database can't say whether the rule is violated or not. Therefore, it allows the value. This can be surprising—for example, a UNIQUE constraint on email might allow multiple rows with NULL email.

**Purposes:**

- To understand why NULL values may bypass constraints
- To design constraints that handle NULLs correctly
- To avoid unexpected behavior in data validation
- To choose appropriate NULL handling strategies per database

**CHECK Constraint NULL Behavior:**

```sql
-- A CHECK constraint passes if it evaluates to TRUE or UNKNOWN
CREATE TABLE products (
    price DECIMAL(10,2) CHECK (price > 0)
);

INSERT INTO products VALUES (10.00);  -- TRUE, passes
INSERT INTO products VALUES (NULL);    -- UNKNOWN, passes (NULL allowed)
INSERT INTO products VALUES (-5.00);   -- FALSE, rejected
```

**UNIQUE Constraint NULL Behavior:**

| Database | Multiple NULLs Allowed in UNIQUE Column? |
|----------|------------------------------------------|
| PostgreSQL | Yes (NULLs are distinct) |
| MySQL | Yes |
| SQLite | Yes |
| SQL Server | No (only one NULL allowed) |
| Oracle | Yes |

```sql
-- PostgreSQL: Multiple NULLs allowed in UNIQUE column
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    email VARCHAR(200) UNIQUE
);

INSERT INTO users VALUES (1, NULL);  -- Allowed
INSERT INTO users VALUES (2, NULL);  -- Allowed (second NULL)
-- In SQL Server, the second insert would fail
```

**Workarounds for SQL Server (only one NULL allowed):**

```sql
-- Use a filtered unique index to allow multiple NULLs
CREATE UNIQUE INDEX uq_users_email
ON users(email)
WHERE email IS NOT NULL;
```

**Annotated Code Examples:**

```sql
-- Demonstrate CHECK bypass with NULL
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100),
    age INT CHECK (age >= 18)
);

INSERT INTO employees VALUES (1, 'Alice', 25);   -- TRUE, passes
INSERT INTO employees VALUES (2, 'Bob', NULL);   -- UNKNOWN, passes
INSERT INTO employees VALUES (3, 'Charlie', 16); -- FALSE, rejected
```

**Expected Output:**

| emp_id | emp_name | age |
|--------|----------|-----|
| 1      | Alice    | 25  |
| 2      | Bob      | NULL |

**Explanation:** Bob's NULL age passes the CHECK constraint because `NULL >= 18` evaluates to UNKNOWN, not FALSE. Only Charlie's age (16) is rejected.

**Real-World Cases:**

- **Validation:** CHECK constraints on optional columns may allow NULL even when the validation rule seems strict.
- **Unique keys:** Multiple NULLs in a UNIQUE column can lead to unexpected duplicates.
- **Data quality:** NULLs can mask constraint violations.

**References:**

- PostgreSQL: CHECK Constraints and NULL - https://www.postgresql.org/message-id/87r7kgpuq8.fsf%40stark.xeocode.com
- Microsoft Learn: UNIQUE Constraints and NULL - https://learn.microsoft.com/en-us/sql/relational-databases/tables/unique-constraints-and-check-constraints
- MySQL: UNIQUE Constraints and NULL - https://dev.mysql.com/doc/refman/8.0/en/create-table.html


## Summary Tables

### Constraint Types Comparison

| Constraint | Level | NULL Behavior | Deferrable | Multiple per Table |
|------------|-------|--------------|------------|-------------------|
| NOT NULL | Column only | Rejects NULL | No | Yes (per column) |
| UNIQUE | Column or Table | Allows NULLs (varies) | Yes (PG, Oracle) | Yes |
| PRIMARY KEY | Column or Table | Rejects NULL | Yes (PG, Oracle) | No (one per table) |
| FOREIGN KEY | Column or Table | Allows NULL | Yes (PG, Oracle) | Yes |
| CHECK | Column or Table | UNKNOWN passes | No | Yes |
| DEFAULT | Column or Table | N/A | N/A | Yes (per column) |

### Referential Actions Summary

| Action | ON DELETE | ON UPDATE |
|--------|-----------|-----------|
| NO ACTION | Prevent, error | Prevent, error |
| RESTRICT | Prevent, error | Prevent, error |
| CASCADE | Delete child rows | Update child FK values |
| SET NULL | Set child FK to NULL | Set child FK to NULL |
| SET DEFAULT | Set child FK to default | Set child FK to default |

### Constraint Naming Prefixes (Convention)

| Constraint Type | Prefix | Example |
|----------------|--------|---------|
| PRIMARY KEY | PK_ | PK_Employees |
| FOREIGN KEY | FK_ | FK_Orders_Customers |
| UNIQUE | UQ_ | UQ_Users_Email |
| CHECK | CK_ | CK_Products_Price |
| DEFAULT | DF_ | DF_Orders_Status |

---

## References

- Microsoft Learn: Constraints - https://learn.microsoft.com/en-us/sql/relational-databases/tables/unique-constraints-and-check-constraints
- Microsoft Learn: Best Practice: Naming Constraints - https://learn.microsoft.com/en-us/archive/blogs/dtjones/best-practice-naming-constraints
- Microsoft Learn: FOREIGN KEY Constraints - https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-foreign-key-relationships
- Microsoft Learn: WITH CHECK / WITH NOCHECK - https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-table-transact-sql
- PostgreSQL: Constraints - https://www.postgresql.org/docs/current/ddl-constraints.html
- PostgreSQL: Deferrable Constraints - https://www.postgresql.org/docs/current/sql-createtable.html
- PostgreSQL: Identity Columns - https://www.postgresql.org/docs/current/ddl-identity-columns.html
- MySQL: FOREIGN KEY Constraints - https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html
- MySQL: CHECK Constraints - https://dev.mysql.com/doc/refman/8.0/en/create-table-check-constraints.html
- Oracle: Modifying, Renaming, or Dropping Existing Integrity Constraints - https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/managing-integrity-constraints.html
- Oracle: INFORMATION_SCHEMA TABLE_CONSTRAINTS - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/INFORMATION_SCHEMA-TABLE_CONSTRAINTS.html
- Stack Overflow: Column level vs table level constraints - https://stackoverflow.com/questions/9494129/column-level-vs-table-level-constraints-in-sql-server
- PostgreSQL: CHECK Constraints and NULL - https://www.postgresql.org/message-id/87r7kgpuq8.fsf%40stark.xeocode.com