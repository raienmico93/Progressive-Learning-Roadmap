# SQL String Functions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL string functions are built-in operations that examine, transform, extract, modify, concatenate, and search within character data (CHAR, VARCHAR, TEXT, etc.).

**Technical Definition:** Per the ANSI/ISO SQL Standard (ISO/IEC 9075), string functions operate on character string data types and return either string values or numeric values. The SQL-92 standard defined core string manipulation functions including concatenation (`||`), SUBSTRING, POSITION, TRIM, UPPER/LOWER, and CHARACTER_LENGTH. Subsequent standards and vendor extensions have added dozens more functions. String functions are generally deterministic and can be used in SELECT, WHERE, ORDER BY, and GROUP BY clauses.

**Beginner-Friendly Explanation:** String functions are tools for working with text in a database. Just like a word processor lets you uppercase, trim, replace, or find text, SQL string functions let you do the same thing to data stored in database columns. For example, `UPPER('hello')` turns text into `'HELLO'`, `SUBSTRING('Hello World', 1, 5)` extracts `'Hello'`, and `TRIM('  hello  ')` removes extra spaces.

### Key Characteristics

- **Two categories:** Functions that return strings (e.g., UPPER, SUBSTRING, CONCAT) and functions that return numbers (e.g., LENGTH, POSITION, ASCII).
- **Standard vs. vendor:** Core functions (TRIM, SUBSTRING, POSITION, UPPER, LOWER, CHARACTER_LENGTH) are ANSI standard. Others (LEN, INSTR, PATINDEX, STUFF) are vendor-specific.
- **NULL propagation:** Most string functions return NULL if any argument is NULL, with exceptions like CONCAT and CONCAT_WS which treat NULL specially.
- **1-based positioning:** Most databases number string positions starting at 1.
- **Collation-sensitive:** Case conversion and comparison behavior depends on the database collation.

### Prerequisites

- Basic SQL syntax (SELECT, FROM, WHERE)
- Understanding of character data types (CHAR, VARCHAR, TEXT)
- Familiarity with NULL handling

### Related Programming Areas

- **Data Cleaning and ETL:** String functions are essential for normalizing, validating, and transforming text data.
- **Data Validation:** Checking formats, lengths, and patterns.
- **Report Generation:** Formatting text output for presentation.
- **Search Functionality:** Finding and extracting substrings.
- **Application Development:** ORMs and query builders expose string functions through APIs.

---

## 1. Measurement & Inspection

### 1.1 Character Length: LENGTH(str) or LEN(str)

**Core Definitions:**

- **Core Definition:** Returns the number of characters in a string.
- **Technical Definition:** `CHARACTER_LENGTH(string)` is the ANSI SQL standard function that returns the number of characters in a string. `LENGTH(string)` is the most common vendor implementation. `LEN(string)` is SQL Server's implementation, which returns the number of characters excluding trailing spaces. `CHAR_LENGTH()` is a synonym for `CHARACTER_LENGTH()`.
- **Beginner-Friendly Explanation:** Counts how many characters are in a piece of text. `'Hello'` has 5 characters.

**Purposes:**
- To validate that input meets length requirements
- To identify strings that are too long or too short
- To use in sorting or grouping by length

**Syntax Structures and Rules:**

```sql
-- Standard ANSI
CHARACTER_LENGTH(string)

-- Common vendor syntax
LENGTH(string)       -- PostgreSQL, MySQL, Oracle, SQLite
LEN(string)          -- SQL Server
CHAR_LENGTH(string)  -- MySQL synonym for CHARACTER_LENGTH
```

**Component Breakdown:**
- `string`: The string expression to measure

**Syntax Rules:**
- Returns an integer.
- Returns NULL if the input is NULL.
- SQL Server's LEN excludes trailing spaces; LENGTH includes them.
- For multibyte character sets, CHARACTER_LENGTH counts characters, not bytes.

**Constraints and Limitations:**
- LENGTH in MySQL returns bytes, not characters; use CHAR_LENGTH for characters.
- SQL Server's LEN ignores trailing spaces.

**Annotated Code Examples:**

```sql
-- Measure the length of a string
SELECT LENGTH('Hello World') AS char_length;        -- 11 (PostgreSQL, MySQL)
SELECT LEN('Hello World') AS char_length;            -- 11 (SQL Server)
SELECT CHARACTER_LENGTH('Hello World') AS char_length; -- 11 (standard)

-- Measure with trailing spaces
SELECT LEN('Hello   ') AS sql_server_len;            -- 5 (trailing spaces ignored)
SELECT LENGTH('Hello   ') AS postgres_len;           -- 8 (trailing spaces counted)
```

**Expected Output:**

| Query | Result |
|-------|--------|
| `LENGTH('Hello World')` | 11 |
| `LEN('Hello   ')` | 5 |
| `LENGTH('Hello   ')` | 8 |

**Real-World Cases:**
- **User registration:** Validating username length (3–20 characters).
- **Data cleaning:** Finding records with abnormally short or long values.
- **Reporting:** Grouping by name length for analysis.

**References:**
- PostgreSQL: String Functions and Operators - https://www.postgresql.org/docs/current/functions-string.html
- MySQL: String Functions - https://dev.mysql.com/doc/refman/8.0/en/string-functions.html
- Microsoft Learn: LEN (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/len-transact-sql
- SQL Standard: CHARACTER_LENGTH - https://sqlite.org/forum/forumpost/de86d9d98820efff?raw


### 1.2 Byte Length: OCTET_LENGTH(str) or LENGTH() (MySQL)

**Core Definitions:**

- **Core Definition:** Returns the number of bytes required to store a string.
- **Technical Definition:** `OCTET_LENGTH(string)` is the ANSI SQL standard function that returns the number of bytes (octets) in a string. In MySQL, `LENGTH(string)` returns the length in bytes. In PostgreSQL, `octet_length(string)` returns the number of bytes. In SQL Server, `DATALENGTH(string)` returns the number of bytes.
- **Beginner-Friendly Explanation:** Measures storage size, not character count. A character like 'é' may take 2 bytes in UTF-8, even though it's one character.

**Purposes:**
- To check storage requirements
- To validate data against byte-length limits
- To analyze multibyte character data

**Syntax Structures and Rules:**

```sql
-- Standard ANSI
OCTET_LENGTH(string)

-- MySQL: LENGTH returns bytes
LENGTH(string)  -- MySQL returns bytes

-- PostgreSQL
octet_length(string)

-- SQL Server
DATALENGTH(string)
```

**Annotated Code Examples:**

```sql
-- Compare character length vs byte length
SELECT 
    CHAR_LENGTH('café') AS char_length,   -- 4 characters
    LENGTH('café') AS byte_length;        -- 5 bytes (é is 2 bytes in UTF-8)
```

**Expected Output:**

| char_length | byte_length |
|-------------|-------------|
| 4           | 5           |

**Real-World Cases:**
- **Database sizing:** Estimating storage requirements for text columns.
- **API limits:** Ensuring data fits within byte-based transmission limits.
- **Internationalization:** Understanding multibyte character storage.

**References:**
- MySQL: String Functions (LENGTH) - https://dev.mysql.com/doc/refman/8.0/en/string-functions.html
- PostgreSQL: String Functions (octet_length) - https://www.postgresql.org/docs/current/functions-string.html


### 1.3 Data-Type Validation: ASCII(char) or UNICODE(char)

**Core Definitions:**

- **Core Definition:** Returns the numeric code of the leftmost character in a string.
- **Technical Definition:** `ASCII(string)` returns the ASCII code value of the leftmost character. `UNICODE(string)` returns the Unicode code point of the first character (SQL Server). `ORD(string)` (MySQL) returns the character code. In PostgreSQL, `ascii(text)` returns the ASCII code of the first character.
- **Beginner-Friendly Explanation:** Gives you the numeric code behind a character. `ASCII('A')` returns 65.

**Purposes:**
- To identify non-printable or special characters
- To validate character encoding
- To implement custom sorting or classification

**Syntax Structures and Rules:**

```sql
ASCII(string)     -- MySQL, SQL Server, PostgreSQL
UNICODE(string)   -- SQL Server (Unicode code point)
ORD(string)       -- MySQL (character code)
```

**Annotated Code Examples:**

```sql
SELECT ASCII('A') AS ascii_a,       -- 65
       ASCII('a') AS ascii_a_lower, -- 97
       ASCII(' ') AS ascii_space;   -- 32
```

**Expected Output:**

| ascii_a | ascii_a_lower | ascii_space |
|---------|---------------|-------------|
| 65      | 97            | 32          |

**Real-World Cases:**
- **Data cleaning:** Detecting and removing control characters.
- **Validation:** Ensuring strings start with a letter or number.

**References:**
- MySQL: String Functions (ASCII, ORD) - https://dev.mysql.com/doc/refman/8.0/en/string-functions.html
- Microsoft Learn: ASCII - https://learn.microsoft.com/en-us/sql/t-sql/functions/ascii-transact-sql
- PostgreSQL: String Functions (ascii) - https://www.postgresql.org/docs/current/functions-string.html


## 2. Case Transformation & Formatting

### 2.1 Case Normalization: UPPER(str) and LOWER(str)

**Core Definitions:**

- **Core Definition:** UPPER converts all characters to uppercase; LOWER converts all characters to lowercase.
- **Technical Definition:** `UPPER(string)` and `LOWER(string)` are ANSI SQL standard functions that transform case according to the database's locale. Synonyms include `UCASE` (MySQL) and `LCASE` (MySQL).
- **Beginner-Friendly Explanation:** UPPER makes text ALL CAPS; LOWER makes it all lowercase.

**Purposes:**
- To normalize data for case-insensitive comparison
- To format output for display
- To clean inconsistent data entry

**Syntax Structures and Rules:**

```sql
UPPER(string)  -- Standard
LOWER(string)  -- Standard
UCASE(string)  -- MySQL synonym
LCASE(string)  -- MySQL synonym
```

**Annotated Code Examples:**

```sql
SELECT UPPER('Hello World') AS upper_case,   -- 'HELLO WORLD'
       LOWER('Hello World') AS lower_case;   -- 'hello world'
```

**Real-World Cases:**
- **Login systems:** Case-insensitive username comparison: `WHERE LOWER(username) = LOWER('Alice')`.
- **Data cleaning:** Standardizing state codes, country codes.
- **Reports:** Formatting names for display.

**References:**
- PostgreSQL: String Functions (upper, lower) - https://www.postgresql.org/docs/current/functions-string.html
- MySQL: String Functions (UPPER, LOWER) - https://dev.mysql.com/doc/refman/8.0/en/string-functions.html
- Microsoft Learn: UPPER - https://learn.microsoft.com/en-us/sql/t-sql/functions/upper-transact-sql


### 2.2 Title Casing: INITCAP(str)

**Core Definitions:**

- **Core Definition:** INITCAP capitalizes the first letter of each word and lowercases the rest.
- **Technical Definition:** `INITCAP(string)` is available in PostgreSQL, Oracle, and some other databases. It is not part of the ANSI SQL standard. SQL Server does not have a built-in INITCAP; a workaround using string manipulation is required.
- **Beginner-Friendly Explanation:** Turns "hello world" into "Hello World".

**Purposes:**
- To format names and titles consistently
- To clean data entered in all caps or all lowercase
- To improve readability of display text

**Syntax Structures and Rules:**

```sql
INITCAP(string)  -- PostgreSQL, Oracle
```

**Annotated Code Examples:**

```sql
SELECT INITCAP('hello world') AS title_case;  -- 'Hello World'
```

**Real-World Cases:**
- **CRM:** Formatting customer names for display.
- **CMS:** Formatting article titles.
- **Reports:** Standardizing address formatting.

**References:**
- PostgreSQL: String Functions (initcap) - https://www.postgresql.org/docs/current/functions-string.html
- Oracle: SQL Reference (INITCAP) - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/INITCAP.html


### 2.3 Reversal: REVERSE(str)

**Core Definitions:**

- **Core Definition:** REVERSE returns a string with the order of characters reversed.
- **Technical Definition:** `REVERSE(string)` returns the string with all characters in reverse order. It is supported by SQL Server, MySQL, PostgreSQL, and Oracle. It is not part of the ANSI SQL standard.
- **Beginner-Friendly Explanation:** `REVERSE('Hello')` returns `'olleH'`.

**Purposes:**
- To implement custom sorting or comparison logic
- To find palindromes or reversed patterns
- To manipulate string data in creative ways

**Syntax Structures and Rules:**

```sql
REVERSE(string)
```

**Annotated Code Examples:**

```sql
SELECT REVERSE('Hello') AS reversed;  -- 'olleH'
```

**Real-World Cases:**
- **Data obfuscation:** Reversing sensitive values for simple masking.
- **Text processing:** Finding palindromes or anagram patterns.
- **Custom sorting:** Sorting by suffix instead of prefix.

**References:**
- MySQL: String Functions (REVERSE) - https://dev.mysql.com/doc/refman/8.0/en/string-functions.html
- Microsoft Learn: REVERSE - https://learn.microsoft.com/en-us/sql/t-sql/functions/reverse-transact-sql
- PostgreSQL: String Functions (reverse) - https://www.postgresql.org/docs/current/functions-string.html


## 3. Slicing & Extraction

### 3.1 Positional Substrings: SUBSTRING(str FROM pos FOR len)

**Core Definitions:**

- **Core Definition:** SUBSTRING extracts a portion of a string starting at a specified position and optionally of a specified length.
- **Technical Definition:** The ANSI SQL standard defines `SUBSTRING(string FROM start [FOR length])`. Vendor implementations include `SUBSTRING(string, start, length)` (SQL Server, MySQL), `SUBSTR(string, start, length)` (Oracle, MySQL, PostgreSQL), and `MID()` (MySQL synonym).
- **Beginner-Friendly Explanation:** Cuts out a piece of text. `SUBSTRING('Hello World', 1, 5)` returns `'Hello'`.

**Purposes:**
- To extract meaningful parts of structured strings (e.g., area codes)
- To split strings into components
- To truncate strings for display

**Syntax Structures and Rules:**

```sql
-- ANSI standard
SUBSTRING(string FROM start [FOR length])

-- Common vendor syntax
SUBSTRING(string, start, length)   -- SQL Server, MySQL
SUBSTR(string, start, length)      -- Oracle, PostgreSQL, MySQL
MID(string, start, length)         -- MySQL synonym
```

**Component Breakdown:**
- `string`: The source string
- `start`: Starting position (1-based; negative values count from the end in some databases)
- `length`: Number of characters to extract (optional in some databases)

**Annotated Code Examples:**

```sql
SELECT SUBSTRING('Hello World', 1, 5) AS standard_sub,   -- 'Hello'
       SUBSTRING('Hello World', 7, 5) AS mid_sub;        -- 'World'

-- PostgreSQL: SUBSTRING with FROM/FOR
SELECT SUBSTRING('Hello World' FROM 1 FOR 5) AS pg_sub; -- 'Hello'
```

**Real-World Cases:**
- **Phone numbers:** Extracting area codes: `SUBSTRING(phone, 1, 3)`.
- **Dates:** Extracting year, month, day from formatted strings.
- **Name parsing:** Extracting first or last name from a full name field.

**References:**
- PostgreSQL: String Functions (substring) - https://www.postgresql.org/docs/current/functions-string.html
- Microsoft Learn: SUBSTRING - https://learn.microsoft.com/en-us/sql/t-sql/functions/substring-transact-sql
- Oracle: SUBSTR - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SUBSTR.html


### 3.2 Directional Slicing: LEFT(str, n) and RIGHT(str, n)

**Core Definitions:**

- **Core Definition:** LEFT returns the leftmost n characters; RIGHT returns the rightmost n characters.
- **Technical Definition:** `LEFT(string, n)` and `RIGHT(string, n)` are available in SQL Server, MySQL, PostgreSQL, and Oracle. They are not part of the ANSI SQL standard but are widely supported. If n exceeds the string length, the entire string is returned.
- **Beginner-Friendly Explanation:** LEFT takes from the beginning; RIGHT takes from the end.

**Purposes:**
- To extract prefixes or suffixes
- To safely slice strings without worrying about position
- To implement custom formatting

**Syntax Structures and Rules:**

```sql
LEFT(string, n)   -- First n characters
RIGHT(string, n)  -- Last n characters
```

**Annotated Code Examples:**

```sql
SELECT LEFT('Hello World', 5) AS left_part,   -- 'Hello'
       RIGHT('Hello World', 5) AS right_part;  -- 'World'
```

**Real-World Cases:**
- **File extensions:** `RIGHT(filename, 4)` to get the extension.
- **Codes:** `LEFT(code, 2)` to get a prefix.
- **Display:** Truncating long text for previews.

**References:**
- Microsoft Learn: LEFT - https://learn.microsoft.com/en-us/sql/t-sql/functions/left-transact-sql
- MySQL: String Functions (LEFT, RIGHT) - https://dev.mysql.com/doc/refman/8.0/en/string-functions.html


## 4. Text Cleaning & Modification

### 4.1 Whitespace Removal: TRIM(), LTRIM(), and RTRIM()

**Core Definitions:**

- **Core Definition:** TRIM removes leading and/or trailing characters (spaces by default); LTRIM removes leading characters; RTRIM removes trailing characters.
- **Technical Definition:** The ANSI SQL standard defines `TRIM([[LEADING|TRAILING|BOTH] [characters] FROM] string)`. Vendor implementations simplify this: `TRIM(string)` (SQL Server 2017+, MySQL, PostgreSQL), `LTRIM(string)` and `RTRIM(string)`. SQL Server 2017 added optional characters and LEADING/TRAILING/BOTH to TRIM.
- **Beginner-Friendly Explanation:** TRIM cleans up extra spaces at the beginning and end of text.

**Purposes:**
- To clean data entry with accidental spaces
- To standardize input for comparison
- To prepare data for concatenation or display

**Syntax Structures and Rules:**

```sql
-- ANSI standard
TRIM([[LEADING|TRAILING|BOTH] [characters] FROM] string)

-- Common vendor syntax
TRIM(string)         -- Removes spaces from both ends
LTRIM(string)        -- Removes leading spaces
RTRIM(string)        -- Removes trailing spaces
TRIM(string, characters)  -- MySQL: removes specified characters
```

**Annotated Code Examples:**

```sql
SELECT TRIM('  Hello World  ') AS trimmed,      -- 'Hello World'
       LTRIM('  Hello World  ') AS left_trim,   -- 'Hello World  '
       RTRIM('  Hello World  ') AS right_trim;  -- '  Hello World'

-- SQL Server 2017+: TRIM with characters
SELECT TRIM('x' FROM 'xxHelloxx') AS trim_x;    -- 'Hello'
```

**Real-World Cases:**
- **Data cleaning:** Removing extra spaces from imported data.
- **User input:** Standardizing form submissions.
- **Comparison:** Ensuring `'abc'` equals `' abc '` after trimming.

**References:**
- PostgreSQL: String Functions (trim, ltrim, rtrim) - https://www.postgresql.org/docs/current/functions-string.html
- Microsoft Learn: TRIM - https://learn.microsoft.com/en-us/sql/t-sql/functions/trim-transact-sql
- SQL Standard: TRIM - https://sqlite.org/forum/forumpost/de86d9d98820efff?raw


### 4.2 Padding & Alignment: LPAD() and RPAD()

**Core Definitions:**

- **Core Definition:** LPAD left-pads a string to a specified length with a specified character; RPAD right-pads.
- **Technical Definition:** `LPAD(string, length, pad_string)` extends the string to `length` by prepending characters from `pad_string`. If the string is already longer than `length`, it is truncated (on the right for LPAD, on the left for RPAD). Available in PostgreSQL, MySQL, and Oracle.
- **Beginner-Friendly Explanation:** LPAD fills in characters on the left to make the string a certain length; RPAD does the same on the right.

**Purposes:**
- To align fixed-width data (e.g., account numbers, codes)
- To format output with leading zeros
- To create visually aligned reports

**Syntax Structures and Rules:**

```sql
LPAD(string, length, pad_string)
RPAD(string, length, pad_string)
```

**Annotated Code Examples:**

```sql
SELECT LPAD('123', 6, '0') AS padded_left,    -- '000123'
       RPAD('123', 6, 'x') AS padded_right;   -- '123xxx'
```

**Real-World Cases:**
- **Account numbers:** `LPAD(account_id, 10, '0')` for display.
- **Fixed-width reports:** Aligning columns of text.
- **Codes:** Adding leading zeros to product codes.

**References:**
- PostgreSQL: String Functions (lpad, rpad) - https://www.postgresql.org/docs/current/functions-string.html
- MySQL: String Functions (LPAD, RPAD) - https://dev.mysql.com/doc/refman/8.0/en/string-functions.html


### 4.3 Character Replacement: REPLACE(str, target, replacement)

**Core Definitions:**

- **Core Definition:** REPLACE substitutes all occurrences of a target substring with a replacement string.
- **Technical Definition:** `REPLACE(string, target, replacement)` returns a string where all occurrences of `target` are replaced by `replacement`. If `target` is not found, the original string is returned. If any argument is NULL, the result is NULL (except MySQL, which returns the original string if target is NULL).
- **Beginner-Friendly Explanation:** Find and replace. `REPLACE('Hello World', 'World', 'Universe')` returns `'Hello Universe'`.

**Purposes:**
- To clean data by replacing unwanted characters
- To standardize values (e.g., replacing abbreviations)
- To mask sensitive data

**Syntax Structures and Rules:**

```sql
REPLACE(string, target, replacement)
```

**Annotated Code Examples:**

```sql
SELECT REPLACE('Hello World', 'World', 'Universe') AS replaced;  -- 'Hello Universe'
SELECT REPLACE('1-800-FLOWERS', '-', '') AS digits;               -- '1800FLOWERS'
```

**Real-World Cases:**
- **Data cleaning:** Removing commas from numbers: `REPLACE(amount, ',', '')`.
- **Phone numbers:** Standardizing format: `REPLACE(phone, ' ', '-')`.
- **Anonymization:** Replacing names with placeholders.

**References:**
- Microsoft Learn: REPLACE - https://learn.microsoft.com/en-us/sql/t-sql/functions/replace-transact-sql
- MySQL: String Functions (REPLACE) - https://dev.mysql.com/doc/refman/8.0/en/string-functions.html


### 4.4 Positional Overwriting: STUFF() (SQL Server) or INSERT() (MySQL)

**Core Definitions:**

- **Core Definition:** STUFF/INSERT deletes a specified number of characters from a string and inserts another string in their place.
- **Technical Definition:** `STUFF(string, start, length, replace_string)` deletes `length` characters from `string` starting at `start`, then inserts `replace_string` at `start`. SQL Server uses STUFF; MySQL and Oracle use INSERT (with the same arguments).
- **Beginner-Friendly Explanation:** STUFF cuts out a piece of text and puts new text in its place.

**Purposes:**
- To overwrite specific segments of a string
- To format codes by inserting separators
- To modify structured strings in place

**Syntax Structures and Rules:**

```sql
-- SQL Server
STUFF(string, start, length, replace_string)

-- MySQL, Oracle
INSERT(string, start, length, replace_string)
```

**Annotated Code Examples:**

```sql
-- SQL Server: Insert hyphens into a phone number
SELECT STUFF('4155550100', 4, 0, '-') AS phone;  -- '415-5550100'
SELECT STUFF('415-5550100', 4, 1, ' ') AS phone; -- '415 5550100'

-- MySQL: Same operation
SELECT INSERT('4155550100', 4, 0, '-') AS phone;  -- '415-5550100'
```

**Real-World Cases:**
- **Formatting:** Adding separators to ID numbers or dates.
- **Data correction:** Overwriting incorrect segments.
- **Masking:** Replacing middle digits of a credit card number.

**References:**
- Microsoft Learn: STUFF - https://learn.microsoft.com/en-us/sql/t-sql/functions/stuff-transact-sql
- MySQL: String Functions (INSERT) - https://dev.mysql.com/doc/refman/8.0/en/string-functions.html
- Oracle: INSERT Function - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/INSERT.html


## 5. Concatenation & Structure Shifts

### 5.1 Standard Joining: CONCAT(str1, str2, ...) or Pipe Operators (||, +)

**Core Definitions:**

- **Core Definition:** CONCAT joins two or more strings into one. The `||` operator (ANSI standard) and `+` operator (SQL Server) perform the same function.
- **Technical Definition:** The `||` operator is the ANSI SQL standard concatenation operator. `CONCAT(string1, string2, ...)` is a function that accepts multiple arguments. SQL Server uses `+` as the concatenation operator and also supports CONCAT (2012+). CONCAT treats NULL as an empty string in MySQL, PostgreSQL, and SQL Server; the `||` operator returns NULL if any operand is NULL (except in Oracle, where NULL is treated as empty string).
- **Beginner-Friendly Explanation:** Glues strings together. `CONCAT('Hello', ' ', 'World')` returns `'Hello World'`.

**Purposes:**
- To combine multiple columns into one (e.g., first name + last name)
- To build formatted output strings
- To generate dynamic SQL or messages

**Syntax Structures and Rules:**

```sql
CONCAT(string1, string2, ...)
string1 || string2          -- ANSI standard
string1 + string2           -- SQL Server
```

**Annotated Code Examples:**

```sql
-- CONCAT function
SELECT CONCAT('Hello', ' ', 'World') AS greeting;  -- 'Hello World'

-- ANSI pipe operator (PostgreSQL, Oracle, MySQL with PIPES_AS_CONCAT)
SELECT 'Hello' || ' ' || 'World' AS greeting;       -- 'Hello World'

-- SQL Server + operator
SELECT 'Hello' + ' ' + 'World' AS greeting;         -- 'Hello World'
```

**Real-World Cases:**
- **Reports:** Combining first and last names into a full name.
- **Addresses:** Building full addresses from components.
- **Messages:** Generating dynamic notification text.

**References:**
- PostgreSQL: String Functions (||, CONCAT) - https://www.postgresql.org/docs/current/functions-string.html
- MySQL: String Functions (CONCAT) - https://dev.mysql.com/doc/refman/8.0/en/string-functions.html
- Microsoft Learn: CONCAT - https://learn.microsoft.com/en-us/sql/t-sql/functions/concat-transact-sql


### 5.2 Delimiter Joining: CONCAT_WS()

**Core Definitions:**

- **Core Definition:** CONCAT_WS (Concatenate With Separator) joins strings with a specified separator, skipping NULL values.
- **Technical Definition:** `CONCAT_WS(separator, string1, string2, ...)` concatenates the strings using the separator between each. NULL arguments are ignored (the separator is not added for them). Available in MySQL, PostgreSQL, SQL Server 2017+, and Oracle.
- **Beginner-Friendly Explanation:** Like CONCAT, but you specify a separator (e.g., a comma) to put between the values. NULLs are skipped.

**Purposes:**
- To build delimited lists without dealing with NULLs
- To create CSV-style output
- To combine address components with proper formatting

**Syntax Structures and Rules:**

```sql
CONCAT_WS(separator, string1, string2, ...)
```

**Annotated Code Examples:**

```sql
SELECT CONCAT_WS(', ', 'Alice', NULL, 'Smith') AS full_name;  -- 'Alice, Smith'
SELECT CONCAT_WS(' - ', 'New York', 'NY', NULL) AS location;  -- 'New York - NY'
```

**Real-World Cases:**
- **Addresses:** `CONCAT_WS(', ', street, city, state, zip)`.
- **Reports:** Building comma-separated lists from multiple columns.
- **Names:** Combining title, first, middle, last names.

**References:**
- MySQL: String Functions (CONCAT_WS) - https://dev.mysql.com/doc/refman/8.0/en/string-functions.html
- PostgreSQL: String Functions (concat_ws) - https://www.postgresql.org/docs/current/functions-string.html


### 5.3 String Splitting: STRING_SPLIT() or SPLIT_PART()

**Core Definitions:**

- **Core Definition:** STRING_SPLIT divides a string into rows based on a delimiter. SPLIT_PART returns a specific segment from a delimited string.
- **Technical Definition:** `STRING_SPLIT(string, separator)` (SQL Server 2016+, PostgreSQL 14+) returns a table with one column named `value` containing each segment. `SPLIT_PART(string, delimiter, field)` (PostgreSQL, MySQL 8.0+, Redshift) returns the nth field (1-based). `SUBSTRING_INDEX(string, delimiter, count)` (MySQL) returns a substring based on delimiter occurrences.
- **Beginner-Friendly Explanation:** Splitting breaks a single string into multiple pieces. STRING_SPLIT turns `'a,b,c'` into three rows: `a`, `b`, `c`. SPLIT_PART extracts just one piece.

**Purposes:**
- To decompose delimited data into rows or columns
- To extract specific segments from structured strings
- To parse CSV-like data stored in a single column

**Syntax Structures and Rules:**

```sql
-- SQL Server, PostgreSQL 14+
STRING_SPLIT(string, separator)

-- PostgreSQL, MySQL 8.0+, Redshift
SPLIT_PART(string, delimiter, field)

-- MySQL
SUBSTRING_INDEX(string, delimiter, count)
```

**Annotated Code Examples:**

```sql
-- SQL Server: STRING_SPLIT returns a table
SELECT value FROM STRING_SPLIT('apple,banana,cherry', ',');
-- Returns 3 rows: apple, banana, cherry

-- PostgreSQL: SPLIT_PART
SELECT SPLIT_PART('2024-01-15', '-', 1) AS year,   -- '2024'
       SPLIT_PART('2024-01-15', '-', 2) AS month,  -- '01'
       SPLIT_PART('2024-01-15', '-', 3) AS day;    -- '15'
```

**Real-World Cases:**
- **ETL:** Splitting comma-separated values into rows for normalization.
- **Log parsing:** Extracting fields from delimited log entries.
- **Data migration:** Converting denormalized data into relational format.

**References:**
- Microsoft Learn: STRING_SPLIT - https://learn.microsoft.com/en-us/sql/t-sql/functions/string-split-transact-sql
- PostgreSQL: String Functions (split_part) - https://www.postgresql.org/docs/current/functions-string.html
- MySQL: String Functions (SUBSTRING_INDEX) - https://dev.mysql.com/doc/refman/8.0/en/string-functions.html


## 6. Pattern Searching & Matching

### 6.1 Basic Position Searching: POSITION(), CHARINDEX(), or LOCATE()

**Core Definitions:**

- **Core Definition:** Returns the starting position of a substring within a string.
- **Technical Definition:** `POSITION(substring IN string)` is the ANSI SQL standard. `CHARINDEX(substring, string)` is SQL Server's implementation. `LOCATE(substring, string)` is MySQL's implementation. `INSTR(string, substring)` is Oracle/MySQL's implementation. All return 0 if the substring is not found.
- **Beginner-Friendly Explanation:** Tells you where a substring starts. `POSITION('World' IN 'Hello World')` returns 7.

**Purposes:**
- To find the location of a substring
- To validate that a pattern exists
- To support further string manipulation (e.g., extracting based on position)

**Syntax Structures and Rules:**

```sql
-- ANSI standard
POSITION(substring IN string)

-- SQL Server
CHARINDEX(substring, string [, start])

-- MySQL
LOCATE(substring, string [, start])

-- Oracle, MySQL
INSTR(string, substring [, start] [, occurrence])
```

**Annotated Code Examples:**

```sql
-- Standard SQL
SELECT POSITION('World' IN 'Hello World') AS pos;  -- 7

-- SQL Server
SELECT CHARINDEX('World', 'Hello World') AS pos;    -- 7

-- MySQL
SELECT LOCATE('World', 'Hello World') AS pos;       -- 7

-- Oracle
SELECT INSTR('Hello World', 'World') AS pos FROM dual;  -- 7
```

**Real-World Cases:**
- **Validation:** Checking if an email contains `'@'`.
- **Parsing:** Finding delimiters for further extraction.
- **Search:** Implementing custom search functionality.

**References:**
- PostgreSQL: String Functions (position, strpos) - https://www.postgresql.org/docs/current/functions-string.html
- Microsoft Learn: CHARINDEX - https://learn.microsoft.com/en-us/sql/t-sql/functions/charindex-transact-sql
- MySQL: String Functions (LOCATE, INSTR) - https://dev.mysql.com/doc/refman/8.0/en/string-functions.html


### 6.2 Boolean Checks: STARTS_WITH(str, prefix) or ENDS_WITH()

**Core Definitions:**

- **Core Definition:** STARTS_WITH returns true if a string begins with a specified prefix. ENDS_WITH returns true if it ends with a specified suffix.
- **Technical Definition:** `STARTS_WITH(string, prefix)` is available in PostgreSQL, MySQL, and some other databases. `ENDS_WITH` is less common but available in PostgreSQL 11+ and MySQL. SQL Server does not have these functions; use `LEFT(string, LEN(prefix)) = prefix` or `LIKE 'prefix%'` instead.
- **Beginner-Friendly Explanation:** Quick yes/no check: does this text start with / end with something?

**Purposes:**
- To filter rows based on prefix or suffix
- To validate formats (e.g., file extensions)
- To simplify queries that would otherwise use LIKE

**Syntax Structures and Rules:**

```sql
-- PostgreSQL, MySQL
STARTS_WITH(string, prefix)
ENDS_WITH(string, suffix)

-- SQL Server workaround
LEFT(string, LEN(prefix)) = prefix   -- Starts with
RIGHT(string, LEN(suffix)) = suffix  -- Ends with
LIKE 'prefix%'                       -- Starts with (simpler)
```

**Annotated Code Examples:**

```sql
-- PostgreSQL, MySQL
SELECT STARTS_WITH('Hello World', 'Hello') AS starts,  -- TRUE
       ENDS_WITH('Hello World', 'World') AS ends;      -- TRUE

-- SQL Server workaround
SELECT CASE WHEN LEFT('Hello World', 5) = 'Hello' THEN 1 ELSE 0 END AS starts,
       CASE WHEN RIGHT('Hello World', 5) = 'World' THEN 1 ELSE 0 END AS ends;
```

**Real-World Cases:**
- **File filtering:** Finding all files with a specific extension.
- **Code validation:** Checking if product codes start with a specific prefix.
- **Log analysis:** Filtering log entries by level prefix (e.g., `'ERROR'`).

**References:**
- PostgreSQL: String Functions (starts_with) - https://www.postgresql.org/docs/current/functions-string.html
- MySQL: String Functions (STARTS_WITH) - https://dev.mysql.com/doc/refman/8.0/en/string-functions.html


### 6.3 Advanced Expression Matching: PATINDEX() or Regex Integrations

**Core Definitions:**

- **Core Definition:** PATINDEX returns the starting position of a pattern in a string using SQL Server's pattern syntax. Regex functions (REGEXP_LIKE, REGEXP_REPLACE, etc.) provide full regular expression matching.
- **Technical Definition:** `PATINDEX('%pattern%', string)` returns the starting position of the first occurrence of a pattern, or 0 if not found. SQL Server 2025 introduced native REGEXP functions: `REGEXP_LIKE`, `REGEXP_COUNT`, `REGEXP_INSTR`, `REGEXP_REPLACE`, `REGEXP_SUBSTR`, `REGEXP_MATCHES`, and `REGEXP_SPLIT_TO_TABLE`. Oracle and MySQL have long supported REGEXP_LIKE, REGEXP_REPLACE, REGEXP_SUBSTR, and REGEXP_INSTR. PostgreSQL uses `~`, `~*`, and `regexp_*` functions.
- **Beginner-Friendly Explanation:** These are the "power tools" for pattern matching. PATINDEX finds patterns using wildcards; REGEXP functions use full regular expressions for complex matching.

**Purposes:**
- To find complex patterns in text
- To validate formats (email, phone, SSN)
- To extract and replace text based on regex patterns

**Syntax Structures and Rules:**

```sql
-- SQL Server: PATINDEX
PATINDEX('%pattern%', string)

-- SQL Server 2025, Oracle, MySQL: REGEXP functions
REGEXP_LIKE(string, pattern [, flags])
REGEXP_REPLACE(string, pattern, replacement)
REGEXP_SUBSTR(string, pattern)
REGEXP_INSTR(string, pattern)

-- PostgreSQL: POSIX regex operators
string ~ pattern        -- Match (case-sensitive)
string ~* pattern       -- Match (case-insensitive)
regexp_replace(string, pattern, replacement)
```

**Annotated Code Examples:**

```sql
-- SQL Server: PATINDEX - find position of first digit
SELECT PATINDEX('%[0-9]%', 'abc123') AS pos;  -- 4

-- Oracle/MySQL: REGEXP_LIKE - find rows matching a pattern
SELECT * FROM users WHERE REGEXP_LIKE(email, '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$');

-- MySQL: REGEXP_REPLACE
SELECT REGEXP_REPLACE('Phone: 415-555-0100', '[0-9]', '#') AS masked;
-- 'Phone: ###-###-####'

-- PostgreSQL: regexp_replace
SELECT regexp_replace('Phone: 415-555-0100', '[0-9]', '#', 'g') AS masked;
-- 'Phone: ###-###-####'
```

**Expected Output:**

| Function | Input | Output |
|----------|-------|--------|
| PATINDEX('%[0-9]%', 'abc123') | | 4 |
| REGEXP_REPLACE('Phone: 415-555-0100', '[0-9]', '#') | | 'Phone: ###-###-####' |

**Real-World Cases:**
- **Data validation:** Checking email format with regex.
- **Data masking:** Replacing digits in phone numbers or credit cards.
- **Log analysis:** Extracting error codes or timestamps from log lines.
- **Text processing:** Finding and replacing complex patterns.

**References:**
- Microsoft Learn: PATINDEX - https://learn.microsoft.com/en-us/sql/t-sql/functions/patindex-transact-sql
- Redgate: SQL Server 2025 Regex Functions - https://www.red-gate.com/simple-talk/featured/using-regex-in-sql-server-2025-complete-guide/
- Oracle: REGEXP_LIKE - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/REGEXP_LIKE.html
- MySQL: Regular Expressions - https://dev.mysql.com/doc/refman/8.0/en/regexp.html
- PostgreSQL: POSIX Regular Expressions - https://www.postgresql.org/docs/current/functions-matching.html


## Summary Tables

### String Functions by Category

| Category | Function | ANSI | PostgreSQL | MySQL | SQL Server | Oracle |
|----------|----------|------|-----------|-------|------------|--------|
| **Length** | LENGTH | ✓ | ✓ | ✓ (bytes) | LEN | ✓ |
| **Length** | CHARACTER_LENGTH | ✓ | ✓ | ✓ | ✗ | ✓ |
| **Byte Length** | OCTET_LENGTH | ✓ | ✓ | LENGTH | DATALENGTH | ✓ |
| **Case** | UPPER | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Case** | LOWER | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Case** | INITCAP | ✗ | ✓ | ✗ | ✗ | ✓ |
| **Reverse** | REVERSE | ✗ | ✓ | ✓ | ✓ | ✓ |
| **Substring** | SUBSTRING | ✓ | ✓ | ✓ | ✓ | SUBSTR |
| **Slice** | LEFT/RIGHT | ✗ | ✓ | ✓ | ✓ | ✓ |
| **Trim** | TRIM | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Pad** | LPAD/RPAD | ✗ | ✓ | ✓ | ✗ | ✓ |
| **Replace** | REPLACE | ✗ | ✓ | ✓ | ✓ | ✓ |
| **Stuff** | STUFF/INSERT | ✗ | ✗ | INSERT | STUFF | INSERT |
| **Concat** | CONCAT | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Concat** | CONCAT_WS | ✗ | ✓ | ✓ | ✓ (2017+) | ✓ |
| **Split** | STRING_SPLIT | ✗ | ✓ (14+) | ✗ | ✓ (2016+) | ✗ |
| **Split** | SPLIT_PART | ✗ | ✓ | ✓ (8.0+) | ✗ | ✗ |
| **Position** | POSITION | ✓ | ✓ | LOCATE | CHARINDEX | INSTR |
| **Boolean** | STARTS_WITH | ✗ | ✓ | ✓ | ✗ | ✗ |
| **Pattern** | PATINDEX | ✗ | ✗ | ✗ | ✓ | ✗ |
| **Regex** | REGEXP_LIKE | ✗ | ~ / ~* | ✓ | ✓ (2025+) | ✓ |

### Concatenation NULL Behavior

| Database | `\|\|` with NULL | CONCAT with NULL | CONCAT_WS with NULL |
|----------|-----------------|------------------|---------------------|
| PostgreSQL | Returns NULL | Ignores NULL | Ignores NULL |
| MySQL | Returns NULL | Ignores NULL | Ignores NULL |
| SQL Server | N/A | Ignores NULL | Ignores NULL |
| Oracle | Treats NULL as '' | Treats NULL as '' | Ignores NULL |

---

## References

- PostgreSQL: String Functions and Operators - https://www.postgresql.org/docs/current/functions-string.html
- MySQL: String Functions and Operators - https://dev.mysql.com/doc/refman/8.0/en/string-functions.html
- Microsoft Learn: String Functions (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/string-functions-transact-sql
- Oracle: SQL Reference (String Functions) - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/
- SQL Standard: ANSI/ISO SQL String Functions - https://sqlite.org/forum/forumpost/de86d9d98820efff?raw
- Redgate: Regex in SQL Server 2025 - https://www.red-gate.com/simple-talk/featured/using-regex-in-sql-server-2025-complete-guide/
- Alibaba Cloud: String Functions Reference - https://www.alibabacloud.com/help/en/hologres/developer-reference/string-functions
- Oracle: INSERT Function - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/INSERT.html
- Oracle: REGEXP_LIKE - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/REGEXP_LIKE.html
- Microsoft Learn: PATINDEX - https://learn.microsoft.com/en-us/sql/t-sql/functions/patindex-transact-sql