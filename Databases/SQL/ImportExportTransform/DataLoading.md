# SQL Data Loading: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL data loading is the process of inserting data into a relational database system, typically in large volumes, using specialized commands, utilities, or programmatic interfaces designed to maximize throughput and minimize overhead.

**Technical Definition:** SQL data loading encompasses a set of techniques—including bulk insert operations, batch processing, file-based import commands, foreign data wrappers, and command-line utilities—that transfer data from external sources (files, remote databases, application memory) into database tables while managing transactional integrity, constraint enforcement, and write-ahead logging (WAL) behavior.

**Beginner-Friendly Explanation:** SQL data loading is how you get lots of information into a database quickly. Instead of typing one row at a time, you use special tools that can pour thousands or millions of rows into a table in one go, like using a fire hose instead of a water bottle.

### Key Characteristics

- **Volume-oriented:** Designed for large datasets where row-by-row insertion is impractical.
- **Performance-sensitive:** Bypasses SQL parsing and planning overhead where possible.
- **Transactional control:** Often operates within explicit transaction boundaries to ensure atomicity.
- **Constraint management:** May require temporary disabling of indexes, foreign keys, and triggers.
- **Format-specific:** Supports CSV, binary, fixed-width, and native database formats.
- **Vendor-dependent:** Each RDBMS (PostgreSQL, MySQL, Oracle, SQL Server) provides its own syntax and utilities.

### Prerequisites

- Basic understanding of SQL `INSERT` statements and table schemas.
- Familiarity with the target database's connection and authentication mechanisms.
- Knowledge of file system permissions (for file-based imports).
- Understanding of transaction concepts (`BEGIN`, `COMMIT`, `ROLLBACK`).
- Awareness of database-specific configuration parameters (e.g., `wal_level`, `secure_file_priv`).

### Related Programming Areas

- ETL (Extract, Transform, Load) pipelines.
- Data warehousing and analytics.
- Database migration and replication.
- Application performance tuning.
- DevOps and database automation.

### Core Concepts / Features

The following core concepts are covered in this cheat sheet:

1. **Bulk Inserts** (native array parameters and multi-row constructs)
2. **Batch Inserts** (chunk sizes, network round-trips, memory allocation)
3. **CSV Import** (PostgreSQL `COPY`, MySQL `LOAD DATA INFILE`)
4. **External Data Loading** (Foreign Data Wrappers, external schemas, external storage endpoints)
5. **Bulk-Loading Utilities** (`pg_bulkload`, Oracle SQL*Loader, `bcp`)
6. **Transaction Isolation Management** (disabling constraints, dropping indexes, bypassing WAL)


## Core Concept 1: Bulk Inserts

### Definitions

**Core Definition:** Bulk insert is a database operation that inserts multiple rows of data in a single command or programmatic call, rather than issuing separate `INSERT` statements for each row.

**Technical Definition:** Bulk insert leverages native database APIs (such as array parameter binding in JDBC/ODBC) or SQL multi-row `VALUES` constructs to send multiple rows to the server in one network round-trip, reducing per-row overhead including parsing, planning, and network latency.

**Beginner-Friendly Explanation:** Instead of telling the database "add this one row" a thousand times, you tell it "add these thousand rows" once. It's like mailing a package with 1,000 items instead of sending 1,000 separate letters.

### Purposes

- To reduce network round-trips between the application and the database server.
- To amortize SQL parsing and query planning costs across many rows.
- To improve application throughput when inserting large volumes of data.
- To maintain transactional atomicity for a set of rows as a single unit.

### Syntax Rules and Structure

#### Complete General Syntax (SQL Multi-Row)

```sql
INSERT INTO table_name (column1, column2, column3)
VALUES
    (value1a, value2a, value3a),
    (value1b, value2b, value3b),
    (value1c, value2c, value3c);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `table_name` | Target table for insertion. |
| `(column1, column2, ...)` | Explicit column list; optional if values match all columns in order. |
| `VALUES` | Keyword introducing the row data. |
| `(value1a, value2a, ...)` | A parenthesized tuple representing one row. |
| `,` | Separator between row tuples. |
| `;` | Statement terminator. |

#### Syntax Rules

- All row tuples must have the same number of values corresponding to the column list.
- Data types must be compatible with the target column types.
- The entire statement is a single transaction unit (unless explicitly wrapped in a larger transaction).
- PostgreSQL and most modern RDBMS support multi-row `VALUES`; older systems may limit the number of rows per statement.

#### Constraints and Limitations

- **Statement size limits:** Many databases impose a maximum packet size or statement length. PostgreSQL has a 1 GB limit on query string size; SQL Server has a 65,536 × network packet size limit for `INSERT ... VALUES`.
- **Parameter limits:** Some drivers limit the number of parameters per statement (e.g., 65,535 in MySQL).
- **Memory consumption:** Large multi-row inserts consume memory on both client and server for parsing and execution.
- **Not truly "bulk":** Multi-row `VALUES` still goes through the SQL engine and is slower than `COPY` or `LOAD DATA` for very large datasets.

### Annotated Code Examples

#### Example 1: PostgreSQL Multi-Row Insert

```sql
-- Create a simple table for demonstration
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    price NUMERIC(10,2)
);

-- Insert three rows in a single statement
INSERT INTO products (name, price)
VALUES
    ('Widget', 19.99),
    ('Gadget', 29.99),
    ('Doohickey', 9.99);

-- Verify the insertion
SELECT * FROM products ORDER BY id;
```

**Expected Output:**

```
 id |   name    | price
----+-----------+-------
  1 | Widget    | 19.99
  2 | Gadget    | 29.99
  3 | Doohickey |  9.99
```

**Why This Works:** The single `INSERT` statement sends all three rows in one network round-trip. PostgreSQL parses the statement once and executes the multi-row insert efficiently.

#### Example 2: Java JDBC Batch Insert with Array Parameters

```java
import java.sql.*;

public class BulkInsertExample {
    public static void main(String[] args) throws SQLException {
        String url = "jdbc:postgresql://localhost:5432/mydb";
        try (Connection conn = DriverManager.getConnection(url, "user", "pass")) {
            // Prepare a statement for repeated execution
            String sql = "INSERT INTO products (name, price) VALUES (?, ?)";
            try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
                // Batch multiple rows
                pstmt.setString(1, "Thingamajig");
                pstmt.setBigDecimal(2, new BigDecimal("14.99"));
                pstmt.addBatch();

                pstmt.setString(1, "Whatchamacallit");
                pstmt.setBigDecimal(2, new BigDecimal("24.99"));
                pstmt.addBatch();

                pstmt.setString(1, "Gizmo");
                pstmt.setBigDecimal(2, new BigDecimal("34.99"));
                pstmt.addBatch();

                // Execute all batched rows in one round-trip
                int[] updateCounts = pstmt.executeBatch();
                System.out.println("Rows inserted: " + updateCounts.length);
            }
        }
    }
}
```

**Expected Output:**

```
Rows inserted: 3
```

**Why This Works:** JDBC batches the three `addBatch()` calls and sends them to PostgreSQL in a single network round-trip. The `executeBatch()` method returns an array of update counts, confirming three rows were inserted. This is far more efficient than three separate `executeUpdate()` calls.

#### Example 3: Python psycopg2 `execute_values` (Efficient Bulk Insert)

```python
import psycopg2
from psycopg2.extras import execute_values

conn = psycopg2.connect("dbname=mydb user=user password=pass")
cur = conn.cursor()

# Prepare data as a list of tuples
data = [
    ('Alpha', 10.00),
    ('Beta', 20.00),
    ('Gamma', 30.00),
    ('Delta', 40.00),
]

# execute_values sends all rows in one command using a multi-row VALUES clause
execute_values(
    cur,
    "INSERT INTO products (name, price) VALUES %s",
    data
)

conn.commit()
cur.close()
conn.close()
print(f"Inserted {len(data)} rows.")
```

**Expected Output:**

```
Inserted 4 rows.
```

**Why This Works:** `psycopg2.extras.execute_values` constructs a single `INSERT ... VALUES (...), (...), ...` statement from the list of tuples, minimizing round-trips and maximizing throughput for moderate-sized batches.

### Real-World Cases

- **Application seeding:** Populating reference tables (countries, currencies) during application deployment.
- **User import:** Loading a moderate number of user records from an admin panel.
- **Test data generation:** Inserting synthetic test data for development and QA environments.
- **Real-time analytics:** Inserting batches of events from an application server into a logging table.

### References

- PostgreSQL Documentation: INSERT — https://www.postgresql.org/docs/current/sql-insert.html
- Oracle Database SQL Language Reference: INSERT — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/INSERT.html
- MySQL Reference Manual: INSERT Statement — https://dev.mysql.com/doc/refman/8.0/en/insert.html
- Microsoft SQL Server: INSERT (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/statements/insert-transact-sql


## Core Concept 2: Batch Inserts

### Definitions

**Core Definition:** Batch insert is a technique where multiple insert operations are grouped into chunks and executed together, balancing throughput against memory consumption and transaction size.

**Technical Definition:** Batch insertion involves configuring an optimal chunk size (number of rows per batch), managing network round-trips by sending batches rather than individual rows, and controlling memory allocation on both client and server to avoid excessive resource consumption while maintaining acceptable insertion speed.

**Beginner-Friendly Explanation:** Batch inserts are like packing boxes. You don't send one item at a time, but you also don't try to send everything in one giant box that's too heavy. You find the right box size that's efficient to ship and easy to handle.

### Purposes

- To optimize the trade-off between network round-trip overhead and memory consumption.
- To avoid overwhelming the database server with excessively large transactions.
- To manage lock contention and prevent long-running transactions from blocking other operations.
- To provide a tunable performance knob for different hardware and workload configurations.

### Syntax Rules and Structure

#### Complete General Syntax (JDBC Batch)

```java
PreparedStatement pstmt = conn.prepareStatement("INSERT INTO table (col1, col2) VALUES (?, ?)");
for (Row row : rows) {
    pstmt.setObject(1, row.getCol1());
    pstmt.setObject(2, row.getCol2());
    pstmt.addBatch();
    if (++count % BATCH_SIZE == 0) {
        pstmt.executeBatch();
        pstmt.clearBatch();
    }
}
pstmt.executeBatch(); // remaining rows
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `PreparedStatement` | Pre-compiled SQL statement with parameter placeholders. |
| `addBatch()` | Adds the current parameter set to the batch queue. |
| `executeBatch()` | Sends all queued statements to the database in one round-trip. |
| `clearBatch()` | Clears the batch queue after execution. |
| `BATCH_SIZE` | Number of rows to accumulate before flushing to the database. |

#### Syntax Rules

- The batch size should be determined experimentally for each database and workload.
- Common starting points: 1,000–5,000 rows for OLTP; 10,000–50,000 for bulk loads.
- Each batch should typically be wrapped in its own transaction or within a larger transaction depending on consistency requirements.
- `executeBatch()` returns an `int[]` array of update counts; some drivers may return `SUCCESS_NO_INFO` or `EXECUTE_FAILED`.

#### Constraints and Limitations

- **Driver limitations:** Some JDBC drivers do not support true batch execution and simulate it with individual statements.
- **Memory:** Large batch sizes consume client-side memory for parameter storage and server-side memory for query execution.
- **Lock escalation:** Large batches may escalate locks in SQL Server (e.g., row locks to page or table locks).
- **Log growth:** Large batches generate large transaction logs, which can impact replication and recovery.
- **MySQL `max_allowed_packet`:** Limits the size of a single network packet.

### Annotated Code Examples

#### Example 1: Python Batch Insert with Configurable Chunk Size

```python
import psycopg2
from psycopg2.extras import execute_batch

def batch_insert(data, batch_size=1000):
    conn = psycopg2.connect("dbname=mydb user=user password=pass")
    cur = conn.cursor()

    sql = "INSERT INTO products (name, price) VALUES (%s, %s)"

    # execute_batch chunks the data automatically
    execute_batch(cur, sql, data, page_size=batch_size)

    conn.commit()
    cur.close()
    conn.close()
    print(f"Inserted {len(data)} rows in batches of {batch_size}.")

# Generate 10,000 rows of test data
data = [(f"Product_{i}", i * 1.5) for i in range(10000)]
batch_insert(data, batch_size=2000)
```

**Expected Output:**

```
Inserted 10000 rows in batches of 2000.
```

**Why This Works:** `execute_batch` from `psycopg2.extras` groups the rows into pages of 2,000 and sends each page as a separate `INSERT` statement with multiple `VALUES` tuples. This balances network efficiency with memory usage and avoids constructing a single enormous SQL statement.

#### Example 2: Java JDBC with Transaction per Batch

```java
import java.sql.*;

public class BatchInsertWithTransactions {
    public static void main(String[] args) throws SQLException {
        String url = "jdbc:postgresql://localhost:5432/mydb";
        try (Connection conn = DriverManager.getConnection(url, "user", "pass")) {
            conn.setAutoCommit(false); // Begin transaction management manually

            String sql = "INSERT INTO products (name, price) VALUES (?, ?)";
            int batchSize = 5000;
            int count = 0;

            try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
                for (int i = 0; i < 50000; i++) {
                    pstmt.setString(1, "Product_" + i);
                    pstmt.setBigDecimal(2, new BigDecimal(i * 0.99));
                    pstmt.addBatch();

                    if (++count % batchSize == 0) {
                        pstmt.executeBatch(); // Send batch
                        conn.commit();          // Commit this batch
                        pstmt.clearBatch();     // Reset for next batch
                    }
                }
                // Insert remaining rows
                if (count % batchSize != 0) {
                    pstmt.executeBatch();
                    conn.commit();
                }
            }
        }
        System.out.println("Batch insert completed.");
    }
}
```

**Expected Output:**

```
Batch insert completed.
```

**Why This Works:** Each batch of 5,000 rows is committed as its own transaction. This prevents one enormous transaction from consuming excessive undo/redo log space and reduces the risk of long-running locks. If a batch fails, only that batch is rolled back.

#### Example 3: SQL Server `BULK INSERT` with `ROWS_PER_BATCH`

```sql
-- Create target table
CREATE TABLE dbo.Products (
    Id INT IDENTITY(1,1) PRIMARY KEY,
    Name NVARCHAR(100),
    Price DECIMAL(10,2)
);

-- Bulk insert from a CSV file with batch size hint
BULK INSERT dbo.Products
FROM 'C:\data\products.csv'
WITH (
    FORMAT = 'CSV',
    FIRSTROW = 2,                    -- Skip header
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n',
    ROWS_PER_BATCH = 10000,          -- Server optimizes for 10,000 rows per batch
    TABLOCK                          -- Minimize lock contention
);
```

**Expected Output:**

```
(10000 rows affected)
```

**Why This Works:** SQL Server's `BULK INSERT` with `ROWS_PER_BATCH` tells the server to expect approximately 10,000 rows per batch, allowing it to optimize memory allocation and logging. `TABLOCK` reduces lock contention by taking a table-level lock instead of row-level locks.

### Real-World Cases

- **Data migration:** Moving data from legacy systems to modern databases in controlled chunks.
- **Nightly ETL:** Loading daily transactional data into a warehouse with batch commits to avoid log explosion.
- **IoT data ingestion:** Inserting sensor readings in batches of 10,000–50,000 rows from an ingestion service.
- **Application sync:** Synchronizing a mobile app's offline data with the server in batches.

### References

- PostgreSQL: `psycopg2.extras.execute_batch` — https://www.psycopg.org/docs/extras.html#psycopg2.extras.execute_batch
- Microsoft SQL Server: BULK INSERT (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/statements/bulk-insert-transact-sql
- MySQL: `max_allowed_packet` — https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_max_allowed_packet
- CockroachDB: Batch Insert Best Practices — https://www.cockroachlabs.com/docs/stable/performance-best-practices-overview


## Core Concept 3: CSV Import

### Definitions

**Core Definition:** CSV import is the process of loading data from a comma-separated values (or delimiter-separated) file directly into a database table using a database-native command.

**Technical Definition:** CSV import utilizes database-specific copy commands—such as PostgreSQL's `COPY` and MySQL's `LOAD DATA INFILE`—that bypass the SQL parser and execution engine, reading data directly from files or streams and writing it to table storage with minimal overhead.

**Beginner-Friendly Explanation:** CSV import is like using a conveyor belt to move items from a box into a warehouse. Instead of handling each item individually, the database reads the file line by line and places everything in the right spot without stopping to check each item through the normal slow process.

### Purposes

- To achieve maximum data loading speed by bypassing SQL layer overhead.
- To load data from external files (CSV, TSV, fixed-width) generated by other systems.
- To support ETL pipelines where data is staged in files before loading into the database.
- To minimize application-side memory usage by streaming data directly from disk.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL COPY)

```sql
COPY table_name [(column_list)]
FROM 'file_path'
WITH (
    FORMAT csv,
    HEADER true,
    DELIMITER ',',
    QUOTE '"',
    NULL '',
    ENCODING 'UTF8'
);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `table_name` | Target table for loading. |
| `(column_list)` | Optional list of columns in file order. |
| `FROM 'file_path'` | Source file path (server-side) or `STDIN` for client-side. |
| `FORMAT csv` | Specifies CSV format (`text`, `csv`, or `binary`). |
| `HEADER true` | Indicates the first line contains column names and should be skipped. |
| `DELIMITER ','` | Column delimiter character. |
| `QUOTE '"'` | Quoting character for fields containing delimiters. |
| `NULL ''` | String representing NULL values. |
| `ENCODING 'UTF8'` | File encoding. |

#### Complete General Syntax (MySQL LOAD DATA INFILE)

```sql
LOAD DATA [LOCAL] INFILE 'file_path'
INTO TABLE table_name
FIELDS TERMINATED BY ','
    OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(column1, column2, @var1)
SET column3 = STR_TO_DATE(@var1, '%m/%d/%Y');
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `LOCAL` | Reads file from client machine instead of server. |
| `INFILE 'file_path'` | Path to the CSV file. |
| `INTO TABLE` | Target table. |
| `FIELDS TERMINATED BY` | Column delimiter. |
| `OPTIONALLY ENCLOSED BY` | Quoting character. |
| `LINES TERMINATED BY` | Row delimiter. |
| `IGNORE 1 ROWS` | Skip header row. |
| `(column1, column2, @var1)` | Column mapping; `@var` for transformation. |
| `SET` | Apply transformations to loaded values. |

#### Syntax Rules

- PostgreSQL `COPY FROM` requires `INSERT` privilege on the target table and read access to the file (server-side) or use of `STDIN`/`\copy` (client-side).
- MySQL `LOAD DATA INFILE` requires the `FILE` privilege and the file must reside in the directory specified by `secure_file_priv`.
- `LOAD DATA LOCAL INFILE` requires `local_infile=ON` on both client and server.
- Both commands are transactional in PostgreSQL; `COPY` can be rolled back. MySQL's `LOAD DATA` is transactional for InnoDB tables.

#### Constraints and Limitations

- **PostgreSQL:** `COPY FROM` is not supported for tables with row-level security; use `INSERT` instead.
- **MySQL:** `secure_file_priv` restricts file locations; `LOCAL` requires client-side configuration.
- **File format:** Must match the expected delimiter, quote, and escape conventions.
- **Encoding:** Mismatched encoding between file and database can cause data corruption.
- **Permissions:** Server-side file access requires appropriate OS-level permissions for the database process.
- **Version-specific:** PostgreSQL 19 adds JSON output format to `COPY TO`; MySQL 8.0+ supports `LOAD DATA ... FROM S3` in certain cloud configurations.

### Annotated Code Examples

#### Example 1: PostgreSQL `COPY` from CSV

```sql
-- Create a table
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    full_name TEXT NOT NULL,
    email TEXT UNIQUE,
    hire_date DATE,
    salary NUMERIC(10,2)
);

-- Load from a CSV file on the server
COPY employees (full_name, email, hire_date, salary)
FROM '/var/lib/postgresql/data/employees.csv'
WITH (
    FORMAT csv,
    HEADER true,
    DELIMITER ',',
    NULL ''
);

-- Verify
SELECT COUNT(*) AS total_rows FROM employees;
```

**Expected Output:**

```
 total_rows
------------
       1500
(1 row)
```

**Why This Works:** `COPY FROM` reads the CSV file directly and writes rows to the table's storage without going through the SQL executor. The `HEADER true` option skips the first line. The entire operation is atomic—if any row fails, the entire `COPY` is rolled back.

#### Example 2: MySQL `LOAD DATA INFILE`

```sql
-- Ensure the CSV file is in the secure directory
-- SHOW VARIABLES LIKE 'secure_file_priv';  -- e.g., /var/lib/mysql-files/

-- Create table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100),
    signup_date DATE
);

-- Load data
LOAD DATA INFILE '/var/lib/mysql-files/users.csv'
INTO TABLE users
FIELDS TERMINATED BY ','
    OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(username, email, @raw_date)
SET signup_date = STR_TO_DATE(@raw_date, '%m/%d/%Y');

-- Verify
SELECT COUNT(*) FROM users;
```

**Expected Output:**

```
+----------+
| COUNT(*) |
+----------+
|     2500 |
+----------+
```

**Why This Works:** `LOAD DATA INFILE` reads the file directly from the server's filesystem, bypassing the SQL layer. The `SET` clause transforms the raw date string from `MM/DD/YYYY` format into MySQL's `DATE` type. `IGNORE 1 ROWS` skips the header. The operation is atomic for InnoDB tables.

#### Example 3: PostgreSQL `\copy` (Client-Side)

```sql
-- From psql, use \copy to read a local file
\copy employees (full_name, email, hire_date, salary)
FROM '~/data/employees.csv'
WITH (FORMAT csv, HEADER true, DELIMITER ',');
```

**Expected Output:**

```
COPY 1500
```

**Why This Works:** `\copy` is a psql meta-command that reads the file on the client machine and sends the data to the server via the `COPY FROM STDIN` protocol. This avoids server-side file permission issues and is ideal for loading files from a developer's workstation.

### Real-World Cases

- **Data warehouse loading:** Loading daily CSV extracts from operational systems into a warehouse.
- **Log ingestion:** Importing web server logs (CSV format) into a database for analysis.
- **CRM migration:** Moving contact lists from a SaaS CRM (CSV export) into an on-premises database.
- **Financial reconciliation:** Importing bank transaction CSV files for reconciliation.

### References

- PostgreSQL Documentation: COPY — https://www.postgresql.org/docs/current/sql-copy.html
- MySQL Reference Manual: LOAD DATA INFILE — https://dev.mysql.com/doc/refman/8.0/en/load-data.html
- PostgreSQL: Populating a Database — https://www.postgresql.org/docs/current/populate.html
- MySQL: `mysqlimport` — https://dev.mysql.com/doc/refman/8.0/en/mysqlimport.html


## Core Concept 4: External Data Loading

### Definitions

**Core Definition:** External data loading is the ability to query and integrate data stored outside the database—whether in another database, a file, or a cloud storage bucket—as if it were a native table.

**Technical Definition:** Foreign Data Wrappers (FDWs) are PostgreSQL extensions that implement the SQL/MED (Management of External Data) standard, providing a standardized interface for accessing external data sources through foreign servers, user mappings, and foreign tables. Other databases provide analogous features: Oracle External Tables, SQL Server Linked Servers, and MySQL Federated Storage Engine.

**Beginner-Friendly Explanation:** External data loading lets your database see data that lives somewhere else—like another database, a file on disk, or a cloud storage bucket—as if it were a table right inside your database. You can query it with normal SQL without copying it first.

### Purposes

- To query external data without physically importing it into the local database.
- To integrate data from heterogeneous sources (other RDBMS, NoSQL, flat files, cloud storage).
- To support federated queries that join local and remote data in a single SQL statement.
- To enable near-real-time access to data that changes frequently in the source system.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL FDW)

```sql
-- Step 1: Install the FDW extension
CREATE EXTENSION postgres_fdw;

-- Step 2: Create a foreign server
CREATE SERVER foreign_server
FOREIGN DATA WRAPPER postgres_fdw
OPTIONS (host 'remote_host', port '5432', dbname 'remote_db');

-- Step 3: Create a user mapping
CREATE USER MAPPING FOR local_user
SERVER foreign_server
OPTIONS (user 'remote_user', password 'remote_pass');

-- Step 4: Create a foreign table
CREATE FOREIGN TABLE foreign_products (
    id INT,
    name TEXT,
    price NUMERIC(10,2)
)
SERVER foreign_server
OPTIONS (schema_name 'public', table_name 'products');

-- Step 5: Query the foreign table
SELECT * FROM foreign_products WHERE price > 20;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `CREATE EXTENSION` | Installs the FDW module (e.g., `postgres_fdw`, `file_fdw`). |
| `CREATE SERVER` | Defines the remote data source and connection options. |
| `CREATE USER MAPPING` | Maps local database users to remote credentials. |
| `CREATE FOREIGN TABLE` | Defines the structure of the remote table locally. |
| `OPTIONS` | Key-value pairs for connection (host, port, dbname) and table mapping (schema_name, table_name). |

#### Syntax Rules

- The FDW extension must be installed before creating a server.
- `CREATE USER MAPPING` is required for each local user who needs access to the foreign server.
- Foreign table column names and types should match the remote table; column order may differ (matching is by name).
- Not all FDWs support write operations; `postgres_fdw` supports `INSERT`, `UPDATE`, `DELETE`, and `TRUNCATE` on remote tables.

#### Constraints and Limitations

- **`postgres_fdw` limitations:** Does not support `INSERT ... ON CONFLICT DO UPDATE`; supports `ON CONFLICT DO NOTHING` only without unique index inference.
- **Row-level security:** Foreign tables may not support RLS on the remote side.
- **Performance:** Remote queries may transfer large result sets across the network; use `WHERE` clauses to push down predicates where possible.
- **Version-specific:** `postgres_fdw` is bundled with PostgreSQL 9.3+; older versions require `dblink`.
- **`file_fdw`:** Read-only; cannot write back to files.
- **Security:** User mapping credentials are stored in the database catalog; restrict access to `pg_user_mapping`.

### Annotated Code Examples

#### Example 1: PostgreSQL `postgres_fdw` Setup and Query

```sql
-- On the local database:
CREATE EXTENSION IF NOT EXISTS postgres_fdw;

-- Connect to a remote PostgreSQL database
CREATE SERVER remote_pg
FOREIGN DATA WRAPPER postgres_fdw
OPTIONS (host '192.168.1.100', port '5432', dbname 'inventory');

-- Map local user to remote credentials
CREATE USER MAPPING FOR CURRENT_USER
SERVER remote_pg
OPTIONS (user 'remote_reader', password 'secret123');

-- Define a foreign table pointing to remote 'products' table
CREATE FOREIGN TABLE remote_products (
    product_id INT,
    product_name TEXT,
    unit_price NUMERIC(10,2),
    stock_quantity INT
)
SERVER remote_pg
OPTIONS (schema_name 'public', table_name 'products');

-- Query remote data as if it were local
SELECT product_name, unit_price
FROM remote_products
WHERE stock_quantity < 10
ORDER BY unit_price DESC;
```

**Expected Output:**

```
 product_name | unit_price
--------------+------------
 Rare Widget  |     199.99
 Limited Item |      89.99
(2 rows)
```

**Why This Works:** `postgres_fdw` translates the local query into a remote query, pushes down the `WHERE` clause and `ORDER BY` to the remote server, and retrieves only the matching rows. This minimizes network transfer and leverages the remote server's indexing and sorting capabilities.

#### Example 2: PostgreSQL `file_fdw` for CSV Access

```sql
-- Install file_fdw
CREATE EXTENSION file_fdw;

-- Create a foreign server for files
CREATE SERVER csv_server FOREIGN DATA WRAPPER file_fdw;

-- Create a foreign table pointing to a CSV file
CREATE FOREIGN TABLE csv_sales (
    order_id INT,
    customer TEXT,
    amount NUMERIC(10,2),
    order_date DATE
)
SERVER csv_server
OPTIONS (
    filename '/data/sales.csv',
    format 'csv',
    header 'true'
);

-- Query the CSV file directly
SELECT customer, SUM(amount) AS total
FROM csv_sales
GROUP BY customer
ORDER BY total DESC
LIMIT 5;
```

**Expected Output:**

```
 customer  |  total
-----------+---------
 Acme Corp | 15000.00
 Globex    | 12500.00
 Initech   |  9800.00
 Umbrella  |  7200.00
 Stark Ind |  6500.00
(5 rows)
```

**Why This Works:** `file_fdw` reads the CSV file as a foreign table, allowing SQL queries directly against the file. The `header 'true'` option skips the first line. Aggregation and sorting are performed by PostgreSQL after reading the file.

#### Example 3: Oracle External Table (for comparison)

```sql
-- Create a directory object pointing to the file location
CREATE OR REPLACE DIRECTORY ext_data_dir AS '/data/external';

-- Create an external table
CREATE TABLE ext_products (
    product_id    NUMBER,
    product_name  VARCHAR2(100),
    price         NUMBER(10,2)
)
ORGANIZATION EXTERNAL (
    TYPE ORACLE_LOADER
    DEFAULT DIRECTORY ext_data_dir
    ACCESS PARAMETERS (
        RECORDS DELIMITED BY NEWLINE
        FIELDS TERMINATED BY ','
        MISSING FIELD VALUES ARE NULL
    )
    LOCATION ('products.csv')
)
REJECT LIMIT UNLIMITED;

-- Query the external table
SELECT * FROM ext_products WHERE price > 50;
```

**Expected Output:**

```
PRODUCT_ID PRODUCT_NAME          PRICE
---------- -------------------- ----------
       101 Premium Widget         99.99
       205 Deluxe Gadget         149.99
```

**Why This Works:** Oracle External Tables allow querying flat files without loading them into the database. The `ORGANIZATION EXTERNAL` clause defines the file format and location. Data is read on-demand during query execution.

### Real-World Cases

- **Data federation:** Joining customer data in PostgreSQL with order data in Oracle without ETL.
- **Cloud data lake access:** Querying Parquet or CSV files stored in S3 via `file_fdw` or specialized FDWs.
- **Legacy system integration:** Accessing mainframe data through an FDW that speaks the legacy protocol.
- **Cross-database reporting:** Generating reports that combine data from multiple PostgreSQL databases.

### References

- PostgreSQL Documentation: Foreign Data Wrappers — https://www.postgresql.org/docs/current/fdwhandler.html
- PostgreSQL: `postgres_fdw` — https://www.postgresql.org/docs/current/postgres-fdw.html
- PostgreSQL: `file_fdw` — https://www.postgresql.org/docs/current/file-fdw.html
- Oracle Database: External Tables — https://docs.oracle.com/en/database/oracle/oracle-database/21/sutil/oracle-external-tables.html
- SQL/MED Standard (ISO/IEC 9075-9) — https://www.iso.org/standard/76584.html


## Core Concept 5: Bulk-Loading Utilities

### Definitions

**Core Definition:** Bulk-loading utilities are command-line programs or extensions designed specifically for high-speed data loading, often bypassing the normal SQL interface and using direct path or binary protocols.

**Technical Definition:** Bulk-loading utilities such as `pg_bulkload`, Oracle SQL*Loader, and SQL Server `bcp` provide specialized mechanisms for loading data at maximum speed by bypassing shared buffers, disabling logging, using direct path writes, and supporting parallel loading with minimal database overhead.

**Beginner-Friendly Explanation:** These are special tools that are like express lanes at the airport. They skip the normal security line (SQL parsing and planning) and go straight to the gate, getting data into the database much faster than regular methods.

### Purposes

- To achieve the absolute maximum data loading throughput for very large datasets.
- To load data while bypassing shared buffers, reducing memory pressure on the database server.
- To support direct path loads that write directly to data files, avoiding the SQL engine.
- To provide ETL features such as data validation, transformation, and error handling during load.

### Syntax Rules and Structure

#### Complete General Syntax (`pg_bulkload`)

```bash
# Control file format
pg_bulkload control_file.ctl
```

**Control file example:**

```
TABLE = target_table
INFILE = /path/to/data.csv
TYPE = CSV
DELIMITER = ,
QUOTE = "
HEADER = YES
WRITER = DIRECT
PARALLEL = 4
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `TABLE` | Target table name. |
| `INFILE` | Input file path. |
| `TYPE` | File format (`CSV`, `TEXT`, `BINARY`). |
| `DELIMITER` | Column delimiter. |
| `HEADER` | Whether file has a header row. |
| `WRITER` | `DIRECT` bypasses shared buffers; `BUFFERED` uses normal path. |
| `PARALLEL` | Number of parallel loading processes. |

#### Complete General Syntax (Oracle SQL*Loader)

```bash
sqlldr userid=username/password@database control=load.ctl log=load.log bad=load.bad
```

**Control file example:**

```
LOAD DATA
INFILE 'data.csv'
INTO TABLE employees
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
TRAILING NULLCOLS
(
    employee_id INTEGER EXTERNAL,
    first_name CHAR,
    last_name CHAR,
    hire_date DATE "YYYY-MM-DD",
    salary DECIMAL EXTERNAL
)
```

#### Complete General Syntax (SQL Server `bcp`)

```bash
bcp database.schema.table in data.csv -c -t, -r\n -S server -U user -P password -b 10000
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `database.schema.table` | Target table. |
| `in` | Direction (in = import, out = export). |
| `data.csv` | Data file. |
| `-c` | Character mode (as opposed to native). |
| `-t,` | Field terminator (comma). |
| `-r\n` | Row terminator (newline). |
| `-b 10000` | Batch size (rows per batch). |

#### Syntax Rules

- `pg_bulkload` requires a control file and is invoked from the command line.
- Oracle SQL*Loader uses a control file and supports both conventional and direct path loads.
- `bcp` can be used interactively or in scripts; it supports native and character modes.
- Direct path loads (SQL*Loader, `pg_bulkload`) bypass the SQL engine and may not fire triggers or enforce constraints during loading.

#### Constraints and Limitations

- **`pg_bulkload`:** Not part of core PostgreSQL; must be installed separately. Direct writer bypasses shared buffers and WAL, which may affect crash recovery and replication.
- **SQL*Loader:** Direct path loads lock the table and do not fire `INSERT` triggers. Conventional path loads use the SQL engine and are slower.
- **`bcp`:** Requires appropriate permissions; `-b` batch size affects memory and log usage. Native mode is faster but less portable.
- **Version-specific:** `pg_bulkload` supports specific PostgreSQL versions; check compatibility.
- **Unsafe if misused:** Direct path loads during active transactions or with constraints can lead to data corruption or inconsistency.

### Annotated Code Examples

#### Example 1: `pg_bulkload` CSV Load

```bash
# Create a control file: load_products.ctl
cat > load_products.ctl << 'EOF'
TABLE = products
INFILE = /data/products.csv
TYPE = CSV
DELIMITER = ,
QUOTE = "
HEADER = YES
WRITER = DIRECT
PARALLEL = 2
EOF

# Run pg_bulkload
pg_bulkload load_products.ctl
```

**Expected Output:**

```
NOTICE: BULK LOAD START
NOTICE: BULK LOAD END
0 Rows skipped.
50000 Rows successfully loaded.
0 Rows not loaded due to parse errors.
0 Rows not loaded due to duplicate errors.
0 Rows replaced with new rows.
```

**Why This Works:** `pg_bulkload` with `WRITER = DIRECT` bypasses PostgreSQL's shared buffers and writes directly to the table's data files, achieving speeds significantly faster than `COPY`. `PARALLEL = 2` uses two processes to load data concurrently.

#### Example 2: Oracle SQL*Loader Control File and Invocation

**Control file `load_employees.ctl`:**

```
LOAD DATA
INFILE 'employees.csv'
BADFILE 'employees.bad'
DISCARDFILE 'employees.dsc'
INTO TABLE employees
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
TRAILING NULLCOLS
(
    employee_id INTEGER EXTERNAL,
    first_name CHAR,
    last_name CHAR,
    hire_date DATE "YYYY-MM-DD",
    salary DECIMAL EXTERNAL
)
```

**Invocation:**

```bash
sqlldr userid=hr/oracle@orcl control=load_employees.ctl log=load_employees.log
```

**Expected Output (from log):**

```
SQL*Loader: Release 21.0.0.0.0 - Production

Total logical records skipped:          0
Total logical records read:         50000
Total logical records rejected:         0
Total logical records discarded:        0
```

**Why This Works:** SQL*Loader reads the control file, parses the CSV, and loads data into the `employees` table. The `BADFILE` captures rejected rows, and `DISCARDFILE` captures discarded rows. The `TRAILING NULLCOLS` option allows missing trailing columns to be set to `NULL`.

#### Example 3: SQL Server `bcp` Import

```bash
# Export data from a table to a CSV file
bcp AdventureWorks2022.Person.Person out C:\temp\person.csv -c -t, -r\n -S localhost -T

# Import data from a CSV file into a table (with batch size 5000)
bcp AdventureWorks2022.Person.Person in C:\temp\person.csv -c -t, -r\n -S localhost -T -b 5000
```

**Expected Output:**

```
Starting copy...
5000 rows copied.
Network packet size (bytes): 4096
Clock Time (ms.) Total     : 1250   Average : (4000.00 rows per sec.)
```

**Why This Works:** `bcp` in character mode (`-c`) reads the CSV and inserts rows in batches of 5,000 (`-b 5000`). The `-T` flag uses trusted Windows authentication. The output reports the number of rows copied and the throughput.

### Real-World Cases

- **Initial database population:** Loading terabytes of historical data into a new data warehouse.
- **Database migration:** Moving data from a legacy system to a new RDBMS using vendor-specific bulk tools.
- **ETL pipelines:** Using `pg_bulkload` or SQL*Loader as the "Load" step in extract-transform-load workflows.
- **Data archival:** Bulk-loading archived data back into the database for analysis.

### References

- `pg_bulkload` GitHub Repository — https://github.com/ossc-db/pg_bulkload
- Oracle Database Utilities: SQL*Loader — https://docs.oracle.com/en/database/oracle/oracle-database/21/sutil/oracle-sql-loader.html
- Microsoft SQL Server: bcp Utility — https://learn.microsoft.com/en-us/sql/tools/bcp-utility
- `pg_bulkload` Documentation — http://ossc-db.github.io/pg_bulkload/index.html


## Core Concept 6: Transaction Isolation Management

### Definitions

**Core Definition:** Transaction isolation management during data loading involves controlling how concurrent transactions see each other's changes, and managing database features (constraints, indexes, WAL) to maximize load performance while maintaining data integrity.

**Technical Definition:** Transaction isolation management encompasses setting the appropriate isolation level (READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE) for the loading transaction, temporarily disabling foreign key constraints and triggers, dropping and recreating indexes, and configuring write-ahead logging (WAL) parameters to minimize logging overhead during bulk ingestion.

**Beginner-Friendly Explanation:** When you're pouring a lot of data into a database, you can turn off some safety checks temporarily to go faster—like turning off the metal detector at the airport when you're bringing in a huge shipment. But you have to turn them back on afterward to make sure everything is safe.

### Purposes

- To maximize bulk load speed by minimizing logging and constraint-checking overhead.
- To avoid trigger event queue overflow when loading millions of rows with foreign key constraints.
- To reduce WAL generation and replication lag during large data ingestion.
- To maintain data consistency by reapplying constraints and indexes after loading.
- To control the visibility of uncommitted data to other concurrent transactions.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL Bulk Load Optimization)

```sql
-- Begin a single transaction for the entire load
BEGIN;

-- Disable triggers (requires superuser)
ALTER TABLE target_table DISABLE TRIGGER ALL;

-- Drop foreign key constraints (if safe to do so)
ALTER TABLE target_table DROP CONSTRAINT fk_constraint_name;

-- Drop indexes (optional, for very large loads)
DROP INDEX IF EXISTS idx_name;

-- Load data
COPY target_table FROM '/path/to/data.csv' WITH (FORMAT csv, HEADER true);

-- Recreate indexes
CREATE INDEX idx_name ON target_table (column);

-- Reapply foreign key constraints
ALTER TABLE target_table ADD CONSTRAINT fk_constraint_name
    FOREIGN KEY (column) REFERENCES other_table(id);

-- Re-enable triggers
ALTER TABLE target_table ENABLE TRIGGER ALL;

COMMIT;

-- Run ANALYZE to update statistics
ANALYZE target_table;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `BEGIN` / `COMMIT` | Wraps the entire load in a single transaction. |
| `DISABLE TRIGGER ALL` | Prevents triggers (including FK checks) from firing during load. |
| `DROP CONSTRAINT` | Removes FK constraints; avoids trigger event queue overflow. |
| `DROP INDEX` | Removes indexes; faster to rebuild after load than update incrementally. |
| `COPY` | Bulk load command. |
| `CREATE INDEX` | Rebuilds indexes on the fully loaded table. |
| `ADD CONSTRAINT` | Reapplies FK constraints; validates in bulk. |
| `ENABLE TRIGGER ALL` | Restores trigger functionality. |
| `ANALYZE` | Updates query planner statistics. |

#### Configuration Parameters for WAL Bypass (PostgreSQL)

```ini
# postgresql.conf
wal_level = minimal
archive_mode = off
max_wal_senders = 0
```

**Component Breakdown:**

| Parameter | Effect |
|-----------|--------|
| `wal_level = minimal` | Reduces WAL volume; allows certain commands to skip WAL entirely. |
| `archive_mode = off` | Disables WAL archiving. |
| `max_wal_senders = 0` | Disables streaming replication. |

#### Syntax Rules

- Disabling triggers requires table owner or superuser privileges.
- Dropping and recreating indexes requires exclusive table access.
- WAL configuration changes require a **server restart**.
- Isolation levels are set per-transaction using `SET TRANSACTION ISOLATION LEVEL`.
- In MySQL, use `SET foreign_key_checks=0`, `SET unique_checks=0`, `SET autocommit=0` for bulk load sessions.

#### Constraints and Limitations

- **Unsafe if misused:** Disabling constraints and triggers removes error checking; data integrity may be compromised if the source data contains violations.
- **Exclusive locking:** Dropping indexes and disabling triggers lock the table, blocking other users.
- **WAL bypass:** Setting `wal_level = minimal` disables replication and point-in-time recovery; a new base backup is required after the load.
- **Version-specific:** `wal_level = minimal` behavior varies; in PostgreSQL 9.6+, `COPY FROM` can skip WAL only if the table was created or truncated in the same transaction.
- **ANALYZE required:** After bulk loading, statistics are stale; running `ANALYZE` is strongly recommended.

### Annotated Code Examples

#### Example 1: PostgreSQL Full Bulk Load Optimization

```sql
-- Step 1: Begin transaction
BEGIN;

-- Step 2: Create a fresh table (so COPY can skip WAL)
CREATE TABLE large_data (
    id SERIAL PRIMARY KEY,
    value TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Step 3: Load data using COPY
COPY large_data (id, value, created_at)
FROM '/data/large_dataset.csv'
WITH (FORMAT csv, HEADER true);

-- Step 4: Create indexes after loading
CREATE INDEX idx_large_data_created_at ON large_data (created_at);

-- Step 5: Commit
COMMIT;

-- Step 6: Update statistics
ANALYZE large_data;

-- Verify
SELECT COUNT(*) FROM large_data;
```

**Expected Output:**

```
  count
---------
 1000000
(1 row)
```

**Why This Works:** Because the table is created and loaded within the same transaction, and `wal_level` is `minimal`, PostgreSQL can skip WAL logging entirely for the `COPY` operation. Indexes are created after loading (faster than incremental updates). The transaction commits atomically. `ANALYZE` updates planner statistics.

#### Example 2: MySQL Bulk Load Session Settings

```sql
-- Disable autocommit, unique checks, and foreign key checks for the session
SET autocommit = 0;
SET unique_checks = 0;
SET foreign_key_checks = 0;

-- Load data
LOAD DATA INFILE '/var/lib/mysql-files/large_data.csv'
INTO TABLE large_table
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;

-- Commit the load
COMMIT;

-- Re-enable checks
SET unique_checks = 1;
SET foreign_key_checks = 1;
SET autocommit = 1;

-- Update statistics
ANALYZE TABLE large_table;
```

**Expected Output:**

```
Query OK, 2000000 rows affected (15.32 sec)
Records: 2000000  Deleted: 0  Skipped: 0  Warnings: 0
```

**Why This Works:** Disabling `unique_checks` and `foreign_key_checks` during the load eliminates per-row constraint validation. The entire load is one transaction (`autocommit=0`), so it commits atomically. After commit, checks are re-enabled and statistics updated.

#### Example 3: SQL Server `TABLOCK` and Minimal Logging

```sql
-- Ensure the database is in SIMPLE recovery model for minimal logging
ALTER DATABASE MyDB SET RECOVERY SIMPLE;

-- Bulk insert with TABLOCK for minimal logging
BULK INSERT dbo.LargeTable
FROM 'C:\data\large_data.csv'
WITH (
    FORMAT = 'CSV',
    FIRSTROW = 2,
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n',
    TABLOCK,          -- Table-level lock enables minimal logging
    BATCHSIZE = 100000
);

-- Update statistics
UPDATE STATISTICS dbo.LargeTable;

-- Optionally, switch back to FULL recovery
ALTER DATABASE MyDB SET RECOVERY FULL;
```

**Expected Output:**

```
(5000000 rows affected)
```

**Why This Works:** In SQL Server, using `TABLOCK` with `BULK INSERT` under the SIMPLE or BULK_LOGGED recovery model enables **minimal logging**—only page allocations are logged, not individual row inserts. This dramatically reduces transaction log growth and improves performance. `UPDATE STATISTICS` ensures the planner has current information.

### Real-World Cases

- **Data warehouse nightly loads:** Loading billions of rows with indexes and constraints dropped, then rebuilt.
- **Database migration:** Exporting from one system and importing into another with WAL/logging minimized.
- **Analytics ingestion:** Loading large event datasets into a columnar store or partitioned table with constraint checks deferred.
- **Disaster recovery testing:** Bulk loading a backup dataset while replication is disabled to maximize speed.

### References

- PostgreSQL: Populating a Database — https://www.postgresql.org/docs/current/populate.html
- PostgreSQL: Write-Ahead Logging (WAL) Configuration — https://www.postgresql.org/docs/current/wal-configuration.html
- MySQL: `foreign_key_checks` — https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_foreign_key_checks
- Microsoft SQL Server: Prerequisites for Minimal Logging in Bulk Import — https://learn.microsoft.com/en-us/sql/relational-databases/import-export/prerequisites-for-minimal-logging-in-bulk-import
- ISO/IEC 9075-2:2016 (SQL Foundation) — Transaction Isolation Levels — https://www.iso.org/standard/63555.html


## Summary Table: Data Loading Methods Comparison

| Method | Speed | Transactional | Bypasses SQL | Use Case |
|--------|-------|---------------|--------------|----------|
| Multi-row `INSERT` | Moderate | Yes | No | Small to medium batches |
| JDBC/ODBC Batch | Moderate–High | Yes | No | Application-driven bulk inserts |
| PostgreSQL `COPY` | Very High | Yes | Yes | File-based bulk loading |
| MySQL `LOAD DATA INFILE` | Very High | Yes (InnoDB) | Yes | File-based bulk loading |
| Foreign Data Wrapper | N/A (query) | Depends | N/A | Federated queries |
| `pg_bulkload` | Highest | Limited | Yes (direct) | Offline bulk loading |
| Oracle SQL*Loader | Very High | Direct path: no | Yes (direct) | Enterprise bulk loading |
| SQL Server `bcp` | Very High | Yes | Yes | SQL Server bulk loading |


## Final Notes on Deprecated and Unsafe Features

- **PostgreSQL `COPY ... OIDS`:** Deprecated; OIDs are not supported in modern PostgreSQL versions.
- **MySQL `LOAD DATA INFILE` without `LOCAL`:** Requires `secure_file_priv` configuration; often disabled in cloud environments.
- **Oracle SQL*Loader Direct Path:** Does not fire triggers; unsafe if triggers enforce critical business logic.
- **Disabling `wal_level` to `minimal`:** Disables replication and PITR; only safe in offline or non-replicated environments.
- **`SET foreign_key_checks=0` in MySQL:** Unsafe if source data violates FK constraints; only use with validated data.
- **Version-specific behavior:** PostgreSQL `COPY` WAL-skipping requires `wal_level = minimal` and same-transaction `CREATE TABLE`/`TRUNCATE`; this behavior changed in PostgreSQL 9.6.