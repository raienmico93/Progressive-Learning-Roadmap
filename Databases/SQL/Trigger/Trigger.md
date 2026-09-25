# Trigger Fundamentals & Architecture: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A database trigger is a special type of stored procedure that automatically executes (fires) in response to specific events on a table or view, such as an `INSERT`, `UPDATE`, or `DELETE` operation.

**Technical Definition:** A trigger is a named database object associated with a table that activates when a particular event occurs for that table . Triggers are defined with `CREATE TRIGGER` and specify the triggering event (`INSERT`, `UPDATE`, `DELETE`), the timing (`BEFORE`, `AFTER`, or `INSTEAD OF`), the granularity (row-level or statement-level), and the body of code to execute. The trigger and the statement that fires it are treated as a single transaction, which can be rolled back from within the trigger .

**Beginner-Friendly Explanation:** A trigger is like an automatic "if this, then that" rule for your database. When someone inserts, updates, or deletes data, the trigger wakes up and does something—like checking a rule, logging the change, or updating another table. You don't call triggers directly; the database calls them for you.

### Key Characteristics

- **Event-driven:** Triggers activate automatically in response to DML events (`INSERT`, `UPDATE`, `DELETE`) .
- **Transaction-integrated:** The trigger and its triggering statement execute as a single transaction .
- **Cannot be called directly:** Unlike stored procedures, triggers are invoked implicitly by the database engine.
- **Platform-dependent:** Trigger capabilities and syntax vary significantly across RDBMSs (SQL Server, MySQL, PostgreSQL, Oracle) .
- **Row-level vs. statement-level:** Some platforms fire once per affected row, others once per statement .
- **Transition data access:** Triggers can access the old and new states of modified data through special variables or tables .

### Prerequisites

- **SQL proficiency:** Solid understanding of `SELECT`, `INSERT`, `UPDATE`, and `DELETE`.
- **Stored procedure concepts:** Familiarity with procedural SQL constructs.
- **Transaction awareness:** Understanding of `COMMIT` and `ROLLBACK`.
- **Platform knowledge:** Awareness of the target RDBMS's trigger syntax.

### Related Programming Areas

- **Data Integrity:** Enforcing complex business rules that constraints cannot express.
- **Auditing:** Logging changes to sensitive tables.
- **Data Synchronization:** Cascading changes to related tables.
- **Application Logic:** Encapsulating automatic business rules.

### Core Concepts / Features

1. Trigger Definition & Mechanics
2. Trigger Granularity
3. Transition Variables & Tables
4. Execution Timing

---

## 1. Trigger Definition & Mechanics

### Definitions

**Core Definition:** Trigger definition involves creating a named trigger object with `CREATE TRIGGER`, specifying its event binding (which DML operation fires it) and its body (what code to execute).

**Technical Definition:** A DML trigger is created with `CREATE TRIGGER trigger_name { BEFORE | AFTER | INSTEAD OF } { INSERT | UPDATE | DELETE } ON table_name FOR EACH ROW trigger_body`. The trigger is compiled and stored in the database catalog. When a matching DML event occurs, the database engine automatically fires the trigger, executing its body within the same transaction as the triggering statement .

**Beginner-Friendly Explanation:** Creating a trigger is like setting up a motion sensor. You say "when this happens on this table, run this code." Once created, the sensor is always watching.

### Purposes

- To enforce business rules that are too complex for standard constraints .
- To automatically log changes to audit tables.
- To cascade changes through related tables.
- To validate data before or after it is committed.
- To maintain derived or denormalized data.

### Syntax Rules and Structure

**Complete General Syntax (MySQL):**

```sql
CREATE TRIGGER trigger_name
{ BEFORE | AFTER } { INSERT | UPDATE | DELETE }
ON table_name
FOR EACH ROW
trigger_body;
```

**Complete General Syntax (SQL Server):**

```sql
CREATE TRIGGER trigger_name
ON table_name
{ FOR | AFTER | INSTEAD OF } { INSERT | UPDATE | DELETE }
AS
sql_statements;
```

**Complete General Syntax (PostgreSQL):**

```sql
CREATE TRIGGER trigger_name
{ BEFORE | AFTER | INSTEAD OF } { INSERT | UPDATE | DELETE }
ON table_name
[ FOR EACH ROW | FOR EACH STATEMENT ]
EXECUTE FUNCTION trigger_function();
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `CREATE TRIGGER` | Initiates trigger creation |
| `trigger_name` | Unique name within the schema  |
| `BEFORE / AFTER / INSTEAD OF` | Execution timing |
| `INSERT / UPDATE / DELETE` | Event binding |
| `ON table_name` | The table the trigger is associated with |
| `FOR EACH ROW` | Row-level granularity (MySQL, PostgreSQL, Oracle) |
| `AS` / `trigger_body` | The code to execute |

**Syntax Rules:**

- MySQL triggers only support `BEFORE` and `AFTER`; no `INSTEAD OF` .
- SQL Server does not support `BEFORE` triggers; uses `AFTER` and `INSTEAD OF` .
- PostgreSQL requires a separate trigger function (`EXECUTE FUNCTION`) .
- Multiple triggers of the same type can exist on a table in most platforms .
- Trigger names must be unique within a schema .

**Constraints and Limitations:**

- MySQL cannot associate triggers with views .
- SQL Server `INSTEAD OF DELETE/UPDATE` triggers cannot be defined on tables with cascading foreign keys .
- Triggers add overhead to every DML operation.
- Debugging triggers can be difficult because they fire implicitly.

### Annotated Complete Code Examples

Setup: Creating the Base Tables: To run these examples, first create these two simple tables: an employees table and an audit_log table.sql-- Create the main employees table
```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(50),
    salary DECIMAL(10, 2),
    department VARCHAR(50)
);

-- Create a table to track changes automatically
CREATE TABLE audit_log (
    log_id INT AUTO_INCREMENT PRIMARY KEY,
    action_type VARCHAR(50),
    description VARCHAR(255),
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Example 1: AFTER INSERT Trigger (Auditing New Rows):**
This trigger automatically creates a record in the audit_log table whenever a new employee is added.The Code

```sql 
DELIMITER //

CREATE TRIGGER after_employee_insert
AFTER INSERT ON employees
FOR EACH ROW
BEGIN
    -- 'NEW' refers to the row that was just inserted
    INSERT INTO audit_log (action_type, description)
    VALUES ('INSERT', CONCAT('New employee added: ', NEW.name, ' with salary $', NEW.salary));
END //

DELIMITER ;
```

Execution & Testing: Run this statement to add a new employee:
```sql 
INSERT INTO employees (employee_id, name, salary, department) 
VALUES (1, 'Alice Smith', 75000.00, 'Engineering');
```

Expected Output: If you query the audit_log table:
```sql
SELECT action_type, description FROM audit_log;
```

action_type | description
-------|------
INSERT| New employee added: Alice Smith with salary $75000.00

Why this output happens
1. The INSERT INTO employees statement activates the trigger.
2. The trigger captures the data from the newly inserted row using the NEW keyword (NEW.name and NEW.salary).
3. It executes the inner statement, generating a fresh row inside audit_log without requiring a separate manual insert query.

**Example 2: BEFORE UPDATE Trigger (Data Validation & Modification):**
This trigger automatically forces employee names to uppercase before saving them to the database, ensuring clean and uniform data formatting.The Code

```sql
DELIMITER //

CREATE TRIGGER before_employee_update
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    -- 'NEW' allows you to modify values BEFORE they hit the database
    SET NEW.name = UPPER(NEW.name);
END //

DELIMITER ;
```
Execution & Testing: Run this statement to update Alice's department and change her name format to lowercase:
```sql
UPDATE employees 
SET name = 'alice smith', department = 'IT' 
WHERE employee_id = 1;
```

Expected Output: If you query the employees table:
```sql
SELECT employee_id, name, department FROM employees WHERE employee_id = 1;
```

employee_id | name | department
-|-|-
1 | ALICE SMITH | IT 

**Why this output happens**: 
1. The UPDATE statement triggers the logic before writing the change to disk.
2. The code intercepts the incoming lowercase value (alice smith) inside NEW.name.
3. The UPPER() function modifies the value directly in the buffer, forcing it to save as ALICE SMITH.

**Example 3: AFTER UPDATE Trigger (Tracking Historical Changes)**
This trigger records both the old salary and the new salary whenever an employee gets a raise.The Code
```sql
DELIMITER //

CREATE TRIGGER after_salary_update
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    -- Only log if the salary actually changed
    IF OLD.salary <> NEW.salary THEN
        INSERT INTO audit_log (action_type, description)
        VALUES (
            'SALARY_CHANGE', 
            CONCAT('Employee ID ', NEW.employee_id, ' salary changed from $', OLD.salary, ' to $', NEW.salary)
        );
    END IF;
END //

DELIMITER ;
```
Execution & Testing: Give Alice a salary raise:
```sql
UPDATE employees 
SET salary = 82000.00 
WHERE employee_id = 1;
```

Expected Output: If you query the audit_log table:
```sql
SELECT action_type, description FROM audit_log WHERE action_type = 'SALARY_CHANGE';
```

action_type | description | 
-|-
SALARY_CHANGE | Employee ID 1 salary changed from $75000.00 to $82000.00

**Why this output happens:** 
1. The UPDATE statement alters the salary value.
2. The trigger checks if OLD.salary (75000.00) is different from NEW.
3. salary (82000.00).Because the evaluation is true, it inserts a descriptive string summarizing the specific financial transformation into the history log.

### Real-World Cases

**Case 1: Audit Logging**

A trigger on an `employees` table inserts a row into `employees_audit` whenever a salary is updated, recording the old value, new value, and timestamp.

**Case 2: Inventory Validation**

A `BEFORE INSERT` trigger on `order_items` checks that the requested quantity does not exceed available stock, raising an error if it does.

### References

- DML Triggers - SQL Server - https://learn.microsoft.com/vi-vn/SQL/relational-databases/triggers/dml-triggers 
- MySQL Trigger Syntax and Examples - https://dev.mysql.com/doc/refman/8.4/en/trigger-syntax.html 
- PostgreSQL CREATE TRIGGER - https://www.postgresql.org/docs/14/sql-createtrigger.html 
- CREATE TRIGGER (Transact-SQL) - https://learn.microsoft.com/zh-cn/previous-versions/SQL/SQL-server-2008/ms189799(v=sql.100) 

---

## 2. Trigger Granularity

### Definitions

**Core Definition:** Trigger granularity defines whether a trigger fires once for each affected row (row-level) or once for the entire DML statement (statement-level).

**Technical Definition:** A row-level trigger (`FOR EACH ROW`) fires once for every row affected by the triggering DML statement. A statement-level trigger fires once for the whole statement, regardless of how many rows are affected . Oracle, PostgreSQL, and DB2 support both granularities. Microsoft SQL Server only supports statement-level triggers (though they can access all affected rows via the `inserted` and `deleted` tables). MySQL only supports row-level triggers .

**Beginner-Friendly Explanation:** If you update 100 rows, a row-level trigger runs 100 times (once per row). A statement-level trigger runs once, no matter how many rows changed.

### Purposes

- **Row-level:** To apply logic that depends on individual row values (e.g., validation, per-row logging).
- **Statement-level:** To perform actions that apply to the entire operation (e.g., logging "UPDATE occurred on table X").
- **Performance:** Statement-level triggers are generally faster for bulk operations because they run once .

### Syntax Rules and Structure

**Row-Level Trigger (MySQL, Oracle, PostgreSQL):**

```sql
-- MySQL
CREATE TRIGGER trigger_name BEFORE INSERT ON table_name
FOR EACH ROW
SET NEW.column = UPPER(NEW.column);

-- PostgreSQL
CREATE TRIGGER trigger_name BEFORE INSERT ON table_name
FOR EACH ROW EXECUTE FUNCTION trigger_func();
```

**Statement-Level Trigger (SQL Server, Oracle, PostgreSQL):**

```sql
-- SQL Server (all triggers are statement-level)
CREATE TRIGGER trigger_name ON table_name
AFTER UPDATE
AS
    INSERT INTO audit_log (action) VALUES ('UPDATE occurred');

-- Oracle
CREATE TRIGGER trigger_name AFTER UPDATE ON table_name
BEGIN
    INSERT INTO audit_log (action) VALUES ('UPDATE occurred');
END;
```

**Component Breakdown:**

| Granularity | Fires | Access to Row Data |
|-------------|-------|-------------------|
| Row-level (`FOR EACH ROW`) | Once per affected row | `NEW`/`OLD` variables available  |
| Statement-level | Once per statement | `NEW`/`OLD` not available; transition tables may be available  |

**Syntax Rules:**

- MySQL only supports row-level triggers .
- SQL Server only supports statement-level triggers but provides `inserted` and `deleted` tables for row access .
- Oracle and PostgreSQL support both; `FOR EACH ROW` specifies row-level, absence means statement-level.
- PostgreSQL statement-level triggers cannot use `NEW`/`OLD` directly but can use transition tables (PostgreSQL 10+) .

**Constraints and Limitations:**

- Row-level triggers on large updates can be extremely slow (RBAR) .
- Statement-level triggers cannot access individual row values directly (platform-dependent).
- SQL Server's lack of `BEFORE` triggers means validation must be done in `AFTER` triggers with rollback, or via `INSTEAD OF`.

### Annotated Complete Code Examples

**Example 1: Row-Level vs. Statement-Level Performance (Oracle)**

```sql
-- Row-level trigger: fires for each of 1000 updated rows
CREATE OR REPLACE TRIGGER row_level
AFTER UPDATE ON table1
FOR EACH ROW
BEGIN
    INSERT INTO audit_log VALUES (:OLD.id, :NEW.id);
END;

-- Statement-level trigger: fires once for the entire UPDATE
CREATE OR REPLACE TRIGGER stmt_level
AFTER UPDATE ON table1
BEGIN
    INSERT INTO audit_log VALUES (0, 0);
END;
```

**Why performance differs:** The row-level trigger executes 1000 separate INSERT statements (one per row). The statement-level trigger executes one INSERT. In a benchmark, the statement-level approach was at least 15 times faster .

### Real-World Cases

**Case 1: Per-Row Validation**

A row-level `BEFORE INSERT` trigger validates that each new employee's salary is within the department's range.

**Case 2: Bulk Operation Logging**

A statement-level `AFTER UPDATE` trigger logs a single entry saying "Bulk salary update performed" without logging individual rows.

### References

- Revision: Row level vs. Statement level triggers - https://stackoverflow.com/revisions/ef81de5a-282b-4641-a3c8-7df5683dda0f/view-source 
- Revision: Statement-level trigger OLD/NEW - https://stackoverflow.com/revisions/6766a7a0-760d-4189-890a-817430c068cb/view-source 
- Oracle Ask TOM: bind variables in triggers - https://asktom.oracle.com/ords/asktom.search?tag=bind-variables-in-triggers 
- Revision: Row level vs statement level - https://stackoverflow.com/revisions/11315115/1 

---

## 3. Transition Variables & Tables

### Definitions

**Core Definition:** Transition variables (or tables) are special mechanisms that allow triggers to access the old and new states of the data being modified.

**Technical Definition:** In row-level triggers, `NEW` and `OLD` are virtual records representing the row before and after the DML operation. In SQL Server, the `inserted` and `deleted` tables are special temporary tables that hold the affected rows . In PostgreSQL, transition tables (PostgreSQL 10+) allow statement-level `AFTER` triggers to access the entire set of affected rows .

**Beginner-Friendly Explanation:** When a trigger fires, it needs to know what the data looked like before and after the change. `NEW` is the "after" version, `OLD` is the "before" version. In SQL Server, these are given as tables called `inserted` and `deleted`.

### Purposes

- To compare old and new values (e.g., detect salary increases).
- To log changes with both before and after snapshots.
- To enforce referential integrity by checking related tables.
- To synchronize changes across tables.

### Syntax Rules and Structure

**SQL Server: `inserted` and `deleted` Tables**

| Operation | `inserted` | `deleted` |
|-----------|-----------|-----------|
| INSERT | New rows | Empty |
| UPDATE | New rows (after update) | Old rows (before update)  |
| DELETE | Empty | Deleted rows |

**MySQL/Oracle/PostgreSQL: `OLD` and `NEW` Variables**

| Operation | `OLD` | `NEW` |
|-----------|-------|-------|
| INSERT | Undefined/empty | New values |
| UPDATE | Old values | New values  |
| DELETE | Old values | Undefined/empty |

**Component Breakdown:**

| Platform | Mechanism | Availability |
|----------|-----------|-------------|
| SQL Server | `inserted`, `deleted` tables | All DML triggers  |
| MySQL | `OLD`, `NEW` row variables | Row-level triggers  |
| Oracle | `:OLD`, `:NEW` pseudo-records | Row-level triggers  |
| PostgreSQL | `OLD`, `NEW` records | Row-level triggers; transition tables for statement-level  |

**Syntax Rules:**

- SQL Server: `inserted` and `deleted` tables are read-only, memory-resident, and automatically managed .
- MySQL: `NEW.column` and `OLD.column` access individual columns .
- Oracle: Prefix with colon (`:NEW.column`, `:OLD.column`) .
- PostgreSQL: `NEW` and `OLD` are record variables; `NEW` is null in statement-level triggers .
- SQL Server `UPDATE` populates both `deleted` (old) and `inserted` (new), simulating a delete-then-insert .

**Constraints and Limitations:**

- SQL Server does not allow direct modification of `inserted`/`deleted` tables .
- PostgreSQL statement-level triggers cannot use `OLD`/`NEW` without transition tables .
- MySQL `OLD` is read-only; `NEW` can be modified in `BEFORE` triggers.
- Oracle requires colon prefix for `:OLD` and `:NEW` .

### Annotated Complete Code Examples

**Example 1: SQL Server `inserted` and `deleted` Tables**

```sql
-- Trigger to log salary changes
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

**Why this works:** The `inserted` table holds the new salary values, and `deleted` holds the old values. Joining them on `EmployeeID` allows the trigger to compare old and new salaries and log only actual changes .

**Example 2: MySQL `OLD` and `NEW` Variables**

```sql
CREATE TRIGGER before_salary_update
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    IF NEW.salary < OLD.salary THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'Salary cannot be decreased';
    END IF;
END;
```

**Why this works:** The `BEFORE UPDATE` trigger fires before each row is updated. `OLD.salary` is the current value, `NEW.salary` is the proposed value. If the new salary is lower, the trigger raises an error, preventing the update .

### Real-World Cases

**Case 1: Change Data Capture**

A trigger logs all `UPDATE` operations to an audit table, capturing both `OLD` and `NEW` values for each column.

**Case 2: Preventing Invalid Updates**

A `BEFORE UPDATE` trigger checks that a `status` column only moves forward in a workflow (e.g., `pending` → `approved`, never `approved` → `pending`).

### References

- SQL Server inserted and deleted tables - https://learn.microsoft.com/ko-kr/previous-versions/sql/sql-server-2012/ms191300(v=sql.110) 
- SQL Server DML triggers and inserted/deleted tables - https://learn.microsoft.com/ko-kr/sql/relational-databases/triggers/use-the-inserted-and-deleted-tables 
- Oracle OLD and NEW virtual records - https://docs.oracle.com/zh-tw/database/oracle/oracle-database/19/tdddg/creating-triggers.html 
- PostgreSQL statement-level trigger OLD/NEW - https://stackoverflow.com/revisions/6766a7a0-760d-4189-890a-817430c068cb/view-source 

---

## 4. Execution Timing

### Definitions

**Core Definition:** Execution timing determines when a trigger fires relative to the DML operation: `BEFORE` (before the operation), `AFTER` (after the operation), or `INSTEAD OF` (replacing the operation).

**Technical Definition:** A `BEFORE` trigger fires before the DML statement's effects are applied, allowing validation or modification of data. An `AFTER` trigger fires after the DML statement completes successfully, used for auditing or cascading actions. An `INSTEAD OF` trigger replaces the DML statement entirely, allowing updates to views that would otherwise be non-updatable .

**Beginner-Friendly Explanation:** `BEFORE` is like a bouncer checking IDs before letting people in. `AFTER` is like a security camera recording what happened. `INSTEAD OF` is like a substitute teacher—it does the job instead of the regular statement.

### Purposes

**BEFORE Triggers:**
- To validate data before insertion or update.
- To modify data before it is stored (e.g., trimming strings, calculating derived values).
- To enforce complex business rules.

**AFTER Triggers:**
- To audit changes after they are committed.
- To cascade changes to related tables.
- To perform post-processing actions.

**INSTEAD OF Triggers:**
- To update non-updatable views.
- To override standard DML behavior.
- To perform custom validation and redirect operations.

### Syntax Rules and Structure

**BEFORE Trigger (MySQL):**

```sql
CREATE TRIGGER before_insert_employee
BEFORE INSERT ON employees
FOR EACH ROW
SET NEW.hire_date = IFNULL(NEW.hire_date, CURDATE());
```

**AFTER Trigger (SQL Server):**

```sql
CREATE TRIGGER after_insert_employee
ON employees
AFTER INSERT
AS
    INSERT INTO audit_log (action, table_name)
    VALUES ('INSERT', 'employees');
```

**INSTEAD OF Trigger (SQL Server):**

```sql
CREATE TRIGGER instead_of_insert_view
ON EmployeeView
INSTEAD OF INSERT
AS
BEGIN
    INSERT INTO employees (emp_id, emp_name, dept_id)
    SELECT emp_id, emp_name, dept_id FROM inserted;
END;
```

**Component Breakdown:**

| Timing | Fires | Can Modify Data | Use Case |
|--------|-------|-----------------|----------|
| `BEFORE` | Before DML | Yes (in row-level) | Validation, transformation  |
| `AFTER` | After DML | No (data already changed) | Auditing, cascading  |
| `INSTEAD OF` | Replaces DML | Yes (custom logic) | Updating views  |

**Syntax Rules:**

- MySQL supports `BEFORE` and `AFTER` for row-level triggers .
- SQL Server supports `AFTER` and `INSTEAD OF`; no `BEFORE` .
- Oracle and PostgreSQL support all three timing options.
- SQL Server `INSTEAD OF` triggers can be defined on views (one per action per view) .
- Constraints are checked after `INSTEAD OF` trigger execution but before `AFTER` trigger execution .

**Constraints and Limitations:**

- SQL Server `AFTER` triggers never fire if a constraint violation occurs .
- MySQL `BEFORE` triggers can modify `NEW` values but not `OLD` .
- `INSTEAD OF` triggers cannot be defined on tables with cascading foreign keys in SQL Server .
- PostgreSQL `INSTEAD OF` triggers are mainly used for views .

### Annotated Complete Code Examples

**Example 1: BEFORE Trigger for Validation (Oracle)**

```sql
CREATE OR REPLACE TRIGGER check_salary
BEFORE INSERT OR UPDATE ON employees
FOR EACH ROW
BEGIN
    IF :NEW.salary < 0 THEN
        RAISE_APPLICATION_ERROR(-20001, 'Salary cannot be negative');
    END IF;
END;
```

**Why this works:** The `BEFORE` trigger fires before the row is inserted or updated. If the new salary is negative, it raises an error, preventing the operation .

**Example 2: INSTEAD OF Trigger for View (SQL Server)**

```sql
-- Create a view joining two tables
CREATE VIEW EmployeeDepartmentView AS
SELECT e.emp_id, e.emp_name, d.dept_name
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id;

-- INSTEAD OF INSERT trigger to handle inserts through the view
CREATE TRIGGER trg_InsertEmployeeView
ON EmployeeDepartmentView
INSTEAD OF INSERT
AS
BEGIN
    INSERT INTO employees (emp_id, emp_name, dept_id)
    SELECT i.emp_id, i.emp_name, d.dept_id
    FROM inserted i
    JOIN departments d ON i.dept_name = d.dept_name;
END;
```

**Why this works:** The view is not directly updatable because it joins two tables. The `INSTEAD OF INSERT` trigger intercepts the insert, resolves the department name to a department ID, and inserts into the base `employees` table .

### Real-World Cases

**Case 1: Audit Trail with AFTER Trigger**

An `AFTER UPDATE` trigger on a `customers` table inserts a row into `customer_audit` for every change, recording who changed what and when.

**Case 2: View-Based Data Entry with INSTEAD OF**

An application uses a view that joins `orders` and `customers`. An `INSTEAD OF INSERT` trigger on the view routes new rows to the correct base tables.

### References

- DML Triggers - SQL Server - https://learn.microsoft.com/vi-vn/SQL/relational-databases/triggers/dml-triggers 
- CREATE TRIGGER (Transact-SQL) - https://learn.microsoft.com/zh-cn/previous-versions/SQL/SQL-server-2008/ms189799(v=sql.100) 
- MySQL Trigger Syntax - https://dev.mysql.com/doc/refman/8.4/en/trigger-syntax.html 
- Oracle Creating Triggers - https://docs.oracle.com/zh-tw/database/oracle/oracle-database/19/tdddg/creating-triggers.html 
- CREATE TRIGGER (Transact-SQL) SQL Server 2008 R2 - https://learn.microsoft.com/hi-in/previous-versions/sql/sql-server-2008-r2/ms189799(v=sql.105) 

---

## Summary Table of Trigger Features by Platform

| Feature | SQL Server | MySQL | PostgreSQL | Oracle |
|---------|-----------|-------|------------|--------|
| **BEFORE triggers** | ❌ | ✅ | ✅ | ✅ |
| **AFTER triggers** | ✅ | ✅ | ✅ | ✅ |
| **INSTEAD OF triggers** | ✅ | ❌ | ✅ | ✅ |
| **Row-level triggers** | ❌ (statement-level only) | ✅ (row-level only) | ✅ | ✅ |
| **Statement-level triggers** | ✅ | ❌ | ✅ | ✅ |
| **OLD/NEW variables** | ❌ (uses `inserted`/`deleted`) | ✅ (`OLD`/`NEW`) | ✅ (`OLD`/`NEW`) | ✅ (`:OLD`/`:NEW`) |
| **Transition tables** | ✅ (`inserted`/`deleted`) | ❌ | ✅ (PostgreSQL 10+) | ❌ |
| **Multiple events per trigger** | ❌ | ❌ | ✅ (`OR`) | ✅ (`OR`) |

---

## Dialect Comparison: Trigger Syntax

| Platform | Create Statement | Event Binding | Granularity |
|----------|-----------------|---------------|-------------|
| SQL Server | `CREATE TRIGGER name ON table` | `AFTER INSERT, UPDATE` | Statement-level |
| MySQL | `CREATE TRIGGER name BEFORE INSERT ON table` | `BEFORE INSERT` | `FOR EACH ROW` |
| PostgreSQL | `CREATE TRIGGER name BEFORE INSERT ON table` | `BEFORE INSERT OR UPDATE` | `FOR EACH ROW` or statement |
| Oracle | `CREATE TRIGGER name BEFORE INSERT ON table` | `BEFORE INSERT OR UPDATE` | `FOR EACH ROW` or statement |

---

## References

- DML Triggers - SQL Server - https://learn.microsoft.com/vi-vn/SQL/relational-databases/triggers/dml-triggers 
- MySQL Trigger Syntax and Examples - https://dev.mysql.com/doc/refman/8.4/en/trigger-syntax.html 
- PostgreSQL CREATE TRIGGER - https://www.postgresql.org/docs/14/sql-createtrigger.html 
- Oracle Ask TOM: bind variables in triggers - https://asktom.oracle.com/ords/asktom.search?tag=bind-variables-in-triggers 
- CREATE TRIGGER (Transact-SQL) - https://learn.microsoft.com/zh-cn/previous-versions/SQL/SQL-server-2008/ms189799(v=sql.100) 
- Revision: Row level vs. Statement level triggers - https://stackoverflow.com/revisions/ef81de5a-282b-4641-a3c8-7df5683dda0f/view-source 
- SQL Server inserted and deleted tables - https://learn.microsoft.com/ko-kr/previous-versions/sql/sql-server-2012/ms191300(v=sql.110) 
- MySQL Using Triggers - https://dev.mysql.com/doc/refman/26.7/en/triggers.html 
- Oracle Creating Triggers - https://docs.oracle.com/zh-tw/database/oracle/oracle-database/19/tdddg/creating-triggers.html 
- CREATE TRIGGER (Transact-SQL) SQL Server 2008 R2 - https://learn.microsoft.com/hi-in/previous-versions/sql/sql-server-2008-r2/ms189799(v=sql.105) 
- Revision: Row level vs statement level - https://stackoverflow.com/revisions/11315115/1 
- PostgreSQL statement-level trigger OLD/NEW - https://stackoverflow.com/revisions/6766a7a0-760d-4189-890a-817430c068cb/view-source 
- SQL Server DML triggers and inserted/deleted tables - https://learn.microsoft.com/ko-kr/sql/relational-databases/triggers/use-the-inserted-and-deleted-tables 