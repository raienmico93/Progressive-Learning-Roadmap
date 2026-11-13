# Comprehensive Programming Cheat Sheet: SQL Bulk Operations

## Topic Overview

### Definitions

**Core Definition**
Bulk operations are the set of SQL commands and patterns—`COPY`, `\copy`, `BULK INSERT`, `LOAD DATA`, multi-row `INSERT`, chunked `UPDATE`/`DELETE`, and staging-table workflows—designed to move, modify, or remove large volumes of data with far higher throughput than row-by-row DML.

**Technical Definition**
Bulk operations exploit set-based execution, direct storage-engine writes, and minimal logging to amortise per-row overhead. `COPY` is PostgreSQL's bulk-load primitive, moving rows between a table and a file, program, STDIN, or STDOUT at approximately 10–100× the throughput of equivalent `INSERT` statements. `\copy` is the psql client-side meta-command that runs an SQL `COPY` but reads or writes the file on the client rather than the server. `BULK INSERT` is SQL Server's in-process bulk-import statement, which avoids passing data through the client API's network layer and can therefore be faster than `bcp` or SSIS. MySQL's `LOAD DATA INFILE` bypasses SQL parsing entirely and loads data directly into storage-engine tables. Bulk `UPDATE` and `DELETE` are performed by chunking large operations into batches (`DELETE TOP (N)`, `UPDATE TOP (N)`, `LIMIT`) to avoid lock escalation, transaction-log bloat, and replication lag.

**Beginner-Friendly Explanation**
Bulk operations are the heavy-lifting tools of a database. Instead of inserting, updating, or deleting one row at a time—like carrying bricks one by one—you use a conveyor belt. PostgreSQL's `COPY`, SQL Server's `BULK INSERT`, and MySQL's `LOAD DATA` are that conveyor belt for loading data. For modifying or removing huge numbers of rows, bulk operations mean breaking the work into manageable chunks so the database stays responsive instead of locking up for minutes or hours.

### Key Characteristics

- **Order-of-magnitude throughput gains**: `COPY` is roughly 10–100× faster than `INSERT`; `LOAD DATA` is far faster than multi-row `INSERT`; `BULK INSERT` runs in-process and avoids client-network overhead.
- **Minimal logging where possible**: SQL Server can minimally log bulk imports under `SIMPLE`/`BULK_LOGGED` recovery models when `TABLOCK` is specified and the table is not replicated; PostgreSQL's `COPY FREEZE` avoids revisiting rows after commit, increasing performance of subsequent commands.
- **Single-threaded primitives, parallelisable by design**: `COPY` is single-threaded per process, so throughput-bound loads are parallelised by splitting input into N chunks and running N concurrent `COPY` sessions against the same table.
- **Staging-table isolation**: Raw data is loaded into staging tables first, isolating ETL failures from the warehouse and enabling reprocessing without re-extracting.
- **Chunking is mandatory for bulk DML**: Any operation affecting more than ~10,000 rows should be broken into chunks to avoid lock escalation and log-space reservation for the entire operation.

### Prerequisites

- A running RDBMS instance (PostgreSQL, SQL Server, or MySQL) and a client connection.
- For server-side `COPY`: superuser or membership in `pg_read_server_files`, `pg_write_server_files`, or `pg_execute_server_program`.
- For `BULK INSERT`: `INSERT` and `ADMINISTER BULK OPERATIONS` permissions; the file must be accessible from the SQL Server instance.
- For `LOAD DATA INFILE` (non-`LOCAL`): `FILE` privilege; the file must reside on the server host.
- For staging-table workflows: `CREATE`, `INSERT`, `SELECT`, `UPDATE`, `DELETE`, and `MERGE` privileges on the staging and target tables.

### Related Programming Areas

- Data Manipulation Language (DML) – `INSERT`, `UPDATE`, `DELETE`, `MERGE`.
- Extract, Transform, Load (ETL) – staging tables, incremental loads, and change data capture.
- Transaction log management – minimal logging, recovery models, and log-space reservation.
- Concurrency control – lock escalation, chunking, and isolation levels.
- Query optimisation – indexing strategy, `TABLOCK`, `FREEZE`, and batch sizing.

---

## Core Concepts and Features

### 1. Bulk Insert

#### Definitions

**Core Definition**
Bulk insert is the process of loading many rows into a table in a single statement or a small number of statements, rather than issuing one `INSERT` per row.

**Technical Definition**
Bulk insert encompasses multi-row `INSERT ... VALUES` (batched to 500–1,000 rows per statement), `INSERT ... SELECT` from a staging table, and dedicated bulk-loading utilities (`COPY`, `\copy`, `BULK INSERT`, `LOAD DATA`). The SQL standard defines the row-value constructor `VALUES (r1c1, r1c2), (r2c1, r2c2)`, but engines impose different architectural limits: MySQL is bound by `max_allowed_packet`, PostgreSQL by the 65,535-parameter protocol limit, and SQL Server by a 1,000-row limit per `VALUES` statement and a 2,100-parameter limit.

**Beginner-Friendly Explanation**
Bulk insert is loading data in large batches instead of one row at a time. A single `INSERT` with 1,000 rows is often dozens of times faster than 1,000 separate `INSERT` statements, and a dedicated bulk utility like `COPY` or `LOAD DATA` is faster still.

#### Purposes

- To load large datasets (CSV extracts, ETL outputs, initial database population) efficiently.
- To amortise parsing, planning, and network round-trip overhead across many rows.
- To minimise transaction-log volume and lock contention during high-volume ingestion.

#### Syntax Rules and Structure

**Multi-row `VALUES` (portable):**

```sql
INSERT INTO table_name (col1, col2, ...) VALUES
    (v1, v2, ...),
    (v3, v4, ...),
    ...;
```

**PostgreSQL `COPY`:**

```sql
COPY table_name (col1, col2, ...)
FROM '/path/to/file.csv'
WITH (FORMAT csv, HEADER true);
```

**SQL Server `BULK INSERT`:**

```sql
BULK INSERT schema.table_name
FROM 'C:\path\to\file.csv'
WITH (FORMAT = 'CSV', FIRSTROW = 2, TABLOCK, BATCHSIZE = 5000);
```

**MySQL `LOAD DATA INFILE`:**

```sql
LOAD DATA INFILE '/path/to/file.csv'
INTO TABLE table_name
FIELDS TERMINATED BY ',' ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

**Component breakdown:**

| Technique | Scope | Key characteristic |
|-----------|-------|-------------------|
| Multi-row `VALUES` | All RDBMSs | Portable; 500–1,000 rows per statement |
| `COPY` / `\copy` | PostgreSQL | Fastest PostgreSQL bulk path; server-side vs. client-side file access |
| `BULK INSERT` | SQL Server | In-process; avoids client-network overhead |
| `LOAD DATA` | MySQL | Bypasses SQL parsing; loads directly into storage engine |

**Syntax rules:**

- Batch multi-row `VALUES` to 500–1,000 rows to stay within parameter limits and memory pressure.
- `COPY` cannot be used inside a transaction block in some configurations; `BULK INSERT` is a single autocommitted transaction unless `BATCHSIZE` is specified.
- `LOAD DATA LOCAL` reads the file on the client and is slower than server-side `LOAD DATA INFILE` because the file contents travel over the connection.

**Constraints and limitations:**

- Multi-row `VALUES` is unsuitable for millions of rows; use a native bulk utility.
- SQL Server `BULK INSERT` does not skip header rows by itself; `FIRSTROW = 2` skips a row but does not validate the skipped row's fields.
- `COPY` is single-threaded per process; parallelism requires multiple concurrent `COPY` sessions.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – COPY from CSV

```sql
-- Step 1: Create a table.
CREATE TABLE employees (id int PRIMARY KEY, name text, salary numeric);

-- Step 2: Bulk-load the CSV from the server filesystem.
COPY employees (id, name, salary)
FROM '/tmp/employees.csv'
WITH (FORMAT csv, HEADER true);

-- Step 3: Verify the row count.
SELECT count(*) FROM employees;
```

**Expected output:**

```text
CREATE TABLE
COPY 100000
 count
--------
 100000
(1 row)
```

**Why this output appears**: `COPY` reads the CSV on the server and inserts all rows directly into the table's heap pages, bypassing the per-row overhead of `INSERT`. `COPY 100000` confirms the number of rows loaded.

##### Example 2: SQL Server – BULK INSERT with TABLOCK and BATCHSIZE

```sql
-- Step 1: Create a table.
CREATE TABLE dbo.Sales (SaleID int, Amount decimal(12,2), SaleDate date);

-- Step 2: Bulk-insert the CSV with minimal logging options.
BULK INSERT dbo.Sales
FROM 'C:\data\sales.csv'
WITH (
    FORMAT = 'CSV',
    FIRSTROW = 2,
    TABLOCK,
    BATCHSIZE = 5000,
    CHECK_CONSTRAINTS
);
```

**Expected output:**

```text
(5000 rows affected)
(5000 rows affected)
...
```

**Why this output appears**: `BATCHSIZE = 5000` commits each batch of 5,000 rows as a separate transaction, reducing log-space reservation and lock duration. `TABLOCK` enables minimal logging when the database is in `SIMPLE` or `BULK_LOGGED` recovery and the table is not replicated.

#### Real-World Cases

- **Nightly ETL**: A CSV extract from an operational system is loaded into a staging table with `COPY` or `LOAD DATA`, then transformed and merged into the warehouse.
- **Initial database population**: A new environment is seeded from a `pg_dump` restored with `COPY` inside a single transaction, which is the fastest path.

---

### 2. Bulk Update

#### Definitions

**Core Definition**
Bulk update is the process of modifying many rows in a table, typically by applying the change in chunks or through a staging-table-and-`MERGE` pattern, rather than row-by-row.

**Technical Definition**
Bulk updates are performed with set-based `UPDATE` statements, but large single-statement updates lock many rows, reserve log space for the entire operation, and can cause lock escalation. The standard patterns are chunked `UPDATE TOP (N)` in a `WHILE` loop (SQL Server) or `UPDATE ... LIMIT N` (MySQL), each iteration being its own autocommitted transaction. An alternative is to load the new values into a staging table and apply them with a set-based `MERGE`, which is often recommended for large volumes.

**Beginner-Friendly Explanation**
A bulk update changes lots of rows at once. If you try to update a million rows in one statement, the database locks them all and fills the transaction log. Bulk update means doing it in batches—say, 5,000 rows at a time—so the database stays responsive.

#### Purposes

- To apply a large-scale data correction or transformation without blocking concurrent workloads.
- To avoid transaction-log bloat and lock escalation.
- To keep replication lag manageable on replicas.

#### Syntax Rules and Structure

**SQL Server chunked UPDATE:**

```sql
DECLARE @Rows INT = 1;
WHILE @Rows > 0
BEGIN
    UPDATE TOP (5000) dbo.Notification
    SET [Status] = 'Cancelled'
    WHERE [Status] = 'Pending'
      AND ScheduledFor < DATEADD(day, -30, SYSDATETIME());
    SET @Rows = @@ROWCOUNT;
END;
```

**MySQL chunked UPDATE:**

```sql
UPDATE events
SET processed = TRUE
WHERE processed = FALSE
LIMIT 1000;
-- Repeat until 0 rows affected.
```

**Staging-table + MERGE (SQL Server):**

```sql
MERGE dbo.Target AS T
USING dbo.Staging AS S
ON T.Key1 = S.Key1
WHEN MATCHED THEN UPDATE SET T.Value = S.Value;
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UPDATE TOP (N)` / `LIMIT N` | Caps rows per iteration. |
| `WHILE` loop / application loop | Repeats until no rows remain. |
| Staging table + `MERGE` | Applies a set of changes in one set-based statement. |

**Syntax rules:**

- Start with a chunk size of 5,000 rows; reduce proportionally if the table has many indexes, because each modified row acquires one lock per index.
- Each chunk should be its own transaction (or autocommitted) to release locks promptly.
- Monitor `WRITELOG` wait times to detect log I/O bottlenecks and reduce chunk size if needed.

**Constraints and limitations:**

- `MERGE` is not always faster than a straight `UPDATE` when only updates are needed; it may waste time evaluating insert branches.
- Chunked updates can miss rows if the `WHERE` clause is not deterministic across iterations; use a key-based predicate or a stable ordering.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Chunked UPDATE with TOP in a WHILE loop

```sql
-- Step 1: Create a table with many pending rows.
CREATE TABLE dbo.Notification (ID int IDENTITY PRIMARY KEY, Status varchar(20), ScheduledFor datetime2);
INSERT INTO dbo.Notification (Status, ScheduledFor)
SELECT 'Pending', DATEADD(day, -45, SYSDATETIME())
FROM sys.all_objects;

-- Step 2: Cancel old pending notifications in chunks of 5,000.
DECLARE @Rows INT = 1;
WHILE @Rows > 0
BEGIN
    UPDATE TOP (5000) dbo.Notification
    SET [Status] = 'Cancelled'
    WHERE [Status] = 'Pending'
      AND ScheduledFor < DATEADD(day, -30, SYSDATETIME());
    SET @Rows = @@ROWCOUNT;
END;

-- Step 3: Verify.
SELECT [Status], COUNT(*) FROM dbo.Notification GROUP BY [Status];
```

**Expected output:**

```text
Status     (No column name)
---------  ----------------
Cancelled  ...
Pending    ...
```

**Why this output appears**: Each `UPDATE TOP (5000)` modifies at most 5,000 rows and commits independently, releasing locks before the next iteration. The loop continues until `@@ROWCOUNT` reaches zero, meaning all eligible rows have been updated.

#### Real-World Cases

- **Status transitions**: Millions of “pending” jobs are moved to “expired” in chunks overnight without blocking the job-processing application.
- **Data correction**: A column value is updated across a large table using a staging table and `MERGE`, applying only the rows that actually changed.

---

### 3. Bulk Delete

#### Definitions

**Core Definition**
Bulk delete is the process of removing many rows from a table in chunks or via `TRUNCATE`/partition switching, rather than in a single large `DELETE` statement.

**Technical Definition**
A single large `DELETE` locks many rows, reserves log space for the entire operation, and may escalate row locks to a table lock, blocking all readers and writers. The standard chunked pattern is `DELETE TOP (N)` in a `WHILE` loop (SQL Server) or `DELETE ... LIMIT N` (MySQL), with each iteration as its own autocommitted transaction. For full-table removal, `TRUNCATE TABLE` is faster than `DELETE` but cannot be filtered; for partition-aligned tables, partition switching drops entire partitions instantly.

**Beginner-Friendly Explanation**
Bulk delete removes lots of rows at once. Instead of one giant `DELETE` that locks the table for minutes, you delete in batches—5,000 rows at a time—so other queries can keep running.

#### Purposes

- To purge stale or expired data without blocking concurrent workloads.
- To avoid transaction-log bloat and lock escalation.
- To keep replication lag manageable when deleting from replicated tables.

#### Syntax Rules and Structure

**SQL Server chunked DELETE:**

```sql
DECLARE @RowsAffected INT = 1;
WHILE @RowsAffected > 0
BEGIN
    DELETE TOP (5000) FROM dbo.AuditLog
    WHERE LoggedAt < DATEADD(day, -90, SYSDATETIME());
    SET @RowsAffected = @@ROWCOUNT;
    WAITFOR DELAY '00:00:00.100';
END;
```

**MySQL chunked DELETE:**

```sql
DELETE FROM events
WHERE created_at < NOW() - INTERVAL 90 DAY
LIMIT 1000;
-- Repeat until 0 rows affected.
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `DELETE TOP (N)` / `LIMIT N` | Caps rows per iteration. |
| `WHILE` loop | Repeats until no rows remain. |
| `WAITFOR DELAY` / sleep | Optional pause between batches to reduce I/O pressure. |

**Syntax rules:**

- Start with 5,000 rows per chunk; reduce if the table has many indexes.
- Use an indexed `WHERE` column to avoid scanning the table for each batch.
- For full-table removal, `TRUNCATE TABLE` is faster than `DELETE` and resets identity counters.

**Constraints and limitations:**

- A chunked `DELETE` without a deterministic `WHERE` clause can miss rows or revisit rows across iterations.
- `TRUNCATE` cannot be filtered and is not rollback-able in MySQL; use `DELETE` when triggers or row-level filtering are required.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Chunked DELETE with LIMIT

```sql
-- Step 1: Create a table with old rows.
CREATE TABLE logs (id INT PRIMARY KEY, created_at DATE);
INSERT INTO logs SELECT seq, '2023-01-01' FROM seq_1_to_10000;

-- Step 2: Delete in chunks of 1,000 (run repeatedly).
DELETE FROM logs WHERE created_at < '2024-01-01' LIMIT 1000;

-- Step 3: Verify progress.
SELECT COUNT(*) AS remaining FROM logs WHERE created_at < '2024-01-01';
```

**Expected output (after one batch):**

```text
Query OK, 1000 rows affected (0.01 sec)
+-----------+
| remaining |
+-----------+
|      9000 |
+-----------+
```

**Why this output appears**: Each `DELETE ... LIMIT 1000` removes 1,000 rows and commits, releasing locks. Repeating the statement eventually removes all 10,000 rows without a single long-running transaction.

#### Real-World Cases

- **Log retention**: Old log entries are deleted in batches of 5,000 with a 100ms pause between batches to keep replication lag under control.
- **GDPR erasure**: User records are deleted in chunks after an erasure request, avoiding a table-wide lock that would block active users.

---

### 4. `COPY` (PostgreSQL)

#### Definitions

**Core Definition**
`COPY` is PostgreSQL's bulk-load primitive, a single SQL command that moves rows between a table and a file, program, STDIN, or STDOUT.

**Technical Definition**
`COPY` moves rows between a table and a file or memory area at approximately 10–100× the throughput of equivalent `INSERT` statements. Server-side `COPY` reads or writes files on the server filesystem and requires superuser or membership in `pg_read_server_files`, `pg_write_server_files`, or `pg_execute_server_program`. `COPY` is single-threaded per process; parallel loads require multiple concurrent `COPY` sessions against the same table. `COPY FREEZE` is intended as a performance option for initial data loading, freezing rows as if `VACUUM FREEZE` had been run, provided the table was created or truncated in the current transaction.

**Beginner-Friendly Explanation**
`COPY` is PostgreSQL's fastest way to load or unload data. Instead of inserting rows one by one, it streams them in bulk, like pouring water from a bucket instead of using a teaspoon.

#### Purposes

- To load millions of rows from a CSV file in seconds or minutes.
- To export query results to a file for archival or transfer.
- To transfer data between PostgreSQL databases using the binary format.
- To seed a new table with `COPY FREEZE` for optimal post-load performance.

#### Syntax Rules and Structure

**General syntax (PostgreSQL):**

```sql
COPY table_name [ ( column_name [, ...] ) ]
FROM { 'filename' | PROGRAM 'command' | STDIN }
[ [ WITH ] ( option [, ...] ) ];

COPY { table_name [ ( column_name [, ...] ) ] | ( query ) }
TO { 'filename' | PROGRAM 'command' | STDOUT }
[ [ WITH ] ( option [, ...] ) ];
```

**Key options:**

| Option | Description |
|--------|-------------|
| `FORMAT csv` | CSV format with delimiter, quote, and escape options. |
| `HEADER` | First line is a header; `HEADER MATCH` (PG15+) validates column names. |
| `FREEZE` | Freeze rows as if `VACUUM FREEZE` had run; requires the table to be created or truncated in the current transaction. |
| `ON_ERROR` (PG17+) | Error-tolerant ingest: `stop` (default), `ignore`, or `rollback`. |
| `REJECT_LIMIT` (PG18+) | Caps the number of rejected rows. |

**Syntax rules:**

- `COPY` is fastest when used within the same transaction as an earlier `CREATE TABLE` or `TRUNCATE` command.
- `COPY FREEZE` is only effective if the table was created or truncated in the current transaction.
- `COPY` cannot be run inside a transaction block if it uses certain options that require their own transaction.

**Constraints and limitations:**

- Server-side `COPY` requires file access on the server; use `\copy` or `COPY FROM STDIN` for client-side files.
- `COPY` is not suitable for regular OLTP traffic; use prepared `INSERT` for single-row writes.
- PG18 changed the treatment of `\.` in CSV server-side; older psql clients may experience `\copy` problems.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – COPY with FREEZE on a new table

```sql
-- Step 1: Begin a transaction, create the table, and COPY FREEZE.
BEGIN;
CREATE TABLE measurements (logdate date, peaktemp int, unitsales int);
COPY measurements FROM '/tmp/measurements.csv' WITH (FORMAT csv, FREEZE);
COMMIT;

-- Step 2: Verify.
SELECT count(*) FROM measurements;
```

**Expected output:**

```text
BEGIN
CREATE TABLE
COPY 5000000
COMMIT
 count
---------
 5000000
(1 row)
```

**Why this output appears**: `COPY FREEZE` marks the loaded rows as frozen, avoiding the need for a later `VACUUM FREEZE` and improving performance of subsequent queries and maintenance. The optimization works because the table was created in the same transaction.

#### Real-World Cases

- **Initial data load**: A new analytics table is created and populated with `COPY FREEZE` in one transaction, giving the fastest possible load and optimal post-load visibility.
- **Database migration**: Data is unloaded with `COPY ... TO` on the source and loaded with `COPY ... FROM` on the target, using the binary format for speed.

---

### 5. `\copy` (psql)

#### Definitions

**Core Definition**
`\copy` is a psql meta-command that performs a client-side copy, running an SQL `COPY` command but reading or writing the file on the client machine rather than the server.

**Technical Definition**
`\copy` performs a frontend (client) copy: it runs an SQL `COPY` command, but instead of the server reading or writing the specified file, psql reads or writes it and routes the data between the server and the local file system. This means `\copy` works through a normal psql connection without special server-side file privileges. The trade-off is that data must travel over the connection, so client-side `\copy` is generally slower than server-side `COPY`; to parallelise, split the data into multiple files and run separate `\copy` commands concurrently, which can be approximately 10× faster than a single `\copy`.

**Beginner-Friendly Explanation**
`\copy` is the version of `COPY` you use when the file is on your own computer rather than on the database server. It streams the data over the connection, so it works without special server permissions—but it is slower than server-side `COPY` because the data has to travel.

#### Purposes

- To load a local CSV file into a PostgreSQL table without server filesystem access.
- To export query results to a local file from a psql session.
- To parallelise a load by running multiple `\copy` commands against different chunks.

#### Syntax Rules and Structure

**General syntax (psql):**

```text
\copy table_name [ ( column_name [, ...] ) ]
FROM { 'filename' | PROGRAM 'command' | STDIN }
[ [ WITH ] ( option [, ...] ) ]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `\copy` | psql meta-command; not terminated by a semicolon. |
| `table_name` | Target or source table. |
| `FROM 'filename'` | Local file path on the client. |

**Syntax rules:**

- `\copy` is a psql meta-command, not a SQL statement; it does not require a semicolon.
- `\copy` can use the same options as `COPY` (e.g., `FORMAT csv`, `HEADER`).
- `\copy` does not require server-side file privileges, but the user must have `INSERT` or `SELECT` privilege on the table.

**Constraints and limitations:**

- `\copy` is slower than server-side `COPY` because data traverses the client-server connection.
- `\copy` is single-threaded; parallelise by splitting files and running multiple `\copy` commands.
- PG18 clients may have `\copy` issues with CSV files containing literal `\.`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: psql – \copy from a local CSV

```text
-- Step 1: Connect with psql.
$ psql -U postgres -d app

-- Step 2: Load the local file with \copy.
app=# \copy employees (id, name, salary) FROM 'employees.csv' WITH (FORMAT csv, HEADER true)

-- Step 3: Verify.
app=# SELECT count(*) FROM employees;
```

**Expected output:**

```text
COPY 100000
 count
--------
 100000
(1 row)
```

**Why this output appears**: `\copy` reads `employees.csv` on the client machine and streams the rows to the server, which inserts them into the `employees` table. The `COPY 100000` tag confirms the row count.

#### Real-World Cases

- **Developer workstation load**: A developer loads a local CSV into a shared development database without needing server filesystem access.
- **Parallel load**: A large CSV is split into 8 chunks, and 8 `\copy` commands run concurrently, achieving roughly 10× the throughput of a single `\copy`.

---

### 6. `BULK INSERT` (SQL Server)

#### Definitions

**Core Definition**
`BULK INSERT` is SQL Server's T-SQL statement for importing a data file into a table or view, running in-process with the database engine.

**Technical Definition**
`BULK INSERT` runs in-process with the SQL Server database engine and avoids passing data through the network layer of the client API, which makes it faster than `bcp` and SSIS in many scenarios. It supports options such as `FORMAT = 'CSV'`, `FIRSTROW`, `BATCHSIZE`, `ROWS_PER_BATCH`, `TABLOCK`, `CHECK_CONSTRAINTS`, `FIRE_TRIGGERS`, `KEEPIDENTITY`, and `KEEPNULLS`. Minimal logging is possible when the database is in `SIMPLE` or `BULK_LOGGED` recovery, the target table is not replicated, and `TABLOCK` is specified.

**Beginner-Friendly Explanation**
`BULK INSERT` is SQL Server's built-in bulk loader. It runs inside the database engine itself, so it does not have to send data through the client application, making it much faster than inserting rows one by one.

#### Purposes

- To import large CSV or flat files into a SQL Server table efficiently.
- To achieve minimal logging and reduce transaction-log growth during large loads.
- To batch commits with `BATCHSIZE` to control lock duration and log-space reservation.

#### Syntax Rules and Structure

**General syntax (SQL Server):**

```sql
BULK INSERT [ database_name . [ schema_name ] . | schema_name . ] table_name
FROM 'data_file'
[ WITH ( [ [ , ] BATCHSIZE = batch_size ]
          [ [ , ] CHECK_CONSTRAINTS ]
          [ [ , ] FIRE_TRIGGERS ]
          [ [ , ] FORMAT = 'CSV' ]
          [ [ , ] FIRSTROW = first_row ]
          [ [ , ] KEEPIDENTITY ]
          [ [ , ] KEEPNULLS ]
          [ [ , ] ROWS_PER_BATCH = rows_per_batch ]
          [ [ , ] TABLOCK ] ) ];
```

**Component breakdown:**

| Option | Description |
|--------|-------------|
| `BATCHSIZE` | Number of rows per batch; each batch is a separate transaction. |
| `TABLOCK` | Enables minimal logging when recovery model and replication conditions are met. |
| `CHECK_CONSTRAINTS` | Validates `CHECK` and `FOREIGN KEY` constraints during the load. |
| `FIRE_TRIGGERS` | Fires `INSERT` triggers on the target table. |
| `KEEPIDENTITY` | Uses the identity values from the data file instead of generating new ones. |
| `KEEPNULLS` | Inserts `NULL` for empty columns instead of the column default. |

**Syntax rules:**

- `BULK INSERT` does not skip header rows automatically; `FIRSTROW = 2` skips a row but does not validate its fields.
- `CHECK_CONSTRAINTS` should be specified if you need `CHECK` and `FOREIGN KEY` constraints validated during the load; otherwise they are ignored and marked untrusted.
- Minimal logging requires `SIMPLE` or `BULK_LOGGED` recovery, `TABLOCK`, and a table that is not replicated.

**Constraints and limitations:**

- `BULK INSERT` runs entirely on the server; the data file must be accessible from the SQL Server instance.
- `MAXERRORS` does not apply to constraint checks; a constraint violation aborts the batch.
- `BULK INSERT` is faster than `INSERT` but may still be slower than `bcp` for some workloads; benchmark both.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – BULK INSERT with minimal logging

```sql
-- Step 1: Create a heap table (no indexes) for minimal logging.
CREATE TABLE dbo.StageSales (SaleID int, Amount decimal(12,2), SaleDate date);

-- Step 2: Bulk-insert with TABLOCK and BATCHSIZE.
BULK INSERT dbo.StageSales
FROM 'C:\data\sales.csv'
WITH (
    FORMAT = 'CSV',
    FIRSTROW = 2,
    TABLOCK,
    BATCHSIZE = 10000,
    CHECK_CONSTRAINTS
);
```

**Expected output:**

```text
(10000 rows affected)
(10000 rows affected)
...
```

**Why this output appears**: `TABLOCK` enables minimal logging (under `SIMPLE` or `BULK_LOGGED` recovery and a non-replicated table), so only extent allocations are logged rather than individual row inserts. `BATCHSIZE = 10000` commits each batch separately, reducing log-space reservation and lock duration.

#### Real-World Cases

- **Data warehouse load**: A nightly job uses `BULK INSERT` with `TABLOCK` and `BATCHSIZE` to load a fact table from a flat file, minimising log growth and completion time.
- **Staging-table load**: Raw data is bulk-inserted into a staging heap with no indexes, then transformed and merged into the final table.

---

### 7. `LOAD DATA` (MySQL)

#### Definitions

**Core Definition**
`LOAD DATA INFILE` is MySQL's bulk-loading statement that reads rows from a text file into a table, bypassing SQL parsing and loading directly into the storage engine.

**Technical Definition**
`LOAD DATA INFILE` is the fastest single-threaded import method in MySQL, significantly faster than row-by-row `INSERT`s, and far faster than multi-row `INSERT` for large files. Without the `LOCAL` modifier, the server reads the file from the server host and the user needs the `FILE` privilege; with `LOCAL`, the client reads the file and sends its contents over the connection, which is slower but does not require the `FILE` privilege. MySQL's bulk-load extension supports primary-key-ordered loading, parallel threads (`PARALLEL`), ZSTD compression, and loading from object stores or file series.

**Beginner-Friendly Explanation**
`LOAD DATA` is MySQL's super-fast way to import a CSV file. Instead of parsing each `INSERT`, it reads the file directly into the table, which is much faster for large datasets.

#### Purposes

- To load millions of rows from a CSV file into a MySQL table.
- To import data from object storage (S3, URL) or a series of files.
- To achieve high throughput with parallel threads and memory tuning.

#### Syntax Rules and Structure

**General syntax (MySQL):**

```sql
LOAD DATA [LOW_PRIORITY | CONCURRENT] [LOCAL] INFILE 'file_name'
[REPLACE | IGNORE]
INTO TABLE tbl_name
[PARTITION (partition_name [, ...])]
[CHARACTER SET charset_name]
[{FIELDS | COLUMNS}
    [TERMINATED BY 'string']
    [[OPTIONALLY] ENCLOSED BY 'char']
    [ESCAPED BY 'char']
]
[LINES
    [STARTING BY 'string']
    [TERMINATED BY 'string']
]
[IGNORE number {LINES | ROWS}]
[(col_name_or_user_var [, ...])]
[SET col_name = expr [, ...]];
```

**Bulk-load extension (MySQL AI/HeatWave):**

```sql
LOAD DATA [FROM] INFILE | URL | S3 'file_prefix'
[COUNT N]
[IN PRIMARY KEY ORDER]
INTO TABLE tbl_name
[PARALLEL = number]
[MEMORY = M]
[ALGORITHM = BULK];
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `LOCAL` | Reads the file on the client; slower but no `FILE` privilege needed. |
| `PARALLEL` | Number of concurrent threads (default 16 in bulk-load extension). |
| `MEMORY` | Memory to use for the bulk load (e.g., `512M`, `4G`). |
| `IN PRIMARY KEY ORDER` | Optimises loading when data is already sorted by primary key. |
| `IGNORE 1 ROWS` | Skips a header row. |

**Syntax rules:**

- Server-side `LOAD DATA INFILE` requires the `FILE` privilege and the file must be on the server.
- `LOCAL` is slower because the client sends the file over the connection.
- For InnoDB tables, temporarily disabling `unique_checks` and `foreign_key_checks` can speed up the load; for MyISAM, `DISABLE KEYS` is effective, but it has no effect on InnoDB.

**Constraints and limitations:**

- `LOCAL` works only if both the server and the client are configured to permit it.
- `LOAD DATA` interprets all fields as the same character set, regardless of column data types.
- Very large loads can be memory-intensive; tune `MEMORY` and batch the file series.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – LOAD DATA INFILE from a server-side CSV

```sql
-- Step 1: Create a table.
CREATE TABLE events (user_id INT, event_type VARCHAR(50), created_at DATETIME);

-- Step 2: Load the server-side CSV.
LOAD DATA INFILE '/tmp/events.csv'
INTO TABLE events
FIELDS TERMINATED BY ',' ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(user_id, event_type, created_at);

-- Step 3: Verify.
SELECT COUNT(*) FROM events;
```

**Expected output:**

```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 1000000 rows affected (2.15 sec)
Records: 1000000  Deleted: 0  Skipped: 0  Warnings: 0
+----------+
| COUNT(*) |
+----------+
|  1000000 |
+----------+
```

**Why this output appears**: `LOAD DATA INFILE` reads the CSV on the server and loads all 1,000,000 rows directly into the InnoDB storage engine, bypassing SQL parsing. The `Records: 1000000` line confirms the row count.

#### Real-World Cases

- **Bulk ingestion from S3**: The bulk-load extension loads CSV files directly from S3 using `S3 's3://bucket/prefix'` with `PARALLEL = 32` and `MEMORY = 4G`, achieving high throughput without intermediate copies.
- **Application log import**: Daily log files are loaded into a MySQL analytics table with `LOAD DATA INFILE` and `IGNORE 1 ROWS` for the header.

---

### 8. Staging Tables

#### Definitions

**Core Definition**
A staging table is an intermediate table used to hold raw data temporarily during an ETL or bulk-loading process, before the data is validated, transformed, and merged into the final target table.

**Technical Definition**
Staging tables isolate ETL failures from the production warehouse, support troubleshooting and auditing, and enable reprocessing without re-extracting the source data. Best practices include minimising indexes on staging tables (because indexes slow down writes) and using partitioned staging tables for high-volume ingestion to simplify management and purging. The typical workflow is: bulk-load raw data into the staging table, transform it in place or in a query, then apply it to the target with `MERGE`, `INSERT ... SELECT`, or `UPDATE ... FROM`.

**Beginner-Friendly Explanation**
A staging table is a waiting room for data. Raw data is loaded there first, checked and cleaned, and only then moved into the real table. If something goes wrong, you can fix the staging data and try again without touching production.

#### Purposes

- To isolate ETL failures from the production warehouse.
- To enable auditing and reprocessing of raw data.
- To improve performance by loading into a heap with minimal indexes, then transforming and merging.
- To support incremental loads using watermarks or CDC.

#### Syntax Rules and Structure

**Typical staging workflow (SQL Server):**

```sql
-- Step 1: Create a staging heap (no indexes).
CREATE TABLE dbo.Staging (Key1 int, Value1 int);

-- Step 2: Bulk-load raw data into the staging table.
BULK INSERT dbo.Staging FROM 'C:\data\batch.csv' WITH (TABLOCK, BATCHSIZE = 10000);

-- Step 3: Merge the staged data into the target.
MERGE dbo.Target AS T
USING dbo.Staging AS S
ON T.Key1 = S.Key1
WHEN MATCHED THEN UPDATE SET T.Value1 = S.Value1
WHEN NOT MATCHED THEN INSERT (Key1, Value1) VALUES (S.Key1, S.Value1);

-- Step 4: Truncate the staging table for the next batch.
TRUNCATE TABLE dbo.Staging;
```

**Component breakdown:**

| Step | Action | Purpose |
|------|--------|---------|
| Create staging heap | No indexes | Fast writes |
| Bulk-load | `BULK INSERT` / `COPY` / `LOAD DATA` | High-throughput ingest |
| Transform + merge | `MERGE`, `INSERT ... SELECT` | Apply changes to target |
| Truncate | `TRUNCATE TABLE` | Reset for next batch |

**Syntax rules:**

- Staging tables should have as few indexes as possible; indexes slow down bulk loads.
- Use partitioned staging tables for high-volume ingestion to simplify purging.
- The merge step should be set-based (`MERGE` or `INSERT ... SELECT`), not row-by-row.

**Constraints and limitations:**

- `MERGE` against a large target can be slow; batch the merge in the 50,000–200,000 row range if blocking is observed.
- Staging tables consume additional storage; truncate or drop them promptly after use.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Staging table + MERGE

```sql
-- Step 1: Create a staging table and a target table.
CREATE TABLE dbo.Staging (Key1 int PRIMARY KEY, Value1 int);
CREATE TABLE dbo.Target (Key1 int PRIMARY KEY, Value1 int);
INSERT INTO dbo.Target VALUES (1, 100);

-- Step 2: Load staged data (simulated).
INSERT INTO dbo.Staging VALUES (1, 200), (2, 300);

-- Step 3: Merge the staged data into the target.
MERGE dbo.Target AS T
USING dbo.Staging AS S
ON T.Key1 = S.Key1
WHEN MATCHED THEN UPDATE SET T.Value1 = S.Value1
WHEN NOT MATCHED THEN INSERT (Key1, Value1) VALUES (S.Key1, S.Value1);

-- Step 4: Verify.
SELECT * FROM dbo.Target ORDER BY Key1;

-- Step 5: Truncate the staging table.
TRUNCATE TABLE dbo.Staging;
```

**Expected output:**

```text
Key1  Value1
----  ------
1     200
2     300
```

**Why this output appears**: The `MERGE` updates the existing `Key1 = 1` row with the staged value `200` and inserts the new `Key1 = 2` row with value `300`. The staging table is then truncated for the next batch.

#### Real-World Cases

- **Nightly product sync**: A staging table receives a full product feed; a `MERGE` updates changed products, inserts new ones, and a follow-up statement deactivates products absent from the feed.
- **Partitioned staging**: A high-volume clickstream is loaded into a partitioned staging table, transformed, and merged into the fact table, with old partitions dropped instead of deleted.

---

### 9. Batch Processing

#### Definitions

**Core Definition**
Batch processing is the practice of grouping many row-level operations into a smaller number of set-based statements or transactions to reduce per-operation overhead and control resource usage.

**Technical Definition**
Batch processing in SQL covers three related techniques: (1) **multi-row `INSERT`** to amortise statement overhead; (2) **chunked `UPDATE`/`DELETE`** to avoid lock escalation and log bloat; and (3) **transaction batching** to amortise commit overhead. Processing rows one at a time is expensive because each statement incurs network round-trip overhead and, with auto-commit, a full transaction commit per row. The standard guidance is to batch `INSERT` in chunks of 500–1,000 rows, chunk `UPDATE`/`DELETE` in chunks of 5,000 rows (fewer if the table has many indexes), and wrap batches in explicit transactions for atomicity.

**Beginner-Friendly Explanation**
Batch processing means doing work in groups instead of one item at a time. Sending 1,000 inserts as one statement, or deleting 5,000 rows per transaction instead of 5 million at once, makes the database much faster and keeps it responsive.

#### Purposes

- To reduce network round-trips and per-statement overhead.
- To avoid lock escalation, log-space exhaustion, and replication lag.
- To maintain atomicity within each batch while allowing partial progress across batches.

#### Syntax Rules and Structure

**Multi-row INSERT batch (MySQL/PostgreSQL):**

```sql
INSERT INTO events (user_id, event_type, created_at) VALUES
    (1, 'login', NOW()),
    (2, 'signup', NOW()),
    ...;
```

**Chunked UPDATE (SQL Server):**

```sql
WHILE 1 = 1
BEGIN
    UPDATE TOP (5000) dbo.Target SET col = value WHERE condition;
    IF @@ROWCOUNT < 5000 BREAK;
END;
```

**Transaction batching (MySQL):**

```sql
START TRANSACTION;
INSERT INTO audit_log (user_id, action) VALUES (1, 'view');
INSERT INTO audit_log (user_id, action) VALUES (2, 'edit');
-- ... up to a few thousand rows
COMMIT;
```

**Component breakdown:**

| Technique | Typical batch size | Purpose |
|-----------|-------------------|---------|
| Multi-row `INSERT` | 500–1,000 rows | Amortise parsing/network overhead |
| Chunked `UPDATE`/`DELETE` | 5,000 rows (fewer with many indexes) | Avoid lock escalation and log bloat |
| Transaction batching | 1,000–10,000 operations | Amortise commit overhead |

**Syntax rules:**

- Chunk size should start at 5,000 rows; reduce proportionally if the table has many indexes, because each modified row acquires one lock per index.
- Each chunk should be its own transaction (autocommit) unless atomicity across chunks is required; in that case, wrap the chunk in an explicit transaction.
- Avoid extremely large transactions (millions of rows) because they hold locks and consume undo log space.

**Constraints and limitations:**

- `LIMIT` without an indexed `WHERE` clause can cause full-table scans and defeat the purpose of batching.
- `TOP` without a deterministic `ORDER BY` or key-based `WHERE` can miss or revisit rows across iterations.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Multi-row INSERT batching

```sql
-- Step 1: Create a table.
CREATE TABLE events (user_id INT, event_type VARCHAR(50), created_at DATETIME);

-- Step 2: Insert 1,000 rows in a single statement (values simplified).
INSERT INTO events (user_id, event_type, created_at) VALUES
    (1, 'login', NOW()), (2, 'signup', NOW()), (3, 'logout', NOW()),
    ...;  -- 1,000 tuples in total

-- Step 3: Verify.
SELECT COUNT(*) FROM events;
```

**Expected output:**

```text
Query OK, 1000 rows affected (0.01 sec)
Records: 1000  Duplicates: 0  Warnings: 0
+----------+
| COUNT(*) |
+----------+
|     1000 |
+----------+
```

**Why this output appears**: All 1,000 rows are inserted in a single statement, amortising parsing and network overhead. `Records: 1000` confirms the batch size.

#### Real-World Cases

- **Application event logging**: Events are buffered and flushed in batches of 500 rows, reducing database round-trips and improving application throughput.
- **Nightly purge**: Old rows are deleted in chunks of 5,000 with a 100ms pause between chunks, keeping replication lag and lock contention low.

---

### 10. Performance Optimization

#### Definitions

**Core Definition**
Performance optimization for bulk operations is the set of techniques—minimal logging, index management, parallelism, memory tuning, and batch sizing—used to maximise throughput and minimise resource contention during large data loads or modifications.

**Technical Definition**
Key optimization techniques include: (1) **minimal logging**—SQL Server minimally logs bulk imports under `SIMPLE`/`BULK_LOGGED` recovery with `TABLOCK` and a non-replicated table; PostgreSQL `COPY FREEZE` avoids WAL replay overhead for initial loads; (2) **index management**—temporarily dropping or disabling non-unique indexes and foreign keys before a load and recreating them afterwards can accelerate high-volume loads by an order of magnitude; (3) **parallelism**—`COPY` is single-threaded, so split input into N chunks and run N concurrent `COPY` sessions, which can be ~10× faster than a single `\copy`; (4) **memory and threading**—MySQL's bulk-load extension provides `PARALLEL`, `MEMORY`, and `IN PRIMARY KEY ORDER` options for tuning; and (5) **batch sizing**—chunk DML to 5,000 rows (fewer with many indexes) and monitor `WRITELOG` waits to detect log I/O bottlenecks.

**Beginner-Friendly Explanation**
Performance optimization for bulk operations is about removing bottlenecks. Drop indexes during a big load and rebuild them afterward, split a huge file into pieces and load them in parallel, turn off minimal logging where it is safe, and choose a batch size that keeps the database fast without locking it up.

#### Purposes

- To complete bulk loads within maintenance windows or batch SLAs.
- To avoid excessive transaction-log growth and replication lag.
- To minimise lock contention and keep concurrent workloads responsive.

#### Syntax Rules and Structure

**PostgreSQL `COPY FREEZE`:**

```sql
BEGIN;
CREATE TABLE new_table (...);
COPY new_table FROM '/path/file.csv' WITH (FORMAT csv, FREEZE);
COMMIT;
```

**SQL Server minimal logging:**

```sql
BULK INSERT dbo.Target
FROM 'C:\data\file.csv'
WITH (TABLOCK, BATCHSIZE = 10000);
```

**MySQL parallel bulk load:**

```sql
LOAD DATA INFILE 's3://bucket/prefix'
INTO TABLE target
PARALLEL = 32
MEMORY = '4G'
ALGORITHM = BULK;
```

**Index management (PostgreSQL):**

```sql
-- Before load:
DROP INDEX idx_target_col;
-- Load with COPY.
-- After load:
CREATE INDEX idx_target_col ON target (col);
```

**Component breakdown:**

| Technique | RDBMS | Impact |
|-----------|-------|--------|
| `COPY FREEZE` | PostgreSQL | Avoids WAL replay for initial loads |
| `TABLOCK` + `SIMPLE`/`BULK_LOGGED` | SQL Server | Minimal logging; reduces log volume |
| Drop indexes before load | All | Faster writes; rebuild afterward |
| Parallel `COPY` / `\copy` | PostgreSQL | ~10× faster than a single session |
| `PARALLEL`, `MEMORY` | MySQL (bulk extension) | Concurrent threads and memory tuning |
| Chunk DML | All | Avoids lock escalation and log bloat |

**Syntax rules:**

- `COPY FREEZE` requires the table to be created or truncated in the current transaction.
- SQL Server minimal logging requires `SIMPLE` or `BULK_LOGGED` recovery, `TABLOCK`, and a non-replicated table; transactional replication forces full logging even under `BULK_LOGGED`.
- MySQL `DISABLE KEYS` only affects MyISAM; for InnoDB, use `SET unique_checks = 0` and `SET foreign_key_checks = 0` instead.

**Constraints and limitations:**

- Dropping indexes during a load requires exclusive access to the table and may not be possible in a highly concurrent environment.
- Parallel `COPY` sessions contend on the same heap and commit latency; measure before and after.
- Minimal logging is not available under the `FULL` recovery model (except for certain columnstore cases).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Parallel COPY for a large load

```bash
# Step 1: Split the input file into 8 chunks.
split -n l/8 large_file.csv chunk_

# Step 2: Run 8 COPY sessions in parallel.
for f in chunk_*; do
  psql -U postgres -d app -c "COPY target FROM '$PWD/$f' WITH (FORMAT csv)" &
done
wait
```

**Expected output:**

```text
COPY 1250000
COPY 1250000
...
```

**Why this output appears**: Each `COPY` session loads one chunk independently, and the sessions run concurrently. Because `COPY` is single-threaded per process, the aggregate throughput is roughly proportional to the number of parallel sessions, up to I/O limits.

##### Example 2: SQL Server – Drop indexes, BULK INSERT, rebuild indexes

```sql
-- Step 1: Drop nonclustered indexes before the load.
DROP INDEX ix_target_col ON dbo.Target;

-- Step 2: BULK INSERT with TABLOCK and BATCHSIZE.
BULK INSERT dbo.Target
FROM 'C:\data\large.csv'
WITH (FORMAT = 'CSV', FIRSTROW = 2, TABLOCK, BATCHSIZE = 10000);

-- Step 3: Rebuild the index.
CREATE INDEX ix_target_col ON dbo.Target (col);
```

**Expected output:**

```text
(10000 rows affected)
(10000 rows affected)
...
```

**Why this output appears**: Dropping the index removes the per-row index-maintenance overhead during the load. Rebuilding the index afterward is much faster than maintaining it incrementally for millions of rows.

#### Real-World Cases

- **Data warehouse nightly load**: Indexes on the fact table are dropped before the bulk load and rebuilt afterward, and `TABLOCK` is used to enable minimal logging under `BULK_LOGGED` recovery.
- **PostgreSQL initial population**: A new database is seeded with `COPY FREEZE` inside a single transaction, avoiding `VACUUM` overhead and improving post-load query performance.

---

## References

- PostgreSQL Documentation – COPY – https://www.postgresql.org/docs/current/sql-copy.html
- PostgreSQL Documentation – Populating a Database – https://www.postgresql.org/docs/current/populate.html
- PostgreSQL Documentation – COPY FREEZE (git source) – https://git.postgresql.org/gitweb/?p=postgresql.git;a=blob_plain;f=doc/src/sgml/ref/copy.sgml
- PostgreSQL Bulk Operations: COPY (skill reference) – https://github.com/damusix/skills/blob/main/postgres/references/66-bulk-operations-copy.md
- Using the COPY command with Aurora PostgreSQL Limitless Database – https://docs.aws.amazon.com/AmazonAurora/latest/AuroraUserGuide/limitless-copy.html
- Microsoft Learn – BULK INSERT (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/bulk-insert-transact-sql
- Microsoft Learn – Prerequisites for Minimal Logging in Bulk Import – https://learn.microsoft.com/en-us/sql/relational-databases/import-export/prerequisites-for-minimal-logging-in-bulk-import
- Microsoft Learn – Batch Operations (SQL Server performance reference) – https://github.com/damusix/skills/blob/main/sql-server-performance/references/batch-operations.md
- MySQL 8.0 Reference Manual – LOAD DATA Statement – https://dev.mysql.com/doc/refman/8.0/en/load-data.html
- MySQL 8.0 Reference Manual – LOAD DATA LOCAL – https://dev.mysql.com/doc/refman/8.0/en/load-data-local.html
- MySQL AI User Guide – Bulk Ingest Data – https://docs.oracle.com/cd/E17952_01/mysql-ai-9.7-en/mys-ai-bulk-load-mysql-server.html
- SQLiz – SQL Bulk and Batch Insert by Database: Syntax, Limits & Performance – https://www.sqliz.com/posts/bulk-insert-in-sql-by-database/
- OneUptime Blog – How to Implement Batch Processing in MySQL – https://github.com/OneUptime/blog/blob/master/posts/2026-03-31-mysql-batch-processing/README.md
- Integrate.io – 12 Best SQL Server ETL Best Practices – https://www.integrate.io/blog/12-best-practices-for-high-performance-data-pipelines/
- Microsoft Q&A – Optimizing Bulk Merge into Azure SQL Hyperscale Without Staging Tables – https://learn.microsoft.com/en-us/answers/questions/…
- CSV to SQL: Import Spreadsheet Data to Any Database (2026 Guide) – https://www.importcsv.com/
- MySQL 8.0 Reference Manual – INSERT Statement – https://dev.mysql.com/doc/refman/8.0/en/insert.html