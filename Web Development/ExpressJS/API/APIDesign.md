# API Design — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **API design** is the discipline of defining the **contract** between a software service and its consumers — specifying how resources are identified, how requests are structured, how responses are shaped, and how errors are communicated. A well-designed API is **predictable, consistent, and evolvable**, enabling clients to integrate without reading implementation code.

**Technical Definition.** API design encompasses **resource modeling** (what entities the API exposes), **URI design** (how resources are addressed), **HTTP method semantics** (what operations are legal), **representation formats** (how data is encoded), **status code usage** (how outcomes are communicated), **error models** (how failures are described), and **cross-cutting concerns** (versioning, pagination, filtering, security, rate limiting). These decisions form the API’s **interface contract**; changing them is a breaking change for consumers.

**Beginner-Friendly Explanation.** API design is **designing the menu and rules of a restaurant** for your software. The menu is the set of resources (users, orders, products). The rules are how you order them (HTTP methods), what you get back (JSON responses), and what happens when something goes wrong (status codes and errors). A good API feels **obvious** — once you learn a few endpoints, you can guess the rest. A bad API makes you read documentation for every call.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Contract-oriented** | The API is a promise to consumers; changes must be managed through versioning. |
| **Resource-centric** | URLs identify **nouns** (resources); HTTP methods express **verbs** (actions). |
| **Consistency over cleverness** | Uniform naming, casing, error shapes, and pagination patterns reduce cognitive load. |
| **Predictability** | A developer should infer new endpoints from patterns they have already seen. |
| **Evolvability** | Versioning, deprecation policy, and backward-compatible extensions allow the API to change without breaking clients. |
| **Observability** | Logging, metrics, request IDs, and standardized errors make failures diagnosable. |

---

### Prerequisites

- **HTTP fundamentals**: methods, status codes, headers, and the request/response model.
- **REST architectural style**: resources, representations, statelessness.
- **JSON**: the dominant representation format for modern APIs.
- **A tool to test APIs**: `curl`, Postman, or a language-specific HTTP client.
- **Familiarity with `tsconfig` / `tsc`** is not required for API design itself, but understanding type systems helps when designing schemas.

---

### Related Programming Areas

- **REST fundamentals**: API design is the practical application of REST constraints.
- **OpenAPI / Swagger**: machine-readable API specifications generated from designs.
- **GraphQL and gRPC**: alternative API paradigms with different design trade-offs.
- **API gateways and management**: rate limiting, authentication, and observability are often enforced at the gateway layer.
- **SDK and client generation**: consistent API design enables automatic client generation.

---

### Core Concepts / Features

The following seven sections cover the requested concepts. Each follows the uniform structure: definitions, purposes, syntax rules, annotated code examples with expected outputs, and real-world cases.

---

## 1. Resource Naming

### Definitions

**Core Definition.** **Resource naming** is the practice of choosing **nouns** that identify the entities an API exposes, and organizing them into **collections** and **items** using consistent conventions.

**Technical Definition.** In REST, a resource is an abstract entity identified by a URI. Resource naming governs the **path segments** that represent resource types. The conventions are: **plural nouns for collections** (`/users`), **item identifiers as path segments** (`/users/{id}`), **kebab-case for multi-word segments** (`/shipping-addresses`), **lowercase only**, and **no verbs in paths** (the HTTP method is the verb).

**Beginner-Friendly Explanation.** Resource naming is **choosing the words in your URLs**. Use **nouns** (`users`, `orders`), not verbs (`getUsers`). Use **plural** for collections (`/users`), and add an ID for a specific item (`/users/42`). Use **hyphens** for multi-word names (`/shipping-addresses`), not camelCase or underscores. And keep everything **lowercase** so URLs are not case-sensitive bugs waiting to happen.

### Purposes

- **To make URLs self-explanatory** so developers can understand what a resource is without reading docs.
- **To create predictable patterns** that developers can extend to new resources.
- **To avoid duplicating the HTTP method’s meaning** in the path, preserving cacheability and REST semantics.
- **To ensure consistency across teams and services**, reducing integration friction.

### Syntax Rules and Structure

| Rule | ✅ Correct | ❌ Incorrect |
|---|---|---|
| **Nouns, not verbs** | `/users` | `/getUsers`, `/createUser` |
| **Plural collections** | `/users`, `/orders` | `/user`, `/order` |
| **Singular only for cardinality-one singletons** | `/users/42/cart` | `/users/42/carts` |
| **kebab-case for multi-word** | `/shipping-addresses` | `/shippingAddresses`, `/shipping_addresses` |
| **Lowercase only** | `/order-items` | `/OrderItems` |
| **IDs as path segments** | `/users/123` | `/users?id=123` |
| **No deep nesting (>2 levels)** | `/users/42/orders` | `/users/42/orders/9/items/3` |

**Component breakdown.**

```
/v1/users/42/orders
```

- `/v1` — version prefix (optional but recommended).
- `/users` — **collection** of users (plural noun).
- `/42` — **item identifier** (opaque; do not assume sequential integers).
- `/orders` — **nested collection** representing orders belonging to user 42.

**Constraints and limitations.**

- **Verbs in paths** break cache invalidation and duplicate HTTP semantics.
- **Singular collection names** force awkward URLs and are inconsistent with item URLs.
- **camelCase in URLs** invites case-sensitivity bugs on different servers.
- **Deep nesting** makes URLs brittle; prefer query parameters for filters rather than deeper paths.

### Annotated Complete Step by Step Code Examples

#### Example 1: Correct vs. incorrect resource naming

```http
# ✅ Correct: plural nouns, no verbs, kebab-case.
GET    /v1/users
GET    /v1/users/123
POST   /v1/users
PATCH  /v1/users/123
DELETE /v1/users/123

# ✅ Nested relationship (max 2 levels).
GET    /v1/users/123/orders

# ✅ Singleton resource (cardinality one).
GET    /v1/users/123/cart

# ❌ Incorrect: verbs in paths.
GET    /v1/getUsers
POST   /v1/createUser

# ❌ Incorrect: singular collections.
GET    /v1/user
GET    /v1/order

# ❌ Incorrect: camelCase and underscores.
GET    /v1/shippingAddresses
GET    /v1/shipping_addresses
```

**Expected Output (conceptual):**
```
Correct URLs map to resources and methods cleanly.
Incorrect URLs duplicate the action or create case-sensitivity risks.
```

**Why this output occurs.** The HTTP method (`GET`, `POST`) already expresses the action; a verb in the path (`/getUsers`) duplicates it and creates a separate URL for the same resource, breaking caching and predictability. Plural nouns keep collection and item URLs consistent, and kebab-case avoids case and separator issues.

---

#### Example 2: Opaque identifiers

```http
# ✅ Correct: IDs are opaque and not assumed sequential.
GET /v1/users/8fk2-a9x1
GET /v1/users/78257029-1f0b-4e9a-96c6-d1b66db793d9

# ❌ Incorrect: assuming sequential integers leaks information
# and makes enumeration attacks easier.
GET /v1/users/1
GET /v1/users/2
```

**Expected Output (conceptual):**
```
Opaque identifiers prevent enumeration and decouple the API from database internals.
```

**Why this output occurs.** Sequential integer IDs make it trivial to enumerate resources and infer record counts. Opaque identifiers (UUIDs or slugs) are safer and allow the underlying storage to change without breaking URLs.

---

### Real-World Cases

- **GitHub API**: `/repos/{owner}/{repo}/issues` — plural nouns, nested relationship, opaque `owner`/`repo` identifiers.
- **Stripe API**: `/v1/customers/{customer}/payment_methods` — nested collection with opaque IDs.
- **Google API Design Guide**: recommends plural nouns for collections and standard methods (`get`, `list`, `create`, `update`, `delete`).

---

## 2. URL Structures

### Definitions

**Core Definition.** A **URL structure** is the **pattern** by which an API’s endpoints are organized: the version prefix, the resource path, nested relationships, and query parameters for filtering and pagination.

**Technical Definition.** A REST API URL typically follows the pattern `/{version}/{resource-type}/{resource-id}/{sub-resource}?{query-parameters}`. The **version** is the first segment after the base path (e.g., `/v1`). **Path segments** alternate between resource types (plural nouns) and resource identifiers. **Query parameters** carry optional modifiers: filtering, sorting, pagination, and field selection. URLs **should not** end with a trailing slash; the server should redirect to the canonical URI.

**Beginner-Friendly Explanation.** URL structure is **the shape of your API’s addresses**. It starts with a **version** (`/v1`), then a **resource** (`/users`), then an **ID** (`/42`), then maybe a **sub-resource** (`/orders`). Anything optional — filtering, sorting, paging — goes in the **query string** (`?limit=20&offset=0`). A consistent structure means developers can guess how to reach any resource after learning one or two examples.

### Purposes

- **To provide a stable, predictable address space** that clients can navigate and construct.
- **To separate resource identity (path)** from optional modifiers (query parameters).
- **To enable versioning** without changing the resource model.
- **To support relationships** through shallow nesting while avoiding brittle deep hierarchies.

### Syntax Rules and Structure

**General URL pattern.**

```
https://api.example.com/{version}/{resource}/{id}/{sub-resource}?{query}
```

**Component breakdown.**

```
https://api.example.com/v1/users/42/orders?limit=20&offset=0
```

- `https://api.example.com` — base URL (HTTPS enforced).
- `/v1` — **major version prefix** (lowercase `v` + number).
- `/users` — **resource type** (plural noun).
- `/42` — **resource identifier** (path parameter).
- `/orders` — **nested resource type** (relationship).
- `?limit=20&offset=0` — **query parameters** for pagination.

**URL structure rules.**

| Rule | Guidance |
|---|---|
| **Version first** | Place the major version immediately after the base path: `/v1/…`. |
| **No trailing slash** | Paths should not end with `/`; redirect trailing-slash requests to the canonical URI. |
| **Shallow nesting** | Limit nesting to collection → item → collection (2 levels); avoid deeper. |
| **Path parameters for identity** | Use path segments for resource identifiers only; not for filters or options. |
| **Query parameters for modifiers** | Use query strings for filtering, sorting, pagination, and field selection. |
| **Consistent casing** | Lowercase paths; camelCase or snake_case for JSON fields, chosen once and documented. |

**Constraints and limitations.**

- **Deep nesting** creates brittle URLs; flatten where possible or use query filters instead.
- **Path parameters used for filtering** violate REST conventions and make caching unpredictable.
- **No version** in the URL is a common mistake; versioning from day one avoids breaking changes.

### Annotated Complete Step by Step Code Examples

#### Example 1: Correct URL structure for a collection

```http
# Step 1: List users with pagination.
GET /v1/users?limit=20&offset=0 HTTP/1.1
Host: api.example.com

# Step 2: Get a specific user.
GET /v1/users/42 HTTP/1.1
Host: api.example.com

# Step 3: List orders for a user (nested relationship, 2 levels).
GET /v1/users/42/orders?status=shipped HTTP/1.1
Host: api.example.com

# Step 4: Get a specific order for a user.
GET /v1/users/42/orders/1001 HTTP/1.1
Host: api.example.com
```

**Expected Output (conceptual):**
```
Consistent structure: /v1/{resource}/{id}/{sub-resource}?{query}
Developers can construct and guess URLs without reading every endpoint's docs.
```

**Why this output occurs.** The structure is uniform: version first, resource type, identifier, nested resource, and query parameters for optional modifiers. The same pattern applies at every level, making the API predictable.

---

#### Example 2: Versioning and canonical URIs

```http
# ✅ Correct: version in the path, no trailing slash.
GET /v1/users/42 HTTP/1.1

# ❌ Incorrect: no version; future changes will break clients.
GET /users/42 HTTP/1.1

# ❌ Incorrect: trailing slash; should redirect to /v1/users/42.
GET /v1/users/42/ HTTP/1.1
# Response: 301 Moved Permanently
# Location: /v1/users/42
```

**Expected Output (conceptual):**
```
Versioned URL is stable; unversioned URL cannot evolve safely.
Trailing slash is redirected to the canonical form.
```

**Why this output occurs.** The version prefix allows a future `/v2` to coexist with `/v1` without breaking existing clients. The trailing-slash request is a non-canonical form; the server redirects to the canonical URI to maintain a single address per resource.

---

### Real-World Cases

- **IBM Cloud API Handbook**: mandates `/v{version}/{plural-resource}/{id}` and rejects malformed paths with `404`.
- **Stripe API**: `/v1/charges/{charge_id}` — versioned path, opaque IDs, shallow nesting.
- **GitHub API**: `/repos/{owner}/{repo}/issues` — two-level nesting with query parameters for filtering.

---

## 3. Request Formats

### Definitions

**Core Definition.** A **request format** is the structure of an HTTP request sent by a client to an API: the method, the URL, the headers, and the optional request body.

**Technical Definition.** A request consists of a **request line** (`METHOD /path HTTP/1.1`), **headers** (`Content-Type`, `Accept`, `Authorization`, etc.), and an optional **body** carrying a representation (usually JSON). The `Content-Type` header declares the **format of the body**; the `Accept` header declares the **formats the client can receive**. JSON is the dominant format; XML is optional for legacy or specialized systems.

**Beginner-Friendly Explanation.** A request is **everything the client sends**. It includes **what action** (the method), **what address** (the URL), **metadata** (headers like `Content-Type` and `Accept`), and **the data** (the body, usually JSON). The `Content-Type` header tells the server what format the body is in; the `Accept` header tells the server what format the client wants back.

### Purposes

- **To declare the request body’s format** so the server can parse it correctly.
- **To negotiate the response format** through the `Accept` header.
- **To carry authentication and metadata** in headers without polluting the URL.
- **To support content negotiation** between JSON, XML, and other media types.

### Syntax Rules and Structure

**General request structure.**

```
METHOD /path HTTP/1.1
Host: api.example.com
Content-Type: application/json
Accept: application/json
Authorization: Bearer <token>

{ "field": "value" }
```

**Component breakdown.**

- `METHOD` — `GET`, `POST`, `PUT`, `PATCH`, `DELETE`.
- `/path` — the resource endpoint.
- `Content-Type: application/json` — declares the **request body format**.
- `Accept: application/json` — declares the **preferred response format**.
- `Authorization: Bearer <token>` — carries the self-contained credential.
- Body — the **representation** (JSON object, array, etc.).

**Request format rules.**

| Rule | Guidance |
|---|---|
| **Use JSON for bodies** | `application/json` is the standard; support XML only when required. |
| **Declare `Content-Type`** | Servers should return `415 Unsupported Media Type` if the format is unsupported. |
| **Use `Accept` for negotiation** | Servers should return `406 Not Acceptable` if no requested format is available. |
| **Use `camelCase` or `snake_case` consistently** | Pick one JSON field casing and document it. |
| **Dates in ISO 8601** | Use `YYYY-MM-DDThh:mm:ssZ` for timestamps. |
| **Null handling** | Prefer `null` for missing values over empty strings or omission. |

**Constraints and limitations.**

- **Multipart forms** are error-prone and unnecessary for most APIs; use JSON instead.
- **Inconsistent casing** across endpoints forces clients to write transformation logic.
- **Missing `Content-Type`** on requests with bodies leads to parsing failures.

### Annotated Complete Step by Step Code Examples

#### Example 1: JSON request body with proper headers

```http
# Step 1: Create a user with a JSON body.
POST /v1/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

{
  "name": "Alice",
  "email": "alice@example.com",
  "role": "admin"
}

# Response:
# 201 Created
# Location: /v1/users/42
```

**Expected Output (conceptual):**
```
201 Created with a Location header pointing to the new resource.
The request body is parsed as JSON because Content-Type declares it.
```

**Why this output occurs.** The `Content-Type: application/json` header tells the server to parse the body as JSON. The `Accept: application/json` header tells the server the client prefers JSON responses. The server creates the resource and returns `201 Created` with a `Location` header.

---

#### Example 2: Content negotiation with `Accept`

```http
# Step 1: Request JSON (preferred).
GET /v1/users/42 HTTP/1.1
Accept: application/json

# Response:
# 200 OK
# Content-Type: application/json
# { "id": 42, "name": "Alice" }

# Step 2: Request an unsupported format.
GET /v1/users/42 HTTP/1.1
Accept: application/x-yaml

# Response:
# 406 Not Acceptable
```

**Expected Output (conceptual):**
```
200 OK with JSON when the format is supported.
406 Not Acceptable when no requested format can be produced.
```

**Why this output occurs.** The server compares the `Accept` header against the formats it can produce. JSON is supported, so the response is `200 OK` with `Content-Type: application/json`. YAML is not supported, so the server returns `406 Not Acceptable`.

---

### Real-World Cases

- **Stripe API**: accepts JSON request bodies with `Content-Type: application/json` and returns JSON.
- **GitHub API**: supports `Accept: application/vnd.github+json` for versioned JSON representations.
- **Django REST Framework**: supports JSON, XML, and browsable HTML through content negotiation.

---

## 4. Response Formats

### Definitions

**Core Definition.** A **response format** is the structure of the HTTP response returned by an API: the status code, the headers, and the optional body carrying a representation of the resource or the operation’s result.

**Technical Definition.** A response consists of a **status line** (`HTTP/1.1 200 OK`), **headers** (`Content-Type`, `Location`, `Cache-Control`, etc.), and an optional **body**. The body is the **representation** of the resource or the result of the operation. Responses should use a **consistent envelope** for collections (data plus pagination metadata) and a **consistent shape** for errors (see Error Responses). The `Content-Type` header declares the body’s format.

**Beginner-Friendly Explanation.** A response is **what the server sends back**. It includes a **status code** (did it work?), **headers** (metadata like the format), and a **body** (the actual data, usually JSON). For collections, the body should include **both the data and pagination information** so clients know how to fetch more. For errors, the body should follow a **standard shape** so clients can handle failures programmatically.

### Purposes

- **To communicate the outcome** of the request via status code and headers.
- **To return the resource representation** or the result of the operation.
- **To provide pagination metadata** for collections so clients can navigate large datasets.
- **To enable caching** through `Cache-Control` and related headers.

### Syntax Rules and Structure

**General response structure.**

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=3600

{ "data": { ... }, "meta": { ... } }
```

**Component breakdown.**

- `HTTP/1.1 200 OK` — status line.
- `Content-Type: application/json` — body format.
- `Cache-Control: max-age=3600` — caching directive.
- Body — the representation, often an envelope.

**Response format rules.**

| Rule | Guidance |
|---|---|
| **Use a consistent envelope** | For collections: `{ "data": [...], "meta": { "total": 100, "limit": 20, "offset": 0 } }`. |
| **Include pagination metadata** | Always return `total`, `limit`, `offset` (or cursor) so clients can page. |
| **Use ISO 8601 dates** | Timestamps as `YYYY-MM-DDThh:mm:ssZ`. |
| **Use `null` for missing values** | Prefer `null` over empty strings or omitted fields. |
| **Declare `Content-Type`** | Always include the body’s media type. |
| **Use `Location` for created resources** | `201 Created` responses should include a `Location` header. |

**Constraints and limitations.**

- **Inconsistent envelopes** force clients to write per-endpoint parsing logic.
- **Missing pagination metadata** makes large collections unusable.
- **Inconsistent date formats** cause parsing bugs across clients.

### Annotated Complete Step by Step Code Examples

#### Example 1: Collection response with pagination metadata

```http
GET /v1/users?limit=20&offset=0 HTTP/1.1
Accept: application/json

# Response:
# 200 OK
# Content-Type: application/json
# {
#   "data": [
#     { "id": 1, "name": "Alice" },
#     { "id": 2, "name": "Bob" }
#   ],
#   "meta": {
#     "total": 100,
#     "limit": 20,
#     "offset": 0,
#     "next": "/v1/users?limit=20&offset=20"
#   }
# }
```

**Expected Output (conceptual):**
```
200 OK with a data array and pagination metadata.
Clients can request the next page using the next link.
```

**Why this output occurs.** The envelope separates the **data** (the resource representations) from the **metadata** (pagination information). This allows clients to render the data and navigate pages without guessing the API’s pagination scheme.

---

#### Example 2: Created resource with `Location` header

```http
POST /v1/users HTTP/1.1
Content-Type: application/json

{ "name": "Alice", "email": "alice@example.com" }

# Response:
# 201 Created
# Location: /v1/users/42
# Content-Type: application/json
# { "id": 42, "name": "Alice", "email": "alice@example.com" }
```

**Expected Output (conceptual):**
```
201 Created with a Location header pointing to the new resource.
The body contains the created representation.
```

**Why this output occurs.** The `201 Created` status indicates success, and the `Location` header gives the canonical URI of the new resource so the client can retrieve it later. The body contains the created representation for immediate use.

---

### Real-World Cases

- **Stripe API**: returns consistent envelopes with `data`, `has_more`, and `url` for pagination.
- **GitHub API**: returns arrays for collections and `Link` headers for pagination.
- **CATAPA HRIS API**: uses a pagination envelope with `content`, `number`, `size`, `totalElements`, and `totalPages`.

---

## 5. Status Codes

### Definitions

**Core Definition.** **HTTP status codes** are three-digit numbers returned by the server to communicate the **outcome** of a request. They are grouped into five classes: `1xx` (informational), `2xx` (success), `3xx` (redirection), `4xx` (client error), and `5xx` (server error).

**Technical Definition.** Status codes are defined by HTTP specifications (RFC 9110). In REST APIs, the most important codes are `200 OK`, `201 Created`, `204 No Content`, `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`, `429 Too Many Requests`, and `500 Internal Server Error`. The status code is the **primary signal** of success or failure; the response body provides details.

**Beginner-Friendly Explanation.** Status codes are **the server’s shorthand for what happened**. `200` means “here is your data.” `201` means “I created it.” `204` means “it worked, but there is nothing to return.” `400` means “your request was malformed.” `401` means “you are not authenticated.” `403` means “you are authenticated but not allowed.” `404` means “that thing does not exist.” `500` means “I broke.” Using the right code helps clients and intermediaries (caches, proxies) behave correctly.

### Purposes

- **To communicate success or failure** unambiguously.
- **To enable correct client behavior** — retry, re-authenticate, or surface an error.
- **To support caching and intermediaries** — `2xx` responses are generally cacheable; `4xx` and `5xx` are not.
- **To distinguish client errors from server errors** for debugging and alerting.

### Syntax Rules and Structure

**Status code classes.**

| Class | Meaning | Examples |
|---|---|---|
| **2xx** | Success | `200 OK`, `201 Created`, `204 No Content` |
| **3xx** | Redirection | `301 Moved Permanently`, `303 See Other`, `304 Not Modified` |
| **4xx** | Client error | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`, `429 Too Many Requests` |
| **5xx** | Server error | `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable` |

**Status code usage rules.**

| Scenario | Status Code |
|---|---|
| Successful `GET` with body | `200 OK` |
| Successful `POST` that creates a resource | `201 Created` |
| Successful `DELETE` (no body) | `204 No Content` |
| Malformed request syntax | `400 Bad Request` |
| Missing or invalid authentication | `401 Unauthorized` |
| Authenticated but not permitted | `403 Forbidden` |
| Resource does not exist | `404 Not Found` |
| Validation failure on a well-formed request | `422 Unprocessable Entity` |
| Rate limit exceeded | `429 Too Many Requests` |
| Unhandled server error | `500 Internal Server Error` |
| Unsupported request media type | `415 Unsupported Media Type` |
| No acceptable response media type | `406 Not Acceptable` |

**Constraints and limitations.**

- **Using `200` for errors** hides failures from clients and intermediaries.
- **Using `404` for authorization failures** can leak information; prefer `403` or `404` deliberately.
- **Using `500` for client errors** misattributes blame and breaks alerting.
- **`422` vs. `400`**: use `400` for malformed syntax and `422` for semantic validation failures.

### Annotated Complete Step by Step Code Examples

#### Example 1: Correct status codes across CRUD operations

```http
# Step 1: Read → 200 OK
GET /v1/users/42
# Response: 200 OK

# Step 2: Create → 201 Created
POST /v1/users
{ "name": "Alice" }
# Response: 201 Created
# Location: /v1/users/42

# Step 3: Delete → 204 No Content
DELETE /v1/users/42
# Response: 204 No Content

# Step 4: Read a missing resource → 404 Not Found
GET /v1/users/999
# Response: 404 Not Found

# Step 5: Validation failure → 422 Unprocessable Entity
POST /v1/users
{ "email": "not-an-email" }
# Response: 422 Unprocessable Entity
```

**Expected Output (conceptual):**
```
200 → success with body
201 → created with Location
204 → success without body
404 → missing resource
422 → semantic validation failure
```

**Why this output occurs.** Each operation maps to the status code that correctly describes its outcome: `200` for successful reads, `201` for creation, `204` for deletion with no body, `404` for missing resources, and `422` for well-formed requests that fail validation.

---

#### Example 2: Distinguishing `401`, `403`, and `404`

```http
# 401: No authentication credentials provided.
GET /v1/users/me
# Response: 401 Unauthorized

# 403: Authenticated but not allowed to access this resource.
GET /v1/admin/settings
Authorization: Bearer <valid-user-token>
# Response: 403 Forbidden

# 404: Resource does not exist (or is intentionally hidden).
GET /v1/users/999
Authorization: Bearer <valid-user-token>
# Response: 404 Not Found
```

**Expected Output (conceptual):**
```
401 → missing/invalid credentials
403 → credentials valid but insufficient permissions
404 → resource not found
```

**Why this output occurs.** `401` means “identify yourself”; `403` means “I know who you are, but you cannot do this”; `404` means “the resource does not exist.” Using the correct code allows clients to respond appropriately (re-authenticate, request access, or treat the resource as absent).

---

### Real-World Cases

- **Stripe API**: uses `402 Payment Required` for card errors, `429` for rate limiting, and `400`/`404` for standard client errors.
- **GitHub API**: uses `422 Unprocessable Entity` for validation errors on well-formed requests.
- **Azure Architecture Center**: recommends `202 Accepted` for asynchronous operations and `303 See Other` for completed asynchronous resource creation.

---

## 6. Error Responses

### Definitions

**Core Definition.** An **error response** is the body returned alongside a `4xx` or `5xx` status code, describing **what went wrong** in a machine-readable and human-readable way.

**Technical Definition.** The IETF standard for API error responses is **RFC 9457 (Problem Details for HTTP APIs)**, which obsoletes RFC 7807. It defines a JSON (or XML) object with members: `type` (a URI identifying the problem type), `title` (a human-readable summary), `status` (the HTTP status code), `detail` (a human-readable explanation specific to this occurrence), and `instance` (a URI identifying the specific occurrence). Extension members may add field-level validation errors and other metadata.

**Beginner-Friendly Explanation.** An error response is **how the server explains a failure**. Instead of just returning `400` with no body, a good error response says: “Your request failed because the `email` field is invalid, and here is the specific field that caused it.” RFC 9457 defines a **standard shape** for these explanations so clients can parse them consistently across different APIs.

### Purposes

- **To provide machine-readable error details** so clients can handle failures programmatically.
- **To provide human-readable explanations** for developers debugging integrations.
- **To standardize error shapes** across endpoints and services.
- **To include field-level validation errors** for form submissions and data ingestion.

### Syntax Rules and Structure

**General syntax (RFC 9457 Problem Details).**

```json
{
  "type": "https://api.example.com/problems/out-of-credit",
  "title": "You do not have enough credit.",
  "status": 403,
  "detail": "Your current balance is 30, but that costs 50.",
  "instance": "/account/12345/msgs/abc"
}
```

**Component breakdown.**

| Member | Purpose |
|---|---|
| **`type`** | A URI identifying the problem type; defaults to `about:blank`. |
| **`status`** | The HTTP status code (must match the response status). |
| **`title`** | A short, human-readable summary of the problem type. |
| **`detail`** | A human-readable explanation specific to this occurrence. |
| **`instance`** | A URI identifying the specific occurrence. |
| **Extension members** | Additional fields, such as `errors` for field-level validation. |

**Error response rules.**

| Rule | Guidance |
|---|---|
| **Use RFC 9457** | Standardize on `application/problem+json` for error responses. |
| **Include `status`** | Match the HTTP status code in the body for clients that only read the body. |
| **Include field-level errors** | For validation failures, include an `errors` array with `field`, `code`, and `message`. |
| **Use stable `type` URIs** | Document problem types so clients can branch on them. |
| **Never leak internals** | Do not expose stack traces, SQL, or internal paths in error details. |

**Constraints and limitations.**

- **RFC 9457 is a standard, not a mandate**; many APIs use custom error shapes. Consistency matters more than strict adherence.
- **Extension members are open-ended**; document your extensions so clients know what to expect.
- **Overly detailed errors** can leak sensitive information; balance helpfulness with security.

### Annotated Complete Step by Step Code Examples

#### Example 1: RFC 9457 error response for insufficient credit

```http
POST /v1/transfers HTTP/1.1
Content-Type: application/json

{ "amount": 50, "currency": "USD" }

# Response:
# 403 Forbidden
# Content-Type: application/problem+json
# {
#   "type": "https://api.example.com/problems/out-of-credit",
#   "title": "You do not have enough credit.",
#   "status": 403,
#   "detail": "Your current balance is 30, but that costs 50.",
#   "instance": "/v1/transfers/12345"
# }
```

**Expected Output (conceptual):**
```
403 Forbidden with a problem+json body explaining the specific cause.
Clients can branch on the type URI and display the detail to the user.
```

**Why this output occurs.** The status code (`403`) tells generic HTTP software that the request was refused. The body’s `type` URI identifies the problem class, `title` summarizes it, `detail` explains the specific occurrence, and `instance` ties it to the request. This separation lets both humans and machines act on the error.

---

#### Example 2: Validation error with field-level details

```http
POST /v1/users HTTP/1.1
Content-Type: application/json

{ "name": "", "email": "not-an-email" }

# Response:
# 422 Unprocessable Entity
# Content-Type: application/problem+json
# {
#   "type": "https://api.example.com/problems/validation-error",
#   "title": "Validation failed",
#   "status": 422,
#   "detail": "One or more fields are invalid.",
#   "errors": [
#     { "field": "name", "code": "REQUIRED", "message": "Name is required." },
#     { "field": "email", "code": "INVALID_FORMAT", "message": "Email must be a valid address." }
#   ]
# }
```

**Expected Output (conceptual):**
```
422 Unprocessable Entity with an errors array listing each invalid field.
Clients can highlight the specific fields that need correction.
```

**Why this output occurs.** The `errors` extension member provides **field-level** detail, which is essential for form validation. The `type` and `title` describe the general problem class, while each error object pinpoints the field and reason. This structure is both machine-readable (stable `code` values) and human-readable (`message`).

---

### Real-World Cases

- **Stripe API**: returns a custom error object with `type`, `code`, `message`, `param`, and `decline_code` for card errors.
- **GitHub API**: returns errors with `message`, `documentation_url`, and `errors` arrays for validation failures.
- **IETF RFC 9457**: is the standard adopted by modern APIs that want interoperable error shapes.

---

## 7. API Design Best Practices

### Definitions

**Core Definition.** **API design best practices** are empirically established guidelines for building APIs that are **predictable, consistent, secure, scalable, and maintainable** across their entire lifecycle.

**Technical Definition.** Best practices span **resource modeling**, **URL and method design**, **representation formats**, **status code usage**, **error models**, **versioning**, **pagination and filtering**, **security**, **rate limiting**, and **documentation**. They bridge the gap between REST’s abstract constraints and production realities, where consistency, observability, and evolvability are paramount.

**Beginner-Friendly Explanation.** Best practices are **the habits that make an API pleasant to use**. Use nouns in URLs, correct HTTP methods, and standard status codes. Return consistent error shapes. Version from day one. Paginate large results. Enforce HTTPS and authenticate requests. Document everything. These are the things that separate an API developers enjoy from one they dread.

### Purposes

- **To make APIs predictable and easy to learn** through consistent conventions.
- **To prevent breaking changes** by versioning and documenting deprecation.
- **To protect data and users** through HTTPS, authentication, and input validation.
- **To ensure reliability** through rate limiting, pagination, and standardized errors.
- **To enable observability** through logging, metrics, and request IDs.

### Syntax Rules and Structure

**API design checklist.**

| Area | Best Practice |
|---|---|
| **Resources** | Nouns, plural collections, kebab-case, max 2 nesting levels. |
| **HTTP methods** | `GET` read, `POST` create, `PUT` replace, `PATCH` partial update, `DELETE` remove. |
| **Status codes** | `200`, `201`, `204`, `400`, `401`, `403`, `404`, `422`, `429`, `500` used correctly. |
| **Pagination** | All collection endpoints paginated; default and max page size enforced. |
| **Filtering and sorting** | Query parameters for filters, sort, search, and sparse fieldsets. |
| **Versioning** | Version from day one; URL path, header, or media type. |
| **Errors** | RFC 9457 or consistent custom shape; field-level validation errors. |
| **Authentication** | Bearer token or API key; correct `401` vs. `403` usage. |
| **Rate limiting** | Limits per endpoint/user; `429` with `Retry-After`. |
| **Documentation** | OpenAPI/Swagger spec; request/response examples; error documentation. |
| **Security** | HTTPS enforced; input validation; CORS configured; no secrets in URLs. |
| **Performance** | Pagination, caching headers, N+1 prevention. |
| **Monitoring** | Logging, error tracking, health checks, alerts. |

**Constraints and limitations.**

- **Best practices are conventions**, not enforced by the REST style; they must be applied deliberately.
- **Strict adherence to every rule** can be counterproductive for very small or internal APIs; prioritize consistency.
- **Versioning adds complexity**; choose a strategy and apply it consistently.

### Annotated Complete Step by Step Code Examples

#### Example 1: A well-designed endpoint applying multiple best practices

```http
# Step 1: Create a resource with POST, JSON, and authentication.
POST /v1/users HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer <token>

{ "name": "Alice", "email": "alice@example.com" }

# Response:
# 201 Created
# Location: /v1/users/42
# { "id": 42, "name": "Alice", "email": "alice@example.com" }

# Step 2: List with pagination, filtering, and sorting.
GET /v1/users?limit=20&offset=0&role=admin&sort=name HTTP/1.1
Accept: application/json
Authorization: Bearer <token>

# Response:
# 200 OK
# { "data": [...], "meta": { "total": 100, "limit": 20, "offset": 0 } }

# Step 3: Partial update with PATCH.
PATCH /v1/users/42 HTTP/1.1
Content-Type: application/json
Authorization: Bearer <token>

{ "email": "new@example.com" }

# Response:
# 200 OK

# Step 4: Standardized error for a missing resource.
GET /v1/users/999 HTTP/1.1
Authorization: Bearer <token>

# Response:
# 404 Not Found
# Content-Type: application/problem+json
# { "type": "https://api.example.com/problems/not-found",
#   "title": "User not found", "status": 404,
#   "detail": "User 999 does not exist." }
```

**Expected Output (conceptual):**
```
201 Created with Location
200 OK with paginated data and metadata
200 OK for partial update
404 Not Found with RFC 9457 body
```

**Why this output occurs.** The example applies resource naming, correct methods, `201` with `Location`, pagination metadata, `PATCH` for partial updates, and a standard error shape — demonstrating how best practices compose into a coherent API.

---

#### Example 2: Versioning and deprecation headers

```http
# Versioned endpoint (v1).
GET /v1/users/42 HTTP/1.1

# Response:
# 200 OK
# Deprecation: true
# Sunset: Sat, 31 Dec 2026 23:59:59 GMT
# Link: </v2/users/42>; rel="successor-version"
```

**Expected Output (conceptual):**
```
200 OK with Deprecation and Sunset headers.
Clients can detect the upcoming removal and migrate to /v2.
```

**Why this output occurs.** The `Deprecation` header signals that the endpoint will be removed; `Sunset` gives the date; `Link` points to the successor version. This allows clients to migrate proactively rather than discovering the removal through a breaking change.

---

### Real-World Cases

- **Stripe API**: versioned paths, idempotency keys, standardized errors, pagination, and extensive documentation.
- **GitHub API**: resource-oriented URLs, `Link` headers for pagination, and consistent error shapes.
- **Google API Design Guide**: standard methods, resource naming, versioning, and error models that align with these best practices.

---

## References

- Best Practices Guide for REST API Design — abp.io - https://abp.io/community/articles/best-practices-guide-for-rest-api-design-oexc1euj
- REST API Naming Conventions: A Practical Style Guide — apidog.com - https://apidog.com/blog/rest-api-naming-conventions/
- URIs — IBM Cloud API Handbook - https://bvt.console.cloud.ibm.com/docs/api-handbook?topic=api-handbook-uris
- Web API Design Best Practices — Azure Architecture Center (Microsoft Learn) - https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design
- REST API Developer’s Guide — Oracle (Standard HTTP Status Codes) - https://docs.oracle.com/cd/G46969_01/books/OnDemRestAPI/c-Standard-HTTP-Status-Codes-and1030319.html
- RFC 9457: Problem Details for HTTP APIs — IETF - https://www.ietf.org/ietf-ftp/rfc/inline-errata/rfc9457.html
- API Convention — CATAPA HRIS API - https://gdplabs.gitbook.io/catapa-api/get-started/api-convention
- API Design Checklist — aiskillstore/marketplace (GitHub) - https://github.com/aiskillstore/marketplace/blob/main/skills/wshobson/api-design-principles/assets/api-design-checklist.md
- RFC 9110: HTTP Semantics — IETF - https://datatracker.ietf.org/doc/html/rfc9110
- Google API Design Guide - https://cloud.google.com/apis/design
- OpenAPI Specification 3.x - https://spec.openapis.org/oas/v3.1.0