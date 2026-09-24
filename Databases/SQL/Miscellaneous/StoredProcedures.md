# SQL Stored Procedures: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A stored procedure is a precompiled, named collection of SQL statements and procedural logic stored on the database server, which can be executed repeatedly by applications or users.

**Technical Definition:** A stored procedure is a database object created with `CREATE PROCEDURE` that encapsulates a sequence of SQL statements and procedural constructs (control flow, loops, exception handling, transactions). Procedures accept parameters (input, output, or both) and can return result sets, output values, or status codes. They are compiled and stored in the database catalog, offering performance benefits through plan reuse and reduced network traffic.

**Beginner-Friendly Explanation:** A stored procedure is like a saved recipe in a database. Instead of writing out all the steps every time you want to cook a dish, you write the recipe once, save it with a name, and then just say "make that dish." The database follows the saved instructions.

### Key Characteristics

- **Precompiled:** Stored procedures are parsed, validated, and compiled once, with query plans cached for reuse.
- **Reusable:** Can be called from applications, other procedures, or directly by users.
- **Parameterised:** Accept input parameters, return output parameters, and support default values.
- **Transactional:** Can contain transaction control statements (`BEGIN`, `COMMIT`, `ROLLBACK`).
- **Secure:** Can be granted `EXECUTE` permission without granting direct table access.
- **Dialect-specific:** Syntax and capabilities vary significantly across RDBMSs (T-SQL, PL/SQL, PL/pgSQL, MySQL).

### Prerequisites

- **SQL proficiency:** Solid understanding of `SELECT`, `INSERT`, `UPDATE`, `DELETE`.
- **Control flow concepts:** Familiarity with conditionals, loops, and exception handling from any programming language.
- **Transaction fundamentals:** Understanding of `BEGIN`, `COMMIT`, and `ROLLBACK`.
- **Database permissions:** `CREATE PROCEDURE` privilege on the target schema/database.

### Related Programming Areas

- **Application Development:** Encapsulating business logic in the database layer.
- **Data Engineering (ETL):** Batch processing, data transformations, and scheduled jobs.
- **Database Administration:** Maintenance tasks, monitoring, and administrative automation.
- **Security:** Row-level and column-level access control through procedure-based data access.

### Core Concepts / Features

1. Creating Procedures
2. Parameter Management
3. Procedure Execution
4. Control Flow & Conditional Logic
5. Loops and Iterations
6. Exception Handling

---

## 1. Creating Procedures

### Definitions

**Core Definition:** Creating a procedure involves defining its name, parameters, body (SQL statements and procedural logic), and optional characteristics, then storing it in the database catalog.

**Technical Definition:** The `CREATE PROCEDURE` statement registers a procedure with the database. The procedure body can be a single SQL statement or a compound statement (`BEGIN ... END`). Procedures are stored in the database catalog (`sys.procedures` in SQL Server, `information_schema.routines` in MySQL/PostgreSQL). The `OR REPLACE` clause (where supported) allows redefining an existing procedure without dropping it first.

**Beginner-Friendly Explanation:** Creating a procedure means writing down the steps and giving them a name. Once created, the database remembers the procedure and you can run it whenever you need it.

### Purposes

- To encapsulate complex SQL logic into a reusable, callable unit.
- To reduce network traffic by executing multiple statements server-side.
- To provide a stable interface that insulates applications from schema changes.
- To enforce security by granting `EXECUTE` instead of direct table permissions.
- To improve performance through query plan caching.

### Syntax Rules and Structure

**Complete General Syntax (SQL Server):**

```sql
CREATE [ OR ALTER ] PROCEDURE [ schema. ] procedure_name
    [ { @parameter_name data_type [ = default ] [ OUT | OUTPUT ] [ READONLY ] } [ , ... ] ]
    [ WITH { ENCRYPTION | RECOMPILE | EXECUTE AS ... } ]
    [ FOR REPLICATION ]
AS
    { [ BEGIN ] sql_statement [ ; ] [ ... ] [ END ] }
```

**Complete General Syntax (PostgreSQL):**

```sql
CREATE [ OR REPLACE ] PROCEDURE procedure_name ( [ argmode ] [ argname ] argtype [ { DEFAULT | = } default_expr ] [, ...] )
    { LANGUAGE lang_name | ... }
    AS 'definition'
```

**Complete General Syntax (MySQL):**

```sql
DELIMITER //
CREATE PROCEDURE procedure_name ( [ IN | OUT | INOUT ] parameter_name data_type [ , ... ] )
BEGIN
    -- SQL statements
END //
DELIMITER ;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `CREATE PROCEDURE` | Initiates procedure creation |
| `procedure_name` | The name of the procedure |
| `@parameter_name` / `parameter_name` | Parameter declarations |
| `data_type` | The parameter's data type |
| `AS` / `BEGIN` | Introduces the procedure body |
| `sql_statement` | The SQL statements to execute |

**Syntax Rules:**

- In SQL Server, the `AS` keyword introduces the procedure body; the body can be a single statement or a `BEGIN ... END` block.
- In MySQL, the `DELIMITER` command must be used to change the statement delimiter so the server receives the entire procedure definition .
- In PostgreSQL, procedures are created with `CREATE PROCEDURE` and called with `CALL` .
- The `OR REPLACE` clause (PostgreSQL, MySQL, Databricks) replaces an existing procedure without dropping it .
- `CREATE OR ALTER` (SQL Server 2016 SP1+) combines create and alter in one statement.

**Constraints and Limitations:**

- Procedure body syntax varies significantly across dialects.
- SQL Server procedure names are limited to 128 characters.
- MySQL requires `DELIMITER` changes when the body contains semicolons .
- PostgreSQL procedures cannot be executed as part of a `SELECT` query (unlike functions) .

### Annotated Complete Code Examples

**Example 1: Creating a Simple Procedure (SQL Server)**

```sql
-- Create a procedure that selects employees by department
CREATE PROCEDURE dbo.GetEmployeesByDepartment
    @DepartmentName NVARCHAR(50)
AS
BEGIN
    SET NOCOUNT ON;  -- Suppress "rows affected" messages
    SELECT emp_id, emp_name, salary
    FROM employees
    WHERE department = @DepartmentName;
END;
GO

-- Execute the procedure
EXEC dbo.GetEmployeesByDepartment @DepartmentName = 'Engineering';
```

**Why this output occurs:** The procedure accepts a department name and returns all employees in that department. `SET NOCOUNT ON` suppresses the row count messages that would otherwise be returned.

**Example 2: Creating a Procedure (MySQL)**

```sql
-- Change delimiter to allow semicolons inside the procedure
DELIMITER //

CREATE PROCEDURE GetEmployeeCount (IN dept_name VARCHAR(50), OUT emp_count INT)
BEGIN
    SELECT COUNT(*) INTO emp_count
    FROM employees
    WHERE department = dept_name;
END //

-- Restore the default delimiter
DELIMITER ;

-- Call the procedure
CALL GetEmployeeCount('Engineering', @count);
SELECT @count;
```

**Why this output occurs:** The `DELIMITER //` command tells the MySQL client to treat `//` as the statement terminator, allowing the semicolon inside the procedure body to be passed to the server . The procedure counts employees in a department and returns the count via the `OUT` parameter.

### Real-World Cases

**Case 1: Business Logic Encapsulation**

An e-commerce application creates a procedure that processes an order (validates inventory, inserts order, updates stock). The application calls the procedure with order details, and all logic executes server-side.

**Case 2: Security Layer**

A healthcare system creates procedures that expose only permitted patient data. Users are granted `EXECUTE` on the procedures but have no direct table access.

### References

- CREATE PROCEDURE (Transact-SQL) - SQL Server - https://learn.microsoft.com/sk-sk/SQL/t-sql/statements/create-procedure-transact-sql
- Create Stored Procedures - Training - https://learn.microsoft.com/en-us/training/modules/implement-programmability-objects/3-create-stored-procedures
- MySQL :: MySQL 9.1 Reference Manual :: 27.1 Defining Stored Programs - https://dev.mysql.com/doc/refman/9.1/en/stored-programs-defining.html
- PostgreSQL: Documentation: 13: CREATE PROCEDURE - https://www.postgresql.org/docs/13/sql-createprocedure.html

---

## 2. Parameter Management

### Definitions

**Core Definition:** Parameter management involves defining input parameters (values passed into the procedure), output parameters (values returned to the caller), and optional parameters with default values.

**Technical Definition:** Parameters are declared in the procedure header with a name, data type, and optional mode (`IN`, `OUT`, `INOUT`). Input parameters accept values from the caller. Output parameters return values via the `OUTPUT` keyword (SQL Server) or `OUT` (MySQL/PostgreSQL). Default values make parameters optional. The `READONLY` keyword (SQL Server) prevents modification of a parameter inside the procedure.

**Beginner-Friendly Explanation:** Parameters are the procedure's inputs and outputs. Input parameters are like ingredients you provide; output parameters are like the finished dish the procedure hands back to you. Default values let you skip providing some ingredients if you're happy with the standard version.

### Purposes

- To make procedures flexible and reusable with different input values.
- To return computed values to the caller without using result sets.
- To provide optional parameters that default to common values.
- To protect parameters from accidental modification inside the procedure.

### Syntax Rules and Structure

**SQL Server Parameter Syntax:**

```sql
CREATE PROCEDURE procedure_name
    @param1 data_type,
    @param2 data_type = default_value,
    @param3 data_type OUTPUT
AS
BEGIN
    -- procedure body
END;
```

**MySQL Parameter Syntax:**

```sql
CREATE PROCEDURE procedure_name (
    IN param1 data_type,
    OUT param2 data_type,
    INOUT param3 data_type
)
BEGIN
    -- procedure body
END;
```

**PostgreSQL Parameter Syntax:**

```sql
CREATE PROCEDURE procedure_name (
    param1 data_type,
    INOUT param2 data_type DEFAULT default_value
)
LANGUAGE plpgsql
AS $$
BEGIN
    -- procedure body
END;
$$;
```

**Component Breakdown:**

| Parameter Mode | Direction | Description |
|---------------|-----------|-------------|
| `IN` | Caller → Procedure | Input value (default mode) |
| `OUT` | Procedure → Caller | Output value (returned via parameter) |
| `INOUT` | Both | Input and output value |
| `OUTPUT` | Procedure → Caller | SQL Server keyword for output parameters |
| `DEFAULT` | — | Default value if parameter not supplied |
| `READONLY` | — | Parameter cannot be modified inside the procedure |

**Syntax Rules:**

- In SQL Server, output parameters use the `OUTPUT` keyword; cursor parameters also require `VARYING` .
- In MySQL, `OUT` parameters must be passed as user variables (`@var`) when calling the procedure .
- In PostgreSQL, `OUT` parameters are not supported for procedures; use `INOUT` instead .
- Default values must be constants or `NULL`; they cannot reference other parameters .
- When a default is specified for one parameter, all following parameters must also have defaults (PostgreSQL) .

**Constraints and Limitations:**

- `text`, `ntext`, and `image` cannot be `OUTPUT` parameters in SQL Server .
- Table-valued parameters can only be `INPUT` and must be `READONLY` .
- `OUT` parameters in MySQL must be passed as user variables, not expressions .
- PostgreSQL does not support `OUT` parameters for procedures .

### Annotated Complete Code Examples

**Example 1: Input, Output, and Default Parameters (SQL Server)**

```sql
CREATE PROCEDURE dbo.CalculateOrderTotal
    @OrderID int,
    @Discount decimal(3,2) = 0.00,  -- Optional with default
    @TotalAmount decimal(10,2) OUTPUT
AS
BEGIN
    SET NOCOUNT ON;
    
    SELECT @TotalAmount = SUM(Quantity * UnitPrice) * (1 - @Discount)
    FROM OrderDetails
    WHERE OrderID = @OrderID;
    
    RETURN 0;
END;
GO

-- Call the procedure with a declared output variable
DECLARE @Total decimal(10,2);
EXEC dbo.CalculateOrderTotal @OrderID = 1, @TotalAmount = @Total OUTPUT;
SELECT @Total AS OrderTotal;
```

**Why this output occurs:** The `@Discount` parameter has a default of `0.00`, so the caller can omit it. The `@TotalAmount` parameter is marked `OUTPUT`, so its value is returned to the caller after the procedure executes .

**Example 2: IN, OUT, and INOUT Parameters (MySQL)**

```sql
DELIMITER //

CREATE PROCEDURE GetServerInfo (OUT ver_param VARCHAR(25), INOUT incr_param INT)
BEGIN
    -- Set the OUT parameter to the server version
    SELECT VERSION() INTO ver_param;
    -- Increment the INOUT parameter
    SET incr_param = incr_param + 1;
END //

DELIMITER ;

-- Initialize the INOUT variable
SET @increment = 10;

-- Call the procedure
CALL GetServerInfo(@version, @increment);

-- Check the values
SELECT @version AS server_version, @increment AS incremented_value;
```

**Why this output occurs:** The `OUT` parameter `ver_param` receives the server version from the `SELECT ... INTO` statement. The `INOUT` parameter `incr_param` is initialized to 10 by the caller, incremented to 11 inside the procedure, and returned to the caller .

### Real-World Cases

**Case 1: Order Processing with Discount**

A procedure accepts an order ID and an optional discount percentage. It computes the total and returns it via an output parameter. The application displays the total to the user.

**Case 2: Counter Increment**

A procedure accepts an INOUT parameter representing a counter, increments it, and returns the new value. Used in batch processing to track progress.

### References

- Work with parameters - Training - https://learn.microsoft.com/en-us/training/modules/implement-programmability-objects/3-create-stored-procedures
- MySQL :: MySQL 9.7 Reference Manual :: 15.2.1 CALL Statement - https://dev.mysql.com/doc/refman/9.7/en/call.html
- CREATE PROCEDURE (Transact-SQL) - SQL Server - https://learn.microsoft.com/sk-sk/SQL/t-sql/statements/create-procedure-transact-sql
- PostgreSQL: Documentation: 13: CREATE PROCEDURE - https://www.postgresql.org/docs/13/sql-createprocedure.html

---

## 3. Procedure Execution

### Definitions

**Core Definition:** Procedure execution is the act of invoking a stored procedure, either explicitly with `EXECUTE`/`CALL` or implicitly when it is the first statement in a batch.

**Technical Definition:** Procedures are executed using `EXECUTE` (SQL Server) or `CALL` (MySQL, PostgreSQL). SQL Server allows implicit execution if the procedure name is the first statement in a batch. Procedures return a status code (0 for success, non-zero for failure) and may return result sets and output parameter values.

**Beginner-Friendly Explanation:** Executing a procedure means running it. You tell the database "run this procedure with these values" and it follows the saved instructions.

### Purposes

- To invoke a procedure and execute its logic.
- To pass input parameters and receive output values.
- To retrieve result sets from the procedure.
- To check the procedure's status code for error handling.

### Syntax Rules and Structure

**SQL Server Execution:**

```sql
-- Explicit execution
EXECUTE schema.procedure_name @param1 = value1, @param2 = value2;

-- Implicit execution (procedure is first statement in batch)
schema.procedure_name @param1 = value1;
```

**MySQL Execution:**

```sql
CALL procedure_name(value1, value2);
CALL procedure_name(@output_var);  -- With OUT parameters
```

**PostgreSQL Execution:**

```sql
CALL procedure_name(value1, value2);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `EXECUTE` / `EXEC` | SQL Server keyword for explicit execution |
| `CALL` | MySQL/PostgreSQL keyword for execution |
| `@param = value` | Named parameter passing |
| `RETURN` | Status code returned by SQL Server procedures |

**Syntax Rules:**

- SQL Server: `EXEC` and `EXECUTE` are synonymous; both are optional if the procedure is the first statement in a batch .
- MySQL: `CALL` is required; `OUT` parameters must be user variables .
- PostgreSQL: `CALL` is required; procedures cannot be executed in a `SELECT` .
- SQL Server procedures return an integer status code (0 = success) via `RETURN` .
- Result sets are returned implicitly by `SELECT` statements inside the procedure.

**Constraints and Limitations:**

- Procedure names should be schema-qualified for performance and to avoid ambiguity .
- In SQL Server, if a user-defined procedure has the same name as a system procedure, the user-defined one may never execute .
- MySQL requires `CLIENT_MULTI_RESULTS` for procedures returning multiple result sets .
- PostgreSQL procedures cannot be called from within a `SELECT` statement .

### Annotated Complete Code Examples

**Example 1: Explicit vs. Implicit Execution (SQL Server)**

```sql
-- Explicit execution (recommended)
EXECUTE SalesLT.uspGetCustomerCompany @LastName = N'Cannon', @FirstName = N'Chris';

-- Implicit execution (procedure is first statement in batch)
SalesLT.uspGetCustomerCompany N'Cannon', N'Chris';

-- Using positional parameters
EXEC SalesLT.uspGetCustomerCompany N'Cannon', N'Chris';
```

**Why this works:** The explicit form uses `EXECUTE` (or `EXEC`) followed by the schema-qualified procedure name. The implicit form omits `EXECUTE` because the procedure name is the first statement in the batch .

**Example 2: Capturing Output Parameters and Return Code (SQL Server)**

```sql
DECLARE @Total decimal(10,2);
DECLARE @ReturnCode int;

EXEC @ReturnCode = dbo.CalculateOrderTotal 
    @OrderID = 1, 
    @TotalAmount = @Total OUTPUT;

SELECT @Total AS TotalAmount, @ReturnCode AS StatusCode;
```

**Why this output occurs:** The `EXEC` statement assigns the return code to `@ReturnCode` and the output parameter value to `@Total`. Both are available after execution .

### Real-World Cases

**Case 1: Application Integration**

An application calls `EXEC dbo.ProcessOrder @OrderID = 123`. The procedure processes the order and returns a result set with order details.

**Case 2: Batch Scripts**

A SQL script executes multiple procedures in sequence using `EXEC` for each, checking return codes for error handling.

### References

- Eseguire una stored procedure - SQL Server - https://learn.microsoft.com/it-it/sql/relational-databases/stored-procedures/execute-a-stored-procedure
- Create Stored Procedures - Training - https://learn.microsoft.com/en-us/training/modules/implement-programmability-objects/3-create-stored-procedures
- MySQL :: MySQL 9.7 Reference Manual :: 15.2.1 CALL Statement - https://dev.mysql.com/doc/refman/9.7/en/call.html
- 38.4. User-Defined Procedures - https://www.postgresql.org/docs/14/xproc.html

---

## 4. Control Flow & Conditional Logic

### Definitions

**Core Definition:** Control flow and conditional logic in stored procedures determine the order in which statements execute based on conditions, using constructs like `IF...ELSE` and `CASE`.

**Technical Definition:** `IF...ELSE` executes a block of statements based on a Boolean predicate. `CASE` evaluates an expression and returns a value based on matching conditions. `BEGIN...END` groups statements into a block. These constructs enable branching logic within procedures.

**Beginner-Friendly Explanation:** Conditional logic is the "if this, then that" of stored procedures. If a condition is true, do one thing; otherwise, do something else. It's how procedures make decisions.

### Purposes

- To execute different code paths based on parameter values or data conditions.
- To validate input parameters and handle errors gracefully.
- To implement business rules that depend on data state.
- To control the flow of complex procedures.

### Syntax Rules and Structure

**SQL Server IF...ELSE:**

```sql
IF predicate
BEGIN
    -- statements if TRUE
END
ELSE
BEGIN
    -- statements if FALSE or UNKNOWN
END;
```

**SQL Server CASE:**

```sql
-- Simple CASE
CASE expression
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ELSE result_default
END

-- Searched CASE
CASE
    WHEN predicate1 THEN result1
    WHEN predicate2 THEN result2
    ELSE result_default
END
```

**MySQL IF...ELSE:**

```sql
IF condition THEN
    -- statements
ELSEIF condition THEN
    -- statements
ELSE
    -- statements
END IF;
```

**Component Breakdown:**

| Construct | Purpose |
|-----------|---------|
| `IF...ELSE` | Conditional branching based on Boolean predicate |
| `CASE` | Value selection based on matching conditions |
| `BEGIN...END` | Groups multiple statements into a block |
| `ELSEIF` / `ELSE IF` | Additional conditions (dialect-specific) |

**Syntax Rules:**

- SQL Server: The `IF` statement executes the following statement or `BEGIN...END` block if the predicate is `TRUE`; otherwise, the `ELSE` block executes .
- MySQL: Uses `IF...THEN...ELSEIF...ELSE...END IF` syntax.
- PostgreSQL: Uses `IF...THEN...ELSIF...ELSE...END IF` syntax (note: `ELSIF`, not `ELSEIF`).
- `CASE` is an expression and can be used in `SELECT`, `SET`, and `WHERE` clauses.

**Constraints and Limitations:**

- SQL Server's `IF` does not have `ELSEIF`; nested `IF` statements are used instead .
- MySQL's `CASE` and `IF` are statement-level constructs in procedures.
- Predicates that evaluate to `UNKNOWN` (due to `NULL`) are treated as `FALSE` for `IF` purposes.

### Annotated Complete Code Examples

**Example 1: IF...ELSE for Parameter Validation (SQL Server)**

```sql
CREATE PROCEDURE dbo.GetCustomerOrdersByDate
    @CustomerID int,
    @StartDate datetime = NULL,
    @EndDate datetime = NULL
AS
BEGIN
    -- Validate input
    IF @CustomerID IS NULL OR @CustomerID <= 0
    BEGIN
        RAISERROR('CustomerID must be a positive integer.', 16, 1);
        RETURN;
    END
    
    -- Query orders with optional date filters
    SELECT OrderID, OrderDate, TotalAmount
    FROM Orders
    WHERE CustomerID = @CustomerID
        AND (@StartDate IS NULL OR OrderDate >= @StartDate)
        AND (@EndDate IS NULL OR OrderDate <= @EndDate);
END;
```

**Why this works:** The `IF` statement checks whether `@CustomerID` is invalid. If so, it raises an error and returns early. Otherwise, the query executes with optional date filtering .

**Example 2: CASE for Conditional Value Selection (SQL Server)**

```sql
SELECT 
    emp_name,
    salary,
    CASE
        WHEN salary >= 100000 THEN 'High'
        WHEN salary >= 80000 THEN 'Medium'
        ELSE 'Standard'
    END AS salary_band
FROM employees;
```

**Why this output occurs:** The `CASE` expression evaluates each employee's salary and returns the matching band. Employees with salary >= 100000 get 'High', those >= 80000 get 'Medium', and the rest get 'Standard'.

### Real-World Cases

**Case 1: Input Validation**

A procedure validates that a start date is not after an end date. If invalid, it raises an error and returns without executing the main logic.

**Case 2: Tiered Pricing**

A procedure computes a discount based on order quantity using `CASE`. Orders over 100 units get 10% off, over 50 get 5%, otherwise no discount.

### References

- Use IF and WHILE blocks to control program flow - Training - https://learn.microsoft.com/mt-mt/training/modules/get-started-transact-sql-programming/5-use-if-while-blocks-to-control-program-flow
- Create Stored Procedures - Training - https://learn.microsoft.com/en-us/training/modules/implement-programmability-objects/3-create-stored-procedures

---

## 5. Loops and Iterations

### Definitions

**Core Definition:** Loops in stored procedures execute a block of statements repeatedly until a condition is met, using constructs like `WHILE`, `LOOP`, and `REPEAT`.

**Technical Definition:** `WHILE` executes a block as long as a condition is `TRUE`. `LOOP` (MySQL) executes indefinitely until a `LEAVE` statement is encountered. `REPEAT...UNTIL` (MySQL) executes at least once and repeats until a condition is `TRUE`. `BREAK` (SQL Server) and `LEAVE` (MySQL) exit loops; `CONTINUE` skips to the next iteration.

**Beginner-Friendly Explanation:** A loop is like a washing machine cycle: it repeats the same steps until the clothes are clean. You control when it stops with a condition.

### Purposes

- To process rows one at a time (row-by-row operations).
- To repeat a block of statements until a condition is satisfied.
- To iterate over a result set (cursor-based processing).
- To implement retry logic or counters.

### Syntax Rules and Structure

**SQL Server WHILE:**

```sql
WHILE condition
BEGIN
    -- statements
    IF some_condition
        BREAK;  -- Exit the loop
    IF another_condition
        CONTINUE;  -- Skip to next iteration
END;
```

**MySQL LOOP / REPEAT / WHILE:**

```sql
-- LOOP (infinite until LEAVE)
label: LOOP
    -- statements
    IF condition THEN
        LEAVE label;
    END IF;
END LOOP label;

-- REPEAT (execute then check)
REPEAT
    -- statements
UNTIL condition
END REPEAT;

-- WHILE (check then execute)
WHILE condition DO
    -- statements
END WHILE;
```

**PostgreSQL LOOP / WHILE / FOR:**

```sql
-- Basic LOOP
LOOP
    -- statements
    EXIT WHEN condition;
END LOOP;

-- WHILE
WHILE condition LOOP
    -- statements
END LOOP;

-- FOR (range)
FOR i IN 1..10 LOOP
    -- statements
END LOOP;
```

**Component Breakdown:**

| Construct | Behaviour |
|-----------|-----------|
| `WHILE` | Checks condition before each iteration |
| `LOOP` | Infinite loop; requires `LEAVE`/`EXIT` |
| `REPEAT...UNTIL` | Executes at least once; checks after |
| `BREAK` / `LEAVE` / `EXIT` | Exits the loop |
| `CONTINUE` / `ITERATE` | Skips to next iteration |

**Syntax Rules:**

- SQL Server: `WHILE` is the only loop construct; `BREAK` and `CONTINUE` control iteration .
- MySQL: Supports `LOOP`, `REPEAT`, and `WHILE`; `LEAVE` exits loops, `ITERATE` skips .
- PostgreSQL: Supports `LOOP`, `WHILE`, and `FOR`; `EXIT` and `CONTINUE` control iteration.
- MySQL loops can be labeled for targeted `LEAVE`/`ITERATE` .

**Constraints and Limitations:**

- Loops can be slow for large datasets; set-based operations are preferred.
- Infinite loops must be avoided with proper exit conditions.
- Cursor-based loops have overhead; use only when necessary.

### Annotated Complete Code Examples

**Example 1: WHILE Loop (SQL Server)**

```sql
DECLARE @empid AS INT = 1, @lname AS NVARCHAR(20);

WHILE @empid <= 5
BEGIN
    SELECT @lname = lastname 
    FROM HR.Employees
    WHERE empid = @empid;
    
    PRINT @lname;
    
    SET @empid += 1;
END;
```

**Why this output occurs:** The loop starts with `@empid = 1` and continues while `@empid <= 5`. Each iteration selects the last name for the current employee ID, prints it, and increments `@empid` .

**Example 2: REPEAT Loop (MySQL)**

```sql
DELIMITER //

CREATE PROCEDURE dorepeat(p1 INT)
BEGIN
    SET @x = 0;
    REPEAT
        SET @x = @x + 1;
    UNTIL @x > p1 END REPEAT;
END //

DELIMITER ;

CALL dorepeat(1000);
SELECT @x;  -- Output: 1001
```

**Why this output occurs:** The `REPEAT...UNTIL` loop increments `@x` until it exceeds `p1`. Starting at 0, it increments to 1001 before the condition `@x > 1000` becomes true .

### Real-World Cases

**Case 1: Batch Processing**

A procedure processes records in batches using a `WHILE` loop, committing after each batch to avoid long transactions.

**Case 2: Row-by-Row Validation**

A procedure uses a cursor and `LOOP` to validate each row individually, applying complex business rules that cannot be expressed in a single query.

### References

- Use IF and WHILE blocks to control program flow - Training - https://learn.microsoft.com/mt-mt/training/modules/get-started-transact-sql-programming/5-use-if-while-blocks-to-control-program-flow
- MySQL :: MySQL 9.1 Reference Manual :: 27.1 Defining Stored Programs - https://dev.mysql.com/doc/refman/9.1/en/stored-programs-defining.html
- -> -> IF v_order_source = 'Web' OR v_order_source = 'Mobile' THEN -> REPEAT -> SET v_order_source_count = v_order_source_count +... - https://issues.apache.org/jira/secure/attachment/13078623/LLM%20Randomly%20Generated%20Spark%20Scripts%20to%20MySQL%20Scripts.pdf

---

## 6. Exception Handling

### Definitions

**Core Definition:** Exception handling in stored procedures is the mechanism for detecting, responding to, and recovering from errors that occur during procedure execution.

**Technical Definition:** SQL Server uses `TRY...CATCH` blocks. MySQL uses `DECLARE ... HANDLER` for conditions. PostgreSQL uses `EXCEPTION` blocks in PL/pgSQL. Error raising uses `RAISERROR` or `THROW` (SQL Server) and `SIGNAL` (MySQL). Transaction rollback within exception handlers ensures atomicity.

**Beginner-Friendly Explanation:** Exception handling is the "what to do if something goes wrong" part of a procedure. Instead of crashing, the procedure catches the error, rolls back any partial changes, and either recovers or reports the problem.

### Purposes

- To catch errors and prevent procedure termination with unhandled exceptions.
- To roll back transactions when errors occur, maintaining data consistency.
- To log error details for diagnostics.
- To raise custom errors with meaningful messages.
- To implement retry logic for transient errors.

### Syntax Rules and Structure

**SQL Server TRY...CATCH:**

```sql
BEGIN TRY
    BEGIN TRANSACTION;
    -- SQL statements
    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    IF @@TRANCOUNT > 0
        ROLLBACK TRANSACTION;
    -- Error handling
    THROW;  -- Re-throw the caught error
END CATCH;
```

**SQL Server RAISERROR / THROW:**

```sql
-- RAISERROR (legacy)
RAISERROR('Error message', 16, 1);

-- THROW (modern, preferred)
THROW 50001, 'Error message', 1;
THROW;  -- Re-throw current error
```

**MySQL DECLARE ... HANDLER:**

```sql
DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
BEGIN
    -- Handler statements
    ROLLBACK;
END;
```

**MySQL SIGNAL:**

```sql
SIGNAL SQLSTATE '45000'
SET MESSAGE_TEXT = 'Custom error message';
```

**Component Breakdown:**

| Construct | Purpose |
|-----------|---------|
| `TRY...CATCH` | SQL Server error handling block |
| `DECLARE ... HANDLER` | MySQL condition handler |
| `EXCEPTION` | PostgreSQL error handling block |
| `RAISERROR` | SQL Server legacy error raising |
| `THROW` | SQL Server modern error raising/re-throwing |
| `SIGNAL` | MySQL custom error raising |
| `ROLLBACK` | Undo transaction changes |

**Syntax Rules:**

- SQL Server: `TRY` block contains the protected code; `CATCH` block handles errors. `THROW;` without parameters re-throws the current error .
- MySQL: Handlers must be declared after variable/condition declarations. `CONTINUE` handler continues execution; `EXIT` handler terminates the block .
- PostgreSQL: `EXCEPTION` block in PL/pgSQL catches errors; `RAISE` raises custom errors.
- Transaction rollback should be placed in the `CATCH`/handler block to undo partial changes.

**Constraints and Limitations:**

- `RAISERROR` is deprecated in favor of `THROW` in SQL Server .
- `THROW` always uses severity 16; `RAISERROR` allows custom severity .
- MySQL handlers must be declared at the beginning of the block .
- Errors in the `CATCH` block itself are not caught by the same handler.

### Annotated Complete Code Examples

**Example 1: TRY...CATCH with Transaction Rollback (SQL Server)**

```sql
CREATE PROCEDURE Production.uspDeleteWorkOrder (@WorkOrderID INT)
AS
BEGIN
    BEGIN TRY
        BEGIN TRANSACTION;
        
        -- Delete child rows first
        DELETE FROM Production.WorkOrderRouting
        WHERE WorkOrderID = @WorkOrderID;
        
        -- Delete parent rows
        DELETE FROM Production.WorkOrder
        WHERE WorkOrderID = @WorkOrderID;
        
        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        -- Roll back if transaction is still active
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION;
        
        -- Return error information
        DECLARE @ErrorMessage NVARCHAR(4000), @ErrorSeverity INT;
        SELECT @ErrorMessage = ERROR_MESSAGE(), @ErrorSeverity = ERROR_SEVERITY();
        RAISERROR(@ErrorMessage, @ErrorSeverity, 1);
    END CATCH;
END;
```

**Why this works:** The `TRY` block contains the transaction logic and `COMMIT`. If an error occurs (e.g., a constraint violation), the `CATCH` block rolls back the transaction and re-raises the error with its original message and severity .

**Example 2: MySQL CONTINUE HANDLER**

```sql
DELIMITER //

CREATE PROCEDURE ProcessOrders()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE order_id INT;
    DECLARE cur CURSOR FOR SELECT id FROM orders WHERE status = 'pending';
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
    BEGIN
        -- Log error and continue
        INSERT INTO error_log (message) VALUES ('Error processing order');
    END;
    
    OPEN cur;
    read_loop: LOOP
        FETCH cur INTO order_id;
        IF done THEN
            LEAVE read_loop;
        END IF;
        -- Process order
        UPDATE orders SET status = 'processed' WHERE id = order_id;
    END LOOP;
    CLOSE cur;
END //

DELIMITER ;
```

**Why this works:** The `CONTINUE HANDLER FOR SQLEXCEPTION` catches errors during processing, logs them, and continues to the next iteration. The `NOT FOUND` handler exits the cursor loop when no more rows are available .

### Real-World Cases

**Case 1: Financial Transaction Safety**

A banking procedure uses `TRY...CATCH` around a money transfer. If either the debit or credit fails, the `CATCH` block rolls back the entire transaction, preventing partial transfers.

**Case 2: ETL Error Logging**

An ETL procedure uses MySQL handlers to log errors and continue processing, ensuring that one bad record doesn't stop the entire batch.

### References

- THROW in Error Handling - Denali - https://learn.microsoft.com/en-us/archive/blogs/deepakbi/throw-in-error-handling-denali
- New THROW statement in SQL Server 2012 (vs RAISERROR) - https://learn.microsoft.com/zh-cn/archive/blogs/manub22/new-throw-statement-in-sql-server-2012-vs-raiserror
- MySQL :: MySQL 9.7 Reference Manual :: 15.6.7.2 DECLARE ... HANDLER Statement - https://dev.mysql.com/doc/refman/9.7/en/declare-handler.html
- CREATE PROCEDURE(Transact-SQL) - SQL Server - https://learn.microsoft.com/ko-kr/sql/t-sql/statements/create-procedure-transact-sql
- Managing a sql transaction with handling errors - Microsoft Q&A - https://learn.microsoft.com/en-in/answers/questions/1517145/managing-a-sql-transaction-with-handling-errors

---

## Summary Table of Stored Procedure Features

| Feature | Purpose | Key Limitation |
|---------|---------|----------------|
| Creating Procedures | Define reusable logic | Syntax varies by dialect |
| Input Parameters | Accept values from caller | Default values must be constants |
| Output Parameters | Return values to caller | Cannot be `text`/`image` in SQL Server |
| Default Parameters | Make parameters optional | All following parameters need defaults |
| EXECUTE / CALL | Invoke procedures | Schema-qualify for performance |
| IF...ELSE | Conditional branching | `ELSEIF` vs `ELSIF` varies by dialect |
| CASE | Value selection | Statement vs. expression context |
| WHILE / LOOP / REPEAT | Iteration | Performance overhead for row-by-row |
| BREAK / LEAVE / CONTINUE | Loop control | Labeled loops for nested control |
| TRY...CATCH | Error handling | `RAISERROR` deprecated; use `THROW` |
| DECLARE ... HANDLER | MySQL error handling | Must be declared at block start |
| SIGNAL | Raise custom errors | SQLSTATE must be valid |

---

## Dialect Comparison: Stored Procedure Syntax

| Feature | SQL Server (T-SQL) | MySQL | PostgreSQL (PL/pgSQL) |
|---------|-------------------|-------|----------------------|
| Create | `CREATE PROCEDURE` | `CREATE PROCEDURE` | `CREATE PROCEDURE` |
| Execute | `EXECUTE` / `EXEC` | `CALL` | `CALL` |
| Input Param | `@name type` | `IN name type` | `name type` |
| Output Param | `@name type OUTPUT` | `OUT name type` | `INOUT name type` |
| Default Value | `@name type = default` | `IN name type DEFAULT default` | `name type DEFAULT default` |
| Conditional | `IF...ELSE` | `IF...THEN...ELSEIF...END IF` | `IF...THEN...ELSIF...END IF` |
| Loop | `WHILE` | `LOOP`, `REPEAT`, `WHILE` | `LOOP`, `WHILE`, `FOR` |
| Break | `BREAK` | `LEAVE` | `EXIT` |
| Continue | `CONTINUE` | `ITERATE` | `CONTINUE` |
| Error Handling | `TRY...CATCH` | `DECLARE ... HANDLER` | `EXCEPTION` block |
| Raise Error | `RAISERROR` / `THROW` | `SIGNAL` | `RAISE` |

---

## References

- CREATE PROCEDURE (Transact-SQL) - SQL Server - https://learn.microsoft.com/sk-sk/SQL/t-sql/statements/create-procedure-transact-sql
- Create Stored Procedures - Training - https://learn.microsoft.com/en-us/training/modules/implement-programmability-objects/3-create-stored-procedures
- Eseguire una stored procedure - SQL Server - https://learn.microsoft.com/it-it/sql/relational-databases/stored-procedures/execute-a-stored-procedure
- THROW in Error Handling - Denali - https://learn.microsoft.com/en-us/archive/blogs/deepakbi/throw-in-error-handling-denali
- MySQL :: MySQL 9.1 Reference Manual :: 27.1 Defining Stored Programs - https://dev.mysql.com/doc/refman/9.1/en/stored-programs-defining.html
- MySQL :: MySQL 9.7 Reference Manual :: 15.2.1 CALL Statement - https://dev.mysql.com/doc/refman/9.7/en/call.html
- MySQL :: MySQL 26.7 Reference Manual :: 15.6.7 Condition Handling - https://dev.mysql.com/doc/refman/26.7/en/condition-handling.html
- PostgreSQL: Documentation: 13: CREATE PROCEDURE - https://www.postgresql.org/docs/13/sql-createprocedure.html
- Use IF and WHILE blocks to control program flow - Training - https://learn.microsoft.com/mt-mt/training/modules/get-started-transact-sql-programming/5-use-if-while-blocks-to-control-program-flow
- CREATE PROCEDURE(Transact-SQL) - SQL Server - https://learn.microsoft.com/ko-kr/sql/t-sql/statements/create-procedure-transact-sql
- CREATE PROCEDURE - Azure Databricks - https://learn.microsoft.com/en-my/azure/databricks/sql/language-manual/sql-ref-syntax-ddl-create-procedure
- CREATE PROCEDURE (Transact-SQL) - SQL Server - https://learn.microsoft.com/fil-ph/sql/t-sql/statements/create-procedure-transact-sql
- New THROW statement in SQL Server 2012 (vs RAISERROR) - https://learn.microsoft.com/zh-cn/archive/blogs/manub22/new-throw-statement-in-sql-server-2012-vs-raiserror
- MySQL :: MySQL 8.0 リファレンスマニュアル :: 25.1 ストアドプログラムの定義 - https://dev.mysql.com/doc/refman/8.0/ja/stored-programs-defining.html
- MySQL :: MySQL 8.0 Reference Manual :: Search Results - https://dev.mysql.com/doc/search/?q=int.html&d=201&p=22
- MySQL :: MySQL 9.7 Reference Manual :: 15.6.7.2 DECLARE ... HANDLER Statement - https://dev.mysql.com/doc/refman/9.7/en/declare-handler.html
- 38.4. User-Defined Procedures - https://www.postgresql.org/docs/14/xproc.html
- -> -> IF v_order_source = 'Web' OR v_order_source = 'Mobile' THEN -> REPEAT -> SET v_order_source_count = v_order_source_count +... - https://issues.apache.org/jira/secure/attachment/13078623/LLM%20Randomly%20Generated%20Spark%20Scripts%20to%20MySQL%20Scripts.pdf
- Managing a sql transaction with handling errors - Microsoft Q&A - https://learn.microsoft.com/en-in/answers/questions/1517145/managing-a-sql-transaction-with-handling-errors