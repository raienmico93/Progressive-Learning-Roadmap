# Database Fundamentals & Design — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Database fundamentals and design encompass the principles, techniques, and best practices for structuring, organizing, and querying relational databases to ensure data integrity, minimize redundancy, and optimize performance.

**Technical Definition**

Relational database design is the process of organizing data into tables (relations) with well-defined columns (attributes) and rows (tuples), connected through keys and constraints that enforce integrity. The theoretical foundation rests on E. F. Codd's relational model, which separates the logical schema (what data is stored and how it relates) from the physical storage (how it is stored on disk). Normalization — the process of decomposing tables to eliminate redundancy and anomalies — is formalized through a series of normal forms (1NF, 2NF, 3NF, BCNF). Performance is governed by indexes (B-Tree structures), which accelerate reads at the cost of write overhead. Schema evolution is managed through programmatic migrations, which version-control database changes alongside application code.

**Beginner-Friendly Explanation**

Imagine you are organizing a library. You do not throw all books into a single pile. Instead, you create sections (tables): fiction, non-fiction, reference. Within each section, every book has a unique call number (primary key) so you can find it instantly. You create a card catalogue (indexes) to speed up searches. When the library grows, you add new shelves (migrations) without rebuilding the entire building. Database design is the art of organizing data so it is easy to find, hard to corrupt, and efficient to maintain.

### Key Characteristics

- **Data Integrity:** Constraints (PRIMARY KEY, FOREIGN KEY, UNIQUE, NOT NULL, CHECK) enforce rules at the database level, preventing invalid data from entering the system.
- **Normalization:** Tables are decomposed to eliminate redundancy and update/insert/delete anomalies. Most transactional systems target 3NF.
- **Indexing:** B-Tree indexes accelerate `SELECT` queries and enforce uniqueness, but add write overhead on `INSERT`, `UPDATE`, and `DELETE`.
- **Schema Version Control:** Migrations treat schema changes as versioned, reversible code artifacts rather than manual SQL scripts.
- **Set-Based Operations:** SQL operates on entire sets of rows, not individual records; `JOIN`s, aggregates, and `GROUP BY` leverage this set-based nature.
- **Declarative Queries:** SQL describes what data is needed, not how to retrieve it; the query optimizer chooses the execution plan.

### Prerequisites

- **A relational database management system** (MySQL 8.0+, MariaDB 10.6+, PostgreSQL 14+, or SQLite 3).
- **PHP 8.1+** with PDO and a driver extension (`pdo_mysql`, `pdo_pgsql`, `pdo_sqlite`) for executing queries.
- **Composer** for installing migration libraries.
- **A SQL client** (phpMyAdmin, DBeaver, psql, sqlite3) for exploring schemas and running ad-hoc queries.
- **Basic understanding of data types** (INTEGER, VARCHAR, TEXT, DECIMAL, DATETIME, BOOLEAN).

### Related Programming Areas

- **Secure Query Execution & Prepared Statements:** All CRUD operations and migrations should use prepared statements to prevent SQL injection.
- **Advanced Query Patterns & Performance Optimization:** Pagination, eager loading, and indexing strategies build on the fundamentals covered here.
- **Object-Relational Mapping (ORM):** ORMs like Eloquent and Doctrine abstract schema design and query execution, but understanding the underlying principles is essential for debugging and optimization.
- **Data Integrity & PHP Transactions:** Transactions rely on the ACID properties that InnoDB (MySQL) and PostgreSQL provide.

### Core Concepts / Features

1. **Relational Schema Design** — Tables, normal forms, primary keys, composite keys, and foreign keys.
2. **Indexes & Performance** — B-Tree indexes, unique constraints, and the read/write trade-off.
3. **Database Migrations & Version Control** — Programmatic schema changes with `up()`/`down()` methods.
4. **SQL Fundamentals** — `SELECT`, `INSERT`, `UPDATE`, `DELETE`, aggregates, `GROUP BY`, and `JOIN`s.

---

## Core Concept 1: Relational Schema Design

### Definitions

**Core Definition**

Relational schema design is the process of defining the structure of a relational database — its tables, columns, data types, keys, and constraints — to accurately model a domain while preserving data integrity and minimizing redundancy.

**Technical Definition**

A relational schema is a set of relation schemas, each defining a table name, a set of attributes (columns), and a set of constraints (primary key, foreign keys, unique, not null, check). The design process involves identifying entities (things about which data is stored), attributes (properties of entities), and relationships (associations between entities). Normalization — a systematic process of decomposing tables — is applied to eliminate data redundancy and update/insert/delete anomalies. The first three normal forms (1NF, 2NF, 3NF) are the standard target for transactional (OLTP) databases: 1NF requires atomic values and no repeating groups; 2NF requires that every non-key column depend on the entire primary key (relevant only for composite keys); 3NF requires that no non-key column depend transitively on another non-key column.

**Beginner-Friendly Explanation**

Designing a database schema is like designing a set of forms for a business. Each form has a title (table name), blank fields (columns), and rules about what can be written in each field (data types and constraints). If you design the forms poorly — for example, putting a customer's address on every order form instead of keeping a separate "Customers" form — you will eventually have inconsistent data (the same customer with two different addresses). Normalization is the process of redesigning the forms so each piece of information is stored exactly once.

### Purposes

- To create a database structure that accurately reflects the real-world domain it models.
- To eliminate data redundancy and update/insert/delete anomalies through normalization.
- To enforce data integrity through primary keys, foreign keys, and other constraints.
- To provide a stable foundation for application queries, reports, and integrations.
- To enable efficient indexing and query optimization strategies.
- To support schema evolution through well-defined relationships and dependencies.

### Sub-Feature 1.1: Normal Forms (1NF, 2NF, 3NF)

#### Definitions

**Core Definition**

Normal forms are a series of progressively stricter rules for organizing tables to eliminate redundancy and data anomalies.

**Technical Definition**

**First Normal Form (1NF)** requires that all column values be atomic (indivisible) and that there be no repeating groups or arrays within a single column. **Second Normal Form (2NF)** requires that the table be in 1NF and that every non-key column depend on the entire primary key, not just part of it. This only applies when the primary key is composite (made up of more than one column). **Third Normal Form (3NF)** requires that the table be in 2NF and that no non-key column depend transitively on another non-key column. In other words, all non-key columns must depend directly on the primary key.

**Beginner-Friendly Explanation**

Think of normal forms as a set of rules for cleaning up a messy spreadsheet. **1NF** says: "Do not put multiple phone numbers in one cell. Each cell should hold one piece of data." **2NF** says: "If a row is identified by two things (e.g., Order ID + Product ID), do not put product name in that table, because product name only depends on Product ID — not on the full combination." **3NF** says: "Do not put ZIP code and city in the same table if city depends on ZIP code — that is a transitive dependency."

#### Purposes

- To eliminate data redundancy by storing each piece of information in exactly one place.
- To prevent update anomalies, where the same data is updated inconsistently across multiple rows.
- To prevent insert anomalies, where data cannot be entered because of incomplete information.
- To prevent delete anomalies, where deleting one piece of data inadvertently destroys another.
- To create a clean, maintainable schema that is easy to understand and extend.

#### Syntax Rules and Structure

**1NF — Atomic Values, No Repeating Groups**

```sql
-- BAD (1NF violation): non-atomic phones column
CREATE TABLE customers_bad (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    phones VARCHAR(500)  -- "555-1234,555-5678,555-9012"
);

-- GOOD (1NF compliant): separate table for phones
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);
CREATE TABLE customer_phones (
    phone_id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL REFERENCES customers(customer_id),
    phone_number VARCHAR(20) NOT NULL,
    phone_type VARCHAR(20) CHECK (phone_type IN ('mobile', 'home', 'work'))
);
```

**2NF — No Partial Dependencies**

```sql
-- BAD (2NF violation): product_name and product_price depend only on product_id
CREATE TABLE order_items_bad (
    order_id INT,
    product_id INT,
    product_name VARCHAR(100),   -- Depends only on product_id
    product_price DECIMAL(10,2), -- Depends only on product_id
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

-- GOOD (2NF compliant): product attributes moved to products table
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    product_price DECIMAL(10,2) NOT NULL CHECK (product_price >= 0)
);
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10,2) NOT NULL,  -- Snapshot at order time
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

**3NF — No Transitive Dependencies**

```sql
-- BAD (3NF violation): city and state depend on zip_code, not on address_id
CREATE TABLE addresses_bad (
    address_id INT PRIMARY KEY,
    street VARCHAR(200),
    city VARCHAR(100),   -- Depends on zip_code (transitive)
    state VARCHAR(2),    -- Depends on zip_code (transitive)
    zip_code VARCHAR(10)
);

-- GOOD (3NF compliant): separate zip_codes reference table
CREATE TABLE zip_codes (
    zip_code VARCHAR(10) PRIMARY KEY,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(2) NOT NULL,
    county VARCHAR(100)
);
CREATE TABLE addresses (
    address_id SERIAL PRIMARY KEY,
    street VARCHAR(200) NOT NULL,
    zip_code VARCHAR(10) NOT NULL REFERENCES zip_codes(zip_code)
);
```

**Syntax Rules:**

- 1NF: Every column must contain a single, indivisible value. No arrays, no comma-separated lists, no repeating column groups (e.g., `phone1`, `phone2`, `phone3`).
- 2NF: The table must already be in 1NF. Every non-key column must depend on the entire primary key. If the primary key is a single column, the table is automatically in 2NF.
- 3NF: The table must already be in 2NF. No non-key column may depend on another non-key column. All non-key columns must depend directly on the primary key.

**Constraints and Limitations:**

- **Over-normalization:** Splitting tables too aggressively can lead to excessive `JOIN`s and degraded read performance. Many production systems stop at 3NF and selectively denormalize for reporting.
- **Denormalization trade-offs:** Deliberate denormalization (e.g., storing a snapshot of `unit_price` in `order_items`) is often necessary for historical accuracy and performance.
- **BCNF and beyond:** Boyce-Codd Normal Form (BCNF) is stricter than 3NF and eliminates certain edge-case anomalies. Most practical designs stop at 3NF or BCNF.

---

### Sub-Feature 1.2: Primary, Composite, and Foreign Keys

#### Definitions

**Core Definition**

Keys are columns (or sets of columns) that uniquely identify rows and establish relationships between tables.

**Technical Definition**

A **primary key** is a column or set of columns that uniquely identifies each row in a table. It must be unique and not null. A **composite key** is a primary key made up of two or more columns. A **foreign key** is a column (or set of columns) in one table that references the primary key of another table, establishing a referential constraint. A **natural key** is a key with business meaning (e.g., `country_code`); a **surrogate key** is a system-generated identifier with no business meaning (e.g., auto-incrementing `id`). A **candidate key** is any column or set of columns that could serve as a primary key.

**Beginner-Friendly Explanation**

Think of a library. Each book has a unique call number (primary key). If a book has multiple authors, the book's identity might be a combination of "Author + Title" (composite key). When a book is checked out, the checkout record references the book's call number (foreign key) to link the two.

#### Purposes

- To uniquely identify every row in a table without ambiguity.
- To establish and enforce relationships between tables through foreign key constraints.
- To prevent duplicate records through uniqueness guarantees.
- To provide a stable reference for indexes and query optimization.
- To enable cascading operations (`ON DELETE CASCADE`, `ON UPDATE CASCADE`) that maintain referential integrity automatically.

#### Syntax Rules and Structure

**Primary Key (Surrogate)**

```sql
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,  -- Auto-incrementing surrogate key
    email VARCHAR(255) NOT NULL UNIQUE,  -- Natural candidate key
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Composite Primary Key**

```sql
CREATE TABLE student_courses (
    student_id INT,
    course_id INT,
    enrollment_date DATE NOT NULL,
    grade CHAR(2),
    PRIMARY KEY (student_id, course_id),  -- Composite key
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

**Foreign Key with Cascading Actions**

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE DEFAULT CURRENT_DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE CASCADE  -- Delete orders when customer is deleted
        ON UPDATE CASCADE  -- Update orders when customer_id changes
);
```

**Component Breakdown:**

- `SERIAL PRIMARY KEY` / `AUTO_INCREMENT` — Auto-generating surrogate key. PostgreSQL uses `SERIAL` or `GENERATED ALWAYS AS IDENTITY`; MySQL uses `AUTO_INCREMENT`.
- `PRIMARY KEY (col1, col2)` — Composite key definition. Both columns together must be unique.
- `FOREIGN KEY (col) REFERENCES other_table(other_col)` — Establishes a referential constraint. The referenced column must be a primary key or unique key.
- `ON DELETE CASCADE` — When the parent row is deleted, child rows are automatically deleted.
- `ON UPDATE CASCADE` — When the parent key is updated, child foreign keys are automatically updated.

**Syntax Rules:**

- A table can have at most one primary key, but that key can consist of multiple columns (composite).
- Foreign key columns must have the same data type as the referenced column.
- `NULL` values are allowed in foreign key columns (meaning "no relationship"), but not in primary key columns.
- Indexes are automatically created on primary keys and unique constraints.

**Constraints and Limitations:**

- **Natural vs. surrogate keys:** Natural keys (e.g., email addresses) can change over time, causing cascading updates. Surrogate keys (auto-incrementing IDs) are stable but require an extra column.
- **UUIDs in distributed systems:** UUID primary keys avoid sequence conflicts across distributed nodes but are larger (16 bytes vs. 4–8 bytes) and can degrade index performance.
- **Composite key limitations:** Queries must reference all columns of a composite key, which can be cumbersome for application code.

---

## Core Concept 2: Indexes & Performance

### Definitions

**Core Definition**

An index is a data structure that provides fast lookup of rows based on the values of one or more columns, at the cost of additional storage and write overhead.

**Technical Definition**

A B-Tree (Balanced Tree) index is the default index type in MySQL, PostgreSQL, and most relational databases. It maintains a sorted tree structure where each node contains keys and pointers to child nodes. Leaf nodes contain pointers to the actual table rows (or the primary key values in InnoDB). B-Tree indexes support equality lookups (`=`), range queries (`<`, `>`, `BETWEEN`), prefix matching (`LIKE 'abc%'`), and ordered retrieval (`ORDER BY`). A **unique index** enforces uniqueness on the indexed column(s) in addition to providing fast lookup. Primary keys and unique constraints automatically create unique B-Tree indexes.

**Beginner-Friendly Explanation**

An index is like the index at the back of a textbook. Without it, finding a topic means reading every page (a full table scan). With it, you look up the topic, find the page number, and jump directly there. The trade-off: every time you add a new page to the book (INSERT), you must update the index. Every time you change a page (UPDATE), the index may need updating. Every time you remove a page (DELETE), the index entry must be cleaned up. Indexes make reading faster but writing slower.

### Purposes

- To accelerate `SELECT` queries that filter on indexed columns.
- To enforce uniqueness constraints on primary keys and unique columns.
- To speed up `JOIN` operations by providing fast lookups on foreign key columns.
- To support `ORDER BY` and `GROUP BY` without expensive sorting operations.
- To enable efficient range queries (e.g., `WHERE created_at BETWEEN ...`).

### Syntax Rules and Structure

**Creating a Basic B-Tree Index**

```sql
CREATE INDEX idx_users_email ON users (email);
```

**Creating a Unique Index**

```sql
CREATE UNIQUE INDEX idx_users_email_unique ON users (email);
```

**Composite Index**

```sql
CREATE INDEX idx_orders_customer_date ON orders (customer_id, order_date);
```

**Partial Index (PostgreSQL)**

```sql
CREATE INDEX idx_orders_pending ON orders (order_date)
WHERE status = 'pending';
```

**Component Breakdown:**

- `CREATE INDEX idx_name ON table (column)` — Creates a non-unique B-Tree index. The index name is optional in some databases (auto-generated).
- `CREATE UNIQUE INDEX idx_name ON table (column)` — Creates a unique index that enforces uniqueness in addition to providing fast lookup.
- `CREATE INDEX idx_name ON table (col1, col2)` — Creates a composite index. The order of columns matters: the index can be used for queries filtering on `col1`, `col1 AND col2`, but not `col2` alone (in most cases).
- `WHERE condition` — Creates a partial index that only indexes rows matching the condition, reducing index size and write overhead.

**Syntax Rules:**

- Indexes should be created on columns that appear frequently in `WHERE`, `JOIN`, `ORDER BY`, and `GROUP BY` clauses.
- Composite index column order matters: put the most selective (highest cardinality) column first for equality-heavy queries, or the range column last.
- Indexes on foreign key columns are **not** created automatically in MySQL; they must be added explicitly for optimal `JOIN` performance.
- Unique indexes enforce data integrity as well as performance; they prevent duplicate values.

**Constraints and Limitations:**

- **Write overhead:** Every `INSERT`, `UPDATE`, and `DELETE` on a table with indexes requires updating those indexes. A table with 10 indexes requires 10 index updates per insert. The creation of a UNIQUE constraint or PRIMARY KEY results in the creation of a UNIQUE btree index, and this index must be updated whenever any record is inserted, updated, or deleted if any indexed column is changed.
- **Index bloat:** Over time, indexes accumulate dead entries (especially in PostgreSQL) that must be cleaned up by `VACUUM` or `OPTIMIZE TABLE`.
- **Low-cardinality columns:** Indexing a column with very few distinct values (e.g., `gender` with 'M'/'F') provides little benefit; the query optimizer may ignore the index.
- **Unused indexes:** Indexes that are never used by queries still consume storage and slow down writes. Regular index auditing is recommended.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Creating and Using an Index**

```sql
-- Step 1: Create a table without indexes
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    category_id INT NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Step 2: Insert test data (1 million rows, simplified)
-- INSERT INTO products (name, category_id, price) VALUES ...

-- Step 3: Query without index — full table scan
EXPLAIN ANALYZE
SELECT * FROM products WHERE category_id = 5 AND price < 100;

-- Output (without index):
-- Seq Scan on products  (cost=0.00..25000.00 rows=500 width=50)
--   Filter: ((category_id = 5) AND (price < 100))
--   Rows Removed by Filter: 999500
-- Planning Time: 0.100 ms
-- Execution Time: 350.000 ms

-- Step 4: Create a composite index
CREATE INDEX idx_products_category_price ON products (category_id, price);

-- Step 5: Query with index
EXPLAIN ANALYZE
SELECT * FROM products WHERE category_id = 5 AND price < 100;

-- Output (with index):
-- Index Scan using idx_products_category_price on products
--   Index Cond: ((category_id = 5) AND (price < 100))
-- Planning Time: 0.150 ms
-- Execution Time: 2.500 ms
```

**Expected Output:** The query execution time drops from ~350 ms (sequential scan) to ~2.5 ms (index scan), a ~140× improvement.

**Why:** The composite index on `(category_id, price)` allows the database to jump directly to the matching rows instead of scanning the entire table. The index is most effective because `category_id` is the leading column and `price` is a range condition.

**Example 2: Demonstrating Write Overhead**

```sql
-- Step 1: Create a table with no indexes (only primary key)
CREATE TABLE events_no_index (
    event_id SERIAL PRIMARY KEY,
    event_type VARCHAR(50),
    payload JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Step 2: Insert 100,000 rows
-- Timing: ~500 ms

-- Step 3: Create five additional indexes
CREATE INDEX idx_events_type ON events_no_index (event_type);
CREATE INDEX idx_events_created ON events_no_index (created_at);
CREATE INDEX idx_events_type_created ON events_no_index (event_type, created_at);
CREATE INDEX idx_events_payload ON events_no_index USING GIN (payload);
CREATE INDEX idx_events_type_created_payload ON events_no_index (event_type, created_at, payload);

-- Step 4: Insert another 100,000 rows
-- Timing: ~2,800 ms (5.6× slower due to index updates)
```

**Expected Output:** Insert performance degrades significantly as more indexes are added. Each additional index adds a write cost.

**Why:** Every index must be updated on each `INSERT`. With five additional indexes, each row insertion requires six index updates (one per index plus the primary key index). This is the fundamental trade-off of indexing: faster reads, slower writes.

### Real-World Cases

**Case 1: E-Commerce Product Search**

An e-commerce site has a `products` table with millions of rows. Queries frequently filter by `category_id`, `price_range`, and `availability`. A composite index on `(category_id, price, available)` accelerates the most common search patterns. A separate partial index on `(created_at) WHERE available = true` accelerates "new arrivals" queries.

**Case 2: User Authentication**

A `users` table is queried by `email` on every login. Without an index, each login requires a full table scan. A unique index on `email` reduces login query time from ~200 ms to <1 ms while enforcing email uniqueness.

**Case 3: High-Write Logging System**

A logging system writes thousands of events per second. Indexing every column would make writes prohibitively slow. The team indexes only `created_at` (for time-range queries) and uses a partial index on `(severity) WHERE severity IN ('ERROR', 'CRITICAL')` for alerting queries. This minimises write overhead while supporting the most critical read patterns.

---

## Core Concept 3: Database Migrations & Version Control

### Definitions

**Core Definition**

Database migrations are version-controlled, programmatic scripts that describe how to transform a database schema from one version to the next.

**Technical Definition**

A migration is a class or file containing an `up()` method (which applies the schema change) and a `down()` method (which reverts it). A migration runner tracks which migrations have been applied in a dedicated version table (e.g., `migrations_versions`). Migrations are typically named with a timestamp prefix (e.g., `Migration_20240101000000.php`) to ensure sequential ordering. The runner discovers migration files on disk, compares them against the version table, and executes any unapplied migrations in order. This provides a reproducible, auditable way to manage schema changes across development, staging, and production environments.

**Beginner-Friendly Explanation**

Imagine you are building a house (the database schema). Instead of making changes haphazardly — "add a window here, move the door there" — you write down every change on a numbered card. Card #1: "Add a window to the north wall." Card #2: "Move the door to the east wall." When you hire a new builder (deploy to a new environment), you hand them the stack of cards, and they apply them in order. If you make a mistake, you can reverse the last card (`down()`). Migrations are those numbered cards for your database.

### Purposes

- To version-control database schema changes alongside application code in the same repository.
- To ensure that every environment (development, staging, production) has an identical schema.
- To provide a reversible record of every schema change, enabling rollback when a deployment fails.
- To automate schema deployment as part of a CI/CD pipeline.
- To eliminate manual SQL execution, which is error-prone and undocumented.
- To enable new team members to set up a complete database with a single command.

### Syntax Rules and Structure

**Migration Class Structure (InitPHP/Barbarian)**

```php
<?php
declare(strict_types=1);

namespace App\Migrations;

use InitPHP\Barbarian\MigrationAbstract;
use InitPHP\Barbarian\QueryInterface;

final class Migration_20240101000000 extends MigrationAbstract
{
    public function up(QueryInterface $query): bool
    {
        $query->query('
            CREATE TABLE IF NOT EXISTS `users` (
                `id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
                `name` VARCHAR(255) NOT NULL,
                `email` VARCHAR(255) NOT NULL,
                PRIMARY KEY (`id`),
                UNIQUE KEY `idx_users_email` (`email`)
            ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
        ');
        return true;
    }

    public function down(QueryInterface $query): bool
    {
        $query->query('DROP TABLE IF EXISTS `users`');
        return true;
    }
}
```

**Component Breakdown:**

- `Migration_20240101000000` — The class name includes a timestamp (YYYYMMDDHHMMSS) to ensure chronological ordering.
- `up(QueryInterface $query): bool` — The method that applies the schema change. Returns `true` on success, `false` (or throws) to signal failure and prevent the version table from being updated.
- `down(QueryInterface $query): bool` — The method that reverts the schema change. Should undo exactly what `up()` did.
- `$query->query('...')` — Executes a raw SQL statement using the PDO connection provided by the migration runner.

**Programmatic Usage**

```php
<?php
require 'vendor/autoload.php';

use InitPHP\Barbarian\Migrations;

$pdo = new PDO('mysql:host=localhost;dbname=test;charset=utf8mb4', 'root', '');

$migrations = new Migrations($pdo, __DIR__ . '/migrations', [
    'migrationTable' => 'migrations_versions',
    'namespace'      => 'App\\Migrations',
]);

// Apply all pending migrations
foreach ($migrations->getMigrations() as $version => $class) {
    $migrations->upMigration(new $class());
}

echo "All migrations applied.\n";
```

**Syntax Rules:**

- Migration files must follow the naming convention expected by the runner (typically `Migration_<timestamp>.php`).
- The `up()` method should be idempotent where possible (e.g., `CREATE TABLE IF NOT EXISTS`).
- The `down()` method should fully reverse the `up()` method. If a migration adds a column, `down()` should drop it.
- Migrations must be applied in chronological order; the runner enforces this via the version table.

**Constraints and Limitations:**

- **Irreversible migrations:** Some schema changes cannot be safely reversed (e.g., dropping a column that contained data). These migrations should have a `down()` method that throws an exception or logs a warning.
- **Data migrations vs. schema migrations:** Migrations that transform data (e.g., splitting a `name` column into `first_name` and `last_name`) require careful `down()` logic and may not be fully reversible.
- **Concurrent migration execution:** In multi-server environments, care must be taken to prevent two processes from running migrations simultaneously (e.g., using a lock table or running migrations as a single deployment step).
- **DDL implicit commits:** In MySQL, DDL statements (`CREATE TABLE`, `ALTER TABLE`) cause implicit commits, which cannot be rolled back. This means a failed migration that has already executed a DDL statement may leave the schema in a partially migrated state.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Creating and Running a Migration**

```php
<?php
// File: migrations/Migration_20240101000000.php
declare(strict_types=1);

namespace App\Migrations;

use InitPHP\Barbarian\MigrationAbstract;
use InitPHP\Barbarian\QueryInterface;

final class Migration_20240101000000 extends MigrationAbstract
{
    public function up(QueryInterface $query): bool
    {
        // Step 1: Create the users table
        $query->query('
            CREATE TABLE IF NOT EXISTS `users` (
                `id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
                `name` VARCHAR(255) NOT NULL,
                `email` VARCHAR(255) NOT NULL,
                `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                PRIMARY KEY (`id`),
                UNIQUE KEY `idx_users_email` (`email`)
            ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
        ');
        return true;
    }

    public function down(QueryInterface $query): bool
    {
        // Step 2: Revert — drop the users table
        $query->query('DROP TABLE IF EXISTS `users`');
        return true;
    }
}
```

```php
<?php
// File: migrate.php — programmatic migration runner
require 'vendor/autoload.php';

use InitPHP\Barbarian\Migrations;

$pdo = new PDO('mysql:host=localhost;dbname=test;charset=utf8mb4', 'root', '');

$migrations = new Migrations($pdo, __DIR__ . '/migrations', [
    'migrationTable' => 'migrations_versions',
    'namespace'      => 'App\\Migrations',
]);

foreach ($migrations->getMigrations() as $version => $class) {
    $migrations->upMigration(new $class());
    echo "Applied migration: $version\n";
}

echo "Migrations complete.\n";
```

**Expected Output:**

```
Applied migration: 20240101000000
Migrations complete.
```

**Why:** The migration runner discovers `Migration_20240101000000.php`, instantiates the class, calls `up()`, and records the version in the `migrations_versions` table. On subsequent runs, the migration is skipped because it is already recorded.

**Example 2: Rolling Back a Migration**

```php
<?php
require 'vendor/autoload.php';

use InitPHP\Barbarian\Migrations;

$pdo = new PDO('mysql:host=localhost;dbname=test;charset=utf8mb4', 'root', '');

$migrations = new Migrations($pdo, __DIR__ . '/migrations', [
    'migrationTable' => 'migrations_versions',
    'namespace'      => 'App\\Migrations',
]);

// Roll back a specific migration
$migrations->downMigration(new \App\Migrations\Migration_20240101000000());
echo "Migration 20240101000000 rolled back.\n";
```

**Expected Output:**

```
Migration 20240101000000 rolled back.
```

**Why:** The `downMigration()` method calls the `down()` method of the migration class, which drops the `users` table. The version table is updated to reflect that this migration is no longer applied.

### Real-World Cases

**Case 1: Laravel Migrations**

Laravel's migration system is the most widely used PHP migration tool. Developers run `php artisan make:migration create_users_table` to generate a migration file, edit the `up()` and `down()` methods, and run `php artisan migrate` to apply pending migrations. The `migrations` table tracks which migrations have been applied.

**Case 2: Doctrine Migrations**

Doctrine Migrations provides a framework-agnostic migration system for Doctrine ORM projects. It generates migration classes by comparing the current database schema with the entity mapping metadata, producing `up()` and `down()` methods that transform the schema.

**Case 3: InitPHP/Barbarian**

Barbarian is a small, dependency-light PHP migration library with a PDO-based runner. It discovers migration classes on disk, keeps a version table in sync, and runs the `up()`/`down()` methods either programmatically or through a CLI. It supports MySQL/MariaDB, SQLite, and PostgreSQL.

---

## Core Concept 4: SQL Fundamentals

### Definitions

**Core Definition**

SQL (Structured Query Language) is the standard language for interacting with relational databases, used to create, read, update, and delete data (CRUD), and to define and manipulate database structures.

**Technical Definition**

SQL is divided into several sublanguages: **Data Manipulation Language (DML)** — `SELECT`, `INSERT`, `UPDATE`, `DELETE` — which operates on data; **Data Definition Language (DDL)** — `CREATE`, `ALTER`, `DROP` — which defines schema; **Data Control Language (DCL)** — `GRANT`, `REVOKE` — which manages permissions; and **Transaction Control Statements (TCS)** — `COMMIT`, `ROLLBACK`, `SAVEPOINT` — which manages transactions. DML statements are the most frequently used in application development. `SELECT` retrieves data; `INSERT` adds rows; `UPDATE` modifies rows; `DELETE` removes rows. Aggregate functions (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) compute summary values across sets of rows, often combined with `GROUP BY` to produce grouped summaries. `JOIN` clauses combine rows from two or more tables based on a related column.

**Beginner-Friendly Explanation**

SQL is the language you use to talk to a database. You say "give me all rows where the price is less than 10" (`SELECT ... WHERE price < 10`), "add a new row with these values" (`INSERT`), "change the price of this row" (`UPDATE`), or "delete this row" (`DELETE`). Aggregates let you ask "how many rows?" (`COUNT`), "what is the total?" (`SUM`), or "what is the average?" (`AVG`). Joins let you combine data from multiple tables — like matching a customer to their orders.

### Purposes

- To retrieve specific data using `SELECT` with `WHERE`, `ORDER BY`, and `LIMIT`.
- To insert new records into tables using `INSERT`.
- To modify existing records using `UPDATE`.
- To remove records using `DELETE`.
- To compute summary statistics using aggregate functions and `GROUP BY`.
- To combine data from multiple tables using `INNER JOIN`, `LEFT JOIN`, and `RIGHT JOIN`.
- To define and modify database structures using DDL statements.

### Sub-Feature 4.1: Core DML — SELECT, INSERT, UPDATE, DELETE

#### Syntax Rules and Structure

**SELECT**

```sql
SELECT column1, column2, ...
FROM table_name
WHERE condition
ORDER BY column1 ASC, column2 DESC
LIMIT n OFFSET m;
```

**INSERT**

```sql
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);
```

**UPDATE**

```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

**DELETE**

```sql
DELETE FROM table_name
WHERE condition;
```

**Component Breakdown:**

- `SELECT column1, column2` — Specifies which columns to return. Use `*` for all columns (discouraged in production for performance reasons).
- `FROM table_name` — The source table.
- `WHERE condition` — Filters rows. Uses comparison operators (`=`, `<`, `>`, `<=`, `>=`, `<>`, `!=`), `LIKE` for pattern matching, `IN` for sets, `BETWEEN` for ranges, and `IS NULL` / `IS NOT NULL` for null checks.
- `ORDER BY` — Sorts the result set. `ASC` (default) or `DESC`.
- `LIMIT n OFFSET m` — Restricts the number of rows returned and skips `m` rows.
- `INSERT INTO ... VALUES` — Adds a new row. Columns not listed receive their default values (or `NULL`).
- `UPDATE ... SET ... WHERE` — Modifies existing rows. **Omitting `WHERE` updates every row.**
- `DELETE FROM ... WHERE` — Removes rows. **Omitting `WHERE` deletes every row.**

**Constraints and Limitations:**

- **Missing `WHERE` clause:** The most catastrophic mistake in SQL. Always verify the `WHERE` clause before executing an `UPDATE` or `DELETE`.
- **`SELECT *` performance:** Retrieving all columns when only a few are needed wastes memory and bandwidth, especially on wide tables.
- **Implicit type conversion:** Comparing a string to an integer (e.g., `WHERE id = '1'`) can prevent index usage. Use explicit types.

---

### Sub-Feature 4.2: Aggregate Functions and GROUP BY

#### Definitions

**Core Definition**

Aggregate functions compute a single summary value from a set of rows. `GROUP BY` partitions the result set into groups, and aggregates are computed per group.

**Technical Definition**

Aggregate functions include `COUNT()` (number of rows), `SUM()` (sum of values), `AVG()` (average), `MIN()` (minimum), and `MAX()` (maximum). `GROUP BY` collapses rows with identical values in the specified columns into a single row, and aggregate functions operate on each group. `HAVING` filters groups after aggregation (unlike `WHERE`, which filters rows before aggregation).

**Syntax Rules and Structure**

```sql
SELECT column1, aggregate_function(column2)
FROM table_name
WHERE condition
GROUP BY column1
HAVING aggregate_condition
ORDER BY aggregate_function(column2) DESC;
```

**Component Breakdown:**

- `GROUP BY column1` — Partitions rows into groups based on the values in `column1`.
- `aggregate_function(column2)` — Computes a summary value for each group.
- `HAVING` — Filters groups based on aggregate results (e.g., `HAVING COUNT(*) > 5`). `WHERE` cannot be used with aggregate functions.
- `ORDER BY aggregate_function(column2)` — Sorts the grouped results.

**Syntax Rules:**

- Every column in the `SELECT` list that is not an aggregate function must appear in the `GROUP BY` clause.
- `HAVING` is evaluated after `GROUP BY`; `WHERE` is evaluated before.
- Aggregate functions ignore `NULL` values (except `COUNT(*)`, which counts all rows).

**Constraints and Limitations:**

- **`COUNT(*)` vs. `COUNT(column)`:** `COUNT(*)` counts all rows, including those with `NULL` values. `COUNT(column)` counts only non-`NULL` values in that column.
- **`GROUP BY` with large result sets:** Grouping millions of rows can be memory-intensive; indexes on the grouping columns help.

---

### Sub-Feature 4.3: JOINs — INNER, LEFT, RIGHT

#### Definitions

**Core Definition**

A `JOIN` combines rows from two or more tables based on a related column, producing a single result set.

**Technical Definition**

**INNER JOIN** returns only rows where the join condition matches in both tables. **LEFT JOIN** (or `LEFT OUTER JOIN`) returns all rows from the left table, plus matching rows from the right table (or `NULL` if no match). **RIGHT JOIN** (or `RIGHT OUTER JOIN`) returns all rows from the right table, plus matching rows from the left table (or `NULL` if no match). **FULL OUTER JOIN** returns all rows from both tables, with `NULL` where there is no match (not supported in MySQL, but can be emulated with `UNION`).

**Syntax Rules and Structure**

**INNER JOIN**

```sql
SELECT Orders.OrderID, Customers.CustomerName
FROM Orders
INNER JOIN Customers ON Orders.CustomerID = Customers.CustomerID;
```

**LEFT JOIN**

```sql
SELECT Customers.CustomerName, Orders.OrderID
FROM Customers
LEFT JOIN Orders ON Customers.CustomerID = Orders.CustomerID
ORDER BY Customers.CustomerName;
```

**RIGHT JOIN**

```sql
SELECT Orders.OrderID, Employees.LastName, Employees.FirstName
FROM Orders
RIGHT JOIN Employees ON Orders.EmployeeID = Employees.EmployeeID
ORDER BY Orders.OrderID;
```

**Component Breakdown:**

- `FROM table1` — The left table.
- `INNER JOIN table2 ON condition` — Returns rows where the condition matches in both tables.
- `LEFT JOIN table2 ON condition` — Returns all rows from `table1`, with matching rows from `table2` or `NULL`.
- `RIGHT JOIN table2 ON condition` — Returns all rows from `table2`, with matching rows from `table1` or `NULL`.
- `ON condition` — The join condition, typically `table1.column = table2.column`.

**Syntax Rules:**

- The `ON` clause specifies the join condition. It must reference columns from both tables.
- `INNER JOIN` is the default join type; `JOIN` alone is equivalent to `INNER JOIN`.
- `LEFT JOIN` and `RIGHT JOIN` are not commutative: `A LEFT JOIN B` is not the same as `B LEFT JOIN A`.
- `RIGHT JOIN` can always be rewritten as `LEFT JOIN` by swapping the table order, which is generally preferred for readability.

**Constraints and Limitations:**

- **JOIN explosion:** Joining tables with one-to-many relationships can produce a large number of rows (row explosion). Use `DISTINCT` or subqueries to control.
- **Missing indexes on join columns:** `JOIN` performance depends heavily on indexes on the join columns. Without indexes, the database may resort to nested loop joins or hash joins, which are slower.
- **`NULL` values in join columns:** `NULL` never equals `NULL` in SQL, so rows with `NULL` in the join column will not match in an `INNER JOIN`. Use `LEFT JOIN` or `IS NULL` checks to handle them.

---

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Aggregate Functions with GROUP BY**

```sql
-- Step 1: Create a sales table
CREATE TABLE sales (
    sale_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    amount DECIMAL(10,2),
    sale_date DATE
);

-- Step 2: Insert sample data
INSERT INTO sales (product_name, category, amount, sale_date) VALUES
('Widget', 'Hardware', 100.00, '2024-01-15'),
('Gadget', 'Hardware', 200.00, '2024-01-15'),
('Gizmo', 'Software', 150.00, '2024-01-16'),
('Widget', 'Hardware', 120.00, '2024-01-16'),
('Gizmo', 'Software', 130.00, '2024-01-17');

-- Step 3: Aggregate by category
SELECT
    category,
    COUNT(*) AS total_sales,
    SUM(amount) AS total_revenue,
    AVG(amount) AS average_sale,
    MIN(amount) AS smallest_sale,
    MAX(amount) AS largest_sale
FROM sales
GROUP BY category
ORDER BY total_revenue DESC;
```

**Expected Output:**

```
 category | total_sales | total_revenue | average_sale | smallest_sale | largest_sale
----------+-------------+---------------+--------------+---------------+--------------
 Hardware |           3 |        420.00 |       140.00 |        100.00 |       200.00
 Software |           2 |        280.00 |       140.00 |        130.00 |       150.00
```

**Why:** `GROUP BY category` partitions the five sales rows into two groups (Hardware, Software). The aggregate functions compute summary values per group. `ORDER BY total_revenue DESC` sorts the groups by revenue.

**Example 2: INNER, LEFT, and RIGHT JOIN Compared**

```sql
-- Step 1: Create tables
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    product VARCHAR(100),
    amount DECIMAL(10,2)
);

-- Step 2: Insert data
INSERT INTO customers (name) VALUES ('Alice'), ('Bob'), ('Carol');
INSERT INTO orders (customer_id, product, amount) VALUES
(1, 'Widget', 50.00),
(1, 'Gadget', 75.00),
(2, 'Gizmo', 100.00);
-- Carol has no orders. Order 3 has no customer (NULL).

-- Step 3: INNER JOIN — only rows with matches in both tables
SELECT c.name, o.product, o.amount
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
ORDER BY c.name;

-- Step 4: LEFT JOIN — all customers, even those with no orders
SELECT c.name, o.product, o.amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
ORDER BY c.name;

-- Step 5: RIGHT JOIN — all orders, even those with no customer
SELECT c.name, o.product, o.amount
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id
ORDER BY o.order_id;
```

**Expected Output:**

```
-- INNER JOIN:
 name  | product | amount
-------+---------+--------
 Alice | Widget  |  50.00
 Alice | Gadget  |  75.00
 Bob   | Gizmo   | 100.00

-- LEFT JOIN:
 name  | product | amount
-------+---------+--------
 Alice | Widget  |  50.00
 Alice | Gadget  |  75.00
 Bob   | Gizmo   | 100.00
 Carol | NULL    |   NULL

-- RIGHT JOIN:
 name  | product | amount
-------+---------+--------
 Alice | Widget  |  50.00
 Alice | Gadget  |  75.00
 Bob   | Gizmo   | 100.00
 NULL  | NULL    |   NULL
```

**Why:** `INNER JOIN` returns only matching rows (Alice and Bob have orders). `LEFT JOIN` includes Carol (no orders), filling `product` and `amount` with `NULL`. `RIGHT JOIN` includes the orphaned order (order_id 3 with no customer), filling `name` with `NULL`. Note that in the `RIGHT JOIN`, the orphaned order has `product = NULL` and `amount = NULL` because the sample data did not include an order with `customer_id = NULL` — the `NULL` row represents an order that exists but has no matching customer.

### Real-World Cases

**Case 1: Dashboard Sales Report**

A dashboard displays total revenue, order count, and average order value per product category. The query uses `GROUP BY category` with `SUM(amount)`, `COUNT(*)`, and `AVG(amount)`. An index on `(category, sale_date)` accelerates the query.

**Case 2: Customer Order History**

A customer support tool displays a customer's order history. An `INNER JOIN` between `customers` and `orders` returns only customers with orders. A `LEFT JOIN` is used on the customer list page to show all customers, including those who have never placed an order.

**Case 3: Finding Orphaned Records**

A data-quality check uses `LEFT JOIN` to find orders with no matching customer: `SELECT o.* FROM orders o LEFT JOIN customers c ON o.customer_id = c.customer_id WHERE c.customer_id IS NULL`. This identifies referential integrity violations.

---

## References

- Cornell Virtual Workshop: Database Normalization — https://cvw.cac.cornell.edu/RelationalDBs/design-create/database_normalization
- DigitalOcean: Database Normalization — 1NF, 2NF, 3NF & BCNF Examples — https://www.digitalocean.com/community/tutorials/database-normalization
- Jeffallan/claude-skills: Database Design Reference — https://github.com/jeffallan/claude-skills/blob/main/skills/sql-pro/references/database-design.md
- Stack Overflow: How does unique constraint affect write performance — https://stackoverflow.com/revisions/1fbb453f-89f7-4c46-9429-6c903148bb40/view-source
- W3Schools: SQL JOIN Keyword — https://www.w3schools.com/mySQl/sql_ref_join.asp
- Educative: Essential SQL Commands for Database Management — https://www.educative.io/blog/essential-sql-commands-database-management
- InitPHP/Barbarian: PHP Migration Library — https://github.com/InitPHP/Barbarian
- Packagist: initphp/barbarian — https://packagist.org/packages/initphp/barbarian
- Packagist: fabiopaiva/pdo-simple-migration — https://root.packagist.org/packages/fabiopaiva/pdo-simple-migration
- Packagist: luxbet/mysql-php-migrations — https://packagist.org/packages/luxbet/mysql-php-migrations
- Cycle ORM: Database Migrations — https://cycle-orm.dev
- Kanboard Plugin Schema Migrations — https://docs.kanboard.org