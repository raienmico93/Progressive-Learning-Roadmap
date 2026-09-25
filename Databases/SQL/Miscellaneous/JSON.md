# JSON in SQL: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
JSON (JavaScript Object Notation) in SQL refers to the storage, querying, and manipulation of semi-structured JSON documents within relational database systems, bridging the gap between schema-less flexibility and relational integrity.

**Technical Definition**
JSON-in-SQL support encompasses dedicated data types (e.g., PostgreSQL `jsonb`, MySQL `JSON`, SQL Server 2025 native `json`) and a suite of functions for extraction, construction, modification, aggregation, indexing, and validation of JSON documents stored as column values.

**Beginner-Friendly Explanation**
Imagine a traditional database table where every row must have the exact same columns. JSON in SQL lets you store a "flexible box" in a column—you can put different keys and nested structures in each row, then query into those boxes using special functions.

### Key Characteristics

- **Schema Flexibility**: JSON columns accept varying structures per row, unlike fixed relational columns.
- **Two Storage Models**: Text-preserving (`json`) vs. binary-optimized (`jsonb`) in PostgreSQL .
- **Path-Based Access**: Extraction uses `->` (JSON) and `->>` (text) operators or `JSON_EXTRACT()` functions .
- **Indexing via Expression**: JSON paths are indexed through functional/expression indexes or GIN indexes, not native column indexes .

### Prerequisites

- Basic SQL (SELECT, INSERT, UPDATE)
- Understanding of JSON syntax (objects, arrays, nesting)
- Familiarity with indexing concepts

### Related Programming Areas

- **NoSQL Integration**: Storing document data in relational databases
- **API Response Caching**: Persisting JSON payloads from REST APIs
- **Configuration Management**: Dynamic settings per entity
- **Semi-Structured Data Processing**: Log data, event streams

### Core Concepts

| Concept | Description |
|---------|-------------|
| JSON Data Types | `json` vs `jsonb` (PostgreSQL); `JSON` (MySQL); `NVARCHAR(MAX)` (SQL Server 2016-2022) |
| JSON Document Structure | Objects `{}`, arrays `[]`, primitives (string, number, boolean, null) |
| Extraction Operators | `->` (JSON), `->>` (text), `#>` (path array), `#>>` (text path array) |
| Construction Functions | `JSON_OBJECT()`, `JSON_ARRAY()` |
| Modification Functions | `JSON_SET()`, `JSON_REPLACE()`, `JSON_REMOVE()` |
| Aggregation | `JSON_AGG()`, `JSON_OBJECTAGG()` |
| Indexing | Functional indexes, GIN indexes, JSON search indexes |
| Validation | `JSON_VALID()`, `ISJSON()` |


## 1. JSON Data Types

### Definitions

**Core Definition**
JSON data types define how JSON documents are stored internally, affecting query performance, storage size, and whether indexing is supported.

**Technical Definition**
PostgreSQL offers `json` (text-preserving) and `jsonb` (binary-decomposed). MySQL uses `JSON` (binary with validation on insert). SQL Server 2016-2022 uses `NVARCHAR(MAX)` with JSON functions, introducing native `json` in 2025 .

**Beginner-Friendly Explanation**
Think of `json` as storing a photograph of the JSON text—it looks exactly as you typed it. `jsonb` is like storing a translated, indexed version—faster to search but you lose the original formatting.

### Purposes (All begin with "To")

- **To** store semi-structured data without rigid schema constraints
- **To** balance storage efficiency against query performance
- **To** enable indexing for JSON path queries

### Syntax Rules and Structure

**PostgreSQL**
```sql
-- Text-preserving (no indexing)
CREATE TABLE t_json (data json);

-- Binary-optimized (indexable)
CREATE TABLE t_jsonb (data jsonb);
```

**MySQL**
```sql
CREATE TABLE configs (
  id INT PRIMARY KEY,
  data JSON
);
```

**SQL Server (2016-2022)**
```sql
CREATE TABLE customers (
  id INT PRIMARY KEY,
  data NVARCHAR(MAX)  -- Validated via ISJSON()
);
```

**Component Breakdown**

| Database | Type | Storage | Indexable | Preserves Formatting |
|----------|------|---------|-----------|---------------------|
| PostgreSQL | `json` | Text | No | Yes |
| PostgreSQL | `jsonb` | Binary | Yes | No  |
| MySQL | `JSON` | Binary | Via generated columns | No |
| SQL Server | `NVARCHAR(MAX)` | Text | Via computed columns | Yes |

### Annotated Code Examples

**Example 1: PostgreSQL json vs jsonb Behavior**

```sql
-- Create both types
CREATE TABLE json_demo (
  id int,
  data_json json,
  data_jsonb jsonb
);

-- Insert the same document with extra whitespace
INSERT INTO json_demo VALUES (
  1,
  '{"theme": "dark",   "lang": "en"}',  -- json preserves formatting
  '{"theme": "dark",   "lang": "en"}'   -- jsonb normalizes
);

-- Observe differences
SELECT data_json FROM json_demo;   -- {"theme": "dark",   "lang": "en"}
SELECT data_jsonb FROM json_demo;  -- {"lang": "en", "theme": "dark"}
```

**Expected Output:**
```
data_json:  {"theme": "dark",   "lang": "en"}
data_jsonb: {"lang": "en", "theme": "dark"}
```

**Why This Output Occurs**: `json` stores the exact input text, preserving whitespace and key order. `jsonb` parses into binary format, discarding insignificant whitespace and reordering keys for efficient storage .

### Real-World Cases

- **PostgreSQL**: Use `jsonb` for query-heavy workloads; use `json` only when exact textual representation matters (e.g., audit logs)
- **MySQL**: `JSON` type automatically validates on insert and supports generated columns for indexing 
- **SQL Server**: `NVARCHAR(MAX)` for portability across versions; native `json` for 2025+ if whole-document indexing is needed 

### References

- PostgreSQL Documentation - JSON Types - https://www.postgresql.org/docs/current/datatype-json.html
- MySQL JSON Function Reference - https://dev.mysql.com/doc/refman/8.0/en/json-function-reference.html 
- SQL Server JSON Support Guide - https://jamsql.com/blog/sql-server-json-support-guide/ 


## 2. JSON Document Structure

### Definitions

**Core Definition**
A JSON document is a hierarchical structure composed of objects (key-value pairs), arrays (ordered lists), and primitive values (strings, numbers, booleans, null).

**Technical Definition**
JSON grammar per RFC 7159: a value is an object, array, string, number, boolean, or null. Objects are unordered collections of `"key": value` pairs; arrays are ordered sequences of values .

**Beginner-Friendly Explanation**
JSON is like a nested set of labeled boxes. `{"name": "Alice", "hobbies": ["reading", "coding"]}` has a box labeled "name" containing "Alice" and a box labeled "hobbies" containing a list of two items.

### Purposes (All begin with "To")

- **To** represent hierarchical data naturally
- **To** nest objects within objects for complex structures
- **To** combine scalar and array values in a single document

### Syntax Rules and Structure

**JSON Grammar**
```
document  := value
value     := object | array | string | number | 'true' | 'false' | 'null'
object    := '{' (pair (',' pair)*)? '}'
pair      := string ':' value
array     := '[' (value (',' value)*)? ']'
```

**Component Breakdown**

| Structure | Syntax | Example |
|-----------|--------|---------|
| Object | `{key: value}` | `{"name": "Alice"}` |
| Array | `[value, ...]` | `["a", "b", "c"]` |
| Nested | Object in object | `{"user": {"name": "Bob"}}` |
| Array of objects | Array containing objects | `[{"id": 1}, {"id": 2}]` |

### Annotated Code Examples

**Example 1: Querying Nested JSON Structure (PostgreSQL)**

```sql
-- Create table with nested JSON
CREATE TABLE products (
  id int,
  details jsonb
);

INSERT INTO products VALUES (
  1,
  '{
    "name": "Laptop",
    "specs": {"ram": 16, "storage": 512},
    "tags": ["electronics", "computers"],
    "price": 999.99
  }'
);

-- Access top-level keys
SELECT details->>'name' AS product_name FROM products;

-- Access nested object
SELECT details->'specs'->>'ram' AS ram_gb FROM products;

-- Access array element
SELECT details->'tags'->>0 AS first_tag FROM products;
```

**Expected Output:**
```
product_name | ram_gb | first_tag
Laptop       | 16     | electronics
```

**Why This Output Occurs**: The `->` operator returns JSON, while `->>` returns text. Chaining `details->'specs'->>'ram'` navigates into the nested object, then extracts the `ram` value as text .

### Real-World Cases

- **Product Catalogs**: Storing variable specifications per product category
- **User Profiles**: Nested preferences and settings
- **Event Logs**: Structured event data with variable payloads

### References

- PostgreSQL JSON Types - https://www.postgresql.org/docs/current/datatype-json.html 
- MySQL JSON Path Syntax - https://dev.mysql.com/doc/refman/8.0/en/json.html 


## 3. JSON Extraction

### Definitions

**Core Definition**
JSON extraction retrieves values from a JSON document using path expressions, returning either JSON-typed results or SQL text/scalar values.

**Technical Definition**
Extraction operators include `->` (returns JSON), `->>` (returns text), `#>` (path array returns JSON), `#>>` (path array returns text) in PostgreSQL, and `JSON_EXTRACT()` in MySQL. SQL Server uses `JSON_VALUE()` for scalars and `JSON_QUERY()` for objects/arrays .

**Beginner-Friendly Explanation**
Extraction is like using a map to find a specific item in a nested box. You follow the path (`$.user.address.city`) to get the value at that location.

### Purposes (All begin with "To")

- **To** retrieve specific values from nested JSON structures
- **To** filter rows based on JSON content
- **To** transform JSON data into relational columns

### Syntax Rules and Structure

**PostgreSQL Operators**

| Operator | Returns | Example |
|----------|---------|---------|
| `->` | JSON | `data->'name'` |
| `->>` | Text | `data->>'name'` |
| `#>` | JSON (path array) | `data#>'{a,b}'` |
| `#>>` | Text (path array) | `data#>>'{a,b}'` |
| `@>` | Boolean (containment) | `data @> '{"key": "val"}'` |
| `?` | Boolean (key exists) | `data ? 'key'` |

**MySQL Functions**
```sql
JSON_EXTRACT(data, '$.name')  -- Returns JSON
data->'$.name'                 -- Same as JSON_EXTRACT
data->>'$.name'                -- Unquoted text
```

**SQL Server Functions**
```sql
JSON_VALUE(data, '$.name')   -- Returns scalar
JSON_QUERY(data, '$.address') -- Returns object/array
```

### Annotated Code Examples

**Example 1: Path Expressions Across Databases**

```sql
-- PostgreSQL: Arrow operators
SELECT
  data->>'name' AS name,           -- text
  data->'age' AS age_json,         -- json
  data->'address'->>'city' AS city -- nested text
FROM users;

-- MySQL: Path expressions
SELECT
  JSON_EXTRACT(data, '$.name') AS name,
  data->>'$.age' AS age,
  data->>'$.address.city' AS city
FROM users;

-- SQL Server: Function-based
SELECT
  JSON_VALUE(data, '$.name') AS name,
  JSON_VALUE(data, '$.age') AS age,
  JSON_VALUE(data, '$.address.city') AS city
FROM users;
```

**Expected Output:**
```
name  | age | city
Alice | 30  | NYC
```

**Why This Output Occurs**: Each database extracts the same logical values using its native syntax. PostgreSQL uses arrow operators, MySQL uses `->>` shorthand, SQL Server uses `JSON_VALUE()` for scalars .

### Real-World Cases

- **Filtering**: `WHERE data->>'status' = 'active'`
- **Reporting**: Extracting specific fields into report columns
- **Data Migration**: Transforming JSON into normalized tables

### References

- MySQL JSON Functions Cheat Sheet - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-json-functions-cheat-sheet/README.md 
- SQL Server JSON Support - https://jamsql.com/blog/sql-server-json-support-guide/ 
- AWS DMS Playbook - JSON in SQL Server vs PostgreSQL - https://docs.aws.eu/dms/latest/sql-server-to-aurora-postgresql-migration-playbook/chap-sql-server-aurora-pg.tsql.json.html 


## 4. JSON Construction

### Definitions

**Core Definition**
JSON construction functions create JSON documents or arrays from SQL values, enabling dynamic generation of JSON output.

**Technical Definition**
`JSON_OBJECT(key, value, ...)` builds an object; `JSON_ARRAY(value, ...)` builds an array. Both accept SQL expressions as arguments, automatically converting SQL types to JSON types .

**Beginner-Friendly Explanation**
Construction is the reverse of extraction—instead of pulling values out of JSON, you're building JSON from regular SQL data.

### Purposes (All begin with "To")

- **To** generate JSON output from relational data
- **To** build dynamic JSON documents in queries
- **To** create API responses directly from SQL

### Syntax Rules and Structure

**MySQL / PostgreSQL**
```sql
-- Object
SELECT JSON_OBJECT('name', 'Alice', 'age', 30);
-- {"name": "Alice", "age": 30}

-- Array
SELECT JSON_ARRAY(1, 'two', true, NULL);
-- [1, "two", true, null]
```

**Component Breakdown**

| Function | Arguments | Output |
|----------|-----------|--------|
| `JSON_OBJECT(k1, v1, k2, v2, ...)` | Alternating keys and values | JSON object |
| `JSON_ARRAY(v1, v2, ...)` | Values | JSON array |

### Annotated Code Examples

**Example 1: Building JSON from Table Data**

```sql
-- MySQL/PostgreSQL
SELECT JSON_OBJECT(
  'id', id,
  'name', name,
  'active', active
) AS user_json
FROM users
WHERE id = 1;
```

**Expected Output:**
```
user_json
{"id": 1, "name": "Alice", "active": true}
```

**Why This Output Occurs**: The function pairs each key string with the corresponding column value, producing a JSON object per row .

### Real-World Cases

- **API Responses**: Generating JSON directly from SQL queries
- **Data Export**: Converting relational rows to JSON documents
- **Configuration Generation**: Building JSON configs from parameter tables

### References

- MySQL JSON Function Reference - https://dev.mysql.com/doc/refman/8.0/en/json-function-reference.html 
- MySQL JSON Cheat Sheet - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-json-functions-cheat-sheet/README.md 


## 5. JSON Modification

### Definitions

**Core Definition**
JSON modification functions update JSON documents in place, either inserting new keys, replacing existing values, or removing keys.

**Technical Definition**
`JSON_SET()` inserts or updates (upsert); `JSON_INSERT()` only inserts (ignores existing); `JSON_REPLACE()` only updates (ignores missing); `JSON_REMOVE()` deletes keys or array elements .

**Beginner-Friendly Explanation**
These are the "edit" functions for JSON. `SET` is like "add or change," `INSERT` is "only add if new," `REPLACE` is "only change if exists," and `REMOVE` is "delete."

### Purposes (All begin with "To")

- **To** update JSON documents without rewriting the entire value
- **To** add new fields to existing JSON structures
- **To** delete obsolete keys from JSON documents

### Syntax Rules and Structure

**MySQL / PostgreSQL (via jsonb operators)**

| Function | Behavior | Missing Key | Existing Key |
|----------|----------|-------------|--------------|
| `JSON_SET()` | Insert or update | Inserts | Updates |
| `JSON_INSERT()` | Insert only | Inserts | Ignored |
| `JSON_REPLACE()` | Update only | Ignored | Updates |
| `JSON_REMOVE()` | Delete key | Ignored | Removes |

### Annotated Code Examples

**Example 1: Modification Operations (MySQL)**

```sql
-- Original: {"theme": "dark", "lang": "en"}
UPDATE configs
SET data = JSON_SET(data, '$.theme', 'light', '$.version', 2)
WHERE id = 1;
-- Result: {"theme": "light", "lang": "en", "version": 2}

-- JSON_INSERT: only adds new keys
UPDATE configs
SET data = JSON_INSERT(data, '$.newkey', 'value')
WHERE id = 1;
-- Result: adds "newkey" if not present

-- JSON_REPLACE: only changes existing keys
UPDATE configs
SET data = JSON_REPLACE(data, '$.theme', 'system')
WHERE id = 1;
-- Result: changes "theme" if present

-- JSON_REMOVE: deletes key
UPDATE configs
SET data = JSON_REMOVE(data, '$.lang')
WHERE id = 1;
-- Result: removes "lang" key
```

**Expected Output:**
```
After JSON_SET:    {"theme": "light", "lang": "en", "version": 2}
After JSON_INSERT: {"theme": "light", "lang": "en", "version": 2, "newkey": "value"}
After JSON_REPLACE: {"theme": "system", "lang": "en", "version": 2, "newkey": "value"}
After JSON_REMOVE:  {"theme": "system", "version": 2, "newkey": "value"}
```

**Why This Output Occurs**: Each function has distinct semantics: `SET` upserts, `INSERT` only adds missing keys, `REPLACE` only updates existing keys, `REMOVE` deletes specified paths .

### Real-World Cases

- **Configuration Updates**: Changing specific settings in JSON configs
- **Data Enrichment**: Adding computed fields to JSON documents
- **Cleanup**: Removing deprecated keys from stored JSON

### References

- MySQL JSON Function Reference - https://dev.mysql.com/doc/refman/8.0/en/json-function-reference.html 
- MySQL JSON Cheat Sheet - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-json-functions-cheat-sheet/README.md 


## 6. JSON Aggregation

### Definitions

**Core Definition**
JSON aggregation functions combine multiple rows into a single JSON array or object, enabling hierarchical output from relational data.

**Technical Definition**
`JSON_ARRAYAGG()` aggregates values into a JSON array; `JSON_OBJECTAGG(key, value)` aggregates key-value pairs into a JSON object .

**Beginner-Friendly Explanation**
Aggregation is like taking all the rows in a table and packing them into a single JSON list or dictionary, rather than returning one row per record.

### Purposes (All begin with "To")

- **To** produce nested JSON output from flat relational tables
- **To** generate API responses with arrays of objects
- **To** reduce round trips by returning aggregated JSON

### Syntax Rules and Structure

**MySQL / PostgreSQL**
```sql
-- Array aggregation
SELECT JSON_ARRAYAGG(name) FROM employees WHERE dept_id = 1;
-- ["Alice", "Bob", "Charlie"]

-- Object aggregation
SELECT JSON_OBJECTAGG(id, name) FROM employees WHERE dept_id = 1;
-- {"1": "Alice", "2": "Bob", "3": "Charlie"}
```

**Component Breakdown**

| Function | Input | Output |
|----------|-------|--------|
| `JSON_ARRAYAGG(expr)` | Column values | JSON array |
| `JSON_OBJECTAGG(k, v)` | Key, value pairs | JSON object |

### Annotated Code Examples

**Example 1: Aggregating Employees by Department**

```sql
-- MySQL
SELECT
  dept_id,
  JSON_ARRAYAGG(
    JSON_OBJECT('id', id, 'name', name)
  ) AS employees
FROM employees
GROUP BY dept_id;
```

**Expected Output:**
```
dept_id | employees
1       | [{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}]
2       | [{"id": 3, "name": "Charlie"}]
```

**Why This Output Occurs**: `JSON_ARRAYAGG()` collects the per-row `JSON_OBJECT()` results into an array, grouped by department .

### Real-World Cases

- **API Endpoints**: Returning nested JSON with parent-child relationships
- **Reporting**: Building hierarchical summaries
- **Data Export**: Creating document collections from relational data

### References

- MySQL JSON Cheat Sheet - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-json-functions-cheat-sheet/README.md 


## 7. JSON Indexing

### Definitions

**Core Definition**
JSON indexing accelerates path-based queries by creating index structures on extracted JSON values, since JSON columns cannot be indexed directly in most databases.

**Technical Definition**
PostgreSQL uses GIN indexes (`jsonb_ops` or `jsonb_path_ops`) on `jsonb` columns. MySQL and SQL Server use functional/expression indexes on generated columns that extract JSON paths .

**Beginner-Friendly Explanation**
Since the database can't index a flexible JSON document directly, you create a "shadow column" that extracts the value you query, then index that shadow column. PostgreSQL does this automatically with GIN indexes.

### Purposes (All begin with "To")

- **To** avoid full table scans when filtering on JSON keys
- **To** accelerate `@>` containment queries (PostgreSQL)
- **To** enable efficient equality lookups on JSON-extracted values

### Syntax Rules and Structure

**PostgreSQL GIN Index**
```sql
-- Full GIN index (supports all operators)
CREATE INDEX idx_gin ON api USING GIN (jdoc);

-- Path-specific GIN (faster containment, no key-only queries)
CREATE INDEX idx_gin_path ON api USING GIN (jdoc jsonb_path_ops);
```

**MySQL Generated Column Index**
```sql
ALTER TABLE configs
ADD COLUMN theme VARCHAR(50) AS (JSON_UNQUOTE(data->>'$.theme')) STORED,
ADD INDEX idx_theme (theme);
```

**SQL Server Computed Column Index**
```sql
ALTER TABLE customers
ADD email AS JSON_VALUE(data, '$.email') PERSISTED;

CREATE INDEX idx_email ON customers(email);
```

**Component Breakdown**

| Database | Index Type | Query Supported | Limitation |
|----------|-----------|-----------------|------------|
| PostgreSQL | GIN `jsonb_ops` | All JSONB operators | Larger index |
| PostgreSQL | GIN `jsonb_path_ops` | Containment only | No key-only queries  |
| MySQL | Functional index | Equality on extracted value | Per-path |
| SQL Server | Computed column index | Equality on extracted value | Per-path  |

### Annotated Code Examples

**Example 1: PostgreSQL GIN Index Performance**

```sql
-- Create GIN index on jsonb column
CREATE INDEX idx_products_gin ON products USING GIN (details);

-- This query uses the index
SELECT * FROM products
WHERE details @> '{"category": "electronics"}';

-- Without GIN index: sequential scan
-- With GIN index: index scan
EXPLAIN ANALYZE
SELECT * FROM products
WHERE details @> '{"category": "electronics"}';
```

**Expected Output (EXPLAIN):**
```
Bitmap Heap Scan on products
  Recheck Cond: (details @> '{"category": "electronics"}'::jsonb)
  ->  Bitmap Index Scan on idx_products_gin
        Index Cond: (details @> '{"category": "electronics"}'::jsonb)
```

**Why This Output Occurs**: The GIN index indexes all keys and values in the JSONB document. The `@>` containment operator can use the index to quickly narrow down matching rows .

### Real-World Cases

- **Search Filters**: Finding products by category stored in JSON
- **User Queries**: Filtering users by extracted attributes
- **Event Processing**: Indexing specific event fields for fast lookup

### References

- PostgreSQL GIN Index Documentation - https://www.postgresql.org/docs/current/gin.html 
- SQL Server JSON Indexing - https://jamsql.com/blog/sql-server-json-support-guide/ 
- PostgreSQL jsonpath Type - https://www.postgresql.org/docs/current/datatype-json.html 


## 8. JSON Schema Validation

### Definitions

**Core Definition**
JSON schema validation ensures that stored JSON documents conform to expected structural and type constraints, rejecting malformed data at insert time.

**Technical Definition**
MySQL provides `JSON_VALID()` for syntax checking and `JSON_SCHEMA_VALID()` for structural validation. SQL Server uses `ISJSON()`. PostgreSQL `jsonb` validates on input and rejects invalid JSON automatically .

**Beginner-Friendly Explanation**
Validation is a bouncer at the door of your database—it checks that the JSON you're inserting is well-formed and meets your rules before letting it in.

### Purposes (All begin with "To")

- **To** prevent malformed JSON from entering the database
- **To** enforce structural constraints on semi-structured data
- **To** catch data quality issues early

### Syntax Rules and Structure

**MySQL**
```sql
-- Syntax validation (returns 1 or 0)
SELECT JSON_VALID('{"a":1}');   -- 1
SELECT JSON_VALID('bad json');  -- 0

-- Schema validation
SELECT JSON_SCHEMA_VALID('{"type":"object","required":["name"]}', '{"name":"Alice"}');  -- 1
```

**SQL Server**
```sql
-- Check validity
SELECT ISJSON(data) FROM customers;

-- Check with constraint
ALTER TABLE customers
ADD CONSTRAINT valid_json CHECK (ISJSON(data) = 1);
```

**PostgreSQL**
```sql
-- jsonb automatically validates on insert
INSERT INTO t_jsonb VALUES ('{"valid": true}');   -- OK
INSERT INTO t_jsonb VALUES ('not json');          -- ERROR
```

**Component Breakdown**

| Database | Function | Behavior |
|----------|----------|----------|
| MySQL | `JSON_VALID()` | Syntax check (1/0) |
| MySQL | `JSON_SCHEMA_VALID()` | Schema check |
| SQL Server | `ISJSON()` | Syntax check (1/0) |
| PostgreSQL | `jsonb` type | Auto-validates on insert  |

### Annotated Code Examples

**Example 1: MySQL Validation with CHECK Constraint**

```sql
CREATE TABLE logs (
  id INT PRIMARY KEY,
  data JSON,
  CONSTRAINT valid_data CHECK (JSON_VALID(data))
);

-- This succeeds
INSERT INTO logs VALUES (1, '{"level": "info", "message": "started"}');

-- This fails
INSERT INTO logs VALUES (2, '{"level": "info", "message": "started"');
-- ERROR 3819 (HY000): Check constraint 'valid_data' is violated.
```

**Expected Output:**
```
Query OK, 1 row affected (0.00 sec)
ERROR 3819 (HY000): Check constraint 'valid_data' is violated.
```

**Why This Output Occurs**: The `CHECK` constraint calls `JSON_VALID()` on every insert. The first row is valid JSON; the second is missing a closing brace, so `JSON_VALID()` returns 0 and the insert is rejected .

### Real-World Cases

- **Data Ingestion**: Validating JSON from external APIs before storage
- **Configuration Management**: Ensuring config JSON meets expected schema
- **Log Storage**: Rejecting malformed log entries

### References

- MySQL JSON Cheat Sheet - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-json-functions-cheat-sheet/README.md 
- SQL Insert JSON Format Error Handling - http://www.php.cn/faq/3034273.html 
- SQL Server JSON Support - https://jamsql.com/blog/sql-server-json-support-guide/ 


## Consolidated References

- MySQL JSON Functions Cheat Sheet - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-json-functions-cheat-sheet/README.md 
- PostgreSQL 9.5 Documentation - JSON Types - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.5/postgresql-9.5-A4.pdf 
- Stack Overflow - PostgreSQL JSON Path Expressions - https://stackoverflow.com/revisions/83202a7f-5cad-4409-91b4-606a5faa6437/view-source 
- MySQL 8.0 JSON Function Reference - https://dev.mysql.com/doc/refman/8.0/en/json-function-reference.html 
- PostgreSQL 9.5 Documentation - JSON Aggregates - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.5/postgresql-9.5-A4.pdf 
- PostgreSQL 15 Documentation - GIN Indexes - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/15/postgresql-15-A4.pdf 
- SQL Insert JSON Format Error Handling - http://www.php.cn/faq/3034273.html 
- SQL Server JSON Support Guide - https://jamsql.com/blog/sql-server-json-support-guide/ 
- AWS DMS Playbook - JSON in SQL Server vs PostgreSQL - https://docs.aws.eu/dms/latest/sql-server-to-aurora-postgresql-migration-playbook/chap-sql-server-aurora-pg.tsql.json.html 
- PostgreSQL 9.4 Documentation - JSON Containment - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.4/postgresql-9.4-A4.pdf 
- Stack Overflow - PostgreSQL JSONB Filtering - https://stackoverflow.com/posts/75273850/revisions 
- PostgreSQL 正體中文使用手冊 - JSON 函式與運算子 - https://docs.postgresql.tw/18/the-sql-language/functions/functions-json 
- PostgreSQL 15 Documentation - jsonpath Type - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/15/postgresql-15-A4.pdf 
- SQL Insert JSON Failure Solutions - https://global.php.cn/fr/faq/1797088408.html 
- Redgate - Storing and Parsing JSON in SQL Server - https://www.red-gate.com/simple-talk/databases/sql-server/t-sql-programming-sql-server/effective-strategies-for-storing-and-parsing-json-in-sql-server/ 
- PostgreSQL Mailing List - json vs jsonb - https://www.postgresql.org/message-id/CA%2BbJJbxQjAHqsUyWswUwJ65O4kRvam3jqCiN33RW77%2Bg%2By8Xxg%40mail.gmail.com 
- Stack Overflow - PostgreSQL JSONB Operators - https://stackoverflow.com/posts/77046858/timeline 
- MySQL JSON Function Reference (Oracle Mirror) - https://oracle.hydrogen.sagittarius.connect.product.adaptavist.com/cd/E17952_01/mysql-8.0-en/json-function-reference.html 