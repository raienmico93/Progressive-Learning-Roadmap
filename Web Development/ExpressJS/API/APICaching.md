# API Caching — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **API caching** is the technique of storing copies of API responses (or computed results) in a **fast, secondary data store** so that subsequent identical or equivalent requests can be served without re-executing the full backend pipeline. It is a **performance and resilience mechanism** that trades data freshness for latency, throughput, and backend load reduction.

**Technical Definition.** API caching operates by keying stored responses against a **cache key** derived from the request (URL path, query parameters, relevant headers such as `Authorization` or `Accept`), storing the response body and metadata (status, headers) with a **Time-To-Live (TTL)**, and serving the stored entry on a **cache hit** while forwarding to the origin on a **cache miss**. The strategy — **cache-aside**, **read-through**, **write-through**, **write-back**, or **write-around** — determines who manages the cache and how writes interact with it. HTTP caching headers (`Cache-Control`, `ETag`, `Last-Modified`, `Vary`) govern **client-side and intermediary caches**; server-side stores such as **Redis** govern **application-level caching**. **Invalidation** — TTL expiry, event-driven purge, tag-based purge, or versioned keys — is the hardest part: “the greatest challenge in caching is not storage but invalidation.”

**Beginner-Friendly Explanation.** API caching is **keeping a copy of an answer so you do not have to ask the question again**. If 1,000 users request the same product catalog in a minute, the server does not need to query the database 1,000 times. It answers the first request, **stores the answer**, and serves the stored copy to the next 999. This makes responses dramatically faster and protects the backend. The catch is **freshness**: the stored copy can become stale. Caching is therefore a balance between **speed** (serve from cache) and **accuracy** (serve from the source).

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Key-based lookup** | Every cached entry is identified by a **cache key** derived from the request. A wrong key serves the wrong data to the wrong user. |
| **TTL-driven freshness** | Entries expire after a configured **Time-To-Live**; the TTL controls the trade-off between freshness and origin load. |
| **Layered** | Caching happens at multiple tiers: browser (private), CDN/edge (shared), API gateway, and application (Redis/memory). |
| **Strategy-dependent** | The choice of cache-aside, read-through, write-through, write-back, or write-around determines consistency and write behavior. |
| **Header-governed for HTTP caches** | `Cache-Control`, `ETag`, `Last-Modified`, and `Vary` control how browsers, CDNs, and proxies cache responses. |
| **Invalidation is the hard part** | TTL, event-driven purge, tag-based purge, and versioned keys are the primary invalidation mechanisms. |

---

### Prerequisites

- **HTTP fundamentals**: status codes, headers (`Cache-Control`, `ETag`, `Last-Modified`, `Vary`), and conditional requests (`If-None-Match`, `If-Modified-Since`).
- **REST fundamentals**: resources, endpoints, and representations.
- **Express.js basics**: middleware, routing, and async route handlers.
- **Node.js and npm**: to install and use `apicache`, `ioredis`, and related packages.
- **Redis basics**: key-value storage, `GET`/`SET`/`SETEX`/`DEL`, and TTL semantics.
- **A tool to test APIs**: `curl`, Postman, or a browser.

---

### Related Programming Areas

- **API design**: caching is part of the API contract; cache headers are part of the interface.
- **API rate limiting**: caching reduces the load that rate limiting must protect against.
- **CDN and edge computing**: HTTP caching headers govern CDN behavior.
- **Distributed systems**: Redis-backed caches synchronize state across application instances.
- **Database performance**: caching offloads read-heavy workloads from primary databases.

---

### Core Concepts / Features

The following six sections cover the requested concepts. Each follows the uniform structure: definitions, purposes, syntax rules, annotated executable examples with expected outputs, and real-world cases.

---

## 1. Caching

### Definitions

**Core Definition.** **Caching** is the general technique of storing computed or retrieved data in a **temporary, high-speed storage layer** so that future requests for the same data can be served faster, reducing latency and backend load.

**Technical Definition.** A cache is a **key-value store** with a **bounded lifetime** (TTL) and, often, a **bounded size** (eviction policy such as LRU). In API caching, the cached value is typically a **serialized response** (body plus status and relevant headers), and the key is derived from the request’s **path, query parameters, and selected headers**. The primary caching strategies are **cache-aside** (the application manages the cache explicitly: check cache, miss → fetch from DB → store), **read-through** (the cache itself fetches from the database on a miss), **write-through** (writes go through the cache to the database), **write-back** (writes go to the cache and are asynchronously persisted), and **write-around** (writes go directly to the database, bypassing the cache).

**Beginner-Friendly Explanation.** Caching is **a shortcut for repeated questions**. Imagine a librarian who keeps the most-requested books on a desk near the entrance. The first time someone asks for a book, the librarian walks to the shelves. The second time, the book is already on the desk. Caching works the same way: the first request is slow (cache miss), but subsequent requests are fast (cache hit). The **strategy** determines **who** puts the book on the desk and **when** it gets replaced.

### Purposes

- **To reduce latency** by serving responses from fast storage instead of re-executing the full backend pipeline.
- **To reduce backend load** by absorbing redundant read traffic before it reaches the database or upstream services.
- **To improve resilience** by serving cached responses during backend degradation or outages.
- **To lower infrastructure costs** by reducing compute, database, and bandwidth consumption.
- **To improve user experience** through faster page loads and API responses.

### Syntax Rules and Structure

**Cache-aside pattern (the most common strategy).**

```
1. Check the cache for the key.
2. If present (hit), return the cached value.
3. If absent (miss), fetch from the source.
4. Store the fetched value in the cache with a TTL.
5. Return the value.
```

**Strategy comparison.**

| Strategy | Who Manages Cache | Write Behavior | Consistency | Best For |
|---|---|---|---|---|
| **Cache-aside** | Application | Writes go to DB; cache is invalidated | Eventual | General-purpose reads |
| **Read-through** | Cache layer | Writes go to DB; cache reads on miss | Eventual | Transparent read caching |
| **Write-through** | Cache layer | Writes go through cache to DB | Strong | Critical data accuracy |
| **Write-back** | Cache layer | Writes go to cache; async to DB | Weak (risk of loss) | High write throughput |
| **Write-around** | Application | Writes go directly to DB | Eventual | Write-heavy workloads |

**Cache key design rules.**

| Rule | Guidance |
|---|---|
| **Include every response-changing attribute** | Path, query parameters, and relevant headers (`Accept`, `Authorization`, `Accept-Language`). |
| **Enforce privacy boundaries** | Never let one user’s cached response be served to another; use per-user keys or `private` directives. |
| **Keep keys predictable** | Use a documented, stable format (e.g., `product:123`, `products:page:1:limit:20`). |

**Constraints and limitations.**

- **Cache-aside requires explicit invalidation** on writes; forgetting it serves stale data.
- **Write-back risks data loss** if the cache fails before persisting to the database.
- **Cache key mistakes are correctness bugs**, not performance bugs — a wrong key serves the wrong data.

### Annotated Complete Step by Step Code Examples

#### Example 1: Cache-aside with a reusable helper (Redis)

```javascript
// Step 1: Import the Redis client.
const redis = require("./redis");

// Step 2: Define a cache-aside helper.
async function withCache(key, ttl, fetchFn) {
  // Step 3: Check the cache first.
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);   // Cache HIT

  // Step 4: Cache MISS — fetch from the source.
  const data = await fetchFn();
  if (data !== null && data !== undefined) {
    // Step 5: Store the result with a TTL.
    await redis.setex(key, ttl, JSON.stringify(data));
  }
  return data;
}

// Step 6: Use it in a route.
app.get("/api/categories", async (req, res) => {
  const categories = await withCache(
    "categories:all",   // cache key
    3600,               // TTL: 1 hour
    () => db.categories.findAll()
  );
  res.json(categories);
});
```

**Expected Output (conceptual):**
```
First request → fetch from database, store in Redis, return data (cache MISS).
Subsequent requests within 1 hour → return from Redis (cache HIT).
```

**Why this output occurs.** The helper checks Redis first. On a miss, it fetches from the database and stores the result with `SETEX`, which sets the key and its TTL atomically. Subsequent requests within the TTL hit the cache and skip the database.

---

#### Example 2: Cache-aside with explicit invalidation on writes

```javascript
// Step 1: Read endpoint with cache-aside.
app.get("/api/products/:id", async (req, res) => {
  const key = `product:${req.params.id}`;
  const cached = await redis.get(key);
  if (cached) return res.json({ source: "cache", data: JSON.parse(cached) });

  const product = await db.products.findById(req.params.id);
  if (!product) return res.status(404).json({ error: "Not found" });
  await redis.setex(key, 600, JSON.stringify(product));  // TTL: 10 min
  res.json({ source: "database", data: product });
});

// Step 2: Write endpoint that invalidates the cache.
app.put("/api/products/:id", express.json(), async (req, res) => {
  const updated = await db.products.update(req.params.id, req.body);
  await redis.del(`product:${req.params.id}`);   // invalidate item cache
  const listKeys = await redis.keys("products:page:*");
  if (listKeys.length > 0) await redis.del(...listKeys);  // invalidate lists
  res.json(updated);
});
```

**Expected Output (conceptual):**
```
GET after PUT → the first GET after the write is a cache MISS (cache was invalidated), then subsequent GETs are HITs with fresh data.
```

**Why this output occurs.** The write endpoint deletes the specific item key and any list keys that may contain the item. This prevents stale data from being served after an update. Without invalidation, the old value would persist until the TTL expired.

---

### Real-World Cases

- **Product catalogs**: cache-aside with a 5–15 minute TTL dramatically reduces database load for read-heavy catalog endpoints.
- **API gateways**: centralized caching at the gateway layer serves shared resources across all clients without backend changes.
- **Session-independent reference data**: currency rates, configuration, and geolocation data are cached with short-to-medium TTLs.

---

## 2. `apicache`

### Definitions

**Core Definition.** **`apicache`** is a simple API response caching middleware for **Express/Node** that uses **plain-English durations** (e.g., `'5 minutes'`, `'1 hour'`) and supports either a **built-in memory engine** or **Redis** as the backing store.

**Technical Definition.** `apicache` exports a `middleware(duration, [toggleMiddleware], [localOptions])` function that caches route responses for the specified duration. It supports **global options** via `apicache.options({...})`, **cache grouping** (via `req.apicacheGroup`) for bulk invalidation, **manual controls** (`apicache.clear(target)`, `apicache.getIndex()`, `apicache.getPerformance()`), and **Redis integration** via the `redisClient` option. When Redis is configured, `apicache` uses it instead of the built-in memory store, enabling cache sharing across processes and instances. The middleware automatically sets a `Cache-Control: max-age` header matching the cache duration, which can be overridden via the `headers` option.

**Beginner-Friendly Explanation.** `apicache` is **the easiest way to add caching to an Express route**. You write `cache('5 minutes')` in front of a route, and it caches the response for five minutes. You can use the built-in memory store for a single server, or connect Redis if you have multiple servers. It also lets you group routes so you can clear a whole group at once — for example, clearing everything related to “products” after a product update.

### Purposes

- **To add route-level response caching to Express with minimal code** — one middleware call per route.
- **To use plain-English durations** (`'5 minutes'`, `'1 day'`) instead of converting everything to milliseconds.
- **To support both memory and Redis stores** with the same API, allowing a smooth transition from single-server to distributed deployments.
- **To enable cache grouping and manual invalidation** through `req.apicacheGroup` and `apicache.clear()`.

### Syntax Rules and Structure

**General syntax (installation and basic usage).**

```bash
npm install apicache
```

```javascript
import express from 'express';
import apicache from 'apicache';

const app = express();
const cache = apicache.middleware;

// Cache a specific route for 5 minutes.
app.get('/api/collection/:id?', cache('5 minutes'), (req, res) => {
  res.json({ foo: 'bar' });   // runs once per 5 minutes
});

// Cache all routes for 5 minutes.
app.use(cache('5 minutes'));

app.listen(3000);
```

**Component breakdown.**

- `cache('5 minutes')` — the middleware factory; the duration uses the `[length] [unit]` format (`'10 minutes'`, `'1 day'`, `'30 seconds'`).
- `cache(duration, toggleMiddleware, localOptions)` — the optional second parameter is a **toggle function** that decides whether a response should be cached.
- `apicache.options({ redisClient })` — global options; when `redisClient` is provided, Redis is used instead of the memory store.

**Key API methods.**

| Method | Purpose |
|---|---|
| `apicache.middleware(duration, [toggle], [options])` | Returns the caching middleware. |
| `apicache.options({ ... })` | Getter/setter for global options (chainable). |
| `apicache.clear([target])` | Clears a specific key, group, or the entire cache. |
| `apicache.getIndex()` | Returns the current cache index (keys). |
| `apicache.getPerformance()` | Returns cache performance statistics. |

**Grouping and manual control syntax.**

```javascript
// Add routes to a group for bulk invalidation.
app.get('/api/:collection/:item?', (req, res) => {
  req.apicacheGroup = req.params.collection;
  res.json({ success: true });
});

// Clear an entire group.
app.get('/api/cache/clear/:target?', (req, res) => {
  res.json(apicache.clear(req.params.target));
});
```

**Constraints and limitations.**

- **The memory store is per-process**; use Redis for multi-instance deployments.
- **Only `200 OK` responses are cached by default**; use a toggle middleware to cache other status codes.
- **`apicache` sets `Cache-Control: max-age` automatically**; override it via the `headers` option if needed.
- **Cache keys are derived from the request URL**, so routes with different query parameters are cached separately.

### Annotated Complete Step by Step Code Examples

#### Example 1: Caching a route with a toggle for successful responses only

```javascript
// app.js
import express from 'express';
import apicache from 'apicache';

const app = express();
const cache = apicache.middleware;

// Step 1: Define a toggle that caches only 200 responses.
const onlyStatus200 = (req, res) => res.statusCode === 200;

// Step 2: Create a caching middleware with the toggle.
const cacheSuccesses = cache('5 minutes', onlyStatus200);

// Step 3: Apply it to routes.
app.get('/api/missing', cacheSuccesses, (req, res) => {
  res.status(404).json({ results: 'will not be cached' });
});

app.get('/api/found', cacheSuccesses, (req, res) => {
  res.json({ results: 'will be cached' });
});

app.listen(3000);
```

**Expected Output (conceptual):**
```
GET /api/found → cached for 5 minutes.
GET /api/missing → NOT cached (status 404); the handler runs every time.
```

**Why this output occurs.** The `onlyStatus200` toggle checks `res.statusCode` after the handler runs. If the status is not 200, `apicache` skips caching. This prevents error responses from being cached and served to subsequent clients.

---

#### Example 2: Redis-backed `apicache` with grouping

```javascript
// app.js
import express from 'express';
import apicache from 'apicache';
import redis from 'redis';

const app = express();

// Step 1: Configure apicache to use Redis.
const cacheWithRedis = apicache
  .options({ redisClient: redis.createClient() })
  .middleware;

// Step 2: Cache routes and assign groups.
app.get('/api/products/:id', cacheWithRedis('10 minutes'), (req, res) => {
  req.apicacheGroup = 'products';
  res.json({ id: req.params.id, name: 'Widget' });
});

// Step 3: Clear the entire "products" group.
app.get('/api/cache/clear/products', (req, res) => {
  res.json(apicache.clear('products'));
});

app.listen(3000);
```

**Expected Output (conceptual):**
```
GET /api/products/1 → cached in Redis for 10 minutes, grouped under "products".
GET /api/cache/clear/products → clears all product-related cached entries.
```

**Why this output occurs.** `apicache.options({ redisClient })` switches the store to Redis, sharing the cache across processes. `req.apicacheGroup = 'products'` adds the route to the `products` group, and `apicache.clear('products')` purges all entries in that group — useful for invalidating related resources after a write.

---

### Real-World Cases

- **Express APIs with simple caching needs**: `apicache` is the fastest way to add route-level caching without a custom implementation.
- **Multi-instance deployments**: Redis-backed `apicache` shares the cache across all server instances.
- **Group-based invalidation**: e-commerce APIs group product-related routes and clear the group after inventory updates.

---

## 3. Redis Caching

### Definitions

**Core Definition.** **Redis caching** is the use of **Redis**, an in-memory key-value data store, as the backing store for an API cache. Redis provides **fast reads and writes**, **TTL-based expiration**, **atomic operations**, and **shared access** across application instances.

**Technical Definition.** Redis stores cached values as **strings** (typically serialized JSON) keyed by a **cache key**. It supports **`SETEX`** (set with TTL), **`GET`**, **`DEL`**, **`EXPIRE`**, and **pipelining** for multi-key operations. Redis is commonly used with the **cache-aside** pattern: the application checks Redis, falls back to the database on a miss, and writes the result back with a TTL. For invalidation, Redis supports **key deletion**, **pattern-based scanning** (`SCAN` with `MATCH`), and **tag-based grouping** (via auxiliary sets). For stampede protection, patterns include **single-flight locks**, **probabilistic early expiration**, and **TTL jitter**. Redis is preferred over an in-memory store when multiple processes or servers must share the same cache state.

**Beginner-Friendly Explanation.** Redis is **a very fast shared notebook** that all your servers can read and write. Instead of each server keeping its own private cache, they all consult the same Redis instance. This means a cache populated by one server is available to all of them. Redis also supports **expiration times**, so entries disappear automatically after a set period. It is the standard choice for production API caching because it is fast, reliable, and supports multiple servers.

### Purposes

- **To share cache state across multiple application instances**, avoiding per-process caches that diverge.
- **To provide fast, atomic cache operations** (`SETEX`, `GET`, `DEL`) with built-in TTL support.
- **To enable sophisticated invalidation** through key deletion, pattern scanning, and tag-based grouping.
- **To protect against cache stampedes** using locks, early expiration, and TTL jitter.

### Syntax Rules and Structure

**General syntax (Redis client setup with `ioredis`).**

```javascript
const Redis = require("ioredis");
const redis = new Redis(process.env.REDIS_URL || "redis://localhost:6379");
```

**Core cache operations.**

| Operation | Redis Command | Purpose |
|---|---|---|
| **Set with TTL** | `SETEX key seconds value` | Store a value that expires after `seconds`. |
| **Get** | `GET key` | Retrieve a cached value. |
| **Delete** | `DEL key [key ...]` | Invalidate specific keys. |
| **Scan for pattern** | `SCAN cursor MATCH pattern COUNT n` | Find keys matching a pattern (non-blocking). |
| **Pipeline** | `MULTI` / `EXEC` or client pipeline | Batch multiple operations. |

**Cache key design rules.**

| Rule | Guidance |
|---|---|
| **Include all response-changing attributes** | Path, query parameters, and relevant headers. |
| **Use a stable, documented format** | `product:123`, `products:page:1:limit:20:cat:all`. |
| **Enforce privacy boundaries** | Per-user keys for authenticated data; never share private responses. |
| **Apply TTL jitter** | Add ±10% random variation to prevent synchronized expiry. |

**Constraints and limitations.**

- **Redis is a separate service**; network latency applies (typically sub-millisecond in the same region).
- **`KEYS` is blocking and dangerous in production**; use `SCAN` for pattern-based invalidation.
- **Stampede risk**: when a hot key expires, many concurrent requests may miss simultaneously; use a single-flight lock or stale-while-revalidate.

### Annotated Complete Step by Step Code Examples

#### Example 1: Cache-aside with `ioredis` and TTL

```javascript
// redis.js
const Redis = require("ioredis");
const client = new Redis(process.env.REDIS_URL || "redis://localhost:6379");
client.on("connect", () => console.log("Redis connected"));
client.on("error", (err) => console.error("Redis error:", err.message));
module.exports = client;
```

```javascript
// app.js
const express = require("express");
const redis = require("./redis");
const app = express();

app.get("/api/products/:id", async (req, res) => {
  const key = `product:${req.params.id}`;

  // Step 1: Check cache.
  const cached = await redis.get(key);
  if (cached) {
    return res.json({ source: "cache", data: JSON.parse(cached) });
  }

  // Step 2: Cache miss — fetch from database.
  const product = await db.products.findById(req.params.id);
  if (!product) return res.status(404).json({ error: "Not found" });

  // Step 3: Store with a 10-minute TTL.
  await redis.setex(key, 600, JSON.stringify(product));

  res.json({ source: "database", data: product });
});

app.listen(3000);
```

**Expected Output (conceptual):**
```
First request → source: "database", stored in Redis with a 10-minute TTL.
Subsequent requests within 10 minutes → source: "cache".
```

**Why this output occurs.** `redis.get` checks for a cached value. On a miss, the route fetches from the database and uses `redis.setex` to store the JSON-serialized product with a 600-second TTL. This is the canonical cache-aside pattern.

---

#### Example 2: Stampede protection with a single-flight lock

```javascript
// Step 1: Helper that coalesces concurrent misses for the same key.
async function withCacheAndLock(key, ttl, fetchFn) {
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);

  const lockKey = `lock:${key}`;
  const lock = await redis.set(lockKey, "1", "NX", "EX", 10);  // 10s lock

  if (!lock) {
    // Another request is already fetching; wait briefly and retry.
    await new Promise((r) => setTimeout(r, 100));
    return withCacheAndLock(key, ttl, fetchFn);
  }

  try {
    const data = await fetchFn();
    await redis.setex(key, ttl, JSON.stringify(data));
    return data;
  } finally {
    await redis.del(lockKey);
  }
}
```

**Expected Output (conceptual):**
```
When a hot key expires and 100 requests arrive simultaneously, only one fetches from the database; the others wait for the result and read the refreshed cache.
```

**Why this output occurs.** `SET NX EX` creates a short-lived lock only if the lock key does not exist. The first request acquires the lock and fetches; concurrent requests fail to acquire the lock and retry, eventually hitting the refreshed cache. This prevents the **thundering herd** from overwhelming the database.

---

### Real-World Cases

- **Multi-instance Express APIs**: Redis shares cache state across all instances, ensuring consistent responses.
- **E-commerce catalogs**: product data is cached in Redis with short TTLs (e.g., 5–15 minutes) to absorb read spikes during sales.
- **Session-independent reference data**: currency rates and configuration are cached in Redis with medium TTLs and invalidated on change events.

---

## 4. Cache Headers

### Definitions

**Core Definition.** **Cache headers** are HTTP response headers that instruct **browsers, CDNs, proxies, and other intermediaries** on whether to cache a response, for how long, under what conditions, and how to validate it when it becomes stale.

**Technical Definition.** The core HTTP caching headers are **`Cache-Control`** (freshness directives: `max-age`, `s-maxage`, `no-cache`, `no-store`, `must-revalidate`, `stale-while-revalidate`), **`ETag`** (a validator that uniquely identifies a representation version), **`Last-Modified`** (a date-based validator), **`Expires`** (legacy freshness, superseded by `Cache-Control`), and **`Vary`** (declares which request headers make the response variant different). Freshness answers “can I reuse this without asking?”; validators answer “has it changed?”; `Vary` answers “does this response depend on request headers?”. These headers are defined in **RFC 9111** (HTTP Caching).

**Beginner-Friendly Explanation.** Cache headers are **notes on a response that tell every cache in the chain what to do with it**. `Cache-Control: max-age=3600` says “you can reuse this for an hour.” `ETag: "abc123"` says “this is version abc123 — if the client already has that version, don't send the body again.” `Vary: Accept` says “the response differs depending on the `Accept` header, so keep separate copies for each value.” These headers control caching in **browsers and CDNs**, not in your application-level Redis cache.

### Purposes

- **To control freshness** so caches know how long a response remains reusable without revalidation.
- **To enable conditional requests** through `ETag` and `Last-Modified`, so clients and caches can validate cheaply and receive `304 Not Modified` when nothing has changed.
- **To declare cache key variations** through `Vary`, preventing incorrect sharing of responses across different request headers.
- **To prevent caching of sensitive responses** through `private` and `no-store`.

### Syntax Rules and Structure

**General syntax (Cache-Control).**

```
Cache-Control: public, max-age=3600, s-maxage=7200, stale-while-revalidate=60
```

**General syntax (validators).**

```
ETag: "abc123"
Last-Modified: Wed, 01 Feb 2026 12:00:00 GMT
Vary: Accept, Accept-Language, Authorization
```

**Component breakdown.**

- `Cache-Control` — the primary directive header; comma-separated directives, order irrelevant.
- `max-age` — freshness lifetime in **private** caches (browsers).
- `s-maxage` — freshness lifetime in **shared** caches (CDNs, proxies); overrides `max-age`.
- `no-store` — **never** store the response in any cache.
- `private` — only the client’s private cache may store it; shared caches must not.
- `must-revalidate` — once stale, the cache **must** revalidate before serving; prevents serving stale content.
- `stale-while-revalidate` — serve a stale response briefly while refreshing in the background (RFC 5861).
- `ETag` — a version identifier; the client sends `If-None-Match: "abc123"` to validate.
- `Last-Modified` — a timestamp; the client sends `If-Modified-Since` to validate.
- `Vary` — declares which request headers affect the response, expanding the cache key.

**Directive rules.**

| Directive | Applies To | Purpose |
|---|---|---|
| `max-age` | Private caches | Freshness lifetime for browsers. |
| `s-maxage` | Shared caches | Freshness lifetime for CDNs; overrides `max-age`. |
| `no-store` | All caches | Never store (sensitive data). |
| `private` | Shared caches | Do not store in shared caches (user-specific data). |
| `must-revalidate` | All caches | Do not serve stale after expiry without revalidation. |
| `stale-while-revalidate` | All caches | Serve stale briefly while refreshing in background. |
| `no-cache` | All caches | Store but revalidate before each use. |
| `immutable` | All caches | The resource will never change; skip revalidation. |

**Constraints and limitations.**

- **`no-cache` does not mean “do not cache”**; it means “cache but revalidate before use.” Use `no-store` to prevent storage.
- **`Vary: *`** makes a response effectively uncacheable; avoid it unless necessary.
- **`Expires` is legacy**; `Cache-Control` supersedes it in HTTP/1.1+.
- **`Vary` overuse explodes the cache key space** and reduces hit rates.

### Annotated Complete Step by Step Code Examples

#### Example 1: Setting cache headers for a public, cacheable resource

```javascript
// Step 1: Set headers on a GET response.
app.get("/api/public/catalog", (req, res) => {
  const catalog = getCatalog();

  // Step 2: Fresh for 1 hour in browsers, 2 hours in shared caches.
  res.set({
    "Cache-Control": "public, max-age=3600, s-maxage=7200",
    "ETag": '"catalog-v42"',
    "Vary": "Accept-Encoding",
  });

  // Step 3: Handle conditional requests.
  if (req.headers["if-none-match"] === '"catalog-v42"') {
    return res.status(304).end();   // Not Modified
  }

  res.json(catalog);
});
```

**Expected Response (first request):**
```http
HTTP/1.1 200 OK
Cache-Control: public, max-age=3600, s-maxage=7200
ETag: "catalog-v42"
Vary: Accept-Encoding

{ ...catalog... }
```

**Expected Response (conditional request):**
```http
HTTP/1.1 304 Not Modified
ETag: "catalog-v42"
```

**Why this output occurs.** The `Cache-Control` header tells browsers to reuse the response for 1 hour and CDNs for 2 hours. The `ETag` identifies the version; when the client sends `If-None-Match` with the same ETag, the server returns `304 Not Modified` without a body, saving bandwidth. `Vary` ensures separate cached copies for different encodings.

---

#### Example 2: Preventing caching of sensitive responses

```javascript
// Step 1: Set no-store for user-specific sensitive data.
app.get("/api/me/account", authenticate, (req, res) => {
  res.set({
    "Cache-Control": "private, no-store",
  });
  res.json({ balance: req.user.balance });
});
```

**Expected Response:**
```http
HTTP/1.1 200 OK
Cache-Control: private, no-store

{ "balance": 1234.56 }
```

**Why this output occurs.** `private` prevents shared caches from storing the response, and `no-store` prevents even the browser from storing it. This is essential for account balances, personal data, and any response that must never be cached.

---

### Real-World Cases

- **CDN-served static assets**: `Cache-Control: public, max-age=31536000, immutable` for versioned assets (e.g., hashed filenames).
- **API JSON responses**: `Cache-Control: private, max-age=0, must-revalidate` with `ETag` for user-specific data that must be validated on each use.
- **Sensitive endpoints**: `Cache-Control: no-store` for authentication and payment endpoints.

---

## 5. Cache Invalidation

### Definitions

**Core Definition.** **Cache invalidation** is the process of **removing or updating cached entries** when the underlying data changes, so that subsequent requests do not receive stale content. It is widely regarded as the hardest problem in caching.

**Technical Definition.** Invalidation strategies fall into four categories: **TTL-based expiration** (entries expire after a configured duration; simple but can serve stale data up to the TTL), **event-driven invalidation** (a write event triggers an immediate purge of affected keys), **tag-based invalidation** (entries are grouped by tags, and a single tag purge removes all related entries), and **version-based invalidation** (the cache key includes a version identifier; a version bump makes old entries unreachable). The invalidation ladder — from most to least preferred — is: **versioned URLs → `stale-while-revalidate` → surrogate-key purge → path purge → full clear**.

**Beginner-Friendly Explanation.** Cache invalidation is **throwing away the stored answer when it becomes wrong**. If a product’s price changes, the cached price must be updated or removed. There are several ways to do this: let the cache **expire** after a set time (simple but may serve stale data), **delete** the entry immediately when the data changes (event-driven), **group** related entries and clear the whole group (tag-based), or **include a version number** in the key so old entries are ignored (version-based). The key is to **never serve data that is wrong** just because it was cached.

### Purposes

- **To prevent stale data** from being served after writes, deletes, or external changes.
- **To maintain consistency** between the cache and the source of truth.
- **To minimize the performance cost** of invalidation by purging only affected entries.
- **To support complex dependencies** (e.g., a product appears in multiple lists) through tag-based grouping.

### Syntax Rules and Structure

**Invalidation strategy comparison.**

| Strategy | Trigger | Consistency | Complexity | Best For |
|---|---|---|---|---|
| **TTL-based** | Time expiry | Eventual (stale up to TTL) | Low | Data with tolerable staleness |
| **Event-driven** | Write event | Strong (immediate purge) | Medium | Write-heavy, accuracy-critical |
| **Tag-based** | Tag purge | Strong for related entries | Medium | Complex dependencies (product in lists) |
| **Version-based** | Version bump | Strong (old keys ignored) | Low–Medium | Immutable versioned resources |

**Invalidation ladder (prefer the highest you can reach).**

```
Versioned URLs → stale-while-revalidate → surrogate-key (tag) purge → path purge → full clear
```

**Constraints and limitations.**

- **TTL alone cannot guarantee freshness**; it only bounds staleness.
- **Event-driven invalidation requires reliable event delivery**; missed events leave stale entries.
- **Tag-based invalidation requires maintaining tag indexes**; stale tag sets can cause over- or under-purging.
- **Pattern-based deletion (`KEYS`) is blocking** in Redis; use `SCAN` or tag sets instead.

### Annotated Complete Step by Step Code Examples

#### Example 1: Event-driven invalidation on write

```javascript
// Step 1: Write endpoint that updates the database and purges affected keys.
app.put("/api/products/:id", express.json(), async (req, res) => {
  const updated = await db.products.update(req.params.id, req.body);

  // Step 2: Delete the specific product cache.
  await redis.del(`product:${req.params.id}`);

  // Step 3: Delete list caches that may contain the product.
  const listKeys = await redis.keys("products:page:*");
  if (listKeys.length > 0) await redis.del(...listKeys);

  res.json(updated);
});
```

**Expected Output (conceptual):**
```
After PUT /api/products/42, the next GET /api/products/42 is a cache MISS (fresh data fetched and cached). List caches are also cleared.
```

**Why this output occurs.** The write endpoint explicitly deletes the item key and scans for list keys to delete. This event-driven approach ensures that stale entries do not survive the write.

---

#### Example 2: Tag-based invalidation

```javascript
// Step 1: Store a cached entry and associate it with tags.
async function cacheWithTags(key, value, ttl, tags) {
  await redis.setex(key, ttl, JSON.stringify(value));
  for (const tag of tags) {
    await redis.sadd(`tag:${tag}`, key);   // add key to tag set
  }
}

// Step 2: Invalidate all keys associated with a tag.
async function invalidateTag(tag) {
  const keys = await redis.smembers(`tag:${tag}`);
  if (keys.length > 0) await redis.del(...keys);
  await redis.del(`tag:${tag}`);
}

// Step 3: Cache a product and tag it.
await cacheWithTags("product:42", product, 600, ["products", "catalog"]);

// Step 4: Invalidate the "catalog" tag after a catalog-wide update.
await invalidateTag("catalog");
```

**Expected Output (conceptual):**
```
invalidateTag("catalog") deletes all cache entries associated with the "catalog" tag, including product:42.
```

**Why this output occurs.** The `tag:${tag}` Redis set maintains the list of keys associated with each tag. Invalidating a tag retrieves its key set and deletes all keys in one batch. This handles complex dependencies — a product may appear in multiple lists, and clearing the “catalog” tag invalidates all of them.

---

### Real-World Cases

- **E-commerce**: event-driven invalidation of product and category caches after admin updates.
- **Content management**: tag-based invalidation when an article is published or edited, purging all pages that reference it.
- **Versioned assets**: version-based invalidation through fingerprinted filenames (`app.abc123.js`), eliminating the need for active purge.

---

## 6. Caching Best Practices

### Definitions

**Core Definition.** **Caching best practices** are empirically established guidelines for designing, configuring, and operating API caches so that they improve performance and resilience **without serving stale, incorrect, or privacy-violating data**.

**Technical Definition.** Best practices span **endpoint classification** (cacheable, conditionally cacheable, uncacheable), **cache key design** (include every response-changing attribute, enforce privacy boundaries), **TTL strategy** (short TTLs for volatile data, long TTLs for stable data, jitter to prevent synchronized expiry), **stampede protection** (single-flight locks, probabilistic early expiration, `stale-while-revalidate`), **invalidation design** (TTL + event-driven + tag-based), **observability** (hit rate, miss rate, eviction rate, latency), and **security** (never cache private data in shared caches).

**Beginner-Friendly Explanation.** Best practices are **the habits that keep caching from becoming a source of bugs**. Decide which endpoints are safe to cache. Build cache keys that include everything that changes the response. Set TTLs that match how often the data changes. Protect against the “thundering herd” when a hot key expires. Invalidate on writes. Measure your hit rate. And **never cache private data in a shared cache**. These are the things that separate a cache that helps from one that serves wrong answers.

### Purposes

- **To maximize hit rate** while maintaining correctness and privacy.
- **To prevent cache stampedes** that can overwhelm the backend when hot keys expire.
- **To ensure stale data is bounded** by appropriate TTLs and invalidation triggers.
- **To make cache behavior observable** through metrics and monitoring.
- **To choose the right caching strategy** for each endpoint based on its characteristics.

### Syntax Rules and Structure

**Endpoint classification framework.**

| API Type | Cache Suitability | Typical TTL | Example |
|---|---|---|---|
| **Static content** | Excellent | 1 hour – 1 day | API documentation, images |
| **Reference data** | Excellent | 15 min – 1 hour | Product catalogs, currency rates |
| **Aggregated metrics** | Good | 1 min – 15 min | Dashboard statistics |
| **User-specific data** | Fair | 30 sec – 5 min | Per-user profiles (private cache) |
| **Real-time data** | Poor | Avoid caching | Stock prices, live location |

**Best-practice checklist.**

| Area | Best Practice |
|---|---|
| **Endpoint selection** | Cache only read-heavy, low-change endpoints; never cache POST/PUT/DELETE responses by default. |
| **Cache key design** | Include path, query parameters, and relevant headers; enforce privacy with per-user keys or `private`. |
| **TTL** | Match TTL to data volatility; add ±10% jitter to prevent synchronized expiry. |
| **Stampede protection** | Use single-flight locks, `stale-while-revalidate`, or probabilistic early expiration. |
| **Invalidation** | Combine TTL with event-driven purge; use tag-based invalidation for related entries. |
| **Observability** | Track hit rate, miss rate, eviction rate, and latency; set alerts for anomalies. |
| **Security** | Never cache private data in shared caches; use `no-store` for sensitive responses. |

**Constraints and limitations.**

- **There is no universal TTL**; the right TTL depends on the data and the acceptable staleness.
- **Stampede protection adds complexity**; choose the simplest mechanism that meets your concurrency requirements.
- **Observability is not optional**; without metrics, you cannot know whether the cache is helping or hurting.

### Annotated Complete Step by Step Code Examples

#### Example 1: TTL jitter to prevent synchronized expiry

```javascript
// Step 1: Add jitter to a base TTL.
function jitteredTTL(baseSeconds) {
  const jitter = baseSeconds * 0.1; // ±10%
  return Math.floor(baseSeconds + (Math.random() * 2 - 1) * jitter);
}

// Step 2: Use the jittered TTL when caching.
app.get("/api/products/:id", async (req, res) => {
  const key = `product:${req.params.id}`;
  const cached = await redis.get(key);
  if (cached) return res.json(JSON.parse(cached));

  const product = await db.products.findById(req.params.id);
  if (!product) return res.status(404).json({ error: "Not found" });

  const ttl = jitteredTTL(600); // base 10 minutes, jittered
  await redis.setex(key, ttl, JSON.stringify(product));
  res.json(product);
});
```

**Expected Output (conceptual):**
```
Different product keys expire at slightly different times (e.g., 540s, 612s, 597s) instead of all at exactly 600s.
```

**Why this output occurs.** Without jitter, all keys written at the same time expire simultaneously, causing a spike of cache misses (thundering herd). Jitter spreads expiry across a window, smoothing the load on the database.

---

#### Example 2: Observability — tracking cache hit rate

```javascript
// Step 1: Track hits and misses.
let cacheHits = 0;
let cacheMisses = 0;

function trackCache(hit) {
  if (hit) cacheHits++;
  else cacheMisses++;
}

// Step 2: Wrap the cache lookup.
async function getWithMetrics(key, fetchFn, ttl) {
  const cached = await redis.get(key);
  if (cached) {
    trackCache(true);
    return JSON.parse(cached);
  }

  trackCache(false);
  const data = await fetchFn();
  await redis.setex(key, ttl, JSON.stringify(data));
  return data;
}

// Step 3: Expose the metrics.
app.get("/api/cache/stats", (req, res) => {
  const total = cacheHits + cacheMisses;
  res.json({
    hits: cacheHits,
    misses: cacheMisses,
    hitRate: total === 0 ? 0 : cacheHits / total,
  });
});
```

**Expected Output (conceptual):**
```
{ "hits": 8500, "misses": 1500, "hitRate": 0.85 }
```

**Why this output occurs.** The wrapper increments the appropriate counter on each lookup. The `/api/cache/stats` endpoint exposes the hit rate, which is the primary metric for evaluating whether the cache is effective. A low hit rate suggests the TTL is too short, the key is too granular, or the endpoint is not cache-friendly.

---

### Real-World Cases

- **API gateways**: cache hit rates of 70–85% are achievable for read-heavy APIs, translating to backend load reductions of the same magnitude.
- **E-commerce flash sales**: caching absorbs 88% of product catalog requests during peak traffic, preventing backend collapse.
- **Multi-tier caching**: gateway-level caching for shared resources, edge caching for geographic distribution, and application-level Redis for complex aggregations, each with appropriate TTLs.

---

## References

- apicache-extra — npm - https://www.npmjs.com/package/apicache-extra
- apicache — API methods (`clear`, `getIndex`, `middleware`) - https://jspm-packages.deno.dev/package/apicache@1.2.6
- How to Cache Express.js API Responses with Redis — OneUptime - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-redis-cache-express-js-api-responses-with-redis/README.md
- framework-http-caching-headers — Zenodo - https://zenodo.org/records/20405342/preview/framework-http-caching-headers.md
- API Gateway Caching Strategy — API7.ai - https://api7.ai/learning-center/api-gateway-guide/api-gateway-caching-strategy-analysis
- API caching strategies and best practices — TechTarget - https://www.techtarget.com/it-infrastructure/tip/API-caching-strategies-and-best-practices
- Edge Delivery and Cache Invalidation — sujeet.pro - https://raw.githubusercontent.com/sujeet-pro/sujeet.pro/refs/heads/main/content/articles/edge-delivery-and-cache-invalidation/README.md
- Redis Caching Patterns for Node.js — wowhow.cloud - https://wowhow.cloud
- Cache Stampede Prevention — cachee.ai - https://cachee.ai
- RFC 9111 — HTTP Caching - https://www.rfc-editor.org/rfc/rfc9111
- RFC 5861 — HTTP Cache-Control Extensions for Stale Content - https://www.rfc-editor.org/rfc/rfc5861