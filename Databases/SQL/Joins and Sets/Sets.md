# Comprehensive Programming Cheat Sheet: Combining Result Sets

## Topic Overview

### Definitions

**Core Definition**  
Combining result sets is the use of SQL set operators—`UNION`, `UNION ALL`, `INTERSECT`, `INTERSECT ALL`, `EXCEPT`, `EXCEPT ALL`, and `MINUS`—to vertically stack, intersect, or subtract the row sets produced by two or more `SELECT` statements into a single result.

**Technical Definition**  
Set operators are binary or n-ary query operators defined by the SQL standard (SQL:1992 and later refinements) that combine the results of two query expressions. The operands must be *union-compatible*: they must return the same number of columns, and corresponding columns must have compatible data types. `UNION` returns rows appearing in either operand; `INTERSECT` returns rows appearing in both; `EXCEPT` (called `MINUS` in Oracle) returns rows appearing in the first operand but not the second. The `ALL` variants preserve duplicate multiplicity; the default (implicit `DISTINCT`) eliminates duplicates. Precedence varies by engine: in PostgreSQL and MySQL, `INTERSECT` binds more tightly than `UNION` and `EXCEPT`; in Oracle, all set operators have equal precedence and are evaluated left to right unless parenthesized.

**Beginner-Friendly Explanation**  
Set operators let you stack or compare lists of rows. `UNION` is like combining two guest lists into one (with duplicates removed unless you say `ALL`); `INTERSECT` gives you only the people on both lists; `EXCEPT`/`MINUS` gives you the people on the first list but not the second. All the `SELECT` statements involved must return the same number of columns with matching types, in the same order.

### Key Characteristics

- **Vertical combination**: Set operators combine rows from separate queries; they do not join columns horizontally (that is what `JOIN` does).
- **Union-compatible requirement**: Every operand must project the same number of columns, in the same ordinal positions, with compatible data types.
- **Column names come from the first query**: The final result’s column names are taken from the first `SELECT`; subsequent `SELECT` aliases are irrelevant to the output header.
- **Duplicate handling is explicit**: Default behavior eliminates duplicates; `ALL` preserves them.
- **`ORDER BY` applies to the final result**: A single `ORDER BY` may appear at the end, after all set operations, to sort the combined output.
- **Precedence is engine-specific**: `INTERSECT` generally binds tighter than `UNION`/`EXCEPT` in PostgreSQL and MySQL; Oracle evaluates all set operators left to right at equal precedence.
- **`MINUS` is Oracle’s name for `EXCEPT`**: They are synonyms; MySQL does not support `MINUS` and uses `EXCEPT` (8.0.31+).

### Prerequisites

- Solid understanding of `SELECT`, `WHERE`, `ORDER BY`, and column aliasing.
- Familiarity with data types and implicit type conversion rules.
- Awareness of duplicate semantics (`DISTINCT` vs `ALL`).
- Knowledge of the target engine’s precedence rules for set operators.

### Related Programming Areas

- Data integration and ETL (combining sources).
- Reconciliation and auditing (difference detection).
- Reporting and dashboards (stacking fact tables).
- Query optimisation (join vs. set-operator rewrites).
- Data quality (finding common or missing records).

---

## Core Concepts and Features

### 1. `UNION`

#### Definitions

**Core Definition**  
`UNION` combines the rows of two query results into a single result set, eliminating duplicate rows.

**Technical Definition**  
`query1 UNION query2` returns the set union of the rows from `query1` and `query2`. The result contains every row that appears in at least one operand; duplicate rows are removed (equivalent to `UNION DISTINCT`). The operands must be union-compatible. `UNION` is commutative: `A UNION B` and `B UNION A` produce the same set of rows, though row order is unspecified unless an outer `ORDER BY` is added.

**Beginner-Friendly Explanation**  
`UNION` stacks two lists and removes duplicates, giving you a clean combined list.

#### Purposes

- To combine rows from multiple tables or queries into one deduplicated list.
- To stack results from different time periods or categories without repeating rows.
- To produce a distinct set of values across heterogeneous sources.
- To merge lookup or reference data from multiple systems.

#### Syntax Rules and Structure

**General syntax**:

```sql
query1
UNION [DISTINCT]
query2
[ORDER BY ...];
```

**PostgreSQL** (official) :

```sql
query1 UNION [ALL] query2
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `query1` | First `SELECT` (or parenthesized query). |
| `UNION` | Set operator keyword. |
| `query2` | Second `SELECT`. |
| `ORDER BY` | Optional; sorts the final combined result. |

**Syntax rules**

- Operands must have the same number of columns.
- Corresponding columns must be of compatible types.
- Column names in the result come from `query1` only.
- Duplicates are removed unless `ALL` is specified.
- `ORDER BY` can appear only once, at the end, and may reference output column names or ordinal positions.

**Constraints and limitations**

- Duplicate elimination adds a sort or hash step, which can be expensive at scale.
- `UNION` cannot be used to combine columns; it only stacks rows.
- Parentheses are required around individual queries that contain `ORDER BY`, `LIMIT`, or `FOR UPDATE` in PostgreSQL.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Combining two customer lists

```sql
SELECT customer_id, name FROM customers_2024
UNION
SELECT customer_id, name FROM customers_2025
ORDER BY customer_id;
```

**Expected output** (illustrative):

```text
 customer_id | name
-------------+---------
         101 | Alice
         102 | Bob
         103 | Carol
(3 rows)
```

**Why this output appears**: Rows from both years are stacked; duplicates (customers appearing in both tables) are removed. The final `ORDER BY` sorts the combined result.

##### Example 2: MySQL – UNION with compatible types

```sql
SELECT product_id, price FROM products
UNION
SELECT item_id, cost FROM archived_products;
```

**Expected output**: A single list of product IDs and prices, with duplicates removed.

**Why this output appears**: The columns are union-compatible (`INT`, `DECIMAL`); MySQL aligns the column names from the first query and eliminates duplicate rows.

#### Real-World Cases

- **Customer deduplication**: Merging customer lists from acquisitions.
- **Inventory consolidation**: Combining stock from multiple warehouses.
- **Event streams**: Stacking logs from multiple services into a distinct event list.

---

### 2. `UNION ALL`

#### Definitions

**Core Definition**  
`UNION ALL` combines the rows of two query results and preserves all rows, including duplicates.

**Technical Definition**  
`query1 UNION ALL query2` returns the concatenation of the rows from both operands without duplicate elimination. Every row from `query1` appears, followed by every row from `query2` (order unspecified unless an outer `ORDER BY` is added). This is the fastest set operator because it requires no deduplication step.

**Beginner-Friendly Explanation**  
`UNION ALL` stacks two lists exactly as they are, without removing anything.

#### Purposes

- To combine results when duplicates are known to be impossible or are acceptable.
- To achieve maximum performance by avoiding deduplication.
- To stack fact-table partitions or append-only logs.
- To preserve multiplicity for counting or auditing.

#### Syntax Rules and Structure

**Syntax**:

```sql
query1
UNION ALL
query2
[ORDER BY ...];
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `UNION ALL` | Set operator with duplicate preservation. |
| `query1`, `query2` | Union-compatible queries. |
| `ORDER BY` | Optional final sort. |

**Syntax rules**

- Same union-compatibility rules as `UNION`.
- Duplicates are not removed.
- `UNION ALL` is generally faster than `UNION` because it avoids sort/hash deduplication.

**Constraints and limitations**

- If duplicates are not acceptable, `UNION ALL` produces incorrect results; verify that the operands are disjoint or that duplicates are harmless.
- Row order is unspecified without a final `ORDER BY`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Stacking sales partitions

```sql
SELECT sale_id, amount FROM sales_2024
UNION ALL
SELECT sale_id, amount FROM sales_2025
ORDER BY sale_id;
```

**Expected output**:

```text
 sale_id | amount
---------+--------
       1 | 100.00
       2 | 150.00
       3 | 200.00
       4 | 120.00
(4 rows)
```

**Why this output appears**: All rows from both partitions are preserved; the final `ORDER BY` sorts them. If the same `sale_id` existed in both tables, both rows would appear.

##### Example 2: SQL Server – UNION ALL for performance

```sql
SELECT CustomerID, TotalDue FROM Sales.SalesOrderHeader
UNION ALL
SELECT CustomerID, TotalDue FROM Sales.SalesOrderDetail;
```

**Expected output**: Every row from both tables, with no deduplication.

**Why this output appears**: `UNION ALL` avoids the sort/hash step that `UNION` requires, making it the preferred choice when duplicates are not a concern.

#### Real-World Cases

- **Log aggregation**: Appending daily log partitions.
- **ETL staging**: Stacking source extracts without deduplication.
- **Audit trails**: Preserving every event, including repeats.

---

### 3. `INTERSECT`

#### Definitions

**Core Definition**  
`INTERSECT` returns only the rows that appear in both query results, eliminating duplicates.

**Technical Definition**  
`query1 INTERSECT query2` returns the set intersection: rows that are present in both operands. Duplicate rows are removed unless `INTERSECT ALL` is used. `INTERSECT` is commutative (`A INTERSECT B` = `B INTERSECT A`) and typically binds more tightly than `UNION` and `EXCEPT` in PostgreSQL and MySQL.

**Beginner-Friendly Explanation**  
`INTERSECT` gives you the rows that the two lists have in common.

#### Purposes

- To find records present in both datasets (common customers, overlapping products).
- To validate referential integrity between two tables.
- To implement set-based membership tests.
- To compare two result sets for overlap.

#### Syntax Rules and Structure

**Syntax**:

```sql
query1
INTERSECT
query2
[ORDER BY ...];
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `INTERSECT` | Set intersection operator. |
| `query1`, `query2` | Union-compatible queries. |

**Syntax rules**

- Union-compatible operands required.
- Duplicates are eliminated by default; `INTERSECT ALL` preserves multiplicity.
- `INTERSECT` binds more tightly than `UNION` and `EXCEPT` in PostgreSQL and MySQL; Oracle evaluates all set operators left to right at equal precedence.

**Constraints and limitations**

- `INTERSECT` can often be rewritten as an `EXISTS` or `JOIN` for better performance on large tables.
- MySQL only supports `INTERSECT` from version 8.0.31 onward.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Common products

```sql
SELECT product_id FROM inventory
INTERSECT
SELECT product_id FROM orders
ORDER BY product_id;
```

**Expected output**:

```text
 product_id
------------
        101
        103
(2 rows)
```

**Why this output appears**: Only product IDs present in both `inventory` and `orders` are returned; duplicates are removed.

##### Example 2: MySQL 8.0.31+ – INTERSECT with precedence

```sql
TABLE x UNION TABLE y INTERSECT TABLE z;
```

**Expected output**: Evaluated as `x UNION (y INTERSECT z)` because `INTERSECT` binds more tightly.

**Why this output appears**: MySQL gives `INTERSECT` higher precedence than `UNION`, so the intersection of `y` and `z` is computed first.

#### Real-World Cases

- **Customer overlap**: Customers who bought both product A and product B.
- **Inventory reconciliation**: Items present in both physical and system counts.
- **Fraud detection**: Transactions appearing in both suspicious and high-value lists.

---

### 4. `INTERSECT ALL`

#### Definitions

**Core Definition**  
`INTERSECT ALL` returns rows that appear in both query results, preserving duplicate multiplicity.

**Technical Definition**  
`query1 INTERSECT ALL query2` returns a row as many times as it appears in both operands, up to the minimum of its counts in the two inputs. If a row appears `m` times in `query1` and `n` times in `query2`, it appears `MIN(m, n)` times in the result. This is the multiset intersection.

**Beginner-Friendly Explanation**  
`INTERSECT ALL` keeps duplicates: if a value appears three times in the first list and two times in the second, it appears twice in the result.

#### Purposes

- To compute multiset intersection where multiplicity matters.
- To reconcile counts between two datasets.
- To preserve duplicate semantics in auditing.

#### Syntax Rules and Structure

**Syntax**:

```sql
query1
INTERSECT ALL
query2
[ORDER BY ...];
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `INTERSECT ALL` | Multiset intersection operator. |
| `query1`, `query2` | Union-compatible queries. |

**Syntax rules**

- Multiplicity is preserved using the minimum count per value.
- Supported by PostgreSQL and Oracle; MySQL 8.0 supports `INTERSECT` but not `INTERSECT ALL`; SQL Server does not support `INTERSECT ALL`.

**Constraints and limitations**

- Not universally supported; check engine compatibility.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Multiset intersection

```sql
SELECT * FROM (VALUES (1), (1), (2)) AS a(x)
INTERSECT ALL
SELECT * FROM (VALUES (1), (2), (2)) AS b(x);
```

**Expected output**:

```text
 x
---
 1
 2
(2 rows)
```

**Why this output appears**: The value `1` appears twice in `a` and once in `b`, so it appears once (the minimum). The value `2` appears once in `a` and twice in `b`, so it appears once.

#### Real-World Cases

- **Inventory count reconciliation**: Comparing multiset counts of items.
- **Duplicate-aware matching**: Finding overlapping records while respecting multiplicity.

---

### 5. `EXCEPT`

#### Definitions

**Core Definition**  
`EXCEPT` returns the distinct rows that appear in the first query result but not in the second.

**Technical Definition**  
`query1 EXCEPT query2` returns the set difference: rows present in `query1` that are not present in `query2`. Duplicates are eliminated unless `EXCEPT ALL` is used. `EXCEPT` is not commutative: `A EXCEPT B` is generally different from `B EXCEPT A`. In Oracle, `EXCEPT` is a synonym for `MINUS`.

**Beginner-Friendly Explanation**  
`EXCEPT` gives you the rows from the first list that are not in the second list.

#### Purposes

- To find missing records (rows in one table but not another).
- To reconcile datasets and detect differences.
- To implement anti-join semantics.
- To subtract one set of values from another.

#### Syntax Rules and Structure

**Syntax**:

```sql
query1
EXCEPT
query2
[ORDER BY ...];
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `EXCEPT` | Set difference operator. |
| `query1` | Left operand (rows to keep). |
| `query2` | Right operand (rows to subtract). |

**Syntax rules**

- Union-compatible operands required.
- Duplicates eliminated by default; `EXCEPT ALL` preserves multiplicity.
- `EXCEPT` binds at the same level as `UNION` in PostgreSQL; `INTERSECT` binds more tightly.
- Oracle evaluates set operators left to right at equal precedence unless parenthesized.

**Constraints and limitations**

- `EXCEPT` can often be rewritten as `NOT EXISTS` or `LEFT JOIN ... IS NULL` for better performance on large tables.
- MySQL supports `EXCEPT` only from version 8.0.31 onward.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Products not ordered

```sql
SELECT product_id FROM inventory
EXCEPT
SELECT product_id FROM orders
ORDER BY product_id;
```

**Expected output**:

```text
 product_id
------------
        102
        104
(2 rows)
```

**Why this output appears**: Product IDs present in `inventory` but not in `orders` are returned; duplicates are removed.

##### Example 2: Oracle – MINUS as a synonym for EXCEPT

```sql
SELECT product_id FROM order_items
MINUS
SELECT product_id FROM inventories
ORDER BY product_id;
```

**Expected output**: Distinct product IDs in `order_items` that are not in `inventories`.

**Why this output appears**: Oracle’s `MINUS` is semantically identical to `EXCEPT`; duplicates are eliminated.

#### Real-World Cases

- **Data reconciliation**: Records in the source but missing from the target.
- **Churn analysis**: Customers in the prior period but not the current period.
- **Inventory shrink**: Items expected but not found in stock counts.

---

### 6. `EXCEPT ALL`

#### Definitions

**Core Definition**  
`EXCEPT ALL` returns rows from the first query result that are not in the second, preserving duplicate multiplicity.

**Technical Definition**  
`query1 EXCEPT ALL query2` returns rows from `query1` after subtracting the multiset of `query2`. If a value appears `m` times in `query1` and `n` times in `query2`, it appears `MAX(0, m − n)` times in the result. This is the multiset difference.

**Beginner-Friendly Explanation**  
`EXCEPT ALL` subtracts duplicates: if a value appears three times in the first list and two times in the second, it appears once in the result.

#### Purposes

- To compute multiset difference where multiplicity matters.
- To reconcile counts and inventory levels.
- To preserve duplicate semantics in audit comparisons.

#### Syntax Rules and Structure

**Syntax**:

```sql
query1
EXCEPT ALL
query2
[ORDER BY ...];
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `EXCEPT ALL` | Multiset difference operator. |
| `query1`, `query2` | Union-compatible queries. |

**Syntax rules**

- Multiplicity is preserved using `MAX(0, count_in_query1 − count_in_query2)`.
- Supported by PostgreSQL and Oracle (`MINUS ALL`); MySQL 8.0 supports `EXCEPT` but not `EXCEPT ALL`; SQL Server does not support `EXCEPT ALL`.

**Constraints and limitations**

- Not universally supported.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Multiset difference

```sql
SELECT * FROM (VALUES (1), (1), (2), (3)) AS a(x)
EXCEPT ALL
SELECT * FROM (VALUES (1), (2), (2)) AS b(x);
```

**Expected output**:

```text
 x
---
 1
 3
(2 rows)
```

**Why this output appears**: Value `1` appears twice in `a` and once in `b`, so one `1` remains. Value `2` appears once in `a` and twice in `b`, so zero `2`s remain. Value `3` appears once in `a` and not in `b`, so one `3` remains.

#### Real-World Cases

- **Inventory surplus**: Items where physical count exceeds system count.
- **Multiset reconciliation**: Comparing event counts with multiplicity.

---

### 7. `MINUS` in Some Systems

#### Definitions

**Core Definition**  
`MINUS` is Oracle’s name for the set difference operator; it is semantically identical to `EXCEPT`.

**Technical Definition**  
Oracle supports `MINUS` and `MINUS ALL` with the same semantics as `EXCEPT` and `EXCEPT ALL`. `MINUS` returns distinct rows from the first query not present in the second; `MINUS ALL` preserves multiplicity. Oracle also treats `EXCEPT` as a synonym for `MINUS`. MySQL does not support `MINUS`; it uses `EXCEPT` from 8.0.31 onward.

**Beginner-Friendly Explanation**  
`MINUS` is just Oracle’s word for `EXCEPT`—it subtracts the second list from the first.

#### Purposes

- To write portable SQL for Oracle environments.
- To use the traditional Oracle terminology for set difference.
- To leverage `MINUS ALL` for multiset subtraction.

#### Syntax Rules and Structure

**Syntax**:

```sql
query1
MINUS [ALL]
query2
[ORDER BY ...];
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `MINUS` | Oracle’s set difference operator. |
| `MINUS ALL` | Multiset difference in Oracle. |

**Syntax rules**

- Same union-compatibility rules as `EXCEPT`.
- Oracle evaluates all set operators left to right at equal precedence unless parentheses specify otherwise.
- `EXCEPT` is accepted as a synonym in Oracle.

**Constraints and limitations**

- Not portable to PostgreSQL, SQL Server, or MySQL (which use `EXCEPT`).
- MySQL does not support `MINUS` at all.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Oracle – MINUS for missing records

```sql
SELECT customer_id FROM customers_2024
MINUS
SELECT customer_id FROM customers_2025;
```

**Expected output**: Customer IDs present in 2024 but not in 2025 (distinct).

**Why this output appears**: `MINUS` subtracts the 2025 customer set from the 2024 set, returning only those who did not appear in 2025.

#### Real-World Cases

- **Oracle-centric enterprises**: Standardizing on `MINUS` for set difference.
- **Churn reporting**: Customers lost between periods.

---

### 8. Set Operation Rules

#### Definitions

**Core Definition**  
Set operation rules are the syntactic and semantic constraints that govern how set operators can be combined, parenthesized, and ordered.

**Technical Definition**  
Set operators combine query expressions. Without parentheses, precedence determines evaluation order: in PostgreSQL and MySQL, `INTERSECT` binds more tightly than `UNION` and `EXCEPT`; in Oracle, all set operators have equal precedence and evaluate left to right. Parentheses override precedence. `ORDER BY` applies only to the final result of the entire set operation and must appear last. Individual operands cannot contain `ORDER BY`, `LIMIT`, or locking clauses without being parenthesized (in PostgreSQL).

**Beginner-Friendly Explanation**  
When you mix `UNION`, `INTERSECT`, and `EXCEPT`, the database follows precedence rules: `INTERSECT` usually happens first. You can use parentheses to force a different order. The final `ORDER BY` sorts the whole result, not individual pieces.

#### Purposes

- To correctly compose multi-operator set expressions.
- To override default precedence with parentheses.
- To apply ordering to the combined result.
- To avoid syntax errors when operands contain `LIMIT` or `ORDER BY`.

#### Syntax Rules and Structure

**Precedence patterns**:

```sql
-- PostgreSQL/MySQL: INTERSECT binds tighter
query1 UNION query2 INTERSECT query3
-- Equivalent to:
query1 UNION (query2 INTERSECT query3)

-- Parentheses override:
(query1 UNION query2) INTERSECT query3

-- Oracle: equal precedence, left to right
query1 UNION query2 INTERSECT query3
-- Evaluated as:
(query1 UNION query2) INTERSECT query3
```

**Component breakdown**

| Rule | Description |
|------|-------------|
| Precedence | `INTERSECT` > `UNION`/`EXCEPT` in PostgreSQL/MySQL; equal in Oracle. |
| Parentheses | Override precedence and group operands. |
| `ORDER BY` | Applies to the entire set operation; must be last. |
| Operand restrictions | Individual operands with `ORDER BY`/`LIMIT` need parentheses. |

**Syntax rules**

- Use parentheses to make evaluation order explicit.
- `ORDER BY` at the end sorts the combined result.
- In PostgreSQL, `LIMIT` without parentheses applies to the entire union, not the last `SELECT`.

**Constraints and limitations**

- Precedence differences across engines can cause portability bugs.
- Parenthesizing every operand is the safest approach for complex expressions.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Precedence and parentheses

```sql
-- INTERSECT binds tighter than UNION.
SELECT 1 UNION SELECT 2 INTERSECT SELECT 2;
-- Result: 1, 2 (because 2 INTERSECT 2 = 2, then UNION with 1)

-- Force different order.
(SELECT 1 UNION SELECT 2) INTERSECT SELECT 2;
-- Result: 2
```

**Expected output**: First query returns `{1, 2}`; second returns `{2}`.

**Why this output appears**: In the first query, `INTERSECT` is evaluated first, yielding `{2}`, then `UNION` with `{1}` yields `{1, 2}`. Parentheses change the grouping.

##### Example 2: PostgreSQL – LIMIT scope

```sql
SELECT a FROM b UNION SELECT x FROM y LIMIT 10;
```

**Expected output**: The `LIMIT 10` applies to the entire union, not to the second `SELECT`.

**Why this output appears**: Without parentheses, `LIMIT` is understood as applying to the set operation’s output, not to the individual `SELECT`.

#### Real-World Cases

- **Complex reconciliation queries**: Combining multiple `EXCEPT` and `UNION` operations with explicit parentheses.
- **Cross-engine portability**: Writing parenthesized expressions to avoid precedence differences.

---

### 9. Column Compatibility

#### Definitions

**Core Definition**  
Column compatibility (union compatibility) requires that all queries combined by a set operator return the same number of columns in the same order.

**Technical Definition**  
For a set operation to be valid, each operand must project the same number of columns. Column names need not match; the final result’s column names are taken from the first query. Columns are matched by ordinal position, not by name. For example, `SELECT a, b FROM t1 UNION SELECT x, y FROM t2` is valid if both queries return two columns, even if the names differ.

**Beginner-Friendly Explanation**  
The lists you stack must have the same number of columns, in the same order. The names of the columns do not need to match; the first query’s names become the result’s names.

#### Purposes

- To ensure valid set operations.
- To align heterogeneous queries by position.
- To understand why mismatched column counts cause errors.

#### Syntax Rules and Structure

**Valid pattern**:

```sql
SELECT id, name FROM employees
UNION
SELECT dept_id, dept_name FROM departments;
```

**Invalid pattern**:

```sql
SELECT id, name, salary FROM employees
UNION
SELECT dept_id, dept_name FROM departments;  -- ERROR: different column counts
```

**Component breakdown**

| Requirement | Description |
|-------------|-------------|
| Same column count | Each `SELECT` must return the same number of columns. |
| Ordinal matching | Columns are matched by position, not name. |
| Names from first query | Final column names come from the first `SELECT`. |

**Syntax rules**

- Use `NULL` or typed placeholders to pad missing columns if necessary.
- Use column aliases in the first query to name the output columns.
- Verify column order carefully; a swapped column can silently produce wrong results.

**Constraints and limitations**

- Column count mismatches cause syntax/semantic errors.
- Positional matching can be fragile when queries evolve; document the mapping.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Matching column counts

```sql
SELECT 1 AS id, 'A' AS name
UNION
SELECT 2, 'B';
```

**Expected output**:

```text
 id | name
----+------
  1 | A
  2 | B
(2 rows)
```

**Why this output appears**: Both queries return two columns; the second query’s columns are unnamed but aligned by position with the first query’s `id` and `name`.

#### Real-World Cases

- **Cross-system integration**: Aligning columns from different schemas by position.
- **Padding with NULLs**: Adding `NULL` columns to satisfy column-count requirements.

---

### 10. Data-Type Compatibility

#### Definitions

**Core Definition**  
Data-type compatibility means that corresponding columns in set-operation operands must have types that can be compared or implicitly converted by the engine.

**Technical Definition**  
The SQL standard requires corresponding columns to be of compatible types. Engines apply type resolution rules to determine the result type. In PostgreSQL, the result type is chosen using type resolution rules; in Oracle, the type is determined by data-type group and numeric precedence (e.g., `BINARY_DOUBLE` > `BINARY_FLOAT` > `NUMBER`; `VARCHAR2` over `CHAR`); in SQLite, implicit comparisons are performed without affinity conversion. Oracle does not perform implicit conversion across data-type groups (e.g., character vs. numeric), and raises an error if types are incompatible. MySQL and SQL Server generally perform implicit conversion but may raise errors for incompatible types.

**Beginner-Friendly Explanation**  
The columns you match up must have types that the database can reconcile—like two numbers, or two strings. You cannot safely mix a number and a string without an explicit conversion.

#### Purposes

- To ensure set operations do not fail due to type mismatches.
- To understand result types when operands have different but compatible types.
- To add explicit casts where implicit conversion is unsafe.

#### Syntax Rules and Structure

**Explicit casting pattern**:

```sql
SELECT CAST(id AS VARCHAR) FROM t1
UNION
SELECT code FROM t2;
```

**Component breakdown**

| Engine | Type resolution rule |
|--------|----------------------|
| PostgreSQL | Type resolution rules choose the "preferred" type. |
| Oracle | Numeric precedence (`BINARY_DOUBLE` > `BINARY_FLOAT` > `NUMBER`); character types resolved to `VARCHAR2` if lengths differ. |
| SQLite | Implicit comparison without affinity; types compared as stored. |
| MySQL/SQL Server | Implicit conversion when possible; errors for incompatible types. |

**Syntax rules**

- Use `CAST` or `CONVERT` to make types explicit.
- Oracle does not implicitly convert across data-type groups (e.g., `'3'` and `3`).
- SQLite compares values as-is without affinity conversion for set operations.

**Constraints and limitations**

- Implicit conversion can cause unexpected sorting or comparison behavior.
- Some engines (Oracle) are stricter than others.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Oracle – Valid and invalid type combinations

```sql
-- Valid: numeric types with precedence.
SELECT 3 FROM DUAL
INTERSECT
SELECT 3f FROM DUAL;
-- Result: BINARY_FLOAT 3

-- Invalid: character vs. numeric across data-type groups.
SELECT '3' FROM DUAL
INTERSECT
SELECT 3f FROM DUAL;
-- ERROR
```

**Expected output**: The first query succeeds; the second raises an error.

**Why this output appears**: Oracle implicitly converts `3` to `BINARY_FLOAT` for the numeric case, but does not convert `'3'` to a numeric type because they are in different data-type groups.

#### Real-World Cases

- **Cross-database integration**: Casting IDs to strings when combining sources.
- **Numeric precision**: Understanding which type wins when combining `NUMBER` and `BINARY_DOUBLE`.

---

### 11. Ordering

#### Definitions

**Core Definition**  
Ordering in set operations is controlled by a single `ORDER BY` clause that applies to the final combined result, not to individual operands.

**Technical Definition**  
`ORDER BY` may appear only at the end of the entire set operation. It can reference output column names (from the first query) or ordinal positions. Individual `SELECT` statements within a set operation cannot contain their own `ORDER BY` unless they are parenthesized and the `ORDER BY` is scoped to that operand (which is generally not allowed in PostgreSQL without a subquery). The final `ORDER BY` sorts the combined set.

**Beginner-Friendly Explanation**  
You put one `ORDER BY` at the very end to sort the whole combined list; you cannot sort the individual pieces separately.

#### Purposes

- To present the combined result in a deterministic order.
- To use column aliases from the first query for sorting.
- To avoid syntax errors from misplaced `ORDER BY` clauses.

#### Syntax Rules and Structure

**Syntax**:

```sql
query1
UNION
query2
ORDER BY column_name_or_position;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `ORDER BY` | Final sort clause. |
| `column_name` | Output column name (from first query). |
| `position` | Ordinal position (1-based). |

**Syntax rules**

- `ORDER BY` must appear after all set operations.
- It cannot appear inside an operand without parentheses (and even then, some engines disallow it).
- Column names refer to the first query’s output.

**Constraints and limitations**

- Sorting the final result can be expensive for large unions.
- Without `ORDER BY`, row order is unspecified.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Final ORDER BY

```sql
SELECT id, name FROM t1
UNION
SELECT id, name FROM t2
ORDER BY name, id;
```

**Expected output**: The combined rows sorted by `name`, then `id`.

**Why this output appears**: The single `ORDER BY` at the end sorts the entire union result.

##### Example 2: SQL Server – ORDER BY with ordinal

```sql
SELECT CustomerID, TotalDue FROM Table1
UNION ALL
SELECT CustomerID, TotalDue FROM Table2
ORDER BY 2 DESC;
```

**Expected output**: Combined rows sorted by `TotalDue` descending (column 2).

**Why this output appears**: `ORDER BY 2` refers to the second output column of the combined result.

#### Real-World Cases

- **Reporting**: Sorting a merged dataset for presentation.
- **Pagination**: Applying `ORDER BY` with `OFFSET/FETCH` to a union.

---

### 12. Performance

#### Definitions

**Core Definition**  
Performance of set operations is determined primarily by whether duplicate elimination is required and by the algorithms used for deduplication (sort or hash).

**Technical Definition**  
`UNION ALL` is generally the fastest set operator because it concatenates results without deduplication. `UNION`, `INTERSECT`, and `EXCEPT` (and their `ALL` variants where duplicates are preserved) must answer the question “have I seen this row before?” This requires either sorting the combined result (`O(n log n)`) or building a hash table (`O(n)` expected). Sort-based deduplication can spill to disk for large datasets, causing dramatic slowdowns. Hash-based deduplication is faster when memory suffices. On large tables, native `EXCEPT` can often be rewritten as `NOT EXISTS` or `LEFT JOIN ... IS NULL`; `INTERSECT` can be rewritten as `EXISTS` or `JOIN` for better performance.

**Beginner-Friendly Explanation**  
`UNION ALL` is fast because it just stacks rows. `UNION` is slower because it has to check for and remove duplicates, which means sorting or hashing the data. On big tables, that deduplication step can become the bottleneck.

#### Purposes

- To choose `UNION ALL` when duplicates are acceptable or impossible.
- To understand when rewriting `EXCEPT` or `INTERSECT` as joins improves performance.
- To diagnose slow set operations caused by disk-spilling sorts.

#### Syntax Rules and Structure

**Performance decision**:

```sql
-- Fast: no deduplication.
SELECT ... UNION ALL SELECT ...;

-- Slower: deduplication required.
SELECT ... UNION SELECT ...;

-- Alternative to EXCEPT:
SELECT ... FROM t1
WHERE NOT EXISTS (SELECT 1 FROM t2 WHERE t2.key = t1.key);

-- Alternative to INTERSECT:
SELECT ... FROM t1
WHERE EXISTS (SELECT 1 FROM t2 WHERE t2.key = t1.key);
```

**Component breakdown**

| Operation | Cost driver |
|-----------|-------------|
| `UNION ALL` | Concatenation only. |
| `UNION` | Sort or hash deduplication. |
| `INTERSECT` | Hash or sort build + probe. |
| `EXCEPT` | Hash or sort build + probe (anti-join). |

**Syntax rules**

- Use `UNION ALL` when duplicates are acceptable.
- Consider `NOT EXISTS`/`EXISTS` rewrites for large datasets.
- Ensure appropriate indexes on join keys for rewrite patterns.

**Constraints and limitations**

- Rewrites may change semantics if not carefully constructed (e.g., `NULL` handling).
- Sort-based deduplication can spill to disk; monitor temp space.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – UNION ALL vs UNION performance

```sql
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM large_table_1
UNION ALL
SELECT * FROM large_table_2;

EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM large_table_1
UNION
SELECT * FROM large_table_2;
```

**Expected output** (plans, illustrative):

```text
-- UNION ALL: Append only
 Append (actual time=0.010..0.020 rows=100000 loops=1)
   ->  Seq Scan on large_table_1
   ->  Seq Scan on large_table_2

-- UNION: Append + HashAggregate or Sort/Unique
 HashAggregate (actual time=50.000..60.000 rows=80000 loops=1)
   ->  Append
         ->  Seq Scan on large_table_1
         ->  Seq Scan on large_table_2
```

**Why this output appears**: `UNION ALL` only appends; `UNION` adds a `HashAggregate` (or `Sort`/`Unique`) node to remove duplicates, increasing runtime and memory/disk usage.

##### Example 2: SQL Server – EXCEPT vs NOT EXISTS

```sql
-- Native EXCEPT.
SELECT CustomerID FROM Customers
EXCEPT
SELECT CustomerID FROM Orders;

-- Anti-join rewrite.
SELECT c.CustomerID
FROM Customers c
WHERE NOT EXISTS (SELECT 1 FROM Orders o WHERE o.CustomerID = c.CustomerID);
```

**Expected output**: The same set of customer IDs, but the `NOT EXISTS` plan can use an index on `Orders.CustomerID` and avoid a full sort/hash of both inputs.

**Why this output appears**: The anti-join rewrite can leverage indexes and avoid materializing the entire difference set.

#### Real-World Cases

- **ETL reconciliation**: Replacing `EXCEPT` with `NOT EXISTS` on 50-million-row tables to meet SLAs.
- **Dashboard queries**: Using `UNION ALL` instead of `UNION` when source partitions are disjoint.
- **Data integration**: Materializing frequent `UNION ALL` views into tables to avoid recomputation.

---

### 13. Duplicate Handling

#### Definitions

**Core Definition**  
Duplicate handling determines whether duplicate rows are eliminated (default) or preserved (`ALL` variant) in a set operation.

**Technical Definition**  
The SQL standard defines two modes for each set operator: `DISTINCT` (default) eliminates duplicate rows, and `ALL` preserves them. `UNION DISTINCT` removes duplicates; `UNION ALL` keeps all rows. `INTERSECT` returns distinct common rows; `INTERSECT ALL` returns rows with multiplicity equal to the minimum count in both inputs. `EXCEPT` returns distinct rows from the first input not in the second; `EXCEPT ALL` returns rows with multiplicity equal to the positive difference in counts. MySQL supports explicit `DISTINCT` or `ALL` modifiers; PostgreSQL and SQL Server use `UNION`/`UNION ALL` without a separate `DISTINCT` keyword.

**Beginner-Friendly Explanation**  
By default, set operations remove duplicates; adding `ALL` keeps them. `UNION` is like “combine and deduplicate”; `UNION ALL` is like “just stack everything.”

#### Purposes

- To choose the correct semantics for the business problem.
- To avoid unnecessary deduplication cost when duplicates are acceptable.
- To preserve multiplicity for counting and auditing.
- To understand why `UNION` and `UNION ALL` give different row counts.

#### Syntax Rules and Structure

**Duplicate modes**:

```sql
query1 UNION [ALL | DISTINCT] query2
query1 INTERSECT [ALL | DISTINCT] query2
query1 EXCEPT [ALL | DISTINCT] query2
```

**Component breakdown**

| Mode | Behavior |
|------|----------|
| `DISTINCT` (default) | Remove duplicates. |
| `ALL` | Preserve duplicates. |
| `INTERSECT ALL` | Multiplicity = MIN(count in A, count in B). |
| `EXCEPT ALL` | Multiplicity = MAX(0, count in A − count in B). |

**Syntax rules**

- `DISTINCT` is the default and may be omitted.
- `ALL` is supported for `UNION` in all engines; `INTERSECT ALL` and `EXCEPT ALL` are not universally supported (PostgreSQL and Oracle support them; MySQL 8.0 and SQL Server do not support `INTERSECT ALL`/`EXCEPT ALL`).
- MySQL allows explicit `DISTINCT` or `ALL` with `UNION`, `INTERSECT`, and `EXCEPT`.

**Constraints and limitations**

- Duplicate elimination adds cost; use `ALL` when safe.
- `NULL` values are treated as equal for duplicate elimination in most engines (all `NULL`s collapse to one).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – UNION DISTINCT vs UNION ALL

```sql
SELECT 1 UNION SELECT 1;        -- Result: 1
SELECT 1 UNION ALL SELECT 1;    -- Result: 1, 1
SELECT 1 UNION DISTINCT SELECT 1; -- Result: 1
```

**Expected output**: First and third return one row; second returns two rows.

**Why this output appears**: `UNION` defaults to `DISTINCT` (duplicates removed); `UNION ALL` preserves duplicates; `UNION DISTINCT` is explicit but equivalent to the default.

##### Example 2: PostgreSQL – INTERSECT ALL vs INTERSECT

```sql
SELECT * FROM (VALUES (1),(1),(2)) a(x)
INTERSECT
SELECT * FROM (VALUES (1),(2),(2)) b(x);
-- Result: 1, 2 (distinct)

SELECT * FROM (VALUES (1),(1),(2)) a(x)
INTERSECT ALL
SELECT * FROM (VALUES (1),(2),(2)) b(x);
-- Result: 1, 2 (multiset: min counts)
```

**Expected output**: `INTERSECT` returns `{1, 2}`; `INTERSECT ALL` returns `{1, 2}` here because the minimum counts are one each.

**Why this output appears**: `INTERSECT` deduplicates; `INTERSECT ALL` preserves the minimum multiplicity, which is one for both values in this example.

#### Real-World Cases

- **Counting distinct customers**: `UNION` across channels to get unique customers.
- **Event counting**: `UNION ALL` to preserve every event occurrence.
- **Inventory reconciliation**: `EXCEPT ALL` to compute surplus counts.

---

## References

- PostgreSQL Documentation – Combining Queries (`UNION`, `INTERSECT`, `EXCEPT`) – https://www.postgresql.org/docs/current/queries-union.html 
- MySQL 8.0 Reference Manual – Set Operations with UNION, INTERSECT, and EXCEPT – https://dev.mysql.com/doc/refman/8.0/en/set-operations.html 
- Microsoft Learn – SELECT (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/queries/select-transact-sql 
- Oracle Database SQL Language Reference – The Set Operators – https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/The-UNION-ALL-INTERSECT-MINUS-Operators.html 
- SQLite Documentation – Compound SELECT Statements – https://sqlite.org/lang_select.html#compound 
- SQL Engineering Handbook – Set Operator Performance and Optimization – https://github.com/theammarngp-makes/SQL-Engineering-Handbook/blob/main/13_SET_OPERATORS/06_PERFORMANCE_AND_OPTIMIZATION.md 
- SQL Authority – Union vs. Union All Performance – https://blog.sqlauthority.com/2007/03/10/sql-server-union-vs-union-all-which-is-better-for-performance/ 
- SQL:1992 Standard – Set Operations (ISO/IEC 9075:1992)