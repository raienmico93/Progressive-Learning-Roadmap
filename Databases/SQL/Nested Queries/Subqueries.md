# SQL Subquery Fundamentals: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A subquery is a `SELECT` statement nested inside another SQL statement—often called the outer query—that returns a result set used by the outer statement to filter rows, compute values, or derive tables.

**Technical Definition**
In SQL, a subquery (also called a sub-select or nested select) is a complete `SELECT` expression enclosed in parentheses that appears within another SQL statement. The outer statement may be a `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `SET`, or `DO` statement. Subqueries can return a scalar value (one row, one column), a row (one row, multiple columns), a column (multiple rows, one column), or a table (multiple rows and columns). They may be self-contained (non-correlated)—executed once and independent of the outer query—or correlated—referencing columns from the outer query and conceptually evaluated once per outer row. Subqueries can appear in the `SELECT` list, `FROM` clause (as derived tables), `WHERE` clause, `HAVING` clause, and `JOIN` clauses.

**Beginner-Friendly Explanation**
A subquery is a query inside a query. Imagine you are writing a query to find employees who earn more than the company average. You could first write a separate query to calculate the average salary, note the result, and then write a second query using that number. A subquery lets you combine both steps into one statement: the inner query (the subquery) calculates the average, and the outer query uses that result to filter employees. Subqueries make complex questions expressible as a single, readable SQL statement.

---

### Key Characteristics

1. **Parentheses Are Mandatory**: A subquery must always be enclosed in parentheses.
2. **Result Shape Determines Usage**: The number of rows and columns a subquery returns determines where it can be used. Scalar subqueries (one value) can appear in expressions; row subqueries (one row, multiple columns) support composite comparisons; table subqueries (multiple rows and columns) can be used in `FROM` or `JOIN` clauses.
3. **Correlated vs. Non-Correlated**: Non-correlated subqueries execute once and their result is reused; correlated subqueries reference outer columns and execute once per outer row, often creating nested-loop execution patterns.
4. **Placement Flexibility**: Subqueries can appear in the `SELECT` list, `FROM` clause, `WHERE` clause, `HAVING` clause, and `JOIN` clauses, each with different semantics and performance implications.
5. **Readability vs. Performance Trade-offs**: Subqueries can make queries more readable and modular, but correlated subqueries and derived tables can introduce performance overhead compared to joins or CTEs.
6. **NULL Sensitivity**: Scalar subqueries returning no rows yield `NULL`; `NOT IN` with `NULL`s in the subquery result can produce unexpected behaviour. `EXISTS` and `NOT EXISTS` are often safer for existence checks.

---

### Prerequisites

- **Basic `SELECT` syntax**: `SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`.
- **Comparison operators**: `=`, `<>`, `>`, `<`, `IN`, `ANY`, `ALL`, `EXISTS`.
- **`NULL` semantics**: How `NULL` behaves in comparisons and subquery results.
- **Table aliases**: Required for correlated subqueries and derived tables.
- **Execution plan basics**: To reason about correlated subquery performance.

---

### Related Programming Areas

| Area | Connection |
|------|------------|
| **Query Composition** | Subqueries allow complex logic to be broken into modular, readable steps. |
| **Data Filtering** | Subqueries in `WHERE` and `HAVING` enable dynamic, data-driven filtering. |
| **Derived Tables & CTEs** | Subqueries in `FROM` are the foundation of derived tables and CTEs. |
| **Query Performance** | Correlated subqueries can cause N×M nested loops; joins and CTEs are often faster. |
| **Data Existence Checks** | `EXISTS`/`NOT EXISTS` subqueries are the standard way to test for related data. |

---

### Core Concepts / Features

1. **What Subqueries Are**
2. **Subquery Types** — Scalar, Row, Table, Correlated, Non-Correlated, Single-Row, Multi-Row
3. **Subquery Placement** — `SELECT`, `FROM`, `WHERE`, `HAVING`, `JOIN`

---

## 1. What Subqueries Are

### Definitions

**Core Definition**
A subquery is a `SELECT` statement nested inside another SQL statement, enclosed in parentheses, whose result is consumed by the outer statement.

**Technical Definition**
A subquery is a complete query expression that appears within the scope of an outer query. The outer query is the statement containing the subquery; the subquery is the nested `SELECT`. The subquery is evaluated (either once or per outer row, depending on correlation) and its result is used by the outer query in one of several ways: as a scalar value in an expression, as a set of values for `IN`/`ANY`/`ALL`, as a row for composite comparison, or as a table in the `FROM` clause. MySQL documentation notes that subqueries allow queries to be structured so that each part can be isolated, and they provide alternative ways to perform operations that would otherwise require complex joins and unions.

**Beginner-Friendly Explanation**
A subquery is a "query within a query." The outer query is the main question; the subquery is a smaller question whose answer the main question uses. For example, "find employees who earn more than the average salary"—the subquery calculates the average, and the outer query uses that number to filter employees.

### Purposes

- To break complex queries into modular, readable steps.
- To filter rows based on the result of another query.
- To compute intermediate values (averages, totals, maxima) for use in the outer query.
- To check for the existence of related data.
- To derive temporary tables for joining or further processing.

### Syntax Rules and Structure

**General Syntax**
```
SELECT select_list
FROM table
WHERE column operator (SELECT column FROM table2 WHERE condition);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Outer query | The main `SELECT` statement. |
| `(` and `)` | Parentheses enclosing the subquery (required). |
| Subquery | The nested `SELECT` statement. |
| Operator | Comparison operator (`=`, `IN`, `EXISTS`, etc.) connecting outer column to subquery result. |

**Syntax Rules**

1. Subqueries must be enclosed in parentheses.
2. The subquery may contain `DISTINCT`, `GROUP BY`, `ORDER BY`, `LIMIT`, joins, and other `SELECT` clauses, subject to vendor restrictions.
3. The subquery's `SELECT` list must have a single column when used with `IN`, `ANY`, or `ALL` (unless row constructors are used).
4. The subquery cannot contain a `WITH` clause in some systems (e.g., PostgreSQL subqueries do not support `WITH` inside the subquery itself in certain contexts).

**Constraints and Limitations**

- Correlated subqueries can be performance-intensive on large tables.
- `NOT IN` with `NULL`s in the subquery result can return no rows; `NOT EXISTS` is often safer.
- Scalar subqueries returning more than one row cause runtime errors.

### Annotated Code Examples

**Example 1: Basic Subquery in `WHERE`**

```sql
SELECT employee_id, first_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

**Expected Output (illustrative)**

| employee_id | first_name | salary |
|-------------|------------|--------|
| 101         | Alice      | 85000  |
| 102         | Bob        | 92000  |

**Why This Output Occurs**: The subquery computes the average salary once; the outer query returns employees whose salary exceeds that average.

---

### Real-World Cases

- **Dynamic filtering**: "Show orders whose amount is above the average order amount."
- **Existence checks**: "Find customers who have placed at least one order."
- **Intermediate calculations**: "Compare each employee's salary to the department average."

---

## 2. Subquery Types

### 2.1 Scalar Subqueries

#### Definitions

**Core Definition**
A scalar subquery returns exactly one row with exactly one column—a single value.

**Technical Definition**
A scalar subquery is a `SELECT` query in parentheses that returns a single value (one row, one column). PostgreSQL documentation states that it is an error for a scalar subquery to return more than one row or more than one column; if the subquery returns zero rows, the scalar result is `NULL`. Scalar subqueries are commonly used in the `SELECT` list, `WHERE` clause, and `HAVING` clause.

**Beginner-Friendly Explanation**
A scalar subquery gives you one answer—one number, one date, one name. You can use it anywhere you would use a literal value.

#### Purposes

- To compute a single aggregate value (average, total, maximum) for use in the outer query.
- To look up a single related value (e.g., a customer's name by ID).
- To compare each row to a single computed benchmark.
- To embed dynamic values in `SELECT` expressions.

#### Syntax Rules and Structure

```
SELECT ( scalar_subquery ) FROM table;
WHERE column operator ( scalar_subquery );
```

**Syntax Rules**

1. The subquery must return exactly one column and at most one row.
2. Zero rows → result is `NULL`.
3. More than one row → runtime error.

**Constraints and Limitations**

- Cannot return multiple rows or multiple columns.
- Correlated scalar subqueries execute per outer row and can be slow.

#### Annotated Code Examples

**Example 1: Scalar Subquery in `SELECT`**

```sql
SELECT username, level,
       (SELECT AVG(level) FROM Players) AS avg_level
FROM Players;
```

**Expected Output (based on Google Cloud documentation)**

| username | level | avg_level |
|----------|-------|-----------|
| gorbie   | 29    | 24.66     |
| junelyn  | 2     | 24.66     |
| corba    | 43    | 24.66     |

**Why This Output Occurs**: The scalar subquery computes the average level once and attaches it to each row.

**Example 2: Scalar Subquery in `WHERE`**

```sql
SELECT order_id, total_amount
FROM orders
WHERE total_amount > (SELECT AVG(total_amount) FROM orders);
```

**Expected Output (illustrative)**

| order_id | total_amount |
|----------|--------------|
| 1001     | 250.00       |
| 1003     | 300.00       |

**Why This Output Occurs**: The subquery returns the average order amount; the outer query filters orders above that average.

#### Real-World Cases

- **Compensation benchmarking**: Comparing each employee's salary to the company average.
- **Sales reporting**: Finding orders above the average order value.
- **Data validation**: Checking whether a value equals the maximum in its category.

---

### 2.2 Row Subqueries

#### Definitions

**Core Definition**
A row subquery returns exactly one row but may contain multiple columns, allowing composite value comparison.

**Technical Definition**
A row subquery is a subquery variant that returns a single row and can return more than one column value. It is used with row constructors—parenthesised lists of columns—to perform element-wise comparison. For example, `WHERE (col1, col2) = (SELECT col3, col4 FROM t2 WHERE id = 10)` compares two columns against two subquery columns simultaneously.

**Beginner-Friendly Explanation**
A row subquery gives you one row with several fields—like a single record—and lets you compare multiple columns at once against the outer query.

#### Purposes

- To compare composite keys (multiple columns) against a single row.
- To avoid verbose `AND` chains when matching on multiple columns.
- To retrieve a complete record from a related table for comparison.

#### Syntax Rules and Structure

```
WHERE (col1, col2) = (SELECT col3, col4 FROM table WHERE condition);
```

**Syntax Rules**

1. The subquery must return exactly one row.
2. Both sides of the comparison must have the same number of columns.
3. If either side contains `NULL`, the comparison resolves to `NULL` (not `TRUE`).
4. More than one row from the subquery triggers an error; zero rows make the comparison `NULL`.

**Constraints and Limitations**

- Only one row is allowed; multiple rows cause errors.
- `NULL` handling can make comparisons return `UNKNOWN` rather than `FALSE`.

#### Annotated Code Examples

**Example 1: Row Subquery Comparison**

```sql
SELECT employee_id, first_name, last_name
FROM employees
WHERE (department_id, manager_id) = (
  SELECT department_id, manager_id
  FROM departments
  WHERE location_id = 1800
);
```

**Explanation**: The row subquery returns one row with `department_id` and `manager_id`; the outer query finds employees matching both values simultaneously.

#### Real-World Cases

- **Composite key matching**: Finding records that match a specific combination of department and manager.
- **Data validation**: Comparing a set of columns against a reference record.

---

### 2.3 Table Subqueries

#### Definitions

**Core Definition**
A table subquery returns a full result set—multiple rows and multiple columns—and is used in the `FROM` or `JOIN` clause as a derived table.

**Technical Definition**
A table subquery (also called a derived table) is a subquery in the `FROM` clause that acts as a temporary result set. It can be joined, filtered, and aggregated like any real table. MySQL documentation categorises table subqueries as returning "a table (one or more rows of one or more columns)".

**Beginner-Friendly Explanation**
A table subquery is a query result that you treat as if it were a table. You can join it to other tables just like a regular table.

#### Purposes

- To join pre-aggregated or pre-filtered results without creating temporary tables.
- To simplify complex queries by isolating intermediate results.
- To enable condition pushdown optimisations in derived tables.

#### Syntax Rules and Structure

```
SELECT ...
FROM (SELECT ... FROM table WHERE ...) AS derived_alias
[JOIN ...];
```

**Syntax Rules**

1. The subquery must have an alias.
2. It cannot reference columns from other tables in the same `FROM` clause (unless using `LATERAL`).
3. It can be joined using any join type.

**Constraints and Limitations**

- Derived tables may prevent predicate pushdown if the optimiser cannot merge them.
- Without an alias, the query is invalid.

#### Annotated Code Examples

**Example 1: Derived Table Join**

```sql
SELECT c.customer_name, totals.total_spent
FROM customers c
JOIN (
  SELECT customer_id, SUM(amount) AS total_spent
  FROM orders
  GROUP BY customer_id
) AS totals ON c.customer_id = totals.customer_id;
```

**Expected Output (illustrative)**

| customer_name | total_spent |
|---------------|-------------|
| Alice         | 550         |
| Bob           | 150         |

**Why This Output Occurs**: The subquery aggregates orders per customer; the outer query joins the aggregated result to the customers table.

#### Real-World Cases

- **Pre-aggregation**: Joining summary statistics to detail tables.
- **Complex filtering**: Building a filtered subset and joining it to other tables.

---

### 2.4 Correlated Subqueries

#### Definitions

**Core Definition**
A correlated subquery is a subquery that references one or more columns from the outer query, causing it to be evaluated once per outer row.

**Technical Definition**
A correlated subquery depends on the outer query for its values. It cannot be executed independently of the outer query. SQL Server documentation explains that correlated subqueries reference a column in the outer query and conceptually execute once per row. PostgreSQL documentation notes that correlated subqueries can become performance traps because they may introduce nested-loop execution.

**Beginner-Friendly Explanation**
A correlated subquery is a subquery that "looks at" the current row of the outer query. It runs again for every row the outer query processes—like a per-row lookup.

#### Purposes

- To perform per-row existence checks (e.g., "does this customer have any orders?").
- To retrieve the most recent or maximum value per group.
- To compare each row to a group-level aggregate without using window functions.

#### Syntax Rules and Structure

```
SELECT ...
FROM outer_table o
WHERE column operator (
  SELECT ... FROM inner_table i WHERE i.col = o.col
);
```

**Syntax Rules**

1. The subquery references at least one column from the outer query.
2. Table aliases are required to disambiguate the outer and inner tables.
3. The subquery is evaluated once for each row processed by the outer query.

**Constraints and Limitations**

- Can be very slow on large tables due to N×M nested-loop execution.
- Cannot be executed separately from the outer query, complicating testing and debugging.
- PostgreSQL can only execute scalar correlated subqueries as nested loops.

#### Annotated Code Examples

**Example 1: Correlated Subquery for Most Recent Order**

```sql
SELECT SalesOrderID, CustomerID, OrderDate
FROM SalesLT.SalesOrderHeader AS o1
WHERE SalesOrderID = (
  SELECT MAX(SalesOrderID)
  FROM SalesLT.SalesOrderHeader AS o2
  WHERE o2.CustomerID = o1.CustomerID
)
ORDER BY CustomerID, OrderDate;
```

**Expected Output (illustrative)**

| SalesOrderID | CustomerID | OrderDate  |
|--------------|------------|------------|
| 71782        | 29485      | 2008-03-11 |
| 71783        | 29486      | 2008-04-01 |

**Why This Output Occurs**: For each customer row, the subquery finds the maximum order ID for that customer; the outer query returns only that most recent order.

#### Real-World Cases

- **Latest record per entity**: Most recent order per customer.
- **Existence checks**: Customers who have at least one order.
- **Per-row comparisons**: Employees earning more than their department average.

---

### 2.5 Non-Correlated Subqueries

#### Definitions

**Core Definition**
A non-correlated (self-contained) subquery is a subquery that does not reference any columns from the outer query and can be executed independently.

**Technical Definition**
A self-contained subquery is independent of the outer query; it executes once and returns its result to the outer query. Non-correlated subqueries are evaluated once, and their result is used as a constant or set by the outer query. This makes their performance predictable and often better than correlated subqueries.

**Beginner-Friendly Explanation**
A non-correlated subquery is a subquery that does not need anything from the outer query. It runs once, produces a result, and the outer query uses that result.

#### Purposes

- To compute a constant benchmark (average, total, maximum) once for the entire query.
- To filter rows against a static set of values from another table.
- To improve performance by avoiding per-row subquery execution.

#### Syntax Rules and Structure

```
SELECT ...
FROM table
WHERE column operator (SELECT ... FROM other_table WHERE condition);
```

**Syntax Rules**

1. The subquery does not reference outer-query columns.
2. It is evaluated once.
3. Its result is treated as a constant or set for the outer query.

**Constraints and Limitations**

- If the subquery returns multiple rows and is used with `=`, an error occurs.
- `NOT IN` with `NULL`s from a non-correlated subquery can produce no rows.

#### Annotated Code Examples

**Example 1: Non-Correlated Subquery**

```sql
SELECT order_id, total_amount
FROM orders
WHERE total_amount > (SELECT AVG(total_amount) FROM orders);
```

**Explanation**: The subquery computes the average once; the outer query compares every row to that single value.

#### Real-World Cases

- **Benchmark comparisons**: Products priced above the overall average.
- **Static filtering**: Customers in a specific set of cities.

---

### 2.6 Single-Row Subqueries

#### Definitions

**Core Definition**
A single-row subquery returns zero or one row and is used with single-row comparison operators (`=`, `<`, `>`, `<=`, `>=`, `<>`).

**Technical Definition**
A single-row subquery returns at most one row. It is used with scalar comparison operators. If it returns zero rows, the comparison evaluates to `UNKNOWN` (effectively `NULL`); if it returns more than one row, a runtime error occurs. Single-row subqueries may be scalar (one column) or row (multiple columns) in shape.

**Beginner-Friendly Explanation**
A single-row subquery gives you one row—either one value (scalar) or one record (row). You use it with operators like `=` or `>`.

#### Purposes

- To compare against a single value or single record.
- To enforce cardinality expectations (error if more than one row).
- To support composite comparisons with row subqueries.

#### Syntax Rules and Structure

```
WHERE column = (single-row subquery);
WHERE (col1, col2) = (single-row subquery);
```

**Syntax Rules**

1. Returns zero or one row.
2. Zero rows → comparison is `NULL`.
3. More than one row → error.
4. Can be scalar or row-shaped.

**Constraints and Limitations**

- Cardinality violations cause runtime errors.
- `NULL` results require careful handling.

#### Annotated Code Examples

**Example 1: Single-Row Scalar Subquery**

```sql
SELECT employee_id, salary
FROM employees
WHERE salary = (SELECT MAX(salary) FROM employees);
```

**Expected Output**: The employee(s) with the maximum salary.

---

### 2.7 Multi-Row Subqueries

#### Definitions

**Core Definition**
A multi-row subquery returns zero or more rows and is used with set-oriented operators (`IN`, `ANY`, `ALL`, `EXISTS`).

**Technical Definition**
A multi-row subquery returns a set of rows (one or more columns). It is used with operators that accept sets: `IN` (equality against any row), `ANY`/`SOME` (comparison against any row), `ALL` (comparison against all rows), and `EXISTS`/`NOT EXISTS` (existence check).

**Beginner-Friendly Explanation**
A multi-row subquery returns a list of values or rows. You use it with operators like `IN` ("is this value in the list?") or `EXISTS` ("does the list have any rows?").

#### Purposes

- To filter rows based on membership in a set.
- To compare against any or all values in a set.
- To check for existence of related data.

#### Syntax Rules and Structure

```
WHERE column IN (multi-row subquery);
WHERE column > ANY (multi-row subquery);
WHERE column > ALL (multi-row subquery);
WHERE EXISTS (multi-row subquery);
```

**Syntax Rules**

1. `IN` returns `TRUE` if the value matches any row in the subquery result.
2. `ANY`/`SOME` returns `TRUE` if the comparison is true for at least one row.
3. `ALL` returns `TRUE` if the comparison is true for every row.
4. `EXISTS` returns `TRUE` if the subquery returns at least one row.

**Constraints and Limitations**

- `NOT IN` with `NULL`s returns no rows; use `NOT EXISTS` instead.
- `IN` with a subquery returning multiple columns is invalid (unless using row constructors).

#### Annotated Code Examples

**Example 1: `IN` Multi-Row Subquery**

```sql
SELECT 'corba' IN (SELECT username FROM Players) AS result;
```

**Expected Output (Google Cloud documentation)**

| result |
|--------|
| TRUE   |

**Why This Output Occurs**: The subquery returns the set of usernames; `IN` checks whether `'corba'` is in that set.

#### Real-World Cases

- **Set membership**: Customers in a list of target cities.
- **Existence checks**: Products that have at least one order.

---

## 3. Subquery Placement

### 3.1 `SELECT` Clause

#### Definitions

**Core Definition**
A subquery in the `SELECT` clause appears as a column expression and must return a scalar value.

**Technical Definition**
A scalar subquery in the `SELECT` list computes a single value for each output row. It is often correlated, computing a per-row value (e.g., a count of related rows or the maximum value in a related table).

**Beginner-Friendly Explanation**
A subquery in the `SELECT` clause adds an extra column to your result—like "for each customer, show their total order count."

#### Purposes

- To add per-row computed values to the result set.
- To avoid a separate join when only a single value per row is needed.
- To compute running or contextual metrics.

#### Syntax Rules and Structure

```
SELECT column1, (scalar_subquery) AS alias, column2 FROM table;
```

**Syntax Rules**

1. The subquery must be scalar (one row, one column).
2. It is often correlated to the outer row.
3. It must be enclosed in parentheses.

**Constraints and Limitations**

- Correlated scalar subqueries in `SELECT` can be slow on large datasets.
- If the subquery returns multiple rows, an error occurs.

#### Annotated Code Examples

**Example 1: Scalar Subquery in `SELECT`**

```sql
SELECT username, level,
       (SELECT AVG(level) FROM Players) AS avg_level
FROM Players;
```

**Expected Output (Google Cloud documentation)**:

| username | level | avg_level |
|----------|-------|-----------|
| gorbie   | 29    | 24.66     |
| junelyn  | 2     | 24.66     |
| corba    | 43    | 24.66     |

**Why This Output Occurs**: The subquery computes the average level once and attaches it to each row.

---

### 3.2 `FROM` Clause

#### Definitions

**Core Definition**
A subquery in the `FROM` clause acts as a derived table—a temporary result set that can be joined, filtered, and aggregated.

**Technical Definition**
A table subquery in `FROM` produces a derived table. It must have an alias. MySQL documentation describes derived tables as subqueries in the `FROM` clause that act as temporary result sets.

**Beginner-Friendly Explanation**
A subquery in `FROM` is a query result that you treat as a table. You can join it to other tables.

#### Purposes

- To join pre-aggregated or pre-filtered results.
- To isolate complex logic into a named derived table.
- To enable condition pushdown optimisations.

#### Syntax Rules and Structure

```
SELECT ...
FROM (SELECT ...) AS derived_alias
JOIN ...;
```

**Syntax Rules**

1. The subquery must have an alias.
2. It cannot reference columns from other tables in the same `FROM` clause (unless using `LATERAL`).
3. It can be joined like a base table.

**Constraints and Limitations**

- Derived tables may prevent predicate pushdown if the optimiser cannot merge them.

#### Annotated Code Examples

**Example 1: Derived Table**

```sql
SELECT d.dept_name, avg_sal.avg_salary
FROM departments d
JOIN (
  SELECT department_id, AVG(salary) AS avg_salary
  FROM employees
  GROUP BY department_id
) AS avg_sal ON d.department_id = avg_sal.department_id;
```

**Expected Output (illustrative)**

| dept_name   | avg_salary |
|-------------|------------|
| Engineering | 85000      |
| Sales       | 62000      |

---

### 3.3 `WHERE` Clause

#### Definitions

**Core Definition**
A subquery in the `WHERE` clause filters rows based on the result of another query.

**Technical Definition**
Subqueries in `WHERE` can be scalar (comparison against a single value), multi-row (`IN`, `ANY`, `ALL`), or existence-based (`EXISTS`, `NOT EXISTS`). They are the most common subquery placement.

**Beginner-Friendly Explanation**
A subquery in `WHERE` is the classic "filter by another query's result" pattern.

#### Purposes

- To filter rows based on a computed benchmark.
- To check set membership.
- To test for existence of related data.

#### Syntax Rules and Structure

```
WHERE column operator (subquery);
WHERE column IN (subquery);
WHERE EXISTS (subquery);
```

**Syntax Rules**

1. Scalar subqueries use `=`, `<`, `>`, etc.
2. Multi-row subqueries use `IN`, `ANY`, `ALL`.
3. `EXISTS` checks whether the subquery returns any rows.

**Constraints and Limitations**

- `NOT IN` with `NULL`s returns no rows; use `NOT EXISTS`.
- Correlated subqueries in `WHERE` execute per outer row.

#### Annotated Code Examples

**Example 1: Subquery in `WHERE`**

```sql
SELECT order_id, total_amount
FROM orders
WHERE total_amount > (SELECT AVG(total_amount) FROM orders);
```

**Expected Output (illustrative)**:

| order_id | total_amount |
|----------|--------------|
| 1001     | 250.00       |
| 1003     | 300.00       |

**Why This Output Occurs**: The subquery returns the average; the outer query filters rows above it.

---

### 3.4 `HAVING` Clause

#### Definitions

**Core Definition**
A subquery in the `HAVING` clause filters groups based on an aggregate comparison involving a subquery result.

**Technical Definition**
Subqueries in `HAVING` are used when the filtering condition depends on an aggregate value and a subquery-computed benchmark. For example, `HAVING AVG(quantity) > 2 * (SELECT ...)`. Correlated subqueries in `HAVING` can reference grouped columns.

**Beginner-Friendly Explanation**
A subquery in `HAVING` lets you filter groups based on a comparison to another query's result—like "show product groups whose average quantity exceeds twice the in-stock quantity."

#### Purposes

- To filter aggregated groups against a subquery-computed threshold.
- To compare group aggregates to external benchmarks.
- To eliminate groups based on existence conditions.

#### Syntax Rules and Structure

```
GROUP BY ...
HAVING aggregate_function(...) operator (subquery);
```

**Syntax Rules**

1. The subquery must produce a scalar value or a set compatible with the operator.
2. Correlated subqueries can reference grouped columns.
3. The subquery is evaluated after grouping.

**Constraints and Limitations**

- Complex `HAVING` subqueries can be difficult to optimise.
- `NULL` handling requires care.

#### Annotated Code Examples

**Example 1: Subquery in `HAVING`**

```sql
SELECT name, AVG(quantity)
FROM products
GROUP BY name
HAVING AVG(quantity) > 2 * (
  SELECT AVG(quantity) FROM products
);
```

**Explanation**: The subquery computes the overall average quantity; the `HAVING` clause keeps groups whose average is more than twice that benchmark.

---

### 3.5 `JOIN` Clause

#### Definitions

**Core Definition**
A subquery in a `JOIN` clause is a derived table joined to other tables, functioning identically to a subquery in `FROM`.

**Technical Definition**
A subquery can appear as the right-hand operand of any `JOIN` type (`INNER JOIN`, `LEFT JOIN`, etc.), provided it is aliased. It acts as a derived table for join purposes.

**Beginner-Friendly Explanation**
You can join to a subquery just like you join to a table—useful when you want to join to an aggregated or filtered result.

#### Purposes

- To join pre-aggregated results to detail tables.
- To combine multiple derived tables in a single query.
- To support complex multi-step transformations.

#### Syntax Rules and Structure

```
SELECT ...
FROM table1
JOIN (SELECT ...) AS subquery_alias ON ...;
```

**Syntax Rules**

1. The subquery must have an alias.
2. It can be joined using any join type.
3. Join conditions reference the subquery alias.

**Constraints and Limitations**

- Same as derived tables; optimiser may or may not push predicates into the subquery.

#### Annotated Code Examples

**Example 1: Joining a Subquery**

```sql
SELECT c.customer_name, totals.total_spent
FROM customers c
JOIN (
  SELECT customer_id, SUM(amount) AS total_spent
  FROM orders
  GROUP BY customer_id
) AS totals ON c.customer_id = totals.customer_id;
```

**Expected Output (illustrative)**:

| customer_name | total_spent |
|---------------|-------------|
| Alice         | 550         |
| Bob           | 150         |

---

## References

1. **PostgreSQL Subqueries (Google Cloud Documentation)** — https://docs.cloud.google.com/spanner/docs/reference/postgresql/subqueries
2. **MySQL 8.0 Reference Manual — Subqueries** — https://dev.mysql.com/doc/refman/8.0/en/subqueries.html
3. **Microsoft Learn — Use Self-Contained or Correlated Subqueries** — https://learn.microsoft.com/en-us/training/modules/write-subqueries/4-self-contained-correlated-subqueries
4. **Airbyte — PostgreSQL Subquery: A Complete Guide** — https://airbyte.com/data-engineering-resources/postgresql-subquery
5. **Red-Gate — MySQL Subqueries: Scalar, Correlated, Row, Column, and Derived Tables** — https://www.red-gate.com/simple-talk/databases/mysql/subqueries-in-mysql/
6. **MySQL 8.0 Reference Manual — Row Subqueries** — https://dev.mysql.com/doc/refman/8.0/en/row-subqueries.html
7. **MySQL 8.0 Reference Manual — Derived Tables** — https://dev.mysql.com/doc/refman/8.0/en/derived-tables.html
8. **SAP Help — Subquery in HAVING Clause** — https://help.sap.com/doc/saphelp_ewm900/9.0/en-US/4c/fd8b6b1c6d3a4fe10000000a42189e/content.htm
9. **Microsoft Learn — Subqueries (SQL Server)** — https://learn.microsoft.com/en-us/sql/relational-databases/performance/subqueries

---

This cheat sheet provides a comprehensive reference for SQL subquery fundamentals across PostgreSQL, MySQL, and SQL Server, covering scalar, row, table, correlated, non-correlated, single-row, and multi-row subqueries, and their placement in `SELECT`, `FROM`, `WHERE`, `HAVING`, and `JOIN` clauses. Always consult your specific RDBMS documentation for version-specific behaviour and optimiser features.