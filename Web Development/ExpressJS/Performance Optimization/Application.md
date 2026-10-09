# Application Performance — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Application performance optimisation in Node.js is the practice of ensuring the application makes efficient use of the single-threaded event loop, CPU cores, memory, and I/O resources — by avoiding blocking operations, choosing efficient algorithms, managing memory correctly, streaming large data, offloading background work, and scaling across CPU cores with clustering and worker threads.

**Technical Definition:** Node.js runs JavaScript on a single-threaded event loop. All I/O (network, file system, DNS) is offloaded to the libuv thread pool and executed asynchronously, but CPU-bound work (loops, encryption, image processing, JSON serialisation of large payloads) executes on the main thread and blocks the event loop, preventing the application from handling other requests. Application performance optimisation encompasses: identifying and eliminating event-loop blocking, optimising algorithmic complexity and data structure selection, preventing memory leaks through heap analysis and careful closure design, streaming large files instead of buffering them, delegating heavy work to background job queues, scaling horizontally with Node.js clustering or PM2, and running CPU-intensive tasks in `worker_threads` to parallelise computation across cores.

**Beginner-Friendly Explanation:** Node.js is like a single chef in a kitchen. The chef can cook many dishes at once (asynchronous I/O) by starting one dish, putting it in the oven, and starting another while the first bakes. But if the chef has to chop 100 onions by hand (CPU-bound work), everything else stops — no new orders are taken, no dishes come out of the oven. Application performance is about making sure the chef never gets stuck chopping onions: use a food processor (worker threads), hire more chefs (clustering), or send the onions to a prep kitchen (background jobs).

### Key Characteristics

- **The event loop is the bottleneck:** Any synchronous CPU work blocks all requests.
- **Worker threads parallelise CPU work:** `worker_threads` runs JavaScript in separate threads with their own V8 isolates.
- **Clustering scales across cores:** `cluster` module or PM2 spawns multiple Node.js processes, each with its own event loop.
- **Streaming avoids memory buffers:** `fs.createReadStream()` processes files in chunks without loading them entirely into memory.
- **Memory leaks are silent killers:** Closures, event listeners, and caches that grow unbounded eventually crash the process.
- **Algorithmic complexity dominates at scale:** An O(n²) algorithm that works for 1,000 records fails at 1,000,000.
- **Background jobs decouple work from requests:** Heavy tasks are queued and processed by separate workers.

### Prerequisites

- **Node.js runtime** (v18 or higher for stable `worker_threads`).
- **Basic understanding of the event loop and asynchronous programming.**
- **Familiarity with Node.js streams and buffers.**
- **PM2 or Node.js `cluster` module** for multi-process scaling.
- **A job queue system** (BullMQ, Bee-Queue, RabbitMQ) for background jobs.
- **Profiling tools:** Node.js `--inspect`, Chrome DevTools, `clinic.js`, `0x`.

### Related Programming Areas

- **Express Performance:** Application-level middleware and routing optimisation.
- **Database Performance:** Query optimisation and connection pooling.
- **HTTP Performance:** Compression, caching, and CDN offloading.
- **Observability:** Metrics and tracing reveal where CPU and memory are spent.
- **DevOps:** PM2, Kubernetes, and process managers handle clustering and restarts.

### Core Concepts

1. **Avoid Blocking the Event Loop** — offloading heavy CPU computations.
2. **Efficient Algorithms** — optimising data structures for time and space complexity.
3. **Memory Management** — heap snapshots and avoiding closures that retain large scopes.
4. **Streaming** — `fs.createReadStream` and `pipe` for massive files.
5. **Background Jobs** — delegating intensive logic to distributed worker lines.
6. **Node.js Clustering & PM2** — spawning multiple process instances across CPU cores.
7. **Worker Threads** — `worker_threads` for in-process parallel CPU computation.

---

## Core Concept 1: Avoid Blocking the Event Loop

### Definitions

**Core Definition:** Blocking the event loop means executing a synchronous, CPU-bound operation on the main thread, which prevents Node.js from processing any other events — incoming requests, timers, or I/O callbacks — until the operation completes.

**Technical Definition:** The Node.js event loop processes callbacks in phases (timers, pending callbacks, poll, check, close). Any synchronous JavaScript execution that takes more than a few milliseconds delays the processing of all other events. Common blocking operations include: large `for`/`while` loops, synchronous file I/O (`fs.readFileSync`), `JSON.parse`/`JSON.stringify` on massive payloads, cryptographic operations (`crypto.pbkdf2Sync`), regular expression catastrophic backtracking, and image/video processing. The solution is to offload these operations to `worker_threads`, the libuv thread pool (for async I/O), or a separate process. 

**Beginner-Friendly Explanation:** The event loop is like a single checkout lane at a supermarket. If one customer has a problem with their card and takes 5 minutes to resolve, everyone behind them waits. Blocking the event loop is that slow customer. The fix is to move the problem to a separate counter (worker thread) so the main lane keeps moving. 

### Purposes

- To ensure the application remains responsive under all conditions.
- To prevent request timeouts caused by event-loop starvation.
- To identify and eliminate synchronous bottlenecks.
- To maintain throughput and low latency.

### Syntax Rules and Structure

**Blocking (bad):**
```js
app.get('/api/compute', (req, res) => {
  // Blocks the event loop for ~2 seconds
  let result = 0;
  for (let i = 0; i < 1e10; i++) result += i;
  res.json({ result });
});
```

**Non-blocking with worker thread (good):**
```js
const { Worker } = require('node:worker_threads');

app.get('/api/compute', (req, res) => {
  const worker = new Worker('./compute-worker.js', {
    workerData: { limit: 1e10 }
  });

  worker.on('message', (result) => res.json({ result }));
  worker.on('error', (err) => res.status(500).json({ error: err.message }));
});
```

**Detection tools:**
| Tool | Purpose |
|------|---------|
| `--prof` | V8 profiler for CPU hotspots. |
| `clinic doctor` | Diagnoses event loop blocking. |
| `clinic flame` | Flame graph of CPU usage. |
| `node --inspect` | Chrome DevTools profiling. |
| `eventLoopUtilization()` | Built-in ELU metric. |

**Rules:**
- Never use synchronous I/O (`readFileSync`, `execSync`) in request handlers. 
- Offload CPU-bound work (> 10ms) to worker threads or background jobs. 
- Use `setImmediate()` to break up long loops into chunks. 
- Monitor event loop lag — if p99 exceeds 100ms, find the blocking operation. 
- Profile before optimising — identify the actual bottleneck. 

### Annotated Code Example

```js
// event-loop-blocking.js
const express = require('express');
const { Worker } = require('node:worker_threads');
const { performance } = require('node:perf_hooks');

const app = express();

// ❌ Blocking: calculates fibonacci synchronously
function fibonacciBlocking(n) {
  if (n <= 1) return n;
  return fibonacciBlocking(n - 1) + fibonacciBlocking(n - 2);
}

app.get('/api/fib/blocking/:n', (req, res) => {
  const start = performance.now();
  const result = fibonacciBlocking(parseInt(req.params.n));
  const duration = performance.now() - start;
  res.json({ result, duration: `${duration.toFixed(2)}ms`, blocked: true });
});

// ✅ Non-blocking: offloads to a worker thread
app.get('/api/fib/worker/:n', (req, res) => {
  const worker = new Worker('./fib-worker.js', {
    workerData: { n: parseInt(req.params.n) }
  });

  worker.on('message', (result) => res.json({ result, blocked: false }));
  worker.on('error', (err) => res.status(500).json({ error: err.message }));
});

app.get('/api/health', (req, res) => res.json({ status: 'ok' }));

app.listen(3000, () => console.log('Server on 3000'));
```

```js
// fib-worker.js
const { parentPort, workerData } = require('node:worker_threads');

function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

const result = fibonacci(workerData.n);
parentPort.postMessage(result);
```

**Expected Output (blocking endpoint — concurrent health check):**
```
GET /api/fib/blocking/40 → 200 OK (1200ms)
GET /api/health (during computation) → delayed by 1200ms
```

**Expected Output (worker endpoint — concurrent health check):**
```
GET /api/fib/worker/40 → 200 OK (1200ms)
GET /api/health (during computation) → 200 OK (2ms)  ✅ Event loop not blocked
```

**Why this output:** The blocking endpoint calculates fibonacci(40) synchronously, freezing the event loop for 1.2 seconds. During that time, the health check cannot be served. The worker endpoint offloads the same computation to a worker thread, leaving the event loop free to serve the health check immediately. 

### Real-World Cases

- **Image processing:** Resizing and watermarking images in worker threads.
- **Cryptography:** Password hashing (bcrypt, argon2) in worker threads or the libuv thread pool.
- **Data transformation:** Parsing and transforming large CSV or JSON files in workers.
- **Video encoding:** Offloading transcoding to worker threads or child processes.

---

## Core Concept 2: Efficient Algorithms

### Definitions

**Core Definition:** Efficient algorithm selection means choosing data structures and algorithms whose time and space complexity are appropriate for the size of the data being processed.

**Technical Definition:** Algorithmic complexity is expressed in Big O notation. An O(n) algorithm scales linearly with input size; O(n²) scales quadratically (doubling input quadruples time); O(log n) scales logarithmically (ideal for search). Common Node.js performance pitfalls include: using `Array.prototype.includes()` inside a loop (O(n²)), using array methods when a `Set` or `Map` would be O(1), repeatedly re-computing values that could be memoised, and using string concatenation in loops instead of array joins. Choosing the right data structure — `Set` for membership tests, `Map` for key-value lookups, typed arrays for numeric data — often yields larger gains than micro-optimising code. 

**Beginner-Friendly Explanation:** If you have a list of 1,000,000 names and you want to check if "Alice" is in it, using `includes()` means checking every name one by one (O(n)). Using a `Set` means checking one lookup table (O(1)). The Set uses more memory but answers the question almost instantly, regardless of how many names there are. 

### Purposes

- To reduce execution time as data volume grows.
- To avoid performance cliffs when input size increases.
- To choose the right data structure for the access pattern.
- To reduce memory usage through efficient representation.

### Syntax Rules and Structure

**O(n²) — using `includes()` in a loop:**
```js
const userIds = [1, 2, 3, ..., 10000];
const activeUsers = [1, 5, 9, ...];
const filtered = userIds.filter(id => activeUsers.includes(id));  // O(n*m)
```

**O(n) — using a `Set`:**
```js
const activeSet = new Set(activeUsers);
const filtered = userIds.filter(id => activeSet.has(id));  // O(n)
```

**Data structure selection:**
| Operation | Array | Set | Map | Object |
|-----------|-------|-----|-----|--------|
| Lookup by value | O(n) | O(1) | O(1) | O(1) |
| Insert | O(1)* | O(1) | O(1) | O(1) |
| Delete | O(n) | O(1) | O(1) | O(1) |
| Iterate | O(n) | O(n) | O(n) | O(n) |
| Key types | Index | Any | Any | String/Symbol |

*Amortised.

**Rules:**
- Use `Set` for membership tests, not `Array.includes()`. 
- Use `Map` for key-value lookups when keys are not strings. 
- Avoid nested loops when a single pass with a lookup table works. 
- Use `Array.join()` instead of string concatenation in loops. 
- Memoise expensive pure functions with `Map` or a cache. 

### Annotated Code Example

```js
// efficient-algorithms.js
const { performance } = require('node:perf_hooks');

// Generate test data
const userIds = Array.from({ length: 100000 }, (_, i) => i);
const activeUsers = Array.from({ length: 50000 }, (_, i) => i * 2);

// ❌ O(n²) — includes() in a loop
function filterWithIncludes() {
  const start = performance.now();
  const result = userIds.filter(id => activeUsers.includes(id));
  const duration = performance.now() - start;
  console.log(`includes(): ${result.length} results in ${duration.toFixed(2)}ms`);
  return result;
}

// ✅ O(n) — Set lookup
function filterWithSet() {
  const activeSet = new Set(activeUsers);
  const start = performance.now();
  const result = userIds.filter(id => activeSet.has(id));
  const duration = performance.now() - start;
  console.log(`Set:        ${result.length} results in ${duration.toFixed(2)}ms`);
  return result;
}

filterWithIncludes();  // ~2000ms
filterWithSet();       // ~5ms
```

**Expected Output:**
```
includes(): 50000 results in 2145.32ms
Set:        50000 results in 4.87ms
```

**Why this output:** The `includes()` approach checks each of the 100,000 user IDs against all 50,000 active users — up to 5 billion comparisons. The `Set` approach builds a hash table of active users once (O(n)), then checks each user ID in O(1). The result is 440× faster. 

### Real-World Cases

- **Data deduplication:** Using `Set` to remove duplicates in O(n).
- **Permission checks:** Using a `Set` of permissions for O(1) lookups.
- **Graph traversal:** Using `Map` for visited nodes in BFS/DFS.
- **Caching:** Using `Map` for memoisation of expensive computations.

---

## Core Concept 3: Memory Management

### Definitions

**Core Definition:** Memory management is the practice of preventing memory leaks — situations where objects are retained in memory longer than necessary — through careful closure design, event listener cleanup, cache eviction, and heap analysis.

**Technical Definition:** V8's garbage collector reclaims memory that is no longer reachable from the root (global object, call stack, closures). Memory leaks occur when references to large objects are retained unintentionally. Common Node.js memory leaks include: **closures** that capture large scopes and outlive their intended use, **event listeners** that are added but never removed, **global caches** that grow unbounded, **timers** (`setInterval`) that are never cleared, and **detached DOM elements** (in browser contexts). Diagnosis uses `process.memoryUsage()`, `v8.writeHeapSnapshot()`, and Chrome DevTools' Memory panel to compare heap snapshots and identify growing object counts. 

**Beginner-Friendly Explanation:** Imagine a room where you keep every newspaper you have ever read. Eventually, the room is full and you cannot move. A memory leak is keeping references to objects you no longer need. The garbage collector wants to throw them away, but your code is still holding them. 

### Purposes

- To prevent process crashes from out-of-memory errors.
- To reduce garbage collection pauses and CPU usage.
- To maintain stable memory usage over long-running processes.
- To identify and fix leaks before they cause outages.

### Syntax Rules and Structure

**Diagnostic tools:**
```js
// Memory usage
const usage = process.memoryUsage();
console.log({
  rss: `${(usage.rss / 1024 / 1024).toFixed(2)} MB`,
  heapTotal: `${(usage.heapTotal / 1024 / 1024).toFixed(2)} MB`,
  heapUsed: `${(usage.heapUsed / 1024 / 1024).toFixed(2)} MB`,
  external: `${(usage.external / 1024 / 1024).toFixed(2)} MB`
});

// Heap snapshot
const v8 = require('node:v8');
v8.writeHeapSnapshot('/tmp/heap-${Date.now()}.heapsnapshot');
```

**Common leaks and fixes:**
| Leak Source | Fix |
|------------|-----|
| Event listeners | `removeListener()` or `off()` on cleanup. |
| `setInterval` | `clearInterval()` on cleanup. |
| Global cache | Use LRU cache with `maxSize`. |
| Closures retaining large objects | Null out references when done. |

**Rules:**
- Always remove event listeners when the component unmounts. 
- Clear timers (`clearInterval`, `clearTimeout`) in cleanup functions. 
- Use bounded caches (LRU) instead of unbounded `Map` or object. 
- Null out large references (`this.data = null`) when they are no longer needed. 
- Take heap snapshots before and after suspected leak scenarios. 

### Annotated Code Example

```js
// memory-leak.js
const express = require('express');
const app = express();

// ❌ Leak: unbounded cache grows forever
const leakyCache = new Map();

app.get('/api/leak', (req, res) => {
  const key = `request-${Date.now()}-${Math.random()}`;
  const largeObject = { data: new Array(10000).fill('x'.repeat(1000)) };
  leakyCache.set(key, largeObject);  // Never evicted
  res.json({ cached: leakyCache.size });
});

// ✅ Fix: bounded LRU-style cache
const MAX_CACHE_SIZE = 100;
const boundedCache = new Map();

app.get('/api/fixed', (req, res) => {
  const key = `request-${Date.now()}`;
  const largeObject = { data: new Array(10000).fill('x'.repeat(1000)) };

  if (boundedCache.size >= MAX_CACHE_SIZE) {
    const oldestKey = boundedCache.keys().next().value;
    boundedCache.delete(oldestKey);
  }

  boundedCache.set(key, largeObject);
  res.json({ cached: boundedCache.size });
});

// Memory monitoring
setInterval(() => {
  const usage = process.memoryUsage();
  console.log(`Heap: ${(usage.heapUsed / 1024 / 1024).toFixed(2)} MB`);
}, 5000);

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (leak endpoint — heap grows unbounded):**
```
Heap: 45.2 MB
Heap: 120.5 MB
Heap: 380.1 MB
Heap: 890.3 MB
FATAL ERROR: JavaScript heap out of memory
```

**Expected Output (fixed endpoint — heap stays stable):**
```
Heap: 45.2 MB
Heap: 48.5 MB
Heap: 47.1 MB
Heap: 46.8 MB
(heap stabilises)
```

**Why this output:** The leaky cache stores every request's large object forever, growing the heap until V8 crashes with an out-of-memory error. The bounded cache evicts the oldest entry when the size limit is reached, keeping memory usage stable. 

### Real-World Cases

- **Long-running servers:** Preventing heap growth over days or weeks. 
- **WebSocket connections:** Removing listeners when clients disconnect. 
- **Caching layers:** Using LRU eviction to bound memory usage. 
- **Streaming:** Avoiding buffering of large payloads.

---

## Core Concept 4: Streaming

### Definitions

**Core Definition:** Streaming processes data in small chunks as it arrives, rather than loading the entire payload into memory, enabling the handling of files and data sets larger than available memory.

**Technical Definition:** Node.js streams are EventEmitter-based abstractions for reading and writing data incrementally. `fs.createReadStream()` reads a file in chunks (default 64 KB), emitting `data` events. `stream.pipe()` connects a readable stream to a writable stream, automatically managing backpressure — if the destination is slower than the source, the source is paused. `stream.pipeline()` (from `node:stream`) provides error propagation and cleanup across multiple streams. For HTTP responses, piping a file stream directly to `res` avoids loading the file into memory. 

**Beginner-Friendly Explanation:** Streaming is like drinking from a fire hose with a cup instead of trying to swallow the whole hose. You take small sips (chunks), and the water keeps flowing. You never have to hold all the water in your mouth at once. 

### Purposes

- To process files larger than available memory.
- To start sending data to the client before the entire file is read.
- To avoid out-of-memory crashes from large payloads.
- To handle backpressure automatically.

### Syntax Rules and Structure

**❌ Loading a large file into memory:**
```js
app.get('/download', (req, res) => {
  const data = fs.readFileSync('huge-file.zip');  // Loads 5GB into memory
  res.send(data);
});
```

**✅ Streaming a file:**
```js
app.get('/download', (req, res) => {
  const stream = fs.createReadStream('huge-file.zip');
  stream.pipe(res);  // Streams in 64KB chunks
});
```

**✅ Stream with error handling (`pipeline`):**
```js
const { pipeline } = require('node:stream/promises');

app.get('/download', async (req, res) => {
  try {
    await pipeline(
      fs.createReadStream('huge-file.zip'),
      res
    );
  } catch (err) {
    if (!res.headersSent) res.status(500).end();
  }
});
```

| Method | Memory | Backpressure | Error Handling |
|--------|--------|-------------|----------------|
| `readFileSync` | Entire file | ❌ | `try/catch`. |
| `.pipe()` | 64KB chunks | ✅ Automatic | Manual. |
| `pipeline()` | 64KB chunks | ✅ Automatic | ✅ Automatic. |

**Rules:**
- Never use `readFileSync` for files larger than a few MB. 
- Use `pipeline()` instead of `.pipe()` for production — it propagates errors and cleans up. 
- Set `highWaterMark` to tune chunk size (default 64 KB). 
- Handle stream errors before headers are sent. 
- Use streams for database exports, CSV generation, and file uploads. 

### Annotated Code Example

```js
// streaming.js
const express = require('express');
const fs = require('fs');
const { pipeline } = require('node:stream/promises');
const { Transform } = require('node:stream');

const app = express();

// Stream a large file
app.get('/download', async (req, res) => {
  res.setHeader('Content-Type', 'application/octet-stream');
  res.setHeader('Content-Disposition', 'attachment; filename="large-file.zip"');

  try {
    await pipeline(
      fs.createReadStream('large-file.zip'),
      res
    );
  } catch (err) {
    console.error('Stream error:', err.message);
    if (!res.headersSent) res.status(500).end();
  }
});

// Stream with transformation (uppercase)
app.get('/download/transformed', async (req, res) => {
  const upperCase = new Transform({
    transform(chunk, encoding, callback) {
      callback(null, chunk.toString().toUpperCase());
    }
  });

  try {
    await pipeline(
      fs.createReadStream('data.txt'),
      upperCase,
      res
    );
  } catch (err) {
    if (!res.headersSent) res.status(500).end();
  }
});

// Stream database results to CSV
app.get('/export/users', async (req, res) => {
  res.setHeader('Content-Type', 'text/csv');
  res.setHeader('Content-Disposition', 'attachment; filename="users.csv"');

  res.write('id,name,email\n');

  const cursor = db.collection('users').find().stream();
  for await (const user of cursor) {
    res.write(`${user.id},${user.name},${user.email}\n`);
  }
  res.end();
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (memory comparison for a 5GB file):**
```
readFileSync:  Memory usage spikes to 5GB+ → crash
createReadStream: Memory usage stays at ~64MB → success
```

**Why this output:** `readFileSync` loads the entire 5 GB file into memory, causing an out-of-memory crash. `createReadStream` reads the file in 64 KB chunks and pipes each chunk to the response, keeping memory usage constant regardless of file size. 

### Real-World Cases

- **File downloads:** Serving large files without buffering.
- **CSV exports:** Streaming query results directly to the response.
- **Video/audio streaming:** Serving media with range requests.
- **Log processing:** Reading and processing large log files line by line.

---

## Core Concept 5: Background Jobs

### Definitions

**Core Definition:** Background jobs are tasks that are executed outside the request–response cycle, typically by a separate worker process, so that heavy or long-running operations do not block the web server or delay responses.

**Technical Definition:** Background job systems use a message queue (Redis, RabbitMQ, SQS) to store job payloads. The web server enqueues a job and immediately returns a response to the client; a separate worker process (or processes) dequeues the job and executes it. Popular Node.js libraries include BullMQ (Redis-based), Bee-Queue, and Agenda (MongoDB-based). Background jobs are appropriate for: sending emails, generating reports, processing images/videos, syncing data with external services, and any operation that takes more than a few hundred milliseconds. 

**Beginner-Friendly Explanation:** Background jobs are like ordering food at a restaurant. You place your order (enqueue a job) and the waiter immediately goes to the next table (returns a response). The kitchen (worker) prepares your food and brings it out when it is ready. You do not stand at the counter waiting for the chef to finish cooking. 

### Purposes

- To decouple long-running work from the request–response cycle.
- To prevent request timeouts for slow operations.
- To scale background processing independently of web servers.
- To retry failed operations without affecting the user.

### Syntax Rules and Structure

**BullMQ setup:**
```js
// queue.js
const { Queue } = require('bullmq');
const connection = { host: 'localhost', port: 6379 };

const emailQueue = new Queue('email', { connection });

module.exports = { emailQueue };
```

```js
// web server — enqueue a job
app.post('/api/users', async (req, res) => {
  const user = await User.create(req.body);

  // Enqueue welcome email — returns immediately
  await emailQueue.add('welcome', {
    userId: user.id,
    email: user.email
  }, {
    attempts: 3,
    backoff: { type: 'exponential', delay: 1000 }
  });

  res.status(201).json(user);
});
```

```js
// worker.js — process jobs
const { Worker } = require('bullmq');
const connection = { host: 'localhost', port: 6379 };

const worker = new Worker('email', async (job) => {
  console.log(`Processing job ${job.id}: ${job.name}`);

  if (job.name === 'welcome') {
    await sendWelcomeEmail(job.data.email);
  }
}, { connection, concurrency: 5 });

worker.on('completed', (job) => console.log(`Job ${job.id} completed`));
worker.on('failed', (job, err) => console.error(`Job ${job.id} failed:`, err.message));
```

| Library | Backend | Features |
|---------|---------|----------|
| BullMQ | Redis | Delayed jobs, repeatable jobs, rate limiting. |
| Bee-Queue | Redis | Simple, fast, lightweight. |
| Agenda | MongoDB | Cron-like scheduling. |
| SQS | AWS | Managed, serverless-friendly. |

**Rules:**
- Use background jobs for any operation over 500ms. 
- Configure retries with exponential backoff. 
- Monitor queue depth and job failure rates. 
- Scale workers independently of web servers. 
- Make jobs idempotent — they may be retried. 

### Annotated Code Example

```js
// background-jobs.js
const express = require('express');
const { Queue, Worker } = require('bullmq');
const app = express();

const connection = { host: 'localhost', port: 6379 };
const reportQueue = new Queue('reports', { connection });

// Web server — enqueue job
app.post('/api/reports', async (req, res) => {
  const job = await reportQueue.add('generate', {
    userId: req.body.userId,
    dateRange: req.body.dateRange
  }, {
    attempts: 3,
    backoff: { type: 'exponential', delay: 2000 }
  });

  res.status(202).json({
    jobId: job.id,
    status: 'queued',
    message: 'Report generation started'
  });
});

// Check job status
app.get('/api/reports/:jobId', async (req, res) => {
  const job = await reportQueue.getJob(req.params.jobId);
  if (!job) return res.status(404).json({ error: 'Job not found' });

  const state = await job.getState();
  res.json({ jobId: job.id, state, result: job.returnvalue });
});

app.listen(3000, () => console.log('Web server on 3000'));

// --- Worker process (separate file: worker.js) ---
const worker = new Worker('reports', async (job) => {
  const { userId, dateRange } = job.data;

  // Simulate heavy report generation
  await new Promise(resolve => setTimeout(resolve, 10000));

  return { url: `/reports/${userId}-${dateRange}.pdf` };
}, { connection, concurrency: 3 });

worker.on('completed', (job) => console.log(`Job ${job.id} completed`));
worker.on('failed', (job, err) => console.error(`Job ${job.id} failed:`, err.message));
```

**Expected Output (for `POST /api/reports`):**
```json
{"jobId":"1","status":"queued","message":"Report generation started"}
```

**Expected Output (for `GET /api/reports/1` after 10 seconds):**
```json
{"jobId":"1","state":"completed","result":{"url":"/reports/42-2026-01.pdf"}}
```

**Why this output:** The web server enqueues a job and immediately returns a 202 Accepted response. The worker process picks up the job, performs the heavy computation (10 seconds), and stores the result. The client polls the status endpoint to check when the report is ready. The web server is never blocked. 

### Real-World Cases

- **Email sending:** Enqueuing welcome emails, password resets, and notifications.
- **Report generation:** Generating PDF/CSV reports in the background.
- **Image/video processing:** Thumbnails, transcoding, and watermarking.
- **Data synchronisation:** Syncing with external APIs and third-party services.

---

## Core Concept 6: Node.js Clustering & PM2

### Definitions

**Core Definition:** Clustering spawns multiple Node.js process instances — each with its own event loop and memory — on the same machine, distributing incoming connections across CPU cores to maximise throughput.

**Technical Definition:** The Node.js `cluster` module allows a master process to fork worker processes that share the same server port. The master distributes incoming connections across workers using a round-robin algorithm (default on all platforms except Windows). PM2 is a production process manager that wraps clustering with additional features: automatic restarts on crash, log management, zero-downtime reloads, and monitoring. Each worker process is a full Node.js instance with its own V8 isolate and event loop, so CPU-bound work in one worker does not block others. 

**Beginner-Friendly Explanation:** A single Node.js process uses only one CPU core, even if your server has 16. Clustering is like opening 16 checkout lanes instead of one — each lane (process) handles its own customers, so 16 customers can be served simultaneously instead of one. PM2 is the manager that keeps all 16 lanes open and restarts any lane that breaks.

### Purposes

- To utilise all CPU cores on the server.
- To increase throughput by running multiple event loops.
- To provide fault tolerance — if one worker crashes, others continue serving.
- To enable zero-downtime deployments (PM2 reload).

### Syntax Rules and Structure

**Node.js `cluster` module:**
```js
const cluster = require('node:cluster');
const os = require('node:os');

if (cluster.isPrimary) {
  const numCPUs = os.cpus().length;
  console.log(`Master ${process.pid} forking ${numCPUs} workers`);

  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died. Restarting...`);
    cluster.fork();
  });
} else {
  // Worker process — start the Express app
  require('./app');
  console.log(`Worker ${process.pid} started`);
}
```

**PM2 ecosystem file (`ecosystem.config.js`):**
```js
module.exports = {
  apps: [{
    name: 'my-app',
    script: './app.js',
    instances: 'max',        // Use all CPU cores
    exec_mode: 'cluster',    // Cluster mode
    watch: false,
    max_memory_restart: '500M',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    }
  }]
};
```

**PM2 commands:**
```bash
pm2 start ecosystem.config.js    # Start with clustering
pm2 reload my-app                # Zero-downtime reload
pm2 scale my-app 8               # Scale to 8 instances
pm2 monit                        # Monitor CPU/memory
pm2 logs my-app                  # View logs
```

| Feature | `cluster` module | PM2 |
|---------|-----------------|-----|
| Clustering | ✅ | ✅ |
| Auto-restart | Manual | ✅ |
| Zero-downtime reload | Manual | ✅ |
| Log management | Manual | ✅ |
| Monitoring | ❌ | ✅ |

**Rules:**
- Use clustering for all production Node.js deployments. 
- Set `instances: 'max'` (PM2) or `os.cpus().length` (cluster) to use all cores. 
- Use PM2's `reload` (not `restart`) for zero-downtime deployments. 
- Set `max_memory_restart` to automatically restart workers that leak memory. 
- Do not cluster when using WebSockets with in-memory state — use Redis for shared state. 

### Annotated Code Example

```js
// cluster-app.js
const cluster = require('node:cluster');
const os = require('node:os');
const express = require('express');

if (cluster.isPrimary) {
  const numCPUs = os.cpus().length;
  console.log(`Primary ${process.pid} is running`);
  console.log(`Forking ${numCPUs} workers...`);

  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died (code: ${code}, signal: ${signal})`);
    console.log('Forking a new worker...');
    cluster.fork();
  });
} else {
  const app = express();

  app.get('/api/data', (req, res) => {
    res.json({
      pid: process.pid,
      message: `Served by worker ${process.pid}`
    });
  });

  app.listen(3000, () => {
    console.log(`Worker ${process.pid} started on port 3000`);
  });
}
```

**Expected Output (with 4 CPU cores):**
```
Primary 12345 is running
Forking 4 workers...
Worker 12346 started on port 3000
Worker 12347 started on port 3000
Worker 12348 started on port 3000
Worker 12349 started on port 3000
```

**Expected Output (requests distributed across workers):**
```json
{"pid":12346,"message":"Served by worker 12346"}
{"pid":12347,"message":"Served by worker 12347"}
{"pid":12348,"message":"Served by worker 12348"}
{"pid":12349,"message":"Served by worker 12349"}
```

**Why this output:** The primary process forks four workers (one per CPU core). Each worker runs its own Express server on the same port. The cluster module distributes incoming connections across the workers using round-robin. Each request is served by a different worker, as shown by the `pid` field. 

### Real-World Cases

- **Production Node.js deployments:** Every Node.js server should use clustering.
- **High-traffic APIs:** Distributing load across all CPU cores.
- **Zero-downtime deployments:** PM2 reload for rolling restarts.
- **Multi-core VPS:** Maximising resource utilisation.

---

## Core Concept 7: Worker Threads

### Definitions

**Core Definition:** Worker threads (`worker_threads`) run JavaScript in separate threads within the same process, each with its own V8 isolate, enabling parallel execution of CPU-bound tasks without blocking the main event loop.

**Technical Definition:** The `worker_threads` module allows creating workers that run a separate JavaScript file (or inline code) in a new thread. Each worker has its own event loop, V8 heap, and `require` cache. Communication between the main thread and workers uses `postMessage()` and `on('message')`, with structured clone or `SharedArrayBuffer` for shared memory. Worker threads are ideal for CPU-bound work — the main thread stays responsive, and the worker executes the computation in parallel. Worker threads have lower overhead than child processes because they share the same process memory space. 

**Beginner-Friendly Explanation:** Worker threads are like hiring a specialist to do a specific job. The main thread continues handling requests while the specialist (worker) performs the heavy computation. When the specialist finishes, they send the result back. Unlike child processes, worker threads share the same memory space, so communication is faster. 

### Purposes

- To parallelise CPU-bound work without blocking the event loop.
- To avoid the overhead of child processes.
- To share memory between threads via `SharedArrayBuffer`.
- To process large data sets in parallel.

### Syntax Rules and Structure

**Main thread:**
```js
const { Worker } = require('node:worker_threads');

const worker = new Worker('./worker.js', {
  workerData: { n: 40 }
});

worker.on('message', (result) => console.log('Result:', result));
worker.on('error', (err) => console.error('Error:', err));
worker.on('exit', (code) => console.log('Worker exited with code:', code));
```

**Worker thread (`worker.js`):**
```js
const { parentPort, workerData } = require('node:worker_threads');

function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

const result = fibonacci(workerData.n);
parentPort.postMessage(result);
```

**Worker pool pattern:**
```js
const { Worker } = require('node:worker_threads');

function runWorker(workerData) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./worker.js', { workerData });
    worker.on('message', resolve);
    worker.on('error', reject);
    worker.on('exit', (code) => {
      if (code !== 0) reject(new Error(`Worker exited with code ${code}`));
    });
  });
}

// Run multiple workers in parallel
const results = await Promise.all([
  runWorker({ n: 40 }),
  runWorker({ n: 40 }),
  runWorker({ n: 40 }),
  runWorker({ n: 40 })
]);
```

| Aspect | Worker Threads | Child Processes | Cluster |
|--------|---------------|----------------|---------|
| Memory | Shared process | Separate process | Separate process |
| Startup | Fast | Slower | Slow |
| Communication | `postMessage` | IPC | IPC |
| Use case | CPU tasks | Isolation | Network scaling |

**Rules:**
- Use worker threads for CPU-bound tasks > 10ms. 
- Reuse workers via a pool instead of creating one per request. 
- Use `SharedArrayBuffer` for sharing large data without copying. 
- Handle worker errors and exits to prevent resource leaks. 
- Do not use worker threads for I/O — the event loop handles I/O efficiently. 

### Annotated Code Example

```js
// worker-threads.js
const express = require('express');
const { Worker } = require('node:worker_threads');
const os = require('node:os');

const app = express();

// Worker pool
class WorkerPool {
  constructor(workerScript, poolSize = os.cpus().length) {
    this.workers = [];
    this.queue = [];

    for (let i = 0; i < poolSize; i++) {
      this.workers.push({ worker: new Worker(workerScript), busy: false });
    }
  }

  run(data) {
    return new Promise((resolve, reject) => {
      const available = this.workers.find(w => !w.busy);

      if (available) {
        this.execute(available, data, resolve, reject);
      } else {
        this.queue.push({ data, resolve, reject });
      }
    });
  }

  execute(slot, data, resolve, reject) {
    slot.busy = true;
    slot.worker.once('message', (result) => {
      slot.busy = false;
      resolve(result);
      this.processQueue();
    });
    slot.worker.once('error', (err) => {
      slot.busy = false;
      reject(err);
      this.processQueue();
    });
    slot.worker.postMessage(data);
  }

  processQueue() {
    if (this.queue.length === 0) return;
    const available = this.workers.find(w => !w.busy);
    if (available) {
      const { data, resolve, reject } = this.queue.shift();
      this.execute(available, data, resolve, reject);
    }
  }
}

const pool = new WorkerPool('./cpu-worker.js', 4);

app.get('/api/compute/:n', async (req, res) => {
  const result = await pool.run({ n: parseInt(req.params.n) });
  res.json({ result });
});

app.listen(3000, () => console.log('Server on 3000'));
```

```js
// cpu-worker.js
const { parentPort } = require('node:worker_threads');

parentPort.on('message', (data) => {
  function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
  }
  parentPort.postMessage(fibonacci(data.n));
});
```

**Expected Output (4 concurrent requests):**
```
GET /api/compute/40 → 200 OK (1200ms)
GET /api/compute/40 → 200 OK (1200ms)  ← parallel
GET /api/compute/40 → 200 OK (1200ms)  ← parallel
GET /api/compute/40 → 200 OK (1200ms)  ← parallel
```

**Why this output:** The worker pool maintains four workers (one per CPU core). Each request is dispatched to an available worker. The four requests execute in parallel on four CPU cores, each taking 1200ms. Without worker threads, the four requests would execute sequentially (4800ms total). 

### Real-World Cases

- **Image processing:** Resizing, cropping, and watermarking images.
- **Cryptography:** Password hashing and encryption.
- **Data processing:** Parsing large JSON/CSV files.
- **Compression:** Gzip/Brotli compression of large payloads.

---

## References

- Node.js Worker Threads Documentation — https://nodejs.org/api/worker_threads.html
- Node.js Cluster Module Documentation — https://nodejs.org/api/cluster.html
- Node.js Streams Documentation — https://nodejs.org/api/stream.html
- PM2 Documentation — https://pm2.keymetrics.io/docs/usage/quick-start/
- BullMQ Documentation — https://docs.bullmq.io/
- Clinic.js — https://clinicjs.org/
- 0x Flame Graph Profiler — https://github.com/davidmarkclements/0x
- Node.js Memory Management — https://nodejs.org/en/learn/diagnostics/memory
- V8 Heap Snapshots — https://nodejs.org/api/v8.html#v8writeheapsnapshotfilenameoptions
- Understanding the Node.js Event Loop — https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-nexttick
- Avoiding Event Loop Blocking — https://nodejs.org/en/learn/asynchronous-work/dont-block-the-event-loop
- Node.js Performance Best Practices — https://github.com/goldbergyoni/nodebestpractices
- Big O Cheat Sheet — https://www.bigocheatsheet.com/
- LRU Cache for Node.js — https://www.npmjs.com/package/lru-cache
- Stream.pipeline Documentation — https://nodejs.org/api/stream.html#streampipelinesource-transforms-destination-callback