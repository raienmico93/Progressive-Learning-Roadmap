# SQL Procedural Extensions & Language Architecture: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL procedural extensions are programming language constructs added to standard SQL that enable variables, control flow, cursors, exception handling, and other procedural logic to be embedded within database modules (procedures, functions, triggers).

**Technical Definition:** Every major RDBMS extends standard SQL with a proprietary procedural language: T-SQL (Microsoft SQL Server), PL/SQL (Oracle), PL/pgSQL (PostgreSQL), and MySQL Stored Programs. These languages share a common architecture: compound statements (`BEGIN...END`), variable declaration and assignment, conditional logic (`IF`, `CASE`), loops (`WHILE`, `LOOP`, `REPEAT`, `FOR`), cursors for row-by-row processing, and transaction control. SQL:1999 and later standards define a persistent stored module (PSM) facility that these dialects implement with variations .

**Beginner-Friendly Explanation:** Standard SQL is declarative—you say what you want, not how to get it. Procedural extensions add "how" to the mix. They let you write variables, if-then-else logic, loops, and error handling directly inside the database. It's like adding a programming language to SQL.

### Key Characteristics

- **Dialect-specific syntax:** Each RDBMS implements its own procedural language with unique syntax and capabilities .
- **Compound statements:** `BEGIN...END` blocks group statements and define variable scope.
- **Two-phase execution:** The database parses and compiles procedural modules into an intermediate representation before execution.
- **Set-based vs. row-based:** Procedural code often processes rows one at a time (RBAR), which can be dramatically slower than set-based SQL .
- **Transaction awareness:** Procedural modules can contain `BEGIN TRAN`, `COMMIT`, `ROLLBACK`, and savepoints, with system variables tracking transaction state.
- **Recursion support:** Recursive CTEs (`WITH RECURSIVE`) provide set-based recursion; procedural recursion is limited by dialect-specific nesting limits .

### Prerequisites

- **Basic SQL knowledge:** `SELECT`, `INSERT`, `UPDATE`, `DELETE`, and joins.
- **Programming fundamentals:** Variables, conditionals, loops, and functions from any language.
- **Transaction concepts:** `BEGIN`, `COMMIT`, `ROLLBACK`, and ACID properties.
- **Dialect awareness:** Knowledge of the specific RDBMS's procedural language (T-SQL, PL/SQL, PL/pgSQL, MySQL).

### Related Programming Areas

- **Application Development:** Encapsulating business logic in the database layer.
- **Data Engineering:** ETL transformations and batch processing.
- **Database Administration:** Maintenance scripts, monitoring, and automation.
- **Performance Engineering:** Understanding when procedural code helps vs. hurts performance.

### Core Concepts / Features

1. Variables & Scope
2. Cursors
3. Cursor Alternatives: Set-Based vs. Cursor-Based Processing
4. Recursive Logic
5. Transaction Management in Programmability

---

## 1. Variables & Scope

### Definitions

**Core Definition:** Variables in SQL procedural extensions are named storage locations that hold a single data value (scalar) or a set of rows (table variables), with scope determining where they can be referenced.

**Technical Definition:** Scalar variables are declared with `DECLARE @name type` (T-SQL), `DECLARE name type;` (PL/pgSQL, MySQL), or in the `DECLARE` section of PL/SQL blocks. Assignment uses `SET @var = value` or `SELECT @var = column FROM table` (T-SQL), `:=` or `SELECT INTO` (PL/pgSQL, PL/SQL), or `SET var = value` (MySQL). Table variables (`DECLARE @t TABLE (...)` in T-SQL) hold result sets in memory. Scope is defined by the enclosing `BEGIN...END` block; variables are visible from their declaration point to the end of the block. MySQL has a critical scoping rule: column names in SQL statements are resolved to local variables if a variable with the same name exists, requiring careful naming to avoid ambiguity .

**Beginner-Friendly Explanation:** A variable is a named box that holds a value. You declare it (create the box), assign it (put a value in), and use it in your SQL. Its scope is the block of code where it lives. In MySQL, be careful: if you have a variable named `xname` and a column named `xname`, the variable wins—which can surprise you.

### Purposes

- To store intermediate values during procedural execution.
- To hold parameter values for use in queries.
- To accumulate results across loop iterations.
- To provide named, typed storage for calculation results.
- To hold entire result sets in memory for further processing (table variables).

### Syntax Rules and Structure

**Complete General Syntax (T-SQL):**

```sql
DECLARE @var_name data_type [ = initial_value ];
SET @var_name = expression;
SELECT @var_name = column FROM table WHERE condition;
```

**Complete General Syntax (PL/pgSQL):**

```sql
DECLARE
    var_name data_type [ := initial_value ];
BEGIN
    var_name := expression;
    SELECT column INTO var_name FROM table WHERE condition;
END;
```

**Complete General Syntax (MySQL):**

```sql
DECLARE var_name data_type [ DEFAULT initial_value ];
SET var_name = expression;
SELECT column INTO var_name FROM table WHERE condition;
```

**Complete General Syntax (PL/SQL):**

```sql
DECLARE
    var_name data_type [ := initial_value ];
BEGIN
    var_name := expression;
    SELECT column INTO var_name FROM table WHERE condition;
END;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `DECLARE` | Introduces variable declaration |
| `var_name` / `@var_name` | The variable identifier |
| `data_type` | The variable's data type |
| `initial_value` | Optional default value |
| `SET` / `:=` | Assignment operator |
| `SELECT ... INTO` | Assignment from a query result |

**Syntax Rules:**

- T-SQL variables are prefixed with `@` (e.g., `@counter`).
- PL/pgSQL and PL/SQL use `:=` for assignment in procedural code; `SELECT INTO` for query-based assignment.
- MySQL uses `SET` for assignment and `SELECT ... INTO` for query-based assignment .
- Variables must be declared before use; declaration order matters for cursor and handler dependencies in MySQL .
- Table variables (T-SQL) are declared with `DECLARE @t TABLE (columns)` and can hold result sets.

**Constraints and Limitations:**

- MySQL variable names shadow column names in SQL statements, potentially causing unexpected results .
- Variable scope is limited to the enclosing `BEGIN...END` block.
- Table variables in T-SQL do not have statistics, which can lead to poor cardinality estimates.
- PL/pgSQL requires `DECLARE` before `BEGIN` (unlike T-SQL where `DECLARE` is inline).

### Annotated Complete Code Examples

**Example 1: Variable Declaration and Assignment (T-SQL)**

```sql
DECLARE @counter INT = 0;
DECLARE @employeeName NVARCHAR(50);

-- Assign via SELECT
SELECT @employeeName = lastname FROM HR.Employees WHERE empid = 1;

-- Assign via SET
SET @counter = @counter + 1;

-- Use in a query
SELECT @employeeName AS EmployeeName, @counter AS CounterValue;
```

**Why this works:** The variables are declared with types and optional initial values. `SELECT @var = column` assigns from a query, while `SET @var = expression` assigns a computed value. Both are valid assignment methods .

**Example 2: Variable Scope and Shadowing (MySQL)**

```sql
CREATE PROCEDURE sp1 (x VARCHAR(5))
BEGIN
  DECLARE xname VARCHAR(5) DEFAULT 'bob';
  DECLARE newname VARCHAR(5);
  DECLARE xid INT;

  SELECT xname, id INTO newname, xid
    FROM table1 WHERE xname = xname;
  SELECT newname;
END;
```

**Why this output occurs:** MySQL interprets `xname` in the `WHERE xname = xname` as a reference to the local variable `xname` (value `'bob'`), not the column `table1.xname`. The procedure returns `'bob'` regardless of the table's contents .

### Real-World Cases

**Case 1: Accumulator in Loop**

A procedure loops through rows and accumulates a total in a variable. The variable persists across iterations, holding the running sum.

**Case 2: Parameter Processing**

A procedure receives a date parameter, assigns it to a local variable for validation, then uses it in multiple queries.

### References

- MySQL Stored Program Syntax - DECLARE - https://docs.oracle.com/cd/E19078-01/mysql/mysql-refman-5.1/sql-syntax.html#42
- Microsoft SQL Server Cursors and Variables - https://www.ms.mff.cuni.cz/~kopecky/teaching/dbs/lecture05.pdf#1#1
- MySQL Reference Manual - Cursors - https://downloads.mysql.com/docs/refman-5.5-en.pdf#616#262

---

## 2. Cursors

### Definitions

**Core Definition:** A cursor is a database object that allows procedural code to retrieve and process rows from a result set one at a time, providing a window into the data.

**Technical Definition:** Cursors follow a five-step lifecycle: `DECLARE` (define the query), `OPEN` (execute the query and position before the first row), `FETCH` (retrieve the next row into variables), `CLOSE` (release the result set), and `DEALLOCATE` (remove the cursor definition). T-SQL cursors support scrolling options (`SCROLL`, `NEXT`, `PRIOR`, `ABSOLUTE n`, `RELATIVE n`) . MySQL cursors are read-only, non-scrollable, non-holdable, and insensitive; they are materialized into an internal temporary table . PL/pgSQL and PL/SQL support cursor `FOR` loops that automatically open, fetch, and close.

**Beginner-Friendly Explanation:** A cursor is like a bookmark in a book. You declare which book (query) you want to read, open it to the first page, read each page one at a time, and close it when done. It lets you handle rows individually—useful when set-based operations can't do the job.

### Purposes

- To process rows one at a time when set-based logic is impossible or impractical.
- To perform row-specific actions based on column values.
- To call stored procedures or complex logic for each row.
- To handle data transformations that require per-row decision-making.
- To bridge the gap between set-oriented SQL and procedural row-oriented logic.

### Syntax Rules and Structure

**Complete General Syntax (T-SQL):**

```sql
DECLARE cursor_name [SCROLL] CURSOR FOR
    SELECT columns FROM table WHERE condition;

OPEN cursor_name;

FETCH NEXT FROM cursor_name INTO @var1, @var2;

WHILE @@FETCH_STATUS = 0
BEGIN
    -- Process row
    FETCH NEXT FROM cursor_name INTO @var1, @var2;
END;

CLOSE cursor_name;
DEALLOCATE cursor_name;
```

**Complete General Syntax (MySQL):**

```sql
DECLARE cursor_name CURSOR FOR SELECT columns FROM table;
DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

OPEN cursor_name;

read_loop: LOOP
    FETCH cursor_name INTO var1, var2;
    IF done THEN
        LEAVE read_loop;
    END IF;
    -- Process row
END LOOP;

CLOSE cursor_name;
```

**Complete General Syntax (PL/pgSQL):**

```sql
FOR record_var IN SELECT columns FROM table LOOP
    -- Process row
END LOOP;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `DECLARE ... CURSOR FOR` | Defines the cursor's query |
| `OPEN` | Executes the query and prepares for fetching |
| `FETCH ... INTO` | Retrieves the next row into variables |
| `@@FETCH_STATUS` / `NOT FOUND` | Indicates whether more rows exist |
| `CLOSE` | Releases the result set |
| `DEALLOCATE` | Removes the cursor definition (T-SQL) |

**Syntax Rules:**

- T-SQL: `DECLARE` defines the cursor; `SCROLL` enables backward fetching. Only `NEXT` is allowed without `SCROLL` .
- MySQL: Cursors must be declared after variables and conditions but before handlers. They are read-only and non-scrollable .
- PL/pgSQL: `FOR ... IN SELECT ... LOOP` automatically handles open/fetch/close, simplifying cursor usage.
- MySQL: `DECLARE CONTINUE HANDLER FOR NOT FOUND` is required to detect end-of-result-set .
- Cursors must be closed and deallocated to release resources.

**Constraints and Limitations:**

- MySQL cursors cannot be used to update rows (`UPDATE ... WHERE CURRENT OF` not supported) .
- MySQL cursors are materialized into temporary tables; large result sets can be slow .
- T-SQL cursors are notorious performance killers due to row-by-row (RBAR) processing .
- Cursors hold locks and resources for their duration.

### Annotated Complete Code Examples

**Example 1: T-SQL Cursor for Tax Payment**

```sql
DECLARE @acc VARCHAR(25), @bal INT;
DECLARE Cur CURSOR FOR
    SELECT account, balance FROM Accounts;

OPEN Cur;
FETCH NEXT FROM Cur INTO @acc, @bal;

WHILE @@FETCH_STATUS = 0
BEGIN
    -- Process each account: pay 1% tax
    EXEC Payment @acc, '21-87526287/0300', @bal * 0.01;
    FETCH NEXT FROM Cur INTO @acc, @bal;
END;

CLOSE Cur;
DEALLOCATE Cur;
```

**Why this works:** The cursor iterates over all accounts. For each row, the `Payment` procedure is called with 1% of the balance. The loop continues while `@@FETCH_STATUS = 0` (successful fetch) .

**Example 2: MySQL Cursor with Handler**

```sql
CREATE PROCEDURE curdemo()
BEGIN
  DECLARE done INT DEFAULT 0;
  DECLARE a CHAR(16);
  DECLARE b, c INT;
  DECLARE cur1 CURSOR FOR SELECT id, data FROM test.t1;
  DECLARE cur2 CURSOR FOR SELECT i FROM test.t2;
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

  OPEN cur1;
  OPEN cur2;

  read_loop: LOOP
    FETCH cur1 INTO a, b;
    FETCH cur2 INTO c;
    IF done THEN
      LEAVE read_loop;
    END IF;
    -- Process a, b, c
  END LOOP;

  CLOSE cur1;
  CLOSE cur2;
END;
```

**Why this works:** The `CONTINUE HANDLER FOR NOT FOUND` sets `done = 1` when a cursor exhausts its rows. The loop exits when `done` is true. Both cursors are processed in parallel .

### Real-World Cases

**Case 1: Row-by-Row Data Migration**

A migration script uses a cursor to process each record, applying complex transformations that require procedural logic (e.g., calling external APIs, conditional updates).

**Case 2: Hierarchical Processing**

A cursor processes hierarchical data (e.g., organizational chart) where each row's processing depends on its parent's result.

### References

- Microsoft SQL Server Cursors - https://www.ms.mff.cuni.cz/~kopecky/teaching/dbs/lecture05.pdf#1#1
- MySQL Stored Programs - Cursors - https://downloads.mysql.com/docs/refman-5.5-en.pdf#616#262
- Oracle PL/SQL Cursors - https://books.google.com.sg/books?id=aL1aQHrtYnsC

---

## 3. Cursor Alternatives: Set-Based vs. Cursor-Based Processing

### Definitions

**Core Definition:** Set-based processing operates on entire result sets with a single SQL statement, while cursor-based processing (RBAR—row by agonizing row) processes one row at a time through procedural loops.

**Technical Definition:** SQL was conceived as a set-oriented language; RDBMS engines are extremely efficient when working in set-oriented mode . Cursors, while ANSI-standard and sometimes necessary, impose row-by-row processing that prevents the optimizer from using parallel plans, index merges, and other set-level optimizations. A real-world example: a nested cursor processing 204,000 outer rows with 10 million inner rows had completed only 107 outer loops (0.5%) after 30 hours—the set-based rewrite completed in 17 hours (plus 7 hours for index maintenance), a fraction of the estimated 8+ months for the cursor .

**Beginner-Friendly Explanation:** Set-based processing is like telling a librarian "stamp every book on this shelf." Cursor-based processing is like picking up each book one by one, stamping it, and putting it back. For a shelf of 10,000 books, the first approach is vastly faster.

### Purposes

- To understand when cursors are appropriate vs. when set-based logic should be used.
- To recognize performance anti-patterns in procedural code.
- To convert cursor logic to set-based operations for dramatic speedups.
- To make informed decisions about RBAR processing.

### Syntax Rules and Structure

**Set-Based Pattern:**

```sql
-- Single statement processes all rows
UPDATE titles SET contract = 1 WHERE type = 'business';
```

**Cursor-Based Pattern:**

```sql
-- Loop processes rows one at a time
DECLARE cur CURSOR FOR SELECT id FROM titles WHERE type = 'business';
OPEN cur;
FETCH NEXT FROM cur INTO @id;
WHILE @@FETCH_STATUS = 0
BEGIN
    UPDATE titles SET contract = 1 WHERE id = @id;
    FETCH NEXT FROM cur INTO @id;
END;
CLOSE cur;
DEALLOCATE cur;
```

**Component Breakdown:**

| Approach | Statement Count | Optimizer Freedom | Parallelism |
|----------|----------------|-------------------|-------------|
| Set-Based | 1 | Full | Allowed |
| Cursor-Based | N (rows) | Limited | Blocked |

**Syntax Rules:**

- Set-based operations should be the default; cursors are the exception .
- Cursors are appropriate when: row-by-row processing cannot be expressed in set-based SQL, when calling external procedures per row, or when the order of processing matters .
- The "Process Rows" step in a cursor loop is where flexibility lies—but also where performance is lost .
- Set-based rewrites often involve joining the outer and inner tables and using `INSERT INTO ... SELECT` or `UPDATE ... FROM` .

**Constraints and Limitations:**

- Set-based operations cannot handle all procedural logic (e.g., calling external APIs per row).
- Cursors are required by ANSI SQL standards and have legitimate uses .
- Rewriting cursor logic to set-based can require significant refactoring and careful testing.

### Annotated Complete Code Examples

**Example 1: Cursor vs. Set-Based Comparison**

```sql
-- CURSOR APPROACH (slow: RBAR)
DECLARE @EntityId VARCHAR(16), @BaseId VARCHAR(16), @PerfId VARCHAR(16);
DECLARE outerCursor CURSOR FOR SELECT EntityId, BaseId FROM outerTable;
OPEN outerCursor;
FETCH NEXT FROM outerCursor INTO @EntityId, @BaseId;
WHILE @@FETCH_STATUS = 0
BEGIN
    DECLARE innerCursor CURSOR FOR SELECT PRFMR_ID FROM innerTable WHERE ENTY_ID = @BaseId;
    OPEN innerCursor;
    FETCH NEXT FROM innerCursor INTO @PerfId;
    WHILE @@FETCH_STATUS = 0
    BEGIN
        UPDATE 200MilRowTable SET ENTY_ID = @EntityId WHERE PRFMR_ID = @PerfId;
        FETCH NEXT FROM innerCursor INTO @PerfId;
    END;
    CLOSE innerCursor;
    DEALLOCATE innerCursor;
    FETCH NEXT FROM outerCursor INTO @EntityId, @BaseId;
END;
CLOSE outerCursor;
DEALLOCATE outerCursor;
-- Result: 0.5% progress after 30 hours 

-- SET-BASED APPROACH (fast: single pass)
SELECT i.PRFMR_ID, o.EntityId INTO #tempTable
FROM innerTable i JOIN outerTable o ON i.ENTY_ID = o.BaseId;

UPDATE m SET m.ENTY_ID = t.EntityId
FROM 200MilRowTable m JOIN #tempTable t ON m.PRFMR_ID = t.PRFMR_ID;
-- Result: ~17 hours for 100% completion 
```

**Why the difference:** The cursor approach performs one `UPDATE` per matching row (potentially millions of individual updates). The set-based approach joins the tables once, materializes the mapping into a temp table, then performs a single set-based `UPDATE`. The optimizer can parallelize and use hash joins .

### Real-World Cases

**Case 1: Data Warehouse Load**

An ETL process uses set-based `INSERT INTO ... SELECT` instead of a cursor to load millions of rows, completing in minutes instead of hours.

**Case 2: Batch Updates**

A batch job updates all rows matching a condition using a single `UPDATE` statement instead of a cursor loop, avoiding RBAR overhead.

### References

- SAP ASE - Set-Oriented Versus Row-Oriented Programming - https://help.sap.com/docs/SAP_ASE/a7b3e46335184f5caf70a08c91c540f3/a9737d1dbc2b101497c4d4db393d2447.html
- Microsoft - Increase SQL Server performance by replacing cursors with set operations - https://learn.microsoft.com/sl-si/archive/blogs/mssqlisv/increase-your-sql-server-performance-by-replacing-cursors-with-set-operations

---

## 4. Recursive Logic

### Definitions

**Core Definition:** Recursive logic in SQL refers to the ability to process hierarchical or self-referencing data, either through recursive Common Table Expressions (CTEs) or through procedural recursion in stored modules.

**Technical Definition:** A recursive CTE (`WITH RECURSIVE`) consists of a non-recursive anchor member and a recursive member separated by `UNION ALL` or `UNION`. The anchor produces the initial row set; the recursive member references the CTE itself and produces additional rows until no new rows are generated . PostgreSQL and SQLite also support `SEARCH` and `CYCLE` clauses for depth-first/breadth-first control and cycle detection . Procedural recursion (functions calling themselves) is supported in PL/pgSQL, PL/SQL, and T-SQL but has dialect-specific nesting limits (e.g., 32 levels in T-SQL).

**Beginner-Friendly Explanation:** Recursive logic is for data that references itself—like an organizational chart where each employee has a manager who is also an employee. A recursive CTE starts with a base case (the CEO), then repeatedly finds the people who report to the previous level, building the hierarchy step by step.

### Purposes

- To traverse hierarchical data (organizational charts, bill-of-materials, category trees).
- To compute transitive closures (all ancestors, all descendants).
- To generate sequences or series without a numbers table.
- To solve graph problems (shortest path, reachability).
- To process tree-structured data in a single query rather than procedural loops.

### Syntax Rules and Structure

**Complete General Syntax (Recursive CTE):**

```sql
WITH RECURSIVE cte_name (columns) AS (
    -- Anchor member: non-recursive
    SELECT initial_columns FROM table WHERE condition
    UNION ALL
    -- Recursive member: references cte_name
    SELECT next_columns FROM table JOIN cte_name ON condition
)
SELECT * FROM cte_name;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `WITH RECURSIVE` | Declares a recursive CTE |
| Anchor member | Produces the initial row set (base case) |
| `UNION ALL` / `UNION` | Separates anchor from recursive member |
| Recursive member | References the CTE and produces additional rows |
| Termination | Occurs when the recursive member produces no new rows  |

**Syntax Rules:**

- `WITH RECURSIVE` is required if any CTE refers to itself (MySQL requires it; PostgreSQL allows `RECURSIVE` optionally) .
- The anchor member must not reference the CTE.
- The recursive member must reference the CTE exactly once in its `FROM` clause.
- Column types are inferred from the anchor member only; the recursive member is ignored for type determination .
- PostgreSQL supports `SEARCH DEPTH FIRST` / `SEARCH BREADTH FIRST` for ordering and `CYCLE` for cycle detection .
- SQLite supports `ORDER BY` in the recursive member to control depth-first vs. breadth-first search .

**Constraints and Limitations:**

- Without cycle detection, recursive CTEs can loop infinitely if the data contains cycles .
- PostgreSQL's `LIMIT` trick (evaluating only as many rows as needed) works but is not portable .
- Procedural recursion has nesting limits (32 levels in T-SQL).
- Recursive CTEs cannot use aggregates in the recursive member (though they can in the outer query).

### Annotated Complete Code Examples

**Example 1: Recursive CTE for Hierarchy (MySQL)**

```sql
WITH RECURSIVE employee_hierarchy AS (
    -- Anchor: start with the CEO (no manager)
    SELECT emp_id, emp_name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive: find employees reporting to the previous level
    SELECT e.emp_id, e.emp_name, e.manager_id, eh.level + 1
    FROM employees e
    INNER JOIN employee_hierarchy eh ON e.manager_id = eh.emp_id
)
SELECT * FROM employee_hierarchy ORDER BY level, emp_name;
```

**Why this output occurs:** The anchor selects the CEO. The recursive member joins `employees` to the CTE, finding all employees whose manager is in the previous level. Recursion continues until no new rows match. The `level` column tracks depth .

**Example 2: Cycle Detection (PostgreSQL)**

```sql
WITH RECURSIVE search_graph(id, link, data, depth, is_cycle, path) AS (
    SELECT g.id, g.link, g.data, 0, false, ARRAY[g.id]
    FROM graph g
  UNION ALL
    SELECT g.id, g.link, g.data, sg.depth + 1,
           g.id = ANY(path),
           path || g.id
    FROM graph g, search_graph sg
    WHERE g.id = sg.link AND NOT is_cycle
)
SELECT * FROM search_graph;
```

**Why this works:** The `is_cycle` column checks if the current `id` is already in the `path` array. If so, recursion stops for that branch. This prevents infinite loops in cyclic graphs .

### Real-World Cases

**Case 1: Bill of Materials**

A manufacturing system uses a recursive CTE to explode a product's bill of materials, finding all components and sub-components.

**Case 2: Category Tree**

An e-commerce platform uses a recursive CTE to display a category hierarchy, computing the depth of each category for navigation.

### References

- MySQL 8.4 Reference Manual - WITH (Common Table Expressions) - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/with.html#1
- PostgreSQL Documentation - WITH Queries (Common Table Expressions) - https://www.postgresql.org/docs/16/queries-with.html
- SQLite Documentation - The WITH Clause - https://wasm-testing.sqlite.org/draft/lang_with.html

---

## 5. Transaction Management in Programmability

### Definitions

**Core Definition:** Transaction management in procedural modules involves explicitly controlling `BEGIN TRAN`, `COMMIT`, and `ROLLBACK` statements, and using system variables like `@@TRANCOUNT` to track transaction state.

**Technical Definition:** When a procedure or function executes DML statements, the transaction may be implicit (autocommit) or explicit (started with `BEGIN TRAN`). Nested transactions are simulated in T-SQL through `@@TRANCOUNT`: each `BEGIN TRAN` increments it, `COMMIT` decrements it, and `ROLLBACK` (without savepoint) resets it to 0 . Procedures can include `TRY...CATCH` blocks for error handling with rollback. PL/SQL and PL/pgSQL support savepoints for partial rollback. MySQL procedures can use `START TRANSACTION`, `COMMIT`, `ROLLBACK`, and `SAVEPOINT`.

**Beginner-Friendly Explanation:** A transaction is a safety wrapper around database changes. In procedural code, you can start a transaction, do several operations, and then either commit (save) or rollback (undo) everything. `@@TRANCOUNT` tells you how many transactions are currently open—useful for avoiding errors when rolling back.

### Purposes

- To ensure atomicity when a procedure performs multiple DML operations.
- To handle errors gracefully with rollback in `TRY...CATCH` blocks.
- To use savepoints for partial rollback within a procedure.
- To track transaction nesting with `@@TRANCOUNT` and avoid "no corresponding BEGIN" errors.
- To manage implicit vs. explicit transaction boundaries.

### Syntax Rules and Structure

**Complete General Syntax (T-SQL):**

```sql
CREATE PROCEDURE dbo.ProcessOrder (@OrderID INT)
AS
BEGIN
    BEGIN TRY
        BEGIN TRANSACTION;
        -- DML operations
        UPDATE inventory SET quantity = quantity - 1 WHERE product_id = @ProductID;
        INSERT INTO order_log (order_id, action) VALUES (@OrderID, 'processed');
        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION;
        -- Error handling
        THROW;
    END CATCH;
END;
```

**`@@TRANCOUNT` Rules:**

| Statement | Effect on `@@TRANCOUNT` |
|-----------|------------------------|
| `BEGIN TRANSACTION` | +1 |
| `COMMIT TRANSACTION` | -1 |
| `ROLLBACK TRANSACTION` (no savepoint) | 0 |
| `ROLLBACK TRANSACTION savepoint_name` | Unchanged |
| `SAVE TRANSACTION savepoint_name` | Unchanged  |

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `BEGIN TRANSACTION` | Starts an explicit transaction |
| `COMMIT TRANSACTION` | Commits all changes |
| `ROLLBACK TRANSACTION` | Rolls back all changes |
| `@@TRANCOUNT` | System variable tracking transaction nesting level |
| `SAVE TRANSACTION` | Creates a savepoint for partial rollback |
| `TRY...CATCH` | Error handling block for transaction management |

**Syntax Rules:**

- T-SQL: `BEGIN TRAN` increments `@@TRANCOUNT`; `COMMIT` decrements; `ROLLBACK` (without savepoint) resets to 0 .
- In a `CATCH` block, always check `@@TRANCOUNT > 0` before `ROLLBACK` to avoid errors.
- PL/SQL: `COMMIT` and `ROLLBACK` work at the block level; savepoints use `SAVEPOINT name` and `ROLLBACK TO name`.
- MySQL: Procedures can contain `START TRANSACTION`, `COMMIT`, `ROLLBACK`, and `SAVEPOINT` .
- PostgreSQL: Procedures (called with `CALL`) can contain transaction control statements (unlike functions, which cannot).

**Constraints and Limitations:**

- Functions (in most dialects) cannot contain transaction control statements; only procedures and triggers can.
- Nested transactions in T-SQL are not true nested transactions—only the outermost `COMMIT` actually commits .
- Rolling back to a savepoint does not release locks held since the savepoint.
- MySQL cursors are non-holdable: they close after `COMMIT` or `ROLLBACK` .

### Annotated Complete Code Examples

**Example 1: TRY...CATCH with Transaction Rollback (T-SQL)**

```sql
CREATE PROCEDURE Production.uspDeleteWorkOrder (@WorkOrderID INT)
AS
BEGIN
    SET NOCOUNT ON;
    BEGIN TRY
        BEGIN TRANSACTION;
        
        DELETE FROM Production.WorkOrderRouting
        WHERE WorkOrderID = @WorkOrderID;
        
        DELETE FROM Production.WorkOrder
        WHERE WorkOrderID = @WorkOrderID;
        
        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION;
        
        DECLARE @ErrorMessage NVARCHAR(4000);
        SELECT @ErrorMessage = ERROR_MESSAGE();
        RAISERROR(@ErrorMessage, 16, 1);
    END CATCH;
END;
```

**Why this works:** The `TRY` block contains the transaction logic. If any `DELETE` fails, the `CATCH` block checks `@@TRANCOUNT` and rolls back the transaction, undoing all changes. The error is then re-raised .

**Example 2: Savepoint in T-SQL**

```sql
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;

SAVE TRANSACTION AfterDebit;

UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

-- If credit fails, rollback to savepoint (debit is preserved)
ROLLBACK TRANSACTION AfterDebit;

-- @@TRANCOUNT is still 1 (transaction remains open)
COMMIT TRANSACTION;
-- Result: debit applied, credit rolled back
```

**Why this works:** `SAVE TRANSACTION` creates a savepoint. Rolling back to it undoes only the credit, leaving the debit in place. `@@TRANCOUNT` remains unchanged .

### Real-World Cases

**Case 1: Financial Transfer**

A procedure processes a transfer as a transaction. If the credit fails, the entire transaction rolls back, preventing money from being debited without being credited.

**Case 2: Batch Processing with Savepoints**

A procedure processes a batch of records with savepoints after each record. If one record fails, it rolls back to the savepoint and continues with the next record, rather than failing the entire batch.

### References

- SQL Server 2008应用开发案例解析 - @@TRANCOUNT - /hdd/m0201/deepseek/datasets/c1521_duxiu/duxiu_pdf/a05fc64a8742dfb04eb6f58f3c5c872f_111756619.pdf#57#23
- MySQL Stored Programs - Condition Handling - https://downloads.mysql.com/docs/refman-5.5-en.pdf#616#262

---

## Summary Table of Procedural Extensions

| Feature | T-SQL (SQL Server) | PL/pgSQL (PostgreSQL) | MySQL | PL/SQL (Oracle) |
|---------|-------------------|----------------------|-------|-----------------|
| Variable Declaration | `DECLARE @var type` | `DECLARE var type;` | `DECLARE var type;` | `DECLARE var type;` |
| Assignment | `SET` / `SELECT INTO` | `:=` / `SELECT INTO` | `SET` / `SELECT INTO` | `:=` / `SELECT INTO` |
| Cursor Loop | `WHILE @@FETCH_STATUS = 0` | `FOR rec IN SELECT LOOP` | `LOOP ... LEAVE` | `FOR rec IN cursor LOOP` |
| Cursor Update | `WHERE CURRENT OF` | Not supported | Not supported | `WHERE CURRENT OF` |
| Recursive CTE | `WITH` (no RECURSIVE keyword) | `WITH RECURSIVE` | `WITH RECURSIVE` | `WITH` (no RECURSIVE) |
| Transaction Variable | `@@TRANCOUNT` | N/A | N/A | N/A |
| Savepoint | `SAVE TRANSACTION` | `SAVEPOINT` | `SAVEPOINT` | `SAVEPOINT` |
| TRY...CATCH | Yes | `EXCEPTION` block | `DECLARE HANDLER` | `EXCEPTION` block |

---

## References

- SQL:1999 Procedural Constructs - http://people.uncw.edu/narayans/courses/csc455/sixthEdition/ch5.pdf#4#2
- MySQL Stored Program Syntax - DECLARE - https://docs.oracle.com/cd/E19078-01/mysql/mysql-refman-5.1/sql-syntax.html#42
- Microsoft SQL Server Cursors and Variables - https://www.ms.mff.cuni.cz/~kopecky/teaching/dbs/lecture05.pdf#1#1
- MySQL Reference Manual - Cursors - https://downloads.mysql.com/docs/refman-5.5-en.pdf#616#262
- MySQL 8.4 Reference Manual - WITH (Common Table Expressions) - https://docs.oracle.com/cd/E17952_01/mysql-8.4-en/with.html#1
- PostgreSQL Documentation - WITH Queries (Common Table Expressions) - https://www.postgresql.org/docs/16/queries-with.html
- SQLite Documentation - The WITH Clause - https://wasm-testing.sqlite.org/draft/lang_with.html
- SAP ASE - Set-Oriented Versus Row-Oriented Programming - https://help.sap.com/docs/SAP_ASE/a7b3e46335184f5caf70a08c91c540f3/a9737d1dbc2b101497c4d4db393d2447.html
- Microsoft - Increase SQL Server performance by replacing cursors with set operations - https://learn.microsoft.com/sl-si/archive/blogs/mssqlisv/increase-your-sql-server-performance-by-replacing-cursors-with-set-operations
- SQL Server 2008应用开发案例解析 - @@TRANCOUNT - /hdd/m0201/deepseek/datasets/c1521_duxiu/duxiu_pdf/a05fc64a8742dfb04eb6f58f3c5c872f_111756619.pdf#57#23
- Oracle PL/SQL Programming Workbook - https://books.google.com.sg/books?id=aL1aQHrtYnsC
- PostgreSQL PL/pgSQL Trigger Functions - https://www.postgresql.org/files/documentation/pdf/19/postgresql-19-US.pdf#444#235
- PostgreSQL Documentation - PL/pgSQL Trigger Procedures - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.0/postgresql-9.0-A4.pdf?id=42ee56a1b08944b9803eda799dd95d25c5a9cbf7#317#143