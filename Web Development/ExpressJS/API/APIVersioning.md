# API Versioning — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **API versioning** is the practice of exposing and managing **incompatible API contracts** without forcing every consumer to migrate simultaneously. A version identifies a **contract** that a client can rely on; it must change only when the contract itself changes, not because the backend was redeployed or an internal implementation was refactored.

**Technical Definition.** API versioning introduces a **version identifier** into the request-selection process — via the URI path, a request header, a query parameter, or content negotiation — so that the server can route the request to the appropriate contract implementation. The version is part of the **interface contract**; changing it signals that a breaking change has occurred. Breaking changes include removing or renaming a field clients use, changing a field’s type or meaning, making an optional request field required, removing an endpoint, or changing authentication, pagination, error, or idempotency behavior promised by the contract. Backward-compatible changes — adding an optional field, adding a new endpoint, or fixing a bug without changing external behavior — do not require a new version.

**Beginner-Friendly Explanation.** API versioning is **how you change your API without breaking everyone who already uses it**. Imagine your API is a menu at a restaurant. If you change the menu, regulars who ordered the old dishes will be upset. Versioning lets you print a **new menu** (v2) while keeping the **old menu** (v1) available for customers who are not ready to switch. You tell customers which menu they are ordering from by putting the version in the URL (`/v1/users`), in a header (`API-Version: 2`), or in a query parameter (`?version=2`). The goal is to evolve your API while keeping existing clients working.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Version only for breaking changes** | Backward-compatible changes (new optional fields, new endpoints) should not require a new version. |
| **Version is part of the contract** | The version identifies the shape and behavior clients can depend on; it should not change for internal refactors. |
| **Multiple strategies exist** | URI path, header, query parameter, and media type versioning are all valid; the choice depends on clients, infrastructure, and cacheability. |
| **URI path is the most common** | GitHub, Twilio, Google, and Microsoft Graph all use path versioning as their primary strategy. |
| **Header versioning keeps URLs stable** | Stripe uses date-based header versioning with per-account pinning; URLs do not contain version numbers. |
| **Deprecation follows standardization** | RFC 9745 (`Deprecation`) and RFC 8594 (`Sunset`) provide standard headers for signaling end-of-life. |

---

### Prerequisites

- **HTTP fundamentals**: methods, status codes, headers, and the request/response model.
- **REST fundamentals**: resources, endpoints, representations, statelessness.
- **API design**: resource naming, URL structures, and error responses.
- **Familiarity with `curl` or Postman** to test versioned endpoints.

---

### Related Programming Areas

- **API design**: versioning is a cross-cutting concern of the interface contract.
- **REST fundamentals**: versioning is often discussed in the context of REST constraints.
- **Deprecation and lifecycle management**: versioning and retirement are two phases of the same lifecycle.
- **API gateways**: Azure API Management, Kong, and AWS API Gateway support multiple versioning schemes natively.
- **OpenAPI specification**: versioned APIs should have versioned OpenAPI documents.

---

### Core Concepts / Features

The following five sections cover the requested concepts. Each follows the uniform structure: definitions, purposes, syntax rules, annotated executable examples with expected outputs, and real-world cases.

---

## 1. URI Versioning

### Definitions

**Core Definition.** **URI versioning** (also called **path versioning**) embeds the version identifier directly in the **URL path**, so that each major contract occupies a distinct URI space.

**Technical Definition.** In path-based versioning, the version identifier is a mandatory segment of the request path. For example, `/v1/users` and `/v2/users` reference the same logical resource but different contracts. The request URL format is `https://{domain}/{apiName}/{versionIdentifier}/{operationId}`. Path versioning creates a **distinct URI for each version**, which means HTTP caches, load balancers, reverse proxies, and CDNs can route and cache versions natively without custom header configuration. The trade-off is that a resource’s URI changes when the version changes, which REST purists argue violates the principle that a resource’s identity should be stable.

**Beginner-Friendly Explanation.** URI versioning means **the version number is part of the web address**. You access the old API at `/v1/users` and the new API at `/v2/users`. This is the easiest versioning to understand and the most common in public APIs. The downside is that if you bookmark `/v1/users/42`, that link stops working when v1 is retired. But the upside is huge: every cache, proxy, and router already understands URLs, so versioning “just works” without special configuration.

### Purposes

- **To make the version immediately visible** in logs, documentation, caches, and routing rules.
- **To leverage native infrastructure support** — load balancers, API gateways, and CDNs route on URL paths without custom header handling.
- **To enable straightforward caching** — each version has a distinct URL, so caches do not need `Vary` headers or header-aware keys.
- **To provide the simplest developer experience** — testable by pasting the URL into a browser.

### Syntax Rules and Structure

**General syntax (URI path versioning).**

```
GET /v1/users HTTP/1.1
Host: api.example.com

GET /v2/users HTTP/1.1
Host: api.example.com
```

**Component breakdown.**

```
/v1/users/42
```

- `/v1` — the **version identifier** (mandatory path segment).
- `/users` — the resource collection.
- `/42` — the item identifier.

**Versioning strategy rules.**

| Rule | Guidance |
|---|---|
| **Version first after base path** | Place the version immediately after the domain: `/v1/…`. |
| **Use major versions only** | Google’s AIP-185 uses `v1`, never `v1.1` or `v1.4.2`, in the path. |
| **Keep version identifiers simple** | Use numeric (`v1`, `v2`) or date-based (`2026-01-01`) identifiers. |
| **Never change a version’s contract** | Once published, `/v1` must remain backward-compatible. |

**Constraints and limitations.**

- **URI changes break bookmarks and links** — a resource’s address changes when the version changes.
- **URL pollution** — REST purists argue that a resource’s identity should not change with the API version.
- **HATEOAS complexity** — links inside responses must include the version prefix.

### Annotated Complete Step by Step Code Examples

#### Example 1: Same resource, two versions

```http
# Step 1: Access v1 of the users collection.
GET /v1/users HTTP/1.1
Host: api.example.com

# Response (v1 — simple array):
# 200 OK
# [ { "id": 1, "name": "Alice" } ]

# Step 2: Access v2 of the same logical resource.
GET /v2/users HTTP/1.1
Host: api.example.com

# Response (v2 — envelope with metadata):
# 200 OK
# {
#   "data": [ { "id": 1, "name": "Alice", "email": "alice@example.com" } ],
#   "meta": { "total": 1 }
# }
```

**Expected Output (conceptual):**
```
v1 returns a simple array; v2 returns an envelope with additional fields.
The URI path distinguishes the contracts.
```

**Why this output occurs.** The version segment `/v1` or `/v2` routes the request to the corresponding contract implementation. The v2 response includes an `email` field and wraps the data in an envelope, which is a breaking change (the shape changed), so it requires a new version.

---

#### Example 2: Express routing for two versions

```javascript
// Step 1: Define a v1 router.
const v1Router = express.Router();
v1Router.get("/users", (req, res) => {
  const users = getUsers();
  res.json(users.map(u => ({ id: u.id, name: u.name })));
});

// Step 2: Define a v2 router with a different response shape.
const v2Router = express.Router();
v2Router.get("/users", (req, res) => {
  const users = getUsers();
  res.json({
    data: users.map(u => ({ id: u.id, name: u.name, email: u.email })),
    meta: { total: users.length },
  });
});

// Step 3: Mount the routers at versioned paths.
app.use("/api/v1", v1Router);
app.use("/api/v2", v2Router);
```

**Expected Output (conceptual):**
```
GET /api/v1/users → { id, name }
GET /api/v2/users → { data: [{ id, name, email }], meta: { total } }
```

**Why this output occurs.** The version is expressed in the mount path, and each router defines its own contract. The server can run both contracts simultaneously, allowing clients to migrate at their own pace.

---

### Real-World Cases

- **GitHub API**: `/v3/repos/octocat/hello-world` — version embedded in the path.
- **Google APIs (AIP-185)**: `v1` in the path, never minor versions.
- **Microsoft Graph**: `/v1.0/me` — path versioning with explicit major versions.

---

## 2. Header Versioning

### Definitions

**Core Definition.** **Header versioning** transmits the version identifier in a **custom HTTP request header** (e.g., `API-Version: 2`) or through **content negotiation** using the `Accept` header with a versioned media type.

**Technical Definition.** In header versioning, the resource path remains stable (`/users/123`), and the version is selected via a header. The server inspects the header and routes to the appropriate contract. Two variants exist: a **custom version header** (`API-Version: 2026-09-01`) and **media type versioning** (`Accept: application/vnd.example.v2+json`). Header versioning aligns more closely with REST’s principle that a resource’s identity should not change with its representation, but it requires every cache, gateway, client library, and observability tool to preserve and account for the version header. Without a `Vary` header, caches may serve responses from one version to clients requesting another.

**Beginner-Friendly Explanation.** Header versioning means **the version is sent in a separate envelope, not in the address**. You always call `/users/123`, but you add a header that says “I want version 2.” The URL stays clean, which some developers prefer. The catch is that you cannot just paste the URL into a browser — you need a tool like `curl` or Postman that can send custom headers. Also, every cache and proxy in the chain must be told to pay attention to the version header, or it might serve you the wrong version.

### Purposes

- **To keep URLs stable** across versions, preserving resource identity.
- **To align with content negotiation** when using media type versioning (`Accept: application/vnd.example.v1+json`).
- **To support fine-grained versioning** — different resources can have different versions without changing their paths.
- **To enable Stripe-style date-based pinning** — each account is pinned to the version it first called, and requests carry that version automatically.

### Syntax Rules and Structure

**General syntax (custom header versioning).**

```
GET /users/123 HTTP/1.1
Host: api.example.com
API-Version: 2
```

**General syntax (media type versioning).**

```
GET /users/123 HTTP/1.1
Host: api.example.com
Accept: application/vnd.example.v2+json
```

**Component breakdown.**

- `API-Version: 2` — a **custom header** carrying the version identifier.
- `Accept: application/vnd.example.v2+json` — a **versioned media type**; `vnd.example.v2` identifies the version, `json` identifies the format.
- `/users/123` — the **stable resource path** (no version segment).

**Header versioning rules.**

| Rule | Guidance |
|---|---|
| **Require the version header** | If the header is omitted, the server should use a documented default (e.g., v1). |
| **Use `Vary` for caches** | Include `Vary: API-Version` so caches key on the version header. |
| **Document the header name** | Use a consistent name (`API-Version`, `X-API-Version`, or a media type). |
| **Preserve the header in intermediaries** | Gateways, proxies, and CDNs must forward the header without stripping it. |

**Constraints and limitations.**

- **Harder to test in a browser** — you need `curl` or Postman to send custom headers.
- **Cache misconfiguration risk** — without `Vary`, caches may serve the wrong version.
- **Less discoverable** — the version is not visible in the URL, logs, or documentation by default.
- **Media type versioning adds complexity** — clients, documentation, and debugging tools must understand content negotiation.

### Annotated Complete Step by Step Code Examples

#### Example 1: Custom header versioning

```http
# Step 1: Request v1 using a custom header.
GET /customers/3 HTTP/1.1
Host: api.example.com
Custom-Header: api-version=1

# Response:
# 200 OK
# { "id": 3, "name": "Fabrikam, Inc.", "address": "1 Microsoft Way" }

# Step 2: Request v2 of the same resource.
GET /customers/3 HTTP/1.1
Host: api.example.com
Custom-Header: api-version=2

# Response:
# 200 OK
# {
#   "id": 3,
#   "name": "Fabrikam, Inc.",
#   "dateCreated": "2025-03-22T12:11:38Z",
#   "address": { "streetAddress": "1 Microsoft Way", "city": "Redmond" }
# }
```

**Expected Output (conceptual):**
```
Same URI (/customers/3), different response shapes selected by the header.
v1 returns a flat address string; v2 returns a structured address object.
```

**Why this output occurs.** The path identifies the resource; the header selects the contract. The server parses the header and returns the representation for the requested version. If the header is omitted, the server should fall back to a documented default (v1 in this example).

---

#### Example 2: Media type versioning

```http
# Step 1: Request v1 via Accept header.
GET /customers/3 HTTP/1.1
Host: api.example.com
Accept: application/vnd.contoso.v1+json

# Response:
# 200 OK
# Content-Type: application/vnd.contoso.v1+json
# { "id": 3, "name": "Fabrikam, Inc." }

# Step 2: Request v2 via a different media type.
GET /customers/3 HTTP/1.1
Host: api.example.com
Accept: application/vnd.contoso.v2+json

# Response:
# 200 OK
# Content-Type: application/vnd.contoso.v2+json
# { "id": 3, "name": "Fabrikam, Inc.", "dateCreated": "..." }
```

**Expected Output (conceptual):**
```
The Accept header selects both the version and the format.
The server responds with the corresponding Content-Type.
```

**Why this output occurs.** The media type `application/vnd.contoso.v1+json` encodes both the vendor (`contoso`), the version (`v1`), and the format (`json`). The server parses the `Accept` header and returns the matching representation, setting `Content-Type` to confirm the format.

---

### Real-World Cases

- **Stripe API**: date-based header versioning with per-account version pinning; no version in the URL path.
- **Azure API Management**: supports header versioning with a configurable header name (e.g., `Api-Version`).
- **Custom media type APIs**: `Accept: application/vnd.example.v2+json` — content negotiation for versioned representations.

---

## 3. Backward Compatibility

### Definitions

**Core Definition.** **Backward compatibility** is the property of a new API version or change that **allows existing clients to continue functioning without modification**. A backward-compatible change does not break the documented contract.

**Technical Definition.** A change is backward-compatible if it only **adds** elements (new optional fields, new endpoints, new optional parameters) or **fixes bugs** without changing externally observable behavior. It is **breaking** if it removes or renames a field clients use, changes a field’s type or meaning, makes an optional request field required, removes an endpoint, or changes authentication, pagination, error, or idempotency behavior promised by the contract. Strict clients may break when a response gains a field, even if the provider considers the change additive; consumer tests and telemetry are required to validate assumptions.

**Beginner-Friendly Explanation.** Backward compatibility means **your old clients keep working after you change the API**. If you add a new optional field to a response, old clients can ignore it — that is backward-compatible. If you rename `name` to `firstName`, old clients looking for `name` break — that is a breaking change. The golden rule: **add, don’t remove; extend, don’t rename**. When you must break something, you release a new version and keep the old one running until clients migrate.

### Purposes

- **To avoid forcing clients to upgrade immediately** when the API changes.
- **To reduce the operational burden** of maintaining many versions simultaneously.
- **To support gradual migration** — clients can move to the new version at their own pace.
- **To protect revenue and trust** — breaking clients without warning damages relationships.

### Syntax Rules and Structure

**Backward-compatible changes (no new version needed).**

| Change | Why It Is Safe |
|---|---|
| Add a new optional request field | Old clients omit it; the server uses a default. |
| Add a new response field | Old clients ignore unknown fields (if the contract permits). |
| Add a new endpoint | Old clients never call it. |
| Fix a bug without changing external behavior | The contract is unchanged. |
| Treat missing input as normal and use defaults | Old clients that omit fields continue to work. |

**Breaking changes (require a new version).**

| Change | Why It Breaks |
|---|---|
| Remove or rename a field clients use | Clients referencing the old name fail. |
| Change a field’s type or meaning | Clients parsing the old type fail. |
| Make an optional request field required | Old clients omitting it are rejected. |
| Remove an endpoint or operation | Old clients calling it get `404`. |
| Change authentication requirements | Old clients cannot authenticate. |
| Change pagination, error, or idempotency behavior | Clients relying on the old behavior fail. |

**Backward-compatibility rules.**

- **Ignore unknown request fields** so that additive client changes do not break the server.
- **Use sensible defaults for missing input** rather than rejecting it.
- **Never remove or rename a field in the same version** — add a new field and deprecate the old one.
- **Validate compatibility with consumer tests** — assumptions about “safe” additive changes may be wrong for strict clients.

**Constraints and limitations.**

- **Additive changes can still break strict clients** that reject unknown response fields.
- **Backward compatibility does not mean eternal support** — old versions must eventually be deprecated and retired.
- **Telemetry is essential** — without usage data, you cannot know whether a change is safe for real consumers.

### Annotated Complete Step by Step Code Examples

#### Example 1: Backward-compatible additive change

```http
# Step 1: v1 response (original contract).
GET /v1/users/42 HTTP/1.1

# Response:
# 200 OK
# { "id": 42, "name": "Alice" }

# Step 2: v1 response after adding an optional email field.
GET /v1/users/42 HTTP/1.1

# Response:
# 200 OK
# { "id": 42, "name": "Alice", "email": "alice@example.com" }
```

**Expected Output (conceptual):**
```
Old clients that only read id and name continue to work.
New clients can read email if they choose.
No new version is required because the change is additive.
```

**Why this output occurs.** Adding a response field is backward-compatible for clients that ignore unknown fields. The existing contract (`id`, `name`) remains intact. This is why additive changes do not require a version bump.

---

#### Example 2: Breaking change requiring a new version

```http
# ❌ WRONG: Breaking the v1 contract by renaming a field.
GET /v1/users/42 HTTP/1.1

# Old response: { "name": "Alice" }
# New response: { "firstName": "Alice", "lastName": "Smith" }
# Clients reading "name" now get undefined — BREAKING.

# ✅ CORRECT: Introduce v2 for the breaking change.
GET /v2/users/42 HTTP/1.1

# Response: { "firstName": "Alice", "lastName": "Smith" }

# v1 continues to return { "name": "Alice" } unchanged.
```

**Expected Output (conceptual):**
```
v1 clients keep working because v1 is untouched.
v2 clients get the new field structure.
The version boundary protects existing consumers.
```

**Why this output occurs.** Renaming `name` to `firstName` removes a field that v1 clients depend on. The safe approach is to keep v1 as-is and introduce v2 with the new structure, allowing clients to migrate on their own schedule.

---

### Real-World Cases

- **Google, Microsoft, and Stripe** all agree: **version only when you cannot make the change backward-compatible**.
- **Stripe API**: makes only additive changes and uses date-based version pinning per account, so existing integrations never see a breaking change without pinning a new version.
- **Mobile APIs**: old app versions in the wild cannot be updated instantly, so backward compatibility is essential for a defined migration window.

---

## 4. Deprecation Strategies

### Definitions

**Core Definition.** **Deprecation** is the process of formally announcing that an API version, endpoint, or field is **obsolete and scheduled for removal**, while continuing to support it for a defined period so clients can migrate.

**Technical Definition.** Deprecation is signaled through standard HTTP response headers: **`Deprecation`** (RFC 9745) indicates that the resource has been or will be deprecated, and **`Sunset`** (RFC 8594) indicates the date after which the resource will no longer be available. The `Deprecation` header value is a Structured Field Date (Unix timestamp prefixed with `@`, e.g., `@1688169599`), and the `Sunset` header uses an HTTP-date (e.g., `Sat, 31 Dec 2026 23:59:59 GMT`). A **`Link`** header with `rel="deprecation"` or `rel="successor-version"` points to documentation or the replacement resource. **Brownouts** — temporary, scheduled unavailability of the deprecated endpoint — can be used to force clients to notice the deprecation before the final sunset.

**Beginner-Friendly Explanation.** Deprecation is **telling clients that something is going away, and giving them time to move**. You do not just delete the old endpoint one day; you announce it, you add headers to every response saying “this is deprecated” and “it will be removed on this date,” and you point to the new endpoint. You might even temporarily turn the old endpoint off for a few minutes (a “brownout”) so clients that ignore the warnings finally notice. When the sunset date arrives, the old endpoint is removed, and requests return `410 Gone`.

### Purposes

- **To give clients a clear migration window** before removal.
- **To communicate the end-of-life plan in-band** — clients see the deprecation and sunset dates on every response without reading separate documentation.
- **To automate migration pressure** through brownouts and automated blocking after the sunset date.
- **To reduce the maintenance burden** of supporting old versions indefinitely.

### Syntax Rules and Structure

**General syntax (deprecation headers).**

```http
HTTP/1.1 200 OK
Deprecation: @1688169599
Sunset: Sat, 31 Dec 2026 23:59:59 GMT
Link: </v2/users>; rel="successor-version"
```

**Component breakdown.**

- `Deprecation: @1688169599` — **RFC 9745**; the Unix timestamp (`@` prefix) when the resource was or will be deprecated.
- `Sunset: Sat, 31 Dec 2026 23:59:59 GMT` — **RFC 8594**; the HTTP-date after which the resource will be removed.
- `Link: </v2/users>; rel="successor-version"` — points to the replacement resource.

**Deprecation lifecycle rules.**

| Phase | Action |
|---|---|
| **Announce** | Add `Deprecation` and `Sunset` headers; document the change. |
| **Warn** | Continue returning `200 OK` with the headers so clients see the warning. |
| **Brownout** | Temporarily return `410 Gone` or `301 Moved Permanently` during scheduled windows to test client readiness. |
| **Sunset** | After the `Sunset` date, return `410 Gone` (or `301` if an alternative exists). |

**Constraints and limitations.**

- **The `Sunset` date cannot be earlier than the `Deprecation` date**.
- **Clients should treat the `Sunset` date as a hint**, not a guarantee; servers may extend or shorten it.
- **Deprecation headers apply to the resource, not individual body members**; field-level deprecation requires a separate mechanism (e.g., a deprecation manifest).
- **Unsecured channels can allow intermediaries to forge or suppress deprecation headers**.

### Annotated Complete Step by Step Code Examples

#### Example 1: Adding deprecation headers to a v1 endpoint

```javascript
// Step 1: Middleware that adds deprecation headers to all /api/v1/* responses.
app.use("/api/v1/*", (req, res, next) => {
  res.set("Deprecation", "@1688169599");          // RFC 9745 (Unix timestamp)
  res.set("Sunset", "Sat, 31 Dec 2026 23:59:59 GMT"); // RFC 8594 (HTTP-date)
  res.set("Link", `</api/v2${req.path}>; rel="successor-version"`);
  next();
});

// Step 2: The v1 endpoint continues to return 200 OK during the warning phase.
app.get("/api/v1/users", (req, res) => {
  res.json([{ id: 1, name: "Alice" }]);
});
```

**Expected Response:**
```http
HTTP/1.1 200 OK
Deprecation: @1688169599
Sunset: Sat, 31 Dec 2026 23:59:59 GMT
Link: </api/v2/users>; rel="successor-version"

[ { "id": 1, "name": "Alice" } ]
```

**Why this output occurs.** The middleware adds the standard headers to every v1 response. Clients that inspect headers see the deprecation timestamp, the sunset date, and a link to the v2 replacement. The endpoint continues to work during the migration window.

---

#### Example 2: Automated blocking after sunset

```python
# FastAPI-style decorator with sunset enforcement.
@deprecated(
    deprecation_date="2024-01-01",
    sunset_date="2025-01-01",
    alternative="/v2/users",
)
@app.get("/v1/users")
async def list_users_v1():
    return [{"id": 1, "name": "Alice"}]
```

**Expected Response (before sunset):**
```http
200 OK
Deprecation: @1704067200
Sunset: Wed, 01 Jan 2025 00:00:00 GMT
Link: </v2/users>; rel="alternative"
```

**Expected Response (after sunset):**
```http
410 Gone
Content-Type: application/problem+json

{
  "type": "https://api.example.com/problems/gone",
  "title": "Endpoint removed",
  "status": 410,
  "detail": "This endpoint was sunset on 2025-01-01. Use /v2/users instead."
}
```

**Why this output occurs.** Before the sunset date, the endpoint returns `200 OK` with deprecation headers. After the sunset date, the framework automatically returns `410 Gone` (or `301 Moved Permanently` if an alternative is configured), enforcing the end-of-life plan without manual intervention.

---

### Real-World Cases

- **FastAPI Deprecation**: fully implements RFC 9745 and RFC 8594, with support for brownouts and automated blocking after sunset.
- **Azure API Management**: supports deprecation policies and revision-based lifecycle management.
- **HPE GreenLake**: returns the `Deprecation` header on requests for deprecated versions, containing the deprecation date per RFC 9745.

---

## 5. Versioning Best Practices

### Definitions

**Core Definition.** **Versioning best practices** are empirically established guidelines for choosing a versioning strategy, deciding when to version, and retiring versions safely, drawn from large-scale public APIs such as Google, Microsoft, Stripe, and GitHub.

**Technical Definition.** Best practices center on **minimizing version count** (version only for breaking changes), **choosing a strategy that matches infrastructure** (URI path for public APIs, header for stable-URL requirements), **enforcing backward compatibility** for additive changes, **using standard deprecation headers** (RFC 9745 and RFC 8594), **documenting the versioning policy**, and **monitoring version usage** to make informed retirement decisions. Google, Microsoft, and Stripe all agree on one principle: **version only when you cannot make the change backward-compatible**.

**Beginner-Friendly Explanation.** Best practices are **the habits that keep versioning from becoming a mess**. Use the fewest versions possible — most changes should be backward-compatible and not need a new version. Pick the strategy that your infrastructure and clients can actually support. Announce deprecations with standard headers and give clients time to migrate. Document everything. And watch your usage data so you know when it is safe to retire an old version.

### Purposes

- **To minimize the maintenance burden** of supporting multiple versions.
- **To choose a strategy that works with your caches, gateways, and clients**.
- **To protect existing clients** while still allowing the API to evolve.
- **To retire old versions predictably** using standard headers and telemetry.

### Syntax Rules and Structure

**Versioning strategy comparison.**

| Strategy | Client Visibility | Routing & Debugging | Cacheability | Best For |
|---|---|---|---|---|
| **URI Path** | High — in the URL | Straightforward — native URL routing | Excellent — distinct URLs | Public APIs, broad ecosystems |
| **Header** | Low — hidden in headers | Requires header-aware tooling | Requires `Vary` header | Stable URLs, internal APIs, Stripe-style pinning |
| **Query Parameter** | Medium — in the query string | Straightforward if preserved | Tricky — parameter ordering | Migration periods, not long-term |
| **Media Type** | Low — content negotiation | Complex content-type routing | Requires `Vary` header | Precise representation versioning, rare |

**Best-practice rules.**

| Rule | Guidance |
|---|---|
| **Version only for breaking changes** | Additive changes do not require a new version. |
| **Choose URI path for public APIs** | It is the most widely adopted and safest default. |
| **Use major versions only in the path** | Google uses `v1`, never `v1.1`. |
| **Document the default version** | If a version is omitted, clients should get a predictable default. |
| **Always send deprecation headers** | Use `Deprecation`, `Sunset`, and `Link` on old versions. |
| **Monitor version usage** | Use telemetry to know when a version can be retired safely. |
| **Never mutate a published contract** | Once `/v1` is live, it must remain backward-compatible. |

**Constraints and limitations.**

- **Query parameter versioning is widely considered an anti-pattern** for permanent versioning; use it only for migration periods.
- **Content negotiation is theoretically ideal but practically rare**; tooling and developer understanding lag behind.
- **Header versioning is RESTful but painful** — clean URLs come at the cost of testability and discoverability.

### Annotated Complete Step by Step Code Examples

#### Example 1: Versioning decision tree

```
Is the change backward-compatible?
  ├── Yes → Do NOT create a new version.
  │         (Add optional fields, new endpoints, fix bugs.)
  └── No  → Create a new version.
            ├── Public API? → Use URI path versioning (/v2/...).
            ├── Stable URL required? → Use header versioning (API-Version: 2).
            └── Add Deprecation + Sunset + Link headers to the old version.
```

**Expected Output (conceptual):**
```
Most changes → no version bump.
Breaking changes → new version + deprecation headers on the old version.
```

**Why this output occurs.** Versioning is expensive; the best strategy is to need fewer versions by investing in additive changes and strong deprecation policies. When a breaking change is unavoidable, the strategy follows from the client ecosystem and infrastructure constraints.

---

#### Example 2: Complete version lifecycle

```http
# Step 1: Release v2 with the breaking change.
GET /v2/users/42 HTTP/1.1
# 200 OK — new contract.

# Step 2: Add deprecation headers to v1.
GET /v1/users/42 HTTP/1.1
# 200 OK
# Deprecation: @1688169599
# Sunset: Sat, 31 Dec 2026 23:59:59 GMT
# Link: </v2/users/42>; rel="successor-version"

# Step 3: Monitor usage of v1 via telemetry.
# When v1 usage drops to an acceptable threshold, schedule a brownout.

# Step 4: Brownout — temporarily return 410 Gone during a scheduled window.
GET /v1/users/42 HTTP/1.1
# 410 Gone
# { "detail": "v1 will be removed on 2026-12-31. Migrate to /v2/users/42." }

# Step 5: After the sunset date, permanently return 410 Gone.
GET /v1/users/42 HTTP/1.1
# 410 Gone
```

**Expected Output (conceptual):**
```
v2 works immediately.
v1 works but warns of deprecation.
Brownout forces clients to notice the migration.
After sunset, v1 is permanently gone.
```

**Why this output occurs.** This follows the standard deprecation lifecycle: release the new version, announce deprecation with headers, use brownouts to test client readiness, and enforce the sunset date. The `Deprecation` and `Sunset` headers give clients a machine-readable migration window.

---

### Real-World Cases

- **GitHub, Stripe, Twilio, Google, and Microsoft** all use **URI path versioning** as their primary strategy.
- **Stripe** uses **date-based header versioning with per-account pinning**, representing a fifth approach: no URL versioning at all, with additive-only changes.
- **Google (AIP-185)** uses `v1` in the path and **never minor versions** (`v1.1`, `v1.4.2`).
- **Azure API Management** supports path, header, and query-string versioning, and provides revision management for non-breaking changes.

---

## References

- Azure API Management — Versions (Path, Header, Query String) - https://learn.microsoft.com/en-us/azure/api-management/api-management-versions
- API Versioning Strategies and Best Practices — API7.ai - https://api7.ai/learning-center/api-101/api-versioning
- API Versioning Strategies — llm-knowledge-base (GitHub) - https://raw.githubusercontent.com/glennguilloux/llm-knowledge-base/refs/heads/main/api-design/versioning.md
- Web API Design Best Practices — Azure Architecture Center - https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design
- FastAPI Deprecation — PyPI (RFC 9745 + RFC 8594) - https://pypi.org/project/fastapi-deprecation/
- A Deprecation Manifest for Field-Level Lifecycle Signalling — IETF - https://datatracker.ietf.org/doc/draft-rmili-httpapi-deprecation-manifest/
- API Versioning Strategies 2026 — apiscout.dev - https://apiscout.dev/guides/api-versioning-strategies-2026
- Google, Azure and Stripe version APIs three different ways — DEV Community - https://dev.to/freelance_inspector/google-azure-and-stripe-version-apis-three-different-ways-heres-what-they-agree-on-3mbg
- RFC 9745 — The Deprecation HTTP Response Header Field - https://datatracker.ietf.org/doc/rfc9745/
- RFC 8594 — The Sunset HTTP Header Field - https://datatracker.ietf.org/doc/rfc8594/
- Microsoft REST API Guidelines — Versioning - https://github.com/microsoft/api-guidelines
- Stripe API Versioning Policy - https://stripe.com/docs/api/versioning
- Google API Design Guide — Versioning (AIP-185) - https://google.aip.dev/185