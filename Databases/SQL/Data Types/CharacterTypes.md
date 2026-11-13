# Comprehensive Programming Cheat Sheet: SQL Character Types

## Topic Overview

### Definitions

**Core Definition**
Character types are the SQL data types used to store text and string data—fixed-length sequences (`CHAR`, `CHARACTER`), variable-length sequences (`VARCHAR`, `CHARACTER VARYING`), and unbounded text (`TEXT`)—each with distinct storage, padding, and performance characteristics.

**Technical Definition**
SQL character types fall into two broad categories: **fixed-length types** (`CHAR(n)`, `CHARACTER(n)`) store every value padded with spaces to exactly `n` characters, while **variable-length types** (`VARCHAR(n)`, `CHARACTER VARYING(n)`, `TEXT`) store only the actual characters plus a small length overhead. In PostgreSQL, `character(n)` and `char(n)` are the same type; `character varying(n)` and `varchar(n)` are the same type; `text` is an unbounded variable-length type.【6†L52-L55】 MySQL distinguishes `CHAR` (trailing spaces stripped on retrieval, padded on storage) from `VARCHAR` (length prefix, no padding) and `TEXT`/`BLOB` families (off-row storage).【9†L38-L51】 SQL Server provides `char`, `nchar`, `varchar`, and `nvarchar`, with the `n` prefix indicating Unicode storage.【10†L111-L113】 The choice among these types directly affects storage consumption, comparison semantics (padding vs. no padding), index size, and performance.

**Beginner-Friendly Explanation**
Character types are the text containers of a database. `CHAR` is a fixed-size box: every value is padded with spaces to fill the box exactly. `VARCHAR` is a stretchy bag: it holds only as much text as you put in, plus a tiny label that records the length. `TEXT` is an unbounded bag that can hold as much text as you need. Choosing the right type is like choosing the right container: a fixed-size box wastes space if your strings are short, but it is predictable; a stretchy bag is efficient but adds a little overhead; and an unbounded bag is flexible but not ideal for indexing or sorting.

### Key Characteristics

- **Padding semantics differ**: `CHAR(n)` pads values with spaces to length `n` on storage and (in standard SQL) preserves trailing spaces; comparisons ignore trailing spaces in most systems. `VARCHAR(n)` stores values without padding and preserves trailing spaces.【1†L66-L68】
- **Length limits are vendor-specific**: PostgreSQL's `varchar(n)` and `char(n)` accept a length limit; `text` has no limit. MySQL's `CHAR` is 0–255 characters, `VARCHAR` up to 65,535 bytes (subject to row size), and `TEXT` up to 4 GB. SQL Server's `char`/`varchar` are limited to 8,000 bytes, with `varchar(max)` for larger values.【9†L38-L51】【10†L111-L113】
- **Unicode requires explicit types in MySQL and SQL Server**: `NCHAR`/`NVARCHAR` store Unicode data (UTF-16 in SQL Server, UTF-8 in some MySQL configurations); PostgreSQL uses the database encoding (typically UTF-8) for all character types.【10†L111-L113】
- **TEXT is not always a distinct type**: In PostgreSQL, `text` is a first-class type with no performance penalty compared with `varchar`; in MySQL and SQL Server, `TEXT`/`varchar(max)` may use off-row storage, which has performance implications.【8†L62-L66】
- **Best practice is "smallest that fits"**: Use `VARCHAR(n)` with a sensible `n`, reserve `CHAR(n)` for truly fixed-length codes, and use `TEXT` only when the length is genuinely unbounded.

### Prerequisites

- A running RDBMS instance (PostgreSQL, MySQL, or SQL Server) and a client connection.
- Familiarity with `CREATE TABLE` syntax and column definitions.
- Understanding of the application's text data: maximum lengths, whether Unicode is required, and whether values are fixed-length codes or free-form text.

### Related Programming Areas

- Data Definition Language (DDL) – column type declarations.
- Data integrity – truncation, padding, and comparison semantics.
- Query performance – index size and sort behaviour depend on the character type.
- Storage engineering – fixed vs. variable-length types affect row size and page density.
- Application mapping – SQL character types correspond to language-level types such as `String`, `str`, and `nvarchar`.

---

## Core Concepts and Features

### 1. `CHAR`

#### Definitions

**Core Definition**
`CHAR(n)` is a fixed-length character type that stores exactly `n` characters, padding shorter values with spaces.

**Technical Definition**
`CHAR(n)` (also written `CHARACTER(n)`) allocates `n` characters of storage for every value. If a value is shorter than `n`, it is padded with spaces on storage; if it is longer than `n`, it is truncated (or rejected, depending on the RDBMS and SQL mode). In PostgreSQL, `char(n)` is the same as `character(n)`, and `char` without a length is equivalent to `char(1)`.【6†L52-L55】 MySQL documents that `CHAR` values are right-padded with spaces to the specified length when stored, and trailing spaces are removed on retrieval (unless `PAD_CHAR_TO_FULL_LENGTH` SQL mode is enabled).【9†L45-L51】 Comparisons in MySQL are performed without regard to trailing spaces for `CHAR` columns.

**Beginner-Friendly Explanation**
`CHAR(n)` is a fixed-size box for text. If you put a three-letter word into a `CHAR(10)` column, the database fills the remaining seven spaces with blanks. Every value takes exactly 10 characters of space, no matter how short it is.

#### Purposes

- To store values that are genuinely fixed-length, such as two-letter country codes, three-letter currency codes, or fixed-length hash digests.
- To provide predictable storage allocation for rows in fixed-width formats.
- To enforce a strict maximum length at the schema level.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name CHAR(n)
-- or
column_name CHARACTER(n)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `n` | The fixed length in characters. If omitted, defaults to 1 in PostgreSQL and MySQL. |
| `CHAR` / `CHARACTER` | Synonymous keywords for the fixed-length type. |

**Syntax rules:**

- The length `n` must be a positive integer; `n` may be omitted in some systems, defaulting to 1.
- Values longer than `n` are truncated (MySQL, in non-strict mode) or raise an error (PostgreSQL, SQL Server).
- Trailing spaces are stripped on retrieval in MySQL; PostgreSQL preserves them on storage but ignores them in comparisons.

**Constraints and limitations:**

- `CHAR(n)` wastes storage when values are frequently shorter than `n`.
- `CHAR` is not suitable for variable-length data such as names or descriptions.
- In PostgreSQL, `char(n)` is a distinct type from `varchar(n)`, and mixing them in comparisons may require explicit casts.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – CHAR(2) for country codes

```sql
-- Step 1: Create a table with a CHAR(2) column.
CREATE TABLE countries (
    code CHAR(2) PRIMARY KEY,
    name TEXT NOT NULL
);

-- Step 2: Insert values; shorter values are padded with spaces.
INSERT INTO countries (code, name) VALUES ('US', 'United States');
INSERT INTO countries (code, name) VALUES ('CA', 'Canada');

-- Step 3: Query; the stored values are padded.
SELECT code, length(code) AS stored_length, name FROM countries ORDER BY code;
```

**Expected output:**

```text
 code | stored_length |      name
------+---------------+----------------
 CA   |             2 | Canada
 US   |             2 | United States
(2 rows)
```

**Why this output appears**: `CHAR(2)` stores exactly two characters per value. Both `'US'` and `'CA'` are already two characters, so no padding is added. If a one-character value were inserted, it would be padded to two characters.

##### Example 2: MySQL – CHAR with trailing space behaviour

```sql
-- Step 1: Create a table with CHAR(5).
CREATE TABLE codes (id INT PRIMARY KEY, code CHAR(5));

-- Step 2: Insert a value shorter than the declared length.
INSERT INTO codes (id, code) VALUES (1, 'AB');

-- Step 3: Query; trailing spaces are stripped on retrieval.
SELECT code, LENGTH(code) AS len FROM codes;
```

**Expected output:**

```text
+----+------+-----+
| id | code | len |
+----+------+-----+
|  1 | AB   |   2 |
+----+------+-----+
```

**Why this output appears**: MySQL right-pads `'AB'` with spaces to five characters on storage, but strips the trailing spaces when retrieving the value. `LENGTH(code)` returns 2, reflecting the retrieved value, not the stored padded value.

#### Real-World Cases

- **Country and currency codes**: ISO codes such as `US`, `GB`, and `USD` are naturally fixed-length and fit `CHAR(2)` or `CHAR(3)`.
- **Fixed-length hashes**: A SHA-256 hex digest is 64 characters; `CHAR(64)` ensures every row consumes the same storage for that column.

---

### 2. `CHARACTER`

#### Definitions

**Core Definition**
`CHARACTER` is the SQL standard keyword for the fixed-length character type, synonymous with `CHAR`.

**Technical Definition**
`CHARACTER(n)` and `CHAR(n)` are identical in PostgreSQL, MySQL, and SQL Server. PostgreSQL documents `char(n)` as a synonym for `character(n)` and `char` as equivalent to `character(1)`.【6†L52-L55】 The `CHARACTER` keyword is the spelling preferred by the SQL standard, while `CHAR` is the common shorthand.

**Beginner-Friendly Explanation**
`CHARACTER` is just another word for `CHAR`. It behaves exactly the same way: a fixed-length text box padded with spaces.

#### Purposes

- To provide the SQL-standard spelling for fixed-length character columns.
- To improve portability and readability in schemas intended to follow the standard closely.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name CHARACTER(n)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `CHARACTER(n)` | SQL-standard spelling of `CHAR(n)`. |

**Syntax rules:**

- Identical to `CHAR(n)` in every respect.
- `CHARACTER VARYING(n)` is the standard spelling of `VARCHAR(n)`.

**Constraints and limitations:**

- Some tools and developers prefer `CHAR` for brevity; the two spellings are interchangeable.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – CHARACTER is the same as CHAR

```sql
-- Step 1: Create a table using the standard spelling.
CREATE TABLE iso_codes (
    country CHARACTER(2) PRIMARY KEY,
    currency CHARACTER(3) NOT NULL
);

-- Step 2: Insert a row.
INSERT INTO iso_codes VALUES ('US', 'USD');

-- Step 3: Verify the column type.
SELECT column_name, data_type, character_maximum_length
FROM information_schema.columns
WHERE table_name = 'iso_codes';
```

**Expected output:**

```text
 column_name |     data_type     | character_maximum_length
-------------+-------------------+--------------------------
 country     | character         |                        2
 currency    | character         |                        3
(2 rows)
```

**Why this output appears**: The `CHARACTER(2)` and `CHARACTER(3)` declarations are stored as `character` types with the specified maximum lengths. The behaviour is identical to `CHAR(2)` and `CHAR(3)`.

#### Real-World Cases

- **Standards-compliant schemas**: A schema intended to be portable across RDBMSs uses `CHARACTER` and `CHARACTER VARYING` to align with the SQL standard.
- **Documentation**: Using the standard spelling makes the schema self-documenting for readers familiar with the SQL standard.

---

### 3. `VARCHAR`

#### Definitions

**Core Definition**
`VARCHAR(n)` is a variable-length character type that stores up to `n` characters, using only as much space as the actual value requires (plus a small length overhead).

**Technical Definition**
`VARCHAR(n)` (also written `CHARACTER VARYING(n)`) stores variable-length strings with a maximum length of `n` characters. In PostgreSQL, `varchar(n)` and `character varying(n)` are the same type; values longer than `n` are rejected with an error. Unlike `CHAR(n)`, `VARCHAR(n)` does not pad values with spaces. MySQL documents `VARCHAR` as storing a length prefix (1 or 2 bytes) followed by the data; trailing spaces are preserved on storage and retrieval. SQL Server's `varchar(n)` stores non-Unicode data in the database's code page, up to 8,000 bytes, with `varchar(max)` for larger values.

**Beginner-Friendly Explanation**
`VARCHAR(n)` is a stretchy text bag with a maximum size. You tell it the largest string you expect (e.g., 50 characters for a username), and it uses only as much space as each actual value needs, plus a tiny length marker. Shorter values do not waste space.

#### Purposes

- To store variable-length text such as names, emails, addresses, and titles efficiently.
- To enforce a maximum length at the schema level without wasting space on short values.
- To provide a portable, standard type for text data across RDBMSs.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name VARCHAR(n)
-- or
column_name CHARACTER VARYING(n)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `n` | The maximum length in characters (or bytes, depending on the RDBMS and type). |
| `VARCHAR` / `CHARACTER VARYING` | Synonymous keywords for the variable-length type. |

**Syntax rules:**

- The length `n` must be a positive integer.
- Values longer than `n` are rejected (PostgreSQL, SQL Server) or truncated (MySQL in non-strict mode).
- `VARCHAR` without a length is allowed in some systems (e.g., PostgreSQL treats it as `text`; MySQL requires a length).

**Constraints and limitations:**

- Very large `n` values can mislead the optimiser's statistics and cost estimation.
- In MySQL, the maximum `VARCHAR` length is 65,535 bytes, shared across the row.
- In SQL Server, `varchar(n)` is limited to 8,000 bytes; use `varchar(max)` for larger values.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – VARCHAR(50) for usernames

```sql
-- Step 1: Create a table with a VARCHAR(50) column.
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL
);

-- Step 2: Insert values of different lengths.
INSERT INTO users (username, email) VALUES
    ('alice', 'alice@example.com'),
    ('bob_the_builder_with_a_long_name', 'bob@example.com');

-- Step 3: Verify the stored lengths.
SELECT username, length(username) AS len FROM users ORDER BY user_id;
```

**Expected output:**

```text
         username          | len
---------------------------+-----
 alice                     |   5
 bob_the_builder_with_a_long_name |  32
(2 rows)
```

**Why this output appears**: `VARCHAR(50)` stores each username using only the space required for its actual length (5 bytes for `'alice'`, 32 bytes for the longer name) plus a small length overhead. No padding is added.

#### Real-World Cases

- **User profiles**: Usernames, emails, and display names are naturally variable-length and fit `VARCHAR(n)` with sensible maximums.
- **Product catalogues**: Product names and SKUs use `VARCHAR(100)` and `VARCHAR(64)` respectively, balancing maximum length with storage efficiency.

---

### 4. `CHARACTER VARYING`

#### Definitions

**Core Definition**
`CHARACTER VARYING(n)` is the SQL-standard spelling of `VARCHAR(n)`, a variable-length character type.

**Technical Definition**
`CHARACTER VARYING(n)` and `VARCHAR(n)` are identical in PostgreSQL, MySQL, and SQL Server. PostgreSQL documents `character varying(n)` and `varchar(n)` as the same type, and SQL Server's `varchar` is the implementation of `character varying`. The `CHARACTER VARYING` spelling is preferred by the SQL standard for portability.

**Beginner-Friendly Explanation**
`CHARACTER VARYING` is the formal name for `VARCHAR`. It behaves exactly the same way: a stretchy text bag with a maximum length.

#### Purposes

- To provide the SQL-standard spelling for variable-length character columns.
- To maximise portability in schemas intended to run on multiple RDBMSs.
- To improve readability for readers familiar with the standard.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name CHARACTER VARYING(n)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `CHARACTER VARYING(n)` | SQL-standard spelling of `VARCHAR(n)`. |

**Syntax rules:**

- Identical to `VARCHAR(n)` in every respect.
- `CHARACTER VARYING` without a length is allowed in PostgreSQL (equivalent to `text`) but not in MySQL.

**Constraints and limitations:**

- Some developers prefer the brevity of `VARCHAR`; the two spellings are interchangeable.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – CHARACTER VARYING is the same as VARCHAR

```sql
-- Step 1: Create a table using the standard spelling.
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name CHARACTER VARYING(100) NOT NULL,
    description CHARACTER VARYING(500)
);

-- Step 2: Insert a row.
INSERT INTO products (name, description) VALUES ('Widget', 'A small widget');

-- Step 3: Verify the column types.
SELECT column_name, data_type, character_maximum_length
FROM information_schema.columns
WHERE table_name = 'products';
```

**Expected output:**

```text
 column_name |     data_type     | character_maximum_length
-------------+-------------------+--------------------------
 product_id  | integer           |
 name        | character varying |                      100
 description | character varying |                      500
(3 rows)
```

**Why this output appears**: `CHARACTER VARYING(100)` is stored as `character varying` with a maximum length of 100, exactly as `VARCHAR(100)` would be. The behaviour and storage characteristics are identical.

#### Real-World Cases

- **Cross-RDBMS schemas**: A schema designed to run on PostgreSQL, MySQL, and SQL Server uses `CHARACTER VARYING` for maximum portability.
- **Standards documentation**: Using the standard spelling aligns with SQL:2016 and later specifications.

---

### 5. `TEXT`

#### Definitions

**Core Definition**
`TEXT` is an unbounded variable-length character type that stores strings of arbitrary length (subject to system limits).

**Technical Definition**
In PostgreSQL, `text` is a first-class variable-length character type with no declared length limit; it is the preferred type for strings whose maximum length is unknown or very large. PostgreSQL documents `text` as accepting strings of any length. In MySQL, `TEXT` is a family of types (`TINYTEXT`, `TEXT`, `MEDIUMTEXT`, `LONGTEXT`) with maximum lengths from 255 bytes to 4 GB; `TEXT` columns are stored off-row when they exceed a threshold. In SQL Server, `varchar(max)` and `nvarchar(max)` provide similar unbounded behaviour and are stored off-row when large.

**Beginner-Friendly Explanation**
`TEXT` is the unlimited text bag. It can hold as much text as you need—paragraphs, articles, JSON documents—without you declaring a maximum length. In PostgreSQL, `TEXT` is just as fast as `VARCHAR`; in MySQL and SQL Server, very large `TEXT` values may be stored separately from the row, which can affect performance.

#### Purposes

- To store free-form text whose maximum length is genuinely unknown or very large (e.g., articles, comments, log messages).
- To avoid arbitrary length limits that might truncate legitimate data.
- To provide a simple, portable type for unbounded text in PostgreSQL.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name TEXT
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `TEXT` | Unbounded variable-length character type. |

**Syntax rules:**

- `TEXT` does not accept a length modifier; it is always unbounded.
- In PostgreSQL, `text` can be indexed and sorted like any other character type.
- In MySQL, `TEXT` columns cannot have a default value (before MySQL 8.0.13) and require prefix lengths for indexing.

**Constraints and limitations:**

- In MySQL and SQL Server, `TEXT`/`varchar(max)` values may be stored off-row, which can slow down queries that retrieve them.
- In MySQL, `TEXT` columns cannot be used directly in `ORDER BY` without a prefix length in some configurations.
- In PostgreSQL, `text` has no performance penalty compared with `varchar`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – TEXT for article bodies

```sql
-- Step 1: Create a table with a TEXT column.
CREATE TABLE articles (
    article_id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    body TEXT NOT NULL
);

-- Step 2: Insert an article with a long body.
INSERT INTO articles (title, body) VALUES
    ('Hello World', 'This is a long article body that could contain many paragraphs of text. It has no declared maximum length because the TEXT type is unbounded.');

-- Step 3: Query the length of the body.
SELECT title, length(body) AS body_length FROM articles;
```

**Expected output:**

```text
    title    | body_length
-------------+-------------
 Hello World |         152
(1 row)
```

**Why this output appears**: `TEXT` stores the entire body without truncation, and `length(body)` returns the actual character count. There is no padding and no maximum length to worry about.

#### Real-World Cases

- **Content management**: Article bodies, blog posts, and comments use `TEXT` because their length is unpredictable.
- **Logging**: Application logs and audit trails store unstructured messages in `TEXT` columns.
- **JSON documents**: In MySQL and SQL Server, `TEXT` or `varchar(max)` stores JSON payloads when a native JSON type is not available or not needed.

---

### 6. Character Type Best Practices

#### Definitions

**Core Definition**
Character type best practices are the guidelines for selecting, declaring, and using character types to ensure data integrity, storage efficiency, and query performance.

**Technical Definition**
Best practices include: (1) **use `VARCHAR(n)` for variable-length text** and reserve `CHAR(n)` for genuinely fixed-length values; (2) **choose a sensible `n`** that reflects the true maximum length, not an arbitrary large number; (3) **prefer `TEXT` in PostgreSQL** for unbounded strings because it has no performance penalty versus `varchar`; (4) **use `NVARCHAR`/`NCHAR` in SQL Server and MySQL** when Unicode data must be stored; (5) **avoid `CHAR` for variable-length data** because of space padding and trailing-space comparison semantics; and (6) **consider index prefix lengths** for `TEXT` columns in MySQL.

**Beginner-Friendly Explanation**
Best practices are the habits that keep text data efficient and correct. Use `VARCHAR` for names and emails, `CHAR` only for fixed codes, and `TEXT` only for genuinely unbounded text. Always pick a maximum length that reflects reality, and use Unicode types when your data includes non-Latin characters.

#### Purposes

- To minimise storage consumption by avoiding unnecessary padding and oversized columns.
- To ensure correct comparison and sorting behaviour.
- To avoid truncation errors and index limitations.
- To align the schema with the actual data domain.

#### Syntax Rules and Structure

**Best-practice guidance:**

| Scenario | Recommended type | Rationale |
|----------|------------------|-----------|
| Fixed-length code (country, currency) | `CHAR(n)` | Predictable length; no padding waste |
| Variable-length name, email, title | `VARCHAR(n)` with a sensible `n` | Efficient storage; enforces a maximum |
| Unbounded text (article body, log message) | `TEXT` (PostgreSQL) or `varchar(max)` (SQL Server) | No arbitrary limit |
| Unicode text (names in any language) | `NVARCHAR(n)` (SQL Server/MySQL) | Correct storage of non-Latin characters |
| Frequently queried text | `VARCHAR(n)` with an index | Indexes are more efficient on bounded lengths |

**Syntax rules:**

- `VARCHAR(n)` should have an `n` that is small enough to be meaningful but large enough to avoid truncation.
- `CHAR(n)` should only be used when every value is exactly `n` characters.
- `TEXT` columns should be indexed with a prefix length in MySQL if they need to be searchable.

**Constraints and limitations:**

- Changing a column's type later may require a table rewrite.
- Overly large `VARCHAR` lengths can mislead the optimiser and waste statistics memory.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Best-practice character schema

```sql
-- Step 1: Create a table applying character type best practices.
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    country_code CHAR(2) NOT NULL,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(320) NOT NULL UNIQUE,
    notes TEXT
);

-- Step 2: Insert a row.
INSERT INTO customers (country_code, full_name, email, notes)
VALUES ('US', 'Alice Smith', 'alice@example.com', 'Prefers email contact.');

-- Step 3: Verify the column types and lengths.
SELECT column_name, data_type, character_maximum_length
FROM information_schema.columns
WHERE table_name = 'customers';
```

**Expected output:**

```text
 column_name  |     data_type     | character_maximum_length
--------------+-------------------+--------------------------
 customer_id  | integer           |
 country_code | character         |                        2
 full_name    | character varying |                      100
 email        | character varying |                      320
 notes        | text              |
(5 rows)
```

**Why this output appears**: The schema uses `CHAR(2)` for the fixed-length country code, `VARCHAR(100)` for the variable-length name, `VARCHAR(320)` for the email (matching the maximum email length), and `TEXT` for the unbounded notes. Each type reflects the actual characteristics of its data.

#### Real-World Cases

- **International applications**: `NVARCHAR` stores customer names in Chinese, Arabic, or Cyrillic without corruption, while `VARCHAR` stores ASCII-only fields like email addresses.
- **High-volume transaction tables**: `VARCHAR(64)` for SKUs and `CHAR(3)` for currency codes keep rows compact and indexes efficient.

---

## References

- PostgreSQL Documentation – Character Types – https://www.postgresql.org/docs/current/datatype-character.html 【6†L52-L55】
- MySQL 8.0 Reference Manual – The CHAR and VARCHAR Types – https://dev.mysql.com/doc/refman/8.0/en/char.html 【9†L38-L51】
- MySQL 8.0 Reference Manual – The BLOB and TEXT Types – https://dev.mysql.com/doc/refman/8.0/en/blob.html
- Microsoft Learn – char and varchar (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/data-types/char-and-varchar-transact-sql 【10†L111-L113】
- Microsoft Learn – nchar and nvarchar (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/data-types/nchar-and-nvarchar-transact-sql
- SQL Server Central – VARCHAR vs NVARCHAR – https://www.sqlservercentral.com/articles/varchar-vs-nvarchar
- PostgreSQL Documentation – Data Types (overview) – https://www.postgresql.org/docs/current/datatype.html
- MySQL 8.0 Reference Manual – Data Type Storage Requirements – https://dev.mysql.com/doc/refman/8.0/en/storage-requirements.html
- SQLShack – SQL Server VARCHAR vs NVARCHAR – https://www.sqlshack.com/sql-server-varchar-vs-nvarchar/
- Bytebase – How to Use CHAR and VARCHAR in MySQL – https://www.bytebase.com/reference/mysql/how-to/how-to-use-char-varchar-mysql/
- Vertabelo – SQL Data Types: A Comprehensive Guide – https://vertabelo.com/blog/sql-data-types/
- DbVis – SQL Character Data Types – https://www.dbvis.com/thetable/sql-character-data-types/