# SQL JOIN Syntax & Operators: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL JOIN syntax and operators define how two or more tables are combined into a single result set, using a join condition to specify the matching criteria between rows.

**Technical Definition:** In the SQL standard (ISO/IEC 9075-2), a qualified join consists of a table reference, a join type, the `JOIN` keyword, a second table reference, and a join specification. The join specification is one of: `ON <search condition>`, `USING (<column list>)`, or `NATURAL`. The `ON` clause provides the most general form, accepting any Boolean expression involving columns from the joined tables. The `USING` clause is a shorthand for equality comparisons on identically named columns. Joins are further classified by their comparison operator: equi-joins use `=`, while non-equi-joins use operators such as `<`, `>`, `BETWEEN`, or `<>`. Multiple tables can be chained in a single `FROM` clause, with each `JOIN` clause evaluated left to right according to the foreign-key relationships between tables.

**Beginner-Friendly Explanation:** When you join tables, you need to tell the database which rows from one table match which rows in another. The `ON` clause is the detailed instruction: "match these two columns." The `USING` clause is a shortcut when both tables have a column with the same name. Some joins look for exact matches (`=`), while others look for ranges or inequalities. And when you need data from three or more tables, you simply chain the joins together.

### Key Characteristics

- **Join specification required:** Every join (except `CROSS JOIN`) requires exactly one of `ON`, `USING`, or `NATURAL`.
- **ON is the general form:** Accepts any Boolean expression, including comparisons, `AND`/`OR`, and `BETWEEN`.
- **USING is shorthand:** Requires identically named columns and produces a single coalesced column in the output.
- **Equi vs non-equi:** Equi-joins use `=`, non-equi-joins use `<`, `>`, `BETWEEN`, or other inequality operators.
- **Left-to-right evaluation:** Multi-table joins are logically evaluated left to right, with each join building on the result of the previous one.
- **Foreign-key alignment:** Join predicates should mirror the real foreign-key graph of the schema.
- **Dialect variations:** Support for `USING` with `FULL OUTER JOIN`, row constructors, and vendor-specific extensions varies.

### Prerequisites

- **Basic SQL knowledge:** Familiarity with `SELECT`, `FROM`, and `WHERE`.
- **Join types:** Understanding of `INNER`, `LEFT`, `RIGHT`, `FULL`, and `CROSS` joins.
- **Relational concepts:** Knowledge of primary keys, foreign keys, and table relationships.
- **Table aliasing:** Ability to use table aliases to distinguish tables in multi-table queries.

### Related Programming Areas

- **Data Analysis and Business Intelligence:** Multi-table joins are the backbone of analytical queries.
- **Application Development:** Backend services use joins to retrieve related data from normalised schemas.
- **Data Engineering (ETL):** Joins merge data from multiple source systems.
- **Database Administration:** DBAs use joins to inspect and troubleshoot data relationships.

### Core Concepts / Features

1. ON Clause
2. USING Clause
3. Equi-Join vs. Non-Equi-Join
4. Multiple Table Joins

---

## 1. ON Clause

### Definitions

**Core Definition:** The `ON` clause is the standard predicate used to define the specific matching column conditions between tables in a join.

**Technical Definition:** In SQL, the `ON` clause specifies a `<search_condition>` that evaluates to a Boolean value, determining which rows from the joined tables are considered matches. The condition can include any predicate, though column names and comparison operators are most common. Column names do not need to be identical, and data types do not need to match exactly, though they must be compatible or implicitly convertible. Predicates in the `ON` clause are applied to the table before the join is executed, while the `WHERE` clause is applied to the results of the join.

**Beginner-Friendly Explanation:** The `ON` clause is where you tell the database exactly how two tables are related. You write a condition like `customers.customer_id = orders.customer_id` to say "match rows where the customer ID is the same in both tables." It's the most flexible way to join tables because you can use any kind of comparison.

### Purposes

- To specify the exact matching condition between two tables in a join.
- To compare columns that may have different names but represent the same logical entity.
- To include additional filtering logic (e.g., `AND`, `OR`) within the join condition.
- To control the join behaviour for outer joins by placing conditions in the `ON` clause versus the `WHERE` clause.
- To provide the most general and portable join specification across all RDBMSs.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT select_list
FROM table1
[INNER | LEFT | RIGHT | FULL] JOIN table2
    ON search_condition;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `table1`, `table2` | The tables being joined |
| `JOIN` type | `INNER`, `LEFT`, `RIGHT`, or `FULL` |
| `ON` | Keyword introducing the join condition |
| `search_condition` | Boolean expression defining matching rows |

**Syntax Rules:**

- The `ON` clause is required for all qualified joins except `CROSS JOIN`.
- The `ON` condition can reference columns from any table in the join.
- Column names do not need to match between tables; data types must be compatible.
- The `ON` clause can contain multiple conditions combined with `AND`, `OR`, and `NOT`.
- `ON` is the most general join specification; `USING` and `NATURAL` are shorthands for common cases.

**Constraints and Limitations:**

- Predicates in the `ON` clause are applied before the join, which can affect the result of outer joins differently than placing the same predicate in `WHERE`.
- If the `ON` condition references columns that are not in either table, an error occurs.
- All column references must be unambiguous; qualified names or aliases are required when column names are duplicated.

### Annotated Complete Code Examples

**Example 1: Basic ON Clause with INNER JOIN**

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

INSERT INTO customers VALUES
    (1, 'Alice'), (2, 'Bob'), (3, 'Carol');
INSERT INTO orders VALUES
    (101, 1, 250.00), (102, 1, 300.00),
    (103, 2, 150.00), (104, 3, 400.00);

-- Query: INNER JOIN using ON
SELECT c.customer_name, o.order_id, o.order_total
FROM customers AS c
INNER JOIN orders AS o
    ON c.customer_id = o.customer_id
ORDER BY c.customer_name, o.order_id;

-- Expected Output:
--  customer_name | order_id | order_total
-- ---------------+----------+-------------
--  Alice         |      101 |      250.00
--  Alice         |      102 |      300.00
--  Bob           |      103 |      150.00
--  Carol         |      104 |      400.00
```

**Why this output occurs:** The `ON c.customer_id = o.customer_id` condition matches rows where the customer ID is the same in both tables. Alice, Bob, and Carol all have matching orders. The result contains one row for each matching pair.

**Example 2: ON Clause with Additional Filtering**

```sql
-- Query: INNER JOIN with additional filter in ON clause
SELECT c.customer_name, o.order_id, o.order_total
FROM customers AS c
INNER JOIN orders AS o
    ON c.customer_id = o.customer_id
   AND o.order_total > 200
ORDER BY c.customer_name, o.order_id;

-- Expected Output:
--  customer_name | order_id | order_total
-- ---------------+----------+-------------
--  Alice         |      101 |      250.00
--  Alice         |      102 |      300.00
--  Carol         |      104 |      400.00
```

**Why this output occurs:** The `AND o.order_total > 200` condition is part of the join condition, so only orders exceeding $200 are matched. Bob's order (150) does not satisfy the condition and is excluded from the join result.

### Real-World Cases

**Case 1: Customer-Order Reporting**

An e-commerce system uses `ON c.customer_id = o.customer_id` to join customers with their orders, producing a report of customer names alongside order details.

**Case 2: Employee-Manager Hierarchy**

An HR system uses `ON emp.manager_id = mgr.employee_id` to join the employee table to itself, producing an organisational chart with each employee alongside their manager.

### References

- Microsoft Learn — FROM clause with JOIN (ON clause) - https://learn.microsoft.com/en-us/sql/t-sql/queries/from-transact-sql
- PostgreSQL Documentation — JOIN Types (ON clause) - https://www.postgresql.org/docs/current/queries-table-expressions.html
- Oracle Database SQL Language Reference — SELECT (JOIN ON clause) - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/SELECT.html
- PostgreSQL Documentation — FROM Clause (ON join_condition) - https://www.postgresql.org/files/documentation/pdf/7.3/reference-7.3.2-US.pdf
- MIT — INNER JOIN Operation (ON booleanExpression) - https://www.mit.edu/

---

## 2. USING Clause

### Definitions

**Core Definition:** The `USING` clause is a shorthand syntax for joining tables when the matching columns share the exact same name, automatically performing an equality join on those columns.

**Technical Definition:** A clause of the form `USING (a, b, ...)` is shorthand for `ON left_table.a = right_table.a AND left_table.b = right_table.b ...`. Additionally, `USING` implies that only one of each pair of equivalent columns is included in the join output, not both. The output column order places the `USING` columns first, followed by the remaining columns of the left table, then the remaining columns of the right table. `USING` can be used with `INNER`, `LEFT`, `RIGHT`, and `FULL` joins.

**Beginner-Friendly Explanation:** The `USING` clause is a shortcut for when both tables have a column with the same name. Instead of writing `ON a.id = b.id`, you write `USING (id)`. The database automatically joins on that column, and the result shows the column only once instead of twice.

### Purposes

- To simplify join syntax when the matching columns have the same name.
- To automatically coalesce duplicate join columns into a single output column.
- To reduce typing and potential errors in simple join scenarios.
- To improve query readability by making the join condition explicit through column names.
- To support multi-column joins concisely (e.g., `USING (col1, col2)`).

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT select_list
FROM table1
[INNER | LEFT | RIGHT | FULL] JOIN table2
    USING (column1 [, column2, ...]);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `table1`, `table2` | The tables being joined |
| `USING` | Keyword introducing the shorthand join specification |
| `(column1, ...)` | Comma-separated list of common column names |

**Syntax Rules:**

- The columns listed in `USING` must exist in both tables with the same name.
- `USING` is shorthand for `ON left_table.col = right_table.col` for each listed column.
- Only one of each pair of equivalent columns appears in the output; the `USING` columns come first.
- `USING` can be used with `INNER`, `LEFT`, `RIGHT`, and `FULL` joins.
- `USING` cannot be combined with `ON` in the same join.
- `NATURAL` is shorthand for a `USING` list that includes all commonly named columns.

**Constraints and Limitations:**

- The columns must have the same name in both tables.
- The data types of the `USING` columns must be compatible.
- The coalesced output column may cause ambiguity in the `SELECT` list if not aliased.
- MySQL, PostgreSQL, and Oracle support `USING`; SQL Server does not support `USING` (it only supports `ON`).

### Annotated Complete Code Examples

**Example 1: USING Clause with INNER JOIN**

```sql
-- Setup: Tables with a common column name
CREATE TABLE departments (
    department_id   INTEGER PRIMARY KEY,
    department_name VARCHAR(50)
);

CREATE TABLE employees (
    employee_id     INTEGER PRIMARY KEY,
    employee_name   VARCHAR(50),
    department_id   INTEGER
);

INSERT INTO departments VALUES
    (1, 'Engineering'), (2, 'Marketing'), (3, 'Sales');
INSERT INTO employees VALUES
    (10, 'Alice', 1), (11, 'Bob', 2), (12, 'Carol', 1);

-- Query: JOIN USING department_id
SELECT department_name, employee_name
FROM departments
INNER JOIN employees
    USING (department_id)
ORDER BY department_name, employee_name;

-- Expected Output:
--  department_name | employee_name
-- -----------------+---------------
--  Engineering     | Alice
--  Engineering     | Carol
--  Marketing       | Bob
```

**Why this output occurs:** The `USING (department_id)` clause is equivalent to `ON departments.department_id = employees.department_id`. The result includes one `department_id` column (coalesced) rather than two. Alice and Carol are in Engineering; Bob is in Marketing. Sales has no employees and is excluded by the INNER JOIN.

**Example 2: USING with FULL OUTER JOIN**

```sql
-- Query: FULL OUTER JOIN USING department_id
SELECT department_id, department_name, employee_name
FROM departments
FULL OUTER JOIN employees
    USING (department_id)
ORDER BY department_id;

-- Expected Output:
--  department_id | department_name | employee_name
-- ---------------+-----------------+---------------
--              1 | Engineering     | Alice
--              1 | Engineering     | Carol
--              2 | Marketing       | Bob
--              3 | Sales           | (null)
```

**Why this output occurs:** The `FULL OUTER JOIN USING (department_id)` preserves all departments and all employees. Sales (no employees) appears with `NULL` for `employee_name`. The `department_id` column is coalesced and appears only once in the output.

### Real-World Cases

**Case 1: Star Schema Reporting**

A data warehouse uses `USING (date_key)` to join fact tables to date dimension tables, simplifying queries and reducing column duplication.

**Case 2: Multi-Table Chains**

A reporting query joins `orders` to `order_items` using `USING (order_id)`, then to `products` using `USING (product_id)`, producing a clean result set without duplicate key columns.

### References

- PostgreSQL Documentation — JOIN Types (USING clause) - https://www.postgresql.org/docs/current/queries-table-expressions.html
- Oracle Database SQL Language Reference — SELECT (USING clause) - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/SELECT.html
- PostgreSQL Documentation — FROM Clause (USING column list) - https://www.postgresql.org/files/documentation/pdf/7.3/reference-7.3.2-US.pdf
- PostgreSQL Documentation — SELECT (USING shorthand) - https://manpages.debian.org/trixie/postgresql-client-17/SELECT.7.en.html
- Apache Derby — INNER JOIN USING clause - https://db.apache.org/derby/docs/

---

## 3. Equi-Join vs. Non-Equi-Join

### Definitions

**Core Definition:** An equi-join is a join whose condition uses the equality operator (`=`), while a non-equi-join uses a conditional operator such as `<`, `>`, `<=`, `>=`, `<>`, or `BETWEEN` to define the matching criteria.

**Technical Definition:** An equi-join is an inner join whose join condition contains an equality operator. The join condition `e.department_id = d.department_id` is a classic equi-join. A non-equi-join (also called a nonequi-join) is an inner join whose join condition contains an operator that is not an equality operator. Non-equi-joins are relatively rare but useful for range-based matching, such as joining employees to job history based on hire dates falling within a range. A hash join requires at least a partial equi-join condition to be efficient.

**Beginner-Friendly Explanation:** An equi-join matches rows where two columns are exactly equal — it's the most common type of join. A non-equi-join matches rows where one value falls within a range of another, or where two values are not equal. For example, "find all employees whose salary is between the minimum and maximum salary of a job grade" is a non-equi-join.

### Purposes

**Equi-Joins:**
- To match rows based on exact equality of key columns.
- To implement the most common join pattern (foreign key to primary key).
- To enable efficient hash joins and merge joins.
- To combine related data where the relationship is defined by equality.

**Non-Equi-Joins:**
- To match rows based on range conditions (e.g., `BETWEEN`).
- To find rows where values are not equal (e.g., `<>`).
- To implement band joins, where key values fall within a range.
- To compare rows using inequality operators for analytical queries.

### Syntax Rules and Structure

**Equi-Join Syntax:**

```sql
SELECT select_list
FROM table1
JOIN table2
    ON table1.column = table2.column;
```

**Non-Equi-Join Syntax:**

```sql
SELECT select_list
FROM table1
JOIN table2
    ON table1.column BETWEEN table2.low AND table2.high;

-- Or using other operators
ON table1.column < table2.column
ON table1.column <> table2.column
```

**Component Breakdown:**

| Join Type | Operator | Example |
|-----------|----------|---------|
| Equi-Join | `=` | `ON e.dept_id = d.dept_id` |
| Non-Equi-Join | `<` | `ON e.hire_date < h.end_date` |
| Non-Equi-Join | `>` | `ON e.salary > g.min_salary` |
| Non-Equi-Join | `BETWEEN` | `ON e.salary BETWEEN g.low AND g.high` |
| Non-Equi-Join | `<>` | `ON e.dept_id <> d.dept_id` |

**Syntax Rules:**

- Both equi-joins and non-equi-joins use the same `ON` clause syntax; only the operator differs.
- Equi-joins can also use the `USING` clause when column names match.
- Non-equi-joins must use the `ON` clause; `USING` only supports equality.
- Non-equi-joins cannot be efficiently processed by hash joins; merge joins or nested loops are typically used.
- The `BETWEEN` operator is inclusive: `BETWEEN a AND b` is equivalent to `>= a AND <= b`.

**Constraints and Limitations:**

- Non-equi-joins are generally less efficient than equi-joins because they cannot use hash join algorithms.
- `BETWEEN` with `NULL` endpoints can produce unexpected results.
- Non-equi-joins can produce large result sets if the ranges are wide.
- Some optimisers may convert non-equi-joins to nested loop joins, which are slower for large tables.

### Annotated Complete Code Examples

**Example 1: Equi-Join**

```sql
-- Setup: Employees and departments
CREATE TABLE departments (
    dept_id   INTEGER PRIMARY KEY,
    dept_name VARCHAR(50)
);

CREATE TABLE employees (
    emp_id   INTEGER PRIMARY KEY,
    emp_name VARCHAR(50),
    dept_id  INTEGER
);

INSERT INTO departments VALUES (1, 'Engineering'), (2, 'Marketing');
INSERT INTO employees VALUES (10, 'Alice', 1), (11, 'Bob', 2), (12, 'Carol', 1);

-- Query: Equi-join on dept_id
SELECT e.emp_name, d.dept_name
FROM employees e
JOIN departments d
    ON e.dept_id = d.dept_id
ORDER BY e.emp_name;

-- Expected Output:
--  emp_name | dept_name
-- ----------+-------------
--  Alice    | Engineering
--  Bob      | Marketing
--  Carol    | Engineering
```

**Why this output occurs:** The `ON e.dept_id = d.dept_id` condition matches employees to departments where the department ID is exactly equal. Alice and Carol are in Engineering; Bob is in Marketing.

**Example 2: Non-Equi-Join with BETWEEN**

```sql
-- Setup: Salary grades
CREATE TABLE salary_grades (
    grade     VARCHAR(5),
    min_sal   NUMERIC(10, 2),
    max_sal   NUMERIC(10, 2)
);

INSERT INTO salary_grades VALUES
    ('A', 50000, 70000),
    ('B', 70001, 90000),
    ('C', 90001, 120000);

CREATE TABLE employees (
    emp_name VARCHAR(50),
    salary   NUMERIC(10, 2)
);

INSERT INTO employees VALUES
    ('Alice', 95000), ('Bob', 65000), ('Carol', 85000);

-- Query: Non-equi-join using BETWEEN
SELECT e.emp_name, e.salary, g.grade
FROM employees e
JOIN salary_grades g
    ON e.salary BETWEEN g.min_sal AND g.max_sal
ORDER BY e.emp_name;

-- Expected Output:
--  emp_name | salary  | grade
-- ----------+---------+-------
--  Alice    | 95000.00| C
--  Bob      | 65000.00| A
--  Carol    | 85000.00| B
```

**Why this output occurs:** The `ON e.salary BETWEEN g.min_sal AND g.max_sal` condition matches each employee to the salary grade whose range contains their salary. Alice (95,000) is in grade C (90,001–120,000); Bob (65,000) is in grade A (50,000–70,000); Carol (85,000) is in grade B (70,001–90,000).

### Real-World Cases

**Case 1: Employee Salary Grade Assignment**

An HR system uses a non-equi-join with `BETWEEN` to assign each employee to a salary grade based on their salary falling within a grade's range.

**Case 2: Promotional Pricing**

An e-commerce system uses a non-equi-join to match products to promotional discount tiers based on quantity purchased falling within tier ranges.

### References

- Oracle Database SQL Tuning Guide — Equijoins and Nonequijoins - https://docs.oracle.com/en/database/oracle/oracle-database/18/tgsql/joins.html
- Oracle Database SQL Tuning Guide — Nonequijoins - https://docs.oracle.com/en/database/oracle/oracle-database/18/tgsql/joins.html
- LearnSQL — Practical Examples of When to Use Non-Equi JOINs in SQL - https://learnsql.com/blog/non-equi-joins/
- Stack Overflow — Equi join vs non-equi join - https://stackoverflow.com/
- Sams Teach Yourself SQL — Equi-Join vs Non-Equi-Join - https://www.oreilly.com/

---

## 4. Multiple Table Joins

### Definitions

**Core Definition:** Multiple table joins combine three or more tables in a single query by chaining multiple `JOIN` clauses in the `FROM` clause, with each join building on the result of the previous one.

**Technical Definition:** A multi-table join is not a distinct SQL feature; it is simply multiple `JOIN` clauses chained in one `FROM` clause, each with its own `ON` predicate, evaluated left to right logically (though the physical execution order may be reordered by the optimizer). Each `JOIN` clause only references tables already introduced earlier in the `FROM` clause, reflecting the actual foreign-key relationships between tables. The result of each join becomes the input for the next join, effectively treating the intermediate result as a virtual table.

**Beginner-Friendly Explanation:** When you need data from three or more tables, you chain joins together. For example, to show employee names, their department names, and the city where the department is located, you join employees to departments, and then join the result to locations. Each join adds one more table to the picture.

### Purposes

- To retrieve data from three or more related tables in a single query.
- To traverse normalised schemas that spread data across many tables.
- To answer complex business questions requiring data from multiple entities.
- To build production analytics and reporting queries.
- To follow the foreign-key graph of a relational schema.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT select_list
FROM table1
JOIN table2
    ON table1.key = table2.key
JOIN table3
    ON table2.key = table3.key
[JOIN table4
    ON table3.key = table4.key];
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `table1`, `table2`, `table3` | The tables being joined in sequence |
| `JOIN` | Each join clause combines the current result with the next table |
| `ON` | Join condition linking the new table to a previously joined table |

**Syntax Rules:**

- Each `JOIN` clause combines two tables: the cumulative result of previous joins and the new table.
- The `ON` condition for each join should reference tables already introduced in the `FROM` clause.
- Join predicates should mirror the real foreign-key graph of the schema.
- The logical evaluation order is left to right, but the optimizer may reorder joins for performance, except for outer join chains.
- Parentheses can be used to explicitly control join order when mixing inner and outer joins.

**Constraints and Limitations:**

- The optimizer is free to reorder inner joins based on cost estimates, but this reordering freedom does not extend to `LEFT`/`RIGHT` join chains.
- Mixing inner and outer joins in a chain requires careful attention to join order because the semantics depend on the order.
- Very long join chains (5+ tables) can be difficult to read and debug; consider using CTEs or views.
- Each additional join multiplies the potential row combinations; verify row counts incrementally.

### Annotated Complete Code Examples

**Example 1: Three-Table Join Chain**

```sql
-- Setup: Three related tables
CREATE TABLE locations (
    location_id   INTEGER PRIMARY KEY,
    city          VARCHAR(50)
);

CREATE TABLE departments (
    dept_id       INTEGER PRIMARY KEY,
    dept_name     VARCHAR(50),
    location_id   INTEGER
);

CREATE TABLE employees (
    emp_id        INTEGER PRIMARY KEY,
    emp_name      VARCHAR(50),
    dept_id       INTEGER
);

INSERT INTO locations VALUES (1, 'Manila'), (2, 'Cebu');
INSERT INTO departments VALUES
    (10, 'Engineering', 1), (20, 'Marketing', 2);
INSERT INTO employees VALUES
    (100, 'Alice', 10), (101, 'Bob', 20), (102, 'Carol', 10);

-- Query: Join employees → departments → locations
SELECT e.emp_name, d.dept_name, l.city
FROM employees e
INNER JOIN departments d
    ON e.dept_id = d.dept_id
INNER JOIN locations l
    ON d.location_id = l.location_id
ORDER BY e.emp_name;

-- Expected Output:
--  emp_name | dept_name   | city
-- ----------+-------------+-------
--  Alice    | Engineering | Manila
--  Bob      | Marketing   | Cebu
--  Carol    | Engineering | Manila
```

**Why this output occurs:** The first join matches employees to departments on `dept_id`. The second join matches the result to locations on `location_id`. Alice and Carol are in Engineering (location Manila); Bob is in Marketing (location Cebu). Each `JOIN` clause references tables already in scope: `employees` → `departments` → `locations`.

**Example 2: Four-Table Join with Mixed Join Types**

```sql
-- Setup: Add a projects table and project assignments
CREATE TABLE projects (
    project_id   INTEGER PRIMARY KEY,
    project_name VARCHAR(50)
);

CREATE TABLE project_assignments (
    emp_id       INTEGER,
    project_id   INTEGER,
    hours        INTEGER
);

INSERT INTO projects VALUES (1000, 'Project Alpha'), (1001, 'Project Beta');
INSERT INTO project_assignments VALUES
    (100, 1000, 40), (100, 1001, 20), (101, 1000, 30);

-- Query: Four-table join with LEFT JOIN for optional assignments
SELECT e.emp_name, d.dept_name, l.city, p.project_name
FROM employees e
INNER JOIN departments d
    ON e.dept_id = d.dept_id
INNER JOIN locations l
    ON d.location_id = l.location_id
LEFT JOIN project_assignments pa
    ON e.emp_id = pa.emp_id
LEFT JOIN projects p
    ON pa.project_id = p.project_id
ORDER BY e.emp_name, p.project_name;

-- Expected Output:
--  emp_name | dept_name   | city   | project_name
-- ----------+-------------+--------+--------------
--  Alice    | Engineering | Manila | Project Alpha
--  Alice    | Engineering | Manila | Project Beta
--  Bob      | Marketing   | Cebu   | Project Alpha
--  Carol    | Engineering | Manila | (null)
```

**Why this output occurs:** The first two joins are `INNER JOIN` (all employees have departments and locations). The `LEFT JOIN` to `project_assignments` preserves all employees, even those without assignments. Carol has no project assignment, so `project_name` is `NULL` for her row. Alice has two assignments (Alpha and Beta), producing two rows for her.

### Real-World Cases

**Case 1: Sales Reporting Dashboard**

A sales dashboard queries `orders → order_items → products → categories` to produce a report showing product names, categories, and total sales amounts, traversing four tables via foreign-key relationships.

**Case 2: Employee Directory**

An HR system joins `employees → departments → locations → countries` to produce a comprehensive employee directory showing each employee's department, city, and country.

### References

- Microsoft Learn — Tables in Joins (Combining Three or More Tables) - https://learn.microsoft.com/en-us/previous-versions/he95de78(v=vs.140)
- Microsoft Learn — Join Tables (Combining Three or More Tables) - https://learn.microsoft.com/en-us/previous-versions/
- Oracle Database SQL Language Reference — Joins (three or more tables) - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/SELECT.html
- LearnSQL — How to Join 3 Tables (or More) in SQL - https://learnsql.com/blog/how-to-join-3-tables-or-more-in-sql/
- GitHub — Multi-Table Joins (SQL Engineering Handbook) - https://github.com/theammarngp-makes/SQL-Engineering-Handbook

---

## Summary Table of JOIN Syntax & Operators

| Feature | Purpose | Key Limitation |
|---------|---------|----------------|
| `ON` Clause | General join condition | Predicates applied before join (affects outer joins) |
| `USING` Clause | Shorthand for same-named columns | Requires identical column names; not supported in SQL Server |
| Equi-Join | Match on equality (`=`) | Cannot use hash joins for non-equality conditions |
| Non-Equi-Join | Match on ranges/inequalities | Less efficient; cannot use hash joins |
| Multiple Table Joins | Combine 3+ tables | Outer join chains are order-dependent; verify row counts incrementally |

---

## Dialect Support for JOIN Syntax

| Feature | PostgreSQL | MySQL | SQL Server | Oracle | SQLite |
|---------|-----------|-------|------------|--------|--------|
| `ON` Clause | ✅ | ✅ | ✅ | ✅ | ✅ |
| `USING` Clause | ✅ | ✅ | ❌ | ✅ | ✅ |
| `NATURAL JOIN` | ✅ | ✅ | ❌ | ✅ | ✅ |
| Multi-Table Joins | ✅ | ✅ | ✅ | ✅ | ✅ |
| `LATERAL` / `APPLY` | ✅ (`LATERAL`) | ✅ (`LATERAL`) | ✅ (`APPLY`) | ✅ (`LATERAL`) | ❌ |

---

## References

- Microsoft Learn — FROM clause with JOIN (ON clause) - https://learn.microsoft.com/en-us/sql/t-sql/queries/from-transact-sql
- Microsoft Learn — Join Fundamentals (ON clause) - https://learn.microsoft.com/en-us/previous-versions/sql/sql-server-2005/ms191517(v=sql.90)
- Microsoft Learn — Tables in Joins (Combining Three or More Tables) - https://learn.microsoft.com/en-us/previous-versions/he95de78(v=vs.140)
- Microsoft Learn — Join Tables (Combining Three or More Tables) - https://learn.microsoft.com/en-us/previous-versions/
- PostgreSQL Documentation — JOIN Types (ON, USING) - https://www.postgresql.org/docs/current/queries-table-expressions.html
- PostgreSQL Documentation — FROM Clause (ON, USING) - https://www.postgresql.org/files/documentation/pdf/7.3/reference-7.3.2-US.pdf
- PostgreSQL Documentation — SELECT (USING shorthand) - https://manpages.debian.org/trixie/postgresql-client-17/SELECT.7.en.html
- Oracle Database SQL Tuning Guide — Equijoins and Nonequijoins - https://docs.oracle.com/en/database/oracle/oracle-database/18/tgsql/joins.html
- Oracle Database SQL Language Reference — SELECT (USING clause) - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/SELECT.html
- Oracle Database SQL Language Reference — Joins (three or more tables) - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/SELECT.html
- LearnSQL — Practical Examples of When to Use Non-Equi JOINs in SQL - https://learnsql.com/blog/non-equi-joins/
- LearnSQL — How to Join 3 Tables (or More) in SQL - https://learnsql.com/blog/how-to-join-3-tables-or-more-in-sql/
- Stack Overflow — Equi join vs non-equi join - https://stackoverflow.com/
- Apache Derby — INNER JOIN USING clause - https://db.apache.org/derby/docs/
- MIT — INNER JOIN Operation (ON booleanExpression) - https://www.mit.edu/
- GitHub — Multi-Table Joins (SQL Engineering Handbook) - https://github.com/theammarngp-makes/SQL-Engineering-Handbook
- ISO/IEC 9075-2 — SQL Standard (Joined Tables, ON/USING) - https://jtc1sc32.org/doc/N0251-0300/32N0254.pdf