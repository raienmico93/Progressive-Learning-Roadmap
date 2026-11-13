# Comprehensive Programming Cheat Sheet: SQL Selecting Data

## Topic Overview

### Definitions

**Core Definition**
Selecting data is the process of using the `SELECT` statement—together with its clauses for column selection, aliasing, expression evaluation, and function application—to retrieve, compute, and shape rows from one or more tables in a relational database.

**Technical Definition**
The `SELECT` statement’s projection list (the expressions immediately following `SELECT`) determines the columns and computed values returned. A value expression in the projection list can be a constant or literal, a column reference, an operator invocation, a function call, an aggregate expression, a type cast, a scalar subquery, or a parenthesized expression. Column references take the form `correlation.columnname`, where `correlation` is a table name or table alias. PostgreSQL accepts two equivalent type-cast syntaxes: `CAST(expression AS type)` and `expression::type`. Arithmetic, string concatenation (`||`), and function calls (string, numeric, and date functions) are all valid expression forms. Aliases (`AS name`) rename output columns and are essential for computed columns, which otherwise receive auto-generated names.

**Beginner-Friendly Explanation**
Selecting data is how you ask the database questions. You say what you want to see (`SELECT`), which table it lives in (`FROM`), and optionally how to filter (`WHERE`), sort (`ORDER BY`), and limit (`LIMIT`) the results. Within the `SELECT` list you can pick columns by name, compute new values (like `salary * 12`), combine text (like `first_name || ' ' || last_name`), call functions (like `UPPER(name)` or `DATE_TRUNC('month', order_date)`), and convert values from one type to another (like `'2025-01-01'::date`). Aliases let you give those computed columns readable names.

### Key Characteristics

- **Projection is flexible**: The `SELECT` list is not limited to bare columns; it can contain arbitrary expressions, function calls, and casts.
- **Aliases matter for readability**: Computed columns receive auto-generated names (such as `?column?` in PostgreSQL) unless renamed with `AS`. Aliases are visible in `ORDER BY` but **not** in `WHERE`, because `WHERE` is evaluated before `SELECT`.
- **Type casting has two syntaxes**: `CAST(expression AS type)` is the SQL-standard form; `expression::type` is the PostgreSQL shorthand. SQL Server provides both `CAST` and `CONVERT`, with `CONVERT` offering style codes for date formatting.
- **Vendor divergence in string concatenation**: PostgreSQL and SQLite use `||`; MySQL uses `CONCAT()`; SQL Server uses `+` (and `CONCAT()` in newer versions).
- **Formatting is a productivity tool**: Consistent capitalization, indentation, and clause ordering dramatically reduce review time and debugging effort.

### Prerequisites

- A running RDBMS (PostgreSQL, MySQL, or SQL Server) and a client connection.
- At least one table with data, or the ability to run `SELECT` without a `FROM` clause as a calculator.
- Familiarity with basic data types (numeric, character, date/time) and the concept of a `NULL` value.

### Related Programming Areas

- Data Manipulation Language (DML) – `INSERT`, `UPDATE`, `DELETE`.
- Data Definition Language (DDL) – table and column definitions.
- Query optimisation – indexes, expression indexes, and the query planner.
- Application development – result-set mapping, aliasing for ORMs, and API projections.

---

## Core Concepts and Features

### 1. Selecting Columns

#### Definitions

**Core Definition**
Selecting columns is the act of naming specific columns in the `SELECT` list to control which attributes appear in the result set and in what order.

**Technical Definition**
The projection list is the comma-separated sequence of expressions immediately following `SELECT`. Each expression can be a bare column name, a qualified column reference (`table.column` or `alias.column`), or a more complex expression. The order of expressions in the list determines the order of columns in the result set.

**Beginner-Friendly Explanation**
Instead of returning every column in a table, you list exactly the ones you need. This keeps results small, avoids fetching sensitive data, and makes queries faster.

#### Purposes

- To return only the attributes needed by the application, reducing network transfer and memory usage.
- To control the column order in the output, which matters for positional consumers (e.g., CSV exports, `INSERT ... SELECT`).
- To avoid exposing sensitive columns (passwords, personal data) that should not leave the database.

#### Syntax Rules and Structure

```sql
SELECT column1 [, column2, ...]
FROM table_name;
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `column1, column2` | One or more column names, optionally qualified (`table.column`). |
| `FROM table_name` | The source table. |

**Syntax rules:**

- Column names are case-insensitive unless quoted with double quotes.
- Qualifying columns with a table name or alias is required when the same column name appears in multiple tables in the `FROM` clause.

**Constraints and limitations:**

- `SELECT *` is convenient for exploration but discouraged in production because it breaks silently when the schema changes.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Selecting specific columns

```sql
-- Step 1: Select only the columns needed for a contact list.
SELECT first_name, last_name, email
FROM employees;
```

**Expected output:**

```text
 first_name | last_name |        email
------------+-----------+----------------------
 Alice      | Smith     | alice@example.com
 Bob        | Jones     | bob@example.com
(2 rows)
```

**Why this output appears**: The `SELECT` list names three columns. PostgreSQL returns those columns in the specified order for every row in the table. Columns not listed (e.g., `salary`, `department`) are not fetched.

#### Real-World Cases

- **API projections**: A REST endpoint selects only `id`, `username`, and `email` to avoid over-fetching.
- **Reporting extracts**: A CSV export selects columns in a fixed order to match a downstream file format.

---

### 2. Selecting All Columns

#### Definitions

**Core Definition**
Selecting all columns uses the wildcard `*` to return every column from every table in the `FROM` clause.

**Technical Definition**
A single unqualified `*` expands to all columns of all tables in the `FROM` clause. A qualified `table_name.*` expands to all columns of the specified table only. Column expansion occurs at the top level of the `SELECT` output list.

**Beginner-Friendly Explanation**
`SELECT *` means “give me everything.” It is handy while exploring an unfamiliar table but risky in application code.

#### Purposes

- To explore an unfamiliar table during development or debugging.
- To quickly inspect all data without knowing column names.
- To retrieve every column when the application genuinely needs the whole row.

#### Syntax Rules and Structure

```sql
SELECT * FROM table_name;
SELECT table_name.* FROM table_name;
```

**Component breakdown:**

| Form | Description |
|------|-------------|
| `*` | All columns from all tables in `FROM`. |
| `table_name.*` | All columns from one specific table. |

**Syntax rules:**

- `*` must appear at the top level of the `SELECT` list; it cannot be embedded inside an expression.
- When multiple tables are joined, `*` returns columns from all of them, which can produce duplicate column names.

**Constraints and limitations:**

- `SELECT *` fetches columns the application may not need, increasing I/O and network transfer.
- Adding a new column to the table changes the result set of `SELECT *`, which can break positional consumers.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SELECT * for exploration

```sql
-- Step 1: Inspect all columns of an unfamiliar table.
SELECT * FROM employees LIMIT 1;
```

**Expected output:**

```text
 employee_id | first_name | last_name | salary | department_id | hired_at
-------------+------------+-----------+--------+---------------+-----------
           1 | Alice      | Smith     |  80000 |             1 | 2020-01-15
(1 row)
```

**Why this output appears**: The `*` expands to every column of `employees`. The `LIMIT 1` restricts the output to one row for quick inspection.

#### Real-World Cases

- **Debugging**: A developer runs `SELECT * FROM order LIMIT 5` to see what columns and values exist.
- **Ad-hoc analysis**: An analyst uses `SELECT *` in a scratch query, then narrows the column list before sharing the query.

---

### 3. Column Aliases

#### Definitions

**Core Definition**
A column alias is a temporary name assigned to an output column using the `AS` keyword (or an implicit alias without `AS`).

**Technical Definition**
The syntax `expression AS output_name` (or `expression output_name`) renames the column in the result set. If the alias contains spaces or uppercase letters that must be preserved, it must be enclosed in double quotes. Aliases are visible in `ORDER BY` (and in MySQL, also in `GROUP BY` and `HAVING`), but **not** in `WHERE`, because `WHERE` is evaluated before `SELECT`. `AS` is optional but recommended for clarity and to avoid accidental errors.

**Beginner-Friendly Explanation**
An alias is a nickname for a column in the output. Instead of a cryptic auto-generated name like `?column?`, you get `annual_salary` or `full_name`.

#### Purposes

- To give readable names to computed columns.
- To shorten long or ambiguous column names in the output.
- To avoid duplicate column names when joining tables with overlapping column names.
- To rename columns for downstream consumers (APIs, reports) without changing the underlying schema.

#### Syntax Rules and Structure

```sql
SELECT expression AS alias_name
FROM table_name;
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `expression` | A column, computation, or function call. |
| `AS` | Optional keyword; recommended for clarity. |
| `alias_name` | The output name; double-quote if it contains spaces or capitals. |

**Syntax rules:**

- `AS` is optional (`salary annual_salary` works), but include it to read clearly and prevent mistakes.
- Single quotes are for string **values**; double quotes are for **identifiers** (aliases). Mixing them up is a common beginner error.
- Aliases cannot be referenced in `WHERE` in standard SQL.

**Constraints and limitations:**

- The alias is only available for the duration of the query; it does not persist.
- In PostgreSQL, an alias without `AS` that happens to match a keyword can cause a syntax error; `AS` removes the ambiguity.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Aliasing computed columns

```sql
-- Step 1: Give computed columns readable names.
SELECT first_name || ' ' || last_name AS full_name,
       salary * 12 AS annual_salary
FROM employees;
```

**Expected output:**

```text
   full_name    | annual_salary
----------------+---------------
 Alice Smith    |        960000
 Bob Jones      |        840000
(2 rows)
```

**Why this output appears**: Without `AS`, PostgreSQL would invent names like `?column?` for the concatenation and the multiplication. The aliases `full_name` and `annual_salary` make the result self-documenting.

#### Real-World Cases

- **API responses**: `SELECT id AS user_id, username AS name` maps database columns to client-friendly field names.
- **Reports**: `SELECT SUM(amount) AS total_revenue` gives business users a clear header.

---

### 4. Table Aliases

#### Definitions

**Core Definition**
A table alias is a short name assigned to a table in the `FROM` clause, used to qualify column references and simplify queries.

**Technical Definition**
The syntax is `FROM table_name [AS] alias`. Once aliased, the original table name cannot be used to qualify columns in that query; only the alias is valid. Table aliases are essential for self-joins (joining a table to itself) and for shortening long table names.

**Beginner-Friendly Explanation**
A table alias is a nickname for a table in a query. It saves typing and makes self-joins possible.

#### Purposes

- To shorten long table names and improve readability.
- To allow self-joins by distinguishing multiple instances of the same table.
- To qualify columns unambiguously when multiple tables share column names.

#### Syntax Rules and Structure

```sql
SELECT a.column1, b.column2
FROM table1 AS a
JOIN table2 AS b ON a.id = b.id;
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `table1 AS a` | Assigns the alias `a` to `table1`. |
| `a.column1` | Qualifies `column1` with the alias. |

**Syntax rules:**

- `AS` is optional but recommended.
- Once aliased, the original table name is no longer valid in the query.
- Aliases can be used in `WHERE`, `GROUP BY`, and `ORDER BY` to qualify columns.

**Constraints and limitations:**

- Aliases are scoped to the query; they do not persist.
- In PostgreSQL, using the original table name after aliasing raises an error.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Self-join using table aliases

```sql
-- Step 1: Find employees and their managers in the same table.
SELECT e.first_name AS employee,
       m.first_name AS manager
FROM employees AS e
LEFT JOIN employees AS m ON e.manager_id = m.employee_id;
```

**Expected output:**

```text
 employee | manager
----------+---------
 Alice    | Carol
 Bob      | Carol
 Carol    |
(3 rows)
```

**Why this output appears**: The same `employees` table is aliased as `e` (employees) and `m` (managers). The join condition `e.manager_id = m.employee_id` links each employee to their manager. Without aliases, the query could not distinguish the two roles.

#### Real-World Cases

- **Self-referencing hierarchies**: Employee-manager relationships, category-subcategory trees, and comment-reply threads.
- **Multi-table joins**: Aliasing `orders` as `o` and `order_items` as `oi` keeps join conditions readable.

---

### 5. Expressions in SELECT

#### Definitions

**Core Definition**
An expression in `SELECT` is any combination of column references, literals, operators, and function calls that produces a single value for each row.

**Technical Definition**
A value expression can be a constant or literal, a column reference, an operator invocation, a function call, an aggregate expression, a type cast, a scalar subquery, or a parenthesized expression. The expression syntax allows the calculation of values from primitive parts using arithmetic, logical, and other operations.

**Beginner-Friendly Explanation**
An expression is anything in the `SELECT` list that is not just a column name—like `price * quantity`, `UPPER(name)`, or `salary + bonus`.

#### Purposes

- To derive new values from existing data without modifying the table.
- To apply business logic (tax calculations, formatting, classification) at query time.
- To combine data from multiple columns into a single output value.

#### Syntax Rules and Structure

```sql
SELECT expression1 [, expression2, ...]
FROM table_name;
```

**Component breakdown:**

| Expression type | Example |
|-----------------|---------|
| Literal | `'hello'` |
| Column reference | `salary` |
| Arithmetic | `salary * 1.10` |
| Function call | `UPPER(first_name)` |
| Type cast | `'2025-01-01'::date` |
| Parenthesized | `(salary + bonus) * 0.9` |

**Syntax rules:**

- Expressions can be nested and combined with operators.
- Operator precedence follows standard rules; use parentheses to override.
- String literals use single quotes; identifiers use double quotes.

**Constraints and limitations:**

- Division by zero raises an error; use `NULLIF` or `CASE` to guard against it.
- Integer division truncates the result; cast one operand to a decimal type to get a fractional result.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Mixing column references, literals, and arithmetic

```sql
-- Step 1: Compute a discounted price using a literal and arithmetic.
SELECT product_name,
       price,
       price * 0.90 AS discounted_price,
       'USD' AS currency
FROM products;
```

**Expected output:**

```text
 product_name | price | discounted_price | currency
--------------+-------+------------------+----------
 Widget       | 19.99 |           17.991 | USD
 Gadget       | 29.95 |           26.955 | USD
(2 rows)
```

**Why this output appears**: The expression `price * 0.90` computes a 10% discount, the literal `'USD'` adds a constant column, and `AS` gives the computed columns readable names.

#### Real-World Cases

- **Pricing engines**: `price * (1 + tax_rate) AS price_with_tax` computes tax-inclusive prices.
- **Data validation**: `CASE WHEN age >= 18 THEN 'adult' ELSE 'minor' END AS age_group` classifies rows inline.

---

### 6. Literals

#### Definitions

**Core Definition**
A literal is a constant value written directly in the SQL text, such as a number, a quoted string, or a date constant.

**Technical Definition**
String literals are enclosed in single quotes (`'hello'`). Numeric literals are written as plain numbers (`42`, `3.14`). Boolean literals are `TRUE` and `FALSE`. Date and time literals can be written as quoted strings and cast explicitly (e.g., `'2025-01-01'::date`). A literal can stand alone in the `SELECT` list without a `FROM` clause, acting as a calculator. Single quotes are for string values; double quotes are for identifiers, and mixing them up is a common beginner error.

**Beginner-Friendly Explanation**
A literal is a fixed value you type into the query—like `'active'`, `100`, or `'2025-01-01'`. It does not come from a table; you supply it directly.

#### Purposes

- To supply constant values for comparisons, classifications, or output columns.
- To label results (e.g., `'USD' AS currency`).
- To test expressions without querying a table (`SELECT 3 * 4`).

#### Syntax Rules and Structure

| Literal type | Example | Notes |
|--------------|---------|-------|
| String | `'hello'` | Single quotes |
| Number | `42`, `3.14` | No quotes |
| Boolean | `TRUE`, `FALSE` | Keywords |
| Date | `'2025-01-01'::date` | Quoted string + cast |
| NULL | `NULL` | Absence of value |

**Syntax rules:**

- String literals must use single quotes; double quotes denote identifiers.
- To include a single quote inside a string, double it (`'it''s'`).
- Numeric literals do not use quotes.

**Constraints and limitations:**

- A bare `NULL` may need a type cast in some contexts.
- Date literals without an explicit cast may be interpreted differently depending on the database’s date format settings.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Using literals as output columns

```sql
-- Step 1: Add constant labels and a computed value.
SELECT product_name,
       'On Sale' AS promotion,
       0.15 AS discount_rate
FROM products
WHERE on_sale = TRUE;
```

**Expected output:**

```text
 product_name | promotion | discount_rate
--------------+-----------+---------------
 Widget       | On Sale   |          0.15
(1 row)
```

**Why this output appears**: The literals `'On Sale'` and `0.15` are returned as constant columns for every matching row, alongside the table’s `product_name`.

#### Real-World Cases

- **Data tagging**: Adding a literal `'imported' AS source` to distinguish rows in a union.
- **Configuration queries**: `SELECT 'v1.2.3' AS app_version` returns a constant without touching a table.

---

### 7. Calculated Columns

#### Definitions

**Core Definition**
A calculated column (also called a computed column or derived column) is an output column whose value is computed from an expression rather than read directly from a table column.

**Technical Definition**
A calculated column appears in the `SELECT` list as an expression involving arithmetic operators, functions, or concatenation. It is not stored in the table; it exists only in the query result. A computed column receives an auto-generated name unless aliased with `AS`. Calculations can use numeric columns or constants with arithmetic operators, functions, conversions, or nested queries.

**Beginner-Friendly Explanation**
A calculated column is a value the database computes on the fly—like multiplying price by quantity, or combining first and last names into a full name.

#### Purposes

- To derive metrics (revenue, tax, age, duration) from stored data.
- To format or transform values for display without altering the base table.
- To combine multiple columns into a single, more useful output column.

#### Syntax Rules and Structure

```sql
SELECT column1,
       expression AS calculated_column_name
FROM table_name;
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `expression` | Arithmetic, function call, or concatenation. |
| `AS calculated_column_name` | Readable alias for the computed value. |

**Syntax rules:**

- Calculated columns can reference any column in the `FROM` clause.
- They can nest: an expression can contain other expressions.
- Always alias a calculated column to avoid auto-generated names.

**Constraints and limitations:**

- Calculated columns cannot be referenced in `WHERE` because `WHERE` runs before `SELECT`.
- Integer division truncates; cast to a decimal type for fractional results.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Calculated columns for salary metrics

```sql
-- Step 1: Compute annual salary and monthly pay.
SELECT first_name,
       salary,
       salary * 12 AS annual_salary,
       salary / 12 AS monthly_pay
FROM employees;
```

**Expected output:**

```text
 first_name | salary | annual_salary | monthly_pay
------------+--------+---------------+-------------
 Alice      |  80000 |        960000 |        6666
 Bob        |  70000 |        840000 |        5833
(2 rows)
```

**Why this output appears**: `salary * 12` and `salary / 12` are computed for each row. The integer division `salary / 12` truncates the fractional part; casting to `numeric` would preserve it.

#### Real-World Cases

- **E-commerce**: `price * quantity AS line_total` computes order line totals.
- **HR**: `DATEDIFF(year, hire_date, CURRENT_DATE) AS years_of_service` computes tenure.

---

### 8. Computed Columns

#### Definitions

**Core Definition**
A computed column is a synonym for a calculated column—an output column derived from an expression rather than stored data.

**Technical Definition**
In the context of `SELECT`, “computed column” and “calculated column” are used interchangeably to describe an expression in the projection list that produces a derived value. The term is also used in DDL for stored generated columns, but in query context it refers to runtime computation.

**Beginner-Friendly Explanation**
Same as calculated column: a column whose value is computed on the fly.

#### Purposes

- To derive values without persisting them.
- To experiment with formulas before adding a generated column to the table.

#### Syntax Rules and Structure

Identical to calculated columns.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Computed column for a full name

```sql
-- Step 1: Combine first and last names.
SELECT first_name || ' ' || last_name AS full_name,
       salary * 1.10 AS salary_after_raise
FROM employees;
```

**Expected output:**

```text
   full_name    | salary_after_raise
----------------+--------------------
 Alice Smith    |              88000
 Bob Jones      |              77000
(2 rows)
```

**Why this output appears**: The concatenation `first_name || ' ' || last_name` and the multiplication `salary * 1.10` are computed per row. Aliases give the results meaningful names.

#### Real-World Cases

- **CRM**: `first_name || ' ' || last_name AS full_name` for display.
- **Finance**: `amount * exchange_rate AS amount_usd` for currency conversion.

---

### 9. Concatenation

#### Definitions

**Core Definition**
Concatenation is the operation of joining two or more strings end-to-end into a single string.

**Technical Definition**
In PostgreSQL and SQLite, the SQL-standard operator `||` concatenates strings. In MySQL, `CONCAT()` is the standard function. In SQL Server, `+` concatenates strings (and `CONCAT()` is also available). Concatenation can involve column references, literals, and the results of other expressions.

**Beginner-Friendly Explanation**
Concatenation is how you glue text together—like joining a first name and a last name with a space in between.

#### Purposes

- To build full names, addresses, or display labels from separate columns.
- To add prefixes or suffixes to values.
- To generate formatted codes or identifiers.

#### Syntax Rules and Structure

| RDBMS | Operator/Function | Example |
|-------|-------------------|---------|
| PostgreSQL | `||` | `first_name || ' ' || last_name` |
| MySQL | `CONCAT()` | `CONCAT(first_name, ' ', last_name)` |
| SQL Server | `+` or `CONCAT()` | `first_name + ' ' + last_name` |

**Syntax rules:**

- `||` treats `NULL` as an empty string in some databases, but in PostgreSQL, `NULL || 'x'` returns `NULL`; use `COALESCE` to handle nulls.
- In SQL Server, `+` with `NULL` returns `NULL`; `CONCAT()` treats `NULL` as an empty string.

**Constraints and limitations:**

- Concatenating a non-string value may require an explicit cast.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Building a full name with concatenation

```sql
-- Step 1: Concatenate first and last names with a space.
SELECT first_name || ' ' || last_name AS full_name
FROM employees;
```

**Expected output:**

```text
   full_name
----------------
 Alice Smith
 Bob Jones
(2 rows)
```

**Why this output appears**: The `||` operator joins the three string values—`first_name`, the literal space `' '`, and `last_name`—into a single string.

#### Real-World Cases

- **Address formatting**: `street || ', ' || city || ', ' || state AS full_address`.
- **Product labels**: `brand || ' ' || model AS product_label`.

---

### 10. Arithmetic Operations

#### Definitions

**Core Definition**
Arithmetic operations are the mathematical calculations—addition, subtraction, multiplication, division, and modulo—performed on numeric values in expressions.

**Technical Definition**
SQL supports the binary operators `+`, `-`, `*`, `/`, and `%` (modulo). These operators follow standard precedence rules (multiplication and division before addition and subtraction), and parentheses can override precedence. Integer division truncates the fractional part; to obtain a fractional result, at least one operand must be of a decimal or floating-point type.

**Beginner-Friendly Explanation**
Arithmetic operations let you do math in your queries—like computing totals, averages, percentages, or differences.

#### Purposes

- To compute derived numeric values (totals, differences, percentages).
- To apply scaling factors (tax, discount, currency conversion).
- To compare values after arithmetic transformation.

#### Syntax Rules and Structure

| Operator | Description | Example |
|----------|-------------|---------|
| `+` | Addition | `price + tax` |
| `-` | Subtraction | `salary - deductions` |
| `*` | Multiplication | `quantity * unit_price` |
| `/` | Division | `total / count` |
| `%` | Modulo | `id % 10` |

**Syntax rules:**

- Standard precedence: `*` and `/` before `+` and `-`.
- Use parentheses to group operations: `(a + b) * c`.
- Integer division truncates; cast to `numeric` or `decimal` for fractional results.

**Constraints and limitations:**

- Division by zero raises an error; guard with `NULLIF(divisor, 0)` or `CASE`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Arithmetic with integer vs. numeric division

```sql
-- Step 1: Compare integer division with numeric division.
SELECT 10 / 4 AS integer_division,
       10 / 4.0 AS numeric_division,
       salary * 1.10 AS raised_salary
FROM employees
LIMIT 1;
```

**Expected output:**

```text
 integer_division | numeric_division | raised_salary
------------------+------------------+---------------
                2 |             2.50 |         88000
(1 row)
```

**Why this output appears**: `10 / 4` performs integer division and truncates to `2`. `10 / 4.0` promotes to numeric division and returns `2.50`. The `salary * 1.10` expression multiplies the numeric salary by the decimal factor.

#### Real-World Cases

- **Order totals**: `quantity * unit_price AS line_total`.
- **Financial ratios**: `revenue / NULLIF(expenses, 0) AS roi`.

---

### 11. String Functions

#### Definitions

**Core Definition**
String functions are built-in functions that manipulate character data—changing case, extracting substrings, trimming whitespace, replacing text, and measuring length.

**Technical Definition**
Common string functions include `UPPER()`, `LOWER()`, `LENGTH()`, `SUBSTRING()` (or `SUBSTR()`), `TRIM()`, `LTRIM()`, `RTRIM()`, `REPLACE()`, and `CONCAT()`. Availability and exact names vary by RDBMS; for example, SQL Server uses `LEN()` instead of `LENGTH()`, and MySQL uses `SUBSTRING()` and `LOCATE()`.

**Beginner-Friendly Explanation**
String functions let you clean up and reshape text—capitalize names, extract the first three letters of a code, or remove extra spaces.

#### Purposes

- To normalize text for comparison or display.
- To extract meaningful parts of a string (area codes, file extensions).
- To clean imported data (trimming whitespace, replacing characters).

#### Syntax Rules and Structure

| Function | PostgreSQL | MySQL | SQL Server |
|----------|------------|-------|------------|
| Uppercase | `UPPER(s)` | `UPPER(s)` | `UPPER(s)` |
| Lowercase | `LOWER(s)` | `LOWER(s)` | `LOWER(s)` |
| Length | `LENGTH(s)` | `LENGTH(s)` | `LEN(s)` |
| Substring | `SUBSTRING(s FROM start FOR len)` | `SUBSTRING(s, start, len)` | `SUBSTRING(s, start, len)` |
| Trim | `TRIM(s)` | `TRIM(s)` | `TRIM(s)` |
| Replace | `REPLACE(s, from, to)` | `REPLACE(s, from, to)` | `REPLACE(s, from, to)` |

**Syntax rules:**

- String functions accept column references, literals, or nested expressions.
- Functions can be nested: `UPPER(TRIM(name))`.

**Constraints and limitations:**

- String indexing is 1-based in standard SQL.
- `LENGTH` counts characters in PostgreSQL and bytes in some MySQL configurations; use `CHAR_LENGTH` for characters.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Normalizing and extracting string data

```sql
-- Step 1: Normalize email and extract the domain.
SELECT email,
       LOWER(email) AS normalized_email,
       SUBSTRING(email FROM POSITION('@' IN email) + 1) AS domain
FROM users;
```

**Expected output:**

```text
       email        | normalized_email |   domain
--------------------+------------------+------------
 Alice@Example.COM  | alice@example.com| example.com
 Bob@Example.COM    | bob@example.com  | example.com
(2 rows)
```

**Why this output appears**: `LOWER` normalizes the case, and `SUBSTRING` with `POSITION` extracts everything after the `@` symbol, producing the domain.

#### Real-World Cases

- **Data cleaning**: `TRIM(UPPER(country_code))` normalizes imported country codes.
- **Display formatting**: `INITCAP(first_name)` capitalizes names for display.

---

### 12. Numeric Functions

#### Definitions

**Core Definition**
Numeric functions are built-in functions that operate on numbers—rounding, absolute value, powers, square roots, and random number generation.

**Technical Definition**
Common numeric functions include `ROUND()`, `ABS()`, `CEIL()` (or `CEILING()`), `FLOOR()`, `POWER()` (or `POW()`), `SQRT()`, and `RANDOM()` (PostgreSQL) or `RAND()` (MySQL, SQL Server). SQL Server also provides `STDEV`, `VAR`, and other statistical functions.

**Beginner-Friendly Explanation**
Numeric functions let you do math transformations—round a price to two decimals, take the absolute value of a difference, or compute a square root.

#### Purposes

- To round monetary values to a fixed number of decimal places.
- To compute magnitudes (absolute differences, distances).
- To generate random samples or test data.

#### Syntax Rules and Structure

| Function | PostgreSQL | MySQL | SQL Server |
|----------|------------|-------|------------|
| Round | `ROUND(n, d)` | `ROUND(n, d)` | `ROUND(n, d)` |
| Absolute | `ABS(n)` | `ABS(n)` | `ABS(n)` |
| Ceiling | `CEIL(n)` | `CEILING(n)` | `CEILING(n)` |
| Floor | `FLOOR(n)` | `FLOOR(n)` | `FLOOR(n)` |
| Power | `POWER(n, p)` | `POWER(n, p)` | `POWER(n, p)` |
| Square root | `SQRT(n)` | `SQRT(n)` | `SQRT(n)` |
| Random | `RANDOM()` | `RAND()` | `RAND()` |

**Syntax rules:**

- `ROUND` with a second argument specifies decimal places.
- `POWER` and `SQRT` return numeric results.

**Constraints and limitations:**

- `RANDOM()` returns a value between 0 and 1; scale it for other ranges.
- Rounding behaviour (half-up vs. half-even) can vary slightly by RDBMS.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Rounding and absolute difference

```sql
-- Step 1: Round prices and compute absolute variance.
SELECT price,
       ROUND(price, 1) AS rounded_price,
       ABS(price - 20.00) AS deviation_from_20
FROM products;
```

**Expected output:**

```text
 price | rounded_price | deviation_from_20
-------+---------------+-------------------
 19.99 |          20.0 |              0.01
 29.95 |          30.0 |              9.95
(2 rows)
```

**Why this output appears**: `ROUND(price, 1)` rounds to one decimal place, and `ABS(price - 20.00)` computes the absolute difference from 20.

#### Real-World Cases

- **Financial reports**: `ROUND(amount, 2)` ensures monetary values show two decimal places.
- **Quality control**: `ABS(measured - target) AS error` quantifies deviation.

---

### 13. Date Functions

#### Definitions

**Core Definition**
Date functions are built-in functions that extract components from dates, perform date arithmetic, format dates, and obtain the current date/time.

**Technical Definition**
Common date functions include `CURRENT_DATE`, `CURRENT_TIMESTAMP`, `EXTRACT(field FROM date)`, `DATE_TRUNC(unit, date)` (PostgreSQL), `DATEADD(unit, n, date)` (SQL Server), `DATEDIFF(unit, start, end)` (SQL Server, MySQL), and `TO_CHAR(date, format)` (PostgreSQL) or `DATE_FORMAT(date, format)` (MySQL). SQL Server provides `GETDATE()`, `DATEPART()`, `DATENAME()`, and `DATEADD()`.

**Beginner-Friendly Explanation**
Date functions let you ask questions like “what month was this order placed?” or “how many days between these two dates?” or “what is today’s date?”.

#### Purposes

- To group or filter data by date components (year, month, day).
- To compute durations and ages.
- To format dates for display or export.
- To obtain the current date/time for auditing or filtering.

#### Syntax Rules and Structure

| Function | PostgreSQL | MySQL | SQL Server |
|----------|------------|-------|------------|
| Current date | `CURRENT_DATE` | `CURDATE()` | `CAST(GETDATE() AS DATE)` |
| Current timestamp | `CURRENT_TIMESTAMP` | `NOW()` | `GETDATE()` |
| Extract year | `EXTRACT(YEAR FROM d)` | `YEAR(d)` | `YEAR(d)` |
| Add interval | `d + INTERVAL '1 day'` | `DATE_ADD(d, INTERVAL 1 DAY)` | `DATEADD(day, 1, d)` |
| Difference | `d2 - d1` | `DATEDIFF(d2, d1)` | `DATEDIFF(day, d1, d2)` |
| Format | `TO_CHAR(d, 'YYYY-MM')` | `DATE_FORMAT(d, '%Y-%m')` | `FORMAT(d, 'yyyy-MM')` |

**Syntax rules:**

- `EXTRACT` returns a numeric component (e.g., year as an integer).
- `DATE_TRUNC` truncates a timestamp to a specified precision (e.g., month).
- Date arithmetic returns an interval in PostgreSQL.

**Constraints and limitations:**

- Date formats vary by RDBMS and session settings.
- Time zone handling differs; use `timestamptz` in PostgreSQL for absolute instants.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Extracting date components and formatting

```sql
-- Step 1: Extract the order month and format the order date.
SELECT order_id,
       EXTRACT(YEAR FROM order_date) AS order_year,
       EXTRACT(MONTH FROM order_date) AS order_month,
       TO_CHAR(order_date, 'YYYY-MM') AS year_month
FROM orders;
```

**Expected output:**

```text
 order_id | order_year | order_month | year_month
----------+------------+-------------+------------
      101 |       2025 |           1 | 2025-01
      102 |       2025 |           2 | 2025-02
(2 rows)
```

**Why this output appears**: `EXTRACT` pulls the year and month as integers, and `TO_CHAR` formats the date as a `YYYY-MM` string for grouping or display.

#### Real-World Cases

- **Monthly reporting**: `DATE_TRUNC('month', order_date) AS month` groups sales by month.
- **Age calculation**: `EXTRACT(YEAR FROM AGE(birthdate)) AS age` computes age in PostgreSQL.

---

### 14. Type Casting

#### Definitions

**Core Definition**
Type casting is the explicit conversion of a value from one data type to another.

**Technical Definition**
PostgreSQL supports two equivalent cast syntaxes: `CAST(expression AS type)` (SQL standard) and `expression::type` (PostgreSQL shorthand). SQL Server provides `CAST(expression AS datatype)` and `CONVERT(datatype, expression [, style])`; `CONVERT` allows style codes for date formatting. MySQL also supports `CAST(expression AS type)` and `CONVERT()`.

**Beginner-Friendly Explanation**
Casting is how you tell the database “treat this value as a different type”—like turning the string `'2025-01-01'` into a real date, or an integer into a decimal for division.

#### Purposes

- To convert imported text data into proper numeric or date types.
- To control division behaviour (integer vs. decimal).
- To format output (e.g., date to string with a specific pattern).
- To satisfy function signatures that require a specific type.

#### Syntax Rules and Structure

| RDBMS | Standard cast | Alternative |
|-------|---------------|-------------|
| PostgreSQL | `CAST(x AS type)` | `x::type` |
| MySQL | `CAST(x AS type)` | `CONVERT(x, type)` |
| SQL Server | `CAST(x AS type)` | `CONVERT(type, x [, style])` |

**Syntax rules:**

- Casting a value that cannot be converted raises an error (e.g., `'abc'::integer`).
- Use `TRY_CAST` (SQL Server) or `NULLIF` to handle unsafe conversions gracefully.

**Constraints and limitations:**

- Implicit casts happen automatically in some contexts; explicit casts make intent clear.
- Casting can change precision or truncate values (e.g., `1.9::integer` → `1`).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Casting for division and date conversion

```sql
-- Step 1: Cast an integer to numeric for fractional division.
SELECT 10::numeric / 4 AS fractional_result,
       '2025-01-15'::date AS parsed_date;
```

**Expected output:**

```text
 fractional_result | parsed_date
-------------------+-------------
              2.50 | 2025-01-15
(1 row)
```

**Why this output appears**: `10::numeric` converts the integer to a numeric type, so division produces `2.50` instead of the truncated integer `2`. `'2025-01-15'::date` converts the string literal to a date value.

#### Real-World Cases

- **Import cleanup**: `'42'::integer` converts text-stored numbers to integers.
- **API output**: `created_at::text` converts a timestamp to a string for JSON serialization.

---

### 15. Query Readability and Formatting

#### Definitions

**Core Definition**
Query readability and formatting are the conventions—capitalization, indentation, aliasing, and clause ordering—that make SQL code easy to read, review, and maintain.

**Technical Definition**
Clean SQL is built on clarity, consistency, minimalism, and intent-driven syntax. Recommended conventions include: uppercase for SQL keywords (`SELECT`, `FROM`, `WHERE`, `AS`); consistent indentation (typically 4 spaces); each major clause on its own line; aliases with `AS`; and avoiding `SELECT *` in production. Formatting is not merely aesthetic—poorly formatted queries slow down reviews, obscure bugs, and accumulate technical debt.

**Beginner-Friendly Explanation**
Formatting is how you make your SQL easy for humans to read. Databases do not care about whitespace or capitalization, but your teammates (and future you) do.

#### Purposes

- To make queries easy to scan and understand at a glance.
- To speed up code reviews and debugging.
- To reduce the risk of subtle errors caused by misreading dense code.
- To maintain a consistent style across a team or project.

#### Syntax Rules and Structure

**Recommended formatting pattern:**

```sql
SELECT
    e.first_name,
    e.last_name,
    e.salary * 12 AS annual_salary
FROM
    employees AS e
WHERE
    e.department = 'Engineering'
ORDER BY
    annual_salary DESC;
```

**Component breakdown:**

| Convention | Example |
|------------|---------|
| Uppercase keywords | `SELECT`, `FROM`, `WHERE` |
| Each column on its own line | One column per line in the `SELECT` list |
| Indentation | 4 spaces for continuation lines |
| Aliases with `AS` | `salary * 12 AS annual_salary` |
| Clause per line | `FROM` on its own line after the select list |

**Syntax rules:**

- These are conventions, not requirements; the database ignores formatting.
- Consistency is more important than any particular convention.

**Constraints and limitations:**

- Overly rigid formatting rules can become bureaucratic; prioritize readability over dogma.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Unformatted vs. formatted query

```sql
-- Unformatted:
select a.id,a.name,b.total from orders a join payments b on a.id=b.orderid where b.total>1000;

-- Formatted:
SELECT
    a.id,
    a.name,
    b.total
FROM
    orders AS a
JOIN
    payments AS b ON a.id = b.orderid
WHERE
    b.total > 1000;
```

**Expected output:** (identical for both; formatting does not change the result.)

**Why this output appears**: The database parses both queries identically. The formatted version is easier for a human to read, review, and debug—it communicates intent instantly.

#### Real-World Cases

- **Team codebases**: A shared style guide ensures that any developer can read and modify any query.
- **Code reviews**: Well-formatted SQL reduces review time and catches logical errors more easily.

---

## References

- PostgreSQL Documentation – Value Expressions – https://www.postgresql.org/docs/current/sql-expressions.html 
- PostgreSQL Documentation – SELECT (Value Expressions) – https://www.postgresql.org/docs/current/sql-select.html
- Postgres by Example – SELECT Basics – https://github.com/boringcollege/postgres-by-example/blob/main/chapters/03-select-basics.md 
- MySQL Reference Manual – SELECT Statement – https://dev.mysql.com/doc/refman/8.0/en/select.html 
- SQL Server Cheat Sheet – String, Date, and Conversion Functions – https://elearn.nmc.edu/pluginfile.php/2675246/mod_resource/content/3/sql-server-cheat-sheet.pdf 
- SQL Server – Computed Values in the Select List – https://learn.microsoft.com/en-us/previous-versions/sql/sql-server-2008-r2/ms190034(v=sql.105) 
- Lecture 2 — SELECT, Deep (Expressions, Computed Columns, Aliases) – https://raw.githubusercontent.com/CODECRUNCHWORLDWIDE/C33-CRUNCH-SQL/main/curriculum/week-01-relational-model-and-select/lecture-notes/02-select-deep.md 
- Devart – Best Practices for SQL Formatting – https://www.devart.com/blog/sql-formatting-best-practices.html 
- SQL Style Guide – https://github.com/treffynnon/sqlstyle.guide 