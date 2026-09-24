# SQL Performance Optimization (Code Level): A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL performance optimization at the code level is the practice of writing and rewriting SQL queries to minimise resource consumption (CPU, I/O, memory, network) while maximising execution speed, without changing the database schema or server configuration.

**Technical Definition:** Code-level SQL optimization focuses on the declarative structure of queries: the columns selected, the predicates applied, the join strategies implied, the pagination method used, and the use of subqueries, CTEs, and window functions. These factors influence the optimizer's ability to choose efficient execution plans, particularly the use of indexes and the avoidance of unnecessary data movement.

**Beginner-Friendly Explanation:** Writing fast SQL is like giving clear directions to a driver. If you say "take me anywhere" (SELECT *), the driver has to figure out where to go. If you say "take me to 123 Main Street using the highway" (specific columns, indexed predicate), the driver gets there faster. Code-level optimization is about giving the database clear, efficient instructions.

### Key Characteristics

- **Index-aware:** Queries should be written so that the optimizer can use indexes (SARGable predicates).
- **Minimal-data-oriented:** Only the necessary columns and rows should be retrieved.
- **Set-based:** Operations should be performed in the database engine, not in application code.
- **Plan-friendly:** Query structure should not defeat the optimizer's ability to find efficient plans.

### Prerequisites

- **SQL proficiency:** Comfort with SELECT, JOIN, WHERE, and subqueries.
- **Index fundamentals:** Understanding of how B-tree indexes work and when they are used.
- **Execution plan literacy:** Ability to read EXPLAIN output to validate optimization efforts.

### Related Programming Areas

- **Query Optimization:** Code-level optimization is the first line of defence before schema or server tuning.
- **Application Performance:** Slow queries are a common bottleneck in web and enterprise applications.
- **Database Administration:** DBAs often recommend query rewrites as part of performance tuning.

### Core Concepts / Features

1. Avoiding Unnecessary Columns
2. Reducing Unnecessary Rows
3. Predicate Optimization (SARGability)
4. Join Optimization
5. Pagination Optimization
6. Query Rewriting (Subqueries, CTEs)
7. Window Functions Performance

---

## 1. Avoiding Unnecessary Columns

### Definitions

**Core Definition:** Avoiding unnecessary columns means explicitly listing only the columns a query actually needs, rather than using `SELECT *` to retrieve all columns.

**Technical Definition:** `SELECT *` retrieves every column from the referenced tables, including large text fields, BLOBs, JSON documents, and columns not used by the application. This increases I/O (reading additional data pages), memory usage (buffering unused data), network traffic (transmitting unused data), and can prevent covering index optimizations.

**Beginner-Friendly Explanation:** `SELECT *` says "give me everything." But if you only need a user's name and email, you don't need their password hash, avatar image, and JSON preferences. Asking for everything wastes time and resources.

### Purposes

- To reduce I/O by reading only needed data pages.
- To reduce network transfer by sending only relevant columns.
- To reduce application memory usage by not loading unused data.
- To enable covering index optimizations (where the index contains all needed columns).
- To avoid exposing sensitive data (passwords, tokens).

### Syntax Rules and Structure

**Correct Syntax:**

```sql
SELECT column1, column2, column3
FROM table_name
WHERE condition;
```

**Incorrect (over-fetching):**

```sql
SELECT * FROM table_name WHERE condition;
```

**Syntax Rules:**

- Explicitly list every column needed by the application.
- Column order in the SELECT list should match the order the application expects.
- `COUNT(*)` is acceptable because it does not fetch column data; it counts rows.

**Constraints and Limitations:**

- `SELECT *` is acceptable in ad-hoc exploration queries where the full row is genuinely needed.
- In subqueries following `EXISTS` or `NOT EXISTS`, `SELECT *` is semantically equivalent to `SELECT 1` and has no performance penalty .
- Changing table schemas (adding columns) can break applications that rely on `SELECT *` column ordering.

### Annotated Complete Code Examples

**Example 1: Explicit Columns Enable Covering Index**

```sql
-- Assume an index exists: (status, created_at, id, total) on orders

-- SLOW: SELECT * fetches all columns; index cannot cover the query
SELECT * FROM orders WHERE status = 'pending';

-- FAST: Only needed columns; index can cover the query
SELECT id, total, created_at FROM orders WHERE status = 'pending';
```

**Why this matters:** The index `(status, created_at, id, total)` contains all columns needed by the second query. The database can satisfy the query entirely from the index ("Using index" in EXPLAIN), avoiding table access. The first query must fetch the full row from the table because `SELECT *` requests columns not in the index .

**Example 2: Avoiding Large Column Over-Fetch**

```sql
-- Assume users table has: id, email, name, password_hash, avatar_blob, preferences_json

-- BAD: Fetches password_hash, avatar_blob, preferences_json unnecessarily
SELECT * FROM users WHERE id = 123;

-- GOOD: Only what the application needs for display
SELECT id, email, name FROM users WHERE id = 123;
```

**Why this matters:** The `avatar_blob` and `preferences_json` columns may be large. Fetching them wastes network bandwidth and memory. The explicit column list also documents what the application actually uses, improving code clarity and security .

### Real-World Cases

**Case 1: Cloud Database Cost Optimization**

A company migrates to a cloud database (SQL Azure) and discovers that `SELECT *` queries are incurring data transfer charges for columns never used by the application. Replacing `SELECT *` with explicit column lists reduces costs and improves performance .

**Case 2: API Response Optimization**

A REST API returns user profiles. The original implementation uses `SELECT * FROM users`. After switching to `SELECT id, email, name, avatar_url FROM users`, response payload size drops by 60% and response times improve.

### References

- Stack Overflow — Reasons to avoid SELECT * in production code - https://stackoverflow.com/revisions/cf16a939-01f1-412a-b01c-fd031e5b4a05/view-source
- GitHub — MySQL: Avoid Using SELECT * in Production - https://github.com/OneUptime/blog/commit/21e12e16ff8d640c12eba80ac6ece3465ad401be
- Microsoft — SELECT * AND SQL Azure - https://learn.microsoft.com/nb-no/archive/blogs/appfabricannounce/select-and-sql-azure
- GitHub — Select Only Needed Columns - https://github.com/ravnhq/ai-toolkit/blob/main/skills/database/platform-database/rules/query-select-columns.md

---

## 2. Reducing Unnecessary Rows

### Definitions

**Core Definition:** Reducing unnecessary rows means applying filters as early as possible in query execution (predicate pushdown) and avoiding fetching rows to the application only to filter them client-side.

**Technical Definition:** Predicate pushdown is the optimizer's ability to apply `WHERE` conditions as close to the data source as possible, reducing the number of rows that flow through subsequent operations (joins, sorts, aggregations). When filters are applied in the application instead of the database, the database transmits more rows than necessary, wasting I/O and network bandwidth.

**Beginner-Friendly Explanation:** Imagine ordering a pizza and asking for "all pizzas on the menu" just to pick out the pepperoni at home. You'd get a huge delivery for no reason. Applying filters in the database is like saying "just the pepperoni pizza" upfront.

### Purposes

- To reduce the number of rows that must be transmitted to the application.
- To enable the optimizer to use indexes for filtering.
- To reduce the input size for subsequent operations (joins, sorts, aggregates).
- To avoid wasting resources on rows that will be discarded.

### Syntax Rules and Structure

**Correct Approach (filter in database):**

```sql
SELECT id, name, price
FROM products
WHERE category = 'Electronics' AND price > 100;
```

**Incorrect Approach (filter client-side):**

```sql
-- Fetches ALL products, then application filters
SELECT id, name, price, category FROM products;
-- Application code: if (product.category == 'Electronics' && product.price > 100) ...
```

**Syntax Rules:**

- Always include `WHERE` predicates for conditions the application knows at query time.
- Place the most restrictive predicates first logically (though the optimizer reorders them).
- Use `HAVING` only for aggregate filters; use `WHERE` for row-level filters.

**Constraints and Limitations:**

- Some filters depend on application logic that cannot be expressed in SQL (e.g., external API calls, complex business rules).
- Very restrictive filters on unindexed columns may still require a full table scan, but the result set is smaller.

### Annotated Complete Code Examples

**Example 1: Filtering in the Database**

```sql
-- GOOD: Database filters rows
SELECT order_id, customer_id, total
FROM orders
WHERE order_date >= '2026-01-01'
  AND status = 'completed'
  AND total > 1000;

-- Expected Output (conceptual):
--  order_id | customer_id | total
-- ----------+-------------+-------
--      1001 |       12345 | 1500.00
--      1057 |       67890 | 2200.00
```

**Why this matters:** The database applies the date range, status, and total filters before returning rows. If an index exists on `order_date`, the optimizer can use it to seek directly to the relevant rows. Only matching rows are transmitted to the application.

**Example 2: Client-Side Filtering (Anti-Pattern)**

```sql
-- BAD: Fetches all rows, application filters
SELECT order_id, customer_id, total, status, order_date
FROM orders;

-- Application code then loops through millions of rows:
-- if (order.status === 'completed' && order.total > 1000 && order.order_date >= '2026-01-01')
```

**Why this is inefficient:** The database transmits every row in the `orders` table. The application loads all rows into memory, then discards most of them. This wastes I/O, network bandwidth, and application memory. Oracle documentation notes that "row filtering is performed in the Java program. This is very inefficient" .

### Real-World Cases

**Case 1: E-Commerce Order History**

A web application displays a user's recent orders. The original code fetches all orders and filters in JavaScript. After moving the filter to the SQL query (`WHERE user_id = ? AND order_date > ?`), page load time drops from 3 seconds to 200 ms.

**Case 2: Data Warehouse Extraction**

An ETL process extracts only rows modified since the last run. Using `WHERE updated_at > :last_extract_time` in the extraction query reduces the data volume from millions of rows to thousands.

### References

- Oracle Database Performance Tuning Guide - https://docs.oracle.com/en/database/oracle/oracle-database/26/tgdba/designing-and-developing-for-performance.html

---

## 3. Predicate Optimization (SARGability)

### Definitions

**Core Definition:** A SARGable (Search ARGument-able) predicate is a `WHERE` condition that the database can use an index to evaluate efficiently. Non-SARGable predicates force a full table scan even when an index exists.

**Technical Definition:** A predicate is SARGable if it involves a comparison of an indexed column with a constant, a variable, or an expression that does not transform the column. Functions applied to the column (e.g., `YEAR(OrderDate)`, `UPPER(name)`) make a predicate non-SARGable because the index stores raw column values, not function results.

**Beginner-Friendly Explanation:** A SARGable predicate is like a precise search term that the index can look up. A non-SARGable predicate is like asking the index to do math or transformation on every entry—it can't, so it gives up and scans everything.

### Purposes

- To ensure the optimizer can use indexes for filtering.
- To avoid full table scans on large tables.
- To reduce CPU usage by avoiding per-row function evaluation.
- To enable efficient range queries.

### Syntax Rules and Structure

**SARGable vs. Non-SARGable Predicates:**

| SARGable | Non-SARGable | Fix |
|----------|--------------|-----|
| `column = 10` | `column <> 10` | Use `> OR <` if possible |
| `column > 25` | `YEAR(column) = 2000` | Rewrite as range |
| `column LIKE 'pat%'` | `column LIKE '%tern'` | Cannot fix without full-text |
| `column IN (4,5,6)` | `column NOT IN (4,5,6)` | Rewrite as `<>` with caution |
| `column = 20 - 2` | `column + 2 = 20` | Move math to other side |
| `column IS NULL` | `column = NULL` | Use `IS NULL` |

**Syntax Rules:**

- Do not apply functions to indexed columns in `WHERE` clauses.
- Move arithmetic to the constant side: `column = 20 - 2` is SARGable; `column + 2 = 20` is not.
- `LIKE` patterns must be prefix-anchored (`'pat%'`) to use B-tree indexes.
- `<>` and `!=` operators do not use indexes (they match "everything except").

**Constraints and Limitations:**

- Some non-SARGable predicates can be made SARGable with computed columns or function-based indexes.
- `LIKE '%tern'` (suffix search) cannot use a standard B-tree index; full-text search is required.
- The optimizer may automatically transform some non-SARGable predicates (e.g., `LIKE 'pat%'` to range scans) but not all.

### Annotated Complete Code Examples

**Example 1: Non-SARGable to SARGable Rewrite**

```sql
-- NON-SARGABLE: Function on indexed column
SELECT * FROM SalesOrders WHERE YEAR(OrderDate) = '2000';

-- SARGABLE: Range predicate
SELECT * FROM SalesOrders
WHERE OrderDate > '1999-12-31'
  AND OrderDate < '2001-01-01';
```

**Why this matters:** The first query applies `YEAR()` to every row's `OrderDate`, preventing index usage. The second query uses the raw `OrderDate` column with range operators, allowing the B-tree index on `OrderDate` to be used for a range scan .

**Example 2: Arithmetic Rewrite**

```sql
-- NON-SARGABLE: Arithmetic on indexed column
SELECT * FROM products WHERE price + 100 = 500;

-- SARGABLE: Arithmetic on constant
SELECT * FROM products WHERE price = 500 - 100;
```

**Why this matters:** In the first query, the database must compute `price + 100` for every row. In the second query, `500 - 100` is a constant (400), and the predicate becomes `price = 400`, which can use an index on `price`.

### Real-World Cases

**Case 1: Date Range Reporting**

A reporting query uses `WHERE YEAR(transaction_date) = 2026`. The DBA rewrites it as `WHERE transaction_date >= '2026-01-01' AND transaction_date < '2027-01-01'`. Execution time drops from 12 seconds to 0.5 seconds because the index on `transaction_date` is now usable.

**Case 2: Case-Insensitive Search**

An application searches `WHERE UPPER(email) = 'ALICE@EXAMPLE.COM'`. The function on `email` prevents index usage. The fix is a functional index on `UPPER(email)` or storing emails in lowercase and querying `WHERE email = 'alice@example.com'`.

### References

- SAP SQL Anywhere — Query Predicates - https://help.sap.com/docs/SAP_SQL_Anywhere/e38b2f6217f24bdb90a3ff8ae57b1dd5/818efd706ce21014b529ebb906015a7a.html
- SAP SQL Anywhere — Sargable Predicates (German) - https://help.sap.com/doc/57ec2dbe4d3d4be0a91bfd3eb2c0b3f3/17.0/de-DE/SQL-Anywhere-Server-Users-Guide-de.pdf

---

## 4. Join Optimization

### Definitions

**Core Definition:** Join optimization involves writing joins that the optimizer can execute efficiently, avoiding accidental cross joins, placing filtering predicates correctly, and leveraging indexes on join columns.

**Technical Definition:** An accidental cross join occurs when a join condition is missing or incomplete, producing a Cartesian product. Predicate placement matters: filters on the "inner" table of an outer join must be in the `ON` clause, not the `WHERE` clause, to preserve outer join semantics and performance.

**Beginner-Friendly Explanation:** A join combines rows from two tables. If you forget to say how they match, the database pairs every row with every other row—a huge, useless result. Join optimization is about telling the database exactly how to match rows, and filtering early so the join has less work to do.

### Purposes

- To avoid Cartesian products that explode result set sizes.
- To ensure filtering predicates are applied at the right stage.
- To enable the optimizer to choose the best join algorithm.
- To leverage indexes on join columns.

### Syntax Rules and Structure

**Correct Inner Join:**

```sql
SELECT o.order_id, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.status = 'completed';
```

**Accidental Cross Join (Missing Join Condition):**

```sql
SELECT o.order_id, c.customer_name
FROM orders o, customers c
WHERE o.status = 'completed';  -- Missing: o.customer_id = c.customer_id
```

**Outer Join Predicate Placement:**

```sql
-- CORRECT: Filter on inner table in ON clause
SELECT o.order_id, c.customer_name
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id AND c.active = 1;

-- INCORRECT: Filter in WHERE defeats LEFT JOIN
SELECT o.order_id, c.customer_name
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.active = 1;  -- Turns LEFT JOIN into INNER JOIN
```

**Syntax Rules:**

- Always include explicit `ON` conditions for joins.
- Use `JOIN` syntax rather than comma-separated tables with `WHERE` conditions (more readable, harder to accidentally cross join).
- Place inner-table filters in the `ON` clause for outer joins.

**Constraints and Limitations:**

- Some legacy code uses comma joins; these are valid but error-prone.
- The optimizer may reorder joins for performance, but correct predicate placement is the developer's responsibility.

### Annotated Complete Code Examples

**Example 1: Avoiding Accidental Cross Join**

```sql
-- BAD: Accidental cross join (missing join condition)
SELECT o.order_id, c.customer_name
FROM orders o, customers c
WHERE o.status = 'completed';

-- GOOD: Explicit join condition
SELECT o.order_id, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.status = 'completed';
```

**Why this matters:** The first query produces every order paired with every customer—a Cartesian product. If there are 10,000 orders and 1,000 customers, the result has 10 million rows. The second query correctly pairs orders with their customers, producing at most 10,000 rows.

**Example 2: Predicate Placement in Outer Joins**

```sql
-- Assume: orders LEFT JOIN customers, want all orders with active customer info

-- CORRECT: Filter in ON clause
SELECT o.order_id, c.customer_name
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id AND c.active = 1;

-- INCORRECT: Filter in WHERE eliminates unmatched orders
SELECT o.order_id, c.customer_name
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.active = 1;
```

**Why this matters:** The first query returns all orders. Orders with inactive customers show NULL for customer_name. The second query filters out rows where `c.active` is NULL (which happens for unmatched orders), effectively turning the LEFT JOIN into an INNER JOIN.

### Real-World Cases

**Case 1: Legacy Comma Joins**

A legacy application uses comma-separated tables with all conditions in `WHERE`. A developer adds a new table without a join condition, accidentally creating a cross join that multiplies result rows by 1,000x. The fix is to refactor to explicit `JOIN ... ON` syntax.

**Case 2: Outer Join with Filter**

A report lists all products and their sales, including products with no sales. The original query uses `LEFT JOIN sales ON ... WHERE sales.region = 'North'`, which excludes products with no sales. Moving the filter to `ON sales.region = 'North'` preserves the outer join.

### References

- Oracle Software Development Kit Developer Guide — Table Indices and SQL - https://docs.oracle.com/en/industries/energy-water/framework/2510/sdk-dev-guide/Topics/SDK_Table_Indices_and_SQL.html

---

## 5. Pagination Optimization

### Definitions

**Core Definition:** Pagination optimization is the practice of using keyset pagination (the "seek method") instead of high-offset pagination (`OFFSET N`) to retrieve pages of data efficiently.

**Technical Definition:** `OFFSET N` requires the database to generate and discard the first `N` rows before returning the requested page. Keyset pagination uses a `WHERE` predicate on an indexed sort key to "seek" directly to the next page, achieving roughly constant cost regardless of page depth. Row-value (tuple) comparison is essential for correctness with non-unique sort columns.

**Beginner-Friendly Explanation:** `OFFSET 100000` is like reading the first 100,000 pages of a book just to get to page 100,001. Keyset pagination is like using the index to jump straight to the right page. It's faster and doesn't slow down as you go deeper.

### Purposes

- To maintain consistent performance regardless of page depth.
- To avoid the linear cost growth of `OFFSET` on large datasets.
- To prevent duplicate or skipped rows when data changes between page requests.
- To support infinite scroll and feed-style interfaces.

### Syntax Rules and Structure

**Slow Offset Pagination:**

```sql
SELECT id, title, created_at
FROM articles
ORDER BY created_at DESC, id DESC
LIMIT 20 OFFSET 200000;
```

**Fast Keyset Pagination:**

```sql
SELECT id, title, created_at
FROM articles
WHERE (created_at, id) < (:last_created, :last_id)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

**Syntax Rules:**

- Always include a unique tiebreaker (usually the primary key) as the last sort column.
- Use row-value comparison `(col1, col2) < (:val1, :val2)` for compound keys.
- Create a composite index matching the sort columns and direction exactly.
- The comparison operator must align with sort direction: `DESC` pairs with `<`, `ASC` pairs with `>`.

**Constraints and Limitations:**

- Keyset pagination does not support random access to numbered pages (e.g., "go to page 500").
- The client must store and send the cursor (last sort values) with each request.
- Concurrent inserts or updates can affect cursor stability; boundary capture may be needed for mission-critical applications.

### Annotated Complete Code Examples

**Example 1: Offset vs. Keyset Performance**

```sql
-- SLOW: OFFSET 200000 scans and discards 200,000 rows
SELECT id, title, created_at FROM articles
ORDER BY created_at DESC, id DESC
LIMIT 20 OFFSET 200000;

-- FAST: Keyset seeks directly to the next page
SELECT id, title, created_at FROM articles
WHERE (created_at, id) < ('2025-01-15', 12345)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

**Why this matters:** A real-world migration from offset to keyset pagination reported query times dropping from 5 seconds to 500 ms . The `OFFSET` query must walk past 200,000 rows before returning anything. The keyset query uses the index to seek directly to the position where `created_at < '2025-01-15'` or `(created_at = '2025-01-15' AND id < 12345)` .

**Example 2: Compound Keyset with Non-Unique Sort**

```sql
-- Assume created_at is not unique (multiple articles per second)
-- Composite index: (created_at DESC, id DESC)

-- CORRECT: Row-value comparison prevents skipped/duplicate rows
SELECT id, title, created_at FROM articles
WHERE (created_at, id) < ('2025-01-15 10:30:00', 12345)
ORDER BY created_at DESC, id DESC
LIMIT 20;

-- INCORRECT: Independent comparison skips rows
SELECT id, title, created_at FROM articles
WHERE created_at <= '2025-01-15 10:30:00' AND id < 12345
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

**Why this matters:** The independent comparison `created_at <= X AND id < Y` is not the same as `(created_at, id) < (X, Y)`. The row-value comparison correctly handles the case where `created_at = X` and `id` determines ordering. Without it, rows sharing the boundary timestamp may be skipped or duplicated .

### Real-World Cases

**Case 1: Infinite Scroll API**

A social media feed uses keyset pagination to load the next 20 posts when the user scrolls. The client stores the `(created_at, id)` of the last post and sends it with the next request. The API uses a `WHERE (created_at, id) < (:cursor)` predicate, returning results in milliseconds regardless of how deep the user scrolls.

**Case 2: ETL Batch Processing**

An ETL job processes 10 million records in batches of 10,000. Using keyset pagination (`WHERE (batch_timestamp, record_id) > (:last_processed)`), the job maintains consistent performance from the first batch to the last, unlike `OFFSET` which would slow down progressively .

### References

- Stacksync — Postgres Pagination: Keyset vs Offset vs Cursor - https://www.stacksync.com/blog/keyset-cursors-postgres-pagination-fast-accurate-scalable
- GitHub — Keyset Pagination - https://raw.githubusercontent.com/armourinfosec/Secure-PHP-Development/refs/heads/main/Using-PHP-to-Access-MySQL/Keyset-Pagination.md
- jOOQ Manual — Keyset Pagination / SEEK - https://www.jooq.org/doc/3.14/manual-pdf/jOOQ-manual-3.14.pdf

---

## 6. Query Rewriting (Subqueries, CTEs)

### Definitions

**Core Definition:** Query rewriting is the practice of restructuring a query—converting subqueries to joins, or using Common Table Expressions (CTEs) efficiently—to improve readability and, in some cases, performance.

**Technical Definition:** In MySQL 8.0+, CTEs referenced multiple times are materialized once into a temporary table, whereas a view referenced multiple times is materialized separately for each reference. This can halve execution time for queries that reference the same derived result more than once. In other databases, CTEs and subqueries are often treated equivalently by the optimizer.

**Beginner-Friendly Explanation:** A CTE is a named subquery you can reference like a table. If you use the same subquery twice, a CTE lets the database compute it once and reuse the result, instead of doing the work twice.

### Purposes

- To improve readability by naming complex subqueries.
- To materialize reused subqueries once instead of recomputing them.
- To simplify maintenance of complex analytical queries.
- To enable the optimizer to make better decisions with a clearer query structure.

### Syntax Rules and Structure

**CTE Syntax:**

```sql
WITH cte_name (column_list) AS (
    SELECT ...
)
SELECT ... FROM cte_name ...
```

**View vs. CTE Materialization (MySQL 8.0+):**

```sql
-- View: materialized once per reference
CREATE VIEW revenue0 AS SELECT ...;
SELECT ... FROM supplier, revenue0 WHERE ... AND total_revenue = (SELECT MAX(total_revenue) FROM revenue0);

-- CTE: materialized once total
WITH revenue0 AS (SELECT ...)
SELECT ... FROM supplier, revenue0 WHERE ... AND total_revenue = (SELECT MAX(total_revenue) FROM revenue0);
```

**Syntax Rules:**

- CTEs are defined with `WITH` before the main `SELECT`.
- Multiple CTEs can be chained: `WITH a AS (...), b AS (...) SELECT ...`
- CTEs can reference earlier CTEs in the same `WITH` clause.
- In MySQL 8.0, CTEs are materialized if referenced multiple times; they may be merged if referenced once.

**Constraints and Limitations:**

- CTE materialization creates a temporary table, consuming memory/temp space.
- If a CTE is referenced only once, materialization may be an unnecessary overhead (optimizer may merge it).
- PostgreSQL 12+ materializes CTEs by default, but `NOT MATERIALIZED` can force merging.

### Annotated Complete Code Examples

**Example 1: CTE vs. View Materialization (MySQL 8.0)**

```sql
-- DBT-3 Query 15 with a view (materialized twice)
-- The view revenue0 is referenced in FROM and in the subquery
SELECT s_suppkey, s_name, total_revenue
FROM supplier, revenue0
WHERE s_suppkey = supplier_no
  AND total_revenue = (SELECT MAX(total_revenue) FROM revenue0);

-- DBT-3 Query 15 with a CTE (materialized once)
WITH revenue0 AS (
    SELECT l_suppkey, SUM(l_extendedprice * (1 - l_discount)) AS total_revenue
    FROM lineitem
    WHERE l_shipdate >= '1996-07-01'
      AND l_shipdate < DATE_ADD('1996-07-01', INTERVAL '90' DAY)
    GROUP BY l_suppkey
)
SELECT s_suppkey, s_name, total_revenue
FROM supplier, revenue0
WHERE s_suppkey = supplier_no
  AND total_revenue = (SELECT MAX(total_revenue) FROM revenue0);
```

**Why this matters:** MySQL 8.0's CTE implementation materializes the CTE once, even when referenced multiple times. The view-based query materializes `revenue0` twice (once for `FROM`, once for the subquery). For DBT-3 Query 15, this cut execution time almost in half because materialization is the most time-consuming part .

**Example 2: CTE for Readability (Equivalent Performance to Subquery)**

```sql
-- Subquery version (readable but nested)
SELECT * FROM TableA A
JOIN (SELECT * FROM TableB WHERE some_column > 5) B ON A.id = B.id AND A.id2 = B.id2;

-- CTE version (same performance, better readability)
WITH FilteredB AS (
    SELECT * FROM TableB WHERE some_column > 5
)
SELECT * FROM TableA A
JOIN FilteredB B ON A.id = B.id AND A.id2 = B.id2;
```

**Why this matters:** For simple cases where the subquery is referenced once, CTEs and subqueries have the same execution cost. CTEs are preferred for readability because they name and separate complex logic .

### Real-World Cases

**Case 1: Multi-Reference Revenue Calculation**

A financial report calculates total revenue for each supplier, then finds the supplier with maximum revenue, and also lists all suppliers with their revenue. Using a CTE for the revenue calculation avoids computing it twice. In MySQL 8.0, this halves execution time .

**Case 2: Recursive Hierarchies**

A CTE with `RECURSIVE` is used to traverse an employee-manager hierarchy. The recursive CTE references itself to walk the tree, a pattern that is difficult to express with subqueries.

### References

- MySQL Developer Zone — MySQL 8.0: Improved performance with CTE - https://dev.mysql.com/blog-archive/mysql-8-0-improved-performance-with-cte/
- GitHub — Optimizing SQL Performance: Advanced Techniques - https://raw.githubusercontent.com/moh1tt/blog/refs/heads/master/data/blog/QueryFaster.mdx

---

## 7. Window Functions Performance

### Definitions

**Core Definition:** Window functions performance optimization involves structuring `PARTITION BY` and `ORDER BY` clauses to minimise sorting overhead and enabling the optimizer to use indexes and batch mode execution.

**Technical Definition:** Window functions (`ROW_NUMBER()`, `RANK()`, `SUM() OVER`, etc.) require data to be sorted by the `PARTITION BY` and `ORDER BY` columns. If an index exists that matches this ordering, the sort can be avoided. The optimizer can group multiple window functions with compatible orderings into a single sort operation, reducing the number of sorts. Batch mode execution (SQL Server 2019+) can dramatically speed up window aggregates.

**Beginner-Friendly Explanation:** Window functions need data in a specific order to work. If the data is already in that order (because of an index), the database doesn't have to sort it. If not, it sorts first—which can be slow for large datasets. Optimizing window functions means giving them the order they need, or reducing the amount of data they have to sort.

### Purposes

- To avoid expensive sort operations by using existing indexes.
- To reduce memory consumption and disk spills during window function sorts.
- To enable batch mode execution for faster window aggregates.
- To group compatible window functions to minimise the number of sorts.

### Syntax Rules and Structure

**Index-Compatible Window Function:**

```sql
-- Window function ordering matches index (CustomerID, OrderDate)
SELECT CustomerID,
       SUM(TotalDue) OVER (PARTITION BY CustomerID ORDER BY OrderDate) AS RunningTotal
FROM Sales.SalesOrderHeader;

-- Index that supports the window function
CREATE INDEX IX_SalesOrderHeader_Customer_OrderDate
    ON Sales.SalesOrderHeader (CustomerID, OrderDate)
    INCLUDE (TotalDue);
```

**Syntax Rules:**

- The `PARTITION BY` and `ORDER BY` in the window function should match the leading columns of an index.
- Multiple window functions with the same `PARTITION BY` and compatible `ORDER BY` can share one sort.
- Batch mode execution is available in SQL Server 2019+ with compatibility level 150+ or on columnstore indexes.

**Constraints and Limitations:**

- If the window ordering does not match any index, a sort is required.
- Underestimated cardinality can cause the sort to request insufficient memory, leading to spills.
- Batch mode cannot be forced; the optimizer chooses it when beneficial.
- The `RANGE` window frame option can complicate optimization compared to `ROWS`.

### Annotated Complete Code Examples

**Example 1: Index-Supported Window Function**

```sql
-- Index: (CustomerID, OrderDate) INCLUDE (TotalDue)
-- Window function ordering matches index

SELECT CustomerID,
       OrderDate,
       TotalDue,
       SUM(TotalDue) OVER (PARTITION BY CustomerID ORDER BY OrderDate) AS RunningTotal
FROM Sales.SalesOrderHeader
ORDER BY CustomerID, OrderDate;

-- The optimizer can use the index to avoid a sort
```

**Why this works:** The index `(CustomerID, OrderDate)` provides the exact ordering required by the window function's `PARTITION BY CustomerID ORDER BY OrderDate`. The optimizer can scan the index in order, avoiding a sort operation entirely. The `INCLUDE (TotalDue)` makes it a covering index, so no table access is needed .

**Example 2: Sort Spill Due to Underestimated Cardinality**

```sql
-- Window function with sort spill in EXPLAIN ANALYZE
EXPLAIN ANALYZE
SELECT CustomerID,
       OrderDate,
       SUM(TotalDue) OVER (PARTITION BY CustomerID ORDER BY OrderDate) AS RunningTotal
FROM Sales.SalesOrderHeader;

-- Plan output (conceptual):
-- WindowAgg  (cost=... rows=... width=...) (actual time=... rows=... loops=1)
--   ->  Sort  (cost=... rows=... width=...) (actual time=... rows=... loops=1)
--         Sort Key: CustomerID, OrderDate
--         Sort Method: external merge  Disk: 7526kB
--         ->  Seq Scan on SalesOrderHeader  (...)
```

**Why this spill occurs:** The `Sort Method: external merge Disk: 7526kB` indicates the sort spilled to disk. Microsoft documentation notes that "underestimated cardinality can cause a sort operation to require more memory during execution," increasing query cost. To reduce spills: ensure statistics are current on partition and order columns, enable Memory Grant Feedback, and reduce input rows with `WHERE` filters .

### Real-World Cases

**Case 1: Running Total Report**

A financial report calculates running totals for each customer. Without an index on `(CustomerID, OrderDate)`, the database sorts the entire table. After creating the index, the sort is eliminated and the report runs 10x faster.

**Case 2: Ranking with Multiple Window Functions**

A query uses `ROW_NUMBER() OVER (PARTITION BY PostalCode ORDER BY SalesYTD DESC)` and `RANK() OVER (PARTITION BY PostalCode ORDER BY SalesYTD DESC)`. Both window functions share the same `PARTITION BY` and `ORDER BY`, so the optimizer performs a single sort for both. Grouping compatible window functions minimizes sort operations .

### References

- Microsoft Learn — OVER Clause (Transact-SQL) - https://learn.microsoft.com/pl-pl/sql/t-sql/queries/select-over-clause-transact-sql
- US Patent 6,389,410 — Window Function Optimization - https://patentimages.storage.googleapis.com/ae/2a/b8/99788215516254/US6389410.pdf
- Darling Data — Window Functions and Sort Spills - https://erikdarling.com/author/erikdarling/page/46/

---

## Summary Table

| Optimization | Key Action | Primary Benefit |
|--------------|------------|-----------------|
| Avoid Unnecessary Columns | Replace `SELECT *` with explicit columns | Reduces I/O, network, memory; enables covering indexes |
| Reduce Unnecessary Rows | Filter in SQL, not application | Reduces data transfer and downstream processing |
| SARGable Predicates | Avoid functions on indexed columns | Enables index usage; avoids full table scans |
| Join Optimization | Use explicit JOIN syntax; correct predicate placement | Avoids Cartesian products; preserves outer join semantics |
| Keyset Pagination | Use `WHERE (col, id) < (:cursor)` instead of `OFFSET` | Constant performance regardless of page depth |
| CTE Rewriting | Use CTEs for reused subqueries (MySQL 8.0+) | Materialises once instead of multiple times |
| Window Function Optimization | Match index ordering; group compatible windows | Avoids sorts; enables batch mode |

---

## References

- Stack Overflow — Reasons to avoid SELECT * in production code - https://stackoverflow.com/revisions/cf16a939-01f1-412a-b01c-fd031e5b4a05/view-source
- SAP SQL Anywhere — Query Predicates - https://help.sap.com/docs/SAP_SQL_Anywhere/e38b2f6217f24bdb90a3ff8ae57b1dd5/818efd706ce21014b529ebb906015a7a.html
- Stacksync — Postgres Pagination: Keyset vs Offset vs Cursor - https://www.stacksync.com/blog/keyset-cursors-postgres-pagination-fast-accurate-scalable
- Microsoft Learn — OVER Clause (Transact-SQL) - https://learn.microsoft.com/pl-pl/sql/t-sql/queries/select-over-clause-transact-sql
- MySQL Developer Zone — MySQL 8.0: Improved performance with CTE - https://dev.mysql.com/blog-archive/mysql-8-0-improved-performance-with-cte/
- Oracle Database Performance Tuning Guide - https://docs.oracle.com/en/database/oracle/oracle-database/26/tgdba/designing-and-developing-for-performance.html
- GitHub — MySQL: Avoid Using SELECT * in Production - https://github.com/OneUptime/blog/commit/21e12e16ff8d640c12eba80ac6ece3465ad401be
- SAP SQL Anywhere — Sargable Predicates (German) - https://help.sap.com/doc/57ec2dbe4d3d4be0a91bfd3eb2c0b3f3/17.0/de-DE/SQL-Anywhere-Server-Users-Guide-de.pdf
- GitHub — Keyset Pagination - https://raw.githubusercontent.com/armourinfosec/Secure-PHP-Development/refs/heads/main/Using-PHP-to-Access-MySQL/Keyset-Pagination.md
- US Patent 6,389,410 — Window Function Optimization - https://patentimages.storage.googleapis.com/ae/2a/b8/99788215516254/US6389410.pdf
- GitHub — Optimizing SQL Performance: Advanced Techniques - https://raw.githubusercontent.com/moh1tt/blog/refs/heads/master/data/blog/QueryFaster.mdx
- Oracle Software Development Kit Developer Guide — Table Indices and SQL - https://docs.oracle.com/en/industries/energy-water/framework/2510/sdk-dev-guide/Topics/SDK_Table_Indices_and_SQL.html
- Microsoft — SELECT * AND SQL Azure - https://learn.microsoft.com/nb-no/archive/blogs/appfabricannounce/select-and-sql-azure
- Sybase SQL Anywhere — Usage Guide (PDF) - https://infocenter.sybase.com/help/topic/com.sybase.help.sqlanywhere.12.0.0/pdf/dbusage12.pdf
- jOOQ Manual — Keyset Pagination / SEEK - https://www.jooq.org/doc/3.14/manual-pdf/jOOQ-manual-3.14.pdf
- US Patent 6,622,138 — Window Function Ordering Groups - http://patentimages.storage.googleapis.com/7f/1c/f5/3ebfcd022f4ffa/US6622138.pdf
- GitHub — Optimizing SQL Performance (CTEs vs Subqueries) - https://github.com/moh1tt/blog/blob/master/data/blog/QueryFaster.mdx
- Oracle Blogs — OCI PostgreSQL Best Practices for Database Performance - https://blogs.oracle.com/cloud-infrastructure/oci-postgresql-best-practices-for-database
- GitHub — Select Only Needed Columns - https://github.com/ravnhq/ai-toolkit/blob/main/skills/database/platform-database/rules/query-select-columns.md
- Darling Data — Window Functions and Sort Spills - https://erikdarling.com/author/erikdarling/page/46/