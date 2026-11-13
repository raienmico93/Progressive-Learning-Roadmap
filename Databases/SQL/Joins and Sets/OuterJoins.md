# SQL Outer Joins: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An outer join is a relational operation that combines rows from two tables and preserves unmatched rows from one or both sides, filling the missing columns with `NULL` values.

**Technical Definition**
In SQL, an outer join extends the inner join by including rows from one or both input relations that have no matching counterpart in the other relation. The SQL standard defines three outer join types: `LEFT OUTER JOIN` (preserves all rows from the left table), `RIGHT OUTER JOIN` (preserves all rows from the right table), and `FULL OUTER JOIN` (preserves all rows from both tables). For rows without a match, the columns of the non-preserved side are set to `NULL`. Outer joins are used to answer questions about missing relationships, to preserve complete lists while optionally attaching related data, and to implement anti-joins and semi-joins via `NULL` checks or `EXISTS`/`IN` predicates .

**Beginner-Friendly Explanation**
An inner join shows only rows that have a match in both tables. An outer join says: "Show me **all** the rows from one table (or both), and if there's a match in the other table, attach it; if not, just leave those columns blank (NULL)." For example, a `LEFT JOIN` between customers and orders shows **every** customer—even those who have never placed an order.

---

### Key Characteristics

1. **Row Preservation**: Outer joins guarantee that all rows from the "preserved" side appear in the result, regardless of whether a match exists .
2. **NULL Padding**: Unmatched columns from the non-preserved side are filled with `NULL`, creating a distinguishable pattern that can be used to identify non-matches .
3. **Direction Matters**: `LEFT JOIN` and `RIGHT JOIN` are mirror images; swapping the table order and the join direction produces equivalent results (though column order may differ).
4. **`FULL OUTER JOIN` Is the Union of Left and Right**: A full outer join returns matched rows plus unmatched rows from both sides .
5. **`OUTER` Keyword Is Optional**: `LEFT JOIN` and `LEFT OUTER JOIN` are syntactically and semantically identical in all major RDBMS.
6. **MySQL Lacks `FULL OUTER JOIN`**: MySQL does not support `FULL OUTER JOIN` natively; it must be emulated with `UNION` of `LEFT` and `RIGHT` joins.
7. **Anti-Joins and Semi-Joins**: Outer joins with `IS NULL` checks implement anti-joins (finding non-matches); `EXISTS`/`IN` subqueries implement semi-joins (finding matches without duplicating rows).

---

### Prerequisites

- **Inner joins**: Understanding of `INNER JOIN`, join conditions, and join keys.
- **`NULL` semantics**: How `NULL` behaves in comparisons (`NULL = NULL` is `UNKNOWN`).
- **Primary/foreign key relationships**: To understand which side is the "one" and which is the "many."
- **Basic `SELECT` syntax**: `SELECT`, `FROM`, `WHERE`, `GROUP BY`.

---

### Related Programming Areas

| Area | Connection |
|------|------------|
| **Reporting** | Left joins are used to preserve complete lists (e.g., all products, even unsold ones). |
| **Data Quality** | Anti-joins identify orphaned records (e.g., orders without customers). |
| **ETL** | Full outer joins reconcile two data sources. |
| **Application Development** | ORMs use left joins for optional relationships. |
| **Query Optimisation** | Outer joins constrain join reordering; the optimiser must preserve row-preservation semantics. |

---

### Core Concepts / Features

1. **`LEFT JOIN`**
2. **`LEFT OUTER JOIN`**
3. **`RIGHT JOIN`**
4. **`RIGHT OUTER JOIN`**
5. **`FULL JOIN`**
6. **`FULL OUTER JOIN`**
7. **NULL Padding**
8. **Join Direction**
9. **Anti-Joins**
10. **Semi-Joins**
11. **Identifying Unmatched Records**
12. **Preserving Rows from One Side**

---

## 1. `LEFT JOIN`

### Definitions

**Core Definition**
`LEFT JOIN` returns all rows from the left table, plus matching rows from the right table; unmatched right-side columns are `NULL`.

**Technical Definition**
`T1 LEFT JOIN T2 ON condition` produces a result set containing (a) all rows from `T1` that have at least one match in `T2` (paired with their matches), and (b) all rows from `T1` that have no match in `T2`, with all `T2` columns set to `NULL`. It is equivalent to `LEFT OUTER JOIN` .

**Beginner-Friendly Explanation**
`LEFT JOIN` says: "Keep everything from the left table. If the right table has a match, show it; if not, show `NULL`s."

---

### Purposes

- To preserve all rows from the left table while optionally attaching related data.
- To display complete lists (e.g., all customers, with order details where available).
- To identify missing relationships by checking for `NULL`s.
- To avoid losing rows when the related table has no matching data.

---

### Syntax Rules and Structure

**General Syntax**
```
SELECT select_list
FROM left_table
LEFT JOIN right_table
  ON join_condition;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `left_table` | The table whose rows are preserved. |
| `LEFT JOIN` | Join type keyword. |
| `right_table` | The table whose matches are optionally attached. |
| `ON join_condition` | The matching predicate. |

**Syntax Rules**

1. `LEFT JOIN` is equivalent to `LEFT OUTER JOIN`.
2. The `ON` clause is required (unless using `USING`).
3. All rows from `left_table` appear in the result, even if `right_table` has no match.
4. For unmatched rows, all columns from `right_table` are `NULL`.

**Constraints and Limitations**

- If the join condition is placed in the `WHERE` clause instead of `ON`, the outer-join semantics may be lost (e.g., `WHERE right.col = value` filters out `NULL`-padded rows).
- `LEFT JOIN` can produce more rows than the left table if the right table has multiple matches per left row.

---

### Annotated Code Examples

**Example 1: Basic `LEFT JOIN`**

```sql
SELECT c.customer_name, o.order_id, o.amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;
```

**Expected Output (illustrative)**

| customer_name | order_id | amount |
|---------------|----------|--------|
| Alice         | 1001     | 250    |
| Alice         | 1003     | 300    |
| Bob           | 1002     | 150    |
| Carol         | NULL     | NULL   |

**Why This Output Occurs**: Carol has no orders; she still appears because `customers` is the left (preserved) table, with `NULL`s for the order columns .

---

### Real-World Cases

- **Product catalogues**: Listing all products, including those never sold.
- **Customer lists**: Showing all customers, with order history where available.
- **Inventory**: Displaying all warehouses, including those with no stock.

---

## 2. `LEFT OUTER JOIN`

### Definitions

**Core Definition**
`LEFT OUTER JOIN` is the explicit SQL-standard syntax for `LEFT JOIN`; both are identical in meaning.

**Technical Definition**
The `OUTER` keyword is optional and has no effect on semantics. The SQL standard defines `LEFT OUTER JOIN` as the formal name; most RDBMS accept `LEFT JOIN` as a shorthand .

**Beginner-Friendly Explanation**
`LEFT OUTER JOIN` is exactly the same as `LEFT JOIN`—the word `OUTER` is just more explicit.

---

### Purposes

- To write SQL-standard-conforming code.
- To make the outer-join nature explicit for readability.
- To maintain compatibility with SQL standard documentation.

---

### Syntax Rules and Structure

```
SELECT select_list
FROM left_table
LEFT OUTER JOIN right_table
  ON join_condition;
```

**Syntax Rules**

1. `LEFT OUTER JOIN` is semantically identical to `LEFT JOIN`.
2. All other rules from `LEFT JOIN` apply.

---

### Annotated Code Examples

**Example 1: `LEFT OUTER JOIN` (Same Result as `LEFT JOIN`)**

```sql
SELECT c.customer_name, o.order_id
FROM customers c
LEFT OUTER JOIN orders o ON c.customer_id = o.customer_id;
```

**Expected Output**: Identical to the `LEFT JOIN` example above.

---

### Real-World Cases

- **Standard-compliant SQL**: Using explicit `OUTER` in documentation or training materials.
- **Code review**: Making the outer-join semantics unmistakable.

---

## 3. `RIGHT JOIN`

### Definitions

**Core Definition**
`RIGHT JOIN` returns all rows from the right table, plus matching rows from the left table; unmatched left-side columns are `NULL`.

**Technical Definition**
`T1 RIGHT JOIN T2 ON condition` is the mirror image of `LEFT JOIN`: it preserves all rows from `T2`, attaching matches from `T1` where available and padding with `NULL` where not. It is equivalent to `RIGHT OUTER JOIN` and logically equivalent to `T2 LEFT JOIN T1`.

**Beginner-Friendly Explanation**
`RIGHT JOIN` says: "Keep everything from the right table. If the left table has a match, show it; if not, show `NULL`s."

---

### Purposes

- To preserve all rows from the right table while optionally attaching left-table data.
- To express joins where the "complete list" is on the right side of the `FROM` clause.
- To mirror a `LEFT JOIN` when table order is fixed.

---

### Syntax Rules and Structure

```
SELECT select_list
FROM left_table
RIGHT JOIN right_table
  ON join_condition;
```

**Syntax Rules**

1. `RIGHT JOIN` is equivalent to `RIGHT OUTER JOIN`.
2. All rows from `right_table` appear in the result.
3. For unmatched rows, all columns from `left_table` are `NULL`.

**Constraints and Limitations**

- `RIGHT JOIN` is often less readable than `LEFT JOIN` because the preserved table is not the first one listed; many style guides recommend rewriting `RIGHT JOIN` as `LEFT JOIN` by swapping table order.
- MySQL and PostgreSQL support `RIGHT JOIN`; SQL Server also supports it.

---

### Annotated Code Examples

**Example 1: Basic `RIGHT JOIN`**

```sql
SELECT c.customer_name, o.order_id, o.amount
FROM orders o
RIGHT JOIN customers c ON o.customer_id = c.customer_id;
```

**Expected Output (illustrative)**

| customer_name | order_id | amount |
|---------------|----------|--------|
| Alice         | 1001     | 250    |
| Alice         | 1003     | 300    |
| Bob           | 1002     | 150    |
| Carol         | NULL     | NULL   |

**Why This Output Occurs**: `customers` is the right table and is preserved; Carol (no orders) appears with `NULL` order columns. This is equivalent to `customers LEFT JOIN orders` .

---

### Real-World Cases

- **Reverse-direction joins**: When the "complete list" table is naturally written second.
- **Legacy SQL**: Maintaining existing queries that use `RIGHT JOIN`.

---

## 4. `RIGHT OUTER JOIN`

### Definitions

**Core Definition**
`RIGHT OUTER JOIN` is the explicit SQL-standard syntax for `RIGHT JOIN`; both are identical in meaning.

**Technical Definition**
The `OUTER` keyword is optional and has no semantic effect. `RIGHT OUTER JOIN` is the formal name .

**Beginner-Friendly Explanation**
`RIGHT OUTER JOIN` is exactly the same as `RIGHT JOIN`.

---

### Purposes

- To write SQL-standard-conforming code.
- To make the outer-join nature explicit.

---

### Syntax Rules and Structure

```
SELECT select_list
FROM left_table
RIGHT OUTER JOIN right_table
  ON join_condition;
```

**Syntax Rules**

1. Semantically identical to `RIGHT JOIN`.
2. All `RIGHT JOIN` rules apply.

---

### Annotated Code Examples

**Example 1: `RIGHT OUTER JOIN`**

```sql
SELECT o.order_id, c.customer_name
FROM orders o
RIGHT OUTER JOIN customers c ON o.customer_id = c.customer_id;
```

**Expected Output**: Identical to the `RIGHT JOIN` example.

---

### Real-World Cases

- **Documentation**: Using the full standard syntax for clarity.
- **Cross-database compatibility**: Explicit `OUTER` is universally understood.

---

## 5. `FULL JOIN`

### Definitions

**Core Definition**
`FULL JOIN` returns all rows from both tables, matching rows where possible and padding with `NULL` where not.

**Technical Definition**
`T1 FULL JOIN T2 ON condition` is the union of `T1 LEFT JOIN T2` and `T1 RIGHT JOIN T2`: it returns matched rows, unmatched rows from `T1` (with `NULL`s for `T2`), and unmatched rows from `T2` (with `NULL`s for `T1`). It is equivalent to `FULL OUTER JOIN` .

**Beginner-Friendly Explanation**
`FULL JOIN` says: "Keep everything from **both** tables. If there's a match, show it; if not, show `NULL`s on the missing side."

---

### Purposes

- To preserve all rows from both tables.
- To reconcile two data sets and identify differences.
- To find rows that exist in one table but not the other (from either side).
- To produce a complete picture of two related entities.

---

### Syntax Rules and Structure

```
SELECT select_list
FROM left_table
FULL JOIN right_table
  ON join_condition;
```

**Syntax Rules**

1. `FULL JOIN` is equivalent to `FULL OUTER JOIN`.
2. All rows from both tables appear in the result.
3. For unmatched rows, the columns of the non-matching side are `NULL`.
4. Rows that match appear once (not duplicated).

**Constraints and Limitations**

- **MySQL does not support `FULL JOIN`** natively; emulate with `LEFT JOIN UNION RIGHT JOIN`.
- `FULL JOIN` can be expensive because it must process both sides for unmatched rows.

---

### Annotated Code Examples

**Example 1: Basic `FULL JOIN`**

```sql
SELECT c.customer_name, o.order_id
FROM customers c
FULL JOIN orders o ON c.customer_id = o.customer_id;
```

**Expected Output (illustrative)**

| customer_name | order_id |
|---------------|----------|
| Alice         | 1001     |
| Alice         | 1003     |
| Bob           | 1002     |
| Carol         | NULL     |
| NULL          | 1004     |

**Why This Output Occurs**: Carol (customer with no orders) appears with `NULL` order; order 1004 (order with no matching customer) appears with `NULL` customer. All matches appear once .

---

### Real-World Cases

- **Data reconciliation**: Comparing two inventory systems.
- **Migration validation**: Finding records present in source but not target, and vice versa.
- **Audit**: Identifying orphaned records on both sides of a relationship.

---

## 6. `FULL OUTER JOIN`

### Definitions

**Core Definition**
`FULL OUTER JOIN` is the explicit SQL-standard syntax for `FULL JOIN`; both are identical in meaning.

**Technical Definition**
The `OUTER` keyword is optional and has no semantic effect. `FULL OUTER JOIN` is the formal name .

**Beginner-Friendly Explanation**
`FULL OUTER JOIN` is exactly the same as `FULL JOIN`.

---

### Purposes

- To write SQL-standard-conforming code.
- To make the outer-join nature explicit.

---

### Syntax Rules and Structure

```
SELECT select_list
FROM left_table
FULL OUTER JOIN right_table
  ON join_condition;
```

**Syntax Rules**

1. Semantically identical to `FULL JOIN`.
2. All `FULL JOIN` rules apply.

---

### Annotated Code Examples

**Example 1: `FULL OUTER JOIN`**

```sql
SELECT c.customer_name, o.order_id
FROM customers c
FULL OUTER JOIN orders o ON c.customer_id = o.customer_id;
```

**Expected Output**: Identical to the `FULL JOIN` example.

---

### Real-World Cases

- **Standard-compliant reconciliation queries**.
- **Cross-system comparison**.

---

## 7. NULL Padding

### Definitions

**Core Definition**
`NULL` padding is the mechanism by which outer joins fill unmatched columns with `NULL` values to preserve rows from the non-matching side.

**Technical Definition**
When a row from the preserved side has no match in the other table, the outer join produces a result row in which every column from the non-preserved table is set to `NULL`. This creates a detectable pattern: `NULL` in a non-nullable column (or in a column that should always have a value when a match exists) indicates the absence of a match. `NULL` padding is the foundation of anti-join detection .

**Beginner-Friendly Explanation**
`NULL` padding is the "blank filling" that outer joins do. If a customer has no orders, the order columns show `NULL`—meaning "no data here."

---

### Purposes (of Understanding `NULL` Padding)

- To detect unmatched rows by checking for `NULL` in the non-preserved side.
- To design reports that clearly show missing data.
- To implement anti-joins (`WHERE right.key IS NULL`).
- To avoid misinterpreting `NULL` as a real data value.

---

### Syntax Rules and Structure (Behavioural Rules)

1. `NULL` padding occurs only in outer joins, not inner joins.
2. The `NULL` is a placeholder, not a real data value; it must be handled with `IS NULL`, `COALESCE`, or `NULLIF`.
3. `NULL` padding can appear in multiple columns simultaneously.
4. `NULL` padding is not the same as a `NULL` stored in the base table; use `GROUPING` (in grouping-set contexts) or explicit source tracking to disambiguate.

**Constraints and Limitations**

- Checking `column = NULL` never works; use `IS NULL`.
- `NULL` padding can break `NOT NULL` assumptions in application code.
- Aggregates ignore `NULL`-padded rows, which may or may not be desired.

---

### Annotated Code Examples

**Example 1: Detecting `NULL` Padding**

```sql
SELECT c.customer_name, o.order_id
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

**Expected Output (illustrative)**

| customer_name | order_id |
|---------------|----------|
| Carol         | NULL     |

**Why This Output Occurs**: `o.order_id IS NULL` identifies rows where the `LEFT JOIN` found no matching order—i.e., the `NULL` is padding, not real data .

---

### Real-World Cases

- **Missing data reports**: Customers without orders, products without sales.
- **Data quality**: Identifying orphaned foreign keys.
- **Application logic**: Handling optional relationships gracefully.

---

## 8. Join Direction

### Definitions

**Core Definition**
Join direction refers to which side of the join is preserved (`LEFT`, `RIGHT`, or `FULL`) and how swapping the table order and direction produces equivalent results.

**Technical Definition**
`A LEFT JOIN B` is logically equivalent to `B RIGHT JOIN A` (with the columns in a different order). The choice of direction is a readability and maintainability decision, not a semantic one. Most style guides recommend using `LEFT JOIN` consistently and avoiding `RIGHT JOIN` to reduce cognitive load .

**Beginner-Friendly Explanation**
Join direction is about "which table are we keeping all the rows from?" You can express the same result with `LEFT JOIN` or `RIGHT JOIN`—it just depends on which table you list first.

---

### Purposes

- To choose the most readable form of a query.
- To avoid `RIGHT JOIN` by reordering tables.
- To reason about which side is preserved.
- To maintain consistency across a codebase.

---

### Syntax Rules and Structure (Equivalences)

| Original | Equivalent |
|----------|------------|
| `A LEFT JOIN B` | `B RIGHT JOIN A` |
| `A RIGHT JOIN B` | `B LEFT JOIN A` |
| `A FULL JOIN B` | `B FULL JOIN A` |

**Constraints and Limitations**

- Column order in the `SELECT` list may differ even when the row set is the same.
- Some optimisers treat `LEFT` and `RIGHT` joins differently for reordering purposes.

---

### Annotated Code Examples

**Example 1: `LEFT JOIN` vs. `RIGHT JOIN` Equivalence**

```sql
-- Form 1: LEFT JOIN
SELECT c.customer_name, o.order_id
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;

-- Form 2: RIGHT JOIN (equivalent row set)
SELECT c.customer_name, o.order_id
FROM orders o
RIGHT JOIN customers c ON o.customer_id = c.customer_id;
```

**Expected Output**: Both produce the same rows; only the written order differs.

---

### Real-World Cases

- **Code style consistency**: Standardising on `LEFT JOIN` across a team.
- **Query refactoring**: Rewriting `RIGHT JOIN` as `LEFT JOIN` for clarity.

---

## 9. Anti-Joins

### Definitions

**Core Definition**
An anti-join returns rows from one table that have **no** matching rows in another table.

**Technical Definition**
An anti-join is implemented by outer-joining the two tables and filtering for `NULL` in the non-preserved side (`WHERE right.key IS NULL`), or by using `NOT EXISTS` / `NOT IN`. The outer-join form is often called a "null anti-join"; the `NOT EXISTS` form is often more efficient and clearer in intent .

**Beginner-Friendly Explanation**
An anti-join answers "what's missing?"—like "which customers have never placed an order?"

---

### Purposes

- To find rows without a matching counterpart.
- To identify orphaned records.
- To exclude rows that have related data.
- To support data quality and reconciliation.

---

### Syntax Rules and Structure

**Outer-Join Anti-Join Pattern**
```
SELECT left.*
FROM left_table left
LEFT JOIN right_table right ON left.key = right.key
WHERE right.key IS NULL;
```
**`NOT EXISTS` Anti-Join Pattern**
```
SELECT *
FROM left_table left
WHERE NOT EXISTS (
  SELECT 1 FROM right_table right WHERE right.key = left.key
);
```

**Syntax Rules**

1. The `IS NULL` check must reference a column from the non-preserved side that is guaranteed to be non-`NULL` when a match exists (typically the join key or a `NOT NULL` column).
2. `NOT EXISTS` is generally preferred for clarity and often for performance.
3. `NOT IN` can produce unexpected results if the subquery returns `NULL`s.

**Constraints and Limitations**

- `NOT IN` with `NULL`s returns no rows; use `NOT EXISTS` instead.
- The outer-join anti-join pattern may not use indexes as efficiently as `NOT EXISTS` in some systems.

---

### Annotated Code Examples

**Example 1: Anti-Join with `LEFT JOIN ... IS NULL`**

```sql
SELECT c.customer_name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

**Expected Output (illustrative)**

| customer_name |
|---------------|
| Carol         |

**Why This Output Occurs**: Carol has no orders; the `LEFT JOIN` produces a `NULL` order row, and the `WHERE` clause keeps only those rows .

**Example 2: Anti-Join with `NOT EXISTS`**

```sql
SELECT c.customer_name
FROM customers c
WHERE NOT EXISTS (
  SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

**Expected Output**: Same as above; `NOT EXISTS` is often clearer and more efficient.

---

### Real-World Cases

- **Customer churn**: Customers with no recent orders.
- **Inventory**: Products never sold.
- **Data integrity**: Foreign keys with no matching primary key.

---

## 10. Semi-Joins

### Definitions

**Core Definition**
A semi-join returns rows from one table that **have** at least one matching row in another table, without duplicating rows or returning columns from the other table.

**Technical Definition**
A semi-join is logically `SELECT left.* FROM left WHERE EXISTS (SELECT 1 FROM right WHERE right.key = left.key)`. It differs from an inner join in that it does not multiply rows when the right table has multiple matches; it returns each left row at most once. Semi-joins are often expressed with `EXISTS` or `IN` rather than with join syntax .

**Beginner-Friendly Explanation**
A semi-join answers "which rows have a match?"—like "which customers have placed at least one order?"—without showing the order details.

---

### Purposes

- To filter rows based on the existence of related data.
- To avoid row duplication from one-to-many joins.
- To improve performance by stopping at the first match.
- To express "has at least one" logic clearly.

---

### Syntax Rules and Structure

**`EXISTS` Semi-Join**
```
SELECT *
FROM left_table left
WHERE EXISTS (
  SELECT 1 FROM right_table right WHERE right.key = left.key
);
```
**`IN` Semi-Join**
```
SELECT *
FROM left_table
WHERE key IN (SELECT key FROM right_table);
```

**Syntax Rules**

1. `EXISTS` returns `TRUE` as soon as one matching row is found; it does not duplicate left rows.
2. `IN` is equivalent for non-`NULL` keys but can behave unexpectedly with `NULL`s.
3. Semi-joins can also be expressed as `LEFT JOIN ... WHERE right.key IS NOT NULL` followed by `DISTINCT`, but `EXISTS` is generally clearer and more efficient.

**Constraints and Limitations**

- `IN` with a subquery that returns `NULL`s can produce unexpected results.
- `EXISTS` may be correlated (referencing the outer query) or uncorrelated.

---

### Annotated Code Examples

**Example 1: Semi-Join with `EXISTS`**

```sql
SELECT c.customer_name
FROM customers c
WHERE EXISTS (
  SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

**Expected Output (illustrative)**

| customer_name |
|---------------|
| Alice         |
| Bob           |

**Why This Output Occurs**: Only customers with at least one order are returned; Carol is excluded. Alice appears once even though she has two orders .

**Example 2: Semi-Join with `IN`**

```sql
SELECT customer_name
FROM customers
WHERE customer_id IN (SELECT customer_id FROM orders);
```

**Expected Output**: Same as above (assuming no `NULL` customer IDs in `orders`).

---

### Real-World Cases

- **Active customers**: Customers with at least one order.
- **Product availability**: Products with at least one unit in stock.
- **Access control**: Users with at least one assigned role.

---

## 11. Identifying Unmatched Records

### Definitions

**Core Definition**
Identifying unmatched records is the process of finding rows in one table that have no corresponding row in another table, using outer joins and `NULL` checks or `NOT EXISTS`.

**Technical Definition**
Unmatched records are rows for which the join condition evaluates to `FALSE` or `UNKNOWN` for all candidate rows in the other table. They are identified by outer-joining and filtering for `NULL` in the non-preserved side, or by using anti-join predicates (`NOT EXISTS`, `NOT IN` with care). This is a core data quality and reconciliation technique.

**Beginner-Friendly Explanation**
Identifying unmatched records means finding the "orphans"—rows that don't have a partner in the other table. For example, orders that reference a customer ID that doesn't exist in the customers table.

---

### Purposes

- To detect orphaned foreign keys.
- To reconcile data between systems.
- To find missing or incomplete relationships.
- To support data cleansing and migration validation.

---

### Syntax Rules and Structure

**Pattern 1: `LEFT JOIN ... IS NULL`**
```
SELECT left.*
FROM left_table left
LEFT JOIN right_table right ON left.key = right.key
WHERE right.key IS NULL;
```
**Pattern 2: `NOT EXISTS`**
```
SELECT *
FROM left_table left
WHERE NOT EXISTS (
  SELECT 1 FROM right_table right WHERE right.key = left.key
);
```
**Pattern 3: `FULL OUTER JOIN ... IS NULL` (both sides)**
```
SELECT *
FROM left_table left
FULL OUTER JOIN right_table right ON left.key = right.key
WHERE left.key IS NULL OR right.key IS NULL;
```

**Syntax Rules**

1. The `IS NULL` check must reference a non-nullable column on the non-preserved side.
2. For `FULL OUTER JOIN`, check both sides to find unmatched rows on either side.
3. `NOT EXISTS` is often preferred for performance and clarity.

**Constraints and Limitations**

- `NOT IN` can return no rows if the subquery contains `NULL`s.
- The outer-join anti-join pattern may not use indexes as efficiently as `NOT EXISTS` in some optimisers.

---

### Annotated Code Examples

**Example 1: Finding Orders Without Customers**

```sql
SELECT o.order_id, o.customer_id
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;
```

**Expected Output (illustrative)**

| order_id | customer_id |
|----------|-------------|
| 1004     | 999         |

**Why This Output Occurs**: Order 1004 references customer 999, which does not exist in `customers`; the `LEFT JOIN` pads the customer columns with `NULL`, and the `WHERE` clause isolates the orphan .

---

### Real-World Cases

- **Referential integrity checks**: Finding foreign keys with no matching primary key.
- **ETL reconciliation**: Comparing source and target record counts.
- **Data migration**: Identifying records that failed to transfer.

---

## 12. Preserving Rows from One Side

### Definitions

**Core Definition**
Preserving rows from one side means choosing the outer-join direction so that all rows from a designated table appear in the result, regardless of matches.

**Technical Definition**
The preserved side is determined by the join type: `LEFT JOIN` preserves the left table, `RIGHT JOIN` preserves the right table, and `FULL JOIN` preserves both. The preserved side's rows always appear; the non-preserved side contributes matches where available and `NULL`s where not. Choosing the preserved side is a semantic decision that determines which "complete list" the query represents.

**Beginner-Friendly Explanation**
Preserving rows from one side means deciding "which table's rows must I never lose?" If you must show every customer, preserve the customers table. If you must show every order, preserve the orders table.

---

### Purposes

- To ensure complete lists in reports (e.g., all products, all customers).
- To attach optional related data without dropping rows.
- To control which side drives the result set.
- To support "all X, with optional Y" query patterns.

---

### Syntax Rules and Structure

| Goal | Join Type | Preserved Side |
|------|-----------|----------------|
| All rows from `A` | `A LEFT JOIN B` | `A` (left) |
| All rows from `B` | `A RIGHT JOIN B` | `B` (right) |
| All rows from both | `A FULL JOIN B` | Both |

**Syntax Rules**

1. The preserved side is the one whose rows are never dropped.
2. Conditions on the non-preserved side in the `WHERE` clause can unintentionally filter out `NULL`-padded rows; place them in the `ON` clause if they should not eliminate preserved rows.
3. `LEFT JOIN` is the most common choice for "preserve the first table" semantics.

**Constraints and Limitations**

- Placing a filter on the non-preserved side in `WHERE` effectively converts the outer join to an inner join for those rows.
- `FULL JOIN` preserves both sides but cannot be used to "preserve only one side"; use `LEFT` or `RIGHT` for that.

---

### Annotated Code Examples

**Example 1: Preserving All Products (Even Unsold)**

```sql
SELECT p.product_name, SUM(oi.quantity) AS units_sold
FROM products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id
GROUP BY p.product_name;
```

**Expected Output (illustrative)**

| product_name | units_sold |
|--------------|------------|
| Laptop       | 50         |
| Mouse        | 200        |
| Keyboard     | NULL       |

**Why This Output Occurs**: `products` is preserved; `Keyboard` has no order items, so `SUM` returns `NULL` (use `COALESCE(SUM(...), 0)` to show `0`) .

---

**Example 2: Preserving Rows While Filtering the Non-Preserved Side Correctly**

```sql
-- Correct: filter in ON to preserve all products
SELECT p.product_name, oi.order_id
FROM products p
LEFT JOIN order_items oi
  ON p.product_id = oi.product_id
 AND oi.quantity > 10;
```

**Explanation**: The `quantity > 10` condition is in the `ON` clause, so products without large orders still appear (with `NULL` order details). If the condition were in `WHERE`, those products would be filtered out, effectively turning the outer join into an inner join.

---

### Real-World Cases

- **Product catalogues**: All products, with sales data where available.
- **Customer lists**: All customers, with order history where available.
- **Inventory reports**: All warehouses, with stock levels where available.

---

## References

1. **PostgreSQL 16 Documentation — Table Expressions (JOIN Types)** — https://www.postgresql.org/docs/16/queries-table-expressions.html
2. **PostgreSQL 16 Documentation — Joins Between Tables** — https://www.postgresql.org/docs/16/tutorial-join.html
3. **MySQL 8.0 Reference Manual — JOIN Clause** — https://dev.mysql.com/doc/refman/8.0/en/join.html
4. **Microsoft SQL Server — Joins (SQL Server)** — https://learn.microsoft.com/en-us/sql/relational-databases/performance/joins
5. **Microsoft SQL Server — FROM Clause (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/queries/from-transact-sql
6. **Oracle Database SQL Language Reference — Joins** — https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Joins.html
7. **Use The Index, Luke — Outer Joins** — https://use-the-index-luke.com/sql/join
8. **SQL:2016 Standard — ISO/IEC 9075-2:2016 (Foundation)** — https://www.iso.org/standard/63555.html

---

This cheat sheet provides a comprehensive reference for SQL outer joins, covering `LEFT`, `RIGHT`, and `FULL` joins (with and without `OUTER`), `NULL` padding, join direction equivalences, anti-joins, semi-joins, unmatched-record identification, and row-preservation strategies. Always consult your specific RDBMS documentation for version-specific behaviour, particularly MySQL's lack of native `FULL OUTER JOIN` and optimiser differences in outer-join reordering.