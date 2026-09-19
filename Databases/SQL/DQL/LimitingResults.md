# SQL Limiting Results: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL limiting results refers to the family of clauses and techniques used to restrict the number of rows returned by a SELECT statement, and to navigate through large result sets in manageable segments (pagination).

**Technical Definition:** Per the ANSI/ISO SQL Standard, row limiting is implemented through the OFFSET and FETCH FIRST clauses, introduced in SQL:2008. The standard syntax is `OFFSET m ROWS FETCH NEXT n ROWS ONLY`. Vendor-specific implementations predate and often differ from the standard: LIMIT (PostgreSQL, MySQL, SQLite), TOP (SQL Server, Sybase), and ROWNUM (Oracle). These clauses are logically applied after ORDER BY and before the result set is returned to the client.

**Beginner-Friendly Explanation:** When a table has millions of rows, you rarely want to see all of them at once. SQL limiting clauses let you say "give me only the first 10 rows" or "skip the first 50 rows and give me the next 20." This is essential for pagination (showing results page by page) and for quickly previewing data. Without a limiting clause, a query returns every matching row, which can be slow and overwhelming.

### Key Characteristics

- **Three syntax families:** LIMIT (PostgreSQL, MySQL, SQLite), TOP (SQL Server, Sybase), and FETCH FIRST (ANSI standard, Oracle, DB2, modern PostgreSQL/SQL Server).
- **Order-dependent:** Row limiting without ORDER BY produces non-deterministic results — the database may return any subset of rows.
- **Pagination support:** OFFSET allows skipping rows for page navigation, but large OFFSET values degrade performance linearly.
- **Tie handling:** WITH TIES (TOP and FETCH FIRST) includes additional rows that match the last row's sort key.
- **Performance-sensitive:** Top-N optimization and index usage can dramatically speed up limited queries.

### Prerequisites

- Basic SQL syntax (SELECT, FROM, WHERE, ORDER BY)
- Understanding of sorting and indexes
- Familiarity with query execution order (ORDER BY runs before LIMIT)

### Related Programming Areas

- **Web Application Development:** Pagination is fundamental to displaying search results, product listings, and user feeds.
- **API Design:** REST and GraphQL APIs use cursor-based or offset-based pagination for large collections.
- **Data Analysis:** LIMIT is used for quick data exploration and sampling.
- **Query Optimization:** Understanding Top-N optimization and index usage is critical for performance tuning.
- **Database Migration:** Different SQL dialects require translating LIMIT/TOP/FETCH syntax.

---

## Core Concepts / Key Features

### 1. Core Row-Limiting Clauses

#### 1.1 LIMIT Syntax (PostgreSQL, MySQL, SQLite)

**Core Definitions:**

- **Core Definition:** LIMIT restricts the number of rows returned by a query to a specified maximum.
- **Technical Definition:** In PostgreSQL, MySQL, and SQLite, the LIMIT clause accepts one or two numeric arguments. With one argument, it specifies the maximum number of rows to return. With two arguments (MySQL-specific), the first specifies the offset and the second the row count. PostgreSQL also supports the ANSI-standard OFFSET/FETCH syntax as an alternative.
- **Beginner-Friendly Explanation:** LIMIT is the simplest way to say "give me only the first N rows." You add `LIMIT 10` to the end of your query.

**Purposes:**

- To restrict the number of rows returned by a query
- To quickly preview data without retrieving the entire table
- To implement pagination when combined with OFFSET
- To improve performance by reducing data transfer

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- One argument: maximum rows to return
SELECT column_list FROM table_name LIMIT row_count;

-- Two arguments (MySQL): offset, row_count
SELECT column_list FROM table_name LIMIT offset, row_count;

-- PostgreSQL/MySQL/SQLite: LIMIT row_count OFFSET offset
SELECT column_list FROM table_name LIMIT row_count OFFSET offset;
```

**Component Breakdown:**

- `row_count`: Maximum number of rows to return
- `offset`: Number of rows to skip before returning rows (0-based)

**Syntax Rules:**

- LIMIT without ORDER BY returns an unpredictable subset of rows. PostgreSQL documentation explicitly warns: "When using LIMIT, it is important to use an ORDER BY clause that constrains the result rows into a unique order. Otherwise you will get an unpredictable subset of the query's rows".
- `LIMIT ALL` is the same as omitting the LIMIT clause.
- `LIMIT NULL` is the same as omitting the LIMIT clause in PostgreSQL.
- MySQL's `LIMIT row_count` is equivalent to `LIMIT 0, row_count`.

**Constraints and Limitations:**

- LIMIT is not ANSI standard; it is vendor-specific.
- MySQL's two-argument syntax is not portable to PostgreSQL or SQLite.
- LIMIT is not supported in SQL Server or Oracle (use TOP or FETCH FIRST).

**Annotated Code Examples:**

```sql
-- Setup: Create a sample table
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10,2)
);

INSERT INTO products VALUES (1, 'Widget', 19.99);
INSERT INTO products VALUES (2, 'Gadget', 29.99);
INSERT INTO products VALUES (3, 'Gizmo', 9.99);
INSERT INTO products VALUES (4, 'Doohickey', 49.99);
INSERT INTO products VALUES (5, 'Contraption', 39.99);

-- Example 1: LIMIT with one argument
SELECT product_name, price
FROM products
ORDER BY price DESC
LIMIT 3;
```

**Expected Output:**

| product_name | price |
|--------------|-------|
| Doohickey    | 49.99 |
| Contraption  | 39.99 |
| Gadget       | 29.99 |

**Explanation:** The query returns the top 3 most expensive products. ORDER BY ensures a deterministic result.

```sql
-- Example 2: LIMIT with OFFSET
SELECT product_name, price
FROM products
ORDER BY price DESC
LIMIT 2 OFFSET 2;
```

**Expected Output:**

| product_name | price |
|--------------|-------|
| Gadget       | 29.99 |
| Widget       | 19.99 |

**Explanation:** OFFSET 2 skips the first two rows (Doohickey and Contraption), and LIMIT 2 returns the next two.

**Real-World Cases:**

- **E-commerce:** Displaying the first 20 products in a category.
- **Analytics:** Sampling the top 100 highest-revenue customers.
- **APIs:** Implementing offset-based pagination for REST endpoints.

**References:**

- PostgreSQL: LIMIT and OFFSET - https://www.postgresql.org/docs/current/queries-limit.html
- MySQL: SELECT Statement (LIMIT) - https://dev.mysql.com/doc/refman/8.0/en/select.html


#### 1.2 TOP Syntax (Microsoft SQL Server, Sybase)

**Core Definitions:**

- **Core Definition:** TOP restricts the number of rows returned by a query to a specified number or percentage, placed immediately after SELECT.
- **Technical Definition:** In SQL Server, TOP is specified as `SELECT TOP (expression) [PERCENT] [WITH TIES]`. Unlike LIMIT, which comes at the end of the query, TOP appears immediately after SELECT. When TOP is used with ORDER BY, the result set is limited to the first n ordered rows; otherwise, it returns the first n rows in an undefined order.
- **Beginner-Friendly Explanation:** TOP is SQL Server's way of saying "give me only the first N rows." You put it right after SELECT.

**Purposes:**

- To restrict the number of rows returned by a query
- To retrieve a percentage of rows using PERCENT
- To include tied values using WITH TIES
- To limit rows affected by INSERT, UPDATE, DELETE, or MERGE

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
SELECT TOP (expression) [PERCENT] [WITH TIES] column_list
FROM table_name
[ORDER BY ...];
```

**Component Breakdown:**

- `expression`: Number of rows to return (or percentage if PERCENT is specified)
- `PERCENT`: Returns the top expression% of rows
- `WITH TIES`: Returns additional rows that tie with the last row in the limited set

**Syntax Rules:**

- TOP must be used with SELECT, INSERT, UPDATE, DELETE, or MERGE.
- TOP with PERCENT rounds fractional row counts up to the next integer.
- TOP WITH TIES requires ORDER BY and may return more rows than specified.
- Microsoft recommends always using ORDER BY with TOP for predictable results.
- Parentheses are optional for integer constants but recommended.

**Constraints and Limitations:**

- TOP is not ANSI standard.
- TOP WITH TIES cannot be used without ORDER BY.
- TOP is not supported in PostgreSQL, MySQL, or Oracle.

**Annotated Code Examples:**

```sql
-- SQL Server: TOP with integer
SELECT TOP 3 product_name, price
FROM products
ORDER BY price DESC;
```

**Expected Output:** Same as LIMIT 3 example above.

```sql
-- SQL Server: TOP with PERCENT
SELECT TOP 50 PERCENT product_name, price
FROM products
ORDER BY price DESC;
-- Returns 50% of rows (rounded up)
```

```sql
-- SQL Server: TOP WITH TIES
SELECT TOP 3 WITH TIES product_name, price
FROM products
ORDER BY price DESC;
-- If multiple products have the same price as the 3rd product, all are returned
```

**Real-World Cases:**

- **Reporting:** Showing the top 10 customers by revenue.
- **Data Preview:** Quickly inspecting the first 100 rows of a large table.
- **Batch Processing:** Limiting the number of rows affected by an UPDATE or DELETE.

**References:**

- Microsoft Learn: TOP (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/top-transact-sql
- Microsoft Learn: Sort your results - https://learn.microsoft.com/en-us/training/modules/sort-filter-queries/2-sort-your-data-as-its-returned


#### 1.3 ANSI Standard FETCH FIRST n ROWS ONLY Syntax

**Core Definitions:**

- **Core Definition:** FETCH FIRST is the ANSI SQL standard clause for limiting rows, part of the row_limiting_clause introduced in SQL:2008.
- **Technical Definition:** The standard syntax is `OFFSET m ROWS FETCH { FIRST | NEXT } n ROWS { ONLY | WITH TIES }`. It appears after ORDER BY. Oracle, DB2, PostgreSQL, and SQL Server (2012+) support this syntax. The FETCH clause specifies the number of rows or percentage of rows to return.
- **Beginner-Friendly Explanation:** FETCH FIRST is the "standard" way to limit rows. It's more verbose than LIMIT but works across more databases.

**Purposes:**

- To provide a portable, standard-compliant way to limit rows
- To combine row skipping (OFFSET) with row limiting (FETCH)
- To include tied rows using WITH TIES

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
SELECT column_list
FROM table_name
ORDER BY sort_expression
[OFFSET m { ROW | ROWS }]
FETCH { FIRST | NEXT } n { ROW | ROWS } { ONLY | WITH TIES };
```

**Component Breakdown:**

- `OFFSET m ROWS`: Skip m rows before starting to return rows
- `FETCH FIRST | NEXT n ROWS`: Return n rows
- `ONLY`: Return exactly n rows
- `WITH TIES`: Also return rows that tie with the last row

**Syntax Rules:**

- OFFSET is optional; FETCH is required if OFFSET is used (in some databases).
- FIRST and NEXT are interchangeable; ROW and ROWS are interchangeable.
- FETCH FIRST without ORDER BY produces non-deterministic results.
- The OFFSET clause is required if FETCH is used in SQL Server.

**Constraints and Limitations:**

- FETCH FIRST is not supported in MySQL (use LIMIT).
- SQL Server requires OFFSET when using FETCH (cannot use FETCH alone).
- Oracle supports FETCH FIRST n ROWS ONLY; older Oracle versions use ROWNUM.

**Annotated Code Examples:**

```sql
-- Oracle: FETCH FIRST
SELECT product_name, price
FROM products
ORDER BY price DESC
FETCH FIRST 3 ROWS ONLY;
```

**Expected Output:** Same as LIMIT 3 example above.

```sql
-- SQL Server: OFFSET-FETCH
SELECT product_name, price
FROM products
ORDER BY price DESC
OFFSET 0 ROWS
FETCH NEXT 3 ROWS ONLY;
```

```sql
-- Oracle: FETCH FIRST WITH TIES
SELECT product_name, price
FROM products
ORDER BY price DESC
FETCH FIRST 3 ROWS WITH TIES;
```

**Real-World Cases:**

- **Cross-database applications:** Writing portable SQL that works on Oracle, PostgreSQL, and SQL Server.
- **Enterprise reporting:** Standardized pagination across heterogeneous database environments.

**References:**

- Oracle: Row Limiting Clause - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SELECT.html
- Microsoft Learn: Page results (OFFSET-FETCH) - https://learn.microsoft.com/en-us/training/modules/sort-filter-queries/4-page-results
- PostgreSQL: LIMIT and OFFSET - https://www.postgresql.org/docs/current/queries-limit.html


### 2. Pagination Mechanics

#### 2.1 Skipping Rows Using the OFFSET Clause

**Core Definitions:**

- **Core Definition:** OFFSET skips a specified number of rows before the database begins returning rows.
- **Technical Definition:** OFFSET is part of the ORDER BY clause (in SQL Server) or the SELECT statement (in PostgreSQL, Oracle). It specifies the number of rows to skip before row limiting begins.
- **Beginner-Friendly Explanation:** OFFSET is how you get to page 2, 3, or 4. It says "skip the first N rows" so you can start from where the previous page ended.

**Purposes:**

- To implement pagination by skipping rows from previous pages
- To combine with LIMIT or FETCH to retrieve a specific page
- To skip a fixed number of rows for data sampling

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- PostgreSQL/MySQL
SELECT ... ORDER BY ... LIMIT row_count OFFSET offset;

-- SQL Server
SELECT ... ORDER BY ... OFFSET offset ROWS FETCH NEXT row_count ROWS ONLY;

-- Oracle
SELECT ... ORDER BY ... OFFSET offset ROWS FETCH NEXT row_count ROWS ONLY;
```

**Component Breakdown:**

- `offset`: Number of rows to skip (0-based)
- Must be a non-negative integer or expression

**Syntax Rules:**

- OFFSET without ORDER BY is non-deterministic.
- Oracle treats negative offsets as 0; if offset is NULL or exceeds the number of rows, 0 rows are returned.
- OFFSET 0 is the same as omitting the OFFSET clause.
- MySQL allows `LIMIT offset, row_count` as an alternative syntax.

**Constraints and Limitations:**

- Large OFFSET values cause severe performance degradation because the database must scan and discard all skipped rows.
- OFFSET requires ORDER BY for deterministic results.

**Annotated Code Examples:**

```sql
-- Page 1: first 10 rows
SELECT product_name, price FROM products ORDER BY product_id LIMIT 10 OFFSET 0;

-- Page 2: next 10 rows
SELECT product_name, price FROM products ORDER BY product_id LIMIT 10 OFFSET 10;

-- Page 3: next 10 rows
SELECT product_name, price FROM products ORDER BY product_id LIMIT 10 OFFSET 20;
```

**Expected Output (Page 2):** Rows 11–20 from the sorted result set.

**Explanation:** Each page query skips the rows from previous pages. OFFSET 10 skips the first 10 rows; OFFSET 20 skips the first 20.

**Real-World Cases:**

- **Web pagination:** Displaying search results page by page.
- **Data exports:** Processing large tables in batches.
- **Administrative UIs:** Navigating through large lists of records.

**References:**

- PostgreSQL: LIMIT and OFFSET - https://www.postgresql.org/docs/current/queries-limit.html
- Microsoft Learn: Page results - https://learn.microsoft.com/en-us/training/modules/sort-filter-queries/4-page-results


#### 2.2 The Unified ANSI Standard OFFSET m ROWS FETCH NEXT n ROWS ONLY Pattern

**Core Definitions:**

- **Core Definition:** The OFFSET-FETCH pattern combines row skipping and row limiting in a single, standard-compliant clause.
- **Technical Definition:** `OFFSET m ROWS FETCH NEXT n ROWS ONLY` is the ANSI SQL standard for pagination. It is technically part of the ORDER BY clause and enables returning only a range of rows from a sorted result set.
- **Beginner-Friendly Explanation:** This is the "standard" pagination pattern that works on Oracle, SQL Server, DB2, and modern PostgreSQL. It's more verbose than LIMIT/OFFSET but portable.

**Purposes:**

- To provide a portable pagination syntax across major databases
- To combine skipping and limiting in one clause
- To support deterministic, page-based navigation

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
SELECT column_list
FROM table_name
ORDER BY sort_expression
OFFSET m ROWS
FETCH NEXT n ROWS ONLY;
```

**Component Breakdown:**

- `m`: Number of rows to skip (can be 0)
- `n`: Number of rows to return
- `ONLY`: Required keyword (returns exactly n rows unless fewer exist)

**Syntax Rules:**

- OFFSET is required when using FETCH in SQL Server.
- The FETCH clause is optional; if omitted, all rows after OFFSET are returned.
- ROW/ROWS and FIRST/NEXT are interchangeable.
- ORDER BY must provide unique ordering for deterministic pagination.

**Annotated Code Examples:**

```sql
-- SQL Server 2012+: Page 1
SELECT ProductID, ProductName, ListPrice
FROM Production.Product
ORDER BY ListPrice DESC
OFFSET 0 ROWS
FETCH NEXT 10 ROWS ONLY;

-- Page 2
SELECT ProductID, ProductName, ListPrice
FROM Production.Product
ORDER BY ListPrice DESC
OFFSET 10 ROWS
FETCH NEXT 10 ROWS ONLY;
```

**Expected Output:** Page 1 returns the 10 most expensive products; Page 2 returns products 11–20.

**Real-World Cases:**

- **Enterprise applications:** Consistent pagination across SQL Server, Oracle, and PostgreSQL.
- **API development:** Standardized pagination parameters (offset, page_size).

**References:**

- Microsoft Learn: Page results - https://learn.microsoft.com/en-us/training/modules/sort-filter-queries/4-page-results
- Oracle: Row Limiting Clause - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SELECT.html


#### 2.3 State Management Implications for Pagination

**Core Definitions:**

- **Core Definition:** Pagination state management refers to how the application tracks which page the user is on and how to retrieve the next/previous page. Two main approaches exist: stateless offset-based pagination and stateful cursor-based (keyset) pagination.
- **Technical Definition:** Stateless pagination (OFFSET/LIMIT) relies on the client providing the page number or offset, with no server-side state. Cursor-based (keyset) pagination uses a cursor value from the last row of the previous page to retrieve the next page, providing stable, O(1) performance regardless of page depth.
- **Beginner-Friendly Explanation:** With offset pagination, you tell the database "skip 100 rows and give me 10." With keyset pagination, you say "give me 10 rows after this specific row." The second approach is much faster for deep pages.

**Purposes:**

- To understand the trade-offs between offset and keyset pagination
- To choose the appropriate pagination strategy for an application
- To avoid the performance and consistency pitfalls of large OFFSET values

**Comparison of Pagination Strategies:**

| Dimension | Offset Pagination | Keyset (Cursor) Pagination |
|-----------|-------------------|---------------------------|
| Performance model | O(n) — degrades linearly with page depth | O(1) — constant time regardless of depth |
| Page 50,000 latency | 87 ms (unacceptable in production) | Sub-millisecond with composite index |
| Data consistency | Rows can be skipped or duplicated on insert/delete | Stable — position based on actual column values |
| Random page access | Supported (OFFSET n) | Not supported — sequential traversal only |
| Index requirement | Basic single-column index sufficient | Composite index matching keyset column order |
| Best for | Small datasets, admin UIs with page numbers | APIs, data sync, infinite scroll, large tables |

**Annotated Code Examples:**

```sql
-- Offset pagination (page 3, 20 rows per page)
SELECT product_id, product_name, price
FROM products
ORDER BY product_id
LIMIT 20 OFFSET 40;  -- Page 3: skip 40 rows

-- Keyset pagination (next page after last_id = 40)
SELECT product_id, product_name, price
FROM products
WHERE product_id > 40
ORDER BY product_id
LIMIT 20;
```

**Explanation:** Offset pagination must scan and discard 40 rows before returning the page. Keyset pagination directly seeks to the first row after the cursor value, performing a range scan with constant performance.

```sql
-- Keyset pagination with multiple sort columns
-- First page
SELECT id, created_at FROM orders ORDER BY created_at DESC, id DESC LIMIT 20;

-- Next page (using last row's values)
SELECT id, created_at FROM orders
WHERE (created_at, id) < ('2024-06-01 12:00:00', 5432)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

**Real-World Cases:**

- **Social media feeds:** Infinite scroll using cursor-based pagination.
- **E-commerce:** Product listings with page numbers (offset) vs. "load more" (cursor).
- **API design:** Cursor-based pagination for stable, performant data retrieval.

**References:**

- Tinybird: Cursor Pagination Guide - https://guides.tinybird.co/docs/forward/guides/sorting-key-pagination
- Stacksync: PostgreSQL Keyset Pagination vs Offset - https://www.stacksync.com/blog/keyset-cursors-postgres-pagination-fast-accurate-scalable
- Microsoft Learn: Implement Pagination - https://learn.microsoft.com/en-gb/sql/connect/python/mssql-python/pagination


### 3. Critical Operational Rules

#### 3.1 The Non-Deterministic Trap: Why Limiting Without ORDER BY Is Unsafe

**Core Definitions:**

- **Core Definition:** Using LIMIT, TOP, or FETCH without an ORDER BY clause produces a non-deterministic result set — the database may return any rows in any order.
- **Technical Definition:** Without ORDER BY, the row order is unspecified and dependent on the query plan, table storage order, and other factors. A LIMIT without ORDER BY returns an unpredictable subset of rows. PostgreSQL documentation warns: "you will get an unpredictable subset of the query's rows... the ordering is unknown, unless you specified ORDER BY".
- **Beginner-Friendly Explanation:** If you ask for "the first 10 rows" without saying how to sort them, the database might give you 10 completely different rows each time you run the query. Always use ORDER BY.

**Purposes:**

- To understand why ORDER BY is mandatory for predictable row limiting
- To avoid bugs caused by non-deterministic queries
- To ensure consistent pagination results

**Annotated Code Examples:**

```sql
-- DANGEROUS: No ORDER BY
SELECT product_name FROM products LIMIT 3;
-- May return any 3 products, in any order

-- SAFE: With ORDER BY
SELECT product_name FROM products ORDER BY price DESC LIMIT 3;
-- Always returns the 3 most expensive products
```

**Expected Output (Dangerous Query):** The output may vary between executions.

**Expected Output (Safe Query):** Consistent — the top 3 by price.

**Real-World Cases:**

- **Production bugs:** Non-deterministic queries cause intermittent, hard-to-reproduce issues.
- **Pagination errors:** Pages may show duplicate or missing rows if ORDER BY is not unique.
- **Testing:** Non-deterministic queries produce flaky tests.

**References:**

- PostgreSQL: LIMIT and OFFSET (ORDER BY requirement) - https://www.postgresql.org/docs/current/queries-limit.html
- Microsoft Learn: TOP (ORDER BY best practice) - https://learn.microsoft.com/en-us/sql/t-sql/queries/top-transact-sql


#### 3.2 Handling Ties: TOP n WITH TIES and FETCH FIRST n ROWS WITH TIES

**Core Definitions:**

- **Core Definition:** WITH TIES includes additional rows that match the sort key of the last row in the limited result set.
- **Technical Definition:** When using TOP n WITH TIES or FETCH FIRST n ROWS WITH TIES, the query returns all rows that tie with the nth row according to the ORDER BY specification. This may result in more than n rows being returned.
- **Beginner-Friendly Explanation:** If you ask for the top 3 products and there's a tie for 3rd place, WITH TIES includes all tied products, so you might get 4 or 5 rows instead of 3.

**Purposes:**

- To include tied values in Top-N queries
- To avoid arbitrarily excluding rows that are equally ranked
- To provide fair, complete rankings

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- SQL Server
SELECT TOP n WITH TIES column_list FROM table_name ORDER BY sort_expression;

-- Oracle/PostgreSQL (ANSI)
SELECT column_list FROM table_name ORDER BY sort_expression FETCH FIRST n ROWS WITH TIES;
```

**Component Breakdown:**

- `WITH TIES`: Returns all rows that match the last row's sort key
- Requires ORDER BY

**Syntax Rules:**

- WITH TIES cannot be used without ORDER BY.
- The result may contain more rows than specified.
- SQL Server's TOP WITH TIES and Oracle's FETCH FIRST WITH TIES behave similarly.

**Annotated Code Examples:**

```sql
-- SQL Server: TOP 3 WITH TIES
-- Suppose products have prices: 49.99, 39.99, 29.99, 29.99, 19.99
SELECT TOP 3 WITH TIES product_name, price
FROM products
ORDER BY price DESC;
-- Returns 4 rows (including both products priced at 29.99)
```

**Expected Output:**

| product_name | price |
|--------------|-------|
| Doohickey    | 49.99 |
| Contraption  | 39.99 |
| Gadget       | 29.99 |
| Widget       | 29.99 |

**Explanation:** TOP 3 would return only 3 rows, but WITH TIES includes the 4th row because it ties with the 3rd row's price (29.99).

```sql
-- Oracle: FETCH FIRST 3 ROWS WITH TIES
SELECT product_name, price
FROM products
ORDER BY price DESC
FETCH FIRST 3 ROWS WITH TIES;
```

**Real-World Cases:**

- **Leaderboards:** Including all players tied for a ranking position.
- **Sales reports:** Showing all top-performing products with the same revenue.
- **Academic grading:** Including all students with the same top score.

**References:**

- Microsoft Learn: TOP WITH TIES - https://learn.microsoft.com/en-us/sql/t-sql/queries/top-transact-sql
- Oracle: FETCH FIRST WITH TIES - https://docs.oracle.com/en/database/oracle/oracle-database/26/dwhsg/database-data-warehousing-guide.pdf


### 4. Performance & Engine Optimization

#### 4.1 The Deep-Page Performance Bottleneck

**Core Definitions:**

- **Core Definition:** Deep pagination (large OFFSET values) causes severe performance degradation because the database must scan and discard all rows before the offset.
- **Technical Definition:** `LIMIT 20 OFFSET 100000` requires the database to read and discard 100,000 rows before returning 20. This scales linearly — page 10,000 is 10,000 times slower than page 1. Even with an index, the database must read 100,000 index entries.
- **Beginner-Friendly Explanation:** Skipping 100,000 rows means the database has to count through 100,000 rows and throw them away before it can give you the ones you want. That takes time.

**Purposes:**

- To understand why large OFFSET queries are slow
- To justify using keyset pagination for deep pagination
- To identify when offset pagination is acceptable (small datasets)

**Performance Comparison:**

| Page Depth | OFFSET Query Time | Keyset Query Time |
|------------|-------------------|-------------------|
| Page 1 | 468 microseconds | Sub-millisecond |
| Page 50 | 1 millisecond | Sub-millisecond |
| Page 50,000 | 87 milliseconds | Sub-millisecond |

**Explanation:** Offset pagination degrades linearly with page depth. Keyset pagination maintains constant performance because it uses a WHERE clause to seek directly to the next row.

**Annotated Code Examples:**

```sql
-- Slow: Large OFFSET
SELECT * FROM orders ORDER BY id LIMIT 20 OFFSET 100000;
-- MySQL scans 100,000 rows, discards them, returns 20

-- Fast: Keyset pagination (using last_id = 100000)
SELECT * FROM orders WHERE id > 100000 ORDER BY id LIMIT 20;
-- Range scan starting from id > 100000, no rows discarded
```

**Real-World Cases:**

- **API deep pagination:** Users navigating to page 500+ of search results.
- **Data exports:** Batch processing large tables in chunks.
- **Log analysis:** Scanning through millions of log entries.

**References:**

- OneUptime: Optimize LIMIT with Large Offsets in MySQL - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-optimize-limit-large-offsets/README.md
- Stacksync: Keyset Pagination Performance - https://www.stacksync.com/blog/keyset-cursors-postgres-pagination-fast-accurate-scalable


#### 4.2 How Indexes Optimize Limit Operations (Top-N Optimization)

**Core Definitions:**

- **Core Definition:** Top-N optimization allows the database to stop scanning as soon as it has found the required number of rows, using an index to avoid sorting.
- **Technical Definition:** When a query has ORDER BY and LIMIT (or TOP/FETCH), the database can use an index that matches the ORDER BY to read rows in the required order and stop after n rows. MySQL documentation states: "If you combine LIMIT row_count with ORDER BY, MySQL stops sorting as soon as it has found the first row_count rows of the sorted result, rather than sorting the entire result. If ordering is done by using an index, this is very fast".
- **Beginner-Friendly Explanation:** If the data is already stored in the order you want (via an index), the database can just read the first 10 rows and stop. No need to sort the entire table.

**Purposes:**

- To understand how indexes can eliminate sorting overhead
- To design indexes that support Top-N queries
- To write queries that leverage early termination

**Index Usage Rules:**

- The index must match the ORDER BY column(s) and direction(s).
- If the query mixes ASC and DESC, the index must also have mixed directions.
- If the index doesn't cover all selected columns, the database may choose a table scan instead.

**Annotated Code Examples:**

```sql
-- Create an index on price
CREATE INDEX idx_products_price ON products(price DESC);

-- This query can use the index to avoid sorting
SELECT product_name, price
FROM products
ORDER BY price DESC
LIMIT 3;
-- The database reads the first 3 index entries and stops
```

**Expected Performance:**

| Query | Without Index | With Index |
|-------|---------------|------------|
| `ORDER BY price LIMIT 10` | Full scan + sort | Index scan, early termination |
| `ORDER BY price LIMIT 10 OFFSET 1000` | Full scan + sort | Index scan, still fast |

**Real-World Cases:**

- **Leaderboards:** Top 10 scores, backed by an index on score.
- **E-commerce:** Top-selling products, backed by an index on sales count.
- **Recent activity:** Latest 20 events, backed by an index on timestamp.

**References:**

- MySQL: LIMIT Query Optimization - https://dev.mysql.com/doc/refman/8.4/en/limit-optimization.html
- PostgreSQL: LIMIT Optimization - https://www.postgresql.org/message-id/200201270419.g0R4JLD02642@candle.pha.pa.us


### 5. Vendor Implementations & Dialect Translation

#### 5.1 Direct Syntactic Mapping Across Major Engines

**Core Definitions:**

- **Core Definition:** Different database vendors implement row limiting with different syntax. Understanding the mapping allows writing portable SQL or translating between dialects.
- **Technical Definition:** The three main syntax families are LIMIT (PostgreSQL, MySQL, SQLite), TOP (SQL Server, Sybase), and FETCH FIRST (ANSI standard, Oracle, DB2, PostgreSQL, SQL Server 2012+).

**Complete Syntax Comparison Table:**

| Database | Row Limiting Syntax | Pagination Syntax |
|----------|---------------------|-------------------|
| PostgreSQL | `LIMIT n` or `FETCH FIRST n ROWS ONLY` | `LIMIT n OFFSET m` or `OFFSET m ROWS FETCH NEXT n ROWS ONLY` |
| MySQL | `LIMIT n` or `LIMIT m, n` | `LIMIT n OFFSET m` or `LIMIT m, n` |
| SQLite | `LIMIT n` | `LIMIT n OFFSET m` |
| SQL Server | `SELECT TOP (n) ...` | `ORDER BY ... OFFSET m ROWS FETCH NEXT n ROWS ONLY` |
| Oracle | `FETCH FIRST n ROWS ONLY` | `OFFSET m ROWS FETCH NEXT n ROWS ONLY` |
| DB2 | `FETCH FIRST n ROWS ONLY` | `OFFSET m ROWS FETCH NEXT n ROWS ONLY` |

**Annotated Code Examples:**

```sql
-- PostgreSQL / MySQL / SQLite: First 5 rows
SELECT * FROM products ORDER BY price DESC LIMIT 5;

-- SQL Server: First 5 rows
SELECT TOP (5) * FROM products ORDER BY price DESC;

-- Oracle / DB2 / ANSI: First 5 rows
SELECT * FROM products ORDER BY price DESC FETCH FIRST 5 ROWS ONLY;
```

**Dialect Translation Examples:**

```sql
-- MySQL: LIMIT 10 OFFSET 20
-- PostgreSQL: LIMIT 10 OFFSET 20
-- SQL Server: OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY
-- Oracle: OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY
-- DB2: OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY
```

**Real-World Cases:**

- **Multi-database applications:** Writing ORM code that generates the correct syntax per database.
- **Database migration:** Converting queries from MySQL to SQL Server or Oracle.
- **Query builders:** Tools like jOOQ or Hibernate automatically translate LIMIT/TOP/FETCH.

**References:**

- GeeksforGeeks: SQL TOP, LIMIT, FETCH FIRST Clause - https://origin.geeksforgeeks.org/sql/sql-top-limit-fetch-first-clause/
- Stack Overflow: Difference in limit syntax - https://stackoverflow.com/questions/45036438


## Summary Tables

### Row Limiting Syntax Across Databases

| Feature | PostgreSQL | MySQL | SQLite | SQL Server | Oracle | DB2 |
|---------|-----------|-------|--------|------------|--------|-----|
| LIMIT | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ |
| TOP | ✗ | ✗ | ✗ | ✓ | ✗ | ✗ |
| FETCH FIRST | ✓ | ✗ | ✗ | ✓ (2012+) | ✓ | ✓ |
| OFFSET | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| WITH TIES | ✓ | ✗ | ✗ | ✓ | ✓ | ✓ |
| PERCENT | ✗ | ✗ | ✗ | ✓ | ✓ | ✗ |

### Pagination Strategy Comparison

| Aspect | Offset Pagination | Keyset Pagination |
|--------|-------------------|-------------------|
| Syntax | `LIMIT n OFFSET m` | `WHERE key > cursor ORDER BY key LIMIT n` |
| Performance | O(n) — degrades linearly | O(1) — constant time |
| Data consistency | Unstable (skips/duplicates on changes) | Stable |
| Random page access | Supported | Not supported |
| Index requirement | Basic | Composite |
| Best for | Small datasets, admin UIs | APIs, large datasets, infinite scroll |

---

## References

- PostgreSQL: LIMIT and OFFSET - https://www.postgresql.org/docs/current/queries-limit.html
- MySQL: LIMIT Query Optimization - https://dev.mysql.com/doc/refman/8.4/en/limit-optimization.html
- Microsoft Learn: TOP (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/top-transact-sql
- Microsoft Learn: Page results (OFFSET-FETCH) - https://learn.microsoft.com/en-us/training/modules/sort-filter-queries/4-page-results
- Microsoft Learn: Implement Pagination with mssql-python - https://learn.microsoft.com/en-gb/sql/connect/python/mssql-python/pagination
- Oracle: Database Data Warehousing Guide (Row Limiting) - https://docs.oracle.com/en/database/oracle/oracle-database/26/dwhsg/database-data-warehousing-guide.pdf
- GeeksforGeeks: SQL TOP, LIMIT, FETCH FIRST Clause - https://origin.geeksforgeeks.org/sql/sql-top-limit-fetch-first-clause/
- Tinybird: Cursor Pagination Guide - https://guides.tinybird.co/docs/forward/guides/sorting-key-pagination
- Stacksync: PostgreSQL Keyset Pagination vs Offset - https://www.stacksync.com/blog/keyset-cursors-postgres-pagination-fast-accurate-scalable
- OneUptime: Optimize LIMIT with Large Offsets in MySQL - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-optimize-limit-large-offsets/README.md
- Stack Overflow: Difference in limit syntax - https://stackoverflow.com/questions/45036438