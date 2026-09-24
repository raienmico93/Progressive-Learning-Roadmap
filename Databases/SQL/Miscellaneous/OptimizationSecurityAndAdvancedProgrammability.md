# SQL Optimization, Security & Advanced Programmability: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** This topic covers the techniques for making database code execute efficiently (performance tuning), securing it against unauthorized access and injection (security & permissions), verifying its correctness (debugging & testing), and understanding how the same concepts manifest differently across major database platforms.

**Technical Definition:** Performance tuning for database modules involves understanding plan caching (how execution plans are stored and reused), parameter sniffing (how the optimizer uses parameter values during compilation), and recompilation strategies (`WITH RECOMPILE`). Security encompasses the execution context of modules (definer vs. invoker rights), code protection (module encryption), and permission management (`GRANT EXECUTE`). Debugging and testing involve tracing, unit testing, and using platform-specific tools. Platform variations highlight the syntactic and behavioral differences between T-SQL, PL/SQL, PL/pgSQL, and MySQL routines.

**Beginner-Friendly Explanation:** Once you've written stored procedures and functions, you need to make sure they run fast, keep them safe from attackers, test them thoroughly, and know that the rules change depending on which database you use. This cheat sheet covers all four.

### Key Characteristics

- **Plan caching is automatic but fragile:** Execution plans are cached for reuse, but parameter values, SET options, and schema changes can invalidate or skew them .
- **Parameter sniffing is a feature, not a bug:** The optimizer uses parameter values to build the best plan for that invocation, but atypical values can produce plans that are terrible for typical values .
- **Security is layered:** Definer vs. invoker rights control whose permissions are checked; encryption hides code but does not protect against DBAs; `GRANT EXECUTE` is the primary access-control mechanism .
- **Platform syntax diverges significantly:** The same conceptual operation (e.g., dynamic SQL, exception handling, transaction control) uses completely different syntax across T-SQL, PL/SQL, PL/pgSQL, and MySQL.

### Prerequisites

- **Module creation:** Ability to create stored procedures and functions.
- **Execution plans:** Basic understanding of how the optimizer chooses plans.
- **Security fundamentals:** Understanding of users, roles, and permissions.
- **Transaction concepts:** Familiarity with `BEGIN`, `COMMIT`, `ROLLBACK`.

### Related Programming Areas

- **Database Administration:** Performance tuning, security auditing, and maintenance.
- **Application Development:** Writing secure and efficient database code.
- **DevOps/Testing:** Unit testing database logic and CI/CD for database changes.

### Core Concepts / Features

1. Performance Tuning
2. Security & Permissions
3. Debugging & Testing
4. Platform Variations

---

## 1. Performance Tuning

### Definitions

**Core Definition:** Performance tuning for database modules is the practice of ensuring that stored procedures, functions, and triggers execute with minimal resource consumption and maximum speed through plan caching, parameter handling, and recompilation strategies.

**Technical Definition:** When a module is first executed, SQL Server compiles it into an execution plan and stores it in the plan cache. The plan is "fingerprinted" by a query hash (based on T-SQL text) and a query plan hash (based on the plan structure) . Parameter sniffing occurs during compilation: the optimizer uses the parameter values passed to generate cardinality estimates and choose a plan . The `WITH RECOMPILE` option forces fresh compilation for every execution, bypassing plan reuse .

**Beginner-Friendly Explanation:** When you run a stored procedure the first time, the database figures out the best way to run it and saves that "recipe" (the plan). Next time, it uses the same recipe. But if the first time you ran it was with unusual data, the recipe might be wrong for normal data. `WITH RECOMPILE` says "forget the old recipe, make a new one every time."

### Purposes

- To maximize query execution speed by reusing efficient plans.
- To diagnose and resolve performance problems caused by parameter sniffing.
- To force recompilation when plans become stale or inappropriate.
- To analyze execution plans and identify bottlenecks (scans, lookups, spills).
- To use Query Store for historical performance analysis.

### Syntax Rules and Structure

**Plan Cache Inspection:**

```sql
-- Find cached plans for a stored procedure
SELECT cp.memory_object_address, cp.objtype, cp.refcounts, cp.usecounts,
       qs.query_plan_hash, qs.query_hash, qs.plan_handle, qs.sql_handle
FROM sys.dm_exec_cached_plans AS cp
CROSS APPLY sys.dm_exec_sql_text (cp.plan_handle)
CROSS APPLY sys.dm_exec_query_plan (cp.plan_handle)
INNER JOIN sys.dm_exec_query_stats AS qs ON qs.plan_handle = cp.plan_handle
WHERE text LIKE '%usp_SalesByCustomer%';
```

**WITH RECOMPILE:**

```sql
-- In procedure definition
CREATE PROCEDURE dbo.usp_GetData @Param INT
WITH RECOMPILE
AS
BEGIN
    SELECT * FROM Table WHERE Column = @Param;
END;

-- Or at execution time
EXEC dbo.usp_GetData @Param = 5 WITH RECOMPILE;
```

**OPTIMIZE FOR Hint:**

```sql
SELECT * FROM Products WHERE Keyword LIKE @Keyword
OPTION (OPTIMIZE FOR (@Keyword = 'typical%'));
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `sys.dm_exec_cached_plans` | Lists cached plans |
| `sys.dm_exec_query_stats` | Provides execution statistics per query |
| `WITH RECOMPILE` | Forces fresh compilation |
| `OPTIMIZE FOR` | Compiles using a specified parameter value |
| Query Store | Captures historical plan and runtime stats |

**Syntax Rules:**

- `WITH RECOMPILE` can be specified at procedure creation or execution time .
- `OPTIMIZE FOR` tells the optimizer to use a specific value (or `UNKNOWN`) for cardinality estimation .
- Changing `SET` options (e.g., `ANSI_NULLS`, `ARITHABORT`) can cause a different plan to be compiled and cached .
- Query Store (SQL Server 2016+) captures plans and runtime statistics for historical analysis .

**Constraints and Limitations:**

- `WITH RECOMPILE` adds compilation overhead to every execution; use selectively .
- Parameter sniffing problems are considered "by design" and require workarounds (dummy variables, `OPTIMIZE FOR`, `RECOMPILE`) .
- `OPTIMIZE FOR UNKNOWN` may produce a "one-size-fits-all" plan that is mediocre for all parameter values.

### Annotated Complete Code Examples

**Example 1: Parameter Sniffing Problem and Solution**

```sql
-- Problem: Procedure compiled with an atypical parameter
CREATE PROCEDURE dbo.SearchProducts @Keyword VARCHAR(100)
AS
SELECT * FROM Products WHERE Keyword LIKE @Keyword;

-- First execution with 'XBOX%' (few rows) → index seek + lookup plan
-- Later, plan is recompiled with 'KINECT%' (10% of rows) → table scan plan
-- Now 'XBOX%' executions use the table scan plan → terrible performance 

-- Solution 1: Use a dummy variable to hide the parameter
CREATE PROCEDURE dbo.SearchProducts_Fixed @Keyword VARCHAR(100)
AS
DECLARE @KeywordDummy VARCHAR(100);
SET @KeywordDummy = @Keyword;
SELECT * FROM Products WHERE Keyword LIKE @KeywordDummy;

-- Solution 2: OPTIMIZE FOR
SELECT * FROM Products WHERE Keyword LIKE @Keyword
OPTION (OPTIMIZE FOR (@Keyword = 'XBOX%'));
```

**Why these work:** The dummy variable prevents the optimizer from sniffing the actual parameter value, so the plan is based on average density . `OPTIMIZE FOR` forces compilation as if a specific (typical) value was passed.

**Example 2: WITH RECOMPILE for Mixed Workloads**

```sql
CREATE PROCEDURE dbo.ProcessData @InputData dbo.TableType READONLY
WITH RECOMPILE
AS
BEGIN
    -- Process the input data
    SELECT * FROM @InputData;
END;
```

**Why this works:** When the procedure is called with wildly varying input sizes (small and large), `WITH RECOMPILE` ensures a fresh, optimal plan for each call. The compilation overhead is negligible compared to the query execution savings .

### Real-World Cases

**Case 1: High-Concurrency Search**

An e-commerce search procedure suffered from parameter sniffing: a plan optimized for a rare search term caused full scans for common terms. The DBA added `OPTION (OPTIMIZE FOR UNKNOWN)` to produce a stable plan.

**Case 2: Mixed Batch Sizes**

An ETL procedure was called with both small and large datasets. The cached plan optimized for small data caused massive spills for large data. Adding `WITH RECOMPILE` resolved the issue .

### References

- What is Parameter Sniffing? - https://learn.microsoft.com/zh-tw/archive/blogs/mdegre/what-is-parameter-sniffing
- Query Processing Architecture Guide - https://learn.microsoft.com/el-gr/sql/relational-databases/query-processing-architecture-guide
- Plan Caching and Recompilation in SQL Server 2012 - https://download.microsoft.com/download/D/2/0/D20E1C5F-72EA-4505-9F26-FEF9550EFD44/Plan%20Caching%20and%20Recompilation%20in%20SQL%20Server%202012.docx
- Execution plan for stored procedures chosen by SQL Server Database Engine - https://learn.microsoft.com/ko-kr/archive/blogs/beatrice_popa/execution-plan-for-stored-procedures-chosen-by-sql-server-database-engine
- Profiling Query Performance In SQL Server With Extended Events - https://erikdarling.com/profiling-query-performance-in-sql-server-with-extended-events-the-easy-way/
- SQL Server 2025 Query Optimization Features - https://github.com/vasilyu1983/AI-Agents-public/blob/c95ab14ef8cf13e778412c4510960b9da9ef7700/frameworks/shared-skills/skills/data-sql-optimization/assets/mssql/template-mssql-explain.md

---

## 2. Security & Permissions

### Definitions

**Core Definition:** Security and permissions for database modules involve controlling whose rights are checked when a module executes (definer vs. invoker), protecting module code from unauthorized viewing (encryption), and granting the minimum necessary permissions to execute modules.

**Technical Definition:** In SQL Server, `EXECUTE AS` changes the execution context of a session or module, causing permission checks to be performed against a specified login or user instead of the caller . In PostgreSQL, `SECURITY DEFINER` functions execute with the privileges of the function owner, while `SECURITY INVOKER` functions execute with the caller's privileges. Module encryption (`WITH ENCRYPTION` in SQL Server) hides the module definition from ordinary users but does not protect against privileged users or memory dumps . `GRANT EXECUTE` is the standard permission for allowing users to run a stored procedure .

**Beginner-Friendly Explanation:** When you run a stored procedure, whose permissions does the database check? Your own, or the procedure owner's? That's the definer vs. invoker question. Encryption hides the code from curious eyes, but it's not real security. And `GRANT EXECUTE` is how you say "this user is allowed to run this procedure."

### Purposes

- To control whether a module runs with the caller's permissions or the owner's permissions.
- To hide sensitive business logic from unauthorized users.
- To grant the minimum necessary permissions for module execution.
- To protect against privilege escalation through ownership chaining.

### Syntax Rules and Structure

**EXECUTE AS (SQL Server):**

```sql
-- Change session context
EXECUTE AS USER = 'SomeUser';
-- ... operations run as SomeUser ...
REVERT;

-- Set context for a module
CREATE PROCEDURE dbo.SecureProc
WITH EXECUTE AS OWNER
AS
BEGIN
    SELECT * FROM SensitiveTable;
END;
```

**SECURITY DEFINER / INVOKER (PostgreSQL):**

```sql
CREATE FUNCTION secure_function()
RETURNS void
SECURITY DEFINER  -- Runs with owner's privileges
AS $$
BEGIN
    -- Function body
END;
$$ LANGUAGE plpgsql;

CREATE VIEW secure_view
WITH (security_invoker = true)  -- Runs with caller's privileges
AS SELECT * FROM sensitive_table;
```

**Module Encryption (SQL Server):**

```sql
CREATE PROCEDURE dbo.usp_GetUserBalance
    @UserId INT
WITH ENCRYPTION
AS
BEGIN
    SELECT Balance FROM Users WHERE Id = @UserId;
END;
```

**Granting Execute Permission:**

```sql
GRANT EXECUTE ON OBJECT::HumanResources.uspUpdateEmployeeHireInfo
    TO Recruiting11;
```

**Component Breakdown:**

| Mechanism | Platform | Effect |
|-----------|----------|--------|
| `EXECUTE AS` | SQL Server | Changes execution context to another principal  |
| `SECURITY DEFINER` | PostgreSQL | Function runs with owner's privileges |
| `SECURITY INVOKER` | PostgreSQL | Function runs with caller's privileges |
| `WITH ENCRYPTION` | SQL Server | Hides module definition from ordinary users  |
| `GRANT EXECUTE` | All | Grants permission to execute a module  |

**Syntax Rules:**

- `EXECUTE AS` can specify `LOGIN`, `USER`, `CALLER`, or `OWNER` .
- `WITH ENCRYPTION` makes the module definition inaccessible via `sp_helptext` or `sys.sql_modules`; the definition becomes empty or garbled .
- Encrypted procedures cannot be `ALTER`ed; they must be dropped and recreated .
- MySQL has no encryption mechanism for stored procedures; `SHOW CREATE PROCEDURE` always displays the source code .
- `GRANT EXECUTE` can be granted on a procedure, schema, or database level .

**Constraints and Limitations:**

- `WITH ENCRYPTION` does not protect against DBAs, `sysadmin` members, backup extraction, or memory dumps .
- Encryption is for obfuscation, not true security; it can be bypassed by privileged users.
- `EXECUTE AS USER` restricts access to resources outside the current database .
- PostgreSQL function code is visible to everyone who can query the catalog, even if they cannot execute the function .

### Annotated Complete Code Examples

**Example 1: EXECUTE AS for Privilege Control**

```sql
-- Create a procedure that runs with the owner's privileges
CREATE PROCEDURE dbo.GetEmployeeSalaries
WITH EXECUTE AS OWNER
AS
BEGIN
    SELECT emp_name, salary FROM HR.Employees;
END;

-- Grant EXECUTE to a low-privilege user
GRANT EXECUTE ON dbo.GetEmployeeSalaries TO ReportingUser;

-- ReportingUser can execute the procedure and see salaries,
-- even though they have no direct SELECT permission on HR.Employees
```

**Why this works:** `EXECUTE AS OWNER` causes the procedure to run with the permissions of its owner (typically a DBA or the schema owner), not the caller. This allows controlled access to sensitive data without granting direct table permissions .

**Example 2: Encryption vs. No Protection (MySQL)**

```sql
-- MySQL: No encryption option
CREATE PROCEDURE GetSensitiveData()
BEGIN
    SELECT * FROM secret_table;
END;

-- Anyone with access can see the definition
SHOW CREATE PROCEDURE GetSensitiveData;
-- Output includes the full source code
```

**Why this matters:** MySQL has no `WITH ENCRYPTION` equivalent. The only protection is permission isolation (revoking access to `mysql.proc`) and obfuscation, neither of which is a true security measure .

### Real-World Cases

**Case 1: Row-Level Security via Definer Rights**

A multi-tenant SaaS application creates procedures with `EXECUTE AS OWNER` that filter by tenant ID. Users can execute the procedures but cannot see other tenants' data because the procedure filters it.

**Case 2: Protecting Proprietary Logic**

A software vendor encrypts stored procedures containing pricing algorithms to prevent customers from reading or modifying them. The encryption deters casual viewing but does not stop a determined DBA .

### References

- EXECUTE AS (Transact-SQL) - https://learn.microsoft.com/ko-kr/sql/t-sql/statements/execute-as-transact-sql
- How to implement code hardening of SQL stored procedures - https://global.php.cn/faq/1797037269.html
- Grant a Permission to a Principal - https://learn.microsoft.com/th-th/sql/relational-databases/security/authentication-access/grant-a-permission-to-a-principal
- PostgreSQL SECURITY DEFINER / INVOKER - http://repo.postgrespro.ru/doc/sdm/18.3.2/en/postgres-A4.pdf
- Sécurité : paramètres de routine - https://public.dalibo.com/exports/formation/manuels/modules/p2/p2.handout.pdf

---

## 3. Debugging & Testing

### Definitions

**Core Definition:** Debugging and testing database code involves tracing execution, analyzing performance metrics, verifying correctness through unit tests, and using platform-specific debugging tools.

**Technical Definition:** Extended Events (SQL Server) provide a lightweight, configurable tracing framework that captures query plans, wait statistics, and parameter values during execution . Query Store captures historical execution plans and runtime statistics, enabling comparison of performance across plan changes . Execution plan analysis involves examining scan operations, join types, key lookups, sorts, and warnings to identify inefficiencies .

**Beginner-Friendly Explanation:** Debugging means watching what the database actually does when it runs your code. Testing means proving that it does what you think it does. Tools like Extended Events and Query Store give you X-ray vision into query execution.

### Purposes

- To capture live execution traces and identify slow queries.
- To analyze execution plans for inefficiencies (scans, lookups, spills).
- To compare plan performance over time using Query Store.
- To unit test database logic before deployment.
- To reproduce and diagnose parameter sniffing and other performance issues.

### Syntax Rules and Structure

**Extended Events (SQL Server):**

```sql
-- Create an Extended Events session
CREATE EVENT SESSION [SlowQueries] ON SERVER
ADD EVENT sqlserver.sql_statement_completed(
    ACTION(sqlserver.sql_text, sqlserver.query_plan_hash, sqlserver.database_name)
    WHERE ([duration] > 1000000))  -- > 1 second
ADD TARGET package0.event_file(SET filename=N'SlowQueries.xel')
WITH (MAX_DISPATCH_LATENCY=5 SECONDS);
GO

-- Start the session
ALTER EVENT SESSION [SlowQueries] ON SERVER STATE = START;
```

**Query Store (SQL Server 2016+):**

```sql
-- Enable Query Store
ALTER DATABASE YourDatabase SET QUERY_STORE = ON;
GO

-- Find top queries by duration
SELECT TOP 10
    q.query_id,
    qt.query_sql_text,
    rs.avg_duration/1000 AS avg_duration_ms,
    rs.avg_logical_io_reads,
    rs.avg_physical_io_reads
FROM sys.query_store_query q
JOIN sys.query_store_query_text qt ON q.query_text_id = qt.query_text_id
JOIN sys.query_store_plan p ON q.query_id = p.query_id
JOIN sys.query_store_runtime_stats rs ON p.plan_id = rs.plan_id
ORDER BY rs.avg_duration DESC;
```

**Execution Plan Analysis Checklist:**

| Category | What to Check |
|----------|---------------|
| Scans | Table scans on large tables, index scans where seeks would be better  |
| Joins | Nested loops for small sets, hash joins for large unsorted, merge joins for pre-sorted  |
| Key Lookups | High-cost lookups indicate missing covering indexes  |
| Sorts | High-cost sorts, spills to tempdb  |
| Warnings | Missing indexes, implicit conversions, parameter sniffing  |

**Component Breakdown:**

| Tool | Purpose |
|------|---------|
| Extended Events | Lightweight trace capture with query plans and parameters  |
| Query Store | Historical plan and runtime statistics  |
| `SET STATISTICS IO/TIME` | Per-query I/O and CPU metrics  |
| Execution Plan (SSMS) | Graphical plan analysis with operator costs  |

**Syntax Rules:**

- Extended Events sessions are created, started, and stopped with `CREATE`, `ALTER`, and `DROP` statements .
- Query Store must be enabled per database and has configurable retention and size limits .
- `SET STATISTICS IO ON` and `SET STATISTICS TIME ON` output metrics to the client .
- Actual execution plans include runtime row counts; estimated plans do not.

**Constraints and Limitations:**

- Extended Events add observer overhead; stop sessions when not needed .
- Query Store requires SQL Server 2016 or later and consumes storage.
- Execution plan analysis requires understanding of physical operators and their costs.

### Annotated Complete Code Examples

**Example 1: Capturing Slow Queries with Extended Events**

```sql
-- Create session to capture queries > 1 second
CREATE EVENT SESSION [SlowQueries] ON SERVER
ADD EVENT sqlserver.sql_statement_completed(
    ACTION(sqlserver.sql_text, sqlserver.query_plan_hash)
    WHERE ([duration] > 1000000))
ADD TARGET package0.event_file(SET filename=N'SlowQueries.xel');

-- Start the session
ALTER EVENT SESSION [SlowQueries] ON SERVER STATE = START;

-- Run your workload, then stop
ALTER EVENT SESSION [SlowQueries] ON SERVER STATE = STOP;

-- Read the captured data
SELECT 
    event_data.value('(event/@timestamp)[1]', 'datetime2') AS timestamp,
    event_data.value('(event/action[@name="sql_text"]/value)[1]', 'nvarchar(max)') AS sql_text,
    event_data.value('(event/data[@name="duration"]/value)[1]', 'bigint') AS duration_us
FROM (
    SELECT CAST(event_data AS XML) AS event_data
    FROM sys.fn_xe_file_target_read_file('SlowQueries*.xel', null, null, null)
) AS x
ORDER BY duration_us DESC;
```

**Why this works:** The Extended Events session captures only statements that exceed the duration threshold, with the SQL text and plan hash attached. The XML parsing extracts the relevant fields for analysis .

### Real-World Cases

**Case 1: Identifying Table Variable Bottlenecks**

Using Extended Events, a DBA discovered that a procedure spent 12 seconds inserting into a table variable, which prevented parallelism and lacked statistics. The fix was to use a temp table instead .

**Case 2: Tracking Plan Regressions**

Query Store revealed that a query's performance degraded after a plan change. The DBA forced the previous plan and investigated the cause of the regression .

### References

- Profiling Query Performance In SQL Server With Extended Events The Easy Way - https://erikdarling.com/profiling-query-performance-in-sql-server-with-extended-events-the-easy-way/
- SQL Server 2025 Query Optimization Features - https://github.com/vasilyu1983/AI-Agents-public/blob/c95ab14ef8cf13e778412c4510960b9da9ef7700/frameworks/shared-skills/skills/data-sql-optimization/assets/mssql/template-mssql-explain.md
- Plan Cache Internals - https://learn.microsoft.com/zh-hk/previous-versions/tn-archive/cc293624(v=technet.10)

---

## 4. Platform Variations

### Definitions

**Core Definition:** Platform variations are the syntactic and behavioral differences between the procedural languages of major RDBMSs: T-SQL (SQL Server), PL/SQL (Oracle), PL/pgSQL (PostgreSQL), and MySQL Stored Programs.

**Technical Definition:** Each RDBMS implements a proprietary procedural language with distinct syntax for variable declaration, control flow, cursors, exception handling, and transaction control. The SQL:1999 standard defines a persistent stored module (PSM) facility, but vendor implementations diverge significantly in syntax and semantics.

**Beginner-Friendly Explanation:** The same idea—like "declare a variable" or "catch an error"—is written differently in each database. This section maps the key differences so you can translate code between platforms.

### Purposes

- To translate database code between platforms.
- To understand platform-specific capabilities and limitations.
- To choose the right platform for specific requirements.
- To avoid syntax errors when working across multiple databases.

### Syntax Rules and Structure

**Variable Declaration:**

| Platform | Syntax |
|----------|--------|
| T-SQL | `DECLARE @var INT = 0;` |
| PL/pgSQL | `DECLARE var INT := 0;` |
| PL/SQL | `DECLARE var INT := 0;` |
| MySQL | `DECLARE var INT DEFAULT 0;` |

**Exception Handling:**

| Platform | Syntax |
|----------|--------|
| T-SQL | `BEGIN TRY ... END TRY BEGIN CATCH ... END CATCH` |
| PL/pgSQL | `BEGIN ... EXCEPTION WHEN ... THEN ... END;` |
| PL/SQL | `BEGIN ... EXCEPTION WHEN ... THEN ... END;` |
| MySQL | `DECLARE ... HANDLER FOR ... BEGIN ... END;` |

**Dynamic SQL:**

| Platform | Syntax |
|----------|--------|
| T-SQL | `EXEC sp_executesql @sql, N'@p INT', @p = @p;` |
| PL/pgSQL | `EXECUTE 'SELECT ... WHERE x = $1' USING val;` |
| PL/SQL | `EXECUTE IMMEDIATE 'SELECT ... WHERE x = :1' USING val;` |
| MySQL | `PREPARE stmt FROM @sql; EXECUTE stmt USING @val;` |

**Component Breakdown:**

| Feature | T-SQL | PL/pgSQL | PL/SQL | MySQL |
|---------|-------|----------|--------|-------|
| Variable prefix | `@` | None | None | None |
| Assignment | `SET` / `SELECT` | `:=` | `:=` | `SET` |
| Cursor loop | `WHILE @@FETCH_STATUS` | `FOR rec IN SELECT` | `FOR rec IN cursor` | `LOOP ... LEAVE` |
| Exception | `TRY...CATCH` | `EXCEPTION` | `EXCEPTION` | `HANDLER` |
| Dynamic SQL | `sp_executesql` | `EXECUTE ... USING` | `EXECUTE IMMEDIATE` | `PREPARE/EXECUTE` |
| Transaction var | `@@TRANCOUNT` | N/A | N/A | N/A |

**Syntax Rules:**

- T-SQL variables are prefixed with `@`; other platforms use bare identifiers.
- PL/pgSQL and PL/SQL use `:=` for assignment in procedural code.
- MySQL cursors are read-only and non-scrollable; T-SQL cursors support `SCROLL`.
- PL/pgSQL `EXECUTE` is distinct from SQL-level `EXECUTE`; it runs a dynamically constructed string.
- MySQL prepared statements cannot reference local variables directly.

**Constraints and Limitations:**

- MySQL has no `WITH ENCRYPTION` equivalent.
- Oracle PL/SQL does not support `TRY...CATCH`; it uses `EXCEPTION` blocks.
- PostgreSQL functions cannot contain transaction control statements; procedures can.
- SQL Server `EXECUTE AS` is not available in Azure Synapse Analytics .

### Annotated Complete Code Examples

**Example 1: Same Logic in Four Platforms**

**T-SQL:**
```sql
CREATE PROCEDURE dbo.GetCount @Dept VARCHAR(50), @Cnt INT OUTPUT
AS
BEGIN
    SELECT @Cnt = COUNT(*) FROM Employees WHERE Department = @Dept;
END;
```

**PL/pgSQL:**
```sql
CREATE OR REPLACE FUNCTION get_count(dept TEXT)
RETURNS INTEGER
AS $$
DECLARE
    cnt INTEGER;
BEGIN
    SELECT COUNT(*) INTO cnt FROM employees WHERE department = dept;
    RETURN cnt;
END;
$$ LANGUAGE plpgsql;
```

**PL/SQL:**
```sql
CREATE OR REPLACE PROCEDURE get_count(dept IN VARCHAR2, cnt OUT NUMBER)
AS
BEGIN
    SELECT COUNT(*) INTO cnt FROM employees WHERE department = dept;
END;
```

**MySQL:**
```sql
DELIMITER //
CREATE PROCEDURE get_count(IN dept VARCHAR(50), OUT cnt INT)
BEGIN
    SELECT COUNT(*) INTO cnt FROM employees WHERE department = dept;
END //
DELIMITER ;
```

**Why the differences matter:** Each platform uses different syntax for parameter modes (`OUTPUT` vs. `OUT`), assignment (`SELECT INTO`), and delimiter handling (MySQL). Understanding these differences is essential for cross-platform development.

### Real-World Cases

**Case 1: Database Migration**

A company migrating from SQL Server to PostgreSQL must translate all stored procedures from T-SQL to PL/pgSQL, rewriting exception handling, dynamic SQL, and variable declarations.

**Case 2: Multi-Platform Application**

An application supports both MySQL and PostgreSQL backends. The data access layer uses platform-specific SQL strings selected at runtime based on the connection type.

### References

- EXECUTE AS (Transact-SQL) - https://learn.microsoft.com/ko-kr/sql/t-sql/statements/execute-as-transact-sql
- PostgreSQL SECURITY DEFINER / INVOKER - http://repo.postgrespro.ru/doc/sdm/18.3.2/en/postgres-A4.pdf
- MySQL Stored Programs - https://docs.oracle.com/cd/E19078-01/mysql/mysql-refman-5.1/sql-syntax.html

---

## Summary Table: Platform Comparison

| Feature | T-SQL (SQL Server) | PL/pgSQL (PostgreSQL) | PL/SQL (Oracle) | MySQL |
|---------|-------------------|----------------------|-----------------|-------|
| Variable prefix | `@` | None | None | None |
| Exception handling | `TRY...CATCH` | `EXCEPTION` | `EXCEPTION` | `HANDLER` |
| Dynamic SQL | `sp_executesql` | `EXECUTE ... USING` | `EXECUTE IMMEDIATE` | `PREPARE/EXECUTE` |
| Module encryption | `WITH ENCRYPTION` | No | `WRAP` | No |
| Security context | `EXECUTE AS` | `SECURITY DEFINER/INVOKER` | `AUTHID` | `SQL SECURITY` |
| Transaction var | `@@TRANCOUNT` | N/A | N/A | N/A |
| Plan cache visibility | `sys.dm_exec_cached_plans` | `pg_stat_statements` | `V$SQL` | `performance_schema` |

---

## References

- What is Parameter Sniffing? - https://learn.microsoft.com/zh-tw/archive/blogs/mdegre/what-is-parameter-sniffing
- Query Processing Architecture Guide - https://learn.microsoft.com/el-gr/sql/relational-databases/query-processing-architecture-guide
- Plan Caching and Recompilation in SQL Server 2012 - https://download.microsoft.com/download/D/2/0/D20E1C5F-72EA-4505-9F26-FEF9550EFD44/Plan%20Caching%20and%20Recompilation%20in%20SQL%20Server%202012.docx
- Execution plan for stored procedures chosen by SQL Server Database Engine - https://learn.microsoft.com/ko-kr/archive/blogs/beatrice_popa/execution-plan-for-stored-procedures-chosen-by-sql-server-database-engine
- EXECUTE AS (Transact-SQL) - https://learn.microsoft.com/ko-kr/sql/t-sql/statements/execute-as-transact-sql
- How to implement code hardening of SQL stored procedures - https://global.php.cn/faq/1797037269.html
- Grant a Permission to a Principal - https://learn.microsoft.com/th-th/sql/relational-databases/security/authentication-access/grant-a-permission-to-a-principal
- Profiling Query Performance In SQL Server With Extended Events The Easy Way - https://erikdarling.com/profiling-query-performance-in-sql-server-with-extended-events-the-easy-way/
- Plan Cache Internals - https://learn.microsoft.com/zh-hk/previous-versions/tn-archive/cc293624(v=technet.10)
- SQL Server 2025 Query Optimization Features - https://github.com/vasilyu1983/AI-Agents-public/blob/c95ab14ef8cf13e778412c4510960b9da9ef7700/frameworks/shared-skills/skills/data-sql-optimization/assets/mssql/template-mssql-explain.md
- PostgreSQL SECURITY DEFINER / INVOKER - http://repo.postgrespro.ru/doc/sdm/18.3.2/en/postgres-A4.pdf
- MySQL Stored Programs - https://docs.oracle.com/cd/E19078-01/mysql/mysql-refman-5.1/sql-syntax.html