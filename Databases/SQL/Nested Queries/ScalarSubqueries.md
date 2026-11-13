# SQL Scalar Subqueries: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
A scalar subquery is a `SELECT` statement nested inside another SQL statement that returns exactly one value—one row with one column—and can be used wherever a single value expression is permitted.

**Technical Definition**
In SQL, a scalar subquery is an ordinary `SELECT` query enclosed in parentheses that returns exactly one row with one column. PostgreSQL documentation states that "it is an error to use a query that returns more than one row or more than one column as a scalar subquery"; if the subquery returns zero rows, the scalar result is taken to be `NULL`. MySQL describes a scalar subquery as "a simple operand" that can be used almost anywhere a single column value or literal is legal, possessing the same characteristics as any operand: a data type, a length, a nullability indication, and so on. SQL Server defines it as "an inner SELECT statement within an outer query, written to return a single value," usable anywhere a single-valued expression is permitted—in `SELECT`, `WHERE`, `HAVING`, or even `FROM` clauses. Scalar subqueries may be self-contained (non-correlated), executing once and folding into the plan, or correlated, executing once per outer row and often introducing nested-loop execution.

**Beginner-Friendly Explanation**
A scalar subquery is a query that gives you exactly one answer. Imagine you are writing a report and need to compare each employee's salary to the company average. You could calculate the average separately, write it down, and then use that number in your main query. A scalar subquery lets you combine both steps: the inner query calculates the average, and the outer query uses that single value—just as if you had typed it in yourself. It is the SQL equivalent of a sticky note with one number written on it.

### Key Characteristics

1. **Cardinality Constraint**: A scalar subquery must return exactly one row and one column. Zero rows yield `NULL`; more than one row or more than one column is an error.
2. **Operand-Like Behaviour**: A scalar subquery behaves like a literal value—it has a data type, a length, and nullability characteristics, and it can participate in expressions, comparisons, and function calls.
3. **Placement Flexibility**: Scalar subqueries can appear in the `SELECT` list, `WHERE` clause, `HAVING` clause, and—in SQL Server—even the `FROM` clause.
4. **Correlation Option**: Scalar subqueries may reference columns from the outer query (correlated) or be entirely self-contained (non-correlated). Non-correlated subqueries are evaluated once; correlated subqueries execute per outer row.
5. **Error Sensitivity**: The cardinality guarantee is enforced at runtime. A subquery that is assumed to return one row but returns multiple rows causes a runtime error—a common source of production failures.
6. **Performance Variability**: Non-correlated scalar subqueries are generally safe and predictable because they are evaluated once. Correlated scalar subqueries can become "performance traps" because they force per-row evaluation and may introduce nested-loop execution.

### Prerequisites

- **Basic `SELECT` syntax**: `SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`.
- **Aggregate functions**: `AVG`, `MAX`, `MIN`, `COUNT`, `SUM`—aggregates without `GROUP BY` guarantee a single row, making them safe anchors for scalar subqueries.
- **Comparison operators**: `=`, `<`, `>`, `<=`, `>=`, `<>`.
- **`NULL` semantics**: How `NULL` behaves in comparisons and expressions.
- **Table aliases**: Required for correlated scalar subqueries.

### Related Programming Areas

| Area | Connection |
|------|------------|
| **Query Composition** | Scalar subqueries embed one query's result inside another's expression. |
| **Dynamic Filtering** | Comparing each row to a computed benchmark (average, maximum, total). |
| **Per-Row Lookups** | Correlated scalar subqueries retrieve a single related value per outer row. |
| **Query Performance** | Correlated scalar subqueries are a known performance anti-pattern; joins and window functions are often faster. |
| **Error Prevention** | Cardinality violations are a common source of runtime errors in production SQL. |

### Core Concepts / Features

1. **Single-Value Subqueries** — scalar subqueries returning exactly one value
2. **Single-Row Subqueries** — subqueries returning exactly one row (possibly multiple columns)
3. **Single-Column Subqueries** — subqueries returning one column (possibly multiple rows)
4. **Multiple-Row Subqueries** — subqueries returning more than one row
5. **Multiple-Column Subqueries** — subqueries returning more than one column
6. **Error Handling** — cardinality violations and NULL behaviour
7. **Performance** — correlated vs. non-correlated, joins, and optimiser transformations

---

## 1. Single-Value Subqueries

### Definitions

**Core Definition**
A single-value subquery is a scalar subquery that returns exactly one value—one row with one column—and is used as a scalar operand in an expression.

**Technical Definition**
A single-value subquery is the canonical form of a scalar subquery: a parenthesised `SELECT` that yields a single row and a single column. PostgreSQL's grammar treats any subquery inside an expression as a scalar subquery, and the returned value is used in the surrounding value expression. MySQL documentation confirms that the subquery "returns a single value" with a specific data type, length, and nullability, just like any other operand. When the subquery returns zero rows, the result is `NULL`, even if the selected column is declared `NOT NULL`.

**Beginner-Friendly Explanation**
A single-value subquery gives you one answer that you can use anywhere you would use a number or a text value. For example, you can compare every row's price to "the average price"—the subquery calculates the average, and the outer query uses that one number.

### Purposes

- To compute a single aggregate value (average, maximum, total) once and use it as a benchmark for the outer query.
- To look up a single related value by a unique identifier.
- To embed a dynamic value in a `SELECT` expression without a join.
- To compare each row to a single computed figure.

### Syntax Rules and Structure

**General Syntax**
```
SELECT ( scalar_subquery ) FROM table;
WHERE column operator ( scalar_subquery );
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `(` and `)` | Parentheses enclosing the subquery (mandatory). |
| `scalar_subquery` | A `SELECT` statement returning one row and one column. |
| `operator` | Comparison operator (`=`, `<`, `>`, etc.) or expression context. |

**Syntax Rules**

1. The subquery must be enclosed in parentheses, even when it is the sole argument to a function.
2. The subquery must return exactly one column; selecting multiple columns is almost always an error.
3. The subquery must return at most one row. More than one row is a runtime error; zero rows yield `NULL`.
4. Aggregates without `GROUP BY` (e.g., `AVG(column)`) always return exactly one row, making them the safest anchor for scalar subqueries.

**Constraints and Limitations**

- A scalar subquery cannot be used where only a literal value is permitted (e.g., `LIMIT` requires a literal integer; `LOAD DATA` requires a literal file name).
- Correlated scalar subqueries execute per outer row and can be slow on large datasets.
- The planner may treat a scalar subquery as a "black box," leading to suboptimal selectivity estimates in PostgreSQL.

### Annotated Code Examples

**Example 1: Single-Value Subquery in `SELECT`**

```sql
SELECT username, level,
       (SELECT AVG(level) FROM Players) AS avg_level
FROM Players;
```

**Step-by-Step Explanation**

1. The subquery `(SELECT AVG(level) FROM Players)` computes the average level across all players.
2. Because it is an aggregate without `GROUP BY`, it returns exactly one row and one column.
3. The outer query attaches this single value to every row as `avg_level`.

**Expected Output (Google Cloud documentation)**:

| username | level | avg_level |
|----------|-------|-----------|
| gorbie   | 29    | 24.66     |
| junelyn  | 2     | 24.66     |
| corba    | 43    | 24.66     |

**Why This Output Occurs**: The scalar subquery computes the average once and reuses it for each outer row.

---

**Example 2: Single-Value Subquery in `WHERE`**

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

**Why This Output Occurs**: The subquery returns the average order amount; the outer query filters orders above that threshold. Because the subquery is non-correlated, it is evaluated once and folded into the plan.

---

**Example 3: Correlated Single-Value Subquery**

```sql
SELECT username,
       (SELECT mascot FROM Mascots WHERE Players.team = Mascots.team) AS player_mascot
FROM Players;
```

**Expected Output (Google Cloud documentation)**:

| username | player_mascot |
|----------|---------------|
| gorbie   | cardinal      |
| junelyn  | finch         |
| corba    | parrot        |

**Why This Output Occurs**: The subquery references `Players.team` from the outer query, so it executes once per player row, looking up the mascot for that player's team.

### Real-World Cases

- **Compensation benchmarking**: Comparing each employee's salary to the company average.
- **Sales reporting**: Finding orders above the average order value.
- **Dynamic thresholds**: Filtering products priced above the maximum price in a competitor category.

---

## 2. Single-Row Subqueries

### Definitions

**Core Definition**
A single-row subquery returns exactly one row but may return multiple columns, enabling composite (row-wise) comparisons.

**Technical Definition**
A single-row subquery is a subquery that returns exactly one row and can return more than one column value. MySQL documentation distinguishes row subqueries from scalar subqueries: a row subquery returns a single row and can therefore return multiple column values, whereas a scalar subquery returns a single value. Row constructors—parenthesised lists of columns—are used to compare multiple columns against the subquery's multiple columns simultaneously. Valid operators include `=`, `>`, `<`, `>=`, `<=`, `<>`, and `!=`.

**Beginner-Friendly Explanation**
A single-row subquery gives you one complete record—like one row from a spreadsheet—rather than just one cell. You can compare multiple columns at once: "find the employee whose department and manager match the department and manager of the engineering department."

### Purposes

- To compare composite keys (multiple columns) against a single row from another table.
- To avoid verbose `AND` chains when matching on multiple columns.
- To retrieve a complete reference record for row-wise comparison.
- To enforce single-row cardinality while allowing multi-column results.

### Syntax Rules and Structure

**General Syntax**
```
WHERE (col1, col2) = (SELECT col3, col4 FROM table WHERE condition);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `(col1, col2)` | Row constructor on the left side. |
| `operator` | `=`, `>`, `<`, etc. |
| `(SELECT col3, col4 ...)` | Single-row subquery returning multiple columns. |

**Syntax Rules**

1. The subquery must return exactly one row; multiple rows trigger an error.
2. Both sides must have the same number of columns.
3. If the subquery returns zero rows, the comparison evaluates to `NULL` (not `TRUE`).
4. A row constructor cannot be used with a subquery that returns only one column; a single-column subquery is treated as a scalar value, not a row.

**Constraints and Limitations**

- Cardinality violations (more than one row) cause runtime errors.
- `NULL` values in either side make the comparison resolve to `NULL`, which does not pass a `WHERE` filter.
- Row subqueries are less portable in their exact syntax across RDBMS, though the core concept is supported.

### Annotated Code Examples

**Example 1: Single-Row Composite Comparison**

```sql
SELECT employee_id, first_name, last_name
FROM employees
WHERE (department_id, manager_id) = (
  SELECT department_id, manager_id
  FROM departments
  WHERE location_id = 1800
);
```

**Step-by-Step Explanation**

1. The subquery retrieves the `department_id` and `manager_id` of the department at location 1800.
2. Because `location_id = 1800` is expected to identify one department, the subquery returns one row with two columns.
3. The outer query compares the `(department_id, manager_id)` pair of each employee against that single row.

**Expected Output (illustrative)**:

| employee_id | first_name | last_name |
|-------------|------------|-----------|
| 101         | Alice      | Smith     |
| 102         | Bob        | Jones     |

**Why This Output Occurs**: Only employees whose department and manager combination exactly matches the reference row are returned.

---

**Example 2: Row Subquery with `ROW` Constructor (MySQL)**

```sql
SELECT * FROM t1
WHERE ROW(col1, col2) = (SELECT col3, col4 FROM t2 WHERE id = 10);
```

**Explanation**: The `ROW` constructor is equivalent to the parenthesised list `(col1, col2)`. The subquery returns a single row from `t2`; the comparison is element-wise.

### Real-World Cases

- **Composite key matching**: Finding records that match a specific department-manager combination.
- **Reference data validation**: Comparing a set of columns against a known-good record.
- **ETL reconciliation**: Matching rows on multiple business keys simultaneously.

---

## 3. Single-Column Subqueries

### Definitions

**Core Definition**
A single-column subquery returns one column but may return multiple rows, producing a set of values rather than a single scalar.

**Technical Definition**
A single-column subquery is a subquery that returns a single column of values, potentially with many rows. Unlike a scalar subquery, which must return one row, a single-column subquery is used with set-oriented operators such as `IN`, `ANY`, `SOME`, and `ALL`. MySQL documentation describes column subqueries as returning "a column of values". A single-column subquery cannot be used as a scalar operand because it may return more than one row.

**Beginner-Friendly Explanation**
A single-column subquery gives you a list of values—like a column in a spreadsheet—rather than a single cell. You can ask "is this value in that list?" using `IN`.

### Purposes

- To filter rows based on membership in a set of values.
- To compare against any or all values in a list.
- To retrieve a list of keys for use in an outer query.
- To support `IN` and `ANY`/`ALL` predicates.

### Syntax Rules and Structure

**General Syntax**
```
WHERE column IN (single_column_subquery);
WHERE column > ANY (single_column_subquery);
WHERE column > ALL (single_column_subquery);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `column` | Outer-query column to compare. |
| `IN` / `ANY` / `ALL` | Set-oriented operator. |
| `(single_column_subquery)` | Subquery returning one column, multiple rows. |

**Syntax Rules**

1. The subquery must return exactly one column; multiple columns are an error (unless used with `EXISTS`, which ignores the `SELECT` list).
2. `IN` returns `TRUE` if the value matches any row in the subquery result.
3. `ANY`/`SOME` returns `TRUE` if the comparison is true for at least one row.
4. `ALL` returns `TRUE` if the comparison is true for every row.
5. `NOT IN` with `NULL`s in the subquery result returns no rows; `NOT EXISTS` is safer for anti-join logic.

**Constraints and Limitations**

- `NOT IN` with `NULL`s produces an empty result set—a classic SQL pitfall.
- Single-column subqueries cannot be used where a scalar value is required (e.g., `WHERE col = (subquery)`).

### Annotated Code Examples

**Example 1: `IN` with a Single-Column Subquery**

```sql
SELECT CustomerID, SalesOrderID
FROM Sales.SalesOrderHeader
WHERE CustomerID IN (
  SELECT CustomerID FROM Sales.Customer WHERE CountryRegion = 'Canada'
);
```

**Explanation**: The subquery returns a single column (`CustomerID`) with multiple rows (all Canadian customers). The outer query returns orders for those customers.

**Expected Output (illustrative)**:

| CustomerID | SalesOrderID |
|------------|--------------|
| 29485      | 71782        |
| 29486      | 71783        |

---

**Example 2: `ANY` with a Single-Column Subquery**

```sql
SELECT product_name, price
FROM products
WHERE price > ANY (
  SELECT price FROM products WHERE category = 'Electronics'
);
```

**Explanation**: The subquery returns the prices of all electronics products. `> ANY` returns `TRUE` if the outer product's price is greater than at least one electronics price (i.e., greater than the minimum).

### Real-World Cases

- **Set membership filtering**: Customers in a list of target regions.
- **Threshold comparisons**: Products priced above any competitor product.
- **Existence checks**: Orders placed by any customer in a segment.

---

## 4. Multiple-Row Subqueries

### Definitions

**Core Definition**
A multiple-row subquery is a subquery that returns more than one row, producing a set of rows that must be consumed by a set-oriented operator.

**Technical Definition**
A multiple-row subquery returns a result set with one or more rows. It cannot be used as a scalar operand; it must be used with operators that accept sets: `IN`, `ANY`/`SOME`, `ALL`, or `EXISTS`. SQL Server training documentation distinguishes scalar subqueries (single value) from multi-valued subqueries, which "can return more than one row" but still return a single column.

**Beginner-Friendly Explanation**
A multiple-row subquery gives you a list of rows—like a whole column of a spreadsheet—and you must use it with an operator that understands lists, such as `IN` or `EXISTS`.

### Purposes

- To filter rows based on membership in a multi-row set.
- To compare against any or all rows in a set.
- To check for existence of related data.
- To support `IN`, `ANY`, `ALL`, and `EXISTS` predicates.

### Syntax Rules and Structure

**General Syntax**
```
WHERE column IN (multiple_row_subquery);
WHERE column > ANY (multiple_row_subquery);
WHERE column > ALL (multiple_row_subquery);
WHERE EXISTS (multiple_row_subquery);
```

**Syntax Rules**

1. `IN` compares the outer value to each row's value in the subquery result.
2. `ANY`/`SOME` returns `TRUE` if the comparison holds for at least one row.
3. `ALL` returns `TRUE` if the comparison holds for every row.
4. `EXISTS` returns `TRUE` if the subquery returns at least one row, regardless of column values.
5. A multiple-row subquery cannot be used with `=` or other scalar comparison operators unless wrapped in an aggregate or limited to one row.

**Constraints and Limitations**

- `NOT IN` with `NULL`s returns no rows; use `NOT EXISTS` for anti-join logic.
- `IN` with a subquery returning multiple columns is invalid unless using row constructors.

### Annotated Code Examples

**Example 1: `EXISTS` with a Multiple-Row Subquery**

```sql
SELECT c.customer_name
FROM customers c
WHERE EXISTS (
  SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

**Expected Output (illustrative)**:

| customer_name |
|---------------|
| Alice         |
| Bob           |

**Why This Output Occurs**: For each customer, `EXISTS` checks whether the subquery returns any order rows. Only customers with at least one order are returned; Carol (no orders) is excluded.

---

**Example 2: `NOT EXISTS` (Safer Anti-Join)**

```sql
SELECT c.customer_name
FROM customers c
WHERE NOT EXISTS (
  SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

**Expected Output (illustrative)**:

| customer_name |
|---------------|
| Carol         |

**Why This Output Occurs**: `NOT EXISTS` returns `TRUE` when the subquery returns zero rows, correctly identifying customers without orders. Unlike `NOT IN`, it is not affected by `NULL`s in the subquery result.

### Real-World Cases

- **Existence checks**: Customers who have placed at least one order.
- **Anti-joins**: Products that have never been sold.
- **Set comparisons**: Employees earning more than all managers.

---

## 5. Multiple-Column Subqueries

### Definitions

**Core Definition**
A multiple-column subquery returns more than one column, producing a set of rows where each row has multiple values.

**Technical Definition**
A multiple-column subquery returns a result set with multiple columns. It cannot be used with scalar comparison operators or single-column set operators (`IN`, `ANY`, `ALL`) unless row constructors are used on the left side. MySQL documentation states that a row constructor is used for comparisons with subqueries that return two or more columns; when a subquery returns a single column, it is treated as a scalar value, and a row constructor cannot be used with a subquery that returns fewer than two columns. Multiple-column subqueries are typically used with `EXISTS` (where the column list is irrelevant) or with row-constructor comparisons.

**Beginner-Friendly Explanation**
A multiple-column subquery gives you a table-like result—rows with several columns. You can use it with `EXISTS` to check whether any such rows exist, or compare it row-wise to a matching set of columns using a row constructor.

### Purposes

- To check for the existence of rows with multiple attributes.
- To compare composite values against a set of rows.
- To retrieve multiple columns for row-wise `IN` comparisons.
- To support complex existence predicates.

### Syntax Rules and Structure

**General Syntax (Row Constructor Comparison)**
```
WHERE (col1, col2) IN (SELECT col3, col4 FROM table WHERE condition);
```
**General Syntax (`EXISTS`)**
```
WHERE EXISTS (SELECT col1, col2 FROM table WHERE condition);
```

**Syntax Rules**

1. For `IN`/`ANY`/`ALL`, the left side must be a row constructor with the same number of columns as the subquery.
2. `EXISTS` ignores the subquery's `SELECT` list, so multiple columns are permitted (though `SELECT 1` is conventional).
3. A multiple-column subquery cannot be used as a scalar operand.

**Constraints and Limitations**

- Multiple-column `IN` with row constructors is supported in MySQL and PostgreSQL but may have vendor-specific limitations.
- Cardinality and `NULL` behaviour follow the rules of the specific operator.

### Annotated Code Examples

**Example 1: Multiple-Column `IN` (Row Constructor)**

```sql
SELECT column1, column2, column3
FROM t1
WHERE (column1, column2, column3) IN (
  SELECT column1, column2, column3 FROM t2
);
```

**Explanation**: This finds all rows in `t1` whose three-column combination also exists in `t2`. The subquery returns multiple columns and multiple rows; the row constructor on the left matches the subquery's column count.

---

**Example 2: Multiple-Column `EXISTS`**

```sql
SELECT c.customer_name
FROM customers c
WHERE EXISTS (
  SELECT 1 FROM orders o
  WHERE o.customer_id = c.customer_id
    AND o.amount > 1000
);
```

**Explanation**: The subquery's `SELECT` list (`1`) is irrelevant to `EXISTS`; the subquery can return multiple columns without affecting the existence check.

### Real-World Cases

- **Composite key existence**: Checking whether a combination of keys exists in a reference table.
- **Multi-attribute matching**: Finding records that match on several attributes simultaneously.
- **Complex existence predicates**: Checking for related rows that satisfy multiple conditions.

---

## 6. Error Handling

### Definitions

**Core Definition**
Error handling for scalar subqueries concerns the two cardinality edge cases—zero rows (returns `NULL`) and multiple rows (raises an error)—and the strategies for preventing and managing them.

**Technical Definition**
Scalar subqueries have strict cardinality requirements: exactly one row and one column. PostgreSQL documentation states that "it is an error to use a query that returns more than one row or more than one column as a scalar subquery," but if the subquery returns no rows, "the scalar result is taken to be null". SQL Server raises error 512 ("Subquery returned more than 1 value. This is not permitted when the subquery follows =, !=, <, <=, >, >= or when the subquery is used as an expression") when a scalar subquery returns multiple rows. MySQL raises error 1242 ("Subquery returns more than 1 row").

**Beginner-Friendly Explanation**
A scalar subquery must give exactly one answer. If it gives zero answers, the result is `NULL`—which may or may not be what you want. If it gives more than one answer, the database raises an error and the query fails. Both cases need to be anticipated and handled.

### Purposes (of Understanding Error Handling)

- To prevent runtime failures from multiple-row subqueries.
- To handle `NULL` results gracefully when zero rows are possible.
- To choose between scalar subqueries, joins, and `EXISTS` based on cardinality guarantees.
- To write robust, production-safe SQL.

### Syntax Rules and Structure (Error Patterns and Fixes)

| Error | Cause | Fix |
|-------|-------|-----|
| Multiple rows (error 512 / 1242) | Subquery returns >1 row | Use `TOP 1` / `LIMIT 1`, aggregate (`MAX`, `MIN`), `IN`, `EXISTS`, or join |
| Zero rows (`NULL` result) | Subquery returns no rows | Handle with `COALESCE`, `IS NULL`, or restructure as outer join |
| Multiple columns | Subquery selects >1 column | Select only one column, or use `EXISTS` (which ignores the column list) |

**Syntax Rules (Prevention)**

1. Use aggregates without `GROUP BY` (e.g., `MAX`, `AVG`) to guarantee a single row.
2. Use `TOP 1` / `LIMIT 1` with an `ORDER BY` to deterministically select one row when multiple matches are possible.
3. Use `EXISTS` instead of `=` when the intent is to check for the presence of any matching row.
4. Use `COALESCE((scalar_subquery), default_value)` to substitute a default for `NULL`.

**Constraints and Limitations**

- `TOP 1` without `ORDER BY` returns a nondeterministic row; always pair with `ORDER BY` for predictable results.
- The SQL Server optimiser must add an assert operator to validate the single-row guarantee, which can limit plan transformations and force nested loops.

### Annotated Code Examples

**Example 1: Multiple-Row Error and Fix**

```sql
-- Error: subquery returns multiple rows
SELECT SalesOrderID, ProductID
FROM Sales.SalesOrderDetail
WHERE SalesOrderID = (
  SELECT SalesOrderID FROM Sales.SalesOrderHeader
);
```

**Expected Error (SQL Server)**: Msg 512: Subquery returned more than 1 value.

**Fix using an aggregate**:
```sql
SELECT SalesOrderID, ProductID
FROM Sales.SalesOrderDetail
WHERE SalesOrderID = (
  SELECT MAX(SalesOrderID) FROM Sales.SalesOrderHeader
);
```

**Explanation**: `MAX` guarantees exactly one row, satisfying the scalar requirement.

---

**Example 2: Zero-Row `NULL` Handling**

```sql
SELECT username,
       COALESCE(
         (SELECT mascot FROM Mascots WHERE Players.team = Mascots.team),
         'No mascot'
       ) AS player_mascot
FROM Players;
```

**Explanation**: If a player's team has no mascot, the subquery returns zero rows and the scalar result is `NULL`; `COALESCE` substitutes a descriptive label.

---

**Example 3: `EXISTS` as a Safer Alternative**

```sql
-- Instead of: WHERE col = (SELECT ...)  -- risky
SELECT c.customer_name
FROM customers c
WHERE EXISTS (
  SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

**Explanation**: `EXISTS` does not require a single-row guarantee; it simply checks whether any row exists, eliminating the multiple-row error risk entirely.

### Real-World Cases

- **Production incident**: A scalar subquery that "worked" in testing fails in production when duplicate data is introduced, causing error 512.
- **Data quality**: A lookup subquery returns `NULL` for missing reference data, silently affecting downstream calculations.
- **Safe refactoring**: Replacing `=` subqueries with `EXISTS` or joins to eliminate cardinality risks.

---

## 7. Performance

### Definitions

**Core Definition**
Performance of scalar subqueries refers to the execution cost associated with evaluating the subquery—once for non-correlated subqueries, per-row for correlated subqueries—and the optimiser's ability to transform the subquery into a more efficient plan.

**Technical Definition**
Non-correlated scalar subqueries are evaluated once and folded into the query plan, making performance "safe and predictable even on large tables". Correlated scalar subqueries depend on outer-query columns, execute once per row, and "often introduce nested loops that can slow queries by orders of magnitude". In SQL Server, scalar subqueries in the `SELECT` list require the optimiser to add assert operators to validate the single-row guarantee, which "limits the range of transformations that are possible/considered" and can force nested loops. In PostgreSQL, a scalar subquery may be treated as a "black box" by the planner, leading to poor selectivity estimates and suboptimal plans. Aurora PostgreSQL can accelerate correlated scalar subqueries by transforming them into equivalent outer joins when `apg_enable_correlated_scalar_transform` is enabled.

**Beginner-Friendly Explanation**
A non-correlated scalar subquery is fast because it runs once—like calculating the average salary once and reusing the number. A correlated scalar subquery runs again for every row, which can be slow if the table is large. The database tries to rewrite the query for speed, but sometimes it cannot, and that is when performance suffers.

### Purposes (of Understanding Performance)

- To choose between scalar subqueries, joins, and window functions based on performance characteristics.
- To recognise when a correlated subquery is causing a nested-loop bottleneck.
- To leverage optimiser transformations (e.g., subquery-to-join conversion).
- To write queries that scale with data volume.

### Syntax Rules and Structure (Optimisation Guidelines)

1. **Prefer non-correlated subqueries**: If the subquery does not need outer columns, it is evaluated once and is safe.
2. **Replace correlated scalar subqueries with joins or window functions**: A `LEFT JOIN` or a window function (e.g., `AVG() OVER (PARTITION BY ...)`) often performs better than a per-row scalar subquery.
3. **Use `EXISTS` instead of scalar `=` subqueries for existence checks**: `EXISTS` can short-circuit and does not require a single-row guarantee.
4. **Check the execution plan**: Look for nested loops with a "SubPlan" or "Compute Scalar" operator, which indicates per-row subquery evaluation.
5. **Leverage optimiser transformations**: Aurora PostgreSQL's `apg_enable_correlated_scalar_transform` converts correlated scalar subqueries into outer joins, significantly improving performance as outer row counts grow.

**Constraints and Limitations**

- Optimiser transformations are not guaranteed; the scalar subquery may be evaluated as written.
- SQL Server's assert operator (for single-row validation) can prevent join reordering and force nested loops.
- PostgreSQL's planner may not propagate selectivity estimates through scalar subqueries, leading to suboptimal index choices.

### Annotated Code Examples

**Example 1: Non-Correlated vs. Correlated Performance**

```sql
-- Non-correlated: evaluated once, safe and predictable
SELECT order_id, total_amount
FROM orders
WHERE total_amount > (SELECT AVG(total_amount) FROM orders);

-- Correlated: evaluated per outer row, potential performance trap
SELECT o.order_id,
       (SELECT AVG(amount) FROM order_items oi WHERE oi.order_id = o.order_id) AS avg_item_amount
FROM orders o;
```

**Explanation**: The first subquery computes the average once and is folded into the plan. The second subquery references `o.order_id` and executes once per order row, potentially causing a nested-loop bottleneck.

---

**Example 2: Replacing a Correlated Scalar Subquery with a Window Function**

```sql
-- Before: correlated scalar subquery (slow)
SELECT o.order_id,
       (SELECT SUM(amount) FROM order_items oi WHERE oi.order_id = o.order_id) AS order_total
FROM orders o;

-- After: window function (often faster)
SELECT o.order_id,
       SUM(oi.amount) OVER (PARTITION BY o.order_id) AS order_total
FROM orders o
LEFT JOIN order_items oi ON oi.order_id = o.order_id;
```

**Explanation**: The window function computes the sum per order in a single pass, avoiding per-row subquery execution. This is a common optimisation recommended when scalar subqueries cause performance issues.

---

**Example 3: Optimiser Transformation (Aurora PostgreSQL)**

```sql
SET apg_enable_correlated_scalar_transform TO ON;

SELECT ot.a, ot.b
FROM ot
WHERE ot.b < (SELECT AVG(it.b) FROM it WHERE it.a = ot.a);
```

**Explanation**: With the transformation enabled, Aurora PostgreSQL rewrites the correlated scalar subquery into an equivalent outer join, which can significantly improve performance as the number of outer rows grows.

### Real-World Cases

- **Reporting queries**: Replacing per-row scalar subqueries with joins or window functions to avoid nested-loop performance degradation.
- **Dashboard queries**: Using non-correlated scalar subqueries for static benchmarks (averages, totals) that are computed once.
- **SQL Server plan analysis**: Detecting assert operators and nested loops caused by scalar subqueries in the `SELECT` list and refactoring to `OUTER APPLY` or joins.

---

## References

1. **PostgreSQL Documentation — Scalar Subqueries** — https://www.postgresql.org/docs/current/sql-expressions.html#SQL-SYNTAX-SCALAR-SUBQUERIES 
2. **PostgreSQL Documentation — Subquery Expressions** — https://www.postgresql.org/docs/current/functions-subquery.html
3. **MySQL 8.0 Reference Manual — The Subquery as Scalar Operand** — https://dev.mysql.com/doc/refman/8.0/en/scalar-subqueries.html 
4. **MySQL 8.0 Reference Manual — Row Subqueries** — https://dev.mysql.com/doc/refman/8.0/en/row-subqueries.html 
5. **Microsoft Learn — Use Scalar or Multi-Valued Subqueries** — https://learn.microsoft.com/en-us/training/modules/write-subqueries/3-scalar-multi-values-subqueries 
6. **SQL Enlight — SA0251: Subquery Used in Expression Not Ensured to Return a Single Value** — https://docs.sqlenlight.com/sa0251/ 
7. **Google Cloud — Subqueries in PostgreSQL** — https://docs.cloud.google.com/spanner/docs/reference/postgresql/subqueries 
8. **Airbyte — PostgreSQL Subquery: A Complete Guide** — https://airbyte.com/data-engineering-resources/postgresql-subquery 
9. **Amazon Aurora PostgreSQL — Optimizing Correlated Subqueries** — https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/apg-correlated-subquery.html 
10. **SQL:2016 Standard — ISO/IEC 9075-2:2016 (Foundation)** — https://www.iso.org/standard/63555.html

---

This cheat sheet provides a comprehensive reference for SQL scalar subqueries, covering single-value, single-row, single-column, multiple-row, and multiple-column forms, error handling for cardinality violations and `NULL` results, and performance considerations including correlated subquery optimisation and join alternatives. Always verify the cardinality of a scalar subquery before deploying it to production, and consult your specific RDBMS documentation for version-specific behaviour and optimiser transformations.