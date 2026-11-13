# Comprehensive Programming Cheat Sheet: SQL Domain Types

## Topic Overview

### Definitions

**Core Definition**
A domain is a user-defined data type that is based on another underlying type and optionally carries constraints that restrict its valid values to a subset of what the underlying type would allow. It is essentially a data type with optional constraints—restrictions on the allowed set of values.

**Technical Definition**
`CREATE DOMAIN` creates a new domain. The underlying type can be any built-in or user-defined base type, enum type, array type, composite type, range type, or another domain. The domain name must be unique among the types and domains existing in its schema. Domains are useful for abstracting common constraints on fields into a single location for maintenance; for example, several tables might contain email address columns all requiring the same `CHECK` constraint, and a domain avoids setting up each table's constraint individually. When an operator or function of the underlying type is applied to a domain value, the domain is automatically down-cast to the underlying type; assigning a value of the underlying type to a domain column or variable is allowed without an explicit cast, but the domain's constraints are checked.

**Beginner-Friendly Explanation**
A domain is a named, reusable bundle of rules attached to a data type. Imagine you have five tables, each with an `email` column, and every one of them must reject values that are not valid email addresses. Instead of writing the same `CHECK` constraint five times, you define an `email` domain once with the validation rule, and then use `email` as the column type in all five tables. The database applies the rule automatically everywhere the domain is used. A domain is like a custom stamp that says “this column must always obey these rules,” and you can stamp it onto as many columns as you like.

### Key Characteristics

- **Underlying type plus constraints**: A domain is a base type with a name, optional collation, optional default, and zero or more `CHECK` / `NOT NULL` constraints.
- **Reuse across columns and tables**: The primary purpose is to define a constraint once and apply it consistently wherever the domain is used, rather than repeating the same `CHECK` on every column.
- **Automatic down-casting**: When the underlying type's operators or functions are applied to a domain value, the value is automatically treated as the underlying type, and the result is of the underlying type—not the domain type—unless explicitly cast back.
- **Constraint checking on assignment**: Assigning a value of the underlying type to a domain column is allowed without an explicit cast, but the domain's constraints are checked at that moment.
- **`NOT NULL` on domains is a known trap**: Domain `NOT NULL` constraints are checked when converting a value to the domain type, but a column nominally of a domain type can still read as `NULL`—for example, on the nullable side of an outer join, or through an empty scalar subquery that produces a null of the domain type without further constraint checking. Best practice is therefore to design domain constraints so that `NULL` is allowed, and apply column-level `NOT NULL` to columns of the domain type as needed, rather than attaching `NOT NULL` directly to the domain.

### Prerequisites

- A running PostgreSQL instance and a client connection.
- `CREATE` privilege on the schema where the domain will be created; `USAGE` privilege on the underlying type.
- Familiarity with `CREATE TABLE` and `CHECK` constraint syntax.
- Awareness of the domain `NOT NULL` caveat and of the trade-offs between domains, column-level `CHECK` constraints, and reference tables.

### Related Programming Areas

- Data Definition Language (DDL) – `CREATE DOMAIN`, `ALTER DOMAIN`, `DROP DOMAIN`.
- Data Manipulation Language (DML) – inserting and updating values into domain-typed columns.
- Constraint management – centralised integrity rules versus per-column `CHECK` constraints.
- Data modelling – domains versus enums, composite types, and reference tables.
- Schema migration – `ALTER DOMAIN` for constraint changes and `NOT VALID` for online validation.

---

## Core Concepts and Features

### 1. Domains: Definition and Creation

#### Definitions

**Core Definition**
Domain creation is the process of defining a new domain with `CREATE DOMAIN`, specifying an underlying data type and optional constraints (`NOT NULL`, `CHECK`), a default value, and a collation.

**Technical Definition**
The syntax is `CREATE DOMAIN name [ AS ] data_type [ COLLATE collation ] [ DEFAULT expression ] [ domain_constraint [ ... ] ]`, where `domain_constraint` is `[ CONSTRAINT constraint_name ] { NOT NULL | NULL | CHECK (expression) }`. The `CHECK` expression must produce a Boolean result and must use the keyword `VALUE` to refer to the value being tested; expressions evaluating to `TRUE` or `UNKNOWN` succeed, and a `FALSE` result causes an error. When a domain has multiple `CHECK` constraints, they are tested in alphabetical order by name (PostgreSQL 9.5 and later).

**Beginner-Friendly Explanation**
Creating a domain is like defining a new type of “safe” value. You write `CREATE DOMAIN`, give it a name, say which base type it is built on, and then add rules. The most common rule is `CHECK`, which tests the value. For example, `CREATE DOMAIN posint AS integer CHECK (VALUE > 0)` defines a type that only accepts positive integers. Once created, `posint` can be used anywhere an `integer` could be used, but the database will enforce the positivity rule on every assignment.

#### Purposes

- To centralise common constraints so they are defined once and applied consistently across many columns and tables.
- To give a meaningful name to a constrained type, making the schema self-documenting (e.g., `email`, `us_postal_code`, `positive_integer`).
- To attach a default value that applies to every column of the domain type unless overridden.
- To abstract complex validation logic behind a reusable type name.

#### Syntax Rules and Structure

**General syntax:**

```sql
CREATE DOMAIN name [ AS ] data_type
    [ COLLATE collation ]
    [ DEFAULT expression ]
    [ CONSTRAINT constraint_name ]
    { NOT NULL | NULL | CHECK (expression) } [ ... ];
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `name` | The domain name; unique among types and domains in its schema. |
| `data_type` | The underlying type; any built-in or user-defined type, including arrays. |
| `COLLATE` | Optional collation; requires a collatable underlying type. |
| `DEFAULT expression` | Optional default; subqueries are not allowed; overridden by column-level defaults. |
| `CONSTRAINT constraint_name` | Optional name for the constraint; the system generates one if omitted. |
| `NOT NULL` | Prevents null values (with the caveat about outer joins). |
| `CHECK (expression)` | Integrity test using `VALUE`; `TRUE` or `UNKNOWN` succeeds. |

**Syntax rules:**

- The `AS` keyword is optional but conventional.
- `CHECK` expressions cannot contain subqueries or refer to variables other than `VALUE`.
- Multiple `CHECK` constraints are evaluated in alphabetical order by constraint name.

**Constraints and limitations:**

- Domain `NOT NULL` does not provide the same guarantee as a column-level `NOT NULL` when outer joins or null-producing subqueries are involved.
- Adding a `CHECK` constraint to an existing domain checks all existing columns using that domain, which may require a table rewrite or can be deferred with `NOT VALID`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Creating a positive-integer domain and using it in a table

```sql
-- Step 1: Create a domain over integer that only accepts positive values.
CREATE DOMAIN posint AS integer CHECK (VALUE > 0);

-- Step 2: Create a table that uses the domain as a column type.
CREATE TABLE mytable (id posint);

-- Step 3: Insert a valid value.
INSERT INTO mytable VALUES (1);

-- Step 4: Attempt to insert an invalid value.
INSERT INTO mytable VALUES (-1);
```

**Expected output:**

```text
CREATE DOMAIN
CREATE TABLE
INSERT 0 1
ERROR:  value for domain posint violates check constraint "posint_check"
```

**Why this output appears**: The domain `posint` enforces `VALUE > 0` on every value assigned to a column of that type. The first insert satisfies the constraint; the second violates it, so PostgreSQL rejects the value with an error naming the domain's check constraint.

##### Example 2: Creating an email domain with a regular-expression check

```sql
-- Step 1: Create a domain for email addresses.
CREATE DOMAIN email AS varchar(255)
    CHECK (VALUE ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');

-- Step 2: Create two tables that both use the domain.
CREATE TABLE users (user_id int PRIMARY KEY, email email);
CREATE TABLE contacts (contact_id int PRIMARY KEY, email email);

-- Step 3: Insert a valid email into one table.
INSERT INTO users VALUES (1, 'alice@example.com');

-- Step 4: Attempt an invalid email into the other table.
INSERT INTO contacts VALUES (1, 'not-an-email');
```

**Expected output:**

```text
CREATE DOMAIN
CREATE TABLE
CREATE TABLE
INSERT 0 1
ERROR:  value for domain email violates check constraint "email_check"
```

**Why this output appears**: The `email` domain encapsulates the email-validation rule once, and both `users.email` and `contacts.email` inherit it automatically. The valid address is accepted in `users`; the invalid one is rejected in `contacts`, demonstrating the reuse benefit of domains.

#### Real-World Cases

- **Postal codes**: A `us_postal_code` domain over `text` with a `CHECK` for the `\d{5}` or `\d{5}-\d{4}` pattern ensures consistent validation across every address table.
- **Positive quantities**: A `positive_integer` domain eliminates the need to repeat `CHECK (quantity > 0)` on every quantity column.
- **Email addresses**: An `email` domain centralises the validation regex, so a change to the rule is a single `ALTER DOMAIN` rather than a sweep through dozens of tables.

---

### 2. Domain Constraints

#### Definitions

**Core Definition**
Domain constraints are the rules attached to a domain—`NOT NULL` and one or more `CHECK` constraints—that every value of that domain must satisfy.

**Technical Definition**
Domain constraints are checked when a value is converted to the domain type, whether through an explicit cast or an implicit assignment of the underlying type to a domain column. `CHECK` constraints must evaluate to `TRUE` or `UNKNOWN`; a `FALSE` result raises an error and the value is not allowed to be converted to the domain type. When a domain has multiple `CHECK` constraints, they are tested in alphabetical order by name. `NOT NULL` on a domain is intended to prevent null values, but it does not survive outer joins or null-producing subqueries in the same way a column-level `NOT NULL` does.

**Beginner-Friendly Explanation**
Domain constraints are the rules that make a domain useful. A `CHECK` constraint is a test that the value must pass—like “this must look like an email address” or “this must be greater than zero.” The `NOT NULL` constraint says “this value must always be present,” but it has a subtle weakness: if a query generates a `NULL` through an outer join or an empty subquery, the `NULL` can slip through even when the domain says `NOT NULL`. That is why best practice is to allow `NULL` in the domain and put `NOT NULL` on individual columns instead.

#### Purposes

- To enforce value-level integrity rules consistently across every column of the domain type.
- To centralise validation logic so that changing a rule means changing one domain constraint, not many column constraints.
- To provide named constraints that appear in error messages, making debugging easier.
- To enforce a default value consistently across columns of the domain type.

#### Syntax Rules and Structure

**Constraint forms:**

| Constraint | Syntax | Notes |
|------------|--------|-------|
| `NOT NULL` | `NOT NULL` | Prevents nulls, with the outer-join caveat |
| `NULL` | `NULL` | Default; allows nulls; for nonstandard SQL compatibility only |
| `CHECK` | `CHECK (expression)` | Uses `VALUE`; cannot contain subqueries |

**Syntax rules:**

- Each `CHECK` must produce a Boolean result and use `VALUE` to refer to the tested value.
- Expressions evaluating to `TRUE` or `UNKNOWN` succeed; `FALSE` fails.
- Multiple `CHECK` constraints are evaluated in alphabetical order by name.

**Constraints and limitations:**

- Domain `NOT NULL` is not a substitute for column-level `NOT NULL` in queries involving outer joins or scalar subqueries.
- `CHECK` expressions cannot contain subqueries or refer to variables other than `VALUE`.
- Adding a constraint to an existing domain checks all existing columns using the domain, which may require a table rewrite unless `NOT VALID` is used.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Multiple CHECK constraints and alphabetical evaluation

```sql
-- Step 1: Create a domain with two CHECK constraints.
CREATE DOMAIN graded_score AS integer
    CONSTRAINT chk_min CHECK (VALUE >= 0)
    CONSTRAINT chk_max CHECK (VALUE <= 100);

-- Step 2: Insert a valid value.
CREATE TABLE scores (score graded_score);
INSERT INTO scores VALUES (85);

-- Step 3: Attempt an out-of-range value.
INSERT INTO scores VALUES (150);
```

**Expected output:**

```text
CREATE DOMAIN
CREATE TABLE
INSERT 0 1
ERROR:  value for domain graded_score violates check constraint "chk_max"
```

**Why this output appears**: The domain `graded_score` enforces both a minimum (0) and a maximum (100). The value 150 satisfies `chk_min` but violates `chk_max`. Because `chk_max` sorts before `chk_min` alphabetically, the error names `chk_max`.

##### Example 2: Domain NOT NULL versus column NOT NULL in an outer join

```sql
-- Step 1: Create a domain with NOT NULL.
CREATE DOMAIN required_text AS text NOT NULL;

-- Step 2: Create two tables and insert data.
CREATE TABLE a (id int, val required_text);
CREATE TABLE b (id int, val required_text);
INSERT INTO a VALUES (1, 'hello');
INSERT INTO b VALUES (1, 'world');

-- Step 3: Perform a LEFT JOIN with the domain column on the nullable side.
SELECT a.id, b.val
FROM a LEFT JOIN b ON a.id = b.id + 1;
```

**Expected output:**

```text
 id |  val
----+-------
  1 |
(1 row)
```

**Why this output appears**: The `LEFT JOIN` produces a `NULL` for `b.val` because no matching row exists in `b`. Even though `required_text` is declared `NOT NULL`, the domain constraint is not re-checked on the null-extended side of the outer join, so the column reads as `NULL`. This illustrates the documented caveat that domain `NOT NULL` does not survive outer joins.

#### Real-World Cases

- **Range checks**: A `percentage` domain with `CHECK (VALUE >= 0 AND VALUE <= 100)` ensures every percentage column across the schema stays within bounds.
- **String patterns**: A `phone_number` domain with a `CHECK` regex enforces a consistent phone format regardless of which table stores the number.
- **Enumeration-like validation**: A `status` domain with `CHECK (VALUE IN ('active', 'inactive', 'pending'))` provides enum-like validation without a separate type.

---

### 3. Domain Operations: ALTER and DROP

#### Definitions

**Core Definition**
Domain operations are the actions available on an existing domain: altering its default, constraints, owner, name, or schema with `ALTER DOMAIN`, and removing it with `DROP DOMAIN`.

**Technical Definition**
`ALTER DOMAIN name` supports: `SET DEFAULT expression` and `DROP DEFAULT`; `SET NOT NULL` and `DROP NOT NULL`; `ADD domain_constraint [ NOT VALID ]`; `DROP CONSTRAINT [ IF EXISTS ] constraint_name [ RESTRICT | CASCADE ]`; `RENAME CONSTRAINT`; `VALIDATE CONSTRAINT`; `OWNER TO`; `RENAME TO`; and `SET SCHEMA`. `DROP DOMAIN [ IF EXISTS ] name [, ...] [ CASCADE | RESTRICT ]` removes a domain; only the owner can drop it, and `RESTRICT` is the default, refusing to drop if any objects depend on the domain.

**Beginner-Friendly Explanation**
`ALTER DOMAIN` is how you change a domain after it has been created. You can change its default, add or remove constraints, rename it, move it to another schema, or change its owner. `DROP DOMAIN` deletes it entirely—but only if nothing is using it, unless you say `CASCADE` to automatically drop the dependent columns too.

#### Purposes

- To evolve a domain's rules as business requirements change (`ADD`, `DROP`, `VALIDATE` constraints).
- To change or remove the domain's default value.
- To transfer ownership or move the domain to a different schema.
- To remove an obsolete domain, optionally cascading to dependent columns.

#### Syntax Rules and Structure

**Key `ALTER DOMAIN` sub-forms:**

| Operation | Syntax | Notes |
|-----------|--------|-------|
| Set default | `ALTER DOMAIN name SET DEFAULT expression` | Affects future inserts only |
| Drop default | `ALTER DOMAIN name DROP DEFAULT` | |
| Set NOT NULL | `ALTER DOMAIN name SET NOT NULL` | Only if no existing nulls |
| Drop NOT NULL | `ALTER DOMAIN name DROP NOT NULL` | |
| Add constraint | `ALTER DOMAIN name ADD domain_constraint [ NOT VALID ]` | `NOT VALID` skips existing-data check |
| Drop constraint | `ALTER DOMAIN name DROP CONSTRAINT [ IF EXISTS ] constraint_name [ CASCADE \| RESTRICT ]` | |
| Validate constraint | `ALTER DOMAIN name VALIDATE CONSTRAINT constraint_name` | Validates a `NOT VALID` constraint |
| Rename | `ALTER DOMAIN name RENAME TO new_name` | |
| Set schema | `ALTER DOMAIN name SET SCHEMA new_schema` | Requires `CREATE` on new schema |

**`DROP DOMAIN` syntax:**

```sql
DROP DOMAIN [ IF EXISTS ] name [, ...] [ CASCADE | RESTRICT ];
```

**Syntax rules:**

- `ALTER DOMAIN` requires ownership of the domain.
- `DROP DOMAIN` requires ownership; `RESTRICT` is the default and refuses if dependencies exist.
- `ADD CONSTRAINT ... NOT VALID` only applies to `CHECK` constraints; newly inserted or updated rows are always checked.

**Constraints and limitations:**

- Adding a `CHECK` constraint to a domain checks all existing columns using the domain, which may cause a table rewrite; use `NOT VALID` and `VALIDATE CONSTRAINT` for online migration.
- `DROP DOMAIN ... CASCADE` automatically drops dependent table columns, which is destructive and irreversible.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Adding a NOT VALID constraint and validating it

```sql
-- Step 1: Create a domain and a table using it.
CREATE DOMAIN score AS integer CHECK (VALUE >= 0);
CREATE TABLE results (id int, score score);
INSERT INTO results VALUES (1, 50), (2, 75);

-- Step 2: Add a new constraint without checking existing rows.
ALTER DOMAIN score ADD CONSTRAINT chk_max CHECK (VALUE <= 100) NOT VALID;

-- Step 3: New inserts are checked against the new constraint.
INSERT INTO results VALUES (3, 150);

-- Step 4: Validate the constraint against existing data.
ALTER DOMAIN score VALIDATE CONSTRAINT chk_max;
```

**Expected output:**

```text
CREATE DOMAIN
CREATE TABLE
INSERT 0 2
ALTER DOMAIN
ERROR:  value for domain score violates check constraint "chk_max"
ALTER DOMAIN
```

**Why this output appears**: `ADD CONSTRAINT ... NOT VALID` adds the `chk_max` constraint without scanning existing rows, so the `ALTER DOMAIN` succeeds quickly. The new insert of 150 is immediately checked and rejected because new rows are always validated even for `NOT VALID` constraints. `VALIDATE CONSTRAINT` then verifies existing rows (50 and 75), which pass, completing the constraint without a long lock.

##### Example 2: Dropping a domain with RESTRICT versus CASCADE

```sql
-- Step 1: Create a domain and a table using it.
CREATE DOMAIN code AS text;
CREATE TABLE items (id int, code code);
INSERT INTO items VALUES (1, 'ABC');

-- Step 2: Attempt to drop the domain with the default RESTRICT.
DROP DOMAIN code;

-- Step 3: Drop the domain with CASCADE.
DROP DOMAIN code CASCADE;
```

**Expected output:**

```text
CREATE DOMAIN
CREATE TABLE
INSERT 0 1
ERROR:  cannot drop domain code because other objects depend on it
DETAIL:  column code of table items depends on domain code
HINT:  Use DROP ... CASCADE to drop the dependent objects too.
DROP DOMAIN
```

**Why this output appears**: `RESTRICT` (the default) refuses to drop the domain because `items.code` depends on it. `CASCADE` drops the dependent column `items.code` along with the domain, which is why the second `DROP DOMAIN` succeeds.

#### Real-World Cases

- **Online constraint migration**: A new `CHECK` is added to a domain with `NOT VALID`, validated separately during a low-traffic window, avoiding a long `ACCESS EXCLUSIVE` lock on large tables.
- **Schema refactoring**: A domain is renamed with `ALTER DOMAIN ... RENAME TO` and moved to another schema with `SET SCHEMA`, keeping dependent columns intact.
- **Decommissioning**: An obsolete domain is dropped with `RESTRICT` first to inspect dependencies, then `CASCADE` after dependent columns are migrated or removed.

---

### 4. Domain Best Practices

#### Definitions

**Core Definition**
Domain best practices are the guidelines for deciding when to use a domain, how to structure its constraints, how to avoid the `NOT NULL` trap, and how to manage domains safely in production.

**Technical Definition**
Best practices include: (1) **use domains when the same constraint is repeated across many columns or tables**; domains are useful for abstracting common constraints into a single location for maintenance; (2) **allow `NULL` in domain constraints and apply column-level `NOT NULL` to columns of the domain type**, because domain `NOT NULL` is broken by outer joins and null-producing subqueries; (3) **give domains meaningful names** that describe the value they represent (e.g., `email`, `us_postal_code`, `positive_integer`), so the schema is self-documenting; (4) **keep constraints focused**—each domain should have a clear, specific purpose, and documentation should explain its business meaning; (5) **use `ADD CONSTRAINT ... NOT VALID` followed by `VALIDATE CONSTRAINT` for online changes** to avoid table rewrites and long locks; and (6) **be aware of performance considerations**—domains add little storage overhead, but constraint checks are performed on every conversion to the domain type, and complex domains may increase the chance of a change requiring an expensive table rewrite.

**Beginner-Friendly Explanation**
Best practices for domains are about using them where they genuinely help and avoiding their sharp edges. Use a domain when you find yourself copying the same `CHECK` constraint onto many columns—that is the sweet spot. Give the domain a clear, descriptive name. Do not put `NOT NULL` on the domain itself; put it on the individual columns instead, because `NOT NULL` on a domain does not survive outer joins. When adding constraints to an existing domain in production, use `NOT VALID` first and validate later so you do not lock a large table for a long time.

#### Purposes

- To avoid the `NOT NULL` trap that causes domain columns to unexpectedly read as `NULL`.
- To maximise the reuse benefit of domains while keeping the schema understandable.
- To apply constraint changes to large tables without downtime.
- To make informed decisions between domains, column-level `CHECK` constraints, enums, and reference tables.

#### Syntax Rules and Structure

**Best-practice pattern (allow NULL, enforce column-level NOT NULL):**

```sql
-- Domain allows NULL.
CREATE DOMAIN email AS varchar(255)
    CHECK (VALUE ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');

-- Column enforces NOT NULL.
CREATE TABLE users (
    user_id int PRIMARY KEY,
    email   email NOT NULL
);
```

**Best-practice pattern (online constraint addition):**

```sql
-- Add the constraint without scanning existing rows.
ALTER DOMAIN email ADD CONSTRAINT chk_email_length CHECK (length(VALUE) <= 255) NOT VALID;

-- Validate separately, during a low-traffic window.
ALTER DOMAIN email VALIDATE CONSTRAINT chk_email_length;
```

**Decision guide:**

| Scenario | Recommended approach |
|----------|----------------------|
| Same `CHECK` repeated on many columns/tables | `DOMAIN` |
| One-off constraint on a single column | Column-level `CHECK` constraint |
| Need `NOT NULL` guarantees across joins | Column-level `NOT NULL`, not domain `NOT NULL` |
| Vocabulary is a fixed, ordered set | `ENUM` |
| Values need metadata or frequent changes | Reference (lookup) table |

**Constraints and limitations:**

- Domain `NOT NULL` does not survive outer joins.
- Adding constraints to a domain can trigger table rewrites unless `NOT VALID` is used.
- Domains with complex constraints may complicate online migrations.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Best-practice domain with NULL allowed and column-level NOT NULL

```sql
-- Step 1: Create a domain that allows NULL.
CREATE DOMAIN us_postal_code AS text
    CHECK (VALUE ~ '^\d{5}$' OR VALUE ~ '^\d{5}-\d{4}$');

-- Step 2: Create a table with a NOT NULL column of the domain type.
CREATE TABLE addresses (
    address_id int PRIMARY KEY,
    postal     us_postal_code NOT NULL
);

-- Step 3: Insert a valid postal code.
INSERT INTO addresses VALUES (1, '12345');

-- Step 4: Attempt to insert NULL (should fail because the column is NOT NULL).
INSERT INTO addresses VALUES (2, NULL);

-- Step 5: Attempt to insert an invalid postal code.
INSERT INTO addresses VALUES (3, 'ABCDE');
```

**Expected output:**

```text
CREATE DOMAIN
CREATE TABLE
INSERT 0 1
ERROR:  null value in column "postal" violates not-null constraint
ERROR:  value for domain us_postal_code violates check constraint "us_postal_code_check"
```

**Why this output appears**: The domain allows `NULL` (no `NOT NULL` on the domain), but the column `postal` is declared `NOT NULL`, so the `NULL` insert fails with a column-level not-null error. The invalid postal code fails the domain's `CHECK` constraint. This pattern gives you the validation reuse of the domain plus the reliable `NOT NULL` guarantee of the column.

#### Real-World Cases

- **Email validation at scale**: An `email` domain is defined once with a regex `CHECK`, and dozens of tables use it. Columns that must always have an email are declared `email NOT NULL`, combining domain validation with column-level null protection.
- **Postal code consistency**: A `us_postal_code` domain ensures every address table validates the same ZIP format, and a migration that adds a new rule uses `NOT VALID` plus `VALIDATE CONSTRAINT` to avoid locking large tables.
- **Positive numeric values**: A `positive_numeric` domain replaces scattered `CHECK (amount > 0)` constraints with a single named type, making the schema more self-documenting and easier to maintain.

---

## References

- PostgreSQL Documentation – Domain Types – https://www.postgresql.org/docs/current/domains.html
- PostgreSQL Documentation – `CREATE DOMAIN` – https://www.postgresql.org/docs/current/sql-createdomain.html
- PostgreSQL Documentation – `ALTER DOMAIN` – https://www.postgresql.org/docs/current/sql-alterdomain.html
- PostgreSQL Documentation – `DROP DOMAIN` – https://www.postgresql.org/docs/current/sql-dropdomain.html
- PostgreSQL Mailing List – Domains vs data types – https://www.postgresql.org/message-id/flat/...
- PostgreSQL Mailing List – Outer joins and NULLs (domain NOT NULL caveat) – https://www.postgresql.org/message-id/flat/...
- PostgreSQL Mailing List – Performance of DOMAINs – https://www.postgresql.org/message-id/flat/...
- PostgreSQL Documentation (French) – CREATE DOMAIN (best practice on NULL) – http://docs.postgresql.fr/16/pg16.pdf#323#538#323
- Crunchy Data – Intro to Postgres Custom Data Types – https://www.crunchydata.com/blog/intro-to-postgres-custom-data-types
- PostgreSQL Tutorial – User-defined Data Types – https://www.rockdata.net/postgresql-tutorial/postgresql-user-defined-data-types/