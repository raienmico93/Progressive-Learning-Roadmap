# SQL BETWEEN: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The BETWEEN operator is a SQL logical operator that tests whether a value falls within a specified range of values, inclusive of both endpoints.

**Technical Definition:** Per the ANSI/ISO SQL Standard, the BETWEEN search condition is a core feature that evaluates whether an expression lies between two other expressions. It is semantically equivalent to the combination of two inequalities: `expression >= start_expression AND expression <= end_expression`. The BETWEEN search condition can evaluate as TRUE, FALSE, or UNKNOWN, and the NOT keyword reverses the meaning but leaves UNKNOWN unchanged.

**Beginner-Friendly Explanation:** BETWEEN is a shorthand way of saying "this value is greater than or equal to X and less than or equal to Y." Instead of writing two separate conditions with AND, you write one clean condition. For example, "find employees whose salary is between 50,000 and 100,000" includes everyone earning exactly 50,000 and exactly 100,000.

### Key Characteristics

- **Inclusive endpoints:** BETWEEN includes both the lower and upper bounds in the range.
- **Syntactic sugar:** `a BETWEEN x AND y` is equivalent to `a >= x AND a <= y`.
- **Three-valued logic:** Returns TRUE, FALSE, or UNKNOWN (when any operand is NULL).
- **Versatile data types:** Works with numeric, character, and datetime expressions.
- **Vendor extensions:** PostgreSQL supports BETWEEN SYMMETRIC; MySQL supports type conversion rules; Snowflake has specific collation behavior.
- **Performance-friendly:** BETWEEN can use indexes when the column is indexed, unlike some function-based comparisons.

### Prerequisites

- Basic SQL syntax (SELECT, FROM, WHERE)
- Understanding of comparison operators (`>=`, `<=`, `>`, `<`)
- Familiarity with logical operators (AND, OR, NOT)
- Basic knowledge of data types (numeric, character, datetime)
- Understanding of NULL handling and three-valued logic

### Related Programming Areas

- **Data Filtering and Querying:** BETWEEN is primarily used in WHERE clauses for range-based filtering.
- **Data Validation:** Validating that values fall within acceptable ranges (e.g., ages, dates, scores).
- **Reporting and Analytics:** Creating date-range reports, age brackets, and price ranges.
- **Data Cleaning:** Identifying records with values outside expected ranges.
- **Application Development:** Implementing range filters in user interfaces (date pickers, price sliders).

---

## Core Concepts / Key Features

### 1. Core BETWEEN Syntax and Mechanics

#### 1.1 Basic Syntax and Usage

**Core Definitions:**

- **Core Definition:** The BETWEEN operator compares a test expression to a lower bound and an upper bound, returning TRUE if the test expression is within the inclusive range.
- **Technical Definition:** The syntax is `test_expression [NOT] BETWEEN begin_expression AND end_expression`. All three expressions must be of compatible data types (numeric, character, or datetime). BETWEEN returns TRUE if `test_expression >= begin_expression AND test_expression <= end_expression`.
- **Beginner-Friendly Explanation:** You provide a value to test, a starting point, and an ending point. BETWEEN checks if the value falls between those two points, including the points themselves.

**Purposes:**

- To filter rows where a column value falls within a specified range
- To simplify queries by replacing two inequality conditions with one BETWEEN condition
- To perform date-range filtering (e.g., "orders between January 1 and March 31")
- To create numeric ranges (e.g., "products between $10 and $50")

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
test_expression [ NOT ] BETWEEN begin_expression AND end_expression
```

**Component Breakdown:**

- `test_expression`: The expression to test (column, literal, or expression)
- `NOT`: Optional keyword that negates the result
- `BETWEEN`: The operator keyword
- `begin_expression`: The lower bound of the range
- `AND`: Required keyword separating the bounds
- `end_expression`: The upper bound of the range

**Syntax Rules:**

- All three expressions must be of compatible data types.
- `begin_expression` should be less than or equal to `end_expression` for a non-empty range.
- If `end_expression` is less than `begin_expression`, the range is empty and BETWEEN returns FALSE (unless NOT is used).
- The result is a Boolean value (TRUE, FALSE, or UNKNOWN).

**Constraints and Limitations:**

- BETWEEN is inclusive of both endpoints. For an exclusive range, use `>` and `<` operators instead.
- If any input is NULL, the result depends on the constituent comparisons; if either comparison is FALSE, the overall expression is FALSE; otherwise it is UNKNOWN.
- MySQL recommends using CAST() for date/time values to avoid implicit conversion issues.

**Annotated Code Examples:**

```sql
-- Setup: Create a sample table
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100),
    salary DECIMAL(10,2),
    hire_date DATE,
    department VARCHAR(50)
);

INSERT INTO employees VALUES (1, 'Alice', 85000, '2020-03-15', 'Engineering');
INSERT INTO employees VALUES (2, 'Bob', 62000, '2021-06-01', 'Marketing');
INSERT INTO employees VALUES (3, 'Charlie', 95000, '2019-01-10', 'Engineering');
INSERT INTO employees VALUES (4, 'Diana', 48000, '2022-11-20', 'Sales');
INSERT INTO employees VALUES (5, 'Eve', 110000, '2018-07-05', 'Engineering');

-- Example 1: Numeric range - employees earning between 60,000 and 95,000
SELECT emp_name, salary
FROM employees
WHERE salary BETWEEN 60000 AND 95000;
```

**Expected Output:**

| emp_name | salary |
|----------|--------|
| Alice    | 85000  |
| Bob      | 62000  |
| Charlie  | 95000  |

**Explanation:** The range includes both 60,000 and 95,000. Alice (85,000), Bob (62,000), and Charlie (95,000) all fall within the range. Diana (48,000) and Eve (110,000) are excluded.

```sql
-- Example 2: Date range - employees hired between 2019 and 2021
SELECT emp_name, hire_date
FROM employees
WHERE hire_date BETWEEN '2019-01-01' AND '2021-12-31';
```

**Expected Output:**

| emp_name | hire_date  |
|----------|------------|
| Alice    | 2020-03-15 |
| Bob      | 2021-06-01 |
| Charlie  | 2019-01-10 |

**Explanation:** The date range includes January 1, 2019 through December 31, 2021. All three employees were hired within this period.

```sql
-- Example 3: Text range - employees whose names start with letters between 'B' and 'D'
SELECT emp_name
FROM employees
WHERE emp_name BETWEEN 'B' AND 'D';
```

**Expected Output:**

| emp_name |
|----------|
| Bob      |
| Charlie  |

**Explanation:** 'B' <= 'Bob' <= 'D' is TRUE; 'B' <= 'Charlie' <= 'D' is TRUE. 'Alice' starts with 'A' (less than 'B'), and 'Diana' starts with 'D' but the comparison is character-by-character: 'Diana' > 'D' because 'D' is a prefix. Eve starts with 'E' (greater than 'D').

**Real-World Cases:**

- **HR systems:** Filtering employees by salary grade bands or hire date ranges.
- **E-commerce:** Finding products within a price range ($10–$50).
- **Finance:** Identifying transactions within a specific amount range for fraud detection.
- **Education:** Filtering students by test scores within a passing range.

**References:**

- Microsoft Learn: BETWEEN (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/between-transact-sql
- PostgreSQL: Comparison Functions and Operators - https://www.postgresql.org/docs/17/functions-comparison.html
- Oracle: BETWEEN Condition - https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/BETWEEN-Condition.html
- MySQL: Comparison Functions and Operators - https://downloads.mysql.com/docs/refman-5.6-en.a4.pdf
- SAP Help Portal: BETWEEN Search Condition - https://help.sap.com/docs/SAP_SQL_Anywhere/93079d4ba8e44920ae63ffb4def91f5b/816aaf436ce21014bf71db9864f64ec4.html


#### 1.2 NOT BETWEEN Operator for Inverse Matching

**Core Definitions:**

- **Core Definition:** NOT BETWEEN returns TRUE when the test expression is outside the specified range.
- **Technical Definition:** `expr NOT BETWEEN min AND max` is logically equivalent to `NOT (expr BETWEEN min AND max)`, which is equivalent to `expr < min OR expr > max`. The NOT keyword reverses the meaning but leaves UNKNOWN unchanged.
- **Beginner-Friendly Explanation:** NOT BETWEEN finds everything that is NOT in the range. If you want to find employees earning less than 60,000 or more than 95,000, you use NOT BETWEEN.

**Purposes:**

- To exclude values within a specified range
- To find data outside expected boundaries
- To implement negative range filtering
- To identify outliers or anomalies

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
test_expression NOT BETWEEN begin_expression AND end_expression
```

**Component Breakdown:**

- Same components as BETWEEN, with NOT negating the result
- Returns TRUE if `test_expression < begin_expression OR test_expression > end_expression`
- Returns FALSE if `test_expression >= begin_expression AND test_expression <= end_expression`
- Returns UNKNOWN if any operand is NULL (following three-valued logic)

**Syntax Rules:**

- NOT BETWEEN follows the same syntax rules as BETWEEN.
- NULL handling: If any operand is NULL, the result is UNKNOWN (not TRUE).

**Constraints and Limitations:**

- Same data type compatibility requirements as BETWEEN.
- NOT BETWEEN does not include the endpoints in the exclusion (i.e., endpoints ARE excluded from the result).

**Annotated Code Examples:**

```sql
-- Find employees NOT earning between 60,000 and 95,000
SELECT emp_name, salary
FROM employees
WHERE salary NOT BETWEEN 60000 AND 95000;
```

**Expected Output:**

| emp_name | salary |
|----------|--------|
| Diana    | 48000  |
| Eve      | 110000 |

**Explanation:** Diana (48,000) is below 60,000, and Eve (110,000) is above 95,000. Both are outside the range, so NOT BETWEEN returns TRUE for them.

```sql
-- Find employees hired outside the 2019-2021 range
SELECT emp_name, hire_date
FROM employees
WHERE hire_date NOT BETWEEN '2019-01-01' AND '2021-12-31';
```

**Expected Output:**

| emp_name | hire_date  |
|----------|------------|
| Diana    | 2022-11-20 |
| Eve      | 2018-07-05 |

**Explanation:** Diana was hired after 2021, and Eve was hired before 2019. Both fall outside the range.

**Real-World Cases:**

- **Data quality:** Finding records with values outside expected ranges (e.g., negative ages, future dates).
- **Fraud detection:** Identifying transactions outside normal spending patterns.
- **Inventory:** Finding products with stock levels outside safe thresholds.
- **Compliance:** Identifying employees with salaries outside regulatory bands.

**References:**

- Microsoft Learn: BETWEEN (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/between-transact-sql
- PostgreSQL: Comparison Functions and Operators - https://www.postgresql.org/docs/17/functions-comparison.html
- Oracle: BETWEEN Condition - https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/BETWEEN-Condition.html


### 2. Data Type Support

#### 2.1 Numeric Ranges

**Core Definitions:**

- **Core Definition:** BETWEEN works with all numeric data types (INT, DECIMAL, FLOAT, etc.) to test whether a value falls within a numeric range.
- **Technical Definition:** The numeric BETWEEN predicate returns TRUE if the test expression is greater than or equal to the lower bound and less than or equal to the upper bound. All three expressions must be of compatible numeric types; implicit conversion may occur if types differ.
- **Beginner-Friendly Explanation:** You can use BETWEEN with numbers just like you would with any comparison. "Is this salary between 50,000 and 100,000?"

**Purposes:**

- To filter numeric data by range (prices, salaries, quantities, ages)
- To create numeric brackets for analysis
- To validate that numeric values fall within acceptable limits

**Annotated Code Examples:**

```sql
-- Find products with prices between $10 and $50
SELECT product_name, price
FROM products
WHERE price BETWEEN 10.00 AND 50.00;

-- Find employees with ages between 25 and 35
SELECT emp_name, age
FROM employees
WHERE age BETWEEN 25 AND 35;
```

**Real-World Cases:**

- **Retail:** Price range filtering on e-commerce sites.
- **HR:** Age bracket analysis for workforce demographics.
- **Finance:** Transaction amount ranges for fraud detection.

**References:**

- Microsoft Learn: BETWEEN (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/between-transact-sql
- MySQL: Comparison Functions and Operators - https://downloads.mysql.com/docs/refman-5.6-en.a4.pdf


#### 2.2 Date and Time Ranges

**Core Definitions:**

- **Core Definition:** BETWEEN works with date, time, and datetime data types to test whether a date falls within a date range.
- **Technical Definition:** The BETWEEN predicate compares date/time values according to chronological order. All three expressions must be datetime-compatible. Implicit conversion may occur if types differ (e.g., DATE vs. DATETIME), and this can affect results.
- **Beginner-Friendly Explanation:** You can filter records by date ranges, like "show me all orders placed between January 1 and March 31, 2024."

**Purposes:**

- To filter records by date range (monthly, quarterly, yearly reports)
- To find records within a specific time period
- To implement date-based business rules (e.g., "active within the last 30 days")

**Syntax Structures and Rules:**

**Important:** When using BETWEEN with DATETIME or TIMESTAMP columns and DATE literals, the DATE values are implicitly cast to DATETIME with time 00:00:00. This means `BETWEEN '2024-01-01' AND '2024-01-31'` will miss records on January 31 after midnight. Use the next day's date as the upper bound: `BETWEEN '2024-01-01' AND '2024-02-01'`.

**Annotated Code Examples:**

```sql
-- Find orders placed in Q1 2024
SELECT order_id, order_date, total
FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-03-31';

-- For DATETIME columns, use the next day as upper bound
SELECT order_id, order_date
FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-04-01';
```

**Real-World Cases:**

- **E-commerce:** Daily, weekly, monthly, or quarterly sales reports.
- **Banking:** Transaction history within a statement period.
- **Log analysis:** Finding events within a specific time window.

**References:**

- Microsoft Learn: BETWEEN (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/between-transact-sql
- Snowflake: [NOT] BETWEEN - https://docs.snowflake.com/ja/sql-reference/functions/between
- MySQL: Comparison Functions and Operators - https://downloads.mysql.com/docs/refman-5.6-en.a4.pdf


#### 2.3 Text/String Ranges

**Core Definitions:**

- **Core Definition:** BETWEEN works with character data types to test whether a string falls within a lexicographic (alphabetical) range.
- **Technical Definition:** The BETWEEN predicate compares strings character-by-character using the collation of the operands. The result depends on collation rules (case sensitivity, accent sensitivity, etc.). `'b' BETWEEN 'a' AND 'c'` returns TRUE; `'B' BETWEEN 'a' AND 'c'` may return FALSE in case-sensitive collations.
- **Beginner-Friendly Explanation:** You can filter text by alphabetical ranges, like "find all names between 'Anderson' and 'Johnson'."

**Purposes:**

- To filter records by name or text ranges
- To implement alphabetical grouping or segmentation
- To find records within a specific text range

**Annotated Code Examples:**

```sql
-- Find employees whose last name falls between 'B' and 'M'
SELECT first_name, last_name
FROM employees
WHERE last_name BETWEEN 'B' AND 'M';

-- Find products with names between 'C' and 'F'
SELECT product_name
FROM products
WHERE product_name BETWEEN 'C' AND 'F';
```

**Expected Output (for `BETWEEN 'B' AND 'M'`):**

| first_name | last_name |
|------------|-----------|
| John       | Doe       |
| Robert     | Johnson   |

**Explanation:** Last names starting with letters B through M are included. 'Smith' (S) is excluded, and 'Anderson' (A) is excluded.

**Real-World Cases:**

- **CRM:** Segmenting customers by last name for direct mail campaigns.
- **Inventory:** Finding products by alphabetical range.
- **Education:** Grouping students by surname.

**References:**

- PostgreSQL: Comparison Functions and Operators - https://www.postgresql.org/docs/17/functions-comparison.html
- Snowflake: [NOT] BETWEEN - https://docs.snowflake.com/ja/sql-reference/functions/between
- Programiz: SQL BETWEEN Operator - https://www.programiz.com/sql/between-operator


### 3. Vendor-Specific Extensions

#### 3.1 PostgreSQL: BETWEEN SYMMETRIC

**Core Definitions:**

- **Core Definition:** BETWEEN SYMMETRIC is a PostgreSQL extension that allows the range bounds to be specified in any order, automatically sorting them.
- **Technical Definition:** `BETWEEN SYMMETRIC` is like BETWEEN except there is no requirement that the argument to the left of AND be less than or equal to the argument on the right. If the left bound is greater than the right bound, they are swapped before the comparison.
- **Beginner-Friendly Explanation:** Normally, BETWEEN requires the lower bound first. BETWEEN SYMMETRIC is more forgiving—you can give the bounds in either order and it will figure it out.

**Purposes:**

- To write queries where the bound order is not known in advance
- To handle user input where the user might enter the upper bound first
- To simplify application code by removing the need to sort bounds

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
test_expression BETWEEN SYMMETRIC bound1 AND bound2
```

**Component Breakdown:**

- `bound1` and `bound2`: The two bounds, in any order
- PostgreSQL automatically sorts them so that the lower bound is used first

**Syntax Rules:**

- Supported only in PostgreSQL.
- NOT BETWEEN SYMMETRIC is also available.
- The collation used for sorting the bounds is the default collation.

**Constraints and Limitations:**

- Not portable to other databases.
- If bound1 and bound2 are NULL, the result is NULL.

**Annotated Code Examples:**

```sql
-- PostgreSQL: BETWEEN SYMMETRIC with bounds in reverse order
SELECT 2 BETWEEN SYMMETRIC 3 AND 1;  -- Returns TRUE (2 is between 1 and 3)
SELECT 2 BETWEEN 3 AND 1;            -- Returns FALSE (empty range)
```

**Expected Output:**

| Expression | Result |
|------------|--------|
| `2 BETWEEN SYMMETRIC 3 AND 1` | TRUE |
| `2 BETWEEN 3 AND 1` | FALSE |

**Explanation:** BETWEEN SYMMETRIC sorts the bounds (1 and 3) before comparison, so 2 falls within the range. Regular BETWEEN treats 3 AND 1 as an empty range because 3 > 1.

**Real-World Cases:**

- **User input:** When users can enter date ranges in any order.
- **API development:** Handling range parameters that may not be sorted.
- **Data analysis:** Flexible range queries without preprocessing bounds.

**References:**

- PostgreSQL: Comparison Functions and Operators - https://www.postgresql.org/docs/17/functions-comparison.html


#### 3.2 MySQL: Type Conversion and CAST Recommendations

**Core Definitions:**

- **Core Definition:** MySQL's BETWEEN operator performs implicit type conversion when operands are of different types, which can lead to unexpected results.
- **Technical Definition:** If all arguments are of the same type, BETWEEN is equivalent to `(min <= expr AND expr <= max)`. Otherwise, type conversion takes place according to MySQL's type conversion rules applied to all three arguments. MySQL recommends using CAST() for date/time comparisons to ensure correct behavior.
- **Beginner-Friendly Explanation:** MySQL tries to be helpful by converting types automatically, but this can cause surprises. For dates, explicitly converting values with CAST() avoids problems.

**Purposes:**

- To understand implicit type conversion behavior in MySQL
- To write portable, reliable BETWEEN queries with dates
- To avoid subtle bugs from type mismatches

**Annotated Code Examples:**

```sql
-- MySQL: Without CAST (may produce unexpected results)
SELECT * FROM orders WHERE order_date BETWEEN '2024-01-01' AND '2024-01-31';

-- MySQL: With CAST (recommended)
SELECT * FROM orders
WHERE order_date BETWEEN CAST('2024-01-01' AS DATETIME)
                     AND CAST('2024-01-31' AS DATETIME);
```

**Real-World Cases:**

- **Data migration:** Ensuring correct date comparisons when moving data between systems.
- **Application development:** Writing reliable date-range queries in MySQL.

**References:**

- MySQL: Comparison Functions and Operators - https://downloads.mysql.com/docs/refman-5.6-en.a4.pdf


#### 3.3 Snowflake: Collation and Date Casting

**Core Definitions:**

- **Core Definition:** Snowflake's BETWEEN operator follows the same inclusive semantics but has specific rules for collation and date casting.
- **Technical Definition:** In Snowflake, `expr BETWEEN lower_bound AND upper_bound` is equivalent to `expr >= lower_bound AND expr <= upper_bound`. The collations used for comparison with lower_bound and upper_bound are independent and need not be identical, but both must be compatible with the collation of expr.
- **Beginner-Friendly Explanation:** Snowflake works like standard BETWEEN but pays attention to how text is compared (collation) and how dates are converted.

**Annotated Code Examples:**

```sql
-- Snowflake: Using COLLATE with BETWEEN
SELECT 'm' BETWEEN COLLATE('A', 'lower') AND COLLATE('Z', 'lower');
-- Returns TRUE

-- Snowflake: Date casting warning
SELECT * FROM orders
WHERE timestamp_column BETWEEN '2025-04-30' AND '2025-04-31';
-- DATE values are cast to TIMESTAMP with time 00:00:00
-- To include all of April 31, use '2025-05-01' as upper bound
```

**References:**

- Snowflake: [NOT] BETWEEN - https://docs.snowflake.com/ja/sql-reference/functions/between


### 4. Performance & Optimization

#### 4.1 Index Usage with BETWEEN

**Core Definitions:**

- **Core Definition:** BETWEEN can efficiently use B-tree indexes when the tested column is indexed, because it is equivalent to two range comparisons (`>=` and `<=`).
- **Technical Definition:** Unlike leading-wildcard LIKE patterns, BETWEEN conditions on indexed columns allow the query optimizer to perform an index range scan. The database can seek to the lower bound and scan until the upper bound.
- **Beginner-Friendly Explanation:** If you have an index on a column, BETWEEN can use it to quickly find the matching rows without scanning the entire table.

**Purposes:**

- To write efficient range queries that leverage indexes
- To understand when BETWEEN will and won't use an index
- To optimize query performance for large datasets

**Annotated Code Examples:**

```sql
-- Create an index on the salary column
CREATE INDEX idx_salary ON employees(salary);

-- This query can use the index for a range scan
SELECT * FROM employees WHERE salary BETWEEN 50000 AND 100000;

-- The equivalent query using >= and <= also uses the index
SELECT * FROM employees WHERE salary >= 50000 AND salary <= 100000;
```

**Real-World Cases:**

- **Large tables:** Efficiently filtering millions of rows by date range.
- **Reporting:** Fast generation of range-based reports.

**References:**

- PostgreSQL: Comparison Functions and Operators - https://www.postgresql.org/docs/17/functions-comparison.html
- Microsoft Learn: BETWEEN (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/between-transact-sql


#### 4.2 BETWEEN vs. Equivalent Conditions

**Core Definitions:**

- **Core Definition:** BETWEEN is functionally equivalent to using `>=` and `<=` with AND, but BETWEEN is more concise and may be optimized differently by some database engines.
- **Technical Definition:** The SQL standard defines `a BETWEEN x AND y` as equivalent to `a >= x AND a <= y`. The query optimizer may treat them identically or may optimize BETWEEN more aggressively. In Oracle, `expr1` may be evaluated more than once in SQL, but only once in PL/SQL.
- **Beginner-Friendly Explanation:** BETWEEN is cleaner to write than two separate conditions, but they do the same thing. Some databases might optimize BETWEEN better.

**Annotated Code Examples:**

```sql
-- Three equivalent ways to write the same condition
SELECT * FROM employees WHERE salary BETWEEN 50000 AND 100000;
SELECT * FROM employees WHERE salary >= 50000 AND salary <= 100000;
SELECT * FROM employees WHERE salary >= 50000 AND 100000 >= salary;
```

**Real-World Cases:**

- **Code readability:** BETWEEN is more readable for range conditions.
- **Query optimization:** Understanding when BETWEEN and equivalent conditions perform differently.

**References:**

- Oracle: BETWEEN Condition - https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/BETWEEN-Condition.html
- SAP Help Portal: BETWEEN Search Condition - https://help.sap.com/docs/SAP_SQL_Anywhere/93079d4ba8e44920ae63ffb4def91f5b/816aaf436ce21014bf71db9864f64ec4.html


### 5. NULL Handling with BETWEEN

#### 5.1 Three-Valued Logic with BETWEEN

**Core Definitions:**

- **Core Definition:** BETWEEN follows SQL's three-valued logic: it returns TRUE, FALSE, or UNKNOWN. If any operand is NULL, the result may be UNKNOWN.
- **Technical Definition:** The BETWEEN condition evaluates as `test_expression >= begin_expression AND test_expression <= end_expression`. If either comparison returns FALSE, the overall expression returns FALSE. Otherwise, if either is UNKNOWN, the result is UNKNOWN. If `test_expression` is NULL, the result is NULL (UNKNOWN).
- **Beginner-Friendly Explanation:** If any of the three values (the test value, the lower bound, or the upper bound) is NULL, you can't say for sure whether the test value is in the range—so the result is "unknown," which means the row is not returned.

**Purposes:**

- To understand why BETWEEN queries may exclude rows with NULL values
- To handle NULLs appropriately in range queries
- To write queries that account for missing data

**Annotated Code Examples:**

```sql
-- Demonstrate NULL handling with BETWEEN
SELECT 
    5 BETWEEN 1 AND 10 AS non_null_case,     -- TRUE
    5 BETWEEN NULL AND 10 AS null_lower,     -- UNKNOWN (NULL)
    5 BETWEEN 1 AND NULL AS null_upper,      -- UNKNOWN (NULL)
    NULL BETWEEN 1 AND 10 AS null_test;      -- UNKNOWN (NULL)
```

**Expected Output:**

| non_null_case | null_lower | null_upper | null_test |
|---------------|------------|------------|-----------|
| TRUE          | NULL       | NULL       | NULL      |

**Explanation:** When any operand is NULL, the result is NULL (representing UNKNOWN). In a WHERE clause, rows with UNKNOWN conditions are not returned.

**Real-World Cases:**

- **Data quality:** Identifying rows with missing data in range queries.
- **Application logic:** Handling cases where bounds or test values may be NULL.

**References:**

- Microsoft Learn: BETWEEN (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/between-transact-sql
- Oracle: BETWEEN Condition - https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/BETWEEN-Condition.html
- SAP Help Portal: BETWEEN Search Condition - https://help.sap.com/docs/SAP_SQL_Anywhere/93079d4ba8e44920ae63ffb4def91f5b/816aaf436ce21014bf71db9864f64ec4.html


## Summary Tables

### BETWEEN Syntax Across Databases

| Database | Syntax | Key Notes |
|----------|--------|-----------|
| ANSI SQL | `expr BETWEEN lower AND upper` | Inclusive endpoints; core feature |
| PostgreSQL | `expr BETWEEN lower AND upper` | Also supports `BETWEEN SYMMETRIC` |
| SQL Server | `test_expression [NOT] BETWEEN begin AND end` | Returns TRUE/FALSE/UNKNOWN |
| MySQL | `expr BETWEEN min AND max` | Returns 1/0; CAST() recommended for dates |
| Oracle | `expr1 BETWEEN expr2 AND expr3` | Equivalent to `expr2 <= expr1 AND expr1 <= expr3` |
| Snowflake | `expr [NOT] BETWEEN lower_bound AND upper_bound` | Collation-sensitive; date casting caution |

### BETWEEN vs. Equivalent Operators

| Operator | Inclusive? | Syntax | Notes |
|----------|-----------|--------|-------|
| BETWEEN | Yes (both ends) | `col BETWEEN 10 AND 20` | Concise, readable |
| `>=` AND `<=` | Yes (both ends) | `col >= 10 AND col <= 20` | More verbose but explicit |
| `>` AND `<` | No (exclusive) | `col > 10 AND col < 20` | For exclusive ranges |

### BETWEEN NULL Behavior

| Test Value | Lower Bound | Upper Bound | Result |
|------------|-------------|-------------|--------|
| 5 | 1 | 10 | TRUE |
| 5 | NULL | 10 | UNKNOWN |
| 5 | 1 | NULL | UNKNOWN |
| NULL | 1 | 10 | UNKNOWN |
| 5 | 10 | 1 | FALSE (empty range) |

---

## References

- Microsoft Learn: BETWEEN (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/between-transact-sql
- PostgreSQL: Comparison Functions and Operators - https://www.postgresql.org/docs/17/functions-comparison.html
- Oracle: BETWEEN Condition - https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/BETWEEN-Condition.html
- MySQL: Comparison Functions and Operators - https://downloads.mysql.com/docs/refman-5.6-en.a4.pdf
- Snowflake: [NOT] BETWEEN - https://docs.snowflake.com/ja/sql-reference/functions/between
- SAP Help Portal: BETWEEN Search Condition - https://help.sap.com/docs/SAP_SQL_Anywhere/93079d4ba8e44920ae63ffb4def91f5b/816aaf436ce21014bf71db9864f64ec4.html
- Programiz: SQL BETWEEN Operator - https://www.programiz.com/sql/between-operator
- centron: SQL BETWEEN Operator Guide - https://www.centron.de/tutorials/sql-between-operator-guide