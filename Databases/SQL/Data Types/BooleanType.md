# Comprehensive Programming Cheat Sheet: SQL Boolean Type

## Topic Overview

### Definitions

**Core Definition**
The Boolean type is the SQL data type used to store truth values—`TRUE`, `FALSE`, and the special `UNKNOWN` state represented by `NULL`—enabling columns and expressions to represent binary states, flags, and logical conditions directly within the database.

**Technical Definition**
SQL's Boolean type is a three-valued logic system: a Boolean expression or column can evaluate to `TRUE`, `FALSE`, or `UNKNOWN` (the latter represented by `NULL`). The SQL standard defines the `BOOLEAN` type, and PostgreSQL implements it natively as a 1-byte type whose output is rendered as `t` or `f`. MySQL does not have a native Boolean type; `BOOLEAN` and `BOOL` are synonyms for `TINYINT(1)`, where `0` is false and any non-zero value is true. SQL Server uses the `BIT` type, which stores `1`, `0`, or `NULL`, and packs up to 8 `BIT` columns into a single byte for storage efficiency. Boolean logic in SQL is governed by three-valued truth tables, in which `NULL` propagates through comparisons and logical operators, and `WHERE` clauses retain only rows for which the predicate evaluates strictly to `TRUE`.

**Beginner-Friendly Explanation**
A Boolean type is the database's way of storing a yes/no answer. `TRUE` means yes, `FALSE` means no, and `NULL` means “I don't know” or “not applicable.” PostgreSQL has a real Boolean type, MySQL fakes it using a tiny integer, and SQL Server uses a special bit. The tricky part is that `NULL` is not the same as false—it is a third state, “unknown,” and it changes how queries behave when you filter or compare values.

### Key Characteristics

- **Three-valued logic is fundamental**: SQL Boolean logic is not two-valued. A comparison involving `NULL` produces `UNKNOWN`, not `FALSE`. `NULL = NULL` evaluates to `UNKNOWN`, and `WHERE col = NULL` never returns rows.
- **`WHERE` keeps only `TRUE`**: Rows for which the predicate evaluates to `FALSE` or `UNKNOWN` are both discarded, even though the two states are logically distinct.
- **Vendor implementations differ substantially**: PostgreSQL has a native `BOOLEAN` type (1 byte, output `t`/`f`); MySQL maps `BOOLEAN` to `TINYINT(1)` and permits any integer value; SQL Server uses `BIT` and bit-packs columns for storage efficiency.
- **Boolean values are not always what they seem**: In MySQL, a `BOOLEAN` column can store `2`, `-5`, or any other `TINYINT` value; only `0` is false, and any non-zero value is true. A `CHECK (col IN (0,1))` constraint is required to enforce true Boolean semantics.
- **Naming conventions matter for readability**: Boolean columns should read as yes/no questions, typically with `is_`, `has_`, or `can_` prefixes (e.g., `is_active`, `has_permission`, `can_edit`).

### Prerequisites

- A running RDBMS instance (PostgreSQL, MySQL, or SQL Server) and a client connection.
- Familiarity with `CREATE TABLE` and column definitions.
- Understanding of SQL `WHERE` clauses and comparison operators.
- Awareness of the three-valued logic implications of `NULL` in filters, constraints, and joins.

### Related Programming Areas

- Data Definition Language (DDL) – column type declarations and `DEFAULT` values.
- Data Manipulation Language (DML) – inserting, updating, and querying Boolean columns.
- Constraint design – `NOT NULL`, `CHECK`, and default values for Boolean columns.
- Query optimisation – Boolean columns have very low cardinality; indexing strategies differ from those for high-cardinality columns.
- Application mapping – SQL Booleans map to language-level types such as `bool`, `Boolean`, and `boolean`.

---

## Core Concepts and Features

### 1. `BOOLEAN`

#### Definitions

**Core Definition**
`BOOLEAN` is the SQL data type that stores truth values—`TRUE`, `FALSE`, and `NULL`—representing binary states or logical conditions.

**Technical Definition**
In PostgreSQL, `boolean` is a native type occupying **1 byte**, with states “true,” “false,” and “unknown” (represented by `NULL`). The input function accepts `true`, `yes`, `on`, `1` for true and `false`, `no`, `off`, `0` for false, along with unique prefixes and case-insensitive matching; the output function always emits `t` or `f`. In MySQL, `BOOLEAN` and `BOOL` are synonyms for `TINYINT(1)`: the column stores any integer from −128 to 127, with `0` treated as false and any non-zero value as true. In SQL Server, the equivalent is `BIT`, an integer type that accepts `1`, `0`, or `NULL`; string values `TRUE` and `FALSE` convert to `1` and `0`, and any non-zero value is promoted to `1`.

**Beginner-Friendly Explanation**
`BOOLEAN` is the yes/no container. In PostgreSQL it is a real Boolean; in MySQL it is a tiny integer wearing a Boolean costume; in SQL Server it is a bit. The key is that all three can represent true and false, but they differ in what other values they allow and how they are stored.

#### Purposes

- To store binary state flags such as `is_active`, `is_deleted`, `is_verified`, and `email_enabled`.
- To represent the result of logical expressions in queries and constraints.
- To provide a semantically clear and self-documenting alternative to integer flags (`0`/`1`).
- To enable Boolean-specific operators (`IS TRUE`, `IS FALSE`, `IS NOT TRUE`) that handle `NULL` correctly.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
column_name BOOLEAN
```

**MySQL:**

```sql
column_name BOOLEAN   -- alias for TINYINT(1)
column_name BOOL      -- alias for TINYINT(1)
```

**SQL Server:**

```sql
column_name BIT
```

**Component breakdown:**

| RDBMS | Type | Storage | Accepted values |
|-------|------|---------|-----------------|
| PostgreSQL | `BOOLEAN` | 1 byte | `TRUE`, `FALSE`, `NULL` (native) |
| MySQL | `BOOLEAN` / `BOOL` | 1 byte (`TINYINT`) | `0` = false, non-zero = true, `NULL` |
| SQL Server | `BIT` | 1 bit (packed) | `1`, `0`, `NULL` |

**Syntax rules:**

- PostgreSQL accepts `TRUE`, `FALSE`, and `NULL` as Boolean constants; `TRUE` and `FALSE` are the preferred SQL-compliant spellings.
- In PostgreSQL, `NULL` may need an explicit cast to Boolean in some contexts (`NULL::boolean`) because the parser cannot infer its type.
- In MySQL, `TRUE` and `FALSE` are aliases for `1` and `0`; the underlying column can still store other integers unless constrained.
- In SQL Server, `BIT` columns can be compared only with `=`, `<>`, or `IS NULL`; Boolean expressions require explicit comparison.

**Constraints and limitations:**

- MySQL's `BOOLEAN` does not enforce a two-value domain; without a `CHECK` constraint, a `BOOLEAN` column can store `2`, `-5`, or `100`.
- SQL Server's `BIT` is not a true Boolean type; it requires comparison operators in `WHERE` clauses rather than being used as a standalone predicate.
- In PostgreSQL, `BOOLEAN` columns can be used directly as predicates (`WHERE is_active`), which is convenient but can hide `NULL` handling nuances.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Native BOOLEAN column

```sql
-- Step 1: Create a table with a BOOLEAN column.
CREATE TABLE users (
    user_id  serial PRIMARY KEY,
    username text NOT NULL,
    is_active boolean NOT NULL DEFAULT true
);

-- Step 2: Insert rows with explicit and default values.
INSERT INTO users (username, is_active) VALUES ('alice', TRUE);
INSERT INTO users (username) VALUES ('bob');  -- uses default true

-- Step 3: Query using Boolean as a predicate.
SELECT username FROM users WHERE is_active;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
INSERT 0 1
 username
----------
 alice
 bob
(2 rows)
```

**Why this output appears**: PostgreSQL's native `BOOLEAN` type stores the truth values directly. The `WHERE is_active` predicate keeps only rows where the column is `TRUE`; `FALSE` and `NULL` rows would be excluded. Both inserted rows are active, so both appear.

##### Example 2: MySQL – BOOLEAN maps to TINYINT(1)

```sql
-- Step 1: Create a table with BOOLEAN columns.
CREATE TABLE feature_flags (
    flag_id    INT AUTO_INCREMENT PRIMARY KEY,
    flag_name  VARCHAR(100) NOT NULL,
    is_enabled BOOLEAN NOT NULL DEFAULT FALSE,
    is_public  BOOL NOT NULL DEFAULT TRUE
);

-- Step 2: Verify the actual column type.
DESCRIBE feature_flags;

-- Step 3: Insert values using TRUE/FALSE and integers.
INSERT INTO feature_flags (flag_name, is_enabled, is_public)
VALUES ('dark_mode', TRUE, FALSE), ('beta_ui', 1, 0);

-- Step 4: Query using explicit Boolean tests.
SELECT flag_name FROM feature_flags WHERE is_enabled IS TRUE;
```

**Expected output:**

```text
+------------+--------------+------+-----+---------+----------------+
| Field      | Type         | Null | Key | Default | Extra          |
+------------+--------------+------+-----+---------+----------------+
| flag_id    | int          | NO   | PRI | NULL    | auto_increment |
| flag_name  | varchar(100) | NO   |     | NULL    |                |
| is_enabled | tinyint(1)   | NO   |     | 0       |                |
| is_public  | tinyint(1)   | NO   |     | 1       |                |
+------------+--------------+------+-----+---------+----------------+

+-----------+
| flag_name |
+-----------+
| dark_mode |
+-----------+
```

**Why this output appears**: MySQL silently maps `BOOLEAN` and `BOOL` to `TINYINT(1)`. `TRUE` and `FALSE` are aliases for `1` and `0`, and both inserts store equivalent integer values. The `IS TRUE` test matches only the row where `is_enabled` is exactly `1` (true); the `beta_ui` row also has `is_enabled = 1`, so it would match if included, but the query returns `dark_mode` as the first match.

##### Example 3: SQL Server – BIT column with bit-packing

```sql
-- Step 1: Create a table with multiple BIT columns.
CREATE TABLE user_settings (
    user_id          INT PRIMARY KEY,
    email_verified   BIT NOT NULL DEFAULT 0,
    two_fa_enabled   BIT NOT NULL DEFAULT 0,
    is_admin         BIT NOT NULL DEFAULT 0
);

-- Step 2: Insert a row using integer values.
INSERT INTO user_settings (user_id, email_verified, two_fa_enabled, is_admin)
VALUES (1, 1, 0, 1);

-- Step 3: Query using comparison operators.
SELECT user_id FROM user_settings WHERE email_verified = 1;
```

**Expected output:**

```text
user_id
-----------
1
```

**Why this output appears**: SQL Server's `BIT` type stores `1`, `0`, or `NULL`. The three `BIT` columns are packed into a single byte because there are fewer than 8 `BIT` columns in the table. The `WHERE email_verified = 1` comparison filters correctly.

#### Real-World Cases

- **Feature flags**: A `feature_flags` table uses `BOOLEAN` columns (`is_enabled`, `is_public`) to control which features are available to which users.
- **Soft deletes**: An `is_deleted` Boolean column marks rows as logically deleted without physically removing them, preserving referential integrity and audit trails.
- **Account status**: An `is_active` Boolean column determines whether a user account can log in, replacing legacy `status` integer codes with a clear binary flag.

---

### 2. `TRUE`

#### Definitions

**Core Definition**
`TRUE` is the Boolean literal representing the affirmative or satisfied state in SQL's three-valued logic.

**Technical Definition**
In PostgreSQL, `TRUE` is a reserved SQL keyword that produces a Boolean value of true; the parser automatically understands it to be of type `boolean`. The input function also accepts `true`, `yes`, `on`, `1`, and unique prefixes of these strings, case-insensitively. In MySQL, `TRUE` is an alias for the integer `1`; Boolean functions treat any non-zero value as true, but `TRUE` specifically inserts `1`. In SQL Server, the string literal `'TRUE'` converts to `BIT` value `1`; however, `TRUE` is not a reserved keyword and must be quoted or used in a context where conversion to `BIT` occurs.

**Beginner-Friendly Explanation**
`TRUE` means yes. In PostgreSQL you can write it directly; in MySQL it is the number 1 in disguise; in SQL Server you often write `1` or the string `'TRUE'`.

#### Purposes

- To insert or compare affirmative Boolean values.
- To make queries and constraints self-documenting and readable.
- To provide a portable, SQL-standard spelling for the true state.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
-- As a literal in INSERT or comparison:
INSERT INTO flags (is_active) VALUES (TRUE);
SELECT * FROM flags WHERE is_active = TRUE;
-- As a direct predicate:
SELECT * FROM flags WHERE is_active;
```

**MySQL:**

```sql
INSERT INTO flags (is_active) VALUES (TRUE);  -- stores 1
SELECT * FROM flags WHERE is_active = TRUE;   -- compares to 1
```

**SQL Server:**

```sql
INSERT INTO flags (is_active) VALUES (1);     -- BIT column
SELECT * FROM flags WHERE is_active = 1;
```

**Component breakdown:**

| RDBMS | `TRUE` representation | Notes |
|-------|----------------------|-------|
| PostgreSQL | `TRUE` keyword | Native Boolean; preferred spelling |
| MySQL | `TRUE` alias | Maps to `1`; any non-zero is true |
| SQL Server | `1` or `'TRUE'` | `BIT` type; `TRUE` string converts to `1` |

**Syntax rules:**

- In PostgreSQL, `TRUE` is case-insensitive and whitespace-insensitive; `t`, `true`, `yes`, `on`, and `1` are all accepted as input.
- In MySQL, `TRUE` is a synonym for `1`; it does not enforce a two-value domain.
- In SQL Server, `TRUE` is not a reserved keyword; use `1` or `CAST('TRUE' AS BIT)`.

**Constraints and limitations:**

- In MySQL, `TRUE` inserted into a `BOOLEAN` (i.e., `TINYINT(1)`) column stores `1`, but the column can still hold other integers.
- In SQL Server, using `'TRUE'` as a string literal in a `BIT` context requires an implicit or explicit conversion.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – TRUE as literal and predicate

```sql
-- Step 1: Create a table.
CREATE TABLE tasks (task_id serial PRIMARY KEY, title text, completed boolean DEFAULT FALSE);

-- Step 2: Insert a completed task.
INSERT INTO tasks (title, completed) VALUES ('Write report', TRUE);

-- Step 3: Query with TRUE as a comparison and as a predicate.
SELECT title FROM tasks WHERE completed = TRUE;
SELECT title FROM tasks WHERE completed;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
  title
----------
 Write report
(1 row)
  title
----------
 Write report
(1 row)
```

**Why this output appears**: In PostgreSQL, `completed = TRUE` compares the column to the Boolean literal `TRUE`, and `WHERE completed` uses the column value directly as a predicate. Both forms return the row because `completed` is `TRUE`.

#### Real-World Cases

- **Task completion**: A `completed` Boolean column is set to `TRUE` when a task is finished, and queries filter with `WHERE completed = TRUE` or simply `WHERE completed`.
- **Consent tracking**: A `consent_given` Boolean column is set to `TRUE` when a user agrees to terms, enabling compliance queries.

---

### 3. `FALSE`

#### Definitions

**Core Definition**
`FALSE` is the Boolean literal representing the negative or unsatisfied state in SQL's three-valued logic.

**Technical Definition**
In PostgreSQL, `FALSE` is a reserved SQL keyword producing a Boolean value of false, accepted alongside `false`, `no`, `off`, `0`, and unique prefixes thereof. In MySQL, `FALSE` is an alias for the integer `0`; Boolean functions treat `0` as false, but any non-zero value is true. In SQL Server, the string `'FALSE'` converts to `BIT` value `0`, though `FALSE` is not a reserved keyword and `0` is the conventional representation.

**Beginner-Friendly Explanation**
`FALSE` means no. In PostgreSQL it is a real keyword; in MySQL it is `0`; in SQL Server you write `0`.

#### Purposes

- To insert or compare negative Boolean values.
- To set default values for flags that should start as disabled or unverified.
- To make `CHECK` constraints and queries explicit and readable.

#### Syntax Rules and Structure

**PostgreSQL:**

```sql
INSERT INTO flags (is_active) VALUES (FALSE);
SELECT * FROM flags WHERE is_active = FALSE;
SELECT * FROM flags WHERE NOT is_active;  -- excludes NULLs as well
```

**MySQL:**

```sql
INSERT INTO flags (is_active) VALUES (FALSE);  -- stores 0
SELECT * FROM flags WHERE is_active = FALSE;   -- compares to 0
```

**SQL Server:**

```sql
INSERT INTO flags (is_active) VALUES (0);
SELECT * FROM flags WHERE is_active = 0;
```

**Component breakdown:**

| RDBMS | `FALSE` representation | Notes |
|-------|-----------------------|-------|
| PostgreSQL | `FALSE` keyword | Native Boolean |
| MySQL | `FALSE` alias | Maps to `0` |
| SQL Server | `0` or `'FALSE'` | `BIT` type |

**Syntax rules:**

- In PostgreSQL, `FALSE` is case-insensitive; `f`, `false`, `no`, `off`, and `0` are accepted as input.
- In MySQL, `FALSE` is a synonym for `0`.
- In SQL Server, use `0` for `BIT` columns; `'FALSE'` requires conversion.

**Constraints and limitations:**

- `WHERE NOT is_active` in PostgreSQL returns rows where `is_active` is `FALSE`, but it excludes rows where `is_active` is `NULL` because `NOT UNKNOWN` is `UNKNOWN`, not `TRUE`.
- In MySQL, `FALSE` stored in a `BOOLEAN` column is just `0`; other non-zero values are also treated as true.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – FALSE and NOT with NULL

```sql
-- Step 1: Create a table with a nullable boolean.
CREATE TABLE subscriptions (id int, active boolean);

-- Step 2: Insert TRUE, FALSE, and NULL.
INSERT INTO subscriptions VALUES (1, TRUE), (2, FALSE), (3, NULL);

-- Step 3: Compare FALSE directly and with NOT.
SELECT id FROM subscriptions WHERE active = FALSE;
SELECT id FROM subscriptions WHERE NOT active;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 3
 id
----
  2
(1 row)
 id
----
  2
(1 row)
```

**Why this output appears**: Both queries return only row `2`, because `active = FALSE` matches the explicit `FALSE` value, and `NOT active` evaluates to `TRUE` only for row `2`; row `3` has `NULL`, for which `NOT NULL` is `UNKNOWN`, so it is excluded.

#### Real-World Cases

- **Default-disabled features**: A `is_enabled` column defaults to `FALSE` so new features are off until explicitly activated.
- **Unverified accounts**: An `email_verified` column starts as `FALSE` and is set to `TRUE` after the user confirms their email.

---

### 4. `NULL`

#### Definitions

**Core Definition**
`NULL` is the marker for the absence of a value—the “unknown” state in SQL's three-valued logic—and is not equivalent to `TRUE`, `FALSE`, `0`, or an empty string.

**Technical Definition**
In PostgreSQL's Boolean type, the third state “unknown” is represented by the SQL null value. SQL uses three truth values: `TRUE`, `FALSE`, and `UNKNOWN`. Any comparison involving `NULL` produces `UNKNOWN`: `NULL = NULL` is `UNKNOWN`, `NULL <> NULL` is `UNKNOWN`, and `WHERE col = NULL` never returns rows. The correct filters are `IS NULL` and `IS NOT NULL`. In `WHERE` and `JOIN ON` clauses, only rows for which the predicate evaluates strictly to `TRUE` are retained; rows evaluating to `UNKNOWN` are discarded, just like `FALSE`, even though the two states are logically distinct. For nullable Boolean columns, `WHERE flag IS TRUE` differs from `WHERE flag = TRUE`: the former includes only rows that are explicitly `TRUE`, while the latter also excludes rows where `flag` is `NULL` because `NULL = TRUE` is `UNKNOWN`.

**Beginner-Friendly Explanation**
`NULL` means “I don't know” or “no value here.” It is not false—it is a third state. If you ask “is this flag true?” and the flag is `NULL`, the answer is “I don't know,” not “no.” SQL handles this by introducing `UNKNOWN` as a third truth value. The practical consequence is that `NULL` values silently disappear from `WHERE` clauses unless you explicitly handle them with `IS NULL`, `IS NOT NULL`, or `COALESCE`.

#### Purposes

- To represent the absence of a value when a Boolean flag has not been set or is not applicable.
- To distinguish between “explicitly false” and “unknown” in data modelling.
- To support three-valued logic in constraints, joins, and aggregates.

#### Syntax Rules and Structure

**Correct NULL checks:**

```sql
-- PostgreSQL, MySQL, SQL Server:
WHERE col IS NULL
WHERE col IS NOT NULL
```

**Incorrect NULL checks (never match):**

```sql
WHERE col = NULL      -- always UNKNOWN
WHERE col <> NULL     -- always UNKNOWN
```

**Nullable Boolean-safe predicates (PostgreSQL):**

```sql
WHERE flag IS TRUE     -- only explicit TRUE
WHERE flag IS NOT TRUE -- FALSE and NULL
WHERE flag IS FALSE    -- only explicit FALSE
WHERE flag IS NOT FALSE-- TRUE and NULL
```

**Component breakdown:**

| Predicate | Matches `TRUE` | Matches `FALSE` | Matches `NULL` |
|-----------|----------------|-----------------|----------------|
| `flag = TRUE` | Yes | No | No (UNKNOWN) |
| `flag IS TRUE` | Yes | No | No |
| `flag IS NOT TRUE` | No | Yes | Yes |
| `flag = FALSE` | No | Yes | No (UNKNOWN) |
| `flag IS FALSE` | No | Yes | No |
| `flag IS NOT FALSE` | Yes | No | Yes |
| `flag IS NULL` | No | No | Yes |

**Syntax rules:**

- `NULL` is not equal to anything, including itself; use `IS NULL` to test for it.
- In PostgreSQL, `NULL` may need an explicit cast to Boolean in some contexts (`NULL::boolean`) because the parser cannot infer its type.
- In MySQL, `NULL` in a `BOOLEAN` column is a genuine `NULL`, not `0`.

**Constraints and limitations:**

- `NOT IN` with a list containing `NULL` returns no rows, because `x NOT IN (1, NULL)` expands to `x <> 1 AND x <> NULL`, and `x <> NULL` is `UNKNOWN`.
- `CHECK` constraints are satisfied when they evaluate to `UNKNOWN`; a nullable Boolean column with `CHECK (flag = TRUE)` still allows `NULL` because the constraint is not violated by an `UNKNOWN` result.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – NULL in a Boolean column and the difference between `= TRUE` and `IS TRUE`

```sql
-- Step 1: Create a table with a nullable Boolean column.
CREATE TABLE flags (id int, is_active boolean);

-- Step 2: Insert TRUE, FALSE, and NULL.
INSERT INTO flags VALUES (1, TRUE), (2, FALSE), (3, NULL);

-- Step 3: Compare the two query forms.
SELECT id FROM flags WHERE is_active = TRUE;
SELECT id FROM flags WHERE is_active IS TRUE;
SELECT id FROM flags WHERE is_active IS NOT TRUE;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 3
 id
----
  1
(1 row)
 id
----
  1
(1 row)
 id
----
  2
  3
(2 rows)
```

**Why this output appears**: `is_active = TRUE` returns only row `1` because row `3` (`NULL`) produces `UNKNOWN` in the comparison, and `UNKNOWN` rows are discarded by `WHERE`. `is_active IS TRUE` also returns only row `1`, but it is the explicit, NULL-safe form. `is_active IS NOT TRUE` returns rows `2` and `3`, because both `FALSE` and `NULL` are “not true.”

##### Example 2: MySQL – NULL versus FALSE in a BOOLEAN column

```sql
-- Step 1: Create a table with a nullable BOOLEAN column.
CREATE TABLE tasks (id INT, completed BOOLEAN);

-- Step 2: Insert TRUE, FALSE, and NULL.
INSERT INTO tasks VALUES (1, TRUE), (2, FALSE), (3, NULL);

-- Step 3: Query with IS FALSE and IS NULL.
SELECT id FROM tasks WHERE completed IS FALSE;
SELECT id FROM tasks WHERE completed IS NULL;
```

**Expected output:**

```text
+------+
| id   |
+------+
|    2 |
+------+
+------+
| id   |
+------+
|    3 |
+------+
```

**Why this output appears**: `IS FALSE` matches only the row where `completed` is exactly `0` (the false state). `NULL` is not `FALSE`—it is a separate unknown state—so the `NULL` row is returned only by `IS NULL`.

#### Real-World Cases

- **Optional consent**: A `consent_given` column is `NULL` when the user has not yet been asked, `TRUE` when they agreed, and `FALSE` when they declined. Queries must distinguish “not asked” from “declined.”
- **Soft-delete flags**: An `is_deleted` column may be `NULL` for legacy rows that predate the flag; queries must handle `NULL` explicitly to avoid accidentally treating legacy rows as deleted or active.

---

### 5. Boolean Best Practices

#### Definitions

**Core Definition**
Boolean best practices are the guidelines for declaring, naming, querying, and constraining Boolean columns to ensure correctness, clarity, and compatibility across RDBMSs and SQL's three-valued logic.

**Technical Definition**
Best practices include: (1) **use the native Boolean type where available** (PostgreSQL `BOOLEAN`, SQL Server `BIT`) and understand MySQL's `TINYINT(1)` mapping; (2) **name Boolean columns as yes/no predicates** using `is_`, `has_`, or `can_` prefixes (e.g., `is_active`, `has_permission`, `can_edit`); (3) **declare Boolean columns `NOT NULL` with a `DEFAULT`** to avoid three-valued logic complications; (4) **use `IS TRUE` / `IS FALSE` for nullable Boolean columns** rather than `= TRUE` / `= FALSE`, because the latter discard `NULL` rows; (5) **add `CHECK` constraints in MySQL** to enforce `0`/`1` values, since `BOOLEAN` is only an alias and does not enforce the domain; (6) **avoid indexing low-cardinality Boolean columns alone**; use composite indexes with the Boolean column as the last key part for selective queries; and (7) **never use `= NULL` or `<> NULL`**—use `IS NULL` and `IS NOT NULL`.

**Beginner-Friendly Explanation**
Best practices are the habits that keep Boolean data sane. Name your flags so they read like yes/no questions, make them `NOT NULL` with a default so you don't have to think about `NULL`, use `IS TRUE` and `IS FALSE` when `NULL` is possible, and in MySQL add a `CHECK` constraint so the column really stores only `0` and `1`.

#### Purposes

- To prevent the subtle bugs caused by `NULL` in Boolean columns.
- To make schemas self-documenting and consistent across teams.
- To ensure that queries behave the same way across PostgreSQL, MySQL, and SQL Server.
- To avoid performance pitfalls from indexing low-cardinality columns.

#### Syntax Rules and Structure

**Best-practice schema pattern (PostgreSQL):**

```sql
CREATE TABLE users (
    user_id   serial PRIMARY KEY,
    username  text NOT NULL,
    is_active boolean NOT NULL DEFAULT true,
    is_admin  boolean NOT NULL DEFAULT false
);
```

**Best-practice schema pattern (MySQL):**

```sql
CREATE TABLE users (
    user_id   INT AUTO_INCREMENT PRIMARY KEY,
    username  VARCHAR(100) NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    is_admin  BOOLEAN NOT NULL DEFAULT FALSE,
    CONSTRAINT chk_is_active CHECK (is_active IN (0, 1)),
    CONSTRAINT chk_is_admin  CHECK (is_admin  IN (0, 1))
);
```

**Best-practice schema pattern (SQL Server):**

```sql
CREATE TABLE users (
    user_id   INT IDENTITY(1,1) PRIMARY KEY,
    username  NVARCHAR(100) NOT NULL,
    is_active BIT NOT NULL DEFAULT 1,
    is_admin  BIT NOT NULL DEFAULT 0
);
```

**Best-practice query patterns:**

```sql
-- Correct for nullable Booleans:
WHERE is_active IS TRUE
WHERE is_active IS FALSE
WHERE is_active IS NOT TRUE   -- includes FALSE and NULL

-- Correct for NOT NULL Booleans:
WHERE is_active = TRUE
WHERE is_active = FALSE
WHERE NOT is_active           -- same as is_active = FALSE when NOT NULL

-- Never:
WHERE is_active = NULL        -- always UNKNOWN
```

**Component breakdown:**

| Practice | Implementation |
|----------|----------------|
| Native type | PostgreSQL `BOOLEAN`, SQL Server `BIT`, MySQL `TINYINT(1)` |
| Naming | `is_active`, `has_permission`, `can_edit` |
| Nullability | `NOT NULL DEFAULT` |
| MySQL domain | `CHECK (col IN (0,1))` |
| Null-safe query | `IS TRUE` / `IS FALSE` |
| Indexing | Composite index with Boolean as last key part |

**Constraints and limitations:**

- Adding `NOT NULL` later requires backfilling `NULL` values first.
- `CHECK` constraints are not enforced in MySQL before 8.0.16 (they are parsed but ignored).
- `IS TRUE` and `IS FALSE` are PostgreSQL extensions and are not portable to all RDBMSs; use `COALESCE` or explicit comparisons for maximum portability.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Enforcing Boolean domain with CHECK constraints

```sql
-- Step 1: Create a table with BOOLEAN columns and CHECK constraints.
CREATE TABLE user_settings (
    user_id        INT UNSIGNED NOT NULL PRIMARY KEY,
    email_verified BOOLEAN NOT NULL DEFAULT FALSE,
    two_fa_enabled BOOLEAN NOT NULL DEFAULT FALSE,
    CONSTRAINT chk_email_verified CHECK (email_verified IN (0, 1)),
    CONSTRAINT chk_two_fa        CHECK (two_fa_enabled IN (0, 1))
);

-- Step 2: Attempt to insert an invalid value.
INSERT INTO user_settings (user_id, email_verified) VALUES (1, 2);
```

**Expected output:**

```text
ERROR 3819 (HY000): Check constraint 'chk_email_verified' is violated.
```

**Why this output appears**: MySQL maps `BOOLEAN` to `TINYINT(1)`, which would otherwise accept `2`. The `CHECK` constraint enforces the strict Boolean domain, rejecting the invalid value.

##### Example 2: PostgreSQL – NOT NULL with DEFAULT avoids NULL logic

```sql
-- Step 1: Create a table with NOT NULL Boolean columns.
CREATE TABLE articles (
    article_id  serial PRIMARY KEY,
    title       text NOT NULL,
    is_published boolean NOT NULL DEFAULT false,
    is_featured  boolean NOT NULL DEFAULT false
);

-- Step 2: Insert without specifying the flags.
INSERT INTO articles (title) VALUES ('Draft article');

-- Step 3: Query using simple Boolean predicates.
SELECT title FROM articles WHERE NOT is_published;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
       title
-------------------
 Draft article
(1 row)
```

**Why this output appears**: Because `is_published` is `NOT NULL` and defaults to `false`, `WHERE NOT is_published` behaves exactly like `WHERE is_published = FALSE`—there is no `NULL` to complicate the logic. The draft article is returned.

#### Real-World Cases

- **User management**: `users` tables use `is_active`, `is_admin`, and `email_verified` Boolean columns, each `NOT NULL DEFAULT`, with naming that reads as yes/no questions.
- **Feature flags**: A `feature_flags` table uses `is_enabled` and `is_public` Boolean columns with `CHECK` constraints in MySQL to guarantee only `0`/`1` values.
- **Soft deletes**: An `is_deleted` column is `NOT NULL DEFAULT false`, avoiding the three-valued logic trap where a `NULL` deletion flag would ambiguously mean “not deleted” or “unknown.”

---

## References

- PostgreSQL Documentation – Boolean Type – https://www.postgresql.org/docs/16/datatype-boolean.html
- PostgreSQL Documentation – Boolean Type (version 15) – https://www.postgresql.org/docs/15/datatype-boolean.html
- MySQL 8.0 Reference Manual – Numeric Types (BOOLEAN/TINYINT) – https://dev.mysql.com/doc/refman/8.0/en/numeric-types.html
- OneUptime – How to Use BOOLEAN (BOOL) Data Type in MySQL – https://github.com/OneUptime/blog/tree/master/posts/2026-03-31-mysql-boolean-data-type
- Microsoft Learn – bit (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/data-types/bit-transact-sql
- Microsoft Learn – SQL Server Boolean Data Type (BIT) – https://learn.microsoft.com/en-us/sql/t-sql/data-types/bit-transact-sql
- The Art of PostgreSQL, Vol. 1 – Ch 16: Understanding Nulls – https://github.com/BrewHubPHL/supabase-specialist/blob/master/references/book-summaries/taop-vol1-ch16-nulls.md
- SQL Server NULL Handling Reference – https://github.com/damusix/skills/blob/efa8de4096f167cc896af6b869412def1614ccf0/mssql-server/references/25-null-handling.md
- Stack Overflow – Naming Boolean (bit) Values in a SQL Database – https://stackoverflow.com/questions/3762083/naming-boolean-bit-values-in-a-sql-db
- Open-ACE Database Conventions – Boolean Field Naming – https://github.com/open-ace/open-ace/blob/b794dc3cb3453006792e21a533c815d6d38b452d/docs/en/DATABASE-CONVENTIONS.md
- PostgreSQL Wiki – Don't Do This (Boolean NULLs) – https://wiki.postgresql.org/wiki/Don%27t_Do_This
- Use The Index, Luke – Indexing Boolean Columns – https://use-the-index-luke.com/sql/where-clause/searching-for-ranges/null-and-indexing