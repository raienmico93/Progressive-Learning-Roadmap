# SQL Sorting Results: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The ORDER BY clause is a SELECT statement clause that sorts the rows of a result set by one or more expressions, producing an ordered output.

**Technical Definition:** Per the ANSI/ISO SQL Standard, the ORDER BY clause is an optional component of a query expression that specifies a sort specification — a list of sort keys, each with an optional sort direction (ASC or DESC) and optional null ordering (NULLS FIRST or NULLS LAST). It is logically evaluated after the SELECT list has been processed, after DISTINCT, and before LIMIT/OFFSET. If no ORDER BY is specified, the row order is unspecified and dependent on the query plan.

**Beginner-Friendly Explanation:** A database table has no guaranteed order — rows come back however the database finds them most efficient. ORDER BY is how you tell the database "give me these results sorted by this column, from smallest to largest" or "from newest to oldest." It's the equivalent of clicking "Sort A to Z" in a spreadsheet.

### Key Characteristics

- **Logically last:** ORDER BY is the last phase of a SELECT statement to be executed in the logical order of query processing.
- **Default ascending:** If no direction is specified, ASC is assumed.
- **Multi-key sorting:** Multiple sort expressions can be specified, each with its own direction.
- **NULL-aware:** NULL ordering can be controlled explicitly with NULLS FIRST and NULLS LAST.
- **Alias and expression support:** Can sort by column aliases, expressions, and (in most databases) positional numbers.
- **Index-sensitive:** A sorted result can be delivered by an index scan instead of an explicit sort operation, which is much faster on large tables.

### Prerequisites

- Basic SQL syntax (SELECT, FROM, WHERE)
- Understanding of data types and comparison operators
- Familiarity with NULL handling
- Basic knowledge of indexes

### Related Programming Areas

- **Query Optimization:** Sorting is one of the most expensive operations; understanding how to avoid it via indexes is a core tuning skill.
- **Reporting and Analytics:** Sorted output is essential for reports, dashboards, and data presentation.
- **Pagination:** ORDER BY combined with LIMIT/OFFSET is the standard approach for paginated results.
- **Window Functions:** ORDER BY within OVER() clauses controls the order in which window functions process rows.
- **Application Development:** ORMs and query builders expose ORDER BY through API methods.

---

## Core Concepts / Key Features

### 1. The Core Ordering Clause

#### 1.1 ORDER BY Syntax and Logical Execution Order

**Core Definitions:**

- **Core Definition:** ORDER BY is an optional clause that specifies the sort order of the rows returned by a SELECT statement.
- **Technical Definition:** The ORDER BY clause is evaluated after the SELECT list, DISTINCT, and before LIMIT/OFFSET. This means it has access to column aliases defined in the SELECT list. The general form is `ORDER BY sort_expression1 [ASC | DESC] [NULLS { FIRST | LAST }] [, sort_expression2 ...]`.
- **Beginner-Friendly Explanation:** ORDER BY runs almost last — after all the rows have been gathered and filtered, the database sorts them as the final step before sending them to you.

**Purposes:**

- To guarantee a specific, predictable order of rows in the result set
- To prepare data for presentation (reports, user interfaces)
- To support pagination (with LIMIT/OFFSET)
- To define processing order for window functions

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
SELECT select_list
FROM table_expression
ORDER BY sort_expression1 [ASC | DESC] [NULLS { FIRST | LAST }]
       [, sort_expression2 [ASC | DESC] [NULLS { FIRST | LAST }] ...];
```

**Component Breakdown:**

- `sort_expression`: A column name, column alias, positional number, or arbitrary expression
- `ASC | DESC`: Sort direction (ASC is default)
- `NULLS FIRST | NULLS LAST`: Controls where NULLs appear (optional, database-dependent)

**Syntax Rules:**

- ORDER BY is always the last clause in a SELECT statement (before LIMIT/OFFSET in databases that support them).
- Without ORDER BY, the row order is unspecified and must not be relied upon.
- Each sort expression is considered independently for direction and NULL ordering.
- In a UNION/INTERSECT/EXCEPT query, only output column names or numbers can be used, not expressions.

**Constraints and Limitations:**

- ORDER BY on a very large result set can be slow because it requires materializing and sorting all rows.
- Some databases limit the number of columns in ORDER BY (e.g., SQL Server allows up to 8,060 bytes; SAP allows up to 400 columns).
- ORDER BY does not guarantee a stable order for rows with equal sort keys unless a unique key is included in the sort.

**Annotated Code Examples:**

```sql
-- Setup
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10,2)
);

INSERT INTO employees VALUES (1, 'Alice', 'Engineering', 85000);
INSERT INTO employees VALUES (2, 'Bob', 'Marketing', 62000);
INSERT INTO employees VALUES (3, 'Charlie', 'Engineering', 95000);
INSERT INTO employees VALUES (4, 'Diana', 'Sales', 48000);
INSERT INTO employees VALUES (5, 'Eve', 'Engineering', 110000);

-- Basic ORDER BY
SELECT emp_name, salary
FROM employees
ORDER BY salary;
```

**Expected Output:**

| emp_name | salary |
|----------|--------|
| Diana    | 48000  |
| Bob      | 62000  |
| Alice    | 85000  |
| Charlie  | 95000  |
| Eve      | 110000 |

**Explanation:** The result is sorted by salary in ascending order (the default). Diana (48,000) comes first, Eve (110,000) last.

**Real-World Cases:**

- **Reports:** Monthly sales reports sorted by revenue descending.
- **User interfaces:** Contact lists sorted alphabetically.
- **Pagination:** Combined with LIMIT/OFFSET to fetch pages of sorted data.

**References:**

- PostgreSQL: Sorting Rows (ORDER BY) - https://www.postgresql.org/docs/17/queries-order.html
- Microsoft Learn: Sort your results - https://learn.microsoft.com/en-us/training/modules/sort-filter-queries/2-sort-your-data-as-its-returned


### 2. Sorting Directions

#### 2.1 Ascending Order (ASC, the Default)

**Core Definitions:**

- **Core Definition:** ASC specifies ascending sort order: smallest values first, largest last.
- **Technical Definition:** In ascending order, values are compared using the `<` operator; the smallest value comes first. For character data, this depends on the collation (e.g., A-Z, 0-9).
- **Beginner-Friendly Explanation:** ASC is "A to Z" or "1 to 9" order. It's the default, so you don't even need to write it.

**Annotated Code Examples:**

```sql
-- ASC is the default (explicit and implicit are equivalent)
SELECT emp_name, salary 
FROM employees 
ORDER BY salary ASC;

SELECT emp_name, salary 
FROM employees 
ORDER BY salary;
```

**Real-World Cases:**

- **Names:** Alphabetical listing of customers.
- **Dates:** Chronological order (oldest first).
- **Prices:** Lowest price first for budget-conscious users.

**References:**

- PostgreSQL: Sorting Rows - https://www.postgresql.org/docs/17/queries-order.html
- Microsoft Learn: Sort your results - https://learn.microsoft.com/en-us/training/modules/sort-filter-queries/2-sort-your-data-as-its-returned


#### 2.2 Descending Order (DESC)

**Core Definitions:**

- **Core Definition:** DESC specifies descending sort order: largest values first, smallest last.
- **Technical Definition:** In descending order, values are compared using the `>` operator; the largest value comes first.
- **Beginner-Friendly Explanation:** DESC is "Z to A" or "9 to 1" order. You must explicitly write DESC.

**Annotated Code Examples:**

```sql
-- Descending order
SELECT emp_name, salary 
FROM employees 
ORDER BY salary DESC;
```

**Expected Output:**

| emp_name | salary |
|----------|--------|
| Eve      | 110000 |
| Charlie  | 95000  |
| Alice    | 85000  |
| Bob      | 62000  |
| Diana    | 48000  |

**Explanation:** The highest salary comes first, the lowest last.

**Real-World Cases:**

- **Leaderboards:** Highest scores first.
- **News feeds:** Newest articles first.
- **Financial reports:** Largest revenue first.

**References:**

- PostgreSQL: Sorting Rows - https://www.postgresql.org/docs/17/queries-order.html
- Oracle: SQL Beginner's Guide - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SELECT.html


### 3. Sorting Drivers & Targets

#### 3.1 Multi-Column Sorting (Primary, Secondary, Tertiary Hierarchies)

**Core Definitions:**

- **Core Definition:** Multi-column sorting uses two or more sort expressions to establish a hierarchy: the first column is the primary sort, the second breaks ties in the first, and so on.
- **Technical Definition:** When more than one expression is specified, later values are used to sort rows that are equal according to earlier values. Each column has its own independent direction and NULL ordering.
- **Beginner-Friendly Explanation:** Think of sorting a phone book by last name, then first name. The last name is the primary sort; for people with the same last name, the first name breaks the tie.

**Purposes:**

- To create hierarchical sort orders (e.g., country → state → city)
- To break ties in a meaningful way
- To produce fully deterministic output

**Annotated Code Examples:**

```sql
-- Sort by department (ASC), then salary (DESC) within each department
SELECT emp_name, department, salary
FROM employees
ORDER BY department ASC, salary DESC;
```

**Expected Output:**

| emp_name | department   | salary |
|----------|--------------|--------|
| Eve      | Engineering  | 110000 |
| Charlie  | Engineering  | 95000  |
| Alice    | Engineering  | 85000  |
| Bob      | Marketing    | 62000  |
| Diana    | Sales        | 48000  |

**Explanation:** Departments are sorted alphabetically (Engineering, Marketing, Sales). Within Engineering, salaries are sorted descending (Eve, Charlie, Alice).

**Real-World Cases:**

- **E-commerce:** Sort by category, then by price.
- **HR:** Sort by department, then by hire date.
- **Logistics:** Sort by region, then by priority.

**References:**

- PostgreSQL: Sorting Rows - https://www.postgresql.org/docs/17/queries-order.html
- Microsoft Learn: Sort your results - https://learn.microsoft.com/en-us/training/modules/sort-filter-queries/2-sort-your-data-as-its-returned


#### 3.2 Sorting by Column Aliases

**Core Definitions:**

- **Core Definition:** Because ORDER BY is processed after SELECT, it can reference column aliases defined in the SELECT list.
- **Technical Definition:** A sort expression can be the column label (alias) of an output column. The alias must stand alone; it cannot be used within an expression in ORDER BY.
- **Beginner-Friendly Explanation:** If you rename a column in your SELECT (e.g., `salary AS pay`), you can use that new name in ORDER BY.

**Annotated Code Examples:**

```sql
-- Sorting by a column alias
SELECT emp_name AS name, salary * 12 AS annual_salary
FROM employees
ORDER BY annual_salary DESC;
```

**Expected Output:**

| name    | annual_salary |
|---------|---------------|
| Eve     | 1320000       |
| Charlie | 1140000       |
| Alice   | 1020000       |
| Bob     | 744000        |
| Diana   | 576000        |

**Explanation:** The alias `annual_salary` is defined in the SELECT list and used in ORDER BY. This works because ORDER BY is logically evaluated after SELECT.

**Constraints and Limitations:**

- In a UNION query, aliases must come from the first SELECT list.
- The alias cannot be used inside an expression in ORDER BY (e.g., `ORDER BY annual_salary + 1000` is invalid).

**References:**

- PostgreSQL: Sorting Rows - https://www.postgresql.org/docs/17/queries-order.html
- Microsoft Learn: Sort your results - https://learn.microsoft.com/en-us/training/modules/sort-filter-queries/2-sort-your-data-as-its-returned


#### 3.3 Sorting by Expressions, Calculations, or Functions

**Core Definitions:**

- **Core Definition:** ORDER BY can sort by arbitrary expressions, calculations, or function results, not just plain column names.
- **Technical Definition:** A sort expression can be any expression that would be valid in the query's select list. This includes arithmetic expressions, string functions, date functions, and CASE expressions.
- **Beginner-Friendly Explanation:** You can sort by a computed value — for example, by the length of a name, or by a mathematical formula.

**Annotated Code Examples:**

```sql
-- Sort by the length of the employee name
SELECT emp_name, LENGTH(emp_name) AS name_length
FROM employees
ORDER BY LENGTH(emp_name);
```

**Expected Output:**

| emp_name | name_length |
|----------|-------------|
| Bob      | 3           |
| Eve      | 3           |
| Alice    | 5           |
| Diana    | 5           |
| Charlie  | 7           |

**Explanation:** Employees are sorted by the length of their name. Bob and Eve (both 3 characters) come first, Charlie (7 characters) last.

```sql
-- Sort by a computed expression
SELECT emp_name, salary, salary * 0.10 AS bonus
FROM employees
ORDER BY salary * 0.10 DESC;
```

**Real-World Cases:**

- **E-commerce:** Sort products by `price * discount` to show best deals first.
- **HR:** Sort employees by `YEAR(CURRENT_DATE) - YEAR(hire_date)` for tenure.
- **Logistics:** Sort shipments by `DATEDIFF(day, ship_date, CURRENT_DATE)` for aging analysis.

**References:**

- PostgreSQL: Sorting Rows - https://www.postgresql.org/docs/17/queries-order.html
- Oracle: SELECT - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SELECT.html


#### 3.4 Sorting by Positional Column Index Numbers

**Core Definitions:**

- **Core Definition:** ORDER BY can reference columns by their ordinal position in the SELECT list (e.g., `ORDER BY 1, 2`).
- **Technical Definition:** A sort column can be specified as a nonnegative integer representing the position of the name or alias in the select list. Positional references are resolved to the corresponding SELECT list item.
- **Beginner-Friendly Explanation:** Instead of writing the column name, you write its position number. `ORDER BY 1` means "sort by the first column in my SELECT list."

**Purposes:**

- To quickly sort by a column without repeating its name
- To sort by a complex expression that doesn't have an alias
- To debug queries by sorting on different columns without editing the SELECT list

**Annotated Code Examples:**

```sql
-- Sort by the first column (emp_name)
SELECT emp_name, salary FROM employees ORDER BY 1;

-- Sort by the second column (salary) descending
SELECT emp_name, salary FROM employees ORDER BY 2 DESC;
```

**Expected Output (for `ORDER BY 1`):**

| emp_name | salary |
|----------|--------|
| Alice    | 85000  |
| Bob      | 62000  |
| Charlie  | 95000  |
| Diana    | 48000  |
| Eve      | 110000 |

**Constraints and Limitations:**

- **Bad practice in production:** Microsoft documentation explicitly states that using position is not recommended in applications due to decreased readability and the need for careful maintenance of the ORDER BY list.
- Positional references are fragile: adding, removing, or reordering a column in the SELECT list silently changes the sort order.
- Not supported in all databases (e.g., some versions of SQL Server have limitations).

**Real-World Cases:**

- **Ad-hoc queries:** Quickly sorting by different columns during exploration.
- **Troubleshooting:** Debugging complex queries with many columns.

**References:**

- Microsoft Learn: Sort your results (positional references) - https://learn.microsoft.com/en-us/training/modules/sort-filter-queries/2-sort-your-data-as-its-returned
- PostgreSQL: Sorting Rows (positional references) - https://www.postgresql.org/docs/17/queries-order.html


### 4. Specialized Sort Mechanics

#### 4.1 NULL Ordering Behaviors and Explicit Controls (NULLS FIRST / NULLS LAST)

**Core Definitions:**

- **Core Definition:** By default, NULL values sort as if they are larger than any non-NULL value. This means NULLs appear last in ASC order and first in DESC order. The NULLS FIRST and NULLS LAST options override this.
- **Technical Definition:** The NULLS FIRST and NULLS LAST options determine whether NULLs appear before or after non-NULL values. By default, NULLS FIRST is the default for DESC order, and NULLS LAST otherwise.
- **Beginner-Friendly Explanation:** NULLs are treated as "bigger than everything," so they end up at the bottom in ascending order. You can force them to the top or bottom with explicit options.

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
ORDER BY column_name [ASC | DESC] [NULLS { FIRST | LAST }]
```

**Component Breakdown:**

- `NULLS FIRST`: NULLs appear before all non-NULL values
- `NULLS LAST`: NULLs appear after all non-NULL values
- The options are considered independently for each sort column

**Database Support:**

| Database | NULLS FIRST/LAST | Default Behavior |
|----------|-----------------|------------------|
| PostgreSQL | ✓ | NULLS LAST for ASC, NULLS FIRST for DESC |
| Oracle | ✓ | NULLS LAST for ASC, NULLS FIRST for DESC |
| SQL Server | ✗ | NULLs sort as lowest values (NULLS FIRST for ASC) |
| MySQL | ✗ | NULLs sort as lowest values (NULLS FIRST for ASC) |

**Annotated Code Examples:**

```sql
-- PostgreSQL: Explicit NULL ordering
SELECT emp_name, salary
FROM employees
ORDER BY salary ASC NULLS FIRST;
```

**Expected Output (with NULL salaries):**

| emp_name | salary |
|----------|--------|
| [NULL]   | NULL   |
| Diana    | 48000  |
| Bob      | 62000  |
| Alice    | 85000  |
| Charlie  | 95000  |
| Eve      | 110000 |

**Explanation:** NULLS FIRST forces the NULL salary to the top, even though ASC would normally place it last.

```sql
-- PostgreSQL: NULLS LAST with DESC
SELECT emp_name, salary
FROM employees
ORDER BY salary DESC NULLS LAST;
```

**Real-World Cases:**

- **Reports:** Showing "unknown" values last in a sorted list.
- **Data quality:** Identifying rows with missing data by sorting them to the top.
- **User interfaces:** Placing incomplete records at the bottom of a list.

**References:**

- PostgreSQL: Sorting Rows (NULLS FIRST/LAST) - https://www.postgresql.org/docs/17/queries-order.html
- SQL Server: ORDER BY Clause - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-order-by-clause-transact-sql


#### 4.2 Conditional Sorting Using CASE Expressions

**Core Definitions:**

- **Core Definition:** A CASE expression within ORDER BY allows the sort order to change dynamically based on row values or external input.
- **Technical Definition:** The CASE expression returns different values for different rows, and those values determine the sort order. SQL Server documentation demonstrates using CASE to conditionally determine the sort order based on a column value.
- **Beginner-Friendly Explanation:** You can tell the database "sort by column A if this condition is true, otherwise sort by column B."

**Purposes:**

- To implement dynamic sorting based on user input (e.g., a dropdown that changes the sort column)
- To sort by different columns for different subsets of rows
- To implement custom priority ordering

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
ORDER BY CASE
    WHEN condition1 THEN expression1
    WHEN condition2 THEN expression2
    ELSE expression3
END [ASC | DESC];
```

**Annotated Code Examples:**

```sql
-- Sort employees: salaried employees by ID descending, hourly by ID ascending
SELECT emp_name, salary, job_type
FROM employees
ORDER BY 
    CASE WHEN job_type = 'Salaried' THEN emp_id END DESC,
    CASE WHEN job_type = 'Hourly' THEN emp_id END ASC;
```

**Explanation:** This uses two CASE expressions. The first sorts salaried employees by ID descending; the second sorts hourly employees by ID ascending. NULLs (from the ELSE-less CASE) cause the other expression to take over.

```sql
-- Dynamic sort based on a variable (PostgreSQL example)
SELECT emp_name, department, salary
FROM employees
ORDER BY 
    CASE WHEN :sort_by = 'name' THEN emp_name END ASC,
    CASE WHEN :sort_by = 'salary' THEN salary END DESC;
```

**Real-World Cases:**

- **User interfaces:** Sorting by different columns based on a dropdown selection.
- **Reports:** Different sort orders for different report sections.
- **Priority queues:** Sorting by priority, then by date within each priority level.

**References:**

- SQL Server: ORDER BY Clause (CASE examples) - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-order-by-clause-transact-sql
- PostgreSQL: CASE in ORDER BY - https://www.postgresql.org/docs/current/functions-conditional.html


### 5. Under-the-Hood Mechanics & Constraints

#### 5.1 Collation and Case Sensitivity

**Core Definitions:**

- **Core Definition:** Collation defines the rules for comparing and sorting character data, including case sensitivity, accent sensitivity, and language-specific ordering.
- **Technical Definition:** The sort order of a result set depends on the collation applied to the database or specified in a COLLATE clause at the expression level of the query. Collation affects whether 'a' sorts before 'B', whether 'é' equals 'e', and whether case matters.
- **Beginner-Friendly Explanation:** Collation is the rulebook for how letters are compared. Different languages and configurations sort letters differently — for example, in some collations 'a' and 'A' are equal, in others they are not.

**Purposes:**

- To understand why the same data sorts differently in different databases
- To choose the correct collation for language-specific sorting
- To control case sensitivity in ORDER BY

**Annotated Code Examples:**

```sql
-- SQL Server: Case-sensitive vs case-insensitive collation
SELECT emp_name FROM employees ORDER BY emp_name COLLATE Latin1_General_CS_AS;
-- 'Alice' and 'alice' are treated as different

SELECT emp_name FROM employees ORDER BY emp_name COLLATE Latin1_General_CI_AS;
-- 'Alice' and 'alice' are treated as equal
```

**Expected Output (CS collation):** Uppercase letters sort before lowercase letters.
**Expected Output (CI collation):** Case is ignored in sorting.

**Real-World Cases:**

- **International applications:** Sorting names correctly in different languages (e.g., German 'ö' sorts with 'o', Swedish 'ö' sorts after 'z').
- **Data migration:** Understanding collation differences when moving data between systems.
- **Search:** Case-insensitive sorting for user-friendly interfaces.

**References:**

- Microsoft Learn: Collation and Unicode Support - https://learn.microsoft.com/en-us/sql/relational-databases/collations/collation-and-unicode-support
- Oracle: Database Globalization Support Guide - https://docs.oracle.com/en/database/oracle/oracle-database/19/nlspg/


#### 5.2 The DISTINCT Constraint

**Core Definitions:**

- **Core Definition:** When SELECT DISTINCT is used, all ORDER BY expressions must appear in the SELECT list. This is an ANSI SQL requirement.
- **Technical Definition:** ORDER BY is evaluated after DISTINCT, so ORDER BY can only reference columns that are part of the DISTINCT result set. A query like `SELECT DISTINCT a FROM t ORDER BY b` is invalid ANSI SQL because `b` is not in the SELECT list.
- **Beginner-Friendly Explanation:** When you ask for unique rows, you can only sort by columns that are part of those unique rows. You can't sort by a column that isn't in the result.

**Syntax Structures and Rules:**

**Valid:**

```sql
SELECT DISTINCT department FROM employees ORDER BY department;
-- department is in the SELECT list
```

**Invalid (ANSI SQL):**

```sql
SELECT DISTINCT department FROM employees ORDER BY salary;
-- salary is NOT in the SELECT list
```

**Workaround:** Use GROUP BY instead of DISTINCT, with an aggregate function:

```sql
SELECT department FROM employees GROUP BY department ORDER BY MAX(salary);
```

**Real-World Cases:**

- **Reporting:** Getting unique combinations and sorting by a derived value.
- **Data exploration:** Understanding the constraint when building queries interactively.

**References:**

- Stack Overflow: DISTINCT and ORDER BY ANSI constraint - https://stackoverflow.com/revisions/6b7be6fa-3c81-4b4d-9b5d-b094102331a7/view-source
- PostgreSQL: SELECT DISTINCT - https://www.postgresql.org/docs/current/sql-select.html


#### 5.3 Performance Implications of Sorting

**Core Definitions:**

- **Core Definition:** Sorting is an expensive operation that requires the database to materialize and order all rows. It can be avoided if an index provides the required order.
- **Technical Definition:** MySQL uses a filesort operation when no index can satisfy the ORDER BY. PostgreSQL's planner considers either scanning an index that matches the sort specification or scanning the table and performing an explicit sort. An explicit sort is often faster than an index scan for large result sets because it uses sequential I/O.
- **Beginner-Friendly Explanation:** Sorting a million rows is slow. But if the data is already stored in sorted order (via an index), the database can just read it out in order — much faster.

**Purposes:**

- To understand when sorting will be slow
- To design indexes that support ORDER BY
- To write queries that avoid unnecessary sorts

**Key Performance Guidelines:**

| Scenario | Recommendation |
|----------|----------------|
| ORDER BY on indexed columns | Database can use index scan (no sort) |
| ORDER BY on non-indexed columns | Filesort (MySQL) or explicit sort (PostgreSQL) |
| ORDER BY with LIMIT | "Top N" sort — database only needs the top N rows |
| Large result set, sorted | Consider adding an index that matches the sort |

**Annotated Code Examples:**

```sql
-- Create an index that supports the ORDER BY
CREATE INDEX idx_emp_salary ON employees(salary);

-- This query can use the index to avoid sorting
SELECT * FROM employees ORDER BY salary;
```

**Expected Performance:**

| Query | Rows | With Index | Without Index |
|-------|------|------------|---------------|
| `ORDER BY salary` | 1M | 50ms | 2s |

**Index Usage Rules:**

- The index must match the ORDER BY column(s) and direction(s).
- If the query mixes ASC and DESC, the index must also have mixed directions.
- If the index doesn't cover all selected columns, the database may choose a table scan instead of an index scan.

**Real-World Cases:**

- **E-commerce:** Sorting products by price — an index on price makes this fast.
- **Log analysis:** Sorting by timestamp — an index on timestamp enables fast range queries.
- **Reporting:** Sorting by multiple columns — a composite index can cover the sort.

**References:**

- MySQL: ORDER BY Optimization - https://dev.mysql.com/doc/refman/8.0/en/order-by-optimization.html
- PostgreSQL: Indexes and ORDER BY - https://www.postgresql.org/docs/17/indexes-ordering.html


## Summary Tables

### ORDER BY Clause Elements

| Element | Syntax | Description |
|---------|--------|-------------|
| Column name | `ORDER BY last_name` | Sort by a table column |
| Column alias | `ORDER BY annual_salary` | Sort by a SELECT alias |
| Positional number | `ORDER BY 1, 2` | Sort by SELECT list position |
| Expression | `ORDER BY salary * 12` | Sort by a computed value |
| CASE expression | `ORDER BY CASE WHEN ... END` | Conditional sorting |
| Direction | `ASC` / `DESC` | Ascending (default) or descending |
| NULL ordering | `NULLS FIRST` / `NULLS LAST` | Control NULL placement |

### NULL Ordering Defaults by Database

| Database | ASC Default | DESC Default | Explicit Control |
|----------|-------------|--------------|------------------|
| PostgreSQL | NULLS LAST | NULLS FIRST | ✓ |
| Oracle | NULLS LAST | NULLS FIRST | ✓ |
| SQL Server | NULLS FIRST | NULLS LAST | ✗ (workaround with CASE) |
| MySQL | NULLS FIRST | NULLS LAST | ✗ (workaround with CASE) |

### Index Usage for ORDER BY

| ORDER BY Pattern | Index Usage | Notes |
|------------------|-------------|-------|
| `ORDER BY indexed_col` | ✓ | Direct index scan |
| `ORDER BY indexed_col ASC, indexed_col2 ASC` | ✓ | Composite index |
| `ORDER BY indexed_col ASC, indexed_col2 DESC` | ✓ (with matching index) | Requires mixed-direction index |
| `ORDER BY non_indexed_col` | ✗ | Filesort / explicit sort |
| `ORDER BY expression` | ✗ | Unless functional index exists |

---

## References

- PostgreSQL: Sorting Rows (ORDER BY) - https://www.postgresql.org/docs/17/queries-order.html
- PostgreSQL: Indexes and ORDER BY - https://www.postgresql.org/docs/17/indexes-ordering.html
- PostgreSQL: SELECT - https://www.postgresql.org/docs/current/sql-select.html
- Microsoft Learn: Sort your results - https://learn.microsoft.com/en-us/training/modules/sort-filter-queries/2-sort-your-data-as-its-returned
- Microsoft Learn: ORDER BY Clause (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-order-by-clause-transact-sql
- Microsoft Learn: Collation and Unicode Support - https://learn.microsoft.com/en-us/sql/relational-databases/collations/collation-and-unicode-support
- MySQL: ORDER BY Optimization - https://dev.mysql.com/doc/refman/8.0/en/order-by-optimization.html
- Oracle: SQL Beginner's Guide - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SELECT.html
- Oracle: Database Globalization Support Guide - https://docs.oracle.com/en/database/oracle/oracle-database/19/nlspg/
- Stack Overflow: DISTINCT and ORDER BY ANSI constraint - https://stackoverflow.com/revisions/6b7be6fa-3c81-4b4d-9b5d-b094102331a7/view-source