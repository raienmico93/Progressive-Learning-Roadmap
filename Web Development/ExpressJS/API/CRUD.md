# CRUD APIs — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **CRUD** is an acronym for **Create, Read, Update, and Delete** — the four fundamental operations for managing persistent data. A **CRUD API** is an interface that exposes these operations over a resource, typically via HTTP methods: `POST` to create, `GET` to read, `PUT`/`PATCH` to update, and `DELETE` to remove. When these operations are designed according to REST constraints — resource-oriented URLs, standard HTTP semantics, and stateless communication — the result is a predictable, scalable, and maintainable interface.

**Technical Definition.** In a RESTful CRUD API, resources are identified by **URIs** (nouns), and operations are expressed through **HTTP methods** (verbs). The mapping is: `POST /resources` → Create (returns `201 Created` with a `Location` header); `GET /resources` and `GET /resources/{id}` → Read (returns `200 OK`); `PUT /resources/{id}` → full replacement and `PATCH /resources/{id}` → partial update (returns `200 OK` or `204 No Content`); `DELETE /resources/{id}` → Delete (returns `204 No Content` or `200 OK`). CRUD APIs additionally support **collection-level concerns**: pagination (splitting large result sets into pages), filtering (narrowing by field values), sorting (ordering results), and searching (matching free-text queries).

**Beginner-Friendly Explanation.** CRUD APIs are the **bread and butter of web development**. Almost every app you use — social media, e-commerce, banking — needs to create new records (sign up, place an order), read existing records (view a profile, list products), update records (change your email, edit a post), and delete records (cancel an account, remove an item). A CRUD API is the **set of web addresses and rules** that lets clients perform those four operations. When done right, the API feels obvious: once you know how to create a user, you can guess how to create an order.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Resource-centric** | URLs identify **nouns** (e.g., `/users`), and HTTP methods express **actions** (e.g., `POST`). |
| **Uniform interface** | The same four methods apply consistently across resources, reducing cognitive load. |
| **Idempotent writes** | `PUT` and `DELETE` can be safely retried; `POST` requires idempotency keys for safe retries. |
| **Collection-aware** | List endpoints support pagination, filtering, sorting, and searching by default. |
| **Consistent status codes** | Each operation maps to a standard HTTP status code, enabling correct client and intermediary behavior. |
| **Evolvable** | Versioned URLs and backward-compatible extensions allow the API to grow without breaking clients. |

---

### Prerequisites

- **HTTP fundamentals**: methods, status codes, headers, and the request/response model.
- **REST architectural style**: resources, representations, statelessness.
- **JSON**: the dominant request and response format.
- **Query parameters**: how to encode filters, sorting, and pagination in URLs.
- **A tool to test APIs**: `curl`, Postman, or a language-specific HTTP client.

---

### Related Programming Areas

- **REST fundamentals**: CRUD is the practical application of REST to persistent data.
- **API design**: resource naming, URL structure, request/response formats, and error models.
- **Database operations**: CRUD mirrors SQL `INSERT`, `SELECT`, `UPDATE`, and `DELETE`.
- **Frontend data fetching**: CRUD endpoints are consumed by web and mobile clients for data management.
- **API documentation**: OpenAPI/Swagger specifications describe CRUD endpoints and their schemas.

---

### Core Concepts / Features

The following nine sections cover the requested concepts. Each follows the uniform structure: definitions, purposes, syntax rules, annotated executable examples with expected outputs, and real-world cases.

---

## 1. Create

### Definitions

**Core Definition.** **Create** is the CRUD operation that **inserts a new resource** into the system and returns its identifier or representation.

**Technical Definition.** In HTTP, Create is implemented with **`POST /resources`** (server-assigned identifier) or **`PUT /resources/{id}`** (client-assigned identifier). A successful `POST` returns **`201 Created`** with a **`Location` header** pointing to the new resource and, typically, a representation of the created resource in the response body. Create is **not idempotent** when using `POST`: repeating the request may create duplicate resources. To make Create safely retryable, clients should generate an **idempotency key** that the server uses to deduplicate requests.

**Beginner-Friendly Explanation.** Create means **making something new**. When you sign up for an account, post a comment, or add an item to a cart, the API is creating a new record. You send the data in the request body, and the server responds with the new record’s ID and a link to it. If the request times out and you retry, you might accidentally create two records — unless the API supports **idempotency keys**, which let the server recognize the retry as the same request.

### Purposes

- **To insert new records** into persistent storage and return the created representation.
- **To assign and expose a stable identifier** (via `Location` header) for the new resource.
- **To enforce validation rules** before persisting data, returning `400` or `422` on failure.
- **To support safe retries** through idempotency keys for payment and order-creation flows.

### Syntax Rules and Structure

**General syntax (Create with POST).**

```
POST /v1/{resource}
Content-Type: application/json
Authorization: Bearer <token>

{ "field": "value" }
```

**Component breakdown.**

- `POST` — the HTTP method for creating a new resource.
- `/v1/{resource}` — the collection endpoint (e.g., `/v1/users`).
- `Content-Type: application/json` — declares the body format.
- `Authorization` — carries the self-contained credential.
- Body — the new resource’s representation.

**Status codes for Create.**

| Code | Meaning | When to Use |
|---|---|---|
| **`201 Created`** | Resource created successfully | Standard for `POST`; include `Location` header. |
| **`202 Accepted`** | Request accepted for async processing | When creation is asynchronous. |
| **`400 Bad Request`** | Malformed request | Invalid JSON, missing required fields. |
| **`409 Conflict`** | Duplicate resource | Unique constraint violation (e.g., email already exists). |
| **`422 Unprocessable Entity`** | Semantic validation failure | Well-formed JSON but invalid business rules. |

**Constraints and limitations.**

- **`POST` is not idempotent**; retrying without an idempotency key may create duplicates.
- **`PUT` can also create** if the client supplies the identifier and the resource does not yet exist.
- **`Location` header is expected** on `201 Created` responses; omitting it forces clients to guess the new URL.

### Annotated Complete Step by Step Code Examples

#### Example 1: Creating a user with `POST`

```http
# Step 1: Send a POST request to the users collection.
POST /v1/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

{
  "name": "Alice",
  "email": "alice@example.com"
}

# Step 2: Server validates the body.
# Step 3: Server inserts the record and assigns an ID (e.g., 42).
# Step 4: Server responds with 201 Created and a Location header.
```

**Expected Response:**
```
HTTP/1.1 201 Created
Location: /v1/users/42
Content-Type: application/json

{
  "id": 42,
  "name": "Alice",
  "email": "alice@example.com"
}
```

**Why this output occurs.** The `POST` method signals creation. The server validates the JSON body, inserts the record, and returns `201 Created` with a `Location` header pointing to the new resource. The response body includes the created representation, allowing the client to use the resource immediately without a follow-up `GET`.

---

#### Example 2: Idempotent Create with an idempotency key

```http
POST /v1/payments HTTP/1.1
Host: api.example.com
Content-Type: application/json
Idempotency-Key: 7c9e6679-7425-40de-944b-e07fc1f90ae7

{ "amount": 5000, "currency": "USD" }
```

**Expected Response (first request):**
```
HTTP/1.1 201 Created
Location: /v1/payments/pay_123
```

**Expected Response (retry with the same key):**
```
HTTP/1.1 200 OK
Content-Type: application/json

{ "id": "pay_123", "amount": 5000, "currency": "USD" }
```

**Why this output occurs.** The client generates a unique `Idempotency-Key` and sends it with the request. If the first request succeeds but the response is lost, the client retries with the **same key**. The server recognizes the key, finds the existing payment, and returns it instead of creating a duplicate. This pattern is essential for payment and order-creation APIs.

---

### Real-World Cases

- **Stripe API**: `POST /v1/charges` creates a charge; supports `Idempotency-Key` headers to prevent duplicate charges on retry.
- **GitHub API**: `POST /repos/{owner}/{repo}/issues` creates an issue and returns `201 Created` with a `Location` header.
- **Auth0 API**: `POST /api/v2/users` creates a user and returns `201 Created` with the user representation.

---

## 2. Read

### Definitions

**Core Definition.** **Read** is the CRUD operation that **retrieves** one or more resources without modifying server state. It is **safe** and **idempotent**.

**Technical Definition.** Read is implemented with **`GET /resources`** (list/collection) and **`GET /resources/{id}`** (single item). A successful read returns **`200 OK`** with a representation of the resource(s) in the response body. Collection reads should support **pagination**, **filtering**, **sorting**, and **searching** through query parameters. Reads are **safe** (no side effects) and **idempotent** (repeating the request produces the same result), which makes them cacheable and safe to retry.

**Beginner-Friendly Explanation.** Read means **looking at data without changing it**. When you open your profile, scroll through a feed, or search for a product, the API is reading data. You send a `GET` request, and the server responds with the data. Because reading does not change anything, it is safe to retry and safe to cache — which is why your browser can cache images and APIs can use CDNs.

### Purposes

- **To retrieve a single resource** by its identifier for display or processing.
- **To list a collection** of resources with optional pagination, filtering, sorting, and searching.
- **To support caching** through safe, idempotent semantics and cache headers.
- **To enable field selection** (sparse fieldsets) to reduce payload size.

### Syntax Rules and Structure

**General syntax (Read single item).**

```
GET /v1/{resource}/{id}
Accept: application/json
Authorization: Bearer <token>
```

**General syntax (List collection).**

```
GET /v1/{resource}?limit=20&offset=0&status=active&sort=-created_at&q=search
```

**Component breakdown.**

- `GET` — the safe, idempotent method for reading.
- `/v1/{resource}/{id}` — the item endpoint.
- Query parameters — `limit` and `offset` for pagination, field filters, `sort` for ordering, `q` for search.
- `Accept` — declares the preferred response format.

**Status codes for Read.**

| Code | Meaning | When to Use |
|---|---|---|
| **`200 OK`** | Resource(s) returned | Standard for successful `GET`. |
| **`206 Partial Content`** | Partial response | For range requests on large resources. |
| **`304 Not Modified`** | Cached version is still valid | Conditional `GET` with `If-None-Match`. |
| **`404 Not Found`** | Resource does not exist | Invalid or missing identifier. |

**Constraints and limitations.**

- **`GET` must not have side effects**; violating this breaks caching and safety guarantees.
- **Unbounded collection reads** can overwhelm clients and servers; always paginate.
- **Missing `404` handling** for absent resources leads to ambiguous client behavior.

### Annotated Complete Step by Step Code Examples

#### Example 1: Reading a single resource

```http
GET /v1/users/42 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Expected Response:**
```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 42,
  "name": "Alice",
  "email": "alice@example.com"
}
```

**Why this output occurs.** The `GET` request targets a specific resource by its identifier. The server finds the record and returns `200 OK` with its representation. Because `GET` is safe and idempotent, the client can retry this request without side effects.

---

#### Example 2: Listing a collection with pagination and filtering

```http
GET /v1/users?limit=2&offset=0&status=active&sort=-created_at HTTP/1.1
Host: api.example.com
Accept: application/json
```

**Expected Response:**
```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [
    { "id": 42, "name": "Alice", "status": "active" },
    { "id": 41, "name": "Bob", "status": "active" }
  ],
  "meta": {
    "total": 100,
    "limit": 2,
    "offset": 0
  }
}
```

**Why this output occurs.** The query parameters control the result set: `limit` and `offset` paginate, `status=active` filters, and `sort=-created_at` orders by creation date descending. The response includes a `meta` object with pagination metadata so the client knows how to fetch the next page.

---

### Real-World Cases

- **GitHub API**: `GET /repos/{owner}/{repo}/issues?state=open&labels=bug&sort=created&direction=desc` — reads a filtered, sorted collection.
- **Stripe API**: `GET /v1/charges?customer=cus_123&limit=10&starting_after=ch_abc` — cursor-based pagination for charges.
- **Plane API**: `GET /api/v2/workspaces/{slug}/projects/{id}/work-items/` — reads a collection with filtering and cursor pagination.

---

## 3. Update

### Definitions

**Core Definition.** **Update** is the CRUD operation that **modifies an existing resource**. REST distinguishes **full replacement** (`PUT`) from **partial update** (`PATCH`).

**Technical Definition.** **`PUT /resources/{id}`** replaces the entire resource representation; any fields omitted from the request body are removed or reset. **`PATCH /resources/{id}`** applies a partial modification, changing only the fields present in the body. Both are **idempotent** by HTTP definition (repeating the request produces the same end state), though `PATCH` idempotency depends on the patch design. Successful updates return **`200 OK`** with the updated representation or **`204 No Content`** if no body is returned.

**Beginner-Friendly Explanation.** Update means **changing something that already exists**. If you replace your entire profile with a new one, that is `PUT` — everything is overwritten. If you only change your email address, that is `PATCH` — only that field is modified. Both are safe to retry because repeating the same update produces the same result. If two people edit the same record at the same time, the server may return `409 Conflict` to prevent lost updates.

### Purposes

- **To replace an entire resource** with a new representation (`PUT`).
- **To modify a subset of fields** without touching the rest (`PATCH`).
- **To enforce optimistic concurrency** through version fields or `ETag` headers, returning `409 Conflict` on stale updates.
- **To return the updated representation** so clients can immediately reflect the change.

### Syntax Rules and Structure

**General syntax (PUT — full replacement).**

```
PUT /v1/users/42
Content-Type: application/json

{ "name": "Alice Smith", "email": "alice.smith@example.com" }
```

**General syntax (PATCH — partial update).**

```
PATCH /v1/users/42
Content-Type: application/json

{ "email": "new@example.com" }
```

**Component breakdown.**

- `PUT` — replaces the entire resource; omitted fields are removed.
- `PATCH` — modifies only the fields present in the body.
- `/v1/users/42` — the target resource.
- Body — the new or partial representation.

**Status codes for Update.**

| Code | Meaning | When to Use |
|---|---|---|
| **`200 OK`** | Update successful, representation returned | Standard `PUT`/`PATCH`. |
| **`204 No Content`** | Update successful, no body | When the server does not return the updated resource. |
| **`404 Not Found`** | Resource does not exist | Invalid identifier. |
| **`409 Conflict`** | State conflict | Concurrent modification or version mismatch. |
| **`422 Unprocessable Entity`** | Validation failure | Well-formed JSON but invalid values. |

**Constraints and limitations.**

- **`PUT` must not merge**; it replaces, so clients must send the complete representation.
- **`PATCH` idempotency is not guaranteed**; design patches carefully to avoid non-idempotent operations.
- **Concurrent updates** require optimistic locking (`ETag`, `If-Match`) to prevent lost updates.

### Annotated Complete Step by Step Code Examples

#### Example 1: Full replacement with `PUT`

```http
PUT /v1/users/42 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "name": "Alice Smith",
  "email": "alice.smith@example.com"
}
```

**Expected Response:**
```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 42,
  "name": "Alice Smith",
  "email": "alice.smith@example.com"
}
```

**Why this output occurs.** `PUT` replaces the entire resource. Any fields not included in the body are removed or reset. The server returns the updated representation. Repeating the same `PUT` request produces the same end state, satisfying idempotency.

---

#### Example 2: Partial update with `PATCH`

```http
PATCH /v1/users/42 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{ "email": "new@example.com" }
```

**Expected Response:**
```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 42,
  "name": "Alice Smith",
  "email": "new@example.com"
}
```

**Why this output occurs.** `PATCH` modifies only the `email` field, leaving `name` unchanged. The server returns the updated representation. This is useful when clients want to change one field without sending the entire resource.

---

### Real-World Cases

- **GitHub API**: `PATCH /repos/{owner}/{repo}/issues/{issue_number}` partially updates an issue (title, body, state).
- **Stripe API**: `POST /v1/customers/{customer}` updates a customer (Stripe uses `POST` for updates, a common alternative).
- **Google API Design Guide**: recommends `PATCH` for partial updates and `PUT` for full replacement, with `409 Conflict` for concurrent modifications.

---

## 4. Delete

### Definitions

**Core Definition.** **Delete** is the CRUD operation that **removes a resource** from the system. It is **idempotent**: repeating the request produces the same end state (the resource is gone).

**Technical Definition.** Delete is implemented with **`DELETE /resources/{id}`**. A successful delete returns **`204 No Content`** (no response body) or **`200 OK`** with a confirmation body. Because `DELETE` is idempotent, repeating the request after the resource is gone should return `404 Not Found` (or `204` if the server chooses to treat deletion of a missing resource as success). Soft deletion (marking a record as deleted rather than physically removing it) is a common pattern that preserves auditability.

**Beginner-Friendly Explanation.** Delete means **removing something**. When you delete a post or cancel an account, the API removes the record. If you accidentally send the delete request twice, the second request might return `404` because the resource is already gone — but the end state is the same, which is why `DELETE` is idempotent. Many systems use **soft deletion**, marking a record as deleted instead of actually removing it, so it can be recovered or audited later.

### Purposes

- **To permanently remove a resource** when it is no longer needed.
- **To support soft deletion** for auditability and recoverability.
- **To return a confirmation** via `204 No Content` or a minimal body.
- **To maintain idempotency** so retries are safe.

### Syntax Rules and Structure

**General syntax (DELETE).**

```
DELETE /v1/users/42
Authorization: Bearer <token>
```

**Component breakdown.**

- `DELETE` — the idempotent method for removal.
- `/v1/users/42` — the target resource.
- No request body is required (though some APIs accept an optional body for soft-delete metadata).

**Status codes for Delete.**

| Code | Meaning | When to Use |
|---|---|---|
| **`204 No Content`** | Deleted successfully, no body | Standard for `DELETE`. |
| **`200 OK`** | Deleted successfully, body returned | When the server returns a confirmation or the deleted representation. |
| **`404 Not Found`** | Resource does not exist | Invalid or already-deleted identifier. |
| **`409 Conflict`** | Deletion blocked | Resource is referenced by other records. |

**Constraints and limitations.**

- **`DELETE` is idempotent**; repeating it should not change the outcome.
- **Hard delete is irreversible**; consider soft delete for auditability.
- **Cascading deletes** must be handled carefully to avoid orphaned records.

### Annotated Complete Step by Step Code Examples

#### Example 1: Deleting a resource

```http
DELETE /v1/users/42 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Expected Response:**
```
HTTP/1.1 204 No Content
```

**Why this output occurs.** The `DELETE` request targets a specific resource. The server removes the record and returns `204 No Content`, indicating success without a response body. Repeating the same `DELETE` request will return `404 Not Found` because the resource no longer exists — but the end state is unchanged, satisfying idempotency.

---

#### Example 2: Soft delete with `PATCH`

```http
PATCH /v1/users/42 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{ "deleted_at": "2026-10-26T12:00:00Z" }
```

**Expected Response:**
```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 42,
  "name": "Alice",
  "deleted_at": "2026-10-26T12:00:00Z"
}
```

**Why this output occurs.** Instead of physically removing the record, the server marks it as deleted by setting a `deleted_at` timestamp. Subsequent read requests filter out records with a non-null `deleted_at`. This preserves the data for auditing or recovery while still making the resource logically unavailable.

---

### Real-World Cases

- **GitHub API**: `DELETE /repos/{owner}/{repo}/issues/{issue_number}` deletes an issue (returns `204`).
- **Stripe API**: `DELETE /v1/customers/{customer}` deletes a customer (returns a deletion confirmation object).
- **Soft-delete systems**: Many SaaS APIs use `deleted_at` or `is_deleted` flags instead of hard deletes, enabling trash/recovery workflows.

---

## 5. Pagination

### Definitions

**Core Definition.** **Pagination** is the technique of **splitting a large collection of results into smaller, manageable chunks** (pages) so clients can retrieve them incrementally rather than in one massive response.

**Technical Definition.** Two dominant strategies exist. **Offset-based pagination** uses `limit` (page size) and `offset` (number of records to skip) query parameters; it is simple and supports direct page access but degrades on deep pages and becomes inconsistent when the underlying data changes. **Cursor-based pagination** uses an opaque token (cursor) that points to a specific position in the result set; it is stable under concurrent writes and performant for large datasets, but does not support random page access. Responses should include **pagination metadata** (`total`, `limit`, `offset`/`cursor`, and `next` link).

**Beginner-Friendly Explanation.** Pagination is **breaking a long list into pages**. Instead of loading 10,000 products at once, the API returns 20 at a time. With **offset pagination**, you say “skip 40, give me 20” — simple, but slow for deep pages. With **cursor pagination**, the API gives you a bookmark after each page, and you say “continue from this bookmark” — stable and fast, but you cannot jump to page 50 directly.

### Purposes

- **To prevent overwhelming clients and servers** with massive responses.
- **To improve performance** by limiting the amount of data transferred per request.
- **To provide a consistent navigation model** for large datasets.
- **To enable infinite scroll and “load more”** experiences in clients.

### Syntax Rules and Structure

**Offset-based pagination syntax.**

```
GET /v1/{resource}?limit=20&offset=0
```

- `limit` — maximum number of items to return.
- `offset` — number of items to skip.

**Cursor-based pagination syntax.**

```
GET /v1/{resource}?limit=20&cursor=eyJpZCI6MTIzfQ==
```

- `limit` — maximum number of items to return.
- `cursor` — opaque token pointing to the next position.

**Pagination metadata in responses.**

```json
{
  "data": [ ... ],
  "meta": {
    "total": 100,
    "limit": 20,
    "offset": 0,
    "next": "/v1/users?limit=20&offset=20"
  }
}
```

**Pagination rules.**

| Rule | Guidance |
|---|---|
| **Always paginate collections** | Do not return unbounded collections. |
| **Provide sensible defaults** | Default `limit` (e.g., 20) and `offset`/`cursor`. |
| **Enforce a maximum limit** | Prevent clients from requesting excessive pages. |
| **Return metadata** | Include `total`, `limit`, and `next` for navigation. |
| **Choose strategy by dataset** | Offset for small/static; cursor for large/dynamic. |

**Constraints and limitations.**

- **Offset pagination degrades** on deep offsets (`OFFSET 1000000` is slow).
- **Offset pagination is inconsistent** under concurrent writes (records may shift).
- **Cursor pagination does not support random access** to arbitrary pages.

### Annotated Complete Step by Step Code Examples

#### Example 1: Offset-based pagination

```http
# Page 1
GET /v1/users?limit=2&offset=0 HTTP/1.1

# Page 2
GET /v1/users?limit=2&offset=2 HTTP/1.1
```

**Expected Response (Page 1):**
```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [
    { "id": 1, "name": "Alice" },
    { "id": 2, "name": "Bob" }
  ],
  "meta": {
    "total": 6,
    "limit": 2,
    "offset": 0
  }
}
```

**Why this output occurs.** `limit=2` returns two records, and `offset=0` starts at the beginning. The `meta` object tells the client there are 6 total records, so the next page should use `offset=2`.

---

#### Example 2: Cursor-based pagination

```http
# First page
GET /v1/users?limit=2 HTTP/1.1

# Response includes a cursor
# { "data": [...], "meta": { "next_cursor": "abc123", "has_more": true } }

# Next page uses the cursor
GET /v1/users?limit=2&cursor=abc123 HTTP/1.1
```

**Expected Response (next page):**
```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [
    { "id": 3, "name": "Carol" },
    { "id": 4, "name": "Dave" }
  ],
  "meta": {
    "next_cursor": "def456",
    "has_more": true
  }
}
```

**Why this output occurs.** The first response includes an opaque `next_cursor`. The client sends it in the next request to continue from the exact position in the sorted result set. Unlike offset pagination, this remains stable even if records are inserted or deleted between requests.

---

### Real-World Cases

- **Stripe API**: uses cursor-based pagination with `starting_after` and `ending_before` parameters for large datasets.
- **GitHub API**: uses `Link` headers with `rel="next"`, `rel="prev"`, `rel="first"`, and `rel="last"` for pagination.
- **Plane API**: uses cursor pagination with `next_cursor` and `has_more` fields.

---

## 6. Filtering

### Definitions

**Core Definition.** **Filtering** is the technique of **narrowing a collection** by applying conditions to field values, so only matching resources are returned.

**Technical Definition.** Filtering is implemented through **query parameters** on `GET` collection endpoints. Simple filters use direct field-value pairs (`?status=active`); advanced filters use operator suffixes (`?created_after=2024-01-01`, `?price[gte]=100`) or a whitelisted filter grammar. Filterable fields must be **explicitly allowed** by the server; unknown filters should be rejected to prevent injection and abuse. Every filterable column should be **indexed** for performance.

**Beginner-Friendly Explanation.** Filtering is **narrowing down a list**. When you shop online and select “price under $50” and “in stock,” you are filtering. The API receives those conditions as query parameters and returns only the items that match. The server must decide which fields are allowed to be filtered — you cannot filter on a password field, for example — and should reject unknown filters.

### Purposes

- **To reduce the result set** to only relevant records.
- **To support user-driven narrowing** (e.g., by status, category, date range).
- **To improve performance** by reducing the amount of data transferred.
- **To enable faceted navigation** and dashboard-style filtering.

### Syntax Rules and Structure

**Simple filtering syntax.**

```
GET /v1/users?status=active&role=admin
```

**Operator-based filtering syntax.**

```
GET /v1/orders?created_after=2025-01-01&amount_gt=100
GET /v1/tasks?due_date[lte]=2024-12-31
```

**Component breakdown.**

- `status=active` — exact match on the `status` field.
- `created_after=2025-01-01` — date comparison using a suffixed operator.
- `due_date[lte]=2024-12-31` — bracket notation for the “less than or equal” operator.

**Filtering rules.**

| Rule | Guidance |
|---|---|
| **Whitelist filterable fields** | Reject unknown filters. |
| **Index filterable columns** | Ensure performance for common filters. |
| **Document semantics** | Clarify whether `name=John` is exact or substring match. |
| **Use explicit operators** | Avoid query DSL strings; prefer typed parameters. |

**Constraints and limitations.**

- **Unindexed filters** can cause full table scans and degrade performance.
- **Complex filter combinations** may exceed query-parameter limits; consider a `POST /search` endpoint with a body.
- **Filter parameter names are not standardized** across APIs; choose a convention and apply it consistently.

### Annotated Complete Step by Step Code Examples

#### Example 1: Simple filtering

```http
GET /v1/orders?status=shipped&minCost=100 HTTP/1.1
Host: api.example.com
```

**Expected Response:**
```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [
    { "id": 1001, "status": "shipped", "cost": 150 },
    { "id": 1002, "status": "shipped", "cost": 200 }
  ]
}
```

**Why this output occurs.** The `status=shipped` filter returns only shipped orders, and `minCost=100` returns only orders with a cost of at least 100. Both conditions are combined with AND semantics. The server has whitelisted and indexed the `status` and `cost` fields for efficient filtering.

---

#### Example 2: Operator-based filtering

```http
GET /v1/tasks?status=active&due_date[lte]=2024-12-31 HTTP/1.1
Host: api.example.com
```

**Expected Response:**
```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [
    { "id": 1, "status": "active", "due_date": "2024-12-15" },
    { "id": 2, "status": "active", "due_date": "2024-12-31" }
  ]
}
```

**Why this output occurs.** The bracket notation `due_date[lte]` applies the “less than or equal” operator, returning only tasks due on or before 2024-12-31. This operator-based syntax is explicit and avoids ambiguous semantics.

---

### Real-World Cases

- **Azure API Design**: `/orders?minCost=100&status=shipped` — simple field-value filters.
- **CubeCart API**: `/api/v1/products?cat_id=5&status=1` — filtering products by category and status.
- **Plane API**: supports filtering by multiple fields with case-insensitive exact matching.

---

## 7. Sorting

### Definitions

**Core Definition.** **Sorting** is the technique of **ordering a collection** by one or more fields, either ascending or descending, before returning it.

**Technical Definition.** Sorting is implemented through a **`sort` query parameter** on `GET` collection endpoints. The value is typically a comma-separated list of field names, where a leading `-` indicates descending order (`sort=-created_at,priority`). The server must **whitelist sortable fields** and reject unknown ones. Sorting affects **cacheability** because the sort order is part of the response identity; caches must key on the full query string.

**Beginner-Friendly Explanation.** Sorting is **choosing the order of a list**. When you sort products by price low-to-high or reviews by newest first, you are using sorting. The API receives a `sort` parameter and returns the data in that order. The server should only allow sorting on fields it has approved — sorting by a password field would be a security risk — and it should reject sort fields it does not recognize.

### Purposes

- **To present data in a meaningful order** (e.g., newest first, highest rated).
- **To support user-controlled ordering** through query parameters.
- **To enable stable pagination** when combined with cursor-based pagination on a sortable field.
- **To improve user experience** by surfacing relevant results first.

### Syntax Rules and Structure

**General sorting syntax.**

```
GET /v1/{resource}?sort=-created_at,priority
```

**Component breakdown.**

- `sort` — the sorting parameter.
- `-created_at` — descending order on `created_at` (the `-` prefix means descending).
- `priority` — ascending order on `priority` (no prefix means ascending).
- Comma-separated fields are applied in order (multi-field sorting).

**Sorting rules.**

| Rule | Guidance |
|---|---|
| **Whitelist sortable fields** | Reject unknown sort fields. |
| **Use a consistent prefix convention** | `-` for descending, no prefix for ascending. |
| **Document default sort** | Specify the default order when no `sort` is provided. |
| **Index sortable columns** | Ensure performance for common sort fields. |

**Constraints and limitations.**

- **Sorting on unindexed fields** can cause full table scans.
- **Multi-field sorting** requires careful index design for performance.
- **Cache keys must include the sort parameter** to avoid serving incorrectly ordered responses.

### Annotated Complete Step by Step Code Examples

#### Example 1: Single-field descending sort

```http
GET /v1/products?sort=-price HTTP/1.1
Host: api.example.com
```

**Expected Response:**
```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [
    { "id": 1, "name": "Laptop", "price": 1200 },
    { "id": 2, "name": "Phone", "price": 800 },
    { "id": 3, "name": "Tablet", "price": 500 }
  ]
}
```

**Why this output occurs.** The `-price` value sorts products by price in descending order (highest first). The server has whitelisted `price` as a sortable field and indexed it for performance.

---

#### Example 2: Multi-field sorting

```http
GET /v1/tasks?sort=-priority,created_at HTTP/1.1
Host: api.example.com
```

**Expected Response:**
```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [
    { "id": 1, "priority": "high", "created_at": "2024-01-01" },
    { "id": 2, "priority": "high", "created_at": "2024-01-02" },
    { "id": 3, "priority": "low", "created_at": "2024-01-01" }
  ]
}
```

**Why this output occurs.** The sort is applied in order: first by `priority` descending (high before low), then by `created_at` ascending within each priority group. This multi-field sort uses comma-separated fields with the `-` prefix for descending.

---

### Real-World Cases

- **CubeCart API**: `?sort=price&order=desc` — sorts products by price descending.
- **GitHub API**: `?sort=created&direction=desc` — sorts issues by creation date descending.
- **Plane API**: uses `ordering` query parameters for sorting work items.

---

## 8. Searching

### Definitions

**Core Definition.** **Searching** is the technique of **matching a free-text query** against one or more fields of a resource, returning resources that contain or relate to the query.

**Technical Definition.** Searching is implemented through a **`q` or `search` query parameter** on `GET` collection endpoints. Simple search uses `LIKE`/`ILIKE` pattern matching across designated fields; full-text search uses database-native capabilities (e.g., PostgreSQL `to_tsvector`/`plainto_tsquery`); and large-scale search may delegate to a dedicated search engine (Elasticsearch, Meilisearch) for fuzzy matching, faceting, and typo tolerance. Search should be **scoped** (limited to relevant fields) and **documented** (whether it is substring, prefix, or full-text matching).

**Beginner-Friendly Explanation.** Searching is **looking for something by typing words**. When you type “gaming laptop” into a search box, the API searches across fields like product name and description and returns matches. Simple search just checks whether the text appears; full-text search understands words and relevance; and a dedicated search engine can handle typos and fuzzy matches. The API should tell you which fields it searches and how it matches.

### Purposes

- **To find resources by free-text query** without requiring users to know exact field values.
- **To support simple and full-text search** depending on scale and requirements.
- **To scope search to relevant fields** (e.g., title and description, not internal notes).
- **To delegate to a dedicated search engine** when fuzzy matching, faceting, or typo tolerance is required.

### Syntax Rules and Structure

**Simple search syntax.**

```
GET /v1/products?q=gaming+laptop
```

**Scoped search syntax.**

```
GET /v1/products?q=gaming&category=electronics
```

**Component breakdown.**

- `q` — the free-text search query.
- `category=electronics` — an additional filter to scope the search.
- The server searches designated fields (e.g., `title`, `description`) for the query terms.

**Search rules.**

| Rule | Guidance |
|---|---|
| **Document searched fields** | Specify which fields the query matches. |
| **Scope search appropriately** | Do not search sensitive or internal fields. |
| **Choose the right engine** | `LIKE` for small data; full-text for larger; dedicated search engine for fuzzy/faceted needs. |
| **Combine with filters** | Allow search to be narrowed by field filters. |

**Constraints and limitations.**

- **`LIKE '%term%'` cannot use indexes** and is slow on large tables.
- **Full-text search requires database support** and language configuration.
- **Dedicated search engines introduce synchronization complexity** (index updates via CDC or events).

### Annotated Complete Step by Step Code Examples

#### Example 1: Simple search with `q`

```http
GET /v1/products?q=gaming+laptop HTTP/1.1
Host: api.example.com
```

**Expected Response:**
```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [
    { "id": 1, "name": "Gaming Laptop Pro", "description": "High-performance gaming laptop" },
    { "id": 2, "name": "Ultrabook", "description": "Lightweight laptop for gaming" }
  ]
}
```

**Why this output occurs.** The `q=gaming+laptop` query searches designated fields (`name` and `description`) for the terms “gaming” and “laptop.” Both matching resources are returned. Simple search typically uses `LIKE`/`ILIKE` pattern matching and works well for small to medium datasets.

---

#### Example 2: Scoped search with a filter

```http
GET /v1/products?q=laptop&category=electronics HTTP/1.1
Host: api.example.com
```

**Expected Response:**
```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [
    { "id": 1, "name": "Gaming Laptop Pro", "category": "electronics" }
  ]
}
```

**Why this output occurs.** The search query `q=laptop` is combined with the filter `category=electronics`, narrowing the results to laptops in the electronics category. This demonstrates how search and filtering compose to produce precise result sets.

---

### Real-World Cases

- **CubeCart API**: `?search=leather+wallet` — simple search across product fields.
- **GitHub API**: `?q=repo:owner/name+is:issue+label:bug` — scoped, qualified search.
- **Plane API**: uses a dedicated search parameter for free-text matching in list endpoints.

---

## 9. CRUD Best Practices

### Definitions

**Core Definition.** **CRUD best practices** are empirically established guidelines for designing CRUD APIs that are **predictable, consistent, secure, performant, and evolvable** across their entire lifecycle.

**Technical Definition.** Best practices span **resource naming** (nouns, plural collections, kebab-case, max 2 nesting levels), **HTTP method semantics** (`GET` read, `POST` create, `PUT` replace, `PATCH` partial update, `DELETE` remove), **status code usage** (`200`, `201`, `204`, `400`, `401`, `403`, `404`, `409`, `422`, `429`, `500`), **pagination** (offset or cursor), **filtering/sorting/searching** (whitelisted query parameters), **idempotency** (keys for `POST`, natural idempotency for `PUT`/`DELETE`), **versioning** (from day one), **error responses** (RFC 9457 Problem Details), and **security** (HTTPS, authentication, input validation, rate limiting).

**Beginner-Friendly Explanation.** Best practices are **the habits that make a CRUD API pleasant to use**. Use nouns in URLs, not verbs. Use the right HTTP method for each operation. Return meaningful status codes. Paginate collections. Whitelist filterable and sortable fields. Use idempotency keys for payments. Version your API from the start. Return consistent errors. These are the things that separate an API developers enjoy from one they dread.

### Purposes

- **To make APIs predictable and easy to learn** through consistent resource naming and method usage.
- **To prevent breaking changes** by versioning and treating the API as a contract.
- **To protect data and users** through HTTPS, authentication, input validation, and rate limiting.
- **To ensure reliability** through idempotency, pagination, and standardized error responses.
- **To enable observability** through logging, metrics, and request IDs.

### Syntax Rules and Structure

**CRUD best-practice checklist.**

| Area | Best Practice |
|---|---|
| **Resources** | Nouns, plural collections, kebab-case, max 2 nesting levels. |
| **HTTP methods** | `GET` read, `POST` create, `PUT` replace, `PATCH` partial update, `DELETE` remove. |
| **Status codes** | Use the full 2xx/4xx/5xx families correctly; `201` for create, `204` for delete. |
| **Pagination** | Offset for small/static; cursor for large/dynamic; always include metadata. |
| **Filtering** | Whitelist fields; reject unknown filters; index filterable columns. |
| **Sorting** | Whitelist fields; use `-` prefix for descending; document default. |
| **Searching** | Document searched fields; choose engine based on scale. |
| **Idempotency** | Idempotency keys for `POST`; `PUT`/`DELETE` naturally idempotent. |
| **Versioning** | Version from day one; never mutate the contract silently. |
| **Errors** | RFC 9457 Problem Details; field-level validation errors. |
| **Security** | HTTPS enforced; authentication via JWT/OAuth; input validation; rate limiting. |

**Constraints and limitations.**

- **Best practices are conventions**, not enforced by HTTP; they must be applied deliberately.
- **Over-engineering small APIs** can add unnecessary complexity; prioritize consistency.
- **Versioning adds maintenance overhead**; choose a strategy and apply it consistently.

### Annotated Complete Step by Step Code Examples

#### Example 1: A well-designed CRUD endpoint set

```http
# Create
POST /v1/users HTTP/1.1
Content-Type: application/json
Authorization: Bearer <token>

{ "name": "Alice", "email": "alice@example.com" }

# Response: 201 Created
# Location: /v1/users/42

# Read (single)
GET /v1/users/42 HTTP/1.1
Authorization: Bearer <token>

# Response: 200 OK
# { "id": 42, "name": "Alice", "email": "alice@example.com" }

# Read (collection with pagination, filtering, sorting)
GET /v1/users?limit=20&offset=0&status=active&sort=-created_at HTTP/1.1
Authorization: Bearer <token>

# Response: 200 OK
# { "data": [...], "meta": { "total": 100, "limit": 20, "offset": 0 } }

# Update (partial)
PATCH /v1/users/42 HTTP/1.1
Content-Type: application/json
Authorization: Bearer <token>

{ "email": "new@example.com" }

# Response: 200 OK

# Delete
DELETE /v1/users/42 HTTP/1.1
Authorization: Bearer <token>

# Response: 204 No Content
```

**Expected Output (conceptual):**
```
201 Created with Location header
200 OK with the resource
200 OK with paginated, filtered, sorted data
200 OK for partial update
204 No Content for deletion
```

**Why this output occurs.** Each operation uses the correct method and status code. The collection endpoint supports pagination (`limit`, `offset`), filtering (`status=active`), and sorting (`sort=-created_at`). The API is versioned (`/v1`), authenticated (`Authorization`), and consistent in its URL patterns.

---

#### Example 2: Idempotency and error handling

```http
# Idempotent POST with a key
POST /v1/payments HTTP/1.1
Idempotency-Key: 7c9e6679-7425-40de-944b-e07fc1f90ae7
Content-Type: application/json

{ "amount": 5000, "currency": "USD" }

# Retry with the same key returns the same payment, not a duplicate.

# Error response (RFC 9457)
GET /v1/users/999 HTTP/1.1
Authorization: Bearer <token>

# Response: 404 Not Found
# Content-Type: application/problem+json
# {
#   "type": "https://api.example.com/problems/not-found",
#   "title": "User not found",
#   "status": 404,
#   "detail": "User 999 does not exist."
# }
```

**Expected Output (conceptual):**
```
Retry with the same Idempotency-Key returns the original payment (no duplicate).
404 Not Found with a standardized problem+json body.
```

**Why this output occurs.** The idempotency key allows the server to recognize retries and return the original result instead of creating a duplicate. The error response follows RFC 9457, providing a machine-readable `type` and a human-readable `detail` so clients can handle the failure programmatically.

---

### Real-World Cases

- **Stripe API**: versioned paths (`/v1/`), idempotency keys, cursor pagination, standardized errors, and consistent resource naming.
- **GitHub API**: plural resource names, `Link` headers for pagination, and `422` for validation failures.
- **Azure Architecture Center**: recommends offset pagination with a maximum limit, filtering via query parameters, and sorting via a `sort` parameter.

---

## References

- API Design Best Practices (2026) — vasilyu1983/AI-Agents-public - https://github.com/vasilyu1983/AI-Agents-public/blob/8dc5de47c1db00f8ba01806f5dddd798fc78cf22/frameworks/shared-skills/skills/dev-api-design/references/api-design-best-practices.md
- API Design Principles — plaesy/spec-kit - https://github.com/plaesy/spec-kit/blob/e86e020988bd630cabfb36db6d3647cd1e4bfcee/instructions/api-design-principles.instructions.md
- API Design Standards — AsiaOstrich/universal-dev-standards - https://github.com/asiaostrich/universal-dev-standards/blob/0e40bf5151067fffdd9adb35698ed35ea9dc6468/core/api-design-standards.md
- Pagination, Filtering & Sorting — UKHSA Collaboration - https://ukhsa-collaboration.github.io/standards-org/api-design-guidelines/api-guidelines/pagination-filtering-sorting/
- Handling API Pagination: Fetching Large Data Sets — API7.ai - https://api7.ai/learning-center/api-101/handling-api-pagination
- Web API Design Best Practices — Azure Architecture Center (Microsoft Learn) - https://learn.microsoft.com/pl-pl/azure/architecture/best-practices/api-design
- Pagination, Filtering & Sorting — CubeCart Knowledge Base - https://kb.cubecart.com/api-guide/getting-started/pagination-filtering-sorting
- Which HTTP Status Code to Use for Every CRUD App — Ahmed-Alanaswah/Read-Notes - https://raw.githubusercontent.com/Ahmed-Alanaswah/Read-Notes/0a41afba89adf999c9b96500725a3d7907cdf821/read301-12.md
- js-miden API Design Notes — srivtx/js-miden - https://github.com/srivtx/js-miden/tree/b66962fd1e3b6198342f57de8ea48b010ba6742b/docs_js_backend/05-api-design
- Backend Interview Handbook — REST APIs — princepsr/Backend-Interview-Handbook - https://github.com/princepsr/Backend-Interview-Handbook/blob/da35f8ef3779e885d642fc221ff572aa4dd3b319/Volume3_BackendSystems/Chapter9_REST_APIs.md
- Master RESTful API Design with Best Practices — Strapi - https://strapi.io/blog/restful-api-design-guide-principles-best-practices
- API Design Patterns — Gin Web Framework - https://gin-gonic.com/en/docs/routing/api-design/
- RFC 9110: HTTP Semantics — IETF - https://datatracker.ietf.org/doc/html/rfc9110
- RFC 9457: Problem Details for HTTP APIs — IETF - https://www.ietf.org/ietf-ftp/rfc/inline-errata/rfc9457.html