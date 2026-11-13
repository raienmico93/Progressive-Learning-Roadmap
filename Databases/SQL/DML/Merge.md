# Comprehensive Programming Cheat Sheet: SQL MERGE Operations

## Topic Overview

### Definitions

**Core Definition**
The `MERGE` statement is a single, set-based SQL operation that performs `INSERT`, `UPDATE`, and `DELETE` actions on a target table based on the results of a join with a source table, enabling complex synchronization and upsert logic without multiple procedural statements.

**Technical Definition**
`MERGE` compares a target table with a data source (a table, view, or subquery) using a specified join condition. For each candidate row produced by the join, it evaluates a series of `WHEN` clauses—`WHEN MATCHED`, `WHEN NOT MATCHED [BY TARGET]`, and `WHEN NOT MATCHED BY SOURCE`—and executes the first clause whose condition evaluates to true. The statement was introduced in the SQL:2003 standard and is supported with varying syntax and semantics by SQL Server, Oracle, PostgreSQL (since version 15), and other enterprise databases. `MERGE` is widely used for ETL synchronization, upserts, and conditional data maintenance, but it carries well-documented concurrency and correctness pitfalls that require careful implementation, particularly in high-concurrency OLTP environments.

**Beginner-Friendly Explanation**
`MERGE` is like a smart traffic controller for your data. You give it two tables: a “target” table (where data should end up) and a “source” table (where new or updated data comes from). `MERGE` compares them row by row. If a row exists in both, it can update the target. If a row exists only in the source, it can insert it into the target. If a row exists only in the target, it can delete it. All of this happens in one statement, making it a powerful tool for keeping two tables in sync—but also a tool that requires care, because a small mistake in the join condition or a missing concurrency safeguard can cause data loss or race conditions.

### Key Characteristics

- **Set-based synchronization**: `MERGE` processes rows as a set, not one at a time, making it efficient for batch synchronization from staging tables into production tables.
- **Single-statement atomicity**: The entire `MERGE` statement is atomic; either all actions are applied or none are (subject to transaction rollback).
- **First-match semantics**: For each candidate row, the `WHEN` clauses are evaluated in the order specified, and **only the first true clause is executed**. No more than one `WHEN` clause runs per row.
- **Vendor-specific features and caveats**: SQL Server supports `WHEN NOT MATCHED BY SOURCE`, `TOP`, `OUTPUT`, and table hints like `HOLDLOCK`; PostgreSQL (15+) supports `RETURNING`, `DO NOTHING`, and `WHEN NOT MATCHED BY SOURCE`; Oracle has its own `MERGE` syntax. MySQL does not support `MERGE` (it uses `ON DUPLICATE KEY UPDATE` instead).
- **Concurrency-sensitive**: Under concurrent execution, `MERGE` can suffer from race conditions, primary key violations, and deadlocks unless appropriate isolation levels or locking hints (e.g., `HOLDLOCK` / `SERIALIZABLE`) are used.

### Prerequisites

- A running RDBMS that supports `MERGE` (SQL Server, Oracle, PostgreSQL 15+, etc.).
- `INSERT`, `UPDATE`, and `DELETE` privileges on the target table, as required by the actions specified in the `WHEN` clauses; `SELECT` privilege on the source and any columns referenced in conditions.
- A well-defined join condition, preferably on a primary key or unique business key, to ensure deterministic matching.
- Awareness of the concurrency implications and, in high-concurrency environments, a locking or isolation strategy (e.g., `HOLDLOCK` in SQL Server).

### Related Programming Areas

- Data Manipulation Language (DML) – `INSERT`, `UPDATE`, `DELETE`.
- ETL (Extract, Transform, Load) – staging-table synchronization, slowly changing dimensions, and data warehouse loading.
- Concurrency control – transaction isolation levels, locking hints, and deadlock avoidance.
- Upsert patterns – `MERGE` as an alternative to `INSERT ... ON CONFLICT` (PostgreSQL) or `ON DUPLICATE KEY UPDATE` (MySQL).

---

## Core Concepts and Features

### 1. `MERGE`

#### Definitions

**Core Definition**
`MERGE` is the SQL statement that conditionally inserts, updates, or deletes rows of a target table based on a join with a source table.

**Technical Definition**
The SQL Server syntax is `[ WITH <common_table_expression> [,...n] ] MERGE [ TOP ( expression ) [ PERCENT ] ] [ INTO ] <target_table> [ WITH ( <merge_hint> ) ] [ [ AS ] table_alias ] USING <table_source> [ [ AS ] table_alias ] ON <merge_search_condition> [ WHEN MATCHED [ AND <clause_search_condition> ] THEN <merge_matched> ] [ ...n ] [ WHEN NOT MATCHED [ BY TARGET ] [ AND <clause_search_condition> ] THEN <merge_not_matched> ] [ WHEN NOT MATCHED BY SOURCE [ AND <clause_search_condition> ] THEN <merge_matched> ] [ ...n ] [ <output_clause> ] [ OPTION ( <query_hint> [ ,...n ] ) ] ;`. The PostgreSQL 15+ syntax is `[ WITH with_query [, ...] ] MERGE INTO [ ONLY ] target_table_name [ * ] [ [ AS ] target_alias ] USING data_source ON join_condition when_clause [...]`, where `when_clause` can be `WHEN MATCHED`, `WHEN NOT MATCHED [ BY TARGET ]`, or `WHEN NOT MATCHED BY SOURCE`, each followed by `THEN` and an action (`UPDATE`, `INSERT`, `DELETE`, or `DO NOTHING`).

**Beginner-Friendly Explanation**
`MERGE` is the master command for synchronizing two tables. It compares a source and a target and decides, row by row, whether to insert, update, or delete data in the target.

#### Purposes

- To synchronize a target table with a source table in a single set-based statement.
- To implement upsert logic (insert new rows, update existing ones) without multiple round-trips.
- To perform conditional data maintenance (e.g., deleting rows in the target that no longer exist in the source).
- To load data from a staging table into a production table while applying business rules.

#### Syntax Rules and Structure

**General syntax (SQL Server, simplified):**

```sql
MERGE INTO target_table AS T
USING source_table AS S
ON T.key = S.key
WHEN MATCHED THEN
    UPDATE SET T.col = S.col
WHEN NOT MATCHED BY TARGET THEN
    INSERT (key, col) VALUES (S.key, S.col)
WHEN NOT MATCHED BY SOURCE THEN
    DELETE;
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `MERGE INTO target` | The table to be modified. |
| `USING source` | The table, view, or subquery providing the data. |
| `ON join_condition` | The condition that matches rows between source and target. |
| `WHEN MATCHED` | Action when a source row matches a target row. |
| `WHEN NOT MATCHED BY TARGET` | Action when a source row has no match in the target (i.e., a new row to insert). |
| `WHEN NOT MATCHED BY SOURCE` | Action when a target row has no match in the source (i.e., a row to delete or update). |

**Syntax rules:**

- The `MERGE` statement **must be terminated with a semicolon** in SQL Server; otherwise, a syntax error occurs.
- The `ON` clause should contain only the join conditions that determine matching; additional filtering should go into `WHEN ... AND` clauses.
- At most two `WHEN MATCHED` clauses are allowed in SQL Server; if two are specified, the first must include an `AND <search_condition>` clause.

**Constraints and limitations:**

- `MERGE` is not supported in MySQL (use `ON DUPLICATE KEY UPDATE` instead) and was not available in PostgreSQL before version 15.
- In PostgreSQL, `MERGE` is not supported if the target table is a materialized view, a foreign table, or has rules defined on it.
- The target table cannot have a `WITH` clause that filters rows in a way that causes incorrect results; Microsoft explicitly warns against using CTEs to filter source or target rows in the `ON` clause.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Basic MERGE for table synchronization

```sql
-- Step 1: Create target and source tables.
CREATE TABLE dbo.Products (
    ProductID int PRIMARY KEY,
    ProductName nvarchar(100) NOT NULL,
    Price decimal(10,2) NOT NULL
);

CREATE TABLE dbo.ProductsStage (
    ProductID int PRIMARY KEY,
    ProductName nvarchar(100) NOT NULL,
    Price decimal(10,2) NOT NULL
);

INSERT INTO dbo.Products VALUES (1, 'Widget', 10.00);
INSERT INTO dbo.ProductsStage VALUES (1, 'Widget Pro', 12.00), (2, 'Gadget', 20.00);

-- Step 2: Merge the staged data into the target.
MERGE dbo.Products AS T
USING dbo.ProductsStage AS S
ON T.ProductID = S.ProductID
WHEN MATCHED THEN
    UPDATE SET T.ProductName = S.ProductName, T.Price = S.Price
WHEN NOT MATCHED BY TARGET THEN
    INSERT (ProductID, ProductName, Price) VALUES (S.ProductID, S.ProductName, S.Price);

-- Step 3: Verify.
SELECT * FROM dbo.Products ORDER BY ProductID;
```

**Expected output:**

```text
ProductID  ProductName  Price
---------  -----------  -----
1          Widget Pro   12.00
2          Gadget       20.00
```

**Why this output appears**: For `ProductID = 1`, a match is found, so the existing row is updated with the new name and price. For `ProductID = 2`, no match is found, so a new row is inserted. No `WHEN NOT MATCHED BY SOURCE` clause is present, so no rows are deleted.

#### Real-World Cases

- **Product catalog refresh**: A nightly job merges a staging table of updated product information into the live `Products` table, updating changed products and inserting new ones.
- **Dimension synchronization**: A data warehouse dimension table is synchronized with a source system using `MERGE`, keeping the warehouse consistent with operational data.

---

### 2. `WHEN MATCHED`

#### Definitions

**Core Definition**
The `WHEN MATCHED` clause specifies the action to take when a source row finds a matching row in the target table based on the `ON` join condition.

**Technical Definition**
`WHEN MATCHED [ AND <clause_search_condition> ] THEN <merge_matched>` where `<merge_matched>` is either `UPDATE SET <set_clause>` or `DELETE`. SQL Server allows at most two `WHEN MATCHED` clauses; if two are specified, the first must include an `AND <search_condition>` clause, and for any given row, the second clause is applied only if the first is not. PostgreSQL allows `UPDATE`, `DELETE`, or `DO NOTHING` in a `WHEN MATCHED` clause.

**Beginner-Friendly Explanation**
`WHEN MATCHED` is the “what to do when the row already exists” rule. You can update the existing row with new values, delete it, or (in PostgreSQL) skip it.

#### Purposes

- To update existing rows with values from the source table.
- To conditionally delete matching rows (e.g., when a flag in the source indicates removal).
- To skip rows that match but do not meet additional criteria (using `AND` conditions or `DO NOTHING`).

#### Syntax Rules and Structure

**General syntax:**

```sql
WHEN MATCHED [ AND <condition> ] THEN
    { UPDATE SET column = expression [, ...] | DELETE | DO NOTHING }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `WHEN MATCHED` | The clause for rows that exist in both source and target. |
| `AND <condition>` | Optional additional filter; the action applies only if this condition is also true. |
| `UPDATE SET` | Updates the matched row. |
| `DELETE` | Deletes the matched row. |
| `DO NOTHING` | Skips the matched row (PostgreSQL). |

**Syntax rules:**

- In SQL Server, multiple `WHEN MATCHED` clauses are allowed (maximum two), but the first must have an `AND` condition.
- The `UPDATE` action cannot update the columns used in the `ON` join condition (in most implementations).

**Constraints and limitations:**

- Updating a column that is part of a unique constraint can cause a constraint violation if the new value duplicates another row.
- Using `DELETE` in `WHEN MATCHED` without a careful `AND` condition can delete more rows than intended.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Conditional UPDATE in WHEN MATCHED

```sql
-- Step 1: Create tables.
CREATE TABLE dbo.Inventory (ItemID int PRIMARY KEY, Quantity int, LastUpdated datetime2);
CREATE TABLE dbo.InventoryStage (ItemID int PRIMARY KEY, Quantity int);

INSERT INTO dbo.Inventory VALUES (1, 100, '2025-01-01');
INSERT INTO dbo.InventoryStage VALUES (1, 150), (2, 200);

-- Step 2: Merge, updating only when the quantity actually changes.
MERGE dbo.Inventory AS T
USING dbo.InventoryStage AS S
ON T.ItemID = S.ItemID
WHEN MATCHED AND T.Quantity <> S.Quantity THEN
    UPDATE SET T.Quantity = S.Quantity, T.LastUpdated = SYSUTCDATETIME()
WHEN NOT MATCHED BY TARGET THEN
    INSERT (ItemID, Quantity) VALUES (S.ItemID, S.Quantity);

-- Step 3: Verify.
SELECT * FROM dbo.Inventory ORDER BY ItemID;
```

**Expected output:**

```text
ItemID  Quantity  LastUpdated
------  --------  -----------------------
1       150       2026-11-13 10:00:00.000
2       200       NULL
```

**Why this output appears**: For `ItemID = 1`, the quantity changes from 100 to 150, so the `WHEN MATCHED AND T.Quantity <> S.Quantity` condition is true, and the row is updated. For `ItemID = 2`, no match exists, so a new row is inserted with a `NULL` timestamp (since the insert does not specify `LastUpdated`).

#### Real-World Cases

- **Price updates**: Only update products whose price has changed, avoiding unnecessary writes and trigger firings.
- **Soft deletes**: Mark matched rows as deleted by setting a `deleted_at` timestamp in the `UPDATE` clause.

---

### 3. `WHEN NOT MATCHED`

#### Definitions

**Core Definition**
The `WHEN NOT MATCHED` clause (also written `WHEN NOT MATCHED BY TARGET` in SQL Server) specifies the action to take when a source row has no corresponding row in the target table.

**Technical Definition**
`WHEN NOT MATCHED [ BY TARGET ] [ AND <clause_search_condition> ] THEN <merge_not_matched>` where `<merge_not_matched>` is an `INSERT` action: `INSERT [( column_list )] { VALUES ( values_list ) | DEFAULT VALUES }`. In PostgreSQL, the clause can also be `DO NOTHING`. This clause effectively handles new rows that exist in the source but not in the target.

**Beginner-Friendly Explanation**
`WHEN NOT MATCHED` is the “what to do with brand-new rows” rule. When a row exists in the source but not in the target, this clause inserts it into the target.

#### Purposes

- To insert new rows from the source that do not yet exist in the target.
- To conditionally insert rows only when they meet additional criteria (`AND` condition).
- To skip certain new rows using `DO NOTHING` (PostgreSQL).

#### Syntax Rules and Structure

**General syntax:**

```sql
WHEN NOT MATCHED [ BY TARGET ] [ AND <condition> ] THEN
    INSERT [( column_list )] { VALUES ( values_list ) | DEFAULT VALUES }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `WHEN NOT MATCHED` | The clause for source rows without a target match. |
| `BY TARGET` | Optional keyword (SQL Server) clarifying that the clause applies to source rows missing in the target. |
| `INSERT` | Inserts the new row. |

**Syntax rules:**

- In SQL Server, `WHEN NOT MATCHED` is equivalent to `WHEN NOT MATCHED BY TARGET`.
- The `INSERT` column list must match the values list in number and type.
- `DEFAULT VALUES` inserts a row with all defaults (SQL Server, PostgreSQL).

**Constraints and limitations:**

- Inserting a row that violates a unique constraint (e.g., a duplicate key not detected by the join) will cause the entire `MERGE` to fail.
- In high-concurrency scenarios, two concurrent `MERGE` statements can both see a row as “not matched” and both attempt to insert it, leading to a primary-key violation unless `HOLDLOCK` or a similar mechanism is used.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL 15 – WHEN NOT MATCHED with DO NOTHING

```sql
-- Step 1: Create tables.
CREATE TABLE target (id int PRIMARY KEY, val text);
CREATE TABLE source (id int, val text);
INSERT INTO target VALUES (1, 'existing');
INSERT INTO source VALUES (1, 'update'), (2, 'new');

-- Step 2: Merge, skipping new rows that already exist.
MERGE INTO target AS t
USING source AS s
ON t.id = s.id
WHEN MATCHED THEN
    UPDATE SET val = s.val
WHEN NOT MATCHED THEN
    INSERT (id, val) VALUES (s.id, s.val);

-- Step 3: Verify.
SELECT * FROM target ORDER BY id;
```

**Expected output:**

```text
 id |   val
----+----------
  1 | update
  2 | new
(2 rows)
```

**Why this output appears**: For `id = 1`, a match is found, so the `val` is updated to `'update'`. For `id = 2`, no match exists, so a new row is inserted with the source value `'new'`.

#### Real-World Cases

- **New customer onboarding**: New customers from a source system are inserted into the target CRM table.
- **Event ingestion**: New events from a streaming source are inserted into a target fact table while existing events are updated.

---

### 4. `WHEN NOT MATCHED BY SOURCE` (SQL Server)

#### Definitions

**Core Definition**
The `WHEN NOT MATCHED BY SOURCE` clause specifies the action to take when a target row has no corresponding row in the source table.

**Technical Definition**
`WHEN NOT MATCHED BY SOURCE [ AND <clause_search_condition> ] THEN <merge_matched>` where `<merge_matched>` is `UPDATE SET <set_clause>` or `DELETE`. This clause is the mirror image of `WHEN NOT MATCHED BY TARGET`: it handles rows that exist in the target but not in the source. It is particularly useful for synchronization scenarios where rows deleted from the source should also be removed (or marked as deleted) in the target.

**Beginner-Friendly Explanation**
`WHEN NOT MATCHED BY SOURCE` is the “what to do with rows that no longer exist in the source” rule. If a row is in the target but not in the source, this clause can delete it or update it (for example, to mark it as inactive).

#### Purposes

- To delete target rows that no longer have a counterpart in the source (full synchronization).
- To mark orphaned target rows as inactive or archived rather than deleting them.
- To maintain referential consistency between a source of truth and a derived table.

#### Syntax Rules and Structure

**General syntax:**

```sql
WHEN NOT MATCHED BY SOURCE [ AND <condition> ] THEN
    { UPDATE SET column = expression [, ...] | DELETE }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `WHEN NOT MATCHED BY SOURCE` | The clause for target rows without a source match. |
| `AND <condition>` | Optional additional filter. |
| `DELETE` | Deletes the orphaned target row. |
| `UPDATE SET` | Updates the orphaned target row (e.g., to set a status flag). |

**Syntax rules:**

- This clause is available in SQL Server and PostgreSQL 15+ (with `WHEN NOT MATCHED BY SOURCE`); it is not available in all `MERGE` implementations.
- Use `DELETE` only when you are certain that rows absent from the source should be removed from the target.

**Constraints and limitations:**

- A `DELETE` in this clause can remove large numbers of rows if the source is incomplete or filtered; use `AND` conditions and test carefully.
- In PostgreSQL, the `WHEN NOT MATCHED BY SOURCE` clause can also use `DO NOTHING`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Full synchronization with WHEN NOT MATCHED BY SOURCE

```sql
-- Step 1: Create tables.
CREATE TABLE dbo.Customers (CustomerID int PRIMARY KEY, Name nvarchar(100), Active bit);
CREATE TABLE dbo.CustomersStage (CustomerID int PRIMARY KEY, Name nvarchar(100));

INSERT INTO dbo.Customers VALUES (1, 'Acme', 1), (2, 'Globex', 1), (3, 'Initech', 1);
INSERT INTO dbo.CustomersStage VALUES (1, 'Acme Corp'), (2, 'Globex');

-- Step 2: Merge, deleting customers no longer in the source.
MERGE dbo.Customers AS T
USING dbo.CustomersStage AS S
ON T.CustomerID = S.CustomerID
WHEN MATCHED THEN
    UPDATE SET T.Name = S.Name
WHEN NOT MATCHED BY TARGET THEN
    INSERT (CustomerID, Name) VALUES (S.CustomerID, S.Name)
WHEN NOT MATCHED BY SOURCE THEN
    DELETE;

-- Step 3: Verify.
SELECT * FROM dbo.Customers ORDER BY CustomerID;
```

**Expected output:**

```text
CustomerID  Name
----------  ----------
1           Acme Corp
2           Globex
```

**Why this output appears**: Customer 3 (`Initech`) exists in the target but not in the source, so the `WHEN NOT MATCHED BY SOURCE THEN DELETE` clause removes it. Customers 1 and 2 are updated from the source.

#### Real-World Cases

- **Authoritative source synchronization**: A target table is kept as an exact mirror of a source system; rows deleted from the source are removed from the target.
- **Soft-delete synchronization**: Instead of deleting, the `UPDATE` action marks orphaned rows as `Active = 0`, preserving history.

---

### 5. Upsert Patterns

#### Definitions

**Core Definition**
An upsert pattern is a technique that inserts a row if it does not exist and updates it if it does, often implemented with `MERGE` in SQL Server and Oracle.

**Technical Definition**
`MERGE` is a natural fit for upserts: `WHEN MATCHED THEN UPDATE` handles the “update” case, and `WHEN NOT MATCHED THEN INSERT` handles the “insert” case. In SQL Server, `MERGE` is the primary upsert mechanism; in PostgreSQL 15+, `MERGE` can also be used, though `INSERT ... ON CONFLICT DO UPDATE` is often more concise for single-row upserts. The choice between `MERGE` and explicit `UPDATE`/`INSERT` statements depends on concurrency requirements: Microsoft notes that when high concurrency is expected, separate `INSERT`, `UPDATE`, and `DELETE` logic can perform better with less blocking than `MERGE`.

**Beginner-Friendly Explanation**
An upsert is a “save or update” operation. `MERGE` lets you write it as a single statement: update the row if it exists, insert it if it doesn't.

#### Purposes

- To implement idempotent “save” operations that work whether or not the record already exists.
- To synchronize data from external sources without checking existence beforehand.
- To reduce round-trips and procedural code for common insert-or-update logic.

#### Syntax Rules and Structure

**SQL Server upsert with MERGE:**

```sql
MERGE dbo.Target AS T
USING dbo.Source AS S
ON T.Key = S.Key
WHEN MATCHED THEN UPDATE SET T.Value = S.Value
WHEN NOT MATCHED THEN INSERT (Key, Value) VALUES (S.Key, S.Value);
```

**PostgreSQL 15+ upsert with MERGE:**

```sql
MERGE INTO target AS t
USING source AS s
ON t.id = s.id
WHEN MATCHED THEN UPDATE SET val = s.val
WHEN NOT MATCHED THEN INSERT (id, val) VALUES (s.id, s.val);
```

**Syntax rules:**

- The `ON` clause must uniquely identify matching rows; a non-unique join can cause unpredictable results or errors.
- In high-concurrency OLTP paths, consider explicit `UPDATE`-then-`INSERT` logic or `HOLDLOCK` with `MERGE` (see concurrency).

**Constraints and limitations:**

- `MERGE` upserts can raise primary-key violations under concurrent execution without `HOLDLOCK` (SQL Server).
- `MERGE` is more verbose than `ON CONFLICT` (PostgreSQL) or `ON DUPLICATE KEY UPDATE` (MySQL).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – MERGE upsert with OUTPUT

```sql
-- Step 1: Create tables.
CREATE TABLE dbo.Counters (Key1 nvarchar(50) PRIMARY KEY, Value int);
CREATE TABLE dbo.CountersStage (Key1 nvarchar(50) PRIMARY KEY, Value int);
INSERT INTO dbo.CountersStage VALUES ('hits', 1);

-- Step 2: Upsert and capture the action taken.
MERGE dbo.Counters AS T
USING dbo.CountersStage AS S
ON T.Key1 = S.Key1
WHEN MATCHED THEN
    UPDATE SET T.Value = T.Value + S.Value
WHEN NOT MATCHED THEN
    INSERT (Key1, Value) VALUES (S.Key1, S.Value)
OUTPUT $action, INSERTED.Key1, INSERTED.Value;
```

**Expected output:**

```text
$action  Key1  Value
-------  ----  -----
INSERT   hits  1
```

**Why this output appears**: On the first run, the key `'hits'` does not exist, so `MERGE` inserts it, and the `OUTPUT` clause reports `INSERT` with the new value. On subsequent runs, it would report `UPDATE` and the incremented value.

#### Real-World Cases

- **Counter increments**: A `MERGE` upsert atomically increments a counter, inserting the counter if it does not exist.
- **Configuration management**: A settings table is upserted from a configuration source, ensuring all keys exist with the latest values.

---

### 6. Synchronization

#### Definitions

**Core Definition**
Synchronization with `MERGE` is the process of making a target table match a source table by applying inserts, updates, and deletes based on the differences between the two.

**Technical Definition**
`MERGE` is designed for set-based synchronization: the `ON` join condition identifies matching rows, `WHEN MATCHED` handles updates (and possibly deletes), `WHEN NOT MATCHED BY TARGET` handles inserts, and `WHEN NOT MATCHED BY SOURCE` handles deletes (or updates) of target-only rows. This makes it a natural fit for ETL staging-table loads, slowly changing dimensions, and mirroring external data sources.

**Beginner-Friendly Explanation**
Synchronization means keeping two tables in agreement. `MERGE` compares them and applies whatever changes are needed so that the target reflects the source.

#### Purposes

- To load new and changed data from a staging table into a production table.
- To keep a replica table aligned with a source of truth.
- To apply slowly changing dimension (SCD) logic in a data warehouse.

#### Syntax Rules and Structure

A full synchronization `MERGE` typically includes all three `WHEN` clauses:

```sql
MERGE INTO target AS T
USING source AS S
ON T.key = S.key
WHEN MATCHED AND (condition) THEN UPDATE SET ...
WHEN NOT MATCHED BY TARGET THEN INSERT ...
WHEN NOT MATCHED BY SOURCE THEN DELETE;
```

**Syntax rules:**

- Ensure the join key is unique on both sides; duplicates can cause `MERGE` to raise error 8672 (“The MERGE statement attempted to UPDATE or DELETE the same row more than once”).
- Filter the source before the merge (using a CTE or subquery) to include only changed or relevant rows.
- Test the `WHEN NOT MATCHED BY SOURCE` clause carefully; an incomplete source can delete large amounts of target data.

**Constraints and limitations:**

- Large synchronization merges can be resource-intensive; batching and indexing the join columns are recommended.
- In SQL Server, creating a clustered index on the join columns of the target table and disabling non-unique nonclustered indexes during the load can improve performance.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Full synchronization with conditional update

```sql
-- Step 1: Create tables.
CREATE TABLE dbo.DimProduct (ProductID int PRIMARY KEY, Name nvarchar(100), Price decimal(10,2), IsActive bit);
CREATE TABLE dbo.DimProductStage (ProductID int PRIMARY KEY, Name nvarchar(100), Price decimal(10,2));

INSERT INTO dbo.DimProduct VALUES (1, 'Widget', 10.00, 1), (2, 'Gadget', 20.00, 1);
INSERT INTO dbo.DimProductStage VALUES (1, 'Widget Pro', 12.00), (3, 'NewItem', 30.00);

-- Step 2: Synchronize.
MERGE dbo.DimProduct AS T
USING dbo.DimProductStage AS S
ON T.ProductID = S.ProductID
WHEN MATCHED AND (T.Name <> S.Name OR T.Price <> S.Price) THEN
    UPDATE SET T.Name = S.Name, T.Price = S.Price
WHEN NOT MATCHED BY TARGET THEN
    INSERT (ProductID, Name, Price, IsActive) VALUES (S.ProductID, S.Name, S.Price, 1)
WHEN NOT MATCHED BY SOURCE THEN
    UPDATE SET T.IsActive = 0;

-- Step 3: Verify.
SELECT * FROM dbo.DimProduct ORDER BY ProductID;
```

**Expected output:**

```text
ProductID  Name       Price  IsActive
---------  ---------  -----  --------
1          Widget Pro 12.00  1
2          Gadget     20.00  0
3          NewItem    30.00  1
```

**Why this output appears**: Product 1 is updated because its name and price changed. Product 2 is not in the source, so it is soft-deleted (`IsActive = 0`). Product 3 is new and is inserted with `IsActive = 1`.

#### Real-World Cases

- **Product catalog refresh**: A nightly `MERGE` synchronizes a staging catalog with the live catalog, updating changed products, inserting new ones, and deactivating discontinued ones.
- **Data warehouse dimension load**: A daily ETL job merges source-system data into a dimension table, applying SCD Type 1 (overwrite) or Type 2 (history) logic depending on the `WHEN` clauses.

---

### 7. Performance

#### Definitions

**Core Definition**
`MERGE` performance refers to the execution efficiency of the statement, influenced by indexing, join strategy, data volume, and the number of `WHEN` clauses executed.

**Technical Definition**
`MERGE` performance is affected by: (1) **indexing**—the join columns on both source and target should be indexed; a clustered index on the target's join columns is recommended; (2) **join strategy**—the optimizer chooses among nested loops, merge join, and hash join, with merge join being efficient when source and target are similarly sized and indexed; (3) **row volume**—large merges benefit from batching and from disabling non-unique indexes during the load; and (4) **statement complexity**—multiple `WHEN` clauses and complex `AND` conditions add evaluation overhead. Microsoft recommends specifying only join conditions in the `ON` clause and moving filters to `WHEN ... AND` clauses to avoid incorrect results and improve performance. In some scenarios, separate `INSERT`, `UPDATE`, and `DELETE` statements can outperform `MERGE` with less blocking, especially under high concurrency.

**Beginner-Friendly Explanation**
`MERGE` performance depends on how well the tables are indexed and how many rows are involved. Indexing the join columns, filtering the source to only changed rows, and avoiding overly complex `ON` conditions all help. Under heavy concurrency, `MERGE` may not be the fastest option—sometimes separate statements are better.

#### Purposes

- To ensure that synchronization jobs complete within their batch windows.
- To avoid table scans and excessive locking during large merges.
- To choose between `MERGE` and alternative patterns based on workload characteristics.

#### Syntax Rules and Structure

Performance optimization is primarily a matter of design and indexing rather than syntax:

- **Index the join columns**: Create a clustered index on the target's join columns; ensure the source is also indexed on the join columns.
- **Filter the source**: Use a CTE or subquery in the `USING` clause to include only changed rows (`WHERE last_modified > watermark`).
- **Move filters out of `ON`**: Put non-join filters in `WHEN ... AND` clauses, not in the `ON` clause.
- **Disable non-unique indexes**: Temporarily disabling non-unique nonclustered indexes on the target during a large load can improve performance, at the cost of exclusive access.
- **Parameterize**: For repeated executions, parameterization can improve plan reuse.

**Constraints and limitations:**

- `MERGE` is not always faster than separate statements; benchmark both for your workload.
- Under high concurrency, `MERGE` may cause more blocking than separate statements; consider `HOLDLOCK` or alternative patterns.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Indexed MERGE with a filtered source

```sql
-- Step 1: Assume tables with appropriate indexes.
CREATE CLUSTERED INDEX ix_target_key ON dbo.Target (Key1);
CREATE INDEX ix_source_key ON dbo.Source (Key1);

-- Step 2: Merge only changed rows using a CTE.
WITH Changed AS (
    SELECT Key1, Value
    FROM dbo.Source
    WHERE LastModified > @watermark
)
MERGE dbo.Target AS T
USING Changed AS S
ON T.Key1 = S.Key1
WHEN MATCHED THEN UPDATE SET T.Value = S.Value
WHEN NOT MATCHED BY TARGET THEN INSERT (Key1, Value) VALUES (S.Key1, S.Value);
```

**Expected output:** (No direct output; the statement completes successfully.)

**Why this output appears**: The clustered index on the target and the index on the source support efficient join processing. The `Changed` CTE restricts the source to rows modified after the last watermark, reducing the number of candidate rows and the overall work performed by the merge.

#### Real-World Cases

- **Incremental ETL**: A nightly job merges only rows changed since the last run, using a watermark column, keeping the batch window short and resource usage predictable.
- **Large dimension load**: A weekly full synchronization uses a clustered index on the join key and temporarily disables non-critical indexes to complete within the maintenance window.

---

### 8. Concurrency

#### Definitions

**Core Definition**
`MERGE` concurrency refers to the behaviour of the statement when multiple sessions execute it simultaneously or when it runs alongside other DML operations, including the risk of race conditions, primary-key violations, and deadlocks.

**Technical Definition**
By default, `MERGE` performs its match phase without exclusive locks, which increases concurrency when merged data is disjoint but creates a race condition when multiple sessions attempt to insert or update the same key values: both sessions can see a row as “not matched” and both attempt an `INSERT`, causing a primary-key violation. The standard mitigation in SQL Server is to add `WITH (HOLDLOCK)` to the target table (or use the `SERIALIZABLE` isolation level), which holds key-range locks for the duration of the statement and prevents concurrent inserts of matching keys. However, `HOLDLOCK` reduces concurrency and is only necessary when the merge may insert rows that could conflict with concurrent inserts; it does not guarantee full protection against all deadlock scenarios. Microsoft explicitly notes that when high concurrency is expected, separate `INSERT`, `UPDATE`, and `DELETE` statements can perform better with less blocking than `MERGE`.

**Beginner-Friendly Explanation**
`MERGE` concurrency is about what happens when two users or processes run a `MERGE` at the same time. Without protection, both can think a row is missing and both try to insert it, causing an error. Adding `HOLDLOCK` (or using a stricter isolation level) makes `MERGE` reserve the key range so that only one session can insert a given key, but it also blocks other sessions and can reduce performance.

#### Purposes

- To prevent primary-key violations and duplicate-key errors under concurrent `MERGE` execution.
- To avoid deadlocks caused by conflicting lock acquisition orders.
- To choose between `MERGE` and alternative patterns based on the expected concurrency level.

#### Syntax Rules and Structure

**SQL Server – MERGE with HOLDLOCK:**

```sql
MERGE dbo.Target WITH (HOLDLOCK) AS T
USING dbo.Source AS S
ON T.Key1 = S.Key1
WHEN MATCHED THEN UPDATE SET T.Value = S.Value
WHEN NOT MATCHED THEN INSERT (Key1, Value) VALUES (S.Key1, S.Value);
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `WITH (HOLDLOCK)` | Applies `SERIALIZABLE` isolation to the target table for the duration of the statement; prevents concurrent inserts of matching keys. |
| `SERIALIZABLE` isolation | Alternatively, set the transaction isolation level to `SERIALIZABLE` before the `MERGE`. |

**Syntax rules:**

- `HOLDLOCK` is a table hint placed immediately after the target table name.
- `HOLDLOCK` is a synonym for `SERIALIZABLE`; it holds shared locks until the transaction completes, preventing other transactions from modifying the data read by the merge.

**Constraints and limitations:**

- `HOLDLOCK` reduces concurrency; it should be used only when concurrent inserts of the same key are possible.
- `HOLDLOCK` does not prevent all deadlocks; complex merges may still deadlock under certain access patterns.
- In high-concurrency OLTP paths, separate `UPDATE` and `INSERT` statements (or `INSERT ... ON CONFLICT` in PostgreSQL) may be safer and faster than `MERGE`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – MERGE with HOLDLOCK under concurrency

```sql
-- Step 1: Create a table with a unique key.
CREATE TABLE dbo.Counters (Key1 nvarchar(50) PRIMARY KEY, Value int);

-- Step 2: Session A: MERGE with HOLDLOCK.
MERGE dbo.Counters WITH (HOLDLOCK) AS T
USING (SELECT 'hits' AS Key1, 1 AS Value) AS S
ON T.Key1 = S.Key1
WHEN MATCHED THEN UPDATE SET T.Value = T.Value + S.Value
WHEN NOT MATCHED THEN INSERT (Key1, Value) VALUES (S.Key1, S.Value);
```

**Expected output** (Session A succeeds; Session B, running the same statement concurrently, waits and then either updates or inserts without a duplicate-key error):

```text
(1 row affected)
```

**Why this output appears**: The `HOLDLOCK` hint causes the first session to acquire a key-range lock on the `'hits'` key. The second session blocks until the first commits, then sees the inserted row and performs an update instead of a duplicate insert, avoiding a primary-key violation.

#### Real-World Cases

- **Hot OLTP counter updates**: A counter table is updated by many concurrent sessions; `MERGE WITH (HOLDLOCK)` prevents duplicate-key errors, but alternative patterns (explicit `UPDATE` then `INSERT` in a transaction) may be preferred for performance.
- **Concurrent staging loads**: Multiple ETL jobs merge into the same target table; `HOLDLOCK` ensures that concurrent jobs do not conflict on the same keys, at the cost of serializing access to those keys.

---

## References

- MERGE (Transact-SQL) – SQL Server – https://learn.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql
- MERGE (Transact-SQL) – SQL Server (syntax and WHEN clauses) – https://learn.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql?view=sql-server-2017
- PostgreSQL 15 Documentation – MERGE – https://www.postgresql.org/docs/15/sql-merge.html
- PostgreSQL Git – MERGE (source documentation) – https://git.postgresql.org/gitweb/?p=postgresql.git;a=blob_plain;f=doc/src/sgml/ref/merge.sgml;hb=HEAD
- SQL Server MERGE Statement – UPSERT Patterns, WHEN MATCHED, and Safer Alternatives – DbSchema – https://dbschema.com/blog/sql-server/merge-statement/
- Why is TSQL MERGE failing with a Primary Key Violation? Isn’t it atomic? – Microsoft Learn – https://learn.microsoft.com/en-us/archive/blogs/sqlcat/why-is-tsql-merge-failing-with-a-primary-key-violation-isnt-it-atomic
- t-sql-upserts-merge – Microsoft SQL Skills – https://github.com/microsoft/microsoft-sql/blob/main/plugins/microsoft-sql/skills/t-sql-upserts-merge/SKILL.md
- MERGE (Transact-SQL) – SQL Server (performance and concurrency recommendations) – https://learn.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql?view=sql-server-2017#performance
- MERGE (Transact-SQL) – SQL Server (HOLDLOCK recommendation) – https://learn.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql?view=sql-server-2017#concurrency
- Use Caution with SQL Server’s MERGE Statement – MSSQLTips – https://www.mssqltips.com/sqlservertip/3074/use-caution-with-sql-servers-merge/
- PostgreSQL MERGE performance vs INSERT ON CONFLICT – pgsql-performance mailing list – https://www.postgresql.org/message-id/flat/...
- How Citus supports the PostgreSQL MERGE command – Citus Data – https://www.citusdata.com/blog/2023/07/27/how-citus-supports-the-postgresql-merge-command/
- MERGE in Integration Services Packages – Microsoft Learn – https://learn.microsoft.com/en-us/sql/integration-services/control-flow/merge-in-integration-services-packages
- Oracle MERGE: The Complete Guide to Upserts, Data Sync, and ERP Integration – Cleverence – https://www.cleverence.com/articles/oracle-database/oracle-merge-the-complete-guide-to-upserts-data-sync-and-erp-integration/