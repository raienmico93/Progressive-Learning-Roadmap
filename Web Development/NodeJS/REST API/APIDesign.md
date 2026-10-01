# Advanced API Design & Response Formatting — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Advanced API design and response formatting is the discipline of designing HTTP APIs with predictable URL structures, consistent response envelopes, semantically correct status codes, and machine-readable API contracts that enable reliable integration across distributed systems.

**Technical Definition:** Advanced API design applies the architectural constraints of REST — specifically the Uniform Interface — through conventions for resource naming (plural nouns, kebab-case, limited nesting), standardized response formatting (envelope patterns, RFC 7807 Problem Details for errors), semantically accurate HTTP status code usage (201 Created, 202 Accepted, 204 No Content), and contract-driven development using the OpenAPI Specification (OAS). These conventions reduce integration friction, enable automated tooling (client generation, validation, documentation), and improve the safety of AI-driven API consumption via the Model Context Protocol.

**Beginner-Friendly Explanation:** Designing an API is like designing a public library. If every librarian organised books differently, visitors would be lost. Good API design establishes consistent rules: books (resources) are named with plural nouns (`/books`, not `/getBook`), every catalogue card (response) has the same structure, error messages use a standard format (so you know exactly what went wrong), and there's a published manual (OpenAPI spec) that tells everyone exactly how the library works.

### Key Characteristics

- **Resource-oriented URLs:** URLs identify resources (nouns), not operations (verbs); HTTP methods express the operation.
- **Consistent pluralisation:** Collections use plural nouns (`/users`), individual items use the same plural root with an identifier (`/users/42`).
- **Bounded nesting:** Resource hierarchies are limited to one or two levels of nesting to avoid coupling URLs to fragile data models.
- **Standardized error format:** RFC 7807 Problem Details provides a machine-readable, extensible error format using `application/problem+json`.
- **Semantic status codes:** Success codes (200, 201, 202, 204) are used according to the precise semantics of the operation, not interchangeably.
- **Contract-first capability:** OpenAPI Specification (OAS 3.1) enables design-first workflows where the contract is agreed before implementation.
- **AI-consumable:** Clear names, consistent shapes, and complete OpenAPI descriptions enable safe LLM tool selection via MCP.

### Prerequisites

- **HTTP fundamentals:** Request methods, status codes, headers, and caching semantics.
- **REST architectural constraints:** Client-Server, Stateless, Cache, Layered System, Uniform Interface.
- **JSON fundamentals:** Object structure, nesting, and serialization.
- **Basic understanding of API versioning:** URI versioning (`/v1/...`) or header-based versioning.
- **Familiarity with a server framework:** Express, Fastify, or NestJS.

### Related Programming Areas

- **HTTP Server Architecture:** URL routing, method handling, and response generation.
- **REST Fundamentals & Architectural Constraints:** The theoretical basis for resource-oriented design.
- **API Documentation:** OpenAPI Specification, Swagger UI, and Redoc.
- **Error Handling:** RFC 7807 Problem Details and consistent error envelopes.
- **AI Integration:** Model Context Protocol (MCP) and LLM tool selection.

### Core Concepts

1. **URL Structures & Resource Naming** — nouns vs. verbs, pluralisation, hierarchical relationships.
2. **Standardized Response Formats** — envelope vs. envelope-less, RFC 7807 Problem Details.
3. **HTTP Status Codes in Practice** — 2xx, 3xx, 4xx, 5xx; when to use 201, 202, 204.
4. **API Documentation & Contracts** — design-first vs. code-first, OpenAPI Specification.

---

## Core Concept 1: URL Structures & Resource Naming

### Sub-Feature 1.1: Nouns vs. Verbs (Pluralisation Consistency)

#### Definitions

**Core Definition:** REST API URLs should use nouns (not verbs) to identify resources, and collections should be named using plural nouns to maintain consistency across the API surface.

**Technical Definition:** The URL identifies what you're acting on; the HTTP method describes the action. Resource names should generally be plural, not singular, so `/customers` not `/customer`. A singular noun should be used for document names, while a plural noun should be used for collection names and store names. Using verbs in paths duplicates the information conveyed by the HTTP method and breaks the resource model, multiplying the API surface area.

**Beginner-Friendly Explanation:** Think of a URL as a filing cabinet label. You label the drawer "Customers" (a noun), not "GetAllCustomers" (a verb). When you want to retrieve customers, you open the drawer (GET). When you want to add a customer, you put a new file in the drawer (POST). The label stays the same; only the action changes.

#### Purposes

- To make APIs intuitive and predictable so developers can guess endpoints without documentation.
- To maintain a clean separation between resource identity (URL) and operation (HTTP method).
- To reduce the API surface area by avoiding verb-specific endpoints.
- To enable consistent caching behaviour by keeping the same URL for different operations.

#### Syntax Rules and Structure

| Rule | ✅ Correct | ❌ Incorrect |
|------|-----------|-------------|
| Use nouns, not verbs | `POST /orders` | `POST /createOrder` |
| Use plural for collections | `GET /users` | `GET /user` |
| Use same plural root for items | `GET /users/42` | `GET /user/42` |
| Use kebab-case for multi-word | `/order-items` | `/OrderItems`, `/order_items` |
| No file extensions in URLs | `/users` | `/users.json` |
| Version explicitly | `/v1/users` | `/users` (unversioned) |

**Constraints and Limitations:**
- Singletons are the exception: a user's single cart is `/users/42/cart` (singular, cardinality of one).
- Verb-based paths may be acceptable for non-CRUD operations (e.g., `/users/42/activate`), but should be documented as deliberate deviations.
- Path casing conventions must be consistent across the entire API.

#### Annotated Code Example

```javascript
// Express: Consistent noun-based URL structure
const express = require('express');
const app = express();

// Collection resource (plural)
app.get('/v1/orders', (req, res) => {
  res.json({ orders: [] });
});

// Item resource (same plural root)
app.get('/v1/orders/:orderId', (req, res) => {
  res.json({ orderId: req.params.orderId, status: 'pending' });
});

// Sub-resource (hierarchical relationship)
app.get('/v1/orders/:orderId/items', (req, res) => {
  res.json({ orderId: req.params.orderId, items: [] });
});

app.listen(3000);
```

**Expected Output (for `GET /v1/orders/1234/items`):**
```json
{"orderId":"1234","items":[]}
```

**Why this output:** The URL `/v1/orders/1234/items` follows the pattern `/collection/{id}/sub-collection`. The plural `orders` is used for both the collection and the item root. The sub-resource `items` is also plural. The URL is predictable: a developer familiar with the API can guess this endpoint without reading documentation.

#### Real-World Cases

- **Stripe API:** `/v1/charges`, `/v1/customers`, `/v1/payment_intents` — all plural nouns.
- **GitHub API:** `/repos/{owner}/{repo}/issues`, `/users/{username}` — plural collections with kebab-case.
- **Shopify API:** `/admin/api/2024-01/orders.json` — plural with version in path.

---

### Sub-Feature 1.2: Representing Hierarchical Relationships (e.g., `/users/:id/posts`)

#### Definitions

**Core Definition:** Hierarchical relationships in REST APIs are represented by nesting sub-resource paths under their parent resource, leveraging the hierarchical nature of URIs to imply aggregation or composition.

**Technical Definition:** A forward slash separator (`/`) must be used to indicate a hierarchical relationship. Resources should be nested at most one or two levels deep. The path `/customers/123/orders` represents "all orders belonging to customer 123," while `/orders/456` represents the order directly. Nesting deeper than two levels (e.g., `/customers/123/orders/456/items/789/notes`) couples the URL structure to the data model and breaks whenever relationships change.

**Beginner-Friendly Explanation:** Hierarchical URLs are like folder paths on your computer. `/Users/alice/Documents` makes sense — documents inside a user's folder. But `/Users/alice/Documents/Work/Reports/2026/January/Summary` is too deep. If you reorganise your folders, every path breaks. Keep nesting shallow and use direct resource access for deep retrieval.

#### Purposes

- To express clear parent-child relationships between resources.
- To enable intuitive navigation of the resource hierarchy.
- To support aggregation queries (e.g., "all orders for this customer").
- To avoid deep coupling between URL structure and data model.

#### Syntax Rules and Structure

| Pattern | Example | Semantics |
|---------|---------|-----------|
| Parent collection | `/users` | All users. |
| Parent item | `/users/42` | User 42. |
| Child collection | `/users/42/orders` | All orders for user 42. |
| Child item (flat) | `/orders/456` | Order 456 directly (preferred for deep access). |
| Child item (nested) | `/users/42/orders/456` | Order 456 belonging to user 42 (acceptable). |
| **Too deep** | `/users/42/orders/456/items/789/notes` | ❌ Coupled to data model. |

**Constraints and Limitations:**
- One level of nesting is normal; two levels is a smell; three levels is almost always wrong.
- When an item can exist independently of its parent, prefer a flat URL (`/orders/456`) over nested.
- Deeply nested URLs make caching, invalidation, and documentation significantly harder.

#### Annotated Code Example

```javascript
// Express: Shallow hierarchical nesting
const express = require('express');
const app = express();

// One level of nesting: user's posts (acceptable)
app.get('/v1/users/:userId/posts', (req, res) => {
  res.json({ userId: req.params.userId, posts: [] });
});

// Flat access for the post itself (preferred over deep nesting)
app.get('/v1/posts/:postId', (req, res) => {
  res.json({ postId: req.params.postId, title: 'Example' });
});

// Avoid this pattern (too deep):
// app.get('/v1/users/:userId/posts/:postId/comments/:commentId', ...)

// Instead, use flat access:
app.get('/v1/comments/:commentId', (req, res) => {
  res.json({ commentId: req.params.commentId, body: 'Great post!' });
});

app.listen(3000);
```

**Expected Output (for `GET /v1/users/42/posts`):**
```json
{"userId":"42","posts":[]}
```

**Expected Output (for `GET /v1/comments/789`):**
```json
{"commentId":"789","body":"Great post!"}
```

**Why this output:** The user's posts are accessed via `/users/42/posts` (one level of nesting). The post itself and its comments are accessed via flat URLs (`/posts/:postId` and `/comments/:commentId`). This keeps nesting shallow and avoids coupling the URL structure to the data model. If the relationship between users and posts changes, only the nested endpoint is affected.

#### Real-World Cases

- **GitHub API:** `/repos/{owner}/{repo}/issues` — one level of nesting, then flat access for individual issues.
- **Stripe API:** `/customers/{id}/subscriptions` — one level of nesting.
- **REST API Guidelines (Microsoft):** Recommend limiting nesting to one level and using flat access for deep retrieval.

---

## Core Concept 2: Standardized Response Formats

### Sub-Feature 2.1: Consistent Envelope vs. Envelope-less JSON Payloads

#### Definitions

**Core Definition:** A response envelope is a standard wrapper structure (e.g., `{ data, meta, error }`) applied consistently to all API responses, while envelope-less responses return the resource representation directly at the top level.

**Technical Definition:** Wrapping every response in a standard envelope makes life easier for API consumers because they always know where to find data, errors, and metadata. A common envelope pattern includes `data` (the payload), `meta` (pagination, timestamps, request ID), and `error` (error details). Envelope-less responses are simpler and more RESTful in spirit — the response body directly represents the resource — but provide no consistent place for metadata or error details.

**Beginner-Friendly Explanation:** An envelope is like a standard shipping box. Every package from your company arrives in the same shape, with a label in the same place. You always know where to find the invoice (meta) and the product (data). Envelope-less responses are like getting items in whatever packaging the sender had lying around — sometimes a box, sometimes a bag, sometimes nothing at all.

#### Purposes

- To provide a consistent, predictable structure for all responses.
- To include metadata (pagination, request IDs, timestamps) alongside the payload.
- To standardise error handling across all endpoints.
- To simplify client-side parsing and error handling logic.

#### Syntax Rules and Structure

**Envelope pattern:**
```json
{
  "data": { "id": 1, "name": "Alice" },
  "meta": { "requestId": "abc-123", "timestamp": "2026-01-15T12:00:00Z" },
  "error": null
}
```

**Error envelope:**
```json
{
  "data": null,
  "meta": { "requestId": "abc-123" },
  "error": { "code": "NOT_FOUND", "message": "User not found" }
}
```

**Envelope-less pattern:**
```json
{ "id": 1, "name": "Alice" }
```

| Aspect | Envelope | Envelope-less |
|--------|----------|---------------|
| Consistency | Always same top-level shape. | Varies by resource. |
| Metadata | Supported natively. | Requires HTTP headers or side channels. |
| Error handling | Standardised field. | Varies by endpoint. |
| REST purity | Less pure (wrapper). | More pure (direct representation). |
| Client complexity | Lower (single parser). | Higher (per-endpoint logic). |

**Constraints and Limitations:**
- Envelopes add a wrapper that some consider un-RESTful.
- Envelope-less responses require HTTP status codes and headers to carry all metadata.
- The choice must be applied consistently across the entire API.

#### Annotated Code Example

```javascript
// Express: Consistent response envelope middleware
const express = require('express');
const { randomUUID } = require('crypto');
const app = express();

// Middleware: attach request ID and envelope helpers
app.use((req, res, next) => {
  req.requestId = randomUUID();
  res.sendEnvelope = (data, status = 200) => {
    res.status(status).json({
      data,
      meta: {
        requestId: req.requestId,
        timestamp: new Date().toISOString(),
      },
      error: null,
    });
  };
  res.sendError = (code, message, status = 400) => {
    res.status(status).json({
      data: null,
      meta: { requestId: req.requestId },
      error: { code, message },
    });
  };
  next();
});

// Endpoint using the envelope
app.get('/v1/users/:id', (req, res) => {
  const user = { id: req.params.id, name: 'Alice' };
  if (!user) return res.sendError('NOT_FOUND', 'User not found', 404);
  res.sendEnvelope(user);
});

app.listen(3000);
```

**Expected Output (for `GET /v1/users/42`):**
```json
{
  "data": { "id": "42", "name": "Alice" },
  "meta": { "requestId": "a1b2c3d4-...", "timestamp": "2026-01-15T12:00:00.000Z" },
  "error": null
}
```

**Expected Output (for `GET /v1/users/999` if not found):**
```json
{
  "data": null,
  "meta": { "requestId": "a1b2c3d4-..." },
  "error": { "code": "NOT_FOUND", "message": "User not found" }
}
```

**Why this output:** The envelope middleware ensures every response has the same top-level structure. Clients can always access `response.data` for the payload, `response.meta` for metadata, and `response.error` for errors. This consistency simplifies client-side error handling and logging.

#### Real-World Cases

- **JSend specification:** A simple envelope with `status`, `data`, and `message` fields.
- **JSON:API specification:** A comprehensive envelope with `data`, `meta`, `links`, and `included`.
- **Google API Design Guide:** Uses a consistent envelope with `data` and `error` fields.

---

### Sub-Feature 2.2: Standardizing Error Responses (RFC 7807 Problem Details)

#### Definitions

**Core Definition:** RFC 7807 (Problem Details for HTTP APIs) defines a standard, machine-readable format for carrying error details in HTTP responses, using the media type `application/problem+json`.

**Technical Definition:** RFC 7807 defines a "problem detail" as a way to carry machine-readable details of errors in an HTTP response to avoid the need to define new error response formats for HTTP APIs. The Problem Details JSON Object has five canonical members: `type` (a URI reference identifying the problem type), `title` (a short, human-readable summary), `status` (the HTTP status code), `detail` (a human-readable explanation specific to this occurrence), and `instance` (a URI reference identifying the specific occurrence). Extension members may be added for additional context.

**Beginner-Friendly Explanation:** RFC 7807 is a standardised format for error messages in APIs. Instead of every API inventing its own error shape (`{"error": "Not found"}` here, `{"message": "User missing"}` there), RFC 7807 defines one shape that everyone can use. The `type` field tells the client what kind of error occurred, `title` gives a human-readable summary, `detail` explains the specific occurrence, and `instance` identifies which request failed.

#### Purposes

- To eliminate the proliferation of proprietary error schemas across APIs.
- To provide machine-readable error details that clients can parse programmatically.
- To enable error type identification via URIs (e.g., `https://api.example.com/problems/insufficient-credit`).
- To allow extension members for domain-specific error context.

#### Syntax Rules and Structure

```json
{
  "type": "https://api.example.com/problems/insufficient-credit",
  "title": "Insufficient Credit",
  "status": 403,
  "detail": "Your current balance is 30, but that costs 50.",
  "instance": "/account/12345/msgs/abc"
}
```

| Member | Required | Description |
|--------|----------|-------------|
| `type` | No (defaults to `about:blank`) | URI reference identifying the problem type. |
| `title` | No | Short, human-readable summary of the problem type. |
| `status` | No | HTTP status code generated by the origin server. |
| `detail` | No | Human-readable explanation specific to this occurrence. |
| `instance` | No | URI reference identifying the specific occurrence. |
| Extension members | No | Arbitrary additional fields for domain-specific context. |

**Media types:**
| Media Type | Format |
|------------|--------|
| `application/problem+json` | JSON serialization |
| `application/problem+xml` | XML serialization |

**Constraints and Limitations:**
- The `status` member duplicates the HTTP status code, creating the possibility of disagreement between the two.
- The `type` URI is the default value `about:blank` when omitted, indicating no additional semantics beyond the HTTP status code.
- Extension members should be carefully vetted to avoid leaking sensitive implementation details (e.g., stack traces).

#### Annotated Code Example

```javascript
// Express: RFC 7807 Problem Details middleware
const express = require('express');
const app = express();

// Custom problem class
class ProblemDetails extends Error {
  constructor({ type, title, status, detail, instance, ...extensions }) {
    super(detail);
    this.type = type || 'about:blank';
    this.title = title;
    this.status = status;
    this.detail = detail;
    this.instance = instance;
    Object.assign(this, extensions);
  }

  toJSON() {
    const { name, message, stack, ...problem } = this;
    return problem;
  }
}

// Error-handling middleware that emits RFC 7807
app.use((err, req, res, next) => {
  if (err instanceof ProblemDetails) {
    res
      .status(err.status || 500)
      .type('application/problem+json')
      .json(err);
  } else {
    res.status(500).type('application/problem+json').json({
      type: 'about:blank',
      title: 'Internal Server Error',
      status: 500,
      detail: 'An unexpected error occurred.',
    });
  }
});

// Route that throws a problem
app.get('/v1/account/:id', (req, res, next) => {
  next(
    new ProblemDetails({
      type: 'https://api.example.com/problems/insufficient-credit',
      title: 'Insufficient Credit',
      status: 403,
      detail: 'Your current balance is 30, but that costs 50.',
      instance: `/v1/account/${req.params.id}`,
      balance: 30,
      required: 50,
    })
  );
});

app.listen(3000);
```

**Expected Output (for `GET /v1/account/12345`):**
```http
HTTP/1.1 403 Forbidden
Content-Type: application/problem+json

{
  "type": "https://api.example.com/problems/insufficient-credit",
  "title": "Insufficient Credit",
  "status": 403,
  "detail": "Your current balance is 30, but that costs 50.",
  "instance": "/v1/account/12345",
  "balance": 30,
  "required": 50
}
```

**Why this output:** The error is thrown as a `ProblemDetails` instance, which includes the five canonical members plus two extension members (`balance` and `required`). The `Content-Type` is set to `application/problem+json`. Clients can parse the `type` URI to determine the error category, read the `detail` for human understanding, and access extension members for domain-specific context.

#### Real-World Cases

- **IETF Standards Track:** RFC 7807 is the IETF standard for API error responses.
- **Microsoft Azure API Guidelines:** Recommends RFC 7807 for error responses.
- **Google API Design Guide:** Uses a similar error model with `error.code`, `error.message`, and `error.details`.

---

## Core Concept 3: HTTP Status Codes in Practice

### Sub-Feature 3.1: Proper Use of 2xx, 3xx, 4xx, and 5xx

#### Definitions

**Core Definition:** HTTP status codes are 3-digit integers grouped into five classes that communicate the outcome of an HTTP request: 1xx (Informational), 2xx (Success), 3xx (Redirection), 4xx (Client Error), and 5xx (Server Error).

**Technical Definition:** The first digit of the status code defines the class of response. 2xx codes indicate successful processing, 3xx codes indicate that further action is needed to fulfil the request, 4xx codes indicate that the client appears to have erred, and 5xx codes indicate that the server failed to fulfil a valid request. Using the correct status code is essential for caching, retry logic, and client error handling.

**Beginner-Friendly Explanation:** Status codes are the server's way of saying what happened. 200 means "here's your data." 201 means "I created something new." 404 means "I couldn't find that." 500 means "I broke." Using the right code is like using the right word — precision matters.

#### Purposes

- To communicate the precise outcome of a request.
- To enable caching strategies (2xx responses are cacheable; 4xx/5xx are not).
- To inform retry logic (5xx errors may be retried; 4xx errors should not).
- To support client-side error handling with specific status code checks.

#### Syntax Rules and Structure

| Class | Meaning | Common Codes | When to Use |
|-------|---------|--------------|-------------|
| 2xx | Success | 200, 201, 202, 204 | Request processed successfully. |
| 3xx | Redirection | 301, 302, 304, 307 | Further action needed. |
| 4xx | Client Error | 400, 401, 403, 404, 409, 422, 429 | Client erred. |
| 5xx | Server Error | 500, 502, 503, 504 | Server failed. |

**Specific 2xx codes:**
| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 OK | Success | GET, PUT, PATCH successful with response body. |
| 201 Created | Resource created | POST successful; include `Location` header. |
| 202 Accepted | Request accepted | Async processing; may not be complete. |
| 204 No Content | No body | DELETE successful; PUT/PATCH with no body. |

**Constraints and Limitations:**
- 200 should not be used for POST creation; use 201.
- 201 should include a `Location` header pointing to the new resource.
- 202 is intentionally non-committal; the request may not have been acted upon.
- 204 must have an empty response body.
- 4xx codes should not be used for server-side failures.

#### Annotated Code Example

```javascript
// Express: Correct status code usage
const express = require('express');
const app = express();
app.use(express.json());

let users = [{ id: 1, name: 'Alice' }];

// GET → 200 OK
app.get('/v1/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ error: 'Not found' });
  res.status(200).json(user);
});

// POST → 201 Created (with Location header)
app.post('/v1/users', (req, res) => {
  const user = { id: users.length + 1, name: req.body.name };
  users.push(user);
  res
    .status(201)
    .location(`/v1/users/${user.id}`)
    .json(user);
});

// DELETE → 204 No Content
app.delete('/v1/users/:id', (req, res) => {
  users = users.filter(u => u.id !== parseInt(req.params.id));
  res.status(204).end();
});

// Async operation → 202 Accepted
app.post('/v1/reports', (req, res) => {
  const reportId = 'rpt-' + Date.now();
  // Queue async processing...
  res
    .status(202)
    .location(`/v1/reports/${reportId}/status`)
    .json({ reportId, status: 'queued' });
});

app.listen(3000);
```

**Expected Output (for `POST /v1/users` with `{"name":"Bob"}`):**
```http
HTTP/1.1 201 Created
Location: /v1/users/2

{"id":2,"name":"Bob"}
```

**Expected Output (for `DELETE /v1/users/1`):**
```http
HTTP/1.1 204 No Content
```

**Expected Output (for `POST /v1/reports`):**
```http
HTTP/1.1 202 Accepted
Location: /v1/reports/rpt-1712345678901/status

{"reportId":"rpt-1712345678901","status":"queued"}
```

**Why this output:** Each status code is chosen according to the semantics of the operation. POST creation returns 201 with a `Location` header. DELETE returns 204 with no body. The async report generation returns 202 with a status URL, indicating the request was accepted but not yet completed.

#### Real-World Cases

- **Stripe API:** Uses 201 for charge creation, 200 for retrieval, 204 for deletion.
- **GitHub API:** Uses 201 for repository creation with a `Location` header.
- **AWS API Gateway:** Uses 202 for async Lambda invocations.

---

### Sub-Feature 3.2: When to Use 201 Created, 202 Accepted, and 204 No Content

#### Definitions

**Core Definition:** 201 Created confirms that a new resource was created; 202 Accepted confirms that the request was accepted for processing but not yet completed; 204 No Content confirms that the request succeeded and there is no body to return.

**Technical Definition:** 201 Created is returned when the resource was created and the `Location` header indicates where the new resource is accessible. 202 Accepted indicates that the Data Service Request has been accepted and has not yet completed executing asynchronously. 204 No Content indicates that the server successfully processed the request and is not returning any content; there is no need for the client to move to a different location.

**Beginner-Friendly Explanation:** 201 is like a receipt for a new account — it confirms creation and tells you where to find it. 202 is like a ticket for a repair — the shop accepted your item, but it's not fixed yet. 204 is like a nod — "done, nothing more to say."

#### Purposes

- To distinguish between synchronous creation (201) and asynchronous acceptance (202).
- To avoid sending empty response bodies when the operation succeeded but has no content (204).
- To provide the client with a `Location` header for newly created resources.
- To support async processing patterns (job queues, report generation, long-running operations).

#### Syntax Rules and Structure

| Code | Body | Location Header | Use Case |
|------|------|-----------------|----------|
| 201 | Resource representation | Required | POST creates a resource synchronously. |
| 202 | Status/metadata | Recommended | Async operation queued. |
| 204 | None | Not applicable | DELETE success; PUT/PATCH with no body. |

**Constraints and Limitations:**
- 201 without a `Location` header is incomplete.
- 202 should include a way to check status (e.g., `Location` to a status endpoint).
- 204 must not include a body; clients should not expect one.
- 202 is non-committal; the server may ultimately reject the request.

#### Annotated Code Example

```javascript
// Express: 201 vs. 202 vs. 204
const express = require('express');
const app = express();
app.use(express.json());

// 201 Created — synchronous resource creation
app.post('/v1/users', (req, res) => {
  const user = { id: Date.now(), name: req.body.name };
  res.status(201).location(`/v1/users/${user.id}`).json(user);
});

// 202 Accepted — asynchronous processing
app.post('/v1/videos/:id/encode', (req, res) => {
  const jobId = 'job-' + Date.now();
  // Queue encoding job...
  res
    .status(202)
    .location(`/v1/jobs/${jobId}`)
    .json({ jobId, status: 'queued', estimatedTime: '5m' });
});

// 204 No Content — successful deletion
app.delete('/v1/users/:id', (req, res) => {
  res.status(204).end();
});

app.listen(3000);
```

**Expected Output (for `POST /v1/users`):**
```http
HTTP/1.1 201 Created
Location: /v1/users/1712345678901

{"id":1712345678901,"name":"Alice"}
```

**Expected Output (for `POST /v1/videos/42/encode`):**
```http
HTTP/1.1 202 Accepted
Location: /v1/jobs/job-1712345678901

{"jobId":"job-1712345678901","status":"queued","estimatedTime":"5m"}
```

**Expected Output (for `DELETE /v1/users/42`):**
```http
HTTP/1.1 204 No Content
```

**Why this output:** Each status code communicates a different outcome. 201 tells the client the user was created and where to find it. 202 tells the client the encoding job was queued and where to check status. 204 tells the client the deletion succeeded with nothing more to say.

#### Real-World Cases

- **GitHub API:** 201 for repository creation with `Location`; 202 for async operations like repository transfers.
- **Stripe API:** 201 for charge creation; 200 for retrieval; 204 for deletion.
- **AWS S3:** 204 for successful DELETE; 200 for successful PUT with body; 201 for bucket creation.

---

## Core Concept 4: API Documentation & Contracts

### Sub-Feature 4.1: Designing APIs Code-First vs. Design-First

#### Definitions

**Core Definition:** Design-first (contract-first) means the OpenAPI contract is authored and agreed before any implementation code exists; code-first means the API is implemented first and the OpenAPI specification is generated from the code via annotations or runtime introspection.

**Technical Definition:** Design-first means the OpenAPI contract gets authored and agreed before the code exists, and governance runs against the design. Code-first means the code gets written and the OpenAPI is generated from it afterward, usually out of annotations, and governance runs against the result. In the code-first approach, the API is first implemented in code, and then its description is created from it, using code comments, code annotations, or simply written from scratch.

**Beginner-Friendly Explanation:** Design-first is like drawing a blueprint before building a house. You agree on the layout, dimensions, and materials with everyone involved, then you build exactly to spec. Code-first is like building the house and then drawing the blueprint afterward — it might not match what was agreed, and it's harder to change once the walls are up.

#### Purposes

- To prevent drift between what the API claims to do and what it actually does.
- To enable parallel development (frontend and backend can work from the contract simultaneously).
- To generate client SDKs and server stubs from the contract.
- To enforce API governance at design time rather than after implementation.

#### Syntax Rules and Structure

| Aspect | Design-First | Code-First |
|--------|-------------|------------|
| Order | Contract → Code | Code → Contract |
| Source of truth | OpenAPI spec | Source code |
| Parallel development | Yes (frontend + backend) | Limited (backend first) |
| Client generation | Before implementation | After implementation |
| Drift risk | Spec ignored → drift | Annotations not updated → drift |
| Best for | New APIs, contracts | Existing APIs, experimentation |

**Typical design-first flow:**
1. Draft the OpenAPI spec (endpoints, schemas, auth, errors).
2. Review with backend, frontend, and QA.
3. Generate stubs or share the spec as the source of truth.
4. Implement the server to match.
5. Validate requests and responses against the contract.

**Typical code-first flow:**
1. Implement endpoints and models in code.
2. Add annotations for schemas, params, and responses.
3. Generate the OpenAPI spec from the codebase.
4. Adjust the output by tweaking annotations.
5. Use the generated spec for docs and client generation.

**Constraints and Limitations:**
- Design-first drifts when the spec is treated as a one-time design doc and stops being updated.
- Code-first drifts when the code changes but annotations don't.
- Regardless of approach, maintain a single source of truth.

#### Annotated Code Example

```yaml
# Design-first: OpenAPI 3.1 specification (openapi.yaml)
openapi: 3.1.0
info:
  title: User API
  version: 1.0.0
paths:
  /v1/users/{userId}:
    get:
      summary: Get a user by ID
      parameters:
        - name: userId
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: User found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: User not found
          content:
            application/problem+json:
              schema:
                $ref: '#/components/schemas/ProblemDetails'
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
    ProblemDetails:
      type: object
      properties:
        type:
          type: string
        title:
          type: string
        status:
          type: integer
        detail:
          type: string
```

```javascript
// Code-first: Express with JSDoc annotations for OpenAPI generation
/**
 * @openapi
 * /v1/users/{userId}:
 *   get:
 *     summary: Get a user by ID
 *     parameters:
 *       - in: path
 *         name: userId
 *         required: true
 *         schema:
 *           type: integer
 *     responses:
 *       200:
 *         description: User found
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/User'
 */
app.get('/v1/users/:userId', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.userId));
  if (!user) return res.status(404).json({ error: 'Not found' });
  res.json(user);
});
```

**Expected Output (generated OpenAPI spec from code-first annotations):**
```yaml
openapi: 3.0.0
paths:
  /v1/users/{userId}:
    get:
      summary: Get a user by ID
      parameters:
        - in: path
          name: userId
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: User found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
```

**Why this output:** The design-first approach produces the spec before the code. The code-first approach produces the spec from JSDoc annotations in the code. Both result in a valid OpenAPI document, but the design-first document is the source of truth, while the code-first document is derived from the implementation.

#### Real-World Cases

- **Design-first:** Stripe, Twilio, and most public APIs with client SDKs.
- **Code-first:** Internal microservices with rapid iteration and existing codebases.
- **Hybrid:** Design the contract, generate stubs, implement, then sync annotations for documentation.

---

### Sub-Feature 4.2: Writing APIs Using OpenAPI Specification (OAS) / Swagger

#### Definitions

**Core Definition:** The OpenAPI Specification (OAS) is a standard, language-agnostic interface description for HTTP APIs that allows both humans and computers to discover and understand the capabilities of a service without access to source code.

**Technical Definition:** The OpenAPI Specification defines a standard interface description for REST APIs. An OpenAPI document describes the available endpoints, operations, parameters, authentication methods, and response schemas. OpenAPI 3.1 aligns fully with JSON Schema 2020-12. Tools like Swagger UI and Redoc render OpenAPI documents as interactive documentation.

**Beginner-Friendly Explanation:** An OpenAPI specification is like a detailed instruction manual for your API. It tells developers exactly what endpoints exist, what parameters they accept, what they return, and what errors they might produce. Tools can read this manual and automatically generate documentation, client libraries, and server code.

#### Purposes

- To provide a single source of truth for API contracts.
- To generate interactive documentation (Swagger UI, Redoc).
- To generate client SDKs in multiple languages.
- To validate requests and responses against the contract.
- To enable API governance and security analysis.

#### Syntax Rules and Structure

```yaml
openapi: 3.1.0
info:
  title: Example API
  version: 1.0.0
  description: An example API
servers:
  - url: https://api.example.com/v1
paths:
  /users:
    get:
      summary: List users
      parameters:
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: A list of users
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
components:
  schemas:
    User:
      type: object
      required: [id, name]
      properties:
        id:
          type: integer
        name:
          type: string
```

| Section | Purpose |
|---------|---------|
| `openapi` | OAS version (3.1.0 recommended). |
| `info` | API metadata (title, version, description). |
| `servers` | Base URLs for the API. |
| `paths` | Available endpoints and operations. |
| `components` | Reusable schemas, parameters, responses. |

**Constraints and Limitations:**
- OAS 3.1 aligns with JSON Schema 2020-12; OAS 3.0 uses a modified subset.
- Large specs can be difficult to maintain; use `$ref` to split into multiple files.
- Generated documentation must be kept in sync with the spec.

#### Annotated Code Example

```javascript
// Express: Serving OpenAPI documentation with swagger-ui-express
const express = require('express');
const swaggerUi = require('swagger-ui-express');
const YAML = require('yamljs');

const app = express();

// Load the OpenAPI spec
const openapiSpec = YAML.load('./openapi.yaml');

// Serve interactive documentation at /docs
app.use('/docs', swaggerUi.serve, swaggerUi.setup(openapiSpec));

// Serve the raw spec at /openapi.json
app.get('/openapi.json', (req, res) => {
  res.json(openapiSpec);
});

// Implement the endpoints described in the spec
app.get('/v1/users', (req, res) => {
  const limit = parseInt(req.query.limit) || 20;
  res.json([{ id: 1, name: 'Alice' }].slice(0, limit));
});

app.listen(3000, () => {
  console.log('API docs at http://localhost:3000/docs');
  console.log('OpenAPI spec at http://localhost:3000/openapi.json');
});
```

**Expected Output (in browser at `/docs`):**
```
Interactive Swagger UI displaying all endpoints, parameters, and response schemas.
Developers can "Try it out" to send requests directly from the documentation.
```

**Why this output:** The OpenAPI specification is loaded and served by Swagger UI, which renders it as interactive documentation. Developers can explore endpoints, view schemas, and test requests without reading source code. The same spec can be used to generate client SDKs.

#### Real-World Cases

- **Stripe API:** Comprehensive OpenAPI specification with generated client libraries.
- **GitHub API:** OpenAPI specification driving the REST API documentation.
- **Twilio API:** OpenAPI-based documentation with interactive code samples.

---

## References

- RFC 7807 — Problem Details for HTTP APIs — https://datatracker.ietf.org/doc/html/rfc7807
- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- OpenAPI Specification 3.1 — https://spec.openapis.org/oas/v3.1.0
- Microsoft REST API Guidelines — https://github.com/microsoft/api-guidelines
- Best Practices for Naming REST API Endpoints — DreamFactory — https://blog.dreamfactory.com/best-practices-for-naming-rest-api-endpoints
- REST API Naming Conventions: A Practical Style Guide — Apidog — https://apidog.com/blog/rest-api-naming-conventions/
- API Design and Naming Conventions — Jobrad Connect — https://connect.jobrad.org/docs/getting-started/api-guidelines/api-design-and-conventions/
- RESTful API Design — OCTO Quick Reference Card — http://blog.octo.com/wp-content/uploads/2014/10/RESTful-API-design-OCTO-Quick-Reference-Card-2.2.pdf
- OpenAPI-first vs code-first API development — AppMaster — https://appmaster.io/blog/openapi-first-vs-code-first-api-development
- Standardize API Responses (Envelope & Error Handling) — GitHub Issue — https://github.com/Amr-Mohie-eldeen/taste-kid/issues/14
- RESTful API设计规范与版本管理最佳实践 — ZPEDU — https://www.zpedu.com/it/rjyf/40902.html
- RFC 7807 Problem Details — Apache Juneau — https://juneau.staged.apache.org/pages/doc/rest/RFC7807.html
- 576 <https://api.unece.org/transportMovements> — UNECE OpenAPI Naming and Design Rules — https://unece.org/sites/default/files/2023-07/API-TECH-SPEC_OpenAPI_NDR_version1p0.pdf
- HTTP Status Codes — MDN Web Docs — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status
- 201 Created — MDN Web Docs — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/201
- 202 Accepted — MDN Web Docs — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/202
- 204 No Content — MDN Web Docs — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/204
- OpenAPI Best Practices — Learn OpenAPI — https://learn.openapis.org/best-practices.html
- JSend Specification — https://github.com/omniti-labs/jsend
- JSON:API Specification — https://jsonapi.org/