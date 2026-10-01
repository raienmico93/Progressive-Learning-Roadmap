# API Evolution & Content Management — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** API evolution and content management encompasses the strategies, conventions, and lifecycle policies used to change an API over time without breaking existing consumers, including versioning strategies, backward compatibility rules, and standardized deprecation signaling.

**Technical Definition:** API evolution addresses the problem that a published API is a contract: the response shape, field names, and semantics that a client integrated against must continue working. When a breaking change becomes unavoidable, versioning provides a mechanism to serve the old contract to old clients and the new contract to new clients simultaneously. The three dominant versioning strategies are URI versioning (version in the path), header/media-type versioning (content negotiation via `Accept`), and query-parameter versioning (`?version=2`). The API lifecycle is managed through backward compatibility rules (breaking vs. non-breaking changes) and standardized deprecation signaling using the `Deprecation` header (RFC 9745) and `Sunset` header (RFC 8594).

**Beginner-Friendly Explanation:** An API is like a public promise. If you change how it works, everyone who built software on top of it might break. API evolution is the discipline of making changes safely: you add new things without removing old ones (backward compatibility), and when you must remove something, you version the API so old and new clients can coexist. Then you warn clients — using standard HTTP headers — that an endpoint is going away, giving them time to migrate.

### Key Characteristics

- **Contract as a promise:** A published API contract must be honored; breaking changes require a new version.
- **Three versioning strategies:** URI path (`/v1/users`), header/media type (`Accept: application/vnd.company.v1+json`), and query parameter (`?version=1`).
- **Backward compatibility first:** Postel's Law ("be conservative in what you send, liberal in what you accept") and the additive-only principle extend API life without versioning.
- **Breaking vs. non-breaking changes:** Removing/renaming fields, changing types, and altering response semantics are breaking; adding optional fields and new endpoints are non-breaking.
- **Standardized deprecation signaling:** `Deprecation` (RFC 9745) and `Sunset` (RFC 8594) headers provide machine-readable advance warning.
- **Lifecycle stages:** Experimental → GA → Deprecated → Sunset → Removed, with minimum notice periods.
- **Cache implications:** URI and query-parameter versioning are cache-friendly; header and media-type versioning complicate caching.

### Prerequisites

- **HTTP fundamentals:** Request methods, status codes, headers, and caching semantics.
- **REST architectural constraints:** Resource-oriented design and the Uniform Interface.
- **Basic understanding of API contracts:** What a published API promises to consumers.
- **Familiarity with a server framework:** Express, Fastify, or NestJS.
- **JSON/XML familiarity:** Understanding of representation formats.

### Related Programming Areas

- **Advanced API Design & Response Formatting:** Status codes, error handling, and OpenAPI.
- **Data Querying & Partial Updates:** CRUD operations and query parameters.
- **API Documentation & Contracts:** OpenAPI Specification and design-first workflows.
- **HTTP Caching:** `Cache-Control`, `ETag`, and `Vary` headers.
- **API Gateways:** Version routing, traffic splitting, and deprecation enforcement.

### Core Concepts

1. **Versioning Strategies** — URI versioning, header/media-type versioning, and query-parameter versioning.
2. **API Lifecycle & Deprecation** — backward compatibility rules and graceful sun-setting with HTTP headers.

---

## Core Concept 1: Versioning Strategies

### Sub-Feature 1.1: URI Versioning (Path-Based Tracking)

#### Definitions

**Core Definition:** URI versioning places the API version directly in the URL path (e.g., `/v1/users`, `/v2/users`), making the version explicit and immediately visible.

**Technical Definition:** URI versioning puts the version in the path (`/v1/notes`, `/v2/notes`). It is the most explicit, cacheable, and trivially routable approach — you can see the version in a log line or a browser bar. The same resource now has two URLs, which purists object to, but its operational clarity wins in practice. This is the most common versioning strategy, used by PayPal, Stripe (date-based), and most public APIs.

**Beginner-Friendly Explanation:** URI versioning is like having two separate filing cabinets labelled "Version 1" and "Version 2." When you need the old cabinet, you go to `v1`. When you need the new one, you go to `v2`. There's no confusion about which version you're using — it's right there in the address.

#### Purposes

- To make the API version immediately visible in logs, browser bars, and debugging tools.
- To enable trivial routing to version-specific handlers.
- To provide cache-friendly versioning (the same URL always returns the same version).
- To allow clients to explicitly pin to a version for stability.

#### Syntax Rules and Structure

```javascript
// Express: URI versioning
const express = require('express');
const app = express();

// V1 router
const v1Router = express.Router();
v1Router.get('/users', (req, res) => {
  res.json([{ id: 1, name: 'Alice' }]); // Simple list
});

// V2 router (new envelope format)
const v2Router = express.Router();
v2Router.get('/users', (req, res) => {
  res.json({
    data: [{ id: 1, name: 'Alice', email: 'alice@example.com' }],
    meta: { total: 1 },
  });
});

app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);

app.listen(3000);
```

| Aspect | Description |
|--------|-------------|
| Pattern | `/api/{version}/{resource}` |
| Pros | Clear, cacheable, easy to route, visible in logs |
| Cons | URL changes break bookmarks/links; URL pollution |
| Best for | Public APIs, long-lived APIs with multiple versions |

**Constraints and Limitations:**
- URL changes break bookmarks and hardcoded links.
- Multiple versions increase maintenance burden and code duplication.
- Some purists argue that the same resource should not have multiple URLs.

#### Annotated Code Example

```javascript
// uri-versioning.js
const express = require('express');
const app = express();

// Shared data layer
const users = [{ id: 1, name: 'Alice', email: 'alice@example.com' }];

// V1: simple list (no envelope)
app.get('/api/v1/users', (req, res) => {
  res.json(users.map(u => ({ id: u.id, name: u.name })));
});

// V2: envelope format with metadata
app.get('/api/v2/users', (req, res) => {
  res.json({
    data: users,
    meta: { total: users.length, version: 2 },
  });
});

app.listen(3000);
```

**Expected Output (for `GET /api/v1/users`):**
```json
[{"id":1,"name":"Alice"}]
```

**Expected Output (for `GET /api/v2/users`):**
```json
{"data":[{"id":1,"name":"Alice","email":"alice@example.com"}],"meta":{"total":1,"version":2}}
```

**Why this output:** The V1 endpoint returns the original response shape (simple list, no email). The V2 endpoint returns the new envelope format with the additional `email` field. Both versions coexist, allowing old clients to continue using V1 while new clients adopt V2.

#### Real-World Cases

- **Stripe:** Date-based URI versioning (`/v1/charges` with `Stripe-Version` header).
- **PayPal:** URL path versioning (`/v1/payments`).
- **GitHub:** `/repos/{owner}/{repo}/issues` with no version in path (uses `Accept` header).

---

### Sub-Feature 1.2: Header/Media Type Versioning (Content Negotiation)

#### Definitions

**Core Definition:** Header versioning carries the API version in a custom HTTP header (e.g., `X-API-Version: 2`) or in the `Accept` media type (e.g., `Accept: application/vnd.company.v1+json`), keeping URLs clean and treating version as representation metadata.

**Technical Definition:** The client sends an `Accept` header with a custom MIME type (e.g., `application/vnd.contoso.v1+json`) to indicate the desired version. The server processes the `Accept` header and returns the appropriate representation. If the `Accept` header does not specify any known media type, the server may return a 406 Not Acceptable response or a default media type. This approach is considered more RESTful because the same resource URI can serve multiple representations.

**Beginner-Friendly Explanation:** Header versioning is like ordering coffee in different sizes. The address of the coffee shop is the same (`/users`), but you say "I'd like a small" or "I'd like a large" using a separate instruction (the `Accept` header). The shop serves the same coffee, just in different formats. This keeps the URL clean but makes it harder to test in a browser.

#### Purposes

- To keep URLs clean and resource-oriented.
- To treat version as metadata about the representation, not the resource identity.
- To enable content negotiation for multiple formats and versions simultaneously.
- To support HATEOAS by including the MIME type of related data in links.

#### Syntax Rules and Structure

```javascript
// Express: Header/media-type versioning
app.get('/api/users', (req, res) => {
  const accept = req.headers['accept'] || '';

  if (accept.includes('application/vnd.myapp.v2+json')) {
    res.type('application/vnd.myapp.v2+json');
    return res.json({
      data: [{ id: 1, name: 'Alice', email: 'alice@example.com' }],
      meta: { total: 1 },
    });
  }

  if (accept.includes('application/vnd.myapp.v1+json')) {
    res.type('application/vnd.myapp.v1+json');
    return res.json([{ id: 1, name: 'Alice' }]);
  }

  // Default to V1
  res.type('application/vnd.myapp.v1+json');
  res.json([{ id: 1, name: 'Alice' }]);
});
```

| Header | Example Value | Description |
|--------|---------------|-------------|
| `Accept` | `application/vnd.company.v1+json` | Custom media type with version. |
| `X-API-Version` | `2` | Custom header for version. |
| `Accept-Version` | `2` | Alternative custom header. |

**Constraints and Limitations:**
- Invisible in access logs unless logging is configured to capture headers.
- Harder to test in a browser (cannot set `Accept` header easily).
- Complicates caching because cache layers may not consider the `Accept` header.
- Some proxies and intermediaries strip custom headers.

#### Annotated Code Example

```javascript
// header-versioning.js
const express = require('express');
const app = express();

app.get('/api/users', (req, res) => {
  const accept = req.headers['accept'] || 'application/vnd.myapp.v1+json';

  // V2: envelope format
  if (accept.includes('vnd.myapp.v2+json')) {
    res.set('Content-Type', 'application/vnd.myapp.v2+json; charset=utf-8');
    return res.json({
      data: [{ id: 1, name: 'Alice', email: 'alice@example.com' }],
      meta: { total: 1 },
    });
  }

  // V1: simple list (default)
  res.set('Content-Type', 'application/vnd.myapp.v1+json; charset=utf-8');
  res.json([{ id: 1, name: 'Alice' }]);
});

app.listen(3000);
```

**Expected Output (for `Accept: application/vnd.myapp.v1+json`):**
```json
[{"id":1,"name":"Alice"}]
```

**Expected Output (for `Accept: application/vnd.myapp.v2+json`):**
```json
{"data":[{"id":1,"name":"Alice","email":"alice@example.com"}],"meta":{"total":1}}
```

**Expected Output (for `Accept: application/xml`):**
```json
[{"id":1,"name":"Alice"}]
```
*(Falls back to default V1.)*

**Why this output:** The server inspects the `Accept` header and selects the appropriate representation. V1 and V2 are served from the same URI. Unknown media types fall back to the default version. This demonstrates content negotiation in action.

#### Real-World Cases

- **GitHub API:** Uses `Accept: application/vnd.github.v3+json` for versioning.
- **FHIR (Healthcare):** Uses `Accept: application/fhir+json`.
- **Microsoft Azure:** Recommends media-type versioning for HATEOAS compatibility.

---

### Sub-Feature 1.3: Query Parameter Versioning

#### Definitions

**Core Definition:** Query parameter versioning specifies the API version in a URL query parameter (e.g., `/api/users?version=2`), keeping the path clean but introducing caching and discoverability challenges.

**Technical Definition:** Query-parameter versioning (`/notes?version=2`) is the weakest strategy: it is easy to omit, awkward to cache, and easily lost when URLs are copied. The API version is in the URL again (as a query parameter), but without the clear visibility of path-based versioning. Caching systems may not distinguish between versions because the URI is the same without the query parameter, and the `Vary` header is required to inform caches.

**Beginner-Friendly Explanation:** Query parameter versioning is like writing a note on a sticky pad that says "use version 2" and attaching it to a letter. It's easy to forget the sticky note, and if someone photocopies the letter without the note, the version information is lost. It works for internal APIs where all clients are under your control, but it's fragile for public APIs.

#### Purposes

- To keep the URL path clean and unchanged between versions.
- To allow optional version specification (defaulting to latest or a stable version).
- To provide a simple versioning mechanism for internal APIs.
- To enable version selection without changing the resource path.

#### Syntax Rules and Structure

```javascript
// Express: Query parameter versioning
app.get('/api/users', (req, res) => {
  const version = req.query.version || '1';

  if (version === '2') {
    return res.json({
      data: [{ id: 1, name: 'Alice', email: 'alice@example.com' }],
      meta: { total: 1 },
    });
  }

  res.json([{ id: 1, name: 'Alice' }]);
});
```

| Aspect | Description |
|--------|-------------|
| Pattern | `/api/users?version=2` |
| Pros | Easy to test, explicit, clean path |
| Cons | Caching issues, easily lost, not RESTful |
| Best for | Internal APIs, version changes rarely |

**Constraints and Limitations:**
- Caching systems may not distinguish between versions; `Vary: version` header is required but often ignored.
- Query parameters can be confused with business parameters.
- URLs copied without the query parameter lose version information.
- Not considered RESTful because the same resource has multiple URLs.

#### Annotated Code Example

```javascript
// query-versioning.js
const express = require('express');
const app = express();

app.get('/api/users', (req, res) => {
  const version = req.query.version || '1';

  // Set Vary header to inform caches that version matters
  res.set('Vary', 'version');

  if (version === '2') {
    return res.json({
      data: [{ id: 1, name: 'Alice', email: 'alice@example.com' }],
      meta: { total: 1 },
    });
  }

  res.json([{ id: 1, name: 'Alice' }]);
});

app.listen(3000);
```

**Expected Output (for `GET /api/users?version=1`):**
```json
[{"id":1,"name":"Alice"}]
```

**Expected Output (for `GET /api/users?version=2`):**
```json
{"data":[{"id":1,"name":"Alice","email":"alice@example.com"}],"meta":{"total":1}}
```

**Why this output:** The `version` query parameter selects the response format. The `Vary: version` header tells caches that the response depends on the `version` parameter. Without this header, caches might serve the wrong version to clients.

#### Real-World Cases

- **Internal APIs:** Microservice communication where all clients are controlled.
- **AWS API Gateway:** Supports query-parameter versioning as a routing option.
- **IBM API Guidelines:** Recommends a date-based query parameter for minor versions.

---

## Core Concept 2: API Lifecycle & Deprecation

### Sub-Feature 2.1: Backward Compatibility Rules (Breaking vs. Non-Breaking Changes)

#### Definitions

**Core Definition:** A **breaking change** is a backward-incompatible modification that requires existing clients to update their code; a **non-breaking change** is an additive or non-disruptive modification that existing clients can safely ignore.

**Technical Definition:** A breaking change is any change that requires either consumers or implementers to modify their code for it to continue to function correctly. A non-breaking change is typically a new addition to the API that can be implemented at the client's own pace and choosing. Postel's Law (the Robustness Principle) — "be conservative in what you send and liberal in what you accept" — and the additive-only principle ("you MUST NOT take anything away") extend API life without versioning.

**Beginner-Friendly Explanation:** A non-breaking change is like adding a new dish to a restaurant menu — existing customers can still order what they always ordered. A breaking change is like removing the restaurant's most popular dish or renaming it without telling anyone. Non-breaking changes should be made freely; breaking changes require a new version and a migration plan.

#### Purposes

- To maximise the lifespan of a version without forced migrations.
- To guide the decision of whether a new version is required.
- To provide clear, documented rules for API evolution.
- To reduce the maintenance burden of multiple concurrent versions.

#### Syntax Rules and Structure

| Change Type | Breaking? | Example |
|-------------|-----------|---------|
| Adding a new endpoint | ❌ Non-breaking | `POST /v1/reports` added. |
| Adding an optional request parameter | ❌ Non-breaking | New `?format=json` option. |
| Adding a field to a response | ❌ Non-breaking | New `email` field returned. |
| Changing an error message | ❌ Non-breaking | "Invalid input" → "Email format invalid". |
| Removing an endpoint | ✅ Breaking | `DELETE /v1/legacy` removed. |
| Removing a response field | ✅ Breaking | `email` field no longer returned. |
| Renaming a field | ✅ Breaking | `name` → `fullName`. |
| Changing a field's type | ✅ Breaking | `id: string` → `id: integer`. |
| Making an optional parameter required | ✅ Breaking | `?format` now mandatory. |
| Changing authorization rules | ✅ Breaking | Stricter permissions enforced. |

**Constraints and Limitations:**
- Postel's Law is a guideline, not a guarantee — some clients make unwarranted assumptions (e.g., order of JSON fields).
- Even additive changes can break clients that use strict schema validation.
- The Principle of Least Astonishment: anything that violates predictable, consistent behaviour is a breaking change.

#### Annotated Code Example

```javascript
// backward-compatibility.js
const express = require('express');
const app = express();

const users = [
  { id: 1, name: 'Alice', email: 'alice@example.com', createdAt: '2026-01-01' },
];

// V1 endpoint — additive changes only
app.get('/api/v1/users', (req, res) => {
  // Non-breaking: we can add new fields, but never remove or rename existing ones
  res.json(users.map(u => ({
    id: u.id,
    name: u.name,
    // email: u.email,           // ❌ Removing this would break clients
    // createdAt: u.createdAt,   // ✅ Adding this is non-breaking
  })));
});

// If a breaking change is needed, create a new version
app.get('/api/v2/users', (req, res) => {
  res.json({
    data: users.map(u => ({
      id: u.id,
      fullName: u.name,        // Renamed field
      emailAddress: u.email,   // Renamed field
      createdAt: u.createdAt,  // New field
    })),
    meta: { total: users.length },
  });
});

app.listen(3000);
```

**Expected Output (for `GET /api/v1/users`):**
```json
[{"id":1,"name":"Alice"}]
```

**Expected Output (for `GET /api/v2/users`):**
```json
{"data":[{"id":1,"fullName":"Alice","emailAddress":"alice@example.com","createdAt":"2026-01-01"}],"meta":{"total":1}}
```

**Why this output:** The V1 endpoint preserves the original field names (`name`) and does not include the new `email` field (which was never in V1). The V2 endpoint introduces the renamed fields (`fullName`, `emailAddress`) and the new `createdAt` field. This demonstrates the additive-only principle: V1 only gains fields, never loses them.

#### Real-World Cases

- **Stripe:** Maintains backward compatibility for years, deprecating only after long notice.
- **GitHub:** Uses API versioning with a 24-month deprecation window.
- **Microsoft Graph:** Deprecates versions with at least 24 months' notice.

---

### Sub-Feature 2.2: Gracefully Sun-Setting Features Using HTTP Headers (Sunset, Deprecation)

#### Definitions

**Core Definition:** The `Deprecation` header (RFC 9745) signals that a resource will be or has been deprecated; the `Sunset` header (RFC 8594) communicates the date when a URI is likely to become unresponsive, enabling machine-readable advance warning for automated migration.

**Technical Definition:** The `Deprecation` HTTP response header field is used to signal to consumers of a resource that the resource will be or has been deprecated. Its value is a date (Unix timestamp or HTTP date) in the past or future. The `Sunset` HTTP response header field indicates that a URI is likely to become unresponsive at a specified point in the future. The timestamp given in the `Sunset` header MUST NOT be earlier than the one given in the `Deprecation` header. A `Link` header with `rel="deprecation"` or `rel="successor-version"` points to migration documentation.

**Beginner-Friendly Explanation:** Deprecation headers are like a "going out of business" sale sign. The `Deprecation` header says "this endpoint is deprecated" (the sale has started). The `Sunset` header says "this endpoint will stop working on June 1, 2026" (the closing date). The `Link` header says "here's where to find the new endpoint" (the new store location). Together, they give clients a machine-readable warning and a migration path.

#### Purposes

- To provide machine-readable advance warning to clients and automated tools.
- To enable client SDKs to log warnings or raise alerts on deprecation.
- To track migration deadlines programmatically.
- To close the loop with a direct path to migration instructions.

#### Syntax Rules and Structure

```javascript
// Express: Deprecation and Sunset headers middleware
function deprecate({ deprecatedOn, sunsetOn, migrationUrl }) {
  return (req, res, next) => {
    // Deprecation header: Unix timestamp (RFC 9745)
    res.set('Deprecation', `@${Math.floor(deprecatedOn.getTime() / 1000)}`);

    // Sunset header: HTTP date (RFC 8594)
    res.set('Sunset', sunsetOn.toUTCString());

    // Link header with deprecation relation
    res.set('Link', `<${migrationUrl}>; rel="deprecation"`);

    next();
  };
}

// Apply to deprecated V1 routes
app.use('/api/v1', deprecate({
  deprecatedOn: new Date('2026-01-15T00:00:00Z'),
  sunsetOn: new Date('2026-06-01T00:00:00Z'),
  migrationUrl: 'https://api.example.com/docs/migration/v1-to-v2',
}));
```

| Header | RFC | Format | Example |
|--------|-----|--------|---------|
| `Deprecation` | RFC 9745 | Unix timestamp (`@` prefix) or HTTP date | `@1736899200` |
| `Sunset` | RFC 8594 | HTTP date (IMF-fixdate) | `Mon, 01 Jun 2026 00:00:00 GMT` |
| `Link` | RFC 8288 | URI with relation type | `<https://api.example.com/docs/migration>; rel="deprecation"` |

**Constraints and Limitations:**
- The `Sunset` date MUST NOT be earlier than the `Deprecation` date.
- RFC 8594 is informational, not a standards-track specification.
- Headers alone are not enough — documentation and migration guides are essential.
- Some clients and proxies may strip or ignore custom headers.

#### Annotated Code Example

```javascript
// deprecation-headers.js
const express = require('express');
const app = express();

function deprecate({ deprecatedOn, sunsetOn, migrationUrl, successorVersion }) {
  return (req, res, next) => {
    res.set('Deprecation', `@${Math.floor(deprecatedOn.getTime() / 1000)}`);
    res.set('Sunset', sunsetOn.toUTCString());

    const links = [`<${migrationUrl}>; rel="deprecation"`];
    if (successorVersion) {
      links.push(`<${successorVersion}>; rel="successor-version"`);
    }
    res.set('Link', links.join(', '));

    next();
  };
}

// V1 is deprecated; V2 is the successor
app.use('/api/v1', deprecate({
  deprecatedOn:      new Date('2026-01-15T00:00:00Z'),
  sunsetOn:          new Date('2026-06-01T00:00:00Z'),
  migrationUrl:     'https://api.example.com/docs/migration/v1-to-v2',
  successorVersion: 'https://api.example.com/api/v2',
}));

app.get('/api/v1/users', (req, res) => {
  res.json([{ id: 1, name: 'Alice' }]);
});

app.get('/api/v2/users', (req, res) => {
  res.json({ data: [{ id: 1, name: 'Alice', email: 'alice@example.com' }] });
});

app.listen(3000);
```

**Expected Output (for `GET /api/v1/users`):**
```http
HTTP/1.1 200 OK
Deprecation: @1736899200
Sunset: Mon, 01 Jun 2026 00:00:00 GMT
Link: <https://api.example.com/docs/migration/v1-to-v2>; rel="deprecation", <https://api.example.com/api/v2>; rel="successor-version"
Content-Type: application/json

[{"id":1,"name":"Alice"}]
```

**Expected Output (for `GET /api/v2/users`):**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{"data":[{"id":1,"name":"Alice","email":"alice@example.com"}]}
```

**Why this output:** The V1 endpoint includes the `Deprecation`, `Sunset`, and `Link` headers, warning clients that the endpoint is deprecated and will be removed on June 1, 2026. The `Link` header provides migration documentation and the successor version URL. The V2 endpoint does not include these headers because it is the current version. Automated tools can parse these headers and log warnings or alert on the sunset date.

#### Real-World Cases

- **Stripe:** Uses `Sunset` headers on deprecated API versions.
- **GitHub:** Uses `Deprecation` and `Sunset` headers on deprecated endpoints.
- **Microsoft Graph:** Uses `Deprecation` headers with migration links.

---

## References

- RFC 8594 — The Sunset HTTP Header Field — https://datatracker.ietf.org/doc/html/rfc8594
- RFC 9745 — The Deprecation HTTP Response Header Field — https://www.rfc-editor.org/rfc/rfc9745
- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- RFC 8288 — Web Linking — https://www.rfc-editor.org/rfc/rfc8288
- Microsoft Azure Architecture Center — Web API Design Best Practices — https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design
- Google Cloud — Handling API Versioning — https://docs.cloud.google.com/endpoints/docs/frameworks/versioning
- Google Cloud — Versioning an API — https://docs.cloud.google.com/endpoints/docs/openapi/versioning-an-api
- CMU — API Usability Style Guides (PLATEAU 2017) — https://www.cs.cmu.edu/~NatProg/papers/API-Usability-Styleguides-PLATEAU2017.pdf
- DreamFactory — Ultimate Guide to Microservices API Versioning — https://blog.dreamfactory.com/ultimate-guide-to-microservices-api-versioning
- DEV Community — Sunset Your API Endpoints on Purpose: The Deprecation and Sunset Headers — https://dev.to/sunset-api-endpoints
- Cleverence — How and When to Deprecate APIs: Timelines, Versioning, Sunset Headers, and Migrations — https://www.cleverence.com/api-deprecation
- Pipedrive — Breaking vs. Non-Breaking Changes — https://pipedrive.readme.io/docs/breaking-vs-non-breaking-changes
- Appcharge — API Versioning Policy — https://docs.appcharge.com/api-versioning
- Tealium — Developer Portal API versioning — https://docs.tealium.com/api-versioning
- HubSpot — Breaking vs. Non-Breaking Changes — https://developers.hubspot.com/docs/api/breaking-changes
- OpenRouter — API Versioning Policy — https://openrouter.ai/docs/api-versioning
- Proactis — Change Policy — https://docs.proactis.com/change-policy
- Instellix — API Lifecycle Management — https://docs.instellix.io/api-lifecycle
- Tealium — API Lifecycle — https://docs.tealium.com/api-lifecycle
- ZPEDU — API版本控制怎么做？URL版本与Header版本策略 — https://www.zpedu.com/api-versioning