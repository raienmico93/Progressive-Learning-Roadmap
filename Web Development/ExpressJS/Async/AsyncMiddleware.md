# Async Middleware & Network Operations — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Async middleware and network operations refer to the patterns and practices for performing non-blocking I/O — database queries, external API calls, file operations, and background job processing — within Express route handlers and middleware, using Promises and `async/await` to preserve the event loop's responsiveness.

**Technical Definition:** Node.js is single-threaded and uses an event loop with non-blocking I/O via libuv to handle concurrency without spawning a thread per request. Asynchronous middleware leverages this model by returning Promises that resolve when I/O completes, allowing the event loop to process other requests in the meantime. The four primary categories of network operations in Express applications are: **database operations** (connection pooling, transactions, timeouts), **external API calls** (HTTP/gRPC clients, retries, backoffs, circuit breakers), **file operations** (streams, `fs/promises`, `stream.pipeline`), and **background processing** (offloading long-running tasks to message queues like BullMQ or RabbitMQ). Express 5 natively catches rejected Promises in route handlers; Express 4 requires manual handling via `try/catch`, wrapper functions, or `express-async-errors`.

**Beginner-Friendly Explanation:** Imagine a restaurant kitchen with one chef (Node.js's single thread). The chef doesn't stand and wait for the oven to preheat — they put a dish in the oven (start a database query), and while it bakes, they chop vegetables (handle another request). This is how async middleware works: instead of blocking the entire server while waiting for a database query or API call, the server handles other requests. Async middleware and network operations are the patterns that let your Express app cook many dishes at once without burning any of them.

### Key Characteristics

- **Non-blocking I/O:** Database queries, HTTP requests, and file reads do not block the event loop.
- **Connection pooling:** Databases use pools to reuse connections across concurrent requests.
- **Resilience patterns:** External API calls use retries, exponential backoffs, and circuit breakers.
- **Memory efficiency:** Large files are processed via streams, not loaded entirely into RAM.
- **Task offloading:** Long-running operations are deferred to background queues.
- **Express version awareness:** Express 5 catches async errors natively; Express 4 requires wrappers.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x; v16.4+ for stable `AsyncLocalStorage`).
- **Express.js installed:** `npm install express`.
- **A database driver or ORM** (e.g., `pg`, `mysql2`, Sequelize, Prisma).
- **An HTTP client** (Axios or native `fetch`).
- **Basic JavaScript knowledge:** Promises, `async/await`, and the event loop.

### Related Programming Areas

- **Async/Await & Promise Integration:** The foundation for all async middleware patterns.
- **Error Handling Architecture:** Network errors must be caught and propagated correctly.
- **Production Error Handling & Observability:** Monitoring async operation failures.
- **Database Access Layer:** Repository methods return Promises consumed by services.
- **Dependency Injection:** HTTP clients and database pools are injected into services.

### Core Concepts

1. **Database Operations** — connection pools, transactional queries, and timeouts.
2. **External API Calls** — HTTP/gRPC clients, retries, backoffs, and circuit breakers.
3. **File Operations** — `fs/promises`, streams, and `stream.pipeline`.
4. **Background Processing (Offloading)** — message queues (BullMQ, RabbitMQ).

---

## Core Concept 1: Database Operations

### Definitions

**Core Definition:** Database operations in async middleware involve acquiring a connection from a pool, executing queries (including transactions), and handling timeouts — all without blocking the event loop.

**Technical Definition:** Node.js database drivers use native Promises and async/await for non-blocking I/O. The event loop remains free while database queries execute, enabling maximum concurrency. A **connection pool** maintains a set of reusable database connections. Each concurrent query acquires its own connection from the pool, executes, and releases it back. Connection pool settings include `poolSize` (maximum connections), `poolTimeout` (how long to wait for a connection), `idleTimeout` (how long idle connections are kept), and `connectionTimeoutMillis` (how long to wait for a new connection). Transactions group multiple queries into an atomic unit — they must be committed or rolled back within a specified timeout, or the database automatically rolls them back.

**Beginner-Friendly Explanation:** A database connection pool is like a fleet of delivery trucks. Instead of buying a new truck for every delivery (creating a new connection), you have a fleet (the pool) that gets reused. When a delivery is needed (a query), you take an available truck, make the delivery, and return the truck to the fleet. If all trucks are busy, you wait (pool timeout). Transactions are like a multi-stop delivery route — if any stop fails, the entire route is cancelled and no packages are delivered (rolled back).

### Purposes

- To handle non-blocking connection pools for concurrent database access.
- To execute transactional queries that maintain data integrity across multiple operations.
- To manage timeouts that prevent hung queries from holding connections indefinitely.
- To configure pool sizes that balance concurrency with database server limits.

### Syntax Rules and Structure

#### Connection Pool Configuration

```javascript
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,                        // Maximum connections in pool
  idleTimeoutMillis: 30000,       // Close idle connections after 30s
  connectionTimeoutMillis: 5000,  // Wait 5s for a connection before failing
  query_timeout: 10000            // Abort queries running longer than 10s
});
```

| Option | Description | Typical Value |
|--------|-------------|---------------|
| `max` / `poolSize` | Maximum connections in the pool. | `num_cpus * 2 + 1` or 10–20 |
| `idleTimeoutMillis` | Close idle connections after this time. | 30,000 ms |
| `connectionTimeoutMillis` | How long to wait for a connection. | 5,000 ms |
| `query_timeout` | Abort queries that exceed this duration. | 10,000 ms |

#### Transaction Example

```javascript
async function transferFunds(fromId, toId, amount) {
  const client = await pool.connect(); // Acquire a connection
  try {
    await client.query('BEGIN');
    await client.query(
      'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
      [amount, fromId]
    );
    await client.query(
      'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
      [amount, toId]
    );
    await client.query('COMMIT');
  } catch (err) {
    await client.query('ROLLBACK'); // Atomic rollback on failure
    throw err;
  } finally {
    client.release(); // Always release the connection
  }
}
```

#### Parallel Queries

```javascript
async function fetchDashboardData() {
  const [users, orders, products] = await Promise.all([
    db.query('SELECT * FROM users WHERE active = true'),
    db.query('SELECT * FROM orders WHERE status = $1', ['pending']),
    db.query('SELECT * FROM products WHERE in_stock = true')
  ]);
  return { users: users.rows, orders: orders.rows, products: products.rows };
}
```

**Rules:**
- Instantiate the pool **once** at application startup and export it for reuse.
- Always call `client.release()` in a `finally` block when using explicit connections.
- Set `connectionLimit` based on the database server's `max_connections` and the number of Node.js processes.
- Enable `enableKeepAlive` to prevent stale connections.
- Use `Promise.all()` for independent queries; use sequential `await` for dependent queries.

**Constraints:**
- Pool exhaustion occurs if connections are not released; this causes `pool timeout` errors.
- Transactions must not contain external API calls — this holds the connection open and causes pool exhaustion.
- Connection pool sizes on serverless platforms (Lambda) should be set to 1 unless multi-threaded.

### Annotated Code Example

```javascript
// db.js — pool instantiated once
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 10,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 5000
});

module.exports = pool;
```

```javascript
// services/user.service.js
const pool = require('../db');

async function getUserWithOrders(userId) {
  const client = await pool.connect();
  try {
    // Transaction: both queries succeed or both fail
    await client.query('BEGIN');

    const userResult = await client.query(
      'SELECT id, name, email FROM users WHERE id = $1',
      [userId]
    );

    if (userResult.rows.length === 0) {
      await client.query('ROLLBACK');
      throw new Error('User not found');
    }

    const ordersResult = await client.query(
      'SELECT id, total, status FROM orders WHERE user_id = $1',
      [userId]
    );

    await client.query('COMMIT');

    return {
      user: userResult.rows[0],
      orders: ordersResult.rows
    };
  } catch (err) {
    await client.query('ROLLBACK');
    throw err;
  } finally {
    client.release(); // CRITICAL: always release
  }
}
```

**Expected Output (for a valid user):**
```json
{
  "user": { "id": 1, "name": "Alice", "email": "alice@example.com" },
  "orders": [{ "id": 10, "total": 99.99, "status": "paid" }]
}
```

**Why this output:** The transaction begins, both queries execute on the same connection, and the transaction commits. If the user is not found, the transaction rolls back and the error is thrown. The `finally` block guarantees the connection is released back to the pool, preventing pool exhaustion.

### Real-World Cases

- **Banking:** Fund transfers requiring atomic debit/credit operations.
- **E-commerce:** Order creation with inventory decrement and payment record insertion.
- **Multi-tenant SaaS:** Per-tenant connection pools with isolated schemas.
- **Serverless:** Lambda functions with pool size 1 to avoid exhausting database connections.

---

## Core Concept 2: External API Calls

### Definitions

**Core Definition:** External API calls are outbound HTTP or gRPC requests made from an Express application to third-party services or internal microservices, requiring resilience patterns — retries, backoffs, and circuit breakers — to handle network failures gracefully.

**Technical Definition:** External API calls are made using clients like Axios or native `fetch`. Resilience patterns include **retries** (re-attempting failed requests), **exponential backoff** (increasing the delay between retries — e.g., 1s, 2s, 4s — with optional **jitter** to prevent thundering herds), and **circuit breakers** (tracking failure rates and "tripping open" to fail fast when a service is unhealthy, then allowing a "half-open" probe to test recovery). Libraries include `axios-retry` for retry logic and `opossum` for circuit breakers. The `@julr/tenace` library provides a fluent API combining timeout, retry, circuit breaker, and bulkhead patterns.

**Beginner-Friendly Explanation:** External API calls are like calling a customer service line. Sometimes the line is busy (transient failure) — you wait a moment and try again (retry). If the line is always busy, you wait longer between attempts (exponential backoff). If the line is completely dead (the service is down), you stop calling entirely and tell the customer "try later" (circuit breaker). This prevents you from wasting time on a broken service.

### Purposes

- To execute outbound HTTP/gRPC requests using clients like Axios or native `fetch`.
- To configure aggressive retry mechanisms with exponential backoff and jitter.
- To implement circuit breakers that prevent overwhelming failing services.
- To set timeouts that prevent hung requests from blocking the event loop.
- To provide fallback responses when external services are unavailable.

### Syntax Rules and Structure

#### Axios Retry with Exponential Backoff

```javascript
const axios = require('axios');
const axiosRetry = require('axios-retry').default;

axiosRetry(axios, {
  retries: 3,
  retryDelay: (retryCount) => {
    return axiosRetry.exponentialDelay(retryCount) + Math.random() * 1000;
  },
  retryCondition: (error) => {
    return axiosRetry.isNetworkOrIdempotentRequestError(error) ||
           error.response?.status === 429;
  },
  onRetry: (retryCount, error, requestConfig) => {
    console.log(`Retry ${retryCount} for ${requestConfig.url}: ${error.message}`);
  }
});
```

| Option | Description |
|--------|-------------|
| `retries` | Maximum number of retry attempts. |
| `retryDelay` | Function returning delay in ms; use `exponentialDelay` + jitter. |
| `retryCondition` | Predicate determining whether to retry. |
| `onRetry` | Callback for logging each retry. |

#### Circuit Breaker with Opossum

```javascript
const CircuitBreaker = require('opossum');

const options = {
  timeout: 5000,                   // 5s timeout per request
  errorThresholdPercentage: 50,    // Trip open after 50% failures
  resetTimeout: 30000,             // Try half-open after 30s
  volumeThreshold: 5               // Minimum requests before tripping
};

const breaker = new CircuitBreaker(fetchExternalData, options);

breaker.fallback(() => ({ error: 'Service temporarily unavailable' }));

breaker.on('open', () => console.log('Circuit OPEN — failing fast'));
breaker.on('halfOpen', () => console.log('Circuit HALF-OPEN — testing'));
breaker.on('close', () => console.log('Circuit CLOSED — recovered'));

app.get('/api/external', async (req, res) => {
  const result = await breaker.fire();
  res.json(result);
});
```

**Rules:**
- Only retry **idempotent** requests (GET, PUT, DELETE) — never retry POST blindly.
- Use jitter (`Math.random() * 1000`) with exponential backoff to prevent thundering herds.
- The circuit breaker should have a `fallback` function that returns a safe default.
- Set aggressive timeouts (5–10s) to prevent hung requests from holding connections.
- Log circuit state transitions (`open`, `halfOpen`, `close`) for monitoring.

**Constraints:**
- Circuit breakers add state and complexity; use only for unreliable external dependencies.
- Retries can amplify load on an already-struggling service — always combine with circuit breakers.
- Native `fetch` does not have built-in retry logic; use `fetch-retry` or wrap manually.

### Annotated Code Example

```javascript
// services/external-api.service.js
const CircuitBreaker = require('opossum');

async function fetchPaymentStatus(paymentId) {
  const response = await fetch(
    `https://api.payment-provider.com/payments/${paymentId}`,
    { signal: AbortSignal.timeout(5000) } // 5s timeout
  );
  if (!response.ok) {
    throw new Error(`Payment API returned ${response.status}`);
  }
  return response.json();
}

const breaker = new CircuitBreaker(fetchPaymentStatus, {
  timeout: 5000,
  errorThresholdPercentage: 50,
  resetTimeout: 30000,
  volumeThreshold: 5
});

breaker.fallback(() => ({ status: 'unknown', error: 'Payment service unavailable' }));

breaker.on('open', () => console.error('Payment circuit OPEN'));
breaker.on('halfOpen', () => console.warn('Payment circuit HALF-OPEN'));
breaker.on('close', () => console.info('Payment circuit CLOSED'));

// Express route
app.get('/api/payments/:id/status', async (req, res) => {
  const result = await breaker.fire(req.params.id);
  res.json(result);
});
```

**Expected Output (when payment service is healthy):**
```json
{ "status": "paid", "amount": 99.99 }
```

**Expected Output (when payment service is down — circuit open):**
```json
{ "status": "unknown", "error": "Payment service unavailable" }
```

**Why this output:** When the payment service is healthy, the circuit is closed and requests pass through. When failures exceed 50% (with at least 5 requests), the circuit opens and all subsequent requests immediately return the fallback response — no network call is attempted. After 30 seconds, the circuit half-opens and allows a probe request to test recovery.

### Real-World Cases

- **Payment processing:** Retrying failed charges with exponential backoff; circuit breaker for payment gateway outages.
- **Shipping rate quotes:** Circuit breaker with fallback to cached rates when the carrier API is down.
- **Email delivery:** Retrying with backoff and jitter for transient SMTP failures.
- **Microservices:** Circuit breakers between services to prevent cascading failures.

---

## Core Concept 3: File Operations

### Definitions

**Core Definition:** File operations in async middleware involve reading, writing, and transforming files using Promise-based APIs (`fs/promises`) or streams, without blocking the event loop and without loading entire files into memory.

**Technical Definition:** The `fs/promises` module provides Promise-based versions of file system methods (`readFile`, `writeFile`, `appendFile`, `unlink`, `stat`, etc.). For large files (multi-gigabyte), `fs.readFile` loads the entire file into memory, which is unsafe. Instead, **streams** process data piece-by-piece in chunks. The `stream.pipeline()` function (from `stream/promises`) connects readable, transform, and writable streams while propagating errors and cleaning up resources automatically. `pipe()` does not forward errors automatically — `pipeline()` is the modern, safe alternative.

**Beginner-Friendly Explanation:** Reading a large file with `fs.readFile` is like trying to swallow a whole pizza in one bite — you'll choke (run out of memory). Reading it with a stream is like eating one slice at a time — you get the whole pizza eventually, but you never choke. `stream.pipeline` is the conveyor belt that moves slices from the pizza box (readable stream), through a seasoning station (transform stream), and onto your plate (writable stream), with automatic shutdown if anything goes wrong.

### Purposes

- To avoid synchronous `fs` methods that block the event loop.
- To utilise the Promise-based `fs/promises` module for async file operations.
- To use `stream.pipeline` for multi-gigabyte files to preserve RAM.
- To handle backpressure automatically when reading is faster than writing.
- To process files incrementally (e.g., CSV parsing, compression, encryption).

### Syntax Rules and Structure

#### fs/promises Example

```javascript
const fs = require('fs/promises');

async function processConfig() {
  const data = await fs.readFile('config.json', 'utf8');
  const config = JSON.parse(data);
  return config;
}

async function writeLog(entry) {
  await fs.appendFile('app.log', `${new Date().toISOString()} ${entry}\n`);
}
```

#### Stream Pipeline Example

```javascript
const { pipeline } = require('stream/promises');
const fs = require('fs');
const zlib = require('zlib');

async function compressFile(inputPath, outputPath) {
  await pipeline(
    fs.createReadStream(inputPath),
    zlib.createGzip(),
    fs.createWriteStream(outputPath)
  );
}
```

| Method | Use Case | Memory Usage |
|--------|----------|-------------|
| `fs.readFile` | Small files (< 10 MB). | Entire file in RAM. |
| `fs.createReadStream` | Large files (GBs). | One chunk (default 64 KB) at a time. |
| `pipe()` | Simple piping. | Chunk-based, but errors not propagated. |
| `pipeline()` | Complex pipelines with error handling. | Chunk-based, automatic cleanup. |

**Rules:**
- Never use `fs.readFileSync` or `fs.writeFileSync` in request handlers.
- Use `fs/promises` for all async file operations.
- Use `stream.pipeline()` instead of `pipe()` for error-safe pipelines.
- For very large files, use async iterators (`for await (const chunk of stream)`) for fine-grained control.
- Set appropriate `highWaterMark` values for performance tuning.

**Constraints:**
- `pipe()` does not forward errors from source or intermediate streams — use `pipeline()`.
- Custom Transform streams require understanding of internal buffering and backpressure.
- Streams are single-use — once read, they cannot be re-read.

### Annotated Code Example

```javascript
// file-operations.js
const express = require('express');
const fs = require('fs');
const { pipeline } = require('stream/promises');
const zlib = require('zlib');
const app = express();

// Stream a large file to the HTTP response
app.get('/download/large', (req, res) => {
  res.setHeader('Content-Type', 'application/zip');
  res.setHeader('Content-Disposition', 'attachment; filename="archive.zip"');

  pipeline(
    fs.createReadStream('huge-data.csv'),
    zlib.createGzip(),
    res  // HTTP response is a writable stream
  ).catch(err => {
    console.error('Pipeline failed:', err.message);
    res.destroy(); // Clean up on error
  });
});

// Process a CSV file line by line with async iterator
app.get('/process/csv', async (req, res) => {
  const rl = require('readline').createInterface({
    input: fs.createReadStream('data.csv')
  });

  let lineCount = 0;
  for await (const line of rl) {
    lineCount++;
    // Process each line without loading the entire file
  }

  res.json({ lines: lineCount });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /process/csv`):**
```json
{ "lines": 1500000 }
```

**Why this output:** The CSV file is read line by line using `readline` over a `createReadStream`. The entire file is never loaded into memory — only one line at a time. This allows processing files larger than available RAM.

### Real-World Cases

- **Video streaming:** Piping a video file to the HTTP response without loading it into memory.
- **Log rotation:** Compressing old log files with `stream.pipeline` and `zlib.createGzip()`.
- **Data exports:** Streaming database query results to a CSV file chunk by chunk.
- **File uploads:** Piping the request stream directly to a file write stream.

---

## Core Concept 4: Background Processing (Offloading)

### Definitions

**Core Definition:** Background processing (offloading) is the practice of deferring long-running or resource-intensive operations outside the HTTP request-response cycle to a message queue, where a separate worker process handles them asynchronously.

**Technical Definition:** A **job queue** is a system for deferring work outside the request-response cycle. It works in four phases: (1) the HTTP handler **enqueues** a job and returns immediately, (2) the job queue **persists** the job and confirms receipt, (3) a worker process **dequeues** and executes the job, and (4) results are **stored** or **notified**. BullMQ is a Redis-backed job queue library for Node.js, designed for robustness and ease of use. RabbitMQ is a general-purpose message broker using the AMQP protocol with advanced routing, exchange types, and native dead-letter exchange support. BullMQ is typically better suited for background job processing within Node.js applications, while RabbitMQ excels at complex routing and inter-service communication.

**Beginner-Friendly Explanation:** Background processing is like a restaurant's order system. The waiter (HTTP handler) takes your order and gives it to the kitchen (job queue). The waiter doesn't stand in the kitchen waiting for your food — they go serve other tables. When your food is ready, a runner (worker) brings it to you. The kitchen (queue) persists the order so it's not lost if the restaurant has a power outage.

### Purposes

- To defer long-running operations outside the request-response cycle.
- To use message queues (BullMQ, RabbitMQ) instead of executing slow work in the HTTP thread.
- To enable retries with exponential backoff for failed jobs.
- To schedule recurring tasks (cron jobs) using job schedulers.
- To decouple producers from consumers in a microservices architecture.

### Syntax Rules and Structure

#### BullMQ Example

```javascript
// producer.js — enqueue jobs from Express
const { Queue } = require('bullmq');
const connection = { host: 'localhost', port: 6379 };

const emailQueue = new Queue('email', { connection });

app.post('/api/users', async (req, res) => {
  const user = await createUser(req.body);

  // Enqueue — returns immediately
  await emailQueue.add('send-welcome', {
    userId: user.id,
    email: user.email
  });

  res.status(201).json({ data: user });
});
```

```javascript
// worker.js — process jobs in a separate process
const { Worker } = require('bullmq');
const connection = { host: 'localhost', port: 6379 };

const worker = new Worker('email', async (job) => {
  console.log(`Sending email to ${job.data.email}`);
  await sendEmail(job.data.email, 'Welcome!');
}, { connection, concurrency: 10 });

worker.on('failed', (job, err) => {
  console.error(`Job ${job.id} failed: ${err.message}`);
});
```

| Queue System | Best For | Protocol |
|-------------|----------|----------|
| BullMQ | Background jobs, scheduled tasks, Node.js ecosystem. | Redis |
| RabbitMQ | Complex routing, inter-service communication. | AMQP |

**Rules:**
- The producer (Express route) only enqueues — it never executes the job.
- The worker runs in a **separate process** so it doesn't block the HTTP server.
- Configure `defaultJobOptions` with `attempts` and `backoff` for retry logic.
- Use dead-letter queues (BullMQ's failed set, RabbitMQ's DLX) to inspect failed jobs.
- Monitor queue depth and worker health for operational visibility.

**Constraints:**
- Job queues add infrastructure complexity (Redis or RabbitMQ must be deployed).
- Jobs must be idempotent — they may be retried multiple times.
- Serialisation of job data requires JSON-compatible payloads.

### Annotated Code Example

```javascript
// queue.js
const { Queue } = require('bullmq');
const connection = { host: 'localhost', port: 6379 };

const imageQueue = new Queue('image-processing', {
  connection,
  defaultJobOptions: {
    attempts: 3,
    backoff: { type: 'exponential', delay: 1000 }
  }
});

module.exports = { imageQueue };
```

```javascript
// routes/upload.routes.js
const { imageQueue } = require('../queue');
const multer = require('multer');
const upload = multer({ dest: 'uploads/' });

router.post('/upload', upload.single('image'), async (req, res) => {
  // Enqueue — return immediately
  const job = await imageQueue.add('resize', {
    filePath: req.file.path,
    sizes: [200, 600, 1200]
  });

  res.status(202).json({
    message: 'Image queued for processing',
    jobId: job.id
  });
});
```

```javascript
// worker.js (separate process)
const { Worker } = require('bullmq');
const connection = { host: 'localhost', port: 6379 };

new Worker('image-processing', async (job) => {
  console.log(`Processing ${job.data.filePath}`);
  for (const size of job.data.sizes) {
    await resizeImage(job.data.filePath, size);
  }
  console.log(`Completed job ${job.id}`);
}, { connection, concurrency: 5 });
```

**Expected Output (for `POST /upload`):**
```json
{
  "message": "Image queued for processing",
  "jobId": "42"
}
```

**Expected Output (worker log):**
```
Processing uploads/abc123.jpg
Completed job 42
```

**Why this output:** The Express route enqueues the job and returns a 202 Accepted response with the job ID. The worker process picks up the job, resizes the image in three sizes, and logs completion. The HTTP request is not blocked by the image processing.

### Real-World Cases

- **Image processing:** Resizing uploaded images in the background.
- **Email sending:** Queueing welcome emails, password reset emails, and notifications.
- **Report generation:** Generating large PDF or CSV reports asynchronously.
- **Payment reconciliation:** Processing daily payment settlements in the background.
- **Data synchronisation:** Syncing data with third-party APIs on a schedule.

---

## References

- ThemisDB — Node.js Async Patterns (Connection Pooling, Parallel Queries, Streaming Cursors) — https://github.com/makr-code/ThemisDB/blob/bc3811f51430187ca6be64be16dea974d82815d7/docs/compendium/docs/chapter_22_clients.md
- How to Implement Connection Pooling for MySQL in Node.js — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-06-connection-pooling-nodejs/README.md
- CockroachDB — Serverless Function Best Practices (Connection Pool) — https://docs.cockroachlabs.com
- Google Cloud — Setting Connection Timeout with Node.js (Cloud SQL for PostgreSQL) — https://docs.cloud.google.com/sql/docs/postgres/configure-ip
- npm — axios-retry — https://www.npmjs.com/package/axios-retry
- Scrapfly — How to Retry in Axios — https://scrapfly.io/blog/axios-retry/
- FoxReload — Retry/Backoff Patterns for B2B Integrations 2026 — https://foxreload.com
- Opossum — Node.js Circuit Breaker — https://github.com/nodeshift/opossum
- GitHub — Node-Opossum Circuit Breaker Example — https://github.com/fernando-pires47/node-opossum-circuit-breaker
- TRAE-Skills — Node.js Streams (fs/promises, pipeline, backpressure) — https://github.com/HighMark-31/TRAE-Skills/blob/main/backend/Nodejs_Streams.md
- GitNation — JavaScript File Handling Like a Pro (Streams, pipeline, async iterators) — https://gitnation.com
- Pluralsight — Node.js: File System, Streams, Async I/O — https://www.pluralsight.com
- OneUptime — Redis vs RabbitMQ for Job Queues (BullMQ example) — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-redis-vs-rabbitmq-for-job-queues/README.md
- BullMQ — Official Documentation — https://bullmq.io/
- OneUptime — BullMQ vs Other Queue Systems (RabbitMQ, SQS) — https://oneuptime.com
- Express.js — Error Handling Guide (Async Error Handling) — https://expressjs.com/en/guide/error-handling.html
- Node.js Best Practices — Error Handling (Async Wrappers) — https://github.com/goldbergyoni/nodebestpractices
- Stack Overflow — Sequelize Transaction Stays Open Too Long — https://stackoverflow.com
- Google Cloud — Connection Pool Timeout Handling — https://docs.cloud.google.com/sql/docs/postgres/configure-ip
- MongoDB — How to Use Connection Pooling Effectively — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mongodb-connection-pooling/README.md
- OWASP — REST Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
- Node.js — Streams API — https://nodejs.org/api/stream.html
- Node.js — `fs/promises` — https://nodejs.org/api/fs.html#promises-api
- Node.js — `stream.pipeline` — https://nodejs.org/api/stream.html#streampipelinesource-transforms-destination-callback