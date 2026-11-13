# SQL Grouping Data: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
SQL grouping is the process of partitioning a set of rows into subsets that share common values in one or more columns or expressions, so that aggregate functions can be applied independently to each subset.

**Technical Definition**
In SQL, grouping is implemented through the `GROUP BY` clause and its extensions—`ROLLUP`, `CUBE`, and `GROUPING SETS`. The `GROUP BY` clause divides the input rows into groups based on the values of specified grouping expressions; each group is then reduced to a single output row by the aggregate functions in the `SELECT` list and `HAVING` clause. Grouping by expressions, multiple columns, and grouping sets allows the computation of hierarchical subtotals, cross-tabulations, and custom combinations of grouping columns in a single query pass. The `GROUPING` function distinguishes `NULL` values that arise from super-aggregate rows from `NULL` values present in the underlying data. Performance is governed by execution strategies such as HashAggregate and GroupAggregate in PostgreSQL, Loose Index Scan and temporary tables in MySQL, and index and columnstore usage in SQL Server .

**Beginner-Friendly Explanation**
Grouping is how you tell the database to “put similar rows together.” For example, you can group sales data by country to get one total per country, or group by year and month to get monthly totals. Advanced grouping features let you get subtotals and grand totals in the same query, without writing multiple queries and combining them yourself.

---

### Key Characteristics

1. **One Output Row per Group**: Without grouping extensions, `GROUP BY` produces exactly one output row for each distinct combination of grouping expression values .
2. **Grouping by Expressions, Not Just Columns**: Grouping expressions can be arbitrary non-aggregate expressions (e.g., `GROUP BY EXTRACT(YEAR FROM sale_date)`), subject to vendor-specific restrictions.
3. **Grouping Sets Produce Super-Aggregate Rows**: `ROLLUP`, `CUBE`, and `GROUPING SETS` produce additional rows representing subtotals and grand totals. In these rows, columns that are not part of the current grouping set are reported as `NULL` .
4. **`GROUPING` Disambiguates `NULL`**: The `GROUPING(column)` function returns `1` when the `NULL` in a column is due to the column being excluded from the grouping set (a super-aggregate `NULL`), and `0` when the `NULL` is an actual data value .
5. **Vendor Divergence in Extensions**: PostgreSQL and SQL Server support the full SQL-standard `GROUPING SETS`, `ROLLUP`, and `CUBE` syntax. MySQL supports `WITH ROLLUP` (and `GROUPING SETS` in HeatWave), but not `CUBE` in the same form. The `GROUPING` function is available in all three, with slightly different usage contexts.
6. **Performance Is Strategy-Dependent**: PostgreSQL chooses between HashAggregate (hash table in memory) and GroupAggregate (sort + group), influenced by `work_mem` and index availability. MySQL can use Loose Index Scan to read only one key per group, avoiding temporary tables. SQL Server benefits from columnstore indexes and appropriate B-tree indexing .

---

### Prerequisites

- **Aggregate functions**: `SUM`, `COUNT`, `AVG`, `MIN`, `MAX`, and their `NULL`-handling semantics.
- **Basic `SELECT` syntax**: `SELECT`, `FROM`, `WHERE`, `HAVING`, `ORDER BY`.
- **Expression evaluation**: Understanding how expressions are computed per row.
- **NULL semantics**: Three-valued logic and the difference between `NULL` and “not applicable.”
- **Index fundamentals**: B-tree index structure and how it supports sorting and grouping.

---

### Related Programming Areas

| Area | Connection |
|------|------------|
| **OLAP & Data Warehousing** | `ROLLUP`, `CUBE`, and `GROUPING SETS` are the SQL foundation of multidimensional analysis. |
| **Reporting** | Subtotals and grand totals in a single query reduce application-side aggregation logic. |
| **Query Optimisation** | Index design, `work_mem`, and columnstore indexes directly affect grouping performance. |
| **Data Quality** | `GROUPING` helps distinguish missing data from super-aggregate markers. |
| **Business Intelligence** | Grouping by expressions (e.g., date parts) supports time-series and cohort analysis. |

---

### Core Concepts / Features

1. **`GROUP BY`** — the fundamental grouping clause
2. **Grouping by Columns** — grouping on one or more column references
3. **Grouping by Expressions** — grouping on computed expressions
4. **Grouping by Multiple Columns** — composite grouping keys
5. **Grouping Sets** — the general concept of multiple independent groupings
6. **`ROLLUP`** — hierarchical subtotals
7. **`CUBE`** — all combinations of grouping columns
8. **`GROUPING SETS`** — explicit list of grouping sets
9. **`GROUPING` Function** — distinguishing super-aggregate `NULL`s
10. **Grouping NULLs** — behaviour and disambiguation
11. **Grouping Performance** — execution strategies and optimisation

---

## 1. `GROUP BY`

### Definitions

**Core Definition**
`GROUP BY` is the SQL clause that partitions the input rows of a query into groups based on the values of one or more grouping expressions.

**Technical Definition**
`GROUP BY grouping_element [, ...]` divides the rows produced by the `FROM` and `WHERE` clauses into groups such that all rows in a group have equal values for every grouping expression. The `SELECT` list may then contain only the grouping expressions and aggregate function calls; referencing a non-grouped column without an aggregate is an error (with the exception of functional dependencies in some systems). The `HAVING` clause filters groups after aggregation .

**Beginner-Friendly Explanation**
`GROUP BY` is the clause that says “organise the rows into buckets that share the same value.” After grouping, you can compute a summary for each bucket.

---

### Purposes

- To partition rows into meaningful subsets for aggregation.
- To compute one result per category, region, time period, or other grouping key.
- To support `HAVING`-based filtering of aggregated results.
- To enable hierarchical and multidimensional summaries through extensions.

---

### Syntax Rules and Structure

**General Syntax**
```
SELECT select_list
FROM table_expression
[WHERE condition]
GROUP BY grouping_element [, ...]
[HAVING condition]
[ORDER BY order_expression]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `grouping_element` | A column name, an expression, or a grouping-set construct (`ROLLUP`, `CUBE`, `GROUPING SETS`). |
| `HAVING` | Filters groups after aggregation; may reference aggregate functions. |
| `ORDER BY` | Orders the final result. |

**Syntax Rules**

1. Each column in the `SELECT` list that is not inside an aggregate must appear in the `GROUP BY` clause (or be functionally dependent on a grouped column, in systems that support functional dependency detection).
2. `GROUP BY` is evaluated after `WHERE` and before `HAVING` and `ORDER BY`.
3. Grouping columns may be referenced by name, position, or expression, depending on the RDBMS.

**Constraints and Limitations**

- SQL Server disallows grouping on columns of type `text`, `ntext`, `image`, XML methods, and subqueries .
- Grouping by high-cardinality columns (e.g., primary keys) produces many small groups, increasing memory usage and reducing efficiency .

---

### Annotated Code Examples

**Example 1: Basic `GROUP BY` on a Single Column**

```sql
SELECT region, SUM(sales) AS total_sales
FROM orders
GROUP BY region;
```

**Expected Output (illustrative)**

| region | total_sales |
|--------|-------------|
| East   | 45000       |
| West   | 62000       |

**Why This Output Occurs**: Rows are partitioned by `region`; `SUM(sales)` is computed independently for each group.

---

### Real-World Cases

- **Sales reporting**: Total revenue per product category.
- **HR analytics**: Average salary per department.
- **Operations**: Count of tickets per priority level.

---

## 2. Grouping by Columns

### Definitions

**Core Definition**
Grouping by columns means using one or more column references as the grouping keys.

**Technical Definition**
When the grouping elements are simple column references, `GROUP BY column1, column2` produces one group for each distinct combination of values in `column1` and `column2`. The order of columns in the `GROUP BY` list does not affect the set of groups produced, but it can affect performance due to sort order and index usage.

**Beginner-Friendly Explanation**
Grouping by columns is the simplest form of grouping—you pick one or more columns and tell the database to group rows that have the same values in those columns.

---

### Purposes

- To summarise data by categorical attributes.
- To compute per-entity metrics.
- To prepare data for reports and dashboards.

---

### Syntax Rules and Structure

```
GROUP BY column_name [, column_name ...]
```

**Syntax Rules**

1. Column names must be unambiguous and resolvable in the `FROM` clause.
2. Columns of unsupported types (e.g., `text`/`ntext`/`image` in SQL Server) cannot be used directly .
3. Grouping by a column with many distinct values creates many groups; grouping by low-cardinality columns is generally more efficient .

---

### Annotated Code Examples

**Example 1: Grouping by a Low-Cardinality Column**

```sql
SELECT status, COUNT(*) AS order_count
FROM orders
GROUP BY status;
```

**Expected Output (illustrative)**

| status    | order_count |
|-----------|-------------|
| pending   | 120         |
| shipped   | 450         |
| cancelled | 30          |

**Why This Output Occurs**: Rows are grouped by the three distinct `status` values.

---

### Real-World Cases

- **Inventory**: Stock counts per warehouse.
- **Marketing**: Campaign performance per channel.
- **Support**: Ticket volume per severity.

---

## 3. Grouping by Expressions

### Definitions

**Core Definition**
Grouping by expressions means using computed expressions (not just column references) as grouping keys.

**Technical Definition**
Any non-aggregate expression that is valid in the `SELECT` list can generally be used in `GROUP BY`, such as function calls, arithmetic expressions, and `CASE` expressions. The expression is evaluated for each row, and rows with equal expression results are placed in the same group. SQL Server imposes restrictions: expressions cannot contain subqueries, column aliases from the `SELECT` list, or certain data types .

**Beginner-Friendly Explanation**
Grouping by expressions lets you group by something you calculate—like grouping sales by the year extracted from a date, or by a price band computed with `CASE`.

---

### Purposes

- To group by derived attributes (e.g., year, quarter, price band).
- To normalise or categorise data on the fly.
- To avoid adding computed columns to the base table.

---

### Syntax Rules and Structure

```
GROUP BY expression [, expression ...]
```

**Syntax Rules**

1. The expression must be deterministic and free of aggregate functions.
2. The expression in `GROUP BY` must match the expression in `SELECT` (or be functionally dependent on grouping columns).
3. SQL Server disallows grouping expressions that contain subqueries, column aliases, or `xml` data type methods .

**Constraints and Limitations**

- Complex expressions may prevent index usage for grouping.
- Expression evaluation per row adds CPU overhead.

---

### Annotated Code Examples

**Example 1: Grouping by a Date Expression**

```sql
SELECT EXTRACT(YEAR FROM sale_date) AS sale_year, SUM(amount) AS total
FROM sales
GROUP BY EXTRACT(YEAR FROM sale_date);
```

**Expected Output (illustrative)**

| sale_year | total   |
|-----------|---------|
| 2025      | 120000  |
| 2026      | 98000   |

**Why This Output Occurs**: The `EXTRACT` expression is evaluated per row; rows with the same year are grouped together .

---

### Real-World Cases

- **Time-series analysis**: Grouping by month or quarter.
- **Customer segmentation**: Grouping by `CASE`-based age bands.
- **Pricing**: Grouping by rounded price ranges.

---

## 4. Grouping by Multiple Columns

### Definitions

**Core Definition**
Grouping by multiple columns means using a composite key of two or more columns or expressions.

**Technical Definition**
`GROUP BY a, b` produces one group for each distinct combination of values of `a` and `b`. The order of columns does not change the resulting groups, but it can affect the execution plan and performance. PostgreSQL may reorder grouping keys internally to minimise comparison costs, while MySQL’s Loose Index Scan requires the `GROUP BY` columns to form a leftmost prefix of an index .

**Beginner-Friendly Explanation**
Grouping by multiple columns lets you break data down by more than one dimension—like total sales per country **and** per product category.

---

### Purposes

- To compute cross-tabulated summaries.
- To drill down into data across multiple dimensions.
- To prepare data for pivot tables and OLAP cubes.

---

### Syntax Rules and Structure

```
GROUP BY column_or_expression1, column_or_expression2 [, ...]
```

**Syntax Rules**

1. The number of groups is the product of the cardinalities of the grouping columns (in the worst case).
2. The order of columns can affect sort-based grouping performance in PostgreSQL .
3. In MySQL, Loose Index Scan requires the `GROUP BY` columns to be a leftmost prefix of the index .

---

### Annotated Code Examples

**Example 1: Grouping by Two Columns**

```sql
SELECT region, product_category, SUM(sales) AS total
FROM orders
GROUP BY region, product_category;
```

**Expected Output (illustrative)**

| region | product_category | total |
|--------|------------------|-------|
| East   | Electronics      | 20000 |
| East   | Furniture        | 25000 |
| West   | Electronics      | 40000 |
| West   | Furniture        | 22000 |

**Why This Output Occurs**: One group is created for each distinct `(region, product_category)` pair .

---

### Real-World Cases

- **Retail**: Sales by store and by month.
- **HR**: Headcount by department and by location.
- **Finance**: Revenue by business unit and by quarter.

---

## 5. Grouping Sets

### Definitions

**Core Definition**
A grouping set is a single specification of the columns or expressions by which to group the data.

**Technical Definition**
The concept of grouping sets generalises `GROUP BY`: the data selected by `FROM` and `WHERE` is grouped separately by each specified grouping set, aggregates are computed for each group just as for simple `GROUP BY`, and the results are returned together. An empty grouping set `()` aggregates all rows into a single group (the grand total). The `GROUPING SETS` clause, `ROLLUP`, and `CUBE` are shorthands for specifying multiple grouping sets .

**Beginner-Friendly Explanation**
A grouping set is “one way of grouping the data.” Grouping sets let you run several different groupings in a single query—like getting totals by region, totals by product, and a grand total, all at once.

---

### Purposes

- To compute multiple levels of aggregation in a single query pass.
- To avoid multiple queries combined with `UNION ALL`.
- To support OLAP-style multidimensional analysis.

---

### Syntax Rules and Structure (Conceptual)

The general form is:
```
GROUP BY GROUPING SETS ( grouping_set1, grouping_set2, ... )
```
where each `grouping_set` is a parenthesised list of columns/expressions (or `()` for the grand total).

**Syntax Rules**

1. Each grouping set is interpreted as if it were a separate `GROUP BY` clause.
2. References to columns not in a given grouping set are replaced by `NULL` in the output rows for that set .
3. When multiple grouping items are specified in a single `GROUP BY`, the final list of grouping sets is the **cross product** of the individual items. For example, `GROUP BY a, CUBE(b, c), GROUPING SETS ((d), (e))` is equivalent to a cross product of the individual grouping items .

**Constraints and Limitations**

- Grouping sets can produce many rows; the number of sets and the cardinality of the grouping columns determine the output size.
- Some RDBMS (e.g., MySQL) have limited support for the full `GROUPING SETS` syntax.

---

### Annotated Code Examples

**Example 1: Basic Grouping Sets (PostgreSQL)**

```sql
SELECT brand, size, SUM(sales) AS total
FROM items_sold
GROUP BY GROUPING SETS ((brand), (size), ());
```

**Expected Output (illustrative, based on the PostgreSQL example)**

| brand | size | total |
|-------|------|-------|
| Foo   |      | 30    |
| Bar   |      | 20    |
|       | L    | 15    |
|       | M    | 35    |
|       |      | 50    |

**Why This Output Occurs**: Three grouping sets are evaluated: `(brand)`, `(size)`, and `()`. Rows where a column is not in the current grouping set show `NULL` (or blank in this display) for that column .

---

### Real-World Cases

- **Financial reporting**: Totals by account, totals by department, and grand total.
- **Website analytics**: Page views by device, by country, and overall.

---

## 6. `ROLLUP`

### Definitions

**Core Definition**
`ROLLUP` is a shorthand for a set of grouping sets that represent a hierarchy of prefixes, from the full list of columns down to the grand total.

**Technical Definition**
`GROUP BY ROLLUP (e1, e2, e3, ...)` is equivalent to `GROUP BY GROUPING SETS ( (e1, e2, e3, ...), (e1, e2), (e1), () )`. It produces hierarchical subtotals: for `ROLLUP(a, b, c)`, the grouping sets are `(a, b, c)`, `(a, b)`, `(a)`, and `()`. The order of columns matters; `ROLLUP` moves from right to left, decreasing the number of grouping columns .

**Beginner-Friendly Explanation**
`ROLLUP` gives you subtotals that follow a hierarchy. If you roll up by year, then month, then day, you get daily totals, monthly subtotals, yearly subtotals, and a grand total—all in one result.

---

### Purposes

- To produce drill-up reports (e.g., year → quarter → month).
- To compute hierarchical subtotals without multiple queries.
- To support OLAP operations such as slice and dice.

---

### Syntax Rules and Structure

**PostgreSQL / SQL Server (ISO syntax)**
```
GROUP BY ROLLUP ( column_or_expression [, ...] )
```
**MySQL**
```
GROUP BY column_or_expression [, ...] WITH ROLLUP
```

**Syntax Rules**

1. The order of columns in `ROLLUP` determines the hierarchy; the rightmost column is the most detailed level .
2. In the output, columns that are rolled up (not part of the current grouping set) are shown as `NULL`.
3. MySQL’s `WITH ROLLUP` is placed after the `GROUP BY` column list; it does not use parentheses in the same way as the ISO syntax .

**Constraints and Limitations**

- MySQL does not support `CUBE` in the same form as PostgreSQL and SQL Server.
- The number of rows produced grows linearly with the number of columns (n + 1 grouping sets for `ROLLUP(e1, …, en)`).

---

### Annotated Code Examples

**Example 1: `ROLLUP` in SQL Server**

```sql
SELECT Region, Territory, SUM(Sales) AS TotalSales
FROM Sales
GROUP BY ROLLUP (Region, Territory);
```

**Expected Output (illustrative, based on the SQL Server example)**

| Region        | Territory        | TotalSales |
|---------------|------------------|------------|
| Canada        | Alberta          | 100        |
| Canada        | British Columbia | 500        |
| Canada        | NULL             | 600        |
| United States | Montana          | 100        |
| United States | NULL             | 100        |
| NULL          | NULL             | 700        |

**Why This Output Occurs**: Grouping sets are `(Region, Territory)`, `(Region)`, and `()`. The `Canada, NULL` row is the subtotal for Canada; the final `NULL, NULL` row is the grand total .

**Example 2: MySQL `WITH ROLLUP`**

```sql
SELECT year, SUM(profit) AS profit
FROM sales
GROUP BY year WITH ROLLUP;
```

**Expected Output (illustrative)**

| year | profit |
|------|--------|
| 2000 | 4525   |
| 2001 | 3010   |
| NULL | 7535   |

**Why This Output Occurs**: The `NULL` year row represents the grand total across all years .

---

### Real-World Cases

- **Sales hierarchy**: Country → region → city totals.
- **Time hierarchy**: Year → quarter → month totals.
- **Organisational reporting**: Division → department → team headcount.

---

## 7. `CUBE`

### Definitions

**Core Definition**
`CUBE` is a shorthand for the power set of the specified grouping columns, producing all possible combinations of groupings.

**Technical Definition**
`GROUP BY CUBE (a, b, c)` produces grouping sets for every subset of `{a, b, c}`: `(a, b, c)`, `(a, b)`, `(a, c)`, `(b, c)`, `(a)`, `(b)`, `(c)`, and `()`. For *n* columns, `CUBE` produces 2^*n* grouping sets. This supports full multidimensional cross-tabulation .

**Beginner-Friendly Explanation**
`CUBE` gives you every possible combination of subtotals. If you cube by region and product, you get totals by region-and-product, by region alone, by product alone, and the grand total.

---

### Purposes

- To perform full multidimensional analysis across all combinations of dimensions.
- To support data mining and cross-tabulation.
- To compute all possible subtotals in a single query.

---

### Syntax Rules and Structure

**PostgreSQL / SQL Server (ISO syntax)**
```
GROUP BY CUBE ( column_or_expression [, ...] )
```
**SQL Server (legacy, non-ISO)**
```
GROUP BY column_or_expression [, ...] WITH CUBE
```

**Syntax Rules**

1. The number of grouping sets is 2^*n* for *n* columns.
2. Columns not in the current grouping set are shown as `NULL`.
3. Legacy `WITH CUBE` syntax is deprecated and should be avoided in new code .

**Constraints and Limitations**

- `CUBE` can produce a very large number of rows for many columns; use judiciously.
- MySQL does not support `CUBE` in its standard `GROUP BY` syntax (HeatWave has `GROUPING SETS`) .

---

### Annotated Code Examples

**Example 1: `CUBE` in SQL Server**

```sql
SELECT Region, Territory, SUM(Sales) AS TotalSales
FROM Sales
GROUP BY CUBE (Region, Territory);
```

**Expected Output (illustrative, based on the SQL Server example)**

| Region        | Territory        | TotalSales |
|---------------|------------------|------------|
| Canada        | Alberta          | 100        |
| NULL          | Alberta          | 100        |
| Canada        | British Columbia | 500        |
| NULL          | British Columbia | 500        |
| United States | Montana          | 100        |
| NULL          | Montana          | 100        |
| NULL          | NULL             | 700        |
| Canada        | NULL             | 600        |
| United States | NULL             | 100        |

**Why This Output Occurs**: `CUBE(Region, Territory)` produces groupings for `(Region, Territory)`, `(Territory)`, `(Region)`, and `()`. The rows with `NULL` in one column represent subtotals by the other column .

---

### Real-World Cases

- **Market basket analysis**: Sales by product, by store, by both, and overall.
- **Survey analysis**: Responses by age, by gender, by both, and overall.
- **Financial consolidation**: Revenue by entity, by currency, by both, and overall.

---

## 8. `GROUPING SETS`

### Definitions

**Core Definition**
`GROUPING SETS` is the explicit SQL clause for specifying an arbitrary list of grouping sets to be computed in a single query.

**Technical Definition**
`GROUP BY GROUPING SETS ( set1, set2, ... )` computes aggregates for each specified grouping set and returns the union of the results. Each grouping set is a parenthesised list of columns/expressions; `()` represents the grand total. `GROUPING SETS` is the most flexible grouping extension and can contain `ROLLUP` and `CUBE` as nested elements .

**Beginner-Friendly Explanation**
`GROUPING SETS` lets you list exactly which groupings you want. You are not limited to a hierarchy (`ROLLUP`) or all combinations (`CUBE`); you can pick any combination of groupings you need.

---

### Purposes

- To compute an arbitrary collection of groupings in one query.
- To replace multiple `UNION ALL` queries with a single optimised pass.
- To support custom OLAP and reporting requirements.

---

### Syntax Rules and Structure

**PostgreSQL / SQL Server (ISO syntax)**
```
GROUP BY GROUPING SETS ( (column_list1), (column_list2), ... )
```
where each `column_list` can be a list of columns/expressions or `()` for the grand total.

**Syntax Rules**

1. The result is equivalent to a `UNION ALL` of subqueries with the individual grouping sets as their `GROUP BY` clauses .
2. `ROLLUP` and `CUBE` can be nested inside `GROUPING SETS` .
3. When multiple grouping items are present, the final grouping sets are the cross product of the items .

**Constraints and Limitations**

- MySQL’s standard `GROUP BY` does not support the full `GROUPING SETS` syntax; MySQL HeatWave does. MySQL’s `WITH ROLLUP` is a limited alternative.
- Very large grouping-set lists can produce many rows and stress memory.

---

### Annotated Code Examples

**Example 1: `GROUPING SETS` with Custom Combinations**

```sql
SELECT region, product, SUM(sales) AS total
FROM orders
GROUP BY GROUPING SETS ( (region, product), (region), (product), () );
```

**Expected Output (illustrative)**

| region | product  | total |
|--------|----------|-------|
| East   | Widget   | 1000  |
| East   | Gadget   | 2000  |
| East   | NULL     | 3000  |
| NULL   | Widget   | 1500  |
| NULL   | Gadget   | 2500  |
| NULL   | NULL     | 4000  |

**Why This Output Occurs**: Four grouping sets are computed: `(region, product)`, `(region)`, `(product)`, and `()`. The `NULL`s in the output mark the columns that are not part of the current grouping set .

---

### Real-World Cases

- **Custom dashboards**: Specific subtotal combinations requested by business users.
- **Financial reports**: Totals by account, by cost centre, and by both.
- **Operational reporting**: Metrics by shift, by line, and overall.

---

## 9. `GROUPING` Function

### Definitions

**Core Definition**
`GROUPING(column)` returns `1` if the specified column is **not** part of the current grouping set (i.e., the `NULL` in that column is a super-aggregate `NULL`), and `0` if the column is grouped or the `NULL` is an actual data value.

**Technical Definition**
The `GROUPING` function is used in the `SELECT` list, `HAVING`, and `ORDER BY` clauses of a query that uses `GROUP BY`, `ROLLUP`, `CUBE`, or `GROUPING SETS`. It distinguishes `NULL` values that arise because a column is excluded from a grouping set from `NULL` values present in the underlying data. In SQL Server, `GROUPING_ID` extends this by returning a bitmask for multiple columns .

**Beginner-Friendly Explanation**
When you use `ROLLUP` or `CUBE`, the subtotal rows contain `NULL`s that are not real data—they just mean “all values.” `GROUPING` tells you whether a `NULL` is a subtotal marker (`1`) or a real missing value (`0`).

---

### Purposes

- To disambiguate super-aggregate `NULL`s from data `NULL`s.
- To label subtotal and grand-total rows in reports.
- To filter out or include specific grouping levels.
- To order results by grouping level.

---

### Syntax Rules and Structure

**PostgreSQL / MySQL / SQL Server**
```
GROUPING ( column_expression ) → integer
```
**SQL Server `GROUPING_ID`**
```
GROUPING_ID ( column_expression [, ...] ) → integer
```

**Syntax Rules**

1. `GROUPING` returns `1` for super-aggregate `NULL`s and `0` otherwise.
2. In MySQL, `GROUPING()` is available for `WITH ROLLUP` queries; it can be used in the `SELECT` list, `HAVING`, and (since 8.0.12) `ORDER BY` .
3. The `column_expression` in `GROUPING_ID` must match the expression in the `GROUP BY` list .

**Constraints and Limitations**

- `GROUPING` is not a substitute for `IS NULL` when the data itself contains `NULL`s; it only identifies super-aggregate `NULL`s.
- `GROUPING_ID` is SQL Server-specific.

---

### Annotated Code Examples

**Example 1: Using `GROUPING` to Label Subtotal Rows (MySQL)**

```sql
SELECT
  year,
  country,
  SUM(profit) AS profit,
  GROUPING(year) AS grp_year,
  GROUPING(country) AS grp_country
FROM sales
GROUP BY year, country WITH ROLLUP;
```

**Expected Output (illustrative, based on MySQL documentation)**

| year | country | profit | grp_year | grp_country |
|------|---------|--------|----------|-------------|
| 2000 | Finland | 1500   | 0        | 0           |
| 2000 | NULL    | 1500   | 0        | 1           |
| NULL | NULL    | 1500   | 1        | 1           |

**Why This Output Occurs**: `grp_country = 1` marks the row where `country` is rolled up (the year subtotal); `grp_year = 1` and `grp_country = 1` mark the grand total .

---

### Real-World Cases

- **Report labelling**: Replacing `NULL` labels with “All Regions” or “Grand Total.”
- **Conditional aggregation**: Using `GROUPING` in `HAVING` to keep only subtotal rows.
- **Data quality**: Confirming that data `NULL`s are not being confused with super-aggregate `NULL`s.

---

## 10. Grouping NULLs

### Definitions

**Core Definition**
Grouping `NULL`s refers to the behaviour of `GROUP BY` when the grouping column contains `NULL` values, and the interaction of this behaviour with super-aggregate `NULL`s.

**Technical Definition**
In a standard `GROUP BY`, all rows with `NULL` in a grouping column are placed into a single group (because `NULL` is considered equal to `NULL` for grouping purposes, even though it is not equal under `=` comparison). When `ROLLUP`, `CUBE`, or `GROUPING SETS` are used, columns excluded from a grouping set are also reported as `NULL`, creating an ambiguity: a `NULL` in the output may be a real data `NULL` or a super-aggregate marker. The `GROUPING` function resolves this ambiguity .

**Beginner-Friendly Explanation**
If your data contains missing values (`NULL`), `GROUP BY` puts all those rows into one group. But `ROLLUP` and `CUBE` also use `NULL` to mean “this column is rolled up.” `GROUPING` helps you tell the difference.

---

### Purposes (of Understanding NULL Grouping)

- To correctly interpret subtotal rows in grouping-set output.
- To avoid mislabeling real `NULL` data as subtotals.
- To design reports that handle missing data gracefully.

---

### Syntax Rules and Structure (Behavioural Rules)

1. **Standard `GROUP BY`**: `NULL`s in a grouping column form their own group.
2. **`ROLLUP` / `CUBE` / `GROUPING SETS`**: Columns not in the current grouping set are output as `NULL`.
3. **`GROUPING` function**: `GROUPING(col) = 1` identifies super-aggregate `NULL`s; `0` identifies grouped columns (including real `NULL` data).
4. **`ORDER BY`**: In MySQL, `GROUPING()` can be used in `ORDER BY` to control the placement of subtotal rows .

**Constraints and Limitations**

- In PostgreSQL, there have been bug reports where a column not in the current grouping set could show its real value instead of `NULL` in sorted grouping-sets plans; this is a known edge case .
- MySQL does not allow testing super-aggregate `NULL`s in `WHERE` or join conditions because they are added late in query processing .

---

### Annotated Code Examples

**Example 1: Distinguishing Real `NULL` from Super-Aggregate `NULL`**

```sql
SELECT
  COALESCE(region, 'ALL REGIONS') AS region_label,
  SUM(sales) AS total,
  GROUPING(region) AS is_rollup
FROM sales
GROUP BY ROLLUP (region);
```

**Expected Output (illustrative)**

| region_label | total | is_rollup |
|--------------|-------|-----------|
| East         | 3000  | 0         |
| West         | 5000  | 0         |
| ALL REGIONS  | 8000  | 1         |

**Why This Output Occurs**: `GROUPING(region)` returns `1` for the grand-total row, allowing `COALESCE` to label it distinctly. If the data contained a real `NULL` region, `GROUPING` would return `0` for that row, and `COALESCE` would still label it `'ALL REGIONS'`—which is why `GROUPING` is necessary to distinguish the two cases.

---

### Real-World Cases

- **Financial reports**: Avoiding mislabeling accounts with missing codes as subtotals.
- **Survey data**: Distinguishing “not answered” from “all respondents.”
- **Data migration**: Validating that `NULL` handling matches expectations.

---

## 11. Grouping Performance

### Definitions

**Core Definition**
Grouping performance refers to the efficiency with which the database engine computes grouped results, influenced by execution strategy, indexing, memory, and data distribution.

**Technical Definition**
Grouping performance is determined by the execution plan chosen by the optimiser. PostgreSQL uses **HashAggregate** (hash table in memory, faster when the number of groups fits in `work_mem`) or **GroupAggregate** (sort followed by sequential grouping, often used when data is already sorted or memory is limited) . MySQL can use **Loose Index Scan** to read only one key per group from a B-tree index, avoiding temporary tables, provided the `GROUP BY` columns form a leftmost prefix of the index and the only aggregates are `MIN`/`MAX` . SQL Server benefits from columnstore indexes for aggregation queries and appropriate B-tree indexes on grouping columns . Key factors include the number of groups (cardinality), available memory (`work_mem` in PostgreSQL), and whether the grouping columns are indexed.

**Beginner-Friendly Explanation**
Grouping performance is about how fast the database can produce your grouped results. If the right indexes exist and the number of groups is manageable, grouping is fast. If not, the database may have to scan the whole table and sort it, which is slower.

---

### Purposes (of Optimising Grouping Performance)

- To reduce query execution time for large datasets.
- To avoid unnecessary temporary tables and disk spills.
- To enable index-based grouping and loose index scans.
- To choose grouping strategies that fit available memory.

---

### Syntax Rules and Structure (Optimisation Techniques)

1. **Index the grouping columns**: In PostgreSQL, an index on the `GROUP BY` columns can provide sorted input for `GroupAggregate`, reducing the need for a sort step . In MySQL, a B-tree index whose leftmost prefix matches the `GROUP BY` columns enables Loose Index Scan .
2. **Tune `work_mem` (PostgreSQL)**: Increase `work_mem` to allow `HashAggregate` to keep more groups in memory, reducing disk spills .
3. **Prefer low-cardinality grouping columns**: Grouping by columns with few distinct values (e.g., status, category) produces fewer groups and is generally faster than grouping by high-cardinality columns (e.g., IDs) .
4. **Use columnstore indexes (SQL Server)**: For aggregation-heavy queries with `GROUP BY`, columnstore indexes can provide significant performance benefits .
5. **Consider the number of grouping columns**: More columns and more grouping sets (especially `CUBE`) increase the number of groups and memory requirements.
6. **Use `FILTER` or `WHERE` to reduce input rows**: Reducing the number of rows before grouping lowers the aggregation workload.

**Constraints and Limitations**

- Index-based grouping is not always faster; for queries aggregating a large portion of the table, a sequential scan may be cheaper .
- Loose Index Scan conditions are strict and may not apply to all aggregate functions.
- Parallel aggregation support varies by RDBMS and aggregate function.

---

### Annotated Code Examples

**Example 1: MySQL Loose Index Scan**

```sql
-- Index: idx(c1, c2, c3)
SELECT c1, c2, MIN(c3), MAX(c3)
FROM t1
GROUP BY c1, c2;
```

**Explanation**: The `GROUP BY` columns form a leftmost prefix of the index, and the only aggregates are `MIN`/`MAX` on the next indexed column, enabling Loose Index Scan—MySQL reads only one key per group instead of scanning all rows .

**Example 2: PostgreSQL HashAggregate vs. GroupAggregate**

```sql
SET work_mem = '256MB';
SELECT customer_id, SUM(amount)
FROM transactions
GROUP BY customer_id;
```

**Explanation**: With sufficient `work_mem`, PostgreSQL can choose HashAggregate and keep all groups in memory. If the number of groups exceeds memory, it may spill to disk or switch to a sort-based strategy .

**Example 3: SQL Server Columnstore Index**

```sql
CREATE CLUSTERED COLUMNSTORE INDEX cci_sales ON Sales;
SELECT Region, SUM(Sales) AS TotalSales
FROM Sales
GROUP BY Region;
```

**Explanation**: A columnstore index can accelerate aggregation queries that use `GROUP BY`, especially on large tables .

---

### Real-World Cases

- **Data warehousing**: Pre-aggregated summary tables and materialised views for expensive grouping queries.
- **Real-time dashboards**: Indexing grouping columns to support fast OLAP-style queries.
- **Large-scale analytics**: Using columnstore indexes (SQL Server) or Loose Index Scan (MySQL) to avoid full scans.

---

## References

1. **PostgreSQL 16 Documentation — SELECT (GROUP BY Clause)** — https://www.postgresql.org/docs/16/sql-select.html#SQL-GROUPBY
2. **PostgreSQL 16 Documentation — GROUPING SETS, CUBE, and ROLLUP** — https://www.postgresql.org/docs/16/queries-table-expressions.html#QUERIES-GROUPING-SETS
3. **PostgreSQL 16 Documentation — Aggregate Functions (GROUPING)** — https://www.postgresql.org/docs/16/functions-aggregate.html
4. **MySQL 8.0 Reference Manual — GROUP BY Modifiers** — https://dev.mysql.com/doc/refman/8.0/en/group-by-modifiers.html
5. **MySQL 8.0 Reference Manual — GROUP BY Optimization** — https://dev.mysql.com/doc/refman/8.0/en/group-by-optimization.html
6. **Microsoft SQL Server — GROUP BY (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/queries/select-group-by-transact-sql
7. **Microsoft SQL Server — GROUPING_ID (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/functions/grouping-id-transact-sql
8. **Microsoft SQL Server — Columnstore Indexes Overview** — https://learn.microsoft.com/en-us/sql/relational-databases/indexes/columnstore-indexes-overview
9. **PostgreSQL Wiki — Aggregate Functions and GROUP BY** — https://wiki.postgresql.org/wiki/Aggregate_Functions
10. **SQL:2016 Standard — ISO/IEC 9075-2:2016 (Foundation)** — https://www.iso.org/standard/63555.html

---

This cheat sheet provides a comprehensive reference for SQL grouping data across PostgreSQL, MySQL, and SQL Server, covering basic grouping, grouping by expressions and multiple columns, the `GROUPING SETS`/`ROLLUP`/`CUBE` extensions, the `GROUPING` function, `NULL` handling, and performance optimisation. Always consult your specific RDBMS documentation for version-specific syntax and behaviour.