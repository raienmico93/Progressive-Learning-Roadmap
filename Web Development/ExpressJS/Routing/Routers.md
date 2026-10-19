# Express.js Router — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
`express.Router()` is a class in Express.js used to create modular, mountable route handler objects. A router instance is a complete middleware and routing system—often described as a “mini-application”—capable of defining routes, applying middleware, and being mounted onto an Express application or another router.

**Technical Definition**
The `express.Router()` constructor returns a `router` object that functions as both a middleware function and a routing table. It maintains its own stack of middleware and routes, independent of the main application. A router can be mounted at a specific path using `app.use(path, router)`, at which point all routes defined within the router are prefixed with that mount path. Routers support their own `.get()`, `.post()`, `.use()`, `.param()`, and other methods, and they can be nested indefinitely. The `Router()` method accepts an optional `options` object with `caseSensitive`, `mergeParams`, and `strict` settings.

**Beginner-Friendly Explanation**
A router is like a mini-Express app that you can plug into your main app. Instead of putting every route in one giant file, you create separate routers for different parts of your application—like one for users, one for products, and one for orders. Each router handles its own routes, and you attach it to the main app at a specific URL prefix. This keeps your code organized and makes it easy to reuse or move parts of your application.

---

### Key Characteristics

- **Modular and mountable** — Routers encapsulate routes and middleware into standalone units that can be mounted at any path.
- **Middleware-compatible** — A router is itself middleware, so it can be passed to `app.use()` or another router’s `.use()` method.
- **Independent parameter handling** — `router.param()` callbacks are local to the router and are not inherited by mounted sub-routers unless explicitly configured.
- **Configurable behavior** — The `caseSensitive`, `strict`, and `mergeParams` options control how paths and parameters are matched.
- **Hierarchy-supporting** — Routers can be nested to reflect resource hierarchies, with `mergeParams` enabling access to parent route parameters.

---

### Prerequisites

- Node.js and npm installed; Express 5 requires Node.js 18 or higher.
- An Express application initialized with `const app = express()`.
- Basic understanding of routing methods (`app.get()`, `app.post()`) and middleware signatures `(req, res, next)`.
- Familiarity with route parameters and the `req.params` object.

---

### Related Programming Areas

- **Routing** — Routers are the primary tool for organizing route definitions in large applications.
- **Middleware** — Router-level middleware runs only for routes within that router, providing isolated functionality.
- **Route parameters** — `router.param()` enables shared validation and transformation for router-specific parameters.
- **Project architecture** — Feature-based folder structures rely on routers to separate concerns.
- **Error handling** — Error-handling middleware can be mounted at the router level to handle errors from that router’s routes.

---

## Core Concept 1: `express.Router()`

### Definitions

**Core Definition**
`express.Router()` is a factory function that creates a new router instance—a self-contained middleware and routing system that can be mounted onto an Express application or another router.

**Technical Definition**
The top-level `express` object exposes a `Router()` method that returns a `router` object. This object is an instance of `express.Router` and behaves as a middleware function and a complete routing system. Every Express application has a built-in application router (`app.router`), which is created lazily on first access. User-created routers operate independently and can be mounted at any path, at which point their internal routes are prefixed with the mount path.

**Beginner-Friendly Explanation**
`express.Router()` is the function you call to create a new mini-app. You call it once per feature or module—like `const userRouter = express.Router()`—and then define routes on it just like you would on the main `app`.

---

### Purposes

- To create a self-contained routing system that can be developed, tested, and mounted independently.
- To provide a reusable unit of routing logic that can be applied at different mount paths.
- To enable separation of routing concerns from application bootstrap code.
- To support the construction of hierarchical route structures through nesting.

---

### Syntax Rules and Structure

#### Complete General Syntax: Creating a Router

```javascript
const express = require('express');
const router = express.Router(options);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `express` | The top-level Express module. |
| `Router(options)` | Factory function that returns a new router instance. |
| `options` (optional) | Configuration object with `caseSensitive`, `mergeParams`, and `strict` properties. |

#### Syntax Rules

- `express.Router()` must be called to create a new router; the result is both a middleware function and a routing table.
- The `options` object is optional; defaults are `caseSensitive: false`, `strict: false`, and `mergeParams: false`.
- A router instance exposes the same HTTP method routing methods as an application (`router.get()`, `router.post()`, etc.) plus `router.use()` and `router.param()`.

#### Constraints and Limitations

- A router is not a full application; it lacks application-level settings such as template engine configuration and `app.set()`.
- Routers do not automatically inherit `app.param()` callbacks or application-level middleware defined after the router is mounted.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Creating a Basic Router

```javascript
const express = require('express');
const app = express();
const router = express.Router();

// Define routes on the router
router.get('/', (req, res) => {
  res.send('Router root');
});

router.get('/about', (req, res) => {
  res.send('Router about');
});

// Mount the router on the app
app.use('/demo', router);

app.listen(3000);
```

**Step-by-Step Explanation**

1. **Create the router** — `express.Router()` returns a new router instance.
2. **Define routes** — `router.get()` registers handlers on the router’s internal stack.
3. **Mount the router** — `app.use('/demo', router)` prefixes all router routes with `/demo`.

**Expected Output**

```
GET /demo        → 200 "Router root"
GET /demo/about  → 200 "Router about"
GET /            → 404 (not defined on the router)
```

**Why This Output Occurs**

- The router’s `/` route is relative to the mount path `/demo`, so it becomes `/demo`.
- The main app has no route for `/`, so it returns `404`.

---

### Real-World Cases

- **Feature modules** — Each feature (users, products, orders) exports its own router.
- **API versioning** — A version-specific router (`/api/v1`) groups all v1 endpoints.
- **Admin panels** — A dedicated admin router with its own authentication middleware.

---

## Core Concept 2: Router Creation

### Definitions

**Core Definition**
Router creation is the process of instantiating a router object using `express.Router()` and configuring its behavior through the optional `options` argument.

**Technical Definition**
Router creation accepts an `options` object that configures three behaviors: `caseSensitive` (enables case-sensitive route matching, disabled by default), `strict` (enables strict routing, where `/foo` and `/foo/` are treated differently, disabled by default), and `mergeParams` (preserves `req.params` values from the parent router, with child values taking precedence on conflicts). The created router is independent and must be explicitly mounted to become active.

**Beginner-Friendly Explanation**
When you create a router, you can pass a few settings to change how it matches URLs. For example, you can make it case-sensitive so `/Users` and `/users` are different, or make it strict so a trailing slash matters. You can also tell it to remember parameters from the parent route when you nest routers.

---

### Purposes

- To configure router-level matching behavior independently of the main application.
- To enable case-sensitive or strict routing for specific route groups.
- To allow nested routers to access parent route parameters via `mergeParams`.
- To create multiple routers with different behaviors in the same application.

---

### Syntax Rules and Structure

#### Complete General Syntax: Router with Options

```javascript
const router = express.Router({
  caseSensitive: true,
  strict: true,
  mergeParams: true
});
```

**Component Breakdown**

| Option | Default | Effect |
|--------|---------|--------|
| `caseSensitive` | `false` | When `true`, `/Foo` and `/foo` are different routes. |
| `strict` | `false` | When `true`, `/foo` and `/foo/` are different routes. |
| `mergeParams` | `false` | When `true`, preserves parent router `req.params` values. |

#### Syntax Rules

- Options are passed as a single object to `express.Router()`.
- `mergeParams` is essential when a child router is mounted on a path containing parameters.
- `caseSensitive` and `strict` affect only the router on which they are set; they are not inherited by sub-routers.

#### Constraints and Limitations

- `mergeParams` can cause parameter name conflicts; the child’s value takes precedence, which may silently override parent values.
- Enabling `strict` may break clients that expect trailing slashes to be ignored.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Case-Sensitive and Strict Router

```javascript
const express = require('express');
const app = express();

const strictRouter = express.Router({ caseSensitive: true, strict: true });

strictRouter.get('/Data', (req, res) => res.send('Case-sensitive match'));

app.use('/api', strictRouter);

app.listen(3000);
```

**Expected Output**

```
GET /api/Data   → 200 "Case-sensitive match"
GET /api/data   → 404 (case-sensitive)
GET /api/Data/  → 404 (strict routing)
```

**Why This Output Occurs**

- `caseSensitive: true` makes `/Data` distinct from `/data`.
- `strict: true` requires an exact match, so the trailing slash breaks the match.

---

### Real-World Cases

- **Legacy API compatibility** — Strict routing to preserve exact URL behavior.
- **Multi-tenant routing** — `mergeParams` to pass tenant IDs from parent to child routers.
- **Public vs. internal APIs** — Different routers with different case-sensitivity settings.

---

## Core Concept 3: Router Mounting

### Definitions

**Core Definition**
Router mounting is the process of attaching a router instance to an Express application or another router at a specified path, causing the router’s routes to be prefixed with that mount path.

**Technical Definition**
Mounting is performed with `app.use(path, router)` or `parentRouter.use(path, childRouter)`. The `path` argument is optional and defaults to `/`. When mounted, the router behaves as middleware: every request whose URL begins with the mount path is passed to the router, which then strips the mount path and matches the remaining path against its internal routes. Mounting order is significant; routers mounted earlier receive requests first.

**Beginner-Friendly Explanation**
Mounting is how you plug a router into your app. You say, “Take this router and attach it at `/api/users`.” Then any request starting with `/api/users` is handed to that router, and the router looks at the rest of the URL to decide which of its routes matches.

---

### Purposes

- To connect a router’s routes to the application’s URL space.
- To namespace routes under a common prefix (e.g., `/api/v1`).
- To compose multiple routers into a single application or a larger router hierarchy.
- To apply router-level middleware selectively to a subset of URLs.

---

### Syntax Rules and Structure

#### Complete General Syntax: Mounting a Router

```javascript
app.use('/api/users', userRouter);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `app.use()` | Mounts middleware (including routers) at a path. |
| `'/api/users'` | Mount path; all router routes are prefixed with this. |
| `userRouter` | The router instance to mount. |

#### Syntax Rules

- The mount path is optional; omitting it mounts the router at `/`.
- Mounting order matters: routers mounted earlier are matched first.
- Multiple routers can be mounted at the same path; each will be tried in order.

#### Constraints and Limitations

- If a router does not match a request, it calls `next()` and the request continues to subsequent middleware, which may lead to a `404`.
- Mounting a router at the same path as another router can cause unexpected shadowing if route patterns overlap.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Mounting Multiple Routers

```javascript
const express = require('express');
const app = express();

const userRouter = express.Router();
userRouter.get('/', (req, res) => res.send('User list'));
userRouter.get('/:id', (req, res) => res.send(`User ${req.params.id}`));

const productRouter = express.Router();
productRouter.get('/', (req, res) => res.send('Product list'));

app.use('/api/users', userRouter);
app.use('/api/products', productRouter);

app.listen(3000);
```

**Expected Output**

```
GET /api/users       → 200 "User list"
GET /api/users/42    → 200 "User 42"
GET /api/products    → 200 "Product list"
GET /api/orders      → 404
```

**Why This Output Occurs**

- Each router is mounted at a distinct path prefix; requests are routed to the appropriate router based on the prefix.

---

### Real-World Cases

- **API namespacing** — `/api/v1/users`, `/api/v1/products`.
- **Admin isolation** — `/admin` mounted with its own authentication middleware.
- **Static file separation** — Mounting routers for different asset groups.

---

## Core Concept 4: Router Middleware

### Definitions

**Core Definition**
Router middleware is middleware applied to a router instance, executing only for requests that are handled by that router’s routes.

**Technical Definition**
Router-level middleware is registered with `router.use()` or `router.METHOD()`. It behaves identically to application-level middleware, except that it is bound to a specific router instance. The order of `router.use()` calls is critical: middleware is invoked sequentially, and each must call `next()` to pass control to the next middleware or route handler. Middleware can be mounted at a specific path or at the router’s root.

**Beginner-Friendly Explanation**
Router middleware is code that runs only for routes inside a specific router. For example, you can add a logging function to your user router so every user-related request gets logged, without affecting product routes. You register it with `router.use()`, and it runs before the router’s route handlers.

---

### Purposes

- To apply middleware selectively to a group of routes within a router.
- To share preprocessing logic (logging, authentication, validation) across related endpoints.
- To isolate cross-cutting concerns to the router level rather than the application level.
- To control middleware execution order within a router’s stack.

---

### Syntax Rules and Structure

#### Complete General Syntax: Router Middleware

```javascript
router.use((req, res, next) => {
  // middleware logic
  next();
});

router.use('/protected', authMiddleware);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `router.use()` | Registers middleware on the router. |
| Optional path | Limits middleware to paths beginning with that prefix. |
| `next()` | Passes control to the next middleware or route. |

#### Syntax Rules

- Middleware must call `next()` (or send a response) to avoid hanging the request.
- `router.use()` middleware applies to all HTTP methods and all paths that begin with the optional mount path.
- The order of middleware registration determines execution order.

#### Constraints and Limitations

- Router-level middleware does not apply to routes outside the router.
- Middleware mounted after a route handler will not run for that handler’s requests.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Router-Specific Logging Middleware

```javascript
const express = require('express');
const app = express();

const userRouter = express.Router();

// Router-specific middleware
userRouter.use((req, res, next) => {
  console.log(`[USERS] ${req.method} ${req.url}`);
  next();
});

userRouter.get('/', (req, res) => res.send('User list'));
userRouter.get('/:id', (req, res) => res.send(`User ${req.params.id}`));

app.use('/users', userRouter);

app.listen(3000);
```

**Expected Output**

```
GET /users
→ Console: [USERS] GET /
→ 200 "User list"

GET /users/42
→ Console: [USERS] GET /42
→ 200 "User 42"

GET /products
→ 404 (no log, because the middleware is router-specific)
```

**Why This Output Occurs**

- The middleware runs only for requests handled by `userRouter`; requests to other paths never enter the router.

---

### Real-World Cases

- **Authentication gates** — A router-wide `authMiddleware` protects all routes in an admin router.
- **Request logging** — Feature-specific logs for debugging.
- **Rate limiting** — Applying a rate limiter only to authentication routes.

---

## Core Concept 5: Router Parameters

### Definitions

**Core Definition**
Router parameters are callbacks registered with `router.param()` that execute when a named route parameter is present in a route handled by that router, enabling shared validation, loading, or transformation logic.

**Technical Definition**
`router.param(name, callback)` adds a callback trigger for the named parameter. The callback receives `(req, res, next, value, name)`, where `value` is the raw parameter value and `name` is the parameter’s name. Unlike `app.param()`, `router.param()` does **not** accept an array of parameter names. Param callbacks are local to the router on which they are defined; they are not inherited by mounted sub-routers, nor are they triggered for parameters inherited from parent routers. A param callback is called only once per request-response cycle, even if the parameter appears in multiple matching routes.

**Beginner-Friendly Explanation**
`router.param()` lets you run code automatically whenever a specific parameter—like `:userId`—appears in a route. You can use it to load the user from the database, validate the ID, or attach the user object to `req.user`. The code runs before the route handler, so every route using that parameter benefits from the same logic.

---

### Purposes

- To centralize validation and loading logic for a parameter across all routes in a router.
- To automatically attach derived data (e.g., `req.user`) to the request for routes using the parameter.
- To perform parameter transformation or sanitization once per request.
- To avoid duplicating parameter-related code across multiple route handlers.

---

### Syntax Rules and Structure

#### Complete General Syntax: `router.param()`

```javascript
router.param('id', (req, res, next, value, name) => {
  // validation or loading logic
  next();
});
```

**Component Breakdown**

| Parameter | Role |
|-----------|------|
| `'id'` | The parameter name to trigger on. |
| `(req, res, next, value, name)` | Callback signature; `value` is the raw string, `name` is the parameter name. |
| `next()` | Passes control after the callback completes. |

#### Syntax Rules

- `router.param()` does not accept an array of names; register each parameter separately.
- Param callbacks run before any route handler that uses the parameter.
- Callbacks are local to the router; mounted sub-routers do not inherit them.
- A callback runs only once per request, even if the parameter matches multiple routes.

#### Constraints and Limitations

- **Express 5 change** — `router.param(fn)` (the deprecated custom-behavior signature) is no longer supported.
- `router.param()` callbacks are not triggered for parameters inherited from parent routers (use `mergeParams` for access, but the callback itself must be defined on the child router).

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Loading a User with `router.param()`

```javascript
const express = require('express');
const app = express();
const userRouter = express.Router();

const users = { 1: { id: 1, name: 'Alice' }, 2: { id: 2, name: 'Bob' } };

// Param callback: load user and attach to req
userRouter.param('userId', (req, res, next, id) => {
  const user = users[id];
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  req.user = user;
  next();
});

userRouter.get('/:userId', (req, res) => {
  res.json(req.user);
});

app.use('/users', userRouter);

app.listen(3000);
```

**Expected Output**

```
GET /users/1  → 200 { "id": 1, "name": "Alice" }
GET /users/99 → 404 { "error": "User not found" }
```

**Why This Output Occurs**

- `router.param()` intercepts the `:userId` parameter, loads the user, and either attaches it to `req.user` or returns `404`.

---

### Real-World Cases

- **User profiles** — Loading the user record before every route that uses `:userId`.
- **Tenant resolution** — Loading tenant configuration from `:tenantId`.
- **Slug validation** — Ensuring a slug exists before serving content.

---

## Core Concept 6: Router Nesting

### Definitions

**Core Definition**
Router nesting is the practice of mounting one router inside another, creating a hierarchy of routers that mirrors the hierarchical structure of the application’s resources.

**Technical Definition**
A router can be mounted on another router using `parentRouter.use(path, childRouter)`. When nested, the child router’s routes are prefixed with both the parent’s mount path and the child’s mount path. By default, the child router does **not** receive `req.params` values from the parent’s route path. To access parent parameters, the child router must be created with `{ mergeParams: true }`. When parameter names conflict, the child’s value takes precedence.

**Beginner-Friendly Explanation**
Nesting routers is like putting a smaller folder inside a bigger folder. The user router might contain an items router. When you mount the items router inside the user router, URLs become `/users/:userId/items/:itemId`. To let the items router see the `:userId` from the parent, you must enable `mergeParams`.

---

### Purposes

- To model hierarchical resource relationships (e.g., users → items → comments).
- To isolate sub-resource routes into their own modules.
- To share parent route parameters with child routers when needed.
- To build deeply modular applications where each router has a single responsibility.

---

### Syntax Rules and Structure

#### Complete General Syntax: Nested Routers with `mergeParams`

```javascript
const userRouter = express.Router();
const itemRouter = express.Router({ mergeParams: true });

// Mount the child router inside the parent
userRouter.use('/:userId/items', itemRouter);

// Child route can access req.params.userId
itemRouter.get('/', (req, res) => {
  res.send(`Items for user ${req.params.userId}`);
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `express.Router({ mergeParams: true })` | Creates a child router that inherits parent parameters. |
| `userRouter.use('/:userId/items', itemRouter)` | Mounts the child at a parameterized path. |
| `req.params.userId` | Available in child routes due to `mergeParams`. |

#### Syntax Rules

- `mergeParams: true` must be set on the **child** router to access parent parameters.
- If parent and child have parameters with the same name, the child’s value wins.
- `router.param()` callbacks are not inherited; each router must define its own.

#### Constraints and Limitations

- Without `mergeParams`, child routes see only their own parameters.
- Deeply nested parameterized mount paths can become complex and hard to debug.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Nested Routers with `mergeParams`

```javascript
const express = require('express');
const app = express();

const userRouter = express.Router();
const itemRouter = express.Router({ mergeParams: true });

// Parent route
userRouter.get('/:userId', (req, res) => {
  res.send(`User ${req.params.userId}`);
});

// Child routes
itemRouter.get('/', (req, res) => {
  res.send(`Items for user ${req.params.userId}`);
});

itemRouter.get('/:itemId', (req, res) => {
  res.send(`Item ${req.params.itemId} of user ${req.params.userId}`);
});

// Mount child inside parent
userRouter.use('/:userId/items', itemRouter);

// Mount parent on app
app.use('/users', userRouter);

app.listen(3000);
```

**Expected Output**

```
GET /users/5          → 200 "User 5"
GET /users/5/items    → 200 "Items for user 5"
GET /users/5/items/6  → 200 "Item 6 of user 5"
```

**Why This Output Occurs**

- `mergeParams: true` allows `itemRouter` to read `:userId` from the parent mount path.
- The child’s own `:itemId` parameter is also available.

---

### Real-World Cases

- **Nested resources** — `/users/:userId/posts/:postId/comments`.
- **Multi-tenant hierarchies** — `/tenants/:tenantId/projects/:projectId`.
- **Admin sections** — `/admin/users/:userId/orders`.

---

## Core Concept 7: Router Organization

### Definitions

**Core Definition**
Router organization is the practice of structuring router files, folders, and mounting logic to keep a large Express application maintainable, scalable, and easy to navigate.

**Technical Definition**
Router organization typically involves a **feature-based or domain-based folder structure** where each feature (users, products, auth) has its own router file, controller, service, and validation schema. A central `app.js` or `index.js` mounts each router at its appropriate path. This separation of concerns ensures that routing, request handling, business logic, and data access are decoupled, improving testability and collaboration. The application factory pattern—separating `app.js` (configuration) from `server.js` (listening)—is a common organizational best practice.

**Beginner-Friendly Explanation**
Router organization is about where you put your files. Instead of one giant `routes.js` file with hundreds of routes, you create a folder for each feature. Each folder has a router file that exports a router. Then your main app file imports all those routers and mounts them. This makes it easy to find code, add features, and let multiple developers work without conflicts.

---

### Purposes

- To keep route definitions small, focused, and easy to locate.
- To separate routing concerns from business logic and data access.
- To enable parallel development by multiple team members.
- To support scalable project structures that grow with the application.

---

### Syntax Rules and Structure

#### Complete General Syntax: Feature-Based Router Structure

```text
src/
├── app.js                  # App factory; mounts routers
├── server.js               # Entry point; calls app.listen()
├── features/
│   ├── users/
│   │   ├── users.router.js
│   │   ├── users.controller.js
│   │   ├── users.service.js
│   │   └── users.schema.js
│   └── products/
│       ├── products.router.js
│       └── products.controller.js
├── middleware/
│   └── error-handler.js
└── routes/
    └── index.js            # Optional central router aggregator
```

**Component Breakdown**

| Layer | Responsibility |
|-------|----------------|
| `features/users/users.router.js` | Defines user routes and exports the router. |
| `features/users/users.controller.js` | Request handling, validation, response serialization. |
| `features/users/users.service.js` | Business logic and data access. |
| `app.js` | Imports routers and mounts them at their paths. |

#### Syntax Rules

- Each feature should have its own router file that exports an `express.Router()` instance.
- The main application should import routers and mount them with `app.use()`.
- Business logic should not live in router files; delegate to controllers or services.

#### Constraints and Limitations

- Over-fragmentation can make navigation difficult; balance granularity with cohesion.
- Circular dependencies can occur if routers import each other; use a central aggregator if needed.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Feature-Based Router Organization

```javascript
// features/users/users.router.js
const express = require('express');
const router = express.Router();
const controller = require('./users.controller');

router.get('/', controller.list);
router.get('/:id', controller.getById);

module.exports = router;
```

```js
// app.js
const express = require('express');
const app = express();
const usersRouter = require('./features/users/users.router');
const productsRouter = require('./features/products/products.router');

app.use('/api/users', usersRouter);
app.use('/api/products', productsRouter);

app.listen(3000);
```

**Expected Output**

```
GET /api/users       → 200 (user list)
GET /api/users/42    → 200 (user 42)
GET /api/products    → 200 (product list)
```

**Why This Output Occurs**

- Each feature exports a self-contained router; the main app simply mounts them at their respective paths.

---

### Real-World Cases

- **Large monolith APIs** — Feature folders with routers, controllers, and services.
- **Microservice-ready monoliths** — Routers can later be extracted into separate services.
- **Team collaboration** — Multiple developers work on different feature folders without conflicts.

---

## Core Concept 8: Router Best Practices

### Definitions

**Core Definition**
Router best practices are the design, organization, and security patterns that keep router-based Express applications maintainable, predictable, and scalable.

**Technical Definition**
Best practices for Express routers include: using `express.Router()` for every logical group of routes; keeping route handlers thin by delegating business logic to controllers or services; mounting routers with clear, versioned prefixes; applying router-level middleware selectively; using `mergeParams` only when parent parameters are genuinely needed; registering error-handling middleware at the appropriate level; and maintaining consistent naming conventions across router files and mount paths.

**Beginner-Friendly Explanation**
Best practices are the habits that keep your routing code clean. They include putting routes in separate files, not stuffing business logic into route handlers, using clear URL prefixes, and making sure error handling is in the right place.

---

### Purposes

- To maintain a clean separation of concerns between routing, request handling, and business logic.
- To prevent route shadowing and unexpected middleware behavior through deliberate mounting order.
- To ensure error handling is consistent across all routes.
- To make the application easy to test, extend, and refactor.

---

### Syntax Rules and Structure

#### Complete General Syntax: Best-Practice Router Setup

```javascript
// features/users/users.router.js
const express = require('express');
const router = express.Router();
const asyncHandler = require('../../middleware/async-handler');
const controller = require('./users.controller');

// Router-level middleware
router.use(require('../../middleware/request-logger'));

// Routes delegate to controllers
router.get('/', asyncHandler(controller.list));
router.post('/', validateBody, asyncHandler(controller.create));

module.exports = router;

// app.js
app.use('/api/v1/users', usersRouter);
app.use(errorHandler); // error handler last
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Modular router | Each feature exports its own router. |
| Thin handlers | Routes delegate to controller functions. |
| Async safety | `asyncHandler` wraps async controllers. |
| Error handler last | `app.use(errorHandler)` after all routes. |

#### Syntax Rules

- Mount routers at specific prefixes; avoid mounting multiple routers at `/`.
- Keep router files free of business logic.
- Define error-handling middleware after all routes and router mounts.
- Use `mergeParams` only on child routers that genuinely need parent parameters.

#### Constraints and Limitations

- Overusing router-level middleware can add unnecessary layers; apply middleware only where needed.
- `mergeParams` conflicts can silently override parent parameters; use unique names when possible.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Modular Router with Controller Delegation and Error Handling

```javascript
// middleware/async-handler.js
const asyncHandler = (fn) => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

// features/users/users.controller.js
exports.list = async (req, res) => {
  res.json({ users: [] });
};

// features/users/users.router.js
const express = require('express');
const router = express.Router();
const controller = require('./users.controller');
const asyncHandler = require('../../middleware/async-handler');

router.get('/', asyncHandler(controller.list));

module.exports = router;

// app.js
const express = require('express');
const app = express();
const usersRouter = require('./features/users/users.router');

app.use('/api/users', usersRouter);

app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message });
});

app.listen(3000);
```

**Expected Output**

```
GET /api/users  → 200 { "users": [] }
```

**Why This Output Occurs**

- The router delegates to the controller, the async wrapper forwards errors, and the error handler is registered last.

---

### Real-World Cases

- **Production APIs** — Thin routers, controller/service separation, and centralized error handling.
- **Versioned APIs** — `/api/v1` and `/api/v2` routers with shared middleware.
- **Testing** — Routers exported as modules can be tested in isolation with Supertest.

---

## References

- Router · Express.js 5.x — https://expressjs.com/es/5x/api/router/
- `express.Router()` — Express.js 5.x API — https://expressjs.com/es/5x/api/express/#expressrouter
- `router.param(name, callback)` — Express.js 4.x — https://raw.githubusercontent.com/expressjs/expressjs.com/34836ff41fcc44038d0f79c9f1908cf90f753880/_includes/api/en/4x/router-param.md
- Using middleware · Express.js — https://expressjs.com/en/guide/using-middleware/
- How to use express.Router in Node.js · CoreUI — https://coreui.io/answers/how-to-use-express-router-in-nodejs/
- Express.js Router Patterns for Large Applications — https://grizzlypeaksoftware.com/expressjs-router-patterns-for-large-applications/
- Express.js Development Guidelines (GitHub) — https://github.com/snowdreamtech/template/blob/main/.agent/rules/express.md
- Nested Routers with `mergeParams` — Stack Overflow — https://stackoverflow.com/revisions/d8e304d4-6c96-4250-8e64-181e34ab166a/view-source
- Express.js Guide: Mastering Routers and GET Query Parameters — https://requestly.com/blog/express-js-guide-mastering-routers-and-get-query-parameters/