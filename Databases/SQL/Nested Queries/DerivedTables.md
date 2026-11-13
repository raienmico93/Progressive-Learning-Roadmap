# SQL Derived Tables: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A derived table is a subquery in the `FROM` clause of a SQL statement that produces a temporary result set treated as a table for the duration of the query.

**Technical Definition**
In SQL, a derived table (also called an inline view or a subquery in `FROM`) is a `SELECT` statement enclosed in parentheses that appears in the `FROM` clause of an outer `SELECT`, `INSERT`, `UPDATE`, or `DELETE` statement. It must be assigned an alias (a correlation name) in most RDBMS, and its columns can be referenced by the outer query just as if they were columns of a base table. Derived tables are logical constructs: they do not have persistent storage, and their evaluation may be merged into the outer query by the optimiser or materialised as a separate step depending on the query structure, RDBMS, and optimiser decisions. Derived tables are closely related to CTEs (which are named, reusable derived tables) and to lateral subqueries (which can reference preceding tables in the same `FROM` clause).

**Beginner-Friendly Explanation**
A derived table is a query result that you treat as a table. Instead of joining to a physical table, you write a subquery in the `FROM` clause, give it an alias (a name), and then use that alias as if it were a table. It is like creating a temporary table on the fly, just for that one query. For example, you can first calculate each customer's total spending in a derived table, then join that result to the customers table to get customer names.

---

### Key Characteristics

1. **Temporary and Query-Scoped**: Derived tables exist only for the duration of the query. They are not stored in the database and cannot be referenced outside the query.
2. **Alias Requirement**: Most RDBMS (MySQL, PostgreSQL, SQL Server) require derived tables to have an alias. MySQL documentation states that every derived table must have its own alias.
3. **No Correlation Without `LATERAL`**: A standard derived table cannot reference columns from other tables in the same `FROM` clause. `LATERAL` (PostgreSQL, MySQL 8.0.14+) or `APPLY` (SQL Server) removes this restriction.
4. **Optimiser Merging vs. Materialisation**: The optimiser may "merge" a derived table into the outer query (inlining it) or "materialise" it as a separate result set. MySQL documentation describes the `MERGE` and `TEMPTABLE` algorithms for derived table and view processing.
5. **Nesting and Multi-Stage Construction**: Derived tables can be nested, and multiple derived tables can be joined, enabling multi-stage query construction in a single statement.
6. **Close Relationship to CTEs**: CTEs (`WITH` clause) are named derived tables defined before the main query; derived tables are inline and anonymous (except for their alias). CTEs improve readability and reuse; derived tables are more local.

---

### Prerequisites

- **Basic `SELECT` syntax**: `SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`.
- **Table aliases**: How to assign and reference aliases.
- **Subquery fundamentals**: Scalar, row, table, correlated, and non-correlated subqueries.
- **Join syntax**: `INNER JOIN`, `LEFT JOIN`.
- **Execution plan basics**: To reason about materialisation and performance.

---

### Related Programming Areas

| Area | Connection |
|------|------------|
| **Query Composition** | Derived tables break complex logic into modular steps. |
| **Pre-Aggregation** | Aggregating in a derived table before joining to detail tables. |
| **ETL and Reporting** | Multi-stage transformations expressed in a single SQL statement. |
| **Query Performance** | Materialisation vs. merging decisions affect execution time and memory. |
| **CTE Alternatives** | Derived tables are the inline alternative to CTEs. |

---

### Core Concepts / Features

1. **Inline Views**
2. **Derived Tables**
3. **Subqueries in `FROM`**
4. **Named Derived Tables**
5. **Aliasing**
6. **Nesting**
7. **Multi-Stage Query Construction**
8. **Performance**
9. **Materialisation**

---

## 1. Inline Views

### Definitions

**Core Definition**
An inline view is a subquery in the `FROM` clause that acts as a view defined only for the duration of the query.

**Technical Definition**
The term "inline view" is most commonly associated with Oracle, but the concept is universal: a `SELECT` statement in the `FROM` clause that produces a virtual table. It is "inline" because the view definition is embedded in the query rather than stored as a named database object. It is a "view" because it presents a tabular result set to the outer query, just as a stored view would.

**Beginner-Friendly Explanation**
An inline view is a view that exists only inside one query. It is not saved in the database—it is created, used, and discarded all within the same statement.

---

### Purposes

- To isolate a complex subquery result and treat it as a table.
- To pre-aggregate or pre-filter data before the outer query uses it.
- To avoid creating permanent views for one-off queries.
- To improve readability by naming intermediate results.

---

### Syntax Rules and Structure

**General Syntax**
```
SELECT ...
FROM (SELECT ... FROM table WHERE ...) AS inline_view_alias
[JOIN ...];
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `(` and `)` | Parentheses enclosing the subquery. |
| `SELECT ...` | The inline view definition. |
| `AS inline_view_alias` | Required alias (name) for the inline view. |

**Syntax Rules**

1. The subquery must be enclosed in parentheses.
2. The alias is required in MySQL, PostgreSQL, and SQL Server.
3. The outer query can reference the inline view's columns using the alias.
4. The inline view cannot reference other tables in the same `FROM` clause (unless using `LATERAL`/`APPLY`).

**Constraints and Limitations**

- Inline views cannot be indexed.
- The optimiser may merge or materialise the inline view; the choice affects performance.
- Nested inline views can become hard to read.

---

### Annotated Code Examples

**Example 1: Inline View for Pre-Aggregation**

```sql
SELECT c.customer_name, totals.total_spent
FROM customers c
JOIN (
  SELECT customer_id, SUM(amount) AS total_spent
  FROM orders
  GROUP BY customer_id
) AS totals ON c.customer_id = totals.customer_id;
```

**Step-by-Step Explanation**

1. The inline view `totals` aggregates orders per customer.
2. The outer query joins `customers` to `totals` on `customer_id`.
3. The result is each customer's name and total spending.

**Expected Output (illustrative)**:

| customer_name | total_spent |
|---------------|-------------|
| Alice         | 550         |
| Bob           | 150         |

---

### Real-World Cases

- **Sales reporting**: Pre-aggregating daily sales before joining to product details.
- **Data quality**: Filtering invalid records in an inline view before joining.
- **Multi-step transformations**: Building intermediate results in a single query.

---

## 2. Derived Tables

### Definitions

**Core Definition**
A derived table is a subquery in the `FROM` clause that produces a result set used as a table by the outer query. The term is synonymous with "inline view" and "subquery in `FROM`."

**Technical Definition**
MySQL documentation defines a derived table as a subquery in the `FROM` clause that acts as a temporary result set. PostgreSQL documentation calls it a "subquery in `FROM`" and requires an alias. The SQL standard refers to it as a "derived table." The core idea is the same across all RDBMS: a `SELECT` in `FROM` whose result is consumed as a table.

**Beginner-Friendly Explanation**
A derived table is the general name for "a query result used as a table." It is the same thing as an inline view—just a different name.

---

### Purposes

- To join to aggregated, filtered, or transformed results.
- To structure complex queries into logical stages.
- To avoid temporary tables for one-off queries.
- To improve readability and maintainability.

---

### Syntax Rules and Structure

The same as inline views: `FROM (subquery) AS alias`.

**Constraints and Limitations**

- Alias required.
- No correlation without `LATERAL`/`APPLY`.
- Materialisation may affect performance.

---

### Annotated Code Examples

**Example 1: Derived Table with Multiple Columns**

```sql
SELECT d.dept_name, avg_sal.avg_salary, avg_sal.headcount
FROM departments d
JOIN (
  SELECT department_id,
         AVG(salary) AS avg_salary,
         COUNT(*) AS headcount
  FROM employees
  GROUP BY department_id
) AS avg_sal ON d.department_id = avg_sal.department_id;
```

**Expected Output (illustrative)**:

| dept_name   | avg_salary | headcount |
|-------------|------------|-----------|
| Engineering | 85000      | 12        |
| Sales       | 62000      | 8         |

---

## 3. Subqueries in `FROM`

### Definitions

**Core Definition**
A subquery in `FROM` is a `SELECT` statement nested inside the `FROM` clause of an outer query, producing a derived table.

**Technical Definition**
The `FROM` clause accepts a table reference or a parenthesised `SELECT` statement. When it is a `SELECT`, the result is treated as a table. This is the syntactic mechanism by which derived tables are created. MySQL documentation notes that subqueries in `FROM` are permitted and must have an alias.

**Beginner-Friendly Explanation**
A subquery in `FROM` is just a query written inside the `FROM` clause. It is the raw syntax that creates a derived table.

---

### Purposes

- To embed a query result as a table in the `FROM` clause.
- To enable joins between base tables and computed results.
- To support multi-stage query construction.

---

### Syntax Rules and Structure

```
SELECT ...
FROM (SELECT ... FROM table) AS alias;
```

**Syntax Rules**

1. The subquery must be enclosed in parentheses.
2. The alias is required in MySQL, PostgreSQL, and SQL Server.
3. The subquery cannot reference outer tables (unless `LATERAL`/`APPLY`).

---

### Annotated Code Examples

**Example 1: Basic Subquery in `FROM`**

```sql
SELECT *
FROM (SELECT customer_id, SUM(amount) AS total FROM orders GROUP BY customer_id) AS order_totals
WHERE order_totals.total > 500;
```

**Explanation**: The subquery aggregates orders per customer; the outer query filters those with total > 500.

---

## 4. Named Derived Tables

### Definitions

**Core Definition**
A named derived table is a derived table that has been assigned an alias, allowing the outer query to reference it and—in some RDBMS—to reuse it within the same query.

**Technical Definition**
In standard SQL, a derived table is always assigned an alias (correlation name). The alias makes the derived table referencable in the outer query's `SELECT`, `WHERE`, `JOIN`, and `GROUP BY` clauses. CTEs (`WITH` clause) extend this concept by allowing the named derived table to be defined once and referenced multiple times.

**Beginner-Friendly Explanation**
A named derived table is simply a derived table with a name (alias). The name lets you refer to it later in the query.

---

### Purposes

- To reference derived table columns unambiguously.
- To reuse the derived table in multiple parts of the query (with CTEs).
- To improve readability.

---

### Syntax Rules and Structure

```
FROM (SELECT ...) AS alias
```

**Syntax Rules**

1. Alias is required.
2. Alias scope is the entire outer query.
3. CTEs allow reuse of the same named result multiple times.

---

### Annotated Code Examples

**Example 1: Named Derived Table Referenced Twice (via CTE)**

```sql
WITH order_totals AS (
  SELECT customer_id, SUM(amount) AS total FROM orders GROUP BY customer_id
)
SELECT c.customer_name, ot.total
FROM customers c
JOIN order_totals ot ON c.customer_id = ot.customer_id
WHERE ot.total > 500;
```

**Explanation**: The CTE `order_totals` is a named derived table that can be referenced multiple times in the main query.

---

## 5. Aliasing

### Definitions

**Core Definition**
Aliasing is the assignment of a correlation name (alias) to a derived table, allowing the outer query to reference it and its columns.

**Technical Definition**
The alias is a syntactic requirement in most RDBMS. It provides a namespace for the derived table's columns, enabling unambiguous references in the outer query. Without an alias, MySQL raises an error ("Every derived table must have its own alias"); PostgreSQL and SQL Server also require aliases.

**Beginner-Friendly Explanation**
Aliasing is giving your derived table a nickname so you can refer to it later. It is like saying "call this result `totals`."

---

### Purposes

- To satisfy the RDBMS requirement for a correlation name.
- To reference derived table columns unambiguously.
- To improve query readability.

---

### Syntax Rules and Structure

```
FROM (SELECT ...) AS alias_name
```

**Syntax Rules**

1. `AS` is optional in most RDBMS (e.g., `FROM (SELECT ...) alias` works in PostgreSQL and MySQL).
2. The alias must be unique within the `FROM` clause.
3. Column aliases within the derived table can also be assigned.

**Constraints and Limitations**

- Forgetting the alias is a common error, especially in MySQL.
- Ambiguous column names across derived tables require qualification with the alias.

---

### Annotated Code Examples

**Example 1: Alias Required (MySQL)**

```sql
-- Error: every derived table must have its own alias
SELECT * FROM (SELECT 1 AS x);

-- Correct
SELECT * FROM (SELECT 1 AS x) AS t;
```

---

## 6. Nesting

### Definitions

**Core Definition**
Nesting is the use of a derived table inside another derived table, creating multiple levels of query nesting.

**Technical Definition**
Derived tables can be nested to arbitrary depth, subject to RDBMS limits and readability constraints. Each level can reference the level below it. Nesting enables multi-stage transformations (e.g., filter → aggregate → join → filter). Deeply nested queries can be hard to read and optimise; CTEs often improve clarity.

**Beginner-Friendly Explanation**
Nesting is putting a derived table inside another derived table—like layers of an onion. Each layer builds on the one below it.

---

### Purposes

- To perform multiple stages of transformation.
- To isolate complex logic into layers.
- To work around limitations of a single `SELECT` level.

---

### Syntax Rules and Structure

```
SELECT ...
FROM (
  SELECT ...
  FROM (
    SELECT ... FROM table
  ) AS inner_alias
) AS outer_alias;
```

**Syntax Rules**

1. Each nested derived table requires its own alias.
2. Each level can reference only the levels below it.
3. Deep nesting can hurt readability and optimiser performance.

**Constraints and Limitations**

- Deep nesting is harder to read than CTEs.
- Some optimisers cannot merge deeply nested derived tables, forcing materialisation.

---

### Annotated Code Examples

**Example 1: Nested Derived Tables**

```sql
SELECT region, AVG(customer_total) AS avg_customer_spend
FROM (
  SELECT c.region, o.customer_id, SUM(o.amount) AS customer_total
  FROM customers c
  JOIN orders o ON o.customer_id = c.customer_id
  GROUP BY c.region, o.customer_id
) AS customer_totals
GROUP BY region;
```

**Explanation**: The inner derived table computes each customer's total spend; the outer query averages those totals per region.

---

## 7. Multi-Stage Query Construction

### Definitions

**Core Definition**
Multi-stage query construction is the technique of building a complex query through a sequence of derived tables, each stage transforming the result of the previous stage.

**Technical Definition**
Multi-stage construction uses derived tables (or CTEs) as pipeline stages. Each stage can filter, aggregate, join, or reshape the data. This approach is common in ETL, reporting, and analytics, where a single monolithic query would be unreadable. The optimiser may merge stages or materialise them; the choice affects performance.

**Beginner-Friendly Explanation**
Multi-stage construction is like an assembly line: each stage does one job and passes the result to the next stage.

---

### Purposes

- To break complex logic into manageable stages.
- To improve readability and maintainability.
- To enable step-by-step debugging.
- To support ETL and reporting pipelines.

---

### Syntax Rules and Structure

```
SELECT ...
FROM (
  SELECT ...
  FROM (
    SELECT ... FROM base_table WHERE ...
  ) AS stage1
  JOIN other_table ON ...
) AS stage2
WHERE ...;
```

**Syntax Rules**

1. Each stage is a derived table with an alias.
2. Stages can be joined, filtered, and aggregated.
3. CTEs are often preferable for readability.

---

### Annotated Code Examples

**Example 1: Three-Stage Pipeline**

```sql
SELECT s.region, s.total_revenue, s.customer_count
FROM (
  SELECT region,
         SUM(customer_revenue) AS total_revenue,
         COUNT(*) AS customer_count
  FROM (
    SELECT c.region, o.customer_id, SUM(o.amount) AS customer_revenue
    FROM customers c
    JOIN orders o ON o.customer_id = c.customer_id
    GROUP BY c.region, o.customer_id
  ) AS customer_totals
  GROUP BY region
) AS s
WHERE s.total_revenue > 100000;
```

**Explanation**: Stage 1 computes each customer's revenue; Stage 2 aggregates by region; Stage 3 filters regions with revenue above 100,000.

---

## 8. Performance

### Definitions

**Core Definition**
Performance of derived tables concerns the execution cost of evaluating the derived subquery and integrating its result into the outer query, including whether the optimiser merges or materialises it.

**Technical Definition**
The performance of a derived table depends on the optimiser's strategy: **merging** (inlining the derived table into the outer query, allowing predicates to be pushed down and indexes to be used) or **materialising** (evaluating the derived table separately and storing its result, which can be expensive for large results but may be beneficial when the result is reused or when merging is impossible). MySQL documentation describes the `MERGE` and `TEMPTABLE` algorithms for derived tables and views. PostgreSQL's planner can pull up subqueries (merge) or materialise them, depending on the query structure (e.g., `LIMIT`, `DISTINCT`, aggregates may prevent pull-up).

**Beginner-Friendly Explanation**
Performance of derived tables is about whether the database can "look through" the derived table and optimise the whole query together (merging), or whether it has to compute the derived table first and store it (materialising). Merging is usually faster; materialising can be slower but sometimes necessary.

---

### Purposes (of Understanding Performance)

- To diagnose slow queries involving derived tables.
- To decide between derived tables, CTEs, and temporary tables.
- To structure queries so the optimiser can merge rather than materialise.

---

### Syntax Rules and Structure (Optimisation Guidelines)

1. **Avoid `LIMIT`/`OFFSET` in derived tables if merging is desired**: MySQL may switch to materialisation when `LIMIT` is present.
2. **Use CTEs for readability, but be aware of materialisation**: Some RDBMS materialise CTEs by default (older PostgreSQL versions) or when referenced multiple times.
3. **Push filters into derived tables**: Filtering inside the derived table reduces the result size and can improve performance if materialised.
4. **Check execution plans**: Look for "Subquery Scan" or "Materialize" nodes.
5. **Index the underlying tables**: Merging allows the outer query's indexes to be used; materialisation may benefit from indexes on the derived table's result (not possible without a temp table).

**Constraints and Limitations**

- Merging is not always possible (e.g., aggregates, `DISTINCT`, `LIMIT`, window functions).
- Materialisation can consume significant memory/temp space.
- Optimiser behaviour varies by RDBMS and version.

---

### Annotated Code Examples

**Example 1: Merging vs. Materialisation in PostgreSQL**

```sql
-- Likely merged: simple derived table with no LIMIT/DISTINCT/aggregate
SELECT *
FROM (SELECT * FROM orders WHERE amount > 100) AS filtered
WHERE filtered.order_date > '2026-01-01';

-- Likely materialised: derived table with LIMIT
SELECT *
FROM (SELECT * FROM orders ORDER BY amount DESC LIMIT 100) AS top_orders
WHERE top_orders.amount > 500;
```

**Explanation**: The first query allows PostgreSQL to merge the derived table and push both filters to the base table. The second query likely materialises the `LIMIT` subquery before applying the outer filter.

---

## 9. Materialisation

### Definitions

**Core Definition**
Materialisation is the process of evaluating a derived table (or CTE) as a separate step and storing its result for use by the outer query.

**Technical Definition**
Materialisation (also called the `TEMPTABLE` algorithm in MySQL) evaluates the subquery independently, stores the result in a temporary table (in memory or on disk), and then uses that temporary table as the input to the outer query. It is the opposite of merging. Materialisation is required when merging is impossible (e.g., the derived table contains `LIMIT`, `DISTINCT`, aggregates that cannot be pushed down, or is referenced multiple times). Materialised derived tables cannot be indexed (unless explicitly materialised into a temp table with an index).

**Beginner-Friendly Explanation**
Materialisation means the database computes the derived table's result first and stores it, then runs the outer query against that stored result. It is like making a temporary copy of the data.

---

### Purposes (of Understanding Materialisation)

- To predict performance when merging is impossible.
- To decide whether to use a temporary table instead of a derived table.
- To optimise queries that must be materialised.

---

### Syntax Rules and Structure (When Materialisation Occurs)

| Trigger | Example |
|---------|---------|
| `LIMIT` / `OFFSET` | `(SELECT ... LIMIT 10)` |
| `DISTINCT` | `(SELECT DISTINCT ...)` |
| Aggregates that prevent merge | `(SELECT SUM(...) ...)` |
| Window functions | `(SELECT ROW_NUMBER() OVER (...) ...)` |
| Referenced multiple times (CTE) | `WITH cte AS (...) SELECT * FROM cte a JOIN cte b ...` |
| `UNION` | `(SELECT ... UNION SELECT ...)` |

**Constraints and Limitations**

- Materialised results cannot use indexes from the base table.
- Large materialised results can spill to disk.
- Some RDBMS allow `MATERIALIZED` / `NOT MATERIALIZED` hints (PostgreSQL 12+ for CTEs).

---

### Annotated Code Examples

**Example 1: Materialisation Due to `LIMIT`**

```sql
SELECT *
FROM (
  SELECT * FROM orders ORDER BY amount DESC LIMIT 100
) AS top_orders
WHERE top_orders.amount > 500;
```

**Explanation**: The `LIMIT` forces materialisation in most RDBMS; the outer filter is applied after the top 100 orders are materialised.

---

**Example 2: PostgreSQL `MATERIALIZED` Hint**

```sql
WITH cte AS MATERIALIZED (
  SELECT ... FROM large_table WHERE ...
)
SELECT * FROM cte;
```

**Explanation**: PostgreSQL 12+ allows explicit `MATERIALIZED` or `NOT MATERIALIZED` hints to control CTE materialisation.

---

## References

1. **PostgreSQL Documentation — Subqueries in FROM (Derived Tables)** — https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-FROM
2. **PostgreSQL Documentation — WITH Queries (CTEs and Materialization)** — https://www.postgresql.org/docs/current/queries-with.html
3. **MySQL 8.0 Reference Manual — Subqueries in the FROM Clause (Derived Tables)** — https://dev.mysql.com/doc/refman/8.0/en/derived-tables.html
4. **MySQL 8.0 Reference Manual — Optimizing Derived Tables and Views** — https://dev.mysql.com/doc/refman/8.0/en/derived-table-optimization.html
5. **Microsoft SQL Server — Derived Tables (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/queries/from-transact-sql#derived-tables
6. **Microsoft Learn — Use Derived Tables** — https://learn.microsoft.com/en-us/training/modules/query-multiple-tables-with-joins/6-derived-tables
7. **Airbyte — PostgreSQL Subquery: A Complete Guide** — https://airbyte.com/data-engineering-resources/postgresql-subquery
8. **Red-Gate — MySQL Subqueries: Derived Tables** — https://www.red-gate.com/simple-talk/databases/mysql/subqueries-in-mysql/
9. **SQL:2016 Standard — ISO/IEC 9075-2:2016 (Foundation)** — https://www.iso.org/standard/63555.html

---

This cheat sheet provides a comprehensive reference for SQL derived tables, covering inline views, subqueries in `FROM`, aliasing, nesting, multi-stage construction, performance considerations, and materialisation behaviour across PostgreSQL, MySQL, and SQL Server. Always check execution plans to determine whether your derived table is being merged or materialised, and consider CTEs or temporary tables when materialisation is unavoidable.