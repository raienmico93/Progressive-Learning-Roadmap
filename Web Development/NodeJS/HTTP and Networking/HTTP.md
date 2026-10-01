# HTTP Fundamentals & Web Protocols — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** HTTP (Hypertext Transfer Protocol) is a stateless, application-layer request-response protocol used for transmitting hypermedia documents and data between clients and servers on the World Wide Web.

**Technical Definition:** The Hypertext Transfer Protocol (HTTP) is a family of stateless, application-level, request/response protocols that share a generic interface, extensible semantics, and self-descriptive messages to enable flexible interaction with network-based hypertext information systems. HTTP relies on the concept of resources — data or services identified by a Uniform Resource Identifier (URI) — and defines a set of request methods to indicate the desired action on a resource.

**Beginner-Friendly Explanation:** HTTP is the language that web browsers and web servers use to talk to each other. When you type a URL into your browser, the browser sends an HTTP request to a server asking for a page, and the server sends back an HTTP response with the page content. Think of it like ordering food at a restaurant: you (the client) place an order (request), and the kitchen (the server) prepares and delivers your meal (response).

### Key Characteristics

- **Stateless:** Each request is independent; the server does not retain session state between requests by default.
- **Request-Response:** Communication follows a strict client-initiates, server-responds pattern.
- **Text-based (HTTP/1.x):** HTTP/1.x messages are human-readable text; HTTP/2 and HTTP/3 use binary framing.
- **Extensible:** Headers allow metadata to be added without changing the protocol.
- **Cacheable:** Responses can be cached to improve performance.
- **Connection-oriented:** HTTP/1.1 uses persistent connections; HTTP/2 multiplexes over a single connection; HTTP/3 uses QUIC over UDP.

### Prerequisites

- **Basic networking knowledge:** Understanding of TCP/IP, ports, and DNS.
- **Familiarity with URLs:** How URLs are structured (scheme, host, path, query).
- **Basic web development:** Awareness of how browsers and servers interact.
- **Text editor and terminal:** For examining raw HTTP messages with tools like `curl` or `telnet`.

### Related Programming Areas

- **Web Development:** Building REST APIs, web servers, and frontend applications.
- **Networking:** TCP/IP, TLS/SSL, DNS, and socket programming.
- **Security:** HTTPS, CORS, CSP, HSTS, and cookie security.
- **Performance:** Caching, compression, HTTP/2 multiplexing, and CDN optimisation.
- **DevOps:** Load balancing, reverse proxies, and API gateways.

### Core Concepts

1. **Anatomy of an HTTP Request** — start line, headers, blank line, body.
2. **Anatomy of an HTTP Response** — status line, headers, blank line, body.
3. **Safe vs. Idempotent Methods** — method semantics.
4. **HTTP Methods** — GET, POST, PUT, PATCH, DELETE, OPTIONS, HEAD.
5. **Status Codes & Error Semantics** — 1xx through 5xx.
6. **State, Metadata, and Security** — headers, payloads, cookies, sessions.
7. **Protocol Evolutions** — HTTP/1.1, HTTP/2, HTTP/3.

---

## Core Concept 1: Anatomy of an HTTP Request

### Definitions

**Core Definition:** An HTTP request is a message sent by a client to a server, consisting of a request line, zero or more header fields, a blank line, and an optional message body.

**Technical Definition:** HTTP messages consist of requests from client to server and responses from server to client. Both types of message consist of a start-line, zero or more header fields, an empty line (i.e., a line with nothing preceding the CRLF) indicating the end of the header fields, and possibly a message-body. The start-line is called the "request-line" in requests.

**Beginner-Friendly Explanation:** An HTTP request is like a letter you send to a server. The first line says what you want (e.g., "GET me the homepage"). The next lines are labels with extra information (e.g., "I accept JSON" or "I'm using Chrome"). A blank line separates the labels from the actual content (if any). The content is the body — like a file you're uploading.

### Purposes

- To convey the client's intent (method and target resource) to the server.
- To provide metadata about the request (headers) for content negotiation, authentication, and caching.
- To carry data (body) for methods like POST, PUT, and PATCH.

### Syntax Rules and Structure

**General structure:**
```
Request-Line CRLF
Header-Field: value CRLF
...
CRLF
[Message-Body]
```

**Request-Line:**
```
<Method> <Request-URI> <HTTP-Version> CRLF
```
| Component | Breakdown |
|-----------|-----------|
| Method | The HTTP verb (GET, POST, etc.). |
| Request-URI | The path to the resource (e.g., `/index.html`). |
| HTTP-Version | e.g., `HTTP/1.1`. |
| CRLF | Carriage return + line feed (`\r\n`). |

**Header fields:**
```
Header-Name: value CRLF
```
- Field names are case-insensitive.
- The field value may be preceded by whitespace.
- Header fields can be extended over multiple lines by preceding extra lines with at least one space or tab.

**Constraints and Limitations:**
- Each line must end with CRLF (`\r\n`), though servers should gracefully handle lines ending in just LF.
- The blank line (CRLF) is mandatory even if there is no body.
- The request-line and headers must not be prefaced or followed by extra CRLF.

### Annotated Code Example

```http
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Content-Length: 48
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

{"name": "Alice", "email": "alice@example.com"}
```

**Expected Output (server response):**
```http
HTTP/1.1 201 Created
Location: /api/users/42
```

**Why this output:** The client sends a POST request to `/api/users` with a JSON body containing user data. The `Host` header is mandatory in HTTP/1.1. The `Content-Type` header tells the server the body is JSON. The `Content-Length` header specifies the body size. The server responds with 201 Created and a Location header pointing to the newly created resource.

### Real-World Cases

- **Form submissions:** Browser sends POST requests with form data.
- **API calls:** Frontend JavaScript uses `fetch()` to send JSON requests.
- **File uploads:** Multipart form-data requests carry binary files.

---

## Core Concept 2: Anatomy of an HTTP Response

### Definitions

**Core Definition:** An HTTP response is a message sent by a server to a client, consisting of a status line, zero or more header fields, a blank line, and an optional message body.

**Technical Definition:** HTTP response is a response line followed by zero or more response headers, possibly followed by content, with a blank line ("\r\n") separating headers from content. The response line, also called the status line, contains the protocol version, a numeric status code, and a human-readable reason phrase.

**Beginner-Friendly Explanation:** An HTTP response is like the server's reply to your letter. The first line says whether your request succeeded or failed (e.g., "200 OK" or "404 Not Found"). The following lines provide metadata about the response (e.g., content type, caching instructions). After a blank line, the actual content follows — the HTML page, JSON data, or image you requested.

### Purposes

- To inform the client of the outcome of the request (status code).
- To provide metadata about the response (headers) for caching, content type, and security.
- To deliver the requested resource or error details (body).

### Syntax Rules and Structure

**General structure:**
```
Status-Line CRLF
Header-Field: value CRLF
...
CRLF
[Message-Body]
```

**Status-Line:**
```
<HTTP-Version> <Status-Code> <Reason-Phrase> CRLF
```
| Component | Breakdown |
|-----------|-----------|
| HTTP-Version | e.g., `HTTP/1.1`. |
| Status-Code | 3-digit integer (e.g., `200`, `404`). |
| Reason-Phrase | Human-readable text (e.g., `OK`, `Not Found`). |

**Common response headers:**
| Header | Description |
|--------|-------------|
| `Content-Type` | MIME type of the response body. |
| `Content-Length` | Size of the body in bytes. |
| `Cache-Control` | Caching directives. |
| `Set-Cookie` | Sets a cookie on the client. |
| `Location` | Redirect target URL. |

**Constraints and Limitations:**
- The status code must be a 3-digit integer.
- The reason phrase is optional in HTTP/2 and HTTP/3 but present in HTTP/1.1.
- The blank line terminates headers and is mandatory before the body.

### Annotated Code Example

```http
HTTP/1.1 200 OK
Date: Wed, 15 Jan 2026 12:00:00 GMT
Server: nginx/1.24.0
Content-Type: text/html; charset=utf-8
Content-Length: 135
Cache-Control: public, max-age=3600

<!DOCTYPE html>
<html>
<head><title>Hello</title></head>
<body><h1>Hello, World!</h1></body>
</html>
```

**Expected Output (client renders the HTML):**
```
Hello, World!
```

**Why this output:** The server responds with 200 OK, indicating success. The `Content-Type` header tells the browser the body is HTML. The `Content-Length` header tells the browser how many bytes to expect. The `Cache-Control` header instructs the browser to cache the response for one hour. The body contains the HTML document.

### Real-World Cases

- **Web pages:** Browsers receive HTML responses and render them.
- **API responses:** Clients receive JSON responses and parse them.
- **Error pages:** Servers return 404 or 500 responses with error HTML.

---

## Core Concept 3: Safe vs. Idempotent Methods

### Definitions

**Core Definition:** A **safe** method does not modify server state (read-only), while an **idempotent** method produces the same result whether called once or multiple times.

**Technical Definition:** Request methods are considered safe if their defined semantics are essentially read-only; i.e., the client does not request, and does not expect, any state change on the origin server as a result of applying a safe method to a target resource. A request method is considered idempotent if the intended effect on the server of multiple identical requests with that method is the same as the effect for a single such request.

**Beginner-Friendly Explanation:** A **safe** method is like looking at a painting — you can look as many times as you want, and the painting doesn't change. An **idempotent** method is like pressing an elevator button — pressing it once or five times gets you to the same floor. POST is neither safe nor idempotent — it's like depositing money; doing it twice means you've deposited twice as much.

### Purposes

- To guide caching strategies (safe methods are cacheable).
- To inform retry logic (idempotent methods can be safely retried).
- To clarify API design (using the right method for the right operation).
- To improve fault tolerance in distributed systems.

### Syntax Rules and Structure

| Method | Safe | Idempotent | Cacheable |
|--------|------|------------|-----------|
| GET | Yes | Yes | Yes |
| HEAD | Yes | Yes | Yes |
| OPTIONS | Yes | Yes | No |
| POST | No | No | No |
| PUT | No | Yes | No |
| PATCH | No | No | No |
| DELETE | No | Yes | No |

**Constraints and Limitations:**
- Safety and idempotency are semantic guarantees, not enforced by the protocol.
- Servers may implement non-idempotent behaviour for idempotent methods (bad practice).
- Caching proxies rely on these properties; violating them can cause subtle bugs.

### Annotated Code Example

```js
// Demonstrating idempotency in practice
// PUT is idempotent: updating a user's email to the same value is safe to retry
async function updateUserEmail(userId, email) {
  const response = await fetch(`/api/users/${userId}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email }),
  });
  return response;
}

// Calling this function twice with the same email produces the same result
await updateUserEmail(42, 'alice@example.com');
await updateUserEmail(42, 'alice@example.com'); // Safe to retry

// POST is NOT idempotent: creating a user twice creates two users
async function createUser(name, email) {
  const response = await fetch('/api/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ name, email }),
  });
  return response;
}

// Calling this twice creates two separate users!
await createUser('Alice', 'alice@example.com'); // User 1
await createUser('Alice', 'alice@example.com'); // User 2 (duplicate!)
```

**Expected Output:**
```
PUT: Both calls result in the same user state (email is alice@example.com).
POST: Two separate users are created (IDs 1 and 2).
```

**Why this output:** PUT is idempotent — the second call overwrites the same resource with the same value, resulting in the same state. POST is not idempotent — each call creates a new resource. This is why retrying a failed POST can cause duplicate data.

### Real-World Cases

- **Retry logic:** HTTP clients can safely retry GET, PUT, and DELETE requests on network failure.
- **Caching:** Proxies cache GET responses because GET is safe and cacheable.
- **API design:** Using PUT for idempotent updates and POST for non-idempotent creations.

---

## Core Concept 4: HTTP Methods

### Sub-Feature 4.1: GET

#### Definitions

**Core Definition:** GET requests a representation of the specified resource and should only retrieve data without modifying it.

**Technical Definition:** The GET method requests a representation of the specified resource. Requests using GET should only retrieve data and should not contain request content. GET is safe, idempotent, and cacheable.

**Beginner-Friendly Explanation:** GET is like asking a librarian for a book. You're requesting information, not changing anything. You can ask for the same book as many times as you want, and the library stays the same.

#### Purposes

- To retrieve a resource (HTML page, JSON data, image).
- To query an API for data.
- To trigger read-only operations.

#### Syntax Rules and Structure

```http
GET /path?query=value HTTP/1.1
Host: example.com
```

**Constraints:** GET requests should not have a body. Data is sent via the URL query string.

#### Annotated Code Example

```http
GET /api/users?limit=10 HTTP/1.1
Host: api.example.com
Accept: application/json
```

**Expected Output:**
```json
[{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}]
```

**Why this output:** The client requests the first 10 users. The server returns a JSON array.

---

### Sub-Feature 4.2: POST

#### Definitions

**Core Definition:** POST submits data to the specified resource, often causing a change in state or side effects on the server.

**Technical Definition:** The POST method submits an entity to the specified resource, often causing a change in state or side effects on the server. POST is neither safe nor idempotent.

**Beginner-Friendly Explanation:** POST is like submitting a form at the post office. You're sending data that will cause something to happen — a new account created, a message sent, an order placed. Doing it twice means two separate actions.

#### Purposes

- To create a new resource.
- To submit form data.
- To trigger non-idempotent operations.

#### Syntax Rules and Structure

```http
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json

{"name": "Alice"}
```

**Constraints:** POST is not idempotent; retrying may create duplicates.

#### Annotated Code Example

```js
fetch('/api/users', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'Alice', email: 'alice@example.com' }),
});
```

**Expected Output:** `201 Created` with a `Location` header.

**Why this output:** The server creates a new user and returns 201 with the location of the new resource.

---

### Sub-Feature 4.3: PUT

#### Definitions

**Core Definition:** PUT replaces all current representations of the target resource with the request content.

**Technical Definition:** The PUT method replaces all current representations of the target resource with the request content. PUT is idempotent but not safe.

**Beginner-Friendly Explanation:** PUT is like overwriting a file on a USB drive. You replace the entire contents with new data. Doing it twice with the same data produces the same result.

#### Purposes

- To update an existing resource completely.
- To create a resource at a known URL (idempotent).

#### Syntax Rules and Structure

```http
PUT /api/users/42 HTTP/1.1
Content-Type: application/json

{"name": "Alice Updated", "email": "alice@example.com"}
```

**Constraints:** PUT replaces the entire resource; partial updates require PATCH.

---

### Sub-Feature 4.4: PATCH

#### Definitions

**Core Definition:** PATCH applies partial modifications to a resource.

**Technical Definition:** The PATCH method applies partial modifications to a resource. PATCH is neither safe nor idempotent.

**Beginner-Friendly Explanation:** PATCH is like editing a document — you change only a few words, not the whole thing. But unlike PUT, PATCH is not idempotent because applying the same patch twice may produce different results.

#### Purposes

- To update specific fields of a resource without replacing it entirely.
- To apply JSON Patch or Merge Patch operations.

#### Syntax Rules and Structure

```http
PATCH /api/users/42 HTTP/1.1
Content-Type: application/json-patch+json

[{"op": "replace", "path": "/email", "value": "new@example.com"}]
```

**Constraints:** PATCH is not idempotent; retrying may cause issues. Use PUT for idempotent updates.

---

### Sub-Feature 4.5: DELETE

#### Definitions

**Core Definition:** DELETE removes the specified resource.

**Technical Definition:** The DELETE method deletes the specified resource. DELETE is idempotent but not safe.

**Beginner-Friendly Explanation:** DELETE is like throwing away a document. The first time, it's gone. The second time, it's already gone — the result is the same (the resource doesn't exist), so it's idempotent.

#### Purposes

- To remove a resource permanently.
- To clean up data.

#### Syntax Rules and Structure

```http
DELETE /api/users/42 HTTP/1.1
Host: api.example.com
```

**Constraints:** DELETE may be idempotent but the response code may differ (204 vs 404).

---

### Sub-Feature 4.6: OPTIONS

#### Definitions

**Core Definition:** OPTIONS describes the communication options for the target resource.

**Technical Definition:** The OPTIONS method describes the communication options for the target resource. OPTIONS is safe and idempotent.

**Beginner-Friendly Explanation:** OPTIONS is like asking a restaurant, "What methods of payment do you accept?" You're not ordering anything; you're just finding out what's possible.

#### Purposes

- To discover allowed methods on a resource.
- To support CORS preflight requests.

#### Syntax Rules and Structure

```http
OPTIONS /api/users HTTP/1.1
Host: api.example.com
Access-Control-Request-Method: POST
```

**Expected Output:**
```http
HTTP/1.1 204 No Content
Allow: GET, POST, OPTIONS
Access-Control-Allow-Methods: GET, POST, OPTIONS
```

**Why this output:** The server tells the client which methods are allowed on the resource.

---

### Sub-Feature 4.7: HEAD

#### Definitions

**Core Definition:** HEAD requests a response identical to GET but without the response body.

**Technical Definition:** The HEAD method asks for a response identical to a GET request, but without the response body. HEAD is safe, idempotent, and cacheable.

**Beginner-Friendly Explanation:** HEAD is like checking a book's cover and table of contents without reading the whole book. You get metadata (size, content type) without the content.

#### Purposes

- To check if a resource exists without downloading it.
- To get metadata (Content-Length, Content-Type) for caching.
- To test hyperlinks.

#### Syntax Rules and Structure

```http
HEAD /large-file.zip HTTP/1.1
Host: example.com
```

**Expected Output:**
```http
HTTP/1.1 200 OK
Content-Length: 104857600
Content-Type: application/zip
```

**Why this output:** The server returns headers identical to GET but no body. The client learns the file is 100 MB without downloading it.

---

## Core Concept 5: Status Codes & Error Semantics

### Definitions

**Core Definition:** HTTP status codes are 3-digit integers that indicate the outcome of a request, grouped into five classes: 1xx (Informational), 2xx (Success), 3xx (Redirection), 4xx (Client Error), and 5xx (Server Error).

**Technical Definition:** The first digit of the status code defines the class of response. The last two digits do not have any categorising role. The classes are: 1xx Informational, 2xx Success, 3xx Redirection, 4xx Client Error, and 5xx Server Error.

**Beginner-Friendly Explanation:** Status codes are the server's way of saying "here's what happened." 200 means "here's your data." 404 means "I couldn't find that." 500 means "I broke." The first digit tells you the general category.

### Purposes

- To communicate the outcome of a request.
- To enable clients to handle errors programmatically.
- To support caching and redirects.
- To provide semantic meaning for API responses.

### Syntax Rules and Structure

| Class | Meaning | Common Codes |
|-------|---------|--------------|
| 1xx | Informational | 100 Continue, 101 Switching Protocols |
| 2xx | Success | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirection | 301 Moved Permanently, 302 Found, 304 Not Modified |
| 4xx | Client Error | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found |
| 5xx | Server Error | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable |

### Multiple Annotated Code Examples

#### Example 1: 2xx Success

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"id": 1, "name": "Alice"}
```

**Why this output:** 200 means the request succeeded and the response contains the requested data.

#### Example 2: 3xx Redirection

```http
HTTP/1.1 301 Moved Permanently
Location: https://new.example.com/page
```

**Why this output:** The resource has permanently moved to a new URL. The client should update its bookmarks.

#### Example 3: 4xx Client Error

```http
HTTP/1.1 404 Not Found
Content-Type: text/html

<h1>404 Not Found</h1>
```

**Why this output:** The server could not find the requested resource. The client should check the URL.

#### Example 4: 5xx Server Error

```http
HTTP/1.1 500 Internal Server Error
Content-Type: application/json

{"error": "Database connection failed"}
```

**Why this output:** The server encountered an unexpected condition. The client should retry later.

### Real-World Cases

- **REST APIs:** Using 201 for resource creation, 204 for successful deletion, 400 for validation errors.
- **Web browsers:** Handling 301/302 redirects automatically, showing 404 pages.
- **Monitoring:** Alerting on 5xx error rates.

---

## Core Concept 6: State, Metadata, and Security

### Sub-Feature 6.1: HTTP Headers (Core, Security Headers)

#### Definitions

**Core Definition:** HTTP headers are key-value pairs sent in requests and responses that provide metadata about the message, the resource, or the connection.

**Technical Definition:** HTTP header fields are components of the message header of requests and responses. They define the operating parameters of an HTTP transaction. Headers are case-insensitive and are separated from the body by a blank line.

**Beginner-Friendly Explanation:** Headers are like the labels on a package. They tell you what's inside, who sent it, how to handle it, and when it expires — without opening the box.

#### Purposes

- To provide metadata about the request or response.
- To enable content negotiation (Accept, Content-Type).
- To control caching (Cache-Control, ETag).
- To enforce security policies (CSP, HSTS, CORS).

#### Syntax Rules and Structure

```http
Header-Name: value
```

**Core headers:**
| Header | Description |
|--------|-------------|
| `Host` | The domain name of the server (mandatory in HTTP/1.1). |
| `User-Agent` | Client software identifier. |
| `Accept` | Media types the client can handle. |
| `Content-Type` | Media type of the body. |
| `Content-Length` | Size of the body in bytes. |
| `Authorization` | Credentials for authentication. |

**Security headers:**
| Header | Description |
|--------|-------------|
| `Content-Security-Policy` (CSP) | Controls which resources the browser is allowed to load. |
| `Strict-Transport-Security` (HSTS) | Forces HTTPS connections. |
| `X-Content-Type-Options` | Disables MIME sniffing. |
| `X-Frame-Options` | Controls whether the page can be embedded in an iframe. |
| `Access-Control-Allow-Origin` | CORS: which origins can access the resource. |

#### Annotated Code Example

```http
HTTP/1.1 200 OK
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.example.com
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Access-Control-Allow-Origin: https://app.example.com
```

**Why this output:** CSP restricts scripts to the same origin and a trusted CDN. HSTS forces HTTPS for one year. X-Content-Type-Options prevents MIME sniffing. X-Frame-Options prevents clickjacking. CORS allows only `https://app.example.com` to access the resource.

### Sub-Feature 6.2: Body Payloads (JSON, URL-encoded, Multipart form-data)

#### Definitions

**Core Definition:** The request or response body carries the actual data payload, formatted according to the `Content-Type` header.

**Technical Definition:** The message body (if any) of an HTTP message is used to carry the payload body for that message. The `Content-Type` header indicates the media type of the body. Common formats include `application/json`, `application/x-www-form-urlencoded`, and `multipart/form-data`.

**Beginner-Friendly Explanation:** The body is the actual content — the JSON data, the form fields, or the uploaded file. The `Content-Type` header tells the receiver how to interpret it.

#### Purposes

- To carry structured data in API requests and responses.
- To submit form data from browsers.
- To upload files.

#### Syntax Rules and Structure

| Content-Type | Format | Use Case |
|--------------|--------|----------|
| `application/json` | JSON object | REST APIs |
| `application/x-www-form-urlencoded` | `key=value&key2=value2` | HTML forms (simple) |
| `multipart/form-data` | Parts separated by boundary | File uploads |

**Constraints and Limitations:**
- `multipart/form-data` uses a boundary delimiter to separate parts.
- URL-encoded is inefficient for binary data.
- JSON is not suitable for file uploads.

#### Annotated Code Example

```http
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="username"

alice
------WebKitFormBoundary
Content-Disposition: form-data; name="avatar"; filename="photo.jpg"
Content-Type: image/jpeg

(binary JPEG data)
------WebKitFormBoundary--
```

**Why this output:** The boundary separates each form field. The first part is a text field (`username`). The second part is a file field (`avatar`) with its own Content-Type.

### Sub-Feature 6.3: Cookies and Sessions (HttpOnly, Secure, SameSite)

#### Definitions

**Core Definition:** Cookies are small pieces of data stored by the browser and sent with every request to the originating domain, used to maintain state in the otherwise stateless HTTP protocol.

**Technical Definition:** The `Set-Cookie` HTTP response header is used to send a cookie from the server to the user agent so that the user agent can send it back to the server later. Cookies are commonly used for session management, personalisation, and tracking.

**Beginner-Friendly Explanation:** Cookies are like a name tag the server gives you. Every time you visit, you show the name tag, and the server remembers who you are. Without cookies, HTTP would be like a goldfish — it forgets everything between requests.

#### Purposes

- To maintain user sessions after login.
- To store user preferences.
- To track user behaviour (analytics).

#### Syntax Rules and Structure

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600
```

| Attribute | Description |
|-----------|-------------|
| `HttpOnly` | Cookie is inaccessible to JavaScript; prevents XSS theft. |
| `Secure` | Cookie is only sent over HTTPS. |
| `SameSite` | Controls cross-site sending: `Strict`, `Lax`, or `None`. |
| `Max-Age` | Lifetime in seconds. |
| `Path` | URL path scope. |
| `Domain` | Domain scope. |

**Constraints and Limitations:**
- Cookies are limited to ~4 KB each.
- Cookies are sent with every request to the domain, increasing bandwidth.
- `SameSite=None` requires `Secure`.
- `HttpOnly` and `Secure` are independent attributes.

#### Annotated Code Example

```js
// Express.js: setting a secure session cookie
app.post('/login', (req, res) => {
  // Validate credentials...
  req.session.userId = user.id;
  res.cookie('session', sessionToken, {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 3600 * 1000, // 1 hour
  });
  res.json({ success: true });
});
```

**Expected Output:**
```http
Set-Cookie: session=abc123; Path=/; HttpOnly; Secure; SameSite=Strict; Max-Age=3600
```

**Why this output:** The cookie is HttpOnly (inaccessible to JavaScript), Secure (HTTPS only), and SameSite=Strict (not sent with cross-site requests), providing strong protection against XSS and CSRF.

### Real-World Cases

- **Authentication:** Session cookies after login.
- **CSRF protection:** SameSite=Strict prevents cross-site request forgery.
- **Analytics:** Tracking cookies (often SameSite=None; Secure).

---

## Core Concept 7: Protocol Evolutions

### Sub-Feature 7.1: HTTP/1.1 (Keep-Alive, Pipelining Limitations)

#### Definitions

**Core Definition:** HTTP/1.1 introduced persistent connections (keep-alive) and optional request pipelining, but pipelining suffers from head-of-line blocking.

**Technical Definition:** HTTP/1.1 phased out support for keep-alive connections, replacing them with an improved design called persistent connections. HTTP/1.1 permits optional request pipelining. However, pipelining increases the problem of head-of-line blocking since a request on a different connection might complete sooner. The client's inability to predict the length of requested actions limited the usefulness of pipelining.

**Beginner-Friendly Explanation:** HTTP/1.0 opened a new connection for every request. HTTP/1.1 keeps the connection open (keep-alive) so multiple requests can reuse it. Pipelining was supposed to let clients send multiple requests without waiting, but it caused problems because responses must come back in order — if the first request is slow, all others are blocked.

#### Purposes

- To reduce latency by reusing connections.
- To improve throughput by avoiding TCP handshake overhead.
- To enable multiple requests over a single connection.

#### Syntax Rules and Structure

```http
Connection: keep-alive
Keep-Alive: timeout=5, max=100
```

| Header | Description |
|--------|-------------|
| `Connection: keep-alive` | Requests persistent connection. |
| `Keep-Alive: timeout=5` | Connection idle timeout in seconds. |
| `Keep-Alive: max=100` | Maximum requests per connection. |

**Constraints and Limitations:**
- Head-of-line blocking: responses must be returned in request order.
- Pipelining is rarely implemented due to interoperability issues.
- Most browsers limit connections per host (typically 6).

### Sub-Feature 7.2: HTTP/2 (Multiplexing, HPACK, Server Push)

#### Definitions

**Core Definition:** HTTP/2 is a binary protocol that enables multiplexing of multiple requests over a single connection, header compression via HPACK, and optional server push.

**Technical Definition:** HTTP/2 enables a more efficient use of network resources and a reduced latency by introducing field compression and allowing multiple concurrent exchanges on the same connection. Multiplexing of requests is achieved by having each HTTP request/response exchange associated with its own stream. The HPACK format defines header compression.

**Beginner-Friendly Explanation:** HTTP/2 is like upgrading from a single-lane road to a multi-lane highway. Multiple requests can travel simultaneously without blocking each other. Headers are compressed to save bandwidth. The server can even send resources the client hasn't asked for yet (server push).

#### Purposes

- To eliminate head-of-line blocking at the HTTP layer.
- To reduce header overhead through compression.
- To enable server push for improved performance.
- To use a single connection for all requests to a host.

#### Syntax Rules and Structure

| Feature | Description |
|---------|-------------|
| Multiplexing | Multiple streams over one connection. |
| HPACK | Header compression reducing overhead. |
| Server Push | Server sends resources proactively. |
| Binary Framing | Messages split into binary frames. |
| Stream Prioritisation | Clients can prioritise streams. |

**Constraints and Limitations:**
- Server push is disabled in most browsers due to complexity and caching issues.
- TCP-level head-of-line blocking still exists (solved by HTTP/3).
- Requires TLS in practice (though the spec allows cleartext).

### Sub-Feature 7.3: HTTP/3 (QUIC over UDP, Connection Migration)

#### Definitions

**Core Definition:** HTTP/3 maps HTTP semantics over the QUIC transport protocol, which runs over UDP, providing stream multiplexing, per-stream flow control, and connection migration.

**Technical Definition:** The QUIC transport protocol has several features that are desirable in a transport for HTTP, such as stream multiplexing, per-stream flow control, and low-latency connection establishment. HTTP/3 describes a mapping of HTTP semantics over QUIC. Connection migration allows connections to survive changes to endpoint addresses (IP address and port), such as those caused by an endpoint migrating to a new network.

**Beginner-Friendly Explanation:** HTTP/3 is the newest version. Instead of TCP, it uses QUIC (a UDP-based protocol). This eliminates TCP head-of-line blocking entirely. If you switch from Wi-Fi to cellular, your connection survives (connection migration) — you don't need to reconnect.

#### Purposes

- To eliminate TCP head-of-line blocking.
- To reduce connection establishment latency (0-RTT).
- To support connection migration (seamless network changes).
- To provide better performance on lossy networks.

#### Syntax Rules and Structure

| Feature | Description |
|---------|-------------|
| Transport | QUIC over UDP (not TCP). |
| Multiplexing | Native, no head-of-line blocking. |
| Connection Migration | Survives IP/port changes via connection IDs. |
| 0-RTT | Zero round-trip time resumption. |
| Per-stream Flow Control | Independent flow control per stream. |

**Constraints and Limitations:**
- Requires UDP; some firewalls block UDP.
- Implementation complexity is higher than HTTP/2.
- Not all servers and clients support HTTP/3 yet.

### Real-World Cases

- **HTTP/1.1:** Legacy systems, simple APIs, internal services.
- **HTTP/2:** Modern web servers (nginx, Apache), CDNs, most production APIs.
- **HTTP/3:** Google, Cloudflare, Facebook; optimised for mobile and lossy networks.

---

## References

- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 — HTTP Caching — https://www.rfc-editor.org/rfc/rfc9111
- RFC 9112 — HTTP/1.1 — https://www.rfc-editor.org/rfc/rfc9112
- RFC 9113 — HTTP/2 — https://www.rfc-editor.org/rfc/rfc9113
- RFC 9114 — HTTP/3 — https://www.rfc-editor.org/rfc/rfc9114
- RFC 6265 — HTTP State Management Mechanism (Cookies) — https://www.rfc-editor.org/rfc/rfc6265
- IANA — HTTP Status Code Registry — https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
- MDN Web Docs — HTTP — https://developer.mozilla.org/en-US/docs/Web/HTTP
- MDN Web Docs — HTTP Request Methods — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods
- MDN Web Docs — HTTP Response Status Codes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status
- MDN Web Docs — HTTP Headers — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers
- MDN Web Docs — Using HTTP Cookies — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies
- MDN Web Docs — Cross-Origin Resource Sharing (CORS) — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS
- MDN Web Docs — Content Security Policy (CSP) — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP
- MDN Web Docs — Strict-Transport-Security (HSTS) — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
- OWASP — Secure Headers Project — https://owasp.org/www-project-secure-headers/
- web.dev — Security Headers Quick Reference — https://web.dev/articles/security-headers
- Cloudflare — What is HTTP/3? — https://www.cloudflare.com/learning/performance/what-is-http3/
- Google — QUIC, a multiplexed stream transport over UDP — https://www.chromium.org/quic/