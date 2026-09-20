# SQL Subqueries with Operators: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A subquery is a `SELECT` statement nested inside another SQL statement (typically inside a `WHERE`, `HAVING`, or `SELECT` clause), used to return data that the outer query uses for filtering, comparison, or computation.

**Technical Definition:** Per the SQL standard (ISO/IEC 9075), a subquery (also called an inner query or nested query) is a `<query expression>` enclosed in parentheses that appears within another SQL statement. When combined with operators such as `IN`, `EXISTS`, `ANY`, `SOME`, `ALL`, or standard comparison operators (`=`, `<`, `>`, `!=`), the subquery provides a mechanism for multi-step data retrieval within a single statement.

**Beginner-Friendly Explanation:** A subquery is a query inside a query. The inner query runs first and produces a result that the outer query then uses. Think of it like asking a question, getting an answer, and then using that answer to ask a follow-up question—all in one SQL statement.

### Key Characteristics

- **Nested execution:** The inner query executes before the outer query (logically, though optimizers may reorder).
- **Operator-dependent behaviour:** The operator used with the subquery determines how the result is consumed (as a list, a truth value, or a scalar).
- **Scalar vs. multi-row:** Some operators require the subquery to return exactly one value (scalar), while others accept multiple rows.
- **Correlated vs. non-correlated:** A correlated subquery references columns from the outer query and is re-evaluated for each outer row.
- **NULL-sensitive:** NULL values in subquery results can cause unexpected behaviour, especially with `NOT IN`.

### Prerequisites

- Basic SQL `SELECT` syntax
- Understanding of `WHERE`, `HAVING`, and `JOIN` clauses
- Familiarity with NULL handling in SQL
- Basic knowledge of set theory concepts (membership, existence, quantification)

### Related Programming Areas

- **Query Optimization:** Subqueries can often be rewritten as joins, and the optimizer decides which form is more efficient.
- **Data Analysis:** Subqueries enable complex filtering and aggregation logic in reporting.
- **Application Development:** Subqueries are used in ORM-generated SQL and stored procedures.
- **Database Design:** Understanding subquery behaviour informs indexing and schema decisions.

---

## 1. Membership Testing: IN and NOT IN

### Core Definitions

- **Core Definition:** The `IN` operator tests whether a value matches any value in a list or subquery result. `NOT IN` tests whether a value matches none of them.
- **Technical Definition:** `expression IN (subquery)` is equivalent to `expression = ANY (subquery)`, returning TRUE if the expression equals any value returned by the subquery. `NOT IN` is equivalent to `<> ALL (subquery)`, returning TRUE only if the expression does not equal any value.
- **Beginner-Friendly Explanation:** `IN` asks "is this value in the list?" `NOT IN` asks "is this value not in the list?" The list can come from a subquery.

### Purposes

- To filter rows based on membership in a dynamically generated set
- To test whether a value belongs to a list produced by another query
- To implement exclusion logic (find rows NOT in a set)
- To simplify queries that would otherwise require multiple `OR` conditions

### Syntax Structures and Rules

**Complete General Syntax:**

```sql
-- IN with subquery
expression IN (SELECT column FROM table WHERE condition)

-- NOT IN with subquery
expression NOT IN (SELECT column FROM table WHERE condition)
```

**Component Breakdown:**

- `expression`: The value or column being tested
- `IN` / `NOT IN`: The membership operator
- `(SELECT column FROM table WHERE condition)`: A subquery that must return exactly one column

**Syntax Rules:**

- The subquery must return exactly one column; returning multiple columns raises an error.
- `IN` is equivalent to `= ANY`; `NOT IN` is equivalent to `<> ALL`.
- `NOT IN` evaluates to FALSE if any member of the set is NULL.
- If the subquery returns no rows, `NOT IN` returns all rows (because the condition is vacuously true).

**Constraints and Limitations:**

- **The NULL pitfall:** If the subquery returns any NULL value, `NOT IN` will return no rows (or UNKNOWN), because `x <> NULL` is UNKNOWN and `TRUE AND UNKNOWN` is not TRUE.
- Oracle limits the expression list in `IN` to 1,000 expressions when using a literal list (not a subquery).
- `IN` and `NOT IN` cannot be used with row constructors in all databases.

**Annotated Code Examples:**

```sql
-- Example 1: IN with subquery — find products that are in the Wheels subcategory
SELECT Name
FROM Production.Product
WHERE ProductSubcategoryID IN (
    SELECT ProductSubcategoryID
    FROM Production.ProductSubcategory
    WHERE Name = 'Wheels'
);
```

**Expected Output (sample):**

| Name |
|------|
| LL Mountain Front Wheel |
| ML Mountain Front Wheel |
| HL Mountain Front Wheel |
| LL Road Front Wheel |

**Explanation:** The inner query returns the `ProductSubcategoryID` for the "Wheels" subcategory. The outer query then returns all products whose `ProductSubcategoryID` matches that value. This is a clean way to filter based on a dynamic list.

```sql
-- Example 2: NOT IN with subquery — find products NOT in finished bicycle subcategories
SELECT [Name]
FROM Production.Product
WHERE ProductSubcategoryID NOT IN (
    SELECT ProductSubcategoryID
    FROM Production.ProductSubcategory
    WHERE [Name] = 'Mountain Bikes'
       OR [Name] = 'Road Bikes'
       OR [Name] = 'Touring Bikes'
);
```

**Expected Output (sample):**

| Name |
|------|
| Adjustable Race |
| All-Purpose Bike Stand |
| AWC Logo Cap |

**Explanation:** The inner query returns the IDs of the three finished-bicycle subcategories. The outer query returns all products whose `ProductSubcategoryID` is not one of those three. Note that this query cannot be converted to a simple `<>` join because the not-equal join would have a different meaning.

```sql
-- Example 3: The NOT IN NULL pitfall
-- This query returns NO rows if the subquery returns any NULL
SELECT [Name]
FROM Production.Product
WHERE ProductSubcategoryID NOT IN (
    SELECT ProductSubcategoryID
    FROM Production.ProductSubcategory
    WHERE [Name] = 'Mountain Bikes'
    UNION ALL
    SELECT NULL  -- Simulating a NULL in the subquery result
);
```

**Expected Output:** (no rows returned)

**Explanation:** Because the subquery returns a NULL, the `NOT IN` condition evaluates to `ProductSubcategoryID <> 'Mountain Bikes' AND ProductSubcategoryID <> NULL`. The second condition is UNKNOWN, so the entire expression is not TRUE for any row. To avoid this, add `WHERE ProductSubcategoryID IS NOT NULL` inside the subquery.

**Real-World Cases:**

- **E-commerce:** Finding all customers who purchased a specific product.
- **HR:** Finding employees who are not in a particular department.
- **Data cleanup:** Identifying records not referenced by any foreign key.

**References:**

- Microsoft Learn: Subqueries (SQL Server) — NOT IN - https://learn.microsoft.com/en-us/sql/relational-databases/performance/subqueries
- Oracle: IN Condition - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/IN-Condition.html
- PostgreSQL: Subquery Expressions — IN - https://www.postgresql.org/docs/current/functions-subquery.html


## 2. Existence Checking: EXISTS and NOT EXISTS

### Core Definitions

- **Core Definition:** `EXISTS` tests whether a subquery returns any rows at all. `NOT EXISTS` tests whether a subquery returns no rows.
- **Technical Definition:** `EXISTS (subquery)` returns TRUE if the subquery returns at least one row, and FALSE if it returns zero rows. The actual data returned by the subquery is ignored; only the existence of rows matters. `NOT EXISTS` is the logical negation.
- **Beginner-Friendly Explanation:** `EXISTS` asks "does this subquery find anything?" If yes, it's true. `NOT EXISTS` asks "does this subquery find nothing?" If yes, it's true.

### Purposes

- To check for the existence of rows matching a condition without retrieving data
- To implement anti-join logic (find rows with no matching counterpart)
- To enable correlated subqueries that reference the outer query
- To provide a safer alternative to `NOT IN` when NULLs are present

### Syntax Structures and Rules

**Complete General Syntax:**

```sql
-- EXISTS
WHERE EXISTS (SELECT * FROM table WHERE condition)

-- NOT EXISTS
WHERE NOT EXISTS (SELECT * FROM table WHERE condition)
```

**Component Breakdown:**

- `EXISTS` / `NOT EXISTS`: The existence predicate
- `(SELECT * FROM table WHERE condition)`: A subquery whose result set is tested for emptiness

**Syntax Rules:**

- `EXISTS` is not preceded by a column name, constant, or other expression; it stands alone before the subquery.
- The select list of an `EXISTS` subquery conventionally uses `*`, but any expression (e.g., `SELECT 1`) works because the result is ignored.
- The subquery can be correlated (referencing outer columns) or non-correlated.
- `EXISTS` returns TRUE even if the subquery returns rows containing only NULLs, as long as rows exist.

**Constraints and Limitations:**

- `EXISTS` subqueries cannot use aggregate functions without a `GROUP BY` in some databases (implementation-dependent).
- `NOT EXISTS` is generally preferred over `NOT IN` when NULLs may be present, because `NOT EXISTS` is not affected by NULL values in the subquery.

**Annotated Code Examples:**

```sql
-- Example 1: EXISTS — find product types that exist in at least one city
SELECT DISTINCT store_type
FROM stores
WHERE EXISTS (
    SELECT *
    FROM cities_stores
    WHERE cities_stores.store_type = stores.store_type
);
```

**Expected Output (sample):**

| store_type |
|------------|
| Supermarket |
| Convenience |
| Department |

**Explanation:** The outer query iterates over each `store_type`. The correlated subquery checks whether that store type appears in `cities_stores`. If at least one row is found, `EXISTS` is TRUE and the store type is returned.

```sql
-- Example 2: NOT EXISTS — find product types that exist in NO city
SELECT DISTINCT store_type
FROM stores
WHERE NOT EXISTS (
    SELECT *
    FROM cities_stores
    WHERE cities_stores.store_type = stores.store_type
);
```

**Expected Output (sample):**

| store_type |
|------------|
| Warehouse |

**Explanation:** The correlated subquery checks for each store type whether it appears in `cities_stores`. `NOT EXISTS` returns TRUE only when the subquery returns zero rows, meaning the store type exists in no city.

```sql
-- Example 3: Double NOT EXISTS — find store types present in ALL cities
SELECT DISTINCT store_type
FROM stores s1
WHERE NOT EXISTS (
    SELECT *
    FROM cities
    WHERE NOT EXISTS (
        SELECT *
        FROM cities_stores
        WHERE cities_stores.city = cities.city
          AND cities_stores.store_type = s1.store_type
    )
);
```

**Expected Output (sample):**

| store_type |
|------------|
| Supermarket |

**Explanation:** This is a classic relational division pattern. The inner `NOT EXISTS` finds cities where the store type is missing. The outer `NOT EXISTS` finds store types for which no such city exists—meaning the store type is present in every city.

**Real-World Cases:**

- **E-commerce:** Finding customers who have never placed an order.
- **HR:** Finding employees who have no direct reports.
- **Inventory:** Finding products that have never been sold.

**References:**

- MySQL Reference Manual: Subqueries with EXISTS or NOT EXISTS - https://dev.mysql.com/doc/refman/8.0/en/exists-and-not-exists-subqueries.html
- Microsoft Learn: Subqueries with EXISTS - https://learn.microsoft.com/en-us/sql/relational-databases/performance/subqueries
- PostgreSQL: Subquery Expressions — EXISTS - https://www.postgresql.org/docs/current/functions-subquery.html


## 3. Quantified Comparisons: ANY, SOME, and ALL

### Core Definitions

- **Core Definition:** `ANY`, `SOME`, and `ALL` are quantifiers that modify a comparison operator to compare a value against a set of values returned by a subquery.
- **Technical Definition:** `expression operator ANY (subquery)` returns TRUE if the comparison is TRUE for at least one row of the subquery. `expression operator ALL (subquery)` returns TRUE if the comparison is TRUE for every row of the subquery. `SOME` is a synonym for `ANY`.
- **Beginner-Friendly Explanation:** `ANY` means "at least one." `ALL` means "every single one." `SOME` is just another word for `ANY`.

### Purposes

- To compare a value against a set of values with existential (`ANY`/`SOME`) or universal (`ALL`) quantification
- To express conditions like "greater than at least one" or "greater than all"
- To provide a more expressive alternative to `IN` and `NOT IN` when combined with operators other than `=`
- To enable quantified comparisons that cannot be expressed with simple `IN`/`NOT IN`

### Syntax Structures and Rules

**Complete General Syntax:**

```sql
expression comparison_operator { ANY | SOME } (subquery)
expression comparison_operator ALL (subquery)
```

**Component Breakdown:**

- `expression`: The value being compared
- `comparison_operator`: One of `=`, `>`, `<`, `>=`, `<=`, `<>`, `!=`
- `ANY` / `SOME` / `ALL`: The quantifier
- `(subquery)`: A subquery returning exactly one column

**Syntax Rules:**

- `ANY` and `SOME` are synonyms; `SOME` is the ISO-standard keyword.
- `= ANY` is equivalent to `IN`; `<> ALL` is equivalent to `NOT IN`.
- If the subquery returns no rows, `ANY` returns FALSE and `ALL` returns TRUE (vacuously true).
- If the subquery contains NULLs, the result follows SQL three-valued logic: `ANY` returns NULL if no TRUE result is found and at least one NULL is present; `ALL` returns NULL if no FALSE result is found and at least one NULL is present.

**Constraints and Limitations:**

- The subquery must return exactly one column.
- `ANY`/`SOME` and `ALL` cannot be used with `LIKE` or `IS NULL` in most databases.
- Negation rules: `NOT (x > ALL (subquery))` is equivalent to `x <= ANY (subquery)`; `NOT (x > ANY (subquery))` is equivalent to `x <= ALL (subquery)`.

**Annotated Code Examples:**

```sql
-- Example 1: ANY — find products with list price >= ANY subcategory max
SELECT Name
FROM Production.Product
WHERE ListPrice >= ANY (
    SELECT MAX(ListPrice)
    FROM Production.Product
    GROUP BY ProductSubcategoryID
);
```

**Expected Output (sample):**

| Name |
|------|
| Mountain-200 |
| Road-150 |
| Touring-3000 |

**Explanation:** The inner query returns the maximum list price for each product subcategory. The outer query returns products whose list price is greater than or equal to at least one of those maxima (i.e., greater than or equal to the minimum of the maxima). This is equivalent to `ListPrice >= MIN(subquery)`.

```sql
-- Example 2: ALL — find employees whose salary exceeds ALL manager salaries
SELECT emp
FROM Employee emp
WHERE emp.salary > ALL (
    SELECT m.salary
    FROM Manager m
    WHERE m.department = emp.department
);
```

**Expected Output (sample):**

| emp |
|-----|
| Smith |
| Johnson |

**Explanation:** The correlated subquery returns the salaries of all managers in the employee's department. `> ALL` means the employee's salary must be greater than every manager's salary in that department—i.e., greater than the maximum manager salary.

```sql
-- Example 3: SOME — equivalent to ANY
SELECT s1
FROM t1
WHERE s1 > SOME (SELECT s1 FROM t2);
-- This is identical to:
SELECT s1
FROM t1
WHERE s1 > ANY (SELECT s1 FROM t2);
```

**Expected Output:** Identical results from both queries.

**Explanation:** `SOME` is a synonym for `ANY`. The ISO SQL standard defines `SOME` as the preferred keyword, though `ANY` is more commonly used in practice.

**Real-World Cases:**

- **Sales:** Finding products priced above all competitor products.
- **HR:** Finding employees earning more than any manager in their department.
- **Finance:** Finding transactions exceeding any transaction in a reference period.

**References:**

- SAP Help Portal: Subquery That Follows ANY, ALL, or SOME - https://help.sap.com/docs/SAP_SQL_Anywhere/e38b2f6217f24bdb90a3ff8ae57b1dd5/819007fa6ce210148036e2d4518e0129.html
- Microsoft Learn: Comparison Operators Modified by ANY, SOME, or ALL - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/any-transact-sql
- PostgreSQL: Subquery Expressions — ANY/SOME, ALL - https://www.postgresql.org/docs/current/functions-subquery.html


## 4. Standard Operators: Combining Subqueries with Comparison Operators

### Core Definitions

- **Core Definition:** Standard comparison operators (`=`, `<`, `>`, `!=`, `<=`, `>=`) can be used with subqueries when the subquery returns exactly one value (a scalar subquery).
- **Technical Definition:** A subquery introduced with an unmodified comparison operator (i.e., not followed by `ANY` or `ALL`) must return a single value rather than a list of values. If the subquery returns more than one row, the database raises an error.
- **Beginner-Friendly Explanation:** If you use `=`, `<`, or `>` with a subquery, the subquery must return exactly one value. If it returns more than one, you'll get an error.

### Purposes

- To compare a value against a single computed result from another query
- To use aggregate functions (`MAX`, `MIN`, `SUM`, `AVG`, `COUNT`) as comparison targets
- To implement scalar subquery logic where the result is a known single value
- To express conditions that cannot be handled by `IN` or `EXISTS`

### Syntax Structures and Rules

**Complete General Syntax:**

```sql
non_subquery_operand comparison_operator (subquery)
```

**Component Breakdown:**

- `non_subquery_operand`: The value or expression being compared
- `comparison_operator`: One of `=`, `>`, `<`, `>=`, `<=`, `<>`, `!=`
- `(subquery)`: A subquery that must return exactly one value (one row, one column)

**Syntax Rules:**

- The subquery must return exactly one column.
- The subquery must return at most one row. If it returns more than one row, an error occurs.
- Scalar subqueries can be used in `SELECT`, `WHERE`, and `HAVING` clauses.
- The subquery can be correlated (referencing outer columns) or non-correlated.

**Constraints and Limitations:**

- **Single-value requirement:** An unmodified comparison operator requires a scalar subquery. If the subquery returns multiple rows, the query fails.
- **NULL propagation:** If the subquery returns NULL, the comparison evaluates to UNKNOWN, which is treated as FALSE in a `WHERE` clause.
- **Performance consideration:** Scalar subqueries in `SELECT` lists are executed per row, which can be inefficient for large result sets.

**Annotated Code Examples:**

```sql
-- Example 1: = with scalar subquery — find the customer in Linda Mitchell's territory
SELECT CustomerID
FROM Sales.Customer
WHERE TerritoryID = (
    SELECT TerritoryID
    FROM Sales.SalesPerson
    WHERE FirstName = 'Linda' AND LastName = 'Mitchell'
);
```

**Expected Output (sample):**

| CustomerID |
|------------|
| 11001 |
| 11002 |
| 11003 |

**Explanation:** The inner query returns the single `TerritoryID` assigned to Linda Mitchell. The outer query then finds all customers in that territory. Because each salesperson covers exactly one territory, the subquery returns exactly one value.

```sql
-- Example 2: > with aggregate subquery — find products above average price
SELECT ProductName, Price
FROM Products
WHERE Price > (
    SELECT AVG(Price)
    FROM Products
);
```

**Expected Output (sample):**

| ProductName | Price |
|-------------|-------|
| Laptop Pro | 1299.99 |
| Monitor 4K | 599.99 |

**Explanation:** The subquery returns the average price of all products (a single value). The outer query returns products priced above that average. This is a classic scalar subquery pattern.

```sql
-- Example 3: Scalar subquery in SELECT list
SELECT
    ProductName,
    Price,
    (SELECT AVG(Price) FROM Products) AS AvgPrice,
    Price - (SELECT AVG(Price) FROM Products) AS DiffFromAvg
FROM Products;
```

**Expected Output (sample):**

| ProductName | Price | AvgPrice | DiffFromAvg |
|-------------|-------|----------|-------------|
| Laptop Pro | 1299.99 | 450.00 | 849.99 |
| Basic Mouse | 19.99 | 450.00 | -430.01 |

**Explanation:** The scalar subquery `(SELECT AVG(Price) FROM Products)` is executed once and its result is used in each row of the outer query. This allows you to compare each row's value against the overall average.

**Real-World Cases:**

- **Finance:** Finding transactions above the average transaction value.
- **HR:** Finding employees earning more than the department average.
- **E-commerce:** Finding products priced higher than the maximum price of a competitor.

**References:**

- Microsoft Learn: Subqueries with Comparison Operators - https://learn.microsoft.com/en-us/sql/relational-databases/performance/subqueries
- MySQL Reference Manual: Comparisons Using Subqueries - https://dev.mysql.com/doc/refman/8.0/en/subqueries.html
- Oracle: Comparisons Using Subqueries - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Comparisons-Using-Subqueries.html


## Summary of Subquery Operators

| Operator | Equivalent | Subquery Returns | NULL Behaviour | Best For |
|----------|-----------|-----------------|----------------|----------|
| `IN` | `= ANY` | One column, multiple rows | `NOT IN` fails if NULL present | Membership testing |
| `NOT IN` | `<> ALL` | One column, multiple rows | Returns no rows if any NULL | Exclusion testing |
| `EXISTS` | — | Any (result ignored) | Not affected by NULL | Existence checking |
| `NOT EXISTS` | — | Any (result ignored) | Not affected by NULL | Anti-join |
| `= ANY` | `IN` | One column, multiple rows | NULL-sensitive | Quantified equality |
| `> ANY` | — | One column, multiple rows | NULL-sensitive | Greater than minimum |
| `> ALL` | — | One column, multiple rows | NULL-sensitive | Greater than maximum |
| `= (subquery)` | — | Exactly one value | Returns UNKNOWN if NULL | Scalar comparison |
| `> (subquery)` | — | Exactly one value | Returns UNKNOWN if NULL | Scalar comparison |


## References

### Official Documentation

- Microsoft Learn: Subqueries (SQL Server) - https://learn.microsoft.com/en-us/sql/relational-databases/performance/subqueries
- Microsoft Learn: Subqueries with EXISTS - https://learn.microsoft.com/en-us/sql/relational-databases/performance/subqueries
- Microsoft Learn: Comparison Operators Modified by ANY, SOME, or ALL - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/any-transact-sql
- Oracle: IN Condition - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/IN-Condition.html
- Oracle: Comparisons Using Subqueries - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Comparisons-Using-Subqueries.html
- MySQL Reference Manual: Subqueries with EXISTS or NOT EXISTS - https://dev.mysql.com/doc/refman/8.0/en/exists-and-not-exists-subqueries.html
- MySQL Reference Manual: Comparisons Using Subqueries - https://dev.mysql.com/doc/refman/8.0/en/subqueries.html
- MySQL Reference Manual: Subqueries with ANY, IN, or SOME - https://dev.mysql.com/doc/refman/8.0/en/subqueries.html
- PostgreSQL: Subquery Expressions - https://www.postgresql.org/docs/current/functions-subquery.html
- SAP Help Portal: Subquery That Follows ANY, ALL, or SOME - https://help.sap.com/docs/SAP_SQL_Anywhere/e38b2f6217f24bdb90a3ff8ae57b1dd5/819007fa6ce210148036e2d4518e0129.html
- SAP Help Portal: Subquery That Follows EXISTS - https://help.sap.com/docs/SAP_SQL_Anywhere/e38b2f6217f24bdb90a3ff8ae57b1dd5/819007fa6ce210148036e2d4518e0129.html

### Academic and Technical Sources

- ISO/IEC 9075-2:2016 (SQL Standard) - https://www.iso.org/standard/63555.html
- ScienceDirect: Correlated Subqueries - https://www.sciencedirect.com/topics/computer-science/correlated-subquery
- Ask TOM (Oracle): NOT IN operator with NULL data - https://asktom.oracle.com/pls/apex/asktom.search?tag=not-in-operator-with-null-data
- PostgreSQL Mailing List: Strange query problem (NOT IN with NULL) - https://www.postgresql.org/message-id/200901281850.n0SIo1v07374@candle.pha.pa.us