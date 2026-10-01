# Production Error Handling & Observability — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Production error handling and observability is the discipline of ensuring that application failures are detected, logged, traced, and reported in a production environment without leaking sensitive information to clients, while enabling operators to diagnose and recover from incidents.

**Technical Definition:** Production error handling and observability encompasses five interrelated concerns: (1) **sanitising error responses** so that stack traces, internal file paths, and library versions never reach clients; (2) **structured logging** that emits machine-readable JSON records instead of plain-text console lines; (3) **correlation IDs** propagated via `AsyncLocalStorage` to trace a single request's lifecycle across logs and downstream services; (4) **error monitoring** integrations that pipe error events into telemetry platforms (Sentry, OpenTelemetry, Datadog) for active alerting; and (5) **graceful failure and process safety** using Node.js process events (`uncaughtException`, `unhandledRejection`) combined with orchestration layers (PM2, Docker, Kubernetes) to restart cleanly without data corruption.

**Beginner-Friendly Explanation:** In development, you want to see everything — full stack traces, debug logs, and verbose error messages. In production, you want the opposite: clients should never see your internal code paths, logs should be structured so machines can search them, every request should have a unique ID so you can trace it across services, errors should automatically alert your team, and if something catastrophic happens, the process should shut down gracefully and be restarted by the platform — not linger in a corrupted state.

### Key Characteristics

- **Defence in depth:** Multiple layers — response sanitisation, structured logging, correlation, monitoring, and process safety — work together.
- **Environment-aware:** Behaviour differs between `NODE_ENV=development` (verbose) and `NODE_ENV=production` (sanitised).
- **Machine-first:** Logs are JSON, not formatted text; IDs are unique and traceable; errors are structured.
- **Fail-safe:** The process is designed to crash cleanly and be restarted by an orchestrator rather than continue in an unknown state.
- **Observable:** Every error is logged, traced, and (where appropriate) alerted.

### Prerequisites

- **Node.js runtime** (v18 or higher; `AsyncLocalStorage` is stable since v16).
- **Express.js installed:** `npm install express`.
- **A logging library:** `npm install pino` or `npm install winston`.
- **An error monitoring SDK:** `npm install @sentry/node` (optional but recommended).
- **A process manager or orchestrator:** PM2, Docker, or Kubernetes.

### Related Programming Areas

- **Error Handling Architecture:** The propagation and middleware system that feeds errors into the observability pipeline.
- **Error Design:** The response schemas that sanitised errors conform to.
- **Logging and Monitoring:** Structured logging is the foundation of observability.
- **Distributed Tracing:** Correlation IDs and OpenTelemetry spans trace requests across services.
- **Deployment and Operations:** PM2, Docker, and Kubernetes manage process restarts.

### Core Concepts

1. **Avoiding Stack-Trace Leakage** — sanitising response payloads in production.
2. **Structured Logging** — machine-readable single-line JSON streams (Winston, Pino).
3. **Correlation/Request IDs** — unique IDs propagated via `AsyncLocalStorage`.
4. **Error Monitoring** — hooking error events into Sentry, OpenTelemetry, Datadog.
5. **Graceful Failure & Process Safety** — `uncaughtException`, `unhandledRejection`, and orchestration restarts.

---

## Core Concept 1: Avoiding Stack-Trace Leakage

### Definitions

**Core Definition:** Stack-trace leakage is the accidental exposure of internal runtime details — absolute file paths, line numbers, library versions, and call stacks — in error responses sent to clients. Avoiding it requires sanitising production error responses so that only safe, generic messages are returned.

**Technical Definition:** Express in development mode includes full stack traces in error responses by default. `NODE_ENV=production` suppresses some Express error details but does not fix body-parser stack traces. Sending a request with a malformed JSON body returns a full Node.js stack trace that exposes absolute home directory paths, exact `node_modules` paths, and line numbers that fingerprint exact library versions in use. This is a standard hardening step for any production Node.js/Express app: stack traces should be logged server-side but never sent to the client.

**Beginner-Friendly Explanation:** When something goes wrong, your server knows exactly which file, which line, and which function caused the problem. That information is gold for you (the developer) but dangerous for attackers. Stack-trace leakage is like a bank teller accidentally reading out the vault combination to a customer — it reveals internal details that should stay private. Production error handling ensures the customer only hears "we're working on it" while the vault combination goes to the security log.

### Purposes

- To sanitise response payloads to strip out internal runtime paths, lines of code, and stack traces when `NODE_ENV === 'production'`.
- To prevent attackers from fingerprinting library versions and internal architecture.
- To comply with security best practices and regulatory requirements.
- To maintain a professional, non-technical error experience for end users.

### Syntax Rules and Structure

```typescript
// Environment-aware error handler
app.use((err, req, res, next) => {
  // Always log the full error server-side
  console.error(err);

  // In production, never send stack traces or internal details
  if (process.env.NODE_ENV === 'production') {
    return res.status(500).json({
      error: 'Internal server error'
    });
  }

  // In development, include the stack for debugging
  res.status(500).json({
    error: err.message,
    stack: err.stack
  });
});
```

| Component | Breakdown |
|-----------|-----------|
| `console.error(err)` | Logs the full stack server-side (never sent to client). |
| `NODE_ENV === 'production'` | Environment check to switch behaviour. |
| `res.status(500).json(...)` | Sends a sanitised response in production. |

**Rules:**
- Stack traces must **always** be logged server-side but **never** sent to clients in production.
- `NODE_ENV=production` must be set in the deployment environment.
- Body-parser errors (`entity.parse.failed`) require explicit handling — Express does not suppress them automatically.
- Operational errors (e.g., 404, 422) should return their safe messages; programmer errors (bugs) should return a generic 500.

**Constraints:**
- `NODE_ENV=production` does not suppress body-parser stack traces — a global error handler is still required.
- Source maps must be uploaded to monitoring tools separately if you want readable stack traces in the dashboard.

### Annotated Code Example

```typescript
// middleware/errorHandler.ts
export function errorHandler(
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) {
  // Always log the full error server-side
  console.error({
    message: err.message,
    stack: err.stack,
    url: req.originalUrl,
    method: req.method,
    timestamp: new Date().toISOString()
  });

  // Handle body-parser parse errors explicitly
  if ((err as any).type === 'entity.parse.failed') {
    return res.status(400).json({ error: 'Bad request' });
  }

  // Production: sanitised response
  if (process.env.NODE_ENV === 'production') {
    return res.status(500).json({ error: 'Internal server error' });
  }

  // Development: verbose response
  res.status(500).json({
    error: err.message,
    stack: err.stack
  });
}
```

**Expected Output (production — malformed JSON):**
```
HTTP/1.1 400 Bad Request

{"error":"Bad request"}
```

**Expected Output (production — unexpected bug):**
```
HTTP/1.1 500 Internal Server Error

{"error":"Internal server error"}
```

**Why this output:** In production, the client receives only a generic message. The full stack trace — including absolute paths and library versions — is logged server-side for the developer to inspect. The body-parser error is handled explicitly with a 400, preventing the default stack-trace response.

### Real-World Cases

- **Security auditing:** Preventing attackers from fingerprinting Express, body-parser, and other library versions.
- **Compliance:** Meeting requirements that prohibit exposing internal system details.
- **User experience:** Showing a clean error page instead of a raw stack trace.

---

## Core Concept 2: Structured Logging

### Definitions

**Core Definition:** Structured logging is the practice of emitting log records as machine-readable, single-line JSON objects rather than formatted plain-text strings, enabling automated parsing, filtering, aggregation, and alerting by log management platforms.

**Technical Definition:** Structured logging libraries (Winston, Pino) output each log entry as a JSON object with consistent keys: `level` (log severity), `time` (ISO timestamp), `pid`, `hostname`, `msg` (message), and any additional contextual fields. Pino serialises JSON logs 5–10x faster than Winston by avoiding synchronous string formatting in the hot path and offloading I/O to worker threads. Winston prioritises flexibility with 80+ community transports and a composable format pipeline. Both produce structured JSON logs and support log levels. `pino-http` and `morgan` provide HTTP request logging middleware.

**Beginner-Friendly Explanation:** Structured logging is like writing a database record instead of a diary entry. A diary entry ("Today I saw a user try to log in but the password was wrong") is readable by humans but hard for machines to search. A database record (`{"event": "login_failed", "reason": "invalid_password", "userId": 123}`) is instantly searchable, filterable, and alertable. In production, machines read your logs more than humans do.

### Purposes

- To emit runtime data logs purely as machine-readable single-line JSON streams.
- To enable log aggregation platforms (Datadog, Loki, CloudWatch, ELK) to parse and index logs.
- To support structured alerting based on log fields (e.g., alert when `level=error` and `service=payments`).
- To reduce the performance overhead of logging in high-throughput services.

### Sub-Feature 2.1: Pino (Performance-First)

#### Syntax Rules and Structure

```typescript
// logger.ts
import pino from 'pino';

export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level(label) {
      return { level: label }; // "info" instead of 30
    }
  }
});
```

```typescript
// app.ts
import express from 'express';
import pinoHttp from 'pino-http';
import { logger } from './logger';

const app = express();
app.use(pinoHttp({ logger }));

app.get('/', (req, res) => {
  req.log.info('handling root request');
  res.json({ status: 'ok' });
});
```

| Component | Breakdown |
|-----------|-----------|
| `pino({ level })` | Creates a logger with the specified level. |
| `formatters.level` | Overrides numeric levels with string labels. |
| `pinoHttp({ logger })` | HTTP request logging middleware. |
| `req.log` | Child logger with request context bound. |

**Rules:**
- Use `pino-pretty` only in development — production should output raw JSON.
- `pino-http` must be registered before any routes.
- Set `LOG_LEVEL` via environment variable to change verbosity without redeploying.
- Pino is ~5x faster than Winston and adds ~1ms per 1000 log calls.

**Constraints:**
- Pino's default output is raw JSON — pipe through `pino-pretty` for local development.
- Error serialisation requires `pino.stdSerializers.err`.

---

### Sub-Feature 2.2: Winston (Flexibility-First)

#### Syntax Rules and Structure

```typescript
import winston from 'winston';

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'order-service' },
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'error.log', level: 'error' })
  ]
});

logger.info('Order created', { orderId: 'ord_123', userId: 'usr_456' });
```

**Expected Output (Pino JSON):**
```json
{"level":"info","time":1781170570778,"pid":317380,"hostname":"falcon","msg":"handling root request"}
```

**Expected Output (Winston JSON):**
```json
{"level":"info","message":"Order created","orderId":"ord_123","userId":"usr_456","service":"order-service","timestamp":"2026-06-07T12:00:00.000Z"}
```

**Why this output:** Pino emits compact JSON with numeric timestamps. Winston emits JSON with ISO timestamps and custom metadata fields. Both are single-line JSON records that log aggregators can parse.

### Real-World Cases

- **High-throughput APIs:** Pino for services handling 10,000+ requests/second.
- **Multi-transport requirements:** Winston when logs must go to console, file, Elasticsearch, and Datadog simultaneously.
- **Development:** `pino-pretty` or Winston's colourised console format for readable local logs.

---

## Core Concept 3: Correlation / Request IDs

### Definitions

**Core Definition:** Correlation IDs are unique identifiers assigned to each incoming request and propagated through every log line, database query, and downstream service call, enabling the complete lifecycle of a transaction to be traced across distributed systems.

**Technical Definition:** `AsyncLocalStorage` is a Node.js API that creates stores that stay coherent through asynchronous operations. It associates state and propagates it throughout callbacks and promise chains, similar to thread-local storage in other languages. In Express, a middleware generates or reads a correlation ID from the `X-Request-ID` header, runs the request handler within `asyncLocalStorage.run()`, and every log line within that async context automatically includes the correlation ID without manual plumbing. Libraries like `pino-correlation-id` and `express-http-context` provide ready-made implementations.

**Beginner-Friendly Explanation:** A correlation ID is like a tracking number for a package. When you order something online, you get a tracking number that lets you follow the package through every stage — warehouse, sorting facility, delivery truck. In a microservices architecture, a correlation ID follows a request through the API gateway, the auth service, the database, and the payment processor, so you can see exactly where it went and where it failed.

### Purposes

- To generate unique IDs (e.g., via `uuid` or `hyperid`) per request using `AsyncLocalStorage`.
- To trace a transaction's lifecycle across logs and downstream services.
- To correlate logs from different services processing the same request.
- To enable debugging of distributed transactions without manually passing IDs.

### Syntax Rules and Structure

```typescript
// context.ts
import { AsyncLocalStorage } from 'node:async_hooks';

export const asyncLocalStorage = new AsyncLocalStorage<Map<string, string>>();

export function getCorrelationId(): string | undefined {
  return asyncLocalStorage.getStore()?.get('correlationId');
}
```

```typescript
// middleware/correlationId.ts
import { randomUUID } from 'node:crypto';
import { asyncLocalStorage } from '../context';

export function correlationId(req, res, next) {
  const id = req.headers['x-request-id'] || randomUUID();
  res.setHeader('x-request-id', id);

  asyncLocalStorage.run(new Map([['correlationId', id]]), () => {
    next();
  });
}
```

| Component | Breakdown |
|-----------|-----------|
| `AsyncLocalStorage` | Node.js built-in for async context tracking. |
| `randomUUID()` | Generates a UUID v4. |
| `req.headers['x-request-id']` | Reads an upstream correlation ID. |
| `asyncLocalStorage.run()` | Runs the callback within the async context. |

**Rules:**
- The middleware must be registered **early** — before any routes.
- Read `X-Request-ID` from upstream services; generate one only if absent.
- Echo the ID back in the response header for client-side tracing.
- `AsyncLocalStorage` is stable since Node.js v16.4.0.

**Constraints:**
- `AsyncLocalStorage` adds a small performance overhead; benchmark in high-throughput services.
- The store is not available outside the `run()` callback.

### Annotated Code Example

```typescript
// server.ts
import express from 'express';
import { randomUUID } from 'node:crypto';
import { AsyncLocalStorage } from 'node:async_hooks';

const asyncLocalStorage = new AsyncLocalStorage<Map<string, string>>();
const app = express();

// Correlation ID middleware — register BEFORE routes
app.use((req, res, next) => {
  const correlationId = (req.headers['x-request-id'] as string) || randomUUID();
  res.setHeader('x-request-id', correlationId);

  asyncLocalStorage.run(new Map([['correlationId', correlationId]]), () => {
    next();
  });
});

// Route that logs with correlation ID
app.get('/users/:id', async (req, res) => {
  const correlationId = asyncLocalStorage.getStore()?.get('correlationId');
  console.log(`[${correlationId}] Fetching user ${req.params.id}`);
  res.json({ id: req.params.id, correlationId });
});

app.listen(3000);
```

**Expected Output (response header):**
```
x-request-id: 550e8400-e29b-41d4-a716-446655440000
```

**Expected Output (server log):**
```
[550e8400-e29b-41d4-a716-446655440000] Fetching user 42
```

**Why this output:** The middleware generates a UUID and stores it in `AsyncLocalStorage`. The route handler retrieves it and includes it in the log. If the request came from an upstream service with its own `x-request-id`, that ID would be reused instead, linking the two services' logs.

### Real-World Cases

- **Microservices:** Tracing a request through API gateway → auth → users → database.
- **Distributed debugging:** Correlating logs from multiple services for a single failed transaction.
- **Support tracing:** A customer reports an issue; support looks up the `x-request-id` in the logs.

---

## Core Concept 4: Error Monitoring

### Definitions

**Core Definition:** Error monitoring is the integration of an application with a telemetry platform that automatically captures, aggregates, and alerts on errors — providing real-time visibility into production failures with stack traces, breadcrumbs, and contextual metadata.

**Technical Definition:** Sentry is a popular error monitoring platform that provides SDKs for Node.js. Since `@sentry/node` v8, Sentry uses OpenTelemetry-based auto-instrumentation. `Sentry.init()` must run before any instrumented module (http, express, pg) is imported — the recommended pattern is to put the init in its own file and load it first. Sentry automatically captures unhandled exceptions and can be integrated with Express via request handlers. Errors can be correlated with OpenTelemetry traces, Winston logs, and trace IDs.

**Beginner-Friendly Explanation:** Error monitoring is like having a security team that watches every alarm in your building. When a smoke detector goes off, the team is immediately notified, they see which room it's in, what happened leading up to it (breadcrumbs), and they can dispatch help. Sentry is that security team for your code — it tells you what broke, where, and why, before your users even report it.

### Purposes

- To hook error events into telemetry platforms (Sentry, OpenTelemetry, Datadog) for active production alerting.
- To capture unhandled exceptions with full stack traces and contextual metadata.
- To correlate errors with OpenTelemetry traces for distributed tracing.
- To provide breadcrumbs (user actions leading to the error) for debugging.
- To alert the development team in real time when errors occur.

### Syntax Rules and Structure

```typescript
// instrument.ts — must be imported FIRST
import * as Sentry from '@sentry/node';
import { nodeProfilingIntegration } from '@sentry/profiling-node';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  integrations: [nodeProfilingIntegration()],
  tracesSampleRate: 0.1, // 10% in production
  profilesSampleRate: 0.1,
  environment: process.env.NODE_ENV
});
```

```typescript
// app.ts
import './instrument'; // FIRST import
import express from 'express';

const app = express();

app.use(Sentry.Handlers.requestHandler()); // First middleware
// ... routes ...
app.use(Sentry.Handlers.errorHandler());  // Before other error middleware
```

| Component | Breakdown |
|-----------|-----------|
| `Sentry.init()` | Initialises the SDK with DSN and options. |
| `tracesSampleRate` | Percentage of transactions to trace (0.1 = 10%). |
| `requestHandler()` | Express middleware that adds request context. |
| `errorHandler()` | Express middleware that captures errors. |

**Rules:**
- `Sentry.init()` must run **before** any instrumented module is imported.
- The request handler must be the **first** middleware.
- The error handler must be **before** any other error middleware.
- Lower `tracesSampleRate` in high-traffic production environments (e.g., 0.01).
- Use `beforeSend` or data scrubbing to remove PII from reports.

**Constraints:**
- Sentry adds a small performance overhead; sample traces to control cost.
- Source maps must be uploaded during the build for readable stack traces.
- PII must be scrubbed from error reports.

### Annotated Code Example

```typescript
// instrument.ts
import * as Sentry from '@sentry/node';
import { nodeProfilingIntegration } from '@sentry/profiling-node';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  integrations: [nodeProfilingIntegration()],
  tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0,
  profilesSampleRate: 0.1,
  environment: process.env.NODE_ENV || 'development',
  beforeSend(event) {
    // Scrub sensitive data
    if (event.request?.headers) {
      delete event.request.headers['authorization'];
      delete event.request.headers['cookie'];
    }
    return event;
  }
});
```

```typescript
// app.ts
import './instrument'; // Must be first
import express from 'express';
import * as Sentry from '@sentry/node';

const app = express();

app.use(Sentry.Handlers.requestHandler());
app.use(express.json());

app.get('/users/:id', async (req, res) => {
  throw new Error('Database connection failed');
});

app.use(Sentry.Handlers.errorHandler());

app.listen(3000);
```

**Expected Output (Sentry dashboard):**
```
Error: Database connection failed
  at /app/routes/users.js:42:12
  ...
  Breadcrumbs:
    - GET /users/42
    - express.json()
  Tags: environment=production, release=v1.2.3
  Trace ID: abc123def456
```

**Why this output:** Sentry captures the unhandled error, including the stack trace, breadcrumbs (the sequence of middleware and request details), and the trace ID linking it to the OpenTelemetry span. The `beforeSend` hook scrubs the authorization header before sending the event.

### Real-World Cases

- **Production alerting:** Notifying the on-call engineer when error rates spike.
- **Release tracking:** Correlating errors with specific deployments to identify regressions.
- **User impact analysis:** Seeing how many users were affected by a bug.
- **Distributed tracing:** Linking frontend errors to backend traces via trace IDs.

---

## Core Concept 5: Graceful Failure & Process Safety

### Definitions

**Core Definition:** Graceful failure and process safety is the practice of intercepting process-level errors (`uncaughtException`, `unhandledRejection`), performing synchronous cleanup of resources (database connections, file handles, server sockets), and allowing the process to exit cleanly so that an orchestrator (PM2, Docker, Kubernetes) can restart it.

**Technical Definition:** The correct use of `uncaughtException` is to perform synchronous cleanup of allocated resources (e.g., file descriptors, handles, etc.) before shutting down the process. After an uncaught exception, the app is in an unknown state — the safe pattern is to log the error, then exit and let the process manager restart a fresh instance. Modern Node.js (v15+) crashes the process on unhandled promise rejections by default. PM2 automatically restarts processes on uncaught exceptions; Kubernetes restarts containers based on restart policies. Graceful shutdown listeners for `SIGTERM` and `SIGINT` ensure that in-flight requests complete and database connections close before exit.

**Beginner-Friendly Explanation:** Think of a pilot who encounters an engine failure. The pilot doesn't try to "fix it mid-air" — they follow a checklist: shut down the failed engine, notify air traffic control, and land the plane safely. Once on the ground, the mechanics (PM2/Kubernetes) inspect and restart the aircraft. In Node.js, when an uncaught exception occurs, the process is in an unknown state — the safest action is to log, clean up, and exit, letting the orchestrator restart a fresh instance.

### Purposes

- To listen to Node.js execution events like `uncaughtException` and `unhandledRejection`.
- To close ongoing database connections and release resources during shutdown.
- To terminate the process safely via orchestration layers (PM2, Docker, Kubernetes).
- To prevent corrupted state from persisting after an unexpected error.
- To ensure zero-downtime deployments with graceful shutdown on `SIGTERM`.

### Syntax Rules and Structure

```typescript
// Graceful shutdown handler
const server = app.listen(3000);

async function gracefulShutdown(signal: string) {
  console.log(`Received ${signal}, shutting down gracefully`);

  // Stop accepting new connections
  server.close(async () => {
    console.log('HTTP server closed');

    // Close database connections
    await db.close();

    // Close other resources (Redis, message queues)
    await redis.quit();

    console.log('Cleanup complete, exiting');
    process.exit(0);
  });

  // Force exit after timeout
  setTimeout(() => {
    console.error('Forced shutdown after timeout');
    process.exit(1);
  }, 10_000);
}

// Process signal handlers
process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
process.on('SIGINT', () => gracefulShutdown('SIGINT'));

// Process-level error handlers
process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err);
  gracefulShutdown('uncaughtException');
});

process.on('unhandledRejection', (reason) => {
  console.error('Unhandled Rejection:', reason);
  gracefulShutdown('unhandledRejection');
});
```

| Component | Breakdown |
|-----------|-----------|
| `SIGTERM` | Sent by orchestrators to request termination. |
| `SIGINT` | Sent by Ctrl+C in the terminal. |
| `server.close()` | Stops accepting new connections, waits for existing. |
| `db.close()` | Closes database connection pool. |
| `process.exit(0)` | Exits with success code. |
| `uncaughtException` | Catches synchronous errors that escaped all handlers. |
| `unhandledRejection` | Catches rejected Promises with no `.catch()`. |

**Rules:**
- **Never** try to resume normal operation after an `uncaughtException` — the process state is unknown.
- Always perform **synchronous** cleanup in `uncaughtException` (async cleanup may not complete).
- Set a **force-exit timeout** (e.g., 10 seconds) to prevent hung processes.
- Kubernetes sends `SIGTERM` before killing a pod; PM2 uses `SIGUSR2` for graceful reload.
- Node.js 22 crashes the process on unhandled promise rejections by default.

**Constraints:**
- `uncaughtException` handlers cannot reliably perform async operations — use synchronous cleanup.
- Kubernetes sends `SIGTERM` then waits for a grace period before `SIGKILL`.
- PM2 does not restart on unhandled promise rejections by default (unless using a fork that does).

### Annotated Code Example

```typescript
// server.ts
import express from 'express';
import { Pool } from 'pg';

const app = express();
const db = new Pool({ connectionString: process.env.DATABASE_URL });

app.get('/health', (req, res) => {
  res.json({ status: 'ok', uptime: process.uptime() });
});

const server = app.listen(3000, () => {
  console.log('Server listening on port 3000');
});

// Graceful shutdown
async function shutdown(signal: string) {
  console.log(`[${signal}] Shutting down gracefully...`);

  server.close(async () => {
    console.log('HTTP server closed');
    await db.end();
    console.log('Database pool closed');
    process.exit(0);
  });

  setTimeout(() => {
    console.error('Forced shutdown after 10s timeout');
    process.exit(1);
  }, 10_000);
}

process.on('SIGTERM', () => shutdown('SIGTERM'));
process.on('SIGINT', () => shutdown('SIGINT'));

process.on('uncaughtException', (err) => {
  console.error('UNCAUGHT EXCEPTION:', err.message);
  shutdown('uncaughtException');
});

process.on('unhandledRejection', (reason) => {
  console.error('UNHANDLED REJECTION:', reason);
  shutdown('unhandledRejection');
});
```

**Expected Output (on `SIGTERM`):**
```
[SIGTERM] Shutting down gracefully...
HTTP server closed
Database pool closed
(process exits with code 0)
```

**Expected Output (on `uncaughtException`):**
```
UNCAUGHT EXCEPTION: Cannot read property 'foo' of undefined
[uncaughtException] Shutting down gracefully...
HTTP server closed
Database pool closed
(process exits with code 0; orchestrator restarts the process)
```

**Why this output:** The shutdown handler stops accepting new connections, waits for in-flight requests to complete, closes the database pool, and exits cleanly. The orchestrator (PM2, Kubernetes) detects the exit and starts a fresh instance. This prevents corrupted state from persisting.

### Real-World Cases

- **Kubernetes deployments:** `SIGTERM` is sent before pod termination; graceful shutdown ensures zero dropped requests.
- **PM2 cluster mode:** Workers are restarted gracefully with `SIGUSR2`.
- **Database maintenance:** Closing connection pools prevents "too many connections" errors during rolling restarts.
- **Docker Swarm:** `docker stop` sends `SIGTERM` and waits 10 seconds before `SIGKILL`.

---

## References

- Express.js — Error Handling Guide — https://expressjs.com/en/guide/error-handling.html
- Express.js — Production Best Practices: Security — https://expressjs.com/en/advanced/best-practice-security.html
- Node.js — Asynchronous Context Tracking (`AsyncLocalStorage`) — https://nodejs.org/api/async_context.html
- Node.js — Process Events (`uncaughtException`, `unhandledRejection`) — https://nodejs.org/api/process.html#process_event_uncaughtexception
- Pino Documentation — https://getpino.io/
- pino-http — npm — https://www.npmjs.com/package/pino-http
- Winston — npm — https://www.npmjs.com/package/winston
- Winston vs Pino: Choosing a Node.js Logger in 2026 — DevHelm — https://devhelm.io/blog/winston-vs-pino
- Logging in Express.js with Pino (Complete Guide) — Dash0 — https://www.dash0.com/guides/expressjs-logging
- Sentry for Node.js — https://docs.sentry.io/platforms/node/
- Sentry — Express Integration — https://docs.sentry.io/platforms/node/guides/express/
- Sentry — Use Your Own OpenTelemetry Pipeline — https://docs.sentry.io/platforms/node/opentelemetry/use-your-own-pipeline/
- How to Assign Unique IDs to Express API Requests for Tracing — freeCodeCamp — https://www.freecodecamp.org/news/how-to-assign-unique-ids-to-express-api-requests-for-tracing/
- pino-correlation-id — GitHub — https://github.com/axiom-experiment/pino-correlation-id
- express-http-context — GitHub — https://github.com/skonves/express-http-context
- How to handle process signals in Node.js — CoreUI — https://coreui.io/answers/how-to-handle-process-signals-in-nodejs/
- How to Handle Uncaught Exceptions and Unhandled Promise Rejections in Node.js — VernalWeb — https://help.vernalweb.com/kb/how-to-handle-uncaught-exceptions-unhandled-rejections-nodejs/
- PM2 — Graceful Shutdown — https://pm2.keymetrics.io/docs/usage/signals-clean-restart/
- Kubernetes — Pod Lifecycle (Termination) — https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination
- Node.js Best Practices — Distinguish Operational vs Programmer Errors — https://github.com/goldbergyoni/nodebestpractices
- OWASP — Error Handling Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html
- RFC 9457 — Problem Details for HTTP APIs — https://www.rfc-editor.org/rfc/rfc9457