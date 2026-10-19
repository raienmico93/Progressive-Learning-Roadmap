# Express.js Route Versioning — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Route versioning is the practice of exposing multiple generations of an API's routes simultaneously, allowing clients to continue using older behavior while newer clients adopt breaking changes.

**Technical Definition**
In Express.js, route versioning is implemented by isolating version-specific routers, controllers, or logic behind a version-selection mechanism. The selection can occur through the request URI (`/v1/users`), a custom request header (`API-Version: 1`), a query parameter (`?version=1`), or HTTP content negotiation via the `Accept` header (`application/vnd.api.v1+json`). Each approach determines how the server identifies the requested version and dispatches to the appropriate handler.

**Beginner-Friendly Explanation**
Route versioning lets your API change without breaking existing apps. When you need to make a big change—like renaming a field or changing the response shape—you release it as a new version. Old clients keep using `/v1`, and new clients use `/v2`. You decide how the version is communicated: in the URL, in a header, in a query string, or through content negotiation.

---

### Key Characteristics

- **Breaking-change isolation** — Versioning is primarily for **breaking** changes; additive, backward-compatible changes (new optional fields, new endpoints) generally do not require a version bump.
- **Multiple simultaneous versions** — At least two versions should be supported during migration windows.
- **Strategy trade-offs** — URI versioning prioritizes discoverability and cache-friendliness; header and media type versioning prioritize clean URLs at the cost of visibility and caching complexity.
- **Deprecation signaling** — Mature versioning includes `Deprecation` and `Sunset` headers to communicate end-of-life timelines to clients.

---

### Prerequisites

- Node.js and npm installed; Express 5 requires Node.js 18 or higher.
- An Express application with `express.Router()` and middleware familiarity.
- Understanding of HTTP headers (`Accept`, custom headers) and status codes.
- (Optional) Semantic versioning concepts for granular version negotiation.

---

### Related Programming Areas

- **Routing** — Versioning is layered on top of route definitions and router mounting.
- **Content negotiation** — Media type versioning uses HTTP `Accept` header semantics.
- **API design** — Versioning strategy affects client contracts, documentation, and deprecation policy.
- **Caching** — Header-based strategies require `Vary` headers to prevent cache poisoning.
- **Middleware** — Version extraction is typically implemented as Express middleware.

---

## Core Concept 1: URI Versioning

### Definitions

**Core Definition**
URI versioning embeds the version identifier directly in the URL path, so different versions are accessed at distinct URLs such as `/v1/users` and `/v2/users`.

**Technical Definition**
In Express.js, URI versioning is implemented by mounting version-specific routers at version-prefixed paths: `app.use('/api/v1', v1Router)` and `app.use('/api/v2', v2Router)`. Each router contains the routes and handlers for that version. Because different versions occupy different URLs, caching, load-balancer routing, and API gateway rules operate naturally on the version segment.

**Beginner-Friendly Explanation**
URI versioning is the most visible approach: you literally put the version in the web address. Clients call `/api/v1/users` or `/api/v2/users`. It is easy to understand, easy to test in a browser, and works with every HTTP client.

---

### Purposes

- To make the API version explicitly visible and discoverable in every request URL.
- To enable cache-friendly version isolation, since different URLs produce different cache keys.
- To allow infrastructure-level routing (load balancers, gateways) to direct versions independently.
- To provide the simplest migration path for third-party developers who can see the version in documentation and examples.

---

### Syntax Rules and Structure

#### Complete General Syntax: Mounting Versioned Routers

```javascript
app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `'/api/v1'` | Version-specific mount prefix. |
| `v1Router` | Router containing all v1 route definitions. |
| `v2Router` | Router containing all v2 route definitions. |

#### Syntax Rules

- Each version must have its own router or route module; do not conditionally branch inside a single handler for URI versioning.
- Mount order is not semantically significant for distinct version prefixes, but consistency is recommended.
- Version segments should be treated as part of the public contract; changing them breaks clients.

#### Constraints and Limitations

- URL proliferation occurs as versions accumulate.
- Clients must update their base URLs when migrating.
- URI versioning is **not** a substitute for backward-compatible evolution; only breaking changes justify a new URI version.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Versioned Routers for User Endpoints

```javascript
// routes/v1/users.js
const express = require('express');
const router = express.Router();
router.get('/users', (req, res) => {
  res.json({ version: 'v1', users: [{ id: 1, name: 'John' }] });
});
module.exports = router;

// routes/v2/users.js
const express = require('express');
const router = express.Router();
router.get('/users', (req, res) => {
  res.json({ version: 'v2', users: [{ id: 1, name: 'John', email: 'john@example.com' }] });
});
module.exports = router;

// app.js
const express = require('express');
const app = express();
app.use('/api/v1', require('./routes/v1/users'));
app.use('/api/v2', require('./routes/v2/users'));
app.listen(3000);
```

**Step-by-Step Explanation**

1. **Version folders** — `routes/v1` and `routes/v2` isolate version-specific logic.
2. **Router creation** — Each version exports its own `express.Router()`.
3. **Mounting** — `app.use()` binds each router to its versioned prefix.

**Expected Output**

```
GET /api/v1/users
→ 200 { "version": "v1", "users": [{ "id": 1, "name": "John" }] }

GET /api/v2/users
→ 200 { "version": "v2", "users": [{ "id": 1, "name": "John", "email": "john@example.com" }] }
```

**Why This Output Occurs**

- Each version has a distinct router and handler; the URL prefix determines which handler executes.

---

### Real-World Cases

- **Public API platforms** — Stripe, GitHub, and Twitter historically use URI versioning for discoverability.
- **Third-party developer APIs** — Visible version numbers simplify documentation and manual testing.
- **Cache-sensitive APIs** — Distinct URLs produce distinct cache entries without `Vary` header complexity.

---

## Core Concept 2: Header Versioning

### Definitions

**Core Definition**
Header versioning transmits the desired API version in a custom HTTP request header (such as `API-Version` or `Accept-Version`), keeping the URL path constant across versions.

**Technical Definition**
Header versioning extracts the version from a request header and uses it to dispatch to the appropriate handler. The URL remains `/api/users` for all versions. Middleware parses the header, stores the version on the request object (e.g., `req.apiVersion`), and either selects a version-specific handler or sets a routing parameter. The `Vary` response header must include the version header to ensure caches do not serve the wrong version to a client. The `express-route-versioning-middleware` package implements this pattern with an `accept-version` header and a version-to-function map.

**Beginner-Friendly Explanation**
Header versioning keeps the URL clean—`/api/users` is always the same—and puts the version in a custom header like `API-Version: 2`. It looks RESTful and avoids URL clutter, but it is harder to test in a browser and requires careful cache configuration.

---

### Purposes

- To keep URLs clean, stable, and free of version segments.
- To allow multiple versions to be served from the **same** URL.
- To align with RESTful content-negotiation principles without changing resource identifiers.
- To support internal APIs where clients can easily set headers programmatically.

---

### Syntax Rules and Structure

#### Complete General Syntax: Header-Based Version Selection

```javascript
const parseVersion = (req, res, next) => {
  req.apiVersion = req.headers['api-version'] || '1';
  res.setHeader('Vary', 'API-Version'); // critical for caching
  next();
};

app.get('/api/users', parseVersion, (req, res) => {
  if (req.apiVersion === '2') return getUsersV2(req, res);
  return getUsersV1(req, res);
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `req.headers['api-version']` | Reads the version header (case-insensitive). |
| `req.apiVersion` | Stores the normalized version for downstream use. |
| `Vary: API-Version` | Prevents caches from serving mismatched versions. |

#### Syntax Rules

- Header names are case-insensitive in HTTP, but convention favors consistent casing (`API-Version`).
- The `Vary` header **must** include the version header when responses differ by version.
- A default version should be applied when the header is absent, for backward compatibility.

#### Constraints and Limitations

- Less visible than URI versioning; clients and documentation must explicitly mention the header.
- Harder to test manually in a browser; requires `curl` or an API client.
- Caching complexity increases; missing `Vary` headers can cause cross-version cache poisoning.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Version Map with `express-route-versioning-middleware`

```bash
npm install express-route-versioning-middleware
```

```javascript
const express = require('express');
const { routeVersionHandler } = require('express-route-versioning-middleware');
const app = express();
const router = express.Router();

function getUsersV1(req, res) {
  res.json({ version: '0.0.0', users: [{ id: 1, name: 'John' }] });
}

function getUsersV2(req, res) {
  res.json({ version: '0.1.0', users: [{ id: 1, name: 'John', email: 'john@example.com' }] });
}

function getUsersV3(req, res) {
  res.json({ version: '1.2.3', users: [{ id: 1, name: 'John', email: 'john@example.com', active: true }] });
}

router.get('/users', routeVersionHandler({
  '0.0.0': getUsersV1,
  '0.1.0': getUsersV2,
  '1.2.3': getUsersV3
}));

app.use(router);
app.listen(3000);
```

**Step-by-Step Explanation**

1. **Version map** — Keys are minimum version thresholds; values are handlers.
2. **Header parsing** — The middleware reads `accept-version` from the request headers.
3. **Fallback logic** — The highest version whose threshold is `<=` the requested version is selected.

**Expected Output**

```
GET /users  (accept-version: 0.0.0)   → 200 { "version": "0.0.0", ... }
GET /users  (accept-version: 0.0.999) → 200 { "version": "0.0.0", ... }   (highest ≤ 0.0.999)
GET /users  (accept-version: 0.1.5)   → 200 { "version": "0.1.0", ... }
GET /users  (accept-version: 1.2.3)   → 200 { "version": "1.2.3", ... }
```

**Why This Output Occurs**

- The middleware selects the highest version threshold that the client’s version satisfies, providing graceful fallback.

---

### Real-World Cases

- **Internal microservice APIs** — Services can set headers easily; URL cleanliness is prioritized.
- **Hypermedia APIs** — Header-based negotiation aligns with HATEOAS principles.
- **APIs behind CDNs** — `Vary` configuration is feasible and well understood.

---

## Core Concept 3: Query Versioning

### Definitions

**Core Definition**
Query versioning specifies the desired API version as a query parameter, such as `/api/users?version=2`, while keeping the path constant.

**Technical Definition**
In Express.js, query versioning reads the version from `req.query.version`, applies a default when absent, and branches to the appropriate handler or logic. It is the simplest strategy to implement because it requires no custom header parsing and no additional routing middleware. However, it conflates version selection with resource addressing and is generally discouraged for public APIs because query parameters are meant for filtering and pagination rather than protocol negotiation.

**Beginner-Friendly Explanation**
Query versioning puts the version in the URL after a question mark: `/api/users?version=2`. It is easy to try in a browser and easy to implement, but it mixes two different concerns—the resource and the API version—into the same URL component.

---

### Purposes

- To provide the lowest-friction implementation path for simple or internal APIs.
- To allow quick manual testing in a browser without custom headers.
- To keep the base path stable while still varying behavior by version.
- To serve as an interim strategy during migration to a more structured approach.

---

### Syntax Rules and Structure

#### Complete General Syntax: Query-Based Version Branching

```javascript
app.get('/api/users', (req, res) => {
  const version = req.query.version || '1';
  if (version === '2') {
    return res.json({ version: 'v2', users: [{ id: 1, name: 'John', email: 'john@example.com' }] });
  }
  res.json({ version: 'v1', users: [{ id: 1, name: 'John' }] });
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `req.query.version` | Raw query parameter string. |
| Default (`'1'`) | Applied when the parameter is absent. |
| Conditional branching | Dispatches to version-specific logic. |

#### Syntax Rules

- Always apply a default version when the parameter is missing, to avoid breaking existing clients.
- Validate the version value against a known set; do not pass arbitrary strings into branching logic.
- Document the query parameter prominently; it is less discoverable than a URI segment.

#### Constraints and Limitations

- Mixes protocol negotiation with application-level filtering; discouraged for public APIs.
- Caching is possible but requires including the query string in cache keys (which most caches do by default).
- Less RESTful than URI or media type versioning.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Query Parameter Version Switching

```javascript
const express = require('express');
const app = express();

app.get('/api/users', (req, res) => {
  const version = req.query.version || '1';

  if (version === '2') {
    return res.json({
      version: 'v2',
      users: [{ id: 1, name: 'John', email: 'john@example.com' }]
    });
  }

  res.json({
    version: 'v1',
    users: [{ id: 1, name: 'John' }]
  });
});

app.listen(3000);
```

**Expected Output**

```
GET /api/users            → 200 { "version": "v1", ... }
GET /api/users?version=1  → 200 { "version": "v1", ... }
GET /api/users?version=2  → 200 { "version": "v2", ... }
```

**Why This Output Occurs**

- The query parameter is read and defaulted; the handler branches accordingly.

---

### Real-World Cases

- **Simple internal tools** — Low-traffic services where implementation speed matters more than REST purity.
- **Prototyping and MVPs** — Quick version switching before a formal strategy is chosen.
- **Legacy APIs** — Existing endpoints that already expose query parameters and cannot easily change their URL structure.

---

## Core Concept 4: Media Type Versioning

### Definitions

**Core Definition**
Media type versioning uses HTTP content negotiation to encode the API version in the `Accept` header’s media type, such as `application/vnd.api.v1+json` or `application/vnd.api.v2+json`.

**Technical Definition**
Media type versioning leverages the vendor-specific media type subtree defined by RFC 6838 (`vnd.*`) to make version negotiation explicit in the HTTP protocol rather than in the URL. In Express.js, the server inspects `req.accepts()` or the raw `Accept` header, extracts the version from the media type, and dispatches accordingly. If no supported media type is provided, the server may return `406 Not Acceptable` or fall back to a default version.

**Beginner-Friendly Explanation**
Media type versioning is the most “HTTP-native” approach: the client says, “I want version 1 of this resource” by setting the `Accept` header to a custom media type like `application/vnd.myapi.v1+json`. The URL stays clean, and the version is part of the content negotiation handshake. It is powerful but complex, and it is hardest to test manually.

---

### Purposes

- To make version negotiation explicit in the HTTP protocol rather than in the URL or a custom header.
- To align with RESTful content-negotiation semantics and hypermedia APIs.
- To keep URLs completely stable across versions.
- To allow fine-grained media type evolution alongside version changes.

---

### Syntax Rules and Structure

#### Complete General Syntax: Media Type Version Selection

```javascript
const parseMediaVersion = (req, res, next) => {
  const accept = req.headers.accept || '';
  const match = accept.match(/application\/vnd\.myapi\.v(\d+)\+json/);
  req.apiVersion = match ? match[1] : '1';
  res.setHeader('Vary', 'Accept');
  next();
};

app.get('/api/users', parseMediaVersion, (req, res) => {
  if (req.apiVersion === '2') {
    return res.json({ version: 'v2', users: [{ id: 1, name: 'John', email: 'john@example.com' }] });
  }
  res.json({ version: 'v1', users: [{ id: 1, name: 'John' }] });
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `Accept` header | Carries the requested media type and version. |
| Regex extraction | Pulls the version number from `vnd.myapi.vN+json`. |
| `Vary: Accept` | Ensures caches key responses on the `Accept` header. |

#### Syntax Rules

- Use the vendor media type subtree (`vnd.*`) as specified by RFC 6838.
- Set `Vary: Accept` to prevent cross-version cache contamination.
- Provide a default version when the `Accept` header does not specify a recognized media type.

#### Constraints and Limitations

- **Poor discoverability** — clients cannot see the version in the URL; documentation must be explicit.
- **Complex implementation** — requires media type parsing and careful `Vary` handling.
- **Difficult manual testing** — requires custom `Accept` headers in `curl` or an API client.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Vendor Media Type Version Dispatch

```javascript
const express = require('express');
const app = express();

function usersV1(req, res) {
  res.json({ version: 'v1', users: [{ id: 1, name: 'John' }] });
}

function usersV2(req, res) {
  res.json({ version: 'v2', users: [{ id: 1, name: 'John', email: 'john@example.com' }] });
}

app.get('/api/users', (req, res) => {
  res.setHeader('Vary', 'Accept');

  if (req.accepts('application/vnd.myapi.v2+json')) {
    return usersV2(req, res);
  }

  usersV1(req, res);
});

app.listen(3000);
```

**Expected Output**

```
GET /api/users
Accept: application/vnd.myapi.v1+json
→ 200 { "version": "v1", ... }

GET /api/users
Accept: application/vnd.myapi.v2+json
→ 200 { "version": "v2", ... }

GET /api/users
Accept: application/json
→ 200 { "version": "v1", ... }   (default fallback)
```

**Why This Output Occurs**

- `req.accepts()` checks whether the client’s `Accept` header includes the vendor media type for v2; if not, the default v1 handler runs.

---

### Real-World Cases

- **Hypermedia APIs** — Media type versioning aligns with HATEOAS and content-negotiation-driven clients.
- **Standards-heavy environments** — Organizations that mandate RFC 6838 vendor media types for versioning.
- **Multi-format APIs** — APIs that already use media types to distinguish JSON, XML, and other representations.

---

## Core Concept 5: Versioning Best Practices

### Definitions

**Core Definition**
Versioning best practices are the design and operational patterns that keep API versioning predictable, secure, cache-safe, and respectful of client migration timelines.

**Technical Definition**
Best practices encompass: versioning **only breaking changes**; supporting at least two versions during migration; choosing URI versioning for public APIs and header/media type versioning for internal or hypermedia APIs; including `Deprecation` and `Sunset` headers on older versions (RFC 9745 and RFC 8594); setting `Vary` headers for header- and media-type-based strategies; using semantic versioning principles where granular negotiation is needed; and documenting each version’s contract independently.

**Beginner-Friendly Explanation**
Best practices are the habits that keep versioning from becoming a mess. Only make a new version for big, breaking changes. Keep old versions alive long enough for clients to migrate. Tell clients clearly when an old version will be shut down. And make sure your caching setup does not accidentally serve version 2 data to a version 1 client.

---

### Purposes

- To avoid unnecessary version proliferation by reserving new versions for breaking changes only.
- To give clients a predictable, well-communicated migration window.
- To prevent caching errors that could serve incorrect version data.
- To signal deprecation formally through standard HTTP headers.

---

### Syntax Rules and Structure

#### Complete General Syntax: Deprecation and Sunset Headers

```javascript
app.use('/api/v1', (req, res, next) => {
  res.setHeader('Deprecation', '@1688169599');       // RFC 9745 (Unix timestamp)
  res.setHeader('Sunset', 'Wed, 31 Dec 2025 23:59:59 GMT'); // RFC 8594
  res.setHeader('Link', '<https://api.example.com/v2>; rel="successor-version"');
  next();
});
```

**Component Breakdown**

| Header | Purpose |
|--------|---------|
| `Deprecation` | Indicates when the version became deprecated (RFC 9745). |
| `Sunset` | Indicates when the version will be removed (RFC 8594). |
| `Link` | Points to the successor version documentation. |

#### Syntax Rules

- Do **not** create a new version for additive, backward-compatible changes.
- Set `Vary: API-Version` or `Vary: Accept` when using header- or media-type-based versioning.
- Register deprecation and sunset headers on **all** responses from deprecated versions.
- Keep at least two versions active during migration windows.

#### Constraints and Limitations

- Deprecation headers are advisory; clients may ignore them.
- Supporting many versions indefinitely increases maintenance cost; enforce sunset dates.
- URI versioning does not eliminate the need for deprecation communication.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Deprecated v1 Router with Sunset Signaling

```javascript
const express = require('express');
const app = express();

const v1Router = express.Router();
v1Router.use((req, res, next) => {
  res.setHeader('Deprecation', '@1688169599');
  res.setHeader('Sunset', 'Wed, 31 Dec 2025 23:59:59 GMT');
  res.setHeader('Link', '<https://api.example.com/v2>; rel="successor-version"');
  next();
});

v1Router.get('/users', (req, res) => {
  res.json({ version: 'v1', users: [{ id: 1, name: 'John' }] });
});

const v2Router = express.Router();
v2Router.get('/users', (req, res) => {
  res.json({ version: 'v2', users: [{ id: 1, name: 'John', email: 'john@example.com' }] });
});

app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);
app.listen(3000);
```

**Expected Output**

```
GET /api/v1/users
→ 200 { "version": "v1", ... }
Deprecation: @1688169599
Sunset: Wed, 31 Dec 2025 23:59:59 GMT
Link: <https://api.example.com/v2>; rel="successor-version"

GET /api/v2/users
→ 200 { "version": "v2", ... }
(no deprecation headers)
```

**Why This Output Occurs**

- The v1 router’s middleware attaches deprecation metadata to every response, signaling clients to migrate before the sunset date.

---

### Real-World Cases

- **Public API lifecycles** — Stripe-style deprecation policies with explicit sunset dates.
- **Enterprise integrations** — `Deprecation` and `Sunset` headers provide auditable migration signals.
- **Cache-sensitive deployments** — Correct `Vary` configuration prevents version mix-ups at the CDN layer.

---

## References

- Express Route Versioning (Compile-N-Run) — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/1-express-routing/12-express-route-versioning.mdx
- express-route-versioning-middleware (npm) — https://www.npmjs.com/package/express-route-versioning-middleware
- API Versioning Strategies (vasilyu1983 / AI-Agents-public) — https://github.com/vasilyu1983/AI-Agents-public/blob/3424d6f5e94010409da012eeb1cb84aaecec88b3/frameworks/shared-skills/skills/dev-api-design/references/versioning-strategies.md
- API Versioning Strategies Comparison (ancoleman / ai-design-components) — https://raw.githubusercontent.com/ancoleman/ai-design-components/refs/heads/main/skills/implementing-api-patterns/references/versioning-strategies.md
- RFC 6838: Media Type Specifications and Registration Procedures — https://datatracker.ietf.org/doc/html/rfc6838
- RFC 9745: The Deprecation HTTP Response Header Field — https://www.rfc-editor.org/rfc/rfc9745
- RFC 8594: The Sunset HTTP Header Field — https://www.rfc-editor.org/rfc/rfc8594
- Azure Architecture Center: Web API Design Best Practices (Versioning) — https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design#versioning-a-restful-web-api
- express-version-api (npm, semantic versioning) — https://www.npmjs.com/package/@braian-quintian/express-version-api