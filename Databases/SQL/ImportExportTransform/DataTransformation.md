# SQL Data Transformation: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL data transformation is the process of converting, cleansing, standardizing, and validating data within a relational database system to improve its quality, consistency, and usability for downstream applications and analytics.

**Technical Definition:** SQL data transformation encompasses a set of declarative operations—including type casting, string manipulation, null handling, deduplication via window functions, format standardization, and constraint-based validation—that reshape raw data into a canonical, query-optimized form while preserving referential integrity and domain semantics. These operations are typically executed within ETL/ELT pipelines, during data migration, or as part of ongoing data quality management.

**Beginner-Friendly Explanation:** SQL data transformation is like cleaning and organizing a messy room. You sort items into the right places (type conversion), label them consistently (standardization), throw away duplicates (deduplication), fix broken items (data cleansing), and put up rules to keep things tidy (validation). The goal is to make the data reliable and easy to use.

### Key Characteristics

- **Declarative:** Transformations are expressed as SQL expressions and statements, not procedural loops.
- **Set-based:** Operations apply to entire columns or result sets, not individual rows in application code.
- **Idempotent where possible:** Many transformations (trimming, case unification, null replacement) produce the same result regardless of how many times they are applied.
- **Composable:** Transformations can be chained in `SELECT`, `UPDATE`, or `INSERT ... SELECT` statements.
- **Performance-sensitive:** Transformations on large tables may benefit from indexes, batch processing, and avoiding functions on indexed columns in `WHERE` clauses.
- **Vendor-extensible:** Each RDBMS provides proprietary functions alongside SQL-standard constructs.

### Prerequisites

- Proficiency with SQL `SELECT`, `UPDATE`, and `INSERT` statements.
- Understanding of data types (numeric, string, date/time, boolean, JSON).
- Familiarity with `NULL` semantics and three-valued logic.
- Knowledge of regular expressions (for string parsing).
- Awareness of transaction isolation and locking behavior during `UPDATE` operations.

### Related Programming Areas

- ETL (Extract, Transform, Load) and ELT pipelines.
- Data quality and master data management.
- Database migration and integration.
- Analytics engineering and dbt models.
- Application-layer data validation.

### Core Concepts / Features

1. **Type Conversion** (explicit casting, parsing exceptions, time zone management)
2. **String Transformation** (regex parsing, trimming, padding, case unification)
3. **Data Cleansing** (`COALESCE`/`NULLIF`, malformed rows, orphan references)
4. **Deduplication** (`ROW_NUMBER() OVER (PARTITION BY ...)`)
5. **Standardization** (dates, phone numbers, addresses, localized units)
6. **Data Validation** (runtime constraints, domain rules, error-rejection routing)


## Core Concept 1: Type Conversion

### Definitions

**Core Definition:** Type conversion is the process of changing a value from one data type to another, either explicitly via casting functions or implicitly by the database engine.

**Technical Definition:** Explicit type conversion uses functions such as `CAST(expr AS type)`, `CONVERT(type, expr)`, or `TRY_CAST(expr AS type)` to coerce a value into a target type. Implicit conversion occurs automatically when the database engine determines that a type change is required for an operation (e.g., comparing a string to a number). Time zone conversion uses the `AT TIME ZONE` operator (PostgreSQL, SQL Server) or `CONVERT_TIMEZONE` (Amazon Redshift) to shift timestamp values between zones.

**Beginner-Friendly Explanation:** Type conversion is like translating between languages. If a value is stored as text but you need to do math with it, you convert it to a number. If you have a date in one time zone and need it in another, you convert the time zone. It's about making sure values are in the right "format" for the operation you want to perform.

### Purposes

- To prepare data for arithmetic, comparison, or aggregation when the source type differs from the target type.
- To safely handle parsing failures using functions like `TRY_CAST` (SQL Server), `TRY_CONVERT` (SQL Server), or `CAST ... AS ...` with error handling.
- To normalize timestamps across time zones for consistent reporting and analytics.
- To convert between character sets (e.g., Latin-1 to UTF-8) during data integration.

### Syntax Rules and Structure

#### Complete General Syntax (SQL Standard `CAST`)

```sql
CAST(expression AS target_type)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `expression` | The value or column to convert. |
| `AS` | Keyword introducing the target type. |
| `target_type` | The desired data type (e.g., `INTEGER`, `NUMERIC(10,2)`, `DATE`, `TIMESTAMP WITH TIME ZONE`). |

#### Complete General Syntax (PostgreSQL `AT TIME ZONE`)

```sql
timestamp_expression AT TIME ZONE zone_name
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `timestamp_expression` | A `timestamp` or `timestamptz` value. |
| `AT TIME ZONE` | Operator that shifts the timestamp to the named zone. |
| `zone_name` | A time zone name (e.g., `'America/Denver'`, `'UTC'`). |

#### Complete General Syntax (SQL Server `TRY_CAST`)

```sql
TRY_CAST(expression AS target_type)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `expression` | The value to convert. |
| `AS target_type` | The desired target type. |

#### Syntax Rules

- `CAST` follows the SQL standard and is supported by PostgreSQL, MySQL, SQL Server, and Oracle.
- `CONVERT(type, expression)` is a MySQL/SQL Server alternative; in MySQL, `CONVERT(expr USING charset)` performs character set conversion.
- `AT TIME ZONE` in PostgreSQL converts `timestamp without time zone` to `timestamp with time zone` (assuming the input is in the named zone) and `timestamp with time zone` to `timestamp without time zone` (showing the time as it would appear in that zone).
- SQL Server's `AT TIME ZONE` syntax uses two applications to convert between zones: `original_timestamp AT TIME ZONE 'UTC' AT TIME ZONE 'Pacific Standard Time'`.
- `TRY_CAST` and `TRY_CONVERT` return `NULL` on failure instead of raising an error, but still raise errors for explicitly disallowed conversions (e.g., `int` to `date`).

#### Constraints and Limitations

- **Explicitly disallowed conversions:** Some conversions are never permitted (e.g., `INT` to `DATE` in SQL Server) and will raise an error even with `TRY_CAST`.
- **Precision loss:** Converting `NUMERIC` to `INTEGER` truncates; converting `TIMESTAMP` to `DATE` discards time.
- **Time zone ambiguity:** `AT TIME ZONE` on a `timestamp without time zone` assumes the value is in the named zone; this may produce unexpected results if the source data is already in UTC.
- **Implicit conversion performance:** Implicit conversions can prevent index usage and cause full table scans; prefer explicit conversion on the literal side of a comparison.
- **Version-specific:** `TRY_CAST` was introduced in SQL Server 2012; PostgreSQL does not have a built-in `TRY_CAST` but supports `CAST ... AS ...` with `CASE` or `NULLIF` for error handling.

### Annotated Code Examples

#### Example 1: PostgreSQL Explicit Cast with Error Handling

```sql
-- Create a table with a text column that should contain numbers
CREATE TABLE raw_measurements (
    id SERIAL PRIMARY KEY,
    reading_text TEXT
);

INSERT INTO raw_measurements (reading_text) VALUES
('42.5'), ('98.6'), ('not_a_number'), ('0'), ('');

-- Safe conversion: use a CASE expression to handle non-numeric values
SELECT
    id,
    reading_text,
    CASE
        WHEN reading_text ~ '^-?[0-9]+(\.[0-9]+)?$'
        THEN reading_text::NUMERIC
        ELSE NULL
    END AS reading_numeric
FROM raw_measurements;
```

**Expected Output:**

```
 id | reading_text  | reading_numeric
----+---------------+-----------------
  1 | 42.5          |           42.5
  2 | 98.6          |           98.6
  3 | not_a_number  |          (null)
  4 | 0             |              0
  5 |               |          (null)
```

**Why This Works:** The regular expression `^-?[0-9]+(\.[0-9]+)?$` validates that the text is a valid number before casting. Rows that do not match return `NULL`. This approach avoids the `invalid input syntax for type numeric` error that would occur with a direct `reading_text::NUMERIC` cast.

#### Example 2: PostgreSQL `AT TIME ZONE` Conversion

```sql
-- Create a table with timestamps in UTC
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    event_name TEXT,
    created_at TIMESTAMPTZ
);

INSERT INTO events (event_name, created_at) VALUES
('login', '2024-06-15 14:30:00+00'),
('purchase', '2024-06-15 22:15:00+00');

-- Convert UTC timestamps to America/New_York local time
SELECT
    event_name,
    created_at AS utc_time,
    created_at AT TIME ZONE 'America/New_York' AS local_time
FROM events;
```

**Expected Output:**

```
 event_name |      utc_time       |     local_time
------------+---------------------+---------------------
 login      | 2024-06-15 14:30:00+00 | 2024-06-15 10:30:00
 purchase   | 2024-06-15 22:15:00+00 | 2024-06-15 18:15:00
```

**Why This Works:** `created_at` is `TIMESTAMPTZ`, so PostgreSQL knows the absolute instant. `AT TIME ZONE 'America/New_York'` converts that instant to the local wall-clock time in New York. The result is `TIMESTAMP WITHOUT TIME ZONE` because the zone information is now represented by the displayed local time.

#### Example 3: SQL Server `TRY_CAST` with Fallback

```sql
-- Create a table with mixed date formats stored as strings
CREATE TABLE raw_dates (
    id INT IDENTITY(1,1),
    date_string NVARCHAR(20)
);

INSERT INTO raw_dates (date_string) VALUES
('2024-01-15'), ('01/20/2024'), ('invalid'), ('2024-03-10');

-- Attempt conversion; fallback to NULL on failure
SELECT
    id,
    date_string,
    TRY_CAST(date_string AS DATE) AS parsed_date,
    COALESCE(
        TRY_CAST(date_string AS DATE),
        TRY_CONVERT(DATE, date_string, 101)  -- 101 = MM/DD/YYYY
    ) AS best_effort_date
FROM raw_dates;
```

**Expected Output:**

```
id | date_string | parsed_date | best_effort_date
---+-------------+-------------+------------------
1  | 2024-01-15  | 2024-01-15  | 2024-01-15
2  | 01/20/2024  | NULL        | 2024-01-20
3  | invalid     | NULL        | NULL
4  | 2024-03-10  | 2024-03-10  | 2024-03-10
```

**Why This Works:** `TRY_CAST(date_string AS DATE)` handles ISO format (`YYYY-MM-DD`) but returns `NULL` for `MM/DD/YYYY` strings. The `COALESCE` falls back to `TRY_CONVERT` with style `101` (US date format). This layered approach maximizes successful parsing while safely returning `NULL` for unparseable values.

### Real-World Cases

- **Data warehouse loading:** Converting string-typed columns from source systems into proper `DATE`, `NUMERIC`, or `BOOLEAN` types before loading.
- **Log analysis:** Parsing timestamps from log files (often strings) into `TIMESTAMPTZ` for time-series analysis.
- **Financial reconciliation:** Converting currency amounts stored as text with currency symbols into `NUMERIC` for aggregation.
- **Multi-region reporting:** Converting all timestamps to a canonical time zone (UTC) for consistent cross-region reporting.

### References

- PostgreSQL Documentation: Type Conversion — https://www.postgresql.org/docs/current/typeconv.html
- PostgreSQL Documentation: AT TIME ZONE — https://www.postgresql.org/docs/current/functions-datetime.html#FUNCTIONS-DATETIME-ZONECONVERT
- MySQL Reference Manual: Cast Functions and Operators — https://dev.mysql.com/doc/refman/8.0/en/cast-functions.html
- SQL Server: CAST and CONVERT (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/functions/cast-and-convert-transact-sql
- SQL Server: TRY_CAST (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/functions/try-cast-transact-sql
- Oracle Database SQL Language Reference: Conversion Functions — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/Conversion-Functions.html


## Core Concept 2: String Transformation

### Definitions

**Core Definition:** String transformation is the process of manipulating text values—using regular expressions, trimming, padding, and case conversion—to produce clean, consistently formatted strings.

**Technical Definition:** String transformation applies functions such as `regexp_replace`, `trim`, `lpad`, `rpad`, `upper`, `lower`, and `initcap` to character data, enabling pattern-based extraction, substitution, whitespace normalization, fixed-width formatting, and case unification. These operations are fundamental to data cleansing pipelines where source data originates from heterogeneous systems with inconsistent formatting conventions.

**Beginner-Friendly Explanation:** String transformation is like editing a messy document. You remove extra spaces, fix capitalization, replace typos, and format things consistently. In SQL, you use functions like `TRIM` to remove spaces, `UPPER` to make everything uppercase, and `REGEXP_REPLACE` to find and fix patterns.

### Purposes

- To remove leading, trailing, or repeated whitespace from user-entered data.
- To extract structured information (phone numbers, dates, codes) from unstructured text using regular expressions.
- To enforce consistent case (all uppercase, all lowercase, or title case) for joins, comparisons, and display.
- To pad values to a fixed width for fixed-format exports or legacy system integration.
- To replace or remove unwanted characters (punctuation, control characters, non-printable symbols).

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL `regexp_replace`)

```sql
regexp_replace(source text, pattern text, replacement text [, flags text])
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `source` | The input string. |
| `pattern` | A POSIX regular expression to match. |
| `replacement` | The replacement text; `\1`, `\2`, etc. reference capture groups. |
| `flags` | Optional flags: `'g'` for global replace, `'i'` for case-insensitive. |

#### Complete General Syntax (PostgreSQL `trim`)

```sql
trim([LEADING | TRAILING | BOTH] [characters text] FROM string text)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `LEADING` | Remove from the start only. |
| `TRAILING` | Remove from the end only. |
| `BOTH` | Remove from both ends (default). |
| `characters` | The set of characters to remove (default: space). |
| `string` | The input string. |

#### Complete General Syntax (Case Conversion)

```sql
UPPER(string)   -- Convert to uppercase
LOWER(string)   -- Convert to lowercase
INITCAP(string) -- Capitalize first letter of each word
```

#### Syntax Rules

- `regexp_replace` uses POSIX regular expression syntax; the `'g'` flag enables global replacement (all matches), otherwise only the first match is replaced.
- `trim` can remove any set of characters, not just spaces: `trim(BOTH 'xyz' FROM 'yxTomxx')` returns `'Tom'`.
- `initcap` capitalizes the first letter of each word and lowercases the rest; words are sequences of alphanumeric characters separated by non-alphanumeric characters.
- MySQL uses `REGEXP_REPLACE` (MySQL 8.0+), `TRIM`, `LTRIM`, `RTRIM`, `UPPER`, `LOWER`; MySQL's `TRIM` does not support the `LEADING`/`TRAILING`/`BOTH` syntax with a character set in the same way as PostgreSQL.
- SQL Server uses `REPLACE`, `TRIM` (2017+), `LTRIM`, `RTRIM`, `UPPER`, `LOWER`; for regex, use `dbo.RegexReplace` via CLR or a third-party library.

#### Constraints and Limitations

- **Performance:** Regular expressions are computationally expensive; avoid applying `regexp_replace` to indexed columns in `WHERE` clauses.
- **Character encoding:** Regex behavior may differ between UTF-8 and single-byte encodings; use `utf8mb4` in MySQL for full Unicode support.
- **MySQL `REGEXP_REPLACE`:** Introduced in MySQL 8.0; not available in earlier versions.
- **SQL Server:** No built-in regex support; `PATINDEX` and `LIKE` provide limited pattern matching.
- **Version-specific:** PostgreSQL 15+ supports `regexp_replace` with `start` parameter to begin replacement from a specific position.

### Annotated Code Examples

#### Example 1: PostgreSQL String Cleansing Pipeline

```sql
-- Create a table with messy user data
CREATE TABLE raw_users (
    id SERIAL PRIMARY KEY,
    full_name TEXT,
    email TEXT
);

INSERT INTO raw_users (full_name, email) VALUES
('  alice smith  ', 'ALICE@EXAMPLE.COM'),
('BOB   JONES', 'bob@example.com '),
('  carol  ', 'carol@EXAMPLE.com');

-- Apply multiple string transformations in one query
SELECT
    id,
    -- Trim whitespace, collapse internal spaces, title-case
    initcap(regexp_replace(trim(full_name), '\s+', ' ', 'g')) AS clean_name,
    -- Trim and lowercase email
    lower(trim(email)) AS clean_email
FROM raw_users;
```

**Expected Output:**

```
 id |  clean_name  |    clean_email
----+--------------+-------------------
  1 | Alice Smith  | alice@example.com
  2 | Bob Jones    | bob@example.com
  3 | Carol        | carol@example.com
```

**Why This Works:** `trim()` removes leading and trailing spaces. `regexp_replace(..., '\s+', ' ', 'g')` collapses multiple internal spaces into a single space. `initcap()` converts to title case. `lower(trim(email))` trims and lowercases the email. The combination produces clean, standardized output in a single pass.

#### Example 2: MySQL Phone Number Standardization

```sql
-- Create a table with phone numbers in various formats
CREATE TABLE raw_phones (
    id INT AUTO_INCREMENT PRIMARY KEY,
    phone_raw VARCHAR(30)
);

INSERT INTO raw_phones (phone_raw) VALUES
('(555) 123-4567'),
('555.123.4567'),
('5551234567'),
('+1 555 123 4567');

-- Strip all non-digit characters
SELECT
    id,
    phone_raw,
    REGEXP_REPLACE(phone_raw, '[^0-9]', '') AS phone_digits
FROM raw_phones;
```

**Expected Output:**

```
id | phone_raw         | phone_digits
---+-------------------+--------------
1  | (555) 123-4567    | 5551234567
2  | 555.123.4567      | 5551234567
3  | 5551234567        | 5551234567
4  | +1 555 123 4567   | 15551234567
```

**Why This Works:** `REGEXP_REPLACE(phone_raw, '[^0-9]', '')` removes all characters that are not digits. This is the first step in standardization; a second step might add a country code or format the digits as `(XXX) XXX-XXXX` using `INSERT` or `CONCAT`.

#### Example 3: SQL Server String Padding and Trimming

```sql
-- Create a table with codes that need padding
CREATE TABLE product_codes (
    id INT IDENTITY(1,1),
    code NVARCHAR(20)
);

INSERT INTO product_codes (code) VALUES
('ABC'), ('12'), ('XYZ123'), ('  A  ');

-- Pad left with zeros to 10 characters, then trim
SELECT
    id,
    code,
    RIGHT('0000000000' + TRIM(code), 10) AS padded_code,
    LEN(TRIM(code)) AS trimmed_length
FROM product_codes;
```

**Expected Output:**

```
id | code    | padded_code | trimmed_length
---+---------+-------------+----------------
1  | ABC     | 0000000ABC  | 3
2  | 12      | 0000000012  | 2
3  | XYZ123  | 0000XYZ123  | 6
4  |   A     | 000000000A  | 1
```

**Why This Works:** SQL Server does not have a native `LPAD` function in older versions. The idiom `RIGHT('0000000000' + value, length)` pads the value to the specified length with zeros. `TRIM` (SQL Server 2017+) removes spaces before padding. `LEN(TRIM(code))` shows the effective length after trimming.

### Real-World Cases

- **CRM data cleansing:** Standardizing customer names (title case, whitespace collapse) and email addresses (lowercase, trim).
- **Log parsing:** Extracting IP addresses, timestamps, or error codes from unstructured log lines using regex.
- **Product catalog normalization:** Padding SKUs to a fixed width for warehouse systems; removing special characters from product names.
- **Internationalization:** Converting text to a consistent case for case-insensitive matching in multilingual datasets.

### References

- PostgreSQL Documentation: String Functions and Operators — https://www.postgresql.org/docs/current/functions-string.html
- PostgreSQL Documentation: Pattern Matching — https://www.postgresql.org/docs/current/functions-matching.html
- MySQL Reference Manual: String Functions and Operators — https://dev.mysql.com/doc/refman/8.0/en/string-functions.html
- MySQL Reference Manual: Regular Expressions — https://dev.mysql.com/doc/refman/8.0/en/regexp.html
- SQL Server: String Functions (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/functions/string-functions-transact-sql
- Oracle Database SQL Language Reference: String Functions — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/String-Functions.html


## Core Concept 3: Data Cleansing

### Definitions

**Core Definition:** Data cleansing is the process of detecting and correcting (or removing) corrupt, inaccurate, incomplete, or irrelevant records from a dataset, including handling `NULL` values, fixing malformed rows, and resolving orphaned foreign key references.

**Technical Definition:** Data cleansing uses SQL functions such as `COALESCE` (return the first non-NULL argument), `NULLIF` (return NULL if two values are equal), and `CASE` expressions to impute missing values, normalize sentinel values, and enforce referential integrity. Orphan references—foreign key values that do not match any parent key—are identified via `LEFT JOIN ... WHERE parent.id IS NULL` or `NOT EXISTS` subqueries.

**Beginner-Friendly Explanation:** Data cleansing is like fixing a messy spreadsheet. Some cells are empty (NULL), some have placeholder text like "N/A" that should be treated as empty, and some rows refer to things that no longer exist (orphans). Data cleansing fixes these problems so your data is reliable.

### Purposes

- To replace `NULL` values with meaningful defaults or values computed from other columns.
- To convert sentinel values (e.g., `'N/A'`, `0`, `-1`) into proper `NULL` values for consistent handling.
- To identify and remove orphaned rows that violate foreign key relationships.
- To repair malformed rows that cannot be parsed or violate domain constraints.

### Syntax Rules and Structure

#### Complete General Syntax (`COALESCE`)

```sql
COALESCE(value1, value2, ..., valueN)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `value1 ... valueN` | A list of expressions evaluated in order; the first non-NULL value is returned. |

**Rules:** `COALESCE` only evaluates arguments as needed; once a non-NULL value is found, remaining arguments are not evaluated. All arguments must be of compatible types.

#### Complete General Syntax (`NULLIF`)

```sql
NULLIF(value1, value2)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `value1` | The value to return if not equal to `value2`. |
| `value2` | The value to compare against; if equal to `value1`, returns `NULL`. |

**Rules:** `NULLIF(v1, v2)` is equivalent to `CASE WHEN v1 = v2 THEN NULL ELSE v1 END`. It is often used to convert sentinel values to `NULL`: `NULLIF(age, 0)` returns `NULL` if age is 0.

#### Complete General Syntax (Orphan Detection)

```sql
SELECT child.*
FROM child_table child
LEFT JOIN parent_table parent ON child.parent_id = parent.id
WHERE parent.id IS NULL;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `LEFT JOIN` | Includes all child rows, with `NULL` parent columns for non-matching rows. |
| `WHERE parent.id IS NULL` | Filters to rows where no parent match was found—these are orphans. |

#### Syntax Rules

- `COALESCE` is SQL-standard and supported by all major RDBMS. Some databases also provide `ISNULL` (SQL Server), `IFNULL` (MySQL, Oracle), and `NVL` (Oracle) as proprietary alternatives.
- `NULLIF` is SQL-standard and widely supported.
- Orphan detection requires comparing foreign key values against parent keys; use `NOT EXISTS` for better performance on large tables when the parent table is indexed.
- When repairing orphans, the action may be to delete the child row, reassign the foreign key to a default parent, or insert a placeholder parent.

#### Constraints and Limitations

- **`COALESCE` type compatibility:** All arguments must be coercible to a common type; mixing incompatible types raises an error.
- **`NULLIF` with NULL:** If either argument is `NULL`, `NULLIF` returns the first argument (since `NULL = NULL` is unknown, not true).
- **Orphan repair destructive:** Deleting orphan rows is irreversible; always back up or stage before deletion.
- **Performance:** `LEFT JOIN ... WHERE IS NULL` can be slow on very large tables; index the foreign key column and consider `NOT EXISTS` with a semi-join.
- **Version-specific:** SQL Server's `ISNULL` is limited to two arguments; `COALESCE` is preferred for portability.

### Annotated Code Examples

#### Example 1: PostgreSQL `COALESCE` and `NULLIF` for Missing Data

```sql
-- Create a table with missing and sentinel values
CREATE TABLE customer_data (
    id SERIAL PRIMARY KEY,
    name TEXT,
    age INT,
    city TEXT
);

INSERT INTO customer_data (name, age, city) VALUES
('Alice', 30, 'New York'),
('Bob', 0, NULL),        -- age 0 is a sentinel for "unknown"
('Carol', NULL, 'N/A'),  -- city 'N/A' is a sentinel
('Dave', 25, '');

-- Cleanse: replace sentinel values with NULL, then provide defaults
SELECT
    id,
    name,
    -- Treat 0 as NULL, then default to -1 if still NULL
    COALESCE(NULLIF(age, 0), -1) AS clean_age,
    -- Treat 'N/A' and '' as NULL, then default to 'Unknown'
    COALESCE(NULLIF(NULLIF(city, 'N/A'), ''), 'Unknown') AS clean_city
FROM customer_data;
```

**Expected Output:**

```
 id | name  | clean_age | clean_city
----+-------+-----------+------------
  1 | Alice |        30 | New York
  2 | Bob   |        -1 | Unknown
  3 | Carol |        -1 | Unknown
  4 | Dave  |        25 | Unknown
```

**Why This Works:** `NULLIF(age, 0)` converts the sentinel `0` to `NULL`. `COALESCE(..., -1)` then replaces `NULL` with `-1`. For city, nested `NULLIF` handles both `'N/A'` and `''` as sentinels, and `COALESCE` defaults to `'Unknown'`. This layered approach handles multiple sentinel conventions in one pass.

#### Example 2: MySQL Orphan Detection and Cleanup

```sql
-- Parent table
CREATE TABLE departments (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

-- Child table with potential orphans
CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50),
    dept_id INT
);

INSERT INTO departments (id, name) VALUES (1, 'Engineering'), (2, 'Sales');
INSERT INTO employees (name, dept_id) VALUES
('Alice', 1), ('Bob', 2), ('Carol', 99), ('Dave', NULL);

-- Detect orphans
SELECT e.id, e.name, e.dept_id
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id
WHERE e.dept_id IS NOT NULL AND d.id IS NULL;
```

**Expected Output:**

```
id | name  | dept_id
---+-------+---------
3  | Carol | 99
```

**Why This Works:** The `LEFT JOIN` includes all employees. `WHERE e.dept_id IS NOT NULL` excludes employees with no department (which are not orphans, just unassigned). `AND d.id IS NULL` filters to employees whose `dept_id` does not match any department. Carol's `dept_id` of 99 has no matching department, making her an orphan.

#### Example 3: SQL Server `COALESCE` for Imputation

```sql
-- Create a table with missing values
CREATE TABLE sensor_readings (
    id INT IDENTITY(1,1),
    sensor_id INT,
    reading DECIMAL(10,2)
);

INSERT INTO sensor_readings (sensor_id, reading) VALUES
(1, 22.5), (1, NULL), (1, 23.1), (2, NULL), (2, 19.8);

-- Impute missing readings with the average for that sensor
SELECT
    sr.id,
    sr.sensor_id,
    sr.reading,
    COALESCE(sr.reading, avg_readings.avg_reading) AS imputed_reading
FROM sensor_readings sr
LEFT JOIN (
    SELECT sensor_id, AVG(reading) AS avg_reading
    FROM sensor_readings
    WHERE reading IS NOT NULL
    GROUP BY sensor_id
) avg_readings ON sr.sensor_id = avg_readings.sensor_id;
```

**Expected Output:**

```
id | sensor_id | reading | imputed_reading
---+-----------+---------+-----------------
1  | 1         | 22.50   | 22.50
2  | 1         | NULL    | 22.80
3  | 1         | 23.10   | 23.10
4  | 2         | NULL    | 19.80
5  | 2         | 19.80   | 19.80
```

**Why This Works:** The subquery computes the average reading per sensor (excluding NULLs). The `LEFT JOIN` brings that average alongside each reading. `COALESCE(sr.reading, avg_readings.avg_reading)` replaces `NULL` readings with the sensor's average. This is a common imputation strategy for time-series data.

### Real-World Cases

- **Customer database deduplication:** Replacing missing email addresses with a default or a value derived from another column.
- **IoT sensor data:** Imputing missing readings with the last known value or the sensor's historical average.
- **Financial records:** Converting placeholder values (`-1`, `'N/A'`) into `NULL` so aggregations exclude them correctly.
- **Referential integrity repair:** Identifying and deleting orphaned child records after a parent table purge.

### References

- PostgreSQL Documentation: Conditional Expressions (`COALESCE`, `NULLIF`) — https://www.postgresql.org/docs/current/functions-conditional.html
- MySQL Reference Manual: Comparison Functions and Operators — https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html
- SQL Server: COALESCE (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/language-elements/coalesce-transact-sql
- SQL Server: NULLIF (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/language-elements/nullif-transact-sql
- Oracle Database SQL Language Reference: NULL-Related Functions — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/NULL-Related-Functions.html


## Core Concept 4: Deduplication

### Definitions

**Core Definition:** Deduplication is the process of identifying and removing duplicate rows from a dataset, typically retaining one canonical record per logical entity.

**Technical Definition:** SQL deduplication uses window functions—primarily `ROW_NUMBER() OVER (PARTITION BY dedup_key ORDER BY preference)`—to assign a sequential rank to each row within a partition of duplicate keys. Rows with `ROW_NUMBER() > 1` are duplicates and can be deleted or filtered out. The `PARTITION BY` clause defines the columns that constitute a duplicate (the deduplication key), and the `ORDER BY` clause defines which row is retained (e.g., the most recent by timestamp).

**Beginner-Friendly Explanation:** Deduplication is like finding and removing duplicate entries in a phone book. You decide what makes two entries "the same" (e.g., same name and address), then you keep the most recent one and delete the rest. SQL's `ROW_NUMBER()` helps you number the duplicates so you can keep number 1 and delete the others.

### Purposes

- To remove exact duplicate rows that result from data loading errors or repeated imports.
- To retain the most recent or most complete version of a record when multiple versions exist.
- To enforce uniqueness before creating a unique index or primary key constraint.
- To reduce storage and improve query performance by eliminating redundant data.

### Syntax Rules and Structure

#### Complete General Syntax (Deduplication with `ROW_NUMBER`)

```sql
WITH ranked AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY dedup_column1, dedup_column2, ...
            ORDER BY preference_column DESC
        ) AS rn
    FROM source_table
)
SELECT * FROM ranked WHERE rn = 1;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `WITH ranked AS (...)` | Common Table Expression (CTE) that adds a rank column. |
| `ROW_NUMBER()` | Assigns a unique sequential integer to each row within the partition. |
| `PARTITION BY` | Columns that define a duplicate group; all rows with the same values are duplicates. |
| `ORDER BY` | Determines which row gets `rn = 1` (the row to keep). |
| `rn = 1` | Filters to the canonical row; `rn > 1` are duplicates. |

#### Complete General Syntax (Delete Duplicates)

```sql
WITH ranked AS (
    SELECT
        id,
        ROW_NUMBER() OVER (PARTITION BY dedup_key ORDER BY preference DESC) AS rn
    FROM target_table
)
DELETE FROM target_table
WHERE id IN (SELECT id FROM ranked WHERE rn > 1);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ranked` CTE | Identifies duplicate rows by `id`. |
| `DELETE FROM ... WHERE id IN (...)` | Deletes all rows with `rn > 1`. |

#### Syntax Rules

- `ROW_NUMBER()` always returns a unique number, even for tied rows; `RANK()` and `DENSE_RANK()` assign the same rank to ties.
- The `ORDER BY` clause in the window is mandatory for `ROW_NUMBER()`; it determines which row is numbered first.
- To keep the "first" row based on insertion order, use `ORDER BY (SELECT NULL)` or an identity column.
- The CTE approach is standard and works in PostgreSQL, SQL Server, MySQL 8.0+, and Oracle.
- For MySQL versions before 8.0, use a self-join with `GROUP BY` or a temporary table with `AUTO_INCREMENT`.

#### Constraints and Limitations

- **Performance:** Window functions require sorting the entire result set; index the `PARTITION BY` columns if possible.
- **Locking:** `DELETE` on large tables locks rows; batch the deletion in chunks to avoid long transactions.
- **MySQL before 8.0:** Does not support window functions; use `GROUP BY` with `MIN(id)` or a temporary table.
- **Data loss risk:** Always back up or stage before deleting duplicates; verify the `PARTITION BY` columns accurately define duplicates.
- **Version-specific:** PostgreSQL 9.4+ supports `ROW_NUMBER()`; SQL Server 2005+; Oracle 8i+; MySQL 8.0+.

### Annotated Code Examples

#### Example 1: PostgreSQL Deduplication Keeping the Most Recent Row

```sql
-- Create a table with duplicate customer records
CREATE TABLE customers_raw (
    id SERIAL PRIMARY KEY,
    email TEXT,
    name TEXT,
    updated_at TIMESTAMP
);

INSERT INTO customers_raw (email, name, updated_at) VALUES
('alice@example.com', 'Alice', '2024-01-01 10:00:00'),
('alice@example.com', 'Alice Smith', '2024-03-15 14:30:00'),
('bob@example.com', 'Bob', '2024-02-01 09:00:00'),
('bob@example.com', 'Bob Jones', '2024-04-10 11:00:00'),
('carol@example.com', 'Carol', '2024-01-20 08:00:00');

-- Rank duplicates by email, keeping the most recent
WITH ranked AS (
    SELECT
        id,
        email,
        name,
        updated_at,
        ROW_NUMBER() OVER (
            PARTITION BY email
            ORDER BY updated_at DESC
        ) AS rn
    FROM customers_raw
)
SELECT * FROM ranked WHERE rn = 1;
```

**Expected Output:**

```
id | email             | name        | updated_at          | rn
---+-------------------+-------------+---------------------+----
2  | alice@example.com | Alice Smith | 2024-03-15 14:30:00 | 1
4  | bob@example.com   | Bob Jones   | 2024-04-10 11:00:00 | 1
5  | carol@example.com | Carol       | 2024-01-20 08:00:00 | 1
```

**Why This Works:** The `PARTITION BY email` groups rows with the same email. `ORDER BY updated_at DESC` assigns `rn = 1` to the most recent row in each group. The final `WHERE rn = 1` returns only the canonical (most recent) record for each email.

#### Example 2: MySQL 8.0+ Delete Duplicates

```sql
-- Create a table with duplicates
CREATE TABLE raw_events (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    event_type VARCHAR(50),
    received_at TIMESTAMP
);

INSERT INTO raw_events (user_id, event_type, received_at) VALUES
(1, 'click', '2024-06-01 10:00:00'),
(1, 'click', '2024-06-01 10:00:05'),
(1, 'click', '2024-06-01 10:00:10'),
(2, 'view', '2024-06-01 11:00:00'),
(2, 'view', '2024-06-01 11:00:02');

-- Delete duplicates, keeping the earliest occurrence
DELETE re FROM raw_events re
JOIN (
    SELECT id,
           ROW_NUMBER() OVER (
               PARTITION BY user_id, event_type
               ORDER BY received_at ASC
           ) AS rn
    FROM raw_events
) ranked ON re.id = ranked.id
WHERE ranked.rn > 1;
```

**Expected Output (after delete):**

```
id | user_id | event_type | received_at
---+---------+------------+---------------------
1  | 1       | click      | 2024-06-01 10:00:00
4  | 2       | view       | 2024-06-01 11:00:00
```

**Why This Works:** The subquery ranks events by `user_id` and `event_type`, ordering by `received_at ASC`. Rows with `rn > 1` are duplicates. The `DELETE ... JOIN` removes them, keeping only the earliest event per user-event type combination.

#### Example 3: SQL Server Deduplication with `CTE` and `DELETE`

```sql
-- Create a table with duplicate product records
CREATE TABLE products_raw (
    id INT IDENTITY(1,1),
    sku VARCHAR(20),
    name VARCHAR(100),
    price DECIMAL(10,2)
);

INSERT INTO products_raw (sku, name, price) VALUES
('ABC-001', 'Widget', 19.99),
('ABC-001', 'Widget Pro', 24.99),
('DEF-002', 'Gadget', 29.99),
('DEF-002', 'Gadget', 29.99);

-- Delete duplicates, keeping the highest price (most premium version)
WITH ranked AS (
    SELECT
        id,
        ROW_NUMBER() OVER (
            PARTITION BY sku
            ORDER BY price DESC
        ) AS rn
    FROM products_raw
)
DELETE FROM ranked WHERE rn > 1;
```

**Expected Output (after delete):**

```
id | sku     | name       | price
---+---------+------------+-------
2  | ABC-001 | Widget Pro | 24.99
3  | DEF-002 | Gadget     | 29.99
```

**Why This Works:** The CTE ranks rows by `sku`, ordering by `price DESC`. The row with the highest price gets `rn = 1`; all others get `rn > 1`. The `DELETE FROM ranked` directly deletes from the CTE, which propagates to the underlying table. This keeps the most expensive (presumably most complete) version of each product.

### Real-World Cases

- **Customer 360:** Deduplicating customer records that were imported from multiple source systems, keeping the most recently updated record.
- **Event stream processing:** Removing duplicate events caused by at-least-once delivery semantics in message queues.
- **Product catalog:** Eliminating duplicate SKUs after a merger or acquisition.
- **Data migration:** Cleaning up duplicates before creating a unique index on a natural key.

### References

- PostgreSQL Documentation: Window Functions — https://www.postgresql.org/docs/current/functions-window.html
- PostgreSQL Documentation: `ROW_NUMBER()` — https://www.postgresql.org/docs/current/functions-window.html#FUNCTIONS-WINDOW-TABLE
- MySQL Reference Manual: Window Functions — https://dev.mysql.com/doc/refman/8.0/en/window-functions.html
- SQL Server: `ROW_NUMBER()` (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/functions/row-number-transact-sql
- Oracle Database SQL Language Reference: Analytic Functions — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/Analytic-Functions.html


## Core Concept 5: Standardization

### Definitions

**Core Definition:** Standardization is the process of enforcing uniform formats and representations for data elements—such as dates, phone numbers, addresses, and localized units—across a dataset.

**Technical Definition:** Standardization applies deterministic transformations to convert heterogeneous representations of the same semantic value into a single canonical form. This includes converting all dates to ISO 8601 (`YYYY-MM-DD`), normalizing phone numbers to E.164 format, parsing and reformatting addresses into structured components, and converting localized units (imperial to metric, local currency to base currency) to a common measurement system.

**Beginner-Friendly Explanation:** Standardization is like making everyone on a team use the same units and formats. Instead of some people writing dates as "01/15/2024" and others as "15-Jan-2024", everyone uses "2024-01-15". It makes data easier to compare and combine.

### Purposes

- To enable reliable joins and comparisons across datasets that use different date formats.
- To normalize phone numbers for deduplication and communication systems.
- To convert localized measurements (miles, pounds, gallons) to a global standard (kilometers, kilograms, liters).
- To structure free-text addresses into components (street, city, postal code, country) for geocoding and logistics.

### Syntax Rules and Structure

#### Complete General Syntax (Date Standardization)

```sql
-- PostgreSQL: parse various formats and output ISO 8601
SELECT
    raw_date,
    TO_DATE(raw_date, 'MM/DD/YYYY') AS parsed_date,
    TO_CHAR(TO_DATE(raw_date, 'MM/DD/YYYY'), 'YYYY-MM-DD') AS iso_date
FROM raw_dates;

-- MySQL: parse and reformat
SELECT
    raw_date,
    STR_TO_DATE(raw_date, '%m/%d/%Y') AS parsed_date,
    DATE_FORMAT(STR_TO_DATE(raw_date, '%m/%d/%Y'), '%Y-%m-%d') AS iso_date
FROM raw_dates;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `TO_DATE` / `STR_TO_DATE` | Parses a string into a date using a format model. |
| `TO_CHAR` / `DATE_FORMAT` | Formats a date as a string using a format model. |
| `'MM/DD/YYYY'` / `'%m/%d/%Y'` | Format model describing the input string. |
| `'YYYY-MM-DD'` / `'%Y-%m-%d'` | Format model for the ISO 8601 output. |

#### Complete General Syntax (Phone Number Standardization)

```sql
-- Strip non-digits, then format as (XXX) XXX-XXXX
SELECT
    phone_raw,
    REGEXP_REPLACE(phone_raw, '[^0-9]', '', 'g') AS digits_only,
    '(' || SUBSTRING(REGEXP_REPLACE(phone_raw, '[^0-9]', '', 'g') FROM 1 FOR 3) || ') '
    || SUBSTRING(REGEXP_REPLACE(phone_raw, '[^0-9]', '', 'g') FROM 4 FOR 3) || '-'
    || SUBSTRING(REGEXP_REPLACE(phone_raw, '[^0-9]', '', 'g') FROM 7 FOR 4) AS formatted
FROM raw_phones;
```

#### Syntax Rules

- **Date formats:** ISO 8601 (`YYYY-MM-DD`) is the international standard and the only format that sorts correctly as a string. Always output dates in ISO 8601 for interchange.
- **Phone numbers:** E.164 is the international standard for phone numbers (`+[country code][national number]`, e.g., `+14155552671`). Standardization involves stripping non-digits, adding the country code, and optionally formatting for display.
- **Addresses:** Use a postal address parser (libpostal, Google Address Validation API) for production-grade standardization; SQL alone cannot reliably parse free-text addresses.
- **Units:** Use conversion factors (1 mile = 1.60934 km, 1 lb = 0.453592 kg) applied via `CASE` expressions or a lookup table.

#### Constraints and Limitations

- **Ambiguous dates:** Without knowing the source format, `01/02/2024` could be January 2 or February 1. Standardization requires format metadata or heuristic rules.
- **Phone number country codes:** Without a country code, it is impossible to reliably convert to E.164; use a lookup table keyed by the source country.
- **Address parsing:** SQL string functions are insufficient for robust address parsing; use a dedicated library or API.
- **Unit conversion:** Floating-point precision loss can occur with repeated conversions; store the original value and unit alongside the converted value.
- **Version-specific:** MySQL `STR_TO_DATE` and `DATE_FORMAT` are available in all versions; PostgreSQL `TO_DATE` and `TO_CHAR` are available in all versions.

### Annotated Code Examples

#### Example 1: PostgreSQL Date Standardization

```sql
-- Create a table with dates in mixed formats
CREATE TABLE raw_dates (
    id SERIAL PRIMARY KEY,
    date_string TEXT
);

INSERT INTO raw_dates (date_string) VALUES
('2024-01-15'),
('01/20/2024'),
('15-Mar-2024'),
('2024/04/10');

-- Standardize to ISO 8601
SELECT
    id,
    date_string,
    CASE
        WHEN date_string ~ '^\d{4}-\d{2}-\d{2}$'
            THEN date_string
        WHEN date_string ~ '^\d{2}/\d{2}/\d{4}$'
            THEN TO_CHAR(TO_DATE(date_string, 'MM/DD/YYYY'), 'YYYY-MM-DD')
        WHEN date_string ~ '^\d{2}-[A-Za-z]{3}-\d{4}$'
            THEN TO_CHAR(TO_DATE(date_string, 'DD-Mon-YYYY'), 'YYYY-MM-DD')
        WHEN date_string ~ '^\d{4}/\d{2}/\d{2}$'
            THEN TO_CHAR(TO_DATE(date_string, 'YYYY/MM/DD'), 'YYYY-MM-DD')
        ELSE NULL
    END AS iso_date
FROM raw_dates;
```

**Expected Output:**

```
id | date_string   | iso_date
---+---------------+------------
1  | 2024-01-15    | 2024-01-15
2  | 01/20/2024    | 2024-01-20
3  | 15-Mar-2024   | 2024-03-15
4  | 2024/04/10    | 2024-04-10
```

**Why This Works:** The `CASE` expression uses regular expressions to identify the format of each date string, then applies the appropriate `TO_DATE` format model. `TO_CHAR` outputs the standardized ISO 8601 format. This approach handles multiple input formats deterministically.

#### Example 2: MySQL Phone Number Standardization to E.164

```sql
-- Create a table with US phone numbers in various formats
CREATE TABLE raw_phones (
    id INT AUTO_INCREMENT PRIMARY KEY,
    phone_raw VARCHAR(30),
    country_code VARCHAR(5) DEFAULT '1'
);

INSERT INTO raw_phones (phone_raw) VALUES
('(555) 123-4567'),
('555.123.4567'),
('5551234567'),
('+1 555 123 4567'),
('1-555-123-4567');

-- Standardize to E.164 format: +[country code][digits]
SELECT
    id,
    phone_raw,
    CONCAT('+', country_code, REGEXP_REPLACE(phone_raw, '[^0-9]', '')) AS e164
FROM raw_phones;
```

**Expected Output:**

```
id | phone_raw         | e164
---+-------------------+----------------
1  | (555) 123-4567    | +15551234567
2  | 555.123.4567      | +15551234567
3  | 5551234567        | +15551234567
4  | +1 555 123 4567   | +115551234567
5  | 1-555-123-4567    | +115551234567
```

**Why This Works:** `REGEXP_REPLACE` strips all non-digit characters. `CONCAT('+', country_code, ...)` prepends the country code. Note that rows 4 and 5 already include the country code (`1`), so the result duplicates it. A more robust solution would detect and strip a leading `1` before prepending. This illustrates a common pitfall in phone number standardization.

#### Example 3: SQL Server Unit Conversion (Imperial to Metric)

```sql
-- Create a table with measurements in imperial units
CREATE TABLE raw_measurements (
    id INT IDENTITY(1,1),
    distance_miles DECIMAL(10,2),
    weight_pounds DECIMAL(10,2),
    volume_gallons DECIMAL(10,2)
);

INSERT INTO raw_measurements (distance_miles, weight_pounds, volume_gallons) VALUES
(10.00, 150.00, 5.00),
(26.20, 200.00, 13.20),
(3.10, 50.00, 1.00);

-- Convert to metric: miles → km, pounds → kg, gallons → liters
SELECT
    id,
    distance_miles,
    ROUND(distance_miles * 1.60934, 2) AS distance_km,
    weight_pounds,
    ROUND(weight_pounds * 0.453592, 2) AS weight_kg,
    volume_gallons,
    ROUND(volume_gallons * 3.78541, 2) AS volume_liters
FROM raw_measurements;
```

**Expected Output:**

```
id | distance_miles | distance_km | weight_pounds | weight_kg | volume_gallons | volume_liters
---+----------------+-------------+---------------+-----------+----------------+---------------
1  | 10.00          | 16.09       | 150.00        | 68.04     | 5.00           | 18.93
2  | 26.20          | 42.16       | 200.00        | 90.72     | 13.20          | 49.97
3  | 3.10           | 4.99        | 50.00         | 22.68     | 1.00           | 3.79
```

**Why This Works:** The conversion factors (1 mile = 1.60934 km, 1 lb = 0.453592 kg, 1 gallon = 3.78541 L) are applied as simple multiplication. `ROUND(..., 2)` limits the result to two decimal places. Storing the original imperial values alongside the converted metric values preserves the source data for auditing.

### Real-World Cases

- **Global e-commerce:** Converting product weights and dimensions from imperial to metric for international shipping.
- **Healthcare data integration:** Standardizing patient dates of birth, phone numbers, and addresses from multiple hospital systems.
- **Financial reporting:** Converting all monetary values to a base currency and all dates to ISO 8601 for consolidated reporting.
- **Telecommunications:** Normalizing phone numbers to E.164 for SMS gateways and call routing systems.

### References

- ISO 8601: Date and Time Format — https://www.iso.org/iso-8601-date-and-time-format.html
- ITU-T E.164: The International Public Telecommunication Numbering Plan — https://www.itu.int/rec/T-REC-E.164/
- PostgreSQL Documentation: Data Type Formatting Functions — https://www.postgresql.org/docs/current/functions-formatting.html
- MySQL Reference Manual: Date and Time Functions — https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html
- SQL Server: FORMAT (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/functions/format-transact-sql
- Oracle Database SQL Language Reference: Format Models — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/Format-Models.html


## Core Concept 6: Data Validation

### Definitions

**Core Definition:** Data validation is the process of enforcing rules and constraints on data to ensure it conforms to expected domains, formats, and business rules, with mechanisms for rejecting or routing invalid rows.

**Technical Definition:** Data validation uses declarative constraints—`CHECK` constraints, `NOT NULL`, `UNIQUE`, `FOREIGN KEY`, and domain types—to enforce domain integrity at the database level. Runtime validation can be implemented via `CASE` expressions that route invalid rows to an error table, or via `TRY_CAST`/`TRY_CONVERT` that return `NULL` instead of raising errors. Error-rejection routing uses `INSERT ... SELECT ... WHERE condition` to split valid and invalid rows into separate tables.

**Beginner-Friendly Explanation:** Data validation is like a bouncer at a club. The bouncer checks that everyone entering meets the rules (age limit, dress code). If someone does not meet the rules, they are turned away. In SQL, `CHECK` constraints and validation queries make sure only valid data gets into your tables.

### Purposes

- To enforce domain integrity by rejecting values that fall outside defined ranges or formats.
- To validate foreign key references before insertion, routing orphan rows to an error log.
- To apply business rules (e.g., "salary must be positive", "end_date must be after start_date") at the database level.
- To create an auditable trail of rejected rows for data quality monitoring.

### Syntax Rules and Structure

#### Complete General Syntax (`CHECK` Constraint)

```sql
CREATE TABLE table_name (
    column1 data_type CHECK (condition),
    column2 data_type,
    ...
    CONSTRAINT constraint_name CHECK (condition)
);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `CHECK (condition)` | A boolean expression that must evaluate to `TRUE` or `UNKNOWN` (NULL) for the row to be accepted. |
| `constraint_name` | Optional name for the constraint; useful for error reporting. |

#### Complete General Syntax (Domain Validation)

```sql
CREATE DOMAIN domain_name AS data_type
CHECK (condition);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `domain_name` | The name of the custom domain. |
| `data_type` | The base data type (e.g., `TEXT`, `INTEGER`). |
| `CHECK (condition)` | A constraint applied to all columns using this domain. |

#### Complete General Syntax (Error-Rejection Routing)

```sql
-- Stage 1: Insert valid rows
INSERT INTO target_table (columns)
SELECT columns
FROM staging_table
WHERE validation_condition;

-- Stage 2: Insert invalid rows into error table
INSERT INTO error_table (columns, error_reason)
SELECT columns, 'validation failed: reason'
FROM staging_table
WHERE NOT validation_condition;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `validation_condition` | A boolean expression that valid rows must satisfy. |
| `target_table` | The destination for valid rows. |
| `error_table` | A table that captures rejected rows and the reason for rejection. |

#### Syntax Rules

- `CHECK` constraints are evaluated on `INSERT` and `UPDATE`; if the condition evaluates to `FALSE`, the statement is rejected. If it evaluates to `UNKNOWN` (NULL), the row is accepted (SQL three-valued logic).
- Domain constraints are inherited by all columns that use the domain; they are more maintainable than per-column constraints when the same rule applies to multiple tables.
- `NOT NULL` is a separate constraint; `CHECK` cannot enforce non-nullness because `NULL` evaluates to `UNKNOWN`.
- Error-rejection routing requires a staging table and two target tables (valid and error); this is a common pattern in ETL pipelines.
- In PostgreSQL, `CREATE DOMAIN` with `CHECK` provides reusable validation logic.

#### Constraints and Limitations

- **`CHECK` constraint order:** The order in which constraints are evaluated is not defined; do not rely on a specific order.
- **Subqueries in `CHECK`:** Most databases do not allow subqueries in `CHECK` constraints (SQL Server explicitly does not).
- **Domain constraint maintenance:** Domain constraints are more difficult to list on a per-table basis because they are defined outside the table.
- **Performance:** `CHECK` constraints add a small overhead to every `INSERT` and `UPDATE`; validate in the application layer for high-throughput OLTP systems and reserve `CHECK` for critical integrity rules.
- **Version-specific:** PostgreSQL supports `CREATE DOMAIN`; MySQL does not have domains but supports `CHECK` constraints (MySQL 8.0.16+ enforces them; earlier versions parsed but ignored them).

### Annotated Code Examples

#### Example 1: PostgreSQL Domain Validation for Postal Codes

```sql
-- Create a domain for French postal codes (5 digits)
CREATE DOMAIN french_postal_code AS TEXT
CHECK (value ~ '^\d{5}$');

-- Create a table using the domain
CREATE TABLE addresses (
    id SERIAL PRIMARY KEY,
    street TEXT,
    city TEXT,
    postal_code french_postal_code
);

-- Valid insert
INSERT INTO addresses (street, city, postal_code)
VALUES ('1 Rue de la Paix', 'Paris', '75001');

-- Invalid insert (4 digits)
INSERT INTO addresses (street, city, postal_code)
VALUES ('2 Rue de la Paix', 'Paris', '7500');
```

**Expected Output (for the invalid insert):**

```
ERROR:  value for domain french_postal_code violates check constraint "french_postal_code_check"
```

**Why This Works:** The domain `french_postal_code` enforces the regex `^\d{5}$` for all columns using it. The first insert succeeds because `'75001'` has five digits. The second insert fails because `'7500'` has four digits. The error message identifies the domain and constraint name, aiding debugging.

#### Example 2: MySQL `CHECK` Constraint for Business Rules

```sql
-- Create a table with CHECK constraints
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    salary DECIMAL(10,2) CHECK (salary > 0),
    hire_date DATE,
    termination_date DATE,
    CONSTRAINT chk_dates CHECK (termination_date IS NULL OR termination_date > hire_date)
);

-- Valid insert
INSERT INTO employees (name, salary, hire_date) VALUES ('Alice', 50000, '2024-01-15');

-- Invalid: negative salary
INSERT INTO employees (name, salary) VALUES ('Bob', -1000);

-- Invalid: termination before hire
INSERT INTO employees (name, salary, hire_date, termination_date)
VALUES ('Carol', 60000, '2024-01-01', '2023-12-31');
```

**Expected Output (for invalid inserts):**

```
ERROR 3819 (HY000): Check constraint 'employees_chk_1' is violated.
ERROR 3819 (HY000): Check constraint 'chk_dates' is violated.
```

**Why This Works:** The `CHECK (salary > 0)` constraint rejects negative salaries. The named constraint `chk_dates` enforces that `termination_date` is either `NULL` or after `hire_date`. MySQL 8.0.16+ enforces `CHECK` constraints; earlier versions parsed them but did not enforce them.

#### Example 3: PostgreSQL Error-Rejection Routing

```sql
-- Create staging table
CREATE TABLE staging_orders (
    order_id TEXT,
    customer_email TEXT,
    amount TEXT
);

INSERT INTO staging_orders VALUES
('ORD-001', 'alice@example.com', '150.00'),
('ORD-002', 'invalid-email', '200.00'),
('ORD-003', 'bob@example.com', 'not_a_number'),
('ORD-004', 'carol@example.com', '75.50');

-- Create valid and error tables
CREATE TABLE valid_orders (
    order_id TEXT,
    customer_email TEXT,
    amount NUMERIC(10,2)
);

CREATE TABLE error_orders (
    order_id TEXT,
    customer_email TEXT,
    amount TEXT,
    error_reason TEXT
);

-- Route valid rows
INSERT INTO valid_orders (order_id, customer_email, amount)
SELECT
    order_id,
    customer_email,
    amount::NUMERIC
FROM staging_orders
WHERE customer_email ~ '^[^@]+@[^@]+\.[^@]+$'
  AND amount ~ '^\d+(\.\d+)?$';

-- Route invalid rows to error table
INSERT INTO error_orders (order_id, customer_email, amount, error_reason)
SELECT
    order_id,
    customer_email,
    amount,
    CASE
        WHEN customer_email !~ '^[^@]+@[^@]+\.[^@]+$'
             AND amount !~ '^\d+(\.\d+)?$' THEN 'invalid email and amount'
        WHEN customer_email !~ '^[^@]+@[^@]+\.[^@]+$' THEN 'invalid email'
        WHEN amount !~ '^\d+(\.\d+)?$' THEN 'invalid amount'
        ELSE 'unknown'
    END
FROM staging_orders
WHERE NOT (
    customer_email ~ '^[^@]+@[^@]+\.[^@]+$'
    AND amount ~ '^\d+(\.\d+)?$'
);
```

**Expected Output (valid_orders):**

```
order_id | customer_email     | amount
---------+--------------------+--------
ORD-001  | alice@example.com  | 150.00
ORD-004  | carol@example.com  |  75.50
```

**Expected Output (error_orders):**

```
order_id | customer_email     | amount     | error_reason
---------+--------------------+------------+----------------------
ORD-002  | invalid-email      | 200.00     | invalid email
ORD-003  | bob@example.com    | not_a_number | invalid amount
```

**Why This Works:** The `INSERT ... SELECT ... WHERE condition` pattern routes valid rows to `valid_orders`. The `WHERE NOT (...)` clause routes invalid rows to `error_orders`. The `CASE` expression provides a human-readable reason for each rejection. This pattern provides an auditable trail of data quality issues.

### Real-World Cases

- **Financial systems:** Enforcing that transaction amounts are positive and account balances never go negative.
- **Healthcare:** Validating that patient dates of birth are in the past and that diagnosis codes match a valid ICD-10 list.
- **E-commerce:** Validating that product prices are positive, SKUs match a pattern, and inventory quantities are non-negative.
- **ETL pipelines:** Routing malformed rows from a staging table to an error table for manual review and correction.

### References

- PostgreSQL Documentation: Constraints (`CHECK`, `NOT NULL`, `UNIQUE`) — https://www.postgresql.org/docs/current/ddl-constraints.html
- PostgreSQL Documentation: `CREATE DOMAIN` — https://www.postgresql.org/docs/current/sql-createdomain.html
- MySQL Reference Manual: `CHECK` Constraints — https://dev.mysql.com/doc/refman/8.0/en/create-table-check-constraints.html
- SQL Server: `CHECK` Constraints — https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-check-constraints
- Oracle Database SQL Language Reference: Constraints — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/constraint.html
- ISO/IEC 9075-2:2016 (SQL Foundation) — Domain Constraints — https://www.iso.org/standard/63555.html


## Summary Table: Data Transformation Techniques

| Concept | Primary SQL Constructs | Typical Use Case |
|---------|----------------------|------------------|
| Type Conversion | `CAST`, `CONVERT`, `TRY_CAST`, `AT TIME ZONE` | Prepare data for arithmetic, comparison, time zone normalization |
| String Transformation | `REGEXP_REPLACE`, `TRIM`, `UPPER`, `LOWER`, `INITCAP` | Clean names, emails, extract patterns |
| Data Cleansing | `COALESCE`, `NULLIF`, `LEFT JOIN ... IS NULL` | Handle NULLs, sentinel values, orphaned records |
| Deduplication | `ROW_NUMBER() OVER (PARTITION BY ...)` | Remove duplicate rows, keep canonical record |
| Standardization | `TO_DATE`, `STR_TO_DATE`, `REGEXP_REPLACE`, conversion factors | Uniform dates, phone numbers, units |
| Data Validation | `CHECK`, `CREATE DOMAIN`, `INSERT ... WHERE` | Enforce domain rules, route malformed rows |


## Final Notes on Deprecated and Unsafe Features

- **MySQL `CHECK` constraints before 8.0.16:** Parsed but ignored; do not rely on them for validation in older versions.
- **PostgreSQL `CREATE DOMAIN`:** Safe and recommended for reusable validation; note that domain constraints are harder to list per-table than table-level `CHECK` constraints.
- **SQL Server `ISNULL`:** Limited to two arguments; prefer `COALESCE` for portability.
- **Oracle `NVL`:** Proprietary; `COALESCE` is SQL-standard and preferred.
- **Implicit type conversion:** Can silently corrupt data or prevent index usage; always use explicit casts in critical operations.
- **`ROW_NUMBER()` without `ORDER BY`:** Not allowed; the `ORDER BY` clause is mandatory and determines which row is retained.
- **Deleting orphans without backup:** Irreversible; always stage or back up before deleting orphaned rows.
- **Version-specific:** PostgreSQL 15+ supports `NULLS NOT DISTINCT` in unique indexes; MySQL 8.0+ supports window functions and `CHECK` enforcement; SQL Server 2012+ supports `TRY_CAST`/`TRY_CONVERT`.