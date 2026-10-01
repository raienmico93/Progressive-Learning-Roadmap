# Modern HTTP Clients in Node.js — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Modern HTTP clients in Node.js are APIs and libraries that enable applications to send HTTP requests and receive responses, ranging from the legacy `http.request()` and `https.request()` built-in modules to the standards-based global `fetch()` API and the high-performance `undici` library that powers it.

**Technical Definition:** Node.js provides multiple HTTP client interfaces. The legacy `node:http` and `node:https` modules expose `http.request()` and `https.request()` for low-level control over HTTP/1.1 connections. The global `fetch()` API, available since Node.js v18, implements the WHATWG Fetch Standard and is backed by `undici`, a high-performance HTTP/1.1 client written from scratch for Node.js. `undici` also exposes lower-level primitives such as `Client`, `Pool`, and `Agent` for connection management, and supports HTTP/2 via ALPN negotiation.

**Beginner-Friendly Explanation:** When your Node.js program needs to talk to another server — fetching an API, downloading a file, or posting data — it uses an HTTP client. Node.js offers three main options: the **old-school** way (`http.request()`, which gives you full control but requires a lot of boilerplate), the **modern** way (`fetch()`, which is simple and works like the browser's fetch), and the **performance** way (`undici`, which is what `fetch()` uses under the hood but with more tuning options). Think of them as different vehicles: a manual-transmission truck, an automatic sedan, and a sports car.

### Key Characteristics

- **Three generations of clients:** Legacy callback-based (`http.request`), Promise-based standard (`fetch`), and low-level high-performance (`undici`).
- **WHATWG Fetch Standard compliance:** The global `fetch()` API is standards-compliant and interoperable with web streams.
- **Connection pooling:** Both legacy `http.Agent` and undici `Agent` manage persistent connections for reuse.
- **Streaming support:** Request and response bodies can be streamed via Node.js Readable streams or Web Streams.
- **AbortController integration:** Modern clients support cancellation via `AbortController` and `AbortSignal`.
- **Resilience patterns:** Retries, exponential backoff, and circuit breakers can be layered on top of any client.

### Prerequisites

- **Node.js runtime:** Node.js 18+ for global `fetch()`. Undici is bundled with Node.js from v18.
- **Basic JavaScript knowledge:** Understanding of Promises, `async/await`, and callbacks.
- **HTTP fundamentals:** Familiarity with request methods, headers, status codes, and bodies.
- **Stream concepts:** Understanding of Readable and Writable streams.

### Related Programming Areas

- **HTTP Fundamentals & Web Protocols:** Request/response anatomy, methods, status codes, headers.
- **Streams:** Streaming request and response bodies using Node.js streams and Web Streams.
- **Events:** AbortController and AbortSignal for cancellation.
- **Security:** TLS configuration, certificate pinning, and secure headers.
- **Resilience:** Retry logic, circuit breakers, and timeout management.

### Core Concepts

1. **Built-in Client APIs** — `http.request`, `https.request`, global `fetch`, and `undici`.
2. **Request Configuration & Orchestration** — custom headers, streaming vs. buffering bodies.
3. **Response Handling & Resilience** — response consumption, timeouts, retries, circuit breakers, AbortController, and connection pooling.

---

## Core Concept 1: Built-in Client APIs

### Sub-Feature 1.1: Legacy Client Architecture (`http.request()` and `https.request()`)

#### Definitions

**Core Definition:** `http.request()` and `https.request()` are Node.js's original built-in HTTP client APIs, providing low-level control over HTTP/1.1 connections with callback-based response handling.

**Technical Definition:** `http.request(url, options?, callback?)` returns an instance of `http.ClientRequest`. The `ClientRequest` instance is a writable stream. If one needs to upload a file with a POST request, then write to the `ClientRequest` object. The optional `callback` parameter will be added as a one-time listener for the `'response'` event. With `http.request()` one must always call `req.end()` to signify that you're done with the request — even if there is no data being written to the request body. `https.request()` is identical but makes requests to secure web servers, accepting additional TLS options such as `ca`, `cert`, `ciphers`, `key`, and `rejectUnauthorized`. All options from `http.request()` are valid for `https.request()`.

**Beginner-Friendly Explanation:** `http.request()` is the old-fashioned way to make HTTP calls. You create a request object, attach listeners for the response, write the body (if any), and call `req.end()` when you're done. It gives you complete control but requires a lot of manual setup — like building a car from parts instead of buying one.

#### Purposes

- To make HTTP requests with full control over the underlying connection.
- To support custom agents, socket options, and TLS configuration.
- To handle HTTP/1.1 requests in environments where `fetch` is not available.
- To implement low-level HTTP clients and proxies.

#### Syntax Rules and Structure

**`http.request()`:**
```js
const http = require('node:http');
const req = http.request(options, (res) => { /* handle response */ });
req.end();
```

| Option | Description |
|--------|-------------|
| `host` / `hostname` | Domain name or IP address. Default: `'localhost'`. |
| `port` | Port number. Default: 80 (http) / 443 (https). |
| `path` | Request path (e.g., `/api/users`). |
| `method` | HTTP method (e.g., `'GET'`, `'POST'`). |
| `headers` | Object or array of request headers. |
| `auth` | Basic authentication (`'user:password'`). |
| `agent` | `Agent` object or `false` to disable pooling. |
| `timeout` | Socket timeout in milliseconds. |

**`https.request()` additional TLS options:**
| Option | Description |
|--------|-------------|
| `ca` | Trusted CA certificates. |
| `cert` | Client certificate. |
| `key` | Client private key. |
| `rejectUnauthorized` | If `false`, disables certificate validation (insecure). |
| `servername` | Server Name Indication (SNI). |

**Constraints and Limitations:**
- `req.end()` must always be called, even for GET requests.
- The response callback receives an `http.IncomingMessage` (a Readable stream).
- Error handling requires manual `'error'` event listeners on both request and response.
- `https.request()` with `rejectUnauthorized: false` disables certificate validation and is a security risk.

#### Annotated Code Example

```js
// http-request.js
const https = require('node:https');

const options = {
  hostname: 'jsonplaceholder.typicode.com',
  port: 443,
  path: '/posts/1',
  method: 'GET',
  headers: {
    'User-Agent': 'Node.js-Client/1.0',
  },
};

const req = https.request(options, (res) => {
  console.log('Status:', res.statusCode);
  console.log('Headers:', res.headers);

  let body = '';
  res.setEncoding('utf8');
  res.on('data', (chunk) => { body += chunk; });
  res.on('end', () => {
    console.log('Body:', JSON.parse(body));
  });
});

req.on('error', (err) => {
  console.error('Request failed:', err.message);
});

req.setTimeout(5000, () => {
  req.destroy(new Error('Request timed out'));
});

req.end();
```

**Expected Output:**
```
Status: 200
Headers: { ... }
Body: { userId: 1, id: 1, title: '...', body: '...' }
```

**Why this output:** The `https.request()` creates a secure request to the JSONPlaceholder API. The response callback receives the response stream, which is consumed via `'data'` and `'end'` events. The `setTimeout` method applies a 5-second socket timeout. `req.end()` finalises the request.

#### Real-World Cases

- **Legacy codebases:** Maintaining applications that predate `fetch`.
- **Custom proxies:** Building HTTP proxies that require low-level socket control.
- **Certificate pinning:** Implementing HPKP-style certificate pinning with `checkServerIdentity`.

---

### Sub-Feature 1.2: Modern Standard Client (Global `fetch` API and Web Streams Integration)

#### Definitions

**Core Definition:** The global `fetch()` API is a WHATWG-standard, Promise-based HTTP client available in Node.js since v18, returning a `Response` object whose body is a Web `ReadableStream`.

**Technical Definition:** The global `fetch()` method starts the process of fetching a resource from the network, returning a promise that is fulfilled once the response is available. The promise resolves to the `Response` object representing the response to your request. A `fetch()` promise only rejects when the request fails — for example, because of a badly-formed request URL or a network error. A `fetch()` promise does not reject if the server responds with HTTP status codes that indicate errors (404, 504, etc.); instead, the `response.ok` and/or `response.status` properties must be checked. In Node.js, `fetch` is provided by `undici`, and the `Response.body` is a Web `ReadableStream` that can be converted to a Node.js Readable stream via `Readable.fromWeb()`.

**Beginner-Friendly Explanation:** `fetch()` is the modern, simple way to make HTTP requests. It works exactly like `fetch` in the browser — you call `fetch(url)`, and you get back a Promise that resolves to a Response. You can then call `response.json()` or `response.text()` to read the body. In Node.js, `fetch` is powered by `undici`, which means it's fast and supports Web Streams for handling large responses efficiently.

#### Purposes

- To make HTTP requests with a clean, Promise-based API.
- To interoperate with browser-compatible code (isomorphic JavaScript).
- To consume responses as JSON, text, ArrayBuffer, or streams.
- To leverage Web Streams for streaming responses.

#### Syntax Rules and Structure

```js
const response = await fetch(resource, options);
```

| Parameter | Description |
|-----------|-------------|
| `resource` | URL string, `URL` object, or `Request` object. |
| `options.method` | HTTP method (default: `'GET'`). |
| `options.headers` | Headers object or `Headers` instance. |
| `options.body` | Request body (string, Buffer, Stream, FormData, etc.). |
| `options.signal` | `AbortSignal` for cancellation. |
| `options.duplex` | Must be `'half'` when `body` is a stream. |

**Response consumption methods:**
| Method | Returns | Use Case |
|--------|---------|----------|
| `response.json()` | Promise\<Object\> | JSON APIs |
| `response.text()` | Promise\<string\> | Text/HTML responses |
| `response.arrayBuffer()` | Promise\<ArrayBuffer\> | Binary data (images, PDFs) |
| `response.blob()` | Promise\<Blob\> | Binary with MIME type |
| `response.body` | ReadableStream | Streaming large responses |
| `response.ok` | boolean | `true` if status is 200–299 |
| `response.status` | number | HTTP status code |

**Constraints and Limitations:**
- `fetch()` does not reject on HTTP error status codes (4xx, 5xx); check `response.ok`.
- A `fetch()` promise only rejects on network errors or invalid URLs.
- When sending a stream body, `duplex: 'half'` must be set in the options.
- The `Response.body` is a Web Stream, not a Node.js stream; use `Readable.fromWeb()` to convert.

#### Annotated Code Example

```js
// fetch-example.js
const { Readable } = require('node:stream');

async function fetchPost(id) {
  try {
    const response = await fetch(
      `https://jsonplaceholder.typicode.com/posts/${id}`,
      {
        headers: {
          'User-Agent': 'Node.js-Fetch/1.0',
          'Accept': 'application/json',
        },
        signal: AbortSignal.timeout(5000), // 5-second timeout
      }
    );

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    const data = await response.json();
    console.log('Post:', data);
    return data;
  } catch (err) {
    console.error('Fetch failed:', err.message);
  }
}

fetchPost(1);
```

**Expected Output:**
```
Post: { userId: 1, id: 1, title: '...', body: '...' }
```

**Why this output:** `fetch()` sends a GET request with custom headers and a 5-second timeout via `AbortSignal.timeout()`. The `response.ok` check catches HTTP errors. `response.json()` parses the body as JSON. The `catch` block handles network errors and timeouts.

#### Streaming a Response with `fetch`

```js
// fetch-stream.js
const { Readable } = require('node:stream');
const fs = require('node:fs');

async function downloadFile(url, outputPath) {
  const response = await fetch(url);

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }

  // Convert Web ReadableStream to Node.js Readable
  const nodeStream = Readable.fromWeb(response.body);
  const writeStream = fs.createWriteStream(outputPath);

  await new Promise((resolve, reject) => {
    nodeStream.pipe(writeStream);
    writeStream.on('finish', resolve);
    writeStream.on('error', reject);
  });

  console.log('Download complete:', outputPath);
}

downloadFile('https://example.com/large-file.zip', 'large-file.zip');
```

**Expected Output:**
```
Download complete: large-file.zip
```

**Why this output:** `response.body` is a Web `ReadableStream`. `Readable.fromWeb()` converts it to a Node.js Readable stream, which can then be piped to a file write stream. This allows large files to be downloaded without buffering the entire response in memory.

---

### Sub-Feature 1.3: High-Performance Alternative (`undici`)

#### Definitions

**Core Definition:** `undici` is a high-performance HTTP/1.1 client for Node.js that powers the global `fetch()` API and exposes lower-level APIs (`Client`, `Pool`, `Agent`) for fine-grained connection management.

**Technical Definition:** `undici` is an HTTP/1.1 client written from scratch for Node.js. It implements the WHATWG Fetch Standard, providing `fetch()` together with the `Request`, `Response`, `Headers`, and `FormData` classes that mirror the browser APIs. `undici` also exports a `request()` function and a `Client` class. `Client` extends `Dispatcher` and is the lowest-level dispatcher in undici: it manages exactly one origin over one connection. For pooling across multiple connections use `Pool`, and for routing across multiple origins use `Agent`. When the server supports HTTP/2 and selects it during ALPN negotiation, the same connection is used for HTTP/2. Pipelining is disabled by default.

**Beginner-Friendly Explanation:** `undici` is the engine under the hood of `fetch()`. It's like the difference between driving an automatic car (fetch) and having access to the engine and gearbox (undici). With undici, you can tune connection pools, set per-connection timeouts, and use HTTP/2 — all things that `fetch` abstracts away. For most use cases, `fetch` is enough, but for high-throughput APIs or fine-grained control, `undici` is the better choice.

#### Purposes

- To make high-performance HTTP requests with connection pooling.
- To tune connection behaviour (timeouts, pool sizes, pipelining) for specific workloads.
- To access HTTP/2 when the server supports it via ALPN.
- To implement custom dispatchers and interceptors for resilience and observability.

#### Syntax Rules and Structure

**`undici.request()`:**
```js
import { request } from 'undici';
const { statusCode, body } = await request(url, options);
```

| Option | Description |
|--------|-------------|
| `method` | HTTP method. |
| `body` | Request body. |
| `headers` | Request headers. |
| `dispatcher` | Custom dispatcher (Agent, Pool, Client). |

**`undici.Client` options:**
| Option | Default | Description |
|--------|---------|-------------|
| `connectTimeout` | 10,000 ms | Socket connection timeout. |
| `headersTimeout` | 300,000 ms | Time to receive complete headers. |
| `bodyTimeout` | 300,000 ms | Time to receive body data. |
| `keepAliveTimeout` | 4,000 ms | Idle timeout before socket closes. |
| `keepAliveMaxTimeout` | — | Maximum idle timeout. |
| `pipelining` | 1 | Number of pipelined requests. |

**Constraints and Limitations:**
- Undici does not support the `Expect` request header field.
- Undici always assumes that connections are persistent and will immediately pipeline requests, without checking whether the connection is persistent.
- Pipelining should only be enabled for trusted remote servers.

#### Annotated Code Example

```js
// undici-client.js
import { Client, Agent, setGlobalDispatcher } from 'undici';

// Create a custom Agent with connection pooling
const agent = new Agent({
  connections: 10,          // Pool size per origin
  pipelining: 1,            // Conservative pipelining
  keepAliveTimeout: 60000,  // 60 seconds idle timeout
  keepAliveMaxTimeout: 600000,
  connectTimeout: 5000,     // 5 seconds connect timeout
});

setGlobalDispatcher(agent);

// Use fetch with the custom dispatcher
const response = await fetch('https://jsonplaceholder.typicode.com/posts/1');
const data = await response.json();
console.log('Fetched with custom agent:', data.title);

// Or use the lower-level request API
import { request } from 'undici';
const { statusCode, body } = await request(
  'https://jsonplaceholder.typicode.com/posts/2',
  { method: 'GET' }
);
console.log('Status:', statusCode);
const text = await body.text();
console.log('Body:', JSON.parse(text).title);
```

**Expected Output:**
```
Fetched with custom agent: sunt aut facere repellat provident occaecati excepturi optio reprehenderit
Status: 200
Body: qui est esse
```

**Why this output:** The `Agent` is configured with a pool of 10 connections per origin and a 60-second idle timeout. `setGlobalDispatcher(agent)` applies it to all subsequent `fetch` calls. The `request()` API returns a lower-level response object with `statusCode` and `body`, demonstrating undici's direct API.

#### Real-World Cases

- **High-throughput APIs:** Tuning connection pools for services that make thousands of HTTP calls per second.
- **HTTP/2 usage:** Leveraging HTTP/2 multiplexing via ALPN negotiation.
- **Custom dispatchers:** Implementing interceptors for retries, caching, or logging.

---

## Core Concept 2: Request Configuration & Orchestration

### Sub-Feature 2.1: Setting Custom Headers, User-Agents, and Content-Types

#### Definitions

**Core Definition:** Request headers are key-value pairs sent with an HTTP request that provide metadata about the request, including authentication credentials, content type, and client identification.

**Technical Definition:** HTTP header fields are components of the message header of requests and responses. They define the operating parameters of an HTTP transaction. In Node.js clients, headers are set via the `headers` option in `http.request()`, `fetch()`, or `undici.request()`. The `User-Agent` header identifies the client software, and `Content-Type` indicates the media type of the request body.

**Beginner-Friendly Explanation:** Headers are like the labels on a package. They tell the server who you are (`User-Agent`), what you're sending (`Content-Type`), and what you expect back (`Accept`). Setting custom headers is how you authenticate with APIs, request specific formats, and identify your application to servers.

#### Purposes

- To authenticate requests with API keys or tokens (`Authorization`).
- To specify the expected response format (`Accept`).
- To indicate the format of the request body (`Content-Type`).
- To identify the client software (`User-Agent`).
- To pass custom metadata (correlation IDs, tracing headers).

#### Syntax Rules and Structure

**`fetch()` headers:**
```js
fetch(url, {
  headers: {
    'User-Agent': 'MyApp/1.0',
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token123',
    'Accept': 'application/json',
  },
});
```

**`http.request()` headers:**
```js
http.request({
  headers: {
    'User-Agent': 'MyApp/1.0',
    'Content-Type': 'application/json',
  },
});
```

| Header | Purpose |
|--------|---------|
| `User-Agent` | Identifies the client software. |
| `Content-Type` | Media type of the request body. |
| `Accept` | Media types the client can handle. |
| `Authorization` | Credentials for authentication. |
| `Content-Length` | Size of the body in bytes. |

**Constraints and Limitations:**
- Header names are case-insensitive, but values are case-sensitive.
- Some headers are forbidden or restricted (e.g., `Host`, `Content-Length` in some contexts).
- Invalid header names or values can cause `TypeError` in `fetch()`.

#### Annotated Code Example

```js
// custom-headers.js
async function postJSON(url, data, apiKey) {
  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'User-Agent': 'MyNodeApp/1.0.0',
      'Content-Type': 'application/json',
      'Accept': 'application/json',
      'Authorization': `Bearer ${apiKey}`,
      'X-Request-Id': crypto.randomUUID(),
    },
    body: JSON.stringify(data),
  });

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
  }

  return response.json();
}

postJSON(
  'https://api.example.com/users',
  { name: 'Alice', email: 'alice@example.com' },
  'sk-abc123'
).then(console.log);
```

**Expected Output:**
```
{ id: 42, name: 'Alice', email: 'alice@example.com' }
```

**Why this output:** The request includes a custom `User-Agent`, `Content-Type: application/json`, an `Authorization` bearer token, and a unique `X-Request-Id` for tracing. The server authenticates the request, processes the JSON body, and returns the created user.

#### Real-World Cases

- **API authentication:** Sending `Authorization: Bearer <token>` headers.
- **Content negotiation:** Using `Accept` to request JSON or XML responses.
- **Request tracing:** Adding `X-Request-Id` or `X-Correlation-Id` for distributed tracing.

---

### Sub-Feature 2.2: Streaming Request Bodies vs. Buffering Payloads

#### Definitions

**Core Definition:** Streaming request bodies sends data incrementally as it becomes available, while buffering loads the entire payload into memory before sending.

**Technical Definition:** In `http.request()`, the `ClientRequest` instance is a writable stream — data can be written to it incrementally via `req.write(chunk)` and finalised with `req.end()`. In `fetch()` and `undici.request()`, the `body` option accepts a Readable stream or async iterable. When a streaming body is provided to `fetch()`, the `duplex: 'half'` option must be set. Buffering, by contrast, requires the entire payload to be constructed in memory (e.g., `JSON.stringify(data)`) before the request is sent.

**Beginner-Friendly Explanation:** Imagine you're sending a huge file to a friend. Buffering is like copying the whole file onto a USB stick and then mailing it — you need enough space to hold the whole thing. Streaming is like pouring water through a pipe — it flows continuously without needing a container. For large uploads, streaming is far more memory-efficient.

#### Purposes

- To upload large files or data streams without loading them into memory.
- To send data that is generated progressively (e.g., database query results).
- To reduce memory usage in high-throughput applications.
- To enable real-time data transfer.

#### Syntax Rules and Structure

**Buffered body (fetch):**
```js
fetch(url, {
  method: 'POST',
  body: JSON.stringify(largeObject),
  headers: { 'Content-Type': 'application/json' },
});
```

**Streaming body (fetch):**
```js
import { Readable } from 'node:stream';

const stream = Readable.from(generateData());
fetch(url, {
  method: 'POST',
  body: stream,
  duplex: 'half', // Required for streaming bodies
  headers: { 'Content-Type': 'application/octet-stream' },
});
```

**Streaming body (http.request):**
```js
const req = http.request({ method: 'POST', headers: {...} });
sourceStream.pipe(req);
```

**Constraints and Limitations:**
- `fetch()` requires `duplex: 'half'` when the body is a stream.
- `Content-Length` is not set automatically when the body is a stream; `Transfer-Encoding: chunked` is used instead.
- Streaming bodies cannot be retried without re-creating the stream.

#### Annotated Code Example

```js
// streaming-upload.js
const { Readable } = require('node:stream');
const fs = require('node:fs');

async function uploadFile(filePath, uploadUrl) {
  const fileStream = fs.createReadStream(filePath);

  const response = await fetch(uploadUrl, {
    method: 'PUT',
    body: fileStream,
    duplex: 'half', // Required for streaming request bodies
    headers: {
      'Content-Type': 'application/octet-stream',
      'Content-Length': fs.statSync(filePath).size.toString(),
    },
  });

  if (!response.ok) {
    throw new Error(`Upload failed: HTTP ${response.status}`);
  }

  console.log('Upload complete:', await response.json());
}

// Alternatively, using a generated stream
async function* generateData() {
  for (let i = 0; i < 1000; i++) {
    yield `record-${i}\n`;
  }
}

async function uploadGeneratedData(url) {
  const response = await fetch(url, {
    method: 'POST',
    body: Readable.from(generateData()),
    duplex: 'half',
    headers: { 'Content-Type': 'text/plain' },
  });
  console.log('Generated data uploaded:', response.status);
}

uploadFile('large-file.bin', 'https://upload.example.com/file');
```

**Expected Output:**
```
Upload complete: { id: 'abc123', size: 1048576 }
```

**Why this output:** `fs.createReadStream(filePath)` creates a Node.js Readable stream. `fetch()` accepts this stream as the body with `duplex: 'half'`. The file is streamed to the server in chunks, never fully loaded into memory. The `Content-Length` header is set explicitly because streaming bodies use `Transfer-Encoding: chunked` by default.

#### Real-World Cases

- **File uploads:** Streaming large files to cloud storage (S3, Cloudinary).
- **Data export:** Streaming database query results to an API.
- **Log shipping:** Streaming log files to a remote collector.

---

## Core Concept 3: Response Handling & Resilience

### Sub-Feature 3.1: Consuming Responses (JSON, Text, ArrayBuffers, Streaming)

#### Definitions

**Core Definition:** Response consumption methods read and parse the response body into a specific format — JSON, text, ArrayBuffer, Blob, or a stream.

**Technical Definition:** The `Response` object returned by `fetch()` exposes several body mixins: `.json()`, `.text()`, `.arrayBuffer()`, `.blob()`, and `.formData()`. The `.body` property exposes a Web `ReadableStream` for streaming consumption. Once a mixin has been called, the body cannot be reused. The `Response.ok` property indicates whether the status code is in the 200–299 range, and `Response.status` provides the numeric status code.

**Beginner-Friendly Explanation:** When you get a response from `fetch`, you need to decide how to read it. If it's JSON, use `.json()`. If it's plain text, use `.text()`. If it's an image or binary file, use `.arrayBuffer()` or `.blob()`. For very large responses, use `.body` to stream the data. You can only read the body once — like a letter, once you've opened it, you can't un-open it.

#### Purposes

- To parse JSON responses into JavaScript objects.
- To read text/HTML responses as strings.
- To handle binary data (images, PDFs) as ArrayBuffers or Blobs.
- To stream large responses without buffering.

#### Syntax Rules and Structure

| Method | Returns | Content-Type |
|--------|---------|--------------|
| `response.json()` | Promise\<Object\> | `application/json` |
| `response.text()` | Promise\<string\> | `text/*` |
| `response.arrayBuffer()` | Promise\<ArrayBuffer\> | Any binary |
| `response.blob()` | Promise\<Blob\> | Any binary |
| `response.body` | ReadableStream | Any (streaming) |

**Constraints and Limitations:**
- The body can only be consumed once; calling multiple mixins throws an error.
- `.json()` throws a `SyntaxError` if the body is not valid JSON.
- Large `.text()` or `.json()` calls buffer the entire body in memory; use `.body` for streaming.

#### Annotated Code Example

```js
// response-consumption.js
async function consumeResponses() {
  // JSON
  const jsonRes = await fetch('https://jsonplaceholder.typicode.com/posts/1');
  const json = await jsonRes.json();
  console.log('JSON:', json.title);

  // Text
  const textRes = await fetch('https://example.com/');
  const text = await textRes.text();
  console.log('Text length:', text.length);

  // ArrayBuffer (binary)
  const imgRes = await fetch('https://example.com/image.png');
  const buffer = await imgRes.arrayBuffer();
  console.log('Image bytes:', buffer.byteLength);

  // Streaming
  const streamRes = await fetch('https://example.com/large-file');
  const reader = streamRes.body.getReader();
  let totalBytes = 0;
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    totalBytes += value.byteLength;
  }
  console.log('Streamed bytes:', totalBytes);
}

consumeResponses();
```

**Expected Output:**
```
JSON: sunt aut facere repellat provident occaecati excepturi optio reprehenderit
Text length: 1256
Image bytes: 45678
Streamed bytes: 1048576
```

**Why this output:** Each fetch call uses a different body mixin. `.json()` parses the JSON response. `.text()` reads the HTML as a string. `.arrayBuffer()` returns the raw binary data. For streaming, `.body.getReader()` provides a `ReadableStreamDefaultReader` that reads chunks incrementally.

#### Real-World Cases

- **REST APIs:** Parsing JSON responses from backend services.
- **Web scraping:** Reading HTML text from web pages.
- **Image processing:** Downloading images as ArrayBuffers for processing.
- **Large file downloads:** Streaming responses to disk.

---

### Sub-Feature 3.2: Advanced Timeout Configurations (Connect vs. Read Timeouts)

#### Definitions

**Core Definition:** Connect timeout limits the time allowed to establish a TCP connection, while read timeout (or body timeout) limits the time allowed to receive data after the connection is established.

**Technical Definition:** In `undici`, `connectTimeout` is the timeout, in milliseconds, for establishing a socket connection (default: 10 seconds). `headersTimeout` is the timeout the parser waits to receive the complete HTTP headers (default: 300 seconds). `bodyTimeout` is the timeout after which a request times out while receiving body data (default: 300 seconds). In `fetch()`, timeouts are implemented via `AbortSignal.timeout()` or `AbortController`, which abort the entire request if it takes too long. The `fetch()` API has no built-in timeout; without an `AbortSignal`, a request can hang indefinitely.

**Beginner-Friendly Explanation:** Think of a phone call. The **connect timeout** is how long you let the phone ring before giving up. The **read timeout** is how long you wait for the other person to speak after they've answered. Different timeouts serve different purposes: a short connect timeout fails fast if the server is unreachable, while a longer read timeout accommodates slow but valid responses.

#### Purposes

- To prevent requests from hanging indefinitely.
- To fail fast when a server is unreachable (connect timeout).
- To accommodate slow responses without waiting forever (read timeout).
- To implement different timeout strategies for different endpoints.

#### Syntax Rules and Structure

**Undici `Client` timeouts:**
```js
const client = new Client('https://api.example.com', {
  connectTimeout: 5000,    // 5s to establish connection
  headersTimeout: 10000,   // 10s to receive headers
  bodyTimeout: 30000,      // 30s to receive body
  keepAliveTimeout: 60000, // 60s idle before closing
});
```

**Fetch timeout with AbortSignal:**
```js
const response = await fetch(url, {
  signal: AbortSignal.timeout(10000), // Abort after 10s
});
```

**Manual AbortController:**
```js
const controller = new AbortController();
const timeout = setTimeout(() => controller.abort(), 5000);
try {
  const response = await fetch(url, { signal: controller.signal });
} finally {
  clearTimeout(timeout);
}
```

| Timeout | Scope | Default |
|---------|-------|---------|
| `connectTimeout` | TCP connection establishment | 10,000 ms |
| `headersTimeout` | Waiting for response headers | 300,000 ms |
| `bodyTimeout` | Waiting for body chunks | 300,000 ms |
| `keepAliveTimeout` | Idle socket timeout | 4,000 ms |
| `AbortSignal.timeout(ms)` | Entire request lifecycle | No default |

**Constraints and Limitations:**
- `fetch()` has no built-in timeout; you must use `AbortSignal` or a custom dispatcher.
- `AbortSignal.timeout()` aborts the entire request, not just the connect phase.
- Connect timeouts are not guaranteed to fire with exact millisecond precision.

#### Annotated Code Example

```js
// timeouts.js
import { Agent, setGlobalDispatcher } from 'undici';

// Configure a global agent with granular timeouts
const agent = new Agent({
  connectTimeout: 3000,    // 3s to establish connection
  headersTimeout: 10000,   // 10s to receive headers
  bodyTimeout: 30000,      // 30s to receive body
});
setGlobalDispatcher(agent);

async function fetchWithTimeout(url, timeoutMs) {
  try {
    // AbortSignal.timeout aborts the entire request
    const response = await fetch(url, {
      signal: AbortSignal.timeout(timeoutMs),
    });
    return await response.json();
  } catch (err) {
    if (err.name === 'TimeoutError') {
      console.error(`Request to ${url} timed out after ${timeoutMs}ms`);
    } else {
      console.error('Request failed:', err.message);
    }
    throw err;
  }
}

// Connect timeout: fails after 3s if server unreachable
// Read timeout: aborts after 15s if response is too slow
fetchWithTimeout('https://slow-api.example.com/data', 15000);
```

**Expected Output:**
```
Request to https://slow-api.example.com/data timed out after 15000ms
```

**Why this output:** The undici `Agent` sets a 3-second connect timeout and a 10-second headers timeout. The `AbortSignal.timeout(15000)` adds an overall 15-second timeout. If the server is unreachable, the connect timeout fires first (3s). If the server is reachable but slow to respond, the `AbortSignal` fires at 15s.

#### Real-World Cases

- **Microservices:** Short connect timeouts (1–3s) for internal services with fast failover.
- **Third-party APIs:** Longer read timeouts (30–60s) for APIs that may be slow.
- **Health checks:** Very short timeouts (500ms–2s) for liveness probes.

---

### Sub-Feature 3.3: Resiliency Patterns (Exponential Backoff Retries, Circuit Breakers, and AbortController API)

#### Definitions

**Core Definition:** Resiliency patterns are design strategies that make HTTP clients tolerant of transient failures, including retrying failed requests with exponential backoff, preventing cascading failures with circuit breakers, and cancelling in-flight requests with `AbortController`.

**Technical Definition:** Exponential backoff is a retry strategy where the delay between retries increases exponentially (e.g., 1s, 2s, 4s, 8s) with optional jitter to avoid thundering herd problems. A circuit breaker tracks failure rates and "opens" (stops sending requests) when failures exceed a threshold, giving the failing service time to recover. The `AbortController` API provides a signal that can be passed to `fetch()` or `undici.request()` to cancel the request; `AbortSignal.timeout(ms)` creates a signal that aborts after a specified time.

**Beginner-Friendly Explanation:** **Exponential backoff** is like knocking on a door: if no one answers, you wait a bit, then longer, then even longer, instead of knocking continuously. A **circuit breaker** is like a fuse in your electrical panel: if too many things go wrong, it trips and cuts off power to prevent a fire. **AbortController** is like a cancel button — if a request is taking too long or you no longer need the result, you can cancel it.

#### Purposes

- To recover from transient network failures automatically.
- To avoid overwhelming a failing service with retries.
- To prevent cascading failures in distributed systems.
- To cancel requests that are no longer needed.
- To implement timeout and cancellation uniformly across HTTP clients.

#### Syntax Rules and Structure

**Exponential backoff retry:**
```js
async function fetchWithRetry(url, options, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const response = await fetch(url, options);
      if (response.ok) return response;
      if (response.status >= 500) throw new Error(`Server error: ${response.status}`);
      return response; // Don't retry 4xx errors
    } catch (err) {
      if (attempt === maxRetries - 1) throw err;
      const delay = Math.min(1000 * 2 ** attempt + Math.random() * 1000, 30000);
      await new Promise((r) => setTimeout(r, delay));
    }
  }
}
```

**Circuit breaker (using `opossum`):**
```js
const CircuitBreaker = require('opossum');

const breaker = new CircuitBreaker(async (url) => {
  const response = await fetch(url);
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}, {
  timeout: 5000,           // 5s timeout per attempt
  errorThresholdPercentage: 50, // Open after 50% failures
  resetTimeout: 30000,     // Try again after 30s
});

breaker.fallback(() => ({ cached: true }));
```

**AbortController:**
```js
const controller = new AbortController();
const timeout = setTimeout(() => controller.abort(), 5000);

try {
  const response = await fetch(url, { signal: controller.signal });
  // Process response
} catch (err) {
  if (err.name === 'AbortError') console.error('Request cancelled');
} finally {
  clearTimeout(timeout);
}
```

**Constraints and Limitations:**
- Retries should only be applied to idempotent methods (GET, PUT, DELETE) or when the server supports idempotency keys.
- Circuit breakers require careful tuning of thresholds and reset timeouts.
- `AbortController` aborts the request entirely; the stream cannot be resumed.

#### Annotated Code Example

```js
// resilient-fetch.js
const CircuitBreaker = require('opossum');

// Circuit breaker configuration
const breaker = new CircuitBreaker(
  async (url) => {
    const response = await fetch(url, {
      signal: AbortSignal.timeout(5000), // Per-attempt timeout
    });
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
  },
  {
    timeout: 5000,                   // 5s timeout
    errorThresholdPercentage: 50,    // Open after 50% failures
    resetTimeout: 30000,             // Try again after 30s
    volumeThreshold: 5,              // Minimum 5 requests before opening
  }
);

// Fallback when circuit is open
breaker.fallback(() => ({ error: 'Service temporarily unavailable' }));

// Event listeners for observability
breaker.on('open', () => console.log('Circuit opened'));
breaker.on('halfOpen', () => console.log('Circuit half-open'));
breaker.on('close', () => console.log('Circuit closed'));

async function fetchWithResilience(url) {
  try {
    const data = await breaker.fire(url);
    console.log('Data:', data);
    return data;
  } catch (err) {
    console.error('Request failed:', err.message);
  }
}

// Simulate multiple calls
fetchWithResilience('https://jsonplaceholder.typicode.com/posts/1');
fetchWithResilience('https://jsonplaceholder.typicode.com/posts/2');
```

**Expected Output:**
```
Data: { userId: 1, id: 1, title: '...', body: '...' }
Data: { userId: 1, id: 2, title: '...', body: '...' }
Circuit closed
```

**Why this output:** The `CircuitBreaker` wraps the fetch call with a 5-second timeout and a 50% failure threshold. When the circuit is closed (normal operation), requests pass through. If failures exceed the threshold, the circuit opens and the fallback is invoked. The event listeners log circuit state changes for observability.

#### Real-World Cases

- **Third-party API calls:** Retrying transient failures with exponential backoff.
- **Microservice communication:** Circuit breakers to prevent cascading failures.
- **User-initiated requests:** Aborting fetches when the user navigates away or cancels.

---

### Sub-Feature 3.4: Global Agent Configurations and Connection Pooling

#### Definitions

**Core Definition:** Connection pooling reuses TCP connections across multiple HTTP requests to the same origin, reducing latency and resource consumption. The `Agent` class (in both `http` and `undici`) manages connection persistence and reuse.

**Technical Definition:** An `Agent` is responsible for managing connection persistence and reuse for HTTP clients. It maintains a queue of pending requests for a given host and port, reusing a single socket connection for each until the queue is empty, at which time the socket is either destroyed or put into a pool where it is kept to be used again for requests to the same host and port. Whether it is destroyed or pooled depends on the `keepAlive` option. In undici, an `Agent` dispatches requests against multiple different origins, creating and reusing a per-origin `Pool` (or `Client`) on demand. The per-origin `Pool` uses the default unlimited connections, so concurrent requests to the same origin are spread across separate `Client` instances.

**Beginner-Friendly Explanation:** Connection pooling is like keeping a taxi waiting outside your office instead of calling a new one every time you need a ride. The first request establishes a connection, and subsequent requests reuse it — saving the time and overhead of the TCP and TLS handshake. The `Agent` is the dispatcher that manages this pool.

#### Purposes

- To reduce latency by reusing established connections.
- To limit the number of concurrent connections to a server.
- To improve throughput in high-volume applications.
- To manage resource consumption on both client and server.

#### Syntax Rules and Structure

**Undici `Agent` configuration:**
```js
import { Agent, setGlobalDispatcher } from 'undici';

const agent = new Agent({
  connections: 100,        // Max connections per origin
  pipelining: 1,           // Requests per connection
  keepAliveTimeout: 60000, // 60s idle timeout
  keepAliveMaxTimeout: 600000,
});

setGlobalDispatcher(agent);
```

**Legacy `http.Agent`:**
```js
const http = require('node:http');

http.globalAgent = new http.Agent({
  keepAlive: true,
  maxSockets: 50,
  maxFreeSockets: 10,
  timeout: 60000,
});
```

| Option | Description |
|--------|-------------|
| `connections` | Max connections per origin (undici). |
| `maxSockets` | Max sockets per host (legacy). |
| `keepAlive` | Enable persistent connections (legacy). |
| `keepAliveTimeout` | Idle timeout before closing. |
| `maxOrigins` | Max distinct origins (undici Agent). |

**Constraints and Limitations:**
- Unlimited connections can exhaust server resources; set a reasonable limit.
- Idle connections consume memory and file descriptors; tune `keepAliveTimeout`.
- Legacy `http.globalAgent` must be replaced to change pooling defaults.

#### Annotated Code Example

```js
// connection-pooling.js
import { Agent, setGlobalDispatcher, fetch } from 'undici';

// Configure a global agent with connection pooling
const agent = new Agent({
  connections: 50,           // Max 50 connections per origin
  pipelining: 1,             // No pipelining (safe default)
  keepAliveTimeout: 30000,   // 30s idle timeout
  keepAliveMaxTimeout: 300000,
  connectTimeout: 5000,
});

// Apply globally to all fetch calls
setGlobalDispatcher(agent);

async function fetchMany(urls) {
  const start = Date.now();

  // Concurrent requests will reuse pooled connections
  const results = await Promise.all(
    urls.map((url) =>
      fetch(url).then((r) => {
        if (!r.ok) throw new Error(`HTTP ${r.status}`);
        return r.json();
      })
    )
  );

  const elapsed = Date.now() - start;
  console.log(`Fetched ${results.length} URLs in ${elapsed}ms`);
  return results;
}

// 10 requests to the same origin — connections are reused
const urls = Array.from({ length: 10 }, (_, i) =>
  `https://jsonplaceholder.typicode.com/posts/${i + 1}`
);

fetchMany(urls);
```

**Expected Output:**
```
Fetched 10 URLs in 245ms
```

**Why this output:** The `Agent` is configured with a pool of 50 connections per origin. The 10 concurrent `fetch` calls to the same origin reuse connections from the pool, avoiding the overhead of establishing 10 separate TCP/TLS connections. The total time is significantly less than if each request required a new connection.

#### Real-World Cases

- **API clients:** Reusing connections to a backend API for high-throughput calls.
- **Web scrapers:** Managing connections to multiple domains with per-origin pools.
- **Microservice gateways:** Tuning connection pools to match backend capacity.

---

## References

- Node.js Documentation — HTTP — https://nodejs.org/api/http.html
- Node.js Documentation — `http.request()` — https://nodejs.org/api/http.html#httprequestoptions-callback
- Node.js Documentation — HTTPS — https://nodejs.org/api/https.html
- Node.js Documentation — `https.request()` — https://nodejs.org/api/https.html#httpsrequestoptions-callback
- Node.js Documentation — `http.Agent` — https://nodejs.org/api/http.html#class-httpagent
- Node.js Documentation — Global `fetch()` — https://nodejs.org/api/globals.html#fetch
- Node.js Documentation — Web Streams API — https://nodejs.org/api/webstreams.html
- Undici Documentation — Getting Started — https://undici.nodejs.org/#/
- Undici Documentation — `Client` — https://undici.nodejs.org/api/Client
- Undici Documentation — `Agent` — https://undici.nodejs.org/api/Agent
- Undici Documentation — `Pool` — https://undici.nodejs.org/api/Pool
- Undici Documentation — Fetch — https://undici.nodejs.org/api/Fetch
- MDN Web Docs — `fetch()` — https://developer.mozilla.org/en-US/docs/Web/API/fetch
- MDN Web Docs — `Response` — https://developer.mozilla.org/en-US/docs/Web/API/Response
- MDN Web Docs — `AbortController` — https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- MDN Web Docs — `AbortSignal.timeout()` — https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal/timeout_static
- Opossum — Circuit Breaker for Node.js — https://github.com/nodeshift/opossum
- AWS SDK for JavaScript v3 — `@aws-sdk/lib-storage` — https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/modules/_aws_sdk_lib_storage.html