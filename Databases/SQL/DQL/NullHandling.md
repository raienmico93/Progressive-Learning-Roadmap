# SQL NULL Handling: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** NULL is a special marker in SQL used to represent the absence of a value—data that is missing, unknown, or not applicable. It is not a value itself but a state indicating that no value exists.

**Technical Definition:** In the SQL standard (ISO/IEC 9075), NULL is a special value that is distinct from all non-null values and from other NULLs. It is used to represent missing information, and its presence triggers three-valued logic (3VL) in Boolean expressions, where comparisons involving NULL evaluate to UNKNOWN rather than TRUE or FALSE.

**Beginner-Friendly Explanation:** Think of NULL as an empty box on a form. The box exists, but nothing has been written in it. It is not the same as writing "0" (which is a number) or leaving a blank space (which could be an empty string). NULL simply means "we don't know what goes here" or "there is nothing to put here."

### Key Characteristics

- **Not a value:** NULL is a marker for missing or unknown data, not a data value itself.
- **Not equal to anything:** NULL cannot be compared using `=`, `<>`, `<`, `>`, etc. Comparing anything to NULL yields UNKNOWN.
- **Propagates through expressions:** Any arithmetic expression containing NULL evaluates to NULL.
- **Requires special operators:** Use `IS NULL` and `IS NOT NULL` to test for NULL.
- **Affects aggregates differently:** Most aggregate functions ignore NULLs; `COUNT(*)` includes them.
- **Sorts at the end (or beginning) of results:** Default sorting behavior for NULLs varies by database vendor and sort direction.
- **Vendor-specific handling exists:** Functions like `IFNULL`, `NVL`, and `ISNULL` are vendor-specific ways to replace NULLs.

### Prerequisites

To understand SQL NULL handling, you should be familiar with:

- Basic SQL syntax (SELECT, FROM, WHERE, ORDER BY)
- Data types and table structures
- Basic arithmetic operators (+, -, *, /)
- Basic aggregate functions (COUNT, SUM, AVG, MIN, MAX)
- Logical operators (AND, OR, NOT)

### Related Programming Areas

- **Database Design and Normalization:** NULLs affect table design decisions, particularly regarding NOT NULL constraints and primary keys.
- **Data Integrity and Validation:** Understanding NULL is critical for data quality, especially when dealing with optional fields.
- **Query Optimization:** NULL handling can impact index usage and query performance.
- **Application Development:** Most programming languages have their own null/nil concepts that interact with SQL NULLs through database drivers.
- **Data Warehousing and ETL:** NULL handling is a major concern in data transformation and loading processes.
- **Business Intelligence and Reporting:** Aggregations and calculations often require explicit NULL handling to produce accurate results.

---

## Core Concepts / Key Features

### 1. Foundational Concepts

#### 1.1 Meaning of NULL (Missing or Unknown Information)

**Core Definitions:**

- **Core Definition:** NULL represents missing, unknown, or inapplicable information in a database column.
- **Technical Definition:** Per the SQL standard, NULL is a special marker that indicates the absence of a value. It is used when the actual value is not known or when a value would not be meaningful.
- **Beginner-Friendly Explanation:** Imagine a customer record where the "middle name" field is empty. That doesn't mean the customer has no middle name—it means we haven't recorded it. NULL is that "not recorded" state.

**Purposes:**

- To represent missing data without fabricating a value
- To distinguish between "known to be zero/empty" and "unknown"
- To handle optional fields in database schemas
- To support data integration from sources that may not provide all fields

**Syntax Structures and Rules:**

There is no specific syntax for NULL itself—it is a literal keyword used in SQL statements. The general syntax for using NULL is:

```sql
-- Inserting NULL
INSERT INTO table_name (column1, column2) VALUES (value1, NULL);

-- Updating to NULL
UPDATE table_name SET column1 = NULL WHERE condition;

-- Selecting NULL
SELECT column1, NULL AS placeholder FROM table_name;
```

**Syntax Rules:**

- NULL can be used anywhere a literal value is expected.
- Columns must be nullable (not constrained by NOT NULL or PRIMARY KEY) to accept NULL.
- NULL cannot be used in primary key columns.

**Constraints and Limitations:**

- Primary key columns cannot contain NULL.
- Columns with NOT NULL constraints reject NULL values.
- NULL cannot be compared with standard operators; `IS NULL` and `IS NOT NULL` must be used.
- Aggregate functions handle NULL differently (discussed in Section 3.2).

**Annotated Code Examples:**

```sql
-- Example 1: Creating a table with nullable columns
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,          -- NOT NULL enforced by PRIMARY KEY
    first_name VARCHAR(50) NOT NULL,      -- Cannot be NULL
    middle_name VARCHAR(50),              -- Can be NULL (no constraint)
    last_name VARCHAR(50) NOT NULL,       -- Cannot be NULL
    phone_number VARCHAR(20)              -- Can be NULL
);

-- Inserting a row with a NULL middle_name and phone_number
INSERT INTO employees (employee_id, first_name, middle_name, last_name, phone_number)
VALUES (1, 'John', NULL, 'Smith', NULL);

-- Inserting a row with all values provided
INSERT INTO employees (employee_id, first_name, middle_name, last_name, phone_number)
VALUES (2, 'Jane', 'Marie', 'Doe', '555-0100');

-- Querying the table
SELECT * FROM employees;
```

**Expected Output:**

| employee_id | first_name | middle_name | last_name | phone_number |
|-------------|------------|-------------|-----------|--------------|
| 1           | John       | NULL        | Smith     | NULL         |
| 2           | Jane       | Marie       | Doe       | 555-0100     |

**Explanation:** Row 1 shows NULL for `middle_name` and `phone_number` because we explicitly inserted NULL. Row 2 shows actual values because we provided them. The NULLs indicate that the data is missing, not that the values are zero or empty strings.

**Real-World Cases:**

- **Customer databases:** Middle names, secondary phone numbers, and fax numbers are often unknown or not provided.
- **Product catalogs:** Discontinued products may have a NULL discontinue date until they are actually discontinued.
- **HR systems:** Termination dates are NULL for current employees.
- **Financial systems:** A payment date is NULL until the payment is made.

**References:**

- Microsoft Learn: Null Values - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/null-and-unknown-transact-sql
- Oracle Database SQL Reference: Nulls - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Nulls.html
- SAP Help Portal: NULL Special Value - https://help.sap.com/doc/abapdocu_752_index_htm/7.52/en-US/abennull_value.htm


#### 1.2 Difference Between NULL and Zero (0)

**Core Definitions:**

- **Core Definition:** Zero is a known numeric value representing "nothing" in a numerical sense, while NULL represents "unknown" or "missing" information.
- **Technical Definition:** In SQL, 0 is a valid numeric literal that can participate in arithmetic and comparisons. NULL is a special marker that propagates through expressions and cannot be compared using standard operators.
- **Beginner-Friendly Explanation:** If your bank balance is 0, you know you have no money. If your bank balance is NULL, you don't know how much money you have—the information is simply missing.

**Purposes:**

- To avoid confusing "no quantity" with "unknown quantity"
- To ensure accurate mathematical calculations where 0 contributes to sums but NULL does not
- To distinguish between a deliberate zero value and a data entry omission

**Syntax Structures and Rules:**

```sql
-- Zero is a numeric literal
SELECT 0 AS zero_value;

-- NULL is a special keyword
SELECT NULL AS null_value;

-- Arithmetic with zero produces a result
SELECT 5 + 0 AS result_with_zero;  -- Returns 5

-- Arithmetic with NULL produces NULL
SELECT 5 + NULL AS result_with_null;  -- Returns NULL
```

**Syntax Rules:**

- 0 is used wherever a numeric value is expected.
- NULL cannot be used in arithmetic without producing NULL.
- Comparisons with 0 work normally (`column = 0`), but comparisons with NULL require `IS NULL`.

**Constraints and Limitations:**

- Some databases (notably Oracle) treat empty strings as NULL, which can blur the line between 0 and NULL for character data, but this is not standard SQL.
- Numeric columns can store 0 and NULL as distinct values.

**Annotated Code Examples:**

```sql
-- Creating a table to demonstrate the difference
CREATE TABLE inventory (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    quantity INT,          -- Can be 0 or NULL
    price DECIMAL(10,2)
);

-- Inserting data
INSERT INTO inventory VALUES (1, 'Widget', 0, 19.99);    -- Zero quantity
INSERT INTO inventory VALUES (2, 'Gadget', NULL, 29.99); -- Unknown quantity
INSERT INTO inventory VALUES (3, 'Gizmo', 10, 9.99);     -- Known quantity

-- Query to show the difference
SELECT 
    product_name,
    quantity,
    CASE 
        WHEN quantity IS NULL THEN 'Unknown'
        WHEN quantity = 0 THEN 'Out of Stock'
        ELSE 'In Stock'
    END AS stock_status
FROM inventory;
```

**Expected Output:**

| product_name | quantity | stock_status |
|--------------|----------|--------------|
| Widget       | 0        | Out of Stock |
| Gadget       | NULL     | Unknown      |
| Gizmo        | 10       | In Stock     |

**Explanation:** The Widget has a quantity of 0, which we interpret as "out of stock"—we know it's zero. The Gadget has a NULL quantity, meaning we don't know how many are in stock—it's not necessarily zero. The Gizmo has 10, meaning it's in stock.

**Real-World Cases:**

- **Retail:** A product with 0 inventory is "out of stock" (known), while a product with NULL inventory means the inventory count hasn't been taken (unknown).
- **Finance:** A customer with a 0 balance owes nothing, while a NULL balance means the balance hasn't been calculated.
- **Manufacturing:** A machine with 0 defects produced none, while a NULL defect count means the inspection hasn't been performed.

**References:**

- Microsoft Learn: Null Values - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/null-and-unknown-transact-sql
- Oracle: Nulls - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Nulls.html
- Navicat: Comparing the Semantics of Null, Zero, and Empty String - https://www.navicat.com/en/company/aboutus/blog/1796-null-zero-empty-string.html


#### 1.3 Difference Between NULL and an Empty String ('')

**Core Definitions:**

- **Core Definition:** An empty string is a known string value of length zero, while NULL represents the absence of any value.
- **Technical Definition:** In standard SQL, `''` is a valid character string with length 0, distinct from NULL. However, some databases (notably Oracle) historically treat empty strings as NULL, which is a vendor-specific behavior and not ANSI-compliant.
- **Beginner-Friendly Explanation:** An empty string is like an envelope with nothing inside—it's still an envelope. NULL is like not having an envelope at all.

**Purposes:**

- To distinguish between "recorded as blank" and "not recorded"
- To comply with ANSI SQL standards for data integrity
- To avoid unexpected behavior in string concatenation and length calculations

**Syntax Structures and Rules:**

```sql
-- Empty string literal
SELECT '' AS empty_string;

-- NULL literal
SELECT NULL AS null_value;

-- Length of empty string is 0
SELECT LENGTH('') AS empty_length;  -- Returns 0

-- Length of NULL is NULL (not 0)
SELECT LENGTH(NULL) AS null_length;  -- Returns NULL

-- Concatenation behavior varies
SELECT 'Hello' || '' AS with_empty;   -- Returns 'Hello' (in standard SQL)
SELECT 'Hello' || NULL AS with_null;  -- Returns NULL (in most databases)
```

**Syntax Rules:**

- `''` is a valid string literal in all SQL databases.
- NULL cannot be concatenated with `||` or `+` without producing NULL (except in MySQL, where `CONCAT` treats NULL as empty).
- `LENGTH('')` returns 0; `LENGTH(NULL)` returns NULL.

**Constraints and Limitations:**

- **Oracle:** Treats `''` as NULL in VARCHAR2 columns (non-standard behavior). This is documented but may change in future releases.
- **MySQL:** Distinguishes between `''` and NULL in most contexts.
- **PostgreSQL:** Strictly distinguishes between `''` and NULL.
- **SQL Server:** Distinguishes between `''` and NULL.

**Annotated Code Examples:**

```sql
-- PostgreSQL / SQL Server / MySQL (standard-compliant behavior)
CREATE TABLE contacts (
    contact_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(200)
);

INSERT INTO contacts VALUES (1, 'Alice', '');      -- Empty email (known)
INSERT INTO contacts VALUES (2, 'Bob', NULL);      -- Unknown email
INSERT INTO contacts VALUES (3, 'Charlie', 'charlie@example.com');

-- Query showing the difference
SELECT 
    name,
    email,
    LENGTH(email) AS email_length,
    CASE 
        WHEN email IS NULL THEN 'No email on file'
        WHEN email = '' THEN 'Email is blank'
        ELSE 'Email provided'
    END AS email_status
FROM contacts;
```

**Expected Output (PostgreSQL):**

| name    | email               | email_length | email_status      |
|---------|---------------------|--------------|-------------------|
| Alice   |                     | 0            | Email is blank    |
| Bob     | NULL                | NULL         | No email on file  |
| Charlie | charlie@example.com | 19           | Email provided    |

**Explanation:** Alice has an empty string email—we know she has no email address (or we recorded it as blank). Bob has NULL email—we don't know if he has an email or not. Charlie has a real email address.

**Real-World Cases:**

- **CRM systems:** An empty string might mean "customer declined to provide email," while NULL means "email not yet collected."
- **Form processing:** An empty string could mean the user submitted a blank field, while NULL means the field was never rendered.
- **Data migration:** Source systems may use empty strings where the target system expects NULL, requiring careful mapping.

**References:**

- Oracle Ask TOM: Strings of Zero Length Not Equivalent To NULL - https://asktom.oracle.com/pls/apex/asktom.search?tag=strings-of-zero-length-not-equivalent-to-null
- InterSystems: The SQL Zero-Length String - https://cedocs.intersystems.com/latest/csp/docbook/DocBook.UI.Page.cls?KEY=RSQL_null
- PostgreSQL Documentation: NULL - https://www.postgresql.org/docs/current/datatype.html


### 2. Logical Evaluation

#### 2.1 IS NULL and IS NOT NULL Operators

**Core Definitions:**

- **Core Definition:** `IS NULL` and `IS NOT NULL` are SQL operators used to test whether a value is NULL or not.
- **Technical Definition:** These are special predicate operators defined in the SQL standard (ISO/IEC 9075) that return TRUE or FALSE when testing for the presence of NULL, unlike standard comparison operators which return UNKNOWN.
- **Beginner-Friendly Explanation:** You can't ask "is this column equal to NULL?" because NULL isn't a value you can compare. Instead, you ask "is this column NULL?" using the special `IS NULL` operator.

**Purposes:**

- To filter rows where a column has no value
- To filter rows where a column has a value
- To implement conditional logic based on data presence
- To handle NULLs in WHERE clauses and CASE expressions

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- Testing for NULL
expression IS NULL

-- Testing for NOT NULL
expression IS NOT NULL
```

**Component Breakdown:**

- `expression`: Any valid SQL expression (column name, literal, function result, etc.)
- `IS NULL`: Returns TRUE if the expression evaluates to NULL, FALSE otherwise
- `IS NOT NULL`: Returns TRUE if the expression evaluates to a non-NULL value, FALSE otherwise

**Syntax Rules:**

- `IS NULL` and `IS NOT NULL` must be used instead of `= NULL` or `<> NULL`.
- They can be used in WHERE clauses, HAVING clauses, CASE expressions, and SELECT lists.
- They work with any data type.

**Constraints and Limitations:**

- `IS NULL` and `IS NOT NULL` cannot be used with `NOT` in some databases (use `IS NOT NULL` instead of `NOT (IS NULL)`).
- Some databases allow `IS NULL` in index conditions; others do not.

**Annotated Code Examples:**

```sql
-- Example 1: Basic IS NULL and IS NOT NULL
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    phone VARCHAR(20),
    email VARCHAR(200)
);

INSERT INTO customers VALUES (1, 'Alice', '555-0100', 'alice@example.com');
INSERT INTO customers VALUES (2, 'Bob', NULL, 'bob@example.com');
INSERT INTO customers VALUES (3, 'Charlie', '555-0200', NULL);
INSERT INTO customers VALUES (4, 'Diana', NULL, NULL);

-- Find customers without a phone number
SELECT name FROM customers WHERE phone IS NULL;
-- Returns: Bob, Diana

-- Find customers with a phone number
SELECT name FROM customers WHERE phone IS NOT NULL;
-- Returns: Alice, Charlie

-- Find customers with both phone and email
SELECT name FROM customers WHERE phone IS NOT NULL AND email IS NOT NULL;
-- Returns: Alice
```

**Expected Output:**

| Query | Result |
|-------|--------|
| `WHERE phone IS NULL` | Bob, Diana |
| `WHERE phone IS NOT NULL` | Alice, Charlie |
| `WHERE phone IS NOT NULL AND email IS NOT NULL` | Alice |

**Explanation:** `IS NULL` correctly identifies rows where the phone column has no value. Standard comparison `phone = NULL` would return no rows because `phone = NULL` evaluates to UNKNOWN for every row.

```sql
-- Example 2: Using IS NULL in CASE expressions
SELECT 
    name,
    CASE 
        WHEN phone IS NULL THEN 'No phone'
        ELSE 'Phone: ' || phone
    END AS phone_display
FROM customers;
```

**Expected Output:**

| name    | phone_display    |
|---------|------------------|
| Alice   | Phone: 555-0100  |
| Bob     | No phone         |
| Charlie | Phone: 555-0200  |
| Diana   | No phone         |

**Real-World Cases:**

- **E-commerce:** Finding orders with no shipping date (not yet shipped).
- **Healthcare:** Identifying patients with no recorded allergies.
- **Education:** Finding students who haven't submitted an assignment (submission date IS NULL).
- **Finance:** Detecting accounts with no transaction history.

**References:**

- Microsoft Learn: IS NULL (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/queries/is-null-transact-sql
- Oracle: Nulls - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Nulls.html
- PostgreSQL: Comparison Operators - https://www.postgresql.org/docs/current/functions-comparison.html


#### 2.2 Three-Valued Logic (Truth Tables with TRUE, FALSE, and UNKNOWN)

**Core Definitions:**

- **Core Definition:** Three-valued logic (3VL) is a logical system used in SQL where Boolean expressions can evaluate to TRUE, FALSE, or UNKNOWN (represented by NULL).
- **Technical Definition:** As specified in the SQL standard, ternary logic is used for evaluating predicates and Boolean expressions. When any operand in a comparison is NULL, the result is UNKNOWN. Logical operators (AND, OR, NOT) extend their truth tables to account for UNKNOWN.
- **Beginner-Friendly Explanation:** In normal logic, statements are either true or false. In SQL, there's a third option: "I don't know." If you ask "Is this person's age greater than 18?" and the age is NULL, the answer is "I don't know" (UNKNOWN), not "no" (FALSE).

**Purposes:**

- To handle missing data in logical expressions consistently
- To ensure that queries involving NULLs don't produce misleading results
- To provide a formal framework for reasoning about unknown values
- To standardize behavior across different SQL implementations

**Syntax Structures and Rules:**

**Truth Tables for Logical Operators:**

**AND Operator:**

| AND | TRUE | FALSE | UNKNOWN |
|-----|------|-------|---------|
| TRUE | TRUE | FALSE | UNKNOWN |
| FALSE | FALSE | FALSE | FALSE |
| UNKNOWN | UNKNOWN | FALSE | UNKNOWN |

**OR Operator:**

| OR | TRUE | FALSE | UNKNOWN |
|-----|------|-------|---------|
| TRUE | TRUE | TRUE | TRUE |
| FALSE | TRUE | FALSE | UNKNOWN |
| UNKNOWN | TRUE | UNKNOWN | UNKNOWN |

**NOT Operator:**

| NOT | Result |
|-----|--------|
| TRUE | FALSE |
| FALSE | TRUE |
| UNKNOWN | UNKNOWN |

**Comparison Operators:**

| Comparison | Result |
|------------|--------|
| NULL = value | UNKNOWN |
| NULL <> value | UNKNOWN |
| NULL < value | UNKNOWN |
| NULL > value | UNKNOWN |
| NULL <= value | UNKNOWN |
| NULL >= value | UNKNOWN |
| NULL = NULL | UNKNOWN |
| NULL <> NULL | UNKNOWN |

**Syntax Rules:**

- Any comparison with NULL yields UNKNOWN.
- UNKNOWN propagates through AND and OR according to the truth tables above.
- `NOT UNKNOWN` is UNKNOWN.
- In a WHERE clause, rows where the condition evaluates to UNKNOWN are excluded (treated as FALSE for filtering purposes).

**Constraints and Limitations:**

- UNKNOWN is represented as NULL in most database systems.
- `WHERE` clauses treat UNKNOWN as FALSE for row filtering.
- `CHECK` constraints treat UNKNOWN as TRUE (i.e., a CHECK constraint passes if it evaluates to UNKNOWN).
- Some databases have configuration options to change this behavior (e.g., `NULLS DISTINCT` in unique indexes).

**Annotated Code Examples:**

```sql
-- Demonstrating three-valued logic
CREATE TABLE logic_demo (
    id INT PRIMARY KEY,
    value INT
);

INSERT INTO logic_demo VALUES (1, 10);
INSERT INTO logic_demo VALUES (2, NULL);
INSERT INTO logic_demo VALUES (3, 20);

-- Query 1: WHERE clause with NULL comparison
SELECT * FROM logic_demo WHERE value > 15;
-- Returns: id=3 (value=20)
-- id=2 is excluded because value > 15 evaluates to UNKNOWN

-- Query 2: Explicit comparison with NULL
SELECT * FROM logic_demo WHERE value = NULL;
-- Returns: no rows
-- Because value = NULL is UNKNOWN for every row

-- Query 3: Using IS NULL
SELECT * FROM logic_demo WHERE value IS NULL;
-- Returns: id=2

-- Query 4: AND with UNKNOWN
SELECT * FROM logic_demo WHERE value > 5 AND value < 15;
-- Returns: id=1 (value=10)
-- id=2: value > 5 is UNKNOWN, value < 15 is UNKNOWN, so UNKNOWN AND UNKNOWN = UNKNOWN
-- id=3: value > 5 is TRUE, value < 15 is FALSE, so TRUE AND FALSE = FALSE

-- Query 5: OR with UNKNOWN
SELECT * FROM logic_demo WHERE value > 5 OR value < 15;
-- Returns: id=1, id=3
-- id=2: UNKNOWN OR UNKNOWN = UNKNOWN, so excluded
-- id=1: TRUE OR TRUE = TRUE
-- id=3: TRUE OR FALSE = TRUE
```

**Expected Output:**

| Query | Result |
|-------|--------|
| `WHERE value > 15` | id=3 |
| `WHERE value = NULL` | (no rows) |
| `WHERE value IS NULL` | id=2 |
| `WHERE value > 5 AND value < 15` | id=1 |
| `WHERE value > 5 OR value < 15` | id=1, id=3 |

**Explanation:** The key insight is that UNKNOWN is not TRUE. In a WHERE clause, only rows where the condition evaluates to TRUE are returned. UNKNOWN and FALSE both exclude the row, but they are different concepts.

```sql
-- Demonstrating NOT with UNKNOWN
SELECT 
    id,
    value,
    NOT (value > 15) AS not_gt_15,
    (value IS NULL) AS is_null_flag
FROM logic_demo;
```

**Expected Output:**

| id | value | not_gt_15 | is_null_flag |
|----|-------|-----------|--------------|
| 1  | 10    | TRUE      | FALSE        |
| 2  | NULL  | NULL      | TRUE         |
| 3  | 20    | FALSE     | FALSE        |

**Explanation:** For id=2, `value > 15` is UNKNOWN, so `NOT (value > 15)` is also UNKNOWN, which appears as NULL in the result set.

**Real-World Cases:**

- **Compliance queries:** "Find all accounts that are NOT overdue"—accounts with NULL due dates are not returned because `due_date < CURRENT_DATE` is UNKNOWN.
- **Data validation:** "Find all records where the email does NOT match the pattern"—records with NULL emails are excluded because the comparison is UNKNOWN.
- **Report filters:** "Show all products NOT in the discontinued category"—products with NULL categories are excluded from the result.

**References:**

- Snowflake: Ternary Logic - https://docs.snowflake.com/en/sql-reference/ternary-logic
- SAP Help Portal: Three-valued Logic - https://help.sap.com/doc/abapdocu_752_index_htm/7.52/en-US/abenthree_valued_logic.htm
- PostgreSQL: RFC: A brief guide to nulls - https://www.postgresql.org/message-id/200301150038.h0F0cLZ06636@candle.pha.pa.us
- Microsoft Learn: AND (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/and-transact-sql


### 3. Behavior & Propagation

#### 3.1 NULL Propagation in Arithmetic Expressions

**Core Definitions:**

- **Core Definition:** NULL propagation means that any arithmetic expression involving NULL evaluates to NULL.
- **Technical Definition:** In SQL, all arithmetic operators (+, -, *, /) return NULL when any operand is NULL. This is defined by the SQL standard and is consistent across major database systems.
- **Beginner-Friendly Explanation:** If you don't know one of the numbers in a calculation, you can't know the answer. 5 + unknown = unknown. This is exactly how SQL treats NULL in arithmetic.

**Purposes:**

- To prevent misleading results from calculations involving missing data
- To ensure that unknown inputs produce unknown outputs
- To maintain data integrity by not fabricating values

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- Arithmetic with NULL (any operator)
NULL + value        -- Returns NULL
NULL - value        -- Returns NULL
NULL * value        -- Returns NULL
NULL / value        -- Returns NULL
value + NULL        -- Returns NULL
value - NULL        -- Returns NULL
value * NULL        -- Returns NULL
value / NULL        -- Returns NULL
NULL + NULL         -- Returns NULL
```

**Component Breakdown:**

- `NULL`: The special NULL marker
- `value`: Any numeric or date/time expression
- `+`, `-`, `*`, `/`: Standard arithmetic operators

**Syntax Rules:**

- Any arithmetic operation involving NULL yields NULL.
- This applies to all numeric data types (INT, DECIMAL, FLOAT, etc.).
- Date/time arithmetic also propagates NULL (e.g., `DATE '2024-01-01' + NULL` = NULL).
- String concatenation with NULL varies: `||` in standard SQL returns NULL, but MySQL's `CONCAT()` treats NULL as empty string.

**Constraints and Limitations:**

- NULL propagation cannot be disabled in standard SQL.
- Some databases have functions that treat NULL as 0 (e.g., `IFNULL`, `COALESCE`), but these are not automatic.
- Integer division by NULL yields NULL (not an error, unlike division by 0).

**Annotated Code Examples:**

```sql
-- Creating a table with numeric data including NULLs
CREATE TABLE sales (
    sale_id INT PRIMARY KEY,
    quantity INT,
    unit_price DECIMAL(10,2),
    discount DECIMAL(5,2)
);

INSERT INTO sales VALUES (1, 10, 25.00, 0.10);
INSERT INTO sales VALUES (2, NULL, 25.00, 0.10);
INSERT INTO sales VALUES (3, 10, NULL, 0.10);
INSERT INTO sales VALUES (4, 10, 25.00, NULL);
INSERT INTO sales VALUES (5, 5, 50.00, NULL);

-- Demonstrating NULL propagation in arithmetic
SELECT 
    sale_id,
    quantity,
    unit_price,
    discount,
    quantity * unit_price AS gross_amount,           -- NULL if either is NULL
    quantity * unit_price * (1 - discount) AS net_amount  -- NULL if any is NULL
FROM sales;
```

**Expected Output:**

| sale_id | quantity | unit_price | discount | gross_amount | net_amount |
|---------|----------|------------|----------|--------------|------------|
| 1       | 10       | 25.00      | 0.10     | 250.00       | 225.00     |
| 2       | NULL     | 25.00      | 0.10     | NULL         | NULL       |
| 3       | 10       | NULL       | 0.10     | NULL         | NULL       |
| 4       | 10       | 25.00      | NULL     | 250.00       | NULL       |
| 5       | 5        | 50.00      | NULL     | 250.00       | NULL       |

**Explanation:** Sale 2 has NULL quantity, so both gross and net amounts are NULL. Sale 3 has NULL unit_price, so both calculations are NULL. Sales 4 and 5 have NULL discount, so gross amount is calculated (discount not used), but net amount is NULL because the discount is used in that calculation.

```sql
-- Using COALESCE to handle NULL propagation
SELECT 
    sale_id,
    COALESCE(quantity, 0) * COALESCE(unit_price, 0) AS gross_with_default,
    COALESCE(quantity * unit_price, 0) AS gross_with_null_handling
FROM sales;
```

**Expected Output:**

| sale_id | gross_with_default | gross_with_null_handling |
|---------|-------------------|--------------------------|
| 1       | 250.00            | 250.00                   |
| 2       | 0.00              | 0.00                     |
| 3       | 0.00              | 0.00                     |
| 4       | 250.00            | 250.00                   |
| 5       | 250.00            | 250.00                   |

**Explanation:** The first approach replaces NULLs with 0 before multiplying. The second approach performs the multiplication first (which yields NULL) and then replaces NULL with 0. These can produce different results if you want to distinguish between "zero" and "unknown."

**Real-World Cases:**

- **Financial calculations:** If a component of a total is unknown, the total should be unknown (NULL), not zero.
- **Inventory:** If the quantity on hand is unknown, the total value of inventory is unknown.
- **Payroll:** If an employee's hours are NULL, their pay should be NULL (not zero) to flag the missing data.

**References:**

- Joe Celko's SQL for Smarties: Math and NULLs - https://www.oreilly.com/library/view/joe-celkos-sql/9780128007617/
- ScienceDirect: NULLs: Missing Data in SQL - https://www.sciencedirect.com/topics/computer-science/null-values
- Crunchy Data: Postgres Calculations and the Ambiguity of NULL - https://www.crunchydata.com/blog/postgres-calculations-and-the-ambiguity-of-null


#### 3.2 NULL Behavior in Aggregate Functions

**Core Definitions:**

- **Core Definition:** Aggregate functions in SQL handle NULL values in specific ways: most ignore NULLs, while `COUNT(*)` includes them.
- **Technical Definition:** Per SQL standard and common implementations, aggregate functions such as `SUM`, `AVG`, `MIN`, and `MAX` skip NULL values in their calculations. `COUNT(*)` counts all rows (including those with NULLs), while `COUNT(column)` counts only non-NULL values in the specified column.
- **Beginner-Friendly Explanation:** If you're averaging test scores and one student didn't take the test (NULL score), you don't count that student in the average. But if you're counting how many students are in the class, you count everyone.

**Purposes:**

- To ensure aggregate calculations are based on available data
- To distinguish between "no data" and "zero" in aggregations
- To provide accurate summaries when some values are missing
- To allow flexible counting strategies (all rows vs. non-NULL values)

**Syntax Structures and Rules:**

**Complete General Syntaxes:**

```sql
-- COUNT(*) - counts all rows including NULLs
COUNT(*)

-- COUNT(column) - counts non-NULL values in column
COUNT(column_name)

-- COUNT(DISTINCT column) - counts distinct non-NULL values
COUNT(DISTINCT column_name)

-- SUM, AVG, MIN, MAX - ignore NULLs
SUM(column_name)
AVG(column_name)
MIN(column_name)
MAX(column_name)
```

**Component Breakdown:**

- `COUNT(*)`: Counts every row in the result set, regardless of NULL values
- `COUNT(column_name)`: Counts only rows where `column_name` is NOT NULL
- `COUNT(DISTINCT column_name)`: Counts distinct non-NULL values
- `SUM(column_name)`: Sums all non-NULL values in the column
- `AVG(column_name)`: Sums all non-NULL values and divides by the count of non-NULL values
- `MIN(column_name)`: Returns the smallest non-NULL value
- `MAX(column_name)`: Returns the largest non-NULL value

**Syntax Rules:**

- All aggregate functions except `COUNT(*)` ignore NULLs.
- `SUM` returns NULL if all values are NULL.
- `AVG` returns NULL if all values are NULL.
- `MIN` and `MAX` return NULL if all values are NULL.
- `COUNT(column)` returns 0 if all values are NULL.

**Constraints and Limitations:**

- `COUNT(*)` includes rows where every column is NULL.
- `COUNT(1)` and `COUNT(*)` behave identically.
- Some databases allow `COUNT(DISTINCT expression)` with multiple columns (implementation-dependent).
- `SUM` of an empty set returns NULL, not 0.

**Annotated Code Examples:**

```sql
-- Creating a table with NULL values
CREATE TABLE test_scores (
    student_id INT PRIMARY KEY,
    student_name VARCHAR(50),
    score INT
);

INSERT INTO test_scores VALUES (1, 'Alice', 85);
INSERT INTO test_scores VALUES (2, 'Bob', 90);
INSERT INTO test_scores VALUES (3, 'Charlie', NULL);
INSERT INTO test_scores VALUES (4, 'Diana', 75);
INSERT INTO test_scores VALUES (5, 'Eve', NULL);
INSERT INTO test_scores VALUES (6, 'Frank', 85);

-- Demonstrating aggregate behavior with NULLs
SELECT 
    COUNT(*) AS total_rows,                    -- Counts all rows (6)
    COUNT(score) AS non_null_scores,           -- Counts non-NULL scores (4)
    SUM(score) AS sum_scores,                  -- Sum of non-NULL scores (335)
    AVG(score) AS avg_scores,                  -- Average of non-NULL scores (83.75)
    MIN(score) AS min_score,                   -- Minimum non-NULL score (75)
    MAX(score) AS max_score,                   -- Maximum non-NULL score (90)
    COUNT(DISTINCT score) AS distinct_scores   -- Distinct non-NULL scores (3)
FROM test_scores;
```

**Expected Output:**

| total_rows | non_null_scores | sum_scores | avg_scores | min_score | max_score | distinct_scores |
|------------|-----------------|------------|------------|-----------|-----------|-----------------|
| 6          | 4               | 335        | 83.75      | 75        | 90        | 3               |

**Explanation:** 
- `COUNT(*)` = 6 (all students)
- `COUNT(score)` = 4 (only students with scores)
- `SUM(score)` = 85 + 90 + 75 + 85 = 335
- `AVG(score)` = 335 / 4 = 83.75 (not 335 / 6 = 55.83)
- `MIN(score)` = 75
- `MAX(score)` = 90
- `COUNT(DISTINCT score)` = 3 (75, 85, 90)

```sql
-- Demonstrating the empty set behavior
DELETE FROM test_scores WHERE score IS NOT NULL;

SELECT 
    COUNT(*) AS total_rows,
    COUNT(score) AS non_null_scores,
    SUM(score) AS sum_scores,
    AVG(score) AS avg_scores
FROM test_scores;
```

**Expected Output:**

| total_rows | non_null_scores | sum_scores | avg_scores |
|------------|-----------------|------------|------------|
| 2          | 0               | NULL       | NULL       |

**Explanation:** After deleting all rows with non-NULL scores, only the two rows with NULL scores remain. `COUNT(*)` returns 2, `COUNT(score)` returns 0, and `SUM` and `AVG` return NULL (not 0) because there are no non-NULL values to aggregate.

**Real-World Cases:**

- **Survey analysis:** Counting respondents who answered a question (`COUNT(column)`) vs. total respondents (`COUNT(*)`).
- **Financial reporting:** Averaging revenue per transaction, excluding transactions with missing amounts.
- **Healthcare:** Calculating average patient age, excluding patients with unknown birth dates.
- **Inventory:** Summing quantities on hand, treating NULL as "not counted" rather than zero.

**References:**

- Microsoft Learn: Use Aggregate Functions - https://learn.microsoft.com/en-us/training/modules/use-built-functions-transact-sql/5-aggregate-functions
- SAP Help Portal: NULLs in Aggregate Functions - https://help.sap.com/doc/saphelp_ase160/16.0/en-US/1a/efc6c0bc2b10148c308d2d00a6b4c2/content.htm
- OceanBase: NULLs in SQL Functions - https://en.oceanbase.com/docs/common-oceanbase-database-10000000001710095


#### 3.3 NULL Behavior in Sorting (ORDER BY)

**Core Definitions:**

- **Core Definition:** NULL values have special sorting behavior in SQL's ORDER BY clause, with explicit control via `NULLS FIRST` and `NULLS LAST` options.
- **Technical Definition:** By default, NULL values sort as if they are larger than any non-NULL value. This means that with ascending (ASC) order, NULLs appear last; with descending (DESC) order, NULLs appear first. The SQL standard allows explicit control using `NULLS FIRST` or `NULLS LAST`.
- **Beginner-Friendly Explanation:** When sorting a list, NULLs need to go somewhere. By default, they're treated as the "biggest" values, so they come last in ascending order and first in descending order. But you can change this with explicit instructions.

**Purposes:**

- To provide predictable ordering when NULLs are present
- To allow control over where NULLs appear in sorted results
- To ensure consistent behavior across different queries and databases

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
SELECT column_list
FROM table_name
ORDER BY column_name [ASC | DESC] [NULLS { FIRST | LAST }]
[, column_name2 [ASC | DESC] [NULLS { FIRST | LAST }] ...];
```

**Component Breakdown:**

- `ORDER BY`: Clause that specifies sorting
- `column_name`: The column to sort by
- `ASC | DESC`: Sort direction (ASC is default)
- `NULLS FIRST`: Place NULLs before non-NULL values
- `NULLS LAST`: Place NULLs after non-NULL values

**Syntax Rules:**

- Default behavior: NULLs sort as if larger than any non-NULL value.
- ASC order: NULLs appear last (default `NULLS LAST`).
- DESC order: NULLs appear first (default `NULLS FIRST`).
- The `NULLS FIRST` and `NULLS LAST` options can be used independently for each sort column.
- Not all databases support `NULLS FIRST`/`NULLS LAST` (e.g., MySQL does not; PostgreSQL does).

**Constraints and Limitations:**

- MySQL does not support `NULLS FIRST`/`NULLS LAST` syntax; use `ORDER BY column IS NULL, column` as a workaround.
- SQL Server does not support `NULLS FIRST`/`NULLS LAST` syntax; use `CASE` expressions as a workaround.
- The SQL standard specifies `NULLS FIRST`/`NULLS LAST` as optional features.

**Annotated Code Examples:**

```sql
-- Creating a table with NULL values for sorting
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10,2)
);

INSERT INTO products VALUES (1, 'Widget', 19.99);
INSERT INTO products VALUES (2, 'Gadget', 29.99);
INSERT INTO products VALUES (3, 'Gizmo', NULL);
INSERT INTO products VALUES (4, 'Doohickey', 9.99);
INSERT INTO products VALUES (5, 'Thingamajig', NULL);
INSERT INTO products VALUES (6, 'Contraption', 49.99);

-- Default ASC ordering (NULLs last)
SELECT product_name, price
FROM products
ORDER BY price ASC;
```

**Expected Output (PostgreSQL):**

| product_name | price |
|--------------|-------|
| Doohickey    | 9.99  |
| Widget       | 19.99 |
| Gadget       | 29.99 |
| Contraption  | 49.99 |
| Gizmo        | NULL  |
| Thingamajig  | NULL  |

```sql
-- Default DESC ordering (NULLs first)
SELECT product_name, price
FROM products
ORDER BY price DESC;
```

**Expected Output:**

| product_name | price |
|--------------|-------|
| Gizmo        | NULL  |
| Thingamajig  | NULL  |
| Contraption  | 49.99 |
| Gadget       | 29.99 |
| Widget       | 19.99 |
| Doohickey    | 9.99  |

```sql
-- Explicit NULLS LAST with DESC
SELECT product_name, price
FROM products
ORDER BY price DESC NULLS LAST;
```

**Expected Output:**

| product_name | price |
|--------------|-------|
| Contraption  | 49.99 |
| Gadget       | 29.99 |
| Widget       | 19.99 |
| Doohickey    | 9.99  |
| Gizmo        | NULL  |
| Thingamajig  | NULL  |

```sql
-- Explicit NULLS FIRST with ASC
SELECT product_name, price
FROM products
ORDER BY price ASC NULLS FIRST;
```

**Expected Output:**

| product_name | price |
|--------------|-------|
| Gizmo        | NULL  |
| Thingamajig  | NULL  |
| Doohickey    | 9.99  |
| Widget       | 19.99 |
| Gadget       | 29.99 |
| Contraption  | 49.99 |

**Workaround for MySQL/SQL Server (no NULLS FIRST/LAST):**

```sql
-- MySQL: NULLs last in ASC order
SELECT product_name, price
FROM products
ORDER BY price IS NULL, price ASC;

-- MySQL: NULLs first in ASC order
SELECT product_name, price
FROM products
ORDER BY price IS NOT NULL, price ASC;

-- SQL Server: NULLs last in ASC order
SELECT product_name, price
FROM products
ORDER BY CASE WHEN price IS NULL THEN 1 ELSE 0 END, price ASC;
```

**Real-World Cases:**

- **E-commerce:** Sorting products by price, with "price not available" products appearing last.
- **HR systems:** Sorting employees by hire date, with employees who have no hire date appearing at the end.
- **Financial reports:** Sorting transactions by amount, with transactions missing amounts appearing last.
- **Data migration:** Sorting data to identify which records have missing values.

**References:**

- PostgreSQL Documentation: Sorting Rows (ORDER BY) - https://www.postgresql.org/docs/17/queries-order.html
- Amazon Redshift: ORDER BY Clause - https://docs.aws.amazon.com/redshift/latest/dg/r_ORDER_BY_clause.html
- SQL Server: ORDER BY Clause - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-order-by-clause-transact-sql


### 4. Conditional & Substitute Functions

#### 4.1 Standard ANSI SQL: COALESCE

**Core Definitions:**

- **Core Definition:** COALESCE is a standard SQL function that returns the first non-NULL expression from a list of expressions.
- **Technical Definition:** COALESCE evaluates its arguments in order and returns the current value of the first expression that does not evaluate to NULL. If all arguments are NULL, COALESCE returns NULL. It is equivalent to a CASE expression and is part of the SQL standard.
- **Beginner-Friendly Explanation:** COALESCE is like a backup plan. You give it a list of options, and it picks the first one that actually has a value. If all are empty, you get nothing (NULL).

**Purposes:**

- To provide a default value when a column contains NULL
- To combine multiple columns where any might be NULL
- To simplify queries by avoiding complex CASE expressions
- To ensure calculations and displays have fallback values

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
COALESCE(expression1, expression2, ..., expressionN)
```

**Component Breakdown:**

- `expression1, expression2, ..., expressionN`: A list of expressions of compatible data types
- Returns: The first non-NULL expression value, or NULL if all are NULL
- Minimum arguments: 2

**Syntax Rules:**

- At least two expressions must be provided.
- All expressions should be of compatible data types; the result takes the data type of the expression with the highest precedence.
- Evaluation is short-circuited: once a non-NULL value is found, subsequent expressions are not evaluated (though some databases may evaluate all expressions for performance reasons).
- COALESCE is a standard SQL function, supported by all major databases.

**Constraints and Limitations:**

- In SQL Server, COALESCE with subqueries may evaluate the subquery multiple times, which can lead to different results under certain isolation levels.
- The result data type is determined by data type precedence; implicit conversions may occur.
- COALESCE cannot be used with `DEFAULT` keyword as an argument in most databases.

**Examples 1: Creating a table with NULL values**

```sql
CREATE TABLE contact_info (
    contact_id INT PRIMARY KEY,
    name VARCHAR(100),
    home_phone VARCHAR(20),
    work_phone VARCHAR(20),
    mobile_phone VARCHAR(20)
);

INSERT INTO contact_info VALUES (1, 'Alice', '555-0100', '555-0200', '555-0300');
INSERT INTO contact_info VALUES (2, 'Bob', NULL, '555-0400', NULL);
INSERT INTO contact_info VALUES (3, 'Charlie', '555-0500', NULL, NULL);
INSERT INTO contact_info VALUES (4, 'Diana', NULL, NULL, NULL);

-- Using COALESCE to find the best phone number
SELECT 
    name,
    COALESCE(home_phone, work_phone, mobile_phone, 'No phone') AS best_phone
FROM contact_info;
```

**Expected Output:**

| name    | best_phone |
|---------|------------|
| Alice   | 555-0100   |
| Bob     | 555-0400   |
| Charlie | 555-0500   |
| Diana   | No phone   |

**Explanation:** COALESCE returns the first non-NULL phone number in the order: home, work, mobile. For Diana, all are NULL, so it returns the default 'No phone'.

**Example 2: COALESCE with numeric calculations**
```sql
CREATE TABLE budgets (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(50),
    current_year DECIMAL(10,2),
    previous_year DECIMAL(10,2)
);

INSERT INTO budgets VALUES (1, 'Engineering', 100000, 90000);
INSERT INTO budgets VALUES (2, 'Marketing', NULL, 80000);
INSERT INTO budgets VALUES (3, 'Sales', 120000, NULL);
INSERT INTO budgets VALUES (4, 'HR', NULL, NULL);

-- Calculate budget with fallback
SELECT 
    dept_name,
    COALESCE(current_year, previous_year, 0) AS budget
FROM budgets;
```

**Expected Output:**

| dept_name   | budget  |
|-------------|---------|
| Engineering | 100000  |
| Marketing   | 80000   |
| Sales       | 120000  |
| HR          | 0       |

**Real-World Cases:**

- **Contact management:** Finding the best phone number or email from multiple sources.
- **Financial reporting:** Using actual values with fallback to budgeted or estimated values.
- **E-commerce:** Displaying product price with fallback to MSRP or "Call for price."
- **Data migration:** Providing default values when source data has NULLs.

**References:**

- Microsoft Learn: COALESCE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/coalesce-transact-sql
- Oracle: COALESCE - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/COALESCE.html
- PostgreSQL: COALESCE - https://www.postgresql.org/docs/current/functions-conditional.html


#### 4.2 Standard ANSI SQL: NULLIF

**Core Definitions:**

- **Core Definition:** NULLIF is a standard SQL function that returns NULL if two expressions are equal, otherwise returns the first expression.
- **Technical Definition:** NULLIF(expression1, expression2) returns NULL if expression1 equals expression2; otherwise, it returns expression1. It is the inverse of COALESCE in the sense that it introduces NULLs rather than removing them. It is equivalent to a CASE expression.
- **Beginner-Friendly Explanation:** NULLIF is like a filter that says "if this value is the same as that value, treat it as if it doesn't exist (NULL)." It's useful for hiding values that are the same as a default or placeholder.

**Purposes:**

- To convert a specific value (often 0 or a placeholder) to NULL
- To avoid division by zero errors
- To treat "no change" or "default" values as unknown
- To conditionally set values to NULL based on comparison

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
NULLIF(expression1, expression2)
```

**Component Breakdown:**

- `expression1`: The expression to return if not equal to expression2
- `expression2`: The expression to compare against
- Returns: NULL if `expression1 = expression2`; otherwise returns `expression1`

**Syntax Rules:**

- Both expressions must be of comparable data types.
- The return type is the same as the first expression.
- NULLIF is equivalent to:
  ```sql
  CASE WHEN expression1 = expression2 THEN NULL ELSE expression1 END
  ```
- If expression1 is NULL, NULLIF returns NULL (because NULL = expression2 is UNKNOWN, and NULLIF treats UNKNOWN as not equal, returning the first expression... actually, let's verify: SQL standard says NULLIF returns NULL if the first argument equals the second, otherwise returns the first. If the first is NULL, the comparison is UNKNOWN, so it returns the first argument (NULL). So NULLIF(NULL, value) returns NULL.)

**Constraints and Limitations:**

- Time-dependent functions (like `RAND()`) should not be used within NULLIF, as they may be evaluated twice and return different results.
- The result type is the same as the first expression.

**Annotated Code Examples:**

```sql
-- Creating a table with placeholder values
CREATE TABLE employee_bonus (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100),
    bonus DECIMAL(10,2),
    salary DECIMAL(10,2)
);

INSERT INTO employee_bonus VALUES (1, 'Alice', 5000, 80000);
INSERT INTO employee_bonus VALUES (2, 'Bob', 0, 60000);
INSERT INTO employee_bonus VALUES (3, 'Charlie', NULL, 90000);
INSERT INTO employee_bonus VALUES (4, 'Diana', 0, 70000);

-- Using NULLIF to treat 0 bonus as NULL (unknown)
SELECT 
    emp_name,
    bonus,
    NULLIF(bonus, 0) AS bonus_null_if_zero,
    salary + NULLIF(bonus, 0) AS total_comp
FROM employee_bonus;
```

**Expected Output:**

| emp_name | bonus | bonus_null_if_zero | total_comp |
|----------|-------|-------------------|------------|
| Alice    | 5000  | 5000              | 85000      |
| Bob      | 0     | NULL              | NULL       |
| Charlie  | NULL  | NULL              | NULL       |
| Diana    | 0     | NULL              | NULL       |

**Explanation:** NULLIF(bonus, 0) converts 0 bonuses to NULL. For Bob and Diana, this makes total_comp NULL, indicating the bonus is unknown or not applicable (rather than zero).

```sql
-- Using NULLIF to avoid division by zero
CREATE TABLE ratios (
    id INT PRIMARY KEY,
    numerator DECIMAL(10,2),
    denominator DECIMAL(10,2)
);

INSERT INTO ratios VALUES (1, 100, 10);
INSERT INTO ratios VALUES (2, 100, 0);
INSERT INTO ratios VALUES (3, 100, NULL);

SELECT 
    id,
    numerator,
    denominator,
    numerator / NULLIF(denominator, 0) AS ratio
FROM ratios;
```

**Expected Output:**

| id | numerator | denominator | ratio |
|----|-----------|-------------|-------|
| 1  | 100       | 10          | 10    |
| 2  | 100       | 0           | NULL  |
| 3  | 100       | NULL        | NULL  |

**Explanation:** NULLIF(denominator, 0) converts 0 to NULL, preventing a division by zero error. The result is NULL for rows where the denominator is 0 or NULL.

**Real-World Cases:**

- **Financial reporting:** Treating 0 changes as NULL (unknown) in variance analysis.
- **Data cleaning:** Converting placeholder values (like 0 or -1) to NULL.
- **Safe division:** Preventing division by zero errors in calculations.
- **Survey data:** Treating "no response" coded as 0 as NULL (unknown).

**References:**

- Microsoft Learn: NULLIF (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/nullif-transact-sql
- Oracle: NULLIF - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/NULLIF.html
- PostgreSQL: NULLIF - https://www.postgresql.org/docs/current/functions-conditional.html


#### 4.3 Vendor-Specific Alternatives (IFNULL in MySQL, NVL in Oracle, ISNULL in SQL Server)

**Core Definitions:**

- **Core Definition:** IFNULL, NVL, and ISNULL are vendor-specific functions that serve the same purpose as COALESCE (returning a fallback value for NULL) but with only two arguments.
- **Technical Definition:** These functions take two arguments and return the first if it is not NULL, otherwise the second. They are not part of the ANSI SQL standard and are provided as convenience functions by specific database vendors.
- **Beginner-Friendly Explanation:** These are like simplified versions of COALESCE that only take two options. Different databases have different names for the same idea.

**Purposes:**

- To provide a simple, two-argument fallback for NULL values
- To offer database-specific optimizations not available in COALESCE
- To maintain compatibility with existing code written for specific database platforms

**Syntax Structures and Rules:**

**MySQL - IFNULL:**

```sql
IFNULL(expr1, expr2)
```

- Returns `expr1` if not NULL, otherwise `expr2`
- The default result type is the more "general" of the two expressions (STRING, REAL, or INTEGER).

**Oracle - NVL:**

```sql
NVL(expr1, expr2)
```

- Returns `expr1` if not NULL, otherwise `expr2`
- Arguments must be of the same data type or implicitly convertible.

**SQL Server - ISNULL:**

```sql
ISNULL(check_expression, replacement_value)
```

- Returns `check_expression` if not NULL, otherwise `replacement_value`
- The return type is the same as `check_expression`. If `check_expression` is a literal NULL, the return type is the data type of `replacement_value`.

**Syntax Rules:**

- All three functions take exactly two arguments.
- All three return the first argument if it is not NULL.
- All three return the second argument if the first is NULL.
- These are vendor-specific and not portable across database systems.

**Constraints and Limitations:**

- **Portability:** Code using these functions is not portable to other database systems.
- **Data type handling:** Each vendor has specific rules for implicit conversion and return types.
- **Deprecation:** None of these are deprecated, but COALESCE is recommended for new development to maximize portability.

**Annotated Code Examples:**

```sql
-- MySQL: Using IFNULL
CREATE TABLE mysql_products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10,2)
);

INSERT INTO mysql_products VALUES (1, 'Widget', 19.99);
INSERT INTO mysql_products VALUES (2, 'Gadget', NULL);
INSERT INTO mysql_products VALUES (3, 'Gizmo', 9.99);

SELECT 
    product_name,
    IFNULL(price, 0.00) AS price_with_default
FROM mysql_products;
```

**Expected Output (MySQL):**

| product_name | price_with_default |
|--------------|-------------------|
| Widget       | 19.99             |
| Gadget       | 0.00              |
| Gizmo        | 9.99              |

```sql
-- Oracle: Using NVL
CREATE TABLE oracle_employees (
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR2(100),
    commission_pct NUMBER(5,2)
);

INSERT INTO oracle_employees VALUES (1, 'Alice', 0.10);
INSERT INTO oracle_employees VALUES (2, 'Bob', NULL);
INSERT INTO oracle_employees VALUES (3, 'Charlie', 0.15);

SELECT 
    employee_name,
    NVL(TO_CHAR(commission_pct), 'Not Applicable') AS commission
FROM oracle_employees;
```

**Expected Output (Oracle):**

| employee_name | commission     |
|---------------|----------------|
| Alice         | .1             |
| Bob           | Not Applicable |
| Charlie       | .15            |

```sql
-- SQL Server: Using ISNULL
CREATE TABLE sqlserver_orders (
    order_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    ship_date DATE
);

INSERT INTO sqlserver_orders VALUES (1, 'Alice', '2024-01-15');
INSERT INTO sqlserver_orders VALUES (2, 'Bob', NULL);
INSERT INTO sqlserver_orders VALUES (3, 'Charlie', '2024-02-20');

SELECT 
    order_id,
    customer_name,
    ISNULL(CONVERT(VARCHAR, ship_date, 120), 'Not Shipped') AS ship_status
FROM sqlserver_orders;
```

**Expected Output (SQL Server):**

| order_id | customer_name | ship_status |
|----------|---------------|-------------|
| 1        | Alice         | 2024-01-15  |
| 2        | Bob           | Not Shipped |
| 3        | Charlie       | 2024-02-20  |

**Real-World Cases:**

- **Cross-database development:** Understanding these differences when writing database-agnostic code.
- **Legacy system maintenance:** Encountering these functions in existing code written for specific platforms.
- **Performance optimization:** In some databases, the vendor-specific function may perform better than COALESCE.

**References:**

- MySQL Reference Manual: Flow Control Functions - https://dev.mysql.com/doc/refman/9.7/en/flow-control-functions.html
- Oracle Database SQL Reference: NVL - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/NVL.html
- Microsoft Learn: ISNULL (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/isnull-transact-sql
- jOOQ: NVL, ISNULL, IFNULL - https://www.jooq.org/doc/latest/manual/sql-building/column-expressions/null-handling-functions/


## Summary of NULL Handling Functions

| Function | Standard | Arguments | Behavior | Portability |
|----------|----------|-----------|----------|-------------|
| COALESCE | ANSI SQL | 2 or more | Returns first non-NULL | All databases |
| NULLIF | ANSI SQL | 2 | Returns NULL if equal, else first | All databases |
| IFNULL | MySQL | 2 | Returns first if not NULL, else second | MySQL, some others |
| NVL | Oracle | 2 | Returns first if not NULL, else second | Oracle, some others |
| ISNULL | SQL Server | 2 | Returns first if not NULL, else second | SQL Server, some others |

---

## References

### Official Documentation and Standards

- Microsoft Learn: Null Values - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/null-and-unknown-transact-sql
- Microsoft Learn: COALESCE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/coalesce-transact-sql
- Microsoft Learn: NULLIF (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/nullif-transact-sql
- Microsoft Learn: ISNULL (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/isnull-transact-sql
- Oracle Database SQL Reference: Nulls - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Nulls.html
- Oracle Database SQL Reference: NVL - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/NVL.html
- Oracle Database SQL Reference: COALESCE - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/COALESCE.html
- Oracle Database SQL Reference: NULLIF - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/NULLIF.html
- MySQL Reference Manual: Flow Control Functions - https://dev.mysql.com/doc/refman/9.7/en/flow-control-functions.html
- PostgreSQL Documentation: Sorting Rows (ORDER BY) - https://www.postgresql.org/docs/17/queries-order.html
- PostgreSQL Documentation: Comparison Operators - https://www.postgresql.org/docs/current/functions-comparison.html
- Snowflake Documentation: Ternary Logic - https://docs.snowflake.com/en/sql-reference/ternary-logic
- ISO/IEC 9075-2:2016 (SQL Standard) - https://www.iso.org/standard/63555.html

### Academic and Technical Sources

- Joe Celko's SQL for Smarties (3rd Edition) - https://www.oreilly.com/library/view/joe-celkos-sql/9780128007617/
- ScienceDirect: NULLs: Missing Data in SQL - https://www.sciencedirect.com/topics/computer-science/null-values
- Navicat: Comparing the Semantics of Null, Zero, and Empty String - https://www.navicat.com/en/company/aboutus/blog/1796-null-zero-empty-string.html
- Crunchy Data: Postgres Calculations and the Ambiguity of NULL - https://www.crunchydata.com/blog/postgres-calculations-and-the-ambiguity-of-null
- SAP Help Portal: Three-valued Logic - https://help.sap.com/doc/abapdocu_752_index_htm/7.52/en-US/abenthree_valued_logic.htm
- jOOQ: Null Handling Functions - https://www.jooq.org/doc/latest/manual/sql-building/column-expressions/null-handling-functions/