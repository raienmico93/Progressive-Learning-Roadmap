# Comprehensive Programming Cheat Sheet: Multiple CTEs

## Topic Overview

### Definitions

**Core Definition**  
Multiple CTEs is the use of two or more Common Table Expressions within a single `WITH` clause, where each named result set can be referenced by the main statement and (subject to ordering rules) by subsequent CTEs.

**Technical Definition**  
A `WITH` clause may declare a comma‑separated list of CTE definitions. Each CTE is a named query expression with statement‑level scope. A CTE may reference any CTE defined earlier in the same `WITH` clause (forward dependency), and, when `WITH RECURSIVE` is used, a CTE may reference itself (self dependency). Multiple CTEs form a directed acyclic dependency graph (excluding self‑references in recursive CTEs), and the RDBMS resolves dependencies in declaration order. Data‑modifying CTEs (PostgreSQL, SQL Server) may also appear in the list, subject to vendor restrictions.

**Beginner-Friendly Explanation**  
Multiple CTEs let you write a query as a series of named steps: first compute step one, then step two that uses step one, then step three that uses step two, and finally the main query that uses the last step. Each step is a CTE, and together they form a readable pipeline.

### Key Characteristics

- **Comma‑separated list**: CTEs in one `WITH` clause are separated by commas.
- **Declaration order matters**: A CTE can reference only CTEs declared before it (plus itself when recursive).
- **Shared scope**: All CTEs in the clause share the statement‑level scope and are visible to the main statement.
- **Unique names**: Each CTE name must be unique within the `WITH` clause.
- **Dependency graph**: Dependencies between CTEs form a DAG resolved at parse/plan time.
- **Optional column lists**: Each CTE may declare its own column aliases independently.
- **Vendor restrictions**: Some engines limit which statement types can appear in a CTE list or how data‑modifying CTEs interact.

### Prerequisites

- Understanding of single‑CTE syntax and scope.
- Knowledge of `SELECT`, joins, and aggregation.
- Familiarity with the target RDBMS’s rules on CTE ordering and data‑modifying CTEs.
- Awareness of optimiser behaviour (inlining vs materialisation) in the chosen engine.

### Related Programming Areas

- Query composition and pipeline design.
- ETL and data transformation workflows.
- Recursive and hierarchical query design.
- Query optimisation and execution‑plan analysis.
- View and stored‑procedure design.

---

## Core Concepts and Features

### 1. Multiple CTEs in One Query

#### Definitions

**Core Definition**  
Multiple CTEs in one query means declaring more than one CTE in a single `WITH` clause before the main statement.

**Technical Definition**  
The `WITH` clause accepts a comma‑separated list of CTE definitions: `WITH cte1 AS (...), cte2 AS (...), cte3 AS (...) main_statement;`. All CTEs share the same statement scope, and the main statement may reference any of them. The optimiser treats the list as a set of derived relations whose dependencies are resolved in declaration order.

**Beginner-Friendly Explanation**  
Instead of one helper query, you define several helper queries at the top of your statement, each with its own name, and then use them in the final query.

#### Purposes

- To structure a complex query as several named, independent building blocks.
- To separate concerns such as filtering, aggregating, and enriching.
- To reference intermediate results multiple times without duplication.
- To improve readability and testability of long queries.

#### Syntax Rules and Structure

**General syntax**:

```sql
WITH
  cte_name1 [(col1, col2, ...)] AS (
      query_expression1
  ),
  cte_name2 [(col1, col2, ...)] AS (
      query_expression2
  ),
  cte_name3 [(col1, col2, ...)] AS (
      query_expression3
  )
main_statement;
```

**PostgreSQL** (official):

```sql
WITH [ RECURSIVE ] with_query [, ...]
main_query;

with_query:
    query_name [ ( column_name [, ...] ) ] AS [ [ NOT ] MATERIALIZED ] ( query )
```

**SQL Server** (official):

```sql
[ WITH <common_table_expression> [ ,...n ] ]
<common_table_expression>::=
    expression_name [ ( column_name [ ,...n ] ) ]
    AS
    ( CTE_query_definition )
```

**MySQL 8.0+** (official):

```sql
WITH [RECURSIVE]
    cte_name [(col_name [, col_name] ...)] AS (subquery)
    [, cte_name [(col_name [, col_name] ...)] AS (subquery)] ...
SELECT ...;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `WITH` | Introduces the CTE list. |
| `cte_name1, cte_name2, ...` | Unique names for each CTE. |
| `(col1, col2, ...)` | Optional per‑CTE column aliases. |
| `AS (query_expression)` | Each CTE’s defining query. |
| `,` | Separates CTE definitions. |
| `main_statement` | The `SELECT`, `INSERT`, `UPDATE`, `DELETE`, or `MERGE` that consumes the CTEs. |

**Syntax rules**

- Each CTE name must be unique within the `WITH` clause.
- Commas separate CTE definitions; a trailing comma before the main statement is a syntax error.
- The main statement can reference any CTE in the list (subject to vendor rules on data‑modifying CTEs).
- In SQL Server, the `WITH` clause must be preceded by a semicolon if it is not the first statement in a batch.
- In MySQL 8.0+, `WITH` may precede `SELECT`, `UPDATE`, and `DELETE`.

**Constraints and limitations**

- A CTE cannot be referenced outside the statement.
- In SQL Server, `ORDER BY` inside a CTE requires `TOP` or `OFFSET/FETCH`.
- In PostgreSQL, data‑modifying CTEs must include a `RETURNING` clause if their output is referenced.
- MySQL 8.4 does not support data‑modifying CTEs (e.g., `WITH … INSERT`).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Two independent CTEs joined in the main query

```sql
-- Step 1: Define a CTE for completed orders.
WITH completed_orders AS (
    SELECT order_id, customer_id, amount
    FROM orders
    WHERE status = 'completed'
),
-- Step 2: Define an independent CTE for active customers.
active_customers AS (
    SELECT customer_id, name
    FROM customers
    WHERE is_active = TRUE
)
-- Step 3: Join the two CTEs in the main query.
SELECT c.name, COUNT(o.order_id) AS completed_count, SUM(o.amount) AS total
FROM active_customers c
LEFT JOIN completed_orders o ON o.customer_id = c.customer_id
GROUP BY c.name
ORDER BY total DESC NULLS LAST;
```

**Expected output** (illustrative):

```text
   name    | completed_count |   total
-----------+-----------------+-----------
 Acme Corp |               7 |  12345.00
 Globex    |               4 |   4321.50
 Initech   |               0 |
(3 rows)
```

**Why this output appears**: `completed_orders` and `active_customers` are independent CTEs; the `LEFT JOIN` preserves active customers with no completed orders, and `SUM` returns `NULL` for those rows.

##### Example 2: SQL Server – Three CTEs forming a pipeline

```sql
-- Step 1: Base CTE filters recent orders.
;WITH RecentOrders AS (
    SELECT order_id, customer_id, amount
    FROM Orders
    WHERE order_date >= DATEADD(day, -30, GETDATE())
),
-- Step 2: Aggregate per customer.
CustomerTotals AS (
    SELECT customer_id, SUM(amount) AS total
    FROM RecentOrders
    GROUP BY customer_id
),
-- Step 3: Enrich with customer names.
Enriched AS (
    SELECT c.customer_id, c.customer_name, t.total
    FROM Customers c
    JOIN CustomerTotals t ON t.customer_id = c.customer_id
)
-- Step 4: Main query.
SELECT customer_name, total
FROM Enriched
ORDER BY total DESC;
```

**Expected output**:

```text
customer_name   total
--------------- ---------
Acme Corp       58200.00
Globex          41300.50
Initech          9800.00
```

**Why this output appears**: Each CTE references only earlier CTEs, forming a linear pipeline; the main query sorts the enriched result.

##### Example 3: MySQL 8.0 – Multiple CTEs with column aliases

```sql
WITH
  order_totals (cust_id, total) AS (
      SELECT customer_id, SUM(amount)
      FROM orders
      GROUP BY customer_id
  ),
  high_value (cust_id, total) AS (
      SELECT cust_id, total
      FROM order_totals
      WHERE total > 500
  )
SELECT cust_id, total
FROM high_value
ORDER BY total DESC;
```

**Expected output**:

```text
+---------+--------+
| cust_id | total  |
+---------+--------+
|     104 | 1234.5 |
|     112 |  789.0 |
+---------+--------+
2 rows in set (0.00 sec)
```

**Why this output appears**: The first CTE computes totals and renames its columns; the second CTE filters the first; the main query sorts the result.

#### Real-World Cases

- **Data warehouse pipelines**: A query is structured as staging → cleaned → aggregated → final CTEs.
- **Reporting**: Independent CTEs compute metrics from different source tables and are joined in the final query.
- **Testing**: Each CTE can be run independently to verify intermediate results.

---

### 2. CTE Referencing CTEs

#### Definitions

**Core Definition**  
CTE referencing CTEs means one CTE’s query expression uses another CTE by name as a relation.

**Technical Definition**  
Within a `WITH` clause, a CTE may reference any CTE declared earlier in the same clause. The reference is resolved to the named derived relation; dependencies are evaluated in declaration order. A CTE may also reference itself when `WITH RECURSIVE` is used. The referenced CTE acts as a logical table within the referencing CTE’s query.

**Beginner-Friendly Explanation**  
One helper query can use the result of a previous helper query, so you can build step two on top of step one.

#### Purposes

- To build a pipeline where each CTE transforms the output of the previous one.
- To reuse intermediate results across multiple downstream CTEs.
- To isolate each transformation in a separately named step.
- To express complex logic as a sequence of simple, dependent queries.

#### Syntax Rules and Structure

**General pattern**:

```sql
WITH
  cte1 AS (
      SELECT ...
  ),
  cte2 AS (
      SELECT ... FROM cte1 ...   -- cte2 references cte1
  ),
  cte3 AS (
      SELECT ... FROM cte2 ...   -- cte3 references cte2
  )
SELECT ... FROM cte3;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `cte1` | Base CTE with no dependency on other CTEs. |
| `cte2` | References `cte1` in its `FROM` (or other clauses). |
| `cte3` | References `cte2`; may also reference `cte1` directly. |

**Syntax rules**

- A CTE may reference only CTEs declared before it.
- A CTE may reference multiple earlier CTEs in joins or subqueries.
- A CTE may reference itself only in `WITH RECURSIVE` and only within its recursive term.
- The referenced CTE name resolves as a relation; it can be aliased in the referencing query.

**Constraints and limitations**

- Forward references (referencing a later CTE) are not allowed except in recursive self‑references.
- In SQL Server, a CTE cannot reference itself unless `WITH RECURSIVE`‑like behaviour is expressed through recursion (SQL Server does not use the `RECURSIVE` keyword; it infers recursion).
- Some engines restrict data‑modifying CTEs from referencing other data‑modifying CTEs.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Chain of dependent CTEs

```sql
-- Step 1: cte1 filters raw orders.
WITH cte1 AS (
    SELECT order_id, customer_id, amount
    FROM orders
    WHERE amount > 0
),
-- Step 2: cte2 aggregates cte1.
cte2 AS (
    SELECT customer_id, SUM(amount) AS total
    FROM cte1
    GROUP BY customer_id
),
-- Step 3: cte3 filters cte2.
cte3 AS (
    SELECT customer_id, total
    FROM cte2
    WHERE total >= 1000
)
-- Step 4: Main query uses cte3.
SELECT customer_id, total
FROM cte3
ORDER BY total DESC;
```

**Expected output**:

```text
 customer_id |  total
-------------+---------
         104 | 12345.00
         112 |  4321.50
(2 rows)
```

**Why this output appears**: Each CTE references the immediately preceding one; the chain resolves in declaration order, and the final CTE filters aggregated totals.

##### Example 2: SQL Server – A CTE referencing two earlier CTEs

```sql
;WITH OrdersCTE AS (
    SELECT customer_id, amount
    FROM Orders
    WHERE order_date >= '2025-01-01'
),
CustomersCTE AS (
    SELECT customer_id, customer_name, region
    FROM Customers
),
-- Combined references both earlier CTEs.
Combined AS (
    SELECT c.customer_name, c.region, SUM(o.amount) AS total
    FROM CustomersCTE c
    LEFT JOIN OrdersCTE o ON o.customer_id = c.customer_id
    GROUP BY c.customer_name, c.region
)
SELECT customer_name, region, total
FROM Combined
ORDER BY total DESC;
```

**Expected output**:

```text
customer_name   region  total
--------------- ------- ---------
Acme Corp       North   58200.00
Globex          South   41300.50
Initech         West     NULL
```

**Why this output appears**: `Combined` references both `CustomersCTE` and `OrdersCTE`, which are declared earlier. The `LEFT JOIN` preserves customers without orders.

##### Example 3: MySQL 8.0 – CTE referencing an earlier CTE in a subquery

```sql
WITH base AS (
    SELECT product_id, category_id, price
    FROM products
    WHERE price > 10
),
filtered AS (
    SELECT product_id, price
    FROM base
    WHERE category_id IN (
        SELECT category_id FROM categories WHERE is_active = 1
    )
)
SELECT product_id, price
FROM filtered
ORDER BY price DESC;
```

**Expected output**:

```text
+------------+-------+
| product_id | price |
+------------+-------+
|        201 | 99.99 |
|        202 | 49.99 |
+------------+-------+
2 rows in set (0.00 sec)
```

**Why this output appears**: `filtered` references `base`; the subquery on `categories` is independent. The dependency is resolved in declaration order.

#### Real-World Cases

- **Layered transformations**: Raw → cleaned → aggregated → ranked, each layer a CTE referencing the previous.
- **Multi‑source joins**: A CTE references two earlier CTEs to combine data from different pipelines.
- **Recursive expansion**: A recursive CTE references itself and may reference earlier CTEs for base data.

---

### 3. CTE Dependencies

#### Definitions

**Core Definition**  
CTE dependencies describe which CTEs a given CTE relies on, and the order in which they must be resolved.

**Technical Definition**  
Dependencies form a directed graph: an edge from CTE A to CTE B exists if A references B. The graph must be acyclic except for self‑loops introduced by recursive CTEs. The RDBMS resolves dependencies in declaration order; a CTE cannot depend on a CTE declared later. The optimiser may inline, materialise, or share evaluation of dependencies depending on the engine and hints.

**Beginner-Friendly Explanation**  
Dependencies tell you which helper query must come first: if step two uses step one, then step one must be written before step two.

#### Purposes

- To ensure that each CTE’s inputs are defined before it is evaluated.
- To reason about evaluation order and performance.
- To identify opportunities for reuse and shared computation.
- To detect cycles (other than intentional recursion).

#### Syntax Rules and Structure

**Dependency pattern**:

```sql
WITH
  a AS (SELECT ...),                    -- no dependencies
  b AS (SELECT ... FROM a ...),         -- depends on a
  c AS (SELECT ... FROM a JOIN b ...),  -- depends on a and b
  d AS (SELECT ... FROM c ...)          -- depends on c
SELECT ... FROM d;
```

**Component breakdown**

| CTE | Depends on |
|-----|------------|
| `a` | none |
| `b` | `a` |
| `c` | `a`, `b` |
| `d` | `c` |

**Syntax rules**

- A CTE can reference only earlier CTEs; this induces a topological order equal to declaration order.
- Recursive CTEs introduce self‑dependencies only within the recursive term.
- Mutual recursion between two CTEs is not allowed in the `WITH` clause of standard SQL engines.
- Data‑modifying CTEs may have additional dependency restrictions (e.g., a data‑modifying CTE cannot reference another data‑modifying CTE in PostgreSQL).

**Constraints and limitations**

- Circular dependencies among non‑recursive CTEs are syntax/semantic errors.
- Some engines require that a recursive CTE’s recursive term reference the CTE exactly once.
- Dependency order affects materialisation decisions; inlining may propagate predicates into dependencies.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Explicit dependency chain

```sql
-- Dependency graph:
--   base  ->  cleaned  ->  aggregated  ->  final
WITH base AS (
    SELECT order_id, customer_id, amount, status
    FROM orders
),
cleaned AS (
    SELECT order_id, customer_id, amount
    FROM base
    WHERE status = 'completed'
),
aggregated AS (
    SELECT customer_id, SUM(amount) AS total
    FROM cleaned
    GROUP BY customer_id
),
final AS (
    SELECT customer_id, total
    FROM aggregated
    WHERE total > 1000
)
SELECT * FROM final ORDER BY total DESC;
```

**Expected output**:

```text
 customer_id |  total
-------------+---------
         104 | 12345.00
         112 |  4321.50
(2 rows)
```

**Why this output appears**: Each CTE depends only on its predecessor; the chain is topologically ordered by declaration.

##### Example 2: SQL Server – A CTE depending on two earlier CTEs

```sql
;WITH OrdersCTE AS (
    SELECT customer_id, amount FROM Orders
),
CustomersCTE AS (
    SELECT customer_id, customer_name FROM Customers
),
Summary AS (   -- depends on both earlier CTEs
    SELECT c.customer_name, SUM(o.amount) AS total
    FROM CustomersCTE c
    LEFT JOIN OrdersCTE o ON o.customer_id = c.customer_id
    GROUP BY c.customer_name
)
SELECT * FROM Summary ORDER BY total DESC;
```

**Expected output**:

```text
customer_name   total
--------------- ---------
Acme Corp       58200.00
Globex          41300.50
Initech          NULL
```

**Why this output appears**: `Summary` depends on `OrdersCTE` and `CustomersCTE`, both declared earlier; the dependency graph is acyclic and resolvable in order.

##### Example 3: PostgreSQL – Detecting a circular dependency error

```sql
-- This statement is invalid: a references b before b is declared.
WITH a AS (SELECT * FROM b),
     b AS (SELECT 1 AS x)
SELECT * FROM a;
```

**Expected output**:

```text
ERROR:  relation "b" does not exist
LINE 2: WITH a AS (SELECT * FROM b),
```

**Why this output appears**: `a` references `b`, which is declared later; PostgreSQL resolves names in declaration order and reports that `b` does not exist at the point of reference.

#### Real-World Cases

- **Pipeline debugging**: Understanding dependencies helps isolate which CTE produces unexpected results.
- **Performance tuning**: Materialising a heavily used dependency can reduce repeated computation.
- **Code review**: Dependency graphs make the data flow explicit.

---

### 4. CTE Ordering

#### Definitions

**Core Definition**  
CTE ordering is the sequence in which CTEs are declared in the `WITH` clause, which determines which references are legal.

**Technical Definition**  
Declaration order defines a topological order: a CTE may reference only CTEs that appear earlier. The RDBMS may evaluate CTEs in any order permitted by dependencies (subject to inlining and materialisation), but name resolution follows declaration order. Recursive CTEs are exempt only for self‑references within the recursive term.

**Beginner-Friendly Explanation**  
You must write each helper query after the helper queries it uses; the order of declaration is the order of dependency.

#### Purposes

- To ensure that every referenced CTE is already defined.
- To express the logical flow of a query from raw data to final result.
- To satisfy vendor parser rules that resolve names in declaration order.
- To make dependencies visually obvious to readers.

#### Syntax Rules and Structure

**Correct ordering**:

```sql
WITH
  base AS (...),        -- first: no dependencies
  step1 AS (... FROM base ...),
  step2 AS (... FROM step1 ...),
  final AS (... FROM step2 ...)
SELECT ... FROM final;
```

**Incorrect ordering**:

```sql
WITH
  step2 AS (... FROM step1 ...),  -- ERROR: step1 not yet defined
  step1 AS (...)
SELECT ... FROM step2;
```

**Component breakdown**

| Position | CTE | Rule |
|----------|-----|------|
| 1 | base | No dependency on other CTEs. |
| 2 | step1 | May reference base. |
| 3 | step2 | May reference base and step1. |
| 4 | final | May reference any earlier CTE. |

**Syntax rules**

- Declaration order is the only legal reference order (aside from recursion).
- Independent CTEs may be declared in any order relative to each other.
- A CTE may reference any subset of earlier CTEs, not just the immediately preceding one.
- In recursive CTEs, the self‑reference is permitted regardless of position, but base data CTEs must be declared before the recursive CTE if referenced.

**Constraints and limitations**

- Reordering a CTE before its dependencies causes a name‑resolution error.
- Some engines may report the error at parse time, others at planning time.
- Ordering does not dictate evaluation order; the optimiser may reorder evaluation subject to dependencies.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Correct declaration order

```sql
WITH
  raw AS (SELECT * FROM orders),
  filtered AS (SELECT * FROM raw WHERE amount > 100),
  ranked AS (SELECT *, RANK() OVER (ORDER BY amount DESC) AS rnk FROM filtered)
SELECT * FROM ranked WHERE rnk <= 10;
```

**Expected output** (illustrative):

```text
 order_id | customer_id | amount | rnk
----------+-------------+--------+-----
     9001 |         104 | 999.99 |   1
     9002 |         112 | 899.99 |   2
(2 rows)
```

**Why this output appears**: `filtered` references `raw`, and `ranked` references `filtered`; the declaration order matches the dependency chain.

##### Example 2: MySQL – Reordering to fix an error

```sql
-- Invalid: high_value references totals before totals is declared.
-- WITH high_value AS (SELECT * FROM totals WHERE total > 500),
--      totals AS (SELECT customer_id, SUM(amount) AS total FROM orders GROUP BY customer_id)
-- SELECT * FROM high_value;

-- Corrected order:
WITH totals AS (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
),
high_value AS (
    SELECT * FROM totals WHERE total > 500
)
SELECT * FROM high_value;
```

**Expected output**:

```text
+-------------+--------+
| customer_id | total  |
+-------------+--------+
|         104 | 1234.5 |
|         112 |  789.0 |
+-------------+--------+
2 rows in set (0.00 sec)
```

**Why this output appears**: `totals` is declared before `high_value`, so the reference resolves. The commented invalid version would raise `Table 'totals' doesn't exist`.

##### Example 3: SQL Server – Independent CTEs in any order

```sql
;WITH CustomersCTE AS (
    SELECT customer_id, customer_name FROM Customers
),
OrdersCTE AS (
    SELECT customer_id, amount FROM Orders
)
SELECT c.customer_name, SUM(o.amount) AS total
FROM CustomersCTE c
LEFT JOIN OrdersCTE o ON o.customer_id = c.customer_id
GROUP BY c.customer_name;
```

**Expected output**:

```text
customer_name   total
--------------- ---------
Acme Corp       58200.00
Globex          41300.50
Initech          NULL
```

**Why this output appears**: `CustomersCTE` and `OrdersCTE` are independent, so either could be declared first; the main query references both.

#### Real-World Cases

- **Code review**: Declaration order makes the logical flow explicit.
- **Debugging**: Reordering independent CTEs can clarify which data source is primary.
- **Migration**: When porting queries across engines, declaration order must be preserved to avoid errors.

---

### 5. CTE Organization

#### Definitions

**Core Definition**  
CTE organization is the practice of arranging CTEs in a `WITH` clause so that the query is readable, maintainable, and logically structured.

**Technical Definition**  
Organization encompasses naming conventions, grouping of related CTEs, comment placement, consistent indentation, and the separation of independent from dependent CTEs. A well‑organised `WITH` clause follows a logical flow from source data to final output and minimises cross‑references that obscure the data flow.

**Beginner-Friendly Explanation**  
CTE organisation is how you arrange your helper queries so the query reads like a clear recipe, with each step in a sensible place and a descriptive name.

#### Purposes

- To make the query understandable at a glance.
- To group related transformations and separate independent data sources.
- To facilitate testing and debugging of individual steps.
- To document intent through names and comments.

#### Syntax Rules and Structure

**Organisation pattern**:

```sql
WITH
  -- ===== Source data =====
  source_a AS (...),
  source_b AS (...),

  -- ===== Cleaning =====
  cleaned_a AS (... FROM source_a ...),
  cleaned_b AS (... FROM source_b ...),

  -- ===== Integration =====
  joined AS (... FROM cleaned_a JOIN cleaned_b ...),

  -- ===== Final shaping =====
  final AS (... FROM joined ...)
SELECT * FROM final;
```

**Component breakdown**

| Section | Contents | Purpose |
|---------|----------|---------|
| Source data | Base CTEs with no CTE dependencies | Establish inputs. |
| Cleaning | Transformations of each source | Normalise data. |
| Integration | Joins and unions | Combine sources. |
| Final shaping | Filters, ranking, formatting | Produce output. |

**Syntax rules**

- Organise CTEs in dependency order.
- Group related CTEs with comments or blank lines.
- Use consistent naming (e.g., `src_`, `cln_`, `int_`, `fin_` prefixes).
- Keep each CTE focused on one logical transformation.
- Avoid cross‑references that skip sections unless necessary.

**Constraints and limitations**

- Over‑organisation can fragment simple queries; balance structure with brevity.
- Naming conventions are team‑specific; document them.
- Some engines may be sensitive to comments between CTE definitions (rare, but possible in older parsers).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Organised pipeline with sections

```sql
WITH
  -- ===== Source data =====
  src_orders AS (
      SELECT order_id, customer_id, amount, status, order_date
      FROM orders
  ),
  src_customers AS (
      SELECT customer_id, name, region
      FROM customers
  ),

  -- ===== Cleaning =====
  cln_orders AS (
      SELECT order_id, customer_id, amount
      FROM src_orders
      WHERE status = 'completed' AND amount > 0
  ),

  -- ===== Integration =====
  int_joined AS (
      SELECT c.customer_id, c.name, c.region, o.amount
      FROM src_customers c
      JOIN cln_orders o ON o.customer_id = c.customer_id
  ),

  -- ===== Final shaping =====
  fin_totals AS (
      SELECT region, name, SUM(amount) AS total
      FROM int_joined
      GROUP BY region, name
  )
SELECT region, name, total
FROM fin_totals
ORDER BY region, total DESC;
```

**Expected output** (illustrative):

```text
 region |   name    |  total
--------+-----------+---------
 North  | Acme Corp | 12345.00
 North  | Globex    |  4321.50
 South  | Initech   |  2345.00
(3 rows)
```

**Why this output appears**: The CTEs are grouped by role; each section depends only on earlier sections, and the final CTE produces the output.

##### Example 2: SQL Server – Grouping independent CTEs first

```sql
;WITH
  -- Independent source CTEs first.
  OrdersCTE AS (
      SELECT customer_id, amount FROM Orders
  ),
  CustomersCTE AS (
      SELECT customer_id, customer_name FROM Customers
  ),
  -- Dependent CTE last.
  Summary AS (
      SELECT c.customer_name, SUM(o.amount) AS total
      FROM CustomersCTE c
      LEFT JOIN OrdersCTE o ON o.customer_id = c.customer_id
      GROUP BY c.customer_name
  )
SELECT * FROM Summary ORDER BY total DESC;
```

**Expected output**:

```text
customer_name   total
--------------- ---------
Acme Corp       58200.00
Globex          41300.50
Initech          NULL
```

**Why this output appears**: Independent CTEs are grouped first, and the dependent CTE follows, making the dependency structure obvious.

#### Real-World Cases

- **Team standards**: Organisations adopt sectioned CTE layouts for BI queries.
- **Documentation**: Sectioned CTEs double as inline documentation.
- **Reviews**: Organised queries are faster to review and less error‑prone.

---

### 6. CTE Chaining

#### Definitions

**Core Definition**  
CTE chaining is the practice of linking CTEs in sequence, where each CTE transforms the output of the previous one, forming a pipeline.

**Technical Definition**  
Chaining creates a linear dependency graph: `cte1 → cte2 → cte3 → … → final`. Each downstream CTE references one upstream CTE (or a small set), and the main statement references the last CTE. Chaining is a special case of CTE referencing where the graph is a path rather than a general DAG.

**Beginner-Friendly Explanation**  
CTE chaining is like an assembly line: each station (CTE) takes the output of the previous station and performs one transformation, ending with the final product.

#### Purposes

- To express a multi‑step transformation as a clear sequence.
- To make each step independently testable.
- To avoid deeply nested subqueries.
- To support incremental development of complex queries.

#### Syntax Rules and Structure

**Chaining pattern**:

```sql
WITH
  step1 AS (SELECT ... FROM source),
  step2 AS (SELECT ... FROM step1),
  step3 AS (SELECT ... FROM step2),
  step4 AS (SELECT ... FROM step3)
SELECT ... FROM step4;
```

**Component breakdown**

| CTE | Input | Transformation |
|-----|-------|----------------|
| `step1` | source table | Filter/clean |
| `step2` | `step1` | Aggregate |
| `step3` | `step2` | Join/enrich |
| `step4` | `step3` | Rank/format |

**Syntax rules**

- Each CTE references the immediately preceding CTE (or a small set).
- The chain must respect declaration order.
- The main statement references the last CTE.
- Comments between steps help document each transformation.

**Constraints and limitations**

- Long chains can obscure the overall logic if not commented.
- Some optimisers may materialise each step, increasing I/O; hints can help.
- Chaining is not a substitute for good naming; each step should be meaningful.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Four‑step chain

```sql
-- Step 1: Select raw orders.
WITH raw_orders AS (
    SELECT order_id, customer_id, amount, status
    FROM orders
),
-- Step 2: Filter completed orders.
completed AS (
    SELECT order_id, customer_id, amount
    FROM raw_orders
    WHERE status = 'completed'
),
-- Step 3: Aggregate per customer.
totals AS (
    SELECT customer_id, SUM(amount) AS total
    FROM completed
    GROUP BY customer_id
),
-- Step 4: Rank customers.
ranked AS (
    SELECT customer_id, total,
           RANK() OVER (ORDER BY total DESC) AS rnk
    FROM totals
)
SELECT customer_id, total, rnk
FROM ranked
WHERE rnk <= 3;
```

**Expected output**:

```text
 customer_id |  total  | rnk
-------------+---------+-----
         104 | 12345.0 |   1
         112 |  4321.5 |   2
          98 |  2345.0 |   3
(3 rows)
```

**Why this output appears**: Each CTE transforms the previous one; the final CTE ranks customers, and the main query filters the top three.

##### Example 2: MySQL – Chaining with aggregation and filtering

```sql
WITH
  base AS (
      SELECT customer_id, amount FROM orders WHERE amount > 0
  ),
  totals AS (
      SELECT customer_id, SUM(amount) AS total FROM base GROUP BY customer_id
  ),
  high_value AS (
      SELECT customer_id, total FROM totals WHERE total > 500
  )
SELECT customer_id, total FROM high_value ORDER BY total DESC;
```

**Expected output**:

```text
+-------------+--------+
| customer_id | total  |
+-------------+--------+
|         104 | 1234.5 |
|         112 |  789.0 |
+-------------+--------+
2 rows in set (0.00 sec)
```

**Why this output appears**: `base` filters, `totals` aggregates, and `high_value` filters again; the chain is linear and each step depends on the previous one.

##### Example 3: SQL Server – Chaining with joins and ranking

```sql
;WITH
  base AS (
      SELECT order_id, customer_id, amount FROM Orders
  ),
  totals AS (
      SELECT customer_id, SUM(amount) AS total FROM base GROUP BY customer_id
  ),
  enriched AS (
      SELECT c.customer_name, t.total
      FROM Customers c
      JOIN totals t ON t.customer_id = c.customer_id
  ),
  ranked AS (
      SELECT customer_name, total,
             ROW_NUMBER() OVER (ORDER BY total DESC) AS rn
      FROM enriched
  )
SELECT customer_name, total
FROM ranked
WHERE rn <= 5;
```

**Expected output**:

```text
customer_name   total
--------------- ---------
Acme Corp       58200.00
Globex          41300.50
Initech          9800.00
Umbrella         7500.00
Soylent          6200.00
```

**Why this output appears**: The chain moves from raw orders to totals, enriches with customer names, ranks, and filters the top five.

#### Real-World Cases

- **ETL pipelines**: Each CTE is a stage in the transformation.
- **Analytics**: Chaining expresses filter → aggregate → rank → format.
- **Debugging**: Running each CTE independently verifies intermediate results.

---

## References

- PostgreSQL Documentation – `WITH` Queries (Common Table Expressions) – https://www.postgresql.org/docs/current/queries-with.html
- PostgreSQL Documentation – `SELECT` (`WITH` clause) – https://www.postgresql.org/docs/current/sql-select.html
- MySQL 8.4 Reference Manual – `WITH` (Common Table Expressions) – https://dev.mysql.com/doc/refman/8.4/en/with.html
- Microsoft Learn – `WITH common_table_expression` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql
- SQL:1999 Standard – Common Table Expressions (ISO/IEC 9075-2:1999)
- Modern SQL – Common Table Expressions – https://modern-sql.com/feature/with
- Use The Index, Luke – Common Table Expressions – https://use-the-index-luke.com/sql/with
- Baeldung – CTE vs Subquery vs Derived Table – https://www.baeldung.com/sql/cte-vs-subquery
- Microsoft Learn – Recursive Queries Using Common Table Expressions – https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql
- Oracle Database Documentation – `WITH` Clause (Subquery Factoring) – https://docs.oracle.com/en/database/oracle/oracle-database/23/sqlrf/SELECT.html