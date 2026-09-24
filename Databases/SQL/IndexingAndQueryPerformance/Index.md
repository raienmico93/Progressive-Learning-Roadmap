# SQL Index: A Comprehensive Programming Cheat Sheet

---

## Topic Overview: SQL Index Fundamentals

### Definitions

**Core Definition:** An SQL index is a data structure that provides a fast access path to rows in a database table based on the values of one or more columns.

**Technical Definition:** An index is a separate, physically stored data structure (typically a B-tree or B+ tree) containing an ordered list of key values from one or more table columns, each associated with a pointer (such as a rowid) to the corresponding row in the underlying table. The database query optimizer uses indexes to reduce the number of disk I/O operations required to satisfy a query predicate .

**Beginner-Friendly Explanation:** Think of an index like the index at the back of a textbook. Instead of flipping through every page to find a topic, you look it up in the index, which tells you exactly which page to turn to. A database index works the same way—it tells the database exactly where to find the rows you need, without scanning the whole table.

### Key Characteristics

- **Ordered structure:** B-tree indexes maintain key values in sorted order, enabling efficient exact-match and range searches .
- **Separate storage:** Indexes occupy their own storage segments, consuming additional disk space beyond the table data .
- **Automatic maintenance:** The database engine automatically maintains indexes as data is inserted, updated, or deleted .
- **Optimizer-driven:** The query optimizer decides at execution time whether to use an index or perform a table scan based on cost estimates .
- **Multiple types:** Common types include B-tree, bitmap, hash, and columnstore indexes, each suited to different workloads .

### Prerequisites

- **Basic SQL knowledge:** Understanding of `SELECT`, `WHERE`, `JOIN`, and data manipulation statements.
- **Table and column concepts:** Familiarity with relational tables, rows, columns, and data types.
- **Database access:** Privileges to create and inspect indexes (typically `CREATE INDEX` or `ALTER TABLE` permissions).

### Related Programming Areas

- **Query Optimization:** Indexes are the primary mechanism for improving query performance.
- **Database Administration:** Index design, monitoring, and maintenance are core DBA responsibilities.
- **Data Warehousing:** Columnstore indexes and bitmap indexes are optimized for analytical workloads.
- **Application Development:** Understanding index behavior helps developers write performant queries and design effective schemas.

### Core Concepts / Features

The following core concepts are covered:

1. Purpose of Indexes
2. Index Lookup (Point Query)
3. Index Scan
4. Table Scan / Sequential Scan
5. Index Selectivity
6. Index Maintenance Cost
7. Index Fragmentation

---

## 1. Purpose of Indexes

### Definitions

**Core Definition:** The primary purpose of an index is to accelerate data retrieval by providing a fast, direct path to rows matching a query condition, at the cost of additional storage and slower write operations.

**Technical Definition:** Indexes reduce the number of I/O operations required to locate rows satisfying a predicate by replacing a full table scan with a targeted tree traversal. The trade-off is that every `INSERT`, `UPDATE`, or `DELETE` on the indexed table must also maintain the index structure, increasing write overhead .

**Beginner-Friendly Explanation:** An index makes reading data faster but writing data slower. Every time you add, change, or remove a row, the database must also update the index—like updating a textbook's index every time you edit a page.

### Purposes

- To accelerate `SELECT` queries that filter or join on indexed columns.
- To enforce uniqueness constraints (unique indexes).
- To support efficient range queries and ordered retrieval without a sort operation.
- To enable index-only scans when all required columns are contained in the index.

### Syntax Rules and Structure

**Complete General Syntax (PostgreSQL):**

```sql
CREATE [ UNIQUE ] INDEX [ CONCURRENTLY ] [ IF NOT EXISTS ] index_name
    ON table_name [ USING method ]
    ( { column_name | ( expression ) } [ COLLATE collation ] [ opclass ] [ ASC | DESC ] [ NULLS { FIRST | LAST } ] [, ...] )
    [ WITH ( storage_parameter [= value] [, ...] ) ]
    [ TABLESPACE tablespace_name ]
    [ WHERE predicate ];
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `UNIQUE` | Enforces uniqueness of key values |
| `index_name` | Name of the index (must be unique within schema) |
| `table_name` | The table to index |
| `USING method` | Index method: `btree` (default), `hash`, `gist`, `gin`, `brin` |
| `column_name` | Column(s) to include in the index key |
| `ASC \| DESC` | Sort order of key values |
| `WHERE predicate` | Creates a partial index for a subset of rows |

**Syntax Rules:**

- An index name must be unique within its schema.
- Column order in a composite index matters: the leading (leftmost) column is most critical for query matching .
- Unique indexes reject duplicate key values (except for `NULL` values, which are typically treated as distinct).
- The `WHERE` clause creates a partial index, indexing only rows that satisfy the predicate.

**Constraints and Limitations:**

- Indexes consume disk space proportional to the number and width of indexed columns.
- Very wide indexes (many columns or large data types) increase storage and maintenance overhead .
- Some data types (e.g., `TEXT`, `BLOB`, `XML` in some databases) cannot be directly indexed without special handling .
- Low-cardinality columns (few distinct values) may not benefit from B-tree indexes .

### Annotated Complete Code Examples

**Example 1: Creating a B-tree Index**

```sql
-- Setup: Create a sample table
CREATE TABLE employees (
    employee_id   SERIAL PRIMARY KEY,
    first_name    VARCHAR(50),
    last_name     VARCHAR(50),
    department    VARCHAR(50),
    salary        NUMERIC(10, 2)
);

-- Insert sample data
INSERT INTO employees (first_name, last_name, department, salary) VALUES
    ('Alice', 'Johnson', 'Engineering', 95000.00),
    ('Bob',   'Smith',   'Marketing',   72000.00),
    ('Carol', 'Williams','Engineering', 105000.00),
    ('David', 'Brown',   'Sales',       68000.00);

-- Create an index on the last_name column
CREATE INDEX idx_employees_last_name ON employees (last_name);

-- Query that can benefit from the index
SELECT first_name, last_name
FROM employees
WHERE last_name = 'Williams';

-- Expected Output:
--  first_name | last_name
-- ------------+-----------
--  Carol      | Williams
```

**Why this output occurs:** The `CREATE INDEX` statement builds a B-tree structure containing sorted `last_name` values and pointers to the corresponding table rows. When the query searches for `'Williams'`, the optimizer traverses the B-tree to find the matching entry and follows the pointer to retrieve the row, avoiding a full table scan.

**Example 2: Composite Index**

```sql
-- Create a composite index on department and salary
CREATE INDEX idx_emp_dept_salary ON employees (department, salary);

-- Query that uses the leading column only
SELECT first_name, department
FROM employees
WHERE department = 'Engineering';

-- Expected Output:
--  first_name | department
-- ------------+-------------
--  Alice      | Engineering
--  Carol      | Engineering
```

**Why this output occurs:** The composite index is ordered first by `department`, then by `salary`. Because the query filters on `department` (the leading column), the index can be used efficiently. A query filtering on `salary` alone would not use this index effectively .

### Real-World Cases

**Case 1: E-Commerce Product Search**

An online store indexes the `product_name` and `category_id` columns to speed up search queries and category browsing, reducing response times from seconds to milliseconds.

**Case 2: User Authentication**

A unique index on `users.email` accelerates login lookups while enforcing that no two accounts share the same email address.

### References

- PostgreSQL Documentation — Indexes - https://www.postgresql.org/docs/current/indexes.html
- Oracle Database — Indexes and Index-Organized Tables - https://docs.oracle.com/en/database/oracle/oracle-database/26/cncpt/indexes-and-index-organized-tables.html
- Microsoft SQL Server — Index Architecture and Design Guide - https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-index-design-guide

---

## 2. Index Lookup (Point Query)

### Definitions

**Core Definition:** An index lookup (also called a point query or index unique scan) finds a single row or a small number of rows by traversing the index tree to a specific key value.

**Technical Definition:** A unique index scan locates at most one row using an equality predicate on the index key columns. For a non-unique index, an equality predicate may return multiple rows. The database traverses the B-tree from root to leaf, comparing the search key at each level, then follows the rowid pointer to the table row .

**Beginner-Friendly Explanation:** An index lookup is like using a phone book to find one person's number. You flip directly to the right page instead of reading every entry. It is the fastest way to find a specific row.

### Purposes

- To retrieve a single row by its primary key or unique key.
- To resolve equality predicates on indexed columns efficiently.
- To support foreign key lookups and joins on indexed columns.
- To enable sub-millisecond response times for high-frequency point queries.

### Syntax Rules and Structure

**Complete General Syntax (query that triggers an index lookup):**

```sql
SELECT column_list
FROM table_name
WHERE indexed_column = value;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `indexed_column` | A column with a B-tree or hash index |
| `= value` | Equality predicate enabling a point lookup |
| `SELECT column_list` | Columns to retrieve (may include only indexed columns) |

**Syntax Rules:**

- The predicate must be an equality condition (`=`) on the leading column(s) of the index.
- For a unique index, the optimizer can perform a unique scan returning at most one row.
- For a non-unique index, the scan returns all rows matching the key value.
- The index key must be fully specified for a unique scan; partial key matches use a range scan instead.

**Constraints and Limitations:**

- Function-wrapped columns (e.g., `WHERE UPPER(last_name) = 'SMITH'`) may prevent index usage unless a function-based index exists.
- Implicit type conversions (e.g., comparing a string column to a number) can disable index usage.
- Very small tables may not benefit from index lookups because a table scan is faster .

### Annotated Complete Code Examples

**Example 1: Unique Index Lookup on Primary Key**

```sql
-- Primary key creates a unique index automatically
SELECT first_name, last_name, salary
FROM employees
WHERE employee_id = 3;

-- Expected Output:
--  first_name | last_name |  salary
-- ------------+-----------+----------
--  Carol      | Williams  | 105000.00
```

**Why this output occurs:** The `employee_id` column is the primary key, which is backed by a unique B-tree index. The optimizer performs a unique index scan, traversing the B-tree from root to leaf in O(log n) time, then retrieves the single matching row.

**Example 2: Non-Unique Index Lookup**

```sql
-- idx_employees_last_name is a non-unique index
SELECT first_name, last_name
FROM employees
WHERE last_name = 'Johnson';

-- Expected Output:
--  first_name | last_name
-- ------------+-----------
--  Alice      | Johnson
```

**Why this output occurs:** The non-unique index on `last_name` is used to find all rows where `last_name = 'Johnson'`. Even though the index is non-unique, the query returns exactly one row because only one employee has that last name in the sample data. If multiple employees shared the same last name, all would be returned.

### Real-World Cases

**Case 1: REST API Resource Retrieval**

A REST API endpoint `GET /users/{id}` performs a unique index lookup on the `users.id` column to return a single user profile, achieving consistent low-latency performance even as the table grows to millions of rows.

**Case 2: Foreign Key Join**

An orders table has a foreign key `customer_id` referencing `customers.id`. A query joining orders to customers uses the unique index on `customers.id` to perform efficient nested loop joins.

### References

- Oracle Database — Index Unique Scan - https://docs.oracle.com/en/database/oracle/oracle-database/26/cncpt/indexes-and-index-organized-tables.html
- PostgreSQL Documentation — Index-Only Scans and Covering Indexes - https://www.postgresql.org/docs/current/indexes-index-only-scans.html

---

## 3. Index Scan

### Definitions

**Core Definition:** An index scan traverses a portion or the entirety of an index structure to locate rows matching a query predicate, returning them in index key order.

**Technical Definition:** An index range scan reads a contiguous range of index leaf blocks defined by start and stop keys, then follows rowid pointers to retrieve table rows. Variants include index full scan (reads all leaf blocks in sorted order), index fast full scan (reads all leaf blocks without sorting, enabling multi-block I/O), and index skip scan (logically splits a composite index to use non-leading columns) .

**Beginner-Friendly Explanation:** An index scan is like reading a range of entries in a book's index—from "Smith" to "Smythe"—and then looking up each referenced page. It is efficient for range queries and ordered retrieval.

### Purposes

- To retrieve rows within a range of key values (e.g., `BETWEEN`, `>`, `<`).
- To return rows in sorted order without a separate sort operation.
- To support `ORDER BY` clauses on indexed columns.
- To scan only a subset of the index when a partial index or skip scan is applicable.

### Syntax Rules and Structure

**Complete General Syntax (queries that trigger index scans):**

```sql
-- Range scan
SELECT column_list
FROM table_name
WHERE indexed_column BETWEEN low_value AND high_value;

-- Ordered retrieval
SELECT column_list
FROM table_name
ORDER BY indexed_column;

-- Skip scan (non-leading column)
SELECT column_list
FROM table_name
WHERE non_leading_indexed_column = value;
```

**Component Breakdown:**

| Scan Type | Trigger | Characteristic |
|-----------|---------|----------------|
| Range scan | `BETWEEN`, `<`, `>`, `<=`, `>=` | Reads contiguous leaf blocks |
| Full scan | `ORDER BY` indexed column | Reads all leaf blocks in order |
| Fast full scan | Query covered by index, no sort needed | Multi-block I/O, unordered |
| Skip scan | Non-leading column predicate, low cardinality leading column | Logical sub-index scans  |

**Syntax Rules:**

- Range scans require the index key to be the subject of a range predicate.
- Full index scans are chosen when the query needs data in index order and the cost is less than a table scan plus sort.
- Skip scans are beneficial when the leading column has few distinct values and the trailing column is selective .

**Constraints and Limitations:**

- Range scans on large ranges can be slower than table scans due to random I/O when the table is not clustered by the index key .
- Index full scans read single blocks and may be slower than a full table scan for unordered data .
- Skip scans add overhead and are only advantageous under specific cardinality conditions.

### Annotated Complete Code Examples

**Example 1: Index Range Scan**

```sql
SELECT first_name, last_name, salary
FROM employees
WHERE salary BETWEEN 70000 AND 100000;

-- Expected Output:
--  first_name | last_name |  salary
-- ------------+-----------+----------
--  Alice      | Johnson   | 95000.00
--  Bob        | Smith     | 72000.00
```

**Why this output occurs:** If an index exists on `salary`, the optimizer performs a range scan starting at 70000 and ending at 100000, reading only the leaf blocks in that range. Carol (105000) and David (68000) fall outside the range and are not read from the index.

**Example 2: Index Skip Scan (Oracle)**

```sql
-- Composite index on (cust_gender, cust_email)
-- Query does not specify the leading column
SELECT * FROM sh.customers
WHERE cust_email = 'Abbey@company.example.com';

-- Conceptually equivalent to:
SELECT * FROM sh.customers WHERE cust_gender = 'F'
  AND cust_email = 'Abbey@company.example.com'
UNION ALL
SELECT * FROM sh.customers WHERE cust_gender = 'M'
  AND cust_email = 'Abbey@company.example.com';
```

**Why this output occurs:** The leading column `cust_gender` has only two distinct values (`F` and `M`). The database logically splits the index into two sub-indexes and searches each for the target email. This allows the composite index to be used even though the leading column is not in the `WHERE` clause .

### Real-World Cases

**Case 1: Date Range Reporting**

A financial application queries transactions within a date range (`WHERE transaction_date BETWEEN '2024-01-01' AND '2024-12-31'`). A B-tree index on `transaction_date` enables a range scan that reads only the relevant months of data.

**Case 2: Sorted Pagination**

A web application displays products ordered by `price`. An index on `price` allows the database to return results in sorted order without a costly sort operation, speeding up paginated queries.

### References

- Oracle Database — Index Range Scan - https://docs.oracle.com/en/database/oracle/oracle-database/26/cncpt/indexes-and-index-organized-tables.html
- Oracle Database — Index Skip Scan - https://docs.oracle.com/en/database/oracle/oracle-database/26/cncpt/indexes-and-index-organized-tables.html
- CERN Database Blog — Indexes in Oracle DB Part 2 - https://db-blog.web.cern.ch/blog/emil-pilecki/2014-09-indexes-oracle-db-part-2

---

## 4. Table Scan / Sequential Scan

### Definitions

**Core Definition:** A table scan (also called a sequential scan or full table scan) reads every row of a table from beginning to end, evaluating the query predicate against each row.

**Technical Definition:** A full table scan reads all data blocks belonging to the table, including all rows, regardless of whether they satisfy the query predicate. It is the fallback access method when no usable index exists or when the optimizer determines that scanning the entire table is cheaper than using an index .

**Beginner-Friendly Explanation:** A table scan is like reading an entire book from cover to cover to find every mention of a word. It is thorough but slow for large tables.

### Purposes

- To retrieve all rows from a table when no `WHERE` clause is present.
- To scan tables that have no suitable index for the query predicate.
- To efficiently read a large percentage of a table where an index would cause more random I/O than a sequential read.
- To serve as the baseline for cost comparison in the query optimizer .

### Syntax Rules and Structure

**Complete General Syntax (queries that may trigger a table scan):**

```sql
-- No WHERE clause: full table scan
SELECT * FROM table_name;

-- Non-indexed column predicate
SELECT * FROM table_name WHERE non_indexed_column = value;

-- Low-selectivity predicate
SELECT * FROM table_name WHERE low_cardinality_column = value;
```

**Component Breakdown:**

| Trigger Condition | Reason |
|-------------------|--------|
| No `WHERE` clause | All rows are required |
| Predicate on non-indexed column | No index available |
| Low-selectivity predicate | Index would return most rows anyway |
| Small table | Index overhead exceeds scan cost  |
| Stale statistics | Optimizer may choose scan incorrectly  |

**Syntax Rules:**

- A table scan reads all table blocks; the number of rows read equals the table row count.
- The optimizer chooses a table scan when the estimated cost of an index scan plus row lookups exceeds the cost of reading the entire table sequentially.
- Table scans can use multi-block I/O, making them efficient for large sequential reads .

**Constraints and Limitations:**

- Table scans read all rows, even those that do not match the predicate, wasting I/O for selective queries.
- Performance degrades linearly with table size.
- High-concurrency table scans can cause buffer cache contention.

### Annotated Complete Code Examples

**Example 1: Table Scan Due to No Index**

```sql
-- No index exists on the 'department' column in this example
SELECT first_name, last_name
FROM employees
WHERE department = 'Engineering';

-- Expected Output:
--  first_name | last_name
-- ------------+-----------
--  Alice      | Johnson
--  Carol      | Williams
```

**Why this output occurs:** If no index exists on `department`, the database must read every row in the `employees` table, evaluate the `WHERE` condition for each, and return only the matching rows. For a table with millions of rows, this would be slow.

**Example 2: Table Scan Chosen by Optimizer**

```sql
-- Assume an index exists on 'salary', but the query returns most rows
SELECT first_name, salary
FROM employees
WHERE salary > 50000;

-- Expected Output:
--  first_name |  salary
-- ------------+----------
--  Alice      | 95000.00
--  Bob        | 72000.00
--  Carol      | 105000.00
--  David      | 68000.00
```

**Why this output occurs:** Although an index on `salary` exists, the predicate `salary > 50000` matches nearly all rows in the table. The optimizer may determine that scanning the entire table sequentially is cheaper than traversing the index and performing random row lookups for most rows. This is a cost-based decision.

### Real-World Cases

**Case 1: Data Warehouse Full Export**

An ETL process extracts all rows from a staging table nightly. A full table scan is optimal because every row is needed.

**Case 2: Optimizer Choosing Scan Over Index**

A report queries orders from the last 5 years in a table containing 6 years of data. The predicate matches over 80% of rows. The optimizer correctly chooses a table scan, avoiding millions of random I/O operations from an index scan.

### References

- SAP Documentation — Sequential Search (Table Scan) - https://help.sap.com/docs/SAP_NETWEAVER_DBOS/a0fb7a8618844418a7efd4c4a529dc41/44cebb654e8c0446e10000000a114a6b.html
- Oracle Database — Full Table Scan - https://docs.oracle.com/en/database/oracle/oracle-database/26/cncpt/indexes-and-index-organized-tables.html

---

## 5. Index Selectivity

### Definitions

**Core Definition:** Index selectivity is a measure of how well an index distinguishes between rows, defined as the ratio of distinct key values to the total number of rows in the table.

**Technical Definition:** Selectivity is computed as `number of distinct values / total number of rows`. A selectivity close to 1 (high selectivity) indicates that each index entry corresponds to few rows, making the index efficient for filtering. A selectivity close to 0 (low selectivity) indicates many duplicate values, making the index less effective for B-tree lookups .

**Beginner-Friendly Explanation:** An index is "selective" if it helps you narrow down to a small number of rows quickly. A unique column like `email` is highly selective. A column like `gender` (only two values) is not selective—an index on it won't help much because you'll still get half the table.

### Purposes

- To determine whether a column is a good candidate for a B-tree index.
- To guide index design and avoid creating ineffective indexes.
- To help the optimizer estimate the cost of using an index versus a table scan.
- To support the use of bitmap indexes for low-selectivity columns .

### Syntax Rules and Structure

**Selectivity Formula:**

```
Selectivity = Number of Distinct Values / Total Number of Rows
```

**Interpretation:**

| Selectivity Range | Interpretation | Index Suitability |
|-------------------|----------------|-------------------|
| Close to 1 (e.g., 0.9–1.0) | Highly selective; few duplicates | Excellent for B-tree index |
| Moderate (e.g., 0.1–0.9) | Moderately selective | May be useful; evaluate workload |
| Close to 0 (e.g., < 0.1) | Low selectivity; many duplicates | Poor for B-tree; consider bitmap index  |

**Syntax Rules:**

- Selectivity is a property of the data, not the index definition.
- The optimizer uses statistics (histograms) to estimate selectivity for cost-based decisions .
- Skewed data distributions can make an index appear selective globally but perform poorly for specific values .

**Constraints and Limitations:**

- Selectivity alone does not determine index usefulness; query patterns and data distribution also matter.
- Stale statistics can lead the optimizer to misjudge selectivity and choose suboptimal plans .
- Function-based indexes and partial indexes can improve selectivity for specific query patterns.

### Annotated Complete Code Examples

**Example 1: High Selectivity Index**

```sql
-- Setup: Create a table with a unique email column
CREATE TABLE users (
    user_id    SERIAL PRIMARY KEY,
    email      VARCHAR(255) UNIQUE,
    country    VARCHAR(50)
);

-- Insert sample data
INSERT INTO users (email, country) VALUES
    ('alice@example.com', 'USA'),
    ('bob@example.com', 'Canada'),
    ('carol@example.com', 'UK');

-- Query using the highly selective email index
SELECT user_id, email
FROM users
WHERE email = 'alice@example.com';

-- Expected Output:
--  user_id |       email
-- ---------+-------------------
--        1 | alice@example.com
```

**Why this output occurs:** The `email` column has 3 distinct values for 3 rows, giving a selectivity of 1.0. The unique index on `email` allows the optimizer to perform a unique index scan, returning exactly one row with minimal I/O.

**Example 2: Low Selectivity Index**

```sql
-- Query on a low-selectivity column (country)
SELECT user_id, email
FROM users
WHERE country = 'USA';

-- Expected Output:
--  user_id |       email
-- ---------+-------------------
--        1 | alice@example.com
```

**Why this output occurs:** The `country` column has 3 distinct values for 3 rows, but if the table had 1 million rows with only 10 countries, selectivity would be 0.00001. An index on `country` would not be useful for a query returning a large fraction of rows. The optimizer would likely choose a table scan instead.

### Real-World Cases

**Case 1: Choosing Which Columns to Index**

A database designer analyzes a `transactions` table with 10 million rows. The `transaction_id` column has 10 million distinct values (selectivity = 1.0) and is indexed. The `status` column has 4 distinct values (selectivity = 0.0000004) and is not indexed with B-tree. Instead, a bitmap index might be considered for analytical queries .

**Case 2: Skewed Data Distribution**

A `customers` table has a `zip_code` column. In a state dominated by one large city, 40% of customers share the same zip code. A B-tree index on `zip_code` performs well for queries on suburban zip codes (few rows) but poorly for queries on the dominant city zip code (400,000 rows). Cost-based optimizers with histograms can choose table scans for the skewed value .

### References

- Oracle Help Center — Determinants of Index Efficiency - https://docs.oracle.com/cd/E05554_01/books/AnyTuning/AnyTuning_DBservers9.html
- Oracle Database — Index Clustering Factor - https://docs.oracle.com/en/database/oracle/oracle-database/26/cncpt/indexes-and-index-organized-tables.html

---

## 6. Index Maintenance Cost

### Definitions

**Core Definition:** Index maintenance cost is the performance penalty imposed on `INSERT`, `UPDATE`, and `DELETE` operations because the database must update all indexes affected by the data change.

**Technical Definition:** Every DML operation that modifies indexed columns requires the database to locate the affected index entries and modify the B-tree structure, which may involve leaf block splits, merges, or rebalancing. The cost is proportional to the number and width of indexes on the table .

**Beginner-Friendly Explanation:** Every time you add or change data, the database has to update the index too. More indexes mean faster reads but slower writes—like keeping multiple copies of a book's index up to date every time you edit a page.

### Purposes

- To quantify the trade-off between read performance and write performance.
- To guide index design decisions, especially for write-heavy workloads.
- To explain why over-indexing degrades `INSERT`/`UPDATE`/`DELETE` throughput.
- To inform maintenance scheduling and index consolidation strategies.

### Syntax Rules and Structure

**Affected Operations:**

| Operation | Maintenance Required |
|-----------|---------------------|
| `INSERT` | Add new entries to all indexes on the table |
| `UPDATE` | Modify entries only in indexes on updated columns |
| `DELETE` | Remove entries from all indexes on the table |

**Syntax Rules:**

- Every index on a table must be maintained for `INSERT` and `DELETE` operations .
- `UPDATE` statements maintain only indexes whose columns are modified .
- Wide indexes (many columns or large data types) incur higher maintenance cost than narrow indexes .

**Constraints and Limitations:**

- Over-indexing (creating many speculative indexes) slows data modification and can cause concurrency issues .
- Index maintenance consumes CPU, memory, and I/O resources.
- On heavily updated tables, keeping indexes narrow and few is recommended .

### Annotated Complete Code Examples

**Example 1: INSERT with Multiple Indexes**

```sql
-- Assume three indexes exist: on employee_id (PK), last_name, and department
INSERT INTO employees (first_name, last_name, department, salary)
VALUES ('Eve', 'Davis', 'Engineering', 88000.00);

-- The database must:
-- 1. Insert the row into the table
-- 2. Add 'Davis' to idx_employees_last_name
-- 3. Add 'Engineering' to the department index
-- 4. Update the primary key index for the new employee_id
```

**Why this matters:** Each index adds overhead to the `INSERT`. With four indexes, the statement performs five write operations (one table + four index updates). Removing unnecessary indexes directly improves `INSERT` throughput.

**Example 2: UPDATE on Indexed Column**

```sql
-- Update a column that is indexed
UPDATE employees
SET last_name = 'Davis-Smith'
WHERE employee_id = 5;

-- The database must:
-- 1. Update the row in the table
-- 2. Remove 'Davis' from idx_employees_last_name and insert 'Davis-Smith'
-- The department index is NOT maintained because department was not updated
```

**Why this output occurs:** Only indexes on modified columns are updated. If the `UPDATE` had changed `department` instead, the `last_name` index would not be touched. This is why updating indexed columns is more expensive than updating non-indexed columns.

### Real-World Cases

**Case 1: OLTP System with High Write Volume**

An order-processing system handles 10,000 inserts per second. Each `orders` table has 8 indexes. The DBA identifies that 3 indexes are rarely used and drops them, reducing insert latency by 40%.

**Case 2: Batch Update Performance**

A nightly ETL job updates millions of rows in a staging table. To improve performance, the team drops non-essential indexes before the batch load and recreates them afterward.

### References

- Apache Derby — Indexes have a cost for inserts, updates, and deletes - https://svn.apache.org/repos/asf/db/derby/docs/branches/10.3/src/tuning/ctunoptimz856914.dita
- Microsoft SQL Server — General Index Design Guidelines - https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-index-design-guide
- MySQL Reference Manual — Optimizing InnoDB Queries - https://dev.mysql.com/doc/refman/26.7/en/optimizing-innodb-queries.html

---

## 7. Index Fragmentation

### Definitions

**Core Definition:** Index fragmentation is the physical disorganization of index pages caused by data modification operations, where the logical order of index keys no longer matches the physical order of pages on disk.

**Technical Definition:** In a B-tree index, fragmentation occurs when page splits (from `INSERT` operations) and page merges (from `DELETE` operations) cause leaf pages to become scattered across non-contiguous disk blocks. This reduces page density and increases the number of I/O operations required for index scans .

**Beginner-Friendly Explanation:** Imagine a filing cabinet where new folders are shoved in wherever there's space, and removed folders leave gaps. The folders are no longer in order, and you have to open more drawers to find what you need. Index fragmentation is the same—data is scattered, and the database works harder to read it.

### Purposes

- To identify when index maintenance is needed.
- To restore page density and sequential ordering, improving scan performance.
- To reduce the number of I/O operations for range scans and full index scans.
- To balance maintenance cost against performance gains.

### Syntax Rules and Structure

**Measurement (SQL Server):**

```sql
SELECT
    index_id,
    avg_fragmentation_in_percent,
    avg_page_space_used_in_percent
FROM sys.dm_db_index_physical_stats(
    DB_ID(), OBJECT_ID('table_name'), NULL, NULL, 'LIMITED'
);
```

**Maintenance Options:**

| Method | Description | When to Use |
|--------|-------------|-------------|
| `REORGANIZE` | Defragments leaf level online; less resource-intensive | Fragmentation 5%–30%  |
| `REBUILD` | Drops and recreates the index; can be online or offline | Fragmentation > 30%  |

**Syntax Rules:**

- Fragmentation is measured as `avg_fragmentation_in_percent` in SQL Server .
- Page density (`avg_page_space_used_in_percent`) measures how full pages are; low density indicates wasted space .
- `REORGANIZE` is always online and can be interrupted without losing work.
- `REBUILD` can be performed online (Enterprise edition) or offline.

**Constraints and Limitations:**

- Fragmentation is not the only factor in index performance; page density and clustering factor also matter .
- Rebuilding an index requires sufficient disk space and can lock resources.
- Some RDBMSs (e.g., PostgreSQL) handle fragmentation differently and may not expose the same metrics.

### Annotated Complete Code Examples

**Example 1: Measuring Fragmentation (SQL Server)**

```sql
-- Check fragmentation on the idx_employees_last_name index
SELECT
    OBJECT_NAME(ips.object_id) AS table_name,
    i.name AS index_name,
    ips.avg_fragmentation_in_percent,
    ips.avg_page_space_used_in_percent,
    ips.page_count
FROM sys.dm_db_index_physical_stats(
    DB_ID(), OBJECT_ID('employees'), NULL, NULL, 'LIMITED'
) ips
JOIN sys.indexes i ON ips.object_id = i.object_id
    AND ips.index_id = i.index_id
WHERE i.name = 'idx_employees_last_name';

-- Example Output:
--  table_name | index_name            | avg_fragmentation_in_percent | avg_page_space_used_in_percent | page_count
-- ------------+-----------------------+------------------------------+--------------------------------+------------
--  employees  | idx_employees_last_name |                          42.5 |                          68.3 |        120
```

**Why this output occurs:** The `sys.dm_db_index_physical_stats` function returns fragmentation statistics. A fragmentation of 42.5% indicates significant disorganization. The page density of 68.3% means pages are only about two-thirds full, wasting space and I/O.

**Example 2: Reorganizing an Index**

```sql
-- Reorganize the fragmented index (online operation)
ALTER INDEX idx_employees_last_name ON employees REORGANIZE;

-- Verify improvement
SELECT
    avg_fragmentation_in_percent,
    avg_page_space_used_in_percent
FROM sys.dm_db_index_physical_stats(
    DB_ID(), OBJECT_ID('employees'), NULL, NULL, 'LIMITED'
)
WHERE index_id = INDEX_ID('employees', 'idx_employees_last_name');

-- Expected Output (after reorganization):
--  avg_fragmentation_in_percent | avg_page_space_used_in_percent
-- ------------------------------+--------------------------------
--                           0.5 |                           89.2
```

**Why this output occurs:** `REORGANIZE` physically reorders the leaf pages to match logical key order, reducing fragmentation from 42.5% to near zero and increasing page density. This improves the performance of range scans on the index.

### Real-World Cases

**Case 1: Nightly Index Maintenance**

A DBA schedules nightly `REORGANIZE` jobs for indexes with 5–30% fragmentation and `REBUILD` jobs for indexes exceeding 30%. This maintains query performance without excessive resource consumption .

**Case 2: Post-Bulk-Load Rebuild**

After a nightly ETL process inserts millions of rows, index fragmentation spikes. The ETL job includes a final step to rebuild all indexes on affected tables, restoring performance for morning queries.

### References

- Microsoft SQL Server — Reorganize and Rebuild Indexes - https://learn.microsoft.com/en-us/sql/relational-databases/indexes/reorganize-and-rebuild-indexes
- Microsoft SQL Server — Index Architecture and Design Guide - https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-index-design-guide

---

## Summary Table

| Concept | Key Purpose | Primary Cost / Limitation |
|---------|-------------|---------------------------|
| Purpose of Indexes | Accelerate data retrieval | Storage space and write overhead  |
| Index Lookup | Find specific rows quickly | Ineffective for non-unique or function-wrapped predicates |
| Index Scan | Range queries and ordered retrieval | Random I/O if table not clustered by index  |
| Table Scan | Read all rows; fallback access method | Linear performance degradation with table size |
| Index Selectivity | Determine index effectiveness | Skewed data can mislead selectivity estimates  |
| Maintenance Cost | Quantify write penalty | More indexes = slower INSERT/UPDATE/DELETE  |
| Fragmentation | Restore index performance | Maintenance requires time and resources  |

---

## References

- Oracle Database — Indexes and Index-Organized Tables - https://docs.oracle.com/en/database/oracle/oracle-database/26/cncpt/indexes-and-index-organized-tables.html
- PostgreSQL Documentation — Indexes - https://www.postgresql.org/docs/current/indexes.html
- Microsoft SQL Server — Index Architecture and Design Guide - https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-index-design-guide
- Microsoft SQL Server — Reorganize and Rebuild Indexes - https://learn.microsoft.com/en-us/sql/relational-databases/indexes/reorganize-and-rebuild-indexes
- Apache Derby — Indexes have a cost for inserts, updates, and deletes - https://svn.apache.org/repos/asf/db/derby/docs/branches/10.3/src/tuning/ctunoptimz856914.dita
- Oracle Help Center — Determinants of Index Efficiency - https://docs.oracle.com/cd/E05554_01/books/AnyTuning/AnyTuning_DBservers9.html
- SAP Documentation — Sequential Search (Table Scan) - https://help.sap.com/docs/SAP_NETWEAVER_DBOS/a0fb7a8618844418a7efd4c4a529dc41/44cebb654e8c0446e10000000a114a6b.html
- CERN Database Blog — Indexes in Oracle DB Part 2 - https://db-blog.web.cern.ch/blog/emil-pilecki/2014-09-indexes-oracle-db-part-2
- MySQL Reference Manual — Optimizing InnoDB Queries - https://dev.mysql.com/doc/refman/26.7/en/optimizing-innodb-queries.html