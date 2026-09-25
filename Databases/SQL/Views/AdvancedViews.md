# SQL Advanced Views: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Advanced views are database objects that encapsulate complex query logic, provide security boundaries, and offer performance benefits, extending beyond simple `SELECT` statements to include updatable, read-only, security-focused, nested, and materialized variants.

**Technical Definition:** A view is a stored query that acts as a virtual table. The SQL standard defines views as derived tables whose contents are defined by a query expression. Advanced view types include: updatable views (where DML operations on the view propagate to base tables under specific conditions), read-only views (explicitly restricted from modification), security views (used for access control and column/row-level security), complex views (involving joins, aggregates, or unions), nested views (views built upon other views), and materialized views (which physically store the result set and support refresh operations).

**Beginner-Friendly Explanation:** A view is like a saved question you ask the database. Instead of writing the same complex query every time, you save it as a view and then just say "give me the answer to that question." Advanced views take this further: some let you change the data through the view, some are locked for reading only, some hide sensitive columns, some are built on top of other views, and some actually store the answer on disk for faster access.

### Key Characteristics

- **Virtual vs. Physical:** Standard views are virtual (query executes on reference); materialized views are physical (data stored on disk) .
- **Updatability:** Some views support DML operations; others are read-only depending on their definition .
- **Security Layer:** Views can restrict access to specific columns or rows, acting as an API to the database .
- **Nesting:** Views can reference other views, with nesting limits varying by RDBMS (e.g., SQL Server allows up to 32 levels) .

### Prerequisites

- **SQL Proficiency:** Solid understanding of `SELECT`, `JOIN`, `WHERE`, and DML statements.
- **Permissions:** `CREATE VIEW` privilege, plus appropriate permissions on underlying tables.
- **Database-Specific Knowledge:** Materialized view support varies significantly across RDBMSs.

### Related Programming Areas

- **Database Security:** Views are a fundamental tool for implementing column-level and row-level security.
- **Application Development:** Views provide a stable interface between applications and evolving schemas.
- **Performance Tuning:** Materialized views and indexed views are key optimization techniques.
- **Data Warehousing:** Materialized views are extensively used for pre-computing aggregates and joins.

### Core Concepts / Features

1. Updatable Views
2. Read-Only Views
3. Security Views
4. Complex Views
5. Nested Views
6. Materialized Views (Where Supported)

---

## 1. Updatable Views

### Definitions

**Core Definition:** An updatable view is a view through which `INSERT`, `UPDATE`, and `DELETE` statements can be executed, with changes propagating to the underlying base tables.

**Technical Definition:** A view is updatable if there is a one-to-one relationship between the rows in the view and the rows in the underlying table. MySQL documentation specifies that a view is not updatable if it contains aggregate functions, `DISTINCT`, `GROUP BY`, `HAVING`, subqueries in the select list, certain joins, or references to non-updatable views . For insertability, the view must contain all base table columns that do not have default values, and view columns must be simple column references (not expressions) .

**Beginner-Friendly Explanation:** An updatable view is like a window into a table that lets you not only look through it but also reach in and change things. If you update a row through the view, the underlying table is updated. But not every view allows this—only those that map directly to a single table's rows.

### Purposes

- To provide a simplified interface for data modification without exposing base table structure.
- To enforce security constraints while allowing controlled updates.
- To centralize DML logic when combined with `WITH CHECK OPTION`.

### Syntax Rules and Structure

**Complete General Syntax (with Check Option):**

```sql
CREATE VIEW view_name AS
SELECT column_list
FROM table_name
WHERE condition
WITH CHECK OPTION;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `WITH CHECK OPTION` | Ensures that `INSERT`/`UPDATE` through the view cannot produce rows that would not be visible through the view |
| `LOCAL` / `CASCADED` | Controls whether check applies to just this view (`LOCAL`) or all nested views (`CASCADED`, default)  |

**Syntax Rules:**

- A view is updatable if there is a one-to-one relationship between view rows and base table rows .
- Views with `DISTINCT`, `GROUP BY`, aggregates, or `UNION` are generally not updatable .
- `WITH CHECK OPTION` prevents inserting or updating rows that fall outside the view's `WHERE` condition .
- Multi-table views may be updatable if using `MERGE` algorithm and inner joins, but only one base table can be modified per statement .

**Constraints and Limitations:**

- PostgreSQL views are read-only by default; triggers are needed for updatable behavior .
- SQL Server views are updatable under certain conditions; `ORDER BY` is not permitted unless `TOP` or `OFFSET/FETCH` is used .

### Annotated Complete Code Examples

**Example 1: Updatable View with WITH CHECK OPTION**

```sql
-- Create a base table
CREATE TABLE high_value_orders (
    order_id     INT PRIMARY KEY,
    customer_id  INT,
    total_amount DECIMAL(10,2)
);

-- Create an updatable view showing only high-value orders
CREATE VIEW v_high_value_orders AS
SELECT order_id, customer_id, total_amount
FROM high_value_orders
WHERE total_amount > 1000
WITH CHECK OPTION;

-- This INSERT succeeds (total_amount > 1000)
INSERT INTO v_high_value_orders (order_id, customer_id, total_amount)
VALUES (1, 100, 1500.00);

-- This INSERT fails (total_amount <= 1000, violates CHECK OPTION)
INSERT INTO v_high_value_orders (order_id, customer_id, total_amount)
VALUES (2, 101, 500.00);
-- ERROR: new row violates check option for view "v_high_value_orders"
```

**Why this output occurs:** The `WITH CHECK OPTION` ensures that any row inserted through the view must satisfy the view's `WHERE` condition (`total_amount > 1000`). The first insert succeeds because 1500 > 1000. The second insert fails because 500 does not satisfy the condition. SQL Server documentation confirms this behavior: "With this option, you can't insert an order with TotalAmount of 500 through the HighValueOrders view" .

**Example 2: PostgreSQL Updatable View via Trigger**

```sql
-- PostgreSQL views are read-only by default
CREATE VIEW employee_view AS
SELECT emp_id, emp_name, department
FROM employees;

-- To make it updatable, create an INSTEAD OF trigger
CREATE OR REPLACE FUNCTION employee_view_insert()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO employees (emp_id, emp_name, department)
    VALUES (NEW.emp_id, NEW.emp_name, NEW.department);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER employee_view_insert_trigger
INSTEAD OF INSERT ON employee_view
FOR EACH ROW
EXECUTE FUNCTION employee_view_insert();

-- Now INSERT through the view works
INSERT INTO employee_view (emp_id, emp_name, department)
VALUES (100, 'Alice', 'Engineering');
```

**Why this output occurs:** PostgreSQL documentation states: "Currently, views are read only: the system will not allow an insert, update, or delete on a view. You can get the effect of an updatable view by creating INSTEAD triggers on the view" . The trigger intercepts the `INSERT` on the view and redirects it to the base table.

### Real-World Cases

**Case 1: Application Data Entry**

An application uses a view to insert customer records. The view filters out `deleted_at IS NULL` and includes `WITH CHECK OPTION`. This prevents accidental insertion of soft-deleted records and simplifies application code.

**Case 2: Multi-Table View Update**

Oracle supports updatable join views where only one base table is modified per DML statement. A view joining `employees` and `departments` allows updating `employees` columns while `departments` columns remain read-only through the view .

### References

- PostgreSQL Documentation — CREATE VIEW - https://www.postgresql.org/docs/9.1/sql-createview.html
- MySQL 9.7 Reference Manual — Updatable and Insertable Views - https://dev.mysql.com/doc/refman/9.7/en/view-updatability.html
- Microsoft Learn — Create Views - https://learn.microsoft.com/en-us/training/modules/implement-programmability-objects/2-create-views
- Oracle Database — 更新可能な結合ビュー - https://docs.oracle.com/cd/E57425_01/121/CNCPT/schemaob.htm

---

## 2. Read-Only Views

### Definitions

**Core Definition:** A read-only view is a view that explicitly prohibits `INSERT`, `UPDATE`, and `DELETE` operations, allowing only `SELECT` queries.

**Technical Definition:** A view can be made read-only by definition (containing constructs that prevent updatability, such as aggregates or `UNION`) or explicitly using the `WITH READ ONLY` clause. Oracle documentation states that "the database does not allow changes to a read-only view" and that updatable join views are "not restricted by the WITH READ ONLY clause" .

**Beginner-Friendly Explanation:** A read-only view is like a window that only lets you look, not touch. You can see the data, but you cannot change it through the view. This is useful for reports and dashboards where you want to guarantee that no accidental modifications occur.

### Purposes

- To prevent accidental or unauthorized data modifications through the view.
- To enforce read-only access for reporting and analytics users.
- To provide a safe interface for sensitive data that should never be modified directly.

### Syntax Rules and Structure

**Complete General Syntax (Oracle):**

```sql
CREATE VIEW view_name AS
SELECT column_list
FROM table_name
WHERE condition
WITH READ ONLY;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `WITH READ ONLY` | Explicitly disables all DML operations on the view |

**Syntax Rules:**

- `WITH READ ONLY` can be applied to any view, including those that might otherwise be updatable .
- Implicitly read-only views result from constructs like `DISTINCT`, `GROUP BY`, aggregates, `UNION`, or subqueries .
- In PostgreSQL, all views are read-only by default unless triggers are added .

**Constraints and Limitations:**

- `WITH READ ONLY` does not prevent modifications to the underlying base tables directly.
- Read-only views may still be optimized by the query optimizer; they are not materialized unless explicitly defined as such.

### Annotated Complete Code Examples

**Example 1: Explicit Read-Only View (Oracle)**

```sql
-- Create a read-only view of employee salary data
CREATE VIEW v_employee_salaries AS
SELECT employee_id, last_name, salary, department_id
FROM employees
WHERE department_id = 80
WITH READ ONLY;

-- SELECT works
SELECT * FROM v_employee_salaries;

-- DML fails
UPDATE v_employee_salaries SET salary = 99999 WHERE employee_id = 100;
-- ERROR: ORA-01733: virtual column not allowed here
```

**Why this output occurs:** The `WITH READ ONLY` clause explicitly prevents any DML operations on the view. Oracle documentation confirms that "updatable join views are called modifiable join views" and are "not restricted by the WITH READ ONLY clause," implying that read-only views are restricted .

### Real-World Cases

**Case 1: Executive Dashboard**

A dashboard displays company-wide financial metrics through a read-only view. This ensures that report consumers cannot accidentally modify the underlying transaction data.

**Case 2: Compliance Reporting**

A regulated industry requires that all financial reports use immutable views. Read-only views guarantee that reported figures cannot be altered through the reporting interface.

### References

- Oracle Database — 更新可能な結合ビュー - https://docs.oracle.com/cd/E57425_01/121/CNCPT/schemaob.htm
- MySQL 9.7 Reference Manual — Updatable and Insertable Views - https://dev.mysql.com/doc/refman/9.7/en/view-updatability.html

---

## 3. Security Views

### Definitions

**Core Definition:** A security view is a view used to implement access control, restricting users to specific columns or rows of a table without granting direct table access.

**Technical Definition:** Security views leverage the view owner's permissions rather than the querying user's permissions. PostgreSQL documentation states: "Access to tables referenced in the view is determined by permissions of the view owner. In some cases, this can be used to provide secure but restricted access to the underlying tables" . SQL Server training materials note that views "provide a security layer" and allow granting "users access to specific columns through a view while restricting access to the underlying tables" .

**Beginner-Friendly Explanation:** A security view is like giving someone a filtered copy of a document. They can see the parts you want them to see, but they cannot access the original document or the parts you've hidden. It's a way to share data without giving away the keys to the whole database.

### Purposes

- To restrict column-level access (hiding sensitive columns like salary or SSN).
- To restrict row-level access (showing only rows relevant to a user's role or region).
- To provide an abstraction layer that hides schema complexity.
- To enforce least-privilege access without managing complex permission schemes on base tables.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
CREATE VIEW view_name AS
SELECT public_columns
FROM table_name
WHERE row_filter_condition;
```

**Syntax Rules:**

- The view owner must have `SELECT` privilege on the base table .
- Users granted `SELECT` on the view do not need direct table access.
- Row-level security can be implemented using `WHERE` clauses with session variables or user functions.

**Constraints and Limitations:**

- PostgreSQL documentation warns that "not all views are secure against tampering" .
- Functions called in the view are executed with the view owner's permissions; users must have permission to call those functions.
- Complex security views with functions may have security implications if the functions are not carefully written.

### Annotated Complete Code Examples

**Example 1: Column-Level Security View**

```sql
-- Base table with sensitive data
CREATE TABLE employees (
    emp_id       INT PRIMARY KEY,
    emp_name     VARCHAR(100),
    salary       DECIMAL(10,2),
    ssn          VARCHAR(11),
    department   VARCHAR(50)
);

-- Security view exposing only non-sensitive columns
CREATE VIEW v_employee_directory AS
SELECT emp_id, emp_name, department
FROM employees;

-- Grant access to the view, not the table
GRANT SELECT ON v_employee_directory TO reporting_user;

-- reporting_user can query the view
SELECT * FROM v_employee_directory;
-- But cannot access salary or ssn
SELECT salary FROM employees;
-- ERROR: permission denied for table employees
```

**Why this output occurs:** The view exposes only `emp_id`, `emp_name`, and `department`. The `reporting_user` has `SELECT` on the view but not on the base table. PostgreSQL documentation confirms that "access to tables referenced in the view is determined by permissions of the view owner" .

**Example 2: Row-Level Security View**

```sql
-- View showing only active employees in a specific region
CREATE VIEW v_active_regional_employees AS
SELECT emp_id, emp_name, department, region
FROM employees
WHERE is_active = 1
  AND region = current_setting('app.user_region');

-- Grant to regional managers
GRANT SELECT ON v_active_regional_employees TO regional_manager;
```

**Why this output occurs:** The view uses `current_setting('app.user_region')` to dynamically filter rows based on the application's session context. A manager for the "North" region sees only North region employees. This implements row-level security without granting access to the full `employees` table.

### Real-World Cases

**Case 1: Healthcare Data Access**

A hospital database has a `patients` table with sensitive medical information. A security view exposes only patient name, room number, and attending physician for nursing staff, while keeping diagnoses and medications hidden.

**Case 2: Multi-Tenant SaaS**

A SaaS application stores all customer data in shared tables. Each tenant is granted access to a view filtered by `WHERE tenant_id = current_tenant()`. This provides data isolation without complex permission schemas .

### References

- PostgreSQL Documentation — CREATE VIEW - https://www.postgresql.org/docs/9.1/sql-createview.html
- Microsoft Learn — Create Views - https://learn.microsoft.com/en-us/training/modules/implement-programmability-objects/2-create-views
- Microsoft Learn — Create and query views - https://learn.microsoft.com/en-ie/training/modules/create-tables-views-temporary-objects/3-create-query-views

---

## 4. Complex Views

### Definitions

**Core Definition:** A complex view is a view that involves multiple tables (joins), aggregations, unions, subqueries, or calculated columns, presenting a transformed or consolidated view of data.

**Technical Definition:** Complex views typically use `JOIN`, `GROUP BY`, `UNION`, `CASE` expressions, and scalar functions. They are often read-only because the transformation makes direct DML mapping ambiguous or impossible. Oracle documentation shows a view `EMP_LOCATIONS` joining four tables (`EMPLOYEES`, `DEPARTMENTS`, `LOCATIONS`, `COUNTRIES`) as an example of a complex view .

**Beginner-Friendly Explanation:** A complex view is like a summary report. Instead of showing raw data from one table, it combines information from several tables, calculates totals, and presents it in a readable format. You cannot usually edit the summary and expect the raw data to change accordingly.

### Purposes

- To encapsulate complex multi-table joins behind a simple interface.
- To provide pre-aggregated or pre-calculated data for reporting.
- To simplify application code by hiding query complexity.
- To present data in a format that matches business requirements.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
CREATE VIEW view_name AS
SELECT t1.col1, t2.col2, SUM(t3.col3) AS total
FROM table1 t1
JOIN table2 t2 ON t1.id = t2.id
JOIN table3 t3 ON t2.id = t3.id
WHERE t1.status = 'active'
GROUP BY t1.col1, t2.col2;
```

**Syntax Rules:**

- Complex views are generally read-only .
- `ORDER BY` is not permitted in most RDBMS view definitions unless combined with `TOP` or `OFFSET/FETCH` .
- Column aliases are required for calculated columns to give them meaningful names .

**Constraints and Limitations:**

- Complex views may perform poorly if the underlying joins are not optimized.
- Nesting complex views can lead to performance degradation.
- `GROUP BY` and aggregates make views non-updatable .

### Annotated Complete Code Examples

**Example 1: Multi-Table Join View (Oracle)**

```sql
-- Complex view joining four tables (from Oracle documentation)
CREATE VIEW emp_locations AS
SELECT e.employee_id,
       e.last_name || ', ' || e.first_name AS name,
       d.department_name AS department,
       l.city AS city,
       c.country_name AS country
FROM employees e
JOIN departments d ON e.department_id = d.department_id
JOIN locations l ON d.location_id = l.location_id
JOIN countries c ON l.country_id = c.country_id
ORDER BY e.last_name;

-- Query the view
SELECT * FROM emp_locations WHERE department = 'Sales';
```

**Why this output occurs:** The view consolidates employee, department, location, and country data into a single virtual table. Oracle documentation provides this exact example as a demonstration of creating a view that joins four tables . The `ORDER BY` is accepted here (Oracle allows it in view definitions), but other RDBMSs may not.

**Example 2: Aggregated View (SQL Server)**

```sql
-- View with aggregation and CASE expression
CREATE VIEW Sales.OrderSummary AS
SELECT 
    o.CustomerID,
    YEAR(o.OrderDate) AS OrderYear,
    MONTH(o.OrderDate) AS OrderMonth,
    SUM(o.TotalAmount) AS MonthlyTotal,
    CASE 
        WHEN SUM(o.TotalAmount) < 1000 THEN 'Low'
        WHEN SUM(o.TotalAmount) < 10000 THEN 'Medium'
        ELSE 'High'
    END AS RevenueCategory
FROM Sales.Orders o
GROUP BY o.CustomerID, YEAR(o.OrderDate), MONTH(o.OrderDate);

-- Query the view
SELECT * FROM Sales.OrderSummary 
WHERE OrderYear = 2026 AND RevenueCategory = 'High';
```

**Why this output occurs:** The view aggregates order totals by customer and month, then categorizes the revenue. SQL Server training materials show a similar example with `CASE` for order size categorization . This view is read-only because it contains `GROUP BY` and aggregates.

### Real-World Cases

**Case 1: Sales Dashboard**

A complex view joins `orders`, `order_details`, `products`, and `customers` to produce a denormalized sales report. The dashboard queries this single view instead of managing four-table joins in application code.

**Case 2: Financial Reporting**

A view aggregates transactions by account and month, calculates running balances, and applies business rules for categorization. Finance teams query the view for reports without understanding the underlying schema complexity.

### References

- Oracle Database — Creating and Managing Views - https://docs.oracle.com/cd/E57425_01/121/CNCPT/schemaob.htm
- Microsoft Learn — Create and query views - https://learn.microsoft.com/en-ie/training/modules/create-tables-views-temporary-objects/3-create-query-views
- MySQL 9.7 Reference Manual — Updatable and Insertable Views - https://dev.mysql.com/doc/refman/9.7/en/view-updatability.html

---

## 5. Nested Views

### Definitions

**Core Definition:** A nested view is a view whose definition references another view (or views) as its data source, creating a hierarchy of virtual tables.

**Technical Definition:** Nested views are views built upon other views. PostgreSQL documentation states that "building views upon other views is not uncommon" . SQL Server allows views to be nested up to 32 levels, though the actual limit may be lower depending on complexity and available memory .

**Beginner-Friendly Explanation:** A nested view is like a layered cake. The bottom layer might be a simple view of a table. The middle layer is a view that builds on the first view. The top layer is a view that builds on the middle view. Each layer adds or transforms data, creating a hierarchy.

### Purposes

- To build modular, reusable query components.
- To create layered abstractions that simplify complex logic.
- To separate concerns: base views for raw data, middle views for transformations, top views for presentation.
- To enable incremental development of complex query logic.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
-- Layer 1: Base view
CREATE VIEW v_base AS
SELECT * FROM table_name WHERE condition;

-- Layer 2: Intermediate view referencing base view
CREATE VIEW v_intermediate AS
SELECT column_list, calculation
FROM v_base
WHERE condition;

-- Layer 3: Top-level view referencing intermediate view
CREATE VIEW v_top AS
SELECT * FROM v_intermediate;
```

**Syntax Rules:**

- Nested views can reference any existing view.
- Changes to base views (e.g., `DROP`) invalidate dependent views.
- SQL Server allows up to 32 levels of nesting .
- Performance may degrade with deep nesting because the optimizer must flatten the view hierarchy.

**Constraints and Limitations:**

- Deep nesting can make debugging difficult.
- `ORDER BY` in nested views is generally ignored or disallowed .
- Schema changes to base tables may break nested views.

### Annotated Complete Code Examples

**Example 1: Two-Level Nested View**

```sql
-- Base table
CREATE TABLE products (
    product_id   INT PRIMARY KEY,
    product_name VARCHAR(100),
    category     VARCHAR(50),
    price        DECIMAL(10,2),
    stock        INT
);

-- Layer 1: View for in-stock products
CREATE VIEW v_in_stock AS
SELECT product_id, product_name, category, price, stock
FROM products
WHERE stock > 0;

-- Layer 2: View for expensive in-stock products
CREATE VIEW v_expensive_in_stock AS
SELECT product_id, product_name, category, price
FROM v_in_stock
WHERE price > 100;

-- Query the nested view
SELECT * FROM v_expensive_in_stock ORDER BY price DESC;
```

**Why this output occurs:** The top-level view `v_expensive_in_stock` references `v_in_stock`, which references `products`. PostgreSQL documentation confirms that "building views upon other views is not uncommon" . The query against the nested view ultimately executes the combined logic: filter in-stock products, then filter by price.

**Example 2: Three-Level Nested View with Calculation**

```sql
-- Layer 1: Raw data view
CREATE VIEW v_raw_orders AS
SELECT order_id, customer_id, order_date, total_amount
FROM orders;

-- Layer 2: Add calculated column
CREATE VIEW v_orders_with_year AS
SELECT order_id, customer_id, order_date, total_amount,
       YEAR(order_date) AS order_year
FROM v_raw_orders;

-- Layer 3: Aggregate by year
CREATE VIEW v_annual_sales AS
SELECT order_year,
       COUNT(*) AS order_count,
       SUM(total_amount) AS annual_total
FROM v_orders_with_year
GROUP BY order_year;
```

**Why this output occurs:** Each layer adds a transformation. Layer 1 selects raw data. Layer 2 adds a calculated year column. Layer 3 aggregates by year. This modular approach allows different consumers to use the appropriate layer without duplicating logic.

### Real-World Cases

**Case 1: Data Warehouse Staging**

A data warehouse uses nested views: `v_staging` loads raw data, `v_cleansed` applies data quality rules, `v_transformed` applies business logic, and `v_presentation` formats for BI tools. Each layer is a view referencing the previous one.

**Case 2: Application API Layer**

An application exposes different views for different user roles: `v_user_basic` (name, email), `v_user_extended` (adds phone, address), `v_user_admin` (adds role, permissions). These nested views share a common base view and add columns at each level.

### References

- PostgreSQL Documentation — Views - https://www.postgresql.org/docs/14/tutorial-views.html
- PostgreSQL Documentation (9.4) — Views - https://www.postgresql.org/docs/9.4/tutorial-views.html
- Microsoft Learn — Projektowanie i wdrażanie widoków - https://learn.microsoft.com/pl-pl/previous-versions/sql/sql-server-2008-r2/ms189918(v=sql.105)

---

## 6. Materialized Views (Where Supported)

### Definitions

**Core Definition:** A materialized view is a database object that physically stores the result set of its defining query, unlike a standard view which recomputes the query each time it is referenced.

**Technical Definition:** Oracle documentation states that "materialized views are similar to indexes" because "they contain actual data and consume storage space" and "can be refreshed when the data in their master tables changes" . PostgreSQL introduced materialized views in version 9.3 . MySQL does **not** have native materialized views .

**Beginner-Friendly Explanation:** A materialized view is like a photocopy of a report. Instead of running the report every time you need it (which might take a long time), you make a copy and store it. When the underlying data changes, you refresh the copy. It's faster to read but may be slightly out of date.

### Purposes

- To pre-compute and store expensive query results (complex joins, aggregations).
- To improve query performance for frequently accessed, read-heavy workloads.
- To serve as a caching mechanism for remote or slow data sources.
- To support data warehouse query rewrite optimization .

### Syntax Rules and Structure

**Complete General Syntax (PostgreSQL):**

```sql
CREATE MATERIALIZED VIEW view_name AS
SELECT column_list
FROM table_name
WHERE condition
[WITH [NO] DATA];

REFRESH MATERIALIZED VIEW view_name;
REFRESH MATERIALIZED VIEW CONCURRENTLY view_name;  -- Requires unique index
```

**Complete General Syntax (Oracle):**

```sql
CREATE MATERIALIZED VIEW view_name AS
SELECT ...;

-- Refresh methods
REFRESH MATERIALIZED VIEW view_name;
-- Or configured for ON COMMIT / ON DEMAND
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `WITH NO DATA` | Creates the view definition without populating it  |
| `REFRESH` | Re-executes the defining query and updates stored data |
| `CONCURRENTLY` | Allows reads during refresh (PostgreSQL 9.4+)  |

**Syntax Rules:**

- Materialized views must be explicitly refreshed; they do not auto-update .
- PostgreSQL: `REFRESH MATERIALIZED VIEW` takes an exclusive lock, blocking reads .
- PostgreSQL: `CONCURRENTLY` requires a unique index and allows reads during refresh .
- Oracle supports fast refresh (incremental) using materialized view logs, and query rewrite .

**Constraints and Limitations:**

- **MySQL does not support materialized views natively** .
- PostgreSQL: Cannot be temporary or unlogged .
- Refresh can be expensive for large datasets.
- Data may be stale between refreshes.
- PostgreSQL: `pg_dump` dumps only the definition, not the data .

### Annotated Complete Code Examples

**Example 1: PostgreSQL Materialized View**

```sql
-- Create a materialized view (no data initially)
CREATE MATERIALIZED VIEW mv_sales_summary AS
SELECT 
    product_id,
    DATE_TRUNC('month', sale_date) AS month,
    SUM(amount) AS total_sales
FROM sales
GROUP BY product_id, DATE_TRUNC('month', sale_date)
WITH NO DATA;

-- Refresh to populate
REFRESH MATERIALIZED VIEW mv_sales_summary;

-- Query works
SELECT * FROM mv_sales_summary WHERE month = '2026-01-01';

-- Create a unique index for CONCURRENTLY refresh
CREATE UNIQUE INDEX idx_mv_sales_summary
ON mv_sales_summary (product_id, month);

-- Concurrent refresh (allows reads during refresh)
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_sales_summary;
```

**Why this output occurs:** The materialized view stores the aggregated sales data. PostgreSQL documentation confirms that `CREATE MATERIALIZED VIEW ... WITH NO DATA` creates the view without populating it, and `REFRESH MATERIALIZED VIEW` populates it . `CONCURRENTLY` requires a unique index and allows queries during refresh .

**Example 2: Oracle Materialized View with Fast Refresh**

```sql
-- Create materialized view log on base table (for fast refresh)
CREATE MATERIALIZED VIEW LOG ON sales
WITH ROWID, PRIMARY KEY, SEQUENCE (sale_date, amount, product_id)
INCLUDING NEW VALUES;

-- Create materialized view with fast refresh
CREATE MATERIALIZED VIEW sales_mv
REFRESH FAST ON COMMIT
AS
SELECT product_id, SUM(amount) AS total_sales
FROM sales
GROUP BY product_id;

-- Query the materialized view
SELECT * FROM sales_mv WHERE product_id = 100;
```

**Why this output occurs:** Oracle documentation describes materialized view logs as recording changes to base tables so the materialized view can be refreshed incrementally (fast refresh) . `ON COMMIT` refreshes automatically when the base table is modified. Oracle also supports query rewrite, where queries against base tables are transparently rewritten to use materialized views .

### Real-World Cases

**Case 1: Data Warehouse Aggregates**

A data warehouse pre-computes daily sales totals by product and region using a materialized view. Dashboards query the materialized view instead of scanning billions of transaction rows. The view is refreshed nightly.

**Case 2: Cross-Database Reporting**

A reporting system needs data from a remote, slow database. A materialized view is created locally with the remote query. The view is refreshed periodically, providing fast local access to remote data .

**Case 3: MySQL Workaround**

Since MySQL lacks materialized views, developers simulate them using a regular table populated by a scheduled `INSERT ... SELECT` or `CREATE TABLE ... SELECT`, refreshed via cron or events. This is a common pattern documented in community resources.

### References

- Oracle Database — Partitions, Views, and Other Schema Objects (Materialized Views) - https://docs.oracle.com/cd/E57425_01/121/CNCPT/schemaob.htm
- PostgreSQL Wiki — Materialised Views in PostgreSQL - https://wiki.postgresql.org/images/8/85/Materialised_Views_-_FOSDEM.pdf
- MySQL 9.7 FAQ — Does MySQL have materialized views? - https://docs.oracle.com/cd/E17952_01/mysql-9.7-en/faqs-views.html

---

## Summary Table

| View Type | Updatable? | Stores Data? | Primary Use Case |
|-----------|-----------|--------------|------------------|
| **Updatable View** | Yes (with conditions) | No | Data entry through simplified interface |
| **Read-Only View** | No (`WITH READ ONLY`) | No | Reporting, dashboards, compliance |
| **Security View** | Usually No | No | Column/row access control |
| **Complex View** | Usually No | No | Multi-table joins, aggregations |
| **Nested View** | Depends on layers | No | Modular query design |
| **Materialized View** | Limited | Yes | Performance caching, data warehousing |

---

## References

- PostgreSQL Documentation — CREATE VIEW - https://www.postgresql.org/docs/9.1/sql-createview.html
- MySQL 9.7 Reference Manual — Updatable and Insertable Views - https://dev.mysql.com/doc/refman/9.7/en/view-updatability.html
- Microsoft Learn — Create Views - https://learn.microsoft.com/en-us/training/modules/implement-programmability-objects/2-create-views
- Oracle Database — 更新可能な結合ビュー - https://docs.oracle.com/cd/E57425_01/121/CNCPT/schemaob.htm
- Oracle Database — Partitions, Views, and Other Schema Objects (Materialized Views) - https://docs.oracle.com/cd/E57425_01/121/CNCPT/schemaob.htm
- PostgreSQL Documentation — Views - https://www.postgresql.org/docs/14/tutorial-views.html
- PostgreSQL Documentation (9.4) — Views - https://www.postgresql.org/docs/9.4/tutorial-views.html
- PostgreSQL Wiki — Materialised Views in PostgreSQL - https://wiki.postgresql.org/images/8/85/Materialised_Views_-_FOSDEM.pdf
- MySQL 9.7 FAQ — Does MySQL have materialized views? - https://docs.oracle.com/cd/E17952_01/mysql-9.7-en/faqs-views.html
- Microsoft Learn — Create and query views - https://learn.microsoft.com/en-ie/training/modules/create-tables-views-temporary-objects/3-create-query-views
- Microsoft Learn — Projektowanie i wdrażanie widoków - https://learn.microsoft.com/pl-pl/previous-versions/sql/sql-server-2008-r2/ms189918(v=sql.105)
- Oracle Database — Creating and Managing Views - https://docs.oracle.com/en/database/oracle/oracle-database/19/tdddg/creating-and-managing-views.html