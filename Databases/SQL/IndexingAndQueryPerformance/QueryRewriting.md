Query rewriting is a cornerstone of database performance tuning: it is the process by which a database management system (DBMS) transforms a declarative SQL query into an equivalent but more efficient logical form before physical execution planning begins. The core principle is **semantic equivalence**—the rewritten query must produce exactly the same result as the original—while unlocking faster access paths, reducing intermediate data volumes, and enabling more effective use of indexes and join algorithms.

---

## Topic Overview

### Definitions

**Core Definition**: Query rewriting (also called query transformation or query simplification) is the set of rule-based logical transformations applied by a DBMS optimizer to a SQL statement to produce a semantically equivalent statement that can be executed more efficiently.

**Technical Definition**: Query rewriting operates on the **logical query plan**—typically a relational algebra expression tree—using **equivalence rules** that guarantee identical result sets across all database instances. These rules include algebraic identities (e.g., selection distributes over join), predicate logic simplifications (constant folding, transitivity), subquery unnesting (converting nested subqueries into joins or semi-joins), and structural simplifications (removing redundant operators such as unnecessary `DISTINCT` or `GROUP BY`). The rewriter produces one or more alternative logical plans; the physical optimizer then selects the lowest-cost access plan for the chosen logical form.

**Beginner-Friendly Explanation**: Query rewriting is like editing a recipe before you start cooking. The original recipe might say "boil water, then add pasta, then drain, then pour sauce, then serve." A rewrite might combine steps: "boil pasta in salted water, drain, and toss with sauce." The dish is the same, but the kitchen runs more smoothly. In SQL, the database automatically edits your query to find a faster way to get the same answer.


## Key Characteristics

- **Equivalence-preserving**: Every rewrite must produce the same result set (including `NULL` semantics and duplicate behavior) across all valid database states.
- **Cost-based selection**: Rewrites generate alternatives; the optimizer chooses among them based on statistics and cost estimates, not rules alone.
- **Heuristic-driven**: Many rewrites (predicate pushdown, constant folding) are applied unconditionally because they almost always reduce cost.
- **Interaction-sensitive**: `NULL` values, duplicates, aggregation, and subqueries interact in ways that can make a seemingly safe rewrite semantically unsafe.
- **Engine-specific**: Not all engines implement all rewrites; PostgreSQL, MySQL, SQL Server, and Oracle differ in their rewrite capabilities and aggressiveness.


## Prerequisites

- Understanding of `SELECT`, `WHERE`, `JOIN`, `GROUP BY`, `HAVING`, and subqueries.
- Familiarity with relational algebra concepts: selection (σ), projection (π), join (⨝), union (∪).
- Basic knowledge of indexes, statistics, and execution plans (`EXPLAIN`).
- Awareness of SQL's three-valued logic (`TRUE`, `FALSE`, `UNKNOWN`/`NULL`).


## Related Programming Areas

- Cost-based query optimization and plan enumeration.
- Index design and access path selection.
- Materialized view matching and aggregate rewrite.
- Semantic query optimization using integrity constraints.
- Compiler optimization (common subexpression elimination, constant propagation).


## Core Concepts and Features

### 1. Rewriting Subqueries

#### Definitions

**Core Definition**: Subquery rewriting transforms nested `SELECT` statements—correlated or uncorrelated, scalar or multi-row—into joins, semi-joins, anti-joins, or derived tables that the optimizer can execute more efficiently.

**Technical Definition**: Subquery rewriting includes **subquery unnesting** (converting a subquery into a join), **`EXISTS`-to-`IN` conversion** (enabling semi-join strategies), **`IN`-to-`EXISTS` conversion** (enabling early termination), and **correlated aggregate subquery to derived table** transformation (pre-aggregating once instead of re-executing per outer row). The optimizer may also apply **subquery caching**, but rewriting aims to eliminate the need for repeated execution entirely.

**Beginner-Friendly Explanation**: A correlated subquery is like asking "for each order, look up the customer name" one order at a time. A rewrite turns it into "join orders to customers once and get all the names together"—much faster when there are many orders.

#### Purposes

- To eliminate per-row subquery execution (the "N+1" pattern) that multiplies cost by the outer row count.
- To enable the optimizer to choose between nested loop, hash join, and merge join for subquery logic.
- To expose filtering opportunities that allow indexes to be used earlier.
- To reduce the size of intermediate results by pre-aggregating derived tables.

#### Syntax Rules and Structure

**Correlated scalar subquery → `LEFT JOIN`**:

```sql
-- Before
SELECT o.id, (SELECT c.name FROM customers c WHERE c.id = o.customer_id) AS cname
FROM orders o;

-- After
SELECT o.id, c.name AS cname
FROM orders o
LEFT JOIN customers c ON c.id = o.customer_id;
```

**`IN` subquery → `EXISTS` (for early termination)**:

```sql
-- Before
SELECT * FROM customers
WHERE id IN (SELECT customer_id FROM orders WHERE total > 1000);

-- After
SELECT * FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id AND o.total > 1000);
```

**Correlated aggregate subquery → derived table**:

```sql
-- Before: re-runs AVG per product row
SELECT * FROM products p
WHERE price > (SELECT AVG(price) FROM products WHERE category_id = p.category_id);

-- After: aggregation runs once
SELECT p.*
FROM products p
JOIN (SELECT category_id, AVG(price) AS avg_price
      FROM products GROUP BY category_id) agg
  ON p.category_id = agg.category_id
WHERE p.price > agg.avg_price;
```

**Component breakdown**:

| Rewrite | Trigger | Benefit |
|---------|---------|---------|
| Correlated scalar → `LEFT JOIN` | Scalar subquery in `SELECT` or `WHERE` | Eliminates per-row lookup |
| `IN` → `EXISTS` | Large subquery result | Early termination per outer row |
| `EXISTS` → `IN` | Trivially correlated `EXISTS` | Enables hash semi-join |
| Aggregate subquery → derived table | Aggregate in correlated `WHERE` | Aggregation computed once |

**Syntax rules**:

- `NOT IN` and `NOT EXISTS` are not interchangeable when `NULL` values are present: `NOT IN (subquery)` returns no rows if the subquery returns any `NULL`.
- Converting `EXISTS` to `IN` requires `IS NOT NULL` handling in some engines.
- `LEFT JOIN` preserves outer rows with no match; `INNER JOIN` would drop them.

**Constraints and limitations**:

- Subquery unnesting may be blocked when the subquery contains `LIMIT`, window functions, or volatile functions.
- `NOT IN` with nullable columns cannot be safely rewritten to `NOT EXISTS` without semantic verification.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Correlated scalar subquery to `LEFT JOIN`

```sql
-- Before: DEPENDENT SUBQUERY re-executes per order row.
EXPLAIN
SELECT o.id, o.total,
       (SELECT name FROM customers WHERE id = o.customer_id) AS cname
FROM orders o;

-- After: single LEFT JOIN with index lookup per match.
EXPLAIN
SELECT o.id, o.total, c.name AS cname
FROM orders o
LEFT JOIN customers c ON c.id = o.customer_id;
```

**Expected output** (plan excerpt, before):

```text
id | select_type       | table     | type | rows
1  | PRIMARY           | o         | ALL  | 800000
2  | DEPENDENT SUBQUERY | customers | const| 1
```

**Expected output** (plan excerpt, after):

```text
id | select_type | table | type | rows
1  | SIMPLE      | o     | ALL  | 800000
1  | SIMPLE      | c     | ref  | 1
```

**Why this output appears**: The `DEPENDENT SUBQUERY` in the before plan signals that the subquery re-executes once per outer row (800,000 times). The `LEFT JOIN` after rewrite executes the customer lookup using an index per matching row, dramatically reducing total work.

##### Example 2: PostgreSQL – `IN` to `EXISTS` for large subquery

```sql
-- Before
SELECT * FROM customers
WHERE id IN (SELECT DISTINCT customer_id FROM orders WHERE total > 1000);

-- After
SELECT * FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o
              WHERE o.customer_id = c.id AND o.total > 1000);
```

**Expected output** (identical result sets, but the `EXISTS` version can short-circuit after the first match per customer).

**Why this output appears**: `IN` with a large subquery materializes the entire subquery result before comparing. `EXISTS` evaluates per outer row and stops as soon as one matching order is found, which is faster when matches are common.

#### Real-World Cases

- **ORM-generated queries**: Object-relational mappers frequently emit correlated subqueries; rewriting them as joins reduces per-row overhead.
- **Reporting queries**: `IN` subqueries over large dimension tables are often better expressed as `EXISTS` or joins.
- **Data quality checks**: Anti-join rewrites (`NOT EXISTS` or `LEFT JOIN ... IS NULL`) for "missing record" queries.


### 2. Rewriting Joins

#### Definitions

**Core Definition**: Join rewriting changes the order, type, or structure of joins—including converting outer joins to inner joins, reordering inner joins, and eliminating unnecessary joins—while preserving the result set.

**Technical Definition**: Join rewriting leverages the **commutativity and associativity of inner joins** to reorder join operands, and applies **null-rejection analysis** to convert `LEFT JOIN` to `INNER JOIN` when `WHERE` clauses guarantee that non-preserved rows cannot appear in the result. The optimizer may also **pull up** subqueries containing inner joins to expose join reordering opportunities. **Join elimination** removes joins whose tables do not contribute to the output and cannot affect row multiplicity.

**Beginner-Friendly Explanation**: An outer join is like saying "give me all customers, even those without orders." But if the next line says "only show customers who have orders," the outer join is wasted work—the rewrite recognizes this and uses a simpler inner join.

#### Purposes

- To unlock join reordering, which is only safe for inner joins (outer joins constrain the order of evaluation).
- To reduce the number of rows processed by eliminating joins that cannot change the result.
- To allow the optimizer to choose the most efficient join algorithm for the data size.
- To simplify plan enumeration by reducing the join graph.

#### Syntax Rules and Structure

**`LEFT JOIN` → `INNER JOIN` (null-rejection)**:

```sql
-- Before: LEFT JOIN, but WHERE rejects NULLs from b
SELECT a.id, b.value
FROM a
LEFT JOIN b ON b.a_id = a.id
WHERE b.status = 'active';

-- After: INNER JOIN (equivalent because WHERE filters out non-matching rows)
SELECT a.id, b.value
FROM a
INNER JOIN b ON b.a_id = a.id
WHERE b.status = 'active';
```

**Join reordering (inner joins)**:

```sql
-- Before: a JOIN b JOIN c (order chosen arbitrarily)
SELECT ...
FROM a
JOIN b ON b.a_id = a.id
JOIN c ON c.b_id = b.id;

-- After: optimizer may reorder to c → b → a if statistics favor it.
```

**Component breakdown**:

| Rewrite | Safety condition | Effect |
|---------|------------------|--------|
| `LEFT JOIN` → `INNER JOIN` | `WHERE` predicate on right table rejects `NULL` | Unlocks reordering and simpler plans |
| Inner join reordering | All joins are inner | Enables cost-based order selection |
| Join elimination | Joined table contributes no columns and cannot multiply rows | Removes redundant join |

**Syntax rules**:

- Outer join conversion is safe only when a `WHERE` condition on the null-supplying table would reject `NULL`-extended rows.
- Inner join reordering is always semantically safe; outer join reordering requires careful null-preservation analysis.
- Join elimination requires a **unique key** on the join column of the eliminated table (or a guarantee of at most one match) to avoid changing row multiplicity.

**Constraints and limitations**:

- `FULL OUTER JOIN` is not commutative and cannot be freely reordered.
- Join elimination with non-unique keys may change duplicate counts.
- Volatile functions in join predicates may block reordering.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – `LEFT JOIN` to `INNER JOIN` conversion

```sql
-- Before
EXPLAIN
SELECT a.id, b.value
FROM a
LEFT JOIN b ON b.a_id = a.id
WHERE b.status = 'active';

-- After
EXPLAIN
SELECT a.id, b.value
FROM a
INNER JOIN b ON b.a_id = a.id
WHERE b.status = 'active';
```

**Expected output** (before plan, excerpt):

```text
Hash Left Join
  Hash Cond: (a.id = b.a_id)
  Filter: (b.status = 'active')
```

**Expected output** (after plan, excerpt):

```text
Hash Join
  Hash Cond: (a.id = b.a_id)
  Filter: (b.status = 'active')
```

**Why this output appears**: The `WHERE b.status = 'active'` predicate rejects rows where `b` is `NULL` (the null-extended side of the left join). The optimizer recognizes this and converts the `LEFT JOIN` to an `INNER JOIN`, which is cheaper because it can use a plain hash join without null-extension logic.

##### Example 2: Teradata – Outer join elimination

```sql
-- Before: join to t3 is redundant (only t3.key is used, and t3.key is unique)
SELECT t1.id, t2.value
FROM t1
JOIN t2 ON t2.t1_id = t1.id
LEFT JOIN t3 ON t3.t2_id = t2.id
WHERE t3.key = 42;

-- After: t3 can be eliminated if the optimizer proves its columns are unused
SELECT t1.id, t2.value
FROM t1
JOIN t2 ON t2.t1_id = t1.id
WHERE EXISTS (SELECT 1 FROM t3 WHERE t3.t2_id = t2.id AND t3.key = 42);
```

**Why this output appears**: The `LEFT JOIN` to `t3` is converted to a semi-join or eliminated when the optimizer proves that `t3`'s columns are not projected and the join cannot multiply `t2` rows (due to a unique key). Teradata's outer-join elimination performs this rewrite.

#### Real-World Cases

- **View merging**: Views often contain outer joins; predicate pushdown and null-rejection analysis convert them to inner joins.
- **ORM joins**: Tool-generated `LEFT JOIN` chains are frequently converted when application filters reject nulls.
- **Data warehouse star joins**: Join elimination removes dimension tables that are not referenced in the `SELECT` list.


### 3. Rewriting Aggregates

#### Definitions

**Core Definition**: Aggregate rewriting simplifies, repositions, or eliminates `GROUP BY`, `HAVING`, and aggregate functions to reduce computation and intermediate result sizes.

**Technical Definition**: Aggregate rewriting includes **redundant `GROUP BY` removal** (when the grouping columns already contain a unique key), **`HAVING` to `WHERE` conversion** (when the `HAVING` predicate does not reference aggregates and can be applied earlier), **`DISTINCT` elimination** (when the query already produces distinct rows due to a unique key or grouping), and **aggregate pushdown** (moving partial aggregation below joins or unions).

**Beginner-Friendly Explanation**: If you group by a column that is already unique, the `GROUP BY` is doing nothing—the rewrite removes it. If your `HAVING` clause only checks a column value, the rewrite moves that check into `WHERE` so rows are filtered before grouping, not after.

#### Purposes

- To avoid expensive sorting and hashing when grouping is provably unnecessary.
- To reduce the number of rows entering aggregation by filtering earlier.
- To eliminate duplicate-removal operations when uniqueness is guaranteed.
- To enable better join ordering by reducing intermediate result sizes.

#### Syntax Rules and Structure

**Redundant `GROUP BY` removal**:

```sql
-- Before: id is a primary key, so grouping by it adds nothing
SELECT id, COUNT(*) FROM orders GROUP BY id;

-- After: no grouping needed (each id appears once)
SELECT id, 1 AS count FROM orders;
```

**`HAVING` → `WHERE` conversion**:

```sql
-- Before
SELECT department, AVG(salary)
FROM employees
GROUP BY department
HAVING department = 'Engineering';

-- After
SELECT department, AVG(salary)
FROM employees
WHERE department = 'Engineering'
GROUP BY department;
```

**Redundant `DISTINCT` removal**:

```sql
-- Before: DISTINCT is unnecessary because id is a primary key
SELECT DISTINCT id, name FROM products;

-- After
SELECT id, name FROM products;
```

**Component breakdown**:

| Rewrite | Trigger | Benefit |
|---------|---------|---------|
| `GROUP BY` removal | Grouping columns include a unique key | Avoids sort/hash |
| `HAVING` → `WHERE` | `HAVING` references only grouping columns | Filters before aggregation |
| `DISTINCT` removal | Output already unique (key in `SELECT` list) | Avoids deduplication |
| Aggregate pushdown | Aggregation above a union or join | Reduces rows earlier |

**Syntax rules**:

- `HAVING` can be converted to `WHERE` only if it does not reference aggregate functions.
- `DISTINCT` removal requires proof that the output rows are already unique (e.g., a primary key in the `SELECT` list).
- `GROUP BY` removal requires that the grouping columns functionally determine all other selected columns.

**Constraints and limitations**:

- `DISTINCT` may still be needed if joins can multiply rows even when a key is selected.
- `GROUP BY` removal changes the result if the grouping columns are not unique.
- Aggregate pushdown through outer joins is unsafe if null-extension could change aggregate results.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Redundant `DISTINCT` elimination

```sql
-- Before: DISTINCT forces a sort/hash even though id is unique
EXPLAIN SELECT DISTINCT id, name FROM products;

-- After: DISTINCT removed
EXPLAIN SELECT id, name FROM products;
```

**Expected output** (before plan, excerpt):

```text
id | select_type | table    | type | Extra
1  | SIMPLE      | products | ALL  | Using temporary
```

**Expected output** (after plan, excerpt):

```text
id | select_type | table    | type | Extra
1  | SIMPLE      | products | ALL  |
```

**Why this output appears**: The `DISTINCT` on a primary key column is redundant because each `id` already appears once. The optimizer removes the `DISTINCT`, eliminating the `Using temporary` step that would have sorted or hashed the result to remove duplicates.

##### Example 2: Oracle – `HAVING` to `WHERE` conversion

```sql
-- Before: HAVING filters after grouping
SELECT department_id, MIN(salary)
FROM employees
GROUP BY department_id
HAVING department_id = 50;

-- After: WHERE filters before grouping
SELECT department_id, MIN(salary)
FROM employees
WHERE department_id = 50
GROUP BY department_id;
```

**Why this output appears**: The `HAVING department_id = 50` predicate references only the grouping column, not an aggregate. Moving it to `WHERE` reduces the number of rows entering the aggregation from all departments to only department 50.

#### Real-World Cases

- **BI dashboards**: `DISTINCT` is often added defensively; rewriting removes it when joins cannot duplicate key rows.
- **Data quality reports**: `HAVING` clauses that filter on group keys are common candidates for `WHERE` conversion.
- **Aggregate views**: Materialized aggregate views rely on rewrite matching to answer queries without re-aggregating base data.


### 4. Removing Redundant Operations

#### Definitions

**Core Definition**: Redundant operation removal eliminates SQL clauses or operators that cannot affect the result, such as unnecessary `DISTINCT`, redundant `GROUP BY`, no-op filters, or unused joins.

**Technical Definition**: This category includes removing **`DISTINCT`** when uniqueness is guaranteed, removing **`GROUP BY`** when grouping columns contain a unique key, removing **`ORDER BY`** in subqueries where the outer query does not rely on the order (unless `LIMIT` is present), and eliminating **unused joins** that do not contribute columns and cannot multiply rows.

**Beginner-Friendly Explanation**: If you tell the database to "remove duplicates" from a list that is already unique, the instruction is wasted effort. The rewrite drops the instruction.

#### Purposes

- To avoid costly sort and hash operations for duplicate elimination.
- To reduce the number of joins and intermediate result rows.
- To simplify the plan tree, reducing optimizer and execution overhead.
- To allow the optimizer to focus on the operations that actually matter.

#### Syntax Rules and Structure

**Unnecessary `ORDER BY` in subquery without `LIMIT`**:

```sql
-- Before: ORDER BY in subquery is ignored by outer query
SELECT * FROM (SELECT * FROM orders ORDER BY created_at DESC) sub;

-- After: ORDER BY removed
SELECT * FROM orders;
```

**Redundant `DISTINCT` with unique key**:

```sql
-- Before
SELECT DISTINCT customer_id FROM orders;  -- customer_id is not unique

-- After (if customer_id were unique in a dimension table)
SELECT customer_id FROM customers;
```

**Component breakdown**:

| Redundant operation | Removal condition |
|---------------------|-------------------|
| `DISTINCT` | Output rows provably unique |
| `GROUP BY` | Grouping columns include a unique key |
| `ORDER BY` in subquery | No `LIMIT`/`OFFSET`, outer query ignores order |
| Unused join | Joined table contributes no columns and cannot multiply rows |

**Syntax rules**:

- `ORDER BY` in a subquery without `LIMIT` is semantically irrelevant and may be removed by the optimizer.
- `DISTINCT` removal requires proof of uniqueness (primary key, unique constraint, or `GROUP BY` on a key).
- Join elimination requires a unique key on the join column or a guarantee of at most one match.

**Constraints and limitations**:

- Removing `ORDER BY` from a subquery is safe only if the outer query does not rely on the order (which it legally cannot, but some engines preserve it incidentally).
- `DISTINCT` removal is unsafe if a join can multiply rows despite a key being present.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Redundant `DISTINCT` removal

```sql
-- Before: DISTINCT is unnecessary because customer_id is a primary key
EXPLAIN SELECT DISTINCT customer_id, name FROM customers;

-- After: DISTINCT removed
EXPLAIN SELECT customer_id, name FROM customers;
```

**Expected output** (before plan, excerpt):

```text
Unique
  ->  Seq Scan on customers
```

**Expected output** (after plan, excerpt):

```text
Seq Scan on customers
```

**Why this output appears**: The `Unique` node (which implements `DISTINCT` by sorting or hashing) is eliminated because `customer_id` is unique. The resulting plan is a simple sequential scan with no deduplication step.

##### Example 2: SQL Server – Unused join elimination

```sql
-- Before: join to region is not used in SELECT or WHERE
SELECT o.id, o.total
FROM orders o
JOIN region r ON r.id = o.region_id;

-- After: join eliminated if r.id is unique and no columns from r are used
SELECT o.id, o.total
FROM orders o;
```

**Why this output appears**: The join to `region` cannot change the number of `orders` rows (because `r.id` is unique) and no `region` columns are projected. The optimizer eliminates the join entirely.

#### Real-World Cases

- **Reporting queries**: Analysts often add `DISTINCT` defensively; the optimizer removes it when joins are key-preserving.
- **View definitions**: `ORDER BY` inside views is frequently redundant and ignored; rewriting removes it.
- **Star-schema queries**: Joins to dimension tables that are not referenced in the `SELECT` list can be eliminated.


### 5. Predicate Pushdown

#### Definitions

**Core Definition**: Predicate pushdown moves filter conditions (`WHERE` predicates) as close to the data source as possible—ideally to the table scan—so that fewer rows flow through the rest of the query plan.

**Technical Definition**: Predicate pushdown, also called **predicate pushing** or **selection pushdown**, applies the relational algebra rule that selection distributes over joins, unions, and other operations: `σ_p(R ⨝ S) = σ_p(R) ⨝ S` when `p` references only `R`'s columns. The optimizer pushes predicates through views, derived tables, CTEs, and subqueries, and into the storage engine (e.g., pushing a predicate to an index scan). **Predicate pull-up** is the inverse, used when pulling a predicate from a subquery enables better optimization at the outer level.

**Beginner-Friendly Explanation**: If you are looking for orders from 2025, don't first collect all orders and then filter by year—filter at the source. Predicate pushdown tells the database to apply the year filter while reading the table, so only the relevant rows are ever processed.

#### Purposes

- To reduce the number of rows flowing through joins, sorts, and aggregations.
- To enable index usage on base tables by exposing sargable predicates at the scan level.
- To minimize memory and I/O consumption for intermediate results.
- To allow the optimizer to choose better join orders based on filtered cardinalities.

#### Syntax Rules and Structure

**Pushdown through a view**:

```sql
-- View definition
CREATE VIEW active_orders AS
SELECT * FROM orders WHERE status = 'active';

-- Query with additional predicate
SELECT * FROM active_orders WHERE customer_id = 42;

-- Rewritten: both predicates applied at the base table
SELECT * FROM orders
WHERE status = 'active' AND customer_id = 42;
```

**Pushdown through a union**:

```sql
-- Before
SELECT * FROM (
    SELECT * FROM orders_2024
    UNION ALL
    SELECT * FROM orders_2025
) u
WHERE customer_id = 42;

-- After
SELECT * FROM (
    SELECT * FROM orders_2024 WHERE customer_id = 42
    UNION ALL
    SELECT * FROM orders_2025 WHERE customer_id = 42
) u;
```

**Component breakdown**:

| Pushdown target | Rule |
|-----------------|------|
| Join | Push predicate to the operand whose columns it references |
| Union | Push predicate into each branch |
| View / derived table | Merge predicate into the view's query |
| Subquery | Pull predicate up or push it down depending on correlation |
| Storage engine | Push predicate to index or scan level |

**Syntax rules**:

- Predicates can be pushed through deterministic operators that do not change row values.
- Pushdown through outer joins requires care: pushing a predicate on the null-supplying side may convert the outer join to an inner join (which is a valid rewrite when the predicate rejects nulls).
- Pushdown through `LIMIT`, window functions, or volatile functions may be blocked.

**Constraints and limitations**:

- Predicates involving volatile or non-deterministic functions cannot be freely moved.
- Pushdown through `DISTINCT` or `GROUP BY` is only safe if the predicate references grouping columns.
- `OR` predicates may not push down cleanly through joins; they may require union rewrites.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Predicate pushdown through a view

```sql
-- Before: view scan without pushed predicate
EXPLAIN
SELECT * FROM active_orders WHERE customer_id = 42;

-- After: optimizer pushes customer_id into the view's base scan
EXPLAIN
SELECT * FROM orders
WHERE status = 'active' AND customer_id = 42;
```

**Expected output** (before plan, excerpt):

```text
Seq Scan on orders
  Filter: (status = 'active'::text)
```

**Expected output** (after plan, excerpt):

```text
Index Scan using idx_orders_customer on orders
  Index Cond: (customer_id = 42)
  Filter: (status = 'active'::text)
```

**Why this output appears**: The outer predicate `customer_id = 42` is pushed into the view's base query, allowing the optimizer to use an index on `customer_id` that would otherwise be inaccessible through the view.

##### Example 2: MySQL – Pushdown through a derived table

```sql
-- Before
SELECT * FROM (
    SELECT * FROM orders WHERE total > 100
) t
WHERE customer_id = 42;

-- After
SELECT * FROM (
    SELECT * FROM orders WHERE total > 100 AND customer_id = 42
) t;
```

**Why this output appears**: The predicate `customer_id = 42` is pushed into the derived table, combining with `total > 100` at the base scan. This reduces the derived table's size before the outer query processes it.

#### Real-World Cases

- **View-heavy reporting**: Predicates on outer queries are pushed into view definitions, enabling index usage.
- **Partitioned tables**: Pushdown prunes partitions based on date or category predicates.
- **Federated queries**: Predicates are pushed to remote data sources to minimize data transfer.


### 6. Join Elimination

#### Definitions

**Core Definition**: Join elimination removes a join operation when the joined table does not contribute to the query result and cannot change the number or identity of rows produced.

**Technical Definition**: Join elimination applies when a joined table's columns are not referenced in the `SELECT` list, `WHERE` clause, `GROUP BY`, or `HAVING`, **and** the join is **key-preserving**—that is, the join column on the eliminated side has a unique constraint (or the join cannot multiply rows). The optimizer can then remove the join entirely. **Outer join elimination** is a special case: a `LEFT JOIN` can be eliminated when the null-supplying table's columns are unused and the join is provably cardinality-preserving.

**Beginner-Friendly Explanation**: If you join orders to a regions table just to filter on region but never select any region columns, and each region has exactly one row, the join is unnecessary—you can apply the filter directly or remove the join altogether.

#### Purposes

- To avoid reading and joining a table that cannot affect the result.
- To reduce plan complexity and execution time.
- To eliminate unnecessary I/O and hash table builds.
- To simplify queries generated by tools that join more tables than necessary.

#### Syntax Rules and Structure

**Eliminable inner join**:

```sql
-- Before: join to regions, but no region columns used
SELECT o.id, o.total
FROM orders o
JOIN regions r ON r.id = o.region_id
WHERE o.total > 100;

-- After: join removed (r.id is unique)
SELECT o.id, o.total
FROM orders o
WHERE o.total > 100;
```

**Eliminable outer join**:

```sql
-- Before: LEFT JOIN to customers, but only customers with orders are kept
SELECT o.id, c.name
FROM orders o
LEFT JOIN customers c ON c.id = o.customer_id
WHERE c.name IS NOT NULL;

-- After: join converted to INNER and possibly eliminated if c.name is not used
SELECT o.id, c.name
FROM orders o
JOIN customers c ON c.id = o.customer_id;
```

**Component breakdown**:

| Condition | Requirement |
|-----------|-------------|
| Joined table columns unused | Not in `SELECT`, `WHERE`, `GROUP BY`, `HAVING` |
| Join is key-preserving | Unique key on join column, or `DISTINCT` in subquery |
| Outer join elimination | Null-supplying side unused and no row multiplication |

**Syntax rules**:

- Join elimination is safe only if the join cannot duplicate rows. A non-unique join column may multiply rows even if its columns are unused.
- Outer join elimination may require converting to an inner join first (via null-rejection) or proving that the outer join cannot preserve unmatched rows.
- Aggregate subquery joins can sometimes be eliminated if their result is not referenced.

**Constraints and limitations**:

- Joins with non-unique keys cannot be eliminated without changing row counts.
- Joins involving volatile functions or lateral references may be irreducible.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Inner join elimination

```sql
-- Before
EXPLAIN
SELECT o.id, o.total
FROM orders o
JOIN regions r ON r.id = o.region_id
WHERE o.total > 100;

-- After (join eliminated)
EXPLAIN
SELECT o.id, o.total
FROM orders o
WHERE o.total > 100;
```

**Expected output** (before plan, excerpt):

```text
Hash Join
  Hash Cond: (o.region_id = r.id)
  ->  Seq Scan on orders o
  ->  Hash
        ->  Seq Scan on regions r
```

**Expected output** (after plan, excerpt):

```text
Seq Scan on orders o
  Filter: (total > 100)
```

**Why this output appears**: `regions` is joined on its unique key (`r.id`), and no `regions` columns are used in the output or filter. The optimizer proves the join is cardinality-preserving and eliminates it, removing the hash table build and join operation entirely.

##### Example 2: Teradata – Outer join elimination

```sql
-- Before: LEFT JOIN to t3, but t3 columns are unused and t3.key is unique
SELECT t1.id, t2.value
FROM t1
JOIN t2 ON t2.t1_id = t1.id
LEFT JOIN t3 ON t3.t2_id = t2.id;

-- After: t3 join eliminated
SELECT t1.id, t2.value
FROM t1
JOIN t2 ON t2.t1_id = t1.id;
```

**Why this output appears**: Teradata's outer join elimination removes the `LEFT JOIN` to `t3` because no `t3` columns are referenced and `t3`'s join column is unique, so the join cannot change the result.

#### Real-World Cases

- **ORM-generated joins**: ORMs often join tables for relationships that are not referenced in the query result.
- **Star-schema queries**: Dimension tables that are filtered but not projected can sometimes be eliminated or converted to semi-joins.
- **View merging**: Views with joins to lookup tables are simplified when the lookup columns are unused.


### 7. Constant Folding

#### Definitions

**Core Definition**: Constant folding evaluates constant expressions at query compilation time and replaces them with their computed values, eliminating runtime computation.

**Technical Definition**: Constant folding applies the principle that expressions involving only literals (e.g., `1 + 2`, `'2025-01-01' + INTERVAL '1 day'`) can be evaluated once by the optimizer rather than per row. It also includes **constant propagation** (substituting known constant values for variables) and **constant condition removal** (simplifying `WHERE` clauses where a predicate is always `TRUE` or `FALSE`).

**Beginner-Friendly Explanation**: Instead of calculating `100 * 1.1` for every single row in a million-row table, the database calculates it once (110) and uses that value. The result is the same, but the work is done once instead of a million times.

#### Purposes

- To eliminate redundant per-row computation of expressions that never change.
- To simplify `WHERE` clauses into more sargable forms.
- To detect impossible conditions early and return empty results without scanning.
- To reduce CPU usage during query execution.

#### Syntax Rules and Structure

**Arithmetic constant folding**:

```sql
-- Before
SELECT * FROM orders WHERE total > 100 + 50;

-- After
SELECT * FROM orders WHERE total > 150;
```

**Boolean constant folding**:

```sql
-- Before
SELECT * FROM t WHERE (a < b AND b = c) AND a = 5;

-- After: b > 5 AND b = c AND a = 5
SELECT * FROM t WHERE b > 5 AND b = c AND a = 5;
```

**Constant condition removal**:

```sql
-- Before
SELECT * FROM t WHERE (b >= 5 AND b = 5) OR (b = 6 AND 5 = 5) OR (b = 7 AND 5 = 6);

-- After: 5 = 6 is always FALSE
SELECT * FROM t WHERE b = 5 OR b = 6;
```

**Component breakdown**:

| Folding type | Example | Result |
|--------------|---------|--------|
| Arithmetic | `100 + 50` | `150` |
| Boolean | `5 = 5` | `TRUE` |
| Condition removal | `... AND TRUE` | Remove `TRUE` |
| Constant propagation | `a = 5 AND b > a` | `b > 5` |

**Syntax rules**:

- Folding is safe for deterministic literals and immutable functions.
- `CURRENT_DATE`, `NOW()`, and other stable/volatile functions are **not** folded because their values change across executions.
- Folding can expose additional simplifications (e.g., folding a constant makes a predicate sargable).

**Constraints and limitations**:

- Folding of expressions involving columns is impossible without knowing the column's value.
- Volatile functions (e.g., `random()`) must not be folded because each execution should produce a different value.
- Overflow behavior must be consistent between folding and runtime evaluation.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Arithmetic constant folding

```sql
-- Before
EXPLAIN SELECT * FROM orders WHERE total > 100 + 50;

-- After
EXPLAIN SELECT * FROM orders WHERE total > 150;
```

**Expected output** (before plan, excerpt):

```text
Filter: (total > (100 + 50))
```

**Expected output** (after plan, excerpt):

```text
Filter: (total > 150)
```

**Why this output appears**: MySQL folds the constant expression `100 + 50` to `150` during query preparation, eliminating the per-row addition.

##### Example 2: SQL Server – Constant condition detection

```sql
-- Before
SELECT * FROM t WHERE c < 256;  -- c is TINYINT UNSIGNED

-- After: MySQL 8.0.16+ detects that c < 256 is always TRUE
SELECT * FROM t WHERE 1;
```

**Why this output appears**: For a `TINYINT UNSIGNED` column, the maximum value is 255, so `c < 256` is always true. The optimizer folds the predicate to `TRUE` and simplifies the `WHERE` clause.

#### Real-World Cases

- **Date arithmetic**: `WHERE order_date > DATEADD(day, -30, GETDATE())` is folded to a constant date once per query.
- **Generated queries**: ORMs often emit expressions like `1 = 1`; constant folding removes them.
- **Partition pruning**: Constant-folded date predicates enable partition elimination.


### 8. Common Subexpression Elimination

#### Definitions

**Core Definition**: Common subexpression elimination (CSE) identifies repeated identical subqueries or expressions and computes them once, reusing the result instead of recomputing.

**Technical Definition**: In SQL, CSE most commonly applies to **Common Table Expressions (CTEs)** and repeated subqueries. When a CTE is referenced multiple times, the optimizer may **materialize** it once into a temporary result and reuse it (avoiding duplicate computation), or **inline** it into each reference (avoiding materialization but risking recomputation). PostgreSQL 12+ inlines CTEs by default when referenced once; `MATERIALIZED` and `NOT MATERIALIZED` control the choice.

**Beginner-Friendly Explanation**: If you need the same expensive subquery result in two places in your query, the database can compute it once and save it in a temporary table, then read from that table twice instead of running the expensive subquery twice.

#### Purposes

- To avoid recomputing expensive subqueries multiple times.
- To ensure volatile functions or side-effecting expressions are evaluated exactly once.
- To allow the optimizer to choose between materialization (compute once, read multiple times) and inlining (merge into parent for joint optimization).
- To reduce total I/O and CPU when the same intermediate result is reused.

#### Syntax Rules and Structure

**CTE materialization (PostgreSQL)**:

```sql
WITH expensive AS MATERIALIZED (
    SELECT customer_id, SUM(total) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT c.name, e.total
FROM customers c
JOIN expensive e ON e.customer_id = c.id;
```

**CTE inlining (PostgreSQL 12+ default for single reference)**:

```sql
WITH filtered AS NOT MATERIALIZED (
    SELECT * FROM orders WHERE status = 'active'
)
SELECT * FROM filtered WHERE customer_id = 42;
-- Predicate customer_id = 42 is pushed into the base orders scan.
```

**Component breakdown**:

| Mode | Behavior | When to use |
|------|----------|-------------|
| `MATERIALIZED` | Compute CTE once, store result, reuse | Multiple references; expensive computation |
| `NOT MATERIALIZED` | Inline CTE into each reference | Single reference; predicate pushdown beneficial |
| Default (PG 12+) | Inline if single reference and safe | Most common case |

**Syntax rules**:

- `MATERIALIZED` forces separate computation; `NOT MATERIALIZED` forces folding into the parent query.
- Data-modifying CTEs (`WITH ... INSERT/UPDATE/DELETE`) are always materialized in PostgreSQL to ensure side effects occur exactly once.
- Volatile functions in a CTE must be materialized if the result must be consistent across references.

**Constraints and limitations**:

- `NOT MATERIALIZED` may cause the CTE to be computed multiple times if referenced multiple times, which can be slower for expensive queries.
- Materialization of very large CTEs consumes memory or temporary disk space.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – `MATERIALIZED` for a reused expensive CTE

```sql
-- Before: CTE referenced twice; without materialization, may run twice.
WITH totals AS (
    SELECT customer_id, SUM(total) AS total
    FROM orders GROUP BY customer_id
)
SELECT a.customer_id, a.total, b.total
FROM totals a JOIN totals b ON b.customer_id = a.customer_id + 1;

-- After: force materialization to compute once.
WITH totals AS MATERIALIZED (
    SELECT customer_id, SUM(total) AS total
    FROM orders GROUP BY customer_id
)
SELECT a.customer_id, a.total, b.total
FROM totals a JOIN totals b ON b.customer_id = a.customer_id + 1;
```

**Expected output** (identical results; the materialized version computes the aggregation once and scans it twice).

**Why this output appears**: The CTE is referenced twice. Without `MATERIALIZED`, PostgreSQL 12+ may inline it into both references, causing the aggregation to run twice. `MATERIALIZED` forces a single evaluation, which is faster when the aggregation is expensive.

##### Example 2: PostgreSQL – `NOT MATERIALIZED` for predicate pushdown

```sql
-- Before: default materialization blocks predicate pushdown
WITH big AS (
    SELECT * FROM orders WHERE total > 10
)
SELECT * FROM big WHERE customer_id = 42;

-- After: force inlining so customer_id predicate reaches the base scan
WITH big AS NOT MATERIALIZED (
    SELECT * FROM orders WHERE total > 10
)
SELECT * FROM big WHERE customer_id = 42;
```

**Why this output appears**: With `NOT MATERIALIZED`, the CTE is folded into the outer query, allowing `customer_id = 42` to be pushed into the base `orders` scan, potentially using an index.

#### Real-World Cases

- **Reused aggregation**: A CTE computing monthly totals is referenced by multiple downstream joins.
- **Predicate pushdown**: A CTE used once with an outer selective filter benefits from `NOT MATERIALIZED`.
- **Data-modifying CTEs**: `WITH moved_rows AS (DELETE ... RETURNING *) INSERT ...` is always materialized to ensure the delete happens once.


### 9. Query Simplification

#### Definitions

**Core Definition**: Query simplification applies algebraic identities and logical equivalences to reduce a query to a simpler form without changing its result.

**Technical Definition**: Simplification includes **transitive closure** (inferring `a = c` from `a = b` and `b = c`), **removing redundant predicates** (`a = a`), **eliminating `ANY`/`ALL` operators** in favor of equivalent joins or `EXISTS`/`NOT EXISTS`, **merging `HAVING` into `WHERE`** when no `GROUP BY` is present, and **removing unnecessary parentheses**. Simplification is often a prerequisite for other rewrites (e.g., folding constants exposes join elimination opportunities).

**Beginner-Friendly Explanation**: Query simplification is like cleaning up a messy sentence before translating it. If you say "all customers where the customer ID equals 5 and the customer ID equals 5," simplification removes the duplicate condition. The meaning is the same, but the query is cleaner and easier to optimize.

#### Purposes

- To reduce the logical complexity of the query before physical optimization.
- To expose additional rewrite opportunities (e.g., transitive closure can enable join elimination).
- To remove self-referential or tautological conditions that waste computation.
- To normalize query structure for more consistent optimization across engines.

#### Syntax Rules and Structure

**Transitive closure**:

```sql
-- Before: a = b and b = c
SELECT * FROM t1, t2, t3
WHERE t1.id = t2.id AND t2.id = t3.id;

-- After: transitive inference adds t1.id = t3.id, enabling join elimination
```

**Removing redundant predicates**:

```sql
-- Before
SELECT * FROM t WHERE id = id AND 1 = 1;

-- After
SELECT * FROM t;
```

**`HAVING` merge with `WHERE` (no `GROUP BY`)**:

```sql
-- Before
SELECT COUNT(*) FROM orders HAVING COUNT(*) > 10;

-- After (MySQL merges HAVING with WHERE when no GROUP BY/aggregate)
-- The optimizer handles this internally.
```

**Component breakdown**:

| Simplification | Rule | Example |
|----------------|------|---------|
| Transitive closure | `a = b`, `b = c` ⇒ `a = c` | Enables join elimination |
| Redundant predicate removal | `a = a` ⇒ `TRUE` | Removes no-op filters |
| `HAVING` → `WHERE` (no `GROUP BY`) | `HAVING` without aggregates | Filters earlier |
| `ANY`/`ALL` elimination | Convert to `EXISTS`/`NOT EXISTS` | Enables semi-join |

**Syntax rules**:

- Transitive closure is safe for equality predicates on non-nullable columns; `NULL` semantics require care.
- `HAVING` can be merged with `WHERE` only when there is no `GROUP BY` and no aggregate function references.
- `ANY` and `ALL` can be rewritten to `EXISTS`/`NOT EXISTS` with appropriate `NULL` handling.

**Constraints and limitations**:

- Simplification with `NULL` values requires three-valued logic reasoning; `a = a` is not `TRUE` if `a` is `NULL`.
- Transitive closure with `NULL` can infer incorrect equalities; most optimizers restrict it to non-nullable columns.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – `HAVING` merge with `WHERE`

```sql
-- Before: HAVING without GROUP BY or aggregate
SELECT * FROM orders HAVING customer_id = 42;

-- After: HAVING merged into WHERE
SELECT * FROM orders WHERE customer_id = 42;
```

**Why this output appears**: When `HAVING` does not reference any aggregate function and there is no `GROUP BY`, MySQL merges it into `WHERE`, allowing the predicate to be applied before any grouping logic (which is trivial here) and enabling index usage.

##### Example 2: PostgreSQL – Transitive closure enabling join elimination

```sql
-- Before
SELECT t1.id, t1.value
FROM t1, t2, t3
WHERE t1.id = t2.id AND t2.id = t3.id AND t1.id = 5;

-- After: transitive closure infers t1.id = t3.id, and t2/t3 joins may be eliminated
SELECT t1.id, t1.value
FROM t1
WHERE t1.id = 5;
```

**Why this output appears**: The equalities `t1.id = t2.id` and `t2.id = t3.id` imply `t1.id = t3.id`. Combined with `t1.id = 5`, the optimizer can prove that `t2` and `t3` are joined on a constant and eliminate them if their columns are unused.

#### Real-World Cases

- **Tool-generated queries**: Query builders often produce redundant predicates and transitive equalities.
- **View definitions**: Nested views introduce predicates that can be simplified transitively.
- **Semantic optimization**: Integrity constraints (e.g., foreign keys) enable transitive closure and join elimination.


## Summary: Rewrite Decision Matrix

| Rewrite | Primary Trigger | Semantic Risk | Typical Gain |
|---------|-----------------|---------------|--------------|
| Subquery → Join | Correlated subquery in `SELECT`/`WHERE` | `NULL` and duplicate semantics | Eliminates N+1 execution |
| `IN` → `EXISTS` | Large subquery, early termination | `NULL` in `NOT IN` | Short-circuits per row |
| Outer → Inner join | `WHERE` rejects nulls on right side | Null-preservation | Unlocks reordering |
| Join elimination | Unused joined table, unique key | Row multiplication | Removes join cost |
| `DISTINCT` removal | Unique key in `SELECT` list | Join-induced duplicates | Removes sort/hash |
| `GROUP BY` removal | Unique key in grouping columns | Non-unique grouping | Removes aggregation |
| Predicate pushdown | Filter above a view/join/union | Outer-join null semantics | Reduces rows early |
| Constant folding | Literal expressions in predicates | Volatile functions | Removes per-row computation |
| CSE (CTE materialization) | Repeated subquery references | Volatile functions, side effects | Avoids recomputation |
| Transitive closure | Equality chains across joins | `NULL` equality | Enables elimination |


## References

- MySQL 8.0 Reference Manual – WHERE Clause Optimization – https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/where-optimization.html
- PostgreSQL Documentation – WITH Queries (Common Table Expressions) – https://www.postgresql.org/docs/current/queries-with.html
- MariaDB Documentation – Subquery Optimizations – https://mariadb.com/kb/en/subquery-optimizations/
- Oracle Database SQL Tuning Guide – Query Rewrite and Predicate Pushing – https://docs.oracle.com/en/database/oracle/oracle-database/21/tgsql/query-rewrite.html
- CMU 15-415/615 – Query Rewriting Guidelines (Faloutsos) – https://www.cs.cmu.edu/~christos/courses/dbms.S13/slides/19PhysDes.pdf
- Microsoft Learn – Predicate Pushdown – https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-best-practices
- Apache Calcite – Join Elimination Rules – https://issues.apache.org/jira/browse/CALCITE-6162
- Teradata – Outer Join Elimination – https://rd.springer.com/chapter/10.1007/3-540-48300-X_4
- IBM Db2 – Subquery Tuning and Rewriting – https://www.ibm.com/docs/en/db2
- PostgreSQL – Join Removal (join_is_removable) – https://git.postgresql.org/gitweb/?p=postgresql.git
- MySQL 8.0 – Redundant DISTINCT Elimination – https://help.aliyun.com/
- SAP Help – Eliminate Unnecessary DISTINCT Conditions – https://help.sap.com/
- Duke CPS – Heuristics-Based Query Optimization – https://courses.cs.duke.edu/