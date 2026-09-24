# SQL Execution Plan Analysis: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** SQL execution plan analysis is the practice of inspecting, interpreting, and diagnosing the query plans that database optimizers generate for SQL statements, using estimated and actual runtime metrics to identify performance bottlenecks.

**Technical Definition:** An execution plan is a tree-structured representation of the physical operations (plan nodes or operators) that the database engine will perform to execute a SQL statement. Plan analysis involves examining the estimated costs, row counts, access methods, join algorithms, and other operator properties to determine whether the optimizer has chosen an efficient strategy, and identifying mismatches between estimates and reality that indicate missing or stale statistics, suboptimal indexes, or other tuning opportunities.

**Beginner-Friendly Explanation:** An execution plan is like a recipe the database follows to answer your query. It says: "First scan this table, then filter these rows, then join with that table, then sort the result." By looking at this recipe, you can see if the database is doing something silly—like reading a whole table when it could use an index, or using the wrong order to join tables. Execution plan analysis is how you find and fix those problems.

### Key Characteristics

- **Tree-structured:** Plans are hierarchical, with child nodes feeding data to parent nodes; the root node returns the final result.
- **Cost-annotated:** Each node carries estimated startup and total costs, plus estimated row counts.
- **Access-method-explicit:** The plan reveals whether the engine chooses sequential scans, index scans, index seeks, or other access paths.
- **Runtime-observable:** With `EXPLAIN ANALYZE`, the plan includes actual times, row counts, and memory usage.

### Prerequisites

- **Basic SQL knowledge:** Understanding of `SELECT`, `WHERE`, `JOIN`, and subqueries.
- **Index fundamentals:** Knowledge of B-tree indexes, covering indexes, and selectivity.
- **Familiarity with the specific RDBMS:** Syntax and plan output format vary across PostgreSQL, MySQL, SQL Server, and Oracle.

### Related Programming Areas

- **Query Optimization:** Execution plan analysis is the primary diagnostic tool for query tuning.
- **Database Administration:** DBAs use plan analysis to identify missing indexes and stale statistics.
- **Application Development:** Developers use plan analysis to write queries that the optimizer can execute efficiently.

### Core Concepts / Features

1. EXPLAIN: Viewing the Estimated Plan
2. EXPLAIN ANALYZE / SHOW PLAN: Actual Runtime Metrics
3. Query-Plan Operators
4. Estimated versus Actual Rows
5. Access Methods: Sequential Scans versus Index Scans
6. Data Spill to Disk

---

## 1. EXPLAIN: Viewing the Estimated Plan

### Definitions

**Core Definition:** `EXPLAIN` is a command that displays the execution plan the optimizer has chosen for a query, showing estimated costs and row counts without actually executing the statement.

**Technical Definition:** `EXPLAIN` invokes the query planner, generates a plan tree, and outputs a textual or structured description of the plan nodes, their estimated startup and total costs (in arbitrary cost units), estimated rows, and estimated row width. The statement is not executed; no data is read or modified.

**Beginner-Friendly Explanation:** `EXPLAIN` is like asking the database to "show your work" before it actually runs your query. It tells you the recipe it plans to follow, and how much it thinks each step will cost, without actually cooking the meal.

### Purposes

- To inspect the optimizer's chosen execution strategy before running a potentially expensive query.
- To compare alternative query formulations without executing them.
- To identify obvious problems such as full table scans on large tables or nested loop joins with large inputs.
- To understand how the optimizer interprets your query structure.

### Syntax Rules and Structure

**Complete General Syntax (PostgreSQL):**

```sql
EXPLAIN [ ( option [, ...] ) ] statement;
```

Where options include:

| Option | Purpose |
|--------|---------|
| `ANALYZE` | Execute and show actual statistics (default FALSE) |
| `VERBOSE` | Display additional plan details (output columns, schema qualification) |
| `COSTS` | Show estimated costs (default TRUE) |
| `BUFFERS` | Show buffer usage statistics |
| `TIMING` | Show actual times (requires ANALYZE) |
| `FORMAT` | Output format: TEXT, XML, JSON, or YAML |

**Syntax Rules:**

- `EXPLAIN` can be used with `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `MERGE`, `VALUES`, `EXECUTE`, `DECLARE`, `CREATE TABLE AS`, and `CREATE MATERIALIZED VIEW AS`.
- Without `ANALYZE`, the statement is not executed; the plan is only estimated.
- The plan is displayed in a tree structure, with child nodes indented under parent nodes.

**Constraints and Limitations:**

- Estimated costs are in arbitrary units; they are not directly interpretable as seconds or milliseconds.
- Cost estimates depend on the accuracy of statistics; stale statistics produce misleading plans.
- `EXPLAIN` without `ANALYZE` does not reveal runtime surprises such as disk spills.

### Annotated Complete Code Examples

**Example 1: Basic EXPLAIN Output**

```sql
EXPLAIN SELECT * FROM employees WHERE department = 'Engineering';

-- Expected Output:
--                      QUERY PLAN
-- ----------------------------------------------------
--  Seq Scan on employees  (cost=0.00..15.50 rows=5 width=68)
--    Filter: (department = 'Engineering'::text)
```

**Why this output occurs:** The plan shows a single `Seq Scan` node with a `Filter` condition. The estimated cost is 15.50, the estimated rows is 5, and the average row width is 68 bytes. The plan indicates that no index is available (or the optimizer chose not to use one) for `department = 'Engineering'`.

**Example 2: EXPLAIN with Index Scan**

```sql
CREATE INDEX idx_employees_dept ON employees (department);

EXPLAIN SELECT * FROM employees WHERE department = 'Engineering';

-- Expected Output:
--                                QUERY PLAN
-- -------------------------------------------------------------------------
--  Index Scan using idx_employees_dept on employees  (cost=0.29..8.31 rows=5 width=68)
--    Index Cond: (department = 'Engineering'::text)
```

**Why this output occurs:** After creating an index on `department`, the optimizer chooses an `Index Scan` instead of a `Seq Scan`. The cost drops from 15.50 to 8.31, reflecting the reduced I/O from using the index. The `Index Cond` shows the condition used to seek into the index.

### Real-World Cases

**Case 1: Pre-Deployment Plan Review**

A developer writes a complex join query and runs `EXPLAIN` before deploying. The plan reveals a nested loop join with a sequential scan on a 10-million-row table, signaling a potential performance problem. The developer adds an index and re-runs `EXPLAIN` to confirm the plan improves.

**Case 2: Query Formulation Comparison**

Two equivalent queries use different syntax. `EXPLAIN` shows that one uses an index scan while the other triggers a full table scan. The developer chooses the formulation that produces the better plan.

### References

- PostgreSQL Documentation — Using EXPLAIN - https://www.postgresql.org/docs/14/using-explain.html
- PostgreSQL Documentation — EXPLAIN - https://www.postgresql.org/docs/16/sql-explain.html

---

## 2. EXPLAIN ANALYZE / SHOW PLAN: Actual Runtime Metrics

### Definitions

**Core Definition:** `EXPLAIN ANALYZE` executes the query and displays both the estimated plan and the actual runtime statistics, including execution time and row counts for each plan node.

**Technical Definition:** The `ANALYZE` option causes the statement to be actually executed, and the plan output is annotated with actual startup time, actual total time, actual rows, loops, and (with `BUFFERS`) buffer usage. PostgreSQL documentation notes that this adds profiling overhead to query execution, so `EXPLAIN ANALYZE` can sometimes take significantly longer than normal execution.

**Beginner-Friendly Explanation:** `EXPLAIN ANALYZE` is like running the recipe and timing each step. Instead of just telling you what the database *plans* to do, it tells you what it *actually* did, how long each step took, and how many rows each step produced. This is the gold standard for diagnosing real performance issues.

### Purposes

- To compare estimated row counts against actual row counts.
- To identify which plan node consumes the most time.
- To detect disk spills, excessive buffer reads, and other runtime problems.
- To validate that the optimizer's assumptions match reality.

### Syntax Rules and Structure

**Complete General Syntax (PostgreSQL):**

```sql
EXPLAIN ANALYZE SELECT ...;
EXPLAIN (ANALYZE, BUFFERS, TIMING) SELECT ...;
```

**Important Safety Note:** `EXPLAIN ANALYZE` executes the statement. For `INSERT`, `UPDATE`, `DELETE`, or `MERGE`, the modifications will actually happen. To analyze without affecting data, wrap in a transaction and roll back:

```sql
BEGIN;
EXPLAIN ANALYZE UPDATE ...;
ROLLBACK;
```

**Syntax Rules:**

- `ANALYZE` cannot be used with `GENERIC_PLAN`.
- `TIMING` may only be used when `ANALYZE` is enabled.
- `BUFFERS` adds buffer usage statistics, including shared blocks hit/read and temp blocks read/written.

**Constraints and Limitations:**

- Profiling overhead can distort timing, especially for nodes that execute very quickly.
- The statement is actually executed; side effects occur as usual.
- Actual row counts can vary between runs due to concurrent data modifications.

### Annotated Complete Code Examples

**Example 1: EXPLAIN ANALYZE Output**

```sql
EXPLAIN ANALYZE SELECT * FROM employees WHERE department = 'Engineering';

-- Expected Output:
--                                QUERY PLAN
-- -------------------------------------------------------------------------
--  Index Scan using idx_employees_dept on employees  (cost=0.29..8.31 rows=5 width=68) (actual time=0.015..0.018 rows=4 loops=1)
--    Index Cond: (department = 'Engineering'::text)
--  Planning Time: 0.085 ms
--  Execution Time: 0.042 ms
```

**Why this output occurs:** The plan shows both estimated (`cost=0.29..8.31 rows=5`) and actual (`actual time=0.015..0.018 rows=4 loops=1`) statistics. The estimated rows (5) is close to the actual rows (4), indicating accurate statistics. The `Planning Time` and `Execution Time` are shown at the bottom.

**Example 2: Comparing Estimated vs. Actual**

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'pending';

-- Expected Output:
--                                QUERY PLAN
-- -------------------------------------------------------------------------
--  Seq Scan on orders  (cost=0.00..155.00 rows=500 width=52) (actual time=0.020..12.500 rows=45000 loops=1)
--    Filter: (status = 'pending'::text)
--    Rows Removed by Filter: 55000
--  Planning Time: 0.100 ms
--  Execution Time: 15.230 ms
```

**Why this output occurs:** The optimizer estimated 500 rows but the actual count was 45,000—a 90x misestimate. This indicates stale statistics or a skewed distribution. The `Rows Removed by Filter: 55000` shows that 55,000 rows were scanned and discarded. The actual time (12.5 ms) is far higher than the estimate would suggest.

### Real-World Cases

**Case 1: Diagnosing a Slow Query**

A production query takes 8 seconds. `EXPLAIN ANALYZE` reveals that a `Hash Join` node is spilling to disk (`Batches: 8`). Increasing `work_mem` eliminates the spill and reduces execution time to 200 ms.

**Case 2: Validating Statistics Accuracy**

After a bulk data load, a query performs poorly. `EXPLAIN ANALYZE` shows estimated rows = 1 but actual rows = 500,000. Running `ANALYZE` on the table updates statistics, and the optimizer chooses a better plan on the next execution.

### References

- PostgreSQL Documentation — EXPLAIN (ANALYZE) - https://www.postgresql.org/docs/16/sql-explain.html
- PostgreSQL Documentation — Using EXPLAIN (ANALYZE) - https://www.postgresql.org/docs/14/using-explain.html

---

## 3. Query-Plan Operators

### Definitions

**Core Definition:** Query-plan operators (or plan nodes) are the individual processing steps that the execution engine performs, such as scanning a table, filtering rows, sorting, or joining two inputs.

**Technical Definition:** Each operator in a plan tree consumes zero or more input row streams, performs a specific operation (scan, filter, project, join, sort, aggregate, limit), and produces an output row stream for its parent. Operators are typically iterator-based: they support Open, GetNext, and Close operations. Oracle documentation lists common operations including `FILTER`, `SORT`, `HASH JOIN`, `NESTED LOOPS`, `MERGE JOIN`, and `INDEX UNIQUE SCAN`.

**Beginner-Friendly Explanation:** Plan operators are the individual steps in the database's recipe. "Scan this table," "Filter out rows that don't match," "Sort the remaining rows," "Join with that other table." By reading the plan from bottom to top (or inside to outside), you can follow the flow of data.

### Purposes

- To break query execution into discrete, understandable operations.
- To identify expensive operations (sorts, spills, joins with large inputs).
- To understand the order in which operations are performed.
- To diagnose why a particular plan was chosen by the optimizer.

### Syntax Rules and Structure

**Common Operators (PostgreSQL):**

| Operator | Purpose |
|----------|---------|
| `Seq Scan` | Read all rows from a table sequentially |
| `Index Scan` | Traverse a B-tree index and fetch matching rows |
| `Index Only Scan` | Retrieve all needed data from the index alone |
| `Bitmap Heap Scan` | Use a bitmap to fetch rows in physical order |
| `Filter` | Apply a predicate to each row |
| `Sort` | Order rows by specified columns |
| `Hash Join` | Join two inputs by building a hash table |
| `Merge Join` | Join two sorted inputs in a single pass |
| `Nested Loop` | For each outer row, scan the inner input |
| `HashAggregate` | Group rows using a hash table |
| `GroupAggregate` | Group pre-sorted rows |
| `Limit` | Return only the first N rows |

**Common Operators (SQL Server):** SQL Server uses similar operators, including `Clustered Index Scan`, `Clustered Index Seek`, `Hash Match` (hash join), `Merge Join`, `Nested Loops`, and `Sort`.

**Common Operators (Oracle):** Oracle operators include `TABLE ACCESS FULL`, `INDEX RANGE SCAN`, `INDEX UNIQUE SCAN`, `HASH JOIN`, `MERGE JOIN`, `NESTED LOOPS`, `SORT ORDER BY`, and `FILTER`.

**Syntax Rules:**

- Plan trees are read from the innermost (deepest indentation) node outward; the root node is the final operation.
- Each node has a `cost` range (startup cost..total cost) and `rows` estimate.
- `actual` statistics (with `EXPLAIN ANALYZE`) are shown in parentheses after the estimates.

### Annotated Complete Code Examples

**Example 1: Filter, Sort, and Limit Operators**

```sql
EXPLAIN ANALYZE
SELECT first_name, salary
FROM employees
WHERE department = 'Engineering'
ORDER BY salary DESC
LIMIT 3;

-- Expected Output:
--                                    QUERY PLAN
-- -------------------------------------------------------------------------------------
--  Limit  (cost=12.50..12.51 rows=3 width=36) (actual time=0.050..0.051 rows=3 loops=1)
--    ->  Sort  (cost=12.50..12.51 rows=5 width=36) (actual time=0.048..0.048 rows=3 loops=1)
--          Sort Key: salary DESC
--          Sort Method: quicksort  Memory: 25kB
--          ->  Index Scan using idx_employees_dept on employees  (cost=0.29..12.44 rows=5 width=36) (actual time=0.015..0.030 rows=4 loops=1)
--                Index Cond: (department = 'Engineering'::text)
```

**Why this output occurs:** The plan reads from bottom to top:
1. **Index Scan** retrieves rows where `department = 'Engineering'` (4 actual rows).
2. **Sort** orders those rows by `salary DESC` using quicksort in memory.
3. **Limit** returns only the first 3 rows.

The `Sort Method: quicksort Memory: 25kB` indicates the sort fit in memory (no spill).

**Example 2: Hash Join Operator**

```sql
EXPLAIN ANALYZE
SELECT o.order_id, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;

-- Expected Output:
--                                    QUERY PLAN
-- -------------------------------------------------------------------------------------
--  Hash Join  (cost=15.00..35.00 rows=1000 width=36) (actual time=0.100..2.500 rows=1000 loops=1)
--    Hash Cond: (o.customer_id = c.customer_id)
--    ->  Seq Scan on orders o  (cost=0.00..15.00 rows=1000 width=8) (actual time=0.010..0.500 rows=1000 loops=1)
--    ->  Hash  (cost=10.00..10.00 rows=100 width=28) (actual time=0.080..0.080 rows=100 loops=1)
--          Buckets: 1024  Batches: 1  Memory Usage: 9kB
--          ->  Seq Scan on customers c  (cost=0.00..10.00 rows=100 width=28) (actual time=0.005..0.040 rows=100 loops=1)
```

**Why this output occurs:** The `Hash` node builds a hash table from `customers` (the smaller input). The `Hash Join` node then scans `orders` and probes the hash table for matches. `Batches: 1` indicates the hash table fit in memory; `Memory Usage: 9kB` shows the size.

### Real-World Cases

**Case 1: Identifying a Sort That Spills**

A query with `ORDER BY` shows `Sort Method: external merge Disk: 7526kB` in the plan. This indicates the sort spilled to disk. Increasing `work_mem` allows the sort to complete in memory, dramatically improving performance.

**Case 2: Recognizing a Nested Loop on Large Inputs**

A query plan shows `Nested Loop` with a `Seq Scan` on a 1-million-row table as the inner input. This means the inner table is scanned once per outer row, producing catastrophic performance. The fix is to add an index on the join column or force a hash join.

### References

- Oracle Database — EXPLAIN PLAN Operations - https://docs.oracle.com/cd/F25597_01/document/products/iserver/oracle8/80/generic/a56814-1.pdf
- PostgreSQL Documentation — Using EXPLAIN - https://www.postgresql.org/docs/14/using-explain.html
- Microsoft SQL Server — Showplan Operators Reference - https://learn.microsoft.com/he-il/sql/relational-databases/showplan-logical-and-physical-operators-reference

---

## 4. Estimated versus Actual Rows

### Definitions

**Core Definition:** The comparison of estimated row counts (from the optimizer's statistics) against actual row counts (observed during execution) reveals cardinality estimation errors that often lead to poor plan choices.

**Technical Definition:** The optimizer computes estimated rows using statistics and selectivity formulas. During `EXPLAIN ANALYZE`, the execution engine reports actual rows returned by each node. A large discrepancy indicates that the statistics are stale, missing, or unrepresentative of the current data distribution, leading the optimizer to choose a suboptimal plan.

**Beginner-Friendly Explanation:** The optimizer guesses how many rows each step will produce. If it guesses 10 but the step actually produces 100,000, the plan was built on a bad assumption. Comparing estimated vs. actual rows is how you find those bad guesses.

### Purposes

- To detect cardinality estimation errors that cause suboptimal plans.
- To identify tables that need `ANALYZE` to refresh statistics.
- To spot data skew where uniform distribution assumptions fail.
- To validate that the optimizer's cost model reflects reality.

### Syntax Rules and Structure

**In the Plan Output:**

```
Seq Scan on orders  (cost=0.00..155.00 rows=500 width=52) (actual time=0.020..12.500 rows=45000 loops=1)
```

| Field | Meaning |
|-------|---------|
| `rows=500` | Estimated rows (optimizer's guess) |
| `actual ... rows=45000` | Actual rows returned |
| `Rows Removed by Filter` | Rows scanned but discarded by the filter |

**SQL Server:** The execution plan shows `Estimated Number of Rows` and `Actual Number of Rows`. SQL Server 2016 SP1 added `Estimated Number of Rows to be Read` and `Number of Rows Read`, which reveal how many rows were read before predicates were applied.

**Syntax Rules:**

- A ratio of actual/estimated > 10 or < 0.1 often indicates a problem worth investigating.
- Misestimates compound across joins; an error in one node affects all parent nodes.
- Statistics should be refreshed after significant data changes (`ANALYZE table_name` in PostgreSQL, `UPDATE STATISTICS` in SQL Server).

### Annotated Complete Code Examples

**Example 1: Large Misestimate**

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'pending';

-- Expected Output:
--                                QUERY PLAN
-- -------------------------------------------------------------------------
--  Seq Scan on orders  (cost=0.00..155.00 rows=500 width=52) (actual time=0.020..12.500 rows=45000 loops=1)
--    Filter: (status = 'pending'::text)
--    Rows Removed by Filter: 55000
--  Planning Time: 0.100 ms
--  Execution Time: 15.230 ms
```

**Why this output occurs:** The optimizer estimated 500 rows but actually got 45,000. This 90x misestimate suggests that the statistics for `status` are stale or that the data is heavily skewed (e.g., 45% of orders are 'pending'). Running `ANALYZE orders` would update the statistics and potentially change the plan.

**Example 2: Accurate Estimate**

```sql
EXPLAIN ANALYZE SELECT * FROM employees WHERE employee_id = 100;

-- Expected Output:
--                                QUERY PLAN
-- -------------------------------------------------------------------------
--  Index Scan using employees_pkey on employees  (cost=0.29..8.30 rows=1 width=68) (actual time=0.010..0.011 rows=1 loops=1)
--    Index Cond: (employee_id = 100)
```

**Why this output occurs:** The estimated rows (1) matches the actual rows (1). Primary key lookups are typically accurate because the optimizer knows the column is unique. When estimates match reality, the plan is likely well-chosen.

### Real-World Cases

**Case 1: Stale Statistics After Bulk Load**

A nightly ETL job inserts 1 million rows into a table. The next morning, queries perform poorly. `EXPLAIN ANALYZE` shows estimated rows = 1,000 but actual rows = 500,000. The DBA runs `ANALYZE` on the table, and the optimizer immediately chooses a better plan.

**Case 2: Correlated Predicates**

A query filters on `WHERE city = 'Springfield' AND state = 'IL'`. The optimizer assumes the predicates are independent and estimates 100 rows. But because Springfield is a common city name, the actual rows are 50,000. This is a classic cardinality misestimate caused by correlated columns.

### References

- Microsoft SQL Server — Predicate Pushdown and Estimated vs Actual Rows - https://learn.microsoft.com/ja-jp/archive/blogs/blogdoezequiel/predicate-pushdown-and-why-should-i-care
- PostgreSQL Documentation — Statistics Used by the Planner - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.2/postgresql-9.2-US.pdf

---

## 5. Access Methods: Sequential Scans versus Index Scans

### Definitions

**Core Definition:** Access methods are the techniques the execution engine uses to read rows from a table, primarily sequential scans (reading all rows) and index scans (using an index to locate specific rows).

**Technical Definition:** A sequential scan reads every data block in a table, evaluating predicates against each row. An index scan traverses a B-tree index to find row identifiers, then fetches the corresponding rows from the table (unless the index covers the query). The optimizer chooses between them based on estimated cost, which depends on selectivity, table size, index clustering, and the cost parameters.

**Beginner-Friendly Explanation:** A sequential scan is like reading a whole book to find one quote. An index scan is like using the book's index to find the right page. The optimizer chooses based on whether the book is small (read it all) or the quote is rare (use the index).

### Purposes

- To understand why the optimizer chose a particular access method.
- To identify when a sequential scan is appropriate (e.g., selecting most rows).
- To recognize when a missing index is forcing an inefficient sequential scan.
- To evaluate whether an index is being used effectively.

### Syntax Rules and Structure

**Access Method Indicators:**

| Plan Node | Access Method |
|-----------|---------------|
| `Seq Scan` | Sequential table scan |
| `Index Scan` | Index traversal + table fetch |
| `Index Only Scan` | Index traversal only (covering index) |
| `Bitmap Heap Scan` | Bitmap index + table fetch in physical order |
| `Clustered Index Scan` (SQL Server) | Sequential scan of clustered index |
| `Clustered Index Seek` (SQL Server) | Seek in clustered index |

**Cost Factors:**

| Factor | Sequential Scan | Index Scan |
|--------|----------------|------------|
| Selectivity | High (many rows match) | Low (few rows match) |
| Table size | Small tables | Large tables |
| Index clustering | N/A | High clustering factor (rows clustered by index) |
| Covering | N/A | Index contains all needed columns |

**Syntax Rules:**

- PostgreSQL documentation notes that index scans are not always faster; if a query visits most rows, a sequential scan is chosen.
- The optimizer's choice depends on `random_page_cost` and `seq_page_cost` parameters.
- On SSDs, `random_page_cost` is often lowered (e.g., to 1.0), making index scans more attractive.

### Annotated Complete Code Examples

**Example 1: Sequential Scan Chosen (Low Selectivity)**

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'completed';

-- Expected Output:
--                                QUERY PLAN
-- -------------------------------------------------------------------------
--  Seq Scan on orders  (cost=0.00..155.00 rows=45000 width=52) (actual time=0.020..12.500 rows=45000 loops=1)
--    Filter: (status = 'completed'::text)
```

**Why this output occurs:** The predicate `status = 'completed'` matches 45,000 out of 100,000 rows (45%). The optimizer determines that a sequential scan is cheaper than an index scan because using an index would require reading many table pages in random order, while the sequential scan reads the table in one pass.

**Example 2: Index Scan Chosen (High Selectivity)**

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE order_id = 12345;

-- Expected Output:
--                                QUERY PLAN
-- -------------------------------------------------------------------------
--  Index Scan using orders_pkey on orders  (cost=0.29..8.30 rows=1 width=52) (actual time=0.010..0.011 rows=1 loops=1)
--    Index Cond: (order_id = 12345)
```

**Why this output occurs:** The primary key lookup matches exactly one row. The index scan traverses the B-tree in logarithmic time and fetches one table row. A sequential scan would read all 100,000 rows—far more expensive. The optimizer correctly chooses the index.

### Real-World Cases

**Case 1: Planner Choosing Sequential Scan on a Large Table**

A PostgreSQL user reports that the planner prefers a sequential scan on a 6-million-row table, even though an index exists. The query returns a large date range, and the planner estimates that the index would visit too many rows, causing excessive random I/O. Setting `enable_seqscan = 0` forces an index scan, which runs faster in this case, suggesting that `random_page_cost` may need adjustment for SSD storage.

**Case 2: Covering Index Enables Index-Only Scan**

A query selects `(customer_id, order_date, status)` from `orders WHERE customer_id = 12345`. A covering index on `(customer_id) INCLUDE (order_date, status)` allows an `Index Only Scan`, which reads only the index and never touches the table. This eliminates the "random I/O" penalty of index scans and makes the index scan cheaper than a sequential scan.

### References

- PostgreSQL Mailing List — Index usage with sub select or inner joins - https://www.postgresql.org/message-id/20081112135405.GD22032%40polonium.part.net
- PostgreSQL Mailing List — seq vs index scan in join query - https://www.postgresql.org/message-id/CAPfMJ90rmEdCXQ0ox9umUMs6%3DoVfomv9nc9V-bcsdLtZAJgszg%40mail.gmail.com
- PostgreSQL Documentation — Using EXPLAIN - https://www.postgresql.org/docs/14/using-explain.html

---

## 6. Data Spill to Disk

### Definitions

**Core Definition:** Data spill (or disk spill) occurs when a memory-intensive operation—such as a sort, hash join, or hash aggregate—exceeds its allocated memory budget and writes temporary data to disk.

**Technical Definition:** Operations like `Sort` and `Hash Join` are allocated a limited amount of working memory (PostgreSQL: `work_mem`; SQL Server: memory grant; AnalyticDB: `statement_mem`). If the operation's input exceeds this budget, the database spills intermediate results to temporary files on disk. This is called an "external" operation, as opposed to an in-memory operation.

**Beginner-Friendly Explanation:** Sorting and hashing need memory. If there's not enough memory, the database writes part of the data to disk and processes it in chunks. Disk is much slower than memory, so spilling makes queries slower.

### Purposes

- To allow large sorts and hashes to complete without exhausting all memory.
- To detect when memory allocation is too low for the workload.
- To identify queries that need tuning (larger `work_mem`, better indexes, or query restructuring).
- To prevent out-of-memory errors on the database server.

### Syntax Rules and Structure

**Detection in PostgreSQL:**

```
Sort Method: external merge  Disk: 7526kB
```

This line appears in `EXPLAIN ANALYZE` output when a sort spills.

**Detection in AnalyticDB for PostgreSQL:**

Check the `Workfile` field in `EXPLAIN ANALYZE` output:
- Positive integer — spill occurred
- 0 — no spill

**Relevant Parameters:**

| RDBMS | Parameter | Purpose |
|-------|-----------|---------|
| PostgreSQL | `work_mem` | Memory per sort/hash operation |
| PostgreSQL | `temp_file_limit` | Maximum temporary file size per session |
| SQL Server | Memory grant | Memory allocated to a query |
| AnalyticDB | `statement_mem` | Memory per statement |

**Syntax Rules:**

- `work_mem` is per-operation, not per-query. A query with multiple sorts can use multiple `work_mem` allocations.
- Increasing `work_mem` reduces spills but consumes more server memory; setting it too high can cause memory pressure.
- `temp_file_limit` caps temporary file size; exceeding it cancels the query.

**Constraints and Limitations:**

- Spilling is not always avoidable; some queries legitimately need more memory than available.
- Increasing `work_mem` globally can cause memory pressure under high concurrency.
- Hash joins may spill in multiple "batches"; `Batches: 8` means the hash table was divided into 8 chunks, processed in multiple passes.

### Annotated Complete Code Examples

**Example 1: Sort Spill**

```sql
EXPLAIN ANALYZE SELECT * FROM large_table ORDER BY some_column;

-- Expected Output:
--                                QUERY PLAN
-- -------------------------------------------------------------------------
--  Sort  (cost=1000.00..2000.00 rows=100000 width=100) (actual time=500.000..800.000 rows=100000 loops=1)
--    Sort Key: some_column
--    Sort Method: external merge  Disk: 7526kB
--    ->  Seq Scan on large_table  (cost=0.00..500.00 rows=100000 width=100) (actual time=0.020..100.000 rows=100000 loops=1)
```

**Why this output occurs:** The sort operation required more memory than `work_mem` allowed, so it spilled 7,526 kB to disk. The `external merge` method means the data was sorted in chunks and merged from disk. Increasing `work_mem` to at least 8 MB would allow the sort to complete in memory.

**Example 2: Hash Join Spill**

```sql
EXPLAIN ANALYZE
SELECT * FROM orders o JOIN customers c ON o.customer_id = c.customer_id;

-- Expected Output (conceptual):
--                                QUERY PLAN
-- -------------------------------------------------------------------------
--  Hash Join  (cost=500.00..2000.00 rows=100000 width=100) (actual time=2000.000..5000.000 rows=100000 loops=1)
--    Hash Cond: (o.customer_id = c.customer_id)
--    ->  Seq Scan on orders o  (cost=0.00..500.00 rows=100000 width=52)
--    ->  Hash  (cost=300.00..300.00 rows=50000 width=48) (actual time=1000.000..1000.000 rows=50000 loops=1)
--          Buckets: 65536  Batches: 8  Memory Usage: 4096kB
--          ->  Seq Scan on customers c  (cost=0.00..300.00 rows=50000 width=48)
```

**Why this output occurs:** The `Hash` node's `Batches: 8` indicates the hash table was divided into 8 batches, meaning it spilled to disk. `Memory Usage: 4096kB` shows the in-memory portion. The hash join processed the batches in multiple passes, significantly increasing execution time. Increasing `work_mem` or reducing the input size (via better filtering) would reduce or eliminate the spill.

### Real-World Cases

**Case 1: Increasing work_mem to Eliminate Spill**

A PostgreSQL query takes 30 seconds because a sort spills to disk. `EXPLAIN ANALYZE` shows `Disk: 64MB`. The DBA increases `work_mem` from 4MB to 128MB for the session, and the query completes in 2 seconds without spilling.

**Case 2: Data Skew Causing Spill**

An AnalyticDB query spills to disk even with a large `statement_mem`. The root cause is data skew: one join key value appears millions of times, concentrating all that data on one compute node. The fix is to identify and resolve the skew (e.g., by filtering or redistributing data).

### References

- Alibaba Cloud — What do I do if the "ERROR: temporary file size exceeds temp_file_limit" error message is displayed? - https://help.aliyun.com/en/rds/support/error-temporary-file-size-exceeds-temp-file-limit-1
- Alibaba Cloud — Avoid spilling to disk - https://www.alibabacloud.com/help/en/analyticdb/analyticdb-for-postgresql/user-guide/disk-spill-prevention
- PostgreSQL Documentation — EXPLAIN (BUFFERS) - https://www.postgresql.org/docs/16/sql-explain.html

---

## Summary Table

| Concept | Command / Indicator | Key Insight |
|---------|---------------------|-------------|
| EXPLAIN | `EXPLAIN SELECT ...` | Shows estimated plan without executing |
| EXPLAIN ANALYZE | `EXPLAIN ANALYZE SELECT ...` | Shows actual runtime metrics |
| Plan Operators | `Seq Scan`, `Hash Join`, `Sort` | The individual steps in the recipe |
| Estimated vs Actual | `rows=500` vs `actual rows=45000` | Misestimates indicate stale statistics |
| Access Methods | `Seq Scan` vs `Index Scan` | Depends on selectivity and cost |
| Data Spill | `Sort Method: external merge Disk: ...` | Operation exceeded memory budget |

---

## References

- PostgreSQL Documentation — Using EXPLAIN - https://www.postgresql.org/docs/14/using-explain.html
- PostgreSQL Documentation — EXPLAIN - https://www.postgresql.org/docs/16/sql-explain.html
- PostgreSQL Documentation — EXPLAIN (9.5) - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.5/postgresql-9.5-A4.pdf
- Oracle Database — EXPLAIN PLAN Operations - https://docs.oracle.com/cd/F25597_01/document/products/iserver/oracle8/80/generic/a56814-1.pdf
- Microsoft SQL Server — Showplan Operators Reference - https://learn.microsoft.com/he-il/sql/relational-databases/showplan-logical-and-physical-operators-reference
- Microsoft SQL Server — Predicate Pushdown and Estimated vs Actual Rows - https://learn.microsoft.com/ja-jp/archive/blogs/blogdoezequiel/predicate-pushdown-and-why-should-i-care
- PostgreSQL Mailing List — Index usage with sub select or inner joins - https://www.postgresql.org/message-id/20081112135405.GD22032%40polonium.part.net
- PostgreSQL Mailing List — seq vs index scan in join query - https://www.postgresql.org/message-id/CAPfMJ90rmEdCXQ0ox9umUMs6%3DoVfomv9nc9V-bcsdLtZAJgszg%40mail.gmail.com
- Alibaba Cloud — temp_file_limit error - https://help.aliyun.com/en/rds/support/error-temporary-file-size-exceeds-temp-file-limit-1
- Alibaba Cloud — Avoid spilling to disk - https://www.alibabacloud.com/help/en/analyticdb/analyticdb-for-postgresql/user-guide/disk-spill-prevention
- PostgreSQL Documentation — Statistics Used by the Planner - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.2/postgresql-9.2-US.pdf
- PostgreSQL Manpages — EXPLAIN(7) - https://manpages.debian.org/bookworm/postgresql-client-15/EXPLAIN.7
- Amazon AWS — EXPLAIN Operators for Redshift Query Plans - https://docs.aws.amazon.com/fr_fr/prescriptive-guidance/latest/query-lifecycle-redshift/explain-operators.html