# Comprehensive Programming Cheat Sheet: SQL Numeric Types

## Topic Overview

### Definitions

**Core Definition**
Numeric types are the SQL data types used to store numbers—integers, exact decimals, and approximate floating-point values—each with defined ranges, precision, storage requirements, and arithmetic behaviour.

**Technical Definition**
SQL numeric types fall into two broad groups: **exact numeric types** (`SMALLINT`, `INTEGER`, `BIGINT`, `DECIMAL`, `NUMERIC`) store values without rounding, and **approximate numeric types** (`REAL`, `DOUBLE PRECISION`) store binary floating-point approximations. PostgreSQL additionally provides auto-incrementing pseudo-types (`SMALLSERIAL`, `SERIAL`, `BIGSERIAL`) that expand to `integer NOT NULL DEFAULT nextval(...)` with an owned sequence. MySQL extends the standard with `TINYINT`, `MEDIUMINT`, and `UNSIGNED` attributes, while SQL Server provides `TINYINT`, `DECIMAL`/`NUMERIC`, `REAL`, and `FLOAT` with precision-dependent storage. The choice of numeric type directly affects exactness, storage footprint, arithmetic performance, and the risk of overflow or rounding errors.

**Beginner-Friendly Explanation**
Numeric types are the kinds of number containers a database offers. Some containers hold only whole numbers (like `SMALLINT` or `INT`), some hold exact decimal numbers (like `DECIMAL`), and some hold approximate scientific numbers (like `DOUBLE PRECISION`). The container you choose determines how large a number can be, how much space it takes, and whether the value is stored exactly or with tiny rounding errors.

### Key Characteristics

- **Exactness vs. approximation**: Integer and `DECIMAL`/`NUMERIC` types store values exactly; `REAL` and `DOUBLE PRECISION` use IEEE 754 binary floating-point and cannot represent every decimal fraction exactly (0.1, for example, has no exact binary representation).
- **Range and storage scale with size**: `SMALLINT` uses 2 bytes and covers roughly ±32,767; `INTEGER` uses 4 bytes and covers roughly ±2.1 billion; `BIGINT` uses 8 bytes and covers roughly ±9.2 quintillion.
- **Precision and scale are explicit for exact decimals**: `NUMERIC(p, s)` and `DECIMAL(p, s)` declare the total digits (`p`) and the digits after the decimal point (`s`); in PostgreSQL the declared precision is a maximum, and storage is variable.
- **Serial types are syntactic sugar**: PostgreSQL's `SERIAL` family is not a true type; it is shorthand for creating an integer column, a sequence, a `NOT NULL` constraint, and a default that draws from the sequence.
- **Vendor divergence is significant**: MySQL supports `UNSIGNED` integers and `TINYINT`/`MEDIUMINT`; SQL Server's `FLOAT(n)` maps to `REAL` or `DOUBLE PRECISION` depending on `n`; PostgreSQL's `NUMERIC` is arbitrary-precision, whereas SQL Server's `DECIMAL` caps at 38 digits.

### Prerequisites

- A running RDBMS instance (PostgreSQL, MySQL, or SQL Server) and a client connection.
- Familiarity with `CREATE TABLE` syntax and column definitions.
- Understanding of the application's data domain: maximum values, required decimal places, and whether exactness is mandatory.

### Related Programming Areas

- Data Definition Language (DDL) – column type declarations.
- Data integrity – overflow, truncation, and rounding behaviour.
- Query performance – integer arithmetic is faster than `NUMERIC`; smaller types improve cache efficiency.
- Storage engineering – type width directly impacts table and index size.
- Application mapping – SQL types correspond to language-level types such as `int`, `long`, `BigDecimal`, and `double`.

---

## Core Concepts and Features

### 1. `SMALLINT`

#### Definitions

**Core Definition**
`SMALLINT` is a 2-byte exact integer type with a smaller range than `INTEGER`.

**Technical Definition**
`SMALLINT` stores whole numbers from **−32,768 to +32,767** and consumes **2 bytes** per value in PostgreSQL, MySQL, and SQL Server. PostgreSQL describes it as useful only when disk space is at a premium, because `INTEGER` offers a better balance of range and performance.

**Beginner-Friendly Explanation**
`SMALLINT` is the smallest whole-number container. It holds numbers up to about 32 thousand and uses half the space of a regular `INT`.

#### Purposes

- To store small counts, flags, or codes where the value cannot exceed 32,767.
- To reduce row width on very large tables where every byte matters.
- To represent compact enumerated values (e.g., status codes, country numeric codes).

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name SMALLINT
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `SMALLINT` | 2-byte signed integer type. |

**Syntax rules:**

- The range is fixed at −32,768 to +32,767; no length or precision modifier is accepted.
- In MySQL, `SMALLINT UNSIGNED` extends the positive range to 65,535.

**Constraints and limitations:**

- Values outside the range raise a numeric overflow error.
- In PostgreSQL, `SMALLINT` is generally only worth using when storage is constrained.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – SMALLINT column

```sql
-- Step 1: Create a table with a SMALLINT column.
CREATE TABLE flags (id serial PRIMARY KEY, priority smallint);

-- Step 2: Insert a valid value.
INSERT INTO flags (priority) VALUES (100);

-- Step 3: Attempt to insert an out-of-range value.
INSERT INTO flags (priority) VALUES (40000);
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
ERROR:  smallint out of range
```

**Why this output appears**: `40000` exceeds the maximum `SMALLINT` value of 32,767, so PostgreSQL rejects it with an overflow error.

#### Real-World Cases

- **Order priority codes**: A `priority` column with values from 1 to 10 fits comfortably in `SMALLINT`.
- **Year values**: A `birth_year` column covering 1900–2100 fits in `SMALLINT` and saves 2 bytes per row compared with `INTEGER`.

---

### 2. `INTEGER` / `INT`

#### Definitions

**Core Definition**
`INTEGER` (abbreviated `INT`) is the standard 4-byte exact integer type, offering the best balance of range, storage, and performance.

**Technical Definition**
`INTEGER` stores whole numbers from **−2,147,483,648 to +2,147,483,647** and consumes **4 bytes**. PostgreSQL explicitly recommends `integer` as “the common choice, as it offers the best balance between range, storage size, and performance”. The type names `int`, `int4`, and `integer` are synonymous in PostgreSQL.

**Beginner-Friendly Explanation**
`INT` is the everyday whole-number type. It holds numbers up to about 2.1 billion and is the default choice for IDs, counts, and most integer values.

#### Purposes

- To store primary keys, foreign keys, and counters that fit within roughly ±2.1 billion.
- To provide a fast, index-friendly integer type for general-purpose use.
- To serve as the base type for PostgreSQL's `SERIAL` pseudo-type.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name INTEGER
-- or
column_name INT
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `INTEGER` / `INT` | 4-byte signed integer type. |

**Syntax rules:**

- `INT` and `INTEGER` are synonyms in all major RDBMSs.
- No precision or length modifier is accepted.

**Constraints and limitations:**

- Values outside ±2.1 billion cause an overflow error.
- When the table may grow beyond 2.1 billion rows, use `BIGINT` instead.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – INT primary key

```sql
-- Step 1: Create a table with an INT primary key.
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50) NOT NULL
);

-- Step 2: Insert a row.
INSERT INTO users (user_id, username) VALUES (1, 'alice');

-- Step 3: Verify.
SELECT * FROM users;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 1 row affected (0.01 sec)
+---------+----------+
| user_id | username |
+---------+----------+
|       1 | alice    |
+---------+----------+
```

**Why this output appears**: `INT` stores the ID `1` in 4 bytes, and the primary key index uses the compact integer representation.

#### Real-World Cases

- **User IDs**: Most applications use `INT` for surrogate primary keys unless the table is expected to exceed 2.1 billion rows.
- **Counters**: Page views, login counts, and similar metrics fit comfortably in `INT` for most systems.

---

### 3. `BIGINT`

#### Definitions

**Core Definition**
`BIGINT` is an 8-byte exact integer type used when the range of `INTEGER` is insufficient.

**Technical Definition**
`BIGINT` stores whole numbers from **−9,223,372,036,854,775,808 to +9,223,372,036,854,775,807** and consumes **8 bytes**. PostgreSQL states that `bigint` is designed for use when the range of `integer` is insufficient.

**Beginner-Friendly Explanation**
`BIGINT` is the giant whole-number container. It holds numbers up to about 9.2 quintillion, which is far more than most applications will ever need—but it uses twice the space of an `INT`.

#### Purposes

- To store IDs or counters that may exceed 2.1 billion.
- To provide a safe key type for high-volume tables (e.g., event logs, IoT telemetry).
- To avoid overflow in applications with unpredictable growth.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name BIGINT
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `BIGINT` | 8-byte signed integer type. |

**Syntax rules:**

- No precision or length modifier is accepted.
- In MySQL, `BIGINT UNSIGNED` extends the positive range to approximately 1.8 × 10¹⁹.

**Constraints and limitations:**

- `BIGINT` consumes twice the storage of `INT`, increasing index size and reducing rows per page.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – BIGINT for a high-volume ID

```sql
-- Step 1: Create a table with a BIGINT primary key.
CREATE TABLE events (
    event_id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    event_type text NOT NULL
);

-- Step 2: Insert a row.
INSERT INTO events (event_type) VALUES ('login') RETURNING event_id;
```

**Expected output:**

```text
CREATE TABLE
 event_id
----------
        1
(1 row)
```

**Why this output appears**: `BIGINT` accommodates up to 9.2 quintillion events; the `IDENTITY` column generates sequential values without risk of overflow in realistic timeframes.

#### Real-World Cases

- **Event streaming**: A high-volume event table uses `BIGINT` for its primary key to avoid overflow as rows accumulate.
- **Financial transaction IDs**: A ledger table uses `BIGINT` when transaction volumes may exceed the `INT` range over the system's lifetime.

---

### 4. `DECIMAL` / `NUMERIC`

#### Definitions

**Core Definition**
`DECIMAL` and `NUMERIC` are exact numeric types that store values with a user-specified precision and scale, avoiding the rounding errors of floating-point types.

**Technical Definition**
`NUMERIC(p, s)` and `DECIMAL(p, s)` store numbers with `p` total significant digits and `s` digits after the decimal point. In PostgreSQL, `decimal` and `numeric` are equivalent, with a maximum of **131,072 digits before** and **16,383 digits after** the decimal point, and storage is **variable** (approximately 2 bytes per group of 4 decimal digits plus overhead). MySQL treats `DECIMAL` and `NUMERIC` as the same type and stores values as strings to preserve exact decimal precision, using binary packing of 9 digits per 4 bytes. SQL Server caps precision at 38 digits, with storage ranging from 5 to 17 bytes depending on precision.

**Beginner-Friendly Explanation**
`DECIMAL` is the exact-money type. Unlike `FLOAT`, it stores `19.99` as exactly `19.99`, not `19.989999...`. You tell it how many total digits and how many decimal places you need, and it stores the value precisely.

#### Purposes

- To store monetary amounts where exactness is required.
- To store measurements or quantities that must not suffer binary floating-point rounding.
- To enforce a defined precision and scale as part of the schema contract.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name { DECIMAL | NUMERIC } (precision, scale)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `precision` / `p` | Total number of significant digits. |
| `scale` / `s` | Number of digits after the decimal point. |

**Syntax rules:**

- Scale must be between 0 and precision.
- Omitting precision and scale in SQL Server defaults to `DECIMAL(18,0)`, consuming 9 bytes per row.
- PostgreSQL allows very large precision (up to 131,072 digits before the decimal point) with variable storage.

**Constraints and limitations:**

- Values exceeding the declared precision raise an overflow error.
- PostgreSQL's `NUMERIC` arithmetic is slower than integer or floating-point arithmetic.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – NUMERIC for monetary amounts

```sql
-- Step 1: Create a table with a NUMERIC(10,2) column.
CREATE TABLE invoices (id int PRIMARY KEY, amount numeric(10,2));

-- Step 2: Insert a value.
INSERT INTO invoices VALUES (1, 1234.56);

-- Step 3: Insert an out-of-range value.
INSERT INTO invoices VALUES (2, 123456789.00);
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
ERROR:  numeric field overflow
DETAIL:  A field with precision 10, scale 2 must round to an absolute value less than 10^8.
```

**Why this output appears**: `numeric(10,2)` allows 8 digits before the decimal point and 2 after. The second value has 9 digits before the decimal point, exceeding the precision and causing an overflow.

#### Real-World Cases

- **Financial ledgers**: Monetary amounts are stored as `NUMERIC(19,4)` or `DECIMAL(19,4)` to preserve exact values for interest and tax calculations.
- **Scientific measurements**: A sensor-reading table uses `NUMERIC(10,4)` when readings must be stored without binary rounding error.

---

### 5. `REAL`

#### Definitions

**Core Definition**
`REAL` is a 4-byte approximate numeric type that stores single-precision floating-point values.

**Technical Definition**
`REAL` provides at least **6 decimal digits of precision** and consumes **4 bytes**. MySQL describes `REAL` as a synonym for `DOUBLE` by default (unless `REAL_AS_FLOAT` SQL mode is enabled) and as a single-precision floating-point type. In SQL Server, `REAL` is functionally equivalent to `FLOAT(24)`.

**Beginner-Friendly Explanation**
`REAL` is a compact scientific-number container. It stores numbers with decimal points, but the value may be a very close approximation rather than an exact decimal. It is useful for measurements where tiny rounding errors do not matter.

#### Purposes

- To store scientific measurements or sensor readings where approximate precision is acceptable.
- To save storage compared with `DOUBLE PRECISION` when 6 digits of precision suffice.
- To represent values that naturally span a very large dynamic range.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name REAL
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `REAL` | 4-byte single-precision floating-point type. |

**Syntax rules:**

- `REAL` is an approximate type; it does not guarantee exact decimal representation.
- In MySQL, `REAL` may map to `DOUBLE` depending on SQL mode.

**Constraints and limitations:**

- `REAL` cannot exactly represent fractions such as 0.1; comparisons may require tolerance ranges.
- Do not use `REAL` for monetary values or any domain requiring exactness.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – REAL for sensor readings

```sql
-- Step 1: Create a table with a REAL column.
CREATE TABLE readings (id serial PRIMARY KEY, temperature real);

-- Step 2: Insert a value and observe the stored representation.
INSERT INTO readings (temperature) VALUES (23.456) RETURNING temperature;
```

**Expected output:**

```text
CREATE TABLE
 temperature
-------------
      23.456
(1 row)
```

**Why this output appears**: `REAL` stores the value as a 4-byte float. The displayed value may appear exact for this particular number, but the underlying representation is an approximation; for example, `0.1` would not be stored exactly.

#### Real-World Cases

- **Environmental sensors**: Temperature and humidity readings are often stored as `REAL` because the sensors themselves have limited precision.
- **Scientific telemetry**: Values spanning many orders of magnitude (e.g., pressure readings) fit naturally in a floating-point type.

---

### 6. `DOUBLE PRECISION`

#### Definitions

**Core Definition**
`DOUBLE PRECISION` is an 8-byte approximate numeric type that provides roughly twice the precision of `REAL`.

**Technical Definition**
`DOUBLE PRECISION` provides **15 decimal digits of precision** and consumes **8 bytes**. In SQL Server, `FLOAT(53)` maps to `DOUBLE PRECISION`-equivalent precision, consuming 8 bytes. MySQL's `DOUBLE` is the corresponding 8-byte double-precision type.

**Beginner-Friendly Explanation**
`DOUBLE PRECISION` is the high-precision scientific-number container. It holds more digits than `REAL` and is the standard choice for floating-point calculations that need more accuracy, but it still cannot represent every decimal fraction exactly.

#### Purposes

- To store scientific calculations requiring more than 6 decimal digits of precision.
- To perform floating-point arithmetic where the dynamic range exceeds what `REAL` can represent.
- To serve as the default approximate numeric type when precision requirements are not extreme.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name DOUBLE PRECISION
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `DOUBLE PRECISION` | 8-byte double-precision floating-point type. |

**Syntax rules:**

- `DOUBLE PRECISION` is an approximate type; exact decimal representation is not guaranteed.
- In MySQL, `DOUBLE` and `DOUBLE PRECISION` are synonyms.

**Constraints and limitations:**

- Even with 15 digits of precision, `DOUBLE PRECISION` cannot exactly represent values such as `0.1`.
- For monetary or other exact-value domains, use `NUMERIC`/`DECIMAL` instead.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – DOUBLE PRECISION for scientific data

```sql
-- Step 1: Create a table with a DOUBLE PRECISION column.
CREATE TABLE measurements (id serial PRIMARY KEY, value double precision);

-- Step 2: Insert a high-precision value.
INSERT INTO measurements (value) VALUES (1.234567890123456) RETURNING value;
```

**Expected output:**

```text
CREATE TABLE
        value
---------------------
 1.234567890123456
(1 row)
```

**Why this output appears**: `DOUBLE PRECISION` preserves 15 significant digits, enough to store and return the inserted value with high fidelity. The value is still an approximation in the IEEE 754 sense, but the precision is sufficient for most scientific applications.

#### Real-World Cases

- **Physics simulations**: Coordinates, velocities, and other physical quantities use `DOUBLE PRECISION` for their wide range and 15-digit precision.
- **Statistical computations**: Averages, variances, and regression coefficients are commonly stored as `DOUBLE PRECISION`.

---

### 7. `SMALLSERIAL`

#### Definitions

**Core Definition**
`SMALLSERIAL` is a PostgreSQL pseudo-type that creates a 2-byte auto-incrementing integer column with an owned sequence.

**Technical Definition**
`SMALLSERIAL` is not a true data type; it is shorthand for creating a `SMALLINT NOT NULL DEFAULT nextval('sequence_name')` column, along with an owned sequence object. The range is **1 to 32,767**, and the storage size is **2 bytes**.

**Beginner-Friendly Explanation**
`SMALLSERIAL` is a shortcut for “a small auto-numbered column.” PostgreSQL automatically creates a counter behind the scenes and fills the column with the next number whenever you insert a row.

#### Purposes

- To create compact auto-incrementing primary keys for small tables (fewer than about 32,000 rows).
- To avoid manually defining a sequence and a default for small lookup tables.
- To save 2 bytes per row compared with `SERIAL`.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name SMALLSERIAL
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `SMALLSERIAL` | Expands to `SMALLINT NOT NULL DEFAULT nextval(...)`. |

**Syntax rules:**

- `SMALLSERIAL` automatically creates a sequence owned by the column.
- The sequence is dropped when the column or table is dropped.

**Constraints and limitations:**

- `SMALLSERIAL` is PostgreSQL-specific and is not standard SQL.
- PostgreSQL recommends `GENERATED ... AS IDENTITY` over `SERIAL` types for new tables.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – SMALLSERIAL primary key

```sql
-- Step 1: Create a table with a SMALLSERIAL primary key.
CREATE TABLE lookups (id smallserial PRIMARY KEY, name text);

-- Step 2: Insert rows.
INSERT INTO lookups (name) VALUES ('a'), ('b') RETURNING id;
```

**Expected output:**

```text
CREATE TABLE
 id
----
  1
  2
(2 rows)
```

**Why this output appears**: `SMALLSERIAL` creates a sequence and a default; each insert draws the next value (1, then 2) from the sequence.

#### Real-World Cases

- **Small lookup tables**: A `status_codes` table with a few dozen rows uses `SMALLSERIAL` for a compact auto-incrementing key.
- **Temporary staging tables**: A small staging table uses `SMALLSERIAL` when row counts are guaranteed to stay well below 32,000.

---

### 8. `SERIAL`

#### Definitions

**Core Definition**
`SERIAL` is a PostgreSQL pseudo-type that creates a 4-byte auto-incrementing integer column, the most common choice for auto-generated primary keys.

**Technical Definition**
`SERIAL` expands to `INTEGER NOT NULL DEFAULT nextval('sequence_name')` with an owned sequence. Its range is **1 to 2,147,483,647**, and its storage size is **4 bytes**. The underlying type is `INTEGER`.

**Beginner-Friendly Explanation**
`SERIAL` is the standard “auto-numbered ID” shortcut in PostgreSQL. It creates an integer column that automatically fills with the next number on every insert.

#### Purposes

- To create auto-incrementing primary keys for most ordinary tables.
- To avoid writing the sequence and default definitions manually.
- To provide a compact, fast key type for tables expected to stay below 2.1 billion rows.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name SERIAL
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `SERIAL` | Expands to `INTEGER NOT NULL DEFAULT nextval(...)`. |

**Syntax rules:**

- `SERIAL` creates an owned sequence; the sequence is dropped with the column.
- `SERIAL` does not prevent explicit inserts unless a `UNIQUE` or `PRIMARY KEY` constraint is also present.

**Constraints and limitations:**

- `SERIAL` values may have gaps due to rolled-back transactions or failed inserts; this is normal.
- PostgreSQL recommends `GENERATED ALWAYS AS IDENTITY` for new tables because it is SQL standard and provides better control.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – SERIAL primary key

```sql
-- Step 1: Create a table with a SERIAL primary key.
CREATE TABLE posts (id serial PRIMARY KEY, title text NOT NULL);

-- Step 2: Insert rows.
INSERT INTO posts (title) VALUES ('First'), ('Second') RETURNING id, title;
```

**Expected output:**

```text
CREATE TABLE
 id | title
----+--------
  1 | First
  2 | Second
(2 rows)
```

**Why this output appears**: `SERIAL` creates a sequence and a default; the first insert receives `1`, the second receives `2`, and `RETURNING` shows the generated IDs.

#### Real-World Cases

- **Blog posts**: A `posts` table uses `SERIAL` for its primary key, with IDs generated automatically on insert.
- **Order headers**: An `orders` table uses `SERIAL` for `order_id` when order volume is expected to stay within the `INT` range.

---

### 9. `BIGSERIAL`

#### Definitions

**Core Definition**
`BIGSERIAL` is a PostgreSQL pseudo-type that creates an 8-byte auto-incrementing integer column for very large tables.

**Technical Definition**
`BIGSERIAL` expands to `BIGINT NOT NULL DEFAULT nextval('sequence_name')` with an owned sequence. Its range is **1 to 9,223,372,036,854,775,807**, and its storage size is **8 bytes**. The underlying type is `BIGINT`.

**Beginner-Friendly Explanation**
`BIGSERIAL` is the “giant auto-numbered ID” shortcut. It creates a `BIGINT` column that automatically fills with the next number, suitable for tables that may grow beyond 2.1 billion rows.

#### Purposes

- To create auto-incrementing primary keys for high-volume tables.
- To provide a safe key type for distributed systems or event streams where the row count may exceed the `INT` range.
- To avoid the risk of exhausting a 32-bit sequence.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name BIGSERIAL
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `BIGSERIAL` | Expands to `BIGINT NOT NULL DEFAULT nextval(...)`. |

**Syntax rules:**

- `BIGSERIAL` creates an owned sequence, just like `SERIAL`.
- It consumes 8 bytes per value, twice the storage of `SERIAL`.

**Constraints and limitations:**

- PostgreSQL now recommends `BIGINT GENERATED ALWAYS AS IDENTITY` over `BIGSERIAL` for new tables.
- Distributed systems often use `BIGSERIAL` for all tables because 32-bit sequences are too small for global scale.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – BIGSERIAL for an event table

```sql
-- Step 1: Create a high-volume table with BIGSERIAL.
CREATE TABLE events (event_id bigserial PRIMARY KEY, payload jsonb);

-- Step 2: Insert rows.
INSERT INTO events (payload) VALUES ('{"type":"click"}'), ('{"type":"view"}') RETURNING event_id;
```

**Expected output:**

```text
CREATE TABLE
 event_id
----------
        1
        2
(2 rows)
```

**Why this output appears**: `BIGSERIAL` creates a `BIGINT` sequence and default; each insert receives the next 64-bit integer, with no realistic risk of overflow.

#### Real-World Cases

- **Event logging**: A high-throughput event table uses `BIGSERIAL` for its primary key to accommodate billions of rows.
- **Distributed tables**: In systems where multiple nodes generate rows, `BIGSERIAL` (or `BIGINT IDENTITY`) provides a globally unique 64-bit identifier.

---

### 10. Numeric Type Best Practices

#### Definitions

**Core Definition**
Numeric type best practices are the guidelines for selecting, declaring, and using numeric types to ensure correctness, storage efficiency, and query performance.

**Technical Definition**
Best practices include choosing the smallest integer type that safely fits the domain, always specifying precision and scale for `DECIMAL`/`NUMERIC`, avoiding floating-point types for monetary values, preferring SQL-standard `IDENTITY` columns over `SERIAL` pseudo-types, and understanding the storage and performance trade-offs of each type.

**Beginner-Friendly Explanation**
Best practices are the habits that keep numeric data correct and efficient. Pick the smallest container that safely holds your numbers, always say how many digits and decimal places you need, and never use approximate floats for money.

#### Purposes

- To prevent overflow, truncation, and rounding errors.
- To minimise storage consumption and improve cache and I/O efficiency.
- To ensure portability and maintainability across RDBMSs.
- To align with modern SQL standards where possible.

#### Syntax Rules and Structure

**Best-practice guidance:**

- **Choose the smallest integer type that fits the domain**: use `SMALLINT` for values up to 32K, `INT` for values up to 2.1 billion, and `BIGINT` only when the range may exceed that.
- **Always specify precision and scale for `DECIMAL`/`NUMERIC`**: omitting them in SQL Server defaults to `DECIMAL(18,0)`, consuming 9 bytes per row unnecessarily. Use `DECIMAL(9,4)` or `DECIMAL(19,4)` for efficient storage when four decimal places are needed.
- **Avoid `FLOAT`/`REAL` for monetary values**: use `DECIMAL`/`NUMERIC` to avoid binary rounding errors that propagate into ledger mismatches.
- **Prefer `IDENTITY` over `SERIAL` in PostgreSQL**: `GENERATED ALWAYS AS IDENTITY` is SQL standard, prevents accidental manual inserts, and is the recommended choice for new tables. Use `BIGSERIAL` or `BIGINT IDENTITY` for tables that may exceed 2 billion rows.
- **Use integer types for identifiers and counters**: prefer `SMALLINT`, `INT`, or `BIGINT` over `DECIMAL` for whole numbers, because integer arithmetic is faster and indexes more efficiently.
- **Use `NUMERIC` only when exactness is required**: `NUMERIC` arithmetic is slower than integer or floating-point arithmetic, so reserve it for monetary and other exact-value domains.

**Constraints and limitations:**

- Changing a column's numeric type later can require a full table rewrite and may be disruptive on large tables.
- Over-optimising storage with a type that is too small risks overflow as the application grows.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Best-practice numeric schema

```sql
-- Step 1: Create a table applying numeric best practices.
CREATE TABLE orders (
    order_id     bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id  integer NOT NULL,
    total_amount numeric(12,2) NOT NULL,
    item_count   smallint NOT NULL DEFAULT 0,
    created_at   timestamptz NOT NULL DEFAULT now()
);
```

**Expected output:**

```text
CREATE TABLE
```

**Why this output appears**: The schema uses `BIGINT IDENTITY` for a future-proof primary key, `INTEGER` for the customer ID, `NUMERIC(12,2)` for exact monetary amounts, `SMALLINT` for a small item count, and `TIMESTAMPTZ` for time-zone-aware timestamps—each choice reflecting the best practice for its domain.

#### Real-World Cases

- **Financial system**: `DECIMAL(19,4)` stores balances exactly; `BIGINT` stores transaction IDs; `SMALLINT` stores status codes.
- **High-volume event store**: `BIGINT IDENTITY` provides a globally unique, future-proof key; `INTEGER` stores event type IDs; `DOUBLE PRECISION` stores measured values where approximation is acceptable.

---

## References

- PostgreSQL Documentation – Numeric Types – https://www.postgresql.org/docs/current/datatype-numeric.html
- PostgreSQL Documentation – Numeric Types (development version) – https://www.postgresql.org/docs/devel/datatype-numeric.html
- MySQL Reference – Numeric Types – http://dev.cs.ovgu.de/db/mysql/Numeric-types.html
- Microsoft Learn – Understand the 9 Numeric Data Types in SQL Server – https://learn.microsoft.com/en-us/previous-versions/technet-magazine/dd424925(v=msdn.10)
- Bytebase – How to Use SERIAL in PostgreSQL – https://www.bytebase.com/reference/postgres/how-to/how-to-use-serial-postgres/
- Crunchy Data – Postgres Serials Should be BIGINT (and How to Migrate) – https://www.crunchydata.com/blog/postgres-serials-should-be-bigint-and-how-to-migrate
- SQLFluff – CV13: Discourage SERIAL in Favor of GENERATED AS IDENTITY – https://github.com/sqlfluff/sqlfluff/pull/7440
- Microsoft Learn – Performance Guidelines (Fabric) – https://learn.microsoft.com/en-us/fabric/data-warehouse/guidelines-warehouse-performance
- Cleverence – SQL Data Types Explained: Numbers, Text, Dates, JSON, and Best Practices – https://www.cleverence.com/articles/oracle-documentation/sql-data-types-4729/
- Databricks – SQL Data Types: Reference and Best Practices – https://www.databricks.com/blog/sql-data-types
- Microsoft Learn – decimal and numeric (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/data-types/decimal-and-numeric-transact-sql
- MySQL 8.0 Reference Manual – Integer Types – https://dev.mysql.com/doc/refman/8.0/en/integer-types.html
- MySQL 8.0 Reference Manual – Fixed-Point Types (DECIMAL) – https://dev.mysql.com/doc/refman/8.0/en/fixed-point-types.html
- MySQL 8.0 Reference Manual – Floating-Point Types – https://dev.mysql.com/doc/refman/8.0/en/floating-point-types.html