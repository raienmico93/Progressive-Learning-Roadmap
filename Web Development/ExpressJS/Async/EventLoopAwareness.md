# Event Loop Awareness & Performance Optimization — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Event loop awareness is the understanding of how Node.js's single-threaded event loop executes JavaScript, schedules asynchronous callbacks, and processes I/O — and how blocking that loop degrades the entire application's responsiveness.

**Technical Definition:** Node.js runs JavaScript on a single main thread using an event loop that processes phases (timers, pending callbacks, poll, check, close callbacks) in repeated ticks. Asynchronous operations are offloaded to the operating system or libuv threadpool, and their callbacks are queued for execution when complete. If any synchronous operation monopolises the main thread — a CPU-bound loop, a large `JSON.parse`, a synchronous file read, or a synchronous cryptographic function — the event loop cannot advance, and every pending timer, socket callback, and incoming HTTP request is delayed. This phenomenon is called **event loop blocking** or **event loop starvation**. Performance optimization in Node.js consists of identifying and eliminating blocking patterns, offloading CPU-bound work to worker threads, maximising I/O-bound concurrency, and scaling across CPU cores via clustering.

**Beginner-Friendly Explanation:** Node.js is like a single chef in a kitchen. The chef (the event loop) takes orders (requests), starts cooking (asynchronous I/O), and moves on to the next order while the oven bakes. But if the chef decides to chop 10,000 onions by hand (a CPU-bound task), no new orders get taken and no dishes get served — the entire restaurant freezes. Event loop awareness is knowing that the chef must delegate heavy chopping to a prep cook (worker thread) and keep taking orders.

### Key Characteristics

- **Single-threaded execution:** JavaScript runs on one thread; CPU-bound work blocks everything.
- **I/O concurrency:** Asynchronous I/O is handled by the OS/libuv threadpool; the event loop stays free.
- **Loop lag measurable:** `perf_hooks.monitorEventLoopDelay()` quantifies how late the loop wakes up.
- **Microtask priority:** Promise callbacks run before macrotasks (timers, I/O), and a runaway microtask chain can starve the loop entirely.
- **Horizontal scaling available:** `worker_threads` offloads CPU work; `cluster`/PM2 scales across cores.
- **Context propagation:** `AsyncLocalStorage` maintains request-scoped state across async boundaries.

### Prerequisites

- **Node.js runtime** (v18 or higher; `perf_hooks.monitorEventLoopDelay` is stable since v16; `worker_threads` stable since v12; `AsyncLocalStorage` stable since v16.4).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript knowledge:** Promises, `async/await`, and the event loop.

### Related Programming Areas

- **Async/Await & Promise Integration:** The foundation for non-blocking operations.
- **Concurrency Management:** Bounding concurrent operations protects the loop and downstream resources.
- **Production Error Handling & Observability:** Loop lag is a critical production metric.
- **Database & Network Operations:** Async I/O is the primary way the loop stays free.

### Core Concepts

1. **Blocking Operations** — identifying and eliminating blocking patterns.
2. **CPU-Bound Workloads** — offloading heavy computation away from the main thread.
3. **I/O-Bound Workloads** — maximising Node's high-throughput async architecture.
4. **Event-Loop Starvation** — how heavy loops freeze HTTP traffic and how to measure loop lag.
5. **Worker Threads & Clustering** — `worker_threads` for in-process parallelism; `cluster`/PM2 for multi-core scaling.
6. **Asynchronous Context Tracking** — `AsyncLocalStorage` for request-scoped state.

---

## Core Concept 1: Blocking Operations

### Definitions

**Core Definition:** A blocking operation is any synchronous JavaScript execution that prevents the event loop from advancing to its next phase, thereby delaying all pending callbacks, timers, and incoming requests.

**Technical Definition:** Blocking occurs when synchronous code occupies the main thread for a measurable duration. Common sources include: synchronous file system calls (`readFileSync`, `writeFileSync`), synchronous cryptographic functions (`pbkdf2Sync`, `randomBytes` with large sizes), large `JSON.parse`/`JSON.stringify` payloads, catastrophic regex backtracking, and CPU-intensive loops. During the block, the event loop cannot process timers, I/O callbacks, or new connections. CPU usage may show only one core pegged at 100%, while the service appears "healthy" in terms of process status but is completely unresponsive to users.

**Beginner-Friendly Explanation:** A blocking operation is like a cashier who stops serving the entire queue to go count the coins in the register. Everyone behind them waits — not because there's a problem with their orders, but because the cashier is stuck doing something that should have been delegated.

### Purposes

- To identify patterns that freeze the event loop and delay all concurrent requests.
- To eliminate synchronous calls from request-handling paths.
- To replace blocking operations with their asynchronous equivalents.
- To measure and quantify the impact of blocking on latency.

### Syntax Rules and Structure

#### Blocking vs. Non-Blocking Equivalents

| Blocking (❌) | Non-Blocking (✅) | Notes |
|---------------|-------------------|-------|
| `fs.readFileSync()` | `fs.promises.readFile()` or `fs.createReadStream()` | Sync file I/O blocks until the entire file is read. |
| `crypto.pbkdf2Sync()` | `crypto.pbkdf2()` (async) | Sync crypto blocks during key derivation. |
| `JSON.parse(hugeString)` | Stream-based JSON parsing (e.g., `stream-json`) | Large JSON parsing blocks the thread. |
| `calculatePrimes(1000000)` | Worker thread | Pure CPU loops block indefinitely. |

**Rules:**
- **Never** use `*Sync` methods inside request handlers or middleware.
- Replace synchronous crypto with async variants or worker threads.
- Chunk large loops with `await setImmediate()` between batches so the loop can process other work.
- Check regexes for catastrophic backtracking — a regex that is instant on short input but hangs on 10 KB is a backtracking problem.

**Constraints:**
- Some blocking operations (e.g., `JSON.parse` on a 5 MB string) may seem fast in development but become catastrophic under concurrent load.
- CPU usage alone is misleading — a blocked Node process may show only one busy core while the service is unresponsive.

### Annotated Code Example

```javascript
// blocking-detection.js
const { monitorEventLoopDelay } = require('node:perf_hooks');
const histogram = monitorEventLoopDelay({ resolution: 20 });
histogram.enable();

// Report loop lag every 10 seconds
setInterval(() => {
  const p50 = Math.round(histogram.mean / 1e6);
  const p99 = Math.round(histogram.percentile(99) / 1e6);
  const max = Math.round(histogram.max / 1e6);
  console.log({ p50, p99, max });
  histogram.reset();
}, 10_000);

// ❌ BAD: Blocking operation in a request handler
app.get('/bad', (req, res) => {
  const data = require('fs').readFileSync('/path/to/large.json'); // BLOCKS
  res.json(JSON.parse(data)); // ALSO BLOCKS
});

// ✅ GOOD: Non-blocking equivalents
app.get('/good', async (req, res) => {
  const data = await require('fs').promises.readFile('/path/to/large.json');
  res.json(JSON.parse(data));
});
```

**Expected Output (healthy loop):**
```
{ p50: 2, p99: 5, max: 8 }
```

**Expected Output (blocked loop):**
```
{ p50: 45, p99: 320, max: 1200 }
```

**Why this output:** On a healthy service, the mean loop delay sits in single-digit milliseconds. A p99 in the hundreds means something synchronous is running long enough for users to feel it. A max in the thousands means real CPU work is happening on the main thread.

### Real-World Cases

- **API request handlers:** Replacing `fs.readFileSync` with `fs.promises.readFile`.
- **Password hashing:** Replacing `pbkdf2Sync` with async `pbkdf2` or worker threads.
- **Data import:** Streaming large JSON files instead of `JSON.parse` on the entire file.
- **Regex validation:** Auditing regex patterns for catastrophic backtracking.

---

## Core Concept 2: CPU-Bound Workloads

### Definitions

**Core Definition:** A CPU-bound workload is a computation that spends the majority of its time executing CPU instructions rather than waiting for I/O — such as image resizing, cryptographic hashing, compression, or complex calculations.

**Technical Definition:** CPU-bound work runs synchronously on the main thread by default in Node.js. Unlike I/O-bound work, which can be offloaded to the OS, CPU-bound work must run on a thread. The `worker_threads` module creates additional JavaScript execution contexts, each with its own V8 isolate and event loop, allowing CPU-bound computation to run in parallel without blocking the main thread. Worker threads are **not** a replacement for async I/O — they are specifically for CPU-bound work.

**Beginner-Friendly Explanation:** A CPU-bound task is like kneading a huge batch of dough by hand. You can't just "wait for it to finish" — you have to actively do the work. If the chef (main thread) does the kneading, they can't take orders. A worker thread is a prep cook who kneads the dough in the back kitchen while the chef keeps serving customers.

### Purposes

- To offload heavy computational tasks away from the main thread.
- To keep the event loop responsive while CPU-intensive work runs in parallel.
- To utilise multiple CPU cores for a single application process.
- To prevent CPU-bound operations from freezing health checks and concurrent requests.

### Syntax Rules and Structure

```javascript
// main.js — offload CPU work to a worker
const { Worker } = require('node:worker_threads');

function runHeavyTask(input) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./worker.js', { workerData: input });
    worker.on('message', resolve);
    worker.on('error', reject);
    worker.on('exit', (code) => {
      if (code !== 0) reject(new Error(`Worker stopped with code ${code}`));
    });
  });
}

app.get('/api/heavy', async (req, res) => {
  const result = await runHeavyTask({ numbers: [1, 2, 3] });
  res.json({ result });
});
```

```javascript
// worker.js — CPU-heavy work runs here
const { parentPort, workerData } = require('node:worker_threads');

function expensiveComputation(data) {
  // CPU-intensive loop or calculation
  return data.numbers.reduce((sum, n) => sum + n * n, 0);
}

const result = expensiveComputation(workerData);
parentPort.postMessage(result);
```

| Component | Breakdown |
|-----------|-----------|
| `new Worker('./worker.js')` | Spawns a worker thread running the specified file. |
| `workerData` | Data passed to the worker (structured-clone copied). |
| `parentPort.postMessage()` | Sends the result back to the main thread. |
| `worker.on('message')` | Receives the result. |

**Rules:**
- Use worker threads **only** for CPU-bound work — not for I/O (which the event loop already handles).
- Do **not** create a worker per request — use a pool (e.g., `piscina`).
- `workerData` and messages are **copied**, not shared by reference — large payloads add serialisation overhead.
- Workers have their own V8 isolate — startup costs memory and time.

**Constraints:**
- Worker threads add memory and startup overhead — use a pool for repeated tasks.
- Data must be structured-cloneable (no functions, no class instances with methods).
- Workers cannot share memory by default; use `SharedArrayBuffer` for explicit sharing.

### Annotated Code Example

```javascript
// worker-pool-example.js
const express = require('express');
const { Worker } = require('node:worker_threads');
const app = express();

// Simple worker pool
class WorkerPool {
  constructor(workerPath, poolSize) {
    this.workers = [];
    this.queue = [];
    for (let i = 0; i < poolSize; i++) {
      this.workers.push(this.createWorker(workerPath));
    }
  }
  createWorker(path) {
    return { worker: new Worker(path), busy: false };
  }
  run(data) {
    return new Promise((resolve, reject) => {
      const available = this.workers.find(w => !w.busy);
      if (!available) {
        this.queue.push({ data, resolve, reject });
        return;
      }
      this.execute(available, data, resolve, reject);
    });
  }
  execute(entry, data, resolve, reject) {
    entry.busy = true;
    entry.worker.once('message', (result) => {
      entry.busy = false;
      resolve(result);
      if (this.queue.length > 0) {
        const next = this.queue.shift();
        this.execute(entry, next.data, next.resolve, next.reject);
      }
    });
    entry.worker.once('error', reject);
    entry.worker.postMessage(data);
  }
}

const pool = new WorkerPool('./cpu-worker.js', 4);

app.get('/api/process', async (req, res) => {
  const result = await pool.run({ input: req.query.input });
  res.json({ result });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/process?input=100`):**
```json
{ "result": 5050 }
```

**Why this output:** The worker pool keeps four worker threads alive. The main thread receives the request, delegates the CPU work to an available worker, and awaits the result. The event loop remains free to handle other requests while the worker computes.

### Real-World Cases

- **Image processing:** Resizing, cropping, and watermarking uploaded images.
- **Cryptographic hashing:** Argon2, bcrypt, or scrypt password hashing.
- **Compression:** Gzip/Brotli compression of large payloads.
- **Report generation:** Generating PDFs or Excel files with thousands of rows.
- **Data parsing:** Parsing large CSV, XML, or JSON files.

---

## Core Concept 3: I/O-Bound Workloads

### Definitions

**Core Definition:** An I/O-bound workload is a task that spends most of its time waiting for input/output operations — network requests, disk reads, database queries — rather than executing CPU instructions.

**Technical Definition:** Node.js's event loop is specifically optimised for I/O-bound concurrency. When an async I/O operation is initiated (e.g., `fs.promises.readFile`, `fetch`, a database query), libuv hands the operation to the OS or its threadpool and registers a callback. The event loop continues processing other work. When the I/O completes, the callback is queued and executed in the appropriate phase. This model allows a single Node.js process to handle thousands of concurrent I/O operations without spawning threads per request.

**Beginner-Friendly Explanation:** I/O-bound work is like waiting for water to boil. You don't stand and stare at the pot — you chop vegetables, prep the sauce, and set the table. The water boils while you do other things. In Node.js, the event loop doesn't wait for a database query or an HTTP request — it starts the I/O and moves on to the next task.

### Purposes

- To maximise Node's high-throughput architecture via non-blocking network, disk, and database reads.
- To achieve massive I/O concurrency without thread-per-request overhead.
- To overlap I/O waits with other request processing.
- To use streams for large I/O without loading entire payloads into memory.

### Syntax Rules and Structure

```javascript
// I/O-bound concurrency
app.get('/api/parallel', async (req, res) => {
  // All three I/O operations start concurrently
  const [users, orders, products] = await Promise.all([
    db.query('SELECT * FROM users'),
    db.query('SELECT * FROM orders'),
    db.query('SELECT * FROM products')
  ]);
  res.json({ users, orders, products });
});
```

| Approach | Concurrency Model | Best For |
|----------|-------------------|----------|
| Async I/O | Event loop + libuv | Network, disk, database. |
| Worker threads | Separate V8 isolates | CPU-bound computation. |
| Cluster | Multiple processes | Multi-core scaling. |

**Rules:**
- Use async I/O for all network, disk, and database operations.
- Use `Promise.all()` to overlap independent I/O operations.
- Use streams for large file or network payloads.
- Do **not** wrap I/O operations in worker threads — it adds overhead without benefit.

**Constraints:**
- The libuv threadpool has a default size of 4 — DNS lookups, `fs` operations, and some crypto operations compete for these threads.
- I/O concurrency is bounded by the database connection pool and file descriptor limits.

### Annotated Code Example

```javascript
// io-bound-example.js
const express = require('express');
const fs = require('fs/promises');
const app = express();

// Sequential I/O — total time is the sum
app.get('/sequential', async (req, res) => {
  const start = Date.now();
  const file1 = await fs.readFile('data1.json', 'utf8');
  const file2 = await fs.readFile('data2.json', 'utf8');
  const file3 = await fs.readFile('data3.json', 'utf8');
  res.json({ duration: Date.now() - start, files: 3 });
});

// Parallel I/O — total time is the max
app.get('/parallel', async (req, res) => {
  const start = Date.now();
  const [file1, file2, file3] = await Promise.all([
    fs.readFile('data1.json', 'utf8'),
    fs.readFile('data2.json', 'utf8'),
    fs.readFile('data3.json', 'utf8')
  ]);
  res.json({ duration: Date.now() - start, files: 3 });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `/sequential`):**
```json
{ "duration": 45, "files": 3 }
```

**Expected Output (for `/parallel`):**
```json
{ "duration": 18, "files": 3 }
```

**Why this output:** Sequential I/O waits for each file read to complete before starting the next (3 × 15ms = 45ms). Parallel I/O starts all three reads simultaneously and waits for the longest one (18ms). The event loop stays free to handle other requests during the I/O waits.

### Real-World Cases

- **API gateways:** Aggregating responses from multiple microservices.
- **Database queries:** Parallel queries for independent data sets.
- **File processing:** Reading multiple files concurrently.
- **Web scraping:** Fetching multiple URLs in parallel with concurrency limits.

---

## Core Concept 4: Event-Loop Starvation

### Definitions

**Core Definition:** Event-loop starvation occurs when synchronous JavaScript execution or a runaway microtask chain prevents the event loop from advancing, causing all pending timers, I/O callbacks, and incoming requests to be indefinitely delayed.

**Technical Definition:** Starvation can occur in two ways: (1) a long-running synchronous block (e.g., a 5-second CPU loop) occupies the main thread, or (2) a microtask chain (Promise callbacks scheduled by Promise callbacks) grows without yielding to the macrotask queue. Because microtasks are processed to completion before the event loop advances to the next phase, an infinite microtask chain starves I/O and timers entirely. The `perf_hooks.monitorEventLoopDelay()` API measures loop lag by sampling how late the event loop wakes up when it should be idle. Values are in nanoseconds; divide by 1e6 for milliseconds.

**Beginner-Friendly Explanation:** Event-loop starvation is like a customer at a service counter who keeps asking "one more question" — and the next customer never gets served. Or it's like a cashier who refuses to take a break, causing the entire queue to wait indefinitely. In Node.js, the "one more question" is a microtask that schedules another microtask, and the "queue" is all the pending HTTP requests.

### Purposes

- To understand how heavy loops or massive microtask execution can completely freeze incoming HTTP traffic.
- To measure loop lag and quantify starvation.
- To detect the two failure modes: blocked loop (CPU at 100%) vs. idle loop (CPU near 0%).
- To chunk CPU work and yield control back to the loop between batches.

### Syntax Rules and Structure

```javascript
// Chunked CPU work — yields to the event loop between batches
async function processLargeArray(items, batchSize = 100) {
  const results = [];
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    results.push(...batch.map(item => heavyTransform(item)));
    // Yield to the event loop between batches
    await new Promise(resolve => setImmediate(resolve));
  }
  return results;
}
```

| Metric | Healthy | Warning | Critical |
|--------|---------|---------|----------|
| p50 loop delay | < 5ms | 5–20ms | > 20ms |
| p99 loop delay | < 20ms | 20–100ms | > 100ms |
| Max loop delay | < 50ms | 50–200ms | > 200ms |

**Rules:**
- Chunk CPU work with `await setImmediate()` between batches.
- Monitor loop delay with `monitorEventLoopDelay` and export p50, p99, and max.
- Distinguish between **blocked** (high CPU, high lag) and **idle** (low CPU, high lag) starvation.
- Use `--cpu-prof` or `--inspect` to identify the blocking function.

**Constraints:**
- `monitorEventLoopDelay` samples at a resolution — it can miss very short blocks.
- Idle starvation (unsettled promises) is harder to diagnose — the stack is empty.

### Annotated Code Example

```javascript
// starvation-monitor.js
const { monitorEventLoopDelay } = require('node:perf_hooks');

const histogram = monitorEventLoopDelay({ resolution: 20 });
histogram.enable();

// Report loop lag every 5 seconds
setInterval(() => {
  const p50 = Math.round(histogram.mean / 1e6);
  const p99 = Math.round(histogram.percentile(99) / 1e6);
  const max = Math.round(histogram.max / 1e6);

  if (p99 > 100) {
    console.warn(`HIGH LOOP LAG — p99: ${p99}ms, max: ${max}ms`);
  } else {
    console.log(`Loop lag — p50: ${p50}ms, p99: ${p99}ms, max: ${max}ms`);
  }

  histogram.reset();
}, 5_000);

// ❌ BAD: Microtask chain that starves the loop
function starveMicrotasks() {
  Promise.resolve().then(() => {
    // This schedules another microtask, which schedules another...
    return starveMicrotasks();
  });
}

// ✅ GOOD: Chunked with setImmediate — yields to macrotask queue
async function processItems(items) {
  for (let i = 0; i < items.length; i++) {
    processItem(items[i]);
    if (i % 100 === 0) {
      await new Promise(resolve => setImmediate(resolve));
    }
  }
}
```

**Expected Output (healthy):**
```
Loop lag — p50: 2ms, p99: 8ms, max: 15ms
```

**Expected Output (starved):**
```
HIGH LOOP LAG — p99: 450ms, max: 1200ms
```

**Why this output:** The monitor samples how late the event loop wakes up. When microtasks or synchronous code monopolise the thread, the histogram records increasing delay. The p99 and max values indicate the severity of the starvation.

### Real-World Cases

- **Health checks failing:** The load balancer marks the service unhealthy because health check responses are delayed by a blocked loop.
- **Latency spikes:** All concurrent requests experience increased latency when one handler blocks the loop.
- **Microtask storms:** A recursive `Promise.resolve().then()` chain starves I/O and timers.

---

## Core Concept 5: Worker Threads & Clustering

### Definitions

**Core Definition:** `worker_threads` creates additional JavaScript execution contexts within a single process for CPU-bound parallelism; the `cluster` module and PM2 create multiple processes across CPU cores for horizontal scaling of I/O-bound servers.

**Technical Definition:** A **worker thread** is a real operating-system thread that runs JavaScript in parallel with the main thread, with its own V8 isolate and event loop. Workers communicate with the main thread via message passing (`postMessage`/`on('message')`), and data is copied using the structured-clone algorithm. The **cluster module** spawns multiple worker **processes** that share the same server port. The master process accepts connections and distributes them using a round-robin algorithm (except on Windows, where it uses the OS scheduler). PM2 wraps the cluster module and manages process lifecycle, restart on crash, and zero-downtime reloads.

**Beginner-Friendly Explanation:** Worker threads are like hiring prep cooks who work in the same kitchen (process) but at different stations. Clustering is like opening multiple restaurant branches (processes) in the same city, all serving the same menu and sharing the same phone number (port). Worker threads handle CPU-heavy tasks; clustering handles more customers.

### Purposes

- To implement the `worker_threads` module for processing heavy in-process calculations.
- To leverage the cluster module or PM2 to scale Express instances across multiple CPU cores.
- To achieve CPU parallelism within a single process (workers).
- To achieve I/O concurrency across processes (cluster).

### Syntax Rules and Structure

#### Worker Threads vs. Cluster

| Dimension | `worker_threads` | `cluster` / PM2 |
|-----------|------------------|------------------|
| Unit | Thread (same process) | Process (separate memory) |
| Best for | CPU-bound work | I/O-bound servers |
| Memory | Shared process memory | Isolated per process |
| Communication | Message passing | IPC / shared port |
| Startup cost | Lower | Higher |

#### Cluster Example

```javascript
// cluster-app.js
const cluster = require('node:cluster');
const http = require('node:http');
const os = require('node:os');

const numCPUs = os.cpus().length;

if (cluster.isPrimary) {
  console.log(`Primary ${process.pid} running`);
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died — restarting`);
    cluster.fork();
  });
} else {
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end(`Handled by worker ${process.pid}`);
  }).listen(8080);
  console.log(`Worker ${process.pid} started`);
}
```

#### PM2 Example

```bash
# Start with one instance per CPU core
pm2 start app.js -i max

# Or specify the number of instances
pm2 start app.js -i 4

# Monitor
pm2 monit

# Zero-downtime reload
pm2 reload app
```

**Rules:**
- Use worker threads for CPU-bound work; use cluster for I/O-bound HTTP servers.
- Do not create a worker per request — use a pool (`piscina`).
- PM2's `-i max` spawns one instance per CPU core.
- Cluster workers do not share memory — use Redis or a database for shared state.
- `cluster.fork()` restarts crashed workers automatically if an `exit` handler is registered.

**Constraints:**
- Worker threads cannot share memory by default — use `SharedArrayBuffer` for explicit sharing.
- Cluster workers cannot share in-memory sessions — use sticky sessions or a shared store.
- PM2 adds operational complexity but simplifies process management.

### Annotated Code Example

```javascript
// hybrid-app.js — workers for CPU, cluster for I/O
const cluster = require('node:cluster');
const { Worker } = require('node:worker_threads');
const express = require('express');
const os = require('node:os');

const numCPUs = os.cpus().length;

if (cluster.isPrimary) {
  for (let i = 0; i < numCPUs; i++) cluster.fork();
} else {
  const app = express();

  // I/O-bound route — handled by the event loop
  app.get('/api/data', async (req, res) => {
    const data = await fetchData();
    res.json({ data, worker: process.pid });
  });

  // CPU-bound route — offloaded to a worker thread
  app.get('/api/compute', async (req, res) => {
    const result = await runInWorker('./cpu-worker.js', { input: req.query.n });
    res.json({ result, worker: process.pid });
  });

  app.listen(3000);
  console.log(`Worker ${process.pid} listening on 3000`);
}

function runInWorker(path, data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker(path, { workerData: data });
    worker.on('message', resolve);
    worker.on('error', reject);
  });
}
```

**Expected Output (for `GET /api/compute?n=100`):**
```json
{ "result": 5050, "worker": 12345 }
```

**Why this output:** The cluster spawns one process per CPU core. Each process runs an Express server. I/O-bound requests are handled by the event loop. CPU-bound requests are delegated to a worker thread within the process. The main thread of each process stays responsive.

### Real-World Cases

- **Video encoding:** Worker threads for FFmpeg-style CPU work; cluster for the API.
- **API servers:** PM2 cluster mode for multi-core scaling.
- **Data processing:** Worker pools for parallel data transformation.
- **High-traffic APIs:** Cluster mode with Nginx or a load balancer in front.

---

## Core Concept 6: Asynchronous Context Tracking

### Definitions

**Core Definition:** `AsyncLocalStorage` is a Node.js API that creates stores that stay coherent through asynchronous operations, allowing state to be associated with a web request and propagated through callbacks, Promises, and async/await chains — similar to thread-local storage in other languages.

**Technical Definition:** `AsyncLocalStorage` is part of the `node:async_hooks` module. It creates a store that is automatically available in every callback and Promise chain originating from the `asyncLocalStorage.run()` call. `asyncLocalStorage.getStore()` retrieves the current store; `asyncLocalStorage.run(store, callback)` runs the callback within a new context. This eliminates the need to manually pass request IDs, user objects, or tenant identifiers through every function signature. It is the recommended approach for request-scoped context in production, being performant and memory-safe.

**Beginner-Friendly Explanation:** `AsyncLocalStorage` is like a name tag that follows you everywhere in a convention centre. When you enter (a request arrives), you put on your name tag (the store). Every conversation you have (every function call) can see your name tag without you having to show it manually. When you leave (the request ends), the tag is discarded.

### Purposes

- To maintain execution context across complex async boundaries.
- To propagate request IDs through deep service layers for logging and tracing.
- To implement multi-tenant isolation without passing tenant IDs manually.
- To provide request-scoped dependencies (e.g., a user object) to any function.

### Syntax Rules and Structure

```typescript
import { AsyncLocalStorage } from 'node:async_hooks';

const asyncLocalStorage = new AsyncLocalStorage<Map<string, string>>();

// Middleware — sets the store for each request
app.use((req, res, next) => {
  const store = new Map();
  store.set('requestId', req.headers['x-request-id'] || randomUUID());
  store.set('userId', req.user?.id);

  asyncLocalStorage.run(store, () => {
    next(); // All downstream code sees this store
  });
});

// Anywhere in the application — access the store
function getRequestId(): string | undefined {
  return asyncLocalStorage.getStore()?.get('requestId');
}

// Logger automatically includes request ID
function log(message: string) {
  console.log(`[${getRequestId()}] ${message}`);
}
```

| Component | Breakdown |
|-----------|-----------|
| `new AsyncLocalStorage()` | Creates a new context store. |
| `asyncLocalStorage.run(store, fn)` | Runs `fn` within the context of `store`. |
| `asyncLocalStorage.getStore()` | Retrieves the current context store. |

**Rules:**
- Create the `AsyncLocalStorage` instance **once** at module scope.
- Wrap the request handler in `asyncLocalStorage.run()` as early as possible.
- Use `getStore()` anywhere — no manual parameter passing required.
- `AsyncLocalStorage` is stable since Node.js v16.4.0.
- Works across Promises, `async/await`, timers, and I/O callbacks.

**Constraints:**
- Adds a small overhead per async operation — benchmark in high-throughput services.
- The store is not available outside the `run()` callback.
- Context is not automatically propagated to worker threads — pass it explicitly.

### Annotated Code Example

```typescript
// context.ts
import { AsyncLocalStorage } from 'node:async_hooks';

interface RequestContext {
  requestId: string;
  userId?: string;
  tenantId?: string;
}

export const asyncLocalStorage = new AsyncLocalStorage<RequestContext>();

export function getContext(): RequestContext | undefined {
  return asyncLocalStorage.getStore();
}
```

```typescript
// middleware/context.ts
import { randomUUID } from 'node:crypto';
import { asyncLocalStorage } from '../context';

export function contextMiddleware(req, res, next) {
  const context: RequestContext = {
    requestId: req.headers['x-request-id'] || randomUUID(),
    userId: req.user?.id,
    tenantId: req.headers['x-tenant-id']
  };

  res.setHeader('x-request-id', context.requestId);

  asyncLocalStorage.run(context, () => {
    next();
  });
}
```

```typescript
// services/payment.service.ts
import { getContext } from '../context';
import { logger } from '../logger';

export class PaymentService {
  async charge(userId: string, amount: number) {
    const ctx = getContext();
    logger.info('Charging payment', {
      requestId: ctx?.requestId,
      userId,
      amount
    });
    // ... payment logic
  }
}
```

**Expected Output (log entry):**
```json
{
  "level": "info",
  "message": "Charging payment",
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "userId": "usr_456",
  "amount": 99.99
}
```

**Why this output:** The `contextMiddleware` creates a `RequestContext` and runs the rest of the request within `asyncLocalStorage.run()`. The `PaymentService` calls `getContext()` to retrieve the request ID and user ID without receiving them as parameters. Every log line within that request automatically includes the `requestId`.

### Real-World Cases

- **Request tracing:** Propagating request IDs through all logs and downstream services.
- **Multi-tenant isolation:** Storing the tenant ID and using it in database queries.
- **Audit logging:** Attaching the authenticated user to every audit event.
- **Feature flags:** Storing the user's feature flag set in context for quick access.

---

## References

- Node.js — Asynchronous context tracking (`AsyncLocalStorage`) — https://nodejs.org/api/async_context.html
- Node.js — `worker_threads` — https://nodejs.org/api/worker_threads.html
- Node.js — `cluster` — https://nodejs.org/api/cluster.html
- Node.js — `perf_hooks` (`monitorEventLoopDelay`) — https://nodejs.org/api/perf_hooks.html
- Async Survival: Defusing Event Loop Blocking — Pluralsight — https://www.pluralsight.com/labs/codeLabs/async-survival-defusing-event-loop-blocking
- How to Fix 'Event Loop Blocking' Issues — OneUptime — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-24-event-loop-blocking/README.md
- Finding event-loop stalls in Node before your users do — DEV Community — https://dev.to/yaseenyk04/finding-event-loop-stalls-in-node-before-your-users-do-4k21
- Worker Threads in Node.js: How They Work Safely — safeguard.sh — https://safeguard.sh/resources/blog/worker-thread-in-node-js
- Node.js clustering made easy with PM2 — PM2 — https://pm2.io/blog/2018/04/20/Node-js-clustering-made-easy-with-PM2
- Node.js performance optimization: 12 ways to speed up apps — Hostinger — https://www.hostinger.com/tutorials/nodejs-performance-optimization
- Node.js Performance Monitoring: What to Track and How to Fix It — Scout Monitoring — https://www.scoutapm.com/blog/nodejs-performance-monitoring
- Node.js Performance and Scaling: Production Checklist (2026) — WFNext — https://wfnext.com/nodejs-performance-scaling-checklist
- piscina — npm — https://www.npmjs.com/package/piscina
- BullMQ — Concurrency — https://docs.bullmq.io/guide/workers/concurrency