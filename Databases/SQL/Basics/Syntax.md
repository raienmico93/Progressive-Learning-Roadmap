# SQL Syntax Fundamentals

Before writing queries, you need to understand the building blocks of SQL syntax — how statements are structured, what keywords and identifiers mean, how literals and operators work, and the conventions that keep code readable and portable.

---

## 1. SQL Statement Structure

An **SQL statement** is a complete instruction to the database. Every statement follows a general pattern:

```
[clause] [clause] [clause] ... ;
```

### Anatomy of a Statement

```sql
SELECT   first_name, last_name       -- SELECT clause
FROM     employees                   -- FROM clause
WHERE    department_id = 5           -- WHERE clause
ORDER BY last_name ASC;              -- ORDER BY clause
```

Each **clause** begins with a **keyword** and is followed by **expressions**, **identifiers**, or **literals**.

### General Form of Common Statements

| Statement Type | General Form |
|---|---|
| **SELECT** | `SELECT columns FROM table WHERE condition ORDER BY ...` |
| **INSERT** | `INSERT INTO table (columns) VALUES (...)` |
| **UPDATE** | `UPDATE table SET column = value WHERE condition` |
| **DELETE** | `DELETE FROM table WHERE condition` |
| **CREATE TABLE** | `CREATE TABLE name (column type constraints, ...)` |
| **ALTER TABLE** | `ALTER TABLE name ADD/MODIFY/DROP ...` |
| **DROP** | `DROP TABLE name` |

### Clause Order Matters

The order of clauses is **fixed** for each statement type. In a `SELECT`, the order is:

```
SELECT   → what to return
FROM     → where the data is
WHERE    → which rows to keep
GROUP BY → how to aggregate
HAVING   → which groups to keep
ORDER BY → how to sort
LIMIT    → how many to return
```

**Wrong:**
```sql
FROM employees
SELECT first_name;  -- ✗ clauses out of order
```

**Right:**
```sql
SELECT first_name
FROM employees;
```

### Whitespace and Line Breaks

SQL ignores extra whitespace and newlines. These are equivalent:

```sql
SELECT first_name, last_name FROM employees WHERE department_id = 5;
```

```sql
SELECT
    first_name,
    last_name
FROM
    employees
WHERE
    department_id = 5;
```

The second form is preferred for readability.

---

## 2. Keywords

**Keywords** are reserved words that have special meaning in SQL. They define the structure and operations of a statement.

### Categories of Keywords

| Category | Examples |
|---|---|
| **Clauses** | `SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY` |
| **Operators** | `AND`, `OR`, `NOT`, `IN`, `LIKE`, `BETWEEN`, `IS` |
| **Joins** | `JOIN`, `INNER`, `LEFT`, `RIGHT`, `FULL`, `ON`, `USING` |
| **Set operations** | `UNION`, `INTERSECT`, `EXCEPT` |
| **DML** | `INSERT`, `UPDATE`, `DELETE`, `MERGE` |
| **DDL** | `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME` |
| **DCL** | `GRANT`, `REVOKE` |
| **TCL** | `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` |
| **Constraints** | `PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `CHECK`, `NOT NULL`, `DEFAULT` |
| **Data types** | `INTEGER`, `VARCHAR`, `DATE`, `BOOLEAN`, `NUMERIC` |
| **Modifiers** | `DISTINCT`, `ALL`, `ASC`, `DESC`, `NULLS FIRST`, `NULLS LAST` |

### Reserved vs. Non-Reserved

| Type | Description | Example |
|---|---|---|
| **Reserved** | Cannot be used as identifiers unless quoted | `SELECT`, `FROM`, `WHERE` |
| **Non-reserved** | Keywords with contextual meaning; often usable as identifiers | `YEAR`, `NAME` (varies by DBMS) |

**Avoid using keywords as identifiers** — even non-reserved ones — to prevent confusion and portability issues.

### Example

```sql
SELECT DISTINCT department_id
FROM employees
WHERE salary > 50000
ORDER BY department_id;
```

Keywords here: `SELECT`, `DISTINCT`, `FROM`, `WHERE`, `ORDER BY`.

### Case of Keywords

Keywords are **case-insensitive** in standard SQL:

```sql
select * from employees;
SELECT * FROM employees;
Select * From Employees;
```

All three are equivalent. **Convention:** write keywords in **UPPERCASE** for readability.

---

## 3. Identifiers

**Identifiers** are names given to database objects — tables, columns, schemas, indexes, views, etc.

### Types of Identifiers

| Object | Example Identifier |
|---|---|
| Table | `customers` |
| Column | `first_name` |
| Schema | `public`, `sales` |
| Database | `shop` |
| Index | `idx_customers_email` |
| Constraint | `fk_orders_customer` |
| View | `active_customers` |
| Alias | `c`, `total_sales` |

### Rules for Unquoted Identifiers

Rules vary slightly by DBMS, but the general pattern is:

| Rule | Standard | PostgreSQL | MySQL | SQL Server | Oracle |
|---|---|---|---|---|---|
| Starts with letter | ✓ | ✓ | ✓ (or `_`) | ✓ (or `_`, `@`, `#`) | ✓ |
| Letters, digits, `_` | ✓ | ✓ | ✓ | ✓ | ✓ |
| Case-sensitive | No | No (folds to lower) | Depends on OS | Depends on collation | No (folds to upper) |
| Max length | 128 | 63 | 64 | 128 | 30 (old) / 128 (new) |
| Reserved words allowed | ✗ | ✗ | ✗ | ✗ | ✗ |

### Quoted Identifiers

To use spaces, special characters, or reserved words, **quote the identifier**:

| DBMS | Quote Style | Example |
|---|---|---|
| **Standard SQL** | Double quotes | `"first name"` |
| **PostgreSQL** | Double quotes | `"first name"` |
| **Oracle** | Double quotes | `"first name"` |
| **SQL Server** | Brackets or double quotes | `[first name]` or `"first name"` |
| **MySQL / MariaDB** | Backticks (or double quotes in ANSI mode) | `` `first name` `` |

### Example

```sql
-- Unquoted (simple)
SELECT first_name FROM customers;

-- Quoted (spaces or reserved words)
SELECT "first name" FROM "user";   -- standard / PostgreSQL
SELECT [first name] FROM [user];   -- SQL Server
SELECT `first name` FROM `user`;   -- MySQL
```

### Qualified Identifiers

Identifiers can be qualified with their parent object:

```sql
schema_name.table_name
database_name.schema_name.table_name
table_name.column_name
```

**Examples:**
```sql
SELECT public.customers.email FROM public.customers;
SELECT c.email FROM customers AS c;
SELECT shop.public.customers.email FROM shop.public.customers;  -- SQL Server
```

### Aliases

**Aliases** give temporary names to tables or columns in a query:

```sql
SELECT c.first_name AS given_name, c.last_name AS surname
FROM customers AS c;
```

| Rule | Description |
|---|---|
| Use `AS` (optional in most DBMS) | `AS alias` or just `alias` |
| Column aliases affect output headers | Shown in result set |
| Table aliases simplify joins | `c.first_name` instead of `customers.first_name` |
| Aliases last only for the query | Not persisted |

---

## 4. Literals

A **literal** is a fixed value written directly in SQL code.

### Types of Literals

| Type | Syntax | Example |
|---|---|---|
| **String** | Single quotes | `'Hello'`, `'O''Brien'` |
| **Numeric** | Digits, optional sign, decimal point | `42`, `-3.14`, `1.5e3` |
| **Boolean** | `TRUE` / `FALSE` | `TRUE`, `FALSE` |
| **Date** | `DATE 'YYYY-MM-DD'` (standard) or `'YYYY-MM-DD'` | `DATE '2026-01-15'` |
| **Time** | `TIME 'HH:MM:SS'` | `TIME '14:30:00'` |
| **Timestamp** | `TIMESTAMP 'YYYY-MM-DD HH:MM:SS'` | `TIMESTAMP '2026-01-15 14:30:00'` |
| **Interval** | `INTERVAL 'n unit'` | `INTERVAL '3 days'` |
| **NULL** | Keyword (not a literal per se) | `NULL` |
| **Binary** | Hex or escape syntax | `X'1F2A'`, `0x1F2A` |
| **Unicode string** | Prefixed `N` (SQL Server) | `N'Café'` |

### String Literals

Strings use **single quotes** in standard SQL:

```sql
SELECT 'Hello, World!';
INSERT INTO customers (name) VALUES ('John');
```

**Escaping single quotes:** double them.

```sql
INSERT INTO customers (name) VALUES ('O''Brien');
```

**Double quotes are NOT string delimiters** — they are for identifiers:

```sql
SELECT 'name';   -- ✓ string literal
SELECT "name";   -- ✗ identifier (column or table named "name")
```

### Numeric Literals

```sql
SELECT 42;
SELECT -3.14;
SELECT 1.5e3;    -- 1500.0 in scientific notation
```

No quotes for numeric values:

```sql
INSERT INTO products (price) VALUES (19.99);      -- ✓ numeric
INSERT INTO products (price) VALUES ('19.99');    -- ✗ string (may convert, may not)
```

### Date and Time Literals

**Standard SQL:**
```sql
SELECT DATE '2026-01-15';
SELECT TIME '14:30:00';
SELECT TIMESTAMP '2026-01-15 14:30:00';
```

**DBMS-specific:**
```sql
-- PostgreSQL
SELECT '2026-01-15'::date;
SELECT CAST('2026-01-15' AS DATE);

-- MySQL
SELECT DATE('2026-01-15');
SELECT STR_TO_DATE('01/15/2026', '%m/%d/%Y');

-- SQL Server
SELECT CAST('2026-01-15' AS DATE);
SELECT CONVERT(DATE, '2026-01-15');
```

Always prefer the **ISO 8601 format** (`YYYY-MM-DD`) — it's unambiguous.

### NULL Literal

`NULL` is the **absence of a value** — not zero, not empty string.

```sql
INSERT INTO customers (name, email) VALUES ('John', NULL);
SELECT * FROM customers WHERE email IS NULL;
```

### Boolean Literals

```sql
SELECT TRUE;
SELECT FALSE;
UPDATE products SET in_stock = TRUE WHERE id = 1;
```

Some DBMS use `1`/`0` instead (MySQL historically, SQL Server with `BIT`).

---

## 5. Operators

**Operators** perform operations on values. SQL has several categories.

### Arithmetic Operators

| Operator | Meaning | Example |
|---|---|---|
| `+` | Addition | `salary + 1000` |
| `-` | Subtraction | `salary - 100` |
| `*` | Multiplication | `price * quantity` |
| `/` | Division | `total / 12` |
| `%` | Modulo (varies) | `id % 2` |
| `^` | Exponentiation (varies) | `2 ^ 3` |

### Comparison Operators

| Operator | Meaning |
|---|---|
| `=` | Equal |
| `<>` or `!=` | Not equal |
| `<`, `<=`, `>`, `>=` | Less / less-or-equal / greater / greater-or-equal |
| `<=>` | NULL-safe equal (MySQL) |
| `IS NULL`, `IS NOT NULL` | NULL checks |
| `IS DISTINCT FROM` | NULL-safe inequality (standard) |

### Logical Operators

| Operator | Meaning |
|---|---|
| `AND` | Both conditions true |
| `OR` | Either condition true |
| `NOT` | Negation |

**Three-valued logic:** SQL uses `TRUE`, `FALSE`, and `UNKNOWN` (for NULLs).

```sql
SELECT * FROM customers WHERE name = 'John' AND email IS NOT NULL;
```

### Pattern Matching

| Operator | Meaning | Example |
|---|---|---|
| `LIKE` | Wildcard match (`%` any, `_` one) | `name LIKE 'J%'` |
| `ILIKE` | Case-insensitive (PostgreSQL) | `name ILIKE 'j%'` |
| `SIMILAR TO` | SQL regex (PostgreSQL) | `name SIMILAR TO 'J(ohn|ane)'` |
| `~` / `~*` | Regex match (PostgreSQL) | `name ~ '^J'` |
| `REGEXP` | Regex (MySQL) | `name REGEXP '^J'` |

### Set / Membership Operators

| Operator | Meaning |
|---|---|
| `IN` | Value in a list or subquery |
| `NOT IN` | Value not in a list |
| `BETWEEN ... AND ...` | Inclusive range |
| `EXISTS` | Subquery returns rows |
| `ANY` / `SOME` | True for at least one |
| `ALL` | True for all |

### String Concatenation

| DBMS | Operator |
|---|---|
| **Standard / PostgreSQL / Oracle** | `||` |
| **MySQL** | `CONCAT()` or `||` (if PIPES_AS_CONCAT) |
| **SQL Server** | `+` or `CONCAT()` |

```sql
-- PostgreSQL / standard
SELECT first_name || ' ' || last_name AS full_name FROM employees;

-- MySQL
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employees;

-- SQL Server
SELECT first_name + ' ' + last_name AS full_name FROM employees;
```

### Bitwise Operators

| Operator | Meaning |
|---|---|
| `&` | Bitwise AND |
| `|` | Bitwise OR |
| `^` | Bitwise XOR |
| `~` | Bitwise NOT |
| `<<`, `>>` | Left / right shift |

### Other Operators

| Operator | Meaning |
|---|---|
| `::` | Cast (PostgreSQL) |
| `CAST(x AS type)` | Standard cast |
| `.` | Member access (schema.table.column) |
| `*` | All columns, or multiplication |

---

## 6. Expressions

An **expression** is a combination of **literals**, **identifiers**, **operators**, and **function calls** that produces a value.

### Types of Expressions

| Type | Example |
|---|---|
| **Literal** | `42`, `'Hello'` |
| **Column reference** | `salary`, `c.first_name` |
| **Arithmetic** | `salary * 1.1` |
| **String** | `first_name || ' ' || last_name` |
| **Comparison** | `salary > 50000` |
| **Logical** | `salary > 50000 AND department_id = 5` |
| **Function call** | `UPPER(first_name)`, `COUNT(*)` |
| **CASE** | `CASE WHEN ... THEN ... ELSE ... END` |
| **Subquery** | `(SELECT MAX(salary) FROM employees)` |

### Examples

```sql
-- Arithmetic
SELECT price * quantity AS total FROM order_items;

-- String concatenation
SELECT first_name || ' ' || last_name AS full_name FROM employees;

-- Boolean
SELECT * FROM products WHERE price > 100 AND in_stock = TRUE;

-- CASE expression
SELECT name,
       CASE
           WHEN salary > 70000 THEN 'High'
           WHEN salary > 50000 THEN 'Medium'
           ELSE 'Low'
       END AS salary_band
FROM employees;

-- Subquery expression
SELECT name
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

### Expression Precedence

Operators have precedence — higher precedence binds tighter.

| Precedence | Operators |
|---|---|
| 1 (highest) | `()` grouping |
| 2 | `::`, unary `-`, `+` |
| 3 | `*`, `/`, `%` |
| 4 | `+`, `-` (binary) |
| 5 | `||` (concatenation) |
| 6 | `=`, `<>`, `<`, `<=`, `>`, `>=` |
| 7 | `IS`, `LIKE`, `IN`, `BETWEEN` |
| 8 | `NOT` |
| 9 | `AND` |
| 10 (lowest) | `OR` |

**Use parentheses** to make intent explicit:

```sql
-- Ambiguous
SELECT * FROM t WHERE a = 1 OR b = 2 AND c = 3;

-- Clear
SELECT * FROM t WHERE a = 1 OR (b = 2 AND c = 3);
```

### NULL in Expressions

Any operation involving `NULL` yields `NULL` (except for a few special cases).

```sql
SELECT 5 + NULL;              -- NULL
SELECT 'abc' || NULL;         -- NULL
SELECT NULL = NULL;           -- NULL (not TRUE!)
SELECT NULL IS NULL;          -- TRUE
SELECT COALESCE(NULL, 'x');   -- 'x'
```

---

## 7. Clauses

A **clause** is a component of a statement that begins with a keyword and performs a specific function.

### Common Clauses

| Clause | Purpose | Used In |
|---|---|---|
| `SELECT` | Columns to return | `SELECT` |
| `FROM` | Source tables | `SELECT`, `DELETE` |
| `WHERE` | Filter rows | `SELECT`, `UPDATE`, `DELETE` |
| `GROUP BY` | Group rows for aggregation | `SELECT` |
| `HAVING` | Filter groups | `SELECT` |
| `ORDER BY` | Sort results | `SELECT` |
| `LIMIT` / `FETCH FIRST` | Limit rows | `SELECT` |
| `OFFSET` | Skip rows | `SELECT` |
| `JOIN ... ON` | Combine tables | `SELECT` |
| `VALUES` | Provide row values | `INSERT` |
| `SET` | Assign values | `UPDATE` |
| `RETURNING` | Return modified rows | `INSERT`, `UPDATE`, `DELETE` (PostgreSQL) |
| `WITH` | Define CTEs | `SELECT`, `INSERT`, `UPDATE` |

### Logical Processing Order

SQL clauses are **written** in one order but **executed** in another:

| Written Order | Logical Execution Order |
|---|---|
| 1. `SELECT` | 5. `SELECT` |
| 2. `FROM` | 1. `FROM` |
| 3. `WHERE` | 2. `WHERE` |
| 4. `GROUP BY` | 3. `GROUP BY` |
| 5. `HAVING` | 4. `HAVING` |
| 6. `ORDER BY` | 6. `ORDER BY` |
| 7. `LIMIT` | 7. `LIMIT` |

This explains why you can't use a `SELECT` alias in `WHERE` — the alias doesn't exist yet at that stage.

```sql
-- ✗ Fails: alias not available in WHERE
SELECT salary * 12 AS annual_salary
FROM employees
WHERE annual_salary > 600000;

-- ✓ Use the expression directly
SELECT salary * 12 AS annual_salary
FROM employees
WHERE salary * 12 > 600000;

-- ✓ Or use a subquery / CTE
WITH e AS (SELECT salary * 12 AS annual_salary FROM employees)
SELECT * FROM e WHERE annual_salary > 600000;
```

---

## 8. Statements

A **statement** is a complete, executable instruction terminated by a semicolon (or another terminator — see section 10).

### Categories of Statements

| Category | Full Name | Purpose | Examples |
|---|---|---|---|
| **DDL** | Data Definition Language | Define/modify schema | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| **DML** | Data Manipulation Language | Manipulate data | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| **DCL** | Data Control Language | Manage permissions | `GRANT`, `REVOKE` |
| **TCL** | Transaction Control Language | Manage transactions | `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` |
| **DQL** | Data Query Language | Query data | `SELECT` (sometimes split from DML) |

### Examples

**DDL:**
```sql
CREATE TABLE customers (
  id    SERIAL PRIMARY KEY,
  name  VARCHAR(100) NOT NULL
);

ALTER TABLE customers ADD COLUMN email VARCHAR(255);

DROP TABLE customers;
```

**DML:**
```sql
INSERT INTO customers (name) VALUES ('John');
UPDATE customers SET name = 'Johnny' WHERE id = 1;
DELETE FROM customers WHERE id = 1;
SELECT * FROM customers;
```

**DCL:**
```sql
GRANT SELECT ON customers TO app_readonly;
REVOKE SELECT ON customers FROM app_readonly;
```

**TCL:**
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

### Multiple Statements

Multiple statements are typically separated by **semicolons**:

```sql
CREATE TABLE t1 (id INT);
CREATE TABLE t2 (id INT);
INSERT INTO t1 VALUES (1);
```

Some clients require a special terminator (e.g., `GO` in SQL Server) to batch statements:

```sql
CREATE TABLE t1 (id INT);
GO
CREATE TABLE t2 (id INT);
GO
```

---

## 9. Comments

**Comments** are ignored by the SQL engine but help humans understand the code.

### Single-Line Comments

Begin with `--` and continue to the end of the line.

```sql
-- This is a single-line comment
SELECT * FROM customers;   -- Inline comment

SELECT
    id,     -- identifier
    name    -- customer name
FROM customers;
```

**MySQL also supports `#`** for single-line comments:

```sql
# This is also a comment in MySQL
SELECT 1;
```

### Multi-Line Comments

Enclosed in `/* ... */`, may span multiple lines.

```sql
/*
 * This query retrieves
 * all active customers
 * created in 2026.
 */
SELECT *
FROM customers
WHERE is_active = TRUE
  AND created_at >= '2026-01-01';
```

**Inline multi-line:**
```sql
SELECT id, /* name, */ email FROM customers;
```

### Nested Comments

Standard SQL does **not** support nested comments:

```sql
/* outer /* inner */ still outer? */   -- ✗ ambiguous
```

PostgreSQL supports nesting; most others do not. Avoid nested comments.

### Comment Best Practices

| Do | Don't |
|---|---|
| Explain **why**, not **what** | Restate the obvious |
| Document business rules | Comment every line |
| Note assumptions and edge cases | Leave stale comments |
| Use comments to disable code temporarily | Rely on comments instead of clear code |

**Example:**
```sql
-- ✗ Bad: restates code
SELECT id FROM customers;  -- selects id from customers

-- ✓ Good: explains why
-- Exclude test accounts created before the migration
SELECT id FROM customers WHERE id > 1000;
```

### Comments in Scripts

Tools like Flyway and Liquibase recognize special comment syntax for migrations:

```sql
-- +migrate Up
CREATE TABLE users (id SERIAL PRIMARY KEY);

-- +migrate Down
DROP TABLE users;
```

---

## 10. Statement Terminators

A **statement terminator** marks the end of a statement.

### Semicolon (`;`)

The standard terminator in SQL is the **semicolon**.

```sql
SELECT * FROM customers;
INSERT INTO customers (name) VALUES ('John');
```

**Some DBMS allow omitting it** for a single statement (e.g., Oracle SQL*Plus, SQLite), but the semicolon is standard and portable.

### Batch Terminators

Some clients use additional terminators to group statements:

| Client | Terminator | Purpose |
|---|---|---|
| **SQL Server (SSMS)** | `GO` | Send batch to server |
| **Oracle SQL*Plus** | `/` on its own line | Execute PL/SQL block |
| **MySQL client** | `;` or `\g` | Execute statement |
| **MySQL client** | `\G` | Execute with vertical output |
| **PostgreSQL (psql)** | `;` or `\g` | Execute statement |

**SQL Server example:**
```sql
CREATE TABLE t1 (id INT);
GO

CREATE TABLE t2 (id INT);
GO
```

**Oracle SQL*Plus example:**
```sql
CREATE OR REPLACE PROCEDURE hello AS
BEGIN
  DBMS_OUTPUT.PUT_LINE('Hello');
END;
/
```

The `/` on its own line executes the PL/SQL block.

### Terminator Rules

- In scripts, terminate **every statement** with `;`
- Inside stored procedures, `;` may also terminate internal statements (and require a different outer terminator, like `$$` in PostgreSQL)
- Some DBMS (Oracle) use `;` inside PL/SQL blocks and `/` outside

**PostgreSQL dollar-quoting example:**
```sql
CREATE FUNCTION add_one(x INT) RETURNS INT AS $$
BEGIN
  RETURN x + 1;   -- semicolons inside
END;
$$ LANGUAGE plpgsql;   -- semicolon terminates the whole statement
```

---

## 11. Identifier Naming Conventions

Naming conventions are not enforced by SQL but improve readability and maintainability.

### General Conventions

| Object | Convention | Example |
|---|---|---|
| **Table** | Plural, snake_case | `customers`, `order_items` |
| **Column** | Singular, snake_case | `first_name`, `created_at` |
| **Primary key** | `id` or `<table>_id` | `id`, `customer_id` |
| **Foreign key** | `<referenced_table>_id` | `customer_id`, `order_id` |
| **Index** | `idx_<table>_<columns>` | `idx_customers_email` |
| **Unique constraint** | `uq_<table>_<columns>` | `uq_customers_email` |
| **Primary key constraint** | `pk_<table>` | `pk_customers` |
| **Foreign key constraint** | `fk_<table>_<referenced>` | `fk_orders_customer` |
| **Check constraint** | `chk_<table>_<rule>` | `chk_products_price` |
| **View** | Descriptive, snake_case | `active_customers` |
| **Stored procedure** | `verb_noun` | `add_customer`, `calculate_tax` |
| **Function** | `verb_noun` | `get_order_total` |
| **Trigger** | `trg_<table>_<event>` | `trg_customers_updated_at` |

### Style Guides

| Style | Example | Common In |
|---|---|---|
| **snake_case** | `first_name` | PostgreSQL, MySQL, Ruby |
| **camelCase** | `firstName` | Java, JS ecosystems |
| **PascalCase** | `FirstName` | .NET, some SQL Server |
| **UPPER_CASE** | `FIRST_NAME` | Oracle (traditional), legacy |

**Recommendation:** For maximum portability and readability, use **snake_case** with **lowercase** letters.

### Case Sensitivity Reminder

Unquoted identifiers are typically **folded**:

| DBMS | Folding |
|---|---|
| PostgreSQL | → lowercase |
| Oracle | → UPPERCASE |
| MySQL | Depends on OS; often case-insensitive |
| SQL Server | Depends on collation |
| SQLite | Case-insensitive for ASCII |

This means `Customer` and `customer` may or may not refer to the same table depending on the DBMS.

**Rule of thumb:** Use **lowercase snake_case** consistently — it behaves the same on all systems.

### Reserved Words

Never use SQL keywords as identifiers:

```sql
-- ✗ Fails or causes confusion
CREATE TABLE user (id INT);
CREATE TABLE order (id INT);

-- ✓ Safe
CREATE TABLE users (id INT);
CREATE TABLE orders (id INT);
```

If you must, quote them — but avoid this in production.

---

## 12. Case Sensitivity Considerations

Case sensitivity in SQL is a frequent source of confusion. It depends on **what** is being compared.

### What Is Case-Sensitive?

| Element | Case-Sensitive? |
|---|---|
| **Keywords** | No |
| **Function names** | No (usually) |
| **Unquoted identifiers** | No (folded) |
| **Quoted identifiers** | Yes |
| **String literals** | Yes |
| **Data values** | Depends on collation |
| **Aliases** | Yes (if quoted) |

### Keywords and Functions

**Case-insensitive:**

```sql
SELECT COUNT(*) FROM customers;
select count(*) from customers;
Select Count(*) From Customers;
```

All equivalent. **Convention:** write keywords and built-in functions in **UPPERCASE**.

### Unquoted Identifiers

**Case-insensitive** because the DBMS folds them:

**PostgreSQL** — folds to lowercase:
```sql
CREATE TABLE Customers (id INT);
SELECT * FROM customers;   -- ✓ works (both fold to "customers")
```

**Oracle** — folds to uppercase:
```sql
CREATE TABLE Customers (id INT);
SELECT * FROM customers;   -- ✓ works (both fold to "CUSTOMERS")
```

**MySQL** — depends on OS:
- Linux: table names are case-sensitive by default
- Windows / macOS: case-insensitive

### Quoted Identifiers

**Case-sensitive** — the quote preserves the exact case:

```sql
-- PostgreSQL
CREATE TABLE "Customers" (id INT);
SELECT * FROM "Customers";   -- ✓
SELECT * FROM "customers";   -- ✗ (different identifier)
SELECT * FROM customers;     -- ✗ (folds to "customers")
```

**Rule:** If you quote, always quote, and use the exact case.

### String Literals

**Always case-sensitive:**

```sql
SELECT * FROM customers WHERE name = 'John';   -- matches 'John'
SELECT * FROM customers WHERE name = 'john';   -- matches 'john' (different)
```

To compare case-insensitively:

```sql
-- Standard
SELECT * FROM customers WHERE LOWER(name) = LOWER('John');

-- PostgreSQL
SELECT * FROM customers WHERE name ILIKE 'john';

-- MySQL
SELECT * FROM customers WHERE name COLLATE utf8mb4_general_ci = 'john';

-- SQL Server (case-insensitive collation by default)
SELECT * FROM customers WHERE name = 'john';
```

### Data Values and Collation

Whether `'John' = 'john'` is TRUE depends on **collation**:

| Collation | Behavior |
|---|---|
| `en_US.UTF-8` (PostgreSQL) | Case-sensitive |
| `utf8mb4_general_ci` (MySQL) | Case-insensitive (`ci` = case-insensitive) |
| `utf8mb4_bin` (MySQL) | Case-sensitive (`bin` = binary) |
| `SQL_Latin1_General_CP1_CI_AS` (SQL Server) | Case-insensitive (`CI`) |
| `SQL_Latin1_General_CP1_CS_AS` (SQL Server) | Case-sensitive (`CS`) |

### Practical Guidelines

| Guideline | Reason |
|---|---|
| Write **keywords in UPPERCASE** | Readability |
| Write **identifiers in lowercase snake_case** | Portability |
| **Avoid quoted identifiers** in application code | Case-sensitivity pitfalls |
| **Be explicit** about collation when case matters | Predictable behavior |
| **Test on the target DBMS** | Behavior varies |

### Cross-DBMS Portability Tips

1. **Never rely on case-insensitivity** of identifiers — use consistent casing.
2. **Avoid quoted identifiers** — they break portability.
3. **Use `LOWER()` / `UPPER()`** for case-insensitive comparisons.
4. **Document collation choices** in schema migrations.
5. **Use `ILIKE` only if you're committed to PostgreSQL** — otherwise standardize on `LOWER()`.

---

## Summary Table

| Topic | Key Points |
|---|---|
| **Statement structure** | Clause order is fixed; whitespace is flexible |
| **Keywords** | Reserved words; case-insensitive; write in UPPERCASE |
| **Identifiers** | Names for objects; rules vary; quote for special chars |
| **Literals** | Fixed values: strings, numbers, dates, booleans, NULL |
| **Operators** | Arithmetic, comparison, logical, pattern, set |
| **Expressions** | Combination of values and operators producing a value |
| **Clauses** | Statement components; logical execution order ≠ written order |
| **Statements** | Complete instructions: DDL, DML, DCL, TCL |
| **Comments** | `--` single-line, `/* */` multi-line |
| **Terminators** | `;` standard; `GO`, `/` for batches |
| **Naming** | snake_case, lowercase, no reserved words |
| **Case sensitivity** | Keywords no; quoted identifiers yes; strings yes |

---

## Key Takeaways

1. **Statement structure** follows a fixed clause order — learn the pattern for each statement type.
2. **Keywords** are reserved and case-insensitive — write them in **UPPERCASE** for readability.
3. **Identifiers** name database objects; use **lowercase snake_case** for portability.
4. **Literals** are fixed values — use **single quotes** for strings, ISO 8601 for dates.
5. **Operators** — arithmetic, comparison, logical, pattern matching — follow precedence rules; use parentheses to be explicit.
6. **Expressions** combine literals, identifiers, operators, and functions to produce values.
7. **Clauses** are statement components; **logical execution order differs** from written order (e.g., `WHERE` runs before `SELECT`).
8. **Statements** fall into DDL, DML, DCL, and TCL categories.
9. **Comments** (`--` and `/* */`) document code; explain **why**, not **what**.
10. **Terminators** — use `;` consistently; special batch terminators (`GO`, `/`) vary by client.
11. **Naming conventions** — snake_case, lowercase, descriptive, no reserved words.
12. **Case sensitivity** — keywords no, quoted identifiers yes, string literals yes, collation-dependent for data.

---

Would you like me to continue with the next topic — **SQL Data Types**, **SQL Sublanguages (DDL/DML/DCL/TCL)**, or **Writing Basic SELECT Queries**? I can format the next section in the same style.