# Comprehensive Programming Cheat Sheet: SQL Custom Types

## Topic Overview

### Definitions

**Core Definition**
A custom type is a user-defined data type created with `CREATE TYPE` that extends a PostgreSQL database beyond its built-in types, allowing you to define new scalar types, composite structures, enumerated vocabularies, and range types tailored to an application's data model.

**Technical Definition**
`CREATE TYPE` registers a new data type for use in the current database, with the defining user becoming the owner. There are **five forms** of `CREATE TYPE`: a **composite type** (a row-like structure of named fields), an **enum type** (a fixed, ordered list of labels), a **range type** (a span over a subtype with a b-tree operator class), a **base type** (a new scalar type implemented with input/output functions), and a **shell type** (a placeholder used for forward references). The type name must be distinct from all existing types, domains, and tables in the same schema. Custom types integrate fully with SQL: they can be used as column types, function parameters and return types, and array element types, and PostgreSQL automatically creates an associated array type for every user-defined type.

**Beginner-Friendly Explanation**
A custom type is a new kind of data that you teach the database to understand. Instead of being limited to `integer`, `text`, and `date`, you can say “create a type called `mood` that only allows `'sad'`, `'ok'`, or `'happy'`,” or “create a type called `complex` that bundles a real part and an imaginary part.” PostgreSQL then treats that new type like a first-class citizen: you can use it in tables, pass it to functions, and even build arrays of it. The five forms correspond to five different kinds of custom data: a **row of fields** (composite), a **fixed menu of choices** (enum), a **span of values** (range), an **entirely new scalar** (base), or a **placeholder** (shell) used while building the others.

### Key Characteristics

- **Five distinct forms**: Each form of `CREATE TYPE` solves a different modelling problem—composite for grouping fields, enum for fixed vocabularies, range for intervals, base for new scalars, and shell as a forward-declaration placeholder.
- **Ownership and naming**: The creator becomes the type's owner; the type name must be unique among types, domains, and tables in the same schema.
- **Automatic array types**: Creating any user-defined type automatically creates an array type whose name is the element type's name prefixed with an underscore, and this array type is tracked and dropped alongside its element type.
- **Superuser-only base types**: Creating a new base (scalar) type requires superuser privilege, because an erroneous type definition could confuse or crash the server.
- **Shell types enable circular references**: A shell type—`CREATE TYPE name` with no parameters—is a placeholder that has only a name and an owner, allowing input/output functions or canonical functions to be declared before the full type definition exists.
- **Constraints are limited**: Composite types and enums cannot carry constraints directly; use domains over the type, column-level constraints, or `ALTER TYPE` for enum value management.

### Prerequisites

- A running PostgreSQL instance and a client connection.
- `CREATE` privilege on the schema where the type will be created.
- `USAGE` privilege on all attribute types (for composite types) or the subtype (for range types).
- **Superuser privilege** for base types.
- Familiarity with `CREATE TABLE`, `CREATE FUNCTION`, and the distinction between SQL and low-level (C) functions for base types.

### Related Programming Areas

- Data Definition Language (DDL) – `CREATE TYPE`, `ALTER TYPE`, `DROP TYPE`.
- Data Manipulation Language (DML) – inserting, querying, and casting custom type values.
- Function design – input/output functions, canonical functions, and subtype diff functions.
- Data modelling – custom types versus domains, JSONB, and reference tables.
- PL/pgSQL programming – composite variables, enum parameters, and range operations.

---

## Core Concepts and Features

### 1. Custom Types: Overview and `CREATE TYPE`

#### Definitions

**Core Definition**
`CREATE TYPE` is the DDL statement that registers a new data type in the current database, in one of five forms.

**Technical Definition**
The five forms are: `CREATE TYPE name AS ( ... )` for composite types; `CREATE TYPE name AS ENUM ( ... )` for enum types; `CREATE TYPE name AS RANGE ( ... )` for range types; `CREATE TYPE name ( INPUT = ..., OUTPUT = ... )` for base types; and `CREATE TYPE name` for shell types. The full syntax synopsis includes optional parameters for base types such as `RECEIVE`, `SEND`, `TYPMOD_IN`, `TYPMOD_OUT`, `ANALYZE`, `SUBSCRIPT`, `INTERNALLENGTH`, `PASSEDBYVALUE`, `ALIGNMENT`, `STORAGE`, `LIKE`, `CATEGORY`, `PREFERRED`, `DEFAULT`, `ELEMENT`, `DELIMITER`, and `COLLATABLE`.

**Beginner-Friendly Explanation**
`CREATE TYPE` is the master command for inventing new data types. You pick one of five blueprints: a bundle of fields, a fixed list of labels, a span of values, a brand-new scalar with its own input/output rules, or a temporary placeholder.

#### Purposes

- To model application concepts directly as database types (e.g., a `money` type, an `email` type, a `coordinate` type).
- To centralise validation and behaviour in a reusable type rather than repeating it in every table.
- To pass structured values to and from functions without inventing ad-hoc table structures.
- To extend PostgreSQL's type system with domain-specific scalars when built-in types are insufficient.

#### Syntax Rules and Structure

**The five forms at a glance:**

| Form | Syntax opening | Typical use |
|------|----------------|-------------|
| Composite | `CREATE TYPE name AS ( field type, ... )` | Bundling related fields into a row-like value |
| Enum | `CREATE TYPE name AS ENUM ( 'label', ... )` | Fixed, ordered vocabulary |
| Range | `CREATE TYPE name AS RANGE ( SUBTYPE = ... )` | Custom interval type over a subtype |
| Base | `CREATE TYPE name ( INPUT = ..., OUTPUT = ... )` | Brand-new scalar type |
| Shell | `CREATE TYPE name` | Forward reference placeholder |

**Component breakdown (base type, most complex form):**

| Parameter | Description |
|-----------|-------------|
| `INPUT` / `OUTPUT` | Required text I/O functions |
| `RECEIVE` / `SEND` | Optional binary I/O functions |
| `TYPMOD_IN` / `TYPMOD_OUT` | Optional type modifier (e.g., `char(5)`) support |
| `INTERNALLENGTH` | Internal storage length (`VARIABLE` for variable-length) |
| `PASSEDBYVALUE` | Whether values are passed by value rather than reference |
| `ALIGNMENT` / `STORAGE` | Storage alignment and TOAST strategy |
| `LIKE` | Copy representation from an existing type |
| `CATEGORY` / `PREFERRED` | Help the parser choose implicit casts |
| `DEFAULT` | Default value for columns of the type |

**Syntax rules:**

- The type name must be unique among types, domains, and tables in the same schema.
- Base types require superuser privilege.
- Shell types take no parameters other than the name.

**Constraints and limitations:**

- Composite and enum types cannot carry constraints directly.
- Base type I/O functions generally must be coded in C or another low-level language.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Composite and enum types side by side

```sql
-- Step 1: Create a composite type for a 2D coordinate.
CREATE TYPE coordinate AS (
    x double precision,
    y double precision
);

-- Step 2: Create an enum type for a fixed vocabulary of moods.
CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');

-- Step 3: Use both types as column types.
CREATE TABLE checkins (
    id        int PRIMARY KEY,
    location  coordinate,
    feeling   mood
);

-- Step 4: Insert and query.
INSERT INTO checkins VALUES (1, ROW(10.5, 20.3), 'happy');
SELECT id, (location).x AS x, feeling FROM checkins;
```

**Expected output:**

```text
CREATE TYPE
CREATE TYPE
CREATE TABLE
INSERT 0 1
 id |  x   | feeling
----+------+---------
  1 | 10.5 | happy
(1 row)
```

**Why this output appears**: The `coordinate` composite type bundles two fields, and the `mood` enum type restricts the `feeling` column to the three listed labels. The `ROW` expression constructs a composite value, and `(location).x` extracts the `x` field. The enum value `'happy'` is accepted because it is in the declared vocabulary.

#### Real-World Cases

- **Value objects**: A `coordinate` composite type is reused across mapping, logistics, and gaming tables.
- **Workflow states**: A `status` enum centralises the allowed states of an approval process.
- **Domain-specific scalars**: A base type for `isbn` encapsulates ISBN validation in C-level input functions, providing validation no `CHECK` constraint could match.

---

### 2. Composite Types

#### Definitions

**Core Definition**
A composite type is a custom type that represents a row or record—a list of named fields with their data types—without creating an actual table.

**Technical Definition**
The first form of `CREATE TYPE` creates a composite type specified by a list of attribute names and data types, with optional collations for collatable attributes. A composite type is essentially the same as the row type of a table, but `CREATE TYPE` avoids the need to create an actual table when only a type definition is wanted; a stand-alone composite type is useful as the argument or return type of a function. Creating a composite type requires `USAGE` privilege on all attribute types.

**Beginner-Friendly Explanation**
A composite type is a labelled container for several related values—like a struct in a programming language. It lets you treat a bundle of fields as one value, which is convenient for function arguments and for columns that always travel together.

#### Purposes

- To define reusable “value objects” without creating a separate table for every small structure.
- To serve as function arguments and return types that carry multiple values.
- To group related fields in a column when they are always read and written together.

#### Syntax Rules and Structure

**General syntax:**

```sql
CREATE TYPE name AS (
    attribute_name data_type [ COLLATE collation ] [ , ... ]
);
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `attribute_name` | A field name; unique within the type. |
| `data_type` | Any PostgreSQL data type, including other composite types. |
| `COLLATE` | Optional collation for collatable attribute types. |

**Syntax rules:**

- No constraints can be attached to a composite type definition.
- The type name must be distinct from existing types, domains, and tables.
- Accessing fields requires parentheses around the composite column: `(col).field`.

**Constraints and limitations:**

- Composite types cannot carry `NOT NULL`, `CHECK`, or other constraints.
- `ALTER TYPE` does not support the same full range of modifications for composite types as it does for enums; composite changes may require dropping and recreating.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Composite type as a function argument

```sql
-- Step 1: Create a composite type.
CREATE TYPE inventory_item AS (
    name        text,
    supplier_id integer,
    price       numeric
);

-- Step 2: Create a function that accepts the composite type.
CREATE FUNCTION item_value(item inventory_item, qty integer)
RETURNS numeric AS $$
    SELECT (item).price * qty;
$$ LANGUAGE SQL;

-- Step 3: Call the function with a ROW expression.
SELECT item_value(ROW('widget', 42, 9.99), 10) AS total;
```

**Expected output:**

```text
CREATE TYPE
CREATE FUNCTION
 total
-------
  99.90
(1 row)
```

**Why this output appears**: The composite type `inventory_item` defines three fields. The function `item_value` accepts one value of that type and an integer quantity, extracts the `price` field with `(item).price`, and multiplies by the quantity. The `ROW` expression constructs the composite value inline.

#### Real-World Cases

- **Geographic coordinates**: A `geo_point` composite groups latitude and longitude for use across multiple tables and functions.
- **Money amounts**: A `money` composite pairs an amount with a currency code, ensuring they are always handled together.
- **Function signatures**: A function that processes an address accepts an `address` composite rather than four separate parameters, simplifying call sites.

---

### 3. Enumerated Types

#### Definitions

**Core Definition**
An enum type is a custom type consisting of a fixed, ordered list of string labels, restricting a column to that vocabulary and using the declaration order for sorting and comparison.

**Technical Definition**
The second form of `CREATE TYPE` creates an enumerated (enum) type. Enum types take a list of quoted labels, each shorter than `NAMEDATALEN` bytes (64 bytes in a standard build); it is possible to create an enum with zero labels, but such a type cannot hold values until at least one label is added with `ALTER TYPE ... ADD VALUE`. Enum values are stored as 4 bytes internally, with the textual label resolved via the catalog. Labels are case-sensitive and whitespace-sensitive, and each enum type is distinct—cross-type comparison requires a cast to `text`.

**Beginner-Friendly Explanation**
An enum is a dropdown menu of allowed values. The database rejects anything not on the list, and the order you list the values in becomes the sort order.

#### Purposes

- To enforce a fixed vocabulary at the column level without a `CHECK` constraint or a reference-table join.
- To provide an intrinsic sort order that matches the logical progression of values.
- To reduce storage compared with repeating full text labels.

#### Syntax Rules and Structure

**General syntax:**

```sql
CREATE TYPE name AS ENUM ( [ 'label' [, ... ] ] );
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `'label'` | A quoted string; case-sensitive and whitespace-sensitive. |
| Declaration order | Defines the sort order for `<`, `>`, `ORDER BY`, `MIN`, `MAX`. |

**Syntax rules:**

- Labels must be shorter than `NAMEDATALEN` bytes (64 by default).
- An empty enum can be created but cannot hold values until a label is added.
- Each enum type is distinct; cross-type comparison requires `::text` casts.

**Constraints and limitations:**

- There is no `ALTER TYPE ... DROP VALUE`; removing an enum value requires recreating the type and migrating data.
- Adding a value with `ALTER TYPE ... ADD VALUE` is a schema change and cannot be rolled back inside a transaction in older PostgreSQL versions.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Enum ordering and value addition

```sql
-- Step 1: Create an enum.
CREATE TYPE order_status AS ENUM ('draft', 'submitted', 'approved', 'shipped');

-- Step 2: Add a value before 'shipped'.
ALTER TYPE order_status ADD VALUE 'partially_shipped' BEFORE 'shipped';

-- Step 3: Create a table and insert values.
CREATE TABLE orders (id int, status order_status);
INSERT INTO orders VALUES (1, 'approved'), (2, 'partially_shipped'), (3, 'draft');

-- Step 4: Sort by enum order.
SELECT status FROM orders ORDER BY status;
```

**Expected output:**

```text
CREATE TYPE
ALTER TYPE
CREATE TABLE
INSERT 0 3
       status
---------------------
 draft
 approved
 partially_shipped
(3 rows)
```

**Why this output appears**: The enum's declaration order (after the `ADD VALUE ... BEFORE`) is `draft → submitted → approved → partially_shipped → shipped`. `ORDER BY status` follows that order, not alphabetical order, so `draft` appears first and `partially_shipped` appears after `approved`.

#### Real-World Cases

- **Order workflows**: A `status` enum encodes the exact progression of states an order passes through.
- **Priority levels**: A `priority` enum (`'low'`, `'medium'`, `'high'`, `'critical'`) provides the correct severity order for sorting.
- **Days of the week**: A `weekday` enum in weekday order enables simple range queries like `WHERE day > 'Wednesday'`.

---

### 4. Range Types

#### Definitions

**Core Definition**
A range type is a custom type that represents a span of values of a subtype, with a b-tree operator class determining the ordering and optional canonical, difference, and multirange configuration.

**Technical Definition**
The third form of `CREATE TYPE` creates a new range type. The range type's subtype can be any type with an associated b-tree operator class; normally the subtype's default operator class is used, but a non-default one can be specified with `SUBTYPE_OPCLASS`. Optional parameters include `COLLATION`, `CANONICAL` (a function converting range values to canonical form), `SUBTYPE_DIFF` (a function returning a `double precision` difference, enabling more efficient GiST indexes), and `MULTIRANGE_TYPE_NAME` (the name of the corresponding multirange type). Creating a range type requires `USAGE` privilege on the subtype.

**Beginner-Friendly Explanation**
A range type is a custom ruler that measures spans of some underlying value. You tell PostgreSQL which subtype the range is over, and it builds a full interval type with overlap, containment, and union operators.

#### Purposes

- To define domain-specific interval types that behave like the built-in range types.
- To attach canonicalisation or difference functions for efficient indexing and correct equality.
- To model spans of custom scalar types (e.g., a range over a `money` base type).

#### Syntax Rules and Structure

**General syntax:**

```sql
CREATE TYPE name AS RANGE (
    SUBTYPE = subtype
    [ , SUBTYPE_OPCLASS = subtype_operator_class ]
    [ , COLLATION = collation ]
    [ , CANONICAL = canonical_function ]
    [ , SUBTYPE_DIFF = subtype_diff_function ]
    [ , MULTIRANGE_TYPE_NAME = multirange_type_name ]
);
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `SUBTYPE` | The element type of the range; must have a b-tree operator class. |
| `CANONICAL` | Function converting a range to canonical form. |
| `SUBTYPE_DIFF` | Function returning the difference between two subtype values. |
| `MULTIRANGE_TYPE_NAME` | Name of the associated multirange type; auto-derived if omitted. |

**Syntax rules:**

- The canonical function must be declared before the range type, which requires a shell type for the forward reference.
- If the subtype name contains `range`, the multirange name replaces it with `multirange`; otherwise `_multirange` is appended.

**Constraints and limitations:**

- Building a range type with a canonical function requires a multi-step process with a shell type.
- The subtype must have a b-tree operator class.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Creating a range type over a custom subtype

```sql
-- Step 1: Create a base type for the subtype (simplified as a domain for illustration).
CREATE DOMAIN temperature AS numeric CHECK (VALUE > -273.15);

-- Step 2: Create a range type over the temperature domain.
CREATE TYPE temp_range AS RANGE (SUBTYPE = temperature);

-- Step 3: Use the range type in a table.
CREATE TABLE readings (
    id     int PRIMARY KEY,
    period temp_range NOT NULL
);

-- Step 4: Insert and query.
INSERT INTO readings VALUES (1, '[20.0, 25.0)');
SELECT id, period FROM readings;
```

**Expected output:**

```text
CREATE DOMAIN
CREATE TYPE
CREATE TABLE
INSERT 0 1
 id |   period
----+-------------
  1 | [20.0,25.0)
(1 row)
```

**Why this output appears**: The `temp_range` type is a range over the `temperature` domain, inheriting all the standard range operators. The value `[20.0, 25.0)` is stored and rendered in the canonical range notation.

#### Real-World Cases

- **Custom measurement ranges**: A range over a `money` type models price bands with exact decimal endpoints.
- **Temporal variants**: A range over a custom `fiscal_date` type supports fiscal-period intervals with domain-specific arithmetic.

---

### 5. Base Types

#### Definitions

**Core Definition**
A base type (scalar type) is a completely new data type implemented with input/output functions, allowing PostgreSQL to store, display, and operate on values that no built-in type can represent.

**Technical Definition**
The fourth form of `CREATE TYPE` creates a new base type (scalar type). Creating a base type requires **superuser privilege**, because an erroneous type definition could confuse or even crash the server. You must register two or more functions before defining the type: `INPUT` and `OUTPUT` are required, while `RECEIVE`, `SEND`, `TYPMOD_IN`, `TYPMOD_OUT`, `ANALYZE`, and `SUBSCRIPT` are optional. Generally these functions must be coded in C or another low-level language. The input function converts the external textual representation to the internal representation, and the output function performs the reverse transformation. Binary I/O functions (`RECEIVE` and `SEND`) enable binary input and output. A **shell type** is used to declare the I/O functions before the full type exists.

**Beginner-Friendly Explanation**
A base type is a brand-new scalar—something PostgreSQL has never seen before, like an ISBN, a MAC address, or a chemical formula. You have to teach PostgreSQL how to read it (input), how to print it (output), and optionally how to send and receive it in binary. Because this is low-level and potentially dangerous, only superusers can do it.

#### Purposes

- To add domain-specific scalar types that built-in types cannot represent faithfully.
- To encapsulate validation and normalisation in C-level input functions for performance and correctness.
- To provide binary I/O for interoperability with external systems and wire protocols.

#### Syntax Rules and Structure

**General syntax:**

```sql
CREATE TYPE name (
    INPUT = input_function,
    OUTPUT = output_function
    [ , RECEIVE = receive_function ]
    [ , SEND = send_function ]
    [ , TYPMOD_IN = type_modifier_input_function ]
    [ , TYPMOD_OUT = type_modifier_output_function ]
    [ , ANALYZE = analyze_function ]
    [ , SUBSCRIPT = subscript_function ]
    [ , INTERNALLENGTH = { internallength | VARIABLE } ]
    [ , PASSEDBYVALUE ]
    [ , ALIGNMENT = alignment ]
    [ , STORAGE = storage ]
    [ , LIKE = like_type ]
    [ , CATEGORY = category ]
    [ , PREFERRED = preferred ]
    [ , DEFAULT = default ]
    [ , ELEMENT = element ]
    [ , DELIMITER = delimiter ]
    [ , COLLATABLE = collatable ]
);
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `INPUT` | Converts text to internal form; returns the type itself |
| `OUTPUT` | Converts internal form to text; returns `cstring` |
| `RECEIVE` | Converts binary input to internal form; optional |
| `SEND` | Converts internal form to binary; returns `bytea` |
| `TYPMOD_IN` / `TYPMOD_OUT` | Support type modifiers like `char(5)` |
| `ANALYZE` | Custom statistics collection for `ANALYZE` |
| `INTERNALLENGTH` | `VARIABLE` for variable-length types |
| `LIKE` | Copy representation from an existing type |
| `CATEGORY` / `PREFERRED` | Guide implicit cast selection |

**Syntax rules:**

- Superuser privilege is required.
- `INPUT` and `OUTPUT` are mandatory; all other functions and parameters are optional.
- A shell type must be created first if the I/O functions need to reference the new type.

**Constraints and limitations:**

- Base type functions must generally be written in C.
- Incorrect type definitions can crash the server, hence the superuser restriction.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Shell type pattern for a base type

```sql
-- Step 1: Create a shell type as a forward reference.
CREATE TYPE box;

-- Step 2: Define I/O functions referencing the shell type.
CREATE FUNCTION box_in(cstring) RETURNS box AS 'MODULE_PATHNAME' LANGUAGE C STRICT;
CREATE FUNCTION box_out(box) RETURNS cstring AS 'MODULE_PATHNAME' LANGUAGE C STRICT;

-- Step 3: Complete the type definition.
CREATE TYPE box (
    INTERNALLENGTH = 16,
    INPUT = box_in,
    OUTPUT = box_out
);
```

**Expected output:**

```text
CREATE TYPE
CREATE FUNCTION
CREATE FUNCTION
CREATE TYPE
```

**Why this output appears**: The shell type `box` is created first so that the C functions `box_in` and `box_out` can be declared with `box` as an argument or return type. The final `CREATE TYPE` with the full definition replaces the shell entry with a complete type.

#### Real-World Cases

- **Network address types**: PostgreSQL's own `inet` and `cidr` types are base types with C-level input/output functions.
- **Geometric types**: The built-in `point`, `line`, and `box` types follow the same shell-type pattern.
- **Domain-specific scalars**: An `isbn` base type validates ISBN check digits in C, providing validation faster and more thoroughly than a regex `CHECK`.

---

### 6. Shell Types

#### Definitions

**Core Definition**
A shell type is a placeholder type with no properties except a name and an owner, created to allow forward references when defining base types and range types.

**Technical Definition**
A shell type is created by issuing `CREATE TYPE name` with no parameters other than the type name. It is needed when the type's I/O functions or canonical function must be declared before the full type definition exists. After the functions are defined, `CREATE TYPE` with a full definition replaces the shell entry with a complete, valid type definition, after which the new type can be used normally.

**Beginner-Friendly Explanation**
A shell type is a “coming soon” sign. It reserves the type's name so you can write functions that mention it, and later you replace the sign with the real thing.

#### Purposes

- To break the circular dependency between a type and its I/O functions.
- To allow range canonical functions to be declared before the range type exists.

#### Syntax Rules and Structure

**General syntax:**

```sql
CREATE TYPE name;
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `name` | The name of the placeholder type. |

**Syntax rules:**

- Shell types take no parameters other than the name.
- The shell entry is replaced when the full `CREATE TYPE` is issued later.

**Constraints and limitations:**

- A shell type has no properties beyond a name and an owner; it cannot be used as a column type until completed.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Shell type for a range canonical function

```sql
-- Step 1: Create a shell type for the range.
CREATE TYPE floatrange;

-- Step 2: Define a canonical function using the shell type.
CREATE FUNCTION floatrange_canonical(floatrange) RETURNS floatrange AS 'float8_range_canonical' LANGUAGE C IMMUTABLE STRICT;

-- Step 3: Complete the range type definition.
CREATE TYPE floatrange AS RANGE (
    SUBTYPE = float8,
    CANONICAL = floatrange_canonical
);
```

**Expected output:**

```text
CREATE TYPE
CREATE FUNCTION
CREATE TYPE
```

**Why this output appears**: The shell type `floatrange` allows the canonical function to be declared with `floatrange` as its argument and return type before the range type itself exists. The final `CREATE TYPE` replaces the shell entry with the complete range type.

#### Real-World Cases

- **Custom range types with canonical functions**: Any range type that needs a canonical function requires a shell type as an intermediate step.
- **Base type development**: Every base type that has I/O functions referencing the type itself follows the shell-type pattern.

---

### 7. Custom Type Operations: `ALTER TYPE` and `DROP TYPE`

#### Definitions

**Core Definition**
Custom type operations are the DDL actions that modify an existing type with `ALTER TYPE` or remove it with `DROP TYPE`.

**Technical Definition**
`ALTER TYPE` changes the definition of an existing type and has several subforms, including `OWNER TO`, `RENAME TO`, `RENAME ATTRIBUTE`, `ADD VALUE` (for enums), `RENAME VALUE` (for enums), `SET SCHEMA`, `SET ( property = value )`, and `DROP ATTRIBUTE` (for composite types). You must own the type to use `ALTER TYPE`. `DROP TYPE [ IF EXISTS ] name [, ...] [ CASCADE | RESTRICT ]` removes a user-defined type; only the owner can drop it, `CASCADE` automatically drops dependent objects, and `RESTRICT` refuses to drop if dependencies exist.

**Beginner-Friendly Explanation**
`ALTER TYPE` lets you change a type after it exists—rename it, change its owner, add enum values, or move it to another schema. `DROP TYPE` deletes it entirely, but only if nothing depends on it unless you say `CASCADE`.

#### Purposes

- To evolve enum vocabularies with `ADD VALUE` and `RENAME VALUE`.
- To rename types, attributes, or schema membership.
- To remove obsolete types, optionally cascading to dependent columns.

#### Syntax Rules and Structure

**Key `ALTER TYPE` sub-forms:**

| Operation | Syntax |
|-----------|--------|
| Rename type | `ALTER TYPE name RENAME TO new_name` |
| Rename attribute (composite) | `ALTER TYPE name RENAME ATTRIBUTE old TO new [ CASCADE \| RESTRICT ]` |
| Add enum value | `ALTER TYPE name ADD VALUE [ IF NOT EXISTS ] 'new' [ BEFORE \| AFTER 'existing' ]` |
| Rename enum value | `ALTER TYPE name RENAME VALUE 'old' TO 'new'` (PG 10+) |
| Set schema | `ALTER TYPE name SET SCHEMA new_schema` |
| Change owner | `ALTER TYPE name OWNER TO new_owner` |
| Drop attribute (composite) | `ALTER TYPE name DROP ATTRIBUTE [ IF EXISTS ] attr [ CASCADE \| RESTRICT ]` |

**`DROP TYPE` syntax:**

```sql
DROP TYPE [ IF EXISTS ] name [, ...] [ CASCADE | RESTRICT ];
```

**Syntax rules:**

- `ALTER TYPE` requires ownership of the type.
- `DROP TYPE` requires ownership; `RESTRICT` is the default.
- There is no `ALTER TYPE ... DROP VALUE` for enums; removing a value requires recreating the type.

**Constraints and limitations:**

- Enum value removal is a manual, multi-step operation involving type recreation and data migration.
- `DROP TYPE ... CASCADE` can drop dependent table columns, which is destructive.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Adding and renaming enum values

```sql
-- Step 1: Create an enum.
CREATE TYPE status AS ENUM ('new', 'open', 'closed');

-- Step 2: Add a value after 'open'.
ALTER TYPE status ADD VALUE 'pending' AFTER 'open';

-- Step 3: Rename a value.
ALTER TYPE status RENAME VALUE 'new' TO 'created';

-- Step 4: Inspect the labels.
SELECT enumlabel FROM pg_enum WHERE enumtypid = 'status'::regtype ORDER BY enumsortorder;
```

**Expected output:**

```text
CREATE TYPE
ALTER TYPE
ALTER TYPE
 enumlabel
-----------
 created
 open
 pending
 closed
(4 rows)
```

**Why this output appears**: `ADD VALUE ... AFTER 'open'` inserts `pending` immediately after `open`, preserving sort order. `RENAME VALUE` changes `new` to `created` without changing its position. The catalog query confirms the final label order.

##### Example 2: Dropping a type with RESTRICT versus CASCADE

```sql
-- Step 1: Create a type and a table using it.
CREATE TYPE code AS text;
CREATE TABLE items (id int, c code);

-- Step 2: Attempt to drop with RESTRICT.
DROP TYPE code;

-- Step 3: Drop with CASCADE.
DROP TYPE code CASCADE;
```

**Expected output:**

```text
CREATE TYPE
CREATE TABLE
ERROR:  cannot drop type code because other objects depend on it
DETAIL:  column c of table items depends on type code
HINT:  Use DROP ... CASCADE to drop the dependent objects too.
DROP TYPE
```

**Why this output appears**: `RESTRICT` refuses because `items.c` depends on `code`. `CASCADE` drops the dependent column along with the type, so the second `DROP TYPE` succeeds.

#### Real-World Cases

- **Enum evolution**: A workflow adds a `'waiting_on_customer'` state with `ALTER TYPE ... ADD VALUE ... BEFORE 'resolved'` to insert it in the correct position.
- **Schema refactoring**: A type is renamed and moved to a new schema with `ALTER TYPE ... RENAME TO` and `SET SCHEMA`.
- **Decommissioning**: An obsolete type is dropped with `RESTRICT` first to inspect dependencies, then `CASCADE` after dependent columns are migrated.

---

### 8. Custom Type Best Practices

#### Definitions

**Core Definition**
Custom type best practices are the guidelines for choosing the right type form, evolving types safely, avoiding common pitfalls, and knowing when a custom type is the wrong tool.

**Technical Definition**
Best practices include: (1) **use the simplest form that solves the problem**—composite for bundling fields, enum for fixed vocabularies, range for intervals, and base types only when no simpler type suffices; (2) **prefer domains over base types for constraint-only customisation**, because domains attach constraints without requiring C functions; (3) **use enums only for genuinely closed, stable vocabularies**, and switch to a reference table when values change frequently or need metadata; (4) **use `ADD VALUE ... BEFORE/AFTER` to preserve sort order** when evolving enums, and remember there is no `DROP VALUE`; (5) **use shell types for base and range types that need forward references**; and (6) **avoid overusing composite types**—they are best for value objects and function signatures, not for replacing normalized tables.

**Beginner-Friendly Explanation**
Best practices for custom types are about using the right tool for the right job. Use an enum for a fixed menu, a composite for a bundle of fields, a range for an interval, and a base type only when you truly need a new scalar. Do not use an enum if the list changes every week; use a reference table instead. When you add an enum value, place it carefully so the sort order stays correct. And remember that base types are superuser-only and written in C, so they are a last resort, not a first choice.

#### Purposes

- To avoid the rigidity and operational friction of choosing the wrong type form.
- To evolve types safely without breaking dependent columns.
- To keep schema design maintainable and understandable.
- To leverage PostgreSQL's type system without over-engineering.

#### Syntax Rules and Structure

**Decision guide:**

| Modelling need | Recommended approach |
|----------------|----------------------|
| Bundle related fields for functions or value objects | Composite type |
| Fixed, ordered vocabulary (weekdays, workflow states) | Enum type |
| Interval over a custom subtype | Range type |
| New scalar with C-level validation and binary I/O | Base type (superuser only) |
| Attach constraints to an existing type | Domain (not a base type) |
| Vocabulary changes frequently or needs metadata | Reference table |

**Best-practice patterns:**

```sql
-- Good fit for an enum: closed, stable vocabulary.
CREATE TYPE weekday AS ENUM ('Monday','Tuesday','Wednesday','Thursday','Friday','Saturday','Sunday');

-- Better fit for a reference table: statuses with metadata and frequent changes.
CREATE TABLE order_status (
    code        text PRIMARY KEY,
    label       text NOT NULL,
    sort_order  smallint NOT NULL,
    is_terminal boolean NOT NULL
);
```

**Constraints and limitations:**

- Base types require superuser and C coding; they are heavyweight.
- Enum value removal requires type recreation and data migration.
- Composite types cannot carry constraints.

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

-- Step 3: Query with enum ordering.
SELECT * FROM accounts WHERE tier >= 'premium' ORDER BY tier;
```

**Expected output:** (rows with `premium` and `enterprise` tiers, in that order.)

**Why this output appears**: The tier progression is stable and logically ordered. The enum enforces the vocabulary and provides the correct sort order without a lookup table or `CHECK` constraint.

#### Real-World Cases

- **Workflow states**: A stable set of order states is a good enum fit; a rapidly evolving set of support-ticket states is better as a reference table.
- **Value objects**: A `coordinate` composite is reused across mapping and logistics tables, justified by its repeated bundling of two fields.
- **Custom scalars**: An `isbn` base type is justified only when C-level validation and binary I/O are genuinely needed; otherwise a domain with a regex `CHECK` suffices.

---

## References

- PostgreSQL Documentation – `CREATE TYPE` – https://www.postgresql.org/docs/current/sql-createtype.html
- PostgreSQL Documentation – `ALTER TYPE` – https://www.postgresql.org/docs/current/sql-altertype.html
- PostgreSQL Documentation – `DROP TYPE` – https://www.postgresql.org/docs/current/sql-droptype.html
- PostgreSQL Documentation – User-Defined Types (Base Types) – https://www.postgresql.org/docs/current/extend-type-system.html
- PostgreSQL Documentation – Shell Types (Git source) – https://git.postgresql.org/gitweb/?p=postgresql.git;a=blob_plain;f=doc/src/sgml/ref/create_type.sgml
- dbwarden – PostgreSQL Types Lifecycle Reference – https://github.com/dbwarden-org/dbwarden/blob/main/docs/databases/postgresql/types.md
- Custom Type Best Practices Reference – https://github.com/damusix/skills/blob/main/postgres/references/15-data-types-custom.md