# ETL-Oriented SQL: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** ETL-oriented SQL is the application of SQL language constructs, functions, and architectural patterns to perform Extract, Transform, and Load operations within a relational database system, moving data from source systems into target data warehouses or operational data stores.

**Technical Definition:** ETL-oriented SQL encompasses declarative and procedural techniques—including change data capture (CDC), Common Table Expressions (CTEs), UPSERT/MERGE statements, staging table design, watermark-based incremental loading, and slowly changing dimension (SCD) management—that extract data from source systems, transform it through multi-step SQL pipelines, and load it into target schemas while maintaining data quality, referential integrity, and historical traceability. These operations are typically orchestrated by task schedulers, stored procedures, or ETL frameworks.

**Beginner-Friendly Explanation:** ETL-oriented SQL is like a data assembly line. You take raw data from one place (Extract), clean it up and reshape it (Transform), and put it into a new home where it is ready for reporting and analysis (Load). SQL is the language you use to build and run that assembly line.

### Key Characteristics

- **Set-based:** Operates on entire tables or result sets, not individual rows in application loops.
- **Pipeline-oriented:** Transformations are chained through CTEs, views, or stored procedures.
- **Incremental by design:** Processes only new or changed data using watermark columns, CDC, or hash comparisons.
- **Idempotent where possible:** Re-running a load produces the same result (via UPSERT/MERGE).
- **Orchestration-dependent:** Requires a scheduler (cron, Airflow, SQL Agent, dbt) to execute steps in order.
- **Warehouse-oriented:** Optimized for read-heavy analytics workloads, not OLTP transactions.

### Prerequisites

- Proficiency with SQL `SELECT`, `INSERT`, `UPDATE`, `DELETE`, and `MERGE`.
- Understanding of CTEs and window functions.
- Familiarity with transaction isolation and locking behavior.
- Knowledge of the source system's change-tracking capabilities.
- Awareness of target schema design (star schema, snowflake, data vault).

### Related Programming Areas

- Data warehousing and business intelligence.
- Data engineering and pipeline development.
- Master data management and data quality.
- dbt (data build tool) and analytics engineering.
- Database administration and scheduling.

### Core Concepts / Features

1. **Extract** (CDC, transactional replicas, source schema scanning)
2. **Transform** (CTEs, multi-table conditional branching, analytical subqueries)
3. **Load** (UPSERT via `INSERT ... ON CONFLICT` or `MERGE`)
4. **Staging Tables** (untyped schemas, write-optimized heaps, transient tables)
5. **Transformation Pipelines** (sequential views, stored procedures, orchestration)
6. **Incremental Loading** (watermark keys, modified timestamps, identity columns)
7. **Change Detection** (CDC, hash-based variance checks, slowly changing dimensions)


## Core Concept 1: Extract

### Definitions

**Core Definition:** Extract is the first phase of ETL, where data is read from source systems—operational databases, APIs, flat files—and prepared for transformation and loading.

**Technical Definition:** Extract in ETL-oriented SQL uses change data capture (CDC) mechanisms—transaction log mining, change tracking tables, or timestamp-based queries—to read only new or modified rows from source systems. To prevent production lockups, extraction queries are typically directed against read replicas, read-scale availability groups, or log-shipped standby databases, isolating analytical read workloads from OLTP write workloads.

**Beginner-Friendly Explanation:** Extract is like making a photocopy of a document while people are still writing in it. You do not want to stop them from writing, so you use a special copy machine (read replica) that keeps up with the original without slowing it down.

### Purposes

- To read data from source systems without impacting production performance.
- To capture only changed rows using CDC, reducing extraction time and resource consumption.
- To maintain a consistent snapshot of source data for downstream transformation.
- To isolate extraction workloads from transactional workloads using read replicas.

### Syntax Rules and Structure

#### Complete General Syntax (SQL Server CDC Extraction)

```sql
-- Enable CDC on the database
EXEC sys.sp_cdc_enable_db;

-- Enable CDC on a specific table
EXEC sys.sp_cdc_enable_table
    @source_schema = 'dbo',
    @source_name = 'Orders',
    @role_name = NULL;

-- Query changed rows from the CDC change table
SELECT *
FROM cdc.dbo_Orders_CT
WHERE __$start_lsn > @last_extracted_lsn;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `sp_cdc_enable_db` | Enables CDC at the database level. |
| `sp_cdc_enable_table` | Enables CDC on a specific table. |
| `cdc.dbo_Orders_CT` | The change table that captures insert/update/delete operations. |
| `__$start_lsn` | Log sequence number marking the start of the change. |

#### Complete General Syntax (Watermark-Based Extraction)

```sql
SELECT *
FROM source_table
WHERE modified_at > (SELECT last_watermark FROM etl_watermarks WHERE table_name = 'source_table');
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `source_table` | The source table being extracted. |
| `modified_at` | A timestamp or identity column tracking row modifications. |
| `etl_watermarks` | A metadata table storing the high-water mark for each source. |
| `last_watermark` | The maximum `modified_at` value from the previous extraction. |

#### Syntax Rules

- CDC requires the SQL Server Agent to be running; change tables are populated asynchronously by capture jobs.
- CDC change tables retain data for a configurable retention period; ensure extraction runs more frequently than the retention window.
- Watermark-based extraction requires a monotonically increasing column (timestamp, identity, or version number) that is updated on every change.
- Read replicas must be configured with appropriate replication lag monitoring; extraction should not run if the replica is too far behind the primary.

#### Constraints and Limitations

- **CDC limitations:** SQL Server CDC does not capture schema changes (DDL); it captures DML only. It also requires additional storage for change tables.
- **Read replica lag:** Replicas may lag behind the primary; ensure extraction tolerates some staleness or waits for replica synchronization.
- **Watermark gaps:** If a row is modified but the watermark column is not updated, that change will be missed.
- **Version-specific:** PostgreSQL read replicas support logical replication only from version 16 onward; earlier versions require logical decoding on the primary.
- **Unsafe if misused:** Extracting from the primary during peak hours can cause lock contention and slow down production transactions.

### Annotated Code Examples

#### Example 1: SQL Server CDC Setup and Extraction

```sql
-- Step 1: Enable CDC at the database level
EXEC sys.sp_cdc_enable_db;
GO

-- Step 2: Enable CDC on the Orders table
EXEC sys.sp_cdc_enable_table
    @source_schema = 'dbo',
    @source_name = 'Orders',
    @role_name = NULL,
    @supports_net_changes = 1;
GO

-- Step 3: Insert a row to generate a change record
INSERT INTO dbo.Orders (CustomerID, OrderDate, Total)
VALUES (101, GETDATE(), 250.00);
GO

-- Step 4: Query the change table
SELECT
    __$start_lsn,
    __$operation,  -- 2 = INSERT, 3 = UPDATE before, 4 = UPDATE after, 1 = DELETE
    CustomerID,
    OrderDate,
    Total
FROM cdc.dbo_Orders_CT
WHERE __$operation IN (2, 4);
```

**Expected Output:**

```
__$start_lsn        | __$operation | CustomerID | OrderDate              | Total
--------------------+--------------+------------+------------------------+-------
0x0000002A0000018C  | 2            | 101        | 2024-06-15 10:30:00    | 250.00
```

**Why This Works:** `sp_cdc_enable_db` activates CDC for the database, and `sp_cdc_enable_table` creates a change table (`cdc.dbo_Orders_CT`) that captures all inserts, updates, and deletes. `__$operation = 2` indicates an INSERT. The `__$start_lsn` column provides the log sequence number for ordering and watermarking.

#### Example 2: PostgreSQL Watermark-Based Extraction from a Read Replica

```sql
-- On the read replica, extract only rows modified since the last watermark
WITH last_watermark AS (
    SELECT COALESCE(MAX(modified_at), '1900-01-01') AS watermark
    FROM etl_watermarks
    WHERE table_name = 'customers'
)
SELECT
    c.id,
    c.name,
    c.email,
    c.modified_at
FROM customers c, last_watermark lw
WHERE c.modified_at > lw.watermark
ORDER BY c.modified_at;
```

**Expected Output:**

```
id  | name  | email              | modified_at
----+-------+--------------------+---------------------
205 | Alice | alice@example.com  | 2024-06-15 08:00:00
206 | Bob   | bob@example.com    | 2024-06-15 09:30:00
207 | Carol | carol@example.com  | 2024-06-15 11:15:00
```

**Why This Works:** The CTE `last_watermark` retrieves the maximum `modified_at` from the previous extraction. The main query selects rows with `modified_at` greater than that watermark. Running this against a read replica isolates the extraction workload from the primary database.

#### Example 3: MySQL Replica-Based Extraction with `SELECT ... INTO OUTFILE`

```sql
-- On the read replica, export changed rows to a staging file
SELECT id, name, email, updated_at
INTO OUTFILE '/tmp/customers_delta.csv'
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
FROM customers
WHERE updated_at > '2024-06-14 00:00:00';
```

**Expected Output (file `/tmp/customers_delta.csv`):**

```
205,Alice,alice@example.com,2024-06-15 08:00:00
206,Bob,bob@example.com,2024-06-15 09:30:00
207,Carol,carol@example.com,2024-06-15 11:15:00
```

**Why This Works:** `SELECT ... INTO OUTFILE` writes the delta rows directly to a CSV file on the replica's filesystem. This avoids transferring data through the application layer and keeps the extraction workload on the replica. The `updated_at` column acts as the watermark filter.

### Real-World Cases

- **Retail analytics:** Extracting daily sales transactions from an OLTP database replica into a data warehouse.
- **Financial reporting:** Using CDC to capture account balance changes in near-real-time for regulatory reporting.
- **SaaS integration:** Polling a read replica for changed customer records to sync with a CRM system.
- **IoT data pipelines:** Extracting sensor readings from a time-series database using timestamp-based watermarks.

### References

- Microsoft SQL Server: About Change Data Capture — https://learn.microsoft.com/en-us/sql/relational-databases/track-changes/about-change-data-capture-sql-server
- Microsoft SQL Server: About Change Tracking — https://learn.microsoft.com/en-us/sql/relational-databases/track-changes/about-change-tracking-sql-server
- PostgreSQL: Logical Replication — https://www.postgresql.org/docs/current/logical-replication.html
- PostgreSQL: High Availability, Load Balancing, and Replication — https://www.postgresql.org/docs/current/high-availability.html
- MySQL: Replication — https://dev.mysql.com/doc/refman/8.0/en/replication.html


## Core Concept 2: Transform

### Definitions

**Core Definition:** Transform is the second phase of ETL, where extracted data is cleaned, enriched, aggregated, and restructured to conform to the target schema's business rules and analytical requirements.

**Technical Definition:** Transform in ETL-oriented SQL uses Common Table Expressions (CTEs) to decompose complex transformations into sequential, readable steps; multi-table conditional branching via `CASE` expressions and `JOIN` operations to apply different logic based on data conditions; and analytical subqueries with window functions (`ROW_NUMBER()`, `RANK()`, `LAG()`, `LEAD()`) to perform ranking, deduplication, and time-series comparisons within a single SQL statement.

**Beginner-Friendly Explanation:** Transform is like a recipe with multiple steps. You take raw ingredients (extracted data), chop them up (clean), mix them together (join), add seasoning (enrich), and cook them into the final dish (transformed result). CTEs let you write each step as a separate, readable block.

### Purposes

- To clean and standardize data before loading it into the target schema.
- To decompose complex multi-step transformations into readable, maintainable CTE chains.
- To apply conditional business logic that routes or classifies rows based on data values.
- To perform analytical calculations (rankings, running totals, comparisons) within the transformation pipeline.

### Syntax Rules and Structure

#### Complete General Syntax (CTE Chain)

```sql
WITH cte_name_1 AS (
    SELECT ... FROM source_table WHERE ...
),
cte_name_2 AS (
    SELECT ... FROM cte_name_1 JOIN other_table ON ...
),
cte_name_3 AS (
    SELECT ..., CASE WHEN ... THEN ... END AS derived_column
    FROM cte_name_2
)
SELECT * FROM cte_name_3;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `WITH` | Introduces one or more CTEs. |
| `cte_name AS (query)` | A named temporary result set. |
| `,` | Separates multiple CTEs; later CTEs can reference earlier ones. |
| `SELECT * FROM cte_name_3` | Final query consuming the last CTE. |

#### Complete General Syntax (Analytical Subquery with Window Function)

```sql
SELECT
    column1,
    column2,
    ROW_NUMBER() OVER (PARTITION BY group_column ORDER BY sort_column DESC) AS rn,
    LAG(column1) OVER (PARTITION BY group_column ORDER BY sort_column) AS previous_value
FROM source_table;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ROW_NUMBER() OVER (...)` | Assigns a unique sequential number within each partition. |
| `PARTITION BY` | Divides rows into groups for window calculations. |
| `ORDER BY` | Determines the order of rows within each partition. |
| `LAG(column)` | Accesses the value from the previous row in the partition. |

#### Syntax Rules

- CTEs are evaluated once and can be referenced multiple times in the main query.
- CTEs can reference other CTEs defined earlier in the same `WITH` clause.
- Recursive CTEs use `WITH RECURSIVE` and have a base case and a recursive case.
- Window functions cannot be used in `WHERE` or `HAVING` clauses directly; wrap them in a CTE or subquery.

#### Constraints and Limitations

- **CTE materialization:** Some databases materialize CTEs into temporary tables, which can be slower than inline subqueries for simple cases. PostgreSQL 12+ can inline CTEs when they are not recursive and are referenced only once.
- **Readability vs. performance:** Deep CTE chains can be harder for the query optimizer to optimize; test performance with `EXPLAIN`.
- **Window function performance:** Window functions require sorting; large partitions can consume significant memory.
- **Version-specific:** MySQL 8.0+ supports CTEs and window functions; earlier versions do not.

### Annotated Code Examples

#### Example 1: PostgreSQL CTE Chain for Sales Transformation

```sql
-- Create source tables
CREATE TABLE raw_sales (
    id SERIAL PRIMARY KEY,
    product_name TEXT,
    quantity INT,
    unit_price NUMERIC(10,2),
    sale_date DATE,
    region TEXT
);

INSERT INTO raw_sales (product_name, quantity, unit_price, sale_date, region) VALUES
('Widget', 10, 19.99, '2024-01-15', 'North'),
('Gadget', 5, 29.99, '2024-01-16', 'South'),
('Widget', 8, 19.99, '2024-02-10', 'North'),
('Gadget', 3, 29.99, '2024-02-15', 'North'),
('Doohickey', 20, 9.99, '2024-03-01', 'South');

-- Multi-step transformation pipeline
WITH monthly_sales AS (
    -- Step 1: Calculate revenue per product per month
    SELECT
        product_name,
        DATE_TRUNC('month', sale_date) AS sale_month,
        SUM(quantity * unit_price) AS revenue,
        SUM(quantity) AS total_quantity
    FROM raw_sales
    GROUP BY product_name, DATE_TRUNC('month', sale_date)
),
ranked_products AS (
    -- Step 2: Rank products by revenue within each month
    SELECT
        product_name,
        sale_month,
        revenue,
        total_quantity,
        RANK() OVER (
            PARTITION BY sale_month
            ORDER BY revenue DESC
        ) AS revenue_rank
    FROM monthly_sales
),
flagged_products AS (
    -- Step 3: Flag top performers
    SELECT
        product_name,
        sale_month,
        revenue,
        total_quantity,
        revenue_rank,
        CASE
            WHEN revenue_rank = 1 THEN 'Top Seller'
            WHEN revenue_rank <= 3 THEN 'Strong Performer'
            ELSE 'Standard'
        END AS performance_category
    FROM ranked_products
)
-- Final output
SELECT * FROM flagged_products
ORDER BY sale_month, revenue_rank;
```

**Expected Output:**

```
product_name | sale_month | revenue | total_quantity | revenue_rank | performance_category
-------------+------------+---------+----------------+--------------+---------------------
Widget       | 2024-01-01 |  199.90 |             10 |            1 | Top Seller
Gadget       | 2024-01-01 |  149.95 |              5 |            2 | Strong Performer
Widget       | 2024-02-01 |  159.92 |              8 |            1 | Top Seller
Gadget       | 2024-02-01 |   89.97 |              3 |            2 | Strong Performer
Doohickey    | 2024-03-01 |  199.80 |             20 |            1 | Top Seller
```

**Why This Works:** The first CTE aggregates raw sales into monthly revenue per product. The second CTE ranks products by revenue within each month using `RANK()`. The third CTE applies conditional branching with `CASE` to classify performance. The final query consumes the last CTE. Each step is independently readable and testable.

#### Example 2: Multi-Table Conditional Branching

```sql
-- Transform: classify customers based on order history
WITH customer_orders AS (
    SELECT
        c.id,
        c.name,
        c.email,
        COUNT(o.id) AS order_count,
        COALESCE(SUM(o.total), 0) AS lifetime_value,
        MAX(o.order_date) AS last_order_date
    FROM customers c
    LEFT JOIN orders o ON c.id = o.customer_id
    GROUP BY c.id, c.name, c.email
),
classified_customers AS (
    SELECT
        id,
        name,
        email,
        order_count,
        lifetime_value,
        last_order_date,
        CASE
            WHEN order_count = 0 THEN 'Prospect'
            WHEN lifetime_value > 1000 AND last_order_date > CURRENT_DATE - INTERVAL '30 days' THEN 'VIP Active'
            WHEN lifetime_value > 1000 THEN 'VIP Dormant'
            WHEN last_order_date > CURRENT_DATE - INTERVAL '90 days' THEN 'Active'
            ELSE 'At Risk'
        END AS customer_segment
    FROM customer_orders
)
SELECT * FROM classified_customers
ORDER BY lifetime_value DESC;
```

**Expected Output:**

```
id | name  | email             | order_count | lifetime_value | last_order_date | customer_segment
---+-------+-------------------+-------------+----------------+-----------------+------------------
1  | Alice | alice@example.com |           5 |        2500.00 | 2024-06-01      | VIP Active
2  | Bob   | bob@example.com   |           3 |        1200.00 | 2024-03-15      | VIP Dormant
3  | Carol | carol@example.com |           2 |         450.00 | 2024-05-20      | Active
4  | Dave  | dave@example.com  |           0 |           0.00 | NULL            | Prospect
```

**Why This Works:** The first CTE aggregates order data per customer. The second CTE applies a multi-condition `CASE` expression that classifies customers into segments based on order count, lifetime value, and recency. This pattern is common in customer analytics and segmentation pipelines.

### Real-World Cases

- **Sales analytics:** Aggregating daily sales into monthly summaries with ranking and year-over-year comparisons.
- **Customer segmentation:** Classifying customers into RFM (Recency, Frequency, Monetary) segments.
- **Financial reporting:** Computing running balances, period-over-period changes, and variance analysis.
- **Data quality pipelines:** Flagging rows that fail validation rules and routing them to error tables.

### References

- PostgreSQL Documentation: WITH Queries (Common Table Expressions) — https://www.postgresql.org/docs/current/queries-with.html
- PostgreSQL Documentation: Window Functions — https://www.postgresql.org/docs/current/functions-window.html
- MySQL Reference Manual: Window Functions — https://dev.mysql.com/doc/refman/8.0/en/window-functions.html
- SQL Server: WITH common_table_expression (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql


## Core Concept 3: Load

### Definitions

**Core Definition:** Load is the third phase of ETL, where transformed data is written into the target database using UPSERT operations that insert new rows or update existing ones atomically.

**Technical Definition:** Load in ETL-oriented SQL uses `INSERT ... ON CONFLICT` (PostgreSQL, SQLite) or `MERGE` (SQL Server, Oracle, PostgreSQL 15+, MySQL 8.0.19+) statements to perform atomic upserts—inserting rows that do not exist and updating rows that do, based on a match condition (typically a primary key or unique constraint). `MERGE` is a complex hybrid DML/DQL statement that supports conditional `INSERT`, `UPDATE`, and `DELETE` operations in a single statement.

**Beginner-Friendly Explanation:** Load is like filing documents into a filing cabinet. If a folder already exists, you update its contents; if it does not, you create a new folder. UPSERT (UPDATE + INSERT) does this in one step, so you do not have to check first.

### Purposes

- To atomically insert new rows or update existing rows in the target table.
- To synchronize a target table with a source table without deleting and reloading all data.
- To handle duplicate key conflicts gracefully during data loading.
- To support slowly changing dimension (SCD) updates in data warehouse loading.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL `INSERT ... ON CONFLICT`)

```sql
INSERT INTO target_table (column1, column2, column3)
VALUES (value1, value2, value3)
ON CONFLICT (conflict_column)
DO UPDATE SET
    column2 = EXCLUDED.column2,
    column3 = EXCLUDED.column3;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `INSERT INTO target_table` | The target table for insertion. |
| `VALUES (...)` | The row(s) to insert. |
| `ON CONFLICT (conflict_column)` | The unique constraint or primary key that triggers the conflict. |
| `DO UPDATE SET` | The update action when a conflict occurs. |
| `EXCLUDED.column` | Refers to the row that would have been inserted. |

#### Complete General Syntax (SQL Server `MERGE`)

```sql
MERGE INTO target_table AS target
USING source_table AS source
ON target.key = source.key
WHEN MATCHED THEN
    UPDATE SET target.column1 = source.column1
WHEN NOT MATCHED THEN
    INSERT (key, column1) VALUES (source.key, source.column1)
WHEN NOT MATCHED BY SOURCE THEN
    DELETE;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `MERGE INTO target` | The target table. |
| `USING source` | The source table or subquery. |
| `ON condition` | The match condition (typically a primary key). |
| `WHEN MATCHED THEN UPDATE` | Action when the key exists in both source and target. |
| `WHEN NOT MATCHED THEN INSERT` | Action when the key exists in source but not target. |
| `WHEN NOT MATCHED BY SOURCE THEN DELETE` | Action when the key exists in target but not source. |

#### Syntax Rules

- `ON CONFLICT` requires a unique index or primary key on the conflict column.
- `MERGE` requires a deterministic match condition; non-deterministic conditions can cause runtime errors.
- `MERGE` can have at most two `WHEN MATCHED` clauses; the first must include an `AND <condition>` clause.
- PostgreSQL 15+ supports `MERGE`, but with fewer features than SQL Server (no `WHEN NOT MATCHED BY SOURCE`).
- MySQL uses `INSERT ... ON DUPLICATE KEY UPDATE` instead of `ON CONFLICT`.

#### Constraints and Limitations

- **MERGE concurrency:** SQL Server's `MERGE` is not atomic at the engine level; concurrent transactions can cause duplicate-key errors. Use `HOLDLOCK` or a higher isolation level.
- **ON CONFLICT limitations:** PostgreSQL's `ON CONFLICT` does not support `WHEN NOT MATCHED BY SOURCE` (delete); it is insert/update only.
- **Performance:** `MERGE` performs a full table join; ensure the match condition is indexed on both source and target.
- **Version-specific:** `MERGE` was introduced in SQL:2003 and enhanced in SQL:2008. PostgreSQL 15+ supports `MERGE`; MySQL 8.0.19+ supports `MERGE` with limitations.
- **Unsafe if misused:** A `MERGE` with a non-deterministic `ON` condition can cause unpredictable results or runtime errors.

### Annotated Code Examples

#### Example 1: PostgreSQL `INSERT ... ON CONFLICT` for SCD Type 1

```sql
-- Create target table
CREATE TABLE dim_customers (
    customer_id INT PRIMARY KEY,
    name TEXT,
    email TEXT,
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Source data (staging table)
CREATE TABLE stg_customers (
    customer_id INT,
    name TEXT,
    email TEXT
);

INSERT INTO stg_customers VALUES
(1, 'Alice Smith', 'alice.smith@example.com'),
(2, 'Bob Jones', 'bob.jones@example.com'),
(3, 'Carol White', 'carol.white@example.com');

-- Upsert: insert new customers, update existing ones
INSERT INTO dim_customers (customer_id, name, email)
SELECT customer_id, name, email FROM stg_customers
ON CONFLICT (customer_id)
DO UPDATE SET
    name = EXCLUDED.name,
    email = EXCLUDED.email,
    updated_at = NOW();
```

**Expected Output:**

```
INSERT 0 3
```

**Why This Works:** `ON CONFLICT (customer_id)` detects rows where `customer_id` already exists in `dim_customers`. The `DO UPDATE SET` clause updates the `name`, `email`, and `updated_at` columns using the values from the `EXCLUDED` pseudo-table (the rows that would have been inserted). This is an SCD Type 1 pattern: changes overwrite previous values without history.

#### Example 2: SQL Server `MERGE` for SCD Type 2

```sql
-- Target dimension table with SCD Type 2 columns
CREATE TABLE dim_customers (
    customer_sk INT IDENTITY(1,1) PRIMARY KEY,
    customer_id INT,
    name NVARCHAR(100),
    email NVARCHAR(255),
    effective_date DATE,
    end_date DATE,
    is_current BIT
);

-- Staging table
CREATE TABLE stg_customers (
    customer_id INT,
    name NVARCHAR(100),
    email NVARCHAR(255)
);

-- Insert initial data
INSERT INTO dim_customers (customer_id, name, email, effective_date, end_date, is_current)
VALUES (1, 'Alice', 'alice@example.com', '2024-01-01', NULL, 1);

-- MERGE to handle SCD Type 2 changes
MERGE INTO dim_customers AS target
USING stg_customers AS source
ON target.customer_id = source.customer_id AND target.is_current = 1
WHEN MATCHED AND (target.name <> source.name OR target.email <> source.email) THEN
    UPDATE SET
        target.end_date = CAST(GETDATE() AS DATE),
        target.is_current = 0
WHEN NOT MATCHED THEN
    INSERT (customer_id, name, email, effective_date, end_date, is_current)
    VALUES (source.customer_id, source.name, source.email, CAST(GETDATE() AS DATE), NULL, 1);
```

**Expected Output:**

```
(1 row affected)
```

**Why This Works:** The `MERGE` matches current rows (`is_current = 1`) on `customer_id`. When a change is detected in `name` or `email`, the existing row is closed (`end_date` set, `is_current = 0`), and a new row is inserted with the updated values and a new `effective_date`. This is the standard SCD Type 2 pattern: every change creates a new version with validity dates.

#### Example 3: MySQL `INSERT ... ON DUPLICATE KEY UPDATE`

```sql
-- Create table with unique key
CREATE TABLE product_inventory (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    quantity INT,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Upsert: insert or update inventory
INSERT INTO product_inventory (product_id, product_name, quantity)
VALUES
    (101, 'Widget', 50),
    (102, 'Gadget', 30),
    (103, 'Doohickey', 75)
ON DUPLICATE KEY UPDATE
    product_name = VALUES(product_name),
    quantity = VALUES(quantity),
    last_updated = CURRENT_TIMESTAMP;
```

**Expected Output:**

```
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

**Why This Works:** MySQL's `ON DUPLICATE KEY UPDATE` detects conflicts on the primary key (`product_id`). The `VALUES(column)` function refers to the value that would have been inserted. This is MySQL's equivalent of `ON CONFLICT` and is the standard upsert pattern for MySQL.

### Real-World Cases

- **Data warehouse dimension loading:** Updating customer, product, and employee dimensions from source systems.
- **Inventory synchronization:** Merging real-time inventory updates from multiple warehouses into a central table.
- **User profile management:** Upserting user records during registration or profile update operations.
- **Price list updates:** Loading nightly price changes from a supplier feed into a product catalog.

### References

- PostgreSQL Documentation: INSERT — https://www.postgresql.org/docs/current/sql-insert.html
- PostgreSQL Documentation: MERGE — https://www.postgresql.org/docs/current/sql-merge.html
- SQL Server: MERGE (Transact-SQL) — https://learn.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql
- MySQL Reference Manual: INSERT ... ON DUPLICATE KEY UPDATE — https://dev.mysql.com/doc/refman/8.0/en/insert-on-duplicate.html
- Oracle Database SQL Language Reference: MERGE — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/MERGE.html


## Core Concept 4: Staging Tables

### Definitions

**Core Definition:** Staging tables are intermediate tables used to hold raw or minimally processed data between the extract and transform phases of ETL, providing a landing zone for source data before it is cleaned and loaded into target schemas.

**Technical Definition:** Staging tables are typically designed as write-optimized heap tables (no indexes, no constraints) with untyped or minimally typed columns (often all `VARCHAR` or `TEXT`) to accommodate source data without transformation. They are transient—truncated or dropped after each ETL cycle—and may use database-specific optimizations such as Oracle's `FOR STAGING` clause or Azure Synapse's round-robin distribution.

**Beginner-Friendly Explanation:** A staging table is like a loading dock at a warehouse. Trucks (source systems) dump their cargo (raw data) there first. Workers then sort, clean, and move the cargo to its final location (target tables). The loading dock is temporary—once the cargo is moved, the dock is empty and ready for the next shipment.

### Purposes

- To provide a landing zone for raw data before transformation, decoupling extraction from transformation.
- To minimize transformation errors by loading data in its native format before applying type conversions.
- To enable parallel loading of multiple source tables into independent staging tables.
- To serve as a restart point if a transformation fails midway, avoiding re-extraction from the source.

### Syntax Rules and Structure

#### Complete General Syntax (Oracle `FOR STAGING`)

```sql
CREATE TABLE staging_table (
    column1 VARCHAR2(4000),
    column2 VARCHAR2(4000),
    ...
)
FOR STAGING;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `FOR STAGING` | Oracle 23ai/26ai clause that creates a heap table optimized for fast data ingestion. |
| Disables compression | Compression is automatically disabled for staging tables. |
| No indexes | Staging tables are created without indexes to maximize insert throughput. |

#### Complete General Syntax (Generic Staging Table Design)

```sql
CREATE TABLE stg_source_name (
    load_id INT,
    row_number INT,
    column1 TEXT,
    column2 TEXT,
    ...
    loaded_at TIMESTAMP DEFAULT NOW()
);

-- Truncate before each load
TRUNCATE TABLE stg_source_name;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `load_id` | Identifies the ETL batch/run. |
| `row_number` | Row sequence within the load. |
| `column1, column2` | Source columns, often all `TEXT`. |
| `loaded_at` | Timestamp of when the row was staged. |

#### Syntax Rules

- Staging tables should have **no indexes** and **no constraints** (no primary key, no foreign key, no `CHECK`) to maximize insert speed.
- All columns should be `TEXT` or `VARCHAR` to accommodate source data without type conversion errors.
- Staging tables should be **truncated** at the start of each ETL run; do not rely on `DELETE` for cleanup.
- In SQL Server, staging tables can be created in `tempdb` as global temporary tables (`##staging`) for automatic cleanup.
- In Azure Synapse, use `ROUND_ROBIN` distribution for staging tables to maximize load parallelism.

#### Constraints and Limitations

- **No constraints:** Staging tables cannot enforce data quality; validation must happen in the transform phase.
- **Storage overhead:** Staging tables duplicate source data temporarily, requiring additional storage.
- **Transaction log growth:** Large staging loads generate significant transaction log volume; use `TRUNCATE` (minimally logged) instead of `DELETE`.
- **Oracle `FOR STAGING`:** Only available in Oracle 23ai/26ai; earlier versions use standard heap tables.
- **Version-specific:** Azure Synapse `CTAS` (Create Table As Select) is the recommended way to create staging tables for fast loads.

### Annotated Code Examples

#### Example 1: PostgreSQL Staging Table with Untyped Columns

```sql
-- Create a staging table with all TEXT columns
CREATE TABLE stg_raw_orders (
    load_id SERIAL,
    order_id TEXT,
    customer_id TEXT,
    order_date TEXT,
    amount TEXT,
    loaded_at TIMESTAMP DEFAULT NOW()
);

-- Load raw CSV data (all values as text)
COPY stg_raw_orders (order_id, customer_id, order_date, amount)
FROM '/data/raw_orders.csv'
WITH (FORMAT csv, HEADER true);

-- Verify staging data
SELECT * FROM stg_raw_orders LIMIT 5;
```

**Expected Output:**

```
load_id | order_id | customer_id | order_date   | amount  | loaded_at
--------+----------+-------------+--------------+---------+----------------------------
1       | ORD-001  | CUST-101    | 2024-01-15   | 150.00  | 2024-06-15 10:00:00
2       | ORD-002  | CUST-102    | 2024-01-16   | 200.50  | 2024-06-15 10:00:00
3       | ORD-003  | CUST-103    | 2024-01-17   | invalid | 2024-06-15 10:00:00
```

**Why This Works:** All columns are `TEXT`, so even the invalid `amount` value (`'invalid'`) loads successfully. This decouples extraction from transformation: the staging table accepts everything, and the transform phase applies type conversion with error handling (e.g., routing invalid rows to an error table).

#### Example 2: Oracle `FOR STAGING` Table

```sql
-- Create a staging table with the FOR STAGING clause
CREATE TABLE stg_employees (
    employee_id NUMBER,
    first_name VARCHAR2(100),
    last_name VARCHAR2(100),
    hire_date VARCHAR2(20),
    salary VARCHAR2(20)
)
FOR STAGING;

-- Insert raw data (all values as text)
INSERT INTO stg_employees VALUES
(1, 'Alice', 'Smith', '2024-01-15', '50000'),
(2, 'Bob', 'Jones', '01/20/2024', '60000'),
(3, 'Carol', 'White', 'invalid', 'not_a_number');

-- Verify
SELECT * FROM stg_employees;
```

**Expected Output:**

```
EMPLOYEE_ID | FIRST_NAME | LAST_NAME | HIRE_DATE    | SALARY
------------+------------+-----------+--------------+-----------
1           | Alice      | Smith     | 2024-01-15   | 50000
2           | Bob        | Jones     | 01/20/2024   | 60000
3           | Carol      | White     | invalid      | not_a_number
```

**Why This Works:** The `FOR STAGING` clause creates a heap table optimized for fast ingestion with compression disabled. All columns are text-compatible, so malformed values load without error. The transform phase will parse `hire_date` and `salary` with validation rules.

#### Example 3: SQL Server Global Temporary Staging Table

```sql
-- Create a global temporary staging table (automatically dropped when session ends)
CREATE TABLE ##stg_orders (
    order_id NVARCHAR(50),
    customer_id NVARCHAR(50),
    order_date NVARCHAR(50),
    amount NVARCHAR(50)
);

-- Bulk insert raw data
BULK INSERT ##stg_orders
FROM 'C:\data\raw_orders.csv'
WITH (FORMAT = 'CSV', FIRSTROW = 2, FIELDTERMINATOR = ',');

-- Verify
SELECT COUNT(*) AS staged_rows FROM ##stg_orders;
```

**Expected Output:**

```
staged_rows
-----------
10000
```

**Why This Works:** The `##` prefix creates a global temporary table in `tempdb`, which is automatically dropped when the last session using it closes. This avoids cluttering the user database with permanent staging tables. `BULK INSERT` loads the CSV quickly with minimal logging.

### Real-World Cases

- **Data warehouse ETL:** Landing raw extracts from multiple source systems before transformation.
- **Cloud data pipelines:** Staging data in Azure Synapse or Snowflake before loading into dimensional models.
- **Legacy migration:** Staging data from mainframe extracts before parsing fixed-width formats.
- **Real-time ingestion:** Staging streaming data from Kafka before batch transformation.

### References

- Oracle Database: Staging Tables in 23ai/26ai — https://oracle-base.com/articles/23/staging-tables-23
- Microsoft Azure Synapse: Designing Tables — https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-overview
- PostgreSQL: CREATE TABLE — https://www.postgresql.org/docs/current/sql-createtable.html
- SQL Server: Temporary Tables — https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql


## Core Concept 5: Transformation Pipelines

### Definitions

**Core Definition:** A transformation pipeline is an orchestrated sequence of SQL operations—views, stored procedures, or scheduled tasks—that moves data through multiple transformation stages from staging to final target.

**Technical Definition:** Transformation pipelines in ETL-oriented SQL use sequential views (layered views that build on each other), stored procedures (encapsulated multi-step logic with parameters and error handling), or task schedulers (SQL Server Agent, pg_cron, dbt) to orchestrate the execution of transformation steps. Each step consumes the output of the previous step, creating a directed acyclic graph (DAG) of data transformations.

**Beginner-Friendly Explanation:** A transformation pipeline is like an assembly line in a factory. Each station (view or stored procedure) does one specific job, and the product (data) moves from station to station until it is finished. A scheduler (the factory manager) makes sure each station runs in the right order.

### Purposes

- To decompose complex transformations into sequential, independently testable steps.
- To orchestrate the execution of dependent transformation steps in the correct order.
- To provide restart points if a pipeline step fails, avoiding reprocessing of already-completed steps.
- To encapsulate business logic in reusable views or stored procedures.

### Syntax Rules and Structure

#### Complete General Syntax (Layered Views)

```sql
-- Layer 1: Staging view
CREATE VIEW v_stg_orders AS
SELECT * FROM stg_raw_orders;

-- Layer 2: Cleansed view
CREATE VIEW v_clean_orders AS
SELECT
    order_id,
    customer_id,
    order_date::DATE,
    amount::NUMERIC
FROM v_stg_orders
WHERE amount ~ '^\d+(\.\d+)?$';

-- Layer 3: Enriched view
CREATE VIEW v_enriched_orders AS
SELECT
    o.order_id,
    c.customer_name,
    o.order_date,
    o.amount
FROM v_clean_orders o
JOIN dim_customers c ON o.customer_id = c.customer_id;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `v_stg_orders` | Raw staging view. |
| `v_clean_orders` | Cleansed view with type conversion and filtering. |
| `v_enriched_orders` | Enriched view with joins to dimension tables. |

#### Complete General Syntax (Stored Procedure Pipeline)

```sql
CREATE OR REPLACE PROCEDURE run_etl_pipeline()
LANGUAGE plpgsql
AS $$
BEGIN
    -- Step 1: Truncate staging
    TRUNCATE TABLE stg_raw_orders;

    -- Step 2: Extract
    INSERT INTO stg_raw_orders
    SELECT * FROM source_orders WHERE modified_at > (SELECT last_watermark FROM etl_watermarks);

    -- Step 3: Transform and load
    INSERT INTO fact_orders
    SELECT ... FROM stg_raw_orders
    ON CONFLICT (order_id) DO UPDATE SET ...;

    -- Step 4: Update watermark
    UPDATE etl_watermarks SET last_watermark = NOW() WHERE table_name = 'source_orders';
END;
$$;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `CREATE PROCEDURE` | Defines the pipeline as a stored procedure. |
| `TRUNCATE` | Clears staging tables. |
| `INSERT ... SELECT` | Extracts and loads data. |
| `ON CONFLICT` | Handles upserts. |
| `UPDATE etl_watermarks` | Records the high-water mark for the next run. |

#### Syntax Rules

- Layered views should be ordered from raw (staging) to refined (final output); each view should reference only earlier views.
- Stored procedures should be idempotent where possible; re-running should not cause duplicate data.
- Use transactions to ensure atomicity: if any step fails, the entire pipeline rolls back.
- Schedule pipelines using the database's native scheduler (SQL Server Agent, pg_cron) or an external orchestrator (Airflow, dbt).

#### Constraints and Limitations

- **View performance:** Layered views can degrade performance if each layer materializes intermediate results; consider materialized views for expensive transformations.
- **Stored procedure debugging:** Stored procedures are harder to debug than views; use logging tables to track pipeline progress.
- **Error handling:** Stored procedures should include `EXCEPTION` blocks to capture and log errors without aborting the entire pipeline.
- **Version-specific:** PostgreSQL 11+ supports `CREATE PROCEDURE` with transaction control; earlier versions use `CREATE FUNCTION`.

### Annotated Code Examples

#### Example 1: PostgreSQL Layered View Pipeline

```sql
-- Layer 1: Staging view (raw data as text)
CREATE VIEW v_stg_orders AS
SELECT
    order_id,
    customer_id,
    order_date,
    amount
FROM stg_raw_orders;

-- Layer 2: Cleansed view (type conversion and validation)
CREATE VIEW v_clean_orders AS
SELECT
    order_id,
    customer_id,
    order_date::DATE AS order_date,
    amount::NUMERIC(10,2) AS amount
FROM v_stg_orders
WHERE amount ~ '^\d+(\.\d+)?$'
  AND order_date ~ '^\d{4}-\d{2}-\d{2}$';

-- Layer 3: Enriched view (join with customer dimension)
CREATE VIEW v_enriched_orders AS
SELECT
    o.order_id,
    c.name AS customer_name,
    o.order_date,
    o.amount
FROM v_clean_orders o
JOIN dim_customers c ON o.customer_id = c.customer_id;

-- Query the final view
SELECT * FROM v_enriched_orders ORDER BY order_date;
```

**Expected Output:**

```
order_id | customer_name | order_date | amount
---------+---------------+------------+--------
ORD-001  | Alice         | 2024-01-15 | 150.00
ORD-002  | Bob           | 2024-01-16 | 200.50
```

**Why This Works:** Each view adds a layer of transformation. `v_stg_orders` exposes raw staging data. `v_clean_orders` applies type conversion and filters out malformed rows. `v_enriched_orders` joins with the customer dimension. The final query consumes the enriched view. This pipeline is declarative and easy to test at each layer.

#### Example 2: SQL Server Stored Procedure Pipeline

```sql
CREATE PROCEDURE dbo.RunETLPipeline
AS
BEGIN
    SET NOCOUNT ON;

    BEGIN TRY
        BEGIN TRANSACTION;

        -- Step 1: Truncate staging
        TRUNCATE TABLE dbo.stg_raw_orders;

        -- Step 2: Extract from source (read replica)
        INSERT INTO dbo.stg_raw_orders (order_id, customer_id, order_date, amount)
        SELECT order_id, customer_id, order_date, amount
        FROM source_orders
        WHERE modified_at > (SELECT last_watermark FROM dbo.etl_watermarks WHERE table_name = 'source_orders');

        -- Step 3: Transform and load into fact table
        MERGE dbo.fact_orders AS target
        USING (
            SELECT
                order_id,
                customer_id,
                TRY_CAST(order_date AS DATE) AS order_date,
                TRY_CAST(amount AS DECIMAL(10,2)) AS amount
            FROM dbo.stg_raw_orders
            WHERE TRY_CAST(order_date AS DATE) IS NOT NULL
              AND TRY_CAST(amount AS DECIMAL(10,2)) IS NOT NULL
        ) AS source
        ON target.order_id = source.order_id
        WHEN MATCHED THEN
            UPDATE SET target.amount = source.amount, target.order_date = source.order_date
        WHEN NOT MATCHED THEN
            INSERT (order_id, customer_id, order_date, amount)
            VALUES (source.order_id, source.customer_id, source.order_date, source.amount);

        -- Step 4: Update watermark
        UPDATE dbo.etl_watermarks
        SET last_watermark = GETDATE()
        WHERE table_name = 'source_orders';

        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        ROLLBACK TRANSACTION;
        INSERT INTO dbo.etl_error_log (error_message, error_time)
        VALUES (ERROR_MESSAGE(), GETDATE());
    END CATCH;
END;
```

**Expected Output (when executed):**

```
(5000 rows affected)
```

**Why This Works:** The stored procedure encapsulates the entire ETL pipeline in a single transactional unit. `TRY...CATCH` handles errors and rolls back on failure, logging the error to an error table. The `MERGE` statement handles upserts, and the watermark update ensures the next run starts from the correct position.

#### Example 3: Scheduled Pipeline with `pg_cron`

```sql
-- Install pg_cron extension
CREATE EXTENSION pg_cron;

-- Schedule the ETL pipeline to run every night at 2 AM
SELECT cron.schedule(
    'nightly-etl',
    '0 2 * * *',
    $$
    CALL run_etl_pipeline();
    $$
);
```

**Expected Output:**

```
 schedule
----------
       42
```

**Why This Works:** `pg_cron` is a PostgreSQL extension that schedules SQL statements using cron syntax. The `cron.schedule` function registers the `run_etl_pipeline` stored procedure to execute every night at 2 AM. The return value is the job ID.

### Real-World Cases

- **Data warehouse refresh:** Nightly ETL pipelines that transform raw source data into star-schema fact and dimension tables.
- **Real-time analytics:** Micro-batch pipelines that run every 5 minutes to load streaming data.
- **Financial close:** Monthly pipelines that aggregate and reconcile financial data from multiple systems.
- **Marketing attribution:** Pipelines that join clickstream data with campaign data to attribute conversions.

### References

- PostgreSQL Documentation: CREATE PROCEDURE — https://www.postgresql.org/docs/current/sql-createprocedure.html
- PostgreSQL Documentation: CREATE VIEW — https://www.postgresql.org/docs/current/sql-createview.html
- SQL Server: SQL Server Agent — https://learn.microsoft.com/en-us/sql/ssms/agent/sql-server-agent
- pg_cron GitHub Repository — https://github.com/citusdata/pg_cron
- dbt (data build tool) Documentation — https://docs.getdbt.com/


## Core Concept 6: Incremental Loading

### Definitions

**Core Definition:** Incremental loading is the practice of loading only new or changed data into the target table, rather than reloading the entire dataset on every ETL run.

**Technical Definition:** Incremental loading uses watermark keys—monotonically increasing columns such as timestamps, identity columns, or version numbers—to track the high-water mark of the last successful load. On each run, the extraction query filters source rows to those with watermark values greater than the stored high-water mark, and the load step upserts those rows into the target table.

**Beginner-Friendly Explanation:** Incremental loading is like reading only the new chapters of a book instead of re-reading the entire book every night. You bookmark where you left off (the watermark), and next time you start from that bookmark.

### Purposes

- To reduce ETL runtime by processing only new or changed data.
- To minimize the load on source systems by avoiding full-table scans.
- To reduce storage and network bandwidth by transferring only deltas.
- To enable more frequent ETL runs (hourly, every 15 minutes) without increasing resource consumption.

### Syntax Rules and Structure

#### Complete General Syntax (Watermark-Based Incremental Load)

```sql
-- Step 1: Read the current watermark
SELECT last_watermark
FROM etl_watermarks
WHERE table_name = 'source_table';

-- Step 2: Extract new rows
SELECT *
FROM source_table
WHERE modified_at > (SELECT last_watermark FROM etl_watermarks WHERE table_name = 'source_table');

-- Step 3: Load into target
INSERT INTO target_table (columns)
SELECT columns FROM source_table
WHERE modified_at > (SELECT last_watermark FROM etl_watermarks WHERE table_name = 'source_table')
ON CONFLICT (key) DO UPDATE SET ...;

-- Step 4: Update the watermark
UPDATE etl_watermarks
SET last_watermark = (SELECT MAX(modified_at) FROM source_table)
WHERE table_name = 'source_table';
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `etl_watermarks` | Metadata table storing the high-water mark for each source. |
| `last_watermark` | The maximum watermark value from the previous successful load. |
| `modified_at` | The watermark column in the source table. |
| `ON CONFLICT` | Handles upserts for rows that already exist. |

#### Syntax Rules

- The watermark column must be **monotonically increasing** (timestamp, identity, or version number).
- The watermark column must be **updated on every change**; if a row is modified but the watermark is not updated, that change will be missed.
- The watermark should be updated **after** the load commits successfully; updating before commit risks data loss if the transaction rolls back.
- For append-only tables, an identity column is the ideal watermark; for mutable tables, a `modified_at` timestamp is required.
- If multiple rows share the same watermark value (e.g., same timestamp), use `>=` with deduplication to avoid missing rows.

#### Constraints and Limitations

- **Watermark gaps:** Rows modified without updating the watermark column are missed; this is a common source of data quality issues.
- **Late-arriving data:** Data that arrives with a timestamp earlier than the current watermark may be missed; consider a lookback window (e.g., `modified_at > watermark - INTERVAL '1 hour'`).
- **Clock skew:** If the source system's clock is not synchronized, timestamp-based watermarks can miss or duplicate rows.
- **Identity column limitations:** Identity columns only track inserts, not updates or deletes; they are suitable only for append-only tables.
- **Version-specific:** SQL Server's `ROWVERSION` (timestamp) column is ideal for watermarking because it changes on every update.

### Annotated Code Examples

#### Example 1: PostgreSQL Timestamp-Based Incremental Load

```sql
-- Create watermark metadata table
CREATE TABLE etl_watermarks (
    table_name TEXT PRIMARY KEY,
    last_watermark TIMESTAMP
);

-- Initialize watermark
INSERT INTO etl_watermarks (table_name, last_watermark)
VALUES ('customers', '1900-01-01');

-- Create target table
CREATE TABLE dim_customers (
    customer_id INT PRIMARY KEY,
    name TEXT,
    email TEXT,
    updated_at TIMESTAMP
);

-- Incremental load
INSERT INTO dim_customers (customer_id, name, email, updated_at)
SELECT id, name, email, modified_at
FROM source_customers
WHERE modified_at > (SELECT last_watermark FROM etl_watermarks WHERE table_name = 'customers')
ON CONFLICT (customer_id)
DO UPDATE SET
    name = EXCLUDED.name,
    email = EXCLUDED.email,
    updated_at = EXCLUDED.updated_at;

-- Update watermark
UPDATE etl_watermarks
SET last_watermark = (SELECT MAX(modified_at) FROM source_customers)
WHERE table_name = 'customers';
```

**Expected Output:**

```
INSERT 0 150
UPDATE 1
```

**Why This Works:** The extraction query filters `source_customers` to rows with `modified_at` greater than the stored watermark. The `ON CONFLICT` clause upserts those rows into `dim_customers`. Finally, the watermark is updated to the maximum `modified_at` in the source. The next run will extract only rows modified after this point.

#### Example 2: SQL Server Identity Column Incremental Load

```sql
-- Create watermark table
CREATE TABLE etl_watermarks (
    table_name NVARCHAR(100) PRIMARY KEY,
    last_identity BIGINT
);

INSERT INTO etl_watermarks VALUES ('orders', 0);

-- Incremental load using identity column
INSERT INTO fact_orders (order_id, customer_id, order_date, amount)
SELECT
    order_id,
    customer_id,
    order_date,
    amount
FROM source_orders
WHERE order_id > (SELECT last_identity FROM etl_watermarks WHERE table_name = 'orders');

-- Update watermark
UPDATE etl_watermarks
SET last_identity = (SELECT MAX(order_id) FROM source_orders)
WHERE table_name = 'orders';
```

**Expected Output:**

```
(2500 rows affected)
(1 row affected)
```

**Why This Works:** The `order_id` identity column is monotonically increasing, so filtering by `order_id > last_identity` captures only new inserts. This is ideal for append-only tables like order logs. Note that updates to existing orders are not captured; use a `modified_at` timestamp for mutable tables.

#### Example 3: Lookback Window for Late-Arriving Data

```sql
-- Incremental load with a 1-hour lookback window
INSERT INTO fact_orders (order_id, customer_id, order_date, amount)
SELECT
    order_id,
    customer_id,
    order_date,
    amount
FROM source_orders
WHERE modified_at > (
    SELECT last_watermark - INTERVAL '1 hour'
    FROM etl_watermarks
    WHERE table_name = 'orders'
)
ON CONFLICT (order_id)
DO UPDATE SET
    amount = EXCLUDED.amount,
    order_date = EXCLUDED.order_date;

-- Update watermark to the maximum modified_at
UPDATE etl_watermarks
SET last_watermark = (SELECT MAX(modified_at) FROM source_orders)
WHERE table_name = 'orders';
```

**Expected Output:**

```
INSERT 0 50
UPDATE 1
```

**Why This Works:** Subtracting a 1-hour lookback window from the watermark captures late-arriving data that may have been committed after the last ETL run but with an earlier timestamp. The `ON CONFLICT` handles rows that were already loaded (upserting the latest values). This is a common pattern in event-driven systems where events can arrive out of order.

### Real-World Cases

- **Daily sales reports:** Loading only yesterday's transactions into a sales fact table.
- **Customer 360:** Updating customer profiles incrementally as changes occur in the CRM.
- **Inventory sync:** Loading only changed inventory levels from a warehouse management system.
- **Clickstream analytics:** Ingesting new page-view events every 15 minutes into an analytics table.

### References

- Microsoft Azure Data Factory: Incremental Copy — https://learn.microsoft.com/en-us/azure/data-factory/tutorial-incremental-copy-overview
- Databricks: Incremental Loading — https://docs.databricks.com/en/delta/incremental-loading.html
- Dremio: Incremental Processing — https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/incremental_processing.md


## Core Concept 7: Change Detection

### Definitions

**Core Definition:** Change detection is the process of identifying which rows in a source system have been inserted, updated, or deleted since the last ETL run, using mechanisms such as CDC, hash comparison, or timestamp tracking.

**Technical Definition:** Change detection in ETL-oriented SQL uses Change Data Capture (CDC) to read database transaction logs; hash functions (MD5, SHA-256) to compute checksums over business columns and compare hashes between source and target to detect changes; and slowly changing dimension (SCD) patterns to manage historical changes in dimension tables. SCD Type 1 overwrites changes in place; SCD Type 2 creates new rows with validity periods; SCD Type 3 adds columns for previous values.

**Beginner-Friendly Explanation:** Change detection is like comparing two versions of a document to see what changed. CDC reads the document's edit history (transaction log). Hash comparison creates a fingerprint of each version and compares fingerprints to spot differences quickly. SCD patterns are different strategies for keeping track of changes: overwrite (Type 1), keep all versions (Type 2), or keep just the previous version (Type 3).

### Purposes

- To identify which rows have changed without scanning the entire source table.
- To detect changes in source columns using hash comparison when CDC is not available.
- To maintain historical versions of dimension records using SCD Type 2.
- To reduce ETL runtime by processing only changed rows.

### Syntax Rules and Structure

#### Complete General Syntax (Hash-Based Change Detection)

```sql
-- Compute hash of source columns
WITH source_hashes AS (
    SELECT
        customer_id,
        MD5(
            COALESCE(name, '') || '|' ||
            COALESCE(email, '') || '|' ||
            COALESCE(phone, '')
        ) AS row_hash
    FROM source_customers
),
target_hashes AS (
    SELECT
        customer_id,
        MD5(
            COALESCE(name, '') || '|' ||
            COALESCE(email, '') || '|' ||
            COALESCE(phone, '')
        ) AS row_hash
    FROM dim_customers
    WHERE is_current = 1
)
SELECT
    s.customer_id,
    s.row_hash AS source_hash,
    t.row_hash AS target_hash
FROM source_hashes s
JOIN target_hashes t ON s.customer_id = t.customer_id
WHERE s.row_hash <> t.row_hash;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `MD5(...)` | Computes the MD5 hash of concatenated business columns. |
| `COALESCE(column, '')` | Replaces NULL with an empty string to ensure deterministic hashing. |
| `\|` | A separator character to prevent hash collisions from column boundary ambiguity. |
| `WHERE s.row_hash <> t.row_hash` | Identifies rows where the hash differs, indicating a change. |

#### Complete General Syntax (SCD Type 2 with MERGE)

```sql
MERGE INTO dim_customers AS target
USING (
    SELECT
        customer_id,
        name,
        email,
        MD5(COALESCE(name, '') || '|' || COALESCE(email, '')) AS row_hash
    FROM source_customers
) AS source
ON target.customer_id = source.customer_id
    AND target.is_current = 1
    AND target.row_hash <> source.row_hash
WHEN MATCHED THEN
    UPDATE SET
        target.end_date = CURRENT_DATE,
        target.is_current = 0
WHEN NOT MATCHED THEN
    INSERT (customer_id, name, email, effective_date, end_date, is_current, row_hash)
    VALUES (source.customer_id, source.name, source.email, CURRENT_DATE, NULL, 1, source.row_hash);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ON target.customer_id = source.customer_id AND target.is_current = 1` | Matches only the current version of each customer. |
| `AND target.row_hash <> source.row_hash` | Matches only when the hash differs (i.e., a change occurred). |
| `WHEN MATCHED THEN UPDATE` | Closes the current row by setting `end_date` and `is_current = 0`. |
| `WHEN NOT MATCHED THEN INSERT` | Inserts a new row with a new `effective_date` and `is_current = 1`. |

#### Syntax Rules

- Hash functions produce a fixed-length string (MD5 = 32 hex chars; SHA-256 = 64 hex chars); MD5 is faster but more collision-prone; SHA-256 is more secure but slower.
- Always use `COALESCE(column, '')` before hashing to ensure NULL values hash deterministically.
- Use a separator character (e.g., `|`) between columns to prevent collisions (e.g., `'ab' || 'c'` = `'a' || 'bc'`).
- SCD Type 2 requires at least four additional columns: `effective_date`, `end_date`, `is_current`, and a surrogate key.
- For large tables, compute hashes as a persisted column and index it to speed up comparison.

#### Constraints and Limitations

- **Hash collisions:** MD5 has known collision vulnerabilities; SHA-256 is recommended for critical data. Both can produce false positives (same hash, different data), though the probability is extremely low.
- **Column selection:** Hashing all columns is expensive; hash only business columns that are subject to change, excluding metadata columns like `updated_at`.
- **SCD Type 2 growth:** Dimension tables grow with every change; without pruning, they can become very large.
- **CDC limitations:** CDC captures DML but not DDL; schema changes are not tracked.
- **Version-specific:** SQL Server CDC requires SQL Server Agent; PostgreSQL logical replication requires `wal_level = logical`.

### Annotated Code Examples

#### Example 1: PostgreSQL Hash-Based Change Detection

```sql
-- Create source and target tables
CREATE TABLE source_customers (
    customer_id INT PRIMARY KEY,
    name TEXT,
    email TEXT,
    phone TEXT
);

CREATE TABLE dim_customers (
    customer_id INT PRIMARY KEY,
    name TEXT,
    email TEXT,
    phone TEXT,
    row_hash TEXT
);

-- Insert source data
INSERT INTO source_customers VALUES
(1, 'Alice', 'alice@example.com', '555-0101'),
(2, 'Bob', 'bob@example.com', '555-0102'),
(3, 'Carol', 'carol@example.com', '555-0103');

-- Insert target data (slightly different for customer 2)
INSERT INTO dim_customers (customer_id, name, email, phone, row_hash) VALUES
(1, 'Alice', 'alice@example.com', '555-0101', MD5('Alice|alice@example.com|555-0101')),
(2, 'Bob', 'bob.jones@example.com', '555-0102', MD5('Bob|bob.jones@example.com|555-0102')),
(3, 'Carol', 'carol@example.com', '555-0103', MD5('Carol|carol@example.com|555-0103'));

-- Detect changes by comparing hashes
WITH source_hashes AS (
    SELECT customer_id,
           MD5(COALESCE(name,'') || '|' || COALESCE(email,'') || '|' || COALESCE(phone,'')) AS row_hash
    FROM source_customers
)
SELECT
    s.customer_id,
    s.row_hash AS source_hash,
    t.row_hash AS target_hash,
    CASE WHEN s.row_hash <> t.row_hash THEN 'CHANGED' ELSE 'UNCHANGED' END AS status
FROM source_hashes s
JOIN dim_customers t ON s.customer_id = t.customer_id;
```

**Expected Output:**

```
customer_id | source_hash                       | target_hash                       | status
------------+-----------------------------------+-----------------------------------+-----------
1           | a1b2c3d4e5f6...                   | a1b2c3d4e5f6...                   | UNCHANGED
2           | f6e5d4c3b2a1...                   | 9f8e7d6c5b4a...                   | CHANGED
3           | 123456789abc...                   | 123456789abc...                   | UNCHANGED
```

**Why This Works:** Customer 2's email changed from `bob@example.com` to `bob.jones@example.com` in the source, producing a different MD5 hash. Customers 1 and 3 have identical hashes, indicating no change. This pattern is useful when CDC is not available or when the source is a file system rather than a database.

#### Example 2: SQL Server SCD Type 2 with Hash Comparison

```sql
-- Create dimension table with SCD Type 2 columns
CREATE TABLE dim_customers (
    customer_sk INT IDENTITY(1,1) PRIMARY KEY,
    customer_id INT,
    name NVARCHAR(100),
    email NVARCHAR(255),
    row_hash CHAR(32),
    effective_date DATE,
    end_date DATE,
    is_current BIT
);

-- Initial load
INSERT INTO dim_customers (customer_id, name, email, row_hash, effective_date, end_date, is_current)
SELECT customer_id, name, email,
       CONVERT(CHAR(32), HASHBYTES('MD5', CONCAT(ISNULL(name,''), '|', ISNULL(email,''))), 2),
       CAST(GETDATE() AS DATE), NULL, 1
FROM source_customers;

-- SCD Type 2 MERGE for subsequent changes
MERGE INTO dim_customers AS target
USING (
    SELECT customer_id, name, email,
           CONVERT(CHAR(32), HASHBYTES('MD5', CONCAT(ISNULL(name,''), '|', ISNULL(email,''))), 2) AS row_hash
    FROM source_customers
) AS source
ON target.customer_id = source.customer_id
   AND target.is_current = 1
WHEN MATCHED AND target.row_hash <> source.row_hash THEN
    UPDATE SET target.end_date = CAST(GETDATE() AS DATE), target.is_current = 0
WHEN NOT MATCHED THEN
    INSERT (customer_id, name, email, row_hash, effective_date, end_date, is_current)
    VALUES (source.customer_id, source.name, source.email, source.row_hash, CAST(GETDATE() AS DATE), NULL, 1);
```

**Expected Output:**

```
(1 row affected)
```

**Why This Works:** The `MERGE` matches current rows (`is_current = 1`) and compares hashes. When the hash differs, the current row is closed (`end_date` set, `is_current = 0`) and a new row is inserted with the updated values. This preserves the full history of changes in the dimension table.

#### Example 3: PostgreSQL Logical Replication CDC

```sql
-- On the primary database:
-- 1. Set wal_level = logical in postgresql.conf
-- 2. Create a publication
CREATE PUBLICATION etl_pub FOR TABLE customers, orders;

-- On the replica/ETL database:
-- Create a subscription
CREATE SUBSCRIPTION etl_sub
CONNECTION 'host=primary_host dbname=mydb user=replicator password=secret'
PUBLICATION etl_pub;

-- Query the replicated changes
SELECT * FROM customers WHERE modified_at > NOW() - INTERVAL '1 hour';
```

**Expected Output:**

```
 id  | name  | email              | modified_at
-----+-------+--------------------+---------------------
 205 | Alice | alice@example.com  | 2024-06-15 08:00:00
 206 | Bob   | bob@example.com    | 2024-06-15 09:30:00
```

**Why This Works:** Logical replication streams changes from the primary to the subscriber in real time. The ETL pipeline queries the subscriber (which acts as a read replica) without impacting the primary. This is the preferred CDC mechanism for PostgreSQL when supported.

### Real-World Cases

- **Data warehouse dimensions:** Maintaining SCD Type 2 history for customer, product, and employee dimensions.
- **Master data management:** Detecting and propagating changes across multiple systems using hash comparison.
- **Regulatory compliance:** Auditing all changes to financial records using CDC.
- **Real-time replication:** Streaming changes from an OLTP database to a data warehouse using logical replication.

### References

- Microsoft SQL Server: About Change Data Capture — https://learn.microsoft.com/en-us/sql/relational-databases/track-changes/about-change-data-capture-sql-server
- PostgreSQL: Logical Replication — https://www.postgresql.org/docs/current/logical-replication.html
- PostgreSQL: Publication and Subscription — https://www.postgresql.org/docs/current/sql-createpublication.html
- Oracle Database: Slowly Changing Dimensions — https://docs.oracle.com/en/database/oracle/oracle-database/21/dwhsg/implementing-slowly-changing-dimensions.html
- Kimball Group: Slowly Changing Dimensions — https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/type-2/


## Summary Table: ETL-Oriented SQL Patterns

| Concept | Primary SQL Constructs | Typical Use Case |
|---------|----------------------|------------------|
| Extract | CDC change tables, watermark queries, read replicas | Reading changed data without impacting production |
| Transform | CTEs, window functions, `CASE`, analytical subqueries | Cleaning, enriching, and reshaping data |
| Load | `INSERT ... ON CONFLICT`, `MERGE`, `ON DUPLICATE KEY UPDATE` | Upserting rows into target tables |
| Staging Tables | `FOR STAGING`, `##temp`, untyped heap tables | Landing raw data before transformation |
| Transformation Pipelines | Layered views, stored procedures, `pg_cron`, SQL Agent | Orchestrating multi-step transformations |
| Incremental Loading | Watermark tables, `modified_at` filters, identity columns | Loading only new or changed rows |
| Change Detection | CDC, `MD5`/`SHA-256` hashes, SCD Type 2 `MERGE` | Detecting and tracking data changes |


## Final Notes on Deprecated and Unsafe Features

- **SQL Server `MERGE` concurrency:** `MERGE` is not atomic at the engine level; concurrent transactions can cause duplicate-key errors. Use `HOLDLOCK` or a higher isolation level.
- **MD5 hash collisions:** MD5 has known collision vulnerabilities; use SHA-256 for critical data. Both can produce false positives, though the probability is extremely low.
- **CDC retention:** CDC change tables retain data for a configurable period; if extraction runs less frequently than the retention window, changes are lost.
- **Watermark gaps:** Rows modified without updating the watermark column are missed; this is a common source of data quality issues. Always verify that the watermark column is updated on every change.
- **Read replica lag:** Replicas may lag behind the primary; ensure extraction tolerates some staleness or waits for replica synchronization.
- **Version-specific:** PostgreSQL `MERGE` (15+), MySQL `MERGE` (8.0.19+), SQL Server CDC (2008+), PostgreSQL logical replication from read replicas (16+).
- **Unsafe if misused:** Extracting from the primary during peak hours can cause lock contention; always use a read replica or CDC for production ETL.