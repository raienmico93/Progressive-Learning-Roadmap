# Express.js Database Drivers — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Database drivers in Express.js are the low-level Node.js libraries that establish connections to a database server, transmit queries, and return results, forming the essential bridge between the Express application and the data store.

**Technical Definition**
A database driver implements the client side of a database's wire protocol. In the Node.js ecosystem, drivers for SQL databases fall into two categories: **pure-JavaScript drivers** (e.g., `mysql2`, `mssql` via `tedious`, `better-sqlite3` via native bindings) and **native-binding drivers** (e.g., `better-sqlite3`). The dominant drivers are `pg` for PostgreSQL, `mysql2` for MySQL and MariaDB, `better-sqlite3` for SQLite, and `mssql` for Microsoft SQL Server. Each driver provides connection pooling, parameterized query support, and Promise-based APIs (either natively or via a `/promise` entry point). Express is database-agnostic; the driver is chosen by the developer and integrated at the data-access layer.

**Beginner-Friendly Explanation**
A database driver is the translator between your Express app and your database. Your app speaks JavaScript; the database speaks its own protocol (PostgreSQL’s wire protocol, MySQL’s protocol, SQL Server’s TDS). The driver translates JavaScript function calls into that protocol and turns the database’s responses back into JavaScript objects. It also manages connections—opening them, reusing them, and closing them—so your app does not drown the database in new connections on every request.

---

### Key Characteristics

- **Protocol-specific** — Each driver implements its database’s wire protocol; you cannot use `pg` to talk to MySQL or `mysql2` to talk to PostgreSQL.
- **Pooling is foundational** — Production drivers provide connection pools to avoid the handshake cost (20–30 ms per connection) and to respect the database’s connection limit .
- **Parameterized queries are mandatory** — All four drivers support bound parameters (`$1` for `pg`, `?` for `mysql2`, `@param` for `mssql`), which is the primary defense against SQL injection.
- **Transaction scope is driver-specific** — In `pg`, a transaction must use a single checked-out client, never `pool.query` . In `mysql2`, `beginTransaction()` is available on a connection, but `pool.query` is not transaction-scoped.
- **Synchronous vs. asynchronous** — `better-sqlite3` is intentionally synchronous, which simplifies code but serializes writes; the server-based drivers are asynchronous and require careful connection management.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- A running database server (PostgreSQL, MySQL/MariaDB, SQL Server) or an SQLite file.
- The driver package installed for your chosen database.
- Environment variables configured for database credentials (`dotenv`).
- Basic understanding of SQL and connection pooling concepts.

---

### Related Programming Areas

- **Connection pooling** — The most important operational concern for any driver.
- **SQL injection prevention** — Parameterized queries through the driver.
- **Transactions** — Driver-specific transaction APIs and scoping rules.
- **ORMs and query builders** — `pg` underlies Knex and Prisma; `mysql2` underlies Sequelize; `better-sqlite3` is often used directly.
- **Observability** — Query logging and connection-pool monitoring.

---

## Core Concept 1: `pg` (node-postgres)

### Definitions

**Core Definition**
`pg` (node-postgres) is the non-blocking PostgreSQL client for Node.js, providing both a low-level `Client` and a built-in connection pool (`Pool`) for managing reusable connections.

**Technical Definition**
`pg` ships with built-in connection pooling via the `pg-pool` module . The `Pool` class manages a set of reusable clients; `pool.query()` checks out a client, runs the query, and returns the client to the pool automatically, eliminating the risk of leaking a client . For transactions, a single client must be checked out with `pool.connect()` and used for all `BEGIN`/`COMMIT`/`ROLLBACK` statements, because PostgreSQL scopes transactions to individual clients . Queries use `$1`, `$2` placeholders for parameterization.

**Beginner-Friendly Explanation**
`pg` is the standard PostgreSQL driver. It has a built-in pool, so you create one pool at startup and reuse it for every query. The pool handles the handshake cost and connection limits. For a single query, you call `pool.query()`. For a transaction, you check out a client, run all your statements on that same client, and release it when done.

---

### Purposes

- To provide the most direct and widely used connection to PostgreSQL from Express.
- To enable efficient connection reuse through built-in pooling.
- To support parameterized queries and transactions with ACID guarantees.
- To serve as the foundation for query builders (Knex) and ORMs (Prisma, TypeORM, Sequelize).

---

### Syntax Rules and Structure

#### Complete General Syntax: Pool Creation

```javascript
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,                        // maximum clients in pool
  idleTimeoutMillis: 30_000,      // close idle clients after 30s
  connectionTimeoutMillis: 5_000  // fail if connection takes > 5s
});

pool.on('error', (err) => {
  console.error('Unexpected idle client error', err);
  process.exit(-1);
});
```

**Component Breakdown**

| Option | Role |
|--------|------|
| `connectionString` | PostgreSQL connection URI |
| `max` | Maximum number of clients in the pool |
| `idleTimeoutMillis` | How long an idle client remains in the pool |
| `connectionTimeoutMillis` | How long to wait for a connection before failing |
| `pool.on('error')` | Handles errors on idle clients |

#### Complete General Syntax: Single Query

```javascript
const res = await pool.query('SELECT * FROM users WHERE id = $1', [1]);
console.log(res.rows[0]);
```

#### Complete General Syntax: Transaction

```javascript
const client = await pool.connect();
try {
  await client.query('BEGIN');
  await client.query('UPDATE accounts SET balance = balance - $1 WHERE id = $2', [100, 1]);
  await client.query('UPDATE accounts SET balance = balance + $1 WHERE id = $2', [100, 2]);
  await client.query('COMMIT');
} catch (e) {
  await client.query('ROLLBACK');
  throw e;
} finally {
  client.release();
}
```

#### Syntax Rules

- **Use `pool.query()` for single queries** — it removes the risk of leaking a client .
- **Never use `pool.query()` for transactions** — PostgreSQL scopes transactions to individual clients .
- **Always release the client** in a `finally` block, regardless of success or failure .
- **Always use parameterized queries** with `$1`, `$2` placeholders.

#### Constraints and Limitations

- The pool must be tuned to respect PostgreSQL’s `max_connections` limit; an unbounded number of clients can crash the server .
- `pool.query()` does not support transactions.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Pooled Query with Error Handling

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

- `pool.query()` checks out a client, runs the parameterized query, returns the row(s), and releases the client automatically.

---

#### Example 2: Transaction with Client Checkout

```javascript
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
```

**Expected Output**

```
POST /transfer  (valid) → 200 { "success": true }
POST /transfer  (error) → 500 (both balances unchanged due to ROLLBACK)
```

**Why This Output Occurs**

- All statements run on the same checked-out client, so PostgreSQL treats them as one transaction; `ROLLBACK` undoes both updates if either fails .

---

### Real-World Cases

- **Express APIs** — The default PostgreSQL driver for Knex, Prisma, and most Express PostgreSQL tutorials.
- **Financial transactions** — ACID transfers using client-checked-out transactions.
- **Multi-tenant SaaS** — Pooled connections shared across tenants with parameterized tenant-scoped queries.

---

## Core Concept 2: `mysql2`

### Definitions

**Core Definition**
`mysql2` is the modern MySQL driver for Node.js, offering a Promise-based API, connection pooling, and prepared statements, and is the recommended replacement for the legacy `mysql` package.

**Technical Definition**
`mysql2` provides both callback-based and Promise-based APIs; the `mysql2/promise` entry point exposes Promises directly. The pool is created with `createPool()`, and queries can use either `pool.query()` (simple query, no prepared statement) or `pool.execute()` (prepared statement with `?` placeholders). Prepared statements are automatically created, cached, and reused by the connection . For transactions, a connection must be checked out with `pool.getConnection()`, and `beginTransaction()`, `commit()`, and `rollback()` are called on that connection.

**Beginner-Friendly Explanation**
`mysql2` is the standard MySQL and MariaDB driver. It has a pool like `pg`, but it offers two query methods: `query()` for simple queries and `execute()` for prepared statements. Use `execute()` when user input is involved—it protects against SQL injection. For transactions, check out a connection, run your statements, and release the connection when done.

---

### Purposes

- To provide a fast, Promise-friendly MySQL and MariaDB driver.
- To support both simple queries and prepared statements with automatic caching.
- To enable connection pooling and transaction handling.
- To serve as the database layer for Sequelize and many Express MySQL applications.

---

### Syntax Rules and Structure

#### Complete General Syntax: Pool Creation

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

| Option | Role |
|--------|------|
| `waitForConnections` | Queue requests when the pool is exhausted |
| `connectionLimit` | Maximum simultaneous connections |
| `queueLimit` | Maximum queued requests (0 = unlimited) |

#### Complete General Syntax: Prepared Statement

```javascript
const [rows] = await pool.execute(
  'SELECT * FROM users WHERE email = ?',
  [email]
);
```

#### Complete General Syntax: Transaction

```javascript
const connection = await pool.getConnection();
try {
  await connection.beginTransaction();
  await connection.execute('UPDATE accounts SET balance = balance - ? WHERE id = ?', [100, 1]);
  await connection.execute('UPDATE accounts SET balance = balance + ? WHERE id = ?', [100, 2]);
  await connection.commit();
} catch (err) {
  await connection.rollback();
  throw err;
} finally {
  connection.release();
}
```

#### Syntax Rules

- **Use `pool.execute()` for parameterized queries** — prepared statements are cached and reused .
- **Use `pool.query()`** only for queries with no user input (e.g., static DDL).
- **Always release the connection** with `connection.release()` in a `finally` block .
- **Use `beginTransaction()`, `commit()`, and `rollback()`** on a checked-out connection for transactions.

#### Constraints and Limitations

- Prepared statements cannot contain multiple statements .
- The maximum number of prepared statements is `connectionLimit × maxPreparedStatements`; tune carefully .
- `mysql2` works with MariaDB unchanged, but some MySQL-specific JSON functions are unavailable in MariaDB.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Prepared Statement with Promise Pool

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

#### Example 2: Transaction with `beginTransaction()`

```javascript
app.post('/transfer', async (req, res, next) => {
  const connection = await pool.getConnection();
  try {
    await connection.beginTransaction();
    await connection.execute('UPDATE accounts SET balance = balance - ? WHERE id = ?', [req.body.amount, req.body.fromId]);
    await connection.execute('UPDATE accounts SET balance = balance + ? WHERE id = ?', [req.body.amount, req.body.toId]);
    await connection.commit();
    res.json({ success: true });
  } catch (err) {
    await connection.rollback();
    next(err);
  } finally {
    connection.release();
  }
});
```

**Expected Output**

```
POST /transfer  (valid) → 200 { "success": true }
POST /transfer  (error) → 500 (both balances unchanged due to rollback)
```

**Why This Output Occurs**

- All statements run on the same checked-out connection; `commit()` persists both, and `rollback()` undoes both on failure.

---

### Real-World Cases

- **MySQL and MariaDB APIs** — The standard driver for Express applications using either database.
- **High-read web applications** — Read replicas with pooled prepared statements.
- **Sequelize-based applications** — Sequelize uses `mysql2` under the hood.

---

## Core Concept 3: `better-sqlite3`

### Definitions

**Core Definition**
`better-sqlite3` is a synchronous SQLite3 driver for Node.js that provides a simple, fast, and reliable API for embedded file-based databases, using prepared statements and optional write-ahead logging for concurrency.

**Technical Definition**
Unlike server-based drivers, `better-sqlite3` operates on a local database file and exposes a synchronous API. The library recommends enabling **WAL mode** (`db.pragma('journal_mode = WAL')`) to greatly increase overall performance in web applications with concurrent reads and writes . WAL mode allows readers and writers to proceed concurrently, though writes are still serialized. The library compiles SQLite with `SQLITE_DEFAULT_WAL_SYNCHRONOUS=1` (NORMAL synchronous mode) for performance, with the option to override to `FULL` for durability . Checkpoint starvation can occur with multiple processes; `wal_checkpoint(RESTART)` mitigates it .

**Beginner-Friendly Explanation**
`better-sqlite3` is a database in a single file with no server to run. It is synchronous, which means your code waits for each query to finish—simple, but it serializes writes. For web apps, turn on WAL mode so reads and writes can happen at the same time. It is perfect for prototypes, tests, and small apps; for high-concurrency multi-server production, use a server-based database instead.

---

### Purposes

- To provide zero-configuration, embedded persistence for prototypes, tests, and small applications.
- To offer a fast, synchronous API that avoids callback nesting and pooling complexity.
- To enable concurrent reads and writes through WAL mode.
- To serve as a lightweight development database while targeting a server-based database in production.

---

### Syntax Rules and Structure

#### Complete General Syntax: Database Setup

```javascript
const Database = require('better-sqlite3');
const db = new Database('app.db');

// Enable WAL mode for better concurrency
db.pragma('journal_mode = WAL');

// Optional: override durability to FULL
// db.pragma('synchronous = FULL');
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `new Database('app.db')` | Opens (or creates) the database file |
| `db.pragma('journal_mode = WAL')` | Enables write-ahead logging  |
| `db.pragma('synchronous = FULL')` | Optional override for durability  |

#### Complete General Syntax: Prepared Statement

```javascript
const stmt = db.prepare('SELECT id, email, name FROM users WHERE id = ?');
const row = stmt.get(1);
```

#### Syntax Rules

- **Enable WAL mode** for web applications with concurrent reads and writes .
- **Use prepared statements** (`db.prepare()`) with `?` placeholders for parameterized queries.
- **Use `.get()` for single rows**, `.all()` for arrays, and `.run()` for writes.
- **Consider `wal_checkpoint(RESTART)`** if the WAL file grows under multi-process access .

#### Constraints and Limitations

- SQLite serializes writes; high write concurrency is limited.
- WAL mode has a slight durability trade-off (`NORMAL` synchronous) that can be overridden to `FULL` .
- Not suitable for multi-server production deployments.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: WAL-Enabled SQLite with Prepared Statements

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
  const row = db.prepare('SELECT id, email, name FROM users WHERE id = ?').get(req.params.id);
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

- `better-sqlite3` executes the prepared statement synchronously and returns the row directly; WAL mode enables concurrent reads and writes.

---

### Real-World Cases

- **Prototypes and MVPs** — Zero setup; the database is just a file.
- **Testing** — In-memory SQLite databases provide fast, isolated tests.
- **Electron and mobile apps** — Embedded persistence without a server.
- **Small production APIs** — Single-server applications with low write concurrency .

---

## Core Concept 4: `mssql`

### Definitions

**Core Definition**
`mssql` is the Microsoft SQL Server client for Node.js, providing a high-level API with connection pooling, parameterized queries, and streaming, built on top of the low-level `tedious` TDS driver.

**Technical Definition**
`mssql` uses a `ConnectionPool` for connection management; the recommended pattern is to create a pool singleton at startup and reuse it for all requests . Queries use `pool.request().input('name', sql.Type, value).query('... @name ...')` for parameterized execution. The library supports a global pool or a user-managed pool; a singleton is generally recommended . Connection validation can use `'socket'` mode to avoid the overhead of `SELECT 1` health checks at scale . Pool options include `max`, `min`, and `idleTimeoutMillis` .

**Beginner-Friendly Explanation**
`mssql` is the SQL Server driver. It has a pool like `pg` and `mysql2`. You create one pool and use it for all queries. For parameterized queries, you call `.input()` to bind each value with its SQL type, then `.query()` with `@name` placeholders. If you are on a large deployment, `'socket'` validation mode can reduce the overhead of connection health checks.

---

### Purposes

- To integrate Express applications with Microsoft SQL Server and Azure SQL Database.
- To provide connection pooling with configurable size and timeout.
- To support parameterized queries with explicit SQL types.
- To enable streaming for large result sets.

---

### Syntax Rules and Structure

#### Complete General Syntax: Connection Pool

```javascript
const sql = require('mssql');

const config = {
  server: process.env.DB_SERVER,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  options: {
    enableArithAbort: true,
    encrypt: true // for Azure SQL
  },
  pool: {
    max: 10,
    min: 0,
    idleTimeoutMillis: 30_000
  }
};

const poolPromise = new sql.ConnectionPool(config).connect();
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `sql.ConnectionPool` | Manages a pool of connections  |
| `pool.max` | Maximum connections in the pool  |
| `pool.min` | Minimum connections maintained |
| `pool.idleTimeoutMillis` | Idle connection timeout |
| `validateConnection: 'socket'` | Lightweight connection health check  |

#### Complete General Syntax: Parameterized Query

```javascript
const pool = await poolPromise;
const result = await pool.request()
  .input('id', sql.Int, req.params.id)
  .query('SELECT id, email, name FROM users WHERE id = @id');
```

#### Syntax Rules

- **Use a singleton pool** created at startup and reused across requests .
- **Use `.input()`** to bind parameters with explicit SQL types — never concatenate user input into the query string.
- **Set `enableArithAbort: true`** for modern SQL Server versions.
- **Use `'socket'` validation** in large pools to avoid `SELECT 1` overhead .

#### Constraints and Limitations

- SQL Server uses `@param` syntax, not `$1` or `?`.
- `tedious` is the default driver; `msnodesqlv8` is an alternative but has different timeout behavior .

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Parameterized Query with `mssql`

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

#### Example 2: Streaming Large Result Sets

```javascript
app.get('/export', async (req, res, next) => {
  try {
    const pool = await poolPromise;
    const request = pool.request();
    request.stream = true;
    request.query('SELECT * FROM large_table');
    request.on('row', (row) => res.write(JSON.stringify(row) + '\n'));
    request.on('error', next);
    request.on('done', () => res.end());
  } catch (err) {
    next(err);
  }
});
```

**Expected Output**

- Rows are streamed to the client as they arrive, avoiding loading the entire result set into memory .

---

### Real-World Cases

- **Enterprise applications** — ERP, CRM, and internal tools on the Microsoft stack.
- **Azure-hosted APIs** — Azure SQL Database with encrypted connections.
- **Large data exports** — Streaming for reports and ETL pipelines .

---

## Core Concept 5: Database Driver Best Practices

### Definitions

**Core Definition**
Database driver best practices are the design and operational patterns that keep driver usage safe, efficient, and maintainable across an Express application, regardless of which database is used.

**Technical Definition**
Best practices encompass: **creating a single pool singleton at startup** and reusing it across all requests ; **always using parameterized queries** with the driver’s placeholder syntax; **always releasing clients/connections** in `finally` blocks ; **using client-scoped transactions** for `pg` and `mysql2` ; **tuning pool size** to respect the database’s connection limit ; **enabling WAL mode** for `better-sqlite3` ; **using `'socket'` validation** for large `mssql` pools ; and **keeping credentials in environment variables**. The driver should be wrapped in a data-access layer so that business logic does not depend on driver-specific APIs.

**Beginner-Friendly Explanation**
Best practices are the habits that keep your driver from becoming a bottleneck or a security hole. Create one pool and reuse it. Never build SQL by concatenating strings—use parameters. Always release connections, even when an error occurs. Use a single client for transactions. And tune your pool to match what your database can handle.

---

### Purposes

- To prevent connection leaks and pool exhaustion.
- To prevent SQL injection through parameterized queries.
- To ensure transactions are correctly scoped to a single client.
- To optimize performance through appropriate pooling and WAL configuration.
- To keep driver-specific code isolated in a data-access layer.

---

### Syntax Rules and Structure

#### Complete General Syntax: Hardened Driver Wrapper

```javascript
// db.js — single pool, exported once
const { Pool } = require('pg');
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30_000
});

pool.on('error', (err) => {
  console.error('Unexpected idle client error', err);
  process.exit(-1);
});

async function query(text, params) {
  const client = await pool.connect();
  try {
    return (await client.query(text, params)).rows;
  } finally {
    client.release(); // always release
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
| Pool singleton | Created once, exported |
| Query wrapper | Acquires and releases in `finally` |
| Transaction wrapper | Single client, `BEGIN`/`COMMIT`/`ROLLBACK` |
| Error handling | `pool.on('error')` prevents silent failures  |
| Parameterized queries | `$1`, `$2` placeholders |

#### Driver-Specific Best Practices

| Driver | Key Best Practices |
|--------|-------------------|
| **`pg`** | Use `pool.query()` for single queries; use `pool.connect()` for transactions; always release  |
| **`mysql2`** | Use `pool.execute()` for prepared statements; release connections in `finally` ; transaction on a checked-out connection |
| **`better-sqlite3`** | Enable WAL mode ; use prepared statements; consider `wal_checkpoint(RESTART)` under multi-process access  |
| **`mssql`** | Use a singleton pool ; use `.input()` for parameters; use `'socket'` validation for large pools  |

#### Syntax Rules

- **Create one pool per application**, not one per request or module .
- **Always use parameterized queries** — never concatenate user input.
- **Always release clients/connections** in `finally` blocks .
- **Use a single client for transactions** in `pg` and `mysql2` .
- **Tune pool size** to the database’s connection limit .
- **Enable WAL mode** for `better-sqlite3` .
- **Isolate driver code** in a repository or data-access layer.

#### Constraints and Limitations

- Pool size must respect the database server’s `max_connections`; exceeding it can crash the server .
- `better-sqlite3` serializes writes; WAL mode does not remove this limitation .
- `mssql` `'socket'` validation does not detect server-side session invalidation .

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Complete Hardened PostgreSQL Driver Wrapper

```javascript
const express = require('express');
const { Pool } = require('pg');
const app = express();

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30_000
});

pool.on('error', (err) => {
  console.error('Unexpected idle client error', err);
  process.exit(-1);
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

app.get('/users/:id', async (req, res, next) => {
  try {
    const rows = await query('SELECT * FROM users WHERE id = $1', [req.params.id]);
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
GET /users/1   → 200 { user data }
GET /users/99  → 404 { "error": "Not found" }
```

**Why This Output Occurs**

- The wrapper acquires a client, runs the query, and releases the client in `finally`; the pool is created once and reused.

---

### Real-World Cases

- **Production APIs** — Single pool, parameterized queries, and transaction wrappers across all drivers.
- **High-concurrency deployments** — Pool sizing and WAL mode tuned to the workload .
- **Multi-database applications** — Driver-specific wrappers isolated in a data-access layer.

---

## Driver Comparison

| Driver | Database | Placeholder | Pool API | Transaction API | Sync/Async |
|--------|----------|-------------|----------|-----------------|------------|
| **`pg`** | PostgreSQL | `$1`, `$2` | `new Pool()` | `pool.connect()` + `BEGIN`/`COMMIT`/`ROLLBACK`  | Async |
| **`mysql2`** | MySQL/MariaDB | `?` | `createPool()` | `pool.getConnection()` + `beginTransaction()`/`commit()`/`rollback()` | Async |
| **`better-sqlite3`** | SQLite | `?` | Not applicable (embedded) | Manual `BEGIN`/`COMMIT`/`ROLLBACK` | Synchronous |
| **`mssql`** | SQL Server | `@param` | `new sql.ConnectionPool()`  | `pool.request()` with `BEGIN TRAN`/`COMMIT`/`ROLLBACK` | Async |

---

## References

- node-postgres: Pooling — https://node-postgres.com/features/pooling
- node-postgres: Transactions — https://node-postgres.com/features/transactions
- node-postgres: Examples — https://node-postgres.com/guides/project-structure
- mysql2: createPool — https://github.com/sidorares/node-mysql2/blob/2d3cad8a5aad37df19b6ebfcba0667a156101b06/website/docs/examples/connections/create-pool.mdx
- mysql2: Pools, `.execute()` and error handling (Discussion #2282) — https://github.com/sidorares/node-mysql2/discussions/2282
- better-sqlite3: Performance (WAL mode) — https://github.com/WiseLibs/better-sqlite3/blob/master/docs/performance.md
- better-sqlite3: Express.js usage (Issue #1235) — https://github.com/WiseLibs/better-sqlite3/issues/1235
- node-mssql: README (pool configuration) — https://github.com/tediousjs/node-mssql
- node-mssql: Connection validation (`'socket'` mode) — https://github.com/tediousjs/node-mssql#2
- node-mssql: Parameterized queries (`request.input`) — https://github.com/tediousjs/node-mssql#3
- mysql2: Query vs Execute (Discussion #1601) — https://github.com/sidorares/node-mysql2/discussions/1601