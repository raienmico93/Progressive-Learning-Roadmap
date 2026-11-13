# Comprehensive Programming Cheat Sheet: SQL Database Management

## Topic Overview

### Definitions

**Core Definition**  
SQL Database Management is the set of Data Definition Language (DDL) and administrative statements used to create, select, list, rename, drop, configure, and secure databases on a relational database management system (RDBMS) server.

**Technical Definition**  
Database management in SQL encompasses the statements that operate on the database as a logical container: `CREATE DATABASE`, `DROP DATABASE`, `ALTER DATABASE`, `USE` / connection selection, catalog queries (e.g., `SHOW DATABASES`, `pg_database`, `sys.databases`), ownership changes, configuration-parameter assignment, and database‑level privilege grants and revocations. These statements are not part of the core SQL standard; their syntax is vendor‑defined and varies across PostgreSQL, MySQL, SQL Server, and Oracle.

**Beginner-Friendly Explanation**  
Think of a database server as a large filing cabinet. Database management is how you create a new drawer, open an existing drawer, list all the drawers, relabel a drawer, remove a drawer permanently, decide who owns the drawer, adjust the drawer’s settings, and decide who is allowed to open it.

### Key Characteristics

- **Vendor‑specific syntax**: There is no ANSI `CREATE DATABASE` statement; each RDBMS implements its own variation.
- **Privilege‑gated operations**: Creating and dropping databases typically require superuser or elevated privileges.
- **Physical and logical impact**: In some systems (e.g., MySQL) a database corresponds to a directory; in PostgreSQL a database is a catalog with its own system catalogs and transaction isolation.
- **Persistence and irreversibility**: `DROP DATABASE` is permanent and usually irreversible without a backup.
- **Connection‑sensitive**: Many rename and drop operations require that no active connections exist to the target database.

### Prerequisites

- A running RDBMS instance (PostgreSQL, MySQL, SQL Server, etc.).
- An account with the required administrative privileges (e.g., `CREATEDB` in PostgreSQL, `CREATE` in MySQL, `CREATE DATABASE` in SQL Server).
- Basic familiarity with SQL statements and the command‑line or graphical client for the chosen RDBMS.
- For rename and drop operations: the ability to terminate or wait for active connections.

### Related Programming Areas

- Data Definition Language (DDL) – `CREATE`, `ALTER`, `DROP`.
- Data Control Language (DCL) – `GRANT`, `REVOKE`.
- Database administration and DevOps – backup, restore, migration, configuration management.
- Security engineering – least‑privilege access, ownership models.
- Application development – connection strings, database selection, schema migration scripts.

---

## Core Concepts and Features

### 1. Creating Databases (`CREATE DATABASE`)

#### Definitions

**Core Definition**  
`CREATE DATABASE` creates a new, empty database container on the server.

**Technical Definition**  
`CREATE DATABASE` allocates a new catalog (PostgreSQL), directory (MySQL), or database with associated files and filegroups (SQL Server). It may accept options for owner, encoding, collation, template, connection limits, and storage parameters. The statement is not part of the ANSI SQL standard; each vendor defines its own syntax.

**Beginner-Friendly Explanation**  
This is the command that creates a brand‑new, empty database so you can start adding tables and data.

#### Purposes

- To provision a new logical container for an application’s tables, views, and other objects.
- To isolate data belonging to different projects or tenants.
- To create a database with specific character‑set, collation, or template settings.
- To script reproducible database creation for development, testing, or research environments.

#### Syntax Rules and Structure

**General syntax (vendor‑neutral outline)**  

```text
CREATE DATABASE database_name
  [ WITH option [ ... ] ];
```

**PostgreSQL** (official):

```sql
CREATE DATABASE name
    [ [ WITH ]
        [ OWNER [=] user_name ]
        [ TEMPLATE [=] template ]
        [ ENCODING [=] encoding ]
        [ LC_COLLATE [=] lc_collate ]
        [ LC_CTYPE [=] lc_ctype ]
        [ TABLESPACE [=] tablespace_name ]
        [ ALLOW_CONNECTIONS [=] allowconn ]
        [ CONNECTION LIMIT [=] connlimit ]
        [ IS_TEMPLATE [=] istemplate ]
    ];
```

**MySQL** (official):

```sql
CREATE { DATABASE | SCHEMA } [IF NOT EXISTS] db_name
    [ [DEFAULT] CHARACTER SET [=] charset_name ]
    [ [DEFAULT] COLLATE [=] collation_name ];
```

**SQL Server** (official):

```sql
CREATE DATABASE database_name
    [ CONTAINMENT = { NONE | PARTIAL } ]
    [ ON [ PRIMARY ] <filespec> [ ,...n ] ]
    [ LOG ON <filespec> [ ,...n ] ]
    [ COLLATE collation_name ]
    [ WITH <option> [ ,...n ] ];
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `database_name` / `db_name` / `name` | Identifier for the new database. Must follow the RDBMS’s identifier rules. |
| `IF NOT EXISTS` (MySQL) | Suppresses the error if a database with the same name already exists. |
| `OWNER` (PostgreSQL) | Specifies the role that will own the new database. |
| `TEMPLATE` (PostgreSQL) | Database to use as a template; `template1` is the default. |
| `ENCODING`, `LC_COLLATE`, `LC_CTYPE` | Character‑set and locale settings. |
| `CHARACTER SET`, `COLLATE` (MySQL) | Default character set and collation for the new database. |
| `ON`, `LOG ON`, `<filespec>` (SQL Server) | Defines data and log file placement, size, and growth. |
| `CONTAINMENT` (SQL Server) | `NONE` or `PARTIAL` for contained databases. |

**Syntax rules**

- The database name must be unique on the server (or within the instance).
- In PostgreSQL, the executing role must be a superuser or have the `CREATEDB` privilege.
- In MySQL, the `CREATE` privilege is required.
- In SQL Server, `CREATE DATABASE`, `CREATE ANY DATABASE`, or `ALTER ANY DATABASE` permission is required.
- `CREATE DATABASE` cannot be executed inside a transaction block in PostgreSQL.

**Constraints and limitations**

- You cannot create a database with the same name as an existing database.
- In PostgreSQL, `CREATE DATABASE` cannot be run inside a transaction block.
- MySQL treats `CREATE SCHEMA` as a synonym for `CREATE DATABASE`; the two are interchangeable.
- SQL Server may create multiple physical files; the `<filespec>` options control file growth and maximum size.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Create a database with an explicit owner and encoding

```sql
-- Step 1: Connect as a superuser or a role with CREATEDB privilege.
-- (Connection is assumed to be already established.)

-- Step 2: Create a new database named "analytics_db"
--         owned by the role "analyst", using UTF-8 encoding.
CREATE DATABASE analytics_db
    WITH OWNER = analyst
         ENCODING = 'UTF8'
         LC_COLLATE = 'en_US.UTF-8'
         LC_CTYPE = 'en_US.UTF-8'
         CONNECTION LIMIT = 50;
```

**Expected output** (psql client):

```text
CREATE DATABASE
```

**Why this output appears**: PostgreSQL returns the command tag `CREATE DATABASE` when the statement succeeds. The database is created by cloning `template1` and applying the specified owner, encoding, locale, and connection limit.

##### Example 2: MySQL – Create a database with a specific character set and collation

```sql
-- Step 1: Connect to the MySQL server as a user with CREATE privilege.

-- Step 2: Create the database only if it does not already exist,
--         using utf8mb4 and a case-insensitive collation.
CREATE DATABASE IF NOT EXISTS school_db
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

-- Step 3: Verify the database was created.
SHOW DATABASES LIKE 'school_db';
```

**Expected output**:

```text
Query OK, 1 row affected (0.01 sec)

+------------------------+
| Database (school_db)   |
+------------------------+
| school_db              |
+------------------------+
1 row in set (0.00 sec)
```

**Why this output appears**: The `CREATE DATABASE` statement creates a new directory on the MySQL server file system. The `IF NOT EXISTS` clause prevents an error if the database already exists. `SHOW DATABASES LIKE` confirms the database is present.

##### Example 3: SQL Server – Create a database with explicit data and log files

```sql
-- Step 1: Connect to the SQL Server instance with CREATE DATABASE permission.

-- Step 2: Create a database with a primary data file and a log file.
CREATE DATABASE SalesDB
ON PRIMARY (
    NAME = SalesDB_Data,
    FILENAME = 'C:\SQLData\SalesDB_Data.mdf',
    SIZE = 100MB,
    MAXSIZE = 1GB,
    FILEGROWTH = 50MB
)
LOG ON (
    NAME = SalesDB_Log,
    FILENAME = 'C:\SQLData\SalesDB_Log.ldf',
    SIZE = 50MB,
    MAXSIZE = 500MB,
    FILEGROWTH = 25MB
);
GO
```

**Expected output** (SSMS or `sqlcmd`):

```text
Commands completed successfully.
```

**Why this output appears**: SQL Server creates the database along with the specified physical files. The `ON PRIMARY` clause defines the primary data file, and `LOG ON` defines the transaction log file. The `GO` batch separator is used in SQL Server client tools.

#### Real-World Cases

- **Research data management**: A research group creates a database with a specific character set and collation so that survey responses in multiple languages are stored correctly, and the creation script can be saved in a data management plan.
- **Multi‑tenant SaaS**: An application provisions a separate database per tenant, using `CREATE DATABASE` with a templated schema to isolate customer data.
- **Development and testing**: CI/CD pipelines create ephemeral databases with `IF NOT EXISTS` or drop‑and‑recreate patterns to ensure a clean state for each test run.

---

### 2. Selecting Databases

#### Definitions

**Core Definition**  
Selecting a database sets the current database context so that subsequent unqualified object names resolve within that database.

**Technical Definition**  
In MySQL, `USE database_name` changes the default database for the session. In PostgreSQL, selection is achieved by connecting to the target database (e.g., `\c database_name` in `psql` or specifying the database in the connection string). SQL Server uses `USE database_name`. There is no standard SQL statement for “selecting” a database; it is a client or vendor‑specific operation.

**Beginner-Friendly Explanation**  
This is how you “open” a database so that the tables you refer to by name are looked up in the correct place.

#### Purposes

- To direct subsequent SQL statements to the intended database.
- To avoid fully qualifying every object name with the database name.
- To switch between databases within the same client session for development or administration tasks.

#### Syntax Rules and Structure

**MySQL**:

```sql
USE database_name;
```

**SQL Server**:

```sql
USE database_name;
GO
```

**PostgreSQL** (psql meta‑command or connection string):

```text
\c database_name
```

or in a connection URL: `postgresql://host:5432/database_name`.

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `database_name` | The name of the database to make current. |
| `USE` | Keyword in MySQL and SQL Server. |
| `\c` | psql meta‑command for “connect”. |

**Syntax rules**

- The database must exist.
- The user must have permission to connect to the database (e.g., `CONNECT` privilege in PostgreSQL).
- In SQL Server, `USE` takes effect immediately and persists for the session; `GO` is a batch separator used in client tools.

**Constraints and limitations**

- In PostgreSQL, `USE` is not a SQL statement; you must reconnect to a different database.
- In MySQL, `USE` returns `Database changed` on success.
- In SQL Server, `USE` cannot be used inside a stored procedure to change the database context permanently for the caller.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Select a database

```sql
-- Step 1: Connect to the MySQL server.

-- Step 2: Switch to the "school_db" database.
USE school_db;

-- Step 3: Verify the current database.
SELECT DATABASE();
```

**Expected output**:

```text
Database changed

+------------+
| DATABASE() |
+------------+
| school_db  |
+------------+
1 row in set (0.00 sec)
```

**Why this output appears**: `USE` changes the default database for the session; `SELECT DATABASE()` returns the name of the current database.

##### Example 2: SQL Server – Select a database and query a table

```sql
-- Step 1: Connect to SQL Server.

-- Step 2: Switch to the SalesDB database.
USE SalesDB;
GO

-- Step 3: Query a table without fully qualifying the database name.
SELECT TOP 5 * FROM Customers;
GO
```

**Expected output** (assuming the table exists):

```text
CustomerID  CustomerName
----------- -------------
1           Acme Corp
2           Globex
...
```

**Why this output appears**: `USE SalesDB` sets the database context, so `Customers` resolves to `SalesDB.dbo.Customers`.

##### Example 3: PostgreSQL – Connect to a database with psql

```text
-- Step 1: Start psql connected to the default database.
$ psql -U postgres

-- Step 2: Connect to the "analytics_db" database.
postgres=# \c analytics_db
```

**Expected output**:

```text
You are now connected to database "analytics_db" as user "postgres".
```

**Why this output appears**: The `\c` meta‑command closes the current connection and opens a new connection to the specified database.

#### Real-World Cases

- **Application connection pools**: An application specifies the target database in the connection string; selecting the database is part of establishing the connection.
- **Database administration**: A DBA switches between databases to inspect tables, run maintenance, or apply migrations.
- **Reporting**: An analyst uses `USE` to switch between a data warehouse and a staging database within the same query tool.

---

### 3. Listing Databases

#### Definitions

**Core Definition**  
Listing databases retrieves the names (and sometimes metadata) of all databases available on the server or instance.

**Technical Definition**  
Different RDBMSs expose database catalogs through different mechanisms: `SHOW DATABASES` (MySQL), `\l` or `SELECT datname FROM pg_database` (PostgreSQL), `SELECT name FROM sys.databases` (SQL Server), and `SELECT name FROM v$database` (Oracle).

**Beginner-Friendly Explanation**  
This is how you get a list of all the databases that exist on the server, similar to listing the folders in a filing cabinet.

#### Purposes

- To discover what databases exist on a server.
- To verify that a database was created successfully.
- To audit database inventory for compliance or cleanup.
- To obtain database names for scripting or automation.

#### Syntax Rules and Structure

**MySQL**:

```sql
SHOW DATABASES;
```

**PostgreSQL** (psql):

```text
\l
```

or SQL:

```sql
SELECT datname FROM pg_database;
```

**SQL Server**:

```sql
SELECT name FROM sys.databases;
```

**Oracle**:

```sql
SELECT name FROM v$database;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `SHOW DATABASES` | MySQL command that lists all databases the user has permission to see. |
| `\l` | psql meta‑command that lists databases with owner, encoding, and access privileges. |
| `pg_database` | PostgreSQL system catalog containing one row per database. |
| `sys.databases` | SQL Server system catalog view. |
| `v$database` | Oracle dynamic performance view. |

**Syntax rules**

- In MySQL, `SHOW DATABASES` returns only databases for which the user has some privilege.
- In PostgreSQL, `\l` shows all databases; querying `pg_database` requires connect privilege but typically returns all rows visible to the user.
- In SQL Server, `sys.databases` returns one row per database on the instance.

**Constraints and limitations**

- The exact columns and filtering options vary by RDBMS.
- Some systems hide databases that the user cannot access.
- `SHOW DATABASES` in MySQL does not show the current database explicitly; use `SELECT DATABASE()` for that.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – List all databases

```sql
-- Step 1: Connect to MySQL.

-- Step 2: List all databases.
SHOW DATABASES;
```

**Expected output**:

```text
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| school_db          |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

**Why this output appears**: MySQL returns one row per database that the current user can access. System databases such as `information_schema` and `mysql` are always present.

##### Example 2: PostgreSQL – List databases with owner and encoding

```text
-- Step 1: Start psql connected to any database.
$ psql -U postgres

-- Step 2: Use the \l meta-command.
postgres=# \l
```

**Expected output**:

```text
                                  List of databases
     Name      |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
---------------+----------+----------+-------------+-------------+-----------------------
 analytics_db  | analyst  | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 postgres      | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
               |          |          |             |             | postgres=CTc/postgres
 template1     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
               |          |          |             |             | postgres=CTc/postgres
(4 rows)
```

**Why this output appears**: `\l` queries the system catalogs and formats the result as a table with database name, owner, encoding, locale, and access privileges.

##### Example 3: SQL Server – List databases using a system view

```sql
-- Step 1: Connect to SQL Server.

-- Step 2: Query sys.databases.
SELECT name, database_id, create_date
FROM sys.databases
ORDER BY name;
```

**Expected output**:

```text
name        database_id  create_date
----------- ------------ -----------------------
master      1            2003-04-08 09:13:36.390
model       3            2003-04-08 09:13:36.390
msdb        4            2003-04-08 09:13:36.390
SalesDB     5            2025-01-15 10:22:01.123
tempdb      2            2003-04-08 09:13:36.390
```

**Why this output appears**: `sys.databases` contains one row per database on the instance; `name`, `database_id`, and `create_date` are standard columns.

#### Real-World Cases

- **Inventory and cleanup**: A DBA lists all databases to identify stale test databases that can be dropped.
- **Onboarding**: A new developer runs `SHOW DATABASES` or `\l` to discover the available databases.
- **Automation**: A script lists databases and iterates over them for backup or maintenance tasks.

---

### 4. Renaming Databases

#### Definitions

**Core Definition**  
Renaming a database changes its identifier from an old name to a new name without recreating its contents.

**Technical Definition**  
Renaming is implemented differently across RDBMSs: SQL Server uses `ALTER DATABASE … MODIFY NAME = …`; PostgreSQL uses `ALTER DATABASE … RENAME TO …`; MySQL has no direct `RENAME DATABASE` statement in modern versions and requires creating a new database and moving tables (or using `mysqldump`).

**Beginner-Friendly Explanation**  
This is like changing the label on a drawer without emptying it. However, not every database system supports a simple rename command.

#### Purposes

- To align a database name with new naming conventions.
- To correct a misspelled or outdated database name.
- To reflect a project or organisational change.

#### Syntax Rules and Structure

**SQL Server**:

```sql
ALTER DATABASE old_database_name MODIFY NAME = new_database_name;
```

**PostgreSQL**:

```sql
ALTER DATABASE old_database_name RENAME TO new_database_name;
```

**MySQL** (no direct rename; indirect approach):

```sql
CREATE DATABASE new_database_name;
RENAME TABLE old_db.table1 TO new_db.table1, ...;
DROP DATABASE old_database_name;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `ALTER DATABASE` | Statement used in SQL Server and PostgreSQL to modify database attributes. |
| `MODIFY NAME` | SQL Server clause for renaming. |
| `RENAME TO` | PostgreSQL clause for renaming. |
| `old_database_name` | Current name. |
| `new_database_name` | Desired new name. |

**Syntax rules**

- The database must exist and the user must have the appropriate privileges (owner or superuser in PostgreSQL; `ALTER` permission in SQL Server).
- In PostgreSQL, the current database cannot be renamed; you must connect to a different database first.
- In SQL Server, active connections must be terminated (e.g., set to `SINGLE_USER`) before renaming.
- MySQL does not support `RENAME DATABASE` in modern versions; attempting it may produce an error or be unsupported.

**Constraints and limitations**

- System databases cannot be renamed.
- Renaming fails if there are active connections (SQL Server, PostgreSQL).
- In PostgreSQL, non‑superuser owners must also have `CREATEDB` privilege to rename.
- In MySQL, renaming is not atomic and requires moving every table individually.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: SQL Server – Rename a database safely

```sql
-- Step 1: Connect to the master database.
USE master;
GO

-- Step 2: Set the target database to single-user mode to disconnect users.
ALTER DATABASE University SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
GO

-- Step 3: Rename the database.
ALTER DATABASE University MODIFY NAME = NewUniversity;
GO

-- Step 4: Return the database to multi-user mode.
ALTER DATABASE NewUniversity SET MULTI_USER;
GO
```

**Expected output**:

```text
Commands completed successfully.
```

**Why this output appears**: `SINGLE_USER` terminates existing connections, `MODIFY NAME` performs the rename, and `MULTI_USER` restores normal access.

##### Example 2: PostgreSQL – Rename a database after terminating connections

```sql
-- Step 1: Connect to a different database (not the one to be renamed).
-- Step 2: Terminate active connections to the target database.
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'University';

-- Step 3: Rename the database.
ALTER DATABASE University RENAME TO University1;
```

**Expected output**:

```text
 pg_terminate_backend
----------------------
 t
 t
(2 rows)

ALTER DATABASE
```

**Why this output appears**: `pg_terminate_backend` forcibly disconnects sessions; the subsequent `ALTER DATABASE … RENAME TO` succeeds because no active connections remain.

##### Example 3: MySQL – Rename a database by moving tables

```sql
-- Step 1: Create the new database.
CREATE DATABASE New_University;

-- Step 2: Move each table from the old database to the new database.
RENAME TABLE University.Department TO New_University.Department,
             University.Student    TO New_University.Student,
             University.Course     TO New_University.Course;

-- Step 3: Drop the old database.
DROP DATABASE University;
```

**Expected output**:

```text
Query OK, 1 row affected (0.02 sec)

Query OK, 3 rows affected (0.05 sec)

Query OK, 0 rows affected (0.01 sec)
```

**Why this output appears**: MySQL has no `RENAME DATABASE` statement; the rename is simulated by creating a new database, moving tables with `RENAME TABLE`, and dropping the old database.

#### Real-World Cases

- **Rebranding**: A company renames its application database to reflect a new product name.
- **Naming‑convention migration**: An organisation standardises database names and renames legacy databases.
- **Environment promotion**: A staging database is renamed to production after validation.

---

### 5. Dropping Databases (`DROP DATABASE`)

#### Definitions

**Core Definition**  
`DROP DATABASE` permanently removes a database and all its associated objects and data from the server.

**Technical Definition**  
`DROP DATABASE` deletes the database catalog entry and all physical files (or directory) associated with the database. It is irreversible in most systems without a backup. Syntax varies: MySQL supports `IF EXISTS`; PostgreSQL supports `WITH (FORCE)` to disconnect users; SQL Server requires the session to be connected to another database (typically `master`).

**Beginner-Friendly Explanation**  
This deletes the entire database drawer and everything inside it—tables, data, and settings—permanently. There is no recycle bin, so backups are essential.

#### Purposes

- To remove obsolete test or staging databases.
- To decommission a project’s database after retirement.
- To free disk space on the server.
- To reset a development environment.

#### Syntax Rules and Structure

**MySQL**:

```sql
DROP DATABASE [IF EXISTS] database_name;
```

**PostgreSQL**:

```sql
DROP DATABASE [IF EXISTS] database_name [WITH (FORCE)];
```

**SQL Server**:

```sql
USE master;
GO
DROP DATABASE [IF EXISTS] database_name;
GO
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `DROP DATABASE` | The statement that removes the database. |
| `IF EXISTS` | Suppresses the error if the database does not exist. |
| `WITH (FORCE)` (PostgreSQL 13+) | Terminates active connections before dropping. |
| `USE master` (SQL Server) | Ensures the session is not connected to the database being dropped. |

**Syntax rules**

- The user must have the `DROP` privilege on the database (MySQL) or be the owner/superuser (PostgreSQL, SQL Server).
- In PostgreSQL, you cannot drop the current database; connect to another database first.
- In SQL Server, you cannot drop a database while connected to it; switch to `master`.
- In MySQL, `DROP SCHEMA` is equivalent to `DROP DATABASE`.

**Constraints and limitations**

- `DROP DATABASE` is irreversible; no rollback is possible outside a transaction that has not been committed.
- Active connections block the drop in PostgreSQL unless `WITH (FORCE)` is used.
- In SQL Server, active connections require setting the database to `SINGLE_USER` with `ROLLBACK IMMEDIATE` before dropping.
- System databases cannot be dropped.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Drop a database safely

```sql
-- Step 1: Connect to MySQL with DROP privilege.

-- Step 2: Drop the database if it exists.
DROP DATABASE IF EXISTS school_db;

-- Step 3: Verify it is gone.
SHOW DATABASES LIKE 'school_db';
```

**Expected output**:

```text
Query OK, 0 rows affected (0.01 sec)

Empty set (0.00 sec)
```

**Why this output appears**: `IF EXISTS` prevents an error if the database is absent; `SHOW DATABASES LIKE` returns an empty set, confirming the database no longer exists.

##### Example 2: PostgreSQL – Force‑drop a database with active connections

```sql
-- Step 1: Connect to a different database (e.g., postgres).

-- Step 2: Force-drop the database, terminating any active connections.
DROP DATABASE my_app_db WITH (FORCE);
```

**Expected output**:

```text
DROP DATABASE
```

**Why this output appears**: `WITH (FORCE)` (available in PostgreSQL 13 and later) disconnects all sessions connected to `my_app_db` and then drops the database.

##### Example 3: SQL Server – Drop a database after disconnecting users

```sql
-- Step 1: Connect to master.
USE master;
GO

-- Step 2: Set the database to single-user mode and roll back open transactions.
ALTER DATABASE CompanyData SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
GO

-- Step 3: Drop the database.
DROP DATABASE CompanyData;
GO
```

**Expected output**:

```text
Commands completed successfully.
```

**Why this output appears**: `SINGLE_USER` with `ROLLBACK IMMEDIATE` forcibly disconnects other users; the `DROP DATABASE` statement then succeeds because the session is connected to `master` and no other connections exist.

#### Real-World Cases

- **Test‑environment cleanup**: CI pipelines drop and recreate databases for each test run.
- **Project decommissioning**: When an application is retired, its database is dropped after archival.
- **Disk‑space recovery**: A DBA removes stale databases to free storage on a constrained server.

---

### 6. Database Ownership

#### Definitions

**Core Definition**  
Database ownership identifies the role or user that has ultimate control over a database and its attributes.

**Technical Definition**  
In PostgreSQL, the owner of a database is the role that created it (or the role specified by `OWNER`), and ownership can be changed with `ALTER DATABASE … OWNER TO …`. The owner has implicit privileges and can alter database attributes. In SQL Server, ownership is tied to the login that created the database or can be changed with `ALTER AUTHORIZATION`. MySQL does not have a single “database owner” concept in the same way; privileges are granted per user.

**Beginner-Friendly Explanation**  
The owner is the person or role that “owns” the database drawer and has the final say over its settings and who can use it.

#### Purposes

- To transfer administrative responsibility when a team member leaves or changes roles.
- To ensure that a service account or application role owns the database for automated management.
- To comply with security policies that require clear ownership.
- To enable the owner to alter database configuration without superuser intervention.

#### Syntax Rules and Structure

**PostgreSQL**:

```sql
ALTER DATABASE database_name OWNER TO new_owner;
```

**SQL Server**:

```sql
ALTER AUTHORIZATION ON DATABASE::database_name TO new_owner;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `ALTER DATABASE` | PostgreSQL statement for changing database attributes. |
| `OWNER TO` | Clause specifying the new owner. |
| `new_owner` | Role or user that will become the owner. |
| `ALTER AUTHORIZATION` | SQL Server statement for changing ownership. |

**Syntax rules**

- In PostgreSQL, to change the owner you must be able to `SET ROLE` to the new owning role and have the `CREATEDB` privilege.
- The new owner must exist as a role or login.
- In SQL Server, the new owner must be a valid login or database user.

**Constraints and limitations**

- The current owner must have the authority to transfer ownership.
- In PostgreSQL, the owner of a database cannot be changed while the database is being accessed by other sessions in some configurations.
- SQL Server ownership changes may affect permissions and job ownership.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Change database ownership

```sql
-- Step 1: Connect as a superuser or as the current owner with CREATEDB privilege.

-- Step 2: Transfer ownership of analytics_db to the role "analyst".
ALTER DATABASE analytics_db OWNER TO analyst;

-- Step 3: Verify the new owner.
SELECT datname, pg_get_userbyid(datdba) AS owner
FROM pg_database
WHERE datname = 'analytics_db';
```

**Expected output**:

```text
   datname    |  owner
--------------+---------
 analytics_db | analyst
(1 row)
```

**Why this output appears**: `ALTER DATABASE … OWNER TO` updates the `datdba` column in `pg_database` to the OID of the new owner role; the query resolves the OID to the role name.

##### Example 2: SQL Server – Change database ownership

```sql
-- Step 1: Connect to SQL Server with sufficient authority.

-- Step 2: Change the owner of SalesDB to the login "app_owner".
ALTER AUTHORIZATION ON DATABASE::SalesDB TO app_owner;
GO
```

**Expected output**:

```text
Commands completed successfully.
```

**Why this output appears**: `ALTER AUTHORIZATION` changes the owner of the database to the specified login, granting that login ownership‑level permissions.

#### Real-World Cases

- **Employee departure**: When a DBA leaves, ownership of databases is transferred to a service account or another DBA.
- **Application lifecycle**: A database is owned by an application service account so that automated deployments can alter configuration.
- **Security audits**: Ownership is reviewed and standardised to ensure that no individual holds orphaned ownership.

---

### 7. Database Configuration

#### Definitions

**Core Definition**  
Database configuration is the set of per‑database parameters that control behaviour such as connection limits, default tablespace, statement timeout, and other runtime settings.

**Technical Definition**  
Configuration parameters can be set at the database level using `ALTER DATABASE … SET parameter …` in PostgreSQL, `ALTER DATABASE … SET …` in SQL Server, and `ALTER DATABASE …` options in MySQL. These settings become the default for new sessions connecting to that database.

**Beginner-Friendly Explanation**  
This is how you adjust the “settings” of a database drawer—such as how many people can open it at once or how long a query can run—without changing the whole server’s settings.

#### Purposes

- To set a database‑specific statement timeout for an application.
- To limit the number of concurrent connections to a database.
- To control default tablespace or file‑growth behaviour.
- To apply security‑related settings such as `ALLOW_CONNECTIONS`.

#### Syntax Rules and Structure

**PostgreSQL** (official):

```sql
ALTER DATABASE name SET configuration_parameter { TO | = } { value | DEFAULT };
ALTER DATABASE name RESET configuration_parameter;
ALTER DATABASE name RESET ALL;
```

**SQL Server** (SET options):

```sql
ALTER DATABASE database_name SET option_name = value;
```

**MySQL** (database‑level character set and collation via `ALTER DATABASE`):

```sql
ALTER DATABASE db_name
    [DEFAULT] CHARACTER SET [=] charset_name
    [DEFAULT] COLLATE [=] collation_name;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `ALTER DATABASE` | Statement for modifying database attributes. |
| `SET configuration_parameter` | Assigns a session default for the specified parameter. |
| `RESET` | Removes a database‑specific setting, reverting to the server default. |
| `RESET ALL` | Removes all database‑specific settings. |
| `SET option_name = value` (SQL Server) | Sets a database option such as `AUTO_CLOSE`, `READ_ONLY`, etc. |

**Syntax rules**

- Only the database owner or a superuser can change database‑level configuration in PostgreSQL.
- Some parameters cannot be set at the database level or can only be set by a superuser.
- The setting applies to new sessions started after the change; existing sessions are unaffected.
- In SQL Server, some `SET` options take effect immediately, while others require a restart or new connection.

**Constraints and limitations**

- Not all server parameters can be set per database.
- Overrides from `postgresql.conf` or command‑line settings are superseded by database‑level defaults for new sessions.
- In PostgreSQL, `ALTER DATABASE … SET TABLESPACE` physically moves files and requires that no one is connected to the database.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Set a statement timeout for a database

```sql
-- Step 1: Connect as the database owner or a superuser.

-- Step 2: Set the default statement_timeout for the analytics_db database to 60 seconds.
ALTER DATABASE analytics_db SET statement_timeout TO '60s';

-- Step 3: Verify the setting.
SELECT datname, setting
FROM pg_db_role_setting s
JOIN pg_database d ON d.oid = s.setdatabase
CROSS JOIN LATERAL unnest(s.setconfig) AS setting
WHERE d.datname = 'analytics_db'
  AND setting LIKE 'statement_timeout=%';
```

**Expected output**:

```text
   datname    |          setting
--------------+----------------------------
 analytics_db | statement_timeout=60s
(1 row)
```

**Why this output appears**: `ALTER DATABASE … SET` stores the parameter in `pg_db_role_setting` for the database; new sessions connecting to `analytics_db` will inherit `statement_timeout=60s`.

##### Example 2: SQL Server – Set a database to read‑only

```sql
-- Step 1: Connect to master or another database.

-- Step 2: Set the SalesDB database to read-only.
ALTER DATABASE SalesDB SET READ_ONLY;
GO
```

**Expected output**:

```text
Commands completed successfully.
```

**Why this output appears**: The `SET READ_ONLY` option changes the database’s updateability; users can still read data but cannot modify it.

##### Example 3: MySQL – Change the default character set of a database

```sql
-- Step 1: Connect to MySQL with ALTER privilege on the database.

-- Step 2: Change the default character set and collation.
ALTER DATABASE school_db
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_0900_ai_ci;
```

**Expected output**:

```text
Query OK, 1 row affected (0.01 sec)
```

**Why this output appears**: `ALTER DATABASE` updates the database’s default character set and collation, affecting newly created tables that do not specify their own character set.

#### Real-World Cases

- **Application performance tuning**: A database‑specific `statement_timeout` prevents runaway queries in a reporting database.
- **Connection governance**: `CONNECTION LIMIT` ensures that a single database cannot exhaust the server’s connection pool.
- **Compliance**: A database is set to `READ_ONLY` during an audit period to prevent data modification.

---

### 8. Database‑Level Permissions

#### Definitions

**Core Definition**  
Database‑level permissions control which users or roles can perform specific actions on a database, such as connecting, creating objects, or dropping the database.

**Technical Definition**  
Privileges are granted and revoked with `GRANT` and `REVOKE`. In MySQL, privileges such as `CREATE`, `DROP`, `ALTER`, `SELECT`, `INSERT`, and `CONNECT` can be granted at the database level. In PostgreSQL, privileges such as `CONNECT`, `CREATE`, `TEMPORARY`, and `ALL` are granted on databases. SQL Server uses roles and permissions such as `CONNECT`, `CREATE TABLE`, etc.

**Beginner-Friendly Explanation**  
Permissions decide who is allowed to open the database drawer, who can add or remove things inside it, and who can change its settings.

#### Purposes

- To enforce least‑privilege access so users can only perform necessary actions.
- To grant an application account the ability to connect and read/write data without administrative rights.
- To revoke public access to sensitive databases.
- To delegate database‑specific administration to a team lead or service owner.

#### Syntax Rules and Structure

**MySQL** (database‑level grant):

```sql
GRANT privilege_type [, ...] ON database_name.* TO user [,...];
REVOKE privilege_type [, ...] ON database_name.* FROM user [,...];
```

**PostgreSQL** (database‑level grant):

```sql
GRANT privilege_type [, ...] ON DATABASE database_name TO role [,...];
REVOKE privilege_type [, ...] ON DATABASE database_name FROM role [,...];
```

**SQL Server** (database‑level permission) :

```sql
GRANT permission ON DATABASE::database_name TO user;
DENY permission ON DATABASE::database_name TO user;
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `GRANT` | Statement that adds a privilege. |
| `REVOKE` | Statement that removes a privilege. |
| `ON database_name.*` (MySQL) | Scope: all objects in the database. |
| `ON DATABASE database_name` (PostgreSQL) | Scope: the database itself. |
| `TO user` / `FROM user` | Target user or role. |
| `privilege_type` | e.g., `CONNECT`, `CREATE`, `TEMPORARY`, `ALL`. |

**Syntax rules**

- The grantor must have the privilege with grant option or be a superuser.
- In MySQL, database‑level privileges are stored in the `mysql.db` grant table.
- In PostgreSQL, the `PUBLIC` pseudo‑role can be granted or revoked privileges on a database.
- SQL Server uses `DENY` to explicitly block a permission, which overrides grants.

**Constraints and limitations**

- Privileges granted at the database level apply to all objects within the database (unless overridden at a finer granularity).
- Revoking `CONNECT` from `PUBLIC` prevents any user without an explicit `CONNECT` grant from connecting to the database.
- In MySQL, `GRANT ALL ON database.*` does not grant server‑level administrative privileges.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Grant read‑only access to a database

```sql
-- Step 1: Connect as a user with GRANT OPTION.

-- Step 2: Create a user (if not exists).
CREATE USER IF NOT EXISTS 'report_user'@'%' IDENTIFIED BY 'StrongPass123!';

-- Step 3: Grant SELECT on all tables in school_db.
GRANT SELECT ON school_db.* TO 'report_user'@'%';

-- Step 4: Verify the grant.
SHOW GRANTS FOR 'report_user'@'%';
```

**Expected output**:

```text
Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

+---------------------------------------------------+
| Grants for report_user@%                          |
+---------------------------------------------------+
| GRANT USAGE ON *.* TO `report_user`@`%`           |
| GRANT SELECT ON `school_db`.* TO `report_user`@`%`|
+---------------------------------------------------+
2 rows in set (0.00 sec)
```

**Why this output appears**: `GRANT SELECT ON school_db.*` adds a row to the `mysql.db` grant table; `SHOW GRANTS` confirms the database‑level `SELECT` privilege.

##### Example 2: PostgreSQL – Revoke public connect and grant to a specific role

```sql
-- Step 1: Connect as the database owner or a superuser.

-- Step 2: Revoke CONNECT from PUBLIC on the analytics_db database.
REVOKE CONNECT ON DATABASE analytics_db FROM PUBLIC;

-- Step 3: Grant CONNECT to the role analyst.
GRANT CONNECT ON DATABASE analytics_db TO analyst;
```

**Expected output**:

```text
REVOKE
GRANT
```

**Why this output appears**: Revoking `CONNECT` from `PUBLIC` prevents any role from connecting unless explicitly granted; the subsequent `GRANT CONNECT` allows `analyst` to connect.

##### Example 3: SQL Server – Grant database‑level permissions

```sql
-- Step 1: Connect to the target database or master.

-- Step 2: Grant CONNECT and CREATE TABLE to the user app_user.
GRANT CONNECT ON DATABASE::SalesDB TO app_user;
GRANT CREATE TABLE ON DATABASE::SalesDB TO app_user;
GO
```

**Expected output**:

```text
Commands completed successfully.
```

**Why this output appears**: `GRANT CONNECT` allows the user to connect; `GRANT CREATE TABLE` allows the user to create tables within the database.

#### Real-World Cases

- **Application accounts**: An application’s service account is granted `CONNECT`, `SELECT`, `INSERT`, `UPDATE`, and `DELETE` on its database, but no `DROP` or `ALTER` privileges.
- **Reporting users**: A reporting role receives `SELECT` only on a data warehouse database.
- **Security hardening**: `CONNECT` is revoked from `PUBLIC` on sensitive databases, and only named roles are granted access.

---

## References

- PostgreSQL Documentation – `CREATE DATABASE` – https://www.postgresql.org/docs/current/sql-createdatabase.html 
- PostgreSQL Documentation – `ALTER DATABASE` – https://www.postgresql.org/docs/current/sql-alterdatabase.html 
- PostgreSQL Documentation – Database Configuration – https://www.postgresql.org/docs/current/runtime-config.html 
- MySQL 8.4 Reference Manual – `CREATE DATABASE` – https://dev.mysql.com/doc/refman/8.4/en/create-database.html 
- MySQL 8.4 Reference Manual – Privileges Provided by MySQL – https://dev.mysql.com/doc/refman/8.4/en/privileges-provided.html 
- Microsoft Learn – `CREATE DATABASE` (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/statements/create-database-transact-sql 
- Microsoft Learn – `ALTER DATABASE SET` Options – https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-transact-sql-set-options 
- SQL in a Nutshell, 3rd Edition – `CREATE/ALTER DATABASE` – https://www.oreilly.com/library/view/sql-in-a/9780596155322/re07.html 
- CASRAI – Creating a Database in SQL – https://casrai.org/guides/creating-a-database-sql 
- Baeldung – How to Rename a Database in SQL – https://www.baeldung.com/sql/rename-database 
- Info2Soft – How to Drop a Database in SQL Server, MySQL & PostgreSQL – https://www.info2soft.com/blogs/how-to-drop-database-in-sql.html 
- Database.Guide – SQL `CREATE DATABASE` Syntax Listed by DBMS – https://database.guide/sql-create-database-syntax-listed-by-dbms/ 