# SQL Set-Based Thinking: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL set-based thinking is the practice of approaching data manipulation problems by defining the desired result as a set of rows and allowing the database engine to determine the most efficient way to compute that result, rather than specifying row-by-row procedural steps.

**Technical Definition:** Set-based thinking is a mental model grounded in relational algebra—the formal mathematical foundation of SQL—in which queries are expressed as compositions of set operations (selection, projection, join, union, difference, intersection) that transform one or more input relations into an output relation. Each SQL statement defines the result, and allows the database to determine the most efficient way to obtain it. In contrast, iterative algorithms use conditional logic to pull each row or group of rows from the database to the client application, process the data on the client, and then send the data back to the database.

**Beginner-Friendly Explanation:** Set-based thinking is like ordering a pizza for the whole team instead of driving to the restaurant twelve times to pick up one slice each. You describe what you want ("one large pepperoni pizza") and the kitchen figures out the best way to make it. Procedural thinking is like writing a recipe that says "take one slice, drive to the office, come back, take another slice..."—it works, but it is painfully slow. In SQL, you say "update all products where ProductType = 1" and the database handles the iteration internally, in the most efficient way it can.

### Key Characteristics

- **Declarative, not imperative:** You describe *what* result you want, not *how* to compute it step by step.
- **Result-oriented:** The SQL statement defines the result, and allows the database to determine the most efficient way to obtain it.
- **Set-at-a-time:** Operations apply to entire sets of rows in a single statement, not to individual rows in application loops.
- **Relationally grounded:** Set-based thinking is built on relational algebra—selection, projection, join, union, difference, intersection, and Cartesian product.
- **Performance-superior:** The performance on large data sets is orders of magnitude faster than iterative techniques; it is not unusual for the run time of a program to drop from several hours to several seconds.
- **Code-concise:** The length of the code is significantly shorter, as short as two or three lines of code, because SQL defines the result and not the access method.

### Prerequisites

- Basic understanding of SQL `SELECT`, `INSERT`, `UPDATE`, and `DELETE` statements.
- Familiarity with relational concepts: tables, rows, columns, primary keys, foreign keys.
- Awareness of `NULL` semantics and three-valued logic.
- Knowledge of basic relational algebra operations (selection, projection, join).
- Understanding of how a query optimizer works at a high level.

### Related Programming Areas

- Database query optimization and performance tuning.
- ETL/ELT pipeline design.
- Data warehousing and analytics engineering.
- Application data access layer design.
- Stored procedure and function development.

### Core Concepts / Features

1. **Relational Operations** (selection, projection, join, union, difference, intersection, Cartesian product)
2. **Difference Between Row-Wise and Set-Wise Logic** (procedural vs. declarative approaches)
3. **Replacing Procedural Loops with Set Operations** (cursor elimination, two-pass approaches)
4. **Query Composition** (CTEs, derived tables, subqueries, set operators)


## Core Concept 1: Relational Operations

### Definitions

**Core Definition:** Relational operations are the fundamental set-theoretic transformations—selection, projection, join, union, difference, intersection, and Cartesian product—that operate on relations (tables) and produce new relations as output.

**Technical Definition:** Relational algebra is a theory that uses algebraic structures for modeling data and defining queries on it with well-founded semantics. The main purpose of relational algebra is to define operators that transform one or more input relations to an output relation. Given that these operators accept relations as input and produce relations as output, they can be combined and used to express complex queries that transform multiple input relations into a single output relation (the query results). The five primitive operators of Codd's algebra are the selection, the projection, the Cartesian product (also called the cross product or cross join), the set union, and the set difference. Derived operators include intersection, joins (natural, equi-join, theta join, semi-join), division, and renaming.

**Beginner-Friendly Explanation:** Relational operations are the basic "moves" you can make with tables. Selection filters rows (like a `WHERE` clause). Projection picks columns (like a `SELECT` list). Join combines tables (like a `JOIN`). Union stacks tables on top of each other. Difference finds rows in one table but not another. These operations are the building blocks of every SQL query you will ever write.

### Purposes

- **To filter rows** that satisfy a condition using the selection operation (`σ`).
- **To choose specific columns** from a relation using the projection operation (`π`).
- **To combine related rows** from two or more tables using join operations (`⋈`).
- **To combine result sets** from multiple queries using set operations (UNION, INTERSECT, EXCEPT).
- **To express complex queries** as compositions of primitive relational operations.

### Syntax Rules and Structure

#### Complete General Syntax (Selection)

```sql
-- Relational algebra: σ_condition(R)
-- SQL equivalent:
SELECT * FROM table_name WHERE condition;
```

#### Complete General Syntax (Projection)

```sql
-- Relational algebra: π_column1, column2(R)
-- SQL equivalent:
SELECT column1, column2 FROM table_name;
```

#### Complete General Syntax (Join)

```sql
-- Relational algebra: R ⋈_condition S
-- SQL equivalent:
SELECT R.*, S.*
FROM table_name R
JOIN other_table S ON R.key = S.key;
```

#### Complete General Syntax (Set Operations)

```sql
query1 UNION [ALL] query2
query1 INTERSECT [ALL] query2
query1 EXCEPT [ALL] query2
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `UNION` | Appends the result of query2 to query1, eliminating duplicate rows unless `UNION ALL` is used. |
| `INTERSECT` | Returns all rows that are both in the result of query1 and in the result of query2; duplicates eliminated unless `INTERSECT ALL` is used. |
| `EXCEPT` | Returns all rows that are in the result of query1 but not in the result of query2 (set difference); duplicates eliminated unless `EXCEPT ALL` is used. |

#### Syntax Rules

- **Union compatibility:** In order to calculate the union, intersection, or difference of two queries, the two queries must be "union compatible"—they must return the same number of columns and the corresponding columns must have compatible data types.
- **Operator precedence:** Without parentheses, `UNION` and `EXCEPT` associate left-to-right, but `INTERSECT` binds more tightly than those two operators. Thus `query1 UNION query2 INTERSECT query3` means `query1 UNION (query2 INTERSECT query3)`.
- **ALL modifier:** Each set operator supports an `ALL` modifier. When the `ALL` keyword follows a set operator, this causes duplicates to be included in the result.
- **DISTINCT modifier:** All three set operators also support a `DISTINCT` keyword, which suppresses duplicates. Since this is the default behavior, it is usually not necessary to specify `DISTINCT` explicitly.

#### Constraints and Limitations

- **Column count and types:** Set operations require the same number of columns and compatible data types across all query blocks.
- **ORDER BY placement:** `ORDER BY` can only be applied to the final result of a set operation, not to individual query blocks.
- **MySQL version dependency:** MySQL has long supported `UNION`; MySQL 8.0.31 and later adds support for `INTERSECT` and `EXCEPT`.
- **Oracle `MINUS`:** Oracle uses `MINUS` instead of `EXCEPT` for set difference; this is not supported in MySQL.

### Annotated Code Examples

#### Example 1: PostgreSQL — Selection and Projection

```sql
-- Create a sample table
CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    emp_name TEXT,
    dept TEXT,
    salary NUMERIC(10,2)
);

INSERT INTO employees (emp_name, dept, salary) VALUES
('Alice', 'Engineering', 90000),
('Bob',   'Engineering', 85000),
('Carol', 'Sales',       75000),
('Dave',  'Sales',       95000);
```

```sql
-- Selection (σ): filter rows where dept = 'Engineering'
SELECT * FROM employees WHERE dept = 'Engineering';

-- Projection (π): select only emp_name and salary
SELECT emp_name, salary FROM employees;
```

**Expected Output (Selection):**

```
 emp_id | emp_name |    dept     | salary
--------+----------+-------------+--------
      1 | Alice    | Engineering | 90000
      2 | Bob      | Engineering | 85000
```

**Expected Output (Projection):**

```
 emp_name | salary
----------+--------
 Alice    | 90000
 Bob      | 85000
 Carol    | 75000
 Dave     | 95000
```

**Why This Works:** The `WHERE` clause implements the selection operation (σ), filtering rows that satisfy the condition. The `SELECT` list implements the projection operation (π), choosing which columns to return. These are the two most fundamental relational operations.

#### Example 2: PostgreSQL — Set Operations

```sql
-- Create two tables for set operation demonstration
CREATE TABLE current_orders (order_id INT, customer TEXT);
CREATE TABLE archived_orders (order_id INT, customer TEXT);

INSERT INTO current_orders VALUES (1, 'Alice'), (2, 'Bob'), (3, 'Carol');
INSERT INTO archived_orders VALUES (3, 'Carol'), (4, 'Dave'), (5, 'Eve');
```
\
**UNION: all orders from both tables, no duplicates**
```sql
SELECT order_id, customer FROM current_orders
UNION
SELECT order_id, customer FROM archived_orders
ORDER BY order_id;
```
Expected Output (UNION):
```
 order_id | customer
----------+----------
        1 | Alice
        2 | Bob
        3 | Carol
        4 | Dave
        5 | Eve
```
\
**INTERSECT: orders that appear in both tables**
```sql
SELECT order_id, customer FROM current_orders
INTERSECT
SELECT order_id, customer FROM archived_orders;
```
Expected Output (INTERSECT):
```
 order_id | customer
----------+----------
        3 | Carol
```
\
**EXCEPT: orders in current but not in archived**
```sql
SELECT order_id, customer FROM current_orders
EXCEPT
SELECT order_id, customer FROM archived_orders;
```
Expected Output (EXCEPT):

```
 order_id | customer
----------+----------
        1 | Alice
        2 | Bob
```

**Why This Works:** `UNION` combines all rows from both queries, eliminating duplicates. `INTERSECT` returns only rows present in both result sets (Carol's order 3). `EXCEPT` returns rows in the first result set that are not in the second (Alice's and Bob's orders).

### Real-World Cases

- **Data reconciliation:** Using `EXCEPT` to find records in one system that are missing from another.
- **Deduplication:** Using `UNION` to combine data from multiple sources and eliminate duplicates.
- **Common customer analysis:** Using `INTERSECT` to find customers who appear in multiple product categories.
- **Reporting:** Using selection and projection to create focused reports from large tables.

### References

- PostgreSQL: Combining Queries (UNION, INTERSECT, EXCEPT) — https://www.postgresql.org/docs/current/queries-union.html
- MySQL: Set Operations with UNION, INTERSECT, and EXCEPT — https://dev.mysql.com/doc/refman/8.0/en/set-operations.html
- Oracle Database: Set Operators — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/Using-Set-Operators.html
- Wikipedia: Relational Algebra — https://en.wikipedia.org/wiki/Relational_algebra
- IBM: Relational Operations — https://www.ibm.com/docs/en/db2/11.5?topic=model-relational-operations


## Core Concept 2: Difference Between Row-Wise and Set-Wise Logic

### Definitions

**Core Definition:** Row-wise logic processes data one row at a time using procedural constructs (loops, cursors, conditional branches), while set-wise logic processes entire sets of rows in a single declarative statement, allowing the database engine to optimize execution.

**Technical Definition:** Classical programming as taught in many universities leads to an atomic, row-oriented, and procedural style inspired by the structured models of programming. In short, many application developers write in the relational database exactly like in the user interface. The other style of programming is holistic, data set oriented, and coded mainly in SQL. This is the style of the database developer. A set-based solution performs an action over an entire result set, while a cursor is limited to processing one row at a time. The difference is that in the set-based way, all the iteration is done inside the database engine, in the most efficient way it can, rather than manually iterating outside of the database using a cursor.

**Beginner-Friendly Explanation:** Row-wise logic is like a factory worker who picks up one item at a time from a conveyor belt, inspects it, stamps it, and puts it down. Set-wise logic is like a machine that stamps the entire batch at once. The machine is faster, uses fewer resources, and does not get tired. SQL is designed to be a machine—it works best when you give it a whole batch to process at once.

### Purposes

- **To choose the right approach** for a given data processing task based on performance requirements.
- **To understand why cursor-based solutions are slow** and how set-based alternatives can be orders of magnitude faster.
- **To recognize situations where row-wise processing is unavoidable** (e.g., calling an external API per row) and minimize its scope.
- **To adopt a holistic, data set oriented mindset** that aligns with how relational database engines are designed.

### Syntax Rules and Structure

#### Complete General Syntax (Row-Wise — Cursor)

```sql
DECLARE @id INT, @value FLOAT;
DECLARE cursor_name CURSOR FOR
    SELECT id, value FROM source_table WHERE condition;
OPEN cursor_name;
FETCH NEXT FROM cursor_name INTO @id, @value;
WHILE @@FETCH_STATUS = 0
BEGIN
    UPDATE target_table SET column = @value * 1.1 WHERE id = @id;
    FETCH NEXT FROM cursor_name INTO @id, @value;
END;
CLOSE cursor_name;
DEALLOCATE cursor_name;
```

#### Complete General Syntax (Set-Wise — Single UPDATE)

```sql
UPDATE target_table
SET column = value * 1.1
WHERE condition;
```

#### Syntax Rules

- **Row-wise approach:** The cursor-based way (row-by-agonizing-row, or RBAR) sets up a scan based on the condition, then runs a separate UPDATE transaction for each row returned from the scan, incurring all the overhead of setting up the UPDATE query, starting the transaction, seeking to the correct row, updating it, and tearing down the transaction and query framework again each time.
- **Set-wise approach:** The set-based way performs one scan based on the condition, which updates rows matching the condition, but the query, transaction, and scan are only set up once, and then all the rows are processed, one-at-a-time inside the database engine.
- **Avoiding row-by-row processing:** A set-based program is not an all-or-nothing situation. There are some rules that call for row-by-row processing, but these rules are the exception. However, you can have a row-by-row component within a mostly set-based program.

#### Constraints and Limitations

- **Set-based approach performance:** The set-based approach is faster than any other approach for anything but the smallest amounts of data. The cursor approach gets a lot slower quickly (fairly proportionally, at least).
- **Row-wise exceptions:** Some operations—such as calling an external web service per row, or processing rows in a strict sequence—genuinely require row-by-row processing.
- **Cognitive shift required:** The techniques are unfamiliar to many database developers, so they may be more difficult. Because a set-based model is completely different from an iterative model, changing it requires completely rewriting the source code.

### Annotated Code Examples

#### Example 1: SQL Server — RBAR vs. Set-Based Update

```sql
-- Create a sample table
CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    ProductType INT,
    Price DECIMAL(10,2)
);

INSERT INTO Products VALUES
(1, 1, 100.00),
(2, 1, 200.00),
(3, 2, 150.00),
(4, 1, 300.00);

-- ROW-WISE: Cursor-based update (RBAR)
DECLARE @ProductID INT;
DECLARE @Price DECIMAL(10,2);

DECLARE MyUpdate CURSOR FAST_FORWARD FOR
    SELECT ProductID, Price FROM Products WHERE ProductType = 1;

OPEN MyUpdate;
FETCH NEXT FROM MyUpdate INTO @ProductID, @Price;

WHILE @@FETCH_STATUS = 0
BEGIN
    UPDATE Products SET Price = @Price * 1.1 WHERE ProductID = @ProductID;
    FETCH NEXT FROM MyUpdate INTO @ProductID, @Price;
END;

CLOSE MyUpdate;
DEALLOCATE MyUpdate;
```

**Expected Output (after cursor):**

```
ProductID | ProductType | Price
----------+-------------+--------
1         | 1           | 110.00
2         | 1           | 220.00
3         | 2           | 150.00
4         | 1           | 330.00
```

```sql
-- SET-BASED: Single UPDATE statement
UPDATE Products
SET Price = Price * 1.1
WHERE ProductType = 1;
```

**Expected Output (after set-based):**

```
ProductID | ProductType | Price
----------+-------------+--------
1         | 1           | 121.00
2         | 1           | 242.00
3         | 2           | 150.00
4         | 1           | 363.00
```

**Why This Works:** The cursor approach sets up a scan, then runs a separate UPDATE for each row, incurring all the overhead of query setup, transaction start, row seek, update, and teardown each time. The set-based approach performs one scan and one UPDATE statement, processing all rows in a single transaction. The difference is that in the set-based way, all the iteration is done inside the database engine, in the most efficient way it can.

#### Example 2: MySQL — Reselect vs. Set-Based

```sql
-- ROW-WISE: Reselect approach (calling a function per row)
-- This is pseudo-code; in practice, this pattern appears in application code
-- or stored procedures that loop over order IDs and call a function.

-- SET-BASED: Single SELECT with JOIN
SELECT o.order_id, o.order_date, c.customer_name, SUM(oi.quantity * oi.price) AS total
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.order_date >= '2026-01-01'
GROUP BY o.order_id, o.order_date, c.customer_name
HAVING SUM(oi.quantity * oi.price) > 1000;
```

**Why This Works:** The set-based query combines orders, customers, and order items in a single statement, filters by date, aggregates the total, and filters by the aggregate—all in one pass. The row-wise alternative would require fetching each order, then querying its items, then computing the total, then comparing—repeated for every order.

### Real-World Cases

- **Batch processing:** Updating thousands of rows in a single statement instead of looping.
- **Reporting:** Generating aggregated reports with `GROUP BY` instead of application-level loops.
- **Data migration:** Moving and transforming data in bulk using `INSERT ... SELECT` instead of row-by-row inserts.
- **ETL pipelines:** Replacing row-by-row transformation logic with set-based SQL statements.

### References

- Microsoft Learn: Procedural versus Set-Based SQL — https://learn.microsoft.com/en-us/archive/blogs/simonince/procedural-versus-set-based-sql
- SQLSkills: Reconciling Set-Based Operations with Row-by-Row Iterative Processing — https://www.sqlskills.com/blogs/paul/reconciling-set-based-operations-with-row-by-row-iterative-processing/
- Oracle: Set Based Versus Row Based Operating Modes — https://docs.oracle.com/en/database/oracle/oracle-database/21/tdddg/building-effective-applications.html
- Apress: Relational Database Programming: A Set-Oriented Approach — https://rd.springer.com/book/10.1007/978-1-4842-2080-1


## Core Concept 3: Replacing Procedural Loops with Set Operations

### Definitions

**Core Definition:** Replacing procedural loops with set operations is the process of identifying cursor-based or loop-based row-by-row logic and rewriting it as a single declarative SQL statement (or a small number of statements) that operates on the entire set of affected rows at once.

**Technical Definition:** Loops containing query execution statements are often found to be performance bottlenecks. Repeated execution of a query by a loop can often be avoided by rewriting the query into its set-oriented form and moving it outside the loop. This process consists of two key steps: (i) loop distribution (loop fission) to form a canonical query execution loop, and (ii) replacing the canonical query execution loop with the set-oriented form of the query. The rewrite typically involves replacing cursors with simple set join operations.

**Beginner-Friendly Explanation:** Replacing loops with set operations is like replacing a hand-cranked assembly line with an automated conveyor belt. You identify the repetitive steps (the loop body), figure out what they accomplish collectively, and write a single SQL statement that does the same thing for all rows at once. The database engine then handles the iteration internally, far more efficiently than any application loop ever could.

### Purposes

- **To eliminate cursor overhead** and improve query performance by orders of magnitude.
- **To reduce network round-trips** by keeping data inside the database engine.
- **To simplify code** by replacing complex procedural logic with declarative statements.
- **To leverage the query optimizer** to choose the best execution plan for the entire set of rows.
- **To process exceptions row-by-row** only when set-based logic cannot handle them, using a two-pass approach.

### Syntax Rules and Structure

#### Complete General Syntax (Cursor-to-Join Rewrite)

```sql
-- BEFORE (row-wise): Nested cursor
DECLARE outer_cursor CURSOR FOR SELECT EntityId, BaseId FROM outerTable;
OPEN outer_cursor;
FETCH NEXT FROM outer_cursor INTO @EntityId, @BaseId;
WHILE @@FETCH_STATUS = 0
BEGIN
    DECLARE inner_cursor CURSOR FOR
        SELECT PerfId FROM innerTable WHERE EntityId = @BaseId;
    OPEN inner_cursor;
    FETCH NEXT FROM inner_cursor INTO @PerfId;
    WHILE @@FETCH_STATUS = 0
    BEGIN
        UPDATE largeTable SET EntityId = @EntityId WHERE PerfId = @PerfId;
        FETCH NEXT FROM inner_cursor INTO @PerfId;
    END;
    CLOSE inner_cursor;
    DEALLOCATE inner_cursor;
    FETCH NEXT FROM outer_cursor INTO @EntityId, @BaseId;
END;
CLOSE outer_cursor;
DEALLOCATE outer_cursor;

-- AFTER (set-wise): Single JOIN and UPDATE
SELECT i.PerfId, o.EntityId
INTO #tempTable
FROM innerTable i
JOIN outerTable o ON i.EntityId = o.BaseId;

UPDATE largeTable
SET EntityId = t.EntityId
FROM largeTable lt
JOIN #tempTable t ON lt.PerfId = t.PerfId;
```

#### Complete General Syntax (Two-Pass Approach)

```sql
-- PASS 1: Apply the rule to all rows (set-based)
UPDATE target_table
SET status = 'PROCESSED'
WHERE condition;

-- PASS 2: Handle exceptions individually (row-wise, minimal scope)
-- Only the rows that failed the rule in pass 1 are processed row by row
SELECT id, error_detail
FROM target_table
WHERE status = 'EXCEPTION';
-- Process each exception row individually (e.g., via application logic or a focused cursor)
```

#### Syntax Rules

- **Filter early:** Use the `WHERE` clause to minimize the number of rows to reflect only the set of affected rows.
- **Two-pass approach:** Use a two-pass approach, wherein the first pass runs a rule on all of the rows, and the second pass resolves any rows that are exceptions to the rule.
- **Parallel processes:** Divide sets into distinct groups, and then run the appropriate rules or logic against each set in parallel processes.
- **Flat temporary tables:** Flatten your temporary tables. The best temporary tables are denormalized and follow a flat file model for improved transaction processing.

#### Constraints and Limitations

- **Not all logic is set-able:** Some operations—such as calling an external API per row, or processing rows in strict sequential order—genuinely require row-by-row processing.
- **Code rewrite effort:** Because a set-based model is completely different from an iterative model, changing it requires completely rewriting the source code.
- **Debugging complexity:** Set-based code can be harder to debug because the iteration is invisible; use of temporary staging tables and intermediate result sets can help.

### Annotated Code Examples

#### Example 1: SQL Server — Nested Cursor to Set-Based Join

```sql
-- BEFORE: Nested cursor (RBAR) — simplified from a real-world 200-million-row table
-- This cursor had been running for over 30 hours with only 0.5% progress.
DECLARE @EntityId VARCHAR(16), @BaseId VARCHAR(16), @PerfId VARCHAR(16);
DECLARE outerCursor CURSOR FOR SELECT EntityId, BaseId FROM outerTable;
OPEN outerCursor;
FETCH NEXT FROM outerCursor INTO @EntityId, @BaseId;
WHILE @@FETCH_STATUS = 0
BEGIN
    DECLARE innerCursor CURSOR FOR
        SELECT PerfId FROM innerTable WHERE EntityId = @BaseId;
    OPEN innerCursor;
    FETCH NEXT FROM innerCursor INTO @PerfId;
    WHILE @@FETCH_STATUS = 0
    BEGIN
        UPDATE largeTable SET EntityId = @EntityId WHERE PerfId = @PerfId;
        FETCH NEXT FROM innerCursor INTO @PerfId;
    END;
    CLOSE innerCursor; DEALLOCATE innerCursor;
    FETCH NEXT FROM outerCursor INTO @EntityId, @BaseId;
END;
CLOSE outerCursor; DEALLOCATE outerCursor;
```

```sql
-- AFTER: Set-based join — the rewrite is as simple as this
SELECT i.PerfId, o.EntityId
INTO #tempTable
FROM innerTable i
JOIN outerTable o ON i.EntityId = o.BaseId;

UPDATE largeTable
SET EntityId = t.EntityId
FROM largeTable lt
JOIN #tempTable t ON lt.PerfId = t.PerfId;
```

**Expected Output (set-based):**

```
(204000 rows affected)
-- Completed in seconds instead of months
```

**Why This Works:** The nested cursor iterates 204,000 times in the outer loop and many more times in the inner loop, issuing a separate UPDATE for each row. The set-based rewrite joins the two source tables once, materializes the mapping in a temporary table, and then performs a single UPDATE statement that processes all 200 million rows in one pass. The performance improvement is orders of magnitude.

#### Example 2: SAP HANA — Cursor to Table Assignment

```sql
-- BEFORE: Cursor with scalar function (row-by-row)
CREATE FUNCTION sfunc2(...) RETURNS v_result NVARCHAR(2) AS
    CURSOR c1 FOR SELECT ...;
BEGIN
    FOR rec AS c1 DO
        IF (rec.C = 1 OR rec.C = 3) THEN
            ex_result := rec.C;
            RETURN;
        END IF;
    END FOR;
    ex_result := NULL;
END;

-- AFTER: Set-based table assignment and aggregation
CREATE PROCEDURE proc2(..., OUT v_result NVARCHAR(2)) READS SQL DATA AS
BEGIN
    TAB_B = SELECT ip.B, ip.*, view1.esi, view1.esti, view1.au
    FROM (...) view1
    RIGHT OUTER JOIN (...) ip ON view1.B = ip.B;

    CALL PROC_3(:A, :TAB_B, ..., TAB_C);

    TAB_RES = SELECT MAX(C) FROM :TAB_C WHERE C IN (1, 3);
    IF (NOT IS_EMPTY(:TAB_RES)) THEN
        ex_result = :TAB_C.C[1];
        RETURN;
    END IF;
    ex_result := NULL;
END;
```

**Expected Output (from SAP benchmark):**

```
-- Performance improved by a factor of 85 through rewriting
```

**Why This Works:** The original scenario uses cursors with scalar functions called for each row. The scalar functions are rewritten as procedures, and all values are contained in tables. The operation now operates on a set of values rather than a single row at a time. Performance was improved by a factor of 85 through rewriting.

### Real-World Cases

- **ETL performance:** Rewriting a row-by-row ETL that was looping over a temp table with a CTE-based set operation.
- **Batch updates:** Converting a cursor-based update of a 200-million-row table into a single set-based `UPDATE ... FROM` statement.
- **Data transformation:** Replacing nested loops with a single `SELECT ... JOIN` that produces the same result.
- **Exception handling:** Using a two-pass approach: set-based for the majority of rows, row-by-row only for exceptions.

### References

- Microsoft Learn: Increase Your SQL Server Performance by Replacing Cursors with Set Operations — https://learn.microsoft.com/en-us/archive/blogs/mssqlisv/increase-your-sql-server-performance-by-replacing-cursors-with-set-operations
- SAP Help: Replacing Row-Based Calculations with Set-Based Calculations — https://help.sap.com/docs/SAP_HANA_PLATFORM/9de0171a6027400bb3b9bee385222eff/2b12462ffa2040569aaccb1ae3fa6119.html
- Oracle: Avoiding Row-by-Row Processing — https://docs.oracle.com/cd/B31274_01/psft/acrobat/pt848ape-b0606.pdf
- Oracle: Automatic Conversion of Cursor For Loop into Set-Based Operation — https://asktom.oracle.com/pls/apex/f?p=100:11:0::::P11_QUESTION_ID:9528428000346700647


## Core Concept 4: Query Composition

### Definitions

**Core Definition:** Query composition is the practice of building complex queries by combining smaller, reusable query blocks—using Common Table Expressions (CTEs), derived tables, subqueries, and set operators—to express multi-step logic in a readable, maintainable way.

**Technical Definition:** Common Table Expressions (CTEs) provide a mechanism for you to define a subquery that may then be used elsewhere in a query. Unlike a derived table, a CTE is defined at the beginning of a query and may be referenced multiple times in the outer query. CTEs are named expressions defined in a query. Like subqueries and derived tables, CTEs provide a means to break down query problems into smaller, more modular units. CTEs are limited in scope to the execution of the outer query. When the outer query ends, so does the CTE's lifetime. Multiple CTEs may also be defined in the same `WITH` clause. CTEs support recursion, in which the expression is defined with a reference to itself.

**Beginner-Friendly Explanation:** Query composition is like building with LEGO bricks. Instead of writing one giant, tangled query, you break the problem into smaller pieces—each CTE or subquery is a brick. You assemble the bricks with `WITH` clauses and joins, and the result is a clear, readable structure that is easy to debug and modify.

### Purposes

- **To break complex problems into smaller, more modular units** using CTEs and subqueries.
- **To reference the same subquery multiple times** without repeating its definition, using CTEs.
- **To build layered transformations** where each CTE adds a step of logic.
- **To improve query readability** by naming intermediate result sets.
- **To support recursive logic** (e.g., organizational hierarchies, bill-of-materials) using recursive CTEs.

### Syntax Rules and Structure

#### Complete General Syntax (CTE)

```sql
WITH cte_name_1 AS (
    SELECT ... FROM source_table WHERE ...
),
cte_name_2 AS (
    SELECT ... FROM cte_name_1 JOIN other_table ON ...
),
cte_name_3 AS (
    SELECT ..., CASE WHEN ... THEN ... END AS derived_column
    FROM cte_name_2
)
SELECT * FROM cte_name_3;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `WITH` | Introduces one or more CTEs. |
| `cte_name AS (query)` | A named temporary result set. |
| `,` | Separates multiple CTEs; later CTEs can reference earlier ones. |
| `SELECT * FROM cte_name_3` | Final query consuming the last CTE. |

#### Complete General Syntax (Derived Table)

```sql
SELECT outer_columns
FROM (
    SELECT inner_columns
    FROM source_table
    WHERE inner_conditions
    GROUP BY grouping_columns
) AS derived_alias
WHERE outer_conditions;
```

#### Complete General Syntax (Set Operators in Composition)

```sql
WITH cte_a AS (...), cte_b AS (...)
SELECT * FROM cte_a
UNION
SELECT * FROM cte_b;
```

#### Syntax Rules

- **CTE naming:** CTEs require a name for the table expression. Column names in the CTE result set must be unique—either because the underlying `SELECT` columns already have distinct names, or by supplying column aliases.
- **Multiple CTEs:** Multiple CTEs may be defined in the same `WITH` clause, separated by commas. Later CTEs may reference earlier CTEs, but not those defined later. This constraint rules out mutually recursive CTEs.
- **CTE referencing:** Unlike a derived table, a CTE may be referenced multiple times in the same query with one definition.
- **Batch separator:** When a CTE follows another statement in the same batch, the preceding statement must end with a semicolon (;).
- **Recursive CTEs:** A self-referencing CTE is recursive. A recursive CTE is a `UNION ALL` subquery with two parts: a base case and a recursive case.

#### Constraints and Limitations

- **CTE scope:** CTEs are limited in scope to the execution of the outer query. When the outer query ends, so does the CTE's lifetime.
- **CTE materialization:** Some databases materialize CTEs into temporary tables, which can be slower than inline subqueries for simple cases. PostgreSQL 12+ can inline CTEs when they are not recursive and are referenced only once.
- **Derived table alias:** Every derived table must have its own alias. Omitting the alias causes a syntax error in most databases.
- **Mutual recursion:** A CTE can refer to CTEs defined earlier in the same `WITH` clause, but not those defined later. This constraint rules out mutually recursive CTEs, where cte1 references cte2 and cte2 references cte1.

### Annotated Code Examples

#### Example 1: PostgreSQL — Multi-Step CTE Pipeline

```sql
-- Create source tables
CREATE TABLE raw_sales (
    id SERIAL PRIMARY KEY,
    product_name TEXT,
    quantity INT,
    unit_price NUMERIC(10,2),
    sale_date DATE
);

INSERT INTO raw_sales (product_name, quantity, unit_price, sale_date) VALUES
('Widget', 10, 19.99, '2026-01-15'),
('Gadget', 5, 29.99, '2026-01-16'),
('Widget', 8, 19.99, '2026-02-10'),
('Gadget', 3, 29.99, '2026-02-15'),
('Doohickey', 20, 9.99, '2026-03-01');

-- Multi-step CTE pipeline
WITH monthly_sales AS (
    -- Step 1: Calculate revenue per product per month
    SELECT
        product_name,
        DATE_TRUNC('month', sale_date) AS sale_month,
        SUM(quantity * unit_price) AS revenue,
        SUM(quantity) AS total_quantity
    FROM raw_sales
    GROUP BY product_name, DATE_TRUNC('month', sale_date)
),
ranked_products AS (
    -- Step 2: Rank products by revenue within each month
    SELECT
        product_name,
        sale_month,
        revenue,
        total_quantity,
        RANK() OVER (PARTITION BY sale_month ORDER BY revenue DESC) AS revenue_rank
    FROM monthly_sales
),
flagged_products AS (
    -- Step 3: Flag top performers
    SELECT
        product_name,
        sale_month,
        revenue,
        total_quantity,
        revenue_rank,
        CASE
            WHEN revenue_rank = 1 THEN 'Top Seller'
            WHEN revenue_rank <= 3 THEN 'Strong Performer'
            ELSE 'Standard'
        END AS performance_category
    FROM ranked_products
)
-- Final output
SELECT * FROM flagged_products
ORDER BY sale_month, revenue_rank;
```

**Expected Output:**

```
product_name | sale_month | revenue | total_quantity | revenue_rank | performance_category
-------------+------------+---------+----------------+--------------+---------------------
Widget       | 2026-01-01 |  199.90 |             10 |            1 | Top Seller
Gadget       | 2026-01-01 |  149.95 |              5 |            2 | Strong Performer
Widget       | 2026-02-01 |  159.92 |              8 |            1 | Top Seller
Gadget       | 2026-02-01 |   89.97 |              3 |            2 | Strong Performer
Doohickey    | 2026-03-01 |  199.80 |             20 |            1 | Top Seller
```

**Why This Works:** The first CTE aggregates raw sales into monthly revenue per product. The second CTE ranks products by revenue within each month using `RANK()`. The third CTE applies conditional branching with `CASE` to classify performance. The final query consumes the last CTE. Each step is independently readable and testable. CTEs provide a means to break down query problems into smaller, more modular units.

#### Example 2: PostgreSQL — CTE with Multiple References and Set Operators

```sql
-- Use a CTE multiple times in the same query
WITH order_summary AS (
    SELECT customer_id, COUNT(*) AS order_count, SUM(total) AS lifetime_value
    FROM orders
    GROUP BY customer_id
),
high_value AS (
    SELECT customer_id FROM order_summary WHERE lifetime_value > 1000
),
frequent AS (
    SELECT customer_id FROM order_summary WHERE order_count > 5
)
-- Find customers who are both high-value AND frequent
SELECT c.customer_id, c.customer_name
FROM customers c
JOIN high_value hv ON c.customer_id = hv.customer_id
INTERSECT
SELECT c.customer_id, c.customer_name
FROM customers c
JOIN frequent f ON c.customer_id = f.customer_id;
```

**Expected Output:**

```
customer_id | customer_name
------------+---------------
        101 | Alice
        103 | Carol
```

**Why This Works:** The CTE `order_summary` is defined once and referenced twice (in `high_value` and `frequent`). The `INTERSECT` operator combines the two result sets, returning only customers who appear in both. Unlike a derived table, a CTE may be referenced multiple times in the same query with one definition.

### Real-World Cases

- **ETL pipelines:** Building layered transformations with CTEs for staging, cleansing, and aggregating data.
- **Reporting:** Composing complex reports from reusable query blocks.
- **Recursive hierarchies:** Using recursive CTEs to traverse organizational charts or bill-of-materials structures.
- **Data quality:** Using `EXCEPT` to find records in one system that are missing from another.
- **Multi-tenant analytics:** Composing per-tenant queries with shared CTEs for common transformations.

### References

- Microsoft Learn: Use Common Table Expressions — https://learn.microsoft.com/en-us/training/modules/create-tables-views-temporary-objects/5-use-common-table-expressions
- PostgreSQL: WITH Queries (Common Table Expressions) — https://www.postgresql.org/docs/current/queries-with.html
- MySQL: WITH (Common Table Expressions) — https://dev.mysql.com/doc/refman/8.0/en/with.html
- SQL Server: WITH common_table_expression — https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql
- PostgreSQL: Combining Queries (UNION, INTERSECT, EXCEPT) — https://www.postgresql.org/docs/current/queries-union.html


## Summary Table: Set-Based Thinking at a Glance

| Concept | Row-Wise (Procedural) | Set-Wise (Declarative) |
|---------|----------------------|------------------------|
| **Approach** | Loop over rows, process one at a time | Define the result, let the database compute it |
| **Constructs** | Cursors, `WHILE` loops, `FETCH NEXT` | `SELECT`, `UPDATE`, `JOIN`, `GROUP BY`, CTEs |
| **Performance** | Slow, degrades proportionally with row count | Orders of magnitude faster on large datasets |
| **Code Length** | Long, verbose, repetitive | Short, concise, declarative |
| **Network Round-Trips** | One per row (or per batch) | One per statement |
| **Optimizer Leverage** | None (execution plan is fixed by the loop) | Full (optimizer chooses the best plan) |
| **Exception Handling** | Every row is an exception case | Two-pass: set-based for the rule, row-wise for exceptions |
| **When to Use** | External API calls per row, strict sequential processing | Nearly everything else |


## Final Notes on Deprecated and Unsafe Features

- **Cursors:** Cursors are not deprecated, but they are a last resort. The cursor approach gets a lot slower quickly (fairly proportionally, at least) as data volume grows. Use cursors only when row-by-row processing is genuinely unavoidable.
- **`WHILE` loops in T-SQL:** Azure Synapse Analytics documentation notes that a `WHILE` loop is a good replacement for a cursor, but you should still ask: "Could this cursor be rewritten to use set-based operations?" In many cases, the answer is yes and is frequently the best approach.
- **Implicit cursor conversion:** Oracle will not transform looped code into a set-based operation. But what it will do is optimize fetches in cursor for loops in PL/SQL. From 10g onwards, Oracle automatically uses an array fetch size of 100 for these.
- **CTE materialization:** In PostgreSQL versions before 12, CTEs are always materialized; use `NOT MATERIALIZED` (PostgreSQL 12+) to allow inlining. In SQL Server, CTEs are not materialized and can lead to repeated re-execution of the underlying syntax.
- **Mutually recursive CTEs:** A CTE can refer to CTEs defined earlier in the same `WITH` clause, but not those defined later. This constraint rules out mutually recursive CTEs.
- **`EXCEPT` vs. `MINUS`:** Oracle uses `MINUS` instead of `EXCEPT` for set difference; this is not supported in MySQL. Use `EXCEPT` for portability across PostgreSQL, SQL Server, and MySQL 8.0.31+.
- **Version-specific:** MySQL `INTERSECT` and `EXCEPT` require MySQL 8.0.31+; PostgreSQL `NOT MATERIALIZED` requires PostgreSQL 12+; recursive CTEs are supported in PostgreSQL 8.4+, SQL Server 2005+, MySQL 8.0+, and Oracle 11gR2+.