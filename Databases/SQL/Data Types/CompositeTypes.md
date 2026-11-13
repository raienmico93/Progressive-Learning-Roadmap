# Comprehensive Programming Cheat Sheet: PostgreSQL Composite Types

## Topic Overview

### Definitions

**Core Definition**
A composite type is a PostgreSQL data type that represents the structure of a row or record — essentially just a list of field names and their data types — allowing multiple values to be grouped together and treated as a single unit within a column, function parameter, or variable.

**Technical Definition**
PostgreSQL allows composite types to be used in many of the same ways that simple types can be used. A composite type is created with `CREATE TYPE name AS (field1 type1, field2 type2, ...)`, and its syntax is comparable to `CREATE TABLE` except that only field names and types can be specified; no constraints (such as `NOT NULL`) can presently be included. Whenever a table is created, a composite type is also automatically created with the same name as the table, representing the table's row type. Composite values can be constructed using parenthesized literals (`'(val1, val2, ...)'`) or `ROW` expressions, and fields are accessed with dot notation (`(column).field`) — parentheses are often required to disambiguate from table-name syntax. Crucially, **no constraints are associated with a standalone composite type**, so constraints defined on a table with the same name do not apply to values of the composite type outside the table; a domain over the composite type can be used to attach `CHECK` constraints.

**Beginner-Friendly Explanation**
A composite type is a custom container that bundles several related values together under a single name — like a "record" or a "struct" in a programming language. If you frequently need to store a 2D coordinate, you could create a `complex` type with fields `r` and `i`, or a `inventory_item` type with `name`, `supplier_id`, and `price`. Once defined, you can use that type as a column type in a table, as a function parameter, or as a variable in PL/pgSQL. Think of a composite type as a labelled envelope: instead of passing around three separate pieces of paper, you put them all in one envelope, write the field names on the front, and hand over the envelope as a single item. The catch is that the envelope itself does not enforce any rules about what goes inside — constraints must be added separately if needed.

### Key Characteristics

- **Row-like structure without constraints**: A composite type defines field names and data types only; it cannot carry `NOT NULL`, `CHECK`, `DEFAULT`, or other constraints. Constraints on a table with the same name apply only within that table, not to the composite type used elsewhere.
- **Automatic row types**: Every table automatically creates a composite type of the same name that represents its row type, enabling functions to accept "a row of this table" as a single argument.
- **Constructor flexibility**: Composite values can be built with parenthesized literals (`'(1.1,2.2)'`), `ROW` expressions (`ROW(1.1, 2.2)`), or simplified row syntax (`(1.1, 2.2)`), with `ROW` generally easier to read and quote.
- **Parenthesis-sensitive field access**: Accessing a field requires parentheses around the composite column — `(item).name` — because `item.name` would be parsed as `table.column` syntax.
- **Storage and performance trade-offs**: Composite types incur a storage penalty in both the heap and indexes compared with separate columns, and indexes querying only part of a composite key cannot be leveraged. However, composite types allow attributes to be added on the fly without rebuilding the index, and they can be more compact and faster than JSONB for certain workloads.
- **Limited tooling support**: Many client drivers and ORMs have incomplete or no support for composite types, and report-generation tools often struggle with them, frequently requiring views that expose a normalized appearance.

### Prerequisites

- A running PostgreSQL instance (composite types have been available since early versions; the core syntax has remained stable).
- `CREATE` privilege on the schema where the type will be created.
- Familiarity with `CREATE TABLE` and `CREATE TYPE` syntax.
- Awareness of the trade-offs between composite types, normalized child tables, and JSONB before choosing a representation.

### Related Programming Areas

- Data Definition Language (DDL) – `CREATE TYPE`, `ALTER TYPE`, `DROP TYPE`.
- Data Manipulation Language (DML) – inserting, updating, and querying composite values.
- PL/pgSQL programming – composite variables, row-type parameters, and record variables.
- Function design – functions accepting and returning composite types, and computed fields via functional notation.
- Data modelling – composite types versus normalized tables versus JSONB.

---

## Core Concepts and Features

### 1. Composite Types: Definition and Creation

#### Definitions

**Core Definition**
Composite creation is the process of defining a new composite type with `CREATE TYPE`, specifying a list of field names and data types that together form a row-like structure.

**Technical Definition**
The syntax is `CREATE TYPE type_name AS ( field1 data_type1, field2 data_type2, ... )`. An attribute's collation can also be specified if its data type is collatable. The `AS` keyword is essential; without it, PostgreSQL interprets the statement as a different form of `CREATE TYPE` and produces syntax errors. A composite type is essentially the same as the row type of a table, except that no constraints can be attached to it. It is possible to create a table using the composite type as a column type, and functions can accept or return the composite type.

**Beginner-Friendly Explanation**
Creating a composite type is like designing a form with labelled fields. You write `CREATE TYPE` followed by the type's name and a list of fields, each with a name and a data type. Once the type exists, you can use it as a column type, a function parameter, or a variable type. The database does not let you attach rules like “this field cannot be empty” to the type itself — if you need rules, you must enforce them elsewhere (for example, with a domain or application logic).

#### Purposes

- To group related fields into a reusable, named structure that can be shared across tables, functions, and PL/pgSQL code.
- To represent a “value object” or “tuple” in the data model without creating a separate table for every small structure.
- To pass multiple related values as a single argument to a function or procedure.
- To leverage the automatic row types that every table provides for functions that operate on whole rows.

#### Syntax Rules and Structure

**General syntax:**

```sql
CREATE TYPE type_name AS (
    field1 data_type1,
    field2 data_type2,
    ...
);
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `type_name` | The name of the new composite type; must be unique within its schema. |
| `field` | A field name; unique within the type. |
| `data_type` | Any PostgreSQL data type, including other composite types. |
| `AS` | Required keyword; without it, the statement is parsed as a different `CREATE TYPE` form. |

**Syntax rules:**

- No constraints (`NOT NULL`, `CHECK`, `DEFAULT`, `PRIMARY KEY`, etc.) can be attached to a composite type definition.
- The `AS` keyword is mandatory.
- Table row types are automatically created and share the table's name; a standalone type with the same name can coexist but the table's constraints do not apply to it.

**Constraints and limitations:**

- Composite types cannot have constraints; use a domain over the composite type to attach `CHECK` constraints.
- Adding a field to a composite type changes the type's definition; existing values may need to be rebuilt.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Defining and using a standalone composite type

```sql
-- Step 1: Define a composite type for a 2D complex number.
CREATE TYPE complex AS (
    r double precision,
    i double precision
);

-- Step 2: Define a composite type for an inventory item.
CREATE TYPE inventory_item AS (
    name        text,
    supplier_id integer,
    price       numeric
);

-- Step 3: Create a table that uses the composite type as a column type.
CREATE TABLE on_hand (
    item  inventory_item,
    count integer
);

-- Step 4: Insert a row using a ROW expression.
INSERT INTO on_hand VALUES (ROW('fuzzy dice', 42, 1.99), 1000);

-- Step 5: Query the composite column.
SELECT item, count FROM on_hand;
```

**Expected output:**

```text
CREATE TYPE
CREATE TYPE
CREATE TABLE
INSERT 0 1
           item           | count
--------------------------+-------
 ("fuzzy dice",42,1.99)   |  1000
(1 row)
```

**Why this output appears**: The `inventory_item` composite type is created with three fields. The table `on_hand` uses it as a column type. The `ROW` expression constructs a composite value, which is stored and rendered in the parenthesized literal format. The query returns the composite value as a single column alongside the ordinary integer column.

##### Example 2: Automatic row type versus standalone composite type

```sql
-- Step 1: Create a table with constraints.
CREATE TABLE inventory_item (
    name        text,
    supplier_id integer REFERENCES suppliers,
    price       numeric CHECK (price > 0)
);

-- Step 2: Use the table's row type as a function parameter.
CREATE FUNCTION price_extension(item inventory_item, quantity integer)
RETURNS numeric AS $$
    SELECT (item).price * quantity;
$$ LANGUAGE SQL;

-- Step 3: Call the function with a row from the table.
SELECT price_extension(item, 10) FROM on_hand;
```

**Expected output:**

```text
CREATE TABLE
CREATE FUNCTION
 price_extension
-----------------
           19.90
(1 row)
```

**Why this output appears**: Creating the `inventory_item` table automatically creates a composite type of the same name, representing the table's row type. The function accepts that row type as a parameter and extracts the `price` field using `(item).price`. The `CHECK` constraint on `price > 0` applies only within the table, not to the composite type used as a function parameter.

#### Real-World Cases

- **Value objects**: A `money` type (`amount numeric, currency char(3)`) groups an amount and its currency into a single reusable structure.
- **Geographic coordinates**: A `geo_point` type (`latitude double precision, longitude double precision`) can be used across multiple tables without repeating two columns.
- **Function parameters**: A function that processes an address can accept an `address` composite type instead of four separate parameters, simplifying the function signature and call sites.

---

### 2. Composite Operations: Construction, Access, and Modification

#### Definitions

**Core Definition**
Composite operations are the ways to construct composite values, access their individual fields, modify whole values or individual subfields, and pass them to functions and queries.

**Technical Definition**
Composite values can be constructed with parenthesized literals (`'(val1, val2, ...)'`) or `ROW` expressions (`ROW(val1, val2, ...)`), with the `ROW` keyword optional when more than one field is present. To access a field of a composite column, one writes a dot and the field name, but parentheses are often required: `(item).name` rather than `item.name`, because the latter is interpreted as `table.column`. Individual subfields can be updated with `UPDATE mytab SET complex_col.r = (complex_col).r + 1`, and subfields can be specified as `INSERT` targets. Functions accepting a single composite argument can be called with either `func(c)` or `c.func` notation, enabling computed fields. The special field name `*` means “all fields” and triggers column expansion in `SELECT`, `RETURNING`, `VALUES`, and row-constructor contexts.

**Beginner-Friendly Explanation**
Working with composite types is like working with a small form. You can fill out the whole form at once (`ROW(...)`), read a single field from it (`(item).price`), or change just one field (`UPDATE ... SET item.price = ...`). The parentheses around the composite column are essential — without them, PostgreSQL thinks you are naming a table and a column, not a field inside a composite value. Functions can take the whole form as an argument, and you can even call a function as if it were a field of the form, which is how “computed fields” work.

#### Purposes

- To construct composite values for insertion and update without verbose casts.
- To extract individual fields for filtering, joining, and projection.
- To modify specific subfields of a composite column in place.
- To pass whole rows or composite values to functions, enabling reusable logic.
- To expand all fields of a composite value into separate columns with `.*`.

#### Syntax Rules and Structure

**Construction:**

| Form | Example |
|------|---------|
| Literal | `'("fuzzy dice",42,1.99)'` |
| `ROW` expression | `ROW('fuzzy dice', 42, 1.99)` |
| Simplified row | `('fuzzy dice', 42, 1.99)` |

**Access:**

| Access form | Example |
|-------------|---------|
| Field access | `(item).name` |
| Qualified field access | `(on_hand.item).name` |
| Functional notation | `name(item)` |
| All fields | `(item).*` |

**Modification:**

| Operation | Syntax |
|-----------|--------|
| Replace whole value | `UPDATE mytab SET complex_col = ROW(1.1,2.2) WHERE ...` |
| Update subfield | `UPDATE mytab SET complex_col.r = (complex_col).r + 1 WHERE ...` |
| Insert subfields | `INSERT INTO mytab (complex_col.r, complex_col.i) VALUES (1.1, 2.2)` |

**Syntax rules:**

- Parentheses are required around a composite column when accessing a field, unless the column is a simple table name in a `FROM` clause.
- The `ROW` keyword is optional when more than one field is present.
- `.*` triggers column expansion only at the top level of a `SELECT` output list, `RETURNING` list, `VALUES` clause, or row constructor.

**Constraints and limitations:**

- Without parentheses, `item.name` is parsed as `table.column`, which is a common source of syntax errors.
- Updating a subfield rewrites the entire composite value under MVCC, which can be expensive for large composites.
- Column expansion with `.*` may invoke a function multiple times per row unless a `LATERAL` `FROM` item is used.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Constructing, accessing, and updating composite values

```sql
-- Step 1: Create a table with a composite column.
CREATE TABLE mytab (
    id          int PRIMARY KEY,
    complex_col complex
);

-- Step 2: Insert a whole composite value using ROW.
INSERT INTO mytab (id, complex_col) VALUES (1, ROW(1.1, 2.2));

-- Step 3: Access a field with parentheses.
SELECT (complex_col).r AS real_part FROM mytab WHERE id = 1;

-- Step 4: Update a single subfield.
UPDATE mytab SET complex_col.r = (complex_col).r + 1 WHERE id = 1;

-- Step 5: Verify the update.
SELECT (complex_col).r AS updated_real, (complex_col).i AS imag FROM mytab WHERE id = 1;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
 real_part
-----------
       1.1
(1 row)

UPDATE 1
 updated_real | imag
--------------+------
          2.1 |  2.2
(1 row)
```

**Why this output appears**: The `ROW(1.1, 2.2)` expression constructs a `complex` value. `(complex_col).r` extracts the `r` field (parentheses are required). The `UPDATE` adds 1 to the `r` field while leaving `i` unchanged. The final query confirms that `r` is now 2.1 and `i` remains 2.2.

##### Example 2: Using functional notation for computed fields

```sql
-- Step 1: Create a function that takes a composite argument.
CREATE FUNCTION price_extension(item inventory_item, quantity integer)
RETURNS numeric AS $$
    SELECT (item).price * quantity;
$$ LANGUAGE SQL;

-- Step 2: Call the function with functional notation and with field access.
SELECT price_extension(item, 10) AS functional_call,
       item.price_extension(10) AS field_style_call
FROM on_hand;
```

**Expected output:**

```text
 functional_call | field_style_call
-----------------+------------------
           19.90 |            19.90
(1 row)
```

**Why this output appears**: Because PostgreSQL treats `func(composite)` and `composite.func` as equivalent for functions that accept a single composite argument, both call forms produce the same result. This equivalence enables “computed fields” — an application can use `item.price_extension(10)` without knowing that `price_extension` is a function rather than a stored field.

#### Real-World Cases

- **Filtering on composite fields**: A `shipments` table stores a `destination` composite; queries use `(destination).country = 'US'` to filter by a field inside the composite.
- **Partial updates**: A `profiles` table stores an `address` composite; an application updates only the `zip` field with `SET address.zip = '10001'` without rewriting the whole address.
- **Computed fields**: A `products` table stores a `dimensions` composite; a function `volume(dimensions)` is called as `dimensions.volume` to compute the volume as if it were a stored field.

---

### 3. Composite Best Practices

#### Definitions

**Core Definition**
Composite best practices are the guidelines for deciding when to use composite types, how to structure them, how to avoid their performance pitfalls, and how to work around their limitations compared with normalized tables and JSONB.

**Technical Definition**
Best practices include: (1) **prefer normalization for most data** — composite types embedded in rows create wide rows that slow table scans, and updating a subfield rewrites the entire composite under MVCC; (2) **use composite types for reusable tuple shapes** when it is logically appropriate to model something as a “type,” such as a `rate_t` type used with `EXCLUDE` constraints or a `tenant_ctx` type used in row-level security; (3) **be aware of the storage and index penalty** — composite types are “more than the sum of their parts” performance-wise, incurring storage overhead in both the heap and indexes, and indexes cannot be leveraged when querying only part of a composite key; (4) **use domains to attach constraints** — since composite types cannot carry constraints, create a domain over the composite type and add `CHECK` constraints to enforce rules; (5) **avoid giving functions the same name as composite fields** — when a function accepting a single composite argument shares a name with a field, the field-name interpretation wins in field syntax, which can cause silent ambiguity; and (6) **use `LATERAL` to avoid repeated function calls** when expanding `.*` from an expensive function.

**Beginner-Friendly Explanation**
Composite types are a specialised tool, not a default choice. Most of the time, normalizing your data into separate tables is safer and faster. Use composite types when you genuinely have a small, reusable “value object” — like a coordinate pair or a rate range — that appears in many places and is always handled as a unit. Remember that composite types cannot enforce rules, so if you need constraints, wrap the composite type in a domain. And be careful with names: if a function and a field share a name, you might call the wrong one.

#### Purposes

- To avoid the performance and tooling pitfalls of overusing composite types.
- To attach constraints to composite types via domains.
- To choose the right representation (composite, normalized table, or JSONB) for each data modelling problem.
- To write queries that avoid unnecessary rewrites and repeated function calls.

#### Syntax Rules and Structure

**Domain over a composite type:**

```sql
CREATE DOMAIN positive_complex AS complex
    CHECK ((VALUE).r > 0 AND (VALUE).i > 0);
```

**LATERAL expansion to avoid repeated calls:**

```sql
SELECT m.*
FROM some_table, LATERAL myfunc(x) AS m;
```

**Best-practice decision guide:**

| Scenario | Recommended approach |
|----------|----------------------|
| Reusable tuple shape used in many places (coordinates, rates, tenant context) | Composite type |
| Data that is frequently filtered, joined, or aggregated on individual fields | Normalized child table |
| Flexible, schema-less attributes | `JSONB` |
| Need constraints on a composite-like value | Domain over composite type |

**Syntax rules:**

- Domains over composite types can carry `CHECK` constraints, `NOT NULL`, and `DEFAULT`.
- `LATERAL` in a `FROM` clause ensures a function is evaluated once per row when its output is expanded.

**Constraints and limitations:**

- Composite types cannot have constraints directly; domains are the workaround.
- Client drivers and ORMs may not support composite types well, so applications may need to expand them into tuples.
- Updating a subfield of a composite rewrites the whole tuple, which can be expensive for large composites.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Attaching constraints via a domain

```sql
-- Step 1: Create a composite type.
CREATE TYPE coordinate AS (x double precision, y double precision);

-- Step 2: Create a domain over the composite type with constraints.
CREATE DOMAIN valid_coordinate AS coordinate
    CHECK ((VALUE).x >= -180 AND (VALUE).x <= 180
       AND (VALUE).y >= -90  AND (VALUE).y <= 90);

-- Step 3: Use the domain in a table.
CREATE TABLE locations (
    id   int PRIMARY KEY,
    pos  valid_coordinate
);

-- Step 4: Insert a valid coordinate.
INSERT INTO locations VALUES (1, ROW(45.0, 90.0));

-- Step 5: Attempt to insert an invalid coordinate (latitude > 90).
INSERT INTO locations VALUES (2, ROW(100.0, 0.0));
```

**Expected output:**

```text
CREATE TYPE
CREATE DOMAIN
CREATE TABLE
INSERT 0 1
ERROR:  value for domain valid_coordinate violates check constraint "valid_coordinate_check"
```

**Why this output appears**: The domain `valid_coordinate` adds `CHECK` constraints to the `coordinate` composite type, enforcing latitude and longitude bounds. The first insert satisfies the constraint; the second violates it because `100.0` exceeds the maximum latitude of 90, so PostgreSQL rejects it.

##### Example 2: Using LATERAL to avoid repeated function calls

```sql
-- Step 1: Define a function that returns a composite type.
CREATE FUNCTION get_stats() RETURNS TABLE (a int, b int) AS $$
    SELECT 1, 2;
$$ LANGUAGE SQL;

-- Step 2: Expand the result with LATERAL (function called once per row).
SELECT m.*
FROM generate_series(1, 3) AS g, LATERAL get_stats() AS m;
```

**Expected output:**

```text
 a | b
---+---
 1 | 2
 1 | 2
 1 | 2
(3 rows)
```

**Why this output appears**: Placing `get_stats()` in a `LATERAL` `FROM` item ensures it is evaluated once per row of `generate_series`, rather than once per output column. The `m.*` expansion then references the already-computed result, avoiding repeated calls to the function.

#### Real-World Cases

- **Row-level security contexts**: A `tenant_ctx` composite type (`org_id int, unit_id int, user_id int`) is used to pass tenant identity through function calls and row-level security policies, keeping the context bundled as a single value.
- **Rate ranges with exclusion constraints**: A `rate_t` composite type (`price numeric, validity daterange`) is used with a table `OF rate_t` and an `EXCLUDE` constraint to prevent overlapping price periods, leveraging the composite type for reusable tuple shape and constraint support via the table.
- **When normalization wins**: A `orders` table with `order_lines` is better normalized into two tables than stored as an array or composite of line items, because line items are frequently filtered, joined to products, and aggregated independently.

---

## References

- PostgreSQL Documentation – Composite Types – https://www.postgresql.org/docs/14/rowtypes.html 
- PostgreSQL Documentation – Composite Types (PostgreSQL 13) – https://www.postgresql.org/docs/13/rowtypes.html 
- PostgreSQL Documentation – Composite Types (current) – https://www.postgresql.org/docs/current/rowtypes.html
- PostgreSQL Documentation – `CREATE TYPE` – https://www.postgresql.org/docs/devel/sql-createtype.html
- PostgreSQL Mailing List – Important topic is missed: comparing composite types – https://www.postgresql.org/message-id/flat/175816236005.860.4057702842618093971@wrigleys.postgresql.org 
- PostgreSQL Mailing List – Composite types or composite keys? – https://www.postgresql.org/message-id/flat/… 
- Stack Overflow – When to use composite types and arrays and when to normalize a database? – https://stackoverflow.com/questions/21629989/when-to-use-composite-types-and-arrays-and-when-to-normalize-a-database 
- The Art of PostgreSQL, Vol. 1 – Ch 23: Arrays, Composite Types, Enums – https://github.com/BrewHubPHL/supabase-specialist/blob/master/references/book-summaries/taop-vol1-ch23-arrays-composite-enum.md 