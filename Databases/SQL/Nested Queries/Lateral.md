# SQL Lateral Subqueries: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A lateral subquery is a subquery in the `FROM` clause that can reference columns from tables appearing earlier in the same `FROM` clause, enabling row-by-row correlated evaluation of the subquery.

**Technical Definition**
In SQL, the `LATERAL` keyword (PostgreSQL, MySQL 8.0.14+) or the `APPLY` operator (SQL Server) allows a subquery in the `FROM` clause to reference columns from preceding tables in the same `FROM` clause. Without `LATERAL`, a subquery in `FROM` is evaluated independently and cannot see other tables in the same `FROM` clause. With `LATERAL`, the subquery is evaluated once per row of the preceding table(s), and its result is joined to the outer query. `CROSS JOIN LATERAL` behaves like an inner join (excluding rows with no matches), while `LEFT JOIN LATERAL` preserves all rows from the left side, padding with `NULL`s where the lateral subquery returns no rows. SQL Server's `CROSS APPLY` and `OUTER APPLY` are the equivalents of `CROSS JOIN LATERAL` and `LEFT JOIN LATERAL`, respectively.

**Beginner-Friendly Explanation**
A lateral subquery is a subquery that "sees" the current row of the table before it. Imagine you have a list of customers and want to find each customer's top three orders. A regular subquery cannot do this because it does not know which customer it is processing. A lateral subquery can—it runs for each customer row, using that customer's ID to look up their orders. It is the SQL way of saying "for each row on the left, run this subquery on the right."

---

### Key Characteristics

1. **Row-by-Row Correlation in `FROM`**: `LATERAL` extends correlation into the `FROM` clause, allowing a subquery to reference preceding tables—something a standard derived table cannot do.
2. **`CROSS JOIN LATERAL` vs. `LEFT JOIN LATERAL`**: `CROSS JOIN LATERAL` behaves like an inner join (rows with no match are excluded); `LEFT JOIN LATERAL` behaves like a left outer join (all left rows are preserved, with `NULL`s for non-matches).
3. **`APPLY` Is the SQL Server Equivalent**: `CROSS APPLY` corresponds to `CROSS JOIN LATERAL`; `OUTER APPLY` corresponds to `LEFT JOIN LATERAL`.
4. **Enables Top-N Per Group**: Lateral subqueries with `ORDER BY ... LIMIT n` implement top-N-per-group queries elegantly, without window functions or self-joins.
5. **Can Call Set-Returning Functions**: In PostgreSQL, `LATERAL` is commonly used with set-returning functions (e.g., `jsonb_array_elements`) to expand arrays or JSON documents row-by-row.
6. **Performance Depends on Indexes and Subquery Cost**: Lateral subqueries execute once per outer row; an index on the correlation column is essential for good performance. Without it, the query may perform a full scan per row.

---

### Prerequisites

- **Derived tables**: Subqueries in the `FROM` clause.
- **Join syntax**: `INNER JOIN`, `LEFT JOIN`, `CROSS JOIN`.
- **Correlated subqueries**: Understanding of outer references and per-row evaluation.
- **Table aliases**: Required for referencing the lateral subquery and its preceding tables.
- **Set-returning functions** (for PostgreSQL use cases): `jsonb_array_elements`, `generate_series`, etc.

---

### Related Programming Areas

| Area | Connection |
|------|------------|
| **Top-N Per Group** | Lateral joins with `LIMIT` are the cleanest way to get top-N per group. |
| **JSON/Array Expansion** | PostgreSQL lateral joins expand JSON arrays row-by-row. |
| **Table-Valued Functions** | SQL Server `APPLY` is the standard way to invoke table-valued functions per row. |
| **Query Performance** | Lateral joins can replace window functions or self-joins; the optimiser's ability to use indexes is critical. |
| **API Development** | Lateral joins reduce round trips by fetching per-row related data in one query. |

---

### Core Concepts / Features

1. **`LATERAL`**
2. **`CROSS JOIN LATERAL`**
3. **`LEFT JOIN LATERAL`**
4. **`APPLY` (SQL Server)**
5. **Use Cases**
6. **Performance**

---

## 1. `LATERAL`

### Definitions

**Core Definition**
`LATERAL` is a keyword that allows a subquery in the `FROM` clause to reference columns from preceding tables in the same `FROM` clause.

**Technical Definition**
The `LATERAL` keyword is placed before a subquery in the `FROM` clause (or before a function call) to indicate that the subquery may reference columns from tables that appear earlier in the same `FROM` clause. Without `LATERAL`, such references are not permitted in standard SQL. PostgreSQL documentation states that `LATERAL` allows a subquery to reference columns provided by preceding `FROM` items. MySQL 8.0.14+ supports `LATERAL` for derived tables.

**Beginner-Friendly Explanation**
`LATERAL` is the keyword that says "this subquery is allowed to look at the tables before it." It unlocks per-row subqueries in the `FROM` clause.

---

### Purposes

- To enable correlated subqueries in the `FROM` clause.
- To perform per-row lookups, aggregations, or expansions.
- To implement top-N-per-group queries.
- To call set-returning functions row-by-row.

---

### Syntax Rules and Structure

**General Syntax**
```
SELECT ...
FROM table1 t1
[INNER | LEFT] JOIN LATERAL (
  SELECT ... FROM table2 WHERE table2.col = t1.col
) AS sub ON true;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `table1 t1` | The preceding table (outer side of the lateral join). |
| `LATERAL` | Keyword enabling correlation. |
| `(subquery)` | The subquery that can reference `t1`. |
| `ON true` | Placeholder join condition (the real condition is inside the subquery). |

**Syntax Rules**

1. `LATERAL` is placed before the subquery or function call.
2. The subquery can reference columns from `table1`.
3. `ON true` is commonly used when the join condition is inside the subquery.
4. `LEFT JOIN LATERAL` preserves all left rows; `JOIN LATERAL` (inner) excludes rows with no matches.

**Constraints and Limitations**

- Supported in PostgreSQL and MySQL 8.0.14+; SQL Server uses `APPLY` instead.
- Lateral subqueries execute per outer row; without indexes, performance can degrade.
- Some optimisers cannot reorder lateral joins freely due to the correlation.

---

### Annotated Code Examples

**Example 1: Basic `LATERAL` Subquery**

```sql
SELECT u.name, o.amount, o.order_date
FROM users u
LEFT JOIN LATERAL (
  SELECT amount, order_date
  FROM orders
  WHERE orders.user_id = u.id
  ORDER BY order_date DESC
  LIMIT 3
) o ON true;
```

**Step-by-Step Explanation**

1. The outer query iterates over each user.
2. For each user, the lateral subquery retrieves their three most recent orders (`ORDER BY order_date DESC LIMIT 3`), using `u.id` from the outer row.
3. `LEFT JOIN LATERAL ... ON true` preserves users with no orders, showing `NULL`s for `amount` and `order_date`.

**Expected Output (illustrative)**:

| name  | amount | order_date |
|-------|--------|------------|
| Alice | 300    | 2026-03-15 |
| Alice | 250    | 2026-02-10 |
| Bob   | 150    | 2026-01-20 |
| Carol | NULL   | NULL       |

**Why This Output Occurs**: The lateral subquery runs per user, retrieving up to three recent orders; Carol has no orders and appears with `NULL`s.

---

### Real-World Cases

- **Top-N per group**: Most recent orders per customer, highest scores per player.
- **JSON expansion**: Expanding arrays or JSON documents per row.
- **Per-row aggregations**: Computing a running total or average per row.

---

## 2. `CROSS JOIN LATERAL`

### Definitions

**Core Definition**
`CROSS JOIN LATERAL` combines a lateral subquery with the preceding table using inner-join semantics: rows with no match in the lateral subquery are excluded.

**Technical Definition**
`CROSS JOIN LATERAL` is equivalent to `INNER JOIN LATERAL ... ON true`. It returns only rows where the lateral subquery produces at least one row. It is the PostgreSQL/MySQL equivalent of SQL Server's `CROSS APPLY`.

**Beginner-Friendly Explanation**
`CROSS JOIN LATERAL` says "for each row on the left, run this subquery and keep the result—but if the subquery returns nothing, drop that row."

---

### Purposes

- To perform per-row subqueries where every outer row must have a match.
- To expand set-returning functions where empty results should be excluded.
- To implement inner-join semantics with lateral correlation.

---

### Syntax Rules and Structure

```
SELECT ...
FROM table1 t1
CROSS JOIN LATERAL (
  SELECT ... FROM table2 WHERE table2.col = t1.col
) AS sub;
```

**Syntax Rules**

1. `CROSS JOIN LATERAL` does not require an `ON` clause.
2. Rows with no match in the lateral subquery are excluded.
3. The subquery can reference `t1` columns.

**Constraints and Limitations**

- Same as `LATERAL`; no match → row excluded.

---

### Annotated Code Examples

**Example 1: `CROSS JOIN LATERAL` for Top-N**

```sql
SELECT u.name, o.amount
FROM users u
CROSS JOIN LATERAL (
  SELECT amount FROM orders
  WHERE orders.user_id = u.id
  ORDER BY amount DESC
  LIMIT 2
) o;
```

**Expected Output (illustrative)**:

| name  | amount |
|-------|--------|
| Alice | 300    |
| Alice | 250    |
| Bob   | 150    |

**Why This Output Occurs**: Carol has no orders, so she is excluded (inner-join semantics). Alice and Bob appear with their top two order amounts.

---

### Real-World Cases

- **Top-N per group (inner join)**: Only groups with matches are included.
- **Set-returning function expansion**: Excluding rows with empty arrays.

---

## 3. `LEFT JOIN LATERAL`

### Definitions

**Core Definition**
`LEFT JOIN LATERAL` combines a lateral subquery with the preceding table using left-outer-join semantics: all rows from the left table are preserved, with `NULL`s where the lateral subquery returns no rows.

**Technical Definition**
`LEFT JOIN LATERAL ... ON true` is the left-outer-join version of a lateral join. It preserves all rows from the left side, padding the lateral subquery's columns with `NULL`s when the subquery returns no rows. It is the PostgreSQL/MySQL equivalent of SQL Server's `OUTER APPLY`.

**Beginner-Friendly Explanation**
`LEFT JOIN LATERAL` says "for each row on the left, run this subquery—and even if it returns nothing, keep the left row with `NULL`s."

---

### Purposes

- To preserve all left rows while optionally attaching per-row subquery results.
- To implement top-N-per-group where groups with no matches must still appear.
- To safely expand optional set-returning functions.

---

### Syntax Rules and Structure

```
SELECT ...
FROM table1 t1
LEFT JOIN LATERAL (
  SELECT ... FROM table2 WHERE table2.col = t1.col
) AS sub ON true;
```

**Syntax Rules**

1. `ON true` is required (or `ON` with a condition).
2. All rows from `t1` are preserved.
3. `NULL` padding occurs for non-matches.

**Constraints and Limitations**

- Same performance considerations as `LATERAL`.

---

### Annotated Code Examples

**Example 1: `LEFT JOIN LATERAL` Preserving All Users**

```sql
SELECT u.name, o.amount
FROM users u
LEFT JOIN LATERAL (
  SELECT amount FROM orders
  WHERE orders.user_id = u.id
  ORDER BY amount DESC
  LIMIT 1
) o ON true;
```

**Expected Output (illustrative)**:

| name  | amount |
|-------|--------|
| Alice | 300    |
| Bob   | 150    |
| Carol | NULL   |

**Why This Output Occurs**: Carol has no orders, but `LEFT JOIN LATERAL` preserves her row with `NULL` for `amount`.

---

### Real-World Cases

- **Customer reports**: All customers, with their most recent order (if any).
- **Optional JSON expansion**: Expanding arrays where some rows have no array elements.

---

## 4. `APPLY` (SQL Server)

### Definitions

**Core Definition**
`APPLY` is SQL Server's operator for invoking a table-valued subquery or function for each row of an outer table, equivalent in purpose to `LATERAL`.

**Technical Definition**
`CROSS APPLY` behaves like `CROSS JOIN LATERAL`: it returns only rows where the right-side expression returns at least one row. `OUTER APPLY` behaves like `LEFT JOIN LATERAL`: it preserves all left rows, returning `NULL`s where the right-side expression returns no rows. `APPLY` is commonly used with table-valued functions and correlated subqueries.

**Beginner-Friendly Explanation**
`APPLY` is SQL Server's way of saying "for each row on the left, run this subquery or function on the right." `CROSS APPLY` keeps only matches; `OUTER APPLY` keeps all left rows.

---

### Purposes

- To call table-valued functions row-by-row.
- To replace scalar subqueries in `SELECT` with table-valued expressions that can return multiple columns.
- To implement top-N-per-group queries.
- To perform per-row lookups and expansions.

---

### Syntax Rules and Structure

```
SELECT ...
FROM table1 t1
{ CROSS | OUTER } APPLY (
  SELECT ... FROM table2 WHERE table2.col = t1.col
) AS sub;
```

**Syntax Rules**

1. `CROSS APPLY` excludes rows with no match; `OUTER APPLY` preserves them.
2. The subquery can reference `t1` columns.
3. `APPLY` can also invoke table-valued functions: `CROSS APPLY dbo.MyFunction(t1.col)`.

**Constraints and Limitations**

- SQL Server-specific.
- `CROSS APPLY` may be more efficient than `OUTER APPLY` when matches are expected.
- Cannot be used with `LATERAL` (SQL Server uses `APPLY` instead).

---

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

**Explanation**: For each user, the `CROSS APPLY` subquery retrieves their three most recent posts. Users with no posts are excluded because `CROSS APPLY` is like an inner join.

---

**Example 2: `OUTER APPLY` for All Users**

```sql
SELECT u.Id, u.DisplayName, p.Title
FROM dbo.Users AS u
OUTER APPLY (
  SELECT TOP 1 p.Title FROM dbo.Posts AS p
  WHERE p.OwnerUserId = u.Id
  ORDER BY p.CreationDate DESC
) AS p;
```

**Explanation**: `OUTER APPLY` preserves all users, showing `NULL` for `Title` when a user has no posts.

---

### Real-World Cases

- **Top-N per group**: Recent posts per user, recent orders per customer.
- **Table-valued functions**: Calling a function that returns multiple rows per input row.
- **Per-row lookups**: Retrieving related data for each row without a self-join.

---

## 5. Use Cases

### Definitions

**Core Definition**
Use cases for lateral subqueries are scenarios where per-row correlated evaluation in the `FROM` clause provides a clean, efficient solution that would otherwise require complex window functions, self-joins, or application-side logic.

**Technical Definition**
Lateral subqueries are particularly suited to: (1) top-N-per-group queries; (2) expanding set-returning functions (JSON arrays, `generate_series`) row-by-row; (3) per-row aggregations or lookups that return multiple columns; (4) replacing scalar subqueries in `SELECT` that need to return more than one column; and (5) invoking table-valued functions per row.

**Beginner-Friendly Explanation**
Lateral subqueries are the right tool when you need to do something "for each row" that returns multiple columns or rows—like "for each customer, get their top 3 orders" or "for each row, expand this JSON array."

---

### Purposes

- To implement top-N-per-group queries cleanly.
- To expand JSON/array data row-by-row.
- To invoke table-valued functions per row.
- To replace complex self-joins or window functions.

---

### Syntax Rules and Structure

No separate syntax; the use cases are implemented with the syntax from Sections 1–4.

---

### Annotated Code Examples

**Example 1: Top-N Per Group (PostgreSQL)**

```sql
SELECT u.name, o.amount, o.order_date
FROM users u
LEFT JOIN LATERAL (
  SELECT amount, order_date
  FROM orders
  WHERE orders.user_id = u.id
  ORDER BY order_date DESC
  LIMIT 3
) o ON true;
```

**Explanation**: This is the canonical top-N-per-group pattern; no window function or self-join required.

---

**Example 2: JSON Array Expansion (PostgreSQL)**

```sql
SELECT u.id, u.name, tag.value AS tag
FROM users u
CROSS JOIN LATERAL jsonb_array_elements_text(u.tags) AS tag(value);
```

**Explanation**: `jsonb_array_elements_text` is a set-returning function; `CROSS JOIN LATERAL` expands each user's JSON array into one row per tag, excluding users with no tags.

---

**Example 3: Table-Valued Function (SQL Server)**

```sql
SELECT u.Id, u.DisplayName, p.Title
FROM dbo.Users AS u
CROSS APPLY dbo.GetRecentPosts(u.Id) AS p;
```

**Explanation**: `GetRecentPosts` is a table-valued function invoked once per user; `CROSS APPLY` excludes users with no posts.

---

### Real-World Cases

- **E-commerce**: Top 3 products per category.
- **Social media**: Recent posts per user.
- **Analytics**: Expanding JSON event data into rows.
- **Reporting**: Per-row lookup of related records without self-joins.

---

## 6. Performance

### Definitions

**Core Definition**
Performance of lateral subqueries concerns the cost of evaluating the subquery once per outer row, the availability of indexes on the correlation column, and the optimiser's ability to choose efficient plans.

**Technical Definition**
Lateral subqueries execute logically once per outer row, often translating to nested-loop execution. Performance depends on: (1) an index on the correlation column (e.g., `orders.user_id`); (2) the cost of the subquery itself (e.g., sorting and limiting in top-N queries); (3) the number of outer rows; and (4) the optimiser's ability to push predicates and use indexes. SQL Server documentation notes that `CROSS APPLY` can be more efficient than `OUTER APPLY` when matches are expected, because it avoids `NULL` handling. PostgreSQL documentation notes that `LATERAL` subqueries are often used with set-returning functions and can be very efficient with proper indexes.

**Beginner-Friendly Explanation**
Lateral subqueries run again for each outer row, so they can be slow if the outer table is huge and the subquery has no index to use. An index on the correlation column makes them fast—like a lookup instead of a scan.

---

### Purposes (of Understanding Performance)

- To diagnose slow lateral queries.
- To design indexes that support lateral joins.
- To choose between `CROSS JOIN LATERAL` and `LEFT JOIN LATERAL`.
- To compare lateral joins with window functions and self-joins.

---

### Syntax Rules and Structure (Optimisation Guidelines)

1. **Index the correlation column**: An index on `orders.user_id` enables efficient per-row lookups.
2. **Use `CROSS JOIN LATERAL` when matches are expected**: It avoids `NULL` handling and may be more efficient than `LEFT JOIN LATERAL`.
3. **Limit the outer result set**: Filtering the outer table to fewer rows reduces the number of lateral subquery executions.
4. **Limit the lateral subquery result**: `ORDER BY ... LIMIT n` in the lateral subquery reduces per-row output and can use indexes for sorting.
5. **Compare with window functions**: For top-N-per-group, window functions (`ROW_NUMBER()`) and lateral joins are often interchangeable; test both.
6. **Check execution plans**: Look for nested loops with index scans inside the lateral subquery.

**Constraints and Limitations**

- Without an index on the correlation column, each lateral execution may scan the inner table.
- Lateral joins cannot be reordered as freely as regular joins due to correlation.
- `LEFT JOIN LATERAL` with `ON true` may prevent some optimiser transformations.

---

### Annotated Code Examples

**Example 1: Indexed Lateral Join**

```sql
-- Index: CREATE INDEX idx_orders_user_id ON orders(user_id);
SELECT u.name, o.amount
FROM users u
LEFT JOIN LATERAL (
  SELECT amount FROM orders
  WHERE orders.user_id = u.id
  ORDER BY amount DESC
  LIMIT 1
) o ON true;
```

**Explanation**: The index on `orders.user_id` allows the lateral subquery to find the top order per user efficiently, without scanning the entire `orders` table for each user.

---

**Example 2: `CROSS APPLY` vs. `OUTER APPLY` Performance**

```sql
-- CROSS APPLY: excludes users with no posts (inner-join semantics)
SELECT u.Id, p.Title
FROM dbo.Users u
CROSS APPLY (
  SELECT TOP 1 p.Title FROM dbo.Posts p
  WHERE p.OwnerUserId = u.Id
  ORDER BY p.CreationDate DESC
) p;

-- OUTER APPLY: preserves users with no posts (left-outer-join semantics)
SELECT u.Id, p.Title
FROM dbo.Users u
OUTER APPLY (
  SELECT TOP 1 p.Title FROM dbo.Posts p
  WHERE p.OwnerUserId = u.Id
  ORDER BY p.CreationDate DESC
) p;
```

**Explanation**: `CROSS APPLY` may be more efficient when most users have posts, because it avoids producing `NULL` rows; `OUTER APPLY` is required when all users must appear.

---

### Real-World Cases

- **Top-N per group**: Indexed lateral joins scale well for per-group lookups.
- **JSON expansion**: Lateral joins with set-returning functions are the standard way to flatten JSON in PostgreSQL.
- **Table-valued functions**: `CROSS APPLY` with indexed functions is efficient for per-row computations.

---

## References

1. **PostgreSQL Documentation — LATERAL Subqueries** — https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-LATERAL
2. **PostgreSQL Documentation — Set-Returning Functions and LATERAL** — https://www.postgresql.org/docs/current/functions-srf.html
3. **MySQL 8.0 Reference Manual — Lateral Derived Tables** — https://dev.mysql.com/doc/refman/8.0/en/lateral-derived-tables.html
4. **Microsoft SQL Server — APPLY (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/queries/from-transact-sql#using-apply
5. **Microsoft Learn — Use APPLY** — https://learn.microsoft.com/en-us/sql/t-sql/queries/from-transact-sql
6. **Erik Darling — Cross Apply and Outer Apply Cheat Sheet** — https://erikdarling.com/the-how-to-write-sql-server-queries-correctly-cheat-sheet-cross-apply-and-outer-apply/
7. **Cybrosys — How to Use LATERAL JOINs in PostgreSQL** — https://www.cybrosys.com/research-and-development/postgres/how-to-use-lateral-joins-in-postgresql-for-powerful-queries
8. **SQL:2016 Standard — ISO/IEC 9075-2:2016 (Foundation)** — https://www.iso.org/standard/63555.html

---

This cheat sheet provides a comprehensive reference for SQL lateral subqueries, covering `LATERAL`, `CROSS JOIN LATERAL`, `LEFT JOIN LATERAL`, SQL Server's `APPLY`, use cases (top-N per group, JSON expansion, table-valued functions), and performance considerations. Always index the correlation column, choose `CROSS`/`CROSS APPLY` when matches are expected, and compare lateral joins with window functions for top-N-per-group queries.