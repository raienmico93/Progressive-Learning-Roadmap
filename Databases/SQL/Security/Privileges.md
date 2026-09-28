# SQL Privileges: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL privileges are the permissions granted to database users or roles that determine what operations they can perform on database objects—such as reading, writing, or modifying data and structures.

**Technical Definition:** Privileges in SQL databases are the atomic units of authorization, implemented through Data Control Language (DCL) statements—primarily `GRANT` and `REVOKE`. A privilege grants a specific action (e.g., `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `EXECUTE`, `REFERENCES`, `TRIGGER`, `CREATE`, `CONNECT`, `USAGE`) on a specific securable (table, view, column, sequence, schema, database, function, procedure). Privileges can be granted directly to users or roles, and can be propagated using the `WITH GRANT OPTION` clause. The revocation of privileges is governed by cascade semantics (`CASCADE` vs. `RESTRICT`), which determine whether dependent grants are also revoked.

**Beginner-Friendly Explanation:** Privileges are like keys to different rooms in a building. A `SELECT` key lets you look inside a room. An `INSERT` key lets you put things in the room. A `DELETE` key lets you remove things. You can give someone a key (GRANT) or take it away (REVOKE). You can even give someone the ability to make copies of their key for others (WITH GRANT OPTION). Managing privileges is how you control who can do what in your database.

### Key Characteristics

- **Declarative:** Privileges are managed through SQL statements (`GRANT`, `REVOKE`), not application code.
- **Hierarchical:** Privileges granted at higher levels (database, schema) cascade to lower levels (tables, columns).
- **Cumulative:** A user's effective privileges are the union of all privileges granted directly and through roles.
- **Revocable:** Privileges can be revoked with control over cascading effects.
- **Auditable:** Privilege assignments can be inspected using system catalogs (`information_schema`, `pg_roles`, `SHOW GRANTS`).
- **Vendor-varied:** Syntax and available privileges differ across PostgreSQL, MySQL, SQL Server, and Oracle.

### Prerequisites

- Basic understanding of SQL DDL (`CREATE`, `ALTER`, `DROP`) and DML (`SELECT`, `INSERT`, `UPDATE`, `DELETE`).
- Familiarity with database users and roles.
- Knowledge of schemas and object hierarchy.
- Awareness of the principle of least privilege.

### Related Programming Areas

- Database security and access control.
- Role-based access control (RBAC) design.
- Compliance and auditing (SOX, HIPAA, PCI-DSS).
- Multi-tenant application design.
- Database administration (DBA).

### Core Concepts / Features

1. **GRANT** (assigning privileges with `WITH GRANT OPTION`)
2. **REVOKE** (stripping privileges with `CASCADE` vs. `RESTRICT`)
3. **Object Privileges** (table, view, column, sequence)
4. **Schema Privileges** (`USAGE`, `CREATE`, `ALTER DEFAULT PRIVILEGES`)
5. **Database Privileges** (`CONNECT`, `CREATE`, `TEMPORARY`)
6. **Role-Based Access Control** (grouping grants into roles)


## Core Concept 1: GRANT

### Definitions

**Core Definition:** `GRANT` is the SQL statement that assigns specific privileges on database objects to users or roles, optionally allowing the grantee to grant those privileges to others.

**Technical Definition:** The `GRANT` command has two basic variants: one that grants privileges on a database object (table, column, view, foreign table, sequence, database, foreign-data wrapper, foreign server, function, procedure, procedural language, schema, or tablespace), and one that grants membership in a role. The `WITH GRANT OPTION` clause allows the grantee to grant the specified permission to other principals. For databases, `GRANT` requires the grantor to have the privilege themselves and the ability to grant it. The `GRANT` statement enables system administrators to grant privileges and roles, which can be granted to user accounts and roles.

**Beginner-Friendly Explanation:** `GRANT` is the command you use to give someone permission to do something. For example, "I grant you permission to read this table." You can also say "I grant you permission to read this table, and you can give that permission to others" by adding `WITH GRANT OPTION`.

### Purposes

- **To assign explicit administrative or operational access** (SELECT, INSERT, UPDATE, DELETE, EXECUTE, REFERENCES) on specific objects to users or roles.
- **To delegate grant authority** through `WITH GRANT OPTION`, allowing trusted users to manage permissions on your behalf.
- **To implement role-based access control** by granting privileges to roles rather than individual users.
- **To enforce least privilege** by granting only the specific permissions required for a user's job function.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
GRANT { { SELECT | INSERT | UPDATE | DELETE | TRUNCATE | REFERENCES | TRIGGER }
    [, ...] | ALL [ PRIVILEGES ] }
    ON { [ TABLE ] table_name [, ...]
       | ALL TABLES IN SCHEMA schema_name [, ...] }
    TO role_specification [, ...]
    [ WITH GRANT OPTION ]
    [ GRANTED BY role_specification ]

GRANT { { SELECT | INSERT | UPDATE | REFERENCES } ( column_name [, ...] )
    [, ...] | ALL [ PRIVILEGES ] ( column_name [, ...] ) }
    ON [ TABLE ] table_name [, ...]
    TO role_specification [, ...]
    [ WITH GRANT OPTION ]
    [ GRANTED BY role_specification ]

GRANT { { USAGE | SELECT | UPDATE } [, ...] | ALL [ PRIVILEGES ] }
    ON { SEQUENCE sequence_name [, ...]
       | ALL SEQUENCES IN SCHEMA schema_name [, ...] }
    TO role_specification [, ...]
    [ WITH GRANT OPTION ]

GRANT { { CREATE | CONNECT | TEMPORARY | TEMP } [, ...] | ALL [ PRIVILEGES ] }
    ON DATABASE database_name [, ...]
    TO role_specification [, ...]
    [ WITH GRANT OPTION ]

GRANT { { CREATE | USAGE } [, ...] | ALL [ PRIVILEGES ] }
    ON SCHEMA schema_name [, ...]
    TO role_specification [, ...]
    [ WITH GRANT OPTION ]

GRANT { EXECUTE | ALL [ PRIVILEGES ] }
    ON { { FUNCTION | PROCEDURE | ROUTINE } routine_name [ ( [ [ argmode ] [ arg_name ] arg_type [, ...] ] ) ] [, ...]
       | ALL { FUNCTIONS | PROCEDURES | ROUTINES } IN SCHEMA schema_name [, ...] }
    TO role_specification [, ...]
    [ WITH GRANT OPTION ]

GRANT role_name [, ...]
    TO role_specification [, ...]
    [ WITH ADMIN OPTION ]
    [ GRANTED BY role_specification ]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `SELECT`, `INSERT`, `UPDATE`, `DELETE` | Table/view data manipulation privileges. |
| `TRUNCATE` | Privilege to truncate a table. |
| `REFERENCES` | Privilege to create foreign key constraints referencing the table. |
| `TRIGGER` | Privilege to create triggers on the table. |
| `EXECUTE` | Privilege to execute functions and procedures. |
| `USAGE` | Privilege to use a schema, sequence, domain, or other object without specific rights. |
| `CREATE` | Privilege to create objects within a schema or database. |
| `CONNECT` | Privilege to connect to a database. |
| `TEMPORARY` / `TEMP` | Privilege to create temporary tables in a database. |
| `WITH GRANT OPTION` | Allows the grantee to grant the same privilege to others. |
| `WITH ADMIN OPTION` | Allows the grantee to grant role membership to others. |
| `GRANTED BY` | Specifies the role that grants the privilege. |

#### Complete General Syntax (SQL Server)

```sql
GRANT { ALL [ PRIVILEGES ] }
    | permission [ ( column [ , ...n ] ) ] [ , ...n ]
    [ ON [ class :: ] securable ]
    TO principal [ , ...n ]
    [ WITH GRANT OPTION ]
    [ AS principal ]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `permission` | The specific permission to grant (e.g., `SELECT`, `INSERT`, `EXECUTE`). |
| `column` | For column-level grants, the name of the column. Parentheses are required. |
| `class` | The class of the securable (e.g., `OBJECT`, `SCHEMA`, `DATABASE`). The scope qualifier `::` is required. |
| `securable` | The object on which the permission is granted. |
| `TO principal` | The user, login, or group receiving the permission. |
| `WITH GRANT OPTION` | Allows the grantee to grant the permission to other principals. |
| `AS principal` | Specifies the principal recorded as the grantor. |

#### Syntax Rules

- **PostgreSQL:** `GRANT` on a table requires ownership of the table, or the `GRANT` privilege on the object. The `GRANT` command has two basic variants: one for object privileges and one for role membership. The `ALL` keyword grants all privileges available on the object type.
- **SQL Server:** The general concept is `GRANT <some permission> ON <some object> TO <some user, login, or group>`. `ALL` is deprecated and maintained only for backward compatibility; it does not grant all possible permissions. The `GRANT OPTION` indicates that the grantee will also be given the ability to grant the specified permission to other principals.
- **MySQL:** `GRANT` assigns privileges and roles to user accounts and roles. The `ON` clause distinguishes whether the statement grants privileges (with `ON`) or roles (without `ON`). To use `GRANT` to grant privileges, you must have the `GRANT OPTION` privilege and the privileges you are granting.

#### Constraints and Limitations

- **`ALL` is deprecated in SQL Server:** Granting `ALL` is equivalent to granting the following permissions: for databases—`BACKUP DATABASE`, `BACKUP LOG`, `CREATE DATABASE`, `CREATE DEFAULT`, `CREATE FUNCTION`, `CREATE PROCEDURE`, `CREATE RULE`, `CREATE TABLE`, `CREATE VIEW`; for tables—`DELETE`, `INSERT`, `REFERENCES`, `SELECT`, `UPDATE`. Do not rely on `ALL` for complete privilege sets.
- **Column-level grants:** Only `SELECT`, `REFERENCES`, `UPDATE`, and `UNMASK` permissions can be granted on a column in SQL Server. A table-level `DENY` does not take precedence over a column-level `GRANT`.
- **`GRANTED BY`:** Only roles that have the privilege with grant option can use `GRANTED BY`.
- **MySQL:** `GRANT` supports hostnames up to 255 characters; usernames can be up to 32 characters.

### Annotated Code Examples

#### Example 1: PostgreSQL — Granting Table and Column Privileges

```sql
-- Create a sample table
CREATE TABLE sales.orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    order_total NUMERIC(10,2),
    order_date DATE
);

-- Create a role
CREATE ROLE sales_analyst;

-- Grant SELECT on the entire table
GRANT SELECT ON sales.orders TO sales_analyst;

-- Grant SELECT and UPDATE on specific columns
GRANT SELECT (order_id, customer_id, order_total), UPDATE (order_total)
    ON sales.orders TO sales_analyst;

-- Grant SELECT with the ability to grant to others
GRANT SELECT ON sales.customers TO sales_analyst WITH GRANT OPTION;
```

**Expected Output:**

```
GRANT
GRANT
GRANT
```

**Why This Works:** The first `GRANT` gives `sales_analyst` read access to the entire `orders` table. The second `GRANT` restricts access to specific columns—the analyst can read `order_id`, `customer_id`, and `order_total`, and can update only `order_total`. The third `GRANT` on `customers` includes `WITH GRANT OPTION`, allowing `sales_analyst` to grant `SELECT` on `customers` to other users.

#### Example 2: SQL Server — Granting Schema-Level Permissions

```sql
-- Create a role for sales analysts
CREATE ROLE SalesAnalyst;

-- Grant SELECT on all objects in the Sales schema
GRANT SELECT ON SCHEMA::Sales TO SalesAnalyst;

-- Grant EXECUTE on all procedures in the Reports schema
GRANT EXECUTE ON SCHEMA::Reports TO ReportingUsers;

-- Add a user to the role
ALTER ROLE SalesAnalyst ADD MEMBER JohnSmith;
```

**Expected Output:**

```
Commands completed successfully.
```

**Why This Works:** When you grant `SELECT` permission on a schema, users can select from all tables and views in that schema—including objects created in the future. This is a powerful pattern for managing permissions at scale: instead of granting on each table individually, you grant on the schema and all current and future objects are covered.

### Real-World Cases

- **Application access:** Granting `SELECT`, `INSERT`, `UPDATE`, `DELETE` on application tables to a service account.
- **Reporting:** Granting `SELECT` on views to reporting users without exposing base tables.
- **Stored procedures:** Granting `EXECUTE` on procedures to application roles without granting direct table access.
- **Column-level security:** Granting `SELECT` on non-sensitive columns while restricting access to PII columns.

### References

- PostgreSQL: GRANT — https://www.postgresql.org/docs/14/sql-grant.html
- SQL Server: GRANT (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/statements/grant-transact-sql
- MySQL: GRANT Statement — https://dev.mysql.com/doc/refman/8.0/en/grant.html
- Oracle: GRANT — https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/GRANT.html


## Core Concept 2: REVOKE

### Definitions

**Core Definition:** `REVOKE` is the SQL statement that removes previously granted privileges from users or roles, with control over whether dependent grants are also revoked.

**Technical Definition:** The `REVOKE` command revokes previously granted privileges from one or more roles. The `GRANT OPTION FOR` clause revokes only the grant option, not the privilege itself. The `CASCADE` and `RESTRICT` options control the behavior when the privilege being revoked was granted by the grantee to other users: `CASCADE` revokes the privilege from all dependent grantees; `RESTRICT` fails the revocation if dependent grants exist. The `REVOKE` statement can revoke multiple privileges on one object. In SQL Server, `REVOKE` removes a previously granted or denied permission; it does not prevent access through other grants.

**Beginner-Friendly Explanation:** `REVOKE` is the command you use to take away permission. If you gave someone a key, you can take it back. If they made copies of that key for others, you can choose to take back all the copies (`CASCADE`) or refuse to take back the key at all until the copies are returned (`RESTRICT`).

### Purposes

- **To strip previously assigned permissions** when a user changes roles or leaves the organization.
- **To manage cascading revokes** using `CASCADE` or `RESTRICT` to control dependent grants.
- **To revoke the grant option** without revoking the privilege itself, using `GRANT OPTION FOR`.
- **To handle default public privileges** by revoking privileges granted to `PUBLIC`.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
REVOKE [ GRANT OPTION FOR ]
    { { SELECT | INSERT | UPDATE | DELETE | TRUNCATE | REFERENCES | TRIGGER }
    [, ...] | ALL [ PRIVILEGES ] }
    ON { [ TABLE ] table_name [, ...]
       | ALL TABLES IN SCHEMA schema_name [, ...] }
    FROM role_specification [, ...]
    [ GRANTED BY role_specification ]
    [ CASCADE | RESTRICT ]

REVOKE [ GRANT OPTION FOR ]
    { { SELECT | INSERT | UPDATE | REFERENCES } ( column_name [, ...] )
    [, ...] | ALL [ PRIVILEGES ] ( column_name [, ...] ) }
    ON [ TABLE ] table_name [, ...]
    FROM role_specification [, ...]
    [ GRANTED BY role_specification ]
    [ CASCADE | RESTRICT ]

REVOKE [ GRANT OPTION FOR ]
    { { CREATE | CONNECT | TEMPORARY | TEMP } [, ...] | ALL [ PRIVILEGES ] }
    ON DATABASE database_name [, ...]
    FROM role_specification [, ...]
    [ GRANTED BY role_specification ]
    [ CASCADE | RESTRICT ]

REVOKE [ GRANT OPTION FOR ]
    { { CREATE | USAGE } [, ...] | ALL [ PRIVILEGES ] }
    ON SCHEMA schema_name [, ...]
    FROM role_specification [, ...]
    [ GRANTED BY role_specification ]
    [ CASCADE | RESTRICT ]

REVOKE [ ADMIN OPTION FOR ]
    role_name [, ...]
    FROM role_specification [, ...]
    [ GRANTED BY role_specification ]
    [ CASCADE | RESTRICT ]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `GRANT OPTION FOR` | Revokes only the grant option, not the privilege itself. |
| `CASCADE` | Revokes the privilege from all users to whom the grantee granted it. |
| `RESTRICT` | Fails the revocation if the grantee has granted the privilege to others. |
| `GRANTED BY` | Specifies the role that revokes the privilege. |
| `ADMIN OPTION FOR` | Revokes only the admin option for role membership. |

#### Complete General Syntax (SQL Server)

```sql
REVOKE [ GRANT OPTION FOR ]
    { [ ALL [ PRIVILEGES ] ] | permission [ ( column [ , ...n ] ) ] [ , ...n ] }
    [ ON [ class :: ] securable ]
    { TO | FROM } principal [ , ...n ]
    [ CASCADE ]
    [ AS principal ]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `GRANT OPTION FOR` | Revokes only the grant option. |
| `CASCADE` | Revokes the permission from all users to whom the grantee granted it. |
| `TO` / `FROM` | Both are valid; `FROM` is the more common syntax. |
| `AS principal` | Specifies the principal recorded as the revoker. |

#### Syntax Rules

- **PostgreSQL:** The `REVOKE` command revokes previously granted privileges from one or more roles. The keyword `PUBLIC` refers to the implicitly defined group of all roles. The `CASCADE` option revokes the privilege from all dependent grantees; `RESTRICT` fails if dependent grants exist.
- **PostgreSQL:** If `GRANT OPTION FOR` is specified, only the grant option is revoked, not the privilege itself. Otherwise, both the privilege and the grant option are revoked.
- **SQL Server:** `REVOKE` removes a previously granted or denied permission. It does not prevent a user from executing an operation if that operation is permissible due to other grants. To explicitly block access, use `DENY`.
- **Oracle:** The `REVOKE` statement can revoke only privileges and roles that were previously granted directly with a `GRANT` statement. Revoking an object privilege does not revoke the `GRANT OPTION` selectively; you must revoke the object privilege and then grant it again without the `GRANT OPTION`.

#### Constraints and Limitations

- **`RESTRICT` is the default:** If neither `CASCADE` nor `RESTRICT` is specified, the default is `RESTRICT` in PostgreSQL.
- **SQL Server does not support `RESTRICT`:** The `REVOKE` statement exists in Transact-SQL, but the statement does not support the `RESTRICT` keyword.
- **Cascading revokes:** Revoking a privilege with `CASCADE` also revokes the privilege from all users to whom the grantee granted it.
- **Public privileges:** Revoking privileges from `PUBLIC` can affect all users; use with caution.

### Annotated Code Examples

#### Example 1: PostgreSQL — Revoking with CASCADE and RESTRICT

```sql
-- Grant SELECT with GRANT OPTION to user_a
GRANT SELECT ON sales.orders TO user_a WITH GRANT OPTION;

-- user_a grants SELECT to user_b
-- (executed as user_a):
GRANT SELECT ON sales.orders TO user_b;

-- Revoke SELECT from user_a
-- This will fail with RESTRICT (the default) because user_a granted it to user_b
REVOKE SELECT ON sales.orders FROM user_a;
```

**Expected Output (with RESTRICT):**

```
ERROR:  dependent privileges exist
HINT:  Use CASCADE to revoke them too.
```

**Expected Output (with CASCADE):**

```sql
REVOKE SELECT ON sales.orders FROM user_a CASCADE;
```

```
REVOKE
-- user_b also loses SELECT
```

**Why This Works:** The `RESTRICT` option (the default) prevents the revocation if the grantee has granted the privilege to others. The `CASCADE` option revokes the privilege from `user_a` and also from `user_b`, who received it from `user_a`.

#### Example 2: SQL Server — Revoking and Denying

```sql
-- Grant SELECT to a role
GRANT SELECT ON dbo.Customers TO CustomerServiceRole;

-- Revoke the grant
REVOKE SELECT ON dbo.Customers FROM CustomerServiceRole;

-- Explicitly deny access (overrides any grants)
DENY SELECT ON dbo.EmployeeSalaries TO HRAssistants;
```

**Expected Output:**

```
Commands completed successfully.
```

**Why This Works:** `REVOKE` removes the grant but does not prevent access through other grants (e.g., through role membership). `DENY` explicitly blocks the permission and overrides any grants, providing a stronger guarantee of no access.

### Real-World Cases

- **Employee offboarding:** Revoking all privileges from a user account when an employee leaves.
- **Role changes:** Revoking privileges from a user who has moved to a different department.
- **Security incident:** Revoking compromised credentials and their associated privileges.
- **Privilege cleanup:** Removing excessive privileges identified during a security audit.

### References

- PostgreSQL: REVOKE — https://www.postgresql.org/docs/16/sql-revoke.html
- SQL Server: REVOKE (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/statements/revoke-transact-sql
- Oracle: REVOKE — https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/REVOKE.html
- MySQL: REVOKE Statement — https://dev.mysql.com/doc/refman/8.0/en/revoke.html


## Core Concept 3: Object Privileges

### Definitions

**Core Definition:** Object privileges are fine-grained permissions granted on specific database objects—tables, views, columns, sequences, functions, and procedures—that control what actions a user can perform on that object.

**Technical Definition:** Object privileges enable you to perform actions on schema objects, such as tables or indexes. Each type of object has a specific set of privileges that can be granted. For tables and views: `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`, `REFERENCES`, `TRIGGER`. For sequences: `USAGE`, `SELECT`, `UPDATE`. For functions and procedures: `EXECUTE`. Column-level privileges allow granting `SELECT`, `INSERT`, `UPDATE`, and `REFERENCES` on individual columns within a table, providing granular control over sensitive data.

**Beginner-Friendly Explanation:** Object privileges are the most specific kind of permission—they apply to one particular table, view, or column. For example, you can give someone permission to read only the `name` and `email` columns of a `customers` table, but not the `credit_card` column. This is how you protect sensitive data while still allowing access to the rest.

### Purposes

- **To manage fine-grained permissions** at the individual table, view, column, or sequence level.
- **To restrict direct exposure to underlying datasets** by granting on views instead of base tables.
- **To implement column-level security** for PII, financial, or healthcare data.
- **To control access to sequences** for auto-increment ID generation.

### Syntax Rules and Structure

#### Complete General Syntax (Table Privileges)

```sql
GRANT { { SELECT | INSERT | UPDATE | DELETE | TRUNCATE | REFERENCES | TRIGGER }
    [, ...] | ALL [ PRIVILEGES ] }
    ON [ TABLE ] table_name [, ...]
    TO role_specification [, ...]
    [ WITH GRANT OPTION ];

REVOKE [ GRANT OPTION FOR ]
    { { SELECT | INSERT | UPDATE | DELETE | TRUNCATE | REFERENCES | TRIGGER }
    [, ...] | ALL [ PRIVILEGES ] }
    ON [ TABLE ] table_name [, ...]
    FROM role_specification [, ...]
    [ CASCADE | RESTRICT ];
```

#### Complete General Syntax (Column-Level Privileges)

```sql
-- PostgreSQL
GRANT { { SELECT | INSERT | UPDATE | REFERENCES } ( column_name [, ...] )
    [, ...] | ALL [ PRIVILEGES ] ( column_name [, ...] ) }
    ON [ TABLE ] table_name [, ...]
    TO role_specification [, ...]
    [ WITH GRANT OPTION ];

-- SQL Server
GRANT { SELECT | REFERENCES | UPDATE | UNMASK } ( column_name [ , ...n ] )
    ON [ OBJECT :: ] [ schema_name . ] object_name
    TO database_principal [ , ...n ]
    [ WITH GRANT OPTION ];
```

#### Complete General Syntax (Sequence Privileges)

```sql
GRANT { { USAGE | SELECT | UPDATE } [, ...] | ALL [ PRIVILEGES ] }
    ON { SEQUENCE sequence_name [, ...] }
    TO role_specification [, ...]
    [ WITH GRANT OPTION ];
```

#### Syntax Rules

- **PostgreSQL:** `GRANT` on a table allows the grantee to perform the specified operations on all columns of the table, unless column-level grants are used. Column-level grants restrict access to specific columns. Only `SELECT`, `INSERT`, `UPDATE`, and `REFERENCES` can be granted at the column level.
- **SQL Server:** Only `SELECT`, `REFERENCES`, `UPDATE`, and `UNMASK` permissions can be granted on a column. A table-level `DENY` does not take precedence over a column-level `GRANT`; this inconsistency is preserved for backward compatibility and will be removed in future versions.
- **Oracle:** A user automatically has all object privileges for schema objects contained in their own schema. For objects in other schemas, privileges must be granted explicitly.

#### Constraints and Limitations

- **Column-level grants:** Not all permissions can be granted at the column level. In SQL Server, `INSERT` and `DELETE` cannot be column-level permissions.
- **Table-level DENY vs. column-level GRANT:** In SQL Server, a table-level `DENY` does not override a column-level `GRANT`. This is a known inconsistency preserved for backward compatibility.
- **Performance:** Column-level grants may require more metadata and can affect query optimization.
- **Views:** Granting on views rather than base tables is a common security pattern to restrict direct access to underlying data.

### Annotated Code Examples

#### Example 1: PostgreSQL — Column-Level Privileges for PII Protection

```sql
-- Create a customers table with sensitive data
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT,
    phone TEXT,
    credit_card TEXT,
    ssn TEXT
);

-- Create a role for customer service
CREATE ROLE customer_service;

-- Grant SELECT on non-sensitive columns only
GRANT SELECT (customer_id, name, email, phone)
    ON customers TO customer_service;

-- Grant UPDATE on email and phone only
GRANT UPDATE (email, phone)
    ON customers TO customer_service;
```

**Expected Output:**

```
GRANT
GRANT
```

**Why This Works:** The `customer_service` role can read `customer_id`, `name`, `email`, and `phone`, but cannot read `credit_card` or `ssn`. They can update only `email` and `phone`. This enforces column-level security for sensitive data.

#### Example 2: Oracle — Object Privileges on Tables and Views

```sql
-- Create a view that exposes only non-sensitive data
CREATE VIEW customer_public AS
SELECT customer_id, name, email
FROM customers;

-- Grant SELECT on the view
GRANT SELECT ON customer_public TO reporting_role;

-- Grant SELECT, INSERT, UPDATE on the base table to the application role
GRANT SELECT, INSERT, UPDATE ON customers TO app_role;

-- Revoke INSERT from the application role
REVOKE INSERT ON customers FROM app_role;
```

**Expected Output:**

```
View created.
Grant succeeded.
Grant succeeded.
Revoke succeeded.
```

**Why This Works:** The `reporting_role` can read the `customer_public` view but has no access to the base `customers` table. The `app_role` has `SELECT` and `UPDATE` on the base table, but `INSERT` was revoked. This layered approach protects sensitive columns while allowing necessary operations.

### Real-World Cases

- **Healthcare:** Granting `SELECT` on patient demographics but not on medical records.
- **Finance:** Granting `UPDATE` on transaction amounts to a reconciliation role but not `DELETE`.
- **E-commerce:** Granting `EXECUTE` on order-processing procedures without direct table access.
- **SaaS:** Granting per-tenant access to sequences for order ID generation.

### References

- PostgreSQL: GRANT (Object Privileges) — https://www.postgresql.org/docs/14/sql-grant.html
- SQL Server: GRANT Object Permissions — https://learn.microsoft.com/en-us/sql/t-sql/statements/grant-object-permissions-transact-sql
- Oracle: Managing Object Privileges — https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/managing-object-privileges.html
- MySQL: Privileges Provided by MySQL — https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html


## Core Concept 4: Schema Privileges

### Definitions

**Core Definition:** Schema privileges are permissions that control access to a schema as a namespace—allowing users to use objects within the schema (`USAGE`) or create new objects within it (`CREATE`).

**Technical Definition:** Schema-level privileges govern broader object-creation and visibility structures within namespaces. The `USAGE` privilege allows access to objects contained in the specified schema (assuming the objects' own privilege requirements are also met). Without `USAGE`, a user cannot access any object in the schema, even if they have object-level privileges. The `CREATE` privilege allows new objects to be created within the schema. The `ALTER DEFAULT PRIVILEGES` statement defines the default set of access permissions to be applied to objects created in the future by the specified user.

**Beginner-Friendly Explanation:** Schema privileges are like the keys to a floor in a building. `USAGE` lets you enter the floor and access the rooms (tables) inside. `CREATE` lets you build new rooms on that floor. Even if you have a key to a specific room (object privilege), you cannot get to it without the floor key (`USAGE`).

### Purposes

- **To regulate broader object-creation and visibility structures** within namespaces.
- **To enable access to objects within a schema** using the `USAGE` privilege.
- **To allow users to create objects in a schema** using the `CREATE` privilege.
- **To set default privileges for future objects** using `ALTER DEFAULT PRIVILEGES`, avoiding the need to grant privileges on each new object individually.
- **To control which schemas are visible** to which users through the search path.

### Syntax Rules and Structure

#### Complete General Syntax (Schema Privileges)

```sql
-- PostgreSQL
GRANT { { CREATE | USAGE } [, ...] | ALL [ PRIVILEGES ] }
    ON SCHEMA schema_name [, ...]
    TO role_specification [, ...]
    [ WITH GRANT OPTION ];

REVOKE [ GRANT OPTION FOR ]
    { { CREATE | USAGE } [, ...] | ALL [ PRIVILEGES ] }
    ON SCHEMA schema_name [, ...]
    FROM role_specification [, ...]
    [ CASCADE | RESTRICT ];
```

#### Complete General Syntax (ALTER DEFAULT PRIVILEGES)

```sql
ALTER DEFAULT PRIVILEGES
    [ FOR { ROLE | USER } target_role [, ...] ]
    [ IN SCHEMA schema_name [, ...] ]
    abbreviated_grant_or_revoke

-- abbreviated_grant_or_revoke:
GRANT { { SELECT | INSERT | UPDATE | DELETE | TRUNCATE | REFERENCES | TRIGGER }
    [, ...] | ALL [ PRIVILEGES ] }
    ON TABLES
    TO { [ GROUP ] role_name | PUBLIC } [, ...]
    [ WITH GRANT OPTION ]

GRANT { { USAGE | SELECT | UPDATE } [, ...] | ALL [ PRIVILEGES ] }
    ON SEQUENCES
    TO { [ GROUP ] role_name | PUBLIC } [, ...]
    [ WITH GRANT OPTION ]

GRANT { EXECUTE | ALL [ PRIVILEGES ] }
    ON { FUNCTIONS | ROUTINES }
    TO { [ GROUP ] role_name | PUBLIC } [, ...]
    [ WITH GRANT OPTION ]

GRANT { { USAGE | CREATE } [, ...] | ALL [ PRIVILEGES ] }
    ON SCHEMAS
    TO { [ GROUP ] role_name | PUBLIC } [, ...]
    [ WITH GRANT OPTION ]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `USAGE` | Allows access to objects contained in the schema. |
| `CREATE` | Allows new objects to be created within the schema. |
| `FOR { ROLE \| USER }` | Specifies the role(s) whose default privileges are being altered. |
| `IN SCHEMA` | Specifies the schema(s) to which the default privileges apply. |
| `ON TABLES` | Default privileges for tables created in the future. |
| `ON SEQUENCES` | Default privileges for sequences created in the future. |
| `ON FUNCTIONS` / `ON ROUTINES` | Default privileges for functions and procedures. |
| `ON SCHEMAS` | Default privileges for schemas created in the future. |

#### Syntax Rules

- **PostgreSQL:** For schemas, `USAGE` allows access to objects contained in the specified schema (assuming the objects' own privilege requirements are also met). Without `USAGE`, a user cannot access any object in the schema, even if they have object-level privileges.
- **PostgreSQL:** `CREATE` on a schema allows new objects to be created within the schema. This is the only type of privilege that is applicable to functions and procedures at the schema level.
- **PostgreSQL:** By default, everyone has `CREATE` and `USAGE` privileges on the `public` schema. In PostgreSQL 15+, this was changed so that the `public` schema is no longer world-writable by default.
- **ALTER DEFAULT PRIVILEGES:** Allows you to set the privileges that will be applied to objects created in the future. It does not affect privileges assigned to already-existing objects. Privileges can be set globally or for objects created in specified schemas.

#### Constraints and Limitations

- **Public schema changes:** In PostgreSQL 15+, the `public` schema is no longer world-writable. The default privileges on `public` were changed to `USAGE` for all users, but `CREATE` is no longer granted to `PUBLIC`.
- **Schema ownership:** Only the schema owner or a superuser can grant `USAGE` or `CREATE` on a schema.
- **ALTER DEFAULT PRIVILEGES:** You can only alter your own default privileges and the defaults of roles that you are a member of. Only a superuser can specify default permissions for other users.

### Annotated Code Examples

#### Example 1: PostgreSQL — Schema USAGE and CREATE Privileges

```sql
-- Create a schema
CREATE SCHEMA reporting AUTHORIZATION report_admin;

-- Create a role
CREATE ROLE report_user;

-- Grant USAGE (access to objects in the schema)
GRANT USAGE ON SCHEMA reporting TO report_user;

-- Grant CREATE (ability to create objects in the schema)
GRANT CREATE ON SCHEMA reporting TO report_user;

-- Verify: report_user can now access and create objects in the reporting schema
```

**Expected Output:**

```
CREATE SCHEMA
CREATE ROLE
GRANT
GRANT
```

**Why This Works:** `USAGE` allows `report_user` to access objects in the `reporting` schema (assuming they have object-level privileges). `CREATE` allows `report_user` to create new tables, views, and other objects in the schema.

#### Example 2: PostgreSQL — ALTER DEFAULT PRIVILEGES

```sql
-- Set default privileges for tables created in the future by the etl_user
ALTER DEFAULT PRIVILEGES FOR ROLE etl_user IN SCHEMA staging
    GRANT SELECT ON TABLES TO reporting_role;

-- Set default privileges for sequences
ALTER DEFAULT PRIVILEGES FOR ROLE etl_user IN SCHEMA staging
    GRANT USAGE, SELECT ON SEQUENCES TO reporting_role;

-- Now, when etl_user creates a new table in staging,
-- reporting_role automatically gets SELECT on it.
```

**Expected Output:**

```
ALTER DEFAULT PRIVILEGES
ALTER DEFAULT PRIVILEGES
```

**Why This Works:** `ALTER DEFAULT PRIVILEGES` configures automatic grants for future objects. When `etl_user` creates a new table in the `staging` schema, the `reporting_role` automatically receives `SELECT` on that table—without needing a separate `GRANT` statement.

### Real-World Cases

- **Data warehousing:** Granting `USAGE` on the `staging` schema to ETL roles and `SELECT` on `reporting` schema to analysts.
- **ETL pipelines:** Using `ALTER DEFAULT PRIVILEGES` to automatically grant read access to reports as new tables are created.
- **Multi-tenant SaaS:** Granting `USAGE` and `CREATE` on tenant-specific schemas to tenant administrators.
- **Development environments:** Granting `USAGE` on the `public` schema while restricting `CREATE` to prevent schema pollution.

### References

- PostgreSQL: GRANT (Schema Privileges) — https://www.postgresql.org/docs/14/sql-grant.html
- PostgreSQL: ALTER DEFAULT PRIVILEGES — https://www.postgresql.org/docs/16/sql-alterdefaultprivileges.html
- PostgreSQL: Schemas and Privileges — https://www.postgresql.org/docs/current/ddl-schemas.html
- SQL Server: GRANT Schema Permissions — https://learn.microsoft.com/en-us/sql/t-sql/statements/grant-schema-permissions-transact-sql


## Core Concept 5: Database Privileges

### Definitions

**Core Definition:** Database privileges are macroscopic access controls that govern a user's ability to connect to a database, create schemas within it, and create temporary tables during a session.

**Technical Definition:** Database-level privileges in PostgreSQL are `CREATE`, `CONNECT`, and `TEMPORARY` (or `TEMP`). `CONNECT` allows the user to connect to the specified database; this privilege is checked at connection startup (in addition to any restrictions imposed by `pg_hba.conf`). `CREATE` allows new schemas and publications to be created within the database. `TEMPORARY` allows the user to create temporary tables. The `ALL` keyword grants all privileges available on the database.

**Beginner-Friendly Explanation:** Database privileges are like the key to the front door of a building. `CONNECT` lets you enter the building (connect to the database). `CREATE` lets you build new floors (schemas). `TEMPORARY` lets you set up temporary work areas. Without `CONNECT`, you cannot even enter the database, no matter what other permissions you have.

### Purposes

- **To control macroscopic access vectors** like `CONNECT`, `CREATE`, or `TEMPORARY` at the root database structural boundary.
- **To restrict who can connect to a database** at the connection level.
- **To allow or prevent schema creation** within a database.
- **To control temporary table usage** for sessions that need scratch space.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
GRANT { { CREATE | CONNECT | TEMPORARY | TEMP } [, ...] | ALL [ PRIVILEGES ] }
    ON DATABASE database_name [, ...]
    TO role_specification [, ...]
    [ WITH GRANT OPTION ];

REVOKE [ GRANT OPTION FOR ]
    { { CREATE | CONNECT | TEMPORARY | TEMP } [, ...] | ALL [ PRIVILEGES ] }
    ON DATABASE database_name [, ...]
    FROM role_specification [, ...]
    [ CASCADE | RESTRICT ];
```

**Component Breakdown:**

| Privilege | Description |
|-----------|-------------|
| `CONNECT` | Allows the user to connect to the specified database. Checked at connection startup in addition to `pg_hba.conf` restrictions. |
| `CREATE` | Allows new schemas and publications to be created within the database. |
| `TEMPORARY` / `TEMP` | Allows the user to create temporary tables within the database. |
| `ALL` | Grants all privileges available on the database. |

#### Syntax Rules

- **PostgreSQL:** The `CONNECT` privilege is checked at connection startup in addition to the restrictions imposed by `pg_hba.conf`. Revoking `CONNECT` from `PUBLIC` prevents any user who is not explicitly granted `CONNECT` from connecting to the database.
- **MySQL:** Database privileges apply to all objects in a given database. The privileges include `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `DROP`, `GRANT OPTION`, `REFERENCES`, `INDEX`, `ALTER`, `CREATE TEMPORARY TABLES`, `LOCK TABLES`, `CREATE VIEW`, `SHOW VIEW`, `CREATE ROUTINE`, `ALTER ROUTINE`, `EXECUTE`, `EVENT`, `TRIGGER`.
- **SQL Server:** Database-level permissions govern actions within a specific database. These include `CONNECT`, `CREATE TABLE`, `CREATE VIEW`, `CREATE PROCEDURE`, `BACKUP DATABASE`, `BACKUP LOG`, `ALTER`, `CONTROL`.

#### Constraints and Limitations

- **PostgreSQL:** `CONNECT` is checked in addition to `pg_hba.conf`; both must allow the connection for it to succeed.
- **PostgreSQL:** Revoking `CONNECT` from `PUBLIC` is a common security hardening step, but it means you must explicitly grant `CONNECT` to every user who needs access.
- **MySQL:** Database-level privileges are distinct from global privileges (`*.*`) and table-level privileges (`db.table`). `GRANT ALL ON db_name.*` grants all database-level privileges, not global privileges.
- **SQL Server:** `CONNECT` permission is granted by default to the `public` role; revoking it requires explicit management.

### Annotated Code Examples

#### Example 1: PostgreSQL — Database CONNECT and CREATE Privileges

```sql
-- Create a database
CREATE DATABASE appdb;

-- Create a role
CREATE ROLE app_user;

-- Grant CONNECT and CREATE on the database
GRANT CONNECT, CREATE ON DATABASE appdb TO app_user;

-- Revoke CONNECT from PUBLIC (security hardening)
REVOKE CONNECT ON DATABASE appdb FROM PUBLIC;

-- Verify: only explicitly granted users can connect
```

**Expected Output:**

```
CREATE DATABASE
CREATE ROLE
GRANT
REVOKE
```

**Why This Works:** `GRANT CONNECT, CREATE ON DATABASE` allows `app_user` to connect to `appdb` and create schemas within it. `REVOKE CONNECT ON DATABASE appdb FROM PUBLIC` removes the default public connect privilege, ensuring that only users explicitly granted `CONNECT` can access the database.

#### Example 2: MySQL — Database-Level Privileges

```sql
-- Grant all database-level privileges on the 'sales' database
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER
    ON sales.*
    TO 'app_user'@'%';

-- Grant only SELECT on the 'reports' database
GRANT SELECT ON reports.* TO 'report_user'@'%';

-- Revoke INSERT on the 'sales' database
REVOKE INSERT ON sales.* FROM 'app_user'@'%';
```

**Expected Output:**

```
Query OK, 0 rows affected
```

**Why This Works:** The `ON sales.*` syntax grants privileges on all objects in the `sales` database. `GRANT SELECT ON reports.*` gives read-only access to the `reports` database. `REVOKE INSERT ON sales.*` removes the insert privilege from `app_user`.

### Real-World Cases

- **Multi-tenant SaaS:** Each tenant database has `CONNECT` granted only to the tenant's service account.
- **Security hardening:** Revoking `CONNECT` from `PUBLIC` on production databases to prevent unauthorized access.
- **Development environments:** Granting `CREATE` on development databases to allow developers to create schemas.
- **Analytics:** Granting `CONNECT` and `TEMPORARY` to analytics users who need scratch space for ad-hoc queries.

### References

- PostgreSQL: GRANT (Database Privileges) — https://www.postgresql.org/docs/14/sql-grant.html
- MySQL: Privileges Provided by MySQL — https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html
- SQL Server: GRANT Database Permissions — https://learn.microsoft.com/en-us/sql/t-sql/statements/grant-database-permissions-transact-sql
- Oracle: System Privileges — https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/GRANT.html


## Core Concept 6: Role-Based Access Control

### Definitions

**Core Definition:** Role-based access control (RBAC) is a security model in which permissions are assigned to roles (logical containers representing job functions), and users are granted membership in those roles, inheriting the associated permissions.

**Technical Definition:** RBAC in SQL databases is implemented through roles—named collections of privileges that can be granted to users or other roles. The `GRANT role_name TO user_name` statement grants membership in a role. Roles can be nested (a role can be granted to another role), creating a hierarchy. The `WITH ADMIN OPTION` clause allows the grantee to grant membership in the role to others. Default system roles (e.g., PostgreSQL's `pg_read_all_data`, SQL Server's `db_datareader`) provide predefined privilege sets.

**Beginner-Friendly Explanation:** RBAC is like job titles in a company. Instead of giving each person their own set of permissions, you define roles: "Manager," "Analyst," "Intern." Each role has a standard set of permissions. When someone becomes a manager, you assign them the "Manager" role, and they automatically get all the associated permissions. When the role's permissions change, everyone with that role gets the update automatically.

### Purposes

- **To design modular security matrices** by grouping functional grants into application, read-only, read-write, or administrative roles.
- **To simplify privilege management** at scale by avoiding per-user grants.
- **To support role hierarchies** where roles inherit privileges from other roles.
- **To enable default roles** that are automatically activated at login.
- **To enforce separation of duties** through distinct roles for different administrative functions.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
-- Create a role
CREATE ROLE role_name [ [ WITH ] option [ ... ] ];

-- Grant a role to a user (or another role)
GRANT role_name [, ...]
    TO role_specification [, ...]
    [ WITH ADMIN OPTION ]
    [ GRANTED BY role_specification ];

-- Revoke a role from a user
REVOKE [ ADMIN OPTION FOR ]
    role_name [, ...]
    FROM role_specification [, ...]
    [ GRANTED BY role_specification ]
    [ CASCADE | RESTRICT ];
```

#### Complete General Syntax (SQL Server)

```sql
-- Create a user-defined database role
CREATE ROLE role_name [ AUTHORIZATION owner_name ];

-- Add a user to a database role
ALTER ROLE role_name ADD MEMBER user_name;

-- Remove a user from a role
ALTER ROLE role_name DROP MEMBER user_name;

-- Fixed database roles (built-in)
-- db_owner, db_securityadmin, db_accessadmin, db_backupoperator,
-- db_ddladmin, db_datareader, db_datawriter, db_denydatareader,
-- db_denydatawriter
```

#### Complete General Syntax (MySQL)

```sql
-- Create a role
CREATE ROLE 'role_name'@'host';

-- Grant privileges to a role
GRANT SELECT, INSERT ON db_name.* TO 'role_name'@'host';

-- Grant a role to a user
GRANT 'role_name'@'host' TO 'user_name'@'host';

-- Set default role for a user
SET DEFAULT ROLE 'role_name'@'host' TO 'user_name'@'host';
```

#### Syntax Rules

- **PostgreSQL:** `CREATE ROLE` does not include `LOGIN` by default; `CREATE USER` does. Roles can be granted to other roles, creating a hierarchy. The `INHERIT` attribute (default) determines whether a role automatically inherits the privileges of roles it is a member of.
- **PostgreSQL:** Default roles are a set of predefined roles that provide access to commonly needed, privileged capabilities and information. Administrators can `GRANT` these roles to users and/or other roles in their environment.
- **SQL Server:** Fixed database roles include `db_owner` (full control), `db_datareader` (read all data), `db_datawriter` (write all data), and `db_denydatareader`/`db_denydatawriter` (deny read/write). Users can be members of multiple roles; permissions are cumulative.
- **MySQL:** Roles are named collections of privileges. The `SET DEFAULT ROLE` statement specifies which roles are active by default when a user connects.

#### Constraints and Limitations

- **Role explosion:** Over time, an excessive number of roles can become difficult to manage. Regular role consolidation and review are necessary.
- **Privilege inheritance risk:** If a role is granted `WITH ADMIN OPTION`, the grantee can grant the role to others, potentially escalating privileges.
- **Default role dependency:** In MySQL, roles are not active by default unless `SET DEFAULT ROLE` is executed or `activate_all_roles_on_login` is enabled.
- **SQL Server:** Fixed database roles cannot be modified; their permissions are predefined.

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
-- Create custom roles for job functions
CREATE ROLE DataReaders;
CREATE ROLE DataWriters;

-- Grant appropriate permissions to each role
GRANT SELECT ON SCHEMA::dbo TO DataReaders;
GRANT INSERT, UPDATE, DELETE ON SCHEMA::dbo TO DataWriters;

-- Add users to roles
ALTER ROLE DataReaders ADD MEMBER JohnSmith;
ALTER ROLE DataWriters ADD MEMBER JaneDoc;
```

**Expected Output:**

```
Commands completed successfully.
```

**Why This Works:** `DataReaders` has only `SELECT` permission on the `dbo` schema. `DataWriters` has `INSERT`, `UPDATE`, and `DELETE`. Users are assigned to the appropriate role based on their job function. This is a clean, scalable RBAC design.

### Real-World Cases

- **Application roles:** Separate roles for read-only reporting, read-write application access, and administrative functions.
- **DBA roles:** Distinct roles for backup operators, security administrators, and database administrators.
- **Default roles:** PostgreSQL's `pg_read_all_data`, `pg_write_all_data`, and `pg_monitor` roles for common monitoring and reporting needs.
- **Multi-tenant SaaS:** Each tenant has its own role with access to its schema only.

### References

- PostgreSQL: CREATE ROLE — https://www.postgresql.org/docs/current/sql-createrole.html
- PostgreSQL: Default Roles — https://www.postgresql.org/docs/current/default-roles.html
- SQL Server: CREATE ROLE (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-role-transact-sql
- SQL Server: Database-Level Roles — https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/database-level-roles
- MySQL: CREATE ROLE — https://dev.mysql.com/doc/refman/8.0/en/create-role.html
- Oracle: CREATE ROLE — https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/CREATE-ROLE.html


## Summary Table: SQL Privileges at a Glance

| Concept | Key Constructs | Scope | Key Consideration |
|---------|---------------|-------|-------------------|
| **GRANT** | `GRANT privilege ON object TO principal` | Object, schema, database, instance | `WITH GRANT OPTION` delegates grant authority |
| **REVOKE** | `REVOKE privilege ON object FROM principal` | Object, schema, database, instance | `CASCADE` vs. `RESTRICT` controls dependent grants |
| **Object Privileges** | `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `EXECUTE`, `REFERENCES` | Table, view, column, sequence | Column-level grants for PII protection |
| **Schema Privileges** | `USAGE`, `CREATE`, `ALTER DEFAULT PRIVILEGES` | Schema | `USAGE` is required to access objects in a schema |
| **Database Privileges** | `CONNECT`, `CREATE`, `TEMPORARY` | Database | `CONNECT` is checked at connection time |
| **RBAC** | `CREATE ROLE`, `GRANT role TO user` | Instance, database | Roles can be nested; permissions are cumulative |


## Final Notes on Deprecated and Unsafe Features

- **SQL Server `ALL` permission:** `ALL` is deprecated and maintained only for backward compatibility. It does not grant all possible permissions. Granting `ALL` is equivalent to granting a specific subset of permissions (e.g., for tables: `DELETE`, `INSERT`, `REFERENCES`, `SELECT`, `UPDATE`). Do not rely on `ALL` for complete privilege sets.
- **SQL Server table-level `DENY` vs. column-level `GRANT`:** A table-level `DENY` does not take precedence over a column-level `GRANT`. This inconsistency is preserved for backward compatibility and will be removed in future versions.
- **PostgreSQL `public` schema write access:** In PostgreSQL 15+, the `public` schema is no longer world-writable by default. The `CREATE` privilege is no longer granted to `PUBLIC`. Do not rely on `public` for application objects.
- **Cascading revokes:** Revoking a privilege with `CASCADE` also revokes the privilege from all users to whom the grantee granted it. Use with caution in production environments.
- **`GRANT ALL ON *.*` in MySQL:** Grants all global privileges, including `SUPER`, `PROCESS`, and `FILE`. This is extremely powerful and should be restricted to administrative accounts only.
- **Oracle `GRANT ANY OBJECT PRIVILEGE`:** This system privilege allows the grantee to grant and revoke object privileges on behalf of the object owner. It should be granted sparingly.
- **Default privileges in PostgreSQL:** `ALTER DEFAULT PRIVILEGES` only affects objects created by the specified role in the specified schema; it does not retroactively apply to existing objects.
- **Version-specific:** PostgreSQL 15+ changed the default privileges on the `public` schema. MySQL 8.0+ supports roles. SQL Server 2022 introduced the `##MS_LoginManager##` fixed server role for `CREATE LOGIN`.