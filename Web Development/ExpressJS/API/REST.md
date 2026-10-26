# REST Fundamentals — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Representational State Transfer (REST)** is an architectural style for distributed hypermedia systems, introduced by Roy Fielding in his 2000 doctoral dissertation. It is defined by a set of architectural constraints — client-server separation, statelessness, cacheability, a uniform interface, a layered system, and optional code-on-demand — that, when applied together, produce systems with predictable scalability, simplicity, and evolvability.

**Technical Definition.** REST is **not a protocol or a standard** but an architectural style: a coordinated set of constraints applied to components, connectors, and data elements within a distributed system. The central abstraction is the **resource**, identified by a URI, whose state is transferred between client and server via **representations** (typically JSON or XML documents) using a **uniform interface** — primarily the standard HTTP methods GET, POST, PUT, PATCH, and DELETE. Fielding identified **six constraints** that together define REST: client-server, stateless, cacheable, uniform interface, layered system, and code-on-demand (optional).

**Beginner-Friendly Explanation.** REST is a **style for designing web APIs** that works with the grain of the web instead of fighting it. Think of everything your API exposes as a **thing** (a resource) — a user, an order, a product. Each thing gets a **URL** (an endpoint). Clients interact with these things using **standard HTTP verbs**: `GET` to read, `POST` to create, `PUT`/`PATCH` to update, and `DELETE` to remove. The server **does not remember** anything between requests (statelessness), so every request must carry all the information needed to fulfill it. The server sends back a **representation** of the resource — usually JSON — rather than the resource itself. The payoff is an API that is predictable, cacheable, and easy to scale.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Architectural style, not a standard** | REST is a set of constraints, not a specification; “RESTful” is a matter of degree, not a binary. |
| **Resource-centric** | Everything is modeled as a **resource** identified by a URI; actions are expressed through HTTP methods, not URL verbs. |
| **Stateless** | Each request must contain all information needed to process it; the server retains no session context between requests. |
| **Uniform interface** | Resources are manipulated through a fixed set of operations (primarily HTTP methods), decoupling clients from server implementation. |
| **Representation-oriented** | Clients exchange **representations** of resource state (JSON, XML, HTML), not the resource itself. |
| **Cacheable** | Responses declare whether they may be cached, improving performance without protocol changes. |
| **Layered** | Intermediaries (load balancers, CDNs, gateways) can exist between client and server transparently. |

---

### Prerequisites

- **HTTP fundamentals**: request/response model, headers, status codes, and the semantics of `GET`, `POST`, `PUT`, `PATCH`, `DELETE`.
- **URIs and URLs**: how resources are addressed on the web.
- **JSON (or XML)**: the common representation formats used in REST APIs.
- **Basic client-server concepts**: how a client sends a request and receives a response.
- **A tool to make HTTP requests**: `curl`, Postman, or a programming language’s HTTP client.

---

### Related Programming Areas

- **Web API design**: REST is the dominant pattern for web service interfaces across the software industry.
- **Microservices architecture**: stateless REST APIs are foundational to horizontally scalable services.
- **Hypermedia (HATEOAS)**: the most demanding REST constraint, enabling self-documenting APIs.
- **API security**: HTTPS, authentication, authorization, and input validation are essential companions to REST.
- **Caching and CDNs**: REST’s cacheability constraint enables performance optimizations at every layer.

---

### Core Concepts / Features

The following seven sections cover the requested concepts. Each follows the uniform structure: definitions, purposes, syntax rules, annotated executable examples with expected outputs, and real-world cases.

---

## 1. REST

### Definitions

**Core Definition.** **REST (Representational State Transfer)** is an architectural style for distributed hypermedia systems, defined by six constraints: client-server, statelessness, cacheability, uniform interface, layered system, and optional code-on-demand.

**Technical Definition.** REST is derived by identifying and relaxing constraints that produce the scalability, simplicity, and evolvability observed in the World Wide Web. It constrains an architecture to a **client-server** model using a **stateless** communication protocol (typically HTTP), where clients and servers exchange **representations** of resources through a **uniform interface**. The uniform interface has four guiding principles: identification of resources, manipulation of resources through representations, self-descriptive messages, and hypermedia as the engine of application state (HATEOAS).

**Beginner-Friendly Explanation.** REST is the **style behind most modern web APIs**. It says: model your API as a collection of **things** (resources), give each thing a **URL**, use **standard HTTP verbs** to act on them, and make every request **self-contained** so the server does not need to remember anything. When you call `GET /users/42`, you are asking for a representation of the “user 42” resource. When you call `DELETE /users/42`, you are asking to remove it. The style is simple, but following its constraints consistently is what makes an API predictable and scalable.

### Purposes

- **To provide a scalable, simple, and evolvable architecture** for distributed systems by constraining component interactions.
- **To leverage the existing semantics of HTTP** — methods, status codes, headers — so that APIs are understood by standard tooling, caches, and intermediaries.
- **To decouple clients and servers** so that each can evolve independently.
- **To enable horizontal scalability** through statelessness and cacheability.
- **To make APIs self-documenting** through hypermedia controls (HATEOAS).

### Syntax Rules and Structure

REST is an **architectural style**, not a syntax. Its “rules” are the six constraints:

| Constraint | Rule |
|---|---|
| **Client-Server** | Separate the user interface (client) from data storage and business logic (server). |
| **Stateless** | Each request must contain all information needed to process it; the server stores no session context. |
| **Cacheable** | Responses must declare whether they may be cached by clients or intermediaries. |
| **Uniform Interface** | Resources are identified by URIs, manipulated through representations, messages are self-descriptive, and hypermedia drives application state (HATEOAS). |
| **Layered System** | Intermediaries (proxies, gateways, load balancers) may exist between client and server without either endpoint being aware. |
| **Code-on-Demand (optional)** | Servers may extend client functionality by transferring executable scripts. |

**Constraints and limitations.**

- **REST is not a protocol**; there is no conformance test. Many APIs labeled “RESTful” are **partial implementations** — most commonly level-2 HTTP APIs that omit HATEOAS.
- **HATEOAS is the most demanding constraint** and is frequently omitted in practice.
- **Statelessness does not mean no state at all**; it means the server holds no **client session state**.

### Annotated Complete Step by Step Code Examples

#### Example 1: A full CRUD cycle over HTTP (raw requests)

```http
# Step 1: Create a new post.
POST /v1/posts HTTP/1.1
Host: api.example.com
Content-Type: application/json

{ "title": "Hello REST" }

# Response:
# 201 Created
# Location: /v1/posts/8fk2
# { "id": "8fk2", "title": "Hello REST" }

# Step 2: Read the created post.
GET /v1/posts/8fk2 HTTP/1.1
Host: api.example.com

# Response:
# 200 OK
# { "id": "8fk2", "title": "Hello REST" }

# Step 3: Replace the post.
PUT /v1/posts/8fk2 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{ "title": "Hello again" }

# Response:
# 200 OK

# Step 4: Delete the post.
DELETE /v1/posts/8fk2 HTTP/1.1
Host: api.example.com

# Response:
# 204 No Content

# Step 5: Confirm deletion.
GET /v1/posts/8fk2 HTTP/1.1
Host: api.example.com

# Response:
# 404 Not Found
```

**Expected Output (conceptual):**
```
201 Created  → resource created, Location header points to it
200 OK       → resource returned
200 OK       → resource replaced
204 No Content → resource deleted
404 Not Found → resource no longer exists
```

**Why this output occurs.** The CRUD cycle maps directly to HTTP semantics: `POST` creates, `GET` reads, `PUT` replaces, and `DELETE` removes. Each request is **stateless** — the server does not remember the previous request, so the `GET` after `DELETE` correctly returns `404` because the resource no longer exists.

---

### Real-World Cases

- **Stripe, GitHub, and Shopify APIs** are RESTful APIs that use resource-oriented URLs, standard HTTP methods, and JSON representations.
- **Cloud platforms** (AWS, Azure, Google Cloud) expose REST APIs for managing compute, storage, and identity resources.
- **Microservices** commonly communicate through REST APIs because statelessness enables independent scaling and deployment.

---

## 2. Resources

### Definitions

**Core Definition.** A **resource** is the central abstraction in REST: **any conceptual entity that can be named** and identified by a URI — a document, a collection of records, a sensor reading, a user account, or a business object.

**Technical Definition.** A resource is identified by a **Uniform Resource Identifier (URI)**, which gives it a stable address independent of its current representation. Data and functionality are considered resources and are accessed using URIs. The resource itself is an abstract entity; clients interact with **representations** of the resource’s state, not the resource directly. Resources may be **collections** (e.g., `/users`), **individual items** (e.g., `/users/123`), or **singletons** (e.g., `/users/123/cart` when a user has exactly one cart).

**Beginner-Friendly Explanation.** A resource is simply **a thing your API exposes**. It could be a user, an order, a product, a blog post — anything worth naming. Each resource gets a **URL** so clients can find it. For example, `/users` is the collection of all users, and `/users/42` is the specific user with ID 42. The resource is the **idea** of the user; what the server actually sends back is a **representation** of that user (usually JSON).

### Purposes

- **To provide a stable, addressable identity** for every entity the API exposes.
- **To model domain concepts as nouns** rather than actions, keeping URLs predictable and semantically meaningful.
- **To separate identity from representation**, allowing the same resource to be served in multiple formats (JSON, XML, HTML).
- **To enable composability** through nested resources that express relationships (e.g., `/users/123/orders`).

### Syntax Rules and Structure

**General naming rules for resources (URI design).**

| Rule | Correct | Incorrect |
|---|---|---|
| **Use nouns, not verbs** | `/users` | `/getUsers` |
| **Use plural nouns for collections** | `/articles` | `/article` |
| **Use kebab-case** | `/user-profiles` | `/userProfiles`, `/user_profiles` |
| **Use lowercase** | `/orders` | `/Orders` |
| **Nest for relationships, max 2 levels** | `/users/123/orders` | `/users/123/orders/456/items/789/details` |
| **Singleton resources may be singular** | `/users/42/cart` | `/users/42/carts` |

**Component breakdown (resource URI).**

```
/users/123/orders
```

- `/users` — the **collection resource** of all users.
- `/123` — the **item resource** identifying a specific user.
- `/orders` — a **nested collection resource** representing the orders belonging to user 123.
- The relationship is expressed by **nesting**, not by query parameters or verbs.

**Constraints and limitations.**

- **Nesting deeper than two levels** often signals a resource-modeling problem; consider flattening or introducing a dedicated endpoint.
- **Verbs in paths** (e.g., `/getUsers`) duplicate the action already expressed by the HTTP method and break cacheability and predictability.
- **Resource names should be stable**; renaming a resource is a breaking change for clients.

### Annotated Complete Step by Step Code Examples

#### Example 1: Resource-oriented URL design

```http
# ✅ Correct: resources are nouns; HTTP methods are verbs.
GET    /v1/users              # List all users
GET    /v1/users/123          # Get user 123
POST   /v1/users              # Create a new user
PUT    /v1/users/123          # Replace user 123
PATCH  /v1/users/123          # Partially update user 123
DELETE /v1/users/123          # Delete user 123

# ✅ Nested relationship (max 2 levels).
GET    /v1/users/123/orders   # List orders for user 123

# ❌ Incorrect: verbs in paths.
GET    /v1/getUsers
POST   /v1/createUser
DELETE /v1/deleteUser/123
```

**Expected Output (conceptual):**
```
The correct URLs map cleanly to CRUD operations;
the incorrect URLs duplicate the action and are not cacheable or predictable.
```

**Why this output occurs.** The HTTP method already expresses the action (`GET` reads, `POST` creates, `DELETE` removes). Putting a verb in the path duplicates that information and creates separate URLs for what should be the same resource, breaking cache invalidation and API predictability.

---

### Real-World Cases

- **GitHub API**: `/repos/{owner}/{repo}/issues` — a nested collection resource representing issues belonging to a repository.
- **Stripe API**: `/v1/customers/{customer}/payment_methods` — a nested resource representing a customer’s payment methods.
- **E-commerce APIs**: `/products`, `/products/{id}`, `/carts/{cartId}/items` — collections, items, and nested relationship resources.

---

## 3. Endpoints

### Definitions

**Core Definition.** An **endpoint** is a specific **URL path** at which a resource is exposed and through which clients interact with it. It combines a **URI** (identifying the resource) with the **HTTP method** (expressing the action) and the server’s handling logic.

**Technical Definition.** An endpoint is the combination of an HTTP method and a URI path that the server routes to a handler. In REST, endpoints are **resource-oriented**: the path names the resource, and the method names the operation. A single resource URI (e.g., `/users/123`) may support multiple endpoints distinguished by method (`GET /users/123`, `PUT /users/123`, `DELETE /users/123`). Endpoints should be **predictable** — developers should be able to infer the URL patterns from examples they have already seen.

**Beginner-Friendly Explanation.** An endpoint is **a specific address your API responds to**, combined with **a specific action**. `GET /users/123` and `DELETE /users/123` are two different endpoints on the **same resource URL**. The URL says **what** you are acting on; the method says **what you are doing**. A well-designed API has predictable endpoints: once you know `/users` lists users and `/users/123` gets one user, you can guess how `/orders` and `/orders/456` work.

### Purposes

- **To expose resources at stable, predictable addresses** so clients can discover and use them.
- **To map HTTP methods to operations** on a resource, creating a uniform interface.
- **To enable routing and versioning** by organizing endpoints under versioned paths (e.g., `/v1/users`).
- **To support nested relationships** through structured paths that reflect resource ownership.

### Syntax Rules and Structure

**General endpoint structure.**

```
{METHOD} /{version}/{collection}/{id}/{sub-collection}
```

**Component breakdown.**

```
GET /v1/users/123/orders
```

- `GET` — the **HTTP method** (the action).
- `/v1` — the **version prefix** (optional but recommended).
- `/users` — the **collection resource**.
- `/123` — the **item identifier**.
- `/orders` — the **nested collection** representing a relationship.

**Syntax rules.**

- **Use nouns in paths, not verbs**; the method is the verb.
- **Use plural nouns for collections**; item identifiers follow the collection.
- **Use kebab-case and lowercase** for multi-word path segments.
- **Limit nesting to two levels**; deeper nesting suggests a modeling problem.
- **Version in the path** (e.g., `/v1/`) or via headers; path versioning is common and explicit.

**Constraints and limitations.**

- **A single resource URL may have multiple endpoints** (one per method); do not create separate URLs for each operation.
- **Action endpoints** (e.g., `/users/search`, `/orders/bulk-delete`) are acceptable when an operation cannot be expressed as a resource update, but should be used sparingly.
- **Endpoint changes are breaking changes**; versioning is required to evolve them safely.

### Annotated Complete Step by Step Code Examples

#### Example 1: Predictable endpoint patterns

```http
# Collection endpoints
GET    /v1/users              # List users
POST   /v1/users              # Create a user

# Item endpoints
GET    /v1/users/123          # Get user 123
PUT    /v1/users/123          # Replace user 123
PATCH  /v1/users/123          # Partially update user 123
DELETE /v1/users/123          # Delete user 123

# Nested collection endpoints
GET    /v1/users/123/orders   # List orders for user 123
POST   /v1/users/123/orders   # Create an order for user 123

# Action endpoint (used sparingly)
POST   /v1/users/search       # Search users (cannot be expressed as a resource update)
```

**Expected Output (conceptual):**
```
Predictable patterns: /users → collection, /users/123 → item,
/users/123/orders → nested collection. Developers can infer new endpoints.
```

**Why this output occurs.** The patterns are consistent: plural nouns for collections, item IDs as path segments, and nesting for relationships. Once a developer learns the `/users` pattern, they can correctly guess `/orders` and `/orders/456` without reading documentation.

---

### Real-World Cases

- **Twitter/X API**: `/2/tweets/{id}`, `/2/users/{id}/tweets` — versioned, resource-oriented endpoints with nested relationships.
- **Stripe API**: `/v1/charges`, `/v1/charges/{charge_id}` — collection and item endpoints with versioned paths.
- **Google API Design Guide**: recommends standard methods (`get`, `list`, `create`, `update`, `delete`) mapped to predictable endpoint patterns.

---

## 4. HTTP Verbs

### Definitions

**Core Definition.** **HTTP verbs** (also called **methods**) are the standard operations defined by the HTTP protocol that express **what action** is being performed on a resource. In REST, they form the **uniform interface** through which resources are manipulated.

**Technical Definition.** The primary HTTP methods in REST are **GET**, **POST**, **PUT**, **PATCH**, and **DELETE**. Each has defined semantics regarding **safety** (whether it modifies server state) and **idempotency** (whether repeating the request produces the same result). RFC 9110 defines these semantics: **GET** is safe and idempotent; **POST** is neither safe nor idempotent; **PUT** is idempotent but not safe; **PATCH** is neither safe nor necessarily idempotent; **DELETE** is idempotent but not safe. **HEAD** and **OPTIONS** are also safe and idempotent.

**Beginner-Friendly Explanation.** HTTP verbs are the **standard actions** you can perform on a resource. `GET` means “read this.” `POST` means “create something new.” `PUT` means “replace this entirely.” `PATCH` means “update part of this.” `DELETE` means “remove this.” The key difference between `PUT` and `PATCH` is that `PUT` replaces the whole resource, while `PATCH` only changes the fields you send. The key difference between `POST` and `PUT` is that `POST` creates a **new** resource (usually with a server-assigned ID), while `PUT` creates or replaces a resource at a **known URL**.

### Purposes

- **To provide a uniform, standardized interface** for manipulating resources, decoupling clients from server implementation.
- **To communicate intent clearly** — the method tells clients, caches, and intermediaries what the request does.
- **To enable safe retries** — idempotent methods can be retried without fear of duplicating side effects.
- **To support caching and safety guarantees** — safe methods (`GET`, `HEAD`) do not modify state and can be cached.

### Syntax Rules and Structure

**HTTP method semantics (RFC 9110).**

| Method | CRUD Role | Safe | Idempotent | Request Body | Typical Status |
|---|---|---|---|---|---|
| **GET** | Read | Yes | Yes | No | `200 OK` |
| **POST** | Create | No | No | Yes | `201 Created` |
| **PUT** | Replace | No | Yes | Yes | `200 OK` / `204 No Content` |
| **PATCH** | Partial update | No | Not guaranteed | Yes | `200 OK` / `204` |
| **DELETE** | Remove | No | Yes | Optional | `204 No Content` |
| **HEAD** | Same as GET without body | Yes | Yes | No | `200 OK` |
| **OPTIONS** | Describe options | Yes | Yes | No | `200 OK` |

**Component breakdown (request line).**

```
GET /v1/users/123 HTTP/1.1
```

- `GET` — the **method** (the verb).
- `/v1/users/123` — the **request target** (the resource endpoint).
- `HTTP/1.1` — the **protocol version**.

**Syntax rules.**

- **Use `GET` for read-only operations**; never use it for operations that change data.
- **Use `POST` to create new resources** when the server assigns the identifier; return `201 Created` with a `Location` header.
- **Use `PUT` for full replacement**; the request body must contain the complete representation.
- **Use `PATCH` for partial updates**; the body contains only the fields to change.
- **Use `DELETE` to remove a resource**; return `204 No Content` on success.

**Constraints and limitations.**

- **`POST` is not idempotent**; retrying a `POST` may create duplicate resources.
- **`PATCH` idempotency is not guaranteed**; it depends on the patch design.
- **`GET` must not have side effects**; violating this breaks caching, prefetching, and safety guarantees.
- **Some proxies and firewalls restrict `PUT` and `DELETE`**; some APIs provide `POST` alternatives for compatibility.

### Annotated Complete Step by Step Code Examples

#### Example 1: Correct method usage for a CRUD resource

```http
# Step 1: Create a resource with POST.
POST /v1/users HTTP/1.1
Content-Type: application/json

{ "name": "Alice", "email": "alice@example.com" }

# Response:
# 201 Created
# Location: /v1/users/42

# Step 2: Read the resource with GET.
GET /v1/users/42 HTTP/1.1

# Response:
# 200 OK
# { "id": 42, "name": "Alice", "email": "alice@example.com" }

# Step 3: Replace the resource with PUT.
PUT /v1/users/42 HTTP/1.1
Content-Type: application/json

{ "name": "Alice Smith", "email": "alice.smith@example.com" }

# Response:
# 200 OK

# Step 4: Partially update with PATCH.
PATCH /v1/users/42 HTTP/1.1
Content-Type: application/json

{ "email": "new@example.com" }

# Response:
# 200 OK

# Step 5: Delete with DELETE.
DELETE /v1/users/42 HTTP/1.1

# Response:
# 204 No Content
```

**Expected Output (conceptual):**
```
201 Created  → user created with Location header
200 OK       → user returned
200 OK       → user fully replaced
200 OK       → email partially updated
204 No Content → user deleted
```

**Why this output occurs.** Each method is used according to its semantics: `POST` creates, `GET` reads, `PUT` replaces the full representation, `PATCH` modifies a subset of fields, and `DELETE` removes the resource. `POST` returns `201` with a `Location` header; `DELETE` returns `204` because there is no response body.

---

#### Example 2: Idempotency and safety in practice

```http
# Idempotent: repeating PUT produces the same result.
PUT /v1/users/42 HTTP/1.1
{ "name": "Alice", "email": "alice@example.com" }

PUT /v1/users/42 HTTP/1.1
{ "name": "Alice", "email": "alice@example.com" }

# Both requests result in the same server state: one user with these values.

# Not idempotent: repeating POST may create duplicates.
POST /v1/users HTTP/1.1
{ "name": "Bob", "email": "bob@example.com" }

POST /v1/users HTTP/1.1
{ "name": "Bob", "email": "bob@example.com" }

# Two separate users may be created (duplicate side effect).
```

**Expected Output (conceptual):**
```
PUT repeated → same state (idempotent)
POST repeated → possibly two users (not idempotent)
```

**Why this output occurs.** `PUT` is idempotent because it replaces the resource at a known URL; repeating the request does not change the outcome. `POST` is not idempotent because each request may create a new resource with a server-assigned identifier, leading to duplicates on retry.

---

### Real-World Cases

- **Stripe API**: uses `POST` for creating charges (non-idempotent) and supports **idempotency keys** to make retried `POST` requests safe.
- **GitHub API**: uses `GET` for reads, `POST` for creating issues, `PATCH` for partial updates, and `DELETE` for removal.
- **RFC 9110** is the authoritative specification for HTTP method semantics and is the reference for safe and idempotent behavior.

---

## 5. Statelessness

### Definitions

**Core Definition.** **Statelessness** is a REST constraint requiring that **each client request contain all the information needed to process it**, and that the server **retain no session context** between requests. The server stores only **resource state**, not **client session state**.

**Technical Definition.** In a stateless REST API, the server does not remember who the client is or what they did previously. Any session-related information (authentication tokens, pagination cursors, filters) must be supplied by the client with **every request**. Statefulness would require the server to maintain session data in memory or storage, complicating horizontal scaling, failure recovery, and security. The term “state” in REST refers to the **state of the resource**, not the state of a session.

**Beginner-Friendly Explanation.** Statelessness means **the server has no memory of you between requests**. Every request must carry everything needed: who you are (a token), what you want, and any context (pagination, filters). The server does not store a session for you. This is why you send an authentication token with **every** request rather than logging in once and having the server remember you. The benefit is huge: any server can handle any request, so you can add more servers without users losing their sessions.

### Purposes

- **To enable horizontal scalability** — any server instance can handle any request because no session affinity is required.
- **To simplify failure recovery** — if a server crashes, another can serve the next request without losing session state.
- **To improve reliability and reduce complexity** — the server does not need to manage, synchronize, or secure session storage.
- **To support safe retries** — clients can retry idempotent requests without worrying about session state.

### Syntax Rules and Structure

**General pattern: stateless authentication.**

```http
GET /v1/users/me HTTP/1.1
Host: api.example.com
Authorization: Bearer <jwt-token>
```

**Component breakdown.**

- `Authorization: Bearer <jwt-token>` — the **self-contained credential** carrying identity and claims.
- The server validates the token on **every request**; it does not rely on a server-side session.
- The token itself contains the necessary claims (user ID, roles, expiry), so no session lookup is required.

**Statelessness rules.**

- **Every request must be self-contained**: include authentication, resource identifiers, and any context (pagination, filters).
- **Do not rely on server-side sessions** for REST APIs; session state belongs on the client.
- **Use tokens (JWT, OAuth 2.0)** that carry identity and claims, rather than session cookies tied to server memory.
- **Pass state in the request** (headers, query parameters, body) rather than assuming the server remembers previous interactions.

**Constraints and limitations.**

- **Statelessness does not forbid server-side storage**; it forbids storing **client session state** on the server. Resource data is still stored.
- **Passing state from client to backend while making the service technically stateless can be an anti-pattern** if it enables replay or impersonation attacks.
- **Some operations inherently require state** (e.g., multi-step transactions); these should be modeled as resources rather than server sessions.

### Annotated Complete Step by Step Code Examples

#### Example 1: Stateless authentication with a bearer token

```http
# Step 1: Client obtains a token (out of band, e.g., OAuth flow).
# Token: eyJhbGciOiJIUzI1NiIs...

# Step 2: Every request carries the token — no server session.
GET /v1/users/me HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

# Response:
# 200 OK
# { "id": 42, "name": "Alice" }

# Step 3: The next request is independent — same token, no session.
GET /v1/orders HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

# Response:
# 200 OK
# [ { "id": 1001, "status": "shipped" } ]
```

**Expected Output (conceptual):**
```
Both requests succeed independently; the server stores no session between them.
```

**Why this output occurs.** Each request includes the `Authorization` header with a self-contained token. The server validates the token on every request and does not rely on a server-side session. The second request is processed by a potentially different server instance without any session affinity.

---

#### Example 2: Contrast — stateful session (non-RESTful)

```http
# Step 1: Client logs in; server creates a session and returns a cookie.
POST /login HTTP/1.1
{ "username": "alice", "password": "..." }

# Response:
# 200 OK
# Set-Cookie: sessionId=abc123

# Step 2: Client sends the cookie; server looks up session state.
GET /v1/users/me HTTP/1.1
Cookie: sessionId=abc123

# The server must now remember that sessionId=abc123 belongs to Alice.
# If the server restarts, the session is lost.
```

**Expected Output (conceptual):**
```
Stateful: server must store session data; scaling requires session replication or sticky sessions.
```

**Why this output occurs.** The server creates and stores session state (`sessionId=abc123` → Alice). This violates statelessness: the server must remember the client between requests. If the server restarts or a different instance handles the next request, the session may be lost.

---

### Real-World Cases

- **JWT-based APIs**: use self-contained tokens so the server validates identity on every request without session storage.
- **OAuth 2.0 bearer tokens**: the standard mechanism for stateless authentication in REST APIs.
- **Serverless and containerized deployments**: statelessness is essential because any instance may handle any request, and instances may be ephemeral.

---

## 6. Representations

### Definitions

**Core Definition.** A **representation** is a concrete **document or data format** that captures the **current or intended state** of a resource. Clients exchange representations — not the resources themselves — with the server.

**Technical Definition.** A resource is an abstract entity identified by a URI; its **representation** is a sequence of bytes in a specific format (JSON, XML, HTML, plain text, image, etc.) along with metadata (headers) describing the format and how to process it. The same resource may have **multiple representations**; content negotiation (`Accept` and `Content-Type` headers) allows clients and servers to agree on the format. The separation between resource identity and representation is what the name “**representational** state transfer” describes.

**Beginner-Friendly Explanation.** A representation is **what the server actually sends back** when you request a resource. The resource is the abstract “user 42”; the representation is the JSON object `{ "id": 42, "name": "Alice" }` that describes that user. The same user could be represented as XML, HTML, or CSV if the client asks for it. The server sends a **snapshot** of the resource’s state in a format the client can understand.

### Purposes

- **To decouple resource identity from data format**, allowing the same resource to be served in multiple formats.
- **To enable content negotiation**, so clients can request the representation format they prefer.
- **To support self-descriptive messages**, where headers describe the representation’s type and processing instructions.
- **To allow caching and transformation** by intermediaries that understand the representation’s media type.

### Syntax Rules and Structure

**General pattern: content negotiation with headers.**

```http
GET /v1/users/42 HTTP/1.1
Accept: application/json
```

```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 42

{ "id": 42, "name": "Alice" }
```

**Component breakdown.**

- `Accept: application/json` — the client’s **preferred representation format**.
- `Content-Type: application/json` — the server’s **declaration of the response representation format**.
- `Content-Length` — the size of the representation body.
- The body — the **representation** itself (JSON, XML, etc.).

**Representation rules.**

- **Every response must declare its representation format** via `Content-Type`.
- **Clients may request alternative formats** via `Accept`; servers should honor it when supported.
- **The same resource may have multiple representations**; do not create separate resource URLs for each format.
- **Representations are state snapshots**, not the resource itself; they may be stale or transformed by intermediaries.

**Constraints and limitations.**

- **Not all formats are equally supported**; servers may return `406 Not Acceptable` if no requested format is available.
- **Representation size affects performance**; pagination and field filtering reduce payload size.
- **Representations may include metadata** (links, pagination cursors) beyond the resource’s own fields.

### Annotated Complete Step by Step Code Examples

#### Example 1: JSON and XML representations of the same resource

```http
# Step 1: Request JSON representation.
GET /v1/users/42 HTTP/1.1
Accept: application/json

# Response:
# 200 OK
# Content-Type: application/json
# { "id": 42, "name": "Alice", "email": "alice@example.com" }

# Step 2: Request XML representation of the same resource.
GET /v1/users/42 HTTP/1.1
Accept: application/xml

# Response:
# 200 OK
# Content-Type: application/xml
# <user>
#   <id>42</id>
#   <name>Alice</name>
#   <email>alice@example.com</email>
# </user>
```

**Expected Output (conceptual):**
```
The same resource (/users/42) is represented as JSON or XML
depending on the Accept header — the resource identity does not change.
```

**Why this output occurs.** The resource (`/users/42`) is the same in both requests. The `Accept` header tells the server which representation format the client prefers. The server returns the appropriate format and declares it via `Content-Type`. This is **content negotiation**, and it demonstrates the separation between resource identity and representation.

---

### Real-World Cases

- **GitHub API**: returns JSON by default but supports `Accept: application/vnd.github+json` for versioned representations.
- **Django REST Framework**: supports JSON, XML, and browsable HTML representations through content negotiation.
- **CDNs and caching proxies**: use `Content-Type` and cache headers to store and serve representations efficiently.

---

## 7. REST Best Practices

### Definitions

**Core Definition.** **REST best practices** are empirically established guidelines for designing REST APIs that are predictable, consistent, secure, scalable, and maintainable, while adhering to REST’s architectural constraints.

**Technical Definition.** Best practices encompass **resource naming** (nouns, plural, kebab-case, limited nesting), **correct HTTP method usage** (safety and idempotency semantics), **meaningful status codes**, **versioning from day one**, **stateless authentication** (JWT/OAuth), **pagination and filtering**, **standardized error responses**, **idempotency design**, **HTTPS everywhere**, and **observability** (logging, metrics, request IDs). They bridge the gap between REST’s abstract constraints and production realities, where most APIs are partial implementations that omit HATEOAS.

**Beginner-Friendly Explanation.** Best practices are **the habits that make REST APIs pleasant to use**. Use nouns in URLs, not verbs. Use the right HTTP method for each operation. Return meaningful status codes so clients know what happened. Version your API from the start so you can change it without breaking clients. Use HTTPS and tokens for security. Paginate large results. Return consistent error shapes. Design for idempotency so retries do not create duplicates. These are the things that separate an API developers enjoy using from one they dread.

### Purposes

- **To make APIs predictable and easy to learn** through consistent naming and method usage.
- **To prevent breaking changes** by versioning from the start.
- **To protect data and users** through HTTPS, authentication, and authorization.
- **To ensure reliability** through idempotency, meaningful status codes, and standardized errors.
- **To enable observability and debugging** through logs, metrics, and request IDs.

### Syntax Rules and Structure

**Best-practice checklist.**

| Area | Rule |
|---|---|
| **Resource naming** | Use nouns, plural collections, kebab-case, max 2 nesting levels. |
| **HTTP methods** | `GET` read, `POST` create, `PUT` replace, `PATCH` partial update, `DELETE` remove. |
| **Status codes** | `200` success, `201` created, `204` no content, `400` bad request, `401` unauthorized, `403` forbidden, `404` not found, `500` server error. |
| **Versioning** | Version from day one; use `/v1/` path prefix or headers. |
| **Security** | HTTPS only; authenticate with JWT or OAuth 2.0 bearer tokens. |
| **Pagination** | Paginate large collections; use `limit`/`offset` or cursor-based pagination. |
| **Errors** | Standardize error shapes; include a machine-readable code and human-readable message. |
| **Idempotency** | Design `PUT` and `DELETE` to be idempotent; support idempotency keys for `POST`. |
| **Observability** | Log requests, emit metrics, and propagate request IDs. |

**Constraints and limitations.**

- **Best practices are conventions**, not enforced by the REST style itself; they must be applied deliberately.
- **HATEOAS is frequently omitted** in production APIs; most “RESTful” APIs are level-2 HTTP APIs.
- **Versioning adds complexity**; choose a strategy (path, header, or media type) and apply it consistently.

### Annotated Complete Step by Step Code Examples

#### Example 1: A well-designed REST endpoint

```http
# Step 1: Create a resource with proper naming and methods.
POST /v1/users HTTP/1.1
Content-Type: application/json
Authorization: Bearer <token>

{ "name": "Alice", "email": "alice@example.com" }

# Response:
# 201 Created
# Location: /v1/users/42
# { "id": 42, "name": "Alice", "email": "alice@example.com" }

# Step 2: Read with pagination and filtering.
GET /v1/users?limit=20&offset=0&role=admin HTTP/1.1
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

# Step 4: Standardized error response.
GET /v1/users/999 HTTP/1.1

# Response:
# 404 Not Found
# { "error": { "code": "USER_NOT_FOUND", "message": "User 999 does not exist" } }
```

**Expected Output (conceptual):**
```
201 Created with Location header
200 OK with paginated data and metadata
200 OK for partial update
404 Not Found with a standardized error body
```

**Why this output occurs.** The example applies the key best practices: resource-oriented URLs, correct methods, `201` with `Location` for creation, pagination metadata, `PATCH` for partial updates, and a consistent error shape with a machine-readable code.

---

### Real-World Cases

- **Stripe API**: uses versioned paths (`/v1/`), resource-oriented URLs, standardized error objects, idempotency keys, and pagination.
- **GitHub API**: uses `/repos/{owner}/{repo}` patterns, consistent status codes, and hypermedia links for pagination.
- **Google API Design Guide**: recommends standard methods, resource naming, versioning, and error models that align with these best practices.

---

## References

- Fielding, R. T. — Architectural Styles and the Design of Network-based Software Architectures (Doctoral Dissertation, 2000) - https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm
- IEEE Technology Navigator — Representational State Transfer (REST) - https://technav.ieee.org/topic/representational-state-transfer/
- Jakarta EE — Building RESTful Web Services with Jakarta REST - https://jakarta.ee/learn/jakartaee-tutorial/9.1/websvcs/rest/rest.html
- Oracle — What Are RESTful Web Services? - https://docs.oracle.com/javaee/6/tutorial/doc/gijqy.html
- OWASP — REST Security Cheat Sheet - https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
- IETF — RFC 9110: HTTP Semantics (Methods, Safe, Idempotent) - https://datatracker.ietf.org/doc/html/rfc9110
- Postman Blog — REST API Best Practices: A Developer’s Guide - https://blog.postman.com/rest-api-best-practices/
- Strapi — Master RESTful API Design with Best Practices - https://strapi.io/blog/restful-api-design-guide-principles-best-practices
- Back4app — What is a REST API? Constraints, Methods, Status Codes - https://www.back4app.com/glossary/rest-api/
- API Design Standards (Universal Dev Standards) — Resource Naming, HTTP Verbs, Status Codes - https://raw.githubusercontent.com/AsiaOstrich/universal-dev-standards/e7db7ca46667404caf718817b754fb4fe9a4ca60/core/api-design-standards.md
- Apidog — REST API Naming Conventions: A Practical Style Guide - https://apidog.com/blog/rest-api-naming-conventions/
- EBI — What is REST? - https://www.ebi.ac.uk/seqdb/confluence/download/temp/pdfexport-20240721-210724-1121-1193/JDSAT-68165098-210724-1121-1194.pdf
- RFC 7807 — Problem Details for HTTP APIs - https://datatracker.ietf.org/doc/html/rfc7807
- Google API Design Guide - https://cloud.google.com/apis/design
- OpenAPI Specification 3.x - https://spec.openapis.org/oas/v3.1.0