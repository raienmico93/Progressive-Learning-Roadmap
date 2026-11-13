# Comprehensive Programming Cheat Sheet: SQL Bit String Types

## Topic Overview

### Definitions

**Core Definition**
Bit string types are the SQL data types—`bit(n)` and `bit varying(n)`—designed to store sequences of binary digits (1s and 0s) as a single value, enabling compact representation of bit masks, flags, and binary-encoded data directly within a database column.

**Technical Definition**
PostgreSQL provides two SQL bit types: `bit(n)` and `bit varying(n)`, where `n` is a positive integer. The `bit` type requires data to match the declared length `n` exactly; shorter or longer bit strings are rejected with an error. The `bit varying` type (also known as `varbit`) stores variable-length strings up to a maximum length `n`, rejecting longer values. Writing `bit` without a length is equivalent to `bit(1)`, while `bit varying` without a length specification means unlimited length. A bit string value requires 1 byte for each group of 8 bits, plus 5 or 8 bytes of overhead depending on the length, and long values may be compressed or moved out-of-line. Bit strings support a rich set of operators—concatenation (`||`), bitwise AND (`&`), OR (`|`), XOR (`#`), NOT (`~`), and shifts (`<<`, `>>`)—as well as SQL-standard functions such as `length`, `bit_length`, `octet_length`, `position`, `substring`, `overlay`, `get_bit`, and `set_bit`.

**Beginner-Friendly Explanation**
A bit string is a sequence of 1s and 0s stored as a single value—like a row of light switches that can each be on or off. PostgreSQL gives you two containers for bit strings: `BIT(n)` is a fixed-size box that demands exactly `n` switches, while `BIT VARYING(n)` is a stretchy box that holds up to `n` switches. You can combine bit strings with operations that work switch-by-switch (AND, OR, XOR, NOT), shift them left or right, concatenate them, and even cast integers to and from bit strings. They are useful when you need to store a compact set of flags (e.g., permissions, feature toggles) or work with binary data at the bit level.

### Key Characteristics

- **Fixed vs. variable length**: `bit(n)` enforces an exact length; `bit varying(n)` allows any length up to `n`. Attempting to store a shorter or longer string in a `bit(n)` column raises an error.
- **Explicit casts truncate or pad silently**: Casting a value to `bit(n)` truncates or zero-pads it on the right without error; casting to `bit varying(n)` truncates on the right if too long.
- **Bitwise operators require equal lengths**: The `&`, `|`, and `#` operators demand operands of equal length; shifting preserves the original string length.
- **Integer casting is bit-order sensitive**: Casting an integer to `bit(n)` copies the rightmost `n` bits; casting to a width wider than the integer sign-extends on the left. Casting to just `bit` yields `bit(1)`, keeping only the least significant bit.
- **Indexing is possible but often inefficient**: Bit string columns can be indexed with B-tree, but queries that select a large fraction of rows (e.g., when half the bits are set) are better served by a full table scan; GiST operator classes for bit strings are not built in.

### Prerequisites

- A running PostgreSQL instance and a client connection.
- Familiarity with `CREATE TABLE` syntax and SQL data types.
- Understanding of bitwise logic (AND, OR, XOR, NOT) and binary representation of integers.
- Awareness of the storage overhead and indexing limitations before choosing bit strings for flag storage.

### Related Programming Areas

- Data Definition Language (DDL) – column type declarations.
- Data Manipulation Language (DML) – inserting, querying, and updating bit string values.
- Bitwise computation – permission masks, feature flags, and binary protocol fields.
- Data compression and encoding – compact representation of boolean sets.
- Application development – mapping SQL bit strings to language-level bit sets or integer flags.

---

## Core Concepts and Features

### 1. `BIT`

#### Definitions

**Core Definition**
`BIT(n)` is a PostgreSQL type that stores a fixed-length string of exactly `n` binary digits (1s and 0s).

**Technical Definition**
`bit(n)` requires data to match the declared length `n` exactly; it is an error to attempt to store shorter or longer bit strings. Writing `bit` without a length is equivalent to `bit(1)`. The type supports the usual comparison operators and the bitwise operators described in the operations section. A `bit` value requires 1 byte per group of 8 bits plus overhead, and long values may be compressed or moved out-of-line.

**Beginner-Friendly Explanation**
`BIT(n)` is the fixed-size box for bit strings. You tell it exactly how many bits you need, and it will only accept strings of that exact length. If you try to insert a shorter or longer string, PostgreSQL rejects it—unless you explicitly cast the value, in which case it is silently padded or truncated.

#### Purposes

- To store fixed-width bit masks where every value must have the same number of bits.
- To represent compact sets of boolean flags in a single column (e.g., permission bits).
- To work with binary protocol fields that have a defined bit width.
- To support bitwise operations and comparisons on fixed-size binary data.

#### Syntax Rules and Structure

**Column declaration:**

```sql
column_name BIT(n)
```

**Bit string literal:**

```sql
B'101'  -- three-bit string
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `BIT(n)` | Fixed-length bit string of exactly `n` bits. |
| `n` | Positive integer specifying the bit length. |
| `B'...'` | Bit string literal notation. |

**Syntax rules:**

- The value must contain exactly `n` bits; otherwise an error is raised.
- Writing `bit` without `(n)` is equivalent to `bit(1)`.
- Explicit casts to `bit(n)` truncate or zero-pad silently to fit the length.

**Constraints and limitations:**

- Values that do not match the declared length are rejected.
- Bitwise AND, OR, and XOR require both operands to have equal lengths.
- Indexing on low-selectivity bit patterns may not be beneficial.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Fixed-length BIT column with error on mismatched length

```sql
-- Step 1: Create a table with a fixed-length BIT(3) column.
CREATE TABLE flags (
    id   int PRIMARY KEY,
    mask bit(3) NOT NULL
);

-- Step 2: Insert a valid three-bit string.
INSERT INTO flags VALUES (1, B'101');

-- Step 3: Attempt to insert a two-bit string (should fail).
INSERT INTO flags VALUES (2, B'10');

-- Step 4: Insert a two-bit string with an explicit cast (pads to three bits).
INSERT INTO flags VALUES (3, B'10'::bit(3));

-- Step 5: Query the stored values.
SELECT id, mask FROM flags ORDER BY id;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
ERROR:  bit string length 2 does not match type bit(3)
INSERT 0 1
 id | mask
----+------
  1 | 101
  3 | 100
(2 rows)
```

**Why this output appears**: The direct insert of `B'10'` into a `bit(3)` column fails because the length does not match. The explicit cast `B'10'::bit(3)` zero-pads the value on the right to produce `100`, which is accepted. The stored values are `101` and `100`.

#### Real-World Cases

- **Permission masks**: A `roles` table stores a `bit(8)` permission mask, where each bit represents a different permission (read, write, delete, etc.).
- **Protocol fields**: A `network_packets` table stores fixed-width header fields as `bit(n)` for bit-level parsing.
- **Feature flags**: A `users` table uses `bit(16)` to store sixteen independent boolean feature toggles in a single column.

---

### 2. `BIT VARYING`

#### Definitions

**Core Definition**
`BIT VARYING(n)` (also called `VARBIT(n)`) is a PostgreSQL type that stores a variable-length string of binary digits up to a maximum of `n` bits.

**Technical Definition**
`bit varying` data is of variable length up to the maximum length `n`; longer strings are rejected. Writing `bit varying` without a length specification means unlimited length. Values shorter than `n` are stored as-is without padding. Explicit casts to `bit varying(n)` truncate on the right if the value exceeds `n` bits.

**Beginner-Friendly Explanation**
`BIT VARYING(n)` is the stretchy box for bit strings. You set a maximum size, but the actual stored string can be shorter. It is useful when the number of bits varies from row to row—for example, when representing different-length codes or dynamically sized flag sets.

#### Purposes

- To store variable-length bit strings where the number of bits differs between rows.
- To avoid the strict length enforcement of `bit(n)` when values may legitimately vary in size.
- To support concatenation and bitwise operations on dynamically sized binary data.

#### Syntax Rules and Structure

**Column declaration:**

```sql
column_name BIT VARYING(n)
-- or
column_name VARBIT(n)
```

**Bit string literal:**

```sql
B'1011'  -- variable-length bit string
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `BIT VARYING(n)` | Variable-length bit string with maximum length `n`. |
| `n` | Optional maximum length; omitted means unlimited. |
| `VARBIT` | Short alias for `bit varying`. |

**Syntax rules:**

- Values longer than `n` are rejected.
- Explicit casts to `bit varying(n)` truncate values that exceed `n` bits.
- `bit varying` without a length accepts strings of any length.

**Constraints and limitations:**

- Bitwise AND, OR, and XOR still require equal-length operands.
- Very long `varbit` values may be stored out-of-line, affecting retrieval performance.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Variable-length BIT VARYING column

```sql
-- Step 1: Create a table with a BIT VARYING(8) column.
CREATE TABLE codes (
    id   int PRIMARY KEY,
    code bit varying(8) NOT NULL
);

-- Step 2: Insert values of different lengths.
INSERT INTO codes VALUES (1, B'1');
INSERT INTO codes VALUES (2, B'101');
INSERT INTO codes VALUES (3, B'11110000');

-- Step 3: Attempt to insert a nine-bit value (should fail).
INSERT INTO codes VALUES (4, B'111111111');

-- Step 4: Query the stored values.
SELECT id, code FROM codes ORDER BY id;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
INSERT 0 1
INSERT 0 1
ERROR:  bit string length 9 exceeds type bit varying(8)
 id |   code
----+----------
  1 | 1
  2 | 101
  3 | 11110000
(3 rows)
```

**Why this output appears**: `bit varying(8)` accepts values of one, three, and eight bits because each is at most eight. The nine-bit value exceeds the maximum length and is rejected. Shorter values are stored without padding.

#### Real-World Cases

- **Variable-length codes**: A `coupons` table stores redemption codes of different bit lengths as `varbit`.
- **Dynamic flag sets**: A `sessions` table stores a variable number of session flags depending on the user's role.
- **Compressed identifiers**: A `documents` table stores binary identifiers whose bit length depends on the document type.

---

### 3. Bit String Operations

#### Definitions

**Core Definition**
Bit string operations are the operators and functions that combine, compare, shift, extract, and modify bit strings, including concatenation, bitwise AND/OR/XOR/NOT, shifts, `length()`, `substring()`, `get_bit()`, and `set_bit()`.

**Technical Definition**
PostgreSQL provides a rich set of operators for bit strings. Concatenation (`||`) joins two bit strings. Bitwise AND (`&`), OR (`|`), and XOR (`#`) require inputs of equal length. Bitwise NOT (`~`) inverts every bit. Left shift (`<<`) and right shift (`>>`) preserve the original string length. The SQL-standard functions `length`, `bit_length`, `octet_length`, `position`, `substring`, and `overlay` work on bit strings as well as character strings. The functions `get_bit` and `set_bit` number bits from the right, with the first (rightmost) bit as bit 0. Integers can be cast to and from bit strings; casting to `bit(n)` copies the rightmost `n` bits and sign-extends when the target width exceeds the integer's width.

**Beginner-Friendly Explanation**
Bit string operations are the tools for manipulating binary data. You can glue two bit strings together (`||`), combine them bit by bit with AND, OR, or XOR, flip every bit with NOT, slide them left or right, measure their length, extract a slice, or read and write individual bits. You can also convert integers to and from bit strings, which is useful when you want to store numeric flags as bits.

#### Purposes

- To combine or compare bit masks using bitwise logic.
- To extract or modify specific bits within a binary value.
- To shift bits for encoding, decoding, or alignment operations.
- To convert between integer representations and bit string representations.

#### Syntax Rules and Structure

**Bitwise operators:**

| Operator | Description | Example |
|----------|-------------|---------|
| `||` | Concatenation | `B'10001' || B'011'` → `10001011` |
| `&` | Bitwise AND (equal lengths) | `B'10001' & B'01101'` → `00001` |
| `|` | Bitwise OR (equal lengths) | `B'10001' | B'01101'` → `11101` |
| `#` | Bitwise XOR (equal lengths) | `B'10001' # B'01101'` → `11100` |
| `~` | Bitwise NOT | `~ B'10001'` → `01110` |
| `<<` | Shift left (length preserved) | `B'10001' << 3` → `01000` |
| `>>` | Shift right (length preserved) | `B'10001' >> 2` → `00100` |

**Functions:**

| Function | Description |
|----------|-------------|
| `length(bit)` | Number of bits |
| `bit_length(bit)` | Number of bits (same as `length`) |
| `octet_length(bit)` | Number of bytes |
| `substring(bit, start, count)` | Extract a bit slice |
| `get_bit(bit, n)` | Read bit at position `n` (rightmost is 0) |
| `set_bit(bit, n, value)` | Set bit at position `n` |

**Syntax rules:**

- Bitwise AND, OR, and XOR require operands of equal length.
- Shifts preserve the original string length.
- `get_bit` and `set_bit` number bits from the right, starting at 0.

**Constraints and limitations:**

- Bit strings of different lengths cannot be combined with `&`, `|`, or `#`.
- Casting integers to `bit(n)` truncates to the rightmost `n` bits.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Bitwise operations and functions

```sql
-- Step 1: Perform bitwise AND, OR, and XOR on equal-length operands.
SELECT B'10001' & B'01101' AS and_result,
       B'10001' | B'01101' AS or_result,
       B'10001' # B'01101' AS xor_result;

-- Step 2: Apply bitwise NOT and shifts.
SELECT ~ B'10001' AS not_result,
       B'10001' << 3 AS shift_left,
       B'10001' >> 2 AS shift_right;

-- Step 3: Use functions to inspect and modify bits.
SELECT length(B'10001') AS bits,
       get_bit(B'10001', 0) AS rightmost_bit,
       set_bit(B'10001', 2, 1) AS set_bit_result;

-- Step 4: Cast an integer to a bit string.
SELECT 44::bit(10) AS int_to_bit,
       44::bit(3) AS truncated_bit,
       B'1110'::bit(4)::integer AS bit_to_int;
```

**Expected output:**

```text
 and_result | or_result | xor_result
------------+-----------+------------
 00001      | 11101     | 11100
(1 row)

 not_result | shift_left | shift_right
------------+------------+-------------
 01110      | 01000      | 00100
(1 row)

 bits | rightmost_bit | set_bit_result
------+---------------+----------------
    5 |             1 | 10101
(1 row)

 int_to_bit | truncated_bit | bit_to_int
------------+---------------+------------
 0000101100 | 100           |         14
(1 row)
```

**Why this output appears**: Bitwise AND, OR, and XOR combine the bits position by position. NOT inverts every bit. Shifts move bits left or right while preserving length. `get_bit(B'10001', 0)` reads the rightmost bit (1), and `set_bit` sets bit 2 to 1. Casting `44` to `bit(10)` yields `0000101100` (the rightmost 10 bits), while `44::bit(3)` yields `100` (the rightmost 3 bits). Casting `B'1110'` to integer yields 14.

#### Real-World Cases

- **Permission checking**: A `roles` table stores permission masks; queries use `mask & B'1000'` to test whether the “admin” bit is set.
- **Flag combination**: A `users` table combines feature flag bit strings with `|` to compute a user's effective feature set.
- **Binary protocol parsing**: A `packets` table extracts header fields with `substring()` and tests individual flag bits with `get_bit()`.

---

### 4. Bit String Best Practices

#### Definitions

**Core Definition**
Bit string best practices are the guidelines for choosing between `bit` and `bit varying`, indexing bit string columns, using bitwise operations safely, and deciding when bit strings are the right tool versus alternative representations.

**Technical Definition**
Best practices include: (1) **use `bit(n)` for fixed-width masks and `bit varying(n)` for variable-length data**, matching the type to the domain; (2) **use the `B'...'` literal syntax** for clarity rather than casting strings; (3) **remember that explicit casts truncate or pad silently**, so validate lengths explicitly when correctness matters; (4) **use bitwise operators for flag testing** (`mask & B'1000'`), but be aware that indexing bit string columns is often inefficient for low-selectivity patterns—a full table scan may be faster when many rows match; (5) **consider `integer` or `bigint` for bit masks up to 32 or 64 bits** and cast to `bit(n)` only when bit-level operations or fixed-width representations are required; and (6) **use `varbit` only when the bit length genuinely varies**, because fixed-width `bit(n)` enforces integrity and avoids unexpected padding.

**Beginner-Friendly Explanation**
Best practices for bit strings are about using the right container and not tripping over silent casts. Use `BIT(n)` when every value must have the same number of bits, and `BIT VARYING(n)` when the length can vary. Prefer the `B'...'` syntax for readability. Be careful with casts—they silently pad or truncate. For flag storage, integers are often simpler and faster to index, but bit strings shine when you need true bit-level operations on fixed-width data.

#### Purposes

- To avoid silent data corruption caused by implicit padding or truncation.
- To choose the most efficient representation for flag and mask data.
- To write readable and maintainable bitwise queries.
- To understand when indexing bit strings is worthwhile and when it is not.

#### Syntax Rules and Structure

**Best-practice schema pattern:**

```sql
-- Fixed-width permission mask (8 bits).
CREATE TABLE roles (
    role_id   int PRIMARY KEY,
    role_name text NOT NULL,
    perms     bit(8) NOT NULL DEFAULT B'00000000'
);

-- Variable-length code (up to 16 bits).
CREATE TABLE coupons (
    coupon_id int PRIMARY KEY,
    code      bit varying(16) NOT NULL
);
```

**Best-practice query patterns:**

```sql
-- Test whether the admin bit (bit 3) is set.
SELECT role_name FROM roles WHERE perms & B'00001000' = B'00001000';

-- Combine two masks.
SELECT perms | B'00000001' AS updated_perms FROM roles WHERE role_id = 1;
```

**Component breakdown:**

| Practice | Implementation |
|----------|----------------|
| Fixed vs. variable | `bit(n)` for masks; `bit varying(n)` for varying codes |
| Literals | Use `B'...'` instead of string casts |
| Cast caution | Validate lengths before explicit casts |
| Flag testing | `mask & B'...' = B'...'` |
| Indexing | B-tree possible; full scan often better for low selectivity |

**Constraints and limitations:**

- Bitwise AND, OR, and XOR require equal-length operands.
- Indexing bit strings is often ineffective for queries that match a large fraction of rows.
- Casting to `bit(n)` silently truncates or pads, which can hide data errors.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Best-practice permission mask with bitwise testing

```sql
-- Step 1: Create a roles table with an 8-bit permission mask.
CREATE TABLE roles (
    role_id   int PRIMARY KEY,
    role_name text NOT NULL,
    perms     bit(8) NOT NULL
);

-- Step 2: Insert roles with different permission bits.
INSERT INTO roles VALUES
    (1, 'admin',  B'11111111'),
    (2, 'editor', B'00001111'),
    (3, 'viewer', B'00000001');

-- Step 3: Find all roles that have the “write” bit (bit 3) set.
SELECT role_name
FROM roles
WHERE perms & B'00001000' = B'00001000';

-- Step 4: Add the “write” bit to the viewer role.
UPDATE roles
SET perms = perms | B'00001000'
WHERE role_name = 'viewer';

-- Step 5: Verify the updated mask.
SELECT role_name, perms FROM roles WHERE role_name = 'viewer';
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 3
 role_name
-----------
 admin
 editor
(2 rows)
UPDATE 1
 role_name |  perms
-----------+---------
 viewer    | 00001001
(1 row)
```

**Why this output appears**: The `WHERE perms & B'00001000' = B'00001000'` clause tests whether bit 3 is set in each role's mask. `admin` and `editor` have the bit set; `viewer` does not. The `UPDATE` uses bitwise OR to set the bit, turning `00000001` into `00001001`. This pattern is the canonical way to test and manipulate flag bits in PostgreSQL.

#### Real-World Cases

- **Access control**: A `roles` table stores permission masks as `bit(8)` and queries use bitwise AND to test specific permissions without expanding the mask into separate boolean columns.
- **Feature toggles**: A `users` table stores a `bit(16)` feature mask; queries combine masks with `|` to compute the effective feature set for a user.
- **Protocol parsing**: A `packets` table stores fixed-width header fields as `bit(n)` and uses `get_bit()` and `substring()` to decode flags and fields.

---

## References

- PostgreSQL Documentation – Bit String Types (PostgreSQL 16) – https://www.postgresql.org/docs/16/datatype-bit.html
- PostgreSQL Documentation – Bit String Functions and Operators (PostgreSQL 15) – https://www.postgresql.org/docs/15/functions-bitstring.html
- PostgreSQL Documentation – Bit String Functions and Operators (PostgreSQL 12) – https://www.postgresql.org/docs/12/functions-bitstring.html
- PostgreSQL Documentation – Bit String Types (PostgreSQL 18 development) – https://www.postgresql.org/docs/devel/datatype-bit.html
- Stack Overflow – Postgres casting to `bit` vs. `varbit` vs. `"bit"` – https://stackoverflow.com/questions/…
- PostgreSQL Mailing List – Indexing a Bit String column – https://www.postgresql.org/message-id/flat/…