# Express.js Response Design — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Response design is the practice of defining consistent, predictable structures for the data your API returns to clients — including success payloads, error responses, metadata, and pagination information — so that consumers can reliably parse and handle every response without special-case logic.

**Technical Definition:** Response design encompasses the conventions, schemas, and architectural patterns used to shape HTTP response bodies. It includes the choice of envelope vs. bare payload, the representation of resources (including hypermedia links), the structure of error objects, and the inclusion of metadata such as pagination cursors and timestamps. Standards such as JSON:API, RFC 9457 (Problem Details), and JSend provide formal specifications for these structures.

**Beginner-Friendly Explanation:** Imagine ordering from a restaurant where every dish arrives on a differently shaped plate — sometimes a bowl, sometimes a plate, sometimes just thrown on the table. You'd never know what to expect. Response design is about using the same "plate" every time: a predictable container so that clients always know where to find the data, the errors, and the extra information. When every response looks the same, client code becomes simpler, bugs decrease, and documentation shrinks.

### Key Characteristics

- **Consistency:** Every endpoint returns responses in the same structural format.
- **Predictability:** Clients can parse responses without endpoint-specific logic.
- **Extensibility:** New metadata fields can be added without breaking existing clients.
- **Self-describing:** Responses include enough context (type, links, metadata) for clients to navigate the API.
- **Standard-aligned:** Designs draw from JSON:API, RFC 9457, JSend, and industry guidelines.
- **Error-first clarity:** Error responses are as carefully designed as success responses.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript knowledge:** Objects, arrays, JSON serialisation.
- **Understanding of HTTP:** Status codes, headers, and content types.
- **Familiarity with REST principles:** Resources, methods, and statelessness.

### Related Programming Areas

- **API versioning:** Response structure changes are a primary driver of versioning.
- **Error handling middleware:** Centralised error formatting depends on response design.
- **Pagination:** Metadata structure is part of response design.
- **HATEOAS / Hypermedia:** Links embedded in responses enable API discoverability.
- **API documentation:** OpenAPI/Swagger schemas describe the response structures.
- **Client SDK generation:** Consistent responses enable automated client code generation.

### Core Concepts

1. **Consistent JSON Structures** — the envelope pattern and its variants.
2. **Resource Representation** — how individual resources are shaped, including hypermedia links.
3. **Metadata** — timestamps, request IDs, and contextual information.
4. **Pagination Metadata** — offset-based and cursor-based pagination structures.
5. **Error Responses** — RFC 9457 Problem Details and structured error objects.
6. **API Response Conventions** — JSend, Microsoft guidelines, and industry patterns.

---

## Core Concept 1: Consistent JSON Structures

### Definitions

**Core Definition:** A consistent JSON structure is a uniform envelope or shape applied to every API response, ensuring that clients always know where to find the payload, errors, and metadata regardless of the endpoint.

**Technical Definition:** The response envelope pattern wraps the primary payload in a top-level object, typically with keys such as `data`, `meta`, and `error`. This is distinct from returning bare arrays or objects at the top level, which makes it impossible to add metadata without breaking clients. The JSend specification formalises this with a `status` field (`success`, `fail`, or `error`) and a `data` payload. Industry patterns commonly use `{ success, data, meta }` for success and `{ success, error, meta }` for failures.

**Beginner-Friendly Explanation:** Think of a response envelope as a shipping box with labelled compartments. One compartment holds the actual product (your data), another holds the packing slip (metadata), and a third holds any damage report (errors). Because every box has the same compartments, the recipient always knows where to look — no guessing required.

### Purposes

- To provide a predictable structure that clients can parse uniformly across all endpoints.
- To create a clear separation between payload data and response metadata.
- To enable adding new metadata fields without breaking existing clients.
- To standardise error handling so that clients can rely on a consistent error shape.
- To simplify client-side code by eliminating endpoint-specific response parsing logic.

### Syntax Rules and Structure

#### General Envelope Pattern

```json
{
  "success": true,
  "statusCode": 200,
  "message": "Users retrieved successfully",
  "data": [...],
  "meta": {
    "timestamp": "2026-01-15T10:30:00.000Z",
    "requestId": "req-abc-123"
  }
}
```

| Component | Breakdown |
|-----------|-----------|
| `success` | Boolean indicating whether the request succeeded. |
| `statusCode` | The HTTP status code (mirrors the HTTP response status). |
| `message` | Human-readable summary of the outcome. |
| `data` | The primary payload (object, array, or null). |
| `meta` | Contextual information (timestamp, pagination, etc.). |

#### JSend Specification

The JSend specification defines three response types: [24†L2-L6]

| Type | Required Keys | When to Use |
|------|--------------|-------------|
| `success` | `status`, `data` | All went well and data was returned. |
| `fail` | `status`, `data` | Problem with submitted data or a pre-condition. |
| `error` | `status`, `message` (optional: `code`, `data`) | An exception was thrown during processing. |

#### Syntax Rules

- The envelope should be applied to **every** response, including errors.
- Business data belongs inside `data`, never at the envelope root. [3†L35-L39]
- Metadata keys should be reserved and documented; do not repurpose them.
- The `status` field in JSend uses lowercase values: `"success"`, `"fail"`, `"error"`.
- Use `null` for `data` when there is no payload (e.g., after a DELETE).

#### Constraints and Limitations

- Envelopes add a small overhead to every response (bytes and parsing time).
- Not all clients expect envelopes; document the structure clearly.
- Mixing enveloped and non-enveloped responses in the same API causes confusion.

### Annotated Code Example

```js
// response-envelope.js
const express = require('express');
const app = express();

// Helper functions for consistent responses
function successResponse(res, data, meta = {}) {
  return res.status(200).json({
    success: true,
    statusCode: 200,
    message: 'Request successful',
    data,
    meta: {
      timestamp: new Date().toISOString(),
      ...meta
    }
  });
}

function errorResponse(res, statusCode, message, code, details = null) {
  return res.status(statusCode).json({
    success: false,
    statusCode,
    message,
    error: {
      code,
      details
    },
    meta: {
      timestamp: new Date().toISOString()
    }
  });
}

// Routes using the envelope
app.get('/api/users', (req, res) => {
  const users = [
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' }
  ];
  successResponse(res, users, { total: users.length });
});

app.get('/api/users/:id', (req, res) => {
  const user = { id: parseInt(req.params.id), name: 'Alice' };
  if (!user.id) {
    return errorResponse(res, 404, 'User not found', 'USER_NOT_FOUND');
  }
  successResponse(res, user);
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output (for `GET /api/users`):**
```json
{
  "success": true,
  "statusCode": 200,
  "message": "Request successful",
  "data": [
    { "id": 1, "name": "Alice" },
    { "id": 2, "name": "Bob" }
  ],
  "meta": {
    "timestamp": "2026-01-15T10:30:00.000Z",
    "total": 2
  }
}
```

**Expected Output (for `GET /api/users/999`):**
```json
{
  "success": false,
  "statusCode": 404,
  "message": "User not found",
  "error": {
    "code": "USER_NOT_FOUND",
    "details": null
  },
  "meta": {
    "timestamp": "2026-01-15T10:30:00.000Z"
  }
}
```

**Why this output:** The helper functions ensure that every response — success or error — uses the same envelope structure. Clients can check `response.success` to determine the outcome, then access `response.data` or `response.error` accordingly. The `meta.timestamp` provides a consistent record of when the response was generated.

### Real-World Cases

- **Stripe API:** Returns consistent envelopes with `data`, `has_more`, and `url` for list endpoints.
- **Slack API:** Uses `ok` boolean and `response_metadata` for cursor pagination. [32†L21-L25]
- **JSend-compliant APIs:** Use `status`, `data`, and `message` for all responses. [24†L26-L28]

---

## Core Concept 2: Resource Representation

### Definitions

**Core Definition:** Resource representation is the JSON structure used to describe a single resource or a collection of resources, including the resource's attributes, relationships, and hypermedia links.

**Technical Definition:** In JSON:API, a resource object must contain at minimum `type` and `id`, with optional `attributes` and `relationships` members. [0†L10-L12] The `attributes` member holds the resource's data, while `relationships` holds references to other resources. For hypermedia-driven APIs, each resource representation may include a `links` array or `_links` object containing navigational affordances (HATEOAS). The Microsoft REST API Guidelines recommend including `links` with `rel`, `href`, `action`, and `types` fields. [31†L20-L30]

**Beginner-Friendly Explanation:** A resource representation is like a detailed product label. It tells you what the product is (type and ID), what its properties are (attributes), what it's connected to (relationships), and how to find related items (links). Instead of a bare object with no context, a proper representation gives clients everything they need to understand and navigate the resource.

### Purposes

- To provide a self-describing structure for individual resources.
- To explicitly declare the resource type and unique identifier.
- To separate resource attributes from relationship references.
- To enable API discoverability through hypermedia links (HATEOAS).
- To support sparse fieldsets where clients request only specific attributes.

### Syntax Rules and Structure

#### JSON:API Resource Object

```json
{
  "type": "articles",
  "id": "1",
  "attributes": {
    "title": "JSON:API paints my bikeshed!",
    "body": "The shortest article. Ever.",
    "created": "2015-05-22T14:56:29.000Z",
    "updated": "2015-05-22T14:56:28.000Z"
  },
  "relationships": {
    "author": {
      "data": { "id": "42", "type": "people" }
    }
  }
}
```

| Component | Breakdown |
|-----------|-----------|
| `type` | Required. The resource type (plural, lowercase). |
| `id` | Required. The unique identifier. |
| `attributes` | Optional. The resource's data fields. |
| `relationships` | Optional. References to related resources. |
| `links` | Optional. Hypermedia links for navigation. |

#### HATEOAS Links Structure

```json
{
  "orderID": 3,
  "productID": 2,
  "quantity": 4,
  "links": [
    {
      "rel": "customer",
      "href": "https://api.contoso.com/customers/3",
      "action": "GET",
      "types": ["application/json"]
    },
    {
      "rel": "self",
      "href": "https://api.contoso.com/orders/3",
      "action": "PUT",
      "types": ["application/x-www-form-urlencoded"]
    }
  ]
}
```

| Component | Breakdown |
|-----------|-----------|
| `rel` | The relationship type (`self`, `customer`, `next`, `prev`). |
| `href` | The URL of the related resource. |
| `action` | The HTTP method to use. |
| `types` | Supported MIME types for the operation. |

#### Syntax Rules

- The `type` member is **required** in every JSON:API resource object. [0†L10-L12]
- A relationship object must contain a `data` member. [0†L13-L15]
- HATEOAS links should include `rel` and `href`; `action` and `types` are recommended.
- The `self` link points to the resource itself; other links enable state transitions.
- Client applications should be able to navigate the entire API from the root response.

#### Constraints and Limitations

- JSON:API requires the `application/vnd.api+json` content type.
- HATEOAS adds payload size; for high-volume APIs, consider making links optional.
- Not all clients use hypermedia links; many rely on hardcoded URLs.
- The Microsoft guidelines note there is no single standard for modelling HATEOAS. [13†L15-L16]

### Annotated Code Example

```js
// resource-representation.js
const express = require('express');
const app = express();

// JSON:API-style resource representation
app.get('/api/articles/:id', (req, res) => {
  const article = {
    type: 'articles',
    id: req.params.id,
    attributes: {
      title: 'Response Design Patterns',
      body: 'A comprehensive guide...',
      createdAt: '2026-01-15T10:00:00Z',
      updatedAt: '2026-01-15T12:00:00Z'
    },
    relationships: {
      author: { data: { id: '42', type: 'people' } }
    },
    links: {
      self: `/api/articles/${req.params.id}`,
      author: `/api/people/42`
    }
  };

  res.json({ data: article });
});

// HATEOAS-style order with operation links
app.get('/api/orders/:id', (req, res) => {
  const order = {
    orderID: parseInt(req.params.id),
    productID: 2,
    quantity: 4,
    orderValue: 16.60,
    links: [
      {
        rel: 'customer',
        href: `/api/customers/3`,
        action: 'GET',
        types: ['application/json']
      },
      {
        rel: 'self',
        href: `/api/orders/${req.params.id}`,
        action: 'PUT',
        types: ['application/json']
      },
      {
        rel: 'self',
        href: `/api/orders/${req.params.id}`,
        action: 'DELETE',
        types: []
      }
    ]
  };

  res.json(order);
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/articles/1`):**
```json
{
  "data": {
    "type": "articles",
    "id": "1",
    "attributes": {
      "title": "Response Design Patterns",
      "body": "A comprehensive guide...",
      "createdAt": "2026-01-15T10:00:00Z",
      "updatedAt": "2026-01-15T12:00:00Z"
    },
    "relationships": {
      "author": { "data": { "id": "42", "type": "people" } }
    },
    "links": {
      "self": "/api/articles/1",
      "author": "/api/people/42"
    }
  }
}
```

**Why this output:** The resource object follows the JSON:API structure with `type`, `id`, `attributes`, `relationships`, and `links`. Clients can identify the resource type, extract its properties, navigate to the author via the relationship, and discover the self-link for further operations.

### Real-World Cases

- **JSON:API implementations:** Ember Data, JSON:API client libraries.
- **HAL APIs:** Spring HATEOAS, Laravel HATEOAS.
- **Microsoft Graph:** Uses `@odata.context` and `value` for resource representations.
- **GitHub API:** Includes `url`, `html_url`, and `links` for navigability.

---

## Core Concept 3: Metadata

### Definitions

**Core Definition:** Metadata is contextual information included in a response that describes the response itself rather than the primary data — such as timestamps, request identifiers, pagination cursors, and total counts.

**Technical Definition:** Metadata is typically placed in a `meta` object at the envelope level or alongside the `data` member. Common metadata fields include `timestamp` (when the response was generated), `requestId` (for tracing), `total` (total matching records), and `warnings` (non-fatal issues). In JSON:API, the `meta` member is a top-level member that can contain any non-standard information. [8†L39-L41] The Microsoft guidelines recommend including metadata for pagination and tracing.

**Beginner-Friendly Explanation:** Metadata is like the information printed on a receipt — the date, the order number, and the store location. It's not the product you bought (that's the data), but it tells you when and where the transaction happened. In API responses, metadata helps with debugging, pagination, and tracing requests across services.

### Purposes

- To provide contextual information about the response (when, where, and how).
- To enable request tracing across distributed systems via request IDs.
- To communicate non-fatal issues via warnings.
- To support pagination with cursor and count information.
- To include rate-limit and quota information.

### Syntax Rules and Structure

```json
{
  "data": [...],
  "meta": {
    "timestamp": "2026-01-15T10:30:00.000Z",
    "requestId": "req-abc-123",
    "total": 145,
    "warnings": [
      { "code": "PARTIAL_RESULTS", "message": "Some records were excluded" }
    ]
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `timestamp` | ISO 8601 string | When the response was generated. |
| `requestId` | String | Unique identifier for tracing the request. |
| `total` | Integer | Total matching records (for pagination). |
| `warnings` | Array | Non-fatal issues the client should be aware of. |

#### Syntax Rules

- Metadata keys should be reserved and documented in the API specification.
- `timestamp` should use ISO 8601 format with timezone (UTC recommended).
- `requestId` should be propagated from incoming requests or generated server-side.
- Metadata must not contain business data; that belongs in `data`.
- In JSON:API, `meta` is a top-level member that can contain arbitrary information. [8†L39-L41]

#### Constraints and Limitations

- Metadata increases payload size; include only what clients need.
- Metadata keys must not conflict with `data` or `error` keys.
- Not all clients use metadata; document its purpose clearly.

### Annotated Code Example

```js
// response-metadata.js
const express = require('express');
const crypto = require('crypto');
const app = express();

// Middleware to attach request metadata
app.use((req, res, next) => {
  req.requestId = req.get('X-Request-Id') || crypto.randomUUID();
  res.set('X-Request-Id', req.requestId);
  next();
});

app.get('/api/products', (req, res) => {
  const products = [{ id: 1, name: 'Laptop' }];

  res.json({
    data: products,
    meta: {
      timestamp: new Date().toISOString(),
      requestId: req.requestId,
      total: products.length,
      version: 'v1',
      warnings: []
    }
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/products`):**
```json
{
  "data": [{ "id": 1, "name": "Laptop" }],
  "meta": {
    "timestamp": "2026-01-15T10:30:00.000Z",
    "requestId": "550e8400-e29b-41d4-a716-446655440000",
    "total": 1,
    "version": "v1",
    "warnings": []
  }
}
```

**Why this output:** The middleware generates or propagates a `requestId` and sets it in both the response header and the `meta` object. Clients can use this ID when reporting issues. The `timestamp` records when the response was generated, and `total` indicates the total number of matching records.

### Real-World Cases

- **Distributed tracing:** `requestId` propagated across microservices for debugging.
- **Rate limiting:** `meta.rateLimit` with remaining requests and reset time.
- **Data freshness:** `meta.timestamp` to indicate cache age.
- **Partial results:** `meta.warnings` when some records were excluded due to permissions.

---

## Core Concept 4: Pagination Metadata

### Definitions

**Core Definition:** Pagination metadata is the set of fields included in a response that describes how a paginated collection is structured — including the current position, total count, and links or cursors for navigating to other pages.

**Technical Definition:** Pagination metadata can follow offset-based or cursor-based patterns. Offset-based pagination uses `page`, `limit`, `total`, and `totalPages` fields, enabling clients to jump to arbitrary pages. Cursor-based pagination uses an opaque `nextCursor` (or `next_cursor`) token that clients pass back to retrieve the next page, avoiding the performance degradation of deep offsets. [2†L20-L24] The Slack API uses a `response_metadata` object with a `next_cursor` field; an empty or null `next_cursor` indicates no further results. [32†L21-L25]

**Beginner-Friendly Explanation:** Pagination metadata is like the "page 3 of 20" indicator on a book reader, plus a bookmark for where you left off. For offset-based pagination, you get the page number and total pages (like "page 3 of 20"). For cursor-based pagination, you get a bookmark (the cursor) that you hand back to get the next page — more efficient for large, changing datasets.

### Purposes

- To inform clients about the total size of a collection and their current position.
- To provide navigation links (next, previous, first, last) for discoverability.
- To enable efficient cursor-based pagination for large, real-time datasets.
- To prevent clients from having to make extra requests to discover if more data exists.
- To allow clients to render pagination UI controls (page numbers, "load more" buttons).

### Sub-Feature 4.1: Offset-Based Pagination Metadata

#### Syntax Rules and Structure

```json
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 145,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": true,
    "next": "/api/users?page=3&limit=20",
    "prev": "/api/users?page=1&limit=20"
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `page` | Integer | Current page number (1-indexed). |
| `limit` | Integer | Items per page. |
| `total` | Integer | Total matching records. |
| `totalPages` | Integer | Total number of pages. |
| `hasNext` | Boolean | Whether a next page exists. |
| `hasPrev` | Boolean | Whether a previous page exists. |
| `next` / `prev` | String | URL for the next/previous page. |

**Rules:**
- Use consistent parameter names: `limit`, `offset`, `page`, `pageSize`. [10†L25-L27]
- Always set sensible defaults (e.g., `limit=25`) and enforce a maximum. [10†L32-L35]
- Include `next` and `prev` links for discoverability (HATEOAS). [10†L29-L31]
- The `total` count should be omitted or marked as approximate for very large datasets. [15†L33-L38]

---

### Sub-Feature 4.2: Cursor-Based Pagination Metadata

#### Syntax Rules and Structure

```json
{
  "data": [...],
  "meta": {
    "nextCursor": "eyJpZCI6MTIzfQ==",
    "hasNext": true,
    "limit": 20
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `nextCursor` | String or null | Opaque cursor for the next page. |
| `hasNext` | Boolean | Whether more results exist. |
| `limit` | Integer | Items per page. |

**Rules:**
- The cursor must be opaque; clients should not parse or construct it. [22†L39-L40]
- A null or absent `nextCursor` indicates the last page. [27†L25-L26]
- The cursor should encode the sort key and direction.
- Cursor-based pagination is recommended for large, frequently changing datasets. [10†L16-L18]

### Annotated Code Example

```js
// pagination-metadata.js
const express = require('express');
const app = express();

// Simulated dataset
const items = Array.from({ length: 100 }, (_, i) => ({
  id: i + 1,
  name: `Item ${i + 1}`
}));

// Offset-based pagination
app.get('/api/items', (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = Math.min(parseInt(req.query.limit) || 20, 100);
  const offset = (page - 1) * limit;

  const paginatedItems = items.slice(offset, offset + limit);
  const total = items.length;
  const totalPages = Math.ceil(total / limit);

  res.json({
    data: paginatedItems,
    pagination: {
      page,
      limit,
      total,
      totalPages,
      hasNext: page < totalPages,
      hasPrev: page > 1,
      next: page < totalPages
        ? `/api/items?page=${page + 1}&limit=${limit}`
        : null,
      prev: page > 1
        ? `/api/items?page=${page - 1}&limit=${limit}`
        : null
    }
  });
});

// Cursor-based pagination
app.get('/api/items/cursor', (req, res) => {
  const limit = parseInt(req.query.limit) || 20;
  const cursor = req.query.cursor
    ? parseInt(Buffer.from(req.query.cursor, 'base64').toString())
    : 0;

  const filtered = items.filter(item => item.id > cursor);
  const paginated = filtered.slice(0, limit);
  const nextCursor = paginated.length === limit
    ? Buffer.from(String(paginated[paginated.length - 1].id)).toString('base64')
    : null;

  res.json({
    data: paginated,
    meta: {
      nextCursor,
      hasNext: nextCursor !== null,
      limit
    }
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/items?page=2&limit=5`):**
```json
{
  "data": [
    { "id": 6, "name": "Item 6" },
    { "id": 7, "name": "Item 7" },
    { "id": 8, "name": "Item 8" },
    { "id": 9, "name": "Item 9" },
    { "id": 10, "name": "Item 10" }
  ],
  "pagination": {
    "page": 2,
    "limit": 5,
    "total": 100,
    "totalPages": 20,
    "hasNext": true,
    "hasPrev": true,
    "next": "/api/items?page=3&limit=5",
    "prev": "/api/items?page=1&limit=5"
  }
}
```

**Expected Output (for `GET /api/items/cursor?limit=3`):**
```json
{
  "data": [
    { "id": 1, "name": "Item 1" },
    { "id": 2, "name": "Item 2" },
    { "id": 3, "name": "Item 3" }
  ],
  "meta": {
    "nextCursor": "Mw==",
    "hasNext": true,
    "limit": 3
  }
}
```

**Why this output:** The offset-based response includes full pagination metadata with total counts and navigation links. The cursor-based response includes an opaque `nextCursor` (base64-encoded ID 3) that the client passes back as `?cursor=Mw==` to retrieve the next page. Cursor-based pagination avoids the performance cost of deep offsets.

### Real-World Cases

- **Slack:** `response_metadata.next_cursor` for channel and user lists. [32†L21-L25]
- **Stripe:** `has_more` and `starting_after` cursor for list endpoints.
- **GitHub:** Link headers with `rel="next"`, `rel="last"` for paginated resources.
- **E-commerce product listings:** Offset-based pagination with page numbers in the UI.

---

## Core Concept 5: Error Responses

### Definitions

**Core Definition:** An error response is a structured JSON body returned when a request fails, containing machine-readable codes and human-readable messages that help clients understand and handle the failure.

**Technical Definition:** RFC 9457 (Problem Details for HTTP APIs) defines a standard error format with fields `type`, `title`, `status`, `detail`, and `instance`, using the content type `application/problem+json`. [1†L37-L41] The `type` field is a URI that identifies the problem type; `title` is a human-readable summary; `status` mirrors the HTTP status code; `detail` provides specific information; and `instance` identifies the specific occurrence. Additional extension fields can be added for domain-specific context.

**Beginner-Friendly Explanation:** An error response is a well-structured "something went wrong" message. Instead of a cryptic stack trace or a generic "Error 500," it tells the client exactly what type of problem occurred (via `type`), a human-readable explanation (via `title` and `detail`), and the HTTP status (via `status`). This lets client applications handle errors intelligently — for example, showing "Your email is already registered" instead of "An error occurred."

### Purposes

- To provide a machine-readable description of the problem that clients can parse programmatically.
- To include a human-readable message for display to end users.
- To identify the problem type via a URI for documentation and resolution guidance.
- To include extension fields with domain-specific context (e.g., validation errors).
- To standardise error handling across all endpoints.

### Syntax Rules and Structure

#### RFC 9457 Problem Details

```json
{
  "type": "https://example.com/probs/out-of-credit",
  "status": 403,
  "title": "You do not have enough credit.",
  "detail": "Your current balance is 30, but that costs 50.",
  "instance": "/account/12345/msgs/abc",
  "balance": 30,
  "accounts": ["/account/12345", "/account/67890"]
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Recommended | URI identifying the problem type. |
| `title` | Recommended | Human-readable summary. |
| `status` | Recommended | HTTP status code. |
| `detail` | Recommended | Specific explanation for this occurrence. |
| `instance` | Optional | URI identifying the specific occurrence. |
| Extensions | Optional | Domain-specific fields. |

#### Content Type

Error responses using RFC 9457 must use `Content-Type: application/problem+json`. [1†L35-L36]

#### Syntax Rules

- The `type` field should be a URI that, when dereferenced, provides human-readable documentation.
- The `status` field must mirror the HTTP status code.
- Extension fields (e.g., `balance`, `accounts`) provide domain-specific context. [14†L20-L26]
- For validation errors, include an `errors` array with `field` and `message` for each failure. [0†L37-L40]
- The media type should be `application/problem+json`. [1†L38-L39]

#### Constraints and Limitations

- RFC 9457 is extensible; define and document your extension fields.
- Do not expose internal stack traces or database errors in production.
- The `type` URI should be stable and documented.

### Annotated Code Example

```js
// error-responses.js
const express = require('express');
const app = express();
app.use(express.json());

// RFC 9457 Problem Details helper
function problemDetails(res, {
  type = 'about:blank',
  title,
  status,
  detail,
  instance,
  ...extensions
}) {
  res.status(status)
     .set('Content-Type', 'application/problem+json')
     .json({
       type,
       title,
       status,
       ...(detail && { detail }),
       ...(instance && { instance }),
       ...extensions
     });
}

// Validation error endpoint
app.post('/api/users', (req, res) => {
  const errors = [];
  if (!req.body.email) {
    errors.push({ field: 'email', message: 'Email is required' });
  }
  if (!req.body.password || req.body.password.length < 8) {
    errors.push({ field: 'password', message: 'Password must be at least 8 characters' });
  }

  if (errors.length > 0) {
    return problemDetails(res, {
      type: 'https://api.example.com/problems/validation-error',
      title: 'Validation Error',
      status: 422,
      detail: 'The request body failed validation',
      errors
    });
  }

  res.status(201).json({ id: 1, email: req.body.email });
});

// Resource not found
app.get('/api/users/:id', (req, res) => {
  problemDetails(res, {
    type: 'https://api.example.com/problems/not-found',
    title: 'User Not Found',
    status: 404,
    detail: `User with ID ${req.params.id} does not exist`,
    instance: `/api/users/${req.params.id}`
  });
});

// Global error handler
app.use((err, req, res, next) => {
  console.error(err);
  problemDetails(res, {
    type: 'about:blank',
    title: 'Internal Server Error',
    status: 500,
    detail: 'An unexpected error occurred'
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `POST /api/users` with missing email and short password):**
```json
{
  "type": "https://api.example.com/problems/validation-error",
  "title": "Validation Error",
  "status": 422,
  "detail": "The request body failed validation",
  "errors": [
    { "field": "email", "message": "Email is required" },
    { "field": "password", "message": "Password must be at least 8 characters" }
  ]
}
```

**Expected Output (for `GET /api/users/999`):**
```json
{
  "type": "https://api.example.com/problems/not-found",
  "title": "User Not Found",
  "status": 404,
  "detail": "User with ID 999 does not exist",
  "instance": "/api/users/999"
}
```

**Why this output:** The `problemDetails` helper ensures every error response follows RFC 9457. The validation error includes an `errors` extension array with field-level details. The 404 response includes the `instance` field identifying the specific request. The global error handler catches unhandled exceptions and returns a generic 500 without exposing internal details.

### Real-World Cases

- **Microsoft Graph:** Uses RFC 9457-style error responses with `error.code` and `error.message`.
- **Mews Loyalty Partner API:** Follows RFC 9457 as the standard for error responses. [1†L47-L51]
- **Stripe:** Uses `error.type`, `error.code`, `error.message`, and `error.param` for validation errors.
- **RFC 9457 Problem Types Registry:** A centralised catalogue of standard problem types. [1†L28-L30]

---

## Core Concept 6: API Response Conventions

### Definitions

**Core Definition:** API response conventions are the established patterns, specifications, and industry guidelines that define how API responses should be structured, providing a shared vocabulary and expectations between API providers and consumers.

**Technical Definition:** Several formal and informal conventions exist. The **JSend** specification defines three response types (`success`, `fail`, `error`) with required and optional keys. [24†L29-L37] The **Microsoft REST API Guidelines** establish consistency fundamentals, error condition responses, and response format requirements. [25†L12-L18] **JSON:API** provides a comprehensive specification for resource representation and relationships. [0†L5-L9] **RFC 9457** standardises error responses. The **Google API Design Guide** (though not directly accessible in the search results) establishes resource-oriented design principles.

**Beginner-Friendly Explanation:** API response conventions are like the rules of the road for API designers. Just as traffic laws ensure that drivers from different countries can drive safely on the same roads, response conventions ensure that clients built by different teams can consume APIs from different providers. When everyone follows the same conventions, integration becomes predictable and efficient.

### Purposes

- To provide a shared vocabulary for API designers and consumers.
- To reduce the learning curve when integrating with a new API.
- To enable tooling (code generators, documentation, testing) to work across APIs.
- To establish a baseline for consistency and quality.
- To facilitate collaboration between frontend and backend teams.

### Syntax Rules and Structure

#### Comparison of Conventions

| Convention | Success Shape | Error Shape | Content Type |
|------------|--------------|-------------|--------------|
| **JSend** | `{ status: "success", data }` | `{ status: "error", message }` | `application/json` |
| **JSON:API** | `{ data: { type, id, attributes } }` | `{ errors: [{ status, detail }] }` | `application/vnd.api+json` |
| **RFC 9457** | N/A (success uses normal payload) | `{ type, title, status, detail }` | `application/problem+json` |
| **Microsoft** | `{ value: [...] }` | `{ error: { code, message } }` | `application/json` |
| **Custom Envelope** | `{ success: true, data, meta }` | `{ success: false, error, meta }` | `application/json` |

#### Syntax Rules

- Choose **one** convention and apply it consistently across all endpoints.
- Document the chosen convention in your API specification (OpenAPI/Swagger).
- Use the appropriate content type (`application/json`, `application/vnd.api+json`, or `application/problem+json`).
- For error responses, always include a machine-readable code and a human-readable message.
- Extension fields should be documented and versioned.

#### Constraints and Limitations

- No single convention is universally adopted; choose based on your ecosystem.
- JSON:API is the most prescriptive but requires significant upfront design.
- RFC 9457 is focused solely on errors; pair it with a success convention.
- Custom envelopes offer flexibility but require comprehensive documentation.

### Annotated Code Example

```js
// api-conventions.js
const express = require('express');
const app = express();
app.use(express.json());

// JSend-compliant responses
function jsendSuccess(res, data) {
  res.json({ status: 'success', data });
}

function jsendFail(res, data) {
  res.status(400).json({ status: 'fail', data });
}

function jsendError(res, message, code) {
  res.status(500).json({ status: 'error', message, code });
}

// Routes using JSend
app.get('/api/posts', (req, res) => {
  jsendSuccess(res, { posts: [{ id: 1, title: 'Post 1' }] });
});

app.post('/api/posts', (req, res) => {
  if (!req.body.title) {
    return jsendFail(res, { title: 'Title is required' });
  }
  jsendSuccess(res, { post: { id: 2, title: req.body.title } });
});

// Microsoft-style error response
function msError(res, status, code, message) {
  res.status(status).json({
    error: { code, message }
  });
}

app.get('/api/secure', (req, res) => {
  msError(res, 401, 'Unauthorized', 'Authentication required');
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/posts` — JSend):**
```json
{
  "status": "success",
  "data": { "posts": [{ "id": 1, "title": "Post 1" }] }
}
```

**Expected Output (for `POST /api/posts` with missing title — JSend fail):**
```json
{
  "status": "fail",
  "data": { "title": "Title is required" }
}
```

**Expected Output (for `GET /api/secure` — Microsoft-style error):**
```json
{
  "error": { "code": "Unauthorized", "message": "Authentication required" }
}
```

**Why this output:** The JSend responses follow the specification exactly: `success` with `data`, `fail` with `data` describing the validation problem. The Microsoft-style error uses the `error` object with `code` and `message`. Each convention has its own shape, but within each convention, the structure is consistent.

### Real-World Cases

- **JSend:** Used by many Node.js and PHP APIs for simplicity. [24†L2-L6]
- **JSON:API:** Used by Ember Data, Drupal, and other frameworks requiring strict resource modelling.
- **RFC 9457:** Adopted by Microsoft Graph, Mews, and other enterprise APIs.
- **Microsoft REST API Guidelines:** Used across Azure services and Microsoft Graph. [25†L12-L18]
- **Stripe:** Custom envelope with `data`, `has_more`, and `error` objects.

---

## References

- JSON:API Specification — https://jsonapi.org/
- JSON:API Examples — https://jsonapi.org/examples/
- RFC 9457 — Problem Details for HTTP APIs — https://www.rfc-editor.org/rfc/rfc9457
- JSend Specification — https://github.com/omniti-labs/jsend
- Microsoft REST API Guidelines — https://github.com/microsoft/api-guidelines
- Microsoft REST API Guidelines (vNext) — https://raw.githubusercontent.com/domufasa/api-guidelines/refs/heads/vNext/Guidelines.md
- Azure Architecture Center — Web API Design Best Practices — https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design
- Slack API Pagination — https://docs.slack.dev/apis/web-api/pagination/
- Slack Pagination — https://api.slack.com/docs/pagination
- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- IANA HTTP Status Code Registry — https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
- MDN HTTP Status Codes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status
- Express.js Response Object — https://expressjs.com/en/5x/api.html#res
- OWASP REST Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
- HAL Specification — https://stateless.group/hal_specification.html
- JSON:API Pagination — https://jsonapi.org/format/#fetching-pagination