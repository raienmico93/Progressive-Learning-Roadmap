# API Rate Limiting — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **API rate limiting** is a control mechanism that constrains the number of requests a client may make to an API within a defined time window. It is a **server-side enforcement policy** that protects availability, fairness, and security by rejecting or delaying requests that exceed the allowed quota.

**Technical Definition.** Rate limiting operates by **tracking request counts per client key** (typically an IP address, API key, or user ID) within a **time window** and comparing the count against a configured **limit**. When the limit is exceeded, the server returns **HTTP `429 Too Many Requests`** and, ideally, communicates the quota via **standard rate-limit headers** (`RateLimit-Limit`, `RateLimit-Remaining`, `RateLimit-Reset`, or the legacy `X-RateLimit-*` variants) and `Retry-After`. The underlying algorithm — fixed window, sliding window, token bucket, or leaky bucket — determines how counts are tracked and reset. In Node.js/Express ecosystems, **`express-rate-limit`** is the standard middleware for implementing rate limiting, with optional external stores (Redis, Memcached, PostgreSQL) to synchronize state across multiple processes or servers.

**Beginner-Friendly Explanation.** Rate limiting is **the bouncer at the door of your API**. It sets a rule like “no more than 100 requests per minute per client” and enforces it. If a client exceeds the limit, the server says “slow down” with a `429` status and tells them how long to wait. This protects your servers from being overwhelmed — whether by a buggy script, a malicious attack, or just a very busy user — and ensures everyone gets a fair share of the service.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Server-side enforcement** | The server decides whether to accept, reject, or delay each request based on the client’s recent request history. |
| **Client-keyed** | Limits are applied per client identifier — IP address, API key, user ID, or a custom key. |
| **Window-based** | Requests are counted within a time window (e.g., 100 requests per 15 minutes); the window resets according to the chosen algorithm. |
| **Standards-aware** | Modern implementations emit IETF-draft `RateLimit-*` headers; legacy implementations use `X-RateLimit-*`; `Retry-After` is the authoritative signal when both are present. |
| **Algorithm-dependent** | The choice of fixed window, sliding window, token bucket, or leaky bucket determines burst tolerance, memory usage, and accuracy. |
| **Store-dependent for scale** | The default in-memory store works for single-process deployments; multi-process or multi-server deployments require an external store (Redis, Memcached, PostgreSQL). |

---

### Prerequisites

- **HTTP fundamentals**: status codes (`429`), headers, and the request/response model.
- **Express.js basics**: middleware, routing, and app configuration.
- **Node.js and npm**: to install and run `express-rate-limit`.
- **A tool to test APIs**: `curl`, Postman, or a browser.
- **Basic understanding of time windows and counters**: how requests are counted and reset.

---

### Related Programming Areas

- **API security**: rate limiting is a first line of defense against DDoS, brute-force attacks, and credential stuffing.
- **API design**: rate limits are part of the API contract and should be documented.
- **Distributed systems**: external stores (Redis, Memcached) synchronize rate-limit state across nodes.
- **Client-side resilience**: clients should respect `Retry-After` and implement exponential backoff with jitter.
- **API gateways**: rate limiting is often enforced at the gateway layer (Azure API Management, Kong, AWS API Gateway).

---

### Core Concepts / Features

The following five sections cover the requested concepts. Each follows the uniform structure: definitions, purposes, syntax rules, annotated executable examples with expected outputs, and real-world cases.

---

## 1. Rate Limiting

### Definitions

**Core Definition.** **Rate limiting** is the practice of restricting how many requests a client can make to an API within a specified time period. It is a **defensive control** that protects backend resources, ensures fair access, and mitigates abuse.

**Technical Definition.** Rate limiting maintains a **counter per client key** and a **reset time** (the moment the window expires or the bucket refills). On each request, the server increments the counter, compares it to the configured limit, and either **allows** the request (if under the limit) or **rejects** it with `429 Too Many Requests` (if at or over the limit). The response should include rate-limit headers so well-behaved clients can self-regulate. Rate limiting differs from **throttling** in emphasis: rate limiting rejects excess requests; throttling may **delay** or **shape** them. In practice, the terms are often used interchangeably.

**Beginner-Friendly Explanation.** Rate limiting is **setting a speed limit for API requests**. Just like a highway speed limit prevents accidents and keeps traffic flowing, a request limit prevents your servers from being overwhelmed and keeps the API usable for everyone. If you exceed the limit, you get a `429` response telling you to slow down. The server might also tell you exactly how many requests you have left and when the limit resets.

### Purposes

- **To prevent service degradation** by stopping runaway scripts or aggressive integrations from overwhelming the backend.
- **To enhance security** by mitigating DDoS attacks, brute-force login attempts, and scraping.
- **To ensure fair resource allocation** so one high-volume client cannot monopolize capacity.
- **To manage operational costs** by capping request volume and making infrastructure spending predictable.
- **To communicate limits clearly** so clients can self-regulate and avoid being blocked.

### Syntax Rules and Structure

**General syntax (`express-rate-limit` basic configuration).**

```javascript
import { rateLimit } from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  limit: 100,               // 100 requests per window per IP
  standardHeaders: true,    // RateLimit-* headers
  legacyHeaders: false,     // disable X-RateLimit-* headers
  message: 'Too many requests, please try again later.',
});

app.use(limiter);
```

**Component breakdown.**

- `windowMs` — the **time window** in milliseconds. Defaults to `60000` (1 minute).
- `limit` — the **maximum number of requests** allowed per window per key. Renamed from `max` in v7.x.
- `standardHeaders` — when `true`, emits IETF-draft `RateLimit-*` headers.
- `legacyHeaders` — when `false`, disables the older `X-RateLimit-*` headers.
- `message` — the response body sent when the limit is exceeded.

**Rate limiting rules.**

| Rule | Guidance |
|---|---|
| **Choose a window that matches traffic patterns** | 15 minutes is common; shorter windows for sensitive endpoints (login), longer for expensive operations. |
| **Set a limit that protects capacity without blocking legitimate use** | Monitor false positives and adjust. |
| **Use `standardHeaders: true`** | Emit IETF-draft `RateLimit-*` headers for client self-regulation. |
| **Return `429` with `Retry-After`** | Tell clients when they can retry. |
| **Apply limits per key, not globally** | Use IP, API key, or user ID so one abuser does not affect everyone. |

**Constraints and limitations.**

- **The default in-memory store does not synchronize across processes or servers**; use an external store for multi-instance deployments.
- **Setting `max`/`limit` to zero blocks all requests** (since v7.0.0), rather than disabling the limiter.
- **Behind a proxy or load balancer**, the client IP may be the proxy’s IP; configure `trust proxy` correctly.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic rate limiting on an Express app

```javascript
// app.js
import express from 'express';
import { rateLimit } from 'express-rate-limit';

const app = express();

// Step 1: Create a limiter — 100 requests per 15 minutes per IP.
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  limit: 100,
  standardHeaders: true,
  legacyHeaders: false,
  message: 'Too many requests, please try again later.',
});

// Step 2: Apply the limiter to all routes.
app.use(limiter);

// Step 3: Define a simple route.
app.get('/api/data', (req, res) => {
  res.json({ message: 'Hello, world!' });
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

**Expected Output (conceptual):**
```
First 100 requests within 15 minutes → 200 OK
101st request → 429 Too Many Requests
Headers: RateLimit-Limit: 100, RateLimit-Remaining: 0, RateLimit-Reset: <seconds>
```

**Why this output occurs.** The limiter tracks requests per IP within the 15-minute window. The first 100 requests pass; the 101st exceeds the limit and receives `429`. The `RateLimit-*` headers tell the client the quota, remaining count, and reset time.

---

#### Example 2: Limiting a specific route (login)

```javascript
// Step 1: Create a stricter limiter for login attempts.
const loginLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  limit: 5,                 // 5 attempts per hour per IP
  standardHeaders: true,
  legacyHeaders: false,
  message: { error: 'Too many login attempts, please try again later.' },
});

// Step 2: Apply only to the login route.
app.post('/auth/login', loginLimiter, (req, res) => {
  // login logic
  res.json({ success: true });
});
```

**Expected Output (conceptual):**
```
First 5 login attempts per hour → processed
6th attempt → 429 Too Many Requests
```

**Why this output occurs.** The `loginLimiter` is applied as route-specific middleware. It uses a stricter limit (5 per hour) to slow brute-force attacks while leaving other routes unaffected.

---

### Real-World Cases

- **Login endpoints**: strict limits (e.g., 5 attempts per 15 minutes) to prevent credential stuffing.
- **Public APIs**: tiered limits (free vs. paid) with different quotas and burst allowances.
- **Password reset endpoints**: limited to prevent email bombing and abuse.

---

## 2. `express-rate-limit`

### Definitions

**Core Definition.** **`express-rate-limit`** is a popular open-source middleware for Express.js that implements rate limiting. It tracks request counts per client key and rejects requests that exceed the configured limit.

**Technical Definition.** `express-rate-limit` (current major version **8.x**) exports a `rateLimit(options)` function that returns Express middleware. It stores hit counts and reset times in a **store** — by default, an in-memory store — and supports **external stores** (Redis, Memcached, PostgreSQL, MongoDB, SQLite) for distributed deployments. The middleware supports configurable `windowMs`, `limit`, `standardHeaders`, `legacyHeaders`, `message`, `keyGenerator`, `skipSuccessfulRequests`, `skipFailedRequests`, and `store` options. It emits `RateLimit-*` headers when `standardHeaders` is enabled and `X-RateLimit-*` headers when `legacyHeaders` is enabled.

**Beginner-Friendly Explanation.** `express-rate-limit` is **the standard tool for adding rate limiting to an Express app**. You install it with npm, configure a few options (how many requests, over what time window), and add it as middleware. It handles the counting, the `429` responses, and the headers for you. If you run multiple servers, you can swap in a Redis-backed store so they all share the same counts.

### Purposes

- **To add rate limiting to Express applications with minimal code** through a single middleware.
- **To provide sensible defaults** (1-minute window, standard headers off by default but easily enabled) so a working limiter requires almost no configuration.
- **To support distributed deployments** through external stores that synchronize hit counts across processes and servers.
- **To emit standards-based headers** (`RateLimit-*`) so clients can self-regulate.

### Syntax Rules and Structure

**General syntax (installation and import).**

```bash
npm install express-rate-limit
```

```javascript
// ESM
import { rateLimit } from 'express-rate-limit';

// CommonJS
const { rateLimit } = require('express-rate-limit');
```

**Core configuration options.**

| Option | Type | Description |
|---|---|---|
| `windowMs` | `number` | Time window in milliseconds (default `60000`). |
| `limit` | `number \| function` | Max requests per window per key (default `5`). |
| `standardHeaders` | `boolean \| 'draft-6' \| 'draft-7' \| 'draft-8'` | Emit `RateLimit-*` headers. |
| `legacyHeaders` | `boolean` | Emit `X-RateLimit-*` headers. |
| `message` | `any` | Response body when limited (default `'Too many requests, please try again later.'`). |
| `store` | `Store` | Custom store for sharing state across nodes. |
| `keyGenerator` | `function` | Custom key from the request (default: IP). |
| `skipSuccessfulRequests` | `boolean` | If `true`, successful requests do not count against the limit. |
| `skipFailedRequests` | `boolean` | If `true`, failed requests do not count against the limit. |

**Time constants.**

```javascript
import { rateLimit, MINUTE, HOUR, DAY, SECOND } from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 3 * HOUR, // 3 hours
  limit: 1000,
});
```

**Constraints and limitations.**

- **The default memory store does not share state** across processes or servers; use an external store for distributed deployments.
- **Behind a proxy**, set `app.set('trust proxy', 1)` or configure `keyGenerator` to use the correct client IP.
- **`limit: 0` blocks all requests** (v7.0.0+), rather than disabling the limiter.

### Annotated Complete Step by Step Code Examples

#### Example 1: Recommended configuration (from official docs)

```javascript
// app.js
import express from 'express';
import { rateLimit } from 'express-rate-limit';

const app = express();

// Step 1: Create the limiter with recommended options.
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  limit: 100,               // 100 requests per IP per window
  standardHeaders: true,    // RateLimit-* headers
  legacyHeaders: false,     // disable X-RateLimit-* headers
  ipv6Subnet: 56,           // group IPv6 addresses by /56
});

// Step 2: Apply to all requests.
app.use(limiter);

app.get('/', (req, res) => res.send('Hello World'));

app.listen(3000, () => console.log('Server running'));
```

**Expected Output (conceptual):**
```
Up to 100 requests per 15 minutes per IP → 200 OK
Excess requests → 429 Too Many Requests with RateLimit-* headers
```

**Why this output occurs.** The configuration follows the official recommended setup: `standardHeaders: true` emits IETF-draft headers, `legacyHeaders: false` avoids the older `X-RateLimit-*` headers, and `ipv6Subnet: 56` groups IPv6 addresses so a single client cannot bypass limits by cycling through addresses.

---

#### Example 2: Redis-backed store for distributed deployments

```javascript
// app.js
import express from 'express';
import { rateLimit } from 'express-rate-limit';
import { RedisStore } from 'rate-limit-redis';
import { createClient } from 'redis';

const app = express();

// Step 1: Connect to Redis.
const redisClient = createClient({ url: 'redis://localhost:6379' });
await redisClient.connect();

// Step 2: Create a limiter with a Redis store.
const limiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  limit: 100,
  standardHeaders: true,
  legacyHeaders: false,
  store: new RedisStore({
    sendCommand: (...args) => redisClient.sendCommand(args),
  }),
});

// Step 3: Apply the limiter.
app.use(limiter);

app.get('/api', (req, res) => res.json({ ok: true }));

app.listen(3000);
```

**Expected Output (conceptual):**
```
Rate limits are shared across all server instances connected to the same Redis.
Requests are counted consistently regardless of which process handles them.
```

**Why this output occurs.** The Redis store synchronizes hit counts across processes and servers. Unlike the in-memory store, which is per-process, Redis provides a shared counter so limits are enforced uniformly in a distributed deployment.

---

#### Example 3: Custom key generator and skip options

```javascript
// Step 1: Custom limiter for authenticated users only.
const userLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  limit: 30,
  standardHeaders: true,
  legacyHeaders: false,
  // Use the API key from the Authorization header as the key.
  keyGenerator: (req) => req.headers['authorization']?.split(' ')[1] || req.ip,
  // Do not count successful requests against the limit.
  skipSuccessfulRequests: true,
});

// Step 2: Apply to a specific route.
app.get('/api/user-data', userLimiter, (req, res) => {
  res.json({ data: 'sensitive' });
});
```

**Expected Output (conceptual):**
```
Failed requests count against the 30-per-minute limit; successful requests do not.
The limit is keyed by API key, not IP.
```

**Why this output occurs.** `keyGenerator` extracts the bearer token from the `Authorization` header, so limits are per API key. `skipSuccessfulRequests: true` means only failed requests (e.g., `4xx`/`5xx` responses) count toward the limit, which is useful for limiting abusive failed attempts without penalizing normal successful usage.

---

### Real-World Cases

- **Express APIs**: the standard middleware for adding rate limiting in Node.js services.
- **Auth endpoints**: combine `express-rate-limit` with a strict limit and `skipSuccessfulRequests` to slow brute-force attacks.
- **Multi-tenant SaaS**: use `keyGenerator` to key limits by tenant or API key, with tiered `limit` values.

---

## 3. Rate Limit Strategies

### Definitions

**Core Definition.** **Rate limit strategies** (also called **algorithms**) are the methods used to track request counts and decide whether a request is allowed. The principal strategies are **fixed window**, **sliding window** (log and counter), **token bucket**, and **leaky bucket**.

**Technical Definition.** Each strategy differs in **accuracy**, **burst tolerance**, **memory usage**, and **reset behavior**. **Fixed window** counts requests in a fixed interval and resets at the boundary; it is simple but allows bursts at boundaries. **Sliding window log** stores a timestamp per request and checks the count in the trailing window; it is exact but memory-intensive. **Sliding window counter** approximates the sliding window using two counters and a weighted average; it offers near-exact accuracy with low memory. **Token bucket** allows controlled bursts by filling a bucket with tokens at a steady rate; each request consumes a token. **Leaky bucket** smooths traffic by draining requests at a constant rate, either dropping excess (policing) or queuing it (shaping).

**Beginner-Friendly Explanation.** Different strategies are like different **traffic control systems**. **Fixed window** is like a gate that opens and lets 100 cars through every minute, then resets — simple, but 200 cars can sneak through in a few seconds around the boundary. **Sliding window** keeps a rolling count so it is fairer. **Token bucket** gives you a bucket of tokens that refills at a steady rate — you can spend them in a burst if you have saved them up. **Leaky bucket** makes traffic flow out at a constant rate, like a bucket with a hole in the bottom, smoothing out bursts.

### Purposes

- **To choose the right trade-off** between accuracy, memory, and burst tolerance for a given API.
- **To allow controlled bursts** (token bucket) for clients that occasionally send spikes.
- **To smooth traffic** (leaky bucket) for APIs that cannot handle bursts.
- **To enforce simple quotas** (fixed window) where “N per minute” is easy to reason about.

### Syntax Rules and Structure

**Strategy comparison.**

| Strategy | Burst Handling | Memory Usage | Accuracy | Best For |
|---|---|---|---|---|
| **Fixed window** | Allows 2× burst at boundaries | Low (1 key) | Approximate | Simple API limits, login throttling |
| **Sliding window log** | No bursts | High (O(n) entries) | Exact | High-value APIs, audit trails |
| **Sliding window counter** | Smoothed boundaries | Low (2 keys) | Near-exact | General-purpose API rate limiting |
| **Token bucket** | Allows controlled bursts | Low (1 key, 2 fields) | Exact | Bursty traffic with average rate caps |
| **Leaky bucket** | No bursts (steady drain) | Low (1 key, 1–2 fields) | Exact | Policing (drop) or shaping (delay) |

**Choosing a strategy.**

```
Need to allow controlled bursts?
  ├── Yes → Token bucket
  └── No  → Need to smooth traffic?
            ├── Yes → Leaky bucket
            └── No  → Need exact accuracy?
                      ├── Yes → Sliding window log (high memory)
                      └── No  → Sliding window counter (best general balance)
```

**Constraints and limitations.**

- **Fixed window allows up to 2× the limit at boundaries** (e.g., 100 at the end of one minute and 100 at the start of the next).
- **Sliding window log is memory-intensive** because it stores every request timestamp.
- **Token bucket requires choosing both a refill rate and a burst size**; misconfiguration can allow too much burst.
- **Leaky bucket introduces latency** if requests are queued (shaping) rather than dropped (policing).

### Annotated Complete Step by Step Code Examples

#### Example 1: Token bucket logic (simplified, in-process)

```javascript
// token-bucket.js
class TokenBucket {
  constructor(capacity, refillRatePerSec) {
    this.capacity = capacity;           // max tokens
    this.tokens = capacity;             // current tokens
    this.refillRate = refillRatePerSec; // tokens per second
    this.lastRefill = Date.now();
  }

  tryConsume() {
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    this.tokens = Math.min(this.capacity, this.tokens + elapsed * this.refillRate);
    this.lastRefill = now;

    if (this.tokens >= 1) {
      this.tokens -= 1;
      return true;   // allow
    }
    return false;    // reject
  }
}

// Step 1: Create a bucket with capacity 10, refilling 2 tokens/sec.
const bucket = new TokenBucket(10, 2);

// Step 2: Simulate requests.
console.log(bucket.tryConsume()); // true (9 tokens left)
console.log(bucket.tryConsume()); // true (8 tokens left)
```

**Expected Output:**
```
true
true
```

**Why this output occurs.** The bucket starts with 10 tokens. Each `tryConsume` removes one token if available. The refill adds tokens at 2 per second, allowing a burst of up to 10 requests and a sustained rate of 2 requests per second. This is the **token bucket** strategy: controlled bursts with an average rate cap.

---

#### Example 2: Sliding window counter approximation

```javascript
// sliding-window-counter.js
function createSlidingWindow(limit, windowMs) {
  let currentCount = 0;
  let previousCount = 0;
  let windowStart = Date.now();

  return function allow() {
    const now = Date.now();
    const elapsed = now - windowStart;

    if (elapsed >= windowMs) {
      previousCount = currentCount;
      currentCount = 0;
      windowStart = now;
    }

    const weight = 1 - elapsed / windowMs;
    const estimated = previousCount * weight + currentCount;

    if (estimated < limit) {
      currentCount++;
      return true;
    }
    return false;
  };
}

// Step 1: 10 requests per 1 second.
const allow = createSlidingWindow(10, 1000);

// Step 2: Simulate.
for (let i = 0; i < 12; i++) {
  console.log(allow());
}
```

**Expected Output (conceptual):**
```
First 10 calls → true
11th and 12th → false (estimated count exceeds limit)
```

**Why this output occurs.** The sliding window counter uses the previous window’s count weighted by the elapsed fraction plus the current window’s count. This approximates a true sliding window with only two counters, smoothing boundary bursts while keeping memory low.

---

### Real-World Cases

- **Token bucket**: used by APIs that want to allow occasional bursts (e.g., a client syncing data in batches) while enforcing an average rate.
- **Sliding window counter**: the default choice for general-purpose API rate limiting, offering the best balance of accuracy and memory.
- **Leaky bucket**: used for traffic shaping where a steady output rate is required, such as protecting a downstream service that cannot handle bursts.

---

## 4. Rate Limit Headers

### Definitions

**Core Definition.** **Rate limit headers** are HTTP response headers that communicate the server’s quota policy and the client’s current quota state, allowing clients to self-regulate before being throttled.

**Technical Definition.** The IETF draft **“RateLimit header fields for HTTP”** defines `RateLimit-Limit` (the quota in the current window), `RateLimit-Remaining` (the remaining quota), and `RateLimit-Reset` (the seconds until the window resets). A newer draft introduces structured fields such as `RateLimit` and `RateLimit-Policy`. The **legacy de facto** headers are `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset`, which are widely deployed but non-standard. When both `Retry-After` and `RateLimit` headers are present, **`Retry-After` takes precedence** and the reset parameter may be ignored.

**Beginner-Friendly Explanation.** Rate limit headers are **the server telling you how much quota you have left**. Instead of just getting blocked with a `429`, you can read headers like `RateLimit-Remaining: 42` and `RateLimit-Reset: 30` to know you have 42 requests left in the next 30 seconds. Well-behaved clients use these headers to slow down before hitting the limit, rather than being surprised by a `429`.

### Purposes

- **To let clients self-regulate** by exposing the remaining quota and reset time before the limit is reached.
- **To standardize header names and semantics** across APIs, reducing the need for per-API documentation.
- **To improve resilience** of HTTP infrastructures by simplifying enforcement and adoption of rate-limit headers.
- **To support retry logic** alongside `Retry-After`.

### Syntax Rules and Structure

**General syntax (standard IETF-draft headers).**

```http
HTTP/1.1 200 OK
RateLimit-Limit: 100
RateLimit-Remaining: 42
RateLimit-Reset: 30
```

**General syntax (legacy headers).**

```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 42
X-RateLimit-Reset: 1719500000
```

**Component breakdown.**

- `RateLimit-Limit` — the **total quota** in the current window.
- `RateLimit-Remaining` — the **remaining requests** before the limit is reached.
- `RateLimit-Reset` — the **seconds until the window resets** (or a Unix timestamp in some implementations).
- `X-RateLimit-*` — the **legacy** equivalents, still widely used but non-standard.
- `Retry-After` — the authoritative wait time when the limit is exceeded; **takes precedence over `RateLimit-Reset`** if both are present.

**Header rules.**

| Rule | Guidance |
|---|---|
| **Prefer `RateLimit-*` over `X-RateLimit-*`** | The IETF draft is the standardization track; `X-` headers are legacy. |
| **Emit headers on all responses, not just `429`** | Clients need the remaining quota before they are blocked. |
| **Include `Retry-After` on `429`** | It is the authoritative signal; `RateLimit-Reset` is secondary. |
| **Do not expose sensitive quota details** | Servers should cap the ratio between remaining and reset values to avoid leaking volume information. |

**Constraints and limitations.**

- **Header names and semantics are still being standardized**; implementations vary in whether `RateLimit-Reset` is seconds or a timestamp.
- **Clients must tolerate missing headers**; servers are not required to return them on every response.
- **Malformed headers must be ignored** by clients.

### Annotated Complete Step by Step Code Examples

#### Example 1: Emitting standard headers with `express-rate-limit`

```javascript
// Step 1: Configure the limiter to emit standard headers.
const limiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  limit: 100,
  standardHeaders: true,  // emits RateLimit-* headers
  legacyHeaders: false,   // disables X-RateLimit-* headers
});

app.use(limiter);

app.get('/api', (req, res) => res.json({ ok: true }));
```

**Expected Response (conceptual):**
```http
HTTP/1.1 200 OK
RateLimit-Limit: 100
RateLimit-Remaining: 99
RateLimit-Reset: 60
```

**Why this output occurs.** `standardHeaders: true` instructs `express-rate-limit` to emit the IETF-draft `RateLimit-*` headers. The `Remaining` value decrements with each request, and `Reset` shows the seconds until the window expires.

---

#### Example 2: Handling `429` with `Retry-After`

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 30
RateLimit-Limit: 100
RateLimit-Remaining: 0
RateLimit-Reset: 30
Content-Type: application/json

{
  "error": "Too many requests",
  "message": "Please retry after 30 seconds."
}
```

**Expected Output (conceptual):**
```
The client receives 429 with Retry-After: 30.
It should wait 30 seconds before retrying, then resume within the new window.
```

**Why this output occurs.** `Retry-After` is the authoritative signal for when to retry. The `RateLimit-*` headers provide additional context. When both are present, the client should honor `Retry-After` first.

---

### Real-World Cases

- **GitHub API**: returns `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset` on every response.
- **IETF-draft adopters**: APIs that emit `RateLimit-*` headers to align with the standardization track.
- **`express-rate-limit`**: supports both standard and legacy headers via `standardHeaders` and `legacyHeaders` options.

---

## 5. Rate Limiting Best Practices

### Definitions

**Core Definition.** **Rate limiting best practices** are empirically established guidelines for implementing, configuring, and communicating rate limits in a way that protects the API, treats clients fairly, and enables good client behavior.

**Technical Definition.** Best practices span **algorithm selection** (sliding window counter for general use, token bucket for bursts, leaky bucket for shaping), **limit design** (per-key limits, tiered quotas, separate limits for sensitive endpoints), **header communication** (standard `RateLimit-*` headers, `Retry-After` on `429`), **client guidance** (exponential backoff with jitter, respect `Retry-After`), **store selection** (external store for distributed deployments), **proxy awareness** (`trust proxy` configuration), and **monitoring** (logging, false-positive tracking).

**Beginner-Friendly Explanation.** Best practices are **the habits that make rate limiting work well**. Pick the right algorithm for your traffic. Set limits that protect your servers without blocking legitimate users. Tell clients how much quota they have left. Use an external store if you run multiple servers. Configure `trust proxy` correctly if you are behind a load balancer. Make sure clients back off politely when they get a `429`. And watch your logs to know if your limits are too tight or too loose.

### Purposes

- **To protect the API without harming legitimate users** by choosing appropriate limits and algorithms.
- **To enable distributed enforcement** through external stores.
- **To guide clients toward good behavior** with clear headers and retry semantics.
- **To avoid common pitfalls** such as proxy IP confusion and in-memory store inconsistency.

### Syntax Rules and Structure

**Best-practice checklist.**

| Area | Best Practice |
|---|---|
| **Algorithm** | Use sliding window counter for general APIs; token bucket for burst tolerance; leaky bucket for smoothing. |
| **Keying** | Key by API key or user ID for authenticated APIs; by IP for anonymous traffic; use `ipv6Subnet` to group IPv6 addresses. |
| **Tiered limits** | Different quotas for free vs. paid plans; stricter limits for sensitive endpoints (login, password reset). |
| **Headers** | Emit `RateLimit-*` headers; include `Retry-After` on `429`. |
| **Client behavior** | Respect `Retry-After`; implement exponential backoff with jitter; do not retry immediately. |
| **Store** | Use an external store (Redis, Memcached, PostgreSQL) for multi-process or multi-server deployments. |
| **Proxy** | Configure `trust proxy` so the client IP is correctly identified. |
| **Monitoring** | Log rate-limit events; track false positives and adjust limits. |

**Constraints and limitations.**

- **No single algorithm is best for all cases**; the choice depends on traffic patterns and burst tolerance.
- **In-memory stores are per-process** and reset on restart; they are insufficient for distributed deployments.
- **Rate limits are not a substitute for authentication or authorization**; they complement security controls, not replace them.

### Annotated Complete Step by Step Code Examples

#### Example 1: Tiered limits by API key (paid vs. free)

```javascript
// Step 1: Define a function that returns a limit based on the API key tier.
async function getLimitForApiKey(apiKey) {
  const tier = await lookupTier(apiKey); // 'free' or 'paid'
  return tier === 'paid' ? 1000 : 100;
}

// Step 2: Create a limiter with a function-based limit.
const limiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  limit: async (req) => {
    const apiKey = req.headers['x-api-key'];
    return getLimitForApiKey(apiKey);
  },
  keyGenerator: (req) => req.headers['x-api-key'] || req.ip,
  standardHeaders: true,
  legacyHeaders: false,
});

app.use(limiter);
```

**Expected Output (conceptual):**
```
Free-tier API keys → 100 requests per minute
Paid-tier API keys → 1000 requests per minute
```

**Why this output occurs.** The `limit` option accepts an async function that receives the request and returns the limit. By looking up the API key’s tier, the limiter applies different quotas to different clients — a common pattern for SaaS APIs.

---

#### Example 2: Correct proxy configuration

```javascript
// If the app is behind a proxy (e.g., nginx, load balancer), set trust proxy.
app.set('trust proxy', 1); // trust the first proxy

// Then create the limiter normally.
const limiter = rateLimit({
  windowMs: 60 * 1000,
  limit: 100,
  standardHeaders: true,
  legacyHeaders: false,
});

app.use(limiter);
```

**Expected Output (conceptual):**
```
The limiter uses the client IP from X-Forwarded-For, not the proxy's IP.
Limits are enforced per real client, not per proxy.
```

**Why this output occurs.** Without `trust proxy`, Express uses the proxy’s IP as `req.ip`, and the limiter would treat all requests as coming from one client. Setting `trust proxy` to the number of trusted proxies tells Express to read the client IP from `X-Forwarded-For`, enabling correct per-client limiting.

---

### Real-World Cases

- **GitHub API**: tiered limits for authenticated vs. unauthenticated requests, with `X-RateLimit-*` headers on every response.
- **Stripe API**: returns `429` with `Retry-After` and rate-limit headers; clients are expected to implement exponential backoff.
- **Multi-tenant SaaS**: tiered limits by plan, with Redis-backed stores for distributed enforcement.

---

## References

- express-rate-limit — Configuration - https://express-rate-limit.mintlify.app/reference/configuration
- express-rate-limit — Usage Quickstart - https://express-rate-limit.mintlify.app/quickstart/usage
- express-rate-limit — Data Stores - https://express-rate-limit.mintlify.app/reference/stores
- express-rate-limit — Creating a Store - https://express-rate-limit.mintlify.app/guides/creating-a-store
- express-rate-limit — npm - https://www.npmjs.com/package/express-rate-limit
- IETF — RateLimit Header Fields for HTTP (draft-polli-ratelimit-headers-03) - https://www.ietf.org/archive/id/draft-polli-ratelimit-headers-03.xml
- IETF — RateLimit Header Fields for HTTP (draft-ietf-httpapi-ratelimit-headers-11) - https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/
- IETF — RateLimit and Retry-After precedence (httpapi meeting agenda) - https://datatracker.ietf.org/meeting/114/agenda/httpapi-drafts.pdf
- API Rate Limit Best Practices That Actually Work — Zernio - https://zernio.com/blog/api-rate-limit-best-practices
- Build 5 Rate Limiters with Redis — Redis Tutorials - https://redis.io/tutorials/howtos/ratelimiting/
- API Rate Limiting Mechanisms in SaaS Applications — Zenodo - https://zenodo.org/records/21606018
- Rate limiting strategies (token bucket, leaky bucket, sliding window) — pkg.go.dev - https://pkg.go.dev/github.com/strigo/ratelimit
- API Rate Limiting & Throttling: Algorithms, Headers, Backoff (2026) — Back4App - https://www.back4app.com/blog/api-rate-limiting-throttling