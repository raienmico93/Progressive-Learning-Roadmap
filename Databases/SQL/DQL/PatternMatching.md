# SQL Pattern Matching: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL pattern matching is the process of searching for and identifying text strings that conform to a specified pattern, using operators like LIKE, SIMILAR TO, and regular expression functions.

**Technical Definition:** SQL pattern matching is a family of string comparison operations defined by the SQL standard (ISO/IEC 9075) and extended by vendor-specific implementations. The ANSI standard defines the LIKE predicate with `%` and `_` wildcards, and the SIMILAR TO operator for SQL99 regular expressions. Major database systems extend these with POSIX regular expression operators (PostgreSQL `~`, `~*`), function-based regex support (Oracle `REGEXP_LIKE`, `REGEXP_SUBSTR`, `REGEXP_REPLACE`), and bracket-expression wildcards (SQL Server `[a-z]`, `[^0-9]`). Pattern matching is collation-sensitive, meaning case sensitivity and character comparison behavior depend on the database, column, or explicit collation settings.

**Beginner-Friendly Explanation:** Pattern matching is like using "find and replace" in a word processor, but for databases. Instead of searching for an exact word, you search for a pattern. For example, you can find all names that start with "A" (`LIKE 'A%'`), all phone numbers with a specific area code (`LIKE '415%'`), or all email addresses that follow a specific format (using regular expressions). The wildcards `%` and `_` act as placeholders for unknown characters.

### Key Characteristics

- **Two wildcard systems:** Standard SQL uses `%` (zero or more characters) and `_` (exactly one character). SQL Server adds bracket expressions `[a-z]` and `[^0-9]`.
- **Collation-sensitive:** Case sensitivity depends on the database engine and column collation. PostgreSQL's LIKE is case-sensitive; MySQL's default collation is case-insensitive; SQL Server follows the column's collation.
- **Anchoring behavior:** LIKE patterns always match the entire string. To match anywhere within a string, the pattern must start and end with `%`.
- **Escape mechanism:** The ESCAPE clause allows you to search for literal `%` and `_` characters.
- **Regular expression support:** PostgreSQL uses `~` and `~*` operators; MySQL and Oracle provide `REGEXP`/`RLIKE` and `REGEXP_LIKE` functions; SQL Server lacks native regex but offers PATINDEX and LIKE with bracket expressions.
- **Performance-sensitive:** Leading wildcards (`'%abc'`) prevent standard B-tree index usage, requiring alternative indexing strategies like trigram indexes or full-text search.

### Prerequisites

- Basic SQL syntax (SELECT, FROM, WHERE)
- Understanding of string data types (CHAR, VARCHAR, TEXT)
- Familiarity with collation concepts (case sensitivity, character sorting)
- Basic knowledge of indexes and query performance

### Related Programming Areas

- **Full-Text Search:** A specialized form of pattern matching for natural-language documents, using inverted indexes for word-level matching.
- **Data Validation:** Pattern matching is used to validate formats (email addresses, phone numbers, postal codes).
- **Data Cleaning and ETL:** Identifying and correcting inconsistent data using pattern-based rules.
- **Log Analysis:** Searching log files for error patterns or specific event sequences.
- **Security and Filtering:** Validating user input and detecting malicious patterns (SQL injection, XSS).

---

## Core Concepts / Key Features

### 1. Standard SQL Pattern Matching

#### 1.1 LIKE Operator Syntax and Mechanics

**Core Definitions:**

- **Core Definition:** The LIKE operator compares a string expression to a pattern containing wildcard characters, returning TRUE if the string matches the pattern.
- **Technical Definition:** Per the SQL standard, the LIKE predicate is defined as `match_expression [NOT] LIKE pattern [ESCAPE escape_character]`. The pattern is partitioned into sequences of wildcards and non-wildcard strings. The input string matches the pattern if it can be partitioned such that wildcards match one character or any number of characters respectively, and the non-wildcard partitions are equal under the applicable collation.
- **Beginner-Friendly Explanation:** LIKE is the "fuzzy equals" operator. Instead of saying "this column exactly equals 'Smith'", you say "this column looks like 'Sm%'"—anything that starts with "Sm".

**Purposes:**

- To search for strings that match a partial pattern rather than an exact value
- To filter rows based on textual characteristics (prefix, suffix, contains)
- To validate data formats (e.g., phone numbers, postal codes)
- To implement flexible search functionality in applications

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
match_expression [NOT] LIKE pattern [ESCAPE escape_character]
```

**Component Breakdown:**

- `match_expression`: Any valid SQL expression of character data type (column, literal, function result)
- `LIKE`: The pattern-matching operator
- `pattern`: A string containing literal characters and wildcards (`%`, `_`)
- `ESCAPE escape_character`: Optional clause specifying a character to escape wildcards

**Syntax Rules:**

- LIKE pattern matching always covers the entire string.
- To match a sequence anywhere within a string, the pattern must start and end with a percent sign (`%`).
- Trailing spaces in the expression are significant for LIKE (unlike the `=` operator for CHAR data).
- The LIKE condition is collation-sensitive.
- If any argument is not character data, the database engine converts it if possible.

**Constraints and Limitations:**

- LIKE cannot use standard B-tree indexes when the pattern starts with a wildcard (`'%abc'`).
- The `%` wildcard does not match NULL values.
- Pattern length may be limited (e.g., 8,000 bytes in SQL Server).
- Performance degrades significantly with leading wildcards on large tables.

**Annotated Code Examples:**

```sql
-- Setup: Create a sample table
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    phone VARCHAR(20)
);

INSERT INTO employees VALUES (1, 'John', 'Smith', 'john.smith@example.com', '415-555-0100');
INSERT INTO employees VALUES (2, 'Jane', 'Doe', 'jane.doe@example.com', '415-555-0200');
INSERT INTO employees VALUES (3, 'Robert', 'Johnson', 'robert.johnson@example.com', '503-555-0300');
INSERT INTO employees VALUES (4, 'Maria', 'Garcia', 'maria.garcia@example.com', '415-555-0400');
INSERT INTO employees VALUES (5, 'David', 'Lee', 'david.lee@example.com', '212-555-0500');

-- Example 1: Find employees whose last name starts with 'S'
SELECT first_name, last_name
FROM employees
WHERE last_name LIKE 'S%';
```

**Expected Output:**

| first_name | last_name |
|------------|-----------|
| John       | Smith     |

**Explanation:** The pattern `'S%'` matches any string that starts with 'S' followed by zero or more characters. Only Smith matches.

```sql
-- Example 2: Find employees whose email contains 'johnson' anywhere
SELECT first_name, last_name, email
FROM employees
WHERE email LIKE '%johnson%';
```

**Expected Output:**

| first_name | last_name | email                      |
|------------|-----------|----------------------------|
| Robert     | Johnson   | robert.johnson@example.com |

**Explanation:** The pattern `'%johnson%'` matches any string containing 'johnson' anywhere. The leading and trailing `%` allow the match to occur anywhere in the string.

```sql
-- Example 3: Find employees whose first name is exactly 4 characters
SELECT first_name
FROM employees
WHERE first_name LIKE '____';
```

**Expected Output:**

| first_name |
|------------|
| John       |
| Jane       |

**Explanation:** The pattern `'____'` (four underscores) matches any string of exactly four characters. John and Jane both have four-letter names.

**Real-World Cases:**

- **E-commerce:** Searching for products by partial name (e.g., `WHERE product_name LIKE '%phone%'`).
- **CRM:** Finding customers whose last name starts with a specific letter.
- **Data validation:** Checking that phone numbers match a specific format.
- **Log analysis:** Searching for error messages containing specific keywords.

**References:**

- PostgreSQL: LIKE - https://www.postgresql.org/docs/current/functions-matching.html
- Microsoft Learn: LIKE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/like-transact-sql
- Oracle: LIKE Condition - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/LIKE.html


#### 1.2 NOT LIKE Operator for Inverse Matching

**Core Definitions:**

- **Core Definition:** The NOT LIKE operator returns TRUE when the string does not match the pattern.
- **Technical Definition:** NOT LIKE is the logical negation of LIKE. It returns TRUE if the match expression does not match the pattern, FALSE if it does match, and UNKNOWN if either argument is NULL.
- **Beginner-Friendly Explanation:** NOT LIKE is the opposite of LIKE—it finds everything that doesn't match the pattern.

**Purposes:**

- To exclude rows that match a pattern
- To find data that does not conform to an expected format
- To implement negative filtering in queries

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
match_expression NOT LIKE pattern [ESCAPE escape_character]
```

**Component Breakdown:**

- Same components as LIKE, with NOT negating the result

**Syntax Rules:**

- NOT LIKE follows the same rules as LIKE regarding wildcards and escaping.
- NULL handling: If either operand is NULL, the result is UNKNOWN (not TRUE).

**Constraints and Limitations:**

- Same performance implications as LIKE with leading wildcards.
- NULL values are not returned by NOT LIKE (because NULL LIKE pattern is UNKNOWN).

**Annotated Code Examples:**

```sql
-- Find employees whose phone number is NOT in the 415 area code
SELECT first_name, last_name, phone
FROM employees
WHERE phone NOT LIKE '415%';
```

**Expected Output:**

| first_name | last_name | phone        |
|------------|-----------|--------------|
| Robert     | Johnson   | 503-555-0300 |
| David      | Lee       | 212-555-0500 |

**Explanation:** The pattern `'415%'` matches all phone numbers starting with 415. NOT LIKE returns rows where this is false (i.e., the phone number does not start with 415).

**Real-World Cases:**

- **Data cleaning:** Finding records that don't match expected formats.
- **Filtering:** Excluding test accounts (e.g., `WHERE email NOT LIKE '%@test.com'`).
- **Inventory:** Finding products that are not in a specific category.

**References:**

- PostgreSQL: LIKE - https://www.postgresql.org/docs/current/functions-matching.html
- Microsoft Learn: LIKE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/like-transact-sql


### 2. Wildcard Characters

#### 2.1 The % Wildcard (Matching Zero or More Characters)

**Core Definitions:**

- **Core Definition:** The `%` wildcard matches any sequence of zero or more characters.
- **Technical Definition:** In the LIKE predicate, `%` represents any string of zero or more characters. It is the most commonly used wildcard because it allows matching substrings anywhere within a string.
- **Beginner-Friendly Explanation:** `%` is like a blank tile in Scrabble—it can stand for anything, including nothing at all.

**Purposes:**

- To match any number of characters (including none) at a specific position
- To search for substrings anywhere within a string
- To implement prefix, suffix, and contains searches

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
pattern LIKE '%' || value || '%'
```

**Component Breakdown:**

- `%`: Wildcard representing zero or more characters
- Multiple `%` can be used in a single pattern

**Syntax Rules:**

- `%` matches zero or more characters.
- `'%'` alone matches any non-NULL string.
- `%` does not match NULL.

**Constraints and Limitations:**

- `%` at the beginning of a pattern prevents standard index usage.
- `%` does not match across newline characters in some databases.
- The `%` character itself can be matched using the ESCAPE clause.

**Annotated Code Examples:**

```sql
-- Find products with 'phone' anywhere in the name
SELECT product_name FROM products WHERE product_name LIKE '%phone%';

-- Find products starting with 'Phone'
SELECT product_name FROM products WHERE product_name LIKE 'Phone%';

-- Find products ending with 'Phone'
SELECT product_name FROM products WHERE product_name LIKE '%Phone';
```

**Expected Output (for `'%phone%'`):**

| product_name       |
|--------------------|
| Smartphone X       |
| Phone Case         |
| Bluetooth Headphone |

**Explanation:** `'%phone%'` matches any product name containing "phone" anywhere. `'Phone%'` matches only names starting with "Phone". `'%Phone'` matches only names ending with "Phone".

**Real-World Cases:**

- **Search functionality:** Implementing "contains" search in applications.
- **Email domain filtering:** `WHERE email LIKE '%@gmail.com'`.
- **Product categorization:** Finding products by partial name.

**References:**

- PostgreSQL: LIKE - https://www.postgresql.org/docs/current/functions-matching.html
- Microsoft Learn: Wildcard Characters - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/wildcard-character-s-to-match-transact-sql


#### 2.2 The _ Wildcard (Matching Exactly One Character)

**Core Definitions:**

- **Core Definition:** The `_` wildcard matches exactly one character.
- **Technical Definition:** In the LIKE predicate, `_` matches any single character. It always matches exactly one character, independent of collation.
- **Beginner-Friendly Explanation:** `_` is like a single-character placeholder—it stands for exactly one character, no more, no less.

**Purposes:**

- To match a specific number of characters in a pattern
- To match variable content at a specific position
- To validate fixed-length formats with variable characters

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
pattern LIKE '_' || value || '_'
```

**Component Breakdown:**

- `_`: Wildcard representing exactly one character
- Multiple `_` characters match multiple characters

**Syntax Rules:**

- `_` matches exactly one character.
- `'___'` matches exactly three characters.
- `_` does not match zero characters or multiple characters.

**Constraints and Limitations:**

- `_` cannot match partial characters in multibyte character sets.
- `_` at the beginning of a pattern may prevent index usage (depending on the database).

**Annotated Code Examples:**

```sql
-- Find employees whose first name is exactly 4 characters
SELECT first_name FROM employees WHERE first_name LIKE '____';

-- Find employees whose first name starts with 'J' and has exactly 3 more characters
SELECT first_name FROM employees WHERE first_name LIKE 'J___';

-- Find phone numbers matching pattern: 3 digits, dash, 3 digits
SELECT phone FROM employees WHERE phone LIKE '___-___-____';
```

**Expected Output:**

| Query | Result |
|-------|--------|
| `LIKE '____'` | John, Jane |
| `LIKE 'J___'` | John, Jane |
| `LIKE '___-___-____'` | All phone numbers |

**Explanation:** `'____'` matches exactly four characters. `'J___'` matches 'J' followed by exactly three characters. `'___-___-____'` matches the phone number format XXX-XXX-XXXX.

**Real-World Cases:**

- **Validation:** Checking that postal codes match a specific format.
- **Data cleaning:** Finding records with names of a specific length.
- **Format matching:** Validating fixed-length codes.

**References:**

- PostgreSQL: LIKE - https://www.postgresql.org/docs/current/functions-matching.html
- Microsoft Learn: Wildcard Characters - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/wildcard-character-s-to-match-transact-sql


#### 2.3 Shorthand Character Sets (SQL Server Bracket Expressions)

**Core Definitions:**

- **Core Definition:** SQL Server supports bracket expressions `[a-z]` and `[^0-9]` as additional wildcard syntax for matching character ranges or sets.
- **Technical Definition:** In SQL Server's LIKE predicate, `[ ]` matches any single character within the specified range (`[a-f]`) or set (`[abcdef]`), while `[^]` matches any single character not within the specified range or set.
- **Beginner-Friendly Explanation:** Bracket expressions let you match a character from a specific group—like "any letter from A to F" or "any digit."

**Purposes:**

- To match characters from a specific range (e.g., all letters, all digits)
- To exclude specific characters from matching
- To implement more precise pattern matching without full regex

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- Match any character in range/set
pattern LIKE '[a-f]%'
pattern LIKE '[abcdef]%'

-- Match any character NOT in range/set
pattern LIKE '[^a-f]%'
pattern LIKE '[^abcdef]%'
```

**Component Breakdown:**

- `[a-f]`: Matches any single character between 'a' and 'f' (inclusive)
- `[abcdef]`: Matches any single character in the set
- `[^a-f]`: Matches any single character NOT between 'a' and 'f'
- `[^abcdef]`: Matches any single character NOT in the set

**Syntax Rules:**

- Bracket expressions match exactly one character.
- Ranges are collation-dependent.
- `^` inside brackets negates the set.
- `]` can be included in a set with proper escaping (e.g., `[]]` in some contexts).

**Constraints and Limitations:**

- Bracket expressions are SQL Server-specific (not ANSI standard).
- SQL Server does not support native regular expressions.
- Performance depends on whether the pattern starts with a wildcard.

**Annotated Code Examples:**

```sql
-- SQL Server: Find authors whose last name starts with C through P
SELECT au_lname FROM authors WHERE au_lname LIKE '[C-P]%';

-- SQL Server: Find authors whose last name does NOT start with a vowel
SELECT au_lname FROM authors WHERE au_lname LIKE '[^AEIOU]%';

-- SQL Server: Find phone numbers where the second digit is 5
SELECT phone FROM authors WHERE phone LIKE '_5%';
```

**Expected Output (for `'[C-P]%'`):**

| au_lname |
|----------|
| Carson   |
| Dullard  |
| Larsen   |
| O'Leary  |

**Explanation:** `'[C-P]%'` matches last names starting with any letter from C to P. Names starting with A, B, Q, or Z are excluded.

**Real-World Cases:**

- **Validation:** Finding records where a code starts with a letter in a specific range.
- **Data cleaning:** Identifying records with invalid characters.
- **Filtering:** Excluding records with names in a specific range.

**References:**

- Microsoft Learn: LIKE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/like-transact-sql
- Microsoft Learn: PATINDEX (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/patindex-transact-sql


### 3. Execution & Collation Mechanics

#### 3.1 Case Sensitivity Defaults Across Database Engines

**Core Definitions:**

- **Core Definition:** Case sensitivity in pattern matching determines whether `'ABC'` matches `'abc'`. It depends on the database engine and the collation of the column or expression.
- **Technical Definition:** Collation defines the rules for character comparison, including case sensitivity, accent sensitivity, and sort order. PostgreSQL's LIKE is case-sensitive regardless of locale. MySQL's LIKE is case-insensitive by default (using `utf8mb4_0900_ai_ci` or `utf8mb4_general_ci` collations). SQL Server follows the column or database collation, defaulting to case-insensitive (`SQL_Latin1_General_CP1_CI_AS`).
- **Beginner-Friendly Explanation:** Some databases treat uppercase and lowercase letters as the same (case-insensitive), while others treat them as different (case-sensitive). This affects whether searching for "smith" finds "Smith".

**Purposes:**

- To understand and predict matching behavior across database systems
- To choose the appropriate collation for a column
- To implement case-insensitive or case-sensitive searches as needed

**Syntax Structures and Rules:**

**Case Sensitivity Summary:**

| Database | LIKE Default | Case-Insensitive Option |
|----------|-------------|------------------------|
| PostgreSQL | Case-sensitive | `ILIKE` |
| MySQL | Case-insensitive (default collation) | `LIKE BINARY` for case-sensitive |
| SQL Server | Depends on collation (CI default) | `COLLATE` clause |
| Oracle | Case-sensitive | `UPPER()` or `LOWER()` functions |

**Component Breakdown:**

- Collation names follow the pattern: `character_set + language + case_sensitivity + accent_sensitivity`
- `_ci` suffix = case-insensitive; `_cs` suffix = case-sensitive
- `_ai` = accent-insensitive; `_as` = accent-sensitive

**Syntax Rules:**

- PostgreSQL: LIKE is always case-sensitive; ILIKE is case-insensitive.
- MySQL: LIKE uses the column's collation; use `LIKE BINARY` for case-sensitive.
- SQL Server: LIKE uses the column's collation; use `COLLATE` to override.
- Oracle: LIKE is case-sensitive; use `UPPER()` or `LOWER()` for case-insensitive.

**Constraints and Limitations:**

- Changing collation can affect index usage.
- Case-insensitive matching may be slower than case-sensitive if not indexed properly.
- Collation differences can cause portability issues across databases.

**Annotated Code Examples:**

```sql
-- PostgreSQL: Case-sensitive vs case-insensitive
SELECT 'Hello' LIKE 'hello';       -- FALSE (case-sensitive)
SELECT 'Hello' ILIKE 'hello';      -- TRUE (case-insensitive)

-- MySQL: Case-insensitive by default
SELECT 'Hello' LIKE 'hello';       -- TRUE (default collation is _ci)
SELECT 'Hello' LIKE BINARY 'hello'; -- FALSE (explicit case-sensitive)

-- SQL Server: Depends on collation
SELECT 'Hello' LIKE 'hello';       -- TRUE (default CI collation)
SELECT 'Hello' COLLATE Latin1_General_CS_AS LIKE 'hello'; -- FALSE
```

**Expected Output (PostgreSQL):**

| Query | Result |
|-------|--------|
| `'Hello' LIKE 'hello'` | FALSE |
| `'Hello' ILIKE 'hello'` | TRUE |

**Explanation:** PostgreSQL's LIKE is case-sensitive, so 'Hello' does not match 'hello'. ILIKE performs case-insensitive matching, so it returns TRUE.

**Real-World Cases:**

- **User search:** Implementing case-insensitive search for usernames or product names.
- **Data migration:** Understanding collation differences when moving data between databases.
- **Validation:** Enforcing case-sensitive passwords or codes.

**References:**

- DigitalOcean: SQL LIKE and NOT LIKE Operators - https://www.digitalocean.com/community/tutorials/sql-like-not-like-operators
- PostgreSQL: Pattern Matching - https://www.postgresql.org/docs/current/functions-matching.html


#### 3.2 Case-Insensitive Matching Operators (ILIKE and LOWER() Wrappers)

**Core Definitions:**

- **Core Definition:** Case-insensitive matching can be achieved using the ILIKE operator (PostgreSQL), the LIKE BINARY operator (MySQL for case-sensitive), or by wrapping both sides in LOWER() or UPPER() functions.
- **Technical Definition:** ILIKE is a PostgreSQL extension that performs case-insensitive pattern matching according to the active locale. The LOWER()/UPPER() approach normalizes both operands to a single case before comparison, which works across all databases but may prevent index usage unless a functional index is created.
- **Beginner-Friendly Explanation:** If you want "Smith" and "smith" to both match when searching, you can either use ILIKE (in PostgreSQL) or convert everything to lowercase before comparing.

**Purposes:**

- To perform case-insensitive pattern matching
- To implement user-friendly search functionality
- To normalize data before comparison

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
-- PostgreSQL ILIKE
column ILIKE pattern

-- LOWER()/UPPER() wrapper (works in all databases)
LOWER(column) LIKE LOWER(pattern)
UPPER(column) LIKE UPPER(pattern)
```

**Component Breakdown:**

- `ILIKE`: PostgreSQL-specific case-insensitive LIKE
- `LOWER()`/`UPPER()`: Standard SQL functions that convert strings to lowercase/uppercase

**Syntax Rules:**

- ILIKE is not ANSI standard and is PostgreSQL-specific.
- LOWER()/UPPER() work in all major databases.
- Using functions on columns in WHERE clauses prevents standard index usage unless a functional index exists.

**Constraints and Limitations:**

- ILIKE does not support nondeterministic collations.
- LOWER()/UPPER() may not handle all Unicode characters correctly in all databases.
- Performance may degrade with function-based comparisons on large tables.

**Annotated Code Examples:**

```sql
-- PostgreSQL: ILIKE example
SELECT first_name, last_name FROM employees WHERE last_name ILIKE 'smith%';

-- All databases: LOWER() wrapper
SELECT first_name, last_name FROM employees WHERE LOWER(last_name) LIKE LOWER('smith%');

-- MySQL: Using collation for case-insensitive matching
SELECT * FROM employees WHERE last_name LIKE 'smith%' COLLATE utf8mb4_general_ci;
```

**Expected Output (all return Smith):**

| first_name | last_name |
|------------|-----------|
| John       | Smith     |

**Explanation:** All three approaches find employees with the last name "Smith" regardless of case.

**Real-World Cases:**

- **User search:** Allowing users to search for products or contacts without worrying about capitalization.
- **Data deduplication:** Finding records that differ only in case.
- **Login systems:** Case-insensitive username matching.

**References:**

- PostgreSQL: Pattern Matching - https://www.postgresql.org/docs/current/functions-matching.html
- Baeldung: How to Ignore Case While Searching for a String in SQL - https://www.baeldung.com/sql-ignore-case-search


#### 3.3 Escaping Wildcard Characters (ESCAPE Clause)

**Core Definitions:**

- **Core Definition:** The ESCAPE clause allows you to search for literal `%` and `_` characters by specifying an escape character.
- **Technical Definition:** To match a literal underscore or percent sign without matching other characters, the respective character in the pattern must be preceded by the escape character. The default escape character is the backslash in some databases, but the SQL standard allows any single character to be specified using the ESCAPE clause.
- **Beginner-Friendly Explanation:** If you want to search for an actual percent sign (like in "50% off"), you need to tell the database "this % is just a regular character, not a wildcard." The ESCAPE clause does this.

**Purposes:**

- To search for literal `%` and `_` characters in data
- To use wildcards as regular characters in patterns
- To handle data that contains special pattern characters

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
match_expression LIKE pattern ESCAPE escape_character
```

**Component Breakdown:**

- `ESCAPE escape_character`: Specifies a character that, when placed before `%` or `_`, makes them literal
- Common escape characters: `\`, `!`, `#`, `$`

**Syntax Rules:**

- The escape character must be a single character.
- To match the escape character itself, write it twice.
- If no escape character is specified, the SQL standard says there is no escape character (rather than defaulting to backslash).
- PostgreSQL defaults to backslash, which is slightly nonstandard.

**Constraints and Limitations:**

- Some databases have restrictions on the escape character.
- Backslash has special meaning in string literals, requiring doubling in some contexts.

**Annotated Code Examples:**

```sql
-- Setup
CREATE TABLE discounts (
    id INT PRIMARY KEY,
    description VARCHAR(200)
);

INSERT INTO discounts VALUES (1, 'Discount is 10-15% off');
INSERT INTO discounts VALUES (2, 'Discount is .10-.15 off');
INSERT INTO discounts VALUES (3, 'Save 50% on all items');

-- Find records containing a literal '%' character
SELECT description FROM discounts WHERE description LIKE '%!%%' ESCAPE '!';
```

**Expected Output:**

| description                |
|----------------------------|
| Discount is 10-15% off     |
| Save 50% on all items      |

**Explanation:** The pattern `'%!%%'` uses `!` as the escape character. `!%` means a literal percent sign. The pattern reads: any characters (`%`), followed by a literal `%` (`!%`), followed by any characters (`%`).

```sql
-- Find records containing a literal '_' character
SELECT description FROM table_name WHERE column LIKE '%!_%' ESCAPE '!';
```

**Real-World Cases:**

- **Data cleaning:** Finding records that contain literal percent signs (e.g., discount percentages).
- **Log analysis:** Searching for patterns that include special characters.
- **User input:** Allowing users to search for literal wildcard characters.

**References:**

- PostgreSQL: LIKE - https://www.postgresql.org/docs/current/functions-matching.html
- Microsoft Learn: LIKE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/like-transact-sql
- Oracle: LIKE Condition - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/LIKE.html


### 4. Advanced & Regular Expression Matching

#### 4.1 The ANSI Standard SIMILAR TO Operator

**Core Definitions:**

- **Core Definition:** SIMILAR TO is an ANSI SQL standard operator that performs pattern matching using SQL99's definition of regular expressions.
- **Technical Definition:** The SIMILAR TO operator returns true or false depending on whether its pattern matches the given string. It is similar to LIKE, except that it interprets the pattern using the SQL standard's definition of a regular expression. Like LIKE, the SIMILAR TO operator succeeds only if its pattern matches the entire string.
- **Beginner-Friendly Explanation:** SIMILAR TO is like LIKE with superpowers—it supports regular expression features like alternation (`|`), repetition (`*`, `+`), and grouping (`()`).

**Purposes:**

- To perform more complex pattern matching than LIKE allows
- To use regular expression features within the SQL standard
- To validate structured formats (e.g., dates, codes)

**Syntax Structures and Rules:**

**Complete General Syntax:**

```sql
string SIMILAR TO pattern [ESCAPE escape_character]
```

**Component Breakdown:**

- `string`: The string to test
- `pattern`: A SQL99 regular expression pattern
- `|`: Alternation (either/or)
- `*`: Repetition zero or more times
- `+`: Repetition one or more times
- `()`: Grouping
- `[...]`: Character class
- `%` and `_`: LIKE wildcards (also supported)

**Syntax Rules:**

- Pattern must match the entire string (like LIKE).
- Bounded repetition (`{n,m}`) is not supported.
- Dot (`.`) is not a metacharacter.
- Backslash disables special meaning of metacharacters.

**Constraints and Limitations:**

- SIMILAR TO is supported by PostgreSQL, but not by all databases (e.g., SQL Server does not support it).
- It is less powerful than POSIX regular expressions.
- Some databases implement it differently.

**Annotated Code Examples:**

```sql
-- PostgreSQL: SIMILAR TO examples
SELECT 'abc' SIMILAR TO 'abc';        -- TRUE
SELECT 'abc' SIMILAR TO 'a';          -- FALSE
SELECT 'abc' SIMILAR TO '%(b|d)%';    -- TRUE
SELECT 'abc' SIMILAR TO '(b|c)%';     -- FALSE

-- Validate a date format (YYYY-MM-DD)
SELECT '2024-01-15' SIMILAR TO '[0-9]{4}-[0-9]{2}-[0-9]{2}'; -- Not supported (bounded repetition)
-- Use: '[0-9][0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9]'
SELECT '2024-01-15' SIMILAR TO '[0-9][0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9]'; -- TRUE
```

**Expected Output:**

| Query | Result |
|-------|--------|
| `'abc' SIMILAR TO 'abc'` | TRUE |
| `'abc' SIMILAR TO 'a'` | FALSE |
| `'abc' SIMILAR TO '%(b|d)%'` | TRUE |
| `'abc' SIMILAR TO '(b|c)%'` | FALSE |

**Explanation:** `'%'` matches any string. `'(b|d)'` matches either 'b' or 'd'. Since 'abc' contains 'b', it matches. `'(b|c)%'` requires the string to start with 'b' or 'c', but 'abc' starts with 'a', so it returns FALSE.

**Real-World Cases:**

- **Data validation:** Validating that codes follow a specific pattern.
- **Format checking:** Ensuring dates, phone numbers, or IDs match expected formats.
- **Text processing:** Extracting or matching complex patterns.

**References:**

- PostgreSQL: SIMILAR TO Regular Expressions - https://www.postgresql.org/docs/current/functions-matching.html
- PostgreSQL: Pattern Matching - https://www.postgresql.org/docs/current/functions-matching.html


#### 4.2 Native Regular Expression Operators

**Core Definitions:**

- **Core Definition:** Native regex operators provide POSIX regular expression matching directly in SQL, with vendor-specific syntax.
- **Technical Definition:** PostgreSQL uses `~` (case-sensitive match), `~*` (case-insensitive match), `!~` (case-sensitive non-match), and `!~*` (case-insensitive non-match). MySQL uses `REGEXP` or `RLIKE` operators. Oracle uses `REGEXP_LIKE` as a function in WHERE clauses.
- **Beginner-Friendly Explanation:** These operators let you use full regular expressions—a powerful pattern language—directly in your SQL queries.

**Purposes:**

- To perform complex pattern matching beyond LIKE and SIMILAR TO
- To validate data against sophisticated patterns
- To extract and manipulate text using regex features

**Syntax Structures and Rules:**

**PostgreSQL:**

```sql
string ~ pattern        -- Case-sensitive match
string ~* pattern       -- Case-insensitive match
string !~ pattern       -- Case-sensitive non-match
string !~* pattern      -- Case-insensitive non-match
```

**MySQL:**

```sql
string REGEXP pattern   -- Match (case-insensitive by default)
string RLIKE pattern    -- Same as REGEXP
string NOT REGEXP pattern -- Non-match
```

**Oracle:**

```sql
REGEXP_LIKE(source_string, pattern [, match_parameter])
```

**Component Breakdown:**

- `~`: PostgreSQL match operator
- `~*`: PostgreSQL case-insensitive match
- `REGEXP`/`RLIKE`: MySQL match operators
- `REGEXP_LIKE`: Oracle function

**Syntax Rules:**

- PostgreSQL regex patterns are POSIX regular expressions.
- MySQL uses ICU regular expressions (since 8.0) with a syntax similar to POSIX.
- Oracle regex supports POSIX and some Perl extensions.
- Match parameters: `i` (case-insensitive), `c` (case-sensitive), `n` (dot matches newline), `m` (multiline).

**Constraints and Limitations:**

- Regex syntax varies across databases.
- Regex matching can be slower than LIKE.
- Some databases (SQL Server) lack native regex support.

**Annotated Code Examples:**

```sql
-- PostgreSQL: Find names containing 'john' (case-insensitive)
SELECT first_name FROM employees WHERE first_name ~* 'john';

-- PostgreSQL: Find names starting with 'J' or 'j'
SELECT first_name FROM employees WHERE first_name ~ '^[Jj]';

-- MySQL: Find names containing 'john'
SELECT first_name FROM employees WHERE first_name REGEXP 'john';

-- Oracle: Find employees with valid email addresses
SELECT email FROM employees WHERE REGEXP_LIKE(email, '^\w+@\w+\.\w+$');
```

**Expected Output (PostgreSQL):**

| Query | Result |
|-------|--------|
| `~* 'john'` | John, Johnny, JOHN |
| `~ '^[Jj]'` | John, Jane, James |

**Explanation:** `~*` performs case-insensitive matching, so 'john' matches 'John' and 'JOHN'. `~` is case-sensitive; `'^[Jj]'` matches names starting with uppercase or lowercase J.

**Real-World Cases:**

- **Email validation:** Checking that email addresses match a valid pattern.
- **Data extraction:** Finding and extracting specific parts of strings.
- **Log parsing:** Matching complex patterns in log entries.

**References:**

- PostgreSQL: POSIX Regular Expressions - https://www.postgresql.org/docs/current/functions-matching.html
- MySQL: Regular Expressions - https://dev.mysql.com/doc/refman/8.0/en/regexp.html
- Oracle: Regular Expression Support - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Regular-Expression-Support.html


#### 4.3 Regex Functions for Extracting or Replacing Text

**Core Definitions:**

- **Core Definition:** Regex functions provide the ability to extract substrings matching a pattern, replace matched text, and find pattern positions within strings.
- **Technical Definition:** Oracle provides `REGEXP_SUBSTR` (extract matching substring), `REGEXP_REPLACE` (replace matched text), `REGEXP_INSTR` (find position), and `REGEXP_COUNT` (count occurrences). MySQL provides similar functions: `REGEXP_SUBSTR`, `REGEXP_REPLACE`, `REGEXP_INSTR`, `REGEXP_LIKE`. PostgreSQL provides `regexp_match()`, `regexp_matches()`, `regexp_replace()`, and `regexp_instr()`.
- **Beginner-Friendly Explanation:** These functions let you not just find patterns, but also extract specific parts of a string that match a pattern, or replace matched text with something else.

**Purposes:**

- To extract specific information from structured text
- To clean and transform data using pattern-based rules
- To find the position of patterns within strings
- To count occurrences of patterns

**Syntax Structures and Rules:**

**Oracle:**

```sql
REGEXP_SUBSTR(source, pattern [, position [, occurrence [, match_param]]])
REGEXP_REPLACE(source, pattern, replacement [, position [, occurrence [, match_param]]])
REGEXP_INSTR(source, pattern [, position [, occurrence [, return_option [, match_param]]]])
REGEXP_COUNT(source, pattern [, position [, match_param]])
```

**MySQL:**

```sql
REGEXP_SUBSTR(expr, pat [, pos [, occurrence [, match_type]]])
REGEXP_REPLACE(expr, pat, repl [, pos [, occurrence [, match_type]]])
REGEXP_INSTR(expr, pat [, pos [, occurrence [, return_option [, match_type]]]])
```

**PostgreSQL:**

```sql
regexp_match(string, pattern [, flags])
regexp_matches(string, pattern [, flags])
regexp_replace(string, pattern, replacement [, flags])
regexp_instr(string, pattern [, start [, N [, flags]]])
```

**Component Breakdown:**

- `source`/`string`/`expr`: The input string
- `pattern`: The regular expression
- `replacement`/`repl`: The replacement text
- `position`/`pos`: Starting position for search
- `occurrence`: Which match to return (1st, 2nd, etc.)
- `match_param`/`match_type`: Flags like 'i' (case-insensitive), 'c' (case-sensitive)

**Syntax Rules:**

- Position is 1-based in Oracle and MySQL; PostgreSQL's `regexp_instr` is also 1-based by default.
- Occurrence specifies which match to return when multiple matches exist.
- Match parameters control case sensitivity and other behaviors.

**Constraints and Limitations:**

- Function names and parameter orders vary across databases.
- Performance may be slower than simple string functions.
- Some functions return NULL if no match is found.

**Annotated Code Examples:**

```sql
-- Oracle: Extract area code from phone number
SELECT REGEXP_SUBSTR('415-555-0100', '^[0-9]+') FROM dual;
-- Returns: 415

-- Oracle: Replace all digits with '#'
SELECT REGEXP_REPLACE('Phone: 415-555-0100', '[0-9]', '#') FROM dual;
-- Returns: Phone: ###-###-####

-- MySQL: Extract domain from email
SELECT REGEXP_SUBSTR('user@example.com', '@(.+)$') FROM dual;
-- Returns: @example.com

-- MySQL: Replace spaces with underscores
SELECT REGEXP_REPLACE('Hello World', ' ', '_');
-- Returns: Hello_World

-- PostgreSQL: Extract all numbers from a string
SELECT regexp_matches('abc123def456', '[0-9]+', 'g');
-- Returns: {123}, {456}
```

**Expected Output:**

| Function | Input | Output |
|----------|-------|--------|
| REGEXP_SUBSTR | '415-555-0100', '^[0-9]+' | '415' |
| REGEXP_REPLACE | 'Phone: 415-555-0100', '[0-9]', '#' | 'Phone: ###-###-####' |
| REGEXP_SUBSTR | 'user@example.com', '@(.+)$' | '@example.com' |
| regexp_matches | 'abc123def456', '[0-9]+', 'g' | '{123}', '{456}' |

**Real-World Cases:**

- **Data extraction:** Extracting phone numbers, emails, or IDs from unstructured text.
- **Data cleaning:** Removing or replacing unwanted characters.
- **Log analysis:** Parsing log entries to extract timestamps or error codes.
- **Data transformation:** Converting data from one format to another using pattern-based rules.

**References:**

- Oracle: REGEXP_SUBSTR - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/REGEXP_SUBSTR.html
- Oracle: REGEXP_REPLACE - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/REGEXP_REPLACE.html
- MySQL: Regular Expressions - https://dev.mysql.com/doc/refman/8.0/en/regexp.html
- PostgreSQL: Regular Expression Functions - https://www.postgresql.org/docs/current/functions-matching.html


### 5. Performance & Optimization

#### 5.1 Performance Implications of Wildcards (Leading Wildcards and Index Usage)

**Core Definitions:**

- **Core Definition:** Leading wildcards (`'%abc'`) prevent standard B-tree indexes from being used because the index is ordered by the first character of the string.
- **Technical Definition:** A B-tree index can only be used for pattern matching when the pattern has a known prefix (e.g., `'abc%'`). When the pattern starts with `%` or `_`, the database cannot determine a starting point in the index and must perform a full table scan. Oracle explicitly states that if the pattern's first character is `%` or `_`, the optimizer cannot use the index and performance will not improve.
- **Beginner-Friendly Explanation:** An index is like a phone book sorted by last name. If you search for names starting with "Smith", you can jump right to the S section. But if you search for names containing "mith" anywhere, you have to scan the entire phone book.

**Purposes:**

- To understand why certain pattern queries are slow
- To design queries and indexes that perform well
- To choose appropriate indexing strategies for pattern matching

**Syntax Structures and Rules:**

**Index-Friendly Patterns:**

```sql
-- Uses index (known prefix)
WHERE column LIKE 'abc%'

-- Does NOT use index (no known prefix)
WHERE column LIKE '%abc'
WHERE column LIKE '%abc%'
WHERE column LIKE '_abc'
```

**Component Breakdown:**

- `'abc%'`: Known prefix 'abc' allows index seek
- `'%abc'`: Unknown prefix forces full scan
- `'_abc'`: Unknown first character forces full scan

**Syntax Rules:**

- The database can use an index for LIKE only if the pattern has a known, non-wildcard prefix.
- Leading `%` or `_` prevents index usage.
- Trailing `%` (e.g., `'abc%'`) allows index usage.

**Constraints and Limitations:**

- Even with a known prefix, the database may choose a full scan if the optimizer estimates that a large percentage of rows will match.
- Function-based indexes (e.g., on `LOWER(column)`) can help with case-insensitive searches.
- Some databases have special index types for pattern matching (trigram, full-text).

**Annotated Code Examples:**

```sql
-- Setup: Create table with index
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(200)
);

CREATE INDEX idx_product_name ON products(product_name);

-- Index-friendly query (uses index)
SELECT * FROM products WHERE product_name LIKE 'Widget%';

-- Index-unfriendly query (full table scan)
SELECT * FROM products WHERE product_name LIKE '%Widget%';
```

**Expected Performance:**

| Query Pattern | Index Usage | Performance |
|---------------|-------------|-------------|
| `'Widget%'` | Index seek | Fast (milliseconds) |
| `'%Widget%'` | Full scan | Slow (seconds on large tables) |

**Explanation:** The first query can use the index because it has a known prefix. The second query must scan every row because the match could occur anywhere in the string.

**Real-World Cases:**

- **E-commerce search:** Implementing "starts with" search for better performance vs. "contains" search.
- **Autocomplete:** Using prefix matching (`'abc%'`) for fast autocomplete.
- **Log search:** Understanding why searching for a substring in logs is slow.

**References:**

- Oracle: Pattern Matching on Indexed Columns - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/LIKE.html
- SQLPerformance: One Way to Get an Index Seek for a Leading %Wildcard - https://sqlperformance.com/2017/02/sql-indexes/leading-wildcard-index-seek


#### 5.2 Text Indexing for Patterns (Trigram Indexes and Full-Text Search)

**Core Definitions:**

- **Core Definition:** Trigram indexes and full-text search provide alternative indexing strategies for pattern matching that cannot use standard B-tree indexes.
- **Technical Definition:** A trigram index (e.g., PostgreSQL's `pg_trgm` extension) indexes overlapping three-character substrings of text, enabling fast `LIKE '%value%'` and `ILIKE '%value%'` queries without changing query syntax. Full-text search uses inverted indexes for word-level matching, suitable for natural-language text.
- **Beginner-Friendly Explanation:** Trigram indexes are like a phone book that indexes every three-letter combination in every name, so you can quickly find names containing a specific sequence of letters anywhere.

**Purposes:**

- To accelerate leading-wildcard pattern matching
- To enable efficient substring search on large text columns
- To provide word-level relevance ranking for search applications

**Syntax Structures and Rules:**

**PostgreSQL Trigram Index:**

```sql
-- Enable extension
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- Create trigram index
CREATE INDEX idx_column_trgm ON table_name USING gin (column_name gin_trgm_ops);

-- Query (automatically uses trigram index)
SELECT * FROM table_name WHERE column_name ILIKE '%value%';
```

**PostgreSQL Full-Text Search:**

```sql
-- Create tsvector column and GIN index
ALTER TABLE table_name ADD COLUMN tsv tsvector;
UPDATE table_name SET tsv = to_tsvector('english', text_column);
CREATE INDEX idx_tsv ON table_name USING gin(tsv);

-- Query
SELECT * FROM table_name WHERE tsv @@ to_tsquery('english', 'word');
```

**MySQL Full-Text Search:**

```sql
-- Create FULLTEXT index
ALTER TABLE table_name ADD FULLTEXT(column_name);

-- Query
SELECT * FROM table_name WHERE MATCH(column_name) AGAINST('word');
```

**Component Breakdown:**

- `pg_trgm`: PostgreSQL extension for trigram-based similarity and pattern matching
- `gin_trgm_ops`: GIN index operator class for trigram indexes
- `FULLTEXT`: MySQL index type for full-text search
- `tsvector`/`tsquery`: PostgreSQL full-text search data types

**Syntax Rules:**

- Trigram indexes work with `LIKE`, `ILIKE`, `~`, and `~*` queries.
- Full-text search requires a FULLTEXT index (MySQL) or tsvector column with GIN index (PostgreSQL).
- Trigram indexes add overhead to INSERT and UPDATE operations.

**Constraints and Limitations:**

- Trigram indexes are most effective for patterns of 3 or more characters.
- Full-text search does not support arbitrary substring matching.
- Index size can be significant for large text columns.

**Annotated Code Examples:**

```sql
-- PostgreSQL: Trigram index for fast substring search
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX idx_customers_name_trgm ON customers USING gin (customer_name gin_trgm_ops);

-- This query uses the trigram index automatically
SELECT customer_name FROM customers WHERE customer_name ILIKE '%mit%';

-- MySQL: Full-text search
ALTER TABLE customers ADD FULLTEXT(customer_name);
SELECT customer_name FROM customers WHERE MATCH(customer_name) AGAINST('Amit');
```

**Expected Performance:**

| Method | Query Pattern | Performance on 10M rows |
|--------|--------------|------------------------|
| No index | `LIKE '%value%'` | 1-3 seconds |
| Trigram index | `LIKE '%value%'` | Tens of milliseconds |
| Full-text index | `MATCH ... AGAINST` | Milliseconds |

**Explanation:** The trigram index dramatically accelerates leading-wildcard queries by pre-computing all three-character substrings. Full-text search is even faster for word-level matching but does not support arbitrary substrings.

**Real-World Cases:**

- **E-commerce search:** Implementing fast "contains" search on product names and descriptions.
- **Log analysis:** Searching large log files for specific error patterns.
- **Content management:** Full-text search across articles and documents.
- **Customer support:** Searching knowledge base articles.

**References:**

- PostgreSQL: pg_trgm - https://www.postgresql.org/docs/current/pgtrgm.html
- PostgreSQL: Full-Text Search - https://www.postgresql.org/docs/current/textsearch.html
- MySQL: Full-Text Search - https://dev.mysql.com/doc/refman/8.0/en/fulltext-search.html


## Summary Tables

### Pattern Matching Operators Across Databases

| Feature | PostgreSQL | MySQL | SQL Server | Oracle |
|---------|-----------|-------|------------|--------|
| Basic LIKE | ✓ | ✓ | ✓ | ✓ |
| NOT LIKE | ✓ | ✓ | ✓ | ✓ |
| ILIKE (case-insensitive) | ✓ | ✗ | ✗ | ✗ |
| `%` wildcard | ✓ | ✓ | ✓ | ✓ |
| `_` wildcard | ✓ | ✓ | ✓ | ✓ |
| `[a-z]` bracket expressions | ✗ | ✗ | ✓ | ✗ |
| ESCAPE clause | ✓ | ✓ | ✓ | ✓ |
| SIMILAR TO | ✓ | ✗ | ✗ | ✗ |
| Native regex operator | `~`, `~*` | `REGEXP`/`RLIKE` | ✗ | ✗ |
| Regex function | `regexp_match()` | `REGEXP_LIKE()` | ✗ | `REGEXP_LIKE()` |
| Regex extract | `regexp_match()` | `REGEXP_SUBSTR()` | ✗ | `REGEXP_SUBSTR()` |
| Regex replace | `regexp_replace()` | `REGEXP_REPLACE()` | ✗ | `REGEXP_REPLACE()` |

### Case Sensitivity Defaults

| Database | LIKE Default | Case-Insensitive Option |
|----------|-------------|------------------------|
| PostgreSQL | Case-sensitive | `ILIKE` |
| MySQL | Case-insensitive (default collation) | `LIKE BINARY` for case-sensitive |
| SQL Server | Depends on collation (CI default) | `COLLATE` clause |
| Oracle | Case-sensitive | `UPPER()` or `LOWER()` |

### Index Usage for Pattern Matching

| Pattern | Index Usage | Notes |
|---------|-------------|-------|
| `'abc%'` | ✓ (B-tree) | Known prefix |
| `'%abc'` | ✗ | No known prefix; use trigram |
| `'%abc%'` | ✗ | No known prefix; use trigram or full-text |
| `'_abc'` | ✗ | No known prefix |
| `'abc_def'` | ✓ (B-tree) | Known prefix |

---

## References

- PostgreSQL: Pattern Matching - https://www.postgresql.org/docs/current/functions-matching.html
- PostgreSQL: LIKE - https://www.postgresql.org/docs/current/functions-matching.html
- PostgreSQL: SIMILAR TO Regular Expressions - https://www.postgresql.org/docs/current/functions-matching.html
- PostgreSQL: POSIX Regular Expressions - https://www.postgresql.org/docs/current/functions-matching.html
- PostgreSQL: pg_trgm - https://www.postgresql.org/docs/current/pgtrgm.html
- PostgreSQL: Full-Text Search - https://www.postgresql.org/docs/current/textsearch.html
- Microsoft Learn: LIKE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/like-transact-sql
- Microsoft Learn: Wildcard Characters - https://learn.microsoft.com/en-us/sql/t-sql/language-elements/wildcard-character-s-to-match-transact-sql
- Microsoft Learn: PATINDEX (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/patindex-transact-sql
- Oracle: LIKE Condition - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/LIKE.html
- Oracle: REGEXP_SUBSTR - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/REGEXP_SUBSTR.html
- Oracle: REGEXP_REPLACE - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/REGEXP_REPLACE.html
- Oracle: Regular Expression Support - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Regular-Expression-Support.html
- MySQL: Regular Expressions - https://dev.mysql.com/doc/refman/8.0/en/regexp.html
- MySQL: Full-Text Search - https://dev.mysql.com/doc/refman/8.0/en/fulltext-search.html
- DigitalOcean: SQL LIKE and NOT LIKE Operators - https://www.digitalocean.com/community/tutorials/sql-like-not-like-operators
- Baeldung: How to Ignore Case While Searching for a String in SQL - https://www.baeldung.com/sql-ignore-case-search
- SQLPerformance: One Way to Get an Index Seek for a Leading %Wildcard - https://sqlperformance.com/2017/02/sql-indexes/leading-wildcard-index-seek