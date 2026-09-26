# SQL Conditional Expressions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL conditional expressions are language constructs that evaluate one or more conditions and return a result based on whether those conditions are true or false, enabling decision-making logic within SQL statements without procedural code.

**Technical Definition:** SQL conditional expressions—primarily `CASE`, `COALESCE`, and `NULLIF`—are SQL-standard constructs that operate as expressions (returning values) rather than statements (controlling flow). They can be used wherever an expression is valid, including `SELECT`, `WHERE`, `ORDER BY`, `GROUP BY`, and `HAVING` clauses. The `CASE` expression evaluates a list of conditions and returns one of multiple possible result expressions; `COALESCE` returns the first non-NULL argument; `NULLIF` returns NULL if two arguments are equal.

**Beginner-Friendly Explanation:** Conditional expressions in SQL are like the "if-then-else" statements you use in everyday decisions. For example: "If it is raining, take an umbrella; otherwise, wear sunglasses." In SQL, `CASE` lets you say "If the order total is over $100, label it 'Premium'; otherwise, label it 'Standard'." `COALESCE` and `NULLIF` are specialized helpers for dealing with missing (NULL) values.

### Key Characteristics

- **Expression-based:** Conditional expressions return values, not control flow; they can be used inside `SELECT`, `WHERE`, `ORDER BY`, and other clauses.
- **SQL-standard:** `CASE`, `COALESCE`, and `NULLIF` are defined in the SQL standard and supported across PostgreSQL, MySQL, SQL Server, Oracle, and SQLite.
- **Lazy evaluation:** `CASE` does not evaluate subexpressions that are not needed to determine the result; `COALESCE` stops evaluating at the first non-NULL argument.
- **Type-coercion aware:** All result expressions in a `CASE` must be convertible to a single output type; the return type is the highest-precedence type among the results.
- **NULL-aware:** `CASE` comparisons with NULL do not match (since `NULL = NULL` is unknown); `COALESCE` and `NULLIF` are specifically designed for NULL handling.

### Prerequisites

- Basic understanding of SQL `SELECT` statements and expression syntax.
- Familiarity with `NULL` semantics and three-valued logic (`TRUE`, `FALSE`, `UNKNOWN`).
- Knowledge of data types and implicit type conversion rules.
- Awareness of operator precedence and boolean expression evaluation.

### Related Programming Areas

- Data transformation and cleansing pipelines.
- Report generation and business intelligence.
- Application query logic and dynamic filtering.
- Data validation and categorization.
- ETL/ELT transformations.

### Core Concepts / Features

1. **`CASE`** (general conditional expression)
2. **Simple `CASE`** (equality-based comparison)
3. **Searched `CASE`** (boolean condition evaluation)
4. **`COALESCE`** (first non-NULL value)
5. **`NULLIF`** (NULL on equality)
6. **Conditional Calculations** (arithmetic and derived values based on conditions)
7. **Conditional Categorization** (grouping rows into labeled buckets)


## Core Concept 1: CASE

### Definitions

**Core Definition:** `CASE` is a general conditional expression that evaluates a list of conditions and returns one of multiple possible result expressions, similar to if-then-else logic in procedural programming.

**Technical Definition:** The SQL `CASE` expression is a generic conditional expression that evaluates each `WHEN` condition in order; the first condition that evaluates to `TRUE` determines the result. If no condition matches, the `ELSE` result is returned; if `ELSE` is omitted, the result is `NULL`. `CASE` clauses can be used wherever an expression is valid, and the data types of all result expressions must be convertible to a single output type.

**Beginner-Friendly Explanation:** `CASE` is SQL's version of an if-then-else chain. You list several conditions, and SQL checks them in order. The first one that is true determines the answer. If none are true, you can provide a fallback with `ELSE`.

### Purposes

- To return different values based on conditional logic within a single SQL expression.
- To avoid multiple queries or application-side branching for data-dependent transformations.
- To implement business rules directly in SQL, such as tiered pricing or status classification.
- To handle conditional NULL substitution and default values.

### Syntax Rules and Structure

#### Complete General Syntax

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    [ELSE default_result]
END
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `CASE` | Keyword beginning the conditional expression. |
| `WHEN condition THEN result` | A condition-result pair; conditions are evaluated in order. |
| `condition` | A boolean expression that must evaluate to `TRUE` for the result to be returned. |
| `result` | The value or expression returned when the condition is `TRUE`. |
| `ELSE default_result` | Optional fallback returned when no condition is `TRUE`. |
| `END` | Keyword terminating the `CASE` expression. |

#### Syntax Rules

- Conditions are evaluated sequentially; the first `TRUE` condition determines the result, and subsequent conditions are not evaluated.
- If no condition is `TRUE` and `ELSE` is omitted, the result is `NULL`.
- All result expressions must be convertible to a common output type.
- `CASE` can be nested, but SQL Server allows only 10 levels of nesting.
- `CASE` cannot be used to control the flow of execution of Transact-SQL statements, statement blocks, user-defined functions, or stored procedures.

#### Constraints and Limitations

- **Type compatibility:** All `THEN` and `ELSE` results must share a compatible data type; mixing incompatible types raises an error.
- **No short-circuit guarantee:** While most databases short-circuit `CASE` evaluation, the SQL standard does not guarantee that unneeded branches are not evaluated.
- **NULL comparison:** A `CASE` condition like `WHEN column = NULL` will never be `TRUE` because `NULL = NULL` is `UNKNOWN`; use `IS NULL` instead.
- **Version-specific:** `CASE` was introduced in SQL-92; Oracle added support in 8i (as expression in 9i).

### Annotated Code Examples

#### Example 1: PostgreSQL General `CASE` with Multiple Conditions

```sql
-- Create a table with numeric values
CREATE TABLE test (a INT);
INSERT INTO test VALUES (1), (2), (3);

-- General CASE expression
SELECT
    a,
    CASE
        WHEN a = 1 THEN 'one'
        WHEN a = 2 THEN 'two'
        ELSE 'other'
    END AS description
FROM test;
```

**Expected Output:**

```
 a | description
---+-------------
 1 | one
 2 | two
 3 | other
```

**Why This Works:** The `CASE` expression evaluates `WHEN a = 1` first; for `a = 1`, it returns `'one'`. For `a = 2`, the first condition fails and the second condition succeeds, returning `'two'`. For `a = 3`, neither condition matches, so the `ELSE` branch returns `'other'`. This is the canonical example from the PostgreSQL documentation.

#### Example 2: SQL Server `CASE` in `ORDER BY` and `SELECT`

```sql
-- Use CASE in SELECT and ORDER BY
SELECT
    ProductNumber,
    Name,
    CASE ProductLine
        WHEN 'R' THEN 'Road'
        WHEN 'M' THEN 'Mountain'
        WHEN 'T' THEN 'Touring'
        WHEN 'S' THEN 'Other sale items'
        ELSE 'Not for sale'
    END AS Category
FROM Production.Product
ORDER BY ProductNumber;
```

**Expected Output (partial):**

```
ProductNumber | Name              | Category
--------------+-------------------+-------------
AR-5381       | Adjustable Race   | Not for sale
BA-8327       | Bearing Ball      | Not for sale
BE-2349       | BB Ball Bearing   | Not for sale
...
```

**Why This Works:** The `CASE` expression translates single-letter product line codes into readable category names. The `ELSE` clause catches all products with a `NULL` or unrecognized `ProductLine`. This pattern is common in reporting queries to improve readability.

#### Example 3: Oracle `CASE` with `AVG` Aggregation

```sql
-- Oracle: use CASE inside an aggregate to set a floor
SELECT AVG(CASE WHEN e.salary > 2000 THEN e.salary ELSE 2000 END) AS "Average Salary"
FROM employees e;
```

**Expected Output:**

```
Average Salary
--------------
       6461.68
```

**Why This Works:** The `CASE` expression ensures that no salary below 2000 contributes less than 2000 to the average. This demonstrates that `CASE` can be used inside aggregate functions to apply conditional logic before aggregation.

### Real-World Cases

- **Pricing tiers:** Computing different discount rates based on order volume or customer tier.
- **Status mapping:** Translating internal status codes into user-friendly labels.
- **Data masking:** Conditionally replacing sensitive values based on user role or context.
- **Conditional joins:** Using `CASE` in `ON` clauses to match rows differently based on a parameter.

### References

- PostgreSQL Documentation: Conditional Expressions (`CASE`) — https://www.postgresql.org/docs/current/functions-conditional.html
- SQL Server: CASE (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/language-elements/case-transact-sql
- MySQL Reference Manual: CASE Operator — https://dev.mysql.com/doc/refman/8.0/en/flow-control-functions.html
- Oracle Database SQL Language Reference: CASE Expressions — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CASE-Expressions.html


## Core Concept 2: Simple CASE

### Definitions

**Core Definition:** Simple `CASE` is a specialized form of `CASE` that compares a single expression against a set of values for equality, returning a result when a match is found.

**Technical Definition:** The simple `CASE` expression evaluates a `case_value` expression once and compares it to each `when_value` in the `WHEN` clauses until an equal value is found. If no match is found, the `ELSE` result (or `NULL`) is returned. This is similar to a `switch` statement in C.

**Beginner-Friendly Explanation:** Simple `CASE` is like a multiple-choice question. You have one value (the case expression), and you check it against several possible answers. The first answer that matches gives you the result.

### Purposes

- To translate codes or abbreviations into descriptive labels.
- To perform equality-based lookups without writing multiple `OR` conditions.
- To simplify queries that would otherwise require many `WHEN column = value` repetitions.

### Syntax Rules and Structure

#### Complete General Syntax

```sql
CASE expression
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ...
    [ELSE default_result]
END
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `expression` | The value to compare (evaluated once). |
| `WHEN value THEN result` | A comparison-value/result pair. |
| `ELSE default_result` | Optional fallback when no value matches. |

#### Syntax Rules

- The `expression` is evaluated once and compared to each `WHEN value` using the `=` operator.
- The data types of the `expression` and each `WHEN value` must be comparable; implicit conversion may apply.
- Simple `CASE` cannot be used to test for equality with `NULL` because `NULL = NULL` is `FALSE` (not `TRUE`).
- In Oracle, the comparison is collation-sensitive for character data types.

#### Constraints and Limitations

- **NULL comparisons:** `CASE column WHEN NULL THEN ...` will never match; use searched `CASE` with `IS NULL`.
- **No inequality:** Simple `CASE` only supports equality; use searched `CASE` for ranges or inequalities.
- **Type compatibility:** The expression and all `WHEN` values must be of compatible types.

### Annotated Code Examples

#### Example 1: PostgreSQL Simple `CASE` for Code Translation

```sql
-- Create a table with numeric codes
CREATE TABLE test (a INT);
INSERT INTO test VALUES (1), (2), (3);

-- Simple CASE expression
SELECT
    a,
    CASE a
        WHEN 1 THEN 'one'
        WHEN 2 THEN 'two'
        ELSE 'other'
    END AS description
FROM test;
```

**Expected Output:**

```
 a | description
---+-------------
 1 | one
 2 | two
 3 | other
```

**Why This Works:** The simple `CASE` evaluates `a` once, then compares it to `1`, `2`, and so on. This is equivalent to the general `CASE` example but more concise when the comparison is always equality against the same expression.

#### Example 2: Oracle Simple `CASE` for Credit Limit Classification

```sql
-- Oracle: classify credit limits
SELECT
    cust_last_name,
    CASE credit_limit
        WHEN 100 THEN 'Low'
        WHEN 5000 THEN 'High'
        ELSE 'Medium'
    END AS credit
FROM customers
ORDER BY cust_last_name, credit;
```

**Expected Output (partial):**

```
CUST_LAST_NAME | CREDIT
---------------+--------
Adjani         | Medium
Adjani         | Medium
Alexander      | Medium
Altman         | High
Altman         | Medium
```

**Why This Works:** The simple `CASE` compares `credit_limit` against `100`, `5000`, and the `ELSE` fallback. This directly mirrors the Oracle documentation example.

### Real-World Cases

- **Country code to country name:** `CASE country_code WHEN 'US' THEN 'United States' WHEN 'CA' THEN 'Canada' ...`
- **Status code decoding:** Translating numeric status codes (1, 2, 3) into readable labels.
- **Month number to month name:** `CASE month WHEN 1 THEN 'January' WHEN 2 THEN 'February' ...`
- **Product category mapping:** Mapping internal category IDs to display names.

### References

- PostgreSQL Documentation: Simple CASE — https://www.postgresql.org/docs/current/functions-conditional.html#FUNCTIONS-CASE
- Oracle Database SQL Language Reference: Simple CASE Example — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CASE-Expressions.html
- MySQL Reference Manual: CASE Operator — https://dev.mysql.com/doc/refman/8.0/en/flow-control-functions.html


## Core Concept 3: Searched CASE

### Definitions

**Core Definition:** Searched `CASE` is a form of `CASE` that evaluates a set of boolean conditions—rather than comparing a single expression to values—and returns the result associated with the first condition that evaluates to `TRUE`.

**Technical Definition:** The searched `CASE` expression evaluates each `WHEN` clause's boolean expression in turn, from left to right, until one is found that yields `TRUE`. It then returns the corresponding `THEN` result. If no condition is `TRUE`, the `ELSE` result (or `NULL`) is returned. This form supports ranges, inequalities, `LIKE`, `IN`, and any other boolean predicate.

**Beginner-Friendly Explanation:** Searched `CASE` is like asking a series of yes/no questions. "Is the temperature above 30? If yes, say 'Hot'. Is it above 20? If yes, say 'Warm'. Otherwise, say 'Cold'." You check each condition in order and use the first one that is true.

### Purposes

- To classify rows into categories based on ranges (e.g., price bands, age groups).
- To apply complex boolean logic (AND, OR, BETWEEN, LIKE, IN) within a `CASE`.
- To handle NULL checks and inequality comparisons that simple `CASE` cannot express.

### Syntax Rules and Structure

#### Complete General Syntax

```sql
CASE
    WHEN boolean_condition1 THEN result1
    WHEN boolean_condition2 THEN result2
    ...
    [ELSE default_result]
END
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `WHEN boolean_condition THEN result` | A boolean-condition/result pair. |
| `boolean_condition` | Any expression that evaluates to `TRUE`, `FALSE`, or `UNKNOWN`. |
| `ELSE default_result` | Optional fallback when no condition is `TRUE`. |

#### Syntax Rules

- Conditions are evaluated in order; the first `TRUE` condition determines the result.
- Subsequent `WHEN` expressions are not evaluated once a `TRUE` condition is found.
- If no condition is `TRUE` and `ELSE` is omitted, the result is `NULL`.
- Searched `CASE` can test for `NULL` using `IS NULL` and `IS NOT NULL`.

#### Constraints and Limitations

- **Order matters:** Because the first `TRUE` condition wins, more specific conditions should be placed before more general ones.
- **No flow control:** Searched `CASE` is an expression, not a statement; it cannot control execution flow of SQL statements.
- **Performance:** Complex conditions may prevent index usage; test query plans.

### Annotated Code Examples

#### Example 1: PostgreSQL Searched `CASE` for Order Categorization

```sql
-- Create orders table
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    amount NUMERIC(10,2)
);

INSERT INTO orders (amount) VALUES (50.00), (150.00), (250.00), (75.00);

-- Searched CASE: categorize order amounts
SELECT
    id,
    amount,
    CASE
        WHEN amount >= 200 THEN 'High Value'
        WHEN amount >= 100 THEN 'Medium Value'
        ELSE 'Low Value'
    END AS order_category
FROM orders;
```

**Expected Output:**

```
 id | amount | order_category
----+--------+----------------
  1 |  50.00 | Low Value
  2 | 150.00 | Medium Value
  3 | 250.00 | High Value
  4 |  75.00 | Low Value
```

**Why This Works:** The searched `CASE` evaluates `amount >= 200` first; for 250.00, this is true, so it returns `'High Value'`. For 150.00, the first condition is false, but `amount >= 100` is true, returning `'Medium Value'`. For 50.00 and 75.00, both conditions are false, so `ELSE` returns `'Low Value'`.

#### Example 2: SQL Server Searched `CASE` with `BETWEEN` and `IS NULL`

```sql
-- Searched CASE with multiple boolean predicates
SELECT
    ProductNumber,
    Name,
    CASE
        WHEN ListPrice = 0 THEN 'Mfg item - not for resale'
        WHEN ListPrice < 50 THEN 'Under $50'
        WHEN ListPrice BETWEEN 50 AND 250 THEN 'Under $250'
        ELSE 'Over $250'
    END AS "Price Range"
FROM Production.Product
ORDER BY ProductNumber;
```

**Expected Output (partial):**

```
ProductNumber | Name            | Price Range
--------------+-----------------+-----------------------
AR-5381       | Adjustable Race | Mfg item - not for resale
BA-8327       | Bearing Ball    | Mfg item - not for resale
...
BK-M18B-40    | Mountain-500    | Under $250
...
```

**Why This Works:** The searched `CASE` uses `=`, `<`, and `BETWEEN` conditions to classify products into price ranges. The `WHEN ListPrice = 0` branch must come first to avoid zero-price items being captured by the `ListPrice < 50` branch. This matches the SQL Server documentation example.

#### Example 3: PostgreSQL Searched `CASE` with Division-by-Zero Protection

```sql
-- Use CASE to avoid division by zero
SELECT
    numerator,
    denominator,
    CASE
        WHEN denominator <> 0 THEN numerator / denominator
        ELSE NULL
    END AS result
FROM fractions;
```

**Expected Output (for `(10, 2), (5, 0), (9, 3)`):**

```
numerator | denominator | result
----------+-------------+-------
       10 |           2 |   5.0
        5 |           0 |  (null)
        9 |           3 |   3.0
```

**Why This Works:** The `CASE` expression checks `denominator <> 0` before performing the division. When the denominator is zero, it returns `NULL` instead of raising a division-by-zero error. This is a canonical use case from the PostgreSQL documentation.

### Real-World Cases

- **Customer segmentation:** Classifying customers as "Gold", "Silver", or "Bronze" based on lifetime spend.
- **Grade assignment:** Assigning letter grades based on numeric scores.
- **Risk assessment:** Categorizing transactions as "High Risk", "Medium Risk", or "Low Risk" based on amount and frequency.
- **Data quality flagging:** Flagging rows with missing or suspicious values for review.

### References

- PostgreSQL Documentation: Searched CASE — https://www.postgresql.org/docs/current/functions-conditional.html#FUNCTIONS-CASE
- SQL Server: Searched CASE Expression — https://learn.microsoft.com/en-us/sql/t-sql/language-elements/case-transact-sql
- MySQL Reference Manual: CASE Operator — https://dev.mysql.com/doc/refman/8.0/en/flow-control-functions.html


## Core Concept 4: COALESCE

### Definitions

**Core Definition:** `COALESCE` is a SQL function that returns the first non-NULL value from a list of arguments.

**Technical Definition:** The `COALESCE(value1, value2, ...)` function returns the first of its arguments that is not NULL. NULL is returned only if all arguments are NULL. Like `CASE`, `COALESCE` only evaluates the arguments that are needed to determine the result; arguments to the right of the first non-NULL value are not evaluated. This SQL-standard function provides capabilities similar to `NVL` and `IFNULL`, which are used in some other database systems.

**Beginner-Friendly Explanation:** `COALESCE` is like having a backup plan. If the first choice is missing (NULL), try the second choice; if that is missing, try the third, and so on. You get the first value that actually exists.

### Purposes

- To substitute a default value for NULL values in query results.
- To provide fallback values from multiple columns (e.g., mobile phone, home phone, work phone).
- To avoid NULL propagation in arithmetic or string concatenation.
- To simplify `CASE` expressions that check for NULL.

### Syntax Rules and Structure

#### Complete General Syntax

```sql
COALESCE(value1, value2, ..., valueN)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `value1, value2, ..., valueN` | A list of expressions evaluated in order; the first non-NULL value is returned. |

#### Syntax Rules

- `COALESCE` is equivalent to a `CASE` expression: `COALESCE(v1, v2) = CASE WHEN v1 IS NOT NULL THEN v1 ELSE v2 END`.
- All arguments must be of compatible types; the return type is the common type of all arguments.
- `COALESCE` is SQL-standard and supported by all major RDBMS.
- Some databases provide proprietary alternatives: `NVL` (Oracle), `IFNULL` (MySQL), `ISNULL` (SQL Server).

#### Constraints and Limitations

- **Type compatibility:** All arguments must be coercible to a common type.
- **Performance:** `COALESCE` with many arguments may be less efficient than a direct `CASE` if the arguments are expensive subqueries.
- **NULL is not empty string:** `COALESCE` only substitutes for NULL, not for empty strings or zero values.

### Annotated Code Examples

#### Example 1: PostgreSQL `COALESCE` for Display Fallback

```sql
-- Create a table with possibly NULL descriptions
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    description TEXT,
    short_description TEXT
);

INSERT INTO products (description, short_description) VALUES
('A very detailed description', 'Short desc'),
(NULL, 'Short only'),
(NULL, NULL);

-- Use COALESCE to provide a fallback
SELECT
    id,
    COALESCE(description, short_description, '(none)') AS display_text
FROM products;
```

**Expected Output:**

```
 id | display_text
----+----------------------------
  1 | A very detailed description
  2 | Short only
  3 | (none)
```

**Why This Works:** `COALESCE` evaluates `description` first; if it is not NULL, it returns it. If `description` is NULL, it tries `short_description`. If both are NULL, it returns `'(none)'`. This is the exact pattern from the PostgreSQL documentation.

#### Example 2: MySQL `COALESCE` for Phone Number Priority

```sql
-- Select the first available phone number
SELECT
    customer_id,
    COALESCE(mobile_phone, home_phone, work_phone, 'No phone on file') AS primary_phone
FROM customers;
```

**Expected Output (partial):**

```
customer_id | primary_phone
------------+--------------
101         | 555-0101
102         | 555-0202
103         | No phone on file
```

**Why This Works:** The query returns the mobile phone if available; otherwise the home phone; otherwise the work phone; otherwise a default message. This is a common pattern for contact information.

### Real-World Cases

- **Contact information:** Prioritizing mobile, then home, then work phone numbers.
- **Address display:** Using billing address if shipping address is NULL.
- **Report defaults:** Showing `'N/A'` or `0` when a measure is NULL.
- **Configuration fallback:** Using a default setting when a user-specific setting is NULL.

### References

- PostgreSQL Documentation: COALESCE — https://www.postgresql.org/docs/current/functions-conditional.html#FUNCTIONS-COALESCE
- MySQL Reference Manual: Comparison Functions (`COALESCE`) — https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#function_coalesce
- SQL Server: COALESCE (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/language-elements/coalesce-transact-sql
- Oracle Database SQL Language Reference: COALESCE — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/COALESCE.html


## Core Concept 5: NULLIF

### Definitions

**Core Definition:** `NULLIF` is a SQL function that returns NULL if two arguments are equal, otherwise it returns the first argument.

**Technical Definition:** The `NULLIF(value1, value2)` function returns a NULL value if `value1` equals `value2`; otherwise it returns `value1`. This can be used to perform the inverse operation of a `COALESCE` example: if a value is a placeholder like `'(none)'`, return a NULL instead. `NULLIF` must be of compatible data types.

**Beginner-Friendly Explanation:** `NULLIF` is like a filter that says "If this value is equal to this specific thing, treat it as missing (NULL); otherwise, keep it." It is useful for converting placeholder values into real NULLs.

### Purposes

- To convert placeholder or sentinel values (e.g., `'N/A'`, `0`, `'none'`) into proper NULLs.
- To avoid division by zero by returning NULL when the denominator is zero.
- To conditionally treat a value as missing based on comparison with a reference value.

### Syntax Rules and Structure

#### Complete General Syntax

```sql
NULLIF(value1, value2)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `value1` | The value to return if it is not equal to `value2`. |
| `value2` | The value to compare against; if equal to `value1`, NULL is returned. |

#### Syntax Rules

- `NULLIF(v1, v2)` is equivalent to `CASE WHEN v1 = v2 THEN NULL ELSE v1 END`.
- The two arguments must be of comparable data types.
- If `value1` is NULL, `NULLIF` returns NULL (since NULL is not equal to anything).
- `NULLIF` is SQL-standard and supported across all major RDBMS.

#### Constraints and Limitations

- **Type compatibility:** Arguments must be comparable; implicit conversion may apply.
- **NULL semantics:** `NULLIF(NULL, value)` returns NULL; `NULLIF(value, NULL)` returns `value` (since `value = NULL` is `UNKNOWN`, not `TRUE`).
- **Not a general replacement:** `NULLIF` only handles equality; for inequality-based NULL substitution, use `CASE`.

### Annotated Code Examples

#### Example 1: PostgreSQL `NULLIF` to Convert Sentinel to NULL

```sql
-- Create a table with a placeholder value
CREATE TABLE items (
    id SERIAL PRIMARY KEY,
    value TEXT
);

INSERT INTO items (value) VALUES ('actual'), '(none)');

-- Convert '(none)' to NULL
SELECT
    id,
    value,
    NULLIF(value, '(none)') AS cleaned_value
FROM items;
```

**Expected Output:**

```
 id | value   | cleaned_value
----+---------+---------------
  1 | actual  | actual
  2 | (none)  | (null)
```

**Why This Works:** `NULLIF(value, '(none)')` returns NULL when `value` equals `'(none)'`; otherwise it returns `value`. This is the inverse of the `COALESCE` example from the PostgreSQL documentation.

#### Example 2: MySQL `NULLIF` to Prevent Division by Zero

```sql
-- Avoid division by zero using NULLIF
SELECT
    total_revenue,
    total_orders,
    total_revenue / NULLIF(total_orders, 0) AS avg_order_value
FROM sales_summary;
```

**Expected Output (for `(1000, 10), (500, 0)`):**

```
total_revenue | total_orders | avg_order_value
--------------+--------------+-----------------
         1000 |           10 |          100.00
          500 |            0 |          (null)
```

**Why This Works:** When `total_orders` is 0, `NULLIF(total_orders, 0)` returns NULL, and the division `total_revenue / NULL` yields NULL instead of raising a division-by-zero error. This is a widely used idiom in SQL.

### Real-World Cases

- **Sentinel value cleanup:** Converting `'N/A'`, `'Unknown'`, or `0` sentinel values into proper NULLs before aggregation.
- **Safe division:** Preventing division-by-zero errors in calculated columns.
- **Data normalization:** Treating placeholder values as missing data during ETL processing.
- **Conditional NULL:** Marking values as NULL when they match a specific "not applicable" value.

### References

- PostgreSQL Documentation: NULLIF — https://www.postgresql.org/docs/current/functions-conditional.html#FUNCTIONS-NULLIF
- MySQL Reference Manual: NULLIF — https://dev.mysql.com/doc/refman/8.0/en/flow-control-functions.html#function_nullif
- SQL Server: NULLIF (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/language-elements/nullif-transact-sql
- Oracle Database SQL Language Reference: NULLIF — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/NULLIF.html


## Core Concept 6: Conditional Calculations

### Definitions

**Core Definition:** Conditional calculations use `CASE` expressions to compute different arithmetic or derived values depending on which condition is met.

**Technical Definition:** Conditional calculations embed `CASE` expressions inside arithmetic operations, aggregate functions, or derived column definitions to apply different formulas based on row-level or group-level conditions. For example, a discount rate may depend on order volume, or an adjusted price may depend on product category.

**Beginner-Friendly Explanation:** Conditional calculations let you say "If the customer is a premium member, apply a 20% discount; otherwise, apply a 5% discount." The math changes based on the condition.

### Purposes

- To apply tiered pricing, discounts, or tax rates based on conditional rules.
- To compute derived metrics that vary by category or segment.
- To adjust values conditionally (e.g., capping, flooring, or scaling).
- To calculate conditional averages or sums within aggregate queries.

### Syntax Rules and Structure

#### Complete General Syntax

```sql
CASE
    WHEN condition THEN arithmetic_expression1
    WHEN condition THEN arithmetic_expression2
    ELSE arithmetic_expression_default
END
```

#### Syntax Rules

- The result expressions can be arithmetic operations involving columns, constants, or functions.
- `CASE` can be nested inside aggregate functions: `SUM(CASE WHEN ... THEN ... ELSE ... END)`.
- The output type follows the same type-coercion rules as general `CASE`.

#### Constraints and Limitations

- **Division by zero:** Guard against zero denominators using `NULLIF` or a `WHEN denominator <> 0` condition.
- **NULL propagation:** Arithmetic with NULL yields NULL; use `COALESCE` to substitute defaults.
- **Performance:** Complex conditional calculations may prevent index usage; consider computed columns or materialized views.

### Annotated Code Examples

#### Example 1: Oracle Conditional Salary Increase

```sql
-- Oracle: projected salary increase based on hire date
SELECT
    first_name || ' ' || last_name AS "Name",
    hire_date AS "Date Started",
    salary AS "Current Pay",
    CASE
        WHEN hire_date < TO_DATE('01-Jan-00') THEN TRUNC(salary * 1.05, 0)
        ELSE TRUNC(salary * 1.15, 0)
    END AS "Proposed Salary"
FROM employees;
```

**Expected Output (partial):**

```
Name          | Date Started | Current Pay | Proposed Salary
--------------+--------------+-------------+----------------
Steven King   | 17-JUN-03    |     24000   |          27600
Neena Kochhar | 21-SEP-05    |     17000   |          19550
...
```

**Why This Works:** Employees hired before 2000 receive a 5% increase; all others receive 15%. The `CASE` expression computes different arithmetic based on the `hire_date` condition. This is the Oracle documentation example.

#### Example 2: SQL Server Conditional Price Calculation

```sql
-- SQL Server: conditional pricing based on product class
SELECT
    Name,
    ListPrice,
    CASE Class
        WHEN 'H' THEN ROUND(ListPrice * 0.60, 2)
        WHEN 'L' THEN ROUND(ListPrice * 0.70, 2)
        WHEN 'M' THEN ROUND(ListPrice * 0.80, 2)
        ELSE ListPrice
    END AS "Adjusted Price"
FROM Production.Product;
```

**Expected Output (partial):**

```
Name            | ListPrice | Adjusted Price
----------------+-----------+----------------
Adjustable Race |      0.00 |           0.00
Bearing Ball    |      0.00 |           0.00
...
HL Road Frame   |   1431.50 |         859.50
```

**Why This Works:** The `CASE` expression applies different discount multipliers based on the product `Class` (`H` = High, `L` = Low, `M` = Medium). Products with no class fall through to the `ELSE` branch and retain the original price.

### Real-World Cases

- **Tiered commission:** Calculating sales commission rates based on deal size.
- **Tax computation:** Applying different tax rates based on jurisdiction or product type.
- **Inventory valuation:** Using different cost bases (FIFO, LIFO, average) conditionally.
- **Bonus calculation:** Computing employee bonuses based on performance rating and tenure.

### References

- Oracle Database SQL Language Reference: CASE in Calculations — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CASE-Expressions.html
- SQL Server: CASE in Computed Columns — https://learn.microsoft.com/en-us/sql/t-sql/language-elements/case-transact-sql
- PostgreSQL Documentation: Conditional Expressions — https://www.postgresql.org/docs/current/functions-conditional.html


## Core Concept 7: Conditional Categorization

### Definitions

**Core Definition:** Conditional categorization is the use of `CASE` expressions to assign rows to discrete categories or labels based on one or more conditions.

**Technical Definition:** Conditional categorization applies a searched or simple `CASE` expression to map continuous or coded values into categorical buckets—such as price ranges, age groups, or performance tiers—that are more meaningful for reporting and analysis. The `CASE` expression is typically used in the `SELECT` list to produce a new categorical column.

**Beginner-Friendly Explanation:** Conditional categorization is like sorting items into labeled boxes. "If the price is under $50, put it in the 'Budget' box; if it's under $250, put it in the 'Mid-range' box; otherwise, put it in the 'Premium' box."

### Purposes

- To group continuous values into meaningful ranges for reporting.
- To translate coded values into human-readable labels.
- To segment customers, products, or transactions for analysis.
- To simplify filtering and aggregation by category.

### Syntax Rules and Structure

#### Complete General Syntax

```sql
CASE
    WHEN range_condition1 THEN 'Category Label 1'
    WHEN range_condition2 THEN 'Category Label 2'
    ...
    ELSE 'Other'
END AS category_column
```

#### Syntax Rules

- Range conditions should be ordered from most restrictive to least restrictive to ensure correct categorization.
- The `ELSE` clause provides a catch-all category for values that do not match any condition.
- The resulting category column can be used in `GROUP BY`, `ORDER BY`, and `HAVING` clauses.

#### Constraints and Limitations

- **Boundary ambiguity:** Ensure that range boundaries do not overlap or leave gaps.
- **NULL handling:** NULL values do not match any `WHEN` condition unless explicitly handled with `IS NULL`.
- **Performance:** Categorization in `GROUP BY` may prevent index usage; consider a persisted computed column.

### Annotated Code Examples

#### Example 1: SQL Server Price Range Categorization

```sql
-- SQL Server: categorize products by price range
SELECT
    ProductNumber,
    Name,
    ListPrice,
    CASE
        WHEN ListPrice = 0 THEN 'Mfg item - not for resale'
        WHEN ListPrice < 50 THEN 'Under $50'
        WHEN ListPrice >= 50 AND ListPrice < 250 THEN 'Under $250'
        WHEN ListPrice >= 250 AND ListPrice < 1000 THEN 'Under $1000'
        ELSE 'Over $1000'
    END AS "Price Range"
FROM Production.Product
ORDER BY ListPrice;
```

**Expected Output (partial):**

```
ProductNumber | Name            | ListPrice | Price Range
--------------+-----------------+-----------+-----------------------
AR-5381       | Adjustable Race |      0.00 | Mfg item - not for resale
BA-8327       | Bearing Ball    |      0.00 | Mfg item - not for resale
...
BK-M18B-40    | Mountain-500    |    539.99 | Under $1000
...
```

**Why This Works:** The searched `CASE` assigns each product to a price range. The conditions are ordered from most specific (`ListPrice = 0`) to most general (`ListPrice < 50`), ensuring correct categorization. This is the SQL Server documentation example.

#### Example 2: Student GPA Categorization

```sql
-- Categorize students by GPA
SELECT
    student_name,
    gpa,
    CASE
        WHEN gpa >= 3.5 THEN 'Honors'
        WHEN gpa >= 2.5 THEN 'Satisfactory'
        WHEN gpa >= 1.5 THEN 'Needs Improvement'
        ELSE 'At Risk'
    END AS academic_standing
FROM students;
```

**Expected Output:**

```
student_name | gpa | academic_standing
-------------+-----+------------------
Alice        | 3.8 | Honors
Bob          | 2.7 | Satisfactory
Carol        | 1.8 | Needs Improvement
Dave         | 1.2 | At Risk
```

**Why This Works:** The `CASE` expression buckets GPA values into four categories. The conditions are ordered from highest to lowest to ensure the first matching condition determines the category.

### Real-World Cases

- **Customer segmentation:** RFM (Recency, Frequency, Monetary) scoring and tier assignment.
- **Product classification:** Grouping products into price bands, weight classes, or risk categories.
- **Employee grading:** Assigning performance ratings based on quantitative metrics.
- **Financial reporting:** Classifying transactions by amount range for audit sampling.

### References

- SQL Server: Using CASE to Categorize Data — https://learn.microsoft.com/en-us/sql/t-sql/language-elements/case-transact-sql
- PostgreSQL Documentation: Conditional Expressions — https://www.postgresql.org/docs/current/functions-conditional.html
- Oracle Database SQL Language Reference: CASE Expressions — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CASE-Expressions.html


## Summary Table: Conditional Expressions Comparison

| Expression | Type | Primary Use | NULL Handling | SQL Standard |
|------------|------|-------------|---------------|--------------|
| `CASE` (general) | Expression | Any conditional logic | Returns NULL if no match | Yes |
| Simple `CASE` | Expression | Equality-based lookup | Cannot match NULL | Yes |
| Searched `CASE` | Expression | Boolean condition evaluation | Can test `IS NULL` | Yes |
| `COALESCE` | Function | First non-NULL value | Returns NULL only if all NULL | Yes |
| `NULLIF` | Function | NULL on equality | Returns NULL when equal | Yes |
| Conditional Calculations | Pattern | Tiered arithmetic | Depends on expressions | N/A (pattern) |
| Conditional Categorization | Pattern | Bucketing and labeling | Depends on conditions | N/A (pattern) |


## Final Notes on Deprecated and Unsafe Features

- **`CASE` with `NULL` comparison:** Writing `CASE WHEN column = NULL THEN ...` never matches because `NULL = NULL` is `UNKNOWN`. Always use `IS NULL` or `IS NOT NULL`.
- **Simple `CASE` with NULL:** Simple `CASE` cannot test for NULL equality; use searched `CASE` with `IS NULL` instead.
- **MySQL `CASE` statement vs. expression:** MySQL distinguishes between the `CASE` statement (used in stored programs, terminated by `END CASE`) and the `CASE` operator/expression (used in SQL queries, terminated by `END`). Do not confuse them.
- **SQL Server nesting limit:** SQL Server allows only 10 levels of nested `CASE` expressions.
- **Division by zero:** Always guard conditional calculations involving division with `NULLIF` or a `WHEN denominator <> 0` condition.
- **Version-specific:** `CASE` as an expression in Oracle was introduced in 8i and enhanced in 9i; earlier versions required `DECODE`. PostgreSQL has supported `CASE` since early versions.