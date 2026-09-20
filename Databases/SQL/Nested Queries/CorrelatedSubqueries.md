# SQL Correlated Subqueries: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A correlated subquery is a subquery that references one or more columns from the outer query, creating a dependency that causes the subquery to be evaluated once for each row processed by the outer query.

**Technical Definition:** Per the SQL standard (ISO/IEC 9075-2), a correlated subquery contains an outer reference — a column from a table that appears in a parent statement (the outer query) rather than in the subquery's own `FROM` clause. The correlated subquery cannot be evaluated independently of the outer query because it uses the values of the parent statement. The parent statement can be a `SELECT`, `UPDATE`, or `DELETE` statement in which the subquery is nested. Conceptually, the subquery is evaluated once for each row processed by the parent statement; however, the query optimizer may rewrite the query as a join or use other techniques to produce a semantically equivalent result.

**Beginner-Friendly Explanation:** A correlated subquery is a query inside another query that depends on the outer query. For each row the outer query looks at, the inner query runs again using that row's values. It's like asking a question that changes depending on which row you're currently examining. For example, "Find employees who earn more than the average salary of their own department" — the subquery recalculates the average for each employee's department.

### Key Characteristics

- **Outer reference dependency:** The subquery references at least one column from the outer query, making it dependent on the outer query for its values.
- **Per-row evaluation:** The subquery is conceptually evaluated once for each row processed by the outer query.
- **Cannot run independently:** A correlated subquery cannot be executed separately from the outer query, which complicates testing and debugging.
- **Nested-loop execution model:** Think of it as a nested loop — for each row in the outer query, the database evaluates the subquery using that row's values.
- **Optimizer transformations:** Modern query optimizers often transform correlated subqueries into equivalent joins internally for better performance.
- **EXISTS affinity:** Correlated subqueries are most commonly used with the `EXISTS` and `NOT EXISTS` predicates.
- **Dialect variations:** MySQL evaluates from inside to outside for scoping; Oracle allows up to 255 levels of nesting in the `WHERE` clause.

### Prerequisites

- **Basic SQL knowledge:** Familiarity with `SELECT`, `FROM`, and `WHERE` clauses.
- **Subquery fundamentals:** Understanding of scalar, single-row, and multi-row subqueries.
- **Table aliasing:** Ability to use table aliases (also called correlation names) to distinguish the outer and inner table references.
- **Logical operators:** Knowledge of `EXISTS`, `NOT EXISTS`, `IN`, `ANY`, and `ALL`.
- **Relational concepts:** Understanding of primary keys, foreign keys, and table relationships.

### Related Programming Areas

- **Data Analysis and Business Intelligence:** Correlated subqueries enable row-by-row comparisons and group-relative calculations.
- **Application Development:** Backend services use correlated subqueries for existence checks, hierarchical queries, and conditional updates.
- **Data Engineering (ETL):** Correlated subqueries are used in data validation, anti-join patterns, and row-level transformations.
- **Database Administration:** DBAs use correlated subqueries to identify anomalies and perform data audits.

### Core Concepts / Features

1. Correlated Subquery Fundamentals
2. Scalar Correlated Subqueries
3. Multi-Valued Correlated Subqueries
4. Correlated Subqueries with EXISTS / NOT EXISTS
5. Correlated Subqueries in UPDATE and DELETE Statements
6. Performance Optimisation and Decorrelation
7. Correlated vs. Non-Correlated Subqueries

---

## 1. Correlated Subquery Fundamentals

### Definitions

**Core Definition:** A correlated subquery is a nested `SELECT` statement that references a column from a table in the outer query, creating a logical dependency between the two queries.

**Technical Definition:** Oracle performs a correlated subquery when a nested subquery references a column from a table referred to in a parent statement one or more levels above the subquery or nested subquery. The parent statement can be a `SELECT`, `UPDATE`, or `DELETE` statement in which the subquery is nested. A correlated subquery conceptually is evaluated once for each row processed by the parent statement. However, the optimizer may choose to rewrite the query as a join or use some other technique to formulate a query that is semantically equivalent.

**Beginner-Friendly Explanation:** A correlated subquery is a subquery that uses information from the outer query. For each row the outer query processes, the subquery runs again with that row's values. It's like a conversation where the outer query asks, "What's the average salary for this department?" and the subquery answers based on the current department being examined.

### Purposes

- To answer multiple-part questions whose answers depend on the value in each row processed by the parent statement.
- To perform row-by-row comparisons that cannot be expressed with a single static value.
- To filter rows based on conditions relative to their own group or category.
- To check for the existence or absence of related rows for each outer row.
- To update or delete rows based on values from related tables.
- To implement anti-join patterns (finding rows with no matches) using `NOT EXISTS`.

### Syntax Rules and Structure

**Complete General Syntax (SELECT):**

```sql
SELECT select_list
FROM table1 t_alias1
WHERE expr operator (
    SELECT column_list
    FROM table2 t_alias2
    WHERE t_alias1.column operator t_alias2.column
);
```

**Complete General Syntax (UPDATE):**

```sql
UPDATE table1 t_alias1
SET column = (
    SELECT expr
    FROM table2 t_alias2
    WHERE t_alias1.column = t_alias2.column
);
```

**Complete General Syntax (DELETE):**

```sql
DELETE FROM table1 t_alias1
WHERE column operator (
    SELECT expr
    FROM table2 t_alias2
    WHERE t_alias1.column = t_alias2.column
);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `table1 t_alias1` | The outer query's table and its alias |
| `table2 t_alias2` | The subquery's table and its alias |
| `t_alias1.column` | The outer reference — a column from the outer query |
| `t_alias2.column` | The inner column compared against the outer reference |
| `operator` | Comparison operator (`=`, `>`, `<`, `<>`) or set operator (`IN`, `EXISTS`) |

**Syntax Rules:**

- The subquery must reference at least one column from the outer query (the outer reference).
- Table aliases (correlation names) are required to distinguish the outer and inner tables.
- The correlated column can be present only in the subquery's `WHERE` clause (and not in the `SELECT` list) for certain optimizer transformations.
- The correlated column must be resolved in the query block directly containing the subquery being considered for transformation.
- Scoping rule: MySQL evaluates from inside to outside; an alias defined in an inner query block shadows the same alias in an outer block.
- Oracle allows up to 255 levels of nesting in the `WHERE` clause.
- SQL Server allows up to 32 levels of nesting.

**Constraints and Limitations:**

- Correlated subqueries cannot be executed separately from the outer query, which complicates testing and debugging.
- Performance can degrade significantly on large tables because the subquery runs once per outer row (N × M loop).
- Not all correlated subqueries can be decorrelated (rewritten as joins) by the optimizer.
- A correlated column cannot be present in a nested scalar subquery in the `WHERE` clause for MySQL transformation eligibility.
- The subquery cannot contain window functions for MySQL transformation eligibility.

### Annotated Complete Code Examples

**Example 1: Basic Correlated Subquery with Comparison Operator**

```sql
-- Setup: Create employees table
CREATE TABLE employees (
    emp_id        INTEGER PRIMARY KEY,
    last_name     VARCHAR(50),
    department_id INTEGER,
    salary        NUMERIC(10, 2)
);

INSERT INTO employees VALUES
    (1, 'Alice',   10, 95000.00),
    (2, 'Bob',     20, 72000.00),
    (3, 'Carol',   10, 105000.00),
    (4, 'David',   30, 68000.00),
    (5, 'Eve',     20, 88000.00);

-- Query: Find employees earning more than their department's average
SELECT department_id, last_name, salary
FROM employees x
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE x.department_id = department_id
)
ORDER BY department_id;

-- Expected Output:
--  department_id | last_name |  salary
-- ---------------+-----------+----------
--             10 | Alice     | 95000.00
--             10 | Carol     |105000.00
--             20 | Eve       | 88000.00
```

**Why this output occurs:** For each row in the `employees` table (aliased as `x`), the correlated subquery computes the average salary for that employee's department by matching `x.department_id` with `department_id` in the subquery. Alice and Carol both earn above the Engineering average (100,000); Eve earns above the Marketing average (80,000). Bob and David do not exceed their department averages.

**Example 2: Correlated Subquery with EXISTS**

```sql
-- Setup: Create customers and orders tables
CREATE TABLE customers (
    customer_id   INTEGER PRIMARY KEY,
    customer_name VARCHAR(50)
);

CREATE TABLE orders (
    order_id    INTEGER PRIMARY KEY,
    customer_id INTEGER,
    order_total NUMERIC(10, 2)
);

INSERT INTO customers VALUES (1, 'Alice'), (2, 'Bob'), (3, 'Carol'), (4, 'David');
INSERT INTO orders VALUES (101, 1, 250.00), (102, 1, 300.00), (103, 2, 150.00), (104, 3, 400.00);

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

**Why this output occurs:** For each customer row, the `EXISTS` subquery checks whether any order exists with that customer's ID. If at least one order exists, `EXISTS` returns `TRUE` and the customer is returned. David (no orders) is excluded. The subquery references `c.customer_id` from the outer query, making it correlated.

### Real-World Cases

**Case 1: Department Average Salary Comparison**

An HR system uses a correlated subquery to identify employees earning above their department's average salary, as shown in the example above. This is a classic pattern for identifying outliers within groups.

**Case 2: Most Recent Order per Customer**

An e-commerce system uses a correlated subquery to find the most recent order for each customer: `SELECT SalesOrderID, CustomerID, OrderDate FROM SalesOrderHeader AS o1 WHERE SalesOrderID = (SELECT MAX(SalesOrderID) FROM SalesOrderHeader AS o2 WHERE o2.CustomerID = o1.CustomerID)`.

**Case 3: Customer Existence Check**

A CRM system uses `EXISTS` with a correlated subquery to find customers who have placed at least one order, enabling targeted marketing campaigns.

### References

- Oracle Database SQL Language Reference — Using Subqueries (Correlated Subqueries) - https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Using-Subqueries.html
- Oracle Database SQL Language Reference — SELECT (Correlated Subquery Syntax) - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/SELECT.html
- MySQL 8.0 Reference Manual — Correlated Subqueries - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/correlated-subqueries.html
- Microsoft Learn — Use Self-Contained or Correlated Subqueries - https://learn.microsoft.com/en-us/training/modules/write-subqueries/4-self-contained-correlated-subqueries
- Microsoft Learn — Compare Rows with Correlated Subqueries - https://learn.microsoft.com/en-us/training/modules/write-advanced-sql-code/8-correlated-queries

---

## 2. Scalar Correlated Subqueries

### Definitions

**Core Definition:** A scalar correlated subquery is a correlated subquery that returns exactly one row and one column — a single value — which the outer query uses in a comparison or expression.

**Technical Definition:** A scalar correlated subquery is a correlated subquery that returns a single column value from a single row. It is a valid form of expression and can be used in most places where a value expression is allowed, including the `SELECT` list, `WHERE` clause, and `HAVING` clause. If the subquery returns zero rows, the scalar result is `NULL`. If it returns more than one row, the database raises an error.

**Beginner-Friendly Explanation:** A scalar correlated subquery gives you exactly one value per outer row. For example, for each employee, it might return the average salary of that employee's department. You can use that value in a comparison, like `WHERE salary > (correlated scalar subquery)`.

### Purposes

- To provide a single value per outer row for use in a comparison or calculation.
- To compute group-relative aggregates (e.g., department average) for each row.
- To retrieve a specific related value based on the outer row's attributes.
- To use in `SELECT`, `WHERE`, and `HAVING` clauses as a dynamic value.
- To support row-by-row conditional logic.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT select_list,
       (SELECT single_value
        FROM table2 t_alias2
        WHERE t_alias1.column = t_alias2.column) AS derived_column
FROM table1 t_alias1
WHERE columnN operator (
    SELECT single_value
    FROM table2 t_alias2
    WHERE t_alias1.column = t_alias2.column
);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `(SELECT single_value ...)` | Scalar correlated subquery returning one row and one column |
| `t_alias1.column` | The outer reference linking the subquery to the outer query |
| `operator` | Comparison operator (`=`, `>`, `<`, etc.) |
| `derived_column` | Alias for the scalar result in the `SELECT` list |

**Syntax Rules:**

- The subquery must return exactly one column and at most one row.
- If the subquery returns no rows, the scalar result is `NULL`.
- If the subquery returns more than one row, an error occurs (e.g., "subquery returned more than 1 row").
- Scalar correlated subqueries can be used in the `SELECT` list, `WHERE` clause, and `HAVING` clause.
- Parentheses are required even when the subquery is an operand of a function.

**Constraints and Limitations:**

- Scalar correlated subqueries cannot return multiple columns.
- Using a scalar correlated subquery in the `SELECT` list causes it to be evaluated once per outer row.
- Performance can degrade significantly on large tables because the subquery runs once per outer row.
- MySQL's optimizer can transform correlated scalar subqueries into derived tables when the `subquery_to_derived` flag is enabled (MySQL 8.0.24+).

### Annotated Complete Code Examples

**Example 1: Scalar Correlated Subquery in WHERE Clause**

```sql
-- Using the employees table from Example 1
SELECT department_id, last_name, salary
FROM employees x
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE x.department_id = department_id
)
ORDER BY department_id, last_name;

-- Expected Output:
--  department_id | last_name |  salary
-- ---------------+-----------+----------
--             10 | Alice     | 95000.00
--             10 | Carol     |105000.00
--             20 | Eve       | 88000.00
```

**Why this output occurs:** For each employee row, the scalar correlated subquery computes the average salary for that employee's department. The outer query compares the employee's salary to this dynamically calculated average. Alice and Carol (Engineering, avg 100,000) and Eve (Marketing, avg 80,000) earn above their department averages.

**Example 2: Scalar Correlated Subquery in SELECT List**

```sql
-- Query: Show each employee's salary and their department's average
SELECT last_name,
       salary,
       (SELECT AVG(salary)
        FROM employees
        WHERE department_id = x.department_id) AS dept_avg
FROM employees x
ORDER BY department_id, last_name;

-- Expected Output:
--  last_name |  salary  |     dept_avg
-- -----------+----------+--------------------
--  Alice     | 95000.00 | 100000.000000000000
--  Carol     |105000.00 | 100000.000000000000
--  Bob       | 72000.00 |  80000.000000000000
--  Eve       | 88000.00 |  80000.000000000000
--  David     | 68000.00 |  68000.000000000000
```

**Why this output occurs:** The scalar correlated subquery in the `SELECT` list computes the average salary for each employee's department. The result is a new column (`dept_avg`) showing each employee's department average alongside their individual salary.

### Real-World Cases

**Case 1: Salary Benchmarking**

An HR system uses a scalar correlated subquery to show each employee's salary alongside their department's average, enabling managers to quickly identify who is above or below the benchmark.

**Case 2: Product Price Comparison**

An e-commerce analyst uses a scalar correlated subquery to compare each product's price to the average price in its category: `SELECT product_name, price, (SELECT AVG(price) FROM products WHERE category = p.category) AS avg_category_price FROM products p`.

### References

- Oracle Database SQL Language Reference — Scalar Subquery Expressions - https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Using-Subqueries.html
- MySQL 8.0 Reference Manual — Correlated Subqueries (Scalar Transform) - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/correlated-subqueries.html
- Microsoft Learn — Compare Rows with Correlated Subqueries - https://learn.microsoft.com/en-us/training/modules/write-advanced-sql-code/8-correlated-queries

---

## 3. Multi-Valued Correlated Subqueries

### Definitions

**Core Definition:** A multi-valued correlated subquery is a correlated subquery that returns multiple rows (and typically one column), used with set operators such as `IN`, `ANY`, or `ALL`.

**Technical Definition:** A multi-valued correlated subquery returns a set of rows, much like a single-column table. It is used with operators that can handle multiple values: `IN` (equality to any value in the list), `ANY` (comparison with any value returned), and `ALL` (comparison with all values returned). The subquery must return only one column when used with `IN`, `ANY`, or `ALL`. The subquery references at least one column from the outer query, making it correlated.

**Beginner-Friendly Explanation:** A multi-valued correlated subquery gives you a list of values per outer row. You use it when you want to check if something is in that list (`IN`), or compare against all of them (`ALL`), or compare against any of them (`ANY`). For example, "Find products whose price is higher than all products in a specific category."

### Purposes

- To filter rows based on membership in a set of values that depends on the outer row.
- To compare a value against all values returned by a correlated subquery (`ALL`).
- To compare a value against any value returned by a correlated subquery (`ANY`).
- To use a correlated subquery result as a set for further filtering.
- To check for existence or absence of related rows using `IN` or `NOT IN`.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT select_list
FROM table1 t_alias1
WHERE columnN IN (
    SELECT columnX
    FROM table2 t_alias2
    WHERE t_alias1.column = t_alias2.column
);

WHERE columnN operator ANY (
    SELECT columnX
    FROM table2 t_alias2
    WHERE t_alias1.column = t_alias2.column
);

WHERE columnN operator ALL (
    SELECT columnX
    FROM table2 t_alias2
    WHERE t_alias1.column = t_alias2.column
);
```

**Component Breakdown:**

| Operator | Purpose |
|----------|---------|
| `IN` | True if the value matches any value in the subquery result |
| `ANY` | True if the comparison is true for at least one value in the result |
| `ALL` | True if the comparison is true for all values in the result |

**Syntax Rules:**

- The subquery must return exactly one column when used with `IN`, `ANY`, or `ALL`.
- `ANY` and `ALL` are used with comparison operators (`=`, `>`, `<`, etc.).
- `IN` is equivalent to `= ANY`.
- `NOT IN` is equivalent to `<> ALL`.
- The subquery references at least one column from the outer query.

**Constraints and Limitations:**

- `NOT IN` with a subquery that returns `NULL` can produce unexpected results (returns no rows).
- `ANY` and `ALL` cannot be used with `EXISTS`.
- Multi-valued correlated subqueries can be less efficient than `EXISTS` in some RDBMSs.

### Annotated Complete Code Examples

**Example 1: Multi-Valued Correlated Subquery with IN**

```sql
-- Setup: Create products and categories tables
CREATE TABLE categories (
    category_id   INTEGER PRIMARY KEY,
    category_name VARCHAR(50)
);

CREATE TABLE products (
    product_id   INTEGER PRIMARY KEY,
    product_name VARCHAR(50),
    category_id  INTEGER,
    price        NUMERIC(10, 2)
);

INSERT INTO categories VALUES (10, 'Electronics'), (20, 'Clothing');
INSERT INTO products VALUES
    (1, 'Laptop', 10, 1200.00),
    (2, 'Phone',  10, 800.00),
    (3, 'Shirt',  20, 50.00),
    (4, 'Jeans',  20, 75.00);

-- Query: Find products in categories that have at least one product over $1000
SELECT product_name, price
FROM products p
WHERE category_id IN (
    SELECT category_id
    FROM products
    WHERE price > 1000
);

-- Expected Output:
--  product_name |  price
-- --------------+---------
--  Laptop       | 1200.00
--  Phone        |  800.00
```

**Why this output occurs:** The correlated subquery returns the category IDs (10) that have at least one product priced over $1,000. The outer query then returns all products in those categories. Only category 10 (Electronics) qualifies, so Laptop and Phone are returned.

**Example 2: Multi-Valued Correlated Subquery with ALL**

```sql
-- Query: Find products priced higher than all products in category 20
SELECT product_name, price
FROM products p
WHERE price > ALL (
    SELECT price
    FROM products
    WHERE category_id = 20
);

-- Expected Output:
--  product_name |  price
-- --------------+---------
--  Laptop       | 1200.00
--  Phone        |  800.00
```

**Why this output occurs:** The correlated subquery returns all prices in category 20 (50, 75). The outer query uses `> ALL` to find products priced higher than both 50 and 75. Laptop (1200) and Phone (800) both exceed 75, so they are returned. This is a non-correlated example for clarity, but the pattern applies to correlated subqueries as well.

### Real-World Cases

**Case 1: Products in High-Value Categories**

A retail analyst uses a multi-valued correlated subquery to find all products in categories that contain at least one premium product (price > $1000), identifying categories worth expanding.

**Case 2: Employees in Departments with High Average Salaries**

An HR system uses `IN` with a correlated subquery to find employees in departments where the average salary exceeds a threshold.

### References

- MySQL 8.0 Reference Manual — Subqueries with ANY, IN, or SOME - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/any-in-some-subqueries.html
- MySQL 8.0 Reference Manual — Subqueries with ALL - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/all-subqueries.html
- Oracle Database SQL Language Reference — Using Subqueries (Multi-Row Subqueries) - https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Using-Subqueries.html

---

## 4. Correlated Subqueries with EXISTS / NOT EXISTS

### Definitions

**Core Definition:** `EXISTS` is a Boolean predicate that returns `TRUE` if a correlated subquery returns at least one row, while `NOT EXISTS` returns `TRUE` if the subquery returns no rows.

**Technical Definition:** The `EXISTS` predicate determines whether any rows meeting a specified condition exist. Rather than returning the rows, it returns `TRUE` or `FALSE`. When a subquery is related to the outer query using the `EXISTS` predicate, the database handles the results of the subquery in a special way — it does not retrieve a scalar value or a multi-valued list but simply checks for the existence of any row. The `EXISTS` subquery is almost always correlated because it references a column from the outer query to check for matching rows.

**Beginner-Friendly Explanation:** `EXISTS` asks "does at least one row exist that matches this condition?" It's like checking if a box contains anything without actually opening it and listing the contents. `NOT EXISTS` asks the opposite: "does no matching row exist?" These are the most common and often the most efficient ways to write correlated subqueries.

### Purposes

- To check for the existence of related rows without retrieving them.
- To filter the outer table based on whether matching rows exist in another table.
- To implement anti-join patterns using `NOT EXISTS` (finding rows with no matches).
- To validate data without incurring the overhead of retrieving and processing results.
- To enable the optimizer to stop processing as soon as the first match is found.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT select_list
FROM table1 t_alias1
WHERE EXISTS (
    SELECT 1
    FROM table2 t_alias2
    WHERE t_alias1.column = t_alias2.column
);

WHERE NOT EXISTS (
    SELECT 1
    FROM table2 t_alias2
    WHERE t_alias1.column = t_alias2.column
);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `EXISTS` | Returns `TRUE` if the subquery returns at least one row |
| `NOT EXISTS` | Returns `TRUE` if the subquery returns no rows |
| `SELECT 1` | The select list is irrelevant; `1` is a common convention |
| `t_alias1.column = t_alias2.column` | The correlation condition |

**Syntax Rules:**

- The `SELECT` list in the subquery is irrelevant; `SELECT 1`, `SELECT *`, or `SELECT column` all work identically.
- `EXISTS` returns `TRUE` if the subquery returns one or more rows, `FALSE` otherwise.
- `EXISTS` is almost always correlated; the correlation condition is in the `WHERE` clause.
- `EXISTS` can be negated with `NOT EXISTS`.
- `EXISTS` typically has better performance than `IN` with subqueries, especially when checking existence in large tables, because the optimizer can stop after finding the first match.

**Constraints and Limitations:**

- `EXISTS` cannot be used with comparison operators like `=`, `>`, `<`.
- `EXISTS` does not return the values from the subquery; it only checks for existence.
- `NOT EXISTS` with a subquery that returns `NULL` values behaves correctly (unlike `NOT IN`).
- The subquery must reference at least one column from the outer query to be correlated.

### Annotated Complete Code Examples

**Example 1: EXISTS to Find Customers with Orders**

```sql
-- Using the customers and orders tables from Example 2
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

**Why this output occurs:** For each customer row, the `EXISTS` subquery checks whether any order exists with that customer's ID. Alice, Bob, and Carol have orders; David does not. The subquery returns `TRUE` for the first three customers and `FALSE` for David.

**Example 2: NOT EXISTS to Find Customers Without Orders**

```sql
-- Query: Find customers who have never placed an order
SELECT customer_name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);

-- Expected Output:
--  customer_name
-- ---------------
--  David
```

**Why this output occurs:** For each customer row, the `NOT EXISTS` subquery checks whether no order exists with that customer's ID. Only David has no orders, so only David is returned. This is the classic anti-join pattern.

### Real-World Cases

**Case 1: Churn Analysis**

A subscription business uses `NOT EXISTS` to find users who have not logged in for 30 days: `SELECT user_id FROM users WHERE NOT EXISTS (SELECT 1 FROM logins WHERE logins.user_id = users.id AND logins.login_date > NOW() - INTERVAL '30 days')`.

**Case 2: Missing Data Audit**

A data engineer uses `NOT EXISTS` to find products in an inventory table that have no corresponding record in a suppliers table, flagging them for investigation.

**Case 3: Active User Identification**

A social media platform uses `EXISTS` to find users who have posted at least one comment: `SELECT user_id FROM users WHERE EXISTS (SELECT 1 FROM comments WHERE comments.user_id = users.user_id)`.

### References

- MySQL 8.0 Reference Manual — Subqueries with EXISTS or NOT EXISTS - https://dev.mysqld.com.cn/
- Oracle Database SQL Language Reference — EXISTS and NOT EXISTS - https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Using-Subqueries.html
- Microsoft Learn — Use Self-Contained or Correlated Subqueries (EXISTS) - https://learn.microsoft.com/en-us/training/modules/write-subqueries/4-self-contained-correlated-subqueries
- PostgreSQL Documentation — Subquery Expressions (EXISTS) - https://www.postgresql.org/docs/current/functions-subquery.html

---

## 5. Correlated Subqueries in UPDATE and DELETE Statements

### Definitions

**Core Definition:** Correlated subqueries can be used in `UPDATE` and `DELETE` statements to modify or remove rows based on values from related tables.

**Technical Definition:** A correlated subquery in an `UPDATE` statement provides the value for the `SET` clause based on the current row being updated. In a `DELETE` statement, the correlated subquery defines which rows to delete based on conditions involving other tables. The subquery references a column from the table being updated or deleted from, creating the correlation.

**Beginner-Friendly Explanation:** You can use a correlated subquery to update or delete rows based on information from other tables. For example, "Update all employees' salaries to the average salary of their department" or "Delete all customers who have no orders."

### Purposes

- To update rows in one table based on values from related tables.
- To delete rows from one table based on conditions involving another table.
- To perform data synchronisation and reconciliation tasks.
- To implement complex business rules that depend on related data.

### Syntax Rules and Structure

**UPDATE with Correlated Subquery:**

```sql
UPDATE table1 t_alias1
SET column = (
    SELECT expr
    FROM table2 t_alias2
    WHERE t_alias1.column = t_alias2.column
);
```

**DELETE with Correlated Subquery:**

```sql
DELETE FROM table1 t_alias1
WHERE column operator (
    SELECT expr
    FROM table2 t_alias2
    WHERE t_alias1.column = t_alias2.column
);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `UPDATE table1 t_alias1` | The table being updated, with an alias |
| `SET column = (subquery)` | The value to assign, computed per row |
| `DELETE FROM table1 t_alias1` | The table being deleted from, with an alias |
| `WHERE column operator (subquery)` | The condition determining which rows to delete |

**Syntax Rules:**

- The correlated subquery must reference a column from the table being updated or deleted from.
- The subquery must return exactly one row and one column for `SET` assignments.
- For `DELETE`, the subquery can return multiple rows when used with `IN`.
- Oracle supports correlated subqueries in `UPDATE` and `DELETE` statements.

**Constraints and Limitations:**

- The subquery must return a single value for `SET` assignments; multiple rows cause an error.
- Updating a table based on a subquery that references the same table can cause unexpected results.
- Performance can be slow for large tables because the subquery runs per row.

### Annotated Complete Code Examples

**Example 1: Correlated UPDATE**

```sql
-- Setup: Departments table with average salaries
CREATE TABLE departments (
    dept_id       INTEGER PRIMARY KEY,
    dept_name     VARCHAR(50),
    avg_salary    NUMERIC(10, 2)
);

INSERT INTO departments VALUES
    (10, 'Engineering', 100000.00),
    (20, 'Marketing',   80000.00),
    (30, 'Sales',       68000.00);

-- Query: Update each department's average salary to the actual average
UPDATE departments d
SET avg_salary = (
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = d.dept_id
);

-- Expected Output:
-- UPDATE 3
```

**Why this output occurs:** For each department row, the correlated subquery computes the actual average salary from the `employees` table for that department. The `SET` clause assigns this value to `avg_salary`. All three departments are updated.

**Example 2: Correlated DELETE**

```sql
-- Query: Delete customers who have never placed an order
DELETE FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);

-- Expected Output:
-- DELETE 1
```

**Why this output occurs:** For each customer row, the `NOT EXISTS` subquery checks whether no order exists for that customer. David has no orders, so his row is deleted. The other customers are retained.

### Real-World Cases

**Case 1: Salary Adjustment**

An HR system uses a correlated `UPDATE` to adjust salaries based on department performance metrics: `UPDATE employees e SET salary = salary * 1.05 WHERE department_id IN (SELECT dept_id FROM departments WHERE performance_rating = 'Excellent')`.

**Case 2: Orphan Cleanup**

A database maintenance script uses a correlated `DELETE` to remove orphaned records: `DELETE FROM orders WHERE NOT EXISTS (SELECT 1 FROM customers WHERE customers.customer_id = orders.customer_id)`.

### References

- Oracle Database SQL Language Reference — Correlated UPDATE and DELETE - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/SELECT.html
- Oracle Database SQL Language Reference — Using Subqueries (UPDATE, DELETE) - https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Using-Subqueries.html

---

## 6. Performance Optimisation and Decorrelation

### Definitions

**Core Definition:** Decorrelation is a query optimisation technique that removes correlation from a subquery, transforming it into an equivalent set-oriented operation (typically a join) that can be executed more efficiently.

**Technical Definition:** Correlated subqueries refer to columns defined by their outer queries and can be evaluated by tuple-at-a-time execution strategies, which are generally slow. Query decorrelation is a technique that removes correlation from subqueries and typically results in faster set-oriented execution. The query optimizer often transforms correlated subqueries into equivalent joins internally. MySQL 8.0.24+ can transform a correlated scalar subquery to a derived table when the `subquery_to_derived` flag is enabled. Aurora PostgreSQL can accelerate correlated subqueries by transforming them into equivalent outer joins.

**Beginner-Friendly Explanation:** Decorrelation is the database's way of rewriting a slow correlated subquery into a faster join. Instead of running the subquery once for every row (like a nested loop), the database rewrites it as a single set-based operation. This can make a huge difference in performance.

### Purposes

- To improve query performance by converting row-by-row processing into set-based operations.
- To reduce the number of times the subquery is executed.
- To enable the use of hash joins and merge joins, which are more efficient than nested loops.
- To optimise queries that would otherwise cause N × M loops on large tables.
- To leverage optimizer transformations without rewriting the query manually.

### Syntax Rules and Structure

**Decorrelation Techniques:**

| Technique | Description |
|-----------|-------------|
| Subquery transformation | Rewriting correlated subqueries as equivalent outer joins |
| Subquery caching | Storing the results of correlated subqueries for reuse (Aurora PostgreSQL) |
| Derived table transformation | Converting a correlated scalar subquery into a derived table (MySQL 8.0.24+) |
| Query flattening | Removing correlated subqueries by merging them into the outer query (SQL Server) |

**Syntax Rules (Enabling Transformation):**

```sql
-- Aurora PostgreSQL: Enable correlated scalar transformation
SET apg_enable_correlated_scalar_transform TO ON;

-- MySQL: Enable subquery-to-derived transformation
SET optimizer_switch = 'subquery_to_derived=on';
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `apg_enable_correlated_scalar_transform` | Aurora PostgreSQL parameter to enable transformation |
| `subquery_to_derived` | MySQL optimizer switch to enable derived table transformation |
| `EXPLAIN` | Command to verify whether transformation occurred |

**Constraints and Limitations:**

- Not all correlated subqueries can be decorrelated; some must be executed as nested loops.
- MySQL transformation eligibility requires the correlated column to be in the subquery's `WHERE` clause only, not in the `SELECT` list.
- The correlated column must be resolved in the query block directly containing the subquery.
- Aurora PostgreSQL transformation applies only to subqueries that return a single aggregate value in the `SELECT` list or `WHERE` clause.
- Transformation parameters are version-specific (Aurora PostgreSQL 16.8+, MySQL 8.0.24+).

### Annotated Complete Code Examples

**Example 1: Verifying Transformation with EXPLAIN (Aurora PostgreSQL)**

```sql
-- Setup
CREATE TABLE ot (a INT, b INT);
CREATE TABLE it (a INT, b INT);

-- Enable transformation
SET apg_enable_correlated_scalar_transform TO ON;

-- Query with correlated subquery
EXPLAIN (COSTS FALSE)
SELECT ot.a, ot.b
FROM ot
WHERE ot.b < (SELECT AVG(it.b) FROM it WHERE it.a = ot.a);

-- Expected Output (transformed):
--  QUERY PLAN
-- ----------------------------------------------------------------------
--  Hash Join
--    Hash Cond: (ot.a = apg_scalar_subquery.scalar_output)
--    Join Filter: ((ot.b)::numeric < apg_scalar_subquery.avg)
--    ->  Seq Scan on ot
--    ->  Hash
--          ->  Subquery Scan on apg_scalar_subquery
--                ->  HashAggregate
--                      Group Key: it.a
--                      ->  Seq Scan on it
```

**Why this plan occurs:** With the transformation enabled, Aurora PostgreSQL rewrites the correlated subquery as a hash join between the outer table (`ot`) and a derived table that pre-aggregates the inner table (`it`) by the join key (`it.a`). This avoids executing the subquery once per row.

**Example 2: MySQL Derived Table Transformation**

```sql
-- Query with correlated scalar subquery
SELECT * FROM t1 WHERE (SELECT a FROM t2 WHERE t2.a = t1.a) > 0;

-- Transformed equivalent (conceptual)
SELECT t1.*
FROM t1
LEFT OUTER JOIN (
    SELECT a, COUNT(*) AS ct
    FROM t2
    GROUP BY a
) AS derived
ON t1.a = derived.a
WHERE derived.a > 0;
```

**Why this transformation occurs:** MySQL 8.0.24+ can transform the correlated scalar subquery into a derived table that groups the inner table by the join column, then joins it to the outer table. This materialises the subquery result once instead of executing it per row.

### Real-World Cases

**Case 1: Large-Scale Reporting**

A data warehouse runs a report with a correlated subquery that calculates a per-customer aggregate. Without decorrelation, the query takes hours. With the optimizer's decorrelation transformation, it completes in minutes.

**Case 2: OLTP Query Tuning**

A DBA notices a slow order-processing query with a correlated subquery. By enabling `subquery_to_derived` in MySQL, the query is rewritten as a join and the response time drops from seconds to milliseconds.

### References

- MySQL 8.0 Reference Manual — Correlated Subqueries (Optimizer Transformations) - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/correlated-subqueries.html
- Amazon Aurora User Guide — Optimizing Correlated Subqueries in Aurora PostgreSQL - https://docs.amazonaws.cn/en_us/AmazonRDS/latest/AuroraUserGuide/apg-correlated-subquery.html
- Microsoft Learn — Decorrelating Subqueries - https://learn.microsoft.com/en-us/archive/blogs/conorvsql/decorrelating-subqueries
- ACM Digital Library — Query Decorrelation in the Fabric Data Warehouse - https://dl.acm.org/
- ACM Digital Library — Orthogonal Optimization of Subqueries and Aggregation - https://dlnext.acm.org/

---

## 7. Correlated vs. Non-Correlated Subqueries

### Definitions

**Core Definition:** A non-correlated subquery is a subquery that can be evaluated independently of the outer query, while a correlated subquery depends on the outer query for its values.

**Technical Definition:** A non-correlated subquery does not contain references to objects in a parent statement; it can be evaluated as if it were an independent query. It executes once and returns a fixed result to the outer query. A correlated subquery contains an outer reference and cannot be evaluated independently; it conceptually executes once for each row processed by the outer query.

**Beginner-Friendly Explanation:** A non-correlated subquery runs once and gives the same answer to the outer query every time. A correlated subquery runs again for each row of the outer query, giving a different answer each time based on the current row.

### Purposes

- To understand the fundamental difference in evaluation behaviour between the two subquery types.
- To choose the appropriate subquery type for a given problem.
- To predict performance characteristics (non-correlated is generally faster).
- To recognise when a correlated subquery can be rewritten as a non-correlated one or a join.

### Syntax Rules and Structure

**Non-Correlated Subquery:**

```sql
SELECT select_list
FROM table1
WHERE columnN operator (
    SELECT columnX
    FROM table2
    WHERE condition
);
-- No reference to table1 inside the subquery
```

**Correlated Subquery:**

```sql
SELECT select_list
FROM table1 t_alias1
WHERE columnN operator (
    SELECT columnX
    FROM table2 t_alias2
    WHERE t_alias1.column = t_alias2.column
);
-- References t_alias1.column from the outer query
```

**Component Breakdown:**

| Aspect | Non-Correlated | Correlated |
|--------|---------------|------------|
| Outer reference | None | At least one column from the outer query |
| Execution | Once | Once per outer row |
| Independent execution | Yes | No |
| Performance | Generally faster | Can be slower on large tables |
| Common operators | `IN`, `=`, `>`, `<` | `EXISTS`, `NOT EXISTS`, `IN` |

**Syntax Rules:**

- The presence of an outer reference is the sole distinguishing factor.
- A non-correlated subquery can be converted to a correlated one by adding a reference to the outer query (though this is usually not desirable).
- A correlated subquery can sometimes be rewritten as a non-correlated one using joins or CTEs.

**Constraints and Limitations:**

- Non-correlated subqueries cannot reference outer query columns.
- Correlated subqueries cannot be tested independently.
- The optimizer may transform a correlated subquery into a non-correlated equivalent (decorrelation) for performance.

### Annotated Complete Code Examples

**Example 1: Non-Correlated vs. Correlated Comparison**

```sql
-- Non-correlated subquery (executes once)
SELECT ProductID, Name, ListPrice
FROM SalesLT.Product
WHERE ListPrice > (SELECT AVG(ListPrice) FROM SalesLT.Product);

-- Correlated subquery (executes per outer row)
SELECT p1.ProductID, p1.Name, p1.ListPrice
FROM SalesLT.Product AS p1
WHERE p1.ListPrice > (
    SELECT AVG(p2.ListPrice)
    FROM SalesLT.Product AS p2
    WHERE p2.ProductCategoryID = p1.ProductCategoryID
);
```

**Why this difference matters:** In the non-correlated example, the subquery calculates a single average price across all products. This value is computed once and every product is compared to it. In the correlated example, the subquery calculates the average price for each product's specific category. A product in "Bicycles" is compared with the average of bicycles, while a product in "Accessories" is compared with the average of accessories.

### Real-World Cases

**Case 1: Choosing the Right Subquery Type**

A business analyst needs to find products priced above the company-wide average. A non-correlated subquery is the right choice because the comparison is against a single static value. If the analyst needs to find products priced above their category average, a correlated subquery is required because the comparison value changes per row.

**Case 2: Performance Decision**

A developer notices a slow query with a correlated subquery on a large table. By understanding the difference, they rewrite the query as a join, achieving significant performance improvement.

### References

- Oracle Ask TOM — Difference Between Correlated and Non-Correlated Subqueries - https://asktom.oracle.com/
- SAP Help — Correlated and Uncorrelated Subqueries - https://help.sap.com/
- Microsoft Learn — Compare Rows with Correlated Subqueries - https://learn.microsoft.com/en-us/training/modules/write-advanced-sql-code/8-correlated-queries

---

## Summary Table of Correlated Subquery Features

| Feature | Purpose | Key Limitation |
|---------|---------|----------------|
| Correlated Subquery Fundamentals | Row-by-row dependency | Cannot run independently |
| Scalar Correlated Subqueries | Single value per outer row | Returns error if more than one row |
| Multi-Valued Correlated Subqueries | Set of values per outer row | `NOT IN` with `NULL` returns no rows |
| EXISTS / NOT EXISTS | Existence checking | Does not return subquery values |
| Correlated UPDATE / DELETE | Modify rows based on related data | Subquery must return single value for SET |
| Performance & Decorrelation | Optimise execution via joins | Not all subqueries can be decorrelated |
| Correlated vs. Non-Correlated | Understand evaluation differences | Correlated is generally slower |

---

## Dialect Support for Correlated Subqueries

| Feature | PostgreSQL | MySQL | SQL Server | Oracle | SQLite |
|---------|-----------|-------|------------|--------|--------|
| Correlated Subqueries | ✅ | ✅ | ✅ | ✅ | ✅ |
| `EXISTS` / `NOT EXISTS` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `ANY` / `ALL` | ✅ | ✅ | ✅ | ✅ | ✅ |
| Correlated `UPDATE` | ✅ | ✅ | ✅ | ✅ | ✅ |
| Correlated `DELETE` | ✅ | ✅ | ✅ | ✅ | ✅ |
| Decorrelation Transform | ✅ | ✅ (8.0.24+) | ✅ | ✅ | ❌ |
| Subquery Caching | ✅ (Aurora 16.8+) | ❌ | ❌ | ❌ | ❌ |

---

## References

- Oracle Database SQL Language Reference — Using Subqueries - https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Using-Subqueries.html
- Oracle Database SQL Language Reference — SELECT (Correlated Subquery Syntax) - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/SELECT.html
- MySQL 8.0 Reference Manual — Correlated Subqueries - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/correlated-subqueries.html
- MySQL 8.0 Reference Manual — Subqueries with EXISTS or NOT EXISTS - https://dev.mysqld.com.cn/
- MySQL 8.0 Reference Manual — Subqueries with ANY, IN, or SOME - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/any-in-some-subqueries.html
- MySQL 8.0 Reference Manual — Subqueries with ALL - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/all-subqueries.html
- Microsoft Learn — Use Self-Contained or Correlated Subqueries - https://learn.microsoft.com/en-us/training/modules/write-subqueries/4-self-contained-correlated-subqueries
- Microsoft Learn — Compare Rows with Correlated Subqueries - https://learn.microsoft.com/en-us/training/modules/write-advanced-sql-code/8-correlated-queries
- Microsoft Learn — Decorrelating Subqueries - https://learn.microsoft.com/en-us/archive/blogs/conorvsql/decorrelating-subqueries
- Amazon Aurora User Guide — Optimizing Correlated Subqueries in Aurora PostgreSQL - https://docs.amazonaws.cn/en_us/AmazonRDS/latest/AuroraUserGuide/apg-correlated-subquery.html
- PostgreSQL Documentation — Subquery Expressions - https://www.postgresql.org/docs/current/functions-subquery.html
- SAP Help — Correlated and Uncorrelated Subqueries - https://help.sap.com/
- Oracle Ask TOM — Difference Between Correlated and Non-Correlated Subqueries - https://asktom.oracle.com/
- ACM Digital Library — Query Decorrelation in the Fabric Data Warehouse - https://dl.acm.org/
- ACM Digital Library — Orthogonal Optimization of Subqueries and Aggregation - https://dlnext.acm.org/
- ISO/IEC 9075-2 — SQL Standard (Subqueries) - https://jtc1sc32.org/doc/N0251-0300/32N0254.pdf