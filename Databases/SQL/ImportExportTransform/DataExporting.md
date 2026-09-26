# SQL Data Export: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL data export is the process of extracting data from a relational database system into external formats—files, streams, or other databases—for purposes including backup, migration, reporting, and integration with external systems.

**Technical Definition:** SQL data export encompasses a set of techniques—including cursor-driven streaming, file-format serialization (CSV, JSON), logical dump generation (`pg_dump`, `mysqldump`), and physical block-level backup—that transfer database contents out of the database engine while managing memory consumption, transactional consistency, and output format fidelity.

**Beginner-Friendly Explanation:** SQL data export is how you get information out of a database and into a format you can use elsewhere—like saving a spreadsheet as CSV to open in Excel, or making a complete copy of a database so you can move it to a new server.

### Key Characteristics

- **Directional:** Operates in the outbound direction (database → external target).
- **Format-flexible:** Supports CSV, JSON, SQL text, binary archives, and physical file copies.
- **Memory-sensitive:** Large exports require streaming techniques to avoid exhausting client or server memory.
- **Consistency-aware:** Must respect transaction isolation to produce coherent snapshots.
- **Portability-dependent:** Logical exports are portable across versions and platforms; physical exports are version-locked.

### Prerequisites

- Basic understanding of SQL `SELECT` statements and result sets.
- Familiarity with the target database's export utilities and permissions model.
- Knowledge of file system permissions (for file-based exports).
- Understanding of transaction isolation concepts.
- Awareness of character encoding (UTF-8, Latin-1) for text-based exports.

### Related Programming Areas

- Data engineering and ETL pipelines.
- Database administration and backup/recovery.
- API development (serving data as JSON).
- Business intelligence and reporting.
- DevOps and infrastructure automation.

### Core Concepts / Features

1. **Query Result Export** (streaming cursors vs. buffered queries)
2. **CSV Export** (text qualifiers, delimiter escaping, UTF-8 encoding)
3. **JSON Export** (native JSON aggregation functions)
4. **SQL Dump** (logical schema and row state scripting)
5. **Database Backup Export** (physical vs. logical exports, block-level copying)


## Core Concept 1: Query Result Export

### Definitions

**Core Definition:** Query result export is the process of retrieving rows returned by a SQL query and writing them to an external destination, using either streaming (cursor-driven) or buffered (full-result-set-in-memory) approaches.

**Technical Definition:** Streaming cursor-driven export uses server-side cursors (also called portals or named cursors) that maintain query state on the database server and return rows incrementally in configurable batch sizes, keeping client-side memory consumption constant regardless of result set size. Buffered export retrieves the entire result set into client memory in a single operation before writing begins.

**Beginner-Friendly Explanation:** Imagine reading a long book. Buffered export is like photocopying the entire book before you start reading—it takes a lot of paper (memory) and you cannot start until everything is copied. Streaming export is like reading page by page—you only hold one page at a time, so you never run out of space, even for a very long book.

### Purposes

- To export arbitrarily large result sets without exhausting client or server memory.
- To begin delivering export data immediately without waiting for the entire query to complete.
- To enable incremental processing of query results in application code.
- To support HTTP response streaming for web APIs that serve large datasets.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL Server-Side Cursor)

```sql
BEGIN;
DECLARE export_cursor CURSOR FOR
    SELECT * FROM large_table ORDER BY id;
FETCH FORWARD 1000 FROM export_cursor;
-- ... process rows ...
FETCH FORWARD 1000 FROM export_cursor;
-- ... repeat until no rows ...
CLOSE export_cursor;
COMMIT;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `BEGIN` | Starts a transaction (required for cursors). |
| `DECLARE ... CURSOR FOR` | Defines a named cursor bound to a query. |
| `FETCH FORWARD n` | Retrieves the next `n` rows from the cursor. |
| `CLOSE` | Releases the cursor and its server-side resources. |
| `COMMIT` | Ends the transaction and releases the snapshot. |

#### Complete General Syntax (PostgreSQL `COPY TO STDOUT`)

```sql
COPY (SELECT * FROM large_table)
TO STDOUT
WITH (FORMAT csv, HEADER true);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `COPY (query)` | Subquery whose results are exported. |
| `TO STDOUT` | Streams output to the client connection. |
| `FORMAT csv` | Output format. |
| `HEADER true` | Includes column names as the first line. |

#### Syntax Rules

- Server-side cursors require an explicit transaction block (`BEGIN` … `COMMIT`).
- `COPY TO STDOUT` is the fastest export mechanism in PostgreSQL; it bypasses SQL-level row processing.
- `COPY TO` can export to a server-side file (requires superuser or `pg_write_server_files` role) or to `STDOUT` (client-side).
- In MySQL, use `SELECT ... INTO OUTFILE` for server-side file export or `mysql --batch` for client-side streaming.
- JDBC `setFetchSize()` enables cursor-based streaming; without it, the driver buffers the entire result set.

#### Constraints and Limitations

- **PostgreSQL:** `COPY TO` cannot be used inside a transaction block that has already performed certain operations; `COPY TO STDOUT` must be the only statement in its transaction context.
- **MySQL:** `SELECT INTO OUTFILE` requires `FILE` privilege and writes only to server-side paths allowed by `secure_file_priv`.
- **JDBC:** Some drivers ignore `setFetchSize()` unless auto-commit is disabled.
- **Cursor overhead:** Server-side cursors consume server memory and hold a transaction snapshot open, preventing vacuum from cleaning up dead tuples.
- **Version-specific:** PostgreSQL 19 introduces `COPY ... TO ... WITH (FORMAT json)` for direct JSON export.

### Annotated Code Examples

#### Example 1: PostgreSQL `COPY TO STDOUT` with CSV

```bash
psql -U postgres -d myapp -c "
COPY (SELECT id, name, email FROM users ORDER BY id)
TO STDOUT
WITH (FORMAT csv, HEADER true, DELIMITER ',');
" > /tmp/users.csv
```

**Expected Output (file `/tmp/users.csv`):**

```
id,name,email
1,Alice,alice@example.com
2,Bob,bob@example.com
3,Carol,carol@example.com
```

**Why This Works:** `COPY TO STDOUT` streams rows directly from the database to the `psql` client process, which redirects them to the file. Memory usage on both the server and client remains constant because rows are transmitted in chunks rather than buffered in their entirety.

#### Example 2: Python psycopg2 Server-Side Cursor

```python
import psycopg2
import csv

conn = psycopg2.connect("dbname=myapp user=postgres")
# Server-side cursor: specify a name and itersize
cur = conn.cursor(name='export_cursor')
cur.itersize = 1000  # Fetch 1000 rows per network round-trip

cur.execute("SELECT id, name, email FROM users ORDER BY id")

with open('/tmp/users_stream.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(['id', 'name', 'email'])  # Header
    for row in cur:
        writer.writerow(row)

cur.close()
conn.close()
print("Export complete.")
```

**Expected Output:**

```
Export complete.
```

**Why This Works:** The `cursor(name='export_cursor')` call instructs psycopg2 to use a PostgreSQL server-side cursor. The `itersize` attribute controls how many rows are fetched per round-trip. The `for row in cur` loop processes rows one at a time, so client memory usage remains constant regardless of table size.

#### Example 3: Java JDBC Streaming Export with `setFetchSize`

```java
import java.sql.*;

public class StreamingExport {
    public static void main(String[] args) throws SQLException {
        String url = "jdbc:postgresql://localhost:5432/myapp";

        try (Connection conn = DriverManager.getConnection(url, "postgres", "pass")) 
        {
            conn.setAutoCommit(false); // Required for fetch size to take effect

            try (Statement stmt = conn.createStatement()) 
            {
                stmt.setFetchSize(5000); // Stream 5000 rows per round-trip
                
                try (ResultSet rs = stmt.executeQuery("SELECT id, name FROM users ORDER BY id")) 
                {
                    while (rs.next()) {
                        System.out.println(rs.getInt("id") + "," + rs.getString("name"));
                    }
                }
            }
            conn.commit();
        }
    }
}
```

**Expected Output (partial, first 3 rows):**

```
1,Alice
2,Bob
3,Carol
```

**Why This Works:** `setFetchSize(5000)` tells the JDBC driver to fetch 5,000 rows at a time from the server. Combined with `setAutoCommit(false)`, this enables cursor-based streaming instead of buffering the entire result set. Memory usage remains bounded by the fetch size.

### Real-World Cases

- **API data export:** A REST endpoint that streams a user's complete transaction history as CSV without loading it all into memory.
- **Data warehouse extraction:** Nightly extraction of millions of rows from an OLTP database into a staging area.
- **Report generation:** Generating a monthly report over a large fact table with constant memory usage.
- **Database migration:** Exporting a table to a CSV file for import into a different database system.

### References

- PostgreSQL Documentation: COPY — https://www.postgresql.org/docs/current/sql-copy.html
- PostgreSQL Documentation: DECLARE CURSOR — https://www.postgresql.org/docs/current/sql-declare.html
- psycopg2 Documentation: Server-side Cursors — https://www.psycopg.org/docs/usage.html#server-side-cursors
- MySQL: SELECT ... INTO OUTFILE — https://dev.mysql.com/doc/refman/8.0/en/select-into.html


## Core Concept 2: CSV Export

### Definitions

**Core Definition:** CSV export is the process of writing query results or table data to a file in Comma-Separated Values format, with configurable delimiters, quoting rules, and character encoding.

**Technical Definition:** CSV export serializes relational rows into a text format where each row is a line, each column is separated by a delimiter (typically a comma), and values containing delimiters, quotes, or newlines are enclosed in quote characters (typically double quotes) with internal quotes escaped by doubling.

**Beginner-Friendly Explanation:** CSV export turns a database table into a plain text file that opens in any spreadsheet program. Each row becomes a line, and each column is separated by a comma. If a value contains a comma, it gets wrapped in quotes so the spreadsheet knows it is part of the value.

### Purposes

- To produce human-readable and spreadsheet-compatible exports of tabular data.
- To handle text qualifiers (quoting) correctly when values contain delimiters, quotes, or newlines.
- To control character encoding (UTF-8, Latin-1) for cross-platform compatibility.
- To exchange data with external systems that consume CSV (finance, CRM, analytics).

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL COPY TO CSV)

```sql
COPY table_name [(column_list)]
TO 'file_path'
WITH (
    FORMAT csv,
    HEADER true,
    DELIMITER ',',
    QUOTE '"',
    ESCAPE '"',
    NULL '',
    ENCODING 'UTF8'
);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `table_name` | Source table. |
| `(column_list)` | Optional column selection. |
| `TO 'file_path'` | Server-side output file, or `STDOUT`. |
| `FORMAT csv` | Enables CSV escaping rules. |
| `HEADER true` | Writes column names as the first line. |
| `DELIMITER ','` | Column separator. |
| `QUOTE '"'` | Character used to enclose values containing special characters. |
| `ESCAPE '"'` | Character used to escape the quote character inside quoted values. |
| `NULL ''` | String representing NULL. |
| `ENCODING 'UTF8'` | Output file encoding. |

#### Complete General Syntax (MySQL SELECT INTO OUTFILE)

```sql
SELECT column1, column2, column3
INTO OUTFILE '/path/to/file.csv'
FIELDS TERMINATED BY ','
    OPTIONALLY ENCLOSED BY '"'
    ESCAPED BY '"'
LINES TERMINATED BY '\n'
FROM table_name
WHERE condition;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `SELECT ... INTO OUTFILE` | Writes query results to a server-side file. |
| `FIELDS TERMINATED BY` | Column delimiter. |
| `OPTIONALLY ENCLOSED BY` | Quote character for values containing delimiters. |
| `ESCAPED BY` | Escape character for the quote character. |
| `LINES TERMINATED BY` | Row separator. |

#### Syntax Rules

- **CSV quoting:** If a value contains the delimiter, the quote character, the NULL string, a carriage return, or a line feed, the whole value is prefixed and suffixed by the quote character, and any occurrence of the quote character or escape character within the value is preceded by the escape character.
- **NULL vs. empty string:** In PostgreSQL CSV, NULL is written as the unquoted NULL string (default: empty), while a non-NULL empty string is written as `""` (quoted empty string).
- **Encoding:** Always specify `ENCODING 'UTF8'` to ensure cross-platform compatibility; mismatched encoding causes character corruption.
- **MySQL `secure_file_priv`:** Restricts the directories into which `SELECT INTO OUTFILE` can write.
- **PostgreSQL server-side write:** Requires superuser or membership in `pg_write_server_files`.

#### Constraints and Limitations

- **CSV has no standard:** Many programs produce "strange and occasionally perverse CSV files"; some files cannot be imported by PostgreSQL's `COPY`, and `COPY` may produce files other programs cannot process.
- **All characters are significant:** In CSV format, trailing white space is preserved; files padded to fixed width must be preprocessed.
- **MySQL `SELECT INTO OUTFILE`:** Cannot overwrite existing files; the file must not already exist.
- **PostgreSQL `COPY TO`:** Does not support `WHERE` on `COPY table` (without subquery) in versions before PostgreSQL 12.
- **Version-specific:** PostgreSQL 18 and later recognize `\.` as an end-of-data marker only in `psql` in-line COPY; earlier versions treated it specially in files as well.

### Annotated Code Examples

#### Example 1: PostgreSQL `COPY TO` with Full CSV Options

```sql
-- Create sample table
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    description TEXT,
    price NUMERIC(10,2)
);

INSERT INTO products (name, description, price) VALUES
('Widget', 'A "great" widget, 10cm', 19.99),
('Gadget', 'Multi-line
description', 29.99),
('Empty', '', 0.00),
('NullDesc', NULL, 5.00);

-- Export with explicit CSV options
COPY products
TO '/tmp/products.csv'
WITH (
    FORMAT csv,
    HEADER true,
    DELIMITER ',',
    QUOTE '"',
    ESCAPE '"',
    NULL 'NULL',
    ENCODING 'UTF8'
);
```

**Expected Output (file `/tmp/products.csv`):**

```
id,name,description,price
1,Widget,"A ""great"" widget, 10cm",19.99
2,Gadget,"Multi-line
description",29.99
3,Empty,"",0.00
4,NullDesc,NULL,5.00
```

**Why This Works:** The description containing commas and quotes is enclosed in double quotes, and internal quotes are doubled (`""`). The multi-line description is enclosed in quotes so the newline is preserved within the value. The empty string is written as `""`, while NULL is written as `NULL` (the configured NULL string).

#### Example 2: MySQL `SELECT INTO OUTFILE`

```sql
-- Ensure secure_file_priv allows the target directory
-- SHOW VARIABLES LIKE 'secure_file_priv';  -- e.g., /var/lib/mysql-files/

SELECT id, name, email, signup_date
INTO OUTFILE '/var/lib/mysql-files/users.csv'
FIELDS TERMINATED BY ','
    OPTIONALLY ENCLOSED BY '"'
    ESCAPED BY '"'
LINES TERMINATED BY '\n'
FROM users
WHERE signup_date >= '2024-01-01';
```

**Expected Output (file `/var/lib/mysql-files/users.csv`):**

```
1,Alice,alice@example.com,2024-01-15
2,Bob,bob@example.com,2024-02-20
3,Carol,carol@example.com,2024-03-10
```

**Why This Works:** `SELECT INTO OUTFILE` writes query results directly to a server-side file. The `OPTIONALLY ENCLOSED BY '"'` ensures values containing commas are quoted. The `ESCAPED BY '"'` doubles internal quotes. The file is created with the MySQL server process's permissions.

#### Example 3: Python Streaming CSV Export with `copy_expert`

```python
import psycopg2
import io

conn = psycopg2.connect("dbname=myapp user=postgres")

# Use copy_expert to stream COPY output directly to a file object
with open('/tmp/users_export.csv', 'w', encoding='utf-8') as f:
    cur = conn.cursor()
    cur.copy_expert(
        "COPY (SELECT id, name, email FROM users ORDER BY id) "
        "TO STDOUT WITH (FORMAT csv, HEADER true, ENCODING 'UTF8')",
        f
    )
    cur.close()

conn.close()
print("CSV export complete.")
```

**Expected Output:**

```
CSV export complete.
```

**Why This Works:** `copy_expert` executes a `COPY TO STDOUT` command and streams the output directly to the file object `f`. Memory usage remains constant because data is transferred in chunks. This is the most efficient way to export large tables to CSV from Python.

### Real-World Cases

- **Financial reporting:** Exporting transaction records to CSV for import into accounting software.
- **Data interchange:** Sending customer lists to a marketing automation platform that accepts CSV uploads.
- **Spreadsheet analysis:** Exporting query results to CSV for ad-hoc analysis in Excel or Google Sheets.
- **Legacy system integration:** Producing CSV files for mainframe systems that require fixed-format delimited input.

### References

- PostgreSQL Documentation: COPY (CSV Format) — https://www.postgresql.org/docs/current/sql-copy.html
- MySQL Reference Manual: SELECT ... INTO OUTFILE — https://dev.mysql.com/doc/refman/8.0/en/select-into.html
- RFC 4180: Common Format and MIME Type for CSV Files — https://www.rfc-editor.org/rfc/rfc4180
- psycopg2 Documentation: `copy_expert` — https://www.psycopg.org/docs/cursor.html#cursor.copy_expert


## Core Concept 3: JSON Export

### Definitions

**Core Definition:** JSON export is the process of serializing relational query results into JavaScript Object Notation documents, using database-native JSON aggregation functions to build nested structures that reflect relational joins.

**Technical Definition:** JSON export leverages functions such as PostgreSQL's `json_agg`, `json_build_object`, and `row_to_json` (or MySQL's `JSON_ARRAYAGG` and `JSON_OBJECT`) to transform normalized rows into denormalized JSON documents, where one-to-many relationships are represented as nested arrays within a parent object.

**Beginner-Friendly Explanation:** JSON export turns database rows into JSON—the format used by web APIs and modern applications. Instead of exporting flat tables, you can build nested documents: an order with all its line items inside it, rather than two separate tables.

### Purposes

- To produce JSON documents suitable for consumption by web APIs and JavaScript applications.
- To denormalize one-to-many relationships into nested arrays within a single document.
- To export data for migration to document-oriented databases (MongoDB, CouchDB).
- To generate API responses directly from SQL queries without application-side serialization.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL JSON Aggregation)

```sql
SELECT
    json_agg(
        json_build_object(
            'key1', column1,
            'key2', column2,
            'nested_array', (
                SELECT json_agg(json_build_object('sub_key', sub_column))
                FROM child_table
                WHERE child_table.parent_id = parent_table.id
            )
        )
    ) AS result
FROM parent_table
WHERE condition;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `json_agg(expr)` | Aggregates rows into a JSON array. |
| `json_build_object(k1, v1, ...)` | Constructs a JSON object from key-value pairs. |
| Subquery with `json_agg` | Builds a nested array for a one-to-many relationship. |
| `row_to_json(table_alias)` | Converts an entire row to a JSON object. |

#### Complete General Syntax (MySQL JSON Aggregation)

```sql
SELECT
    JSON_ARRAYAGG(
        JSON_OBJECT(
            'key1', column1,
            'key2', column2
        )
    ) AS result
FROM table_name;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `JSON_ARRAYAGG(expr)` | Aggregates rows into a JSON array (MySQL 5.7.22+). |
| `JSON_OBJECT(k, v, ...)` | Builds a JSON object. |

#### Syntax Rules

- `json_agg` and `json_build_object` return `json` type; use `jsonb_agg` and `jsonb_build_object` for binary JSON (indexable, more efficient).
- `row_to_json` converts a table row to a JSON object using column names as keys.
- Subqueries with `json_agg` inside `json_build_object` cannot be correlated in all contexts; use a lateral join or a subquery in the `SELECT` list.
- `COPY (SELECT json_agg(...) FROM ...) TO STDOUT` produces a single JSON array as output.
- MySQL's `JSON_ARRAYAGG` does not support `ORDER BY` inside the aggregate before MySQL 8.0.

#### Constraints and Limitations

- **Performance:** Nested `json_agg` subqueries can be slow on large datasets because they execute per parent row. Consider using `LATERAL` joins or application-side assembly for very large exports.
- **NULL handling:** `json_agg` includes NULL values as JSON `null`; use `FILTER (WHERE ... IS NOT NULL)` to exclude them.
- **Version-specific:** `json_build_object` is available in PostgreSQL 9.4+; `jsonb_build_object` in 9.4+. MySQL `JSON_ARRAYAGG` requires 5.7.22+.
- **Output size:** A single JSON document containing millions of rows may exceed client memory limits; stream using `COPY TO STDOUT` with `row_to_json` for JSON Lines format.

### Annotated Code Examples

#### Example 1: PostgreSQL Nested JSON Export with `json_agg`

```sql
-- Create normalized schema
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name TEXT,
    email TEXT
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(id),
    order_date DATE,
    total NUMERIC(10,2)
);

INSERT INTO customers (name, email) VALUES
('Alice', 'alice@example.com'),
('Bob', 'bob@example.com');

INSERT INTO orders (customer_id, order_date, total) VALUES
(1, '2024-01-15', 150.00),
(1, '2024-02-20', 75.50),
(2, '2024-03-10', 200.00);

-- Export customers with nested orders
SELECT json_agg(
    json_build_object(
        'customer_id', c.id,
        'name', c.name,
        'email', c.email,
        'orders', (
            SELECT json_agg(
                json_build_object(
                    'order_id', o.id,
                    'date', o.order_date,
                    'total', o.total
                )
            )
            FROM orders o
            WHERE o.customer_id = c.id
        )
    )
) AS customers_json
FROM customers c;
```

**Expected Output:**

```json
[
  {
    "customer_id": 1,
    "name": "Alice",
    "email": "alice@example.com",
    "orders": [
      {"order_id": 1, "date": "2024-01-15", "total": 150.00},
      {"order_id": 2, "date": "2024-02-20", "total": 75.50}
    ]
  },
  {
    "customer_id": 2,
    "name": "Bob",
    "email": "bob@example.com",
    "orders": [
      {"order_id": 3, "date": "2024-03-10", "total": 200.00}
    ]
  }
]
```

**Why This Works:** The outer `json_agg` aggregates customer rows into an array. For each customer, `json_build_object` constructs the customer object, and a correlated subquery with `json_agg` builds the nested `orders` array. This denormalizes the one-to-many relationship into a single JSON document.

#### Example 2: JSON Lines Export with `row_to_json`

```sql
COPY (
    SELECT row_to_json(u)
    FROM (
        SELECT id, name, email, created_at AS "createdAt"
        FROM users
        ORDER BY id
    ) u
) TO STDOUT;
```

**Expected Output (file `/tmp/users.jsonl`):**

```
{"id":1,"name":"Alice","email":"alice@example.com","createdAt":"2024-01-01T10:00:00"}
{"id":2,"name":"Bob","email":"bob@example.com","createdAt":"2024-01-02T11:30:00"}
{"id":3,"name":"Carol","email":"carol@example.com","createdAt":"2024-01-03T09:15:00"}
```

**Why This Works:** `row_to_json` converts each row into a JSON object. Since `COPY TO STDOUT` writes one row per line, the output is JSON Lines format (one JSON document per line). This is memory-efficient and streamable, unlike a single large JSON array.

#### Example 3: MySQL JSON Export

```sql
SELECT
    JSON_ARRAYAGG(
        JSON_OBJECT(
            'id', id,
            'name', name,
            'email', email,
            'orders', (
                SELECT JSON_ARRAYAGG(
                    JSON_OBJECT('order_id', o.id, 'total', o.total)
                )
                FROM orders o
                WHERE o.customer_id = customers.id
            )
        )
    ) AS customers_json
FROM customers;
```

**Expected Output:**

```json
[
  {
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com",
    "orders": [
      {"order_id": 1, "total": 150.00},
      {"order_id": 2, "total": 75.50}
    ]
  },
  {
    "id": 2,
    "name": "Bob",
    "email": "bob@example.com",
    "orders": [
      {"order_id": 3, "total": 200.00}
    ]
  }
]
```

**Why This Works:** MySQL's `JSON_ARRAYAGG` and `JSON_OBJECT` functions provide equivalent functionality to PostgreSQL's `json_agg` and `json_build_object`. The correlated subquery builds nested arrays for the one-to-many relationship.

### Real-World Cases

- **API response generation:** Producing JSON directly from SQL for REST endpoints, avoiding application-side serialization.
- **NoSQL migration:** Exporting relational data as nested JSON documents for import into MongoDB or CouchDB.
- **Configuration export:** Exporting application settings stored in relational tables as JSON configuration files.
- **Data interchange:** Sending structured data to JavaScript frontends or third-party APIs that consume JSON.

### References

- PostgreSQL Documentation: JSON Functions and Operators — https://www.postgresql.org/docs/current/functions-json.html
- PostgreSQL Documentation: Aggregate Functions (`json_agg`) — https://www.postgresql.org/docs/current/functions-aggregate.html
- MySQL Reference Manual: JSON Functions — https://dev.mysql.com/doc/refman/8.0/en/json-functions.html
- JSON Lines Specification — https://jsonlines.org/


## Core Concept 4: SQL Dump

### Definitions

**Core Definition:** SQL dump is a logical export of a database's schema and data as a sequence of SQL statements (`CREATE TABLE`, `INSERT`, `COPY`) that can be replayed to recreate the database.

**Technical Definition:** A SQL dump is a logical backup produced by utilities such as PostgreSQL's `pg_dump` or MySQL's `mysqldump`, which generate a text or binary archive containing the data definition language (DDL) for schema objects and the data manipulation language (DML) or `COPY` statements for table contents. Unlike physical backups, SQL dumps are portable across database versions and architectures.

**Beginner-Friendly Explanation:** A SQL dump is like a recipe for rebuilding your database from scratch. It contains all the instructions—"create this table, add this row, add that row"—so you can reconstruct the database anywhere.

### Purposes

- To create portable, version-independent backups of database schema and data.
- To migrate a database from one server, version, or architecture to another.
- To generate SQL text representations of schema for version control and auditing.
- To selectively restore individual tables or schemas from a backup.

### Syntax Rules and Structure

#### Complete General Syntax (`pg_dump`)

```bash
pg_dump [connection-option] [option] dbname
```

**Common options:**

| Option | Description |
|--------|-------------|
| `-Fc` | Custom format (compressed, supports selective restore). |
| `-Fd` | Directory format (supports parallel dump/restore). |
| `-Fp` | Plain SQL text format. |
| `-Ft` | Tar format. |
| `-j N` | Parallel dump with N jobs (directory format only). |
| `--schema-only` | Dump only schema (no data). |
| `--data-only` | Dump only data (no schema). |
| `-n schema` | Dump only the named schema. |
| `-t table` | Dump only the named table. |
| `-Z 0-9` | Compression level. |

#### Complete General Syntax (`mysqldump`)

```bash
mysqldump [options] db_name [tbl_name ...]
```

**Common options:**

| Option | Description |
|--------|-------------|
| `--single-transaction` | Consistent snapshot without locking (InnoDB). |
| `--routines` | Include stored procedures and functions. |
| `--triggers` | Include triggers. |
| `--events` | Include scheduled events. |
| `--all-databases` | Dump all databases. |
| `--no-data` | Schema only. |
| `--no-create-info` | Data only. |

#### Syntax Rules

- `pg_dump` dumps a **single database**; `pg_dumpall` dumps the entire cluster (roles, tablespaces, all databases).
- Custom (`-Fc`) and directory (`-Fd`) formats are the production defaults; plain (`-Fp`) cannot be selectively restored with `pg_restore`.
- The directory format is the **only** format that supports parallel dump (`-j N`); both custom and directory formats support parallel restore via `pg_restore -j`.
- `pg_dump` produces a consistent snapshot using MVCC without requiring downtime.
- `mysqldump --single-transaction` provides a consistent snapshot for InnoDB tables without locking; it requires `RELOAD` or `FLUSH_TABLES` privilege when GTID mode is enabled.

#### Constraints and Limitations

- **Materialized views:** `pg_dump` dumps only the schema of materialized views, not their data; `REFRESH MATERIALIZED VIEW` is required after restore.
- **Large objects:** Large objects are dumped separately; ensure `-b` is included if needed.
- **Version compatibility:** `pg_dump` from a newer version can usually dump an older server, but not vice versa.
- **MySQL `mysqldump`:** Buffering in memory can be an issue for very large tables; use `--quick` to stream rows.
- **Restore requires a running server:** Logical dumps cannot be simply copied into place like physical backups.
- **Version-specific:** PostgreSQL 18+ supports `--statistics-only` for dumping planner statistics.

### Annotated Code Examples

#### Example 1: `pg_dump` Custom Format with Compression

```bash
# Dump the 'myapp' database in custom format with compression
pg_dump -U postgres -Fc -Z 6 -f /backups/myapp.dump myapp

# List contents of the dump
pg_restore -l /backups/myapp.dump
```

**Expected Output (from `pg_restore -l`):**

```
;
; Archive created at 2024-06-15 10:30:00 UTC
;     dbname: myapp
;     TOC Entries: 45
;     Compression: -1
;     Dump Version: 1.14-0
;     Format: CUSTOM
;
; Selected TOC Entries:
;
3; 2615 2200 SCHEMA - public postgres
215; 1259 16385 TABLE public users postgres
216; 1259 16386 TABLE public orders postgres
...
```

**Why This Works:** The custom format (`-Fc`) produces a compressed archive that supports selective restore with `pg_restore`. The `-Z 6` specifies compression level 6 (moderate). The `pg_restore -l` command lists the archive's table of contents, showing all objects that can be selectively restored.

#### Example 2: `mysqldump` with Consistent Snapshot

```bash
mysqldump -u root -p \
    --single-transaction \
    --routines \
    --triggers \
    --events \
    --databases myapp \
    > /backups/myapp_$(date +%Y%m%d).sql
```

**Expected Output (partial file content):**

```sql
-- MySQL dump 10.13  Distrib 8.0.36, for Linux (x86_64)
--
-- Host: localhost    Database: myapp
-- ------------------------------------------------------
-- Server version       8.0.36

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
...
CREATE TABLE `users` (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(100) DEFAULT NULL,
  `email` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

LOCK TABLES `users` WRITE;
INSERT INTO `users` VALUES (1,'Alice','alice@example.com'),(2,'Bob','bob@example.com');
UNLOCK TABLES;
```

**Why This Works:** `--single-transaction` uses InnoDB's MVCC to produce a consistent snapshot without locking tables. `--routines`, `--triggers`, and `--events` include stored programs. The output is a plain SQL file that can be replayed with the `mysql` client. The `LOCK TABLES` around `INSERT` is a standard mysqldump pattern for data consistency during restore.

#### Example 3: `pg_dump` Directory Format with Parallel Dump

```bash
# Parallel dump using directory format with 4 jobs
pg_dump -U postgres -Fd -j 4 -f /backups/myapp_dir myapp

# Restore in parallel with 4 jobs
pg_restore -U postgres -d myapp_restored -j 4 /backups/myapp_dir
```

**Expected Output (during dump):**

```
pg_dump: dumping contents of table "public.users"
pg_dump: dumping contents of table "public.orders"
pg_dump: dumping contents of table "public.products"
pg_dump: dumping contents of table "public.transactions"
```

**Why This Works:** The directory format (`-Fd`) writes each table to a separate file within the output directory, allowing `-j 4` to dump four tables concurrently. Parallel dump dramatically reduces wall-clock time for large databases. Parallel restore (`pg_restore -j 4`) reads from the directory concurrently.

### Real-World Cases

- **Database migration:** Moving a database from an on-premises server to a cloud-managed service.
- **Version upgrade:** Dumping from PostgreSQL 14 and restoring to PostgreSQL 17.
- **Development environment seeding:** Creating a dump of production schema (schema-only) for developer workstations.
- **Compliance archiving:** Generating periodic SQL dumps for long-term retention and audit.

### References

- PostgreSQL Documentation: `pg_dump` — https://www.postgresql.org/docs/current/app-pgdump.html
- PostgreSQL Documentation: `pg_dumpall` — https://www.postgresql.org/docs/current/app-pg-dumpall.html
- PostgreSQL Documentation: `pg_restore` — https://www.postgresql.org/docs/current/app-pgrestore.html
- MySQL Reference Manual: `mysqldump` — https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html
- MySQL Shell Dump Utility — https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-dump-instance-schema.html


## Core Concept 5: Database Backup Export

### Definitions

**Core Definition:** Database backup export is the process of creating a copy of a database—either as a logical dump or a physical file-level copy—for disaster recovery, replication, or migration purposes.

**Technical Definition:** Physical backup export uses tools such as PostgreSQL's `pg_basebackup` to copy the raw data directory (data files, WAL segments, configuration) directly from the server's storage or replication stream, producing a binary-identical copy of the database cluster that can be started immediately. Logical backup export (covered in Concept 4) produces SQL statements that must be replayed.

**Beginner-Friendly Explanation:** A physical backup is like photocopying every page of a book—you get an exact duplicate that you can immediately start reading. A logical backup is like writing down every sentence in the book—you can recreate the book, but it takes time to write it all down again.

### Purposes

- To create a directly startable copy of a database cluster for disaster recovery.
- To support point-in-time recovery (PITR) by combining base backups with WAL archiving.
- To initialize streaming replication standby servers.
- To provide fast, complete backups of large databases where logical dump time exceeds recovery time objectives.

### Syntax Rules and Structure

#### Complete General Syntax (`pg_basebackup`)

```bash
pg_basebackup [option] ... -D destination_directory
```

**Common options:**

| Option | Description |
|--------|-------------|
| `-D dir` | Target directory for the backup. |
| `-F t` | Tar format output. |
| `-F p` | Plain format (default). |
| `-X stream` | Stream WAL during backup (recommended). |
| `-X fetch` | Fetch WAL after backup completes. |
| `-P` | Show progress. |
| `-R` | Write recovery configuration for standby. |
| `-c fast` | Fast checkpoint mode. |
| `--slot=slotname` | Use a replication slot. |

#### Complete General Syntax (MySQL Physical Backup — Percona XtraBackup)

```bash
xtrabackup --backup --target-dir=/backups/mysql/
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `--backup` | Performs a physical backup of InnoDB data files. |
| `--target-dir` | Destination directory for the backup. |

#### Syntax Rules

- **`pg_basebackup`** requires a connection with `REPLICATION` privilege and the server must have `max_wal_senders` set high enough to accommodate the backup plus any streaming replication connections.
- **`-X stream`** is recommended for most HA/DR cases; use a replication slot to ensure the primary does not remove needed WAL segments.
- **Physical backups are version-locked:** The backup must be restored with the same major PostgreSQL version and platform architecture.
- **No selective restore:** Physical backups restore the entire cluster; you cannot extract a single table.
- **Consistency:** `pg_basebackup` uses the replication protocol, which guarantees a crash-consistent snapshot even on a busy server.

#### Constraints and Limitations

- **Version-locked:** Physical backups cannot be restored across major PostgreSQL versions.
- **No selective restore:** You restore the entire cluster, not individual tables.
- **Storage requirements:** Physical backups are larger than logical dumps because they include indexes, free space, and WAL segments.
- **Cloud managed services:** Many managed database services (Amazon RDS, Google Cloud SQL) do not expose the underlying data directory; use logical backups or the provider's snapshot mechanism.
- **MySQL:** `xtrabackup` requires the InnoDB storage engine; MyISAM tables are not backed up consistently.
- **Unsafe if misused:** A physical backup taken without proper WAL streaming may not be recoverable to a consistent state.

### Annotated Code Examples

#### Example 1: `pg_basebackup` with WAL Streaming

```bash
# Create a physical base backup with WAL streaming
pg_basebackup \
    -h primary.example.com \
    -U replicator \
    -D /backups/pg_base_$(date +%Y%m%d) \
    -F p \
    -X stream \
    -P \
    -R \
    -c fast
```

**Expected Output:**

```
pg_basebackup: initiating base backup, waiting for checkpoint to complete
pg_basebackup: checkpoint completed
pg_basebackup: write-ahead log start point: 0/2000028 on timeline 1
pg_basebackup: starting background WAL receiver
pg_basebackup: created temporary replication slot "pg_basebackup_12345"
    0/2100000 kB (100%), 1/1 tablespace
pg_basebackup: write-ahead log end point: 0/2100100
pg_basebackup: waiting for background process to finish streaming ...
pg_basebackup: syncing data to disk ...
pg_basebackup: base backup completed
```

**Why This Works:** `-X stream` streams WAL segments concurrently with the base backup, ensuring the backup includes all WAL needed for consistency. `-R` writes a `postgresql.auto.conf` and `standby.signal` file, making the backup immediately startable as a standby. `-c fast` forces an immediate checkpoint, reducing backup time.

#### Example 2: MySQL `xtrabackup` Physical Backup

```bash
# Full physical backup
xtrabackup --backup --target-dir=/backups/mysql/full/

# Prepare the backup for restore
xtrabackup --prepare --target-dir=/backups/mysql/full/

# Restore: stop MySQL, copy files back, start MySQL
systemctl stop mysqld
xtrabackup --copy-back --target-dir=/backups/mysql/full/
chown -R mysql:mysql /var/lib/mysql
systemctl start mysqld
```

**Expected Output (from `--prepare`):**

```
xtrabackup: Starting InnoDB instance for recovery.
xtrabackup: Using 104857600 bytes for buffer pool (set by --use-memory parameter)
...
InnoDB: Starting crash recovery.
InnoDB: Doing recovery: scanned up to log sequence number 1234567890
InnoDB: 128 rollback segment(s) are active.
xtrabackup: Transaction log of lsn (1234567890) to (1234567890) was copied.
xtrabackup: completed OK!
```

**Why This Works:** `xtrabackup` copies InnoDB data files while the server is running (hot backup). The `--prepare` step applies transaction logs to make the data files consistent. The `--copy-back` step restores the prepared files to the MySQL data directory. This is faster than mysqldump for large databases.

#### Example 3: Combining Physical Base Backup with WAL Archiving for PITR

```bash
# Step 1: Take a base backup
pg_basebackup -D /backups/pitr_base -F p -X fetch -c fast

# Step 2: Archive WAL segments continuously
# In postgresql.conf:
# archive_mode = on
# archive_command = 'cp %p /backups/wal_archive/%f'

# Step 3: To restore to a point in time:
# - Restore the base backup to a new data directory
# - Create recovery.signal
# - Configure restore_command to copy WAL from archive
# - Set recovery_target_time = '2024-06-15 14:30:00'
```

**Expected Output (during recovery):**

```
LOG:  starting point-in-time recovery to 2024-06-15 14:30:00+00
LOG:  restored log file "000000010000000000000002" from archive
LOG:  consistent recovery state reached at 0/2000100
LOG:  recovery stopping before commit of transaction 1234, time 2024-06-15 14:29:55
LOG:  redo done at 0/2000100
LOG:  last completed transaction was at log time 2024-06-15 14:29:50+00
LOG:  database system is ready to accept connections
```

**Why This Works:** The base backup provides a starting point, and archived WAL segments contain all changes since the backup. By specifying a `recovery_target_time`, PostgreSQL replays WAL up to (but not including) that timestamp, enabling precise point-in-time recovery.

### Real-World Cases

- **Disaster recovery:** Restoring a production database after hardware failure using a physical base backup and WAL archives.
- **Standby server provisioning:** Creating a streaming replication standby by taking a base backup and configuring `primary_conninfo`.
- **Major version upgrade:** Taking a physical backup before `pg_upgrade` to enable rollback.
- **Cloud migration:** Using `pg_basebackup` to seed a replica in a different cloud region for cross-region disaster recovery.

### References

- PostgreSQL Documentation: `pg_basebackup` — https://www.postgresql.org/docs/current/app-pgbasebackup.html
- PostgreSQL Documentation: Continuous Archiving and Point-in-Time Recovery — https://www.postgresql.org/docs/current/continuous-archiving.html
- PostgreSQL Documentation: Log-Shipping Standby Servers — https://www.postgresql.org/docs/current/warm-standby.html
- Percona XtraBackup Documentation — https://docs.percona.com/percona-xtrabackup/
- MySQL Reference Manual: Backup and Recovery — https://dev.mysql.com/doc/refman/8.0/en/backup-and-recovery.html


## Summary Table: Data Export Methods Comparison

| Method | Output Format | Memory Profile | Portability | Use Case |
|--------|--------------|----------------|-------------|----------|
| `COPY TO STDOUT` | CSV, text, binary | Constant (streaming) | High (CSV/text) | Large table exports |
| Server-side cursor | Application-defined | Constant (streaming) | N/A | Application-driven export |
| `SELECT INTO OUTFILE` | CSV, custom delimiter | Server-side (file) | High | MySQL server-side export |
| `json_agg` / `json_build_object` | JSON | Query-dependent | High | API responses, NoSQL migration |
| `pg_dump` (custom) | Binary archive | Low | High (cross-version) | Logical backup, selective restore |
| `pg_dump` (directory) | Directory of files | Low (parallel) | High | Large logical backup |
| `mysqldump` | SQL text | Low–moderate | High | MySQL logical backup |
| `pg_basebackup` | Physical files | Low (streaming) | Low (version-locked) | Disaster recovery, replication |
| `xtrabackup` | Physical files | Low | Low (version-locked) | MySQL physical backup |


## Final Notes on Deprecated and Unsafe Features

- **PostgreSQL `COPY ... OIDS`:** Deprecated; OIDs are not supported in modern PostgreSQL versions.
- **MySQL `SELECT INTO OUTFILE`:** Requires `FILE` privilege; often disabled in cloud environments. Cannot overwrite existing files.
- **`pg_dump` plain format (`-Fp`):** Cannot be selectively restored with `pg_restore`; use custom (`-Fc`) or directory (`-Fd`) for production use.
- **Physical backups without WAL streaming:** A `pg_basebackup` without `-X stream` or `-X fetch` may produce a backup that cannot be recovered to a consistent state without the appropriate WAL segments.
- **Materialized view data:** Not included in `pg_dump`; must be refreshed after restore.
- **Unsafe if misused:** Disabling transaction consistency (`--single-transaction` omitted in mysqldump) during production export can lead to inconsistent snapshots.
- **Version-specific:** PostgreSQL 19 introduces `COPY ... TO ... WITH (FORMAT json)` for direct JSON export; not available in earlier versions.