# SQL User-Defined Functions (UDFs): A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A User-Defined Function (UDF) is a named, reusable routine created by the user that accepts parameters, performs a calculation or query, and returns either a single scalar value or a result set (table).

**Technical Definition:** A UDF is a database object created with `CREATE FUNCTION` that encapsulates Transact-SQL (or PL/pgSQL, PL/SQL) logic. Unlike stored procedures, UDFs are designed to return a value or table, cannot modify database state, and are meant to be used within queries (in `SELECT`, `WHERE`, `JOIN`, or `FROM` clauses). SQL Server provides three types: scalar functions (return a single value), inline table-valued functions (return a table from a single `SELECT`), and multi-statement table-valued functions (return a table built through procedural logic) .

**Beginner-Friendly Explanation:** A UDF is like a custom formula or a mini-query that you save and reuse. For example, you can create a function that takes a price and tax rate and returns the final price. Then you can use that function anywhere you would use a built-in function or a table.

### Key Characteristics

- **Reusable encapsulation:** Encapsulates logic for reuse across queries, applications, and other functions .
- **Returns a value or table:** Scalar UDFs return one value; table-valued UDFs return a result set .
- **Read-only:** UDFs cannot modify database state (no `INSERT`, `UPDATE`, `DELETE` on permanent tables) .
- **Limited error handling:** UDFs do not support `TRY...CATCH`, `RAISERROR`, or `@ERROR` in SQL Server .
- **Nesting limit:** UDFs can be nested up to 32 levels .
- **Performance varies by type:** Scalar UDFs have historically caused severe row-by-row (RBAR) performance issues; inline TVFs are the preferred choice for performance .

### Prerequisites

- **Basic SQL knowledge:** `SELECT`, `FROM`, `WHERE`, `JOIN`, and aggregate functions.
- **Programming concepts:** Variables, control flow (`IF...ELSE`), and return values.
- **Database permissions:** `CREATE FUNCTION` privilege on the target schema.
- **Type awareness:** Understanding of scalar vs. table return types.

### Related Programming Areas

- **Application Development:** Encapsulating business calculations and data access logic.
- **Data Analysis:** Reusable computed columns and filtered views with parameters.
- **Data Engineering:** Transforming data in ETL pipelines with reusable functions.
- **Performance Tuning:** Understanding when UDFs help vs. hurt query performance.

### Core Concepts / Features

1. Scalar Functions
2. Table-Valued Functions (TVFs)
3. Function Parameters
4. Function Limitations
5. Performance Costs

---

## 1. Scalar Functions

### Definitions

**Core Definition:** A scalar function is a UDF that accepts parameters and returns a single scalar value (e.g., a number, string, or date).

**Technical Definition:** A scalar UDF is created with `RETURNS <data_type>` and contains a `BEGIN...END` block that performs calculations and ends with a `RETURN @value` statement. Scalar UDFs can be used anywhere a scalar expression is valid—in `SELECT` lists, `WHERE` clauses, `SET` statements, and `ORDER BY` clauses. However, they are executed once per row in the outer query, creating a row-by-row (RBAR) processing pattern that prevents parallelism and often produces poor execution plans .

**Beginner-Friendly Explanation:** A scalar function is like a custom calculator. You give it some inputs (like a price and tax rate), and it gives you back one number (the final price). You can use it in any query where you'd use a calculation.

### Purposes

- To encapsulate simple, reusable calculations (e.g., formatting, mathematical operations).
- To provide a single-value result for use in `SELECT` lists or predicates.
- To simplify procedural code by wrapping complex logic into a callable unit .
- To support computed columns and constraints with custom logic.

### Syntax Rules and Structure

**Complete General Syntax (SQL Server):**

```sql
CREATE FUNCTION [schema_name.]function_name
(
    @parameter_name data_type [ = default_value ] [ , ... ]
)
RETURNS return_data_type
[ WITH { SCHEMABINDING | RETURNS NULL ON NULL INPUT | ... } ]
AS
BEGIN
    -- function body
    RETURN @return_value;
END;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `CREATE FUNCTION` | Initiates function creation |
| `@parameter_name` | Input parameter declaration |
| `RETURNS return_data_type` | Specifies the scalar return type |
| `WITH SCHEMABINDING` | Binds function to referenced objects |
| `RETURNS NULL ON NULL INPUT` | Skips execution if any input is NULL |
| `BEGIN...END` | Groups the function body statements |
| `RETURN @return_value` | Returns the computed scalar value |

**Syntax Rules:**

- The `RETURN` statement is mandatory and must return a value of the declared data type.
- Scalar UDFs can use variables (`DECLARE @var`), `SET`, and `SELECT @var = ...` for assignments .
- `SCHEMABINDING` is recommended for performance and prevents underlying object changes .
- `RETURNS NULL ON NULL INPUT` short-circuits execution when any input is NULL, avoiding unnecessary function calls .

**Constraints and Limitations:**

- Cannot modify database state (no `INSERT`, `UPDATE`, `DELETE`) .
- Cannot use `TRY...CATCH` or `RAISERROR` .
- Cannot call stored procedures (can call extended stored procedures) .
- Cannot use dynamic SQL or temporary tables (table variables allowed) .
- Cannot use side-effecting functions like `NEWID()`, `RAND()`, `NEWSEQUENTIALID()` .
- Executed once per row in outer query (RBAR) .
- Prevents parallelism in the query plan .

### Annotated Complete Code Examples

**Example 1: Simple Scalar Function**

```sql
CREATE FUNCTION dbo.CalculateTax
(
    @Price MONEY,
    @TaxRate DECIMAL(5,4)
)
RETURNS MONEY
WITH SCHEMABINDING
AS
BEGIN
    DECLARE @TaxAmount MONEY;
    SET @TaxAmount = @Price * @TaxRate;
    RETURN @TaxAmount;
END;
GO

-- Usage in a query
SELECT ProductID, ListPrice, dbo.CalculateTax(ListPrice, 0.08) AS TaxAmount
FROM Production.Product;
```

**Why this works:** The function encapsulates the tax calculation. `SCHEMABINDING` binds it to the `Production.Product` table (though it doesn't reference it) and provides performance benefits. Each row in the result calls the function once .

### Real-World Cases

**Case 1: Formatting Sensitive Data**

A UDF masks credit card numbers by returning `'****-****-****-' + RIGHT(@CardNumber, 4)`. Used in `SELECT` statements for customer-facing reports.

**Case 2: Complex Business Calculations**

A UDF computes customer lifetime value based on purchase history parameters. Used in analytical queries, though performance may suffer on large datasets .

### References

- Create User-defined Functions (Database Engine) - https://learn.microsoft.com/en-us/sql/relational-databases/user-defined-functions/create-user-defined-functions-database-engine
- UDF and Performance - https://learn.microsoft.com/en-us/archive/blogs/vipulshah/udf-and-performance
- Create user-defined functions - Training - https://learn.microsoft.com/en-gb/training/modules/create-stored-procedures-table-valued-functions/6-create-inline

---

## 2. Table-Valued Functions (TVFs)

### Definitions

**Core Definition:** A table-valued function (TVF) is a UDF that returns a result set (a table) instead of a single value, allowing it to be used in the `FROM` clause of a query like a parameterized view.

**Technical Definition:** SQL Server supports two types of TVFs: **inline table-valued functions (ITVFs)** and **multi-statement table-valued functions (MSTVFs)**. An inline TVF contains a single `RETURN (SELECT ...)` statement; SQL Server infers the return schema from the query, and the optimizer treats it like a view with parameters, often producing optimal plans . An MSTVF uses a `BEGIN...END` block with a declared table variable (`RETURNS @table TABLE (...)`) and can contain multiple statements, loops, and conditional logic; however, the optimizer treats it as a black box with limited cardinality estimates, often leading to poor performance .

**Beginner-Friendly Explanation:** A table-valued function is like a saved query that takes parameters and returns a table. You can use it in a `JOIN` or `FROM` clause just like a regular table. Inline TVFs are simple and fast. Multi-statement TVFs can do more complex logic but are slower.

### Purposes

- To encapsulate parameterized queries for reuse in `JOIN` and `FROM` clauses .
- To provide a modular alternative to complex views with filtering parameters.
- To return calculated or filtered result sets for further processing.
- To support `CROSS APPLY` and `OUTER APPLY` operations.

### Syntax Rules and Structure

**Inline TVF Syntax:**

```sql
CREATE FUNCTION [schema.]function_name ( @param data_type )
RETURNS TABLE
AS
RETURN
(
    SELECT columns
    FROM table
    WHERE condition = @param
);
```

**Multi-Statement TVF Syntax:**

```sql
CREATE FUNCTION [schema.]function_name ( @param data_type )
RETURNS @result TABLE ( column1 data_type, column2 data_type, ... )
AS
BEGIN
    INSERT INTO @result (column1, column2, ...)
    SELECT ...
    FROM ...
    WHERE ...;
    
    -- Additional statements
    RETURN;
END;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `RETURNS TABLE` | Inline TVF: schema inferred from query |
| `RETURNS @result TABLE (...)` | MSTVF: explicit table schema declaration |
| `RETURN (SELECT ...)` | Inline TVF: single query returning result |
| `INSERT INTO @result` | MSTVF: populates the table variable |
| `RETURN;` | MSTVF: returns the populated table variable |

**Syntax Rules:**

- Inline TVFs require a single `RETURN` statement with a `SELECT` query .
- MSTVFs must declare the return table schema explicitly and use `INSERT INTO @table` to populate it .
- Inline TVFs can be used with `CROSS APPLY` and `OUTER APPLY` operators .
- MSTVFs can contain multiple statements, conditional logic, and loops.
- Inline TVFs are the preferred choice for performance .

**Constraints and Limitations:**

- Inline TVFs cannot contain `BEGIN...END` blocks or multiple statements.
- MSTVFs have a fixed cardinality estimate (100 rows in SQL Server 2014+, 1 in earlier versions) unless Interleaved Execution is used (SQL Server 2017+) .
- MSTVFs cannot be inlined by the optimizer, leading to less efficient plans .
- Both TVF types cannot modify database state .

### Annotated Complete Code Examples

**Example 1: Inline Table-Valued Function**

```sql
CREATE FUNCTION dbo.GetCustomerOrders
(
    @CustomerID INT
)
RETURNS TABLE
AS
RETURN
(
    SELECT OrderID, OrderDate, TotalAmount
    FROM Sales.Orders
    WHERE CustomerID = @CustomerID
);
GO

-- Usage in a query with additional filtering
SELECT OrderID, OrderDate, TotalAmount
FROM dbo.GetCustomerOrders(1001)
WHERE OrderDate >= '2024-01-01';
```

**Why this works:** The inline TVF parameterizes the customer filter. The optimizer treats it as a parameterized view, allowing predicate pushdown and optimal plan generation .

**Example 2: Multi-Statement Table-Valued Function**

```sql
CREATE FUNCTION dbo.GetProductSalesSummary
(
    @StartDate DATE,
    @EndDate DATE
)
RETURNS @SalesSummary TABLE
(
    ProductID INT,
    ProductName NVARCHAR(100),
    TotalQuantity INT,
    TotalRevenue DECIMAL(18,2),
    AveragePrice DECIMAL(18,2)
)
AS
BEGIN
    INSERT INTO @SalesSummary
    SELECT
        p.ProductID,
        p.ProductName,
        SUM(od.Quantity) AS TotalQuantity,
        SUM(od.Quantity * od.UnitPrice) AS TotalRevenue,
        AVG(od.UnitPrice) AS AveragePrice
    FROM Production.Products p
    INNER JOIN Sales.OrderDetails od ON p.ProductID = od.ProductID
    INNER JOIN Sales.Orders o ON od.OrderID = o.OrderID
    WHERE o.OrderDate BETWEEN @StartDate AND @EndDate
    GROUP BY p.ProductID, p.ProductName;
    
    RETURN;
END;
GO

-- Usage with CROSS APPLY
SELECT c.CustomerName, s.ProductName, s.TotalRevenue
FROM Customers c
CROSS APPLY dbo.GetProductSalesSummary('2024-01-01', '2024-12-31') s
WHERE s.TotalRevenue > 10000
ORDER BY s.TotalRevenue DESC;
```

**Why this works:** The MSTVF declares the return schema and populates it via `INSERT INTO @SalesSummary`. The function can contain complex logic. However, the optimizer has limited visibility into the row count estimate, which can lead to suboptimal plans when joined .

### Real-World Cases

**Case 1: Parameterized Reporting Views**

An inline TVF accepts a date range and returns sales data. Report writers use it in `JOIN` clauses, avoiding complex `WHERE` clauses in every report.

**Case 2: Complex Business Logic**

An MSTVF processes multiple steps (e.g., validate, filter, aggregate, rank) and returns a summary table. Used when the logic cannot be expressed in a single `SELECT` .

### References

- Create Table-Valued Functions - Training - https://learn.microsoft.com/en-us/training/modules/implement-programmability-objects/5-create-table-valued-functions
- Create user-defined functions - Training - https://learn.microsoft.com/en-gb/training/modules/create-stored-procedures-table-valued-functions/6-create-inline
- Create User-defined Functions (Database Engine) - https://learn.microsoft.com/en-us/sql/relational-databases/user-defined-functions/create-user-defined-functions-database-engine

---

## 3. Function Parameters

### Definitions

**Core Definition:** Function parameters are the inputs that a UDF accepts, defined with a name, data type, and optional default value or determinism characteristic.

**Technical Definition:** UDF parameters are declared in the function header with `@name data_type`. Parameters are input-only for functions (no `OUTPUT` parameters, unlike stored procedures). Default values can be specified for optional parameters. In MySQL, functions can be marked `DETERMINISTIC` or `NOT DETERMINISTIC`, and with characteristics like `READS SQL DATA`, `NO SQL`, or `MODIFIES SQL DATA` . SQL Server uses `RETURNS NULL ON NULL INPUT` to short-circuit execution when inputs are NULL .

**Beginner-Friendly Explanation:** Parameters are the "ingredients" your function needs. You define what type each ingredient is (number, text, date) and whether it's optional. Some databases also let you declare whether the function always gives the same output for the same input (deterministic).

### Purposes

- To pass values into the function for processing.
- To provide optional parameters with sensible defaults.
- To declare determinism for query optimizer hints.
- To enable `RETURNS NULL ON NULL INPUT` optimization.

### Syntax Rules and Structure

**SQL Server Parameter Syntax:**

```sql
CREATE FUNCTION dbo.fn_example
(
    @param1 INT,
    @param2 VARCHAR(50) = 'default',
    @param3 DATE = NULL
)
RETURNS INT
WITH RETURNS NULL ON NULL INPUT
AS
BEGIN
    RETURN @param1;
END;
```

**MySQL Parameter Syntax:**

```sql
CREATE FUNCTION fn_example (param1 INT, param2 VARCHAR(50))
RETURNS INT
DETERMINISTIC
READS SQL DATA
RETURN param1 + LENGTH(param2);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `@param data_type` | Parameter declaration (SQL Server) |
| `param data_type` | Parameter declaration (MySQL/PostgreSQL) |
| `= default_value` | Optional default value |
| `DETERMINISTIC` | Function always returns same result for same input |
| `NOT DETERMINISTIC` | Function may return different results |
| `READS SQL DATA` | Function reads data but does not modify |
| `NO SQL` | Function contains no SQL statements |
| `RETURNS NULL ON NULL INPUT` | Skip execution if any input is NULL |

**Syntax Rules:**

- All function parameters are input-only; functions cannot have `OUTPUT` parameters (unlike procedures) .
- Default values must be constants; they cannot reference other parameters.
- In MySQL, `DETERMINISTIC` is required for replication safety if the function is used in certain contexts .
- `RETURNS NULL ON NULL INPUT` causes the function to return NULL immediately if any input is NULL, avoiding function body execution .

**Constraints and Limitations:**

- Functions cannot return multiple result sets .
- SQL Server functions cannot have `OUTPUT` parameters .
- MySQL functions must declare determinism characteristics for certain uses .
- `text`, `ntext`, and `image` data types cannot be used as parameters in some contexts.

### Annotated Complete Code Examples

**Example 1: Parameters with Defaults and Determinism (MySQL)**

```sql
CREATE FUNCTION CalculateDiscount
(
    Price DECIMAL(10,2),
    DiscountRate DECIMAL(3,2)
)
RETURNS DECIMAL(10,2)
DETERMINISTIC
READS SQL DATA
RETURN Price * (1 - DiscountRate);
```

**Why this works:** The function is marked `DETERMINISTIC` (always same output for same input) and `READS SQL DATA` (it reads but doesn't modify data). These characteristics help the optimizer and ensure replication safety .

**Example 2: RETURNS NULL ON NULL INPUT Optimization (SQL Server)**

```sql
CREATE FUNCTION dbo.ufnLeadingZeros_new
(
    @Value INT
)
RETURNS CHAR(8)
WITH RETURNS NULL ON NULL INPUT
AS
BEGIN
    RETURN RIGHT('00000000' + CONVERT(CHAR(8), @Value), 8);
END;
```

**Why this works:** If `@Value` is NULL, the function returns NULL without executing the body. This short-circuits the call, reducing execution count and CPU time .

### Real-World Cases

**Case 1: Optional Parameter Defaults**

A date formatting function uses `@FormatString VARCHAR(20) = 'YYYY-MM-DD'`. Callers can omit the format to use the default.

**Case 2: Deterministic Functions for Indexed Views**

A deterministic function is required for use in indexed views. Marking it `DETERMINISTIC` enables this .

### References

- Create User-defined Functions (Database Engine) - https://learn.microsoft.com/en-us/sql/relational-databases/user-defined-functions/create-user-defined-functions-database-engine
- Object-Relational Databases and OR Extensions - https://github.com/munners17/INFO257-Sp2019/raw/master/10_Meeting/Lecture17_257.pdf
- Improve Performance of UDFs with NULL ON NULL INPUT - https://sqlperformance.com/2018/12/sql-performance/improve-udfs-null-on-null-input

---

## 4. Function Limitations

### Definitions

**Core Definition:** Function limitations are the restrictions on what UDFs can and cannot do, including prohibitions on database modification, error handling, and transaction control.

**Technical Definition:** SQL Server UDFs cannot perform actions that modify database state (`INSERT`, `UPDATE`, `DELETE` on permanent tables), cannot return multiple result sets, cannot use `TRY...CATCH` or `RAISERROR`, cannot call stored procedures, cannot use dynamic SQL or temporary tables, and cannot contain `SET` statements (except for variable assignment) . Functions are meant to be read-only and side-effect-free .

**Beginner-Friendly Explanation:** Functions are like calculators—they can only read data and compute results. They can't change anything. If you need to modify data, use a stored procedure instead.

### Purposes

- To ensure functions are side-effect-free and safe for use in queries.
- To maintain the distinction between functions (read-only) and procedures (read-write).
- To prevent unexpected behaviour when functions are used in `SELECT` statements.
- To ensure deterministic and predictable results.

### Syntax Rules and Structure

**Prohibited Operations in SQL Server UDFs:**

| Prohibited | Error |
|-----------|-------|
| `INSERT`, `UPDATE`, `DELETE` on permanent tables | "Invalid use of side-effecting operator"  |
| `TRY...CATCH` | Not supported  |
| `RAISERROR` / `THROW` | Not supported  |
| Dynamic SQL (`EXEC`, `sp_executesql`) | Not supported  |
| Temporary tables | Not allowed (table variables allowed)  |
| `SET` statements (e.g., `SET NOCOUNT ON`) | Not allowed  |
| `NEWID()`, `RAND()`, `NEWSEQUENTIALID()` | Side-effecting; not allowed  |
| Calling stored procedures | Not allowed  |
| `FOR XML` clause | Not allowed  |
| `OUTPUT INTO` clause | Not allowed  |

**Component Breakdown:**

| Restriction | Rationale |
|-------------|-----------|
| No DML | Functions must be side-effect-free  |
| No error handling | Functions are expected to succeed or fail as expressions |
| No stored procedure calls | Procedures may modify state |
| No dynamic SQL | Prevents unpredictable behaviour |
| No temp tables | Temp tables imply state modification |

**Syntax Rules:**

- Functions can call other functions (nesting up to 32 levels) .
- Table variables (`DECLARE @t TABLE`) are allowed in MSTVFs .
- `SET` is allowed for variable assignment (e.g., `SET @x = @y + 1`) but not for session settings .
- To work around side-effecting function prohibition, wrap the function in a view and call the view from within the UDF .

**Constraints and Limitations:**

- These limitations are enforced at `CREATE FUNCTION` time for some (DML), and at runtime for others (side-effecting functions) .
- MySQL functions can have `MODIFIES SQL DATA` characteristic but still cannot perform DML in practice .
- PostgreSQL functions (in PL/pgSQL) can modify data if declared `VOLATILE`, but this is a different model from SQL Server UDFs.

### Annotated Complete Code Examples

**Example 1: Prohibited DML in UDF**

```sql
-- This will FAIL
CREATE FUNCTION dbo.TryUpdate ()
RETURNS INT
AS
BEGIN
    UPDATE accounts SET balance = 0 WHERE account_id = 1;
    RETURN 1;
END;
-- Error: Invalid use of side-effecting or time-dependent operator in 'UPDATE' within a function.
```

**Why this fails:** UDFs cannot modify database state. The `UPDATE` statement is prohibited .

**Example 2: Workaround for Side-Effecting Functions**

```sql
-- Create a view that wraps the side-effecting function
CREATE VIEW vw_NewID AS
SELECT NEWID() AS NewIDValue;

-- Create a UDF that calls the view (allowed)
CREATE FUNCTION dbo.GetNewID ()
RETURNS UNIQUEIDENTIFIER
AS
BEGIN
    DECLARE @id UNIQUEIDENTIFIER;
    SELECT @id = NewIDValue FROM vw_NewID;
    RETURN @id;
END;
```

**Why this works:** The side-effecting function (`NEWID()`) is wrapped in a view, and the UDF reads from the view. This bypasses the prohibition .

### Real-World Cases

**Case 1: Choosing Between Function and Procedure**

A developer needs to insert a row and return the new ID. Since functions cannot perform `INSERT`, a stored procedure is used instead.

**Case 2: Deterministic Computation**

A function computes a checksum of a string. Since it only reads and computes, it is valid as a UDF.

### References

- Create User-defined Functions (Database Engine) - https://learn.microsoft.com/en-us/sql/relational-databases/user-defined-functions/create-user-defined-functions-database-engine
- Insert/Update/Delete with function in SQL Server - https://stackoverflow.com/feeds/question/6150888

---

## 5. Performance Costs

### Definitions

**Core Definition:** Performance costs of UDFs are the runtime overheads incurred by using functions in queries, primarily due to row-by-row (RBAR) execution and lack of optimizer support.

**Technical Definition:** Scalar UDFs are executed once per row in the outer query, creating an "RBAR" (row-by-agonizing-row) pattern that prevents set-based processing, blocks parallelism, and prevents the optimizer from accurately estimating costs . Multi-statement TVFs have fixed cardinality estimates, leading to poor plans when joined . SQL Server 2019 introduced Scalar UDF Inlining, which transforms scalar UDFs into relational expressions, enabling set-based optimization and parallelism .

**Beginner-Friendly Explanation:** Using a scalar function in a query is like asking someone to calculate a value for every single row individually. If you have a million rows, the function runs a million times. Inlining (SQL Server 2019+) rewrites the function as part of the query so the database can process all rows at once.

### Purposes

- To understand why scalar UDFs often cause performance problems.
- To identify when to use inline TVFs instead of scalar UDFs.
- To leverage inlining optimizations in newer SQL Server versions.
- To use `RETURNS NULL ON NULL INPUT` to reduce unnecessary calls.

### Syntax Rules and Structure

**Performance Anti-Patterns:**

| Pattern | Impact | Solution |
|---------|--------|----------|
| Scalar UDF in `SELECT` list | RBAR execution per row | Inline the expression or use TVF  |
| Scalar UDF in `WHERE` clause | RBAR execution, prevents index use | Rewrite as join or inline expression |
| Scalar UDF in `UPDATE` | Additional spools, poor plan | Inline expression  |
| MSTVF in `JOIN` | Fixed cardinality estimate, poor plan | Use inline TVF if possible  |
| UDF preventing parallelism | Serial plan | Scalar UDF Inlining (SQL Server 2019+)  |

**Inlining Optimization (SQL Server 2019+):**

```sql
-- Scalar UDF Inlining automatically transforms this:
SELECT dbo.CalculateTax(Price, 0.08) FROM Products;

-- Into something like:
SELECT Price * 0.08 FROM Products;
```

**Component Breakdown:**

| Optimization | Effect |
|-------------|--------|
| Scalar UDF Inlining | Converts UDF to relational expression; enables set-based processing  |
| `RETURNS NULL ON NULL INPUT` | Skips function execution for NULL inputs  |
| Interleaved Execution (MSTVF) | Captures actual row count before final optimization  |
| Native Compilation (SQL Server 2016+) | Compiles UDF to native code for faster execution  |

**Syntax Rules:**

- Scalar UDF Inlining requires compatibility level 150 (SQL Server 2019+) .
- Inlining is automatic for eligible scalar UDFs; no syntax change required.
- `RETURNS NULL ON NULL INPUT` is declared in the function header .
- Inline TVFs are always preferred for performance over MSTVFs .

**Constraints and Limitations:**

- Scalar UDF Inlining has eligibility requirements (no certain constructs like `BEGIN...END` with multiple statements, no `EXEC`, etc.) .
- Not all UDFs can be inlined; complex logic may still cause RBAR.
- Inline TVFs still require the optimizer to treat them as parameterized views; complex TVFs may not merge perfectly .

### Annotated Complete Code Examples

**Example 1: Performance Comparison — Scalar UDF vs. TVF**

```sql
-- Scalar UDF (slow: RBAR)
CREATE FUNCTION dbo.FarePerMile (@Fare MONEY, @Miles INT)
RETURNS MONEY
WITH SCHEMABINDING
AS
BEGIN
    DECLARE @retVal MONEY = (@Fare / @Miles);
    RETURN @retVal;
END;
GO

-- Query using scalar UDF (serial plan, executed per row)
SELECT FlightID, dbo.FarePerMile(Fare, Miles) AS FarePerMile
FROM Flights;
-- Result: 13,986 ms elapsed time 

-- Inline TVF (fast: set-based)
CREATE FUNCTION dbo.FarePerMile_TVF (@Fare MONEY, @Miles INT)
RETURNS TABLE
AS
RETURN
(
    SELECT (@Fare / @Miles) AS RetVal
);
GO

-- Query using TVF (parallel plan, set-based)
SELECT f.FlightID, t.RetVal AS FarePerMile
FROM Flights f
CROSS APPLY dbo.FarePerMile_TVF(f.Fare, f.Miles) t;
-- Result: 3,742 ms elapsed time (native compiled) 
```

**Why the difference:** The scalar UDF is executed once per row, forcing a serial plan. The TVF allows the optimizer to treat the computation as part of the query, enabling parallelism and set-based processing. The native compiled version further reduces overhead .

**Example 2: RETURNS NULL ON NULL INPUT Optimization**

```sql
-- Without optimization: function runs for every row
CREATE FUNCTION dbo.ufnLeadingZeros (@Value INT)
RETURNS CHAR(8)
AS
BEGIN
    RETURN RIGHT('00000000' + CONVERT(CHAR(8), @Value), 8);
END;
-- Execution: 31,465 calls, 204ms CPU 

-- With optimization: function skipped for NULL inputs
CREATE FUNCTION dbo.ufnLeadingZeros_new (@Value INT)
RETURNS CHAR(8)
WITH RETURNS NULL ON NULL INPUT
AS
BEGIN
    RETURN RIGHT('00000000' + CONVERT(CHAR(8), @Value), 8);
END;
-- Execution: 13,976 calls (skipped NULLs), 78ms CPU 
```

**Why this works:** `RETURNS NULL ON NULL INPUT` short-circuits the function when `@Value` is NULL, avoiding the function call overhead for those rows .

### Real-World Cases

**Case 1: Report Query Optimization**

A report uses a scalar UDF for currency conversion. The query takes 30 seconds. The developer rewrites it as a join to a conversion rate table, reducing runtime to 2 seconds.

**Case 2: SQL Server 2019 Inlining**

An application uses a scalar UDF for pricing logic. After upgrading to SQL Server 2019 with compatibility level 150, the UDF is automatically inlined, and query performance improves by 2x without code changes .

### References

- UDF and Performance - https://learn.microsoft.com/en-us/archive/blogs/vipulshah/udf-and-performance
- Soften the RBAR impact with Native Compiled UDFs in SQL Server 2016 - https://learn.microsoft.com/en-us/archive/blogs/sqlcat/soften-the-rbar-impact-with-native-compiled-udfs-in-sql-server-2016
- What is Inlining a fuction? - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/69757151/what-is-inlining-a-fuction
- Improve Performance of UDFs with NULL ON NULL INPUT - https://sqlperformance.com/2018/12/sql-performance/improve-udfs-null-on-null-input
- Create Table-Valued Functions - Training - https://learn.microsoft.com/en-us/training/modules/implement-programmability-objects/5-create-table-valued-functions
- Create user-defined functions - Training - https://learn.microsoft.com/en-gb/training/modules/create-stored-procedures-table-valued-functions/6-create-inline

---

## Summary Table of UDF Types

| Feature | Scalar Function | Inline TVF | Multi-Statement TVF |
|---------|----------------|------------|---------------------|
| Return Type | Single value | Table (inferred) | Table (declared) |
| Syntax | `RETURNS type` | `RETURNS TABLE` | `RETURNS @table TABLE (...)` |
| Body | `BEGIN...END` | Single `RETURN (SELECT)` | `BEGIN...END` with `INSERT INTO @table` |
| Performance | RBAR, serial | Optimal, parallel | Fixed estimate, serial |
| Optimizer Support | Poor (inlined in 2019+) | Excellent (parameterized view) | Poor (Interleaved Execution in 2017+) |
| Parallelism | Blocked | Allowed | Blocked |
| Best Use | Procedural code, simple calculations | Parameterized queries in FROM | Complex multi-step logic |
| DML Allowed | No | No | No |
| Error Handling | No `TRY...CATCH` | No `TRY...CATCH` | No `TRY...CATCH` |
| Recommended? | Sparingly | Yes, preferred | Only when necessary |

---

## References

- Create User-defined Functions (Database Engine) - https://learn.microsoft.com/en-us/sql/relational-databases/user-defined-functions/create-user-defined-functions-database-engine
- UDF and Performance - https://learn.microsoft.com/en-us/archive/blogs/vipulshah/udf-and-performance
- Create Table-Valued Functions - Training - https://learn.microsoft.com/en-us/training/modules/implement-programmability-objects/5-create-table-valued-functions
- Create user-defined functions - Training - https://learn.microsoft.com/en-gb/training/modules/create-stored-procedures-table-valued-functions/6-create-inline
- Soften the RBAR impact with Native Compiled UDFs in SQL Server 2016 - https://learn.microsoft.com/en-us/archive/blogs/sqlcat/soften-the-rbar-impact-with-native-compiled-udfs-in-sql-server-2016
- Improve Performance of UDFs with NULL ON NULL INPUT - https://sqlperformance.com/2018/12/sql-performance/improve-udfs-null-on-null-input
- What is Inlining a fuction? - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/69757151/what-is-inlining-a-fuction
- SQL Functions - http://users.cms.caltech.edu/~donnie/dbcourse/intro0607/lectures/Lecture10.pdf
- Object-Relational Databases and OR Extensions - https://github.com/munners17/INFO257-Sp2019/raw/master/10_Meeting/Lecture17_257.pdf
- Insert/Update/Delete with function in SQL Server - https://stackoverflow.com/feeds/question/6150888
- Revision 46cad0ca-08fc-4f9b-82c2-4a4b74ed2553 - https://stackoverflow.com/revisions/46cad0ca-08fc-4f9b-82c2-4a4b74ed2553/view-source