# Comprehensive Programming Cheat Sheet: SQL Enumerated Types (ENUM)

## Topic Overview

### Definitions

**Core Definition**
`ENUM` (enumerated type) is a PostgreSQL data type that comprises a **static, ordered set of values**, equivalent to the `enum` types supported in many programming languages. It constrains a column to accept only a predefined list of labeled values, with a fixed sort order determined at type-creation time.

**Technical Definition**
An enum type is created with `CREATE TYPE name AS ENUM ( 'label1', 'label2', ... )`. Each label is a quoted string shorter than `NAMEDATALEN` (64 bytes in standard builds). Internally, each enum value occupies **4 bytes** on disk, regardless of the label's length, making enums extremely compact compared with storing the label as text. Enum labels are **case-sensitive**, whitespace is significant, and **each enum type is separate**—values of one enum type cannot be compared with values of another enum type without an explicit cast to `text`. The ordering of values is the order in which they were listed at creation; all standard comparison operators and aggregate functions work on enums, and `ORDER BY` sorts them in declaration order.

**Beginner-Friendly Explanation**
An `ENUM` is a fixed menu of allowed values. If a column is defined as `mood AS ENUM ('sad', 'ok', 'happy')`, then the database will only accept those three strings—`'ecstatic'` or `'Sad'` would be rejected. The values also have a built-in order: `'sad' < 'ok' < 'happy'`, so you can sort and compare them naturally. Enums are compact (4 bytes per value) and type-safe at the column level, but they are rigid: adding a value is a schema change, removing a value is difficult, and you cannot attach extra metadata (such as a display label or a “terminal” flag) to an enum value.

### Key Characteristics

- **Static and ordered vocabulary**: The set of allowed values is fixed at type-creation time, and the declaration order defines the sort order used by `ORDER BY`, `<`, `>`, and `MIN`/`MAX`.
- **Compact storage**: An enum value uses **4 bytes** on disk regardless of the label length, whereas storing the label as `text` repeats the full string on every row and inflates table size.
- **Type safety across enum types**: Each enum type is distinct; comparing values from two different enum types raises “operator does not exist” unless you cast both sides to `text`.
- **Schema-change friction**: `ALTER TYPE ... ADD VALUE` adds new values (with optional `BEFORE`/`AFTER` positioning), and `ALTER TYPE ... RENAME VALUE` renames them, but there is **no** `DROP VALUE`; removing a value requires recreating the type and migrating columns.
- **No attached metadata**: Enums store only the label and nothing else. If you later need a display name, a sort order independent of declaration order, or a boolean flag, a reference (lookup) table is the better choice.

### Prerequisites

- A running PostgreSQL instance (enum types have been available since early versions; the documentation shown covers PostgreSQL 15).
- `CREATE` privilege on the schema where the type will be created.
- Familiarity with `CREATE TABLE` and `ALTER TABLE` syntax.
- An understanding of whether the value set is genuinely **closed and stable** (a good fit for enums) or likely to grow or require metadata (a better fit for a reference table).

### Related Programming Areas

- Data Definition Language (DDL) – `CREATE TYPE`, `ALTER TYPE`, `DROP TYPE`.
- Data Manipulation Language (DML) – inserting, updating, and comparing enum values.
- Data modelling – enums versus `CHECK` constraints versus reference (lookup) tables.
- Application development – mapping enum values to language-level enums or status constants.
- Query optimisation – enums are compact and avoid joins, but schema changes carry deployment friction.

---

## Core Concepts and Features

### 1. `ENUM`

#### Definitions

**Core Definition**
`ENUM` is a PostgreSQL user-defined type that restricts a column to a fixed, ordered list of string labels.

**Technical Definition**
An enum type is a named type created with `CREATE TYPE ... AS ENUM (...)`. It is stored as a 4-byte internal value, with the textual label resolved via the type's catalog entry. Enum types are separate from one another and from `text`; explicit casts are required for cross-type comparison. Standard comparison operators (`=`, `<>`, `<`, `>`, `<=`, `>=`), `ORDER BY`, and aggregate functions such as `MIN()` and `MAX()` operate on enums according to their declaration order.

**Beginner-Friendly Explanation**
An `ENUM` is a custom type that acts like a dropdown menu in a form. It guarantees that only the listed values can be stored, and it remembers the order you listed them in.

#### Purposes

- To enforce a fixed vocabulary at the column level without a `CHECK` constraint or a foreign-key join.
- To provide an intrinsic sort order that matches the logical progression of the values (e.g., `draft → submitted → approved → shipped`).
- To reduce storage compared with storing repeated text labels, because each value is 4 bytes regardless of label length.
- To serve as a parameter type in functions and procedures, providing type safety across the application.

#### Syntax Rules and Structure

**Column declaration using an existing enum type:**

```sql
CREATE TABLE orders (
    order_id   serial PRIMARY KEY,
    status     order_status NOT NULL DEFAULT 'draft'
);
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| Enum type name | A user-defined type created with `CREATE TYPE`. |
| Label | A quoted string that is case-sensitive and whitespace-sensitive. |
| Storage | 4 bytes per value. |

**Syntax rules:**

- Enum labels are case-sensitive (`'happy'` ≠ `'HAPPY'`) and whitespace in labels is significant.
- Each enum type is distinct; cross-type comparison requires casting to `text`.
- The sort order is the declaration order and cannot be changed without recreating the type.

**Constraints and limitations:**

- Existing values cannot be removed from an enum type, nor can the sort order be changed, short of dropping and recreating the enum type.
- Enums cannot carry extra columns or metadata.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Using an enum in a table and observing ordering

```sql
-- Step 1: Create an enum type for moods.
CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');

-- Step 2: Create a table that uses the enum.
CREATE TABLE person (
    name         text,
    current_mood mood
);

-- Step 3: Insert rows.
INSERT INTO person VALUES ('Moe', 'happy');
INSERT INTO person VALUES ('Larry', 'sad');
INSERT INTO person VALUES ('Curly', 'ok');

-- Step 4: Compare using enum ordering.
SELECT * FROM person WHERE current_mood > 'sad' ORDER BY current_mood;
```

**Expected output:**

```text
 name  | current_mood
-------+--------------
 Curly | ok
 Moe   | happy
(2 rows)
```

**Why this output appears**: The enum `mood` has the order `'sad' < 'ok' < 'happy'`. The filter `current_mood > 'sad'` excludes `'sad'` and keeps `'ok'` and `'happy'`; `ORDER BY current_mood` sorts them in declaration order (`ok` before `happy`), not alphabetically.

##### Example 2: Type safety across enum types

```sql
-- Step 1: Create a second, distinct enum type.
CREATE TYPE happiness AS ENUM ('happy', 'very happy', 'ecstatic');

-- Step 2: Create a table using the new enum.
CREATE TABLE holidays (
    num_weeks integer,
    happiness happiness
);
INSERT INTO holidays VALUES (4, 'happy'), (6, 'very happy');

-- Step 3: Attempt to compare values from different enum types.
SELECT person.name, holidays.num_weeks
FROM person, holidays
WHERE person.current_mood = holidays.happiness;
```

**Expected output:**

```text
ERROR:  operator does not exist: mood = happiness
```

**Why this output appears**: `mood` and `happiness` are separate enum types and cannot be compared directly. PostgreSQL requires an explicit cast to `text` (or a custom operator) to compare them.

#### Real-World Cases

- **Order status**: A `status` column uses an enum with values `('draft', 'submitted', 'approved', 'shipped', 'delivered', 'cancelled')`, providing type safety and a natural sort order for reporting.
- **Ticket priority**: A `priority` column uses an enum `('low', 'medium', 'high', 'critical')` so that `ORDER BY priority` produces the correct severity order without a lookup table.
- **Days of the week**: A `day_of_week` column uses an enum in weekday order, enabling simple range queries such as `WHERE day > 'Wednesday'`.

---

### 2. Enum Creation

#### Definitions

**Core Definition**
Enum creation is the process of defining a new enumerated type with `CREATE TYPE`, optionally specifying the labels in the desired sort order.

**Technical Definition**
The syntax is `CREATE TYPE name AS ENUM ( [ 'label' [, ... ] ] )`. Labels are quoted strings shorter than `NAMEDATALEN` (64 bytes in standard builds). The order in which labels are listed becomes the type's sort order. It is possible to create an enum with zero labels, but such a type cannot hold values until at least one label is added with `ALTER TYPE ... ADD VALUE`.

**Beginner-Friendly Explanation**
Enum creation is where you define the menu of allowed values. The order you write them in is the order the database will use for sorting and comparison.

#### Purposes

- To establish the allowed vocabulary and its sort order before using it in tables.
- To reuse the same vocabulary across multiple tables and functions.
- To replace ad-hoc `text` columns with a type-safe, compact alternative.

#### Syntax Rules and Structure

**General syntax:**

```sql
CREATE TYPE type_name AS ENUM ( 'label1', 'label2', ... );
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `type_name` | A new type name, unique within its schema. |
| `'label'` | A quoted string; case-sensitive and whitespace-sensitive. |
| Declaration order | Defines the sort order. |

**Syntax rules:**

- Each label must be shorter than `NAMEDATALEN` bytes (64 by default).
- An empty enum can be created but cannot hold values until a label is added.
- The type name must be distinct from existing types, domains, and tables in the same schema.

**Constraints and limitations:**

- Labels cannot be removed or reordered without recreating the type.
- Enum labels are case-sensitive and whitespace-sensitive.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Creating an enum with ordered status values

```sql
-- Step 1: Create an enum for order statuses in logical progression order.
CREATE TYPE order_status AS ENUM (
    'draft',
    'submitted',
    'approved',
    'shipped',
    'delivered',
    'cancelled'
);

-- Step 2: Create a table using the enum.
CREATE TABLE orders (
    order_id serial PRIMARY KEY,
    status   order_status NOT NULL DEFAULT 'draft'
);

-- Step 3: Insert rows and sort by the enum.
INSERT INTO orders (status) VALUES
    ('delivered'), ('approved'), ('draft'), ('shipped');

SELECT status FROM orders ORDER BY status;
```

**Expected output:**

```text
   status
------------
 draft
 approved
 shipped
 delivered
(4 rows)
```

**Why this output appears**: The enum’s declaration order is `draft → submitted → approved → shipped → delivered → cancelled`. `ORDER BY status` follows that order, not alphabetical order, so `draft` appears before `approved`, and `delivered` appears last among the inserted values.

#### Real-World Cases

- **Workflow systems**: An `approval_status` enum is created with the exact progression of states a document passes through.
- **E‑commerce**: An `order_status` enum is created once and reused across the `orders` table and any status-history tables.
- **Survey responses**: A `satisfaction` enum (`'very dissatisfied'`, `'dissatisfied'`, `'neutral'`, `'satisfied'`, `'very satisfied'`) provides a natural ordinal scale.

---

### 3. Enum Operations

#### Definitions

**Core Definition**
Enum operations are the supported actions on enum values and types: comparison, ordering, aggregation, casting to and from text, adding new values with `ALTER TYPE ... ADD VALUE`, and renaming values with `ALTER TYPE ... RENAME VALUE`.

**Technical Definition**
PostgreSQL supports **all standard comparison operators** (`=`, `<>`, `<`, `>`, `<=`, `>=`) and related aggregate functions (`MIN`, `MAX`, and others) on enum values. The `ORDER BY` clause sorts enums by declaration order. Adding a value is done with `ALTER TYPE ... ADD VALUE 'new_label' [ BEFORE | AFTER 'existing_label' ]`. Renaming is done with `ALTER TYPE ... RENAME VALUE 'old' TO 'new'`. There is **no** `DROP VALUE`; removing a value requires recreating the type and migrating data. Casting between enum and `text` is explicit and is the only way to compare values from different enum types.

**Beginner-Friendly Explanation**
You can compare and sort enums naturally, take `MIN` and `MAX`, add new choices to the menu with `ALTER TYPE`, and rename a choice. But you cannot remove a choice from the menu—you would have to rebuild the whole menu and update every row that used the old choice.

#### Purposes

- To filter and sort rows by enum values using standard SQL operators.
- To evolve the vocabulary by adding new values without recreating the type.
- To correct or clarify a label by renaming it.
- To bridge enum types (or compare with text) by casting.

#### Syntax Rules and Structure

| Operation | Syntax | Notes |
|-----------|--------|-------|
| Add value | `ALTER TYPE type_name ADD VALUE 'new' [ BEFORE \| AFTER 'existing' ]` | Cannot be done inside a transaction block in older PostgreSQL versions (v12+ often allows it). |
| Rename value | `ALTER TYPE type_name RENAME VALUE 'old' TO 'new'` | The underlying stored value is updated. |
| Compare | `enum_col = 'label'` | Uses declaration order for `<`, `>`, etc. |
| Aggregate | `MIN(enum_col)`, `MAX(enum_col)` | Returns the enum value, not text. |
| Cast to text | `enum_col::text` | Required for cross-enum comparison. |

**Syntax rules:**

- All standard comparison operators work on a single enum type.
- Cross-type comparison requires `::text` casts on both sides.
- `ADD VALUE` with `BEFORE` or `AFTER` controls the new value’s position in the sort order.

**Constraints and limitations:**

- There is no `DROP VALUE`; removing a value requires recreating the type.
- Each new value is a schema change, which creates deployment friction for frequently changing vocabularies.
- Changing the sort order of existing values requires recreating the type.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Adding and renaming enum values

```sql
-- Step 1: Create an enum.
CREATE TYPE order_status AS ENUM ('draft', 'submitted', 'approved', 'shipped');

-- Step 2: Add a new value before 'shipped'.
ALTER TYPE order_status ADD VALUE 'partially_shipped' BEFORE 'shipped';

-- Step 3: Rename 'draft' to 'new'.
ALTER TYPE order_status RENAME VALUE 'draft' TO 'new';

-- Step 4: Inspect the enum labels and their order.
SELECT enumlabel, enumsortorder
FROM pg_enum
WHERE enumtypid = 'order_status'::regtype
ORDER BY enumsortorder;
```

**Expected output:**

```text
     enumlabel      | enumsortorder
--------------------+---------------
 new                |             1
 submitted          |             2
 approved           |             3
 partially_shipped  |             4
 shipped            |             5
(5 rows)
```

**Why this output appears**: `ADD VALUE ... BEFORE 'shipped'` inserts `partially_shipped` immediately before `shipped`, shifting `shipped`’s sort order. `RENAME VALUE` changes the label `draft` to `new` without changing its position. The catalog query shows the final labels and their sort orders.

##### Example 2: Aggregating and casting enums

```sql
-- Step 1: Use MIN and MAX on an enum column.
SELECT MIN(status) AS min_status, MAX(status) AS max_status
FROM orders;

-- Step 2: Cast an enum to text for comparison with another enum or text.
SELECT status::text FROM orders WHERE status::text = 'shipped';
```

**Expected output:**

```text
 min_status | max_status
------------+------------
 new        | delivered
(1 row)

 status
----------
 shipped
(1 row)
```

**Why this output appears**: `MIN` and `MAX` respect the enum’s declaration order, returning the earliest and latest values present. The explicit `::text` cast allows comparison with a string literal and is also the workaround for cross-enum comparison.

#### Real-World Cases

- **Workflow evolution**: A support-ticket workflow starts with `('open', 'in_progress', 'resolved', 'closed')` and later adds `'waiting_on_customer'` with `BEFORE 'resolved'` to reflect the new state in the correct position.
- **Label correction**: A typo in a status label (`'recieved'`) is fixed with `RENAME VALUE` without touching the data.
- **Cross-system reporting**: Enum values are cast to `text` when joining with external systems that store status names as strings.

---

### 4. Enum Best Practices

#### Definitions

**Core Definition**
Enum best practices are the guidelines for deciding when to use an enum versus a `CHECK` constraint or a reference (lookup) table, how to evolve enums safely, and how to avoid the common pitfalls of rigid vocabularies.

**Technical Definition**
Enums are best suited for **closed, stable, value-only vocabularies** where the set of values is unlikely to change frequently and no extra metadata is needed. A **reference table** is preferable when you need extra columns (display labels, sort orders, flags), foreign-key targets, or runtime additions without schema changes. A `CHECK` constraint on a `text` column is a middle ground that is easy to apply but lacks the compact storage and intrinsic ordering of an enum. When adding enum values, be aware that `ALTER TYPE ... ADD VALUE` is a schema change and may not be allowed inside a transaction in older PostgreSQL versions. If the vocabulary changes frequently, the deployment friction of enums can become a liability.

**Beginner-Friendly Explanation**
Use an enum when the list of allowed values is basically fixed—like days of the week or a small set of workflow states. Do not use an enum if you expect to add or remove values often, or if you need to store extra information about each value (a display name, a sort order, a “finished” flag). In those cases, a reference table is more flexible. And remember: adding a value to an enum is a schema change, so it requires a deployment, not just an `INSERT`.

#### Purposes

- To avoid the rigidity of enums when the vocabulary is likely to evolve.
- To choose the representation that best balances type safety, storage, and operational flexibility.
- To evolve enums safely with `ADD VALUE` and `RENAME VALUE` while understanding their limitations.
- To prevent deployment friction caused by frequent schema changes.

#### Syntax Rules and Structure

**Decision guide:**

| Scenario | Recommended approach |
|----------|----------------------|
| Closed, stable, value-only vocabulary (e.g., weekdays, months) | `ENUM` |
| Vocabulary may change at runtime, or extra metadata is needed | Reference (lookup) table |
| Simple validation with minimal operational overhead | `text` with `CHECK` constraint |
| Need foreign-key targets or cascade behaviour | Reference table |
| Need compact storage and intrinsic ordering | `ENUM` |

**Best-practice patterns:**

```sql
-- Good fit for an enum: closed, stable vocabulary.
CREATE TYPE weekday AS ENUM (
    'Monday','Tuesday','Wednesday','Thursday','Friday','Saturday','Sunday'
);

-- Better fit for a reference table: statuses with metadata and frequent changes.
CREATE TABLE order_status (
    code        text PRIMARY KEY,
    label       text NOT NULL,
    sort_order  smallint NOT NULL,
    is_terminal boolean NOT NULL
);
```

**Component breakdown:**

| Practice | Implementation |
|----------|----------------|
| Choose enum for stability | Closed, value-only sets that rarely change |
| Choose reference table for flexibility | Runtime additions, extra columns, FK targets |
| Add values carefully | `ALTER TYPE ... ADD VALUE` is a schema change |
| Avoid frequent enum churn | Each new value requires a deployment |

**Constraints and limitations:**

- Enums cannot have values removed without recreating the type.
- Enums cannot carry extra metadata.
- `ALTER TYPE ... ADD VALUE` may be blocked inside a transaction in older PostgreSQL versions.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Choosing an enum for a stable vocabulary

```sql
-- Step 1: Create an enum for a genuinely stable set of values.
CREATE TYPE subscription_tier AS ENUM ('free', 'basic', 'premium', 'enterprise');

-- Step 2: Use it in a table.
CREATE TABLE accounts (
    account_id serial PRIMARY KEY,
    tier       subscription_tier NOT NULL DEFAULT 'free'
);

-- Step 3: Query by tier ordering.
SELECT * FROM accounts WHERE tier >= 'premium' ORDER BY tier;
```

**Expected output:** (rows with `premium` and `enterprise` tiers, in that order.)

**Why this output appears**: The tier progression is stable and logically ordered. The enum enforces the vocabulary and provides the correct sort order without a lookup table or `CHECK` constraint.

##### Example 2: Refactoring an enum to a reference table when metadata is needed

```sql
-- Step 1: Create a reference table instead of an enum.
CREATE TABLE ticket_status (
    code        text PRIMARY KEY,
    label       text NOT NULL,
    sort_order  smallint NOT NULL,
    is_closed   boolean NOT NULL DEFAULT false
);

-- Step 2: Seed the reference data.
INSERT INTO ticket_status VALUES
    ('open',        'Open',        10, false),
    ('in_progress', 'In Progress', 20, false),
    ('resolved',    'Resolved',    30, true),
    ('closed',      'Closed',      40, true);

-- Step 3: Use it with a foreign key.
CREATE TABLE tickets (
    ticket_id serial PRIMARY KEY,
    status    text NOT NULL DEFAULT 'open' REFERENCES ticket_status(code)
);
```

**Expected output:** (tables created and seeded successfully.)

**Why this output appears**: The reference table stores extra metadata (`label`, `sort_order`, `is_closed`) that an enum cannot hold. New statuses can be added with a plain `INSERT`, and child tables can reference the status with foreign-key integrity—advantages that justify the small join overhead.

#### Real-World Cases

- **Weekdays and months**: Enums are an excellent fit because the vocabulary is closed, stable, and has a natural order.
- **Order statuses**: If the status list is small and changes rarely, an enum is compact and type-safe; if statuses change frequently or require display labels and terminal flags, a reference table is better.
- **Feature flags**: A reference table is almost always preferable because flags are added, deprecated, and annotated with metadata over time.

---

## References

- PostgreSQL Documentation – Enumerated Types – https://www.postgresql.org/docs/15/datatype-enum.html
- PostgreSQL Documentation – `CREATE TYPE` – https://www.postgresql.org/docs/devel/sql-createtype.html
- PostgreSQL Documentation – Enumerated Types (PostgreSQL 18) – https://www.postgresql.org/docs/current/datatype-enum.html
- PostgreSQL Wiki – Enum – https://wiki.postgresql.org/wiki/Enum
- Enums vs Reference Tables – https://github.com/damusix/skills/blob/efa8de4096f167cc896af6b869412def1614ccf0/postgres-writing-guidelines/references/enums-vs-references.md
- Cybertec – What is better: a lookup table or an enum type? – https://www.cybertec-postgresql.com/en/lookup-table-or-enum-type/
- Sixteen Pillars – Postgres Enum Vs Lookup Table Vs Check Constraint – https://sixteenpillars.com/p/postgres-enum-vs-lookup-table-vs-check-constraint/
- AWS – Custom data types and enumerations with RDS for PostgreSQL – https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.PostgreSQL.CommonDBATasks.CustomDataTypes.html