# Relational Databases — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Relational databases are structured data stores that organise information into tables with rows and columns, using SQL (Structured Query Language) to define, query, and manipulate data, and enforcing relationships between tables through keys and constraints.

**Technical Definition:** A relational database management system (RDBMS) implements the relational model proposed by E.F. Codd in 1970, where data is represented as relations (tables) consisting of tuples (rows) and attributes (columns). PostgreSQL, MySQL, MariaDB, and SQL Server are the four dominant RDBMSs used in the Node.js ecosystem. Each provides ACID transactions, referential integrity, indexing, and SQL compliance, but differs in architecture, feature set, replication capabilities, authentication mechanisms, and Node.js driver maturity.

**Beginner-Friendly Explanation:** A relational database is like a well-organised filing cabinet. Each drawer is a table (users, orders, products). Each folder in the drawer is a row (one user, one order). Each label on the folder is a column (name, email, price). Relationships between drawers are expressed through shared keys — for example, an order's "user_id" points to a user's "id." SQL is the language you use to ask the cabinet questions like "show me all orders from users in London."

### Key Characteristics

- **PostgreSQL:** The most advanced open-source relational database, dominant in the Node.js ecosystem for new projects, with JSONB, RLS, extensions (PostGIS, pgvector), and strong ACID compliance.
- **MySQL:** The most widely deployed open-source RDBMS, known for replication, scalability, and a vast hosting ecosystem.
- **MariaDB:** A community fork of MySQL with open-source governance, drop-in driver compatibility, and performance optimisations.
- **SQL Server:** Microsoft's enterprise-grade RDBMS with Active Directory integration, T-SQL, and strong tooling, accessed from Node.js via the `tedious` driver.
- **Node.js drivers:** `pg` (PostgreSQL), `mysql2` (MySQL/MariaDB), and `tedious`/`mssql` (SQL Server).
- **Connection pooling:** All production deployments use connection pools (e.g., `pg.Pool`, `mysql2` pool) to manage concurrency.
- **Replication:** MySQL, MariaDB, and SQL Server support primary-replica replication; PostgreSQL supports streaming replication and logical replication.

### Prerequisites

- **Node.js runtime:** Node.js 18+ recommended.
- **SQL fundamentals:** SELECT, INSERT, UPDATE, DELETE, JOIN, and WHERE clauses.
- **Database concepts:** Tables, rows, columns, primary keys, foreign keys, and indexes.
- **Node.js async programming:** Promises, async/await, and connection pooling.

### Related Programming Areas

- **NoSQL Databases:** MongoDB, Redis, and Cassandra as alternatives.
- **ORMs and Query Builders:** Prisma, TypeORM, Sequelize, Knex, and Drizzle.
- **Database Migrations:** Schema versioning and deployment strategies.
- **Security:** SQL injection prevention, authentication, and encryption.
- **Performance:** Indexing, query optimisation, and connection pooling.

### Core Concepts

1. **PostgreSQL** — ecosystem dominance, JSONB, and RLS.
2. **MySQL** — architecture, replication, and high-concurrency tuning.
3. **MariaDB** — open-source optimisations and MySQL compatibility.
4. **SQL Server** — enterprise integration and Active Directory authentication.

---

## Core Concept 1: PostgreSQL

### Sub-Feature 1.1: Dominance in the Node Ecosystem and Advanced Feature Support

#### Definitions

**Core Definition:** PostgreSQL is an advanced, open-source object-relational database system known for its extensibility, standards compliance, and feature richness, and it has become the default choice for Node.js applications that require relational data storage.

**Technical Definition:** PostgreSQL (often called "Postgres") is an open-source object-relational database system that has been actively developed for over 35 years. It supports ACID transactions, MVCC (Multi-Version Concurrency Control), foreign keys, joins, views, triggers, and stored procedures. It also supports advanced features such as JSONB, full-text search, window functions, Common Table Expressions (CTEs), and extensibility through extensions like PostGIS (geospatial), pgvector (vector embeddings), and TimescaleDB (time-series). Node.js connects via the `pg` driver.

**Beginner-Friendly Explanation:** PostgreSQL is like a Swiss Army knife among databases. It handles traditional tables and rows perfectly, but it can also store JSON documents, run full-text searches, handle geographic data, and even power AI vector searches — all in the same database. That's why most new Node.js projects choose it.

#### Purposes

- To provide a robust, standards-compliant relational database for Node.js applications.
- To support advanced data types (JSONB, arrays, ranges, geometric types).
- To enable hybrid relational-document patterns in a single database.
- To leverage extensions for specialised use cases (geospatial, vector, time-series).

#### Syntax Rules and Structure

**Node.js connection (pg driver):**
```javascript
const { Pool } = require('pg');

const pool = new Pool({
  host: process.env.PGHOST,
  port: process.env.PGPORT,
  user: process.env.PGUSER,
  password: process.env.PGPASSWORD,
  database: process.env.PGDATABASE,
  max: 20,                   // Max pool connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});
```

**Parameterised query:**
```javascript
const result = await pool.query(
  'SELECT * FROM users WHERE email = $1',
  [email]
);
```

| Node.js Driver | Package | Features |
|----------------|---------|----------|
| `pg` | `npm install pg` | Pure JavaScript, pooling, parameterised queries, streaming. |
| `postgres` | `npm install postgres` | Lightweight, tagged template literals. |
| `slonik` | `npm install slonik` | Type-safe, connection pooling, strict validation. |

**Constraints and Limitations:**
- PostgreSQL is not the fastest for simple read-heavy workloads compared to MySQL in some benchmarks, but its feature set compensates.
- Connection pools must be sized to match PostgreSQL's `max_connections` setting (default 100).
- Large objects (BLOB) should be stored in object storage (S3), not in the database.

#### Annotated Code Example

```javascript
// postgres-basic.js
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000,
});

async function main() {
  // Create table
  await pool.query(`
    CREATE TABLE IF NOT EXISTS users (
      id SERIAL PRIMARY KEY,
      name TEXT NOT NULL,
      email TEXT UNIQUE NOT NULL,
      created_at TIMESTAMPTZ DEFAULT NOW()
    )
  `);

  // Insert with parameterised query
  const insertResult = await pool.query(
    'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING id, name, email',
    ['Alice', 'alice@example.com']
  );
  console.log('Inserted:', insertResult.rows[0]);

  // Select
  const selectResult = await pool.query('SELECT * FROM users WHERE email = $1', ['alice@example.com']);
  console.log('Selected:', selectResult.rows[0]);

  await pool.end();
}

main().catch(console.error);
```

**Expected Output:**
```
Inserted: { id: 1, name: 'Alice', email: 'alice@example.com' }
Selected: { id: 1, name: 'Alice', email: 'alice@example.com', created_at: 2026-01-15T12:00:00.000Z }
```

**Why this output:** The `pg` driver's connection pool manages connections to PostgreSQL. Parameterised queries (`$1`, `$2`) prevent SQL injection. The `RETURNING` clause returns the inserted row.

#### Real-World Cases

- **SaaS applications:** Multi-tenant data with row-level security.
- **E-commerce:** Products, orders, and inventory with ACID transactions.
- **Analytics:** Window functions and CTEs for complex reporting.
- **AI applications:** pgvector for embedding similarity search.

---

### Sub-Feature 1.2: JSONB Data Types and Indexing for Hybrid Relational-Document Patterns

#### Definitions

**Core Definition:** JSONB is a PostgreSQL data type that stores JSON data in a decomposed binary format, enabling indexing and efficient querying of semi-structured data alongside traditional relational columns.

**Technical Definition:** JSONB stores JSON data in a binary format that eliminates whitespace, duplicates keys, and reorders keys. It supports GIN (Generalized Inverted Index) indexes for fast containment queries (`@>`, `?`, `?|`, `?&`), and can be queried using operators and functions like `->`, `->>`, `#>`, `#>>`, `jsonb_path_query`, and `jsonb_each`. JSONB is ideal for hybrid relational-document patterns where some fields are fixed (relational) and others are dynamic (document-like).

**Beginner-Friendly Explanation:** JSONB lets you store flexible JSON data in a PostgreSQL column while still using all the relational features around it. Think of it as a filing cabinet with drawers for structured data and a special compartment for flexible documents. You can index the JSON so queries are fast.

#### Purposes

- To store dynamic or schema-less data alongside relational columns.
- To enable hybrid relational-document patterns without a separate document database.
- To support flexible attributes (e.g., product metadata, user preferences).
- To leverage PostgreSQL's indexing and query capabilities on JSON data.

#### Syntax Rules and Structure

**Creating a JSONB column:**
```sql
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  attributes JSONB NOT NULL DEFAULT '{}'::jsonb
);
```

**Inserting JSONB:**
```sql
INSERT INTO products (name, attributes)
VALUES ('Laptop', '{"brand": "Dell", "ram": 16, "storage": "512GB SSD"}');
```

**Querying JSONB:**
```sql
-- Containment query
SELECT * FROM products WHERE attributes @> '{"brand": "Dell"}';

-- Extract field
SELECT name, attributes->>'brand' AS brand FROM products;

-- Nested extraction
SELECT attributes#>>'{specs,cpu}' AS cpu FROM products;

-- Check key existence
SELECT * FROM products WHERE attributes ? 'ram';
```

**Indexing JSONB:**
```sql
-- GIN index for containment queries
CREATE INDEX idx_products_attributes ON products USING GIN (attributes);

-- Expression index for a specific key
CREATE INDEX idx_products_brand ON products ((attributes->>'brand'));
```

| Operator | Description | Example |
|----------|-------------|---------|
| `->` | Get JSON element (returns JSONB). | `attributes->'brand'` |
| `->>` | Get JSON element as text. | `attributes->>'brand'` |
| `#>` | Get nested element (returns JSONB). | `attributes#>'{specs,cpu}'` |
| `#>>` | Get nested element as text. | `attributes#>>'{specs,cpu}'` |
| `@>` | Contains. | `attributes @> '{"brand":"Dell"}'` |
| `?` | Key exists. | `attributes ? 'ram'` |
| `?|` | Any key exists. | `attributes ?| array['ram','cpu']` |
| `?&` | All keys exist. | `attributes ?& array['ram','cpu']` |

**Constraints and Limitations:**
- JSONB is slower to write than JSON (due to binary conversion) but faster to query.
- GIN indexes are large and slow to update; consider `jsonb_path_ops` for containment-only queries.
- JSONB does not preserve key order or whitespace.
- JSONB is not a replacement for proper relational modelling; use it for genuinely dynamic data.

#### Annotated Code Example

```javascript
// postgres-jsonb.js
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

async function main() {
  await pool.query(`
    CREATE TABLE IF NOT EXISTS products (
      id SERIAL PRIMARY KEY,
      name TEXT NOT NULL,
      attributes JSONB NOT NULL DEFAULT '{}'::jsonb
    )
  `);

  await pool.query('CREATE INDEX IF NOT EXISTS idx_products_attrs ON products USING GIN (attributes)');

  // Insert products with dynamic attributes
  await pool.query(
    'INSERT INTO products (name, attributes) VALUES ($1, $2)',
    ['Laptop', { brand: 'Dell', ram: 16, storage: '512GB SSD' }]
  );
  await pool.query(
    'INSERT INTO products (name, attributes) VALUES ($1, $2)',
    ['Phone', { brand: 'Apple', ram: 8, storage: '256GB' }]
  );

  // Query by containment (uses GIN index)
  const dellProducts = await pool.query(
    "SELECT name, attributes->>'brand' AS brand FROM products WHERE attributes @> $1",
    [{ brand: 'Dell' }]
  );
  console.log('Dell products:', dellProducts.rows);

  // Query by key existence
  const withRam = await pool.query(
    "SELECT name, attributes->>'ram' AS ram FROM products WHERE attributes ? 'ram'"
  );
  console.log('Products with RAM:', withRam.rows);

  await pool.end();
}

main().catch(console.error);
```

**Expected Output:**
```
Dell products: [ { name: 'Laptop', brand: 'Dell' } ]
Products with RAM: [ { name: 'Laptop', ram: '16' }, { name: 'Phone', ram: '8' } ]
```

**Why this output:** The JSONB column stores dynamic attributes. The containment query `attributes @> '{"brand":"Dell"}'` uses the GIN index for fast lookup. The key existence query `attributes ? 'ram'` finds products with a RAM attribute. The `->>` operator extracts values as text.

#### Real-World Cases

- **E-commerce:** Product attributes vary by category (electronics vs. clothing).
- **User preferences:** Flexible settings stored alongside user records.
- **IoT:** Sensor readings with dynamic schemas per device type.
- **Content management:** Articles with custom metadata fields.

---

### Sub-Feature 1.3: Row-Level Security (RLS) Configuration

#### Definitions

**Core Definition:** Row-Level Security (RLS) is a PostgreSQL feature that restricts which rows a user can see or modify based on policies evaluated per query, enabling multi-tenant isolation at the database level.

**Technical Definition:** Row-Level Security (RLS) is a PostgreSQL feature that restricts rows based on policies. Policies are defined with `CREATE POLICY` and attached to a table with `ALTER TABLE ... ENABLE ROW LEVEL SECURITY`. When RLS is enabled, all queries against the table are filtered by the applicable policies. Superusers, table owners, and roles with the `BYPASSRLS` attribute bypass RLS by default. RLS is essential for multi-tenant SaaS applications where tenants share tables but must not see each other's data.

**Beginner-Friendly Explanation:** RLS is like having a personal bodyguard for each row in your database. Even if a user runs `SELECT * FROM orders`, the database automatically filters the results to show only the orders they're allowed to see. It's multi-tenant isolation built into the database itself, so you can't forget to add a `WHERE tenant_id = ?` clause.

#### Purposes

- To enforce multi-tenant data isolation at the database level.
- To prevent application bugs from leaking cross-tenant data.
- To implement compliance requirements (GDPR, HIPAA) for data access.
- To simplify application code by centralising access control in the database.

#### Syntax Rules and Structure

**Enabling RLS:**
```sql
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Force RLS for table owners too
ALTER TABLE orders FORCE ROW LEVEL SECURITY;
```

**Creating policies:**
```sql
-- Allow users to see their own orders
CREATE POLICY user_orders ON orders
  FOR SELECT
  USING (user_id = current_setting('app.current_user_id')::int);

-- Allow users to insert their own orders
CREATE POLICY user_insert_orders ON orders
  FOR INSERT
  WITH CHECK (user_id = current_setting('app.current_user_id')::int);

-- Allow admins to see all orders
CREATE POLICY admin_orders ON orders
  FOR ALL
  TO admin_role
  USING (true);
```

**Setting session context:**
```sql
SET app.current_user_id = '42';
```

| Policy Command | Applies To |
|----------------|-----------|
| `FOR SELECT` | SELECT queries. |
| `FOR INSERT` | INSERT statements. |
| `FOR UPDATE` | UPDATE statements. |
| `FOR DELETE` | DELETE statements. |
| `FOR ALL` | All commands. |

**Constraints and Limitations:**
- RLS adds query overhead; indexes on the policy columns are essential.
- Table owners bypass RLS unless `FORCE ROW LEVEL SECURITY` is set.
- Connection pooling requires careful session context management.
- RLS policies can be complex; test thoroughly.

#### Annotated Code Example

```javascript
// postgres-rls.js
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

async function main() {
  // Setup: enable RLS and create policy
  await pool.query(`
    CREATE TABLE IF NOT EXISTS orders (
      id SERIAL PRIMARY KEY,
      user_id INT NOT NULL,
      amount NUMERIC NOT NULL,
      created_at TIMESTAMPTZ DEFAULT NOW()
    )
  `);

  await pool.query('ALTER TABLE orders ENABLE ROW LEVEL SECURITY');

  await pool.query(`
    DROP POLICY IF EXISTS user_orders ON orders
  `);
  await pool.query(`
    CREATE POLICY user_orders ON orders
      FOR ALL
      USING (user_id = current_setting('app.current_user_id')::int)
      WITH CHECK (user_id = current_setting('app.current_user_id')::int)
  `);

  // Insert data as user 1
  await pool.query("SET app.current_user_id = '1'");
  await pool.query('INSERT INTO orders (user_id, amount) VALUES ($1, $2)', [1, 100]);
  await pool.query("SET app.current_user_id = '2'");
  await pool.query('INSERT INTO orders (user_id, amount) VALUES ($1, $2)', [2, 200]);

  // Query as user 1 — only sees their own orders
  const client = await pool.connect();
  try {
    await client.query("SET app.current_user_id = '1'");
    const result = await client.query('SELECT * FROM orders');
    console.log('User 1 sees:', result.rows);

    await client.query("SET app.current_user_id = '2'");
    const result2 = await client.query('SELECT * FROM orders');
    console.log('User 2 sees:', result2.rows);
  } finally {
    client.release();
  }

  await pool.end();
}

main().catch(console.error);
```

**Expected Output:**
```
User 1 sees: [ { id: 1, user_id: 1, amount: '100', created_at: ... } ]
User 2 sees: [ { id: 2, user_id: 2, amount: '200', created_at: ... } ]
```

**Why this output:** The RLS policy filters rows based on `current_setting('app.current_user_id')`. When user 1 queries, only their orders are returned. When user 2 queries, only their orders are returned. The application never needs to add `WHERE user_id = ?` — the database enforces it.

#### Real-World Cases

- **Multi-tenant SaaS:** Each tenant sees only their own data.
- **Healthcare:** Patients can only see their own records.
- **Financial services:** Account holders see only their own transactions.
- **Government:** Citizens access only their own data.

---

## Core Concept 2: MySQL

### Sub-Feature 2.1: Architecture, Replication, and Widespread Production Scaling

#### Definitions

**Core Definition:** MySQL is a widely deployed open-source relational database known for its replication capabilities, scalability, and extensive hosting ecosystem.

**Technical Definition:** MySQL uses a pluggable storage engine architecture (InnoDB being the default and most common). It supports primary-replica replication (asynchronous by default, semi-synchronous as an option), Group Replication for multi-primary clustering, and MySQL InnoDB Cluster for high availability. MySQL is designed for high read throughput, making it a common choice for web applications that need to scale reads across replicas.

**Beginner-Friendly Explanation:** MySQL is like the workhorse of the database world. It's been around forever, is supported by virtually every hosting provider, and can scale to handle massive web traffic by spreading read queries across multiple copies of the database (replicas).

#### Purposes

- To provide a reliable, widely supported relational database for web applications.
- To scale read-heavy workloads through replication.
- To leverage the extensive MySQL hosting and tooling ecosystem.
- To support high-concurrency workloads with InnoDB.

#### Syntax Rules and Structure

**Node.js connection (mysql2 driver):**
```javascript
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  waitForConnections: true,
  connectionLimit: 20,
  queueLimit: 0,
});
```

**Replication (read/write splitting):**
```javascript
const masterPool = mysql.createPool({ /* write connection */ });
const replicaPool = mysql.createPool({ /* read connection */ });

// Writes go to master
await masterPool.query('INSERT INTO users (name) VALUES (?)', ['Alice']);

// Reads go to replica
const [rows] = await replicaPool.query('SELECT * FROM users');
```

| Replication Type | Description |
|------------------|-------------|
| Asynchronous | Replica lags behind master; default. |
| Semi-synchronous | Master waits for at least one replica to acknowledge. |
| Group Replication | Multi-primary clustering with automatic failover. |
| InnoDB Cluster | MySQL Shell + Group Replication + MySQL Router. |

**Constraints and Limitations:**
- Replication is asynchronous by default; replicas may lag.
- Write scaling is limited to a single primary (unless using Group Replication).
- InnoDB is required for transactions and foreign keys.

#### Annotated Code Example

```javascript
// mysql-replication.js
const mysql = require('mysql2/promise');

// Write pool (master)
const masterPool = mysql.createPool({
  host: process.env.MASTER_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  connectionLimit: 10,
});

// Read pool (replica)
const replicaPool = mysql.createPool({
  host: process.env.REPLICA_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  connectionLimit: 20,
});

async function createUser(name, email) {
  // Write to master
  const [result] = await masterPool.query(
    'INSERT INTO users (name, email) VALUES (?, ?)',
    [name, email]
  );
  return result.insertId;
}

async function getUsers() {
  // Read from replica
  const [rows] = await replicaPool.query('SELECT * FROM users');
  return rows;
}

(async () => {
  const id = await createUser('Alice', 'alice@example.com');
  console.log('Created user ID:', id);

  // Small delay for replication
  await new Promise(r => setTimeout(r, 100));

  const users = await getUsers();
  console.log('Users:', users);
})();
```

**Expected Output:**
```
Created user ID: 1
Users: [ { id: 1, name: 'Alice', email: 'alice@example.com' } ]
```

**Why this output:** Writes go to the master pool; reads go to the replica pool. The small delay accounts for asynchronous replication lag. This pattern scales read throughput horizontally.

#### Real-World Cases

- **E-commerce:** Read-heavy product catalogue served from replicas; writes to master.
- **Social media:** Feed reads from replicas; posts write to master.
- **Analytics:** Reporting queries against replicas to avoid impacting production writes.

---

### Sub-Feature 2.2: High-Concurrency Performance Tuning with Node Drivers

#### Definitions

**Core Definition:** High-concurrency performance tuning in MySQL involves configuring connection pools, query optimisation, and InnoDB settings to handle many simultaneous requests efficiently.

**Technical Definition:** The `mysql2` driver provides a connection pool with `connectionLimit`, `queueLimit`, and `waitForConnections` options. MySQL's `max_connections` server setting limits total connections. For high-concurrency workloads, use small, well-managed pools, enable prepared statements, use indexes, and configure InnoDB's `innodb_buffer_pool_size` to ~70% of available RAM.

**Beginner-Friendly Explanation:** High-concurrency tuning is like managing a busy restaurant kitchen. You don't hire 1,000 chefs (connections); you hire a few excellent ones (pool) and make sure the orders (queries) are efficient.

#### Purposes

- To handle high request volumes without exhausting database connections.
- To minimise query latency through indexing and prepared statements.
- To optimise InnoDB buffer pool usage.
- To prevent connection storms during traffic spikes.

#### Syntax Rules and Structure

```javascript
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  waitForConnections: true,
  connectionLimit: 20,      // Max 20 connections
  queueLimit: 0,             // Unlimited queued requests
  enableKeepAlive: true,
  keepAliveInitialDelay: 0,
  namedPlaceholders: true,
});
```

| Option | Description | Recommended |
|--------|-------------|-------------|
| `connectionLimit` | Max connections in pool. | 10–50 |
| `queueLimit` | Max queued requests (0 = unlimited). | 0 |
| `waitForConnections` | Queue when pool is full. | `true` |
| `enableKeepAlive` | TCP keep-alive. | `true` |

**Constraints and Limitations:**
- `max_connections` (MySQL) limits total connections; pool size must be `pool_size * app_instances <= max_connections`.
- Prepared statements are cached per connection; use them for repeated queries.
- Always use parameterised queries to prevent SQL injection.

#### Annotated Code Example

```javascript
// mysql-tuning.js
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  waitForConnections: true,
  connectionLimit: 20,
  queueLimit: 0,
  enableKeepAlive: true,
  namedPlaceholders: true,
});

async function getUser(email) {
  // Prepared statement with named placeholders
  const [rows] = await pool.execute(
    'SELECT id, name, email FROM users WHERE email = :email',
    { email }
  );
  return rows[0];
}

async function batchInsert(users) {
  const connection = await pool.getConnection();
  try {
    await connection.beginTransaction();
    for (const user of users) {
      await connection.execute(
        'INSERT INTO users (name, email) VALUES (:name, :email)',
        user
      );
    }
    await connection.commit();
  } catch (err) {
    await connection.rollback();
    throw err;
  } finally {
    connection.release();
  }
}

(async () => {
  const user = await getUser('alice@example.com');
  console.log('User:', user);
})();
```

**Expected Output:**
```
User: { id: 1, name: 'Alice', email: 'alice@example.com' }
```

**Why this output:** The pool manages 20 connections. `pool.execute` uses prepared statements (with `:email` named placeholders) for efficiency. The batch insert uses a transaction with `beginTransaction`, `commit`, and `rollback` for atomicity.

#### Real-World Cases

- **High-traffic APIs:** Managing connection pools to handle thousands of requests per second.
- **Batch processing:** Using transactions for bulk inserts.
- **Real-time applications:** Keep-alive connections to reduce handshake overhead.

---

## Core Concept 3: MariaDB

### Sub-Feature 3.1: Open-Source Optimisations and Compatibility with MySQL Drivers

#### Definitions

**Core Definition:** MariaDB is a community-developed, open-source fork of MySQL that maintains binary and driver compatibility while adding performance optimisations and new features.

**Technical Definition:** MariaDB was forked from MySQL in 2009 by the original MySQL developers after Oracle acquired Sun Microsystems. It maintains drop-in compatibility with MySQL at the protocol, SQL, and driver levels. MariaDB includes additional storage engines (Aria, ColumnStore, MyRocks), improved query optimiser, and features like system-versioned tables. The `mysql2` Node.js driver works with MariaDB without modification.

**Beginner-Friendly Explanation:** MariaDB is like MySQL's twin sibling — it looks and behaves almost identically, so your Node.js code works without changes. But it's developed by a community rather than a corporation, and it includes some extra features.

#### Purposes

- To provide a fully open-source alternative to MySQL.
- To leverage MySQL driver compatibility without code changes.
- To benefit from MariaDB-specific optimisations and features.
- To avoid vendor lock-in with Oracle-owned MySQL.

#### Syntax Rules and Structure

```javascript
// MariaDB uses the same mysql2 driver as MySQL
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  connectionLimit: 20,
});
```

| Feature | MariaDB | MySQL |
|---------|---------|-------|
| License | GPL v2 | GPL v2 + proprietary |
| Storage engines | Aria, ColumnStore, MyRocks, InnoDB | InnoDB, MyISAM, etc. |
| System-versioned tables | Yes | No |
| Driver compatibility | Full (mysql2 works) | N/A |
| JSON support | Yes (alias for LONGTEXT) | Yes (native JSON type) |

**Constraints and Limitations:**
- Some MySQL-specific features (e.g., native JSON type, specific replication features) are not available in MariaDB.
- MariaDB's JSON is an alias for LONGTEXT with a check constraint, not a binary type.
- Feature divergence increases over time.

#### Annotated Code Example

```javascript
// mariadb-basic.js
const mysql = require('mysql2/promise');

// Same driver, same API as MySQL
const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  connectionLimit: 20,
});

async function main() {
  // Create table with system versioning (MariaDB-specific)
  await pool.query(`
    CREATE TABLE IF NOT EXISTS users (
      id INT PRIMARY KEY AUTO_INCREMENT,
      name VARCHAR(255) NOT NULL,
      email VARCHAR(255) UNIQUE NOT NULL
    )
  `);

  await pool.query(
    'INSERT INTO users (name, email) VALUES (?, ?)',
    ['Alice', 'alice@example.com']
  );

  const [rows] = await pool.query('SELECT * FROM users');
  console.log('Users:', rows);

  await pool.end();
}

main().catch(console.error);
```

**Expected Output:**
```
Users: [ { id: 1, name: 'Alice', email: 'alice@example.com' } ]
```

**Why this output:** The `mysql2` driver connects to MariaDB identically to MySQL. The SQL syntax is compatible. MariaDB-specific features (like system-versioned tables) can be used with `ALTER TABLE users ADD SYSTEM VERSIONING`.

#### Real-World Cases

- **Open-source projects:** Avoiding Oracle-owned MySQL.
- **Hosting providers:** Many shared hosts offer MariaDB as a MySQL replacement.
- **Government and education:** Preference for community-governed software.

---

## Core Concept 4: SQL Server

### Sub-Feature 4.1: Enterprise-Grade Integration (Tedious Driver Constraints)

#### Definitions

**Core Definition:** SQL Server is Microsoft's enterprise-grade relational database, accessed from Node.js via the `tedious` driver (a pure JavaScript TDS protocol implementation) or the `mssql` wrapper.

**Technical Definition:** The `tedious` driver is a pure JavaScript implementation of the TDS (Tabular Data Stream) protocol used by SQL Server. It supports connection pooling, parameterised queries, transactions, and Windows authentication. However, it has constraints: no built-in connection pool in older versions, callback-based API (though `mssql` provides Promise support), and limited support for some SQL Server-specific data types. The `mssql` package wraps `tedious` with a Promise-based API and connection pooling.

**Beginner-Friendly Explanation:** SQL Server is Microsoft's database, and `tedious` is the Node.js driver that speaks its language. It's not as popular in the Node.js world as PostgreSQL or MySQL because it's more common in .NET environments, and the driver has some rough edges compared to `pg` or `mysql2`.

#### Purposes

- To integrate Node.js applications with existing SQL Server databases.
- To leverage enterprise features (Active Directory, T-SQL, SSRS).
- To support .NET and Node.js in the same data ecosystem.
- To use Windows authentication for database access.

#### Syntax Rules and Structure

**Connection (mssql wrapper):**
```javascript
const sql = require('mssql');

const config = {
  server: process.env.DB_SERVER,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  options: {
    encrypt: true,
    trustServerCertificate: false,
  },
  pool: {
    max: 10,
    min: 0,
    idleTimeoutMillis: 30000,
  },
};

const pool = await sql.connect(config);
```

| Driver | Package | Description |
|--------|---------|-------------|
| `tedious` | `npm install tedious` | Low-level TDS protocol driver. |
| `mssql` | `npm install mssql` | Promise-based wrapper around `tedious`. |

**Constraints and Limitations:**
- `tedious` is callback-based; `mssql` provides Promises.
- Connection pooling is provided by `mssql`, not `tedious` directly.
- Windows authentication requires the `msnodesqlv8` driver (not pure JS).
- SQL Server-specific features (e.g., `OUTPUT` clause) require T-SQL knowledge.

#### Annotated Code Example

```javascript
// sqlserver-basic.js
const sql = require('mssql');

const config = {
  server: process.env.DB_SERVER,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  options: {
    encrypt: true,
    trustServerCertificate: true, // For development only
  },
  pool: { max: 10, min: 0, idleTimeoutMillis: 30000 },
};

async function main() {
  const pool = await sql.connect(config);

  // Parameterised query
  const result = await pool.request()
    .input('email', sql.NVarChar, 'alice@example.com')
    .query('SELECT id, name, email FROM users WHERE email = @email');

  console.log('User:', result.recordset[0]);

  // Insert with OUTPUT clause
  const insertResult = await pool.request()
    .input('name', sql.NVarChar, 'Bob')
    .input('email', sql.NVarChar, 'bob@example.com')
    .query('INSERT INTO users (name, email) OUTPUT INSERTED.id VALUES (@name, @email)');

  console.log('Inserted ID:', insertResult.recordset[0].id);

  await pool.close();
}

main().catch(console.error);
```

**Expected Output:**
```
User: { id: 1, name: 'Alice', email: 'alice@example.com' }
Inserted ID: 2
```

**Why this output:** The `mssql` wrapper provides a Promise-based API. `pool.request()` creates a request, `.input()` binds parameters, and `.query()` executes the SQL. The `OUTPUT INSERTED.id` clause returns the inserted ID.

#### Real-World Cases

- **Enterprise migrations:** Adding Node.js services to existing SQL Server environments.
- **Windows shops:** Organizations heavily invested in Microsoft technology.
- **Legacy integration:** Connecting to older SQL Server databases.

---

### Sub-Feature 4.2: Active Directory/MS Entra Authentication in Node Apps

#### Definitions

**Core Definition:** Active Directory (AD) and Microsoft Entra ID (formerly Azure AD) authentication allows Node.js applications to connect to SQL Server using identity-based authentication instead of SQL usernames and passwords.

**Technical Definition:** Microsoft Entra ID (formerly Azure AD) authentication for SQL Server uses OAuth 2.0 access tokens. The `tedious` driver supports `azure-active-directory-access-token`, `azure-active-directory-password`, `azure-active-directory-service-principal-secret`, and `azure-active-directory-msi-*` authentication types. For on-premises Active Directory, NTLM or Kerberos authentication is used via the `msnodesqlv8` driver.

**Beginner-Friendly Explanation:** Instead of using a database username and password, your Node.js app can authenticate using its Azure identity — like using a company badge instead of a key. This is more secure because there are no passwords to leak, and access can be revoked centrally.

#### Purposes

- To eliminate SQL passwords from configuration.
- To leverage centralised identity management (Entra ID).
- To support managed identities for Azure-hosted applications.
- To enforce conditional access and MFA policies.

#### Syntax Rules and Structure

**Connection with access token:**
```javascript
const sql = require('mssql');
const { DefaultAzureCredential } = require('@azure/identity');

const credential = new DefaultAzureCredential();
const token = await credential.getToken('https://database.windows.net/.default');

const config = {
  server: process.env.DB_SERVER,
  database: process.env.DB_NAME,
  authentication: {
    type: 'azure-active-directory-access-token',
    options: { token: token.token },
  },
  options: { encrypt: true },
};

const pool = await sql.connect(config);
```

| Auth Type | Use Case |
|-----------|----------|
| `azure-active-directory-access-token` | Pre-acquired token. |
| `azure-active-directory-password` | Username + password (not recommended). |
| `azure-active-directory-service-principal-secret` | Service principal with client secret. |
| `azure-active-directory-msi-vm` | Managed identity on Azure VM. |
| `azure-active-directory-msi-app-service` | Managed identity on App Service. |

**Constraints and Limitations:**
- Requires Azure infrastructure (Entra ID, managed identities).
- Token acquisition adds latency; cache tokens until near expiry.
- On-premises AD requires `msnodesqlv8` (not pure JavaScript).

#### Annotated Code Example

```javascript
// sqlserver-entra-auth.js
const sql = require('mssql');
const { DefaultAzureCredential } = require('@azure/identity');

async function connectWithManagedIdentity() {
  const credential = new DefaultAzureCredential();
  const tokenResponse = await credential.getToken('https://database.windows.net/.default');

  const config = {
    server: process.env.DB_SERVER,
    database: process.env.DB_NAME,
    authentication: {
      type: 'azure-active-directory-access-token',
      options: { token: tokenResponse.token },
    },
    options: {
      encrypt: true,
      trustServerCertificate: false,
    },
    pool: { max: 10, min: 0, idleTimeoutMillis: 30000 },
  };

  const pool = await sql.connect(config);
  const result = await pool.request().query('SELECT @@VERSION AS version');
  console.log('Connected to:', result.recordset[0].version);
  await pool.close();
}

connectWithManagedIdentity().catch(console.error);
```

**Expected Output:**
```
Connected to: Microsoft SQL Server 2022 (RTM) ...
```

**Why this output:** `DefaultAzureCredential` automatically picks up the managed identity assigned to the Azure resource (VM, App Service, Container App). The token is passed to `tedious` via the `azure-active-directory-access-token` authentication type. No password is stored in the application.

#### Real-World Cases

- **Azure-hosted Node.js apps:** Using managed identities to connect to Azure SQL.
- **Enterprise SSO:** Centralised access control through Entra ID.
- **Compliance:** Meeting requirements for password-less authentication.

---

## References

- PostgreSQL Documentation — https://www.postgresql.org/docs/
- PostgreSQL — JSON Types — https://www.postgresql.org/docs/current/datatype-json.html
- PostgreSQL — Row Security Policies — https://www.postgresql.org/docs/current/ddl-rowsecurity.html
- node-postgres (pg) — https://node-postgres.com/
- MySQL Documentation — https://dev.mysql.com/doc/
- MySQL — Replication — https://dev.mysql.com/doc/refman/8.0/en/replication.html
- mysql2 — GitHub — https://github.com/sidorares/node-mysql2
- MariaDB Documentation — https://mariadb.com/kb/en/documentation/
- Microsoft SQL Server Documentation — https://learn.microsoft.com/en-us/sql/
- Tedious — GitHub — https://github.com/tediousjs/tedious
- mssql — npm — https://www.npmjs.com/package/mssql
- Microsoft Entra ID — https://learn.microsoft.com/en-us/entra/identity/
- Azure Identity SDK for JavaScript — https://learn.microsoft.com/en-us/javascript/api/overview/azure/identity-readme
- RFC 7807 — Problem Details for HTTP APIs — https://datatracker.ietf.org/doc/html/rfc7807
- OWASP — SQL Injection Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- Building a Simple and Effective Error-Handling System in Node.js — https://tsecurity.de/de/2462968/
- pgvector — GitHub — https://github.com/pgvector/pgvector
- PostGIS — https://postgis.net/
- Heroku — Best Practices in Error Handling — https://www.heroku.com/codeish-podcasts/51-best-practices-in-error-handling