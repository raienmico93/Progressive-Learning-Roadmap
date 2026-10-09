# Production Logging — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Production logging is the practice of emitting structured, machine-readable, correlated, and redacted log events from a deployed application so that operators can search, aggregate, alert on, and diagnose issues across distributed systems without exposing sensitive data or degrading application performance. 

**Technical Definition:** Production logging in Node.js encompasses: emitting newline-delimited JSON (NDJSON) via a high-performance logger such as Pino; attaching a unique request identifier (`requestId`) and a cross-service correlation identifier (`traceId`) to every log line; propagating request-scoped context through asynchronous call chains using `AsyncLocalStorage`; shipping logs to a centralised aggregation platform (Elasticsearch, Loki, Datadog, CloudWatch); redacting credentials and PII at the logger boundary; offloading I/O to worker threads so the main event loop is never blocked; and applying sampling and rate-limiting rules to prevent log flooding during traffic spikes. 

**Beginner-Friendly Explanation:** In development, you write `console.log('User logged in')` and read it in your terminal. In production, you have hundreds of servers, thousands of requests per second, and no terminal to watch. Production logging means every log line is a JSON object with a unique ID so you can find it later, a correlation ID so you can trace a request across services, redaction so passwords never appear, and a worker thread so logging never slows down your app. 

### Key Characteristics

- **Structured by default:** JSON logs with consistent field names are searchable and aggregatable.
- **Correlated across services:** `requestId` and `traceId` appear on every log line, enabling end-to-end tracing.
- **Context-propagated automatically:** `AsyncLocalStorage` carries request context through promises, callbacks, and event emitters without prop-drilling. 
- **Redacted at the boundary:** Credentials, tokens, and PII are censored before serialisation.
- **Non-blocking:** Pino transports run in worker threads, keeping the main event loop free. 
- **Sampled under load:** Healthy traffic is sampled; errors and slow requests are always logged.
- **Shipped to a central platform:** Logs are aggregated in ELK, Loki, Datadog, or CloudWatch for search and alerting. 

### Prerequisites

- **Node.js runtime** (v18 or higher; Pino v10 requires Node.js 20+).
- **Pino installed:** `npm install pino pino-http`.
- **For development pretty-printing:** `npm install pino-pretty`.
- **For context propagation:** Node.js built-in `node:async_hooks` (`AsyncLocalStorage`).
- **For redaction:** Pino's built-in `redact` option. 
- **A centralised logging platform** (Elasticsearch, Loki, Datadog, CloudWatch) for production.

### Related Programming Areas

- **Observability:** Logging, metrics, and tracing form the three pillars. 
- **Distributed tracing:** `traceId` and `spanId` propagate through HTTP headers (W3C Trace Context).
- **Security:** Redaction and audit logging prevent credential leakage.
- **Performance:** Asynchronous logging prevents the event loop from blocking. 
- **Compliance:** GDPR, HIPAA, and PCI-DSS mandate PII redaction and retention policies.

### Core Concepts

1. **Request IDs** — unique per-request identifiers.
2. **Correlation IDs** — cross-service trace identifiers.
3. **JSON Logs** — structured, machine-parseable output.
4. **Centralised Log Collection** — shipping to ELK, Loki, Datadog, CloudWatch.
5. **Sensitive-Data Redaction** — censoring credentials and PII.
6. **Asynchronous Logging** — Pino worker threads to prevent blocking.
7. **Context Propagation via AsyncLocalStorage** — request-scoped context without prop-drilling.
8. **Sampling and Rate-Limiting Logs** — preventing log flooding.

---

## Core Concept 1: Request IDs

### Definitions

**Core Definition:** A request ID is a unique identifier generated for each incoming HTTP request, attached to every log line produced while handling that request, and returned to the client in a response header. 

**Technical Definition:** The request ID is typically a UUID v4 or a 16-byte hex string generated in middleware at the entry point of the application. It is stored in `AsyncLocalStorage` for automatic propagation and attached to every log line via a Pino child logger or a Pino `mixin`. The ID is also set on the response as `X-Request-Id`, allowing clients to reference it in support requests. If the client provides an `X-Request-Id` header, the server should reuse it; otherwise, it generates a new one. 

**Beginner-Friendly Explanation:** A request ID is a unique "ticket number" for every request. If a user reports a problem, they can give you the ticket number, and you can search your logs for that exact ID to see everything that happened during that request.

### Purposes

- To uniquely identify each request for log correlation and debugging.
- To enable support teams to locate the exact logs for a user-reported issue.
- To trace a request through middleware, controllers, and database calls.
- To propagate to downstream services via the `X-Request-Id` header.

### Syntax Rules and Structure

```js
const { randomUUID } = require('node:crypto');

// Express middleware
app.use((req, res, next) => {
  req.id = req.headers['x-request-id'] || randomUUID();
  res.setHeader('X-Request-Id', req.id);
  next();
});
```

| Component | Breakdown |
|-----------|-----------|
| `req.headers['x-request-id']` | Client-provided request ID (if any). |
| `randomUUID()` | Generates a new UUID v4 if none provided. |
| `res.setHeader('X-Request-Id', req.id)` | Returns the ID to the client. |

**Rules:**
- Always check for a client-provided `X-Request-Id` before generating a new one. 
- Return the request ID in the `X-Request-Id` response header.
- Store the ID in `AsyncLocalStorage` for automatic propagation.
- Use UUID v4 or a 16-byte hex string for uniqueness. 

### Annotated Code Example

```js
// request-id-middleware.js
const { randomUUID } = require('node:crypto');
const { AsyncLocalStorage } = require('node:async_hooks');

const asyncLocalStorage = new AsyncLocalStorage();

function requestIdMiddleware(req, res, next) {
  const requestId = req.headers['x-request-id'] || randomUUID();
  res.setHeader('X-Request-Id', requestId);

  // Run the rest of the request inside the context
  asyncLocalStorage.run({ requestId }, () => {
    next();
  });
}

// Access the request ID anywhere
function getRequestId() {
  return asyncLocalStorage.getStore()?.requestId;
}

module.exports = { requestIdMiddleware, getRequestId };
```

**Expected Output (response headers):**
```
X-Request-Id: 550e8400-e29b-41d4-a716-446655440000
```

**Why this output:** The middleware checks for a client-provided `X-Request-Id` header. If absent, it generates a UUID v4. The ID is returned in the response header and stored in `AsyncLocalStorage`, making it available to any function called during the request without passing it as a parameter.

### Real-World Cases

- **Customer support:** A user reports "I got an error at 3:42 PM." The support agent searches for the user's `X-Request-Id` in the log platform. 
- **API debugging:** A client includes the `X-Request-Id` from a failed response in a bug report; the developer searches for it in Kibana or Grafana. 
- **Distributed tracing:** The request ID becomes the `traceId` that propagates through microservices. 

---

## Core Concept 2: Correlation IDs

### Definitions

**Core Definition:** A correlation ID is an identifier that is propagated across multiple services handling a single logical request, enabling log correlation across distributed system boundaries.

**Technical Definition:** Correlation IDs (also called `traceId` in W3C Trace Context) are generated at the entry point of a distributed system — typically an API gateway or the first service — and propagated to downstream services via HTTP headers (`X-Correlation-Id`, `X-Request-Id`, or `traceparent`). Each service extracts the incoming correlation ID, includes it in its logs, and forwards it on outgoing requests. All logs across all services for a single logical request share the same correlation ID. 

**Beginner-Friendly Explanation:** A correlation ID is like a tracking number for a package that passes through multiple delivery depots. Every depot scans the number, so you can see the package's complete journey. In a microservices system, the correlation ID lets you see a request's journey through every service. 

### Purposes

- To trace a single request across multiple services.
- To correlate logs from different services in a centralised platform.
- To measure end-to-end latency across service boundaries.
- To diagnose failures that span multiple services.

### Syntax Rules and Structure

```js
// Propagate correlation ID to downstream services
const traceId = getContext()?.traceId || randomUUID();

await fetch('https://downstream-service/api/data', {
  headers: {
    'X-Correlation-Id': traceId,
    'traceparent': `00-${traceId}-${spanId}-01`  // W3C format
  }
});
```

| Header | Purpose |
|--------|---------|
| `X-Correlation-Id` | Custom correlation header. |
| `X-Request-Id` | Alternative request ID header. |
| `traceparent` | W3C Trace Context standard. |

**Rules:**
- Generate the correlation ID at the entry point (gateway or first service).
- Extract it from incoming headers in every downstream service.
- Include it in every log line via `AsyncLocalStorage` or a child logger.
- Forward it on all outgoing requests (HTTP, gRPC, message queues). 
- Use W3C Trace Context (`traceparent`) for standard compliance. 

### Annotated Code Example

```js
// correlation-id-propagation.js
const { AsyncLocalStorage } = require('node:async_hooks');
const { randomUUID } = require('node:crypto');

const contextStorage = new AsyncLocalStorage();

// Middleware: extract or generate correlation ID
function correlationMiddleware(req, res, next) {
  const traceId = req.headers['x-correlation-id'] || req.headers['traceparent']?.split('-')[1] || randomUUID();
  const spanId = randomBytes(8).toString('hex');

  contextStorage.run({ traceId, spanId, requestId: randomUUID() }, () => {
    res.setHeader('X-Correlation-Id', traceId);
    next();
  });
}

// Propagate to downstream service
async function callDownstream(url) {
  const ctx = contextStorage.getStore();
  const response = await fetch(url, {
    headers: {
      'X-Correlation-Id': ctx.traceId,
      'traceparent': `00-${ctx.traceId}-${ctx.spanId}-01`
    }
  });
  return response.json();
}
```

**Expected Output (downstream service logs):**
```json
{"level":"info","traceId":"abc123def456","spanId":"a1b2c3d4","msg":"Request received"}
```

**Why this output:** The middleware extracts the correlation ID from the incoming `X-Correlation-Id` or W3C `traceparent` header. It generates a new `spanId` for this service and stores both in `AsyncLocalStorage`. When calling a downstream service, the correlation ID and span ID are forwarded, allowing the downstream service to continue the trace.

### Real-World Cases

- **E-commerce checkout:** A checkout request touches the cart service, inventory service, payment service, and order service — all sharing the same correlation ID. 
- **Banking transfers:** A transfer touches the account service, fraud service, and ledger service; the correlation ID links all logs. 
- **Microservices debugging:** Finding the root cause of a failure that cascades through three services. 

---

## Core Concept 3: JSON Logs

### Definitions

**Core Definition:** JSON logging emits each log entry as a JSON object — typically newline-delimited (NDJSON) — with consistent, named fields rather than a free-form string.

**Technical Definition:** Pino outputs NDJSON by default: one JSON object per line. Each entry includes `level` (numeric), `time` (epoch milliseconds or ISO string), `pid`, `hostname`, and `msg` (the message). Additional fields — `requestId`, `traceId`, `userId`, `orderId` — are included as top-level keys. JSON logs are parseable by ELK, Loki, Datadog, and CloudWatch without custom grok patterns or regex parsing. 

**Beginner-Friendly Explanation:** Instead of writing "User 456 placed order 123 for $50", a JSON log writes `{"level":"info","userId":"456","orderId":"123","amount":50,"msg":"Order placed"}`. Now your log platform can search for all orders by a specific user, sum the amounts, or alert when `amount` exceeds a threshold — all without writing custom parsers.

### Purposes

- To enable machine-parseable, searchable, and aggregatable logs.
- To include consistent fields across all services and teams.
- To integrate seamlessly with log aggregation platforms.
- To support structured queries (e.g., "find all logs where `userId` is `456`"). 

### Syntax Rules and Structure

```js
const logger = pino({
  formatters: {
    level: (label) => ({ level: label })  // "info" instead of 30
  },
  timestamp: pino.stdTimeFunctions.isoTime
});

logger.info({ orderId: 'ord_123', userId: 'usr_456' }, 'Order created');
```

**Example NDJSON output:**
```json
{"level":"info","time":"2026-06-07T12:00:00.000Z","orderId":"ord_123","userId":"usr_456","msg":"Order created"}
```

| Field | Source | Purpose |
|-------|--------|---------|
| `level` | Pino (configurable as string or number) | Severity. |
| `time` | Pino timestamp function | When the event occurred. |
| `msg` | Second argument to logger | Human-readable message. |
| `orderId`, `userId` | First argument (object) | Searchable context. |

**Rules:**
- Use JSON format for production; use `pino-pretty` for development only. 
- Never embed variable data in the message string — use structured fields. 
- Use consistent field names across services (e.g., always `userId`, never `user_id` or `uid`). 
- Include `service` and `environment` in `base` for multi-service aggregation. 

### Annotated Code Example

```js
// json-logging.js
const pino = require('pino');

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  base: { service: 'order-service', environment: process.env.NODE_ENV },
  timestamp: pino.stdTimeFunctions.isoTime,
  formatters: {
    level: (label) => ({ level: label })
  }
});

logger.info({ orderId: 'ord_123', userId: 'usr_456', amount: 99.99 }, 'Order created');
logger.warn({ responseTime: 3200, endpoint: '/api/orders' }, 'Slow request');
logger.error({ err: new Error('Payment failed'), orderId: 'ord_123' }, 'Payment processing error');
```

**Expected Output (NDJSON):**
```json
{"level":"info","time":"2026-06-07T12:00:00.000Z","service":"order-service","environment":"production","orderId":"ord_123","userId":"usr_456","amount":99.99,"msg":"Order created"}
{"level":"warn","time":"2026-06-07T12:00:01.000Z","service":"order-service","environment":"production","responseTime":3200,"endpoint":"/api/orders","msg":"Slow request"}
{"level":"error","time":"2026-06-07T12:00:02.000Z","service":"order-service","environment":"production","err":{"type":"Error","message":"Payment failed","stack":"..."},"orderId":"ord_123","msg":"Payment processing error"}
```

**Why this output:** Each line is a valid JSON object. `service` and `environment` are automatically included via `base`. `orderId`, `userId`, and `amount` are top-level fields, searchable in any aggregation platform. Errors are serialised with `err` key, preserving the stack trace. 

### Real-World Cases

- **ELK Stack:** JSON logs are ingested by Logstash and indexed by Elasticsearch without custom parsing.
- **Grafana Loki:** JSON logs are stored with label extraction for fast queries.
- **Datadog:** JSON logs enable facet search by `userId`, `orderId`, or any field.
- **CloudWatch Logs Insights:** JSON logs support structured queries with `filter userId = '456'`.

---

## Core Concept 4: Centralised Log Collection

### Definitions

**Core Definition:** Centralised log collection is the practice of shipping logs from all application instances to a single platform where they can be searched, filtered, aggregated, and alerted on.

**Technical Definition:** In production, logs are written to `stdout` as NDJSON. A log collector (Fluent Bit, Fluentd, Vector, Promtail) reads from the container's stdout, enriches the logs with metadata (pod name, namespace, region), and forwards them to a central platform: Elasticsearch (with Kibana for visualisation), Grafana Loki (with Grafana for dashboards), Datadog, or AWS CloudWatch. Pino can also ship logs directly via its transport mechanism to a remote endpoint. 

**Beginner-Friendly Explanation:** Each of your servers writes logs to its own stdout. A collector reads those logs, tags them with which server they came from, and sends them to one central place — like a library — where you can search across all servers at once. 

### Purposes

- To aggregate logs from all services and instances into one searchable platform.
- To enable cross-service correlation and tracing.
- To provide dashboards and alerting based on log content.
- To retain logs for compliance and auditing.

### Syntax Rules and Structure

**Deployment model:**
```
Application (Pino JSON to stdout)
  → Container runtime captures stdout
  → Fluent Bit / Promtail reads container logs
  → Forwards to Loki / Elasticsearch / Datadog / CloudWatch
```

**Pino direct transport:**
```js
const logger = pino({
  transport: {
    target: 'pino-loki',  // or 'pino-elasticsearch', '@chisme/pino-cloudwatch'
    options: {
      host: 'http://loki:3100',
      labels: { service: 'order-service' }
    }
  }
});
```

| Platform | Collector | Best For |
|----------|-----------|----------|
| ELK | Logstash / Fluent Bit | Full-text search, complex queries. |
| Loki + Grafana | Promtail | Lightweight, cloud-native. |
| Datadog | Datadog Agent | Managed, APM integration. |
| CloudWatch | CloudWatch Agent | AWS-native. |

**Rules:**
- Log to `stdout` — let the platform capture and ship. 
- Use structured JSON — aggregators parse JSON natively. 
- Enrich with metadata (service, environment, pod name) via `base` or collector labels. 
- For direct shipping, use Pino transports (worker thread, non-blocking). 
- Set retention policies: 30 days for app logs, longer for audit logs. 

### Annotated Code Example

```js
// centralised-logging.js
const pino = require('pino');

// Development: pretty-print to console
// Production: JSON to stdout, collected by platform
const isProd = process.env.NODE_ENV === 'production';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  base: {
    service: 'order-service',
    environment: process.env.NODE_ENV,
    version: process.env.APP_VERSION
  },
  transport: isProd
    ? undefined  // JSON to stdout; platform collects
    : { target: 'pino-pretty', options: { colorize: true } }
});

logger.info({ orderId: 'ord_123' }, 'Order created');
```

**Expected Output (production stdout):**
```json
{"level":30,"time":1737000000000,"service":"order-service","environment":"production","version":"1.0.0","orderId":"ord_123","msg":"Order created"}
```

**Expected Output (Loki query in Grafana):**
```
{service="order-service"} |= "orderId=ord_123"
```

**Why this output:** In production, the logger writes raw JSON to stdout. The container runtime captures stdout, and Promtail ships it to Loki with labels (`service="order-service"`). In Grafana, you can query all logs for a specific `orderId` across all instances of the service. 

### Real-World Cases

- **Kubernetes:** Fluent Bit DaemonSet collects logs from all pods and ships to Elasticsearch. 
- **AWS ECS:** CloudWatch Logs captures stdout from all tasks. 
- **Grafana Loki:** Promtail ships logs from Kubernetes pods; Grafana provides dashboards and alerting. 
- **Multi-cloud:** Datadog Agent collects logs from all instances and provides APM integration. 

---

## Core Concept 5: Sensitive-Data Redaction

### Definitions

**Core Definition:** Sensitive-data redaction is the process of censoring credentials, tokens, passwords, credit card numbers, and PII before they are written to logs.

**Technical Definition:** Pino provides a built-in `redact` option that uses `fast-redact` to censor specified paths in the log object. Paths use dot notation and support wildcards (e.g., `*.password`, `req.headers.authorization`). The `censor` option specifies the replacement string (default `[Redacted]`). Redaction occurs before serialisation, so sensitive data never reaches the transport or the log aggregator. 

**Beginner-Friendly Explanation:** Redaction is like blacking out sensitive information on a document before filing it. If a password or API key accidentally appears in a log line, redaction replaces it with `[REDACTED]` so it never ends up in your log platform. 

### Purposes

- To prevent credentials, tokens, and PII from appearing in logs.
- To comply with GDPR, HIPAA, and PCI-DSS requirements.
- To protect against accidental exposure in log aggregators and monitoring tools.
- To enforce consistent redaction across all services.

### Syntax Rules and Structure

```js
const logger = pino({
  redact: {
    paths: [
      'req.headers.authorization',
      'req.headers.cookie',
      '*.password',
      '*.token',
      '*.apiKey',
      '*.secret',
      'user.ssn',
      'user.creditCard'
    ],
    censor: '[REDACTED]'
  }
});
```

| Path Pattern | Matches |
|-------------|---------|
| `req.headers.authorization` | Authorization header. |
| `*.password` | Any field named `password` one level deep. |
| `*.token` | Any field named `token`. |
| `user.ssn` | Social Security Number. |

**Rules:**
- Redact **before** the data reaches the transport. 
- Use wildcards (`*.password`) to catch fields wherever they appear. 
- Never log entire request or response objects without redaction. 
- Treat the redaction config as security-critical — review it whenever new fields are added. 
- Log explicit field objects rather than whole request bodies. 

### Annotated Code Example

```js
// redaction.js
const pino = require('pino');

const logger = pino({
  redact: {
    paths: [
      'req.headers.authorization',
      'req.headers.cookie',
      '*.password',
      '*.token',
      '*.apiKey',
      '*.secret',
      'user.ssn',
      'user.creditCard',
      'body.password',
      'body.creditCard'
    ],
    censor: '[REDACTED]'
  }
});

// Safe: only logs non-sensitive fields
logger.info({ userId: 42, method: 'oauth' }, 'User login');

// Unsafe object is redacted automatically
logger.info({
  req: { headers: { authorization: 'Bearer secret-token' } },
  user: { ssn: '123-45-6789', creditCard: '4111-1111-1111-1111' }
}, 'User data');
```

**Expected Output:**
```json
{"level":30,"userId":42,"method":"oauth","msg":"User login"}
{"level":30,"req":{"headers":{"authorization":"[REDACTED]"}},"user":{"ssn":"[REDACTED]","creditCard":"[REDACTED]"},"msg":"User data"}
```

**Why this output:** The first log line contains no sensitive fields. The second log line contains `authorization`, `ssn`, and `creditCard` — all censored to `[REDACTED]` before serialisation. The sensitive values never appear in the output. 

### Real-World Cases

- **Authentication services:** Redacting `authorization` headers and `password` fields.
- **Payment processing:** Redacting credit card numbers and CVV codes.
- **Healthcare:** Redacting patient SSNs and medical record numbers.
- **GDPR compliance:** Redacting email addresses and phone numbers where required.

---

## Core Concept 6: Asynchronous Logging (Pino)

### Definitions

**Core Definition:** Asynchronous logging uses worker threads or buffered writes to move log I/O off the main event loop, preventing logging from blocking request processing.

**Technical Definition:** Pino v7+ supports transports that run in worker threads. When you configure `pino({ transport: { target: 'pino/file' } })`, the serialisation of log objects happens on the main thread (fast), but the actual I/O — writing to disk, sending over HTTP — happens in a worker thread. This keeps the main event loop free to handle requests. Pino can also use `pino.destination({ sync: false })` for buffered asynchronous writes without a worker thread. 

**Beginner-Friendly Explanation:** Writing to a file takes time. If your application writes to the log file on the main thread, it pauses handling requests while the disk write completes. Asynchronous logging moves the disk write to a separate thread, so your application never pauses. 

### Purposes

- To prevent logging from blocking the Node.js event loop.
- To maintain request throughput under high logging volume.
- To decouple log production from log consumption.
- To enable multiple destinations without blocking. 

### Syntax Rules and Structure

```js
const pino = require('pino');

// Worker-thread transport (non-blocking)
const logger = pino({
  transport: {
    target: 'pino/file',
    options: { destination: '/var/log/app.log' }
  }
});

// Multiple transports (different levels per destination)
const logger = pino({
  transport: {
    targets: [
      { target: 'pino/file', options: { destination: 1 }, level: 'info' },
      { target: 'pino/file', options: { destination: '/var/log/error.log' }, level: 'error' }
    ]
  }
});

// Async destination (buffered, no worker thread)
const logger = pino(pino.destination({ dest: '/var/log/app.log', sync: false }));
```

| Method | Blocking? | Use Case |
|--------|-----------|----------|
| `pino.transport()` | No (worker thread) | Production, multiple destinations. |
| `pino.destination({ sync: false })` | No (buffered) | Single destination, lower overhead. |
| Default (stdout) | Minimal (stdout is fast) | Containers, platform collects. |

**Rules:**
- Use `pino.transport()` for file writes and external shipping. 
- Never use `pino-pretty` in production — it blocks the main thread. 
- `pino.destination({ sync: false })` is a lighter alternative for a single destination. 
- Pin transport packages and vet them — they run with your application's privileges. 

### Annotated Code Example

```js
// async-logging.js
const pino = require('pino');

const isProd = process.env.NODE_ENV === 'production';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: isProd
    ? {
        targets: [
          {
            target: 'pino/file',
            options: { destination: 1 },  // stdout
            level: 'info'
          },
          {
            target: 'pino/file',
            options: { destination: '/var/log/app-error.log' },
            level: 'error'
          }
        ]
      }
    : {
        target: 'pino-pretty',
        options: { colorize: true }
      }
});

// These calls never block the event loop
logger.info({ requestId: 'abc-123' }, 'Request received');
logger.error({ err: new Error('DB timeout') }, 'Database error');
```

**Expected Output (production):**
```
# stdout (captured by platform)
{"level":30,"requestId":"abc-123","msg":"Request received"}

# /var/log/app-error.log (worker thread write)
{"level":50,"err":{"type":"Error","message":"DB timeout"},"msg":"Database error"}
```

**Why this output:** The `info`-level log is written to stdout (fast). The `error`-level log is written to a file by the worker thread. Neither write blocks the main event loop. In development, `pino-pretty` formats logs for readability, but it is not used in production. 

### Real-World Cases

- **High-traffic APIs:** 50,000+ logs per second without event-loop lag. 
- **File-heavy deployments:** Writing to disk without blocking request handlers.
- **Multi-destination routing:** Sending errors to a file and all logs to stdout simultaneously.

---

## Core Concept 7: Context Propagation via AsyncLocalStorage

### Definitions

**Core Definition:** `AsyncLocalStorage` is a Node.js built-in API that maintains request-scoped context across asynchronous operations — promises, callbacks, and event emitters — without explicitly passing context as a parameter.

**Technical Definition:** `AsyncLocalStorage` creates a storage instance that is available throughout the lifetime of an async operation. When you call `asyncLocalStorage.run(context, fn)`, the `context` object is available via `asyncLocalStorage.getStore()` anywhere inside `fn` and any function it calls, even across `await`, `setTimeout`, and Promise chains. This enables attaching `requestId`, `traceId`, `userId`, and `tenantId` to every log line without prop-drilling — passing the context through every function signature. 

**Beginner-Friendly Explanation:** Normally, if a deeply nested function needs the request ID, you have to pass it down through every intermediate function. `AsyncLocalStorage` lets you "set it and forget it" — you store the request ID once at the start of the request, and any function can retrieve it, no matter how deep it is. 

### Purposes

- To attach request context to every log line without prop-drilling.
- To enable automatic correlation ID propagation.
- To carry user identity, tenant ID, and trace ID across async boundaries.
- To integrate with Pino child loggers for request-scoped logging.

### Syntax Rules and Structure

```js
const { AsyncLocalStorage } = require('node:async_hooks');
const asyncLocalStorage = new AsyncLocalStorage();

// Middleware: store context
app.use((req, res, next) => {
  const context = {
    requestId: req.headers['x-request-id'] || randomUUID(),
    traceId: req.headers['x-correlation-id'] || randomUUID(),
    userId: req.user?.id,
    tenantId: req.user?.tenantId
  };
  asyncLocalStorage.run(context, () => next());
});

// Access anywhere
function getContext() {
  return asyncLocalStorage.getStore();
}
```

| Method | Purpose |
|--------|---------|
| `asyncLocalStorage.run(ctx, fn)` | Run `fn` with `ctx` as the current store. |
| `asyncLocalStorage.getStore()` | Retrieve the current context. |

**Rules:**
- Create a singleton `AsyncLocalStorage` instance per application.
- Store context in middleware at the entry point.
- Access context anywhere via `getStore()` — no prop-drilling.
- Context propagates automatically across `await`, callbacks, and event emitters. 
- Integrate with Pino via `logger.child()` or a `mixin`.

### Annotated Code Example

```js
// async-local-storage.js
const { AsyncLocalStorage } = require('node:async_hooks');
const { randomUUID } = require('node:crypto');
const pino = require('pino');

const asyncLocalStorage = new AsyncLocalStorage();

const logger = pino({
  mixin() {
    const ctx = asyncLocalStorage.getStore();
    if (!ctx) return {};
    return {
      requestId: ctx.requestId,
      traceId: ctx.traceId,
      userId: ctx.userId
    };
  }
});

// Express middleware
app.use((req, res, next) => {
  const context = {
    requestId: req.headers['x-request-id'] || randomUUID(),
    traceId: req.headers['x-correlation-id'] || randomUUID(),
    userId: req.user?.id
  };
  asyncLocalStorage.run(context, () => next());
});

// Deeply nested function — no context parameter needed
async function processOrder(orderId) {
  logger.info({ orderId }, 'Processing order');
  await validateInventory(orderId);
  await chargePayment(orderId);
  logger.info({ orderId }, 'Order processed');
}

async function validateInventory(orderId) {
  // Context is automatically available
  logger.info({ orderId }, 'Inventory validated');
}
```

**Expected Output:**
```json
{"level":30,"requestId":"req-abc","traceId":"trace-xyz","userId":42,"orderId":"ord_123","msg":"Processing order"}
{"level":30,"requestId":"req-abc","traceId":"trace-xyz","userId":42,"orderId":"ord_123","msg":"Inventory validated"}
{"level":30,"requestId":"req-abc","traceId":"trace-xyz","userId":42,"orderId":"ord_123","msg":"Order processed"}
```

**Why this output:** The middleware stores `requestId`, `traceId`, and `userId` in `AsyncLocalStorage`. The Pino `mixin` function reads this store and includes the fields in every log line. `processOrder` and `validateInventory` do not receive any context parameters — yet their log lines contain the correct `requestId`, `traceId`, and `userId`. 

### Real-World Cases

- **Multi-tenant SaaS:** Every log line includes `tenantId` for tenant-specific log isolation.
- **Distributed tracing:** `traceId` and `spanId` are propagated without prop-drilling.
- **Audit logging:** `userId` is automatically attached to every log line for compliance.
- **Debugging:** Deeply nested async functions log with full request context.

---

## Core Concept 8: Sampling and Rate-Limiting Logs

### Definitions

**Core Definition:** Log sampling is the practice of logging only a percentage of high-volume, low-value events (e.g., 5% of healthy requests), while always logging errors and slow requests. Log rate-limiting caps the number of log entries emitted per unit of time to prevent log flooding.

**Technical Definition:** Sampling reduces log volume by dropping a configurable fraction of low-severity logs. The `widelog` library implements "wide event" sampling: it emits one structured log per request instead of many, samples 5% of normal requests, and always logs requests slower than a threshold (e.g., 500 ms). Rate-limiting, implemented by libraries like `pinito`, monitors event loop utilisation and dynamically throttles log output when the server is under high load. 

**Beginner-Friendly Explanation:** If every healthy request produces 10 log lines and you handle 10,000 requests per second, that is 100,000 log lines per second — most of them useless. Sampling says "only log 5% of healthy requests, but always log errors and slow requests." This keeps your logs useful and your costs down. 

### Purposes

- To prevent log flooding during high-traffic periods.
- To reduce log storage and ingestion costs.
- To preserve visibility into errors and performance issues.
- To maintain application performance when the event loop is under pressure.
- To avoid drowning important events in a sea of routine logs. 

### Syntax Rules and Structure

```js
const { createWideLogger } = require('widelog');
const { pinoAdapter } = require('widelog/adapters/pino');

const wideLogger = createWideLogger(pinoAdapter(pinoLogger), {
  sampling: {
    sampleRate: 0.05,              // 5% of normal requests
    slowRequestThresholdMs: 500,   // Always log requests > 500ms
    neverLogPaths: [/^\/health$/, /^\/metrics$/]  // Skip these endpoints
  }
});
```

| Option | Purpose |
|--------|---------|
| `sampleRate` | Fraction of normal requests to log (0.05 = 5%). |
| `slowRequestThresholdMs` | Always log requests slower than this. |
| `neverLogPaths` | Paths that are never logged (health checks). |

**Rules:**
- **Sample, but never sample errors.** Drop 90% of healthy `info` logs; keep 100% of `warn`, `error`, and `fatal`. 
- **Always log slow requests.** Set a threshold (e.g., 500 ms) and log every request that exceeds it. 
- **Skip health checks and metrics.** These endpoints generate noise with no diagnostic value. 
- **Rate-limit under load.** Libraries like `pinito` monitor event loop utilisation and throttle dynamically. 
- **Use wide events.** One structured log per request (with all context) is more useful than 10 scattered logs. 

### Annotated Code Example

```js
// sampling.js
const pino = require('pino');
const { createWideLogger, logContext } = require('widelog');
const { pinoAdapter } = require('widelog/adapters/pino');

const pinoLogger = pino({ level: 'info' });

const wideLogger = createWideLogger(pinoAdapter(pinoLogger), {
  sampling: {
    sampleRate: 0.05,              // 5% of normal requests
    slowRequestThresholdMs: 500,   // Always log slow requests
    neverLogPaths: [/^\/health$/, /^\/metrics$/]
  }
});

// Express middleware
app.use(wideLogger.middleware());

app.get('/api/users/:id', async (req, res) => {
  // Enrich the wide event with route-specific context
  logContext.set({
    eventName: 'get-user-profile',
    userId: req.params.id,
    operationType: 'read'
  });

  const user = await getUser(req.params.id);
  res.json(user);
});

app.get('/health', (req, res) => {
  res.json({ status: 'ok' });  // Never logged
});
```

**Expected Output (for a fast request — sampled at 5%):**
```json
// 95% of fast requests produce no log
// 5% produce:
{"level":"info","eventName":"get-user-profile","userId":"123","operationType":"read","durationMs":45,"msg":"request_completed"}
```

**Expected Output (for a slow request — always logged):**
```json
{"level":"info","eventName":"get-user-profile","userId":"123","operationType":"read","durationMs":1200,"msg":"request_completed","slow":true}
```

**Expected Output (for a health check — never logged):**
```
(no log output)
```

**Why this output:** The `sampleRate: 0.05` means only 5% of normal requests produce a log entry. The `slowRequestThresholdMs: 500` ensures that any request taking longer than 500 ms is always logged, regardless of sampling. Health checks and metrics endpoints are excluded via `neverLogPaths`. 

### Real-World Cases

- **High-traffic APIs:** 10,000 requests per second; sampling reduces log volume from 100,000 to 5,000 lines per second. 
- **Kubernetes clusters:** Health check probes generate thousands of logs per minute; `neverLogPaths` eliminates them. 
- **Incident response:** Slow requests are always visible even when healthy traffic is sampled. 
- **Cost optimisation:** Sampling reduces Datadog or CloudWatch ingestion costs by 90% while preserving error visibility. 

---

## References

- Structured Logging Best Practices (llm-knowledge-base) — https://github.com/glennguilloux/llm-knowledge-base/blob/main/patterns/structured-logging.md
- How to Implement Log Context Propagation (OneUptime Blog) — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-30-log-context-propagation/README.md
- Pino Logging Skill (oakoss/agent-skills) — https://github.com/oakoss/agent-skills/blob/main/skills/pino-logging/SKILL.md
- How Safe Is the npm pino Logger? A Security Review (Safeguard) — https://safeguard.sh/resources/blog/npm-pino
- Set Up Structured Logging with Pino and Log Aggregation (Healthy-Stellar Issue #158) — https://github.com/Healthy-Stellar/Healthy-Stellar-backend/issues/158
- widelog on npm — https://www.npmjs.com/package/widelog
- @dex-monit/observability-request-context on npm — https://www.npmjs.com/package/@dex-monit/observability-request-context
- @norialabs/logger on npm — https://www.npmjs.com/package/@norialabs/logger
- Pino Logger for Node.js: Setup, Configuration & Best Practices (Last9) — https://last9.io/blog/npm-pino-logger/
- Winston vs Pino: Choosing a Node.js Logger in 2026 (DevHelm) — https://devhelm.io/blog/winston-vs-pino
- structured-logging Skill (LobeHub) — https://lobehub.com/skills/structured-logging
- pinito on npm — https://www.npmjs.com/package/pinito
- pino-ctx on npm — https://www.npmjs.com/package/pino-ctx
- @jazim/logger on npm — https://www.npmjs.com/package/@jazim/logger
- Generate and propagate X-Request-ID for request correlation (sip-protocol Issue #649) — https://github.com/sip-protocol/sip-protocol/issues/649