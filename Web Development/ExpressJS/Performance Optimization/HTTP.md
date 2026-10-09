# HTTP Performance — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** HTTP performance optimisation is the practice of reducing the time and bandwidth required to transfer resources between clients and servers, by persisting TCP connections, compressing payloads, leveraging caching layers, validating cached content with ETags and conditional requests, upgrading to HTTP/2 or HTTP/3 for multiplexing, and offloading static asset delivery to Content Delivery Networks.

**Technical Definition:** HTTP performance is governed by several factors across the request lifecycle: the **connection layer** (TCP handshake, TLS handshake, keep-alive), the **transfer layer** (compression algorithms — Gzip, Brotli, Zstandard), the **caching layer** (Cache-Control directives, ETags, conditional requests), the **protocol layer** (HTTP/1.1 vs. HTTP/2 multiplexing vs. HTTP/3 QUIC), and the **delivery layer** (CDNs, edge caching). Each layer addresses a different source of latency: connection setup, payload size, redundant transfers, head-of-line blocking, and geographic distance.

**Beginner-Friendly Explanation:** When a browser loads a webpage, it has to connect to the server, ask for files, receive them, and render them. HTTP performance is about making each of those steps faster: keeping the connection open so you don't reconnect for every file (keep-alive), shrinking the files before sending (compression), telling the browser it can reuse files it already has (caching), checking whether a file has changed before re-downloading it (ETags), sending multiple files at once without waiting in line (HTTP/2), and serving files from a server physically close to the user (CDN).

### Key Characteristics

- **Keep-alive eliminates repeated handshakes:** One TCP connection serves many requests, saving 1–3 round trips per request. 
- **Compression reduces transfer size:** Brotli achieves 15–25% better compression than Gzip on text assets. 
- **Cache-Control is the primary caching directive:** `max-age`, `s-maxage`, `immutable`, and `stale-while-revalidate` control browser and CDN caching. 
- **ETags enable conditional requests:** The server sends an ETag; the client sends `If-None-Match`; the server returns 304 Not Modified if unchanged. 
- **HTTP/2 multiplexes streams:** Multiple requests share one connection without head-of-line blocking at the HTTP layer. 
- **HTTP/3 eliminates TCP head-of-line blocking:** QUIC runs over UDP, so a lost packet does not block other streams. 
- **CDNs offload static assets:** Edge servers cache assets geographically close to users, reducing latency and origin load. 

### Prerequisites

- **Node.js runtime** (v18 or higher for HTTP/2 support).
- **Express.js installed:** `npm install express`.
- **For compression:** `npm install compression` (or configure at the reverse proxy).
- **For HTTP/2:** Node.js built-in `http2` module.
- **A reverse proxy** (Nginx, Caddy) or CDN (Cloudflare, CloudFront) for production deployments.
- **Basic understanding of HTTP headers and caching semantics.**

### Related Programming Areas

- **Express Performance:** Application-level optimisation complements HTTP-level optimisation.
- **CDN configuration:** Cache-Control and ETags drive CDN caching behaviour.
- **TLS/SSL:** HTTP/2 and HTTP/3 require HTTPS.
- **Reverse proxies:** Nginx and Cloudflare handle compression and caching at the edge.
- **Frontend performance:** Asset hashing, code splitting, and lazy loading complement HTTP caching.

### Core Concepts

1. **Keep-Alive** — persisting TCP connections.
2. **Compression** — Gzip/Brotli at the reverse proxy or application level.
3. **Caching** — Cache-Control directives for browser and CDN layers.
4. **ETags** — asset hashes for validation control.
5. **Conditional Requests** — `If-None-Match` and `If-Modified-Since`.
6. **HTTP/2 and HTTP/3** — multiplexing and eliminating head-of-line blocking.
7. **Content Delivery Networks (CDNs)** — offloading static asset delivery.

---

## Core Concept 1: Keep-Alive

### Definitions

**Core Definition:** Keep-alive (persistent connections) allows a single TCP connection to be reused for multiple HTTP requests and responses, eliminating the TCP and TLS handshake overhead for each subsequent request.

**Technical Definition:** In HTTP/1.0, each request required a new TCP connection. HTTP/1.1 introduced persistent connections by default, controlled by the `Connection: keep-alive` header. The server keeps the connection open for a configurable timeout (`keepAliveTimeout`), during which the client can send additional requests. For HTTPS, the TLS session is also reused, avoiding the TLS handshake. In Node.js, the HTTP server's `keepAliveTimeout` (default 5 seconds) and `headersTimeout` control connection persistence. For outbound requests, `http.Agent` or `undici.Agent` with `keepAlive: true` enables connection reuse.

**Beginner-Friendly Explanation:** Without keep-alive, every request is like making a phone call: dial, wait for an answer, talk, hang up. With keep-alive, you dial once, have a conversation with multiple questions, and hang up when you are done. This saves the dialling time for every question after the first.

### Purposes

- To eliminate TCP handshake overhead (1 round trip per connection).
- To eliminate TLS handshake overhead (1–2 round trips per connection).
- To reduce latency for subsequent requests on the same connection.
- To reduce CPU usage from repeated connection setup.

### Syntax Rules and Structure

**Node.js HTTP server:**
```js
const server = app.listen(3000);
server.keepAliveTimeout = 65000;  // 65 seconds (longer than typical load balancer)
server.headersTimeout = 66000;    // Slightly longer than keepAliveTimeout
```

**Nginx configuration:**
```nginx
http {
  keepalive_timeout 65s;
  keepalive_requests 1000;
}
```

**Outbound requests (Undici):**
```js
const { Agent } = require('undici');
const agent = new Agent({ keepAliveTimeout: 60000, connections: 100 });
```

| Setting | Default | Recommended |
|---------|---------|-------------|
| `keepAliveTimeout` | 5s | 65s (behind load balancer). |
| `headersTimeout` | 60s | `keepAliveTimeout + 1s`. |
| `keepalive_requests` (Nginx) | 1000 | 1000. |

**Rules:**
- Set `keepAliveTimeout` **longer** than the load balancer's idle timeout to avoid race conditions. 
- Set `headersTimeout` slightly longer than `keepAliveTimeout`. 
- Use `keepAlive: true` on outbound HTTP agents. 
- Monitor active connections to detect leaks or misconfiguration. 

### Annotated Code Example

```js
// keep-alive.js
const express = require('express');
const app = express();

app.get('/api/data', (req, res) => {
  res.json({ data: 'ok' });
});

const server = app.listen(3000, () => {
  console.log('Server on 3000 with keep-alive');
});

// Keep connections open for 65 seconds
server.keepAliveTimeout = 65000;
server.headersTimeout = 66000;

console.log('keepAliveTimeout:', server.keepAliveTimeout);
console.log('headersTimeout:', server.headersTimeout);
```

**Expected Output (headers):**
```
Connection: keep-alive
Keep-Alive: timeout=65
```

**Expected Output (performance comparison):**
```
Without keep-alive (100 requests): ~850ms total (TCP+TLS per request)
With keep-alive (100 requests):    ~120ms total (connection reused)
```

**Why this output:** The first request establishes the TCP and TLS connection. Subsequent requests reuse it. The `Keep-Alive: timeout=65` header tells the client the connection will remain open for 65 seconds. The performance gain is most pronounced for HTTPS, where the TLS handshake is the largest overhead.

### Real-World Cases

- **API gateways:** Reusing connections between the gateway and backend services.
- **Microservices:** Persistent connections between services reduce inter-service latency.
- **Browser connections:** Keeping the connection open while loading a webpage with multiple assets.

---

## Core Concept 2: Compression

### Definitions

**Core Definition:** HTTP compression reduces the size of response bodies by applying a compression algorithm (Gzip, Brotli, or Zstandard) before transmission, which the client decompresses automatically.

**Technical Definition:** The server advertises supported algorithms via the `Accept-Encoding` request header. The client sends `Accept-Encoding: gzip, deflate, br, zstd`; the server selects the best-supported algorithm and compresses the response, setting `Content-Encoding: br` (or `gzip`) and adjusting `Content-Length` to the compressed size. Brotli (RFC 7932) achieves 15–25% better compression than Gzip on text assets. Zstandard (RFC 8878) offers a speed/ratio trade-off between Gzip and Brotli. In production, compression is typically handled by a reverse proxy (Nginx, Cloudflare) rather than the application server, freeing Node.js CPU.

**Beginner-Friendly Explanation:** Compression is like vacuum-sealing your clothes before packing them. The clothes take up less space in the suitcase, so they fit more easily and are lighter to carry. The recipient (the browser) opens the vacuum bag and the clothes are back to normal. 

### Purposes

- To reduce bandwidth consumption and transfer time.
- To improve page load speed for clients on slow connections.
- To reduce egress costs in cloud environments.
- To support modern algorithms (Brotli, Zstd) alongside legacy Gzip.

### Syntax Rules and Structure

**Nginx compression configuration:**
```nginx
http {
  # Gzip
  gzip on;
  gzip_types text/plain text/css application/json application/javascript text/xml application/xml image/svg+xml;
  gzip_min_length 1000;
  gzip_comp_level 6;
  gzip_vary on;

  # Brotli (requires ngx_brotli module)
  brotli on;
  brotli_types text/plain text/css application/json application/javascript text/xml application/xml image/svg+xml;
  brotli_comp_level 6;
  brotli_min_length 1000;

  # Zstandard (requires ngx_zstd module)
  zstd on;
  zstd_types text/plain text/css application/json application/javascript;
  zstd_comp_level 3;
  zstd_min_length 1000;
}
```

**Cloudflare (automatic):**
```
Cloudflare automatically compresses:
- Brotli for HTTPS (default)
- Gzip for HTTP
- Zstandard for supported clients
No configuration required.
```

**Express (application-level, not recommended for production):**
```js
const compression = require('compression');
app.use(compression({ level: 6, brotli: { enabled: true } }));
```

| Algorithm | Compression Ratio | Speed | Browser Support |
|-----------|------------------|-------|-----------------|
| Gzip | Baseline | Fast | Universal. |
| Brotli | 15–25% better | Slower | All modern browsers. |
| Zstandard | Similar to Brotli | Very fast | Chrome, Firefox, Edge. |

**Rules:**
- Delegate compression to the reverse proxy or CDN in production — it has dedicated CPU. 
- Use Brotli for HTTPS; Gzip for HTTP (or Brotli with fallback). 
- Set `gzip_min_length` / `brotli_min_length` to 1000 bytes — compressing tiny responses wastes CPU. 
- Do not compress already-compressed assets (images, videos). 
- Set `Vary: Accept-Encoding` so caches store compressed and uncompressed variants separately. 

### Annotated Code Example

```nginx
# nginx-compression.conf
http {
  # Gzip configuration
  gzip on;
  gzip_vary on;
  gzip_min_length 1000;
  gzip_comp_level 6;
  gzip_types
    text/plain
    text/css
    text/javascript
    application/json
    application/javascript
    application/xml
    application/xml+rss
    image/svg+xml;

  # Brotli configuration (requires ngx_brotli)
  brotli on;
  brotli_comp_level 6;
  brotli_min_length 1000;
  brotli_types
    text/plain
    text/css
    text/javascript
    application/json
    application/javascript
    application/xml
    image/svg+xml;

  server {
    listen 443 ssl http2;
    server_name example.com;

    location /api/ {
      proxy_pass http://localhost:3000;
      proxy_set_header Accept-Encoding "";
    }
  }
}
```

**Expected Output (response headers):**
```
# With Accept-Encoding: br
Content-Encoding: br
Content-Length: 8420
Vary: Accept-Encoding

# With Accept-Encoding: gzip
Content-Encoding: gzip
Content-Length: 12450
Vary: Accept-Encoding
```

**Expected Output (compression ratios for a 145KB JSON response):**
```
No compression:  145,000 bytes
Gzip (level 6):   12,450 bytes  (91.4% reduction)
Brotli (level 6):  8,420 bytes  (94.2% reduction)
```

**Why this output:** Nginx compresses the JSON response using Brotli when the client supports it, falling back to Gzip. The `Vary: Accept-Encoding` header tells caches that the response varies by encoding. Brotli achieves a 94% reduction — 4 KB smaller than Gzip.

### Real-World Cases

- **APIs:** Compressing JSON responses for mobile clients.
- **Web applications:** Compressing HTML, CSS, and JavaScript bundles.
- **CDNs:** Cloudflare automatically compresses all text-based assets.

---

## Core Concept 3: Caching (Cache-Control Directives)

### Definitions

**Core Definition:** Cache-Control is an HTTP header that specifies caching directives for browsers and intermediary caches (CDNs, proxies), controlling how long a response can be cached and under what conditions.

**Technical Definition:** Cache-Control directives include: `max-age=N` (seconds the response is fresh in browser caches), `s-maxage=N` (seconds the response is fresh in shared caches like CDNs), `public` (may be cached by any cache), `private` (only browser cache), `no-cache` (must revalidate before using cached copy), `no-store` (never cache), `immutable` (never revalidate during freshness lifetime), `stale-while-revalidate=N` (serve stale while revalidating in background), and `stale-if-error=N` (serve stale if origin is down). The choice of directives depends on the resource type: static assets with hashed filenames can be cached for a year with `immutable`; API responses typically use shorter `max-age` or `no-store`. 

**Beginner-Friendly Explanation:** Cache-Control is like a "best before" date on food. `max-age=3600` means the food is good for an hour. `no-store` means eat it immediately, don't save it. `immutable` means it never goes bad (use the hashed version). The browser and CDN read these instructions and decide whether to reuse the cached copy or fetch a fresh one. 

### Purposes

- To reduce server load by serving cached responses.
- To reduce latency for repeat visitors.
- To enable CDN caching of static and API responses.
- To control cache behaviour precisely per resource type.

### Syntax Rules and Structure

**Common Cache-Control directives:**
```
# Static assets with hashed filenames (1 year, immutable)
Cache-Control: public, max-age=31536000, immutable

# API responses (5 minutes, CDN caches for 10 minutes)
Cache-Control: public, max-age=300, s-maxage=600

# User-specific data (browser only, 5 minutes)
Cache-Control: private, max-age=300

# Sensitive data (never cache)
Cache-Control: no-store

# Always revalidate (no-cache doesn't mean no store)
Cache-Control: no-cache

# Serve stale while revalidating
Cache-Control: public, max-age=300, stale-while-revalidate=60
```

| Directive | Purpose |
|-----------|---------|
| `max-age=N` | Browser cache lifetime (seconds). |
| `s-maxage=N` | CDN/shared cache lifetime. |
| `public` | Any cache may store. |
| `private` | Browser only. |
| `no-cache` | Revalidate before use. |
| `no-store` | Never store. |
| `immutable` | Never revalidate during freshness. |
| `stale-while-revalidate` | Serve stale while refreshing. |

**Rules:**
- Use `immutable` only for assets with hashed filenames (e.g., `app.a1b2c3.js`). 
- Use `no-store` for sensitive data (auth tokens, personal data). 
- Use `private` for user-specific responses. 
- Set `s-maxage` higher than `max-age` to let CDNs cache longer than browsers. 
- Use `stale-while-revalidate` to improve perceived performance. 

### Annotated Code Example

```js
// cache-control.js
const express = require('express');
const path = require('path');
const app = express();

// Static assets — immutable, 1 year
app.use('/static', express.static(path.join(__dirname, 'public'), {
  maxAge: '1y',
  immutable: true,
  setHeaders: (res) => {
    res.set('Cache-Control', 'public, max-age=31536000, immutable');
  }
}));

// API — CDN caches for 10 min, browser for 5 min
app.get('/api/products', (req, res) => {
  res.set('Cache-Control', 'public, max-age=300, s-maxage=600');
  res.json([{ id: 1, name: 'Laptop' }]);
});

// User-specific — browser only, 5 min
app.get('/api/me', (req, res) => {
  res.set('Cache-Control', 'private, max-age=300');
  res.json({ id: 42, name: 'Alice' });
});

// Sensitive — never cache
app.get('/api/auth/token', (req, res) => {
  res.set('Cache-Control', 'no-store');
  res.json({ token: 'secret' });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (headers for each endpoint):**
```
GET /static/app.a1b2c3.js
Cache-Control: public, max-age=31536000, immutable

GET /api/products
Cache-Control: public, max-age=300, s-maxage=600

GET /api/me
Cache-Control: private, max-age=300

GET /api/auth/token
Cache-Control: no-store
```

**Why this output:** The static asset is cached for one year with `immutable` — the browser will never revalidate it because the filename contains a hash that changes when the content changes. The API products endpoint is cached by both browser and CDN with different TTLs. The user-specific endpoint is `private` (browser only). The token endpoint is `no-store` (never cached). 

### Real-World Cases

- **Static asset hosting:** Hashed filenames with `immutable` and one-year expiry.
- **Public APIs:** `s-maxage` for CDN caching, shorter `max-age` for browsers.
- **Authenticated APIs:** `private` or `no-store` to prevent shared cache leakage.
- **E-commerce:** Product listings cached for minutes; prices revalidated frequently.

---

## Core Concept 4: ETags

### Definitions

**Core Definition:** An ETag (entity tag) is a unique identifier for a specific version of a resource, used by clients to validate whether their cached copy is still current.

**Technical Definition:** An ETag is an opaque string that the server generates — typically an MD5 or SHA hash of the resource content, or a version identifier. The server sends it in the `ETag` response header. On subsequent requests, the client sends the ETag in the `If-None-Match` header. If the server's current ETag matches, the server returns `304 Not Modified` with no body. ETags are either **strong** (e.g., `"abc123"`) or **weak** (e.g., `W/"abc123"`). Strong ETags guarantee byte-for-byte identity; weak ETags indicate semantic equivalence. Express's `res.sendFile()` and `express.static()` generate ETags automatically based on file size and modification time. 

**Beginner-Friendly Explanation:** An ETag is like a fingerprint for a file. When the browser first downloads a file, the server sends its fingerprint. The next time the browser needs the file, it sends the fingerprint back and asks "is this still current?" If the server says "yes" (304 Not Modified), the browser uses its cached copy. If not, the server sends the new version. 

### Purposes

- To enable conditional requests that avoid re-downloading unchanged resources.
- To validate cached content without relying on timestamps.
- To detect content changes for APIs with frequently updated data.
- To reduce bandwidth for repeat visitors.

### Syntax Rules and Structure

**Express automatic ETags:**
```js
// express.static() and res.sendFile() generate ETags automatically
app.use(express.static('public', { etag: true }));  // Default: true

// Disable ETags
app.use(express.static('public', { etag: false }));
```

**Manual ETag for API responses:**
```js
const crypto = require('crypto');

app.get('/api/data', (req, res) => {
  const data = { version: 1, content: 'Hello' };
  const json = JSON.stringify(data);
  const etag = `"${crypto.createHash('md5').update(json).digest('hex')}"`;

  // Check If-None-Match
  if (req.headers['if-none-match'] === etag) {
    return res.status(304).end();
  }

  res.set('ETag', etag);
  res.set('Cache-Control', 'public, max-age=300');
  res.json(data);
});
```

| ETag Type | Example | Meaning |
|-----------|---------|---------|
| Strong | `"abc123"` | Byte-for-byte identical. |
| Weak | `W/"abc123"` | Semantically equivalent. |

**Rules:**
- Use strong ETags for exact byte matching; weak for semantic equivalence. 
- Express generates ETags automatically for static files and `res.sendFile()`. 
- For APIs, generate ETags from a hash of the response body. 
- Return `304 Not Modified` with no body when the ETag matches. 
- ETags are per-resource — do not use the same ETag for different resources. 

### Annotated Code Example

```js
// etags.js
const express = require('express');
const crypto = require('crypto');
const app = express();

const data = { version: 1, content: 'Hello, World!' };
const etag = `"${crypto.createHash('md5').update(JSON.stringify(data)).digest('hex')}"`;

app.get('/api/data', (req, res) => {
  // Check conditional request
  if (req.headers['if-none-match'] === etag) {
    console.log('304 Not Modified — client cache is valid');
    return res.status(304).end();
  }

  console.log('200 OK — sending fresh data');
  res.set('ETag', etag);
  res.set('Cache-Control', 'public, max-age=300');
  res.json(data);
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (first request):**
```
200 OK — sending fresh data
ETag: "5d41402abc4b2a76b9719d911017c592"
Content-Length: 34
```

**Expected Output (subsequent request with `If-None-Match: "5d41402abc4b2a76b9719d911017c592"`):**
```
304 Not Modified — client cache is valid
(no body)
```

**Why this output:** The server computes an MD5 hash of the JSON body and uses it as the ETag. The client sends the ETag back in `If-None-Match`. If it matches, the server returns 304 with no body — saving the bandwidth of re-sending the data. The client uses its cached copy. 

### Real-World Cases

- **Static assets:** Express generates ETags for files served via `express.static()`. 
- **API responses:** Generating ETags from response body hashes for cacheable endpoints.
- **CDN integration:** CDNs use ETags to validate cached content against the origin.

---

## Core Concept 5: Conditional Requests

### Definitions

**Core Definition:** Conditional requests are HTTP requests that include a precondition header (`If-None-Match`, `If-Modified-Since`, `If-Match`, `If-Unmodified-Since`, `If-Range`) telling the server to execute the request only if the condition is met.

**Technical Definition:** The two most common conditional request headers for caching are `If-None-Match` (used with ETags) and `If-Modified-Since` (used with `Last-Modified` timestamps). When the server receives a conditional GET, it checks the condition; if the cached version is still valid, it returns `304 Not Modified` with no body. If the condition fails, it returns `200 OK` with the full response. Express handles `If-None-Match` and `If-Modified-Since` automatically for `res.sendFile()` and `express.static()`. For custom API endpoints, the developer must implement the check manually. 

**Beginner-Friendly Explanation:** A conditional request is like asking "has anything changed since last time?" before downloading a file. If the answer is no (304 Not Modified), you skip the download and use what you already have. If the answer is yes (200 OK), you download the new version. This saves bandwidth and time.

### Purposes

- To avoid re-downloading unchanged resources.
- To reduce bandwidth and server load.
- To enable efficient caching with precise validation.
- To support partial content with `If-Range`.

### Syntax Rules and Structure

**Conditional request headers:**

| Header | Paired With | Purpose |
|--------|------------|---------|
| `If-None-Match` | `ETag` | Return 304 if ETag matches. |
| `If-Modified-Since` | `Last-Modified` | Return 304 if not modified since. |
| `If-Match` | `ETag` | Execute only if ETag matches. |
| `If-Unmodified-Since` | `Last-Modified` | Execute only if not modified since. |
| `If-Range` | `ETag` or `Last-Modified` | Partial content if unchanged. |

**Express automatic handling:**
```js
// res.sendFile() handles If-None-Match and If-Modified-Since automatically
app.get('/file', (req, res) => {
  res.sendFile('/path/to/file.pdf');
});
```

**Manual handling for APIs:**
```js
app.get('/api/data', (req, res) => {
  const lastModified = new Date('2026-01-15T00:00:00Z').toUTCString();
  const ifModifiedSince = req.headers['if-modified-since'];

  if (ifModifiedSince && new Date(ifModifiedSince) >= new Date(lastModified)) {
    return res.status(304).end();
  }

  res.set('Last-Modified', lastModified);
  res.json({ data: 'current' });
});
```

**Rules:**
- `If-None-Match` takes precedence over `If-Modified-Since` when both are present. 
- Return `304 Not Modified` with no body when the condition indicates the cache is valid. 
- Include `Cache-Control` headers alongside ETags and Last-Modified. 
- Express handles conditional requests automatically for file serving; APIs require manual handling. 
- Use `If-Range` with range requests to ensure partial content is from the same version. 

### Annotated Code Example

```js
// conditional-requests.js
const express = require('express');
const crypto = require('crypto');
const app = express();

const data = { version: 1, content: 'Hello' };
const lastModified = new Date('2026-01-15T00:00:00Z').toUTCString();
const etag = `"${crypto.createHash('md5').update(JSON.stringify(data)).digest('hex')}"`;

app.get('/api/data', (req, res) => {
  // Check If-None-Match (ETag)
  if (req.headers['if-none-match'] === etag) {
    console.log('304 via If-None-Match');
    return res.status(304).end();
  }

  // Check If-Modified-Since (timestamp)
  const ifModifiedSince = req.headers['if-modified-since'];
  if (ifModifiedSince && new Date(ifModifiedSince) >= new Date(lastModified)) {
    console.log('304 via If-Modified-Since');
    return res.status(304).end();
  }

  // No valid cache — send full response
  res.set('ETag', etag);
  res.set('Last-Modified', lastModified);
  res.set('Cache-Control', 'public, max-age=300');
  res.json(data);
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (first request):**
```
200 OK
ETag: "5d41402abc4b2a76b9719d911017c592"
Last-Modified: Wed, 15 Jan 2026 00:00:00 GMT
Cache-Control: public, max-age=300
{"version":1,"content":"Hello"}
```

**Expected Output (second request with `If-None-Match`):**
```
304 via If-None-Match
(no body)
```

**Expected Output (second request with `If-Modified-Since`):**
```
304 via If-Modified-Since
(no body)
```

**Why this output:** The first request receives the full response with ETag, Last-Modified, and Cache-Control. The second request includes `If-None-Match` (with the ETag) or `If-Modified-Since` (with the timestamp). The server checks the condition and returns 304 Not Modified with no body — the client uses its cached copy.

### Real-World Cases

- **Static assets:** Browsers send `If-None-Match` for cached CSS, JS, and images.
- **APIs:** Clients validate cached API responses with `If-None-Match`.
- **Resumable downloads:** `If-Range` ensures partial content is from the same file version.

---

## Core Concept 6: HTTP/2 and HTTP/3 Upgrades

### Definitions

**Core Definition:** HTTP/2 and HTTP/3 are major revisions of the HTTP protocol that introduce multiplexing (multiple requests over one connection), header compression, and — for HTTP/3 — the elimination of TCP head-of-line blocking via QUIC over UDP.

**Technical Definition:** HTTP/2 (RFC 7540, 2015) introduces binary framing, multiplexing (multiple streams over one TCP connection), header compression (HPACK), and server push. HTTP/3 (RFC 9114, 2022) runs over QUIC (RFC 9000), a UDP-based transport that provides stream multiplexing at the transport layer, eliminating TCP head-of-line blocking — a lost packet only blocks the stream it belongs to, not all streams. HTTP/3 also reduces connection setup latency (0-RTT or 1-RTT handshakes) and supports connection migration (changing networks without dropping the connection). Node.js supports HTTP/2 via the built-in `http2` module and HTTP/3 via experimental modules or third-party libraries.

**Beginner-Friendly Explanation:** HTTP/1.1 is like a single-lane road — cars (requests) must travel one at a time, and if one car breaks down (a lost packet), everyone behind it waits. HTTP/2 is like a multi-lane highway — multiple cars travel side by side on the same road. HTTP/3 is like a fleet of drones — each package travels independently, so one drone crashing does not affect the others.

### Purposes

- To eliminate HTTP-level head-of-line blocking (HTTP/2).
- To eliminate TCP-level head-of-line blocking (HTTP/3).
- To reduce connection setup latency with 0-RTT/1-RTT handshakes (HTTP/3).
- To enable connection migration on mobile networks (HTTP/3).
- To compress headers and reduce overhead.

### Syntax Rules and Structure

**Node.js HTTP/2 server:**
```js
const http2 = require('node:http2');
const fs = require('node:fs');

const server = http2.createSecureServer({
  key: fs.readFileSync('server.key'),
  cert: fs.readFileSync('server.crt')
});

server.on('stream', (stream, headers) => {
  stream.respond({
    'content-type': 'application/json',
    ':status': 200
  });
  stream.end(JSON.stringify({ message: 'Hello HTTP/2' }));
});

server.listen(8443, () => console.log('HTTP/2 server on 8443'));
```

**Nginx HTTP/2 and HTTP/3:**
```nginx
server {
  listen 443 ssl;
  http2 on;
  http3 on;
  quic_retry on;

  ssl_certificate /path/to/cert.pem;
  ssl_certificate_key /path/to/key.pem;
}
```

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---------|----------|--------|--------|
| Multiplexing | ❌ | ✅ | ✅ |
| Header compression | ❌ | HPACK | QPACK |
| Head-of-line blocking | HTTP + TCP | TCP only | None |
| Transport | TCP | TCP | QUIC (UDP) |
| Connection setup | 1–3 RTT | 1–3 RTT | 0–1 RTT |
| Server push | ❌ | ✅ | ✅ |

**Rules:**
- HTTP/2 and HTTP/3 require HTTPS (TLS). 
- Use Nginx or a CDN to terminate HTTP/2 and HTTP/3; Node.js handles the backend HTTP/1.1. 
- HTTP/3 requires UDP port 443 to be open. 
- Enable HTTP/2 first (broader support); add HTTP/3 when clients support it. 
- Monitor protocol adoption via analytics (Cloudflare reports HTTP/3 usage). 

### Annotated Code Example

```nginx
# http2-http3.conf
server {
    # HTTP/2 over TLS
    listen 443 ssl;
    http2 on;

    # HTTP/3 over QUIC
    listen 443 quic reuseport;
    http3 on;
    quic_retry on;

    # TLS configuration
    ssl_certificate /etc/ssl/certs/example.com.pem;
    ssl_certificate_key /etc/ssl/private/example.com.key;
    ssl_protocols TLSv1.2 TLSv1.3;

    # Advertise HTTP/3 availability
    add_header Alt-Svc 'h3=":443"; ma=86400';

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
    }
}
```

**Expected Output (response headers):**
```
# HTTP/2 client
HTTP/2 200
content-type: application/json

# HTTP/3 client (Alt-Svc advertisement)
alt-svc: h3=":443"; ma=86400
```

**Expected Output (performance comparison):**
```
HTTP/1.1 (6 requests, 100ms RTT): ~600ms (sequential)
HTTP/2   (6 requests, 100ms RTT): ~150ms (multiplexed)
HTTP/3   (6 requests, 100ms RTT): ~120ms (multiplexed + 0-RTT)
```

**Why this output:** HTTP/1.1 processes requests sequentially (or with limited parallelism). HTTP/2 multiplexes all six requests over one connection, reducing total time to roughly one round trip plus processing. HTTP/3 further reduces latency with faster connection setup and no TCP head-of-line blocking. The `Alt-Svc` header tells the browser that HTTP/3 is available on port 443.

### Real-World Cases

- **Cloudflare:** Enables HTTP/2 and HTTP/3 automatically for all proxied domains.
- **Google:** Reports that HTTP/3 reduces search latency by 12% and YouTube rebuffer time by 15%.
- **Nginx:** Terminates HTTP/2 and HTTP/3 at the edge, proxying to Node.js over HTTP/1.1.

---

## Core Concept 7: Content Delivery Networks (CDNs)

### Definitions

**Core Definition:** A Content Delivery Network (CDN) is a geographically distributed network of edge servers that cache and serve content from locations close to the user, reducing latency and offloading traffic from the origin server.

**Technical Definition:** A CDN works by DNS resolution: when a user requests `cdn.example.com/asset.js`, DNS returns the IP address of the nearest edge server (based on geographic proximity or anycast routing). The edge server checks its cache; on a hit, it serves the asset directly; on a miss, it fetches from the origin, caches it, and serves it. CDNs cache based on `Cache-Control` headers (`s-maxage`, `public`) and ETags. They also provide TLS termination, compression (Brotli, Gzip), DDoS protection, and HTTP/3 support. 

**Beginner-Friendly Explanation:** A CDN is like having copies of your store in every city instead of one central warehouse. When someone in Tokyo orders a product, they get it from the Tokyo store, not the New York warehouse. This is faster for the customer and reduces the load on the central warehouse. 

### Purposes

- To reduce latency by serving content from edge servers near the user.
- To offload static asset delivery from the origin server.
- To absorb traffic spikes and DDoS attacks.
- To provide TLS termination, compression, and HTTP/3 at the edge.
- To reduce origin bandwidth costs.

### Syntax Rules and Structure

**CloudFront distribution (AWS):**
```js
// CloudFront distribution configuration
{
  "Origins": [
    {
      "DomainName": "origin.example.com",
      "OriginPath": "/static",
      "CustomOriginConfig": {
        "HTTPSPort": 443,
        "OriginProtocolPolicy": "https-only"
      }
    }
  ],
  "DefaultCacheBehavior": {
    "ViewerProtocolPolicy": "redirect-to-https",
    "CachePolicyId": "658327ea-f89d-4fab-a63d-7e88639e58f6",
    "Compress": true
  }
}
```

**Cache-Control for CDN:**
```
# CDN caches for 1 hour, browser for 5 minutes
Cache-Control: public, max-age=300, s-maxage=3600

# CDN caches for 1 year (immutable hashed asset)
Cache-Control: public, max-age=31536000, immutable
```

| CDN | Features |
|-----|----------|
| Cloudflare | HTTP/3, Brotli, automatic caching, DDoS protection. |
| CloudFront | AWS integration, Lambda@Edge, signed URLs. |
| Fastly | VCL configuration, instant purge, edge computing. |
| Akamai | Global scale, enterprise features. |

**Rules:**
- Set `Cache-Control: s-maxage` to control CDN caching separately from browser caching. 
- Use hashed filenames (`app.a1b2c3.js`) with `immutable` for long-term caching. 
- Purge the CDN cache when deploying new content or changing API responses. 
- Use CDN-signed URLs for private or authenticated content. 
- Configure the CDN to forward `Accept-Encoding` and `Authorization` headers appropriately. 

### Annotated Code Example

```js
// cdn-setup.js
const express = require('express');
const app = express();

// Static assets — CDN caches for 1 year
app.use('/static', express.static('public', {
  maxAge: '1y',
  immutable: true,
  setHeaders: (res) => {
    res.set('Cache-Control', 'public, max-age=31536000, s-maxage=31536000, immutable');
  }
}));

// API — CDN caches for 10 minutes, browser for 5
app.get('/api/products', (req, res) => {
  res.set('Cache-Control', 'public, max-age=300, s-maxage=600');
  res.set('CDN-Cache-Control', 'max-age=600');  // Cloudflare-specific
  res.json([{ id: 1, name: 'Laptop' }]);
});

// Private — never CDN cached
app.get('/api/me', (req, res) => {
  res.set('Cache-Control', 'private, no-store');
  res.json({ id: 42 });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (CDN cache behaviour):**
```
GET /static/app.a1b2c3.js
X-Cache: HIT (from CDN edge)
Cache-Control: public, max-age=31536000, s-maxage=31536000, immutable

GET /api/products
X-Cache: MISS (first request, fetched from origin)
X-Cache: HIT (subsequent requests within 10 minutes)

GET /api/me
X-Cache: BYPASS (private, no-store)
```

**Why this output:** The static asset is cached at the CDN edge for one year — subsequent requests are served from the edge without touching the origin. The API products endpoint is cached for 10 minutes at the CDN, reducing origin load. The private endpoint is never cached by the CDN (`no-store`). 

### Real-World Cases

- **Static asset delivery:** JS, CSS, images, and fonts served from CDN edges.
- **Video streaming:** CDNs deliver video segments from edge servers.
- **API acceleration:** CDNs cache API responses with short TTLs.
- **DDoS protection:** CDNs absorb volumetric attacks at the edge.

---

## References

- MDN HTTP Caching — https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching
- MDN Cache-Control — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control
- MDN ETag — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag
- MDN Conditional Requests — https://developer.mozilla.org/en-US/docs/Web/HTTP/Conditional_requests
- RFC 7232 — Conditional Requests — https://www.rfc-editor.org/rfc/rfc7232
- RFC 7540 — HTTP/2 — https://www.rfc-editor.org/rfc/rfc7540
- RFC 9114 — HTTP/3 — https://www.rfc-editor.org/rfc/rfc9114
- RFC 9000 — QUIC — https://www.rfc-editor.org/rfc/rfc9000
- Node.js HTTP/2 Documentation — https://nodejs.org/api/http2.html
- Nginx HTTP/2 Module — https://nginx.org/en/docs/http/ngx_http_v2_module.html
- Cloudflare: What is HTTP/3? — https://www.cloudflare.com/learning/performance/what-is-http3/
- Cloudflare: What is a CDN? — https://www.cloudflare.com/learning/cdn/what-is-a-cdn/
- Google: HTTP/3 Performance — https://blog.chromium.org/2020/10/chrome-http3.html
- web.dev: HTTP Caching — https://web.dev/articles/http-cache
- web.dev: Content Delivery Networks — https://web.dev/articles/content-delivery-networks
- Cloudflare Cache-Control — https://developers.cloudflare.com/cache/concepts/cache-control/
- AWS CloudFront Caching — https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/ConfiguringCaching.html