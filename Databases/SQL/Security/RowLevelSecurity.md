# SQL Row-Level Security: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL Row-Level Security (RLS) is a database security feature that restricts which rows a user can see or modify in a table, based on the user's identity, role, or session context—enforced automatically by the database engine at query time.

**Technical Definition:** Row-level security (RLS) enables you to use group membership or execution context to control access to rows in a database table. The access restriction logic is located in the database tier rather than away from the data in another application tier. The database system applies the access restrictions every time that data access is attempted from any tier, making the security system more reliable and robust by reducing the surface area of the security system . RLS is implemented through policy functions (predicates) that return a filter condition, which the database engine appends to queries automatically. When RLS is enabled on a table and no applicable policies exist, a "default deny" policy is assumed, so that no rows will be visible or updatable .

**Beginner-Friendly Explanation:** Row-level security is like giving each employee a keycard that only unlocks the filing cabinets and drawers they are allowed to see. Even if they walk into the same room (the same table), they can only open the drawers (rows) that belong to them. The database checks their keycard automatically on every query—no application code needs to add "WHERE user_id = me" to every statement.

### Key Characteristics

- **Engine-enforced:** Filtering is applied by the database engine at query time, not by application code.
- **Transparent:** Applications do not need to be modified; the database silently filters rows.
- **Policy-based:** Rules are expressed as SQL predicate functions or USING/WITH CHECK expressions.
- **Default-deny:** When RLS is enabled and no policy applies, no rows are visible or modifiable.
- **Context-dependent:** Filters can reference `CURRENT_USER`, `SESSION_USER`, `SESSION_CONTEXT()`, or custom session variables.
- **Multi-tenant-ready:** Ideal for SaaS architectures where multiple tenants share a single database.
- **Superuser-aware:** Superusers and table owners typically bypass RLS unless `FORCE ROW LEVEL SECURITY` is applied.

### Prerequisites

- Basic understanding of SQL `SELECT`, `INSERT`, `UPDATE`, `DELETE`, and `WHERE` clauses.
- Familiarity with database users, roles, and privileges.
- Knowledge of functions and stored procedures.
- Awareness of session context mechanisms (e.g., `SET ROLE`, `SESSION_CONTEXT()`).
- Understanding of multi-tenant architecture patterns.

### Related Programming Areas

- Multi-tenant SaaS application design.
- Database security and compliance (GDPR, HIPAA, PCI-DSS).
- Application security and access control.
- Data governance and privacy.
- Database administration and performance tuning.

### Core Concepts / Features

1. **User-Specific Visibility** (filtering rows by `CURRENT_USER`, `SESSION_USER`, or session context)
2. **Tenant Isolation** (locking records to a `tenant_id` column value)
3. **Policy-Based Filtering** (`CREATE POLICY`, Oracle VPD)
4. **Multi-Tenant Access Control** (enabling/disabling RLS, superuser bypass, default-deny mechanics)


## Core Concept 1: User-Specific Visibility

### Definitions

**Core Definition:** User-specific visibility is the row-level security technique that isolates table rows natively by applying automatic filter predicates based on the running session user context, using functions like `CURRENT_USER` or `SESSION_USER`.

**Technical Definition:** User-specific visibility policies filter rows based on the identity of the user executing the query. In PostgreSQL, RLS policies reference `current_user` (the user identifier applicable for permission checking, which can be changed with `SET ROLE`) or `session_user` (the user who initiated the current database connection, which superusers can change with `SET SESSION AUTHORIZATION`) . In SQL Server, `SESSION_CONTEXT()` stores key-value pairs set by the application after opening a connection, allowing the application to inject the current user ID into the RLS predicate . In Oracle VPD, policy functions reference `SYS_CONTEXT('USERENV', 'SESSION_USER')` or application contexts to determine the current user .

**Beginner-Friendly Explanation:** User-specific visibility is like a personalized newspaper that only shows articles relevant to you. The database looks at who you are (your username) and automatically filters out rows that do not belong to you—without you having to ask for "my rows only." The filtering happens behind the scenes.

### Purposes

- **To isolate table rows natively** by applying automatic filter predicates based on the running session user context.
- **To enforce "see your own data" rules** without relying on application code to add `WHERE` clauses.
- **To support application-level user impersonation** through session context injection (e.g., `SESSION_CONTEXT()` in SQL Server).
- **To provide defense in depth** so that even if application code forgets a filter, the database still enforces isolation.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL — User-Specific Policy)

```sql
-- Step 1: Enable RLS on the table
ALTER TABLE table_name ENABLE ROW LEVEL SECURITY;

-- Step 2: Create a policy using CURRENT_USER
CREATE POLICY policy_name ON table_name
    FOR SELECT
    TO authenticated_role
    USING (owner = CURRENT_USER);

-- Step 3: Create a policy for INSERT/UPDATE with WITH CHECK
CREATE POLICY policy_name_insert ON table_name
    FOR INSERT
    TO authenticated_role
    WITH CHECK (owner = CURRENT_USER);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ALTER TABLE ... ENABLE ROW LEVEL SECURITY` | Enables RLS enforcement on the table. |
| `CREATE POLICY` | Defines a new RLS policy. |
| `FOR SELECT` | Restricts the policy to SELECT operations. |
| `TO authenticated_role` | Applies the policy only to the specified role. |
| `USING (owner = CURRENT_USER)` | Filter condition for visible rows. |
| `WITH CHECK (owner = CURRENT_USER)` | Validation for inserted/updated rows. |

#### Complete General Syntax (SQL Server — Session Context)

```sql
-- Step 1: Create a predicate function that reads SESSION_CONTEXT
CREATE FUNCTION dbo.fn_user_filter(@user_id INT)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN
    SELECT 1 AS fn_result
    WHERE @user_id = CAST(SESSION_CONTEXT(N'UserId') AS INT);

-- Step 2: Create a security policy
CREATE SECURITY POLICY UserFilterPolicy
ADD FILTER PREDICATE dbo.fn_user_filter(user_id)
ON dbo.Sales
WITH (STATE = ON);

-- Step 3: Application sets the session context after opening a connection
EXEC sp_set_session_context @key=N'UserId', @value=1;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `SESSION_CONTEXT(N'UserId')` | Reads the session-scoped key-value pair set by the application. |
| `sp_set_session_context` | Sets a session-scoped key-value pair. |
| `@read_only = 1` | Prevents modification of the value until the connection closes. |

#### Syntax Rules

- **PostgreSQL:** `current_user` is the effective user for permission checking; `session_user` is the real user who opened the connection. RLS uses `current_user` for policy evaluation.
- **SQL Server:** `SESSION_CONTEXT()` is set by the application after opening a connection. The application is responsible for setting the current user ID in `SESSION_CONTEXT()` after opening a connection .
- **Oracle:** `SYS_CONTEXT('USERENV', 'SESSION_USER')` returns the database user name of the user logged in. Application contexts (`SYS_CONTEXT('context_name', 'attribute')`) provide additional flexibility.

#### Constraints and Limitations

- **SQL Server `SESSION_CONTEXT`:** Session-scoped, not transaction-scoped. A pooled connection that forgets to reset the variable between requests can leak context across users.
- **PostgreSQL `CURRENT_USER`:** Changes with `SET ROLE`; a user who can switch roles may see different rows.
- **Oracle VPD:** Policy functions must be carefully written to avoid performance bottlenecks.

### Annotated Code Examples

#### Example 1: PostgreSQL — User-Specific Visibility with CURRENT_USER

```sql
-- Create a table with an owner column
CREATE TABLE documents (
    document_id SERIAL PRIMARY KEY,
    title TEXT,
    content TEXT,
    owner TEXT NOT NULL
);

-- Insert sample data
INSERT INTO documents (title, content, owner) VALUES
('Alice Report', 'Q1 financials', 'alice'),
('Bob Report', 'Q2 financials', 'bob'),
('Alice Notes', 'Meeting notes', 'alice');

-- Create application role
CREATE ROLE app_user;

-- Enable RLS
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- Create policy: users can only see their own documents
CREATE POLICY documents_owner_policy ON documents
    FOR ALL
    TO app_user
    USING (owner = CURRENT_USER)
    WITH CHECK (owner = CURRENT_USER);

-- Grant SELECT/INSERT/UPDATE/DELETE on the table
GRANT SELECT, INSERT, UPDATE, DELETE ON documents TO app_user;

-- Test: set role to 'alice' and query
SET ROLE alice;
SELECT * FROM documents;
```

**Expected Output:**

```
 document_id |    title     |     content     | owner
-------------+--------------+-----------------+-------
           1 | Alice Report | Q1 financials   | alice
           3 | Alice Notes  | Meeting notes   | alice
```

**Why This Works:** The `USING (owner = CURRENT_USER)` clause filters rows so that only documents where `owner` matches the current user are visible. When `SET ROLE alice` is executed, `CURRENT_USER` becomes `alice`, and only Alice's documents are returned.

#### Example 2: SQL Server — User-Specific Visibility with SESSION_CONTEXT

```sql
-- Create a sales table with a salesperson ID
CREATE TABLE Sales (
    SaleID INT PRIMARY KEY,
    SalesPersonID INT,
    Amount DECIMAL(10,2)
);

INSERT INTO Sales VALUES (1, 1, 1000.00), (2, 2, 2000.00), (3, 1, 1500.00);

-- Create a predicate function
CREATE FUNCTION dbo.fn_salesperson_filter(@SalesPersonID INT)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN
    SELECT 1 AS fn_result
    WHERE @SalesPersonID = CAST(SESSION_CONTEXT(N'UserId') AS INT);

-- Create a security policy
CREATE SECURITY POLICY SalesFilterPolicy
ADD FILTER PREDICATE dbo.fn_salesperson_filter(SalesPersonID) ON dbo.Sales
WITH (STATE = ON);

-- Application sets the session context (e.g., after opening a connection)
EXEC sp_set_session_context @key=N'UserId', @value=1;

-- Query: only sales for SalesPersonID 1
SELECT * FROM Sales;
```

**Expected Output:**

```
SaleID | SalesPersonID | Amount
-------+---------------+---------
1      | 1             | 1000.00
3      | 1             | 1500.00
```

**Why This Works:** The `fn_salesperson_filter` function compares the row's `SalesPersonID` to the value stored in `SESSION_CONTEXT(N'UserId')`. After the application sets `UserId = 1`, only rows where `SalesPersonID = 1` are visible. The `FILTER PREDICATE` silently filters rows without raising an error.

### Real-World Cases

- **Document management:** Users can only see documents they created or own.
- **Sales CRM:** Sales representatives see only their own accounts and opportunities.
- **HR systems:** Managers see only their direct reports' records.
- **Project management:** Team members see only tasks assigned to their projects.

### References

- PostgreSQL: CREATE POLICY — https://www.postgresql.org/docs/current/sql-createpolicy.html
- PostgreSQL: Row Security Policies — https://www.postgresql.org/docs/current/ddl-rowsecurity.html
- SQL Server: Row-Level Security — https://learn.microsoft.com/en-us/sql/relational-databases/security/row-level-security
- SQL Server: SESSION_CONTEXT — https://learn.microsoft.com/en-us/sql/t-sql/functions/session-context-transact-sql


## Core Concept 2: Tenant Isolation

### Definitions

**Core Definition:** Tenant isolation is the row-level security technique that restricts multi-company software setups by locking records to a strict `tenant_id` column value mapping, preventing unauthorized cross-boundary visibility.

**Technical Definition:** Tenant isolation uses RLS policies that filter rows based on a `tenant_id` (or `account_id`, `organization_id`) column. In PostgreSQL, the policy references a session variable (e.g., `current_setting('app.tenant_id')`) that the application sets at the start of each request-scoped transaction. The policy is `USING (tenant_id = current_setting('app.tenant_id')::uuid)`. In SQL Server, the predicate function reads `SESSION_CONTEXT(N'TenantId')`. The key best practice is to set the tenant per transaction using `SET LOCAL` (not `SET`), so the value auto-clears when the transaction ends, preventing context leakage across pooled connections .

**Beginner-Friendly Explanation:** Tenant isolation is like an apartment building where each tenant has their own mailbox. Even though all mailboxes are in the same room (the same database), each tenant can only open their own mailbox (see their own rows). The key that opens the mailbox is the tenant ID, which is set when they log in.

### Purposes

- **To restrict multi-company software setups** by locking records to a strict `tenant_id` column value mapping.
- **To prevent unauthorized cross-boundary visibility** so that one tenant cannot see another tenant's data.
- **To enable shared-database multi-tenancy** where all tenants share the same schema and tables, but data is logically isolated.
- **To simplify application code** by moving tenant filtering from the application layer to the database layer.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL — Tenant Isolation)

```sql
-- Step 1: Enable RLS
ALTER TABLE tenant_data ENABLE ROW LEVEL SECURITY;

-- Step 2: Create a policy using current_setting
CREATE POLICY tenant_isolation_policy ON tenant_data
    FOR ALL
    TO app_role
    USING (tenant_id = current_setting('app.tenant_id', true)::uuid)
    WITH CHECK (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- Step 3: Application sets the tenant ID at the start of each transaction
SET LOCAL app.tenant_id = '00000000-0000-0000-0000-000000000001';

-- Step 4: Query (only tenant's rows are visible)
SELECT * FROM tenant_data;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `current_setting('app.tenant_id', true)` | Reads a custom session variable. The `true` parameter returns NULL instead of an error if the setting does not exist. |
| `SET LOCAL` | Sets the variable for the current transaction only; auto-clears at transaction end. |
| `USING` | Filter for visible rows. |
| `WITH CHECK` | Validation for inserted/updated rows. |

#### Complete General Syntax (SQL Server — Tenant Isolation)

```sql
-- Predicate function using SESSION_CONTEXT for tenant ID
CREATE FUNCTION dbo.fn_tenant_filter(@TenantId INT)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN
    SELECT 1 AS fn_result
    WHERE @TenantId = CAST(SESSION_CONTEXT(N'TenantId') AS INT);

CREATE SECURITY POLICY TenantFilterPolicy
ADD FILTER PREDICATE dbo.fn_tenant_filter(TenantId) ON dbo.Orders,
ADD BLOCK PREDICATE dbo.fn_tenant_filter(TenantId) ON dbo.Orders AFTER INSERT
WITH (STATE = ON);
```

#### Syntax Rules

- **PostgreSQL:** Use `SET LOCAL` (not `SET`) so the tenant ID is transaction-scoped and auto-clears. Use `current_setting('app.tenant_id', true)` to avoid errors if the setting is missing.
- **SQL Server:** `SESSION_CONTEXT()` is session-scoped. For connection pooling, the application must reset the value after each request or use `@read_only = 1` and re-establish the connection.
- **Best practice:** Enable RLS on all tables containing tenant data. Filter on an indexed `tenant_id` column for performance .

#### Constraints and Limitations

- **Connection pooling leakage:** Session-scoped variables (`SESSION_CONTEXT`, `SET` without `LOCAL`) can leak across requests if the connection is reused without resetting.
- **Performance:** Every query on a tenant-isolated table carries the RLS filter. Index the `tenant_id` column to avoid full table scans .
- **Superuser bypass:** Superusers and table owners bypass RLS by default. Use `FORCE ROW LEVEL SECURITY` and a non-superuser application role.

### Annotated Code Examples

#### Example 1: PostgreSQL — Multi-Tenant SaaS with Tenant Isolation

```sql
-- Create a table with tenant_id
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    tenant_id UUID NOT NULL,
    customer_name TEXT,
    amount NUMERIC(10,2)
);

-- Insert data for two tenants
INSERT INTO orders (tenant_id, customer_name, amount) 
VALUES
    ('11111111-1111-1111-1111-111111111111', 'Alice', 100.00),
    ('11111111-1111-1111-1111-111111111111', 'Bob',   200.00),
    ('22222222-2222-2222-2222-222222222222', 'Carol', 300.00);

-- Create application role
CREATE ROLE app_role;

-- Enable RLS and create policy
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON orders
    FOR ALL
    TO app_role
    USING (tenant_id = current_setting('app.tenant_id', true)::uuid)
    WITH CHECK (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- Grant table access
GRANT SELECT, INSERT, UPDATE, DELETE ON orders TO app_role;

-- Test: set tenant and query
SET LOCAL app.tenant_id = '11111111-1111-1111-1111-111111111111';
SELECT * FROM orders;
```

**Expected Output:**

```
 order_id |              tenant_id               | customer_name | amount
----------+--------------------------------------+---------------+--------
        1 | 11111111-1111-1111-1111-111111111111 | Alice         | 100.00
        2 | 11111111-1111-1111-1111-111111111111 | Bob           | 200.00
```

**Why This Works:** The `USING` clause compares the row's `tenant_id` to the value stored in `app.tenant_id`. After `SET LOCAL` sets the tenant to tenant 1, only tenant 1's orders are visible. Carol's order (tenant 2) is automatically filtered out.

#### Example 2: SQL Server — Tenant Isolation with FILTER and BLOCK Predicates

```sql
-- Create table
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    TenantID INT,
    Amount DECIMAL(10,2)
);

INSERT INTO Orders VALUES (1, 1, 100.00), (2, 1, 200.00), (3, 2, 300.00);

-- Predicate function
CREATE FUNCTION dbo.fn_tenant_filter(@TenantID INT)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN
    SELECT 1 AS fn_result
    WHERE @TenantID = CAST(SESSION_CONTEXT(N'TenantId') AS INT);

-- Security policy with FILTER and BLOCK
CREATE SECURITY POLICY TenantPolicy
ADD FILTER PREDICATE dbo.fn_tenant_filter(TenantID) ON dbo.Orders,
ADD BLOCK PREDICATE dbo.fn_tenant_filter(TenantID) ON dbo.Orders AFTER INSERT
WITH (STATE = ON);

-- Set tenant context
EXEC sp_set_session_context @key=N'TenantId', @value=1;

-- Query (only tenant 1 rows)
SELECT * FROM Orders;
```

**Expected Output:**

```
OrderID | TenantID | Amount
--------+----------+--------
1       | 1        | 100.00
2       | 1        | 200.00
```

**Why This Works:** The `FILTER PREDICATE` silently filters rows for read operations. The `BLOCK PREDICATE ... AFTER INSERT` prevents inserting rows that violate the predicate—so a user in tenant 1 cannot insert a row with `TenantID = 2`.

### Real-World Cases

- **SaaS platforms:** Each customer (organization, workspace) is a tenant; their data is isolated at the database level .
- **Marketplace platforms:** Sellers, venues, or merchants each have their own tenant; products, orders, and analytics are scoped per-tenant.
- **Education/healthcare:** Schools, clinics, or departments are tenants; data residency requirements are met via regional routing.
- **Internal tools:** Departments or business units are tenants; strict mode catches accidental cross-department data exposure during development.

### References

- PostgreSQL: Row Security Policies — https://www.postgresql.org/docs/current/ddl-rowsecurity.html
- PostgreSQL: Custom Options (`current_setting`) — https://www.postgresql.org/docs/current/runtime-config-custom.html
- SQL Server: Row-Level Security — https://learn.microsoft.com/en-us/sql/relational-databases/security/row-level-security
- AWS: Multi-Tenant Data Isolation with PostgreSQL — https://docs.aws.amazon.com/whitepapers/latest/multi-tenant-saas-storage-strategies/multi-tenant-data-partitioning.html
- Supabase: RLS Performance — https://supabase.com/docs/guides/database/postgres/row-level-security-performance


## Core Concept 3: Policy-Based Filtering

### Definitions

**Core Definition:** Policy-based filtering is the construction, alteration, and execution of conditional security logic directly on tables using commands like `CREATE POLICY ... FOR SELECT USING (...)` or Oracle Virtual Private Database (VPD) packages.

**Technical Definition:** In PostgreSQL, policies are defined using `CREATE POLICY`, which specifies the table, the command (ALL, SELECT, INSERT, UPDATE, DELETE), the roles to which the policy applies, and the `USING` and `WITH CHECK` expressions . Permissive policies are combined using Boolean `OR`; restrictive policies are combined using Boolean `AND`. If only restrictive policies exist, no records are accessible . In Oracle, Virtual Private Database (VPD) policies are created using the `DBMS_RLS.ADD_POLICY` procedure, which attaches a PL/SQL policy function to a table. The policy function returns a predicate (a `WHERE` clause condition) that is appended to queries .

**Beginner-Friendly Explanation:** Policy-based filtering is like writing the rules for who can see what. You write a rule (policy) that says "if you are in the sales department, you can see sales rows." The database reads the rule and automatically applies it to every query. In PostgreSQL, you write the rule as a `CREATE POLICY` statement. In Oracle, you write a function that returns a `WHERE` clause, and then attach it to the table.

### Purposes

- **To construct, alter, and execute conditional security logic** directly on tables using declarative SQL commands.
- **To support multiple policies** that can be combined (permissive OR, restrictive AND) for complex access rules.
- **To separate concerns** by defining policies for different commands (SELECT, INSERT, UPDATE, DELETE) independently.
- **To use Oracle VPD** for fine-grained access control in Oracle databases.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL — CREATE POLICY)

```sql
CREATE POLICY name ON table_name
    [ AS { PERMISSIVE | RESTRICTIVE } ]
    [ FOR { ALL | SELECT | INSERT | UPDATE | DELETE } ]
    [ TO { role_name | PUBLIC | CURRENT_ROLE | CURRENT_USER | SESSION_USER } [, ...] ]
    [ USING ( using_expression ) ]
    [ WITH CHECK ( check_expression ) ];
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `AS PERMISSIVE` | Policies combined with OR (default). |
| `AS RESTRICTIVE` | Policies combined with AND. |
| `FOR ALL` | Applies to all commands (default). |
| `FOR SELECT` | Applies only to SELECT. |
| `TO role_name` | Applies only to the specified role. |
| `USING` | Filter condition for visible rows. |
| `WITH CHECK` | Validation for inserted/updated rows. |

#### Complete General Syntax (Oracle VPD)

```sql
-- Step 1: Create a policy function
CREATE OR REPLACE FUNCTION policy_func (
    schema_name IN VARCHAR2,
    object_name IN VARCHAR2
) RETURN VARCHAR2 AS
    condition VARCHAR2(200);
BEGIN
    condition := 'deptno = 30';
    IF SYS_CONTEXT('USERENV', 'SESSION_USER') IN ('SCOTT') THEN
        RETURN NULL;
    ELSE
        RETURN condition;
    END IF;
END;

-- Step 2: Attach the policy to the table
BEGIN
    DBMS_RLS.ADD_POLICY(
        object_schema    => 'scott',
        object_name      => 'emp',
        policy_name      => 'emp_policy',
        function_schema  => 'sysadmin_vpd',
        policy_function  => 'emp_policy_func',
        policy_type      => dbms_rls.dynamic
    );
END;
```

#### Syntax Rules

- **PostgreSQL:** Multiple permissive policies are combined with OR; multiple restrictive policies are combined with AND. At least one permissive policy must pass for a row to be accessible .
- **PostgreSQL:** `WITH CHECK` is enforced after BEFORE triggers and before any actual data modifications.
- **Oracle VPD:** The policy function returns a `WHERE` clause condition (e.g., `'deptno = 30'`). Returning `NULL` means no filtering for that user. The `policy_type` can be `DYNAMIC`, `CONTEXT_SENSITIVE`, or `STATIC`.

#### Constraints and Limitations

- **PostgreSQL:** Policies are per-table; one policy name can be used for many different tables with different definitions.
- **Oracle VPD:** Policy functions must be carefully written to avoid performance bottlenecks. The predicate is appended to every query on the table.
- **Performance:** RLS policies add a filter to every query. Index the columns used in policy expressions .

### Annotated Code Examples

#### Example 1: PostgreSQL — Permissive and Restrictive Policies

```sql
-- Create a table
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name TEXT,
    category TEXT,
    price NUMERIC(10,2)
);

INSERT INTO products (name, category, price) VALUES
('Widget', 'Electronics', 100.00),
('Gadget', 'Electronics', 200.00),
('Shirt', 'Clothing', 50.00),
('Jacket', 'Clothing', 150.00);

-- Enable RLS
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

-- Permissive policy: users with 'electronics_access' role can see electronics
CREATE POLICY electronics_policy ON products
    AS PERMISSIVE
    FOR SELECT
    TO electronics_role
    USING (category = 'Electronics');

-- Permissive policy: users with 'clothing_access' role can see clothing
CREATE POLICY clothing_policy ON products
    AS PERMISSIVE
    FOR SELECT
    TO clothing_role
    USING (category = 'Clothing');

-- Restrictive policy: no products over $500 (applies to all)
CREATE POLICY price_limit ON products
    AS RESTRICTIVE
    FOR SELECT
    USING (price <= 500);
```

**Why This Works:** The two permissive policies are combined with OR: a user with `electronics_role` sees electronics rows, and a user with `clothing_role` sees clothing rows. The restrictive policy is combined with AND: no user sees products over $500. This layered approach allows fine-grained control.

#### Example 2: Oracle — VPD Policy Function

```sql
-- Create a VPD policy function that shows only department 30
CREATE OR REPLACE FUNCTION emp_policy_func (
    v_schema IN VARCHAR2,
    v_objname IN VARCHAR2
) RETURN VARCHAR2 AS
    condition VARCHAR2(200);
BEGIN
    condition := 'deptno = 30';
    IF SYS_CONTEXT('USERENV', 'SESSION_USER') IN ('SCOTT') THEN
        RETURN NULL;  -- SCOTT sees all rows
    ELSE
        RETURN condition;  -- Other users see only deptno 30
    END IF;
END emp_policy_func;
/

-- Attach the policy to the EMP table
BEGIN
    DBMS_RLS.ADD_POLICY(
        object_schema    => 'scott',
        object_name      => 'emp',
        policy_name      => 'emp_policy',
        function_schema  => 'sysadmin_vpd',
        policy_function  => 'emp_policy_func',
        policy_type      => dbms_rls.dynamic
    );
END;
/
```

**Expected Output (for non-SCOTT user):**

```
SELECT * FROM scott.emp;
-- Returns only rows where deptno = 30
```

**Why This Works:** The `emp_policy_func` function returns the predicate `'deptno = 30'` for all users except SCOTT. The `DBMS_RLS.ADD_POLICY` procedure attaches this function to the `scott.emp` table. When a non-SCOTT user queries the table, Oracle automatically appends `WHERE deptno = 30` to the query.

### Real-World Cases

- **Department-based access:** Employees see only rows from their department (Oracle VPD example) .
- **Regional filtering:** Sales representatives see only customers in their region.
- **Tiered access:** Premium customers see premium content; free customers see only free content.
- **Compliance:** Redacting rows that do not meet regulatory criteria.

### References

- PostgreSQL: CREATE POLICY — https://www.postgresql.org/docs/current/sql-createpolicy.html
- PostgreSQL: Row Security Policies — https://www.postgresql.org/docs/current/ddl-rowsecurity.html
- Oracle: Using Oracle VPD to Control Data Access — https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/using-oracle-vpd-to-control-data-access.html
- Oracle: DBMS_RLS — https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/DBMS_RLS.html


## Core Concept 4: Multi-Tenant Access Control

### Definitions

**Core Definition:** Multi-tenant access control is the configuration and management of RLS enforcement features—enabling/disabling RLS, managing superuser bypass contexts, and configuring default-deny fallback mechanics—to ensure robust tenant isolation.

**Technical Definition:** RLS is enabled with `ALTER TABLE ... ENABLE ROW LEVEL SECURITY` (PostgreSQL) or `CREATE SECURITY POLICY` (SQL Server). By default, superusers and roles with the `BYPASSRLS` attribute always bypass the row security system. Table owners also bypass RLS unless `FORCE ROW LEVEL SECURITY` is applied . When RLS is enabled and no applicable policies exist, a "default deny" policy is assumed, so no rows are visible or updatable . In connection pooling environments, tenant context must be set per transaction using `SET LOCAL` (PostgreSQL) to prevent context leakage across reused connections .

**Beginner-Friendly Explanation:** Multi-tenant access control is like setting up the rules for a shared office building. You decide which floors (tables) have security doors (RLS enabled). You give the building manager (superuser) a master key, but you also make sure the manager cannot accidentally leave the doors open. You also ensure that if someone forgets their key (no policy applies), the door stays locked (default deny). And you make sure that when people share a desk (connection pooling), they clear their personal settings before the next person sits down.

### Purposes

- **To enable or disable explicit engine enforcement features** using `ALTER TABLE ... ENABLE ROW LEVEL SECURITY`.
- **To manage exceptional superuser bypass contexts** by using `FORCE ROW LEVEL SECURITY` and non-superuser application roles.
- **To configure default-deny fallback mechanics** so that no rows are visible when no policy applies.
- **To prevent context leakage in connection pooling** by using transaction-scoped session variables.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL — Enable RLS and Force)

```sql
-- Enable RLS
ALTER TABLE table_name ENABLE ROW LEVEL SECURITY;

-- Force RLS even for table owners
ALTER TABLE table_name FORCE ROW LEVEL SECURITY;

-- Disable RLS
ALTER TABLE table_name DISABLE ROW LEVEL SECURITY;
```

#### Complete General Syntax (PostgreSQL — Non-Superuser Role)

```sql
-- Create a non-superuser application role
CREATE ROLE app_role WITH LOGIN PASSWORD 'secure_password' NOBYPASSRLS;

-- Grant only necessary table privileges
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA app_schema TO app_role;
```

#### Complete General Syntax (Connection Pooling — Transaction-Scoped Context)

```sql
-- Set tenant context per transaction (auto-clears at transaction end)
BEGIN;
SET LOCAL app.tenant_id = '00000000-0000-0000-0000-000000000001';
SELECT * FROM orders;
COMMIT;
-- After COMMIT, app.tenant_id is reset to NULL
```

#### Syntax Rules

- **PostgreSQL:** `FORCE ROW LEVEL SECURITY` applies RLS to the table owner as well, but superusers and roles with `BYPASSRLS` still bypass it .
- **PostgreSQL:** Use `SET LOCAL` (not `SET`) for transaction-scoped session variables. `SET LOCAL` auto-clears at the end of the transaction, preventing leakage in connection pools.
- **SQL Server:** `CREATE SECURITY POLICY ... WITH (STATE = ON)` enables the policy. `FILTER` and `BLOCK` predicates define the filtering and validation logic.
- **Default-deny:** If RLS is enabled and no policy applies to the querying role, no rows are visible.

#### Constraints and Limitations

- **Superuser bypass:** PostgreSQL superusers and roles with `BYPASSRLS` always bypass RLS. Use a non-superuser application role.
- **Table owner bypass:** Table owners bypass RLS unless `FORCE ROW LEVEL SECURITY` is set.
- **Connection pooling:** Session-scoped variables can leak across requests. Use transaction-scoped variables (`SET LOCAL`) or reset the context on connection checkout .
- **Performance:** RLS adds a filter to every query. Index the policy columns .

### Annotated Code Examples

#### Example 1: PostgreSQL — Force RLS and Non-Superuser Role

```sql
-- Create a table
CREATE TABLE tenant_data (
    id SERIAL PRIMARY KEY,
    tenant_id UUID NOT NULL,
    data TEXT
);

INSERT INTO tenant_data (tenant_id, data) VALUES
('11111111-1111-1111-1111-111111111111', 'Tenant 1 data'),
('22222222-2222-2222-2222-222222222222', 'Tenant 2 data');

-- Enable RLS and force it for the owner
ALTER TABLE tenant_data ENABLE ROW LEVEL SECURITY;
ALTER TABLE tenant_data FORCE ROW LEVEL SECURITY;

-- Create policy
CREATE POLICY tenant_policy ON tenant_data
    FOR ALL
    USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- Create a non-superuser application role
CREATE ROLE app_role WITH LOGIN PASSWORD 'secure_password' NOBYPASSRLS;
GRANT SELECT, INSERT, UPDATE, DELETE ON tenant_data TO app_role;
GRANT USAGE ON SEQUENCE tenant_data_id_seq TO app_role;

-- Test: even the table owner is subject to RLS
SET LOCAL app.tenant_id = '11111111-1111-1111-1111-111111111111';
SELECT * FROM tenant_data;
```

**Expected Output:**

```
 id |              tenant_id               |      data
----+--------------------------------------+----------------
  1 | 11111111-1111-1111-1111-111111111111 | Tenant 1 data
```

**Why This Works:** `FORCE ROW LEVEL SECURITY` ensures that even the table owner is subject to RLS. The `app_role` is created with `NOBYPASSRLS`, so it cannot bypass RLS. The `SET LOCAL` sets the tenant context for the transaction, and only tenant 1's data is visible.

#### Example 2: SQL Server — Default Deny and Filter Predicate

```sql
-- Create a table
CREATE TABLE Documents (
    DocumentID INT PRIMARY KEY,
    OwnerID INT,
    Content NVARCHAR(MAX)
);

INSERT INTO Documents VALUES (1, 1, 'Doc 1'), (2, 2, 'Doc 2');

-- Predicate function
CREATE FUNCTION dbo.fn_doc_filter(@OwnerID INT)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN
    SELECT 1 AS fn_result
    WHERE @OwnerID = CAST(SESSION_CONTEXT(N'UserId') AS INT);

-- Security policy
CREATE SECURITY POLICY DocPolicy
ADD FILTER PREDICATE dbo.fn_doc_filter(OwnerID) ON dbo.Documents
WITH (STATE = ON);

-- If SESSION_CONTEXT is not set, no rows are visible (default deny)
-- Set context for user 1
EXEC sp_set_session_context @key=N'UserId', @value=1;
SELECT * FROM Documents;  -- Only Doc 1 is visible
```

**Expected Output:**

```
DocumentID | OwnerID | Content
-----------+---------+--------
1          | 1       | Doc 1
```

**Why This Works:** If `SESSION_CONTEXT(N'UserId')` is not set, the predicate `@OwnerID = CAST(NULL AS INT)` evaluates to `UNKNOWN`, and no rows are visible (default deny). After setting `UserId = 1`, only documents owned by user 1 are visible.

### Real-World Cases

- **SaaS multi-tenant:** Enable RLS on all tenant tables and use a non-superuser application role.
- **Connection pooling:** Use `SET LOCAL` (PostgreSQL) or reset `SESSION_CONTEXT` (SQL Server) on each connection checkout.
- **Compliance:** Use `FORCE ROW LEVEL SECURITY` to ensure that even DBAs cannot accidentally see tenant data without proper context.
- **Development vs. production:** Enable RLS in development to catch cross-tenant data leaks early (strict mode) .

### References

- PostgreSQL: Row Security Policies — https://www.postgresql.org/docs/current/ddl-rowsecurity.html
- PostgreSQL: ALTER TABLE (FORCE ROW LEVEL SECURITY) — https://www.postgresql.org/docs/current/sql-altertable.html
- SQL Server: Row-Level Security — https://learn.microsoft.com/en-us/sql/relational-databases/security/row-level-security
- SQL Server: CREATE SECURITY POLICY — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-security-policy-transact-sql
- Supabase: RLS Performance — https://supabase.com/docs/guides/database/postgres/row-level-security-performance


## Summary Table: Row-Level Security Across DBMS

| Feature | PostgreSQL | SQL Server | Oracle | MySQL |
|---------|-----------|------------|--------|-------|
| **Native RLS** | Yes (9.5+) | Yes (2016+) | Yes (VPD) | No |
| **Enable Syntax** | `ALTER TABLE ... ENABLE ROW LEVEL SECURITY` | `CREATE SECURITY POLICY` | `DBMS_RLS.ADD_POLICY` | Views + `CURRENT_USER()` |
| **Policy Function** | `CREATE POLICY ... USING/WITH CHECK` | Inline table-valued function | PL/SQL function returning `WHERE` clause | View `WHERE` clause |
| **Session Context** | `current_setting()`, `current_user` | `SESSION_CONTEXT()` | `SYS_CONTEXT()` | `@session_variable` |
| **Superuser Bypass** | Yes (`BYPASSRLS`) | Yes (`sysadmin`) | Yes (`DBA`) | N/A |
| **Force for Owner** | `FORCE ROW LEVEL SECURITY` | N/A (policy always applies) | N/A | N/A |
| **Default Deny** | Yes | Yes | Yes | N/A |
| **Best For** | Multi-tenant SaaS | Enterprise applications | Enterprise Oracle apps | View-based workarounds |


## Final Notes on Deprecated and Unsafe Features

- **PostgreSQL superuser bypass:** Superusers and roles with `BYPASSRLS` always bypass RLS. Never use a superuser account for application connections. Create a non-superuser role with `NOBYPASSRLS` .
- **PostgreSQL table owner bypass:** Table owners bypass RLS by default. Use `FORCE ROW LEVEL SECURITY` to apply RLS to the table owner .
- **Connection pooling leakage:** Session-scoped variables (`SESSION_CONTEXT` in SQL Server, `SET` without `LOCAL` in PostgreSQL) can leak across requests in a connection pool. Use transaction-scoped variables (`SET LOCAL`) or reset the context on every connection checkout .
- **MySQL has no native RLS:** MySQL does not support row-level security. The closest equivalent is updatable views filtered by `CURRENT_USER()` or session variables. This is weaker than native RLS because a session variable is connection-scoped, not transaction-scoped .
- **Oracle VPD policy function performance:** VPD policy functions are appended to every query on the table. Poorly written functions can cause severe performance degradation. Keep policies simple and index the filtered columns .
- **SQL Server BLOCK predicates:** `BLOCK` predicates prevent write operations that violate the predicate. Without a `BLOCK` predicate, a user can insert rows that they cannot see .
- **Version-specific:** PostgreSQL RLS requires 9.5+; `FORCE ROW LEVEL SECURITY` requires 9.5+. SQL Server RLS requires 2016+. Oracle VPD is available in all supported versions. MySQL does not support RLS.
- **Index policy columns:** RLS policies add a filter to every query. Always index the columns used in policy expressions (e.g., `tenant_id`, `user_id`, `owner`) .