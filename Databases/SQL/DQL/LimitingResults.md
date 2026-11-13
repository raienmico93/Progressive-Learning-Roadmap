# Comprehensive Programming Cheat Sheet: SQL Limiting Results

## Topic Overview

### Definitions

**Core Definition**
Limiting results is the set of SQL clauses and techniques—`LIMIT`, `OFFSET`, `FETCH FIRST`, `FETCH NEXT`, `TOP`, and `OFFSET ... FETCH`—used to restrict the number of rows returned by a query and to skip a specified number of rows, thereby enabling top‑N queries, pagination, and bounded result sets.

**Technical Definition**
Result limiting is expressed either as a **standalone clause** (`LIMIT`, `TOP`) or as a **row‑limiting clause** attached to `ORDER BY` (`OFFSET ... FETCH`). PostgreSQL supports both `LIMIT` and the SQL:2008 `OFFSET ... FETCH` syntax; MySQL supports `LIMIT` with an optional offset; SQL Server supports `TOP` (legacy, proprietary) and, since SQL Server 2012, the ANSI‑compliant `OFFSET ... FETCH` syntax; Oracle supports `OFFSET ... FETCH` since Oracle 12*c*. The SQL:2008 standard defines `[ OFFSET start { ROW | ROWS } ] [ FETCH { FIRST | NEXT } [ count ] { ROW | ROWS } { ONLY | WITH TIES } ]`, which is the most portable form across modern RDBMSs. Pagination—the practice of retrieving results in pages—is built on top of these primitives, using either **offset pagination** (skip N, take M), **keyset (seek) pagination** (start after a known key), or **cursor pagination** (an opaque cursor encoding the position).

**Beginner-Friendly Explanation**
Limiting results is how you stop a query from returning everything. A table might have ten million rows, but a web page only shows twenty at a time. `LIMIT 20` says “give me twenty rows.” `OFFSET 40` says “skip the first forty rows first.” Together they implement pagination: page 1 is `LIMIT 20 OFFSET 0`, page 2 is `LIMIT 20 OFFSET 20`, and so on. Different databases spell this differently—MySQL and PostgreSQL use `LIMIT`, SQL Server uses `TOP` or `OFFSET ... FETCH`, Oracle uses `OFFSET ... FETCH`—but the idea is the same: bound the number of rows and optionally choose where to start. The crucial performance lesson is that offset pagination gets slower the deeper you go, because the database must scan and discard every skipped row; keyset and cursor pagination avoid this by starting from a known position instead of counting from the beginning.

### Key Characteristics

- **Without `ORDER BY`, limiting is non‑deterministic**: `LIMIT`, `TOP`, and `OFFSET ... FETCH` all depend on a deterministic row order. `TOP` with no `ORDER BY` “is non‑deterministic and may return any rows up to the number specified”; the same applies to `LIMIT` without `ORDER BY`. Limiting without ordering is only safe when you genuinely do not care which rows you get.
- **`FETCH FIRST` and `FETCH NEXT` are synonyms**: The SQL standard defines both spellings, and they behave identically; `FETCH NEXT` is conventionally used after an `OFFSET` for readability.
- **`TOP` is legacy and proprietary**: `TOP` appears in the `SELECT` list (or after `SELECT`), supports `PERCENT` and `WITH TIES`, and cannot be combined with `OFFSET ... FETCH` in the same query expression.
- **`OFFSET ... FETCH` is the ANSI‑standard form**: Introduced in SQL:2008, supported by PostgreSQL, SQL Server 2012+, and Oracle 12*c*+, it is the most portable modern syntax.
- **Offset pagination is O(n) in page depth**: The database must scan and discard every skipped row, so page 50,000 is dramatically slower than page 1; keyset pagination is O(1) or O(log n) because it seeks directly to the starting key.

### Prerequisites

- A running RDBMS (PostgreSQL, MySQL, SQL Server, or Oracle) and a client connection.
- At least one table with data, or the ability to run `SELECT` with literal values.
- Familiarity with `SELECT`, `FROM`, and `ORDER BY`.
- An understanding of indexes, because pagination performance depends heavily on whether the sort and filter columns are indexed.
- Awareness of the difference between **offset pagination** (simple but slow at depth) and **keyset/cursor pagination** (faster and more consistent but less flexible for random page access).

### Related Programming Areas

- Query optimisation – indexes, sort avoidance, and the cost of `OFFSET`.
- Application development – API pagination, infinite scroll, and page‑number UIs.
- Data export and reporting – top‑N reports and bounded extracts.
- Concurrency – consistency of paginated results when the underlying data changes.
- SQL standardisation – portability across PostgreSQL, MySQL, SQL Server, and Oracle.

---

## Core Concepts and Features

### 1. `LIMIT`

#### Definitions

**Core Definition**
`LIMIT` is a PostgreSQL and MySQL clause that restricts the number of rows returned by a query to at most a specified count.

**Technical Definition**
In PostgreSQL, the `LIMIT` clause consists of `LIMIT { count | ALL }`, where `ALL` means no limit. In MySQL, the clause is `LIMIT {[offset,] row_count | row_count OFFSET offset}`, and `LIMIT row_count` is equivalent to `LIMIT 0, row_count`. `LIMIT` is applied **last**, after `ORDER BY` and `OFFSET`, so it determines which rows survive after sorting and skipping. SQL Server does not support `LIMIT`; Oracle does not support `LIMIT`.

**Beginner-Friendly Explanation**
`LIMIT` is the simplest way to say “only give me this many rows.” It is the most common limiting clause in PostgreSQL and MySQL.

#### Purposes

- To restrict the result set to a maximum number of rows for display or processing.
- To implement top‑N queries when combined with `ORDER BY`.
- To bound the work done by a query when only a few rows are needed.
- To paginate results when combined with `OFFSET`.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
SELECT ... [ ORDER BY ... ] LIMIT { count | ALL }
```

**MySQL:**

```sql
SELECT ... [ ORDER BY ... ] LIMIT {[offset,] row_count | row_count OFFSET offset}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `count` / `row_count` | Maximum number of rows to return. |
| `ALL` (PostgreSQL) | No limit; same as omitting `LIMIT`. |
| `offset` (MySQL) | Number of rows to skip; see `OFFSET`. |

**Syntax rules:**

- `LIMIT` should be paired with `ORDER BY` for deterministic results.
- `LIMIT` is applied after `ORDER BY` and `OFFSET`.

**Constraints and limitations:**

- `LIMIT` is not available in SQL Server or Oracle; use `TOP` or `OFFSET ... FETCH` instead.
- Without `ORDER BY`, the rows returned are unpredictable.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Top‑N with LIMIT and ORDER BY

```sql
-- Step 1: Create a table.
CREATE TABLE employees (
    employee_id serial PRIMARY KEY,
    first_name  text NOT NULL,
    salary      numeric(10,2) NOT NULL
);

-- Step 2: Insert sample rows.
INSERT INTO employees (first_name, salary) VALUES
    ('Alice', 80000), ('Bob', 70000), ('Carol', 60000), ('Dave', 65000);

-- Step 3: Retrieve the top two earners.
SELECT first_name, salary
FROM employees
ORDER BY salary DESC
LIMIT 2;
```

**Expected output:**

```text
 first_name | salary
------------+--------
 Alice      |  80000
 Bob        |  70000
(2 rows)
```

**Why this output appears**: `ORDER BY salary DESC` sorts all four employees from highest to lowest salary, and `LIMIT 2` returns only the first two rows of that sorted result—Alice and Bob.

#### Real-World Cases

- **Leaderboards**: `ORDER BY score DESC LIMIT 10` returns the top ten players.
- **Recent activity**: `ORDER BY created_at DESC LIMIT 20` returns the twenty most recent events.

---

### 2. `OFFSET`

#### Definitions

**Core Definition**
`OFFSET` skips a specified number of rows before the query starts returning rows, enabling pagination.

**Technical Definition**
`OFFSET start { ROW | ROWS }` skips the first `start` rows of the sorted result. In MySQL, the offset is the first argument of a two‑argument `LIMIT` (`LIMIT offset, row_count`) or the `OFFSET` keyword in the `row_count OFFSET offset` form. `OFFSET` is applied after `ORDER BY` and before `LIMIT`/`FETCH`. The cost of `OFFSET` grows linearly with the offset value because the database must scan and discard every skipped row.

**Beginner-Friendly Explanation**
`OFFSET` is the “skip” part of pagination. `LIMIT 20 OFFSET 20` means “skip the first twenty rows, then give me the next twenty.”

#### Purposes

- To implement page‑based pagination (`page N` = `LIMIT page_size OFFSET (N-1)*page_size`).
- To skip a known number of leading rows in a sorted result.
- To preview data beyond the first page during exploration.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
OFFSET start { ROW | ROWS }
```

**MySQL:**

```sql
LIMIT [offset,] row_count
-- or
LIMIT row_count OFFSET offset
```

**SQL Server / Oracle (standard form):**

```sql
OFFSET start { ROW | ROWS }
```

**Syntax rules:**

- `OFFSET` should always be paired with `ORDER BY` for deterministic pagination.
- In PostgreSQL, `OFFSET` and `LIMIT` can appear in either order (PostgreSQL is laxer than the standard, which requires `OFFSET` before `FETCH`).

**Constraints and limitations:**

- Large offsets force the database to scan and discard many rows, making deep pagination slow.
- `OFFSET` without `ORDER BY` is unpredictable.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Offset pagination

```sql
-- Step 1: Retrieve the second page of employees (page size 2).
SELECT first_name, salary
FROM employees
ORDER BY salary DESC
LIMIT 2 OFFSET 2;
```

**Expected output:**

```text
 first_name | salary
------------+--------
 Dave       |  65000
 Carol      |  60000
(2 rows)
```

**Why this output appears**: The query sorts all employees by salary descending, skips the first two (Alice and Bob), and returns the next two (Dave and Carol).

#### Real-World Cases

- **Page‑numbered APIs**: `GET /items?page=3&per_page=20` translates to `LIMIT 20 OFFSET 40`.
- **Admin tables**: A data‑grid UI shows page numbers and uses offset pagination for random page access.

---

### 3. `FETCH FIRST`

#### Definitions

**Core Definition**
`FETCH FIRST` is the SQL:2008 standard clause that limits the result to the first *n* rows of the sorted result.

**Technical Definition**
`FETCH { FIRST | NEXT } [ count ] { ROW | ROWS } ONLY` returns at most `count` rows. `FETCH FIRST` and `FETCH NEXT` are identical in effect; the standard provides both spellings, and `FETCH NEXT` is conventionally used when an `OFFSET` precedes it. `FETCH FIRST` is supported by PostgreSQL, SQL Server 2012+, and Oracle 12*c*+. The standard requires `OFFSET` to come before `FETCH` if both are present, though PostgreSQL allows either order.

**Beginner-Friendly Explanation**
`FETCH FIRST 5 ROWS ONLY` is the standard‑SQL way of saying `LIMIT 5`. It is more portable across modern databases than `LIMIT`.

#### Purposes

- To limit results in a standard‑compliant, portable way.
- To express top‑N queries that work across PostgreSQL, SQL Server, and Oracle.
- To combine with `OFFSET` for standard pagination.

#### Syntax Rules and Structure

```sql
[ OFFSET start { ROW | ROWS } ]
FETCH { FIRST | NEXT } [ count ] { ROW | ROWS } ONLY
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `FIRST` / `NEXT` | Synonyms; `FIRST` for the beginning, `NEXT` after an `OFFSET`. |
| `count` | Number of rows to return; the standard requires a literal, parameter, or variable. |
| `ROW` / `ROWS` | Noise word; both are equivalent. |
| `ONLY` | Return exactly the requested rows (contrast with `WITH TIES`). |

**Syntax rules:**

- `FETCH FIRST` is typically used without `OFFSET`; `FETCH NEXT` is used after `OFFSET`.
- `FETCH` requires `ORDER BY` for deterministic results in practice, though the standard permits it without.

**Constraints and limitations:**

- SQL Server 2012+ requires `ORDER BY` and `OFFSET` together with `FETCH`; `FETCH` alone is not allowed in SQL Server.
- MySQL does not support `FETCH FIRST`; use `LIMIT`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – FETCH FIRST with ORDER BY

```sql
-- Step 1: Retrieve the first three employees by salary.
SELECT first_name, salary
FROM employees
ORDER BY salary DESC
FETCH FIRST 3 ROWS ONLY;
```

**Expected output:**

```text
 first_name | salary
------------+--------
 Alice      |  80000
 Bob        |  70000
 Dave       |  65000
(3 rows)
```

**Why this output appears**: `ORDER BY salary DESC` sorts the employees, and `FETCH FIRST 3 ROWS ONLY` returns exactly the first three rows of that sorted result.

#### Real-World Cases

- **Portable top‑N queries**: An application that runs on PostgreSQL and SQL Server uses `FETCH FIRST` to avoid maintaining two query variants.
- **Standard‑compliant reporting**: SQL:2008‑based tools and ORMs generate `FETCH FIRST` for cross‑database compatibility.

---

### 4. `FETCH NEXT`

#### Definitions

**Core Definition**
`FETCH NEXT` is the SQL:2008 standard clause that returns the next *n* rows after an `OFFSET`, and is semantically identical to `FETCH FIRST`.

**Technical Definition**
`FETCH NEXT [ count ] { ROW | ROWS } ONLY` behaves exactly like `FETCH FIRST`; the two spellings differ only in readability. `FETCH NEXT` is conventionally used when an `OFFSET` has already skipped rows, making the query read naturally as “skip N rows and fetch the next M”.

**Beginner-Friendly Explanation**
`FETCH NEXT` is the same as `FETCH FIRST`; it exists so that a paginated query reads naturally: “OFFSET 20 ROWS FETCH NEXT 20 ROWS ONLY.”

#### Purposes

- To express pagination in standard SQL with a natural reading order.
- To make the intent clear when an `OFFSET` precedes the fetch.
- To maintain portability across PostgreSQL, SQL Server, and Oracle.

#### Syntax Rules and Structure

```sql
OFFSET start { ROW | ROWS }
FETCH NEXT [ count ] { ROW | ROWS } ONLY
```

**Syntax rules:**

- `FETCH NEXT` must follow `OFFSET` in SQL Server; in PostgreSQL and Oracle the order is flexible.
- The effect is identical to `FETCH FIRST`.

**Constraints and limitations:**

- Same as `FETCH FIRST`; not supported in MySQL.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – OFFSET ... FETCH NEXT

```sql
-- Step 1: Retrieve the second page of employees (page size 2).
SELECT first_name, salary
FROM employees
ORDER BY salary DESC
OFFSET 2 ROWS
FETCH NEXT 2 ROWS ONLY;
```

**Expected output:**

```text
 first_name | salary
------------+--------
 Dave       |  65000
 Carol      |  60000
(2 rows)
```

**Why this output appears**: `OFFSET 2 ROWS` skips Alice and Bob, and `FETCH NEXT 2 ROWS ONLY` returns the next two rows in the sorted order—Dave and Carol.

#### Real-World Cases

- **SQL Server pagination**: Since SQL Server 2012, `OFFSET ... FETCH NEXT` is the recommended ANSI‑compliant pagination syntax.
- **Oracle 12c+ pagination**: `OFFSET ... FETCH NEXT` replaces the older `ROWNUM` and `ROW_NUMBER()` workarounds.

---

### 5. `TOP` (SQL Server)

#### Definitions

**Core Definition**
`TOP` is SQL Server’s proprietary clause that limits the number of rows returned (or the percentage of rows) based on the `ORDER BY` clause.

**Technical Definition**
`TOP (n)` appears immediately after `SELECT` (or after `SELECT DISTINCT`) and limits the result to the first *n* rows of the sorted result. `TOP (n) PERCENT` returns a percentage of the rows, and `TOP (n) WITH TIES` includes all rows that tie with the last row of the top‑N set. `TOP` is a legacy keyword still supported for backward compatibility; Microsoft recommends `OFFSET ... FETCH` for new pagination code. `TOP` cannot be combined with `OFFSET ... FETCH` in the same query expression.

**Beginner-Friendly Explanation**
`TOP` is SQL Server’s original “limit” keyword. `SELECT TOP 10 * FROM t ORDER BY score DESC` returns the ten highest scores.

#### Purposes

- To limit results in SQL Server, especially in legacy code.
- To return a percentage of rows (`TOP (n) PERCENT`).
- To include ties with `TOP (n) WITH TIES`.

#### Syntax Rules and Structure

```sql
SELECT [ ALL | DISTINCT ] [ TOP ( expression ) [ PERCENT ] [ WITH TIES ] ] ...
[ ORDER BY ... ]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `TOP (n)` | Return the first *n* rows. |
| `PERCENT` | Interpret *n* as a percentage (1–100). |
| `WITH TIES` | Include additional rows that tie with the last row. |

**Syntax rules:**

- `TOP` without `ORDER BY` is non‑deterministic.
- `TOP` cannot be combined with `OFFSET ... FETCH`.

**Constraints and limitations:**

- `TOP` is SQL Server‑specific and not portable.
- `TOP` does not support an offset; it can only take from the beginning.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – TOP with ties

```sql
-- Step 1: Return the top 2 items by quantity, including ties.
SELECT TOP (2) WITH TIES OrderID, Item, Quantity
FROM OrderItems
ORDER BY Quantity DESC;
```

**Expected output:**

```text
OrderID  Item            Quantity
-------  --------------  --------
3        M6 Locking Nut  300
3        M8 Washer       200
(2 rows, plus any rows tied at 200)
```

**Why this output appears**: `TOP (2) WITH TIES` returns the two highest‑quantity rows and any additional rows whose `Quantity` equals the last row’s value (200), ensuring that ties are not arbitrarily truncated.

#### Real-World Cases

- **Legacy SQL Server code**: Existing applications use `TOP` for top‑N queries and have not migrated to `OFFSET ... FETCH`.
- **Percentage sampling**: `TOP (10) PERCENT` returns roughly ten percent of a table for exploratory analysis.

---

### 6. `OFFSET ... FETCH` (Combined)

#### Definitions

**Core Definition**
`OFFSET ... FETCH` is the combined standard syntax that skips a specified number of rows and then returns a specified number of rows, forming the basis of standard pagination.

**Technical Definition**
The full form is `[ OFFSET start { ROW | ROWS } ] [ FETCH { FIRST | NEXT } [ count ] { ROW | ROWS } { ONLY | WITH TIES } ]`. `OFFSET` specifies the number of rows to skip, and `FETCH` specifies the number of rows to return. The standard requires `OFFSET` before `FETCH` if both are present, and the `count` must be a literal, parameter, or variable. `ONLY` returns exactly the requested rows; `WITH TIES` also includes rows that tie with the last row. This syntax is supported by PostgreSQL, SQL Server 2012+, and Oracle 12*c*+.

**Beginner-Friendly Explanation**
`OFFSET ... FETCH` is the standard pagination syntax: “skip this many rows, then fetch this many rows.” It is the portable equivalent of `LIMIT ... OFFSET`.

#### Purposes

- To implement standard, portable pagination across PostgreSQL, SQL Server, and Oracle.
- To express both the skip and the take in a single, readable clause.
- To use `WITH TIES` to avoid arbitrarily truncating tied results.

#### Syntax Rules and Structure

```sql
[ OFFSET start { ROW | ROWS } ]
[ FETCH { FIRST | NEXT } [ count ] { ROW | ROWS } { ONLY | WITH TIES } ]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `OFFSET start` | Skip `start` rows. |
| `FETCH FIRST|NEXT count` | Return `count` rows. |
| `ONLY` | Exactly `count` rows. |
| `WITH TIES` | Also include rows tied with the last returned row. |

**Syntax rules:**

- In SQL Server, `OFFSET ... FETCH` requires an `ORDER BY` clause.
- In PostgreSQL, `OFFSET` and `FETCH` can appear in either order.
- `FETCH` without `OFFSET` is allowed in PostgreSQL and Oracle; SQL Server requires both together.

**Constraints and limitations:**

- MySQL does not support `OFFSET ... FETCH`; use `LIMIT ... OFFSET` instead.
- The `count` must be a constant, parameter, or variable, not an arbitrary expression.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Standard OFFSET ... FETCH pagination

```sql
-- Step 1: Retrieve the second page of employees (page size 2).
SELECT first_name, salary
FROM employees
ORDER BY salary DESC
OFFSET 2 ROWS
FETCH NEXT 2 ROWS ONLY;
```

**Expected output:**

```text
 first_name | salary
------------+--------
 Dave       |  65000
 Carol      |  60000
(2 rows)
```

**Why this output appears**: `OFFSET 2 ROWS` skips the first two employees in the sorted result, and `FETCH NEXT 2 ROWS ONLY` returns exactly the next two—Dave and Carol.

#### Real-World Cases

- **Cross‑database pagination**: A query that runs on PostgreSQL and SQL Server uses `OFFSET ... FETCH` to avoid maintaining two pagination syntaxes.
- **Oracle 12c+ migration**: Applications migrating from `ROWNUM` to the standard syntax adopt `OFFSET ... FETCH` for clarity and performance.

---

### 7. Vendor Differences

#### Definitions

**Core Definition**
Vendor differences are the syntactic and semantic variations in result‑limiting clauses across PostgreSQL, MySQL, SQL Server, and Oracle.

**Technical Definition**
Each RDBMS supports a different combination of limiting clauses: **PostgreSQL** supports `LIMIT`, `OFFSET`, and the standard `OFFSET ... FETCH`; **MySQL** supports `LIMIT` with an optional offset and does not support `FETCH`; **SQL Server** supports `TOP` and, since 2012, `OFFSET ... FETCH` (but not `LIMIT`); **Oracle** supports `OFFSET ... FETCH` since 12*c* and previously used `ROWNUM`. The SQL:2008 standard defines `OFFSET ... FETCH`, making it the most portable modern syntax. `TOP` supports `PERCENT` and `WITH TIES`; `LIMIT` does not support `PERCENT` or `WITH TIES`; `FETCH` supports `WITH TIES` but not `PERCENT` in the standard (Oracle supports `PERCENT` in its `FETCH` clause).

**Beginner-Friendly Explanation**
Every database has its own dialect for limiting results. If you want portability, `OFFSET ... FETCH` is the safest choice among modern databases. If you are working in MySQL, you use `LIMIT`; in SQL Server legacy code, you use `TOP`; in Oracle 12c+, you use `OFFSET ... FETCH`.

#### Purposes

- To write queries that run correctly on multiple RDBMSs.
- To know which features (percentages, ties) are available in each dialect.
- To migrate queries from one database to another without surprises.

#### Syntax Rules and Structure

| RDBMS | Standalone limit | Offset support | Standard `FETCH` | `PERCENT` | `WITH TIES` |
|-------|------------------|----------------|------------------|-----------|-------------|
| PostgreSQL | `LIMIT` | `OFFSET` | Yes | No | Yes |
| MySQL | `LIMIT` | `LIMIT offset, count` | No | No | No |
| SQL Server | `TOP` | `OFFSET ... FETCH` (2012+) | Yes (2012+) | `TOP ... PERCENT` | `TOP ... WITH TIES` |
| Oracle | `FETCH FIRST/NEXT` (12c+) | `OFFSET` (12c+) | Yes | `FETCH ... PERCENT` | `FETCH ... WITH TIES` |
| SQL standard | — | `OFFSET` | `FETCH FIRST/NEXT` | Oracle‑specific | `FETCH ... WITH TIES` |

**Syntax rules:**

- `LIMIT` is PostgreSQL/MySQL‑specific; `TOP` is SQL Server‑specific; `FETCH` is standard.
- `OFFSET ... FETCH` is the closest thing to a portable pagination syntax.
- `TOP` cannot be combined with `OFFSET ... FETCH` in SQL Server.

**Constraints and limitations:**

- Porting a query between MySQL and SQL Server requires rewriting `LIMIT` ↔ `TOP`/`FETCH`.
- `WITH TIES` is not available in MySQL; emulating it requires a subquery or a join on the boundary value.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: The same top‑N query in four dialects

```sql
-- PostgreSQL / MySQL:
SELECT * FROM employees ORDER BY salary DESC LIMIT 5;

-- SQL Server (legacy):
SELECT TOP (5) * FROM employees ORDER BY salary DESC;

-- SQL Server 2012+ / Oracle 12c+:
SELECT * FROM employees ORDER BY salary DESC
OFFSET 0 ROWS FETCH NEXT 5 ROWS ONLY;
```

**Expected output:** (identical in all four cases—the five highest‑salary employees.)

**Why this output appears**: Each dialect expresses the same intent—return the first five rows of the sorted result—using its own limiting syntax. The standard `OFFSET ... FETCH` form is the most portable among the four.

#### Real-World Cases

- **Multi‑database ORMs**: ORMs abstract these differences, but understanding the underlying syntax helps diagnose generated queries.
- **Database migration**: Moving from MySQL to PostgreSQL requires converting `LIMIT` (which both support) but converting `LIMIT` to `TOP`/`FETCH` when moving to SQL Server.

---

### 8. Pagination Patterns

#### Definitions

**Core Definition**
Pagination patterns are the strategies used to divide a large result set into pages, including **offset pagination**, **keyset (seek) pagination**, and **cursor pagination**.

**Technical Definition**
**Offset pagination** uses `LIMIT`/`OFFSET` (or `OFFSET ... FETCH`) to skip a computed number of rows. **Keyset (seek) pagination** uses a `WHERE` predicate on the last seen sort key (e.g., `WHERE id > last_seen ORDER BY id LIMIT N`) to start directly after the previous page. **Cursor pagination** is an application‑level abstraction that encodes the position (often as an opaque token) and typically implements keyset pagination underneath. Offset pagination is O(n) in page depth; keyset pagination is O(1) or O(log n) because it seeks directly to the starting key.

**Beginner-Friendly Explanation**
There are three ways to paginate. Offset pagination is the simplest: “skip 40, take 20.” It is easy to implement but gets slower the deeper you go. Keyset pagination remembers the last row you saw and starts from there: “give me the next 20 rows after ID 1234.” It stays fast no matter how deep you are. Cursor pagination is like keyset pagination, but the “last seen” information is packaged into an opaque token that the client passes back to the server.

#### Purposes

- To choose the right pagination strategy for the workload (small admin UI vs. large API).
- To avoid the performance cliff of deep offset pagination.
- To maintain correctness when the underlying data changes between pages.

#### Syntax Rules and Structure

**Offset pagination:**

```sql
SELECT ... ORDER BY sort_column
LIMIT page_size OFFSET (page_number - 1) * page_size;
```

**Keyset pagination:**

```sql
-- First page:
SELECT ... ORDER BY sort_column, id LIMIT page_size;
-- Next page (last_seen_sort, last_seen_id from previous page):
SELECT ... WHERE (sort_column, id) > (last_seen_sort, last_seen_id)
ORDER BY sort_column, id LIMIT page_size;
```

**Cursor pagination (application‑level):**

```text
GET /items?cursor=eyJpZCI6MTIzNH0&limit=20
-- Server decodes cursor to {id: 1234}, then:
SELECT ... WHERE id > 1234 ORDER BY id LIMIT 20;
```

**Component breakdown:**

| Pattern | Key mechanism | Strength | Weakness |
|---------|---------------|----------|----------|
| Offset | `OFFSET N` | Random page access; simple | O(n) at depth; inconsistent under changes |
| Keyset | `WHERE key > last_seen` | O(1)/O(log n); consistent | No random page access; needs composite index |
| Cursor | Opaque token | Encapsulates keyset; API‑friendly | Requires application support; less transparent |

**Syntax rules:**

- Offset pagination requires a deterministic `ORDER BY`.
- Keyset pagination requires a **unique** tiebreaker (usually the primary key) in the `ORDER BY` and a composite index matching the sort columns.
- Cursor pagination typically uses keyset underneath; the cursor encodes the last seen values.

**Constraints and limitations:**

- Keyset pagination cannot jump to an arbitrary page; it only supports next/previous traversal.
- Offset pagination can miss or duplicate rows when data is inserted or deleted between page requests.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Offset pagination versus keyset pagination

```sql
-- Offset pagination (page 500, page size 20):
SELECT id, created_at
FROM orders
ORDER BY created_at DESC, id DESC
LIMIT 20 OFFSET 9980;

-- Keyset pagination (after the last row of the previous page):
SELECT id, created_at
FROM orders
WHERE (created_at, id) < ('2025-01-15 10:30:00', 12345)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

**Expected output:** (both return the next 20 rows, but the keyset query seeks directly to the starting position.)

**Why this output appears**: The offset query must scan and discard 9,980 rows before returning the 20 requested rows. The keyset query uses the composite `(created_at, id)` predicate to start exactly after the last seen row, avoiding the scan entirely. On large tables, the keyset query remains fast while the offset query degrades linearly.

#### Real-World Cases

- **Admin UIs**: Use offset pagination when users need to jump to page 50 or page 1,000.
- **Public APIs and infinite scroll**: Use keyset or cursor pagination for consistent, fast traversal of large datasets.
- **Data synchronisation**: Use keyset pagination to stream records without the risk of missing or duplicating rows as the data changes.

---

### 9. Offset Pagination

#### Definitions

**Core Definition**
Offset pagination retrieves pages by skipping a calculated number of rows (`OFFSET`) and taking a fixed page size (`LIMIT`/`FETCH`).

**Technical Definition**
For page number *p* (1‑based) and page size *s*, the query is `LIMIT s OFFSET (p - 1) * s`. The database scans and discards `(p - 1) * s` rows before returning the requested page. This makes the cost **O(n)** in page depth: page 1 is fast, page 50,000 is slow. Offset pagination also suffers from consistency problems: if rows are inserted or deleted between page requests, the same row may appear on two pages or be skipped entirely.

**Beginner-Friendly Explanation**
Offset pagination is the “skip N, take M” approach. It is simple and supports random page access, but it gets slower the deeper you go and can produce inconsistent pages when the data changes.

#### Purposes

- To provide random page access in admin interfaces and reports.
- To implement simple, well‑understood pagination in small datasets.
- To support page‑number navigation where users expect to jump to a specific page.

#### Syntax Rules and Structure

```sql
SELECT ... ORDER BY sort_column
LIMIT page_size OFFSET (page_number - 1) * page_size;
```

**Syntax rules:**

- Always include `ORDER BY`; without it, the pages are not deterministic.
- The offset grows linearly with page number.

**Constraints and limitations:**

- Deep offsets are slow because every skipped row must be scanned.
- Concurrent inserts/deletes can shift rows across page boundaries, causing duplicates or gaps.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Offset pagination performance at depth

```sql
-- Page 1 (fast):
SELECT * FROM users ORDER BY created_at DESC LIMIT 20 OFFSET 0;

-- Page 50,000 (slow):
SELECT * FROM users ORDER BY created_at DESC LIMIT 20 OFFSET 1000000;
```

**Expected output (illustrative timings from benchmarks):**

```text
-- Page 1: ~468 microseconds
-- Page 50,000: ~87 milliseconds
```

**Why this output appears**: The page‑1 query scans almost nothing; the page‑50,000 query must scan and discard one million rows before returning its twenty, which is why the latency grows by orders of magnitude.

#### Real-World Cases

- **Admin tables**: A user‑management screen with page numbers uses offset pagination because users need to jump to arbitrary pages.
- **Small datasets**: For a few thousand rows, the performance difference is negligible and offset pagination is the simplest choice.

---

### 10. Keyset Pagination (Seek Method)

#### Definitions

**Core Definition**
Keyset pagination (also called seek pagination) retrieves the next page by starting after the last seen value of the sort key, using a `WHERE` predicate instead of an `OFFSET`.

**Technical Definition**
Instead of `OFFSET N`, the query uses `WHERE sort_key > last_seen_value ORDER BY sort_key LIMIT page_size` (or `<` for descending order). When the sort key is not unique, a unique tiebreaker (typically the primary key) is included in both the `ORDER BY` and the `WHERE` row comparison. The query can seek directly to the starting position via an index, giving **O(1)** or **O(log n)** performance regardless of page depth. The trade‑off is that keyset pagination supports only sequential traversal; it cannot jump to an arbitrary page number.

**Beginner-Friendly Explanation**
Keyset pagination is the “start after the last row I saw” approach. Instead of counting from the beginning, it jumps straight to the next page. It is much faster at depth, but you cannot use it for “go to page 50.”

#### Purposes

- To paginate large datasets with constant performance regardless of depth.
- To avoid the consistency problems of offset pagination when data changes.
- To implement infinite scroll and API pagination efficiently.

#### Syntax Rules and Structure

```sql
-- First page:
SELECT ... ORDER BY sort_column, id LIMIT page_size;

-- Next page (last_seen_sort, last_seen_id from previous page):
SELECT ... WHERE (sort_column, id) > (last_seen_sort, last_seen_id)
ORDER BY sort_column, id LIMIT page_size;
```

**Syntax rules:**

- The `ORDER BY` must include a unique tiebreaker (usually the primary key) for deterministic results.
- A composite index on `(sort_column, id)` is required for efficient seeks.
- Use `>` for ascending order and `<` for descending order.

**Constraints and limitations:**

- No random page access; only next/previous traversal.
- The client must remember and pass back the last seen values (or an encoded cursor).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Keyset pagination with a composite sort

```sql
-- Step 1: Create a composite index.
CREATE INDEX idx_orders_created_id ON orders (created_at DESC, id DESC);

-- Step 2: First page.
SELECT id, created_at FROM orders
ORDER BY created_at DESC, id DESC
LIMIT 20;

-- Step 3: Next page (last row was created_at = '2025-01-15 10:30:00', id = 12345).
SELECT id, created_at FROM orders
WHERE (created_at, id) < ('2025-01-15 10:30:00', 12345)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

**Expected output:** (the next 20 rows after the last seen row, in the same order.)

**Why this output appears**: The row comparison `(created_at, id) < (...)` seeks directly to the position after the last seen row, using the composite index. No rows are scanned and discarded, so the query remains fast at any depth.

#### Real-World Cases

- **API pagination**: A REST API returns a `next_cursor` containing the last seen `(created_at, id)` values, and the client passes it back for the next page.
- **Data export**: A nightly export streams records in keyset‑paginated batches without the risk of missing rows as new data arrives.

---

### 11. Cursor Pagination

#### Definitions

**Core Definition**
Cursor pagination is an application‑level pagination pattern in which the server returns an opaque **cursor** representing the position in the result set, and the client passes that cursor back to request the next page.

**Technical Definition**
Cursor pagination typically implements keyset pagination underneath: the cursor encodes the last seen sort key values (and often a direction or filters), and the server decodes it to build the `WHERE ... > last_seen` predicate. The cursor is opaque to the client, which allows the server to change the underlying pagination implementation without breaking clients. Cursor pagination provides **O(1)** performance regardless of page depth and is stable under concurrent inserts and deletes. It is the standard approach for public APIs and infinite‑scroll interfaces.

**Beginner-Friendly Explanation**
Cursor pagination is like a bookmark. The server says “here are your first 20 items, and here is a bookmark pointing to the last one.” When the client wants more, it sends the bookmark back, and the server continues from there. The bookmark is opaque—the client does not need to know what is inside it.

#### Purposes

- To provide fast, stable pagination for APIs and infinite‑scroll UIs.
- To hide the underlying keyset implementation behind an opaque token.
- To allow the server to change pagination strategies without breaking clients.
- To avoid the performance and consistency problems of offset pagination.

#### Syntax Rules and Structure

**Server‑side query:**

```sql
-- Decode the cursor to get last_seen_sort and last_seen_id.
SELECT ... WHERE (sort_column, id) > (last_seen_sort, last_seen_id)
ORDER BY sort_column, id LIMIT page_size;
```

**Client‑side request:**

```text
GET /items?cursor=eyJpZCI6MTIzNH0&limit=20
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| Cursor | Opaque token encoding the last seen position. |
| Server decode | Extract the sort key values from the cursor. |
| Keyset query | `WHERE ... > last_seen` with the decoded values. |
| Next cursor | Encode the last row of the returned page for the next request. |

**Syntax rules:**

- The cursor should encode all columns used in the `ORDER BY`, including the unique tiebreaker.
- The cursor can be base64‑encoded JSON, a signed token, or any opaque format.
- The server should validate and reject malformed cursors.

**Constraints and limitations:**

- Requires application‑level support; not a pure SQL feature.
- The cursor format is server‑defined and must remain stable across deployments (or be versioned).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Cursor pagination end‑to‑end

```sql
-- Step 1: Server receives cursor = 'eyJjcmVhdGVkX2F0IjoiMjAyNS0wMS0xNVQxMDozMDowMCIsImlkIjoxMjM0NX0='
-- Decoded: {created_at: '2025-01-15T10:30:00', id: 12345}

-- Step 2: Query the next page using the decoded values.
SELECT id, created_at, total
FROM orders
WHERE (created_at, id) < ('2025-01-15T10:30:00', 12345)
ORDER BY created_at DESC, id DESC
LIMIT 20;

-- Step 3: Server encodes the last row of the result as the next cursor.
-- Next cursor: base64({created_at: '2025-01-14T09:00:00', id: 12000})
```

**Expected output:** (the next 20 orders, plus a `next_cursor` token.)

**Why this output appears**: The server decodes the opaque cursor into the last seen sort values, builds a keyset predicate, and returns the next page along with a new cursor for the following page. The client never needs to know how the cursor is constructed.

#### Real-World Cases

- **REST APIs**: Stripe, GitHub, and Slack use cursor pagination for list endpoints, returning `next_cursor` values in responses.
- **Infinite scroll**: A mobile app loads the next batch of feed items by passing the cursor from the previous response.

---

### 12. Pagination Performance

#### Definitions

**Core Definition**
Pagination performance is the study of how the choice of pagination pattern—offset, keyset, or cursor—affects query latency, database load, and consistency as the page depth and dataset size grow.

**Technical Definition**
Offset pagination has **O(n)** cost in page depth because the database must scan and discard every skipped row. Keyset and cursor pagination have **O(1)** or **O(log n)** cost because they seek directly to the starting key via an index. Benchmarks show that offset pagination at page 50,000 can take tens of milliseconds, while keyset pagination remains sub‑millisecond; at one million rows, offset pagination can take seconds while keyset pagination stays in the low milliseconds. The performance gap widens with dataset size and page depth, and offset pagination also suffers from consistency problems when data changes between page requests.

**Beginner-Friendly Explanation**
Pagination performance is about how fast your pages load. Offset pagination starts fast and gets slower and slower as you go deeper—page 1 is instant, page 50,000 is painful. Keyset and cursor pagination stay fast no matter how deep you go, because they jump straight to the right place instead of counting from the beginning.

#### Purposes

- To choose the right pagination pattern for the dataset size and access pattern.
- To avoid production slowdowns caused by deep offset pagination.
- To understand why a query that was fast in development becomes slow in production.
- To design indexes that support efficient pagination.

#### Syntax Rules and Structure

Performance is determined by the pattern and the indexes, not by a single syntax:

| Pattern | Complexity | Index requirement | Consistency |
|---------|------------|-------------------|-------------|
| Offset | O(n) in page depth | Basic index on sort column | Can skip/duplicate rows on changes |
| Keyset | O(1) / O(log n) | Composite index on `(sort, tiebreaker)` | Stable |
| Cursor | O(1) / O(log n) (keyset underneath) | Composite index on `(sort, tiebreaker)` | Stable |

**Benchmark data (illustrative):**

- Page 1: ~468 microseconds; page 50,000: ~87 milliseconds (offset, PostgreSQL).
- 100K rows, page 1,000: offset ~500 ms; cursor ~5 ms.
- 1M rows, page 1,000: offset ~5 s; cursor ~5 ms.

**Constraints and limitations:**

- Keyset and cursor pagination cannot jump to an arbitrary page number.
- Composite indexes increase write overhead and storage.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Measuring the performance difference

```sql
-- Offset pagination at depth:
EXPLAIN ANALYZE
SELECT * FROM orders ORDER BY created_at DESC LIMIT 20 OFFSET 1000000;

-- Keyset pagination at the same position:
EXPLAIN ANALYZE
SELECT * FROM orders
WHERE (created_at, id) < ('2025-01-15 10:30:00', 12345)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

**Expected output (illustrative):**

```text
-- Offset: Seq Scan / Index Scan with 1,000,020 rows read, ~87 ms.
-- Keyset: Index Scan with 20 rows read, ~1 ms.
```

**Why this output appears**: The offset query must read every row up to the offset (one million rows) before returning its twenty. The keyset query uses the composite index to seek directly to the starting position and reads only the twenty requested rows. The `EXPLAIN ANALYZE` output shows the dramatic difference in rows read and execution time.

#### Real-World Cases

- **High‑traffic APIs**: A public API that paginates millions of records uses cursor pagination to keep response times low and consistent.
- **Reporting dashboards**: An internal dashboard that only needs the first few pages can use offset pagination without noticeable performance problems.
- **Hybrid approach**: Some systems use offset pagination for the first few pages (where it is fast) and switch to keyset pagination for deeper pages, combining the simplicity of offset with the performance of keyset.

---

## References

- PostgreSQL Documentation – SELECT (LIMIT, OFFSET, FETCH) – https://manpages.debian.org/unstable/postgresql-client-16/SELECT.7.en.html
- PostgreSQL Documentation – SELECT (PostgreSQL 9.6, LIMIT clause) – https://manpages.debian.org/unstable/postgresql-client-9.6/SELECT.7.en.html
- MySQL 8.4 Reference Manual – SELECT Statement (LIMIT, OFFSET) – https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/select.html
- SQL Server – TOP and OFFSET/FETCH (AWS DMS Playbook) – https://docs.aws.eu/dms/latest/sql-server-to-aurora-postgresql-migration-playbook/chap-sql-server-aurora-pg.tsql.topfetch.html
- Oracle Database – Row Limiting Clause (OFFSET ... FETCH) – https://docs.oracle.com/en/database/oracle/oracle-database/26/dwhsg/database-data-warehousing-guide.pdf
- SQL:2008 OFFSET ... FETCH FIRST/NEXT (PostgreSQL SELECT) – https://manpages.ubuntu.com/manpages/…/SELECT.7.html
- StackSync – Postgres Pagination: Keyset vs Offset vs Cursor – https://www.stacksync.com/blog/keyset-cursors-postgres-pagination-fast-accurate-scalable
- OneUptime – How to Optimize LIMIT with Large Offsets in MySQL – https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-optimize-limit-large-offsets/README.md
- Supabase Best Practices – Cursor-Based Pagination Instead of OFFSET – https://github.com/fcakyon/claude-codex-settings/blob/cd53c6a75fe64d291afd3d7139a6b71ebf9d86f3/plugins/supabase-skills/skills/supabase-postgres-best-practices/references/data-pagination.md
- Erik Darling – Learn T-SQL: TOP and OFFSET/FETCH – https://erikdarling.com/learn-t-sql-with-erik-top-and-offset-fetch/
- Encina – Cursor Pagination Benchmarks – https://github.com/dlrivada/Encina/blob/main/docs/features/cursor-pagination.md
- PostgreSQL Git – SQL:2008 OFFSET/FETCH syntax – https://www.postgresql.org/message-id/…