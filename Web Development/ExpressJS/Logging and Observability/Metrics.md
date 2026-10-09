# Metrics — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Metrics are numeric measurements captured at regular intervals that quantify the behaviour, performance, and health of a system over time. Unlike logs (discrete events) or traces (request paths), metrics are aggregated time-series data optimised for dashboards, alerting, and capacity planning.

**Technical Definition:** In Node.js, metrics are collected using the Prometheus client library (`prom-client`), which provides four metric types: **Counters** (cumulative values that only increase), **Gauges** (point-in-time values that can go up or down), **Histograms** (observations bucketed into configurable ranges, with sum and count), and **Summaries** (sliding-window quantiles). Metrics are exposed on a `/metrics` HTTP endpoint in Prometheus exposition format and scraped by a Prometheus server at regular intervals. The Prometheus server stores the time series, evaluates alerting rules, and serves queries to Grafana for visualisation. `prom-client` also provides `collectDefaultMetrics()`, which automatically instruments Node.js runtime metrics including event loop lag, garbage collection, heap memory, active handles, and CPU usage. 

**Beginner-Friendly Explanation:** Metrics are numbers that tell you how your application is doing. How many requests per second? How long do they take? How many are failing? How much memory is the app using? These numbers are collected automatically and displayed on dashboards so you can see trends, spot problems before they become outages, and get alerted when something crosses a threshold. 

### Key Characteristics

- **Four metric types:** Counters (monotonic), Gauges (up/down), Histograms (distributions), Summaries (quantiles). 
- **Label-based dimensionality:** Metrics can be sliced by labels (`method`, `route`, `status_code`) for detailed breakdowns. 
- **Pull-based collection:** Prometheus scrapes the `/metrics` endpoint; the application does not push. 
- **Automatic runtime instrumentation:** `collectDefaultMetrics()` captures event loop lag, GC, heap, and CPU without custom code. 
- **RED vs. USE frameworks:** RED (Rate, Errors, Duration) for services; USE (Utilization, Saturation, Errors) for resources. 
- **Custom business metrics:** Counters and gauges for domain-specific events (orders placed, active sessions, conversion rates). 
- **Histogram buckets matter:** Latency distributions require carefully chosen bucket boundaries (e.g., 5ms, 10ms, 25ms, 50ms, 100ms, 250ms, 500ms, 1s, 2.5s, 5s, 10s). 

### Prerequisites

- **Node.js runtime** (v18 or higher).
- **prom-client installed:** `npm install prom-client`.
- **Express app** (or Fastify/Koa) for exposing the `/metrics` endpoint.
- **Prometheus server** for scraping and storage (or Grafana Cloud, which includes Prometheus).
- **Grafana** (optional) for dashboards.
- **Basic understanding of HTTP and time-series data.**

### Related Programming Areas

- **Logging:** Complementary observability pillar (discrete events vs. aggregated numbers).
- **Distributed tracing:** The third observability pillar (request paths across services).
- **Alerting:** Prometheus alerting rules trigger on metric thresholds (error rate > 5%, p99 latency > 1s).
- **SLOs and error budgets:** Metrics are the raw material for Service Level Objectives.
- **Capacity planning:** Gauges (memory, CPU, connections) inform scaling decisions.
- **RED and USE frameworks:** Methodologies that prescribe which metrics to collect.

### Core Concepts

1. **Request Count** — the total number of requests received.
2. **Request Latency** — the distribution of response times.
3. **Error Rate** — the proportion of requests that fail.
4. **Throughput** — requests per second.
5. **Database Latency** — query execution time.
6. **Node.js Runtime Metrics** — event loop lag, GC, heap memory.
7. **RED vs. USE Frameworks** — service-oriented vs. resource-oriented monitoring.
8. **Custom Business Metrics** — active sessions, conversion rates, orders placed.

---

## Core Concept 1: Request Count

### Definitions

**Core Definition:** Request count is a cumulative counter that increments each time the application receives an HTTP request, optionally labelled by method, route, and status code.

**Technical Definition:** A Prometheus Counter is a monotonically increasing metric. `http_requests_total` is the conventional name. The `prom-client` Counter exposes an `.inc()` method and supports `labelNames` for dimensional breakdown. The Prometheus server computes the per-second rate using `rate(http_requests_total[5m])`. Counters should never decrease; if a process restarts, the counter resets to zero, and Prometheus handles this via counter reset detection. 

**Beginner-Friendly Explanation:** Request count is simply "how many requests have we received?" It goes up by one every time someone calls your API. By adding labels for method and status code, you can see not just how many requests, but how many were GETs, how many were POSTs, and how many returned errors. 

### Purposes

- To measure total traffic volume and growth trends.
- To break down traffic by HTTP method, route, and status code.
- To calculate request rate (requests per second) over a time window.
- To provide the denominator for error rate calculations.

### Syntax Rules and Structure

```js
const client = require('prom-client');
const register = new client.Registry();

const httpRequestsTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
  registers: [register]
});

// Increment in middleware after response is sent
app.use((req, res, next) => {
  res.on('finish', () => {
    httpRequestsTotal.inc({
      method: req.method,
      route: req.route?.path || req.path,
      status_code: res.statusCode
    });
  });
  next();
});
```

| Component | Breakdown |
|-----------|-----------|
| `name` | Metric name (must be unique). |
| `help` | Human-readable description. |
| `labelNames` | Dimensions for filtering. |
| `registers` | Registry the metric belongs to. |
| `.inc(labels)` | Increment by 1 (or specified value). |

**Rules:**
- Counters only increase — never decrement. 
- Use `res.on('finish')` to record after the response status code is known. 
- Use the route pattern (`req.route?.path`) rather than the raw URL to avoid cardinality explosion. 
- Never use high-cardinality labels (user ID, request ID) — they create millions of time series. 
- The `rate()` function in Prometheus handles counter resets automatically. 

### Annotated Code Example

```js
// request-count.js
const express = require('express');
const client = require('prom-client');

const app = express();
const register = new client.Registry();

const httpRequestsTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
  registers: [register]
});

// Middleware: increment counter after response finishes
app.use((req, res, next) => {
  res.on('finish', () => {
    const route = req.route?.path || req.path;
    httpRequestsTotal.inc({
      method: req.method,
      route,
      status_code: res.statusCode
    });
  });
  next();
});

app.get('/api/users', (req, res) => res.json([]));
app.post('/api/users', (req, res) => res.status(201).json({}));
app.get('/api/error', (req, res) => res.status(500).json({ error: 'fail' }));

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (`GET /metrics` after 3 requests):**
```
# HELP http_requests_total Total HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="GET",route="/api/users",status_code="200"} 1
http_requests_total{method="POST",route="/api/users",status_code="201"} 1
http_requests_total{method="GET",route="/api/error",status_code="500"} 1
```

**Why this output:** Each request increments the counter with its method, route, and status code. The `/metrics` endpoint exposes the cumulative counts. In Prometheus, `rate(http_requests_total[5m])` computes the per-second request rate over the last 5 minutes. 

### Real-World Cases

- **API gateways:** Tracking total requests to detect traffic spikes or drops.
- **SaaS platforms:** Measuring growth in API usage per tenant.
- **E-commerce:** Counting checkout requests vs. successful orders.

---

## Core Concept 2: Request Latency

### Definitions

**Core Definition:** Request latency is the distribution of time taken to handle HTTP requests, measured as a histogram or summary and queried as percentiles (p50, p95, p99).

**Technical Definition:** A Histogram in Prometheus counts observations into configurable buckets and tracks the sum and count of all observations. The conventional metric name is `http_request_duration_seconds` (a histogram). Prometheus computes percentiles using the `histogram_quantile()` function on the bucket time series. Latency distributions are preferred over averages because the mean hides the slow tail that users actually experience. Bucket boundaries should be chosen to cover the expected range of latencies (e.g., 5ms, 10ms, 25ms, 50ms, 100ms, 250ms, 500ms, 1s, 2.5s, 5s, 10s). 

**Beginner-Friendly Explanation:** Request latency tells you how long requests take. Instead of just "the average is 200ms," a histogram tells you "50% take under 100ms, 95% take under 500ms, and 99% take under 2 seconds." The p99 is what your slowest users experience — the average would hide those users entirely. 

### Purposes

- To measure the distribution of response times, not just the average.
- To compute percentiles (p50, p95, p99) for SLO monitoring.
- To identify slow endpoints and performance regressions.
- To correlate latency spikes with deployments or traffic changes.

### Syntax Rules and Structure

```js
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10],
  registers: [register]
});

app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer();
  res.on('finish', () => {
    end({
      method: req.method,
      route: req.route?.path || req.path,
      status_code: res.statusCode
    });
  });
  next();
});
```

| Component | Breakdown |
|-----------|-----------|
| `buckets` | Upper bounds for each bucket (in seconds). |
| `.startTimer()` | Returns a function that observes duration. |
| `histogram_quantile(0.95, rate(...))` | PromQL for p95 latency. |

**Rules:**
- Always use **seconds** as the unit — Prometheus convention. 
- Choose buckets that cover your expected range; too few buckets give inaccurate percentiles. 
- Use `histogram_quantile()` in PromQL to compute percentiles. 
- Track latency by route and method to identify slow endpoints. 
- Histograms are preferred over summaries for aggregatable percentiles. 

### Annotated Code Example

```js
// request-latency.js
const express = require('express');
const client = require('prom-client');

const app = express();
const register = new client.Registry();

const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10],
  registers: [register]
});

app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer();
  res.on('finish', () => {
    end({
      method: req.method,
      route: req.route?.path || req.path,
      status_code: res.statusCode
    });
  });
  next();
});

app.get('/api/fast', (req, res) => res.json({ ok: true }));
app.get('/api/slow', (req, res) => {
  setTimeout(() => res.json({ ok: true }), 1500);
});

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (`GET /metrics` after requests):**
```
# HELP http_request_duration_seconds Duration of HTTP requests
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{method="GET",route="/api/fast",status_code="200",le="0.005"} 1
http_request_duration_seconds_bucket{method="GET",route="/api/fast",status_code="200",le="0.01"} 1
http_request_duration_seconds_bucket{method="GET",route="/api/fast",status_code="200",le="+Inf"} 1
http_request_duration_seconds_sum{method="GET",route="/api/fast",status_code="200"} 0.002
http_request_duration_seconds_count{method="GET",route="/api/fast",status_code="200"} 1
http_request_duration_seconds_bucket{method="GET",route="/api/slow",status_code="200",le="1.0"} 0
http_request_duration_seconds_bucket{method="GET",route="/api/slow",status_code="200",le="2.5"} 1
http_request_duration_seconds_bucket{method="GET",route="/api/slow",status_code="200",le="+Inf"} 1
http_request_duration_seconds_sum{method="GET",route="/api/slow",status_code="200"} 1.502
http_request_duration_seconds_count{method="GET",route="/api/slow",status_code="200"} 1
```

**PromQL for p95 latency:**
```
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

**Why this output:** Each bucket counts observations less than or equal to the `le` boundary. The fast request falls into the `0.005` bucket; the slow request falls into the `2.5` bucket. The `sum` and `count` allow Prometheus to compute averages and percentiles. The `histogram_quantile()` function interpolates within buckets to estimate percentiles. 

### Real-World Cases

- **SLO monitoring:** "95% of requests must complete within 500ms."
- **Performance regression detection:** Alert when p99 latency exceeds 2 seconds.
- **Capacity planning:** Track latency trends as traffic grows.

---

## Core Concept 3: Error Rate

### Definitions

**Core Definition:** Error rate is the proportion of requests that fail, calculated as `rate(http_requests_total{status_code=~"5.."}[5m]) / rate(http_requests_total[5m])`.

**Technical Definition:** Error rate is derived from the `http_requests_total` counter by filtering for `status_code` values in the 5xx range (server errors) or 4xx range (client errors). PromQL computes the ratio of error requests to total requests over a rolling window. Alerting rules typically fire when the error rate exceeds a threshold (e.g., 5% for 5 minutes). 

**Beginner-Friendly Explanation:** Error rate tells you what percentage of your requests are failing. If 5% of requests return a 500 error, your error rate is 5%. This is one of the most important signals for user-facing services — high error rate means users are experiencing failures. 

### Purposes

- To measure the proportion of requests that fail.
- To alert when error rates exceed acceptable thresholds.
- To distinguish between 4xx (client errors) and 5xx (server errors).
- To provide the "Errors" component of the RED method.

### Syntax Rules and Structure

**PromQL expressions:**
```promql
# 5xx error rate (server errors)
rate(http_requests_total{status_code=~"5.."}[5m])
  / rate(http_requests_total[5m])

# 4xx error rate (client errors)
rate(http_requests_total{status_code=~"4.."}[5m])
  / rate(http_requests_total[5m])

# Error rate by route
sum by (route) (rate(http_requests_total{status_code=~"5.."}[5m]))
  / sum by (route) (rate(http_requests_total[5m]))
```

**Alerting rule example:**
```yaml
groups:
  - name: error-rate
    rules:
      - alert: HighErrorRate
        expr: |
          rate(http_requests_total{status_code=~"5.."}[5m])
          / rate(http_requests_total[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Error rate exceeds 5% for 5 minutes"
```

**Rules:**
- Filter by `status_code` using regex (`=~"5.."` for 5xx, `=~"4.."` for 4xx). 
- Use `rate()` over a window that balances responsiveness and noise reduction (5m is common). 
- Alert on error rate, not absolute error count — a 100 errors/minute is fine if you serve 1 million requests. 
- Distinguish between 4xx (expected, client-side) and 5xx (unexpected, server-side). 

### Annotated Code Example

```js
// error-rate.js
// The error rate is derived from the http_requests_total counter
// No additional instrumentation is needed — the status_code label provides the data

// In Prometheus, the error rate is computed as:
// rate(http_requests_total{status_code=~"5.."}[5m]) / rate(http_requests_total[5m])

// To test, create endpoints that return errors:
app.get('/api/ok', (req, res) => res.json({ ok: true }));
app.get('/api/fail', (req, res) => res.status(500).json({ error: 'fail' }));
app.get('/api/not-found', (req, res) => res.status(404).json({ error: 'nf' }));
```

**Expected Output (`GET /metrics` after 1 success, 1 failure, 1 not-found):**
```
http_requests_total{method="GET",route="/api/ok",status_code="200"} 1
http_requests_total{method="GET",route="/api/fail",status_code="500"} 1
http_requests_total{method="GET",route="/api/not-found",status_code="404"} 1
```

**PromQL result:**
```
# 5xx error rate
rate(http_requests_total{status_code=~"5.."}[5m])
  / rate(http_requests_total[5m]) = 0.333  (33.3%)

# 4xx error rate
rate(http_requests_total{status_code=~"4.."}[5m])
  / rate(http_requests_total[5m]) = 0.333  (33.3%)
```

**Why this output:** The `status_code` label on `http_requests_total` provides all the data needed to compute error rates. PromQL filters the counter by status code range and divides by the total. No additional metrics are required. 

### Real-World Cases

- **SLO alerting:** Page when 5xx error rate exceeds 1% for 5 minutes.
- **Deployment validation:** Monitor error rate during canary releases.
- **Incident response:** Correlate error rate spikes with deployments or infrastructure changes.

---

## Core Concept 4: Throughput

### Definitions

**Core Definition:** Throughput is the rate at which requests are processed, typically measured as requests per second (RPS) using `rate(http_requests_total[1m])`.

**Technical Definition:** Throughput is the first component of the RED method: Rate (requests per second). It is computed from the `http_requests_total` counter using PromQL's `rate()` function over a time window. Throughput can be measured in total, by route, by method, or by status code. It is the denominator for error rate and the baseline for capacity planning. 

**Beginner-Friendly Explanation:** Throughput is "how many requests per second is my server handling?" If your throughput is 1,000 RPS and suddenly drops to 100 RPS, something is wrong — either traffic dropped or your server stopped accepting requests. 

### Purposes

- To measure the volume of traffic the system is handling.
- To detect traffic drops (outages) or spikes (attacks, viral events).
- To provide the "Rate" component of the RED method.
- To inform capacity planning and auto-scaling decisions.

### Syntax Rules and Structure

**PromQL expressions:**
```promql
# Total requests per second
rate(http_requests_total[1m])

# Requests per second by route
sum by (route) (rate(http_requests_total[1m]))

# Requests per second by method
sum by (method) (rate(http_requests_total[1m]))

# Peak throughput (max over 1 hour)
max_over_time(rate(http_requests_total[1m])[1h:1m])
```

| Function | Purpose |
|----------|---------|
| `rate(metric[1m])` | Per-second rate over 1 minute. |
| `sum by (label)` | Aggregate by label. |
| `max_over_time()` | Peak value over a window. |

**Rules:**
- Use `rate()` over a window that smooths noise but remains responsive (1m or 5m). 
- Throughput is derived from the counter — no separate metric is needed. 
- Monitor throughput alongside latency and error rate (the three RED signals). 
- Use throughput for auto-scaling triggers (e.g., scale when RPS > 1000 per instance). 

### Annotated Code Example

```js
// throughput.js
// Throughput is derived from http_requests_total — no additional instrumentation

// In Prometheus:
// rate(http_requests_total[1m]) gives requests per second

// To test, send a burst of requests:
// curl -X GET http://localhost:3000/api/users (repeat 100 times)
```

**Expected Output (PromQL):**
```
# Total throughput
rate(http_requests_total[1m]) = 1.67 requests/second

# Throughput by route
sum by (route) (rate(http_requests_total[1m]))
  {route="/api/users"} 1.5
  {route="/api/orders"} 0.17
```

**Why this output:** The `rate()` function computes the per-second average over the last minute. With 100 requests in 60 seconds, the rate is approximately 1.67 RPS. Breaking down by route shows which endpoints are handling the most traffic. 

### Real-World Cases

- **Auto-scaling:** Scale out when RPS per instance exceeds a threshold.
- **Traffic analysis:** Identify the most popular endpoints.
- **Anomaly detection:** Alert when throughput drops below expected levels.

---

## Core Concept 5: Database Latency

### Definitions

**Core Definition:** Database latency is the time taken to execute database queries, measured as a histogram and tracked by query type, table, or operation.

**Technical Definition:** A custom Histogram named `db_query_duration_seconds` measures query execution time. Labels include `operation` (SELECT, INSERT, UPDATE, DELETE), `table`, and `status` (success/error). The histogram buckets should cover the expected range of query latencies (e.g., 1ms to 5s). Prometheus computes percentiles using `histogram_quantile()`. Database latency is critical because slow queries cascade into slow API responses. 

**Beginner-Friendly Explanation:** Database latency tells you how long your database queries take. If your API is slow, the database is often the culprit. Tracking query latency by table and operation helps you identify which queries need optimisation. 

### Purposes

- To identify slow database queries and tables.
- To correlate database latency with API latency.
- To detect connection pool exhaustion (queries queuing).
- To monitor the impact of indexing or schema changes.

### Syntax Rules and Structure

```js
const dbQueryDuration = new client.Histogram({
  name: 'db_query_duration_seconds',
  help: 'Duration of database queries',
  labelNames: ['operation', 'table', 'status'],
  buckets: [0.001, 0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5],
  registers: [register]
});

async function queryWithMetrics(operation, table, fn) {
  const end = dbQueryDuration.startTimer();
  try {
    const result = await fn();
    end({ operation, table, status: 'success' });
    return result;
  } catch (err) {
    end({ operation, table, status: 'error' });
    throw err;
  }
}

// Usage
const users = await queryWithMetrics('SELECT', 'users', () =>
  db.query('SELECT * FROM users WHERE id = $1', [id])
);
```

| Component | Breakdown |
|-----------|-----------|
| `operation` | SQL operation (SELECT, INSERT, UPDATE, DELETE). |
| `table` | Target table name. |
| `status` | `'success'` or `'error'`. |

**Rules:**
- Wrap database queries in a helper that records duration and status. 
- Label by `operation` and `table` to identify slow queries. 
- Track `status` to distinguish successful queries from failures. 
- Use buckets that cover 1ms to 5s (most queries fall in this range). 
- Monitor connection pool metrics separately (active, idle, waiting). 

### Annotated Code Example

```js
// database-latency.js
const express = require('express');
const client = require('prom-client');
const { Pool } = require('pg');

const app = express();
const register = new client.Registry();
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

const dbQueryDuration = new client.Histogram({
  name: 'db_query_duration_seconds',
  help: 'Database query duration',
  labelNames: ['operation', 'table', 'status'],
  buckets: [0.001, 0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5],
  registers: [register]
});

async function trackedQuery(operation, table, sql, params) {
  const end = dbQueryDuration.startTimer();
  try {
    const result = await pool.query(sql, params);
    end({ operation, table, status: 'success' });
    return result;
  } catch (err) {
    end({ operation, table, status: 'error' });
    throw err;
  }
}

app.get('/api/users/:id', async (req, res) => {
  const { rows } = await trackedQuery(
    'SELECT', 'users',
    'SELECT * FROM users WHERE id = $1',
    [req.params.id]
  );
  res.json(rows[0]);
});

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (`GET /metrics` after queries):**
```
# HELP db_query_duration_seconds Database query duration
# TYPE db_query_duration_seconds histogram
db_query_duration_seconds_bucket{operation="SELECT",table="users",status="success",le="0.001"} 5
db_query_duration_seconds_bucket{operation="SELECT",table="users",status="success",le="0.005"} 12
db_query_duration_seconds_bucket{operation="SELECT",table="users",status="success",le="0.01"} 18
db_query_duration_seconds_bucket{operation="SELECT",table="users",status="success",le="+Inf"} 20
db_query_duration_seconds_sum{operation="SELECT",table="users",status="success"} 0.087
db_query_duration_seconds_count{operation="SELECT",table="users",status="success"} 20
```

**PromQL for p95 database latency:**
```
histogram_quantile(0.95, rate(db_query_duration_seconds_bucket[5m]))
```

**Why this output:** The histogram records each query's duration into the appropriate bucket. The `sum` and `count` allow Prometheus to compute averages and percentiles. The labels (`operation`, `table`, `status`) allow filtering to specific queries. 

### Real-World Cases

- **Query optimisation:** Identify the slowest tables and operations.
- **Connection pool monitoring:** Track queuing when queries wait for a connection.
- **Migration validation:** Compare query latency before and after index changes.

---

## Core Concept 6: Node.js Runtime Metrics

### Definitions

**Core Definition:** Node.js runtime metrics are automatically collected measurements of the Node.js process itself — event loop lag, garbage collection, heap memory, active handles, and CPU usage — exposed via `collectDefaultMetrics()`.

**Technical Definition:** `prom-client`'s `collectDefaultMetrics()` registers a set of default metrics recommended by Prometheus, including Node.js-specific ones. The key runtime metrics are: `nodejs_eventloop_lag_seconds` (event loop lag), `nodejs_gc_duration_seconds` (GC pause duration histogram), `nodejs_heap_size_total_bytes` and `nodejs_heap_size_used_bytes` (heap memory), `nodejs_active_handles_total` (active handles), and `nodejs_process_cpu_seconds_total` (CPU usage). These metrics are prefixed with `nodejs_` by default. 

**Beginner-Friendly Explanation:** Node.js has unique performance characteristics that generic monitoring tools miss. The event loop can become blocked, causing all requests to slow down. Garbage collection pauses can freeze the process. Memory leaks can cause crashes. These metrics give you visibility into exactly these Node.js-specific problems. 

### Purposes

- To detect event loop blocking (the #1 cause of Node.js performance problems).
- To monitor garbage collection frequency and pause duration.
- To track heap memory usage and detect leaks.
- To monitor active handles (file descriptors, sockets, timers).
- To provide the "Utilization" and "Saturation" components of the USE method.

### Syntax Rules and Structure

```js
const client = require('prom-client');
const register = new client.Registry();

// Collect all default metrics including Node.js runtime metrics
client.collectDefaultMetrics({
  register,
  prefix: 'nodejs_',           // Prefix for all metric names
  gcDurationBuckets: [0.001, 0.01, 0.1, 1, 2, 5],  // GC buckets
  eventLoopMonitoringPrecision: 10  // Sampling rate in ms
});
```

| Metric | Description | Alerting Threshold |
|--------|-------------|-------------------|
| `nodejs_eventloop_lag_seconds` | Event loop lag | p99 > 100ms |
| `nodejs_gc_duration_seconds` | GC pause duration | p99 > 100ms |
| `nodejs_heap_size_used_bytes` | Heap memory used | > 80% of total |
| `nodejs_active_handles_total` | Active handles | Sudden growth |
| `nodejs_process_cpu_seconds_total` | CPU usage | > 80% sustained |

**Configuration options:**
| Option | Default | Purpose |
|--------|---------|---------|
| `prefix` | none | Prefix for metric names. |
| `gcDurationBuckets` | `[0.001, 0.01, 0.1, 1, 2, 5]` | GC histogram buckets. |
| `eventLoopMonitoringPrecision` | `10` (ms) | Event loop sampling rate. |
| `eventLoopUtilizationTimeout` | `100` (ms) | ELU calculation interval. |

**Rules:**
- Call `collectDefaultMetrics()` once at application startup. 
- Event loop lag is the most critical Node.js metric — alert on p99 > 100ms. 
- Heap memory should be monitored for sustained growth (leak detection). 
- GC duration p99 > 100ms indicates GC pressure. 
- Some metrics (file descriptors, memory) are only available on Linux. 

### Annotated Code Example

```js
// nodejs-runtime-metrics.js
const express = require('express');
const client = require('prom-client');

const app = express();
const register = new client.Registry();

// Collect default metrics including Node.js runtime metrics
client.collectDefaultMetrics({
  register,
  prefix: 'nodejs_',
  gcDurationBuckets: [0.001, 0.01, 0.1, 1, 2, 5],
  eventLoopMonitoringPrecision: 10
});

// Simulate event loop blocking (bad practice — for demonstration only)
app.get('/api/block', (req, res) => {
  const start = Date.now();
  while (Date.now() - start < 200) {}  // Block for 200ms
  res.json({ blocked: true });
});

// Simulate memory allocation
app.get('/api/allocate', (req, res) => {
  const arr = new Array(1e6).fill('x'.repeat(100));
  res.json({ allocated: arr.length });
});

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (`GET /metrics` — Node.js runtime metrics):**
```
# HELP nodejs_eventloop_lag_seconds Lag of event loop in seconds
# TYPE nodejs_eventloop_lag_seconds gauge
nodejs_eventloop_lag_seconds 0.001
nodejs_eventloop_lag_p99_seconds 0.205
nodejs_eventloop_lag_max_seconds 0.205

# HELP nodejs_gc_duration_seconds Garbage collection duration
# TYPE nodejs_gc_duration_seconds histogram
nodejs_gc_duration_seconds_bucket{kind="minor",le="0.001"} 45
nodejs_gc_duration_seconds_bucket{kind="minor",le="0.01"} 78
nodejs_gc_duration_seconds_bucket{kind="major",le="0.1"} 2
nodejs_gc_duration_seconds_sum{kind="minor"} 0.023
nodejs_gc_duration_seconds_count{kind="minor"} 78

# HELP nodejs_heap_size_used_bytes Process heap size used
# TYPE nodejs_heap_size_used_bytes gauge
nodejs_heap_size_used_bytes 45678912

# HELP nodejs_heap_size_total_bytes Process heap size total
# TYPE nodejs_heap_size_total_bytes gauge
nodejs_heap_size_total_bytes 67108864

# HELP nodejs_active_handles_total Number of active handles
# TYPE nodejs_active_handles_total gauge
nodejs_active_handles_total 12

# HELP nodejs_process_cpu_seconds_total Total CPU time
# TYPE nodejs_process_cpu_seconds_total counter
nodejs_process_cpu_seconds_total 1.23
```

**Why this output:** The `/api/block` endpoint blocks the event loop for 200ms, which is reflected in `nodejs_eventloop_lag_p99_seconds` (0.205). The `/api/allocate` endpoint allocates memory, reflected in `nodejs_heap_size_used_bytes`. GC metrics show minor and major collection counts and durations. These metrics provide visibility into Node.js-specific performance issues. 

### Real-World Cases

- **Event loop blocking detection:** Alert when event loop lag p99 exceeds 100ms.
- **Memory leak detection:** Alert when heap memory grows continuously without dropping.
- **GC pressure:** Tune V8 flags when GC pauses exceed acceptable thresholds.
- **Connection leak detection:** Monitor `nodejs_active_handles_total` for unexpected growth.

---

## Core Concept 7: RED vs. USE Frameworks

### Definitions

**Core Definition:** RED (Rate, Errors, Duration) is a service-oriented monitoring framework that measures what users experience; USE (Utilization, Saturation, Errors) is a resource-oriented framework that measures the health of the underlying infrastructure. 

**Technical Definition:** The RED method, created by Tom Wilkie in 2015, instruments every service with three metrics: Rate (requests per second), Errors (failed requests per second), and Duration (the distribution of request latency). The USE method, created by Brendan Gregg in 2012, checks three properties for every resource: Utilization (the percentage of time the resource was busy), Saturation (the amount of queued work it cannot service), and Errors (the count of error events). RED answers "Are users having a bad time?"; USE answers "Is a resource the bottleneck?" 

**Beginner-Friendly Explanation:** RED is about your users. USE is about your hardware. If your API is slow (RED: high Duration), the cause might be that your CPU is maxed out (USE: high Utilization) or your disk queue is full (USE: high Saturation). You need both frameworks: RED to know something is wrong, USE to know why. 

### Purposes

- To provide a systematic checklist for monitoring every service (RED) and every resource (USE).
- To enable faster diagnosis by moving from user-facing symptoms (RED) to resource-level causes (USE).
- To standardise dashboards and alerts across teams.
- To ensure no critical metric is forgotten.

### Syntax Rules and Structure

**RED vs. USE comparison:**

| Dimension | RED | USE |
|-----------|-----|-----|
| **Metrics** | Rate, Errors, Duration | Utilization, Saturation, Errors |
| **Origin** | Tom Wilkie, 2015 | Brendan Gregg, 2012 |
| **Unit of analysis** | Each service | Each resource (CPU, memory, disk, network) |
| **Question** | Are users having a bad time? | Is a resource the bottleneck? |
| **Best for** | Microservices, APIs | Infrastructure, capacity problems |
| **Blind spots** | Saturation and capacity | User experience |

**Red metrics implementation:**
```js
// Rate
httpRequestsTotal.inc({ method, route, status_code });

// Errors — derived from status_code label
// rate(http_requests_total{status_code=~"5.."}[5m])

// Duration
httpRequestDuration.observe({ method, route, status_code }, seconds);
```

**USE metrics implementation:**
```js
// CPU Utilization — from collectDefaultMetrics
// nodejs_process_cpu_seconds_total

// Memory Utilization — from collectDefaultMetrics
// nodejs_heap_size_used_bytes / nodejs_heap_size_total_bytes

// Saturation — event loop lag
// nodejs_eventloop_lag_seconds

// Errors — from application counters
// db_query_errors_total
```

**Rules:**
- Use RED for every service that handles requests. 
- Use USE for every resource (CPU, memory, disk, network, connection pools). 
- RED and USE are complementary — neither replaces the other. 
- Start with RED (user-facing) and drill down to USE (resource-level) when diagnosing. 
- The four golden signals (Latency, Traffic, Errors, Saturation) sit above both as the umbrella set. 

### Annotated Code Example

```js
// red-use-metrics.js
const express = require('express');
const client = require('prom-client');

const app = express();
const register = new client.Registry();

// --- RED metrics ---
const httpRequestsTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
  registers: [register]
});

const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5],
  registers: [register]
});

// --- USE metrics (Node.js runtime) ---
client.collectDefaultMetrics({
  register,
  prefix: 'nodejs_',
  eventLoopMonitoringPrecision: 10
});

// RED middleware
app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer();
  res.on('finish', () => {
    const labels = {
      method: req.method,
      route: req.route?.path || req.path,
      status_code: res.statusCode
    };
    httpRequestsTotal.inc(labels);
    end(labels);
  });
  next();
});

app.get('/api/data', (req, res) => res.json({ data: 'ok' }));

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (RED metrics):**
```
# RED: Rate
rate(http_requests_total[1m]) = 10 req/s

# RED: Errors
rate(http_requests_total{status_code=~"5.."}[5m]) = 0.1 req/s

# RED: Duration (p95)
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) = 0.25s
```

**Expected Output (USE metrics):**
```
# USE: Utilization (CPU)
nodejs_process_cpu_seconds_total = 0.45

# USE: Utilization (Memory)
nodejs_heap_size_used_bytes / nodejs_heap_size_total_bytes = 0.68

# USE: Saturation (Event Loop Lag p99)
nodejs_eventloop_lag_p99_seconds = 0.012

# USE: Errors (GC)
nodejs_gc_duration_seconds_count{kind="major"} = 2
```

**Why this output:** The RED metrics tell you that the service is handling 10 requests/second with 0.1 errors/second and 250ms p95 latency. The USE metrics tell you that CPU is at 45%, memory at 68%, event loop lag is low (12ms), and there have been 2 major GC cycles. Together, they provide a complete picture: RED shows the user experience; USE shows the resource health. 

### Real-World Cases

- **Microservices debugging:** RED shows a service is slow; USE shows the database connection pool is saturated.
- **Kubernetes monitoring:** RED for pod-level service metrics; USE for node-level resource metrics.
- **Capacity planning:** RED shows growing traffic; USE shows when resources will be exhausted.
- **Incident response:** Start with RED dashboards (what's broken for users?) and drill down to USE dashboards (which resource is the bottleneck?).

---

## Core Concept 8: Custom Business Metrics

### Definitions

**Core Definition:** Custom business metrics are counters and gauges that measure domain-specific events — orders placed, user registrations, active sessions, checkout conversion rates — rather than technical infrastructure metrics.

**Technical Definition:** Custom metrics use the same Prometheus metric types (Counter, Gauge, Histogram, Summary) but are named and labelled to reflect business concepts. Examples: `orders_total` (counter, labelled by `type`, `payment_method`, `region`), `active_users` (gauge), `checkout_conversion_rate` (gauge, computed from two counters). Best practices include: naming with the unit (e.g., `_total` for counters, `_seconds` for durations), using labels for dimensions, and avoiding high-cardinality labels (user IDs). 

**Beginner-Friendly Explanation:** Technical metrics tell you if your server is healthy. Business metrics tell you if your business is healthy. How many orders were placed today? How many users signed up? What's the conversion rate from cart to checkout? These metrics are what executives and product managers care about. 

### Purposes

- To track business-critical events (orders, signups, conversions).
- To provide real-time visibility into product usage and revenue.
- To alert on business anomalies (sudden drop in orders, spike in cancellations).
- To inform product decisions with data.

### Syntax Rules and Structure

```js
// Counter — cumulative business events
const ordersTotal = new client.Counter({
  name: 'orders_total',
  help: 'Total orders placed',
  labelNames: ['type', 'payment_method', 'region'],
  registers: [register]
});

// Gauge — current business state
const activeSessions = new client.Gauge({
  name: 'active_sessions',
  help: 'Currently active user sessions',
  registers: [register]
});

// Usage
app.post('/orders', async (req, res) => {
  const order = await createOrder(req.body);
  ordersTotal.inc({
    type: order.type,
    payment_method: order.paymentMethod,
    region: order.region
  });
  res.json(order);
});

// Session tracking
app.use((req, res, next) => {
  activeSessions.inc();
  res.on('finish', () => activeSessions.dec());
  next();
});
```

| Metric | Type | Example Value | Business Meaning |
|--------|------|--------------|-----------------|
| `orders_total` | Counter | 1,234 | Total orders placed. |
| `active_sessions` | Gauge | 456 | Users currently online. |
| `user_registrations_total` | Counter | 789 | Total signups. |
| `checkout_conversion_rate` | Gauge | 0.72 | % of carts that convert. |

**Rules:**
- Use `_total` suffix for counters (Prometheus convention). 
- Use labels for dimensions (`payment_method`, `region`) but avoid high-cardinality labels (`user_id`). 
- Compute rates (`checkout_conversion_rate`) from counters in PromQL, not as separate metrics. 
- Business metrics should be defined in collaboration with product and business teams. 
- Alert on business metric anomalies (e.g., orders dropping 50% from baseline). 

### Annotated Code Example

```js
// business-metrics.js
const express = require('express');
const client = require('prom-client');

const app = express();
const register = new client.Registry();

// --- Business metrics ---
const ordersTotal = new client.Counter({
  name: 'orders_total',
  help: 'Total orders placed',
  labelNames: ['type', 'payment_method', 'region'],
  registers: [register]
});

const userRegistrationsTotal = new client.Counter({
  name: 'user_registrations_total',
  help: 'Total user registrations',
  labelNames: ['source'],
  registers: [register]
});

const activeSessions = new client.Gauge({
  name: 'active_sessions',
  help: 'Currently active user sessions',
  registers: [register]
});

// --- Routes that increment business metrics ---
app.post('/api/orders', (req, res) => {
  const { type, paymentMethod, region } = req.body;
  ordersTotal.inc({ type, payment_method: paymentMethod, region });
  res.status(201).json({ id: 'ord_123' });
});

app.post('/api/register', (req, res) => {
  const { source } = req.body;
  userRegistrationsTotal.inc({ source: source || 'direct' });
  res.status(201).json({ id: 'usr_456' });
});

// --- Session tracking middleware ---
app.use((req, res, next) => {
  activeSessions.inc();
  res.on('finish', () => activeSessions.dec());
  next();
});

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (`GET /metrics` after business events):**
```
# HELP orders_total Total orders placed
# TYPE orders_total counter
orders_total{type="physical",payment_method="credit_card",region="us"} 45
orders_total{type="digital",payment_method="paypal",region="eu"} 23

# HELP user_registrations_total Total user registrations
# TYPE user_registrations_total counter
user_registrations_total{source="google"} 120
user_registrations_total{source="direct"} 80

# HELP active_sessions Currently active user sessions
# TYPE active_sessions gauge
active_sessions 67
```

**PromQL for checkout conversion rate:**
```
rate(orders_total[1h]) / rate(cart_additions_total[1h])
```

**Why this output:** The `orders_total` counter is incremented with business-relevant labels. The `active_sessions` gauge increases on request start and decreases on request finish, reflecting real-time concurrency. The `user_registrations_total` counter tracks signups by source. These metrics enable business dashboards and alerts. 

### Real-World Cases

- **E-commerce:** Track orders by payment method and region; alert when orders drop 50% from the same hour last week.
- **SaaS platforms:** Monitor active sessions and trial conversions; alert when trial-to-paid conversion drops.
- **Fintech:** Track transactions by type and amount; alert on unusual transaction patterns.
- **Media platforms:** Monitor video plays and completion rates; alert on CDN or encoding failures.

---

## References

- Prometheus Client for Node.js (GitHub) — https://github.com/prometheus/client_js
- express-prom-bundle (GitHub) — https://github.com/jochen-schweizer/express-prom-bundle
- RED Method vs USE Method (ClickHouse) — https://clickhouse.com/resources/engineering/red-use-methods
- How to Add Custom Metrics to Node.js Applications with Prometheus (OneUptime) — https://oneuptime.com/blog/post/2026-01-06-nodejs-custom-metrics-prometheus/view
- prom-client Default Metrics Documentation — https://github.com/prometheus/client_js#default-metrics
- Node.js Monitoring Guide: Performance Metrics, APM & Alerts (2026) — https://apistatuscheck.com
- Pino Logger for Node.js: Setup, Configuration & Best Practices (Last9) — https://last9.io/blog/npm-pino-logger/
- Monitoring Node.js Apps with Prometheus (Better Stack) — https://betterstack.com/community/guides/scaling-nodejs/nodejs-prometheus/
- Monitoring Node.js Applications on OpenShift with Prometheus (Red Hat Developer) — https://developers.redhat.com/blog/2018/12/21/monitoring-node-js-applications-on-openshift-with-prometheus
- Prometheus Histograms and Summaries — https://prometheus.io/docs/practices/histograms/
- Google SRE Book — Monitoring Distributed Systems — https://sre.google/sre-book/monitoring-distributed-systems/
- Brendan Gregg — The USE Method — https://www.brendangregg.com/usemethod.html
- Grafana Dashboard Best Practices — https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/best-practices/