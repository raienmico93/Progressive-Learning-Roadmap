# SQL Data Quality Testing: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**: SQL data quality testing is the systematic process of validating that data stored in a database meets defined quality dimensions—completeness, accuracy, consistency, uniqueness, validity, and referential integrity—and that automated frameworks enforce these rules continuously.

**Technical Definition**: Data quality testing encompasses completeness checks (verifying that required records or columns have no critical gaps), accuracy validation (ensuring computed columns, summaries, and mathematical formulas align precisely with raw inputs), consistency verification (checking that mirrored or denormalized values match across database nodes or data warehouses), uniqueness enforcement (ensuring primary keys and unique indexes prevent duplicate identity states), validity matching (confirming column values adhere to business formatting rules like phone syntax, email patterns, or postal codes), referential integrity validation (verifying that all active child items connect to verifiable parent identifiers), automated data quality frameworks (Great Expectations, dbt tests, Soda), and CI/CD schema validation (automating migration testing to verify that schema updates do not silently break existing software features).

**Beginner-Friendly Explanation**: Data quality testing is like quality control at a factory. You check that boxes are full (completeness), labels are correct (accuracy), the same product matches across warehouses (consistency), no two boxes have the same serial number (uniqueness), labels follow the standard format (validity), every part fits into an assembly (referential integrity), and automated robots do these checks on every production line (frameworks), with final inspection before shipping (CI/CD).

### Key Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Dimension-Based** | Six core dimensions: completeness, accuracy, consistency, uniqueness, validity, referential integrity |
| **Rule-Driven** | Each dimension is tested through specific SQL queries or framework assertions |
| **Continuous** | Data quality must be monitored continuously, not just at deployment |
| **Business-Aligned** | Rules reflect business expectations, not just schema constraints |
| **Automated** | Modern frameworks (dbt, Great Expectations) run tests on every pipeline execution |
| **Migration-Safe** | CI/CD schema validation catches breaking changes before production |

### Prerequisites

- **Data Quality Rules**: Documented business rules for each dimension (e.g., "email must be non-NULL and match RFC 5322")
- **Test Database**: A database with production-like data for validation
- **Framework**: dbt, Great Expectations, Soda, or custom SQL scripts
- **CI/CD Pipeline**: Automation to run tests on every schema migration or data load
- **Monitoring Stack**: Alerting on data quality failures
- **Data Catalog**: Knowledge of which columns are critical and what rules apply

### Related Programming Areas

- **Data Engineering**: ETL/ELT pipeline quality gates
- **Analytics Engineering**: dbt tests, data contracts, metrics validation
- **Database Administration**: Constraint enforcement, statistics, monitoring
- **Application Development**: Input validation, data integrity at the source
- **Compliance and Governance**: Data quality SLAs, audit trails

### Core Concepts Overview

Data quality testing comprises eight complementary categories:

1. **Completeness**: Verifying required records or columns have no critical gaps
2. **Accuracy**: Ensuring computed columns align precisely with raw inputs
3. **Consistency**: Checking mirrored or denormalized values match across nodes
4. **Uniqueness**: Ensuring primary keys and unique indexes prevent duplicates
5. **Validity**: Matching column states against business formatting rules
6. **Referential Integrity**: Verifying child items connect to parent identifiers
7. **Automated Data Quality Frameworks**: Great Expectations, dbt tests
8. **CI/CD Schema Validation**: Automating migration testing

---

## Core Concept 1: Completeness

### Definitions

**Core Definition**: Completeness testing verifies that required records exist and required columns contain non-NULL, non-empty values.

**Technical Definition**: Completeness encompasses row-level completeness (all expected rows are present; no missing records), column-level completeness (required columns have no NULLs or empty strings), and referential completeness (all expected foreign-key relationships exist). Detection queries count NULLs, empty strings, and missing rows against expected counts. Metrics include completeness percentage (`non_null_count / total_count`).

**Beginner-Friendly Explanation**: Completeness testing is like checking that every form submitted by a customer has all required fields filled in. If the "email" field is blank or the "country" field is missing, the form is incomplete.

### Purposes

- **To** detect missing required values in critical columns
- **To** verify that expected rows are present after ETL loads
- **To** quantify completeness as a percentage for SLAs
- **To** trigger alerts when completeness drops below thresholds

### Syntax Rules and Structure

#### Column-Level Completeness

```sql
SELECT
    COUNT(*) AS total_rows,
    COUNT(email) AS non_null_emails,
    COUNT(*) - COUNT(email) AS null_emails,
    ROUND(100.0 * COUNT(email) / COUNT(*), 2) AS completeness_pct
FROM customers;
```

#### Row-Level Completeness (Expected vs. Actual)

```sql
-- Compare source and target row counts
SELECT
    (SELECT COUNT(*) FROM source_orders) AS source_count,
    (SELECT COUNT(*) FROM target_orders) AS target_count,
    (SELECT COUNT(*) FROM source_orders) - (SELECT COUNT(*) FROM target_orders) AS missing_rows;
```

#### Multi-Column Completeness

```sql
SELECT
    COUNT(*) AS total,
    COUNT(*) FILTER (WHERE email IS NULL) AS null_email,
    COUNT(*) FILTER (WHERE phone IS NULL) AS null_phone,
    COUNT(*) FILTER (WHERE address IS NULL) AS null_address,
    COUNT(*) FILTER (WHERE email IS NULL OR phone IS NULL OR address IS NULL) AS any_null
FROM customers;
```

#### Component Breakdown

| Metric | Formula | Interpretation |
|--------|---------|----------------|
| Column completeness | `COUNT(col) / COUNT(*)` | Percentage of non-NULL values |
| Row completeness | `target_count / source_count` | Percentage of expected rows present |
| Any-null count | `COUNT(*) FILTER (WHERE col1 IS NULL OR col2 IS NULL)` | Rows missing at least one required value |

#### Syntax Rules

- `COUNT(column)` ignores NULLs; `COUNT(*)` counts all rows.
- Empty strings (`''`) are not NULL; check for both (`col IS NULL OR col = ''`).
- Completeness thresholds should be defined per column (e.g., email 100%, phone 90%).
- Row completeness requires a source of truth for expected counts.

#### Constraints and Limitations

- 100% completeness is not always achievable (e.g., optional phone numbers).
- Completeness does not guarantee accuracy; a non-NULL value can still be wrong.
- `COUNT(*)` on large tables can be slow; use approximate counts or sampling for monitoring.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Column Completeness Audit

```sql
-- Step 1: Create a customers table with mixed completeness
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    email VARCHAR(255),
    phone VARCHAR(20),
    address TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

INSERT INTO customers (email, phone, address) VALUES
    ('alice@example.com', '555-1234', '123 Main St'),
    ('bob@example.com', NULL, '456 Oak Ave'),
    (NULL, '555-5678', NULL),
    ('charlie@example.com', '555-9012', '789 Pine Rd'),
    (NULL, NULL, NULL);
```
```sql
-- Step 2: Measure completeness per column
SELECT
    COUNT(*) AS total_rows,
    COUNT(email) AS non_null_email,
    ROUND(100.0 * COUNT(email) / COUNT(*), 1) AS email_completeness,
    COUNT(phone) AS non_null_phone,
    ROUND(100.0 * COUNT(phone) / COUNT(*), 1) AS phone_completeness,
    COUNT(address) AS non_null_address,
    ROUND(100.0 * COUNT(address) / COUNT(*), 1) AS address_completeness
FROM customers;
```

**Expected Output**:
 total_rows | non_null_email | email_completeness | non_null_phone | phone_completeness | non_null_address | address_completeness
------------|----------------|--------------------|----------------|--------------------|------------------|----------------------
5           |              3 |               60.0 |              3 |               60.0 |                3 |                 60.0

```sql
-- Step 3: Identify rows with any missing required field
SELECT customer_id, email, phone, address
FROM customers
WHERE email IS NULL OR phone IS NULL OR address IS NULL;
```

**Expected Output**:
 customer_id | email             | phone    | address
-------------|-------------------|----------|-------------
 2 | bob@example.com   |          | 456 Oak Ave
 3 |                   | 555-5678 |
 5 |                   |          |

**Why This Output Occurs**: Step 2 computes the completeness percentage for each column. Step 3 identifies the specific rows with missing values, enabling targeted remediation. In this example, all three columns have 60% completeness (3 of 5 rows).

### Real-World Cases

**Case 1: ETL Load Verification**: After a nightly ETL load, a completeness check verifies that the target table has the same row count as the source. A 99.5% completeness threshold triggers an alert.

**Case 2: Customer Onboarding**: A completeness rule requires email and phone for every customer. The test identifies 40 customers missing phone numbers, triggering a data collection campaign.

**Case 3: Healthcare Records**: A completeness check verifies that every patient record has a date of birth and insurance ID. Missing values are flagged for manual review.

---

## Core Concept 2: Accuracy

### Definitions

**Core Definition**: Accuracy testing ensures that computed columns, summaries, and mathematical formulas align precisely with raw inputs.

**Technical Definition**: Accuracy testing verifies derived data (denormalized columns, materialized views, aggregate tables) against the raw data they are derived from. It checks that `order_total` equals the sum of `order_items`, that `customer_lifetime_value` equals the sum of all orders, and that percentages sum to 100. Discrepancies indicate stale caches, faulty triggers, or ETL bugs.

**Beginner-Friendly Explanation**: Accuracy testing is like verifying that a receipt total matches the sum of the line items. If the receipt says $50 but the items add up to $45, the receipt is inaccurate.

### Purposes

- **To** verify that denormalized columns match their source data
- **To** detect stale caches and materialized views
- **To** validate mathematical formulas (totals, averages, percentages)
- **To** catch ETL bugs that produce incorrect derived data

### Syntax Rules and Structure

#### Denormalized Column Accuracy

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

#### Aggregate Table Accuracy

```sql
SELECT
    c.customer_id,
    c.total_spent AS cached_total,
    COALESCE(SUM(o.total), 0) AS actual_total,
    c.total_spent - COALESCE(SUM(o.total), 0) AS difference
FROM customer_summary c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.total_spent
HAVING c.total_spent <> COALESCE(SUM(o.total), 0);
```

#### Percentage Accuracy (Sum to 100)

```sql
SELECT
    category,
    SUM(percentage) AS total_percentage
FROM category_distribution
GROUP BY category
HAVING ABS(SUM(percentage) - 100.0) > 0.01;
```

#### Component Breakdown

| Accuracy Check | Formula | Tolerance |
|----------------|---------|-----------|
| Denormalized total | `stored_total = SUM(line_items)` | Exact or ±0.01 |
| Aggregate sum | `cached_sum = SUM(source)` | Exact |
| Percentage sum | `SUM(percentages) = 100` | ±0.01 |
| Average | `stored_avg = SUM/COUNT` | Exact or ±0.001 |

#### Syntax Rules

- Use `HAVING` to filter only rows with discrepancies (not all rows).
- Floating-point comparisons require tolerance (`ABS(a - b) > 0.01`).
- Reconcile derived data against the source of truth, not another derived source.
- Run accuracy tests after every ETL load or cache refresh.

#### Constraints and Limitations

- Floating-point arithmetic accumulates rounding errors; use `NUMERIC` for money.
- Accuracy tests can be slow on large tables; sample if necessary.
- Some discrepancies are acceptable (e.g., eventual consistency windows).

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Denormalized Order Total Accuracy

```sql
-- Step 1: Create tables with denormalized total
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    order_total NUMERIC(10,2)
);
CREATE TABLE order_items (
    item_id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(order_id),
    quantity INT,
    unit_price NUMERIC(10,2)
);

-- Step 2: Insert data with an inaccurate total
INSERT INTO orders (order_id, order_total) VALUES (1, 100.00);
INSERT INTO order_items (order_id, quantity, unit_price) VALUES
    (1, 2, 25.00),   -- 50.00
    (1, 3, 20.00);   -- 60.00
-- Actual total: 110.00, stored: 100.00

-- Step 3: Detect accuracy discrepancies
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

**Case 1: Financial Reconciliation**: A bank's `account_balance` column is denormalized from transaction records. Daily accuracy tests verify that the stored balance equals the sum of all transactions.

**Case 2: Inventory Accuracy**: A warehouse's `stock_quantity` is denormalized from `stock_movements`. Accuracy tests detect discrepancies caused by failed triggers.

**Case 3: Sales Dashboard**: A daily sales summary table is validated against the raw `orders` table. Discrepancies indicate a failed ETL job.

---

## Core Concept 3: Consistency

### Definitions

**Core Definition**: Consistency testing checks that mirrored or denormalized values match across different database nodes, replicas, or data warehouses.

**Technical Definition**: Consistency testing verifies that the same logical data has the same value across multiple systems: primary vs. replica, OLTP vs. OLAP, application cache vs. database, or across sharded databases. It detects replication lag, ETL bugs, and synchronization failures. Metrics include row-count consistency, checksum consistency, and value-level consistency.

**Beginner-Friendly Explanation**: Consistency testing is like checking that the same book has the same content in two different libraries. If the first library says chapter 5 has 20 pages and the second says 25 pages, the books are inconsistent.

### Purposes

- **To** verify that replication has propagated all changes
- **To** detect ETL bugs that produce different values in the warehouse
- **To** validate that caches match the source of truth
- **To** ensure cross-shard data consistency in distributed databases

### Syntax Rules and Structure

#### Primary vs. Replica Consistency

```sql
-- On primary
SELECT COUNT(*) AS primary_count, SUM(total) AS primary_sum FROM orders;

-- On replica
SELECT COUNT(*) AS replica_count, SUM(total) AS replica_sum FROM orders;

-- Compare: counts and sums should match
```

#### Cross-Database Consistency (dblink)

```sql
-- PostgreSQL: compare with remote database
SELECT
    local.order_id,
    local.total AS local_total,
    remote.total AS remote_total
FROM orders local
JOIN dblink('dbname=warehouse', 'SELECT order_id, total FROM orders') 
    AS remote(order_id INT, total NUMERIC) 
    ON local.order_id = remote.order_id
WHERE local.total <> remote.total;
```

#### Checksum Consistency

```sql
-- Compare checksums across systems
SELECT
    'primary' AS source,
    MD5(STRING_AGG(order_id::text || total::text, ',' ORDER BY order_id)) AS checksum
FROM orders
UNION ALL
SELECT
    'replica' AS source,
    MD5(STRING_AGG(order_id::text || total::text, ',' ORDER BY order_id)) AS checksum
FROM orders;
```

#### Component Breakdown

| Consistency Type | Comparison | Tolerance |
|------------------|------------|-----------|
| Row count | `COUNT(*)` primary vs. replica | 0 (eventual) |
| Aggregate | `SUM(col)` primary vs. replica | 0 (eventual) |
| Checksum | `MD5(STRING_AGG(...))` | Exact |
| Value-level | Row-by-row comparison | Exact |

#### Syntax Rules

- Consistency checks must account for replication lag; run after replication catches up.
- Checksums are fast for detecting differences but require re-computation on both sides.
- Value-level comparisons require a common key (e.g., `order_id`).
- Cross-database checks require `dblink`, foreign data wrappers, or ETL tools.

#### Constraints and Limitations

- Replication lag means consistency is eventual, not immediate.
- Checksum computation can be expensive on large tables.
- Schema differences between systems complicate value-level comparison.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Primary vs. Replica Consistency Check

```sql
-- Step 1: Create a function to compare primary and replica
CREATE OR REPLACE FUNCTION check_replica_consistency()
RETURNS TABLE (
    metric TEXT,
    primary_value BIGINT,
    replica_value BIGINT,
    difference BIGINT
) AS $$
BEGIN
    RETURN QUERY
    SELECT 'row_count', 
           (SELECT COUNT(*) FROM orders),
           (SELECT COUNT(*) FROM dblink('dbname=replica', 'SELECT COUNT(*) FROM orders') 
            AS r(count BIGINT)),
           (SELECT COUNT(*) FROM orders) - 
           (SELECT COUNT(*) FROM dblink('dbname=replica', 'SELECT COUNT(*) FROM orders') 
            AS r(count BIGINT))
    UNION ALL
    SELECT 'sum_total',
           (SELECT SUM(total)::BIGINT FROM orders),
           (SELECT SUM(total)::BIGINT FROM dblink('dbname=replica', 'SELECT SUM(total) FROM orders') 
            AS r(sum BIGINT)),
           (SELECT SUM(total)::BIGINT FROM orders) - 
           (SELECT SUM(total)::BIGINT FROM dblink('dbname=replica', 'SELECT SUM(total) FROM orders') 
            AS r(sum BIGINT));
END;
$$ LANGUAGE plpgsql;

-- Step 2: Run the consistency check
SELECT * FROM check_replica_consistency();
```

**Expected Output**:
```
   metric   | primary_value | replica_value | difference
------------+---------------+---------------+------------
 row_count  |       1000000 |       1000000 |          0
 sum_total  |    5000000000 |    5000000000 |          0
```

**Why This Output Occurs**: The function compares row counts and sum totals between primary and replica. A difference of 0 indicates full consistency. A non-zero difference indicates replication lag or a replication failure.

### Real-World Cases

**Case 1: Read Replica Lag**: A consistency check runs every 5 minutes. When replication lag exceeds 1,000 rows, an alert fires, and the application routes reads to the primary.

**Case 2: Data Warehouse Sync**: A nightly ETL loads data into a warehouse. A consistency check compares row counts and checksums between OLTP and OLAP. Discrepancies trigger a re-run.

**Case 3: Multi-Region Replication**: A global application replicates data across three regions. Consistency checks verify that all regions converge to the same state after replication completes.

---

## Core Concept 4: Uniqueness

### Definitions

**Core Definition**: Uniqueness testing ensures that primary keys and unique indexes prevent duplicate identity states, and that business-level uniqueness rules are enforced.

**Technical Definition**: Uniqueness testing validates that primary keys have no duplicates (enforced by the database), unique constraints have no duplicates, and business keys (e.g., email, SSN, product SKU) are unique. It also detects near-duplicates (case-insensitive, whitespace-trimmed) that violate business rules but pass database uniqueness checks.

**Beginner-Friendly Explanation**: Uniqueness testing is like checking that no two students have the same student ID. The database enforces this with a primary key, but business rules may also require unique email addresses—even if the email is capitalized differently.

### Purposes

- **To** verify that primary keys have no duplicates
- **To** detect business-key duplicates (email, SSN, SKU)
- **To** find near-duplicates (case, whitespace variations)
- **To** validate that unique constraints are enforced

### Syntax Rules and Structure

#### Primary Key Uniqueness

```sql
SELECT
    customer_id,
    COUNT(*) AS duplicate_count
FROM customers
GROUP BY customer_id
HAVING COUNT(*) > 1;
-- Expected: 0 rows (primary key prevents duplicates)
```

#### Business Key Uniqueness

```sql
SELECT
    LOWER(TRIM(email)) AS normalized_email,
    COUNT(*) AS duplicate_count,
    ARRAY_AGG(customer_id) AS customer_ids
FROM customers
GROUP BY LOWER(TRIM(email))
HAVING COUNT(*) > 1;
```

#### Near-Duplicate Detection

```sql
SELECT
    customer_id,
    customer_name,
    SOUNDEX(customer_name) AS soundex_code,
    COUNT(*) OVER (PARTITION BY SOUNDEX(customer_name)) AS similar_count
FROM customers
WHERE SOUNDEX(customer_name) IN (
    SELECT SOUNDEX(customer_name)
    FROM customers
    GROUP BY SOUNDEX(customer_name)
    HAVING COUNT(*) > 1
);
```

#### Component Breakdown

| Uniqueness Type | Detection | Enforcement |
|-----------------|-----------|-------------|
| Primary key | `GROUP BY pk HAVING COUNT(*) > 1` | Database constraint |
| Unique constraint | `GROUP BY col HAVING COUNT(*) > 1` | Database constraint |
| Business key | `GROUP BY LOWER(TRIM(col)) HAVING COUNT(*) > 1` | Application or partial index |
| Near-duplicate | `SOUNDEX`, `LEVENSHTEIN` | Manual review |

#### Syntax Rules

- Primary key uniqueness is guaranteed by the database; the test verifies the constraint exists.
- Business-key uniqueness requires normalization (case, whitespace) before comparison.
- `UNIQUE` constraints allow multiple NULLs in most engines.
- Near-duplicate detection uses fuzzy matching (`SOUNDEX`, `LEVENSHTEIN`, `DIFFERENCE`).

#### Constraints and Limitations

- Fuzzy matching produces false positives; manual review is required.
- Case-insensitive uniqueness requires a functional unique index (`LOWER(email)`).
- Multi-column uniqueness requires a composite unique constraint.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Detecting Business-Key Duplicates

```sql
-- Step 1: Create table with case-insensitive duplicate emails
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    email VARCHAR(255),
    customer_name VARCHAR(100)
);

INSERT INTO customers (email, customer_name) VALUES
    ('alice@example.com', 'Alice'),
    ('ALICE@example.com', 'Alice Smith'),  -- Duplicate (case-insensitive)
    ('bob@example.com', 'Bob'),
    (' alice@example.com ', 'Alice Jones');  -- Duplicate (whitespace + case)

-- Step 2: Detect case-insensitive, trimmed duplicates
SELECT
    LOWER(TRIM(email)) AS normalized_email,
    COUNT(*) AS duplicate_count,
    ARRAY_AGG(customer_id) AS customer_ids,
    ARRAY_AGG(email) AS email_variants
FROM customers
GROUP BY LOWER(TRIM(email))
HAVING COUNT(*) > 1;
```

**Expected Output**:
```
 normalized_email   | duplicate_count | customer_ids | email_variants
--------------------+-----------------+--------------+------------------------------------------
 alice@example.com  |               3 | {1,2,4}      | {alice@example.com,ALICE@example.com," alice@example.com "}
```

```sql
-- Step 3: Add a functional unique index to prevent future duplicates
CREATE UNIQUE INDEX uq_customers_email_normalized 
ON customers (LOWER(TRIM(email)));

-- Step 4: Verify the constraint works
INSERT INTO customers (email, customer_name) VALUES ('Alice@Example.com', 'Another Alice');
-- Expected: ERROR: duplicate key value violates unique constraint
```

**Why This Output Occurs**: Step 2 normalizes emails with `LOWER(TRIM(email))` and groups by the normalized value. Three rows share the same normalized email, revealing business-key duplicates. Step 3 adds a functional unique index that prevents future case/whitespace duplicates.

### Real-World Cases

**Case 1: User Registration**: A registration system checks for case-insensitive email duplicates. A functional unique index on `LOWER(email)` prevents duplicate accounts.

**Case 2: Product SKU Uniqueness**: A product catalog enforces unique SKUs. A uniqueness test detects duplicates caused by a failed import.

**Case 3: Patient Records**: A healthcare system checks for duplicate patients by name and date of birth. Fuzzy matching identifies potential duplicates for manual review.

---

## Core Concept 5: Validity

### Definitions

**Core Definition**: Validity testing matches column values against expected business formatting rules, such as phone syntax, email patterns, or postal code configurations.

**Technical Definition**: Validity testing uses regular expressions, `CHECK` constraints, and domain rules to verify that values conform to expected formats. Examples include email RFC 5322 patterns, phone number formats (E.164, NANP), postal codes (US ZIP, UK postcode, Canadian postal code), ISO country codes, currency codes, and date formats. Invalid values indicate data entry errors or application bugs.

**Beginner-Friendly Explanation**: Validity testing is like checking that a phone number looks like a phone number. "555-1234" is valid; "abc-defg" is not. The test verifies that every value matches the expected pattern.

### Purposes

- **To** detect malformed emails, phone numbers, and postal codes
- **To** verify that codes (country, currency) match standards
- **To** enforce format rules at the database level with `CHECK` constraints
- **To** validate data before loading into analytics systems

### Syntax Rules and Structure

#### Email Validity (PostgreSQL)

```sql
SELECT customer_id, email
FROM customers
WHERE email !~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$';
```

#### Phone Validity (E.164 Format)

```sql
SELECT customer_id, phone
FROM customers
WHERE phone !~ '^\+[1-9]\d{1,14}$';
```

#### Postal Code Validity (US ZIP)

```sql
SELECT customer_id, postal_code
FROM customers
WHERE postal_code !~ '^\d{5}(-\d{4})?$';
```

#### CHECK Constraint for Validity

```sql
ALTER TABLE customers
ADD CONSTRAINT chk_email_format
CHECK (email ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');

ALTER TABLE customers
ADD CONSTRAINT chk_phone_e164
CHECK (phone IS NULL OR phone ~ '^\+[1-9]\d{1,14}$');
```

#### Component Breakdown

| Field | Pattern | Example |
|-------|---------|---------|
| Email | `^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$` | `user@example.com` |
| Phone (E.164) | `^\+[1-9]\d{1,14}$` | `+14155552671` |
| US ZIP | `^\d{5}(-\d{4})?$` | `94105` or `94105-1234` |
| UK Postcode | `^[A-Z]{1,2}\d[A-Z\d]? ?\d[A-Z]{2}$` | `SW1A 1AA` |
| ISO Country | `^[A-Z]{2}$` | `US`, `GB` |
| ISO Currency | `^[A-Z]{3}$` | `USD`, `EUR` |

#### Syntax Rules

- PostgreSQL uses `~` (matches) and `!~` (does not match) for regex.
- MySQL uses `REGEXP` and `NOT REGEXP`.
- SQL Server uses `LIKE` (limited) or CLR regex functions.
- `CHECK` constraints validate on INSERT and UPDATE.
- Regex patterns should be tested against known valid and invalid samples.

#### Constraints and Limitations

- Regex cannot validate every edge case (e.g., email deliverability).
- Complex regex patterns can be slow on large tables.
- `CHECK` constraints add write overhead.
- Some databases (MySQL) have limited regex support in `CHECK` constraints.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Detecting Invalid Emails and Phones

```sql
-- Step 1: Create table with invalid values
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    email VARCHAR(255),
    phone VARCHAR(20),
    postal_code VARCHAR(20)
);

INSERT INTO customers (email, phone, postal_code) VALUES
    ('alice@example.com', '+14155552671', '94105'),
    ('bob@example', '+1-415-555-2671', '94105-1234'),      -- Invalid email, non-E.164 phone
    ('charlie@example.com', '555-1234', 'ABCDE'),           -- Non-E.164 phone, invalid ZIP
    ('invalid-email', '+441234567890', 'SW1A 1AA');         -- Invalid email

-- Step 2: Find invalid emails
SELECT customer_id, email
FROM customers
WHERE email !~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$';
```

**Expected Output**:
```
 customer_id | email
-------------+---------------
           2 | bob@example
           4 | invalid-email
```

```sql
-- Step 3: Find invalid phones (non-E.164)
SELECT customer_id, phone
FROM customers
WHERE phone !~ '^\+[1-9]\d{1,14}$';
```

**Expected Output**:
```
 customer_id | phone
-------------+-----------------
           2 | +1-415-555-2671
           3 | 555-1234
```

```sql
-- Step 4: Find invalid postal codes (non-US ZIP)
SELECT customer_id, postal_code
FROM customers
WHERE postal_code !~ '^\d{5}(-\d{4})?$';
```

**Expected Output**:
```
 customer_id | postal_code
-------------+-------------
           3 | ABCDE
           4 | SW1A 1AA
```

**Why This Output Occurs**: Each regex pattern defines the expected format. `!~` returns rows that do not match, identifying invalid values. The tests reveal specific data quality issues (malformed emails, non-E.164 phones, non-US postal codes).

### Real-World Cases

**Case 1: Email Validation**: A marketing system validates email formats before sending campaigns. Invalid emails are flagged for correction.

**Case 2: Phone Number Normalization**: A CRM system requires E.164 phone numbers. Validity tests identify non-conforming numbers for normalization.

**Case 3: Postal Code Validation**: A shipping system validates postal codes against country-specific patterns. Invalid codes cause delivery failures.

---

## Core Concept 6: Referential Integrity

### Definitions

**Core Definition**: Referential integrity testing verifies that all active child items connect to verifiable parent identifiers, with no orphaned or invalid references.

**Technical Definition**: Referential integrity testing validates that every non-NULL foreign-key value in a child table matches a primary-key value in the parent table. It detects orphans (child rows with no parent), invalid references (foreign keys pointing to non-existent parents), and soft-delete inconsistencies (child rows referencing soft-deleted parents). Tests use anti-joins (`LEFT JOIN ... WHERE parent.id IS NULL` or `NOT EXISTS`).

**Beginner-Friendly Explanation**: Referential integrity testing is like checking that every employee's ID badge references a valid department. If an employee's badge says "Department 999" but no such department exists, the reference is broken.

### Purposes

- **To** detect orphaned child rows with no matching parent
- **To** verify that foreign-key constraints are enforced
- **To** check for soft-delete inconsistencies (active children of deleted parents)
- **To** validate cross-database references in distributed systems

### Syntax Rules and Structure

#### Orphan Detection

```sql
SELECT
    o.order_id,
    o.customer_id
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL
  AND o.customer_id IS NOT NULL;
```

#### NOT EXISTS Alternative

```sql
SELECT o.order_id, o.customer_id
FROM orders o
WHERE NOT EXISTS (
    SELECT 1 FROM customers c WHERE c.customer_id = o.customer_id
)
AND o.customer_id IS NOT NULL;
```

#### Soft-Delete Consistency

```sql
SELECT
    o.order_id,
    o.customer_id,
    c.deleted_at
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE c.deleted_at IS NOT NULL
  AND o.status = 'ACTIVE';
```

#### Component Breakdown

| Check | Query Pattern | Detects |
|-------|---------------|---------|
| Orphans | `LEFT JOIN ... WHERE parent.id IS NULL` | Missing parents |
| Invalid FK | `NOT EXISTS` | Non-existent parents |
| Soft-delete | `JOIN ... WHERE parent.deleted_at IS NOT NULL` | Active children of deleted parents |
| Null FK | `WHERE fk IS NULL` | Missing references (may be valid) |

#### Syntax Rules

- Foreign-key constraints prevent orphans at the database level; tests verify the constraints exist and are not bypassed.
- Soft-delete patterns require application-level referential integrity checks.
- Cross-database references cannot use foreign keys; tests use `dblink` or ETL validation.
- `NOT NULL` foreign keys require every child to have a parent; nullable foreign keys allow orphans.

#### Constraints and Limitations

- Foreign-key constraints add write overhead; some systems disable them during bulk loads.
- Soft-delete patterns complicate referential integrity; child rows may reference deleted parents intentionally.
- Cross-database references cannot be enforced by the database.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Detecting Orphans and Soft-Delete Inconsistencies

```sql
-- Step 1: Create tables with soft delete
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100),
    deleted_at TIMESTAMP
);
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    status VARCHAR(20) DEFAULT 'ACTIVE'
);

-- Step 2: Insert data with orphans and soft-delete issues
INSERT INTO customers (customer_id, customer_name, deleted_at) VALUES
    (1, 'Alice', NULL),
    (2, 'Bob', '2025-01-01'),  -- Soft-deleted
    (3, 'Charlie', NULL);

INSERT INTO orders (customer_id, status) VALUES
    (1, 'ACTIVE'),
    (2, 'ACTIVE'),      -- Active order for soft-deleted customer
    (999, 'ACTIVE');    -- Orphan (no customer 999)

-- Step 3: Detect orphans
SELECT o.order_id, o.customer_id
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL AND o.customer_id IS NOT NULL;
```

**Expected Output**:
```
 order_id | customer_id
----------+-------------
        3 |         999
```

```sql
-- Step 4: Detect active orders for soft-deleted customers
SELECT o.order_id, o.customer_id, c.deleted_at
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE c.deleted_at IS NOT NULL AND o.status = 'ACTIVE';
```

**Expected Output**:
```
 order_id | customer_id |      deleted_at
----------+-------------+---------------------
        2 |           2 | 2025-01-01 00:00:00
```

**Why This Output Occurs**: Step 3's anti-join reveals order 3 with customer_id 999 (no matching customer). Step 4 joins orders to customers and filters for soft-deleted customers with active orders, revealing order 2 for the soft-deleted customer Bob.

### Real-World Cases

**Case 1: E-Commerce Order Integrity**: A referential integrity test verifies that every order references an existing customer. Orphans indicate a failed cascade or a data import bug.

**Case 2: Soft-Delete Cleanup**: A SaaS application soft-deletes customers but leaves their orders active. The test identifies active orders for deleted customers, triggering a cleanup or reassignment.

**Case 3: Cross-Database References**: A microservices architecture stores orders in one database and customers in another. Referential integrity tests use API calls or ETL validation to detect broken references.

---

## Core Concept 7: Automated Data Quality Frameworks

### Definitions

**Core Definition**: Automated data quality frameworks are tools that allow data quality rules to be defined as code, run automatically on every data load, and integrated into CI/CD pipelines.

**Technical Definition**: Modern data quality frameworks include **dbt tests** (schema tests and data tests defined in YAML and SQL), **Great Expectations** (Python-based expectation suites with automated validation), and **Soda** (SQL-based checks with a YAML configuration). These frameworks run assertions against data, produce pass/fail results, and integrate with orchestration tools (Airflow, Dagster) and CI/CD pipelines. Tests are version-controlled alongside data transformations.

**Beginner-Friendly Explanation**: Automated frameworks are like having a robot quality inspector on the production line. Instead of manually checking data quality, you write rules once, and the robot checks every batch automatically, alerting you when something is wrong.

### Purposes

- **To** codify data quality rules as version-controlled tests
- **To** run quality checks automatically on every data load
- **To** integrate data quality into CI/CD pipelines
- **To** produce machine-readable results for alerting and dashboards

### Syntax Rules and Structure

#### dbt Schema Tests (YAML)

```yaml
# models/schema.yml
version: 2

models:
  - name: orders
    columns:
      - name: order_id
        tests:
          - unique
          - not_null
      - name: customer_id
        tests:
          - not_null
          - relationships:
              to: ref('customers')
              field: customer_id
      - name: status
        tests:
          - accepted_values:
              values: ['PENDING', 'SHIPPED', 'DELIVERED', 'CANCELLED']
```

#### dbt Data Tests (SQL)

```sql
-- tests/assert_order_total_matches_items.sql
SELECT
    o.order_id,
    o.order_total,
    SUM(oi.quantity * oi.unit_price) AS computed_total
FROM {{ ref('orders') }} o
JOIN {{ ref('order_items') }} oi ON o.order_id = oi.order_id
GROUP BY o.order_id, o.order_total
HAVING o.order_total <> SUM(oi.quantity * oi.unit_price)
```

#### Great Expectations (Python)

```python
import great_expectations as gx

context = gx.get_context()
validator = context.sources.pandas_default.read_csv("orders.csv")

# Expectation 1: order_id is unique
validator.expect_column_values_to_be_unique("order_id")

# Expectation 2: email matches regex
validator.expect_column_values_to_match_regex(
    "email",
    r"^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$"
)

# Expectation 3: status is in allowed set
validator.expect_column_values_to_be_in_set(
    "status",
    ["PENDING", "SHIPPED", "DELIVERED", "CANCELLED"]
)

# Expectation 4: customer_id has no nulls
validator.expect_column_values_to_not_be_null("customer_id")

# Run validation
results = validator.validate()
print(results)
```

#### Soda (YAML)

```yaml
# soda/checks.yml
checks for orders:
  - row_count > 0
  - missing_count(customer_id) = 0
  - duplicate_count(order_id) = 0
  - invalid_count(email) = 0:
      valid format: email
  - values in (status) must be in ['PENDING', 'SHIPPED', 'DELIVERED', 'CANCELLED']
```

#### Component Breakdown

| Framework | Language | Test Definition | Integration |
|-----------|----------|-----------------|-------------|
| dbt | YAML + SQL | `schema.yml`, `tests/*.sql` | dbt Cloud, Airflow, CI |
| Great Expectations | Python | Expectation suites | Airflow, Prefect, CI |
| Soda | YAML + SQL | `checks.yml` | Airflow, Dagster, CI |
| Custom SQL | SQL | SQL scripts | Any scheduler |

#### Syntax Rules

- Tests are version-controlled alongside transformation code.
- Tests run after data loads, before downstream consumption.
- Failures should block downstream processing (fail-fast) or alert (monitor).
- Tests should be granular: one assertion per rule.
- Use `severity` levels (warn vs. error) for non-critical rules.

#### Constraints and Limitations

- Framework setup and maintenance require engineering effort.
- Tests add execution time to data pipelines.
- False positives require tuning (e.g., regex patterns).
- Not all frameworks support all databases.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: dbt Tests for an Orders Model

```yaml
# models/schema.yml
version: 2

models:
  - name: orders
    description: "Order fact table"
    columns:
      - name: order_id
        description: "Primary key"
        tests:
          - unique
          - not_null
      - name: customer_id
        description: "Foreign key to customers"
        tests:
          - not_null
          - relationships:
              to: ref('customers')
              field: customer_id
      - name: order_total
        description: "Total order amount"
        tests:
          - not_null
          - dbt_utils.expression_is_true:
              expression: ">= 0"
      - name: status
        tests:
          - accepted_values:
              values: ['PENDING', 'SHIPPED', 'DELIVERED', 'CANCELLED']
```

```bash
# Run dbt tests
dbt test --select orders
```

**Expected Output**:
```
Running with dbt=1.7.0
Found 1 model, 8 tests, 0 snapshots, 0 analyses, 0 macros, 0 operations, 0 seed files, 0 sources

19:45:00 | Concurrency: 8 threads (target='dev')
19:45:00 1 of 8 START test not_null_orders_order_id.................... [RUN]
19:45:01 1 of 8 PASS not_null_orders_order_id............................ [PASS in 0.5s]
19:45:01 2 of 8 START test unique_orders_order_id...................... [RUN]
19:45:01 2 of 8 PASS unique_orders_order_id.............................. [PASS in 0.4s]
19:45:01 3 of 8 START test not_null_orders_customer_id................. [RUN]
19:45:01 3 of 8 PASS not_null_orders_customer_id......................... [PASS in 0.4s]
19:45:01 4 of 8 START test relationships_orders_customer_id__customer_id__ref_customers_ [RUN]
19:45:01 4 of 8 PASS relationships_orders_customer_id__customer_id__ref_customers_ [PASS in 0.6s]
19:45:01 5 of 8 START test accepted_values_orders_status__PENDING__SHIPPED__DELIVERED__CANCELLED [RUN]
19:45:01 5 of 8 PASS accepted_values_orders_status__PENDING__SHIPPED__DELIVERED__CANCELLED [PASS in 0.5s]
19:45:01 6 of 8 START test not_null_orders_order_total................. [RUN]
19:45:01 6 of 8 PASS not_null_orders_order_total......................... [PASS in 0.4s]
19:45:01 7 of 8 START test expression_is_true_orders_order_total___________ [RUN]
19:45:01 7 of 8 PASS expression_is_true_orders_order_total___________ [PASS in 0.5s]
19:45:01 8 of 8 START test dbt_utils_expression_is_true_orders_........ [RUN]
19:45:01 8 of 8 PASS dbt_utils_expression_is_true_orders_............ [PASS in 0.6s]

Finished running 8 tests in 3.9s.
Completed successfully
Done. PASS=8 WARN=0 ERROR=0 SKIP=0 TOTAL=8
```

**Why This Output Occurs**: dbt compiles each test into a SQL query, runs it against the data warehouse, and reports pass/fail. All 8 tests pass, indicating the `orders` model meets all quality rules.

#### Example 2: Great Expectations Suite

```python
import great_expectations as gx
import pandas as pd

# Step 1: Load data
df = pd.read_csv("orders.csv")
validator = gx.from_pandas(df)

# Step 2: Define expectations
validator.expect_column_values_to_be_unique("order_id")
validator.expect_column_values_to_not_be_null("customer_id")
validator.expect_column_values_to_be_between("order_total", min_value=0)
validator.expect_column_values_to_be_in_set(
    "status", ["PENDING", "SHIPPED", "DELIVERED", "CANCELLED"]
)
validator.expect_column_values_to_match_regex(
    "email", r"^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$"
)

# Step 3: Run validation
results = validator.validate()

# Step 4: Check results
print(f"Success: {results.success}")
for result in results.results:
    print(f"  {result.expectation_config.expectation_type}: {'PASS' if result.success else 'FAIL'}")
```

**Expected Output**:
```
Success: True
  expect_column_values_to_be_unique: PASS
  expect_column_values_to_not_be_null: PASS
  expect_column_values_to_be_between: PASS
  expect_column_values_to_be_in_set: PASS
  expect_column_values_to_match_regex: PASS
```

**Why This Output Occurs**: Great Expectations runs each expectation against the DataFrame and reports pass/fail. All expectations pass, indicating the data meets quality rules.

### Real-World Cases

**Case 1: dbt in Production**: A data team uses dbt tests to validate 200 models on every run. Failures block downstream models and alert the on-call data engineer.

**Case 2: Great Expectations in ML Pipelines**: An ML team uses Great Expectations to validate training data before model training. Invalid data is quarantined.

**Case 3: Soda for Monitoring**: A data platform uses Soda checks to monitor data quality in production, with alerts sent to Slack when checks fail.

---

## Core Concept 8: CI/CD Schema Validation

### Definitions

**Core Definition**: CI/CD schema validation automates migration testing to verify that schema updates do not silently break existing software features.

**Technical Definition**: CI/CD schema validation runs database migrations against a test database in the CI pipeline, then executes application tests (unit, integration, contract) to verify that the schema change does not break existing queries, constraints, or application behavior. It catches breaking changes (dropped columns, renamed tables, changed data types, removed constraints) before production deployment. Tools include Flyway, Liquibase, Alembic, and framework-native migrations (Django, Rails, EF Core).

**Beginner-Friendly Explanation**: CI/CD schema validation is like test-driving a car after replacing a part. You replace the part (run the migration), then drive the car (run the tests) to make sure everything still works. If the car breaks, you don't ship it.

### Purposes

- **To** catch breaking schema changes before production
- **To** verify that migrations are reversible (rollback tested)
- **To** validate that application code works with the new schema
- **To** automate migration testing in CI/CD pipelines

### Syntax Rules and Structure

#### Migration Script (Flyway)

```sql
-- V1__create_users.sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT NOW()
);

-- V2__add_name_to_users.sql
ALTER TABLE users ADD COLUMN name VARCHAR(100);

-- V3__add_index_on_email.sql
CREATE INDEX idx_users_email ON users(email);
```

#### CI/CD Pipeline (GitHub Actions)

```yaml
name: Database Migration Test
on: [push, pull_request]

jobs:
  test-migrations:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
    steps:
      - uses: actions/checkout@v3
      
      - name: Run migrations
        run: |
          flyway -url=jdbc:postgresql://localhost:5432/test \
                 -user=postgres -password=test migrate
      
      - name: Run application tests
        run: |
          npm test
      
      - name: Test rollback
        run: |
          flyway -url=jdbc:postgresql://localhost:5432/test \
                 -user=postgres -password=test undo
      
      - name: Verify rollback succeeded
        run: |
          psql -h localhost -U postgres -d test -c "\dt"
```

#### Schema Drift Detection

```sql
-- Compare expected schema with actual schema
SELECT
    table_name,
    column_name,
    data_type,
    is_nullable,
    column_default
FROM information_schema.columns
WHERE table_schema = 'public'
ORDER BY table_name, ordinal_position;
-- Diff against expected schema file
```

#### Component Breakdown

| Step | Action | Purpose |
|------|--------|---------|
| 1 | Checkout code | Get migration and application code |
| 2 | Start test database | Isolated environment |
| 3 | Run migrations | Apply schema changes |
| 4 | Run application tests | Verify code works with new schema |
| 5 | Test rollback | Verify migrations are reversible |
| 6 | Compare schema | Detect drift from expected |

#### Syntax Rules

- Migrations must be idempotent and versioned.
- CI must use a fresh database for each test run.
- Rollback must be tested; not all migrations are reversible.
- Application tests must run after migrations to catch breaking changes.
- Schema drift detection compares actual schema to expected schema.

#### Constraints and Limitations

- Some migrations are irreversible (dropped columns, data loss).
- Rollback testing may not catch all issues (e.g., data-dependent bugs).
- CI databases may be smaller than production, hiding performance issues.
- Schema drift detection requires a source of truth for expected schema.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Testing a Breaking Migration

```sql
-- Step 1: Migration V1 creates the users table
-- V1__create_users.sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    name VARCHAR(100)
);

-- Step 2: Application code depends on the name column
-- SELECT user_id, email, name FROM users;

-- Step 3: Migration V2 accidentally drops the name column
-- V2__drop_name.sql
ALTER TABLE users DROP COLUMN name;

-- Step 4: Application tests run after migration and FAIL
-- ERROR: column "name" does not exist
```

**Expected CI Output**:
```
Running migration V2__drop_name.sql... SUCCESS
Running application tests...
FAIL: test_get_user_profile - column "name" does not exist
Tests failed: 1
CI pipeline FAILED
```

```sql
-- Step 5: Fix the migration to rename instead of drop
-- V2__rename_name_to_full_name.sql
ALTER TABLE users RENAME COLUMN name TO full_name;

-- Step 6: Update application code to use full_name
-- SELECT user_id, email, full_name FROM users;

-- Step 7: Re-run CI — tests pass
```

**Expected CI Output**:
```
Running migration V2__rename_name_to_full_name.sql... SUCCESS
Running application tests...
All tests passed: 15
CI pipeline PASSED
```

**Why This Output Occurs**: The first migration drops the `name` column, breaking application code that references it. The CI pipeline catches the failure before production. The fix renames the column and updates the application code, so both the migration and tests pass.

#### Example 2: Schema Drift Detection

```sql
-- Step 1: Expected schema (from version control)
-- expected_schema.sql
-- users: user_id (int, PK), email (varchar(255), NOT NULL, UNIQUE), created_at (timestamp)

-- Step 2: Detect drift in CI
SELECT
    column_name,
    data_type,
    is_nullable,
    column_default
FROM information_schema.columns
WHERE table_schema = 'public' AND table_name = 'users'
ORDER BY ordinal_position;
```

**Expected Output (Actual Schema)**:
```
 column_name | data_type                   | is_nullable | column_default
-------------+-----------------------------+-------------+----------------
 user_id     | integer                     | NO          | nextval('users_user_id_seq')
 email       | character varying           | NO          | NULL
 created_at  | timestamp without time zone | YES         | now()
```

```bash
# Step 3: Compare with expected schema
diff expected_schema.sql actual_schema.sql
```

**Expected Diff**:
```
< email       | character varying(255)      | NO          | NULL
---
> email       | character varying           | NO          | NULL
```

**Why This Output Occurs**: The actual schema has `character varying` without the length limit, while the expected schema specifies `character varying(255)`. The drift detection reveals the discrepancy, which could affect application validation.

### Real-World Cases

**Case 1: Column Drop Regression**: A migration drops a column still used by the application. CI catches the failure, and the migration is fixed to rename instead.

**Case 2: Data Type Change**: A migration changes `INT` to `BIGINT` on a foreign key. CI tests verify that all application queries still work.

**Case 3: Constraint Addition**: A migration adds a `NOT NULL` constraint to a column with existing NULLs. CI catches the failure before production.

---

## References

| Name | Link |
|------|------|
| dbt Documentation — Tests | https://docs.getdbt.com/docs/build/tests |
| Great Expectations Documentation | https://docs.greatexpectations.io/ |
| Soda Documentation | https://docs.soda.io/ |
| Flyway Documentation | https://documentation.red-gate.com/flyway |
| Liquibase Documentation | https://docs.liquibase.com/ |
| PostgreSQL Documentation — Constraints | https://www.postgresql.org/docs/current/ddl-constraints.html |
| PostgreSQL Documentation — Regular Expressions | https://www.postgresql.org/docs/current/functions-matching.html |
| MySQL 8.0 Reference Manual — CHECK Constraints | https://dev.mysql.com/doc/refman/8.0/en/create-table-check-constraints.html |
| Microsoft Learn — CHECK Constraints | https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-check-constraints |
| DAMA-DMBOK — Data Quality Dimensions | https://www.dama.org/ |
| ISO 8000 — Data Quality | https://www.iso.org/standard/50798.html |
| Google Cloud — Data Quality Best Practices | https://cloud.google.com/architecture/dq-managed |
| OWASP — Input Validation Cheat Sheet | https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html |
| RFC 5322 — Email Format | https://datatracker.ietf.org/doc/html/rfc5322 |
| ITU-T E.164 — Phone Number Format | https://www.itu.int/rec/T-REC-E.164/ |
| USPS — ZIP Code Format | https://www.usps.com/ |