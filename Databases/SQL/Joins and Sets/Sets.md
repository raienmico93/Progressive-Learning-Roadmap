# SQL Combining Result Sets: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL set operators (UNION, UNION ALL, INTERSECT, EXCEPT/MINUS) combine the results of two or more SELECT statements into a single result set, treating each query's output as a mathematical set of rows.

**Technical Definition:** In the SQL standard (ISO/IEC 9075-2), a `<query expression>` can combine multiple `<query primary>` elements using the set operators `UNION`, `INTERSECT`, and `EXCEPT`. Each operator takes two query results and produces a combined result based on set-theoretic semantics: UNION for union, INTERSECT for intersection, and EXCEPT for set difference. The operators require the participating queries to be "union compatible" — that is, they must return the same number of columns with compatible data types. By default, all set operators eliminate duplicate rows; the `ALL` modifier preserves them.

**Beginner-Friendly Explanation:** Think of two separate queries as two lists of rows. A set operator is like a combination rule: UNION puts both lists together (removing duplicates), UNION ALL puts both lists together (keeping duplicates), INTERSECT finds rows that appear in both lists, and EXCEPT finds rows that appear in the first list but not the second. It's like comparing two shopping lists: what's on both, what's only on one, or what's on both combined.

### Key Characteristics

- **Set-theoretic semantics:** Operations follow mathematical set theory (union, intersection, difference).
- **Duplicate handling by default:** All operators except `UNION ALL` eliminate duplicate rows unless the `ALL` modifier is used.
- **Union compatibility requirement:** Queries must return the same number of columns with compatible data types.
- **Column names from first query:** The result set's column names are taken from the first SELECT statement.
- **Precedence rules:** `INTERSECT` binds more tightly than `UNION` and `EXCEPT`, which have equal precedence and associate left-to-right.
- **Dialect variations:** Oracle uses `MINUS` as a synonym for `EXCEPT`; MySQL added `INTERSECT` and `EXCEPT` only in version 8.0.31; SQLite does not support `ALL` modifiers for `INTERSECT` or `EXCEPT`.

### Prerequisites

- **Basic SQL knowledge:** Familiarity with `SELECT`, `FROM`, and `WHERE`.
- **Set theory concepts:** Understanding of union, intersection, and difference.
- **Data type awareness:** Knowledge of compatible data types (numeric, character, date/time).
- **NULL awareness:** Recognition of how `NULL` values interact with duplicate elimination.

### Related Programming Areas

- **Data Analysis and Business Intelligence:** Set operators combine results from different queries for comparative analysis.
- **Data Engineering (ETL):** Set operators are used to merge data from multiple sources and identify differences.
- **Application Development:** Backend services use set operations for tasks like combining search results from different sources.
- **Database Administration:** DBAs use set operators to compare schemas and find data discrepancies.

### Core Concepts / Features

1. `UNION`
2. `UNION ALL`
3. `INTERSECT`
4. `EXCEPT` / `MINUS`
5. Column Compatibility
6. Data-Type Compatibility
7. Duplicate Handling

---

## 1. UNION

### Definitions

**Core Definition:** `UNION` combines the results of two queries into a single result set, eliminating duplicate rows.

**Technical Definition:** `query1 UNION query2` effectively appends the result of `query2` to the result of `query1` and then removes duplicate rows, behaving like a `DISTINCT` operation on the combined result. The result set contains all rows that appear in either or both input queries, with each distinct row appearing exactly once. Rows are compared for duplicates based on all columns in the result set.

**Beginner-Friendly Explanation:** `UNION` combines two query results and removes any duplicate rows. If a row appears in both queries, it appears only once in the final result.

### Purposes

- To combine results from multiple queries into a single result set.
- To eliminate duplicate rows from the combined result.
- To merge data from similar tables or different conditions.
- To create a unified view of data from multiple sources.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
query1 UNION [ALL | DISTINCT] query2
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `query1`, `query2` | Two SELECT statements producing compatible result sets |
| `UNION` | Combines results and eliminates duplicates |
| `ALL` | Preserves duplicates |
| `DISTINCT` | Explicitly specifies duplicate elimination (default) |

**Syntax Rules:**

- Both queries must return the same number of columns.
- Corresponding columns must have compatible data types.
- The `ORDER BY` clause can appear only at the end of the entire compound query and applies to the final result.
- `LIMIT` can appear only on the final query or in the entire compound query.
- Column names in the result are taken from the first query.

**Constraints and Limitations:**

- `UNION` performs a duplicate-elimination step, which requires sorting or hashing and can be slower than `UNION ALL`.
- `UNION` cannot be used with `FOR UPDATE`, `FOR SHARE`, or similar locking clauses on the inputs.
- In Oracle, set operators are not valid on `LONG` columns.

### Annotated Complete Code Examples

**Example 1: UNION of Two Tables**

```sql
-- Setup: Create two tables with overlapping data
CREATE TABLE employees (
    name VARCHAR(50),
    department VARCHAR(50)
);

CREATE TABLE contractors (
    name VARCHAR(50),
    department VARCHAR(50)
);

INSERT INTO employees VALUES
    ('Alice', 'Engineering'), ('Bob', 'Marketing'), ('Carol', 'Engineering');
INSERT INTO contractors VALUES
    ('Bob', 'Marketing'), ('David', 'Sales'), ('Eve', 'Engineering');

-- Query: Combine all people, removing duplicates
SELECT name, department FROM employees
UNION
SELECT name, department FROM contractors
ORDER BY name;

-- Expected Output:
--  name  | department
-- -------+-------------
--  Alice | Engineering
--  Bob   | Marketing
--  Carol | Engineering
--  David | Sales
--  Eve   | Engineering
```

**Why this output occurs:** The `UNION` combines the five rows from `employees` and three rows from `contractors`, then removes duplicate rows. Bob appears in both tables with the same department, so he appears only once. The result contains five distinct rows.

### Real-World Cases

**Case 1: Unified Contact List**

A CRM system uses `UNION` to combine contacts from different source tables (e.g., `leads` and `customers`) into a single deduplicated list for marketing campaigns.

**Case 2: Cross-Database Reporting**

A reporting tool uses `UNION` to combine sales data from multiple regional databases into a single result set for global analysis.

### References

- PostgreSQL Documentation — Combining Queries (UNION) - https://www.postgresql.org/docs/current/queries-union.html
- MySQL Reference Manual — Set Operations with UNION - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/set-operations.html
- Microsoft Learn — Set Operators — UNION - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/set-operators-union-transact-sql
- Oracle Database SQL Language Reference — The UNION [ALL], INTERSECT, MINUS Operators - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/The-UNION-ALL-INTERSECT-MINUS-Operators.html
- SQLite Documentation — SELECT (compound_op) - http://svn.code.sf.net/p/dolserver/code/DOLSharp/branch/dol%20database%20v2/SQLiteDBProvider/source/Doc/Extra/lang_select.html

---

## 2. UNION ALL

### Definitions

**Core Definition:** `UNION ALL` combines the results of two queries into a single result set, preserving all rows including duplicates.

**Technical Definition:** `query1 UNION ALL query2` appends the result of `query2` to the result of `query1` without any duplicate-elimination step. Every row from both queries appears in the result set, even if identical rows exist within or across the queries. This makes `UNION ALL` significantly faster than `UNION` because it avoids the sorting or hashing required for deduplication.

**Beginner-Friendly Explanation:** `UNION ALL` combines two query results and keeps everything, including duplicates. If a row appears in both queries, it appears twice in the final result.

### Purposes

- To combine results from multiple queries while preserving all rows.
- To improve performance by skipping duplicate elimination.
- To concatenate data where duplicates are meaningful or acceptable.
- To append data from one table to another without deduplication.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
query1 UNION ALL query2
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `query1`, `query2` | Two SELECT statements producing compatible result sets |
| `UNION ALL` | Combines results without removing duplicates |

**Syntax Rules:**

- Same union-compatibility requirements as `UNION` (same number of columns, compatible types).
- The `ALL` keyword immediately follows `UNION`.
- No duplicate elimination occurs; all rows from both queries are returned.
- Performance is generally better than `UNION` because no deduplication is performed.

**Constraints and Limitations:**

- The result set may be larger than `UNION` because duplicates are retained.
- If duplicates are not desired, a subsequent `DISTINCT` or `GROUP BY` must be applied.

### Annotated Complete Code Examples

**Example 1: UNION ALL of Two Tables**

```sql
-- Using the same employees and contractors tables
SELECT name, department FROM employees
UNION ALL
SELECT name, department FROM contractors
ORDER BY name;

-- Expected Output:
--  name  | department
-- -------+-------------
--  Alice | Engineering
--  Bob   | Marketing
--  Bob   | Marketing
--  Carol | Engineering
--  David | Sales
--  Eve   | Engineering
```

**Why this output occurs:** `UNION ALL` combines the five employee rows and three contractor rows without removing duplicates. Bob appears twice because he exists in both tables. The result contains eight rows total.

### Real-World Cases

**Case 1: Log Aggregation**

A logging system uses `UNION ALL` to combine application logs from multiple servers, preserving all entries including any duplicates that may indicate repeated events.

**Case 2: ETL Data Loading**

An ETL pipeline uses `UNION ALL` to append incremental data to a staging table, accepting that some rows may be duplicates to be resolved later in the pipeline.

### References

- PostgreSQL Documentation — Combining Queries (UNION ALL) - https://www.postgresql.org/docs/current/queries-union.html
- MySQL Reference Manual — UNION ALL - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/set-operations.html

---

## 3. INTERSECT

### Definitions

**Core Definition:** `INTERSECT` returns only the rows that appear in both query results, eliminating duplicates.

**Technical Definition:** `query1 INTERSECT query2` returns all rows that are both in the result of `query1` and in the result of `query2`. Duplicate rows are eliminated unless `INTERSECT ALL` is used. With `INTERSECT ALL`, a row that appears `m` times in the left table and `n` times in the right table appears `min(m, n)` times in the result set.

**Beginner-Friendly Explanation:** `INTERSECT` finds the common rows between two query results. It shows only rows that appear in both.

### Purposes

- To find rows common to two or more queries.
- To identify overlapping data between tables.
- To filter results based on presence in another result set.
- To compute set intersections in data analysis.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
query1 INTERSECT [ALL | DISTINCT] query2
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `query1`, `query2` | Two SELECT statements producing compatible result sets |
| `INTERSECT` | Returns rows present in both queries |
| `ALL` | Preserves duplicates using min(m, n) semantics |
| `DISTINCT` | Explicitly specifies duplicate elimination (default) |

**Syntax Rules:**

- Same union-compatibility requirements as `UNION`.
- `INTERSECT` binds more tightly than `UNION` and `EXCEPT`. For example, `A UNION B INTERSECT C` is parsed as `A UNION (B INTERSECT C)`.
- `INTERSECT` is left-associative when multiple `INTERSECT` operators appear.
- `NULL` values are treated as equal for the purpose of intersection (i.e., `NULL` intersects with `NULL`).

**Constraints and Limitations:**

- MySQL supports `INTERSECT` only from version 8.0.31 onward.
- SQLite does not support `INTERSECT ALL`.
- Oracle supports `INTERSECT ALL`.

### Annotated Complete Code Examples

**Example 1: INTERSECT of Two Tables**

```sql
-- Using the same employees and contractors tables
SELECT name, department FROM employees
INTERSECT
SELECT name, department FROM contractors
ORDER BY name;

-- Expected Output:
--  name | department
-- ------+-------------
--  Bob  | Marketing
```

**Why this output occurs:** The `INTERSECT` returns only rows that appear in both the `employees` and `contractors` tables. Bob is the only person who appears in both tables with the same department. The result contains one row.

### Real-World Cases

**Case 1: Common Customers**

A marketing team uses `INTERSECT` to find customers who purchased from both the online store and the physical store: `SELECT customer_id FROM online_orders INTERSECT SELECT customer_id FROM store_orders`.

**Case 2: Overlapping Skills**

An HR system uses `INTERSECT` to find employees who possess both of two required skills: `SELECT employee_id FROM skills WHERE skill = 'Python' INTERSECT SELECT employee_id FROM skills WHERE skill = 'SQL'`.

### References

- PostgreSQL Documentation — INTERSECT - https://www.postgresql.org/docs/current/queries-union.html
- MySQL Reference Manual — INTERSECT - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/set-operations.html
- Oracle Database SQL Language Reference — INTERSECT - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/The-UNION-ALL-INTERSECT-MINUS-Operators.html
- SQLite Documentation — INTERSECT - http://svn.code.sf.net/p/dolserver/code/DOLSharp/branch/dol%20database%20v2/SQLiteDBProvider/source/Doc/Extra/lang_select.html

---

## 4. EXCEPT / MINUS

### Definitions

**Core Definition:** `EXCEPT` (called `MINUS` in Oracle) returns all rows from the first query that do not appear in the second query, eliminating duplicates.

**Technical Definition:** `query1 EXCEPT query2` returns all rows that are in the result of `query1` but not in the result of `query2`. This is the set difference operation. Duplicates are eliminated unless `EXCEPT ALL` is used. With `EXCEPT ALL`, a row that has `m` duplicates in the left table and `n` duplicates in the right table appears `max(m - n, 0)` times in the result set.

**Beginner-Friendly Explanation:** `EXCEPT` (or `MINUS`) finds rows that are in the first query but not in the second. It's like subtracting one list from another.

### Purposes

- To find rows in one query that are not present in another.
- To identify data discrepancies between tables.
- To perform set difference operations.
- To filter out rows that appear in a second result set.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
query1 EXCEPT [ALL | DISTINCT] query2
query1 MINUS [ALL | DISTINCT] query2   -- Oracle synonym
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `query1`, `query2` | Two SELECT statements producing compatible result sets |
| `EXCEPT` / `MINUS` | Returns rows in query1 not in query2 |
| `ALL` | Preserves duplicates using max(m - n, 0) semantics |
| `DISTINCT` | Explicitly specifies duplicate elimination (default) |

**Syntax Rules:**

- Same union-compatibility requirements as `UNION`.
- `EXCEPT` binds at the same level as `UNION` (left-to-right associativity).
- Oracle uses `MINUS` as a synonym for `EXCEPT` with identical semantics.
- SQLite does not support `EXCEPT ALL`.
- `NULL` values are treated as equal for difference purposes.

**Constraints and Limitations:**

- MySQL supports `EXCEPT` only from version 8.0.31 onward.
- The order of queries matters: `A EXCEPT B` is not the same as `B EXCEPT A`.

### Annotated Complete Code Examples

**Example 1: EXCEPT of Two Tables**

```sql
-- Using the same employees and contractors tables
SELECT name, department FROM employees
EXCEPT
SELECT name, department FROM contractors
ORDER BY name;

-- Expected Output:
--  name  | department
-- -------+-------------
--  Alice | Engineering
--  Carol | Engineering
```

**Why this output occurs:** The `EXCEPT` returns rows from `employees` that do not appear in `contractors`. Bob is excluded because he appears in both tables. Alice and Carol appear only in `employees`, so they are returned. The result contains two rows.

### Real-World Cases

**Case 1: Discontinued Products**

An inventory system uses `EXCEPT` to find products that were in last year's catalog but not in this year's: `SELECT product_id FROM last_year_catalog EXCEPT SELECT product_id FROM this_year_catalog`.

**Case 2: Missing Data Audit**

A data engineer uses `EXCEPT` to find records in a source table that are missing from a target table after an ETL load: `SELECT id FROM source EXCEPT SELECT id FROM target`.

### References

- PostgreSQL Documentation — EXCEPT - https://www.postgresql.org/docs/current/queries-union.html
- MySQL Reference Manual — EXCEPT - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/set-operations.html
- Oracle Database SQL Language Reference — MINUS - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/The-UNION-ALL-INTERSECT-MINUS-Operators.html
- SQLite Documentation — EXCEPT - http://svn.code.sf.net/p/dolserver/code/DOLSharp/branch/dol%20database%20v2/SQLiteDBProvider/source/Doc/Extra/lang_select.html

---

## 5. Column Compatibility

### Definitions

**Core Definition:** Column compatibility (also called union compatibility) is the requirement that all queries combined by a set operator return the same number of columns, with corresponding columns having compatible data types.

**Technical Definition:** For a set operation to be valid, the participating queries must be "union compatible," meaning they return the same number of columns and the corresponding columns in each query must have data types that are either identical or implicitly convertible to a common type. Column names need not match; the result set's column names are taken from the first query. The order of columns is significant: the first column of query1 is compared with the first column of query2, the second with the second, and so on.

**Beginner-Friendly Explanation:** When you combine two query results, both must have the same "shape" — the same number of columns, and each column must hold similar types of data. For example, you can't combine a query that returns two columns with one that returns three.

### Purposes

- To ensure set operations are mathematically valid.
- To prevent type mismatches and runtime errors.
- To define the structure of the combined result set.
- To enable predictable column naming in the output.

### Syntax Rules and Structure

**Rules for Column Compatibility:**

| Rule | Requirement |
|------|-------------|
| Number of columns | All queries must return the same number of columns |
| Column order | Corresponding expressions must be in the same order |
| Column names | Not required to match; result uses names from the first query |
| Data types | Corresponding columns must have compatible data types |

**Syntax Rules:**

- If the column counts differ, the database raises an error.
- If the data types are incompatible and no implicit conversion exists, an error is raised.
- Column aliases from the first query define the output column names.
- `ORDER BY` in the compound query must reference output column names or ordinal positions.

**Constraints and Limitations:**

- Oracle does not perform implicit conversion across data type groups (e.g., character to numeric) without an explicit conversion.
- In SQL Server, `text`, `ntext`, and `image` data types cannot be used with set operators.
- Column compatibility is checked at query compile time, not at runtime.

### Annotated Complete Code Examples

**Example 1: Valid Column Compatibility**

```sql
-- Valid: Both queries return two columns with compatible types
SELECT first_name, salary FROM employees
UNION
SELECT name, amount FROM contractors;

-- Expected Output:
--  first_name | salary
-- ------------+--------
--  Alice      | 95000
--  Bob        | 72000
--  Carol      | 105000
--  David      | 68000
--  Eve        | 88000
```

**Why this output occurs:** Both queries return two columns. The first column of each is a character type (`first_name` and `name`); the second is numeric (`salary` and `amount`). The column names in the result come from the first query (`first_name`, `salary`).

**Example 2: Invalid Column Compatibility (Error)**

```sql
-- Invalid: Different number of columns
SELECT first_name, last_name, salary FROM employees
UNION
SELECT name, amount FROM contractors;

-- Expected Error (PostgreSQL):
-- ERROR: each UNION query must have the same number of columns
```

**Why this error occurs:** The first query returns three columns, but the second returns two. Set operations require the same number of columns in all participating queries.

### Real-World Cases

**Case 1: Cross-System Data Merging**

An integration platform combines user data from two systems. It ensures both queries select the same columns in the same order (e.g., `user_id`, `email`, `created_at`) so the `UNION` produces a consistent result set.

**Case 2: Schema Evolution**

When a table schema changes (e.g., a column is added), queries using set operations must be updated to maintain column compatibility across all branches.

### References

- SAP Help — Rules for Set Operations (Same number of items) - https://help.sap.com/docs/SAP_SQL_Anywhere/e38b2f6217f24bdb90a3ff8ae57b1dd5/8191e8826ce210149b9df45b17ca78f3.html
- PostgreSQL Documentation — Union Compatibility - https://www.postgresql.org/docs/current/queries-union.html
- Oracle Database SQL Language Reference — Set Operator Restrictions - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/The-UNION-ALL-INTERSECT-MINUS-Operators.html

---

## 6. Data-Type Compatibility

### Definitions

**Core Definition:** Data-type compatibility is the requirement that corresponding columns in set-operation queries have data types that are either identical or can be implicitly converted to a common type.

**Technical Definition:** For each pair of corresponding columns in the participating queries, the database must be able to determine a common data type for the result. If the types are identical, that type is used. If they differ but an implicit conversion exists (e.g., `INT` to `BIGINT`, `CHAR` to `VARCHAR`), the database applies the conversion and uses the wider or more general type. If no implicit conversion exists (e.g., `CHAR` to `INT`), the query fails with a type mismatch error.

**Beginner-Friendly Explanation:** When combining columns from different queries, the data types must be "compatible" — meaning they can be treated as the same kind of data. Numbers can combine with numbers, text with text, dates with dates. You can't combine a number column with a text column unless you explicitly convert one of them.

### Purposes

- To ensure that values from different sources can be meaningfully combined.
- To determine the result set's column data types.
- To prevent type errors and data corruption.
- To enable implicit type promotion for numeric and string types.

### Syntax Rules and Structure

**Data-Type Compatibility Rules:**

| Type Category | Compatible With | Notes |
|--------------|----------------|-------|
| Numeric | Numeric | Implicit promotion to wider type |
| Character | Character | `CHAR` → `VARCHAR` promotion |
| Date/Time | Date/Time | Compatible within category |
| Numeric vs. Character | **Incompatible** | Requires explicit cast |
| `NULL` | Any | `NULL` is compatible with all types |

**Syntax Rules:**

- If both columns are the same type, that type is used.
- If one is `CHAR` and the other `VARCHAR`, the result is `VARCHAR`.
- If numeric types differ, the result follows numeric precedence (e.g., `BINARY_DOUBLE` > `BINARY_FLOAT` > `NUMBER` in Oracle).
- Oracle does not perform implicit conversion across data type groups.
- SQL Server does not allow `text`, `ntext`, or `image` with set operators.

**Constraints and Limitations:**

- Implicit conversion rules vary by RDBMS; what works in PostgreSQL may not work in Oracle.
- Explicit `CAST` or `CONVERT` may be required for cross-type operations.
- `NULL` values are compatible with any data type but can affect type inference.

### Annotated Complete Code Examples

**Example 1: Compatible Numeric Types**

```sql
-- Setup
CREATE TABLE table_a (amount INTEGER);
CREATE TABLE table_b (amount BIGINT);
INSERT INTO table_a VALUES (100), (200);
INSERT INTO table_b VALUES (300), (400);

-- Query: INTEGER and BIGINT are compatible
SELECT amount FROM table_a
UNION ALL
SELECT amount FROM table_b
ORDER BY amount;

-- Expected Output:
--  amount
-- --------
--     100
--     200
--     300
--     400
```

**Why this output occurs:** `INTEGER` and `BIGINT` are both numeric types with an implicit conversion from `INTEGER` to `BIGINT`. The result column has type `BIGINT` (the wider type), and all values are returned without error.

**Example 2: Incompatible Data Types (Error)**

```sql
-- Setup
CREATE TABLE table_c (id INTEGER);
CREATE TABLE table_d (id VARCHAR(10));
INSERT INTO table_c VALUES (1), (2);
INSERT INTO table_d VALUES ('a'), ('b');

-- Query: INTEGER and VARCHAR are incompatible in Oracle
SELECT id FROM table_c
UNION
SELECT id FROM table_d;

-- Expected Error (Oracle):
-- ORA-01790: expression must have same datatype as corresponding expression
```

**Why this error occurs:** Oracle does not implicitly convert between numeric and character data type groups. The `INTEGER` and `VARCHAR` types are incompatible without an explicit conversion.

### Real-World Cases

**Case 1: Combining Integer and Decimal Columns**

A financial system combines `amount` columns from two tables where one stores integers and the other stores decimals. The implicit conversion to `DECIMAL` allows the `UNION` to succeed.

**Case 2: Cross-Database Type Mismatch**

An ETL process combines data from PostgreSQL (which may use `TEXT`) and Oracle (which uses `VARCHAR2`). The developer must ensure data type compatibility or apply explicit casts.

### References

- SAP Help — Rules for Set Operations (Data types must match) - https://help.sap.com/docs/SAP_SQL_Anywhere/e38b2f6217f24bdb90a3ff8ae57b1dd5/8191e8826ce210149b9df45b17ca78f3.html
- Oracle Database SQL Language Reference — Data Type Determination in Set Operations - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/The-UNION-ALL-INTERSECT-MINUS-Operators.html
- IBM Netezza — Set Operation Data Type Compatibility - https://www.ibm.com/docs/en/netezza

---

## 7. Duplicate Handling

### Definitions

**Core Definition:** Duplicate handling refers to how set operators treat duplicate rows — either eliminating them (the default for `UNION`, `INTERSECT`, and `EXCEPT`) or preserving them (when the `ALL` modifier is used).

**Technical Definition:** The SQL standard defines two modes for set operators: `DISTINCT` (the default) and `ALL`. In `DISTINCT` mode, duplicate rows are eliminated from the result, and each distinct row appears once. In `ALL` mode, duplicates are preserved according to specific multiplicity rules: for `UNION ALL`, all rows from both queries are included; for `INTERSECT ALL`, a row with `m` duplicates in the left and `n` duplicates in the right appears `min(m, n)` times; for `EXCEPT ALL`, a row with `m` duplicates in the left and `n` duplicates in the right appears `max(m - n, 0)` times.

**Beginner-Friendly Explanation:** By default, set operators remove duplicate rows. If you want to keep duplicates, you add the word `ALL` after the operator. The rules for how many duplicates appear with `ALL` depend on the operator: `UNION ALL` keeps everything, `INTERSECT ALL` keeps the smaller count, and `EXCEPT ALL` subtracts counts.

### Purposes

- To control whether duplicate rows appear in the combined result.
- To optimise performance by skipping deduplication when duplicates are acceptable.
- To implement multiset (bag) semantics for data analysis.
- To produce accurate results when duplicates carry meaning.

### Syntax Rules and Structure

**Duplicate Handling Modes:**

| Operator | Default (DISTINCT) | With ALL |
|----------|-------------------|----------|
| `UNION` | Removes duplicates | Preserves all rows |
| `INTERSECT` | Removes duplicates | Appears `min(m, n)` times |
| `EXCEPT` | Removes duplicates | Appears `max(m - n, 0)` times |

**Syntax Rules:**

- `DISTINCT` can be written explicitly but is the default.
- `ALL` immediately follows the set operator keyword.
- Duplicate elimination treats `NULL` values as equal to each other for comparison purposes.
- `UNION ALL` is generally faster than `UNION` because it skips the deduplication step.

**Constraints and Limitations:**

- MySQL supports `ALL` modifiers for all three operators from version 8.0.31.
- SQLite does not support `INTERSECT ALL` or `EXCEPT ALL`.
- The `ALL` modifier cannot be combined with `DISTINCT` in the same operation.

### Annotated Complete Code Examples

**Example 1: Duplicate Counts with INTERSECT ALL and EXCEPT ALL**

```sql
-- Setup: Tables with duplicate values
CREATE TABLE left_table  (val INTEGER);
CREATE TABLE right_table (val INTEGER);

INSERT INTO left_table  VALUES (1), (1), (1), (2), (3);
INSERT INTO right_table VALUES (1), (1), (2), (4);

-- INTERSECT ALL: min(m, n) duplicates
SELECT val FROM left_table
INTERSECT ALL
SELECT val FROM right_table
ORDER BY val;

-- Expected Output:
--  val
-- -----
--    1
--    1
--    2

-- EXCEPT ALL: max(m - n, 0) duplicates
SELECT val FROM left_table
EXCEPT ALL
SELECT val FROM right_table
ORDER BY val;

-- Expected Output:
--  val
-- -----
--    1
--    3
```

**Why this output occurs:** For `INTERSECT ALL`, value `1` appears 3 times in the left table and 2 times in the right, so it appears `min(3, 2) = 2` times. Value `2` appears 1 time in each, so it appears once. For `EXCEPT ALL`, value `1` appears 3 times in the left and 2 times in the right, so it appears `max(3 - 2, 0) = 1` time. Value `3` appears 1 time in the left and 0 times in the right, so it appears once. Value `2` appears 1 time in the left and 1 time in the right, so it appears 0 times.

### Real-World Cases

**Case 1: Inventory Reconciliation**

A warehouse system uses `EXCEPT ALL` to find items that appear more times in the physical count than in the system count, revealing discrepancies.

**Case 2: Log Analysis**

A security analyst uses `UNION ALL` to combine authentication logs from multiple servers, preserving all entries to detect patterns of repeated failed login attempts.

### References

- PostgreSQL Documentation — Duplicate Handling in Set Operations - https://www.postgresql.org/docs/current/queries-union.html
- Ubuntu Manpage — SELECT (INTERSECT ALL, EXCEPT ALL) - https://manpages.ubuntu.com/manpages/focal/man7/SELECT.7.html
- Oracle Database SQL Language Reference — ALL Modifiers - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/The-UNION-ALL-INTERSECT-MINUS-Operators.html

---

## Summary Table of Set Operators

| Operator | Returns | Duplicate Handling (Default) | ALL Modifier Behaviour | Oracle Synonym |
|----------|---------|------------------------------|------------------------|----------------|
| `UNION` | All rows from both queries | Eliminates duplicates | Preserves all rows | — |
| `UNION ALL` | All rows from both queries | N/A | Preserves all rows | — |
| `INTERSECT` | Rows in both queries | Eliminates duplicates | `min(m, n)` copies | — |
| `INTERSECT ALL` | Rows in both queries | N/A | `min(m, n)` copies | — |
| `EXCEPT` | Rows in first but not second | Eliminates duplicates | `max(m - n, 0)` copies | `MINUS` |
| `EXCEPT ALL` | Rows in first but not second | N/A | `max(m - n, 0)` copies | `MINUS ALL` |

---

## Dialect Support for Set Operators

| Feature | PostgreSQL | MySQL | SQL Server | Oracle | SQLite |
|---------|-----------|-------|------------|--------|--------|
| `UNION` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `UNION ALL` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `INTERSECT` | ✅ | ✅ (8.0.31+) | ✅ | ✅ | ✅ |
| `INTERSECT ALL` | ✅ | ✅ (8.0.31+) | ❌ | ✅ | ❌ |
| `EXCEPT` | ✅ | ✅ (8.0.31+) | ✅ | ✅ (synonym) | ✅ |
| `EXCEPT ALL` | ✅ | ✅ (8.0.31+) | ❌ | ✅ | ❌ |
| `MINUS` | ❌ | ❌ | ❌ | ✅ | ❌ |
| `MINUS ALL` | ❌ | ❌ | ❌ | ✅ | ❌ |

---

## References

- PostgreSQL Documentation — Combining Queries (UNION, INTERSECT, EXCEPT) - https://www.postgresql.org/docs/current/queries-union.html
- MySQL Reference Manual — Set Operations with UNION, INTERSECT, and EXCEPT - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/set-operations.html
- Microsoft Learn — Set Operators — UNION - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/set-operators-union-transact-sql
- Microsoft Learn — Set Operators — EXCEPT and INTERSECT - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/set-operators-except-and-intersect-transact-sql
- Oracle Database SQL Language Reference — The UNION [ALL], INTERSECT, MINUS Operators - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/The-UNION-ALL-INTERSECT-MINUS-Operators.html
- SQLite Documentation — SELECT (compound_op) - http://svn.code.sf.net/p/dolserver/code/DOLSharp/branch/dol%20database%20v2/SQLiteDBProvider/source/Doc/Extra/lang_select.html
- SAP Help — Rules for Set Operations - https://help.sap.com/docs/SAP_SQL_Anywhere/e38b2f6217f24bdb90a3ff8ae57b1dd5/8191e8826ce210149b9df45b17ca78f3.html
- IBM Netezza — Set Operations - https://www.ibm.com/docs/en/netezza
- Ubuntu Manpage — SELECT (INTERSECT, EXCEPT) - https://manpages.ubuntu.com/manpages/focal/man7/SELECT.7.html
- ISO/IEC 9075-2 — SQL Standard (Query Expression, Set Operators) - https://jtc1sc32.org/doc/N0251-0300/32N0254.pdf