# Logging — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Logging is the practice of recording discrete, timestamped events that occur during an application's execution — errors, warnings, informational messages, and debug details — to a durable medium (console, file, or external service) for the purposes of debugging, monitoring, auditing, and observability. 

**Technical Definition:** In Node.js, logging begins with the built-in `console` module, which writes to `process.stdout` (for `log`, `info`, `debug`) and `process.stderr` (for `warn`, `error`). As applications mature, they adopt structured logging libraries — Pino, Winston, or Bunyan — that emit machine-readable JSON (typically newline-delimited JSON, NDJSON) with consistent fields, severity levels, timestamps, and correlation identifiers. These logs are consumed by aggregation platforms (Elasticsearch, Loki, Datadog, CloudWatch) for search, alerting, and analysis. 

**Beginner-Friendly Explanation:** Logging is your application's diary. When something happens — a user logs in, a payment fails, a server crashes — your application writes it down. During development, you read the diary in your terminal. In production, those diary entries are collected, stored, and searched so you can figure out what went wrong at 3 AM without having to reproduce the problem yourself.

### Key Characteristics

- **Structured over unstructured:** JSON logs with consistent fields are machine-parseable and searchable; free-form strings are not. 
- **Level-based filtering:** Log levels (debug, info, warn, error, fatal) allow you to control verbosity per environment.
- **Transport abstraction:** Logs can be routed to multiple destinations — console, file, HTTP endpoint, syslog — without changing application code. 
- **Performance matters:** Pino serialises JSON logs 5–10× faster than Winston by offloading I/O to worker threads; Winston prioritises flexibility with 80+ community transports. 
- **Rotation and retention:** Log files must be rotated (daily or by size) and old files purged to prevent disk exhaustion.
- **Redaction by default:** Credentials, tokens, credit card numbers, and PII must never appear in logs. 

### Prerequisites

- **Node.js runtime** (v18 or higher).
- **Basic understanding of JavaScript modules and async/await.**
- **For Pino:** `npm install pino`, and for HTTP logging `npm install pino-http`, for pretty printing `npm install pino-pretty`.
- **For Winston:** `npm install winston`, and for rotation `npm install winston-daily-rotate-file`.
- **A log aggregation platform** (optional but recommended for production).

### Related Programming Areas

- **Observability:** Logging, metrics, and tracing form the three pillars of observability.
- **Error handling:** Logging is the primary mechanism for recording unhandled exceptions and rejected Promises.
- **Security:** Audit logging captures who did what and when, with tamper-evident properties. 
- **DevOps:** Log aggregation and alerting are core to production operations.
- **Compliance:** GDPR, HIPAA, and PCI-DSS impose requirements on log retention and PII redaction.

### Core Concepts

1. **Console Logging** — the built-in `console` module.
2. **Structured Logging** — JSON logs with consistent fields.
3. **Log Levels** — debug, info, warn, error, fatal.
4. **Log Rotation and Retention** — daily/size-based rotation with purging.
5. **Transport Configurations** — file vs. stdout vs. external services.

---

## Core Concept 1: Console Logging

### Definitions

**Core Definition:** Console logging uses the built-in `console` module — available in every Node.js program without import — to write messages to `process.stdout` or `process.stderr`.

**Technical Definition:** The `console` module provides methods `log`, `info`, `debug`, `warn`, and `error`. The first three write to `process.stdout`; `warn` and `error` write to `process.stderr`. `console.log` accepts multiple arguments, which are formatted with `util.format` and concatenated with spaces. Objects are inspected with `util.inspect`, which produces a human-readable representation. In production, stdout and stderr are often routed to different destinations by the process manager or container runtime. 

**Beginner-Friendly Explanation:** `console.log` is the simplest way to see what your code is doing. You write `console.log('Server started')` and the message appears in your terminal. It requires no setup, no dependencies, and no configuration. But it produces unstructured text that is difficult to search, filter, or aggregate at scale. 

### Purposes

- To provide immediate, zero-dependency visibility during development.
- To debug variable values, confirm function execution, and trace control flow.
- To separate normal output (stdout) from errors and warnings (stderr).
- To serve as a fallback when a logging library cannot be used.

### Syntax Rules and Structure

```js
console.log('Server is running');                 // stdout
console.info('User logged in', { userId: 42 });   // stdout
console.debug('Cache hit', { key: 'user:42' });   // stdout
console.warn('System idle for 5 minutes');        // stderr
console.error('Database connection failed');      // stderr
```

| Method | Stream | Typical Use |
|--------|--------|-------------|
| `console.log` | stdout | General messages. |
| `console.info` | stdout | Alias for `log`. |
| `console.debug` | stdout | Debug information. |
| `console.warn` | stderr | Warnings. |
| `console.error` | stderr | Errors. |

**Rules:**
- `console.log` and `console.error` write to different streams — use them accordingly. 
- Multiple arguments are space-separated in the output. 
- Objects are inspected with `util.inspect` (depth 2 by default). 
- Never use `console.log` for sensitive data (passwords, tokens).
- In production, replace `console.*` calls with a structured logger.

### Annotated Code Example

```js
// console-logging.js
console.log('Server is running');
console.log('User:', { name: 'olly', age: 21 });
console.error('Database connection failed');
console.warn('Warning: System idle for more than 5 min');
```

**Expected Output:**
```
Server is running
User: { name: 'olly', age: 21 }
Database connection failed
Warning: System idle for more than 5 min
```

**Why this output:** `console.log` writes to stdout with space-separated arguments. The object is inspected and printed in a readable format. `console.error` and `console.warn` write to stderr, which in a terminal appears interleaved but can be redirected separately. 

### Real-World Cases

- **Local development:** Quick inspection of variables and control flow.
- **CLI tools:** User-facing output that is part of the tool's UX. 
- **Early prototyping:** Before structured logging is configured.

---

## Core Concept 2: Structured Logging

### Definitions

**Core Definition:** Structured logging emits each log entry as a machine-readable JSON object with consistent, named fields rather than a free-form string, enabling programmatic search, filtering, and aggregation.

**Technical Definition:** A structured log entry is a JSON object containing at minimum a level, timestamp, and message, plus any contextual fields. Pino outputs newline-delimited JSON (NDJSON) by default: one JSON object per line. The fields include `level` (numeric), `time` (epoch milliseconds or ISO string), and `pid`/`hostname` (automatically added). Winston uses a composable format pipeline (`format.combine()`) to achieve the same result. Structured fields should be searchable keys — `{ userId: 'usr_456' }` — not embedded in the message string. 

**Beginner-Friendly Explanation:** Instead of writing "User 456 placed order ord_123 for $50", you write a JSON object: `{"level":"info","userId":"usr_456","orderId":"ord_123","amount":50,"msg":"Order placed"}`. Now a log aggregation tool can search for all orders by a specific user, sum the amounts, or alert when a field is missing. 

### Purposes

- To enable programmatic search, filtering, and aggregation of logs.
- To include correlation IDs (request ID, trace ID) in every log line for distributed tracing.
- To facilitate integration with log aggregation platforms (Elasticsearch, Loki, Datadog, CloudWatch).
- To provide consistent field names across services and teams.
- To support redaction of sensitive fields at the logging boundary.

### Syntax Rules and Structure

**Pino (recommended for performance):**
```js
import pino from 'pino';

const logger = pino({
  level: 'info',
  base: { service: 'order-service' },
  timestamp: pino.stdTimeFunctions.isoTime,
  redact: ['req.headers.authorization', 'password']
});

logger.info({ orderId: 'ord_123', userId: 'usr_456' }, 'Order created');
```

**Winston (flexibility and ecosystem):**
```js
import { createLogger, format, transports } from 'winston';

const logger = createLogger({
  level: 'info',
  format: format.combine(
    format.timestamp(),
    format.errors({ stack: true }),
    format.json()
  ),
  defaultMeta: { service: 'order-service' },
  transports: [new transports.Console()]
});

logger.info('Order created', { orderId: 'ord_123', userId: 'usr_456' });
```

| Library | Performance | Format | Transports | Best For |
|---------|-------------|--------|-----------|----------|
| **Pino** | 5–10× faster | JSON by default | Worker-thread transports | High-throughput services. |
| **Winston** | Good | Composable formats | 80+ community transports | Complex formatting needs. |
| **Bunyan** | Good | JSON by default | Built-in CLI | Legacy projects. |

**Rules:**
- Use JSON format for machine parsing; use `pino-pretty` for human-readable development output. 
- Include a `requestId` or `traceId` in every log line for correlation. 
- Never log passwords, tokens, credit card numbers, or PII. 
- Use `redact` (Pino) or a custom format (Winston) to mask sensitive fields. 
- Structured fields should be top-level keys, not embedded in the message.

### Annotated Code Example

```js
// structured-logging.js
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  base: { service: 'order-service', version: '1.0.0' },
  timestamp: pino.stdTimeFunctions.isoTime,
  redact: {
    paths: ['req.headers.authorization', 'body.password'],
    censor: '[REDACTED]'
  }
});

logger.info({ orderId: 'ord_123', userId: 'usr_456' }, 'Order created');
logger.warn({ responseTime: 3200 }, 'Slow request detected');
logger.error({ err: new Error('Payment failed') }, 'Payment processing error');
```

**Expected Output (NDJSON):**
```json
{"level":30,"time":"2026-06-07T12:00:00.000Z","service":"order-service","version":"1.0.0","orderId":"ord_123","userId":"usr_456","msg":"Order created"}
{"level":40,"time":"2026-06-07T12:00:01.000Z","service":"order-service","version":"1.0.0","responseTime":3200,"msg":"Slow request detected"}
{"level":50,"time":"2026-06-07T12:00:02.000Z","service":"order-service","version":"1.0.0","err":{"type":"Error","message":"Payment failed","stack":"..."},"msg":"Payment processing error"}
```

**Why this output:** Each line is a valid JSON object. The `level` field is numeric (30=info, 40=warn, 50=error). The `time` is ISO 8601. The `service` and `version` are automatically included via `base`. The `orderId` and `userId` are top-level fields, searchable by any log aggregation tool. The `redact` configuration ensures that `authorization` headers and `password` fields are never logged.

### Real-World Cases

- **Microservices:** Structured logs with `traceId` enable end-to-end request tracing across services. 
- **E-commerce:** Searching for all orders by a specific user across millions of log lines. 
- **Security auditing:** Capturing who accessed what resource and when, with tamper-evident properties. 
- **Compliance:** Redacting PII from logs to meet GDPR and HIPAA requirements.

---

## Core Concept 3: Log Levels

### Definitions

**Core Definition:** Log levels are named severity categories — debug, info, warn, error, fatal — assigned to each log entry to indicate its importance and urgency.

**Technical Definition:** The standard severity order (from most to least verbose) is: `trace` (10), `debug` (20), `info` (30), `warn` (40), `error` (50), `fatal` (60). A logger configured at a given level emits messages at that level and all higher-severity levels. For example, a logger set to `info` emits `info`, `warn`, `error`, and `fatal`, but suppresses `debug` and `trace`. Pino uses numeric values; Winston follows the same npm convention. 

**Beginner-Friendly Explanation:** Log levels let you control how much your application talks. In development, you want `debug` — everything. In production, you want `info` or higher — only important things. When something goes wrong, you can temporarily lower the level to see more detail without changing code.

### Purposes

- To control verbosity per environment (debug in dev, info in prod).
- To filter logs by severity during search and alerting.
- To prioritise operational attention: `fatal` demands immediate action; `warn` is a note for later.
- To reduce log volume and storage costs by suppressing low-severity logs in production.

### Syntax Rules and Structure

| Level | Numeric Value | Purpose | Example |
|-------|--------------|---------|---------|
| `fatal` | 60 | Service is going to stop or become unusable. | Uncaught exception; database connection lost. |
| `error` | 50 | Fatal for a particular request; service continues. | Payment processing failure; 500 response. |
| `warn` | 40 | Something that should be looked at eventually. | Slow request; deprecated API usage. |
| `info` | 30 | Detail on regular operation. | User logged in; order created. |
| `debug` | 20 | Verbose information for debugging. | Cache hit; query parameters. |
| `trace` | 10 | Very detailed execution tracing. | Function entry/exit; external library logs. |

**Rules:**
- Configure the level via `LOG_LEVEL` environment variable: `process.env.LOG_LEVEL || 'info'`. 
- Use `debug` in development and `info` in production. 
- Reserve `fatal` for unrecoverable crashes that trigger `process.exit`. 
- Never use `error` for expected conditions (e.g., 404 responses) — use `info` or `warn`. 
- Use `trace` sparingly — it is extremely verbose and typically only enabled temporarily.

### Annotated Code Example

```js
// log-levels.js
import pino from 'pino';

const logger = pino({ level: process.env.LOG_LEVEL || 'info' });

// Fatal — unrecoverable
logger.fatal('Database connection pool exhausted. Shutting down.');

// Error — request-level failure, service continues
logger.error({ err: new Error('Payment gateway timeout') }, 'Payment processing failed');

// Warn — something to watch
logger.warn({ responseTime: 3200, url: '/api/orders' }, 'Slow request detected');

// Info — regular operation
logger.info({ userId: 42 }, 'User logged in');

// Debug — suppressed in production (level: info)
logger.debug({ query: 'SELECT * FROM users' }, 'Executing query');

// Trace — suppressed in production
logger.trace({ fn: 'calculateTax' }, 'Function entered');
```

**Expected Output (with `LOG_LEVEL=info`):**
```json
{"level":60,"msg":"Database connection pool exhausted. Shutting down."}
{"level":50,"err":{"message":"Payment gateway timeout"},"msg":"Payment processing failed"}
{"level":40,"responseTime":3200,"url":"/api/orders","msg":"Slow request detected"}
{"level":30,"userId":42,"msg":"User logged in"}
```

**Expected Output (with `LOG_LEVEL=debug`):**
```json
{"level":60,"msg":"Database connection pool exhausted. Shutting down."}
{"level":50,"err":{"message":"Payment gateway timeout"},"msg":"Payment processing failed"}
{"level":40,"responseTime":3200,"url":"/api/orders","msg":"Slow request detected"}
{"level":30,"userId":42,"msg":"User logged in"}
{"level":20,"query":"SELECT * FROM users","msg":"Executing query"}
```

**Why this output:** With `LOG_LEVEL=info`, the `debug` and `trace` calls are suppressed. With `LOG_LEVEL=debug`, the `debug` call appears but `trace` is still suppressed. This allows developers to control verbosity without changing code — simply set an environment variable. 

### Real-World Cases

- **Development:** `LOG_LEVEL=debug` for maximum visibility.
- **Production:** `LOG_LEVEL=info` to reduce volume and cost.
- **Incident response:** Temporarily set `LOG_LEVEL=debug` on a single instance to diagnose an issue.
- **Alerting:** Configure alerts only for `error` and `fatal` levels. 

---

## Core Concept 4: Log Rotation and Retention

### Definitions

**Core Definition:** Log rotation is the process of moving the active log file to an archive and creating a new one, typically on a daily schedule or when a size threshold is reached. Retention is the policy that determines how long archived logs are kept before deletion.

**Technical Definition:** Winston's `winston-daily-rotate-file` transport rotates files based on a date pattern (`YYYY-MM-DD`), size limit (`maxSize: '20m'`), or both. Old files are compressed (`zippedArchive: true`) and removed after a retention period (`maxFiles: '14d'` retains 14 days). Pino does not include built-in rotation; it relies on external tools like `logrotate` (Linux) or the `pino-roll` transport. 

**Beginner-Friendly Explanation:** If you write every log entry to a single file, that file will grow forever and eventually fill your disk. Rotation splits logs into manageable chunks — one per day, or one per 20 MB — and retention deletes the oldest chunks after a set period. This keeps your logging sustainable without manual intervention.

### Purposes

- To prevent disk exhaustion from unbounded log growth.
- To make logs easier to search, archive, and manage.
- To comply with retention policies (e.g., 14 days for operational logs, 90 days for audit logs).
- To reduce storage costs by compressing old logs.

### Syntax Rules and Structure

**Winston with `winston-daily-rotate-file`:**
```js
const winston = require('winston');
require('winston-daily-rotate-file');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.DailyRotateFile({
      filename: 'logs/error-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      level: 'error',
      zippedArchive: true,
      maxSize: '20m',
      maxFiles: '14d'
    }),
    new winston.transports.DailyRotateFile({
      filename: 'logs/combined-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      zippedArchive: true,
      maxSize: '20m',
      maxFiles: '14d'
    })
  ]
});
```

| Option | Purpose | Example |
|--------|---------|---------|
| `filename` | Log file name with `%DATE%` placeholder. | `logs/error-%DATE%.log` |
| `datePattern` | Rotation frequency. | `YYYY-MM-DD` (daily). |
| `zippedArchive` | Compress old logs. | `true` |
| `maxSize` | Maximum size per file before rotation. | `'20m'` (20 MB). |
| `maxFiles` | Retention period. | `'14d'` (14 days). |
| `level` | Minimum level for this transport. | `'error'` |

**Rules:**
- Store logs outside the project directory — `/var/log/myapp/` is conventional. 
- Rotate daily and by size — either trigger creates a new file. 
- Compress old logs (`zippedArchive: true`) to save disk space. 
- Set `maxFiles` to match your retention policy. 
- For Pino, use `pino-roll` or external `logrotate` configuration.

### Annotated Code Example

```js
// log-rotation.js
const winston = require('winston');
require('winston-daily-rotate-file');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    // Error-level logs only
    new winston.transports.DailyRotateFile({
      filename: 'logs/error-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      level: 'error',
      zippedArchive: true,
      maxSize: '20m',
      maxFiles: '14d'
    }),
    // All logs
    new winston.transports.DailyRotateFile({
      filename: 'logs/combined-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      zippedArchive: true,
      maxSize: '20m',
      maxFiles: '14d'
    })
  ]
});

logger.info('Application started');
logger.error('Database connection failed');
```

**Expected Output (after 14 days of operation):**
```
logs/
├── combined-2026-06-01.log.gz
├── combined-2026-06-02.log.gz
├── ...
├── combined-2026-06-14.log
├── error-2026-06-01.log.gz
├── ...
└── error-2026-06-14.log
```

**Why this output:** Each day, a new file is created with the date in the name. When a file reaches 20 MB, it is also rotated. Files older than 14 days are automatically deleted. The `.gz` extension indicates compressed archives. The `error-` files contain only `error`-level logs; the `combined-` files contain all levels. 

### Real-World Cases

- **High-traffic APIs:** Rotating by size (20 MB) prevents huge single files.
- **Compliance:** Retaining audit logs for 90 days or 1 year depending on regulation.
- **Cost optimisation:** Compressing old logs reduces S3 or cold-storage costs.

---

## Core Concept 5: Transport Configurations

### Definitions

**Core Definition:** A transport is an output destination for log messages — console (stdout/stderr), file, HTTP endpoint, syslog, or a cloud service.

**Technical Definition:** Winston's transport architecture allows multiple simultaneous outputs with independent formats and levels. Pino uses a similar concept: `pino.transport()` spawns worker threads to handle I/O, keeping the main event loop free. The choice between file and stdout depends on deployment: bare-metal servers typically write to files with rotation; containers and serverless functions write to stdout and let the platform handle collection. 

**Beginner-Friendly Explanation:** You can send your logs to multiple places at once. During development, you want them in your terminal (stdout). In production, you might want them in a file (for local access) and also in a cloud service like Datadog or CloudWatch (for aggregation). Transports let you do both without changing your logging code.

### Purposes

- To route logs to the appropriate destination for the environment.
- To write to stdout for containerised deployments (Docker, Kubernetes).
- To write to files for bare-metal deployments with local retention.
- To send logs to HTTP endpoints, syslog, or cloud services for aggregation.
- To apply different formats and levels per transport.

### Syntax Rules and Structure

**Winston — multiple transports:**
```js
const logger = winston.createLogger({
  transports: [
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      )
    }),
    new winston.transports.File({ filename: 'logs/combined.log' }),
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.Http({ host: 'logs.example.com', port: 443 })
  ]
});
```

**Pino — transport with worker thread:**
```js
import pino from 'pino';

const logger = pino({
  transport: {
    targets: [
      { target: 'pino/file', options: { destination: '/var/log/app.log' } },
      { target: 'pino-pretty', options: { colorize: true }, level: 'debug' }
    ]
  }
});
```

| Transport | Destination | Use Case |
|-----------|-------------|----------|
| Console | stdout/stderr | Development; containers. |
| File | Disk file | Bare-metal; local retention. |
| HTTP | Remote endpoint | Log aggregation services. |
| Syslog | Syslog daemon | Traditional Unix infrastructure. |
| CloudWatch/Datadog | Cloud service | Managed aggregation. |

**Rules:**
- **Containers/Kubernetes:** Log to stdout; let the platform collect and route. 
- **Bare-metal:** Log to files with rotation; use `logrotate` or Winston's daily rotate.
- **Serverless:** Log to stdout (CloudWatch, Cloud Logging capture it automatically). 
- Use different levels per transport: `debug` to console, `info` to file, `error` to alerting.
- Pino transports run in worker threads — this keeps the main event loop fast. 

### Annotated Code Example

```js
// transport-config.js
import pino from 'pino';

const isProduction = process.env.NODE_ENV === 'production';

const logger = pino({
  level: process.env.LOG_LEVEL || (isProduction ? 'info' : 'debug'),
  transport: isProduction
    ? {
        targets: [
          // Production: write JSON to stdout for container collection
          { target: 'pino/file', options: { destination: 1 } },  // fd 1 = stdout
          // Also send errors to a file for local retention
          {
            target: 'pino/file',
            options: { destination: '/var/log/app-error.log' },
            level: 'error'
          }
        ]
      }
    : {
        // Development: pretty-print to console
        target: 'pino-pretty',
        options: { colorize: true, translateTime: 'HH:MM:ss' }
      }
});

logger.info({ userId: 42 }, 'User logged in');
logger.error({ err: new Error('DB timeout') }, 'Database error');
```

**Expected Output (development — `pino-pretty`):**
```
12:00:00 INFO  User logged in { userId: 42 }
12:00:01 ERROR Database error { err: Error: DB timeout }
```

**Expected Output (production — JSON to stdout):**
```json
{"level":30,"time":1737000000000,"userId":42,"msg":"User logged in"}
{"level":50,"time":1737000001000,"err":{"type":"Error","message":"DB timeout"},"msg":"Database error"}
```

**Why this output:** In development, `pino-pretty` formats logs as readable, colourised text. In production, the `pino/file` target writes raw JSON to file descriptor 1 (stdout), which the container runtime collects. A second target writes only `error`-level logs to a file on disk for local retention. The application code (`logger.info`, `logger.error`) is identical in both environments. 

### Real-World Cases

- **Kubernetes:** Log to stdout; the cluster's log agent (Fluentd, Vector) ships to Elasticsearch or Loki.
- **AWS Lambda:** Log to stdout; CloudWatch Logs captures automatically.
- **Bare-metal Nginx/Node:** Log to files with `logrotate` configuration.
- **Multi-cloud:** Use Pino's HTTP transport to send logs to a centralised aggregation service.

---

## References

- Winston vs Pino: Choosing a Node.js Logger in 2026 — https://devhelm.io/blog/winston-vs-pino
- Pino Logger for Node.js: Setup, Configuration & Best Practices | Last9 — https://last9.io/blog/npm-pino-logger/
- Winston Logger for Node.js: Setup, Transports & Configuration | Last9 — https://last9.io/blog/winston-logging-in-nodejs/
- Node.js Logging: Fundamentals & Best Practices | SigNoz — https://signoz.io/guides/nodejs-logging/
- How to Implement Custom Logging Framework in Node.js (OneUptime) — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-30-nodejs-custom-logging-framework/README.md
- Logging (uark-backend-class course materials) — https://raw.githubusercontent.com/uark-backend-class/course-materials/refs/heads/master/lectures/19-logging/notes.md
- Structured Logging (glennguilloux/llm-knowledge-base) — https://github.com/glennguilloux/llm-knowledge-base
- api-security-checklist — Logging chapter — https://github.com/batuhan-satilmis/api-security-checklist
- pino-http HTTP and Frameworks Reference (oakoss/agent-skills) — https://github.com/oakoss/agent-skills
- Winston daily-rotate-file (npm) — https://www.npmjs.com/package/winston-daily-rotate-file
- Pino — https://getpino.io/
- Node.js Console Module Documentation — https://nodejs.org/api/console.html
- Bunyan Logging Library — https://github.com/trentm/node-bunyan