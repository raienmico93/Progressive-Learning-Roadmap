# REST Fundamentals & Architectural Constraints — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** REST (REpresentational State Transfer) is an architectural style for distributed hypermedia systems defined by Roy Thomas Fielding in his 2000 doctoral dissertation, providing a set of constraints that, when applied, yield desirable properties such as scalability, simplicity, modifiability, and performance.

**Technical Definition:** REST is a hybrid architectural style derived by combining several network-based architectural styles. Fielding defines REST through a set of architectural constraints: Client-Server, Stateless, Cache, Layered System, Uniform Interface, and optionally Code-on-Demand. An architecture satisfying all mandatory constraints gains the advantages of REST, including scalability and loose coupling between systems. The Uniform Interface constraint is further decomposed into four sub-constraints: identification of resources, manipulation of resources through representations, self-descriptive messages, and hypermedia as the engine of application state (HATEOAS).

**Beginner-Friendly Explanation:** REST is a set of design rules for building web APIs. Think of it as a recipe for making APIs that are easy to understand, scale well, and work consistently across different systems. Instead of inventing your own rules for how clients and servers talk to each other, you follow REST's proven principles. The key idea is that everything is a "resource" (like a user, an order, or a product), and you interact with these resources using standard HTTP methods.

### Key Characteristics

- **Resource-oriented:** Resources are the key abstraction of information; any information that can be named can be a resource.
- **Constraint-driven:** REST is defined by six architectural constraints that must be satisfied for an API to be considered RESTful.
- **HTTP-native:** REST leverages HTTP methods (GET, POST, PUT, DELETE) and status codes as the uniform interface.
- **Stateless:** Each request contains all necessary information for the server to understand and process it; no server-side session context is stored.
- **Cacheable:** Responses must be explicitly labeled as cacheable or non-cacheable to improve network efficiency.
- **Layered:** Clients cannot tell whether they are connected directly to the end server or to an intermediary.
- **Hypermedia-driven (HATEOAS):** Clients interact with the application entirely through hypermedia provided dynamically by the server.

### Prerequisites

- **Basic HTTP knowledge:** Understanding of request methods, headers, status codes, and bodies.
- **Web development fundamentals:** Familiarity with URLs, clients, servers, and APIs.
- **JSON/XML familiarity:** Understanding of data representation formats.
- **Basic understanding of distributed systems:** Client-server model, statelessness, and caching concepts.

### Related Programming Areas

- **HTTP Fundamentals:** Request/response anatomy, methods, status codes, and headers.
- **API Design:** URI design, versioning, pagination, and error handling.
- **Web Frameworks:** Express, Fastify, NestJS, and other REST-oriented frameworks.
- **Microservices:** REST as a communication pattern between services.
- **GraphQL and gRPC:** Alternative API paradigms often compared to REST.

### Core Concepts

1. **The REST Constraints** — Client-Server, Stateless, Layered System, Cacheability, Uniform Interface.
2. **Core Concepts** — Resources, Endpoints, HTTP Verbs, and Representations.

---

## Core Concept 1: The REST Constraints

### Sub-Feature 1.1: Client-Server Separation and Statelessness

#### Definitions

**Core Definition:** Client-Server separation requires that the user interface (client) and data storage (server) be separated, while statelessness requires that each request contain all information necessary for the server to understand it, without relying on stored server-side session context.

**Technical Definition:** The Client-Server constraint mandates that communication within network-based applications takes place between a client that makes requests and a server that responds. This separation improves scalability and simplifies server components. The Stateless constraint requires that each request must contain all the necessary information so that the server can understand it; the server is not allowed to use any server-side context held in memory. Session state must reside entirely on the client.

**Beginner-Friendly Explanation:** Client-Server is like a restaurant: the customer (client) places an order, and the kitchen (server) prepares it. They have clear roles and responsibilities. Statelessness is like a vending machine: every time you want a snack, you insert money and make your selection from scratch. The machine doesn't remember your last purchase — each transaction is complete on its own.

#### Purposes

- To improve scalability by separating concerns between client and server.
- To simplify server components by removing the need to store session state.
- To enable independent evolution of client and server components.
- To increase reliability and visibility for monitoring and debugging.

#### Syntax Rules and Structure

| Constraint | Fielding's Description | Implementation |
|------------|----------------------|----------------|
| Client-Server | Communication takes place between client and server; the client initiates requests. | HTTP request/response model. |
| Stateless | Each request must contain all necessary information; no server-side context. | No server-side sessions; use tokens, API keys, or self-contained payloads. |

**Constraints and Limitations:**
- Statelessness means every request must carry authentication credentials (e.g., tokens).
- Server-side sessions violate the Stateless constraint.
- Statelessness improves scalability but may increase request size.

#### Annotated Code Example

```javascript
// Client-Server: Express server handling a stateless request
const express = require('express');
const app = express();

app.use(express.json());

// Stateless: every request carries its own authentication token
app.get('/api/users/:id', authenticate, (req, res) => {
  // The server does not store any session context
  res.json({ userId: req.params.id, authenticated: true });
});

function authenticate(req, res, next) {
  const token = req.headers['authorization'];
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  // Validate token on every request (stateless)
  req.user = { id: 1, name: 'Alice' };
  next();
}

app.listen(3000);
```

**Expected Output (with `Authorization: Bearer token123`):**
```json
{"userId":"42","authenticated":true}
```

**Expected Output (without token):**
```json
{"error":"No token provided"}
```

**Why this output:** The server does not store any session state. Every request must include the authentication token. This is the Stateless constraint in action — the server can be restarted, scaled horizontally, or replaced without losing client state.

#### Real-World Cases

- **Microservices:** Each service is stateless and can be scaled independently.
- **Load balancing:** Any server can handle any request because no session state is stored locally.
- **JWT authentication:** Tokens carry all necessary authentication information.

---

### Sub-Feature 1.2: Layered System Architecture

#### Definitions

**Core Definition:** The Layered System constraint allows an architecture to be composed of hierarchical layers by constraining component behaviour such that each component cannot "see" beyond the immediate layer with which it is interacting.

**Technical Definition:** The Layered System constraint enforces that a client cannot see beyond the server with which it is interacting. This allows for added security and load balancing by placing intermediaries (proxies, gateways, load balancers) between the client and the actual server. The client is unaware of whether it is connected directly to the end server or to an intermediary along the way.

**Beginner-Friendly Explanation:** A layered system is like ordering food through a delivery app. You don't know (or care) whether the restaurant cooks the food itself or subcontracts to a central kitchen. You just place your order and get your food. The app, the restaurant, and the kitchen are all separate layers.

#### Purposes

- To enable load balancing, caching, and security through intermediaries.
- To allow independent deployment and scaling of layers.
- To improve security by isolating backend services from direct client access.
- To enable legacy encapsulation and system migration.

#### Syntax Rules and Structure

| Layer | Role | Example |
|-------|------|---------|
| Client | User interface | Browser, mobile app |
| Intermediary | Proxy, gateway, load balancer | Nginx, Cloudflare, AWS ALB |
| Server | Application logic and data | Express, Fastify, NestJS |

**Constraints and Limitations:**
- Intermediaries add latency.
- Debugging across layers can be complex.
- The client must not depend on direct communication with the origin server.

#### Annotated Code Example

```javascript
// Layered System: Nginx as a reverse proxy in front of Express
// nginx.conf
// server {
//   listen 80;
//   location /api/ {
//     proxy_pass http://localhost:3000/;
//     proxy_set_header X-Forwarded-For $remote_addr;
//   }
// }

// Express server (behind the proxy)
const express = require('express');
const app = express();

app.get('/api/users', (req, res) => {
  // The server sees the proxy's IP, not the client's
  // X-Forwarded-For carries the original client IP
  const clientIp = req.headers['x-forwarded-for'] || req.ip;
  res.json({ users: [], clientIp });
});

app.listen(3000);
```

**Expected Output (through the proxy):**
```json
{"users":[],"clientIp":"203.0.113.42"}
```

**Why this output:** The Nginx proxy sits between the client and the Express server. The Express server sees the proxy's IP address, not the client's. The `X-Forwarded-For` header carries the original client IP, demonstrating the layered architecture.

#### Real-World Cases

- **API gateways:** AWS API Gateway, Kong, and Apigee sit between clients and backend services.
- **CDNs:** Cloudflare and Akamai cache and serve content from edge locations.
- **Load balancers:** AWS ALB distributes traffic across multiple backend instances.

---

### Sub-Feature 1.3: Cacheability of Data

#### Definitions

**Core Definition:** Cacheability requires that the data within a response to a request be implicitly or explicitly labeled as cacheable or non-cacheable, allowing clients and intermediaries to reuse cached data for future identical requests.

**Technical Definition:** The Cache constraint specifies that the data within each response to a request must be labeled as cacheable or non-cacheable. If a response is cacheable, the client is allowed to reuse the cached data for future identical requests. HTTP provides several headers to define caching behaviour, including `Cache-Control`, `ETag`, and `Last-Modified`.

**Beginner-Friendly Explanation:** Caching is like keeping a copy of a frequently used document on your desk instead of walking to the filing cabinet every time. The server tells you whether it's safe to keep a copy (cacheable) or whether you must always fetch the latest version (non-cacheable).

#### Purposes

- To reduce network traffic and latency.
- To improve scalability by reducing server load.
- To enable offline or degraded-mode operation.
- To optimise bandwidth usage for mobile and constrained networks.

#### Syntax Rules and Structure

| Header | Purpose | Example |
|--------|---------|---------|
| `Cache-Control` | Directives for caching mechanisms. | `max-age=3600, public` |
| `ETag` | Unique identifier for a specific version of a resource. | `"33a64df551425fcc55e4d42a148795d9f25f89d4"` |
| `Last-Modified` | When the resource was last modified. | `Wed, 15 Jan 2026 12:00:00 GMT` |

**Cache-Control directives:**
| Directive | Description |
|-----------|-------------|
| `public` | Response may be cached by any cache. |
| `private` | Response is for a single user; not shared caches. |
| `no-cache` | Response must be validated before use. |
| `no-store` | Response must not be cached. |
| `max-age=<seconds>` | Maximum time the response is fresh. |

**Constraints and Limitations:**
- Caching can serve stale data if not properly invalidated.
- `no-store` should be used for sensitive data (e.g., banking details).
- ETag-based validation requires a round trip (304 Not Modified).

#### Annotated Code Example

```javascript
// Cacheability in Express
const express = require('express');
const app = express();

// Cacheable endpoint (public, 1 hour)
app.get('/api/products', (req, res) => {
  res.set('Cache-Control', 'public, max-age=3600');
  res.json([{ id: 1, name: 'Widget' }]);
});

// Non-cacheable endpoint (sensitive data)
app.get('/api/account', (req, res) => {
  res.set('Cache-Control', 'no-store');
  res.json({ balance: 1000 });
});

// ETag-based validation
app.get('/api/config', (req, res) => {
  const data = { theme: 'dark', version: '1.0.0' };
  const etag = '"' + require('crypto').createHash('md5').update(JSON.stringify(data)).digest('hex') + '"';
  res.set('ETag', etag);

  if (req.headers['if-none-match'] === etag) {
    return res.status(304).end(); // Not Modified
  }
  res.json(data);
});

app.listen(3000);
```

**Expected Output (first request to `/api/config`):**
```json
{"theme":"dark","version":"1.0.0"}
```
**Response header:** `ETag: "abc123..."`

**Expected Output (second request with `If-None-Match: "abc123..."`):**
```
304 Not Modified
```

**Why this output:** The first request returns the full response with an ETag. The client caches it. On subsequent requests, the client sends the ETag in `If-None-Match`. If the resource hasn't changed, the server returns 304 Not Modified, saving bandwidth.

#### Real-World Cases

- **CDN caching:** Static assets (CSS, JS, images) are cached at edge locations.
- **API caching:** Frequently accessed read-only data (product catalogues, configurations).
- **Browser caching:** Using `Cache-Control` and `ETag` to avoid re-downloading unchanged resources.

---

### Sub-Feature 1.4: Uniform Interface Principles

#### Definitions

**Core Definition:** The Uniform Interface constraint, described by Fielding as the central feature that distinguishes REST from other network-based styles, prescribes a uniform interface between all components, decomposed into four sub-constraints: identification of resources, manipulation of resources through representations, self-descriptive messages, and HATEOAS.

**Technical Definition:** The Uniform Interface is a constraint placed on REST services to simplify things and ensure that services can be managed independently from one another. It is defined by four guiding principles: (1) Identification of Resources — individual resources are identified in requests using URIs; (2) Manipulation of Resources Through Representations — when a client holds a representation, it has enough information to modify or delete the resource; (3) Self-Descriptive Messages — each message includes enough information to describe how to process it; (4) Hypermedia as the Engine of Application State (HATEOAS) — clients interact with the application entirely through hypermedia provided dynamically by the server.

**Beginner-Friendly Explanation:** The Uniform Interface is like a universal language for APIs. Instead of each API inventing its own way of doing things, REST APIs all speak the same language: they use URIs to identify things, HTTP methods to act on them, and standard formats to describe them. HATEOAS takes this further by making the API self-discoverable — like a website where you can click links to navigate, rather than needing to know all the URLs in advance.

#### Purposes

- To simplify and decouple the architecture, enabling independent evolution.
- To enable self-discoverability through hypermedia links (HATEOAS).
- To ensure that messages are self-descriptive and can be processed without external context.
- To provide a consistent interaction model across all resources.

#### Syntax Rules and Structure

| Sub-Constraint | Description | Example |
|----------------|-------------|---------|
| Identification of Resources | Resources identified by URIs. | `/users/42` |
| Manipulation Through Representations | Client sends representation to modify resource. | PUT with JSON body |
| Self-Descriptive Messages | Messages contain enough info to process. | `Content-Type: application/json` |
| HATEOAS | Hypermedia drives state transitions. | Links in response body |

**HATEOAS response example:**
```json
{
  "orderID": 3,
  "productID": 2,
  "quantity": 4,
  "links": [
    { "rel": "customer", "href": "https://api.contoso.com/customers/3", "action": "GET" },
    { "rel": "self", "href": "https://api.contoso.com/orders/3", "action": "GET" }
  ]
}
```

**Constraints and Limitations:**
- HATEOAS has yet to be adopted as a mainstream feature of REST APIs.
- There is no general-purpose standard for modelling HATEOAS.
- HATEOAS complicates versioning because all links must include the version number.

#### Annotated Code Example

```javascript
// HATEOAS in Express
const express = require('express');
const app = express();

app.get('/api/orders/:id', (req, res) => {
  const orderId = req.params.id;

  res.json({
    orderID: orderId,
    productID: 2,
    quantity: 4,
    links: [
      {
        rel: 'self',
        href: `https://api.example.com/orders/${orderId}`,
        action: 'GET',
      },
      {
        rel: 'customer',
        href: `https://api.example.com/customers/3`,
        action: 'GET',
      },
      {
        rel: 'cancel',
        href: `https://api.example.com/orders/${orderId}`,
        action: 'DELETE',
      },
    ],
  });
});

app.listen(3000);
```

**Expected Output (for `GET /api/orders/3`):**
```json
{
  "orderID": "3",
  "productID": 2,
  "quantity": 4,
  "links": [
    { "rel": "self", "href": "https://api.example.com/orders/3", "action": "GET" },
    { "rel": "customer", "href": "https://api.example.com/customers/3", "action": "GET" },
    { "rel": "cancel", "href": "https://api.example.com/orders/3", "action": "DELETE" }
  ]
}
```

**Why this output:** The response includes hypermedia links that tell the client what actions are available for this order. The client can navigate to the customer, cancel the order, or re-fetch the order — all without hardcoding URLs. This is HATEOAS in practice.

#### Real-World Cases

- **PayPal REST API:** Uses HATEOAS links extensively.
- **Spring HATEOAS:** Java library for building hypermedia-driven APIs.
- **Hypermedia APIs:** GitHub's API includes hypermedia links for pagination and navigation.

---

## Core Concept 2: Core REST Concepts

### Sub-Feature 2.1: Resources — Real-World Entities vs. Abstractions

#### Definitions

**Core Definition:** A resource is the key abstraction of information in REST; any information that can be named can be a resource, and each resource is identified by a unique resource identifier (URI) that remains the same even if the content of the resource changes.

**Technical Definition:** Resources are conceptually separate from the representations that are returned to the client. The server does not send its database; rather, it sends some HTML, XML, or JSON that represents some database records. A resource can be a real-world entity (a user, an order, a product) or an abstraction (a collection, a relationship, a computed value).

**Beginner-Friendly Explanation:** A resource is anything worth talking about in your API — a user, a photo, a shopping cart, or even a calculation like "the current temperature in London." Think of it as a noun. The URI is the resource's address, and the representation is how you describe it (in JSON, XML, etc.).

#### Purposes

- To provide a clear, noun-based model of the API domain.
- To enable stable, persistent identifiers for entities.
- To separate the concept of a resource from its representation.
- To support hierarchical resource relationships.

#### Syntax Rules and Structure

| Resource Type | Example URI | Description |
|---------------|-------------|-------------|
| Collection | `/users` | A list of users. |
| Item | `/users/42` | A single user. |
| Sub-resource | `/users/42/orders` | Orders belonging to user 42. |
| Singleton | `/users/me` | The current authenticated user. |

**Constraints and Limitations:**
- Resources should be nouns, not verbs.
- URIs should be stable; the same resource identifier should persist even if content changes.
- Resources are conceptually separate from their representations.

#### Annotated Code Example

```javascript
// Resource-oriented design in Express
const express = require('express');
const app = express();

// Collection resource
app.get('/api/users', (req, res) => {
  res.json([
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' },
  ]);
});

// Item resource
app.get('/api/users/:id', (req, res) => {
  res.json({ id: req.params.id, name: 'Alice' });
});

// Sub-resource
app.get('/api/users/:id/orders', (req, res) => {
  res.json({ userId: req.params.id, orders: [] });
});

app.listen(3000);
```

**Expected Output (for `GET /api/users`):**
```json
[{"id":1,"name":"Alice"},{"id":2,"name":"Bob"}]
```

**Expected Output (for `GET /api/users/42/orders`):**
```json
{"userId":"42","orders":[]}
```

**Why this output:** The URIs are noun-based and hierarchical. `/users` is the collection, `/users/42` is an item, and `/users/42/orders` is a sub-resource. This mirrors the resource model of the domain.

#### Real-World Cases

- **Social media:** `/users`, `/posts`, `/comments`, `/likes`.
- **E-commerce:** `/products`, `/carts`, `/orders`, `/customers`.
- **IoT:** `/devices`, `/sensors`, `/readings`.

---

### Sub-Feature 2.2: Endpoints — Designing Predictable URIs

#### Definitions

**Core Definition:** An endpoint is a specific URI (or URI pattern) that a client can access to interact with a resource. Designing predictable URIs means following consistent naming conventions that make the API intuitive and navigable.

**Technical Definition:** A Web API should be modelled as a resource hierarchy to leverage the hierarchical nature of the URI to imply structure (association, composition, or aggregation). Each node in the hierarchy is either a simple resource or a collection resource. URIs should follow the IETF RFC 3986 standard and avoid potential collisions with page URLs.

**Beginner-Friendly Explanation:** Endpoints are like addresses on a map. Predictable URIs follow a consistent pattern — like a postal system where you know that `/users/42` means "the user with ID 42." If you know the pattern, you can guess the address of any resource without looking it up.

#### Purposes

- To make APIs intuitive and self-documenting.
- To enable clients to discover resources without prior knowledge of the URI schema.
- To support hierarchical resource relationships.
- To follow web standards (RFC 3986).

#### Syntax Rules and Structure

| Pattern | Example | Description |
|---------|---------|-------------|
| Collection | `/users` | All users. |
| Item | `/users/{id}` | A specific user. |
| Sub-resource | `/users/{id}/orders` | Orders for a specific user. |
| Action (controller) | `/users/{id}/activate` | A controller action (last segment). |
| Versioned | `/api/v1/users` | API version in the path. |

**Constraints and Limitations:**
- Use nouns, not verbs (except for controller actions).
- Use plural nouns for collections (`/users`, not `/user`).
- Use forward slashes for hierarchy; avoid trailing slashes.
- Version your API (`/api/v1/...`).

#### Annotated Code Example

```javascript
// Predictable URI design in Express
const express = require('express');
const app = express();

// Collection
app.get('/api/v1/users', (req, res) => { /* list users */ });

// Item
app.get('/api/v1/users/:id', (req, res) => { /* get user */ });

// Sub-resource
app.get('/api/v1/users/:id/orders', (req, res) => { /* user orders */ });

// Controller action (verb as last segment)
app.post('/api/v1/users/:id/activate', (req, res) => { /* activate user */ });

app.listen(3000);
```

**Expected Output (for `GET /api/v1/users/42/orders`):**
```json
{"userId":"42","orders":[]}
```

**Why this output:** The URI follows a predictable pattern: `/api/{version}/{resource}/{id}/{sub-resource}`. Any developer familiar with REST conventions can guess the URI for a user's orders without reading documentation.

#### Real-World Cases

- **Stripe API:** `/v1/charges`, `/v1/customers`, `/v1/payment_intents`.
- **GitHub API:** `/repos/{owner}/{repo}/issues`, `/users/{username}`.
- **Twitter API:** `/2/tweets`, `/2/users/{id}/tweets`.

---

### Sub-Feature 2.3: HTTP Verbs — Safe vs. Idempotent vs. Non-Idempotent Methods

#### Definitions

**Core Definition:** HTTP verbs (methods) indicate the desired action on a resource. Safe methods are read-only; idempotent methods produce the same result whether called once or multiple times; non-idempotent methods may produce different results on repeated calls.

**Technical Definition:** A request method is considered "safe" if its defined semantics are essentially read-only; the client does not request or expect any state change on the origin server. A request method is considered "idempotent" if the intended effect on the server of multiple identical requests with that method is the same as the effect for a single such request. Of the request methods defined by RFC 9110, PUT, DELETE, and safe request methods (GET, HEAD, OPTIONS) are idempotent.

**Beginner-Friendly Explanation:** Think of HTTP methods as different types of actions. GET is like looking at a painting — you can look as many times as you want, and it doesn't change. PUT is like replacing a painting with a new one — doing it twice gives the same result. POST is like depositing money — doing it twice means you've deposited twice as much.

#### Purposes

- To provide a standardised way to express intent for resource operations.
- To guide caching strategies (safe methods are cacheable).
- To inform retry logic (idempotent methods can be safely retried).
- To align API design with HTTP semantics.

#### Syntax Rules and Structure

| Method | Safe | Idempotent | Cacheable | Typical Use |
|--------|------|------------|-----------|-------------|
| GET | Yes | Yes | Yes | Retrieve a resource |
| HEAD | Yes | Yes | Yes | Retrieve headers only |
| OPTIONS | Yes | Yes | No | Discover allowed methods |
| POST | No | No | No | Create a resource |
| PUT | No | Yes | No | Replace a resource |
| PATCH | No | No | No | Partially modify a resource |
| DELETE | No | Yes | No | Remove a resource |

**Constraints and Limitations:**
- Safety and idempotency are semantic guarantees, not enforced by the protocol.
- Servers may implement non-idempotent behaviour for idempotent methods (bad practice).
- Caching proxies rely on these properties; violating them can cause subtle bugs.

#### Annotated Code Example

```javascript
// HTTP verbs in Express
const express = require('express');
const app = express();
app.use(express.json());

let users = [{ id: 1, name: 'Alice' }];

// GET (safe, idempotent, cacheable)
app.get('/api/users', (req, res) => {
  res.json(users);
});

// POST (non-safe, non-idempotent)
app.post('/api/users', (req, res) => {
  const user = { id: users.length + 1, name: req.body.name };
  users.push(user);
  res.status(201).json(user);
});

// PUT (non-safe, idempotent)
app.put('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (user) {
    user.name = req.body.name;
    res.json(user);
  } else {
    res.status(404).json({ error: 'Not found' });
  }
});

// DELETE (non-safe, idempotent)
app.delete('/api/users/:id', (req, res) => {
  users = users.filter(u => u.id !== parseInt(req.params.id));
  res.status(204).end();
});

app.listen(3000);
```

**Expected Output (for `POST /api/users` with `{"name":"Bob"}` twice):**
```
Two separate users are created (IDs 2 and 3).
```

**Expected Output (for `PUT /api/users/1` with `{"name":"Alice"}` twice):**
```
The same user is updated with the same name (idempotent).
```

**Why this output:** POST creates a new resource each time it is called (non-idempotent). PUT replaces the resource with the same data each time (idempotent). This demonstrates the fundamental difference between the two methods.

#### Real-World Cases

- **REST APIs:** Using GET for retrieval, POST for creation, PUT for full updates, PATCH for partial updates, DELETE for removal.
- **Payment systems:** Using idempotency keys with POST to make payment creation idempotent.
- **Caching proxies:** Caching GET responses because GET is safe and cacheable.

---

### Sub-Feature 2.4: Representations — Content-Type Negotiation

#### Definitions

**Core Definition:** A representation is a specific encoding of a resource's state, and content negotiation is the process by which the client and server agree on the representation format using the `Accept` and `Content-Type` headers.

**Technical Definition:** The interaction between client and server consists of sending and receiving representations of resources. When a client holds a representation of a resource, including any metadata attached, it has enough information to modify or delete the resource on the server. Content negotiation uses the `Accept` request header (client preferences) and the `Content-Type` request/response header (actual format) to select the appropriate representation.

**Beginner-Friendly Explanation:** A resource is like a person, and a representation is like a photograph of that person. You can photograph someone in colour or black and white, from different angles, or in different sizes. The resource (the person) stays the same, but the representation (the photo) changes. Content negotiation is how the client says "I prefer colour photos in JSON format."

#### Purposes

- To allow the same resource to be served in multiple formats (JSON, XML, HTML).
- To respect client preferences for content type.
- To enable API versioning through content types.
- To support both human-readable and machine-readable responses.

#### Syntax Rules and Structure

| Header | Direction | Purpose |
|--------|-----------|---------|
| `Accept` | Request | Media types the client can handle. |
| `Content-Type` | Request/Response | Media type of the body. |

**Quality values (`q` parameter):**
```
Accept: application/json, text/html;q=0.9, */*;q=0.8
```
| Media Type | Quality | Meaning |
|------------|---------|---------|
| `application/json` | 1.0 (default) | Most preferred. |
| `text/html` | 0.9 | Less preferred. |
| `*/*` | 0.8 | Any media type. |

**Constraints and Limitations:**
- The `Accept` header can be complex; a proper parser is recommended.
- If the server cannot satisfy the `Accept` header, it should return 406 Not Acceptable.
- JSON is not suitable for all data types (e.g., binary data).

#### Annotated Code Example

```javascript
// Content negotiation in Express
const express = require('express');
const app = express();

const users = [{ id: 1, name: 'Alice' }];

app.get('/api/users', (req, res) => {
  const accept = req.headers['accept'] || '*/*';

  if (accept.includes('application/json')) {
    res.type('application/json').send(users);
  } else if (accept.includes('text/html')) {
    const rows = users.map(u => `<tr><td>${u.id}</td><td>${u.name}</td></tr>`).join('');
    res.type('text/html').send(`<table>${rows}</table>`);
  } else {
    res.status(406).send('Not Acceptable');
  }
});

app.listen(3000);
```

**Expected Output (with `Accept: application/json`):**
```json
[{"id":1,"name":"Alice"}]
```

**Expected Output (with `Accept: text/html`):**
```html
<table><tr><td>1</td><td>Alice</td></tr></table>
```

**Expected Output (with `Accept: application/xml`):**
```
406 Not Acceptable
```

**Why this output:** The server inspects the `Accept` header and selects the appropriate representation. JSON and HTML are supported; XML is not, so the server returns 406 Not Acceptable.

#### Real-World Cases

- **GitHub API:** Supports JSON and XML representations via `Accept` headers.
- **Django REST Framework:** Built-in content negotiation for JSON, XML, and browsable HTML.
- **API versioning:** Using `Accept: application/vnd.api.v2+json` for versioning.

---

## References

- Fielding, R. T. (2000). Architectural Styles and the Design of Network-based Software Architectures. Doctoral dissertation, University of California, Irvine. — https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm
- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 — HTTP Caching — https://www.rfc-editor.org/rfc/rfc9111
- RFC 3986 — Uniform Resource Identifier (URI): Generic Syntax — https://www.rfc-editor.org/rfc/rfc3986
- Microsoft Azure Architecture Center — Web API Design Best Practices — https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design
- Microsoft Learn — What is Uniform Interface in REST — https://learn.microsoft.com/th-th/archive/msdn-technet-forums/2827b9e0-a87c-4137-b68a-865f79632b8c
- IANA — HTTP Method Registry — https://www.iana.org/assignments/http-methods/http-methods.xhtml
- MDN Web Docs — HTTP Methods — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods
- MDN Web Docs — Content Negotiation — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Content_negotiation
- MDN Web Docs — HTTP Caching — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Caching
- OWASP — REST Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
- HATEOAS — Wikipedia — https://en.wikipedia.org/wiki/HATEOAS
- Fielding, R. T. — REST APIs must be hypertext-driven (Blog) — https://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven