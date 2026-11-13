# SQL Casting and Conversion: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
SQL casting and conversion are the mechanisms by which a value of one data type is transformed into a value of another data type, either explicitly through functions such as `CAST` and `CONVERT`, or implicitly by the database engine according to predefined rules.

**Technical Definition**
In SQL, type conversion encompasses the processes and rules that govern how the database engine reconciles operands of differing data types in expressions, assignments, and function calls. The SQL standard defines `CAST` as the primary explicit conversion operator. Each RDBMS implements a type system with a precedence hierarchy that determines which type an expression will resolve to when operands differ, and a set of implicit conversion rules that permit certain conversions without explicit syntax. Explicit conversions use `CAST` or `CONVERT` (vendor-specific); implicit conversions occur automatically and can affect both query results and performance characteristics such as index usage.

**Beginner-Friendly Explanation**
Sometimes a database needs to treat a number as text, or a piece of text as a date. Casting and conversion are how you tell the database to change a value from one type to another. You can do it explicitly (by asking for it) or let the database do it automatically (implicitly)—but automatic conversions can sometimes cause surprises.

---

### Key Characteristics

1. **Explicit vs. Implicit**: Explicit conversions are requested by the developer using `CAST` or `CONVERT`; implicit conversions are performed automatically by the engine when operands of different types appear in an expression.
2. **Data Type Precedence**: SQL Server uses a documented precedence order (e.g., `nvarchar` > `varchar`, `float` > `int`) to decide which operand to convert when types differ. PostgreSQL and MySQL have their own resolution rules.
3. **Performance Impact of Implicit Conversion**: Comparing an indexed column to a literal of a different type can force the engine to convert the column on a per-row basis, preventing index seeks and causing full scans—a well-documented performance pitfall.
4. **Vendor-Specific Syntax and Behaviour**: `CAST` is SQL-standard; `CONVERT` is SQL Server-specific (though MySQL also has a `CONVERT` function with different semantics). The set of allowed implicit conversions varies by RDBMS.
5. **Lossy vs. Information-Preserving Conversions**: Some conversions preserve all information (e.g., `SMALLINT` → `INTEGER`), while others may truncate, round, or fail (e.g., `FLOAT` → `INTEGER`, long string → short string).
6. **NULL Propagation**: Converting `NULL` generally yields `NULL` (with the exception of certain vendor-specific behaviours or `TRY_CAST` variants that return `NULL` on failure).

---

### Prerequisites

- **Numeric, string, and temporal data types** and their ranges.
- **Data type precedence** in your specific RDBMS.
- **SQL expression evaluation** and operator resolution.
- **Index fundamentals** (B-tree seeks vs. scans) to understand performance implications.

---

### Related Programming Areas

| Area | Connection |
|------|------------|
| **Query Performance Tuning** | Implicit conversions are a leading cause of index scans and CPU overhead. |
| **Data Integration / ETL** | Source and target systems often use different types; explicit casting ensures correctness. |
| **Application Development** | ORMs and parameterised queries can trigger implicit conversions (e.g., sending `NVARCHAR` to a `VARCHAR` column). |
| **Data Quality** | Lossy conversions can silently truncate or round data. |
| **Cross-Platform Portability** | Conversion syntax and rules differ across PostgreSQL, MySQL, and SQL Server. |

---

### Core Concepts / Features

1. **`CAST`**
2. **`CONVERT`**
3. **Type Conversion (General)**
4. **Implicit Conversion**
5. **Explicit Conversion**
6. **Conversion Pitfalls**

---

## 1. `CAST`

### Definitions

**Core Definition**
`CAST` is the SQL-standard operator for explicitly converting an expression from one data type to another.

**Technical Definition**
`CAST(expression AS target_type)` invokes a registered cast function (or binary coercion, where types are binary-compatible) to convert `expression` to `target_type`. In PostgreSQL, `CAST` is equivalent to the `::` shorthand; if no suitable cast is defined, the conversion fails. SQL Server implements `CAST` as a built-in function that follows its data type precedence and conversion matrix.

**Beginner-Friendly Explanation**
`CAST` is the standard way to say “treat this value as a different type.” You write `CAST(value AS new_type)`, and the database performs the conversion.

---

### Purposes

- To explicitly convert a value when implicit conversion is not allowed or not desirable.
- To control the result type of an expression (e.g., casting an integer to `NUMERIC` for division).
- To ensure portability across RDBMS by using standard syntax.
- To avoid the performance pitfalls of implicit conversion by making conversions explicit on the literal/parameter side rather than the column side.

---

### Syntax Rules and Structure

**SQL Standard / PostgreSQL / SQL Server**
```
CAST ( expression AS target_type )
```
**PostgreSQL Alternative**
```
expression :: target_type
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `CAST` | Keyword. |
| `expression` | Any value expression. |
| `AS` | Keyword. |
| `target_type` | The desired data type. |
| `::` (PostgreSQL) | Shorthand cast operator. |

**Syntax Rules**

1. The target type must be a valid type name in the RDBMS.
2. If a cast is not defined between the source and target types, the statement fails (PostgreSQL) or raises a conversion error (SQL Server).
3. `CAST(NULL AS type)` returns `NULL`.
4. In PostgreSQL, `CAST` and `::` are functionally identical; `::` is PostgreSQL-specific.

**Constraints and Limitations**

- Not all type pairs have defined casts; cross-category conversions (e.g., `text` → `integer`) are typically explicit-only in PostgreSQL.
- SQL Server’s `CAST` follows its conversion matrix; some conversions are not permitted even explicitly.

---

### Annotated Code Examples

**Example 1: Casting Integer to Numeric for Division (PostgreSQL)**

```sql
SELECT CAST(7 AS NUMERIC) / 2 AS result;
-- Or equivalently:
SELECT 7::NUMERIC / 2 AS result;
```

**Step-by-Step Explanation**

1. `CAST(7 AS NUMERIC)` converts the integer `7` to `7.0` (numeric).
2. Dividing numeric `7.0` by `2` yields `3.5` (numeric division), whereas integer division `7 / 2` would yield `3`.

**Expected Output**

| result |
|--------|
| 3.5    |

**Why This Output Occurs**: The explicit cast changes the arithmetic from integer division to numeric division.

---

**Example 2: Standard `CAST` in SQL Server**

```sql
SELECT CAST('2026-11-13' AS DATE) AS parsed_date;
```

**Expected Output**

| parsed_date |
|-------------|
| 2026-11-13  |

**Why This Output Occurs**: `CAST` parses the string according to SQL Server’s date format rules and returns a `DATE` value.

---

**Example 3: PostgreSQL `::` Shorthand**

```sql
SELECT '42'::INTEGER AS answer;
```

**Expected Output**

| answer |
|--------|
| 42     |

**Why This Output Occurs**: The `::` syntax is PostgreSQL’s shorthand for `CAST`; it converts the string literal `'42'` to an integer.

---

### Real-World Cases

- **Division precision**: `CAST(total AS NUMERIC) / COUNT(*)` prevents integer truncation.
- **Date parsing**: `CAST(string_column AS DATE)` standardises date strings.
- **Data export**: `CAST(id AS VARCHAR)` when concatenating numeric IDs with strings.

---

## 2. `CONVERT`

### Definitions

**Core Definition**
`CONVERT` is a vendor-specific function for explicit type conversion, most prominently in SQL Server, where it additionally supports style codes for formatting date/time and numeric values.

**Technical Definition**
In SQL Server, `CONVERT(data_type, expression [, style])` converts `expression` to `data_type`, optionally using a `style` integer to control formatting (especially for date/time and money types). In MySQL, `CONVERT(expr, type)` or `CONVERT(expr USING charset)` serves different purposes: the first is type conversion, the second is character set conversion.

**Beginner-Friendly Explanation**
`CONVERT` does the same job as `CAST`—change a value’s type—but in SQL Server you can also supply a style number to format the result (e.g., `103` for `dd/mm/yyyy`). MySQL uses `CONVERT` for character set changes.

---

### Purposes

- To perform explicit type conversion with optional style/format control (SQL Server).
- To convert character sets (MySQL `USING` clause).
- To use a syntax familiar to developers coming from SQL Server backgrounds.
- To format dates and numbers during conversion.

---

### Syntax Rules and Structure

**SQL Server**
```
CONVERT ( data_type [ ( length ) ] , expression [ , style ] )
```
**MySQL (type conversion)**
```
CONVERT ( expression , type )
```
**MySQL (character set conversion)**
```
CONVERT ( expression USING transcoding_name )
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `data_type` / `type` | Target type. |
| `expression` | Value to convert. |
| `style` (SQL Server) | Optional integer controlling format. |
| `USING` (MySQL) | Character set conversion clause. |

**Syntax Rules**

1. In SQL Server, `CONVERT` is functionally similar to `CAST` but adds the `style` argument.
2. In MySQL, `CONVERT(expr, type)` is roughly equivalent to `CAST(expr AS type)`.
3. MySQL’s `CONVERT(expr USING charset)` converts between character sets, not data types.

**Constraints and Limitations**

- `CONVERT` is not standard SQL; `CAST` is preferred for portability.
- Style codes are SQL Server-specific and not understood by other RDBMS.
- MySQL’s `CONVERT` has different semantics from SQL Server’s.

---

### Annotated Code Examples

**Example 1: SQL Server `CONVERT` with Style**

```sql
SELECT CONVERT(VARCHAR(10), GETDATE(), 103) AS formatted_date;
```

**Expected Output (illustrative)**

| formatted_date |
|----------------|
| 13/11/2026     |

**Why This Output Occurs**: Style `103` formats the date as `dd/mm/yyyy`.

---

**Example 2: MySQL `CONVERT` for Type Conversion**

```sql
SELECT CONVERT(38.8, CHAR) AS string_value;
```

**Expected Output**

| string_value |
|--------------|
| 38.8         |

**Why This Output Occurs**: MySQL’s `CONVERT` with a type argument converts the numeric value to a string.

---

**Example 3: MySQL `CONVERT` for Character Set**

```sql
SELECT CONVERT('abc' USING utf8mb4) AS utf8_string;
```

**Expected Output**

| utf8_string |
|-------------|
| abc         |

**Why This Output Occurs**: The `USING` clause converts the string to the specified character set.

---

### Real-World Cases

- **SQL Server reporting**: Formatting dates with `CONVERT(VARCHAR, date, 112)` for `yyyymmdd`.
- **MySQL internationalisation**: Converting strings between `latin1` and `utf8mb4` character sets.
- **Legacy code maintenance**: Understanding existing `CONVERT` calls when migrating from SQL Server.

---

## 3. Type Conversion (General)

### Definitions

**Core Definition**
Type conversion is the general process of changing a value’s data type, encompassing both implicit and explicit mechanisms.

**Technical Definition**
Type conversion in SQL is governed by a set of rules that determine when an implicit conversion is permitted, when an explicit conversion is required, and what the result type of a mixed-type expression will be. These rules are documented per RDBMS and involve type categories (numeric, string, temporal, etc.), preferred types within categories, and a precedence hierarchy.

**Beginner-Friendly Explanation**
Type conversion is the broad idea of changing a value’s type. It includes both the automatic conversions the database does for you and the explicit conversions you request yourself.

---

### Purposes

- To reconcile operands of different types in expressions.
- To prepare values for storage in a column of a specific type.
- To enable comparisons and joins across columns with different types.
- To support functions that expect arguments of a particular type.

---

### Syntax Rules and Structure

Type conversion has no single syntax; it is governed by:

1. **Explicit syntax**: `CAST`, `CONVERT`, `::`.
2. **Implicit rules**: Defined in the RDBMS documentation (e.g., SQL Server’s conversion matrix; PostgreSQL’s `pg_cast` catalog).
3. **Type precedence**: SQL Server’s documented precedence order.

**Constraints and Limitations**

- Conversion rules are not portable.
- Implicit conversion can produce unexpected results, especially with strings and numbers in MySQL.

---

### Real-World Cases

- **Joins on mismatched types**: Joining a `VARCHAR` column to an `INTEGER` column triggers implicit conversion and potential performance issues.
- **INSERT statements**: Assigning a string literal to a date column relies on implicit assignment conversion.

---

## 4. Implicit Conversion

### Definitions

**Core Definition**
Implicit conversion is the automatic conversion of a value from one type to another by the database engine without explicit cast syntax.

**Technical Definition**
Implicit conversions occur when an expression involves operands of different types and the RDBMS can resolve the type mismatch by applying a predefined conversion marked as “implicit” in its system catalogs. In PostgreSQL, only casts marked `AS IMPLICIT` are applied automatically; other casts require explicit syntax. In SQL Server, implicit conversions are allowed according to the documented conversion matrix and data type precedence rules. MySQL performs implicit conversions extensively, which can lead to non-intuitive results.

**Beginner-Friendly Explanation**
Implicit conversion is the database “helping” you by automatically changing one type to another when you mix them—like turning a number into text so it can be concatenated with a string. It can be convenient, but it can also cause performance problems and surprising results.

---

### Purposes

- To allow expressions with mixed types to evaluate without developer intervention.
- To simplify assignment of literals to columns (e.g., inserting `'42'` into an integer column).
- To make SQL more forgiving for beginners.

---

### Syntax Rules and Structure

Implicit conversion has no syntax; it is a set of engine rules. Key rules in SQL Server include:

- Data type precedence determines which operand is converted.
- `nvarchar` has higher precedence than `varchar`; comparing a `varchar` column to an `nvarchar` literal converts the **column**, not the literal, to `nvarchar`—which can prevent index seeks.

In PostgreSQL:

- Only casts marked `AS IMPLICIT` are applied automatically; cross-category casts (e.g., `text` → `integer`) are explicit-only.
- Assignment casts (`AS ASSIGNMENT`) allow implicit conversion during `INSERT`/`UPDATE` but not in expressions.

In MySQL:

- Strings are automatically converted to numbers in numeric context and vice versa.
- Comparing a string column to a number can prevent index usage.

**Constraints and Limitations**

- **Performance**: Implicit conversion of a column in a `WHERE` clause can suppress index usage, forcing a full scan.
- **Correctness**: Implicit conversion can produce unexpected results, especially with string-number comparisons in MySQL.
- **Portability**: Implicit conversion rules differ significantly across RDBMS.

---

### Annotated Code Examples

**Example 1: Implicit Conversion in SQL Server (Performance Pitfall)**

```sql
-- Assume dbo.Users.UserID is VARCHAR(50) and has an index
SELECT UserID
FROM dbo.Users
WHERE UserID = 12345;  -- integer literal
```

**What Happens**: SQL Server follows data type precedence: `int` has higher precedence than `varchar`, so the **column** `UserID` is implicitly converted to `int` for every row. This prevents an index seek and forces an index scan.

**Expected Performance Impact**: Full index/table scan; CPU overhead from per-row conversion.

**Correct Approach**:

```sql
SELECT UserID
FROM dbo.Users
WHERE UserID = '12345';  -- string literal matches column type
```

---

**Example 2: Implicit Conversion in MySQL**

```sql
SELECT '1' + 1 AS result;
```

**Expected Output**

| result |
|--------|
| 2      |

**Why This Output Occurs**: MySQL implicitly converts the string `'1'` to the number `1` for the addition.

---

**Example 3: PostgreSQL Assignment Cast**

```sql
CREATE TABLE t (f1 text);
INSERT INTO t (f1) VALUES (42);  -- allowed if integer→text is an assignment cast
```

**Explanation**: PostgreSQL allows implicit assignment of `integer` to `text` if the cast is marked `AS ASSIGNMENT`; otherwise the insert fails.

---

### Real-World Cases

- **ORM parameter mismatch**: Sending `NVARCHAR` parameters to a `VARCHAR` column, causing implicit conversion and index scans.
- **Legacy schema joins**: Joining `VARCHAR` IDs to `INTEGER` IDs, triggering implicit conversion and poor performance.
- **Cross-type comparisons**: `WHERE date_column = '2026-11-13'` relies on implicit string→date conversion.

---

## 5. Explicit Conversion

### Definitions

**Core Definition**
Explicit conversion is a type conversion requested by the developer using `CAST`, `CONVERT`, or a vendor-specific shorthand.

**Technical Definition**
Explicit conversions override the engine’s implicit conversion rules and force a specific target type. They are required for conversions that are not permitted implicitly (e.g., `text` → `integer` in PostgreSQL) and are recommended when the developer wants to control both the result type and the performance characteristics of a query.

**Beginner-Friendly Explanation**
Explicit conversion is when you deliberately tell the database to change a type—using `CAST` or `CONVERT`—rather than letting it decide on its own.

---

### Purposes

- To force a conversion that the engine would not perform implicitly.
- To control which side of a comparison is converted (ideally the literal/parameter, not the indexed column).
- To make the intent of the query clear and avoid surprises.
- To ensure consistent behaviour across RDBMS.

---

### Syntax Rules and Structure

Explicit conversion uses `CAST` or `CONVERT` syntax as described in sections 1 and 2.

**Best Practice**: Convert the **literal/parameter** side, not the column side, to preserve index usage:

```sql
-- Good: explicit conversion on the literal
WHERE column = CAST('12345' AS INT)

-- Bad: implicit conversion on the column (performance pitfall)
WHERE column = 12345  -- when column is VARCHAR
```

**Constraints and Limitations**

- Explicit conversion may still be lossy (e.g., `CAST(1.9 AS INT)` truncates or rounds depending on RDBMS).
- Not all conversions are permitted even explicitly.

---

### Annotated Code Examples

**Example 1: Explicit Conversion to Preserve Index Usage (SQL Server)**

```sql
-- Assume UserID is VARCHAR(50) with an index
SELECT UserID
FROM dbo.Users
WHERE UserID = CAST(12345 AS VARCHAR(50));
```

**Explanation**: By explicitly converting the integer literal to `VARCHAR`, the column remains unchanged and the index can be used for a seek.

---

**Example 2: Explicit Conversion for Division Precision**

```sql
SELECT CAST(7 AS NUMERIC) / 2 AS result;  -- 3.5
```

**Explanation**: Explicitly casting to `NUMERIC` ensures numeric division rather than integer division.

---

### Real-World Cases

- **Index-friendly queries**: Casting parameters to match column types in application queries.
- **Data cleansing**: Explicitly casting strings to dates or numbers with error handling.
- **Reporting**: Controlling output types for downstream systems.

---

## 6. Conversion Pitfalls

### Definitions

**Core Definition**
Conversion pitfalls are the unintended consequences—performance degradation, incorrect results, or data loss—that arise from poorly controlled or misunderstood type conversions.

**Technical Definition**
Conversion pitfalls encompass both semantic issues (e.g., implicit string→number conversion producing unexpected matches) and performance issues (e.g., implicit conversion of an indexed column preventing index seeks and causing full scans). They are documented in vendor performance guidance and are a common source of production incidents.

**Beginner-Friendly Explanation**
Conversion pitfalls are the “gotchas” of type conversion—times when the database’s automatic type-changing behaviour causes your query to run slowly or return the wrong answer.

---

### Purposes (of Understanding Pitfalls)

- To avoid query performance degradation from index suppression.
- To prevent silent data truncation or rounding.
- To ensure correct comparison semantics across types.
- To write portable SQL that behaves consistently across RDBMS.

---

### Syntax Rules and Structure (Diagnostic Approaches)

There is no syntax for pitfalls per se, but diagnosis involves:

1. **Execution plans**: Looking for `CONVERT_IMPLICIT` warnings in SQL Server plans.
2. **`EXPLAIN` / `EXPLAIN ANALYZE`**: Detecting per-row conversion in PostgreSQL.
3. **Schema review**: Ensuring that literal/parameter types match column types.

**Constraints and Limitations**

- Pitfalls are often invisible at development time because they depend on data types and index design.
- ORMs may generate implicit conversions without the developer’s knowledge.

---

### Annotated Code Examples

**Pitfall 1: Implicit Conversion of Indexed Column (SQL Server)**

```sql
-- Indexed column: VARCHAR(50)
-- Query uses integer literal → column converted per row → index scan
SELECT *
FROM dbo.Users
WHERE UserID = 12345;  -- BAD
```

**Performance Diagnosis**: Execution plan shows `CONVERT_IMPLICIT(int, [UserID])` and an Index Scan instead of an Index Seek.

**Fix**:

```sql
SELECT *
FROM dbo.Users
WHERE UserID = '12345';  -- GOOD: literal matches column type
```

---

**Pitfall 2: String-to-Number Comparison in MySQL**

```sql
-- str_col is VARCHAR and indexed
SELECT * FROM tbl_name WHERE str_col = 1;
```

**Problem**: MySQL implicitly converts the string column to a number for the comparison, preventing index usage.

**Fix**:

```sql
SELECT * FROM tbl_name WHERE str_col = '1';
```

---

**Pitfall 3: Lossy Conversion (Truncation)**

```sql
SELECT CAST(123.456 AS INTEGER);  -- may truncate or round
```

**Explanation**: The fractional part is lost; depending on RDBMS, the result may be `123` (truncation) or `123`/`124` (rounding). Always verify semantics.

---

**Pitfall 4: `NVARCHAR` vs. `VARCHAR` Mismatch (SQL Server)**

```sql
-- Column: VARCHAR(100); parameter sent as NVARCHAR
WHERE varchar_column = N'some_value';
```

**Problem**: `NVARCHAR` has higher precedence, so the `varchar` column is converted to `nvarchar` per row, preventing index seek.

**Fix**: Send the parameter as `VARCHAR` or explicitly convert the parameter, not the column.

---

### Real-World Cases

- **Production incident**: A query with an implicit conversion runs for minutes instead of milliseconds because an index scan replaces an index seek.
- **Silent data corruption**: A lossy cast truncates monetary values without warning.
- **ORM-generated queries**: Framework sends parameters with types that mismatch the schema, causing widespread implicit conversions.

---

## References

1. **PostgreSQL Documentation — Type Conversion** — https://www.postgresql.org/docs/current/typeconv.html
2. **PostgreSQL Documentation — CREATE CAST** — https://www.postgresql.org/docs/current/sql-createcast.html
3. **PostgreSQL Documentation — Type Casts (SQL Syntax)** — https://www.postgresql.org/docs/current/sql-syntax.html
4. **MySQL 8.0 Reference Manual — Type Conversion in Expression Evaluation** — https://dev.mysql.com/doc/refman/8.0/en/type-conversion.html
5. **MySQL 8.0 Reference Manual — Cast Functions and Operators** — https://dev.mysql.com/doc/refman/8.0/en/cast-functions.html
6. **Microsoft SQL Server — CAST and CONVERT (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/functions/cast-and-convert-transact-sql
7. **Microsoft SQL Server — Data Type Precedence (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/data-types/data-type-precedence-transact-sql
8. **Microsoft Learn — Choose Appropriate Data Types (Performance Impact of Conversions)** — https://learn.microsoft.com/en-us/training/modules/choose-data-types/
9. **SQL:2016 Standard — ISO/IEC 9075-2:2016 (Foundation)** — https://www.iso.org/standard/63555.html

---

This cheat sheet provides a comprehensive reference for SQL casting and conversion across PostgreSQL, MySQL, and SQL Server, with emphasis on the performance implications of implicit conversion and best practices for explicit, index-friendly queries.