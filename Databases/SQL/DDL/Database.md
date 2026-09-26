# SQL Database Management: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL database management is the set of administrative operations—creating, selecting, listing, renaming, dropping, owning, configuring, and securing databases—that control the lifecycle and behavior of database instances within a Database Management System (DBMS).

**Technical Definition:** SQL database management encompasses the Data Definition Language (DDL) and Data Control Language (DCL) statements that operate at the database level rather than the table or row level. These operations include `CREATE DATABASE` (with file specification, sizing, and growth parameters), session-level database selection, metadata queries against system catalogs (`sys.databases`, `information_schema`), `ALTER DATABASE` for renaming and configuration changes, `DROP DATABASE` with connection handling, ownership assignment via `ALTER AUTHORIZATION` or `OWNER TO`, recovery model and compatibility level configuration, encryption enablement through TDE, and database-scoped privilege management via `GRANT`/`REVOKE`/`DENY`.

**Beginner-Friendly Explanation:** Database management is like being the landlord of an apartment building. You decide how many apartments (databases) to build, who gets the keys (permissions), what the rules are (configuration), and what happens when someone moves out (drop). You also keep a directory of all apartments (listing databases) and can renovate or rename them as needed.

### Key Characteristics

- **Instance-scoped:** Database-level operations affect an entire database, not individual tables or rows.
- **Vendor-specific:** Syntax and capabilities vary significantly across PostgreSQL, MySQL, SQL Server, and Oracle.
- **Storage-aware:** `CREATE DATABASE` in SQL Server and Oracle requires explicit file specification (data files, log files, filegroups, tablespaces).
- **Connection-sensitive:** Renaming and dropping databases require exclusive access or special connection states (`SINGLE_USER`, `pg_terminate_backend`).
- **Configuration-rich:** Recovery models, compatibility levels, collation, and encryption are database-scoped settings.
- **Privilege-controlled:** Creating, altering, and dropping databases require elevated privileges (`CREATEDB`, `SYSDBA`, `dbcreator` role).

### Prerequisites

- Understanding of the relational model and SQL DDL/DCL statements.
- Familiarity with the target DBMS's system catalogs and metadata views.
- Knowledge of file system paths, storage sizing, and growth planning (for SQL Server and Oracle).
- Awareness of backup and recovery concepts (recovery models, TDE).
- Basic understanding of role-based access control.

### Related Programming Areas

- Database administration (DBA) and DevOps.
- Application deployment and environment provisioning.
- Data warehousing and ETL infrastructure.
- Security and compliance (encryption, auditing).
- Cloud database management (RDS, Aurora, Azure SQL).

### Core Concepts / Features

1. **Creating Databases** (syntax, file locations, sizing, auto-growth, templates)
2. **Selecting Databases** (USE, connection strings, environment settings)
3. **Listing Databases** (system catalogs, CLI utilities)
4. **Renaming Databases** (SINGLE_USER, active connections, ALTER DATABASE)
5. **Dropping Databases** (DROP DATABASE, IF EXISTS, FORCE)
6. **Database Ownership** (ALTER AUTHORIZATION, OWNER TO)
7. **Database Configuration** (recovery models, collation, compatibility levels, TDE)
8. **Database-Level Permissions** (GRANT, REVOKE, DENY)


## Core Concept 1: Creating Databases

### Definitions

**Core Definition:** Creating a database is the operation of instantiating a new, named database instance within a DBMS using the `CREATE DATABASE` statement, optionally specifying storage files, sizing parameters, character encoding, and ownership.

**Technical Definition:** `CREATE DATABASE` is a DDL statement that creates a new database. In SQL Server and Oracle, the statement requires explicit specification of data files (`.mdf`/`.ndf`, `.dbf`) and log files (`.ldf`, redo logs), including initial size, maximum size, and auto-growth increment. In PostgreSQL, `CREATE DATABASE` works by copying an existing template database (default: `template1`) and supports options for encoding, collation, owner, and tablespace. MySQL and MariaDB support `CREATE DATABASE` with character set and collation options. Oracle's `CREATE DATABASE` is a complex statement requiring an initialization parameter file and `SYSDBA` privilege.

**Beginner-Friendly Explanation:** Creating a database is like building a new filing cabinet. You give it a name, decide how much space it gets (initial size), how much it can grow (auto-growth), where the files live (file locations), and what language rules it follows (collation). In SQL Server and Oracle, you also choose the physical files; in PostgreSQL, you copy an existing empty database as a starting point.

### Purposes

- To provision a new database instance for a new application, tenant, or environment.
- To control physical storage characteristics (file locations, sizing, growth) for performance and capacity planning.
- To set character encoding and collation at creation time, ensuring correct text handling.
- To clone an existing database structure and settings using a template.

### Syntax Rules and Structure

#### Complete General Syntax (SQL Server)

```sql
CREATE DATABASE database_name
[ ON
    [ PRIMARY ] <filespec> [ ,...n ]
    [ , <filegroup> [ ,...n ] ]
    [ LOG ON <filespec> [ ,...n ] ]
]
[ COLLATE collation_name ]
[ WITH <option> [ ,...n ] ];

<filespec> ::= {
    ( NAME = logical_file_name,
      FILENAME = 'os_file_name'
      [ , SIZE = size [ KB | MB | GB | TB ] ]
      [ , MAXSIZE = { max_size [ KB | MB | GB | TB ] | UNLIMITED } ]
      [ , FILEGROWTH = growth_increment [ KB | MB | GB | TB | % ] ] )
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ON` | Defines the data files. |
| `PRIMARY` | Designates the primary filegroup. |
| `<filespec>` | One or more file specifications with `NAME`, `FILENAME`, `SIZE`, `MAXSIZE`, and `FILEGROWTH`. |
| `LOG ON` | Defines the transaction log file(s). |
| `COLLATE` | Sets the database collation. |
| `SIZE` | Initial size of the file. |
| `FILEGROWTH` | Growth increment when the file fills up. |

#### Complete General Syntax (PostgreSQL)

```sql
CREATE DATABASE name
    [ [ WITH ] [ OWNER [=] user_name ]
           [ TEMPLATE [=] template ]
           [ ENCODING [=] encoding ]
           [ LOCALE [=] locale ]
           [ LC_COLLATE [=] lc_collate ]
           [ LC_CTYPE [=] lc_ctype ]
           [ TABLESPACE [=] tablespace_name ]
           [ ALLOW_CONNECTIONS [=] allowconn ]
           [ CONNECTION LIMIT [=] connlimit ] ]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `OWNER` | The role that will own the new database. |
| `TEMPLATE` | The template database to copy (default: `template1`). |
| `ENCODING` | Character set encoding (e.g., `'UTF8'`). |
| `LC_COLLATE` / `LC_CTYPE` | Collation order and character classification. |
| `TABLESPACE` | Default tablespace for the database. |
| `CONNECTION LIMIT` | Maximum concurrent connections (`-1` = unlimited). |

#### Complete General Syntax (MySQL)

```sql
CREATE DATABASE [IF NOT EXISTS] database_name
    [CHARACTER SET charset_name]
    [COLLATE collation_name];
```

#### Complete General Syntax (Oracle)

```sql
CREATE DATABASE database_name
    USER SYS IDENTIFIED BY sys_password
    USER SYSTEM IDENTIFIED BY system_password
    LOGFILE GROUP 1 ('/path/log1.rdo') SIZE 100M,
            GROUP 2 ('/path/log2.rdo') SIZE 100M
    MAXLOGFILES 5
    MAXLOGMEMBERS 5
    MAXDATAFILES 100
    MAXINSTANCES 1
    CHARACTER SET AL32UTF8
    NATIONAL CHARACTER SET AL16UTF16
    DATAFILE '/path/system01.dbf' SIZE 500M
    SYSAUX DATAFILE '/path/sysaux01.dbf' SIZE 300M
    DEFAULT TABLESPACE users
        DATAFILE '/path/users01.dbf' SIZE 100M
    DEFAULT TEMPORARY TABLESPACE temp
        TEMPFILE '/path/temp01.dbf' SIZE 50M
    UNDO TABLESPACE undotbs
        DATAFILE '/path/undotbs01.dbf' SIZE 200M;
```

#### Syntax Rules

- **SQL Server:** If `SIZE` is not specified, the default is the size of the `model` database's primary file. If `FILEGROWTH` is not specified, the default is 1 MB for data files and 10% for log files, with a minimum of 64 KB.
- **SQL Server:** If `LOG ON` is not specified, a single log file is automatically created at 25% of the sum of all data file sizes (minimum 512 KB).
- **PostgreSQL:** `CREATE DATABASE` works by copying an existing database. By default, it copies `template1`. To create a "virgin" database, use `TEMPLATE template0`.
- **PostgreSQL:** The `template1` database is the template from which new databases are made. If you add objects to `template1`, they will be copied into subsequently created user databases.
- **MySQL:** `IF NOT EXISTS` prevents an error if the database already exists.
- **Oracle:** Requires `SYSDBA` privilege and must be in `STARTUP NOMOUNT` mode. An initialization parameter file matching the database name must exist.

#### Constraints and Limitations

- **SQL Server:** The `database_name` can be up to 128 characters; if a logical log file name is not specified, the limit is 123 characters.
- **PostgreSQL:** There is no option to use a database locale with nondeterministic comparisons.
- **PostgreSQL:** The character set encoding specified for the new database must be compatible with the chosen locale settings (`LC_COLLATE` and `LC_CTYPE`).
- **Oracle:** The database name can be up to 8 bytes long and must match the `DB_NAME` initialization parameter.
- **MySQL:** `CREATE DATABASE` is a MySQL extension to the SQL standard.

### Annotated Code Examples

#### Example 1: SQL Server — CREATE DATABASE with File Specification

```sql
-- Create a database with explicit data and log files
CREATE DATABASE EmployeesDB
ON PRIMARY (
    NAME = 'EmployeesDB_Data',
    FILENAME = 'D:\SQLServer\Data\EmployeesDB.mdf',
    SIZE = 100MB,
    MAXSIZE = 1GB,
    FILEGROWTH = 25MB
),
FILEGROUP SecondaryFG (
    NAME = 'EmployeesDB_Data2',
    FILENAME = 'E:\SQLServer\Data\EmployeesDB_2.ndf',
    SIZE = 50MB,
    FILEGROWTH = 10MB
)
LOG ON (
    NAME = 'EmployeesDB_Log',
    FILENAME = 'D:\SQLServer\Log\EmployeesDB_Log.ldf',
    SIZE = 50MB,
    MAXSIZE = 500MB,
    FILEGROWTH = 25MB
)
COLLATE SQL_Latin1_General_CP1_CI_AS;
```

**Expected Output:**

```
Commands completed successfully.
```

**Why This Works:** The `ON PRIMARY` clause defines the primary data file with an initial size of 100 MB, a maximum of 1 GB, and 25 MB auto-growth. The `FILEGROUP SecondaryFG` clause creates a secondary filegroup for additional data files. `LOG ON` defines the transaction log. The `COLLATE` clause sets the database collation.

#### Example 2: PostgreSQL — CREATE DATABASE with Encoding and Template

```sql
-- Create a database with UTF-8 encoding, owned by a specific role,
-- copying template0 to avoid site-local additions
CREATE DATABASE appdb
    WITH OWNER = app_user
         TEMPLATE = template0
         ENCODING = 'UTF8'
         LC_COLLATE = 'en_US.UTF-8'
         LC_CTYPE = 'en_US.UTF-8'
         CONNECTION LIMIT = 100;

-- Verify
SELECT datname, pg_encoding_to_char(encoding) AS encoding,
       datcollate, datconnlimit
FROM pg_database
WHERE datname = 'appdb';
```

**Expected Output:**

```
 datname | encoding | datcollate  | datconnlimit
---------+----------+-------------+--------------
 appdb   | UTF8     | en_US.UTF-8 |          100
```

**Why This Works:** `TEMPLATE = template0` creates a "virgin" database without any site-local objects that may have been added to `template1`. `ENCODING = 'UTF8'` sets the character set. `CONNECTION LIMIT = 100` restricts concurrent connections.

### Real-World Cases

- **Multi-tenant SaaS:** Creating a separate database per tenant with isolated storage and connection limits.
- **Environment provisioning:** Creating development, staging, and production databases with appropriate sizing.
- **Data warehousing:** Creating a database with a large initial size and aggressive auto-growth for analytics workloads.
- **Compliance:** Creating a database with a specific collation to meet regulatory text-handling requirements.

### References

- SQL Server: CREATE DATABASE (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-database-transact-sql
- PostgreSQL: CREATE DATABASE — https://www.postgresql.org/docs/current/sql-createdatabase.html
- PostgreSQL: Template Databases — https://www.postgresql.org/docs/current/manage-ag-templatedbs.html
- MySQL: CREATE DATABASE — https://dev.mysql.com/doc/refman/8.0/en/create-database.html
- Oracle Database: CREATE DATABASE — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CREATE-DATABASE.html


## Core Concept 2: Selecting Databases

### Definitions

**Core Definition:** Selecting a database is the operation of switching the active database context within a session, so that subsequent unqualified object references resolve against that database.

**Technical Definition:** Database selection mechanisms vary by vendor. SQL Server uses the `USE database_name` statement. MySQL uses `USE database_name`. PostgreSQL does not support switching databases within an existing connection; the database must be specified at connection time (via connection string parameters or the `\c` psql meta-command, which opens a new connection). Oracle uses `ALTER SESSION SET CURRENT_SCHEMA = schema_name` to change the default schema (Oracle does not have a database-level selection concept; a connection is always to a single database instance). Applications typically specify the database in connection strings (e.g., JDBC URL `jdbc:postgresql://host:port/dbname`).

**Beginner-Friendly Explanation:** Selecting a database is like choosing which filing cabinet to work with. In SQL Server and MySQL, you say "USE this database" and all subsequent commands go to that cabinet. In PostgreSQL, you have to open a new connection to a different cabinet—you cannot just switch within the same session.

### Purposes

- To direct subsequent SQL statements to the intended database without qualifying every object name.
- To isolate application workloads to specific databases.
- To enable multi-database administration from a single client session (where supported).

### Syntax Rules and Structure

#### Complete General Syntax (SQL Server)

```sql
USE database_name;
```

#### Complete General Syntax (MySQL)

```sql
USE database_name;
```

#### Complete General Syntax (PostgreSQL — connection-level)

```sql
-- In psql, use the \c meta-command (opens a new connection)
\c database_name

-- In application connection strings:
-- jdbc:postgresql://host:port/database_name
-- host=localhost port=5432 dbname=database_name
```

#### Complete General Syntax (Oracle — schema-level)

```sql
ALTER SESSION SET CURRENT_SCHEMA = schema_name;
```

#### Syntax Rules

- **SQL Server:** `USE` is valid in T-SQL batches; it is commonly used in scripts to switch context between databases (e.g., `USE master;` before `DROP DATABASE`).
- **MySQL:** `USE` is a client command that can also be executed as `USE database_name;`.
- **PostgreSQL:** You cannot switch databases within a connection. The `\c` command in `psql` disconnects and reconnects to the specified database. In embedded SQL (ECPG), `EXEC SQL CONNECT TO target` establishes a connection to a specific database.
- **PostgreSQL connection strings:** Support URI format (`postgresql://host:port/dbname?options`) or keyword/value format (`host=... dbname=...`).
- **Oracle:** `ALTER SESSION SET CURRENT_SCHEMA` changes the default schema for unqualified object resolution, not the database (Oracle connections are always to a single database instance).

#### Constraints and Limitations

- **PostgreSQL:** In PostgreSQL you have to specify the database when you open a connection to the server; you cannot switch to a different database using SQL from `psql` (use `\connect` instead).
- **SQL Server:** `USE` cannot be used inside a stored procedure or function to change the database context for the caller.
- **MySQL:** `USE` has no effect on the permissions of the session; permissions are checked per-database.
- **Oracle:** There is no `USE DATABASE` statement; the connection is always to the instance specified in the connect string.

### Annotated Code Examples

#### Example 1: SQL Server — Switching Database Context

```sql
-- Connect to master (default)
USE master;
GO

-- Switch to a specific database
USE EmployeesDB;
GO

-- Now unqualified references resolve against EmployeesDB
SELECT COUNT(*) AS employee_count FROM dbo.Employees;
```

**Expected Output:**

```
employee_count
--------------
          1500
```

**Why This Works:** `USE EmployeesDB` changes the database context for the session. Subsequent unqualified references (e.g., `dbo.Employees`) resolve against `EmployeesDB`. The `GO` batch separator is required in T-SQL scripts.

#### Example 2: PostgreSQL — Connecting via Connection String

```bash
# Connect using psql with a connection URI
psql "postgresql://app_user:secret@localhost:5432/appdb"

# Or using keyword/value format
psql "host=localhost port=5432 dbname=appdb user=app_user"
```

**Expected Output:**

```
psql (16.0)
Type "help" for help.

appdb=#
```

**Why This Works:** PostgreSQL requires the database to be specified at connection time. The connection URI `postgresql://app_user:secret@localhost:5432/appdb` encodes the user, password, host, port, and database name. The `psql` prompt shows `appdb=#`, confirming the active database.

### Real-World Cases

- **Multi-database administration:** DBAs switching between databases to run maintenance scripts.
- **Application connection pooling:** Connection strings specify the target database for each application pool.
- **Cross-database queries (SQL Server):** Using three-part identifiers (`Database.Schema.Object`) without switching context.
- **Reporting tools:** Connecting to different databases for different reports via connection configuration.

### References

- SQL Server: USE (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/language-elements/use-transact-sql
- MySQL: USE — https://dev.mysql.com/doc/refman/8.0/en/use.html
- PostgreSQL: Managing Database Connections (ECPG) — https://www.postgresql.org/docs/current/ecpg-connect.html
- PostgreSQL: Connection Strings — https://www.postgresql.org/docs/current/libpq-connect.html
- Oracle Database: ALTER SESSION — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/ALTER-SESSION.html


## Core Concept 3: Listing Databases

### Definitions

**Core Definition:** Listing databases is the operation of retrieving the names and metadata of all databases available on a database server instance, using system catalog views or command-line utilities.

**Technical Definition:** Each DBMS provides system catalog views and CLI commands for enumerating databases. SQL Server provides the `sys.databases` catalog view, which contains one row per database, including `name`, `database_id`, `create_date`, `state`, `recovery_model`, and other metadata. MySQL provides the `SHOW DATABASES` command and the `information_schema.SCHEMATA` view. PostgreSQL provides the `pg_database` catalog and the `\l` meta-command in psql. Oracle provides `V$DATABASE` (for the current instance) and `DBA_USERS`/`ALL_USERS` for schema-level enumeration (Oracle does not have multiple databases per instance in the SQL Server sense; it has one database per instance with multiple schemas).

**Beginner-Friendly Explanation:** Listing databases is like looking at a directory of all the filing cabinets in a room. You can see their names and some basic information about each one. Different database systems have different ways to show you this list.

### Purposes

- To discover available databases on a server for connection and administration.
- To audit database inventory for compliance and capacity planning.
- To programmatically check for the existence of a database before creating or dropping it.
- To monitor database states (online, offline, restoring).

### Syntax Rules and Structure

#### Complete General Syntax (SQL Server)

```sql
SELECT name, database_id, create_date, state_desc, recovery_model_desc
FROM sys.databases;

-- Or use the stored procedure
EXEC sp_helpdb;

-- To exclude system databases:
SELECT name FROM sys.databases WHERE database_id > 4;
```

#### Complete General Syntax (MySQL)

```sql
SHOW DATABASES;

-- Or query information_schema
SELECT SCHEMA_NAME, DEFAULT_CHARACTER_SET_NAME, DEFAULT_COLLATION_NAME
FROM information_schema.SCHEMATA;
```

#### Complete General Syntax (PostgreSQL)

```sql
-- SQL query against pg_database
SELECT datname, encoding, datcollate, datistemplate, datallowconn
FROM pg_database;

-- psql meta-command
\l
```

#### Complete General Syntax (Oracle)

```sql
-- Current database instance
SELECT name, db_unique_name, open_mode, created FROM v$database;

-- All schemas (users)
SELECT username, account_status, default_tablespace
FROM dba_users;
```

#### Syntax Rules

- **SQL Server:** `sys.databases` is the recommended catalog view for listing databases. `database_id > 4` excludes the four system databases (`master`, `tempdb`, `model`, `msdb`).
- **MySQL:** `SHOW DATABASES` lists databases the current user has privileges to see. `information_schema.SCHEMATA` provides additional metadata (character set, collation).
- **PostgreSQL:** `pg_database` contains one row per database. `datistemplate` indicates whether the database is a template; `datallowconn` indicates whether connections are allowed.
- **Oracle:** Oracle uses a single database per instance; `V$DATABASE` returns one row for the current instance. Schemas (users) are enumerated via `DBA_USERS`.

#### Constraints and Limitations

- **SQL Server:** `sys.databases` is visible only to users with appropriate permissions; non-privileged users may see only databases they can access.
- **MySQL:** `SHOW DATABASES` returns only databases the user has privileges on; `information_schema` is accessible to all users but shows only accessible databases.
- **PostgreSQL:** `pg_database` is visible to all users but connection permissions are controlled separately.
- **Oracle:** `DBA_USERS` requires `DBA` role or `SELECT_CATALOG_ROLE`; `ALL_USERS` is accessible to all users.

### Annotated Code Examples

#### Example 1: SQL Server — Listing Databases with Metadata

```sql
-- List all user databases with key metadata
SELECT
    name AS database_name,
    database_id,
    create_date,
    state_desc,
    recovery_model_desc,
    compatibility_level
FROM sys.databases
WHERE database_id > 4  -- Exclude system databases
ORDER BY name;
```

**Expected Output:**

```
database_name | database_id | create_date          | state_desc | recovery_model_desc | compatibility_level
--------------+-------------+----------------------+------------+---------------------+---------------------
EmployeesDB   |           5 | 2026-09-26 10:00:00  | ONLINE     | FULL                |                 160
AppDB         |           6 | 2026-09-26 10:05:00  | ONLINE     | SIMPLE              |                 160
WarehouseDB   |           7 | 2026-09-26 10:10:00  | ONLINE     | BULK_LOGGED         |                 150
```

**Why This Works:** `sys.databases` returns one row per database on the instance. Filtering `database_id > 4` excludes the system databases. The `recovery_model_desc` and `compatibility_level` columns provide configuration metadata.

#### Example 2: MySQL — SHOW DATABASES and information_schema

```sql
-- List databases
SHOW DATABASES;

-- Query information_schema for more detail
SELECT
    SCHEMA_NAME AS database_name,
    DEFAULT_CHARACTER_SET_NAME AS charset,
    DEFAULT_COLLATION_NAME AS collation
FROM information_schema.SCHEMATA
WHERE SCHEMA_NAME NOT IN ('information_schema', 'mysql', 'performance_schema', 'sys')
ORDER BY SCHEMA_NAME;
```

**Expected Output:**

```
+--------------------+
| Database           |
+--------------------+
| appdb              |
| employees_db       |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| warehouse_db       |
+--------------------+

database_name | charset | collation
--------------+---------+--------------------
appdb         | utf8mb4 | utf8mb4_unicode_ci
employees_db  | utf8mb4 | utf8mb4_unicode_ci
warehouse_db  | utf8mb4 | utf8mb4_0900_ai_ci
```

**Why This Works:** `SHOW DATABASES` provides a simple list. The `information_schema.SCHEMATA` query provides character set and collation metadata for each database, excluding MySQL's system databases.

### Real-World Cases

- **Inventory auditing:** Generating a report of all databases, their sizes, and recovery models for compliance.
- **Automated provisioning:** Scripts checking `sys.databases` or `pg_database` before creating a new database.
- **Capacity planning:** Monitoring database creation dates and growth patterns.
- **Multi-tenant management:** Listing all tenant databases on a shared server.

### References

- SQL Server: sys.databases — https://learn.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-databases-transact-sql
- MySQL: SHOW DATABASES — https://dev.mysql.com/doc/refman/8.0/en/show-databases.html
- MySQL: information_schema.SCHEMATA — https://dev.mysql.com/doc/refman/8.0/en/information-schema-schemata-table.html
- PostgreSQL: pg_database — https://www.postgresql.org/docs/current/catalog-pg-database.html
- Oracle Database: V$DATABASE — https://docs.oracle.com/en/database/oracle/oracle-database/21/refrn/V-DATABASE.html


## Core Concept 4: Renaming Databases

### Definitions

**Core Definition:** Renaming a database is the operation of changing a database's logical name without affecting its physical files or contents, using `ALTER DATABASE ... MODIFY NAME` (SQL Server) or `ALTER DATABASE ... RENAME TO` (PostgreSQL).

**Technical Definition:** Renaming a database modifies the database name in the system catalog. In SQL Server, the syntax is `ALTER DATABASE [old_name] MODIFY NAME = [new_name]`, but the database must be set to `SINGLE_USER` mode with `ROLLBACK IMMEDIATE` to disconnect active users. In PostgreSQL, the syntax is `ALTER DATABASE old_name RENAME TO new_name`, but active connections must be terminated first (using `pg_terminate_backend` or, in PostgreSQL 13+, `WITH (FORCE)`). MySQL does not support renaming a database directly; the workaround is to create a new database and use `RENAME TABLE` to move each table, or use `mysqldump` and restore.

**Beginner-Friendly Explanation:** Renaming a database is like changing the name on a filing cabinet. The contents inside do not change—only the label on the outside. In SQL Server, you have to kick everyone out of the room first (set to single-user mode). In PostgreSQL, you have to close all the doors (terminate connections) before you can change the label.

### Purposes

- To align database names with updated naming conventions or business terminology.
- To correct naming errors made during initial creation.
- To prepare a database for migration or handover with a more descriptive name.

### Syntax Rules and Structure

#### Complete General Syntax (SQL Server)

```sql
-- Step 1: Set to SINGLE_USER to disconnect active users
ALTER DATABASE [OldDatabaseName] SET SINGLE_USER WITH ROLLBACK IMMEDIATE;

-- Step 2: Rename
ALTER DATABASE [OldDatabaseName] MODIFY NAME = [NewDatabaseName];

-- Step 3: Revert to MULTI_USER
ALTER DATABASE [NewDatabaseName] SET MULTI_USER;
```

#### Complete General Syntax (PostgreSQL)

```sql
-- Step 1: Terminate active connections (PostgreSQL 13+ can use FORCE)
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'old_database_name'
  AND pid <> pg_backend_pid();

-- Step 2: Rename
ALTER DATABASE old_database_name RENAME TO new_database_name;
```

#### Complete General Syntax (MySQL — workaround)

```sql
-- Step 1: Create new database
CREATE DATABASE new_database_name;

-- Step 2: Move all tables
RENAME TABLE old_db.table1 TO new_db.table1,
             old_db.table2 TO new_db.table2,
             ...;

-- Step 3: Drop old database
DROP DATABASE old_database_name;
```

#### Syntax Rules

- **SQL Server:** `ALTER DATABASE ... MODIFY NAME` is the correct syntax (not `RENAME`). The database must not have active connections. `SINGLE_USER WITH ROLLBACK IMMEDIATE` disconnects all users immediately.
- **PostgreSQL:** `ALTER DATABASE ... RENAME TO` is the correct syntax. PostgreSQL does not allow renaming a database if there are active connections. Only the database owner or a superuser can rename a database; non-superuser owners must also have the `CREATEDB` privilege.
- **PostgreSQL:** The current database cannot be renamed; you must connect to a different database to rename it.
- **MySQL:** There is no direct `RENAME DATABASE` support. The recommended approach is to create a new database and move tables, or use `mysqldump` and restore.

#### Constraints and Limitations

- **SQL Server:** Renaming a database does not rename the physical files (`.mdf`, `.ldf`); these retain their original names.
- **PostgreSQL:** `pg_terminate_backend` requires superuser or `pg_signal_backend` membership.
- **MySQL:** The `RENAME TABLE` approach requires dropping and recreating foreign keys and adjusting permissions.
- **Oracle:** `ALTER DATABASE RENAME FILE` renames data files, not the database name; changing the database name requires recreating the control file.

### Annotated Code Examples

#### Example 1: SQL Server — Renaming a Database Safely

```sql
-- Step 1: Set to SINGLE_USER to disconnect everyone
ALTER DATABASE University SET SINGLE_USER WITH ROLLBACK IMMEDIATE;

-- Step 2: Rename the database
ALTER DATABASE University MODIFY NAME = NewUniversity;

-- Step 3: Revert to MULTI_USER
ALTER DATABASE NewUniversity SET MULTI_USER;

-- Verify
SELECT name FROM sys.databases WHERE name = 'NewUniversity';
```

**Expected Output:**

```
name
--------------
NewUniversity
```

**Why This Works:** `SET SINGLE_USER WITH ROLLBACK IMMEDIATE` disconnects all active users and rolls back their transactions immediately. `MODIFY NAME` changes the logical name. `SET MULTI_USER` restores normal access.

#### Example 2: PostgreSQL — Renaming a Database

```sql
-- Step 1: Terminate active connections
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'university'
  AND pid <> pg_backend_pid();

-- Step 2: Rename
ALTER DATABASE university RENAME TO newuniversity;
```

**Expected Output:**

```
 pg_terminate_backend
----------------------
 t
 t

ALTER DATABASE
```

**Why This Works:** `pg_terminate_backend` forcibly disconnects sessions connected to the target database. The `pid <> pg_backend_pid()` condition avoids terminating the current session. `ALTER DATABASE ... RENAME TO` changes the database name in the catalog.

### Real-World Cases

- **Rebranding:** Renaming a database after a company or product rebrand.
- **Naming convention alignment:** Standardizing database names across environments (e.g., `app_dev` → `appdb_dev`).
- **Migration preparation:** Renaming a database to a more descriptive name before a migration.
- **Test environment reset:** Renaming a production copy to a test database name.

### References

- SQL Server: ALTER DATABASE (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-transact-sql
- PostgreSQL: ALTER DATABASE — https://www.postgresql.org/docs/current/sql-alterdatabase.html
- MySQL: RENAME TABLE — https://dev.mysql.com/doc/refman/8.0/en/rename-table.html
- Oracle Database: ALTER DATABASE — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/ALTER-DATABASE.html


## Core Concept 5: Dropping Databases

### Definitions

**Core Definition:** Dropping a database is the operation of permanently removing a database and all its associated objects—tables, views, indexes, procedures, and data—from the database server using the `DROP DATABASE` statement.

**Technical Definition:** `DROP DATABASE` is a DDL statement that deletes the database's catalog entries and removes its physical files (data and log). Once executed, the operation cannot be rolled back (in most DBMS). SQL Server requires the connection to be in the `master` database and the target database to not be in use; if active connections exist, the database must be set to `SINGLE_USER WITH ROLLBACK IMMEDIATE` first. PostgreSQL requires that no other sessions are connected to the database; PostgreSQL 13+ supports `WITH (FORCE)` to disconnect active sessions automatically. MySQL supports `DROP DATABASE IF EXISTS` to suppress errors. Oracle uses `DROP DATABASE` but it is a much more destructive operation (drops the entire instance).

**Beginner-Friendly Explanation:** Dropping a database is like demolishing a filing cabinet and everything inside it. There is no recycle bin—once it is gone, it is gone. Before you drop a database, you must make sure no one is using it, and you should always have a backup in case you need to restore it later.

### Purposes

- To remove obsolete or unused databases to free disk space.
- To clean up test or development environments.
- To decommission a database that is no longer needed.
- To reset a database to a clean state (drop and recreate).

### Syntax Rules and Structure

#### Complete General Syntax (SQL Server)

```sql
-- Connect to master first
USE master;
GO

-- If active connections exist, set to SINGLE_USER first
ALTER DATABASE [DatabaseName] SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
GO

DROP DATABASE [DatabaseName];
GO

-- Or with IF EXISTS (SQL Server 2016+)
DROP DATABASE IF EXISTS [DatabaseName];
GO
```

#### Complete General Syntax (PostgreSQL)

```sql
-- Standard drop (fails if active connections exist)
DROP DATABASE database_name;

-- Force drop (PostgreSQL 13+)
DROP DATABASE database_name WITH (FORCE);

-- If EXISTS
DROP DATABASE IF EXISTS database_name;
```

#### Complete General Syntax (MySQL)

```sql
DROP DATABASE database_name;
DROP DATABASE IF EXISTS database_name;
```

#### Complete General Syntax (Oracle)

```sql
-- Connect as SYSDBA, then:
STARTUP FORCE MOUNT RESTRICTED;
DROP DATABASE;
```

#### Syntax Rules

- **SQL Server:** You must be connected to `master` to drop a database. You cannot drop a database while it is in use. Use `SINGLE_USER WITH ROLLBACK IMMEDIATE` to force disconnection.
- **SQL Server:** `DROP DATABASE IF EXISTS` is supported in SQL Server 2016+; for earlier versions, use `IF DB_ID('name') IS NOT NULL` before dropping.
- **PostgreSQL:** `DROP DATABASE` cannot be executed inside a transaction block. Active connections must be terminated first. PostgreSQL 13+ supports `WITH (FORCE)` to disconnect active sessions.
- **PostgreSQL:** `DROP DATABASE IF EXISTS` is supported and does not raise an error if the database does not exist.
- **MySQL:** `DROP DATABASE IF EXISTS` prevents an error if the database does not exist. MySQL requires the `DROP` privilege on the database.
- **Oracle:** `DROP DATABASE` drops the entire database instance and all data files; it is irreversible and requires `SYSDBA` privilege.

#### Constraints and Limitations

- **Irreversibility:** `DROP DATABASE` permanently deletes all data and cannot be rolled back. Always maintain a backup.
- **Active connections:** SQL Server and PostgreSQL reject `DROP DATABASE` if active connections exist; use `SINGLE_USER` (SQL Server) or `WITH (FORCE)` (PostgreSQL 13+) to disconnect them.
- **SQL Server:** The `master` database cannot be dropped; `model`, `msdb`, and `tempdb` also cannot be dropped.
- **PostgreSQL:** The `template1` database cannot be dropped if it is the last user database; you must connect to `template1` to drop other databases.
- **MySQL:** Dropping a database does not automatically revoke privileges granted on that database.

### Annotated Code Examples

#### Example 1: SQL Server — Dropping a Database with Active Connections

```sql
-- Connect to master
USE master;
GO

-- Force disconnect and set to SINGLE_USER
ALTER DATABASE [CompanyData] SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
GO

-- Drop the database
DROP DATABASE [CompanyData];
GO

-- Verify it is gone
SELECT name FROM sys.databases WHERE name = 'CompanyData';
```

**Expected Output:**

```
Commands completed successfully.
Commands completed successfully.

name
----
(0 rows)
```

**Why This Works:** `SINGLE_USER WITH ROLLBACK IMMEDIATE` disconnects all active connections. `DROP DATABASE` removes the database and its files. The verification query returns no rows, confirming the database is dropped.

#### Example 2: PostgreSQL — Dropping a Database with FORCE

```sql
-- Drop a database, forcing disconnect of active sessions
DROP DATABASE IF EXISTS my_app_db WITH (FORCE);
```

**Expected Output:**

```
DROP DATABASE
```

**Why This Works:** `IF EXISTS` prevents an error if the database does not exist. `WITH (FORCE)` (PostgreSQL 13+) terminates all active connections to the database before dropping it.

### Real-World Cases

- **Environment cleanup:** Dropping temporary test databases after automated testing.
- **Project decommissioning:** Removing databases for applications that have been retired.
- **Disk space reclamation:** Dropping unused databases to free storage.
- **Development reset:** Dropping and recreating a development database to start fresh.

### References

- SQL Server: DROP DATABASE (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/statements/drop-database-transact-sql
- PostgreSQL: DROP DATABASE — https://www.postgresql.org/docs/current/sql-dropdatabase.html
- MySQL: DROP DATABASE — https://dev.mysql.com/doc/refman/8.0/en/drop-database.html
- Oracle Database: DROP DATABASE — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/DROP-DATABASE.html


## Core Concept 6: Database Ownership

### Definitions

**Core Definition:** Database ownership is the assignment of a database to a specific user or role that has full control over the database, including the ability to grant permissions, alter configuration, and drop the database.

**Technical Definition:** Database ownership determines the principal that has ultimate authority over the database. In SQL Server, ownership is managed via `ALTER AUTHORIZATION ON DATABASE::database_name TO principal;`. In PostgreSQL, ownership is set at creation time (`CREATE DATABASE ... OWNER = role`) or changed with `ALTER DATABASE name OWNER TO new_owner;`. Only the database owner or a superuser can rename a database; non-superuser owners must also have the `CREATEDB` privilege. To alter the owner in PostgreSQL, you must own the database, be a direct or indirect member of the new owning role, and have the `CREATEDB` privilege. Oracle does not have a database ownership concept in the same sense; schema (user) ownership is the relevant model.

**Beginner-Friendly Explanation:** Database ownership is like having the deed to a house. The owner can decide who gets keys (permissions), what renovations happen (configuration changes), and even whether to demolish the house (drop the database). Transferring ownership is like selling the house—the new owner takes full control.

### Purposes

- To ensure a specific principal (user or role) has ultimate authority over the database.
- To transfer database administration responsibilities when a DBA changes.
- To enable application teams to manage their own databases without superuser intervention.
- To comply with security policies requiring clear ownership and accountability.

### Syntax Rules and Structure

#### Complete General Syntax (SQL Server)

```sql
ALTER AUTHORIZATION ON DATABASE::database_name TO principal;
```

#### Complete General Syntax (PostgreSQL)

```sql
ALTER DATABASE name OWNER TO { new_owner | CURRENT_ROLE | CURRENT_USER | SESSION_USER };
```

#### Complete General Syntax (MySQL)

```sql
-- MySQL does not have a database owner concept;
-- ownership is effectively determined by privileges.
-- Use GRANT ALL ON database_name.* TO 'user'@'host';
```

#### Complete General Syntax (Oracle)

```sql
-- Oracle uses schema-level ownership (users own objects)
ALTER USER schema_name IDENTIFIED BY password;
-- To change a user's default tablespace:
ALTER USER schema_name DEFAULT TABLESPACE new_tablespace;
```

#### Syntax Rules

- **SQL Server:** `ALTER AUTHORIZATION` can be used on many object types (databases, schemas, tables). For databases, use `ON DATABASE::database_name`.
- **PostgreSQL:** Only the database owner or a superuser can rename a database. To alter the owner, you must own the database and also be a direct or indirect member of the new owning role, and you must have the `CREATEDB` privilege.
- **PostgreSQL:** `ALTER DATABASE name OWNER TO` changes the owner; the previous owner loses all privileges on the database.
- **MySQL:** There is no formal database owner; the concept is replaced by privilege grants.
- **Oracle:** Schema (user) ownership is the model; the user who creates an object owns it.

#### Constraints and Limitations

- **SQL Server:** Changing database ownership requires `TAKE OWNERSHIP` or `ALTER ANY DATABASE` permission.
- **PostgreSQL:** You cannot change the owner of the current database if you are connected to it in some cases; connect to a different database first.
- **PostgreSQL:** The new owner must be able to `SET ROLE` to the new owning role and have the `CREATEDB` privilege.
- **MySQL:** No formal ownership; privilege-based control is the only mechanism.
- **Oracle:** A user cannot be dropped if they own objects; the objects must be dropped first or the `CASCADE` clause used.

### Annotated Code Examples

#### Example 1: SQL Server — Changing Database Ownership

```sql
-- Transfer ownership to a specific login
ALTER AUTHORIZATION ON DATABASE::EmployeesDB TO [app_admin];
```

**Expected Output:**

```
Commands completed successfully.
```

**Why This Works:** `ALTER AUTHORIZATION ON DATABASE::EmployeesDB TO [app_admin]` transfers ownership of the `EmployeesDB` database to the `app_admin` principal. The new owner has full control over the database.

#### Example 2: PostgreSQL — Changing Database Owner

```sql
-- Change the owner of the appdb database
ALTER DATABASE appdb OWNER TO app_user;

-- Verify
SELECT datname, pg_get_userbyid(datdba) AS owner
FROM pg_database
WHERE datname = 'appdb';
```

**Expected Output:**

```
 datname | owner
---------+----------
 appdb   | app_user
```

**Why This Works:** `ALTER DATABASE appdb OWNER TO app_user` changes the database owner. The `pg_get_userbyid(datdba)` function retrieves the owner's name from the `pg_database` catalog.

### Real-World Cases

- **DBA handover:** Transferring database ownership when a DBA leaves the organization.
- **Application team self-service:** Granting application teams ownership of their databases.
- **Security compliance:** Ensuring each database has a clearly identified owner for audit purposes.
- **Multi-tenant SaaS:** Each tenant database is owned by a tenant-specific role.

### References

- SQL Server: ALTER AUTHORIZATION — https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-authorization-transact-sql
- PostgreSQL: ALTER DATABASE (OWNER TO) — https://www.postgresql.org/docs/current/sql-alterdatabase.html
- MySQL: GRANT — https://dev.mysql.com/doc/refman/8.0/en/grant.html
- Oracle Database: ALTER USER — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/ALTER-USER.html


## Core Concept 7: Database Configuration

### Definitions

**Core Definition:** Database configuration is the set of database-scoped settings—recovery model, collation, compatibility level, and encryption—that control data durability, text comparison behavior, query optimizer behavior, and data-at-rest security.

**Technical Definition:** Database configuration encompasses several categories of settings. The **recovery model** (SQL Server) determines how transactions are logged and what restore operations are possible: `SIMPLE` (log is auto-truncated, no point-in-time recovery), `FULL` (all operations fully logged, point-in-time recovery possible), and `BULK_LOGGED` (minimal logging for bulk operations, log retained until backup). **Collation** determines sort order, case sensitivity, and accent sensitivity for character data. **Compatibility level** (SQL Server) controls the query optimizer behavior and T-SQL feature availability, allowing a database to behave like an older SQL Server version. **Transparent Data Encryption (TDE)** encrypts the database files at rest using a database encryption key (DEK) protected by a server certificate or asymmetric key.

**Beginner-Friendly Explanation:** Database configuration is like the settings on a security system and a filing cabinet. The recovery model decides how much protection you have if something goes wrong (can you restore to a specific moment?). Collation decides how text is sorted and compared (is "a" the same as "A"?). Compatibility level decides which version of the rules the database follows. TDE encrypts the files so no one can read them without the key.

### Purposes

- To select the appropriate recovery model based on RPO (Recovery Point Objective) and RTO (Recovery Time Objective).
- To enforce consistent text sorting and comparison through collation.
- To maintain backward compatibility with applications written for older SQL Server versions.
- To protect data at rest through encryption.

### Syntax Rules and Structure

#### Complete General Syntax (SQL Server — Recovery Model)

```sql
ALTER DATABASE database_name SET RECOVERY { SIMPLE | FULL | BULK_LOGGED };
```

#### Complete General Syntax (SQL Server — Compatibility Level)

```sql
ALTER DATABASE database_name SET COMPATIBILITY_LEVEL = { 170 | 160 | 150 | 140 | 130 | 120 | 110 | 100 | 90 | 80 };
```

#### Complete General Syntax (SQL Server — TDE)

```sql
-- Step 1: Create a master key in master
USE master;
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'StrongPassword123!';

-- Step 2: Create a certificate
CREATE CERTIFICATE TDECert WITH SUBJECT = 'TDE Certificate';

-- Step 3: Create a database encryption key
USE MyDatabase;
CREATE DATABASE ENCRYPTION KEY
WITH ALGORITHM = AES_256
ENCRYPTION BY SERVER CERTIFICATE TDECert;

-- Step 4: Enable encryption
ALTER DATABASE MyDatabase SET ENCRYPTION ON;
```

#### Complete General Syntax (PostgreSQL — Collation)

```sql
-- Set collation at database creation
CREATE DATABASE mydb
    LC_COLLATE = 'en_US.UTF-8'
    LC_CTYPE = 'en_US.UTF-8'
    TEMPLATE = template0;
```

#### Syntax Rules

- **SQL Server Recovery Models:** `SIMPLE` recovery model writes every DML/DDL to the transaction log for basic recovery (rollback of unfinished operations), but log records are cleaned after commit and checkpoint. `BULK_LOGGED` acts like `SIMPLE` for log cleaning but retains log records until `BACKUP LOG` is executed. `FULL` fully logs all operations, including minimally logged operations, supporting point-in-time recovery.
- **SQL Server Compatibility Level:** The default for SQL Server 2022 (16.x) is 160. Compatibility level 170 is available for Azure SQL Database and SQL Server 2025. The level determines which T-SQL features and query optimizer behaviors are available.
- **SQL Server TDE:** The database encryption key (DEK) must be created before the database can be encrypted. The DEK is encrypted by a server certificate or asymmetric key. Algorithms: `AES_128`, `AES_192`, `AES_256`, `TRIPLE_DES_3KEY` (deprecated).
- **PostgreSQL Collation:** `LC_COLLATE` sets the default collation order for the database; `LC_CTYPE` sets the character classification. Both must be compatible with the database encoding.
- **PostgreSQL:** There is currently no option to use a database locale with nondeterministic comparisons.

#### Constraints and Limitations

- **SQL Server:** After changing the recovery model to `FULL` or `BULK_LOGGED`, a full or differential backup is required to start the log sequence; otherwise the database continues in `SIMPLE` mode.
- **SQL Server:** Changing compatibility level may invalidate the plan cache, requiring recompilation of queries.
- **SQL Server:** TDE is not available in SQL Server Express or Standard editions (Enterprise only, except in Azure SQL).
- **PostgreSQL:** Collation cannot be changed after database creation without recreating the database.
- **Oracle:** TDE requires an Oracle Wallet or Hardware Security Module (HSM) to store the master encryption key.

### Annotated Code Examples

#### Example 1: SQL Server — Setting Recovery Model and Compatibility Level

```sql
-- Set the recovery model to FULL
ALTER DATABASE EmployeesDB SET RECOVERY FULL;

-- Set compatibility level to SQL Server 2022 (160)
ALTER DATABASE EmployeesDB SET COMPATIBILITY_LEVEL = 160;

-- Verify
SELECT name, recovery_model_desc, compatibility_level
FROM sys.databases
WHERE name = 'EmployeesDB';
```

**Expected Output:**

```
name         | recovery_model_desc | compatibility_level
-------------+---------------------+---------------------
EmployeesDB  | FULL                |                 160
```

**Why This Works:** `SET RECOVERY FULL` enables full transaction logging, supporting point-in-time recovery. `SET COMPATIBILITY_LEVEL = 160` configures the database to use SQL Server 2022 query optimizer behavior. The verification query confirms both settings.

#### Example 2: SQL Server — Enabling TDE

```sql
-- Step 1: Create master key (if not exists)
USE master;
IF NOT EXISTS (SELECT * FROM sys.symmetric_keys WHERE name = '##MS_DatabaseMasterKey##')
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'Str0ngP@ssw0rd!';

-- Step 2: Create server certificate
CREATE CERTIFICATE TDECert
WITH SUBJECT = 'TDE Certificate for EmployeesDB';

-- Step 3: Create database encryption key
USE EmployeesDB;
CREATE DATABASE ENCRYPTION KEY
WITH ALGORITHM = AES_256
ENCRYPTION BY SERVER CERTIFICATE TDECert;

-- Step 4: Enable TDE
ALTER DATABASE EmployeesDB SET ENCRYPTION ON;

-- Verify encryption state
SELECT DB_NAME(database_id) AS database_name, encryption_state_desc
FROM sys.dm_database_encryption_keys
WHERE DB_NAME(database_id) = 'EmployeesDB';
```

**Expected Output:**

```
database_name | encryption_state_desc
--------------+-----------------------
EmployeesDB   | ENCRYPTED
```

**Why This Works:** The master key protects the certificate, the certificate protects the DEK, and the DEK encrypts the database. `SET ENCRYPTION ON` initiates encryption of the data files. The `sys.dm_database_encryption_keys` view confirms the encryption state.

### Real-World Cases

- **Financial systems:** Using `FULL` recovery model for point-in-time recovery to meet regulatory requirements.
- **Bulk ETL:** Using `BULK_LOGGED` recovery model during large data loads to minimize log growth.
- **International applications:** Setting collation to `Latin1_General_CI_AI` for case-insensitive, accent-insensitive sorting.
- **Security compliance:** Enabling TDE to protect data at rest on disk.

### References

- SQL Server: Recovery Models — https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/recovery-models-sql-server
- SQL Server: ALTER DATABASE Compatibility Level — https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-transact-sql-compatibility-level
- SQL Server: CREATE DATABASE ENCRYPTION KEY — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-database-encryption-key-transact-sql
- SQL Server: Transparent Data Encryption (TDE) — https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/transparent-data-encryption
- PostgreSQL: CREATE DATABASE (Collation) — https://www.postgresql.org/docs/current/sql-createdatabase.html
- Oracle Database: Transparent Data Encryption — https://docs.oracle.com/en/database/oracle/oracle-database/21/asoag/introduction-to-transparent-data-encryption.html


## Core Concept 8: Database-Level Permissions

### Definitions

**Core Definition:** Database-level permissions are privileges granted at the database scope—such as `CONNECT`, `CREATE TABLE`, `BACKUP DATABASE`, and `CREATE VIEW`—that control what operations a principal (user or role) can perform on the database as a whole.

**Technical Definition:** Database-level permissions are managed through `GRANT`, `REVOKE`, and `DENY` statements. In SQL Server, database permissions are granted to database principals (users, roles, application roles) and include `CONNECT`, `CREATE TABLE`, `CREATE VIEW`, `CREATE PROCEDURE`, `BACKUP DATABASE`, `BACKUP LOG`, `ALTER`, `CONTROL`, and others. Granting `ALL` is equivalent to granting `BACKUP DATABASE`, `BACKUP LOG`, `CREATE DEFAULT`, `CREATE FUNCTION`, `CREATE PROCEDURE`, `CREATE RULE`, `CREATE TABLE`, and `CREATE VIEW`. In PostgreSQL, database-level privileges are `CREATE`, `CONNECT`, and `TEMPORARY`/`TEMP`. `CONNECT` allows the user to connect to the specified database; this privilege is checked at connection startup in addition to `pg_hba.conf` restrictions. `DENY` explicitly prohibits a permission, overriding any grants.

**Beginner-Friendly Explanation:** Database-level permissions are like the keys to different rooms in a building. `CONNECT` lets you enter the building. `CREATE TABLE` lets you build new rooms. `BACKUP DATABASE` lets you make a copy of the entire building's contents. You can give someone a key (GRANT), take it away (REVOKE), or explicitly forbid them from entering (DENY).

### Purposes

- To control who can connect to a database and what they can do within it.
- To delegate administrative tasks (backup, table creation) without granting full control.
- To comply with the principle of least privilege.
- To explicitly deny access to sensitive operations while granting general access.

### Syntax Rules and Structure

#### Complete General Syntax (SQL Server)

```sql
GRANT <permission> [ ,...n ] TO <database_principal> [ ,...n ]
    [ WITH GRANT OPTION ]
    [ AS <database_principal> ];

REVOKE [ GRANT OPTION FOR ] <permission> [ ,...n ]
    FROM <database_principal> [ ,...n ];

DENY <permission> [ ,...n ] TO <database_principal> [ ,...n ];
```

**Common database permissions:**

| Permission | Description |
|-----------|-------------|
| `CONNECT` | Connect to the database. |
| `CREATE TABLE` | Create tables in the database. |
| `CREATE VIEW` | Create views in the database. |
| `CREATE PROCEDURE` | Create stored procedures. |
| `BACKUP DATABASE` | Back up the database. |
| `BACKUP LOG` | Back up the transaction log. |
| `ALTER` | Alter the database. |
| `CONTROL` | Full control over the database. |
| `EXECUTE` | Execute stored procedures and functions. |

#### Complete General Syntax (PostgreSQL)

```sql
GRANT { { CREATE | CONNECT | TEMPORARY | TEMP } [, ...] | ALL [ PRIVILEGES ] }
    ON DATABASE database_name [, ...]
    TO role_specification [, ...]
    [ WITH GRANT OPTION ];

REVOKE { { CREATE | CONNECT | TEMPORARY | TEMP } [, ...] | ALL [ PRIVILEGES ] }
    ON DATABASE database_name [, ...]
    FROM role_specification [, ...];
```

**PostgreSQL database-level privileges:**

| Privilege | Description |
|-----------|-------------|
| `CREATE` | Create new schemas in the database. |
| `CONNECT` | Connect to the database. |
| `TEMPORARY` / `TEMP` | Create temporary tables. |

#### Complete General Syntax (MySQL)

```sql
GRANT ALL PRIVILEGES ON database_name.* TO 'user'@'host';
GRANT SELECT, INSERT, UPDATE ON database_name.* TO 'user'@'host';
REVOKE INSERT ON database_name.* FROM 'user'@'host';
```

#### Syntax Rules

- **SQL Server:** `GRANT ALL` is equivalent to granting `BACKUP DATABASE`, `BACKUP LOG`, `CREATE DEFAULT`, `CREATE FUNCTION`, `CREATE PROCEDURE`, `CREATE RULE`, `CREATE TABLE`, and `CREATE VIEW`.
- **SQL Server:** `WITH GRANT OPTION` allows the grantee to grant the same permission to other principals.
- **PostgreSQL:** `CONNECT` allows the user to connect to the specified database; this privilege is checked at connection startup in addition to `pg_hba.conf` restrictions.
- **PostgreSQL:** `CREATE` allows the user to create new schemas in the database.
- **MySQL:** Database-level privileges are granted on `database_name.*` and include `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `DROP`, `INDEX`, `ALTER`, `CREATE VIEW`, `SHOW VIEW`, `CREATE ROUTINE`, `ALTER ROUTINE`, `EXECUTE`, `CREATE TEMPORARY TABLES`, `LOCK TABLES`, and others.

#### Constraints and Limitations

- **SQL Server:** `DENY` overrides `GRANT`; a denied permission cannot be granted by another principal.
- **SQL Server:** Users with both `ALTER` and `REFERENCES` permissions can create and execute computed columns, potentially exposing data not otherwise permitted.
- **PostgreSQL:** Superusers bypass all permission checks.
- **MySQL:** `GRANT ALL` on a database does not include the `GRANT OPTION` privilege.
- **Oracle:** Database-level privileges are granted via system privileges (`CREATE TABLE`, `CREATE SESSION`, etc.) and object privileges; `GRANT` is the same statement but the permission set differs.

### Annotated Code Examples

#### Example 1: SQL Server — Granting and Revoking Database Permissions

```sql
-- Grant CONNECT and CREATE TABLE to a user
GRANT CONNECT, CREATE TABLE TO app_user;

-- Grant BACKUP DATABASE and BACKUP LOG to a backup operator role
GRANT BACKUP DATABASE, BACKUP LOG TO backup_operator;

-- Revoke CREATE TABLE from app_user
REVOKE CREATE TABLE FROM app_user;

-- Explicitly deny BACKUP DATABASE to a specific user
DENY BACKUP DATABASE TO intern_user;
```

**Expected Output:**

```
Commands completed successfully.
```

**Why This Works:** `GRANT CONNECT, CREATE TABLE TO app_user` gives the user the ability to connect and create tables. `REVOKE` removes a permission. `DENY` explicitly prohibits the permission, overriding any grants.

#### Example 2: PostgreSQL — Granting Database-Level Privileges

```sql
-- Grant CONNECT to a role
GRANT CONNECT ON DATABASE appdb TO app_user;

-- Grant CREATE (schema creation) and TEMPORARY (temp tables)
GRANT CREATE, TEMPORARY ON DATABASE appdb TO developer_role;

-- Revoke CONNECT from a user
REVOKE CONNECT ON DATABASE appdb FROM temp_user;

-- Verify
SELECT datname, datacl FROM pg_database WHERE datname = 'appdb';
```

**Expected Output:**

```
 datname | datacl
---------+------------------------------------------------------------
 appdb   | {=Tc/postgres,postgres=CTc/postgres,app_user=c/postgres,developer_role=CT/postgres}
```

**Why This Works:** `GRANT CONNECT` allows the user to connect to the database. `GRANT CREATE, TEMPORARY` allows the developer role to create schemas and temporary tables. The `datacl` column in `pg_database` shows the access control list.

### Real-World Cases

- **Application accounts:** Granting `CONNECT` and `EXECUTE` to application service accounts, but not `CREATE TABLE` or `BACKUP DATABASE`.
- **DBA delegation:** Granting `BACKUP DATABASE` and `BACKUP LOG` to a backup operator without granting full `CONTROL`.
- **Developer access:** Granting `CREATE` (schema creation) and `TEMPORARY` to developer roles in development databases.
- **Security hardening:** Using `DENY` to explicitly prohibit sensitive operations (e.g., `BACKUP DATABASE`) for non-DBA users.

### References

- SQL Server: GRANT Database Permissions — https://learn.microsoft.com/en-us/sql/t-sql/statements/grant-database-permissions-transact-sql
- SQL Server: REVOKE Database Permissions — https://learn.microsoft.com/en-us/sql/t-sql/statements/revoke-database-permissions-transact-sql
- SQL Server: DENY Database Permissions — https://learn.microsoft.com/en-us/sql/t-sql/statements/deny-database-permissions-transact-sql
- PostgreSQL: GRANT — https://www.postgresql.org/docs/current/sql-grant.html
- MySQL: GRANT — https://dev.mysql.com/doc/refman/8.0/en/grant.html
- Oracle Database: GRANT — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/GRANT.html


## Summary Table: Database Management Operations Across DBMS

| Operation | SQL Server | PostgreSQL | MySQL | Oracle |
|-----------|-----------|------------|-------|--------|
| **CREATE DATABASE** | `CREATE DATABASE name ON ... LOG ON ...` | `CREATE DATABASE name WITH OWNER, TEMPLATE, ENCODING` | `CREATE DATABASE name CHARACTER SET, COLLATE` | `CREATE DATABASE name USER SYS ... LOGFILE ... DATAFILE ...` |
| **Select Database** | `USE name` | Connection-level only (`\c`, connection string) | `USE name` | `ALTER SESSION SET CURRENT_SCHEMA` |
| **List Databases** | `sys.databases` | `pg_database`, `\l` | `SHOW DATABASES`, `information_schema.SCHEMATA` | `V$DATABASE`, `DBA_USERS` |
| **Rename Database** | `ALTER DATABASE ... MODIFY NAME` | `ALTER DATABASE ... RENAME TO` | Create new + `RENAME TABLE` | Recreate control file |
| **Drop Database** | `DROP DATABASE [IF EXISTS]` | `DROP DATABASE [IF EXISTS] [WITH (FORCE)]` | `DROP DATABASE [IF EXISTS]` | `DROP DATABASE` (SYSDBA) |
| **Ownership** | `ALTER AUTHORIZATION ON DATABASE::...` | `ALTER DATABASE ... OWNER TO` | Privilege-based (no owner) | Schema ownership (users) |
| **Recovery Model** | `ALTER DATABASE ... SET RECOVERY` | N/A (WAL archiving) | N/A (binary logging) | `ARCHIVELOG`/`NOARCHIVELOG` |
| **TDE** | `CREATE DATABASE ENCRYPTION KEY`, `ALTER DATABASE SET ENCRYPTION ON` | `pgcrypto` extension | InnoDB tablespace encryption | `ADMINISTER KEY MANAGEMENT` |
| **Permissions** | `GRANT CONNECT, CREATE TABLE, BACKUP DATABASE` | `GRANT CONNECT, CREATE, TEMPORARY` | `GRANT ALL ON db.* TO user` | `GRANT CREATE SESSION, CREATE TABLE` |


## Final Notes on Deprecated and Unsafe Features

- **SQL Server `sp_renamedb`:** Deprecated; use `ALTER DATABASE ... MODIFY NAME` instead.
- **SQL Server `sp_dboption`:** Deprecated; use `ALTER DATABASE ... SET` instead.
- **SQL Server `TRIPLE_DES_3KEY`:** Deprecated encryption algorithm for TDE; use `AES_256` instead. Starting with SQL Server 2016, all algorithms except AES_128, AES_192, and AES_256 are deprecated.
- **PostgreSQL `template1` modifications:** Objects added to `template1` are propagated to all subsequently created databases; avoid adding site-local objects unless intentional.
- **MySQL `RENAME DATABASE`:** Not supported; the workaround of creating a new database and moving tables is error-prone and requires careful handling of foreign keys and privileges.
- **Oracle `DROP DATABASE`:** Drops the entire database instance and is irreversible; requires `SYSDBA` privilege and should only be used with extreme caution.
- **SQL Server `SINGLE_USER WITH ROLLBACK IMMEDIATE`:** Immediately disconnects all users and rolls back their transactions; use with caution in production environments.
- **Version-specific:** SQL Server `DROP DATABASE IF EXISTS` (2016+), PostgreSQL `WITH (FORCE)` (13+), SQL Server compatibility level 170 (SQL Server 2025 / Azure SQL), SQL Server TDE (Enterprise edition).