# SQL SELECT Operations: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The SQL `SELECT` statement is the primary Data Manipulation Language (DML) command used to retrieve data from one or more tables, views, or other database objects. It forms the foundation of all data querying operations in relational database management systems (RDBMS).

**Technical Definition:** Per the ISO/IEC 9075-2 standard, a `<query specification>` consists of `SELECT [ <set quantifier> ] <select list> <table expression>`, where the `<table expression>` comprises a `<from clause>` followed by optional `<where clause>`, `<group by clause>`, `<having clause>`, and `<window clause>`. The SELECT statement returns a result set consisting of zero or more rows, each with a fixed number of columns.

**Beginner-Friendly Explanation:** Think of a `SELECT` statement as a request you make to a database: "Show me this specific data from this table, but only the parts I care about, and present it in a way I can understand." It is the way you ask questions of your data.

### Key Characteristics

- **Declarative nature:** You specify *what* data you want, not *how* to retrieve it. The database engine determines the optimal execution plan.
- **Non-destructive:** `SELECT` never modifies data in the database; it only reads and returns data.
- **Composable:** `SELECT` statements can be nested as subqueries, combined with set operators (`UNION`, `INTERSECT`, `EXCEPT`), and used within other statements.
- **Set-based:** Operations work on entire sets of rows rather than individual records, though results can be limited with clauses like `LIMIT` or `TOP`.
- **Clause-ordered:** While written in a specific textual order, the logical processing order differs (FROM → WHERE → GROUP BY → HAVING → WINDOW → SELECT → ORDER BY).

### Prerequisites

- **Database access:** A connection to an RDBMS (PostgreSQL, MySQL, SQL Server, Oracle, SQLite, etc.) with appropriate `SELECT` privileges on the target objects.
- **Basic relational concepts:** Understanding of tables, rows, columns, data types, and primary/foreign keys.
- **SQL environment:** A SQL client, command-line tool, or programming language with a database driver.
- **Sample data:** At least one table with data to query, or the ability to create one.

### Related Programming Areas

- **Data Analysis and Business Intelligence:** SELECT forms the backbone of analytical queries, reporting, and dashboard generation.
- **Application Development:** Backend services use SELECT to fetch data for APIs, user interfaces, and business logic.
- **Database Administration:** DBAs use SELECT to inspect data, verify integrity, and troubleshoot performance.
- **Data Engineering (ETL):** Extract phases of Extract-Transform-Load pipelines rely heavily on SELECT.
- **Machine Learning:** Feature extraction from databases typically begins with SELECT queries.

### Core Concepts / Features

The following core concepts are covered in this cheat sheet:

1. `SELECT` Statement (base syntax)
2. Selecting Specific Columns
3. Selecting All Columns
4. Column Aliases
5. Expressions
6. Calculated Columns
7. Literal Values
8. `DISTINCT` Results
9. Query Readability and Formatting

---

## 1. SELECT Statement

### Definitions

**Core Definition:** The `SELECT` statement is the SQL command that retrieves rows from database tables or views, returning a result set that matches the specified criteria.

**Technical Definition:** In standard SQL (ISO/IEC 9075-2), the `<query specification>` is defined as `SELECT [ <set quantifier> ] <select list> <table expression>`. The `<select list>` contains one or more `<derived column>` specifications, and the `<table expression>` must include a `<from clause>` in conformant SQL. However, many implementations (including PostgreSQL, MySQL, and SQL Server) permit `SELECT` without a `FROM` clause for evaluating expressions or testing connections.

**Beginner-Friendly Explanation:** `SELECT` is the command you type when you want to see data. You tell the database which table to look in and which columns to show, and it hands you back a table of results.

### Purposes

- To retrieve data from one or more database tables or views.
- To filter, sort, group, and transform data for analysis or presentation.
- To serve as the foundation for subqueries, views, and derived tables.
- To test database connectivity and evaluate expressions without referencing a table.

### Syntax Rules and Structure

**Complete General Syntax (PostgreSQL):**

```sql
[ WITH [ RECURSIVE ] with_query [, ...] ]
SELECT [ ALL | DISTINCT [ ON ( expression [, ...] ) ] ]
    [ { * | expression [ [ AS ] output_name ] } [, ...] ]
    [ FROM from_item [, ...] ]
    [ WHERE condition ]
    [ GROUP BY [ ALL | DISTINCT ] grouping_element [, ...] ]
    [ HAVING condition ]
    [ WINDOW window_name AS ( window_definition ) [, ...] ]
    [ { UNION | INTERSECT | EXCEPT } [ ALL | DISTINCT ] select ]
    [ ORDER BY expression [ ASC | DESC | USING operator ] [ NULLS { FIRST | LAST } ] [, ...] ]
    [ LIMIT { count | ALL } ]
    [ OFFSET start [ ROW | ROWS ] ]
    [ FETCH { FIRST | NEXT } [ count ] { ROW | ROWS } { ONLY | WITH TIES } ]
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `WITH` | Defines common table expressions (CTEs) available to the main query |
| `SELECT` | Begins the query specification; must be present |
| `ALL \| DISTINCT` | Controls whether duplicate rows are returned |
| `select_list` | Specifies the columns, expressions, or `*` to return |
| `FROM` | Identifies the table(s), view(s), or subquery(ies) to query |
| `WHERE` | Filters rows before grouping |
| `GROUP BY` | Aggregates rows into groups |
| `HAVING` | Filters groups after aggregation |
| `WINDOW` | Defines named window specifications |
| `ORDER BY` | Sorts the final result set |
| `LIMIT` / `OFFSET` / `FETCH` | Restricts the number of rows returned |

**Syntax Rules:**

- The `SELECT` keyword and the select list are mandatory in all implementations.
- In standard-conformant SQL, the `FROM` clause is mandatory; however, many RDBMSs allow `SELECT` without `FROM` for expression evaluation.
- Clause order in the text is significant: clauses must appear in the order defined by the grammar.
- The `select_list` must contain at least one expression (column reference, literal, or computed value).
- Set quantifiers (`ALL`, `DISTINCT`) apply to the entire select list, not individual columns.

**Constraints and Limitations:**

- The maximum number of expressions in a select list is implementation-dependent (e.g., 4,096 in SQL Server).
- `SELECT` without `FROM` is not permitted in strict ISO SQL but works in PostgreSQL, MySQL, SQL Server, SQLite, and Oracle.
- Column aliases cannot be referenced in the `WHERE` clause in standard SQL (though some dialects permit it).
- `DISTINCT ON` is a PostgreSQL extension and is not part of the SQL standard.

### Annotated Complete Code Examples

**Example 1: Basic SELECT with FROM (PostgreSQL)**

```sql
-- Setup: Create a sample table
CREATE TABLE employees (
    id          SERIAL PRIMARY KEY,   -- Auto-incrementing integer
    first_name  VARCHAR(50) NOT NULL,
    last_name   VARCHAR(50) NOT NULL,
    department  VARCHAR(50),
    salary      NUMERIC(10, 2)
);

-- Insert sample data
INSERT INTO employees (first_name, last_name, department, salary) VALUES
    ('Alice', 'Johnson', 'Engineering', 95000.00),
    ('Bob',   'Smith',   'Marketing',   72000.00),
    ('Carol', 'Williams','Engineering', 105000.00),
    ('David', 'Brown',   'Sales',       68000.00);

-- Query 1: Retrieve all columns from all rows
SELECT *
FROM employees;

-- Expected Output:
--  id | first_name | last_name | department  |  salary
-- ----+------------+-----------+-------------+----------
--   1 | Alice      | Johnson   | Engineering | 95000.00
--   2 | Bob        | Smith     | Marketing   | 72000.00
--   3 | Carol      | Williams  | Engineering | 105000.00
--   4 | David      | Brown     | Sales       | 68000.00
```

**Why this output occurs:** The `SELECT *` retrieves every column from every row in the `employees` table. The `FROM employees` clause tells the database to look in the `employees` table. Because there is no `WHERE` clause, no rows are filtered out. Four rows were inserted, so four rows are returned.

**Example 2: SELECT Without FROM (Expression Evaluation)**

```sql
-- PostgreSQL, MySQL, SQL Server, SQLite support this
SELECT 1 + 1 AS result;

-- Expected Output:
--  result
-- --------
--       2
```

**Why this output occurs:** The database evaluates the arithmetic expression `1 + 1` and returns the result `2` as a single row with a single column named `result`. No table is referenced, so no data is read from disk. This is useful for testing connections or evaluating functions.

### Real-World Cases

**Case 1: Generating a Daily Sales Report**

A retail company uses `SELECT` to pull daily transaction totals from a `sales` table, filtering by date and grouping by product category. The query runs nightly to populate a dashboard.

**Case 2: User Authentication Lookup**

A web application issues a `SELECT` query against a `users` table when a user logs in, retrieving the stored password hash and account status based on the submitted username. This is one of the most frequent database operations in any application.

**Case 3: Database Health Check**

A DBA runs `SELECT 1` to verify that a database connection is alive and responsive. This is a standard technique in connection pooling and monitoring systems.

### References

- PostgreSQL Documentation — SELECT - https://www.postgresql.org/docs/current/sql-select.html
- MySQL 8.4 Reference Manual — SELECT Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/select.html
- SQLite Documentation — SELECT - https://www2.sqlite.org/lang_select.html
- Microsoft SQL Server Documentation — SELECT (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-transact-sql
- Oracle Database SQL Language Reference — SELECT - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SELECT.html

---

## 2. Selecting Specific Columns

### Definitions

**Core Definition:** Selecting specific columns means explicitly naming the columns to be returned in the result set, rather than using the wildcard `*` to return all columns.

**Technical Definition:** In the `<select list>`, each item is a `<derived column>` that can be either a column reference (`<column name>`), an expression, or a literal. When column references are used, only those columns appear in the result set, in the order specified.

**Beginner-Friendly Explanation:** Instead of asking the database for "everything," you say "I only want the first name and salary columns." This makes your results smaller, faster, and easier to read.

### Purposes

- To reduce the amount of data transferred over the network.
- To improve query performance by avoiding unnecessary column reads.
- To control the column order and presence in the output.
- To avoid exposing sensitive columns (e.g., passwords, personal data).
- To make result sets more readable and focused.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT column1 [, column2, ...]
FROM table_name;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `SELECT` | Initiates the query |
| `column1, column2, ...` | Comma-separated list of column names to retrieve |
| `FROM table_name` | Specifies the source table |

**Syntax Rules:**

- Column names are separated by commas.
- The order of columns in the select list determines their order in the result set.
- Column names can be qualified with a table name or alias (e.g., `employees.first_name`).
- Identifiers containing spaces or special characters must be enclosed in double quotes (standard SQL) or backticks (MySQL) or square brackets (SQL Server).

**Constraints and Limitations:**

- Every column name in the select list must exist in one of the tables referenced in the `FROM` clause.
- If two tables in a join have columns with the same name, the column reference must be qualified to avoid ambiguity.

### Annotated Complete Code Examples

**Example 1: Selecting Two Columns**

```sql
-- Query: Retrieve only first_name and salary
SELECT first_name, salary
FROM employees;

-- Expected Output:
--  first_name |  salary
-- ------------+----------
--  Alice      | 95000.00
--  Bob        | 72000.00
--  Carol      | 105000.00
--  David      | 68000.00
```

**Why this output occurs:** The select list contains exactly two column references: `first_name` and `salary`. The database reads only these two columns from the `employees` table and returns one row per record. Columns `id`, `last_name`, and `department` are omitted because they were not requested.

**Example 2: Selecting Columns in a Different Order**

```sql
-- Query: Retrieve salary first, then last_name
SELECT salary, last_name
FROM employees;

-- Expected Output:
--   salary  | last_name
-- ----------+-----------
--  95000.00 | Johnson
--  72000.00 | Smith
-- 105000.00 | Williams
--  68000.00 | Brown
```

**Why this output occurs:** The column order in the select list determines the column order in the result set. Even though `last_name` appears before `salary` in the table definition, the result set places `salary` first because it was listed first in the query.

### Real-World Cases

**Case 1: API Response Optimization**

A REST API endpoint that returns a list of products only needs the product ID, name, and price—not the full description or image blob. Selecting specific columns reduces payload size and improves response times.

**Case 2: Compliance and Privacy**

A GDPR-compliant query for a customer support tool retrieves only `customer_id`, `first_name`, and `email`, deliberately excluding sensitive fields like `national_id` or `date_of_birth`.

### References

- PostgreSQL Documentation — SELECT List - https://www.postgresql.org/docs/current/sql-select.html
- Microsoft SQL Server — SELECT Clause - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-clause-transact-sql

---

## 3. Selecting All Columns

### Definitions

**Core Definition:** Selecting all columns uses the asterisk (`*`) wildcard in the select list to return every column from every table referenced in the `FROM` clause.

**Technical Definition:** The `*` in the `<select list>` is shorthand for all columns of all tables, views, or derived tables in the `<from clause>`. The columns are returned in the order in which they were defined in the table schema.

**Beginner-Friendly Explanation:** `SELECT *` means "give me everything." It's convenient for quick exploration but can be wasteful in production queries.

### Purposes

- To quickly explore an unfamiliar table and see all available data.
- To avoid typing out every column name during ad-hoc queries.
- To retrieve complete records when all fields are genuinely needed.
- To simplify queries in development and debugging.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT * FROM table_name;
SELECT table_name.* FROM table_name;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `*` | Wildcard returning all columns from all tables in the FROM clause |
| `table_name.*` | Wildcard returning all columns from a specific table |

**Syntax Rules:**

- `*` must be the sole item in the select list (cannot be combined with other columns).
- `table_name.*` can be combined with other columns or expressions.
- The order of columns in the result set follows the table definition order.

**Constraints and Limitations:**

- Adding a new column to the table changes the result set of `SELECT *` queries, potentially breaking application code that relies on a fixed column count or order.
- `SELECT *` retrieves unnecessary data, increasing I/O and network traffic.
- In joins, `SELECT *` returns duplicate column names if both tables share column names.

### Annotated Complete Code Examples

**Example 1: SELECT * from a Single Table**

```sql
SELECT * FROM employees;
```

**Expected Output:**

```
 id | first_name | last_name | department  |  salary
----+------------+-----------+-------------+----------
  1 | Alice      | Johnson   | Engineering | 95000.00
  2 | Bob        | Smith     | Marketing   | 72000.00
  3 | Carol      | Williams  | Engineering | 105000.00
  4 | David      | Brown     | Sales       | 68000.00
```

**Why this output occurs:** The `*` expands to all five columns (`id`, `first_name`, `last_name`, `department`, `salary`) in the order they appear in the `employees` table definition. All four rows are returned because there is no `WHERE` clause.

**Example 2: SELECT * in a Join (Illustrating the Pitfall)**

```sql
-- Setup: Add a departments table
CREATE TABLE departments (
    dept_name   VARCHAR(50),
    location    VARCHAR(50)
);

INSERT INTO departments VALUES
    ('Engineering', 'Building A'),
    ('Marketing',   'Building B'),
    ('Sales',       'Building C');

-- Query: Join and select all
SELECT *
FROM employees e
JOIN departments d ON e.department = d.dept_name;

-- Expected Output (showing duplicate/ambiguous columns):
--  id | first_name | last_name | department  |  salary  | dept_name   | location
-- ----+------------+-----------+-------------+----------+-------------+-----------
--   1 | Alice      | Johnson   | Engineering | 95000.00 | Engineering | Building A
--   2 | Bob        | Smith     | Marketing   | 72000.00 | Marketing   | Building B
--   3 | Carol      | Williams  | Engineering |105000.00 | Engineering | Building A
--   4 | David      | Brown     | Sales       | 68000.00 | Sales       | Building C
```

**Why this output occurs:** `SELECT *` returns all columns from both tables. The `department` column from `employees` and the `dept_name` column from `departments` are semantically identical but appear as separate columns. This is why `SELECT *` in joins is generally discouraged in production code.

### Real-World Cases

**Case 1: Exploratory Data Analysis**

A data analyst connecting to a new database runs `SELECT * FROM customers LIMIT 10;` to understand the schema and data before writing targeted queries.

**Case 2: Row Duplication in ETL**

An ETL developer copies rows between staging and production tables using `INSERT INTO target SELECT * FROM source`. This is acceptable when schemas are identical and controlled.

### References

- Microsoft SQL Server — SELECT Clause (Arguments) - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-clause-transact-sql
- MySQL 8.4 Reference Manual — SELECT Statement - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/select.html

---

## 4. Column Aliases

### Definitions

**Core Definition:** A column alias is a temporary name assigned to a column or expression in the result set, used to improve readability or provide a meaningful heading.

**Technical Definition:** In the `<select list>`, a `<derived column>` may be followed by `[ [ AS ] column_alias ]`, where `column_alias` is an identifier or quoted identifier that renames the column for the duration of the query. The alias does not change the column name in the database schema.

**Beginner-Friendly Explanation:** An alias is a nickname for a column in your results. Instead of showing `SUM(salary)` as the column header, you can rename it to "Total Payroll."

### Purposes

- To give meaningful names to computed columns and expressions.
- To make result sets more readable for end users.
- To rename columns that have cryptic or technical names.
- To resolve naming conflicts when joining tables with duplicate column names.
- To allow reference to columns in `ORDER BY` and (in some dialects) `GROUP BY`.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT expression [ AS ] column_alias
FROM table_name;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `expression` | A column name, calculation, function call, or literal |
| `AS` | Optional keyword introducing the alias |
| `column_alias` | The temporary name for the column |

**Syntax Rules:**

- `AS` is optional but recommended for clarity.
- The alias must follow all standard identifier naming rules (letters, digits, underscores; not starting with a digit).
- Aliases containing spaces or special characters must be enclosed in double quotes (standard SQL), backticks (MySQL), or square brackets (SQL Server).
- The alias can be used in `ORDER BY` in all major RDBMSs, but its use in `WHERE` is restricted in standard SQL.

**Constraints and Limitations:**

- Column aliases cannot be referenced in the `WHERE` clause in standard SQL (PostgreSQL, for example, disallows this).
- Aliases do not persist beyond the query; they are not stored in the database schema.
- Oracle limits alias length to 30 characters (older versions) or 128 characters (12.2+).

### Annotated Complete Code Examples

**Example 1: Aliasing a Column**

```sql
SELECT first_name AS "First Name", salary AS "Annual Salary"
FROM employees;

-- Expected Output:
--  First Name | Annual Salary
-- ------------+--------------
--  Alice      | 95000.00
--  Bob        | 72000.00
--  Carol      | 105000.00
--  David      | 68000.00
```

**Why this output occurs:** The `AS "First Name"` clause renames the `first_name` column to `First Name` in the result set. Double quotes are required because the alias contains a space. The underlying column name in the database remains `first_name`.

**Example 2: Aliasing an Expression**

```sql
SELECT first_name || ' ' || last_name AS full_name,
       salary * 12 AS annual_salary
FROM employees;

-- Expected Output:
--  full_name     | annual_salary
-- ---------------+---------------
--  Alice Johnson |    1140000.00
--  Bob Smith     |     864000.00
--  Carol Williams|    1260000.00
--  David Brown   |     816000.00
```

**Why this output occurs:** The `||` operator concatenates strings in PostgreSQL. `first_name || ' ' || last_name` produces "Alice Johnson", aliased as `full_name`. The expression `salary * 12` computes the annual salary and is aliased as `annual_salary`. Without aliases, the column headers would be `?column?` and `?column?`.

### Real-World Cases

**Case 1: Business Reports**

A financial report query aliases `SUM(revenue) - SUM(cost)` as `"Net Profit"` so the spreadsheet or dashboard displays a meaningful header instead of `?column?`.

**Case 2: API Response Fields**

A backend query aliases `usr_nm` as `username` and `acct_bal` as `balance` to present a clean JSON response to frontend consumers.

### References

- MySQL Reference Manual — Problems with Column Aliases - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/select.html
- Oracle Database — Column Aliases - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SELECT.html

---

## 5. Expressions

### Definitions

**Core Definition:** An SQL expression is a combination of symbols, values, operators, and functions that the database evaluates to produce a single value.

**Technical Definition:** An expression in SQL is formed from `<simple expression>`, `<set function specification>`, `<scalar subquery>`, or combinations thereof. Simple expressions include column references, literals, variables, and scalar function calls, combined with unary or binary operators.

**Beginner-Friendly Explanation:** An expression is a calculation or operation you write in a SQL query. For example, `salary * 1.1` is an expression that increases salary by 10%. `UPPER(first_name)` is an expression that converts a name to uppercase.

### Purposes

- To perform arithmetic calculations (addition, subtraction, multiplication, division, modulus).
- To manipulate string values (concatenation, substring, case conversion).
- To compare values (equality, inequality, greater/less than).
- To evaluate logical conditions (AND, OR, NOT).
- To call built-in or user-defined functions.
- To transform data without modifying the underlying table.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT expression [ AS alias ]
FROM table_name;
```

Where `expression` can be:

```
column_name
| literal
| expression operator expression
| function_name(arguments)
| ( expression )
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `column_name` | References a column value |
| `literal` | A fixed value (number, string, date, NULL) |
| `operator` | Arithmetic (`+`, `-`, `*`, `/`), string (`||`), comparison (`=`, `<`, `>`) |
| `function_name` | A built-in or user-defined function |
| `( expression )` | Parentheses to control evaluation order |

**Syntax Rules:**

- Operator precedence follows standard mathematical rules: parentheses first, then exponentiation, multiplication/division, addition/subtraction, comparison, logical.
- String concatenation operators vary by dialect: `||` (standard SQL, PostgreSQL, Oracle, SQLite), `+` (SQL Server, with `SET CONCAT_NULL_YIELDS_NULL OFF`), `CONCAT()` function (MySQL, SQL Server 2012+).
- NULL propagation: most operators return NULL if any operand is NULL.

**Constraints and Limitations:**

- Division by zero behavior varies by dialect (error in PostgreSQL, NULL in MySQL, warning in SQL Server).
- Operator precedence can be overridden with parentheses; relying on default precedence can lead to bugs.
- Implicit type conversion rules are implementation-dependent and can cause unexpected results or performance issues.

### Annotated Complete Code Examples

**Example 1: Arithmetic Expression**

```sql
SELECT first_name,
       salary,
       salary * 1.10 AS salary_after_raise,
       salary / 12 AS monthly_salary
FROM employees
WHERE department = 'Engineering';

-- Expected Output:
--  first_name |  salary  | salary_after_raise | monthly_salary
-- ------------+----------+--------------------+----------------
--  Alice      | 95000.00 |          104500.00 |        7916.67
--  Carol      |105000.00 |          115500.00 |        8750.00
```

**Why this output occurs:** The expression `salary * 1.10` computes a 10% raise. `salary / 12` computes the monthly equivalent. The `WHERE` clause limits results to Engineering employees. Arithmetic operations are performed per-row.

**Example 2: String Expression**

```sql
SELECT UPPER(first_name) || ' ' || UPPER(last_name) AS formal_name,
       LENGTH(first_name) AS name_length
FROM employees
WHERE LENGTH(first_name) > 4;

-- Expected Output:
--  formal_name    | name_length
-- ----------------+-------------
--  ALICE JOHNSON  |           5
--  CAROL WILLIAMS |           5
--  DAVID BROWN    |           5
```

**Why this output occurs:** `UPPER()` converts text to uppercase. The `||` operator concatenates strings. `LENGTH()` returns the number of characters. The `WHERE` clause filters rows where the first name is longer than 4 characters. Bob (3 characters) is excluded.

### Real-World Cases

**Case 1: Pricing Calculations**

An e-commerce query computes `price * quantity * (1 - discount)` to calculate the final line-item total for an order, applying discounts and quantity multipliers in a single expression.

**Case 2: Data Cleansing**

A query uses `TRIM(UPPER(email))` to normalise email addresses stored with inconsistent casing and whitespace before loading them into a data warehouse.

### References

- Microsoft SQL Server — Expressions (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/expressions-transact-sql
- Oracle Database SQL Language Reference — Expressions - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Expressions.html

---

## 6. Calculated Columns

### Definitions

**Core Definition:** A calculated column is a column in a query result whose value is derived from an expression involving other columns, literals, or functions, rather than being stored directly in a table.

**Technical Definition:** In SQL Server, a computed column is a virtual column that is not physically stored in the table unless marked `PERSISTED`. In the context of `SELECT` queries, a calculated column refers to a derived column in the select list produced by an expression.

**Beginner-Friendly Explanation:** A calculated column is like a spreadsheet formula: you create a new column in your results that calculates something based on existing data—for example, `salary * 12` to show annual pay.

### Purposes

- To derive new information from existing data without altering the table.
- To perform on-the-fly calculations for reports and analytics.
- To normalise data presentation (e.g., concatenating first and last names).
- To avoid storing redundant computed values when storage is a concern.
- To support dynamic filtering and sorting based on computed values.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT column1,
       expression AS calculated_column_name
FROM table_name;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `column1` | Regular column reference |
| `expression` | Calculation or function producing the derived value |
| `AS calculated_column_name` | Alias for the derived column |

**Syntax Rules:**

- Calculated columns must have an alias if they are to be referenced by name in `ORDER BY` or by application code.
- The expression can reference any column in the `FROM` clause.
- Aggregate functions (e.g., `SUM`, `AVG`) cannot be mixed with non-aggregate columns unless a `GROUP BY` clause is present.

**Constraints and Limitations:**

- Calculated columns are not stored in the database (unless defined as persisted computed columns at the table level).
- Performance may suffer if the expression is complex and applied to large result sets.
- Some RDBMSs restrict the use of calculated columns in `WHERE` and `GROUP BY` clauses.

### Annotated Complete Code Examples

**Example 1: Derived Column in SELECT**

```sql
SELECT first_name,
       last_name,
       salary,
       salary * 12 AS annual_salary
FROM employees;

-- Expected Output:
--  first_name | last_name |  salary  | annual_salary
-- ------------+-----------+----------+---------------
--  Alice      | Johnson   | 95000.00 |    1140000.00
--  Bob        | Smith     | 72000.00 |     864000.00
--  Carol      | Williams  |105000.00 |    1260000.00
--  David      | Brown     | 68000.00 |     816000.00
```

**Why this output occurs:** The expression `salary * 12` is evaluated for each row, producing the annual salary. The alias `annual_salary` provides a column header. The value is not stored in the table; it exists only in the query result.

**Example 2: Calculated Column with Conditional Logic**

```sql
SELECT first_name,
       salary,
       CASE
           WHEN salary >= 100000 THEN 'High'
           WHEN salary >= 80000  THEN 'Medium'
           ELSE 'Standard'
       END AS salary_band
FROM employees;

-- Expected Output:
--  first_name |  salary  | salary_band
-- ------------+----------+-------------
--  Alice      | 95000.00 | Medium
--  Bob        | 72000.00 | Standard
--  Carol      |105000.00 | High
--  David      | 68000.00 | Standard
```

**Why this output occurs:** The `CASE` expression evaluates each row's salary against the specified thresholds and returns the matching text label. Alice's salary (95,000) is ≥ 80,000 but < 100,000, so she is "Medium." Carol's salary (105,000) is ≥ 100,000, so she is "High."

### Real-World Cases

**Case 1: Financial Reporting**

A quarterly report query calculates `revenue - expenses AS profit` and `(revenue - expenses) / revenue * 100 AS profit_margin` for each business unit, providing derived metrics without storing them.

**Case 2: Inventory Management**

A warehouse query computes `quantity_on_hand * unit_cost AS total_value` to determine the total value of each product in stock.

### References

- Microsoft SQL Server — Specify Computed Columns in a Table - https://learn.microsoft.com/en-us/sql/relational-databases/tables/specify-computed-columns-in-a-table
- PostgreSQL Documentation — Value Expressions - https://www.postgresql.org/docs/current/sql-expressions.html

---

## 7. Literal Values

### Definitions

**Core Definition:** A literal is a fixed, constant value written directly into an SQL statement, as opposed to a value stored in a column or computed by an expression.

**Technical Definition:** Literals (also called constants) are fixed data values that appear in SQL text. They can be character strings (`'text'`), numeric values (`42`, `3.14`), hexadecimal values (`0xFF`), bit strings (`B'101'`), boolean values (`TRUE`, `FALSE`), or `NULL`. They are self-defining: the database knows their type from their format.

**Beginner-Friendly Explanation:** A literal is a value you type directly into your query. For example, `'Active'`, `100`, or `'2024-01-01'` are literals. They don't come from a table; they are constants you write yourself.

### Purposes

- To filter rows by a specific value in `WHERE` clauses.
- To provide default or fallback values in expressions.
- To test queries with known values during development.
- To insert constant values alongside column data in query results.
- To evaluate expressions involving constants.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT literal_value [ AS alias ]
FROM table_name;
```

**Component Breakdown:**

| Literal Type | Syntax | Example |
|-------------|--------|---------|
| String | `'text'` | `'Hello'` |
| National string | `N'text'` | `N'Hello'` |
| Numeric | `123`, `3.14` | `42`, `99.99` |
| Hexadecimal | `0xFF` or `X'FF'` | `0x1A` |
| Boolean | `TRUE`, `FALSE` | `TRUE` |
| NULL | `NULL` | `NULL` |
| Date/Time | `'2024-01-01'` (dialect-dependent) | `'2024-01-01'` |

**Syntax Rules:**

- String literals are enclosed in single quotes (`'`). Double quotes are typically used for identifiers, not string literals.
- Single quotes inside a string literal are escaped by doubling them: `'It''s'`.
- Numeric literals are written without quotes.
- Date and time literals are written as strings but interpreted according to the database's date format settings.
- `NULL` is a keyword, not a string; `'NULL'` is the four-character string.

**Constraints and Limitations:**

- Literal syntax for dates and times varies significantly across dialects.
- Some dialects require explicit casts for certain literal types (e.g., `DATE '2024-01-01'` in standard SQL).
- Large literals in queries can impact parsing performance and make queries harder to read.

### Annotated Complete Code Examples

**Example 1: Using Literals in SELECT**

```sql
SELECT 'Employee' AS record_type,
       first_name,
       salary,
       1000 AS bonus_flat
FROM employees;

-- Expected Output:
--  record_type | first_name |  salary  | bonus_flat
-- -------------+------------+----------+------------
--  Employee    | Alice      | 95000.00 |       1000
--  Employee    | Bob        | 72000.00 |       1000
--  Employee    | Carol      |105000.00 |       1000
--  Employee    | David      | 68000.00 |       1000
```

**Why this output occurs:** `'Employee'` is a string literal returned for every row. `1000` is a numeric literal returned as a constant column. Neither value comes from the `employees` table; they are constants embedded in the query. The aliases provide column headers.

**Example 2: Using Literals in WHERE**

```sql
SELECT first_name, department
FROM employees
WHERE department = 'Engineering'
  AND salary > 90000;

-- Expected Output:
--  first_name | department
-- ------------+-------------
--  Alice      | Engineering
--  Carol      | Engineering
```

**Why this output occurs:** The literal `'Engineering'` filters rows where the department matches that exact string. The literal `90000` filters rows where salary exceeds that amount. Bob and David are excluded because their departments or salaries do not match the literal conditions.

### Real-World Cases

**Case 1: Status Filtering**

An application query uses `WHERE status = 'active'` to retrieve only active records. The literal `'active'` defines the filter criterion.

**Case 2: Multi-Tenant Data Isolation**

A SaaS platform appends `WHERE tenant_id = 'tenant_abc123'` to every query, using a literal to ensure data isolation between customers.

### References

- MySQL Reference Manual — Literal Values - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/literals.html
- Oracle Database — Literal Values - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Literals.html

---

## 8. DISTINCT Results

### Definitions

**Core Definition:** The `DISTINCT` keyword eliminates duplicate rows from a query's result set, returning only unique combinations of values in the select list.

**Technical Definition:** `SELECT DISTINCT` removes duplicate rows from the result set, keeping one row from each group of duplicates. In standard SQL, `DISTINCT` applies to the entire row (all columns in the select list). PostgreSQL extends this with `DISTINCT ON (expression)`, which keeps the first row of each set where the given expressions evaluate to equal.

**Beginner-Friendly Explanation:** `DISTINCT` tells the database: "If the same result appears more than once, show it only once." It's like removing duplicate entries from a list.

### Purposes

- To eliminate duplicate rows from query results.
- To find unique values in a column or combination of columns.
- To prepare data for aggregation where duplicates would skew results.
- To generate lists of distinct categories, statuses, or types.
- To reduce result set size when duplicates are not meaningful.

### Syntax Rules and Structure

**Complete General Syntax:**

```sql
SELECT DISTINCT column1 [, column2, ...]
FROM table_name;
```

**PostgreSQL Extension:**

```sql
SELECT DISTINCT ON (expression1 [, expression2, ...])
       column1 [, column2, ...]
FROM table_name
[ORDER BY expression1 [ASC | DESC], ...];
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `DISTINCT` | Removes duplicate rows from the result |
| `DISTINCT ON (expression)` | (PostgreSQL) Keeps first row per distinct expression value |
| `column1, column2` | Defines what constitutes a "row" for deduplication purposes |

**Syntax Rules:**

- `DISTINCT` must immediately follow `SELECT` and precede the select list.
- `DISTINCT` applies to the entire select list, not individual columns.
- `DISTINCT ON` expressions must match the leftmost `ORDER BY` expressions in PostgreSQL.
- `NULL` values are considered duplicates of each other for `DISTINCT` purposes; only one `NULL` is returned.

**Constraints and Limitations:**

- `DISTINCT` requires a sort or hash operation, which can be expensive on large result sets.
- `DISTINCT ON` is a PostgreSQL extension and is not portable to other RDBMSs.
- `DISTINCT` cannot be used with certain aggregate functions in the same select list without `GROUP BY`.
- Some RDBMSs restrict `DISTINCT` in subqueries with `ORDER BY`.

### Annotated Complete Code Examples

**Example 1: Basic DISTINCT**

```sql
-- Setup: Add duplicate departments to make DISTINCT meaningful
-- (In the original table, departments already repeat: Engineering appears twice)

SELECT DISTINCT department
FROM employees;

-- Expected Output:
--  department
-- -------------
--  Engineering
--  Marketing
--  Sales
```

**Why this output occurs:** Without `DISTINCT`, the query would return four rows: Engineering, Marketing, Engineering, Sales. `DISTINCT` collapses the two Engineering rows into one, returning only three unique department names.

**Example 2: DISTINCT on Multiple Columns**

```sql
SELECT DISTINCT department, salary_band
FROM (
    SELECT department,
           CASE
               WHEN salary >= 100000 THEN 'High'
               ELSE 'Standard'
           END AS salary_band
    FROM employees
) sub;

-- Expected Output:
--  department  | salary_band
-- -------------+-------------
--  Engineering | High
--  Engineering | Standard
--  Marketing   | Standard
--  Sales       | Standard
```

**Why this output occurs:** The inner query computes a salary band. The outer `DISTINCT` then removes duplicate combinations of `department` and `salary_band`. Engineering appears twice (once with "High" for Carol and once with "Standard" for Alice), but the combination `(Engineering, Standard)` is unique, so both rows are returned.

### Real-World Cases

**Case 1: Customer Segmentation**

A marketing query uses `SELECT DISTINCT country FROM customers` to build a list of countries for a dropdown filter.

**Case 2: Data Quality Auditing**

A data engineer runs `SELECT DISTINCT status FROM orders` to discover all status values present in the database, including unexpected or deprecated ones.

### References

- PostgreSQL Documentation — SELECT (DISTINCT Clause) - https://www.postgresql.org/docs/current/sql-select.html
- Microsoft SQL Server — SELECT Clause (DISTINCT) - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-clause-transact-sql

---

## 9. Query Readability and Formatting

### Definitions

**Core Definition:** Query readability refers to the practice of writing SQL code in a consistent, well-structured manner that is easy for humans to read, understand, debug, and maintain.

**Technical Definition:** SQL formatting involves the consistent use of indentation, line breaks, keyword casing, aliasing, and commenting to visually organise query logic. While SQL engines ignore whitespace and case (for keywords), human readers do not.

**Beginner-Friendly Explanation:** Formatting your SQL is like organising your desk: it doesn't change what you're working on, but it makes it much easier to find things and get work done. A well-formatted query is a pleasure to read; a poorly formatted one is a puzzle.

### Purposes

- To make complex queries understandable at a glance.
- To reduce the time required to debug and modify queries.
- To facilitate code reviews and team collaboration.
- To establish consistent coding standards across a team or organisation.
- To make it easier to spot syntax errors and logical mistakes.

### Syntax Rules and Structure

**Recommended Formatting Conventions:**

1. **Capitalise SQL keywords** (`SELECT`, `FROM`, `WHERE`, `JOIN`, `GROUP BY`, etc.) to distinguish them from identifiers.
2. **Place each major clause on its own line.**
3. **Indent subqueries and nested logic** to show hierarchy.
4. **Use meaningful aliases** for tables and columns.
5. **Align related items** in the select list and conditions.
6. **Use comments** (`--` for single-line, `/* */` for multi-line) to explain non-obvious logic.
7. **Break long conditions** across multiple lines, with each condition on its own line.

**Poorly Formatted Example:**

```sql
select e.first_name,e.last_name,d.dept_name from employees e join departments d on e.department=d.dept_name where e.salary>80000 order by e.last_name;
```

**Well-Formatted Example:**

```sql
SELECT
    e.first_name,
    e.last_name,
    d.dept_name
FROM employees AS e
JOIN departments AS d
    ON e.department = d.dept_name
WHERE e.salary > 80000
ORDER BY e.last_name;
```

**Syntax Rules:**

- SQL keywords are case-insensitive; `SELECT`, `select`, and `SeLeCt` are equivalent to the engine.
- Whitespace (spaces, tabs, newlines) is generally ignored by the engine but is critical for human readability.
- Comments are ignored by the engine and are for documentation.

**Constraints and Limitations:**

- Excessive formatting can make simple queries unnecessarily verbose.
- Some tools and ORMs generate poorly formatted SQL automatically; manual formatting may be required for review.
- There is no single universally enforced SQL style guide; conventions vary by team, project, and RDBMS.

### Annotated Complete Code Examples

**Example 1: Formatting a Simple Query**

**Before formatting:**

```sql
select first_name,salary from employees where department='Engineering' order by salary desc;
```

**After formatting:**

```sql
SELECT
    first_name,
    salary
FROM employees
WHERE department = 'Engineering'
ORDER BY salary DESC;
```

**Expected Output:**

```
 first_name |  salary
------------+----------
 Carol      | 105000.00
 Alice      |  95000.00
```

**Why this output occurs:** The query is logically identical to the unformatted version. The formatting makes it immediately clear which columns are selected, which table is queried, what filter is applied, and how results are sorted. In a production codebase, this clarity prevents mistakes and speeds up maintenance.

**Example 2: Formatting a Complex Query with Subquery**

```sql
-- Calculate average salary per department, then find departments
-- whose average exceeds the company-wide average.

SELECT
    department,
    ROUND(AVG(salary), 2) AS avg_dept_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > (
    SELECT AVG(salary)
    FROM employees
)
ORDER BY avg_dept_salary DESC;
```

**Expected Output:**

```
 department  | avg_dept_salary
-------------+-----------------
 Engineering |       100000.00
```

**Why this output occurs:** The inner subquery computes the company-wide average salary (85,000). The outer query groups by department and keeps only those departments whose average exceeds 85,000. Engineering's average of 100,000 passes the filter; Marketing and Sales do not. The `ROUND` function limits the output to two decimal places. The formatting makes the nested query structure immediately visible.

### Real-World Cases

**Case 1: Code Review Efficiency**

A development team adopts a shared SQL style guide (keywords uppercase, one clause per line, aliases required). Code reviews become faster because reviewers can focus on logic rather than deciphering formatting.

**Case 2: Legacy Query Maintenance**

An engineer inherits a 500-line stored procedure with no formatting or comments. By reformatting the query and adding comments, the engineer reduces the time required to understand and modify it from hours to minutes.

### References

- Microsoft SQL Server — Formatting SQL in VS Code - https://learn.microsoft.com/en-us/answers/questions/5552640/formatting-sql-in-vs-code
- Databricks SQL — Custom Format SQL Statements - https://learn.microsoft.com/en-us/azure/databricks/sql/user/queries/formatting
- Devart — Best Practices for SQL Formatting - https://www.devart.com/dbforge/sql/studio/best-practices-sql-formatting.html

---

## Summary Table of Key Features

| Feature | Purpose | Key Limitation |
|---------|---------|----------------|
| `SELECT` | Retrieve data from tables | FROM clause required in standard SQL |
| Specific Columns | Reduce data transfer and improve performance | Must exist in referenced tables |
| `SELECT *` | Quick exploration and full row retrieval | Breaks when schema changes |
| Column Aliases | Improve readability and resolve conflicts | Not usable in `WHERE` (standard SQL) |
| Expressions | Perform calculations and transformations | NULL propagation, dialect-specific operators |
| Calculated Columns | Derive new values on the fly | Not stored; performance overhead for complex expressions |
| Literal Values | Provide constant values in queries | Dialect-specific date/time syntax |
| `DISTINCT` | Remove duplicate rows | Expensive sort/hash operation |
| `DISTINCT ON` | Keep first row per group (PostgreSQL) | Non-standard, not portable |
| Formatting | Improve readability and maintainability | No enforcement; subjective conventions |