# SQL Table Partitioning: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**: Table partitioning is the database design technique of dividing a large logical table into smaller physical pieces called partitions, where each partition stores a subset of rows based on a defined partitioning key.

**Technical Definition**: Table partitioning is a data organization strategy where a table's rows are horizontally divided into multiple physical segments (partitions) according to a partitioning method—range, list, hash, or composite—with each partition stored in its own segment, allowing the query optimizer to perform partition pruning (eliminating irrelevant partitions from execution plans) and enabling partition-level maintenance operations such as switching, splitting, merging, and dropping.

**Beginner-Friendly Explanation**: Table partitioning is like organizing a huge filing cabinet into labeled drawers. Instead of one giant drawer stuffed with every document, you have separate drawers for "2024 Orders," "2025 Orders," and "2026 Orders." When you need 2025 data, you open only that drawer—you don't search the entire cabinet. This makes finding data faster and lets you archive or delete old drawers without disrupting the rest.

### Key Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Partitioning Key** | Column(s) or expression used to determine partition assignment |
| **Partition Pruning** | Optimizer eliminates irrelevant partitions from query plans |
| **Partition-Level Operations** | DDL operations (switch, split, merge, drop) target individual partitions |
| **Local vs. Global Indexes** | Indexes may be partitioned identically to the table (local) or independently (global) |
| **Maximum Partitions** | SQL Server: 15,000; PostgreSQL: no hard limit; MySQL: 8,192 per table |
| **Storage Placement** | Partitions may reside in different filegroups, tablespaces, or storage tiers |

### Prerequisites

- **Partitioning Key Selection**: A column or expression that supports efficient pruning (e.g., date, region, ID range)
- **Sufficient Data Volume**: Partitioning benefits typically appear when the table exceeds physical memory (PostgreSQL rule of thumb)
- **Partition Function/Scheme (SQL Server)** : Requires `CREATE PARTITION FUNCTION` and `CREATE PARTITION SCHEME`
- **Storage Planning**: Partitions may require separate filegroups or tablespaces
- **Application Compatibility**: Queries must include partition-key predicates to benefit from pruning

### Related Programming Areas

- **Database Administration (DBA)** : Partition maintenance, sliding window automation, index management
- **Performance Engineering**: Query plan analysis, partition pruning verification, index strategy
- **Data Warehousing**: ETL optimization, rolling window loads, data retention
- **Cloud Engineering**: Storage tiering (hot/warm/cold partitions), managed partition services
- **Application Development**: Partition-aware query design, partition key selection

### Core Concepts Overview

SQL table partitioning comprises seven complementary techniques:

1. **Range Partitioning**: Partitions defined by value ranges on a continuous key
2. **List Partitioning**: Partitions defined by discrete value lists
3. **Hash Partitioning**: Partitions defined by hash function of the key
4. **Composite Partitioning**: Multi-level partitioning (e.g., range-list, range-hash)
5. **Partition Pruning**: Optimizer elimination of irrelevant partitions
6. **Partition Maintenance Operations**: Automated creation, splitting, merging, and dropping
7. **Partitioning Constraints & Indexes**: Local vs. global indexes and unique key constraints

---

## Core Concept 1: Range Partitioning

### Definitions

**Core Definition**: Range partitioning divides a table into partitions based on a continuous range of values in the partition key.

**Technical Definition**: In range partitioning, each partition is assigned a range of values with non-overlapping bounds; the partition key values are continuous (e.g., dates, numeric identifiers), and each range's bounds are inclusive at the lower end and exclusive at the upper end.

**Beginner-Friendly Explanation**: Range partitioning is like organizing books by publication year. You have one shelf for books published 2000–2009, another for 2010–2019, and so on. If you're looking for a 2015 book, you go directly to that shelf.

### Purposes

- **To** organize time-series or sequential data for efficient historical queries
- **To** enable sliding window archiving where old ranges are dropped or archived
- **To** improve query performance through partition pruning on range predicates
- **To** distribute data across storage tiers based on age

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
CREATE TABLE table_name (
    column_definitions
) PARTITION BY RANGE (partition_key_column);

CREATE TABLE table_name_partition_name
    PARTITION OF table_name
    FOR VALUES FROM (lower_bound) TO (upper_bound);
```

#### Complete General Syntax (MySQL)

```sql
CREATE TABLE table_name (
    column_definitions
)
PARTITION BY RANGE (partition_function_expression) (
    PARTITION p0 VALUES LESS THAN (value0),
    PARTITION p1 VALUES LESS THAN (value1),
    ...
);
```

#### Complete General Syntax (SQL Server)

```sql
-- Step 1: Create partition function
CREATE PARTITION FUNCTION PF_name (data_type)
AS RANGE LEFT|RIGHT FOR VALUES (boundary1, boundary2, ...);

-- Step 2: Create partition scheme
CREATE PARTITION SCHEME PS_name
AS PARTITION PF_name TO (filegroup1, filegroup2, ...);

-- Step 3: Create table on partition scheme
CREATE TABLE table_name (
    column_definitions
) ON PS_name (partition_key_column);
```

#### Complete General Syntax (Oracle)

```sql
CREATE TABLE table_name (
    column_definitions
)
PARTITION BY RANGE (partition_key_column) (
    PARTITION p1 VALUES LESS THAN (value1),
    PARTITION p2 VALUES LESS THAN (value2),
    ...
);
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `PARTITION BY RANGE` | Specifies range partitioning method |
| `FOR VALUES FROM ... TO ...` | PostgreSQL: defines inclusive lower and exclusive upper bounds |
| `VALUES LESS THAN` | MySQL/Oracle: defines the upper bound (exclusive) |
| `AS RANGE LEFT/RIGHT` | SQL Server: LEFT means boundary belongs to left partition |
| `PARTITION FUNCTION` | SQL Server: defines boundary values |
| `PARTITION SCHEME` | SQL Server: maps partitions to filegroups |

#### Syntax Rules

- PostgreSQL: Bounds are `[lower, upper)` — lower inclusive, upper exclusive
- MySQL: `VALUES LESS THAN` is exclusive; use `MAXVALUE` for the highest partition
- SQL Server: `RANGE LEFT` places boundary in left partition; `RANGE RIGHT` in right
- Oracle: `VALUES LESS THAN` is exclusive; `MAXVALUE` for the highest partition

#### Constraints and Limitations

- Partition key must support comparison operators (`<`, `>`, `=`)
- Overlapping ranges are not allowed
- PostgreSQL: A default partition may be created with `DEFAULT` for rows outside defined ranges
- SQL Server: Maximum 15,000 partitions per table
- MySQL: Partitioning key must be part of every unique key in the table

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL Range Partitioning by Date

**Setup**: PostgreSQL 14+ with a database.

```sql
-- Step 1: Create the partitioned table
CREATE TABLE orders (
    order_id SERIAL,
    customer_id INT,
    order_date DATE NOT NULL,
    total NUMERIC(10,2)
) PARTITION BY RANGE (order_date);

-- Step 2: Create partitions for each quarter
CREATE TABLE orders_2025_q1 PARTITION OF orders
    FOR VALUES FROM ('2025-01-01') TO ('2025-04-01');

CREATE TABLE orders_2025_q2 PARTITION OF orders
    FOR VALUES FROM ('2025-04-01') TO ('2025-07-01');

CREATE TABLE orders_2025_q3 PARTITION OF orders
    FOR VALUES FROM ('2025-07-01') TO ('2025-10-01');

CREATE TABLE orders_2025_q4 PARTITION OF orders
    FOR VALUES FROM ('2025-10-01') TO ('2026-01-01');

-- Step 3: Insert data (automatically routed to correct partition)
INSERT INTO orders (customer_id, order_date, total) VALUES
    (1, '2025-02-15', 150.00),
    (2, '2025-05-20', 250.00),
    (3, '2025-08-10', 350.00),
    (4, '2025-11-25', 450.00);

-- Step 4: Query with partition pruning
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM orders WHERE order_date >= '2025-05-01' AND order_date < '2025-08-01';
```

**Expected Output**:
```
                                    QUERY PLAN
---------------------------------------------------------------------------------
 Append (actual time=0.015..0.018 rows=1 loops=1)
   ->  Seq Scan on orders_2025_q2 (actual time=0.010..0.011 rows=1 loops=1)
         Filter: ((order_date >= '2025-05-01'::date) AND (order_date < '2025-08-01'::date))
 Planning Time: 0.150 ms
 Execution Time: 0.050 ms
```

**Why This Output Occurs**: The `EXPLAIN` output shows only `orders_2025_q2` being scanned. The other three partitions are pruned because the `WHERE` clause's date range falls entirely within Q2's bounds. This demonstrates partition pruning in action.

#### Example 2: MySQL Range Partitioning

```sql
-- Step 1: Create a range-partitioned table
CREATE TABLE sales (
    id INT NOT NULL,
    sale_date DATE NOT NULL,
    amount DECIMAL(10,2),
    PRIMARY KEY (id, sale_date)
)
PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p2026 VALUES LESS THAN MAXVALUE
);

-- Step 2: Insert data
INSERT INTO sales VALUES
    (1, '2023-06-15', 100.00),
    (2, '2024-03-20', 200.00),
    (3, '2025-01-10', 300.00);

-- Step 3: Verify partition pruning
EXPLAIN SELECT * FROM sales WHERE sale_date BETWEEN '2024-01-01' AND '2024-12-31';
```

**Expected Output**:
```
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | sales | p2024      | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
```

**Why This Output Occurs**: The `partitions` column shows `p2024` only—the optimizer has pruned all other partitions based on the date range predicate. This reduces I/O significantly compared to scanning the entire table.

### Real-World Cases

**Case 1: Time-Series Data Management**: A monitoring system stores metrics in a table partitioned by day. Queries for the last 7 days access only 7 partitions, and old partitions are dropped after 90 days without affecting active data.

**Case 2: Sliding Window Archiving**: A financial system partitions transactions by month. At the start of each month, a new partition is created and the oldest is switched to an archive table, maintaining a fixed 24-month active window.

**Case 3: Storage Tiering**: A data warehouse partitions by year; recent years reside on fast SSD storage, while older years are moved to cheaper HDD storage—all transparent to queries.

---

## Core Concept 2: List Partitioning

### Definitions

**Core Definition**: List partitioning divides a table into partitions based on discrete, explicitly listed values of the partition key.

**Technical Definition**: In list partitioning, each partition is assigned a specific set of key values; rows with key values in that set are routed to the corresponding partition. Values not covered by any partition are rejected or placed in a default partition.

**Beginner-Friendly Explanation**: List partitioning is like organizing clothes by type: one drawer for shirts, one for pants, one for socks. Each item goes to a specific drawer based on its category.

### Purposes

- **To** organize data by categorical or discrete values (region, status, department)
- **To** enable partition pruning on equality and `IN` predicates
- **To** isolate frequently accessed categories into dedicated partitions
- **To** support data governance by placing sensitive categories in separate storage

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
CREATE TABLE table_name (
    column_definitions
) PARTITION BY LIST (partition_key_column);

CREATE TABLE table_name_partition_name
    PARTITION OF table_name
    FOR VALUES IN (value1, value2, ...);
```

#### Complete General Syntax (MySQL)

```sql
CREATE TABLE table_name (
    column_definitions
)
PARTITION BY LIST (partition_key_column) (
    PARTITION p1 VALUES IN (value1, value2, ...),
    PARTITION p2 VALUES IN (value3, value4, ...),
    ...
);
```

#### Complete General Syntax (Oracle)

```sql
CREATE TABLE table_name (
    column_definitions
)
PARTITION BY LIST (partition_key_column) (
    PARTITION p1 VALUES (value1, value2, ...),
    PARTITION p2 VALUES (value3, value4, ...),
    PARTITION p_default VALUES (DEFAULT)
);
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `PARTITION BY LIST` | Specifies list partitioning method |
| `FOR VALUES IN` | PostgreSQL: lists values for the partition |
| `VALUES IN` | MySQL: lists values for the partition |
| `VALUES` | Oracle: lists values for the partition |
| `DEFAULT` | Oracle/PostgreSQL: catches values not in any list |

#### Syntax Rules

- All values must be explicitly listed; values cannot overlap between partitions
- MySQL: Supports only integer and date/time types for list partitioning
- PostgreSQL: Supports any type with an equality operator
- Oracle: Supports a `DEFAULT` partition for unmatched values

#### Constraints and Limitations

- MySQL: List partitioning supports only integer and date/time columns (not strings)
- PostgreSQL: `DEFAULT` partition catches all values not explicitly listed
- Oracle: Only one `DEFAULT` partition is allowed
- List partitioning is unsuitable for high-cardinality continuous values

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL List Partitioning by Region

```sql
-- Step 1: Create a list-partitioned table
CREATE TABLE customers (
    customer_id SERIAL,
    customer_name TEXT,
    region TEXT NOT NULL
) PARTITION BY LIST (region);

-- Step 2: Create partitions for each region
CREATE TABLE customers_north PARTITION OF customers
    FOR VALUES IN ('North', 'Northeast', 'Northwest');

CREATE TABLE customers_south PARTITION OF customers
    FOR VALUES IN ('South', 'Southeast', 'Southwest');

CREATE TABLE customers_east PARTITION OF customers
    FOR VALUES IN ('East');

CREATE TABLE customers_west PARTITION OF customers
    FOR VALUES IN ('West');

CREATE TABLE customers_other PARTITION OF customers
    DEFAULT;

-- Step 3: Insert data
INSERT INTO customers (customer_name, region) VALUES
    ('Acme Corp', 'North'),
    ('Global Inc', 'South'),
    ('Tech Solutions', 'East'),
    ('Startup LLC', 'West'),
    ('Intl Corp', 'Central');

-- Step 4: Query with partition pruning
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM customers WHERE region IN ('North', 'South');
```

**Expected Output**:
```
                                       QUERY PLAN
-----------------------------------------------------------------------------------------
 Append (actual time=0.012..0.015 rows=2 loops=1)
   ->  Seq Scan on customers_north (actual time=0.008..0.009 rows=1 loops=1)
         Filter: (region = ANY ('{North,South}'::text[]))
   ->  Seq Scan on customers_south (actual time=0.006..0.007 rows=1 loops=1)
         Filter: (region = ANY ('{North,South}'::text[]))
 Planning Time: 0.120 ms
 Execution Time: 0.040 ms
```

**Why This Output Occurs**: The `EXPLAIN` output shows only `customers_north` and `customers_south` being scanned. The `East`, `West`, and `other` partitions are pruned because their values are not in the `IN` list. The `Central` region value routes to the `DEFAULT` partition.

### Real-World Cases

**Case 1: Multi-Tenant SaaS by Region**: A SaaS platform partitions customer data by region, enabling region-specific maintenance and compliance with data residency requirements.

**Case 2: E-Commerce Order Status**: An orders table partitions by status (`pending`, `shipped`, `delivered`, `cancelled`), allowing fast queries for active orders (pending/shipped) while archiving completed orders.

**Case 3: Departmental Data Isolation**: A university partitions student records by department, enabling departmental administrators to query only their partition while maintaining a unified logical view.

---

## Core Concept 3: Hash Partitioning

### Definitions

**Core Definition**: Hash partitioning divides a table into partitions by applying a hash function to the partition key and assigning rows based on the hash value.

**Technical Definition**: In hash partitioning, a hash function is applied to the partition key, and the result modulo the number of partitions determines the target partition. This distributes data evenly across partitions without user-defined ranges or lists.

**Beginner-Friendly Explanation**: Hash partitioning is like dealing a deck of cards to four players by shuffling first. The shuffle (hash) ensures each player gets a random but roughly equal number of cards. You can't predict which player gets which card, but the distribution is balanced.

### Purposes

- **To** distribute data evenly across partitions without manual range or list definitions
- **To** improve I/O parallelism by spreading writes across multiple storage devices
- **To** avoid hotspots that can occur with range or list partitioning
- **To** support partition-wise joins when both tables use the same hash partitioning

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
CREATE TABLE table_name (
    column_definitions
) PARTITION BY HASH (partition_key_column);

CREATE TABLE table_name_partition_name
    PARTITION OF table_name
    FOR VALUES WITH (MODULUS n, REMAINDER r);
```

#### Complete General Syntax (MySQL)

```sql
CREATE TABLE table_name (
    column_definitions
)
PARTITION BY HASH (partition_key_expression)
PARTITIONS number_of_partitions;
```

#### Complete General Syntax (Oracle)

```sql
CREATE TABLE table_name (
    column_definitions
)
PARTITION BY HASH (partition_key_column)
PARTITIONS number_of_partitions;
```

#### Complete General Syntax (SQL Server)

```sql
-- SQL Server does not have native HASH partitioning;
-- it uses partition functions with computed columns for hash-like distribution.
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `PARTITION BY HASH` | Specifies hash partitioning method |
| `MODULUS n` | PostgreSQL: number of partitions |
| `REMAINDER r` | PostgreSQL: which partition this table represents (0 to n-1) |
| `PARTITIONS n` | MySQL/Oracle: number of partitions |
| `partition_key_expression` | Expression whose hash value determines partition |

#### Syntax Rules

- PostgreSQL: Each partition must specify `MODULUS` and `REMAINDER`
- MySQL: `PARTITIONS` clause specifies the number of partitions
- Oracle: `PARTITIONS` clause specifies the number of partitions
- SQL Server: Hash-like distribution requires computed columns and range partitioning

#### Constraints and Limitations

- Hash partitioning does not support partition pruning on range predicates
- Equal hash values always go to the same partition, enabling partition-wise joins
- Adding or removing partitions requires redistributing data
- MySQL: Hash partitioning does not support `MAXVALUE` or `DEFAULT`

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL Hash Partitioning

```sql
-- Step 1: Create a hash-partitioned table
CREATE TABLE user_sessions (
    session_id UUID,
    user_id INT,
    login_time TIMESTAMP,
    ip_address INET
) PARTITION BY HASH (user_id);

-- Step 2: Create 4 partitions
CREATE TABLE user_sessions_p0 PARTITION OF user_sessions
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE user_sessions_p1 PARTITION OF user_sessions
    FOR VALUES WITH (MODULUS 4, REMAINDER 1);

CREATE TABLE user_sessions_p2 PARTITION OF user_sessions
    FOR VALUES WITH (MODULUS 4, REMAINDER 2);

CREATE TABLE user_sessions_p3 PARTITION OF user_sessions
    FOR VALUES WITH (MODULUS 4, REMAINDER 3);

-- Step 3: Insert data
INSERT INTO user_sessions (session_id, user_id, login_time, ip_address)
SELECT gen_random_uuid(), i, NOW(), '192.168.1.' || (i % 255)
FROM generate_series(1, 1000) AS i;

-- Step 4: Verify even distribution
SELECT tableoid::regclass AS partition, count(*)
FROM user_sessions
GROUP BY tableoid
ORDER BY tableoid;
```

**Expected Output**:
```
     partition      | count 
--------------------+-------
 user_sessions_p0   |   243
 user_sessions_p1   |   251
 user_sessions_p2   |   255
 user_sessions_p3   |   251
(4 rows)
```

**Why This Output Occurs**: Hash partitioning distributes rows roughly evenly across the four partitions (roughly 250 rows each). The exact counts depend on the hash function and data values, but the distribution is statistically balanced.

#### Example 2: MySQL Hash Partitioning

```sql
-- Step 1: Create a hash-partitioned table
CREATE TABLE sensor_readings (
    reading_id INT AUTO_INCREMENT,
    sensor_id INT NOT NULL,
    reading_value DECIMAL(10,2),
    reading_time TIMESTAMP,
    PRIMARY KEY (reading_id, sensor_id)
)
PARTITION BY HASH (sensor_id)
PARTITIONS 8;

-- Step 2: Insert data
INSERT INTO sensor_readings (sensor_id, reading_value, reading_time)
SELECT (i % 100) + 1, RAND() * 100, NOW()
FROM generate_series(1, 1000) AS i;

-- Step 3: Check partition distribution
SELECT 
    PARTITION_NAME,
    TABLE_ROWS
FROM INFORMATION_SCHEMA.PARTITIONS
WHERE TABLE_NAME = 'sensor_readings'
ORDER BY PARTITION_ORDINAL_POSITION;
```

**Expected Output**:
```
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p0             |        125 |
| p1             |        127 |
| p2             |        122 |
| p3             |        128 |
| p4             |        124 |
| p5             |        125 |
| p6             |        123 |
| p7             |        126 |
+----------------+------------+
```

**Why This Output Occurs**: MySQL's `HASH` partitioning distributes rows across 8 partitions using the hash of `sensor_id`. The `TABLE_ROWS` column in `INFORMATION_SCHEMA.PARTITIONS` shows approximately equal distribution.

### Real-World Cases

**Case 1: Session Data Distribution**: A web application partitions session data by `user_id` hash, ensuring even load distribution across storage devices without hotspots.

**Case 2: IoT Sensor Data**: An IoT platform uses hash partitioning by `sensor_id` to distribute sensor readings across multiple disks, enabling parallel writes and reads.

**Case 3: Partition-Wise Joins**: Two large tables are hash-partitioned on the same join key, enabling partition-wise joins that reduce network and I/O overhead.

---

## Core Concept 4: Composite Partitioning

### Definitions

**Core Definition**: Composite partitioning applies two levels of partitioning—a primary method (e.g., range) and a secondary method (e.g., list or hash)—creating subpartitions within each partition.

**Technical Definition**: Composite partitioning (also called sub-partitioning) involves partitioning a table by one method and then further partitioning each partition by another method, producing a two-level hierarchy of partitions and subpartitions.

**Beginner-Friendly Explanation**: Composite partitioning is like organizing a library first by floor (range: year), then by section (list: genre). Each floor has its own set of genre sections, giving you two levels of organization.

### Purposes

- **To** combine the benefits of two partitioning methods (e.g., range for manageability, hash for distribution)
- **To** enable partition-wise joins along two dimensions
- **To** support complex data lifecycle policies (e.g., range by date, hash by customer)
- **To** isolate data at a finer granularity for maintenance and query optimization

### Syntax Rules and Structure

#### Complete General Syntax (Oracle — Range-List)

```sql
CREATE TABLE table_name (
    column_definitions
)
PARTITION BY RANGE (range_key_column)
SUBPARTITION BY LIST (list_key_column)
SUBPARTITION TEMPLATE (
    SUBPARTITION sub1 VALUES ('value1'),
    SUBPARTITION sub2 VALUES ('value2'),
    ...
)
(
    PARTITION p1 VALUES LESS THAN (value1),
    PARTITION p2 VALUES LESS THAN (value2),
    ...
);
```

#### Complete General Syntax (Oracle — Range-Hash)

```sql
CREATE TABLE table_name (
    column_definitions
)
PARTITION BY RANGE (range_key_column)
SUBPARTITION BY HASH (hash_key_column)
SUBPARTITIONS number_of_subpartitions
(
    PARTITION p1 VALUES LESS THAN (value1),
    PARTITION p2 VALUES LESS THAN (value2)
);
```

#### Complete General Syntax (PostgreSQL — Range-List)

```sql
-- PostgreSQL supports sub-partitioning by declaring a partition as partitioned
CREATE TABLE table_name (
    column_definitions
) PARTITION BY RANGE (range_key_column);

CREATE TABLE table_name_p1 PARTITION OF table_name
    FOR VALUES FROM (value1) TO (value2)
    PARTITION BY LIST (list_key_column);

CREATE TABLE table_name_p1_sub1 PARTITION OF table_name_p1
    FOR VALUES IN (value_a);
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `PARTITION BY` | Primary partitioning method |
| `SUBPARTITION BY` | Secondary partitioning method |
| `SUBPARTITION TEMPLATE` | Oracle: defines subpartition structure for all partitions |
| `SUBPARTITIONS n` | Oracle: number of hash subpartitions per partition |

#### Syntax Rules

- Oracle: `SUBPARTITION TEMPLATE` ensures uniform subpartition definitions across all partitions
- PostgreSQL: Sub-partitioning is achieved by declaring a partition as partitioned
- MySQL: Supports composite partitioning with `SUBPARTITION BY` clause
- All partitions must have the same number of subpartitions (MySQL requirement)

#### Constraints and Limitations

- Composite partitioning increases metadata overhead and query complexity
- Oracle: `SUBPARTITION TEMPLATE` cannot be changed after table creation
- PostgreSQL: Sub-partitioned tables cannot have a primary key on the parent table
- MySQL: All partitions must have the same number of subpartitions

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Oracle Range-List Composite Partitioning

```sql
-- Step 1: Create a range-list composite partitioned table
CREATE TABLE sales (
    sale_id NUMBER,
    sale_date DATE NOT NULL,
    region VARCHAR2(20) NOT NULL,
    amount NUMBER(10,2)
)
PARTITION BY RANGE (sale_date)
SUBPARTITION BY LIST (region)
SUBPARTITION TEMPLATE (
    SUBPARTITION north VALUES ('North'),
    SUBPARTITION south VALUES ('South'),
    SUBPARTITION east VALUES ('East'),
    SUBPARTITION west VALUES ('West')
)
(
    PARTITION sales_2025_q1 VALUES LESS THAN (TO_DATE('2025-04-01', 'YYYY-MM-DD')),
    PARTITION sales_2025_q2 VALUES LESS THAN (TO_DATE('2025-07-01', 'YYYY-MM-DD')),
    PARTITION sales_2025_q3 VALUES LESS THAN (TO_DATE('2025-10-01', 'YYYY-MM-DD')),
    PARTITION sales_2025_q4 VALUES LESS THAN (TO_DATE('2026-01-01', 'YYYY-MM-DD'))
);

-- Step 2: Query subpartition information
SELECT partition_name, subpartition_name, high_value
FROM user_tab_subpartitions
WHERE table_name = 'SALES'
ORDER BY partition_name, subpartition_name;
```

**Expected Output** (excerpt):
```
PARTITION_NAME    SUBPARTITION_NAME    HIGH_VALUE
----------------  -------------------  ---------------------------
SALES_2025_Q1     SALES_2025_Q1_NORTH  'North'
SALES_2025_Q1     SALES_2025_Q1_SOUTH  'South'
SALES_2025_Q1     SALES_2025_Q1_EAST   'East'
SALES_2025_Q1     SALES_2025_Q1_WEST   'West'
SALES_2025_Q2     SALES_2025_Q2_NORTH  'North'
...
```

**Why This Output Occurs**: Each range partition (quarter) is subdivided into four list subpartitions (regions). Queries filtering by both date and region prune to a single subpartition.

#### Example 2: PostgreSQL Range-List Sub-Partitioning

```sql
-- Step 1: Create the top-level range-partitioned table
CREATE TABLE orders (
    order_id SERIAL,
    order_date DATE NOT NULL,
    region TEXT NOT NULL,
    amount NUMERIC(10,2)
) PARTITION BY RANGE (order_date);

-- Step 2: Create a partition that is itself partitioned by list
CREATE TABLE orders_2025 PARTITION OF orders
    FOR VALUES FROM ('2025-01-01') TO ('2026-01-01')
    PARTITION BY LIST (region);

-- Step 3: Create sub-partitions
CREATE TABLE orders_2025_north PARTITION OF orders_2025
    FOR VALUES IN ('North');

CREATE TABLE orders_2025_south PARTITION OF orders_2025
    FOR VALUES IN ('South');

CREATE TABLE orders_2025_default PARTITION OF orders_2025
    DEFAULT;

-- Step 4: Insert and query
INSERT INTO orders (order_date, region, amount) VALUES
    ('2025-06-15', 'North', 100.00),
    ('2025-06-20', 'South', 200.00);

EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM orders 
WHERE order_date >= '2025-06-01' AND order_date < '2025-07-01'
  AND region = 'North';
```

**Expected Output**:
```
                                            QUERY PLAN
--------------------------------------------------------------------------------------------------
 Append (actual time=0.012..0.014 rows=1 loops=1)
   ->  Seq Scan on orders_2025_north (actual time=0.008..0.009 rows=1 loops=1)
         Filter: ((order_date >= '2025-06-01'::date) AND (order_date < '2025-07-01'::date) 
                  AND (region = 'North'::text))
 Planning Time: 0.180 ms
 Execution Time: 0.045 ms
```

**Why This Output Occurs**: The query prunes to the `orders_2025_north` subpartition. The range predicate eliminates all other years, and the list predicate eliminates the `south` and `default` subpartitions within 2025.

### Real-World Cases

**Case 1: Data Warehouse with Time and Region**: A data warehouse partitions sales data by month (range) and then by region (list), enabling efficient queries for "Q2 2025 North region sales."

**Case 2: Multi-Tenant SaaS with Time and Tenant ID**: A SaaS platform partitions by month (range) and subpartitions by tenant ID (hash), combining time-based retention with even tenant distribution.

**Case 3: Compliance-Driven Data Isolation**: A healthcare system partitions by year (range) and subpartitions by data sensitivity (list: public, restricted, confidential), enabling separate security policies per subpartition.

---

## Core Concept 5: Partition Pruning

### Definitions

**Core Definition**: Partition pruning is the query optimizer's ability to eliminate irrelevant partitions from a query's execution plan based on predicates in the `WHERE` clause.

**Technical Definition**: Partition pruning is an optimization technique where the database's query planner analyzes the `FROM` and `WHERE` clauses of a SQL statement, constructs a partition access list, and removes partitions that cannot contain matching rows—reflected in the execution plan's `PSTART`/`PSTOP` columns (Oracle) or the `partitions` column (MySQL) or `Append` node children (PostgreSQL).

**Beginner-Friendly Explanation**: Partition pruning is like knowing which drawers to open. If you're looking for a 2025 document, you don't open the 2024 drawer. The database does the same thing—it only scans the partitions that could contain your data.

### Purposes

- **To** dramatically reduce I/O by scanning only relevant partitions
- **To** improve query response time for partitioned tables
- **To** enable efficient partition-wise joins and aggregations
- **To** reduce CPU and memory consumption for large-table queries

### Syntax Rules and Structure

#### Partition Pruning Verification (PostgreSQL)

```sql
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM table_name WHERE partition_key = 'value';

-- Look for "Subplans Removed" or count the Append children
```

#### Partition Pruning Verification (MySQL)

```sql
EXPLAIN SELECT * FROM table_name WHERE partition_key = 'value';
-- Check the "partitions" column in the output
```

#### Partition Pruning Verification (Oracle)

```sql
EXPLAIN PLAN FOR
SELECT * FROM table_name WHERE partition_key = 'value';

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
-- Look for PSTART and PSTOP columns
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `PSTART` / `PSTOP` | Oracle: partition start and stop numbers in execution plan |
| `partitions` | MySQL: shows which partitions are accessed |
| `Subplans Removed` | PostgreSQL: number of partitions pruned |
| `Append` node | PostgreSQL: lists scanned partitions as children |

#### Syntax Rules

- Pruning occurs at **plan time** for constant predicates and at **execution time** for parameterized predicates
- Functions applied to the partition key generally disable pruning (e.g., `WHERE YEAR(date) = 2025`)
- Pruning works with equality (`=`), range (`<`, `>`, `BETWEEN`), and `IN` predicates

#### Constraints and Limitations

- Applying functions to partition columns disables pruning (Oracle)
- Implicit data type conversions can prevent pruning
- Pruning does not occur for non-partition-key columns
- Hash partitioning supports pruning only with equality and `IN` predicates
- PostgreSQL: Pruning with parameterized nested loop joins may require `enable_partitionwise_join`

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL Partition Pruning Verification

```sql
-- Step 1: Create a range-partitioned table
CREATE TABLE measurements (
    city_id INT,
    logdate DATE NOT NULL,
    peaktemp INT
) PARTITION BY RANGE (logdate);

CREATE TABLE measurements_2025_m01 PARTITION OF measurements
    FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');
CREATE TABLE measurements_2025_m02 PARTITION OF measurements
    FOR VALUES FROM ('2025-02-01') TO ('2025-03-01');
CREATE TABLE measurements_2025_m03 PARTITION OF measurements
    FOR VALUES FROM ('2025-03-01') TO ('2025-04-01');

-- Step 2: Insert data
INSERT INTO measurements VALUES
    (1, '2025-01-15', 10),
    (2, '2025-02-15', 20),
    (3, '2025-03-15', 30);

-- Step 3: Verify pruning with a range predicate
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM measurements 
WHERE logdate >= '2025-02-01' AND logdate < '2025-03-01';
```

**Expected Output**:
```
                                          QUERY PLAN
----------------------------------------------------------------------------------------------
 Append (actual time=0.010..0.012 rows=1 loops=1)
   Subplans Removed: 2
   ->  Seq Scan on measurements_2025_m02 (actual time=0.008..0.009 rows=1 loops=1)
         Filter: ((logdate >= '2025-02-01'::date) AND (logdate < '2025-03-01'::date))
 Planning Time: 0.140 ms
 Execution Time: 0.040 ms
```

**Why This Output Occurs**: `Subplans Removed: 2` indicates two partitions were pruned. Only `measurements_2025_m02` remains. This is plan-time pruning because the bounds are constants.

#### Example 2: MySQL Partition Pruning with EXPLAIN

```sql
-- Step 1: Create a list-partitioned table
CREATE TABLE orders (
    order_id INT NOT NULL,
    region VARCHAR(20) NOT NULL,
    amount DECIMAL(10,2),
    PRIMARY KEY (order_id, region)
)
PARTITION BY LIST COLUMNS(region) (
    PARTITION p_north VALUES IN ('North', 'Northeast'),
    PARTITION p_south VALUES IN ('South', 'Southeast'),
    PARTITION p_east VALUES IN ('East'),
    PARTITION p_west VALUES IN ('West')
);

-- Step 2: Insert data
INSERT INTO orders VALUES
    (1, 'North', 100.00),
    (2, 'South', 200.00),
    (3, 'East', 300.00);

-- Step 3: Verify pruning
EXPLAIN SELECT * FROM orders WHERE region = 'North';
```

**Expected Output**:
```
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table  | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | orders | p_north    | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using where |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
```

**Why This Output Occurs**: The `partitions` column shows `p_north` only. The other three partitions are pruned because the `WHERE` clause uses an equality predicate on the partition key.

### Real-World Cases

**Case 1: Time-Range Reports**: A financial dashboard queries "last quarter" data. Partition pruning ensures only the relevant quarter's partition is scanned, reducing query time from seconds to milliseconds.

**Case 2: Multi-Region Query Optimization**: A global retailer queries regional sales. List partitioning by region enables pruning to the specific region's partition, avoiding a full table scan.

**Case 3: Partition-Wise Joins**: Two tables partitioned on the same key are joined. The optimizer performs a partition-wise join, matching corresponding partitions and reducing data movement.

---

## Core Concept 6: Partition Maintenance Operations

### Definitions

**Core Definition**: Partition maintenance operations are DDL statements that add, drop, split, merge, switch, or truncate partitions to manage data lifecycle.

**Technical Definition**: Partition maintenance operations include `ADD PARTITION`, `DROP PARTITION`, `SPLIT PARTITION`, `MERGE PARTITIONS`, `SWITCH PARTITION` (SQL Server), `TRUNCATE PARTITION`, and `EXCHANGE PARTITION` (Oracle/MySQL), each implemented through `ALTER TABLE` statements and affecting only the target partition's metadata and data.

**Beginner-Friendly Explanation**: Partition maintenance is like managing the drawers in your filing cabinet. You can add a new drawer for the coming year, archive and remove an old drawer, or split a crowded drawer into two. These operations affect only the drawer you're working on—the rest of the cabinet stays untouched.

### Purposes

- **To** automate partition lifecycle (create new, drop old) for rolling windows
- **To** archive or purge data by switching or dropping partitions
- **To** reorganize partitions by splitting or merging to match data distribution
- **To** perform partition-level maintenance (rebuild indexes, update statistics) efficiently

### Sub-Concept 6.1: Automated Partition Creation (pg_partman)

#### Syntax Rules and Structure

```sql
-- Install pg_partman extension
CREATE EXTENSION pg_partman;

-- Create a partition set
SELECT partman.create_parent(
    p_parent_table := 'public.orders',
    p_control := 'order_date',
    p_type := 'range',
    p_interval := '1 month',
    p_premake := 4  -- Create 4 future partitions
);

-- Schedule maintenance
SELECT partman.run_maintenance();
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `p_parent_table` | Parent partitioned table |
| `p_control` | Partition key column |
| `p_type` | Partitioning type (range, list, hash) |
| `p_interval` | Partition interval (e.g., '1 month', '1 day') |
| `p_premake` | Number of future partitions to pre-create |

#### Constraints and Limitations

- pg_partman requires a background worker or external scheduler (cron) for automatic maintenance
- Requires the `pg_partman_bgw` background worker for fully automatic operation
- Retention policies can automatically drop old partitions

### Sub-Concept 6.2: Partition Splitting and Merging

#### Syntax Rules and Structure (MySQL)

```sql
-- Split a partition
ALTER TABLE table_name REORGANIZE PARTITION p1 INTO (
    PARTITION p1a VALUES LESS THAN (value1),
    PARTITION p1b VALUES LESS THAN (value2)
);

-- Merge partitions
ALTER TABLE table_name REORGANIZE PARTITION p1a, p1b INTO (
    PARTITION p1 VALUES LESS THAN (value2)
);
```

#### Syntax Rules and Structure (Oracle)

```sql
-- Split a partition
ALTER TABLE table_name SPLIT PARTITION p1 AT (value) INTO (
    PARTITION p1a, PARTITION p1b
);

-- Merge partitions
ALTER TABLE table_name MERGE PARTITIONS p1a, p1b INTO PARTITION p1;
```

#### Constraints and Limitations

- MySQL: `REORGANIZE PARTITION` can split or merge but cannot change the partitioning method
- Oracle: `SPLIT PARTITION` can split range and list partitions but not hash partitions
- Splitting a non-empty partition may require data movement

### Sub-Concept 6.3: Partition Dropping and Switching

#### Syntax Rules and Structure (SQL Server)

```sql
-- Switch a partition to an archive table
ALTER TABLE Orders SWITCH PARTITION 24 TO OrdersArchive;

-- Truncate a partition
TRUNCATE TABLE Orders WITH (PARTITIONS (24));

-- Merge two partitions
ALTER PARTITION FUNCTION PF_OrderDate() MERGE RANGE ('2025-01-01');
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `SWITCH PARTITION` | Moves partition to another table (metadata operation) |
| `TRUNCATE TABLE ... WITH (PARTITIONS (...))` | Removes all rows from specified partitions |
| `MERGE RANGE` | Removes a boundary from the partition function |

#### Constraints and Limitations

- SQL Server: `SWITCH` requires matching filegroup, schema, and compression settings
- SQL Server: `SWITCH` is a metadata operation—no data movement
- SQL Server: `MERGE RANGE` combines two adjacent partitions
- Oracle: `EXCHANGE PARTITION` swaps a partition with a standalone table

### Annotated Complete Step-by-Step Code Examples

#### Example 1: MySQL Partition Reorganization

```sql
-- Step 1: Create a range-partitioned table
CREATE TABLE logs (
    id INT NOT NULL,
    log_date DATE NOT NULL,
    message TEXT,
    PRIMARY KEY (id, log_date)
)
PARTITION BY RANGE (YEAR(log_date)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p2026 VALUES LESS THAN MAXVALUE
);

-- Step 2: Split p2025 into two half-year partitions
ALTER TABLE logs REORGANIZE PARTITION p2025 INTO (
    PARTITION p2025_h1 VALUES LESS THAN ('2025-07-01'),
    PARTITION p2025_h2 VALUES LESS THAN (2026)
);

-- Step 3: Verify new partitions
SELECT PARTITION_NAME, PARTITION_DESCRIPTION
FROM INFORMATION_SCHEMA.PARTITIONS
WHERE TABLE_NAME = 'logs'
ORDER BY PARTITION_ORDINAL_POSITION;
```

**Expected Output**:
```
+----------------+-----------------------+
| PARTITION_NAME | PARTITION_DESCRIPTION |
+----------------+-----------------------+
| p2023          | 2024                  |
| p2024          | 2025                  |
| p2025_h1       | '2025-07-01'          |
| p2025_h2       | 2026                  |
| p2026          | MAXVALUE              |
+----------------+-----------------------+
```

**Why This Output Occurs**: `REORGANIZE PARTITION` splits `p2025` into `p2025_h1` and `p2025_h2`. The `PARTITION_DESCRIPTION` column shows the new boundaries.

#### Example 2: SQL Server Sliding Window Archiving

```sql
-- Step 1: Create partition function and scheme (simplified)
CREATE PARTITION FUNCTION PF_OrderDate (DATE)
AS RANGE RIGHT FOR VALUES ('2024-01-01', '2025-01-01', '2026-01-01');

CREATE PARTITION SCHEME PS_OrderDate
AS PARTITION PF_OrderDate TO (fg2024, fg2025, fg2026, fg2027);

-- Step 2: Create partitioned table
CREATE TABLE Orders (
    OrderID INT,
    OrderDate DATE,
    Amount DECIMAL(10,2)
) ON PS_OrderDate(OrderDate);

-- Step 3: Switch oldest partition to archive table
-- First, create identical archive table
CREATE TABLE OrdersArchive (
    OrderID INT,
    OrderDate DATE,
    Amount DECIMAL(10,2)
) ON fg2024;

-- Switch partition 1 (2024 data) to archive
ALTER TABLE Orders SWITCH PARTITION 1 TO OrdersArchive;

-- Step 4: Merge the empty partition
ALTER PARTITION FUNCTION PF_OrderDate() MERGE RANGE ('2024-01-01');

-- Step 5: Add new boundary for 2027
ALTER PARTITION FUNCTION PF_OrderDate() SPLIT RANGE ('2027-01-01');
```

**Why This Output Occurs**: `SWITCH PARTITION` moves the 2024 data to `OrdersArchive` as a metadata operation. `MERGE RANGE` removes the now-empty 2024 boundary. `SPLIT RANGE` adds a 2027 boundary, completing the sliding window.

### Real-World Cases

**Case 1: Rolling Window Log Management**: A logging system uses pg_partman to automatically create daily partitions and drop partitions older than 90 days, maintaining a fixed active window.

**Case 2: Monthly Sales Archiving**: A retail system uses SQL Server partition switching to archive the oldest month's sales to an archive table, then merges the empty partition and splits a new one—all within seconds.

**Case 3: Dynamic Partition Splitting**: A MySQL table's partitions grow unevenly. `REORGANIZE PARTITION` splits crowded partitions into smaller ones, balancing data distribution.

---

## Core Concept 7: Partitioning Constraints & Indexes

### Definitions

**Core Definition**: Partitioning constraints and indexes define how primary keys, unique constraints, and indexes interact with partitioned tables, distinguishing local (partition-aligned) from global (cross-partition) indexes.

**Technical Definition**: Local indexes are equipartitioned with the table—each index partition maps to exactly one table partition. Global indexes are partitioned independently of the table and can contain keys from multiple table partitions. Unique constraints on partitioned tables require the partitioning key to be a subset of the unique key for local indexes, or a global index for non-partition-key uniqueness.

**Beginner-Friendly Explanation**: Local indexes are like having a separate index card box for each drawer of your filing cabinet—one box per drawer. Global indexes are like having a master index that covers all drawers. Local indexes are faster to maintain but can only enforce uniqueness within a drawer; global indexes can enforce uniqueness across all drawers but are more complex to maintain.

### Purposes

- **To** enforce uniqueness across partitions using global indexes
- **To** enable partition-level index maintenance (rebuild one partition at a time)
- **To** optimize query performance by aligning indexes with partitions
- **To** support partition switching by ensuring index alignment

### Sub-Concept 7.1: Local Partitioned Indexes

#### Syntax Rules and Structure (PostgreSQL)

```sql
-- Create a local index on a partitioned table (PostgreSQL 11+)
CREATE INDEX idx_orders_customer ON orders (customer_id);

-- Unique local index (requires partition key in index)
CREATE UNIQUE INDEX idx_orders_pk ON orders (order_id, order_date);
```

#### Syntax Rules and Structure (Oracle)

```sql
-- Create a local index
CREATE INDEX idx_orders_customer ON orders (customer_id) LOCAL;

-- Unique local index (partition key must be subset of index key)
CREATE UNIQUE INDEX idx_orders_pk ON orders (order_id, order_date) LOCAL;
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `LOCAL` | Oracle: creates a local index (equipartitioned with table) |
| `UNIQUE` | Enforces uniqueness within the partition |
| Partition key in index | Required for unique local indexes |

#### Constraints and Limitations

- PostgreSQL: Unique local indexes ensure uniqueness only within each partition, not across partitions
- Oracle: Unique local nonprefixed indexes require the partitioning key to be a subset of the index key
- Local indexes cannot enforce uniqueness on columns that do not include the partition key
- Bitmap indexes on partitioned tables are always local (Oracle)

### Sub-Concept 7.2: Global Partitioned Indexes

#### Syntax Rules and Structure (Oracle)

```sql
-- Create a global partitioned index
CREATE INDEX idx_orders_customer ON orders (customer_id)
GLOBAL PARTITION BY RANGE (customer_id) (
    PARTITION p1 VALUES LESS THAN (1000),
    PARTITION p2 VALUES LESS THAN (2000),
    PARTITION p3 VALUES LESS THAN (MAXVALUE)
);

-- Non-partitioned global index
CREATE INDEX idx_orders_customer ON orders (customer_id) GLOBAL;
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `GLOBAL` | Oracle: creates a global index (independently partitioned) |
| `GLOBAL PARTITION BY` | Specifies a different partitioning scheme than the table |

#### Constraints and Limitations

- Global indexes can enforce uniqueness on columns not containing the partition key
- Global indexes are more complex to maintain during partition operations
- Oracle: Only B-tree indexes can be global; bitmap indexes must be local
- Global indexes may become unusable after partition maintenance operations

### Sub-Concept 7.3: SQL Server Aligned Indexes

#### Syntax Rules and Structure

```sql
-- Create an aligned index (default behavior)
CREATE INDEX IX_Orders_Date ON Orders (OrderDate)
ON PS_OrderDate(OrderDate);

-- Create a non-aligned index on a different partition scheme
CREATE INDEX IX_Orders_Customer ON Orders (CustomerID)
ON PS_Customer(CustomerID);
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `ON PS_OrderDate(OrderDate)` | Aligns index with the table's partition scheme |
| `ON PS_Customer(CustomerID)` | Creates a non-aligned index on a different scheme |

#### Constraints and Limitations

- Aligned indexes enable partition switching without rebuilding indexes
- Non-aligned indexes prevent partition switching (must drop and recreate)
- SQL Server recommends aligned indexes only when partitions exceed 1,000
- Unique constraints on partitioned tables require the partitioning column to be part of the unique key

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL Local Unique Index

```sql
-- Step 1: Create a partitioned table
CREATE TABLE orders (
    order_id INT,
    order_date DATE NOT NULL,
    customer_id INT,
    amount NUMERIC(10,2)
) PARTITION BY RANGE (order_date);

CREATE TABLE orders_2025 PARTITION OF orders
    FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');

CREATE TABLE orders_2026 PARTITION OF orders
    FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');

-- Step 2: Create a unique local index (includes partition key)
CREATE UNIQUE INDEX orders_pk ON orders (order_id, order_date);

-- Step 3: Test uniqueness within partition
INSERT INTO orders VALUES (1, '2025-06-15', 100, 99.99);
INSERT INTO orders VALUES (1, '2026-06-15', 100, 99.99);  -- Allowed (different partitions)

-- Step 4: Test duplicate within same partition
INSERT INTO orders VALUES (1, '2025-07-15', 100, 99.99);
-- Expected: ERROR: duplicate key value violates unique constraint "orders_2025_pk"
-- DETAIL: Key (order_id, order_date)=(1, 2025-07-15) already exists.
```

**Why This Output Occurs**: The unique local index enforces uniqueness per partition. The same `order_id` can exist in different partitions (2025 and 2026) but not twice within the same partition.

#### Example 2: Oracle Global Unique Index

```sql
-- Step 1: Create a range-partitioned table
CREATE TABLE sales (
    sale_id NUMBER,
    sale_date DATE NOT NULL,
    customer_id NUMBER NOT NULL,
    amount NUMBER(10,2)
)
PARTITION BY RANGE (sale_date) (
    PARTITION p2025 VALUES LESS THAN (TO_DATE('2026-01-01', 'YYYY-MM-DD')),
    PARTITION p2026 VALUES LESS THAN (TO_DATE('2027-01-01', 'YYYY-MM-DD'))
);

-- Step 2: Create a global unique index on customer_id
CREATE UNIQUE INDEX sales_customer_uk ON sales (customer_id) GLOBAL;

-- Step 3: Test global uniqueness
INSERT INTO sales VALUES (1, TO_DATE('2025-06-15', 'YYYY-MM-DD'), 100, 500);
INSERT INTO sales VALUES (2, TO_DATE('2026-06-15', 'YYYY-MM-DD'), 100, 600);
-- Expected: ORA-00001: unique constraint (SALES_CUSTOMER_UK) violated
```

**Why This Output Occurs**: The global unique index enforces uniqueness across **all partitions**. The same `customer_id` cannot exist in both the 2025 and 2026 partitions, demonstrating global uniqueness enforcement.

### Real-World Cases

**Case 1: Global Unique Constraint**: A user table partitioned by `created_date` requires unique `email` across all partitions. A global unique index on `email` enforces this constraint.

**Case 2: Partition Switching with Aligned Indexes**: A SQL Server table uses aligned indexes, enabling `SWITCH PARTITION` to archive old data without dropping and rebuilding indexes.

**Case 3: Local Index for Partition-Wise Maintenance**: A PostgreSQL table with local indexes allows rebuilding one partition's index at a time, reducing maintenance windows from hours to minutes.

---

## References

| Name | Link |
|------|------|
| PostgreSQL Documentation — Table Partitioning | https://www.postgresql.org/docs/current/ddl-partitioning.html |
| MySQL 8.0 Reference Manual — Partitioning | https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/partitioning.html |
| MySQL 9.7 Reference Manual — Partition Management | https://docs.oracle.com/cd/E17952_01/mysql-9.7-en/partitioning-management.html |
| Microsoft Learn — Partitioned Tables and Indexes | https://learn.microsoft.com/en-us/sql/relational-databases/partitions/partitioned-tables-and-indexes |
| Oracle VLDB and Partitioning Guide — Partition Pruning | https://docs.oracle.com/cd/G11854_01/vldbg/vldb-and-partitioning-guide.pdf |
| Oracle Database VLDB and Partitioning Guide — Composite Partitioning | https://docs.oracle.com/cd/G11854_01/vldbg/vldb-and-partitioning-guide.pdf |
| Oracle Database VLDB and Partitioning Guide — Local and Global Indexes | https://docs.oracle.com/cd/G11854_01/vldbg/vldb-and-partitioning-guide.pdf |
| MySQL 9.7 Reference Manual — Partition Pruning | https://dev.mysql.com/doc/refman/9.7/en/partitioning-pruning.html |
| PostgreSQL Documentation — Partition Pruning | https://www.postgresql.org/docs/current/ddl-partitioning.html |
| pg_partman Documentation | https://github.com/pgpartman/pg_partman |
| Microsoft Learn — SQL Server Partition Switching | https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-table-transact-sql |
| Oracle Live SQL — Composite Hash-Hash Partitioning | https://livesql.oracle.com/ |
| Alibaba Cloud — Indexes in Partitioned Tables | https://help.aliyun.com/ |
| AWS — SQL Server to Aurora PostgreSQL Migration Playbook — Partitioning | https://docs.aws.amazon.com/dms/latest/sql-server-to-aurora-postgresql-migration-playbook/ |