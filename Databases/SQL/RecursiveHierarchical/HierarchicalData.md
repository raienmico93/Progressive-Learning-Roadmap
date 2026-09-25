# SQL Hierarchical Data: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** SQL hierarchical data refers to data organized in tree-like or graph-like structures where records are connected through parent-child relationships, enabling representation of organizational structures, category trees, folder systems, bill-of-materials, and network topologies.

**Technical Definition:** Hierarchical data in relational databases is modeled using self-referencing relationships, where a record contains a foreign key reference to another record in the same table (its parent). This creates a directed acyclic graph (DAG) or tree structure. SQL provides specialized techniques—Recursive Common Table Expressions (CTEs), materialized paths, nested sets, and closure tables—to query and traverse these structures efficiently .

**Beginner-Friendly Explanation:** Hierarchical data is like a family tree. Each person has a parent, and that parent has their own parent, all the way up to the root. SQL databases don't naturally store trees, so we use special techniques—like recursive queries—to walk up and down the tree, finding all descendants of a manager or all categories under "Electronics."

### Key Characteristics

- **Self-Referencing:** Hierarchical tables contain a column (e.g., `parent_id`, `manager_id`) that points to another row in the same table .
- **Root Node:** The topmost node has no parent (NULL parent reference) .
- **Arbitrary Depth:** Hierarchies can be any number of levels deep; SQL must handle variable depth .
- **Tree vs. Graph:** Trees have one parent per node (simple hierarchies); graphs allow multiple connections (social networks, network topologies) .

### Prerequisites

- **SQL Proficiency:** Understanding of SELECT, JOIN, and basic CTE syntax.
- **Recursive CTE Support:** PostgreSQL, MySQL 8+, SQL Server, BigQuery, and Oracle all support recursive CTEs .
- **Hierarchy Modeling Knowledge:** Understanding of Adjacency List, Materialized Path, or Closure Table approaches .

### Related Programming Areas

- **Organizational Management:** Employee reporting structures, management chains.
- **E-Commerce:** Product category trees, navigation menus.
- **File Systems:** Folder and directory hierarchies.
- **Manufacturing:** Bill-of-Materials (BOM) for assembly components.
- **Graph Analytics:** Social networks, network topologies, recommendation systems .

### Core Concepts / Features

1. Parent-Child Relationships
2. Organizational Structures
3. Category Trees
4. Folder Structures
5. Bill-of-Materials (BOM) Structures
6. Graph Structures

---

## 1. Parent-Child Relationships

### Definitions

**Core Definition:** A parent-child relationship is a one-to-many association where one record (the parent) is linked to zero or more records (the children) within the same table.

**Technical Definition:** In relational databases, a parent-child relationship is implemented via a self-referencing foreign key. The child table has a column (e.g., `parent_id`) that references the primary key of the parent row in the same table. This creates a directed edge in the hierarchy graph .

**Beginner-Friendly Explanation:** A parent-child relationship is like a folder containing files. The folder (parent) can contain multiple files (children), and each file knows which folder it belongs to. But in hierarchical data, a child can also be a parent to its own children, creating nested levels.

### Purposes

- To model one-to-many relationships within a single entity type.
- To enable recursive traversal up (ancestors) or down (descendants) the hierarchy.
- To support tree-structured data without requiring separate tables for each level.
- To represent natural hierarchies like folders, categories, or management chains.

### Syntax Rules and Structure

**Complete General Syntax (Table Definition):**

```sql
CREATE TABLE table_name (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    parent_id INT,
    FOREIGN KEY (parent_id) REFERENCES table_name(id)
);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `id` | Primary key, uniquely identifies each node |
| `parent_id` | Self-referencing foreign key to parent node |
| `FOREIGN KEY` | Enforces referential integrity (parent must exist) |

**Complete General Syntax (Recursive CTE Traversal):**

```sql
WITH RECURSIVE cte_name AS (
    -- Anchor: starting rows (e.g., root or specific node)
    SELECT id, name, parent_id, 0 AS depth
    FROM table_name
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- Recursive: join back to CTE to get next level
    SELECT t.id, t.name, t.parent_id, cte.depth + 1
    FROM table_name t
    INNER JOIN cte_name cte ON t.parent_id = cte.id
)
SELECT * FROM cte_name;
```

**Syntax Rules:**

- The anchor member selects the starting rows (root or specific node) .
- The recursive member references the CTE itself, joining child rows to the CTE's current level .
- `UNION ALL` is required (not `UNION`) to preserve all rows .
- A depth or level column tracks how deep the traversal has gone .

**Constraints and Limitations:**

- Recursive CTEs have a maximum recursion depth (varies by database; default often 100 in SQL Server) .
- Cycles in the data (A → B → A) cause infinite recursion; cycle detection is the developer's responsibility.
- Deep hierarchies may cause performance issues; materialized paths or closure tables are alternatives .

### Annotated Complete Code Examples

**Example 1: Employee Hierarchy Table and Top-Down Traversal**

```sql
-- Create employee hierarchy table
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    manager_id INT,
    FOREIGN KEY (manager_id) REFERENCES employees(id)
);

-- Insert sample data (CEO has NULL manager)
INSERT INTO employees VALUES
    (1, 'Alice', NULL),      -- CEO (root)
    (2, 'Bob', 1),          -- VP Engineering
    (3, 'Carol', 1),        -- VP Sales
    (4, 'Dave', 2),         -- Engineer (reports to Bob)
    (5, 'Eve', 2),          -- Engineer (reports to Bob)
    (6, 'Frank', 3),        -- Sales Rep (reports to Carol)
    (7, 'Grace', 3);        -- Sales Rep (reports to Carol)

-- Top-down traversal: find all employees and their level
WITH RECURSIVE org_chart AS (
    -- Anchor: start with the CEO (no manager)
    SELECT id, name, manager_id, 0 AS level, 
           CAST(name AS CHAR(1000)) AS path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: find direct reports of current level
    SELECT e.id, e.name, e.manager_id, oc.level + 1,
           CONCAT(oc.path, ' > ', e.name)
    FROM employees e
    INNER JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT id, name, level, path
FROM org_chart
ORDER BY path;

-- Expected Output:
--  id | name  | level | path
-- ----+-------+-------+-------------------------
--   1 | Alice |     0 | Alice
--   2 | Bob   |     1 | Alice > Bob
--   4 | Dave  |     2 | Alice > Bob > Dave
--   5 | Eve   |     2 | Alice > Bob > Eve
--   3 | Carol |     1 | Alice > Carol
--   6 | Frank |     2 | Alice > Carol > Frank
--   7 | Grace |     2 | Alice > Carol > Grace
```

**Why this output occurs:** The anchor member selects Alice (the CEO) because `manager_id IS NULL`. The recursive member joins employees whose `manager_id` matches the CTE's `id`, finding direct reports. The `level` column increments with each recursion, and `path` builds a breadcrumb trail. The recursion stops when no more employees match .

**Example 2: Bottom-Up Traversal (Finding Ancestors)**

```sql
-- Bottom-up: find all managers above a specific employee
WITH RECURSIVE ancestors AS (
    -- Anchor: start with the target employee
    SELECT id, name, manager_id, 0 AS level
    FROM employees
    WHERE id = 5  -- Starting from Eve
    
    UNION ALL
    
    -- Recursive: find the manager of each row
    SELECT e.id, e.name, e.manager_id, a.level + 1
    FROM employees e
    INNER JOIN ancestors a ON e.id = a.manager_id
)
SELECT id, name, level
FROM ancestors
ORDER BY level;

-- Expected Output:
--  id | name  | level
-- ----+-------+-------
--   5 | Eve   |     0
--   2 | Bob   |     1
--   1 | Alice |     2
```

**Why this output occurs:** The anchor selects Eve (`id = 5`). The recursive member finds employees whose `id` matches the CTE's `manager_id`. Eve's manager is Bob (id=2), Bob's manager is Alice (id=1), and Alice has no manager (`manager_id IS NULL`), so recursion stops. This walks up the tree from a leaf to the root .

### Real-World Cases

**Case 1: Employee Reporting Structure**

An HR system displays the management chain for any employee. A bottom-up recursive CTE finds all managers above a given employee for approval workflows or permission inheritance.

**Case 2: Comment Threads**

A forum stores comments with a `parent_comment_id`. A top-down recursive CTE retrieves a full thread with proper indentation and nesting depth.

### References

- How to Write Recursive CTEs in BigQuery for Hierarchical Data Traversal - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-02-17-how-to-write-recursive-ctes-in-bigquery-for-hierarchical-data-traversal/README.md 
- How to Use Recursive CTEs in MySQL 8 - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-how-to-use-recursive-ctes-in-mysql-8/README.md 
- SQL Server Hierarchical Data - https://learn.microsoft.com/ja-jp/SQL/relational-databases/hierarchical-data-sql-server 

---

## 2. Organizational Structures

### Definitions

**Core Definition:** An organizational structure is a hierarchical representation of an organization's reporting relationships, where each employee reports to exactly one manager (except the top executive).

**Technical Definition:** Organizational hierarchies are classic parent-child trees with a single root (CEO/President) and leaf nodes (individual contributors). The structure is typically stored as an adjacency list with a `manager_id` foreign key .

**Beginner-Friendly Explanation:** An org chart shows who reports to whom. The CEO is at the top, VPs report to the CEO, managers report to VPs, and so on. SQL can traverse this tree to answer questions like "Who are all the people in Bob's organization?" or "What is Alice's management chain?"

### Purposes

- To model reporting relationships for HR and access control systems.
- To enable organizational roll-up reporting (total headcount by department).
- To support approval workflows (manager chain of command).
- To implement permission inheritance based on organizational position.

### Syntax Rules and Structure

**Complete General Syntax (Recursive CTE for Subtree):**

```sql
WITH RECURSIVE org_subtree AS (
    -- Anchor: start with a specific manager
    SELECT id, name, manager_id, 0 AS depth
    FROM employees
    WHERE id = :manager_id
    
    UNION ALL
    
    -- Recursive: find all direct reports
    SELECT e.id, e.name, e.manager_id, os.depth + 1
    FROM employees e
    INNER JOIN org_subtree os ON e.manager_id = os.id
)
SELECT * FROM org_subtree;
```

**Complete General Syntax (Aggregated Headcount):**

```sql
WITH RECURSIVE org_subtree AS (
    SELECT id, name, manager_id
    FROM employees WHERE id = :manager_id
    UNION ALL
    SELECT e.id, e.name, e.manager_id
    FROM employees e
    INNER JOIN org_subtree os ON e.manager_id = os.id
)
SELECT COUNT(*) AS total_headcount FROM org_subtree;
```

**Syntax Rules:**

- The anchor selects the starting manager for subtree queries .
- The recursive member finds all descendants at each level .
- Aggregations (COUNT, SUM) can be applied to the CTE result for roll-up reporting.

**Constraints and Limitations:**

- Org structures change frequently (reorganizations, promotions); recursive queries reflect current data.
- Deep hierarchies (10+ levels) may cause performance issues.
- SQL Server's `hierarchyid` data type provides an optimized alternative for organizational trees .

### Annotated Complete Code Examples

**Example 1: Finding a Manager's Entire Organization**

```sql
-- Find all employees in Bob's organization (Bob and all his reports)
WITH RECURSIVE bob_org AS (
    -- Anchor: Bob
    SELECT id, name, manager_id, 0 AS depth
    FROM employees
    WHERE id = 2  -- Bob
    
    UNION ALL
    
    -- Recursive: all reports
    SELECT e.id, e.name, e.manager_id, bo.depth + 1
    FROM employees e
    INNER JOIN bob_org bo ON e.manager_id = bo.id
)
SELECT id, name, depth
FROM bob_org
ORDER BY depth, name;

-- Expected Output:
--  id | name  | depth
-- ----+-------+-------
--   2 | Bob   |     0
--   4 | Dave  |     1
--   5 | Eve   |     1
```

**Why this output occurs:** The anchor selects Bob (id=2). The recursive member finds employees whose `manager_id = 2` (Dave and Eve). No employees report to Dave or Eve, so recursion stops. Bob's organization contains Bob, Dave, and Eve .

**Example 2: Headcount by Organization**

```sql
-- Count total employees in each manager's organization
WITH RECURSIVE org_counts AS (
    -- Anchor: all employees as starting points
    SELECT id AS manager_id, id AS employee_id
    FROM employees
    
    UNION ALL
    
    -- Recursive: add all descendants
    SELECT oc.manager_id, e.id
    FROM employees e
    INNER JOIN org_counts oc ON e.manager_id = oc.employee_id
)
SELECT m.name AS manager_name, COUNT(DISTINCT oc.employee_id) - 1 AS team_size
FROM org_counts oc
JOIN employees m ON oc.manager_id = m.id
GROUP BY m.name
ORDER BY team_size DESC;

-- Expected Output (conceptual):
--  manager_name | team_size
-- --------------+-----------
--  Alice        |         6
--  Bob          |         2
--  Carol        |         2
```

**Why this output occurs:** The CTE starts each employee as a manager, then recursively adds all their descendants. The final `COUNT` counts all unique employees in each manager's organization (subtracting 1 to exclude the manager themselves) .

### Real-World Cases

**Case 1: Access Control by Organization**

A document management system grants file access based on organizational hierarchy. A user can access documents shared by anyone in their management chain (ancestors) or anyone in their organization (descendants). Recursive CTEs compute these relationships.

**Case 2: Reorganization Planning**

During a merger, HR needs to simulate the combined organizational tree. Recursive CTEs merge two org charts and identify overlapping roles or reporting conflicts.

### References

- How to Write Recursive CTEs in BigQuery for Hierarchical Data Traversal - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-02-17-how-to-write-recursive-ctes-in-bigquery-for-hierarchical-data-traversal/README.md 
- SQL Server Hierarchical Data - https://learn.microsoft.com/ja-jp/SQL/relational-databases/hierarchical-data-sql-server 

---

## 3. Category Trees

### Definitions

**Core Definition:** A category tree is a hierarchical classification system where products or content items are organized into nested categories, typically for e-commerce navigation, content management, or knowledge bases.

**Technical Definition:** Category trees use parent-child relationships (adjacency list) with a `parent_category_id` foreign key. The root categories have NULL parent references. Recursive queries retrieve category paths, subtrees, or leaf categories .

**Beginner-Friendly Explanation:** An e-commerce site has categories like "Electronics" > "Computers" > "Laptops." Each category can have subcategories. SQL can find all products in "Electronics" (including all subcategories) or build a breadcrumb trail showing the path from the root.

### Purposes

- To organize products/content into navigable hierarchies.
- To enable category-based filtering and search.
- To generate breadcrumb navigation for user interfaces.
- To support dynamic menu generation and sitemaps.

### Syntax Rules and Structure

**Complete General Syntax (Category Table):**

```sql
CREATE TABLE categories (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    parent_id INT,
    FOREIGN KEY (parent_id) REFERENCES categories(id)
);
```

**Complete General Syntax (Breadcrumb Path):**

```sql
WITH RECURSIVE category_path AS (
    -- Anchor: start with a specific category
    SELECT id, name, parent_id, 
           CAST(name AS CHAR(1000)) AS breadcrumb
    FROM categories
    WHERE id = :category_id
    
    UNION ALL
    
    -- Recursive: walk up to parent
    SELECT c.id, c.name, c.parent_id,
           CONCAT(c.name, ' > ', cp.breadcrumb)
    FROM categories c
    INNER JOIN category_path cp ON c.id = cp.parent_id
)
SELECT breadcrumb FROM category_path WHERE parent_id IS NULL;
```

**Syntax Rules:**

- The anchor selects the starting category (leaf or specific node) .
- The recursive member joins parent categories by matching `c.id = cp.parent_id` .
- Building the breadcrumb requires prepending parent names during recursion.
- The final selection filters for the root (`parent_id IS NULL`) to get the complete path.

**Constraints and Limitations:**

- Categories can be nested arbitrarily deep; recursive CTEs handle this.
- Category trees often need to support sorting (display order) and active/inactive status.
- Multiple parent categories (DAG) require different modeling (closure table) .

### Annotated Complete Code Examples

**Example 1: Category Tree with Breadcrumb Path**

```sql
-- Create category table
CREATE TABLE categories (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    parent_id INT,
    FOREIGN KEY (parent_id) REFERENCES categories(id)
);

-- Insert sample categories
INSERT INTO categories VALUES
    (1, 'Electronics', NULL),
    (2, 'Computers', 1),
    (3, 'Laptops', 2),
    (4, 'Desktops', 2),
    (5, 'Phones', 1),
    (6, 'Smartphones', 5),
    (7, 'Clothing', NULL),
    (8, 'Men', 7),
    (9, 'Women', 7);

-- Build breadcrumb path for Laptops (id=3)
WITH RECURSIVE category_path AS (
    -- Anchor: start with Laptops
    SELECT id, name, parent_id, 
           CAST(name AS CHAR(1000)) AS breadcrumb
    FROM categories
    WHERE id = 3
    
    UNION ALL
    
    -- Recursive: walk up to parent
    SELECT c.id, c.name, c.parent_id,
           CONCAT(c.name, ' > ', cp.breadcrumb)
    FROM categories c
    INNER JOIN category_path cp ON c.id = cp.parent_id
)
SELECT breadcrumb FROM category_path WHERE parent_id IS NULL;

-- Expected Output:
--  breadcrumb
-- ----------------------------
--  Electronics > Computers > Laptops
```

**Why this output occurs:** The anchor selects Laptops (id=3). The recursive member finds its parent (Computers, id=2), then Computers' parent (Electronics, id=1). The `breadcrumb` column prepends each parent's name. When `parent_id IS NULL` (Electronics), the full path is "Electronics > Computers > Laptops" .

**Example 2: Finding All Products in a Category Subtree**

```sql
-- Find all categories under Electronics (including Electronics)
WITH RECURSIVE electronics_subtree AS (
    -- Anchor: Electronics
    SELECT id, name, parent_id, 0 AS depth
    FROM categories
    WHERE id = 1
    
    UNION ALL
    
    -- Recursive: all descendants
    SELECT c.id, c.name, c.parent_id, es.depth + 1
    FROM categories c
    INNER JOIN electronics_subtree es ON c.parent_id = es.id
)
SELECT id, name, depth
FROM electronics_subtree
ORDER BY depth, name;

-- Expected Output:
--  id | name        | depth
-- ----+-------------+-------
--   1 | Electronics |     0
--   2 | Computers   |     1
--   5 | Phones      |     1
--   3 | Laptops     |     2
--   4 | Desktops    |     2
--   6 | Smartphones |     2
```

**Why this output occurs:** The anchor selects Electronics. The recursive member finds all categories whose `parent_id` matches the CTE's `id`, repeating for each level. This returns the entire subtree under Electronics, including nested subcategories .

### Real-World Cases

**Case 1: E-Commerce Navigation**

An online store generates a dynamic category menu. Recursive CTEs build the nested HTML structure from the category table, showing all levels with proper indentation.

**Case 2: Content Management Breadcrumbs**

A CMS displays breadcrumb navigation on article pages: "Home > Technology > Programming > SQL." A bottom-up recursive CTE builds this path from the article's category .

### References

- How to Use Recursive CTEs in MySQL 8 - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-how-to-use-recursive-ctes-in-mysql-8/README.md 
- How to Write Recursive CTEs in BigQuery for Hierarchical Data Traversal - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-02-17-how-to-write-recursive-ctes-in-bigquery-for-hierarchical-data-traversal/README.md 

---

## 4. Folder Structures

### Definitions

**Core Definition:** A folder structure is a hierarchical file system organization where folders (directories) contain files and subfolders, forming a tree rooted at the file system's top level.

**Technical Definition:** Folder structures use parent-child relationships with a `parent_folder_id` foreign key. The root folder has NULL parent. Recursive queries traverse the tree to list contents, compute folder paths, or calculate folder sizes .

**Beginner-Friendly Explanation:** Your computer's file system is a folder structure. "Documents" contains "Work" and "Personal." "Work" contains "Reports" and "Presentations." SQL can answer "What's the full path to this file?" or "How much space does this folder use including all subfolders?"

### Purposes

- To model file system hierarchies for document management systems.
- To compute full paths from root to any file or folder.
- To calculate aggregate sizes (sum of all files in a folder subtree).
- To support folder-based permission inheritance.

### Syntax Rules and Structure

**Complete General Syntax (Folder Table):**

```sql
CREATE TABLE folders (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    parent_id INT,
    is_folder BOOLEAN DEFAULT TRUE,
    file_size BIGINT DEFAULT 0,
    FOREIGN KEY (parent_id) REFERENCES folders(id)
);
```

**Complete General Syntax (Full Path):**

```sql
WITH RECURSIVE path AS (
    SELECT id, name, parent_id, CAST(name AS CHAR(4000)) AS full_path
    FROM folders WHERE id = :file_id
    UNION ALL
    SELECT f.id, f.name, f.parent_id, CONCAT(f.name, '/', p.full_path)
    FROM folders f
    INNER JOIN path p ON f.id = p.parent_id
)
SELECT full_path FROM path WHERE parent_id IS NULL;
```

**Syntax Rules:**

- The anchor selects the target file or folder .
- The recursive member prepends parent names with `/` separator.
- The root is identified by `parent_id IS NULL` .

**Constraints and Limitations:**

- File names may contain special characters; paths need proper escaping.
- Very deep folder structures (100+ levels) may hit recursion limits.
- File sizes must be aggregated carefully (folders have size 0, files have actual size).

### Annotated Complete Code Examples

**Example 1: Building a Full File Path**

```sql
-- Create folder structure table
CREATE TABLE filesystem (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    parent_id INT,
    is_folder BOOLEAN DEFAULT TRUE,
    file_size BIGINT DEFAULT 0
);

-- Insert sample structure
INSERT INTO filesystem VALUES
    (1, 'root', NULL, TRUE, 0),
    (2, 'home', 1, TRUE, 0),
    (3, 'user', 2, TRUE, 0),
    (4, 'documents', 3, TRUE, 0),
    (5, 'report.pdf', 4, FALSE, 1024000),
    (6, 'photo.jpg', 4, FALSE, 2048000);

-- Get full path for report.pdf (id=5)
WITH RECURSIVE file_path AS (
    -- Anchor: start with report.pdf
    SELECT id, name, parent_id, 
           CAST(name AS CHAR(4000)) AS full_path
    FROM filesystem
    WHERE id = 5
    
    UNION ALL
    
    -- Recursive: prepend parent names
    SELECT f.id, f.name, f.parent_id,
           CONCAT(f.name, '/', fp.full_path)
    FROM filesystem f
    INNER JOIN file_path fp ON f.id = fp.parent_id
)
SELECT full_path FROM file_path WHERE parent_id IS NULL;

-- Expected Output:
--  full_path
-- ----------------------------------
--  root/home/user/documents/report.pdf
```

**Why this output occurs:** The anchor selects report.pdf. The recursive member walks up: documents → user → home → root. The `full_path` column builds the path by prepending each parent with `/`. The final selection filters for the root (`parent_id IS NULL`), yielding the complete path .

**Example 2: Calculating Total Folder Size**

```sql
-- Calculate total size of the 'documents' folder (id=4)
WITH RECURSIVE folder_contents AS (
    -- Anchor: documents folder
    SELECT id, name, parent_id, file_size
    FROM filesystem
    WHERE id = 4
    
    UNION ALL
    
    -- Recursive: all files and folders inside
    SELECT f.id, f.name, f.parent_id, f.file_size
    FROM filesystem f
    INNER JOIN folder_contents fc ON f.parent_id = fc.id
)
SELECT SUM(file_size) AS total_size_bytes
FROM folder_contents
WHERE is_folder = FALSE;  -- Only count files, not folders

-- Expected Output:
--  total_size_bytes
-- ------------------
--           3072000
```

**Why this output occurs:** The CTE collects all descendants of the documents folder. The final aggregation sums the `file_size` of all files (excluding folders, which have size 0). This computes the total storage used by the folder and all its contents .

### Real-World Cases

**Case 1: Document Management System**

A DMS stores documents in a virtual folder structure. Users can navigate folders, and the system computes breadcrumb paths and folder sizes using recursive CTEs.

**Case 2: Cloud Storage Quota Calculation**

A cloud storage provider tracks each user's folder structure and calculates total storage used per user by summing all file sizes in their folder tree.

### References

- SQL Server Hierarchical Data - https://learn.microsoft.com/ja-jp/SQL/relational-databases/hierarchical-data-sql-server 
- How to Use Recursive CTEs in MySQL 8 - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-how-to-use-recursive-ctes-in-mysql-8/README.md 

---

## 5. Bill-of-Materials (BOM) Structures

### Definitions

**Core Definition:** A Bill-of-Materials (BOM) is a hierarchical list of all raw materials, components, and sub-assemblies required to manufacture a product, organized by assembly levels.

**Technical Definition:** BOM structures use parent-child relationships where a parent part contains multiple child parts. Each relationship has a quantity (how many of the child are needed per parent). Recursive queries explode the BOM to list all components or compute total quantities .

**Beginner-Friendly Explanation:** A BOM is like a recipe for a manufactured product. A bicycle has 2 wheels, 1 frame, and 1 handlebar. Each wheel has 1 tire and 32 spokes. SQL can "explode" the BOM to list every component needed to build one bicycle, including all sub-components.

### Purposes

- To model manufacturing assembly hierarchies.
- To explode BOMs for material requirements planning (MRP).
- To compute total component quantities (including sub-assemblies).
- To identify where-used relationships (which products use a specific part).

### Syntax Rules and Structure

**Complete General Syntax (BOM Table):**

```sql
CREATE TABLE bom (
    id INT PRIMARY KEY,
    parent_part_id INT,
    child_part_id INT,
    quantity DECIMAL(10,4),
    FOREIGN KEY (parent_part_id) REFERENCES parts(id),
    FOREIGN KEY (child_part_id) REFERENCES parts(id)
);
```

**Complete General Syntax (BOM Explosion):**

```sql
WITH RECURSIVE bom_explosion AS (
    -- Anchor: start with the top-level product
    SELECT parent_part_id, child_part_id, quantity, 
           1 AS level, quantity AS total_quantity
    FROM bom
    WHERE parent_part_id = :product_id
    
    UNION ALL
    
    -- Recursive: multiply quantities by parent quantity
    SELECT b.parent_part_id, b.child_part_id, b.quantity,
           be.level + 1, b.quantity * be.total_quantity
    FROM bom b
    INNER JOIN bom_explosion be ON b.parent_part_id = be.child_part_id
)
SELECT child_part_id, SUM(total_quantity) AS total_needed
FROM bom_explosion
GROUP BY child_part_id;
```

**Syntax Rules:**

- The anchor selects the top-level product's immediate components .
- The recursive member multiplies the child's quantity by the parent's cumulative quantity.
- The `total_quantity` accumulates the multiplier through the hierarchy.

**Constraints and Limitations:**

- BOMs can have multiple levels (sub-assemblies within sub-assemblies).
- Circular BOMs (part A contains B, B contains A) are invalid; cycle detection is needed.
- Quantity precision requires appropriate data types (DECIMAL, not FLOAT).

### Annotated Complete Code Examples

**Example 1: BOM Explosion for a Bicycle**

```sql
-- Create parts table
CREATE TABLE parts (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

-- Create BOM table
CREATE TABLE bom (
    id INT PRIMARY KEY,
    parent_part_id INT,
    child_part_id INT,
    quantity DECIMAL(10,4),
    FOREIGN KEY (parent_part_id) REFERENCES parts(id),
    FOREIGN KEY (child_part_id) REFERENCES parts(id)
);

-- Insert parts
INSERT INTO parts VALUES
    (1, 'Bicycle'),
    (2, 'Wheel'),
    (3, 'Frame'),
    (4, 'Tire'),
    (5, 'Spoke'),
    (6, 'Handlebar');

-- Insert BOM relationships
INSERT INTO bom VALUES
    (1, 1, 2, 2),      -- Bicycle needs 2 Wheels
    (2, 1, 3, 1),      -- Bicycle needs 1 Frame
    (3, 1, 6, 1),      -- Bicycle needs 1 Handlebar
    (4, 2, 4, 1),      -- Wheel needs 1 Tire
    (5, 2, 5, 32);     -- Wheel needs 32 Spokes

-- Explode BOM for Bicycle (id=1)
WITH RECURSIVE bom_explosion AS (
    -- Anchor: direct components of Bicycle
    SELECT parent_part_id, child_part_id, quantity, 
           1 AS level, quantity AS total_quantity
    FROM bom
    WHERE parent_part_id = 1
    
    UNION ALL
    
    -- Recursive: multiply quantities through sub-assemblies
    SELECT b.parent_part_id, b.child_part_id, b.quantity,
           be.level + 1, b.quantity * be.total_quantity
    FROM bom b
    INNER JOIN bom_explosion be ON b.parent_part_id = be.child_part_id
)
SELECT p.name AS part_name, 
       SUM(be.total_quantity) AS total_quantity
FROM bom_explosion be
JOIN parts p ON be.child_part_id = p.id
GROUP BY p.name
ORDER BY total_quantity DESC;

-- Expected Output:
--  part_name  | total_quantity
-- ------------+----------------
--  Spoke      |             64
--  Wheel      |              2
--  Tire       |              2
--  Frame      |              1
--  Handlebar  |              1
```

**Why this output occurs:** The anchor selects direct Bicycle components: Wheel (qty 2), Frame (qty 1), Handlebar (qty 1). The recursive member expands Wheel into Tire (qty 1 × 2 = 2) and Spoke (qty 32 × 2 = 64). The final aggregation sums quantities per part, showing that building one bicycle requires 64 spokes, 2 tires, 2 wheels, etc. .

### Real-World Cases

**Case 1: Manufacturing Material Planning**

An ERP system explodes BOMs to generate purchase orders for all raw materials needed to fulfill production orders, accounting for sub-assembly quantities.

**Case 2: Cost Roll-Up**

A product's total cost is computed by rolling up component costs through the BOM. Recursive queries multiply each component's unit cost by its total quantity and sum the results.

### References

- How to Write Recursive CTEs in BigQuery for Hierarchical Data Traversal - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-02-17-how-to-write-recursive-ctes-in-bigquery-for-hierarchical-data-traversal/README.md 

---

## 6. Graph Structures (Network Topologies, Social Connections)

### Definitions

**Core Definition:** Graph structures represent entities (nodes) and their relationships (edges) as a network, where nodes can have multiple connections and cycles are possible, unlike strict hierarchies.

**Technical Definition:** Graph data in SQL is modeled using node tables and edge tables. Node tables contain entity attributes; edge tables contain relationship attributes and foreign keys to the connected nodes. SQL Server 2017+ provides native `AS NODE` and `AS EDGE` syntax with `MATCH` pattern matching .

**Beginner-Friendly Explanation:** A graph is like a social network. People (nodes) are connected by friendships (edges). Unlike a tree, you can have cycles (A knows B, B knows C, C knows A) and multiple connection types (friend, follows, likes). SQL can traverse these networks to find paths, connections, and communities.

### Purposes

- To model social networks (friends, followers, connections).
- To represent network topologies (routers, links, paths).
- To support recommendation engines (similar users, products).
- To enable pathfinding and connectivity analysis.

### Syntax Rules and Structure

**Complete General Syntax (SQL Server Graph):**

```sql
-- Create node table
CREATE TABLE Person (
    ID INTEGER PRIMARY KEY,
    name VARCHAR(100)
) AS NODE;

-- Create edge table
CREATE TABLE friendOf AS EDGE;

-- Insert nodes
INSERT INTO Person (ID, name) VALUES (1, 'John'), (2, 'Mary');

-- Insert edges (using $node_id from node tables)
INSERT INTO friendOf VALUES (
    (SELECT $node_id FROM Person WHERE ID = 1),
    (SELECT $node_id FROM Person WHERE ID = 2)
);
```

**Complete General Syntax (Pattern Matching):**

```sql
SELECT p1.name AS person1, p2.name AS person2
FROM Person p1, friendOf, Person p2
WHERE MATCH(p1-(friendOf)->p2);
```

**Syntax Rules:**

- Node tables use `AS NODE`; edge tables use `AS EDGE` .
- Edge tables have implicit `$from_id` and `$to_id` columns .
- `MATCH` clause uses ASCII-art patterns to describe graph traversals .
- `SHORTEST_PATH` finds the shortest path between nodes (SQL Server 2019+) .

**Constraints and Limitations:**

- SQL Server graph features require 2017+ .
- PostgreSQL and MySQL lack native graph syntax; use recursive CTEs instead.
- Graph queries can be expensive; indexes on edge columns are essential.

### Annotated Complete Code Examples

**Example 1: Social Network with SQL Server Graph**

```sql
-- Create graph database tables
CREATE TABLE Person (
    ID INTEGER PRIMARY KEY,
    name VARCHAR(100)
) AS NODE;

CREATE TABLE friendOf AS EDGE;

-- Insert people
INSERT INTO Person (ID, name)
VALUES (1, 'John'), (2, 'Mary'), (3, 'Alice'), (4, 'Jacob');

-- Insert friendships (using $node_id)
INSERT INTO friendOf
VALUES 
    ((SELECT $node_id FROM Person WHERE ID = 1), (SELECT $node_id FROM Person WHERE ID = 2)),
    ((SELECT $node_id FROM Person WHERE ID = 2), (SELECT $node_id FROM Person WHERE ID = 3)),
    ((SELECT $node_id FROM Person WHERE ID = 3), (SELECT $node_id FROM Person WHERE ID = 4));

-- Find friends of friends (2 hops)
SELECT p1.name AS person, p3.name AS friend_of_friend
FROM Person p1, friendOf f1, Person p2, friendOf f2, Person p3
WHERE MATCH(p1-(f1)->p2-(f2)->p3)
  AND p1.name = 'John';

-- Expected Output (conceptual):
--  person | friend_of_friend
-- --------+------------------
--  John   | Alice
```

**Why this output occurs:** The `MATCH` pattern `p1-(f1)->p2-(f2)->p3` describes a path of two friendship edges. Starting from John, the query finds John → Mary → Alice. The result shows Alice as a friend-of-a-friend of John .

**Example 2: Shortest Path in SQL Server Graph**

```sql
-- Find shortest path between John and Jacob
SELECT p1.name AS start_person, 
       STRING_AGG(p2.name, ' -> ') AS path
FROM Person p1, friendOf, Person p2
WHERE MATCH(SHORTEST_PATH(p1(-(friendOf)->p2)+))
  AND p1.name = 'John'
  AND p2.name = 'Jacob';

-- Expected Output (conceptual):
--  start_person | path
-- --------------+---------------------------
--  John         | John -> Mary -> Alice -> Jacob
```

**Why this output occurs:** `SHORTEST_PATH` finds the minimum number of hops between John and Jacob. The `+` quantifier allows the pattern to repeat until the destination is reached. The result shows the path John → Mary → Alice → Jacob .

**Example 3: Social Network with Recursive CTE (MySQL/PostgreSQL)**

```sql
-- Model friendship as undirected edges
CREATE TABLE friendships (
    person1_id INT,
    person2_id INT,
    PRIMARY KEY (person1_id, person2_id)
);

-- Find all friends of friends for person 1
WITH RECURSIVE friend_network AS (
    -- Anchor: direct friends of person 1
    SELECT person2_id AS friend_id, 1 AS depth
    FROM friendships WHERE person1_id = 1
    UNION
    SELECT person1_id, 1
    FROM friendships WHERE person2_id = 1
    
    UNION ALL
    
    -- Recursive: friends of friends
    SELECT CASE 
               WHEN f.person1_id = fn.friend_id THEN f.person2_id
               ELSE f.person1_id
           END,
           fn.depth + 1
    FROM friendships f
    JOIN friend_network fn ON f.person1_id = fn.friend_id 
                           OR f.person2_id = fn.friend_id
    WHERE fn.depth < 3  -- Limit depth to prevent infinite recursion
)
SELECT DISTINCT friend_id, MIN(depth) AS distance
FROM friend_network
GROUP BY friend_id
ORDER BY distance;
```

**Why this output occurs:** The anchor finds direct friends of person 1. The recursive member joins friendships where either person matches the current friend, expanding the network. `UNION` (not `UNION ALL`) prevents duplicates. The `depth < 3` condition limits traversal to 3 hops .

### Real-World Cases

**Case 1: Social Network Friend Recommendations**

A social platform recommends "People You May Know" by finding friends-of-friends who aren't already direct friends. Recursive CTEs or `MATCH` patterns compute these second-degree connections.

**Case 2: Network Topology Analysis**

A network monitoring system models routers as nodes and links as edges. Shortest path queries identify optimal routes; connectivity queries detect network partitions.

### References

- SQL Graph Sample (SQL Server) - https://learn.microsoft.com/de-de/sql/relational-databases/graphs/sql-graph-sample 
- MATCH (Transact-SQL) - https://learn.microsoft.com/nl-be/sql/t-sql/queries/match-sql-graph 
- Social Network Table Strategy (PostgreSQL) - https://www.postgresql.org/message-id/BANLkTimQ%3D8vk-eGxEyYCefB3RJYdN%3D9Xxg%40mail.gmail.com 

---

## Summary Table

| Concept | Model | Key SQL Technique | Primary Use Case |
|---------|-------|-------------------|------------------|
| **Parent-Child** | Adjacency List | Recursive CTE (`WITH RECURSIVE`) | General hierarchies |
| **Organizational Structures** | Adjacency List / `hierarchyid` | Recursive CTE, `GetAncestor()` | HR, access control |
| **Category Trees** | Adjacency List | Recursive CTE (top-down, bottom-up) | E-commerce, CMS |
| **Folder Structures** | Adjacency List | Recursive CTE (path building) | File systems, DMS |
| **Bill-of-Materials** | Adjacency List with quantity | Recursive CTE (quantity multiplication) | Manufacturing, ERP |
| **Graph Structures** | Node/Edge Tables | `MATCH`, `SHORTEST_PATH` (SQL Server) | Social networks, topology |

---

## References

- How to Write Recursive CTEs in BigQuery for Hierarchical Data Traversal - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-02-17-how-to-write-recursive-ctes-in-bigquery-for-hierarchical-data-traversal/README.md 
- How to Use Recursive CTEs in MySQL 8 - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-how-to-use-recursive-ctes-in-mysql-8/README.md 
- SQL Server Hierarchical Data - https://learn.microsoft.com/ja-jp/SQL/relational-databases/hierarchical-data-sql-server 
- SQL Graph Sample (SQL Server) - https://learn.microsoft.com/de-de/sql/relational-databases/graphs/sql-graph-sample 
- MATCH (Transact-SQL) - https://learn.microsoft.com/nl-be/sql/t-sql/queries/match-sql-graph 
- SQL Property Graph Queries (PostgreSQL) - https://postgrespro.com/list/id/CAAAe_zDbS-uDYgbQ3pQn+BD02JvCTek2LzpdK4b-xy48k7ZMmg@mail.gmail.com 
- Social Network Table Strategy (PostgreSQL) - https://www.postgresql.org/message-id/BANLkTimQ%3D8vk-eGxEyYCefB3RJYdN%3D9Xxg%40mail.gmail.com 
- DuckPGQ: Bringing SQL/PGQ to DuckDB - https://www.vldb.org/pvldb/vol16/p4034-wolde.pdf 
- Hierarchical Model (Database Systems) - https://db-book.com/Previous-editions/db5/slide-dir/appB.pdf 
- Methods of Modeling Hierarchical Structures in Relational Databases - https://apcssm.vnu.edu.ua/index.php/Journalone/article/download/126/95/425 
- Stack Overflow: Hierarchical Data Options - https://stackoverflow.com/revisions/249a7e57-8355-4dfb-a690-5eaaf0848d2f/view-source 
- SQL Server Recursive CTE DML Reference - https://download.microsoft.com/download/0/F/B/0FBFAA46-2BFD-478F-8E56-7BF3C672DF9D/SQL%20Server%202012%20Transact-SQL%20DML%20Reference.pdf 
- Oracle Analytics Cloud: Parent-Child Hierarchies - https://docs.oracle.com/cd/E83857_01/paas/analytics-cloud/acmdg/building-semantic-models-oracle-analytics-cloud.pdf 
- 数据库技术基础：常见基本模型介绍笔记 - https://bbs.huaweicloud.com/blogs/388575 
- 数据库模型:层次模型、网状模型、关系模型 - https://developer.aliyun.com/article/1453050 