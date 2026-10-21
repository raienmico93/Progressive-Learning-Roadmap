# SQL Databases in Express.js — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
SQL databases are relational data systems that store information in structured tables with defined columns and rows, enforcing relationships through primary and foreign keys, and queried through Structured Query Language (SQL). In an Express.js application, a SQL database serves as the durable system of record—the authoritative source of truth for users, orders, content, and every other piece of business data.

**Technical Definition**
A SQL database management system (DBMS) implements the relational model: data is organized into **relations (tables)** consisting of **tuples (rows)** and **attributes (columns)**. **Primary keys** uniquely identify rows; **foreign keys** enforce referential integrity across tables. SQL databases are **ACID-compliant**—transactions are atomic, consistent, isolated, and durable. Express.js is database-agnostic: it connects to SQL databases through **native drivers** (direct protocol communication), **query builders** (programmatic SQL construction), or **ORMs** (object-relational mappers). The dominant SQL databases in the Node.js ecosystem are PostgreSQL, MySQL, MariaDB, Microsoft SQL Server, and SQLite.

**Beginner-Friendly Explanation**
A SQL database is a set of linked spreadsheets. Each spreadsheet is a table—customers, orders, products. Each row is one record, each column is a field. A primary key uniquely identifies a row (like a customer ID), and a foreign key links rows across tables (an order’s `customer_id` points to the customer who placed it). This structure prevents impossible states, like an order belonging to a customer who does not exist. Express does not include a database; you choose one and connect to it using a driver or ORM.

---

### Key Characteristics

- **Predefined schema** — Tables and columns are defined before data is inserted; schema changes require migrations.
- **ACID transactions** — Multi-step operations either fully succeed or fully roll back.
- **Referential integrity** — Foreign keys prevent orphaned records and impossible states.
- **Expressive querying** — Joins, aggregations, window functions, and subqueries support complex reporting.
- **Mature ecosystem** — Decades of tooling, monitoring, backup, and replication infrastructure.
- **Express is driver-agnostic** — The framework imposes no database; connection is made through native drivers, query builders, or ORMs.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- A database server or embedded database (PostgreSQL, MySQL, MariaDB, SQL Server, or SQLite).
- A database driver or ORM package appropriate to the chosen database.
- Environment variables configured for database credentials (`dotenv`).
- Basic understanding of SQL syntax and relational data modeling.

---

### Related Programming Areas

- **Data modeling** — Schema design, normalization, and indexing.
- **Query optimization** — Connection pooling, prepared statements, and N+1 query prevention.
- **Transactions** — ACID guarantees, commit and rollback handling.
- **Security** — SQL injection prevention, credential management, and least-privilege database users.
- **Migrations** — Version-controlled schema evolution through tools like Knex, Prisma, or node-pg-migrate.

---

## Core Concept 1: PostgreSQL

### Definitions

**Core Definition**
PostgreSQL is an advanced, open-source object-relational database system known for standards compliance, extensibility, and robustness, and is widely regarded as the default choice for new Express.js applications unless a specific constraint dictates otherwise.

**Technical Definition**
PostgreSQL implements a significant portion of the SQL standard and adds extensions such as JSONB, full-text search, window functions, common table expressions, and native partitioning. In Express.js, it is accessed through `pg` (node-postgres), which provides a connection pool (`Pool`), parameterized queries, and transaction support via `BEGIN`/`COMMIT`/`ROLLBACK`. The recommended production pattern is a `pg.Pool` with a thin repository layer, combined with migrations via `node-pg-migrate` or `dbmate`.

**Beginner-Friendly Explanation**
PostgreSQL is the most capable open-source SQL database. It handles complex queries, JSON data, and strict consistency with equal grace. For most new Express projects, it is the safest default choice.

---

### Purposes

- To provide a standards-compliant, feature-rich relational database for complex applications.
- To support advanced data types (JSONB, arrays, ranges) alongside traditional relational modeling.
- To offer strong ACID guarantees for financial, inventory, and other critical data.
- To serve as the system of record while complementing specialized stores (Redis for caching, Elasticsearch for search).

---

### Syntax Rules and Structure

#### Complete General Syntax: `pg.Pool` Connection and Query Wrapper

```javascript
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,                        // maximum connections in pool
  idleTimeoutMillis: 30_000,      // close idle connections after 30s
  connectionTimeoutMillis: 5_000  // fail if connection takes > 5s
});

pool.on('error', (err) => {
  console.error('Unexpected pool error:', err);
  process.exit(1);
});

async function query(text, params) {
  const client = await pool.connect();
  try {
    const result = await client.query(text, params);
    return result.rows;
  } finally {
    client.release();             // always release
  }
}
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `Pool` | Manages a set of reusable connections |
| `max: 20` | Caps concurrent connections |
| `idleTimeoutMillis` | Closes idle connections to free database resources |
| `pool.connect()` | Acquires a client from the pool |
| `client.release()` | Returns the client to the pool in `finally` |

#### Complete General Syntax: Transaction Wrapper

```javascript
async function withTransaction(fn) {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    const result = await fn(client);
    await client.query('COMMIT');
    return result;
  } catch (err) {
    await client.query('ROLLBACK');
    throw err;
  } finally {
    client.release();
  }
}
```

#### Syntax Rules

- **Always use a pool**, never a new `Client` per request.
- **Always use parameterized queries** (`$1`, `$2`) — never string concatenation.
- **Always release** the client in a `finally` block.
- Use `withTransaction` for any multi-step write that must be atomic.
- Add migrations via `node-pg-migrate` or `dbmate`.

#### Constraints and Limitations

- Pool size must be tuned to the database’s `max_connections` setting; exceeding it causes connection failures.
- JSONB is powerful but not a substitute for proper relational modeling when joins are required.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: PostgreSQL User Lookup with Pooled Connection

```javascript
const express = require('express');
const { Pool } = require('pg');
const app = express();

const pool = new Pool({ connectionString: process.env.DATABASE_URL, max: 20 });

app.get('/users/:id', async (req, res, next) => {
  try {
    const result = await pool.query(
      'SELECT id, email, name FROM users WHERE id = $1',
      [req.params.id]
    );
    if (result.rows.length === 0) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(result.rows[0]);
  } catch (err) {
    next(err);
  }
});

app.listen(3000);
```

**Expected Output**

```
GET /users/1   → 200 { "id": 1, "email": "alice@example.com", "name": "Alice" }
GET /users/99  → 404 { "error": "User not found" }
```

**Why This Output Occurs**

- `pool.query()` acquires a client, runs the parameterized query, and releases the client automatically.

---

### Real-World Cases

- **Financial systems** — ACID transactions and strong consistency for transfers and ledgers.
- **SaaS platforms** — JSONB for flexible metadata alongside relational core data.
- **Geospatial applications** — PostGIS extension for location-aware queries.

---

## Core Concept 2: MySQL

### Definitions

**Core Definition**
MySQL is the world’s most widely deployed open-source relational database, known for speed, reliability, and a vast ecosystem of managed hosting providers and tooling.

**Technical Definition**
MySQL uses InnoDB as its default storage engine, providing ACID transactions, row-level locking, and foreign key support. In Express.js, it is accessed through `mysql2`, which offers a Promise-based API, connection pooling via `createPool()`, and prepared statements via `pool.execute()`. `mysql2` is preferred over the legacy `mysql` package for its Promise support and improved performance. For complex applications, Sequelize or Prisma can be layered on top.

**Beginner-Friendly Explanation**
MySQL is the database behind a huge portion of the web. It is fast, reliable, and available on every managed hosting platform. If you are building a typical web application, MySQL is a safe, well-supported choice.

---

### Purposes

- To provide a fast, widely supported relational database for web applications.
- To leverage the largest ecosystem of hosting, tooling, and community knowledge.
- To support ACID transactions and foreign key integrity through InnoDB.
- To offer straightforward scaling through read replicas and managed services.

---

### Syntax Rules and Structure

#### Complete General Syntax: `mysql2` Connection Pool

```javascript
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `mysql2/promise` | Promise-based API |
| `createPool()` | Creates a reusable connection pool |
| `connectionLimit: 10` | Caps concurrent connections |
| `pool.execute()` | Prepared statement with `?` placeholders |

#### Complete General Syntax: Prepared Statement

```javascript
const [rows] = await pool.execute(
  'SELECT * FROM users WHERE email = ?',
  [email]
);
```

#### Syntax Rules

- **Use `pool.execute()`** for parameterized queries — it uses prepared statements and prevents SQL injection.
- **Use `pool.query()`** only for queries with no user input (e.g., static DDL).
- **Acquire explicit connections** only for transactions, and release them in `finally`.
- **Load credentials from `.env`** before initializing the pool.

#### Constraints and Limitations

- `mysql2` prepared statements use `?` placeholders (not `$1` like PostgreSQL).
- Connection limits must respect MySQL’s `max_connections` setting.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: MySQL Prepared Statement with Pool

```javascript
const express = require('express');
const mysql = require('mysql2/promise');
const app = express();

const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  connectionLimit: 10
});

app.get('/users/:id', async (req, res, next) => {
  try {
    const [rows] = await pool.execute(
      'SELECT id, email, name FROM users WHERE id = ?',
      [req.params.id]
    );
    if (rows.length === 0) return res.status(404).json({ error: 'Not found' });
    res.json(rows[0]);
  } catch (err) {
    next(err);
  }
});

app.listen(3000);
```

**Expected Output**

```
GET /users/1   → 200 { "id": 1, "email": "alice@example.com", "name": "Alice" }
GET /users/99  → 404 { "error": "Not found" }
```

**Why This Output Occurs**

- `pool.execute()` sends a prepared statement with the ID bound as a parameter, preventing injection.

---

### Real-World Cases

- **E-commerce platforms** — Orders, inventory, and customer data with InnoDB transactions.
- **Content management systems** — WordPress and countless CMSs rely on MySQL.
- **High-read web applications** — Read replicas scale query throughput.

---

## Core Concept 3: MariaDB

### Definitions

**Core Definition**
MariaDB is a community-developed, fully open-source fork of MySQL, designed as a drop-in replacement with additional storage engines, performance improvements, and a purely GPL license.

**Technical Definition**
MariaDB shares MySQL’s protocol, SQL syntax, and client libraries, making it a **drop-in replacement** in most Express.js applications. The `mysql2` driver works with MariaDB without modification, or the dedicated `mariadb` connector can be used. MariaDB adds features such as system-versioned tables, sequences, thread pooling without an Enterprise license, and the ColumnStore analytics engine. Benchmarks have shown MariaDB performing 13–36% faster than MySQL in some workloads.

**Beginner-Friendly Explanation**
MariaDB is MySQL’s open-source twin. It speaks the same language and uses the same drivers, so switching between them is usually effortless. It adds some extra features and is guaranteed to remain fully open source.

---

### Purposes

- To provide a fully open-source MySQL-compatible database without proprietary editions.
- To leverage MariaDB-specific features such as system-versioned tables and sequences.
- To use Galera Cluster for synchronous multi-master replication.
- To run the ColumnStore engine for analytical workloads alongside transactional data.

---

### Syntax Rules and Structure

#### Complete General Syntax: MariaDB with `mysql2`

```javascript
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  connectionLimit: 10
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `mysql2` | Works with MariaDB because the protocol is compatible |
| `mariadb` (alternative) | Dedicated MariaDB connector with its own API |
| `.query()` vs `.execute()` | `mysql2` uses `.execute()` for prepared statements; the `mariadb` connector uses `.query()` for everything |

#### Syntax Rules

- **`mysql2` works unchanged** with MariaDB — no code changes are required for a basic migration.
- The `mariadb` connector uses `.query()` for both simple and prepared queries.
- Verify JSON functions: MariaDB does **not** support MySQL’s `JSON_TABLE` function.

#### Constraints and Limitations

- MariaDB and MySQL diverge in JSON storage, GTID replication format, and available storage engines.
- MySQL’s Group Replication has no direct MariaDB equivalent; MariaDB uses Galera instead.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: MariaDB with `mysql2` Pool

```javascript
const express = require('express');
const mysql = require('mysql2/promise');
const app = express();

const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  connectionLimit: 10
});

app.get('/products', async (req, res, next) => {
  try {
    const [rows] = await pool.execute('SELECT id, name, price FROM products');
    res.json(rows);
  } catch (err) {
    next(err);
  }
});

app.listen(3000);
```

**Expected Output**

```
GET /products → 200 [{ "id": 1, "name": "Widget", "price": 9.99 }]
```

**Why This Output Occurs**

- `mysql2` connects to MariaDB using the shared MySQL protocol; the query and response handling are identical.

---

### Real-World Cases

- **Open-source-first organizations** — No proprietary licensing concerns.
- **Analytics + transactions** — ColumnStore for analytics alongside InnoDB for OLTP.
- **High-availability clusters** — Galera Cluster for synchronous multi-master replication.

---

## Core Concept 4: SQL Server

### Definitions

**Core Definition**
Microsoft SQL Server is a commercial relational database management system widely used in enterprise environments, particularly in organizations standardized on the Microsoft technology stack.

**Technical Definition**
In Express.js, SQL Server is accessed through the `mssql` package, which wraps the low-level `tedious` driver and provides connection pooling, Promise-based queries, and parameterized inputs via `pool.request()`. Direct use of `tedious` is appropriate only when fine-grained protocol control is required; `mssql` is the right choice for the vast majority of applications. Configuration includes `server`, `database`, `user`, `password`, and options such as `enableArithAbort` and `encrypt`.

**Beginner-Friendly Explanation**
SQL Server is the enterprise database from Microsoft. If your company already runs Windows servers and Microsoft tooling, SQL Server fits naturally. In Express, the `mssql` package makes it easy to connect and query.

---

### Purposes

- To integrate Express applications with existing Microsoft enterprise infrastructure.
- To leverage SQL Server’s enterprise features: Always On availability groups, row-level security, and advanced analytics.
- To use a mature, commercially supported database with strong tooling (SSMS, Azure Data Studio).

---

### Syntax Rules and Structure

#### Complete General Syntax: `mssql` Connection Pool

```javascript
const sql = require('mssql');

const config = {
  server: process.env.DB_SERVER,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  options: {
    enableArithAbort: true,   // required for newer SQL Server versions
    encrypt: false,            // set to true for Azure SQL
    instanceName: 'sqlexpress' // optional
  }
};

const poolPromise = new sql.ConnectionPool(config).connect();
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `mssql` | High-level client library |
| `tedious` | Underlying low-level TDS driver |
| `ConnectionPool` | Manages reusable connections |
| `enableArithAbort` | Required option for modern SQL Server versions |
| `encrypt` | `true` for Azure SQL and encrypted connections |

#### Complete General Syntax: Parameterized Query

```javascript
const pool = await poolPromise;
const result = await pool.request()
  .input('id', sql.Int, req.params.id)
  .query('SELECT id, name FROM users WHERE id = @id');
```

#### Syntax Rules

- **Use `mssql`**, not raw `tedious`, for application development — it provides pooling and a friendlier API.
- **Use `.input()`** for parameterized queries; never concatenate user input into the SQL string.
- **Set `enableArithAbort: true`** for newer SQL Server versions.
- **Set `encrypt: true`** for Azure SQL and any encrypted connection.

#### Constraints and Limitations

- SQL Server uses `@parameter` syntax (not `$1` or `?`).
- `tedious` is the default driver under `mssql`; you do not need to install it separately unless you need low-level control.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: SQL Server Query with `mssql`

```javascript
const express = require('express');
const sql = require('mssql');
const app = express();

const config = {
  server: process.env.DB_SERVER,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  options: { enableArithAbort: true, encrypt: false }
};

const poolPromise = new sql.ConnectionPool(config).connect();

app.get('/users/:id', async (req, res, next) => {
  try {
    const pool = await poolPromise;
    const result = await pool.request()
      .input('id', sql.Int, req.params.id)
      .query('SELECT id, email, name FROM users WHERE id = @id');
    if (result.recordset.length === 0) {
      return res.status(404).json({ error: 'Not found' });
    }
    res.json(result.recordset[0]);
  } catch (err) {
    next(err);
  }
});

app.listen(3000);
```

**Expected Output**

```
GET /users/1   → 200 { "id": 1, "email": "alice@example.com", "name": "Alice" }
GET /users/99  → 404 { "error": "Not found" }
```

**Why This Output Occurs**

- `.input()` binds the ID as a typed parameter; `.query()` executes the parameterized statement and returns `recordset`.

---

### Real-World Cases

- **Enterprise applications** — ERP, CRM, and internal tools on the Microsoft stack.
- **Azure-hosted APIs** — Azure SQL Database with encrypted connections.
- **Reporting and analytics** — SQL Server Reporting Services and tabular models.

---

## Core Concept 5: SQLite

### Definitions

**Core Definition**
SQLite is a serverless, file-based relational database engine that stores the entire database in a single file, requiring no separate server process and no configuration.

**Technical Definition**
SQLite is embedded directly into the application process. In Express.js, `better-sqlite3` is the preferred driver: it provides a **synchronous** API that is simpler and often faster than callback-based drivers for simple queries, and it avoids accidental deadlocks. Write-Ahead Logging (WAL) mode significantly improves concurrent read/write throughput. SQLite is ideal for prototypes, small-to-medium applications, desktop and mobile apps, and testing, but it does not scale to high-concurrency multi-server deployments.

**Beginner-Friendly Explanation**
SQLite is a database in a single file. There is no server to install or manage. It is perfect for small projects, prototypes, and tests. For a large production API with many concurrent users, you will want a server-based database instead.

---

### Purposes

- To provide zero-configuration persistence for prototypes, tests, and small applications.
- To enable embedded databases in desktop, mobile, and edge deployments.
- To offer fast, synchronous queries without connection-pool overhead.
- To serve as a lightweight development database while targeting a server-based database in production.

---

### Syntax Rules and Structure

#### Complete General Syntax: `better-sqlite3` Setup

```javascript
const Database = require('better-sqlite3');
const db = new Database('app.db');

// Enable WAL mode for better concurrency
db.pragma('journal_mode = WAL');

// Prepared statement
const getUser = db.prepare('SELECT id, email, name FROM users WHERE id = ?');
const user = getUser.get(1);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `new Database('app.db')` | Opens (or creates) the database file |
| `db.pragma('journal_mode = WAL')` | Enables write-ahead logging for concurrency |
| `db.prepare()` | Creates a reusable prepared statement |
| `.get()` | Returns the first matching row |

#### Complete General Syntax: Schema Creation

```javascript
db.exec(`
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE NOT NULL,
    name TEXT NOT NULL
  )
`);
```

#### Syntax Rules

- **Use `better-sqlite3`** for its synchronous API and performance.
- **Enable WAL mode** (`journal_mode = WAL`) for concurrent read/write workloads.
- **Use prepared statements** with `?` placeholders for parameterized queries.
- **Do not use SQLite** for high-concurrency multi-server production APIs.

#### Constraints and Limitations

- SQLite serializes writes; concurrent writes can be slow under load.
- WAL mode can experience “checkpoint starvation” under certain workloads.
- The entire database is a single file; backups and replication require file-level tooling.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: SQLite User Lookup with `better-sqlite3`

```javascript
const express = require('express');
const Database = require('better-sqlite3');
const app = express();

const db = new Database('app.db');
db.pragma('journal_mode = WAL');
db.exec(`
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE NOT NULL,
    name TEXT NOT NULL
  )
`);

app.get('/users/:id', (req, res) => {
  const row = db.prepare('SELECT id, email, name FROM users WHERE id = ?')
    .get(req.params.id);
  if (!row) return res.status(404).json({ error: 'Not found' });
  res.json(row);
});

app.listen(3000);
```

**Expected Output**

```
GET /users/1   → 200 { "id": 1, "email": "alice@example.com", "name": "Alice" }
GET /users/99  → 404 { "error": "Not found" }
```

**Why This Output Occurs**

- `better-sqlite3` executes the prepared statement synchronously and returns the row directly.

---

### Real-World Cases

- **Prototypes and MVPs** — Zero setup; the database is just a file.
- **Testing** — In-memory SQLite databases provide fast, isolated tests.
- **Electron and mobile apps** — Embedded persistence without a server.
- **Small production APIs** — Single-server applications with low write concurrency.

---

## SQL Database Comparison

| Database | License | Driver | Placeholder Style | Pooling | Best For |
|----------|---------|--------|-------------------|---------|----------|
| **PostgreSQL** | Open source (PostgreSQL License) | `pg` | `$1`, `$2` | `pg.Pool` | Complex queries, JSONB, ACID, default choice |
| **MySQL** | GPL / commercial | `mysql2` | `?` | `createPool()` | Widest hosting ecosystem, web applications |
| **MariaDB** | GPL only | `mysql2` or `mariadb` | `?` | `createPool()` | Fully open-source MySQL alternative |
| **SQL Server** | Commercial | `mssql` (wraps `tedious`) | `@param` | `ConnectionPool` | Enterprise Microsoft environments |
| **SQLite** | Public domain | `better-sqlite3` | `?` | Not applicable (embedded) | Prototypes, tests, embedded apps |

---

## Core Concept 6: SQL Database Best Practices

### Definitions

**Core Definition**
SQL database best practices in Express.js are the design, security, and operational patterns that keep database interactions reliable, performant, secure, and maintainable across the application lifecycle.

**Technical Definition**
Best practices encompass: **connection pooling** to reuse database connections rather than opening a new one per request; **parameterized queries** to prevent SQL injection; **environment-variable credential management**; **transaction handling** with explicit commit and rollback; **migration-based schema evolution** with version-controlled up/down scripts; **indexing** of query-critical columns; **N+1 query prevention** through joins or eager loading; **least-privilege database users**; and **TLS** for remote connections. The database should be treated as an external dependency with a clear interface—a repository or data-access layer—rather than as a global variable that any file can `require()`.

**Beginner-Friendly Explanation**
Best practices are the habits that keep your database fast and safe. Use a pool of connections instead of opening a new one for every request. Use parameterized queries so attackers cannot inject SQL. Keep credentials in environment variables. Use transactions when multiple changes must succeed or fail together. Track schema changes with migrations. And never let user input become part of a SQL string.

---

### Purposes

- To prevent SQL injection and credential leakage.
- To optimize performance through pooling, indexing, and query design.
- To maintain data integrity through transactions and constraints.
- To keep schema changes version-controlled and reproducible.
- To enforce least privilege and encrypt connections.

---

### Syntax Rules and Structure

#### Complete General Syntax: Best-Practice Data Layer

```javascript
// db.js — single pool, exported once
const { Pool } = require('pg');
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30_000
});

async function query(text, params) {
  const client = await pool.connect();
  try {
    return (await client.query(text, params)).rows;
  } finally {
    client.release();
  }
}

async function withTransaction(fn) {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    const result = await fn(client);
    await client.query('COMMIT');
    return result;
  } catch (err) {
    await client.query('ROLLBACK');
    throw err;
  } finally {
    client.release();
  }
}

module.exports = { query, withTransaction };
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Single pool | Created once and exported |
| Query wrapper | Acquires, releases in `finally` |
| Transaction wrapper | `BEGIN` → `COMMIT` / `ROLLBACK` |
| Parameterized queries | `$1`, `$2` placeholders |
| Repository pattern | Data access isolated from controllers |

#### Complete General Syntax: Parameterized Query (Never Concatenate)

```javascript
// SAFE
await query('SELECT * FROM users WHERE email = $1', [email]);

// NEVER DO THIS
// await query(`SELECT * FROM users WHERE email = '${email}'`);
```

#### Complete General Syntax: Migration with Knex

```javascript
exports.up = async (knex) => {
  await knex.schema.createTable('users', (t) => {
    t.increments('id').primary();
    t.string('email').unique().notNullable();
    t.string('name').notNullable();
    t.timestamps(true, true);
  });
};

exports.down = async (knex) => {
  await knex.schema.dropTable('users');
};
```

#### Syntax Rules

- **Always use a connection pool** — never open a new connection per request.
- **Always use parameterized queries** — never concatenate user input into SQL.
- **Always release connections** in a `finally` block.
- **Always use transactions** for multi-step writes that must be atomic.
- **Store credentials in environment variables** and enable TLS for remote connections.
- **Use migrations** (Knex, Prisma, node-pg-migrate, dbmate) — never modify production schemas by hand.
- **Index columns** used in `WHERE`, `JOIN`, and `ORDER BY` clauses.
- **Prevent N+1 queries** by using joins or eager loading.
- **Treat the database as an external dependency** with a clear repository interface.

#### Constraints and Limitations

- Pool size must be tuned to the database’s connection limit; too many connections degrade performance.
- ORMs do not eliminate SQL injection risk entirely; raw query fragments still require parameterization.
- Migrations in production require careful planning and rollback strategies.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Complete Best-Practice PostgreSQL Pipeline

```javascript
const express = require('express');
const { Pool } = require('pg');
const { body, param, validationResult } = require('express-validator');
const app = express();
app.use(express.json());

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30_000
});

// Parameterized query + validation
app.get('/users/:id',
  param('id').isInt({ min: 1 }).toInt(),
  async (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });

    try {
      const result = await pool.query(
        'SELECT id, email, name FROM users WHERE id = $1',
        [req.params.id]
      );
      if (result.rows.length === 0) {
        return res.status(404).json({ error: 'User not found' });
      }
      res.json(result.rows[0]);
    } catch (err) {
      next(err);
    }
  }
);

// Transaction with rollback
app.post('/transfer', async (req, res, next) => {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    await client.query(
      'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
      [req.body.amount, req.body.fromId]
    );
    await client.query(
      'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
      [req.body.amount, req.body.toId]
    );
    await client.query('COMMIT');
    res.json({ success: true });
  } catch (err) {
    await client.query('ROLLBACK');
    next(err);
  } finally {
    client.release();
  }
});

app.listen(3000);
```

**Expected Output**

```
GET /users/1      → 200 { "id": 1, "email": "...", "name": "..." }
GET /users/abc    → 400 { "errors": [...] }
POST /transfer    (valid) → 200 { "success": true }
POST /transfer    (error) → 500 (both balances unchanged due to ROLLBACK)
```

**Why This Output Occurs**

- Validation rejects invalid input; the parameterized query prevents injection; the transaction ensures atomicity; the connection is released in `finally`.

---

### Real-World Cases

- **Production APIs** — Connection pooling, parameterized queries, and transaction handling.
- **Financial systems** — ACID transactions with rollback and least-privilege database users.
- **High-traffic applications** — Indexed columns, N+1 query prevention, and read replicas.

---

## References

- Database Integration in Express.js: Practical Patterns for MongoDB, PostgreSQL, and MySQL (2026) · TheLinuxCode — https://thelinuxcode.com/database-integration-in-expressjs-practical-patterns-for-mongodb-postgresql-and-mysql-2026/
- Database Architecture Patterns · PeterHdd/agent-skills — https://github.com/peterhdd/agent-skills/blob/main/skills/engineering-backend-architect/references/database-patterns.md
- What are the best practices for using MySQL with Node.js and Express? · GitHub Community — https://github.com/orgs/community/discussions/177127
- Express.js and MySQL via `mysql2` pools · OneUptime — https://github.com/OneUptime/blog/blob/master/posts/2026-03-31-mysql-use-mysql-with-expressjs/README.md
- MySQL vs MariaDB in 2026 · Mako — https://mako.ai/blog/mysql-vs-mariadb
- MySQL vs MariaDB · Layerbase — https://layerbase.com/blog/mysql-vs-mariadb
- Microsoft SQL Server · Remult Documentation — https://remult.dev/docs/installation/database/mssql
- tedious vs mssql · npm-compare — https://npm-compare.com/mssql,tedious
- tediuos vs mssql reviewed for Node 22 · MrKeyoor — https://mrkeyoor.com/blog/tedious-vs-mssql
- better-sqlite3 with Express · GitHub Issue #1235 — https://github.com/WiseLibs/better-sqlite3/issues/1235
- better-sqlite3 performance and WAL mode · GitCode Blog — https://blog.gitcode.com
- SQL Injection Prevention · Compile-N-Run — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/8-express-security/0-express-security-overview.mdx
- Database Migrations: Version Control for Your Schema · KindaTechnical — https://kindatechnical.com/database/database-migrations-version-control-for-your-schema.html
- Zero-Downtime Database Migrations for Node.js APIs · Chirag Mehta — https://chiragmehta.dev/blog/zero-downtime-database-migrations
- node-pg-migrate · GitHub — https://github.com/salsita/node-pg-migrate
- Express.js and MySQL work well together via `mysql2` pools · OneUptime Blog — https://oneuptime.com/blog/post/2026-03-31-mysql-use-mysql-with-expressjs/view