# SQL Index Design & Strategies: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL index design is the process of selecting, defining, and maintaining database indexes to optimise query performance while balancing the costs of storage, write overhead, and maintenance.

**Technical Definition:** Index design involves analysing query workloads, data distribution, and table characteristics to create index structures that enable the query optimiser to efficiently locate and retrieve rows. Key decisions include selecting columns for index keys, determining column order in composite indexes, choosing between single-column and multi-column indexes, including payload columns for covering queries, enforcing uniqueness, filtering subsets of data, and consolidating redundant indexes .

**Beginner-Friendly Explanation:** Index design is like deciding where to put bookmarks in a book. You could put a bookmark on every page (too many indexes, slows down writing), or none at all (slow reading). The art is to place bookmarks where you look most often—so you can flip to the right page fast without cluttering the book.

### Key Characteristics

- **Workload-driven:** Index design depends on the queries your application runs, not just the schema .
- **Trade-off-based:** Every index speeds up reads but slows down writes and consumes storage .
- **Column-order-sensitive:** In composite indexes, the order of columns determines which queries can use the index .
- **Maintenance-aware:** Indexes must be updated on INSERT, UPDATE, and DELETE, so over-indexing degrades write performance .

### Prerequisites

- **Query analysis skills:** Ability to identify frequently executed queries and their predicates.
- **Basic index structure knowledge:** Understanding of B-tree, hash, and other index types (covered in previous sections).
- **Database access:** Privileges to create, alter, and drop indexes.

### Related Programming Areas

- **Query Optimisation:** The optimizer chooses among available indexes based on cost estimates.
- **Database Administration:** Index maintenance, monitoring, and consolidation are DBA tasks.
- **Application Development:** Developers write queries that determine which indexes are useful.

### Core Concepts / Features

1. Single-Column Indexes
2. Composite Indexes
3. Column Order (Left-Most Prefix Rule)
4. Covering Indexes (INCLUDE Clause)
5. Unique Indexes
6. Partial or Filtered Indexes
7. Index Consolidation

---

## 1. Single-Column Indexes

### Definitions

**Core Definition:** A single-column index is built on exactly one column of a table, providing a fast access path for queries that filter or sort on that specific column.

**Technical Definition:** A single-column B-tree index maintains a sorted list of values from one column, each with a pointer to the corresponding row. The index is used when a query's predicate references the indexed column in a SARGable (Search ARGumentable) manner .

**Beginner-Friendly Explanation:** A single-column index is like a bookmark for one specific topic. If you frequently look up information by "Last Name," you put a bookmark on the Last Name column. It helps for that one thing, but not for other searches.

### Purposes

- To accelerate equality and range queries on a frequently filtered column.
- To support sorting on a specific column without a full table sort.
- To serve as the simplest building block for more complex indexing strategies.

### Syntax Rules and Structure

**Complete General Syntax (SQL Server / PostgreSQL):**

```sql
CREATE [UNIQUE] INDEX index_name ON table_name (column_name [ASC | DESC]);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `index_name` | Unique identifier for the index |
| `table_name` | The table containing the column |
| `column_name` | The single column to index |
| `ASC \| DESC` | Sort order (default ASC) |

**Syntax Rules:**

- The indexed column must be of an indexable data type (not `text`, `ntext`, `image` in SQL Server) .
- For very long string columns, consider indexing a prefix rather than the full column.

**Constraints and Limitations:**

- Indexing small tables may be counterproductive; a table scan can be faster than index traversal .
- A single-column index on a low-cardinality column (few distinct values) is often ineffective for B-tree lookups.

### Annotated Complete Code Examples

**Example 1: Creating a Single-Column Index**

```sql
-- Create a table of orders
CREATE TABLE orders (
    order_id     INT PRIMARY KEY,
    customer_id  INT,
    status       VARCHAR(20),
    order_date   DATE
);

-- Create a single-column index on customer_id
-- This accelerates queries that filter by customer
CREATE INDEX idx_orders_customer ON orders (customer_id);

-- Query that benefits from the index
SELECT order_id, order_date
FROM orders
WHERE customer_id = 12345;

-- Expected Output (conceptual):
--  order_id | order_date
-- ----------+------------
--      1001 | 2026-01-15
--      1057 | 2026-02-20
```

**Why this output occurs:** The index on `customer_id` allows the database to locate all orders for customer 12345 without scanning the entire `orders` table. The optimizer performs an index seek on the B-tree, then follows pointers to retrieve the matching rows.

### Real-World Cases

**Case 1: Foreign Key Lookups**

Every foreign key column should typically have a single-column index to support efficient joins and referential integrity checks. A query joining `orders` to `customers` uses the index on `orders.customer_id` to find matching rows.

**Case 2: Status Filtering**

An application frequently queries `WHERE status = 'pending'`. A single-column index on `status` can accelerate this query, though its effectiveness depends on selectivity.

### References

- Microsoft SQL Server — Index Architecture and Design Guide - https://learn.microsoft.com/sql/relational-databases/sql-server-index-design-guide

---

## 2. Composite Indexes

### Definitions

**Core Definition:** A composite index (also called a multi-column index) is an index built on two or more columns of a table, with the columns ordered in a specific sequence.

**Technical Definition:** A composite B-tree index sorts rows first by the leading column, then by the second column within equal values of the first, and so on. MySQL documentation states that a composite index can be used for queries that test all columns in the index, or queries that test just the first column, the first two columns, the first three columns, and so on .

**Beginner-Friendly Explanation:** A composite index is like a phone book sorted by Last Name, then First Name. You can look up someone by last name alone, or by last name and first name together. But you cannot look up by first name alone—the phone book isn't organised that way.

### Purposes

- To accelerate queries that filter on multiple columns simultaneously.
- To support queries that filter on the leading column alone (as a "bonus").
- To enable index-only retrieval for queries whose columns are all in the index.
- To enforce uniqueness across multiple columns (unique composite index).

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
CREATE INDEX index_name ON table_name (column1, column2, ..., columnN);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `column1` | Leading (leftmost) column—most critical for query matching |
| `column2 ... columnN` | Trailing columns, usable only when preceding columns are constrained |

**Syntax Rules:**

- MySQL supports up to 16 columns in a composite index .
- The order of columns is determined at creation time and cannot be changed without recreating the index.
- PostgreSQL can combine separate single-column indexes, but a composite index may be dramatically faster for some queries .

**Constraints and Limitations:**

- A composite index is not a substitute for single-column indexes on each column; it serves different query patterns.
- Very wide composite indexes consume significant storage and increase write overhead .
- Index merge optimisation (combining multiple single-column indexes) is not always as efficient as a well-designed composite index .

### Annotated Complete Code Examples

**Example 1: Composite Index on (Last Name, First Name)**

```sql
-- Table of employees
CREATE TABLE employees (
    emp_id     INT PRIMARY KEY,
    last_name  VARCHAR(50),
    first_name VARCHAR(50),
    department VARCHAR(50)
);

-- Composite index on last_name, first_name
CREATE INDEX idx_emp_name ON employees (last_name, first_name);

-- Query 1: Uses the index (leading column)
SELECT emp_id FROM employees WHERE last_name = 'Smith';

-- Query 2: Uses the index (both columns)
SELECT emp_id FROM employees WHERE last_name = 'Smith' AND first_name = 'John';

-- Query 3: Does NOT use the index (non-leading column)
SELECT emp_id FROM employees WHERE first_name = 'John';
```

**Why this behavior occurs:** MySQL documentation confirms that the index on `(last_name, first_name)` is used for queries testing `last_name` alone, or `last_name` and `first_name` together. It is not used for queries testing `first_name` alone because `first_name` is not a leftmost prefix .

### Real-World Cases

**Case 1: Multi-Tenant Applications**

A SaaS application stores data for multiple tenants in shared tables. Every query includes `WHERE tenant_id = ? AND ...`. A composite index on `(tenant_id, created_at)` allows efficient filtering by tenant and sorting by date.

**Case 2: E-Commerce Order History**

An online store queries `WHERE customer_id = ? AND order_status = ?`. A composite index on `(customer_id, order_status)` serves this query directly, while also supporting queries on `customer_id` alone.

### References

- MySQL 8.0 Reference Manual — Multiple-Column Indexes - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/multiple-column-indexes.html
- PostgreSQL Mailing List — Composite vs Single-Column Indexes - https://www.postgresql.org/message-id/20240213152917.qwjikj6gcfegjt3a%40hjp.at

---

## 3. Column Order (Left-Most Prefix Rule)

### Definitions

**Core Definition:** The left-most prefix rule states that a composite index can be used for queries that reference a contiguous set of columns starting from the leftmost (first) column in the index definition.

**Technical Definition:** For an index on `(col1, col2, col3)`, the index can satisfy queries with predicates on `(col1)`, `(col1, col2)`, or `(col1, col2, col3)`. It cannot be used for predicates on `(col2)`, `(col3)`, or `(col2, col3)` alone because these do not form a leftmost prefix of the index .

**Beginner-Friendly Explanation:** Think of a phone book sorted by Last Name, then First Name. You can search by Last Name alone, or by Last Name and First Name together. You cannot search by First Name alone—the book isn't sorted that way. The left-most prefix rule says the index must be used from the first column onward.

### Purposes

- To guide composite index column ordering decisions.
- To determine whether an existing index can serve a given query.
- To avoid creating redundant indexes that duplicate capabilities of existing composite indexes.

### Syntax Rules and Structure

**The Rule Applied to Index (a, b, c):**

| Query Predicate | Index Usable? | Columns Used |
|-----------------|---------------|--------------|
| `WHERE a = ?` | Yes | a |
| `WHERE a = ? AND b = ?` | Yes | a, b |
| `WHERE a = ? AND b = ? AND c = ?` | Yes | a, b, c |
| `WHERE a = ? AND c = ?` | Partial | a only (c cannot filter without b) |
| `WHERE b = ?` | No | — |
| `WHERE b = ? AND c = ?` | No | — |
| `WHERE c = ?` | No | — |

**Syntax Rules:**

- The leading column should generally be the most frequently used in query predicates.
- Range predicates (`>`, `<`, `BETWEEN`) stop index usage for subsequent columns .
- Columns after a range predicate may still be useful for `ORDER BY` or `GROUP BY` in some cases .

**Constraints and Limitations:**

- The rule applies to B-tree and similar ordered indexes; hash indexes do not follow this rule.
- Some databases (e.g., Oracle with skip scan) can partially work around the rule for low-cardinality leading columns, but this is not universal.

### Annotated Complete Code Examples

**Example 1: Demonstrating the Left-Most Prefix Rule**

```sql
-- Composite index on (department, salary)
CREATE INDEX idx_dept_salary ON employees (department, salary);

-- Query 1: Uses index (leading column)
EXPLAIN SELECT * FROM employees WHERE department = 'Engineering';
-- Uses: idx_dept_salary (index seek on department)

-- Query 2: Uses index (both columns)
EXPLAIN SELECT * FROM employees WHERE department = 'Engineering' AND salary > 90000;
-- Uses: idx_dept_salary (index seek on department, then range on salary)

-- Query 3: Cannot use index efficiently (non-leading column)
EXPLAIN SELECT * FROM employees WHERE salary > 90000;
-- Likely: Full table scan (salary is not a leftmost prefix)
```

**Why this behavior occurs:** MySQL documentation explicitly states that for an index on `(col1, col2, col3)`, search capabilities exist on `(col1)`, `(col1, col2)`, and `(col1, col2, col3)`. Queries on `(col2)` or `(col2, col3)` cannot use the index because they do not form a leftmost prefix .

### Real-World Cases

**Case 1: Designing for Query Patterns**

A reporting system runs queries that always include `region` and often include `year`. An index on `(region, year)` serves both query patterns. Reversing the order to `(year, region)` would break queries that filter on `region` alone.

**Case 2: Avoiding Redundant Indexes**

If an index on `(customer_id, order_date)` exists, a separate index on `(customer_id)` is usually redundant because the composite index's leading column already serves queries on `customer_id` alone .

### References

- MySQL 8.0 Reference Manual — Leftmost Prefix Rule - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/multiple-column-indexes.html
- Composite Index Design (GitHub) - https://raw.githubusercontent.com/planetscale/database-skills/refs/heads/main/skills/mysql/references/composite-indexes.md

---

## 4. Covering Indexes (INCLUDE Clause)

### Definitions

**Core Definition:** A covering index is an index that contains all the columns needed to satisfy a query, eliminating the need to access the base table.

**Technical Definition:** A covering index either includes all required columns as key columns or uses the `INCLUDE` clause to add non-key columns to the leaf level of the index. Microsoft documentation states that covering indexes "can improve query performance because all the data needed to meet the requirements of the query exists within the index itself" .

**Beginner-Friendly Explanation:** A covering index is like having a mini-table that contains exactly the columns your query needs. Instead of looking up the index, then going to the main table to get the other columns, everything is right there in the index.

### Purposes

- To eliminate key lookups (SQL Server) or heap fetches (PostgreSQL).
- To reduce disk I/O by avoiding base table access.
- To accelerate high-traffic queries that select a small number of columns.

### Syntax Rules and Structure

**Complete General Syntax (SQL Server):**

```sql
CREATE NONCLUSTERED INDEX index_name
ON table_name (key_column1, key_column2)
INCLUDE (included_column1, included_column2);
```

**Complete General Syntax (PostgreSQL):**

```sql
CREATE INDEX index_name
ON table_name (key_column)
INCLUDE (included_column1, included_column2);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `key_column` | Columns used for filtering, sorting, and seeking |
| `INCLUDE` | Non-key columns stored only at the leaf level |
| `included_column` | Columns returned by the query but not used for filtering |

**Syntax Rules:**

- INCLUDE columns are not part of the index key and do not affect index ordering.
- Included columns can be of data types that cannot be key columns (e.g., `varchar(max)` in SQL Server) .
- PostgreSQL documentation notes that INCLUDE columns "can save the overhead of fetching the heap" for index-only scans .

**Constraints and Limitations:**

- INCLUDE columns increase index size and write overhead.
- PostgreSQL's `INCLUDE` feature requires version 11 or later.
- Covering indexes are most beneficial for queries that select a small number of columns from a large table .

### Annotated Complete Code Examples

**Example 1: Covering Index with INCLUDE**

```sql
-- Orders table with many columns
-- Query frequently needs: customer_id (filter), order_date, status (output)
CREATE NONCLUSTERED INDEX idx_orders_covering
ON orders (customer_id)
INCLUDE (order_date, status);

-- Query fully covered by the index
SELECT order_date, status
FROM orders
WHERE customer_id = 12345;

-- Expected Output (conceptual):
--  order_date | status
-- ------------+---------
--  2026-01-15 | Shipped
--  2026-02-20 | Pending
```

**Why this output occurs:** The index's key column `customer_id` enables efficient seeking. The INCLUDE columns `order_date` and `status` provide the query's output without accessing the `orders` table. The query is satisfied entirely from the index leaf level, avoiding a key lookup or table access .

### Real-World Cases

**Case 1: High-Traffic API Endpoint**

A REST API endpoint returns user profiles: `SELECT name, email, avatar_url FROM users WHERE user_id = ?`. A covering index on `user_id` with INCLUDE `(name, email, avatar_url)` satisfies the query entirely from the index, reducing I/O by orders of magnitude .

**Case 2: Reporting Query Optimisation**

A dashboard query selects `SUM(amount), COUNT(*)` from `transactions WHERE account_id = ? AND transaction_date BETWEEN ? AND ?`. A covering index on `(account_id, transaction_date) INCLUDE (amount)` allows the aggregation to be computed from the index alone.

### References

- Microsoft SQL Server — Index Design Guide (Covering Indexes) - https://learn.microsoft.com/sql/relational-databases/sql-server-index-design-guide
- PostgreSQL — Index-Only Scans and Covering Indexes - https://www.postgresql.org/docs/current/indexes-index-only-scans.html

---

## 5. Unique Indexes

### Definitions

**Core Definition:** A unique index enforces that no two rows in the table have the same values in the indexed column(s), serving both as a data integrity constraint and a performance structure.

**Technical Definition:** A unique index guarantees that any attempt to insert or update a duplicate key value fails. Microsoft documentation states that "there are no significant differences between creating a UNIQUE constraint and creating a unique index that is independent of a constraint" in terms of data validation and optimizer behavior .

**Beginner-Friendly Explanation:** A unique index is like a rule that says "no two rows can have the same value here"—and it also makes lookups fast. For example, a unique index on `email` prevents two users from registering with the same email, and speeds up login lookups.

### Purposes

- To enforce uniqueness constraints on one or more columns.
- To accelerate equality lookups on the unique column(s).
- To provide additional information to the query optimiser for more efficient plans .

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
CREATE UNIQUE INDEX index_name ON table_name (column1 [, column2, ...]);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `UNIQUE` | Enforces no duplicate key values |
| `index_name` | Index identifier |
| `column(s)` | The column(s) that must be unique |

**Syntax Rules:**

- A unique index cannot be created if duplicate values already exist in the key columns .
- `NULL` values are considered equal for uniqueness purposes; you cannot have more than one row with `NULL` in a unique index column (SQL Server behavior) .
- The `IGNORE_DUP_KEY` option (SQL Server) controls whether duplicate rows in a multi-row INSERT are skipped (ON) or the entire statement rolls back (OFF, default) .

**Constraints and Limitations:**

- Unique indexes on `NULL`-able columns allow only one `NULL` value (in SQL Server).
- Oracle and PostgreSQL treat `NULL` values as distinct for unique constraints, allowing multiple `NULL`s.
- Unique indexes add write overhead: every INSERT and UPDATE must check for duplicates.

### Annotated Complete Code Examples

**Example 1: Creating a Unique Index**

```sql
-- Users table
CREATE TABLE users (
    user_id    INT PRIMARY KEY,
    email      VARCHAR(255),
    username   VARCHAR(50)
);

-- Create a unique index on email
CREATE UNIQUE INDEX idx_users_email ON users (email);

-- This INSERT succeeds
INSERT INTO users (user_id, email, username) VALUES (1, 'alice@example.com', 'alice');

-- This INSERT fails with a duplicate key error
INSERT INTO users (user_id, email, username) VALUES (2, 'alice@example.com', 'alice2');
-- Error: Violation of UNIQUE KEY constraint 'idx_users_email'.
-- Cannot insert duplicate key in object 'dbo.users'.
```

**Why this output occurs:** The unique index on `email` prevents the second INSERT from creating a duplicate email address. The database checks the index before inserting and rejects the row. This enforces data integrity while also accelerating lookups like `SELECT * FROM users WHERE email = 'alice@example.com'` .

**Example 2: Unique Composite Index**

```sql
-- Prevent duplicate (last_name, first_name) combinations
CREATE UNIQUE INDEX idx_unique_name ON employees (last_name, first_name);

-- This succeeds
INSERT INTO employees (emp_id, last_name, first_name) VALUES (1, 'Smith', 'John');

-- This succeeds (different first name)
INSERT INTO employees (emp_id, last_name, first_name) VALUES (2, 'Smith', 'Jane');

-- This fails (duplicate combination)
INSERT INTO employees (emp_id, last_name, first_name) VALUES (3, 'Smith', 'John');
-- Error: Duplicate key value.
```

**Why this output occurs:** The composite unique index enforces that the combination of `last_name` and `first_name` is unique. The third INSERT fails because `('Smith', 'John')` already exists. This is useful for enforcing natural keys in addition to surrogate primary keys .

### Real-World Cases

**Case 1: User Registration**

Every user must have a unique email address. A unique index on `email` prevents duplicate registrations and accelerates login lookups. This is one of the most common uses of unique indexes.

**Case 2: Order Numbers**

An e-commerce system generates order numbers that must be unique. A unique index on `order_number` prevents duplicates and provides fast order lookup.

### References

- Microsoft SQL Server — Creating Unique Indexes - https://learn.microsoft.com/sql/relational-databases/indexes/create-unique-indexes
- Microsoft SQL Server — Unique Indexes (SQL Server 2008 R2) - https://learn.microsoft.com/previous-versions/sql/sql-server-2008-r2/ms175132(v=sql.105)

---

## 6. Partial or Filtered Indexes

### Definitions

**Core Definition:** A filtered index (SQL Server) or partial index (PostgreSQL) is a nonclustered index that indexes only a subset of rows in a table, defined by a `WHERE` clause predicate.

**Technical Definition:** A filtered index uses a filter predicate to index only a portion of rows. Microsoft documentation states that a "well-designed filtered index can improve query performance, reduce index maintenance costs, and reduce index storage costs compared with full-table indexes" .

**Beginner-Friendly Explanation:** A filtered index is like indexing only the important pages of a book. If you frequently look up chapters that are "Active," you create an index that only covers Active chapters. It's smaller, faster, and cheaper to maintain than indexing every chapter.

### Purposes

- To reduce index size and storage costs by indexing only relevant rows.
- To reduce maintenance overhead by updating the index only when indexed rows change.
- To improve query performance through smaller index size and more accurate statistics .

### Syntax Rules and Structure

**Complete General Syntax (SQL Server):**

```sql
CREATE NONCLUSTERED INDEX index_name
ON table_name (column1, column2)
WHERE filter_predicate;
```

**Complete General Syntax (PostgreSQL):**

```sql
CREATE INDEX index_name
ON table_name (column1)
WHERE filter_predicate;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `index_name` | Index identifier |
| `column(s)` | Key columns for the index |
| `WHERE filter_predicate` | Condition defining which rows are indexed |

**Syntax Rules:**

- The filter predicate must use simple comparison operators (`=`, `<`, `>`, `IS NULL`, `IN`, `BETWEEN`) .
- Filtered indexes cannot reference multiple tables or use complex logic.
- The filter predicate must be included in the query's `WHERE` clause for the index to be used.

**Constraints and Limitations:**

- Filtered indexes are most beneficial when the filtered subset is small compared to the full table .
- SQL Server documentation notes that if the filtered index includes most rows, it may cost more to maintain than a full-table index .
- Some older SQL Server versions had bugs with filtered indexes in complex joins .

### Annotated Complete Code Examples

**Example 1: Filtered Index on Non-NULL Values**

```sql
-- BillOfMaterials table with many NULL EndDate values
-- Only 199 of 2679 rows have non-NULL EndDate
CREATE NONCLUSTERED INDEX FIBillOfMaterialsWithEndDate
ON Production.BillOfMaterials (ComponentID, StartDate)
WHERE EndDate IS NOT NULL;

-- Query that uses the filtered index
SELECT ProductAssemblyID, ComponentID, StartDate
FROM Production.BillOfMaterials
WHERE EndDate IS NOT NULL
  AND ComponentID = 5
  AND StartDate > '2008-01-01';
```

**Why this output occurs:** The filtered index contains only the 199 rows with non-NULL `EndDate`, making it much smaller than a full-table index. Queries that select only non-NULL EndDate rows use this smaller index, resulting in less I/O and faster execution .

**Example 2: Filtered Index for Heterogeneous Data**

```sql
-- Orders table with status column: 'Pending', 'Shipped', 'Delivered'
-- 95% of queries filter on 'Pending' orders
CREATE NONCLUSTERED INDEX idx_pending_orders
ON orders (order_date)
WHERE status = 'Pending';

-- Query that uses the filtered index
SELECT order_id, order_date
FROM orders
WHERE status = 'Pending'
  AND order_date > '2026-01-01';
```

**Why this output occurs:** The filtered index indexes only pending orders (perhaps 5% of the table). Queries for pending orders use this small, focused index, avoiding the overhead of scanning or indexing shipped and delivered orders .

### Real-World Cases

**Case 1: Soft-Delete Pattern**

Applications that use soft deletes (e.g., `WHERE deleted_at IS NULL`) benefit from filtered indexes that index only non-deleted rows. The index stays small because deleted rows are excluded.

**Case 2: Sparse Columns**

A table with a column that is `NULL` for most rows (e.g., `termination_date` for active employees) benefits from a filtered index on `WHERE termination_date IS NOT NULL`.

### References

- Microsoft SQL Server — Filtered Index Design Guidelines - https://learn.microsoft.com/sql/relational-databases/sql-server-index-design-guide
- SQL Server Advent Calendar — Filtered Indexes - https://learn.microsoft.com/archive/blogs/andrew/sql-server-advent-calendar-4-filtered-indexes

---

## 7. Index Consolidation

### Definitions

**Core Definition:** Index consolidation is the process of identifying and eliminating redundant, overlapping, or unused indexes to reduce storage, write overhead, and maintenance costs.

**Technical Definition:** Index consolidation involves analysing index usage statistics to find indexes that are duplicates (identical key columns), subsets (key columns are a prefix of another index), or completely unused. These indexes can be disabled or dropped without impacting query performance .

**Beginner-Friendly Explanation:** Index consolidation is like cleaning out your bookmarks. If you have three bookmarks on the same page, you only need one. If you have bookmarks you never use, remove them. Fewer bookmarks mean the book stays cleaner and is easier to manage.

### Purposes

- To reduce storage consumption from redundant indexes.
- To reduce write overhead on INSERT, UPDATE, and DELETE operations.
- To simplify index maintenance and reduce confusion.
- To improve optimizer efficiency by removing unnecessary candidates .

### Syntax Rules and Structure

**Identifying Redundant Indexes:**

| Redundancy Type | Definition | Example |
|-----------------|------------|---------|
| Exact Duplicate | Same key columns, same order | Index (A, B) and Index (A, B) |
| Subset/Superset | One index's keys are a prefix of another | Index (A) and Index (A, B, C) |
| Reverse Order | Same columns, different order | Index (A, B) and Index (B, A) |
| Unused Index | No reads recorded since last restart | Index with zero seeks/scans |

**Syntax Rules:**

- Subset indexes are generally redundant if the superset index has the same leading columns .
- Reverse-order indexes may both be useful if queries use different column orders.
- Unused indexes should only be dropped after a sufficient observation period (weeks, not days) .

**Constraints and Limitations:**

- An index that appears unused may be used for a rare but critical query.
- Removing a unique index that enforces data integrity is not recommended.
- Clustered indexes cannot be consolidated away if they define the table's physical order.

### Annotated Complete Code Examples

**Example 1: Identifying Redundant Indexes**

```sql
-- Assume these indexes exist:
-- idx1 ON users (email)
-- idx2 ON users (email, username)
-- idx3 ON users (username)

-- idx1 is redundant: email is a leftmost prefix of idx2
-- idx3 is NOT redundant: username is not a leftmost prefix of idx2

-- Script to drop the redundant index
DROP INDEX idx1 ON users;

-- Verify remaining indexes
SELECT name, type_desc
FROM sys.indexes
WHERE object_id = OBJECT_ID('users');
```

**Why this output occurs:** The index on `(email)` is a subset of the index on `(email, username)`. Any query that can use `idx1` can also use `idx2` (by using the leading `email` column). Dropping `idx1` saves storage and write overhead without impacting query performance .

**Example 2: Using sp_IndexCleanup (Third-Party Tool)**

```sql
-- Erik Darling's sp_IndexCleanup analyzes index usage
EXEC sp_IndexCleanup
    @database_name = 'YourDatabase',
    @min_reads = 100,
    @min_writes = 0,
    @min_size_gb = 0.5;

-- Output includes recommendations:
-- - Merge overlapping indexes
-- - Disable unused indexes
-- - Compress uncompressed indexes
```

**Why this output occurs:** The tool queries `sys.dm_db_index_usage_stats` to find indexes with zero reads and analyses index definitions to find duplicates and subsets. It then generates scripts for review, not automatic execution .

### Real-World Cases

**Case 1: Post-Development Cleanup**

After a development cycle, a DBA runs index usage reports to find indexes created for testing but never used in production. These are dropped to improve write performance.

**Case 2: Quarterly Index Review**

A large database with hundreds of indexes undergoes quarterly consolidation. The DBA identifies 15% of indexes as redundant or unused, reducing storage by 10% and improving batch load times by 20% .

### References

- Erik Darling — Introducing sp_IndexCleanup! - https://erikdarling.com/introducing-sp_indexcleanup/
- Oracle Ask TOM — Single-Column Index vs. Composite Index - https://asktom.oracle.com/ords/asktom.search?tag=is-it-recommendable-to-have-a-single-column-index-when-it-is-also-the-first-in-a-multicolumn-index

---

## Summary Table

| Strategy | Best For | Key Consideration | Primary Limitation |
|----------|----------|-------------------|-------------------|
| Single-Column | Simple, frequent filters | Keep narrow; avoid over-indexing | May not serve multi-column queries |
| Composite | Multi-column predicates | Column order is critical | Non-leading columns unusable |
| Left-Most Prefix | Understanding index usability | Leading column most important | Cannot skip leading columns |
| Covering (INCLUDE) | High-traffic SELECT queries | Eliminates key lookups | Increases index size |
| Unique | Data integrity + lookup speed | Enforces business rules | Write overhead for duplicate checks |
| Filtered/Partial | Subset queries, sparse data | Smaller index, less maintenance | Only used when predicate matches |
| Consolidation | Reducing index bloat | Remove dupes, subsets, unused | Requires usage statistics over time |

---

## References

- Microsoft SQL Server — Index Architecture and Design Guide - https://learn.microsoft.com/sql/relational-databases/sql-server-index-design-guide
- MySQL 8.0 Reference Manual — Multiple-Column Indexes - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/multiple-column-indexes.html
- Microsoft SQL Server — Creating Unique Indexes - https://learn.microsoft.com/sql/relational-databases/indexes/create-unique-indexes
- Erik Darling — Introducing sp_IndexCleanup! - https://erikdarling.com/introducing-sp_indexcleanup/
- PostgreSQL Mailing List — Composite vs Single-Column Indexes - https://www.postgresql.org/message-id/20240213152917.qwjikj6gcfegjt3a%40hjp.at
- Oracle Ask TOM — Single-Column Index vs. Composite Index - https://asktom.oracle.com/ords/asktom.search?tag=is-it-recommendable-to-have-a-single-column-index-when-it-is-also-the-first-in-a-multicolumn-index
- Composite Index Design (GitHub) - https://raw.githubusercontent.com/planetscale/database-skills/refs/heads/main/skills/mysql/references/composite-indexes.md
- Microsoft SQL Server — Filtered Index Design Guidelines - https://learn.microsoft.com/sql/relational-databases/sql-server-index-design-guide