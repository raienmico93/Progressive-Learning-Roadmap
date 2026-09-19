# SQL DELETE Operations: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The SQL DELETE statement is a Data Manipulation Language (DML) command that removes one or more rows from a table or view.

**Technical Definition:** Per the SQL standard (ISO/IEC 9075), DELETE is a DML statement that removes rows satisfying a given search condition from a specified table. The statement supports optional WHERE, USING (PostgreSQL), RETURNING (PostgreSQL, Oracle), OUTPUT (SQL Server), and vendor-specific extensions such as LIMIT and ORDER BY (MySQL). DELETE is a logged operation, meaning each row removal is recorded in the transaction log, and it can be rolled back within a transaction.

**Beginner-Friendly Explanation:** Think of a database table as a spreadsheet. DELETE is the command that removes entire rows (records) from that spreadsheet. You tell the database which table to delete from and which rows to remove using a condition. If you don't specify a condition, every row in the table is deleted.

### Key Characteristics

- **Row-level removal:** DELETE removes entire rows, not individual column values.
- **Conditional execution:** The WHERE clause determines which rows are deleted; without it, all rows are removed.
- **Logged operation:** Each row deletion is recorded in the transaction log, enabling rollback and point-in-time recovery.
- **Transactional:** DELETE operations can be committed or rolled back.
- **Trigger-aware:** DELETE triggers fire for each row deleted (unlike TRUNCATE).
- **Vendor extensions:** RETURNING (PostgreSQL, Oracle), OUTPUT (SQL Server), USING (PostgreSQL), multi-table DELETE (MySQL), and TOP (SQL Server).

### Prerequisites

- Basic SQL syntax (SELECT, FROM, WHERE)
- Understanding of tables, columns, and data types
- Familiarity with NULL handling (see SQL NULL Handling cheat sheet)
- Knowledge of constraints (PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK)
- Transaction concepts (COMMIT, ROLLBACK, SAVEPOINT)

### Related Programming Areas

- **Database Administration:** DELETE operations affect table metadata, indexes, and statistics.
- **Application Development:** Most applications perform DELETE operations through ORMs or parameterized queries.
- **Data Warehousing and ETL:** Bulk deletions and soft-delete patterns are common in data transformation pipelines.
- **Auditing and Compliance:** Tracking deletions through triggers and audit tables.
- **Concurrency Control:** DELETE statements interact with locking mechanisms and isolation levels.
- **Data Recovery:** Understanding logging and recovery models is essential for restoring deleted data.

---

## Core Concepts / Key Features

### 1. Core Deletion Methods

#### 1.1 DELETE FROM Basic Syntax and Mechanics

**Core Definitions:**

- **Core Definition:** The basic DELETE FROM statement identifies a target table and optionally filters rows to remove using a WHERE clause.
- **Technical Definition:** The ANSI SQL standard defines the DELETE statement with the syntax `DELETE FROM table_name [WHERE condition]`. The statement removes all rows that satisfy the condition. Without a WHERE clause, all rows are removed, resulting in an empty but structurally intact table.
- **Beginner-Friendly Explanation:** The DELETE statement is like telling a spreadsheet: "Remove these rows from this sheet, but only the ones that match this condition."

**Purposes:**

- To permanently remove unwanted or obsolete data from a table
- To clean up temporary or staging data
- To implement data retention policies (deleting old records)
- To enforce referential integrity through cascading deletes

**Syntax Structures and Rules:**

**Complete General Syntax (ANSI SQL):**

```sql
DELETE FROM table_name
[WHERE condition];
```

**PostgreSQL Syntax:**

```sql
[ WITH [ RECURSIVE ] with_query [, ...] ]
DELETE FROM [ ONLY ] table_name [ * ] [ [ AS ] alias ]
[ USING from_item [, ...] ]
[ WHERE condition | WHERE CURRENT OF cursor_name ]
[ RETURNING { * | output_expression [ [ AS ] output_name ] } [, ...] ]
```

**SQL Server Syntax:**

```sql
[ WITH <common_table_expression> [ ,...n ] ]
DELETE [ TOP ( expression ) [ PERCENT ] ]
[ FROM ] { { table_alias | <object> | rowset_function_limited }
[ <OUTPUT Clause> ]
[ FROM table_source [ ,...n ] ]
[ WHERE { <search_condition> | { [ CURRENT OF { cursor_name } ] } } ]
[ OPTION ( <Query Hint> [ ,...n ] ) ] [;]
```

**MySQL Syntax:**

```sql
DELETE [LOW_PRIORITY] [QUICK] [IGNORE] FROM tbl_name [[AS] tbl_alias]
[PARTITION (partition_name [, partition_name] ...)]
[WHERE where_condition]
[ORDER BY ...]
[LIMIT row_count]
```

**Oracle Syntax:**

```sql
DELETE [FROM] table_name [alias]
[WHERE condition]
[RETURNING expression INTO variable [, ...]];
```

**Component Breakdown:**

- `table_name`: The name of the table or view from which to delete rows
- `WHERE`: Optional clause that filters which rows are deleted
- `USING` (PostgreSQL): Additional tables that can be referenced in the WHERE condition
- `RETURNING` (PostgreSQL, Oracle): Returns values from the deleted rows
- `OUTPUT` (SQL Server): Returns values from the deleted rows
- `LIMIT` (MySQL): Restricts the number of rows deleted
- `TOP` (SQL Server): Restricts the number of rows deleted

**Syntax Rules:**

- Only the rows that satisfy the WHERE condition are deleted.
- Without a WHERE clause, all rows in the target table are deleted.
- Views can be deleted from only if they are simple enough (typically based on a single table).
- DELETE requires the DELETE privilege on the target table and the SELECT privilege on any tables referenced in the WHERE clause.

**Constraints and Limitations:**

- FOREIGN KEY constraints may prevent deletion if dependent rows exist (unless ON DELETE CASCADE is specified).
- DELETE triggers fire for each row deleted.
- Large DELETE operations can lock tables and consume significant transaction log space.
- DELETE cannot be used on tables referenced by FOREIGN KEY constraints if the referencing table has rows (unless CASCADE or SET NULL is configured).

**Annotated Code Examples:**

```sql
-- Setup: Create sample tables
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    email VARCHAR(200),
    status VARCHAR(20)
);

INSERT INTO customers VALUES (1, 'Alice', 'alice@example.com', 'Active');
INSERT INTO customers VALUES (2, 'Bob', 'bob@example.com', 'Inactive');
INSERT INTO customers VALUES (3, 'Charlie', 'charlie@example.com', 'Active');
INSERT INTO customers VALUES (4, 'Diana', 'diana@example.com', 'Inactive');

-- Example 1: Delete a specific row
DELETE FROM customers
WHERE customer_id = 2;

-- Verify
SELECT * FROM customers;
```

**Expected Output:**

| customer_id | customer_name | email               | status |
|-------------|---------------|---------------------|--------|
| 1           | Alice         | alice@example.com   | Active |
| 3           | Charlie       | charlie@example.com | Active |
| 4           | Diana         | diana@example.com   | Inactive |

**Explanation:** The WHERE clause targets only the row where `customer_id = 2`. Only Bob's row is removed.

```sql
-- Example 2: Delete all rows (no WHERE clause)
DELETE FROM customers;

-- Verify
SELECT * FROM customers;
```

**Expected Output:** Empty table (no rows).

**Explanation:** Without a WHERE clause, every row is deleted. The table structure remains, but all data is gone.

**Real-World Cases:**

- **Data retention:** Deleting customer records that have been inactive for more than 5 years.
- **GDPR compliance:** Removing personal data upon user request.
- **Staging tables:** Clearing temporary data after ETL processing.
- **Log management:** Deleting old log entries to manage storage.

**References:**

- PostgreSQL: DELETE - https://www.postgresql.org/docs/14/sql-delete.html
- MySQL: DELETE Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/delete.html
- Microsoft Learn: DELETE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/delete-transact-sql


#### 1.2 Deleting Selected Rows (Targeted Filtering)

**Core Definitions:**

- **Core Definition:** Targeted deletion uses a WHERE clause with a unique condition (typically a primary key) to delete exactly one row.
- **Technical Definition:** When the WHERE clause uses a condition that uniquely identifies a row (e.g., `WHERE primary_key = value`), the DELETE affects at most one row.
- **Beginner-Friendly Explanation:** If you know exactly which row you want to remove—for example, order number 42—you use a WHERE clause that targets only that row.

**Purposes:**

- To make precise, targeted data removals
- To delete a specific record without affecting others
- To minimize the risk of unintended data loss
- To support application-level row-level operations

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
DELETE FROM table_name
WHERE unique_column = unique_value;
```

**Component Breakdown:**

- `unique_column`: A column with a PRIMARY KEY or UNIQUE constraint
- `unique_value`: A value that uniquely identifies one row

**Syntax Rules:**

- The WHERE clause must use a condition that uniquely identifies the row.
- Primary key or unique index columns are the safest choices.
- If multiple rows match the condition, multiple rows will be deleted.

**Constraints and Limitations:**

- If the unique value does not exist, no rows are deleted (not an error).
- If the condition matches multiple rows, multiple rows are deleted.

**Annotated Code Examples:**

```sql
-- Setup
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    order_date DATE,
    total DECIMAL(10,2)
);

INSERT INTO orders VALUES (1, 'Alice', '2024-01-15', 150.00);
INSERT INTO orders VALUES (2, 'Bob', '2024-01-20', 200.00);
INSERT INTO orders VALUES (3, 'Charlie', '2024-02-10', 75.00);

-- Delete a single row by primary key
DELETE FROM orders
WHERE order_id = 2;

-- Verify
SELECT * FROM orders;
```

**Expected Output:**

| order_id | customer_name | order_date | total  |
|----------|---------------|------------|--------|
| 1        | Alice         | 2024-01-15 | 150.00 |
| 3        | Charlie       | 2024-02-10 | 75.00  |

**Explanation:** The WHERE clause uses the primary key (`order_id = 2`), guaranteeing that only one row is deleted.

**Real-World Cases:**

- **Order management:** Cancelling a specific order by order ID.
- **User management:** Deleting a specific user account.
- **Inventory:** Removing a specific product from the catalog.
- **Customer support:** Deleting a specific support ticket.

**References:**

- PostgreSQL: DELETE - https://www.postgresql.org/docs/14/sql-delete.html
- Microsoft Learn: DELETE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/delete-transact-sql


#### 1.3 Conditional Deletion (Complex WHERE Clauses)

**Core Definitions:**

- **Core Definition:** Conditional deletion uses complex WHERE clauses involving multiple conditions, subqueries, and logical operators to delete groups of rows.
- **Technical Definition:** The WHERE clause can contain any valid SQL condition, including AND/OR/NOT operators, IN, EXISTS, BETWEEN, LIKE, and subqueries. All rows for which the condition evaluates to TRUE are deleted.
- **Beginner-Friendly Explanation:** You can delete rows based on multiple criteria—for example, "delete all inactive customers who haven't logged in for a year."

**Purposes:**

- To delete rows based on multiple related conditions
- To delete rows based on data in other tables (subqueries)
- To implement complex business rules for data removal
- To clean up data based on temporal or status-based criteria

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
DELETE FROM table_name
WHERE condition1
  AND condition2
  OR condition3
  AND column IN (subquery)
  AND EXISTS (subquery);
```

**Component Breakdown:**

- `condition1, condition2, ...`: Any valid SQL conditions
- `AND`, `OR`, `NOT`: Logical operators
- `IN`: Checks if a value matches any value in a subquery or list
- `EXISTS`: Checks if a subquery returns any rows
- `BETWEEN`: Checks if a value is within a range

**Syntax Rules:**

- The WHERE clause can reference columns from the target table and, in some databases, from other tables (via subqueries or USING/FROM clauses).
- Subqueries must return a single column for IN comparisons.
- EXISTS subqueries should be correlated with the outer query.
- NULL comparisons in the WHERE clause evaluate to UNKNOWN, so rows with NULLs are excluded.

**Constraints and Limitations:**

- Complex WHERE clauses can be slow on large tables without proper indexes.
- Some databases limit the complexity of WHERE clauses.
- Subqueries in DELETE statements may have restrictions in some databases (e.g., MySQL cannot modify the same table in a subquery).

**Annotated Code Examples:**

```sql
-- Setup
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    email VARCHAR(200),
    last_login DATE,
    status VARCHAR(20),
    total_spent DECIMAL(10,2)
);

INSERT INTO customers VALUES (1, 'Alice', 'alice@example.com', '2024-06-01', 'Active', 5000);
INSERT INTO customers VALUES (2, 'Bob', 'bob@example.com', '2023-01-15', 'Inactive', 200);
INSERT INTO customers VALUES (3, 'Charlie', 'charlie@example.com', '2024-05-20', 'Active', 3000);
INSERT INTO customers VALUES (4, 'Diana', 'diana@example.com', '2022-12-01', 'Inactive', 100);
INSERT INTO customers VALUES (5, 'Eve', 'eve@example.com', NULL, 'Active', 0);

-- Delete inactive customers who haven't logged in for over a year
DELETE FROM customers
WHERE status = 'Inactive'
  AND last_login < '2023-06-01';

-- Verify
SELECT * FROM customers;
```

**Expected Output:**

| customer_id | customer_name | email               | last_login | status | total_spent |
|-------------|---------------|---------------------|------------|--------|-------------|
| 1           | Alice         | alice@example.com   | 2024-06-01 | Active | 5000        |
| 3           | Charlie       | charlie@example.com | 2024-05-20 | Active | 3000        |
| 5           | Eve           | eve@example.com     | NULL       | Active | 0           |

**Explanation:** Bob and Diana are deleted because they are Inactive and their last login was before June 1, 2023. Eve is not deleted because she is Active (even though her last_login is NULL). Alice and Charlie are Active, so they remain.

```sql
-- Delete customers who have never placed an order (using subquery)
DELETE FROM customers
WHERE customer_id NOT IN (
    SELECT DISTINCT customer_id FROM orders
);
```

**Explanation:** This deletes customers who do not appear in the orders table.

**Real-World Cases:**

- **Data cleaning:** Deleting records with invalid or incomplete data.
- **Retention policies:** Deleting records older than a specified date.
- **Fraud detection:** Deleting accounts flagged as fraudulent.
- **Inventory:** Deleting discontinued products with no stock.

**References:**

- PostgreSQL: DELETE - https://www.postgresql.org/docs/14/sql-delete.html
- MySQL: DELETE Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/delete.html


#### 1.4 Deleting All Rows vs. Truncating a Table (DELETE vs. TRUNCATE)

**Core Definitions:**

- **Core Definition:** DELETE removes rows one at a time and logs each deletion, while TRUNCATE removes all rows by deallocating data pages and logs only page deallocations.
- **Technical Definition:** TRUNCATE TABLE is functionally equivalent to DELETE with no WHERE clause, but it is faster and uses fewer system and transaction log resources. TRUNCATE resets identity columns, does not fire triggers, and cannot be used on tables referenced by FOREIGN KEY constraints.
- **Beginner-Friendly Explanation:** DELETE is like erasing each row individually with an eraser. TRUNCATE is like tearing out the entire page and replacing it with a blank one.

**Purposes:**

- To quickly empty a table when all rows must be removed
- To reset identity/auto-increment counters
- To minimize transaction log usage
- To improve performance for bulk removal operations

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- DELETE all rows
DELETE FROM table_name;

-- TRUNCATE
TRUNCATE TABLE table_name;
```

**Component Breakdown:**

- `TRUNCATE TABLE`: Removes all rows from the specified table
- No WHERE clause is allowed with TRUNCATE

**Syntax Rules:**

- TRUNCATE requires ALTER privilege on the table (SQL Server) or DROP privilege (MySQL).
- TRUNCATE cannot be used on tables referenced by FOREIGN KEY constraints.
- TRUNCATE cannot activate triggers (SQL Server).
- TRUNCATE resets identity columns to their seed value (SQL Server, MySQL).
- DELETE can be rolled back; TRUNCATE may or may not be rollback-able depending on the database.

**Constraints and Limitations:**

- **SQL Server:** TRUNCATE cannot be used on tables that are referenced by FOREIGN KEY constraints, participate in indexed views, or are published by transactional replication.
- **MySQL:** TRUNCATE requires DROP privilege and implicitly commits the transaction.
- **PostgreSQL:** TRUNCATE is transactional and can be rolled back. It also supports CASCADE and RESTART IDENTITY options.
- **Oracle:** TRUNCATE is DDL, not DML, and cannot be rolled back.

**Annotated Code Examples:**

```sql
-- Setup
CREATE TABLE temp_data (
    id INT PRIMARY KEY,
    value VARCHAR(100)
);

INSERT INTO temp_data VALUES (1, 'A');
INSERT INTO temp_data VALUES (2, 'B');
INSERT INTO temp_data VALUES (3, 'C');

-- Using DELETE to remove all rows
DELETE FROM temp_data;
-- Log: Each row deletion is logged individually
-- Triggers fire for each row
-- Identity counter is NOT reset

-- Using TRUNCATE to remove all rows
TRUNCATE TABLE temp_data;
-- Log: Only page deallocations are logged
-- Triggers do NOT fire
-- Identity counter IS reset
```

**Expected Output:** Both statements result in an empty table. The difference is in performance, logging, and side effects.

**Real-World Cases:**

- **Staging tables:** TRUNCATE is preferred for clearing staging tables before reload.
- **Test databases:** TRUNCATE is used to reset tables between tests.
- **Log tables:** DELETE is used to remove old logs based on date criteria.
- **Production tables:** DELETE is used when FOREIGN KEY constraints or triggers must be respected.

**References:**

- Microsoft Learn: TRUNCATE TABLE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/truncate-table-transact-sql
- MySQL: TRUNCATE TABLE Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/truncate-table.html


### 2. Complex & Related Deletions

#### 2.1 Cascading Deletes (Foreign Key Constraint Behaviors)

**Core Definitions:**

- **Core Definition:** Cascading deletes automatically remove or modify rows in child tables when a parent row is deleted, based on the ON DELETE action defined in the FOREIGN KEY constraint.
- **Technical Definition:** The SQL standard defines four ON DELETE actions: NO ACTION (default), CASCADE, SET NULL, and SET DEFAULT. CASCADE deletes corresponding child rows; SET NULL sets foreign key columns to NULL; SET DEFAULT sets them to their default values; NO ACTION raises an error and rolls back the delete.
- **Beginner-Friendly Explanation:** When you delete a parent record (like a customer), cascading deletes automatically remove all related child records (like their orders). You configure this behavior when creating the foreign key.

**Purposes:**

- To maintain referential integrity automatically
- To simplify application logic by delegating cleanup to the database
- To prevent orphaned records
- To implement complex data hierarchies

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
CREATE TABLE child_table (
    ...
    FOREIGN KEY (column_name)
    REFERENCES parent_table(parent_column)
    ON DELETE { NO ACTION | CASCADE | SET NULL | SET DEFAULT }
);
```

**Component Breakdown:**

- `NO ACTION`: Raises an error and rolls back the delete if dependent rows exist (default).
- `CASCADE`: Deletes corresponding rows from the referencing table.
- `SET NULL`: Sets all foreign key values to NULL if the parent row is deleted. The foreign key columns must be nullable.
- `SET DEFAULT`: Sets all foreign key values to their default values. All foreign key columns must have default definitions.

**Syntax Rules:**

- ON DELETE CASCADE cannot be defined if an INSTEAD OF trigger ON DELETE already exists on the table.
- SET NULL requires the foreign key columns to be nullable.
- SET DEFAULT requires default values defined for all foreign key columns.
- Cascading actions can chain across multiple levels of foreign keys.

**Constraints and Limitations:**

- Cascading deletes can trigger other cascading deletes, potentially causing a chain reaction.
- Some databases limit the depth of cascading.
- Cascading deletes can be slow on large tables.
- NO ACTION is the default and is the safest option.

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
);

INSERT INTO departments VALUES (1, 'Engineering');
INSERT INTO departments VALUES (2, 'Marketing');

INSERT INTO employees VALUES (1, 'Alice', 1);
INSERT INTO employees VALUES (2, 'Bob', 1);
INSERT INTO employees VALUES (3, 'Charlie', 2);

-- Delete a department (cascades to employees)
DELETE FROM departments WHERE dept_id = 1;

-- Verify
SELECT * FROM employees;
```

**Expected Output:**

| emp_id | emp_name | dept_id |
|--------|----------|---------|
| 3      | Charlie  | 2       |

**Explanation:** Deleting the Engineering department (dept_id=1) automatically deletes Alice and Bob because the foreign key has ON DELETE CASCADE.

```sql
-- Example: SET NULL
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

- **E-commerce:** Deleting a customer cascades to their orders and order items.
- **CMS:** Deleting a blog post cascades to comments.
- **HR:** Deleting a department cascades to employees (or sets their dept_id to NULL).
- **Inventory:** Deleting a product cascades to inventory records.

**References:**

- Microsoft Learn: CREATE TABLE (Transact-SQL) - FOREIGN KEY - https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql
- MySQL: FOREIGN KEY Constraints - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/create-table-foreign-keys.html


#### 2.2 Deleting from Related Data / Joins

**Core Definitions:**

- **Core Definition:** DELETE statements can remove rows based on data in other tables using subqueries, the USING clause (PostgreSQL), multi-table syntax (MySQL), or joins in the FROM clause (SQL Server).
- **Technical Definition:** The SQL standard provides subqueries in the WHERE clause as the primary mechanism. PostgreSQL adds the USING clause; MySQL supports multi-table DELETE syntax; SQL Server supports DELETE with a FROM clause that includes joins.
- **Beginner-Friendly Explanation:** Sometimes you need to delete rows from one table based on information in another table. You can use a subquery or a join to identify which rows to delete.

**Purposes:**

- To delete rows based on conditions in related tables
- To clean up orphaned records
- To implement complex data retention rules
- To synchronize data between related tables

**Syntax Structures and Rules:**

**PostgreSQL (USING clause):**

```sql
DELETE FROM target_table
USING source_table
WHERE target_table.join_key = source_table.join_key
  AND condition;
```

**MySQL (Multi-Table Syntax):**

```sql
DELETE target_table
FROM target_table
JOIN source_table ON target_table.join_key = source_table.join_key
WHERE condition;
```

**SQL Server (DELETE with FROM):**

```sql
DELETE target_table
FROM target_table
JOIN source_table ON target_table.join_key = source_table.join_key
WHERE condition;
```

**Component Breakdown:**

- `target_table`: The table from which to delete rows
- `source_table`: The table providing the condition
- `join_key`: The column(s) used to join the tables
- `condition`: Additional filtering conditions

**Syntax Rules:**

- The join condition must uniquely identify matching rows to avoid ambiguous deletions.
- Subqueries in the WHERE clause must be correlated with the outer DELETE.
- MySQL's multi-table DELETE cannot use ORDER BY or LIMIT.
- PostgreSQL's USING clause supports the same syntax as SELECT's FROM clause.

**Constraints and Limitations:**

- Deleting from a join may delete the same row multiple times if the join produces multiple matches (implementation-dependent).
- Some databases restrict deleting from a table that is also used in a subquery.
- Multi-table DELETE in MySQL requires the DELETE privilege on all tables involved.

**Annotated Code Examples:**

```sql
-- Setup
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total DECIMAL(10,2)
);

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    status VARCHAR(20)
);

INSERT INTO customers VALUES (1, 'Alice', 'Active');
INSERT INTO customers VALUES (2, 'Bob', 'Inactive');
INSERT INTO customers VALUES (3, 'Charlie', 'Inactive');

INSERT INTO orders VALUES (101, 1, '2024-01-15', 150.00);
INSERT INTO orders VALUES (102, 2, '2024-01-20', 200.00);
INSERT INTO orders VALUES (103, 3, '2024-02-10', 75.00);
INSERT INTO orders VALUES (104, 1, '2024-03-01', 300.00);

-- PostgreSQL: Delete orders for inactive customers
DELETE FROM orders
USING customers
WHERE orders.customer_id = customers.customer_id
  AND customers.status = 'Inactive';

-- Verify
SELECT * FROM orders;
```

**Expected Output (PostgreSQL):**

| order_id | customer_id | order_date | total  |
|----------|-------------|------------|--------|
| 101      | 1           | 2024-01-15 | 150.00 |
| 104      | 1           | 2024-03-01 | 300.00 |

**Explanation:** Orders 102 and 103 are deleted because they belong to Bob and Charlie, who are Inactive.

```sql
-- MySQL: Equivalent multi-table DELETE
DELETE orders
FROM orders
JOIN customers ON orders.customer_id = customers.customer_id
WHERE customers.status = 'Inactive';

-- SQL Server: Equivalent DELETE with FROM
DELETE orders
FROM orders
JOIN customers ON orders.customer_id = customers.customer_id
WHERE customers.status = 'Inactive';
```

```sql
-- Subquery approach (works in all databases)
DELETE FROM orders
WHERE customer_id IN (
    SELECT customer_id FROM customers WHERE status = 'Inactive'
);
```

**Real-World Cases:**

- **E-commerce:** Deleting orders for customers who have been banned.
- **CRM:** Deleting contacts associated with inactive companies.
- **Inventory:** Deleting products from discontinued categories.
- **Logging:** Deleting log entries for users who have been deleted.

**References:**

- PostgreSQL: DELETE - https://www.postgresql.org/docs/14/sql-delete.html
- MySQL: DELETE Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/delete.html
- Microsoft Learn: DELETE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/delete-transact-sql


### 3. Safety & Data Integrity

#### 3.1 Safe Deletion Strategies

**Core Definitions:**

- **Core Definition:** Safe deletion strategies are techniques used to verify which rows will be affected before executing a DELETE, and to test changes in a controlled manner.
- **Technical Definition:** Safe deletion practices include using SELECT statements to preview affected rows, wrapping deletions in transactions that can be rolled back, and using database-specific safe-update modes.
- **Beginner-Friendly Explanation:** Before you delete anything, look at what you're about to delete. Run a SELECT first, then run the DELETE inside a transaction so you can undo it if something goes wrong.

**Purposes:**

- To prevent accidental data loss
- To verify the scope of a DELETE before executing it
- To provide a recovery mechanism if the deletion produces unexpected results
- To comply with change management and auditing requirements

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- Step 1: Verify with SELECT
SELECT * FROM table_name WHERE condition;

-- Step 2: Begin transaction
BEGIN TRANSACTION;  -- or START TRANSACTION in MySQL

-- Step 3: Execute DELETE
DELETE FROM table_name WHERE condition;

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

- The SELECT statement should use the same WHERE clause as the DELETE.
- The transaction should include both the DELETE and the verification SELECT.
- Rollback is only possible if the transaction has not been committed.

**Constraints and Limitations:**

- Some databases auto-commit each statement by default (e.g., MySQL with autocommit=1).
- DDL statements (e.g., ALTER TABLE) may implicitly commit transactions.
- Long-running transactions can cause locking and performance issues.
- MySQL's TRUNCATE implicitly commits and cannot be rolled back.

**Annotated Code Examples:**

```sql
-- Setup
CREATE TABLE inventory (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    quantity INT,
    discontinued BOOLEAN
);

INSERT INTO inventory VALUES (1, 'Widget', 100, FALSE);
INSERT INTO inventory VALUES (2, 'Gadget', 0, TRUE);
INSERT INTO inventory VALUES (3, 'Gizmo', 50, FALSE);
INSERT INTO inventory VALUES (4, 'Doohickey', 0, TRUE);

-- Step 1: Verify which rows will be deleted
SELECT * FROM inventory WHERE discontinued = TRUE;

-- Step 2: Begin transaction
BEGIN TRANSACTION;

-- Step 3: Delete discontinued products
DELETE FROM inventory WHERE discontinued = TRUE;

-- Step 4: Verify
SELECT * FROM inventory;

-- Step 5: Commit if correct
COMMIT;
```

**Expected Output (after COMMIT):**

| product_id | product_name | quantity | discontinued |
|------------|--------------|----------|--------------|
| 1          | Widget       | 100      | FALSE        |
| 3          | Gizmo        | 50       | FALSE        |

**Explanation:** The SELECT before the DELETE confirms that Gadget and Doohickey will be affected. The transaction allows verification before committing.

**Real-World Cases:**

- **Production databases:** Always test DELETE operations in a transaction before committing.
- **Data migrations:** Verify row counts before and after bulk deletions.
- **Financial systems:** Use transactions to ensure atomicity of multi-step deletions.
- **Compliance:** Maintain an audit trail of what was deleted and when.

**References:**

- PostgreSQL: Transactions - https://www.postgresql.org/docs/current/tutorial-transactions.html
- MySQL: START TRANSACTION - https://dev.mysql.com/doc/refman/8.0/en/commit.html


#### 3.2 Handling Constraint Violations

**Core Definitions:**

- **Core Definition:** Constraint violations occur when a DELETE statement attempts to remove a row that is referenced by a FOREIGN KEY constraint, or when the deletion would violate referential integrity.
- **Technical Definition:** When a DELETE would violate a FOREIGN KEY constraint with NO ACTION (the default), the database raises an error and rolls back the statement. The error code is typically SQLSTATE 23000.
- **Beginner-Friendly Explanation:** If you try to delete a parent record that still has child records referencing it, the database refuses to delete it and gives you an error.

**Purposes:**

- To maintain referential integrity
- To prevent orphaned records
- To enforce business rules at the database level
- To ensure that deletions do not corrupt relationships between tables

**Syntax Structures and Rules:**

**Common Constraint Types:**

| Constraint | Violation Scenario | Error Code |
|------------|-------------------|------------|
| FOREIGN KEY (NO ACTION) | Deleting a parent row with existing child rows | 23000 |
| FOREIGN KEY (RESTRICT) | Deleting a parent row with existing child rows | 23000 |
| CHECK | Deleting a row that would violate a check condition | 23000 |

**Syntax Rules:**

- FOREIGN KEY constraints with NO ACTION or RESTRICT prevent deletion of parent rows if child rows exist.
- FOREIGN KEY constraints with CASCADE or SET NULL allow deletion and handle child rows automatically.
- Deferred constraints (in some databases) check integrity at the end of the transaction.

**Constraints and Limitations:**

- Some databases allow disabling constraints temporarily.
- MySQL's IGNORE keyword causes constraint violations to be warnings instead of errors.
- SQL Server's OUTPUT clause returns the rows affected before constraint violations.

**Annotated Code Examples:**

```sql
-- Setup
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(100)
);

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100),
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
        ON DELETE NO ACTION  -- Default behavior
);

INSERT INTO departments VALUES (1, 'Engineering');
INSERT INTO departments VALUES (2, 'Marketing');

INSERT INTO employees VALUES (1, 'Alice', 1);
INSERT INTO employees VALUES (2, 'Bob', 1);
INSERT INTO employees VALUES (3, 'Charlie', 2);

-- Attempt to delete a department with employees
DELETE FROM departments WHERE dept_id = 1;
-- Error: The DELETE statement conflicted with the REFERENCE constraint

-- Valid delete (no employees in dept 2)
DELETE FROM departments WHERE dept_id = 2;
```

**Expected Output:**

| Statement | Result |
|-----------|--------|
| Delete dept 1 | Error 23000 (FK violation) |
| Delete dept 2 | Success (Charlie's dept_id becomes NULL if SET NULL, or error if NO ACTION) |

**Explanation:** Deleting Engineering fails because Alice and Bob reference it. Deleting Marketing succeeds because Charlie references it—but only if the FK allows it (NO ACTION would also block this).

**Real-World Cases:**

- **HR systems:** Preventing deletion of a department that still has employees.
- **E-commerce:** Preventing deletion of a customer with active orders.
- **CMS:** Preventing deletion of a category with published articles.
- **Inventory:** Preventing deletion of a product with pending orders.

**References:**

- Microsoft Learn: CREATE TABLE (Transact-SQL) - FOREIGN KEY - https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql
- MySQL: FOREIGN KEY Constraints - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/create-table-foreign-keys.html


#### 3.3 Safe Delete Modes

**Core Definitions:**

- **Core Definition:** Safe delete modes are database configuration settings that prevent DELETE statements without a key-based WHERE clause or LIMIT clause from executing.
- **Technical Definition:** MySQL's `sql_safe_updates` system variable, when enabled, causes DELETE (and UPDATE) statements that do not use a key in the WHERE clause or a LIMIT clause to produce an error.
- **Beginner-Friendly Explanation:** Safe delete mode is like a safety lock on a power tool—it prevents you from accidentally deleting everything.

**Purposes:**

- To prevent accidental mass deletions
- To enforce deliberate, verified deletion operations
- To catch missing WHERE clauses before execution
- To protect against SQL injection and application bugs

**Syntax Structures and Rules:**

**MySQL Safe Updates Mode:**

```sql
-- Enable safe updates (session-level)
SET sql_safe_updates = 1;

-- Enable safe updates (global)
SET GLOBAL sql_safe_updates = 1;

-- Check current value
SELECT @@sql_safe_updates;

-- Disable safe updates
SET sql_safe_updates = 0;
```

**Component Breakdown:**

- `sql_safe_updates`: MySQL system variable (boolean, default OFF)
- When enabled, DELETE statements must use a key in the WHERE clause or a LIMIT clause
- The mysql client can enable it with `--safe-updates` option

**Syntax Rules:**

- Safe updates mode requires the WHERE clause to use a key column (PRIMARY KEY or UNIQUE index) or a LIMIT clause.
- The mode also restricts SELECT statements that produce very large result sets.
- Safe updates mode is disabled by default.

**Constraints and Limitations:**

- Safe updates mode is MySQL-specific.
- Even with safe updates, a well-crafted WHERE clause with a non-unique key can still delete many rows.
- Some databases (e.g., PostgreSQL, SQL Server) do not have a built-in safe update mode.

**Annotated Code Examples:**

```sql
-- MySQL: Enable safe updates
SET sql_safe_updates = 1;

-- This will FAIL because there's no WHERE clause
DELETE FROM inventory;
-- Error: You are using safe update mode and you tried to update a table
-- without a WHERE that uses a KEY column

-- This will SUCCEED because WHERE uses the primary key
DELETE FROM inventory WHERE product_id = 3;

-- This will SUCCEED because of the LIMIT clause
DELETE FROM inventory LIMIT 1;

-- Disable safe updates
SET sql_safe_updates = 0;
```

**Real-World Cases:**

- **Production databases:** Always enable safe updates or use transactions.
- **Development environments:** Enable safe updates to catch mistakes early.
- **Database migration scripts:** Include explicit WHERE clauses and verification steps.

**References:**

- MySQL: Safe Updates Mode - https://downloads.mysql.com/docs/refman-5.6-en.a4.pdf


### 4. Data Recovery & Architecture Standards

#### 4.1 Soft Deletes vs. Hard Deletes

**Core Definitions:**

- **Core Definition:** A hard delete physically removes rows from the table, while a soft delete marks rows as deleted using a flag (e.g., `is_deleted`) or timestamp (e.g., `deleted_at`) without removing the data.
- **Technical Definition:** Soft deletes preserve data for auditing, recovery, and historical analysis. Applications must filter out soft-deleted rows in queries using `WHERE deleted_at IS NULL` or `WHERE is_deleted = FALSE`.
- **Beginner-Friendly Explanation:** A hard delete is like throwing a document in the trash. A soft delete is like marking it "deleted" but keeping it in a folder—you can still find it later if needed.

**Purposes:**

- To preserve data for auditing and compliance
- To enable data recovery after accidental deletion
- To maintain referential integrity with historical data
- To support undo functionality in applications

**Syntax Structures and Rules:**

**Soft Delete Pattern (is_deleted flag):**

```sql
-- Add soft delete column
ALTER TABLE table_name ADD COLUMN is_deleted BOOLEAN DEFAULT FALSE;

-- Soft delete
UPDATE table_name SET is_deleted = TRUE WHERE condition;

-- Query active records
SELECT * FROM table_name WHERE is_deleted = FALSE;
```

**Soft Delete Pattern (deleted_at timestamp):**

```sql
-- Add soft delete column
ALTER TABLE table_name ADD COLUMN deleted_at TIMESTAMP NULL;

-- Soft delete
UPDATE table_name SET deleted_at = NOW() WHERE condition;

-- Query active records
SELECT * FROM table_name WHERE deleted_at IS NULL;
```

**Component Breakdown:**

- `is_deleted`: Boolean flag indicating whether the row is deleted
- `deleted_at`: Timestamp indicating when the row was deleted
- Both approaches require application-level filtering

**Syntax Rules:**

- Soft-deleted rows remain in the table and count toward table size.
- Unique constraints may need to be adjusted to allow re-use of values.
- Foreign key relationships still reference soft-deleted rows.

**Constraints and Limitations:**

- Soft deletes consume storage space for deleted records.
- Queries must consistently filter out soft-deleted rows.
- Unique constraints (e.g., email) may conflict when re-inserting deleted values.
- Performance may degrade as the table grows with soft-deleted rows.

**Annotated Code Examples:**

```sql
-- Setup
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(100),
    email VARCHAR(200),
    is_deleted BOOLEAN DEFAULT FALSE,
    deleted_at TIMESTAMP NULL
);

INSERT INTO users VALUES (1, 'alice', 'alice@example.com', FALSE, NULL);
INSERT INTO users VALUES (2, 'bob', 'bob@example.com', FALSE, NULL);
INSERT INTO users VALUES (3, 'charlie', 'charlie@example.com', FALSE, NULL);

-- Soft delete Bob
UPDATE users
SET is_deleted = TRUE,
    deleted_at = NOW()
WHERE user_id = 2;

-- Query active users only
SELECT * FROM users WHERE is_deleted = FALSE;
```

**Expected Output:**

| user_id | username | email             | is_deleted | deleted_at          |
|---------|----------|-------------------|------------|---------------------|
| 1       | alice    | alice@example.com | FALSE      | NULL                |
| 3       | charlie  | charlie@example.com | FALSE    | NULL                |

**Explanation:** Bob is soft-deleted and does not appear in the active users query, but his data remains in the table.

**Real-World Cases:**

- **User management:** Soft-deleting user accounts to allow recovery.
- **E-commerce:** Soft-deleting products while preserving order history.
- **CMS:** Soft-deleting articles while maintaining audit trails.
- **Financial systems:** Soft-deleting transactions for regulatory compliance.

**References:**

- Microsoft Learn: Soft Delete Pattern - https://learn.microsoft.com/en-us/azure/architecture/patterns/soft-delete


#### 4.2 Retrieving Deleted Data (RETURNING / OUTPUT Clause)

**Core Definitions:**

- **Core Definition:** The RETURNING (PostgreSQL, Oracle) and OUTPUT (SQL Server) clauses return values from rows affected by a DELETE statement, eliminating the need for a separate SELECT.
- **Technical Definition:** The optional RETURNING clause causes DELETE to compute and return value(s) based on each row actually deleted. The default data available is the content of the deleted row. SQL Server's OUTPUT clause provides similar functionality using the DELETED pseudo-table.
- **Beginner-Friendly Explanation:** After you delete rows, you often want to see what was deleted. Instead of running a separate SELECT, you can ask the DELETE statement to return the deleted rows.

**Purposes:**

- To retrieve deleted row data in a single round trip
- To capture deleted values for auditing
- To archive deleted rows into a history table
- To feed deleted values into subsequent operations

**Syntax Structures and Rules:**

**PostgreSQL / Oracle (RETURNING):**

```sql
DELETE FROM table_name
WHERE condition
RETURNING * | output_expression [ [ AS ] output_name ] [, ...];
```

**SQL Server (OUTPUT):**

```sql
DELETE FROM table_name
OUTPUT DELETED.column_name [, ...]
WHERE condition;
```

**Component Breakdown:**

- `RETURNING *`: Returns all columns of the deleted rows
- `RETURNING expression`: Returns a specific expression or column
- `DELETED.column`: The deleted row values in SQL Server
- `INTO table_variable`: Stores the output in a table variable (SQL Server)

**Syntax Rules:**

- RETURNING can use any expression involving the table's columns.
- In SQL Server, OUTPUT can reference the DELETED pseudo-table.
- The RETURNING list syntax is identical to the SELECT output list.
- Oracle's RETURNING INTO clause requires PL/SQL variables to receive the values.

**Constraints and Limitations:**

- RETURNING cannot be used with multi-table DELETE in MySQL.
- SQL Server's OUTPUT clause cannot return values if the DELETE statement has a FROM clause with certain join types.
- The RETURNING clause returns values after triggers have fired.

**Annotated Code Examples:**

```sql
-- PostgreSQL: RETURNING example
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10,2),
    discontinued BOOLEAN
);

INSERT INTO products VALUES (1, 'Widget', 19.99, FALSE);
INSERT INTO products VALUES (2, 'Gadget', 29.99, TRUE);
INSERT INTO products VALUES (3, 'Gizmo', 9.99, TRUE);

-- Delete discontinued products and return them
DELETE FROM products
WHERE discontinued = TRUE
RETURNING product_id, product_name, price;
```

**Expected Output:**

| product_id | product_name | price |
|------------|--------------|-------|
| 2          | Gadget       | 29.99 |
| 3          | Gizmo        | 9.99  |

**Explanation:** The RETURNING clause shows the deleted rows in a single statement.

```sql
-- SQL Server: OUTPUT example
DELETE FROM products
OUTPUT DELETED.product_id, DELETED.product_name, DELETED.price
WHERE discontinued = 1;
```

**Expected Output (SQL Server):** Same as above.

```sql
-- Archiving deleted rows (PostgreSQL)
INSERT INTO product_archive
SELECT * FROM products WHERE discontinued = TRUE
RETURNING *;
-- Or more practically:
WITH deleted AS (
    DELETE FROM products
    WHERE discontinued = TRUE
    RETURNING *
)
INSERT INTO product_archive
SELECT * FROM deleted;
```

**Real-World Cases:**

- **Auditing:** Capturing deleted rows for audit trails.
- **Archiving:** Moving deleted rows to a history table.
- **Application development:** Getting deleted values without a second query.
- **Data synchronization:** Feeding deleted values into other systems.

**References:**

- PostgreSQL: Returning Data from Modified Rows - https://www.postgresql.org/docs/19/dml-returning.html
- Microsoft Learn: OUTPUT Clause - https://learn.microsoft.com/en-us/sql/t-sql/queries/output-clause-transact-sql


## Summary Tables

### DELETE Syntax Comparison Across Databases

| Feature | PostgreSQL | SQL Server | MySQL | Oracle |
|---------|-----------|------------|-------|--------|
| Basic DELETE | ✓ | ✓ | ✓ | ✓ |
| DELETE ... USING | ✓ | ✗ | ✗ | ✗ |
| Multi-table DELETE | ✗ | ✗ | ✓ | ✗ |
| DELETE ... FROM (join) | ✗ | ✓ | ✗ | ✗ |
| RETURNING clause | ✓ | ✗ | ✗ | ✓ |
| OUTPUT clause | ✗ | ✓ | ✗ | ✗ |
| LIMIT clause | ✗ | ✗ | ✓ | ✗ (use ROWNUM) |
| ORDER BY in DELETE | ✗ | ✗ | ✓ | ✗ |
| TOP clause | ✗ | ✓ | ✗ | ✗ |
| Safe updates mode | ✗ | ✗ | ✓ | ✗ |

### DELETE vs. TRUNCATE Comparison

| Feature | DELETE | TRUNCATE |
|---------|--------|----------|
| WHERE clause | ✓ | ✗ |
| Transaction log | Row-by-row | Page deallocations |
| Triggers | Fire | Do not fire |
| Identity reset | ✗ | ✓ |
| Rollback | ✓ (in transaction) | Varies by database |
| FK constraints | Works with CASCADE | Cannot be used |
| Speed | Slower | Faster |
| Privilege | DELETE | ALTER (SQL Server) / DROP (MySQL) |

### ON DELETE Actions Comparison

| Action | Behavior | Requirements |
|--------|----------|--------------|
| NO ACTION | Raises error, rolls back | None (default) |
| CASCADE | Deletes child rows | None |
| SET NULL | Sets FK to NULL | FK columns must be nullable |
| SET DEFAULT | Sets FK to default | FK columns must have defaults |

---

## References

- PostgreSQL: DELETE - https://www.postgresql.org/docs/14/sql-delete.html
- PostgreSQL: Returning Data from Modified Rows - https://www.postgresql.org/docs/19/dml-returning.html
- PostgreSQL: Transactions - https://www.postgresql.org/docs/current/tutorial-transactions.html
- MySQL: DELETE Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/delete.html
- MySQL: FOREIGN KEY Constraints - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/create-table-foreign-keys.html
- MySQL: TRUNCATE TABLE Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/truncate-table.html
- MySQL: Safe Updates Mode - https://downloads.mysql.com/docs/refman-5.6-en.a4.pdf
- MySQL: START TRANSACTION - https://dev.mysql.com/doc/refman/8.0/en/commit.html
- Microsoft Learn: DELETE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/delete-transact-sql
- Microsoft Learn: TRUNCATE TABLE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/truncate-table-transact-sql
- Microsoft Learn: CREATE TABLE (Transact-SQL) - FOREIGN KEY - https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql
- Microsoft Learn: OUTPUT Clause - https://learn.microsoft.com/en-us/sql/t-sql/queries/output-clause-transact-sql
- Microsoft Learn: MERGE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql
- Microsoft Learn: Soft Delete Pattern - https://learn.microsoft.com/en-us/azure/architecture/patterns/soft-delete
- Oracle: DELETE Statement - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/DELETE.html
- Oracle: RETURNING INTO Clause - https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/RETURNING-INTO-clause.html