# Comprehensive Programming Cheat Sheet: Join Optimization

## Topic Overview

### Definitions

**Core Definition**: Join optimization is the process by which a database query optimizer selects the most efficient strategy for combining rows from two or more tables, encompassing the choice of join algorithm, join order, and the management of memory and I/O resources required to execute the join.

**Technical Definition**: Join optimization operates within the cost-based optimizer. Given a query with N relations, the optimizer enumerates alternative join trees (join orders) and, for each join operator, considers alternative physical algorithms—nested loop, hash join, and merge join—along with access paths (sequential scans, index scans). It estimates the cost of each candidate plan using statistics about data distribution, cardinality, and available memory, and selects the plan with the lowest estimated cost. The optimization must respect semantic constraints (e.g., outer join ordering restrictions), memory limits (`work_mem` in PostgreSQL, memory grants in SQL Server), and the possibility of spill to disk when memory is insufficient.

**Beginner-Friendly Explanation**: Join optimization is the database's strategy for combining tables without doing unnecessary work. Imagine you need to match every order to its customer. You could read every order and look up each customer one by one (nested loop), or you could build a lookup table of customers in memory and scan orders against it (hash join), or you could sort both lists and walk through them together (merge join). Join optimization is how the database decides which of these approaches will be fastest for your specific data—and it also decides the order in which to combine multiple tables.

### Key Characteristics

- **Cost-based, not rule-based**: The optimizer uses statistics to estimate the cost of alternatives; it does not follow fixed rules like "always use hash join."
- **Algorithm-agnostic at the SQL level**: You write declarative joins; the optimizer chooses the physical algorithm. Hints can override this choice, but they are not part of standard SQL.
- **Join order search space grows exponentially**: For N tables, there are N! possible join orders (ignoring associativity/commutativity); exhaustive search is only feasible for small N, so engines use dynamic programming (≤ ~12 tables) or genetic algorithms (PostgreSQL GEQO) for larger queries.
- **Memory-sensitive**: Hash joins require memory for the hash table; merge joins require memory for sorting; nested loops require memory only for buffers. Insufficient memory causes spill to disk, degrading performance by orders of magnitude.
- **Outer joins constrain reordering**: `LEFT`/`RIGHT`/`FULL OUTER JOIN` cannot be freely reordered because null-extension semantics must be preserved.
- **Elimination and reordering are rewrites, not just algorithm choices**: Removing unnecessary joins and reordering inner joins are logical rewrites that happen before physical algorithm selection.

### Prerequisites

- Understanding of `SELECT`, `JOIN` syntax, and join predicates.
- Familiarity with indexes, statistics, and `EXPLAIN`/execution plans.
- Knowledge of the three join algorithms at a conceptual level.
- Awareness of the target engine's memory parameters (`work_mem`, memory grants).

### Related Programming Areas

- Query optimization and execution plan analysis.
- Index design and statistics maintenance.
- Database performance tuning and capacity planning.
- Query rewriting and subquery unnesting.
- Parallel query execution and adaptive query processing.

---

## Core Concepts and Features

### 1. Join Order

#### Definitions

**Core Definition**: Join order is the sequence in which the optimizer combines the tables in a multi-table query, determining which tables are joined first and which intermediate results feed subsequent joins.

**Technical Definition**: For an N-way join, the optimizer constructs a join tree where each internal node represents a join between two inputs (base tables or intermediate results). Inner joins are commutative and associative, so the optimizer can consider many equivalent orders. The search space is bounded by `N!` if all orders are considered, but the optimizer prunes using cost estimates. PostgreSQL uses exhaustive dynamic programming when the number of relations is below `geqo_threshold` (default 12) and switches to the genetic algorithm GEQO for larger queries. SQL Server and Oracle use similar search strategies with their own thresholds.

**Beginner-Friendly Explanation**: If you join three tables A, B, and C, you can join A to B first and then join the result to C, or join B to C first and then join A to the result. The final answer is the same, but one order might be much faster because it produces smaller intermediate results or uses indexes better.

#### Purposes

- To minimize the size of intermediate results by joining selective tables first.
- To enable the use of indexes on join keys when the smaller table drives the join.
- To reduce the number of rows that must be processed by later joins, sorts, and aggregations.
- To stay within the optimizer's search-space limits while finding a near-optimal plan.

#### Syntax Rules and Structure

**Explicit join order (when hints or `join_collapse_limit` are used)**:

```sql
-- PostgreSQL: prevent reordering by setting join_collapse_limit = 1
SET join_collapse_limit = 1;
SELECT ...
FROM small_selective a
JOIN large_fact b ON b.a_id = a.id
JOIN dimension c ON c.id = b.c_id;
```

**Cost-based reordering (default behavior)**:

```sql
-- The optimizer is free to choose any inner-join order.
SELECT ...
FROM large_fact b
JOIN small_selective a ON b.a_id = a.id
JOIN dimension c ON c.id = b.c_id;
```

**Component breakdown**:

| Component | Description |
|-----------|-------------|
| Base relations | Tables or views at the leaves of the join tree. |
| Join tree | Binary tree of join operators. |
| Search strategy | Exhaustive DP (small N) or GEQO/genetic (large N). |
| Cost model | Estimated I/O, CPU, and memory for each candidate. |

**Syntax rules**:

- Inner joins are commutative and associative; the optimizer can reorder them freely.
- Outer joins constrain ordering: the preserved side must be identified correctly.
- `join_collapse_limit = 1` forces PostgreSQL to use the explicit join order written in the query.
- SQL Server's `FORCE ORDER` hint forces the join order specified in the query.

**Constraints and limitations**:

- Exhaustive search is only feasible for small numbers of tables; larger queries use heuristic search and may miss the optimal order.
- Join order interacts with join algorithm choice: a hash join may be optimal for one order and a nested loop for another.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Forcing join order with `join_collapse_limit`

```sql
-- Step 1: Set the limit to 1 to prevent reordering.
SET join_collapse_limit = 1;

-- Step 2: Write joins in the desired order (selective table first).
EXPLAIN ANALYZE
SELECT ...
FROM small_selective s
JOIN large_fact f ON f.s_id = s.id
JOIN dimension d ON d.id = f.d_id;
```

**Expected output** (plan excerpt):

```text
Nested Loop
  ->  Seq Scan on small_selective s
  ->  Nested Loop
        ->  Index Scan using large_fact_s_id on large_fact f
        ->  Index Scan using dimension_pkey on dimension d
```

**Why this output appears**: With `join_collapse_limit = 1`, PostgreSQL preserves the explicit order: `small_selective` drives the join, and the nested loop uses indexes on `large_fact` and `dimension` for each outer row. If the optimizer had been free to reorder, it might have chosen a hash join with a different order.

#### Real-World Cases

- **Star-schema queries**: Joining a large fact table to several dimension tables; the optimizer should join the most selective dimensions first.
- **OLTP queries**: Small result sets benefit from nested loops with the selective table as the outer input.
- **Reporting queries**: Large joins benefit from hash joins with build-side selection based on cardinality estimates.

---

### 2. Join Algorithms

#### Definitions

**Core Definition**: Join algorithms are the physical methods used to execute a join: nested loop (including index nested loop), hash join, and merge join (sort-merge join).

**Technical Definition**: **Nested loop join** scans the outer input and, for each row, scans or seeks the inner input for matches; with an index on the inner join key, it becomes an index nested loop join, which is efficient when the outer input is small. **Hash join** builds a hash table on the smaller input (the build side) using the join keys, then probes the hash table with rows from the larger input (the probe side); it is efficient for equi-joins on large unsorted inputs but requires memory. **Merge join** sorts both inputs on the join keys (or uses existing sorted indexes) and merges them in a single pass; it is efficient when both inputs are large and presorted, and it produces sorted output.

**Beginner-Friendly Explanation**: A nested loop is like looking up each word in a dictionary one at a time. A hash join is like building a phone book (hash table) from one list and then scanning the other list, checking each entry against the phone book. A merge join is like sorting two decks of cards and then walking through them side by side to find matches.

#### Purposes

- To choose the algorithm that minimizes total I/O and CPU for the given data sizes, indexes, and sort order.
- To avoid unnecessary sorting when a hash join can handle unsorted inputs.
- To exploit indexes when the outer input is small and the inner input is indexed.
- To produce sorted output when the query requires ordering (merge join).

#### Syntax Rules and Structure

**Nested loop**:

```text
Nested Loop
  ->  outer_input (small)
  ->  inner_input (indexed; loops = outer_rows)
```

**Hash join**:

```text
Hash Join
  Hash Cond: (outer.key = inner.key)
  ->  probe_input (large)
  ->  Hash
        ->  build_input (small)
```

**Merge join**:

```text
Merge Join
  Merge Cond: (outer.key = inner.key)
  ->  outer_input (sorted by key)
  ->  inner_input (sorted by key)
```

**Component breakdown**:

| Algorithm | Requires index? | Requires sort? | Memory risk | Best for |
|-----------|----------------|----------------|-------------|----------|
| Nested Loop | Inner side index | No | Low | Small outer + indexed inner |
| Hash Join | No | No | High (spill) | Large unsorted equi-joins |
| Merge Join | No (but indexes help) | Yes, unless presorted | Sort may spill | Large presorted inputs or ordered output |

**Syntax rules**:

- All three algorithms support equi-joins; nested loop and merge join also support non-equi joins (range predicates) with varying efficiency.
- Hash join requires at least one equi-join condition.
- Merge join requires both inputs sorted on the join keys; sorts may be explicit or provided by indexes.
- PostgreSQL, MySQL, SQL Server, and Oracle implement all three algorithms (MySQL uses hash join for equi-joins without indexes and has deprecated block nested loop).

**Constraints and limitations**:

- Hash join cannot handle non-equi joins efficiently.
- Merge join's sort cost may exceed hash join's build cost for large unsorted inputs.
- Nested loop with a large outer input and no inner index is catastrophic.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Nested loop with index lookup

```sql
EXPLAIN ANALYZE
SELECT o.id, c.email
FROM orders o
JOIN customers c ON c.id = o.customer_id
WHERE o.customer_id = 42;
```

**Expected output** (plan excerpt):

```text
Nested Loop
  ->  Index Scan using idx_orders_customer on orders o
        Index Cond: (customer_id = 42)
  ->  Index Scan using customers_pkey on customers c
        Index Cond: (id = o.customer_id)
        loops=3
```

**Why this output appears**: The outer input produces only 3 orders, and for each order, the inner `customers` table is accessed via primary key index lookup. The nested loop is efficient because the outer set is tiny and the inner side is indexed.

##### Example 2: MySQL 8.0 – Hash join for equi-join without indexes

```sql
EXPLAIN ANALYZE
SELECT * FROM t1 JOIN t2 ON t1.c1 = t2.c1;
```

**Expected output** (plan excerpt):

```text
-> Inner hash join (t1.c1 = t2.c1)
    -> Table scan on t1
    -> Hash
        -> Table scan on t2
```

**Why this output appears**: MySQL 8.0.18+ uses hash join for equi-joins when no index is available on the join columns. It builds a hash table on `t2` and probes it with `t1`, avoiding the block nested loop algorithm used in older versions.

#### Real-World Cases

- **OLTP point lookups**: Nested loop with index seeks.
- **Analytical joins**: Hash joins for large fact-to-dimension joins.
- **Ordered reporting**: Merge joins to avoid an extra sort.

---

### 3. Join Hints

#### Definitions

**Core Definition**: Join hints are vendor-specific directives that instruct the optimizer to use a specific join algorithm or join order, overriding its cost-based choice.

**Technical Definition**: Join hints are non-standard SQL extensions. SQL Server provides `LOOP JOIN`, `HASH JOIN`, `MERGE JOIN`, and `FORCE ORDER` in the `OPTION` clause. Oracle provides `USE_NL`, `USE_HASH`, `USE_MERGE`, and `LEADING`/`ORDERED` hints in `/*+ ... */` comments. MySQL provides `BNL`/`NO_BNL` hints for block nested loop control. PostgreSQL does not provide direct join algorithm hints in core; `join_collapse_limit = 1` controls join order, and extensions like `pg_hint_plan` add hint functionality.

**Beginner-Friendly Explanation**: Hints are like telling the database "I know better than you—use this join method." They are useful when the optimizer makes a bad choice due to stale statistics or complex predicates, but they can backfire if data changes and the hinted plan becomes suboptimal.

#### Purposes

- To force a specific join algorithm when the optimizer's choice is demonstrably suboptimal.
- To pin a join order when the optimizer's reordering produces a worse plan.
- To work around cardinality estimation errors or parameter sniffing.
- To stabilize plans for critical queries across deployments.

#### Syntax Rules and Structure

**SQL Server**:

```sql
SELECT ...
FROM t1
JOIN t2 ON t1.id = t2.id
OPTION (HASH JOIN, FORCE ORDER);
```

**Oracle**:

```sql
SELECT /*+ USE_HASH(t1 t2) LEADING(t1 t2) */ ...
FROM t1, t2
WHERE t1.id = t2.id;
```

**MySQL**:

```sql
SELECT /*+ NO_BNL(t1 t2) */ ...
FROM t1 JOIN t2 ON t1.id = t2.id;
```

**Component breakdown**:

| Hint | Engine | Effect |
|------|--------|--------|
| `LOOP JOIN` / `USE_NL` | SQL Server / Oracle | Force nested loop |
| `HASH JOIN` / `USE_HASH` | SQL Server / Oracle | Force hash join |
| `MERGE JOIN` / `USE_MERGE` | SQL Server / Oracle | Force merge join |
| `FORCE ORDER` / `ORDERED` | SQL Server / Oracle | Use the written join order |
| `NO_BNL` | MySQL | Disable block nested loop (use hash join) |

**Syntax rules**:

- Hints are suggestions; the optimizer may ignore them if the hinted plan is invalid or impossible.
- SQL Server hints in `OPTION` apply to the entire query; Oracle hints apply to the specified tables.
- Hints can be combined but may conflict; the optimizer resolves conflicts according to engine rules.

**Constraints and limitations**:

- Hints are not portable across engines.
- Overusing hints can prevent the optimizer from adapting to changing data and statistics.
- Hints do not guarantee performance; they only constrain the plan shape.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Forcing a hash join and join order

```sql
SELECT ...
FROM small_table s
JOIN large_table l ON l.s_id = s.id
OPTION (HASH JOIN, FORCE ORDER);
```

**Expected output** (plan excerpt):

```text
Hash Match (Inner Join)
  Hash Cond: (l.s_id = s.id)
  ->  Table Scan on l
  ->  Hash
        ->  Table Scan on s
```

**Why this output appears**: `HASH JOIN` forces a hash join, and `FORCE ORDER` preserves the written order (`small_table` as build side, `large_table` as probe side). Without hints, the optimizer might have chosen a nested loop or a different order.

#### Real-World Cases

- **Parameter sniffing**: A cached plan optimized for one parameter value performs poorly for another; a hint can stabilize the plan.
- **Complex reporting queries**: The optimizer's cost estimates are inaccurate due to correlated predicates; a hint forces the known-good plan.
- **Migration stabilization**: Hints ensure plan consistency across database versions.

---

### 4. Join Elimination

#### Definitions

**Core Definition**: Join elimination is a query rewrite that removes a join operation when the joined table does not contribute to the query result and cannot change the number or identity of rows produced.

**Technical Definition**: Join elimination applies when the joined table's columns are not referenced in the `SELECT` list, `WHERE`, `GROUP BY`, or `HAVING`, and the join is **key-preserving**—the join column on the eliminated side has a unique constraint, or the join cannot multiply rows. **Outer join elimination** is a special case: a `LEFT JOIN` can be eliminated when the null-supplying table's columns are unused and the join is provably cardinality-preserving.

**Beginner-Friendly Explanation**: If you join orders to a regions table just to filter on region but never select any region columns, and each region has exactly one row, the join is unnecessary—the database can remove it and give the same answer faster.

#### Purposes

- To avoid reading and joining a table that cannot affect the result.
- To reduce plan complexity and execution time.
- To eliminate unnecessary I/O and hash table builds.
- To simplify queries generated by ORMs or reporting tools that join more tables than necessary.

#### Syntax Rules and Structure

**Eliminable inner join**:

```sql
-- Before: join to regions, but no region columns used
SELECT o.id, o.total
FROM orders o
JOIN regions r ON r.id = o.region_id
WHERE o.total > 100;

-- After: join eliminated (r.id is unique)
SELECT o.id, o.total
FROM orders o
WHERE o.total > 100;
```

**Eliminable outer join**:

```sql
-- Before: LEFT JOIN to t3, but t3 columns unused and t3.key is unique
SELECT t1.id, t2.value
FROM t1
JOIN t2 ON t2.t1_id = t1.id
LEFT JOIN t3 ON t3.t2_id = t2.id;

-- After: t3 join eliminated
SELECT t1.id, t2.value
FROM t1
JOIN t2 ON t2.t1_id = t1.id;
```

**Component breakdown**:

| Condition | Requirement |
|-----------|-------------|
| Joined table columns unused | Not in `SELECT`, `WHERE`, `GROUP BY`, `HAVING` |
| Join is key-preserving | Unique key on join column, or `DISTINCT` in subquery |
| Outer join elimination | Null-supplying side unused and no row multiplication |

**Syntax rules**:

- Join elimination is safe only if the join cannot duplicate rows. A non-unique join column may multiply rows even if its columns are unused.
- Outer join elimination may require converting to an inner join first (via null-rejection analysis).
- PostgreSQL's `reduce_outer_joins()` and `join_is_removable()` implement these rewrites.

**Constraints and limitations**:

- Joins with non-unique keys cannot be eliminated without changing row counts.
- Joins involving volatile functions or lateral references may be irreducible.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Inner join elimination

```sql
EXPLAIN
SELECT o.id, o.total
FROM orders o
JOIN regions r ON r.id = o.region_id
WHERE o.total > 100;
```

**Expected output** (plan excerpt):

```text
Seq Scan on orders o
  Filter: (total > 100)
```

**Why this output appears**: `regions` is joined on its unique key (`r.id`), and no `regions` columns are used. The optimizer eliminates the join, removing the hash table build and join operation entirely.

#### Real-World Cases

- **ORM-generated joins**: ORMs often join tables for relationships that are not referenced in the query result.
- **Star-schema queries**: Dimension tables that are filtered but not projected can sometimes be eliminated.
- **View merging**: Views with joins to lookup tables are simplified when the lookup columns are unused.

---

### 5. Join Reordering

#### Definitions

**Core Definition**: Join reordering is the optimizer's process of changing the sequence in which inner joins are evaluated, subject to semantic constraints, to find a lower-cost join order.

**Technical Definition**: Because inner joins are commutative and associative, the optimizer can reorder them freely. PostgreSQL explores join orders exhaustively via dynamic programming when the number of relations is below `geqo_threshold` (default 12); above that, it uses the genetic algorithm GEQO. `join_collapse_limit = 1` disables reordering of explicit `JOIN` syntax. SQL Server and Oracle use their own search strategies and provide `FORCE ORDER`/`ORDERED` hints to disable reordering.

**Beginner-Friendly Explanation**: The optimizer tries different orders of joining your tables to see which one produces the smallest intermediate results. It is like trying different routes on a map to find the fastest one.

#### Purposes

- To minimize intermediate result sizes by joining selective tables early.
- To enable index usage when a small table drives the join.
- To reduce the number of rows processed by subsequent operations.
- To adapt to changing data distributions and statistics.

#### Syntax Rules and Structure

**Disabling reordering (PostgreSQL)** :

```sql
SET join_collapse_limit = 1;
-- The written join order is preserved.
```

**Forcing a join order (SQL Server)** :

```sql
SELECT ...
FROM t1 JOIN t2 ON ... JOIN t3 ON ...
OPTION (FORCE ORDER);
```

**Component breakdown**:

| Control | Engine | Effect |
|---------|--------|--------|
| `join_collapse_limit = 1` | PostgreSQL | No reordering of explicit JOINs |
| `FORCE ORDER` | SQL Server | Use written join order |
| `ORDERED` hint | Oracle | Use written join order |
| `geqo_threshold` | PostgreSQL | Switch to genetic search above N relations |

**Syntax rules**:

- Reordering is safe for inner joins but constrained by outer joins.
- Reordering interacts with algorithm choice: a hash join may be optimal for one order and a nested loop for another.
- `join_collapse_limit` also affects subquery pull-up; setting it to 1 disables some optimizations.

**Constraints and limitations**:

- Exhaustive search is exponential; large queries use heuristic search and may miss the optimal order.
- Forcing join order can prevent the optimizer from adapting to better indexes or statistics.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Comparing reordered vs forced join order

```sql
-- Default: optimizer reorders.
EXPLAIN
SELECT ...
FROM large_fact f
JOIN small_dim s ON s.id = f.s_id
JOIN medium_dim m ON m.id = f.m_id;

-- Forced: written order preserved.
SET join_collapse_limit = 1;
EXPLAIN
SELECT ...
FROM large_fact f
JOIN small_dim s ON s.id = f.s_id
JOIN medium_dim m ON m.id = f.m_id;
```

**Expected output** (default plan, illustrative):

```text
Hash Join
  Hash Cond: (f.m_id = m.id)
  ->  Hash Join
        Hash Cond: (f.s_id = s.id)
        ->  Seq Scan on large_fact f
        ->  Hash
              ->  Seq Scan on small_dim s
  ->  Hash
        ->  Seq Scan on medium_dim m
```

**Expected output** (forced plan, illustrative):

```text
Hash Join
  Hash Cond: (s.id = f.s_id)
  ->  Nested Loop
        ->  Seq Scan on large_fact f
        ->  Index Scan on medium_dim m
  ->  Hash
        ->  Seq Scan on small_dim s
```

**Why this output appears**: The default optimizer reorders to join `large_fact` with `small_dim` first, producing a smaller intermediate result before joining `medium_dim`. With `join_collapse_limit = 1`, the written order is forced, which may or may not be optimal depending on data distribution.

#### Real-World Cases

- **Star-schema queries**: The optimizer joins the most selective dimension first to reduce fact-table rows early.
- **OLTP queries**: Reordering ensures the small result set drives the join.
- **Plan stability**: `join_collapse_limit = 1` pins the plan for critical queries.

---

### 6. Hash Join vs. Merge Join

#### Definitions

**Core Definition**: Hash join builds a hash table on one input and probes it with the other; merge join sorts both inputs on the join keys and merges them in a single pass.

**Technical Definition**: **Hash join** has a build phase (scan the smaller input, insert into a hash table) and a probe phase (scan the larger input, look up each row in the hash table). It requires memory proportional to the build-side size and does not require sorted input. **Merge join** requires both inputs sorted on the join keys; it scans both inputs in parallel, advancing the pointer on the side with the smaller key, and combines matching rows. It produces sorted output and is efficient when inputs are presorted or when sorted output is needed. SQL Server documentation notes that merge join and hash join are both required for optimal decision-support performance: when both inputs are large and similar in size, merge join with prior sorting and hash join offer similar performance; when input sizes differ significantly, hash join is often much faster.

**Beginner-Friendly Explanation**: A hash join is like building a dictionary from the smaller list and then scanning the larger list, looking up each item. A merge join is like sorting both lists alphabetically and then walking through them together. Hash join is better when one list is much smaller than the other; merge join is better when both lists are similar in size and already sorted—or when you need the output sorted anyway.

#### Purposes

- To choose the algorithm that minimizes total cost given input sizes, sort order, and memory.
- To avoid sorting when a hash join can handle unsorted inputs.
- To produce sorted output without an additional sort when a merge join is used.
- To spill gracefully when memory is insufficient (hash join batches, merge join sorts).

#### Syntax Rules and Structure

**Hash join plan**:

```text
Hash Join
  Hash Cond: (a.key = b.key)
  ->  probe_input (large)
  ->  Hash
        Buckets: 1024  Batches: 1  Memory Usage: 12kB
        ->  build_input (small)
```

**Merge join plan**:

```text
Merge Join
  Merge Cond: (a.key = b.key)
  ->  Index Scan using a_key_idx on a  (sorted by key)
  ->  Sort
        Sort Key: b.key
        ->  Seq Scan on b
```

**Component breakdown**:

| Aspect | Hash Join | Merge Join |
|--------|-----------|------------|
| Input requirement | At least one equi-join | Sorted on join keys |
| Memory | Build-side hash table | Sort buffers |
| Output order | Unsorted | Sorted |
| Spill behavior | Batches to disk | Sort spills to disk |
| Best when | Input sizes differ significantly | Inputs presorted or similar size |

**Syntax rules**:

- Hash join requires at least one equi-join condition.
- Merge join requires both inputs sorted on the join keys; sorts may be explicit or index-provided.
- Both can spill to disk; hash join uses batches, merge join uses external merge sort.

**Constraints and limitations**:

- Hash join cannot handle non-equi joins.
- Merge join's sort cost may exceed hash join's build cost for large unsorted inputs.
- Both are memory-sensitive; insufficient memory causes severe performance degradation.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Hash join with batch spill

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT ...
FROM large_probe p
JOIN large_build b ON b.key = p.key;
```

**Expected output** (problematic):

```text
Hash Join
  Hash Cond: (b.key = p.key)
  Buffers: shared hit=1000 read=50000, temp read=8000 written=8000
  ->  Seq Scan on large_probe p
  ->  Hash
        Buckets: 65536  Batches: 8  Memory Usage: 4096kB
        ->  Seq Scan on large_build b
```

**Why this output appears**: The hash table for `large_build` exceeded `work_mem`, so it was split into 8 batches (spilled to disk). `Batches > 1` and `temp read/written` are clear indicators of spill. Increasing `work_mem` or reducing the build-side size would eliminate the spill.

#### Real-World Cases

- **Data warehousing**: Hash joins for large fact-to-dimension joins where dimension tables fit in memory.
- **Ordered reporting**: Merge joins when the query also requires `ORDER BY` on the join keys.
- **Skewed builds**: Hash join with a highly skewed build side may spill; a merge join may be more robust.

---

### 7. Nested Loop Performance

#### Definitions

**Core Definition**: Nested loop join performance is determined by the number of outer rows, the cost of accessing the inner input per outer row, and the availability of an index on the inner join key.

**Technical Definition**: A nested loop join scans the outer input once and, for each outer row, scans or seeks the inner input. With an index on the inner join key, the inner access is an index lookup (index nested loop), costing approximately `O(log M)` per outer row. Without an index, the inner input is scanned fully for each outer row, costing `O(N × M)`. The total cost is dominated by `outer_rows × inner_access_cost`. PostgreSQL uses values from the current outer row as keys for the inner index scan when possible.

**Beginner-Friendly Explanation**: A nested loop is fast when the outer list is short and the inner list has an index—like looking up 5 words in a dictionary. It is slow when the outer list is long and the inner list has no index—like reading the entire dictionary 10,000 times.

#### Purposes

- To efficiently join a small outer result set to an indexed inner table.
- To support non-equi joins (range predicates) where hash and merge joins are less efficient.
- To avoid the memory overhead of hash or merge joins when the outer set is small.
- To enable early termination when only a few rows are needed (e.g., with `LIMIT`).

#### Syntax Rules and Structure

**Index nested loop**:

```text
Nested Loop
  ->  outer_input (small)
  ->  Index Scan on inner_table
        Index Cond: (inner.key = outer.key)
        loops = outer_rows
```

**Naive nested loop (no inner index)** :

```text
Nested Loop
  ->  outer_input
  ->  Seq Scan on inner_table
        Filter: (inner.key = outer.key)
        loops = outer_rows
```

**Component breakdown**:

| Factor | Impact |
|--------|--------|
| Outer rows | Linear multiplier on inner executions |
| Inner index | Converts inner scan to `O(log M)` seek |
| `loops` | `actual time × loops` = total inner cost |
| Join predicate | Index-compatible predicates are crucial |

**Syntax rules**:

- Nested loop is the only algorithm that can efficiently handle non-equi joins (e.g., `BETWEEN`, `<`, `>`).
- Index nested loop requires an index on the inner join column.
- The outer input should be the smaller or more selective input.

**Constraints and limitations**:

- Without an inner index, nested loop is catastrophic for large inputs (`O(N × M)`).
- Even with an index, a large outer input (e.g., 1 million rows) causes 1 million index lookups, which may be slower than a hash join.
- SQL Server documentation notes that nested loops can be effective if the outer input is small and the inner input is preindexed and large, but index nested loops are superior in many small transactions.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Index nested loop with `loops` revealing cost

```sql
EXPLAIN ANALYZE
SELECT o.id, c.email
FROM orders o
JOIN customers c ON c.id = o.customer_id
WHERE o.status = 'pending';
```

**Expected output** (illustrative):

```text
Nested Loop  (actual time=0.050..2500.000 rows=10000 loops=1)
  ->  Seq Scan on orders o  (actual time=0.010..100.000 rows=10000 loops=1)
        Filter: (status = 'pending')
  ->  Index Scan using customers_pkey on customers c
        (actual time=0.001..0.001 rows=1 loops=10000)
        Index Cond: (id = o.customer_id)
```

**Why this output appears**: The inner `customers` index scan has `loops=10000`, meaning it was executed once for each pending order. Per-loop time is 0.001 ms, but the total contribution is approximately 10,000 × 0.001 ms = 10 ms (plus overhead). The outer sequential scan took 100 ms. The total query time is about 2.5 seconds, much of which comes from the repeated inner executions.

#### Real-World Cases

- **OLTP point lookups**: Small outer result sets with indexed inner tables.
- **Non-equi joins**: Range predicates where hash join is inapplicable.
- **LIMIT queries**: Nested loop can stop early when the outer input is small and the limit is reached.

---

### 8. Join Memory

#### Definitions

**Core Definition**: Join memory is the amount of working memory allocated to join operators—hash tables for hash joins, sort buffers for merge joins—controlled by database parameters such as `work_mem` (PostgreSQL) and memory grants (SQL Server).

**Technical Definition**: In PostgreSQL, `work_mem` is the maximum memory each individual operation (sort, hash, etc.) can use before spilling to disk. A query with multiple hash joins may use `work_mem` per node, so total memory can be several times `work_mem`. In SQL Server, the optimizer calculates a **memory grant** for the query based on estimated row sizes and cardinalities; insufficient grants cause spills. MySQL uses `join_buffer_size` and related parameters. Oracle uses the PGA (Program Global Area) and `PGA_AGGREGATE_TARGET`.

**Beginner-Friendly Explanation**: Join memory is the desk space the database has to work with. A hash join needs enough desk space to hold the hash table; if the desk is too small, it has to put some of the table on the floor (disk), which is much slower. The database has settings to control how much desk space each operation gets.

#### Purposes

- To ensure hash joins and merge joins can operate in memory without spilling.
- To balance memory allocation across concurrent queries.
- To avoid excessive memory consumption that could cause swapping or OOM errors.
- To understand why increasing `work_mem` sometimes fixes a slow join.

#### Syntax Rules and Structure

**PostgreSQL**:

```sql
-- Session-level work_mem.
SET work_mem = '64MB';

-- Per-query setting.
SET LOCAL work_mem = '256MB';
```

**SQL Server**:

```sql
-- Memory grants are automatic; monitor with:
SELECT granted_memory_kb, used_memory_kb, max_used_memory_kb
FROM sys.dm_exec_query_memory_grants;
```

**Component breakdown**:

| Engine | Parameter | Unit | Scope |
|--------|-----------|------|-------|
| PostgreSQL | `work_mem` | Per operation | Session or global |
| SQL Server | Memory grant | Per query | Automatic |
| MySQL | `join_buffer_size` | Per join buffer | Session or global |
| Oracle | PGA | Per process | Automatic |

**Syntax rules**:

- PostgreSQL's `work_mem` is not a total memory limit; it applies to each sort/hash node separately.
- SQL Server's memory grant is based on estimated cardinality; underestimates cause spills.
- Increasing `work_mem` globally affects all concurrent queries and can cause memory pressure.

**Constraints and limitations**:

- Setting `work_mem` too high can cause excessive memory use under high concurrency.
- Memory grants in SQL Server are limited by `max server memory` and Resource Governor settings.
- Hash joins may still spill even with generous memory if the build side is extremely large.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Increasing `work_mem` for a hash join

```sql
-- Before: default work_mem causes spill.
SET work_mem = '4MB';
EXPLAIN ANALYZE
SELECT ... FROM large_probe p JOIN large_build b ON b.key = p.key;
-- Plan shows "Batches: 16" and temp read/written.

-- After: increase work_mem for this query.
SET LOCAL work_mem = '128MB';
EXPLAIN ANALYZE
SELECT ... FROM large_probe p JOIN large_build b ON b.key = p.key;
-- Plan shows "Batches: 1" and no temp read/written.
```

**Expected output** (after):

```text
Hash
  Buckets: 65536  Batches: 1  Memory Usage: 98304kB
```

**Why this output appears**: Increasing `work_mem` allows the hash table to fit entirely in memory, eliminating the multi-batch spill and the associated disk I/O.

#### Real-World Cases

- **Large hash joins**: Increasing `work_mem` for a reporting query eliminates spill and reduces runtime by orders of magnitude.
- **Concurrent workloads**: Setting `work_mem` too high globally causes memory exhaustion; `SET LOCAL` per query is safer.
- **SQL Server memory grants**: Monitoring `sys.dm_exec_query_memory_grants` reveals spills and underestimated grants.

---

### 9. Join Spill

#### Definitions

**Core Definition**: Join spill is the condition in which a join operator (hash join or merge join) exhausts its allocated memory and writes intermediate data to temporary disk storage.

**Technical Definition**: **Hash join spill** occurs when the build-side hash table exceeds `work_mem` (PostgreSQL) or the memory grant (SQL Server); the hash table is partitioned into batches, and batches that do not fit are written to temporary files. Probe rows are then matched against batches on disk, requiring repeated reads. **Merge join spill** occurs when the sort buffers for either input exceed memory, causing an external merge sort that writes sorted runs to disk and merges them. Spill degrades performance by orders of magnitude because disk random I/O replaces in-memory pointer chasing.

**Beginner-Friendly Explanation**: Spill is like running out of desk space and having to put papers on the floor. A hash join that spills has to write part of its hash table to disk and read it back later, which is much slower than keeping it in memory. Spill happens when the database underestimated how much memory the join would need.

#### Purposes

- To understand why a join that was fast yesterday is slow today.
- To diagnose memory-related performance regressions.
- To tune `work_mem`, memory grants, or query structure to avoid spill.
- To recognize the plan indicators of spill (`Batches > 1`, `temp read/written`, `external merge Disk`).

#### Syntax Rules and Structure

**PostgreSQL spill indicators**:

```text
Hash
  Buckets: 65536  Batches: 16  Memory Usage: 4096kB
  Buffers: temp read=8000 written=8000
```

```text
Sort
  Sort Method: external merge  Disk: 102400kB
```

**SQL Server spill indicators**:

```text
Sort Operator
  Warning: Sort spilled to tempdb
```

**Component breakdown**:

| Indicator | Meaning |
|-----------|---------|
| `Batches > 1` | Hash join spilled to disk (PostgreSQL). |
| `temp read/written` | Temporary file I/O (PostgreSQL). |
| `external merge Disk` | Sort spilled to disk (PostgreSQL). |
| `spilled to tempdb` | SQL Server spill warning. |
| `granted_memory_kb` vs `used_memory_kb` | SQL Server grant vs usage mismatch. |

**Syntax rules**:

- Spill is caused by insufficient memory relative to the build-side size or sort input size.
- Root causes include stale statistics (underestimating cardinality), data skew, and type mismatches.
- Increasing memory is a temporary fix; fixing statistics or query structure is more robust.

**Constraints and limitations**:

- Spill cannot always be avoided; extremely large joins may exceed any practical memory limit.
- Temporary disk space must be available; spill can fill `tempdb` or the temporary tablespace.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Hash join spill with `Batches > 1`

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT ...
FROM large_probe p
JOIN large_build b ON b.key = p.key;
```

**Expected output** (problematic):

```text
Hash Join
  Hash Cond: (b.key = p.key)
  Buffers: shared hit=1000 read=50000, temp read=8000 written=8000
  ->  Seq Scan on large_probe p
  ->  Hash
        Buckets: 65536  Batches: 8  Memory Usage: 4096kB
        ->  Seq Scan on large_build b
```

**Why this output appears**: The hash table for `large_build` exceeded `work_mem`, so it was split into 8 batches. `Batches > 1` and `temp read/written` indicate spill. The fix is to increase `work_mem`, update statistics, or restructure the query to reduce the build-side size.

#### Real-World Cases

- **Stale statistics**: The optimizer estimated 10,000 build rows but the actual is 2 million, causing a massive spill.
- **Data skew**: One key value dominates the build side, causing uneven batches and poor spill performance.
- **Type mismatch**: Implicit casts prevent index usage and inflate the build side.

---

## Summary: Join Optimization Decision Matrix

| Decision | Key Factors | Default Engine Behavior |
|----------|-------------|--------------------------|
| Join order | Table sizes, selectivity, indexes | Cost-based DP (≤12 tables) or GEQO |
| Algorithm | Input sizes, sort order, memory | Cost-based choice among NL, hash, merge |
| Hints | When optimizer errs | SQL Server `OPTION`, Oracle `/*+ */` |
| Elimination | Unused joined tables, unique keys | Automatic rewrite |
| Reordering | Inner joins, `join_collapse_limit` | Enabled by default; disable with limit=1 |
| Hash vs. merge | Input size similarity, sort needs | Hash for unequal sizes; merge for presorted |
| Nested loop | Small outer, indexed inner | Chosen when outer is small |
| Memory | `work_mem`, memory grants | Per-operation; spill if insufficient |
| Spill | `Batches > 1`, `temp read/written` | Indicates memory exhaustion |

---

## References

- PostgreSQL Documentation – Planner/Optimizer Join Strategies – https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/15/postgresql-15-A4.pdf
- PostgreSQL Documentation – Controlling the Planner with Explicit JOIN Clauses – https://www.postgresql.org/docs/current/explicit-joins.html
- PostgreSQL Documentation – Genetic Query Optimization (GEQO) – https://www.postgresql.org/docs/current/geqo.html
- MySQL 8.0 Reference Manual – Hash Join Optimization – https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/hash-joins.html
- MySQL 8.0 Reference Manual – Nested-Loop Join Algorithms – https://dev.mysql.com/doc/refman/8.0/en/nested-loop-joins.html
- Microsoft Learn – Joins (SQL Server) – https://learn.microsoft.com/en-us/sql/relational-databases/performance/joins
- Microsoft Learn – OPTION Clause (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/queries/option-clause-transact-sql
- Microsoft Learn – SQL Server Physical Joins – https://learn.microsoft.com/en-us/previous-versions/sql/sql-server-2008-r2/ms191426(v=sql.105)
- Oracle Database SQL Tuning Guide – Joins – https://docs.oracle.com/en/database/oracle/oracle-database/26/tgsql/joins.html
- Oracle Database SQL Language Reference – Hints (USE_HASH, USE_NL, USE_MERGE) – https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Comments.html
- PostgreSQL Documentation – `work_mem` – https://www.postgresql.org/docs/current/runtime-config-resource.html
- SQL Server – `sys.dm_exec_query_memory_grants` – https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-memory-grants-transact-sql
- PostgreSQL Wiki – Join Removal – https://wiki.postgresql.org/wiki/Join_Removal
- Codeberg – `ra` Join Elimination Rules – https://codeberg.org/gregburd/ra
- DiSC – The Value of Merge-Join and Hash-Join in SQL Server – https://sigmod.org