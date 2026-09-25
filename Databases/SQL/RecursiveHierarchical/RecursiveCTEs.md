# SQL Recursive CTEs: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** A Recursive Common Table Expression (CTE) is a named subquery that references itself to iteratively process hierarchical or graph-structured data, enabling tree traversal, sequence generation, and path finding within a single SQL statement.

**Technical Definition:** A recursive CTE is defined using the `WITH RECURSIVE` clause and consists of two mandatory parts separated by `UNION ALL` (or `UNION`): an anchor member (a non-recursive `SELECT` that produces the initial rows) and a recursive member (a `SELECT` that references the CTE name and builds upon the previous iteration's output). Recursion terminates when the recursive member produces no new rows . The SQL standard and some database systems (PostgreSQL, MariaDB) provide built-in `SEARCH` and `CYCLE` clauses to control ordering and detect cycles .

**Beginner-Friendly Explanation:** A recursive CTE is like a set of instructions that keeps repeating until the job is done. You start with a starting point (the anchor), then say "and then find anything connected to that." The database keeps following connections until it runs out of new things to find. It's how SQL walks up and down family trees, org charts, folder structures, and social networks.

### Key Characteristics

- **Self-Referencing:** The recursive member references the CTE's own name in its `FROM` clause .
- **Anchor + Recursive Structure:** Every recursive CTE has a base case (anchor) and an iterative step (recursive member) joined by `UNION ALL` or `UNION` .
- **Iterative Execution:** Each iteration operates only on rows produced by the previous iteration .
- **Termination-Driven:** Recursion stops when the recursive member returns zero rows; explicit termination conditions are the developer's responsibility .
- **Database-Specific Features:** PostgreSQL and MariaDB support the SQL-standard `CYCLE` clause; MySQL relies on `cte_max_recursion_depth` .

### Prerequisites

- **SQL Proficiency:** Understanding of `SELECT`, `JOIN`, and basic CTE syntax.
- **Recursive CTE Support:** PostgreSQL, MySQL 8.0+, MariaDB 10.2+, SQL Server, and Oracle all support recursive CTEs.
- **Hierarchical Data Model:** Familiarity with parent-child relationships (adjacency list model).

### Related Programming Areas

- **Hierarchical Data Management:** Org charts, category trees, folder structures .
- **Graph Analytics:** Social networks, network topologies, path finding .
- **Sequence Generation:** Number sequences, date ranges, Fibonacci series .
- **Bill-of-Materials (BOM):** Manufacturing component explosion .

### Core Concepts / Features

1. Anchor Member
2. Recursive Member
3. Termination Conditions
4. Depth Tracking
5. Path Construction
6. Cycle Detection and Loop Prevention
7. BFS vs. DFS Ordering

---

## 1. Anchor Member

### Definitions

**Core Definition:** The anchor member is the non-recursive `SELECT` statement that produces the initial row set for a recursive CTE.

**Technical Definition:** The anchor member is the first part of a recursive CTE, placed before the `UNION ALL` or `UNION` operator. It does not reference the CTE name and establishes the starting point for recursion. The column types of the entire CTE result are inferred exclusively from the anchor member .

**Beginner-Friendly Explanation:** The anchor is where you start. It's like saying "begin with this person" or "start at the root category." Everything else builds from here.

### Purposes

- To establish the starting row(s) for recursive traversal.
- To define the column structure and data types of the CTE.
- To provide the base case for the recursion.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
WITH RECURSIVE cte_name (column_list) AS (
    -- Anchor member (non-recursive)
    SELECT initial_columns
    FROM initial_table
    WHERE starting_condition
    
    UNION ALL
    
    -- Recursive member
    SELECT ...
)
SELECT * FROM cte_name;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `WITH RECURSIVE` | Declares a recursive CTE |
| `cte_name (column_list)` | Names the CTE and its output columns |
| `SELECT ... WHERE starting_condition` | Anchor member defining initial rows |
| `UNION ALL` | Separates anchor and recursive members |

**Syntax Rules:**

- The anchor member must not reference the CTE name .
- Column types are determined from the anchor member only; the recursive member's types are ignored for this purpose .
- The anchor can be a `UNION` of multiple `SELECT` statements .

**Constraints and Limitations:**

- If the anchor returns no rows, the CTE produces no rows.
- The anchor must provide values for all CTE columns.

### Annotated Complete Code Examples

**Example 1: Number Sequence (Anchor Starts at 1)**

```sql
-- Generate numbers from 1 to 10
WITH RECURSIVE counter(prev_val) AS (
    -- Anchor: start with 1
    SELECT 1
    
    UNION ALL
    
    -- Recursive: add 1 to previous value
    SELECT prev_val + 1
    FROM counter
    WHERE prev_val < 10
)
SELECT * FROM counter;

-- Expected Output:
--  prev_val
-- ----------
--         1
--         2
--         3
--         4
--         5
--         6
--         7
--         8
--         9
--        10
```

**Why this output occurs:** The anchor member `SELECT 1` produces the initial row. The recursive member adds 1 to each previous value, stopping when `prev_val < 10` is no longer true. This simple pattern demonstrates the anchor's role as the starting point .

**Example 2: Tree Traversal (Anchor Selects Root)**

```sql
-- Assume categories table with id, name, parent_id
-- Anchor selects the root category
WITH RECURSIVE category_tree AS (
    -- Anchor: start with the root (no parent)
    SELECT id, name, parent_id, 0 AS level
    FROM categories
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- Recursive: find children
    SELECT c.id, c.name, c.parent_id, ct.level + 1
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT * FROM category_tree;

-- Expected Output (conceptual):
--  id | name        | parent_id | level
-- ----+-------------+-----------+-------
--   1 | Electronics | NULL      |     0
--   2 | Computers   | 1         |     1
--   3 | Laptops     | 2         |     2
```

**Why this output occurs:** The anchor selects the root category (`parent_id IS NULL`). The recursive member finds all children of the current level, incrementing the level each time .

### Real-World Cases

**Case 1: Organizational Chart**

An HR system uses an anchor that selects the CEO (the only employee with no manager) to build the complete org chart.

**Case 2: File System Root**

A document management system uses an anchor that selects the root folder (`parent_id IS NULL`) to list all files in the hierarchy.

### References

- Sisense Blog — Solving the Traveling Salesman Problem with Postgres Recursive CTEs - https://www.sisense.com/blog/postgres-recursive-cte/
- MySQL 8.2 Reference Manual (PDF) - https://downloads.mysql.com/docs/refman-8.2-en.a4.pdf
- Cybrosys — How to Write Hierarchical Queries in PostgreSQL - https://www.cybrosys.com/research-and-development/postgres/how-to-write-hierarchical-queries-in-postgresql

---

## 2. Recursive Member

### Definitions

**Core Definition:** The recursive member is the `SELECT` statement that references the CTE's own name, generating additional rows by joining against the previous iteration's output.

**Technical Definition:** The recursive member follows the `UNION ALL` (or `UNION`) operator and must reference the CTE name exactly once in its `FROM` clause (not in any subquery). It can join the CTE with other tables. The recursive member operates only on rows produced by the previous iteration, not the entire accumulated result .

**Beginner-Friendly Explanation:** The recursive member is the "and then what?" step. After finding the starting point, it says "now find everything connected to what we just found." It keeps doing this until there's nothing new.

### Purposes

- To generate new rows based on the previous iteration's output.
- To traverse parent-child relationships (up or down the hierarchy).
- To build iterative computations (sequences, running totals).

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
WITH RECURSIVE cte_name AS (
    -- Anchor
    SELECT ...
    
    UNION ALL
    
    -- Recursive member
    SELECT ...
    FROM source_table
    JOIN cte_name ON join_condition
    WHERE termination_condition
)
SELECT * FROM cte_name;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `FROM source_table JOIN cte_name` | Joins the base table with the previous iteration's output |
| `WHERE termination_condition` | Controls when recursion stops |

**Syntax Rules:**

- The recursive member must reference the CTE only once, in the `FROM` clause .
- The CTE cannot appear on the right side of a `LEFT JOIN` .
- Aggregate functions, window functions, `GROUP BY`, `ORDER BY`, and `DISTINCT` are prohibited in the recursive member .
- `LIMIT` and `OFFSET` are permitted in the recursive member (MySQL) .

**Constraints and Limitations:**

- The recursive member must produce rows of the same column count and compatible types as the anchor.
- Without a termination condition, the recursion may run until the database's recursion limit is hit .

### Annotated Complete Code Examples

**Example 1: Downward Traversal (Finding Descendants)**

```sql
-- Find all descendants of 'Electronics' (id = 1)
WITH RECURSIVE descendants AS (
    -- Anchor: Electronics itself
    SELECT id, name, parent_id
    FROM categories
    WHERE id = 1
    
    UNION ALL
    
    -- Recursive: find children of current level
    SELECT c.id, c.name, c.parent_id
    FROM categories c
    JOIN descendants d ON c.parent_id = d.id
)
SELECT * FROM descendants;

-- Expected Output (conceptual):
--  id | name        | parent_id
-- ----+-------------+-----------
--   1 | Electronics | NULL
--   2 | Computers   | 1
--   5 | Phones      | 1
--   3 | Laptops     | 2
--   4 | Desktops    | 2
--   6 | Smartphones | 5
```

**Why this output occurs:** The anchor selects Electronics. The recursive member finds all categories whose `parent_id` matches the previous level's `id`, repeating until no new children are found .

**Example 2: Upward Traversal (Finding Ancestors)**

```sql
-- Find all ancestors of 'Laptops' (id = 3)
WITH RECURSIVE ancestors AS (
    -- Anchor: Laptops
    SELECT id, name, parent_id
    FROM categories
    WHERE id = 3
    
    UNION ALL
    
    -- Recursive: find parent of current node
    SELECT c.id, c.name, c.parent_id
    FROM categories c
    JOIN ancestors a ON c.id = a.parent_id
)
SELECT * FROM ancestors;

-- Expected Output:
--  id | name        | parent_id
-- ----+-------------+-----------
--   3 | Laptops     | 2
--   2 | Computers   | 1
--   1 | Electronics | NULL
```

**Why this output occurs:** The recursive member joins `categories` where `c.id = a.parent_id`, walking up the tree from Laptops to Computers to Electronics .

### Real-World Cases

**Case 1: Bill-of-Materials Explosion**

A manufacturing system uses the recursive member to expand sub-assemblies: given a bicycle, find all wheels, then all tires and spokes, then all raw materials for those components .

**Case 2: Social Network Friend-of-Friend**

A social platform uses the recursive member to find friends of friends: start with user A, find A's friends, then find their friends, up to a depth limit.

### References

- MySQL 8.1 Reference Manual (PDF) - https://downloads.mysql.com/docs/refman-8.1-en.pdf
- PostgreSQL 17.7.1 Documentation (PDF) - http://repo.postgrespro.ru/doc/sdm/17.7.1/en/postgres-A4.pdf
- Sisense Blog — Solving the Traveling Salesman Problem with Postgres Recursive CTEs - https://www.sisense.com/blog/postgres-recursive-cte/

---

## 3. Termination Conditions

### Definitions

**Core Definition:** A termination condition is a `WHERE` clause or other mechanism in the recursive member that prevents infinite recursion by ensuring the recursive member eventually produces no new rows.

**Technical Definition:** Recursion terminates naturally when the recursive member returns an empty set. For tree structures, this happens automatically when leaf nodes have no children. For graphs with cycles, explicit termination conditions (depth limits, path tracking, or engine-native cycle clauses) are required . MySQL enforces a default recursion depth limit of 1000 via `cte_max_recursion_depth` .

**Beginner-Friendly Explanation:** Termination is the "stop" condition. Without it, the database would keep asking "any more?" forever. It's like a recipe that says "keep adding flour until the dough is no longer sticky"—when the condition is met, you stop.

### Purposes

- To prevent infinite loops and runaway queries.
- To limit recursion depth for performance or business reasons.
- To define the natural end of a hierarchy traversal (leaves have no children).

### Syntax Rules and Structure

**Termination Mechanisms:**

| Mechanism | Syntax | Use Case |
|-----------|--------|----------|
| Natural (tree) | No explicit condition needed | Trees (leaf nodes have no children) |
| Depth limit | `WHERE depth < N` | Limiting traversal depth |
| Path tracking | `WHERE NOT node = ANY(path)` | Cycle detection (manual) |
| Engine limit | `cte_max_recursion_depth` (MySQL) | Safety guard against runaway recursion |
| `LIMIT` in recursive member | `... LIMIT 1000` | MySQL-specific row limit |

**Syntax Rules:**

- Place the termination condition in the recursive member's `WHERE` clause .
- For trees, the natural termination (no matching children) is sufficient.
- For graphs, explicit cycle detection or depth limits are required .

**Constraints and Limitations:**

- MySQL's default `cte_max_recursion_depth` is 1000; exceeding it causes an error .
- `LIMIT` in the recursive member is MySQL-specific and not portable .
- A depth limit may miss deep paths if set too low.

### Annotated Complete Code Examples

**Example 1: Natural Tree Termination**

```sql
-- Tree traversal terminates naturally at leaves
WITH RECURSIVE tree AS (
    SELECT id, name, parent_id FROM nodes WHERE id = 1
    UNION ALL
    SELECT n.id, n.name, n.parent_id
    FROM nodes n
    JOIN tree t ON n.parent_id = t.id
)
SELECT * FROM tree;
-- Stops automatically when no node has parent_id matching the current level
```

**Why this works:** In a tree, leaf nodes have no children. When the recursive member finds no matching rows, it returns an empty set, and recursion stops .

**Example 2: Depth Limit (MySQL)**

```sql
-- Traverse up to 5 levels deep
WITH RECURSIVE descendants AS (
    SELECT id, name, parent_id, 0 AS depth
    FROM categories WHERE id = 1
    
    UNION ALL
    
    SELECT c.id, c.name, c.parent_id, d.depth + 1
    FROM categories c
    JOIN descendants d ON c.parent_id = d.id
    WHERE d.depth < 5  -- Termination: stop at depth 5
)
SELECT * FROM descendants;
```

**Why this works:** The `WHERE d.depth < 5` condition ensures the recursive member stops producing rows once depth reaches 5. This is a safety guard against deep hierarchies .

**Example 3: MySQL LIMIT in Recursive Member**

```sql
-- Generate at most 10000 rows
WITH RECURSIVE cte (n) AS (
    SELECT 1
    UNION ALL
    SELECT n + 1 FROM cte LIMIT 10000
)
SELECT * FROM cte;
```

**Why this works:** MySQL documentation confirms that `LIMIT` in the recursive member "stops the generation of rows as soon as the requested number of them has been produced," which is more efficient than filtering in the outer query .

### Real-World Cases

**Case 1: Preventing Runaway Queries**

A development team adds `WHERE depth < 100` to all recursive CTEs as a safety guard. If a data anomaly creates a cycle, the query stops at depth 100 instead of running indefinitely.

**Case 2: Business Rule Limiting**

A reporting query only needs to traverse 3 levels of a category tree. The termination condition `WHERE level < 3` ensures performance remains consistent.

### References

- MySQL 8.2 Reference Manual (PDF) - https://downloads.mysql.com/docs/refman-8.2-en.a4.pdf
- GitHub — Recursive CTEs (Data Engineering Interview Patterns) - https://raw.githubusercontent.com/sergiorgiraldo/Python-lang/refs/heads/master/data-engineering-interview-patterns/sql/04_recursive_ctes/README.md
- Vertabelo Academy — Recursion Syntax - https://academy.vertabelo.com/course/postgresql-recursive-queries/recursive-cte/the-very-basics/recursion-syntax

---

## 4. Depth Tracking

### Definitions

**Core Definition:** Depth tracking is the practice of adding a column to a recursive CTE that increments with each recursion level, indicating how deep a node is in the hierarchy.

**Technical Definition:** A depth column is initialized to 0 (or 1) in the anchor member and incremented by 1 in the recursive member. It serves as both a termination condition and a useful output column for understanding hierarchy structure and implementing level-based logic.

**Beginner-Friendly Explanation:** Depth tracking is like counting floors in a building. The ground floor is 0, the first floor is 1, and so on. SQL adds a number to each row showing how many "steps" it is from the starting point.

### Purposes

- To provide a termination condition for depth-limited traversal.
- To display hierarchy levels in output (indentation, grouping).
- To enforce business rules based on hierarchy depth.
- To analyze tree structure (average depth, maximum depth).

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
WITH RECURSIVE cte_name AS (
    -- Anchor: depth starts at 0
    SELECT id, name, 0 AS depth
    FROM table_name
    WHERE starting_condition
    
    UNION ALL
    
    -- Recursive: increment depth
    SELECT t.id, t.name, cte.depth + 1
    FROM table_name t
    JOIN cte_name cte ON t.parent_id = cte.id
    WHERE cte.depth < max_depth  -- Optional depth limit
)
SELECT * FROM cte_name;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `0 AS depth` | Initializes depth at the anchor |
| `cte.depth + 1` | Increments depth each level |
| `WHERE cte.depth < N` | Optional depth-based termination |

**Syntax Rules:**

- Initialize depth to 0 (or 1) in the anchor member.
- Increment by 1 in the recursive member.
- Use depth in `WHERE` for level-limited traversal.
- Depth can be used in the outer query for indentation or grouping.

### Annotated Complete Code Examples

**Example 1: Hierarchy with Depth Display**

```sql
-- Display org chart with indentation based on depth
WITH RECURSIVE org_chart AS (
    SELECT id, name, manager_id, 0 AS depth
    FROM employees WHERE manager_id IS NULL
    
    UNION ALL
    
    SELECT e.id, e.name, e.manager_id, oc.depth + 1
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT 
    id,
    name,
    depth,
    REPEAT('  ', depth) || name AS indented_name
FROM org_chart
ORDER BY depth, name;

-- Expected Output:
--  id | name  | depth | indented_name
-- ----+-------+-------+----------------
--   1 | Alice |     0 | Alice
--   2 | Bob   |     1 |   Bob
--   3 | Carol |     1 |   Carol
--   4 | Dave  |     2 |     Dave
--   5 | Eve   |     2 |     Eve
```

**Why this output occurs:** The anchor sets depth 0 for Alice (CEO). Bob and Carol get depth 1, Dave and Eve get depth 2. The `REPEAT('  ', depth)` function creates indentation based on depth .

**Example 2: Depth-Limited Traversal**

```sql
-- Find employees up to 2 levels below the CEO
WITH RECURSIVE limited_org AS (
    SELECT id, name, manager_id, 0 AS depth
    FROM employees WHERE manager_id IS NULL
    
    UNION ALL
    
    SELECT e.id, e.name, e.manager_id, lo.depth + 1
    FROM employees e
    JOIN limited_org lo ON e.manager_id = lo.id
    WHERE lo.depth < 2  -- Stop at depth 2
)
SELECT * FROM limited_org;

-- Expected Output: Only Alice (0), Bob (1), Carol (1), Dave (2), Eve (2)
-- Frank and Grace (depth 3) are excluded
```

**Why this output occurs:** The `WHERE lo.depth < 2` condition prevents the recursive member from generating rows at depth 3 or beyond. Frank and Grace (Carol's reports) are excluded .

### Real-World Cases

**Case 1: Menu Indentation**

An e-commerce site generates a category menu with visual indentation. Depth tracking provides the level, and `REPEAT` or `CONCAT` creates the indentation .

**Case 2: Report Grouping**

A financial report groups accounts by hierarchy level. Depth 0 accounts are headers, depth 1 are sub-headers, depth 2 are line items.

### References

- Cybrosys — How to Write Hierarchical Queries in PostgreSQL - https://www.cybrosys.com/research-and-development/postgres/how-to-write-hierarchical-queries-in-postgresql
- GitHub — Recursive CTEs (Data Engineering Interview Patterns) - https://raw.githubusercontent.com/sergiorgiraldo/Python-lang/refs/heads/master/data-engineering-interview-patterns/sql/04_recursive_ctes/README.md

---

## 5. Path Construction

### Definitions

**Core Definition:** Path construction is the technique of building a cumulative string or array in a recursive CTE that records the sequence of nodes visited from the starting point to the current row.

**Technical Definition:** A path column is initialized in the anchor member with the starting node's identifier and updated in the recursive member by appending (or prepending) the current node's identifier. PostgreSQL documentation notes that the path array is "useful in its own right as representing the 'path' taken to reach any particular row" . Paths can be used for breadcrumb generation, cycle detection, and depth-first ordering.

**Beginner-Friendly Explanation:** Path construction is like writing down every step you took to get somewhere. Starting at "Home," then "Kitchen," then "Fridge"—the path tells you exactly how you got there. It's useful for breadcrumbs ("Home > Kitchen > Fridge") and for detecting if you've been somewhere before.

### Purposes

- To generate breadcrumb navigation (e.g., "Electronics > Computers > Laptops").
- To enable cycle detection by checking if a node appears in the path.
- To provide materialized path storage for faster future queries.
- To order results in depth-first order .

### Syntax Rules and Structure

**Complete General Syntax (String Path):**

```sql
WITH RECURSIVE cte_name AS (
    -- Anchor: initialize path
    SELECT id, name, CAST(name AS CHAR(1000)) AS path
    FROM table_name WHERE starting_condition
    
    UNION ALL
    
    -- Recursive: append to path
    SELECT t.id, t.name, CONCAT(cte.path, ' > ', t.name)
    FROM table_name t
    JOIN cte_name cte ON t.parent_id = cte.id
)
SELECT * FROM cte_name;
```

**Complete General Syntax (Array Path - PostgreSQL):**

```sql
WITH RECURSIVE cte_name AS (
    SELECT id, name, ARRAY[id] AS path
    FROM table_name WHERE starting_condition
    
    UNION ALL
    
    SELECT t.id, t.name, path || t.id
    FROM table_name t
    JOIN cte_name cte ON t.parent_id = cte.id
)
SELECT * FROM cte_name;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `CAST(name AS CHAR(1000))` | Initializes path with anchor name (string) |
| `ARRAY[id]` | Initializes path with anchor ID (array) |
| `CONCAT(cte.path, ' > ', t.name)` | Appends child name to parent path |
| `path || t.id` | Appends child ID to parent array |

**Syntax Rules:**

- Initialize the path in the anchor member .
- Append or prepend in the recursive member.
- String paths require sufficient column width (`CAST` to avoid truncation).
- Array paths are PostgreSQL-specific but more memory-efficient for cycle detection.

**Constraints and Limitations:**

- String paths grow longer with each level; very deep hierarchies may exceed column limits.
- Array paths consume more memory per row.
- Path construction adds overhead to each recursion step.

### Annotated Complete Code Examples

**Example 1: Breadcrumb Path (String)**

```sql
-- Build breadcrumb path for categories
WITH RECURSIVE category_path AS (
    -- Anchor: root categories
    SELECT id, name, parent_id, 
           CAST(name AS CHAR(1000)) AS breadcrumb
    FROM categories
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- Recursive: append child to parent path
    SELECT c.id, c.name, c.parent_id,
           CONCAT(cp.breadcrumb, ' > ', c.name)
    FROM categories c
    JOIN category_path cp ON c.parent_id = cp.id
)
SELECT id, name, breadcrumb
FROM category_path
ORDER BY breadcrumb;

-- Expected Output:
--  id | name        | breadcrumb
-- ----+-------------+-------------------------------------
--   1 | Electronics | Electronics
--   2 | Computers   | Electronics > Computers
--   3 | Laptops     | Electronics > Computers > Laptops
--   4 | Desktops    | Electronics > Computers > Desktops
--   5 | Phones      | Electronics > Phones
--   6 | Smartphones | Electronics > Phones > Smartphones
```

**Why this output occurs:** The anchor starts with root categories (Electronics). The recursive member appends child names to the parent's breadcrumb, building the full path .

**Example 2: Array Path for Cycle Detection (PostgreSQL)**

```sql
-- Use array path to detect cycles in a graph
WITH RECURSIVE search_graph(id, link, data, depth, path, is_cycle) AS (
    -- Anchor: start with all nodes
    SELECT g.id, g.link, g.data, 1, ARRAY[g.id], false
    FROM graph g
    
    UNION ALL
    
    -- Recursive: follow links, track path
    SELECT g.id, g.link, g.data, sg.depth + 1,
           path || g.id,
           g.id = ANY(path)  -- Cycle detected if ID already in path
    FROM graph g, search_graph sg
    WHERE g.id = sg.link AND NOT is_cycle
)
SELECT * FROM search_graph;
```

**Why this output occurs:** The `path` array accumulates visited node IDs. The `is_cycle` flag becomes true when `g.id = ANY(path)`, indicating a revisit. The `WHERE ... AND NOT is_cycle` condition stops traversal on cycles .

### Real-World Cases

**Case 1: E-Commerce Breadcrumbs**

An online store displays breadcrumb navigation on product pages: "Home > Electronics > Computers > Laptops > MacBook Pro." A recursive CTE builds this path from the category hierarchy.

**Case 2: Materialized Path Storage**

Some systems store the path as a permanent column (materialized path) for faster descendant queries. Recursive CTEs are used during initial population or migration .

### References

- PostgreSQL 9.4 Documentation (PDF) - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.4/postgresql-9.4-A4.pdf
- Cybrosys — How to Write Hierarchical Queries in PostgreSQL - https://www.cybrosys.com/research-and-development/postgres/how-to-write-hierarchical-queries-in-postgresql

---

## 6. Cycle Detection and Loop Prevention

### Definitions

**Core Definition:** Cycle detection is the mechanism that prevents infinite recursion when a graph contains cycles (A → B → A), while loop prevention is the broader set of techniques to ensure recursion terminates.

**Technical Definition:** Cycles occur when following relationships leads back to a previously visited node. Detection methods include: manual path tracking (checking if a node ID appears in a path array), `UNION` instead of `UNION ALL` (deduplication), depth limits, and engine-native `CYCLE` clauses. PostgreSQL and MariaDB support the SQL-standard `CYCLE` clause, which internally rewrites to path-based detection .

**Beginner-Friendly Explanation:** A cycle is like a circular path—you keep going around and around. Cycle detection is the "have I been here before?" check that stops the loop. The database says "I've seen this node already, so I'll stop."

### Purposes

- To prevent infinite recursion and query timeouts.
- To correctly traverse graphs with cycles (social networks, network topologies).
- To identify and report cycles as data quality issues.
- To ensure query results are complete without duplicates.

### Syntax Rules and Structure

**Manual Cycle Detection (PostgreSQL):**

```sql
WITH RECURSIVE search_graph(id, link, data, depth, path, is_cycle) AS (
    SELECT g.id, g.link, g.data, 1, ARRAY[g.id], false
    FROM graph g
    UNION ALL
    SELECT g.id, g.link, g.data, sg.depth + 1,
           path || g.id,
           g.id = ANY(path)
    FROM graph g, search_graph sg
    WHERE g.id = sg.link AND NOT is_cycle
)
SELECT * FROM search_graph;
```

**Engine-Native CYCLE Clause (PostgreSQL):**

```sql
WITH RECURSIVE search_graph(id, link, data, depth) AS (
    SELECT g.id, g.link, g.data, 1
    FROM graph g
    UNION ALL
    SELECT g.id, g.link, g.data, sg.depth + 1
    FROM graph g, search_graph sg
    WHERE g.id = sg.link
) CYCLE id SET is_cycle USING path
SELECT * FROM search_graph;
```

**MariaDB CYCLE ... RESTRICT:**

```sql
WITH RECURSIVE cte (depth, from_, to_) AS (
    SELECT 0, 1, 1
    UNION DISTINCT
    SELECT depth+1, t1.from_, t1.to_
    FROM t1, cte WHERE t1.from_ = cte.to_
) CYCLE from_, to_ RESTRICT
SELECT * FROM cte;
```

**Syntax Rules:**

- Manual detection uses `path || id` to append and `id = ANY(path)` to check .
- PostgreSQL `CYCLE id SET is_cycle USING path` adds cycle detection columns implicitly .
- MariaDB `CYCLE col1, col2 RESTRICT` stops recursion on cycle detection without adding columns .
- MySQL lacks a `CYCLE` clause; use manual detection or `UNION` deduplication.

**Constraints and Limitations:**

- `UNION` (not `UNION ALL`) deduplicates but may not detect all cycles in path-tracking scenarios .
- Manual path tracking consumes memory proportional to path length.
- `CYCLE` clause is not portable across all database systems.
- MySQL's `cte_max_recursion_depth` (default 1000) is a safety net, not a cycle detector .

### Annotated Complete Code Examples

**Example 1: Manual Cycle Detection with Path Array**

```sql
-- Graph with a cycle: A → B → C → A
WITH RECURSIVE search AS (
    -- Anchor: start at A
    SELECT id, link, ARRAY[id] AS path, false AS is_cycle
    FROM graph WHERE id = 'A'
    
    UNION ALL
    
    -- Recursive: follow links, detect cycles
    SELECT g.id, g.link, path || g.id, g.id = ANY(path)
    FROM graph g
    JOIN search s ON g.id = s.link
    WHERE NOT s.is_cycle
)
SELECT * FROM search;

-- Expected Output (conceptual):
--  id | link | path       | is_cycle
-- ----+------+------------+----------
--  A  | B    | {A}        | false
--  B  | C    | {A,B}      | false
--  C  | A    | {A,B,C}    | false
--  A  | B    | {A,B,C,A}  | true    ← Cycle detected
```

**Why this output occurs:** The `path` array accumulates visited nodes. When `g.id = ANY(path)` is true (A is already in the path), `is_cycle` becomes true, and the `WHERE NOT s.is_cycle` condition prevents further traversal .

**Example 2: PostgreSQL CYCLE Clause**

```sql
-- Same graph traversal using built-in CYCLE clause
WITH RECURSIVE search AS (
    SELECT id, link FROM graph WHERE id = 'A'
    UNION ALL
    SELECT g.id, g.link FROM graph g JOIN search s ON g.id = s.link
) CYCLE id SET is_cycle USING path
SELECT * FROM search;

-- The CYCLE clause adds is_cycle and path columns automatically
-- Internally equivalent to manual path tracking
```

**Why this works:** PostgreSQL documentation states that the `CYCLE` clause "specifies first the list of columns to track for cycle detection, then a column name that will show whether a cycle has been detected, and finally the name of another column that will track the path" .

**Example 3: MariaDB CYCLE ... RESTRICT**

```sql
-- MariaDB relaxed cycle detection
SET max_recursive_iterations = 10;

WITH RECURSIVE cte (depth, from_, to_) AS (
    SELECT 0, 1, 1
    UNION DISTINCT
    SELECT depth+1, t1.from_, t1.to_
    FROM t1, cte WHERE t1.from_ = cte.to_
) CYCLE from_, to_ RESTRICT
SELECT * FROM cte;
```

**Why this works:** MariaDB documentation explains that "with the use of CYCLE ... RESTRICT it makes no difference whether the CTE uses UNION ALL or UNION DISTINCT anymore. UNION ALL means 'all rows, but without cycles'" .

### Real-World Cases

**Case 1: Social Network Friend Recommendations**

A social platform traverses friendship graphs that may contain mutual connections. Cycle detection prevents recommending a user to themselves or creating duplicate recommendation paths.

**Case 2: Network Topology Validation**

A network management system uses cycle detection to identify routing loops in network topologies, reporting them as configuration errors.

**Case 3: Data Quality Auditing**

A data steward runs cycle detection queries to find circular references in organizational hierarchies (e.g., A manages B, B manages C, C manages A), flagging them for correction.

### References

- PostgreSQL 15 Documentation (PDF) - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/15/postgresql-15-A4.pdf
- PostgreSQL 17.7.1 Documentation (PDF) - http://repo.postgrespro.ru/doc/sdm/17.7.1/en/postgres-A4.pdf
- MariaDB Documentation (GitHub) - https://raw.githubusercontent.com/mariadb-corporation/mariadb-docs/refs/heads/main/server/reference/sql-statements/data-manipulation/selecting-data/common-table-expressions/with.md
- PostgreSQL 9.4 Documentation (PDF) - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.4/postgresql-9.4-A4.pdf
- GitHub — Recursive CTEs (Data Engineering Interview Patterns) - https://raw.githubusercontent.com/sergiorgiraldo/Python-lang/refs/heads/master/data-engineering-interview-patterns/sql/04_recursive_ctes/README.md

---

## 7. Breadth-First Search (BFS) vs. Depth-First Search (DFS) Ordering

### Definitions

**Core Definition:** BFS and DFS are traversal orders for recursive queries. BFS explores all nodes at the current depth before going deeper; DFS explores one branch fully before backtracking.

**Technical Definition:** PostgreSQL documentation states that "the recursive query evaluation algorithm produces its output in breadth-first search order" by default. DFS order can be achieved by constructing a path column and ordering by it in the outer query . SQL-standard `SEARCH DEPTH FIRST` and `SEARCH BREADTH FIRST` clauses explicitly control this ordering .

**Beginner-Friendly Explanation:** BFS is like reading a book level by level: first all chapter titles, then all section titles, then all subsections. DFS is like reading one chapter completely before moving to the next. SQL can do both, but you often need to tell it which one you want.

### Purposes

- To control the order in which hierarchy nodes are returned.
- To match traversal order to business requirements (e.g., menu display = DFS).
- To enable efficient processing of level-based operations (BFS for batch processing).

### Syntax Rules and Structure

**BFS (Default):**

```sql
-- Default recursive CTE output is BFS
WITH RECURSIVE tree AS (
    SELECT id, name, parent_id, 0 AS depth FROM nodes WHERE id = 1
    UNION ALL
    SELECT n.id, n.name, n.parent_id, t.depth + 1
    FROM nodes n JOIN tree t ON n.parent_id = t.id
)
SELECT * FROM tree;
-- Output: level 0, then level 1, then level 2, ...
```

**DFS via Path Ordering:**

```sql
WITH RECURSIVE tree AS (
    SELECT id, name, parent_id, ARRAY[name] AS path
    FROM nodes WHERE id = 1
    UNION ALL
    SELECT n.id, n.name, n.parent_id, path || n.name
    FROM nodes n JOIN tree t ON n.parent_id = t.id
)
SELECT * FROM tree ORDER BY path;
-- Output: one branch fully, then next branch
```

**SQL-Standard SEARCH Clause:**

```sql
WITH RECURSIVE walk(node) AS (
    VALUES ('A')
    UNION ALL
    SELECT e.dst FROM walk w JOIN edges e ON e.src = w.node
)
SEARCH DEPTH FIRST BY node SET traversal_order
SELECT node FROM walk ORDER BY traversal_order;
-- Expected output: A, B, D, C (DFS)
-- BREADTH FIRST would produce: A, B, C, D (BFS)
```

**Syntax Rules:**

- Default recursive CTE output is BFS .
- DFS requires ordering by a path column .
- `SEARCH DEPTH FIRST BY column SET order_col` explicitly requests DFS .
- `SEARCH BREADTH FIRST BY column SET order_col` explicitly requests BFS .

**Constraints and Limitations:**

- The `SEARCH` clause is SQL-standard but not universally supported (PostgreSQL supports it; MySQL does not).
- DFS ordering via path column may have performance overhead for deep hierarchies.
- BFS output from the CTE does not guarantee BFS ordering after outer query sorting.

### Annotated Complete Code Examples

**Example 1: BFS Output (Default)**

```sql
-- Tree: A → B, A → C, B → D
WITH RECURSIVE walk AS (
    SELECT 'A' AS node, 0 AS depth
    UNION ALL
    SELECT CASE 
        WHEN w.node = 'A' AND depth = 0 THEN 'B'
        WHEN w.node = 'A' AND depth = 0 THEN 'C'
        ELSE 'D'
    END, w.depth + 1
    FROM walk w
    WHERE w.depth < 2
)
SELECT node, depth FROM walk;

-- Expected Output (BFS):
--  node | depth
-- ------+-------
--  A    |     0
--  B    |     1
--  C    |     1
--  D    |     2
```

**Why this output occurs:** PostgreSQL's recursive evaluation naturally produces BFS order: all depth-1 nodes before depth-2 nodes .

**Example 2: DFS via Path Ordering**

```sql
-- Same tree, ordered by path for DFS
WITH RECURSIVE walk AS (
    SELECT node, ARRAY[node] AS path, 0 AS depth
    FROM (VALUES ('A')) AS t(node)
    UNION ALL
    SELECT e.dst, path || e.dst, w.depth + 1
    FROM walk w
    JOIN edges e ON e.src = w.node
)
SELECT node, path, depth
FROM walk
ORDER BY path;

-- Expected Output (DFS):
--  node | path      | depth
-- ------+-----------+-------
--  A    | {A}       |     0
--  B    | {A,B}     |     1
--  D    | {A,B,D}   |     2
--  C    | {A,C}     |     1
```

**Why this output occurs:** Ordering by the `path` array produces DFS order: A → B → D (one branch fully), then C (next branch) .

**Example 3: SQL-Standard SEARCH Clause**

```sql
-- BFS example using SEARCH BREADTH FIRST
WITH RECURSIVE walk(node) AS (
    VALUES ('A')
    UNION ALL
    SELECT e.dst FROM walk w JOIN edges e ON e.src = w.node
)
SEARCH BREADTH FIRST BY node SET traversal_order
SELECT node FROM walk ORDER BY traversal_order;
```

**Why this works:** The `SEARCH BREADTH FIRST BY node SET traversal_order` clause adds a traversal ordering column. Ordering by it produces BFS output .

### Real-World Cases

**Case 1: Menu Display (DFS)**

An e-commerce menu displays categories depth-first: "Electronics > Computers > Laptops" appears as a contiguous block before "Electronics > Phones." DFS ordering matches user expectations.

**Case 2: Level-by-Level Processing (BFS)**

A batch job processes hierarchy levels in order: first all root nodes, then all level-1 nodes, etc. BFS ordering enables level-based processing where each level depends on the previous.

### References

- PostgreSQL 17.7.1 Documentation (PDF) - http://repo.postgrespro.ru/doc/sdm/17.7.1/en/postgres-A4.pdf
- PostgreSQL 15 Documentation (PDF) - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/15/postgresql-15-A4.pdf
- Apache Calcite JIRA — Support the SQL-standard SEARCH Clause - https://issues.apache.org/jira/browse/CALCITE-7815

---

## Summary Table

| Concept | Key SQL Element | Primary Use Case |
|---------|----------------|------------------|
| **Anchor Member** | First `SELECT` before `UNION ALL` | Starting point (root, seed value) |
| **Recursive Member** | `SELECT` referencing CTE name | Iterative traversal (children, next value) |
| **Termination** | `WHERE` condition or natural empty set | Stopping recursion safely |
| **Depth Tracking** | `depth + 1` column | Level display, depth limits |
| **Path Construction** | Array or string accumulation | Breadcrumbs, cycle detection |
| **Cycle Detection** | `CYCLE` clause or path array | Preventing infinite loops |
| **BFS vs. DFS** | Default BFS; `SEARCH` clause or `ORDER BY path` for DFS | Ordering traversal results |

---

## References

- PostgreSQL 17.7.1 Documentation (PDF) - http://repo.postgrespro.ru/doc/sdm/17.7.1/en/postgres-A4.pdf
- PostgreSQL 15 Documentation (PDF) - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/15/postgresql-15-A4.pdf
- PostgreSQL 9.4 Documentation (PDF) - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.4/postgresql-9.4-A4.pdf
- MySQL 8.1 Reference Manual (PDF) - https://downloads.mysql.com/docs/refman-8.1-en.pdf
- MySQL 8.2 Reference Manual (PDF) - https://downloads.mysql.com/docs/refman-8.2-en.a4.pdf
- MariaDB Documentation (GitHub) - https://raw.githubusercontent.com/mariadb-corporation/mariadb-docs/refs/heads/main/server/reference/sql-statements/data-manipulation/selecting-data/common-table-expressions/with.md
- Apache Calcite JIRA — Support the SQL-standard SEARCH Clause - https://issues.apache.org/jira/browse/CALCITE-7815
- Sisense Blog — Solving the Traveling Salesman Problem with Postgres Recursive CTEs - https://www.sisense.com/blog/postgres-recursive-cte/
- Cybrosys — How to Write Hierarchical Queries in PostgreSQL - https://www.cybrosys.com/research-and-development/postgres/how-to-write-hierarchical-queries-in-postgresql
- GitHub — Recursive CTEs (Data Engineering Interview Patterns) - https://raw.githubusercontent.com/sergiorgiraldo/Python-lang/refs/heads/master/data-engineering-interview-patterns/sql/04_recursive_ctes/README.md
- Vertabelo Academy — Recursion Syntax - https://academy.vertabelo.com/course/postgresql-recursive-queries/recursive-cte/the-very-basics/recursion-syntax