# HTTP Responses — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** HTTP response status codes are three-digit integers returned by a server in response to a client's request, indicating whether the request was successfully completed, requires further action, or encountered an error.

**Technical Definition:** HTTP response status codes are standardized by the IETF in RFC 9110 (HTTP Semantics). They are grouped into five classes based on the first digit: 1xx (Informational), 2xx (Success), 3xx (Redirection), 4xx (Client Error), and 5xx (Server Error). Each code carries specific semantic meaning that both clients and servers use to communicate the outcome of a request. In Express.js, status codes are set using `res.status(code)` or `res.sendStatus(code)`.

**Beginner-Friendly Explanation:** When you visit a website or call an API, the server responds with a three-digit number that tells the client what happened. Think of it like a traffic light system: 2xx means "green light, everything worked," 3xx means "yellow light, you need to go somewhere else," 4xx means "red light, you made a mistake," and 5xx means "the server has a problem." The first digit tells you the category, and the full number tells you the specific situation.

### Key Characteristics

- **Five categories:** 1xx, 2xx, 3xx, 4xx, and 5xx, each with a distinct purpose.
- **Standardized:** Defined by RFC 9110 and maintained by IANA's HTTP Status Code Registry.
- **Semantic:** Each code carries specific meaning that clients interpret automatically.
- **Express integration:** Status codes are set via `res.status(code)` or `res.sendStatus(code)`.
- **Chainable:** In Express, `res.status()` returns the response object, enabling method chaining.
- **Default 200:** Express defaults to 200 OK if no status is explicitly set.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript knowledge:** Functions, objects, and asynchronous code.
- **Understanding of HTTP:** Requests, responses, headers, and methods.

### Related Programming Areas

- **REST API design:** Status codes are the primary way to communicate API outcomes.
- **Error handling:** 4xx and 5xx codes drive error-handling middleware.
- **Caching:** 304 Not Modified enables efficient cache validation.
- **Authentication:** 401 and 403 distinguish between unauthenticated and unauthorised requests.
- **Rate limiting:** 429 Too Many Requests enforces API quotas.
- **Redirection:** 301, 302, 307, and 308 handle URL migration and post-submission redirects.

### Core Concepts

1. **1xx Informational** — provisional responses indicating the request is being processed.
2. **2xx Success** — the request was received, understood, and accepted.
3. **3xx Redirection** — further action is needed to complete the request.
4. **4xx Client Error** — the request contains bad syntax or cannot be fulfilled.
5. **5xx Server Error** — the server failed to fulfill a valid request.

---

## Core Concept 1: 1xx Informational Responses

### Definitions

**Core Definition:** 1xx status codes indicate that the server has received the request and is continuing to process it. These are provisional responses that precede the final response.

**Technical Definition:** 1xx responses are informational and indicate that the client should continue the request or that the server is switching protocols. Unlike final responses, 1xx responses do not contain a body. They are primarily used for protocol-level negotiation and performance optimisation.

**Beginner-Friendly Explanation:** Think of 1xx responses as the server saying "hold on, I'm working on it" or "let's switch to a better way of talking." They're not the final answer — just a heads-up that the request is being handled.

### Purposes

- To indicate that the server has received the request headers and the client should proceed to send the body.
- To confirm a protocol upgrade (e.g., HTTP to WebSocket).
- To send preliminary headers before the final response, enabling resource preloading.
- To improve performance by allowing clients to begin work before the full response arrives.

---

### Sub-Feature 1.1: 100 Continue

#### Definitions

**Core Definition:** 100 Continue indicates that the server has received the initial part of the request and the client should proceed to send the remainder of the request body.

**Technical Definition:** The 100 Continue status code is sent in response to an `Expect: 100-continue` header from the client. It tells the client that the request headers are acceptable and the client should send the request body. If the server would reject the request based on the headers alone, it can send a 4xx response instead of 100 Continue, saving bandwidth.

**Beginner-Friendly Explanation:** Imagine you're sending a large package. Before you ship it, you ask the recipient: "Can you accept this?" If they say yes (100 Continue), you send the package. If they say no, you save the shipping cost.

#### Purposes

- To tell the client that the request headers are acceptable and to proceed with the body.
- To avoid sending large request bodies when the server would reject them based on headers.
- To optimise bandwidth usage in high-latency networks.

#### Syntax Rules and Structure

In Express/Node.js, 100 Continue is typically handled automatically by the HTTP server. To send it manually:

```js
res.writeContinue();
```

| Component | Breakdown |
|-----------|-----------|
| `res.writeContinue()` | Sends a 100 Continue response to the client. |
| Usage | Call before processing the request body. |

**Rules:**
- Sent in response to an `Expect: 100-continue` header.
- Contains no body.
- Node.js's HTTP server handles this automatically in most cases.

#### Constraints and Limitations

- Not all clients use the `Expect: 100-continue` mechanism.
- Must be sent before any other response data.
- Cannot be sent after a final response has begun.

#### Annotated Code Example

```js
// 100-continue.js
const express = require('express');
const app = express();

app.post('/upload', (req, res) => {
  // Node.js/Express handles 100 Continue automatically
  // when the client sends "Expect: 100-continue"
  console.log('Headers received, continuing to read body...');

  let body = '';
  req.on('data', chunk => { body += chunk; });
  req.on('end', () => {
    res.json({ received: body.length, message: 'Upload complete' });
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for a client sending `Expect: 100-continue` followed by a body):**
```
HTTP/1.1 100 Continue

HTTP/1.1 200 OK
Content-Type: application/json

{"received":1024,"message":"Upload complete"}
```

**Why this output:** The client sends the request headers with `Expect: 100-continue`. The server automatically responds with 100 Continue, prompting the client to send the body. After receiving the body, the server sends the final 200 OK response.

#### Real-World Cases

- **Large file uploads:** Clients ask for permission before uploading gigabytes of data.
- **API requests with large payloads:** Avoid sending rejected bodies over slow connections.
- **IoT devices:** Constrained devices verify acceptance before transmitting sensor data.

---

### Sub-Feature 1.2: 101 Switching Protocols

#### Definitions

**Core Definition:** 101 Switching Protocols indicates that the server is switching to the protocol requested by the client in the `Upgrade` header.

**Technical Definition:** The 101 status code is sent in response to an `Upgrade` request header from the client. It indicates the protocol the server is switching to. This is most commonly used for upgrading an HTTP connection to a WebSocket connection. The `Connection: Upgrade` and `Upgrade: <protocol>` headers must be present.

**Beginner-Friendly Explanation:** Imagine you're talking on a walkie-talkie and someone asks to switch to a phone call. If you agree, you both switch to the phone. 101 Switching Protocols is that agreement — the server says "yes, let's switch to a better protocol for this conversation."

#### Purposes

- To confirm a protocol upgrade (HTTP to WebSocket, HTTP/2, etc.).
- To establish persistent, bidirectional communication channels.
- To enable real-time features (chat, live updates, gaming).

#### Syntax Rules and Structure

```js
// WebSocket upgrade (typically handled by a library like ws)
server.on('upgrade', (req, socket, head) => {
  // Perform handshake and respond with 101
  socket.write(
    'HTTP/1.1 101 Switching Protocols\r\n' +
    'Upgrade: websocket\r\n' +
    'Connection: Upgrade\r\n' +
    '\r\n'
  );
});
```

| Component | Breakdown |
|-----------|-----------|
| `Upgrade: websocket` | The protocol being switched to. |
| `Connection: Upgrade` | Indicates the connection is being upgraded. |

**Rules:**
- Must include `Upgrade` and `Connection: Upgrade` headers.
- Sent only in response to an `Upgrade` request header.
- After 101, the connection uses the new protocol.

#### Constraints and Limitations

- Only one protocol upgrade per connection.
- Not all intermediaries (proxies, load balancers) support protocol upgrades.
- WebSocket libraries (e.g., `ws`, `socket.io`) handle this automatically.

#### Annotated Code Example

```js
// 101-switching.js
const http = require('http');
const crypto = require('crypto');

const server = http.createServer((req, res) => {
  res.writeHead(200);
  res.end('Regular HTTP response');
});

// WebSocket upgrade handling
server.on('upgrade', (req, socket) => {
  const key = req.headers['sec-websocket-key'];
  const acceptKey = crypto
    .createHash('sha1')
    .update(key + '258EAFA5-E914-47DA-95CA-C5AB0DC85B11')
    .digest('base64');

  socket.write(
    'HTTP/1.1 101 Switching Protocols\r\n' +
    'Upgrade: websocket\r\n' +
    'Connection: Upgrade\r\n' +
    `Sec-WebSocket-Accept: ${acceptKey}\r\n` +
    '\r\n'
  );

  // Now the socket speaks WebSocket protocol
  console.log('WebSocket connection established');
});

server.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for a WebSocket upgrade request):**
```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

**Why this output:** The server receives the WebSocket handshake request, computes the `Sec-WebSocket-Accept` key, and responds with 101. The connection is then upgraded to the WebSocket protocol.

#### Real-World Cases

- **Real-time chat applications:** WebSocket connections for instant messaging.
- **Live sports updates:** Persistent connections pushing score updates.
- **Multiplayer games:** Real-time bidirectional game state synchronisation.
- **Collaborative editing:** Live cursor and document updates.

---

### Sub-Feature 1.3: 103 Early Hints

#### Definitions

**Core Definition:** 103 Early Hints allows the server to send preliminary response headers before the final response, enabling the client to begin preloading resources.

**Technical Definition:** The 103 Early Hints status code is primarily intended to be used with the `Link` header, letting the user agent start preloading resources while the server prepares a response. It is defined in RFC 8297. The server can send one or more 103 responses before the final response.

**Beginner-Friendly Explanation:** Imagine ordering food at a restaurant. Before your meal is ready, the waiter brings you bread and tells you what's coming. 103 Early Hints is like that — the server tells the browser "here are some resources you'll need, start loading them now" while it prepares the full response.

#### Purposes

- To send preliminary headers before the final response.
- To enable resource preloading (CSS, JavaScript, fonts).
- To improve perceived performance by reducing time-to-first-render.
- To optimise the critical rendering path.

#### Syntax Rules and Structure

```js
// Sending 103 Early Hints
res.writeEarlyHints({
  link: '</styles.css>; rel=preload; as=style, </script.js>; rel=preload; as=script'
});
// Then send the final response
res.send('<html>...</html>');
```

| Component | Breakdown |
|-----------|-----------|
| `res.writeEarlyHints()` | Sends a 103 response with headers. |
| `link` | The `Link` header with preload directives. |
| `rel=preload` | Tells the browser to preload the resource. |
| `as=style` | Specifies the resource type. |

**Rules:**
- Only `Link` headers are typically included.
- Can be sent multiple times before the final response.
- Requires Node.js v18.11.0+ or a framework that supports it.

#### Constraints and Limitations

- Not all clients support 103 Early Hints.
- Adds complexity to response handling.
- The final response must still include the actual resources.

#### Annotated Code Example

```js
// 103-early-hints.js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  // Send 103 Early Hints with preload directives
  res.writeEarlyHints({
    link: [
      '</styles/main.css>; rel=preload; as=style',
      '</scripts/app.js>; rel=preload; as=script'
    ].join(', ')
  });

  // Simulate slow data fetching
  setTimeout(() => {
    res.send(`
      <!DOCTYPE html>
      <html>
      <head>
        <link rel="stylesheet" href="/styles/main.css">
      </head>
      <body>
        <h1>Page loaded</h1>
        <script src="/scripts/app.js"></script>
      </body>
      </html>
    `);
  }, 1000);
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /`):**
```
HTTP/1.1 103 Early Hints
Link: </styles/main.css>; rel=preload; as=style, </scripts/app.js>; rel=preload; as=script

HTTP/1.1 200 OK
Content-Type: text/html

<!DOCTYPE html>...
```

**Why this output:** The server sends 103 Early Hints with preload directives. The browser starts downloading `/styles/main.css` and `/scripts/app.js` immediately. After the simulated delay, the final 200 OK response is sent.

#### Real-World Cases

- **Content-heavy websites:** Preloading critical CSS and JavaScript.
- **E-commerce:** Preloading product images while the page renders.
- **News sites:** Preloading fonts and stylesheets for faster text rendering.
- **SPAs:** Preloading JavaScript bundles while the HTML is being generated.

---

## Core Concept 2: 2xx Success Responses

### Definitions

**Core Definition:** 2xx status codes indicate that the client's request was successfully received, understood, and accepted by the server.

**Technical Definition:** 2xx responses confirm that the action requested by the client was successfully completed. The specific code within the 2xx range indicates the nature of the success (e.g., resource fetched, resource created, request accepted for processing).

**Beginner-Friendly Explanation:** 2xx codes are the "green light" responses. Everything worked as expected. The server did what the client asked.

### Purposes

- To confirm successful resource retrieval, creation, or modification.
- To indicate that a request has been accepted for asynchronous processing.
- To signal that no response body is needed (e.g., after a DELETE).
- To support partial content delivery for streaming and resumable downloads.

---

### Sub-Feature 2.1: 200 OK

#### Definitions

**Core Definition:** 200 OK indicates that the request succeeded. The meaning of "success" depends on the HTTP method used.

**Technical Definition:** The 200 OK status code indicates that the request has succeeded. The response body contains the result of the action. For GET, the resource is in the body. For POST or PUT, the result of the action is described in the body.

**Beginner-Friendly Explanation:** 200 OK is the standard "everything worked" response. The server found what you asked for and is sending it back.

#### Purposes

- To confirm successful resource retrieval, update, or general request execution.
- To return the requested data in the response body.
- To serve as the default success status for most HTTP methods.

#### Syntax Rules and Structure

```js
res.status(200).json({ data: 'success' });
res.status(200).send('OK');
```

| Component | Breakdown |
|-----------|-----------|
| `res.status(200)` | Sets the status code to 200. |
| `.json()` / `.send()` | Sends the response body. |

#### Annotated Code Example

```js
// 200-ok.js
const express = require('express');
const app = express();

app.get('/api/users', (req, res) => {
  const users = [{ id: 1, name: 'Alice' }];
  res.status(200).json(users);  // 200 is default, explicit for clarity
});

app.get('/health', (req, res) => {
  res.status(200).send('OK');
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/users`):**
```
HTTP/1.1 200 OK
Content-Type: application/json

[{"id":1,"name":"Alice"}]
```

**Why this output:** The request succeeded, and the JSON array is returned. The status code 200 confirms success.

#### Real-World Cases

- **API data retrieval:** `GET /api/products` returns the product list.
- **Health checks:** `GET /health` returns `200 OK` to indicate the service is running.
- **Form submissions:** `POST /contact` returns 200 with a success message.

---

### Sub-Feature 2.2: 201 Created

#### Definitions

**Core Definition:** 201 Created indicates that the request succeeded and a new resource was created as a result.

**Technical Definition:** The 201 Created status code indicates that the request has been fulfilled and has resulted in one or more new resources being created. The newly created resource is typically returned in the response body, and the `Location` header may contain the URI of the new resource.

**Beginner-Friendly Explanation:** When you create something new — like registering a user or posting a comment — the server responds with 201 to say "done, and here's the new thing I made."

#### Purposes

- To confirm successful creation of a resource.
- To return the newly created resource or its URI location.
- To provide the client with information for subsequent access.

#### Syntax Rules and Structure

```js
app.post('/api/users', (req, res) => {
  const newUser = createUser(req.body);
  res.status(201)
     .location(`/api/users/${newUser.id}`)
     .json(newUser);
});
```

| Component | Breakdown |
|-----------|-----------|
| `res.status(201)` | Sets the status code to 201. |
| `.location()` | Sets the `Location` header. |
| `.json()` | Sends the new resource. |

#### Annotated Code Example

```js
// 201-created.js
const express = require('express');
const app = express();
app.use(express.json());

let users = [];
let nextId = 1;

app.post('/api/users', (req, res) => {
  const newUser = { id: nextId++, ...req.body };
  users.push(newUser);

  res.status(201)
     .location(`/api/users/${newUser.id}`)
     .json(newUser);
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `POST /api/users` with `{ "name": "Alice" }`):**
```
HTTP/1.1 201 Created
Location: /api/users/1
Content-Type: application/json

{"id":1,"name":"Alice"}
```

**Why this output:** The request created a new user with ID 1. The server responds with 201, includes the `Location` header pointing to the new resource, and returns the user object.

#### Real-World Cases

- **User registration:** `POST /api/register` returns 201 with the new user.
- **Blog post creation:** `POST /api/posts` returns 201 with the post data.
- **Order placement:** `POST /api/orders` returns 201 with the order confirmation.

---

### Sub-Feature 2.3: 202 Accepted

#### Definitions

**Core Definition:** 202 Accepted indicates that the request has been accepted for processing, but the processing is not yet complete.

**Technical Definition:** The 202 Accepted status code indicates that the request has been accepted for processing, but the processing has not been completed. The request might or might not eventually be acted upon, as it might be disallowed when processing actually takes place. This is commonly used for asynchronous operations.

**Beginner-Friendly Explanation:** Imagine ordering a custom-made item. The shop says "we've received your order and will start working on it, but it's not ready yet." 202 Accepted is that acknowledgment — the request is in the queue.

#### Purposes

- To acknowledge receipt of a request that will be processed asynchronously.
- To indicate that processing is not yet complete.
- To provide a reference for checking the status later.

#### Syntax Rules and Structure

```js
app.post('/api/reports', (req, res) => {
  const jobId = queueReportGeneration(req.body);
  res.status(202).json({ jobId, status: 'pending' });
});
```

| Component | Breakdown |
|-----------|-----------|
| `res.status(202)` | Sets the status code to 202. |
| `jobId` | Identifier for tracking the job. |

#### Annotated Code Example

```js
// 202-accepted.js
const express = require('express');
const app = express();

const jobs = new Map();

app.post('/api/reports', (req, res) => {
  const jobId = `job-${Date.now()}`;
  jobs.set(jobId, { status: 'pending', createdAt: new Date() });

  // Simulate async processing
  setTimeout(() => {
    jobs.set(jobId, { status: 'completed', result: 'report.pdf' });
  }, 5000);

  res.status(202).json({
    jobId,
    status: 'pending',
    message: 'Report generation started'
  });
});

app.get('/api/reports/:jobId', (req, res) => {
  const job = jobs.get(req.params.jobId);
  if (!job) return res.status(404).json({ error: 'Job not found' });
  res.json(job);
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `POST /api/reports`):**
```
HTTP/1.1 202 Accepted
Content-Type: application/json

{"jobId":"job-1234567890","status":"pending","message":"Report generation started"}
```

**Expected Output (for `GET /api/reports/job-1234567890` after 5 seconds):**
```
{"status":"completed","result":"report.pdf"}
```

**Why this output:** The request is accepted (202) and a job ID is returned. The client can poll the status endpoint to check when the report is ready. After 5 seconds, the job status changes to "completed".

#### Real-World Cases

- **Report generation:** Large reports processed in the background.
- **Video encoding:** Uploading a video returns 202 with a job ID.
- **Batch processing:** Bulk operations queued for later processing.
- **Email sending:** Acknowledging that an email has been queued.

---

### Sub-Feature 2.4: 204 No Content

#### Definitions

**Core Definition:** 204 No Content indicates that the server successfully processed the request but is not returning any content.

**Technical Definition:** The 204 No Content status code indicates that the server has fulfilled the request but does not need to return an entity-body. The response may include headers, but the body must be empty. This is commonly used for DELETE operations and form submissions where the client should remain on the same page.

**Beginner-Friendly Explanation:** When you delete something, there's nothing to show. 204 means "done, and there's nothing to send back."

#### Purposes

- To confirm successful action where no response body is required.
- To signal completion of DELETE operations.
- To avoid sending unnecessary data.

#### Syntax Rules and Structure

```js
res.sendStatus(204);  // Sends 204 with "No Content" text
// OR
res.status(204).end();  // Sends 204 with no body
```

| Component | Breakdown |
|-----------|-----------|
| `res.sendStatus(204)` | Sets status 204 and sends "No Content". |
| `res.status(204).end()` | Sets status 204 and ends with no body. |

#### Annotated Code Example

```js
// 204-no-content.js
const express = require('express');
const app = express();

let items = [{ id: 1, name: 'Item 1' }, { id: 2, name: 'Item 2' }];

app.delete('/api/items/:id', (req, res) => {
  const index = items.findIndex(i => i.id === parseInt(req.params.id));
  if (index === -1) {
    return res.status(404).json({ error: 'Item not found' });
  }

  items.splice(index, 1);
  res.sendStatus(204);  // No body needed
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `DELETE /api/items/1`):**
```
HTTP/1.1 204 No Content
```

**Why this output:** The item was successfully deleted. The server responds with 204 and no body. The client knows the operation succeeded based on the status code alone.

#### Real-World Cases

- **DELETE endpoints:** `DELETE /api/posts/:id` returns 204 after deletion.
- **Logout:** `POST /api/logout` returns 204 after invalidating the session.
- **Form submissions:** `POST /api/settings` returns 204 to keep the user on the same page.

---

### Sub-Feature 2.5: 206 Partial Content

#### Definitions

**Core Definition:** 206 Partial Content indicates that the server is delivering only part of the resource due to a `Range` header sent by the client.

**Technical Definition:** The 206 Partial Content status code is used when the client sends a `Range` header requesting only part of a resource. The response includes a `Content-Range` header indicating which portion of the resource is being returned. This is essential for video streaming, audio playback, and resumable downloads.

**Beginner-Friendly Explanation:** Imagine watching a video online. Instead of downloading the entire video before playing, the browser requests small chunks. 206 Partial Content is the server saying "here's the chunk you asked for."

#### Purposes

- To deliver only the requested portion of a resource.
- To enable video and audio streaming.
- To support resumable downloads after interruption.
- To optimise bandwidth for large files.

#### Syntax Rules and Structure

```js
app.get('/video', (req, res) => {
  const range = req.headers.range;
  const fileSize = getFileSize('video.mp4');
  const { start, end } = parseRange(range, fileSize);

  res.status(206);
  res.set('Content-Range', `bytes ${start}-${end}/${fileSize}`);
  res.set('Content-Length', end - start + 1);
  res.set('Accept-Ranges', 'bytes');

  fs.createReadStream('video.mp4', { start, end }).pipe(res);
});
```

| Component | Breakdown |
|-----------|-----------|
| `Content-Range` | Specifies the byte range returned. |
| `Content-Length` | Size of the returned chunk. |
| `Accept-Ranges: bytes` | Indicates the server supports range requests. |

#### Annotated Code Example

```js
// 206-partial.js
const express = require('express');
const fs = require('fs');
const app = express();

app.get('/stream', (req, res) => {
  const range = req.headers.range;
  const filePath = './large-video.mp4';
  const stat = fs.statSync(filePath);
  const fileSize = stat.size;

  if (!range) {
    // No range header — send the whole file
    res.writeHead(200, { 'Content-Length': fileSize });
    fs.createReadStream(filePath).pipe(res);
    return;
  }

  const parts = range.replace(/bytes=/, '').split('-');
  const start = parseInt(parts[0], 10);
  const end = parts[1] ? parseInt(parts[1], 10) : fileSize - 1;
  const chunkSize = end - start + 1;

  res.writeHead(206, {
    'Content-Range': `bytes ${start}-${end}/${fileSize}`,
    'Accept-Ranges': 'bytes',
    'Content-Length': chunkSize,
    'Content-Type': 'video/mp4'
  });

  fs.createReadStream(filePath, { start, end }).pipe(res);
});

app.listen(3000, () => console.log('Streaming server on 3000'));
```

**Expected Output (for `GET /stream` with `Range: bytes=0-1023`):**
```
HTTP/1.1 206 Partial Content
Content-Range: bytes 0-1023/10485760
Accept-Ranges: bytes
Content-Length: 1024
Content-Type: video/mp4

(binary chunk)
```

**Why this output:** The client requested the first 1024 bytes of the file. The server responds with 206, includes the `Content-Range` header specifying the byte range, and streams only that portion of the file.

#### Real-World Cases

- **Video streaming:** YouTube, Netflix, and other video platforms use range requests.
- **Audio playback:** Music streaming services request chunks as needed.
- **Resumable downloads:** Download managers resume interrupted downloads.
- **PDF viewers:** Loading specific pages without downloading the entire document.

---

## Core Concept 3: 3xx Redirection Responses

### Definitions

**Core Definition:** 3xx status codes indicate that further action must be taken by the client to fulfill the request, typically redirecting to a different URL.

**Technical Definition:** 3xx responses indicate that the target resource has moved, either temporarily or permanently, or that the client should use a cached version of the resource. The `Location` header in the response contains the new URL.

**Beginner-Friendly Explanation:** 3xx codes are like mail forwarding. The server says "the thing you're looking for isn't here anymore, but I know where it went — go there instead."

### Purposes

- To redirect clients to a different URL.
- To indicate that a resource has moved permanently or temporarily.
- To tell clients to use their cached version of a resource.
- To preserve HTTP method and body during redirects.

---

### Sub-Feature 3.1: 301 Moved Permanently

#### Definitions

**Core Definition:** 301 Moved Permanently indicates that the target resource has been assigned a new permanent URI, and future references should use this URI.

**Technical Definition:** The 301 status code indicates that the target resource has been assigned a new permanent URI, and any future references to this resource ought to use one of the enclosed URIs. Clients with link editing capabilities should automatically relink references to the new URI. The `Location` header contains the new URL.

**Beginner-Friendly Explanation:** 301 is like a permanent change of address. If you move to a new house, you tell the post office to forward all your mail — and update your address with everyone. Search engines update their indexes to point to the new URL.

#### Purposes

- To redirect clients to a new permanent URL.
- To preserve SEO rankings when moving content.
- To consolidate duplicate content under a single URL.

#### Syntax Rules and Structure

```js
res.redirect(301, '/new-path');
```

| Component | Breakdown |
|-----------|-----------|
| `res.redirect(301, url)` | Sends a 301 redirect to the specified URL. |
| `Location` | Set automatically by Express. |

#### Annotated Code Example

```js
// 301-permanent.js
const express = require('express');
const app = express();

app.get('/old-blog-post', (req, res) => {
  res.redirect(301, '/blog/new-url');
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /old-blog-post`):**
```
HTTP/1.1 301 Moved Permanently
Location: /blog/new-url
```

**Why this output:** The old URL permanently redirects to the new one. Browsers and search engines cache this redirect and use the new URL for all future requests.

#### Real-World Cases

- **URL migration:** Moving from HTTP to HTTPS.
- **Domain change:** Redirecting from `oldsite.com` to `newsite.com`.
- **Content restructuring:** Moving blog posts to a new URL structure.
- **SEO optimisation:** Consolidating duplicate content.

#### Constraints and Limitations

- **Cached aggressively:** Browsers and search engines cache 301 redirects indefinitely. Use with caution.
- **Method preservation:** Some clients change POST to GET when following 301 redirects.
- **Not reversible:** Changing a 301 later can take months for caches to update.

---

### Sub-Feature 3.2: 302 Found (Moved Temporarily)

#### Definitions

**Core Definition:** 302 Found indicates that the resource resides temporarily under a different URI.

**Technical Definition:** The 302 Found status code indicates that the target resource resides temporarily under a different URI. Since the redirection might be altered on occasion, the client ought to continue to use the effective request URI for future requests. The `Location` header contains the temporary URL.

**Beginner-Friendly Explanation:** 302 is like a temporary detour. The resource is at a different URL right now, but it'll be back at the original URL later. Don't update your bookmarks.

#### Purposes

- To redirect clients temporarily to a different URL.
- To handle maintenance pages without changing the canonical URL.
- To perform post-login redirects without breaking bookmarks.

#### Syntax Rules and Structure

```js
res.redirect(302, '/temporary-location');  // 302 is default
```

#### Annotated Code Example

```js
// 302-temporary.js
const express = require('express');
const app = express();

app.get('/promo', (req, res) => {
  // During the promotion, redirect to the promo page
  res.redirect(302, '/promo/2026-sale');
});

app.get('/login-success', (req, res) => {
  // After login, redirect to dashboard
  res.redirect('/dashboard');  // 302 by default
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /promo`):**
```
HTTP/1.1 302 Found
Location: /promo/2026-sale
```

**Why this output:** The client is temporarily redirected to the promo page. After the promotion ends, the redirect can be removed, and the original URL will work again.

#### Real-World Cases

- **Post-login redirect:** Redirecting to the dashboard after authentication.
- **Temporary promotions:** Redirecting to a sale page for a limited time.
- **A/B testing:** Redirecting some users to a variant page.
- **Maintenance pages:** Redirecting to a "we'll be back soon" page.

#### Constraints and Limitations

- **Method changing:** Some clients change POST to GET when following 302 redirects, which can cause data loss.
- **Not cached:** Browsers do not cache 302 redirects (unlike 301).
- **SEO impact:** Search engines do not transfer link equity through 302 redirects as effectively as 301.

---

### Sub-Feature 3.3: 304 Not Modified

#### Definitions

**Core Definition:** 304 Not Modified indicates that the resource has not changed since the last request, allowing clients to use their cached version.

**Technical Definition:** The 304 Not Modified status code indicates that the client's cached copy is still valid and the server has not modified the resource. The response must not contain a body. It is sent in response to a conditional request (using `If-Modified-Since` or `If-None-Match` headers).

**Beginner-Friendly Explanation:** 304 is like asking "has anything changed?" and getting the answer "no, use what you already have." It saves bandwidth because the server doesn't need to resend data the client already has.

#### Purposes

- To indicate the resource has not changed since the last request.
- To allow clients to use their cached version.
- To reduce bandwidth usage and improve performance.

#### Syntax Rules and Structure

```js
app.get('/api/data', (req, res) => {
  const lastModified = new Date('2026-01-15');
  const ifModifiedSince = req.headers['if-modified-since'];

  if (ifModifiedSince && new Date(ifModifiedSince) >= lastModified) {
    return res.status(304).end();  // Not modified
  }

  res.set('Last-Modified', lastModified.toUTCString());
  res.json({ data: 'current' });
});
```

| Header | Purpose |
|--------|---------|
| `Last-Modified` | When the resource was last changed. |
| `If-Modified-Since` | Client's cached timestamp. |
| `ETag` | Unique identifier for the resource version. |
| `If-None-Match` | Client's cached ETag. |

#### Annotated Code Example

```js
// 304-not-modified.js
const express = require('express');
const app = express();

const data = { version: 1, content: 'Hello' };
const lastModified = new Date('2026-01-15T00:00:00Z');

app.get('/api/data', (req, res) => {
  const ifModifiedSince = req.headers['if-modified-since'];

  if (ifModifiedSince) {
    const clientDate = new Date(ifModifiedSince);
    if (clientDate >= lastModified) {
      return res.status(304).end();  // Not modified
    }
  }

  res.set('Last-Modified', lastModified.toUTCString());
  res.set('Cache-Control', 'public, max-age=3600');
  res.json(data);
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for first request):**
```
HTTP/1.1 200 OK
Last-Modified: Wed, 15 Jan 2026 00:00:00 GMT
Cache-Control: public, max-age=3600

{"version":1,"content":"Hello"}
```

**Expected Output (for subsequent request with `If-Modified-Since: Wed, 15 Jan 2026 00:00:00 GMT`):**
```
HTTP/1.1 304 Not Modified
```

**Why this output:** The client sends the `If-Modified-Since` header with the last known modification date. Since the resource has not changed, the server responds with 304 and no body, telling the client to use its cached copy.

#### Real-World Cases

- **Static assets:** CSS, JavaScript, and images cached by browsers.
- **API responses:** Reducing bandwidth for frequently accessed data.
- **CDN caching:** Edge servers validating cached content.
- **Mobile apps:** Reducing data usage for users on limited plans.

---

### Sub-Feature 3.4: 307 Temporary Redirect

#### Definitions

**Core Definition:** 307 Temporary Redirect indicates that the resource is temporarily at a different URI, and the client must reuse the original request method.

**Technical Definition:** The 307 Temporary Redirect status code is similar to 302 Found, except that it guarantees the request method and body will not be changed when the client follows the redirect. This is crucial for POST requests, where changing the method to GET would lose data.

**Beginner-Friendly Explanation:** 307 is like 302, but with a guarantee: "I'm redirecting you temporarily, and I promise your request will stay the same — same method, same data."

#### Purposes

- To redirect clients temporarily while preserving the request method.
- To handle POST redirects without data loss.
- To provide a safer alternative to 302 for non-idempotent requests.

#### Syntax Rules and Structure

```js
res.redirect(307, '/new-location');
```

#### Annotated Code Example

```js
// 307-temporary-preserve.js
const express = require('express');
const app = express();
app.use(express.urlencoded({ extended: true }));

app.post('/old-form', (req, res) => {
  // Redirect to new endpoint, preserving POST method and body
  res.redirect(307, '/new-form');
});

app.post('/new-form', (req, res) => {
  res.json({
    message: 'Form received at new endpoint',
    method: req.method,      // Still POST
    body: req.body           // Body preserved
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `POST /old-form` with form data):**
```
HTTP/1.1 307 Temporary Redirect
Location: /new-form

(Client re-sends POST to /new-form with the same body)
```

**Expected Output (at `/new-form`):**
```
{"message":"Form received at new endpoint","method":"POST","body":{"name":"Alice"}}
```

**Why this output:** The 307 redirect tells the client to repeat the request at the new location using the same method (POST) and body. This prevents the data loss that can occur with 302 redirects.

#### Real-World Cases

- **Form processing:** Redirecting POST requests without losing form data.
- **API versioning:** Temporarily redirecting old API endpoints to new ones.
- **Load balancing:** Redirecting requests to different servers while preserving the method.
- **Maintenance:** Redirecting POST requests during server maintenance.

---

### Sub-Feature 3.5: 308 Permanent Redirect

#### Definitions

**Core Definition:** 308 Permanent Redirect indicates that the resource is permanently at a different URI, and the client must reuse the original request method.

**Technical Definition:** The 308 Permanent Redirect status code is similar to 301 Moved Permanently, except that it guarantees the request method and body will not be changed. It is the permanent counterpart to 307.

**Beginner-Friendly Explanation:** 308 is like 301 but with a guarantee: "This resource has permanently moved, and your request will stay exactly the same when you follow the redirect."

#### Purposes

- To redirect clients permanently while preserving the request method.
- To handle POST redirects without data loss in permanent scenarios.
- To provide a method-safe alternative to 301.

#### Syntax Rules and Structure

```js
res.redirect(308, '/new-permanent-location');
```

#### Annotated Code Example

```js
// 308-permanent-preserve.js
const express = require('express');
const app = express();
app.use(express.json());

app.post('/api/v1/resource', (req, res) => {
  res.redirect(308, '/api/v2/resource');
});

app.post('/api/v2/resource', (req, res) => {
  res.json({
    message: 'Resource created at v2',
    method: req.method,
    body: req.body
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `POST /api/v1/resource` with JSON body):**
```
HTTP/1.1 308 Permanent Redirect
Location: /api/v2/resource
```

**Why this output:** The 308 redirect permanently moves the endpoint from v1 to v2 while preserving the POST method and body. Clients and search engines will update their references to the new URL.

#### Real-World Cases

- **API version migration:** Permanently moving from `/api/v1` to `/api/v2`.
- **Domain consolidation:** Permanently redirecting a subdomain to the main domain.
- **URL restructuring:** Permanently moving resources to a new URL scheme.

---

## Core Concept 4: 4xx Client Error Responses

### Definitions

**Core Definition:** 4xx status codes indicate that the client's request contains an error — bad syntax, missing authentication, insufficient permissions, or a non-existent resource.

**Technical Definition:** 4xx responses indicate that the client seems to have erred. Except when responding to a HEAD request, the server SHOULD send an explanation of the error situation, and whether it is a temporary or permanent condition.

**Beginner-Friendly Explanation:** 4xx codes mean "you made a mistake." The server is fine, but your request has a problem — maybe you're not logged in, don't have permission, or asked for something that doesn't exist.

### Purposes

- To indicate that the request contains bad syntax or cannot be fulfilled.
- To guide clients toward correcting their requests.
- To enforce authentication and authorisation.
- To protect server resources from abuse.

---

### Sub-Feature 4.1: 400 Bad Request

#### Definitions

**Core Definition:** 400 Bad Request indicates that the server cannot or will not process the request due to a client error (e.g., malformed request syntax, invalid request message framing).

**Technical Definition:** The 400 Bad Request status code indicates that the server cannot or will not process the request due to something that is perceived to be a client error (e.g., malformed request syntax, invalid request message framing, or deceptive request routing).

**Beginner-Friendly Explanation:** 400 means "I don't understand your request." Maybe you sent invalid JSON, forgot a required field, or sent data in the wrong format.

#### Purposes

- To indicate malformed payloads, syntax errors, or structural schema validation failures.
- To reject requests that the server cannot parse.
- To guide clients toward sending correctly formatted requests.

#### Syntax Rules and Structure

```js
app.post('/api/users', (req, res) => {
  if (!req.body.name) {
    return res.status(400).json({
      error: 'Bad Request',
      message: 'Name field is required'
    });
  }
  // Process valid request...
});
```

#### Annotated Code Example

```js
// 400-bad-request.js
const express = require('express');
const app = express();
app.use(express.json());

app.post('/api/users', (req, res) => {
  const { name, email } = req.body;

  // Validate required fields
  if (!name || typeof name !== 'string') {
    return res.status(400).json({
      error: 'Bad Request',
      details: 'Name is required and must be a string'
    });
  }

  if (!email || !email.includes('@')) {
    return res.status(400).json({
      error: 'Bad Request',
      details: 'Valid email is required'
    });
  }

  res.status(201).json({ id: 1, name, email });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `POST /api/users` with `{ "name": 123 }`):**
```
HTTP/1.1 400 Bad Request
Content-Type: application/json

{"error":"Bad Request","details":"Name is required and must be a string"}
```

**Why this output:** The request body contains an invalid `name` field (a number instead of a string). The server rejects the request with 400 and a descriptive error message.

#### Real-World Cases

- **API validation:** Rejecting requests with missing or invalid fields.
- **Malformed JSON:** The client sent invalid JSON syntax.
- **Invalid query parameters:** A query parameter is the wrong type.
- **Schema violations:** The request body doesn't match the expected schema.

---

### Sub-Feature 4.2: 401 Unauthorized

#### Definitions

**Core Definition:** 401 Unauthorized indicates that the request lacks valid authentication credentials for the target resource.

**Technical Definition:** The 401 Unauthorized status code indicates that the request has not been applied because it lacks valid authentication credentials for the target resource. The server MUST send a `WWW-Authenticate` header field containing a challenge applicable to the target resource.

**Beginner-Friendly Explanation:** 401 means "I don't know who you are." You need to log in or provide valid credentials before accessing this resource.

#### Purposes

- To indicate missing, invalid, or expired authentication credentials.
- To challenge the client to authenticate.
- To protect resources that require login.

#### Syntax Rules and Structure

```js
app.get('/api/protected', (req, res) => {
  const token = req.headers.authorization;
  if (!token) {
    return res.status(401)
      .set('WWW-Authenticate', 'Bearer realm="api"')
      .json({ error: 'Authentication required' });
  }
  // Verify token...
});
```

#### Annotated Code Example

```js
// 401-unauthorized.js
const express = require('express');
const jwt = require('jsonwebtoken');
const app = express();

const SECRET = 'my-secret';

app.get('/api/profile', (req, res) => {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401)
      .set('WWW-Authenticate', 'Bearer realm="api"')
      .json({ error: 'Authentication required' });
  }

  const token = authHeader.split(' ')[1];
  try {
    const decoded = jwt.verify(token, SECRET);
    res.json({ userId: decoded.userId, message: 'Authenticated' });
  } catch (err) {
    res.status(401)
      .set('WWW-Authenticate', 'Bearer realm="api", error="invalid_token"')
      .json({ error: 'Invalid or expired token' });
  }
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/profile` without token):**
```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="api"
Content-Type: application/json

{"error":"Authentication required"}
```

**Expected Output (for `GET /api/profile` with invalid token):**
```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="api", error="invalid_token"

{"error":"Invalid or expired token"}
```

**Why this output:** The request lacks a valid `Authorization` header. The server responds with 401 and the `WWW-Authenticate` header, challenging the client to provide valid credentials.

#### Real-World Cases

- **Protected API endpoints:** Requiring a valid JWT or API key.
- **Expired tokens:** Rejecting requests with expired authentication tokens.
- **Missing credentials:** Requests without an `Authorization` header.
- **Invalid API keys:** API keys that don't match any known client.

---

### Sub-Feature 4.3: 403 Forbidden

#### Definitions

**Core Definition:** 403 Forbidden indicates that the server understood the request but refuses to authorise it.

**Technical Definition:** The 403 Forbidden status code indicates that the server understood the request but refuses to fulfill it. A server that wishes to make public why the request has been forbidden can describe that reason in the response content. If authentication credentials were provided, the server considers them insufficient to grant access.

**Beginner-Friendly Explanation:** 403 means "I know who you are, but you're not allowed to do this." Unlike 401, you're authenticated — you just don't have permission.

#### Purposes

- To indicate that the client is authenticated but lacks the necessary authorization/roles.
- To deny access to resources the user is not permitted to view.
- To enforce role-based access control.

#### Syntax Rules and Structure

```js
app.get('/admin/dashboard', requireAuth, (req, res) => {
  if (req.user.role !== 'admin') {
    return res.status(403).json({
      error: 'Forbidden',
      message: 'Admin access required'
    });
  }
  res.json({ dashboard: 'data' });
});
```

#### Annotated Code Example

```js
// 403-forbidden.js
const express = require('express');
const app = express();

// Simulated authentication middleware
app.use((req, res, next) => {
  req.user = { id: 1, role: 'user' };  // Regular user
  next();
});

app.get('/admin/users', (req, res) => {
  if (req.user.role !== 'admin') {
    return res.status(403).json({
      error: 'Forbidden',
      message: 'You do not have permission to access this resource'
    });
  }
  res.json({ users: [] });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /admin/users` as a regular user):**
```
HTTP/1.1 403 Forbidden
Content-Type: application/json

{"error":"Forbidden","message":"You do not have permission to access this resource"}
```

**Why this output:** The user is authenticated (role: 'user') but does not have the required 'admin' role. The server responds with 403, indicating that access is denied despite valid authentication.

#### Real-World Cases

- **Role-based access:** Non-admin users trying to access admin endpoints.
- **Resource ownership:** Users trying to edit other users' posts.
- **Feature gating:** Free-tier users trying to access premium features.
- **IP restrictions:** Requests from blocked IP ranges.

---

### Sub-Feature 4.4: 404 Not Found

#### Definitions

**Core Definition:** 404 Not Found indicates that the server cannot find the requested resource.

**Technical Definition:** The 404 Not Found status code indicates that the origin server did not find a current representation for the target resource or is not willing to disclose that one exists.

**Beginner-Friendly Explanation:** 404 means "I looked, but I couldn't find what you asked for." The URL might be wrong, or the resource might have been deleted.

#### Purposes

- To indicate that the targeted resource, route, or endpoint does not exist.
- To inform clients that the URL is incorrect.
- To handle requests for deleted or never-existing resources.

#### Syntax Rules and Structure

```js
app.get('/api/users/:id', (req, res) => {
  const user = findUser(req.params.id);
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  res.json(user);
});
```

#### Annotated Code Example

```js
// 404-not-found.js
const express = require('express');
const app = express();

const users = [{ id: 1, name: 'Alice' }];

app.get('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) {
    return res.status(404).json({
      error: 'Not Found',
      message: `User with ID ${req.params.id} does not exist`
    });
  }
  res.json(user);
});

// Catch-all 404 handler
app.use((req, res) => {
  res.status(404).json({ error: 'Not Found', path: req.path });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/users/99`):**
```
HTTP/1.1 404 Not Found
Content-Type: application/json

{"error":"Not Found","message":"User with ID 99 does not exist"}
```

**Expected Output (for `GET /unknown-route`):**
```
HTTP/1.1 404 Not Found
Content-Type: application/json

{"error":"Not Found","path":"/unknown-route"}
```

**Why this output:** The requested user does not exist in the database, so the server responds with 404. The catch-all handler ensures that any unmatched route also returns 404.

#### Real-World Cases

- **Missing resources:** `GET /api/products/999` for a non-existent product.
- **Invalid URLs:** Users typing a wrong URL.
- **Deleted content:** Accessing a post that has been removed.
- **API versioning:** Old API endpoints that no longer exist.

---

### Sub-Feature 4.5: 405 Method Not Allowed

#### Definitions

**Core Definition:** 405 Method Not Allowed indicates that the request method is known by the server but is not supported by the target resource.

**Technical Definition:** The 405 Method Not Allowed status code indicates that the method received in the request-line is known by the origin server but not supported by the target resource. The origin server MUST generate an `Allow` header field in a 405 response containing a list of the target resource's currently supported methods.

**Beginner-Friendly Explanation:** 405 means "I know what you're trying to do, but I don't support that action on this URL." For example, you're trying to DELETE a resource that only supports GET.

#### Purposes

- To indicate that the request method is recognized but not supported by the target resource.
- To inform clients of the allowed methods via the `Allow` header.
- To prevent invalid operations on resources.

#### Syntax Rules and Structure

```js
app.all('/api/resource', (req, res) => {
  if (!['GET', 'POST'].includes(req.method)) {
    res.set('Allow', 'GET, POST');
    return res.status(405).json({
      error: 'Method Not Allowed',
      allowed: ['GET', 'POST']
    });
  }
  // Handle GET and POST...
});
```

#### Annotated Code Example

```js
// 405-method-not-allowed.js
const express = require('express');
const app = express();

app.all('/api/resource', (req, res, next) => {
  const allowed = ['GET', 'POST'];
  if (!allowed.includes(req.method)) {
    res.set('Allow', allowed.join(', '));
    return res.status(405).json({
      error: 'Method Not Allowed',
      message: `${req.method} is not supported`,
      allowed
    });
  }
  next();
});

app.get('/api/resource', (req, res) => {
  res.json({ method: 'GET', data: 'resource' });
});

app.post('/api/resource', (req, res) => {
  res.json({ method: 'POST', created: true });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `DELETE /api/resource`):**
```
HTTP/1.1 405 Method Not Allowed
Allow: GET, POST
Content-Type: application/json

{"error":"Method Not Allowed","message":"DELETE is not supported","allowed":["GET","POST"]}
```

**Why this output:** The resource supports GET and POST, but the client sent a DELETE request. The server responds with 405 and the `Allow` header listing the supported methods.

#### Real-World Cases

- **REST API design:** Resources that only support specific methods.
- **Read-only endpoints:** `GET /api/config` that doesn't support POST.
- **Form handling:** Endpoints that only accept POST, not GET.
- **Collection vs. item:** `/api/users` supports GET and POST, `/api/users/:id` supports GET, PUT, DELETE.

---

### Sub-Feature 4.6: 408 Request Timeout

#### Definitions

**Core Definition:** 408 Request Timeout indicates that the server timed out waiting for the client's request.

**Technical Definition:** The 408 Request Timeout status code indicates that the server did not receive a complete request message within the time that it was prepared to wait. This can happen when the client takes too long to send the request headers or body.

**Beginner-Friendly Explanation:** 408 means "you took too long to send your request, so I gave up waiting." It's like a shop closing because the customer took too long to decide.

#### Purposes

- To indicate that the client took too long to send the request.
- To free up server resources tied up by slow clients.
- To protect against slowloris-style attacks.

#### Syntax Rules and Structure

```js
// Node.js server timeout configuration
server.setTimeout(30000); // 30 seconds

// Express-level timeout handling
app.use((req, res, next) => {
  req.setTimeout(30000, () => {
    res.status(408).json({ error: 'Request Timeout' });
  });
  next();
});
```

#### Annotated Code Example

```js
// 408-request-timeout.js
const express = require('express');
const app = express();

app.use((req, res, next) => {
  // Set a 5-second timeout for the request
  req.setTimeout(5000, () => {
    if (!res.headersSent) {
      res.status(408).json({
        error: 'Request Timeout',
        message: 'Request took too long to complete'
      });
    }
  });
  next();
});

app.get('/slow', (req, res) => {
  // Simulate slow processing
  setTimeout(() => {
    res.json({ data: 'slow response' });
  }, 10000);  // 10 seconds — will timeout
});

app.get('/fast', (req, res) => {
  res.json({ data: 'fast response' });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /slow` after 5 seconds):**
```
HTTP/1.1 408 Request Timeout
Content-Type: application/json

{"error":"Request Timeout","message":"Request took too long to complete"}
```

**Why this output:** The server has a 5-second timeout. The `/slow` endpoint takes 10 seconds to respond, so the server sends a 408 before the handler completes.

#### Real-World Cases

- **Slow clients:** Clients on poor network connections.
- **Slowloris attacks:** Malicious clients holding connections open.
- **File uploads:** Large uploads that exceed the timeout window.
- **API gateways:** Gateway timeouts to backend services.

---

### Sub-Feature 4.7: 409 Conflict

#### Definitions

**Core Definition:** 409 Conflict indicates that the request could not be completed due to a conflict with the current state of the target resource.

**Technical Definition:** The 409 Conflict status code indicates that the request could not be completed due to a conflict with the current state of the target resource. This code is used in situations where the user might be able to resolve the conflict and resubmit the request.

**Beginner-Friendly Explanation:** 409 means "there's a conflict — something already exists or the state doesn't match." For example, trying to register with an email that's already taken.

#### Purposes

- To indicate state violations, such as duplicate registrations.
- To handle resource concurrency issues (optimistic locking failures).
- To inform clients that their request conflicts with existing data.

#### Syntax Rules and Structure

```js
app.post('/api/users', (req, res) => {
  if (emailExists(req.body.email)) {
    return res.status(409).json({
      error: 'Conflict',
      message: 'Email already registered'
    });
  }
  // Create user...
});
```

#### Annotated Code Example

```js
// 409-conflict.js
const express = require('express');
const app = express();
app.use(express.json());

const users = [{ id: 1, email: 'alice@example.com' }];

app.post('/api/users', (req, res) => {
  const { email } = req.body;

  const existing = users.find(u => u.email === email);
  if (existing) {
    return res.status(409).json({
      error: 'Conflict',
      message: `Email ${email} is already registered`,
      existingUserId: existing.id
    });
  }

  const newUser = { id: users.length + 1, email };
  users.push(newUser);
  res.status(201).json(newUser);
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `POST /api/users` with `{ "email": "alice@example.com" }`):**
```
HTTP/1.1 409 Conflict
Content-Type: application/json

{"error":"Conflict","message":"Email alice@example.com is already registered","existingUserId":1}
```

**Why this output:** The email is already registered, so the server rejects the request with 409 and provides information about the existing user.

#### Real-World Cases

- **Duplicate registration:** Email or username already taken.
- **Optimistic locking:** Resource was modified by another request.
- **Inventory conflicts:** Trying to book an already-reserved item.
- **Version conflicts:** Editing a resource with an outdated version number.

---

### Sub-Feature 4.8: 415 Unsupported Media Type

#### Definitions

**Core Definition:** 415 Unsupported Media Type indicates that the server refuses to service the request because the payload format is unsupported.

**Technical Definition:** The 415 Unsupported Media Type status code indicates that the origin server is refusing to service the request because the payload is in a format not supported by this method on the target resource. The format problem might be due to the request's indicated `Content-Type` or `Content-Encoding`, or as a result of inspecting the data directly.

**Beginner-Friendly Explanation:** 415 means "I don't understand the format of the data you sent." For example, you sent XML when the server only accepts JSON.

#### Purposes

- To indicate that the payload format (e.g., XML instead of JSON) is unsupported.
- To enforce API content-type requirements.
- To guide clients toward sending data in the correct format.

#### Syntax Rules and Structure

```js
app.post('/api/data', (req, res) => {
  if (!req.is('application/json')) {
    return res.status(415).json({
      error: 'Unsupported Media Type',
      message: 'Content-Type must be application/json'
    });
  }
  // Process JSON...
});
```

#### Annotated Code Example

```js
// 415-unsupported-media.js
const express = require('express');
const app = express();
app.use(express.json());

app.post('/api/data', (req, res) => {
  // Check if the request is JSON
  if (!req.is('application/json')) {
    return res.status(415).json({
      error: 'Unsupported Media Type',
      message: 'This endpoint only accepts application/json',
      received: req.get('Content-Type') || 'none'
    });
  }

  res.json({ received: req.body });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `POST /api/data` with `Content-Type: text/xml`):**
```
HTTP/1.1 415 Unsupported Media Type
Content-Type: application/json

{"error":"Unsupported Media Type","message":"This endpoint only accepts application/json","received":"text/xml"}
```

**Why this output:** The client sent XML data, but the server only accepts JSON. The server rejects the request with 415 and explains the accepted format.

#### Real-World Cases

- **JSON-only APIs:** Rejecting XML or form-data payloads.
- **File uploads:** Rejecting unsupported file types.
- **Content negotiation:** Ensuring clients send the correct format.
- **API versioning:** Different versions may accept different formats.

---

### Sub-Feature 4.9: 422 Unprocessable Entity

#### Definitions

**Core Definition:** 422 Unprocessable Entity indicates that the request is well-formed and syntactically correct but contains semantic validation errors.

**Technical Definition:** The 422 Unprocessable Entity status code indicates that the server understands the content type of the request entity, and the syntax of the request entity is correct, but it was unable to process the contained instructions. This is used for semantic validation errors, such as a field failing domain-logic rules.

**Beginner-Friendly Explanation:** 422 means "your request is formatted correctly, but the content doesn't make sense." For example, you provided a date that's in the past when the server requires a future date.

#### Purposes

- To indicate semantic validation errors (field fails domain-logic rules).
- To distinguish between syntax errors (400) and semantic errors (422).
- To provide detailed validation feedback for form submissions.

#### Syntax Rules and Structure

```js
app.post('/api/users', (req, res) => {
  const errors = validateUser(req.body);
  if (errors.length > 0) {
    return res.status(422).json({
      error: 'Unprocessable Entity',
      details: errors
    });
  }
  // Create user...
});
```

#### Annotated Code Example

```js
// 422-unprocessable.js
const express = require('express');
const app = express();
app.use(express.json());

app.post('/api/events', (req, res) => {
  const { title, date, capacity } = req.body;
  const errors = [];

  // Semantic validation (well-formed but invalid)
  if (date && new Date(date) < new Date()) {
    errors.push({ field: 'date', message: 'Event date must be in the future' });
  }

  if (capacity && capacity < 1) {
    errors.push({ field: 'capacity', message: 'Capacity must be at least 1' });
  }

  if (errors.length > 0) {
    return res.status(422).json({
      error: 'Unprocessable Entity',
      message: 'Validation failed',
      details: errors
    });
  }

  res.status(201).json({ id: 1, title, date, capacity });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `POST /api/events` with `{ "title": "Meeting", "date": "2020-01-01", "capacity": 0 }`):**
```
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json

{
  "error": "Unprocessable Entity",
  "message": "Validation failed",
  "details": [
    {"field": "date", "message": "Event date must be in the future"},
    {"field": "capacity", "message": "Capacity must be at least 1"}
  ]
}
```

**Why this output:** The request is syntactically valid JSON with all required fields, but the `date` is in the past and the `capacity` is zero. These are semantic validation errors, so the server responds with 422 and detailed error information.

#### Real-World Cases

- **Form validation:** Submitting a form with invalid field values.
- **API validation:** Business rule violations (e.g., insufficient funds).
- **Data integrity:** Referencing non-existent related resources.
- **Domain logic:** A booking date that conflicts with existing reservations.

---

### Sub-Feature 4.10: 429 Too Many Requests

#### Definitions

**Core Definition:** 429 Too Many Requests indicates that the user has sent too many requests in a given amount of time (rate limiting).

**Technical Definition:** The 429 Too Many Requests status code indicates that the user has sent too many requests in a given amount of time. The response should include a `Retry-After` header indicating how long to wait before making a new request.

**Beginner-Friendly Explanation:** 429 means "slow down, you're sending requests too fast." It's like a bouncer saying "you've had enough, come back later."

#### Purposes

- To indicate that the user has sent too many requests in a given amount of time.
- To protect server resources from abuse.
- To enforce API rate limits and quotas.

#### Syntax Rules and Structure

```js
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                   // 100 requests per window
  message: { error: 'Too Many Requests' },
  standardHeaders: true,
  legacyHeaders: false
});

app.use('/api/', limiter);
```

| Header | Purpose |
|--------|---------|
| `Retry-After` | Seconds to wait before retrying. |
| `RateLimit-Limit` | Maximum requests allowed. |
| `RateLimit-Remaining` | Requests remaining in the window. |

#### Annotated Code Example

```js
// 429-too-many.js
const express = require('express');
const rateLimit = require('express-rate-limit');
const app = express();

const limiter = rateLimit({
  windowMs: 60 * 1000,  // 1 minute
  max: 5,                // 5 requests per minute
  message: {
    error: 'Too Many Requests',
    message: 'You have exceeded the 5 requests per minute limit'
  },
  standardHeaders: true,
  legacyHeaders: false
});

app.use('/api/', limiter);

app.get('/api/data', (req, res) => {
  res.json({ data: 'success' });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for the 6th request within 1 minute):**
```
HTTP/1.1 429 Too Many Requests
Retry-After: 45
RateLimit-Limit: 5
RateLimit-Remaining: 0
Content-Type: application/json

{"error":"Too Many Requests","message":"You have exceeded the 5 requests per minute limit"}
```

**Why this output:** The client has made 5 requests in 1 minute. The 6th request is rejected with 429. The `Retry-After` header tells the client to wait 45 seconds before trying again.

#### Real-World Cases

- **API rate limiting:** Limiting requests per API key.
- **Login attempts:** Preventing brute-force attacks.
- **Scraping prevention:** Blocking automated scraping.
- **Free-tier limits:** Limiting free users to a certain number of requests.

---

## Core Concept 5: 5xx Server Error Responses

### Definitions

**Core Definition:** 5xx status codes indicate that the server failed to fulfill an apparently valid request due to an error on the server side.

**Technical Definition:** 5xx responses indicate that the server is aware that it has erred or is incapable of performing the requested method. The server SHOULD send an explanation of the error situation and indicate whether it is a temporary or permanent condition.

**Beginner-Friendly Explanation:** 5xx codes mean "it's not you, it's me." The request was valid, but the server encountered a problem and couldn't complete it.

### Purposes

- To indicate that the server failed to fulfill a valid request.
- To distinguish between client-side (4xx) and server-side (5xx) errors.
- To provide information about the nature of the server failure.

---

### Sub-Feature 5.1: 500 Internal Server Error

#### Definitions

**Core Definition:** 500 Internal Server Error is the generic fallback code for unhandled exceptions, runtime crashes, or unexpected server conditions.

**Technical Definition:** The 500 Internal Server Error status code indicates that the server encountered an unexpected condition that prevented it from fulfilling the request. This is the generic "catch-all" error response when no more specific message is suitable.

**Beginner-Friendly Explanation:** 500 means "something went wrong on my end and I don't know exactly what." It's the server equivalent of a shrug.

#### Purposes

- To indicate an unhandled exception or unexpected server condition.
- To serve as the generic fallback error response.
- To hide internal error details from clients for security reasons.

#### Syntax Rules and Structure

```js
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    error: 'Internal Server Error',
    message: 'Something went wrong'
  });
});
```

#### Annotated Code Example

```js
// 500-internal-error.js
const express = require('express');
const app = express();

app.get('/api/crash', (req, res) => {
  // Simulate an unexpected error
  throw new Error('Database connection failed');
});

app.get('/api/error', (req, res, next) => {
  // Pass error to error handler
  next(new Error('Something broke'));
});

// Error-handling middleware (4 arguments)
app.use((err, req, res, next) => {
  console.error('Error:', err.message);
  res.status(500).json({
    error: 'Internal Server Error',
    message: 'An unexpected error occurred'
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/crash`):**
```
HTTP/1.1 500 Internal Server Error
Content-Type: application/json

{"error":"Internal Server Error","message":"An unexpected error occurred"}
```

**Why this output:** The route throws an error, which is caught by Express's error-handling middleware. The middleware logs the error and sends a 500 response with a generic message (not exposing internal details).

#### Real-World Cases

- **Unhandled exceptions:** Code bugs that throw errors.
- **Database failures:** Connection timeouts or query errors.
- **Third-party API failures:** External services that are down.
- **Configuration errors:** Missing environment variables.

---

### Sub-Feature 5.2: 502 Bad Gateway

#### Definitions

**Core Definition:** 502 Bad Gateway indicates that the server, acting as a gateway or proxy, received an invalid response from an upstream server.

**Technical Definition:** The 502 Bad Gateway status code indicates that the server, while acting as a gateway or proxy, received an invalid response from an inbound server it accessed while attempting to fulfill the request.

**Beginner-Friendly Explanation:** 502 means "I'm the middleman, and the server behind me gave me a bad response." The problem is with the upstream server, not you.

#### Purposes

- To indicate that a gateway or proxy received an invalid response from an upstream server.
- To signal communication problems between servers.
- To help diagnose infrastructure issues.

#### Syntax Rules and Structure

```js
// Typically generated by proxy/load balancer
// In Express, you can simulate it:
app.get('/api/upstream', async (req, res) => {
  try {
    const response = await fetch('http://backend/api/data');
    if (!response.ok) throw new Error('Invalid upstream response');
    res.json(await response.json());
  } catch (err) {
    res.status(502).json({ error: 'Bad Gateway' });
  }
});
```

#### Annotated Code Example

```js
// 502-bad-gateway.js
const express = require('express');
const app = express();

app.get('/api/proxy', async (req, res) => {
  try {
    // Simulate calling an upstream service that fails
    const response = await fetch('http://unreachable-service/api/data', {
      signal: AbortSignal.timeout(5000)
    });

    if (!response.ok) {
      return res.status(502).json({
        error: 'Bad Gateway',
        message: 'Upstream service returned an invalid response',
        upstreamStatus: response.status
      });
    }

    res.json(await response.json());
  } catch (err) {
    res.status(502).json({
      error: 'Bad Gateway',
      message: 'Failed to communicate with upstream service'
    });
  }
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/proxy` when upstream is unreachable):**
```
HTTP/1.1 502 Bad Gateway
Content-Type: application/json

{"error":"Bad Gateway","message":"Failed to communicate with upstream service"}
```

**Why this output:** The Express server acts as a proxy to an unreachable upstream service. When the upstream fails, the server returns 502 to indicate the gateway problem.

#### Real-World Cases

- **Load balancers:** Nginx or HAProxy receiving invalid responses from backends.
- **API gateways:** Kong or AWS API Gateway with failing upstream services.
- **Microservices:** Service mesh communication failures.
- **Reverse proxies:** Cloudflare or AWS ALB with unhealthy targets.

---

### Sub-Feature 5.3: 503 Service Unavailable

#### Definitions

**Core Definition:** 503 Service Unavailable indicates that the server is currently unable to handle the request due to temporary overloading or maintenance.

**Technical Definition:** The 503 Service Unavailable status code indicates that the server is currently unable to handle the request due to a temporary overload or scheduled maintenance, which will likely be alleviated after some delay. The server MAY send a `Retry-After` header suggesting an appropriate wait time.

**Beginner-Friendly Explanation:** 503 means "I'm temporarily unavailable — either I'm overloaded or I'm under maintenance. Try again later."

#### Purposes

- To indicate temporary server unavailability due to overloading or maintenance.
- To inform clients about expected recovery time via `Retry-After`.
- To gracefully handle downtime without crashing.

#### Syntax Rules and Structure

```js
app.use((req, res, next) => {
  if (maintenanceMode) {
    res.set('Retry-After', '3600');
    return res.status(503).json({
      error: 'Service Unavailable',
      message: 'Scheduled maintenance in progress'
    });
  }
  next();
});
```

#### Annotated Code Example

```js
// 503-service-unavailable.js
const express = require('express');
const app = express();

let maintenanceMode = true;

app.use((req, res, next) => {
  if (maintenanceMode) {
    res.set('Retry-After', '3600');  // 1 hour
    return res.status(503).json({
      error: 'Service Unavailable',
      message: 'Scheduled maintenance in progress. Please try again later.',
      retryAfter: 3600
    });
  }
  next();
});

app.get('/api/data', (req, res) => {
  res.json({ data: 'available' });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (during maintenance):**
```
HTTP/1.1 503 Service Unavailable
Retry-After: 3600
Content-Type: application/json

{"error":"Service Unavailable","message":"Scheduled maintenance in progress. Please try again later.","retryAfter":3600}
```

**Why this output:** The server is in maintenance mode. All requests are rejected with 503, and the `Retry-After` header tells clients to wait 1 hour before retrying.

#### Real-World Cases

- **Scheduled maintenance:** Deploying new code or database migrations.
- **Overload protection:** Server at capacity and rejecting new requests.
- **Dependency failures:** Critical upstream service is down.
- **Emergency shutdown:** Graceful degradation during incidents.

---

### Sub-Feature 5.4: 504 Gateway Timeout

#### Definitions

**Core Definition:** 504 Gateway Timeout indicates that the server, acting as a gateway or proxy, did not receive a timely response from an upstream server.

**Technical Definition:** The 504 Gateway Timeout status code indicates that the server, while acting as a gateway or proxy, did not receive a timely response from an upstream server it needed to access in order to complete the request.

**Beginner-Friendly Explanation:** 504 means "I'm the middleman, and the server behind me took too long to respond, so I gave up waiting."

#### Purposes

- To indicate that a gateway or proxy did not receive a timely response from an upstream server.
- To signal timeout conditions in multi-tier architectures.
- To help diagnose performance bottlenecks.

#### Syntax Rules and Structure

```js
app.get('/api/slow-upstream', async (req, res) => {
  try {
    const response = await fetch('http://slow-backend/api/data', {
      signal: AbortSignal.timeout(5000)
    });
    res.json(await response.json());
  } catch (err) {
    if (err.name === 'TimeoutError') {
      return res.status(504).json({ error: 'Gateway Timeout' });
    }
    next(err);
  }
});
```

#### Annotated Code Example

```js
// 504-gateway-timeout.js
const express = require('express');
const app = express();

app.get('/api/upstream', async (req, res) => {
  try {
    const response = await fetch('http://slow-backend/api/data', {
      signal: AbortSignal.timeout(3000)  // 3-second timeout
    });
    res.json(await response.json());
  } catch (err) {
    if (err.name === 'TimeoutError') {
      return res.status(504).json({
        error: 'Gateway Timeout',
        message: 'Upstream server did not respond within the allowed time',
        timeout: 3000
      });
    }
    res.status(502).json({ error: 'Bad Gateway' });
  }
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/upstream` when upstream times out):**
```
HTTP/1.1 504 Gateway Timeout
Content-Type: application/json

{"error":"Gateway Timeout","message":"Upstream server did not respond within the allowed time","timeout":3000}
```

**Why this output:** The Express server acts as a proxy and sets a 3-second timeout. The upstream server takes longer than 3 seconds to respond, so the proxy returns 504.

#### Real-World Cases

- **Slow databases:** Queries that exceed the gateway timeout.
- **Third-party APIs:** External services that are slow or unresponsive.
- **Microservices:** Service mesh timeouts between services.
- **CDN origins:** CDN edge servers waiting for origin responses.

---

## References

- IANA HTTP Status Code Registry — https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
- RFC 9110 — HTTP Semantics (Section 15: Status Codes) — https://www.rfc-editor.org/rfc/rfc9110#section-15
- RFC 8297 — 103 Early Hints — https://www.rfc-editor.org/rfc/rfc8297
- MDN — HTTP response status codes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status
- MDN — 100 Continue — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/100
- MDN — 101 Switching Protocols — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/101
- MDN — 103 Early Hints — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/103
- MDN — 200 OK — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/200
- MDN — 201 Created — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/201
- MDN — 202 Accepted — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/202
- MDN — 204 No Content — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/204
- MDN — 206 Partial Content — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/206
- MDN — 301 Moved Permanently — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/301
- MDN — 302 Found — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/302
- MDN — 304 Not Modified — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/304
- MDN — 307 Temporary Redirect — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/307
- MDN — 308 Permanent Redirect — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/308
- MDN — 400 Bad Request — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/400
- MDN — 401 Unauthorized — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401
- MDN — 403 Forbidden — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/403
- MDN — 404 Not Found — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404
- MDN — 405 Method Not Allowed — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/405
- MDN — 408 Request Timeout — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/408
- MDN — 409 Conflict — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/409
- MDN — 415 Unsupported Media Type — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/415
- MDN — 422 Unprocessable Entity — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/422
- MDN — 429 Too Many Requests — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429
- MDN — 500 Internal Server Error — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/500
- MDN — 502 Bad Gateway — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/502
- MDN — 503 Service Unavailable — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/503
- MDN — 504 Gateway Timeout — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/504
- Express.js — res.status() — https://expressjs.com/en/5x/api.html#res.status
- Express.js — res.sendStatus() — https://expressjs.com/en/5x/api.html#res.sendStatus
- Express.js — res.redirect() — https://expressjs.com/en/5x/api.html#res.redirect
- Express.js — Error Handling — https://expressjs.com/en/guide/error-handling.html
- express-rate-limit — npm — https://www.npmjs.com/package/express-rate-limit
- Compile-N-Run — Express Status Codes — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/4-express-response-handling/2-express-status-codes.mdx