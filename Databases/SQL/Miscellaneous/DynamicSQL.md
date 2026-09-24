# SQL Dynamic SQL & Metadata-Driven Development: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Dynamic SQL is the practice of constructing SQL statements as strings at runtime and executing them programmatically, rather than writing fixed SQL text. Metadata-driven development uses system catalog queries to generate SQL code dynamically based on the database schema.

**Technical Definition:** Dynamic SQL involves building a query string using concatenation, variables, and conditional logic, then passing that string to an execution engine (`EXEC`, `sp_executesql`, `PREPARE`/`EXECUTE`, or PL/pgSQL `EXECUTE`). Metadata-driven development queries system catalogs (`INFORMATION_SCHEMA`, `sys.objects`, `pg_catalog`) to discover tables, columns, constraints, and other schema objects, then uses that metadata to generate DDL or DML statements programmatically .

**Beginner-Friendly Explanation:** Normally, SQL statements are fixed—you write exactly what you want to run. Dynamic SQL lets you build the statement "on the fly" as a string, then run it. This is useful when you don't know the table name, column name, or filter values until the program is running. Metadata-driven development takes this further: you ask the database "what tables do you have?" and then write code to generate queries based on the answer.

### Key Characteristics

- **Runtime construction:** SQL text is assembled as a string during program execution.
- **Two execution paths in SQL Server:** `EXEC` (simple but injection-prone) and `sp_executesql` (parameterized, plan-cache-friendly) .
- **Plan cache implications:** `EXEC` creates a new plan for each distinct string; `sp_executesql` with parameters reuses plans .
- **Injection risk:** String concatenation of user input is the primary SQL injection vector .
- **Parameterization is the defense:** Separating SQL text from data values prevents injection structurally .
- **Metadata introspection:** System catalogs expose schema information programmatically .
- **Dialect differences:** SQL Server, MySQL, PostgreSQL, and Oracle each have distinct dynamic SQL mechanisms .

### Prerequisites

- **SQL proficiency:** `SELECT`, `INSERT`, `UPDATE`, `DELETE`, and stored procedures.
- **String manipulation:** Concatenation, variables, and conditional string building.
- **Security awareness:** Understanding of SQL injection and parameterization.
- **System catalog knowledge:** Familiarity with querying metadata views.

### Related Programming Areas

- **Database Administration:** Generating maintenance scripts dynamically.
- **Data Engineering:** ETL pipelines that adapt to varying schemas.
- **Application Development:** Flexible search and reporting features.
- **Security Engineering:** Preventing SQL injection in dynamic query paths.

### Core Concepts / Features

1. Dynamic Statement Construction
2. Execution Contexts: EXECUTE() vs. sp_executesql
3. Parameterized Dynamic SQL
4. Security & Vulnerabilities
5. Metadata-Driven Queries

---

## 1. Dynamic Statement Construction

### Definitions

**Core Definition:** Dynamic statement construction is the process of building a SQL statement string at runtime using concatenation, variables, and conditional logic, then executing it.

**Technical Definition:** A dynamic SQL statement is assembled by combining literal SQL fragments with variable values (or other string expressions) into a single string variable. The string is then passed to an execution mechanism. Construction typically uses `CONCAT()`, `+`, `||`, or `SET @sql = @sql + fragment`. The resulting string is not parsed or validated until execution time .

**Beginner-Friendly Explanation:** You build the SQL statement like you build a sentence with word tiles: start with a base, add pieces based on conditions, and then "say" the whole sentence to the database.

### Purposes

- To build queries where table names, column names, or filter values are not known until runtime.
- To implement generic procedures that work with any table or column.
- To generate DDL statements programmatically (e.g., creating backup tables with dynamic names).
- To support flexible search interfaces with optional filter conditions.
- To automate repetitive database tasks.

### Syntax Rules and Structure

**Complete General Syntax (SQL Server):**

```sql
DECLARE @sql NVARCHAR(MAX);
SET @sql = N'SELECT * FROM ' + QUOTENAME(@table_name) + N' WHERE 1=1';

IF @filter IS NOT NULL
    SET @sql = @sql + N' AND column = @filter';

EXEC sp_executesql @sql, N'@filter INT', @filter = @filter;
```

**Complete General Syntax (MySQL):**

```sql
SET @sql = CONCAT('SELECT * FROM ', @table_name, ' WHERE id = ?');
PREPARE stmt FROM @sql;
EXECUTE stmt USING @id;
DEALLOCATE PREPARE stmt;
```

**Complete General Syntax (PostgreSQL PL/pgSQL):**

```sql
EXECUTE 'SELECT * FROM ' || quote_ident(table_name) || ' WHERE id = $1'
    USING id_value;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `DECLARE @sql` | Variable to hold the SQL string |
| `SET @sql = ...` | Assignment of the base SQL text |
| `IF ... SET @sql = @sql + ...` | Conditional appending of clauses |
| `QUOTENAME()` | Safely quoting identifiers |
| `sp_executesql` / `EXECUTE` | Execution mechanism |

**Syntax Rules:**

- SQL Server: Use `NVARCHAR(MAX)` for the SQL string; `VARCHAR` may truncate .
- MySQL: Use `PREPARE stmt FROM @sql` to prepare, `EXECUTE stmt` to run, `DEALLOCATE PREPARE stmt` to clean up .
- PostgreSQL: Use `EXECUTE` in PL/pgSQL; use `quote_ident()` for identifiers and `USING` for values .
- Identifiers (table/column names) must be quoted appropriately (`QUOTENAME()` in SQL Server, `quote_ident()` in PostgreSQL).
- Values must be passed as parameters, never concatenated .

**Constraints and Limitations:**

- SQL Server: `EXEC` cannot parameterize; only `sp_executesql` can .
- MySQL: Prepared statements cannot be used in stored functions or triggers; only in procedures .
- MySQL: Local variables cannot be referenced in prepared statements (scope limitation) .
- PostgreSQL: PL/pgSQL `EXECUTE` is not the same as the SQL-level `EXECUTE` statement .

### Annotated Complete Code Examples

**Example 1: Dynamic Table Query (SQL Server)**

```sql
CREATE PROCEDURE dbo.GetTableData
    @TableName NVARCHAR(128),
    @FilterValue INT = NULL
AS
BEGIN
    DECLARE @sql NVARCHAR(MAX);
    
    -- Build base query with safely quoted table name
    SET @sql = N'SELECT * FROM ' + QUOTENAME(@TableName) + N' WHERE 1=1';
    
    -- Conditionally add filter clause
    IF @FilterValue IS NOT NULL
        SET @sql = @sql + N' AND id = @filter';
    
    -- Execute with parameterization
    EXEC sp_executesql @sql, N'@filter INT', @filter = @FilterValue;
END;
```

**Why this works:** The table name is quoted with `QUOTENAME()` to prevent injection through the identifier. The filter value is passed as a parameter to `sp_executesql`, keeping it separate from the SQL text .

**Example 2: Dynamic Backup Table (MySQL)**

```sql
DELIMITER //
CREATE PROCEDURE create_backup_table(IN original_table VARCHAR(100))
BEGIN
    DECLARE current_date VARCHAR(8);
    SET current_date = DATE_FORMAT(NOW(), '%Y%m%d');
    
    SET @backup_table = CONCAT(original_table, '_backup_', current_date);
    SET @sql = CONCAT('CREATE TABLE ', @backup_table, 
                      ' AS SELECT * FROM ', original_table);
    
    PREPARE stmt FROM @sql;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
    
    SELECT CONCAT('Backup created: ', @backup_table) AS message;
END //
DELIMITER ;

CALL create_backup_table('employees');
```

**Why this works:** The procedure constructs a dynamic `CREATE TABLE ... AS SELECT` statement with a date-stamped backup table name. `PREPARE` and `EXECUTE` run the dynamically built DDL .

### Real-World Cases

**Case 1: Flexible Search Interface**

An application builds a `WHERE` clause dynamically based on user-selected filters. Only the selected filters are appended to the base query, and all values are parameterized.

**Case 2: Schema Maintenance Automation**

A DBA script generates `ALTER TABLE` statements by querying `INFORMATION_SCHEMA.COLUMNS` and comparing the current schema to a target definition.

### References

- Brug dynamisk SQL med EXEC og sp-execute-sql - https://learn.microsoft.com/da-dk/training/modules/create-stored-procedures-table-valued-functions/5-use-dynamic-sql-exec-sp-execute-sql 
- MySQL Dynamic SQL - https://raw.githubusercontent.com/Compile-N-Run/Compile-N-Run/refs/heads/main/docs/database/mysql/7-mysql-stored-procedures/8-mysql-dynamic-sql.mdx 
- EXEC vs. sp_executeSQL - https://learn.microsoft.com/vi-vn/archive/blogs/turgays/exec-vs-sp_executesql 

---

## 2. Execution Contexts: EXECUTE() vs. sp_executesql

### Definitions

**Core Definition:** `EXECUTE()` (or `EXEC`) and `sp_executesql` are two mechanisms in SQL Server for executing dynamically built SQL strings, differing in parameterization support, security, and plan caching.

**Technical Definition:** `EXEC(@sql)` executes a string as a T-SQL batch. It does not accept parameters; any values must be concatenated into the string. `sp_executesql` is a system stored procedure that accepts a parameterized statement (`@stmt`), a parameter definition list (`@params`), and parameter values. Because the statement text remains constant across executions with different values, SQL Server can reuse the cached execution plan .

**Beginner-Friendly Explanation:** `EXEC` is like reading a sentence out loud exactly as written. `sp_executesql` is like reading a sentence with blanks, and then filling in the blanks separately. The second approach is safer and faster because the sentence (the SQL structure) stays the same, only the blanks (values) change.

### Purposes

- To execute dynamically constructed SQL strings.
- To choose the appropriate execution mechanism based on security and performance needs.
- To leverage plan caching for repeated executions with varying values.
- To parameterize dynamic queries for SQL injection prevention.

### Syntax Rules and Structure

**EXEC Syntax:**

```sql
DECLARE @sql VARCHAR(1000);
SET @sql = 'SELECT * FROM table WHERE id = ' + CAST(@id AS VARCHAR);
EXEC(@sql);
```

**sp_executesql Syntax:**

```sql
DECLARE @sql NVARCHAR(1000);
SET @sql = N'SELECT * FROM table WHERE id = @id';
EXEC sp_executesql @sql, N'@id INT', @id = @id;
```

**Component Breakdown:**

| Mechanism | Parameterization | Plan Caching | Security |
|-----------|-----------------|--------------|----------|
| `EXEC` | No | Ad-hoc plans (one per string) | Vulnerable to injection  |
| `sp_executesql` | Yes | Reuses parameterized plans | Safe when parameterized  |

**Syntax Rules:**

- `sp_executesql` requires `NVARCHAR` for `@stmt` and `@params`; `VARCHAR` is not allowed .
- The `@params` string defines parameter names and types (e.g., `N'@id INT, @name NVARCHAR(50)'`).
- Parameter values are passed after the `@params` string.
- `OUTPUT` parameters are supported in `sp_executesql` .
- `EXEC` can execute multiple statements separated by semicolons; `sp_executesql` can too.

**Constraints and Limitations:**

- `EXEC` cannot use parameters; values must be concatenated.
- `sp_executesql` `@stmt` must be a Unicode constant or variable; complex expressions with `+` are not allowed .
- Temp tables created in `EXEC` cannot use temp table caching .
- `sp_executesql` cannot reference variables declared in the calling batch .

### Annotated Complete Code Examples

**Example 1: EXEC vs. sp_executesql Plan Cache Comparison**

```sql
-- Clear plan cache
DBCC FREEPROCCACHE;

-- EXEC: Two different strings, two plans
DECLARE @str VARCHAR(MAX) = 'SELECT * FROM Person.Address WHERE AddressID IN (1,2)';
EXEC(@str);

SET @str = 'SELECT * FROM Person.Address WHERE AddressID IN (3,4)';
EXEC(@str);

-- Check cached plans: 2 plans for the same query structure
SELECT st.text FROM sys.dm_exec_cached_plans cp
CROSS APPLY sys.dm_exec_sql_text(cp.plan_handle) st
WHERE st.text LIKE '%Person.Address%';

-- sp_executesql: Same statement text, one plan
DECLARE @param1 INT = 1, @param2 INT = 2;
EXEC sp_executesql 
    N'SELECT * FROM Person.Address WHERE AddressID IN (@1,@2)',
    N'@1 INT, @2 INT',
    @param1, @param2;

SET @param1 = 3; SET @param2 = 4;
EXEC sp_executesql 
    N'SELECT * FROM Person.Address WHERE AddressID IN (@1,@2)',
    N'@1 INT, @2 INT',
    @param1, @param2;

-- Check cached plans: 1 plan reused
```

**Why this matters:** `EXEC` creates a new plan for each distinct string because the values are embedded in the text. `sp_executesql` keeps the text constant and passes values separately, so the same plan is reused .

### Real-World Cases

**Case 1: High-Frequency Dynamic Queries**

An application executes the same query structure with different parameter values thousands of times. `sp_executesql` reuses the cached plan, reducing CPU and memory overhead.

**Case 2: Security-Critical Dynamic Queries**

A search feature uses `sp_executesql` with parameters to prevent injection, even when filter values come from user input.

### References

- EXEC vs. sp_executeSQL - https://learn.microsoft.com/vi-vn/archive/blogs/turgays/exec-vs-sp_executesql 
- sp_executesql (Transact-SQL) - https://learn.microsoft.com/zh-cn/SQL/relational-databases/system-stored-procedures/sp-executesql-transact-sql 
- Brug dynamisk SQL med EXEC og sp-execute-sql - https://learn.microsoft.com/da-dk/training/modules/create-stored-procedures-table-valued-functions/5-use-dynamic-sql-exec-sp-execute-sql 

---

## 3. Parameterized Dynamic SQL

### Definitions

**Core Definition:** Parameterized dynamic SQL passes variable values into a dynamic SQL statement through parameters rather than string concatenation, keeping the SQL text constant and the data separate.

**Technical Definition:** In `sp_executesql`, the statement contains parameter placeholders (e.g., `@id`), and the `@params` argument defines their names and types. In MySQL, `PREPARE stmt FROM @sql` uses `?` placeholders and `EXECUTE stmt USING @var` supplies values. In PL/pgSQL, `EXECUTE ... USING` supplies values. The database driver or engine binds values to placeholders, never parsing them as SQL code .

**Beginner-Friendly Explanation:** Parameterization is like a fill-in-the-blank form. The form (SQL structure) is fixed. You write values in the blanks (parameters). The database knows which parts are instructions and which parts are just data.

### Purposes

- To prevent SQL injection by structurally separating code from data.
- To enable execution plan reuse for dynamic queries.
- To simplify dynamic query construction by avoiding manual quoting and escaping.
- To support type-safe value binding.

### Syntax Rules and Structure

**SQL Server sp_executesql Parameterized Syntax:**

```sql
EXEC sp_executesql 
    N'SELECT * FROM table WHERE column = @param',
    N'@param datatype',
    @param = value;
```

**MySQL Parameterized Syntax:**

```sql
SET @sql = 'SELECT * FROM table WHERE column = ?';
PREPARE stmt FROM @sql;
SET @value = 100;
EXECUTE stmt USING @value;
DEALLOCATE PREPARE stmt;
```

**PostgreSQL PL/pgSQL Parameterized Syntax:**

```sql
EXECUTE 'SELECT * FROM table WHERE column = $1' USING value;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `@stmt` / `@sql` | SQL text with parameter placeholders |
| `@params` | Parameter definitions (name + type) |
| `@param = value` | Parameter value binding |
| `USING` (MySQL/PG) | Value binding clause |
| `?` (MySQL) / `$1` (PG) | Placeholder symbols |

**Syntax Rules:**

- SQL Server: Parameter names in `@stmt` must match names in `@params` .
- MySQL: Placeholders are `?`; values are bound in order via `USING` .
- PostgreSQL: Placeholders are `$1`, `$2`, etc.; values bound via `USING` .
- Values must be passed as variables, constants, or simple expressions—not as complex expressions .
- Parameter types must be compatible with the values being passed.

**Constraints and Limitations:**

- SQL Server: `@stmt` and `@params` must be Unicode; `VARCHAR` causes an error .
- MySQL: Prepared statements cannot reference local variables within the prepared statement text .
- MySQL: Prepared statements are not allowed in stored functions or triggers .
- PostgreSQL: Identifiers (table/column names) cannot be parameterized; use `quote_ident()` and string concatenation for them .

### Annotated Complete Code Examples

**Example 1: Safe Parameterized Query (SQL Server)**

```sql
DECLARE @sql NVARCHAR(1000);
DECLARE @company NVARCHAR(128) = N'Sharp Bikes';

SET @sql = N'SELECT * FROM SalesLT.Customer WHERE CompanyName = @company';

EXEC sp_executesql 
    @sql,
    N'@company NVARCHAR(128)',
    @company = @company;
```

**Why this works:** The SQL text contains `@company` as a placeholder. The value `'Sharp Bikes'` is passed separately via the parameter list. The database treats it as a value, not as SQL code, eliminating injection risk .

**Example 2: MySQL Parameterized Dynamic Query**

```sql
CREATE PROCEDURE dynamic_query_with_params(IN emp_id INT)
BEGIN
    SET @sql = 'SELECT * FROM employees WHERE employee_id = ?';
    SET @id = emp_id;
    
    PREPARE stmt FROM @sql;
    EXECUTE stmt USING @id;
    DEALLOCATE PREPARE stmt;
END;

CALL dynamic_query_with_params(1003);
```

**Why this works:** The `?` placeholder is bound to `@id` via `EXECUTE ... USING`. The value `1003` is never concatenated into the SQL string .

### Real-World Cases

**Case 1: User Search with Multiple Filters**

A procedure accepts optional filter values. Parameterized dynamic SQL ensures that any user-supplied value is treated as data, not code.

**Case 2: Plan Cache Efficiency**

A high-volume application uses `sp_executesql` with parameters, reusing a single plan for thousands of executions with different values .

### References

- Brug dynamisk SQL med EXEC og sp-execute-sql - https://learn.microsoft.com/da-dk/training/modules/create-stored-procedures-table-valued-functions/5-use-dynamic-sql-exec-sp-execute-sql 
- sp_executesql (Transact-SQL) - https://learn.microsoft.com/zh-cn/SQL/relational-databases/system-stored-procedures/sp-executesql-transact-sql 
- MySQL Dynamic SQL - https://raw.githubusercontent.com/Compile-N-Run/Compile-N-Run/refs/heads/main/docs/database/mysql/7-mysql-stored-procedures/8-mysql-dynamic-sql.mdx 
- SQL Injection Prevention Cheat Sheet - https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html 

---

## 4. Security & Vulnerabilities

### Definitions

**Core Definition:** Dynamic SQL introduces security risks, primarily SQL injection, where untrusted input is concatenated into the SQL string and interpreted as executable code.

**Technical Definition:** SQL injection occurs when an application builds a SQL statement by concatenating untrusted input into the query string. The database cannot distinguish between developer-written SQL and attacker-injected SQL. Parameterization (prepared statements) is the definitive defense: it sends SQL text and parameter values separately, so values are never parsed as SQL .

**Beginner-Friendly Explanation:** If you let a user type something that goes directly into your SQL string, they might type something like `'; DROP TABLE users; --` and the database will run it. Parameterization prevents this by treating user input as pure data, never as instructions.

### Purposes

- To understand how SQL injection works conceptually.
- To recognize vulnerable code patterns.
- To apply the correct defense: parameterization first, allow-list validation for identifiers.
- To implement the principle of least privilege for database accounts.

### Syntax Rules and Structure

**Vulnerable Pattern (Never Do This):**

```sql
-- SQL Server: DANGEROUS
SET @sql = 'SELECT * FROM users WHERE username = ''' + @userInput + '''';
EXEC(@sql);
```

**Safe Pattern (Parameterization):**

```sql
-- SQL Server: SAFE
SET @sql = N'SELECT * FROM users WHERE username = @username';
EXEC sp_executesql @sql, N'@username NVARCHAR(50)', @username = @userInput;
```

**Identifier Allow-List Pattern (When Parameterization Is Impossible):**

```sql
-- Safe table name selection via allow-list
IF @tableName = 'Value1'
    SET @tableName = 'fooTable';
ELSE IF @tableName = 'Value2'
    SET @tableName = 'barTable';
ELSE
    THROW 50000, 'Invalid table name', 1;

SET @sql = 'SELECT * FROM ' + QUOTENAME(@tableName);
```

**Component Breakdown:**

| Defense | Mechanism | Effectiveness |
|---------|-----------|---------------|
| Parameterization | Values sent separately | Definitive for values  |
| Allow-list validation | Map input to known-safe identifiers | Required for table/column names  |
| Least privilege | App account lacks DDL/admin rights | Limits blast radius  |
| Input validation | Type/length/format checks | Secondary defense  |

**Syntax Rules:**

- Never concatenate user input into SQL text .
- Use parameters for all data values .
- Use allow-lists for identifiers (table names, column names, sort directions) .
- Grant only the minimum permissions needed (no `db_owner` for application accounts) .
- Stored procedures that use dynamic SQL must use parameterization or allow-lists .

**Constraints and Limitations:**

- Parameterization cannot be used for identifiers (table/column names) .
- ORMs can be injection-prone when using raw query escape hatches .
- WAFs are mitigations, not fixes .

### Annotated Complete Code Examples

**Example 1: Vulnerable vs. Safe Dynamic Query**

```sql
-- VULNERABLE: user input concatenated directly
DECLARE @userInput NVARCHAR(100) = N'admin'' OR ''1''=''1';
DECLARE @vulnSql NVARCHAR(500);
SET @vulnSql = N'SELECT * FROM users WHERE username = ''' + @userInput + '''';
EXEC(@vulnSql);
-- Result: returns ALL users (injection successful)

-- SAFE: parameterized
DECLARE @safeSql NVARCHAR(500);
SET @safeSql = N'SELECT * FROM users WHERE username = @username';
EXEC sp_executesql @safeSql, N'@username NVARCHAR(100)', @username = @userInput;
-- Result: returns 0 rows (input treated as literal value)
```

**Why this matters:** In the vulnerable version, the single quote in the input breaks out of the string literal, and `OR '1'='1'` makes the `WHERE` clause always true. In the safe version, the entire input is treated as a string value .

### Real-World Cases

**Case 1: Login Bypass Prevention**

A login form uses parameterized queries. Even if a user enters `' OR '1'='1`, the database looks for a username that literally equals that string, and the login fails.

**Case 2: Least Privilege for Application Accounts**

An application's database account is granted `SELECT` and `EXECUTE` only—no `DROP`, `ALTER`, or `db_owner` rights. If injection occurs, the attacker cannot destroy schema objects .

### References

- SQL Injection Prevention Cheat Sheet - https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html 
- SQLi Cheat Sheet: Detect and Prevent SQL Injection - https://safeguard.sh/resources/blog/sqli-cheat-sheet 
- OWASP Code Review Guide - https://wiki.owasp.org/images/7/78/OWASP_AlphaRelease_CodeReviewGuide2.0.pdf 

---

## 5. Metadata-Driven Queries

### Definitions

**Core Definition:** Metadata-driven queries use system catalogs (`INFORMATION_SCHEMA`, `sys.objects`, `pg_catalog`) to discover database schema information, then generate SQL dynamically based on that metadata.

**Technical Definition:** System catalogs expose tables, columns, data types, constraints, and relationships as queryable views. By querying these views, a program can discover the current schema at runtime and generate DDL or DML statements that adapt to the actual database structure. This enables generic tools, schema comparison utilities, and automated migration scripts .

**Beginner-Friendly Explanation:** Metadata-driven queries ask the database "what do you contain?" and use the answer to write new queries. For example, you can ask "what columns does the `employees` table have?" and then generate a `SELECT` statement listing only those columns.

### Purposes

- To generate code dynamically based on the actual database schema.
- To build generic utilities that work across different tables.
- To compare schemas and generate migration scripts.
- To automate schema documentation and discovery.
- To support type-safe query builders that introspect the database at design time.

### Syntax Rules and Structure

**SQL Server Metadata Queries:**

```sql
-- List tables and columns
SELECT TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME, DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'employees';

-- Using sys.objects
SELECT name, type_desc FROM sys.objects WHERE type = 'U';  -- User tables
```

**MySQL Metadata Queries:**

```sql
SELECT TABLE_NAME, COLUMN_NAME, DATA_TYPE, IS_NULLABLE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'mydb' AND TABLE_NAME = 'employees';
```

**PostgreSQL Metadata Queries:**

```sql
SELECT table_schema, table_name, column_name, data_type
FROM information_schema.columns
WHERE table_schema = 'public' AND table_name = 'employees';
```

**Component Breakdown:**

| View | Purpose |
|------|---------|
| `INFORMATION_SCHEMA.TABLES` | Table discovery  |
| `INFORMATION_SCHEMA.COLUMNS` | Column metadata  |
| `sys.objects` (SQL Server) | Object discovery |
| `pg_catalog` (PostgreSQL) | PostgreSQL system catalog |

**Syntax Rules:**

- `INFORMATION_SCHEMA` is a standard set of views available in most RDBMSs .
- SQL Server also provides `sys.*` catalog views, which are often more detailed and performant.
- PostgreSQL provides `pg_catalog` and `information_schema` (the latter is a compatibility layer) .
- Metadata queries can be combined with dynamic SQL to generate and execute statements based on discovered schema.

**Constraints and Limitations:**

- `INFORMATION_SCHEMA` may not expose all vendor-specific features.
- Performance of metadata queries depends on catalog complexity.
- Metadata queries themselves are typically static SQL; the dynamic part comes from using their results to build other queries.

### Annotated Complete Code Examples

**Example 1: Generate Column List Dynamically (SQL Server)**

```sql
DECLARE @tableName NVARCHAR(128) = N'employees';
DECLARE @columnList NVARCHAR(MAX);
DECLARE @sql NVARCHAR(MAX);

-- Get comma-separated column names from metadata
SELECT @columnList = STRING_AGG(QUOTENAME(COLUMN_NAME), ', ')
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = @tableName AND TABLE_SCHEMA = 'dbo';

-- Build and execute a SELECT with the discovered columns
SET @sql = N'SELECT ' + @columnList + N' FROM ' + QUOTENAME(@tableName);
EXEC sp_executesql @sql;

-- Result: SELECT [emp_id], [emp_name], [department], [salary] FROM [employees]
```

**Why this works:** The metadata query discovers the actual column names in the `employees` table. `STRING_AGG` concatenates them into a comma-separated list. The dynamic SQL then uses that list in a `SELECT` statement. This adapts automatically if columns are added or removed .

**Example 2: Schema Comparison for Migration (PostgreSQL)**

```sql
-- Find columns in target that don't exist in source
SELECT c.column_name, c.data_type
FROM information_schema.columns c
WHERE c.table_schema = 'public' 
  AND c.table_name = 'employees'
  AND c.column_name NOT IN (
      SELECT column_name 
      FROM information_schema.columns 
      WHERE table_schema = 'staging' AND table_name = 'employees'
  );
```

**Why this works:** The query compares column metadata between two schemas, identifying columns that need to be added or removed during migration. This is a common metadata-driven development pattern .

### Real-World Cases

**Case 1: Automated Schema Documentation**

A tool queries `INFORMATION_SCHEMA.COLUMNS` for all tables and generates a data dictionary with column names, types, and nullability.

**Case 2: Type-Safe Query Builders**

Tools like SqlArtisan query system catalogs to generate strongly-typed C# classes representing database tables, enabling compile-time type checking for SQL queries .

**Case 3: ETL Schema Evolution**

An ETL pipeline queries metadata to detect new columns in source tables and generates `ALTER TABLE ... ADD COLUMN` statements automatically .

### References

- dlt job_client_impl.py (schema update from INFORMATION_SCHEMA) - https://github.com/dlt-hub/dlt/blob/a77192f748defa732f7600f1bf16648a9adb700e/p/dlt/destinations/job_client_impl.py 
- Table Class Generation System | SqlArtisan - https://deepwiki.com/h-tacayama/SqlArtisan/2-table-class-generation-system 
- PostgreSQL 9.4 Documentation (EXECUTE and USING) - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.4/postgresql-9.4-A4.pdf 

---

## Summary Table of Dynamic SQL Features

| Feature | Purpose | Key Limitation |
|---------|---------|----------------|
| Dynamic Statement Construction | Build SQL at runtime | Injection risk without parameterization |
| EXEC vs. sp_executesql | Execute dynamic strings | `EXEC` cannot parameterize or cache plans |
| Parameterized Dynamic SQL | Prevent injection, reuse plans | Cannot parameterize identifiers |
| Security & Vulnerabilities | Defend against SQLi | Parameterization first, allow-list for identifiers |
| Metadata-Driven Queries | Generate code from schema | Vendor-specific catalog differences |

---

## Dialect Comparison: Dynamic SQL Mechanisms

| Feature | SQL Server | MySQL | PostgreSQL | Oracle |
|---------|-----------|-------|------------|--------|
| Execution Keyword | `EXEC` / `sp_executesql` | `PREPARE`/`EXECUTE` | `EXECUTE` (PL/pgSQL) | `EXECUTE IMMEDIATE` |
| Parameterization | `sp_executesql` | `PREPARE ... USING` | `EXECUTE ... USING` | `USING` clause |
| Placeholders | `@name` | `?` | `$1`, `$2` | `:name` |
| Identifier Quoting | `QUOTENAME()` | Backticks | `quote_ident()` | `DBMS_ASSERT.ENQUOTE_NAME` |
| Metadata Views | `INFORMATION_SCHEMA`, `sys.*` | `INFORMATION_SCHEMA` | `information_schema`, `pg_catalog` | `ALL_TABLES`, `ALL_TAB_COLUMNS` |

---

## References

- Brug dynamisk SQL med EXEC og sp-execute-sql - https://learn.microsoft.com/da-dk/training/modules/create-stored-procedures-table-valued-functions/5-use-dynamic-sql-exec-sp-execute-sql 
- EXEC vs. sp_executeSQL - https://learn.microsoft.com/vi-vn/archive/blogs/turgays/exec-vs-sp_executesql 
- sp_executesql (Transact-SQL) - https://learn.microsoft.com/zh-cn/SQL/relational-databases/system-stored-procedures/sp-executesql-transact-sql 
- MySQL Dynamic SQL - https://raw.githubusercontent.com/Compile-N-Run/Compile-N-Run/refs/heads/main/docs/database/mysql/7-mysql-stored-procedures/8-mysql-dynamic-sql.mdx 
- MySQL 8.0 Restrictions on Stored Programs - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/stored-program-restrictions.html 
- MySQL 5.1 Reference Manual (Prepared Statement Scope) - https://docs.oracle.com/cd/E19078-01/mysql/mysql-refman-5.1/refman-5.1.pdf 
- PostgreSQL 10 Documentation (PL/pgSQL EXECUTE) - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/10/postgresql-10-US.pdf 
- PostgreSQL 9.4 Documentation (EXECUTE and USING) - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.4/postgresql-9.4-A4.pdf 
- SQL Injection Prevention Cheat Sheet - https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html 
- SQLi Cheat Sheet: Detect and Prevent SQL Injection - https://safeguard.sh/resources/blog/sqli-cheat-sheet 
- OWASP Code Review Guide - https://wiki.owasp.org/images/7/78/OWASP_AlphaRelease_CodeReviewGuide2.0.pdf 
- dlt job_client_impl.py (schema update from INFORMATION_SCHEMA) - https://github.com/dlt-hub/dlt/blob/a77192f748defa732f7600f1bf16648a9adb700e/p/dlt/destinations/job_client_impl.py 
- Table Class Generation System | SqlArtisan - https://deepwiki.com/h-tacayama/SqlArtisan/2-table-class-generation-system 
- metadata issue - Microsoft Q&A - https://learn.microsoft.com/en-my/answers/questions/70143/metadata-issue 
- Note that parameter symbols can only be used for data values (PostgreSQL 9.4) - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.4/postgresql-9.4-A4.pdf 