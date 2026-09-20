# SQL Common Table Expressions (CTEs): A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A Common Table Expression (CTE) is a named temporary result set that exists only within the execution scope of a single SQL statement and can be referenced multiple times within that statement.

**Technical Definition:** Per the ANSI SQL-99 standard, a CTE is defined using the `WITH` clause, which precedes a `SELECT`, `INSERT`, `UPDATE`, or `DELETE` statement. A CTE acts as a named subquery—a "temporary view"—whose result set is scoped to the single statement in which it is defined. CTEs support both non-recursive and recursive forms, the latter enabling self-referential queries for hierarchical and graph data traversal.

**Beginner-Friendly Explanation:** A CTE is like giving a name to a subquery so you can use it later in the same query. Instead of nesting subqueries inside each other (which becomes hard to read), you define them at the top with `WITH`, give each a name, and then reference them by name. Think of it as creating a temporary "view" that lives only for the duration of one query.

### Key Characteristics

- **Single-statement scope:** A CTE exists only for the duration of the query it is attached to. When the outer query ends, the CTE's lifetime ends.
- **Multiple references:** Unlike derived tables, a CTE can be referenced multiple times in the same query with only one definition.
- **Composable:** Multiple CTEs can be defined in a single `WITH` clause, separated by commas.
- **Chainable:** A CTE can reference other CTEs defined earlier in the same `WITH` clause.
- **Recursion support:** CTEs can reference themselves, enabling recursive queries for hierarchical data.
- **Readability tool:** CTEs break down complex queries into simpler, more modular units.

### Prerequisites

- Basic SQL syntax (`SELECT`, `FROM`, `WHERE`, `GROUP BY`, `ORDER BY`)
- Understanding of subqueries and derived tables
- Familiarity with JOIN operations
- Basic knowledge of aggregate functions

### Related Programming Areas

- **Query Optimization:** CTEs affect how the query optimizer plans execution; materialization strategies can dramatically impact performance.
- **Data Warehousing and ETL:** CTEs simplify complex transformation pipelines.
- **Reporting and Business Intelligence:** CTEs make multi-step reporting queries maintainable.
- **Graph and Network Analysis:** Recursive CTEs enable traversal of tree and graph structures.
- **Application Development:** CTEs are used extensively in ORM-generated queries and stored procedures.

---

## 1. Non-Recursive (Standard) CTEs

### Core Definitions

- **Core Definition:** A non-recursive CTE is a named temporary result set defined in a `WITH` clause that does not reference itself, used to simplify complex queries.
- **Technical Definition:** An ordinary (non-recursive) CTE is defined by the syntax `WITH cte_name AS (subquery)`, where the subquery is a standard `SELECT` statement that does not reference the CTE name. It behaves like a view that exists for the duration of a single SQL statement.
- **Beginner-Friendly Explanation:** A non-recursive CTE is just a way to give a name to a subquery so you can reuse it and keep your query organized.

### Purposes

- To break down complicated queries into simpler, more readable parts
- To avoid repeating the same subquery multiple times
- To improve query maintainability by naming logical data sets
- To enable modular query construction where each CTE represents one logical step

### Sub-feature 1.1: The WITH Clause

**Core Definitions:**

- **Core Definition:** The `WITH` clause is the SQL construct that declares one or more CTEs before the main query body.
- **Technical Definition:** The `WITH` clause attaches auxiliary statements to a primary statement. Each auxiliary statement can be a `SELECT`, `INSERT`, `UPDATE`, or `DELETE`, and the `WITH` clause itself is attached to a primary statement that can also be any of these types.
- **Beginner-Friendly Explanation:** The `WITH` clause is like a "setup" section at the top of your query where you define all the temporary tables you'll need.

**Purposes:**

- To declare temporary named result sets that exist only within the execution scope of a single query
- To organize query logic from the top down
- To enable reuse of subquery logic without repetition

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
WITH cte_name [(column1, column2, ...)] AS (
    SELECT column1, column2, ...
    FROM table_name
    WHERE condition
)
SELECT *
FROM cte_name
WHERE condition;
```

**Component Breakdown:**

- `WITH`: Keyword that begins the CTE declaration
- `cte_name`: The name assigned to the temporary result set
- `(column1, column2, ...)`: Optional explicit column name list
- `AS (subquery)`: The query that defines the CTE's result set
- Outer query: The main statement that references the CTE

**Syntax Rules:**

- The CTE name must be a valid identifier.
- The subquery in parentheses must be a complete `SELECT` statement.
- If a column list is provided, the number of names must match the number of columns in the result set.
- If no column list is provided, column names come from the select list of the first `SELECT` within the CTE.
- A `WITH` clause can be used at the beginning of `SELECT`, `UPDATE`, and `DELETE` statements, as well as before `SELECT` in `INSERT`, `REPLACE`, `CREATE TABLE`, and `CREATE VIEW` statements.

**Constraints and Limitations:**

- When a CTE follows another statement in the same batch, the preceding statement must end with a semicolon (`;`).
- Only one `WITH` clause is permitted at the same level; multiple CTEs go inside a single `WITH` clause.
- CTE names cannot be the same as any table name referenced in the same query (implementation-dependent).
- CTE scope is limited to the statement it is attached to.

**Annotated Code Examples:**

Here are three complete, executable SQL examples using the WITH clause (Common Table Expressions or CTEs).
The WITH clause creates a temporary result set that you can reference within a SELECT, INSERT, UPDATE, or DELETE statement. It improves readability and allows you to break down complex queries into smaller, modular building blocks.

**Setup Guide (Run This First)**
To run these examples, copy and paste this setup script into your SQL environment (PostgreSQL, MySQL 8.0+, SQL Server, or Oracle). It creates an employees table and populates it with sample data.
```sql
-- Step 1: Create the sample table
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    department VARCHAR(50),
    salary DECIMAL(10, 2),
    manager_id INT
);

-- Step 2: Insert sample dataINSERT INTO employees VALUES
(1, 'Alice', 'Smith', 'HR', 60000.00, NULL),
(2, 'Bob', 'Jones', 'IT', 95000.00, NULL),
(3, 'Charlie', 'Brown', 'IT', 80000.00, 2),
(4, 'David', 'Miller', 'IT', 110000.00, 2),
(5, 'Emma', 'Davis', 'Sales', 70000.00, NULL),
(6, 'Frank', 'Wilson', 'Sales', 90000.00, 5);
```

**Example 1: Basic CTE (Simplifying Aggregations)**
```sql
-- Define the CTE named 'dept_averages'
WITH dept_averages AS (
    -- This internal query runs first to calculate department averages
    SELECT 
        department, 
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)

-- Main query: Reference the CTE just like a standard table
SELECT 
    department, 
    ROUND(avg_salary, 2) AS formatted_avg_salary
    FROM dept_averages-- Filter using the pre-calculated aggregate from the CTE
    WHERE avg_salary > 75000.00;
```
**Expected Output**

| department | formatted_avg_salary |
|---|---|
| IT | 95000.00 |

**Why It Works**
   1. CTE Execution: The dept_averages CTE groups the 6 employees by their departments and calculates the average: HR ($60,000), IT ($95,000), and Sales ($80,000).
   2. Main Query Filtering: The main SELECT filters the temporary dept_averages table, leaving only the IT department because its average ($95,000) is greater than the $75,000 threshold.

**Example 2: Multiple CTEs (Combining Data Steps)**
```sql
-- Define the first CTE to get payroll totals per department
WITH dept_payroll AS (
    SELECT department, SUM(salary) AS dept_total
    FROM employees
    GROUP BY department
),
-- Define a second CTE (separated by a comma) to get the global company payroll
company_payroll AS (
    SELECT SUM(salary) AS total_company_cost
    FROM employees
)

-- Main query: Join and use both CTEs to find percentage contribution
SELECT 
    d.department,
    d.dept_total,
    -- Calculate what percentage of total company payroll goes to this department
    ROUND((d.dept_total / c.total_company_cost) * 100, 2) AS pct_of_company_cost
    FROM dept_payroll d
    CROSS JOIN company_payroll c -- Cross join lets us append the single total cost row to every dept row
    ORDER BY pct_of_company_cost DESC;
```
**Expected Output**

| department | dept_total | pct_of_company_cost |
|---|---|---|
| IT | 285000.00 | 56.44 |
| Sales | 160000.00 | 31.68 |
| HR | 60000.00 | 11.88 |

**Why It Works**

   1. Step-by-step Processing: dept_payroll summarizes costs by department (IT: $285k, Sales: $160k, HR: $60k). company_payroll calculates the grand total ($505k).
   2. Cross Joining: Because company_payroll returns exactly one row, a CROSS JOIN cleanly appends that single grand total to every single row in the department breakdown, allowing for easy percentage division.

------------------------------
**Example 3: Recursive CTE (Hierarchical Data)**
```sql
-- Define a recursive CTE named 'org_hierarchy'
WITH RECURSIVE org_hierarchy AS (
    -- Anchor Member: Find top-level managers (where manager_id IS NULL)
    SELECT 
        employee_id, 
        first_name, 
        manager_id, 
        1 AS org_level -- Top level is designated as 1
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive Member: Join the anchor results back to the original table
    SELECT 
        e.employee_id, 
        e.first_name, 
        e.manager_id, 
        h.org_level + 1 AS org_level -- Increment the level for each reporting layer down
    FROM employees e
    INNER JOIN org_hierarchy h ON e.manager_id = h.employee_id -- Link sub-employees to their managers
)
-- Main query: Output the complete built hierarchy
SELECT org_level, first_name, manager_id
FROM org_hierarchyORDER 
BY org_level, manager_id;
```

**Expected Output**

| org_level | first_name | manager_id |
|---|---|---|
| 1 | Alice | null |
| 1 | Bob | null |
| 1 | Emma | null |
| 2 | Charlie | 2 |
| 2 | David | 2 |
| 2 | Frank | 5 |

**Why It Works**
   1. The Anchor Base: The query first finds Alice, Bob, and Emma because they have no managers. They are assigned org_level = 1.
   2. The Recursion Loop: The query loops through the remaining records to find who reports to Alice, Bob, or Emma. It finds Charlie and David (reporting to Bob) and Frank (reporting to Emma). They are assigned 1 + 1 = 2 for their org_level.
   3. Termination: The loop runs again, checking if anyone reports to Charlie, David, or Frank. Finding no matches, the recursion naturally stops and compiles the final list.


**Real-World Cases:**

- **Financial reporting:** A CTE can compute monthly totals, and the main query can filter to top-performing months.
- **E-commerce:** A CTE can identify high-value customers, and the main query can retrieve their recent orders.
- **HR analytics:** A CTE can calculate average department salaries, and the main query can find departments above the company average.

**References:**

- PostgreSQL Documentation: WITH Queries (Common Table Expressions) - https://www.postgresql.org/docs/current/queries-with.html
- MySQL Reference Manual: WITH (Common Table Expressions) - https://dev.mysql.com/doc/refman/8.0/en/with.html
- Microsoft Learn: WITH common_table_expression (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql


### Sub-feature 1.2: CTEs vs. Subqueries / Views

**Core Definitions:**

- **Core Definition:** CTEs, subqueries, and views are all mechanisms for encapsulating query logic, but they differ in scope, reusability, and readability.
- **Technical Definition:** A subquery is a `SELECT` statement nested inside another statement. A derived table is a subquery used in the `FROM` clause. A view is a named, persistent query stored in the database schema. A CTE is a named, temporary result set scoped to a single statement.
- **Beginner-Friendly Explanation:** A subquery is like a note tucked inside a larger document. A view is like a published article anyone can reference. A CTE is like a scratchpad note you write at the top of your page and use throughout that page only.

**Purposes:**

- To choose the right tool for query readability and maintainability
- To understand performance implications of each approach
- To decide when temporary (CTE) vs. persistent (view) abstraction is appropriate

**Syntax Structures and Rules:**

```sql
-- Subquery (inline derived table)
SELECT * FROM (
    SELECT column1, column2 FROM table_name
) AS subquery_name;

-- CTE
WITH cte_name AS (
    SELECT column1, column2 FROM table_name
)
SELECT * FROM cte_name;

-- View (persistent)
CREATE VIEW view_name AS
SELECT column1, column2 FROM table_name;

SELECT * FROM view_name;
```

**Key Differences:**

| Feature | CTE | Subquery / Derived Table | View |
|---------|-----|-------------------------|------|
| Scope | Single statement | Single location | Persistent |
| Reusable | Yes, multiple references | No, single use | Yes, across statements |
| Recursive | Yes | No | No |
| Performance | Same as subquery (theory) | Baseline | Can be indexed |
| Readability | High | Low for nested | Medium |

**Constraints and Limitations:**

- CTEs and subqueries offer essentially the same performance in theory because they provide the same information to the query optimizer.
- Views can be indexed (materialized views), which can provide performance advantages.
- CTEs cannot be indexed directly (except through materialization hints in PostgreSQL).
- Subqueries cannot be referenced multiple times without duplication.

**Annotated Code Examples:**

```sql
-- Using a subquery (derived table)
SELECT dept_name, avg_salary
FROM (
    SELECT d.dept_name, AVG(e.salary) AS avg_salary
    FROM employees e
    JOIN departments d ON e.dept_id = d.dept_id
    GROUP BY d.dept_name
) AS dept_avg
WHERE avg_salary > 50000;

-- Using a CTE (equivalent but more readable)
WITH dept_avg AS (
    SELECT d.dept_name, AVG(e.salary) AS avg_salary
    FROM employees e
    JOIN departments d ON e.dept_id = d.dept_id
    GROUP BY d.dept_name
)
SELECT dept_name, avg_salary
FROM dept_avg
WHERE avg_salary > 50000;
```

**Expected Output (both queries):**

| dept_name | avg_salary |
|-----------|------------|
| Engineering | 75000    |
| Sales     | 62000      |

**Explanation:** Both queries produce identical results. The CTE version is easier to read because the logic is defined top-down rather than nested inside the `FROM` clause. The CTE can also be referenced multiple times in the outer query without redefining it.

**Real-World Cases:**

- **Ad-hoc analysis:** Use CTEs for one-off queries where readability matters.
- **Recurring reports:** Use views when the same logic is needed across many queries.
- **Complex joins:** Use CTEs to break a multi-join query into logical steps.

**References:**

- Stack Overflow: Performance difference between CTE, view, and subquery - https://stackoverflow.com/questions/2572051/is-there-any-performance-difference-btw-using-cte-view-and-subquery
- PostgreSQL Documentation: WITH Queries - https://www.postgresql.org/docs/current/queries-with.html


### Sub-feature 1.3: Column Aliasing in CTEs

**Core Definitions:**

- **Core Definition:** Column aliasing in CTEs is the practice of explicitly naming the output columns of a CTE, either in the CTE header or within the CTE's `SELECT` list.
- **Technical Definition:** A CTE's column names are determined by (1) a parenthesized column list following the CTE name, or (2) the select list of the first `SELECT` within the CTE if no explicit list is provided.
- **Beginner-Friendly Explanation:** You can name the columns of your CTE in two ways: in the header parentheses after the CTE name, or by using `AS` aliases inside the CTE's `SELECT`.

**Purposes:**

- To define output column headers explicitly inside the CTE declaration vector
- To resolve duplicate column names when joining multiple CTEs
- To provide meaningful names for computed expressions
- To ensure consistent column naming across the query

**Syntax Structures and Rules:**

```sql
-- Method 1: Explicit column list in CTE header
WITH cte_name (col1, col2, col3) AS (
    SELECT expression1, expression2, expression3
    FROM table_name
)
SELECT * FROM cte_name;

-- Method 2: Inline aliasing in the SELECT list
WITH cte_name AS (
    SELECT expression1 AS col1, expression2 AS col2
    FROM table_name
)
SELECT * FROM cte_name;
```

**Syntax Rules:**

- If a parenthesized list of names follows the CTE name, those names are the column names. The number of names must match the number of columns in the result set.
- If no column list is provided, column names come from the select list of the first `SELECT` within the CTE.
- Column names in the CTE result set must be unique—either because the underlying `SELECT` columns already have distinct names, or by supplying column aliases.

**Constraints and Limitations:**

- Explicit column lists are mandatory in some database systems (Oracle is the common case) when using recursive CTEs.
- If the underlying `SELECT` uses `*`, the CTE inherits all column names from the source table.

**Annotated Code Examples:**

```sql
-- Example 1: Explicit column list
WITH CTE_year (orderyear, custid) AS (
    SELECT YEAR(orderdate), custid
    FROM Sales.Orders
)
SELECT orderyear, COUNT(DISTINCT custid) AS cust_count
FROM CTE_year
GROUP BY orderyear;
```

**Expected Output:**

| orderyear | cust_count |
|-----------|------------|
| 2019      | 67         |
| 2020      | 86         |

**Explanation:** The column list `(orderyear, custid)` names the two columns produced by the CTE. Without this list, the columns would be unnamed expressions, and the outer query could not reference them by name.

```sql
-- Example 2: Inline aliasing
WITH cte (col1, col2) AS (
    SELECT 1, 2
    UNION ALL
    SELECT 3, 4
)
SELECT col1, col2 FROM cte;
```

**Expected Output:**

| col1 | col2 |
|------|------|
| 1    | 2    |
| 3    | 4    |

**Explanation:** Here, the explicit column list defines `col1` and `col2` as the column names, overriding the unnamed expressions in the `SELECT`. This is equivalent to writing `SELECT 1 AS col1, 2 AS col2` inside the CTE.

**Real-World Cases:**

- **Multi-CTE joins:** When joining two CTEs that both have a column named `id`, explicit aliasing prevents ambiguity.
- **Reporting:** Computed columns like `SUM(amount)` benefit from meaningful aliases like `total_sales`.
- **Recursive CTEs:** Oracle requires explicit column lists for recursive CTEs, making this a mandatory practice on that platform.

**References:**

- MySQL Reference Manual: WITH (Common Table Expressions) - https://dev.mysql.com/doc/refman/8.0/en/with.html
- Microsoft Learn: Use Common Table Expressions - https://learn.microsoft.com/en-us/training/modules/create-tables-views-temporary-objects/5-use-common-table-expressions


## 2. Architectural Composition & Chaining

### Core Definitions

- **Core Definition:** Architectural composition refers to the practice of defining multiple CTEs in a single `WITH` clause and chaining them together so that downstream CTEs reference upstream CTEs.
- **Technical Definition:** A single `WITH` clause can contain one or more comma-separated CTE definitions. A CTE can reference any CTE defined earlier in the same `WITH` clause, enabling the construction of multi-step data transformation pipelines.
- **Beginner-Friendly Explanation:** You can define several CTEs in one `WITH` clause, and later CTEs can use the results of earlier ones—like a chain of steps where each step builds on the previous one.

### Purposes

- To separate distinct data definitions using a single `WITH` clause followed by commas
- To construct pipeline architectures where a downstream CTE queries an upstream CTE
- To leverage CTEs inside data modification blocks (`WITH ... INSERT`, `UPDATE`, or `DELETE`)
- To improve query modularity by assigning one logical transformation per CTE

### Sub-feature 2.1: Multiple CTEs

**Core Definitions:**

- **Core Definition:** Multiple CTEs are two or more named temporary result sets defined in a single `WITH` clause, separated by commas.
- **Technical Definition:** A `WITH` clause can contain one or more comma-separated subclauses, each providing a subquery that produces a result set and associates a name with that subquery.
- **Beginner-Friendly Explanation:** You can define several CTEs at once by separating them with commas. All of them are available in the main query.

**Purposes:**

- To define multiple distinct data sets in one query
- To join several CTEs together in the main query
- To keep related logic organized in a single `WITH` clause

**Syntax Structures and Rules:**

```sql
WITH cte1 AS (
    SELECT a, b FROM table1
),
cte2 AS (
    SELECT c, d FROM table2
)
SELECT b, d
FROM cte1
JOIN cte2 ON cte1.a = cte2.c;
```

**Syntax Rules:**

- CTEs are separated by commas.
- Each CTE has its own name and subquery.
- All CTEs are defined in a single `WITH` clause.
- The main query can reference any or all of the CTEs.

**Constraints and Limitations:**

- The comma after each CTE definition (except the last) is mandatory.
- CTE names must be unique within the same `WITH` clause.

**Annotated Code Examples:**

```sql
-- Multiple CTEs: sales data and quota data
WITH Sales_CTE (SalesPersonID, TotalSales, SalesYear) AS (
    -- First CTE: total sales per salesperson per year
    SELECT SalesPersonID, SUM(TotalDue) AS TotalSales, YEAR(OrderDate) AS SalesYear
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID IS NOT NULL
    GROUP BY SalesPersonID, YEAR(OrderDate)
),
Sales_Quota_CTE (BusinessEntityID, SalesQuota, SalesQuotaYear) AS (
    -- Second CTE: sales quota per salesperson per year
    SELECT BusinessEntityID, SUM(SalesQuota) AS SalesQuota, YEAR(QuotaDate) AS SalesQuotaYear
    FROM Sales.SalesPersonQuotaHistory
    GROUP BY BusinessEntityID, YEAR(QuotaDate)
)
-- Main query joins both CTEs
SELECT 
    SalesPersonID,
    SalesYear,
    TotalSales,
    SalesQuota,
    TotalSales - SalesQuota AS Amt_Above_or_Below_Quota
FROM Sales_CTE
INNER JOIN Sales_Quota_CTE 
    ON Sales_Quota_CTE.BusinessEntityID = Sales_CTE.SalesPersonID
    AND Sales_CTE.SalesYear = Sales_Quota_CTE.SalesQuotaYear
ORDER BY SalesPersonID, SalesYear;
```

**Expected Output (sample):**

| SalesPersonID | SalesYear | TotalSales | SalesQuota | Amt_Above_or_Below_Quota |
|---------------|-----------|------------|------------|--------------------------|
| 274           | 2005      | 32567.92   | 35000.00   | -2432.08                 |
| 274           | 2006      | 406620.07  | 455000.00  | -48379.93                |
| 274           | 2007      | 515622.91  | 544000.00  | -28377.09                |

**Explanation:** Two independent CTEs are defined in one `WITH` clause. `Sales_CTE` computes actual sales, while `Sales_Quota_CTE` computes quota targets. The main query joins them on salesperson ID and year to compare actual vs. quota. This structure would be far more complex if written with nested subqueries.

**Real-World Cases:**

- **Sales performance:** Comparing actual sales against quotas.
- **Inventory management:** Joining current stock levels with reorder thresholds.
- **Financial analysis:** Comparing actual expenses against budgets.

**References:**

- MySQL Reference Manual: WITH (Common Table Expressions) - https://dev.mysql.com/doc/refman/8.0/en/with.html
- Microsoft Learn: WITH common_table_expression (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql


### Sub-feature 2.2: CTE Chaining

**Core Definitions:**

- **Core Definition:** CTE chaining is the practice of having one CTE reference another CTE that was defined earlier in the same `WITH` clause, creating a pipeline of transformations.
- **Technical Definition:** A CTE name can be referenced in other CTEs defined later in the same `WITH` clause, enabling CTEs to be defined based on other CTEs. This creates a dependency chain where each CTE builds on the results of previous ones.
- **Beginner-Friendly Explanation:** CTE chaining is like an assembly line: the first CTE produces something, the second CTE uses that result to produce something else, and so on.

**Purposes:**

- To construct pipeline architectures where a downstream CTE queries an upstream CTE
- To break complex transformations into sequential, understandable steps
- To enable intermediate results to be reused without duplicating logic

**Syntax Structures and Rules:**

```sql
WITH cte_a AS (
    SELECT columns FROM table1
),
cte_b AS (
    -- cte_b references cte_a
    SELECT columns FROM cte_a WHERE condition
),
cte_c AS (
    -- cte_c references cte_b
    SELECT columns FROM cte_b JOIN table2 ON ...
)
SELECT * FROM cte_c;
```

**Syntax Rules:**

- A CTE can only reference CTEs defined before it in the same `WITH` clause.
- Forward references are not allowed (cte_a cannot reference cte_b if cte_b is defined later).
- The chain can be arbitrarily long.

**Constraints and Limitations:**

- Circular references are not allowed in non-recursive CTEs.
- Each CTE in the chain is evaluated in order of dependency.

**Annotated Code Examples:**

```sql
-- CTE chaining: step-by-step data transformation
WITH raw_data AS (
    -- Step 1: Extract raw data
    SELECT order_id, customer_id, order_date, amount
    FROM orders
    WHERE order_date >= '2024-01-01'
),
cleaned_data AS (
    -- Step 2: Clean and filter (references raw_data)
    SELECT order_id, customer_id, order_date, amount
    FROM raw_data
    WHERE amount > 0 AND customer_id IS NOT NULL
),
customer_totals AS (
    -- Step 3: Aggregate (references cleaned_data)
    SELECT customer_id, SUM(amount) AS total_spent, COUNT(*) AS order_count
    FROM cleaned_data
    GROUP BY customer_id
),
top_customers AS (
    -- Step 4: Filter top customers (references customer_totals)
    SELECT customer_id, total_spent, order_count
    FROM customer_totals
    WHERE total_spent > 10000
)
-- Final query
SELECT * FROM top_customers ORDER BY total_spent DESC;
```

**Expected Output (sample):**

| customer_id | total_spent | order_count |
|-------------|-------------|-------------|
| 101         | 25000.00    | 15          |
| 204         | 18750.00    | 12          |
| 309         | 12300.00    | 8           |

**Explanation:** Each CTE builds on the previous one. `raw_data` extracts orders. `cleaned_data` filters invalid records. `customer_totals` aggregates per customer. `top_customers` filters to high-value customers. This pipeline is easy to read and debug because each step is isolated.

**Real-World Cases:**

- **ETL pipelines:** Extract, transform, and load data through a chain of CTEs.
- **Financial analysis:** Step-by-step calculation of financial metrics.
- **Customer segmentation:** Progressive filtering and aggregation of customer data.

**References:**

- Alibaba Cloud: WITH Clause - https://www.alibabacloud.com/help/en/analyticdb/analyticdb-for-postgresql/developer-reference/common-table-expressions
- Microsoft Learn: Use Common Table Expressions - https://learn.microsoft.com/en-us/training/modules/create-tables-views-temporary-objects/5-use-common-table-expressions


### Sub-feature 2.3: Multi-Statement Interactions (CTEs in INSERT, UPDATE, DELETE)

**Core Definitions:**

- **Core Definition:** CTEs can be used as part of data modification statements (`INSERT`, `UPDATE`, `DELETE`), not just `SELECT` queries.
- **Technical Definition:** The `WITH` clause can precede an `INSERT`, `UPDATE`, or `DELETE` statement, and the CTE can be referenced within that statement. This enables complex data modifications based on multi-step query logic.
- **Beginner-Friendly Explanation:** You can use CTEs not just for reading data but also for writing data—inserting, updating, or deleting rows based on a CTE's result.

**Purposes:**

- To leverage CTEs inside data modification blocks (`WITH ... INSERT`, `UPDATE`, or `DELETE`)
- To perform complex updates based on aggregated or transformed data
- To delete rows identified through multi-step logic

**Syntax Structures and Rules:**

```sql
-- CTE with INSERT
WITH cte AS (
    SELECT columns FROM source_table WHERE condition
)
INSERT INTO target_table (columns)
SELECT columns FROM cte;

-- CTE with UPDATE
WITH cte AS (
    SELECT id, new_value FROM source_table
)
UPDATE target_table
SET column = cte.new_value
FROM cte
WHERE target_table.id = cte.id;

-- CTE with DELETE
WITH cte AS (
    SELECT id FROM table WHERE condition
)
DELETE FROM table
WHERE id IN (SELECT id FROM cte);
```

**Syntax Rules:**

- The `WITH` clause precedes the data modification statement.
- The CTE can be referenced in the `INSERT`, `UPDATE`, or `DELETE` statement.
- For `UPDATE`, the syntax varies by database (e.g., SQL Server uses `UPDATE ... FROM cte`, while PostgreSQL uses `UPDATE ... FROM cte` or a subquery).

**Constraints and Limitations:**

- Not all databases support CTEs with all DML statements. MySQL supports `WITH ... INSERT`, `WITH ... UPDATE`, and `WITH ... DELETE`.
- SQL Server supports CTEs with `INSERT`, `UPDATE`, and `DELETE`.
- Oracle supports CTEs with `INSERT` and `UPDATE` but has restrictions on `DELETE`.

**Annotated Code Examples:**

```sql
-- Example 1: Using a CTE with UPDATE to update hierarchical data
WITH Parts (AssemblyID, ComponentID, PerAssemblyQty, ComponentLevel) AS (
    SELECT b.ProductAssemblyID, b.ComponentID, b.PerAssemblyQty, 0 AS ComponentLevel
    FROM Production.BillOfMaterials AS b
    WHERE b.ProductAssemblyID = 800 AND b.EndDate IS NULL

    UNION ALL
    
    SELECT bom.ProductAssemblyID, bom.ComponentID, p.PerAssemblyQty, ComponentLevel + 1
    FROM Production.BillOfMaterials AS bom
    INNER JOIN Parts AS p ON bom.ProductAssemblyID = p.ComponentID
    WHERE bom.EndDate IS NULL
)

UPDATE Parts
SET PerAssemblyQty = PerAssemblyQty * 1.1
WHERE ComponentLevel > 0;
```

**Expected Output:** The `PerAssemblyQty` for all components at levels greater than 0 is increased by 10%.

**Explanation:** The recursive CTE identifies all parts in a bill of materials. The `UPDATE` statement then modifies only those rows returned by the CTE. This is a powerful pattern for bulk updates based on hierarchical relationships.

```sql
-- Example 2: Using a CTE with DELETE
WITH inactive_customers AS (
    SELECT customer_id
    FROM customers
    WHERE last_order_date < '2020-01-01'
)

DELETE FROM customers
WHERE customer_id IN (SELECT customer_id FROM inactive_customers);
```

**Expected Output:** All customers whose last order was before 2020 are deleted.

**Explanation:** The CTE identifies inactive customers, and the `DELETE` statement removes them. This is cleaner than writing the subquery inline, especially if the identification logic is complex.

**Real-World Cases:**

- **Data archiving:** Deleting old records identified by a CTE.
- **Bulk updates:** Updating prices or quantities based on a multi-step calculation.
- **Data migration:** Inserting transformed data into a new table.

**References:**

- PostgreSQL Documentation: WITH Queries - https://www.postgresql.org/docs/current/queries-with.html
- Microsoft Learn: WITH common_table_expression (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql


## 3. Recursive CTEs & Network Structures

### Core Definitions

- **Core Definition:** A recursive CTE is a CTE that references itself, enabling iterative processing of hierarchical or graph-structured data.
- **Technical Definition:** A recursive CTE is defined using `WITH RECURSIVE` (or just `WITH` in SQL Server/Oracle) and consists of an anchor member (non-recursive initial query) and a recursive member (query that references the CTE itself), joined by `UNION ALL` or `UNION`. Recursion continues until the recursive member returns no rows.
- **Beginner-Friendly Explanation:** A recursive CTE is a query that calls itself. It starts with a base result, then keeps applying the same logic to the new rows until no more rows are found. This is perfect for tree structures like org charts.

### Purposes

- To traverse hierarchical data (org charts, bill of materials, category trees)
- To process graph structures (flight networks, social networks)
- To generate sequences and series
- To answer questions like "who reports to whom" or "what parts make up this product"

### Sub-feature 3.1: The Anchor Member

**Core Definitions:**

- **Core Definition:** The anchor member is the initial, non-recursive query in a recursive CTE that establishes the starting result set for the recursion.
- **Technical Definition:** The anchor member is one or more `SELECT` statements that do not reference the CTE itself. It forms the base result set (T0) upon which the recursive member operates. All anchor-member query definitions must be positioned before the first recursive member definition.
- **Beginner-Friendly Explanation:** The anchor member is the starting point of the recursion—like the root of a tree or the first row in a sequence.

**Purposes:**

- To provide the starting dataset for recursion
- To establish the initial rows that the recursive member will expand
- To define the "root" or "seed" of the hierarchy

**Syntax Structures and Rules:**

```sql
WITH RECURSIVE cte_name AS (
    -- Anchor member (non-recursive)
    SELECT columns
    FROM table_name
    WHERE condition  -- e.g., manager_id IS NULL
    UNION ALL
    -- Recursive member follows
    ...
)
```

**Syntax Rules:**

- The anchor member must not reference the CTE itself.
- There must be at least one anchor member.
- Anchor members must come before recursive members.
- Anchor members are joined to recursive members with `UNION ALL` (or `UNION`).

**Annotated Code Examples:**

```sql
-- Anchor member: starting with the CEO (no manager)
WITH RECURSIVE org_chart AS (
    -- Anchor: CEO has NULL manager_id
    SELECT employee_id, name, manager_id, title, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    -- Recursive member follows
    ...
)
```

**Expected Output (anchor only):**

| employee_id | name | manager_id | title | level |
|-------------|------|------------|-------|-------|
| 1           | Alice Chen | NULL | CEO | 1 |

**Explanation:** The anchor member selects the CEO—the employee with no manager. This is the root of the organizational hierarchy. The recursion will then expand to direct reports, then their reports, and so on.

**References:**

- Microsoft Learn: Recursive Queries Using Common Table Expressions - https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql
- PostgreSQL Documentation: WITH Queries - https://www.postgresql.org/docs/current/queries-with.html


### Sub-feature 3.2: The Recursive Member

**Core Definitions:**

- **Core Definition:** The recursive member is the part of a recursive CTE that references the CTE's own name, enabling the query to iterate over the data.
- **Technical Definition:** The recursive member is one or more `SELECT` statements that reference the CTE name in their `FROM` clause. It is joined to the anchor member (and any previous recursive members) with `UNION ALL` or `UNION`. The recursive member takes the previous iteration's output (Ti) as input and produces the next iteration's output (Ti+1).
- **Beginner-Friendly Explanation:** The recursive member is the "loop" part of the CTE. It says: "Take the rows from the last round and find the next level."

**Purposes:**

- To iterate over hierarchical data by joining back to the CTE
- To expand the result set level by level
- To enable traversal of tree and graph structures

**Syntax Structures and Rules:**

```sql
WITH RECURSIVE cte_name AS (
    -- Anchor member
    SELECT columns FROM table WHERE condition
    UNION ALL
    -- Recursive member: references cte_name
    SELECT columns
    FROM table_name
    INNER JOIN cte_name ON table_name.parent_id = cte_name.id
)
```

**Syntax Rules:**

- The recursive member must reference the CTE name exactly once in its `FROM` clause.
- The recursive member must be joined to the anchor member with `UNION ALL` or `UNION`.
- Recursive members may not use aggregate functions or window functions (in SQLite).
- In SQL Server, the recursive member must be joined with `UNION ALL` (not `UNION`).

**Constraints and Limitations:**

- Recursive CTEs can only reference the CTE once in the recursive member.
- The recursive member's column list must match the anchor member's column list.
- `UNION` (not `UNION ALL`) discards duplicates and can prevent infinite loops in cyclic data.

**Annotated Code Examples:**

```sql
-- Recursive member: finding direct reports
WITH RECURSIVE org_chart AS (
    -- Anchor: CEO
    SELECT employee_id, name, manager_id, title, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    -- Recursive: find direct reports of current level
    SELECT e.employee_id, e.name, e.manager_id, e.title, oc.level + 1
    FROM employees e
    INNER JOIN org_chart oc ON e.manager_id = oc.employee_id
)
SELECT * FROM org_chart ORDER BY level, name;
```

**Expected Output:**

| employee_id | name | manager_id | title | level |
|-------------|------|------------|-------|-------|
| 1           | Alice Chen | NULL | CEO | 1 |
| 2           | Bob Martinez | 1 | VP Eng | 2 |
| 3           | Carol White | 1 | VP Sales | 2 |
| 5           | Diana Ross | 2 | Eng Manager | 3 |

**Explanation:** The recursive member joins the `employees` table back to the `org_chart` CTE on `e.manager_id = oc.employee_id`. This finds all employees whose manager was in the previous level. The `level` column increments with each iteration. The recursion stops when no more rows are returned (i.e., no employee has a manager in the current level).

**References:**

- Microsoft Learn: Recursive Queries Using Common Table Expressions - https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql
- SQLite Documentation: The WITH Clause - https://www2.sqlite.org/matrix/lang_with.html


### Sub-feature 3.3: The Termination Condition

**Core Definitions:**

- **Core Definition:** The termination condition is the implicit or explicit mechanism that stops the recursion when no more rows satisfy the recursive member's join condition.
- **Technical Definition:** The termination check in a recursive CTE is implicit—recursion stops when the previous invocation of the recursive member returns no rows. This occurs when the join between the recursive reference and the source table produces an empty result set.
- **Beginner-Friendly Explanation:** The recursion stops when there's nothing new to add. If the recursive query returns zero rows, the loop ends.

**Purposes:**

- To prevent infinite recursion
- To define the structural point where the join fails
- To ensure the query completes in finite time

**Syntax Structures and Rules:**

- The termination condition is not written explicitly in the SQL; it is inherent in the join condition.
- If the recursive member's `WHERE` clause or join condition never produces zero rows, the recursion will continue indefinitely (or until a recursion limit is hit).
- In SQL Server, `MAXRECURSION` can be used to enforce a hard limit.

**Constraints and Limitations:**

- Cyclic data (e.g., A reports to B, B reports to A) can cause infinite loops if not handled with `UNION` (which discards duplicates) or path-tracking.
- Some databases have default recursion limits (e.g., SQL Server defaults to 100 levels; MySQL has `cte_max_recursion_depth`).

**Annotated Code Examples:**

```sql
-- Recursion terminates when no employee has a manager in the current level
WITH RECURSIVE org_chart AS (
    SELECT employee_id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.name, e.manager_id, oc.level + 1
    FROM employees e
    INNER JOIN org_chart oc ON e.manager_id = oc.employee_id
)
SELECT * FROM org_chart;
-- The query stops when the recursive member returns 0 rows
-- (i.e., no employee has a manager who is at the deepest level)
```

**Explanation:** The recursion continues as long as the `INNER JOIN` produces rows. When the deepest-level employees have no direct reports, the join produces zero rows, and the recursion terminates automatically.

**Real-World Cases:**

- **Org charts:** Recursion stops at leaf employees (those with no direct reports).
- **Bill of materials:** Recursion stops at raw materials (components with no sub-components).
- **Category trees:** Recursion stops at leaf categories.

**References:**

- Microsoft Learn: Recursive Queries Using Common Table Expressions - https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql
- SQLite Documentation: The WITH Clause - https://www2.sqlite.org/matrix/lang_with.html


### Sub-feature 3.4: Hierarchical and Graph Processing

**Core Definitions:**

- **Core Definition:** Recursive CTEs are used to traverse hierarchical structures (trees) and graph structures (networks) where relationships between rows form parent-child or arbitrary connections.
- **Technical Definition:** Hierarchical processing involves traversing a tree where each node has at most one parent. Graph processing involves traversing a network where nodes can have multiple connections in any direction. Recursive CTEs handle both by repeatedly joining the CTE's output back to the source table.
- **Beginner-Friendly Explanation:** Recursive CTEs let you answer questions like "who are all the descendants of this manager?" or "what are all the parts needed to build this product?" by following relationships level by level.

**Purposes:**

- To traverse complex organizational charts (employee-manager trees)
- To explode parts assemblies (bill of materials)
- To find paths in flight networks
- To traverse category trees, file systems, and social networks

**Syntax Structures and Rules:**

```sql
-- Hierarchical traversal (top-down)
WITH RECURSIVE cte_name AS (
    SELECT root_columns FROM table WHERE root_condition
    UNION ALL
    SELECT child_columns FROM table
    INNER JOIN cte_name ON table.parent_id = cte_name.id
)
SELECT * FROM cte_name;

-- Graph traversal (with path tracking to avoid cycles)
WITH RECURSIVE cte_name AS (
    SELECT id, path FROM table WHERE start_condition
    UNION ALL
    SELECT t.id, cte_name.path || ',' || t.id
    FROM table t
    INNER JOIN cte_name ON t.parent_id = cte_name.id
    WHERE cte_name.path NOT LIKE '%' || t.id || '%'
)
SELECT * FROM cte_name;
```

**Annotated Code Examples:**

```sql
-- Bill of Materials: find all components of a product
WITH RECURSIVE Parts (AssemblyID, ComponentID, PerAssemblyQty, ComponentLevel) AS (

    -- Anchor: top-level assembly
    SELECT b.ProductAssemblyID, b.ComponentID, b.PerAssemblyQty, 0 AS ComponentLevel
    FROM Production.BillOfMaterials AS b
    WHERE b.ProductAssemblyID = 800 AND b.EndDate IS NULL
    UNION ALL

    -- Recursive: find sub-components
    SELECT bom.ProductAssemblyID, bom.ComponentID, p.PerAssemblyQty, ComponentLevel + 1
    FROM Production.BillOfMaterials AS bom
    INNER JOIN Parts AS p ON bom.ProductAssemblyID = p.ComponentID
    WHERE bom.EndDate IS NULL
)

SELECT AssemblyID, ComponentID, Name, PerAssemblyQty, ComponentLevel
FROM Parts AS p
INNER JOIN Production.Product AS pr ON p.ComponentID = pr.ProductID
ORDER BY ComponentLevel, AssemblyID, ComponentID;
```

**Expected Output (sample):**

| AssemblyID | ComponentID | Name | PerAssemblyQty | ComponentLevel |
|------------|-------------|------|----------------|----------------|
| 800        | 801         | Frame | 1              | 0              |
| 801        | 802         | Tube  | 4              | 1              |
| 801        | 803         | Bolt  | 10             | 1              |

**Explanation:** The anchor member selects the top-level assembly (ID 800). The recursive member then finds all components whose `ProductAssemblyID` matches a component from the previous level. This continues until all sub-components are found. The `ComponentLevel` column tracks the depth.

```sql
-- Finding all ancestors (bottom-up traversal)
WITH RECURSIVE ancestors AS (
    -- Anchor: start with a specific employee
    SELECT employee_id, name, manager_id, 0 AS distance
    FROM employees
    WHERE employee_id = 42
    UNION ALL
    -- Recursive: find the manager of the current employee
    SELECT e.employee_id, e.name, e.manager_id, a.distance + 1
    FROM employees e
    INNER JOIN ancestors a ON e.employee_id = a.manager_id
)
SELECT * FROM ancestors ORDER BY distance;
```

**Expected Output:**

| employee_id | name | manager_id | distance |
|-------------|------|------------|----------|
| 42          | Diana | 5         | 0        |
| 5           | Bob   | 2         | 1        |
| 2           | Alice | NULL      | 2        |

**Explanation:** This bottom-up traversal starts with employee 42 and walks up the management chain to the CEO. The `distance` column tracks how many levels up the employee is from the starting point.

**Real-World Cases:**

- **Org charts:** Finding all employees under a manager, or all managers above an employee.
- **Bill of materials:** Exploding a product into all its raw components.
- **Flight networks:** Finding all connections from one city to another.
- **Category trees:** Navigating product categories.

**References:**

- BigQuery Recursive CTEs for Hierarchical Data - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-02-17-how-to-write-recursive-ctes-in-bigquery-for-hierarchical-data-traversal/README.md
- Microsoft Learn: Recursive Queries Using Common Table Expressions - https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql


## 4. Execution Safeguards & Performance

### Core Definitions

- **Core Definition:** Execution safeguards are mechanisms and dialect-specific keywords that control the behavior, safety, and performance of CTEs.
- **Technical Definition:** Different database engines implement recursive CTEs with varying syntax requirements (e.g., `WITH RECURSIVE` vs. plain `WITH`) and provide different mechanisms for preventing infinite recursion (e.g., `MAXRECURSION`, `cte_max_recursion_depth`) and controlling materialization (e.g., `MATERIALIZED`, `NOT MATERIALIZED`).
- **Beginner-Friendly Explanation:** Different databases have different rules for how CTEs work, especially recursive ones. Some need a special keyword; others don't. And you can sometimes tell the database how to handle the CTE for better performance.

### Purposes

- To ensure portability across database systems
- To prevent infinite loops in recursive queries
- To tune query performance through materialization hints
- To understand dialect-specific behaviors and constraints

### Sub-feature 4.1: Dialect Keyword Variations

**Core Definitions:**

- **Core Definition:** Dialect keyword variations refer to the differences in syntax required by different database systems for defining recursive CTEs.
- **Technical Definition:** PostgreSQL, MySQL, and SQLite require the `RECURSIVE` keyword explicitly (`WITH RECURSIVE`), while SQL Server and Oracle use plain `WITH` and infer recursion automatically. Oracle also requires an explicit column list for recursive CTEs.
- **Beginner-Friendly Explanation:** Some databases need you to write `WITH RECURSIVE` to use recursion; others figure it out on their own.

**Purposes:**

- To write portable SQL code that works across multiple database systems
- To understand dialect-specific requirements when migrating between databases
- To avoid syntax errors when switching database platforms

**Syntax Structures and Rules:**

| Database | Recursive Keyword | Column List Required | Cycle Detection |
|----------|-------------------|---------------------|-----------------|
| PostgreSQL | `WITH RECURSIVE` | No | Native `CYCLE` clause (14+) |
| MySQL 8+ | `WITH RECURSIVE` | No | Path tracking (`LOCATE`) |
| SQLite | `WITH RECURSIVE` | No | Path tracking (`INSTR`) |
| SQL Server | `WITH` (no RECURSIVE) | No | Path tracking (`CHARINDEX`) |
| Oracle 11g2+ | `WITH` (no RECURSIVE) | Yes (explicit) | Path tracking (`INSTR`) |

**Constraints and Limitations:**

- Code written with `WITH RECURSIVE` will fail on SQL Server and Oracle (which don't recognize the keyword in this context).
- Code written without `RECURSIVE` will fail on PostgreSQL, MySQL, and SQLite.
- Oracle requires an explicit column list for recursive CTEs, which other databases don't mandate.

**Annotated Code Examples:**

```sql
-- PostgreSQL / MySQL / SQLite syntax
WITH RECURSIVE series AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM series WHERE n < 10
)
SELECT * FROM series;

-- SQL Server / Oracle syntax
WITH series AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM series WHERE n < 10
)
SELECT * FROM series;
```

**Expected Output (both):**

| n |
|---|
| 1 |
| 2 |
| ... |
| 10 |

**Explanation:** PostgreSQL, MySQL, and SQLite require `WITH RECURSIVE`. SQL Server and Oracle use `WITH` alone. The logical result is identical, but the syntax differs. This is a critical portability consideration.

**References:**

- knex Issue #6335: Make WITH RECURSIVE behaviour more consistent across dialects - https://github.com/knex/knex/issues/6335
- SQLFactory: Hierarchical CTE Queries - https://feed.nuget.org/packages/SQLFactory


### Sub-feature 4.2: Infinite Loop Prevention

**Core Definitions:**

- **Core Definition:** Infinite loop prevention refers to techniques for stopping recursive CTEs that would otherwise run indefinitely due to cyclic data or incorrectly defined recursion.
- **Technical Definition:** Infinite loops occur when the recursive member continuously returns rows, typically because of cyclic data (e.g., A reports to B, B reports to A) or a missing termination condition. Prevention techniques include path tracking, `UNION` (instead of `UNION ALL`), and engine-level recursion limits such as SQL Server's `MAXRECURSION` hint or MySQL's `cte_max_recursion_depth` system variable.
- **Beginner-Friendly Explanation:** If your data has loops, a recursive CTE can run forever. You can prevent this by tracking visited paths, using `UNION` to remove duplicates, or setting a maximum number of iterations.

**Purposes:**

- To spot runaway cyclic data loops
- To enforce hard query bounds using engine overrides
- To ensure recursive queries complete in predictable time

**Syntax Structures and Rules:**

```sql
-- SQL Server: MAXRECURSION hint
WITH cte AS (
    ...
)
SELECT * FROM cte
OPTION (MAXRECURSION 100);  -- Limits recursion to 100 levels

-- MySQL: Set session variable
SET SESSION cte_max_recursion_depth = 1000;

-- Path tracking (portable approach)
WITH RECURSIVE cte AS (
    SELECT id, CAST(id AS CHAR(1000)) AS path FROM table WHERE ...
    UNION ALL
    SELECT t.id, CONCAT(cte.path, ',', t.id)
    FROM table t
    INNER JOIN cte ON t.parent_id = cte.id
    WHERE cte.path NOT LIKE CONCAT('%', t.id, '%')
)
```

**Constraints and Limitations:**

- `MAXRECURSION` is specific to SQL Server; other databases have different mechanisms.
- MySQL's `cte_max_recursion_depth` defaults to 1000.
- Path tracking adds overhead and may not work for very deep hierarchies.
- SQL Server's default `MAXRECURSION` is 100; set to 0 for no limit (dangerous).

**Annotated Code Examples:**

```sql
-- SQL Server: Using MAXRECURSION to prevent infinite loop
-- This CTE has a cyclic data problem
WITH cte (EmployeeID, ManagerID, Title) AS (
    SELECT EmployeeID, ManagerID, Title
    FROM dbo.MyEmployees
    WHERE ManagerID IS NOT NULL
    UNION ALL
    SELECT cte.EmployeeID, cte.ManagerID, cte.Title
    FROM cte
    INNER JOIN dbo.MyEmployees AS e ON cte.ManagerID = e.EmployeeID
)
SELECT EmployeeID, ManagerID, Title
FROM cte
OPTION (MAXRECURSION 2);  -- Stops after 2 levels
```

**Expected Output:** The query returns rows from the first two levels of recursion, then stops (even though the data would allow more). Without `MAXRECURSION`, the query would run indefinitely.

**Explanation:** The recursive member joins `cte.ManagerID` to `e.EmployeeID`, but because the data is cyclic, this produces endless rows. `MAXRECURSION 2` limits the recursion to two levels, preventing the infinite loop. This is a safety measure, not a fix—the real fix is to correct the data or query logic.

**Real-World Cases:**

- **Org charts with reporting cycles:** When employee A reports to B and B reports to A (data error).
- **Graph traversal:** Flight networks with round-trip routes.
- **Bill of materials:** Circular dependencies in product assemblies.

**References:**

- Microsoft Learn: WITH common_table_expression (Transact-SQL) - MAXRECURSION - https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql
- MySQL Reference Manual: Recursion Limits - https://dev.mysql.com/doc/refman/8.0/en/with.html


### Sub-feature 4.3: Materialization Controls

**Core Definitions:**

- **Core Definition:** Materialization controls are optimizer hints that tell the database engine whether to compute a CTE once and store it in temporary memory, or to inline it into the outer query.
- **Technical Definition:** In PostgreSQL 12+, CTEs can be annotated with `MATERIALIZED` (compute once, store result) or `NOT MATERIALIZED` (fold into outer query, allowing predicate pushdown). Before PostgreSQL 12, CTEs were always materialized. Other databases (MySQL, SQL Server) have different default behaviors and optimization strategies.
- **Beginner-Friendly Explanation:** `MATERIALIZED` means "calculate this CTE once and save the result." `NOT MATERIALIZED` means "treat this CTE like a subquery and let the optimizer merge it into the main query."

**Purposes:**

- To understand whether the engine recalculates the CTE inline or saves it to temporary memory
- To tune performance bottlenecks by choosing the right materialization strategy
- To control predicate pushdown and join order

**Syntax Structures and Rules:**

```sql
-- PostgreSQL 12+: MATERIALIZED
WITH cte_name AS MATERIALIZED (
    SELECT columns FROM table WHERE condition
)
SELECT * FROM cte_name WHERE outer_condition;

-- PostgreSQL 12+: NOT MATERIALIZED
WITH cte_name AS NOT MATERIALIZED (
    SELECT columns FROM table WHERE condition
)
SELECT * FROM cte_name WHERE outer_condition;
```

**Syntax Rules:**

- `MATERIALIZED` forces the CTE to be computed and stored as a temporary result.
- `NOT MATERIALIZED` folds the CTE into the outer query, allowing the optimizer to push outer conditions into the CTE.
- The default behavior (when neither is specified) depends on the database version and the query's characteristics.

**Constraints and Limitations:**

- Materialization hints are PostgreSQL-specific (available from version 12). Not all databases support them.
- `MATERIALIZED` is beneficial when the CTE is referenced multiple times and the computation is expensive.
- `NOT MATERIALIZED` is beneficial when the outer query has highly selective predicates that should be pushed into the CTE.

**Annotated Code Examples:**

```sql
-- MATERIALIZED: compute CTE once, then filter
EXPLAIN
WITH x1 AS MATERIALIZED (SELECT * FROM t1)
SELECT * FROM x1 WHERE a > 1;
-- Execution plan: CTE is computed first via Shared Scan,
-- then the filter is applied to the result.
```

**Expected Output (execution plan):**

```
Gather Motion
  -> Subquery Scan on x1
       Filter: (x1.a > 1)
       -> Shared Scan
            -> Seq Scan on t1
```

**Explanation:** With `MATERIALIZED`, the CTE is computed once and stored. The outer filter is applied afterward. This is beneficial when the CTE is referenced multiple times, as the computation is done only once.

```sql
-- NOT MATERIALIZED: fold CTE into outer query, push filter
EXPLAIN
WITH x1 AS NOT MATERIALIZED (SELECT * FROM t1)
SELECT * FROM x1 WHERE a > 1;
-- Execution plan: CTE is merged into a single scan with the filter pushed down.
```

**Expected Output (execution plan):**

```
Gather Motion
  -> Seq Scan on t1
       Filter: (a > 1)
```

**Explanation:** With `NOT MATERIALIZED`, the CTE is inlined into the outer query, and the filter is pushed down into the base table scan. This is beneficial when the outer query has selective predicates, as it reduces the amount of data processed.

**Real-World Cases:**

- **Performance tuning:** Using `NOT MATERIALIZED` to enable predicate pushdown in reporting queries.
- **Expensive computations:** Using `MATERIALIZED` when a CTE involves complex joins or aggregations referenced multiple times.
- **ETL pipelines:** Controlling materialization to optimize data transformation workflows.

**References:**

- PostgreSQL Documentation: WITH Queries - https://www.postgresql.org/docs/current/queries-with.html
- Alibaba Cloud: Common Table Expressions - MATERIALIZED and NOT MATERIALIZED - https://www.alibabacloud.com/help/en/analyticdb/analyticdb-for-postgresql/developer-reference/common-table-expressions


## Summary of Key Differences Across Databases

| Feature | PostgreSQL | MySQL 8+ | SQLite | SQL Server | Oracle |
|---------|-----------|----------|--------|------------|--------|
| Recursive keyword | `WITH RECURSIVE` | `WITH RECURSIVE` | `WITH RECURSIVE` | `WITH` | `WITH` |
| Column list required | No | No | No | No | Yes (recursive) |
| Materialization hints | `MATERIALIZED` / `NOT MATERIALIZED` | No | No | No | No |
| Recursion limit | `MAXRECURSION`-like via statement timeout | `cte_max_recursion_depth` | No built-in | `MAXRECURSION` | `MAXRECURSION`-like |
| Cycle detection | Native `CYCLE` clause (14+) | Path tracking | Path tracking | Path tracking | Path tracking |

---

## References

### Official Documentation

- PostgreSQL Documentation: WITH Queries (Common Table Expressions) - https://www.postgresql.org/docs/current/queries-with.html
- MySQL Reference Manual: WITH (Common Table Expressions) - https://dev.mysql.com/doc/refman/8.0/en/with.html
- Microsoft Learn: WITH common_table_expression (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql
- SQLite Documentation: The WITH Clause - https://www2.sqlite.org/matrix/lang_with.html
- Oracle Database SQL Reference: WITH Clause - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SELECT.html
- BigQuery Documentation: Recursive CTEs - https://cloud.google.com/bigquery/docs/reference/standard-sql/query-syntax#recursive_ctes

### Academic and Technical Sources

- ISO/IEC 9075-2:2016 (SQL Standard) - https://www.iso.org/standard/63555.html
- PostgreSQL 12 Release Notes: CTE Materialization - https://www.postgresql.org/docs/12/release-12.html
- knex Issue #6335: Make WITH RECURSIVE behaviour more consistent across dialects - https://github.com/knex/knex/issues/6335
- Alibaba Cloud: Common Table Expressions - MATERIALIZED and NOT MATERIALIZED - https://www.alibabacloud.com/help/en/analyticdb/analyticdb-for-postgresql/developer-reference/common-table-expressions
- BigQuery Recursive CTEs for Hierarchical Data Traversal - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-02-17-how-to-write-recursive-ctes-in-bigquery-for-hierarchical-data-traversal/README.md