# Concurrency Management — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Concurrency management is the discipline of orchestrating multiple asynchronous operations within a Node.js application to maximise throughput, minimise latency, and prevent resource exhaustion — deciding when to run tasks sequentially, when to run them in parallel, and how to bound the number of operations in flight.

**Technical Definition:** Node.js executes JavaScript on a single thread, using an event loop with non-blocking I/O via libuv to handle concurrency without spawning a thread per request. Concurrency management leverages this model by structuring asynchronous operations to overlap I/O waits rather than blocking the event loop. The critical decisions are: whether operations have dependencies (requiring sequential execution), whether they are independent (permitting parallel execution via `Promise.all` or `Promise.allSettled`), and whether the number of concurrent operations must be bounded to protect database connection pools, external API rate limits, or local resources (using concurrency limiters like `p-limit`).

**Beginner-Friendly Explanation:** Imagine a restaurant kitchen with one chef (Node.js's single thread). The chef doesn't stand and wait for the oven to preheat — they put a dish in the oven (start a database query), and while it bakes, they chop vegetables (handle another request). This is how Node.js achieves concurrency. But the chef can't cook 50 dishes at once — the oven has limited space and the counter has limited room. Concurrency management is the chef's system for deciding which dishes to cook together, which must wait, and how many to start at a time so the kitchen doesn't collapse.

### Key Characteristics

- **Single-threaded event loop:** Node.js handles I/O concurrency natively without thread management, but CPU-bound work must be offloaded.
- **Dependency-driven sequencing:** Operations with data dependencies must run sequentially; independent operations should run in parallel.
- **Fail-fast vs. fault-tolerant:** `Promise.all` fails fast on the first rejection; `Promise.allSettled` waits for all outcomes.
- **Resource-bounded concurrency:** Unbounded parallelism exhausts file descriptors, memory, and database connections — concurrency limiters bound the fan-out.
- **Race condition awareness:** Concurrent access to shared mutable state (database rows, files) requires locks or atomic operations.
- **Observability:** Concurrency metrics (active count, pending count) provide visibility into system pressure.

### Prerequisites

- **Node.js runtime** (v18 or higher; `Promise.allSettled` is available since v12.9.0).
- **Express.js installed:** `npm install express`.
- **A concurrency limiting library:** `npm install p-limit` (optional but recommended).
- **Basic JavaScript knowledge:** Promises, `async/await`, and the event loop.

### Related Programming Areas

- **Async/Await & Promise Integration:** The foundation for all concurrency patterns.
- **Async Middleware & Network Operations:** Database pools and API clients are the primary resources being concurrency-managed.
- **Database Operations:** Connection pools bound database concurrency; transactions prevent race conditions.
- **External API Calls:** Rate limits and circuit breakers interact with concurrency limits.
- **Background Processing:** BullMQ workers have their own concurrency controls.

### Core Concepts

1. **Sequential vs. Parallel Async Operations** — avoiding the async/await waterfall anti-pattern.
2. **Promise.all()** — aggregating independent, concurrent tasks with fail-fast behaviour.
3. **Promise.allSettled()** — handling batch executions where individual failures should not crash the operation.
4. **Race Conditions** — mitigating database overwrites and out-of-order execution with locks.
5. **Resource Contention & Throttling** — preventing exhaustion with concurrency pools and semaphores.

---

## Core Concept 1: Sequential vs. Parallel Async Operations

### Definitions

**Core Definition:** Sequential execution runs asynchronous operations one after another, each waiting for the previous to complete; parallel execution starts multiple independent operations simultaneously and waits for all to finish.

**Technical Definition:** In an `async` function, each `await` pauses execution until the awaited Promise settles. When independent operations are awaited sequentially, the total time is the **sum** of their individual durations — this is the "async/await waterfall" anti-pattern. When the same operations are started concurrently (via `Promise.all` or by collecting Promises and awaiting them together), the total time is the **maximum** of their individual durations, because the event loop overlaps the I/O waits.

**Beginner-Friendly Explanation:** Imagine ordering three coffees at a café. **Sequential** is like ordering one coffee, waiting for it to be made, then ordering the second, waiting again, then the third — you wait three times. **Parallel** is like ordering all three at once and waiting for the barista to make them together — you wait once. The coffee-making time is the same, but the total waiting time is dramatically shorter.

### Purposes

- To maximise performance by avoiding the "async/await waterfall" anti-pattern.
- To reduce end-to-end latency when operations are independent.
- To ensure correctness when operations have data dependencies (sequential).
- To free the event loop to handle other requests while I/O waits overlap.

### Syntax Rules and Structure

#### Sequential (Waterfall) — Correct Only When Dependent

```javascript
async function getPostData(slug) {
  const post = await fetchPost(slug);        // Must complete first
  const author = await fetchAuthor(post.authorId); // Depends on post
  const comments = await fetchComments(post.id);   // Depends on post
  return { post, author, comments };
}
```

#### Parallel — Correct for Independent Operations

```javascript
async function getPostData(slug) {
  const post = await fetchPost(slug);
  // author and comments are independent of each other
  const [author, comments] = await Promise.all([
    fetchAuthor(post.authorId),
    fetchComments(post.id)
  ]);
  return { post, author, comments };
}
```

| Scenario | Strategy | Time Complexity |
|----------|----------|----------------|
| Operations have dependencies | Sequential (`await` one by one). | O(sum of durations). |
| Independent, few operations | Parallel (`Promise.all`). | O(max duration). |
| Independent, many operations | Bounded parallel (`p-limit`). | O(max duration / concurrency). |

**Rules:**
- Only use sequential `await` when the next operation **depends** on the previous result.
- Use `Promise.all` when operations are independent and all must succeed.
- Use `Promise.allSettled` when operations are independent and partial failure is acceptable.
- **Never** start all operations without a bound if the list length is user-controlled.

**Constraints:**
- `Promise.all` does not cancel already-started operations when one rejects — they continue to completion (or until the process exits).
- `Promise.all` with 10,000 items creates 10,000 concurrent operations, exhausting file descriptors and memory.

### Annotated Code Example

```javascript
// sequential-vs-parallel.js
const wait = (ms) => new Promise(resolve => setTimeout(resolve, ms));

async function fetchUser() {
  await wait(200);
  return { id: 1, name: 'Alice' };
}

async function fetchPosts() {
  await wait(200);
  return [{ id: 1, title: 'Post 1' }];
}

async function fetchStats() {
  await wait(200);
  return { views: 1000 };
}

// ❌ WATERFALL: Sequential awaits for independent operations
async function sequentialFetch() {
  const start = Date.now();
  const user = await fetchUser();
  const posts = await fetchPosts();
  const stats = await fetchStats();
  console.log(`Sequential: ${Date.now() - start}ms`);
  return { user, posts, stats };
}

// ✅ PARALLEL: All independent operations start together
async function parallelFetch() {
  const start = Date.now();
  const [user, posts, stats] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchStats()
  ]);
  console.log(`Parallel: ${Date.now() - start}ms`);
  return { user, posts, stats };
}

(async () => {
  await sequentialFetch(); // ~600ms
  await parallelFetch();   // ~200ms
})();
```

**Expected Output:**
```
Sequential: 603ms
Parallel: 202ms
```

**Why this output:** The sequential version runs three 200ms operations one after another (200 + 200 + 200 = 600ms). The parallel version starts all three simultaneously and waits for the longest one (200ms). The parallel version is approximately 3x faster. Order of results is guaranteed: `user`, `posts`, `stats` correspond to the input array positions.

### Real-World Cases

- **Dashboard aggregation:** Fetching user, orders, and analytics data in parallel.
- **Product page:** Fetching product details, reviews, and inventory concurrently.
- **Data export:** Reading multiple files or database tables in parallel.

---

## Core Concept 2: Promise.all()

### Definitions

**Core Definition:** `Promise.all()` is a static method that takes an iterable of Promises and returns a single Promise that fulfils with an array of results when all input Promises fulfil, or rejects with the first rejection reason if any input Promise rejects.

**Technical Definition:** `Promise.all(iterable)` returns a Promise that resolves to an array of the fulfilled values, in the same order as the input iterable, when every input Promise has fulfilled. If any input Promise rejects, the returned Promise immediately rejects with that reason — this is **fail-fast** behaviour. Already-started Promises are not cancelled and continue to completion. `Promise.all` is the correct tool when independent operations must all succeed for the result to be meaningful.

**Beginner-Friendly Explanation:** `Promise.all` is like ordering a set meal where all courses must arrive. If the appetiser, main, and dessert are all ready, you get the full meal. If the kitchen runs out of dessert, the whole order is cancelled — you don't get the appetiser and main without dessert.

### Purposes

- To aggregate independent, concurrent async tasks with fail-fast behaviour.
- To maximise performance by starting all operations simultaneously.
- To ensure all-or-nothing semantics for a batch of operations.
- To maintain result order that matches the input array.

### Syntax Rules and Structure

```javascript
const [result1, result2, result3] = await Promise.all([
  operation1(),
  operation2(),
  operation3()
]);
```

| Component | Breakdown |
|-----------|-----------|
| `Promise.all(iterable)` | Takes an iterable of Promises. |
| Returns | A Promise resolving to an array of values. |
| Order | Results match input order, regardless of completion order. |
| Failure | Rejects immediately on the first rejection. |

**Rules:**
- Use `Promise.all` when all operations must succeed.
- Use `Promise.all` when the number of operations is small and bounded.
- The result array preserves input order — destructuring is safe.
- Already-started operations continue on rejection — the event loop still processes them.

**Constraints:**
- **Fail-fast:** One rejection discards all other results.
- **No concurrency limit:** All operations start immediately.
- **No cancellation:** Rejected operations do not cancel siblings.

### Annotated Code Example

```javascript
// promise-all.js
const express = require('express');
const app = express();

async function fetchUser(id) {
  return { id, name: 'Alice', email: 'alice@example.com' };
}

async function fetchOrders(userId) {
  return [{ id: 1, total: 99.99 }, { id: 2, total: 49.99 }];
}

async function fetchCart(userId) {
  return { items: 3, total: 149.97 };
}

app.get('/api/dashboard/:userId', async (req, res) => {
  const { userId } = req.params;

  // All three operations are independent — run in parallel
  const [user, orders, cart] = await Promise.all([
    fetchUser(userId),
    fetchOrders(userId),
    fetchCart(userId)
  ]);

  res.json({ data: { user, orders, cart } });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/dashboard/1`):**
```json
{
  "data": {
    "user": { "id": "1", "name": "Alice", "email": "alice@example.com" },
    "orders": [{ "id": 1, "total": 99.99 }, { "id": 2, "total": 49.99 }],
    "cart": { "items": 3, "total": 149.97 }
  }
}
```

**Why this output:** The three fetch functions are independent. `Promise.all` starts them concurrently and waits for all three to resolve. The results are destructured in the same order as the input array. Total latency equals the longest individual operation, not the sum.

### Real-World Cases

- **Dashboard aggregation:** Fetching user profile, order history, and cart in parallel.
- **Product detail pages:** Fetching product, reviews, and recommendations concurrently.
- **Bulk inserts:** `Promise.all(users.map(u => db.insert(u)))` for small batches.

---

## Core Concept 3: Promise.allSettled()

### Definitions

**Core Definition:** `Promise.allSettled()` is a static method that takes an iterable of Promises and returns a single Promise that fulfils with an array of outcome objects — each describing whether the corresponding Promise fulfilled or rejected — after all input Promises have settled.

**Technical Definition:** `Promise.allSettled(iterable)` returns a Promise that resolves to an array of objects. Each object has a `status` property (`"fulfilled"` or `"rejected"`). Fulfilled objects have a `value` property; rejected objects have a `reason` property. Unlike `Promise.all`, `allSettled` never rejects — it waits for every Promise to settle and reports the outcome of each. This makes it the correct tool when individual failures should not cancel the entire batch.

**Beginner-Friendly Explanation:** `Promise.allSettled` is like ordering a buffet where you want to know how every dish turned out. Even if the soup is cold and the salad is wilted, you still get the bread, the main course, and the dessert — and a full report on each dish. One bad dish doesn't cancel the whole meal.

### Purposes

- To handle batch async executions where individual task failures should not crash the entire operation.
- To collect both successful and failed results for later processing.
- To provide fault tolerance when processing large batches of independent records.
- To avoid the unwrapping complexity of `Promise.all` when partial success is acceptable.

### Syntax Rules and Structure

```javascript
const results = await Promise.allSettled([
  operation1(),
  operation2(),
  operation3()
]);

results.forEach((result, index) => {
  if (result.status === 'fulfilled') {
    console.log(`Operation ${index}:`, result.value);
  } else {
    console.log(`Operation ${index} failed:`, result.reason);
  }
});
```

| Property | Description |
|----------|-------------|
| `status` | `"fulfilled"` or `"rejected"`. |
| `value` | Present only on fulfilled results. |
| `reason` | Present only on rejected results. |

**Rules:**
- Use `Promise.allSettled` when partial success is acceptable.
- Check `result.status` before accessing `result.value` or `result.reason`.
- Results are in input order, regardless of settlement order.
- `allSettled` never rejects — errors are in the results, not in the Promise.

**Constraints:**
- Adds unwrapping overhead compared to `Promise.all` when all operations are expected to succeed.
- Does not provide a fail-fast mechanism — all operations run to completion.

### Annotated Code Example

```javascript
// promise-allsettled.js
const express = require('express');
const app = express();

// Simulated external API that sometimes fails
async function fetchPrice(productId) {
  if (productId === 'fail') {
    throw new Error(`Product ${productId} not found`);
  }
  return { productId, price: 99.99 };
}

app.get('/api/prices', async (req, res) => {
  const productIds = ['a1', 'fail', 'b2', 'c3'];

  const results = await Promise.allSettled(
    productIds.map(id => fetchPrice(id))
  );

  const successful = [];
  const failed = [];

  for (const [index, result] of results.entries()) {
    if (result.status === 'fulfilled') {
      successful.push(result.value);
    } else {
      failed.push({
        productId: productIds[index],
        error: result.reason.message
      });
    }
  }

  res.json({ successful, failed });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/prices`):**
```json
{
  "successful": [
    { "productId": "a1", "price": 99.99 },
    { "productId": "b2", "price": 99.99 },
    { "productId": "c3", "price": 99.99 }
  ],
  "failed": [
    { "productId": "fail", "error": "Product fail not found" }
  ]
}
```

**Why this output:** One product ID (`'fail'`) causes a rejection. `Promise.allSettled` waits for all four operations to settle and returns an array of outcome objects. The handler separates successful results from failures and returns both — the one failure does not prevent the other three from succeeding.

### Real-World Cases

- **Bulk data import:** Processing thousands of records where some may be invalid.
- **Multi-channel notifications:** Sending email, SMS, and push notifications — partial success is acceptable.
- **Price comparison:** Fetching prices from multiple vendors where some may be unavailable.
- **Batch API calls:** Calling an external API for multiple items where some may not exist.

---

## Core Concept 4: Race Conditions

### Definitions

**Core Definition:** A race condition occurs when the correctness of a system depends on the relative timing or interleaving of concurrent operations — typically when two or more processes read, modify, and write shared state without proper synchronisation.

**Technical Definition:** Race conditions arise when concurrent transactions access shared mutable state (database rows, in-memory caches, files) without coordination. The classic pattern is the **read-modify-write** cycle: two transactions read the same value, both modify it based on the stale read, and the second write overwrites the first (a **lost update**). Mitigation strategies include **optimistic locking** (version columns checked at write time), **pessimistic locking** (row-level locks such as `SELECT ... FOR UPDATE`), **atomic operations** (increment/decrement in the database), and **distributed locks** (Redis-based locks such as Redlock for cross-process coordination).

**Beginner-Friendly Explanation:** A race condition is like two people editing the same Google Doc at the same time. Both open the document (read), both change a sentence (modify), and both save (write). The second person's save overwrites the first person's changes — the first person's edit is lost. Locks are like a "one editor at a time" rule: the second person waits until the first is done before they can edit.

### Purposes

- To mitigate database overwrites or out-of-order execution states.
- To ensure data integrity when multiple processes access shared resources.
- To prevent lost updates in read-modify-write cycles.
- To coordinate work across distributed instances of an application.

### Sub-Feature 4.1: Database-Level Locking

#### Syntax Rules and Structure

```sql
-- Pessimistic lock: lock the row at read time
BEGIN;
SELECT balance FROM accounts WHERE id = $1 FOR UPDATE;
-- Only one transaction can hold this lock at a time
UPDATE accounts SET balance = balance - $2 WHERE id = $1;
COMMIT;
```

| Approach | Mechanism | Use Case |
|----------|-----------|----------|
| **Pessimistic** | `SELECT ... FOR UPDATE` | High conflict; blocks concurrent writers. |
| **Optimistic** | Version column checked at write. | Low conflict; retries on failure. |
| **Atomic** | `UPDATE ... SET balance = balance - 10`. | Simple increments/decrements. |

**Rules:**
- Use `SELECT ... FOR UPDATE` when a write depends on the value read.
- Use atomic operations (`UPDATE ... SET x = x + 1`) for simple arithmetic.
- Use optimistic locking (version columns) when conflicts are rare.
- Keep transactions short — holding locks blocks other transactions.

#### Annotated Code Example

```javascript
// pessimistic-lock.js
const pool = require('./db');

async function transferFunds(fromId, toId, amount) {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');

    // Lock both rows in a consistent order to prevent deadlock
    await client.query(
      'SELECT balance FROM accounts WHERE id = $1 FOR UPDATE', [fromId]
    );
    await client.query(
      'SELECT balance FROM accounts WHERE id = $1 FOR UPDATE', [toId]
    );

    await client.query(
      'UPDATE accounts SET balance = balance - $1 WHERE id = $2', [amount, fromId]
    );
    await client.query(
      'UPDATE accounts SET balance = balance + $1 WHERE id = $2', [amount, toId]
    );

    await client.query('COMMIT');
  } catch (err) {
    await client.query('ROLLBACK');
    throw err;
  } finally {
    client.release();
  }
}
```

**Expected Output (for a valid transfer):**
```json
{ "success": true, "message": "Transfer completed" }
```

**Why this output:** The `FOR UPDATE` clause locks both account rows at read time. A concurrent transfer for the same accounts blocks until the first transaction commits. The second transaction then reads the updated balances, preventing the lost update.

---

### Sub-Feature 4.2: Distributed Locks (Redis Redlock)

#### Syntax Rules and Structure

```javascript
const { default: Redlock } = require('redlock');
const Redis = require('ioredis');

const redis = new Redis({ host: 'localhost', port: 6379 });
const redlock = new Redlock([redis], {
  retryCount: 3,
  retryDelay: 200,
  retryJitter: 100
});

async function processPayment(paymentId) {
  const lock = await redlock.acquire([`locks:payment:${paymentId}`], 30000);
  try {
    // Critical section — only one process can execute this
    await chargePayment(paymentId);
  } finally {
    await lock.release();
  }
}
```

| Component | Breakdown |
|-----------|-----------|
| `Redlock([redis])` | Creates a Redlock instance with Redis clients. |
| `acquire(resources, ttl)` | Acquires locks on the specified resources with a TTL. |
| `lock.release()` | Releases the lock (must be called in `finally`). |

**Rules:**
- Use distributed locks when multiple processes or machines access shared resources.
- Set a TTL on locks to prevent deadlock if the lock holder crashes.
- Always release the lock in a `finally` block.
- Use the Redlock algorithm (multiple Redis nodes) for high-availability safety.

**Constraints:**
- Redlock requires N independent Redis instances for the Redlock algorithm.
- Locks add latency — use only when necessary.
- A single Redis node with `SET NX EX` is simpler but less fault-tolerant.

### Real-World Cases

- **E-commerce:** Preventing double-booking of a seat or product during checkout.
- **Payment processing:** Ensuring a payment is charged exactly once.
- **Inventory management:** Preventing overselling of limited-stock items.
- **Scheduled jobs:** Ensuring a cron job runs on only one instance in a cluster.

---

## Core Concept 5: Resource Contention & Throttling

### Definitions

**Core Definition:** Resource contention occurs when multiple concurrent operations compete for a limited resource (database connections, network sockets, CPU, file descriptors), and throttling is the practice of bounding the number of concurrent operations to prevent exhaustion of those resources.

**Technical Definition:** Unbounded concurrency — such as `Promise.all` over an array of 10,000 URLs — creates 10,000 concurrent operations in the same tick. This exhausts file descriptors, balloons memory with in-flight buffers, triggers upstream API rate limiting, and starves the event loop. **Concurrency limiters** (such as the `p-limit` package) cap the number of operations in flight at any instant, queuing the remainder and releasing them as running operations complete. This keeps offered load near the resource's sustainable capacity instead of spiking far above it.

**Beginner-Friendly Explanation:** Imagine a checkout counter with 10 cashiers. If 1,000 customers all rush the counter at once, chaos ensues — the cashiers are overwhelmed, the queue spills into the street, and nobody gets served efficiently. A concurrency limiter is a queue management system: only 10 customers are at the counter at any time, and the rest wait in an orderly line. The cashiers work at a steady pace, and the queue drains predictably.

### Purposes

- To prevent database connection exhaustion or external API rate-limiting.
- To utilise concurrency pools, batching, and semaphore limits.
- To maintain system stability under heavy load.
- To provide observability into system pressure (active and pending counts).

### Sub-Feature 5.1: p-limit (Promise Concurrency Limiter)

#### Syntax Rules and Structure

```javascript
import pLimit from 'p-limit';

const limit = pLimit(8); // At most 8 concurrent operations

const results = await Promise.all(
  urls.map(url => limit(() => fetch(url)))
);
```

| Component | Breakdown |
|-----------|-----------|
| `pLimit(n)` | Creates a limiter with maximum concurrency `n`. |
| `limit(() => task())` | Wraps a task in the limiter; **pass a function**, not a Promise. |
| `limit.activeCount` | Number of tasks currently running. |
| `limit.pendingCount` | Number of tasks queued. |

**Rules:**
- **Always pass a function** to the limiter — `limit(fetch(url))` starts the request before the limiter sees it.
- Set concurrency based on the resource: 4–10 for outbound HTTP, CPU count for filesystem/CPU work, connection pool size for database writes.
- Make the limit a configuration value, not a hard-coded literal.
- Use `limit.activeCount` and `limit.pendingCount` as monitoring gauge metrics.

**Constraints:**
- `p-limit` only limits concurrency — it does not handle retries, timeouts, or backoff.
- The limiter’s internal queue holds references to pending tasks, which can accumulate memory.

#### Annotated Code Example

```javascript
// p-limit-example.js
const express = require('express');
const pLimit = require('p-limit');
const app = express();

// Simulated external API with rate limit
async function fetchExternalData(id) {
  await new Promise(r => setTimeout(r, 100)); // Simulate latency
  return { id, data: `data-${id}` };
}

app.get('/api/batch', async (req, res) => {
  const ids = Array.from({ length: 100 }, (_, i) => i + 1);

  // Limit to 10 concurrent requests
  const limit = pLimit(10);

  const start = Date.now();
  const results = await Promise.all(
    ids.map(id => limit(() => fetchExternalData(id)))
  );
  const duration = Date.now() - start;

  res.json({
    count: results.length,
    duration: `${duration}ms`,
    maxConcurrency: limit.activeCount, // Peak concurrent count
    sample: results.slice(0, 3)
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/batch`):**
```json
{
  "count": 100,
  "duration": "1003ms",
  "maxConcurrency": 10,
  "sample": [
    { "id": 1, "data": "data-1" },
    { "id": 2, "data": "data-2" },
    { "id": 3, "data": "data-3" }
  ]
}
```

**Why this output:** With 100 tasks at 100ms each and a concurrency limit of 10, the total time is approximately `(100 / 10) × 100ms = 1000ms`. The limiter ensures that at most 10 tasks run simultaneously, preventing the external API from being overwhelmed.

### Sub-Feature 5.2: Database Connection Pool as a Natural Limiter

#### Syntax Rules and Structure

```javascript
const { Pool } = require('pg');

const pool = new Pool({
  max: 20,                        // Maximum 20 concurrent connections
  connectionTimeoutMillis: 5000   // Wait 5s for a connection
});
```

| Pool Parameter | Concurrency Implication |
|----------------|------------------------|
| `max` | Maximum concurrent queries. |
| `connectionTimeoutMillis` | How long a query waits for a connection. |

**Rules:**
- The database connection pool is a natural concurrency limiter — queries beyond `max` queue.
- Set `max` based on the database server's `max_connections` and the number of Node.js processes.
- Never hold a connection open for external API calls — this exhausts the pool.

### Real-World Cases

- **Web scraping:** Limiting concurrent requests to avoid being blocked by the target site.
- **Bulk database writes:** Limiting concurrent inserts to match the connection pool size.
- **Image processing:** Limiting concurrent CPU-bound operations to the number of available cores.
- **External API integration:** Respecting the API provider's documented rate limits.

---

## References

- Vercel Academy — Data Fetching Without Waterfalls — https://vercel.com/academy/nextjs-foundations/data-fetching-without-waterfalls 
- Vercel Academy — Query Performance Patterns — https://vercel.com/academy/nextjs-foundations/query-performance-patterns 
- Soleur Knowledge Base — Promise.all Parallel Filesystem I/O Patterns in Node.js — https://raw.githubusercontent.com/jikig-ai/soleur/923179f62b97ff018c467f34db329b250fdb4082/knowledge-base/project/learnings/2026-04-07-promise-all-parallel-fs-io-patterns.md 
- OneUptime — How to Design a Distributed Lock Using Redis — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-redis-how-to-design-a-distributed-lock-using-redi/README.md 
- Pluralsight — Resilient Concurrency and Rate-Limiting for LLM Callbacks — https://www.pluralsight.com/labs/codeLabs/resilient-concurrency-and-rate-limiting-for-llm-callbacks 
- W3CSchool — Node.js 异步任务怎么控并发？串行、并行与限流实战 — https://www.w3cschool.cn/article/66386240.html 
- BullMQ — Concurrency — https://docs.bullmq.io/guide/workers/concurrency 
- Safeguard.sh — p-limit npm: Safe Concurrency Control in Node.js — https://safeguard.sh/resources/blog/p-limit-npm-concurrency-guide 
- MDN — Promise.all() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all
- MDN — Promise.allSettled() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled
- Node.js — `async_hooks` (AsyncLocalStorage) — https://nodejs.org/api/async_context.html
- npm — p-limit — https://www.npmjs.com/package/p-limit
- npm — redlock — https://www.npmjs.com/package/redlock
- PostgreSQL — `SELECT ... FOR UPDATE` — https://www.postgresql.org/docs/current/sql-select.html#SQL-FOR-UPDATE-SHARE
- Redis — Distributed Locks — https://redis.io/docs/manual/patterns/distributed-locks/
- Node.js Best Practices — Do Not Block the Event Loop — https://github.com/goldbergyoni/nodebestpractices