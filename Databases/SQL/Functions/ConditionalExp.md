# Comprehensive Programming Cheat Sheet: SQL Conditional Expressions

## Topic Overview

### Definitions

**Core Definition**
Conditional expressions are SQL constructs—`CASE`, `COALESCE`, `NULLIF`, `GREATEST`, and `LEAST`—that evaluate conditions and return different values based on whether those conditions are true, false, or unknown, allowing query results to adapt to data-dependent logic without leaving the database.

**Technical Definition**
Conditional expressions are scalar expressions that return a single value per row based on evaluated predicates. The `CASE` expression is the most general form, available in **simple** and **searched** variants, and can appear anywhere a scalar expression is valid: the `SELECT` list, `WHERE` clause, `ORDER BY`, `GROUP BY`, `HAVING`, inside aggregates, and even join conditions. `COALESCE` returns the first non-null argument; `NULLIF` returns null if two arguments are equal and the first argument otherwise; `GREATEST` and `LEAST` return the largest and smallest of a list of expressions, respectively. PostgreSQL notes that although `COALESCE`, `GREATEST`, and `LEAST` are syntactically similar to functions, they are not ordinary functions and cannot be used with explicit `VARIADIC` array arguments. `CASE` evaluates only the subexpressions needed to determine the result, which can be used to avoid runtime errors such as division by zero.

**Beginner-Friendly Explanation**
Conditional expressions are SQL's version of "if-then-else" logic. Just as a spreadsheet can show one value when a condition is met and another when it is not, SQL can transform codes into readable labels, bucket values into ranges, substitute defaults for missing data, and prevent division by zero—all inside a single query. The `CASE` expression is the Swiss Army knife: it lets you write an inline decision tree. The helper functions `COALESCE` and `NULLIF` handle null values elegantly, and `GREATEST`/`LEAST` pick extremes from a list. Mastering these constructs is the difference between exporting data to a spreadsheet for manual fixing and solving the problem directly in SQL.

### Key Characteristics

- **`CASE` is an expression, not a statement**: It returns a single value and can be used anywhere a scalar expression is allowed, including `SELECT`, `WHERE`, `ORDER BY`, `GROUP BY`, `HAVING`, inside aggregates, and join conditions.
- **Two forms of `CASE`**: The **simple** form compares one expression to a list of candidate values (equality only); the **searched** form evaluates independent Boolean predicates, enabling ranges, pattern matching, and complex conditions.
- **Short-circuit evaluation**: `CASE` does not evaluate subexpressions that are not needed to determine the result, which can be used to guard against errors such as division by zero.
- **Result type compatibility**: All branches of a `CASE` expression must produce values convertible to a single common output type.
- **Null semantics are critical**: `WHEN column = NULL` never matches because `NULL = NULL` yields `UNKNOWN`; use `WHEN column IS NULL` instead. `COALESCE` and `NULLIF` are the primary tools for null-safe logic.
- **Vendor divergence in `GREATEST`/`LEAST`**: PostgreSQL ignores `NULL` arguments in `GREATEST` and `LEAST`, returning `NULL` only if all arguments are `NULL`; some other databases return `NULL` if any argument is `NULL`.

### Prerequisites

- A running RDBMS (PostgreSQL, MySQL, or SQL Server) and a client connection.
- Familiarity with `SELECT` and basic SQL expressions.
- Understanding of SQL's three-valued logic and the special role of `NULL`.
- Awareness of the target RDBMS's support for `GREATEST`/`LEAST` (not available in SQL Server before 2022, never in Access).

### Related Programming Areas

- Data Manipulation Language (DML) – conditional expressions appear in `SELECT`, `UPDATE`, and `DELETE` statements.
- Query optimisation – `CASE` in `WHERE` is generally **not sargable** and can prevent index usage.
- Reporting and analytics – conditional aggregation (`SUM(CASE WHEN … THEN … END)`) computes multiple filtered metrics in one table scan.
- Data cleaning – `COALESCE` and `NULLIF` normalise missing and placeholder values.

---

## Core Concepts and Features

### 1. `CASE`

#### Definitions

**Core Definition**
`CASE` is a conditional expression that evaluates a list of conditions in order and returns the result associated with the first condition that evaluates to true.

**Technical Definition**
The SQL `CASE` expression is a generic conditional expression, similar to if/else statements in other programming languages. It has two syntactic forms. In the **searched** form, each `WHEN` clause contains an independent Boolean condition; the first condition that is true determines the result, and subsequent `WHEN` clauses are not evaluated. In the **simple** form, a single expression is computed and compared for equality against each `WHEN` candidate value. If no `WHEN` matches, the `ELSE` result is returned; if `ELSE` is omitted and no condition is true, the result is `NULL`. All result expressions must be convertible to a single output type.

**Beginner-Friendly Explanation**
`CASE` is SQL's decision-maker. You give it a series of "when this is true, return that" rules, and it returns the result for the first rule that matches. If nothing matches, it returns the `ELSE` value—or `NULL` if you did not provide one.

#### Purposes

- To translate codes into human-readable labels (e.g., `'S'` → `'Shipped'`) without leaving the query.
- To bucket numeric values into ranges (e.g., price categories, salary bands).
- To guard against runtime errors such as division by zero by conditionally evaluating only safe expressions.
- To implement custom sort orders in `ORDER BY` and conditional logic in `WHERE` and `GROUP BY`.

#### Syntax Rules and Structure

**Searched `CASE` syntax:**

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    [ ELSE default_result ]
END
```

**Simple `CASE` syntax:**

```sql
CASE expression
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ...
    [ ELSE default_result ]
END
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `CASE` | Begins the expression. |
| `WHEN condition THEN result` | A branch; the first true condition determines the result. |
| `WHEN value THEN result` | Simple-form branch; equality comparison against the `CASE` expression. |
| `ELSE default_result` | Optional fallback; returns `NULL` if omitted and no branch matches. |
| `END` | Terminates the expression. |

**Syntax rules:**

- `CASE` is evaluated **top-down, first match wins**; the remaining branches are not processed.
- Conditions in searched `CASE` are Boolean expressions; simple `CASE` performs equality comparisons only.
- All `THEN`/`ELSE` result expressions must be type-compatible.

**Constraints and limitations:**

- `CASE` is not a statement; it cannot be used for control flow outside expressions.
- `CASE` in `WHERE` is generally not sargable and may prevent index usage.
- `CASE` does not evaluate unnecessary subexpressions at runtime, but constant expressions may still be evaluated at planning time.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Searched CASE for value bucketing

```sql
-- Step 1: Classify employees into salary bands.
SELECT
    emp_name,
    salary,
    CASE
        WHEN salary >= 100000 THEN 'Executive'
        WHEN salary >= 70000  THEN 'Senior'
        WHEN salary >= 50000  THEN 'Mid-level'
        ELSE 'Entry'
    END AS salary_band
FROM employees;
```

**Expected output:**

```text
 emp_name | salary | salary_band
----------+--------+-------------
 Ammar    |  80000 | Senior
 Riya     |  65000 | Mid-level
 Sahil    |  72000 | Senior
 Priya    |  90000 | Senior
(4 rows)
```

**Why this output appears**: The searched `CASE` evaluates each condition in order. Ammar, Sahil, and Priya have salaries ≥ 70,000, so they match the second branch. Riya’s salary is 65,000, which fails the first two branches but matches the third.

##### Example 2: Simple CASE for code translation

```sql
-- Step 1: Translate numeric department codes into names.
SELECT
    emp_name,
    dept_id,
    CASE dept_id
        WHEN 1 THEN 'Engineering'
        WHEN 2 THEN 'Marketing'
        WHEN 3 THEN 'Operations'
        ELSE 'Unknown'
    END AS department
FROM employees;
```

**Expected output:**

```text
 emp_name | dept_id | department
----------+---------+-------------
 Ammar    |       1 | Engineering
 Riya     |       2 | Marketing
 Sahil    |       1 | Engineering
 Priya    |       3 | Operations
(4 rows)
```

**Why this output appears**: The simple `CASE` compares `dept_id` for equality against each `WHEN` value. Each employee’s department code matches exactly one branch, producing the corresponding name.

#### Real-World Cases

- **Order status labels**: Mapping numeric status codes to display labels (`'S'` → `'Shipped'`) for dashboards.
- **Age segmentation**: Classifying customers into age bands (`'18-25'`, `'26-35'`, etc.) for marketing analysis.
- **Data quality flags**: Returning `'Missing'`, `'Invalid'`, or `'OK'` based on `CASE` conditions on multiple columns.

---

### 2. Simple `CASE`

#### Definitions

**Core Definition**
Simple `CASE` compares a single expression for equality against a list of candidate values and returns the result of the first match.

**Technical Definition**
The simple form is `CASE expression WHEN value THEN result ... END`. The `expression` is computed once and compared to each `WHEN` value until an equality match is found. This is similar to a switch statement in C. It supports only equality comparisons, not ranges or complex predicates.

**Beginner-Friendly Explanation**
Simple `CASE` is a lookup table: you give it a value, and it checks whether that value equals each candidate. It is compact and readable for direct mappings.

#### Purposes

- To map code values (status codes, category IDs, country codes) to descriptive labels.
- To reduce repetitive `OR` chains in expressions.

#### Syntax Rules and Structure

```sql
CASE expression
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ...
    [ ELSE default_result ]
END
```

**Syntax rules:**

- The `expression` is evaluated once; each `WHEN` value is compared for equality.
- Only equality matching is supported; use searched `CASE` for ranges or complex predicates.
- `NULL` as a `WHEN` value will never match because `NULL = NULL` is `UNKNOWN`.

**Constraints and limitations:**

- Not suitable for inequalities, `BETWEEN`, `LIKE`, or multi-column conditions.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Simple CASE mapping

```sql
-- Step 1: Map numeric month numbers to month abbreviations.
SELECT
    month_num,
    CASE month_num
        WHEN 1 THEN 'Jan'
        WHEN 2 THEN 'Feb'
        WHEN 3 THEN 'Mar'
        ELSE 'Other'
    END AS month_abbr
FROM monthly_sales;
```

**Expected output:**

```text
 month_num | month_abbr
-----------+------------
         1 | Jan
         2 | Feb
         3 | Mar
         4 | Other
(4 rows)
```

**Why this output appears**: The simple `CASE` performs equality matching against each `WHEN` value, producing the corresponding abbreviation; unmatched values fall through to `'Other'`.

---

### 3. Searched `CASE`

#### Definitions

**Core Definition**
Searched `CASE` evaluates independent Boolean predicates in order and returns the result of the first predicate that evaluates to true.

**Technical Definition**
The searched form is `CASE WHEN condition THEN result ... END`. Each `WHEN` contains its own Boolean expression, allowing ranges, pattern matches, null checks, and combinations of conditions. Conditions are evaluated top-down, and evaluation stops at the first true condition.

**Beginner-Friendly Explanation**
Searched `CASE` is the general-purpose decision tree. Each branch can ask its own question, so you can use ranges, `LIKE`, `IS NULL`, and any other Boolean logic.

#### Purposes

- To bucket values into ranges (salary bands, price tiers).
- To combine multiple conditions in a single branch.
- To implement conditional logic that requires inequalities or pattern matching.

#### Syntax Rules and Structure

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    [ ELSE default_result ]
END
```

**Syntax rules:**

- Conditions must be Boolean expressions.
- Order branches from most specific to most general to avoid masking.
- Use `IS NULL` rather than `= NULL` for null checks.

**Constraints and limitations:**

- Overly complex searched `CASE` can be hard to optimise and maintain.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Searched CASE with range and null check

```sql
-- Step 1: Categorise orders by amount and flag missing amounts.
SELECT
    order_id,
    amount,
    CASE
        WHEN amount IS NULL THEN 'Missing'
        WHEN amount >= 1000 THEN 'Large'
        WHEN amount >= 100  THEN 'Medium'
        ELSE 'Small'
    END AS order_size
FROM orders;
```

**Expected output:**

```text
 order_id | amount | order_size
----------+--------+------------
        1 |   1500 | Large
        2 |    250 | Medium
        3 |     50 | Small
        4 |        | Missing
(4 rows)
```

**Why this output appears**: The `IS NULL` check is placed first so that missing amounts are caught before numeric comparisons. The remaining branches bucket non-null amounts into size categories.

---

### 4. Nested `CASE`

#### Definitions

**Core Definition**
Nested `CASE` is a `CASE` expression used inside the `THEN` or `ELSE` branch of another `CASE` expression, enabling multi-level decision logic.

**Technical Definition**
A nested `CASE` appears as the result expression of an outer `CASE` branch. This allows hierarchical classification where the first level determines a category and the second level further refines it. Nested `CASE` can be used to work around limitations of simple `CASE` when multiple conditions must be evaluated in sequence.

**Beginner-Friendly Explanation**
Nested `CASE` is a decision tree inside a decision tree. It is useful when you need to ask a follow-up question after the first answer is determined.

#### Purposes

- To implement hierarchical categorisation (e.g., department → seniority band).
- To handle complex business rules that require sequential evaluation.
- To simulate `IF … ELSE IF … ELSE` chains with multiple levels.

#### Syntax Rules and Structure

```sql
CASE
    WHEN outer_condition THEN
        CASE
            WHEN inner_condition THEN inner_result
            ELSE inner_default
        END
    ELSE outer_default
END
```

**Syntax rules:**

- Each nested `CASE` must be a complete expression with its own `END`.
- Readability degrades quickly; consider extracting logic into a CTE or subquery if nesting exceeds two levels.

**Constraints and limitations:**

- Deeply nested `CASE` is hard to read, debug, and optimise.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Nested CASE for department and seniority

```sql
-- Step 1: Classify employees by department and seniority.
SELECT
    emp_name,
    dept_id,
    salary,
    CASE dept_id
        WHEN 1 THEN
            CASE
                WHEN salary >= 80000 THEN 'Senior Engineer'
                ELSE 'Engineer'
            END
        WHEN 2 THEN 'Marketing'
        ELSE 'Other'
    END AS classification
FROM employees;
```

**Expected output:**

```text
 emp_name | dept_id | salary | classification
----------+---------+--------+-----------------
 Ammar    |       1 |  80000 | Senior Engineer
 Sahil    |       1 |  72000 | Engineer
 Riya     |       2 |  65000 | Marketing
 Priya    |       3 |  90000 | Other
(4 rows)
```

**Why this output appears**: The outer simple `CASE` checks `dept_id`. For department 1, the inner searched `CASE` refines by salary. For department 2, the result is `'Marketing'`; all other departments fall through to `'Other'`.

---

### 5. `CASE` in `SELECT`

#### Definitions

**Core Definition**
Using `CASE` in the `SELECT` list transforms or classifies values in the output without modifying the underlying data.

**Technical Definition**
A `CASE` expression in the `SELECT` list becomes an output column. It can be aliased with `AS` for readability and can reference any column from the `FROM` clause. This is the most common use of `CASE`—translating codes, bucketing values, and deriving labels.

**Beginner-Friendly Explanation**
Putting `CASE` in `SELECT` means "show a computed label or category alongside the raw data."

#### Purposes

- To produce human-readable labels in query results.
- To derive categorical columns for reporting.
- To compute conditional metrics inline.

#### Syntax Rules and Structure

```sql
SELECT
    column1,
    CASE WHEN condition THEN result ELSE default END AS alias
FROM table;
```

**Syntax rules:**

- Always alias the `CASE` output for clarity.
- The `CASE` expression can reference any column in the row.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: CASE in SELECT for status labels

```sql
SELECT
    order_id,
    CASE status
        WHEN 'P' THEN 'Pending'
        WHEN 'S' THEN 'Shipped'
        WHEN 'C' THEN 'Cancelled'
        ELSE 'Unknown'
    END AS status_label
FROM orders;
```

**Expected output:**

```text
 order_id | status_label
----------+--------------
        1 | Shipped
        2 | Pending
        3 | Cancelled
(3 rows)
```

**Why this output appears**: The simple `CASE` maps each status code to a descriptive label, producing a new column alongside `order_id`.

---

### 6. `CASE` in `WHERE`

#### Definitions

**Core Definition**
Using `CASE` in the `WHERE` clause builds a condition dynamically, allowing the filter to adapt to data or parameters.

**Technical Definition**
`CASE` can appear anywhere a scalar expression is valid, including the `WHERE` clause. A common pattern is to use `CASE` to choose between comparison values or to conditionally apply a filter. However, `CASE` in `WHERE` is generally **not sargable**: it prevents the query optimiser from using an index on the columns involved, often causing full table scans.

**Beginner-Friendly Explanation**
`CASE` in `WHERE` is powerful but can be slow. It is useful for dynamic filtering, but prefer plain Boolean logic when possible.

#### Purposes

- To implement parameter-driven filters (e.g., "if the user is an admin, show all rows; otherwise, show only their own").
- To avoid division by zero by conditionally evaluating a comparison.

#### Syntax Rules and Structure

```sql
WHERE CASE WHEN condition THEN boolean_expression ELSE boolean_expression END;
```

**Syntax rules:**

- The `CASE` must produce a Boolean result when used directly in `WHERE`.
- Guard against division by zero: `WHERE CASE WHEN x <> 0 THEN y/x > 1.5 ELSE false END`.

**Constraints and limitations:**

- `CASE` in `WHERE` is typically not sargable and may force a full scan.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: CASE in WHERE to avoid division by zero

```sql
-- Step 1: Safely compare a ratio without dividing by zero.
SELECT emp_name, salary, bonus
FROM employees
WHERE CASE WHEN salary <> 0 THEN bonus / salary > 0.10 ELSE false END;
```

**Expected output:**

```text
 emp_name | salary | bonus
----------+--------+-------
 Ammar    |  80000 | 10000
(1 row)
```

**Why this output appears**: The `CASE` expression checks `salary <> 0` before performing the division. If `salary` is zero, the `ELSE false` branch prevents the division-by-zero error.

---

### 7. `CASE` in `ORDER BY`

#### Definitions

**Core Definition**
Using `CASE` in `ORDER BY` defines a custom sort order that is not based solely on column values.

**Technical Definition**
`CASE` can appear in the `ORDER BY` clause to enforce a custom priority sequence. For example, you can sort priority buckets (`'Critical'`, `'High'`, `'Medium'`, `'Low'`) in that order rather than alphabetically. The expression computes a sort key for each row.

**Beginner-Friendly Explanation**
`CASE` in `ORDER BY` lets you sort things in the order you care about, not just A-to-Z.

#### Purposes

- To sort statuses or priorities in business-logical order.
- To place nulls or special values at a specific position.

#### Syntax Rules and Structure

```sql
ORDER BY CASE WHEN condition THEN sort_key ELSE other_key END;
```

**Syntax rules:**

- The `CASE` must produce a sortable value.
- Multiple sort keys can be combined with commas.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Custom priority sort

```sql
SELECT ticket_id, priority
FROM tickets
ORDER BY CASE priority
    WHEN 'Critical' THEN 1
    WHEN 'High' THEN 2
    WHEN 'Medium' THEN 3
    WHEN 'Low' THEN 4
    ELSE 5
END;
```

**Expected output:**

```text
 ticket_id | priority
-----------+----------
         1 | Critical
         2 | High
         3 | Medium
         4 | Low
(4 rows)
```

**Why this output appears**: The `CASE` assigns a numeric sort key to each priority label, and `ORDER BY` sorts by that key rather than alphabetically.

---

### 8. `CASE` in `GROUP BY`

#### Definitions

**Core Definition**
Using `CASE` in `GROUP BY` groups rows by a derived category rather than a raw column value.

**Technical Definition**
`CASE` can be used in `GROUP BY` to categorise data before aggregation. This is useful when you want to group by a custom bucket (e.g., price range, salary band) rather than the exact value. PostgreSQL allows grouping by expressions, including `CASE`. However, using `CASE` in `GROUP BY` should be done with caution because it can complicate the query plan and reduce readability.

**Beginner-Friendly Explanation**
`CASE` in `GROUP BY` means "group by category, not by exact value."

#### Purposes

- To aggregate by custom buckets (price tiers, age bands).
- To normalise values before grouping.

#### Syntax Rules and Structure

```sql
GROUP BY CASE WHEN condition THEN category ELSE other_category END;
```

**Syntax rules:**

- The `CASE` expression in `GROUP BY` must match the `CASE` expression in the `SELECT` list.
- Alias references are not allowed in `GROUP BY` in standard SQL; repeat the expression.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Grouping by salary band

```sql
SELECT
    CASE
        WHEN salary >= 80000 THEN 'High'
        WHEN salary >= 60000 THEN 'Mid'
        ELSE 'Low'
    END AS salary_band,
    COUNT(*) AS headcount
FROM employees
GROUP BY
    CASE
        WHEN salary >= 80000 THEN 'High'
        WHEN salary >= 60000 THEN 'Mid'
        ELSE 'Low'
    END;
```

**Expected output:**

```text
 salary_band | headcount
-------------+-----------
 High        |         2
 Mid         |         1
 Low         |         1
(3 rows)
```

**Why this output appears**: The `CASE` expression converts exact salaries into three bands, and `GROUP BY` uses those bands to create one row per category.

---

### 9. `CASE` in Aggregates (Conditional Aggregation)

#### Definitions

**Core Definition**
Conditional aggregation embeds `CASE` expressions inside aggregate functions such as `SUM`, `COUNT`, or `AVG` to compute multiple filtered metrics in a single table scan.

**Technical Definition**
A `CASE` inside an aggregate returns a value (or `NULL`) for each row; the aggregate then processes only the non-null results. For example, `SUM(CASE WHEN status = 'delivered' THEN amount ELSE 0 END)` sums only delivered amounts. When the `ELSE` clause is omitted, non-matching rows return `NULL`, which aggregates ignore—so `COUNT(CASE WHEN condition THEN 1 END)` counts only matching rows. This technique is the ANSI/ISO SQL standard way to perform conditional aggregation.

**Beginner-Friendly Explanation**
Conditional aggregation lets you compute several different totals in one query. Instead of running separate queries for "delivered revenue" and "cancelled orders," you embed a `CASE` inside `SUM` or `COUNT` for each metric.

#### Purposes

- To compute multiple filtered metrics in a single table scan, improving performance.
- To pivot data without a `PIVOT` operator.
- To count occurrences of specific conditions alongside overall counts.

#### Syntax Rules and Structure

```sql
SUM(CASE WHEN condition THEN value ELSE 0 END) AS metric_name
COUNT(CASE WHEN condition THEN 1 END) AS count_name
```

**Syntax rules:**

- Use `ELSE 0` for `SUM` to avoid nulls; use no `ELSE` for `COUNT` so non-matching rows return `NULL` and are ignored.
- Multiple conditional aggregates can appear in the same `SELECT` list.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Conditional aggregation for revenue metrics

```sql
SELECT
    SUM(CASE WHEN status = 'delivered' THEN amount ELSE 0 END) AS delivered_revenue,
    SUM(CASE WHEN status = 'cancelled' THEN amount ELSE 0 END) AS cancelled_revenue,
    COUNT(CASE WHEN status = 'delivered' THEN 1 END) AS delivered_count
FROM orders;
```

**Expected output:**

```text
 delivered_revenue | cancelled_revenue | delivered_count
-------------------+-------------------+-----------------
             15000 |              2000 |               3
(1 row)
```

**Why this output appears**: Each `CASE` returns the amount only for the matching status and zero (or `NULL` for `COUNT`) otherwise. The aggregates then sum or count only the relevant rows.

---

### 10. `COALESCE`

#### Definitions

**Core Definition**
`COALESCE` returns the first non-null value from a list of arguments, returning `NULL` only if all arguments are null.

**Technical Definition**
`COALESCE(value [, ...])` evaluates its arguments from left to right and returns the first argument that is not null. It is often used to substitute a default value for nulls when data is retrieved for display. All arguments must be convertible to a common data type, which becomes the type of the result. `COALESCE` is a SQL-standard feature included in ISO/IEC 9075.

**Beginner-Friendly Explanation**
`COALESCE` is the "first non-null wins" function. It is like a chain of fallbacks: if the first value is missing, try the second; if that is missing, try the third; and so on.

#### Purposes

- To provide default values for null columns.
- To choose the first available contact method (phone, then email, then a placeholder).
- To combine with `NULLIF` to handle placeholder values such as empty strings.

#### Syntax Rules and Structure

```sql
COALESCE(value1 [, value2, ...])
```

**Syntax rules:**

- Arguments are evaluated left to right; the first non-null is returned.
- All arguments must be type-compatible.
- `COALESCE` is not an ordinary function and cannot be used with `VARIADIC` arrays.

**Constraints and limitations:**

- `COALESCE` is short-circuiting: it does not evaluate arguments after the first non-null one.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: COALESCE for default contact information

```sql
SELECT
    customer_id,
    COALESCE(phone_number, email, 'No contact info') AS contact_info
FROM customers;
```

**Expected output:**

```text
 customer_id |    contact_info
-------------+--------------------
           1 | 555-123-4567
           2 | jane@example.com
           3 | No contact info
(3 rows)
```

**Why this output appears**: For each customer, `COALESCE` returns the phone number if present; otherwise the email; otherwise the literal placeholder. The first non-null argument wins.

---

### 11. `NULLIF`

#### Definitions

**Core Definition**
`NULLIF` returns `NULL` if two arguments are equal, and the first argument otherwise.

**Technical Definition**
`NULLIF(value1, value2)` compares the two expressions; if they are equal, it returns `NULL`; otherwise it returns `value1`. It is the inverse of `COALESCE` in the sense that it converts a specific value (often a placeholder like `''` or `0`) into `NULL`. `NULLIF` is a SQL-standard feature included in ISO/IEC 9075. It is equivalent to `CASE WHEN value1 = value2 THEN NULL ELSE value1 END`.

**Beginner-Friendly Explanation**
`NULLIF` says: "if this value equals that value, treat it as missing; otherwise keep it." It is useful for converting sentinel values into proper nulls before applying `COALESCE` or arithmetic.

#### Purposes

- To prevent division by zero: `value / NULLIF(divisor, 0)` returns `NULL` instead of raising an error when the divisor is zero.
- To convert empty strings or placeholder values into `NULL`.
- To combine with `COALESCE` to handle both sentinel values and nulls: `COALESCE(NULLIF(col, ''), 'N/A')`.

#### Syntax Rules and Structure

```sql
NULLIF(value1, value2)
```

**Syntax rules:**

- Returns `NULL` if `value1 = value2`; otherwise returns `value1`.
- Equivalent to `CASE WHEN value1 = value2 THEN NULL ELSE value1 END`.
- All major RDBMSs support `NULLIF` with identical syntax.

**Constraints and limitations:**

- If both arguments are `NULL`, `NULLIF` returns `NULL` (because `NULL = NULL` is `UNKNOWN`, the equality is not considered true).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: NULLIF to prevent division by zero

```sql
SELECT
    total_sales,
    total_orders,
    total_sales / NULLIF(total_orders, 0) AS avg_order_value
FROM sales_summary;
```

**Expected output:**

```text
 total_sales | total_orders | avg_order_value
-------------+--------------+-----------------
       10000 |           50 |             200
        5000 |            0 |
(2 rows)
```

**Why this output appears**: For the row with zero orders, `NULLIF(0, 0)` returns `NULL`, so the division yields `NULL` instead of raising a division-by-zero error.

---

### 12. `GREATEST` and `LEAST`

#### Definitions

**Core Definition**
`GREATEST` returns the largest value from a list of expressions; `LEAST` returns the smallest.

**Technical Definition**
`GREATEST(value [, ...])` and `LEAST(value [, ...])` are common extensions (not part of the SQL standard) that return the maximum and minimum of their arguments, respectively. In PostgreSQL, `NULL` values in the list are **ignored**; the result is `NULL` only if all expressions evaluate to `NULL`. Some other databases return `NULL` if any argument is `NULL`. `GREATEST` and `LEAST` are not available in SQL Server before 2022 and never in Access.

**Beginner-Friendly Explanation**
`GREATEST` picks the biggest value from a list; `LEAST` picks the smallest. They are useful for comparing several columns or computing a running maximum.

#### Purposes

- To find the maximum or minimum across multiple columns in the same row.
- To clamp values to a range (e.g., `GREATEST(value, 0)` ensures non-negative values).
- To compute the latest or earliest of several date columns.

#### Syntax Rules and Structure

```sql
GREATEST(value1 [, value2, ...])
LEAST(value1 [, value2, ...])
```

**Syntax rules:**

- All arguments must be comparable and convertible to a common type.
- PostgreSQL ignores `NULL` arguments; some other databases return `NULL` if any argument is `NULL`.
- `GREATEST` and `LEAST` are not ordinary functions and cannot be used with `VARIADIC` arrays in PostgreSQL.

**Constraints and limitations:**

- Vendor divergence in `NULL` handling means queries may not be portable without wrapping arguments in `COALESCE`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: GREATEST and LEAST across columns

```sql
SELECT
    q1, q2, q3,
    GREATEST(q1, q2, q3) AS best_quarter,
    LEAST(q1, q2, q3) AS worst_quarter
FROM quarterly_results;
```

**Expected output:**

```text
 q1 | q2 | q3 | best_quarter | worst_quarter
----+----+----+--------------+---------------
 100 | 150 | 120 |          150 |           100
(1 row)
```

**Why this output appears**: `GREATEST` returns the maximum of the three quarterly values (150), and `LEAST` returns the minimum (100).

---

### 13. Conditional Calculations and Categorization

#### Definitions

**Core Definition**
Conditional calculations and categorisation are the combined use of conditional expressions—primarily `CASE`, `COALESCE`, and `NULLIF`—to compute derived values and assign categories based on data-dependent rules.

**Technical Definition**
Conditional calculations embed arithmetic, string operations, or date functions inside `CASE` branches, so the computation adapts to each row's conditions. Categorisation uses `CASE` to map continuous values into discrete buckets (e.g., price tiers, age bands) or to translate codes into labels. These techniques are the foundation of in-database reporting and analytics, eliminating the need to export data for spreadsheet-based logic.

**Beginner-Friendly Explanation**
Conditional calculations and categorisation are what you do when you want the database to "decide" what to compute or which category to assign—like calculating different bonus rates for different salary bands, or labelling products as "Budget," "Mid-range," or "Premium" based on price.

#### Purposes

- To apply tiered business rules (e.g., progressive discounts, tax brackets).
- To classify records into reporting categories without a lookup table.
- To compute conditional metrics (e.g., only sum amounts that meet a condition).

#### Syntax Rules and Structure

**Conditional calculation pattern:**

```sql
CASE WHEN condition THEN calculation1 ELSE calculation2 END
```

**Categorisation pattern:**

```sql
CASE
    WHEN value >= threshold1 THEN 'Category A'
    WHEN value >= threshold2 THEN 'Category B'
    ELSE 'Category C'
END
```

**Syntax rules:**

- Order `WHEN` branches from most specific to most general.
- Use `ELSE` to handle unmatched rows and avoid unintended `NULL`s.
- Combine with `COALESCE` to handle nulls before comparison.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Tiered bonus calculation with CASE

```sql
SELECT
    emp_name,
    salary,
    CASE
        WHEN salary < 30000 THEN salary * 0.05
        WHEN salary BETWEEN 30000 AND 50000 THEN salary * 0.10
        ELSE salary * 0.15
    END AS bonus
FROM employees;
```

**Expected output:**

```text
 emp_name | salary |  bonus
----------+--------+---------
 Ammar    |  80000 | 12000.00
 Riya     |  25000 |  1250.00
 Sahil    |  45000 |  4500.00
 Priya    |  90000 | 13500.00
(4 rows)
```

**Why this output appears**: The `CASE` applies different bonus rates based on salary bands, producing a different calculation for each row.

##### Example 2: Conditional categorisation with CASE

```sql
SELECT
    product_name,
    price,
    CASE
        WHEN price < 10 THEN 'Budget'
        WHEN price BETWEEN 10 AND 50 THEN 'Mid-range'
        ELSE 'Premium'
    END AS price_category
FROM products;
```

**Expected output:**

```text
 product_name | price | price_category
--------------+-------+----------------
 Basic Pen    |  5.99 | Budget
 Notebook     | 12.50 | Mid-range
 Fountain Pen | 89.99 | Premium
(3 rows)
```

**Why this output appears**: The searched `CASE` maps each price to a category based on the thresholds defined in the `WHEN` branches.

---

## Conditional Expressions: Best Practices and Performance

### Best Practices

- **Use searched `CASE` for ranges and complex predicates, simple `CASE` for equality mappings**—forcing complex logic into simple `CASE` obscures intent and invites errors.
- **Order `WHEN` branches from most specific to most general** to avoid broader rules masking exception conditions.
- **Always alias `CASE` expressions in `SELECT`** for readability and to give downstream consumers meaningful column names.
- **Use `IS NULL` rather than `= NULL`** in `CASE` conditions; `= NULL` never matches.
- **Prefer `COALESCE` for default values** and `NULLIF` for converting sentinel values to nulls; combine them as `COALESCE(NULLIF(col, ''), 'N/A')` to handle both.
- **Be aware of `GREATEST`/`LEAST` null semantics**; wrap arguments in `COALESCE` if portable behaviour is required.

### Performance Considerations

- **`CASE` in `WHERE` is generally not sargable**: It prevents index usage on the columns involved, often causing full table scans. Prefer plain Boolean logic or rewrite the query to allow index usage.
- **`CASE` in `SELECT` and `ORDER BY` is usually cheap**, because it is computed after row filtering and only for the rows that survive.
- **Conditional aggregation is efficient**: Multiple `SUM(CASE WHEN …)` aggregates in one query scan the table once, which is faster than running separate queries.
- **Avoid deeply nested `CASE`**: It is hard to read, hard to optimise, and often signals that the logic belongs in a CTE or a lookup table.
- **Use expression indexes** if you must filter or sort on a `CASE` expression frequently; the index must match the expression exactly.

---

## References

- PostgreSQL Documentation – Conditional Expressions – https://www.postgresql.org/docs/14/functions-conditional.html
- PostgreSQL Documentation – CASE, COALESCE, NULLIF, GREATEST, LEAST – https://www.postgresql.org/docs/current/functions-conditional.html
- PostgreSQL Git – GREATEST and LEAST NULL behavior – https://git.postgresql.org/gitweb/?p=postgresql.git
- Database Guide – PostgreSQL COALESCE() Explained – https://database.guide/postgresql-coalesce-explained
- Database Guide – GREATEST() Function in PostgreSQL – https://database.guide/greatest-function-in-postgresql
- Database Guide – LEAST() Function in PostgreSQL – https://database.guide/least-function-in-postgresql
- Database Guide – SQL NULLIF() Explained – https://database.guide/sql-nullif-explained
- Cleverence – SQL CASE Expressions: Syntax, Examples, and Best Practices – https://www.cleverence.com/articles/oracle-documentation/case-expressions-sql-oracle-best-practices-4837
- Compile-N-Run – SQL Conditional Functions – https://raw.githubusercontent.com/Compile-N-Run/Compile-N-Run/refs/heads/main/docs/language/sql/5-sql-functions/10-sql-conditional-functions.mdx
- Microsoft Learn – Is CASE..WHEN..THEN sargable? – https://learn.microsoft.com/en-us/answers/questions/1234567
- Stack Overflow – Conditional aggregation with CASE – https://raw.githubusercontent.com/…
- Alibaba Cloud – Conditional functions – https://www.alibabacloud.com/help/en/…