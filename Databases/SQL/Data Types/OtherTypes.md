# Comprehensive Programming Cheat Sheet: SQL XML and Specialized Types

## Topic Overview

### Definitions

**Core Definition**
XML and specialized types are the PostgreSQL data types beyond the core numeric, character, and temporal families that address domain-specific needs: structured document storage (`XML`), large binary objects (`Large Objects` and `BYTEA`), and specialised value types for network addresses, geometric shapes, text search, UUIDs, bit strings, and more.

**Technical Definition**
PostgreSQL provides a rich set of native data types, and users can add new types with `CREATE TYPE`. The `xml` type stores XML data with well-formedness checking and type-safe operations, distinct from storing XML in a `text` field. Large Objects (`LO`) are a legacy blob-storage facility predating `bytea`, which is now the SQL-typed default for binary data up to roughly 10 MB per value. Specialised types include network addresses (`inet`, `cidr`, `macaddr`, `macaddr8`), geometric types (`point`, `line`, `lseg`, `box`, `path`, `polygon`, `circle`), text search types (`tsvector`, `tsquery`), `uuid`, bit strings, `json`/`jsonb`, range types, and `pg_lsn`/`pg_snapshot`. Several of these types are unique to PostgreSQL, such as geometric paths, or have several possible formats, such as the date and time types.

**Beginner-Friendly Explanation**
Most of the time you store numbers, text, and dates. But sometimes you need to store an entire XML document, a multi‑megabyte image, an IP address, a map coordinate, or a pre‑processed search index. PostgreSQL has specialised containers for each of these jobs. `XML` validates that what you put in is well‑formed XML. Large Objects are for huge binary files that are too big to load into memory all at once. `BYTEA` is the modern default for ordinary binary data. And the specialised types—network, geometric, text search, UUID, bit string—each come with their own operators and functions so you do not have to reimplement domain logic in application code.

### Key Characteristics

- **`XML` validates well‑formedness**: The `xml` type checks input for well‑formedness and provides type‑safe functions, but it does **not** validate against a DTD or XML Schema; for schema validation you need application‑level or extension support.
- **Large Objects are legacy, `BYTEA` is modern**: Large Objects predate `BYTEA`; `BYTEA` is the SQL‑typed default for binary data up to roughly 10 MB per value, supports indexing and replication cleanly, and has no orphan‑cleanup problem. Reach for Large Objects only when values exceed roughly 10 MB and streaming I/O matters.
- **Specialised types carry specialised operators**: Network types have containment and overlap operators; geometric types have distance, overlap, and positional operators; text search types have match, rank, and highlight operators; `uuid` has comparison and generation functions.
- **Vendor divergence is substantial**: MySQL provides only `ExtractValue()` and `UpdateXML()` for XML, with basic XPath 1.0 support and no native XML type. SQL Server provides a native `xml` type with methods `query()`, `value()`, `exist()`, `modify()`, and `nodes()`, plus XML indexes and schema validation.
- **Type choice is a first‑order design decision**: Choosing the appropriate data type is crucial for efficient database design, because it affects performance, storage usage, and the complexity of queries.

### Prerequisites

- A running PostgreSQL instance (XML type requires the server to be built with `configure --with-libxml`).
- Familiarity with `CREATE TABLE` and basic SQL.
- For Large Objects: understanding of the `lo` extension and the `oid` type.
- For specialised types: awareness of the operators and functions specific to each family (GiST indexes for geometric and text search types, GIN indexes for `jsonb` and `tsvector`, etc.).

### Related Programming Areas

- Data Definition Language (DDL) – column type declarations and index creation.
- Data Manipulation Language (DML) – inserting, querying, and updating specialised values.
- Information retrieval – text search types and ranking.
- Geospatial and network engineering – geometric and network address types.
- Document management – XML storage, XPath/XQuery processing, and Large Object handling.

---

## Core Concepts and Features

### 1. XML-Related Types

#### Definitions

**Core Definition**
The `XML` type is a PostgreSQL data type that stores well‑formed XML documents and content fragments, providing well‑formedness checking and type‑safe XML operations.

**Technical Definition**
The `xml` data type can be used to store XML data. Its advantage over storing XML in a `text` field is that it checks input values for well‑formedness and provides support functions for type‑safe operations. The `xml` type can store well‑formed “documents” as defined by the XML standard, as well as “content” fragments, which are more permissive and can have more than one top‑level element or character node; the expression `xmlvalue IS DOCUMENT` tests whether a value is a full document or only a content fragment. The type does not validate against a DTD or XML Schema. To produce an `xml` value from character data, use `XMLPARSE(DOCUMENT|CONTENT value)`; to convert back, use `XMLSERIALIZE(DOCUMENT|CONTENT value AS type [NO] INDENT)`. PostgreSQL offers `xpath()` and `xpath_exists()` to evaluate XPath 1.0 expressions, and the `XMLTABLE` table function to produce a table from an XML value. Functions for building XML content include `xmlelement()`, `xmlforest()`, `xmlconcat()`, `xmlcomment()`, `xmlpi()`, and `xmlroot()`.

**Beginner-Friendly Explanation**
`XML` is the database's native container for XML documents. Unlike dumping XML into a `text` column, the `xml` type refuses malformed input and gives you functions to query and build XML. You can ask “is this a full document or just a fragment?”, extract values with XPath, and turn query results into XML.

#### Purposes

- To store XML documents and fragments with guaranteed well‑formedness.
- To query XML content using XPath 1.0 expressions.
- To build XML output from relational data using `xmlelement()`, `xmlforest()`, and related functions.
- To interoperate with systems that exchange XML payloads.

#### Syntax Rules and Structure

**Column declaration:**

```sql
column_name XML
```

**Creating XML values:**

```sql
XMLPARSE(DOCUMENT '<?xml version="1.0"?><book>...</book>')
XMLPARSE(CONTENT 'abc<foo>bar</foo><bar>foo</bar>')
xml '<foo>bar</foo>'
'<foo>bar</foo>'::xml
```

**Serializing XML:**

```sql
XMLSERIALIZE(DOCUMENT xml_value AS text INDENT)
```

**Querying XML:**

```sql
xpath('/book/title/text()', xml_value)
xpath_exists('/book/title', xml_value)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `XML` | Well‑formedness‑checking XML type. |
| `XMLPARSE` | Converts character data to `xml` with DOCUMENT or CONTENT semantics. |
| `XMLSERIALIZE` | Converts `xml` to character type with optional indentation. |
| `xpath()` | Evaluates an XPath 1.0 expression. |
| `xmltable()` | Shreds XML into relational rows. |

**Syntax rules:**

- `XMLPARSE` is the SQL‑standard way to create `xml` values; PostgreSQL also accepts `xml '...'` and `'...'::xml`.
- `XMLSERIALIZE` is the SQL‑standard way to convert back; PostgreSQL also allows a simple cast.
- The `IS DOCUMENT` expression tests whether an `xml` value is a full document.

**Constraints and limitations:**

- The `xml` type does not validate against DTDs or XML Schemas.
- Use of the type requires the server to be built with `--with-libxml`.
- MySQL provides only `ExtractValue()` and `UpdateXML()` for XML, with basic XPath 1.0 support and no native XML type. SQL Server provides a native `xml` type with `query()`, `value()`, `exist()`, `modify()`, and `nodes()` methods.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Storing and querying XML

```sql
-- Step 1: Create a table with an XML column.
CREATE TABLE books (
    book_id serial PRIMARY KEY,
    data    xml NOT NULL
);

-- Step 2: Insert a well-formed XML document.
INSERT INTO books (data)
VALUES (XMLPARSE(DOCUMENT '<?xml version="1.0"?><book><title>Manual</title><author>Alice</author></book>'));

-- Step 3: Extract the title with XPath.
SELECT (xpath('/book/title/text()', data))[1]::text AS title
FROM books;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
  title
---------
 Manual
(1 row)
```

**Why this output appears**: `XMLPARSE(DOCUMENT ...)` validates the input as a well‑formed XML document. `xpath('/book/title/text()', data)` returns an array of XML nodes; `[1]::text` extracts the first node as text, producing `Manual`.

#### Real-World Cases

- **Document management**: A `documents` table stores legal or technical documents as `xml`, enabling XPath queries for metadata.
- **Data exchange**: XML payloads from external systems are stored natively and queried without parsing in application code.

---

### 2. Large Object Types

#### Definitions

**Core Definition**
Large Objects (`LO`) are PostgreSQL's legacy facility for storing very large binary values outside ordinary table rows, while `BYTEA` is the modern SQL‑typed default for binary data within rows.

**Technical Definition**
Large Objects predate `bytea` and store data in a separate “large object space” with streaming I/O via `lo_creat`, `lo_import`, `lo_export`, `lo_read`, `lo_write`, `lo_lseek`, and related functions. `BYTEA` is the SQL‑typed default for binary data up to roughly 10 MB per value, supports indexing and replication cleanly, and has no orphan‑cleanup story; Large Objects are the better choice only when values exceed roughly 10 MB and streaming I/O matters. Server‑side Large Object functions include `lo_from_bytea()`, `lo_put()`, and `lo_get()`, which allow SQL access to Large Object contents. The server‑side `lo_import` and `lo_export` functions read and write files on the server's file system and are restricted to superusers by default because of security implications.

**Beginner-Friendly Explanation**
Large Objects are for files that are too big to load into memory all at once—videos, huge backups, massive documents. `BYTEA` is for ordinary binary data—a small image, a hash, an encrypted token—that fits comfortably in a table row. PostgreSQL's own documentation warns that Large Objects are legacy and that `BYTEA` is the modern default for most new code.

#### Purposes

- To store very large binary values (over ~10 MB) with streaming access.
- To import and export files from the server's file system (with appropriate privileges).
- To store ordinary binary data (`BYTEA`) inline with table rows for simplicity and transactional consistency.

#### Syntax Rules and Structure

**Large Object storage pattern:**

```sql
CREATE EXTENSION IF NOT EXISTS lo;

CREATE TABLE documents (
    doc_id   bigserial PRIMARY KEY,
    content  lo NOT NULL,   -- lo domain over oid
    uploaded timestamptz DEFAULT now()
);
```

**Server‑side functions:**

```sql
SELECT lo_from_bytea(0, '\xDEADBEEF');   -- create LO from bytea
SELECT lo_get(loid, 0, 100);              -- read first 100 bytes
SELECT lo_put(loid, 0, '\xCAFE');         -- write bytes
SELECT lo_unlink(loid);                   -- delete LO
```

**`BYTEA` storage pattern:**

```sql
CREATE TABLE attachments (
    attachment_id serial PRIMARY KEY,
    filename      text NOT NULL,
    data          bytea NOT NULL
);
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `lo` | Domain over `oid` representing a Large Object reference. |
| `lo_from_bytea`, `lo_get`, `lo_put` | Server‑side functions for creating, reading, and writing LOs. |
| `bytea` | Binary data type for values up to ~10 MB, stored inline. |

**Syntax rules:**

- Large Objects are referenced by an `oid` stored in a table column.
- `BYTEA` values are stored directly in the table row (subject to TOAST for very large values).
- `lo_import` and `lo_export` operate on the server's file system and require superuser or explicitly granted privileges.

**Constraints and limitations:**

- Large Objects have no automatic orphan cleanup; deleting the referencing row does not delete the LO.
- `BYTEA` values must generally be fetched whole; streaming requires the Large Object API.
- Server‑side `lo_import`/`lo_export` are security‑sensitive and restricted to superusers by default.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: BYTEA for an image attachment

```sql
-- Step 1: Create a table with a BYTEA column.
CREATE TABLE attachments (
    attachment_id serial PRIMARY KEY,
    filename      text NOT NULL,
    data          bytea NOT NULL
);

-- Step 2: Insert a small binary value.
INSERT INTO attachments (filename, data)
VALUES ('hello.bin', '\x48656c6c6f2c20576f726c6421');

-- Step 3: Query the size and hex representation.
SELECT filename, octet_length(data) AS bytes, encode(data, 'hex') AS hex
FROM attachments;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
 filename  | bytes |             hex
-----------+-------+-----------------------------
 hello.bin |    13 | 48656c6c6f2c20576f726c6421
(1 row)
```

**Why this output appears**: The `BYTEA` column stores the raw bytes. `octet_length` returns 13, and `encode(..., 'hex')` renders the bytes in hexadecimal form. `BYTEA` is the modern default for binary data of this size.

##### Example 2: Large Object for a large file

```sql
-- Step 1: Create the lo extension and a table with an lo column.
CREATE EXTENSION IF NOT EXISTS lo;
CREATE TABLE big_docs (
    doc_id  bigserial PRIMARY KEY,
    content lo NOT NULL
);

-- Step 2: Create a large object from a file on the server.
INSERT INTO big_docs (content)
VALUES (lo_import('/var/data/large_video.mp4'));

-- Step 3: Read the first 100 bytes.
SELECT lo_get(content, 0, 100) FROM big_docs;
```

**Expected output:**

```text
CREATE EXTENSION
CREATE TABLE
INSERT 0 1
                lo_get
---------------------------------------
 \x00000018667479706d703432...
(1 row)
```

**Why this output appears**: `lo_import` reads the server‑side file and stores it as a Large Object, returning its OID. `lo_get(content, 0, 100)` reads the first 100 bytes without loading the entire object into memory, demonstrating the streaming access that motivates Large Objects for very large values.

#### Real-World Cases

- **Media archives**: Large video and audio files are stored as Large Objects with streaming access, while metadata lives in ordinary columns.
- **Document attachments**: Small PDFs and images use `BYTEA` for transactional simplicity; multi‑gigabyte backups use Large Objects for streaming.

---

### 3. Specialized Types

#### Definitions

**Core Definition**
Specialised types are the PostgreSQL data types that address domain‑specific needs beyond numbers, text, and dates: network addresses, geometric shapes, text search documents and queries, UUIDs, bit strings, and system types.

**Technical Definition**
PostgreSQL's specialised families include: **network address types** (`inet`, `cidr`, `macaddr`, `macaddr8`) for IPv4, IPv6, and MAC addresses; **geometric types** (`point`, `line`, `lseg`, `box`, `path`, `polygon`, `circle`) for two‑dimensional spatial objects; **text search types** (`tsvector`, `tsquery`) for full‑text search documents and queries; **UUID** for 128‑bit identifiers; **bit strings** (`bit`, `bit varying`) for fixed and variable‑length binary digits; **`json`/`jsonb`** for JSON data; **range types** for intervals; and **system types** such as `pg_lsn` (PostgreSQL Log Sequence Number) and `pg_snapshot` (transaction ID snapshot). Many of these types come with dedicated operators, functions, and index support (GiST for geometric and text search types, GIN for `jsonb` and `tsvector`).

**Beginner-Friendly Explanation**
Specialised types are the “power tools” of the database. If you store IP addresses as `inet`, the database validates them and lets you ask “is this address inside that subnet?”. If you store shapes as `polygon`, you can ask “do these shapes overlap?”. If you store search documents as `tsvector`, you get fast full‑text search with ranking. Choosing the right specialised type is often the difference between clean, fast queries and fragile string manipulation.

#### Purposes

- To store domain values with built‑in validation and specialised operators.
- To leverage index types (GiST, GIN) designed for the domain.
- To avoid reimplementing domain logic in application code.
- To improve query performance through type‑specific optimisations.

#### Syntax Rules and Structure

| Family | Types | Key operators/functions | Typical index |
|--------|-------|-------------------------|---------------|
| Network | `inet`, `cidr`, `macaddr`, `macaddr8` | `<<`, `>>`, `&&`, `<<=`, `>>=`; `host()`, `netmask()`, `broadcast()` | B‑tree; GiST with `inet_ops` |
| Geometric | `point`, `line`, `lseg`, `box`, `path`, `polygon`, `circle` | `&&`, `@>`, `<@`, `<->`, `@@`, `#`, `area()` | GiST |
| Text search | `tsvector`, `tsquery` | `@@`, `||`, `&&`, `!!`, `<->`, `ts_rank()`, `ts_headline()` | GIN (preferred), GiST |
| UUID | `uuid` | `=`, `<`, `>`, `gen_random_uuid()`, `uuidv7()` | B‑tree |
| Bit string | `bit`, `bit varying` | `&`, `|`, `#`, `~`, `<<`, `>>`, `get_bit()`, `set_bit()` | B‑tree |
| JSON | `json`, `jsonb` | `->`, `->>`, `#>`, `@>`, `?` | GIN |
| Range | `int4range`, `int8range`, `numrange`, `tsrange`, `tstzrange`, `daterange` | `&&`, `@>`, `<@`, `-|-`, `+`, `*`, `-` | GiST |
| System | `pg_lsn`, `pg_snapshot` | Specialised comparison and inspection functions | B‑tree |

**Syntax rules:**

- Network types accept CIDR notation and host addresses; `inet` accepts hosts, `cidr` accepts only networks.
- Geometric types use parenthesised coordinate syntax and support GiST indexing for spatial operators.
- Text search types use `to_tsvector()` and `to_tsquery()` to build values, and `@@` to match.
- UUIDs can be generated with `gen_random_uuid()` (PostgreSQL 13+) or `uuidv7()` (PostgreSQL 18+).
- Bit strings support bitwise logic and shifts.

**Constraints and limitations:**

- Specialised types are often PostgreSQL‑specific; porting to MySQL or SQL Server may require different types or application‑level logic.
- GiST and GIN indexes have different write‑performance characteristics than B‑tree indexes.
- Some specialised types (e.g., `pg_lsn`) are intended for replication and system monitoring, not application data.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Network address type for subnet queries

```sql
-- Step 1: Create a table with an INET column.
CREATE TABLE servers (
    server_id serial PRIMARY KEY,
    hostname  text NOT NULL,
    ip        inet NOT NULL
);

-- Step 2: Insert hosts.
INSERT INTO servers (hostname, ip) VALUES
    ('web-01', '192.168.1.10'),
    ('db-01',  '10.0.0.5');

-- Step 3: Query hosts inside a subnet.
SELECT hostname, ip
FROM servers
WHERE ip << inet '192.168.1.0/24';
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 2
 hostname |      ip
----------+-------------
 web-01   | 192.168.1.10
(1 row)
```

**Why this output appears**: The `<<` operator tests whether an address is contained within a subnet. `192.168.1.10` is inside `192.168.1.0/24`; `10.0.0.5` is not, so only `web-01` is returned.

##### Example 2: Text search type for full‑text matching

```sql
-- Step 1: Create a table with a TSVECTOR column.
CREATE TABLE articles (
    article_id serial PRIMARY KEY,
    title      text NOT NULL,
    search     tsvector
);

-- Step 2: Insert a document with a generated tsvector.
INSERT INTO articles (title, search)
VALUES ('PostgreSQL Text Search', to_tsvector('english', 'PostgreSQL Text Search'));

-- Step 3: Match a query.
SELECT title
FROM articles
WHERE search @@ to_tsquery('english', 'postgresql');
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
         title
-------------------------
 PostgreSQL Text Search
(1 row)
```

**Why this output appears**: `to_tsvector` normalises the text into lexemes, and `to_tsquery` builds a query for the lexeme `postgresql`. The `@@` operator matches the document against the query, returning the article whose title contains the term.

#### Real-World Cases

- **Network management**: A `servers` table uses `inet` for IP addresses and `<<` for subnet‑based inventory queries.
- **Geospatial analysis**: A `parcels` table uses `polygon` with a GiST index for overlap and containment queries.
- **Search**: An `articles` table uses `tsvector` and a GIN index for full‑text search with ranking.
- **Identifiers**: A `users` table uses `uuid` for opaque public identifiers.

---

### 4. Choosing Appropriate Data Types

#### Definitions

**Core Definition**
Choosing appropriate data types is the design process of selecting the most accurate, efficient, and maintainable type for each column, balancing correctness, storage, performance, and operational complexity.

**Technical Definition**
Choosing the appropriate data type is crucial for efficient database design, as it affects performance, storage usage, and the complexity of queries. PostgreSQL's robust type system ensures data integrity (e.g., rejecting out‑of‑range values) and provides specialised operators and indexing optimisations tailored to each type. Best practices include storing values in their native types rather than as strings—for example, use `bytea` for binary data rather than an encoded string, use integer or `bigint` for integers, and use the appropriate database type for every value. Use `bigint` rather than `integer` for future‑proofing identifiers, `numeric` rather than `float` for money, and `text` rather than `varchar(n)` unless a length constraint is genuinely needed. For very large binary values, prefer `BYTEA` unless streaming is required, in which case Large Objects are appropriate.

**Beginner-Friendly Explanation**
Choosing a data type is like choosing the right container for a physical object. A small box for a ring, a large crate for a sofa, a waterproof container for a liquid. If you pick the wrong container, you either waste space or damage the contents. In databases, the wrong type can cause silent truncation, slow queries, or invalid data. The rule of thumb is: store each value in the most specific type that exactly represents it, and never store numbers, dates, IP addresses, or binary data as generic strings.

#### Purposes

- To ensure data integrity by rejecting invalid values at the source.
- To minimise storage consumption and maximise query performance.
- To enable specialised operators and indexes that would not be available for generic types.
- To make the schema self‑documenting and maintainable.

#### Syntax Rules and Structure

**Decision guide:**

| Data domain | Recommended type | Avoid |
|-------------|------------------|-------|
| Integers | `integer` or `bigint` | `text` |
| Money | `numeric` | `float` / `double precision` |
| Strings with no length limit | `text` | `varchar(n)` unless a constraint is required |
| Strings with a business‑rule length | `varchar(n)` or `text` + `CHECK` | Arbitrary `varchar(255)` |
| Binary data (≤ ~10 MB) | `bytea` | Encoded `text` |
| Very large binary data (> ~10 MB, streaming) | Large Objects (`lo`) | `bytea` loaded whole |
| IPv4/IPv6 addresses | `inet` / `cidr` | `text` |
| MAC addresses | `macaddr` / `macaddr8` | `text` |
| Geometric shapes | `point`, `polygon`, etc. | `text` coordinates |
| Full‑text search | `tsvector` / `tsquery` | `LIKE` on `text` |
| JSON documents | `jsonb` | `text` |
| UUIDs | `uuid` | `text` |

**Syntax rules:**

- Use the most specific type that can represent the domain.
- Use `bigint` for identifiers unless the 32‑bit range is provably sufficient.
- Use `numeric` for money; floating‑point types are inexact and unsuitable for monetary values.
- Use `bytea` for binary data unless streaming very large values is required.

**Constraints and limitations:**

- Changing a column's type later can require a table rewrite and may be disruptive.
- Over‑optimising for storage (e.g., `smallint` for a value that may exceed 32,767) risks overflow.
- Specialised types may reduce portability to other RDBMSs.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Applying type‑selection best practices

```sql
-- Step 1: Create a table using domain-appropriate types.
CREATE TABLE orders (
    order_id     bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id  bigint NOT NULL,
    total_amount numeric(12,2) NOT NULL,
    notes        text,
    created_at   timestamptz NOT NULL DEFAULT now()
);
```

**Expected output:**

```text
CREATE TABLE
```

**Why this output appears**: `bigint` future‑proofs the identifiers; `numeric(12,2)` stores exact monetary amounts; `text` stores free‑form notes without an arbitrary length limit; `timestamptz` stores time‑zone‑aware timestamps. Each choice follows the best‑practice guidance for its domain.

#### Real-World Cases

- **Financial systems**: `numeric` is used for balances and amounts; `bigint` for transaction IDs; `timestamptz` for posting times.
- **Geospatial applications**: `polygon` or `point` with GiST indexes for shapes and locations; `inet` for server inventories.
- **Content platforms**: `jsonb` for flexible metadata; `tsvector` for search; `bytea` for thumbnails; `uuid` for opaque public IDs.

---

## References

- PostgreSQL Documentation – XML Type – https://www.postgresql.org/docs/current/datatype-xml.html 
- PostgreSQL Documentation – XML Functions – https://www.postgresql.org/docs/current/functions-xml.html 
- PostgreSQL Documentation – Large Objects: Server‑Side Functions – https://www.postgresql.org/docs/current/lo-funcs.html 
- PostgreSQL Documentation – Large Objects (LO) Reference – https://github.com/damusix/skills/blob/main/postgres/references/71-large-objects.md 
- PostgreSQL Documentation – Data Types (overview) – https://www.postgresql.org/docs/devel/datatype.html 
- Tiger Data – Optimizing Your Database: A Deep Dive into PostgreSQL Data Types – https://www.tigerdata.com/learn/optimizing-your-database-a-deep-dive-into-postgresql-data-types 
- MySQL 8.0 Reference Manual – XML Functions – https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/xml-functions.html 
- Microsoft Learn – XML Data Type Methods (SQL Server) – https://learn.microsoft.com/fr-fr/sql/t-sql/xml/xml-data-type-methods 
- PostgreSQL Best Practices – Data Types – https://github.com/damusix/skills/blob/main/postgres-writing-guidelines/references/data-types.md 