# Node.js Database Connectivity — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Node.js database connectivity refers to the drivers, connection management strategies, and query execution patterns that enable Node.js applications to communicate with relational and non-relational databases.

**Technical Definition:** Node.js database connectivity is achieved through driver libraries that implement database wire protocols (e.g., PostgreSQL's frontend/backend protocol, MySQL's protocol, SQL Server's TDS) over TCP or Unix sockets. Drivers expose connection pooling, parameterised queries, prepared statements, and transaction management. Modern drivers span three categories: native bindings (C++ addons), pure JavaScript implementations, and HTTP/WebSocket-based drivers for edge runtimes. Connection pooling is essential for production applications to avoid per-request connection overhead. Prepared statements leverage query plan caching for performance. Parameterised queries prevent SQL injection at the protocol level.

**Beginner-Friendly Explanation:** A database driver is like a telephone that lets your Node.js app call the database. Different databases speak different languages (protocols), so you need different phones (drivers). Some phones are fast but require special hardware (native drivers); others work anywhere (pure JavaScript). Connection pooling is like keeping a few phone lines open instead of dialing a new number for every call. Prepared statements are like saving a frequently used phone number for speed dial. Parameterised queries are like using a secure code instead of reading your password aloud.

### Key Characteristics

- **Driver categories:** Native bindings (fast, platform-specific), pure JavaScript (portable, slightly slower), and HTTP/WebSocket (edge-compatible).
- **Connection pooling:** Reuses database connections across requests, reducing handshake overhead and connection count.
- **Prepared statements:** Pre-compile SQL on the server, enabling query plan caching and reuse.
- **Parameterised queries:** Bind values to placeholders, preventing SQL injection at the protocol level.
- **SSL/TLS:** Encrypts database traffic and verifies server certificates via CA.
- **Serverless pooling:** PgBouncer, Prisma Accelerate, and Neon's proxy solve connection exhaustion in serverless environments.
- **Environment configuration:** Connection strings and credentials stored in environment variables, never hardcoded.

### Prerequisites

- **Node.js runtime:** Node.js 18+ recommended.
- **SQL fundamentals:** SELECT, INSERT, UPDATE, DELETE, and JOIN.
- **Database concepts:** Tables, indexes, transactions, and connection limits.
- **Node.js async programming:** Promises, async/await, and event loop concepts.
- **Basic security knowledge:** SQL injection, TLS, and credential management.

### Related Programming Areas

- **Relational Databases:** PostgreSQL, MySQL, MariaDB, and SQL Server.
- **ORMs and Query Builders:** Prisma, TypeORM, Sequelize, Knex, and Drizzle.
- **Serverless Computing:** AWS Lambda, Vercel, Cloudflare Workers.
- **Security:** SQL injection prevention, TLS configuration, and credential management.
- **Performance:** Connection pooling, query optimisation, and prepared statements.

### Core Concepts

1. **Database Drivers** — native vs. pure JavaScript, and edge-runtime compatible drivers.
2. **Connection Configuration** — environment variables, connection strings, and SSL/TLS.
3. **Connection Pooling** — pool sizing, exhaustion, idle timeouts, and serverless architectures.
4. **Prepared Statements** — performance benefits and thread safety.
5. **Parameterised Queries** — SQL injection prevention and complex data handling.

---

## Core Concept 1: Database Drivers

### Sub-Feature 1.1: Native vs. Pure JavaScript Drivers (e.g., `pg` vs. `postgres.js`)

#### Definitions

**Core Definition:** Database drivers are libraries that implement the wire protocol for a specific database, enabling Node.js applications to send queries and receive results. They are categorised as native (using C++ bindings) or pure JavaScript.

**Technical Definition:** Native drivers use C++ addons (via N-API or node-gyp) to interface directly with the database's client library, offering higher performance and access to low-level features. Pure JavaScript drivers implement the wire protocol entirely in JavaScript, offering portability, easier installation, and better compatibility with serverless and edge environments. For PostgreSQL, `pg` is a pure JavaScript driver with optional native bindings (`pg-native`), while `postgres.js` is a modern pure JavaScript driver with tagged template literals.

**Beginner-Friendly Explanation:** Native drivers are like a high-performance sports car — they're fast but need special fuel (native dependencies). Pure JavaScript drivers are like an electric car — slightly less raw power but much easier to maintain and run anywhere.

#### Purposes

- To provide a standard interface for communicating with a database.
- To abstract the wire protocol and connection management.
- To enable parameterised queries and prepared statements.
- To support connection pooling and transaction management.

#### Syntax Rules and Structure

**`pg` driver (PostgreSQL):**
```javascript
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const result = await pool.query('SELECT * FROM users WHERE id = $1', [42]);
```

**`postgres.js` driver (PostgreSQL):**
```javascript
const postgres = require('postgres');
const sql = postgres(process.env.DATABASE_URL);
const users = await sql`SELECT * FROM users WHERE id = ${42}`;
```

**`mysql2` driver (MySQL/MariaDB):**
```javascript
const mysql = require('mysql2/promise');
const pool = mysql.createPool({ /* config */ });
const [rows] = await pool.query('SELECT * FROM users WHERE id = ?', [42]);
```

| Driver | Database | Type | Package |
|--------|----------|------|---------|
| `pg` | PostgreSQL | Pure JS (+ optional native) | `pg` |
| `postgres.js` | PostgreSQL | Pure JS | `postgres` |
| `mysql2` | MySQL/MariaDB | Pure JS | `mysql2` |
| `mssql`/`tedious` | SQL Server | Pure JS | `mssql`, `tedious` |
| `better-sqlite3` | SQLite | Native | `better-sqlite3` |
| `sqlite3` | SQLite | Native | `sqlite3` |

**Constraints and Limitations:**
- Native drivers require compilation, which can fail on some platforms.
- Pure JavaScript drivers may have slightly lower throughput.
- `pg` uses `$1, $2` placeholders; `mysql2` uses `?`; `postgres.js` uses tagged templates.

#### Annotated Code Example

```javascript
// pg-vs-postgresjs.js
// Using pg
const { Pool } = require('pg');
const pgPool = new Pool({ connectionString: process.env.DATABASE_URL });

async function pgQuery() {
  const result = await pgPool.query(
    'SELECT id, name FROM users WHERE email = $1',
    ['alice@example.com']
  );
  console.log('pg:', result.rows[0]);
}

// Using postgres.js
const postgres = require('postgres');
const sql = postgres(process.env.DATABASE_URL);

async function postgresQuery() {
  const users = await sql`
    SELECT id, name FROM users WHERE email = ${'alice@example.com'}
  `;
  console.log('postgres.js:', users[0]);
}

(async () => {
  await pgQuery();
  await postgresQuery();
  await pgPool.end();
  await sql.end();
})();
```

**Expected Output:**
```
pg: { id: 1, name: 'Alice' }
postgres.js: { id: 1, name: 'Alice' }
```

**Why this output:** Both drivers execute the same query against PostgreSQL. `pg` uses numbered placeholders (`$1`); `postgres.js` uses tagged template literals with automatic parameterisation. Both return the same result.

#### Real-World Cases

- **`pg`:** Traditional Node.js applications, mature ecosystems, full control.
- **`postgres.js`:** Modern applications with tagged templates, edge compatibility.
- **`mysql2`:** MySQL and MariaDB applications, high concurrency.
- **`better-sqlite3`:** Embedded databases, desktop applications, testing.

---

### Sub-Feature 1.2: Edge-Runtime Compatible Drivers (HTTP/WebSocket-Based Database Connections)

#### Definitions

**Core Definition:** Edge-runtime compatible drivers connect to databases over HTTP or WebSocket instead of TCP, enabling database access from environments like Cloudflare Workers, Vercel Edge Functions, and Deno Deploy.

**Technical Definition:** Edge runtimes do not provide raw TCP sockets, so traditional drivers (which use TCP) cannot run there. Edge-compatible drivers communicate over HTTP or WebSocket APIs, which are available in edge runtimes. Examples include Neon's serverless driver (`@neondatabase/serverless`), PlanetScale's `@planetscale/database`, and Prisma Accelerate. These drivers proxy SQL over HTTP, often with connection pooling managed by the provider.

**Beginner-Friendly Explanation:** Edge runtimes are like small kiosks at the edge of the network. They can't make phone calls (TCP) but they can send letters (HTTP). Edge-compatible drivers bundle the database request into an HTTP request, send it to a proxy, and get the results back.

#### Purposes

- To enable database access from edge runtimes (Cloudflare Workers, Vercel Edge).
- To avoid TCP connection limits in serverless environments.
- To leverage provider-managed connection pooling.
- To reduce cold start times in serverless functions.

#### Syntax Rules and Structure

**Neon serverless driver:**
```javascript
import { neon } from '@neondatabase/serverless';
const sql = neon(process.env.DATABASE_URL);
const users = await sql`SELECT * FROM users WHERE id = ${42}`;
```

**PlanetScale serverless driver:**
```javascript
import { connect } from '@planetscale/database';
const conn = connect({ url: process.env.DATABASE_URL });
const results = await conn.execute('SELECT * FROM users WHERE id = ?', [42]);
```

**Prisma Accelerate:**
```javascript
import { PrismaClient } from '@prisma/client';
import { withAccelerate } from '@prisma/extension-accelerate';

const prisma = new PrismaClient().$extends(withAccelerate());
const users = await prisma.user.findMany({
  cacheStrategy: { ttl: 60 },
});
```

| Driver | Provider | Protocol |
|--------|----------|----------|
| `@neondatabase/serverless` | Neon | HTTP/WebSocket |
| `@planetscale/database` | PlanetScale | HTTP |
| Prisma Accelerate | Prisma | HTTP |
| `@vercel/postgres` | Vercel | HTTP |

**Constraints and Limitations:**
- HTTP-based drivers may have higher latency per query than TCP drivers.
- Transactions over HTTP may require special handling.
- Edge drivers are provider-specific.
- WebSocket connections may be needed for session-based features.

#### Annotated Code Example

```javascript
// edge-driver.js
// Cloudflare Worker with Neon serverless driver
import { neon } from '@neondatabase/serverless';

export default {
  async fetch(request, env) {
    const sql = neon(env.DATABASE_URL);

    const url = new URL(request.url);
    const userId = url.searchParams.get('id');

    const users = await sql`
      SELECT id, name, email FROM users WHERE id = ${userId}
    `;

    return new Response(JSON.stringify(users), {
      headers: { 'Content-Type': 'application/json' },
    });
  },
};
```

**Expected Output (for `GET /?id=42`):**
```json
[{"id":42,"name":"Alice","email":"alice@example.com"}]
```

**Why this output:** The Neon serverless driver sends the SQL query over HTTP to Neon's proxy, which executes it against PostgreSQL and returns the results. No TCP connection is needed, making it compatible with Cloudflare Workers.

#### Real-World Cases

- **Cloudflare Workers:** API endpoints running at the edge with database access.
- **Vercel Edge Functions:** Low-latency APIs with Neon or PlanetScale.
- **Deno Deploy:** Edge applications with HTTP-based database connections.

---

## Core Concept 2: Connection Configuration

### Sub-Feature 2.1: Environment Variable Management and Secure Connection Strings

#### Definitions

**Core Definition:** Connection configuration involves storing database credentials and connection parameters in environment variables and constructing secure connection strings that are never hardcoded in source code.

**Technical Definition:** A connection string (or DSN — Data Source Name) is a URI containing all parameters needed to connect to a database: protocol, username, password, host, port, database name, and options. Environment variables (loaded via `process.env` or `.env` files) keep credentials out of source control. Connection strings should be validated at startup, and secrets should be managed via a secrets manager (AWS Secrets Manager, HashiCorp Vault) in production.

**Beginner-Friendly Explanation:** A connection string is like a full address for the database, including the password. You never write it down in your code (where it could be committed to Git); instead, you store it in a secure place (environment variables) and read it at runtime.

#### Purposes

- To keep credentials out of source code and version control.
- To enable different configurations for development, staging, and production.
- To support secret rotation without code changes.
- To validate connection parameters at startup.

#### Syntax Rules and Structure

**Connection string format (PostgreSQL):**
```
postgresql://username:password@host:port/database?sslmode=require
```

**Environment variables (`.env`):**
```
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb
DB_HOST=localhost
DB_PORT=5432
DB_USER=myuser
DB_PASSWORD=mypassword
DB_NAME=mydb
```

**Loading and validating:**
```javascript
require('dotenv').config();
const { z } = require('zod');

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  NODE_ENV: z.enum(['development', 'production', 'test']),
});

const env = envSchema.parse(process.env);
```

| Component | Description | Example |
|-----------|-------------|---------|
| Protocol | Database type | `postgresql://` |
| Username | Database user | `user` |
| Password | Database password | `pass` |
| Host | Server address | `localhost` |
| Port | Server port | `5432` |
| Database | Database name | `mydb` |
| Query params | Options | `?sslmode=require` |

**Constraints and Limitations:**
- `.env` files must never be committed to version control.
- Connection strings may contain special characters that require URL encoding.
- Secrets should be rotated regularly.
- Environment variables are visible to child processes.

#### Annotated Code Example

```javascript
// connection-config.js
require('dotenv').config();
const { Pool } = require('pg');
const { z } = require('zod');

// Validate environment variables at startup
const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  PGSSLMODE: z.enum(['disable', 'require', 'verify-full']).default('require'),
});

const env = envSchema.parse(process.env);

// Create pool with validated config
const pool = new Pool({
  connectionString: env.DATABASE_URL,
  ssl: env.PGSSLMODE === 'disable' ? false : { rejectUnauthorized: env.PGSSLMODE === 'verify-full' },
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Health check
async function checkConnection() {
  const client = await pool.connect();
  try {
    const result = await client.query('SELECT NOW() AS now');
    console.log('Connected at:', result.rows[0].now);
  } finally {
    client.release();
  }
}

checkConnection().catch(console.error);
```

**Expected Output:**
```
Connected at: 2026-01-15T12:00:00.000Z
```

**Why this output:** The `dotenv` package loads `.env` into `process.env`. Zod validates that `DATABASE_URL` is a valid URL and `PGSSLMODE` is one of the allowed values. If validation fails, the application exits with a clear error before attempting to connect.

#### Real-World Cases

- **Twelve-Factor Apps:** Storing config in environment variables.
- **Container deployments:** Injecting secrets via Kubernetes secrets or Docker secrets.
- **Cloud deployments:** Using AWS Secrets Manager, Azure Key Vault, or GCP Secret Manager.

---

### Sub-Feature 2.2: SSL/TLS Configuration and Certificate Authority (CA) Setup

#### Definitions

**Core Definition:** SSL/TLS configuration encrypts database traffic and verifies the server's identity using certificates signed by a trusted Certificate Authority (CA).

**Technical Definition:** PostgreSQL, MySQL, and SQL Server support TLS-encrypted connections. The client presents a CA certificate (or uses the system's trust store) to verify the server's certificate. Mutual TLS (mTLS) additionally requires the client to present its own certificate. TLS modes include `disable` (no encryption), `require` (encryption without verification), `verify-ca` (verify server certificate against CA), and `verify-full` (verify CA and hostname).

**Beginner-Friendly Explanation:** TLS is like sealing your database conversations in an envelope that only the server can open. The CA certificate is like a notarised ID that proves the server is who it claims to be. Without verification, you're encrypting your data but might be sending it to an impostor.

#### Purposes

- To encrypt database traffic against eavesdropping.
- To verify the database server's identity.
- To comply with security standards (PCI DSS, HIPAA, GDPR).
- To enable mutual authentication (mTLS) between application and database.

#### Syntax Rules and Structure

**PostgreSQL SSL modes:**
| Mode | Encryption | Server Cert | Hostname |
|------|-----------|-------------|----------|
| `disable` | No | No | No |
| `allow` | Preferred | No | No |
| `prefer` | Preferred | No | No |
| `require` | Yes | No | No |
| `verify-ca` | Yes | Yes | No |
| `verify-full` | Yes | Yes | Yes |

**Node.js pg SSL configuration:**
```javascript
const fs = require('fs');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: {
    rejectUnauthorized: true,
    ca: fs.readFileSync('./certs/ca-certificate.crt').toString(),
    cert: fs.readFileSync('./certs/client-certificate.crt').toString(),
    key: fs.readFileSync('./certs/client-key.key').toString(),
  },
});
```

**MySQL SSL configuration:**
```javascript
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  ssl: {
    ca: fs.readFileSync('./certs/ca.pem'),
    cert: fs.readFileSync('./certs/client-cert.pem'),
    key: fs.readFileSync('./certs/client-key.pem'),
    rejectUnauthorized: true,
  },
});
```

**Constraints and Limitations:**
- `rejectUnauthorized: false` disables certificate validation and is a security risk.
- Managed databases (AWS RDS, Azure Database) provide CA certificates for download.
- Certificate rotation requires updating the CA file and restarting the pool.
- mTLS requires configuring the database to request client certificates.

#### Annotated Code Example

```javascript
// ssl-config.js
const fs = require('fs');
const { Pool } = require('pg');

const isProduction = process.env.NODE_ENV === 'production';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: isProduction ? {
    rejectUnauthorized: true,
    ca: fs.readFileSync(process.env.DB_CA_CERT_PATH, 'utf8'),
  } : false,
  max: 20,
  idleTimeoutMillis: 30000,
});

async function testSSL() {
  const client = await pool.connect();
  try {
    // Check SSL status
    const result = await client.query(`
      SELECT ssl, version, cipher
      FROM pg_stat_ssl
      WHERE pid = pg_backend_pid()
    `);
    console.log('SSL status:', result.rows[0]);
  } finally {
    client.release();
  }
}

testSSL().catch(console.error);
```

**Expected Output (in production):**
```
SSL status: { ssl: true, version: 'TLSv1.3', cipher: 'TLS_AES_256_GCM_SHA384' }
```

**Why this output:** In production, the pool is configured with `rejectUnauthorized: true` and the CA certificate. The `pg_stat_ssl` view confirms the connection is encrypted with TLS 1.3 and a strong cipher. In development, SSL is disabled for simplicity.

#### Real-World Cases

- **Cloud databases:** AWS RDS, Azure Database, and GCP Cloud SQL require TLS by default.
- **Compliance:** PCI DSS, HIPAA, and GDPR mandate encryption in transit.
- **Multi-region deployments:** TLS protects data crossing public networks.

---

## Core Concept 3: Connection Pooling

### Sub-Feature 3.1: Pool Size Optimization Relative to Node Cluster Instances

#### Definitions

**Core Definition:** Connection pool sizing determines how many database connections each Node.js process maintains, and must be coordinated with the number of cluster instances to avoid exceeding the database's connection limit.

**Technical Definition:** The total number of database connections is `pool_size × number_of_instances`. PostgreSQL's default `max_connections` is 100; MySQL's is 151. If the application has 4 cluster workers and each has a pool of 20, that's 80 connections — within limits. But if the application scales to 10 pods, that's 200 connections — exceeding the limit. The formula is: `pool_size = (max_connections - reserved_connections) / number_of_instances`.

**Beginner-Friendly Explanation:** Connection pooling is like having a limited number of phone lines to the database. Each Node.js process (cluster worker or pod) gets some lines. If you have 4 workers and each gets 20 lines, that's 80 lines total. If the database only supports 100 lines, you're fine. But if you add more workers, you'll run out of lines.

#### Purposes

- To prevent connection exhaustion in clustered and containerised deployments.
- To balance concurrency against database resource limits.
- To avoid "too many connections" errors under load.
- To right-size infrastructure costs.

#### Syntax Rules and Structure

```javascript
const os = require('os');
const { Pool } = require('pg');

// Calculate pool size based on instance count
const INSTANCE_COUNT = parseInt(process.env.INSTANCE_COUNT || '1', 10);
const DB_MAX_CONNECTIONS = parseInt(process.env.DB_MAX_CONNECTIONS || '100', 10);
const RESERVED = 10; // For admin, migrations, monitoring

const poolSize = Math.floor((DB_MAX_CONNECTIONS - RESERVED) / INSTANCE_COUNT);

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: Math.max(poolSize, 2), // At least 2 connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 5000,
});

console.log(`Pool size: ${pool.options.max} (instances: ${INSTANCE_COUNT})`);
```

| Factor | Consideration |
|--------|---------------|
| DB `max_connections` | Hard limit (100 for PG, 151 for MySQL). |
| Instance count | Cluster workers, pods, or serverless functions. |
| Reserved connections | Superuser, monitoring, migrations. |
| Workload type | OLTP (more connections) vs. OLAP (fewer, longer queries). |

**Constraints and Limitations:**
- Pool size must be `>= 2` for transaction safety.
- Serverless functions scale unpredictably; use a proxy (PgBouncer, Prisma Accelerate).
- Long-running queries hold connections; monitor pool saturation.

#### Annotated Code Example

```javascript
// pool-sizing.js
const cluster = require('node:cluster');
const os = require('node:os');
const { Pool } = require('pg');

const DB_MAX_CONNECTIONS = 100;
const RESERVED = 10;

function createPool() {
  const instanceCount = cluster.isWorker
    ? os.availableParallelism()
    : 1;

  const poolSize = Math.max(
    Math.floor((DB_MAX_CONNECTIONS - RESERVED) / instanceCount),
    2
  );

  return new Pool({
    connectionString: process.env.DATABASE_URL,
    max: poolSize,
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 5000,
  });
}

if (cluster.isPrimary) {
  for (let i = 0; i < os.availableParallelism(); i++) {
    cluster.fork();
  }
} else {
  const pool = createPool();
  console.log(`Worker ${process.pid}: pool size = ${pool.options.max}`);
}
```

**Expected Output (8-core machine):**
```
Worker 12346: pool size = 11
Worker 12347: pool size = 11
...
```
*(Total: 8 workers × 11 = 88 connections, within the 100 limit.)*

**Why this output:** The pool size is calculated as `(100 - 10) / 8 = 11.25`, floored to 11. With 8 workers, total connections are 88, leaving 12 connections reserved for admin tasks.

#### Real-World Cases

- **Kubernetes deployments:** Adjusting pool size based on replica count.
- **Cluster mode:** Distributing connections across CPU cores.
- **Serverless:** Using PgBouncer to multiplex thousands of functions into a few database connections.

---

### Sub-Feature 3.2: Managing Connection Exhaustion and Idle Timeout Handling

#### Definitions

**Core Definition:** Connection exhaustion occurs when all pool connections are in use and new requests must wait; idle timeout handling closes unused connections after a period to free resources.

**Technical Definition:** The `pg` Pool has `connectionTimeoutMillis` (how long to wait for a connection) and `idleTimeoutMillis` (how long an idle connection stays in the pool). When the pool is exhausted, requests queue until a connection is released. If the queue wait exceeds `connectionTimeoutMillis`, the request fails with a timeout error. Monitoring pool metrics (`totalCount`, `idleCount`, `waitingCount`) is essential for detecting exhaustion.

**Beginner-Friendly Explanation:** Connection exhaustion is like a busy restaurant with all tables full — new customers wait in line. If the line gets too long, some customers leave (timeout). Idle timeout is like closing tables that have been empty for a while to save space.

#### Purposes

- To detect and prevent connection pool exhaustion.
- To release idle connections and reduce database load.
- To provide graceful degradation when the pool is saturated.
- To monitor pool health for capacity planning.

#### Syntax Rules and Structure

```javascript
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000,        // Close idle after 30s
  connectionTimeoutMillis: 5000,    // Fail if wait > 5s
  maxUses: 7500,                    // Recycle connection after 7500 uses
});

// Monitor pool events
pool.on('connect', () => console.log('New connection established'));
pool.on('acquire', () => console.log('Connection acquired'));
pool.on('release', () => console.log('Connection released'));
pool.on('remove', () => console.log('Connection removed'));
pool.on('error', (err) => console.error('Pool error:', err));

// Health check
function poolMetrics() {
  return {
    total: pool.totalCount,
    idle: pool.idleCount,
    waiting: pool.waitingCount,
  };
}
```

| Event | Description |
|-------|-------------|
| `connect` | New connection established. |
| `acquire` | Connection acquired from pool. |
| `release` | Connection returned to pool. |
| `remove` | Connection removed from pool. |
| `error` | Pool-level error. |

**Constraints and Limitations:**
- `connectionTimeoutMillis` too low causes false failures under load.
- `idleTimeoutMillis` too high holds connections unnecessarily.
- Long-running queries block connections; use statement timeouts.

#### Annotated Code Example

```javascript
// pool-exhaustion.js
const express = require('express');
const { Pool } = require('pg');
const app = express();

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 5,                          // Small pool for demonstration
  idleTimeoutMillis: 10000,
  connectionTimeoutMillis: 2000,
});

// Pool monitoring endpoint
app.get('/health/pool', (req, res) => {
  res.json({
    total: pool.totalCount,
    idle: pool.idleCount,
    waiting: pool.waitingCount,
  });
});

// Slow query that holds a connection
app.get('/slow', async (req, res) => {
  const client = await pool.connect();
  try {
    await client.query('SELECT pg_sleep(3)');
    res.json({ done: true });
  } finally {
    client.release();
  }
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output (for `/health/pool` under load):**
```json
{"total":5,"idle":0,"waiting":3}
```

**Why this output:** All 5 connections are in use (`idle: 0`), and 3 requests are waiting for a connection (`waiting: 3`). If the wait exceeds 2 seconds (`connectionTimeoutMillis`), those requests fail with a timeout error. Monitoring the pool helps detect exhaustion before it causes failures.

#### Real-World Cases

- **High-traffic APIs:** Monitoring pool metrics to detect bottlenecks.
- **Long-running reports:** Using separate pools for OLTP and OLAP workloads.
- **Graceful degradation:** Returning 503 when the pool is exhausted.

---

### Sub-Feature 3.3: Serverless Connection Pooling Architectures (PgBouncer, Prisma Accelerate)

#### Definitions

**Core Definition:** Serverless connection pooling uses a proxy or gateway to multiplex thousands of short-lived serverless function invocations into a small number of persistent database connections.

**Technical Definition:** Serverless functions (AWS Lambda, Vercel, Cloudflare Workers) can scale to thousands of concurrent instances, each opening database connections. This exhausts the database's `max_connections`. Solutions include PgBouncer (a lightweight connection pooler that sits between the application and PostgreSQL), Prisma Accelerate (a managed connection pool and cache), Neon's serverless driver (HTTP/WebSocket proxy), and AWS RDS Proxy (a managed proxy for RDS). PgBouncer supports transaction pooling mode, where a connection is assigned to a transaction and returned to the pool when the transaction completes.

**Beginner-Friendly Explanation:** Serverless functions are like thousands of people calling a restaurant at once. The restaurant only has 100 tables. PgBouncer is like a reservation system that takes all the calls and assigns tables as they become free, so no one is turned away and the restaurant doesn't need more tables.

#### Purposes

- To prevent connection exhaustion from serverless function scaling.
- To reduce database load from connection establishment overhead.
- To enable transaction pooling for multi-tenant serverless applications.
- To provide a managed solution without infrastructure management.

#### Syntax Rules and Structure

**PgBouncer connection:**
```javascript
// Application connects to PgBouncer (port 6432) instead of PostgreSQL (5432)
const pool = new Pool({
  connectionString: 'postgresql://user:pass@pgbouncer-host:6432/mydb',
  max: 10,
});
```

**Prisma Accelerate:**
```javascript
import { PrismaClient } from '@prisma/client';
import { withAccelerate } from '@prisma/extension-accelerate';

const prisma = new PrismaClient().$extends(withAccelerate());

const users = await prisma.user.findMany({
  cacheStrategy: { ttl: 60, swr: 120 },
});
```

**Neon serverless driver:**
```javascript
import { neon } from '@neondatabase/serverless';
const sql = neon(process.env.DATABASE_URL);
const users = await sql`SELECT * FROM users`;
```

| Solution | Type | Pooling Mode | Best For |
|----------|------|--------------|----------|
| PgBouncer | Self-hosted | Transaction/Session/Statement | Full control, self-managed |
| Prisma Accelerate | Managed | Managed + cache | Prisma users, global caching |
| Neon Serverless | Managed | HTTP proxy | Edge, serverless, Neon |
| AWS RDS Proxy | Managed | Managed | AWS RDS, IAM auth |

**Constraints and Limitations:**
- Transaction pooling mode does not support session-level features (prepared statements, advisory locks, `SET` commands).
- PgBouncer requires separate infrastructure and monitoring.
- Prisma Accelerate is a paid service with usage limits.
- Serverless drivers may not support all PostgreSQL features.

#### Annotated Code Example

```javascript
// serverless-pooling.js
// AWS Lambda with Prisma Accelerate
import { PrismaClient } from '@prisma/client';
import { withAccelerate } from '@prisma/extension-accelerate';

// Create Prisma client with Accelerate extension
const prisma = new PrismaClient().$extends(withAccelerate());

export const handler = async (event) => {
  const userId = event.pathParameters?.id;

  try {
    // This query is pooled by Accelerate and cached for 60s
    const user = await prisma.user.findUnique({
      where: { id: userId },
      cacheStrategy: { ttl: 60, swr: 120 },
    });

    if (!user) {
      return {
        statusCode: 404,
        body: JSON.stringify({ error: 'User not found' }),
      };
    }

    return {
      statusCode: 200,
      body: JSON.stringify(user),
    };
  } catch (err) {
    console.error('Database error:', err);
    return {
      statusCode: 500,
      body: JSON.stringify({ error: 'Internal Server Error' }),
    };
  }
};
```

**Expected Output (for `GET /users/42`):**
```json
{"id":"42","name":"Alice","email":"alice@example.com"}
```

**Why this output:** Prisma Accelerate manages the connection pool externally. The Lambda function connects to Accelerate over HTTP, and Accelerate maintains persistent connections to PostgreSQL. The `cacheStrategy` option caches the result for 60 seconds, reducing database load for repeated queries.

#### Real-World Cases

- **AWS Lambda:** Using RDS Proxy or Prisma Accelerate for PostgreSQL.
- **Vercel Functions:** Using Neon serverless driver or Prisma Accelerate.
- **Cloudflare Workers:** Using Neon or PlanetScale HTTP drivers.
- **Multi-tenant SaaS:** PgBouncer transaction pooling for tenant isolation.

---

## Core Concept 4: Prepared Statements

### Sub-Feature 4.1: Performance Benefits Through Query Plan Caching

#### Definitions

**Core Definition:** Prepared statements are pre-compiled SQL statements that the database parses, plans, and optimises once, then executes multiple times with different parameters, reusing the cached query plan.

**Technical Definition:** When a prepared statement is created, the database parses the SQL, generates an execution plan, and stores it. Subsequent executions with different parameters reuse the plan, skipping parsing and planning overhead. This is especially beneficial for complex queries (multiple joins, subqueries) executed frequently. In PostgreSQL, prepared statements are session-scoped; in MySQL, they are also session-scoped. The `pg` driver automatically prepares statements when you use parameterised queries. The `mysql2` driver caches prepared statements per connection.

**Beginner-Friendly Explanation:** A prepared statement is like a recipe that's already been memorised by the chef. The first time, the chef reads the recipe and cooks the dish. The next time, the chef already knows the steps and just uses different ingredients. It's much faster.

#### Purposes

- To reduce query parsing and planning overhead.
- To improve performance for frequently executed queries.
- To provide automatic SQL injection protection.
- To enable query plan reuse across executions.

#### Syntax Rules and Structure

**PostgreSQL (pg):**
```javascript
// pg automatically prepares parameterised queries
const result = await pool.query(
  'SELECT * FROM users WHERE email = $1 AND status = $2',
  ['alice@example.com', 'active']
);
```

**MySQL (mysql2):**
```javascript
// Use execute() for prepared statements
const [rows] = await pool.execute(
  'SELECT * FROM users WHERE email = ? AND status = ?',
  ['alice@example.com', 'active']
);
```

**Explicit prepare (pg):**
```javascript
// Named prepared statement (per connection)
const client = await pool.connect();
await client.query({
  name: 'get-user-by-email',
  text: 'SELECT * FROM users WHERE email = $1',
  values: ['alice@example.com'],
});
```

| Aspect | `query()` | `execute()` (mysql2) | `pg` parameterised |
|--------|-----------|---------------------|-------------------|
| Prepared | No (pg prepares automatically) | Yes | Yes (after first use) |
| Cached | N/A | Per connection | Per connection |
| Injection safe | Yes (parameterised) | Yes | Yes |
| Best for | Simple queries | Repeated queries | Repeated queries |

**Constraints and Limitations:**
- Prepared statements are per-connection; pool connections each maintain their own cache.
- PostgreSQL prepared statements cannot be used in transaction pooling mode with PgBouncer.
- Very large numbers of distinct prepared statements consume memory.
- Prepared statements cannot parameterise table names or column names.

#### Annotated Code Example

```javascript
// prepared-statements.js
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

async function benchmark() {
  const client = await pool.connect();
  try {
    // Warm up: first execution prepares the statement
    await client.query('SELECT * FROM users WHERE email = $1', ['test@example.com']);

    // Benchmark: subsequent executions use the cached plan
    const iterations = 1000;
    const start = Date.now();

    for (let i = 0; i < iterations; i++) {
      await client.query('SELECT * FROM users WHERE email = $1', [`user${i}@example.com`]);
    }

    const elapsed = Date.now() - start;
    console.log(`${iterations} queries in ${elapsed}ms (${(elapsed / iterations).toFixed(2)}ms/query)`);
  } finally {
    client.release();
  }
  await pool.end();
}

benchmark().catch(console.error);
```

**Expected Output:**
```
1000 queries in 450ms (0.45ms/query)
```

**Why this output:** The first query prepares the statement and caches the plan. Subsequent queries reuse the cached plan, resulting in ~0.45ms per query (dominated by round-trip latency). Without prepared statements, each query would require parsing and planning, adding overhead.

#### Real-World Cases

- **High-frequency queries:** User lookups, session validation, and configuration reads.
- **Complex reports:** Multi-join queries executed repeatedly with different filters.
- **API endpoints:** Queries executed on every request.

---

### Sub-Feature 4.2: Thread Safety Implications in Asynchronous Node Loops

#### Definitions

**Core Definition:** Prepared statements in Node.js are session-scoped and not thread-safe across connections; each pool connection maintains its own prepared statement cache.

**Technical Definition:** Node.js is single-threaded for JavaScript execution, but the `pg` and `mysql2` drivers use libuv's threadpool for DNS and crypto operations. Prepared statements are bound to a specific connection (session), not to the pool. When a query is executed, the driver acquires a connection from the pool, uses its prepared statement cache, and releases it. If the pool has 20 connections, there may be up to 20 copies of the same prepared statement. This is safe but may increase memory usage. Explicit named prepared statements must be used on the same connection that prepared them.

**Beginner-Friendly Explanation:** Prepared statements are like notes written on a specific phone line. If you call from a different line (connection), your note isn't there. In Node.js, the pool manages which line you get, so each line learns the note separately. This is safe but means each connection has its own copy.

#### Purposes

- To understand the scope and lifecycle of prepared statements in pooled environments.
- To avoid errors when using explicit named prepared statements.
- To optimise prepared statement usage in high-concurrency applications.

#### Syntax Rules and Structure

```javascript
// ❌ Wrong: explicit named statement on a different connection
const client1 = await pool.connect();
await client1.query({ name: 'my-stmt', text: 'SELECT ...' });
client1.release();

const client2 = await pool.connect();
await client2.query({ name: 'my-stmt' }); // Error: statement not found
client2.release();

// ✅ Correct: use the same connection
const client = await pool.connect();
try {
  await client.query({ name: 'my-stmt', text: 'SELECT ...' });
  await client.query({ name: 'my-stmt' }); // Works
} finally {
  client.release();
}
```

**Constraints and Limitations:**
- Explicit named prepared statements must be executed on the same connection.
- Pool connections may be recycled; prepared statements are lost when a connection is closed.
- `maxUses` option recycles connections, discarding prepared statements.

#### Annotated Code Example

```javascript
// prepared-thread-safety.js
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL, max: 5 });

async function unsafeExample() {
  // ❌ Named statement on connection A
  const clientA = await pool.connect();
  await clientA.query({ name: 'get-user', text: 'SELECT * FROM users WHERE id = $1', values: [1] });
  clientA.release();

  // Connection B may not have the prepared statement
  const clientB = await pool.connect();
  try {
    await clientB.query({ name: 'get-user' }); // May fail
  } catch (err) {
    console.error('Error:', err.message); // "prepared statement 'get-user' does not exist"
  } finally {
    clientB.release();
  }
}

async function safeExample() {
  // ✅ Use parameterised queries (auto-prepared per connection)
  const result = await pool.query('SELECT * FROM users WHERE id = $1', [1]);
  console.log('User:', result.rows[0]);
}

(async () => {
  await safeExample();
  await pool.end();
})();
```

**Expected Output:**
```
User: { id: 1, name: 'Alice' }
```

**Why this output:** The safe example uses parameterised queries via `pool.query()`, which automatically prepares the statement on whichever connection is acquired. The unsafe example shows that explicit named statements may not exist on a different connection.

#### Real-World Cases

- **High-concurrency APIs:** Using parameterised queries instead of named prepared statements.
- **Transaction blocks:** Using the same connection for all queries in a transaction.
- **PgBouncer transaction mode:** Avoiding prepared statements because they are not supported.

---

## Core Concept 5: Parameterised Queries

### Sub-Feature 5.1: Preventing SQL Injection Attacks at the Driver Level

#### Definitions

**Core Definition:** Parameterised queries (also called prepared statements with parameters) separate SQL code from data, sending parameters separately so they are never interpreted as SQL.

**Technical Definition:** In a parameterised query, the SQL statement contains placeholders (`$1`, `?`, or `:name`) instead of literal values. The driver sends the SQL template and the parameter values separately to the database. The database executes the query with the parameters bound to the placeholders, never interpreting them as SQL code. This prevents SQL injection because the parameter values cannot alter the query structure.

**Beginner-Friendly Explanation:** Parameterised queries are like a fill-in-the-blank form. The form's structure (the SQL) is fixed; you only fill in the blanks (the parameters). An attacker can't change the form — they can only provide values for the blanks, and those values are treated as data, not code.

#### Purposes

- To prevent SQL injection attacks at the protocol level.
- To eliminate the need for manual escaping.
- To improve performance through query plan reuse.
- To handle data types correctly (strings, numbers, dates, arrays).

#### Syntax Rules and Structure

**PostgreSQL (pg):**
```javascript
// ✅ Parameterised
await pool.query('SELECT * FROM users WHERE email = $1', [email]);

// ❌ Vulnerable: string concatenation
await pool.query(`SELECT * FROM users WHERE email = '${email}'`);
```

**MySQL (mysql2):**
```javascript
// ✅ Parameterised
await pool.query('SELECT * FROM users WHERE email = ?', [email]);

// ❌ Vulnerable
await pool.query(`SELECT * FROM users WHERE email = '${email}'`);
```

**SQL Server (mssql):**
```javascript
// ✅ Parameterised
await pool.request()
  .input('email', sql.NVarChar, email)
  .query('SELECT * FROM users WHERE email = @email');
```

| Database | Placeholder | Example |
|----------|-------------|---------|
| PostgreSQL | `$1, $2` | `WHERE id = $1 AND name = $2` |
| MySQL | `?` | `WHERE id = ? AND name = ?` |
| SQL Server | `@name` | `WHERE id = @id AND name = @name` |
| SQLite | `?` or `:name` | `WHERE id = ?` |

**Constraints and Limitations:**
- Placeholders cannot represent table names, column names, or SQL keywords.
- Some databases limit the number of parameters per query.
- Arrays and complex types require driver-specific handling.

#### Annotated Code Example

```javascript
// parameterised-queries.js
const express = require('express');
const { Pool } = require('pg');
const app = express();
app.use(express.json());

const pool = new Pool({ connectionString: process.env.DATABASE_URL });

// ✅ Safe: parameterised query
app.get('/users/search', async (req, res) => {
  const { email } = req.query;

  // Validate type
  if (typeof email !== 'string') {
    return res.status(400).json({ error: 'Invalid email parameter' });
  }

  const result = await pool.query(
    'SELECT id, name, email FROM users WHERE email = $1',
    [email]
  );

  res.json({ data: result.rows });
});

// ❌ Vulnerable (DO NOT USE): string concatenation
app.get('/users/unsafe', async (req, res) => {
  const { email } = req.query;
  // Attacker sends: ' OR '1'='1
  const result = await pool.query(
    `SELECT id, name, email FROM users WHERE email = '${email}'`
  );
  res.json({ data: result.rows });
});

app.listen(3000);
```

**Expected Output (for `GET /users/search?email=' OR '1'='1`):**
```json
{"data":[]}
```
*(The parameterised query treats the entire input as a literal string, returning no results.)*

**Expected Output (for `GET /users/unsafe?email=' OR '1'='1`):**
```json
{"data":[{"id":1,"name":"Alice","email":"alice@example.com"}, ...]}
```
*(The vulnerable query returns ALL users because the injection altered the SQL.)*

**Why this output:** The parameterised query binds the malicious input as a string value, so the `WHERE` clause compares `email = "' OR '1'='1"` — which matches no user. The vulnerable query concatenates the input into the SQL, producing `WHERE email = '' OR '1'='1'`, which is always true.

#### Real-World Cases

- **Login forms:** Parameterised queries for email/password validation.
- **Search endpoints:** Parameterised queries for user-supplied search terms.
- **Filters:** Parameterised queries for date ranges, categories, and tags.

---

### Sub-Feature 5.2: Handling Complex Data Arrays and Type Casting

#### Definitions

**Core Definition:** Handling complex data types (arrays, JSON, dates, UUIDs) in parameterised queries requires driver-specific type casting and sometimes explicit type annotations.

**Technical Definition:** PostgreSQL supports array types (`INT[]`, `TEXT[]`), JSON/JSONB, UUID, and custom types. The `pg` driver serialises JavaScript arrays into PostgreSQL array literals and deserialises them back. However, type inference can be ambiguous; explicit casts (`$1::int[]`) resolve ambiguity. MySQL does not support array parameters; arrays must be serialised to JSON or expanded into multiple placeholders. SQL Server uses table-valued parameters for arrays.

**Beginner-Friendly Explanation:** Databases have different ways of handling lists and complex data. PostgreSQL understands arrays natively, so you can pass `[1, 2, 3]` and it works. MySQL doesn't understand arrays, so you have to convert them to JSON strings. Each database has its own rules.

#### Purposes

- To pass arrays of values to `IN` clauses safely.
- To store and query JSON/JSONB data.
- To handle dates, UUIDs, and custom types correctly.
- To avoid type mismatch errors.

#### Syntax Rules and Structure

**PostgreSQL array parameter:**
```javascript
// Pass array directly (pg serialises it)
const result = await pool.query(
  'SELECT * FROM users WHERE id = ANY($1::int[])',
  [[1, 2, 3]]
);
```

**PostgreSQL JSONB parameter:**
```javascript
await pool.query(
  'INSERT INTO products (name, attributes) VALUES ($1, $2::jsonb)',
  ['Laptop', JSON.stringify({ brand: 'Dell', ram: 16 })]
);
```

**PostgreSQL UUID parameter:**
```javascript
await pool.query(
  'SELECT * FROM users WHERE id = $1::uuid',
  ['550e8400-e29b-41d4-a716-446655440000']
);
```

**MySQL array (JSON workaround):**
```javascript
await pool.query(
  'SELECT * FROM users WHERE JSON_CONTAINS(tags, ?)',
  [JSON.stringify(['admin'])]
);
```

| Database | Array Support | Approach |
|----------|--------------|----------|
| PostgreSQL | Native | `$1::int[]` or `ANY($1)` |
| MySQL | None | JSON serialisation or `IN (?, ?, ?)` |
| SQL Server | Table-valued parameters | `mssql` TVP |
| SQLite | None | JSON serialisation or multiple placeholders |

**Constraints and Limitations:**
- PostgreSQL arrays must be type-cast when the driver cannot infer the type.
- MySQL JSON functions require MySQL 5.7+.
- SQL Server TVPs require a user-defined table type.
- Large arrays may hit parameter limits.

#### Annotated Code Example

```javascript
// complex-parameters.js
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

async function examples() {
  // 1. Array parameter with ANY
  const userIds = [1, 2, 3];
  const users = await pool.query(
    'SELECT id, name FROM users WHERE id = ANY($1::int[])',
    [userIds]
  );
  console.log('Users by IDs:', users.rows);

  // 2. JSONB parameter
  const attributes = { brand: 'Dell', ram: 16, storage: '512GB SSD' };
  const product = await pool.query(
    'INSERT INTO products (name, attributes) VALUES ($1, $2::jsonb) RETURNING id, attributes',
    ['Laptop', JSON.stringify(attributes)]
  );
  console.log('Inserted product:', product.rows[0]);

  // 3. UUID parameter
  const uuid = '550e8400-e29b-41d4-a716-446655440000';
  const uuidResult = await pool.query(
    'SELECT $1::uuid AS id',
    [uuid]
  );
  console.log('UUID:', uuidResult.rows[0].id);

  // 4. Date parameter
  const date = new Date('2026-01-15T00:00:00Z');
  const dateResult = await pool.query(
    'SELECT * FROM orders WHERE created_at >= $1::timestamptz',
    [date]
  );
  console.log('Orders since:', dateResult.rows.length);

  await pool.end();
}

examples().catch(console.error);
```

**Expected Output:**
```
Users by IDs: [ { id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }, { id: 3, name: 'Charlie' } ]
Inserted product: { id: 1, attributes: { brand: 'Dell', ram: 16, storage: '512GB SSD' } }
UUID: 550e8400-e29b-41d4-a716-446655440000
Orders since: 5
```

**Why this output:** The array parameter `$1::int[]` is cast to an integer array, and `ANY()` checks if `id` is in the array. The JSONB parameter is stringified and cast to `jsonb`. The UUID is cast to `uuid`. The date is passed as a JavaScript Date and cast to `timestamptz`. Explicit casts resolve type ambiguity.

#### Real-World Cases

- **Bulk operations:** Fetching multiple users by ID array.
- **JSONB storage:** Storing product attributes, user preferences, or event metadata.
- **UUID primary keys:** Using UUIDs for distributed systems.
- **Date range filters:** Querying orders, logs, or events by date range.

---

## References

- node-postgres (pg) Documentation — https://node-postgres.com/
- postgres.js Documentation — https://github.com/porsager/postgres
- mysql2 Documentation — https://github.com/sidorares/node-mysql2
- mssql Documentation — https://www.npmjs.com/package/mssql
- tedious Documentation — https://github.com/tediousjs/tedious
- PostgreSQL Documentation — https://www.postgresql.org/docs/
- MySQL Documentation — https://dev.mysql.com/doc/
- Neon Serverless Driver — https://neon.tech/docs/serverless/serverless-driver
- PlanetScale Database Driver — https://github.com/planetscale/database-js
- Prisma Accelerate — https://www.prisma.io/accelerate
- PgBouncer Documentation — https://www.pgbouncer.org/
- AWS RDS Proxy — https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.html
- OWASP — SQL Injection Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- OWASP — Query Parameterization Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html
- Node.js Documentation — Environment Variables — https://nodejs.org/api/environment_variables.html
- The Twelve-Factor App — Config — https://12factor.net/config
- PostgreSQL — SSL Support — https://www.postgresql.org/docs/current/libpq-ssl.html
- PostgreSQL — Prepared Statements — https://www.postgresql.org/docs/current/sql-prepare.html
- PostgreSQL — Array Types — https://www.postgresql.org/docs/current/arrays.html
- PostgreSQL — JSON Types — https://www.postgresql.org/docs/current/datatype-json.html
- MySQL — Prepared Statements — https://dev.mysql.com/doc/refman/8.0/en/sql-prepared-statements.html
- Heroku — Best Practices in Error Handling — https://www.heroku.com/codeish-podcasts/51-best-practices-in-error-handling
- Building a Simple and Effective Error-Handling System in Node.js — https://tsecurity.de/de/2462968/
- pgvector — GitHub — https://github.com/pgvector/pgvector
- PostGIS — https://postgis.net/