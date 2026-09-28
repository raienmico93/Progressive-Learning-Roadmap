# SQL Data Protection: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL data protection encompasses the policies, technologies, and SQL-based controls designed to safeguard sensitive data throughout its lifecycle—at rest, in transit, and in use—by enforcing encryption, masking, access restrictions, and comprehensive auditing.

**Technical Definition:** Data protection in SQL databases is a multi-layered security discipline that combines encryption at rest (protecting physical storage volumes and data files), encryption in transit (securing network communication), data masking and redaction (obscuring sensitive values from unauthorized users), sensitive-data identification and isolation (classifying PII, PCI, and PHI data), and auditing (creating immutable logs of access and administrative actions). These controls are implemented through a combination of database engine features (Transparent Data Encryption, Dynamic Data Masking, Unified Auditing), SQL statements (GRANT, CREATE VIEW, CREATE FUNCTION), and configuration parameters. The goal is to ensure confidentiality, integrity, and compliance with regulations such as GDPR, HIPAA, and PCI-DSS.

**Beginner-Friendly Explanation:** Data protection is like the security system of a bank vault. Encryption at rest is the solid steel walls—even if someone steals the vault, they cannot read the contents. Encryption in transit is the armored truck—data is safe while moving between locations. Data masking is the blacked-out text on documents—you can see the document exists, but not the sensitive details. Auditing is the security camera—it records who entered, when, and what they did. SQL is the language you use to configure all of these protections.

### Key Characteristics

- **Defense in depth:** Multiple overlapping controls (encryption, masking, access control, auditing) protect against different threat vectors.
- **Regulation-driven:** Compliance requirements (PCI-DSS, HIPAA, GDPR, SOX) mandate specific protections for specific data types.
- **Transparent where possible:** TDE and TLS are designed to be transparent to applications; masking and auditing may require application awareness.
- **Granular:** Protections can be applied at the column, row, table, schema, or database level.
- **Auditable:** Every protection mechanism must itself be auditable to prove compliance.
- **Vendor-varied:** Implementation details differ significantly across PostgreSQL, MySQL, SQL Server, and Oracle.

### Prerequisites

- Understanding of database security fundamentals (authentication, authorization, least privilege).
- Familiarity with encryption concepts (symmetric keys, asymmetric keys, certificates).
- Knowledge of SQL DCL (`GRANT`, `REVOKE`) and DDL (`CREATE`, `ALTER`).
- Awareness of compliance requirements relevant to your industry.

### Related Programming Areas

- Database security and compliance.
- Application security and secure coding.
- Data governance and privacy (GDPR, CCPA).
- Cloud security and shared responsibility models.
- Audit and forensic analysis.

### Core Concepts / Features

1. **Sensitive-Data Handling** (PII/PCI isolation, limited column exposure)
2. **Encryption at Rest** (TDE, file system block encryption)
3. **Encryption in Transit** (TLS/SSL connection configurations)
4. **Data Masking** (Dynamic Data Masking, view overlays, hashing, tokenization)
5. **Auditing** (immutable logs, DDL audit, data read audit)
6. **Access Logging** (server-wide events, connection attempts, source IP tracking)


## Core Concept 1: Sensitive-Data Handling

### Definitions

**Core Definition:** Sensitive-data handling is the practice of identifying, classifying, and isolating Personally Identifiable Information (PII), Payment Card Industry (PCI) data, and Protected Health Information (PHI) to ensure limited column exposure and regulatory compliance.

**Technical Definition:** Sensitive-data handling involves discovering where sensitive data resides, classifying it by type (PII, PCI, PHI), and applying controls to restrict access. Technically, this is achieved through column-level GRANT/REVOKE, view-based access (exposing only non-sensitive columns), and the use of database security products that enforce realm-based access control—a security mechanism that restricts access to data based on real-name, least privilege principles. Sensitive data is data that must be protected against unauthorized access, including personal information, financial data, and health records.

**Beginner-Friendly Explanation:** Sensitive-data handling is like sorting mail into different security levels. Some documents are public (company newsletter), some are confidential (employee salaries), and some are strictly confidential (medical records). You put the strictly confidential documents in a locked safe, and only people with the right clearance get the key. In SQL, you identify which columns contain sensitive data (like credit card numbers or social security numbers) and restrict who can see them.

### Purposes

- **To isolate and identify PII/PCI data** so that appropriate protections can be applied.
- **To ensure limited column exposure** by granting access only to non-sensitive columns.
- **To enforce separation of duties** so that no single user can access all sensitive data.
- **To comply with regulations** such as PCI-DSS (payment card data), HIPAA (health data), and GDPR (personal data).
- **To reduce the blast radius** of a data breach by minimizing the number of users with access to sensitive data.

### Syntax Rules and Structure

#### Complete General Syntax (Column-Level Restriction)

```sql
-- Create a table with sensitive and non-sensitive columns
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT,
    phone TEXT,
    ssn TEXT,              -- Sensitive (PII)
    credit_card TEXT       -- Sensitive (PCI)
);

-- Create a role for customer service representatives
CREATE ROLE csr_role;

-- Grant SELECT only on non-sensitive columns
GRANT SELECT (customer_id, name, email, phone)
    ON customers TO csr_role;

-- Grant UPDATE only on contact information
GRANT UPDATE (email, phone)
    ON customers TO csr_role;
```

#### Complete General Syntax (View-Based Access)

```sql
-- Create a view that exposes only non-sensitive columns
CREATE VIEW customer_public AS
SELECT customer_id, name, email, phone
FROM customers;

-- Grant SELECT on the view
GRANT SELECT ON customer_public TO csr_role;

-- Revoke direct table access
REVOKE SELECT ON customers FROM csr_role;
```

#### Syntax Rules

- **Column-level GRANT:** Only `SELECT`, `INSERT`, `UPDATE`, and `REFERENCES` can be granted at the column level.
- **Views:** A view can expose a subset of columns and rows, acting as a security boundary.
- **Realm-based access control:** Oracle Database Vault realms restrict access to data based on real-name, least privilege principles.
- **Data classification:** Identify sensitive data by scanning column names, data patterns (e.g., 16-digit credit card numbers), and metadata.

#### Constraints and Limitations

- **Column-level grants are not supported by all databases:** SQL Server supports `SELECT`, `REFERENCES`, and `UPDATE` at the column level; MySQL does not.
- **Views can be bypassed:** Users with direct table access can bypass view-based restrictions. Revoke direct table access.
- **Performance:** Column-level grants may add overhead to query optimization.

### Annotated Code Examples

#### Example 1: PostgreSQL — Column-Level Access Restriction

```sql
-- Create a customers table with sensitive data
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT,
    phone TEXT,
    ssn TEXT,
    credit_card TEXT
);

-- Insert sample data
INSERT INTO customers (name, email, phone, ssn, credit_card) VALUES
('Alice Smith', 'alice@example.com', '555-0101', '123-45-6789', '4111-1111-1111-1111'),
('Bob Jones', 'bob@example.com', '555-0102', '987-65-4321', '5500-0000-0000-0004');

-- Create a role for customer service
CREATE ROLE csr_role;

-- Grant SELECT on non-sensitive columns only
GRANT SELECT (customer_id, name, email, phone)
    ON customers TO csr_role;

-- Grant UPDATE on contact information only
GRANT UPDATE (email, phone)
    ON customers TO csr_role;

-- Verify: csr_role cannot see ssn or credit_card
SELECT customer_id, name, email, phone FROM customers;
```

**Expected Output:**

```
 customer_id |    name     |       email       |   phone
-------------+-------------+-------------------+-----------
           1 | Alice Smith | alice@example.com | 555-0101
           2 | Bob Jones   | bob@example.com   | 555-0102
```

**Why This Works:** The `GRANT SELECT (columns)` syntax restricts access to only the listed columns. The `csr_role` cannot select `ssn` or `credit_card`, even though those columns exist in the table. This enforces limited column exposure without requiring a view.

### Real-World Cases

- **Healthcare:** Isolating patient names, dates of birth, and medical record numbers from billing data.
- **E-commerce:** Restricting access to credit card numbers while allowing access to order history.
- **Financial services:** Separating account numbers from transaction details.
- **HR:** Restricting access to salary and social security numbers to authorized payroll staff.

### References

- Oracle: Managing Security for Oracle Database Users — https://docs.oracle.com/en/database/oracle/oracle-database/26/dbseg/managing-security-for-oracle-database-users.html
- Oracle: Database Vault Realms — https://docs.oracle.com/en/database/oracle/oracle-database/19/dvgsg/database-vault-getting-started-guide.pdf
- PostgreSQL: GRANT — https://www.postgresql.org/docs/14/sql-grant.html
- SQL Server: GRANT Object Permissions — https://learn.microsoft.com/en-us/sql/t-sql/statements/grant-object-permissions-transact-sql


## Core Concept 2: Encryption at Rest

### Definitions

**Core Definition:** Encryption at rest is the cryptographic protection of data when it is stored on physical media—data files, backup files, transaction logs, and disk volumes—to prevent unauthorized access if the storage medium is compromised.

**Technical Definition:** Encryption at rest protects data stored in the database by encrypting data files, backup files, and transaction logs. The two primary approaches are Transparent Data Encryption (TDE), which encrypts the database files at the storage level and is transparent to applications, and file system or block-level encryption, which encrypts the entire storage volume. TDE is a technique that encrypts sensitive data in a database before it is written to disk. TDE protects data at rest, including data and log files. With TDE, the data is encrypted before it is written to disk and decrypted when read into memory. TDE performs real-time I/O encryption and decryption of the data and log files. The encryption uses a database encryption key (DEK), which is stored in the database boot record for availability during recovery.

**Beginner-Friendly Explanation:** Encryption at rest is like putting your documents in a locked safe before storing them in a warehouse. Even if someone breaks into the warehouse and steals the safe, they cannot read the documents without the key. In a database, TDE encrypts the files on disk so that if someone steals the physical hard drive, they cannot read the data. Applications do not notice any difference—they read and write data as usual, and the database handles encryption and decryption automatically.

### Purposes

- **To safeguard physical storage volumes** and data files against theft or unauthorized access.
- **To protect backup files** and transaction logs from being read if they are stolen.
- **To comply with regulations** that require encryption of sensitive data at rest (PCI-DSS, HIPAA).
- **To enable secure decommissioning** of storage media by ensuring data is unreadable without the key.

### Syntax Rules and Structure

#### Complete General Syntax (SQL Server — TDE)

```sql
-- Step 1: Create a master key in the master database
USE master;
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'StrongPassword123!';

-- Step 2: Create a server certificate
CREATE CERTIFICATE TDECert
    WITH SUBJECT = 'TDE Certificate for CompanyDB';

-- Step 3: Create a database encryption key in the user database
USE CompanyDB;
CREATE DATABASE ENCRYPTION KEY
    WITH ALGORITHM = AES_256
    ENCRYPTION BY SERVER CERTIFICATE TDECert;

-- Step 4: Enable encryption on the database
ALTER DATABASE CompanyDB SET ENCRYPTION ON;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `CREATE MASTER KEY` | Creates the database master key that protects certificates. |
| `CREATE CERTIFICATE` | Creates a server certificate used to protect the DEK. |
| `CREATE DATABASE ENCRYPTION KEY` | Creates the DEK with AES_256 encryption. |
| `ENCRYPTION BY SERVER CERTIFICATE` | Protects the DEK with the server certificate. |
| `ALTER DATABASE SET ENCRYPTION ON` | Initiates the encryption process. |

#### Complete General Syntax (Oracle — TDE)

```sql
-- Step 1: Configure the wallet (sqlnet.ora)
-- ENCRYPTION_WALLET_LOCATION = (SOURCE = (METHOD = FILE) (METHOD_DATA = (DIRECTORY = /etc/oracle/wallet)))

-- Step 2: Create the master encryption key
ADMINISTER KEY MANAGEMENT CREATE KEYSTORE '/etc/oracle/wallet' IDENTIFIED BY "wallet_password";

-- Step 3: Open the wallet
ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY "wallet_password";

-- Step 4: Create a table with encrypted columns
CREATE TABLE customers (
    customer_id NUMBER PRIMARY KEY,
    name VARCHAR2(100),
    ssn VARCHAR2(11) ENCRYPT,
    credit_card VARCHAR2(19) ENCRYPT
);

-- Or encrypt an existing column
ALTER TABLE customers MODIFY (ssn ENCRYPT);
```

#### Syntax Rules

- **SQL Server TDE:** The database encryption key must be created before the database can be encrypted. The DEK is protected by a server certificate or asymmetric key. TDE is available in SQL Server Enterprise and Developer editions.
- **Oracle TDE:** Column-level encryption uses `ENCRYPT` in the column definition. TDE tablespace encryption uses `ENCRYPT` in the `CREATE TABLESPACE` statement.
- **PostgreSQL:** Does not have built-in TDE; use file system encryption (e.g., LUKS, BitLocker) or `pgcrypto` for column-level encryption.
- **MySQL:** InnoDB tablespace encryption requires a keyring plugin.

#### Constraints and Limitations

- **TDE does not protect data in memory:** Data is decrypted when read into memory; TDE protects data at rest only.
- **Key management is critical:** If the certificate or master key is lost, the data is unrecoverable.
- **Performance overhead:** TDE adds CPU overhead for encryption and decryption; benchmark before production.
- **Edition restrictions:** SQL Server TDE requires Enterprise edition (or Developer for testing).

### Annotated Code Examples

#### Example 1: SQL Server — Enabling TDE

```sql
-- Step 1: Create master key (if not exists)
USE master;
IF NOT EXISTS (SELECT * FROM sys.symmetric_keys WHERE name = '##MS_DatabaseMasterKey##')
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'Str0ngP@ssw0rd!';

-- Step 2: Create server certificate
CREATE CERTIFICATE TDECert
    WITH SUBJECT = 'TDE Certificate for CompanyDB';

-- Step 3: Create database encryption key
USE CompanyDB;
CREATE DATABASE ENCRYPTION KEY
    WITH ALGORITHM = AES_256
    ENCRYPTION BY SERVER CERTIFICATE TDECert;

-- Step 4: Enable encryption
ALTER DATABASE CompanyDB SET ENCRYPTION ON;

-- Step 5: Verify encryption state
SELECT DB_NAME(database_id) AS database_name, encryption_state_desc
FROM sys.dm_database_encryption_keys
WHERE DB_NAME(database_id) = 'CompanyDB';
```

**Expected Output:**

```
database_name | encryption_state_desc
--------------+-----------------------
CompanyDB     | ENCRYPTED
```

**Why This Works:** The master key protects the certificate, the certificate protects the DEK, and the DEK encrypts the database. `SET ENCRYPTION ON` initiates encryption of the data and log files. The `sys.dm_database_encryption_keys` view confirms the encryption state.

#### Example 2: Oracle — Column-Level Encryption

```sql
-- Create a table with encrypted columns
CREATE TABLE customers (
    customer_id NUMBER PRIMARY KEY,
    name VARCHAR2(100),
    ssn VARCHAR2(11) ENCRYPT,
    credit_card VARCHAR2(19) ENCRYPT
);

-- Insert data (transparently encrypted)
INSERT INTO customers VALUES (1, 'Alice Smith', '123-45-6789', '4111-1111-1111-1111');

-- Query (transparently decrypted for authorized users)
SELECT customer_id, name, ssn FROM customers;
```

**Expected Output:**

```
CUSTOMER_ID | NAME        | SSN
------------+-------------+------------
          1 | Alice Smith | 123-45-6789
```

**Why This Works:** The `ENCRYPT` clause encrypts the column data at rest. Authorized users see decrypted values when querying; unauthorized users (without the wallet) see ciphertext. The encryption is transparent to the application.

### Real-World Cases

- **Cloud databases:** Enabling TDE to protect data on shared storage infrastructure.
- **Backup security:** Encrypting backups to prevent data exposure if backup media is lost.
- **Compliance:** Meeting PCI-DSS Requirement 3 (protect stored cardholder data).
- **Laptop/mobile databases:** Encrypting local database files on portable devices.

### References

- Microsoft: Transparent Data Encryption (TDE) — https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/transparent-data-encryption
- Microsoft: CREATE DATABASE ENCRYPTION KEY — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-database-encryption-key-transact-sql
- Oracle: Transparent Data Encryption — https://docs.oracle.com/en/database/oracle/oracle-database/19/asoag/introduction-to-transparent-data-encryption.html
- Oracle: Configuring Transparent Data Encryption — https://docs.oracle.com/en/database/oracle/oracle-database/19/asoag/configuring-transparent-data-encryption.html


## Core Concept 3: Encryption in Transit

### Definitions

**Core Definition:** Encryption in transit is the cryptographic protection of data while it travels over a network between a client and a database server, using TLS/SSL to prevent eavesdropping and man-in-the-middle attacks.

**Technical Definition:** Encryption in transit secures network communication using Transport Layer Security (TLS) or its predecessor, Secure Sockets Layer (SSL). This is distinct from encryption at rest, which protects data on storage media. Database servers enforce TLS through configuration parameters (e.g., `ssl = on` in PostgreSQL, `require_secure_transport = ON` in MySQL, Force Encryption in SQL Server) and client connection strings (`sslmode=require` in PostgreSQL, `Encrypt=true` in SQL Server). Client-side verification flags (e.g., `sslmode=verify-full`) ensure the client validates the server's certificate, preventing man-in-the-middle attacks.

**Beginner-Friendly Explanation:** Encryption in transit is like sending a letter in a sealed, tamper-proof envelope. Anyone can see the envelope moving through the postal system, but they cannot open it or read the contents. In a database context, TLS encrypts the data flowing between your application and the database server, so anyone intercepting the network traffic sees only scrambled data.

### Purposes

- **To force cryptographic network communication** over the wire, preventing eavesdropping.
- **To prevent man-in-the-middle attacks** by validating server certificates.
- **To comply with regulations** that require encryption of data in transit (PCI-DSS Requirement 4).
- **To protect credentials** (usernames, passwords) during authentication.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL — Require SSL)

```sql
-- postgresql.conf
ssl = on
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'
ssl_ca_file = 'root.crt'

-- pg_hba.conf: Require SSL for remote connections
# TYPE  DATABASE  USER  ADDRESS       METHOD
hostssl all       all   0.0.0.0/0     scram-sha-256
hostnossl all     all   0.0.0.0/0     reject
```

#### Complete General Syntax (MySQL — Require SSL)

```sql
-- Create a user that requires SSL
CREATE USER 'secure_app'@'%'
  IDENTIFIED WITH caching_sha2_password BY 'SecurePass123!'
  REQUIRE SSL;

-- Or require a valid X.509 certificate
ALTER USER 'secure_app'@'%' REQUIRE X509;
```

#### Complete General Syntax (SQL Server — Force Encryption)

```text
-- In SQL Server Configuration Manager:
--   1. Expand SQL Server Network Configuration
--   2. Right-click Protocols for <instance>
--   3. Select Properties
--   4. Set Force Encryption to Yes
--   5. Set Certificate to the installed server certificate
```

#### Syntax Rules

- **PostgreSQL:** `hostssl` requires SSL for matching connections; `hostnossl` rejects non-SSL connections. The `ssl` parameter must be `on`. Client `sslmode` options: `disable`, `allow`, `prefer`, `require`, `verify-ca`, `verify-full`.
- **MySQL:** `REQUIRE SSL` enforces encrypted connections. `REQUIRE X509` requires a valid client certificate. `require_secure_transport = ON` enforces TLS globally.
- **SQL Server:** Force Encryption forces all connections to use TLS. A trusted certificate must be installed on the server.
- **Client verification:** `sslmode=verify-full` (PostgreSQL) validates both the certificate chain and the hostname. `Encrypt=true;TrustServerCertificate=false` (SQL Server) performs full validation.

#### Constraints and Limitations

- **Certificate management:** TLS requires certificate provisioning, renewal, and distribution.
- **Performance overhead:** TLS handshake and encryption add CPU overhead; hardware acceleration can mitigate this.
- **Client compatibility:** Older clients may not support modern TLS versions or cipher suites.
- **Self-signed certificates:** Using self-signed certificates with `TrustServerCertificate=true` disables validation, making the connection vulnerable to man-in-the-middle attacks.

### Annotated Code Examples

#### Example 1: PostgreSQL — Requiring SSL Connections

```sql
-- Step 1: Enable SSL in postgresql.conf
ALTER SYSTEM SET ssl = on;
ALTER SYSTEM SET ssl_cert_file = '/etc/ssl/certs/server.crt';
ALTER SYSTEM SET ssl_key_file = '/etc/ssl/private/server.key';
SELECT pg_reload_conf();

-- Step 2: Configure pg_hba.conf to require SSL
-- hostssl all all 0.0.0.0/0 scram-sha-256
-- hostnossl all all 0.0.0.0/0 reject

-- Step 3: Verify SSL is active
SHOW ssl;

-- Step 4: Check current connection
SELECT ssl, version FROM pg_stat_ssl WHERE pid = pg_backend_pid();
```

**Expected Output:**

```
 ssl
-----
 on

 ssl | version
-----+---------
 t   | TLSv1.3
```

**Why This Works:** `ssl = on` enables TLS on the server. `hostssl` in `pg_hba.conf` requires SSL for matching connections, while `hostnossl` rejects non-SSL connections. The `pg_stat_ssl` view confirms the current connection is using TLSv1.3.

#### Example 2: MySQL — Requiring SSL for a User

```sql
-- Create a user that requires SSL
CREATE USER 'app_user'@'%'
  IDENTIFIED WITH caching_sha2_password BY 'SecurePass123!'
  REQUIRE SSL;

-- Verify the user's SSL requirement
SELECT user, host, ssl_type
FROM mysql.user
WHERE user = 'app_user';
```

**Expected Output:**

```
user     | host | ssl_type
---------+------+----------
app_user | %    | ANY
```

**Why This Works:** `REQUIRE SSL` forces the user to connect over an encrypted TLS connection. The `ssl_type` column in `mysql.user` shows `ANY`, indicating that any SSL connection is acceptable (for `REQUIRE X509`, it would show `X509`).

### Real-World Cases

- **Cloud databases:** Enforcing TLS for all connections to databases hosted in the cloud.
- **Remote work:** Protecting database traffic over untrusted networks (public Wi-Fi, home networks).
- **Compliance:** Meeting PCI-DSS Requirement 4 (encrypt transmission of cardholder data across open, public networks).
- **Regulated industries:** Healthcare (HIPAA) and financial services (SOX) require encryption in transit.

### References

- PostgreSQL: SSL Support — https://www.postgresql.org/docs/current/ssl-tcp.html
- MySQL: Using Encrypted Connections — https://dev.mysql.com/doc/refman/8.0/en/encrypted-connections.html
- Microsoft: Encrypting Connections to SQL Server — https://learn.microsoft.com/en-us/sql/database-engine/configure-windows/encrypt-connections-to-sql-server
- OWASP: Transport Layer Protection Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html


## Core Concept 4: Data Masking

### Definitions

**Core Definition:** Data masking is the process of obscuring sensitive data values—through redaction, substitution, shuffling, or tokenization—so that unauthorized users see only masked or fictional values while authorized users see the real data.

**Technical Definition:** Data masking replaces sensitive data with realistic but fictitious data. The two primary approaches are static data masking (SDM), which creates a sanitized copy of the database for non-production environments, and dynamic data masking (DDM), which masks data in real-time as it is queried. SQL Server Dynamic Data Masking limits sensitive data exposure by masking it to non-privileged users. Oracle Data Redaction selectively redacts data returned from a query. Hash-based masking uses cryptographic hash functions (SHA-256) to irreversibly replace values. Tokenization replaces sensitive data with a non-sensitive surrogate (token) that maps back to the original through a secure vault.

**Beginner-Friendly Explanation:** Data masking is like giving someone a redacted document. They can see the document exists and its general structure, but the sensitive parts are blacked out. In a database, masking might show the last four digits of a credit card (`****-****-****-1234`) while hiding the rest. Authorized users see the full number; unauthorized users see only the masked version.

### Purposes

- **To redact sensitive values dynamically** via engine tools without modifying the underlying data.
- **To create safe copies of production data** for development and testing using static masking.
- **To implement conditional view overlays** that expose only non-sensitive columns or masked values.
- **To apply hashing algorithms** for irreversible pseudonymization of sensitive identifiers.
- **To use tokenization services** for PCI-compliant storage of payment card data.

### Syntax Rules and Structure

#### Complete General Syntax (SQL Server — Dynamic Data Masking)

```sql
-- Add masking to an existing column
ALTER TABLE Customers
ALTER COLUMN Email ADD MASKED WITH (FUNCTION = 'email()');

ALTER TABLE Customers
ALTER COLUMN Phone ADD MASKED WITH (FUNCTION = 'partial(0, "XXX-XXX-", 4)');

ALTER TABLE Customers
ALTER COLUMN CreditCard ADD MASKED WITH (FUNCTION = 'default()');

-- Grant UNMASK to authorized users
GRANT UNMASK ON Customers TO [AuthorizedUser];
```

**Component Breakdown:**

| Masking Function | Description | Example |
|------------------|-------------|---------|
| `default()` | Full masking (replaces with `xxxx` for strings, `0` for numbers). | `xxxx` |
| `email()` | Masks email to `aXXX@XXXX.com`. | `aXXX@XXXX.com` |
| `partial(prefix, padding, suffix)` | Shows prefix and suffix, masks the middle. | `XXX-XXX-1234` |
| `random(low, high)` | Replaces number with random value in range. | `random(1, 100)` |

#### Complete General Syntax (Oracle — Data Redaction)

```sql
-- Create a redaction policy
BEGIN
    DBMS_REDACT.ADD_POLICY(
        object_schema    => 'HR',
        object_name      => 'EMPLOYEES',
        column_name      => 'SALARY',
        policy_name      => 'redact_salary',
        function_type    => DBMS_REDACT.PARTIAL,
        function_parameters => '9,1,4',
        expression       => 'SYS_CONTEXT(''USERENV'',''SESSION_USER'') != ''HR_ADMIN'''
    );
END;
```

#### Complete General Syntax (PostgreSQL — View-Based Masking)

```sql
-- Create a view with masked columns
CREATE VIEW customers_masked AS
SELECT
    customer_id,
    name,
    regexp_replace(email, '^(.).*@', '\1***@') AS email_masked,
    regexp_replace(phone, '^.*(\d{4})$', 'XXX-XXX-\1') AS phone_masked,
    regexp_replace(credit_card, '^\d{12}', '************') AS card_masked
FROM customers;
```

#### Syntax Rules

- **SQL Server DDM:** Masking is applied at the column level. Users with `UNMASK` permission see the real data. `SELECT INTO` and `INSERT INTO ... SELECT` copy masked data, not real data.
- **Oracle Data Redaction:** Policies can be expression-based (e.g., redact for all users except HR_ADMIN). Function types include `FULL`, `PARTIAL`, `RANDOM`, `REGEXP`, `NULL`.
- **Hashing:** Use `SHA2()` or `MD5()` for irreversible pseudonymization. Hash values cannot be reversed, but can be matched for equality.
- **Tokenization:** Replace sensitive data with a token; store the mapping in a secure vault. PCI-DSS compliant.

#### Constraints and Limitations

- **DDM is not encryption:** Masked data is still stored in plaintext on disk; DDM only controls what is returned to non-privileged users.
- **DDM bypass:** Users with `UNMASK` permission can see the real data. Users with `SELECT` and `INSERT` can infer data through side channels.
- **Oracle Data Redaction:** Redaction applies to queries, but not to `WHERE` clause predicates in all cases.
- **Hashing:** Not reversible; cannot recover original values. Use for pseudonymization, not for reversible masking.
- **Performance:** Redaction policies add overhead to query execution.

### Annotated Code Examples

#### Example 1: SQL Server — Dynamic Data Masking

```sql
-- Create a table with sensitive data
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    Name NVARCHAR(100),
    Email NVARCHAR(100) MASKED WITH (FUNCTION = 'email()'),
    Phone NVARCHAR(20) MASKED WITH (FUNCTION = 'partial(0, "XXX-XXX-", 4)'),
    CreditCard NVARCHAR(19) MASKED WITH (FUNCTION = 'default()')
);

-- Insert data
INSERT INTO Customers VALUES (1, 'Alice', 'alice@example.com', '555-0101', '4111-1111-1111-1111');

-- Query as non-privileged user (masked)
SELECT * FROM Customers;
```

**Expected Output (non-privileged user):**

```
CustomerID | Name  | Email           | Phone        | CreditCard
-----------+-------+-----------------+--------------+------------
1          | Alice | aXXX@XXXX.com   | XXX-XXX-0101 | xxxx
```

**Expected Output (privileged user with UNMASK):**

```
CustomerID | Name  | Email             | Phone      | CreditCard
-----------+-------+-------------------+------------+---------------------
1          | Alice | alice@example.com | 555-0101   | 4111-1111-1111-1111
```

**Why This Works:** The `MASKED WITH` clause defines the masking function for each column. Non-privileged users see masked values; users with `UNMASK` permission see the real data.

#### Example 2: PostgreSQL — View-Based Masking with Regex

```sql
-- Create a customers table
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name TEXT,
    email TEXT,
    phone TEXT,
    credit_card TEXT
);

INSERT INTO customers (name, email, phone, credit_card) VALUES
('Alice', 'alice@example.com', '555-0101', '4111111111111111');

-- Create a masked view
CREATE VIEW customers_masked AS
SELECT
    customer_id,
    name,
    regexp_replace(email, '^(.).*@', '\1***@') AS email_masked,
    regexp_replace(phone, '^.*(\d{4})$', 'XXX-XXX-\1') AS phone_masked,
    regexp_replace(credit_card, '^\d{12}', '************') AS card_masked
FROM customers;

-- Query the masked view
SELECT * FROM customers_masked;
```

**Expected Output:**

```
customer_id | name  | email_masked    | phone_masked | card_masked
------------+-------+-----------------+--------------+-------------
1           | Alice | a***@example.com| XXX-XXX-0101 | ************1111
```

**Why This Works:** `regexp_replace` applies regular expression patterns to mask the email, phone, and credit card columns. The view exposes only masked values, protecting sensitive data from users who lack access to the base table.

### Real-World Cases

- **Development and testing:** Creating masked copies of production data for non-production environments.
- **Customer service:** Showing masked credit card numbers (`****-****-****-1234`) to support agents.
- **Analytics:** Allowing analysts to query masked datasets without exposing PII.
- **Compliance:** Meeting PCI-DSS requirements for protecting cardholder data in non-production environments.

### References

- Microsoft: Dynamic Data Masking — https://learn.microsoft.com/en-us/sql/relational-databases/security/dynamic-data-masking
- Microsoft: ALTER TABLE (ADD MASKED) — https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-table-transact-sql
- Oracle: Data Redaction — https://docs.oracle.com/en/database/oracle/oracle-database/19/asoag/introduction-to-oracle-data-redaction.html
- Oracle: DBMS_REDACT — https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/DBMS_REDACT.html
- PostgreSQL: String Functions (regexp_replace) — https://www.postgresql.org/docs/current/functions-string.html


## Core Concept 5: Auditing

### Definitions

**Core Definition:** Auditing is the systematic recording of database activities—administrative commands, schema changes, and data access—into immutable, non-repudiable logs for compliance, security monitoring, and forensic analysis.

**Technical Definition:** Auditing tracks database events such as logins, DDL changes, DML operations, and privilege changes. Oracle Unified Auditing writes audit records to internal tables that administrators can query using standard SQL. SQL Server Audit uses Extended Events to create server-level and database-level audit specifications. PostgreSQL uses `pgaudit` extension for session and object auditing. Audit records should be immutable and non-repudiable, meaning they cannot be modified or deleted by the users being audited. The primary purpose of auditing is to detect and deter unauthorized access, enforce accountability, and provide evidence for compliance audits.

**Beginner-Friendly Explanation:** Auditing is like a security camera system in a building. It records who entered, when, and what they did. If something goes wrong, you can review the footage to find out what happened. In a database, auditing records every login, every table access, every schema change, and every administrative command. The logs are immutable—even administrators cannot delete them—so they provide a trustworthy record of activity.

### Purposes

- **To create immutable, non-repudiable logs** of administrative commands and schema alterations.
- **To track explicit target data read accesses** for compliance (e.g., "who viewed patient records?").
- **To detect and deter unauthorized access** by creating accountability.
- **To provide evidence for compliance audits** (PCI-DSS, HIPAA, SOX, GDPR).
- **To support forensic analysis** after a security incident.

### Syntax Rules and Structure

#### Complete General Syntax (Oracle — Unified Auditing)

```sql
-- Create an audit policy
CREATE AUDIT POLICY audit_ddl
    ACTIONS CREATE TABLE, ALTER TABLE, DROP TABLE;

-- Enable the policy
AUDIT POLICY audit_ddl;

-- Create an audit policy for data access
CREATE AUDIT POLICY audit_select
    ACTIONS SELECT ON hr.employees;

AUDIT POLICY audit_select;

-- Query audit records
SELECT event_timestamp, dbusername, action_name, object_name
FROM unified_audit_trail
WHERE action_name = 'SELECT'
ORDER BY event_timestamp DESC;
```

#### Complete General Syntax (SQL Server — SQL Server Audit)

```sql
-- Step 1: Create a server audit
CREATE SERVER AUDIT Audit_Login
TO FILE (FILEPATH = 'C:\Audit\', MAXSIZE = 100 MB);

-- Step 2: Enable the audit
ALTER SERVER AUDIT Audit_Login WITH (STATE = ON);

-- Step 3: Create a server audit specification
CREATE SERVER AUDIT SPECIFICATION Audit_Login_Spec
FOR SERVER AUDIT Audit_Login
ADD (FAILED_LOGIN_GROUP);

ALTER SERVER AUDIT SPECIFICATION Audit_Login_Spec WITH (STATE = ON);

-- Step 4: Create a database audit specification
CREATE DATABASE AUDIT SPECIFICATION Audit_DB_Spec
FOR SERVER AUDIT Audit_Login
ADD (SELECT ON dbo.Employees BY public);

ALTER DATABASE AUDIT SPECIFICATION Audit_DB_Spec WITH (STATE = ON);
```

#### Complete General Syntax (PostgreSQL — pgaudit)

```sql
-- Install pgaudit extension
CREATE EXTENSION pgaudit;

-- Configure pgaudit in postgresql.conf
-- pgaudit.log = 'ddl, role, write'
-- pgaudit.log_catalog = on
-- pgaudit.log_relation = on

-- Query audit logs (when using CSV log format)
SELECT * FROM pg_stat_activity WHERE query LIKE '%CREATE%';
```

#### Syntax Rules

- **Oracle Unified Auditing:** Replaces traditional auditing; writes to `unified_audit_trail` in the `AUDSYS` schema. Policies can be created for specific actions, objects, or users.
- **SQL Server Audit:** Uses Extended Events; audit logs are written to files or Windows Security logs. Server audits capture server-level events; database audit specifications capture database-level events.
- **PostgreSQL pgaudit:** A contrib extension that provides detailed session and object audit logging. Requires `shared_preload_libraries = 'pgaudit'`.
- **Audit records must be immutable:** Store audit logs on write-once media or in a separate, access-controlled schema.

#### Constraints and Limitations

- **Performance overhead:** Auditing all events can significantly impact performance. Audit selectively.
- **Storage requirements:** Audit logs can grow quickly; plan for retention and archival.
- **Oracle Unified Auditing:** The traditional audit trail is deprecated; migrate to unified auditing.
- **SQL Server Audit:** Requires sysadmin or ALTER ANY SERVER AUDIT permission to create audits.
- **PostgreSQL pgaudit:** Requires a restart to load the extension into shared memory.

### Annotated Code Examples

#### Example 1: Oracle — Unified Auditing for Data Access

```sql
-- Create an audit policy for SELECT on the employees table
CREATE AUDIT POLICY audit_emp_select
    ACTIONS SELECT ON hr.employees;

-- Enable the policy
AUDIT POLICY audit_emp_select;

-- Query audit records
SELECT event_timestamp, dbusername, action_name, object_name, sql_text
FROM unified_audit_trail
WHERE action_name = 'SELECT'
  AND object_name = 'EMPLOYEES'
ORDER BY event_timestamp DESC;
```

**Expected Output (partial):**

```
EVENT_TIMESTAMP      | DBUSERNAME | ACTION_NAME | OBJECT_NAME | SQL_TEXT
---------------------+------------+-------------+-------------+------------------
2026-09-28 10:30:00  | HR_ADMIN   | SELECT      | EMPLOYEES   | SELECT * FROM employees
2026-09-28 10:25:00  | APP_USER   | SELECT      | EMPLOYEES   | SELECT name FROM employees
```

**Why This Works:** The `CREATE AUDIT POLICY` statement defines the actions and objects to audit. `AUDIT POLICY` enables the policy. All `SELECT` statements on `hr.employees` are recorded in `unified_audit_trail`, including the user, timestamp, and SQL text.

#### Example 2: SQL Server — Auditing Failed Logins

```sql
-- Create a server audit
CREATE SERVER AUDIT Audit_FailedLogins
TO FILE (FILEPATH = 'C:\Audit\', MAXSIZE = 100 MB);

ALTER SERVER AUDIT Audit_FailedLogins WITH (STATE = ON);

-- Create a server audit specification for failed logins
CREATE SERVER AUDIT SPECIFICATION Audit_FailedLogins_Spec
FOR SERVER AUDIT Audit_FailedLogins
ADD (FAILED_LOGIN_GROUP);

ALTER SERVER AUDIT SPECIFICATION Audit_FailedLogins_Spec WITH (STATE = ON);

-- Query audit logs using sys.fn_get_audit_file
SELECT event_time, action_id, server_principal_name, client_ip
FROM sys.fn_get_audit_file('C:\Audit\*.sqlaudit', DEFAULT, DEFAULT)
WHERE action_id = 'LGIF'  -- Failed login
ORDER BY event_time DESC;
```

**Expected Output (partial):**

```
event_time           | action_id | server_principal_name | client_ip
---------------------+-----------+-----------------------+------------
2026-09-28 10:30:00  | LGIF      | sa                    | 192.168.1.100
2026-09-28 10:29:55  | LGIF      | sa                    | 192.168.1.100
```

**Why This Works:** The `FAILED_LOGIN_GROUP` captures failed login attempts. The audit log records the event time, action ID, principal name, and client IP address. This information is critical for detecting brute-force attacks.

### Real-World Cases

- **PCI-DSS:** Requires auditing of all access to cardholder data (Requirement 10).
- **HIPAA:** Requires audit controls that record and examine access to ePHI.
- **SOX:** Requires auditing of financial data access and changes.
- **GDPR:** Requires logging of personal data processing activities.
- **Insider threat detection:** Auditing data access to detect unauthorized viewing of sensitive records.

### References

- Oracle: Introduction to Auditing — https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/introduction-to-auditing.html
- Oracle: Unified Auditing — https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/part_6.html
- Microsoft: SQL Server Audit — https://learn.microsoft.com/en-us/sql/relational-databases/security/auditing/sql-server-audit-database-engine
- Microsoft: CREATE SERVER AUDIT — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-server-audit-transact-sql
- PostgreSQL: pgaudit — https://www.pgaudit.org/


## Core Concept 6: Access Logging

### Definitions

**Core Definition:** Access logging is the capture of server-wide operational events—connection attempts, failed logins, disconnections, and session activity—including the source IP addresses of clients, for security monitoring and forensic analysis.

**Technical Definition:** Access logging records connection-level events at the database server. In SQL Server, the audit action group `FAILED_LOGIN_GROUP` captures failed login attempts, while `SUCCESSFUL_LOGIN_GROUP` captures successful connections. Oracle Unified Auditing captures `LOGON` and `LOGOFF` actions. PostgreSQL logs connection attempts in the server log (controlled by `log_connections` and `log_disconnections`). MySQL logs connection attempts in the error log. Access logs include the client IP address, username, timestamp, and the result of the connection attempt (success or failure).

**Beginner-Friendly Explanation:** Access logging is like the sign-in sheet at a building's front desk. Every time someone enters or leaves, the security guard writes down their name, the time, and which door they used. If there is a security incident, you can review the sign-in sheet to see who was in the building. In a database, access logging records every connection attempt, including the IP address of the client and whether the attempt succeeded or failed.

### Purposes

- **To capture server-wide operational events** such as startups, shutdowns, and configuration changes.
- **To track connection attempts** (successful and failed) and failed login vectors.
- **To record transaction source IP addresses** for forensic analysis.
- **To detect brute-force attacks** by monitoring repeated failed login attempts.
- **To provide evidence of unauthorized access attempts** for compliance audits.

### Syntax Rules and Structure

#### Complete General Syntax (SQL Server — Login Auditing)

```sql
-- Enable login auditing at the server level
-- In SSMS: Server Properties > Security > Login Auditing
-- Or via T-SQL:
EXEC xp_instance_regwrite
    N'HKEY_LOCAL_MACHINE',
    N'Software\Microsoft\MSSQLServer\MSSQLServer',
    N'AuditLevel',
    REG_DWORD,
    2;  -- 0 = None, 1 = Successful only, 2 = Failed only, 3 = Both

-- Or use SQL Server Audit (recommended)
CREATE SERVER AUDIT Audit_Logins
TO FILE (FILEPATH = 'C:\Audit\', MAXSIZE = 100 MB);

ALTER SERVER AUDIT Audit_Logins WITH (STATE = ON);

CREATE SERVER AUDIT SPECIFICATION Audit_Logins_Spec
FOR SERVER AUDIT Audit_Logins
ADD (SUCCESSFUL_LOGIN_GROUP),
ADD (FAILED_LOGIN_GROUP);
```

#### Complete General Syntax (Oracle — Unified Auditing for Logon)

```sql
-- Create an audit policy for logon events
CREATE AUDIT POLICY audit_logon
    ACTIONS LOGON, LOGOFF;

AUDIT POLICY audit_logon;

-- Query logon events
SELECT event_timestamp, dbusername, action_name, userhost, return_code
FROM unified_audit_trail
WHERE action_name IN ('LOGON', 'LOGOFF')
ORDER BY event_timestamp DESC;
```

#### Complete General Syntax (PostgreSQL — Connection Logging)

```sql
-- Enable connection logging in postgresql.conf
-- log_connections = on
-- log_disconnections = on
-- log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '

-- Or set dynamically (requires superuser)
ALTER SYSTEM SET log_connections = on;
ALTER SYSTEM SET log_disconnections = on;
SELECT pg_reload_conf();

-- Query connection logs
-- The logs are written to the server log file (e.g., /var/log/postgresql/)
```

#### Syntax Rules

- **SQL Server:** Login auditing levels: `0` (none), `1` (successful only), `2` (failed only), `3` (both). `FAILED_LOGIN_GROUP` captures failed logins; `SUCCESSFUL_LOGIN_GROUP` captures successful logins.
- **Oracle:** `LOGON` and `LOGOFF` actions capture connection and disconnection events. `userhost` contains the client host information.
- **PostgreSQL:** `log_connections` logs each connection attempt; `log_disconnections` logs each disconnection. The `client=%h` prefix includes the client IP address.
- **MySQL:** Connection attempts are logged in the error log when `log_error_verbosity` is set appropriately.

#### Constraints and Limitations

- **Log volume:** Connection logging can generate large volumes of log data, especially in high-traffic environments.
- **Sensitive information:** Logs may contain usernames, IP addresses, and other sensitive information; protect log files with appropriate permissions.
- **Performance:** Excessive logging can impact performance; balance logging detail with performance requirements.
- **Retention:** Access logs should be retained according to compliance requirements (e.g., PCI-DSS requires 12 months of audit logs).

### Annotated Code Examples

#### Example 1: SQL Server — Auditing Failed Logins

```sql
-- Create a server audit for failed logins
CREATE SERVER AUDIT Audit_FailedLogins
TO FILE (FILEPATH = 'C:\Audit\', MAXSIZE = 100 MB);

ALTER SERVER AUDIT Audit_FailedLogins WITH (STATE = ON);

CREATE SERVER AUDIT SPECIFICATION Audit_FailedLogins_Spec
FOR SERVER AUDIT Audit_FailedLogins
ADD (FAILED_LOGIN_GROUP);

ALTER SERVER AUDIT SPECIFICATION Audit_FailedLogins_Spec WITH (STATE = ON);

-- Query audit logs
SELECT event_time, action_id, server_principal_name, client_ip
FROM sys.fn_get_audit_file('C:\Audit\*.sqlaudit', DEFAULT, DEFAULT)
WHERE action_id = 'LGIF'
ORDER BY event_time DESC;
```

**Expected Output:**

```
event_time           | action_id | server_principal_name | client_ip
---------------------+-----------+-----------------------+------------
2026-09-28 10:30:00  | LGIF      | sa                    | 192.168.1.100
2026-09-28 10:29:55  | LGIF      | sa                    | 192.168.1.100
```

**Why This Works:** The `FAILED_LOGIN_GROUP` captures failed login attempts. The audit log records the event time, action ID (`LGIF` for login failed), principal name, and client IP address. This information is essential for detecting and responding to brute-force attacks.

#### Example 2: Oracle — Auditing Logon Events

```sql
-- Create an audit policy for logon events
CREATE AUDIT POLICY audit_logon
    ACTIONS LOGON, LOGOFF;

AUDIT POLICY audit_logon;

-- Query logon events
SELECT event_timestamp, dbusername, action_name, userhost, return_code
FROM unified_audit_trail
WHERE action_name = 'LOGON'
ORDER BY event_timestamp DESC;
```

**Expected Output:**

```
EVENT_TIMESTAMP      | DBUSERNAME | ACTION_NAME | USERHOST        | RETURN_CODE
---------------------+------------+-------------+-----------------+------------
2026-09-28 10:30:00  | HR_ADMIN   | LOGON       | 192.168.1.100   | 0
2026-09-28 10:25:00  | APP_USER   | LOGON       | 192.168.1.101   | 0
2026-09-28 10:20:00  | UNKNOWN    | LOGON       | 192.168.1.102   | 1017
```

**Why This Works:** The `CREATE AUDIT POLICY` statement defines the logon and logoff actions to audit. All connection attempts are recorded in `unified_audit_trail`, including the username, client host, and return code. `RETURN_CODE = 1017` indicates a failed login (invalid username/password).

### Real-World Cases

- **Brute-force detection:** Monitoring failed login attempts to detect password-guessing attacks.
- **Forensic analysis:** Using access logs to trace unauthorized access after a security incident.
- **Compliance:** Meeting PCI-DSS Requirement 10 (track and monitor all access to network resources and cardholder data).
- **Insider threat detection:** Monitoring after-hours access and unusual connection patterns.

### References

- Microsoft: SQL Server Audit Action Groups — https://learn.microsoft.com/en-us/sql/relational-databases/security/auditing/sql-server-audit-action-groups-and-actions
- Oracle: Unified Auditing — https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/part_6.html
- PostgreSQL: Logging Configuration — https://www.postgresql.org/docs/current/runtime-config-logging.html
- MySQL: The Error Log — https://dev.mysql.com/doc/refman/8.0/en/error-log.html


## Summary Table: SQL Data Protection Techniques

| Technique | Primary Mechanism | Protects Against | Key Limitation |
|-----------|------------------|------------------|----------------|
| **Sensitive-Data Handling** | Column-level GRANT, views, realms | Unauthorized column access | Requires discovery and classification |
| **Encryption at Rest** | TDE, file system encryption | Physical media theft | Key management critical; CPU overhead |
| **Encryption in Transit** | TLS/SSL, certificate validation | Network eavesdropping | Certificate provisioning overhead |
| **Data Masking** | DDM, redaction, views, hashing | Unauthorized data viewing | Not encryption; masked data still on disk |
| **Auditing** | Unified auditing, SQL Server Audit, pgaudit | Unauthorized actions | Performance and storage overhead |
| **Access Logging** | Login auditing, connection logging | Brute-force, unauthorized access | Log volume; sensitive data in logs |


## Final Notes on Deprecated and Unsafe Features

- **Unencrypted connections:** Most database default configurations start with unencrypted network connections. Configure the database to only allow encrypted connections (TLSv1.2+) and install a trusted digital certificate.
- **Self-signed certificates with disabled validation:** Using `TrustServerCertificate=true` (SQL Server) or `sslmode=require` without `verify-full` (PostgreSQL) disables certificate validation, making the connection vulnerable to man-in-the-middle attacks.
- **Dynamic Data Masking is not encryption:** Masked data is still stored in plaintext on disk. DDM only controls what is returned to non-privileged users. A user with direct file access can read the unmasked data.
- **Oracle traditional auditing:** Deprecated in favor of Unified Auditing. Migrate to unified auditing for better performance and more comprehensive coverage.
- **Audit log tampering:** Audit logs must be immutable and non-repudiable. Store them on write-once media or in a separate, access-controlled schema to prevent tampering.
- **PCI-DSS compliance:** Requires encryption of cardholder data at rest and in transit, masking of PAN when displayed, and comprehensive auditing of all access to cardholder data.
- **Version-specific:** SQL Server TDE requires Enterprise edition. Oracle TDE requires the Advanced Security Option. MySQL InnoDB tablespace encryption requires the keyring plugin. PostgreSQL does not have built-in TDE; use file system encryption or `pgcrypto`.