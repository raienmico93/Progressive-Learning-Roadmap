# First Express Application

This guide walks through creating your first Express application — from instantiating the core app object to understanding the full lifecycle of a request, from TCP packet to final response.

---

## 1. Create an Express Application

### 1.1 Instantiating the Core Application Object

Every Express application begins by calling the `express()` function, which returns an **application object**.

```javascript
// src/app.js
import express from 'express';

const app = express();

export default app;
```

That single call is deceptively simple. Behind the scenes, `express()` creates a **function** that:

- Is callable as a request handler: `app(req, res)`
- Has methods attached: `app.get()`, `app.post()`, `app.use()`, `app.listen()`
- Maintains internal state: middleware stack, settings, routes

**The app object is a function with properties.** You can pass it directly to Node's `http.createServer()`:

```javascript
import http from 'http';
import app from './app.js';

const server = http.createServer(app);
server.listen(3000);
```

This is exactly what `app.listen()` does internally.

### 1.2 Understanding Singleton Behavior

Each call to `express()` returns a **new, independent application instance**:

```javascript
const app1 = express();
const app2 = express();

app1.get('/', (req, res) => res.send('App 1'));
app2.get('/', (req, res) => res.send('App 2'));

// app1 and app2 are separate — no shared routes
```

**However**, within a single file, `app` behaves like a **singleton** because it is created once and reused:

```javascript
// app.js — created ONCE
import express from 'express';
const app = express();          // ← single instance
app.use(express.json());
app.use('/api', routes);
export default app;             // ← exported for reuse
```

**Why this matters:**

| Pattern | Behavior |
|---|---|
| `import app from './app.js'` in multiple files | All receive the **same** instance |
| Calling `express()` again in a different file | Creates a **new** instance |
| Node.js module cache | Ensures the module (and its `app`) is instantiated only once |

**Practical implication:** Middleware registered in `app.js` is visible everywhere the app is imported — because it is the same object.

### 1.3 Full Minimal Application

```javascript
// src/server.js
import express from 'express';

const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.send('Hello, Express!');
});

app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});
```

Run it:

```bash
node src/server.js
```

Visit `http://localhost:3000` in a browser, or test with `curl`:

```bash
curl http://localhost:3000
# Hello, Express!
```

That's a complete Express application in 10 lines.

---

## 2. Start an HTTP Server

### 2.1 `app.listen()` vs. `http.createServer()`

`app.listen()` is a convenience wrapper:

```javascript
app.listen(PORT, callback);
```

is equivalent to:

```javascript
import http from 'http';
const server = http.createServer(app);
server.listen(PORT, callback);
```

**`app.listen()` returns the underlying `http.Server` instance**, so you can still access server-level methods:

```javascript
const server = app.listen(3000);
server.on('error', handleError);
server.close(() => console.log('Closed'));
```

### 2.2 Binding to Specific Interfaces

By default, `app.listen(PORT)` binds to **all interfaces** (`0.0.0.0` and `::`). To restrict:

```javascript
// Localhost only — safe for development
app.listen(PORT, '127.0.0.1', () => {
  console.log(`Server running at http://127.0.0.1:${PORT}`);
});

// All interfaces — required in containers
app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on all interfaces at port ${PORT}`);
});
```

| Binding | Reachable From | Use Case |
|---|---|---|
| `127.0.0.1` | Local machine only | Local development |
| `0.0.0.0` | Any network interface | Containers, cloud servers |
| Specific IP | That interface only | Multi-homed hosts |
| `localhost` | Resolves to `127.0.0.1` or `::1` | Local development |

**Security tip:** Binding to `0.0.0.0` in development exposes the app to anyone on your local network (e.g., coffee shop Wi-Fi).

### 2.3 Handling Startup Event Callbacks

The callback passed to `app.listen()` fires **once**, when the server is bound and ready:

```javascript
const server = app.listen(PORT, '0.0.0.0', () => {
  const addr = server.address();
  console.log(`Server listening on ${addr.address}:${addr.port}`);
});
```

**The `listening` event** is equivalent:

```javascript
server.on('listening', () => {
  console.log('Server is ready');
});
```

### 2.4 Handling Startup Errors

Startup can fail for many reasons — port in use, permission denied, invalid host.

```javascript
server.on('error', (err) => {
  if (err.code === 'EADDRINUSE') {
    console.error(`Port ${PORT} is already in use`);
  } else if (err.code === 'EACCES') {
    console.error(`Port ${PORT} requires elevated privileges`);
  } else {
    console.error('Server error:', err);
  }
  process.exit(1);
});
```

### 2.5 Dynamic Port Allocation

Cloud platforms assign ports dynamically via `process.env.PORT`:

```javascript
const PORT = process.env.PORT || 3000;

app.listen(PORT, '0.0.0.0', () => {
  console.log(`Listening on port ${PORT}`);
});
```

**Two rules for cloud/container environments:**

1. Always read `process.env.PORT`.
2. Always bind to `0.0.0.0`.

Binding to `localhost` inside a container makes the app unreachable from outside.

### 2.6 Graceful Shutdown

```javascript
process.on('SIGTERM', () => {
  console.log('SIGTERM received — shutting down gracefully');
  server.close(() => {
    console.log('HTTP server closed');
    process.exit(0);
  });
});
```

Graceful shutdown matters in production — it lets in-flight requests finish before the process exits.

---

## 3. Define a Route

### 3.1 Basic Route Structure

```javascript
app.METHOD(PATH, HANDLER);
```

| Part | Description |
|---|---|
| `app` | The Express application object |
| `METHOD` | HTTP verb: `get`, `post`, `put`, `patch`, `delete`, `all` |
| `PATH` | URL pattern string, regex, or array |
| `HANDLER` | Function `(req, res, next)` or array of functions |

### 3.2 Implementing Basic Root Routes

```javascript
app.get('/', (req, res) => {
  res.send('Welcome to the homepage');
});

app.get('/about', (req, res) => {
  res.send('About us');
});

app.get('/contact', (req, res) => {
  res.send('Contact page');
});
```

### 3.3 Route Methods

Express supports all standard HTTP methods:

```javascript
app.get('/users', (req, res) => { /* list users */ });
app.post('/users', (req, res) => { /* create user */ });
app.put('/users/:id', (req, res) => { /* replace user */ });
app.patch('/users/:id', (req, res) => { /* update user */ });
app.delete('/users/:id', (req, res) => { /* delete user */ });

app.all('/secret', (req, res) => {
  res.send('Every HTTP method reaches here');
});
```

### 3.4 Testing with `curl`

**Basic GET:**

```bash
curl http://localhost:3000/
# Welcome to the homepage
```

**Inspect headers:**

```bash
curl -i http://localhost:3000/
```

Output:

```
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 20
ETag: W/"14-..."
Date: Wed, 01 Oct 2026 12:00:00 GMT
Connection: keep-alive

Welcome to the homepage
```

**POST request:**

```bash
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John"}'
```

**PUT with JSON:**

```bash
curl -X PUT http://localhost:3000/users/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"Johnny"}'
```

**Verbose mode (see the whole exchange):**

```bash
curl -v http://localhost:3000/
```

**Test multiple endpoints:**

```bash
for path in / /about /contact; do
  echo "=== $path ==="
  curl -s http://localhost:3000$path
  echo
done
```

**Other terminal tools:**

| Tool | Command |
|---|---|
| `wget` | `wget -qO- http://localhost:3000/` |
| `httpie` | `http GET localhost:3000/` |
| `fetch` (Node) | `node -e "fetch('http://localhost:3000/').then(r=>r.text()).then(console.log)"` |

---

## 4. Send a Response

### 4.1 `res.send()` vs. `res.end()` vs. `res.write()`

Express provides high-level response methods that abstract the underlying stream.

| Method | Behavior |
|---|---|
| `res.send(body)` | Sends body, sets `Content-Type`, sets `Content-Length`, ends response |
| `res.json(obj)` | Serializes to JSON, sets `Content-Type: application/json`, ends |
| `res.end([data])` | Node native — ends response, no content-type detection |
| `res.write(chunk)` | Node native — writes a chunk without ending |

**Example:**

```javascript
// High-level (Express)
res.send('Hello');

// Low-level (Node native, still available)
res.setHeader('Content-Type', 'text/plain');
res.end('Hello');
```

### 4.2 Plain Text vs. HTML Strings

`res.send()` **guesses** the content type based on the argument:

| Argument Type | Content-Type Set |
|---|---|
| String containing HTML tags | `text/html; charset=utf-8` |
| String without HTML | `text/html; charset=utf-8` (default) |
| Buffer | `application/octet-stream` |
| Object / Array | `application/json; charset=utf-8` |
| Number / Boolean | `text/html; charset=utf-8` |

**Plain text — explicit:**

```javascript
res.type('text/plain').send('Hello, world');
// Content-Type: text/plain; charset=utf-8
```

**HTML string:**

```javascript
res.send('<h1>Hello, world</h1>');
// Content-Type: text/html; charset=utf-8
```

**JSON object:**

```javascript
res.send({ message: 'Hello' });
// Content-Type: application/json; charset=utf-8

// Equivalent to:
res.json({ message: 'Hello' });
```

### 4.3 Automatic Content-Type Guessing

Express uses `res.send()`'s logic to infer content type. **When in doubt, be explicit.**

```javascript
// Ambiguous — Express guesses "text/html"
res.send('{"a":1}');
// Content-Type: text/html  ← WRONG if you meant JSON

// Explicit — you control the type
res.type('application/json').send('{"a":1}');
// Content-Type: application/json  ← correct
```

### 4.4 Setting Status Codes

```javascript
res.status(201).json({ id: 1, name: 'John' });
res.status(404).send('Not found');
res.sendStatus(204);  // sends "No Content" with status 204
```

### 4.5 Chaining

Response methods return `res`, so they chain:

```javascript
res
  .status(200)
  .type('text/plain')
  .set('X-Custom', 'value')
  .send('Hello');
```

### 4.6 Common Response Methods

| Method | Purpose |
|---|---|
| `res.send(body)` | Send body with inferred content-type |
| `res.json(obj)` | Send JSON response |
| `res.jsonp(obj)` | Send JSONP response |
| `res.sendFile(path)` | Send a file |
| `res.download(path)` | Prompt download |
| `res.redirect([status], url)` | Redirect |
| `res.render(view, data)` | Render a template |
| `res.status(code)` | Set status code |
| `res.set(header, value)` | Set header |
| `res.type(mime)` | Set Content-Type |
| `res.cookie(name, value)` | Set cookie |
| `res.clearCookie(name)` | Clear cookie |
| `res.end()` | End response (Node native) |

---

## 5. Understand Request and Response Objects

### 5.1 `req` — The Incoming Request

`req` is an **enhanced version of Node's `http.IncomingMessage`**, with Express-specific properties and methods added.

**Inherited from Node's `IncomingMessage`:**

| Property / Method | Description |
|---|---|
| `req.headers` | Request headers object |
| `req.method` | HTTP method |
| `req.url` | Full URL (path + query) |
| `req.httpVersion` | HTTP version |
| `req.socket` | Underlying TCP socket |
| `req.on('data', cb)` | Stream data events |
| `req.on('end', cb)` | Stream end event |

**Added by Express:**

| Property / Method | Description |
|---|---|
| `req.params` | Route parameters (`/users/:id` → `{ id: '1' }`) |
| `req.query` | Parsed query string (`?a=1` → `{ a: '1' }`) |
| `req.body` | Parsed body (requires middleware) |
| `req.cookies` | Parsed cookies (requires `cookie-parser`) |
| `req.path` | URL path without query |
| `req.hostname` | Host header value |
| `req.ip` | Client IP address |
| `req.protocol` | `http` or `https` |
| `req.secure` | Boolean — true if HTTPS |
| `req.get(header)` | Get a request header |
| `req.is(type)` | Check Content-Type |
| `req.accepts(type)` | Check Accept header |

**Example:**

```javascript
app.get('/users/:id', (req, res) => {
  console.log(req.params.id);        // route parameter
  console.log(req.query.page);       // query string
  console.log(req.headers['user-agent']);
  console.log(req.method);           // "GET"
  console.log(req.path);             // "/users/42"
  console.log(req.ip);
  res.json({ id: req.params.id });
});
```

### 5.2 `req` as an Incoming Stream Wrapper

`req` **is a readable stream** — Node's `IncomingMessage` is a `Readable`. This means:

```javascript
app.post('/upload', (req, res) => {
  const chunks = [];
  req.on('data', (chunk) => chunks.push(chunk));
  req.on('end', () => {
    const body = Buffer.concat(chunks).toString();
    res.send(`Received ${body.length} bytes`);
  });
});
```

This is the **raw streaming interface**. Express middleware (like `express.json()`) consumes this stream and attaches the parsed result to `req.body`.

**Key stream properties:**

| Property | Meaning |
|---|---|
| `req.readable` | Is the stream still readable? |
| `req.readableEnded` | Has the stream ended? |
| `req.complete` | Has the full message been received? |
| `req.pipe(dest)` | Pipe to a writable stream |
| `req.destroy()` | Abort the request |

**Example — streaming a large upload to disk:**

```javascript
import fs from 'fs';

app.post('/upload', (req, res) => {
  const stream = fs.createWriteStream('./upload.bin');
  req.pipe(stream);
  stream.on('finish', () => res.send('Uploaded'));
});
```

### 5.3 `res` — The Outgoing Response

`res` is an **enhanced version of Node's `http.ServerResponse`**, with Express-specific helpers.

**Inherited from Node's `ServerResponse`:**

| Property / Method | Description |
|---|---|
| `res.writeHead(status, headers)` | Write status + headers |
| `res.setHeader(name, value)` | Set a header |
| `res.getHeader(name)` | Read a header |
| `res.write(chunk)` | Write a chunk |
| `res.end([data])` | End the response |
| `res.statusCode` | Status code |

**Added by Express:**

| Property / Method | Description |
|---|---|
| `res.send(body)` | Send body with inference |
| `res.json(obj)` | Send JSON |
| `res.status(code)` | Set status |
| `res.set(field, value)` | Set header(s) |
| `res.type(mime)` | Set Content-Type |
| `res.redirect(url)` | Redirect |
| `res.render(view, data)` | Render template |
| `res.locals` | Per-request data for templates |
| `res.headersSent` | Boolean — are headers already sent? |

### 5.4 `res` as an Outgoing Stream Wrapper

`res` **is a writable stream** — Node's `ServerResponse` is `Writable`. This enables streaming responses:

```javascript
app.get('/stream', (req, res) => {
  res.type('text/plain');
  res.write('chunk 1\n');
  res.write('chunk 2\n');
  res.write('chunk 3\n');
  res.end();
});
```

**Piping a file:**

```javascript
import fs from 'fs';

app.get('/video', (req, res) => {
  res.type('video/mp4');
  fs.createReadStream('./video.mp4').pipe(res);
});
```

### 5.5 `req` and `res` Are Streams — Why It Matters

| Feature | Benefit |
|---|---|
| **Backpressure** | Node manages slow consumers automatically |
| **Memory efficiency** | Large files don't need to fit in memory |
| **Composability** | Pipe streams together with `.pipe()` |
| **Real-time** | Server-Sent Events, chunked transfer |

Express's `res.send()` buffers the whole body in memory before sending. For large payloads, prefer `res.write()` / `res.end()` or `.pipe()`.

### 5.6 `req` and `res` Are Per-Request

Each incoming request creates **fresh `req` and `res` objects**. They are **not shared** across requests.

```javascript
app.get('/', (req, res) => {
  console.log(req === previousReq); // false — new object each time
});
```

**Never store `req` or `res` in module-level variables** — that causes memory leaks and cross-request contamination.

---

## 6. Understand Application Lifecycle

The Express application lifecycle has five phases.

### Phase 1 — Initialization

```javascript
const app = express();          // create app
app.use(express.json());        // register middleware
app.use('/api', routes);        // mount routers
app.use(errorHandler);          // register error handler
```

At this point, no server is running. The app is a **middleware stack + router** in memory.

### Phase 2 — Binding

```javascript
const server = app.listen(PORT, HOST, onReady);
```

- Node creates a TCP socket
- Binds to host:port
- Listens for connections
- Emits `listening` event when ready

### Phase 3 — Accepting Connections

For each incoming connection:

1. TCP handshake completes
2. Node's HTTP parser reads the request
3. An `IncomingMessage` (`req`) and `ServerResponse` (`res`) pair are created
4. The request is handed to the Express app

### Phase 4 — Request Processing

The request travels through the **middleware pipeline** and finally a **route handler**.

### Phase 5 — Shutdown

- `server.close()` stops accepting new connections
- In-flight requests finish
- Process exits when the event loop empties
- `SIGTERM` / `SIGINT` handlers enable graceful shutdown

**Full lifecycle diagram:**

```
express() ──► app created
    │
    ▼
app.use(...) ──► middleware stack built
    │
    ▼
app.listen() ──► TCP socket bound, listening
    │
    ▼
connection ──► req/res created
    │
    ▼
middleware chain ──► route handler ──► response sent
    │
    ▼
server.close() ──► graceful shutdown
```

---

## 7. The Journey of an Incoming TCP Packet

This section traces a single HTTP request from wire to response.

### Step 1 — TCP Packet Arrives

A client sends an HTTP request. TCP segments arrive at the network interface, are reassembled by the OS kernel, and placed in the socket's receive buffer.

```
[ Ethernet frame ] → [ IP packet ] → [ TCP segment ] → [ HTTP bytes ]
```

### Step 2 — OS Delivers to Node's Socket

The kernel wakes Node's event loop. The `net.Server` listening on `host:port` receives the connection, and libuv reads the data.

### Step 3 — Node's HTTP Parser

`http_parser` (a C library) parses the raw bytes into:

- Request line (`GET /users/42 HTTP/1.1`)
- Headers (`Host: localhost:3000`, `User-Agent: curl/8.0`)
- Body (if any)

### Step 4 — `IncomingMessage` Created

Node constructs an `http.IncomingMessage` object — this is **`req`** — populated with method, URL, headers, and a readable stream for the body.

### Step 5 — `ServerResponse` Created

Node constructs an `http.ServerResponse` — this is **`res`** — a writable stream ready to send the response.

### Step 6 — Express Receives the Pair

Node calls the request listener registered by `app.listen()`. That listener is the **Express app function**:

```javascript
function app(req, res) {
  app.handle(req, res);
}
```

Express begins processing.

### Step 7 — Express Sets Up `req` and `res`

Express augments `req` and `res` with its own properties and methods:

- `req.params = {}`
- `req.query = {}` (lazy)
- `req.path = parseurl(req).pathname`
- `res.send`, `res.json`, `res.status`, etc.

### Step 8 — Middleware Pipeline

Express walks the **middleware stack** in order:

```javascript
[
  helmet(),
  cors(),
  express.json(),
  router,
  errorHandler
]
```

Each middleware function receives `(req, res, next)` and either:

- Calls `next()` — continue to the next middleware
- Calls `next('route')` — skip to next route
- Calls `next(err)` — jump to error handler
- Sends a response — pipeline ends

### Step 9 — Router Matching

The router (the last "middleware") examines `req.method` and `req.path` against registered routes:

- **Layer 1**: `app.get('/')`
- **Layer 2**: `app.get('/about')`
- **Layer 3**: `app.get('/users/:id')`

When a route matches, Express invokes its handler.

### Step 10 — Route Handler Executes

```javascript
app.get('/users/:id', (req, res) => {
  res.json({ id: req.params.id });
});
```

The handler produces a response.

### Step 11 — Response Construction

`res.json()`:

1. Sets `Content-Type: application/json`
2. Serializes the object to a string
3. Sets `Content-Length`
4. Writes the body to the underlying socket
5. Ends the response

### Step 12 — Writable Stream Flush

`res` (a `Writable` stream) writes the response bytes to the TCP socket. The kernel segments them into packets, which travel back to the client.

### Step 13 — Connection Closes (or Persists)

Under HTTP/1.1, the connection stays open for keep-alive. Under HTTP/2 or HTTP/3, multiple requests share the same connection. Under HTTP/1.0, the connection closes.

### Step 14 — Node's Event Loop Continues

The loop moves to the next request, callback, or timer. Node remains single-threaded for JavaScript; I/O is delegated to libuv.

### Full Pipeline Diagram

```
┌──────────────────────────────────────────────────────────────┐
│  CLIENT                                                      │
│  curl http://localhost:3000/users/42                         │
└───────────────────────────┬──────────────────────────────────┘
                            │ TCP packets
                            ▼
┌──────────────────────────────────────────────────────────────┐
│  OS KERNEL                                                   │
│  Socket receive buffer → wake Node event loop                │
└───────────────────────────┬──────────────────────────────────┘
                            │
                            ▼
┌──────────────────────────────────────────────────────────────┐
│  NODE HTTP SERVER (libuv + http_parser)                      │
│  Parse bytes → IncomingMessage (req) + ServerResponse (res)  │
└───────────────────────────┬──────────────────────────────────┘
                            │ app(req, res)
                            ▼
┌──────────────────────────────────────────────────────────────┐
│  EXPRESS APP                                                 │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ Middleware 1: helmet()        → next()                 │  │
│  │ Middleware 2: cors()          → next()                 │  │
│  │ Middleware 3: express.json()  → next()                 │  │
│  │ Middleware 4: router          → matches /users/:id     │  │
│  └────────────────────────────────────────────────────────┘  │
│                            │                                 │
│                            ▼                                 │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ Route Handler: app.get('/users/:id', (req, res) => {   │  │
│  │   res.json({ id: req.params.id });                     │  │
│  │ });                                                    │  │
│  └────────────────────────────────────────────────────────┘  │
└───────────────────────────┬──────────────────────────────────┘
                            │ res.json(...)
                            ▼
┌──────────────────────────────────────────────────────────────┐
│  HTTP RESPONSE                                               │
│  Status: 200                                                 │
│  Content-Type: application/json                              │
│  Body: {"id":"42"}                                           │
└───────────────────────────┬──────────────────────────────────┘
                            │ TCP packets
                            ▼
┌──────────────────────────────────────────────────────────────┐
│  CLIENT                                                      │
│  {"id":"42"}                                                 │
└──────────────────────────────────────────────────────────────┘
```

---

## Complete First Application

Putting it all together — a first Express application demonstrating every concept:

```javascript
// src/server.js
import express from 'express';

const app = express();
const PORT = process.env.PORT || 3000;
const HOST = process.env.HOST || '127.0.0.1';

// ─── Middleware ──────────────────────────────────────────────
app.use(express.json());          // parse JSON bodies
app.use(express.urlencoded({ extended: true }));  // parse form bodies

// Simple request logger
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next();
});

// ─── Routes ──────────────────────────────────────────────────
app.get('/', (req, res) => {
  res.send('Welcome to the homepage');
});

app.get('/about', (req, res) => {
  res.send('<h1>About Us</h1><p>We build APIs.</p>');
});

app.get('/users/:id', (req, res) => {
  const { id } = req.params;
  const { verbose } = req.query;
  res.json({
    id,
    verbose: verbose === 'true',
    method: req.method,
    path: req.path,
    ip: req.ip,
  });
});

app.post('/echo', (req, res) => {
  res.status(201).json({
    received: req.body,
    contentType: req.get('content-type'),
  });
});

app.get('/stream', (req, res) => {
  res.type('text/plain');
  res.write('Streaming...\n');
  setTimeout(() => {
    res.write('...still streaming...\n');
    setTimeout(() => {
      res.end('Done.\n');
    }, 500);
  }, 500);
});

app.get('/error', (req, res, next) => {
  next(new Error('Something broke'));
});

// ─── 404 Handler ─────────────────────────────────────────────
app.use((req, res) => {
  res.status(404).json({ error: 'Not found', path: req.path });
});

// ─── Error Handler ───────────────────────────────────────────
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: err.message });
});

// ─── Start Server ────────────────────────────────────────────
const server = app.listen(PORT, HOST, () => {
  console.log(`Server listening on http://${HOST}:${PORT}`);
});

server.on('error', (err) => {
  if (err.code === 'EADDRINUSE') {
    console.error(`Port ${PORT} is already in use`);
  } else {
    console.error(err);
  }
  process.exit(1);
});

// ─── Graceful Shutdown ───────────────────────────────────────
process.on('SIGTERM', () => {
  console.log('Shutting down...');
  server.close(() => {
    console.log('Server closed');
    process.exit(0);
  });
});
```

**Test each endpoint:**

```bash
curl http://localhost:3000/
curl http://localhost:3000/about
curl http://localhost:3000/users/42
curl "http://localhost:3000/users/42?verbose=true"
curl -X POST http://localhost:3000/echo \
  -H "Content-Type: application/json" \
  -d '{"hello":"world"}'
curl http://localhost:3000/stream
curl -i http://localhost:3000/error
curl -i http://localhost:3000/missing
```

---

## Summary Table

| Topic | Key Points |
|---|---|
| **App instantiation** | `express()` returns a callable function with methods |
| **Singleton behavior** | One instance per module; new instance per `express()` call |
| **Start server** | `app.listen(PORT, HOST, cb)` wraps `http.createServer(app)` |
| **Binding** | `127.0.0.1` locally; `0.0.0.0` in containers |
| **Startup callback** | Fires once, when bound and ready |
| **Routes** | `app.METHOD(PATH, HANDLER)` |
| **Test with curl** | `curl -i`, `-X POST`, `-d`, `-H` |
| **Send response** | `res.send()`, `res.json()`, `res.end()` |
| **Content-Type** | `res.send()` infers; `res.type()` overrides |
| **`req`** | `IncomingMessage` + Express additions (params, query, body) |
| **`res`** | `ServerResponse` + Express additions (send, json, status) |
| **Streams** | `req` is readable; `res` is writable; both support `.pipe()` |
| **Lifecycle** | Init → Bind → Accept → Process → Shutdown |
| **TCP packet journey** | Kernel → Node parser → `req`/`res` → Express → middleware → route → response |

---

## Key Takeaways

1. **`express()` returns a callable function** — the app object is both a function and a container for routes and middleware.
2. **The app is a singleton per module** — all files importing `app.js` share the same instance.
3. **`app.listen()` wraps `http.createServer(app)`** — the underlying `http.Server` remains accessible.
4. **Bind to `127.0.0.1` for local dev** and `0.0.0.0` in containers — this is a common production bug.
5. **`app.listen(PORT, HOST, cb)`** fires the callback once when ready; use `server.on('error')` for startup failures.
6. **Routes use `app.METHOD(PATH, HANDLER)`** — `get`, `post`, `put`, `patch`, `delete`, `all`.
7. **`res.send()` guesses Content-Type** — be explicit with `res.type()` when it matters.
8. **`req` and `res` are Node streams** — they inherit from `IncomingMessage` and `ServerResponse`, and Express adds helpers.
9. **`req` is a readable stream** — consume body with middleware or manually via `.on('data')` and `.on('end')`.
10. **`res` is a writable stream** — stream responses with `res.write()`, `res.end()`, or `.pipe()`.
11. **The application lifecycle** has five phases: initialization, binding, accepting, processing, shutdown.
12. **A TCP packet's journey** passes through the OS kernel, Node's HTTP parser, Express middleware, and finally a route handler — then back through `res` to the client.

---

Would you like me to continue with the next topic — **Express Routing**, **Express Middleware**, or **Request and Response Objects in Depth**? I can format the next section in the same style.