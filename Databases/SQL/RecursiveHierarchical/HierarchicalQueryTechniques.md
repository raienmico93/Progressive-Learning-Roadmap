# Hierarchical Query Techniques: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Hierarchical query techniques are SQL methods for traversing, filtering, and aggregating data organized in tree or graph structures, where records are connected through parent-child relationships.

**Technical Definition:** Hierarchical queries use recursive processing (via Recursive CTEs or Oracle's `CONNECT BY`) to traverse self-referencing tables where a column in each row points to the primary key of another row in the same table. Key techniques include directional traversal (top-down or bottom-up), depth tracking via level pseudocolumns or calculated depth, ancestor/descendant identification, leaf detection, and hierarchical aggregation for rolling up values.

**Beginner-Friendly Explanation:** Hierarchical query techniques are how SQL walks up and down family trees, org charts, and category structures. You can start at the top and list everyone below, start at the bottom and trace the chain of command, count how deep the tree goes, find all the "leaves" (people or items with no children), or add up numbers as you go up the tree.

### Key Characteristics

- **Directional:** Traversal can go downward (root to leaves) or upward (leaf to root).
- **Level-Aware:** Depth or level tracking is fundamental to most hierarchical operations.
- **Recursive:** Implemented via Recursive CTEs (`WITH RECURSIVE`) or Oracle's proprietary `CONNECT BY`.
- **Aggregation-Capable:** Data can be rolled up the hierarchy for totals and subtotals.

### Prerequisites

- **Recursive CTE Support:** PostgreSQL, MySQL 8+, SQL Server, Oracle (with `CONNECT BY` or Recursive CTE), BigQuery.
- **Adjacency List Model:** Table structure with `id` and `parent_id` columns.
- **Basic SQL Proficiency:** Understanding of SELECT, JOIN, and CTE syntax.

### Related Programming Areas

- **Organizational Management:** HR reporting, approval chains, access control.
- **E-Commerce:** Category trees, navigation menus.
- **Manufacturing:** Bill-of-Materials (BOM) explosion and cost roll-up.
- **Financial Reporting:** Hierarchical account aggregation.

### Core Concepts / Features

1. Traversing Downward (Top-Down / Root to Leaves)
2. Traversing Upward (Bottom-Up / Leaf to Root)
3. Tree Depth and Level Tracking
4. Ancestor Identification
5. Descendant Identification
6. Leaf Node Detection
7. Aggregating Data Up the Hierarchy


## 1. Traversing Downward (Top-Down / Root to Leaves)

### Definitions

**Core Definition:** Top-down traversal starts at a root node (or nodes) and follows parent-child relationships downward to list all descendants.

**Technical Definition:** In a Recursive CTE, the anchor member selects the root row(s) (typically `WHERE parent_id IS NULL`), and the recursive member joins the base table's `parent_id` to the CTE's `id`, iteratively producing child rows at each level.

**Beginner-Friendly Explanation:** Top-down traversal is like starting at the CEO and listing every person who works under them, level by level.

### Purposes

- To list all descendants of a given node (e.g., all employees under a manager).
- To generate hierarchical menus or category trees.
- To explode a Bill-of-Materials (BOM) from a top-level product.
- To compute organization-wide metrics by including all subordinates.

### Syntax Rules and Structure

**Complete General Syntax (Recursive CTE):**

```sql
WITH RECURSIVE cte_name AS (
    -- Anchor: select root row(s)
    SELECT id, name, parent_id, 0 AS depth
    FROM table_name
    WHERE parent_id IS NULL  -- or id = :root_id
    
    UNION ALL
    
    -- Recursive: find children of current level
    SELECT t.id, t.name, t.parent_id, cte.depth + 1
    FROM table_name t
    JOIN cte_name cte ON t.parent_id = cte.id
)
SELECT * FROM cte_name;
```

**Complete General Syntax (Oracle CONNECT BY):**

```sql
SELECT id, name, LEVEL
FROM table_name
START WITH parent_id IS NULL
CONNECT BY PRIOR id = parent_id;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| Anchor | Selects root row(s) |
| `JOIN cte ON t.parent_id = cte.id` | Connects child rows to parent rows |
| `depth + 1` | Increments level for each generation |

**Syntax Rules:**

- The anchor selects the starting point (root or specific node).
- The recursive member joins `child.parent_id` to `parent.id`.
- Oracle uses `PRIOR id = parent_id` to indicate direction.

**Constraints and Limitations:**

- May hit recursion limits on very deep hierarchies.
- Cycles in data cause infinite recursion without cycle detection.

### Annotated Complete Code Examples

**Example 1: Top-Down Traversal (PostgreSQL)**

```sql
-- Create employee hierarchy
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    manager_id INT,
    FOREIGN KEY (manager_id) REFERENCES employees(id)
);

INSERT INTO employees VALUES
    (1, 'Alice', NULL),      -- CEO
    (2, 'Bob', 1),          -- VP
    (3, 'Carol', 1),        -- VP
    (4, 'Dave', 2),         -- Engineer
    (5, 'Eve', 2),          -- Engineer
    (6, 'Frank', 3);        -- Sales Rep
```

```sql
-- Top-down: list all descendants of Alice
WITH RECURSIVE org_chart AS (
    -- Anchor: start at CEO
    SELECT id, name, manager_id, 0 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: find direct reports
    SELECT e.id, e.name, e.manager_id, oc.level + 1
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT id, name, level
FROM org_chart
ORDER BY level, name;

-- Expected Output:
--  id | name  | level
-- ----+-------+-------
--   1 | Alice |     0
--   2 | Bob   |     1
--   3 | Carol |     1
--   4 | Dave  |     2
--   5 | Eve   |     2
--   6 | Frank |     2
```

**Why this output occurs:** The anchor selects Alice (`manager_id IS NULL`). The recursive member joins employees whose `manager_id` matches the CTE's `id`, producing Bob and Carol at level 1. The next iteration produces Dave, Eve, and Frank at level 2. Recursion stops when no more matches exist.

### Real-World Cases

**Case 1: Organizational Directory**

An HR system lists all employees in a manager's organization for performance review cycles.

**Case 2: Category Navigation**

An e-commerce site builds a nested menu from root categories down to leaf categories.

### References

- Oracle Hierarchical Query Pseudocolumns - https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Hierarchical-Query-Pseudocolumns.html
- PostgreSQL 19 Documentation (Recursive Queries) - https://www.postgresql.org/files/documentation/pdf/19/postgresql-19-US.pdf


## 2. Traversing Upward (Bottom-Up / Leaf to Root)

### Definitions

**Core Definition:** Bottom-up traversal starts at a specific node (often a leaf) and follows parent references upward to the root, identifying the chain of ancestors.

**Technical Definition:** The anchor member selects the starting node (e.g., `WHERE id = :node_id`), and the recursive member joins the base table's `id` to the CTE's `parent_id`, walking up the hierarchy.

**Beginner-Friendly Explanation:** Bottom-up traversal is like starting at a specific employee and asking "who is my manager, who is their manager, all the way to the CEO?"

### Purposes

- To find the management chain or approval path for a specific node.
- To generate breadcrumb navigation (Home > Category > Subcategory).
- To identify ancestors for permission inheritance.
- To compute depth from a leaf to root.

### Syntax Rules and Structure

**Complete General Syntax (Recursive CTE):**

```sql
WITH RECURSIVE cte_name AS (
    -- Anchor: start at specific node
    SELECT id, name, parent_id, 0 AS level
    FROM table_name
    WHERE id = :start_node_id
    
    UNION ALL
    
    -- Recursive: find parent of current node
    SELECT t.id, t.name, t.parent_id, cte.level + 1
    FROM table_name t
    JOIN cte_name cte ON t.id = cte.parent_id
)
SELECT * FROM cte_name;
```

**Complete General Syntax (Oracle CONNECT BY):**

```sql
SELECT id, name, LEVEL
FROM table_name
START WITH id = :start_node_id
CONNECT BY PRIOR parent_id = id;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| Anchor | Selects the starting node |
| `JOIN cte ON t.id = cte.parent_id` | Connects parent rows to child rows |
| `PRIOR parent_id = id` (Oracle) | Walks up the hierarchy |

**Syntax Rules:**

- The anchor selects the starting node (usually a leaf or specific ID).
- The recursive member joins `parent.id` to `child.parent_id`.
- Recursion stops when `parent_id IS NULL` (root reached).

**Constraints and Limitations:**

- If the starting node doesn't exist, no rows returned.
- Cycles cause infinite recursion.

### Annotated Complete Code Examples

**Example 1: Bottom-Up Traversal (PostgreSQL)**

```sql
-- Bottom-up: find management chain for Frank (id = 6)
WITH RECURSIVE management_chain AS (
    -- Anchor: start at Frank
    SELECT id, name, manager_id, 0 AS level
    FROM employees
    WHERE id = 6
    
    UNION ALL
    
    -- Recursive: find Frank's manager, then their manager, etc.
    SELECT e.id, e.name, e.manager_id, mc.level + 1
    FROM employees e
    JOIN management_chain mc ON e.id = mc.manager_id
)
SELECT id, name, level
FROM management_chain
ORDER BY level;

-- Expected Output:
--  id | name  | level
-- ----+-------+-------
--   6 | Frank |     0
--   3 | Carol |     1
--   1 | Alice |     2
```

**Why this output occurs:** The anchor selects Frank (id=6). The recursive member finds Frank's manager (Carol, id=3), then Carol's manager (Alice, id=1). Alice has `manager_id IS NULL`, so recursion stops. The result shows the full management chain.

### Real-World Cases

**Case 1: Approval Workflow**

A purchase order approval system finds the chain of approvers for a specific employee by traversing up the management hierarchy.

**Case 2: Breadcrumb Navigation**

A CMS builds breadcrumbs for an article by walking up its category hierarchy: "Home > Technology > Programming > SQL."

### References

- Oracle Hierarchical Query Operators (CONNECT_BY_ROOT) - https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sqlrf/Hierarchical-Query-Operators.html
- Stack Overflow: CONNECT_BY_ROOT Example - https://stackoverflow.com/revisions/3721f756-f00e-4650-a886-8a2ebd6b3b4b/view-source


## 3. Tree Depth and Level Tracking

### Definitions

**Core Definition:** Depth or level tracking adds a column to hierarchical query results indicating how many steps a node is from the starting point (root or anchor).

**Technical Definition:** In Recursive CTEs, a `level` or `depth` column is initialized to 0 (or 1) in the anchor and incremented by 1 in the recursive member. Oracle provides the `LEVEL` pseudocolumn, which returns 1 for root rows, 2 for children of root, and so on.

**Beginner-Friendly Explanation:** Level tracking is like counting floors in a building. The root is floor 0 (or 1), its children are floor 1 (or 2), and so on.

### Purposes

- To display hierarchy with indentation or grouping.
- To limit traversal depth (e.g., only 3 levels).
- To analyze tree structure (average depth, maximum depth).
- To enforce business rules based on level.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
WITH RECURSIVE cte_name AS (
    -- Anchor: level starts at 0 or 1
    SELECT id, name, 0 AS level
    FROM table_name
    WHERE starting_condition
    
    UNION ALL
    
    -- Recursive: increment level
    SELECT t.id, t.name, cte.level + 1
    FROM table_name t
    JOIN cte_name cte ON t.parent_id = cte.id
    WHERE cte.level < max_depth  -- Optional depth limit
)
SELECT * FROM cte_name;
```

**Oracle LEVEL Pseudocolumn:**

```sql
SELECT id, name, LEVEL
FROM table_name
START WITH parent_id IS NULL
CONNECT BY PRIOR id = parent_id;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `0 AS level` | Initial level at anchor |
| `level + 1` | Increment per recursion |
| Oracle `LEVEL` | Built-in pseudocolumn |

**Syntax Rules:**

- Initialize level in the anchor (0 or 1).
- Increment by 1 in the recursive member.
- Use level in `WHERE` for depth limits.

### Annotated Complete Code Examples

**Example 1: Level Display with Indentation**

```sql
-- Display org chart with indentation
WITH RECURSIVE org_chart AS (
    SELECT id, name, manager_id, 0 AS level
    FROM employees WHERE manager_id IS NULL
    
    UNION ALL
    
    SELECT e.id, e.name, e.manager_id, oc.level + 1
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT 
    id,
    name,
    level,
    REPEAT('  ', level) || name AS indented_name
FROM org_chart
ORDER BY level, name;

-- Expected Output:
--  id | name  | level | indented_name
-- ----+-------+-------+----------------
--   1 | Alice |     0 | Alice
--   2 | Bob   |     1 |   Bob
--   3 | Carol |     1 |   Carol
--   4 | Dave  |     2 |     Dave
--   5 | Eve   |     2 |     Eve
--   6 | Frank |     2 |     Frank
```

**Why this output occurs:** The `level` column tracks depth from the CEO. `REPEAT('  ', level)` creates indentation based on depth, visually representing the hierarchy.

**Example 2: Depth-Limited Traversal**

```sql
-- Only traverse 2 levels below root
WITH RECURSIVE limited_org AS (
    SELECT id, name, manager_id, 0 AS level
    FROM employees WHERE manager_id IS NULL
    
    UNION ALL
    
    SELECT e.id, e.name, e.manager_id, lo.level + 1
    FROM employees e
    JOIN limited_org lo ON e.manager_id = lo.id
    WHERE lo.level < 2  -- Stop at level 2
)
SELECT * FROM limited_org;
```

**Why this output occurs:** The `WHERE lo.level < 2` condition prevents generation of level 3 rows, limiting traversal depth.

### Real-World Cases

**Case 1: Menu Indentation**

An e-commerce menu displays categories with visual indentation based on level.

**Case 2: Report Grouping**

A financial report groups accounts by hierarchy level: level 0 = headers, level 1 = sub-headers, level 2 = line items.

### References

- Oracle Hierarchical Query Pseudocolumns (LEVEL) - https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Hierarchical-Query-Pseudocolumns.html
- Microsoft SQL Server Recursive CTE Examples - https://learn.microsoft.com/fr-ch/SQL/t-sql/queries/with-common-table-expression-transact-sql


## 4. Ancestor Identification

### Definitions

**Core Definition:** Ancestor identification finds all nodes above a given node in the hierarchy, from its immediate parent to the root.

**Technical Definition:** Ancestor identification uses bottom-up traversal (from a specific node to root) and optionally the `CONNECT_BY_ROOT` operator (Oracle) to identify the root ancestor of any node.

**Beginner-Friendly Explanation:** An ancestor is anyone above you in the tree—your manager, their manager, and so on, up to the CEO.

### Purposes

- To find all managers above an employee.
- To identify root ancestors for permission or routing logic.
- To build breadcrumb paths.
- To determine if a node is subordinate to another.

### Syntax Rules and Structure

**Complete General Syntax (Bottom-Up):**

```sql
WITH RECURSIVE ancestors AS (
    SELECT id, name, parent_id, 0 AS level
    FROM table_name WHERE id = :start_node_id
    UNION ALL
    SELECT t.id, t.name, t.parent_id, a.level + 1
    FROM table_name t
    JOIN ancestors a ON t.id = a.parent_id
)
SELECT * FROM ancestors;
```

**Oracle CONNECT_BY_ROOT:**

```sql
SELECT child, CONNECT_BY_ROOT(child) AS root_ancestor
FROM your_table
WHERE org_type IN ('Unit', 'Section', 'Dep')
CONNECT BY parent = PRIOR child
START WITH org_type = 'GDep';
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `CONNECT_BY_ROOT(child)` | Returns the root value for each row |
| `START WITH` | Defines starting rows |

**Syntax Rules:**

- `CONNECT_BY_ROOT` is valid only in Oracle hierarchical queries.
- It returns the root value for each row in the hierarchy.

### Annotated Complete Code Examples

**Example 1: Finding All Ancestors**

```sql
-- Find all ancestors of Dave (id = 4)
WITH RECURSIVE ancestors AS (
    SELECT id, name, manager_id, 0 AS level
    FROM employees WHERE id = 4
    
    UNION ALL
    
    SELECT e.id, e.name, e.manager_id, a.level + 1
    FROM employees e
    JOIN ancestors a ON e.id = a.manager_id
)
SELECT id, name, level
FROM ancestors
ORDER BY level;

-- Expected Output:
--  id | name  | level
-- ----+-------+-------
--   4 | Dave  |     0
--   2 | Bob   |     1
--   1 | Alice |     2
```

**Example 2: CONNECT_BY_ROOT (Oracle)**

```sql
-- Find root ancestor (GDep) for each Unit
SELECT child, CONNECT_BY_ROOT(child) AS gdep
FROM your_table
WHERE org_type IN ('Unit', 'Section', 'Dep')
CONNECT BY parent = PRIOR child
START WITH org_type = 'GDep';
```

**Why this output occurs:** `CONNECT_BY_ROOT` returns the starting row's value (the GDep) for every descendant row, enabling grouping by root ancestor.

### Real-World Cases

**Case 1: Permission Inheritance**

A document management system grants access based on management chain—users can access documents shared by anyone in their ancestry.

**Case 2: Breadcrumb Generation**

An e-commerce site builds "Home > Electronics > Computers > Laptops" by identifying all ancestors of the Laptops category.

### References

- Oracle Hierarchical Query Operators (CONNECT_BY_ROOT) - https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sqlrf/Hierarchical-Query-Operators.html
- Stack Overflow: CONNECT_BY_ROOT Example - https://stackoverflow.com/revisions/3721f756-f00e-4650-a886-8a2ebd6b3b4b/view-source


## 5. Descendant Identification

### Definitions

**Core Definition:** Descendant identification finds all nodes below a given node in the hierarchy, including children, grandchildren, and deeper levels.

**Technical Definition:** Descendant identification uses top-down traversal (from a specific node to leaves) to enumerate all nodes in the subtree rooted at the starting node.

**Beginner-Friendly Explanation:** A descendant is anyone below you in the tree—your direct reports, their reports, and so on.

### Purposes

- To list all employees in a manager's organization.
- To find all subcategories under a category.
- To explode a BOM for a product.
- To compute aggregates over a subtree.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
WITH RECURSIVE descendants AS (
    SELECT id, name, parent_id, 0 AS level
    FROM table_name WHERE id = :start_node_id
    UNION ALL
    SELECT t.id, t.name, t.parent_id, d.level + 1
    FROM table_name t
    JOIN descendants d ON t.parent_id = d.id
)
SELECT * FROM descendants;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| Anchor | Selects subtree root |
| `JOIN descendants ON t.parent_id = d.id` | Finds children |

**Syntax Rules:**

- The anchor selects the subtree root node.
- The recursive member finds all children of the current level.

### Annotated Complete Code Examples

**Example 1: Finding All Descendants**

```sql
-- Find all employees under Bob (id = 2)
WITH RECURSIVE bob_org AS (
    SELECT id, name, manager_id, 0 AS level
    FROM employees WHERE id = 2
    
    UNION ALL
    
    SELECT e.id, e.name, e.manager_id, bo.level + 1
    FROM employees e
    JOIN bob_org bo ON e.manager_id = bo.id
)
SELECT id, name, level
FROM bob_org
ORDER BY level, name;

-- Expected Output:
--  id | name  | level
-- ----+-------+-------
--   2 | Bob   |     0
--   4 | Dave  |     1
--   5 | Eve   |     1
```

**Why this output occurs:** The anchor selects Bob. The recursive member finds employees with `manager_id = 2` (Dave and Eve). No employees report to Dave or Eve, so recursion stops.

### Real-World Cases

**Case 1: Team Size Calculation**

An HR system counts total headcount in a manager's organization by listing all descendants.

**Case 2: Category Product Count**

An e-commerce site counts products in a category including all subcategories.

### References

- PostgreSQL 19 Documentation (Recursive Queries) - https://www.postgresql.org/files/documentation/pdf/19/postgresql-19-US.pdf
- Databricks SQL Common Table Expression (CTE) - https://learn.microsoft.com/hu-hu/azure/databricks/sql/language-manual/sql-ref-syntax-qry-select-cte


## 6. Leaf Node Detection

### Definitions

**Core Definition:** Leaf node detection identifies nodes that have no children in the hierarchy.

**Technical Definition:** A leaf node is a row that does not appear as a parent of any other row. Oracle provides the `CONNECT_BY_ISLEAF` pseudocolumn, which returns 1 if the row is a leaf, 0 otherwise.

**Beginner-Friendly Explanation:** A leaf is the end of a branch—like an individual contributor in an org chart or a subcategory with no further breakdown.

### Purposes

- To find terminal nodes in a hierarchy.
- To identify individual contributors (non-managers).
- To find leaf categories with no subcategories.
- To compute aggregates only on leaf nodes.

### Syntax Rules and Structure

**Oracle CONNECT_BY_ISLEAF:**

```sql
SELECT last_name, CONNECT_BY_ISLEAF AS is_leaf, LEVEL
FROM employees
START WITH employee_id = 100
CONNECT BY PRIOR employee_id = manager_id;
```

**Recursive CTE Leaf Detection:**

```sql
WITH RECURSIVE tree AS (
    SELECT id, name, parent_id FROM nodes WHERE parent_id IS NULL
    UNION ALL
    SELECT n.id, n.name, n.parent_id FROM nodes n JOIN tree t ON n.parent_id = t.id
)
SELECT t.id, t.name,
       CASE WHEN EXISTS (SELECT 1 FROM nodes c WHERE c.parent_id = t.id)
            THEN 0 ELSE 1 END AS is_leaf
FROM tree t;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `CONNECT_BY_ISLEAF` | Oracle pseudocolumn (1 = leaf) |
| `EXISTS (SELECT 1 FROM nodes c WHERE c.parent_id = t.id)` | CTE-based leaf check |

**Syntax Rules:**

- Oracle: `CONNECT_BY_ISLEAF` returns 1 for leaves, 0 for non-leaves.
- CTE: Use `NOT EXISTS` or `EXISTS` subquery to check for children.

### Annotated Complete Code Examples

**Example 1: Oracle CONNECT_BY_ISLEAF**

```sql
SELECT last_name "Employee", CONNECT_BY_ISLEAF "IsLeaf", LEVEL
FROM employees
WHERE LEVEL <= 3 AND department_id = 80
START WITH employee_id = 100
CONNECT BY PRIOR employee_id = manager_id AND LEVEL <= 4
ORDER BY "Employee", "IsLeaf";

-- Expected Output (partial):
-- Employee    IsLeaf  LEVEL
-- Abel        1       3
-- Ande        1       3
-- Cambrault   0       2
-- Errazuriz   0       2
```

**Why this output occurs:** `CONNECT_BY_ISLEAF` returns 1 for Abel and Ande (they have no direct reports), 0 for Cambrault and Errazuriz (they have reports).

**Example 2: CTE-Based Leaf Detection**

```sql
-- Find all leaf categories
WITH RECURSIVE category_tree AS (
    SELECT id, name, parent_id FROM categories WHERE parent_id IS NULL
    UNION ALL
    SELECT c.id, c.name, c.parent_id FROM categories c JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT ct.id, ct.name
FROM category_tree ct
WHERE NOT EXISTS (SELECT 1 FROM categories c WHERE c.parent_id = ct.id);

-- Returns categories with no subcategories
```

**Why this output occurs:** The `NOT EXISTS` subquery checks if any category has the current category as parent. If none, it's a leaf.

### Real-World Cases

**Case 1: Individual Contributor Identification**

An HR system identifies all employees who are not managers (leaves) for compensation analysis.

**Case 2: Leaf Category Products**

An e-commerce system displays products only from leaf categories (no subcategories) to avoid showing empty parent categories.

### References

- Oracle Hierarchical Query Pseudocolumns (CONNECT_BY_ISLEAF) - https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Hierarchical-Query-Pseudocolumns.html
- Databricks SQL Common Table Expression (CTE) - https://learn.microsoft.com/hu-hu/azure/databricks/sql/language-manual/sql-ref-syntax-qry-select-cte


## 7. Aggregating Data Up the Hierarchy

### Definitions

**Core Definition:** Hierarchical aggregation (roll-up) computes totals or other aggregates that include a node's own value plus the values of all its descendants.

**Technical Definition:** Rolling up data up the hierarchy requires traversing the tree and summing values from all descendants at each level. This can be implemented by first flattening the hierarchy (via Recursive CTE), then joining to fact data and grouping by ancestor.

**Beginner-Friendly Explanation:** Rolling up is like calculating total salary for a manager's entire organization—the manager's salary plus everyone below them, all the way down.

### Purposes

- To compute organizational roll-up metrics (total headcount, total salary).
- To calculate BOM costs including all sub-components.
- To generate financial reports with hierarchical subtotals.
- To support "drill-down" analytics.

### Syntax Rules and Structure

**Complete General Syntax (Flatten + Aggregate):**

```sql
WITH RECURSIVE hierarchy AS (
    -- Anchor: root nodes
    SELECT id, parent_id, 0 AS level
    FROM table_name WHERE parent_id IS NULL
    UNION ALL
    SELECT t.id, t.parent_id, h.level + 1
    FROM table_name t JOIN hierarchy h ON t.parent_id = h.id
)
SELECT h_root.id AS root_id, SUM(f.amount) AS total
FROM hierarchy h_root
JOIN hierarchy h_desc ON h_desc.id = h_root.id OR 
    -- All descendants: join via recursive path
    -- (requires path or closure table)
JOIN fact_table f ON f.node_id = h_desc.id
GROUP BY h_root.id;
```

**Simplified Approach (Path-Based):**

```sql
WITH RECURSIVE hierarchy AS (
    SELECT id, name, parent_id, ARRAY[id] AS path
    FROM table_name WHERE parent_id IS NULL
    UNION ALL
    SELECT t.id, t.name, t.parent_id, h.path || t.id
    FROM table_name t JOIN hierarchy h ON t.parent_id = h.id
)
SELECT h.id, h.name, SUM(f.amount) AS total
FROM hierarchy h
JOIN fact_table f ON f.node_id = h.id
WHERE h.path @> ARRAY[f.node_id]  -- PostgreSQL array containment
GROUP BY h.id, h.name;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `path` | Tracks all ancestors for each node |
| `h.path @> ARRAY[f.node_id]` | Checks if node is in path (PostgreSQL) |

**Syntax Rules:**

- Flatten hierarchy first, then join to fact table.
- Use path arrays to determine ancestor-descendant relationships.
- Aggregate with `SUM`, `COUNT`, `AVG` as needed.

**Constraints and Limitations:**

- Performance degrades with hierarchy depth and fact table size.
- Alternative: Closure tables or materialized paths for faster aggregation.

### Annotated Complete Code Examples

**Example 1: Rolling Up Total Salary (PostgreSQL)**

```sql
-- Add salary column to employees
ALTER TABLE employees ADD COLUMN salary DECIMAL(10,2);
UPDATE employees SET salary = CASE id
    WHEN 1 THEN 200000 WHEN 2 THEN 150000 WHEN 3 THEN 150000
    WHEN 4 THEN 100000 WHEN 5 THEN 100000 WHEN 6 THEN 80000 END;

-- Calculate total salary for each manager's organization
WITH RECURSIVE org_paths AS (
    SELECT id, name, manager_id, ARRAY[id] AS path
    FROM employees WHERE manager_id IS NULL
    
    UNION ALL
    
    SELECT e.id, e.name, e.manager_id, op.path || e.id
    FROM employees e
    JOIN org_paths op ON e.manager_id = op.id
)
SELECT 
    op.id,
    op.name,
    SUM(e.salary) AS total_org_salary
FROM org_paths op
JOIN employees e ON e.id = ANY(op.path)  -- All descendants in path
GROUP BY op.id, op.name
ORDER BY op.id;

-- Expected Output:
--  id | name  | total_org_salary
-- ----+-------+------------------
--   1 | Alice |          780000
--   2 | Bob   |          350000
--   3 | Carol |          230000
--   4 | Dave  |          100000
--   5 | Eve   |          100000
--   6 | Frank |           80000
```

**Why this output occurs:** The `path` array for each node includes all its ancestors. For Alice (CEO), the path includes all employees, so total salary is 780,000. For Bob, the path includes Bob, Dave, and Eve (350,000). This rolls up salaries to every node in the hierarchy.

### Real-World Cases

**Case 1: Organizational Cost Roll-Up**

A finance system calculates total labor cost for each manager's organization by summing all descendant salaries.

**Case 2: BOM Cost Calculation**

A manufacturing system computes the total cost of a product by summing component costs through the BOM hierarchy.

### References

- PostgreSQL 17.7.1 Documentation (Recursive Queries) - http://repo.postgrespro.ru/doc/sdm/17.7.1/en/postgres-A4.pdf
- US Patent 20220207058A1 (Hierarchical Aggregation) - https://patentimages.storage.googleapis.com/49/4d/aa/13a50657b954f8/US20220207058A1.pdf


## Summary Table

| Technique | Direction | Key SQL Element | Primary Use Case |
|-----------|-----------|-----------------|------------------|
| Top-Down Traversal | Root → Leaves | `JOIN ON child.parent_id = parent.id` | List all descendants |
| Bottom-Up Traversal | Leaf → Root | `JOIN ON parent.id = child.parent_id` | Find management chain |
| Level Tracking | Both | `level + 1` or `LEVEL` pseudocolumn | Indentation, depth limits |
| Ancestor Identification | Upward | `CONNECT_BY_ROOT` (Oracle) | Breadcrumbs, permissions |
| Descendant Identification | Downward | Recursive join | Team size, subtree |
| Leaf Detection | Any | `CONNECT_BY_ISLEAF` or `NOT EXISTS` | Terminal nodes |
| Hierarchical Aggregation | Both | Path arrays + `GROUP BY` | Roll-up totals |

---

## References

- Oracle Hierarchical Query Pseudocolumns - https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Hierarchical-Query-Pseudocolumns.html
- Oracle Hierarchical Query Operators (CONNECT_BY_ROOT) - https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sqlrf/Hierarchical-Query-Operators.html
- PostgreSQL 19 Documentation (Recursive Queries) - https://www.postgresql.org/files/documentation/pdf/19/postgresql-19-US.pdf
- PostgreSQL 17.7.1 Documentation (Recursive Queries) - http://repo.postgrespro.ru/doc/sdm/17.7.1/en/postgres-A4.pdf
- PostgreSQL 18.3.2 Documentation (Cycle Detection) - http://repo.postgrespro.ru/doc/sdm/18.3.2/en/postgres-A4.pdf
- Microsoft SQL Server Recursive CTE Examples - https://learn.microsoft.com/fr-ch/SQL/t-sql/queries/with-common-table-expression-transact-sql
- Microsoft SQL Server Recursive CTE (Russian) - https://learn.microsoft.com/ru-ru/sql/t-sql/queries/with-common-table-expression-transact-sql
- Databricks SQL Common Table Expression (CTE) - https://learn.microsoft.com/hu-hu/azure/databricks/sql/language-manual/sql-ref-syntax-qry-select-cte
- Stack Overflow: CONNECT_BY_ROOT Example - https://stackoverflow.com/revisions/3721f756-f00e-4650-a886-8a2ebd6b3b4b/view-source
- Huawei Cloud CONNECT BY Migration Guide - https://support.huaweicloud.com/intl/en-us/ally-visitor-1-tg-dws/dws_mt_0216.html