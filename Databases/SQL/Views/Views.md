# SQL Views: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A SQL view is a named, stored query that acts as a virtual table, allowing users to interact with the result of a `SELECT` statement as if it were a physical table.

**Technical Definition:** Per the SQL standard (ISO/IEC 9075-2), a view is a table whose contents are defined by a `<query expression>` stored in the database catalog. A view is a virtual table: it has a name, a set of named columns, and a defining query, but it does not physically store data (unless it is a materialized view). To the parser, a view is a relation, just like a base table. When a query references a view, the database performs view resolution, translating the query over the view into an equivalent query over the underlying base tables.

**Beginner-Friendly Explanation:** A view is like a saved query that you can use like a table. Instead of writing the same complex `JOIN` every time, you save it as a view and then simply `SELECT * FROM my_view`. The view doesn't store data itself—it just remembers the query and runs it whenever you ask for data.

### Key Characteristics

- **Virtual table:** Appears as a table to users but does not store data (for regular views).
- **Named query:** The view definition is a stored `SELECT` statement.
- **Data abstraction:** Hides complexity of underlying tables and joins.
- **Security mechanism:** Can restrict access to specific rows and columns.
- **Logical independence:** Insulates applications from schema changes in base tables.
- **Updatability:** Some views are updatable (allow `INSERT`, `UPDATE`, `DELETE`); others are read-only.
- **Materialization option:** Materialized views physically store query results for performance.
- **Dialect variations:** Syntax for `CREATE VIEW`, `OR REPLACE`, `WITH CHECK OPTION`, and materialized views varies across RDBMSs.

### Prerequisites

- **Basic SQL knowledge:** Familiarity with `SELECT`, `FROM`, `WHERE`, `JOIN`, and aggregate functions.
- **Schema understanding:** Knowledge of the tables and relationships in the database.
- **Privilege awareness:** Understanding of `GRANT` and `REVOKE` for access control.
- **Transaction concepts:** Basic understanding of how views interact with DML operations.

### Related Programming Areas

- **Data Security and Access Control:** Views enforce column-level and row-level security.
- **Application Development:** Views provide a stable interface for application queries.
- **Business Intelligence and Reporting:** Views encapsulate complex reporting logic.
- **Data Warehousing:** Materialized views pre-compute expensive aggregations.
- **Database Administration:** DBAs use views for schema abstraction and simplification.

### Core Concepts / Features

1. View Definition and Purpose
2. View Types: Simple, Complex, and Materialized
3. CREATE VIEW Syntax
4. Updatable Views and WITH CHECK OPTION
5. View Resolution (How Views Are Processed)
6. Security Through Views
7. Materialized Views and REFRESH
8. Views vs. CTEs vs. Derived Tables
9. Dropping and Altering Views
10. Performance Considerations

---

## 1. View Definition and Purpose

### Definitions

**Core Definition:** A view is a named query stored in the database that presents data from one or more base tables as a single virtual table.

**Technical Definition:** A view is defined by a `CREATE VIEW` statement containing a `<query expression>` (the defining query). The view's columns are derived from the select list of that query. The view name and column names are stored in the system catalog, but the view does not contain data of its own (for non-materialized views). When a query references the view, the optimizer resolves the view by substituting the defining query into the referencing query.

**Beginner-Friendly Explanation:** A view is a way to save a query and give it a name. It's like creating a shortcut for a complex `SELECT` statement. Once created, you can query the view as if it were a real table, and the database will run the underlying query for you.

### Purposes

- To simplify complex queries by encapsulating joins, filters, and aggregations.
- To provide a stable interface that insulates applications from underlying schema changes.
- To restrict access to sensitive data by exposing only specific rows or columns.
- To present data in a customised, business-friendly format.
- To reuse common query logic across multiple queries and applications.
- To provide a consistent data view even if the underlying tables are restructured.

### Syntax Rules and Structure

**Complete General Syntax (Standard SQL):**

```sql
CREATE [OR REPLACE] VIEW view_name [(column1, column2, ...)]
AS select_statement
[WITH [CASCADED | LOCAL] CHECK OPTION];
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `CREATE VIEW` | Initiates view creation |
| `OR REPLACE` | Replaces an existing view of the same name (dialect-specific) |
| `view_name` | The name of the view |
| `(column1, ...)` | Optional explicit column names for the view |
| `AS select_statement` | The defining query |
| `WITH CHECK OPTION` | Restricts inserts/updates through the view |

**Syntax Rules:**

- The defining query can reference base tables, other views, and (in some dialects) CTEs.
- If column names are not specified, the view inherits the names from the select list.
- Column names in the view must be unique.
- The `WITH CHECK OPTION` clause is only valid for updatable views.
- The view definition is "frozen" at creation time: changes to base table schemas after creation are not reflected in the view definition.

**Constraints and Limitations:**

- A view cannot reference itself or create a circular dependency.
- The `SELECT` statement in the view cannot contain `ORDER BY` unless it also has a `LIMIT`/`TOP` (varies by RDBMS).
- Views cannot have indexes directly (except indexed views in SQL Server).
- In MySQL, a view's `SELECT` statement cannot reference system variables or user variables.

### Annotated Complete Code Examples

**Example 1: Basic View Creation**

```sql
-- Setup: Create employees and departments tables
CREATE TABLE departments (
    dept_id   INTEGER PRIMARY KEY,
    dept_name VARCHAR(50)
);

CREATE TABLE employees (
    emp_id    INTEGER PRIMARY KEY,
    emp_name  VARCHAR(50),
    dept_id   INTEGER,
    salary    NUMERIC(10, 2)
);

INSERT INTO departments VALUES (10, 'Engineering'), (20, 'Marketing');
INSERT INTO employees VALUES
    (1, 'Alice', 10, 95000.00),
    (2, 'Bob',   20, 72000.00),
    (3, 'Carol', 10, 105000.00);

-- Create a view that joins employees and departments
CREATE VIEW employee_details AS
SELECT e.emp_id,
       e.emp_name,
       d.dept_name,
       e.salary
FROM employees e
INNER JOIN departments d
    ON e.dept_id = d.dept_id;

-- Query the view like a table
SELECT * FROM employee_details;

-- Expected Output:
--  emp_id | emp_name | dept_name   |  salary
-- --------+----------+-------------+----------
--       1 | Alice    | Engineering | 95000.00
--       2 | Bob      | Marketing   | 72000.00
--       3 | Carol    | Engineering |105000.00
```

**Why this output occurs:** The view `employee_details` stores the join query. When queried, the database resolves the view by executing the underlying join, returning the combined result. The user does not need to know about the `employees` or `departments` tables.

**Example 2: View with Column Aliases**

```sql
-- Create a view with explicit column names
CREATE VIEW staff (Number, Name, Job) AS
SELECT emp_id, emp_name, dept_name
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id;

-- Query the view
SELECT Name, Job FROM staff;

-- Expected Output:
--  name  | job
-- -------+-------------
--  Alice | Engineering
--  Bob   | Marketing
--  Carol | Engineering
```

**Why this output occurs:** The view `staff` renames the columns to `Number`, `Name`, and `Job`. The underlying query still references the original column names, but the view presents them under the new names.

### Real-World Cases

**Case 1: Simplifying Complex Reporting Queries**

A BI team creates a view that joins five tables and applies business rules. Report writers simply query the view by name, avoiding the need to understand the underlying schema.

**Case 2: Providing a Stable Interface**

An application queries a view instead of base tables. When the DBA refactors the schema (e.g., splitting a table), the view is updated to preserve the original column names and structure, so the application continues to work without changes.

### References

- MySQL 8.0 Reference Manual — CREATE VIEW Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/create-view.html
- PostgreSQL Documentation — CREATE VIEW - https://www.postgresql.org/docs/current/sql-createview.html
- Microsoft Learn — CREATE VIEW (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/create-view-transact-sql
- Oracle Database SQL Language Reference — CREATE VIEW - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CREATE-VIEW.html

---

## 2. View Types: Simple, Complex, and Materialized

### Definitions

**Core Definition:** Views are classified into three main types: simple views (based on a single table), complex views (based on multiple tables with joins/aggregations), and materialized views (physically storing query results).

**Technical Definition:** A simple view is based on a single base table and does not contain `GROUP BY`, functions, or `DISTINCT`. A complex view is based on multiple tables and may contain joins, `GROUP BY`, functions, and aggregations. A materialized view physically stores the result of its defining query in the database, unlike regular views which compute results on demand.

**Beginner-Friendly Explanation:** Simple views are based on one table—like a filtered or column-limited version of that table. Complex views combine multiple tables, like a saved join. Materialized views are special because they actually save the query result to disk, making them faster but requiring periodic refreshes.

### Purposes

**Simple Views:**
- To hide specific columns or rows from certain users.
- To provide a simplified version of a single table.
- To create updatable views for data entry forms.

**Complex Views:**
- To join multiple tables and present a unified view of related data.
- To pre-aggregate data for reporting.
- To encapsulate complex business logic.

**Materialized Views:**
- To dramatically improve query performance for expensive aggregations.
- To provide fast access to pre-computed results.
- To reduce load on base tables for read-heavy workloads.
- To support data warehousing and OLAP scenarios.

### Syntax Rules and Structure

**Simple View:**

```sql
CREATE VIEW simple_view AS
SELECT column1, column2
FROM single_table
WHERE condition;
```

**Complex View:**

```sql
CREATE VIEW complex_view AS
SELECT t1.col1, t2.col2, SUM(t1.amount) AS total
FROM table1 t1
JOIN table2 t2 ON t1.id = t2.id
GROUP BY t1.col1, t2.col2;
```

**Materialized View (PostgreSQL):**

```sql
CREATE MATERIALIZED VIEW matview AS
SELECT seller_no, invoice_date, SUM(invoice_amt) AS sales_amt
FROM invoice
WHERE invoice_date < CURRENT_DATE
GROUP BY seller_no, invoice_date;
```

**Component Breakdown:**

| View Type | Based On | Contains | Updatable? | Stores Data? |
|-----------|----------|----------|------------|-------------|
| Simple | Single table | No GROUP BY, functions | Yes (usually) | No |
| Complex | Multiple tables | Joins, GROUP BY, functions | No (usually) | No |
| Materialized | Any query | Any | No | Yes (physical storage) |

**Syntax Rules:**

- Simple views are typically updatable; complex views are generally not.
- Materialized views are created with `CREATE MATERIALIZED VIEW` (PostgreSQL) or `CREATE SNAPSHOT` (Oracle).
- Materialized views can have indexes, unlike regular views.
- Materialized views require explicit `REFRESH` to update their data.

**Constraints and Limitations:**

- Simple views are updatable only if they meet specific criteria (single table, no `DISTINCT`, no derived columns, etc.).
- Materialized views are not supported in all RDBMSs (MySQL does not support them natively).
- Materialized views may become stale and require refresh scheduling.

### Annotated Complete Code Examples

**Example 1: Simple View**

```sql
-- Simple view: single table, no aggregation
CREATE VIEW active_employees AS
SELECT emp_id, emp_name, salary
FROM employees
WHERE salary > 70000;

SELECT * FROM active_employees;

-- Expected Output:
--  emp_id | emp_name |  salary
-- --------+----------+----------
--       1 | Alice    | 95000.00
--       3 | Carol    |105000.00
```

**Example 2: Complex View**

```sql
-- Complex view: multiple tables, aggregation
CREATE VIEW dept_salary_summary AS
SELECT d.dept_name,
       COUNT(e.emp_id) AS headcount,
       AVG(e.salary) AS avg_salary,
       SUM(e.salary) AS total_salary
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
GROUP BY d.dept_name;

SELECT * FROM dept_salary_summary;

-- Expected Output:
--  dept_name   | headcount |     avg_salary      | total_salary
-- -------------+-----------+---------------------+-------------
--  Engineering |         2 | 100000.000000000000 |    200000.00
--  Marketing   |         1 |  72000.000000000000 |     72000.00
```

**Example 3: Materialized View (PostgreSQL)**

```sql
-- Materialized view: stores results physically
CREATE MATERIALIZED VIEW sales_summary AS
SELECT seller_no,
       invoice_date,
       SUM(invoice_amt)::numeric(13,2) AS sales_amt
FROM invoice
WHERE invoice_date < CURRENT_DATE
GROUP BY seller_no, invoice_date;

-- Query the materialized view
SELECT * FROM sales_summary;

-- Refresh the materialized view when data changes
REFRESH MATERIALIZED VIEW sales_summary;
```

**Why materialized views are useful:** The materialized view stores the aggregated sales data on disk. Querying it is much faster than running the aggregation over the `invoice` table each time. The `REFRESH` command regenerates the stored data.

### Real-World Cases

**Case 1: Simple View for Data Entry**

A data entry application uses a simple view that includes only active employees, preventing users from seeing or modifying terminated employee records.

**Case 2: Complex View for Reporting**

A monthly sales report uses a complex view that joins orders, order lines, products, and customers, and computes revenue by category.

**Case 3: Materialized View for Dashboard**

A real-time dashboard uses a materialized view that pre-aggregates sales by hour. The view is refreshed every 15 minutes, providing near-real-time data with fast query response.

### References

- Baeldung — Databases: Simple vs. Complex vs. Materialized Views - https://www.baeldung.com/sql/databases-views-simple-complex-materialized
- PostgreSQL Documentation — Materialized Views - https://www.postgresql.org/docs/current/rules-materializedviews.html
- Oracle Database — Materialized Views - https://docs.oracle.com/en/database/oracle/oracle-database/21/dwhsg/

---

## 3. CREATE VIEW Syntax

### Definitions

**Core Definition:** `CREATE VIEW` is the SQL statement used to define a new view in the database, specifying its name, optional column names, and the defining query.

**Technical Definition:** `CREATE VIEW view_name [(column_list)] AS select_statement` creates a new view. If `OR REPLACE` is specified, an existing view of the same name is replaced. The `ALGORITHM` clause (MySQL) controls whether the view is merged or materialized. The `DEFINER` and `SQL SECURITY` clauses (MySQL) specify the security context for privilege checking.

**Beginner-Friendly Explanation:** `CREATE VIEW` is how you create a view. You give it a name, optionally name its columns, and provide the query that defines what the view shows. If you use `OR REPLACE`, it updates an existing view instead of failing.

### Purposes

- To create a new view with a specified name and defining query.
- To replace an existing view definition without dropping it first (`OR REPLACE`).
- To specify column names for the view.
- To control the security context for privilege checking (`DEFINER`, `SQL SECURITY`).
- To influence how the view is processed (`ALGORITHM` in MySQL).

### Syntax Rules and Structure

**Complete General Syntax (MySQL):**

```sql
CREATE [OR REPLACE]
    [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
    [DEFINER = user]
    [SQL SECURITY {DEFINER | INVOKER}]
    VIEW view_name [(column_list)]
    AS select_statement
    [WITH [CASCADED | LOCAL] CHECK OPTION];
```

**Complete General Syntax (PostgreSQL):**

```sql
CREATE [OR REPLACE] [TEMP | TEMPORARY] VIEW view_name [(column_list)]
AS select_statement;
```

**Component Breakdown:**

| Component | Purpose | Dialect |
|-----------|---------|---------|
| `OR REPLACE` | Replace existing view | MySQL, PostgreSQL, Oracle |
| `ALGORITHM` | Merge vs. temptable processing | MySQL |
| `DEFINER` | Security context | MySQL |
| `SQL SECURITY` | Privilege checking mode | MySQL |
| `column_list` | Explicit column names | All |
| `AS select_statement` | Defining query | All |
| `WITH CHECK OPTION` | Restrict DML through view | MySQL, PostgreSQL, Oracle |

**Syntax Rules:**

- `OR REPLACE` creates the view if it doesn't exist, or replaces it if it does.
- The `column_list` must have the same number of columns as the select list.
- If `column_list` is omitted, column names come from the select list.
- `ALGORITHM = MERGE` attempts to merge the view into the outer query; `TEMPTABLE` materializes it.
- `DEFINER` specifies which user's privileges are used for access control.
- `SQL SECURITY DEFINER` uses the definer's privileges; `INVOKER` uses the invoker's.

**Constraints and Limitations:**

- `OR REPLACE` requires `DROP` privilege on the existing view (MySQL).
- `ALGORITHM = MERGE` is not possible if the view uses aggregation, `DISTINCT`, `GROUP BY`, etc.
- PostgreSQL supports `CREATE OR REPLACE VIEW`, but the new query must generate the same columns (same names, types, order) as the existing view.
- SQL Server does not support `OR REPLACE`; it uses `CREATE OR ALTER VIEW` (SQL Server 2016 SP1+).

### Annotated Complete Code Examples

**Example 1: CREATE OR REPLACE VIEW (MySQL)**

```sql
-- Create a view
CREATE VIEW employee_view AS
SELECT emp_id, emp_name, salary FROM employees;

-- Replace the view with a new definition
CREATE OR REPLACE VIEW employee_view AS
SELECT emp_id, emp_name, salary, dept_id FROM employees;

-- Verify
SELECT * FROM employee_view LIMIT 1;
```

**Why this works:** `CREATE OR REPLACE` drops the old view definition and creates a new one with the same name, without requiring a separate `DROP VIEW` statement.

**Example 2: CREATE VIEW with ALGORITHM (MySQL)**

```sql
-- Force merging (view is inlined into outer query)
CREATE ALGORITHM = MERGE VIEW simple_view AS
SELECT emp_id, emp_name FROM employees WHERE salary > 50000;

-- Force materialization (view is computed as temporary table)
CREATE ALGORITHM = TEMPTABLE VIEW aggregated_view AS
SELECT dept_id, AVG(salary) AS avg_sal
FROM employees
GROUP BY dept_id;
```

**Why the difference matters:** Merging avoids the overhead of a temporary table and allows the outer query's conditions to be pushed down. Materialization is necessary when merging is not possible (e.g., with aggregation).

### Real-World Cases

**Case 1: Deploying View Updates**

A DevOps pipeline uses `CREATE OR REPLACE VIEW` to deploy updated view definitions without dropping and recreating them, ensuring continuous availability.

**Case 2: Security Context Configuration**

A DBA uses `SQL SECURITY DEFINER` so that users can query a view containing sensitive data without having direct privileges on the base tables.

### References

- MySQL 8.0 Reference Manual — CREATE VIEW Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/create-view.html
- PostgreSQL Documentation — CREATE VIEW - https://www.postgresql.org/docs/current/sql-createview.html
- Microsoft Learn — CREATE VIEW (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/create-view-transact-sql
- Oracle Database SQL Language Reference — CREATE VIEW - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CREATE-VIEW.html

---

## 4. Updatable Views and WITH CHECK OPTION

### Definitions

**Core Definition:** An updatable view is a view through which `INSERT`, `UPDATE`, and `DELETE` operations can be performed, modifying the underlying base tables. `WITH CHECK OPTION` restricts these operations to ensure that modified rows remain visible through the view.

**Technical Definition:** A view is updatable if it meets specific criteria defined by the SQL standard: it must involve a single base relation, must not use `DISTINCT`, must not contain derived columns or aggregates, must contain all `NOT NULL` columns of the base relation, and must not have `GROUP BY`, `HAVING`, or nested subqueries in the `WHERE` clause. The `WITH CHECK OPTION` clause prevents inserts and updates that would create rows not visible through the view (i.e., rows that do not satisfy the view's `WHERE` condition).

**Beginner-Friendly Explanation:** An updatable view lets you change data through the view, and the changes are applied to the real table. `WITH CHECK OPTION` is a safety feature: it stops you from inserting or updating a row in a way that would make it disappear from the view.

### Purposes

- To allow data modification through a simplified, filtered view of a table.
- To enforce business rules by restricting what data can be inserted or updated.
- To provide a secure interface for data entry without exposing the full table.
- To prevent "vanishing rows"—rows that are inserted or updated but then cannot be seen through the view.

### Syntax Rules and Structure

**Updatable View Criteria (SQL Standard):**

| Criterion | Requirement |
|-----------|-------------|
| Base tables | Exactly one base table in `FROM` |
| Select list | All columns must be simple column references (no expressions, no aggregates) |
| `DISTINCT` | Not allowed |
| `GROUP BY` / `HAVING` | Not allowed |
| `WHERE` | No nested subqueries (varies by dialect) |
| `NOT NULL` columns | All must be included in the view |

**WITH CHECK OPTION Syntax:**

```sql
CREATE VIEW view_name AS
SELECT ...
FROM table
WHERE condition
WITH [CASCADED | LOCAL] CHECK OPTION;
```

**Component Breakdown:**

| Keyword | Behaviour |
|---------|-----------|
| `WITH CHECK OPTION` | Checks the view's `WHERE` condition |
| `CASCADED` | Checks this view and all underlying views (default) |
| `LOCAL` | Checks only this view's `WHERE` clause, not underlying views |

**Syntax Rules:**

- `WITH CHECK OPTION` is only valid for updatable views.
- Without `CHECK OPTION`, the view's `WHERE` clause is not checked on `INSERT`/`UPDATE`.
- `CASCADED` is the default when neither keyword is specified.
- With `CASCADED`, the check recurses to underlying views and applies the same rules.
- With `LOCAL`, the view's own `WHERE` clause is checked, then checking recurses to underlying views.

**Constraints and Limitations:**

- `WITH CHECK OPTION` is not supported on views that have `INSTEAD OF` triggers.
- Updatable views cannot modify columns not included in the view.
- Complex views (joins, aggregation) are generally not updatable.
- MySQL's `WITH CHECK OPTION` testing is standard-compliant but has specific recursion rules.

### Annotated Complete Code Examples

**Example 1: Updatable View with CHECK OPTION**

```sql
-- Setup: Table
CREATE TABLE t1 (a INT);
INSERT INTO t1 VALUES (1);

-- Create a view with CHECK OPTION
CREATE VIEW v1 AS
SELECT * FROM t1 WHERE a < 2
WITH CHECK OPTION;

-- This works: 1 < 2
INSERT INTO v1 VALUES (1);  -- OK

-- This fails: 2 is not < 2
INSERT INTO v1 VALUES (2);
-- Expected Error (MySQL):
-- ERROR 1369 (HY000): CHECK OPTION failed 'test.v1'
```

**Why this happens:** The view `v1` only shows rows where `a < 2`. The `WITH CHECK OPTION` prevents inserting a row with `a = 2` because it would not be visible through the view.

**Example 2: LOCAL vs. CASCADED CHECK OPTION**

```sql
-- Setup
CREATE TABLE t1 (a INT);

-- Base view with CHECK OPTION
CREATE VIEW v1 AS SELECT * FROM t1 WHERE a < 2 WITH CHECK OPTION;

-- View based on v1 with LOCAL check option
CREATE VIEW v2 AS SELECT * FROM v1 WHERE a > 0 WITH LOCAL CHECK OPTION;

-- View based on v1 with CASCADED check option
CREATE VIEW v3 AS SELECT * FROM v1 WHERE a > 0 WITH CASCADED CHECK OPTION;

-- Insert into v2 (LOCAL): checks v2's WHERE, then recurses to v1
INSERT INTO v2 VALUES (2);
-- ERROR: CHECK OPTION failed 'test.v2'

-- Insert into v3 (CASCADED): checks v3, recurses to v1, adds CASCADED
INSERT INTO v3 VALUES (2);
-- ERROR: CHECK OPTION failed 'test.v3'
```

**Why both fail:** In MySQL, both `LOCAL` and `CASCADED` ultimately check the base view's constraint (`a < 2`). The difference lies in the recursion rules: `CASCADED` explicitly adds `WITH CASCADED CHECK OPTION` to underlying views for checking purposes, while `LOCAL` applies the same rules without modification.

### Real-World Cases

**Case 1: Data Entry Form**

A data entry application uses an updatable view that includes only active employees. `WITH CHECK OPTION` prevents users from accidentally inserting a record that would immediately be invisible (e.g., a terminated employee).

**Case 2: Multi-Tenant Isolation**

A SaaS platform creates per-tenant views with `WITH CHECK OPTION` that filter by `tenant_id`. Inserts through the view automatically enforce the tenant ID, preventing data leakage.

### References

- MySQL 8.0 Reference Manual — The View WITH CHECK OPTION Clause - https://dev.mysql.com/doc/refman/8.0/en/view-check-option.html
- PostgreSQL Documentation — CREATE VIEW (WITH CHECK OPTION) - https://www.postgresql.org/docs/current/sql-createview.html
- SQL Standard — Updatable Views (ISO/IEC 9075-2)
- Microsoft Learn — CREATE VIEW (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/create-view-transact-sql

---

## 5. View Resolution (How Views Are Processed)

### Definitions

**Core Definition:** View resolution is the process by which the database translates a query that references a view into an equivalent query over the underlying base tables.

**Technical Definition:** When a query uses a view, the database performs view resolution: it replaces the view name in the `FROM` clause with the view's defining query, merges the `WHERE`, `GROUP BY`, and `HAVING` clauses, and renames columns as needed. The result is a query over base relations that the optimizer can then optimize.

**Beginner-Friendly Explanation:** When you query a view, the database doesn't just run the view's query separately—it combines your query with the view's query and runs the combined query against the real tables. This is called view resolution.

### Purposes

- To understand how views are executed and optimized.
- To predict performance characteristics of queries against views.
- To debug view-related performance issues.
- To write views that can be efficiently merged into outer queries.

### Syntax Rules and Structure

**View Resolution Steps:**

| Step | Operation |
|------|-----------|
| 1 | Replace column names in outer `SELECT` with names from defining query |
| 2 | Replace view name in `FROM` with defining query's `FROM` clause |
| 3 | Combine outer and defining query `WHERE` clauses with `AND` |
| 4 | Copy `GROUP BY` and `HAVING` from defining query |
| 5 | Rename fields in `ORDER BY` as needed |

**Component Breakdown:**

| Step | Example |
|------|---------|
| Original query | `SELECT Name FROM staff WHERE Job = 'EE'` |
| Step 1 | `SELECT ename FROM staff WHERE title = 'EE'` |
| Step 2 | `SELECT ename FROM emp WHERE title = 'EE'` |
| Step 3 | `SELECT ename FROM emp WHERE title = 'EE' AND dno = 'D2'` |

**Syntax Rules:**

- View resolution is performed by the query optimizer.
- The optimizer may choose to merge the view (inline it) or materialize it (compute it as a temporary table).
- Merging is generally preferred when possible.
- Materialization is used when merging is not feasible (e.g., aggregation, `DISTINCT`, `LIMIT`).

**Constraints and Limitations:**

- Merging is not possible when the view uses aggregation, `DISTINCT`, `GROUP BY`, `HAVING`, `LIMIT`, or `UNION`.
- If merging would result in an outer query referencing more than a maximum number of tables (e.g., 61 in MySQL), materialization is used.
- The optimizer's choice between merge and materialize can significantly affect performance.

### Annotated Complete Code Examples

**Example 1: View Resolution**

```sql
-- View definition
CREATE VIEW staff (Number, Name, Job) AS
SELECT eno, ename, title
FROM emp
WHERE dno = 'D2';

-- User query
SELECT Number, Name FROM staff WHERE Job = 'EE' ORDER BY Number;

-- Step 1: Replace column names
-- SELECT eno, ename FROM staff WHERE title = 'EE' ORDER BY Number;

-- Step 2: Replace view name with FROM clause
-- SELECT eno, ename FROM emp WHERE title = 'EE' ORDER BY Number;

-- Step 3: Combine WHERE clauses
-- SELECT eno, ename FROM emp WHERE title = 'EE' AND dno = 'D2' ORDER BY Number;

-- Step 5: Rename ORDER BY field
-- SELECT eno, ename FROM emp WHERE title = 'EE' AND dno = 'D2' ORDER BY eno;
```

**Why this matters:** The resolved query combines both the view's filter (`dno = 'D2'`) and the user's filter (`title = 'EE'`). The optimizer can then use indexes on either column to execute the query efficiently.

### Real-World Cases

**Case 1: Performance Tuning**

A DBA notices that a query against a complex view is slow. By examining the view resolution, they discover that the view cannot be merged due to a `GROUP BY`, forcing materialization. They rewrite the view to avoid `GROUP BY` or add appropriate indexes on the base tables.

**Case 2: Optimizer Hints**

A developer uses MySQL's `ALGORITHM = MERGE` hint to force merging of a view, enabling the outer query's `WHERE` conditions to be pushed down to the base tables.

### References

- Views (CS377) — View Resolution Steps - http://www.billmongan.com/Ursinus-CS377-Fall2023/files/cosc_304/topics/16_views/304_16_Views.pdf
- MySQL 8.0 Reference Manual — Optimizing Derived Tables and View References - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/derived-table-optimization.html

---

## 6. Security Through Views

### Definitions

**Core Definition:** Views provide security by restricting access to specific rows and columns of base tables, allowing users to query only the data they are authorised to see.

**Technical Definition:** Views add two levels of security: column-level security (exposing only selected columns) and value-based security (exposing only selected rows). By granting access to a view rather than the base table, administrators can control exactly what data users see. Combined with `DEFINER` and `SQL SECURITY` clauses, views can enforce access control even when users lack direct privileges on the base tables.

**Beginner-Friendly Explanation:** Views are a security tool. Instead of giving someone access to the entire employee table (with salaries, personal data, etc.), you create a view that shows only names and departments. The user can query the view but cannot see the sensitive columns.

### Purposes

- To restrict access to sensitive columns (e.g., salary, SSN) while allowing access to other columns.
- To restrict access to specific rows based on business rules (e.g., only rows for a specific department).
- To provide a secure interface for data access without granting direct table privileges.
- To implement row-level and column-level security.
- To mask or transform sensitive data before exposing it.

### Syntax Rules and Structure

**Security Through Views Example:**

```sql
-- Create a view that exposes only non-sensitive columns
CREATE VIEW employee_public AS
SELECT emp_id, emp_name, dept_name
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id;

-- Grant access to the view, not the base tables
GRANT SELECT ON employee_public TO reporting_user;
```

**Component Breakdown:**

| Security Level | How Views Provide It |
|---------------|---------------------|
| Column-level | View includes only selected columns |
| Row-level | View's `WHERE` clause filters rows |
| Value-based | View can transform or mask values |

**Syntax Rules:**

- Grant `SELECT` on the view, not on the base tables, to restrict access.
- Use `DEFINER` to specify the security context for privilege checking.
- `SQL SECURITY DEFINER` uses the definer's privileges; `SQL SECURITY INVOKER` uses the invoker's.
- Views can combine column-level and row-level security.

**Constraints and Limitations:**

- Users with direct table privileges can bypass view security.
- View definitions may expose data through metadata (e.g., column names).
- `DEFINER` security requires careful management of definer accounts.

### Annotated Complete Code Examples

**Example 1: Column-Level Security**

```sql
-- Full table with sensitive data
-- employees (emp_id, emp_name, salary, ssn, dept_id)

-- View exposing only non-sensitive columns
CREATE VIEW employee_directory AS
SELECT emp_id, emp_name, dept_id
FROM employees;

-- Grant access to the view only
GRANT SELECT ON employee_directory TO public_user;

-- public_user can query the view but NOT the base table
SELECT * FROM employee_directory;  -- OK
SELECT salary FROM employees;       -- ERROR: permission denied
```

**Why this works:** The view exposes only `emp_id`, `emp_name`, and `dept_id`. The `salary` and `ssn` columns are not accessible through the view, and the user has no direct privileges on the `employees` table.

**Example 2: Row-Level Security**

```sql
-- View that shows only employees in the Engineering department
CREATE VIEW engineering_staff AS
SELECT emp_id, emp_name, salary
FROM employees
WHERE dept_id = 10;

-- Grant access
GRANT SELECT ON engineering_staff TO eng_manager;
```

**Why this works:** The view's `WHERE` clause filters rows to only the Engineering department. The `eng_manager` can see Engineering employees but cannot see employees in other departments.

### Real-World Cases

**Case 1: Healthcare Data Access**

A hospital creates views that expose patient data without sensitive identifiers (e.g., name, SSN). Different roles get different views: doctors see clinical data, billing staff see financial data, and researchers see anonymised data.

**Case 2: Multi-Tenant SaaS**

A SaaS platform creates per-tenant views that filter by `tenant_id`. Each tenant's users can only see their own data, enforced by the view's `WHERE` clause.

### References

- Oracle Database Security Guide — Views and Security - https://docs.oracle.com/en/database/oracle/oracle-database/21/dbseg/
- PostgreSQL Documentation — CREATE VIEW (Security) - https://www.postgresql.org/docs/current/sql-createview.html
- SQL Engineering Handbook — View Security - https://github.com/theammarngp-makes/SQL-Engineering-Handbook

---

## 7. Materialized Views and REFRESH

### Definitions

**Core Definition:** A materialized view is a view that physically stores the result of its defining query in the database, providing fast access to pre-computed data at the cost of potential staleness.

**Technical Definition:** Materialized views in PostgreSQL use the rule system like views do, but persist the results in a table-like form. The main differences from `CREATE TABLE ... AS` are that the materialized view cannot subsequently be directly updated and that the query used to create the materialized view is stored in exactly the same way that a view's query is stored, so that fresh data can be generated with `REFRESH MATERIALIZED VIEW`.

**Beginner-Friendly Explanation:** A materialized view is like a regular view but it saves the results. Instead of running the query every time, the database stores the answer. This makes queries much faster, but the data can become outdated. You refresh it periodically to update the stored data.

### Purposes

- To dramatically improve query performance for expensive aggregations and joins.
- To provide fast access to pre-computed results for dashboards and reports.
- To reduce load on base tables for read-heavy workloads.
- To support data warehousing and OLAP scenarios.
- To allow indexing of view results (unlike regular views).

### Syntax Rules and Structure

**Complete General Syntax (PostgreSQL):**

```sql
CREATE MATERIALIZED VIEW view_name
[ (column_name [, ...]) ]
[ USING method ]
[ WITH ( storage_parameter [= value] [, ...] ) ]
[ TABLESPACE tablespace_name ]
AS query
[ WITH [ NO ] DATA ];

REFRESH MATERIALIZED VIEW [ CONCURRENTLY ] view_name;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `CREATE MATERIALIZED VIEW` | Creates a materialized view |
| `AS query` | The defining query |
| `WITH DATA` | Populates the view at creation (default) |
| `WITH NO DATA` | Creates the view but does not populate it |
| `REFRESH MATERIALIZED VIEW` | Regenerates the stored data |
| `CONCURRENTLY` | Refreshes without locking out reads (requires unique index) |

**Syntax Rules:**

- The materialized view is populated at creation time unless `WITH NO DATA` is used.
- `REFRESH MATERIALIZED VIEW` regenerates the data by re-executing the defining query.
- `REFRESH ... CONCURRENTLY` allows queries to continue during refresh but requires a unique index.
- Materialized views can have indexes, unlike regular views.
- Materialized views are not directly updatable.

**Constraints and Limitations:**

- MySQL does not support materialized views natively.
- Materialized views may become stale; refresh scheduling is required.
- `REFRESH ... CONCURRENTLY` is slower than a regular refresh but does not block reads.
- The defining query must be deterministic for materialized views to be meaningful.

### Annotated Complete Code Examples

**Example 1: Creating and Refreshing a Materialized View**

```sql
-- Create a materialized view for sales summary
CREATE MATERIALIZED VIEW sales_summary AS
SELECT seller_no,
       invoice_date,
       SUM(invoice_amt)::numeric(13,2) AS sales_amt
FROM invoice
WHERE invoice_date < CURRENT_DATE
GROUP BY seller_no, invoice_date;

-- Query the materialized view (fast, uses stored data)
SELECT * FROM sales_summary;

-- Refresh when source data changes
REFRESH MATERIALIZED VIEW sales_summary;

-- Refresh concurrently (requires unique index)
CREATE UNIQUE INDEX sales_summary_seller ON sales_summary (seller_no, invoice_date);
REFRESH MATERIALIZED VIEW CONCURRENTLY sales_summary;
```

**Why this is useful:** The materialized view stores aggregated sales data. Dashboard queries retrieve results instantly from the stored data instead of aggregating the `invoice` table each time. The nightly refresh job keeps the data current.

### Real-World Cases

**Case 1: Sales Dashboard**

A sales dashboard uses a materialized view that aggregates daily sales by product and region. The view is refreshed every hour, providing near-real-time data with sub-second query response.

**Case 2: Data Warehouse Summary Tables**

A data warehouse uses materialized views to pre-compute complex joins between fact and dimension tables, serving as summary tables for BI tools.

### References

- PostgreSQL Documentation — Materialized Views - https://www.postgresql.org/docs/current/rules-materializedviews.html
- PostgreSQL Documentation — CREATE MATERIALIZED VIEW - https://www.postgresql.org/docs/current/sql-creatematerializedview.html
- Azure Synapse Analytics — Materialized Views - https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/performance-tuning-materialized-views

---

## 8. Dropping and Altering Views

### Definitions

**Core Definition:** `DROP VIEW` removes a view from the database, while `ALTER VIEW` modifies an existing view's definition or properties.

**Technical Definition:** `DROP VIEW view_name [RESTRICT | CASCADE]` removes the view from the catalog. `RESTRICT` (default) prevents dropping if other objects depend on the view; `CASCADE` drops dependent views as well. `ALTER VIEW` (MySQL) or `CREATE OR REPLACE VIEW` (PostgreSQL) modifies the view definition.

**Beginner-Friendly Explanation:** `DROP VIEW` deletes a view. `ALTER VIEW` or `CREATE OR REPLACE VIEW` changes what the view does without deleting it first.

### Purposes

- To remove obsolete or unused views from the database.
- To modify a view's definition without dropping and recreating it.
- To manage dependencies between views.
- To maintain a clean database schema.

### Syntax Rules and Structure

**DROP VIEW:**

```sql
DROP VIEW [IF EXISTS] view_name [, ...] [RESTRICT | CASCADE];
```

**ALTER VIEW (MySQL):**

```sql
ALTER VIEW view_name [(column_list)]
AS select_statement
[WITH [CASCADED | LOCAL] CHECK OPTION];
```

**CREATE OR REPLACE VIEW (PostgreSQL, Oracle):**

```sql
CREATE OR REPLACE VIEW view_name AS
select_statement;
```

**Component Breakdown:**

| Statement | Purpose |
|-----------|---------|
| `DROP VIEW` | Removes a view |
| `IF EXISTS` | Suppresses error if view doesn't exist |
| `RESTRICT` | Fails if dependent objects exist |
| `CASCADE` | Drops dependent views |
| `ALTER VIEW` | Modifies a view (MySQL) |
| `CREATE OR REPLACE VIEW` | Replaces a view (PostgreSQL, Oracle) |

**Syntax Rules:**

- `DROP VIEW` with `RESTRICT` fails if other views depend on the target view.
- `DROP VIEW` with `CASCADE` drops the view and all dependent views.
- `ALTER VIEW` in MySQL requires the same privileges as `CREATE VIEW`.
- `CREATE OR REPLACE VIEW` in PostgreSQL requires that the new query generate the same columns.

**Constraints and Limitations:**

- `ALTER VIEW` is not standard SQL; different dialects use different syntaxes.
- Dropping a view does not affect the underlying base tables.
- `CASCADE` can cause unintended deletion of dependent views.

### Annotated Complete Code Examples

**Example 1: Dropping a View**

```sql
-- Drop a view
DROP VIEW employee_details;

-- Drop with IF EXISTS (no error if missing)
DROP VIEW IF EXISTS employee_details;

-- Drop with CASCADE (drops dependent views)
DROP VIEW employee_details CASCADE;
```

**Example 2: Altering a View (MySQL)**

```sql
-- Modify the view definition
ALTER VIEW employee_view AS
SELECT emp_id, emp_name, salary, dept_id, hire_date
FROM employees;
```

**Example 3: Replacing a View (PostgreSQL)**

```sql
CREATE OR REPLACE VIEW employee_view AS
SELECT emp_id, emp_name, salary, dept_id, hire_date
FROM employees;
```

### Real-World Cases

**Case 1: Schema Migration**

A DBA drops obsolete views during a schema migration and creates new ones with updated definitions.

**Case 2: View Definition Update**

A developer uses `CREATE OR REPLACE VIEW` to update a view's logic without dropping it, ensuring continuous availability for applications.

### References

- MySQL 8.0 Reference Manual — DROP VIEW Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/drop-view.html
- MySQL 8.0 Reference Manual — ALTER VIEW Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/alter-view.html
- PostgreSQL Documentation — DROP VIEW - https://www.postgresql.org/docs/current/sql-dropview.html

---

## 9. Performance Considerations

### Definitions

**Core Definition:** Performance considerations for views involve understanding how views affect query execution, optimization, and resource consumption.

**Technical Definition:** Regular views do not store data; their performance depends on the underlying query and the optimizer's ability to merge or materialize them. Materialized views store data physically, providing fast access but requiring refresh. Indexed views (SQL Server) can dramatically improve performance for certain query types but add maintenance overhead.

**Beginner-Friendly Explanation:** A view doesn't make queries faster by itself—it's just a saved query. The performance depends on what the view does and how the database executes it. Materialized views can be much faster because they save the results, but they need to be refreshed.

### Purposes

- To understand when views improve or degrade performance.
- To choose between regular views, materialized views, and indexed views.
- To optimise queries that use views.
- To diagnose performance issues related to views.

### Syntax Rules and Structure

**Performance Characteristics:**

| View Type | Performance Impact |
|-----------|-------------------|
| Simple view (mergeable) | Minimal overhead; inlined into outer query |
| Simple view (materialized) | Overhead of temporary table creation |
| Complex view (aggregation) | Must be materialized; potential overhead |
| Materialized view | Fast reads; refresh cost |
| Indexed view (SQL Server) | Fast reads; maintenance cost on writes |

**Syntax Rules:**

- Use `EXPLAIN` to see whether a view is merged or materialized.
- MySQL's `ALGORITHM` clause can force merging or materialization.
- Indexed views (SQL Server) require `SCHEMABINDING` and specific `SET` options.
- Materialized views can have indexes to speed up queries.

**Constraints and Limitations:**

- Views cannot be indexed directly (except indexed views in SQL Server).
- Complex views may prevent predicate pushdown, leading to full scans.
- Materialized views require refresh scheduling and consume storage.
- Indexed views have strict requirements and can slow down `INSERT`/`UPDATE` operations.

### Annotated Complete Code Examples

**Example 1: Mergeable vs. Non-Mergeable View**

```sql
-- Mergeable view (simple): optimizer inlines it
CREATE ALGORITHM = MERGE VIEW simple_view AS
SELECT emp_id, emp_name FROM employees WHERE salary > 50000;

-- Non-mergeable view (aggregation): must be materialized
CREATE ALGORITHM = TEMPTABLE VIEW aggregated_view AS
SELECT dept_id, AVG(salary) AS avg_sal
FROM employees
GROUP BY dept_id;
```

**Why the difference matters:** The mergeable view is inlined into the outer query, allowing the optimizer to push down conditions and use indexes. The aggregated view must be computed as a temporary table, which can be slower for large datasets.

### Real-World Cases

**Case 1: Dashboard Performance**

A dashboard uses a materialized view that aggregates millions of rows. Query response time drops from 30 seconds to under a second. A nightly refresh job keeps the data current.

**Case 2: Avoiding View Overhead**

A developer notices that a simple view is being materialized unnecessarily. They add `ALGORITHM = MERGE` to force merging, and the query speeds up.

### References

- Percona Server for MySQL — Views (Advantages/Disadvantages) - https://docs.percona.com/
- SQLite Forum — Efficiency: Views vs Select - https://www2.sqlite.org/
- Microsoft Learn — CREATE VIEW (Indexed Views) - https://learn.microsoft.com/en-us/sql/t-sql/statements/create-view-transact-sql
- Microsoft Learn — Performance tune with materialized views - https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/performance-tuning-materialized-views

---

## Summary Table of View Concepts

| Concept | Purpose | Key Limitation |
|---------|---------|----------------|
| View Definition | Named, stored query | Does not store data |
| Simple View | Single table, filter/rearrange | Limited complexity |
| Complex View | Multi-table joins/aggregations | Generally not updatable |
| Materialized View | Pre-computed results | Requires refresh; may be stale |
| CREATE VIEW | Define a new view | Requires CREATE VIEW privilege |
| Updatable View | DML through view | Strict criteria must be met |
| WITH CHECK OPTION | Prevent invisible rows | Only on updatable views |
| View Resolution | Translate view query | Merge or materialize |
| Security Through Views | Column/row-level access | Direct table grants bypass |
| REFRESH MATERIALIZED VIEW | Update stored data | May block reads (unless CONCURRENTLY) |
| DROP / ALTER VIEW | Remove or modify view | CASCADE can drop dependents |
| Performance | Merge vs. materialize | Complex views may not merge |

---

## Dialect Support for View Features

| Feature | PostgreSQL | MySQL | SQL Server | Oracle | SQLite |
|---------|-----------|-------|------------|--------|--------|
| CREATE VIEW | ✅ | ✅ | ✅ | ✅ | ✅ |
| OR REPLACE | ✅ | ✅ | ❌ (`CREATE OR ALTER`) | ✅ | ❌ |
| WITH CHECK OPTION | ✅ | ✅ | ✅ | ✅ | ❌ |
| Materialized Views | ✅ | ❌ | ❌ (Indexed Views) | ✅ | ❌ |
| REFRESH MATERIALIZED VIEW | ✅ | ❌ | ❌ | ✅ | ❌ |
| ALGORITHM Clause | ❌ | ✅ | ❌ | ❌ | ❌ |
| DEFINER / SQL SECURITY | ❌ | ✅ | ❌ | ❌ | ❌ |
| Indexed Views | ❌ | ❌ | ✅ | ❌ | ❌ |
| Updatable Views | ✅ | ✅ | ✅ | ✅ | ✅ |
| INSTEAD OF Triggers | ✅ | ❌ | ✅ | ✅ | ❌ |

---

## References

- MySQL 8.0 Reference Manual — CREATE VIEW Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/create-view.html
- MySQL 8.0 Reference Manual — The View WITH CHECK OPTION Clause - https://dev.mysql.com/doc/refman/8.0/en/view-check-option.html
- MySQL 8.0 Reference Manual — DROP VIEW Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/drop-view.html
- MySQL 8.0 Reference Manual — ALTER VIEW Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/alter-view.html
- PostgreSQL Documentation — CREATE VIEW - https://www.postgresql.org/docs/current/sql-createview.html
- PostgreSQL Documentation — Materialized Views - https://www.postgresql.org/docs/current/rules-materializedviews.html
- PostgreSQL Documentation — CREATE MATERIALIZED VIEW - https://www.postgresql.org/docs/current/sql-creatematerializedview.html
- PostgreSQL Documentation — DROP VIEW - https://www.postgresql.org/docs/current/sql-dropview.html
- Microsoft Learn — CREATE VIEW (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/statements/create-view-transact-sql
- Microsoft Learn — Performance tune with materialized views - https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/performance-tuning-materialized-views
- Oracle Database SQL Language Reference — CREATE VIEW - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CREATE-VIEW.html
- Oracle Database Security Guide — Views and Security - https://docs.oracle.com/en/database/oracle/oracle-database/21/dbseg/
- Baeldung — Databases: Simple vs. Complex vs. Materialized Views - https://www.baeldung.com/sql/databases-views-simple-complex-materialized
- Views (CS377) — View Resolution - http://www.billmongan.com/Ursinus-CS377-Fall2023/files/cosc_304/topics/16_views/304_16_Views.pdf
- Percona Server for MySQL — Views - https://docs.percona.com/
- SQL Engineering Handbook — View Security - https://github.com/theammarngp-makes/SQL-Engineering-Handbook
- ISO/IEC 9075-2 — SQL Standard (Views) - https://jtc1sc32.org/doc/N0251-0300/32N0254.pdf