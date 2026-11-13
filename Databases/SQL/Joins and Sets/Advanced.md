# SQL Advanced Join Patterns: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Advanced join patterns are SQL techniques that extend basic table-to-table joins by incorporating subqueries, common table expressions (CTEs), derived tables, lateral correlations, conditional logic, non-equality predicates, and optimiser control mechanisms to solve complex relational problems.

**Technical Definition**
In SQL, advanced join patterns encompass the syntactic and semantic constructs that allow the `FROM` clause to reference not only base tables but also the results of subqueries, CTEs, and derived tables; to correlate subqueries row-by-row via `LATERAL` (PostgreSQL, MySQL 8.0.14+) or `APPLY` (SQL Server); to express join conditions using inequality and range predicates; and to influence the query optimiser’s choice of join order and algorithm through hints and configuration parameters. These patterns are implemented with significant vendor-specific syntax and semantics across PostgreSQL, SQL Server, and MySQL.

**Beginner-Friendly Explanation**
Basic joins connect tables using simple equality conditions. Advanced join patterns let you do much more: join to the *result of a query* rather than a table, let a subquery *see* the current row from the outer table (LATERAL/APPLY), join on *ranges* instead of exact matches, or *tell the optimiser* how to execute the query when it makes a poor choice on its own. These techniques are essential for writing efficient, expressive SQL when the data model or performance requirements go beyond simple joins.

---

### Key Characteristics

1. **Subquery and CTE Joining**: The `FROM` clause can reference subqueries (derived tables) and CTEs, allowing complex intermediate results to be joined like tables.
2. **Lateral Correlation**: `LATERAL` (PostgreSQL, MySQL 8.0.14+) and `APPLY` (SQL Server) allow a subquery on the right side of a join to reference columns from the left side, enabling per-row subqueries that return multiple columns and rows.
3. **Conditional Join Logic**: Join predicates can include `CASE` expressions or `OR` conditions, though these often prevent index usage and require careful performance analysis.
4. **Non-Equi and Range Joins**: Joins are not limited to equality; inequality operators and `BETWEEN` enable range-based matching, typically using nested loop algorithms.
5. **Join Elimination**: The optimiser can remove redundant joins (especially `LEFT JOIN`s) when the joined table does not affect the final result, based on primary-key/foreign-key constraints or uniqueness guarantees.
6. **Join Reordering**: Cost-based optimisers reorder joins for efficiency. PostgreSQL uses `join_collapse_limit` to control reordering; SQL Server uses `FORCE ORDER`; MySQL’s optimiser reorders based on statistics.
7. **Join Hints**: Vendor-specific hints (SQL Server `LOOP JOIN`, `HASH JOIN`, `MERGE JOIN`, `FORCE ORDER`; PostgreSQL `enable_hashjoin`/`enable_mergejoin`/`enable_nestloop` settings; MySQL `STRAIGHT_JOIN`) can override optimiser decisions, but should be used as a last resort.

---

### Prerequisites

- **Inner and outer joins**: `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL JOIN`.
- **Subqueries and derived tables**: Understanding of `SELECT` in the `FROM` clause.
- **Common table expressions (CTEs)**: `WITH` syntax.
- **Execution plans**: Reading `EXPLAIN` / `EXPLAIN ANALYZE` output.
- **Basic optimiser concepts**: Cost estimation, cardinality, and join algorithms.

---

### Related Programming Areas

| Area | Connection |
|------|------------|
| **Query Performance Tuning** | Join elimination, reordering, and hints directly affect execution plans. |
| **Data Warehousing & ETL** | Lateral joins and derived tables simplify complex transformations. |
| **Application Development** | ORMs generate derived tables and CTEs; understanding their execution helps debugging. |
| **Business Intelligence** | Range joins support banding and temporal analysis. |

---

### Core Concepts / Features

1. **Joining Subqueries**
2. **Joining CTEs**
3. **Joining Derived Tables**
4. **Joining Lateral Subqueries**
5. **`LATERAL JOIN`**
6. **`APPLY` (SQL Server)**
7. **Conditional Joins**
8. **Non-Equijoin Conditions**
9. **Range Joins**
10. **Join Elimination**
11. **Join Reordering**
12. **Join Hints**
13. **Join Filtering Strategies**

---

## 1. Joining Subqueries

### Definitions

**Core Definition**
Joining a subquery means using the result set of a `SELECT` statement (enclosed in parentheses) as one of the relations in a `FROM` clause join.

**Technical Definition**
A subquery in the `FROM` clause—often called a derived table—is evaluated first, and its result is treated as a temporary table that can be joined to other tables or subqueries. The subquery must have an alias, and its columns can be referenced in the outer query’s `SELECT`, `ON`, and `WHERE` clauses.

**Beginner-Friendly Explanation**
Instead of joining directly to a table, you can join to the *result of a query*. For example, you can first calculate total sales per customer in a subquery, then join that result to the customers table to get customer names.

### Purposes

- To join pre-aggregated or pre-filtered results without creating temporary tables.
- To simplify complex queries by breaking them into logical steps.
- To isolate expensive computations before joining.

### Syntax Rules and Structure

```
SELECT ...
FROM (SELECT ... FROM table WHERE ...) AS subquery_alias
JOIN other_table ON ...
```

**Syntax Rules**

1. The subquery must be enclosed in parentheses.
2. The subquery must have an alias.
3. Columns from the subquery are referenced using the alias.
4. The subquery cannot reference columns from other tables in the same `FROM` clause (unless using `LATERAL`).

### Annotated Code Examples

**Example 1: Joining a Pre-Aggregated Subquery**

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

---

## 2. Joining CTEs

### Definitions

**Core Definition**
Joining a CTE means using a named common table expression (defined with `WITH`) as a relation in a join.

**Technical Definition**
A CTE is a temporary named result set defined before the main query. It can be referenced multiple times in the main query, including in joins. CTEs improve readability and allow recursive queries.

**Beginner-Friendly Explanation**
A CTE is like a named temporary table that you define at the top of your query. You can then join to it just like a regular table—and even reference it more than once.

### Purposes

- To improve readability by naming intermediate results.
- To reference the same intermediate result multiple times without repeating the subquery.
- To support recursive queries (`WITH RECURSIVE`).

### Syntax Rules and Structure

```
WITH cte_name AS (
  SELECT ...
)
SELECT ...
FROM cte_name
JOIN other_table ON ...
```

**Syntax Rules**

1. CTEs are defined before the main `SELECT`.
2. Multiple CTEs can be defined, separated by commas.
3. CTEs can reference previously defined CTEs.
4. CTEs can be joined to tables or other CTEs.

### Annotated Code Examples

**Example 1: Joining a CTE**

```sql
WITH customer_totals AS (
  SELECT customer_id, SUM(amount) AS total
  FROM orders
  GROUP BY customer_id
)
SELECT c.customer_name, ct.total
FROM customers c
JOIN customer_totals ct ON c.customer_id = ct.customer_id;
```

**Expected Output**: Same as the subquery example, but more readable.

---

## 3. Joining Derived Tables

### Definitions

**Core Definition**
A derived table is a subquery used in the `FROM` clause, producing a temporary result set that can be joined. The terms "derived table" and "subquery in FROM" are often used interchangeably.

**Technical Definition**
Derived tables are inline views: they exist only for the duration of the query. They can be joined like base tables and are particularly useful for joining aggregated or filtered results.

**Beginner-Friendly Explanation**
A derived table is the "subquery in the FROM clause" pattern. It is called "derived" because the table is derived from a query rather than stored in the database.

### Purposes

- To join aggregated results without creating permanent views.
- To simplify complex multi-step queries.
- To isolate logic for readability.

### Syntax Rules and Structure

```
SELECT ...
FROM (SELECT ...) AS derived_alias
JOIN ...
```

**Syntax Rules**

1. The derived table must have an alias.
2. It cannot reference other tables in the same `FROM` clause (unless using `LATERAL`).
3. It can be joined using any join type.

### Annotated Code Examples

**Example 1: Derived Table Join**

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

## 4. Joining Lateral Subqueries

### Definitions

**Core Definition**
A lateral subquery is a subquery in the `FROM` clause that can reference columns from tables appearing earlier in the same `FROM` clause.

**Technical Definition**
Without `LATERAL`, a subquery in `FROM` is evaluated independently and cannot see other tables in the same `FROM` clause. With `LATERAL` (PostgreSQL, MySQL 8.0.14+), the subquery is evaluated per row of the preceding table, enabling correlated joins that return multiple columns and rows.

**Beginner-Friendly Explanation**
Lateral lets a subquery "see" the current row from the table before it. This is essential for queries like "for each customer, find their top 3 orders"—the subquery needs to know which customer it is processing.

### Purposes

- To perform per-row subqueries that return multiple columns or rows.
- To implement top-N-per-group queries efficiently.
- To call set-returning functions row-by-row.

### Syntax Rules and Structure

**PostgreSQL**
```
SELECT ...
FROM table1 t1
LEFT JOIN LATERAL (
  SELECT ... FROM table2 WHERE table2.col = t1.col ...
) AS sub ON true;
```

**MySQL 8.0.14+**
```
SELECT ...
FROM table1 t1
JOIN LATERAL (
  SELECT ... FROM table2 WHERE table2.col = t1.col ...
) AS sub ON true;
```

**Syntax Rules**

1. `LATERAL` is placed before the subquery.
2. The subquery can reference columns from preceding tables.
3. `ON true` is commonly used when the join condition is inside the subquery.
4. `LEFT JOIN LATERAL ... ON true` preserves rows with no matches.

### Annotated Code Examples

**Example 1: Top-N Per Group with `LATERAL` (PostgreSQL)**

```sql
SELECT u.name, o.amount, o.order_date
FROM users u
LEFT JOIN LATERAL (
  SELECT amount, order_date
  FROM orders
  WHERE orders.user_id = u.id
  ORDER BY order_date DESC
  LIMIT 2
) o ON true;
```

**Expected Output (illustrative)**

| name  | amount | order_date |
|-------|--------|------------|
| Alice | 300    | 2026-03-15 |
| Alice | 250    | 2026-02-10 |
| Bob   | 150    | 2026-01-20 |
| Carol | NULL   | NULL       |

**Why This Output Occurs**: For each user, the lateral subquery retrieves their two most recent orders. Carol has no orders and appears with `NULL`s because of the `LEFT JOIN LATERAL` .

---

## 5. `LATERAL JOIN`

### Definitions

**Core Definition**
`LATERAL JOIN` is the explicit syntax for a join whose right-hand subquery can reference columns from the left-hand table.

**Technical Definition**
In PostgreSQL, `LATERAL` can precede any subquery in `FROM`, including those in `JOIN` clauses. It is most commonly used with `LEFT JOIN LATERAL ... ON true` to perform per-row subqueries while preserving unmatched left rows.

**Beginner-Friendly Explanation**
`LATERAL JOIN` is the "for each row, run this subquery" join. It is the PostgreSQL way of doing what SQL Server calls `APPLY`.

### Purposes

- To express correlated subqueries in the `FROM` clause.
- To replace window functions or `DISTINCT ON` for top-N-per-group queries with cleaner logic.
- To work with JSON and set-returning functions row-by-row.

### Syntax Rules and Structure

```
SELECT ...
FROM table1
[LEFT] JOIN LATERAL (subquery) AS alias ON true;
```

**Syntax Rules**

1. `LATERAL` is a keyword placed before the subquery.
2. The subquery can reference columns from `table1`.
3. `ON true` is used because the actual filtering is inside the subquery.
4. `LEFT JOIN LATERAL` preserves all left rows; `JOIN LATERAL` (inner) excludes rows with no matches.

### Annotated Code Examples

**Example 1: `LEFT JOIN LATERAL` with `LIMIT`**

```sql
SELECT u.name, o.amount
FROM users u
LEFT JOIN LATERAL (
  SELECT amount FROM orders
  WHERE orders.user_id = u.id
  ORDER BY order_date DESC
  LIMIT 1
) o ON true;
```

**Expected Output**: One row per user, with their most recent order amount (or `NULL` if no orders).

---

## 6. `APPLY` (SQL Server)

### Definitions

**Core Definition**
`APPLY` is SQL Server's operator for invoking a table-valued subquery for each row of an outer table, equivalent in purpose to `LATERAL`.

**Technical Definition**
`CROSS APPLY` is similar to an inner join: it returns only rows where the right-side expression returns at least one row. `OUTER APPLY` is similar to a left join: it preserves all left-side rows, even when the right-side expression returns no rows .

**Beginner-Friendly Explanation**
`APPLY` is SQL Server's way of saying "for each row on the left, run this subquery on the right." `CROSS APPLY` keeps only matching rows; `OUTER APPLY` keeps all left rows.

### Purposes

- To replace scalar subqueries in the `SELECT` list with table-valued expressions that can return multiple columns.
- To implement top-N-per-group queries.
- To call table-valued functions row-by-row.

### Syntax Rules and Structure

```
SELECT ...
FROM table1
{ CROSS | OUTER } APPLY (subquery) AS alias;
```

**Syntax Rules**

1. `CROSS APPLY` behaves like `INNER JOIN` on the subquery result.
2. `OUTER APPLY` behaves like `LEFT JOIN`.
3. The subquery can reference columns from `table1`.
4. Both can return multiple columns and multiple rows per left row .

### Annotated Code Examples

**Example 1: `CROSS APPLY` for Top 3 Posts Per User**

```sql
SELECT u.Id, u.DisplayName, p.Title, p.Score
FROM dbo.Users AS u
CROSS APPLY (
  SELECT p.* FROM dbo.Posts AS p
  WHERE p.OwnerUserId = u.Id
    AND p.PostTypeId = 1
  ORDER BY p.CreationDate DESC
  OFFSET 0 ROWS FETCH NEXT 3 ROWS ONLY
) AS p;
```

**Explanation**: For each user, the `CROSS APPLY` subquery retrieves their three most recent posts. Users with no posts are excluded because `CROSS APPLY` is like an inner join .

---

## 7. Conditional Joins

### Definitions

**Core Definition**
A conditional join is a join whose matching logic depends on a condition evaluated at query time, often expressed with `CASE` expressions or `OR` predicates in the `ON` clause.

**Technical Definition**
Conditional joins allow the join predicate to vary based on data values. For example, `ON t1.col1 = t2.col1 AND (t1.col2 = t2.col2 OR t2.col2 IS NULL)` joins on `col2` only when it is not `NULL` in `t2` . More complex conditional logic can use `CASE` inside the `ON` clause.

**Beginner-Friendly Explanation**
A conditional join lets you say: "Join on this column, but only if that other column has a value; otherwise just use the first column."

### Purposes

- To handle optional join columns.
- To implement polymorphic relationships.
- To avoid multiple queries for different join scenarios.

### Syntax Rules and Structure

```
ON condition1 AND (condition2 OR fallback_condition)
ON CASE WHEN ... THEN col1 ELSE col2 END = other_table.col
```

**Syntax Rules**

1. `CASE` expressions can appear in the `ON` clause.
2. `OR` conditions in `ON` can prevent index usage and force nested loop joins.
3. The fallback condition (e.g., `t2.col2 IS NULL`) controls when the optional condition is ignored.

### Annotated Code Examples

**Example 1: Conditional Join with `OR` and `IS NULL`**

```sql
SELECT *
FROM Table1 AS t1
INNER JOIN Table2 AS t2
  ON t1.col1 = t2.col1
 AND (t1.col2 = t2.col2 OR t2.col2 IS NULL);
```

**Explanation**: This joins on `col1` always, and on `col2` only when `t2.col2` is not `NULL` .

---

## 8. Non-Equijoin Conditions

### Definitions

**Core Definition**
A non-equijoin is a join whose condition uses an operator other than equality (`<`, `>`, `<=`, `>=`, `<>`, `BETWEEN`).

**Technical Definition**
Non-equijoins match rows based on inequality or range conditions. They cannot use hash or merge join algorithms efficiently and typically require nested loop joins. Oracle documentation defines a non-equijoin as an inner join whose join condition contains a non-equality operator .

**Beginner-Friendly Explanation**
A non-equijoin matches rows based on something other than "equals"—like "salary is greater than the grade minimum" or "date falls between the start and end."

### Purposes

- To match rows on ranges (temporal, numeric, or categorical).
- To implement banding, grading, or tiering logic.
- To join events to time periods.

### Syntax Rules and Structure

```
ON table1.col > table2.col
ON table1.col BETWEEN table2.min_col AND table2.max_col
ON table1.col <> table2.col
```

**Syntax Rules**

1. Any comparison operator other than `=` makes the join non-equi.
2. Non-equi conditions often prevent hash/merge join and force nested loops.
3. Indexes on the range columns can improve performance.

### Annotated Code Examples

**Example 1: Range Join (Band Join)**

```sql
SELECT e1.last_name AS employee1, e2.last_name AS employee2
FROM employees e1, employees e2
WHERE e1.salary BETWEEN e2.salary - 100 AND e2.salary + 100;
```

**Explanation**: This finds pairs of employees whose salaries are within $100 of each other. Oracle documentation calls this a "band join," a special type of non-equijoin .

---

## 9. Range Joins

### Definitions

**Core Definition**
A range join is a non-equijoin that matches rows where a value in one table falls within a range defined in another table.

**Technical Definition**
Range joins use `BETWEEN`, `>=`/`<=`, or similar operators to match a point value (e.g., a date, salary, or measurement) to an interval (e.g., a period, grade band, or tolerance range).

**Beginner-Friendly Explanation**
A range join answers "which bucket does this value fall into?"—like matching a temperature reading to a temperature band.

### Purposes

- To match transactions to accounting periods.
- To assign grades based on score ranges.
- To join sensor readings to calibration ranges.

### Syntax Rules and Structure

```
ON point_table.value BETWEEN range_table.min_value AND range_table.max_value
```

**Syntax Rules**

1. The range table should have `min` and `max` columns (or `start`/`end`).
2. `BETWEEN` is inclusive of both endpoints.
3. Indexes on `min_value` and `max_value` can help, but range joins often use nested loops.

### Annotated Code Examples

**Example 1: Salary Band Join**

```sql
SELECT e.name, g.grade
FROM employees e
JOIN salary_grades g
  ON e.salary BETWEEN g.min_salary AND g.max_salary;
```

**Expected Output (illustrative)**

| name  | grade |
|-------|-------|
| Alice | A     |
| Bob   | B     |

---

## 10. Join Elimination

### Definitions

**Core Definition**
Join elimination is an optimiser transformation that removes a join when the joined table does not affect the final result.

**Technical Definition**
Join elimination (also called join removal) is a semantic query-rewriting optimisation. A `LEFT JOIN` can be removed when: (1) the right-hand table is not referenced in `SELECT`, `WHERE`, `GROUP BY`, `HAVING`, or `ORDER BY`; (2) the join condition covers a unique key on the right-hand table, guaranteeing at most one match per left row; and (3) the join is a `LEFT JOIN` (which never filters left rows) .

**Beginner-Friendly Explanation**
Join elimination is the database saying: "You joined this table, but you didn't actually use anything from it, and the join can't change the number of rows. So I'm going to skip it and make your query faster."

### Purposes

- To reduce unnecessary table scans and join computations.
- To improve performance of ORM-generated queries with redundant joins.
- To simplify execution plans.

### Syntax Rules and Structure (Conditions for Elimination)

| Condition | Requirement |
|-----------|-------------|
| Join type | `LEFT JOIN` (inner joins are not eliminated if they could filter rows) |
| Right table referenced? | No columns from right table in `SELECT`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY` |
| Uniqueness | Join condition covers a unique key on the right table |
| Grouping/Distinct | If the right table only amplifies rows and a later `GROUP BY`/`DISTINCT` removes duplicates, elimination is possible |

**Constraints and Limitations**

- Elimination is **optimiser-dependent**; not all databases implement it.
- PostgreSQL, MySQL (TaurusDB), and Alibaba Cloud support left join elimination in various forms .
- Elimination is safe only when the optimiser can prove the join does not affect the result.

### Annotated Code Examples

**Example 1: Join Elimination in Action**

```sql
-- t2.a is a unique key; no columns from t2 are used
SELECT t1.a
FROM t1
LEFT JOIN t2 ON t2.a = t1.b;
```

**Rewritten Query (after elimination)**:
```sql
SELECT t1.a FROM t1;
```

**Explanation**: Because `t2.a` is unique and no `t2` columns are referenced, each `t1` row matches at most one `t2` row, and the `LEFT JOIN` cannot filter or multiply rows. The optimiser removes the join .

---

## 11. Join Reordering

### Definitions

**Core Definition**
Join reordering is the optimiser's process of changing the sequence in which tables are joined to produce a more efficient execution plan.

**Technical Definition**
For multi-table queries, the order in which tables are joined can dramatically affect intermediate result sizes. Cost-based optimisers (PostgreSQL, SQL Server, MySQL) explore alternative join orders and select the one with the lowest estimated cost. The written order in the `FROM` clause does not necessarily determine the execution order.

**Beginner-Friendly Explanation**
Join reordering is the database saying: "You wrote the joins in this order, but I think a different order will be faster, so I'll rearrange them."

### Purposes

- To minimise intermediate result sizes.
- To reduce I/O and CPU cost.
- To exploit indexes and sorted data.

### Syntax Rules and Structure (Controlling Reordering)

| Database | Mechanism | Effect |
|----------|-----------|--------|
| PostgreSQL | `join_collapse_limit = 1` | Forces the planner to use the written join order  |
| SQL Server | `OPTION (FORCE ORDER)` | Preserves the join order specified in the query  |
| MySQL | `STRAIGHT_JOIN` | Forces the left-to-right join order (use with caution) |

**Constraints and Limitations**

- Forcing join order can prevent the optimiser from finding a better plan.
- Exhaustive join-order search is impractical for many tables; optimisers use heuristics or genetic algorithms.

### Annotated Code Examples

**Example 1: PostgreSQL `join_collapse_limit`**

```sql
SET join_collapse_limit = 1;
SELECT ...
FROM small_table s
JOIN large_table l ON s.id = l.s_id
JOIN other_table o ON l.id = o.l_id;
```

**Explanation**: With `join_collapse_limit = 1`, PostgreSQL joins `s` to `l` first, then to `o`, following the written order. Without this setting, the planner may reorder .

---

## 12. Join Hints

### Definitions

**Core Definition**
Join hints are vendor-specific directives that instruct the query optimiser to use a particular join algorithm or join order.

**Technical Definition**
SQL Server provides join hints (`LOOP JOIN`, `HASH JOIN`, `MERGE JOIN`, `REMOTE`) specified per join, and query hints (`FORCE ORDER`, `MAXDOP`, `RECOMPILE`) specified in the `OPTION` clause . PostgreSQL does not support per-join hints; instead, it provides planner configuration parameters (`enable_hashjoin`, `enable_mergejoin`, `enable_nestloop`) that can be toggled at the session or query level .

**Beginner-Friendly Explanation**
A join hint is you telling the database: "I know you think you have a better plan, but use *this* algorithm or *this* order." It is generally a last resort for experienced developers.

### Purposes

- To override a poor optimiser choice caused by bad statistics or complex predicates.
- To test alternative plans during performance tuning.
- To stabilise performance in edge cases.

### Syntax Rules and Structure

**SQL Server Join Hint**
```
FROM table1
{ LOOP | HASH | MERGE } JOIN table2 ON ...
```

**SQL Server Query Hint**
```
OPTION (FORCE ORDER)
OPTION (HASH JOIN)
OPTION (MAXDOP 4)
```

**PostgreSQL Planner Settings**
```
SET enable_hashjoin = off;
SET enable_mergejoin = off;
```

**Constraints and Limitations**

- Hints are **not portable** between databases.
- Incorrect hints can make performance worse.
- SQL Server documentation recommends using hints only as a last resort .

### Annotated Code Examples

**Example 1: SQL Server `HASH JOIN` Hint**

```sql
SELECT *
FROM table1 t1
INNER HASH JOIN table2 t2 ON t1.id = t2.id;
```

**Explanation**: Forces a hash join between the two tables .

**Example 2: PostgreSQL Disabling Hash Join**

```sql
SET enable_hashjoin = off;
SELECT ...
FROM table1 t1
JOIN table2 t2 ON t1.id = t2.id;
```

**Explanation**: Prevents the planner from considering hash join plans, forcing nested loop or merge join .

---

## 13. Join Filtering Strategies

### Definitions

**Core Definition**
Join filtering strategies are techniques for applying selective predicates as early as possible in the query plan to reduce the number of rows participating in joins.

**Technical Definition**
Predicate pushdown is the optimiser transformation that moves filter conditions from outer query blocks into inner subqueries or base table scans, reducing intermediate result sizes. In join contexts, filters on individual tables should be applied before the join (in `WHERE` or in the derived table), not after (in `HAVING` or outer `WHERE`), whenever logically equivalent .

**Beginner-Friendly Explanation**
Join filtering is about doing the "cheap" filtering first: if you only need 10% of the rows from a table, filter them out before joining, not after. This makes the join process far fewer rows.

### Purposes

- To reduce the size of intermediate results.
- To enable index usage on filtered columns.
- To avoid unnecessary computation in joins and aggregations.

### Syntax Rules and Structure (Strategies)

1. **Filter in `WHERE`, not `HAVING`**: Non-aggregate conditions belong in `WHERE`.
2. **Push filters into derived tables**: `FROM (SELECT ... WHERE filter) AS dt` reduces the derived table size.
3. **Use `LATERAL` for per-row filtering**: When the filter depends on the outer row.
4. **Apply selective predicates before joins**: Put the most selective conditions on the tables that are scanned first.

**Constraints and Limitations**

- Not all filters can be pushed down (e.g., conditions involving aggregates or window functions).
- Some optimisers implement predicate pushdown automatically; others need explicit query structuring.

### Annotated Code Examples

**Example 1: Filtering Before Joining**

```sql
-- Good: filter in WHERE before grouping/joining
SELECT c.customer_name, o.amount
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date >= '2026-01-01'
  AND c.status = 'active';
```

**Explanation**: The `WHERE` conditions reduce the rows from both tables before the join, improving performance.

---

## References

1. **PostgreSQL 16 Documentation — Table Expressions (LATERAL)** — https://www.postgresql.org/docs/16/queries-table-expressions.html
2. **MySQL 8.0 Reference Manual — Lateral Derived Tables** — https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/lateral-derived-tables.html#1
3. **Microsoft SQL Server — Query Hints (Transact-SQL)** — https://github.com/microsoftdocs/sql-docs/blob/4c7374e97347c3723cd42ea99fb540a4d09e1ae6/docs/t-sql/queries/hints-transact-sql-query.md
4. **Microsoft SQL Server — Joins (SQL Server)** — https://learn.microsoft.com/en-us/sql/relational-databases/performance/joins
5. **PostgreSQL 16 Documentation — Planner Method Configuration (enable_hashjoin)** — https://www.postgresql.org/docs/current/runtime-config-query.html
6. **Oracle SQL Tuning Guide — Joins (Non-Equijoins and Band Joins)** — https://docs.oracle.com/en/database/oracle/oracle-database/26/tgsql/joins.html
7. **Huawei Cloud — Left Join Elimination (TaurusDB)** — https://support.huaweicloud.com/intl/pt-br/kerneldesc-taurusdb/taurusdb_20_0082.html#1
8. **Erik Darling — Cross Apply and Outer Apply Cheat Sheet** — https://erikdarling.com/the-how-to-write-sql-server-queries-correctly-cheat-sheet-cross-apply-and-outer-apply/
9. **Cybrosys — How to Use LATERAL JOINs in PostgreSQL** — https://www.cybrosys.com/research-and-development/postgres/how-to-use-lateral-joins-in-postgresql-for-powerful-queries
10. **Microsoft Learn — Conditional Join (Q&A)** — https://learn.microsoft.com/en-za/answers/questions/1689590/conditional-join
11. **PostgreSQL Documentation — join_collapse_limit** — https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-JOIN-COLLAPSE-LIMIT
12. **SQL:2016 Standard — ISO/IEC 9075-2:2016 (Foundation)** — https://www.iso.org/standard/63555.html

---

This cheat sheet provides a comprehensive reference for advanced SQL join patterns across PostgreSQL, MySQL, and SQL Server, covering subquery and CTE joins, lateral joins and `APPLY`, conditional and non-equi joins, range joins, join elimination and reordering, hints, and filtering strategies. Always consult your specific RDBMS documentation for version-specific syntax and optimiser behaviour.