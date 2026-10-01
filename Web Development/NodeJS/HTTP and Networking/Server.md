# Production HTTP Server Architecture — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Production HTTP server architecture in Node.js refers to the design and implementation of robust, scalable, and secure web servers using the built-in `node:http` and `node:https` modules, without relying on external frameworks.

**Technical Definition:** The `node:http` module provides the `http.createServer()` method, which returns an `http.Server` instance. The `requestListener` is a function automatically added to the `'request'` event, which is emitted each time there is a request. The server uses `http.IncomingMessage` (the `req` object, a Readable stream) and `http.ServerResponse` (the `res` object, which implements the Writable stream interface) to process requests and send responses. A production architecture must also address graceful shutdown, middleware pipelines, routing, security headers, and connection management.

**Beginner-Friendly Explanation:** Building a production HTTP server is like running a restaurant kitchen. You need a door to let customers in (`createServer`), a way to take orders (`req`), a way to send out dishes (`res`), a system for handling special requests (`routing`), and a procedure for closing time that lets current customers finish their meals (`graceful shutdown`). This cheat sheet shows you how to build all of that using only Node.js's built-in tools — no Express, no Fastify, just the core platform.

### Key Characteristics

- **Zero dependencies:** Built entirely on `node:http` and `node:https`, reducing supply chain risk and attack surface.
- **Full control:** Every aspect of request handling, routing, and response generation is explicit.
- **Stream-native:** Requests are Readable streams and responses are Writable streams, enabling efficient handling of large payloads.
- **Event-driven:** The server emits lifecycle events (`'listening'`, `'request'`, `'close'`) that enable observability and graceful shutdown.
- **Security-conscious:** Requires explicit configuration of CORS, security headers, and input validation.
- **Performance-tunable:** Connection pooling, keep-alive, and timeouts can be configured via server options.

### Prerequisites

- **Node.js runtime:** Node.js 18+ recommended for modern features like `AbortController` and global `fetch`.
- **Basic JavaScript knowledge:** Understanding of functions, callbacks, Promises, and `async/await`.
- **HTTP fundamentals:** Familiarity with request methods, headers, status codes, and bodies.
- **Stream concepts:** Understanding of Readable and Writable streams.
- **Event loop concepts:** How asynchronous operations and backpressure work.

### Related Programming Areas

- **HTTP Fundamentals & Web Protocols:** Request/response anatomy, methods, status codes, headers.
- **Streams:** Reading request bodies and writing responses as streams.
- **Events:** Server lifecycle events and signal handling for graceful shutdown.
- **Security:** CORS, CSP, HSTS, and input validation.
- **Performance:** Connection pooling, keep-alive, and compression.

### Core Concepts

1. **Core Server Infrastructure** — bootstrapping, lifecycle, and graceful shutdown.
2. **Request Processing & Middleware Pipeline** — parsing streams, responses, and content negotiation.
3. **Routing, State, and Security** — native routing, parameter extraction, and CORS.

---

## Core Concept 1: Core Server Infrastructure

### Sub-Feature 1.1: Bootstrapping Servers Using `http.createServer()` and `https.createServer()`

#### Definitions

**Core Definition:** `http.createServer()` and `https.createServer()` create HTTP and HTTPS server instances that listen for incoming connections and invoke a request listener for each request.

**Technical Definition:** `http.createServer(options?, requestListener?)` returns a new `http.Server` object. The `requestListener` is a function automatically added to the `'request'` event. The `options` object supports configuration such as `connectionsCheckingInterval`, `headersTimeout`, `highWaterMark`, `keepAlive`, `keepAliveTimeout`, and `insecureHTTPParser`. `https.createServer(options, requestListener)` returns a new HTTPS web server object; the `options` object is similar to `tls.createServer()` and requires `key` and `cert` (or `pfx`) for TLS.

**Beginner-Friendly Explanation:** `http.createServer()` is the door to your server. You tell it what to do when someone knocks (the request listener), and it handles all the networking details. `https.createServer()` is the same door, but with a lock (TLS/SSL) — you provide a certificate and key so that all communication is encrypted.

#### Purposes

- To create a listening HTTP or HTTPS server that accepts incoming connections.
- To configure server-level options such as timeouts and keep-alive behaviour.
- To provide the entry point for all incoming HTTP requests.

#### Syntax Rules and Structure

**`http.createServer()`:**
```js
const http = require('node:http');
const server = http.createServer(options?, requestListener?);
```

| Option | Description | Default |
|--------|-------------|---------|
| `connectionsCheckingInterval` | Interval to check for request/header timeouts. | 30000 |
| `headersTimeout` | Timeout for receiving complete HTTP headers. | 60000 |
| `highWaterMark` | Overrides socket readable/writable highWaterMark. | — |
| `keepAlive` | Enables keep-alive on sockets. | false |
| `keepAliveInitialDelay` | Initial delay before first keepalive probe. | 0 |
| `keepAliveTimeout` | Idle timeout before socket destruction. | 5000 |
| `insecureHTTPParser` | Uses a lenient HTTP parser (avoid). | false |

**`https.createServer()`:**
```js
const https = require('node:https');
const server = https.createServer({
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem'),
}, requestListener);
```

| Option | Description |
|--------|-------------|
| `key` | Private key (PEM format). |
| `cert` | Certificate (PEM format). |
| `pfx` | PFX or PKCS12 encoded key and certificate bundle. |
| `ca` | Trusted CA certificates. |
| `ciphers` | Cipher suite specification. |
| `minVersion` | Minimum TLS version. |

**Constraints and Limitations:**
- `https.createServer()` requires valid TLS credentials; without them, the server will not start.
- The `insecureHTTPParser` option should never be used in production.
- `http.createServer()` does not provide encryption; use `https.createServer()` for production.

#### Annotated Code Example

```js
// server-bootstrap.js
const http = require('node:http');
const https = require('node:https');
const fs = require('node:fs');

// HTTP server
const httpServer = http.createServer({
  keepAlive: true,
  keepAliveTimeout: 60000,
  headersTimeout: 30000,
}, (req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello from HTTP!\n');
});

httpServer.listen(8080, () => {
  console.log('HTTP server listening on port 8080');
});

// HTTPS server (requires TLS certificates)
try {
  const httpsServer = https.createServer({
    key: fs.readFileSync('server-key.pem'),
    cert: fs.readFileSync('server-cert.pem'),
    minVersion: 'TLSv1.2',
  }, (req, res) => {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello from HTTPS!\n');
  });

  httpsServer.listen(8443, () => {
    console.log('HTTPS server listening on port 8443');
  });
} catch (err) {
  console.error('HTTPS server failed to start:', err.message);
}
```

**Expected Output:**
```
HTTP server listening on port 8080
HTTPS server listening on port 8443
```

**Why this output:** The HTTP server is created with keep-alive enabled and a 60-second idle timeout. The HTTPS server requires a valid key and certificate; if they are missing, the `try/catch` block catches the error. Both servers listen on their respective ports and log a confirmation message.

#### Real-World Cases

- **API servers:** Bootstrapping a REST API with `http.createServer()` and a routing layer.
- **Secure web applications:** Using `https.createServer()` with Let's Encrypt certificates.
- **Internal services:** Using `http.createServer()` for services behind a TLS-terminating load balancer.

---

### Sub-Feature 1.2: Managing Server Lifecycles (`listen()`, `close()`, and Graceful Shutdown)

#### Definitions

**Core Definition:** Server lifecycle management involves starting the server with `listen()`, stopping it with `close()`, and handling termination signals to allow in-flight requests to complete before shutting down (graceful shutdown).

**Technical Definition:** `server.listen(port[, host][, backlog][, callback])` begins accepting connections on the specified port and host. `server.close([callback])` stops the server from accepting new connections and keeps existing connections until all requests are ended, at which point the server is closed and the callback is invoked. Graceful shutdown involves listening for `SIGTERM` and `SIGINT` signals, calling `server.close()`, and then exiting the process once all connections have drained. A timeout should be implemented to force-exit if graceful shutdown takes too long.

**Beginner-Friendly Explanation:** Starting a server is like opening a shop. `listen()` opens the doors. `close()` is like putting up a "closed" sign — no new customers come in, but the ones already inside are allowed to finish their shopping. Graceful shutdown is the process of handling the "closed" sign properly: waiting for current customers to leave, then locking up.

#### Purposes

- To start accepting connections on a specified port and host.
- To stop accepting new connections while allowing existing requests to complete.
- To handle termination signals from process managers (Kubernetes, Docker, Heroku) gracefully.
- To prevent data loss or corruption during deployments and restarts.

#### Syntax Rules and Structure

**`server.listen()`:**
```js
server.listen(port, host?, backlog?, callback?);
```

| Parameter | Description |
|-----------|-------------|
| `port` | Port number (0 for random). |
| `host` | Hostname or IP (default: all interfaces). |
| `backlog` | Maximum pending connections. |
| `callback` | Invoked when the server starts listening. |

**`server.close()`:**
```js
server.close(callback?);
```

| Parameter | Description |
|-----------|-------------|
| `callback` | Invoked when the server is closed. |

**Graceful shutdown pattern:**
```js
function gracefulShutdown(signal) {
  console.log(`${signal} received, shutting down gracefully...`);

  server.close(() => {
    console.log('Server closed. Exiting process.');
    process.exit(0);
  });

  // Force exit after timeout
  setTimeout(() => {
    console.error('Forced shutdown after timeout.');
    process.exit(1);
  }, 30000);
}

process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
process.on('SIGINT', () => gracefulShutdown('SIGINT'));
```

**Constraints and Limitations:**
- `server.close()` does not destroy keep-alive connections immediately; they may remain open.
- A force-exit timeout is essential to prevent hanging processes.
- `SIGKILL` cannot be handled; the process will terminate immediately.

#### Annotated Code Example

```js
// graceful-shutdown.js
const http = require('node:http');

const server = http.createServer((req, res) => {
  // Simulate a slow request
  setTimeout(() => {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Request completed.\n');
  }, 5000);
});

server.listen(3000, () => {
  console.log('Server listening on port 3000');
});

let isShuttingDown = false;

function gracefulShutdown(signal) {
  if (isShuttingDown) return;
  isShuttingDown = true;

  console.log(`${signal} received. Starting graceful shutdown...`);

  server.close(() => {
    console.log('All connections closed. Exiting.');
    process.exit(0);
  });

  // Force exit after 30 seconds
  setTimeout(() => {
    console.error('Forced shutdown after 30s timeout.');
    process.exit(1);
  }, 30000);
}

process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
process.on('SIGINT', () => gracefulShutdown('SIGINT'));

console.log('Press Ctrl+C to trigger graceful shutdown.');
```

**Expected Output (when Ctrl+C is pressed during an active request):**
```
Server listening on port 3000
Press Ctrl+C to trigger graceful shutdown.
^C
SIGINT received. Starting graceful shutdown...
Request completed.
All connections closed. Exiting.
```

**Why this output:** The `SIGINT` signal triggers `gracefulShutdown()`. `server.close()` stops accepting new connections but allows the in-flight request (which takes 5 seconds) to complete. Once the response is sent and the connection closes, the `close` callback fires and the process exits cleanly. The `isShuttingDown` flag prevents multiple shutdown triggers.

#### Real-World Cases

- **Kubernetes deployments:** Handling `SIGTERM` during rolling updates.
- **Docker containers:** Ensuring in-flight requests complete before container termination.
- **Heroku dynos:** Handling `SIGTERM` during dyno cycling.

---

## Core Concept 2: Request Processing & Middleware Pipeline

### Sub-Feature 2.1: Parsing Incoming Streams via `req` (Readable Stream)

#### Definitions

**Core Definition:** The `req` object (`http.IncomingMessage`) is a Readable stream that provides access to the request headers, method, URL, and body data.

**Technical Definition:** `http.IncomingMessage` implements the Readable stream interface. It exposes properties such as `req.method`, `req.url`, `req.headers`, and `req.httpVersion`. The request body is consumed by listening to `'data'` events or using `for await...of`. For JSON bodies, the data must be accumulated and parsed. For streaming bodies (e.g., file uploads), the stream can be piped directly to a destination.

**Beginner-Friendly Explanation:** The `req` object is like a delivery package. It has labels on the outside (headers: who sent it, what's inside) and contents inside (the body). You can read the labels immediately, but to read the contents, you need to unpack the package piece by piece — that's what reading from the stream does.

#### Purposes

- To access request metadata (method, URL, headers).
- To read the request body for POST, PUT, and PATCH requests.
- To stream large request bodies without buffering them entirely.
- To parse JSON, URL-encoded, or multipart form data.

#### Syntax Rules and Structure

**Reading the body as a buffer:**
```js
let body = '';
req.on('data', (chunk) => { body += chunk; });
req.on('end', () => { /* parse body */ });
```

**Reading the body with async iteration:**
```js
let body = '';
for await (const chunk of req) {
  body += chunk;
}
```

**Parsing JSON:**
```js
async function parseJSONBody(req) {
  let body = '';
  for await (const chunk of req) {
    body += chunk;
  }
  return JSON.parse(body);
}
```

| Property | Description |
|----------|-------------|
| `req.method` | HTTP method (GET, POST, etc.). |
| `req.url` | Request URL path and query string. |
| `req.headers` | Object of request headers. |
| `req.httpVersion` | HTTP version string. |

**Constraints and Limitations:**
- The body can only be consumed once.
- Large bodies can exhaust memory if buffered entirely; use streaming for large payloads.
- JSON parsing throws `SyntaxError` on invalid input; wrap in `try/catch`.

#### Annotated Code Example

```js
// request-parsing.js
const http = require('node:http');

const server = http.createServer(async (req, res) => {
  console.log('Method:', req.method);
  console.log('URL:', req.url);
  console.log('Headers:', req.headers);

  if (req.method === 'POST' && req.headers['content-type'] === 'application/json') {
    try {
      let body = '';
      for await (const chunk of req) {
        body += chunk;
      }

      const data = JSON.parse(body);
      console.log('Parsed JSON:', data);

      res.writeHead(200, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ received: data }));
    } catch (err) {
      res.writeHead(400, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ error: 'Invalid JSON' }));
    }
  } else {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Send a POST request with JSON body.\n');
  }
});

server.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output (for a POST with `{"name":"Alice"}`):**
```
Method: POST
URL: /api/users
Headers: { 'content-type': 'application/json', ... }
Parsed JSON: { name: 'Alice' }
```

**Why this output:** The server checks the method and content type. The `for await...of` loop accumulates the request body chunks into a string. `JSON.parse` converts it to an object. The response echoes the parsed data back to the client.

#### Real-World Cases

- **REST APIs:** Parsing JSON request bodies for POST/PUT/PATCH endpoints.
- **File uploads:** Streaming multipart form data to disk or cloud storage.
- **Webhooks:** Receiving and validating JSON payloads from third-party services.

---

### Sub-Feature 2.2: Streamlining Text or Binary Responses via `res` (Writable Stream)

#### Definitions

**Core Definition:** The `res` object (`http.ServerResponse`) implements the Writable stream interface, allowing data to be written incrementally and finalised with `res.end()`.

**Technical Definition:** `http.ServerResponse` implements, but does not inherit from, the Writable Stream interface. When `response.write()` is called, it invokes `http.ServerResponse.write()` rather than `stream.Writable.write()`. Internally, `http.ServerResponse` is an `OutgoingMessage` which writes to the connection socket. The response can be written in chunks using `res.write()` and must be finalised with `res.end()`. Headers are set via `res.writeHead()` or `res.setHeader()`.

**Beginner-Friendly Explanation:** The `res` object is like a conveyor belt that carries your response back to the client. You can place items on the belt one at a time (`res.write()`) or all at once (`res.end()`). Once you've sent everything, you signal that the belt is done.

#### Purposes

- To send responses incrementally, enabling streaming of large data.
- To set status codes and headers before sending the body.
- To send JSON, text, HTML, or binary data.
- To pipe file streams directly to the response for efficient file serving.

#### Syntax Rules and Structure

**Setting status and headers:**
```js
res.writeHead(statusCode, statusMessage?, headers?);
res.setHeader(name, value);
res.statusCode = 200;
```

**Writing the body:**
```js
res.write(chunk, encoding?);
res.end(chunk?, encoding?, callback?);
```

**Piping a stream:**
```js
fs.createReadStream('file.txt').pipe(res);
```

| Method | Description |
|--------|-------------|
| `res.writeHead()` | Sends response headers with status code. |
| `res.setHeader()` | Sets a single header value. |
| `res.write()` | Sends a chunk of the response body. |
| `res.end()` | Finalises the response. |
| `res.statusCode` | Property to set the status code. |

**Constraints and Limitations:**
- `res.writeHead()` must be called before `res.write()` or `res.end()`.
- Headers cannot be changed after they have been sent.
- `res.end()` must be called to complete the response; otherwise the client hangs.

#### Annotated Code Example

```js
// response-streaming.js
const http = require('node:http');
const fs = require('node:fs');

const server = http.createServer((req, res) => {
  if (req.url === '/file') {
    // Stream a file to the response
    const stream = fs.createReadStream('large-file.txt');
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    stream.pipe(res);
  } else if (req.url === '/json') {
    // Send JSON incrementally
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.write('{"message":');
    setTimeout(() => {
      res.write('"Hello, World!"');
      res.end('}');
    }, 1000);
  } else {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello, World!\n');
  }
});

server.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output (for `/json`):**
```json
{"message":"Hello, World!"}
```

**Why this output:** The response is written in three chunks: `{"message":`, then `"Hello, World!"` after a 1-second delay, then `}`. The client receives the complete JSON object. This demonstrates incremental writing, which is useful for streaming large responses or server-sent events.

#### Real-World Cases

- **File serving:** Piping file streams to HTTP responses.
- **Server-Sent Events (SSE):** Writing data incrementally for real-time updates.
- **Video streaming:** Streaming video files with range request support.

---

### Sub-Feature 2.3: Content-Type Negotiation and Structured Serialization

#### Definitions

**Core Definition:** Content negotiation is the process by which the server selects the best representation of a resource based on the client's `Accept` header, and serialization is the conversion of data structures into a specific format (JSON, XML, etc.).

**Technical Definition:** The client sends an `Accept` header specifying which media types it can handle, ordered by preference using quality values (`q`). The server examines this header and picks the best match from the formats it supports. In native Node.js, this requires parsing the `Accept` header manually or using a library like `negotiator`. Structured serialization converts JavaScript objects into the selected format.

**Beginner-Friendly Explanation:** Content negotiation is like a restaurant menu with multiple languages. The customer (client) says "I can read English and Spanish" (the `Accept` header), and the server responds in whichever language the customer prefers. Serialization is the act of writing the menu in that language.

#### Purposes

- To serve the same resource in multiple formats (JSON, XML, HTML).
- To respect client preferences for content type.
- To enable API versioning through content types.
- To support both human-readable and machine-readable responses.

#### Syntax Rules and Structure

**Parsing `Accept` header:**
```js
const acceptHeader = req.headers['accept'] || '*/*';
// Example: 'application/json, text/html;q=0.9, */*;q=0.8'
```

**Simple content negotiation:**
```js
function negotiateContentType(req) {
  const accept = req.headers['accept'] || '*/*';

  if (accept.includes('application/json')) return 'application/json';
  if (accept.includes('text/html')) return 'text/html';
  if (accept.includes('text/plain')) return 'text/plain';

  return 'application/json'; // default
}
```

**Serialization:**
```js
function serialize(data, contentType) {
  if (contentType === 'application/json') {
    return JSON.stringify(data);
  }
  if (contentType === 'text/html') {
    return `<pre>${JSON.stringify(data, null, 2)}</pre>`;
  }
  return String(data);
}
```

| Header | Description |
|--------|-------------|
| `Accept` | Media types the client can handle. |
| `Content-Type` | Media type of the response body. |
| `q` parameter | Quality value (0 to 1) indicating preference. |

**Constraints and Limitations:**
- The `Accept` header can be complex; a proper parser is recommended.
- If the server cannot satisfy the `Accept` header, it should return 406 Not Acceptable.
- JSON is not suitable for all data types (e.g., binary data).

#### Annotated Code Example

```js
// content-negotiation.js
const http = require('node:http');

const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
];

function negotiate(req) {
  const accept = req.headers['accept'] || '*/*';
  if (accept.includes('application/json')) return 'json';
  if (accept.includes('text/html')) return 'html';
  return 'json';
}

function serialize(data, format) {
  if (format === 'json') {
    return JSON.stringify(data);
  }
  if (format === 'html') {
    const rows = data.map((u) => `<tr><td>${u.id}</td><td>${u.name}</td></tr>`).join('');
    return `<table>${rows}</table>`;
  }
  return String(data);
}

const server = http.createServer((req, res) => {
  const format = negotiate(req);
  const contentType = format === 'json' ? 'application/json' : 'text/html';
  const body = serialize(users, format);

  res.writeHead(200, { 'Content-Type': contentType });
  res.end(body);
});

server.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output (for `Accept: application/json`):**
```json
[{"id":1,"name":"Alice"},{"id":2,"name":"Bob"}]
```

**Expected Output (for `Accept: text/html`):**
```html
<table><tr><td>1</td><td>Alice</td></tr><tr><td>2</td><td>Bob</td></tr></table>
```

**Why this output:** The server parses the `Accept` header and selects JSON or HTML. The `serialize` function converts the `users` array into the selected format. The `Content-Type` header reflects the chosen format.

#### Real-World Cases

- **REST APIs:** Serving JSON by default, HTML for browser clients.
- **API versioning:** Using `Accept: application/vnd.api.v2+json` for versioning.
- **Data export:** Serving CSV or XML based on client preferences.

---

## Core Concept 3: Routing, State, and Security

### Sub-Feature 3.1: Implementing Native Static and Dynamic URL Routing (Without External Frameworks)

#### Definitions

**Core Definition:** Native routing involves parsing the request URL and dispatching to the appropriate handler based on the path and method, without using a framework like Express.

**Technical Definition:** In native Node.js, routing requires parsing `req.url` using the `node:url` module (or the WHATWG `URL` class) to extract the pathname and query parameters. Static routes match exact paths; dynamic routes use patterns (e.g., `/users/:id`) that are converted to regular expressions. The `URLPattern` API (available in Node.js 23.8+) provides a standard way to match URL patterns. Without `URLPattern`, developers typically use `path-to-regexp` or manual regex matching.

**Beginner-Friendly Explanation:** Routing is like a receptionist at a hotel. When a guest arrives, the receptionist looks at their request ("I need room 302") and directs them to the right place. Static routing is for exact requests ("room 302"), while dynamic routing handles patterns ("any room number").

#### Purposes

- To dispatch requests to the correct handler based on path and method.
- To extract dynamic path parameters (e.g., user IDs).
- To organise server logic into modular handlers.
- To support RESTful API design.

#### Syntax Rules and Structure

**Parsing the URL:**
```js
const url = new URL(req.url, `http://${req.headers.host}`);
const pathname = url.pathname;
const query = Object.fromEntries(url.searchParams);
```

**Static routing:**
```js
const routes = {
  'GET /': handleHome,
  'GET /about': handleAbout,
  'POST /api/users': handleCreateUser,
};

const key = `${req.method} ${pathname}`;
const handler = routes[key];
if (handler) handler(req, res); else notFound(res);
```

**Dynamic routing (regex):**
```js
const dynamicRoutes = [
  { pattern: /^\/users\/(\d+)$/, method: 'GET', handler: handleGetUser },
  { pattern: /^\/posts\/(\w+)$/, method: 'GET', handler: handleGetPost },
];

for (const route of dynamicRoutes) {
  const match = pathname.match(route.pattern);
  if (match && req.method === route.method) {
    return route.handler(req, res, match[1]);
  }
}
```

| Component | Description |
|-----------|-------------|
| `URL` | Parses the request URL. |
| `pathname` | The path portion (e.g., `/users/42`). |
| `searchParams` | Query parameters. |
| `RegExp` | Used for dynamic route matching. |

**Constraints and Limitations:**
- Regex-based routing can be error-prone; consider a dedicated router library.
- The `URLPattern` API is relatively new (Node.js 23.8+).
- Route ordering matters for overlapping patterns.

#### Annotated Code Example

```js
// native-routing.js
const http = require('node:http');

// Static routes
const staticRoutes = {
  'GET /': (req, res) => {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Home page\n');
  },
  'GET /about': (req, res) => {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('About page\n');
  },
};

// Dynamic routes
const dynamicRoutes = [
  {
    method: 'GET',
    pattern: /^\/users\/(\d+)$/,
    handler: (req, res, id) => {
      res.writeHead(200, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ userId: id, name: `User ${id}` }));
    },
  },
  {
    method: 'GET',
    pattern: /^\/posts\/(\w+)$/,
    handler: (req, res, slug) => {
      res.writeHead(200, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ slug, title: `Post ${slug}` }));
    },
  },
];

const server = http.createServer((req, res) => {
  const url = new URL(req.url, `http://${req.headers.host}`);
  const { pathname } = url;

  // Check static routes first
  const staticKey = `${req.method} ${pathname}`;
  if (staticRoutes[staticKey]) {
    return staticRoutes[staticKey](req, res);
  }

  // Check dynamic routes
  for (const route of dynamicRoutes) {
    if (req.method !== route.method) continue;
    const match = pathname.match(route.pattern);
    if (match) {
      return route.handler(req, res, match[1]);
    }
  }

  // 404
  res.writeHead(404, { 'Content-Type': 'text/plain' });
  res.end('404 Not Found\n');
});

server.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output (for `GET /users/42`):**
```json
{"userId":"42","name":"User 42"}
```

**Expected Output (for `GET /nonexistent`):**
```
404 Not Found
```

**Why this output:** The server parses the URL and checks static routes first. If no static route matches, it iterates through dynamic routes, matching the pathname against regex patterns and extracting the captured group (`42` for `/users/42`). If no route matches, a 404 response is returned.

#### Real-World Cases

- **REST APIs:** Routing `/users`, `/users/:id`, `/posts/:slug`.
- **Static file servers:** Routing `/assets/*` to a file-serving handler.
- **Webhook endpoints:** Routing `/webhooks/github`, `/webhooks/stripe`.

---

### Sub-Feature 3.2: Extracting Path Parameters, Query Strings, and Body Data Safely

#### Definitions

**Core Definition:** Path parameters are dynamic segments in the URL path (e.g., `/users/:id`), query strings are key-value pairs after the `?` (e.g., `?page=1&limit=10`), and body data is the payload of POST/PUT/PATCH requests.

**Technical Definition:** Path parameters are extracted by matching the pathname against a regex pattern with capture groups. Query strings are parsed using `URL.searchParams` or `querystring.parse()`. Body data is read from the request stream and parsed according to the `Content-Type` header (JSON, URL-encoded, multipart). All input must be validated and sanitised to prevent injection attacks.

**Beginner-Friendly Explanation:** Path parameters identify a specific resource (which user?), query strings customise the request (which page?), and body data contains the actual content (what data are you sending?). All three must be handled carefully because they come from untrusted clients.

#### Purposes

- To identify specific resources via path parameters.
- To filter, sort, and paginate via query strings.
- To receive data submissions via request bodies.
- To validate and sanitise all client input.

#### Syntax Rules and Structure

**Path parameters:**
```js
const match = pathname.match(/^\/users\/(\d+)$/);
const userId = match[1]; // "42"
```

**Query strings:**
```js
const url = new URL(req.url, `http://${req.headers.host}`);
const page = url.searchParams.get('page') || '1';
const limit = url.searchParams.get('limit') || '10';
```

**Body data (JSON):**
```js
async function getJSONBody(req) {
  let body = '';
  for await (const chunk of req) body += chunk;
  return JSON.parse(body);
}
```

**Body data (URL-encoded):**
```js
const querystring = require('node:querystring');
const params = querystring.parse(body);
```

| Source | Extraction Method |
|--------|-------------------|
| Path parameter | Regex capture group. |
| Query string | `URL.searchParams.get()`. |
| JSON body | `JSON.parse()` after stream read. |
| URL-encoded body | `querystring.parse()`. |

**Constraints and Limitations:**
- Always validate path parameters (e.g., ensure IDs are numeric).
- Query parameters are always strings; convert types explicitly.
- JSON body parsing can throw; wrap in `try/catch`.
- Body size should be limited to prevent denial-of-service.

#### Annotated Code Example

```js
// safe-extraction.js
const http = require('node:http');

const server = http.createServer(async (req, res) => {
  const url = new URL(req.url, `http://${req.headers.host}`);
  const { pathname } = url;

  // Path parameter: /users/:id
  const userMatch = pathname.match(/^\/users\/(\d+)$/);
  if (userMatch) {
    const userId = parseInt(userMatch[1], 10);
    if (isNaN(userId) || userId < 1) {
      res.writeHead(400, { 'Content-Type': 'application/json' });
      return res.end(JSON.stringify({ error: 'Invalid user ID' }));
    }

    // Query strings
    const fields = url.searchParams.get('fields') || 'id,name';
    const allowedFields = ['id', 'name', 'email'];
    const selected = fields.split(',').filter((f) => allowedFields.includes(f));

    res.writeHead(200, { 'Content-Type': 'application/json' });
    return res.end(JSON.stringify({
      userId,
      fields: selected,
      query: Object.fromEntries(url.searchParams),
    }));
  }

  // Body data: POST /users
  if (req.method === 'POST' && pathname === '/users') {
    try {
      let body = '';
      for await (const chunk of req) body += chunk;

      if (body.length > 1024 * 1024) {
        res.writeHead(413, { 'Content-Type': 'application/json' });
        return res.end(JSON.stringify({ error: 'Body too large' }));
      }

      const data = JSON.parse(body);

      // Validate required fields
      if (!data.name || typeof data.name !== 'string') {
        res.writeHead(400, { 'Content-Type': 'application/json' });
        return res.end(JSON.stringify({ error: 'name is required' }));
      }

      res.writeHead(201, { 'Content-Type': 'application/json' });
      return res.end(JSON.stringify({ id: 1, name: data.name }));
    } catch (err) {
      res.writeHead(400, { 'Content-Type': 'application/json' });
      return res.end(JSON.stringify({ error: 'Invalid JSON' }));
    }
  }

  res.writeHead(404, { 'Content-Type': 'text/plain' });
  res.end('Not Found\n');
});

server.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output (for `GET /users/42?fields=id,name,invalid`):**
```json
{"userId":42,"fields":["id","name"],"query":{"fields":"id,name,invalid"}}
```

**Why this output:** The path parameter `42` is validated as a positive integer. The `fields` query parameter is split and filtered against an allowlist, removing `invalid`. The response includes the validated user ID and the sanitised field list.

#### Real-World Cases

- **User profiles:** `GET /users/:id` with query-based field selection.
- **Search APIs:** `GET /search?q=term&page=1&limit=20`.
- **Form submissions:** `POST /users` with JSON body validation.

---

### Sub-Feature 3.3: Setting Status Codes, Managing Headers, and Configuring Global CORS Access Policies

#### Definitions

**Core Definition:** CORS (Cross-Origin Resource Sharing) is a browser-enforced security mechanism that uses HTTP headers to control which origins can read responses from your API.

**Technical Definition:** CORS is a set of HTTP headers your server sends that tell the browser which other origins are allowed to read your API's responses, relaxing the browser's same-origin policy in a controlled way. The key header is `Access-Control-Allow-Origin`. For requests with credentials (cookies, auth headers), the origin must be specified explicitly (not `*`) and `Access-Control-Allow-Credentials: true` must be set. Preflight requests (`OPTIONS`) must be handled for methods other than simple GET/POST.

**Beginner-Friendly Explanation:** CORS is like a bouncer at a club. By default, the bouncer only lets in people from the same neighbourhood (same origin). CORS headers are like telling the bouncer "also let in people from this specific other neighbourhood." You can't just let everyone in (that's `*`) if you're checking IDs (credentials) — you have to name the specific neighbourhoods you trust.

#### Purposes

- To allow legitimate cross-origin requests from trusted frontends.
- To protect against cross-site request forgery (CSRF) and data theft.
- To handle preflight `OPTIONS` requests correctly.
- To configure security headers that protect against XSS, clickjacking, and MIME sniffing.

#### Syntax Rules and Structure

**CORS headers:**
```js
res.setHeader('Access-Control-Allow-Origin', 'https://app.example.com');
res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
res.setHeader('Access-Control-Allow-Credentials', 'true');
res.setHeader('Access-Control-Max-Age', '86400');
```

**Security headers:**
```js
res.setHeader('X-Content-Type-Options', 'nosniff');
res.setHeader('X-Frame-Options', 'DENY');
res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
res.setHeader('Content-Security-Policy', "default-src 'self'");
res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
```

| Header | Purpose |
|--------|---------|
| `Access-Control-Allow-Origin` | Which origins can read the response. |
| `Access-Control-Allow-Methods` | Allowed HTTP methods. |
| `Access-Control-Allow-Headers` | Allowed request headers. |
| `Access-Control-Allow-Credentials` | Whether cookies/auth are allowed. |
| `Strict-Transport-Security` | Forces HTTPS. |
| `Content-Security-Policy` | Controls resource loading. |
| `X-Content-Type-Options` | Prevents MIME sniffing. |
| `X-Frame-Options` | Prevents clickjacking. |

**Constraints and Limitations:**
- `Access-Control-Allow-Origin: *` cannot be used with credentials.
- Preflight requests must return 204 or 200 with the correct headers.
- Security headers should be configured carefully to avoid breaking legitimate functionality.

#### Annotated Code Example

```js
// cors-security.js
const http = require('node:http');

const ALLOWED_ORIGINS = [
  'https://app.example.com',
  'https://admin.example.com',
];

function setSecurityHeaders(res) {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  res.setHeader('Content-Security-Policy', "default-src 'self'");
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
}

function setCORSHeaders(req, res) {
  const origin = req.headers.origin;

  if (origin && ALLOWED_ORIGINS.includes(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin);
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    res.setHeader('Access-Control-Allow-Credentials', 'true');
    res.setHeader('Access-Control-Max-Age', '86400');
  }
}

const server = http.createServer((req, res) => {
  setSecurityHeaders(res);
  setCORSHeaders(req, res);

  // Handle preflight
  if (req.method === 'OPTIONS') {
    res.writeHead(204);
    return res.end();
  }

  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ message: 'Hello, CORS!' }));
});

server.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output (for a request from `https://app.example.com`):**
```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
Content-Type: application/json
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

**Why this output:** The server checks the `Origin` header against an allowlist. If the origin is allowed, it sets the CORS headers with the specific origin (not `*`) and enables credentials. Security headers are set on every response. Preflight requests are handled with a 204 response.

#### Real-World Cases

- **SPA frontends:** Allowing a React app on a different domain to call the API.
- **Multi-tenant applications:** Allowing tenant-specific origins.
- **Security hardening:** Adding CSP, HSTS, and X-Frame-Options to all responses.

---

## References

- Node.js Documentation — HTTP — https://nodejs.org/api/http.html
- Node.js Documentation — `http.createServer()` — https://beta.docs.nodejs.org/http/createServer
- Node.js Documentation — HTTPS — https://nodejs.org/api/https.html
- Node.js Documentation — `http.Server` — https://nodejs.org/api/http.html#class-httpserver
- Node.js Documentation — `http.IncomingMessage` — https://nodejs.org/api/http.html#class-httpincomingmessage
- Node.js Documentation — `http.ServerResponse` — https://nodejs.org/api/http.html#class-httpserverresponse
- Node.js Documentation — Stream — https://nodejs.org/api/stream.html
- Node.js Documentation — URL — https://nodejs.org/api/url.html
- Node.js Documentation — `URLPattern` — https://nodejs.org/api/url.html#class-urlpattern
- MDN Web Docs — CORS — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS
- MDN Web Docs — HTTP Headers — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers
- OWASP — Secure Headers Project — https://owasp.org/www-project-secure-headers/
- CORS in Node.js: Secure Configuration Guide — https://safeguard.sh/resources/blog/cors-in-node-js
- Graceful Shutdown in Node.js — https://legacy.cs.indiana.edu/~dgerman/mean-tutorial/holmes005.pdf
- Content Negotiation in APIs — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-30-api-content-negotiation/README.md