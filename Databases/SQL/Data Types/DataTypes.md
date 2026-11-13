# Comprehensive Programming Cheat Sheet: SQL Data Type Fundamentals

## Topic Overview

### Definitions

**Core Definition**
Data types are the fundamental attributes assigned to every column in a relational database that determine the kind of values the column can hold, how those values are stored, and what operations can be performed on them.

**Technical Definition**
A data type is a specification that constrains the set of values a column can store, defines the storage format and size, and determines the valid operations for comparison, arithmetic, and sorting. SQL data types are organised into categories—exact numeric, approximate numeric, character string, Unicode character string, binary, date and time, boolean, JSON, UUID, and spatial types—each with vendor-specific implementations. The choice of data type directly affects storage consumption, query performance, index efficiency, and data integrity. PostgreSQL provides a rich set of native data types and allows users to add new types with `CREATE TYPE`. MySQL supports numeric, date and time, string, spatial, and JSON types, with documented storage requirements for each. SQL Server categorises types into exact numeric, approximate numeric, date and time, character string, Unicode character string, binary, and special types.

**Beginner-Friendly Explanation**
Data types are the "kind of thing" each column in a database table is allowed to hold. An age column holds whole numbers, a name column holds text, a price column holds decimal numbers, and a created-at column holds dates and times. Choosing the right data type is like choosing the right container for a liquid: a small cup for a shot of espresso, a large jug for water. Pick too small a container and it overflows (data truncation or errors); pick too large and you waste space and slow things down.

### Key Characteristics

- **Type categories drive behaviour**: Exact numeric types (`INTEGER`, `NUMERIC`, `DECIMAL`) store values with no rounding, while approximate numeric types (`FLOAT`, `REAL`, `DOUBLE PRECISION`) store approximations using IEEE 754 floating-point arithmetic.
- **Storage is not always fixed**: Variable-length types (`VARCHAR`, `TEXT`, `VARBINARY`) consume only the space needed for the actual value, whereas fixed-length types (`CHAR`, `NCHAR`) pad to the declared length.
- **Precision and scale matter for exactness**: `NUMERIC(p, s)` and `DECIMAL(p, s)` specify the total number of digits (`p`) and the digits after the decimal point (`s`); the declared precision is a maximum, not a fixed allocation in PostgreSQL.
- **Vendor divergence is substantial**: PostgreSQL, MySQL, and SQL Server use different type names, ranges, storage sizes, and semantics for seemingly equivalent types (e.g., `TIMESTAMP` with or without time zone, `TEXT` storage, `JSON` vs. `JSONB`).
- **Selection is a trade-off**: Smaller types generally mean faster queries and less I/O, but a type that is too small risks data loss; nullable columns add overhead and should be avoided when possible.

### Prerequisites

- A running RDBMS instance (PostgreSQL, MySQL, or SQL Server) and a client connection.
- Basic familiarity with SQL `CREATE TABLE` syntax and the concept of a column definition.
- Understanding of the application's data domain: the range of values, whether exact precision is required, and whether values include Unicode characters or time zones.

### Related Programming Areas

- Data Definition Language (DDL) – `CREATE TABLE`, `ALTER TABLE`.
- Query optimisation – data types affect index size, statistics, and execution plans.
- Data integrity – incorrect types can lead to truncation, rounding, or overflow errors.
- Application development – data types must align with application-level types (e.g., Java `BigDecimal`, Python `decimal.Decimal`).
- Storage engineering – type selection directly impacts disk usage, memory, and backup size.

---

## Core Concepts and Features

### 1. Data Types

#### Definitions

**Core Definition**
A data type defines the kind of values a column can hold, such as numbers, text, dates, or binary data.

**Technical Definition**
In SQL, every column has a declared data type that constrains the values it can store, the storage space it consumes, and the operations that can be applied. PostgreSQL's built-in general-purpose types include `bigint`, `integer`, `smallint`, `numeric`, `real`, `double precision`, `boolean`, `text`, `varchar`, `char`, `date`, `timestamp`, `timestamptz`, `interval`, `json`, `jsonb`, `uuid`, and `bytea`, among others. MySQL groups its types into numeric, date and time, string, spatial, and JSON categories, with storage requirements documented per type. SQL Server distinguishes between exact numeric, approximate numeric, date and time, character string, Unicode character string, binary, and special types such as `UNIQUEIDENTIFIER` and `XML`.

**Beginner-Friendly Explanation**
A data type tells the database what kind of value a column holds. When you create a table, you assign a type to each column, and the database uses that type to decide how to store the value, how much space to reserve, and what operations are valid.

#### Purposes

- To constrain the values stored in a column to a valid set, preventing invalid data from entering the database.
- To determine the storage format and size, directly affecting disk usage and memory consumption.
- To enable type-specific operations such as arithmetic on numbers, concatenation on strings, and date arithmetic on temporal values.
- To ensure data consistency and interoperability across applications and database systems.

#### Syntax Rules and Structure

**General syntax (column definition):**

```sql
column_name data_type [ column_constraint ... ]
```

**Examples across RDBMSs:**

```sql
-- PostgreSQL
CREATE TABLE example (
    id       integer,
    name     varchar(100),
    price    numeric(10,2),
    created  timestamptz
);

-- MySQL
CREATE TABLE example (
    id       INT,
    name     VARCHAR(100),
    price    DECIMAL(10,2),
    created  DATETIME
);

-- SQL Server
CREATE TABLE example (
    id       INT,
    name     NVARCHAR(100),
    price    DECIMAL(10,2),
    created  DATETIME2
);
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `column_name` | The name of the column. |
| `data_type` | The type of values the column holds. |
| `(p, s)` | Precision and scale for exact numeric types. |
| `(n)` | Length for character and binary types. |

**Syntax rules:**

- The data type must be specified for every column.
- Some types accept length, precision, or scale modifiers; others do not.
- Type names are case-insensitive in most RDBMSs, but the SQL standard uses uppercase for built-in types.

**Constraints and limitations:**

- Type names and features vary by RDBMS; `SERIAL` exists in PostgreSQL but not MySQL, `AUTO_INCREMENT` exists in MySQL but not PostgreSQL (which uses `GENERATED ... AS IDENTITY`).
- Using an inappropriate type can lead to truncation, rounding, or overflow errors.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Create a table with common data types

```sql
-- Step 1: Create a table with integer, text, numeric, and timestamp types.
CREATE TABLE products (
    product_id   integer PRIMARY KEY,
    name         text NOT NULL,
    price        numeric(10,2) NOT NULL,
    created_at   timestamptz DEFAULT now()
);

-- Step 2: Insert a row.
INSERT INTO products (product_id, name, price) VALUES (1, 'Widget', 9.99);

-- Step 3: Verify.
SELECT * FROM products;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
 product_id |  name  | price |          created_at
------------+--------+-------+-------------------------------
          1 | Widget |  9.99 | 2026-11-13 10:00:00.123+00
(1 row)
```

**Why this output appears**: Each column is assigned a specific type: `integer` for IDs, `text` for variable-length strings, `numeric(10,2)` for exact monetary values, and `timestamptz` for time-zone-aware timestamps. The default `now()` supplies a timestamp automatically.

#### Real-World Cases

- **Financial systems**: `numeric` or `decimal` is used for monetary values to avoid floating-point rounding errors.
- **Content management**: `text` or `varchar` stores article bodies, titles, and metadata.
- **IoT telemetry**: `jsonb` (PostgreSQL) stores flexible sensor payloads that do not fit a fixed schema.

---

### 2. Type Categories

#### Definitions

**Core Definition**
Type categories group data types by their fundamental nature—numeric, character, temporal, boolean, binary, and special—so that similar types share common behaviours and constraints.

**Technical Definition**
SQL data types fall into several broad categories. **Exact numeric** types (`INTEGER`, `SMALLINT`, `BIGINT`, `NUMERIC`, `DECIMAL`) store values without rounding; **approximate numeric** types (`FLOAT`, `REAL`, `DOUBLE PRECISION`) store approximations using binary floating-point arithmetic. **Character string** types (`CHAR`, `VARCHAR`, `TEXT`) store text, while **Unicode character string** types (`NCHAR`, `NVARCHAR`) store Unicode data. **Date and time** types (`DATE`, `TIME`, `TIMESTAMP`, `DATETIME`, `DATETIME2`, `TIMESTAMPTZ`) store temporal values with varying precision and time-zone awareness. **Binary** types (`BYTEA`, `VARBINARY`, `IMAGE`) store raw bytes. **Boolean** types store true/false values. **Special** types include `JSON`, `JSONB`, `UUID`, `XML`, and spatial types.

**Beginner-Friendly Explanation**
Type categories are like food groups for data. Numbers go in the numeric group, text in the character group, dates in the temporal group, and so on. Knowing the category helps you choose the right specific type.

#### Purposes

- To provide a mental model for selecting types based on the nature of the data.
- To understand the fundamental trade-offs (exactness vs. range, fixed vs. variable length) within each category.
- To navigate vendor-specific type names by mapping them to their category.

#### Syntax Rules and Structure

**Category overview (common types across RDBMSs):**

| Category | PostgreSQL | MySQL | SQL Server |
|----------|------------|-------|------------|
| Exact integer | `smallint`, `integer`, `bigint` | `TINYINT`, `SMALLINT`, `MEDIUMINT`, `INT`, `BIGINT` | `tinyint`, `smallint`, `int`, `bigint` |
| Exact decimal | `numeric(p,s)`, `decimal(p,s)` | `DECIMAL(p,s)`, `NUMERIC(p,s)` | `decimal(p,s)`, `numeric(p,s)` |
| Approximate | `real`, `double precision` | `FLOAT`, `DOUBLE` | `real`, `float` |
| Character | `char(n)`, `varchar(n)`, `text` | `CHAR(n)`, `VARCHAR(n)`, `TEXT` | `char(n)`, `varchar(n)`, `varchar(max)` |
| Unicode character | (via UTF-8 database encoding) | `NCHAR`, `NVARCHAR` | `nchar(n)`, `nvarchar(n)`, `nvarchar(max)` |
| Date/time | `date`, `time`, `timestamp`, `timestamptz`, `interval` | `DATE`, `TIME`, `DATETIME`, `TIMESTAMP` | `date`, `time`, `datetime2`, `datetimeoffset` |
| Boolean | `boolean` | `BOOLEAN` (synonym `TINYINT(1)`) | `bit` |
| JSON | `json`, `jsonb` | `JSON` | `nvarchar` with `ISJSON` |
| UUID | `uuid` | `CHAR(36)` | `uniqueidentifier` |

**Component breakdown:**

| Category | Key characteristic |
|----------|-------------------|
| Exact numeric | No rounding; exact precision and scale. |
| Approximate numeric | Rounding may occur; suitable for scientific measurements. |
| Character | Fixed or variable length; collation affects comparison. |
| Unicode character | Stores characters from any language; uses more space. |
| Date/time | Precision from seconds to nanoseconds; time-zone awareness varies. |
| Boolean | Stores true/false (or 1/0 in MySQL). |
| Binary | Stores raw bytes, images, or documents. |
| Special | JSON, UUID, XML, spatial, and network types. |

**Syntax rules:**

- Type names within a category may have aliases (e.g., `int` = `integer`, `varchar` = `character varying`).
- The choice within a category depends on range, precision, and storage requirements.

**Constraints and limitations:**

- Not all types are available in all RDBMSs (e.g., PostgreSQL `jsonb` has no direct MySQL or SQL Server equivalent).
- Some categories overlap (e.g., MySQL `BOOLEAN` is a synonym for `TINYINT(1)`).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Using different type categories

```sql
-- Step 1: Create a table spanning multiple categories.
CREATE TABLE sensor_data (
    sensor_id   integer,
    reading     double precision,
    label       varchar(50),
    recorded_at timestamptz,
    metadata    jsonb,
    is_active   boolean
);

-- Step 2: Insert a row.
INSERT INTO sensor_data VALUES
    (1, 23.456, 'temperature', now(), '{"unit":"C"}', true);

-- Step 3: Query with a JSON operator.
SELECT sensor_id, metadata->>'unit' AS unit FROM sensor_data;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
 sensor_id | unit
-----------+------
         1 | C
(1 row)
```

**Why this output appears**: The table uses an exact integer for the ID, an approximate numeric for the reading, a variable-length string for the label, a time-zone-aware timestamp, a binary JSON document (`jsonb`), and a boolean. The `->>` operator extracts the `unit` field from the JSON document.

#### Real-World Cases

- **Mixed data types**: An IoT table combines numeric readings, timestamps, and JSON metadata.
- **Financial data**: Exact numeric types are used for amounts, while approximate types are used for statistical projections.

---

### 3. Type Selection

#### Definitions

**Core Definition**
Type selection is the process of choosing the most appropriate data type for each column based on the nature of the data, the required precision, the storage budget, and the operations the column must support.

**Technical Definition**
Type selection involves matching the application's data domain to the RDBMS's type system. Key considerations include: (1) **exactness**—whether the data requires exact precision (use `NUMERIC`/`DECIMAL`) or tolerates approximation (use `FLOAT`/`DOUBLE`); (2) **range**—whether the type can accommodate the maximum value (e.g., `INT` vs. `BIGINT`); (3) **length**—whether the type supports the maximum string length (e.g., `VARCHAR(50)` vs. `VARCHAR(255)`); (4) **time-zone awareness**—whether timestamps must be stored with or without time zone; and (5) **nullability**—whether the column must reject `NULL` values. Microsoft's guidance recommends using the most specific length or precision and choosing the smallest type that can accommodate the data without risk of overflow.

**Beginner-Friendly Explanation**
Type selection is like picking the right size of box for shipping an item. Too small and the item breaks (data loss); too large and you pay for empty space (wasted storage and slower queries). The goal is to pick the smallest box that safely holds the item.

#### Purposes

- To ensure that all valid values for the column can be stored without truncation or overflow.
- To minimise storage consumption and improve query performance.
- To avoid the pitfalls of implicit conversions and type mismatches.
- To align the database schema with application-level types and business requirements.

#### Syntax Rules and Structure

Type selection is a design decision, not a single syntax. Key guidelines:

- **Use exact numeric types for money**: `NUMERIC(10,2)` or `DECIMAL(10,2)` instead of `FLOAT`.
- **Use integer types for identifiers and counters**: `INT` for IDs that fit within 2.1 billion; `BIGINT` when the range may grow beyond that.
- **Use `VARCHAR(n)` for variable-length strings**: set `n` to the maximum expected length, not an arbitrary large number.
- **Use `CHAR(n)` only for fixed-length strings**: e.g., country codes (`CHAR(2)`), hash values (`CHAR(64)`).
- **Use `TIMESTAMPTZ` (PostgreSQL) or `DATETIMEOFFSET` (SQL Server) for cross-time-zone applications**.
- **Avoid `NULL` when possible**: define columns as `NOT NULL` unless the absence of a value is meaningful.

**Component breakdown:**

| Decision | Recommended type | Rationale |
|----------|------------------|-----------|
| Monetary amount | `NUMERIC(p,2)` / `DECIMAL(p,2)` | Exactness; no floating-point rounding |
| Primary key | `INT` or `BIGINT` | Compact, fast, index-friendly |
| Short label | `VARCHAR(50)` | Minimal storage; no padding |
| Fixed code | `CHAR(2)` / `CHAR(3)` | Predictable length; no padding waste |
| Timestamp with time zone | `TIMESTAMPTZ` (PostgreSQL), `DATETIMEOFFSET` (SQL Server) | Correct cross-zone ordering |
| Boolean flag | `BOOLEAN` / `BIT` | Clear semantics; minimal storage |

**Constraints and limitations:**

- Some type choices are irreversible without a full table rewrite (`ALTER TABLE ... ALTER COLUMN ... TYPE`).
- Overly large `VARCHAR` lengths can mislead the optimiser's statistics and cost estimation.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Choosing types for an orders table

```sql
-- Step 1: Create an orders table with carefully selected types.
CREATE TABLE orders (
    order_id     bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id  integer NOT NULL,
    total_amount numeric(12,2) NOT NULL CHECK (total_amount >= 0),
    status       varchar(20) NOT NULL DEFAULT 'pending',
    created_at   timestamptz NOT NULL DEFAULT now(),
    notes        text
);
```

**Expected output:**

```text
CREATE TABLE
```

**Why this output appears**: `bigint` accommodates potentially large order volumes; `numeric(12,2)` stores exact monetary amounts up to 9,999,999,999.99; `varchar(20)` bounds the status string; `timestamptz` stores time-zone-aware creation times; `text` allows unbounded notes; `NOT NULL` constraints reject missing essential values.

#### Real-World Cases

- **E-commerce**: Order totals use `DECIMAL(12,2)`, status uses `VARCHAR(20)`, and timestamps use `TIMESTAMPTZ`.
- **User management**: Usernames use `VARCHAR(50)`, emails use `VARCHAR(255)`, and passwords use `CHAR(60)` for bcrypt hashes.

---

### 4. Data Type Best Practices

#### Definitions

**Core Definition**
Data type best practices are the established guidelines for selecting, defining, and maintaining data types to ensure data integrity, optimal performance, and maintainable schemas.

**Technical Definition**
Best practices include: (1) **choose the smallest type that safely accommodates the data**—smaller types mean fewer bytes per row, more rows per page, better cache efficiency, and faster queries; (2) **avoid nullable columns when possible**—nullable columns add metadata overhead and can reduce the effectiveness of query optimisation and statistics; (3) **use `VARCHAR` over `CHAR` for variable-length data**—`CHAR` pads values to the declared length, wasting space; (4) **always specify precision and scale for decimal types**—omitting them defaults to `DECIMAL(18,0)` in SQL Server, consuming 9 bytes per row unnecessarily; (5) **use Unicode types (`NVARCHAR`) only when necessary**—they consume twice the storage of non-Unicode types; and (6) **use `TIMESTAMP` with time zone for cross-zone data**—`TIMESTAMPTZ` or `DATETIMEOFFSET` prevents incorrect ordering when data spans multiple time zones.

**Beginner-Friendly Explanation**
Best practices are the habits that keep your database fast, compact, and reliable. Choose small types, avoid unnecessary `NULL`s, use variable-length strings instead of padded fixed-length ones, and always specify how many digits you need for decimals.

#### Purposes

- To minimise storage consumption and improve I/O efficiency.
- To avoid performance degradation caused by overly large types or nullable columns.
- To prevent data loss caused by types that are too small.
- To ensure that the schema is maintainable and semantically clear.

#### Syntax Rules and Structure

Best practices are not enforced by a single syntax but by design choices:

- **Smallest type**: `SMALLINT` instead of `INT` when the range fits; `INT` instead of `BIGINT` when the range fits.
- **`VARCHAR` over `CHAR`**: `VARCHAR(50)` for names, `CHAR(2)` only for fixed-length codes.
- **Explicit precision**: `DECIMAL(10,2)` instead of `DECIMAL`.
- **`NOT NULL`**: add `NOT NULL` to columns that must always have a value.
- **`TIMESTAMPTZ`**: prefer `timestamptz` over `timestamp` in PostgreSQL for cross-zone applications.

**Constraints and limitations:**

- Changing a column's type later can require a full table rewrite and may be disruptive on large tables.
- Over-optimising for storage (e.g., using `TINYINT` for a value that might exceed 255) can lead to data loss.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Applying best practices in a table definition

```sql
-- Step 1: Create a table with best-practice type choices.
CREATE TABLE customers (
    customer_id  INT IDENTITY(1,1) PRIMARY KEY,
    full_name    NVARCHAR(100) NOT NULL,
    email        NVARCHAR(320) NOT NULL,
    country_code CHAR(2) NOT NULL,
    balance      DECIMAL(12,2) NOT NULL DEFAULT 0.00,
    created_at   DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME()
);
```

**Expected output:**

```text
Commands completed successfully.
```

**Why this output appears**: `INT` provides a compact primary key; `NVARCHAR(100)` accommodates Unicode names without fixed padding; `NVARCHAR(320)` matches the maximum email length; `CHAR(2)` is appropriate for a fixed-length country code; `DECIMAL(12,2)` stores exact balances; `DATETIME2` provides higher precision than legacy `DATETIME`; `NOT NULL` constraints reject missing essential values.

#### Real-World Cases

- **High-throughput OLTP**: Small, fixed-width types (`INT`, `CHAR(2)`) keep rows compact and indexes shallow.
- **International applications**: `NVARCHAR` stores names and addresses in any language without corruption.

---

### 5. Precision and Scale

#### Definitions

**Core Definition**
Precision is the total number of significant digits a numeric column can store; scale is the number of digits stored to the right of the decimal point.

**Technical Definition**
For exact numeric types such as `NUMERIC(p, s)` and `DECIMAL(p, s)`, `p` (precision) specifies the maximum total number of digits, and `s` (scale) specifies the number of digits after the decimal point. In MySQL, `M` is the precision (total digits) and `D` is the scale (digits after the decimal point), with `M` ranging from 1 to 65 and `D` from 0 to 30. PostgreSQL's `numeric` type allows up to 131,072 digits before the decimal point and 16,383 digits after, and the SQL standard requires the scale to be in the range 0 to precision. In SQL Server, `DECIMAL(p, s)` has a precision of 1–38 digits, and omitting `(p, s)` defaults to `DECIMAL(18,0)`, consuming 9 bytes per row. Crucially, the declared precision and scale are **maximums, not fixed allocations**: PostgreSQL stores numeric values without extra leading or trailing zeroes, so a `numeric(10,2)` column does not always consume the same number of bytes.

**Beginner-Friendly Explanation**
Precision is how many digits a number can have in total; scale is how many of those digits come after the decimal point. `DECIMAL(8,2)` can store up to 999,999.99—eight digits total, two after the point. Choosing the right precision and scale is like choosing the right ruler: too short and you cannot measure the length; too long and you carry unnecessary bulk.

#### Purposes

- To ensure that numeric values fit within the column without overflow or truncation.
- To avoid storing unnecessary digits that waste storage and mislead statistics.
- To enforce exactness for financial and scientific data where rounding is unacceptable.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name { NUMERIC | DECIMAL } (precision, scale)
```

**Examples:**

```sql
-- PostgreSQL: exact numeric with 10 total digits and 2 decimal places.
price numeric(10,2)

-- MySQL: same concept.
price DECIMAL(10,2)

-- SQL Server: same concept.
price DECIMAL(10,2)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `precision` / `p` / `M` | Maximum total number of digits. |
| `scale` / `s` / `D` | Number of digits after the decimal point. |
| `numeric` / `decimal` | Synonym types for exact numeric values. |

**Syntax rules:**

- Scale must be less than or equal to precision.
- In MySQL, the maximum precision is 65 and the maximum scale is 30.
- In SQL Server, precision ranges from 1 to 38; the default is 18 if omitted.

**Constraints and limitations:**

- A value with more digits than the declared precision causes an overflow error.
- A value with more decimal places than the declared scale is rounded (PostgreSQL) or truncated (MySQL, depending on SQL mode).
- The declared precision is a maximum, not a fixed allocation in PostgreSQL; storage is variable.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Precision and scale in action

```sql
-- Step 1: Create a table with numeric(8,2).
CREATE TABLE prices (id int, amount numeric(8,2));

-- Step 2: Insert values within and outside the precision.
INSERT INTO prices VALUES (1, 123456.78);  -- fits: 6 integer digits + 2 decimal = 8 total
INSERT INTO prices VALUES (2, 1234567.89); -- fails: 7 integer digits + 2 decimal = 9 > 8
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
ERROR:  numeric field overflow
DETAIL:  A field with precision 8, scale 2 must round to an absolute value less than 10^6.
```

**Why this output appears**: `numeric(8,2)` allows a maximum of 6 digits before the decimal point and 2 after. The first value (`123456.78`) fits; the second (`1234567.89`) has 7 digits before the decimal point and exceeds the precision, causing an overflow error.

##### Example 2: MySQL – DECIMAL storage calculation

```sql
-- Step 1: Create a table with DECIMAL(18,9) and DECIMAL(20,6).
CREATE TABLE decimals (
    a DECIMAL(18,9),
    b DECIMAL(20,6)
);
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
```

**Why this output appears**: `DECIMAL(18,9)` has 9 integer digits and 9 fractional digits; each part requires 4 bytes (since 9 digits pack into 4 bytes), totaling 8 bytes. `DECIMAL(20,6)` has 14 integer digits and 6 fractional digits; the integer part requires 4 bytes for 9 digits plus 3 bytes for the remaining 5 digits, and the fractional part requires 3 bytes, totaling 10 bytes.

#### Real-World Cases

- **Financial ledgers**: `NUMERIC(19,4)` stores large monetary amounts with four decimal places for interest calculations.
- **Scientific measurements**: `NUMERIC(10,6)` stores readings with micro-unit precision without floating-point error.

---

### 6. Storage Implications

#### Definitions

**Core Definition**
Storage implications refer to the amount of disk and memory space consumed by different data types, and how those choices affect I/O efficiency, cache behaviour, and overall database performance.

**Technical Definition**
Storage consumption varies dramatically by type. Fixed-length types (`INT`, `BIGINT`, `CHAR`) consume a predictable number of bytes per row. Variable-length types (`VARCHAR`, `TEXT`, `VARBINARY`) consume only the actual value length plus a small overhead (typically 1–2 bytes for the length). In PostgreSQL, `numeric` storage is variable: two bytes for each group of four decimal digits, plus three to eight bytes overhead. In MySQL, `DECIMAL` values are stored in a binary format packing nine decimal digits into four bytes, with separate storage calculations for the integer and fractional parts. SQL Server's `DECIMAL` storage ranges from 5 to 17 bytes depending on precision. The maximum row size also differs: MySQL has a hard limit of 65,535 bytes per row (excluding `BLOB`/`TEXT`), while SQL Server and PostgreSQL have their own page and row limits. Nullable columns add a bitmap overhead; in SQL Server, the null bitmap adds 2 bytes per row plus 1 byte per 8 nullable columns.

**Beginner-Friendly Explanation**
Storage implications are about how much disk space your data types use. A `BIGINT` uses 8 bytes per row while an `INT` uses 4—double the space for the same number of rows. Choosing smaller types means more rows fit on each disk page, which means fewer I/O operations and faster queries.

#### Purposes

- To estimate database size and plan storage capacity.
- To choose types that minimise waste and maximise rows per page.
- To understand the performance impact of type choices on I/O and caching.
- To avoid row-size limits and overflow errors.

#### Syntax Rules and Structure

**Storage comparison (approximate, per row):**

| Type | PostgreSQL | MySQL | SQL Server |
|------|------------|-------|------------|
| `SMALLINT` | 2 bytes | 2 bytes | 2 bytes |
| `INTEGER` / `INT` | 4 bytes | 4 bytes | 4 bytes |
| `BIGINT` | 8 bytes | 8 bytes | 8 bytes |
| `NUMERIC` / `DECIMAL` | variable (2 bytes per 4 digits + overhead) | binary packed (9 digits per 4 bytes) | 5–17 bytes depending on precision |
| `CHAR(n)` | n bytes (padded) | n bytes (padded) | n bytes (padded) |
| `VARCHAR(n)` | variable (4 + n bytes) | variable (1 or 2 bytes length + data) | variable (2 bytes overhead + data) |
| `TEXT` / `VARCHAR(MAX)` | variable | variable | variable |
| `DATE` | 4 bytes | 3 bytes | 3 bytes |
| `TIMESTAMP` / `DATETIME` | 8 bytes | 4–8 bytes | 8 bytes |
| `BOOLEAN` / `BIT` | 1 byte | 1 byte (`TINYINT`) | 1 bit (packed) |

**Component breakdown:**

| Factor | Impact |
|--------|--------|
| Fixed vs. variable length | Fixed types consume the same space regardless of value; variable types consume only what is needed. |
| Precision and scale | Higher precision requires more bytes for decimal types. |
| Nullability | Nullable columns add a bitmap overhead (SQL Server: 2 bytes + 1 byte per 8 nullable columns). |
| Row size limits | MySQL: 65,535 bytes per row (excluding BLOB/TEXT); SQL Server: 8,060 bytes per row for fixed-length data. |

**Syntax rules:**

- The declared length of `VARCHAR(n)` is a maximum, not a fixed allocation.
- `CHAR(n)` always consumes `n` bytes, padding with spaces if the value is shorter.
- `NUMERIC` storage is variable in PostgreSQL; the declared precision is a maximum, not a fixed allocation.

**Constraints and limitations:**

- Exceeding the maximum row size causes an error or requires off-row storage (`TEXT`, `BLOB`).
- Very wide rows reduce the number of rows per page, increasing I/O for full-table scans.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Comparing storage of INT vs. BIGINT

```sql
-- Step 1: Create two tables, one with INT and one with BIGINT.
CREATE TABLE small_ids (id integer PRIMARY KEY, name text);
CREATE TABLE large_ids (id bigint PRIMARY KEY, name text);

-- Step 2: Insert the same number of rows into each.
INSERT INTO small_ids SELECT g, 'name' FROM generate_series(1, 1000000) AS g;
INSERT INTO large_ids SELECT g, 'name' FROM generate_series(1, 1000000) AS g;

-- Step 3: Compare table sizes.
SELECT pg_size_pretty(pg_total_relation_size('small_ids')) AS small_table,
       pg_size_pretty(pg_total_relation_size('large_ids')) AS large_table;
```

**Expected output:**

```text
 small_table | large_table
-------------+-------------
 42 MB       | 50 MB
(1 row)
```

**Why this output appears**: The `BIGINT` column consumes 8 bytes per row versus 4 bytes for the `INT` column, resulting in a larger table. The difference (approximately 8 MB for 1,000,000 rows) reflects the additional 4 bytes per row plus index overhead.

#### Real-World Cases

- **High-volume OLTP**: Choosing `INT` over `BIGINT` for IDs can reduce table and index size by 40–50%, improving cache hit rates and query speed.
- **Data warehousing**: Using `SMALLINT` for low-cardinality codes and `DATE` instead of `TIMESTAMP` where time-of-day is not needed can significantly reduce storage.

---

## References

- PostgreSQL Documentation – Data Types – https://www.postgresql.org/docs/current/datatype.html
- PostgreSQL Documentation – Numeric Types – https://www.postgresql.org/docs/current/datatype-numeric.html
- PostgreSQL Documentation – Character Types – https://www.postgresql.org/docs/current/datatype-character.html
- PostgreSQL Documentation – Date/Time Types – https://www.postgresql.org/docs/current/datatype-datetime.html
- PostgreSQL Documentation – Numeric Type Storage (mailing list) – https://www.postgresql.org/message-id/...
- MySQL 8.0 Reference Manual – Data Types – https://dev.mysql.com/doc/refman/8.0/en/data-types.html
- MySQL 8.0 Reference Manual – DECIMAL Data Type Characteristics – https://dev.mysql.com/doc/refman/8.0/en/precision-math-decimal-characteristics.html
- MySQL 8.0 Reference Manual – Data Type Storage Requirements – https://dev.mysql.com/doc/refman/8.0/en/storage-requirements.html
- Microsoft Learn – Data Types (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/data-types/data-types-transact-sql
- Microsoft Learn – decimal and numeric (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/data-types/decimal-and-numeric-transact-sql
- Microsoft Learn – Performance Guidelines (Fabric) – https://learn.microsoft.com/en-us/fabric/data-warehouse/guidelines-warehouse-performance
- Devart – SQL Server Data Types: Categories, Examples, and Best Practices – https://www.devart.com/blog/sql-server-data-types.html
- Educative – How to design an optimized SQL schema – https://www.educative.io/blog/optimized-sql-schema
- Pythian – MySQL Query Best Practices – https://www.pythian.com/blog/mysql-query-best-practices
- PostgreSQL Documentation – Numeric Type Storage Details (PDF) – https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.2/postgresql-9.2-US.pdf