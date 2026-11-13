# Comprehensive Programming Cheat Sheet: Recursive CTEs

## Topic Overview

### Definitions

**Core Definition**  
A Recursive Common Table Expression (Recursive CTE) is a CTE that references itself in its own query definition, enabling iterative computation such as hierarchy traversal, graph reachability, and sequence generation.

**Technical Definition**  
A recursive CTE is declared with `WITH RECURSIVE` (PostgreSQL, MySQL, SQLite) or with the recursive keyword implied by self‑reference (SQL Server, Oracle). It consists of an anchor member (base case) combined via `UNION [ALL]` with a recursive member that references the CTE name. Evaluation proceeds iteratively: the anchor produces the initial working table, then the recursive member is applied repeatedly to the working table until it produces no new rows (or a termination condition is reached). The SQL standard defines recursive CTEs in SQL:1999; vendor implementations differ in recursion depth limits, cycle detection syntax, and optimiser behaviour.

**Beginner-Friendly Explanation**  
A recursive CTE is a query that calls itself to repeat a step over and over—like climbing a family tree one generation at a time, or following a chain of managers upward until you reach the boss—and stops when there is nothing new to process.

### Key Characteristics

- **Self‑reference**: The recursive member references the CTE name.
- **Anchor + recursive members**: The definition combines a base case with one or more recursive cases via `UNION` or `UNION ALL`.
- **Iterative evaluation**: Rows produced in one iteration feed the next iteration.
- **Termination**: Iteration stops when the recursive member returns no rows, or when a depth/cycle guard is reached.
- **Vendor syntax**: PostgreSQL, MySQL, SQLite require `RECURSIVE`; SQL Server and Oracle infer recursion from self‑reference.
- **Cycle risk**: Without guards, cyclic data causes infinite loops (or engine‑specific recursion limits).
- **Performance sensitivity**: Cost grows with recursion depth and branching factor; indexes and materialisation choices matter.

### Prerequisites

- Solid understanding of single and multiple CTEs.
- Knowledge of `UNION` vs `UNION ALL`.
- Familiarity with hierarchical or graph data modelling.
- Awareness of the target engine’s recursion limits and cycle‑detection syntax.

### Related Programming Areas

- Hierarchical data modelling (adjacency lists, nested sets, materialised paths).
- Graph algorithms (reachability, shortest paths).
- Data warehousing and hierarchy flattening.
- ETL and organisational reporting.
- Query optimisation and execution plans.

---

## Core Concepts and Features

### 1. `WITH RECURSIVE`

#### Definitions

**Core Definition**  
`WITH RECURSIVE` is the SQL clause that enables self‑referencing Common Table Expressions.

**Technical Definition**  
`WITH RECURSIVE` (PostgreSQL, MySQL, SQLite) introduces a `WITH` clause in which any CTE may reference itself. SQL Server and Oracle omit the keyword but permit recursion when a CTE references itself. The keyword applies to the entire CTE list, but only CTEs that actually self‑reference are evaluated recursively. Recursive CTEs must be formed by a non‑recursive anchor member unioned with a recursive member.

**Beginner-Friendly Explanation**  
`WITH RECURSIVE` is the special version of `WITH` that tells the database, “This query is allowed to call itself.”

#### Purposes

- To enable self‑referencing queries that iterate over hierarchical or graph data.
- To satisfy the parser requirement in engines that mandate the keyword.
- To declare intent explicitly in the query text.
- To combine recursive and non‑recursive CTEs in one `WITH` clause.

#### Syntax Rules and Structure

**PostgreSQL / MySQL / SQLite**:

```sql
WITH RECURSIVE cte_name [(col1, col2, ...)] AS (
    anchor_query
    UNION [ALL]
    recursive_query
)
main_statement;
```

**SQL Server** (no `RECURSIVE` keyword):

```sql
WITH cte_name [(col1, col2, ...)] AS (
    anchor_query
    UNION ALL
    recursive_query
)
main_statement;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `RECURSIVE` | Keyword enabling self‑reference (omitted in SQL Server/Oracle). |
| `cte_name` | Name that the recursive member references. |
| `anchor_query` | Non‑recursive base case. |
| `UNION [ALL]` | Combines anchor and recursive members; `UNION ALL` is typical for recursion. |
| `recursive_query` | Query that references `cte_name`. |
| `main_statement` | Consumes the recursive CTE. |

**Syntax rules**

- The recursive member must reference the CTE name exactly once in most engines (PostgreSQL, MySQL).
- The anchor and recursive members must produce compatible column lists (same number and compatible types).
- `UNION ALL` is strongly preferred; `UNION` (distinct) is allowed but adds deduplication cost and is disallowed in some engines for recursive members.
- In PostgreSQL and MySQL, `RECURSIVE` applies to the whole `WITH` list; non‑recursive CTEs in the same list are unaffected.
- In SQL Server, the CTE must be preceded by a semicolon if it is not the first statement in a batch.

**Constraints and limitations**

- Recursive member cannot use `ORDER BY`, `GROUP BY`, `DISTINCT`, aggregate functions, `LIMIT`, or outer joins in some engines (PostgreSQL restricts many of these).
- Recursion depth is bounded by engine defaults (SQL Server default `MAXRECURSION` = 100; PostgreSQL has no explicit limit but is bounded by memory; MySQL default `cte_max_recursion_depth` = 1000).
- The recursive member must not reference the CTE in a subquery in some engines.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Basic `WITH RECURSIVE`

```sql
-- Step 1: Anchor member produces the starting row.
-- Step 2: Recursive member adds 1 until it reaches 5.
WITH RECURSIVE numbers AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM numbers WHERE n < 5
)
SELECT n FROM numbers ORDER BY n;
```

**Expected output**:

```text
 n
---
 1
 2
 3
 4
 5
(5 rows)
```

**Why this output appears**: The anchor produces `1`; each iteration adds `1` while `n < 5`; when `n = 5`, the recursive member returns no rows, terminating the recursion.

##### Example 2: MySQL 8.0 – `WITH RECURSIVE` with column alias

```sql
WITH RECURSIVE seq (n) AS (
    SELECT 1
    UNION ALL
    SELECT n + 1 FROM seq WHERE n < 4
)
SELECT n FROM seq;
```

**Expected output**:

```text
+------+
| n    |
+------+
|    1 |
|    2 |
|    3 |
|    4 |
+------+
4 rows in set (0.00 sec)
```

**Why this output appears**: The column alias `(n)` names the single output column; recursion stops when `n` reaches `4` because the recursive member no longer qualifies.

##### Example 3: SQL Server – Recursion without the `RECURSIVE` keyword

```sql
;WITH Numbers AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM Numbers WHERE n < 5
)
SELECT n FROM Numbers ORDER BY n;
```

**Expected output**:

```text
n
-----------
1
2
3
4
5
```

**Why this output appears**: SQL Server infers recursion from the self‑reference; the leading semicolon terminates any previous statement.

#### Real-World Cases

- **Sequence generation**: Producing date ranges or numeric sequences for reporting.
- **Hierarchy traversal**: Walking an adjacency list of managers or categories.
- **Graph reachability**: Finding all nodes reachable from a start node.

---

### 2. Recursive CTE Structure

#### Definitions

**Core Definition**  
The structure of a recursive CTE is the combination of an anchor member and one or more recursive members joined by `UNION [ALL]`.

**Technical Definition**  
A recursive CTE is defined as `anchor UNION [ALL] recursive`, where the anchor is a non‑recursive query and the recursive is a query referencing the CTE name. Evaluation proceeds by initialising a working table with the anchor’s result, then repeatedly executing the recursive member against the previous iteration’s output, accumulating results, until the recursive member returns no new rows. Some engines support multiple recursive members combined via `UNION ALL`.

**Beginner-Friendly Explanation**  
The structure is: “Start with these rows (anchor), then repeatedly apply this rule (recursive) until nothing new comes out.”

#### Purposes

- To define a base case and an iterative step in one declarative statement.
- To separate initial data from the iterative transformation.
- To support multiple recursive branches (multiple recursive members) where the engine allows.
- To make termination conditions explicit.

#### Syntax Rules and Structure

**General structure**:

```sql
WITH RECURSIVE cte AS (
    -- anchor member
    SELECT ... FROM base_table WHERE ...
    UNION ALL
    -- recursive member
    SELECT ... FROM cte JOIN other_table ON ...
    WHERE termination_condition
)
SELECT ... FROM cte;
```

**Component breakdown**

| Component | Role |
|-----------|------|
| Anchor member | Produces the initial working table. |
| `UNION [ALL]` | Combines anchor and recursive results. |
| Recursive member | References the CTE and produces the next iteration. |
| Termination condition | Predicate in the recursive member that eventually yields no rows. |

**Syntax rules**

- Anchor must not reference the CTE name.
- Recursive member must reference the CTE name (typically in `FROM` or `JOIN`).
- Column lists of anchor and recursive members must be compatible.
- `UNION ALL` is preferred; `UNION` (distinct) is allowed in some engines but adds deduplication.
- Multiple recursive members can be combined with additional `UNION ALL` branches in engines that support it (SQL Server, PostgreSQL).

**Constraints and limitations**

- Recursive member cannot use `GROUP BY`, `ORDER BY`, `DISTINCT`, or aggregates in PostgreSQL.
- SQL Server does not allow `GROUP BY`, `HAVING`, `TOP`, or `ORDER BY` in the recursive member.
- MySQL restricts the recursive member to a single reference to the CTE.
- Termination must be guaranteed by the data or by a depth guard.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Anchor plus recursive member walking a hierarchy

```sql
-- Assumes table employees(employee_id, manager_id, name)
WITH RECURSIVE org AS (
    -- Anchor: top-level manager.
    SELECT employee_id, manager_id, name, 1 AS depth
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    -- Recursive: employees reporting to the previous level.
    SELECT e.employee_id, e.manager_id, e.name, o.depth + 1
    FROM employees e
    JOIN org o ON e.manager_id = o.employee_id
)
SELECT employee_id, name, depth
FROM org
ORDER BY depth, employee_id;
```

**Expected output** (illustrative):

```text
 employee_id |  name  | depth
-------------+--------+-------
           1 | Alice  |     1
           2 | Bob    |     2
           3 | Carol  |     2
           4 | David  |     3
(4 rows)
```

**Why this output appears**: The anchor selects the root (`manager_id IS NULL`); each recursive iteration joins employees whose manager was in the previous iteration. `depth` increments, and recursion stops when no employee reports to the last level.

##### Example 2: SQL Server – Structure with `UNION ALL` and depth guard

```sql
;WITH Org AS (
    SELECT employee_id, manager_id, name, 1 AS depth
    FROM Employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.manager_id, e.name, o.depth + 1
    FROM Employees e
    JOIN Org o ON e.manager_id = o.employee_id
    WHERE o.depth < 5
)
SELECT employee_id, name, depth
FROM Org
ORDER BY depth, employee_id
OPTION (MAXRECURSION 10);
```

**Expected output**:

```text
employee_id name    depth
----------- ------- -----
1           Alice   1
2           Bob     2
3           Carol   2
4           David   3
```

**Why this output appears**: The `WHERE o.depth < 5` guard caps the recursion depth, and `OPTION (MAXRECURSION 10)` sets a safety limit; the query terminates when the recursive member produces no rows.

##### Example 3: MySQL 8.0 – Anchor + recursive with column aliases

```sql
WITH RECURSIVE org (employee_id, manager_id, name, depth) AS (
    SELECT employee_id, manager_id, name, 1
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.manager_id, e.name, o.depth + 1
    FROM employees e
    JOIN org o ON e.manager_id = o.employee_id
)
SELECT employee_id, name, depth
FROM org
ORDER BY depth, employee_id;
```

**Expected output**:

```text
+-------------+-------+-------+
| employee_id | name  | depth |
+-------------+-------+-------+
|           1 | Alice |     1 |
|           2 | Bob   |     2 |
|           3 | Carol |     2 |
|           4 | David |     3 |
+-------------+-------+-------+
4 rows in set (0.00 sec)
```

**Why this output appears**: The column alias list names the four output columns; the recursive member increments `depth` and references the CTE `org`.

#### Real-World Cases

- **Org charts**: Flattening manager‑employee hierarchies.
- **Category trees**: Expanding product categories to any depth.
- **Dependency resolution**: Resolving build dependencies in a DAG.

---

### 3. Anchor Member

#### Definitions

**Core Definition**  
The anchor member is the non‑recursive part of a recursive CTE that provides the initial rows of the result set.

**Technical Definition**  
The anchor member is a query expression that does not reference the CTE name. It is combined with the recursive member via `UNION [ALL]`. Its result forms the initial working table, and it is evaluated exactly once. The anchor’s column types determine the result types of the CTE; the recursive member must produce compatible types.

**Beginner-Friendly Explanation**  
The anchor is the starting point—the root of a tree or the first value in a sequence.

#### Purposes

- To seed the recursion with starting rows.
- To define the base case of a hierarchical or iterative computation.
- To determine the result column types for the entire CTE.
- To allow multiple starting points where appropriate.

#### Syntax Rules and Structure

**Anchor pattern**:

```sql
WITH RECURSIVE cte AS (
    SELECT ... FROM base_table WHERE starting_condition
    UNION ALL
    SELECT ... FROM cte ...
)
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `SELECT ... FROM base_table` | Non‑recursive query producing initial rows. |
| `WHERE starting_condition` | Predicate selecting the base case (e.g., `manager_id IS NULL`). |

**Syntax rules**

- The anchor must not reference the CTE name.
- The anchor’s column list defines the CTE’s output columns.
- The anchor must produce at least the same number of columns as the recursive member.
- Multiple anchor queries can be combined with `UNION ALL` to seed multiple roots (engine permitting).

**Constraints and limitations**

- If the anchor returns no rows, the recursion terminates immediately with an empty result.
- Column types must be compatible with the recursive member; mismatches cause errors.
- Some engines restrict the use of `ORDER BY` in the anchor (usually allowed if combined with `LIMIT` or `TOP`).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Anchor selecting the root of a hierarchy

```sql
WITH RECURSIVE org AS (
    -- Anchor: only the CEO (no manager).
    SELECT employee_id, manager_id, name, 1 AS depth
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.manager_id, e.name, o.depth + 1
    FROM employees e
    JOIN org o ON e.manager_id = o.employee_id
)
SELECT * FROM org;
```

**Expected output**:

```text
 employee_id | manager_id |  name  | depth
-------------+------------+--------+-------
           1 |            | Alice  |     1
           2 |          1 | Bob    |     2
           3 |          1 | Carol  |     2
           4 |          2 | David  |     3
(4 rows)
```

**Why this output appears**: The anchor selects Alice because her `manager_id` is `NULL`; recursion then finds her direct reports, then their reports, and so on.

##### Example 2: MySQL – Multiple anchor queries seeding several roots

```sql
WITH RECURSIVE tree (id, parent_id, name, depth) AS (
    -- Anchor 1: root category 10.
    SELECT id, parent_id, name, 1
    FROM categories WHERE id = 10
    UNION ALL
    -- Anchor 2: root category 20.
    SELECT id, parent_id, name, 1
    FROM categories WHERE id = 20
    UNION ALL
    -- Recursive member.
    SELECT c.id, c.parent_id, c.name, t.depth + 1
    FROM categories c
    JOIN tree t ON c.parent_id = t.id
)
SELECT * FROM tree ORDER BY depth, id;
```

**Expected output**:

```text
+----+-----------+-----------+-------+
| id | parent_id | name      | depth |
+----+-----------+-----------+-------+
| 10 |      NULL | Electronics |   1 |
| 20 |      NULL | Clothing    |   1 |
| 11 |        10 | Phones      |   2 |
| 21 |        20 | Shirts      |   2 |
+----+-----------+-----------+-------+
4 rows in set (0.00 sec)
```

**Why this output appears**: Two anchor queries seed two roots; the recursive member expands each root independently.

##### Example 3: SQL Server – Anchor with `TOP` to limit starting rows

```sql
;WITH RecentChain AS (
    -- Anchor: only the 3 most recent events.
    SELECT TOP 3 event_id, parent_event_id, event_time, 1 AS depth
    FROM Events
    ORDER BY event_time DESC
    UNION ALL
    SELECT e.event_id, e.parent_event_id, e.event_time, c.depth + 1
    FROM Events e
    JOIN RecentChain c ON e.parent_event_id = c.event_id
)
SELECT * FROM RecentChain;
```

**Expected output**:

```text
event_id parent_event_id event_time              depth
-------- --------------- ----------------------- -----
101      NULL            2025-03-01 10:00:00.000 1
102      101             2025-03-01 10:05:00.000 1
103      102             2025-03-01 10:10:00.000 1
104      103             2025-03-01 10:15:00.000 2
```

**Why this output appears**: The anchor uses `TOP 3` with `ORDER BY` to seed three starting events; the recursive member then follows parent chains.

#### Real-World Cases

- **Org charts**: Anchoring on the CEO or a specific department head.
- **Category trees**: Anchoring on top‑level categories.
- **Sequence generation**: Anchoring on a start value or date.

---

### 4. Recursive Member

#### Definitions

**Core Definition**  
The recursive member is the part of a recursive CTE that references the CTE name and produces the next iteration of rows.

**Technical Definition**  
The recursive member is a query expression that references the CTE name in its `FROM` clause (or a join), typically combined with other tables. It is evaluated repeatedly against the working table produced by the previous iteration until it returns no rows. In most engines it cannot contain `GROUP BY`, `ORDER BY`, `DISTINCT`, aggregates, or outer joins.

**Beginner-Friendly Explanation**  
The recursive member is the “repeat this step” rule: it takes the rows from the previous round and finds the next set of related rows.

#### Purposes

- To define the iterative step of a recursive computation.
- To traverse relationships such as parent‑child or edge‑node.
- To extend paths, depths, or accumulations with each iteration.
- To express the termination condition that eventually yields no rows.

#### Syntax Rules and Structure

**Recursive member pattern**:

```sql
SELECT ... FROM cte
JOIN related_table ON ...
WHERE termination_predicate
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `FROM cte` | References the CTE name (the previous iteration’s rows). |
| `JOIN related_table` | Finds related rows (e.g., children of the current node). |
| `WHERE termination_predicate` | Optional guard limiting depth or excluding visited nodes. |

**Syntax rules**

- Must reference the CTE name exactly once in most engines.
- Cannot use `GROUP BY`, `ORDER BY`, `DISTINCT`, aggregate functions, or outer joins in PostgreSQL and SQL Server.
- Must produce columns compatible with the anchor.
- Must include a termination condition (implicit through data or explicit through a depth/cycle guard).

**Constraints and limitations**

- Recursive member may not reference the CTE inside a subquery in some engines.
- In MySQL, the recursive member cannot reference the CTE more than once.
- In SQL Server, the recursive member cannot use `TOP`, `ORDER BY`, `GROUP BY`, `HAVING`, or aggregates.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Recursive member walking parent‑child

```sql
WITH RECURSIVE org AS (
    SELECT employee_id, manager_id, name, 1 AS depth
    FROM employees WHERE manager_id IS NULL
    UNION ALL
    -- Recursive member: children of the previous level.
    SELECT e.employee_id, e.manager_id, e.name, o.depth + 1
    FROM employees e
    JOIN org o ON e.manager_id = o.employee_id
)
SELECT * FROM org;
```

**Expected output**:

```text
 employee_id | manager_id |  name  | depth
-------------+------------+--------+-------
           1 |            | Alice  |     1
           2 |          1 | Bob    |     2
           3 |          1 | Carol  |     2
           4 |          2 | David  |     3
(4 rows)
```

**Why this output appears**: Each iteration joins employees to the previous level via `manager_id = employee_id`; when no employee reports to the last level, the recursive member returns no rows and recursion stops.

##### Example 2: SQL Server – Recursive member with depth guard

```sql
;WITH Org AS (
    SELECT employee_id, manager_id, name, 1 AS depth
    FROM Employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.manager_id, e.name, o.depth + 1
    FROM Employees e
    JOIN Org o ON e.manager_id = o.employee_id
    WHERE o.depth < 4     -- termination guard
)
SELECT * FROM Org;
```

**Expected output**:

```text
employee_id manager_id name    depth
----------- ---------- ------- -----
1           NULL       Alice   1
2           1          Bob     2
3           1          Carol   2
4           2          David   3
```

**Why this output appears**: The `WHERE o.depth < 4` predicate stops recursion at depth 3; without it, recursion would continue until no rows match (or `MAXRECURSION` is hit).

##### Example 3: MySQL – Recursive member joining a path table

```sql
WITH RECURSIVE reach (node_id, path, depth) AS (
    SELECT node_id, CAST(node_id AS CHAR(200)), 1
    FROM nodes WHERE node_id = 1
    UNION ALL
    SELECT e.to_node, CONCAT(r.path, '->', e.to_node), r.depth + 1
    FROM edges e
    JOIN reach r ON e.from_node = r.node_id
    WHERE r.depth < 10
)
SELECT * FROM reach;
```

**Expected output**:

```text
+---------+----------+-------+
| node_id | path     | depth |
+---------+----------+-------+
|       1 | 1        |     1 |
|       2 | 1->2     |     2 |
|       3 | 1->2->3  |     3 |
+---------+----------+-------+
3 rows in set (0.00 sec)
```

**Why this output appears**: Each iteration extends the path by following `from_node → to_node` edges; `depth < 10` is a safety guard against cycles.

#### Real-World Cases

- **Hierarchies**: Walking parent‑child relationships.
- **Graphs**: Following edges from a start node.
- **Build systems**: Resolving dependent tasks iteratively.

---

### 5. Termination Conditions

#### Definitions

**Core Definition**  
Termination conditions are the rules that cause a recursive CTE to stop iterating.

**Technical Definition**  
A recursive CTE terminates when the recursive member produces an empty result set. This can occur naturally (no more matching rows) or be forced by a guard such as a depth limit, a path check, or an engine‑level recursion cap (`MAXRECURSION` in SQL Server, `cte_max_recursion_depth` in MySQL). Cycle detection mechanisms (visited‑set checks, `CYCLE` clause) also enforce termination when cycles are present.

**Beginner-Friendly Explanation**  
Termination conditions are what make the recursion stop instead of running forever—like reaching the bottom of a tree or deciding you have gone deep enough.

#### Purposes

- To prevent infinite loops when data contains cycles.
- To bound recursion depth for performance or safety.
- To limit the result set to a relevant subtree or depth.
- To make the query’s stopping criterion explicit and auditable.

#### Syntax Rules and Structure

**Natural termination** (data‑driven):

```sql
WHERE ...   -- no more matching rows
```

**Depth guard**:

```sql
WHERE depth < max_depth
```

**Path‑based cycle guard** (PostgreSQL, using arrays):

```sql
SELECT ..., path || new_node
FROM cte
WHERE new_node <> ALL(path)
```

**Engine‑level guard**:

- SQL Server: `OPTION (MAXRECURSION n)`
- MySQL: `SET SESSION cte_max_recursion_depth = n;`

**PostgreSQL `CYCLE` clause**:

```sql
WITH RECURSIVE cte AS (...) CYCLE node_id SET is_cycle TO 'Y' DEFAULT 'N' USING path
```

**Component breakdown**

| Mechanism | Description |
|-----------|-------------|
| Natural termination | Recursive member returns no rows when data exhausts. |
| Depth guard | Predicate on a depth column. |
| Path guard | Array/string of visited nodes checked in the `WHERE` clause. |
| Engine cap | Server‑level maximum recursion depth. |
| `CYCLE` clause | PostgreSQL 14+ declarative cycle detection. |

**Syntax rules**

- Guards should be expressed in the recursive member’s `WHERE` clause or via engine options.
- Depth guards must be incremented in each iteration (e.g., `depth + 1`).
- Path guards must extend the path with the new node and test against previously visited nodes.
- Engine caps are safety nets, not substitutes for explicit guards.

**Constraints and limitations**

- Natural termination is unsafe on cyclic data; explicit guards are required.
- Engine caps may silently truncate results or raise errors when exceeded.
- Path guards add per‑row storage and comparison cost.
- `CYCLE` is PostgreSQL‑specific.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Depth guard

```sql
WITH RECURSIVE tree AS (
    SELECT id, parent_id, 1 AS depth
    FROM categories WHERE parent_id IS NULL
    UNION ALL
    SELECT c.id, c.parent_id, t.depth + 1
    FROM categories c
    JOIN tree t ON c.parent_id = t.id
    WHERE t.depth < 4         -- depth guard
)
SELECT * FROM tree;
```

**Expected output**:

```text
 id | parent_id | depth
----+-----------+-------
  1 |           |     1
  2 |         1 |     2
  3 |         1 |     2
  4 |         2 |     3
  5 |         4 |     4
(5 rows)
```

**Why this output appears**: The recursive member stops when `depth` reaches 4; deeper categories are excluded.

##### Example 2: PostgreSQL – Path‑based cycle detection

```sql
WITH RECURSIVE paths AS (
    SELECT node_id, ARRAY[node_id] AS visited, 1 AS depth
    FROM nodes WHERE node_id = 1
    UNION ALL
    SELECT e.to_node, p.visited || e.to_node, p.depth + 1
    FROM edges e
    JOIN paths p ON e.from_node = p.node_id
    WHERE NOT e.to_node = ANY(p.visited)   -- cycle guard
)
SELECT * FROM paths;
```

**Expected output**:

```text
 node_id | visited | depth
---------+---------+-------
       1 | {1}     |     1
       2 | {1,2}   |     2
       3 | {1,2,3} |     3
(3 rows)
```

**Why this output appears**: The `visited` array accumulates the path; `NOT e.to_node = ANY(p.visited)` prevents revisiting nodes, terminating on cycles.

##### Example 3: SQL Server – `MAXRECURSION` safety net

```sql
;WITH Tree AS (
    SELECT id, parent_id, 1 AS depth FROM Categories WHERE parent_id IS NULL
    UNION ALL
    SELECT c.id, c.parent_id, t.depth + 1
    FROM Categories c JOIN Tree t ON c.parent_id = t.id
)
SELECT * FROM Tree
OPTION (MAXRECURSION 5);
```

**Expected output** (truncated if depth exceeds 5):

```text
id  parent_id depth
--- --------- -----
1   NULL      1
2   1         2
3   1         2
4   2         3
```

**Why this output appears**: `MAXRECURSION 5` caps iterations at 5; if the hierarchy is deeper, the query stops or raises an error depending on settings.

#### Real-World Cases

- **Graph reachability**: Preventing infinite loops on cyclic graphs.
- **Org charts**: Limiting reporting depth to a specific level.
- **Bill of materials**: Avoiding cycles in malformed BOM data.

---

### 6. Cycle Detection

#### Definitions

**Core Definition**  
Cycle detection is the set of techniques used to prevent a recursive CTE from looping indefinitely when the underlying data contains cycles.

**Technical Definition**  
Cycle detection can be implemented manually (path arrays or strings checked in the recursive member), declaratively (PostgreSQL `CYCLE` clause), or by relying on engine recursion limits. Manual detection stores the visited path per row and excludes already‑visited nodes. The `CYCLE` clause automatically adds cycle‑mark and path columns and terminates the recursion when a cycle is detected.

**Beginner-Friendly Explanation**  
Cycle detection is how you stop the recursion from going in circles by remembering where you have already been.

#### Purposes

- To ensure termination on cyclic graphs or malformed hierarchies.
- To mark rows that participate in a cycle.
- To compute paths and visited sets for auditing.
- To comply with data‑quality requirements that demand cycle‑aware traversal.

#### Syntax Rules and Structure

**Manual path‑based detection** (PostgreSQL, MySQL, SQL Server):

```sql
WITH RECURSIVE cte AS (
    SELECT node_id, CAST(node_id AS CHAR(200)) AS path
    FROM nodes WHERE node_id = 1
    UNION ALL
    SELECT e.to_node, CONCAT(c.path, ',', e.to_node)
    FROM edges e
    JOIN cte c ON e.from_node = c.node_id
    WHERE FIND_IN_SET(e.to_node, c.path) = 0   -- MySQL guard
)
```

**PostgreSQL `CYCLE` clause**:

```sql
WITH RECURSIVE cte AS (
    SELECT node_id, parent_id FROM nodes WHERE node_id = 1
    UNION ALL
    SELECT n.node_id, n.parent_id FROM nodes n JOIN cte ON n.parent_id = cte.node_id
)
CYCLE node_id SET is_cycle TO 'Y' DEFAULT 'N' USING path
```

**Component breakdown**

| Mechanism | Description |
|-----------|-------------|
| Path array/string | Stores visited nodes per row. |
| `WHERE node <> ALL(path)` | Excludes revisits (PostgreSQL). |
| `FIND_IN_SET(node, path) = 0` | Excludes revisits (MySQL). |
| `CYCLE ... SET ... USING ...` | Declarative cycle detection (PostgreSQL 14+). |

**Syntax rules**

- The path must be extended in each iteration.
- The guard must exclude nodes already present in the path.
- `CYCLE` automatically creates `is_cycle` and `path` columns; the `USING` column stores the path.
- Path guards must be compatible with column types (arrays, strings, JSON).

**Constraints and limitations**

- Path storage grows with depth; very deep paths can exceed column limits.
- `CYCLE` is PostgreSQL‑specific.
- MySQL path guards often use `FIND_IN_SET`, which requires comma‑separated strings.
- SQL Server lacks a native `CYCLE` clause; manual path columns or `MAXRECURSION` are used.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – `CYCLE` clause

```sql
WITH RECURSIVE tree AS (
    SELECT id, parent_id FROM nodes WHERE id = 1
    UNION ALL
    SELECT n.id, n.parent_id FROM nodes n JOIN tree t ON n.parent_id = t.id
)
CYCLE id SET is_cycle TO 'Y' DEFAULT 'N' USING path
SELECT id, parent_id, is_cycle, path
FROM tree
ORDER BY path;
```

**Expected output**:

```text
 id | parent_id | is_cycle |    path
----+-----------+----------+------------
  1 |           | N        | {(1)}
  2 |         1 | N        | {(1),(2)}
  3 |         2 | Y        | {(1),(2),(3)}
(3 rows)
```

**Why this output appears**: The `CYCLE` clause tracks the path and marks `is_cycle = 'Y'` when a node repeats, terminating that branch.

##### Example 2: MySQL – Path string with `FIND_IN_SET`

```sql
WITH RECURSIVE paths (node_id, path, depth) AS (
    SELECT node_id, CAST(node_id AS CHAR(200)), 1
    FROM nodes WHERE node_id = 1
    UNION ALL
    SELECT e.to_node, CONCAT(p.path, ',', e.to_node), p.depth + 1
    FROM edges e
    JOIN paths p ON e.from_node = p.node_id
    WHERE FIND_IN_SET(e.to_node, p.path) = 0
      AND p.depth < 20
)
SELECT * FROM paths;
```

**Expected output**:

```text
+---------+---------+-------+
| node_id | path    | depth |
+---------+---------+-------+
|       1 | 1       |     1 |
|       2 | 1,2     |     2 |
|       3 | 1,2,3   |     3 |
+---------+---------+-------+
3 rows in set (0.00 sec)
```

**Why this output appears**: `FIND_IN_SET` checks whether the next node is already in the path string; the depth guard adds a secondary safety limit.

##### Example 3: SQL Server – Manual path guard with a `VARCHAR` column

```sql
;WITH Paths AS (
    SELECT node_id, CAST('/' + CAST(node_id AS VARCHAR(10)) AS VARCHAR(200)) AS path, 1 AS depth
    FROM Nodes WHERE node_id = 1
    UNION ALL
    SELECT e.to_node, p.path + '/' + CAST(e.to_node AS VARCHAR(10)), p.depth + 1
    FROM Edges e
    JOIN Paths p ON e.from_node = p.node_id
    WHERE CHARINDEX('/' + CAST(e.to_node AS VARCHAR(10)) + '/', p.path + '/') = 0
)
SELECT * FROM Paths
OPTION (MAXRECURSION 100);
```

**Expected output**:

```text
node_id path    depth
------- ------- -----
1       /1      1
2       /1/2    2
3       /1/2/3  3
```

**Why this output appears**: `CHARINDEX` checks for the next node in the path string, and `MAXRECURSION` is a safety net.

#### Real-World Cases

- **Network topology**: Detecting loops in routing graphs.
- **Workflow engines**: Preventing infinite task dependencies.
- **Data quality**: Flagging malformed hierarchies.

---

### 7. Hierarchical Queries

#### Definitions

**Core Definition**  
Hierarchical queries are recursive CTEs that traverse parent‑child relationships, returning rows with depth, path, or ancestry information.

**Technical Definition**  
Hierarchical queries operate on adjacency‑list data (each row references its parent). The anchor selects roots (e.g., `parent_id IS NULL`), and the recursive member joins children to the previous level. Output often includes a computed depth and an ordering path (e.g., PostgreSQL `SEARCH DEPTH FIRST`).

**Beginner-Friendly Explanation**  
Hierarchical queries walk up or down a family tree of records, such as employees reporting to managers or categories containing subcategories.

#### Purposes

- To flatten organisational or category hierarchies for reporting.
- To compute depth, path, or ancestry for each node.
- To filter or aggregate over specific levels.
- To produce sorted output that respects hierarchy order.

#### Syntax Rules and Structure

**Pattern**:

```sql
WITH RECURSIVE h AS (
    SELECT id, parent_id, name, 1 AS depth
    FROM nodes WHERE parent_id IS NULL
    UNION ALL
    SELECT n.id, n.parent_id, n.name, h.depth + 1
    FROM nodes n JOIN h ON n.parent_id = h.id
)
SELECT * FROM h;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| Anchor | Roots (`parent_id IS NULL`). |
| Recursive member | Children (`n.parent_id = h.id`). |
| `depth` | Level counter. |
| Path | Optional path column for ordering. |

**Syntax rules**

- Anchor must select roots or a specific subtree start.
- Recursive member joins children to the previous level.
- Depth counter increments each iteration.
- Path can be constructed with array/string concatenation.

**Constraints and limitations**

- Assumes acyclic data; cycles require detection.
- Depth counter type must support the maximum depth.
- Path columns have size limits.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Org chart with depth and path

```sql
WITH RECURSIVE org AS (
    SELECT employee_id, manager_id, name, 1 AS depth, ARRAY[name] AS path
    FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.manager_id, e.name, o.depth + 1, o.path || e.name
    FROM employees e
    JOIN org o ON e.manager_id = o.employee_id
)
SELECT depth, path, name
FROM org
ORDER BY path;
```

**Expected output**:

```text
 depth |        path         | name
-------+---------------------+-------
     1 | {Alice}             | Alice
     2 | {Alice,Bob}         | Bob
     3 | {Alice,Bob,David}   | David
     2 | {Alice,Carol}       | Carol
(4 rows)
```

**Why this output appears**: The `path` array accumulates names, and `ORDER BY path` produces hierarchy‑aware ordering.

##### Example 2: SQL Server – Department hierarchy with level filtering

```sql
;WITH DeptHierarchy AS (
    SELECT dept_id, parent_dept_id, dept_name, 1 AS level
    FROM Departments WHERE parent_dept_id IS NULL
    UNION ALL
    SELECT d.dept_id, d.parent_dept_id, d.dept_name, h.level + 1
    FROM Departments d
    JOIN DeptHierarchy h ON d.parent_dept_id = h.dept_id
)
SELECT dept_name, level
FROM DeptHierarchy
WHERE level <= 3
ORDER BY level, dept_name;
```

**Expected output**:

```text
dept_name   level
----------- -----
Executive   1
Finance     2
Operations  2
HR          2
Payroll     3
```

**Why this output appears**: The recursive member walks the hierarchy, and the main query filters levels 1–3.

##### Example 3: MySQL – Category hierarchy with path string

```sql
WITH RECURSIVE cat_tree (id, name, path, depth) AS (
    SELECT id, name, CAST(name AS CHAR(200)), 1
    FROM categories WHERE parent_id IS NULL
    UNION ALL
    SELECT c.id, c.name, CONCAT(t.path, ' > ', c.name), t.depth + 1
    FROM categories c
    JOIN cat_tree t ON c.parent_id = t.id
)
SELECT depth, path FROM cat_tree ORDER BY path;
```

**Expected output**:

```text
+-------+-----------------------------+
| depth | path                        |
+-------+-----------------------------+
|     1 | Electronics                 |
|     2 | Electronics > Phones        |
|     3 | Electronics > Phones > Smart|
|     2 | Electronics > Laptops       |
+-------+-----------------------------+
4 rows in set (0.00 sec)
```

**Why this output appears**: The path string is extended with `>` separators, and `ORDER BY path` produces hierarchical ordering.

#### Real-World Cases

- **HR reporting**: Flattening org charts for headcount reports.
- **E‑commerce**: Category trees for navigation and breadcrumbs.
- **Content management**: Page hierarchies for sitemaps.

---

### 8. Tree Traversal

#### Definitions

**Core Definition**  
Tree traversal is the systematic visiting of all nodes in a tree structure, typically depth‑first or breadth‑first, using recursion.

**Technical Definition**  
In SQL, tree traversal is expressed as a recursive CTE whose anchor selects the root and whose recursive member expands children. Depth‑first order is achieved by ordering on a path column or using PostgreSQL’s `SEARCH DEPTH FIRST`; breadth‑first order is achieved by ordering on depth and a tie‑breaker, or using `SEARCH BREADTH FIRST`.

**Beginner-Friendly Explanation**  
Tree traversal is walking through every branch of a tree—either going deep down one branch first (depth‑first) or level by level (breadth‑first).

#### Purposes

- To enumerate all nodes in a tree for processing or display.
- To produce depth‑first or breadth‑first orderings.
- To compute aggregate values over subtrees.
- To support tree‑shaped UI and reporting.

#### Syntax Rules and Structure

**Depth‑first pattern** (path ordering):

```sql
WITH RECURSIVE t AS (
    SELECT id, parent_id, ARRAY[id] AS path, 1 AS depth
    FROM nodes WHERE parent_id IS NULL
    UNION ALL
    SELECT n.id, n.parent_id, t.path || n.id, t.depth + 1
    FROM nodes n JOIN t ON n.parent_id = t.id
)
SELECT * FROM t ORDER BY path;
```

**Breadth‑first pattern** (depth ordering):

```sql
SELECT * FROM t ORDER BY depth, id;
```

**PostgreSQL `SEARCH` clause**:

```sql
WITH RECURSIVE t AS (...) SEARCH DEPTH FIRST BY id SET ordercol
SELECT * FROM t ORDER BY ordercol;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| Path column | Encodes traversal order for depth‑first. |
| Depth column | Encodes level for breadth‑first. |
| `SEARCH` clause | Declarative traversal ordering (PostgreSQL 14+). |

**Syntax rules**

- Depth‑first requires a path column ordered lexicographically or a `SEARCH DEPTH FIRST` clause.
- Breadth‑first requires ordering by depth then a stable tie‑breaker.
- Recursive member must not use `ORDER BY` internally in most engines.

**Constraints and limitations**

- Path columns grow with depth.
- `SEARCH` is PostgreSQL‑specific.
- Ordering by arrays requires compatible element types.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Depth‑first traversal with `SEARCH`

```sql
WITH RECURSIVE t AS (
    SELECT id, parent_id, name FROM nodes WHERE parent_id IS NULL
    UNION ALL
    SELECT n.id, n.parent_id, n.name FROM nodes n JOIN t ON n.parent_id = t.id
)
SEARCH DEPTH FIRST BY id SET ordercol
SELECT id, name FROM t ORDER BY ordercol;
```

**Expected output**:

```text
 id | name
----+-------
  1 | Root
  2 | Child A
  4 | Grandchild A1
  3 | Child B
(4 rows)
```

**Why this output appears**: `SEARCH DEPTH FIRST` produces an ordering column that yields depth‑first order.

##### Example 2: SQL Server – Breadth‑first traversal

```sql
;WITH Tree AS (
    SELECT id, parent_id, name, 1 AS depth
    FROM Nodes WHERE parent_id IS NULL
    UNION ALL
    SELECT n.id, n.parent_id, n.name, t.depth + 1
    FROM Nodes n JOIN Tree t ON n.parent_id = t.id
)
SELECT id, name, depth
FROM Tree
ORDER BY depth, id;
```

**Expected output**:

```text
id name        depth
-- ----------- -----
1  Root        1
2  Child A     2
3  Child B     2
4  Grandchild  3
```

**Why this output appears**: Ordering by `depth` then `id` produces a breadth‑first traversal.

##### Example 3: MySQL – Depth‑first with path string

```sql
WITH RECURSIVE t (id, name, path, depth) AS (
    SELECT id, name, CAST(LPAD(id, 3, '0') AS CHAR(200)), 1
    FROM nodes WHERE parent_id IS NULL
    UNION ALL
    SELECT n.id, n.name, CONCAT(t.path, '.', LPAD(n.id, 3, '0')), t.depth + 1
    FROM nodes n JOIN t ON n.parent_id = t.id
)
SELECT id, name FROM t ORDER BY path;
```

**Expected output**:

```text
+----+--------------+
| id | name         |
+----+--------------+
|  1 | Root         |
|  2 | Child A      |
|  4 | Grandchild A1|
|  3 | Child B      |
+----+--------------+
4 rows in set (0.00 sec)
```

**Why this output appears**: Zero‑padded path segments ensure lexicographic ordering matches numeric ordering.

#### Real-World Cases

- **File systems**: Traversing directories and files.
- **Menus**: Rendering nested navigation menus.
- **Decision trees**: Enumerating branches in order.

---

### 9. Graph Traversal

#### Definitions

**Core Definition**  
Graph traversal is the recursive exploration of nodes and edges in a graph, often computing reachability or paths.

**Technical Definition**  
Graph traversal in SQL uses a recursive CTE over an edge table (`from_node`, `to_node`). The anchor selects starting nodes; the recursive member follows outgoing edges. Cycle detection is essential, and path or depth columns track traversal. Variants compute shortest paths (with cost accumulation) or all reachable nodes.

**Beginner-Friendly Explanation**  
Graph traversal is following connections—like friend‑of‑a‑friend links—from a starting point to discover all reachable nodes.

#### Purposes

- To compute reachability from a start node.
- To enumerate paths or shortest paths.
- To analyse network, social, or dependency graphs.
- To support recommendation and link‑analysis queries.

#### Syntax Rules and Structure

**Pattern**:

```sql
WITH RECURSIVE reach AS (
    SELECT start_node AS node, 0 AS hops
    FROM (VALUES (1)) AS s(start_node)
    UNION ALL
    SELECT e.to_node, r.hops + 1
    FROM edges e JOIN reach r ON e.from_node = r.node
    WHERE r.hops < max_hops
)
SELECT * FROM reach;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| Anchor | Start node(s) and initial hops/cost. |
| Recursive member | Follows edges from the previous frontier. |
| `hops` | Distance counter. |
| Cycle guard | Path or visited set to prevent loops. |

**Syntax rules**

- Edges table must have `from` and `to` columns (directed graph).
- Cycle detection or depth limit is required for graphs with cycles.
- Cost accumulation can be added for weighted shortest‑path variants.

**Constraints and limitations**

- Naïve recursion enumerates all paths; use `DISTINCT` or visited sets to avoid duplicates.
- Weighted shortest paths in SQL recursion are complex; consider dedicated graph engines.
- Depth limits may truncate reachability.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Reachability with cycle guard

```sql
WITH RECURSIVE reach AS (
    SELECT 1 AS node, ARRAY[1] AS visited, 0 AS hops
    UNION ALL
    SELECT e.to_node, r.visited || e.to_node, r.hops + 1
    FROM edges e
    JOIN reach r ON e.from_node = r.node
    WHERE NOT e.to_node = ANY(r.visited)
)
SELECT node, hops FROM reach;
```

**Expected output**:

```text
 node | hops
------+------
    1 |    0
    2 |    1
    3 |    2
(3 rows)
```

**Why this output appears**: The `visited` array prevents revisiting nodes, ensuring termination on cyclic graphs.

##### Example 2: SQL Server – All reachable nodes with depth limit

```sql
;WITH Reach AS (
    SELECT 1 AS node, 0 AS hops
    UNION ALL
    SELECT e.to_node, r.hops + 1
    FROM Edges e JOIN Reach r ON e.from_node = r.node
    WHERE r.hops < 5
)
SELECT DISTINCT node FROM Reach;
```

**Expected output**:

```text
node
----
1
2
3
4
```

**Why this output appears**: `DISTINCT` collapses duplicate reachability, and the depth limit prevents infinite recursion.

##### Example 3: MySQL – Path enumeration

```sql
WITH RECURSIVE paths (node, path, hops) AS (
    SELECT 1, CAST('1' AS CHAR(200)), 0
    UNION ALL
    SELECT e.to_node, CONCAT(p.path, '->', e.to_node), p.hops + 1
    FROM edges e
    JOIN paths p ON e.from_node = p.node
    WHERE FIND_IN_SET(e.to_node, REPLACE(p.path, '->', ',')) = 0
      AND p.hops < 10
)
SELECT * FROM paths;
```

**Expected output**:

```text
+------+--------+------+
| node | path   | hops |
+------+--------+------+
|    1 | 1      |    0 |
|    2 | 1->2   |    1 |
|    3 | 1->2->3|    2 |
+------+--------+------+
3 rows in set (0.00 sec)
```

**Why this output appears**: The path string and `FIND_IN_SET` guard prevent cycles; `hops` limits depth.

#### Real-World Cases

- **Social networks**: Friend‑of‑friend reachability.
- **Network routing**: Reachable hosts from a gateway.
- **Dependency analysis**: Transitive dependencies in build graphs.

---

### 10. Bill of Materials

#### Definitions

**Core Definition**  
A bill of materials (BOM) query is a recursive CTE that explodes a product into its components, subcomponents, and raw materials.

**Technical Definition**  
BOM data is typically stored as a parent‑child relationship (`parent_part`, `child_part`, `quantity`). A recursive CTE starts from a finished product and multiplies quantities down the hierarchy. The result lists all components with cumulative quantities, levels, and paths.

**Beginner-Friendly Explanation**  
A BOM query breaks a product into all the parts it is made of, including parts of parts, along with how many of each are needed.

#### Purposes

- To compute total material requirements for manufacturing.
- To calculate cumulative costs or weights.
- To validate BOM data for cycles or missing parts.
- To support procurement and inventory planning.

#### Syntax Rules and Structure

**Pattern**:

```sql
WITH RECURSIVE bom AS (
    SELECT parent, child, quantity, 1 AS level, quantity AS total_qty
    FROM bom_table WHERE parent = 'ProductX'
    UNION ALL
    SELECT b.parent, b.child, b.quantity, bom.level + 1, bom.total_qty * b.quantity
    FROM bom_table b
    JOIN bom ON b.parent = bom.child
)
SELECT * FROM bom;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| Anchor | Top‑level product and its direct components. |
| Recursive member | Subcomponents with quantity multiplication. |
| `level` | BOM depth. |
| `total_qty` | Cumulative quantity. |

**Syntax rules**

- Quantity multiplication accumulates down the hierarchy.
- Cycle detection is essential; malformed BOMs may contain loops.
- Anchor may select multiple top‑level products.

**Constraints and limitations**

- Decimal precision must accommodate multiplication.
- Cycles must be detected to avoid infinite recursion.
- Very deep BOMs may hit recursion limits.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – BOM explosion with quantities

```sql
WITH RECURSIVE bom AS (
    SELECT parent, child, quantity, 1 AS level, quantity::numeric AS total_qty
    FROM bom_table WHERE parent = 'Bike'
    UNION ALL
    SELECT b.parent, b.child, b.quantity, bom.level + 1, bom.total_qty * b.quantity
    FROM bom_table b
    JOIN bom ON b.parent = bom.child
)
SELECT level, parent, child, total_qty
FROM bom
ORDER BY level, parent, child;
```

**Expected output**:

```text
 level | parent | child  | total_qty
-------+--------+--------+-----------
     1 | Bike   | Frame  |         1
     1 | Bike   | Wheel  |         2
     2 | Wheel  | Spoke  |        64
     2 | Wheel  | Tire   |         2
(4 rows)
```

**Why this output appears**: The anchor selects the top‑level product; the recursive member multiplies quantities (`2 wheels × 32 spokes = 64`).

##### Example 2: SQL Server – BOM with cost roll‑up

```sql
;WITH BOM AS (
    SELECT parent, child, quantity, unit_cost, quantity * unit_cost AS cost, 1 AS level
    FROM BOMTable WHERE parent = 'Bike'
    UNION ALL
    SELECT b.parent, b.child, b.quantity, b.unit_cost,
           bom.cost * b.quantity, bom.level + 1
    FROM BOMTable b JOIN BOM ON b.parent = BOM.child
)
SELECT level, child, quantity, cost
FROM BOM
ORDER BY level, child;
```

**Expected output**:

```text
level child  quantity cost
----- ------ -------- -----
1     Frame  1        150.00
1     Wheel  2        40.00
2     Spoke  64       32.00
2     Tire   2        20.00
```

**Why this output appears**: Cost is multiplied down the hierarchy, producing cumulative cost contributions.

#### Real-World Cases

- **Manufacturing**: Material requirements planning.
- **Construction**: Component lists for assemblies.
- **Software**: Dependency trees with transitive dependencies.

---

### 11. Organizational Charts

#### Definitions

**Core Definition**  
An organizational chart query is a recursive CTE that returns the reporting hierarchy of employees, often with depth, path, and manager information.

**Technical Definition**  
Org chart data is an adjacency list (`employee_id`, `manager_id`). The anchor selects the top executive(s); the recursive member joins direct reports. Output typically includes depth, chain of command, and ordering for display.

**Beginner-Friendly Explanation**  
An org chart query lists who reports to whom, from the CEO down through every level of management.

#### Purposes

- To produce reporting lines for HR and management.
- To compute span of control and headcount per manager.
- To identify indirect reports and chain of command.
- To support succession planning and organisational analysis.

#### Syntax Rules and Structure

**Pattern**:

```sql
WITH RECURSIVE org AS (
    SELECT employee_id, manager_id, name, 1 AS level
    FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.manager_id, e.name, o.level + 1
    FROM employees e JOIN org o ON e.manager_id = o.employee_id
)
SELECT * FROM org;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| Anchor | Top executive(s). |
| Recursive member | Direct reports. |
| `level` | Depth in the org. |

**Syntax rules**

- Anchor should select exactly the roots (or a specific manager’s subtree).
- Recursive member joins on `manager_id = employee_id`.
- Cycle detection is needed if data is malformed (e.g., mutual reporting).

**Constraints and limitations**

- Multiple roots (e.g., co‑CEOs) require multiple anchor queries or a broader predicate.
- Reporting changes over time may require effective‑dated tables.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Full org chart with path

```sql
WITH RECURSIVE org AS (
    SELECT employee_id, manager_id, name, 1 AS level, ARRAY[name] AS chain
    FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.manager_id, e.name, o.level + 1, o.chain || e.name
    FROM employees e
    JOIN org o ON e.manager_id = o.employee_id
)
SELECT level, chain, name
FROM org
ORDER BY chain;
```

**Expected output**:

```text
 level |            chain             |  name
-------+------------------------------+--------
     1 | {Alice}                      | Alice
     2 | {Alice,Bob}                  | Bob
     3 | {Alice,Bob,David}            | David
     2 | {Alice,Carol}                | Carol
(4 rows)
```

**Why this output appears**: The `chain` array documents the reporting line; ordering by `chain` produces a readable org chart.

##### Example 2: MySQL – Headcount per manager

```sql
WITH RECURSIVE org (employee_id, manager_id, level) AS (
    SELECT employee_id, manager_id, 1
    FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.manager_id, o.level + 1
    FROM employees e JOIN org o ON e.manager_id = o.employee_id
)
SELECT manager_id, COUNT(*) AS direct_and_indirect_reports
FROM org
WHERE manager_id IS NOT NULL
GROUP BY manager_id;
```

**Expected output**:

```text
+------------+-----------------------------+
| manager_id | direct_and_indirect_reports |
+------------+-----------------------------+
|          1 |                           3 |
|          2 |                           1 |
+------------+-----------------------------+
2 rows in set (0.00 sec)
```

**Why this output appears**: The recursive CTE enumerates all reporting relationships; aggregation counts reports per manager.

#### Real-World Cases

- **HR analytics**: Span of control and depth analysis.
- **Succession planning**: Identifying potential successors.
- **Restructuring**: Visualising proposed changes.

---

### 12. Path Enumeration

#### Definitions

**Core Definition**  
Path enumeration is the construction of a path (sequence of nodes or edges) for each row produced by a recursive CTE.

**Technical Definition**  
Path enumeration stores the traversal history in an array or delimited string. Each iteration appends the next node, enabling ordering, cycle detection, and path‑based reporting. PostgreSQL uses arrays and `SEARCH`/`CYCLE` clauses; MySQL and SQL Server use delimited strings.

**Beginner-Friendly Explanation**  
Path enumeration records the route taken to reach each node, like breadcrumbs showing how you got there.

#### Purposes

- To produce depth‑first ordering of hierarchy output.
- To detect cycles by checking previously visited nodes.
- To report the chain of command or dependency chain.
- To support path‑based filtering and analytics.

#### Syntax Rules and Structure

**Array‑based path** (PostgreSQL):

```sql
SELECT id, ARRAY[id] AS path FROM roots
UNION ALL
SELECT n.id, p.path || n.id FROM nodes n JOIN p ON n.parent_id = p.id
```

**String‑based path** (MySQL, SQL Server):

```sql
SELECT id, CAST(id AS CHAR(200)) AS path FROM roots
UNION ALL
SELECT n.id, CONCAT(p.path, '/', n.id) FROM nodes n JOIN p ON n.parent_id = p.id
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| Path column | Accumulates visited nodes. |
| Concatenation | Appends the next node each iteration. |
| Ordering | Lexicographic/array ordering for depth‑first. |
| Cycle guard | Membership test on the path. |

**Syntax rules**

- Path column type must accommodate maximum depth.
- Concatenation must use a consistent delimiter.
- Cycle guards test membership before appending.

**Constraints and limitations**

- Strings have length limits (e.g., 8,000 bytes in SQL Server `VARCHAR`).
- Arrays are PostgreSQL‑specific.
- Delimiter choice must avoid collisions with node identifiers.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Array path with ordering

```sql
WITH RECURSIVE paths AS (
    SELECT id, ARRAY[id] AS path FROM nodes WHERE parent_id IS NULL
    UNION ALL
    SELECT n.id, p.path || n.id
    FROM nodes n JOIN paths p ON n.parent_id = p.id
)
SELECT id, path FROM paths ORDER BY path;
```

**Expected output**:

```text
 id |  path
----+---------
  1 | {1}
  2 | {1,2}
  4 | {1,2,4}
  3 | {1,3}
(4 rows)
```

**Why this output appears**: Array ordering produces depth‑first order.

##### Example 2: SQL Server – String path with delimiter

```sql
;WITH Paths AS (
    SELECT id, CAST('/' + CAST(id AS VARCHAR(10)) + '/' AS VARCHAR(200)) AS path
    FROM Nodes WHERE parent_id IS NULL
    UNION ALL
    SELECT n.id, p.path + CAST(n.id AS VARCHAR(10)) + '/'
    FROM Nodes n JOIN Paths p ON n.parent_id = p.id
)
SELECT id, path FROM Paths ORDER BY path;
```

**Expected output**:

```text
id path
-- -----------
1  /1/
2  /1/2/
4  /1/2/4/
3  /1/3/
```

**Why this output appears**: Delimited paths ensure that ordering matches hierarchy order and that membership tests are unambiguous.

#### Real-World Cases

- **Breadcrumbs**: Website navigation paths.
- **Audit trails**: Chain of approvals.
- **Data lineage**: Path from source to derived field.

---

### 13. Recursive CTE Performance

#### Definitions

**Core Definition**  
Recursive CTE performance is the cost of evaluating a recursive query, influenced by depth, branching factor, indexing, materialisation, and cycle detection overhead.

**Technical Definition**  
Performance depends on the number of iterations (depth), rows processed per iteration (branching factor), index availability on join keys, and the engine’s strategy (iterative evaluation with a working table). Cost grows multiplicatively with depth and breadth. Optimisations include indexing join columns, limiting depth, avoiding unnecessary `UNION` deduplication, materialising where supported, and using `SEARCH`/`CYCLE` efficiently.

**Beginner-Friendly Explanation**  
Recursive queries can be slow if they go deep or branch widely; performance depends on how many rows each step processes and whether the join columns are indexed.

#### Purposes

- To keep recursive queries responsive on large hierarchies.
- To avoid excessive memory or temporary storage use.
- To prevent runaway recursion from degrading the server.
- To choose between recursive CTEs and alternative models (e.g., materialised paths).

#### Syntax Rules and Structure

**Performance‑relevant patterns**:

```sql
-- Index the join key
CREATE INDEX idx_employees_manager ON employees(manager_id);

-- Depth guard
WHERE depth < 10

-- Prefer UNION ALL over UNION
UNION ALL

-- Use SEARCH for ordering instead of sorting large results repeatedly
SEARCH DEPTH FIRST BY id SET ordercol
```

**Component breakdown**

| Factor | Impact |
|--------|--------|
| Index on join key | Speeds up each iteration’s lookup. |
| `UNION ALL` | Avoids deduplication cost. |
| Depth guard | Bounds iterations. |
| Cycle guard | Adds per‑row comparison cost. |
| Materialisation | May help or hurt depending on reuse. |

**Syntax rules**

- Index the columns used in the recursive join (e.g., `manager_id`, `to_node`).
- Avoid `UNION` (distinct) unless deduplication is required.
- Use depth guards to bound work.
- Filter early in the anchor to reduce the working set.

**Constraints and limitations**

- Some engines materialise each iteration’s result; temporary storage grows with breadth.
- PostgreSQL may inline non‑recursive CTEs but recursive CTEs are evaluated iteratively.
- SQL Server’s `MAXRECURSION` default (100) may need adjustment for deeper hierarchies.
- MySQL’s `cte_max_recursion_depth` default (1000) may need adjustment.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Indexing and depth guard

```sql
-- Step 1: Ensure the join column is indexed.
CREATE INDEX IF NOT EXISTS idx_employees_manager ON employees(manager_id);

-- Step 2: Recursive query with depth guard.
WITH RECURSIVE org AS (
    SELECT employee_id, manager_id, 1 AS depth
    FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.manager_id, o.depth + 1
    FROM employees e
    JOIN org o ON e.manager_id = o.employee_id
    WHERE o.depth < 10
)
SELECT * FROM org;
```

**Expected output**:

```text
 employee_id | manager_id | depth
-------------+------------+-------
           1 |            |     1
           2 |          1 |     2
           3 |          2 |     3
(3 rows)
```

**Why this output appears**: The index speeds up the join, and the depth guard prevents excessive iterations.

##### Example 2: SQL Server – `MAXRECURSION` tuning

```sql
;WITH Org AS (
    SELECT employee_id, manager_id, 1 AS depth
    FROM Employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.manager_id, o.depth + 1
    FROM Employees e JOIN Org o ON e.manager_id = o.employee_id
)
SELECT * FROM Org
OPTION (MAXRECURSION 0);   -- 0 = no limit (use with caution)
```

**Expected output**:

```text
employee_id manager_id depth
----------- ---------- -----
1           NULL       1
2           1          2
3           2          3
...
```

**Why this output appears**: `MAXRECURSION 0` removes the default 100‑iteration cap; use only when the data is guaranteed acyclic or guarded by depth.

##### Example 3: MySQL – Session recursion depth

```sql
-- Step 1: Increase recursion depth for the session.
SET SESSION cte_max_recursion_depth = 5000;

-- Step 2: Recursive query.
WITH RECURSIVE org (employee_id, manager_id, depth) AS (
    SELECT employee_id, manager_id, 1
    FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.manager_id, o.depth + 1
    FROM employees e JOIN org o ON e.manager_id = o.employee_id
)
SELECT * FROM org;
```

**Expected output**:

```text
+-------------+------------+-------+
| employee_id | manager_id | depth |
+-------------+------------+-------+
|           1 |       NULL |     1 |
|           2 |          1 |     2 |
|           3 |          2 |     3 |
+-------------+------------+-------+
3 rows in set (0.00 sec)
```

**Why this output appears**: The session variable raises the recursion cap; the query runs to natural termination.

#### Real-World Cases

- **Large org charts**: Indexing manager columns and capping depth keeps reports fast.
- **Graph reachability**: Cycle guards and depth limits prevent exponential blow‑up.
- **BOM explosion**: Indexing parent/child columns and avoiding `UNION` deduplication improves performance.

---

## References

- PostgreSQL Documentation – `WITH` Queries (Recursive Queries) – https://www.postgresql.org/docs/current/queries-with.html
- PostgreSQL Documentation – `SEARCH` and `CYCLE` clauses – https://www.postgresql.org/docs/current/queries-with.html#QUERIES-WITH-CYCLE
- MySQL 8.4 Reference Manual – `WITH` (Common Table Expressions, Recursive) – https://dev.mysql.com/doc/refman/8.4/en/with.html
- Microsoft Learn – Recursive Queries Using Common Table Expressions – https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql
- Microsoft Learn – `MAXRECURSION` hint – https://learn.microsoft.com/en-us/sql/t-sql/queries/option-clause-transact-sql
- SQLite Documentation – Recursive Common Table Expressions – https://www.sqlite.org/lang_with.html
- Oracle Database Documentation – Recursive Subquery Factoring – https://docs.oracle.com/en/database/oracle/oracle-database/23/sqlrf/SELECT.html
- SQL:1999 Standard – Recursive Common Table Expressions (ISO/IEC 9075-2:1999)
- Use The Index, Luke – Recursive Queries – https://use-the-index-luke.com/sql/with
- Modern SQL – Recursive CTEs – https://modern-sql.com/feature/with
- Baeldung – Recursive CTEs in SQL – https://www.baeldung.com/sql/recursive-cte