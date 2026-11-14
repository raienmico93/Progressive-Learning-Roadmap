# Comprehensive Programming Cheat Sheet: Set-Based Thinking

## Topic Overview

### Definitions

**Core Definition**
Set-based thinking is the practice of designing SQL queries by defining the result set as a whole—using relational operations like selection, projection, join, and set union—rather than describing a step-by-step, row-by-row procedure.

**Technical Definition**
Set-based thinking is a declarative problem-solving approach rooted in relational algebra, in which a query specifies *what* result is required, and the database optimiser determines *how* to compute it. The relational model defines a set of fundamental operations—selection (σ), projection (π), Cartesian product (×), union (∪), set difference (−), and join (⨝)—that operate on entire relations (tables) at once. SQL is a declarative language loosely based on these operations; a set-based query composes these operations to transform input relations into an output relation, without prescribing iteration order or procedural control flow. In contrast, row-wise (procedural) logic uses cursors, `WHILE` loops, and per-row conditional statements to process data one row at a time, which conflicts with SQL's set-oriented execution engine.

**Beginner-Friendly Explanation**
Set-based thinking means describing your problem as "what rows do I want in the final answer?" instead of "for each row, do this, then do that." Imagine sorting a deck of cards: you don't pick up each card individually and place it—you spread the whole deck out and rearrange the entire set at once. SQL is designed to work this way, and queries that think in sets are usually shorter, faster, and easier to maintain.

### Key Characteristics

- **Declarative, not procedural**: You state the desired result; the database engine chooses the execution strategy.
- **Whole-relation operations**: Selection, projection, join, union, intersection, and difference operate on entire tables, not individual rows.
- **Order independence**: Sets have no inherent order; row order is meaningful only when an outer `ORDER BY` is specified.
- **Composability**: Relational operations can be nested and combined freely, enabling complex transformations from simple building blocks.
- **Optimiser-friendly**: Set-based queries give the optimiser freedom to reorder joins, push predicates, and choose indexes; procedural loops do not.
- **Performance at scale**: Set-based processing is typically orders of magnitude faster than row-by-row iteration on large data sets.

### Prerequisites

- Understanding of `SELECT`, `WHERE`, `GROUP BY`, `HAVING`, and `ORDER BY`.
- Familiarity with joins (`INNER`, `LEFT`, `FULL`) and set operators (`UNION`, `INTERSECT`, `EXCEPT`).
- Knowledge of subqueries, CTEs (`WITH`), and derived tables.
- Awareness of window functions for analytical calculations without self-joins.
- Basic understanding of relational algebra terminology (selection, projection, join).

### Related Programming Areas

- Relational database theory and relational algebra.
- Query optimisation and execution plans.
- Data warehousing and ETL (extract, transform, load).
- Declarative programming paradigms (functional programming, logic programming).
- Database engine internals (join algorithms, predicate pushdown).

---

## Core Concepts and Features

### 1. Relational Operations

#### Definitions

**Core Definition**
Relational operations are the set of formal operations—selection, projection, Cartesian product, union, set difference, and join—that transform one or more relations (tables) into a new relation, forming the theoretical foundation of SQL.

**Technical Definition**
Relational algebra, originally defined by E.F. Codd, consists of fundamental operations on relations: **selection** (σ) filters rows by a predicate; **projection** (π) selects a subset of columns; **Cartesian product** (×) combines every row of one relation with every row of another; **union** (∪) combines rows from two union-compatible relations; **set difference** (−) returns rows in one relation not in another; **intersection** (∩) returns rows in both; and **join** (⨝) combines related rows from two relations based on a condition. SQL implements these operations through `WHERE` (selection), `SELECT` column lists (projection), `FROM` with multiple tables (product/join), `UNION`/`INTERSECT`/`EXCEPT` (set operations), and `JOIN` syntax. A set-based query composes these operations to express complex transformations declaratively.

**Beginner-Friendly Explanation**
Relational operations are the basic "moves" you can make on tables: pick some rows (`WHERE`), pick some columns (`SELECT`), combine tables (`JOIN`), stack tables (`UNION`), or find common/unique rows (`INTERSECT`/`EXCEPT`). Set-based thinking means composing these moves to describe the result you want, rather than writing a loop to walk through rows one by one.

#### Purposes

- To provide a formal, mathematical foundation for SQL query semantics.
- To enable declarative query composition without procedural control flow.
- To allow the database optimiser to reason about queries algebraically and choose efficient execution plans.
- To express complex data transformations as combinations of simple, well-understood operations.

#### Syntax Rules and Structure

**General mapping of relational operations to SQL**:

```sql
-- Selection (σ): filter rows
SELECT ... FROM table WHERE predicate;

-- Projection (π): choose columns
SELECT col1, col2 FROM table;

-- Cartesian product (×) and Join (⨝)
SELECT ... FROM t1 CROSS JOIN t2;
SELECT ... FROM t1 JOIN t2 ON t1.key = t2.key;

-- Union (∪)
SELECT ... FROM t1 UNION SELECT ... FROM t2;

-- Set difference (−)
SELECT ... FROM t1 EXCEPT SELECT ... FROM t2;

-- Intersection (∩)
SELECT ... FROM t1 INTERSECT SELECT ... FROM t2;
```

**Component breakdown**

| Relational operation | SQL construct | Description |
|----------------------|---------------|-------------|
| Selection (σ) | `WHERE` | Filters rows by a predicate. |
| Projection (π) | `SELECT` column list | Chooses a subset of columns. |
| Cartesian product (×) | `CROSS JOIN` | Combines every row with every row. |
| Join (⨝) | `JOIN ... ON ...` | Combines related rows from two tables. |
| Union (∪) | `UNION` / `UNION ALL` | Combines rows from two queries. |
| Set difference (−) | `EXCEPT` / `MINUS` | Returns rows in one query not in another. |
| Intersection (∩) | `INTERSECT` | Returns rows common to both queries. |

**Syntax rules**

- Set operations require union-compatible operands (same number of columns, compatible types).
- Join conditions belong in the `ON` clause; filter conditions for individual tables belong in `WHERE`.
- Selection and projection can be composed in any order; the optimiser may reorder them.
- Set operations eliminate duplicates by default; use `ALL` to preserve them.

**Constraints and limitations**

- SQL is not a perfect implementation of relational algebra (e.g., it allows duplicate rows, nulls, and ordered results), but the set-based mindset still applies.
- Not every relational operation has a direct SQL keyword (e.g., division requires composition of other operations).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Selection, projection, and join as relational operations

```sql
-- Step 1: Selection (filter active customers).
-- Step 2: Projection (choose customer name and order total).
-- Step 3: Join (combine customers with their orders).
SELECT c.customer_name, o.order_total
FROM customers c
JOIN orders o ON o.customer_id = c.customer_id
WHERE c.status = 'active'
  AND o.order_total > 100;
```

**Expected output** (illustrative):

```text
 customer_name | order_total
---------------+-------------
 Acme Corp     |      150.00
 Globex        |      220.00
(2 rows)
```

**Why this output appears**: The `WHERE` clause implements selection (only active customers with orders over 100); the `SELECT` list implements projection (only two columns); the `JOIN` implements the join operation (combining related rows). The optimiser is free to reorder these operations for efficiency.

##### Example 2: MySQL – Set operations as relational union, intersection, and difference

```sql
-- Union: customers from either year.
SELECT customer_id FROM customers_2024
UNION
SELECT customer_id FROM customers_2025;

-- Intersection: customers in both years.
SELECT customer_id FROM customers_2024
INTERSECT
SELECT customer_id FROM customers_2025;

-- Difference: customers in 2024 but not 2025.
SELECT customer_id FROM customers_2024
EXCEPT
SELECT customer_id FROM customers_2025;
```

**Expected output**: The union returns all distinct customer IDs; the intersection returns only those present in both years; the difference returns those only in 2024.

**Why this output appears**: These queries directly implement the relational operations of union, intersection, and set difference, treating each `SELECT` as a relation.

#### Real-World Cases

- **Reporting**: Combining selection, projection, and join to produce a business report.
- **Data reconciliation**: Using set difference (`EXCEPT`) to find records present in one system but missing from another.
- **Data integration**: Using union (`UNION ALL`) to stack data from multiple source systems.

---

### 2. Difference Between Row-Wise and Set-Wise Logic

#### Definitions

**Core Definition**
Row-wise logic processes data one row at a time using procedural constructs (loops, cursors, per-row conditionals), while set-wise logic operates on entire sets of rows at once using declarative relational operations.

**Technical Definition**
Row-wise (procedural) logic uses `CURSOR`, `WHILE`, `LOOP`, and per-row `IF` statements to iterate over a result set, fetch each row into variables, process it, and move to the next row. This approach is common among developers with a background in procedural languages, but it conflicts with SQL's set-oriented engine. Set-wise (declarative) logic expresses the transformation as a single SQL statement that defines the result set, allowing the optimiser to choose the most efficient execution strategy—often using hash joins, sorts, and parallel scans that process many rows at once. The performance difference is typically orders of magnitude: row-wise processing can take hours where set-based processing takes seconds.

**Beginner-Friendly Explanation**
Row-wise logic is like walking through a warehouse and picking up one box at a time, inspecting it, and putting it down. Set-wise logic is like using a forklift to move the entire pallet at once. SQL's engine is built for the forklift approach; using it one box at a time wastes its power.

#### Purposes

- To understand why procedural loops are often slow in SQL and when they might still be necessary.
- To recognise row-wise patterns in existing code and rewrite them as set-based queries.
- To choose the right approach based on data volume, complexity, and engine capabilities.
- To align query design with the declarative nature of SQL and the optimiser's strengths.

#### Syntax Rules and Structure

**Row-wise pattern (to be avoided)**:

```sql
-- Procedural cursor loop (pseudo-code)
DECLARE cursor c FOR SELECT id, amount FROM orders;
OPEN c;
LOOP
    FETCH c INTO @id, @amount;
    IF @amount > 100 THEN
        UPDATE orders SET status = 'priority' WHERE id = @id;
    END IF;
END LOOP;
CLOSE c;
```

**Set-wise equivalent**:

```sql
UPDATE orders
SET status = 'priority'
WHERE amount > 100;
```

**Component breakdown**

| Aspect | Row-wise | Set-wise |
|--------|----------|----------|
| Processing unit | One row at a time | Entire set of rows |
| Control flow | Explicit loops and conditionals | Declarative predicates and expressions |
| Optimiser freedom | Limited (forced order) | Full (reorder, index, parallelise) |
| Typical performance | Slow, scales linearly or worse | Fast, leverages set algorithms |
| Code length | Longer, more error-prone | Shorter, easier to verify |

**Syntax rules**

- Avoid `CURSOR`, `WHILE`, and row-by-row `LOOP` constructs unless they are genuinely necessary (e.g., calling a stored procedure per row that cannot be set-based).
- Express filters in `WHERE`, aggregations in `GROUP BY`, and transformations in `SELECT` expressions.
- Use window functions for running totals and rankings instead of correlated subqueries or loops.

**Constraints and limitations**

- Some operations (e.g., calling an external API per row, complex procedural logic with side effects) may still require row-wise processing.
- Rewriting procedural code requires careful validation to preserve business rules.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Cursor vs set-based update

```sql
-- Row-wise: cursor loops through each row.
DECLARE @OrderId INT, @Amount DECIMAL(10,2);
DECLARE cur CURSOR FOR SELECT OrderId, Amount FROM OrderCharge WHERE Fee IS NULL;
OPEN cur;
FETCH NEXT FROM cur INTO @OrderId, @Amount;
WHILE @@FETCH_STATUS = 0
BEGIN
    UPDATE OrderCharge
    SET Fee = ROUND(@Amount * (SELECT Rate FROM CustomerRate WHERE CustomerId = ...), 2)
    WHERE OrderId = @OrderId;
    FETCH NEXT FROM cur INTO @OrderId, @Amount;
END
CLOSE cur; DEALLOCATE cur;

-- Set-based: single UPDATE with JOIN.
UPDATE oc
SET Fee = ROUND(oc.Amount * cr.Rate, 2)
FROM dbo.OrderCharge AS oc
JOIN dbo.CustomerRate AS cr
  ON cr.CustomerId = oc.CustomerId AND cr.IsActive = 1
WHERE oc.Fee IS NULL;
```

**Expected output**: Both approaches update the same rows, but the set-based `UPDATE` performs a single join operation, while the cursor executes one `UPDATE` per row, resulting in dramatically slower performance on large tables.

**Why this output appears**: The set-based `UPDATE` lets the database use a hash join or nested loop join to match orders to rates in one pass; the cursor forces row-by-row processing with repeated lookups.

##### Example 2: Oracle – PL/SQL loop vs single `INSERT ... SELECT`

```sql
-- Row-wise: loop over external table and insert one row at a time.
FOR rec IN (SELECT * FROM external_table) LOOP
    IF rec.valid = 1 THEN
        INSERT INTO production_table VALUES rec.*;
    ELSE
        INSERT INTO error_log VALUES rec.*;
    END IF;
END LOOP;

-- Set-based: single INSERT with conditional logic.
INSERT ALL
    WHEN valid = 1 THEN INTO production_table VALUES (...)
    ELSE INTO error_log VALUES (...)
SELECT * FROM external_table;
```

**Expected output**: Both load the data, but the set-based `INSERT ALL` processes the entire external table in one operation, avoiding per-row PL/SQL context switches and leveraging direct-path load.

**Why this output appears**: The set-based version expresses the branching logic declaratively in the `WHEN` clauses, allowing the database to evaluate all rows together.

#### Real-World Cases

- **ETL loading**: Replacing row-by-row inserts with `INSERT ... SELECT` or `MERGE`.
- **Data cleanup**: Using `UPDATE` with `JOIN` instead of cursor loops.
- **Reporting**: Using window functions and aggregations instead of procedural calculations.

---

### 3. Replacing Procedural Loops with Set Operations

#### Definitions

**Core Definition**
Replacing procedural loops with set operations means rewriting cursor- or loop-based code as declarative SQL that uses joins, aggregations, set operators, and window functions to achieve the same result in a single statement.

**Technical Definition**
Procedural loops (cursors, `WHILE` loops, `FOR` loops) process data row by row, which prevents the optimiser from using set-based algorithms and incurs per-row overhead (context switches, repeated lookups, network round-trips if the loop is in application code). Replacing them requires identifying the relational operation that each iteration performs—often a join, an aggregation, a window calculation, or a set operation—and expressing it in a single SQL statement. The set-based version allows the database to choose hash joins, sorts, parallel scans, and other optimisations that are orders of magnitude faster on large data sets.

**Beginner-Friendly Explanation**
If your code says "for each order, look up the customer's rate and update the fee," you can usually replace it with a single `UPDATE ... FROM ... JOIN` that does the lookup and update for all orders at once. The database is much better at doing things in bulk than one at a time.

#### Purposes

- To eliminate the performance bottleneck of row-by-row processing.
- To reduce code volume and complexity, making queries easier to maintain.
- To allow the database optimiser to choose efficient join and aggregation strategies.
- To avoid network round-trips between application and database that occur when processing rows in application code.

#### Syntax Rules and Structure

**Common rewrites**:

| Procedural pattern | Set-based equivalent |
|--------------------|----------------------|
| Cursor + per-row `UPDATE` | `UPDATE ... FROM ... JOIN ... WHERE ...` |
| Cursor + per-row `INSERT` | `INSERT INTO ... SELECT ...` |
| Loop with `IF` to choose destination | `INSERT ALL WHEN ...` or `UNION ALL` with discriminator |
| Loop computing running totals | Window function (`SUM(...) OVER (ORDER BY ...)`) |
| Loop finding missing records | `NOT EXISTS` or `LEFT JOIN ... IS NULL` |
| Loop deduplicating rows | `ROW_NUMBER() OVER (PARTITION BY ...)` with `rn = 1` |

**Component breakdown**

| Rewrite step | Description |
|--------------|-------------|
| Identify per-row operation | What does each iteration do? (lookup, update, insert) |
| Express as relational operation | Join, aggregate, window function, set operation |
| Verify grain and semantics | Ensure the set-based result matches the loop's output |
| Test with edge cases | NULLs, missing lookups, duplicates |

**Syntax rules**

- Use `UPDATE ... FROM` (PostgreSQL, SQL Server) or `UPDATE ... JOIN` (MySQL) for per-row updates.
- Use `INSERT ... SELECT` for per-row inserts.
- Use `MERGE` for combined insert/update/delete logic.
- Use window functions for running totals and rankings instead of correlated subqueries or loops.
- Use `NOT EXISTS` for anti-join logic instead of looping and checking existence.

**Constraints and limitations**

- Some procedural logic (calling an external API per row, complex sequential state) may not be expressible in a single set-based statement.
- Rewrites must preserve business rules, especially around NULLs, ordering, and duplicate handling.
- Always preview the join results before running a set-based `UPDATE` or `DELETE` to avoid unintended data changes.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Replacing a nested cursor with a set-based update

```sql
-- Original nested cursor (simplified):
DECLARE outerCursor CURSOR FOR SELECT EntityId, BaseId FROM outerTable;
OPEN outerCursor;
FETCH NEXT FROM outerCursor INTO @EntityId, @BaseId;
WHILE @@FETCH_STATUS = 0
BEGIN
    DECLARE innerCursor CURSOR FOR SELECT PRFMR_ID FROM innerTable WHERE ENTY_ID = @BaseId;
    OPEN innerCursor;
    FETCH NEXT FROM innerCursor INTO @PerfId;
    WHILE @@FETCH_STATUS = 0
    BEGIN
        UPDATE 200MilRowTable SET ENTY_ID = @EntityId WHERE PRFMR_ID = @PerfId;
        FETCH NEXT FROM innerCursor INTO @PerfId;
    END
    CLOSE innerCursor; DEALLOCATE innerCursor;
    FETCH NEXT FROM outerCursor INTO @EntityId, @BaseId;
END
CLOSE outerCursor; DEALLOCATE outerCursor;

-- Set-based rewrite:
UPDATE t
SET ENTY_ID = o.EntityId
FROM 200MilRowTable t
JOIN innerTable i ON t.PRFMR_ID = i.PRFMR_ID
JOIN outerTable o ON i.ENTY_ID = o.BaseId;
```

**Expected output**: Both update the same 200-million-row table, but the set-based version completes in seconds or minutes, while the nested cursor had been running for over 30 hours and processed only 0.5% of the outer rows.

**Why this output appears**: The nested cursor performs one `UPDATE` per inner-table match, resulting in millions of individual statements. The set-based `UPDATE ... FROM` performs a single join across all three tables, leveraging the optimiser's choice of join algorithm.

##### Example 2: PostgreSQL – Replacing a loop with `INSERT ... SELECT`

```sql
-- Procedural loop (PL/pgSQL):
FOR rec IN SELECT * FROM staging_orders WHERE status = 'new' LOOP
    INSERT INTO orders (order_id, customer_id, amount)
    VALUES (rec.order_id, rec.customer_id, rec.amount);
END LOOP;

-- Set-based equivalent:
INSERT INTO orders (order_id, customer_id, amount)
SELECT order_id, customer_id, amount
FROM staging_orders
WHERE status = 'new';
```

**Expected output**: Both insert the same rows, but the set-based `INSERT ... SELECT` performs a single bulk insert, which is significantly faster and reduces transaction overhead.

**Why this output appears**: The loop executes one `INSERT` per row, incurring per-statement overhead; the set-based version processes all qualifying rows in one operation.

#### Real-World Cases

- **Order processing**: Replacing cursor-based order status updates with a single `UPDATE ... JOIN`.
- **Data migration**: Replacing row-by-row inserts with `INSERT ... SELECT` or `MERGE`.
- **Sessionization**: Replacing procedural grouping with window functions (`LAG`, `SUM ... OVER`).

---

### 4. Query Composition

#### Definitions

**Core Definition**
Query composition is the practice of building complex SQL queries by combining smaller, well-defined query blocks—subqueries, CTEs, derived tables, and set operations—rather than writing one monolithic, deeply nested statement.

**Technical Definition**
Query composition uses the structural elements of SQL to express multi-step logic declaratively: **subqueries** in `FROM` (derived tables) or `WHERE` (scalar/`IN`/`EXISTS` subqueries), **Common Table Expressions** (`WITH` clauses) that define named temporary result sets reusable within the query, and **set operators** (`UNION`, `INTERSECT`, `EXCEPT`) that combine result sets. Each composed block can be reasoned about independently, tested in isolation, and optimised by the database as part of the whole query. CTEs are particularly useful for separating logic into named stages, making the query read like a pipeline of transformations.

**Beginner-Friendly Explanation**
Query composition is like building with LEGO blocks: instead of one giant, tangled piece, you build small, named pieces (CTEs) and snap them together. Each piece is easy to understand and test on its own, and the final result is the combination of all the pieces.

#### Purposes

- To break complex logic into readable, testable, named stages.
- To avoid deeply nested subqueries that are hard to read and maintain.
- To reuse intermediate results multiple times without recomputation (via CTEs).
- To allow the optimiser to see the whole query and choose efficient execution strategies.

#### Syntax Rules and Structure

**CTE pipeline pattern**:

```sql
WITH
  stage1 AS (
      SELECT ... FROM source WHERE ...
  ),
  stage2 AS (
      SELECT ... FROM stage1 JOIN other ...
  ),
  stage3 AS (
      SELECT ... FROM stage2 GROUP BY ...
  )
SELECT ... FROM stage3;
```

**Derived table pattern**:

```sql
SELECT ...
FROM (
    SELECT ... FROM source WHERE ...
) AS sub
JOIN other ON ...;
```

**Set operation composition**:

```sql
SELECT ... FROM t1
UNION ALL
SELECT ... FROM t2
WHERE ...;
```

**Component breakdown**

| Composition tool | Use case | Reusability |
|------------------|----------|-------------|
| CTE (`WITH`) | Multi-step pipelines, named stages | Can be referenced multiple times |
| Derived table | Single-use inline subquery | Single reference |
| Subquery in `WHERE` | Filtering with `IN`/`EXISTS` | Single reference |
| Set operator | Combining result sets | Combines entire queries |

**Syntax rules**

- CTEs are defined at the beginning of the query and can reference earlier CTEs.
- Derived tables require an alias.
- Set operators require union-compatible operands.
- The final query can reference any CTE defined earlier.

**Constraints and limitations**

- Overusing CTEs can obscure performance characteristics; some engines materialise CTEs, others inline them.
- Deeply nested derived tables reduce readability; prefer CTEs for multi-step logic.
- Not all engines support all set operators (e.g., MySQL supports `INTERSECT`/`EXCEPT` only from 8.0.31 onward).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – CTE pipeline for multi-step transformation

```sql
WITH active_orders AS (
    SELECT id, customer_id, total
    FROM orders
    WHERE status = 'active'
),
with_discounts AS (
    SELECT ao.id, ao.customer_id,
           ao.total * (1 - COALESCE(d.pct, 0)) AS discounted_total
    FROM active_orders ao
    LEFT JOIN discounts d USING (customer_id)
),
aggregated AS (
    SELECT customer_id, SUM(discounted_total) AS revenue
    FROM with_discounts
    GROUP BY customer_id
)
SELECT * FROM aggregated ORDER BY revenue DESC;
```

**Expected output** (illustrative):

```text
 customer_id | revenue
-------------+---------
         104 | 12500.00
         112 |  8900.00
          98 |  4500.00
(3 rows)
```

**Why this output appears**: Each CTE performs one logical transformation: `active_orders` filters, `with_discounts` applies discounts, and `aggregated` sums per customer. The final query reads from the last stage, and the optimiser can inline or materialise stages as appropriate.

##### Example 2: SQL Server – Replacing nested subqueries with CTEs

```sql
-- Deeply nested derived tables (hard to read):
SELECT ...
FROM (
    SELECT ...
    FROM (
        SELECT ... FROM source WHERE ...
    ) AS inner1
    WHERE ...
) AS inner2
WHERE ...;

-- CTE pipeline (readable, composable):
WITH base AS (
    SELECT ... FROM source WHERE ...
),
filtered AS (
    SELECT ... FROM base WHERE ...
),
final AS (
    SELECT ... FROM filtered WHERE ...
)
SELECT ... FROM final;
```

**Expected output**: Same result, but the CTE version names each stage and makes the logic easy to follow.

**Why this output appears**: CTEs flatten the nesting into a linear sequence of named stages, improving readability and allowing independent testing of each stage.

#### Real-World Cases

- **ETL pipelines**: Building transformations as a chain of CTEs.
- **Reporting queries**: Separating filtering, aggregation, and formatting into stages.
- **Complex analytics**: Using CTEs to compute multiple metrics and join them in a final query.

---

## References

- Joe Celko's Thinking in Sets: Auxiliary, Temporal, and Virtual Tables in SQL – https://www.google.com/books/edition/Thinking_in_Sets
- Oracle 2 Day Developer's Guide: Set-Based Processing – https://docs.oracle.com/en/database/oracle/oracle-database/21/tdddg/building-effective-applications.html
- SQL Authority: Solving SQL Puzzles With Set-Based Thinking – https://blog.sqlauthority.com/2008/03/20/solving-sql-puzzles-with-set-based-thinking/
- SQL Authority: Replacing a Cursor With a Set-Based Query – https://blog.sqlauthority.com/2007/07/25/replacing-a-cursor-with-a-set-based-query/
- SAP HANA Performance Guide: Replacing Row-Based Calculations with Set-Based Calculations – https://help.sap.com/docs/SAP_HANA_PLATFORM/9de0171a6027400bb3b9bee385222eff/2b12462ffa2040569aaccb1ae3fa6119.html
- Microsoft Learn: Increase Your SQL Server Performance by Replacing Cursors with Set Operations – https://learn.microsoft.com/en-us/archive/blogs/mssqlisv/increase-your-sql-server-performance-by-replacing-cursors-with-set-operations
- Ask TOM: Think in Sets – https://asktom.oracle.com/ords/asktom.search?tag=think-in-sets
- Itzik Ben-Gan, T-SQL Best Practices, Part 1: Think in Sets – https://prod.itprotoday.com/sql-server/t-sql-best-practices-part-1
- PostgreSQL Documentation: WITH Queries (Common Table Expressions) – https://www.postgresql.org/docs/current/queries-with.html
- Microsoft Learn: Use Common Table Expressions – https://learn.microsoft.com/en-us/training/modules/implement-subqueries/3-use-common-table-expressions
- PostgreSQL — FP Patterns: CTEs as Named Pipeline Stages, Set-Based Operations Over Row-by-Row Cursors – https://cdn.jsdelivr.net/npm/pi-rnd@0.4.7/skills/fp-practices/postgresql.md
- Relational Algebra Operations – https://www.sciencedirect.com/topics/computer-science/relational-algebra-operation
- CS 317/387: Relational Algebra and SQL – https://www.cse.iitb.ac.in/~sudarsha/db-book/slide-dir/ch5.pdf