# Comprehensive Programming Cheat Sheet: SQL Removing Duplicates

## Topic Overview

### Definitions

**Core Definition**
Removing duplicates is the process of eliminating repeated rows from a query result or a table, using SQL constructs such as `DISTINCT`, `DISTINCT ON`, `GROUP BY`, window functions, or `EXISTS` to retain only one representative row per unique value or combination of values.

**Technical Definition**
Duplicate removal in SQL operates on the level of **complete rows** (or a defined subset of columns). `SELECT DISTINCT` compares entire rows across the select list and retains one row per unique combination of values. `DISTINCT ON (expression, ...)` — a PostgreSQL extension — retains the **first row** of each group defined by the specified expressions, where “first” is determined by the `ORDER BY` clause. `GROUP BY` achieves similar effects by grouping rows and applying aggregate functions. Window functions (`ROW_NUMBER()`, `RANK()`) assign a rank within each partition, enabling the retention of a specific row (e.g., the latest) per group. `EXISTS`/`NOT EXISTS` filter rows based on the existence of a related row, avoiding the need to deduplicate a joined result set. Each approach has distinct performance characteristics, index requirements, and semantic nuances.

**Beginner-Friendly Explanation**
Duplicate removal is about getting a clean list without repeats. Imagine a table of orders where the same customer appears many times. If you just want the list of customers, you need to remove the repeats. `DISTINCT` does that by looking at the entire row you selected and keeping only one copy of each unique combination. `DISTINCT ON` is more precise: it lets you keep one specific row per customer—say, the latest order—based on how you sort. `GROUP BY` is another way to deduplicate, and it also lets you calculate totals or counts per group. Window functions and `EXISTS` are more advanced tools for keeping exactly the right row when duplicates differ in other columns.

### Key Characteristics

- **`DISTINCT` operates on the entire select list**: `SELECT DISTINCT a, b` removes duplicate **(a, b)** pairs, not duplicate `a` values independently. Adding more columns to the select list makes the DISTINCT operation more expensive and can produce results that differ from what a user expecting “unique a” intended.
- **`DISTINCT ON` is PostgreSQL-specific and order-sensitive**: The `DISTINCT ON` expressions must match the **leftmost `ORDER BY` expressions**, and the `ORDER BY` determines which row is kept from each group. This makes `DISTINCT ON` the ideal tool for “latest/first row per group” queries.
- **Indexes are the primary performance lever**: If the columns used for deduplication are covered by an index, the database can often read unique values directly from the index without a separate sort or hash step, dramatically improving performance. Missing or mismatched indexes force temporary tables and filesorts.
- **`DISTINCT` must process the full result set before returning any rows**: Unlike `EXISTS`, which can short-circuit after finding a single match, `DISTINCT` cannot know a row is unique until it has seen all rows (or built a hash table). This “full-set-then-deduplicate” behaviour is the root of its performance cost on large data sets.
- **Alternatives are often faster and more semantically precise**: `GROUP BY` can leverage index-based grouping; `EXISTS` avoids deduplication entirely by short-circuiting; `ROW_NUMBER()` allows keeping a precisely chosen row per group without the order-sensitivity constraints of `DISTINCT ON`.

### Prerequisites

- A running RDBMS (PostgreSQL, MySQL, or SQL Server) and a client connection.
- At least one table with duplicate data, or the ability to construct duplicates with `UNION ALL` or a `VALUES` list.
- Familiarity with `SELECT`, `FROM`, `WHERE`, and `ORDER BY`.
- Awareness of indexes on the columns involved in deduplication, as performance depends heavily on index availability.
- For PostgreSQL-specific features: knowledge that `DISTINCT ON` is an extension of the SQL standard and is not available in MySQL or SQL Server.

### Related Programming Areas

- Query optimisation – indexes, execution plans (`EXPLAIN`), and the choice between sort-based and hash-based deduplication.
- Data cleansing – identifying and removing duplicate records in ETL pipelines.
- Analytics – computing unique counts (`COUNT(DISTINCT ...)`) and distinct lists for reporting.
- Pagination and result shaping – removing duplicates introduced by joins before paginating.
- SQL dialect portability – `DISTINCT ON` (PostgreSQL) vs. `ROW_NUMBER()` (standard) vs. `GROUP BY` (portable).

---

## Core Concepts and Features

### 1. `DISTINCT`

#### Definitions

**Core Definition**
`SELECT DISTINCT` removes duplicate rows from the result set, retaining one row for each unique combination of values in the select list.

**Technical Definition**
`SELECT DISTINCT select_list` evaluates the select list for every candidate row and then eliminates rows whose entire select-list values duplicate an earlier row. Internally, the database may implement this with a **sort-based** approach (sort the result, then skip adjacent duplicates) or a **hash-based** approach (build a hash table keyed on the select-list values, keeping the first occurrence). Null values are considered equal for the purpose of duplicate elimination. `DISTINCT` applies to the **entire row** produced by the select list, so `SELECT DISTINCT a, b` removes duplicate (a, b) pairs, not duplicate `a` values independently.

**Beginner-Friendly Explanation**
`DISTINCT` is the simplest way to say “give me each unique row only once.” If you select two columns, it looks at the pair of values together; if you select one column, it looks at that column alone.

#### Purposes

- To produce a list of unique values from one or more columns.
- To remove duplicate rows introduced by joins or denormalised data.
- To compute unique counts with `COUNT(DISTINCT column)`.
- To prepare a distinct result set for downstream processing or display.

#### Syntax Rules and Structure

```sql
SELECT DISTINCT select_list
FROM table_expression
[WHERE ...]
[ORDER BY ...];
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `DISTINCT` | Applies to the entire select list. |
| `select_list` | The columns or expressions whose combined values must be unique. |

**Syntax rules:**

- `DISTINCT` must appear immediately after `SELECT`.
- It applies to the whole row, not to individual columns.
- `DISTINCT` can be combined with `ORDER BY`; the `ORDER BY` expressions must appear in the select list when `DISTINCT` is used (in standard SQL).

**Constraints and limitations:**

- `DISTINCT` on many columns is expensive because the uniqueness check involves all of them.
- `DISTINCT` cannot be used with `DISTINCT ON` in the same query.
- `DISTINCT` requires the database to process the full candidate set before it can guarantee uniqueness.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: DISTINCT on a single column

```sql
-- Step 1: Create a table with duplicate values.
CREATE TABLE orders (
    order_id   int PRIMARY KEY,
    customer_id int NOT NULL,
    total       numeric(10,2)
);

INSERT INTO orders VALUES
    (1, 100, 50.00),
    (2, 100, 75.00),
    (3, 200, 30.00),
    (4, 100, 50.00);

-- Step 2: Retrieve the distinct customer IDs.
SELECT DISTINCT customer_id
FROM orders;
```

**Expected output:**

```text
 customer_id
-------------
         100
         200
(2 rows)
```

**Why this output appears**: The `orders` table contains three rows for customer 100 and one row for customer 200. `DISTINCT customer_id` collapses the three 100s into one and returns 100 and 200, each once.

##### Example 2: DISTINCT on multiple columns

```sql
-- Step 1: Retrieve distinct (customer_id, total) pairs.
SELECT DISTINCT customer_id, total
FROM orders;
```

**Expected output:**

```text
 customer_id | total
-------------+-------
         100 | 50.00
         100 | 75.00
         200 | 30.00
(3 rows)
```

**Why this output appears**: Although customer 100 appears three times, the **(customer_id, total)** pairs are (100, 50.00), (100, 75.00), and (100, 50.00). The pair (100, 50.00) appears twice and is deduplicated, leaving three unique pairs.

#### Real-World Cases

- **Unique customer list**: `SELECT DISTINCT customer_id FROM orders` produces the list of customers who have placed orders.
- **Unique product categories**: `SELECT DISTINCT category FROM products` produces the list of categories for a filter menu.

---

### 2. `DISTINCT ON` (PostgreSQL)

#### Definitions

**Core Definition**
`DISTINCT ON (expression, ...)` is a PostgreSQL extension that retains the **first row** of each group of rows that share the same values in the specified expressions, where “first” is determined by the `ORDER BY` clause.

**Technical Definition**
The syntax is `SELECT DISTINCT ON (expression [, ...]) select_list FROM ... ORDER BY expression [, ...] [, ...]`. The `DISTINCT ON` expressions must match the **leftmost `ORDER BY` expressions**. The `ORDER BY` may contain additional expressions that determine which row is kept within each group. `DISTINCT ON` is semantically equivalent to `GROUP BY` on the specified expressions, but it allows selecting arbitrary non-aggregated columns from the chosen row—something `GROUP BY` does not permit without an aggregate function. Null values are considered equal for the purpose of grouping.

**Beginner-Friendly Explanation**
`DISTINCT ON` is the “keep the top row of each group” tool. If you sort each customer’s orders by date descending, `DISTINCT ON (customer_id)` keeps the most recent order for each customer and discards the rest. It is the cleanest way to answer “latest row per group” questions in PostgreSQL.

#### Purposes

- To retrieve the latest, first, highest, or otherwise preferred row per group.
- To deduplicate while controlling **which** row survives, unlike plain `DISTINCT` which keeps an arbitrary one.
- To replace more verbose `ROW_NUMBER()` window-function subqueries with a simpler, often faster query.
- To select non-aggregated columns from the chosen row without violating `GROUP BY` rules.

#### Syntax Rules and Structure

```sql
SELECT DISTINCT ON (expression [, ...]) select_list
FROM table_expression
[WHERE ...]
ORDER BY expression [, ...] [, additional_sort_expressions];
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `DISTINCT ON (expression, ...)` | The grouping key for deduplication. |
| `ORDER BY` | Must begin with the `DISTINCT ON` expressions; additional expressions determine which row is kept. |

**Syntax rules:**

- The `DISTINCT ON` expressions must match the **leftmost** `ORDER BY` expressions.
- Additional `ORDER BY` expressions (after the grouping key) control row selection within each group.
- The `SELECT` list may contain columns not present in `DISTINCT ON` or `ORDER BY`.

**Constraints and limitations:**

- PostgreSQL-specific; not available in MySQL or SQL Server.
- The `ORDER BY` constraint means you cannot sort the final result by a different column without wrapping the query in a subquery.
- `DISTINCT ON` still processes the full set of candidate rows; it reduces the output but not the input scan.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Latest order per customer

```sql
-- Step 1: Create a table with multiple orders per customer.
CREATE TABLE purchases (
    id       int PRIMARY KEY,
    customer text NOT NULL,
    total    numeric(10,2) NOT NULL
);

INSERT INTO purchases VALUES
    (1, 'Alice', 100.00),
    (2, 'Alice', 150.00),
    (3, 'Bob',   200.00),
    (4, 'Bob',   120.00);

-- Step 2: Keep the highest-total purchase per customer.
SELECT DISTINCT ON (customer) customer, id, total
FROM purchases
ORDER BY customer, total DESC, id;
```

**Expected output:**

```text
 customer | id | total
----------+----+--------
 Alice    |  2 | 150.00
 Bob      |  3 | 200.00
(2 rows)
```

**Why this output appears**: `DISTINCT ON (customer)` groups by customer. The `ORDER BY customer, total DESC, id` sorts each customer’s rows by total descending and uses `id` as a tiebreaker. The first row in each group—the one with the highest total—is retained. Alice’s 150 and Bob’s 200 survive.

##### Example 2: DISTINCT ON with an index

```sql
-- Step 1: Create an index matching the DISTINCT ON query.
CREATE INDEX purchases_customer_total_idx ON purchases (customer, total DESC, id);

-- Step 2: Run the same query; the index can provide the required order.
SELECT DISTINCT ON (customer) customer, id, total
FROM purchases
ORDER BY customer, total DESC, id;
```

**Expected output:** (same as above, but with better performance.)

**Why this output appears**: The index `(customer, total DESC, id)` matches the `ORDER BY` clause exactly, allowing PostgreSQL to read rows in the required order without a separate sort step. This is the “perfect index” pattern for `DISTINCT ON` queries.

#### Real-World Cases

- **Latest device reading**: `DISTINCT ON (device_id) ... ORDER BY device_id, reported_at DESC` retrieves the most recent report per device.
- **First purchase per customer**: `DISTINCT ON (customer_id) ... ORDER BY customer_id, created_at ASC` retrieves the earliest order per customer.
- **Cheapest supplier per product**: `DISTINCT ON (product_id) ... ORDER BY product_id, price ASC` retrieves the lowest-priced supplier offer per product.

---

### 3. Multiple Columns

#### Definitions

**Core Definition**
Duplicate removal on multiple columns considers the **combination** of values across all specified columns, retaining one row per unique tuple.

**Technical Definition**
When `DISTINCT` or `DISTINCT ON` is applied to multiple columns, two rows are considered duplicates only if they agree on **every** column in the deduplication key. If any column differs, both rows are retained. This means that adding more columns to the deduplication key generally increases the number of distinct rows and decreases the number of duplicates removed. For `DISTINCT`, the key is the entire select list. For `DISTINCT ON`, the key is the expressions inside the parentheses.

**Beginner-Friendly Explanation**
Deduplicating on multiple columns means “remove rows only if they match on **all** of these columns.” If you deduplicate on (first_name, last_name), two people named “John Smith” are treated as duplicates, even if they have different email addresses—unless you include email in the deduplication key.

#### Purposes

- To define a composite natural key for deduplication (e.g., `(order_id, line_no)`).
- To prevent over-deduplication that would occur on a single low-cardinality column.
- To retain the full row context while deduplicating on a meaningful subset of columns.

#### Syntax Rules and Structure

**DISTINCT on multiple columns:**

```sql
SELECT DISTINCT col1, col2, col3 FROM table;
```

**DISTINCT ON multiple columns:**

```sql
SELECT DISTINCT ON (col1, col2) col1, col2, col3, col4
FROM table
ORDER BY col1, col2, col3 DESC;
```

**Syntax rules:**

- For `DISTINCT`, the entire select list is the deduplication key.
- For `DISTINCT ON`, only the parenthesised expressions form the key.
- Null values are considered equal for duplicate elimination.

**Constraints and limitations:**

- `DISTINCT` on many columns requires comparing more data and can be significantly more expensive than `DISTINCT` on a single indexed column.
- If the goal is to deduplicate on a single logical key but you need additional columns, `DISTINCT ON` or `ROW_NUMBER()` is usually more appropriate than `DISTINCT` on many columns.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Comparing single-column and multi-column DISTINCT

```sql
-- Step 1: Create a table with duplicates on one column but not on two.
CREATE TABLE events (
    event_id   int,
    user_id    int,
    event_type text
);

INSERT INTO events VALUES
    (1, 100, 'click'),
    (2, 100, 'view'),
    (3, 100, 'click'),
    (4, 200, 'view');

-- Step 2: DISTINCT on one column.
SELECT DISTINCT user_id FROM events;

-- Step 3: DISTINCT on two columns.
SELECT DISTINCT user_id, event_type FROM events;
```

**Expected output:**

```text
-- Step 2:
 user_id
---------
     100
     200
(2 rows)

-- Step 3:
 user_id | event_type
---------+------------
     100 | click
     100 | view
     200 | view
(3 rows)
```

**Why this output appears**: `DISTINCT user_id` collapses all of user 100’s events into one row. `DISTINCT user_id, event_type` treats (100, 'click'), (100, 'view'), and (100, 'click') as two unique pairs: (100, 'click') and (100, 'view'), because the (100, 'click') pair appears twice and is deduplicated.

#### Real-World Cases

- **Unique (customer, product) pairs**: `SELECT DISTINCT customer_id, product_id FROM purchases` finds each customer–product combination that has been purchased.
- **Deduplicating on a composite natural key**: `SELECT DISTINCT order_id, line_no FROM order_items` removes accidental duplicate line entries.

---

### 4. Performance Considerations

#### Definitions

**Core Definition**
Performance considerations for duplicate removal are the factors that determine how efficiently `DISTINCT` and related operations execute, including indexing, sort vs. hash strategy, column count, and the size of the candidate set.

**Technical Definition**
Duplicate removal requires the database to compare rows and identify duplicates. The two primary strategies are **sort-based** (sort the rows, then skip adjacent duplicates) and **hash-based** (build a hash table keyed on the deduplication columns, keeping the first occurrence). Both require processing the full candidate set before returning any rows. Performance is dominated by: (1) whether an index covers the deduplication columns and provides the required order; (2) the width of the deduplication key (more columns = more comparison work); (3) the size of the candidate set; and (4) whether the query also requires a sort (`ORDER BY`) that is not satisfied by the index. When `EXPLAIN` shows `Using temporary` or `Using filesort` alongside a `DISTINCT` query, the database is performing an extra deduplication pass that could often be avoided with better indexing or a different query formulation.

**Beginner-Friendly Explanation**
Duplicate removal is expensive because the database must look at every row before it can know which rows are unique. The best way to make it fast is to give it an index that already contains the columns it needs in the right order. If the index matches the deduplication key and any required sort order, the database can read unique values directly from the index without building a temporary table or sorting. If the index does not match, the database has to fall back to sorting or hashing all the rows, which is much slower on large data sets.

#### Purposes

- To understand why a `DISTINCT` query is slow and how to fix it.
- To choose between `DISTINCT`, `GROUP BY`, and `EXISTS` based on performance characteristics.
- To design indexes that support efficient deduplication.
- To avoid the common pitfalls that force temporary tables and filesorts.

#### Syntax Rules and Structure

Performance is influenced by query design and index design, not by a single syntax:

| Factor | Impact | Mitigation |
|--------|--------|------------|
| No index on dedup columns | Full scan + sort/hash | Add an index on the dedup columns |
| Multi-column dedup with wrong index order | Cannot use index ordering | Match the index order to the dedup column order |
| Select list wider than the index | Table lookups (“回表”) inflate the dedup set | Use a covering index |
| `DISTINCT` + `ORDER BY` on different columns | Extra sort after dedup | Align `ORDER BY` with the dedup key or index |
| `DISTINCT` with `JOIN` | Processes all joined rows before dedup | Rewrite with `EXISTS` or `GROUP BY` |

**Key EXPLAIN warning signs:**

| Warning | Meaning | Fix |
|---------|---------|-----|
| `Using filesort` | Extra sort for dedup or ORDER BY | Add an index covering the dedup columns |
| `Using temporary` | Temporary table for dedup | Add an index or rewrite with `EXISTS`/`GROUP BY` |
| `type: ALL` | Full table scan | Add a `WHERE` clause or index |

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Index removes filesort and temporary table

```sql
-- Step 1: A DISTINCT query without an index.
EXPLAIN SELECT DISTINCT customer_id FROM orders;

-- Step 2: Add an index on the DISTINCT column.
CREATE INDEX idx_orders_customer ON orders (customer_id);

-- Step 3: Re-run EXPLAIN; the index should eliminate filesort.
EXPLAIN SELECT DISTINCT customer_id FROM orders;
```

**Expected output (illustrative):**

```text
-- Before index:
Seq Scan on orders  (cost=... rows=...)  -- full scan
-- After index:
Index Only Scan using idx_orders_customer on orders  (cost=... rows=...)  -- no sort
```

**Why this output appears**: Without an index, MySQL must scan all rows and sort or hash them to find distinct values. With an index on `customer_id`, MySQL can read the unique values directly from the index in sorted order, eliminating both the full scan and the sort step.

##### Example 2: DISTINCT vs. EXISTS performance for a join dedup

```sql
-- Slow: DISTINCT over a join.
SELECT DISTINCT c.id, c.name
FROM customers c
JOIN orders o ON c.id = o.customer_id;

-- Faster: EXISTS short-circuits after the first match.
SELECT c.id, c.name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.id
);
```

**Expected output:** (both return customers who have at least one order, but `EXISTS` stops scanning after the first matching order per customer.)

**Why this output appears**: The `DISTINCT` query joins every customer to every order, producing a large intermediate result that must then be deduplicated. The `EXISTS` query, with an index on `orders.customer_id`, checks each customer once and stops as soon as it finds a single matching order—avoiding the large intermediate result and the deduplication step entirely.

#### Real-World Cases

- **Reporting on unique customers**: An index on `orders(customer_id)` turns a slow full-table-scan-and-sort into a fast index-only scan.
- **Deduplicating joined results**: Replacing `SELECT DISTINCT` over a join with `WHERE EXISTS` reduces logical reads by over 80% in million-row benchmarks.

---

### 5. Alternatives to DISTINCT

#### Definitions

**Core Definition**
Alternatives to `DISTINCT` are other SQL constructs—`GROUP BY`, `DISTINCT ON` (PostgreSQL), `ROW_NUMBER()`/`RANK()` window functions, and `EXISTS`/`NOT EXISTS`—that achieve deduplication or its equivalent with different semantics and performance characteristics.

**Technical Definition**
Each alternative addresses a different deduplication need. **`GROUP BY`** groups rows by key and requires aggregate functions for non-grouped columns; it is often more index-friendly than `DISTINCT` and can compute aggregates in the same pass. **`DISTINCT ON`** (PostgreSQL) keeps a specific row per group determined by `ORDER BY`. **`ROW_NUMBER()`** assigns a sequential number within each partition and is the standard-SQL way to keep a specific row per group, compatible with MySQL 8.0+ and SQL Server. **`EXISTS`** avoids deduplication entirely by testing for the existence of a related row, using short-circuit evaluation. **`IN` with a subquery** is another option but is generally less efficient than `EXISTS` for correlated checks.

**Beginner-Friendly Explanation**
`DISTINCT` is not the only way to remove duplicates, and it is often not the best way. `GROUP BY` is a close cousin that is sometimes faster and also lets you calculate totals. `DISTINCT ON` is the PostgreSQL tool for “keep the latest row per group.” `ROW_NUMBER()` is the portable version of the same idea. And `EXISTS` is the trick for “does this row have a match?”—it skips the whole deduplication problem by stopping as soon as it finds one.

#### Purposes

- To choose the most efficient deduplication technique for the data volume and query shape.
- To retain a specific row per group (latest, highest, first) rather than an arbitrary one.
- To avoid the performance cost of deduplicating large joined result sets.
- To write deduplication queries that are portable across PostgreSQL, MySQL, and SQL Server.

#### Syntax Rules and Structure

**`GROUP BY`:**

```sql
SELECT customer_id
FROM orders
GROUP BY customer_id;
```

**`ROW_NUMBER()` (standard SQL, MySQL 8.0+, SQL Server, PostgreSQL):**

```sql
SELECT id, customer_id, total
FROM (
    SELECT id, customer_id, total,
           ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY total DESC, id) AS rn
    FROM purchases
) t
WHERE rn = 1;
```

**`EXISTS`:**

```sql
SELECT c.id, c.name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.id
);
```

**`DISTINCT ON` (PostgreSQL):**

```sql
SELECT DISTINCT ON (customer_id) id, customer_id, total
FROM purchases
ORDER BY customer_id, total DESC, id;
```

**Component breakdown:**

| Alternative | Best for | Portability |
|-------------|----------|-------------|
| `GROUP BY` | Dedup + aggregation; simple unique lists | Standard SQL |
| `ROW_NUMBER()` | Keep a specific row per group | Standard SQL (modern RDBMSs) |
| `DISTINCT ON` | Keep a specific row per group (concise) | PostgreSQL only |
| `EXISTS` | Filtering based on related row existence | Standard SQL |
| `IN` (subquery) | Simple membership tests (small subqueries) | Standard SQL |

**Syntax rules:**

- `GROUP BY` requires all non-aggregated select-list columns to appear in the grouping clause.
- `ROW_NUMBER()` must be used in a subquery or CTE; the outer query filters on the row number.
- `EXISTS` ignores the subquery’s select list; `SELECT 1` is conventional.

**Constraints and limitations:**

- `GROUP BY` cannot return non-aggregated columns not in the grouping key.
- `ROW_NUMBER()` can be more verbose than `DISTINCT ON` but is portable.
- `EXISTS` requires an index on the correlated column to avoid full scans.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: GROUP BY as a DISTINCT alternative

```sql
-- Step 1: DISTINCT.
SELECT DISTINCT customer_id FROM orders;

-- Step 2: Equivalent GROUP BY.
SELECT customer_id FROM orders GROUP BY customer_id;
```

**Expected output:** (identical results; `GROUP BY` may use an index more efficiently in some cases.)

**Why this output appears**: Both queries produce the same unique customer list. In MySQL, the optimizer often rewrites `DISTINCT` as `GROUP BY` internally, and `GROUP BY` can leverage index-based grouping to avoid a sort step.

##### Example 2: ROW_NUMBER() versus DISTINCT ON

```sql
-- DISTINCT ON (PostgreSQL only):
SELECT DISTINCT ON (customer) customer, id, total
FROM purchases
ORDER BY customer, total DESC, id;

-- ROW_NUMBER() (standard, portable):
SELECT customer, id, total
FROM (
    SELECT customer, id, total,
           ROW_NUMBER() OVER (PARTITION BY customer ORDER BY total DESC, id) AS rn
    FROM purchases
) t
WHERE rn = 1;
```

**Expected output:** (identical results—the highest-total purchase per customer.)

**Why this output appears**: `DISTINCT ON` keeps the first row of each customer group according to the `ORDER BY`. `ROW_NUMBER()` assigns a number to each row within the customer partition and filters for `rn = 1`. Both achieve the same result; `ROW_NUMBER()` is more verbose but works on MySQL 8.0+ and SQL Server as well as PostgreSQL.

#### Real-World Cases

- **Unique lists**: `SELECT customer_id FROM orders GROUP BY customer_id` is a portable alternative to `SELECT DISTINCT customer_id`.
- **Latest row per group**: Use `DISTINCT ON` in PostgreSQL for conciseness, or `ROW_NUMBER()` for portability to MySQL and SQL Server.
- **Existence filtering**: Use `EXISTS` instead of `SELECT DISTINCT` over a join when the goal is “customers who have at least one order.”

---

## References

- PostgreSQL Documentation – SELECT (DISTINCT, DISTINCT ON) – https://www.postgresql.org/docs/current/sql-select.html#SQL-DISTINCT
- PostgreSQL Documentation – SELECT Lists (DISTINCT Clause) – https://www.postgresql.org/docs/current/queries-select-lists.html#QUERIES-DISTINCT
- Stack Overflow – Fastest way to get distinct rows in PostgreSQL (DISTINCT ON and index) – https://stackoverflow.com/revisions/a98d8c2b-8359-44af-b7e8-31e7f7fd2cb3/view-source
- Stack Overflow – DISTINCT ON vs ROW_NUMBER() for latest row per group – https://stackoverflow.com/revisions/7630564/52
- OneUptime – How to Optimize DISTINCT Queries in MySQL – https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-optimize-distinct-queries/README.md
- 天翼云 – MySQL DISTINCT 去重引发的 filesort 问题及三种解决方案 – https://www.ctyun.cn/developer/article/814436835082309
- 天翼云 – 用 EXISTS 替代 DISTINCT：子查询去重的性能边界在哪里？ – https://www.ctyun.cn/developer/article/814437003505733
- Educative – SQL Deduplication Techniques for Filtering Duplicate Records – https://www.educative.io/courses/sql-interview-patterns/lta/deduplication
- 华为云 – PostgreSQL DISTINCT ON 取每设备最新上报：索引与执行计划对比 – https://bbs.huaweicloud.com/blogs/513665
- Tiger Data – PostgreSQL DISTINCT – https://www.tigerdata.com/learn/postgresql-distinct
- Erik Darling – The SQL Server Performance Tasting Menu: How DISTINCT Queries Can Hurt Performance – https://erikdarling.com/the-sql-server-performance-tasting-menu-how-distinct-queries-can-hurt-performance/