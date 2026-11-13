# Comprehensive Programming Cheat Sheet: CTE Fundamentals

## Topic Overview

### Definitions

**Core Definition**  
A Common Table Expression (CTE) is a named temporary result set defined within a single SQL statement using the `WITH` clause, which can be referenced one or more times within that statement.

**Technical Definition**  
A Common Table Expression is a language construct introduced in SQL:1999 that defines a named query expression whose scope is limited to the statement in which it appears. The `WITH` clause precedes the main query (and in some dialects, precedes `INSERT`, `UPDATE`, `DELETE`, or `MERGE`), and each CTE comprises a query name, an optional column list, and a query expression. CTEs may be non‑recursive or recursive (the latter using `WITH RECURSIVE`). Their result set is not materialised by definition; the optimiser may inline or materialise the expression depending on the engine.

**Beginner-Friendly Explanation**  
A CTE is like giving a name to a temporary query result so you can refer to it later in the same SQL statement, much like defining a helper variable inside a single sentence. It makes complex queries easier to read and reuse without creating a permanent view or table.

### Key Characteristics

- **Statement‑scoped**: A CTE exists only for the duration of the single statement that defines it.
- **Named**: Each CTE has an identifier that can be referenced like a table within the statement.
- **Reusable within the statement**: A CTE can be referenced multiple times in the main query.
- **Supports recursion**: `WITH RECURSIVE` enables hierarchical and iterative queries.
- **Non‑persistent**: No object is stored in the database catalog; it is not a view.
- **Precedes the main statement**: The `WITH` clause appears before `SELECT`, `INSERT`, `UPDATE`, `DELETE`, or `MERGE`.
- **Dialect‑specific details**: Support for data‑modifying CTEs, `MATERIALIZED` hints, and recursive syntax varies by RDBMS.

### Prerequisites

- A working knowledge of `SELECT` statements, joins, and `WHERE` clauses.
- Familiarity with aliases and column naming.
- Access to an RDBMS that supports the `WITH` clause (PostgreSQL, MySQL 8.0+, SQL Server, Oracle, SQLite 3.8.3+).
- For recursion: understanding of base cases and recursive steps.

### Related Programming Areas

- Query optimisation and execution plans.
- Data warehousing and ETL transformations.
- Recursive queries for hierarchical data (org charts, bill‑of‑materials).
- Data modification statements (`INSERT … WITH`, `UPDATE … FROM`, `DELETE … USING`).
- View and subquery design.

---

## Core Concepts and Features

### 1. `WITH` Clause

#### Definitions

**Core Definition**  
The `WITH` clause is the SQL keyword that introduces one or more Common Table Expressions before the main query.

**Technical Definition**  
The `WITH` clause is a prefix to a `SELECT`, `INSERT`, `UPDATE`, `DELETE`, or `MERGE` statement that declares one or more named query expressions. Each expression is separated by a comma. The clause is defined in SQL:1999 and is supported with vendor variations. In PostgreSQL and SQL Server, `WITH` can also precede data‑modification statements; in MySQL 8.0+, `WITH` is supported at the beginning of `SELECT`, `UPDATE`, and `DELETE`.

**Beginner-Friendly Explanation**  
`WITH` is the word you write at the start of your query to say “first, let me define some temporary named results, and then I’ll use them.”

#### Purposes

- To introduce named temporary result sets that can be referenced later in the same statement.
- To break a complex query into readable, modular pieces.
- To avoid repeating the same subquery multiple times in a statement.
- To enable recursive queries through `WITH RECURSIVE`.
- To support data‑modifying statements that read from and write to the same table.

#### Syntax Rules and Structure

**General syntax**:

```sql
WITH cte_name [(column1, column2, ...)] AS (
    query_expression
)
[, cte_name2 [(...)] AS (
    query_expression2
)]
main_statement;
```

**PostgreSQL** (official):

```sql
WITH [ RECURSIVE ] with_query [, ...]
main_query;

with_query:
    query_name [ ( column_name [, ...] ) ] AS [ [ NOT ] MATERIALIZED ] ( query )
    | query_name [ ( column_name [, ...] ) ] AS [ [ NOT ] MATERIALIZED ] ( query )
      SEARCH { BREADTH | DEPTH } FIRST BY column_name SET search_seq_col_name
      CYCLE column_name SET cycle_mark_col_name TO cycle_value DEFAULT cycle_default_value USING cycle_path_col_name
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
| `WITH` | Keyword that introduces the CTE list. |
| `RECURSIVE` | Optional keyword enabling self‑referencing CTEs. |
| `cte_name` / `query_name` | Identifier for the CTE; must be unique within the `WITH` list. |
| `(column1, column2, ...)` | Optional column alias list; overrides the column names produced by the query. |
| `AS` | Keyword separating the name from the query definition. |
| `(query_expression)` | The `SELECT` (or data‑modifying statement in supported dialects) that defines the CTE. |
| `main_statement` | The `SELECT`, `INSERT`, `UPDATE`, `DELETE`, or `MERGE` that references the CTE(s). |

**Syntax rules**

- The `WITH` clause must appear at the very beginning of the statement in PostgreSQL and MySQL; in SQL Server it may follow a preceding statement only if that statement is terminated with a semicolon.
- Multiple CTEs are separated by commas.
- Each CTE name must be unique within the same `WITH` clause.
- A CTE can reference CTEs defined earlier in the same `WITH` clause, but not later ones (except in recursive CTEs, where it can reference itself).
- The trailing semicolon terminates the entire statement, not the `WITH` clause.

**Constraints and limitations**

- A CTE cannot be referenced outside the statement that defines it.
- In SQL Server, a CTE definition cannot contain `ORDER BY` unless `TOP` or `OFFSET/FETCH` is also specified.
- In MySQL 8.0, a CTE cannot be used in a subquery of another CTE definition in some versions; recursive CTEs have additional restrictions.
- Data‑modifying CTEs (`WITH … INSERT/UPDATE/DELETE`) are supported in PostgreSQL and SQL Server, but not in MySQL as of MySQL 8.4.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – A simple `WITH` clause

```sql
-- Step 1: Define a CTE named "high_value_orders" that selects orders above 1000.
WITH high_value_orders AS (
    SELECT order_id, customer_id, total_amount
    FROM orders
    WHERE total_amount > 1000
)
-- Step 2: Use the CTE in the main query.
SELECT customer_id, COUNT(*) AS order_count
FROM high_value_orders
GROUP BY customer_id
ORDER BY order_count DESC;
```

**Expected output** (illustrative):

```text
 customer_id | order_count
-------------+-------------
         104 |           7
         112 |           5
          98 |           4
(3 rows)
```

**Why this output appears**: The CTE `high_value_orders` computes the filtered set once conceptually, and the main query aggregates it by `customer_id`, producing one row per customer with the count of qualifying orders.

##### Example 2: SQL Server – Multiple CTEs in one `WITH` clause

```sql
-- Step 1: Define two CTEs: one for recent orders and one for active customers.
WITH RecentOrders AS (
    SELECT order_id, customer_id, order_date
    FROM Orders
    WHERE order_date >= DATEADD(day, -30, GETDATE())
),
ActiveCustomers AS (
    SELECT customer_id, customer_name
    FROM Customers
    WHERE is_active = 1
)
-- Step 2: Join the two CTEs in the main query.
SELECT c.customer_name, COUNT(r.order_id) AS recent_orders
FROM ActiveCustomers c
LEFT JOIN RecentOrders r ON r.customer_id = c.customer_id
GROUP BY c.customer_name
ORDER BY recent_orders DESC;
```

**Expected output** (illustrative):

```text
customer_name   recent_orders
--------------- -------------
Acme Corp       12
Globex          8
Initech         0
```

**Why this output appears**: Both CTEs are evaluated as named result sets; `ActiveCustomers` provides the customer list, and `RecentOrders` supplies the filtered order data. The `LEFT JOIN` preserves customers with zero recent orders.

##### Example 3: MySQL 8.0 – `WITH` clause preceding a `SELECT`

```sql
-- Step 1: Define a CTE that computes average order value per customer.
WITH avg_order AS (
    SELECT customer_id, AVG(total_amount) AS avg_amount
    FROM orders
    GROUP BY customer_id
)
-- Step 2: Select customers whose average order exceeds 500.
SELECT customer_id, avg_amount
FROM avg_order
WHERE avg_amount > 500;
```

**Expected output**:

```text
+-------------+------------+
| customer_id | avg_amount |
+-------------+------------+
|         104 |     742.50 |
|         112 |     611.20 |
+-------------+------------+
2 rows in set (0.00 sec)
```

**Why this output appears**: The CTE aggregates `orders` by customer; the main query filters the aggregated result.

#### Real-World Cases

- **Analytics pipelines**: CTEs modularise multi‑step aggregations so each step is readable and testable.
- **Data quality checks**: A CTE computes a set of suspect records, and the main query reports on them.
- **Application queries**: ORM‑generated SQL often uses CTEs to structure complex retrievals.

---

### 2. CTE Syntax

#### Definitions

**Core Definition**  
CTE syntax is the structure of the `WITH` clause: the keyword `WITH`, optional `RECURSIVE`, one or more CTE definitions, and the main statement.

**Technical Definition**  
The syntactic form comprises `WITH [RECURSIVE] name [(columns)] AS (query) [, ...] statement`. Optional clauses include `MATERIALIZED`/`NOT MATERIALIZED` in PostgreSQL, `SEARCH`/`CYCLE` for recursive traversal in PostgreSQL, and column alias lists. Vendor differences exist in allowed statement types and clause placement.

**Beginner-Friendly Explanation**  
CTE syntax is the grammar you follow to write a CTE correctly: the `WITH` keyword, the name, the `AS`, and the query in parentheses.

#### Purposes

- To provide a consistent, standardised way to declare named query expressions.
- To allow explicit column naming for the CTE output.
- To support recursion and traversal semantics where the RDBMS provides them.
- To control materialisation behaviour in engines that expose such hints.

#### Syntax Rules and Structure

**Complete general syntax**:

```sql
WITH [RECURSIVE]
     cte_name [(column1, column2, ...)] AS [MATERIALIZED | NOT MATERIALIZED] (
         query_expression
     )
     [, cte_name2 [(...)] AS (...) ...]
[ SEARCH { BREADTH | DEPTH } FIRST BY column_name SET search_seq_col_name ]
[ CYCLE column_name SET cycle_mark_col_name TO cycle_value DEFAULT cycle_default_value USING cycle_path_col_name ]
main_statement;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `RECURSIVE` | Enables self‑reference within CTE definitions. |
| `cte_name` | Identifier for the CTE. |
| `column_list` | Optional list that names the CTE’s output columns. |
| `AS` | Separator between name and query. |
| `MATERIALIZED` / `NOT MATERIALIZED` | PostgreSQL hints controlling whether the CTE result is materialised or inlined. |
| `query_expression` | The defining query. |
| `SEARCH` / `CYCLE` | PostgreSQL 14+ clauses for recursive traversal ordering and cycle detection. |
| `main_statement` | The statement consuming the CTE. |

**Syntax rules**

- The column list, if provided, must match the number of columns produced by the query.
- Column aliases in the CTE definition take precedence over the query’s own column names.
- `RECURSIVE` applies to the entire `WITH` list; it enables recursion in any CTE within that list.
- `MATERIALIZED` and `NOT MATERIALIZED` are PostgreSQL‑specific.
- In SQL Server, `WITH` must be preceded by a semicolon if it is not the first statement in a batch.
- In MySQL 8.0, `WITH` may be used at the start of `SELECT`, `UPDATE`, and `DELETE`; it cannot precede `INSERT … SELECT` in the same way as PostgreSQL.

**Constraints and limitations**

- A CTE query expression cannot use `ORDER BY` in some engines unless a limiting clause is present.
- Recursive CTEs have stricter rules: the recursive term must reference the CTE exactly once, and the union between base and recursive terms must be compatible.
- Some engines do not allow data‑modifying statements inside CTEs.
- `MATERIALIZED` is a hint; the optimiser may not honour it in all cases.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – CTE with explicit column aliases and `MATERIALIZED`

```sql
-- Step 1: Define a CTE with explicit column names and request materialisation.
WITH monthly_sales (region, month, total) AS MATERIALIZED (
    SELECT region, date_trunc('month', sale_date), SUM(amount)
    FROM sales
    GROUP BY region, date_trunc('month', sale_date)
)
-- Step 2: Query the materialised CTE.
SELECT region, month, total
FROM monthly_sales
WHERE total > 10000
ORDER BY total DESC;
```

**Expected output** (illustrative):

```text
 region |        month        |  total
--------+---------------------+----------
 North  | 2025-01-01 00:00:00 | 15230.00
 South  | 2025-02-01 00:00:00 | 11890.50
(2 rows)
```

**Why this output appears**: The CTE is materialised into a temporary result because of the `MATERIALIZED` hint, and the outer query filters it. Column aliases (`region`, `month`, `total`) rename the CTE’s output.

##### Example 2: PostgreSQL – Recursive CTE with `SEARCH` and `CYCLE`

```sql
-- Step 1: Define a recursive CTE for an employee hierarchy with cycle detection.
WITH RECURSIVE org_chart AS (
    SELECT employee_id, manager_id, name, 1 AS depth
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.manager_id, e.name, oc.depth + 1
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.employee_id
)
SEARCH DEPTH FIRST BY employee_id SET ordercol
CYCLE employee_id SET is_cycle TO 'Y' DEFAULT 'N' USING path
-- Step 2: Query the hierarchy.
SELECT employee_id, name, depth, is_cycle
FROM org_chart
ORDER BY ordercol;
```

**Expected output** (illustrative):

```text
 employee_id |   name    | depth | is_cycle
-------------+-----------+-------+----------
           1 | Alice     |     1 | N
           2 | Bob       |     2 | N
           3 | Carol     |     2 | N
           4 | David     |     3 | N
(4 rows)
```

**Why this output appears**: The recursive term expands the hierarchy level by level, `SEARCH DEPTH FIRST` produces an ordering column, and `CYCLE` prevents infinite loops by tracking visited `employee_id` values.

##### Example 3: SQL Server – CTE preceded by a semicolon

```sql
-- Step 1: The previous statement must be terminated with a semicolon.
SELECT 1 AS dummy;

-- Step 2: Start the WITH clause.
;WITH SalesCTE AS (
    SELECT salesperson_id, SUM(amount) AS total_sales
    FROM Sales
    GROUP BY salesperson_id
)
SELECT salesperson_id, total_sales
FROM SalesCTE
WHERE total_sales > 50000;
```

**Expected output**:

```text
salesperson_id total_sales
-------------- -----------
12             58200.00
27             61300.50
```

**Why this output appears**: In SQL Server, the leading semicolon terminates any prior statement, allowing `WITH` to be parsed correctly. The CTE aggregates sales and the outer query filters the result.

#### Real-World Cases

- **Recursive hierarchies**: Org charts, bill‑of‑materials, and category trees.
- **Modular analytics**: Each CTE represents a logical step that can be tested in isolation.
- **Materialisation control**: Analysts use `MATERIALIZED` to prevent repeated expensive evaluations.

---

### 3. CTE Naming

#### Definitions

**Core Definition**  
CTE naming is the act of assigning an identifier to a Common Table Expression so it can be referenced within the statement.

**Technical Definition**  
A CTE name is an SQL identifier that must be unique within its `WITH` clause. It follows the RDBMS’s identifier rules for unquoted or quoted identifiers. The name shadows any real table with the same name within the statement’s scope, so referencing the name resolves to the CTE rather than the base table. Column names can be renamed via an optional column alias list.

**Beginner-Friendly Explanation**  
CTE naming is simply choosing the label for your temporary result set, like naming a variable, so you can refer to it later.

#### Purposes

- To provide a meaningful, readable label for each temporary result set.
- To allow reuse of the result set by name within the statement.
- To avoid name collisions with base tables by choosing distinctive names.
- To override the output column names via an optional alias list.

#### Syntax Rules and Structure

**General syntax**:

```sql
WITH cte_name [(col1, col2, ...)] AS (query)
SELECT ... FROM cte_name ...;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `cte_name` | Identifier for the CTE; must be unique within the `WITH` list. |
| `col1, col2, ...` | Optional aliases for the CTE’s output columns. |

**Syntax rules**

- The name must be a valid SQL identifier; use double quotes (PostgreSQL, SQL Server with `QUOTED_IDENTIFIER`) or backticks (MySQL) for case‑sensitive or reserved words.
- The name must be unique within the same `WITH` clause.
- If a CTE name matches a base table, the CTE shadows the table for the duration of the statement.
- The optional column list must match the query’s column count.
- Column aliases in the CTE definition override the underlying query’s column names.

**Constraints and limitations**

- You cannot reference a CTE outside its defining statement.
- Using a CTE name that collides with a base table can cause confusion; prefix names (e.g., `cte_`) are a common convention.
- In SQL Server, CTE names are case‑insensitive by default (depending on collation).
- In PostgreSQL, unquoted identifiers are folded to lower case.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Descriptive CTE name with column aliases

```sql
-- Step 1: Name the CTE "customer_totals" and rename its columns.
WITH customer_totals (cust_id, total_spent) AS (
    SELECT customer_id, SUM(amount)
    FROM orders
    GROUP BY customer_id
)
-- Step 2: Reference the CTE by name.
SELECT cust_id, total_spent
FROM customer_totals
WHERE total_spent > 500;
```

**Expected output**:

```text
 cust_id | total_spent
---------+-------------
     104 |     1234.50
     112 |      789.00
(2 rows)
```

**Why this output appears**: The CTE is named `customer_totals`, and its output columns are aliased `cust_id` and `total_spent`. The outer query references the CTE and filters by `total_spent`.

##### Example 2: MySQL – Avoiding name collision with a base table

```sql
-- Step 1: There is a base table named "orders". Name the CTE distinctly.
WITH recent_orders_cte AS (
    SELECT order_id, customer_id
    FROM orders
    WHERE order_date >= CURDATE() - INTERVAL 7 DAY
)
-- Step 2: The CTE name does not collide with the base table.
SELECT o.order_id, o.customer_id
FROM recent_orders_cte o;
```

**Expected output**:

```text
+----------+-------------+
| order_id | customer_id |
+----------+-------------+
|     9001 |         104 |
|     9002 |         112 |
+----------+-------------+
2 rows in set (0.00 sec)
```

**Why this output appears**: The CTE is named `recent_orders_cte` rather than `orders`, preventing ambiguity. The outer query aliases the CTE as `o`.

#### Real-World Cases

- **Naming conventions**: Teams adopt prefixes like `cte_` or `base_` to distinguish CTEs from tables.
- **Readable pipelines**: Each CTE is named for the business concept it represents (e.g., `monthly_revenue`, `active_customers`).
- **Shadowing awareness**: Analysts must know that a CTE named like a table shadows the table within the statement.

---

### 4. CTE Scope

#### Definitions

**Core Definition**  
CTE scope is the region of a SQL statement in which a CTE name is visible and can be referenced.

**Technical Definition**  
A CTE is visible from its definition point to the end of the statement that contains the `WITH` clause. It is not visible in other statements, in other sessions, or in subqueries of other statements. A CTE can reference CTEs defined earlier in the same `WITH` clause. In recursive CTEs, the CTE can reference itself. The scope is statement‑local and ends when the statement completes.

**Beginner-Friendly Explanation**  
A CTE exists only inside the single SQL statement where you define it. Outside that statement, it is as if it never existed.

#### Purposes

- To limit the lifetime of a named result to a single statement.
- To allow earlier CTEs in a `WITH` clause to be reused by later CTEs.
- To prevent name pollution in the database catalog.
- To enable recursion by allowing a CTE to reference itself within its own scope.

#### Syntax Rules and Structure

**General scope pattern**:

```sql
WITH cte1 AS (SELECT ...),
     cte2 AS (SELECT ... FROM cte1 ...)   -- cte2 can reference cte1
SELECT ...
FROM cte2;                               -- main query can reference cte2
-- cte1 and cte2 are NOT visible outside this statement
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `cte1` | Defined first; visible to `cte2` and the main query. |
| `cte2` | Defined second; visible to the main query and later CTEs. |
| `main query` | Can reference all CTEs defined in the `WITH` clause. |

**Syntax rules**

- A CTE can reference CTEs defined earlier in the same `WITH` clause.
- A CTE cannot reference CTEs defined later (except itself, if recursive).
- The scope ends at the end of the statement.
- In recursive CTEs, the self‑reference is permitted only in the recursive term.
- Nested `WITH` clauses have their own scope.

**Constraints and limitations**

- You cannot reference a CTE in a separate statement.
- You cannot create a view from a CTE without embedding the CTE in the view definition.
- In PostgreSQL, a CTE defined in a subquery is visible only within that subquery and its descendants.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Chained CTEs with forward visibility

```sql
-- Step 1: cte1 is defined first.
WITH cte1 AS (
    SELECT id, amount FROM orders WHERE amount > 100
),
-- Step 2: cte2 references cte1, which is already defined.
cte2 AS (
    SELECT id, amount * 1.1 AS amount_with_tax
    FROM cte1
)
-- Step 3: The main query references cte2.
SELECT * FROM cte2;
```

**Expected output**:

```text
 id  | amount_with_tax
-----+-----------------
 101 |          220.00
 102 |          330.00
(2 rows)
```

**Why this output appears**: `cte1` is visible to `cte2`, and `cte2` is visible to the main query. The chain respects the rule that a CTE can reference only earlier CTEs.

##### Example 2: MySQL – CTE scope is limited to one statement

```sql
-- Step 1: First statement with a CTE.
WITH temp AS (SELECT 1 AS n)
SELECT * FROM temp;

-- Step 2: This second statement cannot see the CTE "temp".
SELECT * FROM temp;  -- ERROR: Table 'temp' doesn't exist
```

**Expected output**:

```text
+------+
| n    |
+------+
|    1 |
+------+
1 row in set (0.00 sec)

ERROR 1146 (42S02): Table 'db.temp' doesn't exist
```

**Why this output appears**: The CTE `temp` is scoped to the first statement only; the second statement treats `temp` as a base table and fails.

##### Example 3: SQL Server – Nested `WITH` scope in a subquery

```sql
-- Step 1: Outer CTE.
WITH outer_cte AS (
    SELECT 10 AS x
)
-- Step 2: Use a nested CTE inside a subquery.
SELECT x,
       (WITH inner_cte AS (SELECT x * 2 AS y)
        SELECT y FROM inner_cte) AS doubled
FROM outer_cte;
```

**Expected output**:

```text
x   doubled
--- -------
10  20
```

**Why this output appears**: The inner CTE `inner_cte` is scoped to its subquery, and `outer_cte` is scoped to the outer statement.

#### Real-World Cases

- **Modular query composition**: Analysts chain CTEs step by step, relying on forward visibility.
- **Testing**: Each CTE can be inspected in isolation by running it as a standalone query and replacing the `WITH` clause.
- **Avoiding leakage**: Scope ensures that temporary names do not collide with other queries in the same session.

---

### 5. CTE vs Subqueries

#### Definitions

**Core Definition**  
A subquery is a query nested inside another query; a CTE is a named query expression declared with `WITH`. Both produce intermediate result sets, but CTEs are named and can be referenced multiple times.

**Technical Definition**  
A subquery is an expression that appears in a `FROM`, `WHERE`, `SELECT`, `HAVING`, or `JOIN` clause and is evaluated within the outer query. A CTE is a statement‑level named query defined in a `WITH` clause. Semantically both are derived relations, but CTEs can be referenced multiple times by name, can be recursive, and may be optimised differently (e.g., materialisation hints).

**Beginner-Friendly Explanation**  
A subquery is a query written inside another query; a CTE gives that inner query a name so you can reuse it and keep the outer query tidy.

#### Purposes

- To choose between inline nesting and named modularity based on readability.
- To reuse the same intermediate result in multiple places without repeating the query.
- To enable recursion where subqueries cannot express recursive logic easily.
- To control materialisation via engine hints where available.

#### Syntax Rules and Structure

**Subquery example**:

```sql
SELECT customer_id, total
FROM (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
) sub
WHERE total > 500;
```

**CTE equivalent**:

```sql
WITH totals AS (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT customer_id, total
FROM totals
WHERE total > 500;
```

**Component breakdown**

| Aspect | Subquery | CTE |
|--------|----------|-----|
| Placement | Inside `FROM`, `WHERE`, etc. | Before main statement |
| Naming | Optional alias in `FROM` | Required name in `WITH` |
| Reuse | Must repeat or self‑join | Reference by name multiple times |
| Recursion | Not directly recursive | Supported via `WITH RECURSIVE` |
| Readability | Nested, can become deep | Linear, named steps |
| Optimisation | May be inlined | May be inlined or materialised |

**Syntax rules**

- Subqueries can be correlated or uncorrelated; CTEs are uncorrelated by definition (they cannot reference outer query columns unless they are in a nested `WITH`).
- CTEs appear before the main statement; subqueries appear inside it.
- Both must produce column counts compatible with the surrounding query.

**Constraints and limitations**

- Subqueries can often reference outer query columns (correlated subqueries); CTEs cannot.
- Deeply nested subqueries reduce readability; CTEs flatten the structure.
- CTEs may be materialised in some engines, which can improve or degrade performance depending on the query.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Subquery repeated twice vs CTE referenced twice

```sql
-- Subquery version: the same computation is written twice.
SELECT c.customer_id,
       (SELECT SUM(amount) FROM orders o WHERE o.customer_id = c.customer_id) AS total1,
       (SELECT AVG(amount) FROM orders o WHERE o.customer_id = c.customer_id) AS avg1
FROM customers c;

-- CTE version: the aggregation is computed once and referenced twice.
WITH order_stats AS (
    SELECT customer_id, SUM(amount) AS total, AVG(amount) AS avg
    FROM orders
    GROUP BY customer_id
)
SELECT c.customer_id, s.total, s.avg
FROM customers c
LEFT JOIN order_stats s ON s.customer_id = c.customer_id;
```

**Expected output** (same for both, illustrative):

```text
 customer_id | total  |  avg
-------------+--------+-------
         104 | 1234.5 | 205.75
         112 |  789.0 | 157.80
(2 rows)
```

**Why this output appears**: The CTE version computes the aggregation once and joins it; the subquery version computes it twice. The results are identical, but the CTE is more readable and potentially more efficient.

##### Example 2: MySQL – Subquery in `FROM` vs CTE

```sql
-- Subquery in FROM.
SELECT customer_id, total
FROM (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
) AS sub
WHERE total > 500;

-- Equivalent CTE.
WITH sub AS (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT customer_id, total
FROM sub
WHERE total > 500;
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

**Why this output appears**: Both formulations produce the same derived relation; the CTE is named, while the subquery is aliased `sub`.

#### Real-World Cases

- **Readability refactoring**: Teams replace deeply nested subqueries with CTEs to improve maintainability.
- **Reuse**: A CTE is preferred when the same intermediate result is needed multiple times.
- **Recursive needs**: Recursive queries require CTEs (or equivalent vendor constructs), not plain subqueries.

---

### 6. CTE vs Derived Tables

#### Definitions

**Core Definition**  
A derived table is a subquery in the `FROM` clause that produces a temporary table‑like relation; a CTE is a named query defined with `WITH`.

**Technical Definition**  
A derived table is a parenthesised subquery with an alias in the `FROM` clause. Its scope is the surrounding query block. A CTE is a statement‑level named query defined before the main statement. Derived tables cannot be referenced outside their query block, whereas CTEs can be referenced throughout the statement (and earlier CTEs can reference later ones only in a forward manner).

**Beginner-Friendly Explanation**  
A derived table is an inner query placed directly in the `FROM` clause; a CTE is the same idea but given a name at the top of the query.

#### Purposes

- To choose between local inline relations (derived tables) and statement‑level named relations (CTEs).
- To avoid naming intermediate results when they are used only once.
- To reuse intermediate results when they are needed more than once.
- To improve readability in long queries by naming each step.

#### Syntax Rules and Structure

**Derived table**:

```sql
SELECT ...
FROM (SELECT ... ) AS alias
JOIN ...;
```

**CTE**:

```sql
WITH alias AS (SELECT ...)
SELECT ... FROM alias JOIN ...;
```

**Component breakdown**

| Aspect | Derived Table | CTE |
|--------|---------------|-----|
| Placement | `FROM` clause | Before main statement |
| Alias | Required in `FROM` | Defined in `WITH` |
| Visibility | Only in the query block | Entire statement |
| Multiple references | Requires repeating | Reference by name |
| Recursion | Not supported | Supported |

**Syntax rules**

- A derived table must have an alias in the `FROM` clause.
- A derived table cannot be referenced outside its query block.
- A CTE can be referenced in multiple places in the main statement.
- Both can contain `SELECT`, and in some dialects, `VALUES`.

**Constraints and limitations**

- Derived tables cannot be recursive.
- Derived tables in MySQL are materialised in some cases; CTEs may be optimised differently.
- SQL Server allows derived tables in `FROM` only; CTEs can also precede data‑modification statements.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Derived table vs CTE

```sql
-- Derived table.
SELECT t.customer_id, t.total
FROM (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
) AS t
WHERE t.total > 500;

-- CTE equivalent.
WITH t AS (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT customer_id, total
FROM t
WHERE total > 500;
```

**Expected output**:

```text
 customer_id | total
-------------+--------
         104 | 1234.5
         112 |  789.0
(2 rows)
```

**Why this output appears**: Both forms produce the same derived relation; the CTE is simply named at the top.

##### Example 2: SQL Server – Reusing a CTE where a derived table would need duplication

```sql
-- Derived table requires duplicating the subquery.
SELECT a.customer_id, a.total, b.avg
FROM (SELECT customer_id, SUM(amount) AS total FROM Orders GROUP BY customer_id) a
JOIN (SELECT customer_id, AVG(amount) AS avg  FROM Orders GROUP BY customer_id) b
  ON a.customer_id = b.customer_id;

-- CTE avoids duplication.
WITH stats AS (
    SELECT customer_id, SUM(amount) AS total, AVG(amount) AS avg
    FROM Orders
    GROUP BY customer_id
)
SELECT customer_id, total, avg
FROM stats;
```

**Expected output**:

```text
customer_id total   avg
----------- ------- ------
104         1234.50 205.75
112          789.00 157.80
```

**Why this output appears**: The derived‑table version repeats the aggregation twice; the CTE version computes both aggregates once and references them together.

#### Real-World Cases

- **Readability refactor**: Derived tables are replaced by CTEs when the query grows long.
- **Single‑use relations**: Derived tables are appropriate when the intermediate result is used exactly once.
- **Reuse and recursion**: CTEs are preferred when reuse or recursion is required.

---

### 7. CTE Readability

#### Definitions

**Core Definition**  
CTE readability is the degree to which a query using CTEs is easy to read, understand, and maintain.

**Technical Definition**  
Readability is influenced by naming conventions, linear structure, separation of concerns, consistent indentation, and the avoidance of deeply nested subqueries. CTEs allow a query to be expressed as a sequence of named steps, each representing a logical transformation. The main query then composes these steps.

**Beginner-Friendly Explanation**  
CTEs make SQL easier to read because each step has a name and appears in order, like a recipe with named steps instead of one long sentence.

#### Purposes

- To make complex queries understandable to reviewers and future maintainers.
- To reduce duplication and improve consistency.
- To facilitate testing by isolating each CTE as a standalone query.
- To document the intent of each step via descriptive names.

#### Syntax Rules and Structure

**Readability‑oriented pattern**:

```sql
WITH
  base_data AS (        -- Step 1: filter and clean
    SELECT ...
  ),
  aggregated AS (       -- Step 2: aggregate
    SELECT ...
  ),
  enriched AS (         -- Step 3: join and enrich
    SELECT ...
  )
SELECT ...              -- Final step: present results
FROM enriched;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| Descriptive CTE names | Convey the purpose of each step. |
| Linear order | Steps read top to bottom. |
| Consistent formatting | Indentation and comma placement. |
| Minimal nesting | Prefer chaining CTEs over nested subqueries. |

**Syntax rules**

- Use meaningful names (e.g., `monthly_revenue`, `active_customers`).
- Keep each CTE focused on one transformation.
- Avoid referencing later CTEs (except recursively).
- Use comments sparingly but meaningfully.

**Constraints and limitations**

- Excessive CTE chaining can hurt performance if each is materialised.
- Very long `WITH` clauses can become unwieldy; balance granularity.
- Readability conventions vary by team; document them.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Readable multi‑step CTE

```sql
-- Step 1: Filter valid orders.
WITH valid_orders AS (
    SELECT order_id, customer_id, amount
    FROM orders
    WHERE status = 'completed'
),
-- Step 2: Aggregate per customer.
customer_totals AS (
    SELECT customer_id, SUM(amount) AS total
    FROM valid_orders
    GROUP BY customer_id
),
-- Step 3: Join customer names.
enriched AS (
    SELECT c.customer_id, c.name, t.total
    FROM customers c
    JOIN customer_totals t ON t.customer_id = c.customer_id
)
-- Step 4: Present top customers.
SELECT name, total
FROM enriched
ORDER BY total DESC
LIMIT 10;
```

**Expected output** (illustrative):

```text
   name    |  total
-----------+---------
 Acme Corp | 12345.00
 Globex    |  9876.50
 Initech   |  7654.00
(3 rows)
```

**Why this output appears**: Each CTE performs one logical transformation, and the main query sorts and limits the result. The linear structure makes the query easy to follow.

##### Example 2: MySQL – Readable query with comments

```sql
WITH
  -- Step 1: Select only active products.
  active_products AS (
      SELECT product_id, name, price
      FROM products
      WHERE is_active = 1
  ),
  -- Step 2: Compute total sold per product.
  sales_totals AS (
      SELECT product_id, SUM(quantity) AS qty_sold
      FROM order_items
      GROUP BY product_id
  )
-- Step 3: Combine and rank.
SELECT p.name, p.price, COALESCE(s.qty_sold, 0) AS qty_sold
FROM active_products p
LEFT JOIN sales_totals s ON s.product_id = p.product_id
ORDER BY qty_sold DESC;
```

**Expected output**:

```text
+--------------+-------+----------+
| name         | price | qty_sold |
+--------------+-------+----------+
| Widget A     | 19.99 |      150 |
| Widget B     | 29.99 |       80 |
| Widget C     |  9.99 |        0 |
+--------------+-------+----------+
3 rows in set (0.00 sec)
```

**Why this output appears**: The CTEs separate filtering from aggregation; the `LEFT JOIN` preserves active products with no sales, and `COALESCE` replaces `NULL` with `0`.

#### Real-World Cases

- **Code reviews**: Readable CTEs reduce review time and defects.
- **Documentation**: CTE names serve as inline documentation of the query’s logic.
- **Onboarding**: New team members can follow the linear structure of CTE‑based queries more easily than nested subqueries.

---

## References

- PostgreSQL Documentation – `WITH` Queries (Common Table Expressions) – https://www.postgresql.org/docs/current/queries-with.html
- PostgreSQL Documentation – `SELECT` (WITH clause) – https://www.postgresql.org/docs/current/sql-select.html
- MySQL 8.4 Reference Manual – `WITH` (Common Table Expressions) – https://dev.mysql.com/doc/refman/8.4/en/with.html
- Microsoft Learn – `WITH common_table_expression` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql
- SQL:1999 Standard – Common Table Expressions (ISO/IEC 9075-2:1999)
- Modern SQL – Common Table Expressions – https://modern-sql.com/feature/with
- Use The Index, Luke – Common Table Expressions – https://use-the-index-luke.com/sql/with
- Baeldung – CTE vs Subquery vs Derived Table – https://www.baeldung.com/sql/cte-vs-subquery
- Microsoft Learn – Recursive Queries Using Common Table Expressions – https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql
- Oracle Database Documentation – `WITH` Clause (Subquery Factoring) – https://docs.oracle.com/en/database/oracle/oracle-database/23/sqlrf/SELECT.html