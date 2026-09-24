# Trigger Scope & Advanced Applications: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Trigger scope and advanced applications refer to the use of triggers beyond basic data validation—extending into comprehensive auditing, automated data maintenance, complex multi-table business rules, schema-change governance, and server-level session monitoring.

**Technical Definition:** Trigger scope defines whether a trigger operates at the row level, statement level, database level, or server level. Advanced applications leverage triggers for: (1) building immutable audit trails with lineage tracking, (2) automating derived data maintenance and timestamps, (3) enforcing cross-table constraints that declarative constraints cannot express, (4) intercepting and governing DDL operations (CREATE, ALTER, DROP), and (5) monitoring logon/logoff events and session-level security.

**Beginner-Friendly Explanation:** Basic triggers handle simple rules like "check this value." Advanced triggers do bigger jobs: they can record every change to your data (auditing), automatically update related tables, prevent unauthorized schema changes, and even watch who connects to the database.

### Key Characteristics

- **Layered scope:** DML triggers (row/statement), DDL triggers (schema-level), and logon/system triggers (server-level).
- **Platform divergence:** Oracle and SQL Server support the widest range of trigger types; MySQL is limited to row-level DML triggers; PostgreSQL uses event triggers for DDL .
- **Audit trail integrity:** Triggers can create tamper-resistant audit logs that capture old and new values, user identity, and timestamps.
- **Schema governance:** DDL triggers can block or log schema changes, enforcing naming conventions and preventing unauthorized modifications .
- **Security context risk:** Triggers execute with the privileges of the trigger owner or the invoking user, creating potential privilege escalation paths .

### Prerequisites

- **DML trigger fundamentals:** Understanding of `BEFORE`, `AFTER`, `INSTEAD OF`, and row vs. statement granularity.
- **System catalog knowledge:** Familiarity with querying metadata views.
- **Transaction awareness:** Understanding of how triggers participate in transactions.
- **Platform-specific syntax:** Awareness of the target RDBMS's trigger capabilities.

### Related Programming Areas

- **Compliance & Governance:** HIPAA, PCI-DSS, and SOX auditing requirements .
- **Data Security:** Session monitoring, logon auditing, and privilege escalation prevention.
- **Data Engineering:** Automated lineage tracking and derived data maintenance.
- **DevOps:** Schema change management and deployment governance.

### Core Concepts / Features

1. Data Auditing & Lineage
2. Automation & Maintenance
3. Complex Business-Rule Enforcement
4. DDL Triggers
5. Logon & System Event Triggers

---

## 1. Data Auditing & Lineage

### Definitions

**Core Definition:** Data auditing through triggers is the practice of automatically capturing a historical record of all changes (inserts, updates, deletes) made to a table, including the old and new values, who made the change, and when.

**Technical Definition:** An audit trigger fires on DML events and writes a record to a separate audit table containing the affected row's primary key, the operation type, the old and new column values (often as JSON), the executing user, and the timestamp. This creates an immutable lineage trail that can be queried for compliance, forensics, and change tracking .

**Beginner-Friendly Explanation:** An audit trigger is like a security camera for your data. Every time someone changes a row, the trigger takes a "before" and "after" picture, records who did it and when, and stores it in a separate audit table.

### Purposes

- To create an immutable record of all data changes for compliance (HIPAA, PCI-DSS, SOX) .
- To track data lineage—who changed what, when, and from where.
- To support forensic analysis and data recovery.
- To detect unauthorized or anomalous data modifications.
- To satisfy audit requirements without application-layer boilerplate code .

### Syntax Rules and Structure

**Audit Table Schema:**

```sql
CREATE TABLE audit_log (
    id              SERIAL PRIMARY KEY,
    table_name      VARCHAR(100),
    row_id          VARCHAR(36),
    event_type      VARCHAR(10),  -- INSERT, UPDATE, DELETE
    old_values      JSON,
    new_values      JSON,
    changed_columns JSON,
    user_id         VARCHAR(100),
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**PostgreSQL Audit Function:**

```sql
CREATE OR REPLACE FUNCTION log_student_changes()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO Student_Audit_Log(student_id, action)
        VALUES (NEW.id, 'INSERT');
        RETURN NEW;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO Student_Audit_Log(student_id, action)
        VALUES (NEW.id, 'UPDATE');
        RETURN NEW;
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO Student_Audit_Log(student_id, action)
        VALUES (OLD.id, 'DELETE');
        RETURN OLD;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER student_audit_trigger
    AFTER INSERT OR UPDATE OR DELETE ON Student
    FOR EACH ROW EXECUTE FUNCTION log_student_changes();
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `TG_OP` (PostgreSQL) | Identifies the operation type (INSERT/UPDATE/DELETE)  |
| `NEW` / `OLD` | Access new and old row states |
| Audit table | Separate table storing change history |
| `JSON` columns | Store flexible old/new value snapshots  |

**Syntax Rules:**

- PostgreSQL requires a trigger function (`RETURNS TRIGGER`) separate from the trigger definition .
- SQL Server uses `inserted` and `deleted` pseudo-tables .
- MySQL uses `NEW` and `OLD` within `FOR EACH ROW` triggers.
- Audit tables should not have foreign keys to the audited table, to avoid integrity issues on deletes .
- The `DEFINER` clause (MySQL) specifies whose privileges are used for the trigger's operations .

**Constraints and Limitations:**

- Audit triggers add overhead to every DML operation.
- High-volume systems may need asynchronous audit logging.
- MySQL does not fire triggers on cascading foreign key actions .
- `TRUNCATE` does not fire DML triggers in most systems .

### Annotated Complete Code Examples

**Example 1: PostgreSQL Audit Trigger with OLD/NEW**

```sql
-- Audit function capturing operation type and row ID
CREATE OR REPLACE FUNCTION log_student_changes()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO Student_Audit_Log(student_id, action)
        VALUES (NEW.id, 'INSERT');
        RETURN NEW;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO Student_Audit_Log(student_id, action)
        VALUES (NEW.id, 'UPDATE');
        RETURN NEW;
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO Student_Audit_Log(student_id, action)
        VALUES (OLD.id, 'DELETE');
        RETURN OLD;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER student_audit_trigger
    AFTER INSERT OR UPDATE OR DELETE ON Student
    FOR EACH ROW EXECUTE FUNCTION log_student_changes();
```

**Why this works:** The `TG_OP` variable tells the function which operation fired the trigger. `NEW` provides the inserted/updated row; `OLD` provides the deleted/updated row. The function returns `NEW` or `OLD` to allow the operation to proceed .

**Example 2: SQL Server Audit with inserted/deleted Tables**

```sql
CREATE TRIGGER trg_SalaryAudit ON Employees
AFTER UPDATE
AS
BEGIN
    INSERT INTO SalaryAudit (EmployeeID, OldSalary, NewSalary, ChangeDate)
    SELECT i.EmployeeID, d.Salary, i.Salary, GETDATE()
    FROM inserted i
    INNER JOIN deleted d ON i.EmployeeID = d.EmployeeID
    WHERE i.Salary <> d.Salary;
END;
```

**Why this works:** The `inserted` table holds new values, and `deleted` holds old values. Joining on the primary key and filtering for actual changes ensures only meaningful updates are logged .

### Real-World Cases

**Case 1: HIPAA Compliance**

A healthcare database uses audit triggers on all patient tables. Any access or modification to patient records is logged with user identity, timestamp, and old/new values, satisfying HIPAA audit requirements .

**Case 2: Financial Transaction Lineage**

A banking system uses triggers to capture every balance change, creating a complete lineage from original deposit to current balance, enabling forensic reconstruction of any account state .

### References

- Baeldung — What Is a Trigger in SQL? - https://www.baeldung.com/sql/triggers
- Laravel Audit Log Package — Audit Table Schema - https://packagist.org/packages/soullessthread/laravel-audit-log
- HIPAA Auditing for Oracle Database Security - https://dl.acm.org/doi/abs/10.5555/940377

---

## 2. Automation & Maintenance

### Definitions

**Core Definition:** Automation through triggers involves using triggers to automatically maintain derived data, set default values, update timestamps, and propagate changes across related tables without application intervention.

**Technical Definition:** Automation triggers perform maintenance tasks such as: setting `created_at` or `updated_at` timestamps on insert/update, computing and storing derived/aggregated values (e.g., order totals), maintaining denormalized columns, and cascading non-standard updates to related tables.

**Beginner-Friendly Explanation:** Automation triggers are the database's "autopilot." When data changes, they automatically fix up related information—like stamping the current time, recalculating a total, or updating a summary table.

### Purposes

- To automatically set audit timestamps (`created_at`, `updated_at`) without application code.
- To maintain denormalized or derived data (e.g., order totals, inventory counts).
- To propagate changes to related tables (e.g., update `last_modified` when a child record changes).
- To auto-populate default values based on other column values.
- To reduce application boilerplate code.

### Syntax Rules and Structure

**MySQL: Automatic Timestamp on Insert**

```sql
CREATE TRIGGER set_hire_date
BEFORE INSERT ON employees
FOR EACH ROW
SET NEW.hire_date = IFNULL(NEW.hire_date, CURDATE());
```

**MySQL: Derived Value Maintenance**

```sql
CREATE TRIGGER update_order_total
AFTER INSERT ON order_items
FOR EACH ROW
BEGIN
    UPDATE orders
    SET total = (SELECT SUM(quantity * unit_price)
                 FROM order_items
                 WHERE order_id = NEW.order_id)
    WHERE order_id = NEW.order_id;
END;
```

**Component Breakdown:**

| Automation Task | Trigger Type | Mechanism |
|----------------|-------------|-----------|
| Timestamp setting | `BEFORE INSERT/UPDATE` | `SET NEW.column = NOW()` |
| Derived value | `AFTER INSERT/UPDATE/DELETE` | `UPDATE related_table` |
| Default value | `BEFORE INSERT` | `SET NEW.column = expression` |
| Cascade update | `AFTER UPDATE` | `UPDATE child_table` |

**Syntax Rules:**

- `BEFORE` triggers can modify `NEW` values; `AFTER` triggers cannot.
- MySQL triggers cannot modify the table that fired them (mutating table restriction) .
- PostgreSQL uses trigger functions; the return value determines whether the operation proceeds.
- SQL Server uses `inserted`/`deleted` tables for access.

**Constraints and Limitations:**

- Mutating table errors occur when a row-level trigger queries or modifies the table that fired it .
- Recursive trigger firing (trigger A updates table B, which has a trigger that updates table A) can cause infinite loops.
- Triggers add overhead to every DML operation; automate only high-value tasks.

### Annotated Complete Code Examples

**Example 1: Automatic Timestamps (MySQL)**

```sql
CREATE TRIGGER before_employee_insert
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    SET NEW.created_at = NOW();
    SET NEW.updated_at = NOW();
END;

CREATE TRIGGER before_employee_update
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    SET NEW.updated_at = NOW();
END;
```

**Why this works:** The `BEFORE INSERT` trigger sets both timestamps when a new employee is added. The `BEFORE UPDATE` trigger refreshes only `updated_at`, preserving the original `created_at` .

### Real-World Cases

**Case 1: E-Commerce Order Total**

A trigger on `order_items` recalculates the `orders.total` column whenever a line item is added, updated, or deleted, ensuring the order total is always accurate without application logic.

**Case 2: Inventory Denormalization**

A trigger on `shipments` updates a denormalized `inventory.on_hand` column whenever a shipment is recorded, avoiding expensive aggregate queries at read time.

### References

- Oracle and MySQL Triggers — AWS DMS Migration Playbook - https://docs.aws.amazon.com/dms/latest/oracle-to-aurora-mysql-migration-playbook/chap-oracle-aurora-mysql.tables.triggers.md
- MySQL Reference Manual — Trigger Syntax - http://downloads.mysql.com/docs/refman-5.1-ja.pdf

---

## 3. Complex Business-Rule Enforcement

### Definitions

**Core Definition:** Complex business-rule enforcement through triggers involves implementing constraints that span multiple tables or require procedural logic that standard declarative constraints (CHECK, FOREIGN KEY) cannot express.

**Technical Definition:** Triggers can enforce rules such as: ensuring a column value is immutable after insert, enforcing exclusive associations across multiple tables, preventing deletes when referenced records exist, and validating cross-table conditions that depend on the current state of multiple tables .

**Beginner-Friendly Explanation:** Some business rules are too complicated for simple constraints. For example: "An employee's salary can only increase, never decrease" or "A customer can have either a credit account or a debit account, but not both." Triggers can enforce these rules.

### Purposes

- To enforce immutable columns (values that cannot change after insert).
- To enforce exclusive associations (a record in table A can reference either table B or table C, but not both) .
- To prevent deletion of records that are still referenced.
- To validate multi-row conditions that depend on aggregate state.
- To enforce workflow state transitions (e.g., `pending` → `approved` but never `approved` → `pending`).

### Syntax Rules and Structure

**Immutable Column (MySQL/PostgreSQL):**

```sql
CREATE TRIGGER prevent_id_change
BEFORE UPDATE ON brain
FOR EACH ROW
BEGIN
    IF OLD.person_id <> NEW.person_id THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'person_id is immutable';
    END IF;
END;
```

**SQL Server Immutable Column (Statement-Level Workaround):**

```sql
CREATE TRIGGER prevent_id_change ON brain
AFTER UPDATE
AS
BEGIN
    IF UPDATE(person_id)
    BEGIN
        IF EXISTS (
            SELECT 1 FROM inserted i
            JOIN deleted d ON i.id = d.id
            WHERE i.person_id <> d.person_id
        )
        BEGIN
            RAISERROR('person_id is immutable', 16, 1);
            ROLLBACK TRANSACTION;
        END
    END
END;
```

**Component Breakdown:**

| Rule Type | Mechanism | Platform |
|-----------|-----------|----------|
| Immutable column | `BEFORE UPDATE` row trigger; compare `OLD` vs `NEW` | MySQL, PostgreSQL  |
| Exclusive association | `BEFORE INSERT` on all tables in the set | MySQL, PostgreSQL  |
| Immutable column (SQL Server) | `AFTER UPDATE`; query `inserted`/`deleted` | SQL Server  |

**Syntax Rules:**

- MySQL and PostgreSQL support `BEFORE UPDATE` row-level triggers for direct comparison.
- SQL Server lacks row-level triggers; uses `AFTER` triggers with `inserted`/`deleted` tables .
- `SIGNAL` (MySQL) raises a custom error and prevents the operation.
- `RAISERROR` + `ROLLBACK` (SQL Server) aborts the operation.

**Constraints and Limitations:**

- SQL Server's statement-level approach makes detecting primary key changes impossible .
- Mutating table errors limit what row-level triggers can query .
- Complex rules may require compound triggers (Oracle) or multiple triggers.

### Annotated Complete Code Examples

**Example 1: Exclusive Association (MySQL)**

```sql
-- Ensure a person is either ALIVE or DECEASED, not both
CREATE TRIGGER check_alive_exclusive
BEFORE INSERT ON alive
FOR EACH ROW
BEGIN
    IF EXISTS (SELECT 1 FROM deceased WHERE person_id = NEW.person_id) THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'Person is already deceased';
    END IF;
END;

CREATE TRIGGER check_deceased_exclusive
BEFORE INSERT ON deceased
FOR EACH ROW
BEGIN
    IF EXISTS (SELECT 1 FROM alive WHERE person_id = NEW.person_id) THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'Person is already alive';
    END IF;
END;
```

**Why this works:** Each `BEFORE INSERT` trigger checks the other table for a matching record. If found, `SIGNAL` raises an error and the insert is rejected, enforcing the exclusive association .

### Real-World Cases

**Case 1: Workflow State Machine**

A trigger on an `orders` table ensures that `status` can only move forward (`pending` → `shipped` → `delivered`), preventing accidental reversals.

**Case 2: Multi-Tenant Isolation**

A trigger validates that every inserted row has a `tenant_id` matching the application's current tenant context, preventing cross-tenant data leakage.

### References

- Annals of Computer Science — Trigger-Based Constraint Enforcement - https://annals-csis.org/proceedings/2025/pliks/communication.pdf

---

## 4. Data Definition Language (DDL) Triggers

### Definitions

**Core Definition:** DDL triggers are special triggers that fire in response to Data Definition Language events—statements that modify database schema objects, such as `CREATE`, `ALTER`, `DROP`, `GRANT`, `DENY`, `REVOKE`, and `UPDATE STATISTICS`.

**Technical Definition:** DDL triggers fire after the DDL statement executes (not as `INSTEAD OF`) and can be scoped to the database or the server. They are used for preventing schema changes, auditing schema modifications, and enforcing naming conventions. Information about the event is captured via the `EVENTDATA()` function (SQL Server) or `pg_event_trigger_ddl_commands()` (PostgreSQL) .

**Beginner-Friendly Explanation:** DDL triggers watch for changes to the database's structure itself—like creating or dropping tables. They can block dangerous operations or log every schema change.

### Purposes

- To prevent unauthorized or risky schema changes (e.g., `DROP TABLE`) .
- To audit all DDL operations for compliance and change tracking .
- To enforce naming conventions and schema standards.
- To implement a "DDL recycle bin" for accidental drops .
- To synchronize schema changes across databases.

### Syntax Rules and Structure

**SQL Server DDL Trigger:**

```sql
CREATE TRIGGER safety
ON DATABASE
FOR DROP_TABLE, ALTER_TABLE
AS
    PRINT 'You must disable trigger "safety" to drop or alter tables!';
    ROLLBACK;
```

**PostgreSQL Event Trigger:**

```sql
CREATE OR REPLACE FUNCTION log_ddl_command()
RETURNS event_trigger AS $$
DECLARE cmd TEXT;
BEGIN
    SELECT query INTO cmd FROM pg_stat_activity WHERE pid = pg_backend_pid();
    INSERT INTO ddl_log (event, tag, command)
    VALUES (TG_EVENT, TG_TAG, cmd);
END;
$$ LANGUAGE plpgsql;

CREATE EVENT TRIGGER ddl_recycle_trigger
    ON ddl_command_end
    EXECUTE FUNCTION log_ddl_command();
```

**Oracle DDL Trigger:**

```sql
CREATE OR REPLACE TRIGGER prevent_drop_trigger
BEFORE DROP ON HR.SCHEMA
BEGIN
    RAISE_APPLICATION_ERROR(-20000, 'Cannot drop object');
END;
```

**Component Breakdown:**

| Platform | Event | Mechanism |
|----------|-------|-----------|
| SQL Server | `ON DATABASE` / `ON ALL SERVER` | `FOR event_type`  |
| PostgreSQL | `ddl_command_start`, `ddl_command_end`, `sql_drop` | `CREATE EVENT TRIGGER`  |
| Oracle | Schema/database level | `BEFORE` / `AFTER` DDL events  |

**Syntax Rules:**

- SQL Server: `EVENTDATA()` returns XML with event details .
- PostgreSQL: `TG_EVENT` and `TG_TAG` provide event info; `pg_event_trigger_ddl_commands()` returns affected objects .
- Oracle: `SYS.DICTIONARY_OBJ_NAME`, `SYS.DICTIONARY_OBJ_TYPE`, `SYS.SYSEVENT` provide event context .
- DDL triggers fire only after the DDL statement executes (SQL Server) .

**Constraints and Limitations:**

- SQL Server DDL triggers cannot be `INSTEAD OF` .
- PostgreSQL event triggers do not fire for DDL on shared objects (databases, roles) .
- Oracle DDL triggers can cause `ORA-04021` errors if not managed during patches .

### Annotated Complete Code Examples

**Example 1: SQL Server DDL Trigger Preventing Drops**

```sql
CREATE TRIGGER safety
ON DATABASE
FOR DROP_TABLE, ALTER_TABLE
AS
BEGIN
    PRINT 'You must disable trigger "safety" to drop or alter tables!';
    ROLLBACK;
END;
```

**Why this works:** The trigger fires after any `DROP TABLE` or `ALTER TABLE` statement. It prints a message and rolls back the operation, preventing the schema change .

**Example 2: PostgreSQL DDL Recycle Bin**

```sql
CREATE SCHEMA ddl_recycle;
CREATE TABLE ddl_recycle.ddl_log (
    id SERIAL PRIMARY KEY,
    event_time TIMESTAMPTZ DEFAULT NOW(),
    username TEXT,
    event TEXT,
    tag TEXT,
    command TEXT,
    is_dropped BOOLEAN DEFAULT FALSE
);

CREATE OR REPLACE FUNCTION ddl_recycle.log_ddl_command()
RETURNS event_trigger AS $$
DECLARE cmd TEXT;
BEGIN
    SELECT query INTO cmd FROM pg_stat_activity WHERE pid = pg_backend_pid();
    INSERT INTO ddl_recycle.ddl_log (username, event, tag, command)
    VALUES (current_user, TG_EVENT, TG_TAG, cmd);
END;
$$ LANGUAGE plpgsql;

CREATE EVENT TRIGGER ddl_recycle_trigger
    ON ddl_command_end
    EXECUTE FUNCTION ddl_recycle.log_ddl_command();
```

**Why this works:** The event trigger fires after every DDL command, capturing the command text, event type, and tag into the `ddl_log` table. This creates a DDL audit trail that can be used to recover from accidental drops .

### Real-World Cases

**Case 1: Schema Change Governance**

An enterprise uses DDL triggers to log all schema changes to a central audit table. A compliance team reviews the log weekly to ensure changes follow change-management procedures .

**Case 2: Preventing Accidental Drops**

A DBA creates a `BEFORE DROP` trigger that blocks any `DROP TABLE` outside of a maintenance window, preventing accidental data loss .

### References

- Microsoft Learn — DDL Triggers - https://learn.microsoft.com/ms-my/sql/relational-databases/triggers/ddl-triggers
- PostgreSQL Documentation — Event Trigger Behavior - https://www.postgresql.org/docs/16/event-trigger-definition.html
- Huawei Cloud — Using Event Triggers for DDL Recycle Bin - https://support.huaweicloud.com/intl/en-us/bestpractice-rds-pg/rds_pg_03_0001.html
- Oracle Triggers — System and User Events - https://artur.ii.uph.edu.pl/2021/miobd/wyklad7.pdf

---

## 5. Logon & System Event Triggers

### Definitions

**Core Definition:** Logon and system event triggers fire in response to server-level events such as user logon, logoff, database startup, shutdown, and server errors.

**Technical Definition:** Oracle supports `AFTER LOGON`, `BEFORE LOGOFF`, `AFTER SERVERERROR`, `AFTER STARTUP`, and `BEFORE SHUTDOWN` triggers at the database or schema level. These triggers are used for session auditing, connection rule enforcement, and server-level security monitoring. SQL Server supports logon triggers (`ON ALL SERVER FOR LOGON`) for auditing and controlling login events .

**Beginner-Friendly Explanation:** Logon triggers are like security guards at the database's front door. Every time someone logs in, the guard checks who they are, records the time, and can even block the connection if it violates a rule.

### Purposes

- To audit all user logon and logoff events .
- To enforce server-side connection rules (e.g., block logins outside business hours).
- To monitor and log server errors for diagnostics.
- To track database startup and shutdown for maintenance auditing.
- To implement session-level security policies.

### Syntax Rules and Structure

**Oracle Logon Trigger:**

```sql
CREATE OR REPLACE TRIGGER log_connections
AFTER LOGON ON DATABASE
BEGIN
    INSERT INTO connection_log (user_id, log_date, action)
    VALUES (USER, SYSDATE, 'LOGON');
END;
```

**SQL Server Logon Trigger:**

```sql
CREATE TRIGGER trg_TrackLoginManagement
ON ALL SERVER
FOR LOGON
AS
BEGIN
    INSERT INTO msdb..TrackLoginManagement (EventType, EventTime, LoginName)
    SELECT 
        EVENTDATA().value('(/EVENT_INSTANCE/EventType)[1]', 'varchar(100)'),
        EVENTDATA().value('(/EVENT_INSTANCE/PostTime)[1]', 'datetime'),
        EVENTDATA().value('(/EVENT_INSTANCE/LoginName)[1]', 'varchar(100)');
END;
```

**Component Breakdown:**

| Platform | Event | Scope |
|----------|-------|-------|
| Oracle | `LOGON`, `LOGOFF` | Database or schema  |
| Oracle | `SERVERERROR`, `STARTUP`, `SHUTDOWN` | Database  |
| SQL Server | `LOGON` | Server  |

**Syntax Rules:**

- Oracle: `AFTER LOGON ON DATABASE` or `AFTER LOGON ON SCHEMA` .
- Oracle: `BEFORE LOGOFF` fires before the session terminates.
- SQL Server: `ON ALL SERVER FOR LOGON` creates a server-scoped logon trigger .
- Oracle: `SYS.LOGIN_USER`, `SYS.SYSEVENT`, `SYS.DATABASE_NAME` provide event context .

**Constraints and Limitations:**

- MySQL does not support logon or system event triggers .
- PostgreSQL does not support logon triggers directly; connection auditing is done via `log_connections` setting.
- Logon triggers can block all logins if they fail, creating a denial-of-service risk.
- Oracle logon triggers fire for every session, adding overhead.

### Annotated Complete Code Examples

**Example 1: Oracle Logon Audit Trigger**

```sql
CREATE OR REPLACE TRIGGER log_connections
AFTER LOGON ON DATABASE
BEGIN
    INSERT INTO connection_log (user_id, log_date, action)
    VALUES (USER, SYSDATE, 'LOGON');
END;
```

**Why this works:** The `AFTER LOGON ON DATABASE` trigger fires immediately after a successful login. It records the username, timestamp, and action in the `connection_log` table, creating a session audit trail .

**Example 2: SQL Server Logon Trigger with Email Alert**

```sql
CREATE TRIGGER trg_TrackLoginManagement
ON ALL SERVER
FOR LOGON
AS
BEGIN
    DECLARE @EventType VARCHAR(100), @LoginName VARCHAR(100);
    SET @EventType = EVENTDATA().value('(/EVENT_INSTANCE/EventType)[1]', 'varchar(100)');
    SET @LoginName = EVENTDATA().value('(/EVENT_INSTANCE/LoginName)[1]', 'varchar(100)');
    
    INSERT INTO msdb..TrackLoginManagement (EventType, EventTime, LoginName)
    VALUES (@EventType, GETDATE(), @LoginName);
    
    -- Send email alert for DDL login events
    EXEC msdb.dbo.sp_send_dbmail
        @recipients = 'dba@company.com',
        @subject = 'Login Event: ' + @EventType,
        @body = 'Login by ' + @LoginName + ' at ' + CONVERT(VARCHAR, GETDATE());
END;
```

**Why this works:** The logon trigger captures the event type and login name from `EVENTDATA()`, logs them to a tracking table, and sends an email alert to the DBA team .

### Real-World Cases

**Case 1: After-Hours Access Monitoring**

An Oracle logon trigger checks the current time and logs a warning if a non-admin user logs in outside business hours, flagging potential security incidents.

**Case 2: Failed Login Analysis**

A SQL Server logon trigger records all login attempts (successful and failed) to a table, enabling security teams to detect brute-force attacks.

### References

- Oracle Triggers — System and User Events - https://artur.ii.uph.edu.pl/2021/miobd/wykład7.pdf
- Microsoft Learn — DDL Triggers and LOGON Triggers - https://learn.microsoft.com/ja-jp/archive/blogs/vipulshah/ddl-triggers-and-logon-triggers
- HIPAA Auditing for Oracle Database Security - https://dl.acm.org/doi/abs/10.5555/940377

---

## Summary Table: Advanced Trigger Applications by Platform

| Application | SQL Server | MySQL | PostgreSQL | Oracle |
|-------------|-----------|-------|------------|--------|
| **Data Auditing** | `AFTER` + `inserted`/`deleted` | `AFTER` + `OLD`/`NEW` | `AFTER` + `TG_OP` | `AFTER` + `:OLD`/`:NEW` |
| **Automation** | `AFTER` (statement) | `BEFORE`/`AFTER` (row) | `BEFORE`/`AFTER` | `BEFORE`/`AFTER` |
| **Complex Rules** | Statement-level workarounds | Row-level `BEFORE` | Row-level `BEFORE` | Compound triggers |
| **DDL Triggers** | ✅ (`ON DATABASE`/`ON ALL SERVER`) | ❌ | ✅ (Event Triggers) | ✅ (Schema/DB level) |
| **Logon Triggers** | ✅ (`ON ALL SERVER FOR LOGON`) | ❌ | ❌ | ✅ (`AFTER LOGON`) |
| **System Events** | Limited | ❌ | ❌ | ✅ (`SERVERERROR`, `STARTUP`) |

---

## References

- Baeldung — What Is a Trigger in SQL? - https://www.baeldung.com/sql/triggers
- Microsoft Learn — DDL Triggers - https://learn.microsoft.com/ms-my/sql/relational-databases/triggers/ddl-triggers
- PostgreSQL Documentation — Event Trigger Behavior - https://www.postgresql.org/docs/16/event-trigger-definition.html
- Huawei Cloud — Using Event Triggers for DDL Recycle Bin - https://support.huaweicloud.com/intl/en-us/bestpractice-rds-pg/rds_pg_03_0001.html
- Oracle Triggers — System and User Events - https://artur.ii.uph.edu.pl/2021/miobd/wyklad7.pdf
- Microsoft Learn — DDL Triggers and LOGON Triggers - https://learn.microsoft.com/ja-jp/archive/blogs/vipulshah/ddl-triggers-and-logon-triggers
- HIPAA Auditing for Oracle Database Security - https://dl.acm.org/doi/abs/10.5555/940377
- Annals of Computer Science — Trigger-Based Constraint Enforcement - https://annals-csis.org/proceedings/2025/pliks/communication.pdf
- AWS DMS — Oracle and MySQL Triggers - https://docs.aws.amazon.com/dms/latest/oracle-to-aurora-mysql-migration-playbook/chap-oracle-aurora-mysql.tables.triggers.md
- Laravel Audit Log Package — Audit Table Schema - https://packagist.org/packages/soullessthread/laravel-audit-log
- Redgate — msdb Attack Paths: Securing SQL Server Triggers - https://www.red-gate.com/simple-talk/databases/sql-server/msdb-attack-paths-how-to-secure-sql-server-agent-triggers-and-restores/
- MySQL Reference Manual — Trigger Syntax - http://downloads.mysql.com/docs/refman-5.1-ja.pdf
- Oracle GoldenGate — DDL Trigger Management - https://docs.oracle.com/goldengate/c1230/gg-winux/GGODB/GGODB.pdf
- Cleverence — SQL CREATE TRIGGER Guide - https://www.cleverence.com/articles/oracle-documentation/create-trigger-sql-guide-4827/