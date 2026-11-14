# Comprehensive Programming Cheat Sheet: Advanced Views

## Topic Overview

### Definitions

**Core Definition**
Advanced views are view objects that extend beyond simple read-only projections, encompassing views that support data modification (updatable), enforce security boundaries (security views), combine multiple tables or aggregations (complex views), reference other views (nested views), or physically store their results (materialized views).

**Technical Definition**
Advanced views exploit the full range of view capabilities defined by SQL standards and vendor extensions. **Updatable views** allow `INSERT`, `UPDATE`, and `DELETE` operations that are propagated to underlying base tables, either automatically (when the view satisfies strict structural conditions) or through `INSTEAD OF` triggers. **Read-only views** are those that cannot be modified directly due to structural disqualifiers (joins, aggregates, `DISTINCT`, set operations). **Security views** enforce access control through definer/invoker privilege models and barrier mechanisms. **Complex views** involve multi-table joins, aggregations, window functions, or set operations. **Nested views** reference other views, creating dependency chains that the query rewriter expands. **Materialized views** (called indexed views in SQL Server) physically persist their result set and require explicit refresh policies, trading storage and refresh cost for query performance.

**Beginner-Friendly Explanation**
Basic views are like saved searches: they show you data but you cannot change anything through them. Advanced views go further. Some let you insert or update data through the view (updatable views). Some are designed to hide sensitive rows or columns from certain users (security views). Some combine many tables or calculations (complex views). Some are built on top of other views (nested views). And some actually store their results on disk so queries run faster, at the cost of needing to be refreshed (materialized views).

### Key Characteristics

- **Updatability is structural**: A view is automatically updatable only if it has exactly one `FROM` entry (a table or updatable view), no `DISTINCT`, `GROUP BY`, `HAVING`, `LIMIT`, `OFFSET`, set operations, aggregates, or window functions at the top level; columns must be simple references to base-table columns.
- **`INSTEAD OF` triggers extend updatability**: Non-updatable views can be made writable by defining `INSTEAD OF` triggers that redirect changes to base tables.
- **Security views rely on privilege models**: PostgreSQL uses the view owner's privileges by default; `security_invoker = true` (PG 15+) switches to the caller's privileges. `security_barrier` prevents predicate pushdown that could leak rows.
- **Complex views sacrifice updatability for expressiveness**: Joins, aggregations, and set operations make views read-only unless triggers are added.
- **Nested views are expanded at rewrite time**: The query rewriter substitutes view definitions into the outer query, so performance depends on whether the optimizer can flatten the expansion.
- **Materialized views trade freshness for speed**: They store results physically and must be refreshed; PostgreSQL supports `REFRESH MATERIALIZED VIEW CONCURRENTLY` (requires a unique index), Oracle supports `ON COMMIT`, `ON DEMAND`, and `ON STATEMENT` refresh modes.
- **Vendor divergence is significant**: PostgreSQL, MySQL, SQL Server, and Oracle differ in automatic updatability rules, security view options, and materialized view support.

### Prerequisites

- Understanding of basic `CREATE VIEW`, `SELECT`, joins, and aggregation.
- Familiarity with `INSERT`, `UPDATE`, `DELETE`, and transaction semantics.
- Knowledge of database privileges and ownership models.
- Awareness of query rewriting and execution plans.
- For materialized views: understanding of refresh strategies and index requirements.

### Related Programming Areas

- Database security and access control (row-level security, column masking).
- Query performance tuning and materialized view selection.
- Data warehousing and reporting layers.
- ETL and data refresh orchestration.
- Schema abstraction and API design.

---

## Core Concepts and Features

### 1. Updatable Views

#### Definitions

**Core Definition**
An updatable view is a view through which `INSERT`, `UPDATE`, and `DELETE` statements can be executed, with the database translating those operations into corresponding modifications on the underlying base table(s).

**Technical Definition**
In PostgreSQL, a view is automatically updatable if it has exactly one entry in its `FROM` list (a table or another updatable view) and its top-level definition contains no `WITH`, `DISTINCT`, `GROUP BY`, `HAVING`, `LIMIT`, `OFFSET`, set operations (`UNION`, `INTERSECT`, `EXCEPT`), aggregates, window functions, or set-returning functions. Columns are updatable only if they are simple references to updatable base-table columns; expression columns are read-only. When these conditions are met, the system converts `INSERT`, `UPDATE`, `DELETE`, and `MERGE` statements on the view into the corresponding statement on the base relation. In MySQL, views are updatable if there is a one-to-one relationship between view rows and underlying table rows; `ALGORITHM = TEMPTABLE` always makes a view non-updatable. In SQL Server, views involving aggregates or joins are not directly updatable, but `INSTEAD OF` triggers can make them updatable.

**Beginner-Friendly Explanation**
An updatable view lets you treat the view like a real table for changes. You can `INSERT` a new row into it, `UPDATE` a value, or `DELETE` a row, and the database figures out how to apply that change to the real table behind the view. But this only works if the view is simple enough—if it joins multiple tables or uses `GROUP BY`, the database usually will not let you modify it directly.

#### Purposes

- To provide a simplified modification interface that hides the underlying table structure.
- To enforce row-level security by restricting which rows can be inserted or updated through the view.
- To maintain a stable API when base table schemas change.
- To allow applications to perform set-based modifications through a view without knowing the normalized schema.

#### Syntax Rules and Structure

**PostgreSQL automatic updatability conditions**:

| Condition | Required? |
|-----------|-----------|
| Exactly one `FROM` entry (table or updatable view) | Yes |
| No top-level `WITH`, `DISTINCT`, `GROUP BY`, `HAVING`, `LIMIT`, `OFFSET` | Yes |
| No top-level set operations (`UNION`, `INTERSECT`, `EXCEPT`) | Yes |
| No aggregates, window functions, or set-returning functions in select list | Yes |
| Columns must be simple references to updatable base-table columns | Yes (for updatable columns) |

**`WITH CHECK OPTION`** :

```sql
CREATE VIEW active_customers AS
SELECT customer_id, name, status
FROM customers
WHERE status = 'active'
WITH CHECK OPTION;
```

**`INSTEAD OF` trigger pattern (PostgreSQL)** :

```sql
CREATE FUNCTION view_insert_fn() RETURNS trigger AS $$
BEGIN
    INSERT INTO base_table (col1, col2) VALUES (NEW.col1, NEW.col2);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER view_insert_trg
INSTEAD OF INSERT ON view_name
FOR EACH ROW EXECUTE FUNCTION view_insert_fn();
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `WITH CHECK OPTION` | Rejects modifications that would make rows invisible through the view (e.g., changing status from active to inactive). |
| `INSTEAD OF` trigger | Intercepts DML on a non-updatable view and redirects it to base tables. |
| `ALGORITHM = MERGE` | MySQL: inline the view into the outer query; often updatable. |
| `ALGORITHM = TEMPTABLE` | MySQL: materialize the view; always non-updatable. |

**Syntax rules**

- `WITH CHECK OPTION` is only supported on automatically updatable views without `INSTEAD OF` triggers or rules.
- An automatically updatable view may contain a mix of updatable and non-updatable columns; assigning to a read-only column raises an error.
- `INSERT` into an updatable view requires that all non-defaulted base-table columns are provided or have defaults.
- `UPDATE` through a view can only affect rows visible through the view's `WHERE` condition.

**Constraints and limitations**

- Aggregates, `DISTINCT`, `GROUP BY`, `HAVING`, `UNION`/`INTERSECT`/`EXCEPT`, and window functions at the top level disqualify a view from automatic updatability.
- MySQL views processed with `TEMPTABLE` are never updatable.
- SQL Server views involving aggregates or joins require `INSTEAD OF` triggers for updates.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Automatically updatable view with `WITH CHECK OPTION`

```sql
-- Step 1: Create a simple updatable view.
CREATE VIEW active_customers AS
SELECT customer_id, name, email, status
FROM customers
WHERE status = 'active'
WITH CHECK OPTION;

-- Step 2: Insert a new active customer through the view.
INSERT INTO active_customers (customer_id, name, email, status)
VALUES (105, 'Initech', 'initech@example.com', 'active');

-- Step 3: This update is rejected because it would make the row invisible.
UPDATE active_customers SET status = 'inactive' WHERE customer_id = 105;
```

**Expected output** (Step 3):

```text
ERROR: new row violates check option for view "active_customers"
```

**Why this output appears**: The view is automatically updatable (single table, no disqualifying clauses), so the `INSERT` is rewritten to insert into `customers`. The `WITH CHECK OPTION` enforces that modified rows must continue to satisfy `status = 'active'`; changing to `'inactive'` would make the row disappear from the view, so it is rejected.

##### Example 2: PostgreSQL – `INSTEAD OF` trigger for a join view

```sql
-- Step 1: Create a non-updatable join view.
CREATE VIEW customer_orders AS
SELECT c.customer_id, c.name, o.order_id, o.total
FROM customers c
JOIN orders o ON o.customer_id = c.customer_id;

-- Step 2: Create an INSTEAD OF INSERT trigger.
CREATE FUNCTION insert_customer_order() RETURNS trigger AS $$
BEGIN
    INSERT INTO orders (order_id, customer_id, total)
    VALUES (NEW.order_id, NEW.customer_id, NEW.total);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER customer_orders_insert
INSTEAD OF INSERT ON customer_orders
FOR EACH ROW EXECUTE FUNCTION insert_customer_order();

-- Step 3: Insert through the view.
INSERT INTO customer_orders (customer_id, name, order_id, total)
VALUES (101, 'Acme Corp', 9003, 75.00);
```

**Expected output**:

```text
INSERT 0 1
```

**Why this output appears**: The join view is not automatically updatable. The `INSTEAD OF` trigger intercepts the insert and redirects it to the `orders` table, ignoring the `name` column from `customers`.

#### Real-World Cases

- **Row-level security**: A view filters rows by tenant, and updates through the view can only affect the tenant's rows.
- **Application API stability**: Applications insert/update through a view while the underlying schema is normalized or refactored.
- **Audit trails**: `INSTEAD OF` triggers log changes made through a view.

---

### 2. Read-Only Views

#### Definitions

**Core Definition**
A read-only view is a view that cannot be used with `INSERT`, `UPDATE`, or `DELETE` statements, either because its structure disqualifies it from automatic updatability or because it was explicitly designed to be non-modifiable.

**Technical Definition**
A view is read-only when it fails the automatic updatability conditions: multiple `FROM` entries (joins), `DISTINCT`, `GROUP BY`, `HAVING`, `LIMIT`, `OFFSET`, set operations, aggregates, window functions, or expression columns that cannot be mapped to base-table columns. Read-only views are still fully queryable and are commonly used for reporting, abstraction, and security. In MySQL, views processed with `ALGORITHM = TEMPTABLE` are read-only. In SQL Server, views with aggregates or joins are read-only unless `INSTEAD OF` triggers are added.

**Beginner-Friendly Explanation**
A read-only view is a view you can only `SELECT` from. You cannot insert, update, or delete through it because the view's structure is too complex for the database to figure out which base-table row to change. It is like a window you can look through but not reach through.

#### Purposes

- To provide a stable, queryable interface over complex logic without allowing modification.
- To prevent accidental data changes through reporting views.
- To expose aggregated or joined data for analysis while keeping base tables protected.
- To serve as a security layer that hides writable base tables.

#### Syntax Rules and Structure

**Read-only disqualifiers** (any of the following at the top level):

| Disqualifier | Reason |
|--------------|--------|
| Multiple `FROM` entries (joins) | Cannot unambiguously map a view row to one base-table row. |
| `DISTINCT` | Rows may represent multiple base rows. |
| `GROUP BY` / `HAVING` | Aggregated rows have no direct base-row mapping. |
| `UNION` / `INTERSECT` / `EXCEPT` | Rows may come from different sources. |
| Aggregates (`SUM`, `AVG`, etc.) | Computed values cannot be written back. |
| Window functions | Computed values cannot be written back. |
| Expression columns | Cannot map expression back to a single base column. |
| `LIMIT` / `OFFSET` | Subset semantics are ambiguous for writes. |

**Example of a read-only view**:

```sql
CREATE VIEW sales_summary AS
SELECT region, product_category, SUM(amount) AS total_sales
FROM sales
GROUP BY region, product_category;
```

**Component breakdown**

| Component | Read-only? | Why |
|-----------|------------|-----|
| Single table, simple columns | No (updatable) | Maps directly to base table. |
| Join of two tables | Yes | Cannot unambiguously map rows. |
| `GROUP BY` with aggregates | Yes | Aggregated rows have no single base row. |
| `UNION` of two queries | Yes | Rows may come from either source. |

**Syntax rules**

- Read-only views are still fully queryable with `SELECT`, joins, filters, and aggregations.
- Attempting `INSERT`, `UPDATE`, or `DELETE` on a read-only view raises an error unless `INSTEAD OF` triggers are defined.
- `WITH CHECK OPTION` is not applicable to read-only views.

**Constraints and limitations**

- Read-only views cannot be used as the target of DML unless triggers are added.
- Some engines may still allow DML on certain complex views under specific conditions (e.g., MySQL merge algorithm for simple joins).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Aggregated view is read-only

```sql
CREATE VIEW department_totals AS
SELECT department_id, SUM(salary) AS total_salary
FROM employees
GROUP BY department_id;

-- Attempt to update raises an error.
UPDATE department_totals SET total_salary = 0 WHERE department_id = 10;
```

**Expected output**:

```text
ERROR: cannot update view "department_totals"
DETAIL: Views that do not select from a single table or view are not automatically updatable.
```

**Why this output appears**: The view uses `GROUP BY` and `SUM`, which disqualify it from automatic updatability. The `UPDATE` is rejected because the database cannot map `total_salary` back to individual employee rows.

##### Example 2: MySQL – `ALGORITHM = TEMPTABLE` forces read-only

```sql
CREATE ALGORITHM = TEMPTABLE VIEW recent_orders AS
SELECT order_id, customer_id, total
FROM orders
WHERE order_date >= '2025-01-01';

UPDATE recent_orders SET total = 0 WHERE order_id = 9001;
```

**Expected output**:

```text
ERROR 1288 (HY000): The target table recent_orders of the UPDATE is not updatable
```

**Why this output appears**: `ALGORITHM = TEMPTABLE` forces MySQL to materialize the view into a temporary table, which is always non-updatable. Even though the underlying query is simple, the explicit algorithm choice makes the view read-only.

#### Real-World Cases

- **Reporting views**: Aggregated sales summaries that should never be modified.
- **Security views**: Views exposing only read access to sensitive data while base tables remain writable by privileged users.
- **Cross-system integration**: Views over foreign tables or federated queries that are inherently read-only.

---

### 3. Security Views

#### Definitions

**Core Definition**
A security view is a view designed to enforce access control by restricting which rows or columns a user can see or modify, leveraging privilege-checking models and barrier mechanisms to prevent information leakage.

**Technical Definition**
In PostgreSQL, views are executed with the privileges of the view owner by default: a user granted `SELECT` on a view can access underlying tables through the view even without direct table privileges. `security_barrier = true` prevents the optimizer from pushing user-supplied predicates below the view's own filters, which could otherwise leak rows through side-channel functions. `security_invoker = true` (PostgreSQL 15+) changes privilege checking to use the caller's privileges instead of the view owner's, enabling row-level security policies to apply to the invoking user. In MySQL, `SQL SECURITY { DEFINER | INVOKER }` controls whether the view executes with the definer's or invoker's privileges. SQL Server uses ownership chaining and `GRANT`/`DENY` permissions on views.

**Beginner-Friendly Explanation**
A security view is like a filtered window that only shows certain rows or columns to certain people. In PostgreSQL, by default the view runs with the creator's permissions, so you can give someone access to the view without giving them access to the underlying tables. The `security_barrier` option stops clever users from bypassing the filter, and `security_invoker` makes the view check the user's own permissions instead of the creator's.

#### Purposes

- To grant access to a subset of data without granting direct table privileges.
- To enforce row-level and column-level security through view definitions.
- To prevent predicate pushdown from leaking rows through malicious functions (`security_barrier`).
- To apply the caller's row-level security policies (`security_invoker` in PostgreSQL 15+).
- To implement multi-tenant data isolation.

#### Syntax Rules and Structure

**PostgreSQL security view options**:

```sql
CREATE VIEW secure_view
WITH (security_barrier = true, security_invoker = true)
AS
SELECT * FROM sensitive_table
WHERE tenant_id = current_setting('app.tenant_id')::int;
```

**MySQL `SQL SECURITY`** :

```sql
CREATE SQL SECURITY INVOKER VIEW customer_view AS
SELECT customer_id, name FROM customers;
```

**Component breakdown**

| Component | Engine | Effect |
|-----------|--------|--------|
| `security_barrier = true` | PostgreSQL | Prevents predicate pushdown below the view's `WHERE` clause. |
| `security_invoker = true` | PostgreSQL 15+ | Checks underlying object privileges and RLS policies as the calling user. |
| `SQL SECURITY DEFINER` | MySQL | Executes with the view creator's privileges (default). |
| `SQL SECURITY INVOKER` | MySQL | Executes with the calling user's privileges. |
| Ownership chaining | SQL Server | Permissions on underlying objects are not checked if the view and tables share an owner. |

**Syntax rules**

- `security_barrier` should be set whenever a view is used to restrict rows for security purposes.
- `security_invoker` requires PostgreSQL 15 or later.
- `SQL SECURITY INVOKER` in MySQL means the invoker must have privileges on the underlying tables.
- Without `security_barrier`, the optimizer may push user predicates below the view's filter, potentially exposing rows through side-channel attacks.

**Constraints and limitations**

- `security_barrier` can degrade performance by preventing predicate pushdown.
- `security_invoker` may require granting underlying table privileges to the invoking user, which may conflict with least-privilege goals.
- SQL Server ownership chaining only works when the view and base tables have the same owner.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – `security_invoker` with row-level security

```sql
-- Step 1: Create a table with RLS enabled.
CREATE TABLE documents (
    doc_id SERIAL PRIMARY KEY,
    tenant_id INT NOT NULL,
    content TEXT
);
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_policy ON documents
    USING (tenant_id = current_setting('app.tenant_id')::int);

-- Step 2: Create a security_invoker view.
CREATE VIEW tenant_documents
WITH (security_invoker = true)
AS SELECT * FROM documents;

-- Step 3: A user with app.tenant_id = 1 sees only their rows.
SET app.tenant_id = '1';
SELECT * FROM tenant_documents;
```

**Expected output**: Only rows where `tenant_id = 1` are returned, because `security_invoker` applies the caller's RLS policies to the underlying table.

**Why this output appears**: Without `security_invoker`, the view would run with the owner's privileges and RLS policies, potentially showing all tenants' rows. `security_invoker` ensures that the invoking user's context and policies apply.

##### Example 2: MySQL – `SQL SECURITY INVOKER`

```sql
CREATE SQL SECURITY INVOKER VIEW employee_view AS
SELECT employee_id, name, department FROM employees;

-- A user with SELECT on employee_view but not on employees gets an error.
SELECT * FROM employee_view;
```

**Expected output**:

```text
ERROR 1142 (42000): SELECT command denied to user 'report_user'@'%' for table 'employees'
```

**Why this output appears**: With `SQL SECURITY INVOKER`, MySQL checks the calling user's privileges on the underlying `employees` table, not the view creator's. If the user lacks `SELECT` on `employees`, the view query fails.

#### Real-World Cases

- **Multi-tenant SaaS**: Views filter by tenant ID; `security_invoker` ensures each tenant's RLS policies apply.
- **Column-level security**: Views expose only non-sensitive columns (e.g., hiding salary in an `employees` view).
- **Reporting access**: Granting `SELECT` on a security view without granting direct table access.

---

### 4. Complex Views

#### Definitions

**Core Definition**
A complex view is a view whose defining query involves multiple tables (joins), aggregations (`GROUP BY`, `HAVING`), set operations (`UNION`, `INTERSECT`, `EXCEPT`), window functions, or other constructs that make it structurally non-updatable and potentially expensive to execute.

**Technical Definition**
Complex views are read-only by default because they fail the automatic updatability conditions. They are used for abstraction, reporting, and data integration. Their performance depends on the optimizer's ability to flatten the view into the outer query (view merging) or, when flattening is not possible, on materializing the view's result into a temporary table (as in MySQL's `TEMPTABLE` algorithm). Complex views can be nested, creating multi-level abstraction layers that may obscure performance characteristics.

**Beginner-Friendly Explanation**
A complex view is a view that does a lot of work—joining several tables, grouping and summing data, or combining results from different queries. It is great for simplifying reporting, but because it is complex, you usually cannot write to it directly, and it may be slower to query than a simple view.

#### Purposes

- To encapsulate multi-table joins and aggregations under a single, simple name.
- To provide a stable reporting interface while underlying schemas evolve.
- To combine data from multiple sources into a unified view.
- To simplify application queries by pre-joining and pre-aggregating data.

#### Syntax Rules and Structure

**Complex view pattern**:

```sql
CREATE VIEW sales_dashboard AS
SELECT
    c.region,
    p.category,
    DATE_TRUNC('month', o.order_date) AS month,
    SUM(oi.quantity * oi.unit_price) AS revenue,
    COUNT(DISTINCT o.order_id) AS order_count
FROM orders o
JOIN customers c ON c.customer_id = o.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
JOIN products p ON p.product_id = oi.product_id
GROUP BY c.region, p.category, DATE_TRUNC('month', o.order_date);
```

**Component breakdown**

| Component | Complexity factor |
|-----------|-------------------|
| Multiple joins | Increases plan complexity and potential for row explosion. |
| `GROUP BY` + aggregates | Prevents updatability; may require materialization. |
| `DISTINCT` | Adds deduplication cost. |
| Set operations | May require sorting or hashing. |
| Window functions | Adds analytical computation cost. |

**Syntax rules**

- Complex views are read-only unless `INSTEAD OF` triggers are defined.
- The optimizer may merge the view into the outer query (if possible) or materialize it.
- MySQL's `ALGORITHM = MERGE` may still work for some joins; `TEMPTABLE` is used when merging is not possible.
- Complex views can reference other views (nesting), compounding performance considerations.

**Constraints and limitations**

- Cannot be automatically updated.
- May force materialization into unindexed temporary tables, causing performance degradation.
- Predicate pushdown may be limited, especially with `security_barrier`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Complex join and aggregate view

```sql
CREATE VIEW regional_product_revenue AS
SELECT
    c.region,
    p.category,
    SUM(oi.quantity * oi.unit_price) AS revenue
FROM orders o
JOIN customers c ON c.customer_id = o.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
JOIN products p ON p.product_id = oi.product_id
GROUP BY c.region, p.category;

SELECT * FROM regional_product_revenue WHERE region = 'North';
```

**Expected output** (illustrative):

```text
 region | category    | revenue
--------+-------------+----------
 North  | Electronics | 125000.00
 North  | Clothing    |  45000.00
(2 rows)
```

**Why this output appears**: The view joins four tables and aggregates revenue by region and category. The outer `WHERE region = 'North'` may be pushed into the view's query for efficiency, depending on the optimizer.

##### Example 2: MySQL – `EXPLAIN` reveals `TEMPTABLE` materialization

```sql
EXPLAIN SELECT * FROM regional_product_revenue WHERE region = 'North';
```

**Expected output** (excerpt):

```text
+----+-------------+------------+------+...+-----------------+
| id | select_type | table      | type |...| Extra           |
+----+-------------+------------+------+...+-----------------+
|  1 | PRIMARY     | <derived2> | ALL  |...| Using where     |
|  2 | DERIVED     | orders     | ALL  |...| Using temporary |
+----+-------------+------------+------+...+-----------------+
```

**Why this output appears**: The `Using temporary` in the `Extra` column indicates that MySQL materialized the complex view into a temporary table (`TEMPTABLE`) before applying the outer filter. This can be slower than merging, because the temp table has no indexes.

#### Real-World Cases

- **Business intelligence dashboards**: Complex views pre-join and pre-aggregate data for reporting tools.
- **Data integration**: Views combine data from multiple source systems with different schemas.
- **Application simplification**: A single complex view hides a normalized schema from application queries.

---

### 5. Nested Views

#### Definitions

**Core Definition**
A nested view is a view whose defining query references another view, creating a dependency chain in which the outermost view's result is derived through multiple layers of view definitions.

**Technical Definition**
When a query references a nested view, the database's query rewriter expands the view definition, substituting the inner view's query into the outer view's query. This expansion may be repeated for multiple levels. The performance impact depends on whether the optimizer can flatten the nested expansion into a single query plan or whether it must materialize intermediate results. In MySQL, each layer that cannot be merged forces `TEMPTABLE` materialization into an unindexed temporary table, which can cause severe performance degradation in production. In PostgreSQL and SQL Server, views are expanded like macros, so there is generally no inherent performance penalty for nesting unless the layers introduce materialization barriers.

**Beginner-Friendly Explanation**
A nested view is a view built on top of another view. It is like a chain of summaries: the first view summarizes raw data, the second summarizes the first summary, and so on. Nesting can make queries easier to understand, but if you nest too many layers—especially layers that use `GROUP BY`—the database may have to create temporary tables at each level, which can slow things down dramatically.

#### Purposes

- To build layered abstractions where each view represents one logical transformation.
- To reuse intermediate results across multiple downstream views.
- To simplify complex reporting pipelines by naming each stage.
- To isolate changes: modifying an inner view automatically affects dependent outer views.

#### Syntax Rules and Structure

**Nested view pattern**:

```sql
-- Inner view: daily sales.
CREATE VIEW daily_sales AS
SELECT sale_date, SUM(amount) AS daily_total
FROM sales
GROUP BY sale_date;

-- Outer view: monthly rollup of daily sales.
CREATE VIEW monthly_sales AS
SELECT DATE_TRUNC('month', sale_date) AS month,
       SUM(daily_total) AS monthly_total
FROM daily_sales
GROUP BY DATE_TRUNC('month', sale_date);
```

**Component breakdown**

| Layer | Purpose | Materialization risk |
|-------|---------|----------------------|
| `daily_sales` | Aggregates raw sales by day. | `GROUP BY` may force materialization in MySQL. |
| `monthly_sales` | Aggregates daily view by month. | Adds another aggregation layer; may compound `TEMPTABLE` cost. |

**Syntax rules**

- A view can reference any number of other views in its `FROM` clause.
- Dependency order matters: inner views must be created before outer views.
- Dropping an inner view with `RESTRICT` fails if outer views depend on it; use `CASCADE` to drop dependents.
- The query rewriter expands nested views; the optimizer may or may not flatten the expansion.

**Constraints and limitations**

- Deep nesting of aggregating views can force multiple levels of `TEMPTABLE` materialization in MySQL, causing severe performance degradation.
- SQL Server nested views can make execution plans complex and difficult to troubleshoot.
- Changes to inner view definitions automatically propagate to outer views, which can be beneficial or risky.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Nested aggregating views cause `TEMPTABLE` layers

```sql
-- Inner view: aggregates by day.
CREATE VIEW vw_daily AS
SELECT DATE(sale_date) AS d, SUM(amount) AS daily_total
FROM sales GROUP BY DATE(sale_date);

-- Middle view: aggregates by month.
CREATE VIEW vw_monthly AS
SELECT DATE_FORMAT(d, '%Y-%m') AS m, SUM(daily_total) AS monthly_total
FROM vw_daily GROUP BY DATE_FORMAT(d, '%Y-%m');

-- Outer view: aggregates by year.
CREATE VIEW vw_yearly AS
SELECT LEFT(m, 4) AS y, SUM(monthly_total) AS yearly_total
FROM vw_monthly GROUP BY LEFT(m, 4);

EXPLAIN SELECT * FROM vw_yearly;
```

**Expected output** (plan excerpt):

```text
+----+-------------+------------+------+...+-----------------+
| id | select_type | table      | type |...| Extra           |
+----+-------------+------------+------+...+-----------------+
|  1 | PRIMARY     | <derived3> | ALL  |...| Using temporary |
|  3 | DERIVED     | <derived2> | ALL  |...| Using temporary |
|  2 | DERIVED     | sales      | ALL  |...| Using temporary |
+----+-------------+------------+------+...+-----------------+
```

**Why this output appears**: Each aggregating layer forces MySQL to create an unindexed temporary table (`TEMPTABLE`). The outer query scans these temporary tables, and the lack of indexes causes full scans at each level, leading to severe performance degradation on large data sets.

##### Example 2: PostgreSQL – Nested views without inherent penalty

```sql
CREATE VIEW base_view AS
SELECT id, amount FROM transactions WHERE amount > 0;

CREATE VIEW filtered_view AS
SELECT id, amount * 1.1 AS amount_with_tax FROM base_view;

EXPLAIN (COSTS OFF) SELECT * FROM filtered_view WHERE id = 100;
```

**Expected output** (plan excerpt):

```text
 Index Scan using transactions_pkey on transactions
   Index Cond: (id = 100)
   Filter: (amount > 0)
```

**Why this output appears**: PostgreSQL expands nested views as rewrite rules and the optimizer flattens the expansion, pushing the `id = 100` predicate directly into the base table's index scan. There is no intermediate materialization penalty.

#### Real-World Cases

- **Layered reporting**: Daily → monthly → yearly sales views.
- **Data quality pipelines**: Raw → cleaned → enriched → aggregated views.
- **Legacy abstraction**: New views layered over old views during schema migration.

---

### 6. Materialized Views (Where Supported)

#### Definitions

**Core Definition**
A materialized view is a database object that stores the result of a query physically on disk, like a table, and must be refreshed to reflect changes in the underlying base tables.

**Technical Definition**
Materialized views persist their result set, trading storage and refresh overhead for query performance. Oracle supports `REFRESH ON COMMIT` (refresh when a transaction commits on a master table), `REFRESH ON DEMAND` (manual refresh via `DBMS_MVIEW` procedures), and `REFRESH ON STATEMENT` (refresh on every DML statement on base tables). PostgreSQL supports `CREATE MATERIALIZED VIEW` with `REFRESH MATERIALIZED VIEW`; `REFRESH MATERIALIZED VIEW CONCURRENTLY` (PostgreSQL 9.4+) allows reads during refresh but requires a unique index on the materialized view. SQL Server's indexed views are materialized views: they require `WITH SCHEMABINDING` and a unique clustered index, after which the optimizer can use them transparently.

**Beginner-Friendly Explanation**
A materialized view is like a photocopy of a query's result. Because the data is stored on disk, querying it is fast—like reading the photocopy instead of recalculating the result from scratch. But when the underlying data changes, the photocopy becomes stale, and you have to refresh it. Different databases offer different refresh strategies: some refresh automatically when data changes, others require you to run a refresh command.

#### Purposes

- To accelerate expensive queries by precomputing and storing results.
- To support dashboards and reports that query aggregated data frequently.
- To offload computation from base tables during peak query periods.
- To provide a snapshot of data at a specific point in time.

#### Syntax Rules and Structure

**PostgreSQL**:

```sql
CREATE MATERIALIZED VIEW sales_summary AS
SELECT region, SUM(amount) AS total
FROM sales GROUP BY region
WITH DATA;

REFRESH MATERIALIZED VIEW sales_summary;

-- Concurrent refresh requires a unique index.
CREATE UNIQUE INDEX ON sales_summary (region);
REFRESH MATERIALIZED VIEW CONCURRENTLY sales_summary;
```

**Oracle**:

```sql
CREATE MATERIALIZED VIEW sales_mv
BUILD IMMEDIATE
REFRESH FAST ON COMMIT
AS SELECT region, SUM(amount) AS total
FROM sales GROUP BY region;
```

**SQL Server**:

```sql
CREATE VIEW dbo.SalesSummary
WITH SCHEMABINDING
AS
SELECT region, SUM(amount) AS total, COUNT_BIG(*) AS cnt
FROM dbo.Sales
GROUP BY region;
GO
CREATE UNIQUE CLUSTERED INDEX IX_SalesSummary
ON dbo.SalesSummary (region);
```

**Component breakdown**

| Component | Engine | Description |
|-----------|--------|-------------|
| `WITH DATA` / `WITH NO DATA` | PostgreSQL | Populate immediately or defer population. |
| `REFRESH MATERIALIZED VIEW` | PostgreSQL | Full refresh (locks reads). |
| `REFRESH MATERIALIZED VIEW CONCURRENTLY` | PostgreSQL 9.4+ | Non-blocking refresh; requires unique index. |
| `REFRESH FAST ON COMMIT` | Oracle | Incremental refresh on transaction commit. |
| `REFRESH COMPLETE ON DEMAND` | Oracle | Full refresh only when manually invoked. |
| `WITH SCHEMABINDING` + unique clustered index | SQL Server | Required to create an indexed (materialized) view. |

**Syntax rules**

- PostgreSQL: `CONCURRENTLY` requires a unique index on the materialized view and cannot be used with `WITH NO DATA`.
- Oracle: `ON COMMIT` cannot be combined with `ON DEMAND` or `START WITH`/`NEXT`; it increases commit time because refresh happens during commit.
- SQL Server: the view must be created with `SCHEMABINDING`, and the first index must be a unique clustered index.
- MySQL supports materialized views only on MySQL HeatWave; regular MySQL views are virtual.

**Constraints and limitations**

- Materialized views consume storage proportional to their result size.
- Refreshing can be expensive; `ON COMMIT` refresh can increase transaction commit latency.
- PostgreSQL's non-concurrent `REFRESH` locks the materialized view against reads; `CONCURRENTLY` avoids locking but requires a unique index and may be slower.
- SQL Server indexed views can degrade DML performance on base tables because the view's indexes must be maintained on every modification.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Materialized view with concurrent refresh

```sql
-- Step 1: Create a materialized view.
CREATE MATERIALIZED VIEW monthly_revenue AS
SELECT DATE_TRUNC('month', sale_date) AS month,
       SUM(amount) AS revenue
FROM sales
GROUP BY DATE_TRUNC('month', sale_date)
WITH DATA;

-- Step 2: Create a unique index for concurrent refresh.
CREATE UNIQUE INDEX ON monthly_revenue (month);

-- Step 3: Refresh without blocking reads.
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_revenue;

-- Step 4: Query the materialized view.
SELECT * FROM monthly_revenue ORDER BY month;
```

**Expected output**:

```text
   month    | revenue
------------+---------
 2025-01-01 | 125000.00
 2025-02-01 | 138000.00
 2025-03-01 | 142000.00
(3 rows)
```

**Why this output appears**: The materialized view stores the aggregated monthly revenue. `CONCURRENTLY` refreshes it while allowing queries to continue reading the old data, and the unique index on `month` is required for this mode.

##### Example 2: Oracle – Fast refresh on commit

```sql
CREATE MATERIALIZED VIEW sales_mv
BUILD IMMEDIATE
REFRESH FAST ON COMMIT
AS SELECT region, SUM(amount) AS total
FROM sales
GROUP BY region;
```

**Expected output**:

```text
Materialized view created.
```

**Why this output appears**: Oracle creates the materialized view and populates it immediately (`BUILD IMMEDIATE`). `REFRESH FAST ON COMMIT` means that whenever a transaction commits a change to the `sales` table, Oracle incrementally refreshes the materialized view as part of the commit process.

##### Example 3: SQL Server – Indexed view

```sql
CREATE VIEW dbo.ProductSales
WITH SCHEMABINDING
AS
SELECT ProductID, SUM(LineTotal) AS TotalSales, COUNT_BIG(*) AS OrderCount
FROM dbo.SalesOrderDetail
GROUP BY ProductID;
GO
CREATE UNIQUE CLUSTERED INDEX IX_ProductSales
ON dbo.ProductSales (ProductID);
GO
```

**Expected output**:

```text
Command(s) completed successfully.
```

**Why this output appears**: The view is created with `SCHEMABINDING`, and the unique clustered index physically stores the view's result set. SQL Server's optimizer can use this indexed view to speed up queries even when the query does not reference the view directly.

#### Real-World Cases

- **Dashboards**: Materialized views precompute aggregations for fast dashboard loading.
- **Data warehousing**: Materialized views store summarized data for OLAP queries.
- **Reporting snapshots**: Materialized views capture data at a specific point in time for audit or comparison.
- **Query acceleration**: Indexed views in SQL Server transparently speed up expensive aggregation queries.

---

## References

- PostgreSQL Documentation – CREATE VIEW (Updatable Views) – https://www.postgresql.org/docs/current/sql-createview.html
- PostgreSQL Documentation – Rules and Privileges (Security Barrier Views) – https://www.postgresql.org/docs/current/rules-privileges.html
- PostgreSQL Documentation – CREATE MATERIALIZED VIEW – https://www.postgresql.org/docs/current/sql-creatematerializedview.html
- PostgreSQL Documentation – REFRESH MATERIALIZED VIEW – https://www.postgresql.org/docs/current/sql-refreshmaterializedview.html
- MySQL 8.0 Reference Manual – Restrictions on Views – https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/view-restrictions.html
- MySQL 8.0 Reference Manual – Updatable and Insertable Views – https://dev.mysql.com/doc/refman/8.0/en/view-updatability.html
- MySQL 8.0 Reference Manual – View Processing Algorithms – https://dev.mysql.com/doc/refman/8.0/en/view-algorithms.html
- Microsoft Learn – View Design Patterns (INSTEAD OF Triggers) – https://learn.microsoft.com/en-us/previous-versions/dd129495(v=vs.85)
- Microsoft Learn – Create Indexed Views – https://learn.microsoft.com/en-us/sql/relational-databases/views/create-indexed-views
- Oracle Database SQL Language Reference – CREATE MATERIALIZED VIEW – https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/CREATE-MATERIALIZED-VIEW.html
- Cybertec – View Permissions and Row-Level Security in PostgreSQL – https://www.cybertec-postgresql.com/en/view-permissions-and-row-level-security-in-postgresql/
- SQL Engineering Handbook – View Performance (MySQL MERGE vs TEMPTABLE) – https://github.com/theammarngp-makes/SQL-Engineering-Handbook/blob/main/14_VIEWS/07_VIEW_PERFORMANCE.md