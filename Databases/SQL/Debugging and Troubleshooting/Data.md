# SQL Data Debugging: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**: SQL data debugging is the systematic process of identifying, diagnosing, and correcting data-level defects—duplicates, orphans, invalid references, inconsistent values, unexpected NULLs, stale derived data, and truncation/overflow failures—that violate business rules or integrity constraints.

**Technical Definition**: Data debugging encompasses duplicate detection and removal (finding structurally identical rows that violate implicit uniqueness rules), orphaned record cleanup (child rows whose parent rows were deleted without `ON DELETE CASCADE`), foreign key validation (diagnosing constraint violations when inserting non-existent parent IDs), value inconsistency resolution (trimming whitespace, normalizing case, correcting corrupted strings), NULL auditing (identifying where empty values bypassed application validation into nullable columns), derived data reconciliation (debugging stale cache tables, denormalized columns, and materialized views), and truncation/overflow diagnosis (string values exceeding `VARCHAR(n)` limits or numeric values overflowing integer types).

**Beginner-Friendly Explanation**: Data debugging is like being a detective for your database's contents. The structure (tables, columns) is fine, but the data inside is wrong: there are duplicate customer records, orders pointing to deleted customers, names with extra spaces, missing values where they shouldn't be, and totals that don't match the underlying line items. Data debugging finds and fixes these problems.

### Key Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Silent Corruption** | Data defects don't raise errors; they produce wrong results |
| **Business-Rule Dependent** | What counts as a "duplicate" depends on business rules, not just schema |
| **Cascading Impact** | Orphans and stale derived data propagate errors downstream |
| **Detection via Queries** | Finding defects requires anti-joins, `GROUP BY HAVING`, and constraint queries |
| **Prevention vs. Cleanup** | Constraints prevent future defects; cleanup fixes existing ones |
| **Vendor-Specific Behavior** | Truncation behavior varies: MySQL warns, PostgreSQL errors, SQL Server errors |

### Prerequisites

- **Schema Knowledge**: Understanding of primary keys, foreign keys, unique constraints, and `ON DELETE` behavior
- **Business Rules**: Knowledge of what constitutes a valid record (e.g., email uniqueness, status transitions)
- **Data Profiling Tools**: Ability to run aggregate queries and anti-joins
- **Backup**: A verified backup before any data modification (especially deletions)
- **Constraint Catalog Access**: Ability to query `information_schema` or system catalogs

### Related Programming Areas

- **Data Quality Engineering**: Profiling, cleansing, and monitoring data quality
- **ETL/ELT Development**: Preventing defects during data loading
- **Application Development**: Validating data before insertion
- **Database Administration**: Constraint management, cascade configuration
- **Analytics Engineering**: Ensuring derived tables and materialized views are accurate

### Core Concepts Overview

SQL data debugging comprises seven complementary categories:

1. **Duplicate Data**: Finding and removing structural duplicates violating implicit business rules
2. **Orphaned Records**: Cleaning up child rows after parent deletion without cascade
3. **Invalid Foreign Keys**: Diagnosing constraint errors from non-existent parent IDs
4. **Inconsistent Values**: Handling corruption, trailing whitespace, and case variations
5. **Unexpected NULLs**: Identifying where empty values slipped past application logic
6. **Incorrect Derived Data**: Debugging stale caches, denormalized fields, and materialized views
7. **Truncation & Overflow Failures**: Troubleshooting string and numeric limit violations

---

## Core Concept 1: Duplicate Data

### Definitions

**Core Definition**: Duplicate data occurs when two or more rows represent the same real-world entity, violating an implicit or explicit uniqueness rule.

**Technical Definition**: Structural duplicates are rows where all business-key columns have identical values. They arise from missing `UNIQUE` constraints, failed idempotency in ETL jobs, race conditions in application logic (check-then-insert), or data imports. Detection uses `GROUP BY ... HAVING COUNT(*) > 1`; removal uses `DELETE ... USING` (PostgreSQL), self-join deletion (MySQL), or `ROW_NUMBER()` with `CTE` (SQL Server, PostgreSQL 8.4+).

**Beginner-Friendly Explanation**: Duplicate data is like having two customer records for the same person—same name, same email, same address. The database doesn't know they're the same person because there's no constraint preventing it. You need to find these duplicates and merge or delete them.

### Purposes

- **To** detect duplicates by grouping on business-key columns
- **To** remove duplicates while preserving one canonical row
- **To** prevent future duplicates with `UNIQUE` constraints
- **To** reconcile foreign-key references when merging duplicate parent rows

### Syntax Rules and Structure

#### Detection: GROUP BY HAVING

```sql
SELECT 
    email,
    COUNT(*) AS duplicate_count
FROM customers
GROUP BY email
HAVING COUNT(*) > 1
ORDER BY duplicate_count DESC;
```

#### Removal: ROW_NUMBER() with CTE (PostgreSQL, SQL Server)

```sql
WITH ranked AS (
    SELECT 
        customer_id,
        email,
        ROW_NUMBER() OVER (
            PARTITION BY email 
            ORDER BY customer_id  -- Keep the lowest ID
        ) AS rn
    FROM customers
)
DELETE FROM customers
WHERE customer_id IN (
    SELECT customer_id FROM ranked WHERE rn > 1
);
```

#### Removal: Self-Join (MySQL)

```sql
DELETE c1
FROM customers c1
INNER JOIN customers c2
    ON c1.email = c2.email
   AND c1.customer_id > c2.customer_id;
```

#### Prevention: UNIQUE Constraint

```sql
ALTER TABLE customers
ADD CONSTRAINT uq_customers_email UNIQUE (email);
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `GROUP BY` | Groups rows by the business key |
| `HAVING COUNT(*) > 1` | Filters to groups with duplicates |
| `ROW_NUMBER() OVER (PARTITION BY ...)` | Assigns a rank within each duplicate group |
| `rn > 1` | Selects all but the first row in each group |
| `UNIQUE` constraint | Prevents future duplicates |

#### Syntax Rules

- Duplicate detection requires identifying the **business key** (the columns that should be unique).
- `ROW_NUMBER()` with `PARTITION BY` is the safest deletion method; it preserves one row per group.
- Self-join deletion works in MySQL but can be slow on large tables.
- `UNIQUE` constraints can only be added after duplicates are removed.

#### Constraints and Limitations

- Case sensitivity and trailing whitespace affect duplicate detection; normalize first.
- `UNIQUE` constraints with NULLs allow multiple NULLs in most engines (PostgreSQL, MySQL, SQL Server).
- Deleting duplicates may break foreign-key references; update references before deletion.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Detecting and Removing Duplicate Customers

```sql
-- Step 1: Create the table and insert duplicates
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    email VARCHAR(255),
    customer_name VARCHAR(100)
);

INSERT INTO customers (email, customer_name) VALUES
    ('alice@example.com', 'Alice'),
    ('bob@example.com', 'Bob'),
    ('alice@example.com', 'Alice Smith'),  -- Duplicate email
    ('charlie@example.com', 'Charlie'),
    ('bob@example.com', 'Bob Jones');      -- Duplicate email

-- Step 2: Find duplicates
SELECT 
    email,
    COUNT(*) AS duplicate_count,
    ARRAY_AGG(customer_id) AS customer_ids
FROM customers
GROUP BY email
HAVING COUNT(*) > 1
ORDER BY duplicate_count DESC;
```

**Expected Output**:
```
      email         | duplicate_count | customer_ids 
--------------------+-----------------+--------------
 alice@example.com  |               2 | {1,3}
 bob@example.com    |               2 | {2,5}
```

```sql
-- Step 3: Remove duplicates, keeping the lowest customer_id
WITH ranked AS (
    SELECT 
        customer_id,
        ROW_NUMBER() OVER (
            PARTITION BY email 
            ORDER BY customer_id
        ) AS rn
    FROM customers
)
DELETE FROM customers
WHERE customer_id IN (
    SELECT customer_id FROM ranked WHERE rn > 1
);
-- Expected: DELETE 2

-- Step 4: Verify no duplicates remain
SELECT email, COUNT(*) FROM customers GROUP BY email HAVING COUNT(*) > 1;
-- Expected: 0 rows

-- Step 5: Prevent future duplicates
ALTER TABLE customers
ADD CONSTRAINT uq_customers_email UNIQUE (email);
-- Expected: ALTER TABLE
```

**Why This Output Occurs**: Step 2 groups by `email` and counts occurrences. The `HAVING COUNT(*) > 1` filter returns only emails with duplicates. Step 3 uses `ROW_NUMBER()` to assign rank 1 to the lowest `customer_id` in each group and deletes all rows with rank > 1. Step 5 adds a `UNIQUE` constraint to prevent recurrence.

### Real-World Cases

**Case 1: ETL Idempotency Failure**: A nightly ETL job re-inserts records instead of upserting them, creating duplicates. The fix is to use `INSERT ... ON CONFLICT DO UPDATE` (PostgreSQL) or `MERGE` (SQL Server, Oracle).

**Case 2: Race Condition in Application**: Two concurrent requests check for an existing user, find none, and both insert. The fix is a `UNIQUE` constraint plus handling the duplicate-key exception.

**Case 3: Data Import Without Deduplication**: A CSV import inserts rows without checking for existing records. The fix is to deduplicate before import and add a `UNIQUE` constraint.

---

## Core Concept 2: Orphaned Records

### Definitions

**Core Definition**: Orphaned records are child rows whose parent row has been deleted, leaving a foreign-key value that points to a non-existent parent.

**Technical Definition**: Orphans arise when a parent row is deleted without `ON DELETE CASCADE` and without manually deleting or reassigning child rows. They violate referential integrity but are invisible if no foreign-key constraint exists. Detection uses `LEFT JOIN ... WHERE parent.id IS NULL` or `NOT EXISTS`. Cleanup options include deleting orphans, reassigning them to a valid parent, or adding the missing parent.

**Beginner-Friendly Explanation**: An orphaned record is like a library book catalog entry for a book that was removed from the shelf. The entry says "book #123 is on shelf 5," but book #123 no longer exists. The catalog entry is an orphan.

### Purposes

- **To** detect child rows with no matching parent
- **To** clean up orphans before adding foreign-key constraints
- **To** decide whether to delete orphans, reassign them, or restore the parent
- **To** prevent future orphans with `ON DELETE CASCADE` or `ON DELETE SET NULL`

### Syntax Rules and Structure

#### Detection: LEFT JOIN with IS NULL

```sql
SELECT 
    o.order_id,
    o.customer_id
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;
```

#### Detection: NOT EXISTS

```sql
SELECT 
    o.order_id,
    o.customer_id
FROM orders o
WHERE NOT EXISTS (
    SELECT 1 FROM customers c WHERE c.customer_id = o.customer_id
);
```

#### Cleanup: Delete Orphans

```sql
DELETE FROM orders o
WHERE NOT EXISTS (
    SELECT 1 FROM customers c WHERE c.customer_id = o.customer_id
);
```

#### Prevention: ON DELETE CASCADE

```sql
ALTER TABLE orders
ADD CONSTRAINT fk_orders_customer
FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
ON DELETE CASCADE;
```

#### Prevention: ON DELETE SET NULL

```sql
ALTER TABLE orders
ADD CONSTRAINT fk_orders_customer
FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
ON DELETE SET NULL;
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `LEFT JOIN ... WHERE parent.id IS NULL` | Finds child rows with no matching parent |
| `NOT EXISTS` | Alternative anti-join detection |
| `ON DELETE CASCADE` | Deletes child rows when parent is deleted |
| `ON DELETE SET NULL` | Sets child foreign key to NULL when parent is deleted |
| `ON DELETE RESTRICT` | Prevents parent deletion if children exist |

#### Syntax Rules

- `LEFT JOIN ... IS NULL` and `NOT EXISTS` are semantically equivalent; `NOT EXISTS` often performs better.
- `ON DELETE CASCADE` is the strictest option; `ON DELETE SET NULL` preserves child rows.
- Adding a foreign-key constraint fails if orphans exist; clean them first.
- `ON DELETE RESTRICT` (default in most engines) prevents deletion, forcing explicit child cleanup.

#### Constraints and Limitations

- `ON DELETE CASCADE` can cause unintended mass deletions if not carefully designed.
- `ON DELETE SET NULL` requires the foreign-key column to be nullable.
- Orphans may be intentional in some schemas (e.g., soft-deleted parents); verify business rules.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Detecting and Cleaning Orphaned Orders

```sql
-- Step 1: Create tables without foreign-key constraint
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100)
);
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,  -- No FK constraint
    total NUMERIC(10,2)
);

-- Step 2: Insert data with orphans
INSERT INTO customers (customer_name) VALUES ('Alice'), ('Bob');
INSERT INTO orders (customer_id, total) VALUES
    (1, 100.00),
    (2, 200.00),
    (999, 300.00),  -- Orphan: customer 999 doesn't exist
    (888, 400.00);  -- Orphan

-- Step 3: Detect orphans
SELECT 
    o.order_id,
    o.customer_id,
    o.total
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;
```

**Expected Output**:
```
 order_id | customer_id | total  
----------+-------------+--------
        3 |         999 | 300.00
        4 |         888 | 400.00
```

```sql
-- Step 4: Clean up orphans
DELETE FROM orders o
WHERE NOT EXISTS (
    SELECT 1 FROM customers c WHERE c.customer_id = o.customer_id
);
-- Expected: DELETE 2

-- Step 5: Add foreign-key constraint to prevent future orphans
ALTER TABLE orders
ADD CONSTRAINT fk_orders_customer
FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
ON DELETE CASCADE;
-- Expected: ALTER TABLE

-- Step 6: Verify constraint works
DELETE FROM customers WHERE customer_id = 2;
-- Expected: DELETE 1 (and the order for customer 2 is also deleted)
```

**Why This Output Occurs**: Step 3's anti-join reveals orders 3 and 4 with non-existent customer IDs. Step 4 deletes them. Step 5 adds a foreign-key constraint with `ON DELETE CASCADE`, ensuring that future parent deletions automatically remove child rows. Step 6 demonstrates the cascade in action.

### Real-World Cases

**Case 1: Legacy Data Migration**: A legacy database has orphaned orders because parent customers were deleted without cascade. The fix detects orphans, reassigns them to a "unknown customer" record, then adds a foreign-key constraint.

**Case 2: Soft Delete Without Cascade**: An application soft-deletes customers (sets `deleted_at`) but doesn't update orders. The fix either cascades the soft delete or uses `ON DELETE SET NULL`.

**Case 3: Bulk Parent Deletion**: A DBA deletes old customers without checking for child orders. The fix runs an orphan detection query before deletion and either cascades or archives child rows.

---

## Core Concept 3: Invalid Foreign Keys

### Definitions

**Core Definition**: Invalid foreign keys are constraint violations that occur when an INSERT or UPDATE references a parent row that does not exist.

**Technical Definition**: A foreign-key constraint enforces that every non-NULL value in the child column matches a value in the parent's referenced column. Violations raise SQLSTATE 23503 (PostgreSQL), ERROR 1452 (MySQL), or error 547 (SQL Server). Invalid foreign keys arise from application bugs (using a stale parent ID), data imports (referencing parents not yet loaded), or race conditions (parent deleted between child insert and commit).

**Beginner-Friendly Explanation**: An invalid foreign key is like trying to check out a library book using a library card number that doesn't exist. The system rejects the checkout because the card isn't in the database.

### Purposes

- **To** diagnose foreign-key constraint errors and identify the missing parent
- **To** determine whether the parent should be created, the child corrected, or the reference nullified
- **To** load data in the correct order (parents before children)
- **To** handle constraint violations gracefully in application code

### Syntax Rules and Structure

#### Diagnosing the Error (PostgreSQL)

```sql
-- Attempt insert with invalid FK
INSERT INTO orders (customer_id, total) VALUES (999, 100.00);
```

**Expected Error**:
```
ERROR:  insert or update on table "orders" violates foreign key constraint "fk_orders_customer"
DETAIL:  Key (customer_id)=(999) is not present in table "customers".
SQLSTATE: 23503
```

#### Finding the Missing Parent

```sql
SELECT o.order_id, o.customer_id
FROM orders o
WHERE o.customer_id = 999
  AND NOT EXISTS (SELECT 1 FROM customers c WHERE c.customer_id = 999);
-- Or simply:
SELECT * FROM customers WHERE customer_id = 999;
-- Expected: 0 rows
```

#### Finding All Invalid References (Before Adding Constraint)

```sql
SELECT DISTINCT o.customer_id AS missing_customer_id
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL
  AND o.customer_id IS NOT NULL;
```

#### Handling in Application Code (Java)

```java
try {
    jdbcTemplate.update(
        "INSERT INTO orders (customer_id, total) VALUES (?, ?)",
        customerId, total);
} catch (DataIntegrityViolationException e) {
    if (e.getRootCause() instanceof SQLException) {
        SQLException se = (SQLException) e.getRootCause();
        if ("23503".equals(se.getSQLState())) {
            throw new BusinessError("Customer " + customerId + " does not exist");
        }
    }
    throw e;
}
```

#### Component Breakdown

| Error Code | Meaning | Fix |
|------------|---------|-----|
| 23503 (PostgreSQL) | Foreign key violation | Create parent or fix child |
| 1452 (MySQL) | Cannot add or update child row | Create parent or fix child |
| 547 (SQL Server) | FK constraint conflict | Create parent or fix child |

#### Syntax Rules

- Foreign-key violations occur at INSERT/UPDATE time, not at constraint creation time.
- `DEFERRABLE` constraints (PostgreSQL) can be checked at transaction commit instead of statement time.
- `ON DELETE SET NULL` and `ON DELETE CASCADE` prevent orphans but do not prevent invalid inserts.
- Applications should catch the constraint violation and return a meaningful error.

#### Constraints and Limitations

- Deferring constraints can hide data-quality issues until commit.
- Bulk loads must order parent inserts before child inserts.
- Some ETL tools disable constraints during load; re-enable and validate afterward.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Diagnosing and Fixing an Invalid Foreign Key

```sql
-- Step 1: Create tables with FK constraint
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100)
);
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    total NUMERIC(10,2)
);

-- Step 2: Insert a customer
INSERT INTO customers (customer_name) VALUES ('Alice');  -- customer_id = 1

-- Step 3: Attempt to insert an order with an invalid customer_id
INSERT INTO orders (customer_id, total) VALUES (999, 100.00);
```

**Expected Error (PostgreSQL)**:
```
ERROR:  insert or update on table "orders" violates foreign key constraint "orders_customer_id_fkey"
DETAIL:  Key (customer_id)=(999) is not present in table "customers".
SQLSTATE: 23503
```

```sql
-- Step 4: Verify the parent doesn't exist
SELECT * FROM customers WHERE customer_id = 999;
-- Expected: 0 rows

-- Step 5: Fix by creating the parent first
INSERT INTO customers (customer_id, customer_name) VALUES (999, 'New Customer');
INSERT INTO orders (customer_id, total) VALUES (999, 100.00);
-- Expected: INSERT 0 1
```

**Why This Output Occurs**: Step 3 fails because customer 999 does not exist. The error message includes the constraint name (`orders_customer_id_fkey`), the offending value (`999`), and the referenced table (`customers`). Step 5 creates the parent first, then the child insert succeeds.

#### Example 2: Finding All Invalid References Before Adding a Constraint

```sql
-- Step 1: Find all orders with invalid customer references
SELECT 
    o.order_id,
    o.customer_id AS invalid_customer_id
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL
  AND o.customer_id IS NOT NULL;
```

**Expected Output**:
```
 order_id | invalid_customer_id 
----------+---------------------
        3 |                 999
        4 |                 888
```

```sql
-- Step 2: Create a placeholder parent for orphaned references
INSERT INTO customers (customer_id, customer_name)
SELECT DISTINCT customer_id, 'Unknown (recovered)'
FROM orders o
WHERE NOT EXISTS (SELECT 1 FROM customers c WHERE c.customer_id = o.customer_id)
  AND o.customer_id IS NOT NULL
ON CONFLICT (customer_id) DO NOTHING;

-- Step 3: Now add the foreign-key constraint
ALTER TABLE orders
ADD CONSTRAINT fk_orders_customer
FOREIGN KEY (customer_id) REFERENCES customers(customer_id);
-- Expected: ALTER TABLE
```

**Why This Output Occurs**: Step 1 identifies all invalid references. Step 2 creates placeholder parent rows for the missing customer IDs, preserving the child data while satisfying referential integrity. Step 3 adds the constraint successfully because no orphans remain.

### Real-World Cases

**Case 1: ETL Load Order**: A nightly ETL job loads orders before customers, causing foreign-key violations. The fix reorders the load to load customers first.

**Case 2: Stale Parent ID in Application**: An application caches customer IDs and uses a stale ID after the customer is deleted. The fix validates the parent exists before insert or handles the constraint violation gracefully.

**Case 3: Data Import Without Parent**: A CSV import references customers not yet loaded. The fix creates placeholder parent rows or defers the constraint check until after all data is loaded.

---

## Core Concept 4: Inconsistent Values

### Definitions

**Core Definition**: Inconsistent values are data values that represent the same real-world concept but differ in formatting, casing, whitespace, or encoding.

**Technical Definition**: Value inconsistencies include leading/trailing whitespace (`' Alice '` vs. `'Alice'`), mixed case (`'ALICE'`, `'Alice'`, `'alice'`), non-breaking spaces (`\u00A0`), zero-width characters, Unicode normalization differences (`'é'` as U+00E9 vs. `'e'` + U+0301), and corrupted characters from encoding mismatches. Detection uses `TRIM`, `LOWER`, `UPPER`, and pattern matching; cleanup uses `UPDATE ... SET column = TRIM(LOWER(column))`.

**Beginner-Friendly Explanation**: Inconsistent values are like writing the same name in different ways: "Alice", "alice", "ALICE", " Alice ". They all mean the same person, but the database treats them as different values. Data debugging normalizes these variations.

### Purposes

- **To** detect whitespace, case, and Unicode inconsistencies
- **To** normalize values to a canonical form
- **To** prevent future inconsistencies with `CHECK` constraints or application-level normalization
- **To** compare values correctly in joins and filters

### Syntax Rules and Structure

#### Detection: Whitespace

```sql
SELECT 
    customer_id,
    '[' || customer_name || ']' AS value_with_brackets,
    LENGTH(customer_name) AS raw_length,
    LENGTH(TRIM(customer_name)) AS trimmed_length
FROM customers
WHERE customer_name <> TRIM(customer_name);
```

#### Detection: Case Inconsistency

```sql
SELECT 
    LOWER(customer_name) AS normalized,
    COUNT(DISTINCT customer_name) AS variant_count,
    ARRAY_AGG(DISTINCT customer_name) AS variants
FROM customers
GROUP BY LOWER(customer_name)
HAVING COUNT(DISTINCT customer_name) > 1;
```

#### Cleanup: Normalize Values

```sql
UPDATE customers
SET customer_name = TRIM(customer_name);

UPDATE customers
SET email = LOWER(TRIM(email));

-- Unicode normalization (PostgreSQL)
UPDATE customers
SET customer_name = NORMALIZE(customer_name, NFC);
```

#### Prevention: CHECK Constraint

```sql
ALTER TABLE customers
ADD CONSTRAINT chk_customer_name_trimmed
CHECK (customer_name = TRIM(customer_name));

ALTER TABLE customers
ADD CONSTRAINT chk_email_lowercase
CHECK (email = LOWER(email));
```

#### Component Breakdown

| Function | Purpose | Example |
|----------|---------|---------|
| `TRIM()` | Remove leading/trailing spaces | `TRIM(' Alice ')` → `'Alice'` |
| `LOWER()` | Convert to lowercase | `LOWER('ALICE')` → `'alice'` |
| `UPPER()` | Convert to uppercase | `UPPER('alice')` → `'ALICE'` |
| `NORMALIZE()` | Unicode normalization | `NORMALIZE('é', NFC)` |
| `REPLACE()` | Replace characters | `REPLACE(name, CHR(160), ' ')` |

#### Syntax Rules

- `TRIM()` removes only spaces by default; use `TRIM(BOTH FROM ...)` for explicit behavior.
- `LOWER()` and `UPPER()` are locale-dependent for some characters.
- `NORMALIZE()` is PostgreSQL-specific; other engines may require application-level normalization.
- `CHECK` constraints prevent future inconsistencies but require existing data to be clean first.

#### Constraints and Limitations

- Case-insensitive collations (e.g., `utf8mb4_general_ci`) make comparisons case-insensitive without normalization.
- Unicode normalization can change string length; test carefully.
- `CHECK` constraints add write overhead.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Detecting and Normalizing Inconsistent Names

```sql
-- Step 1: Create table with inconsistent data
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100),
    email VARCHAR(255)
);
INSERT INTO customers (customer_name, email) VALUES
    ('Alice', 'alice@example.com'),
    (' Alice ', 'ALICE@example.com'),
    ('ALICE', 'alice@EXAMPLE.com'),
    ('Bob', 'bob@example.com'),
    ('bob ', 'BOB@example.com');

-- Step 2: Detect whitespace inconsistencies
SELECT 
    customer_id,
    '[' || customer_name || ']' AS name_with_brackets
FROM customers
WHERE customer_name <> TRIM(customer_name);
```

**Expected Output**:
```
 customer_id | name_with_brackets 
-------------+--------------------
           2 | [ Alice ]
           5 | [bob ]
```

```sql
-- Step 3: Detect case inconsistencies
SELECT 
    LOWER(customer_name) AS normalized_name,
    COUNT(DISTINCT customer_name) AS variant_count,
    ARRAY_AGG(DISTINCT customer_name) AS variants
FROM customers
GROUP BY LOWER(customer_name)
HAVING COUNT(DISTINCT customer_name) > 1;
```

**Expected Output**:
```
 normalized_name | variant_count | variants             
-----------------+---------------+----------------------
 alice           |             3 | {Alice," Alice ",ALICE}
 bob             |             2 | {Bob,"bob "}
```

```sql
-- Step 4: Normalize names and emails
UPDATE customers
SET customer_name = TRIM(customer_name);

UPDATE customers
SET email = LOWER(TRIM(email));

-- Step 5: Verify normalization
SELECT customer_id, customer_name, email FROM customers ORDER BY customer_id;
```

**Expected Output**:
```
 customer_id | customer_name | email             
-------------+---------------+-------------------
           1 | Alice         | alice@example.com
           2 | Alice         | alice@example.com
           3 | ALICE         | alice@example.com
           4 | Bob           | bob@example.com
           5 | bob           | bob@example.com
```

**Why This Output Occurs**: Step 2 reveals whitespace issues. Step 3 reveals case variations. Step 4 trims whitespace and lowercases emails. Note that names are still case-inconsistent (`Alice`, `ALICE`, `bob`) because the update only trimmed whitespace, not case. To fully normalize, apply `LOWER()` or `INITCAP()` to names based on business rules.

### Real-World Cases

**Case 1: Email Case Sensitivity**: An application stores emails in mixed case, causing login failures when users type lowercase. The fix normalizes all emails to lowercase and adds a `CHECK` constraint.

**Case 2: Trailing Whitespace in Joins**: A join between `customers.name` and `orders.customer_name` fails because one has trailing spaces. The fix uses `TRIM()` in the join condition or normalizes the data.

**Case 3: Unicode Normalization**: A user's name is stored in NFC form in one system and NFD in another, causing duplicate detection to fail. The fix applies `NORMALIZE()` on both sides.

---

## Core Concept 5: Unexpected NULLs

### Definitions

**Core Definition**: Unexpected NULLs are NULL values in columns where the application or business rule requires a non-NULL value.

**Technical Definition**: Unexpected NULLs arise from application bugs (failing to validate required fields), ETL issues (missing source data), `LEFT JOIN` results (no matching right-side row), or `ON DELETE SET NULL` cascades. Detection queries use `IS NULL` on columns that should be populated. Prevention uses `NOT NULL` constraints, `CHECK` constraints, and application-level validation.

**Beginner-Friendly Explanation**: An unexpected NULL is like a form field that was left blank when it should have been filled in. The database allows NULL because the column is nullable, but the business rule says the value is required.

### Purposes

- **To** identify columns with unexpected NULL values
- **To** determine whether NULLs are legitimate (unknown) or defects (missing data)
- **To** backfill missing values from other sources or defaults
- **To** prevent future NULLs with `NOT NULL` constraints

### Syntax Rules and Structure

#### Detection: Count NULLs per Column

```sql
SELECT
    COUNT(*) AS total_rows,
    COUNT(*) FILTER (WHERE email IS NULL) AS null_emails,
    COUNT(*) FILTER (WHERE customer_name IS NULL) AS null_names,
    COUNT(*) FILTER (WHERE created_at IS NULL) AS null_created
FROM customers;
```

#### Detection: NULLs from LEFT JOIN

```sql
SELECT
    o.order_id,
    o.customer_id,
    c.customer_name  -- NULL when no matching customer
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_name IS NULL
  AND o.customer_id IS NOT NULL;  -- Exclude legitimate no-customer cases
```

#### Backfill: COALESCE from Another Column

```sql
UPDATE customers
SET email = COALESCE(email, 'unknown@example.com')
WHERE email IS NULL;
```

#### Backfill: From Another Table

```sql
UPDATE orders o
SET customer_name = c.customer_name
FROM customers c
WHERE o.customer_id = c.customer_id
  AND o.customer_name IS NULL;
```

#### Prevention: NOT NULL Constraint

```sql
ALTER TABLE customers
ALTER COLUMN email SET NOT NULL;
```

#### Component Breakdown

| Technique | Purpose |
|-----------|---------|
| `COUNT(*) FILTER (WHERE col IS NULL)` | Count NULLs per column |
| `LEFT JOIN ... WHERE right.col IS NULL` | Find missing matches |
| `COALESCE(col, default)` | Substitute NULL with default |
| `UPDATE ... FROM` | Backfill from another table |
| `NOT NULL` constraint | Prevent future NULLs |

#### Syntax Rules

- `COUNT(column)` ignores NULLs; `COUNT(*)` counts all rows.
- `COALESCE` returns the first non-NULL argument.
- `NOT NULL` constraints require all existing rows to be non-NULL before they can be added.
- `LEFT JOIN` produces NULLs for unmatched right-side columns.

#### Constraints and Limitations

- NULLs are legitimate for "unknown" values; not all NULLs are defects.
- `NOT NULL` constraints add write overhead and may break existing application code.
- Backfilling requires a reliable source for the missing values.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Detecting and Backfilling Unexpected NULLs

```sql
-- Step 1: Create table with nullable columns
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100),
    email VARCHAR(255),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Step 2: Insert data with unexpected NULLs
INSERT INTO customers (customer_name, email) VALUES
    ('Alice', 'alice@example.com'),
    ('Bob', NULL),           -- Missing email
    (NULL, 'charlie@example.com'),  -- Missing name
    ('Diana', NULL);         -- Missing email

-- Step 3: Count NULLs per column
SELECT
    COUNT(*) AS total_rows,
    COUNT(*) FILTER (WHERE customer_name IS NULL) AS null_names,
    COUNT(*) FILTER (WHERE email IS NULL) AS null_emails,
    COUNT(*) FILTER (WHERE created_at IS NULL) AS null_created
FROM customers;
```

**Expected Output**:
```
 total_rows | null_names | null_emails | null_created 
------------+------------+-------------+--------------
          4 |          1 |           2 |            0
```

```sql
-- Step 4: Backfill missing emails with a placeholder
UPDATE customers
SET email = 'unknown@example.com'
WHERE email IS NULL;
-- Expected: UPDATE 2

-- Step 5: Backfill missing names from another source (or set a default)
UPDATE customers
SET customer_name = 'Unknown Customer'
WHERE customer_name IS NULL;
-- Expected: UPDATE 1

-- Step 6: Verify no NULLs remain
SELECT COUNT(*) FROM customers WHERE email IS NULL OR customer_name IS NULL;
-- Expected: 0

-- Step 7: Prevent future NULLs
ALTER TABLE customers
ALTER COLUMN email SET NOT NULL,
ALTER COLUMN customer_name SET NOT NULL;
-- Expected: ALTER TABLE
```

**Why This Output Occurs**: Step 3 reveals 1 NULL name and 2 NULL emails. Steps 4–5 backfill the NULLs with placeholders. Step 7 adds `NOT NULL` constraints to prevent future NULLs. The constraints succeed because all existing rows are now non-NULL.

### Real-World Cases

**Case 1: LEFT JOIN Produces NULLs**: A report uses `LEFT JOIN` and shows NULL customer names for orders without matching customers. The fix uses `COALESCE(c.customer_name, 'Unknown')`.

**Case 2: Application Validation Gap**: A form allows submitting an empty email, which becomes NULL in the database. The fix adds `NOT NULL` and application-level validation.

**Case 3: ON DELETE SET NULL Cascade**: Deleting a customer sets `orders.customer_id` to NULL, creating unexpected NULLs. The fix uses `ON DELETE CASCADE` or `ON DELETE RESTRICT` instead.

---

## Core Concept 6: Incorrect Derived Data

### Definitions

**Core Definition**: Incorrect derived data occurs when cached, denormalized, or materialized values do not match the source data they are derived from.

**Technical Definition**: Derived data includes denormalized columns (e.g., `order_total` stored on `orders` instead of computed from `order_items`), cache tables (e.g., `customer_order_counts`), and materialized views. Staleness arises from failed refresh jobs, bugs in the refresh logic, or source changes that bypass the refresh mechanism. Detection compares derived values against recomputed values; reconciliation updates the derived data.

**Beginner-Friendly Explanation**: Derived data is like a summary of a book. If the book is edited but the summary isn't updated, the summary is wrong. Incorrect derived data means the summary (cache, denormalized field) doesn't match the book (source data).

### Purposes

- **To** detect discrepancies between derived data and source data
- **To** identify the cause of staleness (failed refresh, logic bug, missed trigger)
- **To** reconcile derived data with source data
- **To** prevent future staleness with reliable refresh mechanisms

### Syntax Rules and Structure

#### Detection: Denormalized Column Mismatch

```sql
SELECT
    o.order_id,
    o.order_total AS stored_total,
    SUM(oi.quantity * oi.unit_price) AS computed_total,
    o.order_total - SUM(oi.quantity * oi.unit_price) AS difference
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY o.order_id, o.order_total
HAVING o.order_total <> SUM(oi.quantity * oi.unit_price);
```

#### Detection: Cache Table Mismatch

```sql
SELECT
    c.customer_id,
    c.customer_name,
    c.order_count AS cached_count,
    COUNT(o.order_id) AS actual_count
FROM customer_order_counts c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name, c.order_count
HAVING c.order_count <> COUNT(o.order_id);
```

#### Reconciliation: Update Denormalized Column

```sql
UPDATE orders o
SET order_total = sub.computed_total
FROM (
    SELECT order_id, SUM(quantity * unit_price) AS computed_total
    FROM order_items
    GROUP BY order_id
) sub
WHERE o.order_id = sub.order_id
  AND o.order_total <> sub.computed_total;
```

#### Reconciliation: Refresh Materialized View

```sql
-- PostgreSQL
REFRESH MATERIALIZED VIEW CONCURRENTLY order_summary;

-- MySQL: no materialized views; use a summary table with scheduled refresh
-- SQL Server: indexed views are maintained automatically
```

#### Component Breakdown

| Derived Data Type | Detection | Reconciliation |
|-------------------|-----------|----------------|
| Denormalized column | Compare stored vs. computed | `UPDATE ... FROM` |
| Cache table | Compare cache vs. source count | `TRUNCATE` + `INSERT` |
| Materialized view | Compare view vs. base tables | `REFRESH MATERIALIZED VIEW` |
| Trigger-maintained | Compare trigger result vs. source | Fix trigger, re-run |

#### Syntax Rules

- Derived data must have a single source of truth; only the source should be authoritative.
- Reconciliation should be idempotent (safe to run multiple times).
- Materialized view refresh can be `CONCURRENTLY` (PostgreSQL) to avoid locking.
- Triggers that maintain derived data must fire on all relevant operations (INSERT, UPDATE, DELETE).

#### Constraints and Limitations

- Materialized views can be stale between refreshes; define acceptable staleness.
- Denormalized columns add write complexity and risk of inconsistency.
- Cache tables require invalidation logic; failures cause staleness.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Detecting and Fixing Denormalized Order Totals

```sql
-- Step 1: Create tables with denormalized total
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    order_total NUMERIC(10,2)  -- Denormalized
);
CREATE TABLE order_items (
    item_id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(order_id),
    quantity INT,
    unit_price NUMERIC(10,2)
);

-- Step 2: Insert data with inconsistent total
INSERT INTO orders (customer_id, order_total) VALUES (1, 100.00);
INSERT INTO order_items (order_id, quantity, unit_price) VALUES
    (1, 2, 25.00),   -- 50.00
    (1, 3, 20.00);   -- 60.00
-- Actual total: 110.00, but stored total is 100.00

-- Step 3: Detect mismatches
SELECT
    o.order_id,
    o.order_total AS stored_total,
    SUM(oi.quantity * oi.unit_price) AS computed_total,
    o.order_total - SUM(oi.quantity * oi.unit_price) AS difference
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY o.order_id, o.order_total
HAVING o.order_total <> SUM(oi.quantity * oi.unit_price);
```

**Expected Output**:
```
 order_id | stored_total | computed_total | difference 
----------+--------------+----------------+------------
        1 |       100.00 |         110.00 |     -10.00
```

```sql
-- Step 4: Reconcile the denormalized total
UPDATE orders o
SET order_total = sub.computed_total
FROM (
    SELECT order_id, SUM(quantity * unit_price) AS computed_total
    FROM order_items
    GROUP BY order_id
) sub
WHERE o.order_id = sub.order_id
  AND o.order_total <> sub.computed_total;
-- Expected: UPDATE 1

-- Step 5: Verify reconciliation
SELECT order_id, order_total FROM orders;
-- Expected: 1 | 110.00
```

**Why This Output Occurs**: Step 3 computes the actual total from `order_items` and compares it to the stored `order_total`. The difference of -10.00 reveals the discrepancy. Step 4 updates the denormalized column to match the computed value.

### Real-World Cases

**Case 1: Failed Cache Refresh**: A nightly job refreshes a customer order count cache but fails silently. The cache becomes stale. The fix detects mismatches and adds monitoring for refresh failures.

**Case 2: Trigger Bug**: A trigger updates `order_total` on INSERT of `order_items` but not on UPDATE or DELETE. The fix corrects the trigger to handle all operations.

**Case 3: Materialized View Staleness**: A materialized view is refreshed daily but the business needs hourly data. The fix changes the refresh schedule or uses `REFRESH MATERIALIZED VIEW CONCURRENTLY` more frequently.

---

## Core Concept 7: Truncation & Overflow Failures

### Definitions

**Core Definition**: Truncation and overflow failures occur when a value exceeds the maximum size of its target column type, causing silent truncation, a warning, or an error.

**Technical Definition**: String truncation occurs when a value longer than `VARCHAR(n)` is inserted. MySQL raises a warning (or error in strict mode), PostgreSQL raises an error (`value too long for type character varying(n)`), and SQL Server raises an error (`String or binary data would be truncated`). Numeric overflow occurs when an integer exceeds its range (e.g., `INT` max 2,147,483,647; `SMALLINT` max 32,767) or a decimal exceeds its precision. Detection queries compare `LENGTH(column)` to the column definition or check for values near the type limit.

**Beginner-Friendly Explanation**: Truncation is like trying to fit a 10-foot ladder into a 6-foot truck. The ladder either gets cut (truncated) or the loading fails (error). Overflow is like a car odometer that rolls over from 999,999 to 000,000 — the number doesn't fit anymore.

### Purposes

- **To** diagnose truncation and overflow errors from INSERT/UPDATE statements
- **To** detect existing values that are near or exceed column limits
- **To** choose appropriate column types (`VARCHAR(n)`, `TEXT`, `BIGINT`, `NUMERIC(p,s)`)
- **To** handle truncation gracefully in application code

### Syntax Rules and Structure

#### Detection: String Length Near Limit

```sql
-- PostgreSQL: find values approaching VARCHAR(100) limit
SELECT 
    customer_id,
    LENGTH(customer_name) AS name_length,
    customer_name
FROM customers
WHERE LENGTH(customer_name) > 90  -- 90% of limit
ORDER BY name_length DESC;
```

#### Detection: Numeric Values Near Overflow

```sql
-- PostgreSQL: find integers approaching INT max
SELECT 
    order_id,
    quantity,
    quantity::BIGINT AS quantity_bigint
FROM order_items
WHERE quantity > 2000000000  -- Near INT max (2,147,483,647)
ORDER BY quantity DESC;
```

#### Detection: Column Definitions

```sql
-- PostgreSQL: list column types and limits
SELECT 
    table_name,
    column_name,
    data_type,
    character_maximum_length,
    numeric_precision,
    numeric_scale
FROM information_schema.columns
WHERE table_schema = 'public'
  AND table_name = 'customers'
ORDER BY ordinal_position;
```

#### Handling: Safe Truncation

```sql
-- PostgreSQL: truncate before insert
INSERT INTO customers (customer_name)
VALUES (LEFT('Very long name that exceeds the limit', 100));

-- MySQL: enable strict mode to error instead of truncate
SET sql_mode = 'STRICT_ALL_TABLES';
```

#### Component Breakdown

| Type | Range/Limit | Overflow Behavior |
|------|-------------|-------------------|
| `SMALLINT` | -32,768 to 32,767 | Error on overflow |
| `INT` | -2,147,483,648 to 2,147,483,647 | Error on overflow |
| `BIGINT` | -9.2 × 10^18 to 9.2 × 10^18 | Error on overflow |
| `VARCHAR(n)` | n characters | Error (PostgreSQL, SQL Server strict) or truncation (MySQL non-strict) |
| `NUMERIC(p,s)` | p digits total, s after decimal | Error on overflow |
| `TEXT` | Unlimited (PostgreSQL) | No truncation |

#### Syntax Rules

- PostgreSQL always errors on string truncation; it never silently truncates.
- MySQL in non-strict mode silently truncates with a warning; strict mode errors.
- SQL Server errors on truncation by default.
- Numeric overflow always errors; there is no silent truncation for numbers.
- Use `LEFT(string, n)` to safely truncate strings before insert.

#### Constraints and Limitations

- Silent truncation (MySQL non-strict) can cause data loss without errors.
- Widening a column (`VARCHAR(50)` → `VARCHAR(255)`) is a metadata-only change in PostgreSQL 9.2+.
- Changing `INT` to `BIGINT` requires a table rewrite in some engines.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Diagnosing String Truncation

```sql
-- Step 1: Create table with VARCHAR(20) limit
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(20)
);

-- Step 2: Attempt to insert a longer value (PostgreSQL)
INSERT INTO products (product_name) VALUES ('This product name is too long');
```

**Expected Error (PostgreSQL)**:
```
ERROR:  value too long for type character varying(20)
SQLSTATE: 22001
```

**Expected Behavior (MySQL non-strict)**:
```
Query OK, 1 row affected, 1 warning
-- The value is silently truncated to 'This product name is'
```

```sql
-- Step 3: Check column definition
SELECT 
    column_name,
    data_type,
    character_maximum_length
FROM information_schema.columns
WHERE table_name = 'products' AND column_name = 'product_name';
```

**Expected Output**:
```
 column_name  | data_type         | character_maximum_length 
--------------+-------------------+--------------------------
 product_name | character varying |                       20
```

```sql
-- Step 4: Fix by widening the column or truncating the value
ALTER TABLE products ALTER COLUMN product_name TYPE VARCHAR(100);
INSERT INTO products (product_name) VALUES ('This product name is too long');
-- Expected: INSERT 0 1
```

**Why This Output Occurs**: PostgreSQL rejects the insert because the value exceeds `VARCHAR(20)`. MySQL non-strict mode truncates the value with a warning. Step 4 widens the column to `VARCHAR(100)`, allowing the full value.

#### Example 2: Diagnosing Numeric Overflow

```sql
-- Step 1: Create table with SMALLINT column
CREATE TABLE inventory (
    item_id SERIAL PRIMARY KEY,
    quantity SMALLINT  -- Max 32,767
);

-- Step 2: Attempt to insert a value exceeding SMALLINT
INSERT INTO inventory (quantity) VALUES (40000);
```

**Expected Error (PostgreSQL)**:
```
ERROR:  smallint out of range
SQLSTATE: 22003
```

```sql
-- Step 3: Check column type
SELECT 
    column_name,
    data_type,
    numeric_precision
FROM information_schema.columns
WHERE table_name = 'inventory' AND column_name = 'quantity';
```

**Expected Output**:
```
 column_name | data_type | numeric_precision 
-------------+-----------+-------------------
 quantity    | smallint  |                16
```

```sql
-- Step 4: Fix by widening the column to INTEGER
ALTER TABLE inventory ALTER COLUMN quantity TYPE INTEGER;
INSERT INTO inventory (quantity) VALUES (40000);
-- Expected: INSERT 0 1
```

**Why This Output Occurs**: The `SMALLINT` type cannot hold 40,000 (max 32,767), causing an overflow error. Step 4 changes the column to `INTEGER`, which supports values up to 2,147,483,647.

### Real-World Cases

**Case 1: Name Field Too Short**: A `VARCHAR(50)` name column truncates long international names. The fix widens to `VARCHAR(255)` or `TEXT`.

**Case 2: Quantity Overflow**: A `SMALLINT` quantity column overflows when an order exceeds 32,767 units. The fix changes to `INTEGER` or `BIGINT`.

**Case 3: Silent Truncation in MySQL**: MySQL non-strict mode silently truncates a 100-character value to 50 characters. The fix enables `STRICT_ALL_TABLES` to catch the error.

---

## References

| Name | Link |
|------|------|
| PostgreSQL Documentation — Constraints | https://www.postgresql.org/docs/current/ddl-constraints.html |
| PostgreSQL Documentation — Data Types | https://www.postgresql.org/docs/current/datatype.html |
| PostgreSQL Documentation — Foreign Keys | https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-FK |
| MySQL 8.0 Reference Manual — FOREIGN KEY Constraints | https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html |
| MySQL 8.0 Reference Manual — Data Type Storage Requirements | https://dev.mysql.com/doc/refman/8.0/en/storage-requirements.html |
| Microsoft Learn — CREATE TABLE (Foreign Key Constraints) | https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql |
| Microsoft Learn — Data Type Conversion | https://learn.microsoft.com/en-us/sql/t-sql/data-types/data-type-conversion-database-engine |
| PostgreSQL Wiki — Don't Do This | https://wiki.postgresql.org/wiki/Don%27t_Do_This |
| Use The Index, Luke — Data Types and Indexes | https://use-the-index-luke.com/sql/where-clause/obfuscation |
| Redgate — Finding and Removing Duplicate Data | https://www.red-gate.com/simple-talk/databases/sql-server/ |
| Percona — MySQL Data Truncation and Strict Mode | https://www.percona.com/blog/ |
| PostgreSQL — Materialized Views | https://www.postgresql.org/docs/current/rules-materializedviews.html |
| PostgreSQL — Routine Vacuuming (Dead Tuples) | https://www.postgresql.org/docs/current/routine-vacuuming.html |
| Microsoft Learn — Indexed Views | https://learn.microsoft.com/en-us/sql/relational-databases/views/create-indexed-views |