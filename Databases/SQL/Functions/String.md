# SQL String Functions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
SQL string functions are built-in routines provided by relational database management systems (RDBMS) that accept one or more string (character) values as input and return a string or numeric result, enabling systematic manipulation, inspection, and transformation of textual data directly within SQL statements.

**Technical Definition**
In SQL, string functions are scalar or aggregate operators whose domain and range are values of character string types (`CHAR`, `VARCHAR`, `TEXT`, `NVARCHAR`, etc.) or whose output is derived from the byte or character sequence of such values. They are defined by SQL standards (SQL:1999, SQL:2003, SQL:2008, SQL:2016) and implemented with vendor-specific extensions in PostgreSQL, MySQL, SQL Server, Oracle, and others. String functions operate under the collation rules of the database, which govern case conversion, comparison, and sorting behavior.

**Beginner-Friendly Explanation**
Think of string functions as a toolbox for working with text in a database. Just as a word processor lets you change text to uppercase, remove spaces, find and replace words, or extract a portion of a sentence, SQL string functions let you do the same things to data stored in database columns—without pulling the data out into another program. You can clean, reshape, and analyse text directly inside your SQL queries.

---

### Key Characteristics

1. **Declarative and Set-Oriented**: String functions are applied within SQL statements (primarily `SELECT`, `WHERE`, `ORDER BY`, and `GROUP BY`) and operate on sets of rows, not just individual values.
2. **Standardized but Vendor-Extended**: Core functions such as `UPPER`, `LOWER`, `TRIM`, `SUBSTRING`, and `POSITION` are defined by SQL standards, but each RDBMS adds proprietary functions (e.g., PostgreSQL’s `SPLIT_PART`, MySQL’s `GROUP_CONCAT`, SQL Server’s `CHARINDEX`).
3. **Collation-Sensitive**: Functions like `UPPER`, `LOWER`, `INITCAP`, and comparison operators depend on the database’s collation, which determines locale-specific case mapping and sort order.
4. **Character vs. Byte Semantics**: Functions such as `LENGTH` may return byte length in some systems (MySQL) and character length in others (PostgreSQL), while `CHAR_LENGTH` consistently returns character count.
5. **NULL Propagation**: Most string functions return `NULL` when any required argument is `NULL`, unless explicitly documented otherwise (e.g., `CONCAT` in PostgreSQL ignores `NULL` arguments).
6. **Index and Performance Implications**: Applying functions to columns in `WHERE` or `JOIN` clauses often prevents index usage (non-sargable expressions), a critical consideration for query optimisation.

---

### Prerequisites

Before working with SQL string functions, you should understand:

- **Basic SQL syntax**: `SELECT`, `FROM`, `WHERE`, `GROUP BY`, `ORDER BY`.
- **Data types**: `CHAR`, `VARCHAR`, `TEXT`, and the distinction between fixed-length and variable-length strings.
- **NULL semantics**: Three-valued logic and how `NULL` propagates through expressions.
- **Character encoding**: UTF-8, UTF-16, and how byte length differs from character length.
- **Collations**: How the database’s locale settings affect case conversion and comparison.

---

### Related Programming Areas

| Area | Connection |
|------|------------|
| **Data Cleaning / ETL** | String functions are used to trim whitespace, standardise case, and parse delimited fields. |
| **Full-Text Search** | Functions like `POSITION`, `STRPOS`, and regular-expression functions support pattern matching. |
| **Data Validation** | `LENGTH`, `LEFT`, `RIGHT`, and `SUBSTRING` enforce format constraints. |
| **Reporting & Analytics** | String aggregation (`STRING_AGG`, `GROUP_CONCAT`) produces comma-separated lists for dashboards. |
| **Application Development** | ORMs and query builders generate SQL string expressions; understanding them aids debugging and optimisation. |
| **Database Administration** | Collation and normalisation affect index behaviour, uniqueness constraints, and query performance. |

---

### Core Concepts / Features

The cheat sheet is organised into the following conceptual clusters, each containing one or more features:

1. **Length & Measurement** — `LENGTH`, `CHAR_LENGTH`
2. **Case Conversion** — `UPPER`, `LOWER`, `INITCAP`
3. **Trimming & Padding** — `TRIM`, `LTRIM`, `RTRIM`, `LPAD`, `RPAD`
4. **Extraction & Substrings** — `SUBSTRING`, `LEFT`, `RIGHT`, `SPLIT_PART`
5. **Searching & Position** — `POSITION`, `STRPOS`
6. **Replacement & Translation** — `REPLACE`, `TRANSLATE`
7. **Concatenation & Aggregation** — `CONCAT`, `CONCAT_WS`, String Aggregation (`STRING_AGG` / `GROUP_CONCAT`)
8. **Regular Expressions** — `REGEXP_REPLACE`, `REGEXP_MATCHES`, `REGEXP_SPLIT_TO_TABLE`
9. **Comparison & Standards** — String Normalisation, String Comparison, Collations

Each feature is documented with definitions, purposes, syntax rules, constraints, annotated code examples, expected outputs, and real-world cases.

---

## 1. Length & Measurement

### 1.1 `LENGTH`

#### Definitions

**Core Definition**
`LENGTH` returns the length of a string, measured in a database-defined unit (characters or bytes).

**Technical Definition**
`LENGTH(string)` is a scalar function that returns an integer representing the number of characters (in PostgreSQL) or bytes (in MySQL) in the input string. In SQL Server, the equivalent function is `LEN`, which returns the number of characters excluding trailing spaces. The SQL standard does not define `LENGTH` with a uniform unit across vendors; `CHAR_LENGTH` is the standard-compliant function for character count.

**Beginner-Friendly Explanation**
`LENGTH` tells you “how long” a piece of text is. In some databases it counts letters, in others it counts bytes (storage units). If you have a word like `'josé'`, the number of letters is 4, but the number of bytes may be 5 in UTF-8 because `é` takes two bytes.

#### Purposes

- To determine the number of characters in a string for validation (e.g., minimum password length).
- To measure byte length for storage estimation or truncation logic.
- To filter rows based on textual length in `WHERE` clauses.
- To support dynamic string padding or truncation decisions.

#### Syntax Rules and Structure

**Complete General Syntax (PostgreSQL)**
```
LENGTH ( text ) → integer
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `LENGTH` | Function name. |
| `text` | A value of type `text`, `varchar`, or `char`. |
| `→ integer` | Returns the number of characters in the string. |

**MySQL Syntax**
```
LENGTH ( str ) → integer
```
Returns the length of `str` in **bytes**; multibyte-safe in the sense that it counts actual bytes, not characters.

**SQL Server Equivalent**
```
LEN ( string_expression ) → integer
```
Returns the number of characters in the specified string expression, **excluding trailing spaces** (unlike `DATALENGTH`, which includes them).

**Syntax Rules**

1. The argument must be a string expression; numeric arguments are implicitly cast in some systems but this is implementation-dependent.
2. `LENGTH(NULL)` returns `NULL`.
3. In PostgreSQL, `LENGTH` counts characters; in MySQL, it counts bytes. Use `CHAR_LENGTH` for character count in MySQL.

**Constraints and Limitations**

- Not portable across RDBMS without adjustment (byte vs. character semantics).
- In SQL Server, `LEN` ignores trailing spaces; use `DATALENGTH` for byte-level length.
- Applying `LENGTH` to a column in `WHERE` may prevent index usage.

#### Annotated Code Examples

**Example 1: Character Length in PostgreSQL**

```sql
-- Setup: no table required; use a literal string
SELECT
  LENGTH('josé') AS char_length_utf8,   -- counts characters
  LENGTH('hello') AS char_length_ascii;  -- counts characters
```

**Step-by-Step Explanation**

1. `LENGTH('josé')` counts each character, including the accented `é` as one character → **4**.
2. `LENGTH('hello')` counts five ASCII characters → **5**.

**Expected Output**

| char_length_utf8 | char_length_ascii |
|------------------|-------------------|
| 4                | 5                 |

**Why This Output Occurs**: PostgreSQL’s `LENGTH` counts characters, not bytes. Even though `é` requires two bytes in UTF-8, it is one character.

---

**Example 2: Byte Length vs. Character Length in MySQL**

```sql
-- Setup: none
SELECT
  LENGTH('josé')    AS byte_length,   -- bytes
  CHAR_LENGTH('josé') AS char_length; -- characters
```

**Expected Output (MySQL, UTF-8)**

| byte_length | char_length |
|-------------|-------------|
| 5           | 4           |

**Why This Output Occurs**: MySQL’s `LENGTH` returns byte count; `é` is two bytes in UTF-8, so `'josé'` is 5 bytes. `CHAR_LENGTH` returns the number of characters, which is 4.

---

**Example 3: SQL Server `LEN` Ignoring Trailing Spaces**

```sql
-- Setup: none
SELECT
  LEN('hello   ') AS len_result,       -- trailing spaces excluded
  DATALENGTH('hello   ') AS datalength_result; -- bytes including spaces
```

**Expected Output (SQL Server)**

| len_result | datalength_result |
|------------|-------------------|
| 5          | 8                 |

**Why This Output Occurs**: `LEN` excludes trailing spaces, while `DATALENGTH` counts all bytes, including spaces.

#### Real-World Cases

- **Password policy enforcement**: `SELECT username FROM users WHERE LENGTH(password_hash) < 60;` identifies potentially weak hash storage.
- **Data quality checks**: `SELECT * FROM products WHERE LENGTH(product_code) <> 10;` finds product codes of unexpected length.
- **Storage estimation**: In MySQL, `SUM(LENGTH(description))` estimates total byte storage for a text column.

---

### 1.2 `CHAR_LENGTH`

#### Definitions

**Core Definition**
`CHAR_LENGTH` returns the number of characters in a string.

**Technical Definition**
`CHAR_LENGTH(string)` is an SQL-standard scalar function that returns the number of characters (code points) in the input string. It is synonymous with `CHARACTER_LENGTH`. Unlike `LENGTH` in MySQL, `CHAR_LENGTH` is character-based and therefore multibyte-safe.

**Beginner-Friendly Explanation**
`CHAR_LENGTH` answers “how many letters are in this word?” regardless of how many bytes each letter takes up. It is the safe choice when you care about human-perceived length.

#### Purposes

- To count characters consistently across single-byte and multibyte encodings.
- To enforce character-based length constraints (e.g., “username must be 3–20 characters”).
- To compare character counts rather than byte counts in internationalised applications.

#### Syntax Rules and Structure

**General Syntax**
```
CHAR_LENGTH ( string ) → integer
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `CHAR_LENGTH` | Standard SQL function name. |
| `string` | Character string expression. |
| `→ integer` | Number of characters. |

**Syntax Rules**

1. `CHAR_LENGTH(NULL)` returns `NULL`.
2. `CHARACTER_LENGTH` is a synonym in PostgreSQL and MySQL.
3. In SQL Server, use `LEN` for character count (with the caveat about trailing spaces).

**Constraints and Limitations**

- Some systems may treat combining characters or grapheme clusters differently; `CHAR_LENGTH` counts code points, not user-perceived graphemes.
- Not all systems implement `CHARACTER_LENGTH` as a synonym (though most do).

#### Annotated Code Examples

**Example 1: Character Count Across Encodings**

```sql
-- PostgreSQL or MySQL
SELECT
  CHAR_LENGTH('café')   AS latin_char_len,
  CHAR_LENGTH('日本語') AS cjk_char_len;
```

**Expected Output**

| latin_char_len | cjk_char_len |
|----------------|--------------|
| 4              | 3            |

**Why This Output Occurs**: `café` has 4 characters; `日本語` has 3 characters, even though each CJK character may occupy 3 bytes in UTF-8.

**Example 2: Using `CHARACTER_LENGTH` Synonym**

```sql
-- PostgreSQL
SELECT CHARACTER_LENGTH('hello') AS char_len;
```

**Expected Output**

| char_len |
|----------|
| 5        |

#### Real-World Cases

- **Internationalised form validation**: Ensuring a name field contains no more than 50 characters, regardless of script.
- **Display truncation**: Determining how many characters to show in a UI preview.
- **Data migration**: Comparing character counts between source and target systems with different encodings.

---

## 2. Case Conversion

### 2.1 `UPPER`

#### Definitions

**Core Definition**
`UPPER` converts all characters in a string to uppercase.

**Technical Definition**
`UPPER(text)` returns a copy of the input string with all lowercase characters replaced by their uppercase equivalents, according to the rules of the database’s collation and locale.

**Beginner-Friendly Explanation**
`UPPER` shouts your text—it turns “hello” into “HELLO”.

#### Purposes

- To normalise data for case-insensitive comparison (e.g., storing country codes in uppercase).
- To produce formatted output for reports and labels.
- To enforce consistency in user input before storage.
- To enable case-insensitive searching in systems without native case-insensitive collations.

#### Syntax Rules and Structure

**General Syntax**
```
UPPER ( text ) → text
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `UPPER` | Function name. |
| `text` | String expression. |
| `→ text` | Uppercased string. |

**Syntax Rules**

1. `UPPER(NULL)` returns `NULL`.
2. Case mapping is collation-dependent; for example, the German `ß` may or may not map to `SS` depending on locale.
3. `UCASE` is a synonym in MySQL.

**Constraints and Limitations**

- Locale-dependent behaviour: case conversion rules vary by language (e.g., Turkish dotless `i`).
- Applying `UPPER` to a column in `WHERE` can prevent index usage.

#### Annotated Code Examples

**Example 1: Basic Uppercasing**

```sql
SELECT UPPER('hello world') AS upper_result;
```

**Expected Output**

| upper_result |
|--------------|
| HELLO WORLD  |

**Example 2: Case-Insensitive Search Using `UPPER`**

```sql
-- Assume a table: users(username)
-- Goal: find 'alice' regardless of case
SELECT username
FROM users
WHERE UPPER(username) = UPPER('alice');
```

**Step-by-Step Explanation**

1. `UPPER('alice')` → `'ALICE'`.
2. `UPPER(username)` converts every stored username to uppercase.
3. The `WHERE` clause matches rows where the uppercased username equals `'ALICE'`.

**Expected Output (illustrative)**

| username |
|----------|
| Alice    |
| ALICE    |
| alice    |

**Why This Output Occurs**: Uppercasing both sides makes the comparison case-insensitive.

#### Real-World Cases

- **Country/state codes**: Storing `'us'`, `'Us'`, `'US'` consistently as `'US'`.
- **Report headers**: Displaying column labels in uppercase.
- **Search normalisation**: Pre-uppercasing search terms in an application before querying.

---

### 2.2 `LOWER`

#### Definitions

**Core Definition**
`LOWER` converts all characters in a string to lowercase.

**Technical Definition**
`LOWER(text)` returns a copy of the input string with all uppercase characters replaced by their lowercase equivalents, according to the database’s collation and locale.

**Beginner-Friendly Explanation**
`LOWER` whispers your text—it turns “HELLO” into “hello”.

#### Purposes

- To normalise data for case-insensitive comparison (e.g., email addresses).
- To produce lowercase output for URLs or slugs.
- To standardise user input before storage.
- To facilitate case-insensitive matching in `WHERE` and `JOIN` clauses.

#### Syntax Rules and Structure

**General Syntax**
```
LOWER ( text ) → text
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `LOWER` | Function name. |
| `text` | String expression. |
| `→ text` | Lowercased string. |

**Syntax Rules**

1. `LOWER(NULL)` returns `NULL`.
2. Case mapping is locale-sensitive (e.g., Turkish `I` → `ı`).
3. `LCASE` is a synonym in MySQL.

**Constraints and Limitations**

- Same portability and performance caveats as `UPPER`.

#### Annotated Code Examples

**Example 1: Lowercasing an Email Address**

```sql
SELECT LOWER('Alice@Example.COM') AS normalised_email;
```

**Expected Output**

| normalised_email |
|------------------|
| alice@example.com |

**Example 2: Case-Insensitive Join**

```sql
SELECT o.order_id
FROM orders o
JOIN customers c
  ON LOWER(o.customer_email) = LOWER(c.email);
```

**Why This Output Occurs**: `LOWER` on both sides of the join condition ensures that `Alice@Example.com` matches `alice@example.com`.

#### Real-World Cases

- **Email storage**: Normalising email addresses to lowercase before uniqueness checks.
- **Slug generation**: Converting titles to lowercase for URL slugs.
- **Username matching**: Allowing users to log in with any capitalisation of their username.

---

### 2.3 `INITCAP`

#### Definitions

**Core Definition**
`INITCAP` converts the first letter of each word to uppercase and the remaining letters to lowercase.

**Technical Definition**
`INITCAP(text)` returns a string where each word (defined as a sequence of alphanumeric characters separated by non-alphanumeric characters) has its first character uppercased and all subsequent characters lowercased.

**Beginner-Friendly Explanation**
`INITCAP` makes text look like a title—every word starts with a capital letter and the rest is lowercase.

#### Purposes

- To format names and titles for display (e.g., `'john o''brien'` → `'John O''Brien'`).
- To normalise inconsistent capitalisation in user-entered data.
- To generate human-readable labels from database identifiers.
- To standardise address fields (city, street names).

#### Syntax Rules and Structure

**General Syntax (PostgreSQL)**
```
INITCAP ( text ) → text
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `INITCAP` | Function name. |
| `text` | String expression. |
| `→ text` | Title-cased string. |

**Syntax Rules**

1. Word boundaries are defined by non-alphanumeric characters.
2. `INITCAP(NULL)` returns `NULL`.
3. Not available in MySQL or SQL Server natively; must be emulated with `CONCAT`/`UPPER`/`LOWER` or application logic.

**Constraints and Limitations**

- **Vendor-specific**: Available in PostgreSQL and Oracle; not standard SQL.
- Apostrophes and hyphens are treated as word separators, which may or may not be desired (e.g., `O'Brien` → `O'Brien` in PostgreSQL because the apostrophe separates words).
- Does not handle all Unicode title-casing rules (e.g., digraphs).

#### Annotated Code Examples

**Example 1: Formatting a Name**

```sql
-- PostgreSQL
SELECT INITCAP('hi THOMAS') AS formatted_name;
```

**Expected Output**

| formatted_name |
|----------------|
| Hi Thomas      |

**Why This Output Occurs**: Each word (`hi`, `THOMAS`) is converted: first letter uppercase, rest lowercase → `Hi`, `Thomas`.

**Example 2: Handling Apostrophes**

```sql
SELECT INITCAP('john o''brien') AS name_with_apostrophe;
```

**Expected Output (PostgreSQL)**

| name_with_apostrophe |
|----------------------|
| John O'Brien         |

**Why This Output Occurs**: The apostrophe is a non-alphanumeric character, so `o` and `brien` are treated as separate words; `o` becomes `O` and `brien` becomes `Brien`.

#### Real-World Cases

- **Customer name formatting**: Converting `'MARY SMITH'` to `'Mary Smith'` for mail merges.
- **Address standardisation**: `'123 main st'` → `'123 Main St'`.
- **Report labels**: Generating title-cased column headers from snake_case names.

---

## 3. Trimming & Padding

### 3.1 `TRIM`

#### Definitions

**Core Definition**
`TRIM` removes specified characters (or spaces by default) from the beginning, end, or both ends of a string.

**Technical Definition**
`TRIM([LEADING|TRAILING|BOTH] [characters] FROM string)` returns the input string with the longest leading and/or trailing substring containing only the specified characters removed. The default is to trim spaces from both ends.

**Beginner-Friendly Explanation**
`TRIM` cleans up extra spaces (or other characters) from the edges of a piece of text—like trimming the crust off a sandwich.

#### Purposes

- To clean user input by removing accidental leading/trailing spaces.
- To normalise data before storage or comparison.
- To remove specific delimiter characters from both ends of a string.
- To prepare strings for concatenation or parsing.

#### Syntax Rules and Structure

**SQL-Standard Syntax**
```
TRIM ( [ LEADING | TRAILING | BOTH ] [ characters ] FROM string )
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `TRIM` | Function name. |
| `LEADING` | Remove from the start only. |
| `TRAILING` | Remove from the end only. |
| `BOTH` | Remove from both ends (default). |
| `characters` | Optional set of characters to remove (default: space). |
| `string` | Input string. |

**Alternative Non-Standard Syntax (PostgreSQL)**
```
TRIM ( [ LEADING | TRAILING | BOTH ] [ FROM ] string [ , characters ] )
```

**Syntax Rules**

1. If `characters` is omitted, spaces are removed.
2. `TRIM(NULL)` returns `NULL`.
3. Only characters in the specified set are removed; the function stops at the first character not in that set.

**Constraints and Limitations**

- Removes only leading/trailing occurrences, not internal ones.
- Behaviour with multibyte characters depends on character set.
- Some systems (e.g., MySQL) support `TRIM` with `LEADING`, `TRAILING`, and `BOTH` but may have slight syntax differences.

#### Annotated Code Examples

**Example 1: Basic Space Trimming**

```sql
SELECT TRIM('   hello world   ') AS trimmed;
```

**Expected Output**

| trimmed     |
|-------------|
| hello world |

**Why This Output Occurs**: `TRIM` removes all leading and trailing spaces, leaving the internal space intact.

**Example 2: Trimming Specific Characters**

```sql
SELECT TRIM(BOTH 'xyz' FROM 'yxTomxx') AS trimmed_specific;
```

**Expected Output**

| trimmed_specific |
|------------------|
| Tom              |

**Why This Output Occurs**: The function removes the longest substring containing only `x`, `y`, or `z` from both ends: `yx` from the left and `xx` from the right → `Tom`.

**Example 3: Leading-Only Trim**

```sql
SELECT TRIM(LEADING '0' FROM '000123') AS leading_trimmed;
```

**Expected Output**

| leading_trimmed |
|-----------------|
| 123             |

#### Real-World Cases

- **Form input sanitisation**: Removing spaces from user-entered names and emails.
- **CSV parsing**: Trimming quotes or delimiters from parsed fields.
- **Data migration**: Normalising legacy data with inconsistent spacing.

---

### 3.2 `LTRIM`

#### Definitions

**Core Definition**
`LTRIM` removes specified characters (spaces by default) from the left side of a string.

**Technical Definition**
`LTRIM(string [, characters])` returns the input string with the longest leading substring containing only characters from the `characters` set removed. If `characters` is omitted, spaces are removed.

**Beginner-Friendly Explanation**
`LTRIM` is like `TRIM` but only cleans the **left** side of the text.

#### Purposes

- To remove leading spaces from fixed-width or padded strings.
- To strip leading zeros or delimiters from numeric strings.
- To prepare strings for parsing where only the left side needs cleaning.

#### Syntax Rules and Structure

**PostgreSQL Syntax**
```
LTRIM ( string text [, characters text ] ) → text
```
**MySQL Syntax**
```
LTRIM ( str )
```
(MySQL `LTRIM` removes spaces only; for custom characters, use `TRIM(LEADING ...)`.)

**Syntax Rules**

1. Default `characters` is a space.
2. `LTRIM(NULL)` returns `NULL`.
3. Only leading characters are removed; trailing characters are untouched.

**Constraints and Limitations**

- MySQL’s `LTRIM` does not accept a custom character set (only spaces).
- Not standard SQL; vendor-specific behaviour.

#### Annotated Code Examples

**Example 1: Removing Leading Spaces**

```sql
SELECT LTRIM('   hello') AS ltrimmed;
```

**Expected Output**

| ltrimmed |
|----------|
| hello    |

**Example 2: Removing Leading Zeros (PostgreSQL)**

```sql
SELECT LTRIM('000123', '0') AS stripped_zeros;
```

**Expected Output**

| stripped_zeros |
|----------------|
| 123            |

#### Real-World Cases

- **Fixed-width file parsing**: Removing leading padding from fixed-length fields.
- **Numeric string cleanup**: Stripping leading zeros before casting to integer.

---

### 3.3 `RTRIM`

#### Definitions

**Core Definition**
`RTRIM` removes specified characters (spaces by default) from the right side of a string.

**Technical Definition**
`RTRIM(string [, characters])` returns the input string with the longest trailing substring containing only characters from the `characters` set removed. If `characters` is omitted, spaces are removed.

**Beginner-Friendly Explanation**
`RTRIM` is the mirror image of `LTRIM`—it cleans the **right** side of the text.

#### Purposes

- To remove trailing spaces from fixed-width strings.
- To strip trailing delimiters or punctuation.
- To prepare strings for concatenation where trailing whitespace is undesirable.

#### Syntax Rules and Structure

**PostgreSQL Syntax**
```
RTRIM ( string text [, characters text ] ) → text
```
**MySQL Syntax**
```
RTRIM ( str )
```

**Syntax Rules**

1. Default `characters` is a space.
2. `RTRIM(NULL)` returns `NULL`.
3. Only trailing characters are removed.

**Constraints and Limitations**

- Same vendor-specific limitations as `LTRIM` (MySQL only trims spaces).

#### Annotated Code Examples

**Example 1: Removing Trailing Spaces**

```sql
SELECT RTRIM('hello   ') AS rtrimmed;
```

**Expected Output**

| rtrimmed |
|----------|
| hello    |

**Example 2: Removing Trailing Commas**

```sql
-- PostgreSQL
SELECT RTRIM('item1,item2,', ',') AS cleaned_list;
```

**Expected Output**

| cleaned_list |
|--------------|
| item1,item2  |

#### Real-World Cases

- **CSV line cleaning**: Removing trailing commas from exported data.
- **Fixed-width report generation**: Ensuring no trailing spaces in output fields.

---

### 3.4 `LPAD`

#### Definitions

**Core Definition**
`LPAD` left-pads a string to a specified length by prepending a fill string.

**Technical Definition**
`LPAD(string, length [, fill])` returns a string of exactly `length` characters (or bytes, depending on RDBMS) obtained by prepending `fill` characters to `string`. If `string` is already longer than `length`, it is truncated on the right.

**Beginner-Friendly Explanation**
`LPAD` adds characters to the **left** of your text until it reaches a certain length—like adding leading zeros to a number.

#### Purposes

- To format numeric strings with leading zeros (e.g., `'7'` → `'007'`).
- To align text in fixed-width reports.
- To create padded codes for display or storage.
- To truncate strings to a maximum length while ensuring a minimum width.

#### Syntax Rules and Structure

**PostgreSQL Syntax**
```
LPAD ( string text, length integer [, fill text ] ) → text
```
**MySQL Syntax**
```
LPAD ( str, len, padstr )
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `string` / `str` | Input string. |
| `length` / `len` | Desired total length (characters). |
| `fill` / `padstr` | String to prepend (default: space in PostgreSQL; required in MySQL). |

**Syntax Rules**

1. If `string` is longer than `length`, it is truncated to `length` characters from the left.
2. If `fill` is omitted in PostgreSQL, spaces are used.
3. `LPAD(NULL, length, fill)` returns `NULL`.

**Constraints and Limitations**

- In MySQL, `LPAD` returns `NULL` if `len` is less than the length of `str` (it truncates, not returns `NULL`; check version-specific behaviour).
- Padding with multibyte characters may affect byte length differently from character length.

#### Annotated Code Examples

**Example 1: Zero-Padding a Number**

```sql
SELECT LPAD('7', 3, '0') AS padded;
```

**Expected Output**

| padded |
|--------|
| 007    |

**Why This Output Occurs**: The string `'7'` is length 1; to reach length 3, two `'0'` characters are prepended → `'007'`.

**Example 2: Left-Padding with a Custom String**

```sql
SELECT LPAD('hi', 5, 'xy') AS padded_custom;
```

**Expected Output**

| padded_custom |
|---------------|
| xyxhi         |

**Why This Output Occurs**: `'hi'` is length 2; `length=5` requires 3 more characters. The fill string `'xy'` is repeated and truncated to 3 characters → `'xyx'` + `'hi'` = `'xyxhi'`.

**Example 3: Truncation When String Exceeds Length**

```sql
SELECT LPAD('abcdef', 3, 'x') AS truncated;
```

**Expected Output**

| truncated |
|-----------|
| abc       |

**Why This Output Occurs**: Since `'abcdef'` is longer than 3, it is truncated to the first 3 characters.

#### Real-World Cases

- **Invoice numbers**: Formatting sequential IDs as `INV-00042`.
- **Fixed-width exports**: Aligning columns in a text report.
- **Time formatting**: Converting `'9'` to `'09'` for hours.

---

### 3.5 `RPAD`

#### Definitions

**Core Definition**
`RPAD` right-pads a string to a specified length by appending a fill string.

**Technical Definition**
`RPAD(string, length [, fill])` returns a string of exactly `length` characters obtained by appending `fill` characters to `string`. If `string` is already longer than `length`, it is truncated on the right.

**Beginner-Friendly Explanation**
`RPAD` adds characters to the **right** of your text until it reaches the desired length—like adding trailing dots to a table of contents entry.

#### Purposes

- To create fixed-width output for reports and files.
- To pad strings with spaces or dots for alignment.
- To ensure minimum string length for downstream processing.
- To truncate long strings while guaranteeing a maximum width.

#### Syntax Rules and Structure

**PostgreSQL Syntax**
```
RPAD ( string text, length integer [, fill text ] ) → text
```
**MySQL Syntax**
```
RPAD ( str, len, padstr )
```

**Syntax Rules**

1. If `string` is longer than `length`, it is truncated to `length`.
2. Default fill is a space in PostgreSQL.
3. `RPAD(NULL, length, fill)` returns `NULL`.

**Constraints and Limitations**

- Truncation occurs on the right, potentially cutting multibyte characters if `length` is interpreted in bytes (vendor-dependent).
- Not standard SQL; vendor-specific.

#### Annotated Code Examples

**Example 1: Right-Padding with Spaces**

```sql
SELECT RPAD('hello', 8) AS rpadded;
```

**Expected Output**

| rpadded |
|---------|
| hello   |

(Three trailing spaces appended.)

**Example 2: Right-Padding with Dots**

```sql
SELECT RPAD('Chapter 1', 15, '.') AS toc_entry;
```

**Expected Output**

| toc_entry     |
|---------------|
| Chapter 1..... |

**Why This Output Occurs**: `'Chapter 1'` is 9 characters; 6 dots are appended to reach 15.

#### Real-World Cases

- **Table of contents generation**: Aligning page numbers with dot leaders.
- **Fixed-width data export**: Padding fields to exact widths.
- **Display formatting**: Creating uniform-length codes in UI.

---

## 4. Extraction & Substrings

### 4.1 `SUBSTRING`

#### Definitions

**Core Definition**
`SUBSTRING` extracts a portion of a string starting at a specified position.

**Technical Definition**
`SUBSTRING(string FROM start [FOR count])` or `SUBSTRING(string, start [, count])` returns the substring of `string` beginning at the 1-based character position `start` and extending for `count` characters (or to the end of the string if `count` is omitted). SQL-standard syntax uses the `FROM`/`FOR` keywords.

**Beginner-Friendly Explanation**
`SUBSTRING` lets you cut out a piece of text—like taking a slice from a loaf of bread.

#### Purposes

- To extract specific components from structured strings (e.g., area code from a phone number).
- To truncate strings for display.
- To parse fixed-format fields.
- To isolate substrings for comparison or transformation.

#### Syntax Rules and Structure

**SQL-Standard Syntax**
```
SUBSTRING ( string FROM start [ FOR count ] )
```
**PostgreSQL Alternative**
```
SUBSTRING ( string, start [, count] )
```
**MySQL Syntax**
```
SUBSTRING ( str, pos [, len] )
SUBSTRING ( str FROM pos [ FOR len ] )
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `string` / `str` | Input string. |
| `start` / `pos` | 1-based starting position. If negative, counts from the end (vendor-specific). |
| `count` / `len` | Number of characters to extract (optional). |

**Syntax Rules**

1. If `start` is less than 1, behaviour varies: some systems treat it as 1, others count from the end.
2. If `count` is omitted, the substring extends to the end of the string.
3. `SUBSTRING(NULL, ...)` returns `NULL`.
4. SQL-standard syntax uses `FROM` and `FOR`.

**Constraints and Limitations**

- Negative start positions are vendor-specific.
- Byte vs. character semantics may affect multibyte strings.
- Not all systems support the `FROM`/`FOR` keyword syntax (though most modern ones do).

#### Annotated Code Examples

**Example 1: Extracting a Substring with `FROM`/`FOR`**

```sql
SELECT SUBSTRING('Thomas' FROM 2 FOR 3) AS sub;
```

**Expected Output**

| sub  |
|------|
| hom  |

**Why This Output Occurs**: Starting at position 2 (`h`), 3 characters are extracted: `h`, `o`, `m` → `'hom'`.

**Example 2: Omitting the Count**

```sql
SELECT SUBSTRING('alphabet' FROM 3) AS sub_to_end;
```

**Expected Output**

| sub_to_end |
|------------|
| phabet     |

**Example 3: Comma-Separated Syntax**

```sql
SELECT SUBSTRING('alphabet', 3, 2) AS sub_comma;
```

**Expected Output**

| sub_comma |
|-----------|
| ph        |

#### Real-World Cases

- **Phone number parsing**: Extracting area code from `'(555) 123-4567'`.
- **Date parsing**: Extracting year from `'2024-01-15'`.
- **Log analysis**: Pulling request paths from URL strings.

---

### 4.2 `LEFT`

#### Definitions

**Core Definition**
`LEFT` returns the first *n* characters of a string.

**Technical Definition**
`LEFT(string, n)` returns the leftmost `n` characters of `string`. If `n` is negative, returns all but the last `|n|` characters (PostgreSQL behaviour).

**Beginner-Friendly Explanation**
`LEFT` gives you the beginning of a word—the first few letters.

#### Purposes

- To truncate strings for display (e.g., show first 50 characters).
- To extract prefixes (e.g., first three letters of a country code).
- To implement left-side parsing.

#### Syntax Rules and Structure

**PostgreSQL Syntax**
```
LEFT ( string text, n integer ) → text
```
**MySQL Syntax**
```
LEFT ( str, len )
```
**SQL Server Syntax**
```
LEFT ( character_expression, integer_expression )
```

**Syntax Rules**

1. If `n` is greater than the string length, the entire string is returned.
2. If `n` is negative in PostgreSQL, returns all but the last `|n|` characters.
3. `LEFT(NULL, n)` returns `NULL`.

**Constraints and Limitations**

- Negative `n` behaviour is not portable.
- SQL Server’s `LEFT` does not support negative `n`.

#### Annotated Code Examples

**Example 1: Extracting the First Two Characters**

```sql
SELECT LEFT('abcde', 2) AS left_result;
```

**Expected Output**

| left_result |
|-------------|
| ab          |

**Example 2: Negative `n` in PostgreSQL**

```sql
SELECT LEFT('abcde', -1) AS left_negative;
```

**Expected Output**

| left_negative |
|---------------|
| abcd          |

**Why This Output Occurs**: Negative `n` returns all but the last `|n|` characters (removes the last 1 character).

#### Real-World Cases

- **Preview text**: Showing the first 100 characters of an article.
- **Code prefix extraction**: Getting the first two letters of a product code.

---

### 4.3 `RIGHT`

#### Definitions

**Core Definition**
`RIGHT` returns the last *n* characters of a string.

**Technical Definition**
`RIGHT(string, n)` returns the rightmost `n` characters of `string`. If `n` is negative in PostgreSQL, returns all but the first `|n|` characters.

**Beginner-Friendly Explanation**
`RIGHT` gives you the ending of a word—the last few letters.

#### Purposes

- To extract suffixes (e.g., file extensions).
- To display trailing portions of codes.
- To implement right-side parsing.

#### Syntax Rules and Structure

**PostgreSQL Syntax**
```
RIGHT ( string text, n integer ) → text
```
**MySQL / SQL Server Syntax**
```
RIGHT ( str, len )
```

**Syntax Rules**

1. If `n` > string length, the entire string is returned.
2. Negative `n` (PostgreSQL) returns all but the first `|n|` characters.
3. `RIGHT(NULL, n)` returns `NULL`.

**Constraints and Limitations**

- Negative `n` is PostgreSQL-specific.
- Byte vs. character semantics may affect multibyte strings.

#### Annotated Code Examples

**Example 1: Extracting the Last Two Characters**

```sql
SELECT RIGHT('abcde', 2) AS right_result;
```

**Expected Output**

| right_result |
|--------------|
| de           |

**Example 2: Negative `n` in PostgreSQL**

```sql
SELECT RIGHT('abcde', -1) AS right_negative;
```

**Expected Output**

| right_negative |
|----------------|
| bcde           |

**Why This Output Occurs**: Negative `n` returns all but the first `|n|` characters.

#### Real-World Cases

- **File extension extraction**: `RIGHT(filename, 4)` → `'.pdf'`.
- **Last digits**: Extracting the last four digits of a credit card for display.

---

### 4.4 `SPLIT_PART`

#### Definitions

**Core Definition**
`SPLIT_PART` splits a string by a delimiter and returns the *n*-th part.

**Technical Definition**
`SPLIT_PART(string, delimiter, n)` splits `string` at occurrences of `delimiter` and returns the `n`-th field (1-based). If `n` is negative, returns the `|n|`-th field from the end. If the field does not exist, returns an empty string.

**Beginner-Friendly Explanation**
`SPLIT_PART` treats a delimited string like a list and gives you one item from that list by its position.

#### Purposes

- To parse CSV-like strings stored in a single column.
- To extract specific components from composite codes (e.g., `'US-CA-2024'`).
- To access the *n*-th element of a delimited list without using arrays.

#### Syntax Rules and Structure

**PostgreSQL Syntax**
```
SPLIT_PART ( string text, delimiter text, n integer ) → text
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `string` | Input string to split. |
| `delimiter` | String used as the separator. |
| `n` | Field number (1-based; negative counts from end). |
| `→ text` | The `n`-th field, or empty string if out of range. |

**Syntax Rules**

1. `n` must be a positive or negative integer; `n = 0` raises an error.
2. If `delimiter` is an empty string, the entire string is treated as one field.
3. `SPLIT_PART(NULL, ...)` returns `NULL`.

**Constraints and Limitations**

- **PostgreSQL-specific**; not available in MySQL or SQL Server natively.
- Does not support regular expressions (use `REGEXP_SPLIT_TO_TABLE` for regex-based splitting).
- Returns empty string, not `NULL`, when the field index is out of range.

#### Annotated Code Examples

**Example 1: Extracting the Second Field**

```sql
SELECT SPLIT_PART('abc~@~def~@~ghi', '~@~', 2) AS second_field;
```

**Expected Output**

| second_field |
|--------------|
| def          |

**Why This Output Occurs**: The string is split into three fields: `abc`, `def`, `ghi`; the 2nd is `def`.

**Example 2: Negative Index (From the End)**

```sql
SELECT SPLIT_PART('abc,def,ghi,jkl', ',', -2) AS second_from_end;
```

**Expected Output**

| second_from_end |
|-----------------|
| ghi             |

**Why This Output Occurs**: Negative index `-2` returns the second field from the end (`ghi`).

**Example 3: Out-of-Range Field**

```sql
SELECT SPLIT_PART('a,b,c', ',', 5) AS out_of_range;
```

**Expected Output**

| out_of_range |
|--------------|
|              |

(Empty string.)

#### Real-World Cases

- **Parsing hierarchical codes**: `SPLIT_PART(product_code, '-', 2)` extracts the category.
- **Log line parsing**: Extracting the HTTP status code from a space-delimited log line.
- **Configuration values**: Splitting `'key=value'` pairs and accessing the value.

---

## 5. Searching & Position

### 5.1 `POSITION`

#### Definitions

**Core Definition**
`POSITION` returns the 1-based index of the first occurrence of a substring within a string.

**Technical Definition**
`POSITION(substring IN string)` returns an integer indicating the starting position of the first occurrence of `substring` within `string`, or `0` if `substring` is not found. This is SQL-standard syntax.

**Beginner-Friendly Explanation**
`POSITION` tells you where a smaller word first appears inside a larger text—like finding the page number of a word in a book.

#### Purposes

- To locate the position of a delimiter or keyword within a string.
- To support substring extraction based on dynamic positions.
- To filter rows where a substring exists.
- To implement basic search functionality.

#### Syntax Rules and Structure

**SQL-Standard Syntax**
```
POSITION ( substring IN string )
```
**PostgreSQL Alternative**
```
POSITION ( substring IN string ) → integer
```
**MySQL Syntax**
```
POSITION ( substr IN str )
```
(MySQL also provides `LOCATE(substr, str)` with reversed argument order.)

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `substring` | The substring to search for. |
| `string` | The string to search within. |
| `→ integer` | 1-based position, or `0` if not found. |

**Syntax Rules**

1. If `substring` is not found, returns `0`.
2. If `substring` is empty, returns `1` (in most implementations).
3. `POSITION(NULL IN string)` returns `NULL`.
4. Case sensitivity depends on collation.

**Constraints and Limitations**

- Not to be confused with `STRPOS`, which uses comma-separated arguments and reversed order.
- Does not support regular expressions (use `REGEXP_INSTR` or similar for regex).

#### Annotated Code Examples

**Example 1: Finding a Substring**

```sql
SELECT POSITION('lo' IN 'hello') AS position_result;
```

**Expected Output**

| position_result |
|-----------------|
| 4               |

**Why This Output Occurs**: `'lo'` starts at the 4th character of `'hello'`.

**Example 2: Substring Not Found**

```sql
SELECT POSITION('z' IN 'hello') AS not_found;
```

**Expected Output**

| not_found |
|-----------|
| 0         |

#### Real-World Cases

- **URL parsing**: Finding the position of `'?'` to separate path from query string.
- **Email validation**: Checking for the position of `'@'`.
- **Log analysis**: Locating a keyword in log messages.

---

### 5.2 `STRPOS`

#### Definitions

**Core Definition**
`STRPOS` returns the 1-based position of the first occurrence of a substring in a string, using comma-separated arguments.

**Technical Definition**
`STRPOS(string, substring)` is a PostgreSQL function that returns the starting index of `substring` within `string`, or `0` if not found. It is functionally similar to `POSITION` but uses a different argument order: string first, substring second.

**Beginner-Friendly Explanation**
`STRPOS` does the same job as `POSITION` but with a different calling style—both tell you where a substring begins.

#### Purposes

- To locate substrings when the comma-separated argument style is more convenient.
- To provide a portable alternative to `POSITION` in PostgreSQL-centric code.
- To support dynamic string parsing.

#### Syntax Rules and Structure

**PostgreSQL Syntax**
```
STRPOS ( string text, substring text ) → integer
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `string` | The string to search within. |
| `substring` | The substring to find. |
| `→ integer` | 1-based position, or `0` if not found. |

**Syntax Rules**

1. Argument order is **string first**, then substring—the opposite of `POSITION`.
2. Returns `0` if not found.
3. `STRPOS(NULL, ...)` returns `NULL`.

**Constraints and Limitations**

- **PostgreSQL-specific**; not available in MySQL or SQL Server (use `LOCATE` or `CHARINDEX`).
- Does not support regular expressions.

#### Annotated Code Examples

**Example 1: Basic `STRPOS`**

```sql
SELECT STRPOS('high', 'ig') AS strpos_result;
```

**Expected Output**

| strpos_result |
|---------------|
| 2             |

**Why This Output Occurs**: `'ig'` starts at position 2 in `'high'`.

**Example 2: Contrast with `POSITION`**

```sql
SELECT
  POSITION('ig' IN 'high') AS position_result,
  STRPOS('high', 'ig')     AS strpos_result;
```

**Expected Output**

| position_result | strpos_result |
|-----------------|---------------|
| 2               | 2             |

Both return the same value; only the syntax differs.

#### Real-World Cases

- **Parsing delimited fields**: Finding the position of a delimiter to extract subsequent fields.
- **Search filtering**: `WHERE STRPOS(description, 'urgent') > 0`.

---

## 6. Replacement & Translation

### 6.1 `REPLACE`

#### Definitions

**Core Definition**
`REPLACE` replaces all occurrences of a specified substring with another substring.

**Technical Definition**
`REPLACE(string, from, to)` returns `string` with every occurrence of `from` replaced by `to`. If `from` is an empty string, `string` is returned unchanged. If any argument is `NULL`, the result is `NULL` (except in some vendor-specific behaviours).

**Beginner-Friendly Explanation**
`REPLACE` is like the “Find and Replace” feature in a text editor—it swaps every instance of one piece of text for another.

#### Purposes

- To correct systematic spelling errors in data.
- To substitute placeholder tokens with actual values.
- To remove unwanted characters by replacing them with empty strings.
- To normalise data formats.

#### Syntax Rules and Structure

**General Syntax**
```
REPLACE ( string, from, to )
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `string` | Original string. |
| `from` | Substring to be replaced. |
| `to` | Replacement substring. |
| `→ text` | String with all occurrences replaced. |

**Syntax Rules**

1. All occurrences are replaced, not just the first.
2. If `from` is empty, the original string is returned unchanged.
3. `REPLACE(NULL, ...)` returns `NULL`.
4. Case sensitivity depends on collation.

**Constraints and Limitations**

- Cannot use patterns or regular expressions (use `REGEXP_REPLACE` for regex).
- Replacing with `NULL` returns `NULL`.

#### Annotated Code Examples

**Example 1: Simple Replacement**

```sql
SELECT REPLACE('abcdefabcdef', 'cd', 'XX') AS replaced;
```

**Expected Output**

| replaced       |
|----------------|
| abXXefabXXef   |

**Why This Output Occurs**: Every occurrence of `'cd'` is replaced with `'XX'`.

**Example 2: Removing Characters**

```sql
SELECT REPLACE('a-b-c', '-', '') AS hyphens_removed;
```

**Expected Output**

| hyphens_removed |
|-----------------|
| abc             |

#### Real-World Cases

- **Data cleaning**: Replacing `'N/A'` with `NULL`-equivalent values.
- **Format normalisation**: Converting `'/'` to `'-'` in dates.
- **Template rendering**: Substituting `'{{name}}'` with a user’s name.

---

### 6.2 `TRANSLATE`

#### Definitions

**Core Definition**
`TRANSLATE` replaces each character in a string according to a mapping defined by two character sets.

**Technical Definition**
`TRANSLATE(string, from, to)` returns `string` with each character that appears in `from` replaced by the corresponding character in `to`. If `from` is longer than `to`, the extra characters in `from` are deleted. If `to` is longer than `from`, the extra characters in `to` are ignored.

**Beginner-Friendly Explanation**
`TRANSLATE` is like a character-level substitution cipher—it swaps individual letters according to a rule you define.

#### Purposes

- To perform character-level substitutions (e.g., `'a'→'1'`, `'b'→'2'`).
- To remove specific characters by mapping them to nothing.
- To implement simple encryption or obfuscation.
- To normalise accented characters to ASCII equivalents (though `unaccent` is more comprehensive).

#### Syntax Rules and Structure

**PostgreSQL Syntax**
```
TRANSLATE ( string text, from text, to text ) → text
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `string` | Input string. |
| `from` | Set of characters to replace. |
| `to` | Set of replacement characters. |
| `→ text` | Translated string. |

**Syntax Rules**

1. Character-by-character correspondence: the *i*-th character in `from` is replaced by the *i*-th character in `to`.
2. If `from` has more characters than `to`, the excess characters in `from` are removed from the string.
3. If `to` has more characters than `from`, the excess characters in `to` are ignored.
4. `TRANSLATE(NULL, ...)` returns `NULL`.

**Constraints and Limitations**

- Only single-character mappings; cannot replace multi-character substrings (use `REPLACE` for that).
- Behaviour with multibyte characters depends on character set support.

#### Annotated Code Examples

**Example 1: Character Substitution**

```sql
SELECT TRANSLATE('12345', '143', 'abc') AS translated;
```

**Expected Output**

| translated |
|------------|
| a2c5b      |

**Why This Output Occurs**: Mapping: `'1'→'a'`, `'4'→'b'`, `'3'→'c'`. The string `'12345'` becomes `'a2c5b'`.

**Example 2: Removing Characters**

```sql
SELECT TRANSLATE('hello', 'el', '') AS chars_removed;
```

**Expected Output**

| chars_removed |
|---------------|
| h             |

**Why This Output Occurs**: `'e'` and `'l'` are mapped to nothing (because `to` is shorter than `from`), so they are removed → `'h'` remains.

#### Real-World Cases

- **Simple cipher**: Swapping letters for digits in a puzzle.
- **Character cleanup**: Removing punctuation by mapping it to empty strings.
- **Data masking**: Replacing digits with `'X'` for privacy.

---

## 7. Concatenation & Aggregation

### 7.1 `CONCAT`

#### Definitions

**Core Definition**
`CONCAT` joins two or more strings into a single string.

**Technical Definition**
`CONCAT(val1, val2, ...)` returns the concatenation of the text representations of all arguments. In PostgreSQL, `NULL` arguments are **ignored** (treated as empty strings); in MySQL and SQL Server, if any argument is `NULL`, the result is `NULL` (unless `CONCAT_WS` is used).

**Beginner-Friendly Explanation**
`CONCAT` glues pieces of text together—like joining first name and last name into a full name.

#### Purposes

- To combine values from multiple columns into one string.
- To build composite keys or labels.
- To format output by joining literals and column values.
- To create full names, addresses, or descriptive strings.

#### Syntax Rules and Structure

**General Syntax**
```
CONCAT ( val1, val2 [, ...] )
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `val1, val2, ...` | Two or more expressions; non-string values are implicitly cast. |
| `→ text` | Concatenated string. |

**Syntax Rules**

1. PostgreSQL: `NULL` arguments are ignored → `CONCAT('a', NULL, 'b')` = `'ab'`.
2. MySQL / SQL Server: `NULL` argument makes the entire result `NULL` (SQL Server may return empty string in some versions; check version-specific behaviour).
3. At least one argument is required.

**Constraints and Limitations**

- `NULL` handling is vendor-specific and a common source of bugs.
- Implicit casting may fail for incompatible types.

#### Annotated Code Examples

**Example 1: Concatenating Literals and Numbers**

```sql
-- PostgreSQL
SELECT CONCAT('abcde', 2, NULL, 22) AS concat_result;
```

**Expected Output**

| concat_result |
|---------------|
| abcde222      |

**Why This Output Occurs**: `NULL` is ignored; `2` and `22` are cast to text and concatenated → `'abcde' + '2' + '22'` = `'abcde222'`.

**Example 2: MySQL NULL Behaviour**

```sql
-- MySQL
SELECT CONCAT('a', NULL, 'b') AS mysql_concat;
```

**Expected Output**

| mysql_concat |
|--------------|
| NULL         |

**Why This Output Occurs**: In MySQL, any `NULL` argument causes `CONCAT` to return `NULL`.

#### Real-World Cases

- **Full name generation**: `CONCAT(first_name, ' ', last_name)`.
- **Address formatting**: Joining street, city, and postal code.
- **Label creation**: `CONCAT('Order #', order_id)`.

---

### 7.2 `CONCAT_WS`

#### Definitions

**Core Definition**
`CONCAT_WS` concatenates strings with a specified separator, skipping `NULL` values.

**Technical Definition**
`CONCAT_WS(separator, val1, val2, ...)` returns a string formed by joining the non-`NULL` arguments with `separator`. The separator is required and must not be `NULL`. `NULL` arguments are skipped.

**Beginner-Friendly Explanation**
`CONCAT_WS` is like `CONCAT` but inserts a separator (e.g., a comma) between each piece and automatically skips missing (NULL) pieces.

#### Purposes

- To build comma-separated or delimiter-separated lists.
- To safely concatenate optional fields without introducing `NULL` results.
- To create readable composite strings.

#### Syntax Rules and Structure

**General Syntax**
```
CONCAT_WS ( separator, val1, val2 [, ...] )
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `separator` | String placed between non-`NULL` arguments. Must not be `NULL`. |
| `val1, val2, ...` | Values to concatenate. |

**Syntax Rules**

1. `separator` must not be `NULL`; if it is, the result is `NULL`.
2. `NULL` arguments are skipped entirely (no separator is added for them).
3. At least two arguments (separator + one value) are required.

**Constraints and Limitations**

- Not standard SQL (though widely supported).
- Separator behaviour with empty strings may vary.

#### Annotated Code Examples

**Example 1: Building a Delimited List with `NULL`s**

```sql
SELECT CONCAT_WS(',', 'abcde', 2, NULL, 22) AS concat_ws_result;
```

**Expected Output**

| concat_ws_result |
|------------------|
| abcde,2,22       |

**Why This Output Occurs**: The `NULL` is skipped; the non-`NULL` values are joined with commas.

**Example 2: Address Formatting with Optional Fields**

```sql
SELECT CONCAT_WS(', ', street, city, state, zip) AS address
FROM addresses;
```

**Expected Output (illustrative)**

| address                  |
|--------------------------|
| 123 Main St, Springfield, IL, 62701 |

If `state` is `NULL`, it is skipped.

#### Real-World Cases

- **Mailing labels**: Joining address components without empty lines.
- **Tag lists**: `CONCAT_WS(', ', tag1, tag2, tag3)`.
- **Report headers**: Building dynamic titles from optional parts.

---

### 7.3 String Aggregation (`STRING_AGG` / `GROUP_CONCAT`)

#### Definitions

**Core Definition**
String aggregation functions concatenate values from multiple rows into a single string, typically grouped by another column.

**Technical Definition**
`STRING_AGG(expression, delimiter)` in PostgreSQL and SQL Server, and `GROUP_CONCAT(expression [ORDER BY ...] [SEPARATOR sep])` in MySQL, are aggregate functions that produce a single string by concatenating non-`NULL` values from a group of rows, separated by a specified delimiter.

**Beginner-Friendly Explanation**
String aggregation turns many rows into one comma-separated list—like taking all the city names in a country and combining them into a single string.

#### Purposes

- To produce comma-separated lists for reports and dashboards.
- To denormalise one-to-many relationships for display.
- To aggregate tags, categories, or labels associated with an entity.
- To generate CSV-like output from query results.

#### Syntax Rules and Structure

**PostgreSQL**
```
STRING_AGG ( expression, delimiter ) → text
```
**SQL Server**
```
STRING_AGG ( expression, separator ) → nvarchar
```
**MySQL**
```
GROUP_CONCAT ( [DISTINCT] expression [ORDER BY ...] [SEPARATOR sep] )
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `expression` | Column or expression to concatenate. |
| `delimiter` / `separator` | String inserted between values. |
| `DISTINCT` | MySQL: eliminate duplicates. |
| `ORDER BY` | MySQL: order values before concatenation. |
| `SEPARATOR` | MySQL: custom delimiter (default `,`). |

**Syntax Rules**

1. Used with `GROUP BY` to aggregate per group.
2. `NULL` values are skipped (not concatenated) in PostgreSQL and SQL Server; MySQL `GROUP_CONCAT` skips `NULL`s.
3. Delimiter is required in PostgreSQL/SQL Server; optional in MySQL (default comma).
4. Ordering is not guaranteed unless explicitly specified (MySQL `ORDER BY`; PostgreSQL 9.0+ supports `ORDER BY` inside `STRING_AGG`).

**Constraints and Limitations**

- **Vendor-specific naming**: `STRING_AGG` (PostgreSQL, SQL Server), `GROUP_CONCAT` (MySQL).
- Result size limited by `max_allowed_packet` in MySQL and similar limits in other systems.
- `STRING_AGG` in SQL Server requires SQL Server 2017 or later.
- Ordering syntax varies significantly.

#### Annotated Code Examples

**Example 1: PostgreSQL `STRING_AGG`**

```sql
-- Setup: table country_city(country, city)
SELECT
  country,
  STRING_AGG(city, ', ' ORDER BY city) AS cities
FROM country_city
GROUP BY country;
```

**Expected Output (illustrative)**

| country | cities               |
|---------|----------------------|
| USA     | Boston, New York, Seattle |
| UK      | London, Manchester   |

**Why This Output Occurs**: For each country, cities are concatenated with `', '`, ordered alphabetically.

**Example 2: MySQL `GROUP_CONCAT` with `DISTINCT` and `ORDER BY`**

```sql
SELECT
  category,
  GROUP_CONCAT(DISTINCT product_name ORDER BY product_name SEPARATOR ' | ') AS products
FROM products
GROUP BY category;
```

**Expected Output (illustrative)**

| category | products              |
|----------|-----------------------|
| Books    | SQL Guide \| Python 101 |

**Why This Output Occurs**: Duplicate product names are removed, values are ordered, and the separator is `' | '`.

**Example 3: SQL Server `STRING_AGG`**

```sql
SELECT
  department,
  STRING_AGG(employee_name, '; ') AS team
FROM employees
GROUP BY department;
```

**Expected Output (illustrative)**

| department | team             |
|------------|------------------|
| Sales      | Alice; Bob; Carol |

**Why This Output Occurs**: SQL Server 2017+ `STRING_AGG` concatenates with `'; '`.

#### Real-World Cases

- **E-commerce**: Listing all product names in an order as a comma-separated string.
- **CRM**: Aggregating all phone numbers for a contact.
- **Content management**: Joining tags associated with an article.

---

## 8. Regular Expressions

### 8.1 `REGEXP_REPLACE`

#### Definitions

**Core Definition**
`REGEXP_REPLACE` replaces substrings that match a POSIX regular expression pattern with a replacement string.

**Technical Definition**
`REGEXP_REPLACE(string, pattern, replacement [, flags])` scans `string` for matches of the POSIX regular expression `pattern` and replaces each match with `replacement`. Optional `flags` (e.g., `'g'` for global, `'i'` for case-insensitive) modify the behaviour.

**Beginner-Friendly Explanation**
`REGEXP_REPLACE` is a supercharged “Find and Replace” that uses patterns instead of exact text—so you can replace all email addresses, phone numbers, or dates in one go.

#### Purposes

- To perform pattern-based substitutions (e.g., mask credit card numbers).
- To remove complex patterns from text.
- To reformat data using captured groups.
- To clean data by replacing multiple variations of a pattern.

#### Syntax Rules and Structure

**PostgreSQL Syntax**
```
REGEXP_REPLACE ( string text, pattern text, replacement text [, flags text] ) → text
```
**MySQL Syntax (8.0+)**
```
REGEXP_REPLACE ( expr, pat, repl [, pos [, occurrence [, match_type ]]] )
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `string` | Input string. |
| `pattern` | POSIX regular expression. |
| `replacement` | Replacement string; may contain backreferences like `\1`. |
| `flags` | Optional: `g` (global), `i` (case-insensitive), `n`, `m`, etc. |

**Syntax Rules**

1. By default, only the first match is replaced unless the `'g'` flag is specified.
2. Backreferences (`\1`, `\2`, …) in `replacement` refer to captured groups.
3. `REGEXP_REPLACE(NULL, ...)` returns `NULL`.

**Constraints and Limitations**

- Regex dialect varies between PostgreSQL (POSIX), MySQL (ICU), and others.
- Performance can be significantly worse than plain `REPLACE`.
- Not all systems support the same set of flags.

#### Annotated Code Examples

**Example 1: Global Replacement**

```sql
SELECT REGEXP_REPLACE('foobarbaz', 'b..', 'X', 'g') AS regex_replaced;
```

**Expected Output**

| regex_replaced |
|----------------|
| fooXarYXazY    |

**Why This Output Occurs**: The pattern `b..` matches `bar` and `baz`; with the global flag `'g'`, both are replaced with `'X'`.

**Example 2: Using Backreferences**

```sql
SELECT REGEXP_REPLACE('John Smith', '(\w+)\s+(\w+)', '\2, \1') AS swapped;
```

**Expected Output**

| swapped     |
|-------------|
| Smith, John |

**Why This Output Occurs**: Captured groups `\1` = `John`, `\2` = `Smith`; the replacement reorders them.

#### Real-World Cases

- **Data masking**: Replacing `\d{4}-\d{4}-\d{4}-\d{4}` with `'****-****-****-****'`.
- **Log sanitisation**: Removing IP addresses matching a regex.
- **Format conversion**: Transforming date formats.

---

### 8.2 `REGEXP_MATCHES`

#### Definitions

**Core Definition**
`REGEXP_MATCHES` returns the substrings captured by a regular expression match.

**Technical Definition**
`REGEXP_MATCHES(string, pattern [, flags])` returns a set of `text[]` arrays, each containing the captured groups from a match. If no match is found, no rows are returned. With the `'g'` flag, all matches are returned; without it, only the first match is returned.

**Beginner-Friendly Explanation**
`REGEXP_MATCHES` extracts the parts of a string that match a pattern—like pulling out all the phone numbers from a document.

#### Purposes

- To extract structured data from unstructured text.
- To validate that a string matches a pattern.
- To capture multiple components (e.g., year, month, day) from a date string.

#### Syntax Rules and Structure

**PostgreSQL Syntax**
```
REGEXP_MATCHES ( string text, pattern text [, flags text] ) → setof text[]
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `string` | Input string. |
| `pattern` | POSIX regular expression, often with capture groups. |
| `flags` | `'g'` for all matches, `'i'` for case-insensitive. |
| `→ setof text[]` | Each row is an array of captured groups. |

**Syntax Rules**

1. Without the `'g'` flag, only the first match is returned.
2. With `'g'`, all matches are returned.
3. If the pattern contains no capture groups, the entire match is returned as a single-element array.
4. If no match, returns zero rows.

**Constraints and Limitations**

- **PostgreSQL-specific**; MySQL uses `REGEXP_SUBSTR` for similar functionality.
- Returns a set, so it must be used in a `SELECT` list or with `LATERAL`.

#### Annotated Code Examples

**Example 1: Extracting Captured Groups**

```sql
SELECT REGEXP_MATCHES('foobarbequebaz', '(bar)(beque)') AS matched_groups;
```

**Expected Output**

| matched_groups |
|----------------|
| {bar,beque}    |

**Why This Output Occurs**: The pattern captures two groups: `bar` and `beque`.

**Example 2: Global Matching**

```sql
SELECT REGEXP_MATCHES('a1 b2 c3', '[a-z]([0-9])', 'g') AS digits;
```

**Expected Output**

| digits |
|--------|
| {1}    |
| {2}    |
| {3}    |

**Why This Output Occurs**: The `'g'` flag returns all matches; the capture group extracts the digit from each pair.

#### Real-World Cases

- **Log parsing**: Extracting IP addresses, timestamps, and status codes.
- **Text mining**: Finding all mentions or hashtags in social media text.
- **Data validation**: Checking whether a value conforms to a pattern.

---

### 8.3 `REGEXP_SPLIT_TO_TABLE`

#### Definitions

**Core Definition**
`REGEXP_SPLIT_TO_TABLE` splits a string using a regular expression as the delimiter and returns each part as a row.

**Technical Definition**
`REGEXP_SPLIT_TO_TABLE(string, pattern [, flags])` splits `string` at occurrences of the POSIX regular expression `pattern` and returns the resulting fields as a set of text rows. If the pattern does not match, the entire string is returned as a single row.

**Beginner-Friendly Explanation**
`REGEXP_SPLIT_TO_TABLE` is like `SPLIT_PART` but much more powerful—it can split on patterns, not just fixed delimiters, and returns the parts as separate rows.

#### Purposes

- To split strings on variable or complex delimiters (e.g., multiple spaces, punctuation).
- To tokenise text for analysis.
- To expand delimited lists into rows for joining or aggregation.

#### Syntax Rules and Structure

**PostgreSQL Syntax**
```
REGEXP_SPLIT_TO_TABLE ( string text, pattern text [, flags text] ) → setof text
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `string` | Input string. |
| `pattern` | Regular expression used as the delimiter. |
| `flags` | Optional regex flags. |
| `→ setof text` | One row per split field. |

**Syntax Rules**

1. If `pattern` does not match, the entire string is returned as a single row.
2. Empty fields may be produced if delimiters are adjacent.
3. `NULL` input returns no rows.

**Constraints and Limitations**

- **PostgreSQL-specific**.
- Does not return the delimiters themselves.
- Performance may be worse than `string_to_table` for simple delimiters.

#### Annotated Code Examples

**Example 1: Splitting on Whitespace**

```sql
SELECT REGEXP_SPLIT_TO_TABLE('hello   world  foo', '\s+') AS token;
```

**Expected Output**

| token |
|-------|
| hello |
| world |
| foo   |

**Why This Output Occurs**: The pattern `\s+` matches one or more whitespace characters; the string is split into three tokens.

**Example 2: Splitting on Multiple Delimiters**

```sql
SELECT REGEXP_SPLIT_TO_TABLE('a,b;c d', '[,; ]') AS part;
```

**Expected Output**

| part |
|------|
| a    |
| b    |
| c    |
| d    |

#### Real-World Cases

- **Tokenisation**: Splitting sentences into words for text analysis.
- **Parsing mixed-delimiter data**: Handling CSV files with inconsistent separators.
- **Log processing**: Splitting log lines on variable whitespace.

---

## 9. Comparison & Standards

### 9.1 String Normalisation

#### Definitions

**Core Definition**
String normalisation is the process of transforming a string into a canonical form so that equivalent strings compare as equal.

**Technical Definition**
Unicode normalisation converts a string into one of four standard forms (NFC, NFD, NFKC, NFKD) so that characters that are visually or semantically equivalent are represented by the same sequence of code points. In PostgreSQL, the `NORMALIZE(string, form)` function performs this transformation, and `IS [form] NORMALIZED` checks whether a string is already in the specified form.

**Beginner-Friendly Explanation**
Sometimes the same letter can be stored in different ways in a computer (e.g., `é` as one character or as `e` + an accent). Normalisation makes them all use the same representation so that comparisons work correctly.

#### Purposes

- To ensure that visually identical strings compare as equal.
- To preprocess text for case-insensitive or accent-insensitive matching.
- To standardise data before storage or indexing.
- To avoid false negatives in search and uniqueness checks.

#### Syntax Rules and Structure

**PostgreSQL Normalisation Function**
```
NORMALIZE ( string text [, form text ] ) → text
```
**Normalisation Check**
```
string IS [ form ] NORMALIZED → boolean
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `string` | Input string. |
| `form` | `NFC` (default), `NFD`, `NFKC`, or `NFKD`. |
| `→ text` / `boolean` | Normalised string or normalisation check result. |

**Syntax Rules**

1. `NORMALIZE` requires the server encoding to be UTF8.
2. `IS NORMALIZED` returns `true` if the string is already in the specified form.
3. `NFC` is the most common form for storage and interchange.

**Constraints and Limitations**

- Only available when the database encoding is UTF8.
- Normalisation can change string length (e.g., NFD expands composed characters).
- Performance cost for large datasets.

#### Annotated Code Examples

**Example 1: Normalising a String**

```sql
SELECT NORMALIZE(U&'\0061\0308bc', NFC) AS normalised;
```

**Expected Output**

| normalised |
|------------|
| äbc        |

**Why This Output Occurs**: The sequence `a` + combining diaeresis is normalised to the precomposed character `ä` in NFC form.

**Example 2: Checking Normalisation**

```sql
SELECT U&'\0061\0308bc' IS NFD NORMALIZED AS is_nfd;
```

**Expected Output**

| is_nfd |
|--------|
| t      |

**Why This Output Occurs**: The string is already in NFD form (decomposed), so the check returns `true`.

#### Real-World Cases

- **International name matching**: Ensuring `'José'` (precomposed) and `'Jose\u0301'` (decomposed) are treated as equal.
- **Search indexing**: Normalising text before indexing to improve recall.
- **Data deduplication**: Identifying records that differ only in Unicode representation.

---

### 9.2 String Comparison

#### Definitions

**Core Definition**
String comparison determines the ordering and equality of strings according to a collation.

**Technical Definition**
String comparison in SQL is governed by the collation of the operands. Collations define rules for character weighting, case sensitivity, accent sensitivity, and locale-specific ordering. Comparison operators (`=`, `<>`, `<`, `>`, `<=`, `>=`) and `ORDER BY` use these rules.

**Beginner-Friendly Explanation**
String comparison is how the database decides whether two words are the same or which one comes first alphabetically—but the rules can change depending on language and settings.

#### Purposes

- To filter rows based on textual equality or ordering.
- To sort results in a locale-aware manner.
- To enforce uniqueness constraints with collation-aware equality.
- To implement case-insensitive or accent-insensitive matching.

#### Syntax Rules and Structure

**Comparison Operators**
```
expression = expression
expression < expression
expression > expression
expression LIKE pattern
expression ILIKE pattern  -- PostgreSQL case-insensitive
```

**Collation Specification (PostgreSQL)**
```
expression COLLATE collation_name
```

**Syntax Rules**

1. Collation can be specified at the column level, database level, or query level.
2. `=` uses collation rules for equality (which may be case-insensitive if a nondeterministic collation is used).
3. `LIKE` is case-sensitive in most systems; `ILIKE` is PostgreSQL-specific.
4. Binary comparison (`COLLATE "C"`) compares byte values directly.

**Constraints and Limitations**

- Collation behaviour varies significantly between RDBMS.
- Nondeterministic collations may disable certain optimisations.
- Comparing strings with different collations may require explicit `COLLATE`.

#### Annotated Code Examples

**Example 1: Case-Insensitive Comparison with `LOWER`**

```sql
SELECT *
FROM users
WHERE LOWER(username) = LOWER('Alice');
```

**Expected Output**: Rows where the username matches `'Alice'` in any case.

**Example 2: Collation-Aware Sorting**

```sql
SELECT name
FROM customers
ORDER BY name COLLATE "de_DE";
```

**Expected Output**: Names sorted according to German collation rules (e.g., `ä` near `a`).

#### Real-World Cases

- **User directories**: Sorting names according to local alphabetical order.
- **Unique constraints**: Enforcing case-insensitive uniqueness of email addresses with a nondeterministic collation.
- **Search**: Using `ILIKE` for case-insensitive pattern matching.

---

### 9.3 Collations

#### Definitions

**Core Definition**
A collation is a set of rules that determines how strings are compared and sorted.

**Technical Definition**
In SQL, a collation is an object that defines the ordering and equality semantics of character strings, including case sensitivity, accent sensitivity, and locale-specific rules. Collations can be specified at the database, table, column, or expression level. PostgreSQL supports deterministic and nondeterministic collations, with the latter enabling case-insensitive and accent-insensitive comparisons.

**Beginner-Friendly Explanation**
A collation is like a language-specific rulebook for sorting and comparing words—it tells the database whether `'a'` comes before `'A'`, whether `'ä'` is treated as `'a'`, and how to order names in different languages.

#### Purposes

- To sort data according to the conventions of a specific language or region.
- To implement case-insensitive or accent-insensitive uniqueness and comparison.
- To ensure consistent behaviour across databases with different default collations.
- To optimise queries by using appropriate collations for indexes.

#### Syntax Rules and Structure

**Specifying Collation at Query Level**
```
expression COLLATE collation_name
```
**Creating a Collation (PostgreSQL)**
```
CREATE COLLATION collation_name ( locale = '...', provider = '...', deterministic = ... );
```
**Column Definition**
```
column_name data_type COLLATE collation_name
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `collation_name` | Name of the collation (e.g., `"en_US.utf8"`, `"de_DE"`, `"C"`). |
| `deterministic` | If `false`, enables case-insensitive or accent-insensitive comparison. |
| `provider` | `icu`, `libc`, or `builtin`. |

**Syntax Rules**

1. Collation can be set at multiple levels; the most specific applies.
2. Nondeterministic collations (PostgreSQL 12+) allow case-insensitive and accent-insensitive comparisons.
3. The `"C"` collation uses byte-order comparison, which is fast but not locale-aware.
4. Collation names are platform-dependent.

**Constraints and Limitations**

- Nondeterministic collations may prevent the use of certain index types.
- Behaviour differs between PostgreSQL, MySQL, and SQL Server.
- ICU collations require the ICU library to be installed.

#### Annotated Code Examples

**Example 1: Case-Insensitive Uniqueness with Nondeterministic Collation**

```sql
-- PostgreSQL 12+
CREATE COLLATION case_insensitive (
  provider = icu,
  locale = 'und-u-ks-level2',
  deterministic = false
);

CREATE TABLE users (
  email text COLLATE case_insensitive UNIQUE
);

INSERT INTO users VALUES ('Alice@example.com');
INSERT INTO users VALUES ('alice@example.com'); -- violates unique constraint
```

**Expected Behaviour**: The second insert fails because the nondeterministic collation treats both emails as equal.

**Example 2: Sorting with a Specific Collation**

```sql
SELECT name
FROM products
ORDER BY name COLLATE "fr_FR";
```

**Expected Output**: Names sorted according to French collation rules.

#### Real-World Cases

- **Multilingual applications**: Storing data with locale-appropriate collations.
- **Email uniqueness**: Enforcing case-insensitive uniqueness without `LOWER()` in every query.
- **Index optimisation**: Using deterministic collations for fast equality searches.

---

## References

1. **PostgreSQL 16 Documentation — String Functions and Operators** — https://www.postgresql.org/docs/16/functions-string.html
2. **PostgreSQL 16 Documentation — Pattern Matching (Regular Expressions)** — https://www.postgresql.org/docs/16/functions-matching.html
3. **PostgreSQL Documentation — Collation** — https://www.postgresql.org/docs/current/collation.html
4. **MySQL 8.0 Reference Manual — String Functions and Operators** — https://dev.mysql.com/doc/refman/8.0/en/string-functions.html
5. **MySQL 8.0 Reference Manual — Regular Expressions** — https://dev.mysql.com/doc/refman/8.0/en/regexp.html
6. **Microsoft SQL Server — String Functions (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/functions/string-functions-transact-sql
7. **Microsoft SQL Server — STRING_AGG (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/functions/string-agg-transact-sql
8. **SQL:2016 Standard — ISO/IEC 9075-2:2016** — https://www.iso.org/standard/63555.html
9. **Unicode Standard Annex #15 — Unicode Normalization Forms** — https://unicode.org/reports/tr15/
10. **PostgreSQL Wiki — String Functions** — https://wiki.postgresql.org/wiki/String_Functions

---

This cheat sheet provides a comprehensive reference for SQL string functions across PostgreSQL, MySQL, and SQL Server, with standard SQL notes where applicable. Always consult your specific RDBMS documentation for version-specific behaviour and extensions.