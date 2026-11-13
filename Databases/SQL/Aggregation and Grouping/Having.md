# SQL Filtering Groups: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Filtering groups is the process of restricting which aggregated groups appear in a query's result set by applying a condition to the results of aggregation, using the `HAVING` clause.

**Technical Definition**
In SQL, the `HAVING` clause specifies a search condition for a group or an aggregate. It is applied after the `GROUP BY` clause has partitioned rows into groups and after aggregate functions have been evaluated for each group. Unlike the `WHERE` clause, which filters individual rows before grouping, `HAVING` filters entire groups based on the aggregated values they produce. The condition in `HAVING` may reference grouped expressions, aggregate functions, and (in some RDBMS) SELECT-list aliases. The clause is defined in SQL:2003 and SQL:2016 and is implemented with vendor-specific extensions in PostgreSQL, MySQL, and SQL Server.

**Beginner-Friendly Explanation**
When you group data, you often only want some of the groups in your final answer. For example, you might want only product categories with total sales over $10,000, or only customers who have placed more than five orders. `HAVING` is the clause that lets you filter these groups **after** the totals and counts have been calculated.

---

### Key Characteristics

1. **Post-Aggregation Filtering**: `HAVING` is evaluated **after** `GROUP BY` and aggregate computation, whereas `WHERE` is evaluated **before** grouping. This is the fundamental distinction between the two clauses.
2. **Aggregate-Aware Conditions**: `HAVING` can reference aggregate functions such as `SUM()`, `COUNT()`, `AVG()`, `MAX()`, and `MIN()`, which `WHERE` cannot. This allows conditions such as `HAVING SUM(amount) > 1000` or `HAVING COUNT(*) >= 5`.
3. **Optional `GROUP BY`**: `HAVING` can be used without `GROUP BY`; in that case, the entire result set is treated as a single group, and `HAVING` filters that single group (potentially returning zero rows).
4. **Vendor-Specific Alias Support**: MySQL allows `HAVING` to reference SELECT-list aliases (e.g., `HAVING total > 1000`), while PostgreSQL does not allow aliases in `HAVING` (you must repeat the expression or use a subquery).
5. **Performance Sensitivity**: Applying non-aggregate conditions in `HAVING` instead of `WHERE` can significantly degrade performance, because it forces the database to aggregate rows that could have been eliminated earlier. Many optimisers implement “condition pushdown” to mitigate this, but it is not guaranteed.
6. **NULL Group Handling**: `HAVING` conditions follow SQL’s three-valued logic; a group whose condition evaluates to `UNKNOWN` (due to `NULL`) is excluded from the result.

---

### Prerequisites

- **Aggregate functions**: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` and their `NULL`-handling behaviour.
- **`GROUP BY` clause**: How grouping partitions rows into subsets.
- **SQL clause evaluation order**: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY` → `LIMIT`.
- **NULL semantics**: Three-valued logic and the difference between `NULL`, `TRUE`, and `FALSE`.
- **Basic query optimisation concepts**: Index usage, sargability, and predicate pushdown.

---

### Related Programming Areas

| Area | Connection |
|------|------------|
| **Reporting & Analytics** | `HAVING` is used to report only groups that meet business thresholds (e.g., high-value customers, underperforming regions). |
| **Data Quality** | `HAVING` can identify groups with anomalous aggregate values (e.g., duplicate counts, out-of-range sums). |
| **Query Optimisation** | The placement of conditions between `WHERE` and `HAVING` directly affects execution time and resource consumption. |
| **OLAP & Data Warehousing** | `HAVING` works with `ROLLUP`, `CUBE`, and `GROUPING SETS` to filter subtotal and grand-total rows. |
| **Application Development** | ORMs and reporting tools generate `HAVING` clauses for dashboard filters and drill-down queries. |

---

### Core Concepts / Features

1. **`HAVING`** — the clause for filtering groups
2. **`HAVING` vs. `WHERE`** — evaluation order and semantic differences
3. **Aggregate Conditions** — conditions that reference aggregate functions
4. **Multiple Conditions** — combining conditions with `AND`, `OR`, and `NOT`
5. **Multi-Condition Group Filters** — complex `HAVING` clauses with mixed aggregate and non-aggregate predicates
6. **Performance Considerations** — condition placement, pushdown, and index usage

---

## 1. `HAVING`

### Definitions

**Core Definition**
`HAVING` is the SQL clause that specifies a search condition for a group or an aggregate, eliminating groups that do not satisfy the condition.

**Technical Definition**
`HAVING boolean_expression` is evaluated after `GROUP BY` and aggregate computation. The `boolean_expression` may reference grouped columns, aggregate function results, and (in some systems) SELECT-list aliases. Each group is tested against the condition; groups for which the condition evaluates to `TRUE` are included in the result, while those evaluating to `FALSE` or `UNKNOWN` are excluded. When `GROUP BY` is omitted, the entire input is treated as a single group, and `HAVING` can filter that group.

**Beginner-Friendly Explanation**
`HAVING` is the “filter for groups.” After you have grouped your data and calculated summaries, `HAVING` lets you keep only the groups that meet your criteria.

---

### Purposes

- To restrict the result set to groups whose aggregate values meet a threshold (e.g., total sales > $10,000).
- To eliminate groups with insufficient data (e.g., fewer than 5 orders).
- To filter based on calculated metrics that are not available at the row level.
- To refine the output of `ROLLUP`, `CUBE`, or `GROUPING SETS` queries by excluding unwanted subtotal levels.

---

### Syntax Rules and Structure

**General Syntax**
```
SELECT select_list
FROM table_expression
[WHERE row_condition]
GROUP BY grouping_element [, ...]
HAVING group_condition
[ORDER BY order_expression];
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `HAVING` | Keyword introducing the group-filter condition. |
| `group_condition` | A boolean expression referencing grouped columns, aggregate functions, or both. |
| `WHERE` (optional) | Filters individual rows **before** grouping. |
| `GROUP BY` (optional) | Partitions rows into groups; if omitted, all rows form a single group. |

**Syntax Rules**

1. `HAVING` must appear **after** `GROUP BY` and **before** `ORDER BY`.
2. The condition may reference aggregate functions (`SUM`, `COUNT`, `AVG`, `MIN`, `MAX`, etc.).
3. The condition may reference grouped columns directly (e.g., `HAVING region = 'East'`), though this is often more efficiently placed in `WHERE`.
4. In standard SQL, `HAVING` cannot reference SELECT-list aliases; MySQL allows this as an extension.
5. `HAVING` can be used without `GROUP BY`; the entire result set is treated as one group.

**Constraints and Limitations**

- SQL Server disallows `text`, `ntext`, and `image` data types in `HAVING`.
- MySQL applies `HAVING` “nearly last, just before items are sent to the client, with no optimization,” meaning it does not benefit from index-based filtering on its own.
- PostgreSQL does not allow SELECT-list aliases in `HAVING`; the expression must be repeated or wrapped in a subquery.

---

### Annotated Code Examples

**Example 1: Basic `HAVING` with an Aggregate Condition (PostgreSQL)**

```sql
SELECT city, COUNT(*) AS readings, MAX(temp_lo) AS max_temp
FROM weather
GROUP BY city
HAVING MAX(temp_lo) < 40;
```

**Step-by-Step Explanation**

1. `FROM weather` reads all weather rows.
2. `GROUP BY city` partitions rows by city.
3. `COUNT(*)` and `MAX(temp_lo)` are computed for each city group.
4. `HAVING MAX(temp_lo) < 40` eliminates groups whose maximum low temperature is 40 or above.
5. Only groups satisfying the condition are returned.

**Expected Output (illustrative, based on the PostgreSQL tutorial)**

| city    | readings | max_temp |
|---------|----------|----------|
| Hayward | 1        | 37       |

**Why This Output Occurs**: Only the Hayward group has `MAX(temp_lo) < 40`; San Francisco’s maximum of 46 fails the condition and is excluded.

---

**Example 2: `HAVING` Without `GROUP BY`**

```sql
SELECT COUNT(*) AS total_rows
FROM weather
HAVING COUNT(*) > 100;
```

**Explanation**: Without `GROUP BY`, the entire table is one group. If the table has more than 100 rows, the single group is returned; otherwise, zero rows are returned.

---

**Example 3: `HAVING` with a SELECT-List Alias (MySQL Extension)**

```sql
SELECT user_id, SUM(amount) AS total_spent
FROM orders
GROUP BY user_id
HAVING total_spent > 1000;
```

**Explanation**: MySQL allows the alias `total_spent` in `HAVING`, which refers to `SUM(amount)`. This is a MySQL extension; in PostgreSQL, you would need to write `HAVING SUM(amount) > 1000`.

---

### Real-World Cases

- **High-value customers**: `HAVING SUM(order_total) > 10000` identifies customers whose lifetime value exceeds a threshold.
- **Inventory replenishment**: `HAVING SUM(quantity) < 10` finds products with critically low stock.
- **Fraud detection**: `HAVING COUNT(*) > 50 AND SUM(amount) > 100000` flags accounts with unusually high transaction volume.

---

## 2. `HAVING` vs. `WHERE`

### Definitions

**Core Definition**
`WHERE` and `HAVING` are both filtering clauses, but they operate at different stages of query processing: `WHERE` filters individual rows before grouping, while `HAVING` filters groups after aggregation.

**Technical Definition**
The `WHERE` clause is evaluated during the `FROM`/`WHERE` phase, before `GROUP BY` and aggregate computation. It determines which rows participate in the aggregation. The `HAVING` clause is evaluated after `GROUP BY` and aggregate computation; it determines which aggregated groups appear in the result. A condition that does not depend on an aggregate function can legally appear in either clause, but placing it in `WHERE` is generally more efficient because it reduces the number of rows that must be grouped and aggregated.

**Beginner-Friendly Explanation**
`WHERE` is the “bouncer at the door”—it decides which rows even get into the party (the aggregation). `HAVING` is the “filter at the exit”—it decides which groups are allowed to leave with a result.

---

### Purposes (of Distinguishing Them)

- To choose the correct clause for a given condition based on whether it references an aggregate.
- To optimise query performance by filtering rows as early as possible.
- To avoid semantic errors (e.g., using an aggregate in `WHERE`, which is illegal).
- To write portable SQL that behaves consistently across RDBMS.

---

### Syntax Rules and Structure (Comparative)

| Aspect | `WHERE` | `HAVING` |
|--------|---------|----------|
| Evaluation order | Before `GROUP BY` | After `GROUP BY` |
| Operates on | Individual rows | Groups |
| Aggregate functions allowed? | No | Yes |
| Can reference SELECT aliases? | Generally no (except in some extensions) | MySQL: yes; PostgreSQL: no |
| Performance | Generally faster (reduces input early) | Potentially slower (aggregates all rows first) |
| Typical use | Filter raw data | Filter aggregated results |

**Syntax Rules**

1. Aggregate functions are **forbidden** in `WHERE` because the `WHERE` clause determines which rows enter the aggregate computation.
2. Non-aggregate conditions on grouped columns can legally appear in `HAVING`, but should be moved to `WHERE` for efficiency.
3. MySQL’s optimiser may not push conditions from `HAVING` to `WHERE` automatically, so manual placement matters.

**Constraints and Limitations**

- `WHERE` cannot contain aggregate functions; attempting to use one raises an error.
- `HAVING` cannot be used without a `SELECT` statement and is typically used with `GROUP BY`.
- Some databases (e.g., PolarDB) implement automatic condition pushdown from `HAVING` to `WHERE`, but this is not universal.

---

### Annotated Code Examples

**Example 1: `WHERE` vs. `HAVING` — Correct Placement**

```sql
-- Efficient: city filter in WHERE (no aggregate)
SELECT city, COUNT(*), MAX(temp_lo)
FROM weather
WHERE city LIKE 'S%'
GROUP BY city
HAVING MAX(temp_lo) < 40;
```

**Explanation**: The `city LIKE 'S%'` condition does not involve an aggregate, so it belongs in `WHERE`. It eliminates non-matching rows **before** grouping, reducing the number of rows that must be grouped and aggregated. The `MAX(temp_lo) < 40` condition involves an aggregate, so it must remain in `HAVING`.

---

**Example 2: Inefficient `HAVING` Placement**

```sql
-- Less efficient: non-aggregate condition in HAVING
SELECT city, COUNT(*), MAX(temp_lo)
FROM weather
GROUP BY city
HAVING city LIKE 'S%' AND MAX(temp_lo) < 40;
```

**Explanation**: The `city LIKE 'S%'` condition is evaluated **after** grouping and aggregation, meaning all cities are grouped and aggregated even though many will be discarded. This wastes CPU and memory.

---

### Real-World Cases

- **Date-range filtering**: `WHERE sale_date >= '2026-01-01'` should always be in `WHERE`, not `HAVING`.
- **Status filtering**: `WHERE status = 'active'` filters rows before grouping.
- **Aggregate thresholds**: `HAVING SUM(amount) > 1000` must be in `HAVING`.

---

## 3. Aggregate Conditions

### Definitions

**Core Definition**
An aggregate condition is a `HAVING` condition that references one or more aggregate functions, allowing groups to be filtered based on their computed summary values.

**Technical Definition**
Aggregate conditions in `HAVING` compare the result of an aggregate function (or an expression involving aggregates) to a constant or another expression. Common forms include `SUM(column) > value`, `COUNT(*) >= n`, `AVG(column) BETWEEN a AND b`, and `MAX(column) = value`. These conditions are evaluated once per group after the aggregate has been computed.

**Beginner-Friendly Explanation**
An aggregate condition is a rule that uses a summary value—like a total, an average, or a count—to decide whether a group should be included.

---

### Purposes

- To filter groups based on total, average, count, minimum, or maximum values.
- To enforce business thresholds (e.g., minimum order count, maximum average price).
- To identify outliers and anomalies at the group level.
- To support dynamic reporting criteria.

---

### Syntax Rules and Structure

```
HAVING aggregate_function ( expression ) comparison_operator value
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `aggregate_function` | `SUM`, `COUNT`, `AVG`, `MIN`, `MAX`, or a statistical aggregate. |
| `expression` | Column or expression aggregated. |
| `comparison_operator` | `=`, `<>`, `>`, `>=`, `<`, `<=`, `BETWEEN`, `IN`, etc. |

**Syntax Rules**

1. The aggregate function must be valid for the data type of the expression.
2. `COUNT(*)` is permitted and counts all rows in the group.
3. Aggregate conditions can be combined with `AND`, `OR`, and `NOT`.
4. The same aggregate may appear multiple times in `HAVING`; it is computed once per group.

**Constraints and Limitations**

- Aggregate conditions cannot appear in `WHERE`.
- Some RDBMS may not allow certain aggregate functions (e.g., ordered-set aggregates) in `HAVING`.
- Performance depends on the number of groups and the cost of computing the aggregate.

---

### Annotated Code Examples

**Example 1: `HAVING` with `SUM`**

```sql
SELECT customer_id, SUM(amount) AS total_spent
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 5000;
```

**Expected Output (illustrative)**

| customer_id | total_spent |
|-------------|-------------|
| 101         | 7500        |
| 205         | 12000       |

**Why This Output Occurs**: Only customers whose total spending exceeds 5000 are included.

---

**Example 2: `HAVING` with `COUNT` and `AVG`**

```sql
SELECT product_category, COUNT(*) AS product_count, AVG(price) AS avg_price
FROM products
GROUP BY product_category
HAVING COUNT(*) >= 5 AND AVG(price) < 100;
```

**Expected Output (illustrative)**

| product_category | product_count | avg_price |
|------------------|---------------|-----------|
| Accessories      | 12            | 45.50     |
| Books            | 8             | 22.75     |

**Why This Output Occurs**: Groups must have at least 5 products and an average price below 100.

---

### Real-World Cases

- **Sales performance**: `HAVING SUM(revenue) >= 100000` identifies top-performing regions.
- **Customer segmentation**: `HAVING COUNT(order_id) > 10` finds frequent buyers.
- **Quality control**: `HAVING AVG(defect_rate) > 0.05` flags production lines with high defect rates.

---

## 4. Multiple Conditions

### Definitions

**Core Definition**
Multiple conditions in `HAVING` refer to the combination of two or more boolean predicates using logical operators (`AND`, `OR`, `NOT`) to form a compound group-filtering condition.

**Technical Definition**
A `HAVING` clause may contain a `search_condition` composed of multiple predicates connected by `AND`, `OR`, and `NOT`. The condition is evaluated according to SQL’s three-valued logic: `AND` requires both operands to be `TRUE`; `OR` requires at least one to be `TRUE`; `NOT` negates the operand. Parentheses can be used to control evaluation order. A group is included only if the entire compound condition evaluates to `TRUE`.

**Beginner-Friendly Explanation**
Multiple conditions let you be more precise about which groups you want—like saying “keep groups where sales are above 10,000 **and** the customer count is at least 5.”

---

### Purposes

- To express complex business rules in a single filter.
- To combine aggregate and non-aggregate criteria.
- To implement range checks with multiple boundaries.
- To create precise inclusion/exclusion logic.

---

### Syntax Rules and Structure

```
HAVING condition1 AND condition2
HAVING condition1 OR condition2
HAVING NOT condition1
HAVING (condition1 OR condition2) AND condition3
```

**Syntax Rules**

1. `AND` has higher precedence than `OR`; use parentheses to override precedence.
2. Each condition may reference aggregates, grouped columns, or both.
3. `NOT` negates the following condition.
4. Conditions are evaluated with three-valued logic; `UNKNOWN` results cause the group to be excluded unless the logic dictates otherwise.

**Constraints and Limitations**

- Excessively complex `HAVING` clauses can be difficult to optimise.
- Some RDBMS may not short-circuit evaluation of compound conditions consistently.

---

### Annotated Code Examples

**Example 1: `AND` Combination**

```sql
SELECT customer_id, SUM(amount) AS total, COUNT(*) AS orders
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 1000 AND COUNT(*) >= 5;
```

**Expected Output (illustrative)**

| customer_id | total | orders |
|-------------|-------|--------|
| 101         | 2500  | 7      |
| 305         | 4200  | 5      |

**Why This Output Occurs**: Both conditions must be `TRUE`; customers with high totals but fewer than 5 orders are excluded.

---

**Example 2: `OR` Combination**

```sql
SELECT product_id, SUM(quantity) AS total_qty
FROM sales
GROUP BY product_id
HAVING SUM(quantity) > 1000 OR COUNT(DISTINCT customer_id) > 50;
```

**Explanation**: A group is included if it has either high total quantity **or** a large distinct customer base.

---

**Example 3: `BETWEEN` with `AND`**

```sql
SELECT category, AVG(price) AS avg_price, COUNT(*) AS cnt
FROM products
GROUP BY category
HAVING AVG(price) BETWEEN 100 AND 500 AND COUNT(*) > 10;
```

**Explanation**: This finds categories with an average price between 100 and 500 **and** more than 10 products.

---

### Real-World Cases

- **Marketing**: Segments with high engagement **and** sufficient sample size.
- **Risk management**: Accounts with high transaction volume **or** unusual patterns.
- **Inventory**: Products with low stock **and** high demand.

---

## 5. Multi-Condition Group Filters

### Definitions

**Core Definition**
A multi-condition group filter is a `HAVING` clause that combines multiple predicates—potentially mixing aggregate and non-aggregate conditions, and using nested logical operators—to precisely control which groups are returned.

**Technical Definition**
Multi-condition group filters extend the concept of multiple conditions by allowing heterogeneous predicates: aggregate comparisons (`SUM(x) > y`), grouped-column comparisons (`region = 'East'`), and even subqueries or function calls, all combined into a single `HAVING` expression. The optimizer must evaluate the entire compound predicate for each group. In systems that support condition pushdown, eligible non-aggregate predicates may be moved to `WHERE` automatically, but the logical structure of the remaining `HAVING` clause still determines the final group filter.

**Beginner-Friendly Explanation**
A multi-condition group filter is a sophisticated “keep or discard” rule that can mix different kinds of checks—like “keep groups where the region is East **and** total sales exceed 10,000, **or** where the average order value is above 500.”

---

### Purposes

- To express nuanced business logic that cannot be captured by a single condition.
- To combine row-level and group-level constraints in one query.
- To implement tiered or exception-based filtering rules.
- To reduce the need for post-processing in application code.

---

### Syntax Rules and Structure

```
HAVING ( aggregate_condition1 AND grouped_column_condition )
    OR ( aggregate_condition2 AND NOT grouped_column_condition2 )
```

**Syntax Rules**

1. Parentheses should be used liberally to make the intended logic clear.
2. Mixing aggregate and non-aggregate conditions is legal, but non-aggregate conditions are candidates for pushdown to `WHERE`.
3. The overall condition must evaluate to `TRUE` for a group to be included.
4. Alias references are allowed in MySQL but not in PostgreSQL.

**Constraints and Limitations**

- Complex multi-condition filters may obscure the query’s intent; document the logic clearly.
- Optimiser pushdown may change the effective evaluation order, but not the logical result.

---

### Annotated Code Examples

**Example 1: Mixed Aggregate and Grouped-Column Conditions**

```sql
SELECT region, product_category,
       SUM(sales) AS total_sales,
       AVG(unit_price) AS avg_price
FROM orders
GROUP BY region, product_category
HAVING (region = 'East' AND SUM(sales) > 10000)
    OR (AVG(unit_price) > 500 AND COUNT(*) >= 3);
```

**Explanation**: The filter includes groups that are either (a) in the East region with total sales above 10,000, or (b) have an average unit price above 500 and at least 3 orders. The `region = 'East'` condition could be pushed to `WHERE` by an optimiser, but the logical result remains the same.

---

**Example 2: Complex `HAVING` with `NOT`**

```sql
SELECT supplier_id, COUNT(*) AS deliveries, AVG(delay_days) AS avg_delay
FROM shipments
GROUP BY supplier_id
HAVING NOT (AVG(delay_days) > 2 AND COUNT(*) > 10)
   AND SUM(quantity) > 1000;
```

**Explanation**: This excludes suppliers that have both a high average delay and many deliveries, while still requiring a minimum total quantity.

---

### Real-World Cases

- **Supplier scorecards**: Filtering suppliers based on combined quality and volume metrics.
- **Customer tiering**: Identifying customers who meet multiple criteria for VIP status.
- **Anomaly detection**: Flagging groups that violate any of several independent thresholds.

---

## 6. Performance Considerations

### Definitions

**Core Definition**
Performance considerations for group filtering refer to the factors that determine how efficiently a `HAVING` clause is executed, including condition placement, optimiser pushdown, index usage, and the cost of aggregation.

**Technical Definition**
The performance of a query with `HAVING` depends on the execution plan chosen by the optimiser. Key factors include: (1) whether non-aggregate conditions are placed in `WHERE` or `HAVING`; (2) whether the optimiser implements condition pushdown from `HAVING` to `WHERE`; (3) the number of groups produced by `GROUP BY` (cardinality); (4) the cost of computing aggregate functions; and (5) the availability of indexes that support grouping and filtering. MySQL applies `HAVING` “nearly last, just before items are sent to the client, with no optimization,” meaning `HAVING` conditions do not benefit from index-based filtering and are evaluated after aggregation. PostgreSQL distinguishes `WHERE` (pre-aggregation) from `HAVING` (post-aggregation) and recommends placing non-aggregate conditions in `WHERE` for efficiency.

**Beginner-Friendly Explanation**
Performance is about how fast your query runs. If you put a condition in the wrong place—like putting a simple row filter in `HAVING` instead of `WHERE`—the database has to do a lot of unnecessary work. Understanding where conditions are evaluated helps you write faster queries.

---

### Purposes (of Performance-Aware Filtering)

- To minimise the number of rows that must be grouped and aggregated.
- To enable index usage for filtering.
- To avoid unnecessary computation of aggregates for groups that will be discarded.
- To reduce memory and CPU consumption in large queries.

---

### Syntax Rules and Structure (Optimisation Guidelines)

1. **Place non-aggregate conditions in `WHERE`, not `HAVING`**: This reduces the input to the grouping stage. For example, `WHERE city LIKE 'S%'` is more efficient than `HAVING city LIKE 'S%'`.
2. **Keep aggregate conditions in `HAVING`**: Conditions that reference `SUM`, `COUNT`, `AVG`, `MIN`, or `MAX` must remain in `HAVING` because aggregate values are not available before grouping.
3. **Leverage condition pushdown where available**: Some databases (e.g., PolarDB for MySQL) automatically move eligible non-aggregate `HAVING` conditions to `WHERE`. This is controlled by parameters such as `loose_polar_optimizer_switch = having_cond_pushdown=ON`.
4. **Index the `GROUP BY` columns**: An index on the grouping columns can provide sorted input for aggregation and reduce sort overhead.
5. **Limit the number of groups**: High-cardinality `GROUP BY` columns produce many groups, increasing memory usage and the cost of `HAVING` evaluation.
6. **Use `WHERE` to reduce rows before joins and grouping**: Filtering early reduces the data volume for all subsequent operations.

**Constraints and Limitations**

- Not all optimisers implement `HAVING`-to-`WHERE` pushdown; MySQL’s standard optimiser applies `HAVING` last with no optimisation.
- Pushdown cannot move conditions that reference aggregate functions; those must remain in `HAVING`.
- Index usage for `HAVING` itself is generally not possible because `HAVING` operates on aggregated results, not base rows.

---

### Annotated Code Examples

**Example 1: Inefficient vs. Efficient Condition Placement**

```sql
-- Inefficient: non-aggregate filter in HAVING
SELECT region, SUM(sales) AS total
FROM orders
GROUP BY region
HAVING region = 'East' AND SUM(sales) > 10000;

-- Efficient: non-aggregate filter in WHERE
SELECT region, SUM(sales) AS total
FROM orders
WHERE region = 'East'
GROUP BY region
HAVING SUM(sales) > 10000;
```

**Explanation**: In the inefficient version, all regions are grouped and aggregated before the `region = 'East'` condition is applied. In the efficient version, only East rows enter the grouping stage, reducing the aggregation workload.

---

**Example 2: Condition Pushdown in PolarDB for MySQL**

```sql
-- Original query: MAX(c) > 12 cannot be pushed down; t1.a > 2 can be
SELECT t1.a, MAX(t1.b)
FROM t1
GROUP BY t1.a
HAVING (t1.a > 2) AND (MAX(c) > 12);

-- After pushdown by the optimiser:
SELECT t1.a, MAX(t1.b)
FROM t1
WHERE (t1.a > 2)
GROUP BY t1.a
HAVING (MAX(c) > 12);
```

**Explanation**: The condition `t1.a > 2` depends only on a `GROUP BY` column and is moved to `WHERE`. The aggregate condition `MAX(c) > 12` remains in `HAVING` because it requires the aggregate value.

---

**Example 3: Using `WHERE` to Reduce Grouping Input**

```sql
SELECT customer_id, SUM(amount) AS total
FROM orders
WHERE order_date >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY customer_id
HAVING SUM(amount) > 500;
```

**Explanation**: The date filter in `WHERE` restricts the aggregation to the last 30 days, reducing the number of rows that must be grouped and the number of groups that must be evaluated by `HAVING`.

---

### Real-World Cases

- **Large-scale reporting**: Moving date and status filters to `WHERE` reduces the volume of data aggregated, often cutting query time from minutes to seconds.
- **Data warehouse ETL**: Using `WHERE` to filter out irrelevant rows before grouping reduces memory pressure in HashAggregate operations.
- **Interactive dashboards**: Ensuring that non-aggregate filters are pushed down allows the database to use indexes, improving response time.

---

## References

1. **PostgreSQL 16 Documentation — Aggregate Functions (HAVING)** — https://www.postgresql.org/docs/16/tutorial-agg.html
2. **PostgreSQL 16 Documentation — SELECT (HAVING Clause)** — https://www.postgresql.org/docs/16/sql-select.html#SQL-HAVING
3. **PostgreSQL 16 Documentation — Aggregate Expressions** — https://www.postgresql.org/docs/16/sql-expressions.html#SYNTAX-AGGREGATES
4. **MySQL 8.0 Reference Manual — SELECT Statement (HAVING Clause)** — https://dev.mysql.com/doc/refman/8.0/en/select.html
5. **MySQL 8.0 Reference Manual — GROUP BY Modifiers and HAVING** — https://dev.mysql.com/doc/refman/8.0/en/group-by-modifiers.html
6. **Microsoft SQL Server — HAVING (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/queries/select-having-transact-sql
7. **Microsoft SQL Server — SELECT (Transact-SQL) — HAVING Clause** — https://learn.microsoft.com/en-us/sql/t-sql/queries/select-transact-sql
8. **Alibaba Cloud PolarDB — Condition Pushdown (HAVING to WHERE)** — https://help.aliyun.com/en/polardb/polardb-for-mysql/user-guide/condition-pushdown
9. **SQL:2016 Standard — ISO/IEC 9075-2:2016 (Foundation)** — https://www.iso.org/standard/63555.html

---

This cheat sheet provides a comprehensive reference for filtering groups in SQL using the `HAVING` clause, covering its syntax, the critical distinction from `WHERE`, aggregate and multi-condition filters, and performance optimisation strategies across PostgreSQL, MySQL, and SQL Server. Always consult your specific RDBMS documentation for version-specific behaviour and optimiser features.