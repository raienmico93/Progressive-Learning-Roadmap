# SQL Correlated Subqueries: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A correlated subquery is a subquery that references one or more columns from the outer query, causing it to be logically evaluated once for each row processed by the outer query.

**Technical Definition**
In SQL, a correlated subquery is a `SELECT` statement nested inside another statement that depends on the outer query for its values—it cannot be executed independently. The subquery contains an outer reference (a column from a table in the outer query), which creates a correlation between the inner and outer query blocks. PostgreSQL documentation describes correlated subqueries as queries that reference columns of the current outer query level, and notes that they can become performance traps because they may introduce nested-loop execution. SQL Server documentation states that correlated subqueries reference a column in the outer query and execute once per row. MySQL documentation distinguishes correlated subqueries (also called repeating subqueries) as those that depend on the outer query and are re-evaluated for each row, in contrast to non-correlated subqueries, which are evaluated only once.

**Beginner-Friendly Explanation**
A correlated subquery is a subquery that "looks at" the current row of the outer query. Instead of running once and producing a single result that the whole outer query uses, it runs again for every row the outer query processes—like asking a fresh question about each row. For example: "For each customer, find their most recent order" requires the subquery to know which customer it is currently processing.

---

### Key Characteristics

1. **Dependency on the Outer Query**: A correlated subquery contains at least one outer reference—a column from a table in the outer query—which makes it dependent on the outer query's current row.
2. **Row-by-Row Logical Evaluation**: Conceptually, the subquery is evaluated once per outer row. The database may optimise this into a join or semi-join, but the logical semantics are per-row.
3. **Cannot Execute Independently**: A correlated subquery cannot be run as a standalone query; it requires the context of the outer query.
4. **Nested-Loop Execution Risk**: Correlated subqueries often translate into nested-loop execution, which can be slow on large tables. PostgreSQL documentation explicitly warns that correlated subqueries can be performance traps for this reason.
5. **Optimiser Transformations**: Many RDBMS can rewrite correlated subqueries into joins or semi-joins (e.g., Aurora PostgreSQL's `apg_enable_correlated_scalar_transform`), but these transformations are not guaranteed.
6. **`NULL` Sensitivity**: Correlated `NOT IN` subqueries inherit the `NOT IN` `NULL` trap; `NOT EXISTS` is `NULL`-safe and generally preferred for anti-join logic.

---

### Prerequisites

- **Subquery fundamentals**: Scalar, row, table, correlated, and non-correlated subqueries.
- **Table aliases**: Required to disambiguate outer and inner table references.
- **`NULL` semantics**: Three-valued logic and how `NULL` propagates through predicates.
- **Execution plans**: Reading `EXPLAIN` / `EXPLAIN ANALYZE` output to detect nested loops.
- **Basic join concepts**: `INNER JOIN`, `LEFT JOIN`, and semi-join/anti-join patterns.

---

### Related Programming Areas

| Area | Connection |
|------|------------|
| **Existence Checks** | `EXISTS`/`NOT EXISTS` are the standard correlated subquery patterns. |
| **Per-Row Lookups** | Correlated scalar subqueries retrieve a single value per outer row. |
| **Query Optimisation** | Correlated subqueries often translate to nested loops; rewriting as joins can dramatically improve performance. |
| **Data Quality** | Correlated `NOT IN` is a common source of incorrect results due to `NULL`s. |
| **Reporting** | Correlated subqueries implement "latest per group" and "top-N per group" logic. |

---

### Core Concepts / Features

1. **Correlated Subqueries**
2. **Outer-Query References**
3. **Outer References**
4. **Row-by-Row Logical Evaluation**
5. **Execution Model**
6. **Performance Implications**
7. **Rewriting as Joins**
8. **Appropriate Use Cases**
9. **`EXISTS` / `NOT EXISTS`**
10. **`IN` / `NOT IN`**
11. **`ANY` / `ALL`**

---

## 1. Correlated Subqueries

### Definitions

**Core Definition**
A correlated subquery is a subquery that references columns from the outer query, making its result dependent on the outer query's current row.

**Technical Definition**
A correlated subquery is a nested `SELECT` that contains at least one outer reference—a column belonging to a table in the outer query block. The subquery cannot be evaluated independently; it is logically evaluated once for each row of the outer query. The correlation is expressed through table aliases that allow the subquery to refer to the outer table's columns. The SQL standard permits correlated subqueries in `WHERE`, `HAVING`, `SELECT`, and `FROM` (with `LATERAL`) clauses.

**Beginner-Friendly Explanation**
A correlated subquery is a subquery that "knows about" the outer query's current row. It is like a per-row assistant: for each row the outer query processes, the subquery runs again with that row's values.

---

### Purposes

- To perform per-row existence checks (e.g., "does this customer have any orders?").
- To retrieve the latest or maximum value per group (e.g., most recent order per customer).
- To compare each row to a group-level aggregate without using window functions.
- To implement complex anti-join and semi-join logic.

---

### Syntax Rules and Structure

**General Syntax**
```
SELECT ...
FROM outer_table o
WHERE column operator (
  SELECT ... FROM inner_table i WHERE i.col = o.col
);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `outer_table o` | The outer query's table with alias `o`. |
| `inner_table i` | The subquery's table with alias `i`. |
| `i.col = o.col` | The correlation condition (outer reference). |

**Syntax Rules**

1. The subquery must reference at least one column from the outer query to be correlated.
2. Table aliases are required to disambiguate outer and inner references.
3. The subquery is logically evaluated once per outer row.
4. Correlated subqueries can appear in `WHERE`, `HAVING`, `SELECT`, and (with `LATERAL`) `FROM`.

**Constraints and Limitations**

- Correlated subqueries cannot be executed independently.
- They often translate to nested-loop execution, which can be slow on large tables.
- Some optimisers cannot transform complex correlated subqueries into joins.

---

### Annotated Code Examples

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

**Step-by-Step Explanation**

1. The outer query iterates over each row of `SalesOrderHeader` (alias `o1`).
2. For each row, the subquery finds the maximum `SalesOrderID` for that customer (`o2.CustomerID = o1.CustomerID`).
3. The outer query keeps only the row whose `SalesOrderID` equals that maximum—the most recent order per customer.

**Expected Output (illustrative)**:

| SalesOrderID | CustomerID | OrderDate  |
|--------------|------------|------------|
| 71782        | 29485      | 2008-03-11 |
| 71783        | 29486      | 2008-04-01 |

**Why This Output Occurs**: The correlated subquery computes the maximum order ID per customer; the outer query returns only that most recent order.

---

### Real-World Cases

- **Latest record per entity**: Most recent order per customer, latest login per user.
- **Per-row comparisons**: Employees earning more than their department average.
- **Conditional filtering**: Products with sales above the category average.

---

## 2. Outer-Query References

### Definitions

**Core Definition**
An outer-query reference is a reference within a subquery to a column of a table in the outer query, establishing the correlation.

**Technical Definition**
An outer-query reference (also called an outer reference) is a column reference in a subquery that resolves to a table in an enclosing query block rather than to a table in the subquery's own `FROM` clause. The SQL standard defines the scope rules: a column reference resolves to the innermost query block that contains a matching column; if no match is found, it resolves to the next outer block, and so on. Outer references are what make a subquery correlated.

**Beginner-Friendly Explanation**
An outer-query reference is the "link" between the subquery and the outer query. It is the specific column in the outer query that the subquery uses to do its per-row work.

---

### Purposes

- To establish the correlation between inner and outer query blocks.
- To allow the subquery to compute a per-row result.
- To enable per-row filtering and lookups.

---

### Syntax Rules and Structure

```
WHERE inner_table.col = outer_table.col
```

**Syntax Rules**

1. The outer reference must be resolvable through the alias of a table in the outer query.
2. The inner table must not have a column with the same name that would shadow the outer reference (aliases prevent ambiguity).
3. Outer references can appear in `WHERE`, `HAVING`, `SELECT`, and `ON` clauses of the subquery.

**Constraints and Limitations**

- Ambiguous column names can cause the reference to resolve to the wrong query block.
- Outer references in deeply nested subqueries can be hard to read and optimise.

---

### Annotated Code Examples

**Example 1: Outer Reference in a Correlated Subquery**

```sql
SELECT c.customer_name
FROM customers c
WHERE EXISTS (
  SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

**Explanation**: `c.customer_id` is the outer reference; it links the subquery (`orders o`) to the outer query (`customers c`).

---

## 3. Outer References

### Definitions

**Core Definition**
Outer references are the set of all references in a subquery to columns of tables in enclosing query blocks.

**Technical Definition**
Outer references are the mechanism by which a subquery accesses data from its enclosing query. They are resolved at query compilation time using the SQL scoping rules: the innermost query block is searched first, then enclosing blocks. In correlated subqueries, outer references are the only way the subquery can "see" the outer row.

**Beginner-Friendly Explanation**
Outer references are all the "outside" columns that a subquery uses. They are what make the subquery depend on the outer query.

---

### Purposes (of Understanding Outer References)

- To identify whether a subquery is correlated or non-correlated.
- To reason about the subquery's evaluation frequency.
- To rewrite correlated subqueries as joins when appropriate.
- To debug ambiguous column resolution.

---

### Syntax Rules and Structure

The same as Section 2; the distinction is conceptual (a single outer reference vs. the set of all outer references).

**Constraints and Limitations**

- More outer references generally mean more complex correlation and potentially worse performance.
- Outer references to columns not in the `GROUP BY` of the outer query can cause errors.

---

### Annotated Code Examples

**Example 1: Multiple Outer References**

```sql
SELECT e.employee_id, e.salary
FROM employees e
WHERE e.salary > (
  SELECT AVG(e2.salary)
  FROM employees e2
  WHERE e2.department_id = e.department_id
    AND e2.manager_id = e.manager_id
);
```

**Explanation**: The subquery references both `e.department_id` and `e.manager_id` from the outer query, computing the average salary for employees in the same department and under the same manager.

---

## 4. Row-by-Row Logical Evaluation

### Definitions

**Core Definition**
Row-by-row logical evaluation means the correlated subquery is conceptually evaluated once for each row of the outer query, using that row's values.

**Technical Definition**
The SQL standard defines the semantics of correlated subqueries as if the subquery were evaluated once per outer row. This is the **logical** evaluation model. The **physical** execution may differ—the optimiser may rewrite the subquery as a join, use a hash semi-join, or cache results—but the result must be equivalent to the row-by-row model.

**Beginner-Friendly Explanation**
Logically, the database runs the subquery again for every row of the outer query. Physically, it might be smarter than that and run it once—but the result is the same as if it had run it row by row.

---

### Purposes (of Understanding the Model)

- To predict query results correctly.
- To reason about performance (the logical model suggests N executions; the physical model may be better or worse).
- To identify when rewriting is necessary.

---

### Syntax Rules and Structure

The logical model is not a syntax construct; it is a semantic definition.

**Constraints and Limitations**

- The logical model can be misleading for performance: a query that looks like N executions may be optimised into one join.
- Conversely, a query that looks simple may actually execute the subquery N times if the optimiser cannot rewrite it.

---

### Annotated Code Examples

**Example 1: Logical Row-by-Row Evaluation**

```sql
SELECT o.order_id,
       (SELECT SUM(oi.amount) FROM order_items oi WHERE oi.order_id = o.order_id) AS order_total
FROM orders o;
```

**Logical Evaluation**: For each order row, the subquery sums the order items for that order. If there are 10,000 orders, the subquery logically runs 10,000 times.

**Physical Execution**: The optimiser may rewrite this as a `LEFT JOIN` with `GROUP BY`, running once. The result is identical.

---

## 5. Execution Model

### Definitions

**Core Definition**
The execution model of correlated subqueries describes how the database physically evaluates them—typically as nested loops, though optimiser transformations may rewrite them as joins.

**Technical Definition**
The naive execution model for a correlated subquery is a nested-loop join: for each row of the outer table, execute the subquery (often using an index on the correlation column) and evaluate the predicate. Modern optimisers can transform correlated subqueries into semi-joins, anti-joins, or regular joins, eliminating the per-row execution. PostgreSQL documentation notes that the planner may treat a scalar subquery as a "black box," leading to poor selectivity estimates; Aurora PostgreSQL can transform correlated scalar subqueries into outer joins when `apg_enable_correlated_scalar_transform` is enabled.

**Beginner-Friendly Explanation**
The execution model is how the database actually runs the query. It might do a nested loop (slow for large tables) or it might be smart and rewrite it as a join (fast). The query text stays the same; the execution plan changes.

---

### Purposes (of Understanding Execution)

- To interpret execution plans and identify performance bottlenecks.
- To decide when to rewrite a correlated subquery as a join.
- To configure optimiser transformations (e.g., Aurora PostgreSQL settings).

---

### Syntax Rules and Structure

The execution model is not a syntax construct; it is determined by the optimiser.

**Constraints and Limitations**

- Nested-loop execution is O(N×M) in the worst case.
- Optimiser transformations are not guaranteed.
- SQL Server's assert operator (for scalar subquery single-row validation) can force nested loops.

---

### Annotated Code Examples

**Example 1: Nested-Loop Execution Plan**

```sql
EXPLAIN ANALYZE
SELECT c.customer_name
FROM customers c
WHERE EXISTS (
  SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

**Execution Plan (illustrative)**: A `Nested Loop Semi Join` with an `Index Scan` on `orders.customer_id`. For each customer, the index is used to check for orders—efficient when the index exists.

**Without Index**: A `Seq Scan` on `orders` for each customer—catastrophically slow on large tables.

---

## 6. Performance Implications

### Definitions

**Core Definition**
Performance implications of correlated subqueries are the costs associated with per-row evaluation, including nested-loop execution, repeated index lookups, and the optimiser's difficulty in estimating selectivity.

**Technical Definition**
Correlated subqueries can be performance traps because they often force nested-loop execution: for each outer row, the subquery is executed, potentially scanning the inner table or performing an index lookup. PostgreSQL documentation explicitly warns about this. The cost scales with the number of outer rows and the cost of each subquery execution. Optimiser transformations (semi-join, anti-join, scalar subquery to outer join) can eliminate the per-row cost, but they are not always applicable.

**Beginner-Friendly Explanation**
Correlated subqueries can be slow because they run again for every row. If the outer table has 1,000,000 rows and each subquery scans a 10,000-row table, that is 10 billion row comparisons. Indexes and optimiser rewrites can help, but sometimes the only fix is to rewrite the query.

---

### Purposes (of Understanding Performance)

- To diagnose slow queries caused by correlated subqueries.
- To decide between correlated subqueries, joins, and window functions.
- To configure indexes and optimiser settings.
- To write scalable SQL.

---

### Syntax Rules and Structure (Optimisation Guidelines)

1. **Index the correlation column**: An index on the inner table's correlation column (e.g., `orders.customer_id`) enables efficient nested-loop lookups.
2. **Prefer `EXISTS` over `IN` for existence checks**: `EXISTS` can short-circuit.
3. **Avoid `NOT IN` with `NULL`s**: Use `NOT EXISTS`.
4. **Rewrite as joins when possible**: `LEFT JOIN` + `GROUP BY`, or window functions.
5. **Check execution plans**: Look for nested loops with repeated subquery execution.
6. **Leverage optimiser transformations**: e.g., Aurora PostgreSQL `apg_enable_correlated_scalar_transform`.

**Constraints and Limitations**

- Indexes help only if the correlation column is indexed.
- Some subqueries cannot be rewritten automatically.
- Window functions may be a better alternative in many cases.

---

### Annotated Code Examples

**Example 1: Correlated Subquery vs. Join**

```sql
-- Correlated: may execute per row
SELECT o.order_id,
       (SELECT SUM(oi.amount) FROM order_items oi WHERE oi.order_id = o.order_id) AS total
FROM orders o;

-- Join: executes once
SELECT o.order_id, SUM(oi.amount) AS total
FROM orders o
LEFT JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY o.order_id;
```

**Explanation**: The join version scans both tables once and aggregates, often much faster than the correlated subquery on large data sets.

---

## 7. Rewriting as Joins

### Definitions

**Core Definition**
Rewriting a correlated subquery as a join means transforming the per-row subquery into an equivalent join operation that the optimiser can execute more efficiently.

**Technical Definition**
Many correlated subqueries have join equivalents: `EXISTS` → semi-join (`INNER JOIN` with `DISTINCT` or `WHERE ... IS NOT NULL`); `NOT EXISTS` → anti-join (`LEFT JOIN ... WHERE right.key IS NULL`); correlated scalar aggregate → `LEFT JOIN` + `GROUP BY`; "latest per group" → window function (`ROW_NUMBER()` or `DISTINCT ON`). These rewrites are semantically equivalent but often produce better execution plans.

**Beginner-Friendly Explanation**
Rewriting a correlated subquery as a join means turning "for each row, run this subquery" into "join the tables once." It is usually faster and easier for the optimiser to handle.

---

### Purposes

- To eliminate nested-loop execution.
- To enable hash and merge join algorithms.
- To improve scalability on large data sets.
- To make the query easier to optimise.

---

### Syntax Rules and Structure (Common Rewrites)

| Correlated Pattern | Join Rewrite |
|--------------------|--------------|
| `EXISTS (SELECT 1 ...)` | `INNER JOIN` + `DISTINCT` (semi-join) |
| `NOT EXISTS (SELECT 1 ...)` | `LEFT JOIN ... WHERE right.key IS NULL` (anti-join) |
| Correlated scalar aggregate | `LEFT JOIN` + `GROUP BY` |
| "Latest per group" | `ROW_NUMBER() OVER (PARTITION BY ...)` or `DISTINCT ON` |
| `IN (subquery)` | `INNER JOIN` (with `DISTINCT` if needed) |
| `NOT IN (subquery)` | `LEFT JOIN ... WHERE right.key IS NULL` (with `NOT EXISTS` safety) |

**Constraints and Limitations**

- Rewrites must preserve `NULL` semantics (especially for `NOT IN`).
- Some correlated subqueries (e.g., with complex `OR` conditions) are difficult to rewrite.
- Window functions may be the best rewrite for "latest per group."

---

### Annotated Code Examples

**Example 1: Rewriting `EXISTS` as a Semi-Join**

```sql
-- Correlated EXISTS
SELECT c.customer_name
FROM customers c
WHERE EXISTS (
  SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);

-- Rewritten as semi-join
SELECT DISTINCT c.customer_name
FROM customers c
INNER JOIN orders o ON o.customer_id = c.customer_id;
```

**Explanation**: The `INNER JOIN` + `DISTINCT` produces the same result (customers with at least one order) but can use hash or merge join.

---

**Example 2: Rewriting "Latest Per Group" as a Window Function**

```sql
-- Correlated subquery
SELECT o.order_id, o.customer_id, o.order_date
FROM orders o
WHERE o.order_id = (
  SELECT MAX(o2.order_id) FROM orders o2 WHERE o2.customer_id = o.customer_id
);

-- Window function rewrite
SELECT order_id, customer_id, order_date
FROM (
  SELECT order_id, customer_id, order_date,
         ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_id DESC) AS rn
  FROM orders
) t
WHERE rn = 1;
```

**Explanation**: The window function computes the row number per customer in one pass, avoiding per-row subquery execution.

---

## 8. Appropriate Use Cases

### Definitions

**Core Definition**
Appropriate use cases for correlated subqueries are scenarios where the per-row semantics are natural, the data volume is manageable, or the optimiser can transform the subquery efficiently.

**Technical Definition**
Correlated subqueries are appropriate when: (1) the query expresses an existence or per-row comparison naturally; (2) the outer result set is small or highly filtered; (3) an index on the correlation column makes per-row lookups efficient; (4) the subquery cannot be easily expressed as a join (e.g., complex `OR` conditions); or (5) readability is more important than maximum performance, and the data volume is modest.

**Beginner-Friendly Explanation**
Correlated subqueries are fine to use when the outer query returns few rows, when the correlation column is indexed, or when the query is naturally "for each X, check Y."

---

### Purposes (of Knowing Appropriate Use)

- To choose the right tool for the job.
- To avoid premature optimisation that hurts readability.
- To recognise when rewriting is necessary.

---

### Syntax Rules and Structure

No separate syntax; this is a judgment call.

**Constraints and Limitations**

- On large tables without indexes, correlated subqueries can be disastrously slow.
- In high-volume production queries, rewriting is usually preferred.

---

### Annotated Code Examples

**Example 1: Appropriate Correlated Subquery (Small Outer Set + Index)**

```sql
SELECT c.customer_name
FROM customers c
WHERE c.region = 'North'
  AND EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
  );
```

**Explanation**: The outer query is filtered to a small region, and `orders.customer_id` is indexed—per-row lookups are cheap.

---

## 9. `EXISTS`

### Definitions

**Core Definition**
`EXISTS` tests whether a correlated subquery returns any rows, returning `TRUE` if at least one row exists.

**Technical Definition**
`EXISTS (subquery)` returns `TRUE` if the subquery returns one or more rows, and `FALSE` otherwise. It ignores the subquery's `SELECT` list. It is most commonly used with correlated subqueries for existence checks. PostgreSQL documentation states that `EXISTS` is usually the best choice when testing for existence.

**Beginner-Friendly Explanation**
`EXISTS` asks: "Does a matching row exist?" It stops looking as soon as it finds one.

---

### Purposes

- To check for the existence of related rows.
- To implement semi-join logic.
- To filter rows based on correlated conditions.

---

### Syntax Rules and Structure

```
WHERE EXISTS (subquery)
```

**Syntax Rules**

1. The subquery may return any number of columns; the `SELECT` list is irrelevant.
2. Returns `TRUE` if the subquery returns at least one row.
3. Typically correlated.
4. Can short-circuit evaluation.

**Constraints and Limitations**

- Uncorrelated `EXISTS` is rarely useful (same result for every outer row).

---

### Annotated Code Examples

**Example 1: `EXISTS`**

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

---

## 10. `NOT EXISTS`

### Definitions

**Core Definition**
`NOT EXISTS` tests whether a correlated subquery returns zero rows, returning `TRUE` if no rows exist.

**Technical Definition**
`NOT EXISTS (subquery)` returns `TRUE` if the subquery returns zero rows, and `FALSE` otherwise. It is the `NULL`-safe way to implement anti-join logic, unlike `NOT IN`. PostgreSQL documentation states that `NOT EXISTS` is the most efficient and safest way to implement anti-join logic.

**Beginner-Friendly Explanation**
`NOT EXISTS` asks: "Is there no matching row?" It is the safe way to find rows without a match.

---

### Purposes

- To implement anti-join logic safely.
- To find rows with no related data.
- To replace `NOT IN` where `NULL`s are a concern.

---

### Syntax Rules and Structure

```
WHERE NOT EXISTS (subquery)
```

**Syntax Rules**

1. Returns `TRUE` if the subquery returns zero rows.
2. `NULL`-safe (unlike `NOT IN`).
3. Typically correlated.

**Constraints and Limitations**

- Same performance considerations as `EXISTS` (index the correlation column).

---

### Annotated Code Examples

**Example 1: `NOT EXISTS` Anti-Join**

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

**Why This Output Occurs**: `NOT EXISTS` returns `TRUE` for customers with no orders.

---

## 11. `IN`

### Definitions

**Core Definition**
`IN` tests whether a value appears in the result set of a subquery; it can be correlated when the subquery references outer columns.

**Technical Definition**
`value IN (subquery)` returns `TRUE` if `value` equals any row in the subquery result. When the subquery is correlated, `IN` behaves as a correlated membership test. `IN` can often be rewritten as a semi-join.

**Beginner-Friendly Explanation**
`IN` asks: "Is this value in the list produced by the subquery?"

---

### Purposes

- To filter rows based on membership in a set.
- To express semi-join logic.
- To compare against a dynamic list.

---

### Syntax Rules and Structure

```
WHERE value IN (subquery)
```

**Syntax Rules**

1. The subquery must return one column.
2. `IN` returns `TRUE` if the value matches any row.
3. Correlated `IN` is evaluated per outer row.

**Constraints and Limitations**

- `NOT IN` with `NULL`s returns no rows (the famous `NULL` trap).

---

### Annotated Code Examples

**Example 1: Correlated `IN`**

```sql
SELECT c.customer_name
FROM customers c
WHERE c.customer_id IN (
  SELECT o.customer_id FROM orders o WHERE o.customer_id = c.customer_id
);
```

**Explanation**: The subquery references `c.customer_id` (correlation); `IN` returns `TRUE` if the customer has an order.

---

## 12. `NOT IN`

### Definitions

**Core Definition**
`NOT IN` tests whether a value does **not** appear in the result set of a subquery.

**Technical Definition**
`value NOT IN (subquery)` returns `TRUE` if `value` does not equal any row in the subquery result. However, if the subquery returns any `NULL`, `NOT IN` evaluates to `UNKNOWN` for non-matching rows, and the query returns no rows. This is the well-known `NOT IN` `NULL` trap. `NOT EXISTS` is the safe alternative.

**Beginner-Friendly Explanation**
`NOT IN` asks: "Is this value **not** in the list?" But if the list contains a `NULL`, the answer becomes "unknown" for everything—and the query returns nothing.

---

### Purposes

- To filter rows that are not in a set.
- To express anti-join logic (with caution).

---

### Syntax Rules and Structure

```
WHERE value NOT IN (subquery)
```

**Syntax Rules**

1. Returns `TRUE` if the value matches no row in the subquery result.
2. **If the subquery returns any `NULL`, the result is `UNKNOWN` (no rows returned).**
3. Must be used with care when `NULL`s are possible.

**Constraints and Limitations**

- The `NULL` trap is the primary limitation.
- Use `NOT EXISTS` instead when `NULL`s may be present.

---

### Annotated Code Examples

**Example 1: The `NOT IN` `NULL` Trap**

```sql
-- If subquery returns any NULL, this returns zero rows
SELECT c.customer_name
FROM customers c
WHERE c.customer_id NOT IN (
  SELECT o.customer_id FROM orders o
);
```

**Safe Alternative**:
```sql
SELECT c.customer_name
FROM customers c
WHERE NOT EXISTS (
  SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

---

## 13. `ANY` / `ALL`

### Definitions

**Core Definition**
`ANY` and `ALL` compare a value to any or all rows in a subquery result, and can be correlated.

**Technical Definition**
`value operator ANY (subquery)` returns `TRUE` if the comparison is true for at least one row; `value operator ALL (subquery)` returns `TRUE` if the comparison is true for every row. When the subquery is correlated, the comparison is per outer row. `= ANY` is equivalent to `IN`; `<> ALL` is equivalent to `NOT IN` (with the same `NULL` traps).

**Beginner-Friendly Explanation**
`ANY` means "at least one"; `ALL` means "every one." They let you compare against a set of values.

---

### Purposes

- To compare a value against any or all rows in a correlated set.
- To express "greater than the minimum" (`> ANY`) or "greater than the maximum" (`> ALL`).
- To implement set-wide comparisons.

---

### Syntax Rules and Structure

```
WHERE value operator ANY (subquery)
WHERE value operator ALL (subquery)
```

**Syntax Rules**

1. `= ANY` ≡ `IN`; `<> ALL` ≡ `NOT IN`.
2. `> ANY` means "greater than the minimum."
3. `> ALL` means "greater than the maximum."
4. `ANY`/`ALL` must be used with a comparison operator.

**Constraints and Limitations**

- `<> ANY` is usually `TRUE` for non-empty sets (rarely the intended meaning).
- `ALL` with an empty subquery returns `TRUE` (vacuous truth).
- `NOT IN` (`<> ALL`) inherits the `NULL` trap.

---

### Annotated Code Examples

**Example 1: Correlated `> ANY`**

```sql
SELECT e.employee_id, e.salary
FROM employees e
WHERE e.salary > ANY (
  SELECT e2.salary FROM employees e2 WHERE e2.department_id = e.department_id
);
```

**Explanation**: For each employee, the subquery returns the salaries of all employees in the same department. `> ANY` returns `TRUE` if the employee earns more than at least one colleague—i.e., more than the department minimum.

---

## References

1. **PostgreSQL Documentation — Subquery Expressions (EXISTS, IN, ANY, ALL)** — https://www.postgresql.org/docs/current/functions-subquery.html
2. **PostgreSQL Documentation — Scalar Subqueries and Correlated Subqueries** — https://www.postgresql.org/docs/current/sql-expressions.html#SQL-SYNTAX-SCALAR-SUBQUERIES
3. **MySQL 8.0 Reference Manual — Correlated Subqueries** — https://dev.mysql.com/doc/refman/8.0/en/correlated-subqueries.html
4. **MySQL 8.0 Reference Manual — Subqueries with EXISTS or NOT EXISTS** — https://dev.mysql.com/doc/refman/8.0/en/exists-and-not-exists-subqueries.html
5. **Microsoft Learn — Use Self-Contained or Correlated Subqueries** — https://learn.microsoft.com/en-us/training/modules/write-subqueries/4-self-contained-correlated-subqueries
6. **Amazon Aurora PostgreSQL — Optimizing Correlated Subqueries** — https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/apg-correlated-subquery.html
7. **Airbyte — PostgreSQL Subquery: A Complete Guide** — https://airbyte.com/data-engineering-resources/postgresql-subquery
8. **SQL:2016 Standard — ISO/IEC 9075-2:2016 (Foundation)** — https://www.iso.org/standard/63555.html

---

This cheat sheet provides a comprehensive reference for SQL correlated subqueries, covering outer references, row-by-row evaluation, execution models, performance implications, join rewrites, appropriate use cases, and the `EXISTS`, `NOT EXISTS`, `IN`, `NOT IN`, `ANY`, and `ALL` operators. Always index correlation columns, prefer `NOT EXISTS` over `NOT IN` when `NULL`s are possible, and consider rewriting correlated subqueries as joins or window functions for large data sets.