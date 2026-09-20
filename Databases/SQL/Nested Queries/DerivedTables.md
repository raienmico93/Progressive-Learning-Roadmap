# SQL Derived Tables (Subqueries in FROM): A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A derived table is a subquery placed in the `FROM` clause of an outer `SELECT` statement, functioning as a temporary, named table that the outer query can reference.

**Technical Definition:** Per the SQL standard (ISO/IEC 9075-2), a `<derived table>` is a `<table subquery>` — a parenthesised `<query expression>` that appears in the `<from clause>` of a `<table expression>`. The derived table produces a result set that the outer query treats as a table, provided it is assigned a correlation name (alias). The derived table is logical, not physical: it is not stored in the database, and its scope is limited to the outer query in which it is defined. MySQL documentation states: "A derived table is an expression that generates a table within the scope of a query `FROM` clause. For example, a subquery in a `SELECT` statement `FROM` clause is a derived table".

**Beginner-Friendly Explanation:** A derived table is a query result that you use like a table in another query. Instead of querying a real table, you query the result of another `SELECT` statement. It's like creating a temporary, on-the-fly table just for that one query. You write a subquery in the `FROM` clause, give it a name, and then the outer query can select from it, join it, filter it, and aggregate it as if it were a real table.

### Key Characteristics

- **Inline, temporary result set:** The derived table exists only for the duration of the outer query and is not stored in the database.
- **Mandatory alias:** Every derived table must be assigned a correlation name (alias); otherwise, the query fails.
- **Column aliasing:** Columns in the derived table must have unique names; aliases are recommended for all columns.
- **No special privileges required:** Because derived tables are not stored objects, no additional security privileges are needed beyond `SELECT` rights on the source objects.
- **Scope-bound:** The derived table is created at execution time and goes out of scope when the outer query ends.
- **Optimizer merging:** Modern optimisers can merge derived tables into the outer query or materialise them as temporary tables, depending on the query structure.
- **Correlation restrictions:** In MySQL, a derived table cannot be a correlated subquery and cannot contain outer references (though this is a MySQL restriction, not a standard SQL restriction).

### Prerequisites

- **Basic SQL knowledge:** Familiarity with `SELECT`, `FROM`, and `WHERE`.
- **Subquery fundamentals:** Understanding of scalar, single-row, and multi-row subqueries.
- **Table aliasing:** Ability to assign and use table aliases (correlation names).
- **Aggregate functions:** Knowledge of `COUNT`, `SUM`, `AVG`, `MIN`, and `MAX`.

### Related Programming Areas

- **Data Analysis and Business Intelligence:** Derived tables enable multi-step aggregations and pre-join transformations.
- **Application Development:** Backend services use derived tables for modular, readable queries.
- **Data Engineering (ETL):** Derived tables are used for pre-aggregation before joining large tables.
- **Database Administration:** DBAs use derived tables for data validation and ad-hoc analysis.

### Core Concepts / Features

1. Derived Table Syntax and Structure
2. Mandatory Alias and Column Naming
3. Scope, Lifetime, and Visibility
4. Derived Tables vs. Correlated Subqueries
5. Derived Tables vs. Common Table Expressions (CTEs)
6. Derived Tables vs. Views and Temporary Tables
7. Optimisation: Merging vs. Materialisation
8. Common Use Cases and Patterns

---

## 1. Derived Table Syntax and Structure

### Definitions

**Core Definition:** The syntax of a derived table consists of a parenthesised `SELECT` statement placed in the `FROM` clause, followed by an alias that the outer query uses to reference it.

**Technical Definition:** The general form is `FROM (SELECT ...) AS alias`. The inner `SELECT` can contain any valid query expression, including `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`, and joins. The outer query can then reference columns from the derived table using the alias, apply its own `WHERE`, `JOIN`, `GROUP BY`, and `ORDER BY` clauses, and combine the derived table with other tables.

**Beginner-Friendly Explanation:** You write a query inside parentheses, give it a name, and then write another query that uses that name like a table name. The inner query runs first, its result becomes a temporary table, and the outer query operates on that temporary table.

### Purposes

- To break complex queries into modular, readable steps.
- To pre-aggregate data before joining it to other tables.
- To apply filters to aggregated results using `WHERE` on the outer query (a workaround for the logical order of query processing).
- To reuse a subquery result multiple times in the outer query.
- To work around restrictions on column aliases in `WHERE` and `HAVING` clauses.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT <outer_columns>
FROM (
    SELECT <inner_columns>
    FROM <source_table>
    [WHERE <inner_condition>]
    [GROUP BY <grouping_columns>]
    [HAVING <group_condition>]
) AS <derived_table_alias>
[WHERE <outer_condition>]
[ORDER BY <outer_order>];
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `(SELECT ...)` | The inner query that generates the temporary result set |
| `AS <alias>` | Mandatory correlation name for the derived table |
| `<outer_columns>` | Columns selected from the derived table |
| `<outer_condition>` | Filters applied to the derived table's result |

**Syntax Rules:**

- The derived table must be enclosed in parentheses.
- The alias (correlation name) is mandatory. The `AS` keyword is optional in some dialects (e.g., MySQL allows `FROM (SELECT ...) alias`), but the alias itself is required.
- All columns in the derived table must have unique names. If the inner query has duplicate column names, the derived table is invalid.
- The inner query cannot reference tables from the outer query in a correlated manner (in MySQL).
- The `ORDER BY` clause in the inner query is generally not useful unless the outer query uses `LIMIT`; optimisers often ignore it.

**Constraints and Limitations:**

- MySQL restricts derived tables from being correlated subqueries and from containing outer references.
- The alias cannot be omitted; every derived table needs a name.
- The derived table's columns cannot be referenced by the outer query unless they are named or aliased in the inner query.
- Some RDBMSs impose a maximum number of tables that can be merged (e.g., 61 in MySQL).

### Annotated Complete Code Examples

**Example 1: Basic Derived Table**

```sql
-- Setup: Create employees table
CREATE TABLE employees (
    emp_id     SERIAL PRIMARY KEY,
    first_name VARCHAR(50),
    department VARCHAR(50),
    salary     NUMERIC(10, 2)
);

INSERT INTO employees (first_name, department, salary) VALUES
    ('Alice', 'Engineering', 95000.00),
    ('Bob',   'Marketing',   72000.00),
    ('Carol', 'Engineering', 105000.00),
    ('David', 'Sales',       68000.00),
    ('Eve',   'Marketing',   88000.00);

-- Query: Use a derived table to find departments with average salary > 80,000
SELECT dept, avg_sal
FROM (
    SELECT department AS dept,
           AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department
) AS dept_summary
WHERE avg_sal > 80000
ORDER BY avg_sal DESC;

-- Expected Output:
--  dept        |  avg_sal
-- -------------+----------
--  Engineering | 100000.00
--  Marketing   |  80000.00
```

**Why this output occurs:** The inner query groups employees by department and computes the average salary per department. This result becomes the derived table `dept_summary`. The outer query then filters the derived table with `WHERE avg_sal > 80000`, a condition that could not have been applied in the inner query using `WHERE` because `avg_sal` is an aggregate result (it would require `HAVING`). The derived table enables filtering on the alias after aggregation.

**Example 2: Pre-Aggregation Before Join**

```sql
-- Setup: Create orders table
CREATE TABLE orders (
    order_id    SERIAL PRIMARY KEY,
    customer_id INTEGER,
    order_total NUMERIC(10, 2),
    order_date  DATE
);

INSERT INTO orders (customer_id, order_total, order_date) VALUES
    (1, 250.00, '2024-01-15'), (1, 300.00, '2024-03-20'),
    (1, 150.00, '2024-06-10'), (2, 400.00, '2024-02-05'),
    (2, 350.00, '2024-04-18'), (3, 100.00, '2024-05-22');

-- Query: Join customers to pre-aggregated order totals
SELECT c.customer_name, o.total_orders, o.total_spent
FROM customers AS c
INNER JOIN (
    SELECT customer_id,
           COUNT(*)        AS total_orders,
           SUM(order_total) AS total_spent
    FROM orders
    GROUP BY customer_id
) AS o
    ON c.customer_id = o.customer_id
WHERE o.total_spent > 500
ORDER BY o.total_spent DESC;

-- Expected Output:
--  customer_name | total_orders | total_spent
-- ---------------+--------------+-------------
--  Alice         |            3 |      700.00
--  Bob           |            2 |      750.00
```

**Why this output occurs:** The derived table `o` pre-aggregates orders by customer, computing the count and total spend before the join. This avoids row multiplication during the join: without pre-aggregation, joining customers to individual orders and then aggregating would produce the same result but with more intermediate rows. The outer query filters on `total_spent > 500` and sorts the result.

### Real-World Cases

**Case 1: Monthly Sales Summary**

A retail analyst uses a derived table to pre-aggregate daily sales into monthly totals, then joins the result to a product dimension table to produce a monthly sales report by product category.

**Case 2: Top Customers by Revenue**

An e-commerce platform uses a derived table to rank customers by total revenue, then filters the outer query to show only the top 10 customers with a `WHERE` condition on the pre-computed total.

### References

- MySQL 5.7 Reference Manual — Derived Tables - https://docs.oracle.com/cd/E17952_01/mysql-5.7-en/derived-tables.html
- MySQL 8.0 Reference Manual — Derived Tables - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/derived-tables.html
- Microsoft Learn — Write Queries That Use Derived Tables - https://learn.microsoft.com/en-gb/training/modules/create-tables-views-temporary-objects/6-write-queries-that-use-derived-tables
- Microsoft Learn — Write Queries That Use Derived Tables (Filipino) - https://learn.microsoft.com/fil-ph/training/modules/create-tables-views-temporary-objects/6-write-queries-that-use-derived-tables

---

## 2. Mandatory Alias and Column Naming

### Definitions

**Core Definition:** Every derived table must be assigned a correlation name (alias), and every column in the derived table must have a unique name, either inherited from the inner query or explicitly aliased.

**Technical Definition:** In the SQL standard, a `<derived table>` requires a `<correlation name>` (alias). MySQL documentation states: "The `[AS]` clause is mandatory because every table in a `FROM` clause must have a name. Any columns in the derived table must have unique names". Column aliases in the inner query determine the names by which the outer query references those columns. If a column in the inner query is an expression without an alias, the database assigns an implementation-defined name (often `?column?` in PostgreSQL or `Expr1001` in SQL Server), making the derived table difficult to use.

**Beginner-Friendly Explanation:** When you create a derived table, you must give it a name (alias) so the outer query can refer to it. Also, every column inside the derived table needs a name — if you use a calculation or function, you should give it an alias so the outer query can refer to it easily.

### Purposes

- To provide a stable, readable name for the outer query to reference the derived table.
- To ensure columns can be unambiguously referenced in the outer query.
- To improve query readability and maintainability.
- To comply with the SQL standard's requirement for table and column naming.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT <outer_columns>
FROM (
    SELECT column1 AS alias1,
           column2 AS alias2,
           expression AS alias3
    FROM source_table
) AS derived_alias;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `AS derived_alias` | Mandatory alias for the derived table |
| `column1 AS alias1` | Optional but recommended column aliases |
| `expression AS alias3` | Mandatory alias for computed columns if referenced by name |

**Syntax Rules:**

- The derived table alias is mandatory; omitting it causes a syntax error in most RDBMSs.
- Column aliases in the inner query are optional for simple column references but required for computed expressions if the outer query references them by name.
- If two columns in the derived table have the same name, the query fails.
- The alias must be a valid identifier.
- The `AS` keyword is optional in some dialects (e.g., MySQL allows `FROM (SELECT ...) alias`), but the alias itself is required.

**Constraints and Limitations:**

- The alias cannot be a reserved keyword unless quoted.
- Duplicate column names within the derived table cause an error.
- In MySQL, if the inner query uses `SELECT *`, the derived table inherits all column names; duplicates across joined tables can cause errors.

### Annotated Complete Code Examples

**Example 1: Derived Table with Column Aliases**

```sql
-- Query: Derived table with explicit column aliases
SELECT dept_name, avg_salary
FROM (
    SELECT department AS dept_name,
           AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
) AS dept_stats
WHERE avg_salary > 75000;

-- Expected Output:
--  dept_name   | avg_salary
-- -------------+------------
--  Engineering |  100000.00
--  Marketing   |   80000.00
```

**Why this output occurs:** The inner query aliases `department` as `dept_name` and `AVG(salary)` as `avg_salary`. The outer query references these aliases directly in its `SELECT` list and `WHERE` clause. Without the aliases, the outer query would not have a stable name for the computed average column.

**Example 2: Missing Alias Causes Error**

```sql
-- This query will FAIL: missing derived table alias
SELECT *
FROM (
    SELECT department, AVG(salary)
    FROM employees
    GROUP BY department
);
```

**Expected Error (MySQL):**
```
ERROR 1248 (42000): Every derived table must have its own alias
```

**Why this error occurs:** The derived table is not assigned an alias. MySQL requires every derived table in the `FROM` clause to have a name, just like every real table. The fix is to add `AS dept_stats` after the closing parenthesis.

### Real-World Cases

**Case 1: Readable Column Names in Reports**

A reporting query aliases computed columns in the derived table (e.g., `COUNT(*) AS headcount`, `SUM(revenue) AS total_revenue`) so the outer query and application code can reference them with meaningful names instead of generated placeholders.

**Case 2: Joining Multiple Derived Tables**

A query joins two derived tables, each with its own alias and column aliases, to combine pre-aggregated data from different sources. Clear aliases prevent ambiguity and improve readability.

### References

- MySQL 5.7 Reference Manual — Derived Tables (Alias Requirement) - https://docs.oracle.com/cd/E17952_01/mysql-5.7-en/derived-tables.html
- Microsoft Learn — Write Queries That Use Derived Tables (Alias Guidelines) - https://learn.microsoft.com/en-gb/training/modules/create-tables-views-temporary-objects/6-write-queries-that-use-derived-tables
- Stack Overflow — Derived Table Must Have Its Own Alias - https://stackoverflow.com/

---

## 3. Scope, Lifetime, and Visibility

### Definitions

**Core Definition:** The scope of a derived table is limited to the outer query in which it is defined; its lifetime is the duration of the outer query's execution, and it is not visible to other queries or sessions.

**Technical Definition:** A derived table is created at the time of execution of the outer query and goes out of scope when the outer query ends. It is not stored in the database, so no special security privileges are required to use it beyond the rights to select from the source objects. The derived table is visible only within the outer query that contains it; it cannot be referenced by other queries, and it does not persist beyond the execution of the statement.

**Beginner-Friendly Explanation:** A derived table is like a temporary note you write for yourself while solving a problem. Once you're done with the problem (the outer query finishes), the note is thrown away. It doesn't exist in the database, and no one else can see it.

### Purposes

- To provide a temporary, query-scoped result set without creating persistent objects.
- To avoid polluting the database with temporary tables for one-off queries.
- To ensure data isolation: the derived table is visible only to the query that created it.
- To avoid requiring `CREATE TABLE` or `CREATE VIEW` privileges.

### Syntax Rules and Structure

**Scope and Lifetime Rules:**

| Aspect | Behaviour |
|--------|-----------|
| Scope | Visible only within the outer query that defines it |
| Lifetime | Created at execution time; destroyed when the outer query ends |
| Persistence | Not stored in the database |
| Privileges | Only `SELECT` rights on source objects required |
| Visibility | Not visible to other queries, sessions, or users |

**Syntax Rules:**

- The derived table can only be referenced within the outer query.
- It cannot be referenced in other statements or from other queries.
- It is not a schema object; it does not appear in system catalogues.
- The optimizer may merge the derived table into the outer query or materialise it internally, but this is an implementation detail invisible to the user.

**Constraints and Limitations:**

- Derived tables cannot be indexed or have constraints.
- They cannot be referenced by name outside the query.
- They are not reusable across multiple queries (unlike views or temporary tables).

### Annotated Complete Code Examples

**Example 1: Derived Table Scope**

```sql
-- Query: Derived table is scoped to this SELECT only
SELECT dept, avg_sal
FROM (
    SELECT department AS dept,
           AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department
) AS dept_summary;

-- The alias dept_summary is NOT available outside this query.
-- The following would fail:
-- SELECT * FROM dept_summary;  -- ERROR: relation "dept_summary" does not exist
```

**Why this behaviour occurs:** The derived table `dept_summary` exists only for the duration of the `SELECT` statement. After the statement completes, the name `dept_summary` is no longer valid. To reuse the result, you would need to create a view or a temporary table.

### Real-World Cases

**Case 1: Ad-Hoc Analysis**

A data analyst runs a one-off query with a derived table to explore a subset of data. The derived table avoids creating a permanent object for a query that will not be reused.

**Case 2: Avoiding Temporary Table Clutter**

An application developer uses derived tables instead of temporary tables for intermediate results that are only needed within a single query, reducing database object management overhead.

### References

- Microsoft Learn — Write Queries That Use Derived Tables (Scope and Lifetime) - https://learn.microsoft.com/en-gb/training/modules/create-tables-views-temporary-objects/6-write-queries-that-use-derived-tables
- MySQL 5.7 Reference Manual — Derived Tables (Scope) - https://docs.oracle.com/cd/E17952_01/mysql-5.7-en/derived-tables.html

---

## 4. Derived Tables vs. Correlated Subqueries

### Definitions

**Core Definition:** A derived table is a non-correlated subquery in the `FROM` clause that produces a complete result set, while a correlated subquery references columns from the outer query and is evaluated per row.

**Technical Definition:** A derived table cannot be correlated in MySQL; it must be a self-contained query that can be evaluated independently of the outer query. A correlated subquery, by contrast, contains an outer reference and depends on the outer query for its values. Derived tables are evaluated once (or merged/materialised by the optimizer), while correlated subqueries are conceptually evaluated once per outer row.

**Beginner-Friendly Explanation:** A derived table is a complete, independent query that produces a result the outer query uses. A correlated subquery is a query that depends on the outer query and runs again for each row. Derived tables are like preparing a complete ingredient list before cooking; correlated subqueries are like asking a question for each item as you cook.

### Purposes

- To choose the right tool for the job: derived tables for set-based operations, correlated subqueries for row-by-row comparisons.
- To understand why derived tables cannot reference outer columns.
- To optimise queries by preferring derived tables (set-based) over correlated subqueries (row-by-row) when possible.
- To recognise when a correlated subquery can be rewritten as a derived table for better performance.

### Syntax Rules and Structure

**Derived Table:**

```sql
SELECT outer_columns
FROM (
    SELECT inner_columns
    FROM table2
    WHERE condition  -- No reference to outer query
) AS derived_alias
WHERE outer_condition;
```

**Correlated Subquery:**

```sql
SELECT outer_columns
FROM table1 t1
WHERE columnN operator (
    SELECT inner_column
    FROM table2 t2
    WHERE t2.column = t1.column  -- References outer query
);
```

**Component Breakdown:**

| Aspect | Derived Table | Correlated Subquery |
|--------|--------------|---------------------|
| Location | `FROM` clause | `WHERE`, `SELECT`, or `HAVING` clause |
| Correlation | Cannot reference outer query (MySQL) | Must reference outer query |
| Evaluation | Once (or merged/materialised) | Once per outer row |
| Performance | Generally set-based and efficient | Can be slow (N × M loops) |

**Syntax Rules:**

- Derived tables must be self-contained; correlated subqueries must reference at least one outer column.
- Derived tables require an alias; correlated subqueries do not (but table aliases are recommended for clarity).
- A correlated subquery can sometimes be rewritten as a derived table joined to the outer query for better performance.

**Constraints and Limitations:**

- MySQL derived tables cannot be correlated; this is a MySQL restriction, not a standard SQL restriction.
- Correlated subqueries can be decorrelated by the optimizer in some cases, but not always.
- Derived tables cannot reference columns from other tables in the same `SELECT` (in MySQL).

### Annotated Complete Code Examples

**Example 1: Derived Table vs. Correlated Subquery**

```sql
-- Derived table: pre-aggregate department averages, then join
SELECT e.first_name, e.salary, d.avg_sal
FROM employees e
INNER JOIN (
    SELECT department, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department
) AS d
    ON e.department = d.department
WHERE e.salary > d.avg_sal;

-- Correlated subquery: compute department average per row
SELECT e.first_name, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department = e.department
);
```

**Expected Output (both queries produce the same result):**

```
 first_name |  salary
------------+----------
 Alice      | 95000.00
 Carol      |105000.00
 Eve        | 88000.00
```

**Why the derived table version is often preferred:** The derived table version pre-aggregates department averages once and joins the result, avoiding repeated computation of the same average for each employee row. The correlated subquery recomputes the department average for every employee. Modern optimisers may decorrelate the subquery, but the derived table form is explicit and more portable.

### Real-World Cases

**Case 1: Performance Tuning**

A DBA rewrites a slow query with a correlated subquery as a derived table join, reducing execution time from minutes to seconds on a large employee table.

**Case 2: Readability**

A developer chooses a derived table over a correlated subquery because the set-based logic is easier to understand and maintain, especially when the derived table is reused in multiple parts of the outer query.

### References

- MySQL 5.7 Reference Manual — Derived Tables (Correlation Restriction) - https://docs.oracle.com/cd/E17952_01/mysql-5.7-en/derived-tables.html
- Microsoft Learn — Write Queries That Use Derived Tables (Comparison with Subqueries) - https://learn.microsoft.com/en-gb/training/modules/create-tables-views-temporary-objects/6-write-queries-that-use-derived-tables
- OneUptime Blog — How to Use Derived Tables Effectively in MySQL - https://github.com/OneUptime/blog/blob/master/posts/2026-03-31-mysql-how-to-use-derived-tables-effectively-in-mysql/README.md

---

## 5. Derived Tables vs. Common Table Expressions (CTEs)

### Definitions

**Core Definition:** A Common Table Expression (CTE) is a named temporary result set defined with a `WITH` clause that precedes the main query, while a derived table is an inline subquery in the `FROM` clause.

**Technical Definition:** CTEs use the syntax `WITH cte_name AS (SELECT ...) SELECT ... FROM cte_name`. Derived tables use `FROM (SELECT ...) AS alias`. Both produce temporary result sets, but CTEs are defined before the main query and can be referenced multiple times, while derived tables are defined inline and referenced once (unless the optimizer merges them). In MySQL 8.0, non-recursive CTEs are often described as "improved derived tables" because they offer better readability for multi-step queries.

**Beginner-Friendly Explanation:** A CTE is like giving a name to a subquery at the beginning of your query, so you can use it later. A derived table is like writing the subquery directly in the `FROM` clause. CTEs are usually easier to read, especially when you have multiple steps. Derived tables are good for simple, one-off inline transformations.

### Purposes

- To choose the appropriate construct for query organisation and readability.
- To understand when a CTE is preferable (multi-step, reusable logic) versus a derived table (simple inline transformation).
- To migrate between derived tables and CTEs as query complexity grows.
- To leverage CTE-specific features such as recursive queries.

### Syntax Rules and Structure

**Derived Table:**

```sql
SELECT outer_columns
FROM (
    SELECT inner_columns
    FROM source_table
    GROUP BY grouping_columns
) AS derived_alias;
```

**Common Table Expression (CTE):**

```sql
WITH cte_name AS (
    SELECT inner_columns
    FROM source_table
    GROUP BY grouping_columns
)
SELECT outer_columns
FROM cte_name;
```

**Component Breakdown:**

| Aspect | Derived Table | CTE |
|--------|--------------|-----|
| Definition location | Inline in `FROM` | Before the main query (`WITH`) |
| Reusability | Typically once | Multiple times |
| Readability | Good for simple cases | Better for complex, multi-step queries |
| Recursion | Not supported | Supported (`WITH RECURSIVE`) |
| Nesting | Can be nested (harder to read) | Linear, easier to follow |
| Standard | Yes | Yes (SQL:1999) |

**Syntax Rules:**

- CTEs are defined with `WITH name AS (query)` before the main `SELECT`.
- Multiple CTEs can be defined, separated by commas.
- A CTE can reference previously defined CTEs.
- Derived tables are defined inline and cannot be reused by name unless the optimizer merges them.
- CTEs can be recursive; derived tables cannot.

**Constraints and Limitations:**

- CTEs may be materialised by the optimizer, potentially affecting performance.
- Derived tables are generally better for simple, one-off transformations.
- MySQL supports CTEs from version 8.0; older versions require derived tables.
- Recursive CTEs are not supported in all RDBMSs (SQLite supports them from 3.8.3).

### Annotated Complete Code Examples

**Example 1: Derived Table vs. CTE for the Same Query**

**Derived Table (nested, harder to read):**

```sql
SELECT *
FROM (
    SELECT department, AVG(salary) AS avg_salary
    FROM (
        SELECT * FROM employees WHERE active = 1
    ) AS active_emps
    GROUP BY department
) AS dept_avgs
WHERE avg_salary > 50000;
```

**Equivalent CTE (cleaner, easier to follow):**

```sql
WITH active_emps AS (
    SELECT * FROM employees WHERE active = 1
),
dept_avgs AS (
    SELECT department, AVG(salary) AS avg_salary
    FROM active_emps
    GROUP BY department
)
SELECT * FROM dept_avgs WHERE avg_salary > 50000;
```

**Why the CTE is preferred:** The CTE version reads linearly, defining each step in order. The derived table version is nested ("tree-like" and "turned inside-out"), making it harder to follow the logical flow. For queries with more than one level of nesting, CTEs significantly improve readability.

### Real-World Cases

**Case 1: Multi-Step Data Transformation**

A data engineer uses CTEs to define a pipeline of transformations (extract → filter → aggregate → rank), each referencing the previous one. The same pipeline expressed with nested derived tables would be difficult to read and debug.

**Case 2: Simple Inline Transformation**

A developer uses a derived table for a quick, one-off pre-aggregation before a join, avoiding the overhead of defining a CTE for a single use.

### References

- OneUptime Blog — Derived Table vs. CTE - https://github.com/OneUptime/blog/blob/master/posts/2026-03-31-mysql-how-to-use-derived-tables-effectively-in-mysql/README.md
- MySQL 8.0 Labs — Common Table Expressions (CTEs) - https://dev.mysql.com/
- Apress — Common Table Expressions vs. Derived Tables - https://www.apress.com/
- SQL Engineering Handbook — Derived Tables - https://github.com/theammarngp-makes/SQL-Engineering-Handbook

---

## 6. Derived Tables vs. Views and Temporary Tables

### Definitions

**Core Definition:** A view is a stored, named query that persists in the database schema; a temporary table is a physical table that exists for a session or transaction; a derived table is a query-scoped result set that exists only for the duration of a single query.

**Technical Definition:** Views are schema objects created with `CREATE VIEW` and stored in the database catalogue. Temporary tables are created with `CREATE TEMPORARY TABLE` (or `CREATE TABLE #temp` in SQL Server) and persist for the session or transaction. Derived tables are not stored objects; they are inline subqueries that exist only during the execution of the outer query.

**Beginner-Friendly Explanation:** A view is like saving a query in the database so you can reuse it later. A temporary table is like a scratch table that exists for a while. A derived table is like a query result you use immediately and then discard — it doesn't persist at all.

### Purposes

- To choose the right level of persistence for a given task.
- To avoid unnecessary database objects when a one-off query suffices.
- To understand the trade-offs between persistence, reusability, and performance.
- To migrate between constructs as requirements change.

### Syntax Rules and Structure

**Comparison Table:**

| Aspect | Derived Table | View | Temporary Table |
|--------|--------------|------|-----------------|
| Persistence | None (query-scoped) | Stored in schema | Session/transaction |
| Creation syntax | Inline in `FROM` | `CREATE VIEW` | `CREATE TEMPORARY TABLE` |
| Reusability | One query | Many queries | Many queries in session |
| Privileges | `SELECT` on source | `CREATE VIEW` + `SELECT` | `CREATE TEMPORARY TABLE` |
| Indexes | Not possible | Not directly (indexed views exist) | Possible |
| Visibility | Query only | All users with access | Session only |

**Syntax Rules:**

- Derived tables are defined inline and cannot be reused by name.
- Views are created once and can be queried by name in any subsequent query.
- Temporary tables are created with a `CREATE` statement and dropped automatically at session end (or explicitly).
- Derived tables require no special privileges beyond `SELECT` on source objects.

**Constraints and Limitations:**

- Derived tables cannot have indexes or constraints.
- Views can be indexed (indexed views) in some RDBMSs, but with restrictions.
- Temporary tables consume session resources and may need explicit cleanup.
- Derived tables are the most lightweight option but offer the least reusability.

### Annotated Complete Code Examples

**Example 1: Derived Table vs. View vs. Temporary Table**

```sql
-- Derived table: exists only for this query
SELECT dept, avg_sal
FROM (
    SELECT department AS dept, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department
) AS dept_stats;

-- View: persists in the schema
CREATE VIEW dept_stats_view AS
SELECT department AS dept, AVG(salary) AS avg_sal
FROM employees
GROUP BY department;

SELECT * FROM dept_stats_view;

-- Temporary table: persists for the session
CREATE TEMPORARY TABLE dept_stats_temp AS
SELECT department AS dept, AVG(salary) AS avg_sal
FROM employees
GROUP BY department;

SELECT * FROM dept_stats_temp;

-- Cleanup
DROP VIEW dept_stats_view;
DROP TABLE dept_stats_temp;
```

**Why these differences matter:** The derived table is the simplest and requires no cleanup. The view is reusable across queries and sessions, making it suitable for shared reporting logic. The temporary table is useful for multi-step processing within a session and can be indexed for performance.

### Real-World Cases

**Case 1: Shared Reporting Logic**

A BI team creates a view for a commonly used aggregation (e.g., monthly sales summary) so that multiple reports can reference it without duplicating the aggregation logic.

**Case 2: Session-Scoped Data Processing**

An ETL process creates a temporary table to stage intermediate results that will be used by multiple subsequent queries within the same session.

**Case 3: One-Off Analysis**

A data analyst uses a derived table for a quick, one-off query, avoiding the overhead of creating and cleaning up a persistent object.

### References

- Microsoft Learn — Write Queries That Use Derived Tables (Comparison with Views) - https://learn.microsoft.com/en-gb/training/modules/create-tables-views-temporary-objects/6-write-queries-that-use-derived-tables
- MySQL 5.7 Reference Manual — Derived Tables - https://docs.oracle.com/cd/E17952_01/mysql-5.7-en/derived-tables.html

---

## 7. Optimisation: Merging vs. Materialisation

### Definitions

**Core Definition:** Merging and materialisation are the two strategies the query optimizer uses to handle derived tables: merging combines the derived table into the outer query, while materialisation computes the derived table as a temporary internal table.

**Technical Definition:** According to MySQL documentation, the optimizer handles derived table references using two strategies: merge the derived table into the outer query block, or materialise the derived table to an internal temporary table. Merging effectively inlines the derived table's query into the outer query, eliminating the need for a separate temporary table. Materialisation computes the derived table once and stores it in a temporary table, which is then used by the outer query. The optimizer chooses between these strategies based on cost estimates.

**Beginner-Friendly Explanation:** The database has two ways to handle a derived table: it can either "merge" it (combine it directly with the outer query, like inlining a function) or "materialise" it (compute the result and store it in a temporary table, then use that). Merging is usually faster because it avoids creating a temporary table, but materialisation is sometimes necessary (e.g., when the derived table uses aggregation).

### Purposes

- To understand how the optimizer processes derived tables.
- To write queries that enable efficient merging when possible.
- To use optimizer hints to influence the strategy when needed.
- To diagnose performance issues related to derived table materialisation.

### Syntax Rules and Structure

**Optimizer Strategies:**

| Strategy | How It Works | When Used |
|----------|-------------|-----------|
| Merge | Derived table is inlined into the outer query | Simple derived tables without aggregation |
| Materialise | Derived table is computed and stored in a temporary table | Complex derived tables with aggregation, `LIMIT`, or when merging would exceed table limits |

**Optimizer Hints (MySQL):**

```sql
-- Force merging
SELECT /*+ MERGE(derived_alias) */ ...
FROM (SELECT ...) AS derived_alias;

-- Prevent merging (force materialisation)
SELECT /*+ NO_MERGE(derived_alias) */ ...
FROM (SELECT ...) AS derived_alias;
```

**Component Breakdown:**

| Hint | Purpose |
|------|---------|
| `MERGE(derived_alias)` | Instructs the optimizer to merge the derived table |
| `NO_MERGE(derived_alias)` | Instructs the optimizer to materialise the derived table |
| `optimizer_switch` flag | System variable to control merging globally |

**Syntax Rules:**

- Merging is the default when possible; materialisation is used when merging is not feasible.
- The `derived_merge` flag of the `optimizer_switch` system variable controls whether merging is attempted.
- Derived condition pushdown (MySQL 8.0.22+) can push outer `WHERE` conditions into the derived table before materialisation, reducing the number of rows processed.

**Constraints and Limitations:**

- Merging is not possible when the derived table uses aggregation, window functions, `LIMIT`, or `UNION`.
- If merging would result in an outer query block referencing more than 61 base tables, the optimizer chooses materialisation instead.
- Materialisation can be slower but is sometimes unavoidable.

### Annotated Complete Code Examples

**Example 1: Merging vs. Materialisation**

```sql
-- Query with a simple derived table (eligible for merging)
SELECT *
FROM (SELECT * FROM t1) AS derived_t1;

-- With merging, executed similar to:
SELECT * FROM t1;

-- Query with aggregation (requires materialisation)
SELECT dept, avg_sal
FROM (
    SELECT department AS dept, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department
) AS dept_stats
WHERE avg_sal > 50000;

-- The optimizer materialises the derived table, then filters the result.
-- With derived condition pushdown (MySQL 8.0.22+), the outer WHERE
-- condition is pushed into the derived table as a HAVING clause:
-- SELECT dept, avg_sal FROM (
--     SELECT department AS dept, AVG(salary) AS avg_sal
--     FROM employees
--     GROUP BY department
--     HAVING avg_sal > 50000
-- ) AS dept_stats;
```

**Why this matters:** The simple derived table is merged into the outer query, eliminating the overhead of a temporary table. The aggregated derived table must be materialised, but derived condition pushdown reduces the number of rows processed by applying the outer filter earlier.

### Real-World Cases

**Case 1: Performance Tuning with Optimizer Hints**

A DBA notices that a derived table is being materialised unnecessarily, causing a performance bottleneck. They add a `MERGE` hint to force merging, and the query speeds up significantly.

**Case 2: Derived Condition Pushdown**

A developer writes a query with a derived table that aggregates sales by product, then filters for products with total sales above a threshold. MySQL 8.0.22+ automatically pushes the threshold condition into the derived table as a `HAVING` clause, reducing the rows that need to be materialised.

### References

- MySQL 8.0 Reference Manual — Optimizing Derived Tables with Merging or Materialization - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/derived-table-optimization.html
- MySQL 8.0 Reference Manual — Derived Condition Pushdown Optimization - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/derived-condition-pushdown-optimization.html
- Alibaba Cloud — Column Pruning Optimization for Derived Tables and Views - https://www.alibabacloud.com/

---

## 8. Common Use Cases and Patterns

### Definitions

**Core Definition:** Derived tables are used in several recurring patterns: pre-aggregation before joins, filtering on aggregate results, multi-step transformations, and ranking/limiting within groups.

**Technical Definition:** These patterns leverage the derived table's ability to produce a complete result set that the outer query can filter, join, or further transform. Common patterns include: (1) pre-aggregating a table before joining to avoid row multiplication, (2) applying `WHERE` conditions to aggregate results that would otherwise require `HAVING`, (3) chaining multiple derived tables for multi-step transformations, and (4) using derived tables to work around the logical processing order of SQL clauses.

**Beginner-Friendly Explanation:** Derived tables are useful in several common situations: when you want to aggregate data before joining it to another table, when you want to filter on a calculated value (which you can't do directly in `WHERE`), and when you need to perform multiple transformations in sequence.

### Purposes

- To pre-aggregate data before joining to reduce the size of the join.
- To filter on aggregate results using the outer query's `WHERE` clause.
- To break complex transformations into manageable, sequential steps.
- To work around SQL's logical processing order (e.g., using aliases in `WHERE`).
- To compute rankings and select top-N rows within groups.

### Syntax Rules and Structure

**Pattern 1: Pre-Aggregation Before Join**

```sql
SELECT c.customer_name, o.total_orders, o.total_spent
FROM customers c
INNER JOIN (
    SELECT customer_id,
           COUNT(*) AS total_orders,
           SUM(order_total) AS total_spent
    FROM orders
    GROUP BY customer_id
) AS o ON c.customer_id = o.customer_id
WHERE o.total_spent > 500;
```

**Pattern 2: Filter After Aggregation**

```sql
SELECT dept, total_sales
FROM (
    SELECT department AS dept, SUM(sales) AS total_sales
    FROM sales_data
    GROUP BY department
) AS agg
WHERE total_sales > 100000
ORDER BY total_sales DESC;
```

**Pattern 3: Multi-Step Transformation**

```sql
SELECT category, avg_price, price_tier
FROM (
    SELECT category, AVG(price) AS avg_price
    FROM products
    GROUP BY category
) AS category_avgs
CROSS JOIN (
    SELECT AVG(price) + STDDEV(price) AS high_threshold,
           AVG(price) AS mid_threshold
    FROM products
) AS thresholds
WHERE avg_price >= mid_threshold;
```

**Component Breakdown:**

| Pattern | Purpose |
|---------|---------|
| Pre-aggregation before join | Reduces join size and avoids row multiplication |
| Filter after aggregation | Applies `WHERE` to aggregate results (which would require `HAVING` otherwise) |
| Multi-step transformation | Chains multiple derived tables for sequential logic |

**Syntax Rules:**

- Each pattern follows the general derived table syntax: `FROM (SELECT ...) AS alias`.
- The outer query can reference derived table columns by their aliases.
- Multiple derived tables can be joined or cross-joined in the outer query.

**Constraints and Limitations:**

- Multi-step transformations with nested derived tables can become difficult to read; CTEs are often preferred for complex chains.
- Pre-aggregation before join may change the result if not carefully designed (e.g., losing rows with no matches).

### Annotated Complete Code Examples

**Example 1: Pre-Aggregation Before Join (Avoiding Row Multiplication)**

```sql
-- Without derived table: join multiplies rows, then GROUP BY
-- With derived table: aggregate first, then join

SELECT c.customer_name, o.total_orders, o.total_spent
FROM customers c
INNER JOIN (
    SELECT customer_id,
           COUNT(*) AS total_orders,
           SUM(order_total) AS total_spent
    FROM orders
    WHERE created_at >= '2024-01-01'
    GROUP BY customer_id
) AS o ON o.customer_id = c.id
WHERE o.total_orders >= 5
ORDER BY o.total_spent DESC;

-- Expected Output:
--  customer_name | total_orders | total_spent
-- ---------------+--------------+-------------
--  Alice         |            3 |      700.00
--  Bob           |            2 |      750.00
```

**Why this pattern is useful:** Without the derived table, joining customers to individual orders and then grouping would produce the same result but with more intermediate rows. Pre-aggregating reduces the join size and improves performance, especially when orders is a large table.

**Example 2: Filter After Aggregation (Using WHERE on Aggregate Results)**

```sql
-- Cannot use aggregate alias in WHERE at the same level:
-- This would FAIL:
-- SELECT department, SUM(sales) AS total_sales
-- FROM sales_data
-- WHERE total_sales > 100000  -- ERROR: column "total_sales" does not exist
-- GROUP BY department;

-- Correct: use a derived table
SELECT dept, total_sales
FROM (
    SELECT department AS dept, SUM(sales) AS total_sales
    FROM sales_data
    GROUP BY department
) AS agg
WHERE total_sales > 100000
ORDER BY total_sales DESC;

-- Expected Output:
--  dept        | total_sales
-- -------------+-------------
--  Engineering |   250000.00
--  Marketing   |   150000.00
```

**Why this pattern is useful:** SQL's logical processing order evaluates `WHERE` before `SELECT` and `GROUP BY`, so column aliases from the `SELECT` list are not available in `WHERE`. A derived table works around this limitation by computing the aggregate first, then applying the filter in the outer query.

### Real-World Cases

**Case 1: Customer Lifetime Value Report**

An e-commerce analyst uses a derived table to pre-aggregate each customer's total orders and total spend, then joins the result to a customer dimension table and filters for high-value customers.

**Case 2: Product Ranking**

A merchandising team uses a derived table to compute average price per category, then joins thresholds to classify products into price tiers (Budget, Mid-Range, Premium).

**Case 3: Monthly Sales Summary**

A finance team uses a derived table to aggregate daily transactions into monthly totals, then joins the result to a date dimension table for fiscal-year reporting.

### References

- OneUptime Blog — How to Use Derived Tables Effectively in MySQL - https://github.com/OneUptime/blog/blob/master/posts/2026-03-31-mysql-how-to-use-derived-tables-effectively-in-mysql/README.md
- Microsoft Learn — Write Queries That Use Derived Tables (Use Cases) - https://learn.microsoft.com/en-gb/training/modules/create-tables-views-temporary-objects/6-write-queries-that-use-derived-tables
- SAP Help — SQL-Derived Tables - https://help.sap.com/

---

## Summary Table of Derived Table Concepts

| Concept | Purpose | Key Limitation |
|---------|---------|----------------|
| Syntax & Structure | Define a temporary result set in `FROM` | Requires alias and unique column names |
| Mandatory Alias | Name the derived table for outer query reference | Omitting alias causes syntax error |
| Scope & Lifetime | Query-scoped, non-persistent result set | Cannot be reused outside the query |
| vs. Correlated Subqueries | Set-based, non-correlated | Cannot reference outer query (MySQL) |
| vs. CTEs | Inline vs. named, reusable | CTEs better for multi-step queries |
| vs. Views & Temp Tables | Lightweight, no persistence | No indexes, no reusability |
| Optimisation | Merge vs. materialise | Materialisation needed for aggregation |
| Use Cases | Pre-aggregation, filter-after-aggregate | Nested derived tables can be hard to read |

---

## Dialect Support for Derived Tables

| Feature | PostgreSQL | MySQL | SQL Server | Oracle | SQLite |
|---------|-----------|-------|------------|--------|--------|
| Derived Tables | ✅ | ✅ | ✅ | ✅ | ✅ |
| Mandatory Alias | ✅ | ✅ | ✅ | ✅ | ✅ |
| Column Aliases | ✅ | ✅ | ✅ | ✅ | ✅ |
| Correlated Derived Tables | ✅ (`LATERAL`) | ❌ | ✅ (`CROSS APPLY`) | ✅ (`LATERAL`) | ❌ |
| Derived Condition Pushdown | ✅ (optimizer) | ✅ (8.0.22+) | ✅ (optimizer) | ✅ (optimizer) | ❌ |
| Merge / Materialise Hints | ✅ (optimizer) | ✅ (`MERGE`/`NO_MERGE`) | ✅ (optimizer) | ✅ (optimizer) | ❌ |

**Note:** PostgreSQL and Oracle support lateral derived tables (subqueries in `FROM` that can reference earlier tables) using the `LATERAL` keyword. SQL Server uses `CROSS APPLY` and `OUTER APPLY` for similar functionality. MySQL does not support correlated derived tables.

---

## References

- MySQL 5.7 Reference Manual — Derived Tables - https://docs.oracle.com/cd/E17952_01/mysql-5.7-en/derived-tables.html
- MySQL 8.0 Reference Manual — Derived Tables - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/derived-tables.html
- MySQL 8.0 Reference Manual — Optimizing Derived Tables with Merging or Materialization - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/derived-table-optimization.html
- MySQL 8.0 Reference Manual — Derived Condition Pushdown Optimization - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/derived-condition-pushdown-optimization.html
- Microsoft Learn — Write Queries That Use Derived Tables - https://learn.microsoft.com/en-gb/training/modules/create-tables-views-temporary-objects/6-write-queries-that-use-derived-tables
- Microsoft Learn — Write Queries That Use Derived Tables (Filipino) - https://learn.microsoft.com/fil-ph/training/modules/create-tables-views-temporary-objects/6-write-queries-that-use-derived-tables
- SAP Help — SQL-Derived Tables - https://help.sap.com/
- OneUptime Blog — How to Use Derived Tables Effectively in MySQL - https://github.com/OneUptime/blog/blob/master/posts/2026-03-31-mysql-how-to-use-derived-tables-effectively-in-mysql/README.md
- SQL Engineering Handbook — Derived Tables - https://github.com/theammarngp-makes/SQL-Engineering-Handbook
- Alibaba Cloud — Column Pruning Optimization for Derived Tables and Views - https://www.alibabacloud.com/
- Stack Overflow — Derived Table Must Have Its Own Alias - https://stackoverflow.com/
- ISO/IEC 9075-2 — SQL Standard (Derived Tables) - https://jtc1sc32.org/doc/N0251-0300/32N0254.pdf