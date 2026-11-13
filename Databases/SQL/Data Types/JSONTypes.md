# Comprehensive Programming Cheat Sheet: SQL JSON Types

## Topic Overview

### Definitions

**Core Definition**
JSON types are the PostgreSQL data types—`JSON` and `JSONB`—used to store JSON (JavaScript Object Notation) documents directly in a database column, enabling schema-flexible data storage alongside traditional relational columns.

**Technical Definition**
PostgreSQL offers two types for storing JSON data: `json` and `jsonb`. Both accept almost identical sets of values as input, with the major practical difference being efficiency: `json` stores an exact copy of the input text and must reparse it on each execution, while `jsonb` stores a decomposed binary format that is slightly slower to input but significantly faster to process and supports indexing. To implement efficient query mechanisms, PostgreSQL also provides the `jsonpath` data type. The SQL/JSON functions and operators operate on both types, but some operators and functions are available only for `jsonb`, and comparison operators follow B-tree ordering rules that apply to `jsonb` but not to `json`.

**Beginner-Friendly Explanation**
PostgreSQL gives you two containers for JSON: `JSON` and `JSONB`. `JSON` is like a photocopier—it keeps an exact copy of what you put in, including spaces and key order, but it has to re-read the whole document every time you query it. `JSONB` is like a filing system—it breaks the document into a searchable binary form, which takes a moment longer to store but makes queries much faster. Think of `JSON` as a sealed envelope and `JSONB` as an indexed binder. For most applications, `JSONB` is the better choice.

### Key Characteristics

- **`JSON` preserves input text exactly**: `JSON` stores an exact copy of the input text, preserving semantically insignificant whitespace, key order, and duplicate keys (keeping all key/value pairs, with the last value treated as operative).
- **`JSONB` normalises and indexes**: `JSONB` does not preserve whitespace, does not preserve key order, and does not keep duplicate object keys (only the last value is kept); in exchange, it supports GIN indexing and is significantly faster to process.
- **`JSONB` is the recommended default**: PostgreSQL documentation states that most applications should prefer to store JSON data as `jsonb`, unless there are specialised needs such as legacy assumptions about key ordering.
- **Indexing is the key performance differentiator**: `JSONB` supports GIN indexes for efficient searches on keys and key/value pairs; `JSON` does not support indexing.
- **Unicode handling differs**: `JSON` accepts Unicode escapes without checking representability, while `JSONB` is stricter, rejecting `\u0000` and requiring correct surrogate pairs.

### Prerequisites

- A running PostgreSQL instance (version 9.4+ for `JSONB`; most functions are available in modern versions).
- Familiarity with SQL `CREATE TABLE` and basic JSON syntax.
- Understanding of when schema-flexible JSON storage is appropriate versus normalised relational columns.
- Awareness of GIN indexing and query operators (`@>`, `?`, `?&`, `?|`) for `JSONB` performance.

### Related Programming Areas

- Data Definition Language (DDL) – column type declarations (`json`, `jsonb`).
- Data Manipulation Language (DML) – inserting, querying, and updating JSON documents.
- SQL/JSON path language – `jsonpath` expressions for complex queries.
- Indexing and query performance – GIN and B-tree expression indexes.
- Application development – JSON serialisation/deserialisation, API payload storage, and schema evolution.

---

## Core Concepts and Features

### 1. `JSON`

#### Definitions

**Core Definition**
`JSON` is a PostgreSQL data type that stores an exact textual copy of a JSON document, preserving whitespace, key order, and duplicate keys.

**Technical Definition**
The `json` data type stores an exact copy of the input text, which processing functions must reparse on each execution. It preserves semantically insignificant whitespace between tokens, the order of keys within JSON objects, and all key/value pairs when an object contains the same key more than once (processing functions consider the last value as operative). `json` does not support indexing and comparison operators are not available for it, unlike `jsonb`.

**Beginner-Friendly Explanation**
`JSON` is the “exact copy” type. It stores your JSON text verbatim, spaces and all. If you need to preserve the exact formatting or key order of a document—for example, for display or for systems that depend on key ordering—`JSON` is the right choice. The trade-off is that it is slower to query and cannot be indexed.

#### Purposes

- To store JSON documents where exact textual representation, whitespace, and key order must be preserved.
- To store JSON as an opaque payload for later retrieval without in-database querying.
- To support legacy applications that depend on key ordering or duplicate keys.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name JSON
```

**Inserting JSON:**

```sql
INSERT INTO docs (data) VALUES ('{"name": "Alice", "age": 30}');
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `JSON` | Stores an exact copy of the input JSON text. |
| Reparsing | Required on each function execution; slower queries. |
| Indexing | Not supported directly. |

**Syntax rules:**

- Input must be valid JSON per RFC 7159.
- `JSON` preserves whitespace and key order.
- Duplicate keys are all retained, though the last value is operative in processing functions.

**Constraints and limitations:**

- No GIN indexing support, so queries on `JSON` columns perform full scans.
- No comparison operators; `json` values cannot be sorted or compared directly.
- Reparsing overhead makes repeated queries on the same document slower than with `JSONB`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Preserving key order and whitespace with JSON

```sql
-- Step 1: Create a table with a JSON column.
CREATE TABLE json_docs (id int, doc json);

-- Step 2: Insert a document with specific key order and whitespace.
INSERT INTO json_docs VALUES (1, '{ "name" : "Alice" , "age" : 30 }');

-- Step 3: Query the document; the exact text is preserved.
SELECT doc FROM json_docs;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
              doc
-------------------------------
 { "name" : "Alice" , "age" : 30 }
(1 row)
```

**Why this output appears**: The `json` type stores the exact input text, including the extra spaces around the colon and the key order `name` then `age`. No parsing or reformatting occurs on storage.

#### Real-World Cases

- **Document archives**: Storing JSON payloads exactly as received for audit or compliance purposes, where the exact bytes matter.
- **Legacy integration**: Systems that depend on key ordering or duplicate keys in JSON documents.

---

### 2. `JSONB`

#### Definitions

**Core Definition**
`JSONB` is a PostgreSQL data type that stores JSON data in a decomposed binary format, enabling efficient processing, indexing, and comparison at the cost of not preserving whitespace or key order.

**Technical Definition**
`jsonb` stores data in a decomposed binary format that is slightly slower to input due to conversion overhead but significantly faster to process because no reparsing is needed. It does not preserve whitespace, does not preserve key order, and does not keep duplicate object keys (only the last value is kept). `jsonb` supports GIN indexing, which can be a significant advantage, and the usual comparison operators are available for `jsonb` but not for `json`.

**Beginner-Friendly Explanation**
`JSONB` is the “searchable” type. It parses your JSON into a binary format, dropping spaces and reordering keys, but gaining the ability to be indexed and queried quickly. This is what you want for most applications that need to store and query JSON data.

#### Purposes

- To store JSON documents that need to be queried, filtered, and indexed efficiently.
- To support GIN indexes on keys and key/value pairs for fast lookups.
- To enable comparison, sorting, and unique constraints on JSON values.
- To serve as the default JSON type for most applications.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name JSONB
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `JSONB` | Decomposed binary format; faster processing. |
| GIN indexing | Supported for `@>`, `?`, `?&`, `?|` operators. |
| Key order | Not preserved. |
| Duplicate keys | Only the last value is kept. |

**Syntax rules:**

- `jsonb` rejects `\u0000` and requires correct surrogate pairs.
- Comparison operators (`=`, `<`, `>`) and B-tree ordering are available.
- GIN indexes can be created on `jsonb` columns.

**Constraints and limitations:**

- Input conversion overhead makes writes slightly slower than `json`.
- Whitespace, key order, and duplicate keys are lost.
- GIN indexes support only specific operators; expression indexes may be needed for nested paths.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – JSONB normalisation and indexing

```sql
-- Step 1: Create a table with a JSONB column and a GIN index.
CREATE TABLE api_docs (id int, jdoc jsonb);
CREATE INDEX idxgin ON api_docs USING GIN (jdoc);

-- Step 2: Insert a document.
INSERT INTO api_docs VALUES (1, '{"name": "Alice", "tags": ["admin", "user"]}');

-- Step 3: Query using containment.
SELECT id FROM api_docs WHERE jdoc @> '{"tags": ["admin"]}';
```

**Expected output:**

```text
CREATE TABLE
CREATE INDEX
INSERT 0 1
 id
----
  1
(1 row)
```

**Why this output appears**: The GIN index on `jdoc` accelerates the `@>` containment query, allowing the database to quickly find documents whose `tags` array contains `"admin"`. The stored `jsonb` value has been normalised, so key order and whitespace are not preserved.

#### Real-World Cases

- **API payload storage**: Storing JSON API responses in a `jsonb` column for later querying and analysis.
- **Event metadata**: Storing flexible event properties in `jsonb` with a GIN index for fast filtering.

---

### 3. JSON vs JSONB

#### Definitions

**Core Definition**
`JSON` and `JSONB` are two PostgreSQL types for storing JSON data, differing in storage format, processing speed, indexing support, and preservation of input formatting.

**Technical Definition**
The core distinction is efficiency: `json` stores exact text and reparses on each execution, while `jsonb` stores a decomposed binary format that is slower to input but faster to process and indexable. `json` preserves whitespace, key order, and duplicate keys; `jsonb` does not. `jsonb` supports GIN indexing and comparison operators, while `json` does not.

**Beginner-Friendly Explanation**
`JSON` is for exact copies and archival; `JSONB` is for querying and speed. If you need to search inside your JSON, use `JSONB`. If you need to preserve the exact text, use `JSON`. Most of the time, `JSONB` is the right choice.

#### Purposes

- To choose the appropriate type based on whether query performance or exact textual preservation is more important.
- To understand the trade-offs before designing a schema that relies on JSON storage.

#### Syntax Rules and Structure

| Aspect | `JSON` | `JSONB` |
|--------|--------|---------|
| Storage | Exact text | Decomposed binary |
| Whitespace | Preserved | Not preserved |
| Key order | Preserved | Not preserved |
| Duplicate keys | All kept | Last kept |
| Indexing | No | GIN, B-tree (via expression) |
| Comparison operators | No | Yes |
| Input speed | Faster | Slower |
| Query speed | Slower | Faster |

**Syntax rules:**

- Both types accept the same JSON input values.
- Use `json` when the exact input text must be preserved.
- Use `jsonb` when the data will be queried, indexed, or compared.

**Constraints and limitations:**

- `json` cannot be indexed; queries on `json` columns are full scans.
- `jsonb` loses formatting and duplicate keys on input.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Comparing JSON and JSONB behaviour

```sql
-- Step 1: Insert the same input into both types.
CREATE TABLE comparison (id int, j json, jb jsonb);
INSERT INTO comparison VALUES (1, '{"b": 2, "a": 1}', '{"b": 2, "a": 1}');

-- Step 2: Observe the difference in stored representation.
SELECT j, jb FROM comparison;
```

**Expected output:**

```text
              j               |        jb
------------------------------+------------------
 {"b": 2, "a": 1}             | {"a": 1, "b": 2}
(1 row)
```

**Why this output appears**: `json` preserves the original key order (`b` before `a`), while `jsonb` normalises the keys into a canonical order (`a` before `b`). The whitespace difference is also visible.

#### Real-World Cases

- **Analytics**: Use `jsonb` when you need to filter and aggregate on JSON properties.
- **Document archiving**: Use `json` when the exact bytes of a JSON document must be preserved for legal or audit reasons.

---

### 4. JSON Operations

#### Definitions

**Core Definition**
JSON operations are the operators and functions that extract, compare, and manipulate JSON data, including `->`, `->>`, `#>`, `#>>`, `@>`, `?`, `?&`, and `?|`.

**Technical Definition**
PostgreSQL provides a set of operators for `json` and `jsonb` values. The `->` operator extracts a JSON object field or array element (returning `json`/`jsonb`), while `->>` extracts the same but returns `text`. The `#>` and `#>>` operators extract a sub-object at a specified path (as `json`/`jsonb` or `text`, respectively). The `@>` operator tests containment, and `?`, `?&`, `?|` test key/element existence. Comparison operators are available for `jsonb` but not `json`.

**Beginner-Friendly Explanation**
JSON operations are the tools you use to dig into a JSON document. `->` gets a piece and keeps it as JSON; `->>` gets a piece and turns it into text. `#>` follows a path. `@>` checks whether one document contains another. `?` checks whether a key exists.

#### Purposes

- To extract specific fields or elements from JSON documents.
- To filter rows based on JSON content (containment, existence).
- To compare and sort `jsonb` values.
- To navigate nested JSON structures using path expressions.

#### Syntax Rules and Structure

| Operator | Returns | Description |
|----------|---------|-------------|
| `->` | `json`/`jsonb` | Extract array element or object field |
| `->>` | `text` | Extract as text |
| `#>` | `json`/`jsonb` | Extract at path |
| `#>>` | `text` | Extract at path as text |
| `@>` | `boolean` | Containment |
| `?` | `boolean` | Key/element exists |
| `?&` | `boolean` | All keys exist |
| `?|` | `boolean` | Any key exists |

**Syntax rules:**

- `->` and `->>` accept an integer (array index) or text (object key).
- `#>` and `#>>` accept a text array representing the path.
- `@>` tests whether the left JSON value contains the right JSON value.
- `?` and related operators are available for `jsonb` and support GIN indexing.

**Constraints and limitations:**

- `json` does not support comparison operators.
- Some operators (`?`, `?&`, `?|`) are not available for `json` in older versions.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Extraction and containment operators

```sql
-- Step 1: Create a table with JSONB data.
CREATE TABLE events (id int, data jsonb);
INSERT INTO events VALUES
    (1, '{"type": "click", "user": {"name": "Alice"}}'),
    (2, '{"type": "view", "user": {"name": "Bob"}}');

-- Step 2: Extract a nested field as text.
SELECT data->>'type' AS event_type FROM events WHERE id = 1;

-- Step 3: Filter using containment.
SELECT id FROM events WHERE data @> '{"user": {"name": "Alice"}}';
```

**Expected output:**

```text
 event_type
------------
 click
(1 row)
 id
----
  1
(1 row)
```

**Why this output appears**: `data->>'type'` extracts the `type` field as text (`click`). The `@>` operator checks whether the `user` object contains `{"name": "Alice"}`, matching only row 1.

#### Real-World Cases

- **Event filtering**: Querying JSON event data for specific types or user attributes.
- **API query parameters**: Extracting nested values from JSON request payloads stored in the database.

---

### 5. JSON Functions

#### Definitions

**Core Definition**
JSON functions are the built-in PostgreSQL functions for constructing, processing, and querying JSON data, including builder functions, aggregate functions, mutation functions, and path-query functions.

**Technical Definition**
PostgreSQL provides a rich set of JSON functions organised into several categories: creation functions (`json_build_object`, `jsonb_build_object`, `json_build_array`, `jsonb_build_array`, `to_json`, `to_jsonb`), aggregate functions (`json_agg`, `jsonb_agg`, `json_object_agg`, `jsonb_object_agg`), processing functions (`jsonb_set`, `jsonb_insert`, `jsonb_pretty`, `jsonb_strip_nulls`), and path-query functions (`jsonb_path_query`, `jsonb_path_exists`, `jsonb_path_match`). The `RETURNING` clause on some functions specifies the returned data type (default `json`).

**Beginner-Friendly Explanation**
JSON functions are the power tools for working with JSON. Builder functions create JSON from SQL data; aggregate functions roll up multiple rows into a JSON array or object; mutation functions change parts of a JSON document; and path functions query JSON using a path language similar to XPath.

#### Purposes

- To construct JSON objects and arrays from relational data.
- To aggregate rows into JSON structures for API responses or reports.
- To modify specific parts of a JSON document without rewriting the whole value.
- To query JSON documents using powerful path expressions.

#### Syntax Rules and Structure

| Category | Functions |
|----------|-----------|
| Creation | `json_build_object()`, `jsonb_build_object()`, `json_build_array()`, `jsonb_build_array()`, `to_json()`, `to_jsonb()` |
| Aggregate | `json_agg()`, `jsonb_agg()`, `json_object_agg()`, `jsonb_object_agg()` |
| Mutation | `jsonb_set()`, `jsonb_insert()`, `jsonb_strip_nulls()` |
| Formatting | `jsonb_pretty()` |
| Path query | `jsonb_path_query()`, `jsonb_path_exists()`, `jsonb_path_match()` |

**Syntax rules:**

- Builder functions accept a variadic list of alternating keys and values (for objects) or a list of values (for arrays).
- Aggregate functions operate on sets of rows, like `SUM` or `AVG`.
- Mutation functions take a target `jsonb`, a path, and a new value.

**Constraints and limitations:**

- `jsonb_set` and `jsonb_insert` only work on `jsonb`, not `json`.
- Aggregate functions may return `json` or `jsonb` depending on the variant.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Building a JSON object from columns

```sql
-- Step 1: Create a table with relational columns.
CREATE TABLE users (id int, name text, email text);
INSERT INTO users VALUES (1, 'Alice', 'alice@example.com');

-- Step 2: Build a JSON object from the columns.
SELECT jsonb_build_object('id', id, 'name', name, 'email', email) AS user_json
FROM users;
```

**Expected output:**

```text
                                user_json
---------------------------------------------------------------------------
 {"id": 1, "name": "Alice", "email": "alice@example.com"}
(1 row)
```

**Why this output appears**: `jsonb_build_object` pairs the provided keys with the corresponding column values, producing a single `jsonb` object per row.

#### Real-World Cases

- **API responses**: Building JSON objects and arrays from relational data for REST or GraphQL responses.
- **Data export**: Aggregating related rows into JSON documents for export to external systems.

---

### 6. `json_build_object()` / `jsonb_build_object()`

#### Definitions

**Core Definition**
`json_build_object()` and `jsonb_build_object()` build a JSON object from a variadic list that specifies keys with values of arbitrary SQL data type.

**Technical Definition**
These functions accept an alternating list of keys and values and construct a JSON object. The `json_build_object()` variant returns `json`, while `jsonb_build_object()` returns `jsonb`. Each value is converted as per `to_json` or `to_jsonb`. For write-heavy or indexed workloads, `jsonb_build_object()` is preferred because it returns the more efficient `jsonb` type.

**Beginner-Friendly Explanation**
These functions are like a factory that takes pairs of keys and values and assembles them into a JSON object. The `jsonb` version is usually the better choice because it produces the faster, indexable type.

#### Purposes

- To construct JSON objects from relational columns for API responses or denormalised storage.
- To build nested JSON structures by combining multiple builder calls.
- To produce `jsonb` output that can be stored in `jsonb` columns and indexed.

#### Syntax Rules and Structure

**General syntax:**

```sql
json_build_object(key1, value1, key2, value2, ...)
jsonb_build_object(key1, value1, key2, value2, ...)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `key` | A text expression for the object key. |
| `value` | Any SQL expression for the value. |
| Return type | `json` or `jsonb` depending on the function. |

**Syntax rules:**

- Keys and values must alternate.
- The number of arguments must be even.
- Values are converted to JSON using `to_json` or `to_jsonb`.

**Constraints and limitations:**

- An odd number of arguments raises an error.
- Very large numbers of arguments may hit function argument limits.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Building a nested JSON object

```sql
-- Step 1: Build a nested JSON object from SQL data.
SELECT jsonb_build_object(
    'user', jsonb_build_object('name', 'Alice', 'age', 30),
    'active', true
) AS profile;
```

**Expected output:**

```text
                     profile
--------------------------------------------------
 {"user": {"name": "Alice", "age": 30}, "active": true}
(1 row)
```

**Why this output appears**: The outer `jsonb_build_object` creates an object with keys `user` and `active`. The `user` key's value is itself the result of an inner `jsonb_build_object`, producing a nested object.

#### Real-World Cases

- **User profiles**: Building nested profile objects from relational user and settings tables.
- **API payloads**: Constructing JSON responses with structured nested data.

---

### 7. `json_agg()` / `jsonb_agg()`

#### Definitions

**Core Definition**
`json_agg()` and `jsonb_agg()` are aggregate functions that collect all input values, including nulls, into a JSON array.

**Technical Definition**
These functions aggregate record values into a JSON array. `json_agg()` returns `json`, while `jsonb_agg()` returns `jsonb`. They are the JSON counterparts of standard SQL aggregate functions like `array_agg`, and are documented as aggregate functions in Section 9.22 of the PostgreSQL documentation.

**Beginner-Friendly Explanation**
These functions take many rows and turn them into a JSON array. For example, if you have a table of orders and you want a single JSON array containing all the order IDs, `jsonb_agg` does that.

#### Purposes

- To aggregate rows into a JSON array for API responses or nested documents.
- To collect a set of values into a single JSON structure.
- To support denormalisation by embedding related rows as arrays.

#### Syntax Rules and Structure

**General syntax:**

```sql
json_agg(expression)
jsonb_agg(expression)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `expression` | The value to aggregate (a column, expression, or composite row). |
| Return type | `json` or `jsonb` array. |

**Syntax rules:**

- `json_agg` and `jsonb_agg` are aggregate functions and are used with `GROUP BY` like other aggregates.
- `ORDER BY` can be added inside the aggregate to control element order.

**Constraints and limitations:**

- The result is always a JSON array, even for a single row.
- `json` variant does not support indexing; use `jsonb` for indexed workflows.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Aggregating rows into a JSON array

```sql
-- Step 1: Create a table with multiple rows.
CREATE TABLE products (id int, name text);
INSERT INTO products VALUES (1, 'Widget'), (2, 'Gadget'), (3, 'Doohickey');

-- Step 2: Aggregate the names into a JSONB array.
SELECT jsonb_agg(name ORDER BY id) AS product_names FROM products;
```

**Expected output:**

```text
                product_names
----------------------------------------------
 ["Widget", "Gadget", "Doohickey"]
(1 row)
```

**Why this output appears**: `jsonb_agg(name ORDER BY id)` collects all `name` values in `id` order into a single `jsonb` array, producing the ordered list of product names.

#### Real-World Cases

- **Order line items**: Aggregating line items into a JSON array for an order document.
- **Tag lists**: Collecting tags into a JSON array for each post or product.

---

### 8. `json_object_agg()` / `jsonb_object_agg()`

#### Definitions

**Core Definition**
`json_object_agg()` and `jsonb_object_agg()` are aggregate functions that aggregate pairs of values into a JSON object.

**Technical Definition**
These functions take a key expression and a value expression and produce a JSON object where each key maps to the corresponding value. `json_object_agg()` returns `json`, while `jsonb_object_agg()` returns `jsonb`. They are documented alongside `json_agg` and `jsonb_agg` in the aggregate functions section.

**Beginner-Friendly Explanation**
These functions build a JSON object from many rows. Each row contributes one key-value pair. For example, you could turn a table of user settings (setting name, setting value) into a single JSON object.

#### Purposes

- To aggregate key-value pairs into a single JSON object.
- To build configuration maps or lookup dictionaries from relational data.
- To support denormalised JSON object storage for fast retrieval.

#### Syntax Rules and Structure

**General syntax:**

```sql
json_object_agg(key_expression, value_expression)
jsonb_object_agg(key_expression, value_expression)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `key_expression` | The expression that produces each key (must be text or castable to text). |
| `value_expression` | The expression that produces each value. |
| Return type | `json` or `jsonb` object. |

**Syntax rules:**

- Key values must not be null.
- Duplicate keys result in the last value winning.
- `ORDER BY` can be added to control key order in the resulting object.

**Constraints and limitations:**

- `json` variant does not support indexing; use `jsonb` for indexed use cases.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Building a settings object from key-value rows

```sql
-- Step 1: Create a table of settings.
CREATE TABLE settings (key text, value text);
INSERT INTO settings VALUES ('theme', 'dark'), ('language', 'en'), ('timezone', 'UTC');

-- Step 2: Aggregate into a JSONB object.
SELECT jsonb_object_agg(key, value) AS settings_json FROM settings;
```

**Expected output:**

```text
                                 settings_json
--------------------------------------------------------------------------------
 {"theme": "dark", "language": "en", "timezone": "UTC"}
(1 row)
```

**Why this output appears**: `jsonb_object_agg(key, value)` pairs each `key` with its `value` and produces a single `jsonb` object containing all three settings.

#### Real-World Cases

- **Configuration storage**: Aggregating configuration key-value rows into a single JSON object for an API response.
- **Lookup maps**: Building a dictionary of codes to descriptions from a reference table.

---

### 9. `jsonb_set()`

#### Definitions

**Core Definition**
`jsonb_set()` returns the target `jsonb` value with the item designated by a path replaced by a new value, or with the new value added if the item does not exist and `create_if_missing` is true (the default).

**Technical Definition**
The function signature is `jsonb_set(target jsonb, path text[], new_value jsonb [, create_if_missing boolean])`. All items of the `path` parameter except the last must be present in the target. Negative integers in the path count from the end of JSON arrays.

**Beginner-Friendly Explanation**
`jsonb_set` is the “edit a specific field” function. You give it a JSON document, a path to the field you want to change (like `{address, city}`), and the new value. It returns a copy of the document with that field changed.

#### Purposes

- To update a specific field in a JSON document without rewriting the entire value.
- To add a new field to a nested object.
- To support partial updates of JSON-stored configuration or profile data.

#### Syntax Rules and Structure

**General syntax:**

```sql
jsonb_set(target, path, new_value [, create_if_missing])
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `target` | The `jsonb` document to modify. |
| `path` | A `text[]` array of keys and array indexes. |
| `new_value` | The replacement value (must be `jsonb`). |
| `create_if_missing` | Boolean (default `true`); if `false`, the function returns `NULL` when the path does not exist. |

**Syntax rules:**

- All path elements except the last must exist in the target.
- The new value must be cast to `jsonb` (e.g., `'"Bob"'::jsonb` for a string).
- Negative array indexes count from the end.

**Constraints and limitations:**

- Only works on `jsonb`, not `json`.
- If `create_if_missing` is `false` and the path does not exist, the result is `NULL` (the whole document), not an error.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Updating a nested field with jsonb_set

```sql
-- Step 1: Create a table with a JSONB column.
CREATE TABLE profiles (id int, data jsonb);
INSERT INTO profiles VALUES (1, '{"user": {"name": "Alice", "age": 30}}');

-- Step 2: Update the name field.
UPDATE profiles
SET data = jsonb_set(data, '{user, name}', '"Bob"')
WHERE id = 1;

-- Step 3: Verify.
SELECT data FROM profiles WHERE id = 1;
```

**Expected output:**

```text
UPDATE 1
                  data
----------------------------------------
 {"user": {"name": "Bob", "age": 30}}
(1 row)
```

**Why this output appears**: `jsonb_set(data, '{user, name}', '"Bob"')` navigates to the `user` object, replaces the `name` field with the string `"Bob"`, and returns the updated document.

#### Real-World Cases

- **Profile updates**: Updating a single field in a user profile stored as JSON.
- **Configuration changes**: Modifying one setting in a JSON configuration object without touching the rest.

---

### 10. `jsonb_insert()`

#### Definitions

**Core Definition**
`jsonb_insert()` inserts a new value into a `jsonb` document at a specified path, optionally before or after an existing array element.

**Technical Definition**
The function signature is `jsonb_insert(target jsonb, path text[], new_value jsonb [, insert_after boolean])`. It inserts the new value at the specified path. If the path points to an array element, the `insert_after` boolean controls whether the new value is inserted after (default `false`) or before the target element. All path elements except the last must exist in the target.

**Beginner-Friendly Explanation**
`jsonb_insert` adds a new item to a JSON document. If you are adding to an array, you can choose whether the new item goes before or after the specified element.

#### Purposes

- To add a new field to a nested object.
- To insert a new element into an array at a specific position.
- To build up JSON documents incrementally.

#### Syntax Rules and Structure

**General syntax:**

```sql
jsonb_insert(target, path, new_value [, insert_after])
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `target` | The `jsonb` document to modify. |
| `path` | A `text[]` path to the insertion point. |
| `new_value` | The value to insert (must be `jsonb`). |
| `insert_after` | Boolean (default `false`); if `true`, insert after the target array element. |

**Syntax rules:**

- All path elements except the last must exist in the target.
- The new value must be cast to `jsonb`.
- For object insertion, the path's last element is the new key.

**Constraints and limitations:**

- Only works on `jsonb`.
- Inserting into a non-existent array position may require the position to exist.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Inserting a new array element with jsonb_insert

```sql
-- Step 1: Create a table with an array-valued JSONB column.
CREATE TABLE lists (id int, data jsonb);
INSERT INTO lists VALUES (1, '{"items": ["a", "c"]}');

-- Step 2: Insert "b" after "a".
UPDATE lists
SET data = jsonb_insert(data, '{items, 0}', '"b"', true)
WHERE id = 1;

-- Step 3: Verify.
SELECT data FROM lists WHERE id = 1;
```

**Expected output:**

```text
UPDATE 1
            data
----------------------------
 {"items": ["a", "b", "c"]}
(1 row)
```

**Why this output appears**: `jsonb_insert(data, '{items, 0}', '"b"', true)` inserts the string `"b"` after the element at index 0 (`"a"`), producing the array `["a", "b", "c"]`.

#### Real-World Cases

- **Building JSON documents**: Incrementally adding fields to a JSON document as data is processed.
- **Array manipulation**: Inserting elements into a JSON array at a specific position.

---

### 11. `jsonb_pretty()`

#### Definitions

**Core Definition**
`jsonb_pretty()` formats a `jsonb` value into human-readable, indented text for display and debugging purposes.

**Technical Definition**
`jsonb_pretty()` takes a `jsonb` value and returns a `text` representation with indentation and line breaks, making nested structures easier to read. It does not change the underlying data; it only affects the textual display.

**Beginner-Friendly Explanation**
`jsonb_pretty` is the “pretty printer” for JSON. It takes a compact JSON value and formats it with indentation and newlines so you can read it easily.

#### Purposes

- To make JSON output human-readable in query results or logs.
- To debug nested JSON structures by visualising their hierarchy.
- To produce formatted JSON for documentation or reports.

#### Syntax Rules and Structure

**General syntax:**

```sql
jsonb_pretty(jsonb_value)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `jsonb_value` | The value to format. |
| Return type | `text` with indentation. |

**Syntax rules:**

- The input must be `jsonb`; `json` is not accepted.
- The output is text; it cannot be directly stored back into a `jsonb` column without parsing.

**Constraints and limitations:**

- Only for display; not a storage format.
- Very large documents produce very long text output.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Pretty-printing a nested JSONB document

```sql
-- Step 1: Select a nested JSONB value with jsonb_pretty.
SELECT jsonb_pretty('{"user": {"name": "Alice", "roles": ["admin", "user"]}}'::jsonb);
```

**Expected output:**

```text
         jsonb_pretty
-----------------------------
 {                          +
     "user": {              +
         "name": "Alice",   +
         "roles": [         +
             "admin",       +
             "user"         +
         ]                  +
     }                      +
 }
(1 row)
```

**Why this output appears**: `jsonb_pretty` adds indentation and line breaks to make the nested structure readable, with `+` characters at the end of each line indicating line breaks in the psql output.

#### Real-World Cases

- **Debugging**: Inspecting the structure of a complex JSON document during development.
- **Reporting**: Generating human-readable JSON for documentation or email reports.

---

### 12. `jsonb_path_query()`

#### Definitions

**Core Definition**
`jsonb_path_query()` evaluates a `jsonpath` expression against a `jsonb` value and returns all matching items as a set of `jsonb` values.

**Technical Definition**
`jsonb_path_query()` is part of PostgreSQL's SQL/JSON path query functions. It takes a `jsonb` document and a `jsonpath` expression and returns a set of `jsonb` values that match the path. The `jsonpath` language supports filters, predicates, and methods, and the function can be combined with other SQL constructs. Related functions include `jsonb_path_exists()` and `jsonb_path_match()`.

**Beginner-Friendly Explanation**
`jsonb_path_query` is the most powerful way to search JSON. You write a path expression (similar to XPath) that describes what you want, and the function returns everything that matches.

#### Purposes

- To query JSON documents using expressive path expressions.
- To filter and extract nested data based on conditions.
- To support complex JSON analytics and transformations.

#### Syntax Rules and Structure

**General syntax:**

```sql
jsonb_path_query(target jsonb, path jsonpath [, vars jsonb [, silent boolean]])
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `target` | The `jsonb` document to query. |
| `path` | The `jsonpath` expression. |
| `vars` | Optional variables for the path expression. |
| `silent` | If `true`, suppresses errors. |
| Return type | `setof jsonb`. |

**Syntax rules:**

- The `jsonpath` expression uses `$` for the root, `.` for member access, `[*]` for array wildcards, and `?()` for filters.
- The function returns a set of matching items, which can be used in a `FROM` clause or aggregated.

**Constraints and limitations:**

- The `jsonpath` language is powerful but has a learning curve.
- Performance depends on indexing; GIN indexes on the `jsonb` column may help.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Querying JSONB with jsonb_path_query

```sql
-- Step 1: Create a table with JSONB data.
CREATE TABLE products (id int, data jsonb);
INSERT INTO products VALUES
    (1, '{"name": "Widget", "price": 10}'),
    (2, '{"name": "Gadget", "price": 25}');

-- Step 2: Query products with price greater than 15.
SELECT jsonb_path_query(data, '$.price ? (@ > 15)') AS high_price
FROM products;
```

**Expected output:**

```text
 high_price
------------
 25
(1 row)
```

**Why this output appears**: The `jsonpath` expression `$.price ? (@ > 15)` extracts the `price` field and filters for values greater than 15. Only the `Gadget` product's price (25) matches.

#### Real-World Cases

- **Complex JSON filtering**: Finding all records where a nested array contains an element meeting a condition.
- **Data extraction**: Extracting specific nested values from JSON documents for reporting.

---

### 13. JSON Best Practices

#### Definitions

**Core Definition**
JSON best practices are the guidelines for choosing between `json` and `jsonb`, indexing JSON columns, designing JSON schemas, and avoiding common performance and correctness pitfalls.

**Technical Definition**
Best practices include: (1) **prefer `jsonb` for most applications** unless exact textual preservation is required; (2) **use GIN indexes** for containment and existence queries on `jsonb` columns; (3) **use expression indexes** for frequently queried nested paths (e.g., `CREATE INDEX ON api USING GIN ((jdoc -> 'tags'))`); (4) **do not overuse JSON**—store frequently queried scalar values in native columns rather than in JSON, because even with GIN indexes, JSON queries can be slower than native column queries; (5) **validate JSON at the application layer** when structural guarantees are needed, since PostgreSQL only guarantees valid JSON syntax, not schema conformance; and (6) **consider `jsonb_path_ops`** for smaller, faster GIN indexes when only containment queries are needed.

**Beginner-Friendly Explanation**
Best practices for JSON are about using the right tool for the job. Use `jsonb` for querying, add GIN indexes to make searches fast, but do not stuff everything into JSON—keep frequently queried fields in regular columns. Index the specific paths you query often, and remember that JSON is flexible, so your application must still validate the shape of the data.

#### Purposes

- To maximise query performance on JSON data.
- To avoid the pitfalls of storing data in JSON when relational columns would be more efficient.
- To choose the right index type for the workload.
- To balance flexibility with data integrity.

#### Syntax Rules and Structure

**Best-practice patterns:**

```sql
-- Prefer jsonb.
CREATE TABLE api_data (
    id int PRIMARY KEY,
    payload jsonb NOT NULL
);

-- GIN index for containment/existence queries.
CREATE INDEX idx_payload_gin ON api_data USING GIN (payload);

-- Expression index for a frequently queried nested path.
CREATE INDEX idx_payload_tags ON api_data USING GIN ((payload -> 'tags'));

-- Use json only when exact text preservation is required.
CREATE TABLE archives (
    id int PRIMARY KEY,
    raw_doc json NOT NULL
);
```

**Component breakdown:**

| Practice | Implementation |
|----------|----------------|
| Type choice | `jsonb` for querying; `json` for exact preservation |
| Indexing | GIN on `jsonb` column; expression GIN on nested paths |
| Operator class | `jsonb_ops` (default) for `?`, `?&`, `?|`, `@>`; `jsonb_path_ops` for `@>` only |
| Native columns | Store frequently queried scalars in regular columns |

**Syntax rules:**

- GIN indexes support `@>`, `?`, `?&`, `?|` operators.
- `jsonb_path_ops` produces a smaller, faster index but supports only `@>`.
- Expression indexes can index the result of `jsonb -> 'key'`.

**Constraints and limitations:**

- JSON queries, even with GIN indexes, may be slower than native column queries.
- Schema validation is not provided by PostgreSQL; use `CHECK` constraints or application-level validation.
- Large JSON documents increase storage and memory usage.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – GIN index and expression index for JSONB

```sql
-- Step 1: Create a table with a JSONB column.
CREATE TABLE events (id int, data jsonb);
INSERT INTO events VALUES
    (1, '{"type": "click", "tags": ["a", "b"]}'),
    (2, '{"type": "view", "tags": ["b", "c"]}');

-- Step 2: Create a GIN index on the whole column.
CREATE INDEX idx_events_gin ON events USING GIN (data);

-- Step 3: Create an expression index on the tags path.
CREATE INDEX idx_events_tags ON events USING GIN ((data -> 'tags'));

-- Step 4: Query using containment.
SELECT id FROM events WHERE data @> '{"tags": ["a"]}';
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 2
CREATE INDEX
CREATE INDEX
 id
----
  1
(1 row)
```

**Why this output appears**: The GIN index on `data` accelerates the `@>` containment query, quickly finding the event whose `tags` array contains `"a"`. The expression index on `data -> 'tags'` would accelerate queries that specifically filter on the `tags` array.

#### Real-World Cases

- **API gateways**: Storing request/response payloads in `jsonb` with GIN indexes for fast filtering by status or user.
- **Product catalogues**: Storing flexible product attributes in `jsonb` while keeping price and name in native columns for fast sorting and filtering.
- **Event analytics**: Using expression indexes on frequently queried event properties to speed up dashboards.

---

## References

- PostgreSQL Documentation – JSON Types – https://www.postgresql.org/docs/current/datatype-json.html
- PostgreSQL Documentation – JSON Functions and Operators – https://www.postgresql.org/docs/current/functions-json.html
- PostgreSQL Documentation – Aggregate Functions (json_agg, json_object_agg) – https://www.postgresql.org/docs/current/functions-aggregate.html
- PostgreSQL Documentation – jsonb Indexing – https://www.postgresql.org/docs/current/datatype-json.html#JSON-INDEXING
- RFC 7159 – The JavaScript Object Notation (JSON) Data Interchange Format – https://datatracker.ietf.org/doc/html/rfc7159
- PostgreSQL Wiki – JSON – https://wiki.postgresql.org/wiki/JSON