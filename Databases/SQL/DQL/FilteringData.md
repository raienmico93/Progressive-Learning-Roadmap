# Comprehensive Programming Cheat Sheet: SQL Filtering Data

## Topic Overview

### Definitions

**Core Definition**
Filtering data is the process of using the `WHERE` clause and its associated predicates—comparison operators, logical operators, range and set membership tests, pattern matching, null checks, and subquery operators—to restrict the rows returned by a `SELECT`, `UPDATE`, or `DELETE` statement to only those that satisfy a specified condition.

**Technical Definition**
The `WHERE` clause contains a Boolean predicate that is evaluated for every row produced by the `FROM` clause. Only rows for which the predicate evaluates to **`TRUE`** are retained; rows evaluating to `FALSE` or `UNKNOWN` are discarded. SQL uses **three-valued logic**: comparisons involving `NULL` produce `UNKNOWN` rather than `FALSE`, and `WHERE` treats `UNKNOWN` the same as `FALSE` for the purpose of row retention. The predicate can combine comparisons (`=`, `<>`, `<`, `>`, `<=`, `>=`), logical operators (`AND`, `OR`, `NOT`), range tests (`BETWEEN`), set membership (`IN`), pattern matching (`LIKE`, `ILIKE`, `SIMILAR TO`, regular expressions), null checks (`IS NULL`, `IS NOT NULL`, `IS DISTINCT FROM`), and subquery operators (`EXISTS`, `ANY`, `ALL`). `WHERE` is logically evaluated **after `FROM` and before `GROUP BY`**, which is why it cannot reference aggregate functions or `SELECT` aliases.

**Beginner-Friendly Explanation**
Filtering is how you tell the database “only show me the rows I care about.” Without a `WHERE` clause, a query returns every row in the table. With a `WHERE` clause, you write a condition—like “salary greater than 50,000” or “country is not null”—and the database checks each row against that condition. Rows that pass are kept; rows that fail are quietly dropped. The tricky part is `NULL`: it means “unknown,” not “empty” or “zero,” and SQL handles it with a third truth value called `UNKNOWN`. Understanding this three-valued logic is the key to avoiding the classic “my query returned zero rows and I don’t know why” bug.

### Key Characteristics

- **Three-valued logic is foundational**: SQL predicates evaluate to `TRUE`, `FALSE`, or `UNKNOWN`. `WHERE` keeps only `TRUE`; both `FALSE` and `UNKNOWN` are discarded.
- **`NULL` is not a value**: `NULL = NULL` is `UNKNOWN`, not `TRUE`. Use `IS NULL` / `IS NOT NULL` for null checks, and `IS DISTINCT FROM` (or vendor equivalents) for null-safe comparison.
- **`NOT IN` with `NULL` is a silent trap**: If a subquery returns any `NULL`, `NOT IN` returns no rows at all, because `x <> NULL` is `UNKNOWN` and `WHERE` discards `UNKNOWN` rows.
- **Logical precedence is fixed but vendor-extended**: `NOT` binds tighter than `AND`, and `AND` binds tighter than `OR`. Parentheses override precedence and should be used liberally for readability.
- **Pattern matching is vendor-divergent**: `LIKE` is standard; `ILIKE` is PostgreSQL-specific for case-insensitive matching; `SIMILAR TO` blends `LIKE` with SQL:1999 regular expressions; POSIX regular expressions use `~` and `~*`.

### Prerequisites

- A running RDBMS (PostgreSQL, MySQL, or SQL Server) and a client connection.
- At least one table with data to filter.
- Familiarity with the `SELECT` statement and basic data types.
- Awareness of `NULL` and its special role in SQL comparisons.

### Related Programming Areas

- Data Manipulation Language (DML) – `UPDATE` and `DELETE` also use `WHERE` to target rows.
- Query optimisation – indexes, sargability, and predicate pushdown determine filter performance.
- Data quality – filtering is how you isolate invalid, incomplete, or anomalous rows.
- Application development – parameterised queries use `WHERE` to implement search and access control.

---

## Core Filtering: `WHERE` Clause and Boolean Expressions

### 1. `WHERE` Clause

#### Definitions

**Core Definition**
The `WHERE` clause filters rows by evaluating a Boolean predicate for each row and retaining only those for which the predicate is `TRUE`.

**Technical Definition**
`WHERE condition` appears after `FROM` and before `GROUP BY`. The condition is any expression that evaluates to a Boolean value. Only rows for which the condition is `TRUE` are passed to subsequent clauses; rows evaluating to `FALSE` or `UNKNOWN` are discarded. Because `WHERE` runs before `GROUP BY` and `SELECT`, it cannot reference aggregate functions or column aliases defined in the `SELECT` list.

**Beginner-Friendly Explanation**
`WHERE` is the row filter. It sits between reading the table and doing anything else, and it decides which rows survive.

#### Purposes

- To restrict a query’s result set to rows that meet a business or analytical condition.
- To reduce the volume of data processed by later clauses, improving performance.
- To target specific rows for `UPDATE` or `DELETE` operations.

#### Syntax Rules and Structure

```sql
SELECT select_list
FROM table_name
WHERE condition;
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `condition` | A Boolean expression evaluated for each row. |
| Retained rows | Only rows where `condition` is `TRUE`. |
| Discarded rows | Rows where `condition` is `FALSE` or `UNKNOWN`. |

**Syntax rules:**

- The condition must evaluate to a Boolean value.
- Aggregate functions are not allowed in `WHERE`.
- `SELECT` aliases are not visible in `WHERE`.
- `WHERE` is optional; omitting it returns all rows.

**Constraints and limitations:**

- A `WHERE` condition that references a column without an index may force a full table scan.
- `WHERE` cannot filter groups; use `HAVING` for that purpose.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Basic WHERE filtering

```sql
-- Step 1: Create a table and insert sample data.
CREATE TABLE employees (
    emp_id   int PRIMARY KEY,
    emp_name text,
    dept_id  int,
    salary   numeric
);
INSERT INTO employees VALUES
    (1, 'Ammar', 1, 80000),
    (2, 'Riya',  2, 65000),
    (3, 'Sahil', 1, 72000),
    (4, 'Priya', 3, 90000);

-- Step 2: Filter to the Data Analytics department.
SELECT emp_name
FROM employees
WHERE dept_id = 1;
```

**Expected output:**

```text
 emp_name
----------
 Ammar
 Sahil
(2 rows)
```

**Why this output appears**: The `WHERE dept_id = 1` predicate evaluates to `TRUE` for Ammar and Sahil, so those rows are retained; Riya and Priya evaluate to `FALSE` and are dropped.

#### Real-World Cases

- **Order processing**: `WHERE status = 'pending'` selects orders awaiting fulfilment.
- **Access control**: `WHERE tenant_id = $1` ensures a user only sees their own data.
- **Data quality**: `WHERE email IS NULL` identifies incomplete records.

---

### 2. Boolean Expressions

#### Definitions

**Core Definition**
A Boolean expression is any expression that evaluates to `TRUE`, `FALSE`, or `UNKNOWN`, formed from comparisons, logical operators, and predicates.

**Technical Definition**
A Boolean expression combines comparison predicates (`=`, `<>`, `<`, `>`, `<=`, `>=`), logical operators (`AND`, `OR`, `NOT`), and specialised predicates (`BETWEEN`, `IN`, `LIKE`, `IS NULL`, `EXISTS`, etc.). SQL’s three-valued logic means that `NULL` propagates through these expressions: `NULL = 1` is `UNKNOWN`, `NOT UNKNOWN` is `UNKNOWN`, and `TRUE OR UNKNOWN` is `TRUE`.

**Beginner-Friendly Explanation**
A Boolean expression is a yes/no/unknown question you ask each row. The database answers it and keeps the row only if the answer is an unambiguous “yes.”

#### Purposes

- To combine multiple conditions with `AND`, `OR`, and `NOT`.
- To express complex business rules as a single predicate.
- To make filtering logic explicit and readable.

#### Syntax Rules and Structure

```sql
WHERE comparison_1 AND comparison_2
   OR NOT comparison_3;
```

**Component breakdown:**

| Element | Example |
|---------|---------|
| Comparison | `salary > 50000` |
| Logical combination | `dept_id = 1 AND salary > 70000` |
| Negation | `NOT (status = 'cancelled')` |
| Parenthesised group | `(a = 1 OR b = 2) AND c = 3` |

**Syntax rules:**

- Use parentheses to group expressions and override precedence.
- `NULL` in any comparison yields `UNKNOWN`, which `WHERE` treats like `FALSE` for row retention.

**Constraints and limitations:**

- Overly complex Boolean expressions can be hard to optimise and debug.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Combining conditions with AND and OR

```sql
-- Step 1: Find employees in dept 1 OR earning more than 85k.
SELECT emp_name, dept_id, salary
FROM employees
WHERE dept_id = 1
   OR salary > 85000;
```

**Expected output:**

```text
 emp_name | dept_id | salary
----------+---------+--------
 Ammar    |       1 |  80000
 Sahil    |       1 |  72000
 Priya    |       3 |  90000
(3 rows)
```

**Why this output appears**: Ammar and Sahil satisfy `dept_id = 1`; Priya satisfies `salary > 85000`; Riya satisfies neither and is excluded.

#### Real-World Cases

- **Fraud detection**: `WHERE amount > 10000 AND country NOT IN ('US', 'CA')`.
- **Customer segmentation**: `WHERE (age BETWEEN 18 AND 35) AND loyalty_score > 80`.

---

## Comparison Operators

### 3. `=` (Equal To)

#### Definitions

**Core Definition**
`=` tests whether two expressions are equal, returning `TRUE` if they are and `FALSE` otherwise (or `UNKNOWN` if either operand is `NULL`).

**Technical Definition**
The equality operator compares two values using the type’s equality semantics. For most types, `NULL = NULL` returns `UNKNOWN`, not `TRUE`. String comparison is collation-dependent and may be case-sensitive or case-insensitive.

**Beginner-Friendly Explanation**
`=` asks “are these the same?” It returns true only when both sides are non-null and equal.

#### Purposes

- To match exact values (IDs, codes, statuses).
- To join or filter on equality conditions.

#### Syntax Rules and Structure

```sql
WHERE column = value
```

**Syntax rules:**

- `= NULL` never matches; use `IS NULL` instead.
- String comparison depends on collation.

**Constraints and limitations:**

- Floating-point equality can be unreliable due to precision; use tolerance ranges instead.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Equality filter

```sql
SELECT emp_name FROM employees WHERE dept_id = 2;
```

**Expected output:**

```text
 emp_name
----------
 Riya
(1 row)
```

**Why this output appears**: Only Riya has `dept_id = 2`.

---

### 4. `<>` and `!=` (Not Equal To)

#### Definitions

**Core Definition**
`<>` (standard) and `!=` (widely supported alias) test whether two expressions are not equal.

**Technical Definition**
Both operators return `TRUE` when the operands are non-null and unequal, and `UNKNOWN` when either operand is `NULL`. `!=` is accepted by PostgreSQL, MySQL, and SQL Server but is not in the SQL standard; `<>` is the portable form.

**Beginner-Friendly Explanation**
`<>` asks “are these different?” It returns true only when both sides are non-null and unequal.

#### Purposes

- To exclude specific values or ranges.
- To detect changes or differences.

#### Syntax Rules and Structure

```sql
WHERE column <> value
WHERE column != value   -- non-standard but widely supported
```

**Syntax rules:**

- `<>` is preferred for portability.
- `<> NULL` never matches; use `IS NOT NULL`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Not-equal filter

```sql
SELECT emp_name FROM employees WHERE dept_id <> 1;
```

**Expected output:**

```text
 emp_name
----------
 Riya
 Priya
(2 rows)
```

**Why this output appears**: Riya (`dept_id = 2`) and Priya (`dept_id = 3`) are not in department 1, so they are retained.

---

### 5. `<`, `>`, `<=`, `>=` (Relational Operators)

#### Definitions

**Core Definition**
These operators compare two values by magnitude: `<` (less than), `>` (greater than), `<=` (less than or equal), and `>=` (greater than or equal).

**Technical Definition**
Relational operators use the type’s ordering semantics. For `NULL`, all comparisons return `UNKNOWN`. For strings, ordering depends on collation; for dates, chronological order is used.

**Beginner-Friendly Explanation**
These operators ask “is this smaller, larger, or equal to that?” They are used for ranges, thresholds, and dates.

#### Purposes

- To filter numeric thresholds (salary > 50000).
- To filter date ranges (order_date >= '2025-01-01').
- To compare string values lexicographically.

#### Syntax Rules and Structure

```sql
WHERE column < value
WHERE column >= value
```

**Syntax rules:**

- `<=` and `>=` include the boundary value.
- `NULL` comparisons return `UNKNOWN`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Threshold filter

```sql
SELECT emp_name, salary FROM employees WHERE salary >= 80000;
```

**Expected output:**

```text
 emp_name | salary
----------+--------
 Ammar    |  80000
 Priya    |  90000
(2 rows)
```

**Why this output appears**: Ammar’s salary equals 80,000 (included by `>=`) and Priya’s exceeds it.

---

## Logical Operators and Precedence

### 6. `AND`

#### Definitions

**Core Definition**
`AND` combines two Boolean expressions and returns `TRUE` only if both are `TRUE`.

**Technical Definition**
`AND` has higher precedence than `OR` but lower precedence than `NOT`. Its truth table: `TRUE AND TRUE = TRUE`; `TRUE AND FALSE = FALSE`; `TRUE AND UNKNOWN = UNKNOWN`; `FALSE AND anything = FALSE`.

**Beginner-Friendly Explanation**
`AND` means “both conditions must be true.”

#### Purposes

- To require multiple conditions simultaneously.
- To narrow a result set by adding constraints.

#### Syntax Rules and Structure

```sql
WHERE condition1 AND condition2
```

**Syntax rules:**

- Parenthesise complex `AND`/`OR` combinations for clarity.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: AND filter

```sql
SELECT emp_name FROM employees WHERE dept_id = 1 AND salary > 75000;
```

**Expected output:**

```text
 emp_name
----------
 Ammar
(1 row)
```

**Why this output appears**: Ammar is the only employee who is both in department 1 and earns more than 75,000.

---

### 7. `OR`

#### Definitions

**Core Definition**
`OR` combines two Boolean expressions and returns `TRUE` if at least one is `TRUE`.

**Technical Definition**
`OR` has lower precedence than `AND`. Its truth table: `FALSE OR FALSE = FALSE`; `FALSE OR UNKNOWN = UNKNOWN`; `TRUE OR anything = TRUE`.

**Beginner-Friendly Explanation**
`OR` means “at least one condition must be true.”

#### Purposes

- To broaden a result set by accepting alternative conditions.
- To implement “any of” logic.

#### Syntax Rules and Structure

```sql
WHERE condition1 OR condition2
```

**Syntax rules:**

- `AND` binds tighter than `OR`; use parentheses to force `OR` to evaluate first.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: OR filter

```sql
SELECT emp_name FROM employees WHERE dept_id = 1 OR salary > 85000;
```

**Expected output:**

```text
 emp_name
----------
 Ammar
 Sahil
 Priya
(3 rows)
```

**Why this output appears**: Ammar and Sahil satisfy the department condition; Priya satisfies the salary condition.

---

### 8. `NOT`

#### Definitions

**Core Definition**
`NOT` negates a Boolean expression: `TRUE` becomes `FALSE`, `FALSE` becomes `TRUE`, and `UNKNOWN` remains `UNKNOWN`.

**Technical Definition**
`NOT` has the highest precedence among the logical operators, binding more tightly than `AND` and `OR`.

**Beginner-Friendly Explanation**
`NOT` flips the answer.

#### Purposes

- To exclude rows that match a condition.
- To express “does not satisfy” logic.

#### Syntax Rules and Structure

```sql
WHERE NOT condition
```

**Syntax rules:**

- `NOT` with `UNKNOWN` yields `UNKNOWN`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: NOT filter

```sql
SELECT emp_name FROM employees WHERE NOT (dept_id = 1);
```

**Expected output:**

```text
 emp_name
----------
 Riya
 Priya
(2 rows)
```

**Why this output appears**: The negation excludes department 1 employees, leaving Riya and Priya.

---

### 9. Operator Precedence and Parentheses

#### Definitions

**Core Definition**
Operator precedence determines the order in which operators are evaluated; parentheses override precedence.

**Technical Definition**
The standard precedence order from highest to lowest among logical operators is: `NOT` → `AND` → `OR`. Comparison operators bind tighter than logical operators. Arithmetic and bitwise operators bind tighter than comparisons. MySQL’s precedence list places `NOT` above `AND`, and `AND` above `OR`. Oracle similarly evaluates `NOT` before `AND` and `AND` before `OR`. Parentheses can be used to override any precedence and are recommended even when not strictly required, for readability and to prevent subtle mistakes.

**Beginner-Friendly Explanation**
Precedence is the “order of operations” for logic. `AND` is evaluated before `OR`, so `A OR B AND C` means `A OR (B AND C)`. If you want `(A OR B) AND C`, you must use parentheses.

#### Purposes

- To control which conditions are evaluated first.
- To avoid logical errors caused by implicit precedence.
- To improve query readability.

#### Syntax Rules and Structure

```sql
WHERE (condition1 OR condition2) AND condition3
```

**Syntax rules:**

- Parentheses override default precedence.
- Adding parentheses for clarity has no performance penalty.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Precedence in action

```sql
-- Without parentheses: AND binds tighter than OR.
SELECT emp_name FROM employees WHERE dept_id = 2 OR dept_id = 1 AND salary > 75000;

-- With parentheses: OR is forced to evaluate first.
SELECT emp_name FROM employees WHERE (dept_id = 2 OR dept_id = 1) AND salary > 75000;
```

**Expected output:**

```text
-- First query: Riya (dept 2), Ammar (dept 1 and salary > 75000)
 emp_name
----------
 Ammar
 Riya
(2 rows)

-- Second query: only employees in dept 1 or 2 who also earn > 75000
 emp_name
----------
 Ammar
(1 row)
```

**Why this output appears**: The first query is interpreted as `dept_id = 2 OR (dept_id = 1 AND salary > 75000)`, so Riya qualifies unconditionally. The second query requires salary > 75000 for both departments, excluding Riya.

---

## Range and Set Membership

### 10. `BETWEEN`

#### Definitions

**Core Definition**
`BETWEEN a AND b` tests whether a value lies within an inclusive range.

**Technical Definition**
`x BETWEEN a AND b` is equivalent to `x >= a AND x <= b`. Both endpoints are included. `BETWEEN` is widely supported across PostgreSQL, MySQL, SQL Server, and Oracle.

**Beginner-Friendly Explanation**
`BETWEEN` means “within this range, including both ends.”

#### Purposes

- To filter numeric, date, or string ranges.
- To replace two comparisons with one readable predicate.

#### Syntax Rules and Structure

```sql
WHERE column BETWEEN lower AND upper
WHERE column NOT BETWEEN lower AND upper
```

**Syntax rules:**

- Both bounds are inclusive.
- `BETWEEN` is syntactic sugar for `>=` and `<=`.

**Constraints and limitations:**

- With timestamps, `BETWEEN '2026-10-01' AND '2026-10-31'` may miss rows on the last day because the upper bound is midnight at the start of the day.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: BETWEEN filter

```sql
SELECT emp_name, salary FROM employees WHERE salary BETWEEN 70000 AND 85000;
```

**Expected output:**

```text
 emp_name | salary
----------+--------
 Ammar    |  80000
 Sahil    |  72000
(2 rows)
```

**Why this output appears**: Both Ammar (80,000) and Sahil (72,000) fall within the inclusive range; Riya (65,000) and Priya (90,000) do not.

---

### 11. `IN` and `NOT IN`

#### Definitions

**Core Definition**
`IN (list)` tests whether a value equals any value in a list or subquery; `NOT IN` tests whether it equals none of them.

**Technical Definition**
`x IN (a, b, c)` is equivalent to `x = a OR x = b OR x = c`. `IN` supports literals, expressions, and subqueries. `NOT IN` is equivalent to `x <> a AND x <> b AND x <> c`. Both are supported across major RDBMSs.

**Beginner-Friendly Explanation**
`IN` means “is this one of these values?” `NOT IN` means “is this none of these values?”

#### Purposes

- To match against a discrete list of values.
- To filter by the results of a subquery.
- To avoid long chains of `OR` conditions.

#### Syntax Rules and Structure

```sql
WHERE column IN (value1, value2, ...)
WHERE column IN (subquery)
WHERE column NOT IN (value1, value2, ...)
```

**Syntax rules:**

- `NOT IN` with a subquery that returns any `NULL` yields no rows at all, because `x <> NULL` is `UNKNOWN`.
- `IN` is equivalent to `= ANY`.

**Constraints and limitations:**

- Large `IN` lists can be inefficient; consider a temporary table or `EXISTS` for very large sets.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: IN filter

```sql
SELECT emp_name FROM employees WHERE dept_id IN (1, 3);
```

**Expected output:**

```text
 emp_name
----------
 Ammar
 Sahil
 Priya
(3 rows)
```

**Why this output appears**: Ammar and Sahil are in department 1, and Priya is in department 3; Riya (department 2) is excluded.

##### Example 2: The NOT IN NULL trap

```sql
-- Step 1: Create a subquery that returns a NULL.
SELECT emp_name
FROM employees
WHERE dept_id NOT IN (SELECT dept_id FROM employees WHERE dept_id IS NULL);
```

**Expected output:**

```text
 emp_name
----------
(0 rows)
```

**Why this output appears**: The subquery returns `NULL`. `dept_id NOT IN (NULL)` expands to `dept_id <> NULL`, which is `UNKNOWN` for every row, so `WHERE` discards all rows.

---

## Pattern Matching

### 12. `LIKE`

#### Definitions

**Core Definition**
`LIKE` tests whether a string matches a pattern using the wildcards `%` (any sequence of zero or more characters) and `_` (any single character).

**Technical Definition**
`string LIKE pattern` returns `TRUE` if the string matches the entire pattern. `%` matches any sequence of characters, `_` matches exactly one character. `LIKE` is case-sensitive in PostgreSQL; case sensitivity depends on column collation in MySQL and SQL Server.

**Beginner-Friendly Explanation**
`LIKE` is simple pattern matching: `%` means “anything,” `_` means “one character.”

#### Purposes

- To search for prefixes, suffixes, or substrings.
- To validate simple string formats.

#### Syntax Rules and Structure

```sql
WHERE column LIKE 'pattern'
WHERE column NOT LIKE 'pattern'
```

**Syntax rules:**

- `%` and `_` are wildcards; literal occurrences must be escaped.
- The default escape character is backslash in PostgreSQL and MySQL, but SQL Server uses no default and requires `ESCAPE` or bracket escaping.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: LIKE pattern

```sql
SELECT emp_name FROM employees WHERE emp_name LIKE 'A%';
```

**Expected output:**

```text
 emp_name
----------
 Ammar
(1 row)
```

**Why this output appears**: `'A%'` matches any string starting with `A`, so Ammar matches; the others do not.

---

### 13. `ILIKE` (PostgreSQL)

#### Definitions

**Core Definition**
`ILIKE` is PostgreSQL’s case-insensitive version of `LIKE`.

**Technical Definition**
`ILIKE` performs case-insensitive pattern matching according to the active locale. It is a PostgreSQL extension, not part of the SQL standard.

**Beginner-Friendly Explanation**
`ILIKE` is `LIKE` that ignores upper/lower case.

#### Purposes

- To search case-insensitively without wrapping columns in `LOWER()`.

#### Syntax Rules and Structure

```sql
WHERE column ILIKE 'pattern'
```

**Syntax rules:**

- `ILIKE` is PostgreSQL-specific; MySQL and SQL Server require `LOWER()` or collation changes.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: ILIKE filter

```sql
SELECT emp_name FROM employees WHERE emp_name ILIKE 'a%';
```

**Expected output:**

```text
 emp_name
----------
 Ammar
(1 row)
```

**Why this output appears**: `ILIKE` matches regardless of case, so `'a%'` matches `'Ammar'`.

---

### 14. `SIMILAR TO`

#### Definitions

**Core Definition**
`SIMILAR TO` matches a string against an SQL:1999 regular expression pattern that blends `LIKE` wildcards with POSIX-style metacharacters.

**Technical Definition**
`SIMILAR TO` returns `TRUE` if the pattern matches the entire string. It supports `|` for alternation, `*` for zero-or-more repetition, `+` for one-or-more, `?` for optional, and `{}` for repetition counts, in addition to `%` and `_`. It is supported by PostgreSQL and is part of the SQL standard.

**Beginner-Friendly Explanation**
`SIMILAR TO` is `LIKE` plus regular-expression features like alternation and repetition.

#### Purposes

- To express more complex string patterns than `LIKE` allows.
- To stay within the SQL standard rather than using vendor-specific regex operators.

#### Syntax Rules and Structure

```sql
WHERE column SIMILAR TO 'pattern'
WHERE column NOT SIMILAR TO 'pattern'
```

**Syntax rules:**

- `|` means OR; `*`, `+`, `?`, `{}` control repetition.
- Escaping uses `ESCAPE` with a backslash by default.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SIMILAR TO with alternation

```sql
SELECT emp_name FROM employees WHERE emp_name SIMILAR TO '(Ammar|Riya)';
```

**Expected output:**

```text
 emp_name
----------
 Ammar
 Riya
(2 rows)
```

**Why this output appears**: The `(Ammar|Riya)` alternation matches either name.

---

### 15. Regular Expressions (PostgreSQL)

#### Definitions

**Core Definition**
PostgreSQL provides POSIX regular-expression operators (`~`, `~*`, `!~`, `!~*`) for powerful pattern matching.

**Technical Definition**
`~` matches a POSIX regex case-sensitively; `~*` matches case-insensitively; `!~` and `!~*` negate them. These are PostgreSQL-specific and generally faster than `SIMILAR TO` for complex patterns.

**Beginner-Friendly Explanation**
Regular expressions are the most powerful pattern-matching tool, using metacharacters like `.`, `*`, `+`, `[]`, and `{}`.

#### Purposes

- To validate complex formats (emails, phone numbers, postal codes).
- To extract and manipulate text with fine-grained control.

#### Syntax Rules and Structure

```sql
WHERE column ~ 'regex'
WHERE column ~* 'regex'
WHERE column !~ 'regex'
```

**Syntax rules:**

- Use `ESCAPE` to treat regex metacharacters literally when needed.
- Regex patterns can be expensive; avoid accepting untrusted patterns without timeouts.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Regex filter

```sql
SELECT emp_name FROM employees WHERE emp_name ~ '^[A-Z][a-z]+$';
```

**Expected output:**

```text
 emp_name
----------
 Ammar
 Riya
 Sahil
 Priya
(4 rows)
```

**Why this output appears**: The regex requires an uppercase letter followed by lowercase letters, which all four names satisfy.

---

### 16. Escaping Special Characters

#### Definitions

**Core Definition**
Escaping special characters allows literal `%`, `_`, or regex metacharacters to be matched as ordinary characters.

**Technical Definition**
In `LIKE`, the `ESCAPE` clause specifies an escape character (often backslash) that disables the special meaning of the following character. In PostgreSQL, the default escape is backslash; SQL Server has no default and requires `ESCAPE` or bracket escaping (`[_]`). In `SIMILAR TO` and regex, backslash also escapes metacharacters.

**Beginner-Friendly Explanation**
If you want to search for a literal percent sign, you cannot just write `%`; you must escape it—e.g., `LIKE '%\%%' ESCAPE '\'`.

#### Purposes

- To search for literal wildcard characters.
- To prevent pattern metacharacters from being interpreted.

#### Syntax Rules and Structure

```sql
WHERE column LIKE 'pattern' ESCAPE '!'
WHERE column ~ '\.'
```

**Syntax rules:**

- `ESCAPE ''` disables the escape mechanism entirely.
- SQL Server allows bracket escaping: `LIKE '20[_]%'` matches a literal underscore.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Escaping a literal underscore

```sql
SELECT 'a_b' LIKE 'a!_b' ESCAPE '!';
```

**Expected output:**

```text
 ?column?
----------
 t
```

**Why this output appears**: The `ESCAPE '!'` clause makes `!` the escape character, so `!_` matches a literal underscore rather than any single character.

---

## Null and Distinct Checks

### 17. `IS NULL` and `IS NOT NULL`

#### Definitions

**Core Definition**
`IS NULL` tests whether a value is the SQL null marker; `IS NOT NULL` tests the opposite.

**Technical Definition**
`IS NULL` is the only portable way to test for `NULL`. `column = NULL` never matches because `NULL = NULL` is `UNKNOWN`. `IS NULL` returns `TRUE` for null and `FALSE` for non-null; `IS NOT NULL` returns the inverse.

**Beginner-Friendly Explanation**
`IS NULL` asks “is this value missing?” It is the correct way to find nulls.

#### Purposes

- To find rows with missing or unknown values.
- To filter out incomplete records.
- To complement `IS NOT NULL` for data-quality checks.

#### Syntax Rules and Structure

```sql
WHERE column IS NULL
WHERE column IS NOT NULL
```

**Syntax rules:**

- Never use `= NULL` or `<> NULL`.
- `IS NULL` works for all data types.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Finding nulls

```sql
SELECT emp_name FROM employees WHERE manager_id IS NULL;
```

**Expected output:**

```text
 emp_name
----------
 Sahil
(1 row)
```

**Why this output appears**: Only Sahil has a `NULL` manager_id; the `IS NULL` predicate matches that row.

---

### 18. `IS DISTINCT FROM`

#### Definitions

**Core Definition**
`IS DISTINCT FROM` is a null-safe comparison that returns `TRUE` if two values are different, treating `NULL` as a value comparable to itself.

**Technical Definition**
`a IS DISTINCT FROM b` returns `TRUE` if `a` and `b` are different (including one `NULL` and one non-`NULL`), and `FALSE` if they are the same (including both `NULL`). `IS NOT DISTINCT FROM` is the null-safe equality counterpart. PostgreSQL, SQL Server 2022+, and IBM DB2 support this syntax; MySQL uses `<=>`; SQL Server before 2022 requires `(a = b OR (a IS NULL AND b IS NULL))`.

**Beginner-Friendly Explanation**
`IS DISTINCT FROM` is “not equal, but null-friendly.” Two `NULL`s count as the same, not as unknown.

#### Purposes

- To compare nullable columns without special-casing `NULL`.
- To detect changes in change-data-capture and audit queries.

#### Syntax Rules and Structure

```sql
WHERE a IS DISTINCT FROM b
WHERE a IS NOT DISTINCT FROM b   -- null-safe equality
```

**Syntax rules:**

- Portable to PostgreSQL, SQL Server 2022+, and DB2.
- MySQL equivalent: `a <=> b` for null-safe equality.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Null-safe comparison

```sql
SELECT NULL IS DISTINCT FROM NULL AS both_null,
       1 IS DISTINCT FROM NULL AS one_null;
```

**Expected output:**

```text
 both_null | one_null
-----------+----------
 f         | t
(1 row)
```

**Why this output appears**: Two `NULL`s are not distinct (they are “the same” in the null-safe sense), while `1` and `NULL` are distinct.

---

## Subquery Operators

### 19. `EXISTS`

#### Definitions

**Core Definition**
`EXISTS (subquery)` returns `TRUE` if the subquery returns at least one row, and `FALSE` otherwise.

**Technical Definition**
`EXISTS` evaluates the subquery only far enough to determine whether any row is returned; it does not need to fetch the full result. It is typically used with a correlated subquery and is often more efficient than `IN` for existence checks.

**Beginner-Friendly Explanation**
`EXISTS` asks “does this subquery return anything at all?”

#### Purposes

- To test for the existence of related rows.
- To implement semi-joins without duplicate rows.

#### Syntax Rules and Structure

```sql
WHERE EXISTS (subquery)
WHERE NOT EXISTS (subquery)
```

**Syntax rules:**

- The subquery’s select list is irrelevant; `SELECT 1` is a common convention.
- `EXISTS` short-circuits on the first matching row.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: EXISTS semi-join

```sql
SELECT c.name
FROM customers AS c
WHERE EXISTS (
    SELECT 1 FROM orders AS o WHERE o.customer_id = c.id
);
```

**Expected output:**

```text
 name
------
 alice
(1 row)
```

**Why this output appears**: Only Alice has at least one matching order; `EXISTS` returns `TRUE` for her and `FALSE` for the others.

---

### 20. `ANY`

#### Definitions

**Core Definition**
`x operator ANY (subquery)` returns `TRUE` if the comparison is true for at least one row returned by the subquery.

**Technical Definition**
`ANY` (and its synonym `SOME`) compares a value against each row of a subquery and returns `TRUE` if any comparison is true. `= ANY` is equivalent to `IN`. `x > ANY (subquery)` means “greater than at least one value.”

**Beginner-Friendly Explanation**
`ANY` means “is this true for at least one of the values?”

#### Purposes

- To compare against a set with a non-equality operator.
- To express “greater than the minimum” or “less than the maximum” semantics.

#### Syntax Rules and Structure

```sql
WHERE expression operator ANY (subquery)
```

**Syntax rules:**

- `= ANY` ≡ `IN`; `<> ALL` ≡ `NOT IN`.
- `ANY` short-circuits when a true comparison is found.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: ANY with a comparison

```sql
SELECT emp_name
FROM employees
WHERE salary > ANY (SELECT salary FROM employees WHERE dept_id = 2);
```

**Expected output:**

```text
 emp_name
----------
 Ammar
 Sahil
 Priya
(3 rows)
```

**Why this output appears**: The subquery returns Riya’s salary (65,000). Employees earning more than that are returned.

---

### 21. `ALL`

#### Definitions

**Core Definition**
`x operator ALL (subquery)` returns `TRUE` if the comparison is true for every row returned by the subquery.

**Technical Definition**
`ALL` compares a value against all rows of a subquery and returns `TRUE` only if every comparison is true. `x > ALL (subquery)` means “greater than every value,” equivalent to “greater than the maximum.”

**Beginner-Friendly Explanation**
`ALL` means “is this true for every single value?”

#### Purposes

- To express “greater than all” or “less than all” conditions.
- To implement universal quantification over a set.

#### Syntax Rules and Structure

```sql
WHERE expression operator ALL (subquery)
```

**Syntax rules:**

- `> ALL` is equivalent to `> MAX(...)`; `< ALL` is equivalent to `< MIN(...)`.
- `<> ALL` is equivalent to `NOT IN`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: ALL with a comparison

```sql
SELECT emp_name
FROM employees
WHERE salary > ALL (SELECT salary FROM employees WHERE dept_id = 2);
```

**Expected output:**

```text
 emp_name
----------
 Ammar
 Sahil
 Priya
(3 rows)
```

**Why this output appears**: The subquery returns 65,000 (Riya’s salary). Employees earning more than 65,000 satisfy `> ALL`.

---

## References

- SQL-Engineering-Handbook – WHERE Clause – https://github.com/theammarngp-makes/SQL-Engineering-Handbook/blob/e015eccf61bc205342b8aa6e35bc1145dab25d6a/01_Fundamentals/02_WHERE.md
- PostgreSQL Documentation – Pattern Matching – https://www.postgresql.org/docs/17/functions-matching.html
- Stack Overflow – Comparisons with NULLs in SQL – https://stackoverflow.com/questions/7367327/comparisons-with-nulls-in-sql/7367450
- SQLPro Studio – SQL NULL traps: = NULL, NOT IN, and sort order – https://www.sqlprostudio.com/blog/65-sql-null-traps-null-not-in-and-sort-order
- Microsoft Learn – Logical Operator Precedence – https://learn.microsoft.com/el-gr/previous-versions/sql/sql-server-2005/ms186992(v=sql.90)
- MySQL 8.4 Reference Manual – Operator Precedence – https://dev.mysql.com/doc/refman/8.4/en/operator-precedence.html
- Oracle Database SQL Language Reference – About SQL Conditions – https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/About-SQL-Conditions.html
- Apache Flink JIRA – FLINK-39424: Setting LIKE does not support default escape characters – https://issues.apache.org/jira/browse/FLINK-39424
- Stack Overflow – ANY/SOME and EXISTS equivalence – https://stackoverflow.com/revisions/2298576/2
- PostgreSQL 9.6 Documentation – EXISTS and IN subquery operators – http://www.postgresql.fastware.com/hubfs/_Global/Manuals/V9.6-PostgreSQL9-6-2Documentation.pdf
- Yisu – WHERE子句中比较运算符有哪些 – https://www.yisu.com/ask/743315.html
- DataCamp – SQL LIKE 模式匹配：实用指南与示例 – https://www.datacamp.com/tutorial/sql-like-pattern-matching