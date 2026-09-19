# SQL UPDATE Operations: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The SQL UPDATE statement is a Data Manipulation Language (DML) command that modifies existing data in one or more rows of a table or view.

**Technical Definition:** Per the SQL standard (ISO/IEC 9075), UPDATE is a DML statement that changes the values of specified columns in all rows that satisfy a given condition. Only the columns to be modified need be mentioned in the SET clause; columns not explicitly modified retain their previous values. The statement supports optional WHERE, FROM, and RETURNING clauses, as well as vendor-specific extensions such as LIMIT and OUTPUT.

**Beginner-Friendly Explanation:** Think of a database table as a spreadsheet. UPDATE is the command that lets you change the contents of one or more cells (columns) in one or more rows. You tell the database which table to change, which columns to update, what new values to put in them, and which rows to change.

### Key Characteristics

- **Targeted modification:** Updates can affect a single row, multiple rows, or all rows depending on the WHERE clause.
- **Column-level precision:** Only the columns listed in the SET clause are modified; all other columns retain their values.
- **Expression support:** New values can be literals, expressions, self-references, subqueries, or DEFAULT/NULL.
- **Conditional execution:** The WHERE clause determines which rows are updated; without it, all rows are affected.
- **Atomic operation:** In most databases, an UPDATE statement either succeeds entirely or fails entirely (transactional).
- **Vendor extensions:** Different databases support additional features like LIMIT (MySQL), OUTPUT (SQL Server), and RETURNING (PostgreSQL, Oracle).

### Prerequisites

- Basic SQL syntax (SELECT, FROM, WHERE)
- Understanding of tables, columns, and data types
- Familiarity with NULL handling (see SQL NULL Handling cheat sheet)
- Basic knowledge of constraints (PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK)
- Transaction concepts (COMMIT, ROLLBACK)

### Related Programming Areas

- **Database Administration:** UPDATE operations affect table metadata, indexes, and statistics.
- **Application Development:** Most applications perform UPDATE operations through ORMs or parameterized queries.
- **Data Warehousing and ETL:** Bulk updates are common in data transformation pipelines.
- **Auditing and Compliance:** Tracking modifications through triggers and audit tables.
- **Concurrency Control:** UPDATE statements interact with locking mechanisms and isolation levels.
- **Performance Tuning:** UPDATE performance depends on indexes, constraints, and triggers.

---

## Core Concepts / Key Features

### 1. Core Update Syntax

#### 1.1 UPDATE Basic Syntax and Target Tables

**Core Definitions:**

- **Core Definition:** The basic UPDATE statement identifies a target table, specifies column-value assignments in a SET clause, and optionally filters rows with a WHERE clause.
- **Technical Definition:** The ANSI SQL standard defines the UPDATE statement with the syntax `UPDATE table_name SET column = value [, ...] [WHERE condition]`. The target table can be a base table, a view (with restrictions), or a table variable (in some databases).
- **Beginner-Friendly Explanation:** The UPDATE statement is like telling a spreadsheet: "In this sheet (table), change these cells (columns) to these new values, but only for the rows that match this condition."

**Purposes:**

- To modify existing data in a table without deleting and reinserting rows
- To apply corrections or updates to stored information
- To synchronize data across tables through joins or subqueries
- To implement business logic changes (e.g., price adjustments, status updates)

**Syntax Structures and Rules:**

**Complete General Syntax (ANSI SQL):**

```sql
UPDATE table_name
SET column_name = value [, column_name2 = value2, ...]
[WHERE condition];
```

**Vendor-Specific Syntax Extensions:**

**PostgreSQL:**

```sql
[ WITH [ RECURSIVE ] with_query [, ...] ]
UPDATE [ ONLY ] table_name [ * ] [ [ AS ] alias ]
SET { column_name = { expression | DEFAULT } |
      ( column_name [, ...] ) = [ ROW ] ( { expression | DEFAULT } [, ...] ) |
      ( column_name [, ...] ) = ( sub-SELECT ) } [, ...]
[ FROM from_item [, ...] ]
[ WHERE condition | WHERE CURRENT OF cursor_name ]
[ RETURNING { * | output_expression [ [ AS ] output_name ] } [, ...] ]
```


**SQL Server:**

```sql
[ WITH <common_table_expression> [...n] ]
UPDATE [ TOP ( expression ) [ PERCENT ] ]
{ { table_alias | <object> | rowset_function_limited } | @table_variable }
SET { column_name = { expression | DEFAULT | NULL } | ... }
[ <OUTPUT Clause> ]
[ FROM { <table_source> } [ ,...n ] ]
[ WHERE { <search_condition> | { [ CURRENT OF { cursor_name } ] } } ]
[ OPTION ( <query_hint> [ ,...n ] ) ]
```


**MySQL:**

```sql
UPDATE [LOW_PRIORITY] [IGNORE] table_reference
SET assignment_list
[WHERE where_condition]
[ORDER BY ...]
[LIMIT row_count]
```


**Oracle:**

```sql
UPDATE table_name [alias]
SET column_name = value [, column_name2 = value2, ...]
[WHERE condition]
[RETURNING expression INTO variable [, ...]];
```

**Component Breakdown:**

- `table_name`: The name of the table or view to update. May be schema-qualified.
- `SET`: Keyword introducing column-value assignments.
- `column_name`: The column to modify. Cannot be qualified with the table name in standard SQL.
- `value`: The new value—can be a literal, expression, DEFAULT, NULL, or subquery.
- `WHERE`: Optional clause that filters which rows are updated. Without it, all rows are updated.
- `FROM` (PostgreSQL, SQL Server): Allows additional tables to be referenced in the WHERE condition and update expressions.
- `RETURNING` (PostgreSQL, Oracle): Returns values from the updated rows.
- `OUTPUT` (SQL Server): Returns values from the updated rows.
- `LIMIT` (MySQL): Restricts the number of rows updated.
- `ORDER BY` (MySQL): Determines the order in which rows are updated.

**Syntax Rules:**

- Only the columns to be modified need be mentioned in the SET clause.
- Column names in the SET clause cannot be qualified with the table name in standard SQL (e.g., `UPDATE t SET t.col = 1` is invalid in PostgreSQL).
- The WHERE clause is optional; without it, all rows in the target table are updated.
- Views can be updated only if they are simple enough (typically based on a single table without aggregates or DISTINCT).

**Constraints and Limitations:**

- PRIMARY KEY columns cannot be updated to duplicate values.
- FOREIGN KEY constraints may prevent updates that would violate referential integrity.
- CHECK constraints must be satisfied by the new values.
- NOT NULL columns cannot be set to NULL.
- Updating a view may be restricted depending on the database and view definition.

**Annotated Code Examples:**

```sql
-- Setup: Create a sample table
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2),
    category VARCHAR(50),
    last_updated TIMESTAMP
);

INSERT INTO products VALUES (1, 'Widget', 19.99, 'Hardware', '2024-01-01');
INSERT INTO products VALUES (2, 'Gadget', 29.99, 'Electronics', '2024-01-01');
INSERT INTO products VALUES (3, 'Gizmo', 9.99, 'Hardware', '2024-01-01');
INSERT INTO products VALUES (4, 'Doohickey', 49.99, 'Electronics', '2024-01-01');

-- Example 1: Update a single column for a specific row
UPDATE products
SET price = 24.99
WHERE product_id = 1;

-- Verify the change
SELECT * FROM products WHERE product_id = 1;
```

**Expected Output:**

| product_id | product_name | price | category | last_updated |
|------------|--------------|-------|----------|--------------|
| 1          | Widget       | 24.99 | Hardware | 2024-01-01   |

**Explanation:** The WHERE clause targets only the row where `product_id = 1`. Only the `price` column is changed; all other columns retain their original values.

```sql
-- Example 2: Update all rows (dangerous without WHERE)
UPDATE products
SET last_updated = '2024-06-15';

-- Verify
SELECT * FROM products;
```

**Expected Output:**

| product_id | product_name | price | category    | last_updated |
|------------|--------------|-------|-------------|--------------|
| 1          | Widget       | 24.99 | Hardware    | 2024-06-15   |
| 2          | Gadget       | 29.99 | Electronics | 2024-06-15   |
| 3          | Gizmo        | 9.99  | Hardware    | 2024-06-15   |
| 4          | Doohickey    | 49.99 | Electronics | 2024-06-15   |

**Explanation:** Without a WHERE clause, every row in the table is updated. This is powerful but dangerous—always verify with a SELECT first.

**Real-World Cases:**

- **E-commerce:** Updating product prices during a sale event.
- **HR systems:** Updating employee contact information after a move.
- **Financial systems:** Applying interest calculations to account balances.
- **Inventory management:** Updating stock levels after a shipment arrives.

**References:**

- PostgreSQL: UPDATE - https://www.postgresql.org/docs/14/sql-update.html
- MySQL: UPDATE Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/update.html
- Microsoft Learn: UPDATE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/update-transact-sql


#### 1.2 Updating a Single Column vs. Multiple Columns

**Core Definitions:**

- **Core Definition:** A single-column UPDATE modifies one column, while a multi-column UPDATE modifies two or more columns in the same statement.
- **Technical Definition:** The SET clause accepts a comma-separated list of `column = value` assignments. Multi-column updates are atomic: all assignments are evaluated and applied together.
- **Beginner-Friendly Explanation:** You can change one cell at a time, or you can change several cells in the same row(s) all at once. Doing them together is more efficient and ensures consistency.

**Purposes:**

- To minimize the number of UPDATE statements executed
- To ensure multiple column changes are applied atomically
- To improve performance by reducing round trips to the database
- To maintain data consistency when related columns must change together

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- Single column
UPDATE table_name
SET column_name = value
[WHERE condition];

-- Multiple columns
UPDATE table_name
SET column_name1 = value1,
    column_name2 = value2,
    column_name3 = value3
[WHERE condition];
```

**Component Breakdown:**

- `column_name1, column_name2, ...`: The columns to modify
- `value1, value2, ...`: The corresponding new values
- The assignments are separated by commas

**Syntax Rules:**

- Assignments are evaluated from left to right in most databases.
- If a column is referenced in a later assignment, it uses the original (pre-update) value, not the newly assigned value (in standard SQL). However, MySQL evaluates assignments from left to right, so later assignments see earlier updates.
- Multi-column updates can use row constructors: `SET (col1, col2) = (val1, val2)` in PostgreSQL.

**Constraints and Limitations:**

- All assignments must be valid for their respective columns.
- The order of assignments can matter in MySQL but not in standard SQL.
- Some databases limit the number of columns that can be updated in a single statement (rare).

**Annotated Code Examples:**

```sql
-- Setup
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100),
    salary DECIMAL(10,2),
    bonus DECIMAL(10,2),
    department VARCHAR(50)
);

INSERT INTO employees VALUES (1, 'Alice', 80000, 5000, 'Engineering');
INSERT INTO employees VALUES (2, 'Bob', 60000, 3000, 'Marketing');
INSERT INTO employees VALUES (3, 'Charlie', 90000, 8000, 'Engineering');

-- Example 1: Update a single column
UPDATE employees
SET salary = 85000
WHERE emp_id = 1;

-- Example 2: Update multiple columns in one statement
UPDATE employees
SET salary = 95000,
    bonus = 10000,
    department = 'Senior Engineering'
WHERE emp_id = 3;

-- Verify
SELECT * FROM employees;
```

**Expected Output:**

| emp_id | emp_name | salary | bonus | department         |
|--------|----------|--------|-------|--------------------|
| 1      | Alice    | 85000  | 5000  | Engineering        |
| 2      | Bob      | 60000  | 3000  | Marketing          |
| 3      | Charlie  | 95000  | 10000 | Senior Engineering |

**Explanation:** Example 1 updates only Alice's salary. Example 2 updates Charlie's salary, bonus, and department atomically—all three changes are applied together.

```sql
-- Example 3: Multi-column update with expression referencing other columns
UPDATE employees
SET salary = salary * 1.10,
    bonus = bonus * 1.10
WHERE department = 'Engineering';

-- Verify
SELECT * FROM employees WHERE department = 'Engineering';
```

**Expected Output:**

| emp_id | emp_name | salary | bonus | department         |
|--------|----------|--------|-------|--------------------|
| 1      | Alice    | 93500  | 5500  | Engineering        |
| 3      | Charlie  | 104500 | 11000 | Senior Engineering |

**Explanation:** Both salary and bonus are increased by 10% for all Engineering employees. The expressions use the original values of the columns being updated.

**Real-World Cases:**

- **Payroll processing:** Updating salary, bonus, and tax code together.
- **Inventory:** Updating quantity, reorder level, and last restock date.
- **CRM:** Updating customer status, last contact date, and notes.
- **E-commerce:** Updating product price, sale price, and discount percentage.

**References:**

- Microsoft Learn: Update Data - https://learn.microsoft.com/en-us/training/modules/use-dml-statements/2-update-data
- Ask TOM: Performing Updates to Multiple Columns - https://asktom.oracle.com/pls/apex/asktom.search?tag=performing-updates-to-multiple-columns


#### 1.3 Updating a Single Row (Targeted Filtering)

**Core Definitions:**

- **Core Definition:** A single-row UPDATE modifies exactly one row in the target table, typically by filtering on the primary key or a unique constraint.
- **Technical Definition:** When the WHERE clause uses a condition that uniquely identifies a row (e.g., `WHERE primary_key = value`), the UPDATE affects at most one row.
- **Beginner-Friendly Explanation:** If you know exactly which row you want to change—for example, customer number 42—you use a WHERE clause that targets only that row.

**Purposes:**

- To make precise, targeted data corrections
- To update a specific record without affecting others
- To minimize the risk of unintended data changes
- To support application-level row-level operations

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
UPDATE table_name
SET column_name = value
WHERE unique_column = unique_value;
```

**Component Breakdown:**

- `unique_column`: A column with a PRIMARY KEY or UNIQUE constraint
- `unique_value`: A value that uniquely identifies one row

**Syntax Rules:**

- The WHERE clause must use a condition that uniquely identifies the row.
- Primary key or unique index columns are the safest choices.
- If multiple rows match the condition, multiple rows will be updated (not a single-row update).

**Constraints and Limitations:**

- If the unique value does not exist, no rows are updated (not an error).
- If the condition matches multiple rows (e.g., `WHERE category = 'Hardware'`), multiple rows are updated.

**Annotated Code Examples:**

```sql
-- Setup
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    email VARCHAR(200),
    phone VARCHAR(20)
);

INSERT INTO customers VALUES (1, 'Alice', 'alice@old.com', '555-0100');
INSERT INTO customers VALUES (2, 'Bob', 'bob@old.com', '555-0200');
INSERT INTO customers VALUES (3, 'Charlie', 'charlie@old.com', '555-0300');

-- Update a single row by primary key
UPDATE customers
SET email = 'alice@new.com'
WHERE customer_id = 1;

-- Verify
SELECT * FROM customers WHERE customer_id = 1;
```

**Expected Output:**

| customer_id | customer_name | email         | phone    |
|-------------|---------------|---------------|----------|
| 1           | Alice         | alice@new.com | 555-0100 |

**Explanation:** The WHERE clause uses the primary key (`customer_id = 1`), guaranteeing that only one row is updated.

**Real-World Cases:**

- **Customer support:** Updating a specific customer's contact information.
- **Order processing:** Marking a specific order as shipped.
- **User management:** Changing a specific user's password hash.
- **Inventory:** Recording a stock adjustment for a specific product.

**References:**

- SQL Server: UPDATE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/update-transact-sql
- PostgreSQL: UPDATE - https://www.postgresql.org/docs/14/sql-update.html


#### 1.4 Updating Multiple Rows (Bulk Updates via Broad Filtering)

**Core Definitions:**

- **Core Definition:** A multi-row UPDATE modifies all rows that satisfy a given condition, which can range from a few rows to every row in the table.
- **Technical Definition:** When the WHERE clause condition is non-unique (e.g., `WHERE department = 'Sales'`), the UPDATE affects all matching rows. Omitting the WHERE clause updates all rows in the table.
- **Beginner-Friendly Explanation:** If you want to change every product in the "Electronics" category, you use a WHERE clause that matches all of them. The database applies the change to every row that fits.

**Purposes:**

- To apply changes to groups of related records
- To perform bulk data corrections or migrations
- To implement business rules that affect many rows (e.g., annual price increases)
- To synchronize data across tables

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
UPDATE table_name
SET column_name = value
WHERE condition;  -- Condition may match multiple rows
```

**Component Breakdown:**

- `condition`: Any valid SQL condition that may be true for multiple rows
- If no WHERE clause is provided, all rows are updated

**Syntax Rules:**

- The WHERE clause is evaluated for each row; rows where the condition is TRUE are updated.
- Rows where the condition is FALSE or UNKNOWN (due to NULLs) are not updated.
- MySQL supports `LIMIT` to restrict the number of rows updated.
- MySQL supports `ORDER BY` to determine which rows are updated first.

**Constraints and Limitations:**

- Updating many rows can be slow and may lock the table for a long time.
- Constraint violations may cause the entire statement to fail (depending on the database).
- Triggers may fire for each row updated.

**Annotated Code Examples:**

```sql
-- Setup
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10,2),
    category VARCHAR(50)
);

INSERT INTO products VALUES (1, 'Widget', 19.99, 'Hardware');
INSERT INTO products VALUES (2, 'Gadget', 29.99, 'Electronics');
INSERT INTO products VALUES (3, 'Gizmo', 9.99, 'Hardware');
INSERT INTO products VALUES (4, 'Doohickey', 49.99, 'Electronics');
INSERT INTO products VALUES (5, 'Contraption', 39.99, 'Hardware');

-- Bulk update: 10% price increase for all Hardware products
UPDATE products
SET price = price * 1.10
WHERE category = 'Hardware';

-- Verify
SELECT * FROM products WHERE category = 'Hardware';
```

**Expected Output:**

| product_id | product_name | price | category |
|------------|--------------|-------|----------|
| 1          | Widget       | 21.99 | Hardware |
| 3          | Gizmo        | 10.99 | Hardware |
| 5          | Contraption  | 43.99 | Hardware |

**Explanation:** All rows where `category = 'Hardware'` are updated. The price is multiplied by 1.10 using the original price value.

```sql
-- MySQL: Bulk update with LIMIT
UPDATE products
SET price = price * 0.90
WHERE category = 'Electronics'
ORDER BY price DESC
LIMIT 1;
```

**Expected Output (MySQL):** The most expensive Electronics product's price is reduced by 10%.

**Real-World Cases:**

- **Retail:** Applying a seasonal discount to an entire category.
- **Finance:** Increasing interest rates for a specific account type.
- **HR:** Giving a standard raise to all employees in a department.
- **Logistics:** Marking all shipments in a region as delayed.

**References:**

- MySQL: UPDATE Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/update.html
- PostgreSQL: UPDATE - https://www.postgresql.org/docs/14/sql-update.html


### 2. Value Assignments

#### 2.1 Updating Using Expressions and Self-Referencing Operations

**Core Definitions:**

- **Core Definition:** The SET clause can use expressions that reference the current values of columns in the row being updated, enabling self-referencing updates like `SET price = price * 1.1`.
- **Technical Definition:** In standard SQL, expressions in the SET clause are evaluated using the original (pre-update) values of the columns. In MySQL, assignments are evaluated from left to right, so later assignments can reference newly assigned values.
- **Beginner-Friendly Explanation:** You can tell the database: "Take the current price, multiply it by 1.1, and store the result back in the price column." The database reads the old value, computes the new one, and writes it.

**Purposes:**

- To perform incremental updates based on existing data
- To calculate new values using formulas and functions
- To adjust values proportionally (e.g., percentage increases)
- To implement counters and running totals

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
UPDATE table_name
SET column_name = expression
WHERE condition;
```

**Component Breakdown:**

- `expression`: Any valid SQL expression that can reference columns, literals, functions, and operators
- The expression is evaluated for each row that matches the WHERE condition

**Syntax Rules:**

- In standard SQL, all references to columns in the SET clause use the original values.
- MySQL evaluates assignments from left to right; later assignments see earlier updates.
- Expressions can include arithmetic operators (+, -, *, /, %), string functions, date functions, and CASE expressions.

**Constraints and Limitations:**

- Division by zero in an expression may cause an error or return NULL (depending on the database).
- Data type conversions may be required if the expression result type differs from the column type.
- Self-referencing updates cannot reference columns from other rows without a subquery or join.

**Annotated Code Examples:**

```sql
-- Setup
CREATE TABLE accounts (
    account_id INT PRIMARY KEY,
    account_name VARCHAR(100),
    balance DECIMAL(12,2),
    interest_rate DECIMAL(5,4)
);

INSERT INTO accounts VALUES (1, 'Savings', 10000.00, 0.0250);
INSERT INTO accounts VALUES (2, 'Checking', 5000.00, 0.0100);
INSERT INTO accounts VALUES (3, 'Investment', 50000.00, 0.0500);

-- Example 1: Simple self-referencing update (add interest)
UPDATE accounts
SET balance = balance * (1 + interest_rate)
WHERE account_id = 1;

-- Verify
SELECT * FROM accounts WHERE account_id = 1;
```

**Expected Output:**

| account_id | account_name | balance  | interest_rate |
|------------|--------------|----------|---------------|
| 1          | Savings      | 10250.00 | 0.0250        |

**Explanation:** The expression `balance * (1 + interest_rate)` uses the original balance (10000.00) and interest rate (0.0250) to compute the new balance (10250.00).

```sql
-- Example 2: Counter update
UPDATE accounts
SET balance = balance - 500.00
WHERE account_id = 2;

-- Verify
SELECT * FROM accounts WHERE account_id = 2;
```

**Expected Output:**

| account_id | account_name | balance | interest_rate |
|------------|--------------|---------|---------------|
| 2          | Checking     | 4500.00 | 0.0100        |

**Explanation:** The balance is reduced by 500.00 using the original balance (5000.00).

```sql
-- Example 3: Multiple self-referencing updates (MySQL left-to-right evaluation)
-- In MySQL, this sets bonus to salary (old value), then salary to salary * 1.1
UPDATE employees
SET bonus = salary,
    salary = salary * 1.10
WHERE emp_id = 1;
```

**Real-World Cases:**

- **Banking:** Applying interest to account balances.
- **Retail:** Increasing prices by a percentage for inflation.
- **Gaming:** Incrementing player scores or experience points.
- **Inventory:** Adjusting stock levels based on sales or shipments.

**References:**

- PostgreSQL: UPDATE - https://www.postgresql.org/docs/14/sql-update.html
- MySQL: UPDATE Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/update.html


#### 2.2 Setting Columns to NULL or DEFAULT Values

**Core Definitions:**

- **Core Definition:** The SET clause can assign NULL to a column (if the column is nullable) or DEFAULT to reset a column to its default value.
- **Technical Definition:** Setting a column to NULL removes any value from that column, marking it as unknown or missing. Setting a column to DEFAULT assigns the column's default value as defined in the table schema (which is NULL if no default is specified).
- **Beginner-Friendly Explanation:** You can "clear" a column by setting it to NULL, or you can "reset" it to whatever the table considers the default value.

**Purposes:**

- To remove outdated or incorrect information
- To reset columns to their initial state
- To implement soft-delete patterns (setting a deleted_at timestamp or NULL)
- To clear optional fields

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- Set to NULL
UPDATE table_name
SET column_name = NULL
WHERE condition;

-- Set to DEFAULT
UPDATE table_name
SET column_name = DEFAULT
WHERE condition;
```

**Component Breakdown:**

- `NULL`: The special NULL marker (column must be nullable)
- `DEFAULT`: The column's default value (or NULL if no default exists)

**Syntax Rules:**

- Setting to NULL requires the column to be nullable (not constrained by NOT NULL or PRIMARY KEY).
- Setting to DEFAULT assigns the default value defined in the column's schema.
- If no default is defined, DEFAULT assigns NULL.

**Constraints and Limitations:**

- NOT NULL columns reject NULL assignments (error).
- PRIMARY KEY columns cannot be set to NULL.
- FOREIGN KEY columns can be set to NULL only if the column is nullable and the foreign key allows it.

**Annotated Code Examples:**

```sql
-- Setup
CREATE TABLE user_profiles (
    user_id INT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    bio TEXT,
    phone VARCHAR(20) DEFAULT 'Not provided',
    last_login TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO user_profiles (user_id, username, bio, phone)
VALUES (1, 'alice', 'Hello world', '555-0100');

INSERT INTO user_profiles (user_id, username, bio, phone)
VALUES (2, 'bob', 'SQL enthusiast', NULL);

-- Example 1: Set a column to NULL
UPDATE user_profiles
SET bio = NULL
WHERE user_id = 1;

-- Example 2: Set a column to DEFAULT
UPDATE user_profiles
SET phone = DEFAULT
WHERE user_id = 2;

-- Verify
SELECT * FROM user_profiles;
```

**Expected Output:**

| user_id | username | bio          | phone        | last_login          |
|---------|----------|--------------|--------------|---------------------|
| 1       | alice    | NULL         | 555-0100     | 2024-06-15 10:30:00 |
| 2       | bob      | SQL enthusiast | Not provided | 2024-06-15 10:30:00 |

**Explanation:** Alice's bio is cleared to NULL. Bob's phone is reset to the default value 'Not provided'.

**Real-World Cases:**

- **GDPR compliance:** Clearing personal data (setting to NULL) upon user request.
- **Soft deletes:** Setting a `deleted_at` timestamp to NULL when restoring a record.
- **User profiles:** Clearing optional fields like phone numbers or bios.
- **Configuration:** Resetting settings to their default values.

**References:**

- SQL Server: UPDATE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/update-transact-sql
- PostgreSQL: UPDATE - https://www.postgresql.org/docs/14/sql-update.html


#### 2.3 Updating from Related Data / Joins

**Core Definitions:**

- **Core Definition:** UPDATE statements can draw values from other tables using subqueries, FROM clauses (PostgreSQL, SQL Server), JOIN syntax (MySQL), or the MERGE statement (SQL Server, Oracle, PostgreSQL).
- **Technical Definition:** The SQL standard provides two mechanisms for updating from related data: sub-selects in the SET clause, and additional tables in the FROM clause. MySQL uses a multi-table UPDATE syntax with JOIN, while SQL Server and Oracle use MERGE for complex upsert operations.
- **Beginner-Friendly Explanation:** Sometimes the new value for a column comes from another table. For example, you might update a product's price based on the supplier's catalog. You can do this by joining the tables in the UPDATE statement.

**Purposes:**

- To synchronize data between related tables
- To populate columns with values calculated from other tables
- To implement complex business rules involving multiple tables
- To perform upsert (insert or update) operations with MERGE

**Syntax Structures and Rules:**

**PostgreSQL (UPDATE ... FROM):**

```sql
UPDATE target_table
SET target_column = source_table.column
FROM source_table
WHERE target_table.join_key = source_table.join_key
  AND condition;
```


**SQL Server (UPDATE ... FROM):**

```sql
UPDATE target_table
SET target_column = source_table.column
FROM target_table
JOIN source_table ON target_table.join_key = source_table.join_key
WHERE condition;
```


**MySQL (UPDATE ... JOIN):**

```sql
UPDATE table1
[JOIN table2 ON join_condition]
SET table1.column = table2.column
[WHERE condition];
```


**MERGE (SQL Server, Oracle, PostgreSQL):**

```sql
MERGE INTO target_table AS target
USING source_table AS source
ON target.join_key = source.join_key
WHEN MATCHED THEN
    UPDATE SET target.column = source.column
WHEN NOT MATCHED THEN
    INSERT (columns) VALUES (source.values);
```


**Component Breakdown:**

- `target_table`: The table to update
- `source_table`: The table providing the new values
- `SET`: Column assignments using source table columns
- `ON` / `WHERE`: Join condition linking the tables
- `WHEN MATCHED`: Action for rows that match the join condition
- `WHEN NOT MATCHED`: Action for rows that do not match

**Syntax Rules:**

- The join condition must uniquely identify matching rows to avoid ambiguous updates.
- Subqueries in the SET clause must return at most one row per target row.
- MERGE requires a semicolon at the end of the statement in SQL Server.
- MySQL's multi-table UPDATE cannot use ORDER BY or LIMIT.

**Constraints and Limitations:**

- Updating from a join may update the same row multiple times if the join produces multiple matches (implementation-dependent).
- MERGE has known concurrency issues in some databases (e.g., SQL Server) and should be used with caution.
- PostgreSQL's UPDATE ... FROM does not support LIMIT directly; use a subquery with ctid.

**Annotated Code Examples:**

```sql
-- Setup
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10,2),
    supplier_id INT
);

CREATE TABLE supplier_prices (
    supplier_id INT PRIMARY KEY,
    supplier_price DECIMAL(10,2)
);

INSERT INTO products VALUES (1, 'Widget', 19.99, 101);
INSERT INTO products VALUES (2, 'Gadget', 29.99, 102);
INSERT INTO products VALUES (3, 'Gizmo', 9.99, 101);

INSERT INTO supplier_prices VALUES (101, 15.00);
INSERT INTO supplier_prices VALUES (102, 25.00);

-- PostgreSQL: Update product prices from supplier prices
UPDATE products
SET price = supplier_prices.supplier_price * 1.20
FROM supplier_prices
WHERE products.supplier_id = supplier_prices.supplier_id;

-- Verify
SELECT * FROM products;
```

**Expected Output (PostgreSQL):**

| product_id | product_name | price | supplier_id |
|------------|--------------|-------|-------------|
| 1          | Widget       | 18.00 | 101         |
| 2          | Gadget       | 30.00 | 102         |
| 3          | Gizmo        | 18.00 | 101         |

**Explanation:** The UPDATE joins `products` with `supplier_prices` on `supplier_id` and sets each product's price to 120% of the supplier's price.

```sql
-- MySQL: Equivalent UPDATE with JOIN
UPDATE products
JOIN supplier_prices ON products.supplier_id = supplier_prices.supplier_id
SET products.price = supplier_prices.supplier_price * 1.20;

-- SQL Server: Equivalent UPDATE with FROM
UPDATE products
SET price = supplier_prices.supplier_price * 1.20
FROM products
JOIN supplier_prices ON products.supplier_id = supplier_prices.supplier_id;
```

```sql
-- SQL Server: MERGE example (upsert)
MERGE INTO products AS target
USING supplier_prices AS source
ON target.supplier_id = source.supplier_id
WHEN MATCHED THEN
    UPDATE SET target.price = source.supplier_price * 1.20
WHEN NOT MATCHED BY TARGET THEN
    INSERT (supplier_id, product_name, price)
    VALUES (source.supplier_id, 'New Product', source.supplier_price);
```

**Real-World Cases:**

- **E-commerce:** Updating product prices from a supplier feed.
- **CRM:** Copying customer data from a staging table to the main table.
- **Data warehousing:** Loading dimension tables from source systems.
- **Inventory:** Updating stock levels from warehouse management systems.

**References:**

- PostgreSQL: UPDATE - https://www.postgresql.org/docs/14/sql-update.html
- Microsoft Learn: MERGE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql
- MySQL: UPDATE Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/update.html


### 3. Safety & Execution Controls

#### 3.1 Safe Update Practices

**Core Definitions:**

- **Core Definition:** Safe update practices are techniques used to verify which rows will be affected before executing an UPDATE, and to test changes in a controlled manner.
- **Technical Definition:** Safe update practices include using SELECT statements to preview affected rows, wrapping updates in transactions that can be rolled back, and using database-specific safe-update modes.
- **Beginner-Friendly Explanation:** Before you make a change, look at what you're about to change. Run a SELECT first, then run the UPDATE inside a transaction so you can undo it if something goes wrong.

**Purposes:**

- To prevent accidental data loss or corruption
- To verify the scope of an UPDATE before executing it
- To provide a recovery mechanism if the update produces unexpected results
- To comply with change management and auditing requirements

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- Step 1: Verify with SELECT
SELECT * FROM table_name WHERE condition;

-- Step 2: Begin transaction
BEGIN TRANSACTION;  -- or START TRANSACTION in MySQL

-- Step 3: Execute UPDATE
UPDATE table_name SET column = value WHERE condition;

-- Step 4: Verify results
SELECT * FROM table_name WHERE condition;

-- Step 5: Commit or rollback
COMMIT;    -- if results are correct
-- or
ROLLBACK;  -- if results are incorrect
```

**Component Breakdown:**

- `BEGIN TRANSACTION` / `START TRANSACTION`: Starts a transaction block
- `COMMIT`: Permanently saves the changes
- `ROLLBACK`: Undoes all changes made since the transaction began

**Syntax Rules:**

- The SELECT statement should use the same WHERE clause as the UPDATE.
- The transaction should include both the UPDATE and the verification SELECT.
- Rollback is only possible if the transaction has not been committed.

**Constraints and Limitations:**

- Some databases auto-commit each statement by default (e.g., MySQL with autocommit=1).
- DDL statements (e.g., ALTER TABLE) may implicitly commit transactions.
- Long-running transactions can cause locking and performance issues.

**Annotated Code Examples:**

```sql
-- Setup
CREATE TABLE inventory (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    quantity INT,
    price DECIMAL(10,2)
);

INSERT INTO inventory VALUES (1, 'Widget', 100, 19.99);
INSERT INTO inventory VALUES (2, 'Gadget', 50, 29.99);
INSERT INTO inventory VALUES (3, 'Gizmo', 0, 9.99);

-- Step 1: Verify which rows will be affected
SELECT * FROM inventory WHERE quantity = 0;

-- Step 2: Begin transaction
BEGIN TRANSACTION;

-- Step 3: Execute UPDATE (set price to 0 for out-of-stock items)
UPDATE inventory
SET price = 0
WHERE quantity = 0;

-- Step 4: Verify
SELECT * FROM inventory WHERE quantity = 0;

-- Step 5: Since only Gizmo is affected, commit
COMMIT;

-- If something went wrong, ROLLBACK instead
```

**Expected Output (after COMMIT):**

| product_id | product_name | quantity | price |
|------------|--------------|----------|-------|
| 1          | Widget       | 100      | 19.99 |
| 2          | Gadget       | 50       | 29.99 |
| 3          | Gizmo        | 0        | 0.00  |

**Explanation:** The SELECT before the UPDATE confirms that only Gizmo (quantity=0) will be affected. The transaction allows verification before committing.

**Real-World Cases:**

- **Production databases:** Always test UPDATEs in a transaction before committing.
- **Data migrations:** Verify row counts before and after bulk updates.
- **Financial systems:** Use transactions to ensure atomicity of multi-step updates.
- **Compliance:** Maintain an audit trail of what was changed and when.

**References:**

- PostgreSQL: Transactions - https://www.postgresql.org/docs/current/tutorial-transactions.html
- MySQL: START TRANSACTION - https://dev.mysql.com/doc/refman/8.0/en/commit.html


#### 3.2 Preventing Catastrophic Updates

**Core Definitions:**

- **Core Definition:** Catastrophic updates are UPDATE statements that unintentionally modify every row in a table (or a very large number of rows) due to a missing or ineffective WHERE clause.
- **Technical Definition:** The most common cause is omitting the WHERE clause entirely, which causes the UPDATE to affect all rows in the target table. Database safe-update modes can prevent this by requiring a key-based WHERE clause or a LIMIT clause.
- **Beginner-Friendly Explanation:** If you forget the WHERE clause, you update every row in the table. This is the database equivalent of clicking "Reply All" by mistake.

**Purposes:**

- To prevent accidental mass data modifications
- To enforce deliberate, verified update operations
- To catch missing WHERE clauses before execution
- To protect against SQL injection and application bugs

**Syntax Structures and Rules:**

**MySQL Safe Updates Mode:**

```sql
-- Enable safe updates
SET sql_safe_updates = 1;
-- or start the mysql client with --safe-updates
```

With safe updates enabled, UPDATE statements that do not use a key in the WHERE clause or a LIMIT clause produce an error.

**SQL Server Safe Updates:**

```sql
-- SQL Server does not have a built-in safe update mode, but you can:
-- 1. Use SET NOCOUNT OFF to see affected rows
-- 2. Use a transaction with ROLLBACK for testing
-- 3. Use TOP (n) to limit the update
```

**Component Breakdown:**

- `sql_safe_updates`: MySQL system variable (global/session scope, boolean, default OFF)
- `--safe-updates`: MySQL client command-line option
- `SET SAFE_UPDATES ON`: Some third-party tools implement this

**Syntax Rules:**

- In MySQL, safe updates mode requires the WHERE clause to use a key column or include a LIMIT clause.
- The mode also restricts SELECT statements that produce very large result sets.
- Safe updates mode is disabled by default.

**Constraints and Limitations:**

- Safe updates mode is MySQL-specific.
- Even with safe updates, a well-crafted WHERE clause with a non-unique key can still update many rows.
- Some databases (e.g., PostgreSQL) do not have a built-in safe update mode.

**Annotated Code Examples:**

```sql
-- MySQL: Enable safe updates
SET sql_safe_updates = 1;

-- This will FAIL because there's no WHERE clause
UPDATE inventory SET price = 0;
-- Error: You are using safe update mode and you tried to update a table
-- without a WHERE that uses a KEY column

-- This will SUCCEED because WHERE uses the primary key
UPDATE inventory SET price = 0 WHERE product_id = 3;

-- This will SUCCEED because of the LIMIT clause
UPDATE inventory SET price = 0 LIMIT 1;

-- Disable safe updates
SET sql_safe_updates = 0;
```

**Real-World Cases:**

- **Production databases:** Always enable safe updates or use transactions.
- **Development environments:** Enable safe updates to catch mistakes early.
- **Database migration scripts:** Include explicit WHERE clauses and verification steps.

**References:**

- MySQL: Safe Updates Mode - https://downloads.mysql.com/docs/refman-5.6-en.a4.pdf
- Microsoft Q&A: Safe Updates Feature - https://learn.microsoft.com/en-us/answers/questions/2148298/safe-updates-feature-in-sql


#### 3.3 Handling Constraint Violations During Updates

**Core Definitions:**

- **Core Definition:** Constraint violations occur when an UPDATE statement attempts to modify data in a way that violates a PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK, or NOT NULL constraint.
- **Technical Definition:** Integrity constraint violations occur when an insert, update, or delete statement violates a primary key, foreign key, check, or unique constraint or a unique index. The database rejects the statement with an error code (typically SQLSTATE 23000).
- **Beginner-Friendly Explanation:** The database has rules (constraints) that data must follow. If your update breaks one of those rules—for example, trying to set a value that already exists in a unique column—the database refuses to make the change.

**Purposes:**

- To maintain data integrity and consistency
- To enforce business rules at the database level
- To prevent orphaned records and duplicate data
- To ensure that updates do not corrupt relationships between tables

**Syntax Structures and Rules:**

**Common Constraint Types:**

| Constraint | Violation Scenario | Error Code |
|------------|-------------------|------------|
| PRIMARY KEY | Updating a PK to a duplicate value | 23000 |
| UNIQUE | Updating a unique column to a duplicate value | 23000 |
| FOREIGN KEY | Updating a FK to a value not in the parent table | 23000 |
| CHECK | Updating a column to a value that fails the check condition | 23000 |
| NOT NULL | Updating a NOT NULL column to NULL | 23000 |

**Syntax Rules:**

- All constraints are checked at the end of the UPDATE statement (in most databases).
- In some databases, constraints can be deferred to the end of the transaction.
- Foreign key constraints may have ON UPDATE CASCADE or ON UPDATE SET NULL actions.

**Constraints and Limitations:**

- Some databases allow disabling constraints temporarily (e.g., `SET CONSTRAINTS ALL DEFERRED` in PostgreSQL).
- MySQL's IGNORE keyword causes constraint violations to be warnings instead of errors, skipping the offending rows.
- SQL Server's OUTPUT clause returns the rows affected before constraint violations.

**Annotated Code Examples:**

```sql
-- Setup
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(100) UNIQUE
);

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100) NOT NULL,
    dept_id INT,
    salary DECIMAL(10,2) CHECK (salary > 0),
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);

INSERT INTO departments VALUES (1, 'Engineering');
INSERT INTO departments VALUES (2, 'Marketing');

INSERT INTO employees VALUES (1, 'Alice', 1, 80000);
INSERT INTO employees VALUES (2, 'Bob', 2, 60000);

-- Example 1: UNIQUE constraint violation
UPDATE departments SET dept_name = 'Engineering' WHERE dept_id = 2;
-- Error: Duplicate entry 'Engineering' for key 'dept_name'

-- Example 2: FOREIGN KEY constraint violation
UPDATE employees SET dept_id = 999 WHERE emp_id = 1;
-- Error: Cannot add or update a child row: a foreign key constraint fails

-- Example 3: CHECK constraint violation
UPDATE employees SET salary = -1000 WHERE emp_id = 1;
-- Error: Check constraint 'employees_chk_1' is violated

-- Example 4: NOT NULL constraint violation
UPDATE employees SET emp_name = NULL WHERE emp_id = 1;
-- Error: Column 'emp_name' cannot be null

-- Valid updates
UPDATE employees SET salary = 85000 WHERE emp_id = 1;
UPDATE employees SET dept_id = 1 WHERE emp_id = 2;
```

**Expected Output:**

| Statement | Result |
|-----------|--------|
| Duplicate dept_name | Error 23000 |
| Invalid dept_id | Error 23000 |
| Negative salary | Error 23000 |
| NULL emp_name | Error 23000 |
| Valid salary update | Success |
| Valid dept_id update | Success |

**Explanation:** Each constraint violation produces an error and the statement is rejected. Valid updates proceed normally.

```sql
-- MySQL: Using IGNORE to skip constraint violations
UPDATE IGNORE employees SET dept_id = 999 WHERE emp_id = 1;
-- Warning instead of error; row is skipped
```

**Real-World Cases:**

- **Data migration:** Handling duplicate values when merging customer databases.
- **HR systems:** Preventing assignment of employees to non-existent departments.
- **Financial systems:** Ensuring account balances never go negative.
- **E-commerce:** Preventing duplicate product SKUs.

**References:**

- SAP Help Portal: Integrity Constraint Violations - https://help.sap.com/docs/SAP_ASE/41214a0aacb244b4983d171786f06412/aaf96f5cbc2b1014a29ab8f1585ae33e.html
- Cybrosys: How to Debug and Fix Constraint Violations in SQL - https://www.cybrosys.com/blog/how-to-debug-and-fix-constraint-violations-in-sql


### 4. Tracking Modifications

#### 4.1 Retrieving Modified Data (RETURNING / OUTPUT Clause)

**Core Definitions:**

- **Core Definition:** The RETURNING (PostgreSQL, Oracle) and OUTPUT (SQL Server) clauses return values from rows affected by an UPDATE statement, eliminating the need for a separate SELECT.
- **Technical Definition:** The optional RETURNING clause causes UPDATE to compute and return value(s) based on each row actually updated. Any expression using the table's columns can be computed, and the new (post-update) values are used. SQL Server's OUTPUT clause provides similar functionality.
- **Beginner-Friendly Explanation:** After you make a change, you often want to see what changed. Instead of running a separate SELECT, you can ask the UPDATE statement to return the modified rows.

**Purposes:**

- To retrieve the new values of updated rows in a single round trip
- To capture old and new values for auditing
- To feed updated values into subsequent operations
- To reduce network round trips and improve performance

**Syntax Structures and Rules:**

**PostgreSQL / Oracle (RETURNING):**

```sql
UPDATE table_name
SET column_name = value
WHERE condition
RETURNING * | output_expression [ [ AS ] output_name ] [, ...];
```


**SQL Server (OUTPUT):**

```sql
UPDATE table_name
SET column_name = value
OUTPUT INSERTED.column_name, DELETED.column_name
WHERE condition;
```

**Component Breakdown:**

- `RETURNING *`: Returns all columns of the updated rows
- `RETURNING expression`: Returns a specific expression or column
- `INSERTED.column`: The new value (after update) in SQL Server
- `DELETED.column`: The old value (before update) in SQL Server
- `INTO table_variable`: Stores the output in a table variable (SQL Server)

**Syntax Rules:**

- RETURNING can use any expression involving the table's columns.
- In SQL Server, OUTPUT can reference INSERTED and DELETED pseudo-tables.
- The RETURNING list syntax is identical to the SELECT output list.
- Oracle's RETURNING INTO clause requires PL/SQL variables to receive the values.

**Constraints and Limitations:**

- RETURNING cannot be used with multi-table UPDATE in MySQL.
- SQL Server's OUTPUT clause cannot return values if the UPDATE statement has a FROM clause with certain join types.
- The RETURNING clause returns values after triggers have fired.

**Annotated Code Examples:**

```sql
-- PostgreSQL: RETURNING example
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10,2),
    old_price DECIMAL(10,2)
);

INSERT INTO products VALUES (1, 'Widget', 19.99, NULL);
INSERT INTO products VALUES (2, 'Gadget', 29.99, NULL);

-- Update and return the new values
UPDATE products
SET old_price = price,
    price = price * 1.10
WHERE product_id = 1
RETURNING product_id, product_name, old_price, price;
```

**Expected Output:**

| product_id | product_name | old_price | price |
|------------|--------------|-----------|-------|
| 1          | Widget       | 19.99     | 21.99 |

**Explanation:** The RETURNING clause shows the old price (stored in old_price) and the new price in a single statement.

```sql
-- SQL Server: OUTPUT example
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10,2)
);

INSERT INTO products VALUES (1, 'Widget', 19.99);
INSERT INTO products VALUES (2, 'Gadget', 29.99);

-- Update and return old and new values
UPDATE products
SET price = price * 1.10
OUTPUT DELETED.product_id,
       DELETED.price AS old_price,
       INSERTED.price AS new_price
WHERE product_id = 1;
```

**Expected Output (SQL Server):**

| product_id | old_price | new_price |
|------------|-----------|-----------|
| 1          | 19.99     | 21.99     |

```sql
-- Oracle: RETURNING INTO example (PL/SQL)
DECLARE
    v_old_price NUMBER;
    v_new_price NUMBER;
BEGIN
    UPDATE products
    SET price = price * 1.10
    WHERE product_id = 1
    RETURNING old_price, price INTO v_old_price, v_new_price;
    
    DBMS_OUTPUT.PUT_LINE('Old: ' || v_old_price || ', New: ' || v_new_price);
END;
```

**Real-World Cases:**

- **Auditing:** Capturing old and new values for audit trails.
- **Application development:** Getting updated values without a second query.
- **Data synchronization:** Feeding updated values into other systems.
- **Reporting:** Generating change reports from UPDATE statements.

**References:**

- PostgreSQL: UPDATE - https://www.postgresql.org/docs/14/sql-update.html
- Microsoft Learn: OUTPUT Clause - https://learn.microsoft.com/en-us/sql/t-sql/queries/output-clause-transact-sql
- Oracle: RETURNING INTO Clause - https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/RETURNING-INTO-clause.html


#### 4.2 Impact of Updates on Table Metadata (Row Modification Timestamps, Triggers)

**Core Definitions:**

- **Core Definition:** UPDATE operations can affect table metadata, including row modification timestamps (via triggers or application logic) and trigger execution.
- **Technical Definition:** Database triggers are special stored procedures that automatically execute in response to UPDATE events. They can be used to maintain audit trails, update timestamp columns, enforce complex business rules, and synchronize related tables.
- **Beginner-Friendly Explanation:** You can set up automatic actions that happen whenever a row is updated—like stamping the row with the current time, or recording who made the change in an audit table.

**Purposes:**

- To automatically track when rows were last modified
- To maintain audit trails of all changes
- To enforce complex business rules that cannot be expressed as constraints
- To synchronize denormalized data or summary tables

**Syntax Structures and Rules:**

**PostgreSQL Trigger (Update Timestamp):**

```sql
-- Create a trigger function
CREATE OR REPLACE FUNCTION update_timestamp()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create a trigger
CREATE TRIGGER trigger_update_timestamp
BEFORE UPDATE ON table_name
FOR EACH ROW
EXECUTE FUNCTION update_timestamp();
```


**SQL Server Trigger (Update Timestamp):**

```sql
CREATE TRIGGER trg_update_timestamp
ON table_name
AFTER UPDATE
AS
BEGIN
    UPDATE table_name
    SET updated_at = GETDATE()
    FROM table_name t
    JOIN inserted i ON t.id = i.id;
END;
```


**MySQL Trigger (Update Timestamp):**

```sql
DELIMITER //
CREATE TRIGGER trg_update_timestamp
BEFORE UPDATE ON table_name
FOR EACH ROW
BEGIN
    SET NEW.updated_at = NOW();
END;
//
DELIMITER ;
```

**Component Breakdown:**

- `BEFORE UPDATE`: Trigger fires before the update is applied
- `AFTER UPDATE`: Trigger fires after the update is applied
- `FOR EACH ROW`: Trigger fires once per affected row
- `NEW`: Pseudo-record containing the new values (PostgreSQL, MySQL)
- `OLD`: Pseudo-record containing the old values (PostgreSQL, MySQL)
- `INSERTED` / `DELETED`: Pseudo-tables in SQL Server triggers

**Syntax Rules:**

- Triggers can be BEFORE or AFTER the update event.
- Triggers can be row-level (FOR EACH ROW) or statement-level.
- Multiple triggers can fire for the same event; execution order may be configurable.
- Triggers can modify the NEW values in BEFORE triggers (PostgreSQL, MySQL).

**Constraints and Limitations:**

- Triggers add overhead to every UPDATE operation.
- Recursive triggers can cause infinite loops (some databases have recursion limits).
- Triggers can mask application logic and make debugging harder.
- Some databases (e.g., MySQL) do not support statement-level triggers.
- Trigger behavior varies significantly across database vendors.

**Annotated Code Examples:**

```sql
-- PostgreSQL: Audit trail trigger
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10,2),
    updated_at TIMESTAMP,
    updated_by VARCHAR(100)
);

CREATE TABLE product_audit (
    audit_id SERIAL PRIMARY KEY,
    product_id INT,
    old_price DECIMAL(10,2),
    new_price DECIMAL(10,2),
    changed_at TIMESTAMP DEFAULT NOW(),
    changed_by VARCHAR(100)
);

-- Trigger function for timestamp and audit
CREATE OR REPLACE FUNCTION audit_product_update()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    NEW.updated_by = CURRENT_USER;
    
    IF OLD.price IS DISTINCT FROM NEW.price THEN
        INSERT INTO product_audit (product_id, old_price, new_price, changed_by)
        VALUES (OLD.product_id, OLD.price, NEW.price, CURRENT_USER);
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create the trigger
CREATE TRIGGER trg_audit_product_update
BEFORE UPDATE ON products
FOR EACH ROW
EXECUTE FUNCTION audit_product_update();

-- Test the trigger
INSERT INTO products VALUES (1, 'Widget', 19.99, NULL, NULL);

UPDATE products SET price = 24.99 WHERE product_id = 1;

-- Check the results
SELECT * FROM products;
SELECT * FROM product_audit;
```

**Expected Output (products):**

| product_id | product_name | price | updated_at          | updated_by |
|------------|--------------|-------|---------------------|------------|
| 1          | Widget       | 24.99 | 2024-06-15 10:30:00 | postgres   |

**Expected Output (product_audit):**

| audit_id | product_id | old_price | new_price | changed_at          | changed_by |
|----------|------------|-----------|-----------|---------------------|------------|
| 1        | 1          | 19.99     | 24.99     | 2024-06-15 10:30:00 | postgres   |

**Explanation:** The trigger automatically updates the `updated_at` and `updated_by` columns and inserts a record into the audit table whenever the price changes.

```sql
-- SQL Server: Trigger to update timestamp
CREATE TRIGGER trg_product_timestamp
ON products
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;
    UPDATE products
    SET updated_at = GETDATE()
    FROM products p
    JOIN inserted i ON p.product_id = i.product_id;
END;
```

**Real-World Cases:**

- **Audit compliance:** Maintaining a complete history of all data changes.
- **Data warehousing:** Tracking when dimension records were last updated.
- **Application debugging:** Identifying when and by whom data was modified.
- **Data synchronization:** Triggering updates to related tables.

**References:**

- PostgreSQL: CREATE TRIGGER - https://www.postgresql.org/docs/current/sql-createtrigger.html
- Microsoft Learn: CREATE TRIGGER (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/create-trigger-transact-sql
- MySQL: CREATE TRIGGER - https://dev.mysql.com/doc/refman/8.0/en/create-trigger.html


## Summary Tables

### UPDATE Syntax Comparison Across Databases

| Feature | PostgreSQL | SQL Server | MySQL | Oracle |
|---------|-----------|------------|-------|--------|
| Basic UPDATE | ✓ | ✓ | ✓ | ✓ |
| UPDATE ... FROM | ✓ | ✓ | ✗ | ✗ |
| UPDATE ... JOIN | ✗ | ✗ | ✓ | ✗ |
| RETURNING clause | ✓ | ✗ | ✗ | ✓ |
| OUTPUT clause | ✗ | ✓ | ✗ | ✗ |
| MERGE | ✓ (15+) | ✓ | ✗ | ✓ |
| LIMIT clause | ✗ | ✗ | ✓ | ✗ (use ROWNUM) |
| ORDER BY in UPDATE | ✗ | ✗ | ✓ | ✗ |
| Safe updates mode | ✗ | ✗ | ✓ | ✗ |

### Constraint Violation Error Codes

| Constraint | SQLSTATE | Description |
|------------|----------|-------------|
| PRIMARY KEY | 23000 | Duplicate key value |
| UNIQUE | 23000 | Duplicate value in unique column |
| FOREIGN KEY | 23000 | Referenced value does not exist |
| CHECK | 23000 | Check condition evaluates to FALSE |
| NOT NULL | 23000 | NULL value in NOT NULL column |

---

## References

- PostgreSQL: UPDATE - https://www.postgresql.org/docs/14/sql-update.html
- MySQL: UPDATE Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/update.html
- Microsoft Learn: UPDATE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/update-transact-sql
- Microsoft Learn: MERGE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql
- Microsoft Learn: OUTPUT Clause - https://learn.microsoft.com/en-us/sql/t-sql/queries/output-clause-transact-sql
- Oracle: UPDATE Statement - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/UPDATE.html
- Oracle: RETURNING INTO Clause - https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/RETURNING-INTO-clause.html
- MySQL: Safe Updates Mode - https://downloads.mysql.com/docs/refman-5.6-en.a4.pdf
- SAP Help Portal: Integrity Constraint Violations - https://help.sap.com/docs/SAP_ASE/41214a0aacb244b4983d171786f06412/aaf96f5cbc2b1014a29ab8f1585ae33e.html
- PostgreSQL: CREATE TRIGGER - https://www.postgresql.org/docs/current/sql-createtrigger.html
- Microsoft Learn: CREATE TRIGGER (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/create-trigger-transact-sql
- MySQL: CREATE TRIGGER - https://dev.mysql.com/doc/refman/8.0/en/create-trigger.html