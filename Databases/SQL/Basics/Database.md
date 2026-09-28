# Database Fundamentals

A solid understanding of database fundamentals is essential before working with SQL. This guide covers the core concepts, terminology, and structures that underpin all relational database systems.

---

## 1. Data, Information, and Metadata

These three terms are related but distinct. Confusing them leads to poor database design.

| Term | Definition | Example |
|---|---|---|
| **Data** | Raw, unprocessed facts and figures without context | `42`, `Manila`, `2026-01-15` |
| **Information** | Data that has been processed, organized, and given context and meaning | "The temperature in Manila on 2026-01-15 was 42°C" |
| **Metadata** | Data *about* data — describes structure, origin, format, and constraints | "The `temperature` column stores numeric values in Celsius" |

### Data

Data is the **raw material** of a database. On its own, a single value carries little meaning. Data can be:

- **Structured** — organized into rows and columns (e.g., a customer table)
- **Semi-structured** — partially organized (e.g., JSON, XML)
- **Unstructured** — no predefined model (e.g., images, free text)

### Information

Information is **data in context**. The transformation from data to information happens through:

- **Aggregation** — summing, averaging, counting
- **Filtering** — selecting relevant rows
- **Joining** — combining data from multiple tables
- **Contextualizing** — adding meaning via relationships

**Example:**
- Data: `(John, 2026-01-15, 250.00)`
- Information: "John spent ₱250.00 on 2026-01-15."

### Metadata

Metadata describes the **structure and rules** of the data. It answers questions like:

- What tables exist?
- What columns does each table have?
- What data types are allowed?
- What constraints apply?
- Who created the data and when?

**Sources of metadata:**
- The **data dictionary** (system catalog) — e.g., `information_schema` in PostgreSQL/MySQL
- **Schema definitions** — DDL statements that created the objects
- **Documentation** — ER diagrams, data models

**Example query (PostgreSQL):**
```sql
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'customers';
```

---

## 2. Database Definition

A **database** is an organized collection of structured data stored electronically in a computer system, designed to facilitate:

- **Efficient storage** — compact, durable persistence
- **Fast retrieval** — indexed access to relevant records
- **Data integrity** — enforcing rules and constraints
- **Concurrent access** — multiple users without corruption
- **Security** — controlled access to sensitive data

### Formal Definition

> A database is a self-describing collection of integrated records.

- **Self-describing** — contains metadata about its own structure
- **Integrated** — relationships link records across tables
- **Collection of records** — organized into tables, rows, and columns

### Types of Databases

| Type | Model | Example Systems |
|---|---|---|
| **Relational** | Tables with rows/columns | PostgreSQL, MySQL, Oracle |
| **Document** | JSON-like documents | MongoDB, CouchDB |
| **Key-Value** | Simple key → value pairs | Redis, DynamoDB |
| **Column-Family** | Wide rows, columnar storage | Cassandra, HBase |
| **Graph** | Nodes and edges | Neo4j, Amazon Neptune |
| **Time-Series** | Timestamped data | InfluxDB, TimescaleDB |

This guide focuses on **relational databases**.

---

## 3. Database Management System (DBMS)

A **Database Management System (DBMS)** is the software layer between users/applications and the physical data files. It handles all interactions with the database.

### Core Responsibilities

| Function | Description |
|---|---|
| **Data storage** | Manages files, pages, and disk I/O |
| **Query processing** | Parses, optimizes, and executes SQL |
| **Transaction management** | Ensures ACID properties |
| **Concurrency control** | Handles simultaneous access safely |
| **Recovery** | Restores data after failures |
| **Security** | Authentication, authorization, auditing |
| **Integrity enforcement** | Applies constraints and rules |
| **Metadata management** | Maintains the system catalog |

### DBMS vs. Database

| Term | Meaning |
|---|---|
| **Database** | The data itself |
| **DBMS** | The software that manages the data |
| **Database system** | The combination of database + DBMS + applications + users |

### Examples

- **Relational DBMS (RDBMS):** PostgreSQL, MySQL, Oracle Database, SQL Server
- **NoSQL DBMS:** MongoDB, Cassandra, Redis
- **Embedded DBMS:** SQLite, DuckDB

---

## 4. Relational Database Management System (RDBMS)

An **RDBMS** is a DBMS based on the **relational model** proposed by **E. F. Codd** in 1970. It organizes data into **relations** (tables) and enforces relationships via keys.

### Key Characteristics

- Data stored in **tables (relations)**
- Tables consist of **rows (tuples)** and **columns (attributes)**
- **Relationships** defined via foreign keys
- **SQL** as the standard query language
- **ACID** transaction guarantees
- **Set-based operations** (results are sets of rows)

### ACID Properties

| Property | Meaning |
|---|---|
| **Atomicity** | Transactions are all-or-nothing |
| **Consistency** | Data moves from one valid state to another |
| **Isolation** | Concurrent transactions don't interfere |
| **Durability** | Committed changes survive failures |

### RDBMS vs. Other DBMS

| Aspect | RDBMS | NoSQL |
|---|---|---|
| Schema | Fixed, predefined | Flexible / schema-less |
| Query language | SQL | Varies (MQL, CQL, etc.) |
| Consistency | Strong (ACID) | Often eventual (BASE) |
| Scaling | Vertical (mostly) | Horizontal |
| Best for | Structured, transactional data | Unstructured, high-volume data |

### Popular RDBMS

- PostgreSQL
- MySQL / MariaDB
- Microsoft SQL Server
- Oracle Database
- SQLite
- IBM Db2

---

## 5. Database Schema

A **schema** is the **logical structure** of a database — the blueprint that defines how data is organized.

### What a Schema Defines

- **Tables** and their names
- **Columns** and their data types
- **Constraints** (primary keys, foreign keys, checks)
- **Indexes**
- **Views**
- **Stored procedures and functions**
- **Relationships** between tables

### Schema vs. Data

| Aspect | Schema | Data |
|---|---|---|
| What it is | Structure | Content |
| Changes | Rarely (DDL) | Frequently (DML) |
| Analogy | Blueprint | Building |
| Example | `customers(id, name, email)` | `(1, 'John', 'john@x.com')` |

### Schema Example

```sql
CREATE TABLE customers (
  id        SERIAL PRIMARY KEY,
  name      VARCHAR(100) NOT NULL,
  email     VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

This DDL statement defines the *schema* of the `customers` table.

### Schema Levels

| Level | Description |
|---|---|
| **Conceptual** | High-level model, business-oriented (ER diagrams) |
| **Logical** | Tables, columns, keys, relationships |
| **Physical** | Storage details — files, indexes, partitions |

### Schema in Different Systems

- **PostgreSQL:** A schema is a named namespace within a database (e.g., `public`, `sales`).
- **MySQL:** Schema = database (synonyms).
- **SQL Server:** Similar to PostgreSQL — schema is a namespace within a database.
- **Oracle:** Schema = user account with its own objects.

---

## 6. Database Instance

A **database instance** is the **actual data stored in the database at a given moment in time**.

### Schema vs. Instance

| Concept | Schema | Instance |
|---|---|---|
| Nature | Structure | State |
| Changes | Rarely | Constantly |
| Analogy | Class definition | Object instance |
| Example | `customers(id, name)` | The rows currently in `customers` |

### Two Meanings of "Instance"

The term is ambiguous in database literature:

| Meaning | Description |
|---|---|
| **Data instance** | The current set of data in the database at a point in time |
| **Server instance** | A running DBMS process managing one or more databases |

**Example (SQL Server):**
- A single SQL Server installation can host multiple **instances** (e.g., `MSSQLSERVER`, `SQLEXPRESS`), each managing its own databases.

**Example (Oracle):**
- An **instance** = memory structures + background processes
- A **database** = physical files
- Together they form an Oracle **database system**

### Why It Matters

- Backups capture the **instance** (data), not just the schema
- Migrations replicate the **schema** to new environments
- Snapshots capture the **instance** at a specific moment

---

## 7. Tables

A **table** is the fundamental storage structure in a relational database. It organizes data into **rows** and **columns**.

### Characteristics

- Has a **unique name** within its schema
- Has a **fixed set of columns** (defined by the schema)
- Columns have **names** and **data types**
- Rows are **unordered** unless sorted by a query
- Each row is **unique** (enforced by primary key)

### Anatomy of a Table

```
customers
┌────┬──────────┬──────────────────┬─────────────┐
│ id │   name   │      email       │ created_at  │
├────┼──────────┼──────────────────┼─────────────┤
│  1 │ John     │ john@example.com │ 2026-01-01  │
│  2 │ Maria    │ maria@example.com│ 2026-01-02  │
│  3 │ Ahmed    │ ahmed@example.com│ 2026-01-03  │
└────┴──────────┴──────────────────┴─────────────┘
  ↑        ↑            ↑                ↑
columns (attributes)
```

### Table Terminology

| Term | Meaning |
|---|---|
| **Relation** | Formal relational-model term for a table |
| **Tuple** | Formal term for a row |
| **Attribute** | Formal term for a column |
| **Cardinality** | Number of rows in a table |
| **Degree** | Number of columns in a table |

### Creating a Table

```sql
CREATE TABLE products (
  id          SERIAL PRIMARY KEY,
  name        VARCHAR(100) NOT NULL,
  price       NUMERIC(10,2) NOT NULL CHECK (price >= 0),
  in_stock    BOOLEAN DEFAULT TRUE,
  created_at  TIMESTAMP DEFAULT NOW()
);
```

### Base Tables vs. Other Structures

| Structure | Description |
|---|---|
| **Base table** | Physically stored table |
| **View** | Virtual table based on a query |
| **Materialized view** | Physically stored view result |
| **Temporary table** | Exists for the session/transaction |
| **Foreign table** | Points to external data (FDW) |

---

## 8. Rows and Records

A **row** (also called a **record** or **tuple**) represents a **single entity** in a table.

### Characteristics

- Contains one value for each column
- Values can be **NULL** (unknown/missing) unless constrained otherwise
- Order is not guaranteed unless `ORDER BY` is used
- Identified uniquely by a **primary key**

### Example

```
Row 1: (1, 'John', 'john@example.com', '2026-01-01')
Row 2: (2, 'Maria', 'maria@example.com', '2026-01-02')
```

### Row Terminology

| Term | Origin | Meaning |
|---|---|---|
| **Row** | Informal / SQL | A record in a table |
| **Record** | File systems | Same as row |
| **Tuple** | Relational model / math | Same as row |

### NULL Values

`NULL` means **unknown** or **not applicable** — it is not the same as:

- `0` (a number)
- `''` (empty string)
- `FALSE` (a boolean)

```sql
-- NULL comparisons require IS NULL / IS NOT NULL
SELECT * FROM customers WHERE email IS NULL;

-- This does NOT work as expected:
SELECT * FROM customers WHERE email = NULL;  -- returns nothing
```

### Inserting Rows

```sql
INSERT INTO customers (name, email) VALUES ('John', 'john@example.com');
INSERT INTO customers (name, email) VALUES ('Maria', 'maria@example.com');
```

### Row Identity

Two rows in a table must be **distinguishable** — enforced by the **primary key**. Without a primary key, duplicate rows are possible and often undesirable.

---

## 9. Columns and Attributes

A **column** (also called an **attribute** or **field**) defines a **single property** of the entity represented by the table.

### Characteristics

- Has a **name** unique within the table
- Has a **data type** (INTEGER, VARCHAR, DATE, etc.)
- May have **constraints** (NOT NULL, UNIQUE, CHECK, DEFAULT)
- Defines the **domain** of allowed values

### Common Data Types

| Category | Types |
|---|---|
| **Numeric** | `INTEGER`, `BIGINT`, `DECIMAL`, `NUMERIC`, `REAL`, `DOUBLE PRECISION` |
| **Character** | `CHAR`, `VARCHAR`, `TEXT` |
| **Date/Time** | `DATE`, `TIME`, `TIMESTAMP`, `INTERVAL` |
| **Boolean** | `BOOLEAN` |
| **Binary** | `BYTEA`, `BLOB` |
| **JSON** | `JSON`, `JSONB` |
| **UUID** | `UUID` |
| **Array** | `INTEGER[]`, `TEXT[]` (PostgreSQL) |

### Column Definition

```sql
CREATE TABLE employees (
  id          SERIAL PRIMARY KEY,
  first_name  VARCHAR(50) NOT NULL,
  last_name   VARCHAR(50) NOT NULL,
  hire_date   DATE NOT NULL DEFAULT CURRENT_DATE,
  salary      NUMERIC(10,2) CHECK (salary > 0),
  is_active   BOOLEAN DEFAULT TRUE
);
```

### Column Order

Columns have a **defined order** in the table definition. `SELECT *` returns columns in that order, but relying on it is fragile — always list columns explicitly in production queries.

### Adding and Dropping Columns

```sql
-- Add a column
ALTER TABLE employees ADD COLUMN department_id INTEGER;

-- Drop a column
ALTER TABLE employees DROP COLUMN department_id;

-- Rename a column
ALTER TABLE employees RENAME COLUMN first_name TO given_name;
```

### Attribute Terminology

| Term | Meaning |
|---|---|
| **Column** | Informal / SQL term |
| **Attribute** | Relational model term |
| **Field** | File-system / flat-file term |

---

## 10. Relationships

A **relationship** is an association between two tables, established through **keys**.

### Types of Relationships

| Type | Description | Example |
|---|---|---|
| **One-to-One (1:1)** | One row in A relates to exactly one row in B | `users` ↔ `user_profiles` |
| **One-to-Many (1:N)** | One row in A relates to many rows in B | `customers` → `orders` |
| **Many-to-Many (M:N)** | Many rows in A relate to many in B, via a junction table | `students` ↔ `courses` via `enrollments` |

### Implementing Relationships

**One-to-Many:**
```sql
CREATE TABLE orders (
  id          SERIAL PRIMARY KEY,
  customer_id INTEGER NOT NULL REFERENCES customers(id),
  order_date  DATE NOT NULL
);
```

**Many-to-Many (junction table):**
```sql
CREATE TABLE enrollments (
  student_id INTEGER REFERENCES students(id),
  course_id  INTEGER REFERENCES courses(id),
  grade      CHAR(2),
  PRIMARY KEY (student_id, course_id)
);
```

### Relationship Cardinality and Optionality

- **Cardinality:** 1:1, 1:N, M:N
- **Optionality:** Mandatory (must have a related row) vs. optional (may be NULL)

**Example:** A customer *must* have zero or more orders (optional on the order side); every order *must* belong to exactly one customer (mandatory on the customer side).

---

## 11. Keys

A **key** is a column (or set of columns) that uniquely identifies rows and/or establishes relationships.

### Types of Keys

| Key Type | Purpose |
|---|---|
| **Primary Key (PK)** | Uniquely identifies each row; cannot be NULL |
| **Candidate Key** | Any column(s) that could serve as PK |
| **Alternate Key** | Candidate key not chosen as PK |
| **Composite Key** | PK made of multiple columns |
| **Foreign Key (FK)** | References a PK (or unique key) in another table |
| **Surrogate Key** | System-generated ID with no business meaning |
| **Natural Key** | Real-world identifier (email, SSN, ISBN) |
| **Superkey** | Any set of columns that uniquely identifies a row |

### Primary Key

```sql
CREATE TABLE customers (
  id    SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL
);
```

### Composite Primary Key

```sql
CREATE TABLE order_items (
  order_id   INTEGER REFERENCES orders(id),
  product_id INTEGER REFERENCES products(id),
  quantity   INTEGER NOT NULL,
  PRIMARY KEY (order_id, product_id)
);
```

### Foreign Key

```sql
CREATE TABLE orders (
  id          SERIAL PRIMARY KEY,
  customer_id INTEGER NOT NULL,
  CONSTRAINT fk_customer
    FOREIGN KEY (customer_id) REFERENCES customers(id)
    ON DELETE CASCADE
    ON UPDATE CASCADE
);
```

### Referential Actions

| Action | Behavior on DELETE/UPDATE of parent |
|---|---|
| `CASCADE` | Delete/update child rows too |
| `SET NULL` | Set FK column to NULL |
| `SET DEFAULT` | Set FK column to its default |
| `RESTRICT` | Prevent the operation |
| `NO ACTION` | Defer check (default, similar to RESTRICT) |

### Surrogate vs. Natural Keys

| Aspect | Surrogate | Natural |
|---|---|---|
| Example | `id SERIAL` | `email`, `isbn` |
| Stability | Stable | May change |
| Size | Small | Often larger |
| Meaning | None | Business meaning |
| Recommendation | Preferred for PKs | Use as unique constraints |

---

## 12. Constraints

**Constraints** are rules enforced by the DBMS to maintain **data integrity**.

### Types of Constraints

| Constraint | Purpose |
|---|---|
| **NOT NULL** | Column must have a value |
| **UNIQUE** | No duplicate values in column(s) |
| **PRIMARY KEY** | Unique + NOT NULL; one per table |
| **FOREIGN KEY** | Value must exist in referenced table |
| **CHECK** | Value must satisfy a boolean condition |
| **DEFAULT** | Value used when none supplied |
| **EXCLUDE** | Prevents overlapping ranges (PostgreSQL) |

### Examples

```sql
CREATE TABLE products (
  id           SERIAL PRIMARY KEY,
  name         VARCHAR(100) NOT NULL,
  sku          VARCHAR(50) UNIQUE NOT NULL,
  price        NUMERIC(10,2) NOT NULL CHECK (price >= 0),
  stock        INTEGER DEFAULT 0 CHECK (stock >= 0),
  category_id  INTEGER REFERENCES categories(id),
  created_at   TIMESTAMP DEFAULT NOW()
);
```

### Adding Constraints Later

```sql
ALTER TABLE products
  ADD CONSTRAINT chk_price_positive CHECK (price >= 0);

ALTER TABLE products
  ADD CONSTRAINT uq_sku UNIQUE (sku);
```

### Deferrable Constraints (PostgreSQL, Oracle)

Constraints can be checked at **statement** or **transaction** end:

```sql
ALTER TABLE orders
  ADD CONSTRAINT fk_customer
    FOREIGN KEY (customer_id) REFERENCES customers(id)
    DEFERRABLE INITIALLY DEFERRED;
```

### Why Constraints Matter

- **Prevent invalid data** at the source
- **Document business rules** in the schema
- **Enable the optimizer** to make better plans
- **Reduce application code** for validation
- **Improve data quality** across all consumers

---

## 13. Indexes

An **index** is a data structure that speeds up data retrieval at the cost of additional storage and slower writes.

### How Indexes Work

An index is like a book's index — instead of scanning every page (full table scan), you look up the term and jump to the right page.

### Common Index Types

| Index Type | Use Case |
|---|---|
| **B-tree** | Default; equality and range queries |
| **Hash** | Equality only |
| **GIN** | Full-text, arrays, JSONB (PostgreSQL) |
| **GiST** | Geometric, full-text, ranges |
| **BRIN** | Large, naturally ordered tables |
| **Bitmap** | Low-cardinality columns (Oracle) |
| **Clustered** | Physically orders table rows (SQL Server, InnoDB) |

### Creating Indexes

```sql
-- Basic index
CREATE INDEX idx_customers_email ON customers(email);

-- Unique index
CREATE UNIQUE INDEX idx_customers_email_unique ON customers(email);

-- Composite index
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- Partial index
CREATE INDEX idx_active_users ON users(email) WHERE is_active = TRUE;
```

### When to Index

**Good candidates:**
- Columns used in `WHERE`, `JOIN`, `ORDER BY`, `GROUP BY`
- Foreign key columns
- Columns with high selectivity (many distinct values)

**Poor candidates:**
- Small tables (full scan is faster)
- Columns rarely queried
- Columns frequently updated
- Low-cardinality columns (e.g., boolean) — except with partial indexes

### Trade-offs

| Benefit | Cost |
|---|---|
| Faster `SELECT` | Slower `INSERT`/`UPDATE`/`DELETE` |
| Faster joins | Extra disk space |
| Faster sorting | Maintenance overhead |

### Verifying Index Usage

```sql
-- PostgreSQL
EXPLAIN ANALYZE SELECT * FROM customers WHERE email = 'john@example.com';
```

---

## 14. Views

A **view** is a **virtual table** defined by a stored query. It does not store data itself (unless materialized).

### Why Use Views

- **Simplify complex queries** — encapsulate joins and logic
- **Security** — restrict access to specific columns/rows
- **Consistency** — centralize business logic
- **Abstraction** — decouple applications from schema changes

### Creating a View

```sql
CREATE VIEW active_customers AS
SELECT id, name, email
FROM customers
WHERE is_active = TRUE;
```

### Using a View

```sql
SELECT * FROM active_customers WHERE name LIKE 'J%';
```

### Updatable Views

Simple views (single table, no aggregation) are often **updatable**:

```sql
UPDATE active_customers SET name = 'Johnny' WHERE id = 1;
```

Complex views (joins, aggregates) are usually **read-only** unless rules/triggers are defined.

### Materialized Views

A **materialized view** stores the query result physically:

```sql
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT DATE_TRUNC('month', order_date) AS month,
       SUM(total) AS revenue
FROM orders
GROUP BY 1;

-- Refresh when needed
REFRESH MATERIALIZED VIEW monthly_sales;
```

| Aspect | View | Materialized View |
|---|---|---|
| Storage | None (query only) | Stores results |
| Freshness | Always current | Stale until refreshed |
| Speed | Depends on query | Fast reads |
| Use case | Abstraction, security | Reporting, dashboards |

### Dropping a View

```sql
DROP VIEW active_customers;
DROP MATERIALIZED VIEW monthly_sales;
```

---

## 15. Stored Programs

**Stored programs** are SQL code stored and executed inside the database server. They move logic closer to the data.

### Types of Stored Programs

| Type | Description |
|---|---|
| **Stored Procedure** | Named program that performs actions; may or may not return values |
| **Function** | Returns a value; usable in SQL expressions |
| **Trigger** | Automatically executes in response to table events |
| **Event / Job** | Scheduled execution |
| **Package** | Group of procedures/functions (Oracle, PostgreSQL) |

### Stored Procedures

```sql
CREATE PROCEDURE add_customer(
  p_name  VARCHAR,
  p_email VARCHAR
)
LANGUAGE plpgsql
AS $$
BEGIN
  INSERT INTO customers (name, email) VALUES (p_name, p_email);
END;
$$;

-- Call it
CALL add_customer('John', 'john@example.com');
```

### Functions

```sql
CREATE FUNCTION customer_order_count(p_customer_id INTEGER)
RETURNS INTEGER
LANGUAGE SQL
AS $$
  SELECT COUNT(*) FROM orders WHERE customer_id = p_customer_id;
$$;

-- Use in a query
SELECT name, customer_order_count(id) FROM customers;
```

### Triggers

```sql
CREATE FUNCTION set_updated_at()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
  NEW.updated_at := NOW();
  RETURN NEW;
END;
$$;

CREATE TRIGGER trg_customers_updated
BEFORE UPDATE ON customers
FOR EACH ROW
EXECUTE FUNCTION set_updated_at();
```

### Benefits

- **Performance** — reduces network round trips
- **Security** — grants execute permission without table access
- **Reusability** — shared logic across applications
- **Consistency** — single source of truth for business rules
- **Atomicity** — multiple statements in one transaction

### Drawbacks

- **Portability** — dialects differ significantly
- **Debugging** — harder than application code
- **Version control** — awkward to manage in Git
- **Testing** — requires database access
- **Vendor lock-in** — harder to migrate

### Best Practices

- Keep procedures **focused** and **small**
- Version-control **all** DDL and stored programs
- Prefer **functions** for computations, **procedures** for actions
- Use **triggers sparingly** — they hide logic
- Document **side effects** clearly

---

## Summary Table

| Concept | Definition | Example |
|---|---|---|
| **Data** | Raw facts | `42`, `Manila` |
| **Information** | Processed data with meaning | "Manila temp was 42°C" |
| **Metadata** | Data about data | Column types, constraints |
| **Database** | Organized data collection | The `shop` database |
| **DBMS** | Software managing databases | PostgreSQL |
| **RDBMS** | Relational DBMS | PostgreSQL, MySQL |
| **Schema** | Logical structure | Table definitions |
| **Instance** | Data at a moment | Current rows |
| **Table** | Rows and columns | `customers` |
| **Row / Record** | Single entity | `(1, 'John')` |
| **Column / Attribute** | Single property | `name` |
| **Relationship** | Association between tables | `customers → orders` |
| **Key** | Uniquely identifies rows / links tables | PK, FK |
| **Constraint** | Rule enforcing integrity | `NOT NULL`, `CHECK` |
| **Index** | Speeds up retrieval | `idx_customers_email` |
| **View** | Virtual table | `active_customers` |
| **Stored Program** | DB-resident code | Procedure, function, trigger |

---

## Key Takeaways

1. **Data**, **information**, and **metadata** are distinct — data is raw, information is contextual, metadata describes structure.
2. A **database** is an organized collection of data; a **DBMS** is the software managing it; an **RDBMS** is a DBMS based on the relational model.
3. **Schema** defines structure; **instance** is the data at a point in time.
4. **Tables** hold **rows** (records) and **columns** (attributes).
5. **Relationships** link tables via **keys** — one-to-one, one-to-many, many-to-many.
6. **Keys** identify rows (PK) and link tables (FK); **constraints** enforce integrity.
7. **Indexes** speed up reads at the cost of writes and storage.
8. **Views** are virtual tables for abstraction and security; **materialized views** store results.
9. **Stored programs** (procedures, functions, triggers) move logic into the database — powerful but with trade-offs.
