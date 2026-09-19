# SQL Filtering Data: WHERE Clause Syntax and Execution Order — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The `WHERE` clause is the SQL construct that filters rows returned by a query, specifying a Boolean condition that each row must satisfy to be included in the result set.

**Technical Definition:** Per the ISO/IEC 9075 standard, a `<where clause>` consists of the keyword `WHERE` followed by a `<search condition>`, which is a Boolean expression evaluated against each row of the table derived from the `<from clause>`. Only rows for which the search condition evaluates to `TRUE` are retained; rows evaluating to `FALSE` or `UNKNOWN` (due to `NULL`) are eliminated.

**Beginner-Friendly Explanation:** The `WHERE` clause is like a filter in a coffee machine — it lets through only the rows that meet your specific condition. If you ask for "employees earning more than $80,000," the `WHERE` clause checks each row and keeps only those that match.

### Key Characteristics

- **Boolean filtering:** Operates on a Boolean expression that must evaluate to `TRUE` for a row to be included.
- **Three-valued logic:** SQL uses `TRUE`, `FALSE`, and `UNKNOWN` (from `NULL` comparisons); only `TRUE` rows are returned.
- **Row-level operation:** Filters individual rows before any grouping or aggregation occurs.
- **Non-destructive:** Does not modify the underlying data; only restricts what is returned.
- **Clause-position-dependent:** Must appear after `FROM` and before `GROUP BY`, `HAVING`, and `ORDER BY` in the textual order of a `SELECT` statement.
- **Logical evaluation order:** Despite being written after `FROM`, the `WHERE` clause is logically evaluated second, immediately after the `FROM` clause produces source rows.

### Prerequisites

- **Basic SQL knowledge:** Familiarity with `SELECT` and `FROM` clauses.
- **Relational concepts:** Understanding of tables, rows, columns, and data types.
- **Comparison operators:** Knowledge of `=`, `<>`, `<`, `>`, `<=`, `>=`.
- **Logical operators:** Understanding of `AND`, `OR`, and `NOT`.
- **NULL awareness:** Recognition that `NULL` represents an unknown value and comparisons with `NULL` yield `UNKNOWN`.

### Related Programming Areas

- **Query Optimization:** `WHERE` clause predicates directly influence index usage and query performance.
- **Data Analysis:** Filtering is fundamental to extracting meaningful subsets for reporting and analytics.
- **Application Development:** Backend APIs rely heavily on `WHERE` clauses to retrieve specific records.
- **Database Administration:** DBAs use `WHERE` clauses to inspect and troubleshoot data.
- **Data Engineering (ETL):** Extract phases use `WHERE` to select incremental data.

### Core Concepts / Features

1. `WHERE` Clause Syntax
2. Logical Query Processing Order
3. Comparison Operators
4. Logical Operators (`AND`, `OR`, `NOT`)
5. `NULL` Handling and Three-Valued Logic
6. `BETWEEN`, `IN`, `LIKE`, and `IS NULL` Predicates
7. `WHERE` vs `HAVING`
8. Performance and Indexing Considerations

---

## 1. WHERE Clause Syntax

### Definitions

**Core Definition:** The `WHERE` clause specifies a search condition that filters rows from the table(s) referenced in the `FROM` clause, returning only rows where the condition evaluates to `TRUE`.

**Technical Definition:** In the SQL standard (ISO/IEC 9075-2), the `<where clause>` is defined as `WHERE <search condition>`, where `<search condition>` is a `<boolean value expression>`. The clause is part of the `<table expression>` and is applied to the result of the `<from clause>`. Rows for which the `<search condition>` evaluates to `TRUE` are retained; all others are discarded.

**Beginner-Friendly Explanation:** The `WHERE` clause is how you tell the database "only show me rows that match this rule." The rule is written as a condition, and the database checks every row against it.

### Purposes

- To restrict the rows returned by a query to those matching specified criteria.
- To reduce the volume of data transferred and processed.
- To enable precise data retrieval for reporting and analysis.
- To combine multiple conditions using logical operators for complex filtering.
- To leverage indexes for fast row retrieval.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT column1 [, column2, ...]
FROM table_name
WHERE search_condition;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `WHERE` | Keyword introducing the filtering condition |
| `search_condition` | A Boolean expression evaluated for each row |
| `search_condition` forms | Comparison, logical combination, predicate, or subquery |

**Search Condition Structure:**

```
search_condition ::=
    { comparison_predicate
    | between_predicate
    | in_predicate
    | like_predicate
    | null_predicate
    | exists_predicate
    | boolean_expression
    }
    [ { AND | OR } search_condition ]
```

**Comparison Predicate:**

```
expression comparison_operator expression
```

Where `comparison_operator` is one of: `=`, `<>`, `<`, `>`, `<=`, `>=`.

**Syntax Rules:**

- The `WHERE` clause must appear after the `FROM` clause and before `GROUP BY`, `HAVING`, `ORDER BY`, `LIMIT`, and `OFFSET`.
- The search condition must evaluate to a Boolean value (`TRUE`, `FALSE`, or `UNKNOWN`).
- String literals are enclosed in single quotes (`'text'`); numeric literals are unquoted.
- Identifiers can be qualified with table names or aliases (e.g., `employees.salary`).
- Parentheses can be used to group conditions and override default operator precedence.

**Constraints and Limitations:**

- The `WHERE` clause cannot reference column aliases defined in the `SELECT` clause in standard SQL (PostgreSQL, for example, disallows this).
- Aggregate functions (`SUM`, `AVG`, `COUNT`, etc.) cannot appear in the `WHERE` clause; they belong in the `HAVING` clause.
- `NULL` comparisons using `=`, `<>`, `<`, `>` yield `UNKNOWN`, not `TRUE` or `FALSE`.
- The condition cannot reference columns from tables not listed in the `FROM` clause (unless using correlated subqueries).
- Comparison of incompatible data types may cause implicit conversion or errors, depending on the RDBMS.

### Annotated Complete Code Examples

**Example 1: Simple Equality Filter**

```sql
-- Setup: Create a sample table
CREATE TABLE employees (
    id          SERIAL PRIMARY KEY,
    first_name  VARCHAR(50) NOT NULL,
    last_name   VARCHAR(50) NOT NULL,
    department  VARCHAR(50),
    salary      NUMERIC(10, 2)
);

INSERT INTO employees (first_name, last_name, department, salary) VALUES
    ('Alice', 'Johnson', 'Engineering', 95000.00),
    ('Bob',   'Smith',   'Marketing',   72000.00),
    ('Carol', 'Williams','Engineering', 105000.00),
    ('David', 'Brown',   'Sales',       68000.00);

-- Query: Find employees in the Engineering department
SELECT first_name, last_name, salary
FROM employees
WHERE department = 'Engineering';

-- Expected Output:
--  first_name | last_name |  salary
-- ------------+-----------+----------
--  Alice      | Johnson   | 95000.00
--  Carol      | Williams  |105000.00
```

**Why this output occurs:** The `WHERE department = 'Engineering'` condition evaluates the `department` column for each row. Rows where `department` equals `'Engineering'` evaluate to `TRUE` and are retained. Bob (Marketing) and David (Sales) evaluate to `FALSE` and are excluded.

**Example 2: Compound Condition with AND**

```sql
-- Query: Find Engineering employees earning more than $90,000
SELECT first_name, last_name, salary
FROM employees
WHERE department = 'Engineering'
  AND salary > 90000;

-- Expected Output:
--  first_name | last_name |  salary
-- ------------+-----------+----------
--  Alice      | Johnson   | 95000.00
--  Carol      | Williams  |105000.00
```

**Why this output occurs:** Both conditions must be `TRUE` for a row to be included. Alice and Carol satisfy both; Bob and David fail the department condition. The `AND` operator requires all conditions to be true.

**Example 3: Compound Condition with OR**

```sql
-- Query: Find employees in Marketing OR earning less than $70,000
SELECT first_name, last_name, department, salary
FROM employees
WHERE department = 'Marketing'
   OR salary < 70000;

-- Expected Output:
--  first_name | last_name | department |  salary
-- ------------+-----------+------------+----------
--  Bob        | Smith     | Marketing  | 72000.00
--  David      | Brown     | Sales      | 68000.00
```

**Why this output occurs:** The `OR` operator requires at least one condition to be `TRUE`. Bob satisfies the department condition; David satisfies the salary condition. Alice and Carol satisfy neither and are excluded.

### Real-World Cases

**Case 1: E-Commerce Order Filtering**

An online store displays "Orders placed in the last 30 days with status = 'Shipped'" by using `WHERE order_date >= CURRENT_DATE - INTERVAL '30 days' AND status = 'Shipped'`. This is one of the most common `WHERE` clause patterns in application development.

**Case 2: Healthcare Patient Lookup**

A hospital system retrieves patients with `WHERE age BETWEEN 18 AND 65 AND blood_type = 'O+'` to identify eligible blood donors. The `BETWEEN` and equality predicates work together to filter the population.

### References

- PostgreSQL Documentation — SELECT (WHERE Clause) - https://www.postgresql.org/docs/current/sql-select.html
- MySQL 8.4 Reference Manual — Selecting Particular Rows - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/selecting-rows.html
- Microsoft SQL Server — WHERE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/where-transact-sql

---

## 2. Logical Query Processing Order

### Definitions

**Core Definition:** Logical query processing order is the sequence in which SQL clauses are conceptually evaluated by the database engine, which differs from the textual order in which they are written.

**Technical Definition:** SQL engines process a `SELECT` statement in a defined logical order: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY` → `LIMIT`/`OFFSET`. The `WHERE` clause is logically evaluated second, immediately after the `FROM` clause produces the source rows, and before any grouping, aggregation, or column projection occurs.

**Beginner-Friendly Explanation:** Even though you write `SELECT` first and `WHERE` later, the database actually starts by reading the table (`FROM`), then filters rows (`WHERE`), then groups them, then selects columns, and finally sorts. The order you write is not the order the database thinks.

### Purposes

- To understand why column aliases cannot be used in `WHERE` (they are created later in the process).
- To explain why aggregate functions cannot appear in `WHERE` (grouping happens after filtering).
- To predict query behaviour and debug unexpected results.
- To write correct queries that reference the right columns at the right stage.
- To optimize queries by understanding when filtering occurs.

### Syntax Rules and Structure

**Logical Processing Order of a SELECT Statement:**

| Step | Clause | Operation |
|------|--------|-----------|
| 1 | `FROM` / `JOIN` | Assemble source rows from tables |
| 2 | `WHERE` | Filter individual rows |
| 3 | `GROUP BY` | Group rows by specified columns |
| 4 | `HAVING` | Filter groups |
| 5 | `SELECT` | Evaluate expressions and project columns |
| 6 | `DISTINCT` | Remove duplicate rows |
| 7 | `ORDER BY` | Sort the result set |
| 8 | `LIMIT` / `OFFSET` | Restrict the number of rows |

**Textual vs. Logical Order:**

```
Textual order:                    Logical order:
SELECT   ...                      1. FROM / JOIN
FROM     ...                      2. WHERE
WHERE    ...                      3. GROUP BY
GROUP BY ...                      4. HAVING
HAVING   ...                      5. SELECT
ORDER BY ...                      6. DISTINCT
LIMIT    ...                      7. ORDER BY
                                  8. LIMIT / OFFSET
```

**Component Breakdown:**

| Logical Step | What Happens | Consequence |
|-------------|-------------|-------------|
| `FROM` | Tables are joined; a virtual table is created | All source columns are available |
| `WHERE` | Rows filtered based on predicate | Only `TRUE` rows pass; aliases not yet defined |
| `GROUP BY` | Rows grouped by specified columns | Aggregates can be computed |
| `HAVING` | Groups filtered based on predicate | Aggregate results can be referenced |
| `SELECT` | Expressions evaluated; columns projected | Column aliases defined here |
| `ORDER BY` | Result sorted | Aliases from `SELECT` can be used |

**Syntax Rules:**

- The textual order is fixed by SQL grammar; you cannot write clauses in a different order.
- The logical order is conceptual; the physical execution plan may differ due to optimization.
- Column aliases defined in `SELECT` are not available in `WHERE` because `WHERE` is evaluated before `SELECT`.
- Aggregate functions are not allowed in `WHERE` because grouping has not yet occurred.
- `HAVING` can reference aggregates because it is evaluated after `GROUP BY`.

**Constraints and Limitations:**

- The logical order is a conceptual model; actual physical execution varies by RDBMS and query optimizer.
- Some RDBMSs allow aliases in `WHERE` as an extension (e.g., MySQL), but this is not standard SQL.
- Subqueries in `WHERE` are evaluated in the context of the outer query's current row (correlated subqueries) or independently (uncorrelated subqueries).
- The optimizer may reorder operations for performance, but the logical semantics remain unchanged.

### Annotated Complete Code Examples

**Example 1: Demonstrating Why Aliases Fail in WHERE**

```sql
-- Query: Attempt to use a SELECT alias in WHERE (will fail)
SELECT first_name,
       salary * 12 AS annual_salary
FROM employees
WHERE annual_salary > 1000000;

-- Expected Error (PostgreSQL):
-- ERROR: column "annual_salary" does not exist
-- LINE 4: WHERE annual_salary > 1000000;
```

**Why this error occurs:** The `WHERE` clause is logically evaluated before the `SELECT` clause. At the time `WHERE` is processed, the alias `annual_salary` has not yet been defined. The alias is created during the `SELECT` step, which occurs after filtering. To filter on a calculated value, you must either repeat the expression or use a subquery.

**Correct Approach:**

```sql
-- Option 1: Repeat the expression
SELECT first_name,
       salary * 12 AS annual_salary
FROM employees
WHERE salary * 12 > 1000000;

-- Option 2: Use a subquery (derived table)
SELECT *
FROM (
    SELECT first_name,
           salary * 12 AS annual_salary
    FROM employees
) AS sub
WHERE annual_salary > 1000000;
```

**Example 2: Demonstrating Why Aggregates Fail in WHERE**

```sql
-- Query: Attempt to use an aggregate in WHERE (will fail)
SELECT department, AVG(salary)
FROM employees
WHERE AVG(salary) > 80000
GROUP BY department;

-- Expected Error (PostgreSQL):
-- ERROR: aggregate functions are not allowed in WHERE
-- LINE 3: WHERE AVG(salary) > 80000
```

**Why this error occurs:** The `WHERE` clause is evaluated before `GROUP BY`. At the time `WHERE` is processed, rows have not yet been grouped, so aggregate functions cannot be computed. To filter on aggregate results, use the `HAVING` clause.

**Correct Approach:**

```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 80000;
```

**Expected Output:**

```
 department  | avg_salary
-------------+------------
 Engineering |  100000.00
```

**Why this output occurs:** The `GROUP BY` clause groups rows by department and computes the average salary per group. The `HAVING` clause then filters groups where the average exceeds 80,000. Engineering (100,000) passes; Marketing and Sales do not.

### Real-World Cases

**Case 1: Debugging "Column Not Found" Errors**

A developer writes `WHERE total_price > 100` but `total_price` is an alias for `price * quantity` defined in `SELECT`. The query fails because of logical processing order. Understanding this order allows the developer to fix the query by repeating the expression or using a subquery.

**Case 2: Performance Tuning with Predicate Pushdown**

A DBA notices a query running slowly. By understanding that `WHERE` is evaluated early (before `SELECT` and `ORDER BY`), they add an index on the `WHERE` column and achieve a dramatic speedup. The optimizer "pushes" the filter down to the storage engine, reading fewer rows.

### References

- Microsoft Learn — Examine the SELECT Statement (Logical Processing Order) - https://learn.microsoft.com/en-us/training/modules/introduction-to-transact-sql/4-examine-select-statement
- Microsoft Learn — Order of Precedence: GROUP BY vs WHERE Clauses - https://learn.microsoft.com/en-us/archive/msdn-technet-forums/f7f5a689-2f35-419b-b8ae-a055f4230ca9
- PostgreSQL Documentation — SELECT (Logical Processing) - https://www.postgresql.org/docs/current/sql-select.html

---

## 3. Comparison Operators

### Definitions

**Core Definition:** Comparison operators in SQL are symbols that compare two values and return a Boolean result (`TRUE`, `FALSE`, or `UNKNOWN`).

**Technical Definition:** A `<comparison predicate>` in SQL compares two `<value expression>`s using one of six operators: `=`, `<>`, `<`, `>`, `<=`, `>=`. The result is `TRUE` if the comparison holds, `FALSE` if it does not, and `UNKNOWN` if either operand is `NULL`. The `<>` operator is the ISO standard for "not equal"; `!=` is a non-standard extension supported by some dialects.

**Beginner-Friendly Explanation:** Comparison operators are how you ask questions like "Is this value equal to that?" or "Is this number greater than that one?" They are the building blocks of `WHERE` conditions.

### Purposes

- To test equality or inequality between values.
- To compare numeric, string, and date values.
- To establish range conditions (`<`, `>`, `<=`, `>=`).
- To form the basis of Boolean predicates used in `WHERE` and `HAVING`.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
expression comparison_operator expression
```

**Component Breakdown:**

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` | Equal to | `salary = 95000` |
| `<>` | Not equal to (ISO standard) | `department <> 'Sales'` |
| `!=` | Not equal to (non-standard) | `department != 'Sales'` |
| `<` | Less than | `salary < 80000` |
| `>` | Greater than | `salary > 100000` |
| `<=` | Less than or equal to | `salary <= 90000` |
| `>=` | Greater than or equal to | `salary >= 90000` |

**Syntax Rules:**

- Both operands must be of comparable data types.
- String comparisons are typically case-insensitive in MySQL but case-sensitive in PostgreSQL (unless using `ILIKE`).
- Date literals should be written in ISO format (`'YYYY-MM-DD'`) for portability.
- The `<>` operator is preferred over `!=` for ISO standard compliance.
- `NULL` comparisons yield `UNKNOWN`; use `IS NULL` or `IS NOT NULL` instead.

**Constraints and Limitations:**

- Implicit type conversion between incompatible types can cause errors or unexpected results.
- Floating-point comparisons may be unreliable due to precision issues.
- Collation settings affect string comparison results in some RDBMSs.
- The `!=` operator is not part of the ISO standard and may not work in all dialects.

### Annotated Complete Code Examples

**Example 1: Numeric Comparison**

```sql
-- Query: Find employees earning more than $90,000
SELECT first_name, salary
FROM employees
WHERE salary > 90000;

-- Expected Output:
--  first_name |  salary
-- ------------+----------
--  Alice      | 95000.00
--  Carol      |105000.00
```

**Why this output occurs:** The `>` operator compares each row's `salary` to the literal `90000`. Alice (95,000) and Carol (105,000) exceed the threshold; Bob (72,000) and David (68,000) do not.

**Example 2: String Comparison**

```sql
-- Query: Find employees not in the Sales department
SELECT first_name, department
FROM employees
WHERE department <> 'Sales';

-- Expected Output:
--  first_name | department
-- ------------+-------------
--  Alice      | Engineering
--  Bob        | Marketing
--  Carol      | Engineering
```

**Why this output occurs:** The `<>` operator returns `TRUE` for rows where `department` is not `'Sales'`. David is excluded because his department equals `'Sales'`.

### Real-World Cases

**Case 1: Price Range Filtering**

An e-commerce search page uses `WHERE price >= 50 AND price <= 200` to show products in a specific price range. The comparison operators define the boundaries of the range.

**Case 2: Date Range Reports**

A financial report uses `WHERE transaction_date >= '2024-01-01' AND transaction_date < '2024-07-01'` to retrieve transactions for the first half of the year. The `>=` and `<` operators precisely define the period.

### References

- MySQL 8.4 Reference Manual — Comparison Functions and Operators - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/comparison-operators.html
- PostgreSQL Documentation — Comparison Operators - https://www.postgresql.org/docs/current/functions-comparison.html
- Microsoft SQL Server — Comparison Operators (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/comparison-operators-transact-sql

---

## 4. Logical Operators (AND, OR, NOT)

### Definitions

**Core Definition:** Logical operators (`AND`, `OR`, `NOT`) combine or negate Boolean expressions in SQL, enabling complex filtering conditions.

**Technical Definition:** In SQL's three-valued logic, `AND` returns `TRUE` only if both operands are `TRUE`; `OR` returns `TRUE` if at least one operand is `TRUE`; `NOT` returns the opposite truth value (`TRUE` becomes `FALSE`, `FALSE` becomes `TRUE`, and `UNKNOWN` remains `UNKNOWN`). Operator precedence is `NOT` > `AND` > `OR`.

**Beginner-Friendly Explanation:** Logical operators let you combine multiple conditions: `AND` means "both must be true," `OR` means "at least one must be true," and `NOT` means "the opposite."

### Purposes

- To combine multiple filtering conditions into a single `WHERE` clause.
- To express complex business rules involving multiple criteria.
- To negate conditions for exclusion filtering.
- To control operator precedence with parentheses for precise logic.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
WHERE condition1 AND condition2
WHERE condition1 OR condition2
WHERE NOT condition
WHERE (condition1 OR condition2) AND condition3
```

**Component Breakdown:**

| Operator | Truth Table | Precedence |
|----------|------------|------------|
| `NOT` | Inverts truth value | Highest (evaluated first) |
| `AND` | `TRUE` only if both `TRUE` | Middle |
| `OR` | `TRUE` if at least one `TRUE` | Lowest (evaluated last) |

**Truth Table (Three-Valued Logic):**

| A | B | A AND B | A OR B | NOT A |
|---|---|---------|--------|-------|
| TRUE | TRUE | TRUE | TRUE | FALSE |
| TRUE | FALSE | FALSE | TRUE | FALSE |
| TRUE | UNKNOWN | UNKNOWN | TRUE | FALSE |
| FALSE | FALSE | FALSE | FALSE | TRUE |
| FALSE | UNKNOWN | FALSE | UNKNOWN | TRUE |
| UNKNOWN | UNKNOWN | UNKNOWN | UNKNOWN | UNKNOWN |

**Syntax Rules:**

- `NOT` has the highest precedence; `AND` is next; `OR` is lowest.
- Parentheses override default precedence and are strongly recommended for clarity.
- `AND` and `OR` are binary operators; `NOT` is unary.
- Conditions combined with logical operators must each be Boolean expressions.
- SQL may short-circuit evaluation: if `AND`'s first operand is `FALSE`, the second may not be evaluated; if `OR`'s first operand is `TRUE`, the second may not be evaluated.

**Constraints and Limitations:**

- Relying on short-circuit evaluation for side effects is unsafe and non-portable.
- `NOT` combined with `NULL` yields `UNKNOWN`, which excludes the row.
- Deeply nested conditions can become difficult to read and maintain; consider using CTEs or restructuring the query.
- Some RDBMSs limit the number of predicates in a single condition (though this limit is generally very high).

### Annotated Complete Code Examples

**Example 1: Combining AND and OR with Parentheses**

```sql
-- Query: Find Engineering employees with high salary, OR any Marketing employee
SELECT first_name, department, salary
FROM employees
WHERE (department = 'Engineering' AND salary > 90000)
   OR department = 'Marketing';

-- Expected Output:
--  first_name | department  |  salary
-- ------------+-------------+----------
--  Alice      | Engineering | 95000.00
--  Bob        | Marketing   | 72000.00
--  Carol      | Engineering |105000.00
```

**Why this output occurs:** The parentheses ensure that the `AND` condition is evaluated first. The query returns employees who are either (Engineering AND salary > 90,000) OR in Marketing. Without parentheses, `AND` would still take precedence, but explicit parentheses improve readability and prevent mistakes.

**Example 2: Using NOT**

```sql
-- Query: Find employees NOT in Engineering
SELECT first_name, department
FROM employees
WHERE NOT department = 'Engineering';

-- Expected Output:
--  first_name | department
-- ------------+-------------
--  Bob        | Marketing
--  David      | Sales
```

**Why this output occurs:** `NOT department = 'Engineering'` is logically equivalent to `department <> 'Engineering'`. Rows where the department is Engineering evaluate to `FALSE` for the inner condition, which `NOT` inverts to `TRUE` (excluding them). Bob and David are returned.

### Real-World Cases

**Case 1: Multi-Criteria Product Search**

An e-commerce platform allows users to filter products by multiple criteria simultaneously: `WHERE (category = 'Electronics' OR category = 'Computers') AND price BETWEEN 500 AND 2000 AND in_stock = TRUE`. The logical operators combine these conditions into a single precise filter.

**Case 2: Excluding Test Accounts**

A SaaS application excludes internal test accounts from reports using `WHERE NOT email LIKE '%@test.internal'`. The `NOT` operator inverts the `LIKE` condition, excluding matching rows.

### References

- MySQL 8.4 Reference Manual — Logical Operators - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/logical-operators.html
- PostgreSQL Documentation — Logical Operators - https://www.postgresql.org/docs/current/functions-logical.html
- Microsoft SQL Server — Logical Operators (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/logical-operators-transact-sql

---

## 5. NULL Handling and Three-Valued Logic

### Definitions

**Core Definition:** SQL uses three-valued logic, where Boolean expressions can evaluate to `TRUE`, `FALSE`, or `UNKNOWN`. `NULL` represents an unknown or missing value, and any comparison with `NULL` yields `UNKNOWN`.

**Technical Definition:** Per ISO/IEC 9075, SQL's truth values are `TRUE`, `FALSE`, and `UNKNOWN`. `NULL` is not a value but a marker for the absence of a value. Comparisons involving `NULL` (e.g., `NULL = 5`, `NULL <> 5`) evaluate to `UNKNOWN`. The `WHERE` clause retains only rows where the search condition evaluates to `TRUE`; rows evaluating to `UNKNOWN` are discarded, not returned.

**Beginner-Friendly Explanation:** `NULL` means "I don't know." If you ask "Is this value equal to 5?" and the value is unknown, the answer is "I don't know" — not "yes" or "no." SQL handles this with a third truth value called `UNKNOWN`.

### Purposes

- To represent missing, unknown, or not-applicable data.
- To test explicitly for `NULL` using `IS NULL` and `IS NOT NULL`.
- To understand why `WHERE column = NULL` returns no rows.
- To correctly handle `NULL` in filtering, aggregation, and joins.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
WHERE column IS NULL
WHERE column IS NOT NULL
WHERE column IS DISTINCT FROM value   -- PostgreSQL
WHERE column IS NOT DISTINCT FROM value
```

**Component Breakdown:**

| Predicate | Meaning | Returns |
|-----------|---------|---------|
| `IS NULL` | Tests for NULL | `TRUE` if NULL, `FALSE` otherwise |
| `IS NOT NULL` | Tests for non-NULL | `TRUE` if not NULL, `FALSE` otherwise |
| `IS DISTINCT FROM` | NULL-safe inequality | `TRUE` if different, including NULL vs non-NULL |
| `IS NOT DISTINCT FROM` | NULL-safe equality | `TRUE` if equal or both NULL |

**Syntax Rules:**

- `NULL` is a keyword, not a string; `'NULL'` is the four-character string.
- `column = NULL` always yields `UNKNOWN`, never `TRUE`.
- `column <> NULL` also always yields `UNKNOWN`.
- `IS NULL` and `IS NOT NULL` are the only standard ways to test for `NULL`.
- `COALESCE(column, default)` replaces `NULL` with a default value.
- `NULLIF(value1, value2)` returns `NULL` if `value1 = value2`, else `value1`.

**Constraints and Limitations:**

- `NULL` propagation affects arithmetic: `5 + NULL = NULL`.
- Aggregate functions ignore `NULL` values (e.g., `AVG` skips `NULL` rows), which can cause unexpected results.
- `DISTINCT` treats all `NULL`s as duplicates; only one `NULL` is returned.
- `ORDER BY` places `NULL`s either first or last depending on the RDBMS and `NULLS FIRST`/`NULLS LAST` specification.
- `NOT IN` with a subquery containing `NULL` can return no rows due to three-valued logic.

### Annotated Complete Code Examples

**Example 1: Demonstrating NULL Comparison Failure**

```sql
-- Setup: Add a row with NULL department
INSERT INTO employees (first_name, last_name, department, salary)
VALUES ('Eve', 'Davis', NULL, 88000.00);

-- Query: Attempt to find NULL department using = NULL (returns nothing)
SELECT first_name, department
FROM employees
WHERE department = NULL;

-- Expected Output:
-- (0 rows)
```

**Why this output occurs:** The condition `department = NULL` evaluates to `UNKNOWN` for every row (even the row where department is actually NULL), because `NULL` is not a value and cannot be compared with `=`. The `WHERE` clause retains only rows where the condition is `TRUE`, so no rows are returned.

**Correct Approach:**

```sql
SELECT first_name, department
FROM employees
WHERE department IS NULL;

-- Expected Output:
--  first_name | department
-- ------------+------------
--  Eve        | (null)
```

**Example 2: NULL in Aggregation**

```sql
-- Query: Count employees and average salary (NULL salary handling)
SELECT COUNT(*) AS total_employees,
       COUNT(salary) AS employees_with_salary,
       AVG(salary) AS avg_salary
FROM employees;

-- Expected Output (with one NULL salary row added):
--  total_employees | employees_with_salary | avg_salary
-- -----------------+-----------------------+------------
--                5 |                     4 |    85000.00
```

**Why this output occurs:** `COUNT(*)` counts all rows, including those with `NULL` values. `COUNT(salary)` counts only rows where `salary` is not `NULL`. `AVG(salary)` computes the average over non-`NULL` salaries only. This demonstrates that aggregate functions generally ignore `NULL` values.

### Real-World Cases

**Case 1: Optional Form Fields**

A user registration form has an optional "phone number" field. Rows where the user did not provide a phone number have `NULL` in the `phone` column. Queries that need to find users without a phone number must use `WHERE phone IS NULL`, not `WHERE phone = NULL`.

**Case 2: Data Quality Auditing**

A data engineer checks for missing values using `SELECT COUNT(*) FROM table WHERE critical_column IS NULL`. This identifies records requiring remediation before they are used in analysis.

### References

- PostgreSQL Documentation — Comparisons (NULL Handling) - https://www.postgresql.org/docs/current/functions-comparison.html
- Oracle Database SQL Language Reference — NULLs - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Nulls.html
- Microsoft SQL Server — NULL and UNKNOWN (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/null-and-unknown-transact-sql

---

## 6. BETWEEN, IN, LIKE, and IS NULL Predicates

### Definitions

**Core Definition:** SQL provides specialised predicates — `BETWEEN`, `IN`, `LIKE`, and `IS NULL` — that simplify common filtering patterns such as range checks, list membership, pattern matching, and null testing.

**Technical Definition:** These predicates are syntactic sugar for equivalent combinations of comparison and logical operators: `BETWEEN a AND b` is equivalent to `a >= x AND x <= b`; `IN (v1, v2, ...)` is equivalent to `x = v1 OR x = v2 OR ...`; `LIKE` performs pattern matching with `%` (any sequence) and `_` (any single character); `IS NULL` tests for null.

**Beginner-Friendly Explanation:** These are shortcut operators that make your SQL shorter and clearer. Instead of writing `x >= 10 AND x <= 20`, you can write `x BETWEEN 10 AND 20`.

### Purposes

- To express range conditions concisely with `BETWEEN`.
- To test membership in a list of values with `IN`.
- To perform pattern matching on strings with `LIKE`.
- To test for `NULL` values with `IS NULL`.
- To improve query readability and reduce repetition.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
WHERE column BETWEEN low AND high
WHERE column NOT BETWEEN low AND high
WHERE column IN (value1, value2, ...)
WHERE column NOT IN (value1, value2, ...)
WHERE column LIKE 'pattern'
WHERE column NOT LIKE 'pattern'
WHERE column IS NULL
WHERE column IS NOT NULL
```

**Component Breakdown:**

| Predicate | Equivalent To | Notes |
|-----------|--------------|-------|
| `BETWEEN a AND b` | `x >= a AND x <= b` | Inclusive of both endpoints |
| `IN (v1, v2, ...)` | `x = v1 OR x = v2 OR ...` | Values can be literals or subquery results |
| `LIKE 'pattern'` | — | `%` = any sequence; `_` = any single character |
| `IS NULL` | — | Only standard way to test for `NULL` |

**Syntax Rules:**

- `BETWEEN` is inclusive: `BETWEEN 1 AND 5` includes 1 and 5.
- `IN` values must be of comparable data types.
- `LIKE` patterns use `%` and `_` wildcards; some RDBMSs support `ESCAPE` for literal wildcards.
- `LIKE` is case-sensitive in PostgreSQL; use `ILIKE` for case-insensitive matching. MySQL's `LIKE` is case-insensitive by default.
- `IS NULL` and `IS NOT NULL` are the only standard ways to test for `NULL`.

**Constraints and Limitations:**

- `BETWEEN` cannot be used with `NULL` endpoints reliably.
- `NOT IN` with a subquery containing `NULL` returns no rows due to three-valued logic.
- `LIKE` with a leading `%` prevents index usage in most RDBMSs.
- `LIKE` pattern matching is not standardised for non-ASCII characters.
- `IN` with a very large list can cause performance issues; consider using a temporary table or join instead.

### Annotated Complete Code Examples

**Example 1: BETWEEN**

```sql
-- Query: Find employees earning between $70,000 and $100,000
SELECT first_name, salary
FROM employees
WHERE salary BETWEEN 70000 AND 100000;

-- Expected Output:
--  first_name |  salary
-- ------------+----------
--  Alice      | 95000.00
--  Bob        | 72000.00
--  Eve        | 88000.00
```

**Why this output occurs:** `BETWEEN 70000 AND 100000` includes rows where salary is ≥ 70,000 and ≤ 100,000. Carol (105,000) and David (68,000) fall outside the range.

**Example 2: IN**

```sql
-- Query: Find employees in Engineering or Marketing
SELECT first_name, department
FROM employees
WHERE department IN ('Engineering', 'Marketing');

-- Expected Output:
--  first_name | department
-- ------------+-------------
--  Alice      | Engineering
--  Bob        | Marketing
--  Carol      | Engineering
```

**Why this output occurs:** `IN ('Engineering', 'Marketing')` is equivalent to `department = 'Engineering' OR department = 'Marketing'`. David (Sales) and Eve (NULL) are excluded.

**Example 3: LIKE**

```sql
-- Query: Find employees whose first name starts with 'A'
SELECT first_name
FROM employees
WHERE first_name LIKE 'A%';

-- Expected Output:
--  first_name
-- ------------
--  Alice
```

**Why this output occurs:** The pattern `'A%'` matches any string starting with 'A'. Only Alice qualifies. The `%` wildcard matches any sequence of characters (including none).

### Real-World Cases

**Case 1: Date Range Reports**

A monthly report uses `WHERE order_date BETWEEN '2024-01-01' AND '2024-01-31'` to retrieve all orders placed in January. `BETWEEN` provides a clean, readable range filter.

**Case 2: Product Category Filtering**

An e-commerce frontend uses `WHERE category_id IN (1, 3, 5, 7)` to display products from selected categories. `IN` allows the application to pass a dynamic list of category IDs.

**Case 3: Email Domain Search**

A marketing query uses `WHERE email LIKE '%@gmail.com'` to find all customers using Gmail. The `%` wildcard allows any prefix before the domain.

### References

- PostgreSQL Documentation — Pattern Matching (LIKE) - https://www.postgresql.org/docs/current/functions-matching.html
- MySQL 8.4 Reference Manual — Comparison Functions (IN, BETWEEN, LIKE) - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/comparison-operators.html
- Microsoft SQL Server — BETWEEN (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/between-transact-sql

---

## 7. WHERE vs HAVING

### Definitions

**Core Definition:** `WHERE` filters individual rows before grouping, while `HAVING` filters groups after aggregation. They are complementary clauses serving different stages of query processing.

**Technical Definition:** Per the SQL standard, the `<where clause>` filters rows from the result of the `<from clause>` before `GROUP BY` is applied. The `<having clause>` filters groups from the result of `GROUP BY` after aggregation. `WHERE` cannot reference aggregate functions; `HAVING` can. `WHERE` can reference individual column values; `HAVING` can reference grouping columns and aggregates.

**Beginner-Friendly Explanation:** `WHERE` is a filter for rows; `HAVING` is a filter for groups. Use `WHERE` to narrow down which rows you're looking at, and `HAVING` to narrow down which groups you want to keep.

### Purposes

- To filter individual rows before aggregation using `WHERE`.
- To filter aggregated groups after grouping using `HAVING`.
- To optimize performance by filtering early with `WHERE` when possible.
- To express conditions that depend on aggregate results using `HAVING`.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT column1, aggregate_function(column2)
FROM table_name
WHERE row_condition
GROUP BY column1
HAVING aggregate_condition;
```

**Component Breakdown:**

| Clause | Applied To | Can Use Aggregates? | Can Use Aliases? |
|--------|-----------|---------------------|------------------|
| `WHERE` | Individual rows | No | No (standard SQL) |
| `HAVING` | Groups | Yes | Some dialects allow |

**Syntax Rules:**

- `WHERE` must appear before `GROUP BY` in textual order.
- `HAVING` must appear after `GROUP BY` and before `ORDER BY`.
- `WHERE` can be used without `GROUP BY`; `HAVING` without `GROUP BY` is allowed in some dialects but behaves like `WHERE` on a single group.
- Both clauses use the same Boolean expression syntax.
- `WHERE` conditions are evaluated before `GROUP BY`; `HAVING` conditions after.

**Constraints and Limitations:**

- `WHERE` cannot reference aggregate functions.
- `HAVING` without `GROUP BY` is non-standard and behaves inconsistently across dialects.
- Column aliases from `SELECT` are generally not available in `WHERE` but may be available in `HAVING` in some dialects.
- Filtering in `WHERE` is generally more efficient than in `HAVING` because fewer rows need to be grouped.

### Annotated Complete Code Examples

**Example 1: WHERE and HAVING Together**

```sql
-- Query: Find departments with average salary > 80,000, considering
-- only employees earning more than 70,000
SELECT department,
       COUNT(*) AS employee_count,
       AVG(salary) AS avg_salary
FROM employees
WHERE salary > 70000
GROUP BY department
HAVING AVG(salary) > 80000;

-- Expected Output:
--  department  | employee_count | avg_salary
-- -------------+----------------+------------
--  Engineering |              2 |  100000.00
```

**Why this output occurs:** The `WHERE` clause filters out employees earning ≤ 70,000 (David at 68,000) before grouping. The `GROUP BY` then groups the remaining rows by department. The `HAVING` clause filters groups where the average salary exceeds 80,000. Engineering (100,000) passes; Marketing (72,000) does not.

**Example 2: Demonstrating WHERE Cannot Use Aggregates**

```sql
-- This query will FAIL
SELECT department, AVG(salary)
FROM employees
WHERE AVG(salary) > 80000
GROUP BY department;

-- Expected Error:
-- ERROR: aggregate functions are not allowed in WHERE
```

**Why this error occurs:** The `WHERE` clause is evaluated before `GROUP BY`, so aggregates cannot be computed. The correct place for aggregate conditions is the `HAVING` clause.

### Real-World Cases

**Case 1: Sales Performance Report**

A sales manager wants to see "regions with more than 100 orders and average order value above $500." The `WHERE` clause filters out cancelled orders, then `GROUP BY` groups by region, and `HAVING` filters regions meeting the aggregate thresholds.

**Case 2: Customer Segmentation**

A marketing analyst uses `WHERE signup_date >= '2024-01-01'` to focus on recent customers, then `GROUP BY country` and `HAVING COUNT(*) > 1000` to identify countries with significant recent growth.

### References

- PostgreSQL Documentation — Aggregate Functions (WHERE vs HAVING) - https://www.postgresql.org/docs/current/tutorial-agg.html
- Microsoft SQL Server — HAVING (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-having-transact-sql
- MySQL 8.4 Reference Manual — GROUP BY and HAVING - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/select.html

---

## 8. Performance and Indexing Considerations

### Definitions

**Core Definition:** Query performance in the context of `WHERE` clauses refers to how efficiently the database engine can locate and return rows matching the filter condition, heavily influenced by the availability and use of indexes.

**Technical Definition:** A database index is a data structure (typically a B-tree, hash, or bitmap) that provides fast access to rows based on the values of one or more columns. A "sargable" predicate (Search ARGument-able) is one that can use an index to speed up row retrieval. Non-sargable predicates (e.g., functions on indexed columns, leading wildcards in `LIKE`) force full table scans.

**Beginner-Friendly Explanation:** An index is like the index at the back of a book — it helps the database find rows quickly without reading every page. Some `WHERE` conditions can use the index; others force the database to read the entire table.

### Purposes

- To minimize the number of rows read from disk.
- To reduce query execution time for large tables.
- To improve application responsiveness and user experience.
- To reduce server resource consumption (CPU, I/O, memory).
- To enable scaling as data volume grows.

### Syntax Rules and Structure

**Creating an Index:**

```sql
CREATE INDEX index_name
ON table_name (column_name);
```

**Sargable vs Non-Sargable Predicates:**

| Sargable (Index-Friendly) | Non-Sargable (Index-Unfriendly) |
|---------------------------|--------------------------------|
| `WHERE salary > 80000` | `WHERE salary * 12 > 960000` |
| `WHERE name = 'Alice'` | `WHERE UPPER(name) = 'ALICE'` |
| `WHERE date >= '2024-01-01'` | `WHERE YEAR(date) = 2024` |
| `WHERE id IN (1, 2, 3)` | `WHERE id + 1 IN (2, 3, 4)` |

**Syntax Rules:**

- Indexes are created on one or more columns of a table.
- Composite indexes cover multiple columns and can support queries filtering on the leading column(s).
- The query optimizer decides whether to use an index based on statistics, selectivity, and cost estimates.
- `EXPLAIN` (PostgreSQL, MySQL) or `EXPLAIN PLAN` (Oracle, SQL Server) shows the execution plan and whether indexes are used.

**Constraints and Limitations:**

- Indexes consume additional disk space and slow down write operations (`INSERT`, `UPDATE`, `DELETE`).
- Not all predicates can use indexes; functions, type casts, and leading wildcards often prevent index usage.
- Index selectivity matters: an index on a column with few distinct values (e.g., gender) provides little benefit.
- Statistics must be up to date for the optimizer to make good decisions.
- Too many indexes on a table can degrade write performance and increase maintenance overhead.

### Annotated Complete Code Examples

**Example 1: Sargable vs Non-Sargable Query**

```sql
-- Setup: Create an index on salary
CREATE INDEX idx_employees_salary ON employees (salary);

-- Sargable query (can use index)
EXPLAIN SELECT first_name, salary
FROM employees
WHERE salary > 90000;

-- Expected Output (PostgreSQL):
--  QUERY PLAN
-- ----------------------------------------------------------------------
--  Index Scan using idx_employees_salary on employees
--    Index Cond: (salary > 90000.00)
```

**Why this plan occurs:** The predicate `salary > 90000` compares the indexed column directly to a constant value, allowing the B-tree index to locate matching rows efficiently.

**Non-Sargable Query:**

```sql
-- Non-sargable query (cannot use index on salary)
EXPLAIN SELECT first_name, salary
FROM employees
WHERE salary * 12 > 1080000;

-- Expected Output (PostgreSQL):
--  QUERY PLAN
-- ----------------------------------------------------------------------
--  Seq Scan on employees
--    Filter: ((salary * 12) > 1080000.00)
```

**Why this plan occurs:** The expression `salary * 12` applies a calculation to the indexed column, preventing the index from being used. The database must scan every row and compute the expression. The query can be rewritten as `WHERE salary > 90000` to be sargable.

### Real-World Cases

**Case 1: E-Commerce Product Search**

An e-commerce site creates composite indexes on `(category_id, price)` and `(brand_id, in_stock)` to support common search filters. Queries like `WHERE category_id = 5 AND price BETWEEN 100 AND 500` can use the index efficiently.

**Case 2: Log Analysis**

A monitoring system stores millions of log entries per day. An index on `(log_level, timestamp)` allows queries like `WHERE log_level = 'ERROR' AND timestamp >= NOW() - INTERVAL '1 hour'` to return results in milliseconds instead of minutes.

### References

- MySQL 8.4 Reference Manual — WHERE Clause Optimization - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/where-optimization.html
- PostgreSQL Documentation — Indexes - https://www.postgresql.org/docs/current/indexes.html
- Microsoft SQL Server — Query Tuning with Indexes - https://learn.microsoft.com/en-us/sql/relational-databases/indexes/

---

## Summary Table of Key Features

| Feature | Purpose | Key Limitation |
|---------|---------|----------------|
| `WHERE` Syntax | Filter rows by Boolean condition | Cannot reference `SELECT` aliases |
| Logical Order | `WHERE` evaluated after `FROM`, before `GROUP BY` | Conceptual, not physical execution |
| Comparison Operators | Test equality, inequality, ranges | `NULL` comparisons yield `UNKNOWN` |
| `AND`, `OR`, `NOT` | Combine/negate conditions | Precedence: `NOT` > `AND` > `OR` |
| `NULL` / Three-Valued Logic | Handle missing data | Only `TRUE` rows returned |
| `BETWEEN`, `IN`, `LIKE` | Concise range/list/pattern filtering | Non-sargable patterns hurt performance |
| `WHERE` vs `HAVING` | Row filtering vs group filtering | `WHERE` cannot use aggregates |
| Indexing | Speed up row retrieval | Indexes slow writes; not all predicates use them |

---

## References

- PostgreSQL Documentation — SELECT (WHERE Clause) - https://www.postgresql.org/docs/current/sql-select.html
- PostgreSQL Documentation — Querying a Table - https://www.postgresql.org/docs/17/tutorial-select.html
- PostgreSQL Documentation — Comparison Operators - https://www.postgresql.org/docs/current/functions-comparison.html
- MySQL 8.4 Reference Manual — Selecting Particular Rows - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/selecting-rows.html
- MySQL 8.4 Reference Manual — WHERE Clause Optimization - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/where-optimization.html
- Microsoft Learn — WHERE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/where-transact-sql
- Microsoft Learn — Examine the SELECT Statement (Logical Processing Order) - https://learn.microsoft.com/en-us/training/modules/introduction-to-transact-sql/4-examine-select-statement
- Microsoft Learn — Order of Precedence: GROUP BY vs WHERE Clauses - https://learn.microsoft.com/en-us/archive/msdn-technet-forums/f7f5a689-2f35-419b-b8ae-a055f4230ca9
- Oracle Database SQL Language Reference — SELECT - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SELECT.html
- Oracle Database SQL Language Reference — NULLs - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Nulls.html
- ISO/IEC 9075-2:2003 — SQL Standard (WHERE Clause Specification) - https://jtc1sc32.org/doc/N0251-0300/32N0254.pdf
- Databricks SQL — WHERE Clause - https://learn.microsoft.com/en-us/azure/databricks/sql/language-manual/sql-ref-syntax-qry-select-where