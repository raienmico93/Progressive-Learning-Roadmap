# Database Testing: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**: Database testing is the systematic process of validating that a database's schema objects—constraints, relationships, triggers, stored procedures, functions, and transactions—behave according to their specifications and enforce business rules correctly.

**Technical Definition**: Database unit testing encompasses constraint testing (verifying `NOT NULL`, `CHECK`, `UNIQUE`, `DEFAULT`, and foreign-key enforcement), referential-integrity testing (validating `ON DELETE CASCADE`, `ON DELETE SET NULL`, `ON DELETE RESTRICT`, and `ON UPDATE` behaviors), trigger testing (verifying audit logs, timestamp automation, and historical snapshots fire on INSERT/UPDATE/DELETE), stored-procedure testing (passing input parameters and verifying output parameters, result sets, and side effects), function testing (ensuring scalar and table-valued functions return mathematically precise, deterministic results), transaction testing (verifying atomicity by forcing failures mid-transaction and confirming ROLLBACK), and database unit testing frameworks (tSQLt for SQL Server, pgTAP for PostgreSQL, MyTAP for MySQL).

**Beginner-Friendly Explanation**: Database testing is like quality-checking a factory's machinery. You test that the safety guards work (constraints), that conveyor belts move parts correctly (referential integrity), that sensors trigger alarms (triggers), that the robots perform their tasks accurately (procedures and functions), and that if something fails mid-production, the whole batch is scrapped cleanly (transactions). Testing frameworks are the quality-control checklists.

### Key Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Schema-Centric** | Tests validate database objects, not application code |
| **Isolation** | Each test runs in its own transaction, rolled back afterward |
| **Deterministic** | Tests must produce the same result regardless of execution order |
| **Constraint-Focused** | Many tests verify that invalid operations are rejected |
| **Side-Effect Verification** | Triggers and procedures are tested for their side effects, not just outputs |
| **Framework-Supported** | pgTAP, tSQLt, and MyTAP provide assertion functions and test runners |

### Prerequisites

- **Test Database**: A dedicated database separate from development or production
- **Test Framework**: pgTAP (PostgreSQL), tSQLt (SQL Server), or MyTAP (MySQL)
- **Transaction Isolation**: Ability to run tests in transactions that roll back
- **Seed Data**: Known, minimal data sets for deterministic tests
- **CI/CD Integration**: Ability to run database tests automatically on schema changes

### Related Programming Areas

- **Continuous Integration**: Running database tests on every schema migration
- **Test Data Management**: Seeding, resetting, and anonymizing test databases
- **Data Quality Engineering**: Ensuring constraints match business rules
- **Security Engineering**: Testing that constraints prevent invalid data injection
- **Database Administration**: Verifying triggers and procedures after deployment

### Core Concepts Overview

Database testing comprises seven complementary categories:

1. **Constraint Testing**: Breaching NOT NULL, CHECK, UNIQUE, and DEFAULT logic
2. **Referential-Integrity Testing**: Testing cascading behaviors (ON DELETE CASCADE, ON UPDATE RESTRICT)
3. **Trigger Testing**: Verifying audit logging, timestamps, and historical snapshots
4. **Stored-Procedure Testing**: Passing inputs and verifying outputs and side effects
5. **Function Testing**: Ensuring scalar and table-valued functions return precise values
6. **Transaction Testing**: Verifying atomic executions and ROLLBACK states
7. **Database Unit Testing Frameworks**: pgTAP, tSQLt, MyTAP

---

## Core Concept 1: Constraint Testing

### Definitions

**Core Definition**: Constraint testing verifies that database constraints reject invalid data and accept valid data as defined by the schema.

**Technical Definition**: Constraint testing validates `NOT NULL` (rejects NULL), `CHECK` (rejects values failing a boolean predicate), `UNIQUE` (rejects duplicate non-NULL values), `PRIMARY KEY` (combines NOT NULL and UNIQUE), `DEFAULT` (applies when no value is supplied), and `FOREIGN KEY` (rejects non-existent parent references). Tests intentionally attempt to breach each constraint and assert that the expected error is raised.

**Beginner-Friendly Explanation**: Constraint testing is like testing a bouncer at a club. The bouncer (constraint) should reject people without ID (NOT NULL), people under 21 (CHECK), people already inside (UNIQUE), and people not on the guest list (FOREIGN KEY). You test by trying to sneak past each rule.

### Purposes

- **To** verify that NOT NULL constraints reject NULL inserts
- **To** verify that CHECK constraints reject values outside the allowed range
- **To** verify that UNIQUE constraints reject duplicate values
- **To** verify that DEFAULT values are applied when no value is supplied
- **To** verify that constraint violations raise the expected error codes

### Syntax Rules and Structure

#### Constraint Types

```sql
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,                    -- PRIMARY KEY
    product_name VARCHAR(100) NOT NULL,               -- NOT NULL
    price NUMERIC(10,2) CHECK (price > 0),            -- CHECK
    sku VARCHAR(50) UNIQUE,                           -- UNIQUE
    status VARCHAR(20) DEFAULT 'active',              -- DEFAULT
    category_id INT REFERENCES categories(category_id) -- FOREIGN KEY
);
```

#### Test Structure (pgTAP — PostgreSQL)

```sql
BEGIN;
SELECT plan(6);

-- Test 1: NOT NULL rejects NULL
SELECT throws_ok(
    $$INSERT INTO products (product_name, price) VALUES (NULL, 10.00)$$,
    '23502',  -- not_null_violation
    NULL,
    'NOT NULL constraint rejects NULL product_name'
);

-- Test 2: CHECK rejects negative price
SELECT throws_ok(
    $$INSERT INTO products (product_name, price) VALUES ('Test', -5.00)$$,
    '23514',  -- check_violation
    NULL,
    'CHECK constraint rejects negative price'
);

-- Test 3: UNIQUE rejects duplicate SKU
SELECT throws_ok(
    $$INSERT INTO products (product_name, price, sku) VALUES ('A', 10, 'SKU1')$$,
    '23505',  -- unique_violation
    NULL,
    'UNIQUE constraint rejects duplicate SKU'
);

-- Test 4: DEFAULT applies when no value supplied
INSERT INTO products (product_name, price) VALUES ('Default Test', 10.00);
SELECT is(
    (SELECT status FROM products WHERE product_name = 'Default Test'),
    'active',
    'DEFAULT value applied when status not supplied'
);

-- Test 5: FOREIGN KEY rejects non-existent category
SELECT throws_ok(
    $$INSERT INTO products (product_name, price, category_id) VALUES ('Test', 10, 99999)$$,
    '23503',  -- foreign_key_violation
    NULL,
    'FOREIGN KEY rejects non-existent category'
);

-- Test 6: Valid insert succeeds
SELECT lives_ok(
    $$INSERT INTO products (product_name, price, sku) VALUES ('Valid', 10.00, 'SKU2')$$,
    'Valid insert succeeds'
);

SELECT * FROM finish();
ROLLBACK;
```

#### Component Breakdown

| Constraint | SQLSTATE | Test Assertion |
|------------|----------|----------------|
| NOT NULL | 23502 | `throws_ok` |
| CHECK | 23514 | `throws_ok` |
| UNIQUE | 23505 | `throws_ok` |
| FOREIGN KEY | 23503 | `throws_ok` |
| DEFAULT | — | `is` (verify applied value) |
| PRIMARY KEY | 23505/23502 | `throws_ok` |

#### Syntax Rules

- Tests run inside a transaction that is rolled back at the end (`BEGIN; ... ROLLBACK;`).
- `throws_ok` asserts that a statement raises a specific SQLSTATE.
- `lives_ok` asserts that a statement executes without error.
- `is` asserts equality between actual and expected values.
- Each test file uses `SELECT plan(n)` to declare the number of tests.

#### Constraints and Limitations

- Constraint names should be explicitly defined for stable error messages.
- `CHECK` constraints cannot reference other rows or use subqueries.
- `UNIQUE` constraints allow multiple NULLs in most engines (PostgreSQL, MySQL, SQL Server).
- Deferred constraints check at commit time, not statement time.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Testing NOT NULL and CHECK Constraints

```sql
-- Step 1: Set up the test
BEGIN;
SELECT plan(4);

-- Step 2: Test NOT NULL rejection
SELECT throws_ok(
    $$INSERT INTO products (product_name, price) VALUES (NULL, 100.00)$$,
    '23502',
    NULL,
    'NOT NULL rejects NULL product_name'
);

-- Step 3: Test CHECK rejection for negative price
SELECT throws_ok(
    $$INSERT INTO products (product_name, price) VALUES ('Test', -50.00)$$,
    '23514',
    NULL,
    'CHECK rejects negative price'
);

-- Step 4: Test CHECK rejection for zero price
SELECT throws_ok(
    $$INSERT INTO products (product_name, price) VALUES ('Test', 0)$$,
    '23514',
    NULL,
    'CHECK rejects zero price'
);

-- Step 5: Test valid insert succeeds
SELECT lives_ok(
    $$INSERT INTO products (product_name, price) VALUES ('Valid Product', 100.00)$$,
    'Valid insert succeeds'
);

SELECT * FROM finish();
ROLLBACK;
```

**Expected Output**:
```
1..4
ok 1 - NOT NULL rejects NULL product_name
ok 2 - CHECK rejects negative price
ok 3 - CHECK rejects zero price
ok 4 - Valid insert succeeds
```

**Why This Output Occurs**: `throws_ok` executes the INSERT and verifies that it raises SQLSTATE 23502 (NOT NULL) or 23514 (CHECK). `lives_ok` verifies that a valid insert executes without error. The transaction is rolled back, leaving the database unchanged.

### Real-World Cases

**Case 1: Price Constraint Migration**: A new `CHECK (price > 0)` constraint is added. Constraint tests verify that negative and zero prices are rejected while positive prices succeed.

**Case 2: Email Uniqueness**: A `UNIQUE` constraint on `email` is tested to ensure duplicate emails are rejected, but multiple NULL emails are allowed (per business rules).

**Case 3: Default Status**: A `DEFAULT 'active'` on a `status` column is tested to ensure new rows receive the default when no value is supplied.

---

## Core Concept 2: Referential-Integrity Testing

### Definitions

**Core Definition**: Referential-integrity testing verifies that foreign-key constraints enforce the correct behavior when parent rows are deleted or updated.

**Technical Definition**: Referential-integrity testing validates `ON DELETE CASCADE` (child rows are deleted), `ON DELETE SET NULL` (child foreign keys are nullified), `ON DELETE RESTRICT` / `NO ACTION` (parent deletion is prevented), `ON DELETE SET DEFAULT` (child foreign keys are set to their default), and the equivalent `ON UPDATE` behaviors. Tests verify both the parent operation and the resulting child state.

**Beginner-Friendly Explanation**: Referential-integrity testing is like testing a filing system where folders (parents) contain documents (children). If you shred a folder, what happens to the documents? Are they shredded too (CASCADE), moved to a "no folder" pile (SET NULL), or does the shredder refuse (RESTRICT)? The test verifies the correct behavior.

### Purposes

- **To** verify that ON DELETE CASCADE deletes child rows when the parent is deleted
- **To** verify that ON DELETE SET NULL nullifies child foreign keys
- **To** verify that ON DELETE RESTRICT prevents parent deletion when children exist
- **To** verify that ON UPDATE CASCADE propagates parent key changes to children
- **To** verify that ON UPDATE RESTRICT prevents parent key changes when children exist

### Syntax Rules and Structure

#### Foreign-Key Actions

```sql
-- CASCADE: delete child rows
ALTER TABLE orders
ADD CONSTRAINT fk_orders_customer
FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
ON DELETE CASCADE;

-- SET NULL: nullify child foreign key
ALTER TABLE orders
ADD CONSTRAINT fk_orders_customer
FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
ON DELETE SET NULL;

-- RESTRICT: prevent parent deletion
ALTER TABLE orders
ADD CONSTRAINT fk_orders_customer
FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
ON DELETE RESTRICT;

-- SET DEFAULT: set child foreign key to default
ALTER TABLE orders
ADD CONSTRAINT fk_orders_customer
FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
ON DELETE SET DEFAULT;

-- ON UPDATE CASCADE
ALTER TABLE orders
ADD CONSTRAINT fk_orders_customer
FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
ON UPDATE CASCADE;
```

#### Test Structure (pgTAP)

```sql
BEGIN;
SELECT plan(3);

-- Seed parent and children
INSERT INTO customers (customer_id, name) VALUES (1, 'Alice');
INSERT INTO orders (order_id, customer_id, total) VALUES (100, 1, 500.00), (101, 1, 300.00);

-- Test 1: ON DELETE CASCADE deletes children
DELETE FROM customers WHERE customer_id = 1;
SELECT is(
    (SELECT COUNT(*) FROM orders WHERE customer_id = 1),
    0::bigint,
    'ON DELETE CASCADE deletes child orders'
);

-- Test 2: Parent is also deleted
SELECT is(
    (SELECT COUNT(*) FROM customers WHERE customer_id = 1),
    0::bigint,
    'Parent customer is deleted'
);

-- Test 3: Other customers' orders are unaffected
SELECT is(
    (SELECT COUNT(*) FROM orders),
    0::bigint,
    'All orders for deleted customer are gone'
);

SELECT * FROM finish();
ROLLBACK;
```

#### Component Breakdown

| Action | Parent Deleted | Child Behavior |
|--------|---------------|----------------|
| `CASCADE` | Yes | Child rows deleted |
| `SET NULL` | Yes | Child FK set to NULL |
| `SET DEFAULT` | Yes | Child FK set to default |
| `RESTRICT` | No (error) | Parent deletion blocked |
| `NO ACTION` | No (error) | Like RESTRICT, checked at commit |

#### Syntax Rules

- `RESTRICT` checks immediately; `NO ACTION` checks at the end of the statement/transaction.
- `SET NULL` requires the child column to be nullable.
- `SET DEFAULT` requires the child column to have a default value.
- `CASCADE` can propagate recursively through multiple levels of foreign keys.

#### Constraints and Limitations

- `CASCADE` can cause unintended mass deletions; test with realistic data.
- `SET NULL` on a `NOT NULL` column raises an error.
- `ON UPDATE CASCADE` is useful for natural keys but uncommon with surrogate keys.
- MySQL's `RESTRICT` and `NO ACTION` are equivalent.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Testing ON DELETE CASCADE

```sql
-- Step 1: Create tables with CASCADE
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100)
);
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id) ON DELETE CASCADE,
    total NUMERIC(10,2)
);

-- Step 2: Seed data
INSERT INTO customers (customer_id, customer_name) VALUES (1, 'Alice'), (2, 'Bob');
INSERT INTO orders (customer_id, total) VALUES (1, 100.00), (1, 200.00), (2, 300.00);

-- Step 3: Test CASCADE behavior
BEGIN;
SELECT plan(3);

DELETE FROM customers WHERE customer_id = 1;

SELECT is(
    (SELECT COUNT(*) FROM orders WHERE customer_id = 1),
    0::bigint,
    'CASCADE deleted Alice''s orders'
);

SELECT is(
    (SELECT COUNT(*) FROM orders WHERE customer_id = 2),
    1::bigint,
    'Bob''s orders are unaffected'
);

SELECT is(
    (SELECT COUNT(*) FROM customers),
    1::bigint,
    'Only Bob remains'
);

SELECT * FROM finish();
ROLLBACK;
```

**Expected Output**:
```
1..3
ok 1 - CASCADE deleted Alice's orders
ok 2 - Bob's orders are unaffected
ok 3 - Only Bob remains
```

**Why This Output Occurs**: Deleting customer 1 triggers `ON DELETE CASCADE`, which deletes all orders with `customer_id = 1`. Bob's orders (customer 2) are unaffected. The test verifies the cascade behavior and the isolation of other customers' data.

#### Example 2: Testing ON DELETE RESTRICT

```sql
-- Step 1: Create tables with RESTRICT
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100)
);
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id) ON DELETE RESTRICT,
    total NUMERIC(10,2)
);

-- Step 2: Seed data
INSERT INTO customers (customer_id, customer_name) VALUES (1, 'Alice');
INSERT INTO orders (customer_id, total) VALUES (1, 100.00);

-- Step 3: Test RESTRICT behavior
BEGIN;
SELECT plan(1);

SELECT throws_ok(
    $$DELETE FROM customers WHERE customer_id = 1$$,
    '23503',
    NULL,
    'RESTRICT prevents deleting customer with orders'
);

SELECT * FROM finish();
ROLLBACK;
```

**Expected Output**:
```
1..1
ok 1 - RESTRICT prevents deleting customer with orders
```

**Why This Output Occurs**: The `RESTRICT` action blocks the DELETE because orders reference customer 1. PostgreSQL raises SQLSTATE 23503 (foreign key violation). The test verifies that the parent deletion is prevented.

### Real-World Cases

**Case 1: E-Commerce Order Cleanup**: Deleting a customer should delete their orders (CASCADE) but preserve order history for audit. The test verifies the business rule: `ON DELETE RESTRICT` if audit is required, `ON DELETE CASCADE` if not.

**Case 2: Category Deletion**: Deleting a product category should set the category_id of affected products to NULL (SET NULL) so products remain but are uncategorized.

**Case 3: Natural Key Update**: Changing a customer's email (used as a natural key) should propagate to orders via `ON UPDATE CASCADE`.

---

## Core Concept 3: Trigger Testing

### Definitions

**Core Definition**: Trigger testing verifies that database triggers fire at the correct time and perform their intended actions (audit logging, timestamp automation, historical snapshotting).

**Technical Definition**: Trigger testing validates `BEFORE` and `AFTER` triggers on `INSERT`, `UPDATE`, and `DELETE` operations. Tests verify that the trigger fires, that it modifies the correct rows, that it writes the expected audit records, that it updates timestamp columns, and that it respects `WHEN` conditions. Tests also verify that triggers do not fire on operations that should not activate them.

**Beginner-Friendly Explanation**: Trigger testing is like testing a motion-sensor light. When someone walks by (INSERT/UPDATE/DELETE), the light turns on (trigger fires) and records the event (audit log). The test verifies the light turns on at the right time and records the right information.

### Purposes

- **To** verify that audit triggers write the correct old and new values
- **To** verify that timestamp triggers set `updated_at` on UPDATE
- **To** verify that historical snapshot triggers capture the previous row state
- **To** verify that triggers respect WHEN conditions
- **To** verify that triggers do not fire on unrelated operations

### Syntax Rules and Structure

#### Trigger Definition (PostgreSQL)

```sql
-- Audit table
CREATE TABLE audit_log (
    audit_id SERIAL PRIMARY KEY,
    table_name TEXT,
    operation TEXT,
    row_id INT,
    old_values JSONB,
    new_values JSONB,
    changed_by TEXT,
    changed_at TIMESTAMP DEFAULT NOW()
);

-- Audit trigger function
CREATE OR REPLACE FUNCTION audit_trigger_func()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO audit_log (table_name, operation, row_id, new_values, changed_by)
        VALUES (TG_TABLE_NAME, 'INSERT', NEW.product_id, to_jsonb(NEW), current_user);
        RETURN NEW;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_log (table_name, operation, row_id, old_values, new_values, changed_by)
        VALUES (TG_TABLE_NAME, 'UPDATE', NEW.product_id, to_jsonb(OLD), to_jsonb(NEW), current_user);
        RETURN NEW;
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO audit_log (table_name, operation, row_id, old_values, changed_by)
        VALUES (TG_TABLE_NAME, 'DELETE', OLD.product_id, to_jsonb(OLD), current_user);
        RETURN OLD;
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Attach trigger
CREATE TRIGGER products_audit
AFTER INSERT OR UPDATE OR DELETE ON products
FOR EACH ROW EXECUTE FUNCTION audit_trigger_func();
```

#### Test Structure (pgTAP)

```sql
BEGIN;
SELECT plan(3);

-- Seed a product
INSERT INTO products (product_id, product_name, price) VALUES (1, 'Laptop', 1000.00);

-- Test 1: INSERT trigger fires
SELECT is(
    (SELECT COUNT(*) FROM audit_log WHERE operation = 'INSERT' AND row_id = 1),
    1::bigint,
    'INSERT trigger wrote audit record'
);

-- Test 2: UPDATE trigger fires with old and new values
UPDATE products SET price = 900.00 WHERE product_id = 1;
SELECT is(
    (SELECT new_values->>'price' FROM audit_log WHERE operation = 'UPDATE' AND row_id = 1),
    '900.00',
    'UPDATE trigger recorded new price'
);

-- Test 3: DELETE trigger fires
DELETE FROM products WHERE product_id = 1;
SELECT is(
    (SELECT old_values->>'product_name' FROM audit_log WHERE operation = 'DELETE' AND row_id = 1),
    'Laptop',
    'DELETE trigger recorded old values'
);

SELECT * FROM finish();
ROLLBACK;
```

#### Component Breakdown

| Trigger Timing | When It Fires | Use Case |
|----------------|---------------|----------|
| `BEFORE` | Before the operation | Validate or modify NEW values |
| `AFTER` | After the operation | Audit logging, cascading updates |
| `INSTEAD OF` | Instead of the operation | Views (make them updatable) |
| `FOR EACH ROW` | Once per affected row | Row-level auditing |
| `FOR EACH STATEMENT` | Once per statement | Statement-level logging |

#### Syntax Rules

- `AFTER` triggers see the final state of the row.
- `BEFORE` triggers can modify `NEW` to change what is written.
- `WHEN` conditions filter which rows fire the trigger.
- `TG_OP` contains the operation type (`INSERT`, `UPDATE`, `DELETE`).
- `OLD` is available for UPDATE and DELETE; `NEW` is available for INSERT and UPDATE.

#### Constraints and Limitations

- Triggers add write overhead; test performance impact.
- Triggers fire in alphabetical order by name (PostgreSQL).
- Recursive triggers can cause infinite loops; use `WHEN` conditions to prevent them.
- Triggers are not fired by `TRUNCATE` (statement-level trigger required).

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Testing an Audit Trigger

```sql
-- Step 1: Create audit infrastructure
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100),
    price NUMERIC(10,2)
);

CREATE TABLE audit_log (
    audit_id SERIAL PRIMARY KEY,
    operation TEXT,
    row_id INT,
    old_values JSONB,
    new_values JSONB,
    changed_at TIMESTAMP DEFAULT NOW()
);

CREATE OR REPLACE FUNCTION audit_products()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO audit_log (operation, row_id, new_values)
        VALUES ('INSERT', NEW.product_id, to_jsonb(NEW));
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_log (operation, row_id, old_values, new_values)
        VALUES ('UPDATE', NEW.product_id, to_jsonb(OLD), to_jsonb(NEW));
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO audit_log (operation, row_id, old_values)
        VALUES ('DELETE', OLD.product_id, to_jsonb(OLD));
    END IF;
    RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER products_audit
AFTER INSERT OR UPDATE OR DELETE ON products
FOR EACH ROW EXECUTE FUNCTION audit_products();

-- Step 2: Test the trigger
BEGIN;
SELECT plan(3);

INSERT INTO products (product_name, price) VALUES ('Laptop', 1000.00);

SELECT is(
    (SELECT operation FROM audit_log WHERE row_id = 1),
    'INSERT',
    'INSERT trigger fired and logged'
);

UPDATE products SET price = 900.00 WHERE product_id = 1;

SELECT is(
    (SELECT new_values->>'price' FROM audit_log WHERE operation = 'UPDATE'),
    '900.00',
    'UPDATE trigger recorded new price'
);

DELETE FROM products WHERE product_id = 1;

SELECT is(
    (SELECT old_values->>'product_name' FROM audit_log WHERE operation = 'DELETE'),
    'Laptop',
    'DELETE trigger recorded old name'
);

SELECT * FROM finish();
ROLLBACK;
```

**Expected Output**:
```
1..3
ok 1 - INSERT trigger fired and logged
ok 2 - UPDATE trigger recorded new price
ok 3 - DELETE trigger recorded old name
```

**Why This Output Occurs**: The `AFTER INSERT OR UPDATE OR DELETE` trigger fires after each operation and writes to `audit_log`. The test verifies that each operation produces the correct audit record with the expected old and new values.

### Real-World Cases

**Case 1: Compliance Audit Trail**: A financial system requires an audit log of all changes to `accounts`. Trigger tests verify that every INSERT, UPDATE, and DELETE is logged with the user, timestamp, and old/new values.

**Case 2: Updated_At Timestamp**: A `BEFORE UPDATE` trigger sets `updated_at = NOW()` on every update. Tests verify that `updated_at` changes on UPDATE but not on INSERT or SELECT.

**Case 3: Historical Snapshotting**: A trigger copies the old row to a `products_history` table on UPDATE. Tests verify that the history table contains the previous version of each product.

---

## Core Concept 4: Stored-Procedure Testing

### Definitions

**Core Definition**: Stored-procedure testing verifies that a stored procedure, given specific input parameters, produces the expected output parameters, result sets, and side effects.

**Technical Definition**: Stored-procedure testing validates input parameters (valid and invalid), output parameters, return codes, result sets, and side effects (rows inserted, updated, or deleted). Tests verify both the "happy path" and error conditions (e.g., passing a non-existent ID, violating a constraint). Procedures with `OUT` parameters require capturing those values; procedures that return result sets require comparing the result set to expected rows.

**Beginner-Friendly Explanation**: Stored-procedure testing is like testing a vending machine. You put in specific coins (input parameters) and press a button (execute). The machine should dispense the correct item (result set) and return change (output parameters). If you put in the wrong coins, it should reject them (error handling).

### Purposes

- **To** verify that procedures return correct result sets for valid inputs
- **To** verify that OUT parameters contain expected values
- **To** verify that procedures perform expected side effects (INSERT, UPDATE, DELETE)
- **To** verify that procedures handle invalid inputs with controlled errors
- **To** verify return codes for success and failure

### Syntax Rules and Structure

#### Stored Procedure (PostgreSQL)

```sql
CREATE OR REPLACE PROCEDURE transfer_funds(
    p_from_account INT,
    p_to_account INT,
    p_amount NUMERIC,
    OUT p_new_from_balance NUMERIC,
    OUT p_new_to_balance NUMERIC
)
LANGUAGE plpgsql
AS $$
BEGIN
    -- Debit source
    UPDATE accounts SET balance = balance - p_amount
    WHERE account_id = p_from_account
    RETURNING balance INTO p_new_from_balance;
    
    IF NOT FOUND THEN
        RAISE EXCEPTION 'Source account % not found', p_from_account;
    END IF;
    
    -- Credit destination
    UPDATE accounts SET balance = balance + p_amount
    WHERE account_id = p_to_account
    RETURNING balance INTO p_new_to_balance;
    
    IF NOT FOUND THEN
        RAISE EXCEPTION 'Destination account % not found', p_to_account;
    END IF;
END;
$$;
```

#### Test Structure (pgTAP)

```sql
BEGIN;
SELECT plan(4);

-- Seed accounts
INSERT INTO accounts (account_id, balance) VALUES (1, 1000.00), (2, 500.00);

-- Test 1: Procedure executes successfully
SELECT lives_ok(
    $$CALL transfer_funds(1, 2, 100.00, NULL, NULL)$$,
    'transfer_funds executes without error'
);

-- Test 2: Source balance decreased
SELECT is(
    (SELECT balance FROM accounts WHERE account_id = 1),
    900.00::numeric,
    'Source balance decreased by 100'
);

-- Test 3: Destination balance increased
SELECT is(
    (SELECT balance FROM accounts WHERE account_id = 2),
    600.00::numeric,
    'Destination balance increased by 100'
);

-- Test 4: Invalid source account raises error
SELECT throws_ok(
    $$CALL transfer_funds(999, 2, 100.00, NULL, NULL)$$,
    'P0001',  -- raise_exception
    NULL,
    'Non-existent source account raises error'
);

SELECT * FROM finish();
ROLLBACK;
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `IN` parameter | Input value passed to the procedure |
| `OUT` parameter | Value returned by the procedure |
| `INOUT` parameter | Both input and output |
| `CALL` | Executes a procedure (PostgreSQL 11+) |
| `lives_ok` | Asserts procedure executes without error |
| `throws_ok` | Asserts procedure raises expected error |

#### Syntax Rules

- Procedures are called with `CALL` (PostgreSQL) or `EXEC` (SQL Server).
- `OUT` parameters are captured via `CALL proc(..., NULL, NULL)` or by declaring variables.
- Side effects are verified by querying the affected tables after the call.
- Error conditions are tested with `throws_ok` and the expected SQLSTATE.

#### Constraints and Limitations

- PostgreSQL procedures (not functions) support transaction control; functions run inside a transaction.
- `OUT` parameter capture syntax varies between engines.
- Procedures with side effects require transaction rollback to avoid polluting the test database.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Testing a Stored Procedure with Side Effects

```sql
-- Step 1: Create the procedure and tables
CREATE TABLE accounts (
    account_id INT PRIMARY KEY,
    balance NUMERIC(10,2)
);

CREATE OR REPLACE PROCEDURE transfer_funds(
    p_from INT, p_to INT, p_amount NUMERIC
)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE accounts SET balance = balance - p_amount WHERE account_id = p_from;
    IF NOT FOUND THEN
        RAISE EXCEPTION 'Source account % not found', p_from;
    END IF;
    
    UPDATE accounts SET balance = balance + p_amount WHERE account_id = p_to;
    IF NOT FOUND THEN
        RAISE EXCEPTION 'Destination account % not found', p_to;
    END IF;
END;
$$;

-- Step 2: Test the procedure
BEGIN;
SELECT plan(4);

INSERT INTO accounts (account_id, balance) VALUES (1, 1000.00), (2, 500.00);

-- Test: successful transfer
SELECT lives_ok(
    $$CALL transfer_funds(1, 2, 100.00)$$,
    'Transfer executes successfully'
);

SELECT is(
    (SELECT balance FROM accounts WHERE account_id = 1),
    900.00::numeric,
    'Source balance is 900.00'
);

SELECT is(
    (SELECT balance FROM accounts WHERE account_id = 2),
    600.00::numeric,
    'Destination balance is 600.00'
);

-- Test: invalid source account
SELECT throws_ok(
    $$CALL transfer_funds(999, 2, 100.00)$$,
    'P0001',
    'Source account 999 not found',
    'Invalid source raises error'
);

SELECT * FROM finish();
ROLLBACK;
```

**Expected Output**:
```
1..4
ok 1 - Transfer executes successfully
ok 2 - Source balance is 900.00
ok 3 - Destination balance is 600.00
ok 4 - Invalid source raises error
```

**Why This Output Occurs**: The procedure debits account 1 and credits account 2. The test verifies the side effects (balances changed correctly) and the error handling (non-existent account raises an exception). The transaction is rolled back, restoring the original balances.

### Real-World Cases

**Case 1: Monthly Interest Calculation**: A procedure calculates and applies interest to all savings accounts. Tests verify the calculated amounts and the resulting balances.

**Case 2: Order Fulfillment**: A procedure creates an order, reserves inventory, and charges payment. Tests verify each side effect and the error handling if inventory is insufficient.

**Case 3: Batch Archiving**: A procedure moves old records to an archive table and deletes them from the main table. Tests verify the archive contains the correct rows and the main table no longer has them.

---

## Core Concept 5: Function Testing

### Definitions

**Core Definition**: Function testing verifies that scalar and table-valued functions return predictable, mathematically precise values for given inputs.

**Technical Definition**: Function testing validates deterministic functions (same input → same output), scalar functions (return a single value), table-valued functions (return a set of rows), and their behavior with edge cases (NULL, zero, negative numbers, empty strings). Because functions are pure (no side effects), they are easier to test than procedures: each test calls the function with inputs and asserts the output.

**Beginner-Friendly Explanation**: Function testing is like testing a calculator. You input 2 + 2 and expect 4. You input 5 / 0 and expect an error. Functions are predictable: same input, same output, every time.

### Purposes

- **To** verify that scalar functions return correct values for valid inputs
- **To** verify that table-valued functions return the expected rows and columns
- **To** test edge cases: NULL, zero, negative numbers, empty strings
- **To** verify that functions are deterministic (same input → same output)
- **To** verify that functions handle NULL inputs correctly

### Syntax Rules and Structure

#### Scalar Function (PostgreSQL)

```sql
CREATE OR REPLACE FUNCTION calculate_tax(
    p_subtotal NUMERIC,
    p_tax_rate NUMERIC DEFAULT 0.10
)
RETURNS NUMERIC
LANGUAGE plpgsql
IMMUTABLE
AS $$
BEGIN
    IF p_subtotal IS NULL OR p_tax_rate IS NULL THEN
        RETURN NULL;
    END IF;
    RETURN ROUND(p_subtotal * p_tax_rate, 2);
END;
$$;
```

#### Table-Valued Function (PostgreSQL)

```sql
CREATE OR REPLACE FUNCTION get_orders_above(
    p_min_total NUMERIC
)
RETURNS TABLE (order_id INT, customer_id INT, total NUMERIC)
LANGUAGE sql
STABLE
AS $$
    SELECT order_id, customer_id, total
    FROM orders
    WHERE total > p_min_total
    ORDER BY total DESC;
$$;
```

#### Test Structure (pgTAP)

```sql
BEGIN;
SELECT plan(6);

-- Test 1: Standard calculation
SELECT is(
    calculate_tax(100.00, 0.10),
    10.00::numeric,
    'Tax on 100 at 10% is 10.00'
);

-- Test 2: Default tax rate
SELECT is(
    calculate_tax(100.00),
    10.00::numeric,
    'Default tax rate is 10%'
);

-- Test 3: NULL input returns NULL
SELECT is(
    calculate_tax(NULL, 0.10),
    NULL::numeric,
    'NULL subtotal returns NULL'
);

-- Test 4: Zero subtotal
SELECT is(
    calculate_tax(0.00, 0.10),
    0.00::numeric,
    'Zero subtotal returns zero tax'
);

-- Test 5: Negative subtotal
SELECT is(
    calculate_tax(-100.00, 0.10),
    -10.00::numeric,
    'Negative subtotal returns negative tax'
);

-- Test 6: Rounding precision
SELECT is(
    calculate_tax(99.99, 0.075),
    7.50::numeric,
    'Rounding to 2 decimal places'
);

SELECT * FROM finish();
ROLLBACK;
```

#### Component Breakdown

| Function Type | Returns | Test Approach |
|---------------|---------|---------------|
| Scalar | Single value | `is(function(input), expected)` |
| Table-valued | Rows | `results_eq('SELECT * FROM function(input)', expected)` |
| Deterministic | Same output for same input | Call twice, compare |
| IMMUTABLE | No database reads | Safe to call in any context |
| STABLE | Reads database, no writes | May return different values between statements |

#### Syntax Rules

- Scalar functions return a single value; test with `is()`.
- Table-valued functions return rows; test with `results_eq()`.
- `IMMUTABLE` functions can be called in indexes and are ideal for testing.
- `STABLE` functions read the database but do not write.
- `VOLATILE` functions can have side effects and are harder to test.

#### Constraints and Limitations

- Floating-point functions require tolerance for comparison.
- `IMMUTABLE` functions cannot read the database; if they do, they are mislabeled.
- Table-valued functions may return different results if the underlying data changes.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Testing a Scalar Function

```sql
-- Step 1: Create the function
CREATE OR REPLACE FUNCTION calculate_discount(
    p_price NUMERIC,
    p_discount_pct NUMERIC
)
RETURNS NUMERIC
LANGUAGE plpgsql
IMMUTABLE
AS $$
BEGIN
    IF p_price IS NULL OR p_discount_pct IS NULL THEN
        RETURN NULL;
    END IF;
    IF p_discount_pct < 0 OR p_discount_pct > 100 THEN
        RAISE EXCEPTION 'Discount percentage must be between 0 and 100';
    END IF;
    RETURN ROUND(p_price * (1 - p_discount_pct / 100.0), 2);
END;
$$;

-- Step 2: Test the function
BEGIN;
SELECT plan(6);

SELECT is(calculate_discount(100.00, 10), 90.00::numeric, '10% off 100 is 90');
SELECT is(calculate_discount(100.00, 0), 100.00::numeric, '0% off is no change');
SELECT is(calculate_discount(100.00, 100), 0.00::numeric, '100% off is free');
SELECT is(calculate_discount(99.99, 15), 84.99::numeric, '15% off 99.99 is 84.99');
SELECT is(calculate_discount(NULL, 10), NULL::numeric, 'NULL price returns NULL');
SELECT throws_ok(
    $$SELECT calculate_discount(100.00, 150)$$,
    'P0001',
    'Discount percentage must be between 0 and 100',
    'Invalid discount raises error'
);

SELECT * FROM finish();
ROLLBACK;
```

**Expected Output**:
```
1..6
ok 1 - 10% off 100 is 90
ok 2 - 0% off is no change
ok 3 - 100% off is free
ok 4 - 15% off 99.99 is 84.99
ok 5 - NULL price returns NULL
ok 6 - Invalid discount raises error
```

**Why This Output Occurs**: The function computes the discounted price for valid inputs, returns NULL for NULL inputs, and raises an exception for out-of-range discount percentages. Each test asserts the expected output.

#### Example 2: Testing a Table-Valued Function

```sql
-- Step 1: Create the function
CREATE OR REPLACE FUNCTION get_products_in_range(
    p_min_price NUMERIC,
    p_max_price NUMERIC
)
RETURNS TABLE (product_id INT, product_name TEXT, price NUMERIC)
LANGUAGE sql
STABLE
AS $$
    SELECT product_id, product_name, price
    FROM products
    WHERE price BETWEEN p_min_price AND p_max_price
    ORDER BY price;
$$;

-- Step 2: Seed and test
BEGIN;
SELECT plan(3);

INSERT INTO products (product_id, product_name, price) VALUES
    (1, 'Mouse', 25.00),
    (2, 'Keyboard', 75.00),
    (3, 'Monitor', 300.00),
    (4, 'Laptop', 1200.00);

-- Test 1: Range returns expected rows
SELECT results_eq(
    $$SELECT product_id, product_name, price FROM get_products_in_range(50, 500)$$,
    $$VALUES (2, 'Keyboard'::text, 75.00::numeric), (3, 'Monitor'::text, 300.00::numeric)$$,
    'Range 50-500 returns Keyboard and Monitor'
);

-- Test 2: Empty range returns no rows
SELECT is_empty(
    $$SELECT * FROM get_products_in_range(5000, 10000)$$,
    'Range above all prices returns no rows'
);

-- Test 3: Exact boundary included
SELECT results_eq(
    $$SELECT product_id FROM get_products_in_range(25.00, 25.00)$$,
    $$VALUES (1)$$,
    'Exact boundary price is included'
);

SELECT * FROM finish();
ROLLBACK;
```

**Expected Output**:
```
1..3
ok 1 - Range 50-500 returns Keyboard and Monitor
ok 2 - Range above all prices returns no rows
ok 3 - Exact boundary price is included
```

**Why This Output Occurs**: The table-valued function returns rows within the price range. `results_eq` compares the actual result set to the expected values. `is_empty` verifies that an out-of-range query returns no rows.

### Real-World Cases

**Case 1: Tax Calculation**: A `calculate_tax()` function is tested with various subtotals and tax rates, including zero, negative, and NULL inputs.

**Case 2: Age Calculation**: A `calculate_age(birth_date)` function is tested with boundary dates (today, yesterday, one year ago, leap year).

**Case 3: Product Search**: A table-valued function `search_products(term)` is tested with matching, non-matching, and empty search terms.

---

## Core Concept 6: Transaction Testing

### Definitions

**Core Definition**: Transaction testing verifies that a series of operations either all succeed (COMMIT) or all fail (ROLLBACK), leaving the database in a consistent state.

**Technical Definition**: Transaction testing validates atomicity by forcing a failure partway through a multi-step transaction and verifying that all prior changes are rolled back. Tests also verify isolation (concurrent transactions do not see each other's uncommitted changes), durability (committed changes survive a restart), and consistency (constraints are satisfied before and after the transaction).

**Beginner-Friendly Explanation**: Transaction testing is like testing a bank transfer. If the debit succeeds but the credit fails, the debit must be undone. The test forces the credit to fail and verifies the debit was rolled back.

### Purposes

- **To** verify that a failure mid-transaction rolls back all prior changes
- **To** verify that a successful transaction commits all changes
- **To** verify that concurrent transactions respect isolation levels
- **To** verify that constraints are satisfied at transaction boundaries

### Syntax Rules and Structure

#### Transaction Structure (PostgreSQL)

```sql
BEGIN;
-- Step 1: Debit source
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
-- Step 2: Credit destination
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
-- Step 3: If both succeed, commit
COMMIT;
-- If any step fails, ROLLBACK is automatic (or explicit)
```

#### Test Structure (pgTAP) — Forcing a Rollback

```sql
BEGIN;
SELECT plan(2);

-- Seed accounts
INSERT INTO accounts (account_id, balance) VALUES (1, 1000.00), (2, 500.00);

-- Attempt a transaction that fails partway
DO $$
BEGIN
    -- Debit source
    UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
    
    -- Force a failure: try to credit a non-existent account
    UPDATE accounts SET balance = balance + 100 WHERE account_id = 999;
    
    -- This RAISE simulates the application detecting the failure
    IF NOT FOUND THEN
        RAISE EXCEPTION 'Destination account not found';
    END IF;
EXCEPTION
    WHEN OTHERS THEN
        -- The exception handler rolls back the entire DO block
        RAISE;
END;
$$;

-- Test 1: Source balance unchanged (rolled back)
SELECT is(
    (SELECT balance FROM accounts WHERE account_id = 1),
    1000.00::numeric,
    'Source balance rolled back after failure'
);

-- Test 2: Destination balance unchanged
SELECT is(
    (SELECT balance FROM accounts WHERE account_id = 2),
    500.00::numeric,
    'Destination balance unchanged'
);

SELECT * FROM finish();
ROLLBACK;
```

#### Component Breakdown

| Concept | Description |
|---------|-------------|
| `BEGIN` / `START TRANSACTION` | Starts a transaction |
| `COMMIT` | Makes all changes permanent |
| `ROLLBACK` | Undoes all changes since BEGIN |
| `SAVEPOINT` | Creates a point to roll back to |
| `ROLLBACK TO SAVEPOINT` | Rolls back to a savepoint |
| `EXCEPTION` handler | Catches errors and rolls back the block |

#### Syntax Rules

- A transaction begins with `BEGIN` and ends with `COMMIT` or `ROLLBACK`.
- If a statement fails, subsequent statements in the same transaction are aborted unless a `SAVEPOINT` is used.
- `ROLLBACK TO SAVEPOINT` allows partial rollback within a transaction.
- In PostgreSQL, an exception handler in a `DO` block or function rolls back the block's changes.

#### Constraints and Limitations

- DDL statements (CREATE, ALTER, DROP) are transactional in PostgreSQL but not in MySQL (implicit commit).
- Some databases auto-commit DDL; test transaction behavior for each engine.
- Nested transactions are emulated with savepoints, not true nesting.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Testing Atomicity with a Forced Failure

```sql
-- Step 1: Create tables
CREATE TABLE accounts (
    account_id INT PRIMARY KEY,
    balance NUMERIC(10,2) NOT NULL
);

-- Step 2: Seed data
INSERT INTO accounts (account_id, balance) VALUES (1, 1000.00), (2, 500.00);

-- Step 3: Test atomicity
BEGIN;
SELECT plan(3);

-- Attempt transfer with a failure in the middle
DO $$
BEGIN
    UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
    
    -- Simulate a constraint violation on the second update
    UPDATE accounts SET balance = balance + 100 WHERE account_id = 999;
    
    IF NOT FOUND THEN
        RAISE EXCEPTION 'Account 999 not found';
    END IF;
EXCEPTION
    WHEN OTHERS THEN
        RAISE NOTICE 'Transaction failed: %', SQLERRM;
END;
$$;

-- Test 1: Source balance is unchanged (rolled back)
SELECT is(
    (SELECT balance FROM accounts WHERE account_id = 1),
    1000.00::numeric,
    'Source balance rolled back'
);

-- Test 2: Destination balance is unchanged
SELECT is(
    (SELECT balance FROM accounts WHERE account_id = 2),
    500.00::numeric,
    'Destination balance unchanged'
);

-- Test 3: Total balance is conserved
SELECT is(
    (SELECT SUM(balance) FROM accounts),
    1500.00::numeric,
    'Total balance conserved'
);

SELECT * FROM finish();
ROLLBACK;
```

**Expected Output**:
```
NOTICE:  Transaction failed: Account 999 not found
1..3
ok 1 - Source balance rolled back
ok 2 - Destination balance unchanged
ok 3 - Total balance conserved
```

**Why This Output Occurs**: The `DO` block attempts to debit account 1 and credit account 999. The second update matches no rows, so `NOT FOUND` is true, and `RAISE EXCEPTION` throws an error. The exception handler catches it, but the entire `DO` block is rolled back. The source balance remains 1000.00, and the total balance is unchanged at 1500.00.

#### Example 2: Testing COMMIT Persistence

```sql
BEGIN;
SELECT plan(2);

-- Successful transaction
DO $$
BEGIN
    UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
END;
$$;

-- Test 1: Both updates applied
SELECT is(
    (SELECT balance FROM accounts WHERE account_id = 1),
    900.00::numeric,
    'Source balance decreased'
);

SELECT is(
    (SELECT balance FROM accounts WHERE account_id = 2),
    600.00::numeric,
    'Destination balance increased'
);

SELECT * FROM finish();
ROLLBACK;  -- Rollback the test transaction, not the DO block
```

**Why This Output Occurs**: The `DO` block executes both updates successfully (no exception), so both changes are applied within the test transaction. The test verifies the new balances. The outer `ROLLBACK` undoes the test's changes, keeping the database clean.

### Real-World Cases

**Case 1: Order Processing**: An order transaction inserts an order, updates inventory, and charges payment. A forced failure in payment must roll back the order and inventory changes.

**Case 2: Batch Import**: A batch import transaction inserts 1,000 rows. A failure on row 500 must roll back rows 1–499.

**Case 3: Concurrent Transfer**: Two concurrent transfers from the same account are tested to verify that the final balance is correct (no lost updates).

---

## Core Concept 7: Database Unit Testing Frameworks

### Definitions

**Core Definition**: Database unit testing frameworks are tools that provide assertion functions, test runners, and reporting for testing database objects directly within the database.

**Technical Definition**: Database unit testing frameworks include **pgTAP** (PostgreSQL), **tSQLt** (SQL Server), and **MyTAP** (MySQL). These frameworks provide a TAP (Test Anything Protocol) or xUnit-style interface with functions like `plan()`, `ok()`, `is()`, `throws_ok()`, `lives_ok()`, `results_eq()`, and `finish()`. Tests run inside transactions that roll back, ensuring isolation. Frameworks integrate with CI/CD pipelines and provide machine-readable output.

**Beginner-Friendly Explanation**: Database unit testing frameworks are like JUnit or pytest, but for SQL. Instead of testing Java or Python code, you write SQL tests that verify constraints, triggers, procedures, and functions. The framework runs the tests, reports pass/fail, and rolls back changes so the database stays clean.

### Purposes

- **To** provide assertion functions for SQL-based testing (`ok`, `is`, `throws_ok`, `lives_ok`)
- **To** run tests in isolated transactions that roll back
- **To** integrate database tests with CI/CD pipelines
- **To** produce machine-readable test output (TAP or JUnit XML)

### Syntax Rules and Structure

#### pgTAP (PostgreSQL)

```sql
-- Install pgTAP
CREATE EXTENSION pgtap;

-- Test file structure
BEGIN;
SELECT plan(3);

SELECT ok(true, 'This test passes');
SELECT is(2 + 2, 4, 'Math works');
SELECT throws_ok(
    $$INSERT INTO products (product_name, price) VALUES (NULL, 10)$$,
    '23502',
    NULL,
    'NOT NULL constraint works'
);

SELECT * FROM finish();
ROLLBACK;
```

#### tSQLt (SQL Server)

```sql
-- Install tSQLt
-- Execute tSQLt.class.sql in the target database

-- Create a test class
EXEC tSQLt.NewTestClass 'ProductTests';
GO

CREATE PROCEDURE ProductTests.[test NOT NULL constraint rejects NULL]
AS
BEGIN
    -- Arrange
    EXEC tSQLt.FakeTable 'dbo.Products';
    
    -- Act & Assert
    EXEC tSQLt.ExpectException @ExpectedMessage = 'Cannot insert the value NULL';
    INSERT INTO Products (ProductName, Price) VALUES (NULL, 10.00);
END;
GO

-- Run tests
EXEC tSQLt.Run 'ProductTests';
```

#### MyTAP (MySQL)

```sql
-- Install MyTAP
-- Source the MyTAP.sql file

-- Test structure
BEGIN;
SELECT plan(2);

SELECT ok(1 = 1, 'Basic assertion');
SELECT is(2 + 2, 4, 'Math works');

SELECT * FROM finish();
ROLLBACK;
```

#### Component Breakdown

| Function | Purpose | Framework |
|----------|---------|-----------|
| `plan(n)` | Declares the number of tests | pgTAP, MyTAP |
| `ok(boolean, msg)` | Asserts a boolean is true | pgTAP, MyTAP |
| `is(actual, expected, msg)` | Asserts equality | pgTAP, MyTAP |
| `throws_ok(sql, sqlstate, msg, desc)` | Asserts an error is raised | pgTAP |
| `lives_ok(sql, desc)` | Asserts no error is raised | pgTAP |
| `results_eq(sql, expected, desc)` | Compares result sets | pgTAP |
| `is_empty(sql, desc)` | Asserts no rows returned | pgTAP |
| `tSQLt.ExpectException` | Asserts an exception | tSQLt |
| `tSQLt.FakeTable` | Creates a table fake | tSQLt |
| `tSQLt.Run` | Runs test class | tSQLt |

#### Syntax Rules

- Tests run inside a transaction that is rolled back at the end.
- `plan(n)` must match the actual number of tests; otherwise, the test file fails.
- `finish()` outputs the TAP summary and verifies the plan.
- tSQLt uses stored procedures for tests and a schema-based organization.
- MyTAP is a port of pgTAP for MySQL/MariaDB.

#### Constraints and Limitations

- pgTAP requires the `pgtap` extension; MyTAP requires sourcing the framework SQL.
- tSQLt requires installing the framework in the target database.
- Some frameworks do not support all SQL dialects (e.g., pgTAP is PostgreSQL-specific).
- Transaction rollback may not work for DDL in MySQL (implicit commit).

### Annotated Complete Step-by-Step Code Examples

#### Example 1: pgTAP Test Suite for a Products Table

```sql
-- Step 1: Install pgTAP
CREATE EXTENSION IF NOT EXISTS pgtap;

-- Step 2: Create the test suite
BEGIN;
SELECT plan(7);

-- Setup: create a test table
CREATE TABLE test_products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price NUMERIC(10,2) CHECK (price > 0),
    sku VARCHAR(50) UNIQUE,
    status VARCHAR(20) DEFAULT 'active'
);

-- Test 1: Table exists
SELECT has_table('test_products', 'test_products table exists');

-- Test 2: Column exists
SELECT has_column('test_products', 'product_name', 'product_name column exists');

-- Test 3: NOT NULL constraint
SELECT throws_ok(
    $$INSERT INTO test_products (product_name, price) VALUES (NULL, 10)$$,
    '23502',
    NULL,
    'NOT NULL rejects NULL'
);

-- Test 4: CHECK constraint
SELECT throws_ok(
    $$INSERT INTO test_products (product_name, price) VALUES ('Test', -5)$$,
    '23514',
    NULL,
    'CHECK rejects negative price'
);

-- Test 5: UNIQUE constraint
INSERT INTO test_products (product_name, price, sku) VALUES ('A', 10, 'SKU1');
SELECT throws_ok(
    $$INSERT INTO test_products (product_name, price, sku) VALUES ('B', 20, 'SKU1')$$,
    '23505',
    NULL,
    'UNIQUE rejects duplicate SKU'
);

-- Test 6: DEFAULT value
INSERT INTO test_products (product_name, price) VALUES ('Default Test', 10);
SELECT is(
    (SELECT status FROM test_products WHERE product_name = 'Default Test'),
    'active',
    'DEFAULT value applied'
);

-- Test 7: Valid insert succeeds
SELECT lives_ok(
    $$INSERT INTO test_products (product_name, price, sku) VALUES ('Valid', 99.99, 'SKU2')$$,
    'Valid insert succeeds'
);

SELECT * FROM finish();
ROLLBACK;
```

**Expected Output**:
```
1..7
ok 1 - test_products table exists
ok 2 - product_name column exists
ok 3 - NOT NULL rejects NULL
ok 4 - CHECK rejects negative price
ok 5 - UNIQUE rejects duplicate SKU
ok 6 - DEFAULT value applied
ok 7 - Valid insert succeeds
```

**Why This Output Occurs**: Each `SELECT` statement runs an assertion. `has_table` and `has_column` verify schema structure. `throws_ok` verifies constraint rejection. `is` verifies the default value. `lives_ok` verifies a valid insert. All tests pass, and the transaction is rolled back.

#### Example 2: tSQLt Test for SQL Server

```sql
-- Step 1: Install tSQLt (run tSQLt.class.sql in the database)

-- Step 2: Create a test class
EXEC tSQLt.NewTestClass 'OrderTests';
GO

-- Step 3: Create a test procedure
CREATE PROCEDURE OrderTests.[test total is calculated correctly]
AS
BEGIN
    -- Arrange: create a fake table
    EXEC tSQLt.FakeTable 'dbo.OrderItems';
    
    INSERT INTO OrderItems (OrderId, Quantity, UnitPrice)
    VALUES (1, 2, 25.00), (1, 3, 20.00);
    
    -- Act: calculate total
    DECLARE @total DECIMAL(10,2);
    SELECT @total = SUM(Quantity * UnitPrice) FROM OrderItems WHERE OrderId = 1;
    
    -- Assert: total is 110.00
    EXEC tSQLt.AssertEquals 110.00, @total;
END;
GO

-- Step 4: Run the test
EXEC tSQLt.Run 'OrderTests';
```

**Expected Output**:
```
OrderTests.[test total is calculated correctly] ... Success
```

**Why This Output Occurs**: `tSQLt.FakeTable` creates an isolated copy of `OrderItems`. The test inserts known data, computes the total, and asserts it equals 110.00. The test passes because 2×25 + 3×20 = 110. tSQLt rolls back the fake table after the test.

### Real-World Cases

**Case 1: CI/CD Database Testing**: pgTAP tests run on every pull request that modifies the schema, catching constraint and trigger regressions before deployment.

**Case 2: SQL Server Migration**: tSQLt tests verify that stored procedures and functions behave correctly after a SQL Server version upgrade.

**Case 3: MySQL Constraint Testing**: MyTAP tests verify that foreign-key cascades and check constraints work as expected after a MySQL upgrade.

---

## References

| Name | Link |
|------|------|
| pgTAP Documentation | https://pgtap.org/documentation.html |
| tSQLt — Unit Testing for SQL Server | https://tsqlt.org/ |
| MyTAP — Unit Testing for MySQL | https://github.com/hepabolu/mytap |
| PostgreSQL Documentation — CREATE TRIGGER | https://www.postgresql.org/docs/current/sql-createtrigger.html |
| PostgreSQL Documentation — CREATE PROCEDURE | https://www.postgresql.org/docs/current/sql-createprocedure.html |
| PostgreSQL Documentation — CREATE FUNCTION | https://www.postgresql.org/docs/current/sql-createfunction.html |
| PostgreSQL Documentation — Foreign Keys | https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-FK |
| PostgreSQL Documentation — Transactions | https://www.postgresql.org/docs/current/tutorial-transactions.html |
| Microsoft Learn — CREATE TRIGGER (Transact-SQL) | https://learn.microsoft.com/en-us/sql/t-sql/statements/create-trigger-transact-sql |
| Microsoft Learn — CREATE PROCEDURE (Transact-SQL) | https://learn.microsoft.com/en-us/sql/t-sql/statements/create-procedure-transact-sql |
| MySQL 8.0 Reference Manual — CREATE TRIGGER | https://dev.mysql.com/doc/refman/8.0/en/create-trigger.html |
| MySQL 8.0 Reference Manual — CREATE PROCEDURE | https://dev.mysql.com/doc/refman/8.0/en/create-procedure.html |
| MySQL 8.0 Reference Manual — FOREIGN KEY Constraints | https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html |
| Test Anything Protocol (TAP) Specification | https://testanything.org/ |
| OWASP — Database Testing | https://owasp.org/www-project-web-security-testing-guide/ |
| Redgate — Unit Testing SQL Server with tSQLt | https://www.red-gate.com/simple-talk/databases/sql-server/ |