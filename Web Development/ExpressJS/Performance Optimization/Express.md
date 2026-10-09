# Express Performance — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Express performance optimisation is the practice of reducing the latency, memory footprint, and CPU cost of an Express application by minimising middleware overhead, optimising JSON serialisation, enabling compression, caching responses, pooling connections, using high-speed serialisers, and eliminating asynchronous error-handling overhead.

**Technical Definition:** Express is a minimal, unopinionated web framework built on Node.js's `http` module. Its performance is governed by the middleware pipeline (each middleware adds a function call per request), the serialisation cost of JSON responses (the built-in `JSON.stringify` is convenient but not the fastest), network transfer size (mitigated by Gzip/Brotli compression), repeated computation (mitigated by caching), socket reuse for outbound calls (connection pooling), and the overhead of unhandled promise rejections in route handlers. Optimisation targets the hot path — the sequence of operations executed for every request — and prioritises the highest-impact changes first.

**Beginner-Friendly Explanation:** Express is fast by default, but as your application grows, it can slow down. Performance optimisation means making your server handle more requests per second with the same hardware. You do this by removing unnecessary work (middleware you do not need), sending less data over the network (compression), avoiding repeated calculations (caching), reusing connections instead of creating new ones, and using faster tools for common tasks like converting objects to JSON.

### Key Characteristics

- **Middleware is not free:** Every `app.use()` call adds a function invocation to every request that reaches it. 
- **Route order matters:** Middleware and routes are executed in registration order; placing the most common routes first reduces unnecessary matching.
- **Compression is a trade-off:** Gzip/Brotli reduce bandwidth but consume CPU. Brotli compresses better; Gzip is faster. 
- **Caching eliminates work:** In-memory or Redis-backed caches serve repeated responses without re-computation.
- **Connection pooling reuses sockets:** `keep-alive` avoids the TCP and TLS handshake cost for every outbound request. 
- **`fast-json-stringify` is 2–5× faster:** Schema-based serialisation outperforms `JSON.stringify` for large or repetitive payloads. 
- **Async error wrapping has a cost:** Every `try/catch` and every `async` wrapper adds overhead; Express 5 handles async errors natively without wrappers. 

### Prerequisites

- **Node.js runtime** (v18 or higher; Express 5 requires Node.js 18+).
- **Express.js installed:** `npm install express`.
- **For compression:** `npm install compression`.
- **For fast serialisation:** `npm install fast-json-stringify`.
- **For caching:** `npm install node-cache` or `npm install ioredis` (Redis).
- **For connection pooling:** `npm install undici` (built-in in Node.js 18+).
- **Basic understanding of HTTP, middleware, and async/await.**

### Related Programming Areas

- **Middleware:** The primary source of Express overhead.
- **Caching:** Reduces database and computation load.
- **Load balancing:** Distributes traffic across instances; complements per-instance optimisation.
- **Profiling:** Tools like `clinic.js`, `0x`, and Node.js `--prof` identify bottlenecks.
- **Observability:** Metrics and tracing reveal where time is spent.

### Core Concepts

1. **Middleware Overhead** — minimising third-party middleware and optimising route order.
2. **JSON Serialisation** — reducing payload weight and optimising `res.json()`.
3. **Compression** — Gzip/Brotli deflation.
4. **Response Caching** — storing static endpoints or repetitive responses in memory.
5. **Connection Pooling** — reusing active sockets across internal microservices.
6. **High-Speed Serialisation** — `fast-json-stringify` and alternatives.
7. **Asynchronous Error Wrapping** — preventing execution overhead from unhandled promises.

---

## Core Concept 1: Middleware Overhead

### Definitions

**Core Definition:** Middleware overhead is the cumulative CPU and latency cost of executing every middleware function registered on the application for every request, whether or not the middleware performs useful work for that request.

**Technical Definition:** In Express, middleware functions are executed sequentially in the order they are registered. Each middleware invocation adds a function call, a `next()` call, and any synchronous work the middleware performs. Global middleware (registered with `app.use()` and no path) runs for **every** request, including static assets, health checks, and unmatched routes. Route-specific middleware (registered on a specific route or router) runs only when the route matches. Reducing overhead means: removing unused middleware, scoping middleware to the paths that need it, placing common routes before rare routes, and avoiding synchronous blocking work inside middleware. 

**Beginner-Friendly Explanation:** Imagine a queue at the airport. Every passenger (request) must pass through every security checkpoint (middleware) before reaching their gate (route handler). If you add checkpoints that most passengers don't need, everyone waits longer. The fix is to send passengers through only the checkpoints that apply to them — and to put the most-used gates (routes) near the entrance.

### Purposes

- To minimise the number of function calls executed per request.
- To ensure middleware only runs for routes that need it.
- To reduce latency by placing high-traffic routes first.
- To avoid blocking the event loop with synchronous work in middleware.

### Syntax Rules and Structure

**Global vs. scoped middleware:**
```js
// ❌ Global — runs for every request, including static files
app.use(express.json());
app.use(cors());
app.use(helmet());
app.use(logger);

// ✅ Scoped — runs only for API routes
app.use('/api', express.json());
app.use('/api', cors());
app.use('/api', helmet());
app.use('/api', logger);

// ✅ Static files bypass all API middleware
app.use(express.static('public'));
```

**Route order optimisation:**
```js
// ✅ Common routes first
app.get('/health', healthHandler);        // 1M requests/day
app.get('/api/users', listUsers);         // 500K requests/day
app.get('/api/products', listProducts);   // 100K requests/day
app.get('/api/rare-endpoint', rareHandler); // 10 requests/day
```

| Pattern | Impact |
|---------|--------|
| Global `app.use(mw)` | Runs for every request. |
| Path-scoped `app.use('/api', mw)` | Runs only for `/api/*`. |
| Router-scoped `router.use(mw)` | Runs only for the router's paths. |
| Route-specific `app.get(path, mw, handler)` | Runs only for that route. |

**Rules:**
- Mount middleware on the paths that need it, not globally. 
- Place health checks and static file serving **before** heavy API middleware.
- Use `express.Router()` to scope middleware to feature areas.
- Avoid synchronous CPU-heavy work (large loops, crypto) in middleware.
- Remove unused middleware — every function adds overhead.

### Annotated Code Example

```js
// middleware-overhead.js
const express = require('express');
const compression = require('compression');
const cors = require('cors');
const helmet = require('helmet');
const app = express();

// --- Optimised middleware ordering ---

// 1. Static files first — no heavy middleware runs for them
app.use(express.static('public', { maxAge: '1d' }));

// 2. Health check — no JSON parsing, no auth, no logging
app.get('/health', (req, res) => res.json({ status: 'ok' }));

// 3. API middleware — only for /api routes
app.use('/api', helmet());
app.use('/api', cors());
app.use('/api', compression());
app.use('/api', express.json({ limit: '1mb' }));

// 4. API routes in order of frequency
app.get('/api/users', (req, res) => res.json([]));
app.get('/api/products', (req, res) => res.json([]));
app.get('/api/rare', (req, res) => res.json({ rare: true }));

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (performance comparison):**
```
Global middleware:      ~2,500 req/s
Path-scoped middleware: ~4,200 req/s  (68% improvement)
```

**Why this output:** By scoping middleware to `/api`, static file requests and health checks bypass Helmet, CORS, compression, and JSON parsing. This reduces the number of function calls per request, increasing throughput. The performance difference grows with the number of middleware functions and the volume of non-API traffic. 

### Real-World Cases

- **Static asset servers:** Serving images, CSS, and JS without running API middleware.
- **Health check endpoints:** Kubernetes liveness probes hitting `/health` thousands of times per minute.
- **Microservices:** Scoping auth middleware to protected routes only, leaving public routes unauthenticated.

---

## Core Concept 2: JSON Serialisation

### Definitions

**Core Definition:** JSON serialisation is the process of converting JavaScript objects into JSON strings for transmission in HTTP responses. `res.json()` internally calls `JSON.stringify()` and sets the `Content-Type` header.

**Technical Definition:** Express's `res.json()` method serialises the provided object using `JSON.stringify()` (or a custom replacer if `app.set('json replacer', ...)` is configured), sets `Content-Type: application/json`, and sends the response. The cost of serialisation is proportional to the size and complexity of the object. Reducing payload weight means sending fewer fields (sparse fieldsets), excluding null/undefined values, and flattening nested structures where possible. 

**Beginner-Friendly Explanation:** When you send an object from your server, Express converts it to a text string (JSON) that can travel over the network. The bigger the object, the longer the conversion takes and the more bandwidth it uses. You can speed this up by sending only the fields the client needs and by using a faster serialisation library.

### Purposes

- To reduce the CPU cost of converting objects to JSON.
- To reduce the payload size and thus the network transfer time.
- To avoid serialising fields the client does not need.
- To prevent accidental exposure of internal fields.

### Syntax Rules and Structure

```js
// ❌ Sends all fields including internal ones
app.get('/users', async (req, res) => {
  const users = await User.find();  // Includes passwordHash, __v, etc.
  res.json(users);
});

// ✅ Sends only the fields the client needs
app.get('/users', async (req, res) => {
  const users = await User.find().select('id name email');
  res.json(users.map(u => ({ id: u.id, name: u.name, email: u.email })));
});
```

| Optimisation | Impact |
|-------------|--------|
| Exclude internal fields | Smaller payload, less serialisation. |
| Remove null/undefined | Smaller payload. |
| Flatten nested objects | Faster serialisation. |
| Use `select()` (Mongoose) | Less data fetched and serialised. |
| Sparse fieldsets | Client requests only needed fields. |

**Rules:**
- Never send entire database documents — select only the fields the client needs. 
- Use `.lean()` in Mongoose for read-only queries to skip document hydration.
- Avoid circular references — they cause `JSON.stringify` to throw.
- Consider `res.json()` with a replacer to strip sensitive fields globally.

### Annotated Code Example

```js
// json-serialization.js
const express = require('express');
const app = express();

// Simulated data with internal fields
const users = [
  { id: 1, name: 'Alice', email: 'alice@test.com', passwordHash: 'xxx', __v: 0 },
  { id: 2, name: 'Bob', email: 'bob@test.com', passwordHash: 'yyy', __v: 0 }
];

// ❌ Unoptimised — sends everything
app.get('/users/full', (req, res) => {
  res.json(users);
});

// ✅ Optimised — sends only needed fields
app.get('/users/lean', (req, res) => {
  res.json(users.map(u => ({ id: u.id, name: u.name, email: u.email })));
});

// ✅ Sparse fieldsets — client selects fields
app.get('/users/sparse', (req, res) => {
  const fields = (req.query.fields || 'id,name,email').split(',');
  res.json(users.map(u => {
    const result = {};
    fields.forEach(f => { if (u[f] !== undefined) result[f] = u[f]; });
    return result;
  }));
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /users/full`):**
```json
[{"id":1,"name":"Alice","email":"alice@test.com","passwordHash":"xxx","__v":0},{"id":2,"name":"Bob","email":"bob@test.com","passwordHash":"yyy","__v":0}]
```

**Expected Output (for `GET /users/lean`):**
```json
[{"id":1,"name":"Alice","email":"alice@test.com"},{"id":2,"name":"Bob","email":"bob@test.com"}]
```

**Expected Output (for `GET /users/sparse?fields=id,name`):**
```json
[{"id":1,"name":"Alice"},{"id":2,"name":"Bob"}]
```

**Why this output:** The full endpoint sends `passwordHash` and `__v` (internal fields) — a security risk and a performance cost. The lean endpoint sends only the three fields the client needs, reducing the payload by 40%. The sparse endpoint allows the client to request exactly the fields it needs, minimising both serialisation cost and bandwidth.

### Real-World Cases

- **Mobile clients:** Reducing payload weight for users on slow connections.
- **Public APIs:** Preventing accidental exposure of internal fields.
- **High-traffic endpoints:** Reducing serialisation CPU and bandwidth costs.

---

## Core Concept 3: Compression

### Definitions

**Core Definition:** Compression middleware reduces the size of HTTP responses by applying Gzip or Brotli deflation algorithms, decreasing bandwidth usage and improving transfer times at the cost of some CPU.

**Technical Definition:** The `compression` middleware wraps `res.write()` and `res.end()` to compress the response body before it is sent. It negotiates the algorithm based on the client's `Accept-Encoding` header: Brotli (`br`) is preferred for its superior compression ratio, falling back to Gzip (`gzip`), and Deflate (`deflate`). Compression is most effective for text-based responses (JSON, HTML, CSS, JavaScript) and least effective for already-compressed content (JPEG, PNG, MP4). The `threshold` option (default 1 KB) prevents compressing small responses where the overhead outweighs the benefit. 

**Beginner-Friendly Explanation:** Compression shrinks your response before sending it. If your JSON response is 100 KB, Gzip might reduce it to 20 KB, and Brotli to 18 KB. The client decompresses it automatically. This is especially important for mobile users and high-traffic APIs where bandwidth is a bottleneck.

### Purposes

- To reduce bandwidth consumption and transfer time.
- To improve perceived performance for clients on slow connections.
- To reduce egress costs in cloud environments.
- To support Brotli for modern clients and Gzip for legacy clients.

### Syntax Rules and Structure

```js
const compression = require('compression');

app.use(compression({
  level: 6,           // Compression level (0–9 for gzip; 0–11 for brotli)
  threshold: 1024,    // Don't compress responses below 1KB
  filter: (req, res) => {
    // Don't compress Server-Sent Events
    if (res.getHeader('Content-Type') === 'text/event-stream') return false;
    return compression.filter(req, res);
  }
}));
```

| Option | Default | Purpose |
|--------|---------|---------|
| `level` | `zlib.constants.Z_DEFAULT_COMPRESSION` | Compression level. |
| `threshold` | `1024` (1KB) | Minimum size to compress. |
| `filter` | — | Function to decide whether to compress. |
| `brotli` | `{ enabled: true }` | Brotli-specific options. |

**Rules:**
- Place `compression()` before routes but after static file serving (or let `express.static` handle its own compression). 
- Do not compress already-compressed formats (images, videos) — the filter handles this automatically. 
- Use `threshold` to avoid compressing tiny responses. 
- Brotli is preferred for HTTPS; Gzip is the fallback. 
- Compression consumes CPU — measure the trade-off under load.

### Annotated Code Example

```js
// compression.js
const express = require('express');
const compression = require('compression');
const app = express();

// Compression middleware with Brotli and Gzip
app.use(compression({
  level: 6,
  threshold: 1024,
  brotli: { enabled: true, zlib: {} },
  filter: (req, res) => {
    if (req.headers['x-no-compression']) return false;
    return compression.filter(req, res);
  }
}));

// Large JSON response
app.get('/api/large', (req, res) => {
  const data = Array.from({ length: 1000 }, (_, i) => ({
    id: i,
    name: `Item ${i}`,
    description: 'A detailed description of this item with some repetition'
  }));
  res.json(data);
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (headers):**
```
# With Accept-Encoding: br
Content-Encoding: br
Content-Length: 8420

# With Accept-Encoding: gzip
Content-Encoding: gzip
Content-Length: 12450

# Without compression
Content-Length: 145000
```

**Why this output:** The 145 KB JSON response is compressed to 12 KB with Gzip (91% reduction) and 8.4 KB with Brotli (94% reduction). The client automatically decompresses the response. The `Content-Encoding` header tells the client which algorithm was used. 

### Real-World Cases

- **Public APIs:** Reducing bandwidth for millions of requests per day.
- **Mobile applications:** Faster responses on slow networks.
- **SPAs:** Compressing large JavaScript bundles served by the API.

---

## Core Concept 4: Response Caching

### Definitions

**Core Definition:** Response caching stores the result of an expensive operation (database query, computation) in memory or an external store, so that subsequent requests for the same data are served without repeating the work.

**Technical Definition:** Caching can be implemented at multiple levels: **in-memory** (Node.js `Map` or `node-cache`), **distributed** (Redis, Memcached), or **HTTP-level** (Cache-Control headers, ETags). For Express, the most effective pattern is a middleware that checks the cache before the route handler, and a cache invalidation strategy that clears or updates cached entries when the underlying data changes. Cache keys are typically derived from the request URL and query parameters; cache TTLs balance freshness against load reduction.

**Beginner-Friendly Explanation:** If 1,000 users request the same product list in a minute, there is no need to query the database 1,000 times. You query it once, store the result in memory, and serve the cached copy to the next 999 users. The cache expires after a set time, at which point you query the database again. 

### Purposes

- To eliminate repeated database queries for the same data.
- To reduce response latency for cache hits.
- To reduce load on databases and downstream services.
- To improve throughput under high traffic.

### Syntax Rules and Structure

**In-memory caching:**
```js
const NodeCache = require('node-cache');
const cache = new NodeCache({ stdTTL: 60 });  // 60-second TTL

app.get('/api/products', async (req, res) => {
  const cacheKey = 'products:all';
  const cached = cache.get(cacheKey);
  if (cached) return res.json(cached);

  const products = await Product.find();
  cache.set(cacheKey, products);
  res.json(products);
});
```

**Redis caching:**
```js
const Redis = require('ioredis');
const redis = new Redis();

app.get('/api/products', async (req, res) => {
  const cacheKey = 'products:all';
  const cached = await redis.get(cacheKey);
  if (cached) return res.json(JSON.parse(cached));

  const products = await Product.find();
  await redis.setex(cacheKey, 60, JSON.stringify(products));
  res.json(products);
});
```

**Cache middleware pattern:**
```js
function cacheMiddleware(ttl = 60) {
  return (req, res, next) => {
    const key = `cache:${req.originalUrl}`;
    const cached = cache.get(key);
    if (cached) {
      res.set('X-Cache', 'HIT');
      return res.json(cached);
    }
    res.set('X-Cache', 'MISS');
    res.sendResponse = res.json;
    res.json = (body) => {
      cache.set(key, body, ttl);
      res.sendResponse(body);
    };
    next();
  };
}

app.get('/api/products', cacheMiddleware(60), async (req, res) => {
  res.json(await Product.find());
});
```

| Cache Type | Storage | Speed | Persistence |
|-----------|---------|-------|-------------|
| In-memory (`node-cache`) | Process memory | Fastest | Lost on restart. |
| Redis | External server | Fast | Persistent, shared. |
| HTTP (ETag/Cache-Control) | Client/browser | Fastest | Per-client. |

**Rules:**
- Cache only idempotent GET requests — never cache POST/PUT/DELETE. 
- Use a TTL that balances freshness against load reduction. 
- Invalidate the cache when the underlying data changes (write-through or explicit invalidation). 
- Add an `X-Cache: HIT/MISS` header for debugging. 
- Use Redis for distributed deployments where multiple instances must share the cache. 

### Annotated Code Example

```js
// response-caching.js
const express = require('express');
const NodeCache = require('node-cache');
const app = express();

const cache = new NodeCache({ stdTTL: 60, checkperiod: 120 });

// Cache middleware
function cacheMiddleware(ttl) {
  return (req, res, next) => {
    const key = `cache:${req.originalUrl}`;
    const cached = cache.get(key);

    if (cached) {
      res.set('X-Cache', 'HIT');
      return res.json(cached);
    }

    res.set('X-Cache', 'MISS');
    const originalJson = res.json.bind(res);
    res.json = (body) => {
      cache.set(key, body, ttl);
      return originalJson(body);
    };
    next();
  };
}

// Expensive endpoint — cached for 60 seconds
app.get('/api/analytics', cacheMiddleware(60), async (req, res) => {
  // Simulate expensive computation
  const data = await computeAnalytics();  // Takes 500ms
  res.json(data);
});

// Invalidate cache on data change
app.post('/api/analytics/refresh', (req, res) => {
  cache.flushAll();
  res.json({ message: 'Cache cleared' });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (first request):**
```
X-Cache: MISS
(500ms response time)
```

**Expected Output (subsequent requests within 60s):**
```
X-Cache: HIT
(2ms response time)
```

**Why this output:** The first request misses the cache and computes the analytics (500ms). The result is stored in memory with a 60-second TTL. Subsequent requests within 60 seconds hit the cache and return in 2ms. After 60 seconds, the cache expires and the next request recomputes. 

### Real-World Cases

- **Dashboard analytics:** Caching expensive aggregation queries for 1–5 minutes.
- **Product catalogues:** Caching product listings that change infrequently.
- **Configuration endpoints:** Caching feature flags and app configuration.
- **External API responses:** Caching third-party API data to avoid rate limits.

---

## Core Concept 5: Connection Pooling

### Definitions

**Core Definition:** Connection pooling reuses a set of open TCP connections (sockets) for outbound HTTP requests, avoiding the cost of establishing a new connection for every request.

**Technical Definition:** Every outbound HTTP request that creates a new connection incurs a TCP handshake (1 round trip), a TLS handshake (1–2 round trips), and then the actual request. With `keep-alive`, the connection remains open and can be reused for subsequent requests to the same host. Node.js 18+ includes `undici` as the built-in HTTP client, which supports connection pooling via `Agent` with `connections` and `pipelining` options. For `axios`, you configure an `https.Agent` with `keepAlive: true`. 

**Beginner-Friendly Explanation:** Imagine calling a customer service line. If you hang up and call back for every question, you waste time on hold music each time. If you stay on the line and ask all your questions in one call, you save that overhead. Connection pooling is staying on the line — reusing the same connection for multiple requests to the same service.

### Purposes

- To eliminate TCP and TLS handshake overhead for repeated outbound requests.
- To reduce latency for internal microservice calls.
- To reduce CPU usage from repeated TLS negotiations.
- To improve throughput by reusing established connections.

### Syntax Rules and Structure

**Undici (Node.js 18+ built-in):**
```js
const { Agent, request } = require('undici');

const agent = new Agent({
  connections: 100,        // Maximum connections per origin
  pipelining: 10,          // Requests per connection before rotation
  keepAliveTimeout: 60000, // 60 seconds
  keepAliveMaxTimeout: 600000
});

// Use the agent for requests
const { statusCode, body } = await request('http://service-b/api/data', {
  dispatcher: agent
});
```

**Axios with keep-alive:**
```js
const https = require('https');
const axios = require('axios');

const agent = new https.Agent({
  keepAlive: true,
  maxSockets: 100,
  maxFreeSockets: 10,
  timeout: 60000
});

const client = axios.create({ httpsAgent: agent });
```

| Library | Pooling Mechanism | Configuration |
|---------|-------------------|---------------|
| Undici | `Agent` | `connections`, `pipelining`. |
| Axios | `https.Agent` | `keepAlive`, `maxSockets`. |
| Node `http` | `http.Agent` | `keepAlive`, `maxSockets`. |
| `got` | `agent` option | `keepAlive`, `maxSockets`. |

**Rules:**
- Always enable `keepAlive: true` for outbound requests to internal services. 
- Set `maxSockets` to match the expected concurrency. 
- Set `keepAliveTimeout` shorter than the server's timeout to avoid using closed connections. 
- Reuse the same agent instance across all requests to the same origin. 
- For databases, use the driver's built-in pool (`pg.Pool`, `mongoose` connection pool). 

### Annotated Code Example

```js
// connection-pooling.js
const express = require('express');
const { Agent, request } = require('undici');

const app = express();

// Create a shared agent with connection pooling
const agent = new Agent({
  connections: 100,
  pipelining: 10,
  keepAliveTimeout: 60000,
  keepAliveMaxTimeout: 600000
});

app.get('/api/aggregate', async (req, res) => {
  // Reuse the same agent for multiple downstream calls
  const [users, products, orders] = await Promise.all([
    request('http://user-service/api/users', { dispatcher: agent }).then(r => r.body.json()),
    request('http://product-service/api/products', { dispatcher: agent }).then(r => r.body.json()),
    request('http://order-service/api/orders', { dispatcher: agent }).then(r => r.body.json())
  ]);

  res.json({ users, products, orders });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (performance comparison):**
```
Without pooling: ~180ms per aggregate request (3 TCP + TLS handshakes)
With pooling:    ~45ms per aggregate request (connections reused)
```

**Why this output:** The first request establishes three connections (one per service). Subsequent requests reuse those connections, eliminating the handshake overhead. The `pipelining: 10` option allows up to 10 requests to be sent on a single connection before it is rotated, further improving throughput. 

### Real-World Cases

- **API gateways:** Aggregating data from multiple microservices.
- **BFF (Backend-for-Frontend):** Calling multiple internal services per user request.
- **Third-party APIs:** Reusing connections to Stripe, SendGrid, or AWS services.

---

## Core Concept 6: High-Speed Serialisation Libraries

### Definitions

**Core Definition:** High-speed serialisation libraries replace the native `JSON.stringify()` with schema-based serialisers that generate optimised code for a specific object shape, producing JSON 2–5× faster.

**Technical Definition:** `fast-json-stringify` compiles a JSON Schema into a highly optimised serialisation function. Because it knows the exact shape of the object (types, properties, required fields), it can skip runtime type checks and use direct property access. It is the serialiser used internally by Fastify. For extremely large payloads, `JSONStream` or `stream-json` enable streaming serialisation. Benchmarks show that `fast-json-stringify` outperforms `JSON.stringify` by 2–5× depending on schema complexity and payload size. 

**Beginner-Friendly Explanation:** `JSON.stringify` is a general-purpose tool — it handles any object. `fast-json-stringify` is a specialist — you tell it the exact shape of your data, and it generates a function that converts that shape to JSON as fast as possible. For APIs that return the same shape of data thousands of times per second, this can significantly reduce CPU usage.

### Purposes

- To reduce the CPU cost of serialising JSON responses.
- To enforce a consistent schema for all serialised output.
- To improve throughput for high-volume APIs.
- To eliminate unnecessary fields automatically (based on the schema).

### Syntax Rules and Structure

```js
const fastJson = require('fast-json-stringify');

// Define the schema
const stringify = fastJson({
  type: 'object',
  properties: {
    id: { type: 'integer' },
    name: { type: 'string' },
    email: { type: 'string' }
  },
  required: ['id', 'name']
});

const json = stringify({ id: 1, name: 'Alice', email: 'alice@test.com', password: 'secret' });
// → '{"id":1,"name":"Alice","email":"alice@test.com"}'
// Note: password is excluded because it is not in the schema
```

| Library | Approach | Best For |
|---------|----------|----------|
| `fast-json-stringify` | Schema-compiled | APIs with known response shapes. |
| `JSON.stringify` | Runtime reflection | General-purpose. |
| `stream-json` | Streaming | Very large payloads. |
| `protobufjs` | Binary protocol | gRPC, high-performance RPC. |

**Rules:**
- Define the schema for every response type. 
- `fast-json-stringify` excludes fields not in the schema — useful for security. 
- Use `additionalProperties: false` to reject unexpected fields. 
- Benchmark before and after — the gain depends on payload size and request volume. 
- `fast-json-stringify` is not a drop-in replacement for `JSON.stringify` — it requires a schema. 

### Annotated Code Example

```js
// fast-json.js
const express = require('express');
const fastJson = require('fast-json-stringify');

const app = express();

// Schema for the User response
const stringifyUser = fastJson({
  type: 'object',
  properties: {
    id: { type: 'integer' },
    name: { type: 'string' },
    email: { type: 'string' }
  },
  required: ['id', 'name']
});

// Schema for the User list response
const stringifyUsers = fastJson({
  type: 'array',
  items: {
    type: 'object',
    properties: {
      id: { type: 'integer' },
      name: { type: 'string' },
      email: { type: 'string' }
    }
  }
});

app.get('/api/users', (req, res) => {
  const users = [
    { id: 1, name: 'Alice', email: 'alice@test.com', passwordHash: 'xxx' },
    { id: 2, name: 'Bob', email: 'bob@test.com', passwordHash: 'yyy' }
  ];
  res.set('Content-Type', 'application/json');
  res.send(stringifyUsers(users));
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output:**
```json
[{"id":1,"name":"Alice","email":"alice@test.com"},{"id":2,"name":"Bob","email":"bob@test.com"}]
```

**Why this output:** The schema excludes `passwordHash` automatically — any field not defined in the schema is omitted. The serialiser is compiled once at startup and reused for every request. Benchmarks show a 2–5× speed improvement over `JSON.stringify` for this type of repeated serialisation. 

### Real-World Cases

- **High-volume APIs:** Thousands of identical responses per second.
- **Public APIs with stable schemas:** Enforcing response consistency.
- **Security-sensitive APIs:** Automatically excluding internal fields not in the schema.

---

## Core Concept 7: Asynchronous Error Wrapping

### Definitions

**Core Definition:** Asynchronous error wrapping is the practice of handling rejected Promises in route handlers. In Express 4, async errors are not caught automatically, requiring `try/catch` or an `asyncHandler` wrapper. In Express 5, async errors are forwarded to error-handling middleware automatically, eliminating the need for wrappers.

**Technical Definition:** In Express 4, if an `async` route handler throws or rejects, the error is not passed to the error-handling middleware — the request hangs. The common workaround is a higher-order function: `const asyncHandler = fn => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next)`. This adds a `Promise.resolve()` call and a `.catch()` to every async route. Express 5 (released 2024, stable 2025) uses `router` improvements to detect rejected Promises and forward them to `next(err)` natively, eliminating the wrapper overhead. 

**Beginner-Friendly Explanation:** In Express 4, if your async route throws an error, Express does not catch it — the request hangs forever. You have to wrap every async route in a `try/catch` or a helper function. In Express 5, Express catches async errors for you, so you can write async routes naturally without the wrapper. 

### Purposes

- To prevent unhandled Promise rejections from hanging requests.
- To forward async errors to the global error-handling middleware.
- To eliminate the boilerplate and overhead of manual error wrapping (Express 5).
- To ensure consistent error responses for both sync and async routes.

### Syntax Rules and Structure

**Express 4 — manual wrapper:**
```js
// Without wrapper — request hangs on error
app.get('/user', async (req, res) => {
  const user = await db.findUser(req.query.id);  // If this throws, request hangs
  res.json(user);
});

// With wrapper — error forwarded to error handler
const asyncHandler = fn => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

app.get('/user', asyncHandler(async (req, res) => {
  const user = await db.findUser(req.query.id);
  res.json(user);
}));
```

**Express 5 — native async error handling:**
```js
// No wrapper needed — Express 5 catches async errors
app.get('/user', async (req, res) => {
  const user = await db.findUser(req.query.id);  // Errors forwarded automatically
  res.json(user);
});
```

| Version | Async Error Handling | Wrapper Required |
|---------|---------------------|-----------------|
| Express 4 | Manual | Yes. |
| Express 5 | Native | No. |

**Rules:**
- **Express 4:** Use `asyncHandler` or `try/catch` for every async route. 
- **Express 5:** Remove `asyncHandler` wrappers — they add unnecessary overhead. 
- Always register the error-handling middleware last. 
- Ensure the error handler has 4 arguments: `(err, req, res, next)`. 
- Test that async errors produce the correct status code and error format.

### Annotated Code Example

```js
// async-error-handling.js
const express = require('express');
const app = express();

// Express 5 — native async error handling (no wrapper needed)
app.get('/api/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  if (!user) {
    const err = new Error('User not found');
    err.statusCode = 404;
    throw err;  // Express 5 forwards this to the error handler
  }
  res.json(user);
});

// Error-handling middleware (must be last, must have 4 args)
app.use((err, req, res, next) => {
  console.error(err);
  res.status(err.statusCode || 500).json({
    error: err.name || 'InternalServerError',
    message: err.message
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/users/999`):**
```json
{"error":"Error","message":"User not found"}
```

**Expected Output (for `GET /api/users/1` when the database is down):**
```json
{"error":"MongoNetworkError","message":"connection timed out"}
```

**Why this output:** In Express 5, the `throw err` inside the async handler is caught automatically and forwarded to the error-handling middleware. The middleware formats the response with the appropriate status code. No `asyncHandler` wrapper is needed — the code is cleaner and the overhead is eliminated. 

### Real-World Cases

- **Express 4 migration:** Removing `asyncHandler` wrappers when upgrading to Express 5.
- **New projects:** Using Express 5's native async error handling from the start.
- **Consistent error responses:** Ensuring all errors (sync and async) produce the same JSON format.

---

## References

- Express.js Performance Best Practices — https://expressjs.com/en/advanced/best-practice-performance.html
- Express.js Production Best Practices: Performance and Reliability — https://expressjs.com/en/advanced/best-practice-performance.html
- Middleware and Route Performance in Express (Hackernoon) — https://hackernoon.com
- JSON Serialization Performance in Node.js (Fastify) — https://fastify.dev/docs/latest/Reference/Validation-and-Serialization/
- fast-json-stringify on npm — https://www.npmjs.com/package/fast-json-stringify
- compression middleware for Express — https://expressjs.com/en/resources/middleware/compression.html
- connection pooling in Node.js (Undici) — https://undici.nodejs.org/#/docs/api/Agent
- Express 5 async error handling — https://expressjs.com/en/guide/error-handling.html
- Node.js Async Error Handling (StrongLoop) — https://strongloop.com/strongblog/async-error-handling-expressjs-es7-promises-generators/
- express-prom-bundle (performance monitoring) — https://github.com/jochen-schweizer/express-prom-bundle
- node-cache on npm — https://www.npmjs.com/package/node-cache
- ioredis on npm — https://www.npmjs.com/package/ioredis
- Brotli compression in Node.js — https://nodejs.org/api/zlib.html#class-brotlioptions
- Express.js 5.x Migration Guide — https://expressjs.com/en/guide/migrating-5.html
- fast-json-stringify benchmark — https://github.com/fastify/fast-json-stringify#benchmark