# SQL Database Security Fundamentals: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL database security is the collection of tools, controls, and processes designed to protect databases from unauthorized access, misuse, and corruption, encompassing authentication, authorization, least privilege, security boundaries, user management, and role-based access control.

**Technical Definition:** Database security is a layered defense model that operates at multiple scopes: the network perimeter (firewalls, TLS), the instance level (authentication, server roles), the database level (users, database roles, permissions), and the object level (table, column, and row-level permissions). It is governed by the principle of least privilege, which requires that all users in an information system should be granted as few privileges as required to perform their duties. The security model is typically implemented through declarative SQL statements (`CREATE USER`, `CREATE ROLE`, `GRANT`, `REVOKE`) and configuration parameters that control authentication methods, password policies, and resource limits.

**Beginner-Friendly Explanation:** Database security is like the security system of a large office building. Authentication is the front door—it checks who you are. Authorization is the keycard system—it decides which rooms you can enter. Least privilege is the rule that you only get keys to the rooms you actually need. Security boundaries are the locked doors between departments. Users are the people who work in the building, and roles are the job titles that come with a standard set of keys. SQL is the language you use to set up all of these controls.

### Key Characteristics

- **Layered defense:** Security operates at multiple levels—network, instance, database, and object—to provide defense in depth.
- **Declarative:** Users, roles, and permissions are managed through SQL DDL and DCL statements.
- **Principle-driven:** Least privilege is the foundational security principle; every user should have the minimum access necessary.
- **Vendor-varied:** Authentication methods, role systems, and privilege models differ significantly across PostgreSQL, MySQL, SQL Server, and Oracle.
- **Auditable:** Security-relevant actions should be logged for compliance and forensic analysis.
- **Dynamic:** Permissions can be granted, revoked, or denied at any time; changes take effect for new sessions.

### Prerequisites

- Basic understanding of SQL DDL and DCL statements (`CREATE`, `GRANT`, `REVOKE`).
- Familiarity with database users, schemas, and objects.
- Knowledge of network security concepts (firewalls, TLS).
- Awareness of authentication protocols (Kerberos, LDAP, SCRAM).
- Understanding of role-based access control (RBAC).

### Related Programming Areas

- Database administration (DBA) and DevOps.
- Application security and secure coding.
- Compliance and auditing (GDPR, HIPAA, PCI-DSS).
- Identity and access management (IAM).
- Cloud database security (RDS, Azure SQL, Cloud SQL).

### Core Concepts / Features

1. **Authentication** (verifying user identity)
2. **Authorization** (determining access permissions)
3. **Least Privilege** (restricting access to the minimum necessary)
4. **Security Boundaries** (isolating logical environments)
5. **Database Users** (creating, altering, dropping accounts)
6. **Roles** (grouping privileges into logical containers)


## Core Concept 1: Authentication

### Definitions

**Core Definition:** Authentication is the process of verifying a user's identity—confirming that a user is who they claim to be—before allowing access to the database.

**Technical Definition:** Database authentication verifies the identity of a principal (user or application) by validating credentials against an identity store. Authentication methods include: native password authentication (SCRAM-SHA-256, caching_sha2_password, SQL Server authentication), external identity providers (LDAP, RADIUS, OpenID Connect, WebAuthn), Multi-Factor Authentication (MFA), IAM database authentication (AWS, Azure), and Kerberos/Active Directory integration. The authentication method is configured at the instance level (e.g., `pg_hba.conf` in PostgreSQL, `authentication_policy` in MySQL, server security page in SQL Server) and checked on every connection attempt.

**Beginner-Friendly Explanation:** Authentication is like showing your ID at the door. Before you can enter the building, you must prove you are who you say you are—by showing a password, a fingerprint, or a badge. The database checks your ID against its records and only lets you in if it matches.

### Purposes

- **To verify user identity** before granting access to the database.
- **To support enterprise identity integration** by delegating authentication to Active Directory, LDAP, or Kerberos.
- **To enforce strong credentials** through password hashing algorithms (SCRAM-SHA-256, SHA-256) and MFA.
- **To enable single sign-on (SSO)** for users in Windows or Kerberos environments.
- **To support non-domain clients** through SQL Server authentication or IAM database authentication.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL — SCRAM-SHA-256)

```sql
-- Set the password encryption method
ALTER SYSTEM SET password_encryption = 'scram-sha-256';

-- Create a user with SCRAM-SHA-256 password
CREATE ROLE app_user LOGIN PASSWORD 'StrongPassword123!';
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `password_encryption` | `scram-sha-256` is the most secure password-based method; `md5` is legacy and weak. |
| `pg_hba.conf` | Host-based authentication file controls which authentication method is used for each connection type. |

#### Complete General Syntax (MySQL — caching_sha2_password)

```sql
-- Create a user with caching_sha2_password (default in MySQL 8.0+)
CREATE USER 'app_user'@'%'
  IDENTIFIED WITH caching_sha2_password BY 'StrongPassword123!';

-- Or with Kerberos
CREATE USER 'app_user'@'%'
  IDENTIFIED WITH authentication_kerberos AS 'kerberos_principal';
```

#### Complete General Syntax (SQL Server — Windows and SQL Authentication)

```sql
-- Windows Authentication (default, more secure)
-- Login is created from a Windows account
CREATE LOGIN [DOMAIN\username] FROM WINDOWS;

-- SQL Server Authentication (mixed mode)
CREATE LOGIN app_login
  WITH PASSWORD = 'StrongP@ssw0rd!'
  MUST_CHANGE, CHECK_EXPIRATION = ON;
```

#### Syntax Rules

- **PostgreSQL:** The `scram-sha-256` method is the most secure of the currently provided methods, but it is not supported by older client libraries. It stores a salted SCRAM verifier in `pg_authid.rolpassword`.
- **MySQL:** The `caching_sha2_password` plugin uses SHA-256 for password hashing and caches authentication data for better performance. It is the default in MySQL 8.0+.
- **SQL Server:** Windows Authentication is much more secure than SQL Server Authentication. When possible, use Windows Authentication.
- **Oracle:** External authentication allows an external service (operating system or network) to perform password administration and user authentication. Operating system authentication takes precedence over password file authentication.

#### Constraints and Limitations

- **PostgreSQL:** `scram-sha-256` requires PostgreSQL 10+ and client library support. Older clients cannot authenticate.
- **MySQL:** The `mysql_native_password` plugin is deprecated and subject to removal in a future version of MySQL. Use `caching_sha2_password` instead.
- **SQL Server:** Changing the security mode requires a restart of the service. The `sa` account is not automatically enabled when switching to mixed mode.
- **Oracle:** Externally authenticated users are authenticated by the operating system, and Oracle Database allows operating system-authenticated logins only over secure connections.

### Annotated Code Examples

#### Example 1: PostgreSQL — SCRAM-SHA-256 Authentication

```sql
-- Step 1: Set password encryption to SCRAM-SHA-256
ALTER SYSTEM SET password_encryption = 'scram-sha-256';

-- Step 2: Reload configuration
SELECT pg_reload_conf();

-- Step 3: Create a user with a SCRAM-SHA-256 password
CREATE ROLE secure_user LOGIN PASSWORD 'Str0ng!P@ssw0rd';

-- Step 4: Verify the password verifier is SCRAM
SELECT rolname, rolpassword FROM pg_authid WHERE rolname = 'secure_user';
```

**Expected Output:**

```
 rolname     | rolpassword
-------------+----------------------------------------------
 secure_user | SCRAM-SHA-256$4096:...
```

**Why This Works:** `ALTER SYSTEM SET password_encryption = 'scram-sha-256'` changes the default encryption method for new passwords. The password is stored as a salted SCRAM verifier, not as plaintext or an MD5 hash. This is the recommended authentication method for PostgreSQL.

#### Example 2: SQL Server — Windows Authentication

```sql
-- Create a login from a Windows domain account
CREATE LOGIN [CONTOSO\jdoe] FROM WINDOWS;

-- Create a database user mapped to the login
USE CompanyDB;
CREATE USER [CONTOSO\jdoe] FOR LOGIN [CONTOSO\jdoe];

-- Grant permissions
GRANT SELECT ON SCHEMA::dbo TO [CONTOSO\jdoe];
```

**Expected Output:**

```
Commands completed successfully.
```

**Why This Works:** Windows Authentication uses the security credentials of the Windows operating system to validate user connections. SQL Server does not store or manage passwords directly—it relies on the Windows domain controller (Active Directory or local accounts) for credential validation. This provides single sign-on (SSO) and centralized password policy management.

### Real-World Cases

- **Enterprise applications:** Using Windows Authentication or Kerberos for SSO integration.
- **Cloud-native applications:** Using IAM database authentication (AWS, Azure) to avoid managing database passwords.
- **Web applications:** Using SCRAM-SHA-256 or caching_sha2_password with strong password policies.
- **Regulated industries:** Using MFA and certificate-based authentication for compliance.

### References

- PostgreSQL: Authentication Methods — https://www.postgresql.org/docs/current/auth-methods.html
- PostgreSQL: Password Authentication (SCRAM-SHA-256) — https://www.postgresql.org/docs/current/auth-password.html
- MySQL: Pluggable Authentication — https://dev.mysql.com/doc/refman/8.0/en/pluggable-authentication.html
- SQL Server: Server Properties (Security page) — https://learn.microsoft.com/en-us/sql/database-engine/configure-windows/server-properties-security-page
- Oracle: Configuring External Authentication — https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/configuring-external-service-authenticate-users-and-passwords.html


## Core Concept 2: Authorization

### Definitions

**Core Definition:** Authorization is the process of determining what actions an authenticated user is permitted to perform—which objects they can access and which operations they can execute.

**Technical Definition:** Authorization in SQL databases is implemented through the privilege system: permissions are granted to users or roles on specific objects (tables, views, schemas, functions) or at the database/instance level. The execution context of stored procedures and functions is controlled by the `SECURITY DEFINER` (execute with the privileges of the object owner) or `SECURITY INVOKER` (execute with the privileges of the calling user) clause. Session-level security variables (e.g., `SET ROLE`, `SET SESSION AUTHORIZATION`) allow temporary privilege changes within a session.

**Beginner-Friendly Explanation:** Authorization is like the keycard system in an office. After you show your ID at the front door (authentication), the keycard system decides which rooms you can enter. You might have access to your own office, the break room, and the conference room—but not the server room or the executive floor. The `SECURITY DEFINER` vs. `SECURITY INVOKER` choice is like deciding whether a task is performed with your own keys or with the keys of the person who wrote the instructions.

### Purposes

- **To control access to data and operations** at the granularity of objects and actions.
- **To support the principle of least privilege** by granting only the permissions necessary for a user's role.
- **To control execution context** of stored procedures and functions using `SECURITY DEFINER` or `SECURITY INVOKER`.
- **To enable temporary privilege escalation** using `SET ROLE` for specific tasks.
- **To support separation of duties** through distinct roles for different administrative functions.

### Syntax Rules and Structure

#### Complete General Syntax (GRANT and REVOKE)

```sql
-- Grant a privilege
GRANT { { SELECT | INSERT | UPDATE | DELETE | TRUNCATE | REFERENCES | TRIGGER }
    [, ...] | ALL [ PRIVILEGES ] }
    ON { [ TABLE ] table_name [, ...]
       | ALL TABLES IN SCHEMA schema_name [, ...] }
    TO role_specification [, ...]
    [ WITH GRANT OPTION ];

-- Revoke a privilege
REVOKE [ GRANT OPTION FOR ]
    { { SELECT | INSERT | UPDATE | DELETE | TRUNCATE | REFERENCES | TRIGGER }
    [, ...] | ALL [ PRIVILEGES ] }
    ON { [ TABLE ] table_name [, ...]
       | ALL TABLES IN SCHEMA schema_name [, ...] }
    FROM role_specification [, ...]
    [ CASCADE | RESTRICT ];
```

#### Complete General Syntax (SECURITY DEFINER / INVOKER)

```sql
-- Oracle: AUTHID clause
CREATE OR REPLACE PROCEDURE my_proc
AUTHID DEFINER  -- or AUTHID CURRENT_USER (invoker)
AS
BEGIN
    ...
END;

-- MySQL: SQL SECURITY clause
CREATE DEFINER = 'admin'@'localhost' SQL SECURITY DEFINER PROCEDURE my_proc()
BEGIN
    ...
END;
```

#### Syntax Rules

- **PostgreSQL:** Privileges can be granted at the table, schema, database, or instance level. `WITH GRANT OPTION` allows the grantee to grant the same privilege to others.
- **PostgreSQL:** `SECURITY DEFINER` functions execute with the privileges of the function owner; `SECURITY INVOKER` executes with the privileges of the calling user. The default is `SECURITY INVOKER`.
- **MySQL:** The `DEFINER` clause specifies the account used to check access privileges when the routine is executed. The legal `SQL SECURITY` characteristic values are `DEFINER` and `INVOKER`.
- **Oracle:** `AUTHID DEFINER` (users execute code with the owner's privileges) or `AUTHID CURRENT_USER` (users execute with their own privileges). Administrative privileges such as `SYSDBA` are not inherited from the invoking session.

#### Constraints and Limitations

- **Security definer risk:** If a `SECURITY DEFINER` procedure is not carefully written, a low-privileged user could exploit it to gain higher privileges.
- **Oracle 12c:** Use the `INHERIT [ANY] PRIVILEGES` privilege to make it impossible for a lower-privileged user to take advantage of a higher-privileged user via an invoker rights unit.
- **Session-level security:** `SET ROLE` and `SET SESSION AUTHORIZATION` are session-scoped; they revert when the session ends.

### Annotated Code Examples

#### Example 1: PostgreSQL — Granting and Revoking Table Privileges

```sql
-- Create a user
CREATE ROLE analyst LOGIN PASSWORD 'AnalystPass123!';

-- Grant SELECT on all tables in the sales schema
GRANT SELECT ON ALL TABLES IN SCHEMA sales TO analyst;

-- Grant INSERT on a specific table
GRANT INSERT ON sales.orders TO analyst;

-- Revoke INSERT
REVOKE INSERT ON sales.orders FROM analyst;

-- Grant with GRANT OPTION
GRANT SELECT ON sales.customers TO analyst WITH GRANT OPTION;
```

**Expected Output:**

```
GRANT
REVOKE
GRANT
```

**Why This Works:** `GRANT SELECT ON ALL TABLES IN SCHEMA` grants read access to all current and future tables in the schema. `REVOKE` removes a permission. `WITH GRANT OPTION` allows the analyst to grant the same permission to other users.

#### Example 2: MySQL — SECURITY DEFINER vs. INVOKER

```sql
-- Create a procedure with SECURITY DEFINER
DELIMITER //
CREATE DEFINER = 'admin'@'localhost'
SQL SECURITY DEFINER
PROCEDURE get_user_count()
BEGIN
    SELECT COUNT(*) FROM mysql.user;
END;
//
DELIMITER ;

-- The procedure runs with 'admin' privileges, not the caller's.
-- A low-privileged user with EXECUTE can call it.
```

**Expected Output:**

```
Query OK, 0 rows affected
```

**Why This Works:** The procedure is defined with `SECURITY DEFINER` and `DEFINER = 'admin'@'localhost'`. When a low-privileged user calls `get_user_count()`, the procedure executes with the `admin` account's privileges, allowing it to read `mysql.user` even though the caller lacks direct `SELECT` permission on that table.

### Real-World Cases

- **Application access:** Granting `SELECT`, `INSERT`, `UPDATE`, `DELETE` on application tables to the application service account.
- **Reporting:** Granting `SELECT` on views to reporting users without granting access to base tables.
- **Stored procedures:** Using `SECURITY DEFINER` to allow controlled access to sensitive data through a vetted interface.
- **Temporary elevation:** Using `SET ROLE` to temporarily assume a more privileged role for a specific administrative task.

### References

- PostgreSQL: GRANT — https://www.postgresql.org/docs/current/sql-grant.html
- PostgreSQL: CREATE FUNCTION (SECURITY DEFINER) — https://www.postgresql.org/docs/current/sql-createfunction.html
- MySQL: CREATE PROCEDURE (SQL SECURITY) — https://dev.mysql.com/doc/refman/8.0/en/create-procedure.html
- Oracle: About Definer's Rights and Invoker's Rights — https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/managing-security-for-definers-rights-and-invokers-rights.html


## Core Concept 3: Least Privilege

### Definitions

**Core Definition:** Least privilege is the security principle that every user, process, or program should be granted only the minimum privileges necessary to perform its required functions, and no more.

**Technical Definition:** The principle of least privilege requires that all users in an information system should be granted as few privileges as required to perform their duties. Thus, extraneous privileges like resetting passwords are only granted to those users that absolutely require such privileges. In database terms, this means: using named accounts instead of generic superuser accounts (SYS, SYSTEM, root, sa, postgres); granting privileges to roles rather than directly to users; using separate accounts for different duties (separation of duties); and regularly reviewing and revoking unnecessary privileges.

**Beginner-Friendly Explanation:** Least privilege is like giving employees keys to only the doors they need to do their job. A receptionist does not need a key to the server room. A developer does not need access to production customer data. By limiting access, you reduce the damage that can be done if an account is compromised.

### Purposes

- **To minimize the attack surface** by reducing the number of privileged accounts.
- **To limit the blast radius** of a compromised account.
- **To enforce separation of duties** so that no single person can perform all critical operations.
- **To comply with regulatory requirements** (SOX, HIPAA, PCI-DSS) that mandate least privilege.
- **To prevent accidental or malicious misuse** of privileges by insiders.

### Syntax Rules and Structure

#### Complete General Syntax (Named Administrative Accounts)

```sql
-- Oracle: Create a named DBA account instead of using SYS or SYSTEM
CREATE USER jsmith IDENTIFIED BY "StrongPass123!"
  DEFAULT TABLESPACE users
  QUOTA UNLIMITED ON users;

GRANT DBA TO jsmith;

-- Revoke unnecessary privileges from PUBLIC
REVOKE SELECT ON sys.aud$ FROM PUBLIC;
```

#### Complete General Syntax (SQL Server — Custom Administrative Role)

```sql
-- Create a custom role with only the necessary permissions
CREATE ROLE app_admin;

GRANT SELECT, INSERT, UPDATE, DELETE ON SCHEMA::app TO app_admin;
GRANT EXECUTE ON SCHEMA::app TO app_admin;

-- Add a user to the role
ALTER ROLE app_admin ADD MEMBER [app_user];
```

#### Syntax Rules

- **Named accounts:** Oracle recommends using named accounts (e.g., jsmith, cmack, gkramer) instead of shared, or generic, accounts. Generic accounts like SYS and SYSTEM should not be used except for patching, upgrading, and special circumstances.
- **Separation of duties:** If any default database user account is required, a DBA must unlock and activate that account with a new, secure password. Grant necessary privileges only.
- **Role-based access:** Use roles instead of direct user permissions. Roles can be granted and revoked without modifying individual user accounts.
- **Regular review:** Regularly review and revoke unnecessary privileges.

#### Constraints and Limitations

- **Superuser accounts:** PostgreSQL's `postgres` role and SQL Server's `sa` account are superusers; they should not be used for routine work.
- **Oracle SYS/SYSTEM:** Once Oracle Database Vault is enabled, SYS is no longer able to perform certain actions. This is intentional because SYS should not be an account used except for patching, upgrading, and special circumstances.
- **Privilege creep:** Over time, users accumulate privileges as they change roles. Regular audits are necessary to detect and remove excessive privileges.

### Annotated Code Examples

#### Example 1: Oracle — Creating Named Administrative Accounts

```sql
-- Create a named account for a DBA
CREATE USER jsmith IDENTIFIED BY "SecurePass123!"
  DEFAULT TABLESPACE users
  QUOTA UNLIMITED ON users;

-- Grant only the necessary administrative role
GRANT DBA TO jsmith;

-- Create a separate account for account management
CREATE USER cmack IDENTIFIED BY "AnotherPass456!"
  DEFAULT TABLESPACE users;

-- Grant only the account management privilege
GRANT CREATE USER TO cmack;
GRANT ALTER USER TO cmack;
GRANT DROP USER TO cmack;
```

**Expected Output:**

```
User created.
Grant succeeded.
User created.
Grant succeeded.
Grant succeeded.
Grant succeeded.
```

**Why This Works:** Instead of using the generic SYS or SYSTEM accounts, named accounts (jsmith, cmack) are created with specific responsibilities. jsmith has the DBA role for database administration; cmack has only the privileges needed for account management. This enforces separation of duties.

#### Example 2: SQL Server — Custom Administrative Role

```sql
-- Create a custom role for application administration
CREATE ROLE app_admin;

-- Grant only the necessary permissions
GRANT SELECT, INSERT, UPDATE, DELETE ON SCHEMA::app TO app_admin;
GRANT EXECUTE ON SCHEMA::app TO app_admin;

-- Create a login and user, and add to the role
CREATE LOGIN app_admin_login WITH PASSWORD = 'AppAdminP@ss1!';
USE AppDB;
CREATE USER app_admin_user FOR LOGIN app_admin_login;
ALTER ROLE app_admin ADD MEMBER app_admin_user;
```

**Expected Output:**

```
Commands completed successfully.
```

**Why This Works:** The `app_admin` role has only the permissions needed to manage the application schema—not full `sysadmin` or `db_owner` rights. This limits the blast radius if the account is compromised.

### Real-World Cases

- **Production databases:** Application service accounts with only `SELECT`, `INSERT`, `UPDATE`, `DELETE` on their schema.
- **DBA duties:** Separate accounts for backup, patching, and account management.
- **Developer access:** Read-only access to production data for debugging, with separate write access in development.
- **Compliance:** PCI-DSS and HIPAA require least privilege and separation of duties.

### References

- Oracle: Managing Database Users (Least Privilege) — https://docs.oracle.com/en/database/oracle/oracle-database/23/dvgsg/database-vault-getting-started-guide.pdf
- Oracle: Fine-Tune Privilege Management — https://asktom.oracle.com/Misc/oramag/fine-tune-privilege-management.html
- OWASP: Database Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Database_Security_Cheat_Sheet.html
- Microsoft: SQL Server Security Best Practices — https://learn.microsoft.com/en-us/sql/relational-databases/security/sql-server-security-best-practices


## Core Concept 4: Security Boundaries

### Definitions

**Core Definition:** Security boundaries are the logical and physical separations between database environments—clusters, databases, networks, and firewall rules—that prevent unauthorized access and limit the scope of a security breach.

**Technical Definition:** Security boundaries isolate logical environments through: separate physical clusters (different servers or virtual machines), independent databases within a cluster, distinct network topologies (VLANs, subnets), firewall rule configurations (IP allowlists, security groups), and transport layer security (TLS) for encrypted connections. The principle is that the database should be isolated from other servers and only connect with as few hosts as possible.

**Beginner-Friendly Explanation:** Security boundaries are like the walls, doors, and locks between different parts of a building. The server room is behind a locked door. The finance department has its own wing. Visitors can only access the lobby. By creating these boundaries, you make it harder for an attacker to move from one part of the system to another.

### Purposes

- **To isolate production from development and test** environments to prevent accidental data corruption.
- **To restrict network access** to the database to only authorized hosts.
- **To encrypt data in transit** using TLS to prevent eavesdropping.
- **To separate tenants** in a multi-tenant architecture.
- **To comply with data residency requirements** by physically isolating data in specific regions.

### Syntax Rules and Structure

#### Complete General Syntax (Network Isolation — OWASP Guidance)

```text
The application's backend database should be isolated from other servers and only connect with as few hosts as possible.
```

**Recommended measures:** 
- Disabling network (TCP) access and requiring all access is over a local socket file or named pipe.
- Configuring the database to only bind on localhost.
- Restricting access to the network port to specific hosts with firewall rules.
- Placing the database server on a dedicated internal network segment that is isolated from the application server.
- Protecting any web-based management tools (e.g., phpMyAdmin, pgAdmin) with authentication, HTTPS, and network restrictions.

#### Complete General Syntax (TLS Encryption)

```text
-- PostgreSQL: Require SSL for all connections
ALTER SYSTEM SET ssl = on;
-- In pg_hba.conf: hostssl all all 0.0.0.0/0 scram-sha-256

-- MySQL: Require SSL
ALTER USER 'app_user'@'%' REQUIRE SSL;

-- SQL Server: Force Encryption
-- In SQL Server Configuration Manager: Force Encryption = Yes
```

#### Syntax Rules

- **PostgreSQL:** Use `hostssl` in `pg_hba.conf` to require SSL for specific connections. The `ssl` parameter must be `on`.
- **MySQL:** `REQUIRE SSL` enforces encrypted connections for the account. `REQUIRE X509` requires a valid client certificate.
- **SQL Server:** Configure Force Encryption in SQL Server Configuration Manager. Install a trusted certificate on the server.
- **Azure SQL:** Network security boundaries create a logical network boundary around PaaS resources deployed outside a virtual network. IP firewall rules grant access based on the originating IP address.

#### Constraints and Limitations

- **TLS configuration complexity:** Certificate management, cipher suite selection, and client configuration add operational overhead.
- **Network segmentation cost:** Dedicated network segments and firewalls require infrastructure investment.
- **Cloud shared responsibility:** In cloud databases, the provider manages some boundaries (physical, network) while the customer manages others (IAM, database permissions).

### Annotated Code Examples

#### Example 1: PostgreSQL — Requiring SSL Connections

```sql
-- Step 1: Enable SSL in postgresql.conf
-- ssl = on
-- ssl_cert_file = 'server.crt'
-- ssl_key_file = 'server.key'

-- Step 2: Configure pg_hba.conf to require SSL for remote connections
-- hostssl all all 0.0.0.0/0 scram-sha-256
-- hostnossl all all 0.0.0.0/0 reject

-- Step 3: Verify SSL is active
SHOW ssl;
```

**Expected Output:**

```
 ssl
-----
 on
```

**Why This Works:** `hostssl` requires SSL for matching connections; `hostnossl` rejects non-SSL connections. This ensures all remote traffic is encrypted.

#### Example 2: MySQL — Requiring SSL for a User

```sql
-- Create a user that requires SSL
CREATE USER 'secure_app'@'%'
  IDENTIFIED WITH caching_sha2_password BY 'SecurePass123!'
  REQUIRE SSL;

-- Or require a valid X.509 certificate
ALTER USER 'secure_app'@'%' REQUIRE X509;
```

**Expected Output:**

```
Query OK, 0 rows affected
```

**Why This Works:** `REQUIRE SSL` forces the user to connect over an encrypted TLS connection. `REQUIRE X509` additionally requires a valid client certificate, providing mutual authentication.

### Real-World Cases

- **PCI-DSS:** Requires network segmentation and encryption of cardholder data in transit.
- **Multi-tenant SaaS:** Isolating tenant data in separate databases or schemas.
- **Hybrid cloud:** Using VPN or private links to connect on-premises applications to cloud databases.
- **Development vs. production:** Separate clusters with no network connectivity between them.

### References

- OWASP: Database Security Cheat Sheet (Protecting the Backend Database) — https://cheatsheetseries.owasp.org/cheatsheets/Database_Security_Cheat_Sheet.html
- Azure SQL: Network Security Boundaries — https://learn.microsoft.com/en-us/azure/azure-sql/database/network-security-perimeter
- PostgreSQL: SSL Support — https://www.postgresql.org/docs/current/ssl-tcp.html
- MySQL: Using Encrypted Connections — https://dev.mysql.com/doc/refman/8.0/en/encrypted-connections.html


## Core Concept 5: Database Users

### Definitions

**Core Definition:** A database user is a distinct account within a database that can authenticate to the database and be granted privileges to access objects and perform operations.

**Technical Definition:** Database users are principals that can connect to the database. In SQL Server, a login is a server-level principal, and a user is a database-level principal mapped to a login. In PostgreSQL, `CREATE USER` is equivalent to `CREATE ROLE` with the `LOGIN` attribute. In MySQL, a user is identified by a username and host combination (e.g., `'app_user'@'%'`). In Oracle, a user is a schema—creating a user creates a schema with the same name. Users can be configured with connection limits, password expiration policies, resource limits, and account locking.

**Beginner-Friendly Explanation:** A database user is like an employee account in a company. Each person gets their own account with a username and password. The account can be configured with rules: how many connections they can have at once, when their password expires, how much CPU they can use, and whether the account is locked or active.

### Purposes

- **To provide individual accountability** by giving each person or application a unique account.
- **To enforce password policies** (expiration, reuse, complexity) at the account level.
- **To limit resource consumption** through connection limits and query limits.
- **To enable account lifecycle management** (create, alter, lock, unlock, drop).
- **To support external authentication** by mapping users to external identities.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
CREATE ROLE name [ [ WITH ] option [ ... ] ]
-- Or equivalently:
CREATE USER name [ [ WITH ] option [ ... ] ]

-- Options include:
--   SUPERUSER | NOSUPERUSER
--   CREATEDB | NOCREATEDB
--   CREATEROLE | NOCREATEROLE
--   INHERIT | NOINHERIT
--   LOGIN | NOLOGIN
--   CONNECTION LIMIT connlimit
--   PASSWORD 'password'
--   VALID UNTIL 'timestamp'
```

#### Complete General Syntax (MySQL)

```sql
CREATE USER [IF NOT EXISTS] user [auth_option] [default_role] [require_clause]
    [with_option] [password_option] [lock_option];

-- auth_option:
--   IDENTIFIED BY 'auth_string'
--   IDENTIFIED WITH auth_plugin BY 'auth_string'

-- password_option:
--   PASSWORD EXPIRE [DEFAULT | NEVER | INTERVAL N DAY]
--   PASSWORD HISTORY [DEFAULT | N]
--   FAILED_LOGIN_ATTEMPTS N
--   PASSWORD_LOCK_TIME [N | UNBOUNDED]

-- lock_option:
--   ACCOUNT LOCK | ACCOUNT UNLOCK
```

#### Complete General Syntax (SQL Server)

```sql
-- Server-level login
CREATE LOGIN login_name { WITH PASSWORD = 'password' [ MUST_CHANGE ] }
    [ , CHECK_EXPIRATION = { ON | OFF } ]
    [ , CHECK_POLICY = { ON | OFF } ]
    [ , DEFAULT_DATABASE = database ]
    [ , SID = sid ];

-- Database-level user
CREATE USER user_name FOR LOGIN login_name;
```

#### Complete General Syntax (Oracle)

```sql
CREATE USER user_name
  IDENTIFIED BY password
  [ DEFAULT TABLESPACE tablespace ]
  [ TEMPORARY TABLESPACE tablespace ]
  [ QUOTA { integer [ K | M | G | T ] | UNLIMITED } ON tablespace ]
  [ PROFILE profile ]
  [ PASSWORD EXPIRE ]
  [ ACCOUNT { LOCK | UNLOCK } ]
  [ CONTAINER = { CURRENT | ALL } ];
```

#### Syntax Rules

- **PostgreSQL:** `CREATE USER` assumes `LOGIN` by default; `CREATE ROLE` does not. The `CONNECTION LIMIT` option restricts concurrent connections. `VALID UNTIL` sets a password expiration date.
- **MySQL:** `PASSWORD EXPIRE INTERVAL N DAY` sets password expiration. `FAILED_LOGIN_ATTEMPTS N` and `PASSWORD_LOCK_TIME` implement account locking after failed logins.
- **SQL Server:** `CHECK_EXPIRATION = ON` enforces password expiration. `MUST_CHANGE` requires the user to change the password on first login. After creating a login, you must create a database user to connect to a specific database.
- **Oracle:** You cannot set default roles for a user in the `CREATE USER` statement. When you first create a user, the default role setting for the user is `ALL`, which causes all roles subsequently granted to the user to be default roles.

#### Constraints and Limitations

- **PostgreSQL:** Superuser roles bypass all permission checks and should not be used for routine work.
- **MySQL:** The `mysql_native_password` plugin is deprecated; use `caching_sha2_password`.
- **SQL Server:** SQL Server Authentication requires users to provide a username and password in every connection string; it does not support Kerberos delegation by default.
- **Oracle:** You cannot change an existing common user account to be a local user account, or a local user account to be made into a common user account. You must create a new account.

### Annotated Code Examples

#### Example 1: PostgreSQL — Creating a User with Connection Limit and Password Expiration

```sql
-- Create a user with a connection limit and password expiration
CREATE USER app_user
  WITH LOGIN
       PASSWORD 'SecurePass123!'
       CONNECTION LIMIT 10
       VALID UNTIL '2027-01-01';

-- Verify
SELECT rolname, rolconnlimit, rolvaliduntil
FROM pg_roles
WHERE rolname = 'app_user';
```

**Expected Output:**

```
 rolname  | rolconnlimit |     rolvaliduntil
----------+--------------+---------------------
 app_user |           10 | 2027-01-01 00:00:00
```

**Why This Works:** `CONNECTION LIMIT 10` restricts the user to 10 concurrent connections. `VALID UNTIL` sets a password expiration date of January 1, 2027. The `pg_roles` view confirms the settings.

#### Example 2: MySQL — User with Password Expiration and Locking

```sql
-- Create a user with password expiration and failed login locking
CREATE USER 'app_user'@'%'
  IDENTIFIED WITH caching_sha2_password BY 'SecurePass123!'
  PASSWORD EXPIRE INTERVAL 90 DAY
  FAILED_LOGIN_ATTEMPTS 5
  PASSWORD_LOCK_TIME 3
  MAX_CONNECTIONS_PER_HOUR 1000;

-- Lock the account
ALTER USER 'app_user'@'%' ACCOUNT LOCK;

-- Unlock the account
ALTER USER 'app_user'@'%' ACCOUNT UNLOCK;
```

**Expected Output:**

```
Query OK, 0 rows affected
```

**Why This Works:** `PASSWORD EXPIRE INTERVAL 90 DAY` forces password changes every 90 days. `FAILED_LOGIN_ATTEMPTS 5` and `PASSWORD_LOCK_TIME 3` lock the account for 3 days after 5 failed login attempts. `MAX_CONNECTIONS_PER_HOUR 1000` limits connection frequency.

### Real-World Cases

- **Application accounts:** One account per application service with specific resource limits.
- **Human users:** Individual accounts with password expiration and complexity requirements.
- **Service accounts:** Accounts with `NOLOGIN` for ownership of objects (PostgreSQL roles without login).
- **Temporary accounts:** Accounts with `VALID UNTIL` for contractors or temporary staff.

### References

- PostgreSQL: CREATE ROLE — https://www.postgresql.org/docs/current/sql-createrole.html
- MySQL: CREATE USER — https://dev.mysql.com/doc/refman/8.0/en/create-user.html
- MySQL: ALTER USER — https://dev.mysql.com/doc/refman/8.0/en/alter-user.html
- SQL Server: CREATE LOGIN — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-login-transact-sql
- Oracle: CREATE USER — https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/CREATE-USER.html


## Core Concept 6: Roles

### Definitions

**Core Definition:** A role is a named collection of privileges that can be granted to users (or other roles), providing a logical grouping of permissions for easier management.

**Technical Definition:** Roles are the primary mechanism for implementing role-based access control (RBAC) in SQL databases. A role can be granted privileges on objects, and then granted to users (or other roles). In PostgreSQL, roles subsume the concepts of users and groups; `CREATE USER` is equivalent to `CREATE ROLE` with the `LOGIN` attribute. In SQL Server, server roles (fixed and user-defined) group server-level permissions, while database roles group database-level permissions. In Oracle, roles can be granted to users and can be made default (automatically enabled at login) or non-default. Role inheritance determines whether a user automatically receives the privileges of granted roles.

**Beginner-Friendly Explanation:** A role is like a job title in a company. The "Manager" role might include permissions to approve expenses, view reports, and manage employees. When someone becomes a manager, you give them the "Manager" role, and they automatically get all the associated permissions. If the role's permissions change, everyone with that role gets the update automatically. This is much easier than granting permissions to each person individually.

### Purposes

- **To simplify privilege management** by grouping permissions into logical containers.
- **To implement role-based access control (RBAC)** where permissions are associated with job functions.
- **To support role hierarchies** where roles inherit privileges from other roles.
- **To enable default roles** that are automatically activated at login.
- **To enforce separation of duties** by creating distinct roles for different administrative functions.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
-- Create a role
CREATE ROLE role_name [ [ WITH ] option [ ... ] ];

-- Grant a role to a user (or another role)
GRANT role_name TO user_name [ WITH ADMIN OPTION ];

-- Grant a privilege to a role
GRANT SELECT ON table_name TO role_name;

-- Set default role for a user
ALTER ROLE user_name SET role_name TO DEFAULT;
```

#### Complete General Syntax (SQL Server)

```sql
-- Create a user-defined server role
CREATE SERVER ROLE server_role_name [ AUTHORIZATION owner_name ];

-- Add a login to a server role
ALTER SERVER ROLE server_role_name ADD MEMBER login_name;

-- Create a user-defined database role
CREATE ROLE database_role_name [ AUTHORIZATION owner_name ];

-- Add a user to a database role
ALTER ROLE database_role_name ADD MEMBER user_name;

-- Fixed server roles (built-in)
-- sysadmin, serveradmin, securityadmin, processadmin, dbcreator, diskadmin, bulkadmin, public
```

#### Complete General Syntax (Oracle)

```sql
-- Create a role
CREATE ROLE role_name [ NOT IDENTIFIED | IDENTIFIED BY password ];

-- Grant a privilege to a role
GRANT SELECT ON schema.table TO role_name;

-- Grant a role to a user
GRANT role_name TO user_name;

-- Set default roles for a user
ALTER USER user_name DEFAULT ROLE role_name;
-- Or
ALTER USER user_name DEFAULT ROLE ALL;
-- Or
ALTER USER user_name DEFAULT ROLE ALL EXCEPT role_name;
```

#### Syntax Rules

- **PostgreSQL:** `CREATE ROLE` does not include `LOGIN` by default; `CREATE USER` does. Roles can be granted to other roles, creating a hierarchy. The `INHERIT` attribute (default) determines whether a role automatically inherits the privileges of roles it is a member of.
- **PostgreSQL:** Default roles are a set of predefined roles that provide access to certain, commonly needed, privileged capabilities and information. Administrators can `GRANT` these roles to users and/or other roles in their environment.
- **SQL Server:** Fixed server roles include `sysadmin` (unrestricted access to the entire instance), `dbcreator` (create databases), and `securityadmin` (manage logins and their properties). Members of the `dbcreator` fixed server role can create, alter, drop, and restore any database.
- **Oracle:** You cannot set default roles for a user in the `CREATE USER` statement. When you first create a user, the default role setting is `ALL`, which causes all roles subsequently granted to the user to be default roles. Use `ALTER USER` to change default roles.

#### Constraints and Limitations

- **Role explosion:** Over time, an excessive number of roles can become difficult to manage. Regular role consolidation and review are necessary.
- **Privilege inheritance risk:** If a role is granted `WITH ADMIN OPTION`, the grantee can grant the role to others, potentially escalating privileges.
- **Default role dependency:** Oracle's default `ALL` setting means all granted roles are active at login. Use `ALTER USER ... DEFAULT ROLE` to restrict which roles are active by default.
- **Superuser roles:** PostgreSQL's `SUPERUSER` attribute bypasses all permission checks. It should be granted sparingly.

### Annotated Code Examples

#### Example 1: PostgreSQL — Role Hierarchy and Default Roles

```sql
-- Create a role hierarchy
CREATE ROLE reporting;
CREATE ROLE analyst;
GRANT reporting TO analyst;  -- analyst inherits reporting privileges

-- Grant privileges to the reporting role
GRANT SELECT ON sales.orders TO reporting;
GRANT SELECT ON sales.customers TO reporting;

-- Create a user and grant the analyst role
CREATE USER jdoe LOGIN PASSWORD 'JdoePass123!';
GRANT analyst TO jdoe;

-- Verify: jdoe can read orders because analyst inherits reporting
SELECT * FROM sales.orders LIMIT 1;
```

**Expected Output:**

```
 order_id | customer_id | amount
----------+-------------+--------
        1 |         101 | 150.00
```

**Why This Works:** The `analyst` role inherits the `reporting` role's privileges through `GRANT reporting TO analyst`. When `jdoe` is granted the `analyst` role, they automatically inherit both `analyst` and `reporting` privileges. This hierarchical structure simplifies privilege management.

#### Example 2: SQL Server — Custom Database Role

```sql
-- Create a custom database role
USE CompanyDB;
CREATE ROLE app_readwrite;

-- Grant privileges to the role
GRANT SELECT, INSERT, UPDATE, DELETE ON SCHEMA::app TO app_readwrite;
GRANT EXECUTE ON SCHEMA::app TO app_readwrite;

-- Create a user and add to the role
CREATE USER app_user FOR LOGIN app_login;
ALTER ROLE app_readwrite ADD MEMBER app_user;
```

**Expected Output:**

```
Commands completed successfully.
```

**Why This Works:** The `app_readwrite` role has only the permissions needed for application CRUD operations and stored procedure execution. The `app_user` inherits these permissions through role membership.

### Real-World Cases

- **Application roles:** Separate roles for read-only reporting, read-write application access, and administrative functions.
- **DBA roles:** Distinct roles for backup operators, security administrators, and database administrators.
- **Default roles:** PostgreSQL's `pg_read_all_data`, `pg_write_all_data`, and `pg_monitor` roles for common monitoring and reporting needs.
- **Oracle roles:** `CONNECT`, `RESOURCE`, and `DBA` roles for standard privilege bundles.

### References

- PostgreSQL: CREATE ROLE — https://www.postgresql.org/docs/current/sql-createrole.html
- PostgreSQL: Default Roles — https://www.postgresql.org/docs/current/default-roles.html
- SQL Server: CREATE ROLE (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-role-transact-sql
- SQL Server: Server-Level Roles — https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/server-level-roles
- Oracle: CREATE ROLE — https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/CREATE-ROLE.html
- Oracle: Managing Security for Oracle Database Users (Default Roles) — https://docs.oracle.com/en/database/oracle/oracle-database/26/dbseg/managing-security-for-oracle-database-users.html


## Summary Table: Database Security Fundamentals

| Concept | Key SQL Constructs | Primary Purpose | Key Consideration |
|---------|-------------------|-----------------|-------------------|
| **Authentication** | `pg_hba.conf`, `CREATE USER`, `CREATE LOGIN` | Verify identity | Use SCRAM-SHA-256 or Windows Authentication |
| **Authorization** | `GRANT`, `REVOKE`, `SECURITY DEFINER` | Control access | Prefer `SECURITY INVOKER` unless necessary |
| **Least Privilege** | Named accounts, custom roles | Minimize attack surface | Avoid SYS, SYSTEM, sa, postgres for routine work |
| **Security Boundaries** | `hostssl`, `REQUIRE SSL`, firewall rules | Isolate environments | Encrypt all traffic with TLS |
| **Database Users** | `CREATE USER`, `ALTER USER`, `DROP USER` | Manage accounts | Configure password expiration and connection limits |
| **Roles** | `CREATE ROLE`, `GRANT role TO user` | Group privileges | Use role hierarchies; review default roles |


## Final Notes on Deprecated and Unsafe Features

- **MySQL `mysql_native_password`:** Deprecated and subject to removal in a future version of MySQL. Use `caching_sha2_password` instead.
- **PostgreSQL `md5` authentication:** Weak and legacy. Use `scram-sha-256` instead.
- **SQL Server `sa` account:** The `sa` account is not automatically enabled when switching to mixed mode. To use it, execute `ALTER LOGIN sa WITH PASSWORD = '...' ENABLE` (the `ENABLE` option must be explicitly set).
- **Oracle SYS/SYSTEM accounts:** Oracle recommends using named accounts instead of the generic SYS and SYSTEM accounts. Once Database Vault is enabled, SYS is no longer able to perform certain actions.
- **PostgreSQL `SUPERUSER`:** This attribute bypasses all permission checks and should not be used for routine work. Create non-superuser roles for most tasks.
- **Security definer risk:** `SECURITY DEFINER` functions execute with the owner's privileges. If not carefully written, they can be exploited for privilege escalation. In Oracle 12c+, use `INHERIT [ANY] PRIVILEGES` to prevent lower-privileged users from exploiting invoker rights units.
- **Unencrypted connections:** Most database default configurations start with unencrypted network connections. Configure the database to only allow encrypted connections (TLSv1.2+) and install a trusted digital certificate.
- **Version-specific:** `scram-sha-256` requires PostgreSQL 10+. `caching_sha2_password` is the default in MySQL 8.0+. Oracle Database Vault requires Oracle 10g+. SQL Server 2022 requires `##MS_LoginManager##` fixed server role for `CREATE LOGIN`.