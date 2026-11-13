# Comprehensive Programming Cheat Sheet: SQL Binary Types

## Topic Overview

### Definitions

**Core Definition**
Binary types are the SQL data types used to store raw binary data—sequences of bytes that are not associated with any character set or encoding—including fixed-length binary strings, variable-length binary strings, and large binary objects (BLOBs), as well as the closely related character large object (CLOB) type for very large text.

**Technical Definition**
SQL binary types fall into two families: **binary string types** that store sequences of octets as raw bytes (`BYTEA` in PostgreSQL, `BINARY`/`VARBINARY` in SQL Server, `BINARY`/`VARBINARY` in MySQL), and **large object types** designed for very large values (`BLOB` and its size variants in MySQL, `BYTEA` in PostgreSQL up to 1 GB, `VARBINARY(MAX)` in SQL Server up to 2 GB). Binary strings are distinguished from character strings in two crucial ways: they allow storage of zero-valued octets and other non-printable bytes, and operations on them process the actual bytes rather than applying locale-dependent character-set rules. The SQL standard defines a binary string type called `BLOB` (Binary Large Object), and PostgreSQL's `bytea` type provides similar functionality with a different input format but largely the same functions and operators. The `CLOB` (Character Large Object) type, while sometimes conflated with binary types, is a character type for very large text; MySQL implements it through its `TEXT` family (`TINYTEXT`, `TEXT`, `MEDIUMTEXT`, `LONGTEXT`), which carry an associated character set unlike `BLOB` types.

**Beginner-Friendly Explanation**
Binary types are the containers for data that is not text—images, PDFs, encrypted blobs, audio files, compressed archives, and serialized objects. Unlike `VARCHAR` or `TEXT`, which store characters and depend on an encoding, binary types store the exact bytes you put in, byte for byte. PostgreSQL calls its binary type `BYTEA`; MySQL calls its binary types `BLOB` (with four size variants); SQL Server calls them `VARBINARY` (with a special `MAX` variant for large data). `CLOB` is not a binary type at all—it is a character type for very large text, but it is often discussed alongside `BLOB` because the two are used for large objects. Choosing the right binary type is like choosing the right container for a physical object: a small box for a thumbnail, a large crate for a video, and a completely different container (a filing cabinet) for text.

### Key Characteristics

- **No character set, no encoding**: Binary strings store raw octets. They allow zero-valued bytes and non-printable bytes that character strings disallow, and operations on them process bytes directly rather than applying locale or character-set rules.
- **Size tiers define the type family**: MySQL's `BLOB` family spans four size tiers—`TINYBLOB` (255 bytes), `BLOB` (65,535 bytes), `MEDIUMBLOB` (16,777,215 bytes), and `LONGBLOB` (4,294,967,295 bytes). PostgreSQL's `BYTEA` holds up to 1 GB but requires substantial memory to process very large values. SQL Server's `VARBINARY(MAX)` holds up to 2^31−1 bytes (2 GB), with values over 8,000 bytes stored off-row.
- **Storage location varies**: Small binary values are stored inline in the row; large values may be stored off-row (out-of-line) depending on the RDBMS, the type, and the value size. SQL Server stores `VARBINARY(MAX)` values larger than 8,000 bytes in a separate location with only a pointer in the row. PostgreSQL's `TOAST` mechanism moves large values to a side table when a row exceeds its page-size limit.
- **`BYTEA` has two output formats**: PostgreSQL's `bytea` supports “hex” format (default, introduced in PostgreSQL 9.0) and the historical “escape” format; hex is compatible with a wide range of applications and is faster to convert, so its use is preferred.
- **`BLOB` and `TEXT` are not interchangeable**: In MySQL, `BLOB` types have no character set and sort based on numeric byte values; `TEXT` types have an associated character set and collation and sort according to character rules. A `CLOB` is a character type, not a binary type.

### Prerequisites

- A running RDBMS instance (PostgreSQL, MySQL, or SQL Server) and a client connection.
- `CREATE` or `ALTER` privilege on the target table; `INSERT`/`SELECT` privilege for data manipulation.
- For MySQL `LOAD_FILE()`: the `FILE` privilege and the file must reside on the server host.
- For PostgreSQL `BYTEA`: awareness of the `bytea_output` setting (hex by default) and the encoding implications of passing binary data through clients that expect text.
- For very large objects: a strategy for chunking, streaming, or external storage, because loading a 1 GB `BYTEA` into memory can exhaust application resources.

### Related Programming Areas

- Data Definition Language (DDL) – column type declarations.
- Data Manipulation Language (DML) – inserting, updating, and retrieving binary data.
- Application programming – JDBC `getBytes`/`setBytes`/`getBinaryStream`/`setBinaryStream`, `bytea` handling, and file I/O.
- Storage engineering – TOAST, off-row storage, and large object (LOB) management.
- Security and compliance – encryption of sensitive binary data (keys, certificates), hashing, and digital signatures.
- Backup and replication – binary values are included in logical and physical backups; large objects may require special handling.

---

## Core Concepts and Features

### 1. `BYTEA` (PostgreSQL)

#### Definitions

**Core Definition**
`BYTEA` is PostgreSQL's native data type for storing variable-length binary strings—sequences of octets that are not interpreted as characters.

**Technical Definition**
`BYTEA` stores variable-length binary strings with a storage size of **1 or 4 bytes** (depending on the value length) plus the actual binary string. A `BYTEA` column can hold up to **1 GB** of binary data, but processing such a large value requires a huge amount of memory. `BYTEA` supports two input/output formats: **hex** (the default since PostgreSQL 9.0, preferred for speed and compatibility) and **escape** (the historical format, which should be avoided for new applications because it blurs the distinction between binary and character strings). In hex format, binary data is encoded as two hexadecimal digits per byte, preceded by `\x` (e.g., `\xDEADBEEF`).

**Beginner-Friendly Explanation**
`BYTEA` is PostgreSQL's container for raw bytes. You can put an image, a PDF, or an encrypted key into it, and PostgreSQL will store the exact bytes. It uses a special notation called “hex” to display the data, where each byte is shown as two hexadecimal digits—so the byte `255` appears as `ff`. `BYTEA` is convenient because it is backed up with the table and follows the normal replication stream, but it is not ideal for very large files because the whole value must be processed in memory.

#### Purposes

- To store small to medium-sized binary data (thumbnails, icons, encryption keys, hashes, serialized objects) directly alongside structured data.
- To store binary data that must be included in logical backups and replication without special handling.
- To provide a simple, direct binary container that avoids the complexity of PostgreSQL's Large Object (LOB) subsystem.
- To exchange binary data with applications through `getBytes()`/`setBytes()` or `getBinaryStream()`/`setBinaryStream()`.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name BYTEA
```

**Hex literal input:**

```sql
-- Insert a bytea value using hex format.
INSERT INTO files (data) VALUES ('\xDEADBEEF');
```

**Escape format input (legacy):**

```sql
-- Insert using escape format (avoid for new applications).
INSERT INTO files (data) VALUES (E'\\001\\002\\003');
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `BYTEA` | Variable-length binary string type. |
| `\x...` | Hex format: two hex digits per byte, preceded by `\x`. |
| `E'...'` | Escape string syntax for the legacy escape format. |

**Syntax rules:**

- Hex format is the default output; set `bytea_output` to `escape` only for compatibility with legacy tools.
- In hex format, whitespace is permitted between digit pairs but not within a pair or in the `\x` prefix.
- The hex format is compatible with external applications and protocols and is faster to convert than the escape format.

**Constraints and limitations:**

- `BYTEA` can hold up to 1 GB, but very large values require huge memory to process.
- `BYTEA` values are stored inline in the table (subject to TOAST compression and out-of-line storage for large values).
- Binary data passed through text-based protocols may need hex or base64 encoding, doubling or quadrupling the transfer size.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Create a BYTEA column and insert binary data

```sql
-- Step 1: Create a table with a BYTEA column.
CREATE TABLE attachments (
    attachment_id serial PRIMARY KEY,
    filename      text NOT NULL,
    data          bytea NOT NULL
);

-- Step 2: Insert a small binary value using hex format.
INSERT INTO attachments (filename, data)
VALUES ('hello.bin', '\x48656c6c6f2c20576f726c6421');

-- Step 3: Query the binary data and its size.
SELECT filename, octet_length(data) AS bytes, encode(data, 'hex') AS hex
FROM attachments;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
  filename  | bytes |             hex
------------+-------+-----------------------------
 hello.bin  |    13 | 48656c6c6f2c20576f726c6421
(1 row)
```

**Why this output appears**: The hex literal `\x48656c6c6f2c20576f726c6421` represents the ASCII bytes for “Hello, World!”. `octet_length` returns 13 bytes, and `encode(data, 'hex')` renders the same hex string without the `\x` prefix.

##### Example 2: PostgreSQL – Insert binary data using a stream from JDBC

```java
// Step 1: Prepare an INSERT statement with a BYTEA column.
String sql = "INSERT INTO attachments (filename, data) VALUES (?, ?)";
try (PreparedStatement ps = conn.prepareStatement(sql)) {
    ps.setString(1, "image.png");
    // Step 2: Stream the file contents into the BYTEA column.
    try (InputStream fis = new FileInputStream("image.png")) {
        ps.setBinaryStream(2, fis, (int) new File("image.png").length());
        ps.executeUpdate();
    }
}
```

**Expected output:**

```text
(1 row inserted)
```

**Why this output appears**: The PostgreSQL JDBC driver maps `setBinaryStream()` to the `BYTEA` type; the stream is read and sent to the server as binary data, avoiding the need to load the entire file into a `byte[]` first.

#### Real-World Cases

- **User avatars**: A `users` table stores a small profile image in a `BYTEA` column, avoiding a separate file server and keeping the image backed up with the user row.
- **Encryption keys**: A key-management table stores AES keys or certificate blobs as `BYTEA`, benefiting from table-level backup and access controls.
- **Document hashes**: A `documents` table stores SHA-256 hashes of uploaded files as `BYTEA` for fast integrity verification.

---

### 2. BLOB (MySQL)

#### Definitions

**Core Definition**
`BLOB` (Binary Large Object) is MySQL's family of binary data types that store raw binary data without any character set encoding, available in four size tiers.

**Technical Definition**
MySQL provides four `BLOB` types with progressively larger maximum lengths: **`TINYBLOB`** (255 bytes, 1-byte length prefix), **`BLOB`** (65,535 bytes, 2-byte length prefix), **`MEDIUMBLOB`** (16,777,215 bytes, 3-byte length prefix), and **`LONGBLOB`** (4,294,967,295 bytes, 4-byte length prefix). `BLOB` types have no character set, and sorting and comparison are based on the numeric values of the bytes in the column values. `BLOB` columns cannot have a literal `DEFAULT` value; before MySQL 8.0.13, the only allowed default is `NULL`, and expression defaults such as `DEFAULT (X'')` are permitted from 8.0.13 onward. In InnoDB, `BLOB` and `TEXT` values can be stored inline within the clustered index record or externally off-page depending on their size and the row format.

**Beginner-Friendly Explanation**
MySQL's `BLOB` is like PostgreSQL's `BYTEA`, but it comes in four sizes. `TINYBLOB` is for tiny thumbnails and icons; `BLOB` is for small images and documents; `MEDIUMBLOB` is for medium images and audio clips; and `LONGBLOB` is for videos and large backups. Unlike `TEXT`, a `BLOB` does not know about characters or encodings—it just stores bytes. That means you cannot sort a `BLOB` alphabetically; it sorts by byte value.

#### Purposes

- To store images, documents, audio, video, and other binary files directly in the database.
- To store encrypted content, serialized objects, or compressed data that must not be interpreted as text.
- To choose a size tier that matches the expected data volume, avoiding unnecessary length-prefix overhead.
- To keep binary data transactionally consistent with the rest of the row and included in backups.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name { TINYBLOB | BLOB | MEDIUMBLOB | LONGBLOB } [NOT NULL]
```

**Component breakdown:**

| Type | Maximum length | Length prefix | Typical use |
|------|----------------|---------------|-------------|
| `TINYBLOB` | 255 bytes | 1 byte | Thumbnails, small icons |
| `BLOB` | 65,535 bytes | 2 bytes | Small images, documents |
| `MEDIUMBLOB` | 16,777,215 bytes | 3 bytes | Medium images, audio clips |
| `LONGBLOB` | 4,294,967,295 bytes | 4 bytes | Videos, large backups |

**Syntax rules:**

- `BLOB` columns cannot have a literal `DEFAULT` value; use an expression default (MySQL 8.0.13+) or `NULL`.
- `BLOB` types have no character set; they cannot be compared or sorted using collation rules.
- The effective maximum length of `LONGBLOB` depends on `max_allowed_packet` and available memory.

**Constraints and limitations:**

- `BLOB` columns cannot be indexed in full without a prefix length (e.g., `INDEX (blob_col(100))`).
- Very large `BLOB` values can cause performance problems; consider storing them in a separate table or using external storage.
- `BLOB` data is not converted by character-set settings; it is stored and retrieved as raw bytes.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Store an image in a MEDIUMBLOB column

```sql
-- Step 1: Create a table with a MEDIUMBLOB column.
CREATE TABLE product_images (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    product_id  INT NOT NULL,
    image_type  ENUM('thumbnail', 'main', 'gallery') NOT NULL,
    mime_type   VARCHAR(50) NOT NULL,
    image_data  MEDIUMBLOB NOT NULL,
    file_size   INT UNSIGNED NOT NULL,
    uploaded_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    INDEX (product_id)
);

-- Step 2: Insert an image using a hex literal.
INSERT INTO product_images (product_id, image_type, mime_type, image_data, file_size)
VALUES (101, 'main', 'image/jpeg',
        X'ffd8ffe000104a46494600010100000100010000',  -- JPEG header bytes
        125000);

-- Step 3: Query the image size.
SELECT id, mime_type, LENGTH(image_data) AS stored_bytes FROM product_images;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 1 row affected (0.01 sec)
+----+-----------+--------------+
| id | mime_type | stored_bytes |
+----+-----------+--------------+
|  1 | image/jpeg|           20   |
+----+-----------+--------------+
```

**Why this output appears**: The `MEDIUMBLOB` column stores the binary JPEG data. `LENGTH(image_data)` returns the number of bytes in the value, which is 20 for the abbreviated hex literal used in the example. In a real application, the full image bytes would be inserted, and `file_size` would record the original file size.

##### Example 2: MySQL – Load a file from the server filesystem with LOAD_FILE()

```sql
-- Step 1: Create a table with a LONGBLOB column.
CREATE TABLE documents (
    doc_id   INT AUTO_INCREMENT PRIMARY KEY,
    doc_name VARCHAR(255) NOT NULL,
    content  LONGBLOB NOT NULL
);

-- Step 2: Load a file from the server filesystem (requires FILE privilege).
INSERT INTO documents (doc_name, content)
VALUES ('report.pdf', LOAD_FILE('/var/mysql-uploads/report.pdf'));

-- Step 3: Verify the size.
SELECT doc_name, LENGTH(content) AS bytes FROM documents;
```

**Expected output:**

```text
Query OK, 1 row affected (0.05 sec)
+------------+---------+
| doc_name   | bytes   |
+------------+---------+
| report.pdf | 2457600 |
+------------+---------+
```

**Why this output appears**: `LOAD_FILE()` reads the file from the server filesystem and returns its contents as a binary string, which is inserted into the `LONGBLOB` column. The `FILE` privilege is required, and the file must be readable by the MySQL server process.

#### Real-World Cases

- **Product catalogues**: A `product_images` table stores main images, thumbnails, and gallery images in `MEDIUMBLOB` columns, keeping image data transactionally consistent with product records.
- **Document management**: A `documents` table stores PDFs and Office files in `LONGBLOB` columns, enabling full-text search integration and access control at the database level.
- **Encryption key storage**: A `keys` table stores AES keys as `BLOB` values, benefiting from database-level encryption and access auditing.

---

### 3. CLOB (Character Large Object)

#### Definitions

**Core Definition**
`CLOB` (Character Large Object) is the SQL standard term for a character data type designed to store very large text values, implemented in MySQL through the `TEXT` family and in other databases through dedicated `CLOB` types.

**Technical Definition**
The SQL standard defines `CLOB` as a character large object type. MySQL does not have a type literally named `CLOB`; instead, it provides the `TEXT` family—`TINYTEXT`, `TEXT`, `MEDIUMTEXT`, and `LONGTEXT`—which are the CLOB equivalents. Unlike `BLOB` types, `TEXT` types have an associated character set and collation, and sorting and comparison are based on character values rather than numeric byte values. `TEXT` columns can have an optional length `M`, in which case MySQL creates the column as the smallest `TEXT` type large enough to hold values `M` characters long; the effective maximum length is less if the value contains multibyte characters. In InnoDB, `TEXT` values may be stored inline or off-page, similar to `BLOB` values.

**Beginner-Friendly Explanation**
`CLOB` is the text counterpart of `BLOB`. Where `BLOB` stores raw bytes, `CLOB` stores very large text with a known character set and encoding. MySQL calls its CLOB types `TEXT`, `MEDIUMTEXT`, and `LONGTEXT`. If you are storing a novel, a JSON document, or a long HTML page, a `TEXT` type is the right choice; if you are storing an image or a PDF, use a `BLOB` type instead.

#### Purposes

- To store very large text values (articles, books, JSON documents, HTML pages, XML files) that exceed the limits of `VARCHAR`.
- To preserve character-set and collation semantics for large text, enabling correct sorting and comparison.
- To store text that must be searchable with full-text indexes (where supported).
- To provide a character-oriented alternative to `BLOB` when the data is genuinely textual.

#### Syntax Rules and Structure

**General syntax (MySQL):**

```sql
column_name { TINYTEXT | TEXT | MEDIUMTEXT | LONGTEXT }
    [CHARACTER SET charset_name] [COLLATE collation_name]
```

**Component breakdown:**

| Type | Maximum length | Length prefix | Character set |
|------|----------------|---------------|---------------|
| `TINYTEXT` | 255 characters | 1 byte | Yes |
| `TEXT` | 65,535 characters | 2 bytes | Yes |
| `MEDIUMTEXT` | 16,777,215 characters | 3 bytes | Yes |
| `LONGTEXT` | 4,294,967,295 characters | 4 bytes | Yes |

**Syntax rules:**

- `TEXT` types can be given an optional length `M`; MySQL chooses the smallest type that can hold `M` characters.
- `TEXT` columns have a character set and collation; `BLOB` columns do not.
- `TEXT` columns cannot have a literal `DEFAULT` value (similar to `BLOB`).

**Constraints and limitations:**

- The effective maximum length of `TEXT` types is reduced when values contain multibyte characters.
- `TEXT` columns require a prefix length for indexing (e.g., `INDEX (text_col(100))`).
- `TEXT` values may be stored off-page, which can affect performance for queries that retrieve them.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Store an article body in a TEXT column

```sql
-- Step 1: Create a table with a TEXT column.
CREATE TABLE articles (
    article_id  INT AUTO_INCREMENT PRIMARY KEY,
    title       VARCHAR(200) NOT NULL,
    body        TEXT NOT NULL,
    published_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Step 2: Insert a long article body.
INSERT INTO articles (title, body)
VALUES ('SQL Binary Types',
        'Binary types store raw bytes, while character types store text. '
        'The BLOB family is for binary data, and the TEXT family is for text.');

-- Step 3: Query the character length.
SELECT title, CHAR_LENGTH(body) AS chars, LENGTH(body) AS bytes
FROM articles;
```

**Expected output:**

```text
+------------------+-------+-------+
| title            | chars | bytes |
+------------------+-------+-------+
| SQL Binary Types |   133 |   133 |
+------------------+-------+-------+
```

**Why this output appears**: `TEXT` stores the article body as a character string with an associated character set. `CHAR_LENGTH` returns the number of characters (133), and `LENGTH` returns the number of bytes (133 for ASCII text; more for multibyte text). Unlike a `BLOB`, the `TEXT` value can be sorted and compared using collation rules.

#### Real-World Cases

- **Content management**: A `posts` table stores article bodies, HTML content, and JSON metadata in `MEDIUMTEXT` or `LONGTEXT` columns, enabling full-text search and character-aware sorting.
- **Audit logs**: An `audit_log` table stores detailed JSON or XML payloads in `LONGTEXT` columns for compliance and troubleshooting.

---

### 4. Binary Operations (PostgreSQL)

#### Definitions

**Core Definition**
Binary operations are the functions and operators that examine, extract, concatenate, replace, and manipulate `BYTEA` values in PostgreSQL, operating on the raw bytes rather than on characters.

**Technical Definition**
PostgreSQL provides a rich set of binary string functions and operators for `BYTEA` values, many of which are equivalent in purpose and syntax to the text-string functions but operate on bytes. Core operations include concatenation (`||`), `bit_length()` (number of bits), `octet_length()` (number of bytes), `substring()` (extract a byte range), `overlay()` (replace a byte range), `position()` (find a byte sequence), `trim()` (remove bytes from the ends), and `encode()`/`decode()` for converting between `BYTEA` and textual representations (hex, base64, escape). Additional functions include `get_byte()` and `set_byte()` for individual byte access, `md5()` and `sha256()` for hashing, and `convert_to()`/`convert_from()` for encoding conversion.

**Beginner-Friendly Explanation**
Binary operations are the tools you use to work with `BYTEA` data inside PostgreSQL. You can take a slice of a binary value (substring), find where a byte sequence appears (position), concatenate two binary values, compute a hash, or convert the binary data to a hex string for display. These operations treat the data as a sequence of bytes, not as text, so they work correctly even when the bytes include zeros or non-printable values.

#### Purposes

- To extract or replace portions of a binary value (e.g., reading a header from a file blob).
- To compute cryptographic hashes (MD5, SHA-256) for integrity verification.
- To convert between `BYTEA` and textual representations for storage, transport, or debugging.
- To concatenate or compare binary values in queries and constraints.

#### Syntax Rules and Structure

**Core operators and functions:**

| Function / Operator | Return type | Description |
|---------------------|-------------|-------------|
| `bytea || bytea` | `bytea` | Concatenates two binary strings |
| `bit_length(bytea)` | `integer` | Number of bits (8 × `octet_length`) |
| `octet_length(bytea)` | `integer` | Number of bytes |
| `substring(bytea FROM start FOR count)` | `bytea` | Extract `count` bytes starting at `start` |
| `overlay(bytea PLACING new FROM start FOR count)` | `bytea` | Replace `count` bytes starting at `start` with `new` |
| `position(substring IN bytea)` | `integer` | First starting index of `substring` |
| `trim(bytesremoved FROM bytea)` | `bytea` | Remove bytes in `bytesremoved` from both ends |
| `encode(bytea, format)` | `text` | Encode to `hex`, `base64`, or `escape` |
| `decode(text, format)` | `bytea` | Decode from `hex`, `base64`, or `escape` |
| `md5(bytea)` | `text` | MD5 hash (hex) |
| `sha256(bytea)` | `bytea` | SHA-256 hash |

**Syntax rules:**

- `substring()` requires at least one of `start` or `count`.
- `overlay()` defaults `count` to the length of `newsubstring` if omitted.
- `encode()` and `decode()` support `hex`, `base64`, and `escape` formats.

**Constraints and limitations:**

- Binary operations are byte-oriented; they do not respect character boundaries or encodings.
- Hashing functions return hex-encoded text (`md5`) or `bytea` (`sha256`), requiring conversion for consistent comparison.
- Very large `BYTEA` values may be slow to process in memory.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Substring, position, and encode

```sql
-- Step 1: Create a table with a BYTEA column.
CREATE TABLE files (id int, data bytea);
INSERT INTO files VALUES (1, '\x48656c6c6f2c20576f726c6421');

-- Step 2: Extract a substring and find a byte sequence.
SELECT substring(data from 1 for 5) AS first_five,
       position('\x576f726c64' in data) AS world_pos,
       encode(data, 'base64') AS b64
FROM files;
```

**Expected output:**

```text
 first_five | world_pos |           b64
------------+-----------+-------------------------
 \x48656c6c |         8 | SGVsbG8sIFdvcmxkIQ==
(1 row)
```

**Why this output appears**: `substring(data from 1 for 5)` extracts the first five bytes (`Hello`). `position('\x576f726c64' in data)` finds the byte sequence for `World` starting at position 8. `encode(data, 'base64')` renders the entire value as a base64 string.

##### Example 2: PostgreSQL – Hashing with sha256 and encode

```sql
-- Step 1: Compute the SHA-256 hash of a BYTEA value.
SELECT encode(sha256('\x48656c6c6f'), 'hex') AS sha256_hex;
```

**Expected output:**

```text
                             sha256_hex
------------------------------------------------------------------
 185f8db32271fe25f561a6fc938b2e264306ec304eda518007d1764826381969
```

**Why this output appears**: `sha256('\x48656c6c6f')` returns a 32-byte `bytea` hash; `encode(..., 'hex')` converts it to a 64-character hex string for display and comparison.

#### Real-World Cases

- **File integrity checks**: A `documents` table stores a SHA-256 hash in a `BYTEA` column; a query compares the stored hash against `sha256(file_contents)` to detect tampering.
- **Protocol parsing**: A binary message stored in `BYTEA` is parsed with `substring()` and `position()` to extract headers, payloads, and checksums.
- **Data export**: Binary values are encoded to base64 with `encode(data, 'base64')` for transport through JSON APIs or text-based protocols.

---

### 5. Binary Best Practices

#### Definitions

**Core Definition**
Binary best practices are the guidelines for choosing, storing, retrieving, and managing binary data to ensure performance, integrity, and operational simplicity across RDBMSs.

**Technical Definition**
Best practices include: (1) **choose the right type and size tier**—use `BYTEA` for moderate binary values in PostgreSQL, `VARBINARY(MAX)` or `VARBINARY(n)` in SQL Server depending on size, and the appropriate `BLOB` tier in MySQL; (2) **prefer external storage for very large objects**—`BYTEA` is not well suited for very large binary data because processing a 1 GB value requires huge memory, and PostgreSQL's Large Object facility or external object storage is more appropriate; (3) **keep binary values small enough for efficient processing**—values under 1 MB are recommended for adequate performance, and above that threshold write amplification and other considerations may cause significant degradation; (4) **split large BLOBs into a separate table** and reference them with a join to reduce memory requirements for queries that do not use the BLOB column; (5) **compress large textual BLOBs before storing** them; (6) **index BLOB columns only with prefix lengths** and only when necessary; (7) **use streaming APIs** (`getBinaryStream`/`setBinaryStream`) rather than loading the entire value into memory; and (8) **choose the hex format** for `BYTEA` input/output for speed and compatibility.

**Beginner-Friendly Explanation**
Best practices for binary data are about avoiding memory blowups and performance cliffs. Do not stuff gigabytes of video into a `BYTEA` column—use a file server or object storage for that. Keep binary values small, stream them instead of loading them whole, and put large BLOBs in a separate table so queries that do not need them stay fast. Use the hex format for `BYTEA` because it is faster and more compatible than the old escape format.

#### Purposes

- To avoid memory exhaustion and query slowdowns caused by very large binary values.
- To keep tables compact and queries efficient when binary data is not always needed.
- To ensure that binary data is backed up, replicated, and restored correctly.
- To choose the most appropriate storage mechanism (inline, off-row, or external) for each use case.

#### Syntax Rules and Structure

Best practices are design and implementation guidelines rather than a single syntax:

| Practice | Implementation |
|----------|----------------|
| Size-tier selection | `TINYBLOB`/`BLOB`/`MEDIUMBLOB`/`LONGBLOB` (MySQL); `VARBINARY(n)` vs. `VARBINARY(MAX)` (SQL Server) |
| External storage | Use object storage (S3, GCS) or file system for very large files; store only metadata and a reference in the database |
| Separate BLOB table | `CREATE TABLE file_contents (file_id int PRIMARY KEY, data bytea);` and join only when needed |
| Compression | Compress text BLOBs with `compress()` before storage |
| Streaming | `setBinaryStream()` / `getBinaryStream()` (JDBC); `BYTEA` binary protocol in libpq |
| Hex format | `SET bytea_output = 'hex';` (default in PostgreSQL 9.0+) |
| Index prefix | `INDEX (blob_col(100))` for MySQL `BLOB`/`TEXT` columns |

**Constraints and limitations:**

- Storing very large files in the database increases backup size and restore time.
- External storage introduces a separate system that must be backed up and kept consistent with the database.
- Off-row storage of large values can cause additional I/O for queries that retrieve the BLOB column.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Separate table for large BLOBs

```sql
-- Step 1: Create a metadata table and a separate BLOB content table.
CREATE TABLE documents (
    doc_id   serial PRIMARY KEY,
    doc_name text NOT NULL,
    mime_type text NOT NULL
);

CREATE TABLE document_contents (
    doc_id int PRIMARY KEY REFERENCES documents(doc_id),
    data   bytea NOT NULL
);

-- Step 2: Insert metadata and content separately.
INSERT INTO documents (doc_name, mime_type) VALUES ('report.pdf', 'application/pdf') RETURNING doc_id;

INSERT INTO document_contents (doc_id, data) VALUES (1, '\x25504446...');  -- PDF content

-- Step 3: Query metadata without touching the large BLOB.
SELECT doc_id, doc_name, mime_type FROM documents;
```

**Expected output:**

```text
CREATE TABLE
CREATE TABLE
 doc_id | doc_name  | mime_type
--------+-----------+-------------
      1 | report.pdf| application/pdf
(1 row)
```

**Why this output appears**: The metadata query retrieves only the lightweight columns from `documents`; the large `bytea` value in `document_contents` is not read, reducing memory and I/O for queries that do not need the file contents.

##### Example 2: MySQL – Prefix indexing on a BLOB column

```sql
-- Step 1: Create a table with a BLOB column and a prefix index.
CREATE TABLE files (
    file_id  INT AUTO_INCREMENT PRIMARY KEY,
    file_name VARCHAR(255) NOT NULL,
    content   BLOB NOT NULL,
    INDEX idx_content_prefix (content(100))
);

-- Step 2: Query using the prefix index.
SELECT file_id, file_name FROM files WHERE content LIKE X'ffd8%';
```

**Expected output:**

```text
(rows matching the JPEG header prefix)
```

**Why this output appears**: The prefix index on the first 100 bytes of `content` allows MySQL to narrow the search before examining the full BLOB values, reducing I/O for prefix-based queries.

#### Real-World Cases

- **Image hosting**: A `product_images` table stores only metadata and a URL to an object store; the image bytes live in S3 or a CDN, keeping the database small and fast.
- **Document management**: A `documents` table stores metadata, and a separate `document_contents` table stores the large binary content, which is retrieved only when a user downloads the file.
- **Encryption key management**: Small encryption keys are stored in `BLOB` or `BYTEA` columns because they are tiny, security-sensitive, and benefit from database-level access controls and transactional consistency.

---

## References

- PostgreSQL Documentation – Binary Data Types – https://www.postgresql.org/docs/current/datatype-binary.html
- PostgreSQL Documentation – Binary String Functions and Operators – https://www.postgresql.org/docs/current/functions-binarystring.html
- PostgreSQL JDBC Documentation – Storing Binary Data – https://jdbc.postgresql.org/documentation/binary-data/
- MySQL 8.0 Reference Manual – The BLOB and TEXT Types – https://dev.mysql.com/doc/refman/8.0/en/blob.html
- MySQL 8.0 Reference Manual – Data Type Storage Requirements – https://dev.mysql.com/doc/refman/8.0/en/storage-requirements.html
- MySQL Reference Manual – TEXT and BLOB Types (8.2) – https://downloads.mysql.com/docs/refman-8.2-en.a4.pdf
- Microsoft Learn – binary and varbinary (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/data-types/binary-and-varbinary-transact-sql
- Microsoft Learn – SQL Server 2008 and Unstructured Data (VARBINARY(MAX)) – https://learn.microsoft.com/en-us/previous-versions/technet-magazine/dd424925(v=msdn.10)
- OneUptime Blog – How to Use BLOB Data Types (TINYBLOB, BLOB, MEDIUMBLOB, LONGBLOB) in MySQL – https://github.com/OneUptime/blog/blob/master/posts/2026-03-31-mysql-blob-data-types/README.md
- Stack Overflow – MySQL TEXT vs BLOB vs CLOB – https://stackoverflow.com/questions/7056004/mysql-text-vs-blob-vs-clob
- Timescale – Handling Large Objects in Postgres – https://www.timescale.com/blog/handling-large-objects-in-postgres/
- Oracle Docs – Optimizing for BLOB Types – https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/optimizing-blob-types.html