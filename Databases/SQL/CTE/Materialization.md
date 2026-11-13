# Comprehensive Programming Cheat Sheet: CTE Materialization

## Topic Overview

### Definitions

**Core Definition**  
CTE materialization is the database engine’s decision to either compute a Common Table Expression once into a temporary result set (materialised) or to splice its definition directly into the referencing query (inlined).

**Technical Definition**  
When a CTE is materialised, the engine evaluates the CTE’s query expression once, stores the rows in an internal tuplestore or temporary relation, and reuses that stored result for every reference. When a CTE is inlined (or “flattened”), the engine substitutes the CTE’s query text into each reference point, allowing the optimiser to push down predicates, use indexes, and choose join orders across the CTE boundary. PostgreSQL exposes explicit control via the `MATERIALIZED` and `NOT MATERIALIZED` hints (introduced in PostgreSQL 12); other engines make the decision implicitly or via different mechanisms (SQL Server `OPTION (RECOMPILE)`, MySQL 8.0 heuristics).

**Beginner-Friendly Explanation**  
Imagine you need the same list of numbers several times in a calculation. You can either write the list once on a sticky note and reuse it (materialised), or you can recalculate the list each time you need it, possibly skipping parts you don’t need (inlined). Materialisation trades a one‑time cost for reuse; inlining trades repeated work for the chance to optimise each use.

### Key Characteristics

- **Automatic by default in many engines**: PostgreSQL ≤11 always materialised CTEs; PostgreSQL 12+ inlines by default when the CTE is referenced once and has no side effects.
- **Explicit hints**: `MATERIALIZED` forces materialisation; `NOT MATERIALIZED` requests inlining (PostgreSQL 12+).
- **Optimisation fence**: A materialised CTE is an optimisation fence: predicates from the outer query cannot be pushed into it, and join order across it is fixed.
- **Reuse amortisation**: Materialisation pays off when the CTE is referenced multiple times.
- **Side‑effect semantics**: Data‑modifying CTEs are always materialised (PostgreSQL) to ensure their side effects occur exactly once.
- **Vendor variation**: SQL Server, MySQL, and Oracle decide materialisation implicitly; only PostgreSQL exposes direct hints.
- **Recursive CTEs are materialised**: Recursive CTEs are evaluated iteratively into a working table and are effectively materialised.

### Prerequisites

- Understanding of single and multiple CTEs, including recursive CTEs.
- Familiarity with query plans (`EXPLAIN`, `EXPLAIN ANALYZE`, execution plans).
- Knowledge of predicates, indexes, and join strategies.
- Awareness of the target engine’s default materialisation behaviour.

### Related Programming Areas

- Query optimisation and execution‑plan analysis.
- Performance tuning and benchmarking.
- Data‑modifying CTEs and side‑effect semantics.
- View and subquery flattening.
- Database engine internals (tuple stores, CTE scans).

---

## Core Concepts and Features

### 1. Materialized CTEs

#### Definitions

**Core Definition**  
A materialised CTE is a CTE whose result set is computed once and stored temporarily for the duration of the statement.

**Technical Definition**  
When the optimiser chooses materialisation, it evaluates the CTE’s query expression exactly once, stores the result in a temporary structure (a tuplestore in PostgreSQL, a worktable in SQL Server, a temporary table in MySQL), and rewrites every reference to the CTE as a scan over that stored result. The stored result is not indexed by default in PostgreSQL (though a `CTE Scan` node may use a hash for lookups), and predicates from the outer query cannot be pushed into it.

**Beginner-Friendly Explanation**  
A materialised CTE is like writing the result of a helper query on a sticky note and reading from the note whenever the main query needs it, instead of re‑running the helper query each time.

#### Purposes

- To avoid recomputing an expensive CTE that is referenced multiple times.
- To guarantee that volatile or side‑effecting expressions are evaluated exactly once.
- To create a stable snapshot of data within a statement when required for semantics.
- To isolate a CTE from outer predicate pushdown when such pushdown would change results or performance.

#### Syntax Rules and Structure

**PostgreSQL** (explicit):

```sql
WITH cte_name AS MATERIALIZED (
    query_expression
)
main_statement;
```

**PostgreSQL ≤11 / SQL Server / MySQL** (implicit):

```sql
WITH cte_name AS (
    query_expression
)
main_statement;
-- Materialisation is chosen by the optimiser.
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `AS MATERIALIZED` | PostgreSQL 12+ hint forcing materialisation. |
| `query_expression` | The CTE’s defining query, evaluated once. |
| `main_statement` | References the CTE; each reference scans the stored result. |

**Syntax rules**

- `MATERIALIZED` is a PostgreSQL‑specific hint; other engines materialise implicitly based on heuristics.
- The stored result is not indexed automatically; repeated lookups may be slow.
- Predicates from the outer query are not pushed into a materialised CTE by default.
- Data‑modifying CTEs in PostgreSQL are always materialised (and must include `RETURNING` if referenced).

**Constraints and limitations**

- Materialisation consumes memory or temporary disk proportional to the CTE’s result size.
- Materialised CTEs act as optimisation fences, which can hurt performance if the CTE is large and the outer query would have filtered it heavily.
- `MATERIALIZED` is a hint; the engine generally honours it, but behaviour is version‑specific.
- SQL Server and MySQL do not expose an explicit materialisation hint; control is indirect (e.g., `OPTION (RECOMPILE)` or temp tables).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Forcing materialisation for a reused CTE

```sql
-- Step 1: Define a CTE that is expensive and referenced twice.
-- Step 2: Force materialisation so it is computed once.
WITH expensive AS MATERIALIZED (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT c.customer_id, c.total, e.total AS same_total
FROM expensive c
JOIN expensive e ON e.customer_id = c.customer_id
WHERE c.total > 500;
```

**Expected output** (illustrative):

```text
 customer_id |  total  | same_total
-------------+---------+------------
         104 | 12345.0 |    12345.0
         112 |  4321.5 |     4321.5
(2 rows)
```

**Why this output appears**: The `MATERIALIZED` hint forces the aggregation to run once; both references scan the stored result, so `total` and `same_total` match without recomputing the aggregation.

##### Example 2: PostgreSQL ≤11 – Implicit materialisation

```sql
-- In PostgreSQL 11 and earlier, CTEs are always materialised.
WITH totals AS (
    SELECT customer_id, SUM(amount) AS total
    FROM orders GROUP BY customer_id
)
SELECT * FROM totals WHERE total > 500;
```

**Expected output**:

```text
 customer_id |  total
-------------+---------
         104 | 12345.0
         112 |  4321.5
(2 rows)
```

**Why this output appears**: PostgreSQL ≤11 materialises every CTE; the outer `WHERE total > 500` filters the stored result after the aggregation has been computed for all customers.

##### Example 3: Data‑modifying CTE (always materialised in PostgreSQL)

```sql
-- Step 1: A data-modifying CTE is always materialised; its side effect occurs once.
WITH inserted AS (
    INSERT INTO audit_log (event) VALUES ('batch run')
    RETURNING id
)
SELECT id FROM inserted;
```

**Expected output**:

```text
 id
----
  1
(1 row)
```

**Why this output appears**: PostgreSQL materialises data‑modifying CTEs to guarantee the `INSERT` runs exactly once, regardless of how many times `inserted` is referenced.

#### Real-World Cases

- **Expensive aggregations referenced multiple times**: Materialise once, reuse in joins.
- **Volatile expressions**: Ensure a `random()` or sequence call is evaluated once.
- **Audit logging**: Data‑modifying CTEs guarantee a single side effect per statement.

---

### 2. Inline CTEs

#### Definitions

**Core Definition**  
An inline CTE is a CTE whose definition is substituted into each reference point, allowing the optimiser to treat it as part of the surrounding query.

**Technical Definition**  
When a CTE is inlined, the optimiser replaces each reference to the CTE with the CTE’s query expression, effectively flattening it into the outer query. Predicates, join conditions, and projections from the outer query can then be pushed into the inlined query, enabling index usage, partition pruning, and better join orders. PostgreSQL 12+ inlines CTEs by default when they are referenced once and are side‑effect‑free, unless `MATERIALIZED` is specified.

**Beginner-Friendly Explanation**  
An inline CTE is like copying the helper query’s text directly into the main query each time it is used, so the database can optimise the combined query as a whole.

#### Purposes

- To allow predicate pushdown and index usage across the CTE boundary.
- To avoid the overhead of storing and scanning a temporary result.
- To let the optimiser reorder joins and choose better plans.
- To reduce memory and temporary storage consumption for large CTEs used once.

#### Syntax Rules and Structure

**PostgreSQL** (explicit):

```sql
WITH cte_name AS NOT MATERIALIZED (
    query_expression
)
main_statement;
```

**Implicit inlining** (PostgreSQL 12+ default for single‑reference CTEs):

```sql
WITH cte_name AS (
    query_expression
)
main_statement;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `AS NOT MATERIALIZED` | PostgreSQL 12+ hint requesting inlining. |
| `query_expression` | Substituted into each reference. |
| `main_statement` | Optimised together with the inlined expression. |

**Syntax rules**

- `NOT MATERIALIZED` is a PostgreSQL‑specific hint.
- Inlining is possible only if the CTE has no side effects (no data‑modifying statements, no volatile functions that must run once).
- Recursive CTEs cannot be inlined.
- CTEs referenced multiple times may still be inlined, but the engine may prefer materialisation to avoid recomputation; the hint overrides the default in PostgreSQL 12+.

**Constraints and limitations**

- Inlining a CTE referenced multiple times can cause the CTE’s query to run multiple times, increasing cost.
- Volatile functions may be evaluated multiple times when inlined, changing semantics.
- `NOT MATERIALIZED` is a hint; the optimiser may still materialise if inlining is unsafe.
- Other engines do not expose this hint directly; SQL Server may inline or materialise based on its own rules.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Forcing inlining for a filtered CTE

```sql
-- Step 1: Define a CTE that the outer query filters heavily.
-- Step 2: Request inlining so the filter can use an index.
WITH recent AS NOT MATERIALIZED (
    SELECT order_id, customer_id, amount
    FROM orders
    WHERE order_date >= '2025-01-01'
)
SELECT * FROM recent WHERE amount > 1000;
```

**Expected output** (illustrative):

```text
 order_id | customer_id | amount
----------+-------------+---------
     9001 |         104 | 1500.00
     9002 |         112 | 2200.00
(2 rows)
```

**Why this output appears**: With inlining, the predicate `amount > 1000` can be pushed into the `orders` scan, potentially using an index on `amount` and avoiding a full scan of `recent`.

##### Example 2: PostgreSQL 12+ – Default inlining for a single‑reference CTE

```sql
-- In PostgreSQL 12+, a single-reference, side-effect-free CTE is inlined by default.
WITH totals AS (
    SELECT customer_id, SUM(amount) AS total
    FROM orders GROUP BY customer_id
)
SELECT * FROM totals WHERE total > 500;
```

**Expected output**:

```text
 customer_id |  total
-------------+---------
         104 | 12345.0
         112 |  4321.5
(2 rows)
```

**Why this output appears**: The CTE is referenced once and has no side effects, so PostgreSQL 12+ inlines it; the optimiser can push `total > 500` into the aggregation as a `HAVING`‑like filter in some plans.

##### Example 3: MySQL 8.0 – Implicit inlining decision

```sql
-- MySQL 8.0 decides materialisation vs inlining heuristically.
WITH totals AS (
    SELECT customer_id, SUM(amount) AS total
    FROM orders GROUP BY customer_id
)
SELECT * FROM totals WHERE total > 500;
```

**Expected output**:

```text
+-------------+--------+
| customer_id | total  |
+-------------+--------+
|         104 | 12345.0|
|         112 |  4321.5|
+-------------+--------+
2 rows in set (0.00 sec)
```

**Why this output appears**: MySQL 8.0 may inline the CTE and push the predicate, or materialise it and filter afterwards; the result is the same, but the execution plan differs.

#### Real-World Cases

- **Filtered reporting queries**: Inlining lets outer filters use indexes.
- **Single‑use CTEs**: Inlining avoids unnecessary temporary storage.
- **Large CTEs**: Inlining can reduce memory when the CTE is not reused.

---

### 3. `MATERIALIZED` Hint

#### Definitions

**Core Definition**  
`MATERIALIZED` is a PostgreSQL hint that forces a CTE to be evaluated once and stored.

**Technical Definition**  
Introduced in PostgreSQL 12, `MATERIALIZED` overrides the default inlining heuristic and instructs the planner to evaluate the CTE once, storing its result in a tuplestore. The planner then treats each reference as a scan over the stored result (`CTE Scan` node). The hint is honoured even when the CTE is referenced once, and it prevents predicate pushdown into the CTE.

**Beginner-Friendly Explanation**  
`MATERIALIZED` tells PostgreSQL, “Compute this helper query once and save the result; don’t merge it into the main query.”

#### Purposes

- To force a single evaluation of an expensive CTE referenced multiple times.
- To guarantee once‑only evaluation of volatile functions or side effects.
- To prevent the planner from inlining a CTE in ways that change semantics.
- To stabilise performance by creating an optimisation fence.

#### Syntax Rules and Structure

**Syntax**:

```sql
WITH cte_name AS MATERIALIZED (
    query_expression
)
main_statement;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `AS MATERIALIZED` | Hint keyword placed between `AS` and the parenthesised query. |
| `query_expression` | Evaluated once. |
| `main_statement` | References scan the stored result. |

**Syntax rules**

- Only valid in PostgreSQL 12 and later.
- Applies per CTE; multiple CTEs may mix `MATERIALIZED` and `NOT MATERIALIZED`.
- Cannot be applied to recursive CTEs (which are always materialised).
- Data‑modifying CTEs are always materialised regardless of the hint.

**Constraints and limitations**

- Materialisation consumes memory/temp disk for the CTE’s full result.
- Acts as an optimisation fence: outer predicates cannot be pushed in.
- Not supported in other engines; use engine‑specific equivalents (e.g., temp tables).
- The hint is honoured in practice but documented as a hint, not a guarantee.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Comparing plans with and without `MATERIALIZED`

```sql
-- Step 1: Force materialisation.
EXPLAIN (COSTS OFF)
WITH expensive AS MATERIALIZED (
    SELECT customer_id, SUM(amount) AS total
    FROM orders GROUP BY customer_id
)
SELECT * FROM expensive WHERE total > 500;
```

**Expected output** (illustrative plan):

```text
                  QUERY PLAN
----------------------------------------------
 CTE Scan on expensive
   Filter: (total > 500::numeric)
   CTE expensive
     ->  HashAggregate
           Group Key: orders.customer_id
           ->  Seq Scan on orders
(6 rows)
```

**Why this output appears**: The `CTE Scan` node scans the materialised CTE; the `Filter` is applied outside the CTE because materialisation prevents pushdown into the aggregation.

##### Example 2: PostgreSQL – Using `MATERIALIZED` for a reused CTE

```sql
WITH shared AS MATERIALIZED (
    SELECT id, amount FROM orders WHERE amount > 100
)
SELECT a.id, b.id
FROM shared a
JOIN shared b ON b.id = a.id + 1;
```

**Expected output**:

```text
 id  | id
-----+-----
 101 | 102
 102 | 103
(2 rows)
```

**Why this output appears**: The CTE is evaluated once and scanned twice in the self‑join, avoiding duplicate evaluation of the filter.

##### Example 3: PostgreSQL – `MATERIALIZED` with a volatile function

```sql
WITH snapshot AS MATERIALIZED (
    SELECT random() AS r
)
SELECT r FROM snapshot UNION ALL SELECT r FROM snapshot;
```

**Expected output** (illustrative):

```text
         r
--------------------
 0.1234567890123456
 0.1234567890123456
(2 rows)
```

**Why this output appears**: Materialisation guarantees `random()` is evaluated once; both references return the same value. Without materialisation, the two references would produce different random values.

#### Real-World Cases

- **Volatile function consistency**: Ensuring a timestamp or random value is stable across references.
- **Expensive reused CTEs**: Avoiding repeated aggregation.
- **Stable plans**: Forcing a predictable plan shape for benchmarking.

---

### 4. `NOT MATERIALIZED` Hint

#### Definitions

**Core Definition**  
`NOT MATERIALIZED` is a PostgreSQL hint that requests inlining of a CTE.

**Technical Definition**  
Introduced in PostgreSQL 12, `NOT MATERIALIZED` instructs the planner to inline the CTE’s query expression into each reference, if it is safe to do so. This enables predicate pushdown and join reordering across the CTE boundary. The planner may still materialise if inlining is unsafe (e.g., due to side effects or recursion). The hint overrides the default decision for single‑reference CTEs and may cause multiple evaluation when the CTE is referenced more than once.

**Beginner-Friendly Explanation**  
`NOT MATERIALIZED` tells PostgreSQL, “Don’t save this helper query’s result; just plug it into the main query so it can be optimised together.”

#### Purposes

- To allow outer predicates to be pushed into the CTE for index usage.
- To enable join reordering across the CTE boundary.
- To avoid temporary storage for large CTEs used once.
- To give the optimiser freedom to choose a better plan.

#### Syntax Rules and Structure

**Syntax**:

```sql
WITH cte_name AS NOT MATERIALIZED (
    query_expression
)
main_statement;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `AS NOT MATERIALIZED` | Hint requesting inlining. |
| `query_expression` | Substituted into each reference. |
| `main_statement` | Optimised together with the inlined expression. |

**Syntax rules**

- PostgreSQL 12+ only.
- Inlining is refused if the CTE is recursive or contains side‑effect‑relevant constructs (e.g., data‑modifying statements).
- If the CTE is referenced multiple times, inlining may duplicate the CTE’s work; the hint does not prevent this.
- Volatile functions may be evaluated multiple times when inlined.

**Constraints and limitations**

- Duplicate evaluation can increase total cost for reused CTEs.
- Semantics of volatile functions change (evaluated per reference).
- The planner may still materialise if inlining is unsafe.
- Not portable to other engines.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Enabling predicate pushdown with `NOT MATERIALIZED`

```sql
-- Step 1: The CTE has a large result, but the outer query filters heavily.
WITH recent AS NOT MATERIALIZED (
    SELECT * FROM orders WHERE order_date >= '2025-01-01'
)
SELECT * FROM recent WHERE customer_id = 104;
```

**Expected output**:

```text
 order_id | customer_id | amount | order_date
----------+-------------+--------+------------
     9001 |         104 | 1500.00 | 2025-02-01
     9002 |         104 |  250.00 | 2025-03-01
(2 rows)
```

**Why this output appears**: Inlining allows the predicate `customer_id = 104` to be pushed into the `orders` scan, potentially using an index on `(customer_id, order_date)` and scanning fewer rows.

##### Example 2: PostgreSQL – Duplicate evaluation when inlining a reused CTE

```sql
-- Step 1: The CTE is referenced twice; NOT MATERIALIZED may cause it to run twice.
WITH t AS NOT MATERIALIZED (
    SELECT customer_id, SUM(amount) AS total FROM orders GROUP BY customer_id
)
SELECT a.customer_id, a.total, b.total
FROM t a JOIN t b ON b.customer_id = a.customer_id
WHERE a.total > 500;
```

**Expected output**:

```text
 customer_id |  total  |  total
-------------+---------+---------
         104 | 12345.0 | 12345.0
         112 |  4321.5 |  4321.5
(2 rows)
```

**Why this output appears**: The result is correct, but the plan may evaluate the aggregation twice because of inlining. This can be slower than materialising once.

##### Example 3: PostgreSQL – Volatile function evaluated per reference

```sql
WITH t AS NOT MATERIALIZED (
    SELECT random() AS r
)
SELECT (SELECT r FROM t) AS r1, (SELECT r FROM t) AS r2;
```

**Expected output** (illustrative):

```text
         r1         |         r2
--------------------+--------------------
 0.3456789012345678 | 0.8765432109876543
(1 row)
```

**Why this output appears**: Inlining causes `random()` to be evaluated separately for each reference, yielding different values. `MATERIALIZED` would return the same value twice.

#### Real-World Cases

- **Selective outer filters**: Pushdown reduces scanned rows dramatically.
- **Join reordering**: Let the optimiser choose the best join order across the CTE.
- **One‑off CTEs**: Avoid temporary storage for large but single‑use CTEs.

---

### 5. Performance Implications

#### Definitions

**Core Definition**  
Performance implications of CTE materialisation are the costs and benefits that arise from choosing materialisation versus inlining.

**Technical Definition**  
Materialisation incurs a one‑time evaluation cost plus storage and scan costs for each reference; it amortises the evaluation cost across references and provides a stable snapshot. Inlining incurs no storage cost and enables pushdown and reordering, but may re‑evaluate the CTE per reference and may change volatile‑function semantics. The optimal choice depends on the CTE’s cost, the number of references, the selectivity of outer predicates, and the presence of indexes.

**Beginner-Friendly Explanation**  
Materialising is like paying once to make a photocopy and then reading the copy many times; inlining is like re‑reading the original each time, but you can skip pages you don’t need.

#### Purposes

- To choose materialisation when the CTE is expensive and reused.
- To choose inlining when outer predicates are selective and the CTE is used once.
- To avoid optimisation fences that block index usage.
- To control memory and temporary storage usage.

#### Syntax Rules and Structure

**Decision factors**:

| Factor | Favours Materialisation | Favours Inlining |
|--------|-------------------------|------------------|
| References | Multiple | Single |
| CTE cost | High | Low |
| Outer predicate selectivity | Low (filters little) | High (filters much) |
| Volatile functions | Must run once | Fine to run per reference |
| Indexes on base tables | Not usable through fence | Usable via pushdown |

**PostgreSQL hints**:

```sql
WITH cte AS MATERIALIZED (...)       -- force materialisation
WITH cte AS NOT MATERIALIZED (...)   -- request inlining
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `MATERIALIZED` | Pays evaluation once; scans stored result per reference. |
| `NOT MATERIALIZED` | May pay evaluation per reference; enables pushdown. |
| Default (PostgreSQL 12+) | Inline if single reference and safe. |

**Syntax rules**

- Use `EXPLAIN` / `EXPLAIN ANALYZE` to compare plans.
- Measure both variants on realistic data before choosing.
- Recursive CTEs are always materialised.
- Data‑modifying CTEs are always materialised.

**Constraints and limitations**

- Hints are engine‑specific; other engines require alternative techniques (temp tables, `OPTION (RECOMPILE)`).
- Materialisation of very large CTEs can spill to disk and slow the query.
- Inlining of very expensive CTEs referenced many times can multiply cost.
- Statistics and parameter values affect the optimiser’s choice.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Comparing materialised vs inlined plans

```sql
-- Step 1: Materialised plan.
EXPLAIN (ANALYZE, COSTS OFF)
WITH cte AS MATERIALIZED (
    SELECT * FROM orders WHERE amount > 10
)
SELECT * FROM cte WHERE customer_id = 104;

-- Step 2: Inlined plan.
EXPLAIN (ANALYZE, COSTS OFF)
WITH cte AS NOT MATERIALIZED (
    SELECT * FROM orders WHERE amount > 10
)
SELECT * FROM cte WHERE customer_id = 104;
```

**Expected output** (illustrative):

```text
-- Materialised
 CTE Scan on cte (actual time=0.050..0.120 rows=5 loops=1)
   Filter: (customer_id = 104)
   CTE cte
     ->  Seq Scan on orders (actual time=0.010..0.080 rows=200 loops=1)
           Filter: (amount > 10)

-- Inlined
 Index Scan using idx_orders_customer on orders (actual time=0.010..0.020 rows=5 loops=1)
   Index Cond: (customer_id = 104)
   Filter: (amount > 10)
```

**Why this output appears**: Materialisation scans the full `orders` table once, then filters the stored result; inlining pushes both predicates into an index scan, reading far fewer rows.

##### Example 2: SQL Server – Temp table as a manual materialisation

```sql
-- Step 1: Materialise manually with a temp table.
SELECT customer_id, SUM(amount) AS total
INTO #totals
FROM Orders
GROUP BY customer_id;

-- Step 2: Reuse the temp table twice.
SELECT a.customer_id, a.total, b.total
FROM #totals a JOIN #totals b ON b.customer_id = a.customer_id
WHERE a.total > 500;

-- Step 3: Clean up.
DROP TABLE #totals;
```

**Expected output**:

```text
customer_id total   total
----------- ------- -------
104         12345.0 12345.0
112          4321.5  4321.5
```

**Why this output appears**: SQL Server lacks a CTE materialisation hint, so a temp table emulates the one‑time evaluation and reuse.

##### Example 3: MySQL – `EXPLAIN` comparison

```sql
-- Step 1: MySQL 8.0 may materialise or inline automatically.
EXPLAIN FORMAT=JSON
WITH totals AS (
    SELECT customer_id, SUM(amount) AS total
    FROM orders GROUP BY customer_id
)
SELECT * FROM totals WHERE total > 500;
```

**Expected output** (illustrative JSON excerpt):

```json
{
  "query_block": {
    "select_id": 1,
    "having_condition": "(`total` > 500)",
    "table": {
      "table_name": "orders",
      "access_type": "ALL",
      "group_by": "customer_id"
    }
  }
}
```

**Why this output appears**: MySQL 8.0 often merges the CTE into the outer query, producing a plan that applies the filter as part of the grouped query.

#### Real-World Cases

- **Reporting queries**: Materialise expensive aggregations reused in multiple joins.
- **Selective lookups**: Inline CTEs that are heavily filtered by the outer query.
- **Benchmarking**: Compare plans before and after adding hints.

---

### 6. Optimization Fence

#### Definitions

**Core Definition**  
An optimisation fence is a boundary in a query that prevents the optimiser from moving predicates or reordering operations across it.

**Technical Definition**  
A materialised CTE is an optimisation fence: the optimiser treats it as an opaque relation, so outer predicates cannot be pushed into it, and join order across the boundary is constrained. Inlining removes the fence, allowing predicate pushdown, projection pushdown, and join reordering. The fence can be desirable (to stabilise semantics or performance) or undesirable (to block optimisations that would improve performance).

**Beginner-Friendly Explanation**  
An optimisation fence is like a wall in the query that stops the database from moving filters or joins across it; materialising a CTE builds such a wall, while inlining removes it.

#### Purposes

- To stabilise plans by preventing aggressive transformations.
- To guarantee semantics when pushdown would change results (e.g., with volatile functions).
- To isolate expensive subqueries from outer predicates when desired.
- To reason about plans more predictably.

#### Syntax Rules and Structure

**Fence creation**:

```sql
WITH cte AS MATERIALIZED (
    SELECT ... FROM t WHERE ...
)
SELECT ... FROM cte WHERE outer_predicate;
```

**Fence removal**:

```sql
WITH cte AS NOT MATERIALIZED (
    SELECT ... FROM t WHERE ...
)
SELECT ... FROM cte WHERE outer_predicate;
```

**Component breakdown**

| Form | Fence? | Predicate pushdown? |
|------|--------|---------------------|
| `MATERIALIZED` | Yes | No |
| `NOT MATERIALIZED` | No | Yes (subject to safety) |
| Default (PostgreSQL 12+, single ref) | No | Yes |
| Recursive CTE | Yes | No |
| Data‑modifying CTE | Yes | No |

**Syntax rules**

- Materialisation creates a fence by definition.
- Inlining removes the fence only if the CTE is safe to inline.
- The fence affects predicate pushdown, projection pushdown, and join reordering.
- Other engines may create fences implicitly (e.g., temp tables, `OPTION (RECOMPILE)`).

**Constraints and limitations**

- Fences can drastically hurt performance when outer filters are selective.
- Removing fences can multiply evaluation cost if the CTE is reused.
- The optimiser may still apply some transformations across the fence (e.g., limit pushdown in some engines).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Fence blocks predicate pushdown

```sql
-- Materialised CTE: the filter cannot be pushed into the CTE.
EXPLAIN (COSTS OFF)
WITH big AS MATERIALIZED (
    SELECT * FROM orders WHERE amount > 10
)
SELECT * FROM big WHERE customer_id = 104;
```

**Expected output**:

```text
                 QUERY PLAN
--------------------------------------------
 CTE Scan on big
   Filter: (customer_id = 104)
   CTE big
     ->  Seq Scan on orders
           Filter: (amount > 10)
(5 rows)
```

**Why this output appears**: The `CTE Scan` applies the `customer_id` filter after materialisation; the `Seq Scan` on `orders` sees only `amount > 10`, so no index on `customer_id` is used inside the CTE.

##### Example 2: PostgreSQL – Removing the fence with `NOT MATERIALIZED`

```sql
EXPLAIN (COSTS OFF)
WITH big AS NOT MATERIALIZED (
    SELECT * FROM orders WHERE amount > 10
)
SELECT * FROM big WHERE customer_id = 104;
```

**Expected output**:

```text
                 QUERY PLAN
--------------------------------------------
 Index Scan using idx_orders_customer on orders
   Index Cond: (customer_id = 104)
   Filter: (amount > 10)
(3 rows)
```

**Why this output appears**: Inlining removes the fence, so both predicates are pushed into the index scan.

##### Example 3: SQL Server – Fence via a temp table

```sql
SELECT * INTO #big FROM Orders WHERE amount > 10;

SELECT * FROM #big WHERE customer_id = 104;

DROP TABLE #big;
```

**Expected output** (plan excerpt):

```text
-- The temp table scan applies customer_id = 104 after the temp table is built.
```

**Why this output appears**: The temp table materialises the `Orders` filter; the outer filter is applied afterwards, exactly as a materialised CTE would behave.

#### Real-World Cases

- **Stable benchmarks**: Fences make plan shapes predictable.
- **Semantics preservation**: Fences prevent pushdown that would change results with volatile functions.
- **Performance regression diagnosis**: Identifying that a fence blocks index usage is a common tuning insight.

---

## References

- PostgreSQL Documentation – `WITH` Queries (Materialization) – https://www.postgresql.org/docs/current/queries-with.html#QUERIES-WITH-MATERIALIZATION
- PostgreSQL Documentation – `SELECT` (`WITH` clause, `MATERIALIZED` / `NOT MATERIALIZED`) – https://www.postgresql.org/docs/current/sql-select.html
- PostgreSQL 12 Release Notes – CTE Inlining – https://www.postgresql.org/docs/release/12.0/
- MySQL 8.4 Reference Manual – `WITH` (Common Table Expressions, Optimizer Hints) – https://dev.mysql.com/doc/refman/8.4/en/with.html
- Microsoft Learn – `WITH common_table_expression` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql
- Microsoft Learn – `OPTION` Clause (`RECOMPILE`, `MAXRECURSION`) – https://learn.microsoft.com/en-us/sql/t-sql/queries/option-clause-transact-sql
- SQLite Documentation – Common Table Expressions (Materialization) – https://www.sqlite.org/lang_with.html
- SQL:1999 Standard – Common Table Expressions (ISO/IEC 9075-2:1999)
- Modern SQL – CTE Materialization and Optimization – https://modern-sql.com/feature/with
- Use The Index, Luke – CTE Materialization and Performance – https://use-the-index-luke.com/sql/with
- Baeldung – CTE Materialization in PostgreSQL – https://www.baeldung.com/sql/postgresql-cte-materialization