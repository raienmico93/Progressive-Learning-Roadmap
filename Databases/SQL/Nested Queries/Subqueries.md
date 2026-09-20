# SQL Subquery Fundamentals: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A SQL subquery is a `SELECT` statement nested within another SQL statement (the outer query), used to return data that the outer query uses as a condition, a value, or a table source.

**Technical Definition:** Per the SQL standard (ISO/IEC 9075-2), a subquery is a `<query expression>` enclosed in parentheses that appears within a `<query specification>`, `<predicate>`, or other SQL construct. The subquery is evaluated to produce a result set that the outer query consumes. Subqueries are classified by the shape of their result: a scalar subquery returns exactly one row and one column; a row subquery returns exactly one row but may return multiple columns; a table subquery returns a set of rows and columns. Subqueries may be self-contained (independent of the outer query) or correlated (referencing columns from the outer query).

**Beginner-Friendly Explanation:** A subquery is a query inside another query. The inner query runs first (or conceptually first), and its result is used by the outer query. It's like asking a question to get an answer that you then use to ask another question. For example, "Find employees who earn more than the average salary" — the subquery calculates the average, and the outer query finds employees above that average.

### Key Characteristics

- **Nested structure:** A subquery is enclosed in parentheses and embedded within an outer query.
- **Result-driven classification:** Subqueries are categorised by the shape of their result set: scalar (one value), single-row (one row), multi-row (multiple rows), or correlated (dependent on the outer query).
- **Evaluation order:** Self-contained subqueries are evaluated once, before the outer query. Correlated subqueries are re-evaluated for each row of the outer query.
- **Flexible placement:** Subqueries can appear in the `SELECT` list, `FROM` clause, `WHERE` clause, `HAVING` clause, and other SQL clauses.
- **Operator compatibility:** The operators used with a subquery depend on its result shape: scalar subqueries use comparison operators (`=`, `>`, `<`); multi-row subqueries use `IN`, `ANY`, `ALL`, `EXISTS`.
- **Dialect variations:** Column aliasing requirements, row constructor support, and correlated subquery optimisations vary across RDBMSs.

### Prerequisites

- **Basic SQL knowledge:** Familiarity with `SELECT`, `FROM`, and `WHERE` clauses.
- **Comparison operators:** Understanding of `=`, `>`, `<`, `>=`, `<=`, `<>`.
- **Logical operators:** Knowledge of `IN`, `ANY`, `ALL`, `EXISTS`, `NOT EXISTS`.
- **Table aliasing:** Ability to use table aliases to distinguish tables in correlated subqueries.

### Related Programming Areas

- **Data Analysis and Business Intelligence:** Subqueries enable complex filtering and comparison against aggregate values.
- **Application Development:** Backend services use subqueries for conditional data retrieval and hierarchical queries.
- **Data Engineering (ETL):** Subqueries are used in transformations, filtering, and data validation.
- **Database Administration:** DBAs use subqueries to identify data anomalies and perform audits.

### Core Concepts / Features

1. Definition of Subqueries
2. Scalar Subqueries
3. Single-Row Subqueries
4. Multi-Row Subqueries
5. Correlated Subqueries

---

## 1. Definition of Subqueries

### Definitions

**Core Definition:** A subquery (also called an inner query or nested query) is a `SELECT` statement embedded within another SQL statement, returning a result that the outer query uses.

**Technical Definition:** A subquery is a `<query expression>` enclosed in parentheses that appears inside an outer `<query specification>`, `<predicate>`, or other SQL construct. The subquery is evaluated to produce a result set, and the outer query consumes that result. Subqueries are classified by the shape of their result: a scalar subquery returns exactly one row and one column; a row subquery returns exactly one row but may return multiple columns; a table subquery returns a set of rows and columns. Subqueries may be self-contained (independent of the outer query) or correlated (referencing columns from the outer query).

**Beginner-Friendly Explanation:** A subquery is a query inside another query. Think of it as a question within a question: you ask the inner query to get a value or a set of values, and then the outer query uses that result to decide what to return.

### Purposes

- To break complex problems into smaller, manageable pieces.
- To filter data based on the result of another query.
- To compare values against aggregate results (e.g., average, maximum).
- To check for existence or absence of related rows.
- To derive values for calculations without needing a separate query.
- To create derived tables in the `FROM` clause.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT column1, column2
FROM main_table
WHERE columnN operator (
    SELECT expression 
    FROM subquery_table 
    WHERE condition
);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `main_table` | The outer query's source table |
| `columnN` | The column compared against the subquery result |
| `operator` | Comparison or set operator (`=`, `>`, `IN`, `EXISTS`, etc.) |
| `(SELECT ...)` | The subquery, enclosed in parentheses |
| `subquery_table` | The subquery's source table |

**Syntax Rules:**

- A subquery must be enclosed in parentheses.
- The subquery must be placed on the right side of the comparison operator.
- The `ORDER BY` clause cannot be used in a subquery unless `TOP`/`LIMIT` is also specified.
- Column aliases or fully qualified names should be used to prevent ambiguity in correlated subqueries.
- Subqueries can be nested within other subqueries.

**Constraints and Limitations:**

- Subqueries in the `SELECT` clause must be scalar (one row, one column).
- Subqueries in the `FROM` clause must have a table alias.
- Correlated subqueries can be resource-intensive because they are evaluated per outer row.
- Some RDBMSs restrict `ORDER BY` in subqueries.

### Annotated Complete Code Examples

**Example 1: Subquery in WHERE Clause**

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
    ('Eve',   'Engineering', 88000.00);

-- Query: Find employees earning more than the average salary
SELECT first_name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary) 
    FROM employees
);

-- Expected Output:
--  first_name |   salary
-- ------------+-----------
--  Alice      |  95000.00
--  Carol      | 105000.00
```

**Why this output occurs:** The subquery `(SELECT AVG(salary) FROM employees)` computes the average salary across all employees (85,600). The outer query then filters employees whose salary exceeds this average. Alice (95,000) and Carol (105,000) qualify; Bob (72,000), David (68,000), and Eve (88,000) do not.

### Real-World Cases

**Case 1: Dynamic Filtering**

A sales dashboard uses a subquery to find products priced above the average product price, enabling dynamic comparison without hardcoding values.

**Case 2: Data Validation**

A data engineer uses a subquery to find records in a staging table that do not exist in a master table, identifying data quality issues.

### References

- Microsoft Learn — Understand Subqueries - https://learn.microsoft.com/en-gb/training/modules/write-subqueries/2-understand-subqueries
- Microsoft Learn — Use Scalar or Multi-Valued Subqueries - https://learn.microsoft.com/en-us/training/modules/write-subqueries/2-understand-subqueries
- Educative — SQL Subquery: A Complete Guide - https://www.educative.io/blog/sql-subquery
- Snowflake — Working with Subqueries - https://docs.snowflake.com/en/user-guide/queries-subqueries

---

## 2. Scalar Subqueries

### Definitions

**Core Definition:** A scalar subquery is a subquery that returns exactly one row and one column — a single value.

**Technical Definition:** A scalar subquery is an ordinary `SELECT` query in parentheses that returns exactly one row with one column. The query is executed, and the single returned value is used in the surrounding value expression. If the subquery returns zero rows, the result is `NULL`. If it returns more than one row, the database raises an error (or the behaviour is implementation-defined).

**Beginner-Friendly Explanation:** A scalar subquery gives you exactly one value — like a single number or a single name. You can use it anywhere you would use a literal value. For example, you can put it in a calculation, a comparison, or a `SELECT` list.

### Purposes

- To provide a single value for use in an expression or comparison.
- To compute aggregate values (e.g., average, maximum) for use in the outer query.
- To avoid hardcoding values that depend on the data.
- To retrieve a specific value from another table based on a condition.
- To use in `SELECT`, `WHERE`, and `HAVING` clauses.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT column1, (SELECT single_value FROM table2 WHERE condition) AS derived_column
FROM table1
WHERE columnN operator (SELECT single_value FROM table2 WHERE condition);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `(SELECT single_value ...)` | Scalar subquery returning one row and one column |
| `operator` | Comparison operator (`=`, `>`, `<`, etc.) |
| `derived_column` | Alias for the scalar result in the `SELECT` list |

**Syntax Rules:**

- The subquery must return exactly one column and at most one row.
- If the subquery returns no rows, the scalar result is `NULL`.
- If the subquery returns more than one row, an error occurs (e.g., "subquery returned more than 1 row").
- Scalar subqueries can be used in `SELECT`, `WHERE`, `HAVING`, and `ORDER BY` clauses.
- Parentheses are required even when the subquery is an operand of a function.

**Constraints and Limitations:**

- Scalar subqueries cannot return multiple columns.
- Using a scalar subquery in the `SELECT` list causes it to be evaluated once per outer row if it is correlated.
- Performance can degrade if the scalar subquery is correlated and the outer table is large.

### Annotated Complete Code Examples

**Example 1: Scalar Subquery in SELECT List**

```sql
-- Query: Show each employee's salary and the company-wide average
SELECT first_name,
       salary,
       (SELECT AVG(salary) FROM employees) AS company_avg
FROM employees
ORDER BY first_name;

-- Expected Output:
--  first_name |   salary  |     company_avg
-- ------------+-----------+--------------------
--  Alice      |  95000.00 | 85600.000000000000
--  Bob        |  72000.00 | 85600.000000000000
--  Carol      | 105000.00 | 85600.000000000000
--  David      |  68000.00 | 85600.000000000000
--  Eve        |  88000.00 | 85600.000000000000
```

**Why this output occurs:** The scalar subquery `(SELECT AVG(salary) FROM employees)` computes the average salary (85,600). It is evaluated once and the same value is repeated for each row in the result. This is a self-contained scalar subquery, so it does not depend on the outer row.

**Example 2: Scalar Subquery in WHERE Clause**

```sql
-- Query: Find employees earning the maximum salary
SELECT first_name, salary
FROM employees
WHERE salary = (SELECT MAX(salary) FROM employees);

-- Expected Output:
--  first_name |  salary
-- ------------+----------
--  Carol      |105000.00
```

**Why this output occurs:** The scalar subquery computes the maximum salary (105,000). The outer query returns the employee(s) whose salary matches that maximum. Carol is the only employee with that salary.

### Real-World Cases

**Case 1: Pricing Benchmark**

An e-commerce analyst uses a scalar subquery to compare each product's price to the average price in its category: `SELECT product_name, price, (SELECT AVG(price) FROM products WHERE category = p.category) AS avg_category_price FROM products p`.

**Case 2: Most Recent Record**

A support system uses a scalar subquery to find the most recent ticket for each customer: `SELECT customer_id, (SELECT MAX(created_at) FROM tickets WHERE tickets.customer_id = customers.customer_id) AS last_ticket FROM customers`.

### References

- Microsoft Learn — Use Scalar or Multi-Valued Subqueries - https://learn.microsoft.com/en-us/training/modules/write-subqueries/2-understand-subqueries
- PostgreSQL Documentation — Scalar Subqueries - https://www.postgresql.org/docs/current/sql-expressions.html
- Amazon Redshift — Scalar Subqueries - https://docs.aws.amazon.com/redshift/latest/dg/r_scalar_subqueries.html
- Caltech — Subqueries in the SELECT Clause - https://courses.cms.caltech.edu/cs122/notes/notes6.pdf

---

## 3. Single-Row Subqueries

### Definitions

**Core Definition:** A single-row subquery is a subquery that returns zero or one row in its result set, typically used with comparison operators to compare a value from the outer query.

**Technical Definition:** A single-row subquery returns at most one row. It is used with comparison operators (`=`, `>`, `<`, `>=`, `<=`, `<>`) that expect a single value on the right-hand side. If the subquery returns zero rows, the comparison evaluates to `UNKNOWN` (treated as `FALSE` in `WHERE`). If it returns more than one row, an error occurs. A single-row subquery may return multiple columns if used in a row comparison (e.g., `(col1, col2) = (SELECT ...)`).

**Beginner-Friendly Explanation:** A single-row subquery gives you one row of data. You use it when you want to compare something against exactly one row from another query. For example, "Find the employee whose salary equals the maximum salary" — the subquery returns one value.

### Purposes

- To compare a value against a single value returned by another query.
- To filter rows based on a condition that yields exactly one row.
- To use in row comparisons where multiple columns are compared simultaneously.
- To retrieve a specific record's attributes for use in the outer query.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT column1, column2
FROM table1
WHERE columnN operator (SELECT columnX FROM table2 WHERE condition);
```

**Row Comparison Syntax:**

```sql
SELECT * FROM t1
WHERE (col1, col2) = (SELECT col3, col4 FROM t2 WHERE id = 10);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `(SELECT ...)` | Single-row subquery returning one row |
| `operator` | Comparison operator (`=`, `>`, `<`, etc.) |
| `(col1, col2)` | Row constructor for multi-column comparison |

**Syntax Rules:**

- The subquery must return at most one row; zero rows yield `UNKNOWN`.
- Comparison operators (`=`, `>`, `<`, etc.) expect a single value on the right.
- Row constructors can be used for multi-column comparisons.
- The number of columns in the row constructor must match the subquery's column count.
- MySQL supports row subqueries with operators `=`, `>`, `<`, `>=`, `<=`, `<>`, `!=`, `<=>`.

**Constraints and Limitations:**

- If the subquery returns more than one row, the database raises an error.
- Single-row subqueries cannot be used with `IN`, `ANY`, or `ALL` (those are for multi-row subqueries).
- Row comparisons with `NULL` values can yield `UNKNOWN`.

### Annotated Complete Code Examples

**Example 1: Single-Row Subquery with Comparison Operator**

```sql
-- Query: Find the employee with the highest salary
SELECT first_name, salary
FROM employees
WHERE salary = (
    SELECT MAX(salary) 
    FROM employees
);

-- Expected Output:
--  first_name |  salary
-- ------------+----------
--  Carol      | 105000.00
```

**Why this output occurs:** The subquery `(SELECT MAX(salary) FROM employees)` returns exactly one row with one value (105,000). The outer query compares each employee's salary to this single value and returns Carol, whose salary matches.

**Example 2: Row Subquery with Multi-Column Comparison**

```sql
-- Setup: Create a table with product data
CREATE TABLE products (
    product_id   INTEGER,
    product_name VARCHAR(50),
    category_id  INTEGER,
    price        NUMERIC(10, 2)
);

INSERT INTO products VALUES
    (1, 'Laptop', 10, 1200.00),
    (2, 'Phone',  10, 800.00),
    (3, 'Tablet', 20, 600.00);

-- Query: Find products with the same category and price as product_id 2
SELECT * FROM products
WHERE (category_id, price) = (
    SELECT category_id, price 
    FROM products 
    WHERE product_id = 2
);

-- Expected Output:
--  product_id | product_name | category_id | price
-- ------------+--------------+-------------+--------
--           2 | Phone        |          10 | 800.00
```

**Why this output occurs:** The subquery `(SELECT category_id, price FROM products WHERE product_id = 2)` returns a single row with two columns: `(10, 800.00)`. The outer query uses a row constructor `(category_id, price)` to compare against this row. Only the row with matching category and price is returned.

### Real-World Cases

**Case 1: Highest-Paid Employee per Department**

An HR system uses a single-row subquery to find the highest-paid employee in each department: `SELECT department_id, last_name, salary FROM employees x WHERE salary = (SELECT MAX(salary) FROM employees WHERE department_id = x.department_id)`.

**Case 2: Most Recent Order**

An e-commerce system uses a single-row subquery to find the most recent order for each customer: `SELECT customer_id, order_id, order_date FROM orders WHERE (customer_id, order_date) = (SELECT customer_id, MAX(order_date) FROM orders GROUP BY customer_id)`.

### References

- MySQL 8.0 Reference Manual — Row Subqueries - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/row-subqueries.html
- SAP Help — Single-Row and Multiple-Row Subqueries - https://help.sap.com/
- Microsoft Learn — Subqueries with Comparison Operators - https://learn.microsoft.com/en-us/previous-versions/sql/sql-server-2008-r2/ms189070(v=sql.105)
- Educative — SQL Subquery: A Complete Guide - https://www.educative.io/blog/sql-subquery

---

## 4. Multi-Row Subqueries

### Definitions

**Core Definition:** A multi-row subquery is a subquery that returns multiple rows (and typically one column), used with set operators such as `IN`, `ANY`, `ALL`, or `EXISTS`.

**Technical Definition:** A multi-row subquery returns a set of rows, much like a single-column table. It is used with operators that can handle multiple values: `IN` (equality to any value in the list), `ANY` (comparison with any value returned), `ALL` (comparison with all values returned), and `EXISTS`/`NOT EXISTS` (existence check). The subquery must return only one column when used with `IN`, `ANY`, or `ALL`.

**Beginner-Friendly Explanation:** A multi-row subquery gives you a list of values. You use it when you want to check if something is in that list (`IN`), compare against all of them (`ALL`), or compare against any of them (`ANY`).

### Purposes

- To filter rows based on membership in a set of values (`IN`).
- To compare a value against all values returned by a subquery (`ALL`).
- To compare a value against any value returned by a subquery (`ANY`).
- To check for the existence of related rows (`EXISTS`).
- To use a subquery result as a set for further filtering.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT column1, column2
FROM table1
WHERE columnN IN (SELECT columnX FROM table2 WHERE condition);

WHERE columnN operator ANY (SELECT columnX FROM table2 WHERE condition);

WHERE columnN operator ALL (SELECT columnX FROM table2 WHERE condition);

WHERE EXISTS (SELECT 1 FROM table2 WHERE condition);
```

**Component Breakdown:**

| Operator | Purpose |
|----------|---------|
| `IN` | True if the value matches any value in the subquery result |
| `ANY` | True if the comparison is true for at least one value in the result |
| `ALL` | True if the comparison is true for all values in the result |
| `EXISTS` | True if the subquery returns at least one row |

**Syntax Rules:**

- The subquery used with `IN`, `ANY`, or `ALL` must return exactly one column.
- `ANY` and `ALL` are used with comparison operators (`=`, `>`, `<`, etc.).
- `EXISTS` returns `TRUE` if the subquery returns one or more rows, `FALSE` otherwise.
- `NOT IN` is equivalent to `<> ALL`.
- `IN` is equivalent to `= ANY`.

**Constraints and Limitations:**

- `NOT IN` with a subquery that returns `NULL` can produce unexpected results (returns no rows).
- `ANY` and `ALL` cannot be used with `EXISTS`.
- Subqueries with `IN` can be less efficient than `EXISTS` in some RDBMSs.
- The subquery must return only one column when used with `IN`, `ANY`, or `ALL`.

### Annotated Complete Code Examples

**Example 1: Multi-Row Subquery with IN**

```sql
-- Setup: Create a departments table
CREATE TABLE departments (
    dept_id   INTEGER PRIMARY KEY,
    dept_name VARCHAR(50),
    location  VARCHAR(50)
);

INSERT INTO departments VALUES
    (1, 'Engineering', 'Manila'),
    (2, 'Marketing',   'Cebu'),
    (3, 'Sales',       'Manila');

-- Query: Find employees in departments located in Manila
SELECT first_name, department
FROM employees
WHERE department IN (
    SELECT dept_name
    FROM departments
    WHERE location = 'Manila'
);

-- Expected Output:
--  first_name | department
-- ------------+-------------
--  Alice      | Engineering
--  Carol      | Engineering
--  David      | Sales
```

**Why this output occurs:** The subquery `(SELECT dept_name FROM departments WHERE location = 'Manila')` returns two rows: 'Engineering' and 'Sales'. The outer query uses `IN` to check if each employee's department is in that list. Alice, Carol (Engineering) and David (Sales) match; Bob (Marketing) does not.

**Example 2: Multi-Row Subquery with ALL**

```sql
-- Query: Find products priced higher than all products in category 10
SELECT product_name, price
FROM products
WHERE price > ALL (
    SELECT price
    FROM products
    WHERE category_id = 10
);

-- Expected Output:
--  product_name | price
-- --------------+--------
-- (0 rows)
```

**Why this output occurs:** The subquery returns all prices in category 10 (1200, 800). The outer query uses `> ALL` to find products priced higher than both 1200 and 800. No product in the table is priced higher than 1200, so the result is empty.

### Real-World Cases

**Case 1: Department Filtering**

An HR system uses `IN` with a multi-row subquery to find employees in departments located in a specific city, as shown in the example above.

**Case 2: Price Comparison**

A retail analyst uses `> ALL` to find products priced higher than every product in a competitor's catalog, identifying premium items.

**Case 3: Existence Check**

An order system uses `EXISTS` to find customers who have placed at least one order: `SELECT customer_name FROM customers WHERE EXISTS (SELECT 1 FROM orders WHERE orders.customer_id = customers.customer_id)`.

### References

- MySQL 8.0 Reference Manual — Restrictions on Subqueries - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/subquery-restrictions.html
- MySQL 8.0 Reference Manual — Subqueries with ALL - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/all-subqueries.html
- SAP Help — Single-Row and Multiple-Row Subqueries - https://help.sap.com/
- Huawei Cloud — Multi-Row Subqueries - https://support.huaweicloud.com/
- Compile-N-Run — SQL Multiple-Row Subqueries - https://github.com/Compile-N-Run/Compile-N-Run

---

## 5. Correlated Subqueries

### Definitions

**Core Definition:** A correlated subquery is a subquery that references one or more columns from the outer query, making it dependent on the outer query for its values.

**Technical Definition:** A correlated subquery contains a reference to a table that also appears in the outer query. It cannot be run independently because it depends on the outer query for its values. The subquery is re-evaluated for each row processed by the outer query. Correlated subqueries are commonly used with `EXISTS`, `NOT EXISTS`, and comparison operators.

**Beginner-Friendly Explanation:** A correlated subquery is a subquery that uses information from the outer query. For each row the outer query processes, the subquery runs again with that row's values. It's like asking a question that depends on the current row you're looking at.

### Purposes

- To compare each row's value against a value computed from a related subset of data.
- To check for existence of related rows for each outer row.
- To perform row-by-row conditional logic.
- To find records that match a condition relative to their group.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT select_list
FROM table1 t_alias1
WHERE expr operator (
    SELECT column_list
    FROM table2 t_alias2
    WHERE t_alias1.column operator t_alias2.column
);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `t_alias1` | Alias for the outer table |
| `t_alias2` | Alias for the inner table |
| `t_alias1.column operator t_alias2.column` | Correlation condition linking inner and outer queries |

**Syntax Rules:**

- The subquery must reference at least one column from the outer query.
- Table aliases are required to distinguish the outer and inner tables.
- The subquery is evaluated once for each row of the outer query.
- Correlated subqueries can be used with `EXISTS`, `NOT EXISTS`, `IN`, `ANY`, `ALL`, and comparison operators.

**Constraints and Limitations:**

- Correlated subqueries are generally slower than self-contained subqueries because they are re-evaluated per row.
- They cannot be run independently from the outer query.
- Performance can be improved by rewriting as joins or using window functions in some cases.

### Annotated Complete Code Examples

**Example 1: Correlated Subquery with Comparison Operator**

```sql
-- Query: Find employees earning more than their department's average
SELECT department_id, last_name, salary
FROM employees x
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE x.department_id = department_id
)
ORDER BY department_id;

-- Expected Output (using a hypothetical department_id column):
--  department_id | last_name | salary
-- ---------------+-----------+--------
--  1             | Carol     | 105000
--  1             | Alice     | 95000
--  2             | Bob       | 72000
```

**Why this output occurs:** For each employee row, the correlated subquery computes the average salary for that employee's department. If the employee's salary exceeds their department's average, the row is returned. The subquery references `x.department_id` from the outer query, making it correlated.

**Example 2: Correlated Subquery with EXISTS**

```sql
-- Query: Find customers who have placed at least one order
SELECT customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);

-- Expected Output:
--  customer_name
-- ---------------
--  Alice
--  Bob
--  Carol
```

**Why this output occurs:** For each customer row, the `EXISTS` subquery checks whether any order exists with that customer's ID. If at least one order exists, `EXISTS` returns `TRUE` and the customer is returned. David (no orders) is excluded.

### Real-World Cases

**Case 1: Department Average Comparison**

An HR system uses a correlated subquery to find employees earning above their department's average salary, as shown in the example above.

**Case 2: Most Recent Order per Customer**

An e-commerce system uses a correlated subquery to find the most recent order for each customer: `SELECT SalesOrderID, CustomerID, OrderDate FROM SalesOrderHeader AS o1 WHERE SalesOrderID = (SELECT MAX(SalesOrderID) FROM SalesOrderHeader AS o2 WHERE o2.CustomerID = o1.CustomerID)`.

**Case 3: Authors with 100% Royalty**

A publishing database uses a correlated subquery to find authors who earn 100% royalty on a book: `SELECT au_lname FROM authors WHERE 100 IN (SELECT royaltyper FROM titleauthor WHERE titleauthor.au_id = authors.au_id)`.

### References

- Oracle Database SQL Language Reference — Correlated Subqueries - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/SELECT.html
- MySQL 8.0 Reference Manual — Correlated Subqueries - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/correlated-subqueries.html
- Microsoft Learn — Compare Rows with Correlated Subqueries - https://learn.microsoft.com/en-us/training/modules/write-subqueries/4-correlated-subqueries
- Microsoft Learn — Use Self-Contained or Correlated Subqueries - https://learn.microsoft.com/en-us/training/modules/write-subqueries/3-self-contained-subqueries

---

## Summary Table of Subquery Types

| Type | Returns | Operators | Evaluated | Can Run Independently? |
|------|---------|-----------|-----------|------------------------|
| Scalar | One row, one column | `=`, `>`, `<`, etc. | Once (self-contained) or per row (correlated) | Yes (if self-contained) |
| Single-Row | Zero or one row | `=`, `>`, `<`, etc. | Once (self-contained) | Yes |
| Multi-Row | Multiple rows | `IN`, `ANY`, `ALL`, `EXISTS` | Once (self-contained) | Yes |
| Correlated | Depends on outer row | `EXISTS`, `IN`, `=`, etc. | Per outer row | No |

---

## Dialect Support for Subquery Features

| Feature | PostgreSQL | MySQL | SQL Server | Oracle | SQLite |
|---------|-----------|-------|------------|--------|--------|
| Scalar Subqueries | ✅ | ✅ | ✅ | ✅ | ✅ |
| Single-Row Subqueries | ✅ | ✅ | ✅ | ✅ | ✅ |
| Multi-Row Subqueries | ✅ | ✅ | ✅ | ✅ | ✅ |
| Correlated Subqueries | ✅ | ✅ | ✅ | ✅ | ✅ |
| Row Constructors | ✅ | ✅ | ✅ | ✅ | ✅ |
| `ANY` / `ALL` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `EXISTS` / `NOT EXISTS` | ✅ | ✅ | ✅ | ✅ | ✅ |

---

## References

- Microsoft Learn — Understand Subqueries - https://learn.microsoft.com/en-gb/training/modules/write-subqueries/2-understand-subqueries
- Microsoft Learn — Use Scalar or Multi-Valued Subqueries - https://learn.microsoft.com/en-us/training/modules/write-subqueries/2-understand-subqueries
- Microsoft Learn — Compare Rows with Correlated Subqueries - https://learn.microsoft.com/en-us/training/modules/write-subqueries/4-correlated-subqueries
- MySQL 8.0 Reference Manual — Row Subqueries - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/row-subqueries.html
- MySQL 8.0 Reference Manual — Correlated Subqueries - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/correlated-subqueries.html
- MySQL 8.0 Reference Manual — Restrictions on Subqueries - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/subquery-restrictions.html
- Oracle Database SQL Language Reference — SELECT (Correlated Subqueries) - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/SELECT.html
- PostgreSQL Documentation — Subquery Expressions - https://www.postgresql.org/docs/current/functions-subquery.html
- PostgreSQL Documentation — Scalar Subqueries - https://www.postgresql.org/docs/current/sql-expressions.html
- Amazon Redshift — Scalar Subqueries - https://docs.aws.amazon.com/redshift/latest/dg/r_scalar_subqueries.html
- SAP Help — Single-Row and Multiple-Row Subqueries - https://help.sap.com/
- Educative — SQL Subquery: A Complete Guide - https://www.educative.io/blog/sql-subquery
- Snowflake — Working with Subqueries - https://docs.snowflake.com/en/user-guide/queries-subqueries
- Caltech — Subqueries in the SELECT Clause - https://courses.cms.caltech.edu/cs122/notes/notes6.pdf
- Huawei Cloud — Multi-Row Subqueries - https://support.huaweicloud.com/
- ISO/IEC 9075-2 — SQL Standard (Subqueries) - https://jtc1sc32.org/doc/N0251-0300/32N0254.pdf