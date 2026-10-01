# Express.js Routers — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The `express.Router` class is a built-in Express utility that creates isolated, mountable "mini-application" instances, each with its own middleware and routing system, enabling modular organisation of route definitions.

**Technical Definition:** A `Router` object is an instance of middleware and routes. It behaves like middleware itself, so it can be used as an argument to `app.use()` or as the argument to another router's `use()` method. The top-level `express` object has a `Router()` method that creates a new router object. Once created, middleware and HTTP method routes (such as `get`, `put`, `post`) can be added to it just like an application. A router is often referred to as a "mini-app" because it is a complete middleware and routing system that is isolated from the main application.

**Beginner-Friendly Explanation:** Think of `express.Router()` as creating a smaller, separate Express application just for one feature of your site. Instead of putting every single route in your main `app.js` file (which becomes a mess), you create a separate file for users, another for products, and another for orders. Each file is its own "mini-app" that handles only its own feature. Then, in your main file, you simply plug each mini-app into the main application at the right URL prefix.

### Key Characteristics

- **Isolated instances:** Each router maintains its own middleware stack and route definitions, independent of the main application and other routers.
- **Mountable:** A router behaves like middleware itself, so it can be mounted onto an `app` or another router using `use()`.
- **Modular:** Routers enable separating routes into different files, improving maintainability and testability.
- **Composable:** Routers can be nested arbitrarily deep to model hierarchical domain relationships.
- **Optionally configurable:** The `Router()` constructor accepts options such as `caseSensitive`, `mergeParams`, and `strict` to control matching behaviour.
- **Shared `req`/`res`:** When a router is mounted, the `req` and `res` objects flow through the router's middleware stack just as they do in the main application.
- **`mergeParams` option:** By default, parameter values from the parent router are not accessible in the child router; setting `mergeParams: true` preserves them, with the child's values taking precedence if there are conflicts.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript knowledge:** Functions, modules (`require`/`module.exports`), and callbacks.
- **Understanding of Express routing:** HTTP methods, route paths, and handler functions.
- **Understanding of middleware:** The `(req, res, next)` pattern and the request–response cycle.
- **Familiarity with project file structure:** How to organise files into directories.

### Related Programming Areas

- **Middleware architecture:** Routers are a specialised form of middleware with their own stack.
- **Application organisation:** Routers are the primary tool for modularising Express applications.
- **REST API design:** Resource-based routers map naturally to RESTful resource endpoints.
- **Testing:** Isolated routers can be unit-tested independently of the main application.
- **Versioned APIs:** Nested routers are commonly used to mount versioned API namespaces (`/api/v1`, `/api/v2`).
- **Authentication and authorisation:** Router-level middleware is used to scope security guards to specific feature areas.

### Core Concepts

1. **`express.Router()`** — instantiating isolated mini-applications for modular, clean, and testable codebases.
2. **Modular Routes** — decoupling route definitions out of the core app entry point into feature-driven files.
3. **Route Prefixes** — mounting an entire feature router onto a base namespace path inside `app.use()`.
4. **Nested Routers** — mounting sub-routers inside parent routers to scale deep hierarchical domain models.
5. **Router-Level Middleware** — scoping security guards or performance layers exclusively to specific router files.
6. **Separating Routes by Resource** — designing clean file systems that isolate unique domain concerns from each other.

---

## Core Concept 1: `express.Router()` — Instantiating Isolated Mini-Applications

### Definitions

**Core Definition:** `express.Router()` is a factory function that creates a new `Router` instance — an isolated middleware and routing system that can be mounted onto an Express application or another router.

**Technical Definition:** The top-level `express` object exposes a `Router()` method that creates a new router object. A `Router` instance is a complete middleware and routing system; for this reason, it is often referred to as a "mini-app". Once created, you can add middleware and HTTP method routes to it just like an application. The constructor accepts an optional `options` object with the properties `caseSensitive`, `mergeParams`, and `strict`.

**Beginner-Friendly Explanation:** `express.Router()` is like creating a smaller, separate Express application just for one part of your website. Instead of cramming every route into one giant file, you make a dedicated router for users, another for products, and another for orders. Each router is self-contained: it has its own middleware, its own routes, and its own logic. You then plug each router into your main app at the appropriate URL prefix.

### Purposes

- To instantiate isolated mini-applications for modular, clean, and testable codebases.
- To create reusable route handlers that can be mounted in multiple applications.
- To isolate middleware and route definitions so they do not interfere with other parts of the application.
- To enable unit testing of routing logic independently of the main application.
- To reduce the cognitive load of maintaining a single monolithic route file.

### Syntax Rules and Structure

#### General Syntax

```js
const express = require('express');
const router = express.Router([options]);

router.METHOD(path, handler);
```

| Component | Breakdown |
|-----------|-----------|
| `express.Router()` | Factory function that creates a new router instance. |
| `[options]` | Optional object: `{ caseSensitive, mergeParams, strict }`. |
| `router.METHOD()` | Registers a route on the router (`get`, `post`, `put`, `delete`, `all`, etc.). |
| `router.use()` | Registers middleware on the router. |

#### Options Breakdown

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `caseSensitive` | Boolean | `false` | When `true`, `/Foo` and `/foo` are treated as different routes. |
| `mergeParams` | Boolean | `false` | When `true`, preserves `req.params` values from the parent router; child values take precedence on conflict. |
| `strict` | Boolean | `false` | When `true`, `/foo` and `/foo/` are treated as different routes. |

#### Syntax Rules

- `express.Router()` must be called to create a new router instance; the `express` object itself is not a router.
- A router instance exposes the same HTTP method functions as `app` (`get`, `post`, `put`, `delete`, `patch`, `all`, etc.).
- A router can be mounted on an `app` using `app.use(path, router)` or on another router using `parentRouter.use(path, childRouter)`.
- The `Router()` constructor must be called with `new` in some documentation examples, but the factory function form (`express.Router()`) is the standard usage.
- Routers can be exported from modules and imported into the main application file.

#### Constraints and Limitations

- A router does not have its own server; it must be mounted onto an application or another router to be active.
- `router.listen()` does not exist; only the main `app` can listen on a port.
- Router-level settings (`caseSensitive`, `strict`) apply only to the router instance, not to the parent application.
- In Express 5, the `app.param(fn)` signature is deprecated; use `router.param()` for router-scoped parameter middleware.

### Annotated Code Example

```js
// router-basic.js
const express = require('express');
const app = express();

// Step 1: Create a router instance
const userRouter = express.Router();

// Step 2: Define routes on the router (not on the app)
userRouter.get('/', (req, res) => {
  res.send('List of users');
});

userRouter.get('/:id', (req, res) => {
  res.send(`User profile: ${req.params.id}`);
});

userRouter.post('/', (req, res) => {
  res.status(201).send('User created');
});

// Step 3: Mount the router on the app at a base path
app.use('/users', userRouter);

// The router is now active at:
//   GET  /users       -> "List of users"
//   GET  /users/42    -> "User profile: 42"
//   POST /users       -> "User created"

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output (for `GET /users`):**
```
List of users
```

**Expected Output (for `GET /users/42`):**
```
User profile: 42
```

**Expected Output (for `POST /users`):**
```
User created
```

**Why this output:** The `userRouter` is created as an isolated instance. Routes are defined on the router using the same HTTP method functions available on `app`. When `app.use('/users', userRouter)` is called, Express mounts the router at the `/users` prefix. Every incoming request whose path begins with `/users` is forwarded to the router, which then matches the remaining path segment against its own route definitions. The router's `'/'` route becomes `/users`, and `'/:id'` becomes `/users/:id`.

### Real-World Cases

- **Feature modules:** A dedicated router for user management, product management, or order processing.
- **Microservices:** Each service exposes its own router, which can be mounted on a gateway application.
- **Testability:** A router can be imported into a test file and exercised without starting a full HTTP server.
- **Third-party plugins:** Libraries export routers that can be mounted into any Express application.

---

## Core Concept 2: Modular Routes — Decoupling Route Definitions into Feature-Driven Files

### Definitions

**Core Definition:** Modular routes involve moving route definitions out of the main application entry point and into separate, feature-specific files, each exporting its own router instance.

**Technical Definition:** In a modular Express application, each feature or resource has its own route file (e.g., `routes/users.js`, `routes/products.js`). Each file creates an `express.Router()` instance, defines the routes for that resource, and exports the router via `module.exports`. The main application file imports each router and mounts it with `app.use()`. This separation of concerns improves maintainability, testability, and scalability.

**Beginner-Friendly Explanation:** Instead of writing every single route in one massive `app.js` file, you split them into smaller files based on what they do. All user-related routes go in `users.js`, all product routes in `products.js`, and so on. Each file is self-contained and exports its routes. Your main file then simply says "use the user routes for `/users` and the product routes for `/products`." This keeps your code organised and easy to find.

### Purposes

- To decouple route definitions out of the core app entry point into feature-driven files.
- To improve code organisation by grouping related routes together.
- To make routes easier to find, understand, and modify.
- To enable independent testing of each feature's routes.
- To reduce merge conflicts in team environments by isolating changes to specific files.
- To facilitate code reuse across different applications or API versions.

### Syntax Rules and Structure

#### General Syntax

**Feature route file (`routes/users.js`):**
```js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => { /* ... */ });
router.post('/', (req, res) => { /* ... */ });

module.exports = router;
```

**Main application file (`app.js`):**
```js
const express = require('express');
const app = express();

const userRouter = require('./routes/users');
const productRouter = require('./routes/products');

app.use('/users', userRouter);
app.use('/products', productRouter);
```

| Component | Breakdown |
|-----------|-----------|
| `routes/users.js` | Feature-specific route file. |
| `express.Router()` | Creates the router instance. |
| `module.exports = router` | Exports the router for importing elsewhere. |
| `require('./routes/users')` | Imports the router in the main file. |
| `app.use('/users', userRouter)` | Mounts the router at a base path. |

#### Syntax Rules

- Each route file must create its own `express.Router()` instance.
- The router must be exported (`module.exports = router`) to be importable.
- The main application file imports routers and mounts them using `app.use()`.
- Route paths inside a router are relative to the mount path, not absolute.
- A router can define routes for any HTTP method and path, just like `app`.

#### Constraints and Limitations

- Circular dependencies can occur if two route files require each other; use dependency injection or shared services to avoid this.
- Route order matters: more specific routes must be defined before more general ones within the same router.
- When a router is moved to a separate file, any middleware needed by its routes must also be moved or imported.
- Error-handling middleware defined inside a router will only catch errors from that router's routes.

### Annotated Code Example

#### Step 1: Create the route file

```js
// routes/users.js
const express = require('express');
const router = express.Router();

// Simulated database
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' }
];

// GET /users — list all users
router.get('/', (req, res) => {
  res.json(users);
});

// GET /users/:id — get one user
router.get('/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ error: 'User not found' });
  res.json(user);
});

// POST /users — create a user
router.post('/', (req, res) => {
  const newUser = { id: users.length + 1, name: req.body.name };
  users.push(newUser);
  res.status(201).json(newUser);
});

module.exports = router;
```

#### Step 2: Create the main application file

```js
// app.js
const express = require('express');
const app = express();

// Import the modular router
const userRouter = require('./routes/users');

app.use(express.json());                  // Parse JSON request bodies
app.use('/users', userRouter);            // Mount at /users

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output (for `GET /users`):**
```
[{"id":1,"name":"Alice"},{"id":2,"name":"Bob"}]
```

**Expected Output (for `GET /users/1`):**
```
{"id":1,"name":"Alice"}
```

**Expected Output (for `POST /users` with `{ "name": "Charlie" }`):**
```
{"id":3,"name":"Charlie"}
```

**Why this output:** The `users.js` file exports a router with three routes defined relative to `/`. The main `app.js` imports this router and mounts it at `/users`. The router's `'/'` route becomes `GET /users`, its `'/:id'` route becomes `GET /users/:id`, and its `POST '/'` route becomes `POST /users`. The router is completely isolated from the main app; it only knows about its own routes.

#### Real-World File Structure

```
project/
├── app.js                  # Main application entry point
├── routes/
│   ├── users.js            # User routes
│   ├── products.js         # Product routes
│   └── orders.js           # Order routes
├── controllers/
│   ├── userController.js   # Business logic for users
│   ├── productController.js
│   └── orderController.js
├── middleware/
│   ├── auth.js             # Authentication middleware
│   └── logger.js           # Logging middleware
└── models/
    ├── User.js             # User data model
    ├── Product.js
    └── Order.js
```

### Real-World Cases

- **Team collaboration:** Different developers can work on different route files without conflicting.
- **API versioning:** `routes/v1/users.js` and `routes/v2/users.js` for different API versions.
- **Feature flags:** Enabling or disabling entire feature routers based on configuration.
- **Testing:** Importing `routes/users.js` into a test file to test user routes in isolation.

---

## Core Concept 3: Route Prefixes — Mounting a Feature Router onto a Base Namespace Path

### Definitions

**Core Definition:** A route prefix is a base path segment (e.g., `/api`, `/admin`, `/v1`) that is prepended to every route defined inside a mounted router, allowing an entire feature to be namespaced under a common URL segment.

**Technical Definition:** When a router is mounted with `app.use('/prefix', router)`, the prefix path is prepended to every route defined in the router. For example, a router with a route `/:id` mounted at `/books` becomes `/books/:id`. The path passed to `app.use()` is prepended to the router's paths.

**Beginner-Friendly Explanation:** A route prefix is like a street name for a neighbourhood. If your user routes all live at `/users`, you don't have to write `/users` in front of every single route. Instead, you tell Express "everything in this router starts with `/users`" by mounting it at that prefix. The router's internal routes are then written as if they were at the root.

### Purposes

- To mount an entire feature router onto a base namespace path inside `app.use()`.
- To avoid repeating the same path segment in every route definition.
- To create logical groupings of routes under a common URL namespace (e.g., `/api`, `/admin`, `/v1`).
- To enable easy versioning by changing the mount path rather than every individual route.
- To simplify route definitions inside the router (e.g., `'/'` instead of `/users`).

### Syntax Rules and Structure

#### General Syntax

```js
app.use('/prefix', router);
```

| Component | Breakdown |
|-----------|-----------|
| `app` | The main Express application (or a parent router). |
| `'/prefix'` | The base path prepended to all router routes. |
| `router` | The router instance to mount. |

#### Prefix Resolution

| Router route | Mount prefix | Effective full path |
|-------------|-------------|---------------------|
| `'/'` | `/users` | `/users` |
| `'/:id'` | `/users` | `/users/:id` |
| `'/active'` | `/users` | `/users/active` |
| `'/'` | `/api/users` | `/api/users` |
| `'/'` | `/v1/products` | `/v1/products` |

#### Syntax Rules

- The prefix must be a valid Express path string (e.g., `/api`, `/admin`, `/v1`).
- The prefix is **prepended**, not appended, to the router's paths.
- Multiple routers can be mounted at different prefixes on the same application.
- A router can be mounted at multiple prefixes on the same application.
- The prefix can include parameter placeholders (e.g., `/api/:version`).

#### Constraints and Limitations

- The prefix path is matched with Express's default case-insensitivity and trailing-slash tolerance unless the router is configured with `caseSensitive` or `strict`.
- Mounting two routers at the same prefix can cause the first-mounted router to intercept requests intended for the second.
- The prefix itself is not accessible from inside the router as a route; it is consumed by the mounting mechanism.

### Annotated Code Example

```js
// route-prefix.js
const express = require('express');
const app = express();

// User router — routes defined relative to the router root
const userRouter = express.Router();
userRouter.get('/', (req, res) => res.send('User list'));
userRouter.get('/:id', (req, res) => res.send(`User ${req.params.id}`));

// Product router
const productRouter = express.Router();
productRouter.get('/', (req, res) => res.send('Product list'));
productRouter.get('/:id', (req, res) => res.send(`Product ${req.params.id}`));

// Mount with prefixes
app.use('/users', userRouter);              // All user routes under /users
app.use('/api/products', productRouter);    // All product routes under /api/products

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output (for `GET /users`):**
```
User list
```

**Expected Output (for `GET /users/42`):**
```
User 42
```

**Expected Output (for `GET /api/products`):**
```
Product list
```

**Expected Output (for `GET /api/products/7`):**
```
Product 7
```

**Why this output:** The `userRouter` is mounted at `/users`, so its `'/'` route matches `GET /users` and its `'/:id'` route matches `GET /users/42`. The `productRouter` is mounted at `/api/products`, so its routes are prefixed accordingly. The routers themselves do not know about the prefix; they only define routes relative to their own root.

#### Real-World Cases

- **API versioning:** `app.use('/api/v1', v1Router)` and `app.use('/api/v2', v2Router)`.
- **Admin panel:** `app.use('/admin', adminRouter)` to isolate all admin routes.
- **Static resources:** `app.use('/static', express.static('public'))` (a similar pattern for static files).
- **Multi-tenant applications:** `app.use('/:tenantId', tenantRouter)` to prefix routes with a tenant identifier.

---

## Core Concept 4: Nested Routers — Mounting Sub-Routers Inside Parent Routers

### Definitions

**Core Definition:** Nested routers are routers that are mounted inside other routers, creating a hierarchical routing structure that models deep domain relationships.

**Technical Definition:** Because a router behaves like middleware itself, it can be used as an argument to another router's `use()` method. This allows mounting a sub-router onto a parent router at a specific path prefix. The parent router's mount path is prepended to the child router's paths, and the child router's paths are prepended to the grandchild router's paths, and so on. The `mergeParams` option (introduced in Express 4.5.0) allows parameter values from the parent router to be accessible in the child router.

**Beginner-Friendly Explanation:** Nested routers are like folders inside folders on your computer. You have a "users" folder, and inside it, a "posts" folder, and inside that, a "comments" folder. Each folder is independent, but they're nested inside each other. In Express, you can mount a router inside another router, so `/users/42/posts/7/comments` is handled by a chain of three routers, each responsible for its own segment of the URL.

### Purposes

- To mount sub-routers inside parent routers to scale deep hierarchical domain models.
- To model relationships such as users → posts → comments or categories → products → reviews.
- To avoid repeating parent path segments in every child route.
- To isolate logic for each level of a hierarchy into its own file and router.
- To enable independent development and testing of each hierarchy level.

### Syntax Rules and Structure

#### General Syntax

```js
// Parent router
const parentRouter = express.Router();
const childRouter = express.Router();

childRouter.get('/', (req, res) => { /* ... */ });
parentRouter.use('/:parentId/children', childRouter);
app.use('/parents', parentRouter);
```

| Component | Breakdown |
|-----------|-----------|
| `parentRouter` | The router that will hold the child router. |
| `childRouter` | The sub-router to be nested. |
| `parentRouter.use('/:parentId/children', childRouter)` | Mounts the child router under a path on the parent. |
| `app.use('/parents', parentRouter)` | Mounts the parent router on the app. |

#### Full Path Resolution

| Router level | Route | Mount path | Full effective path |
|-------------|-------|-----------|---------------------|
| App | `/parents` | — | `/parents` |
| Parent router | `/:parentId/children` | `/parents` | `/parents/:parentId/children` |
| Child router | `/:childId` | (inherited) | `/parents/:parentId/children/:childId` |

#### The `mergeParams` Option

By default, a child router does **not** have access to parameters defined in the parent router's path. To preserve them, create the child router with `{ mergeParams: true }`:

```js
const childRouter = express.Router({ mergeParams: true });
```

If the parent and child have conflicting parameter names, the child's value takes precedence.

#### Syntax Rules

- A router can be mounted on another router using `parentRouter.use(path, childRouter)`.
- The parent's mount path is prepended to the child's paths.
- Nested routers can be arbitrarily deep (grandchild, great-grandchild, etc.).
- Parameter values from the parent are **not** automatically available in the child unless `mergeParams: true` is set.
- The child router must be created **before** it is mounted on the parent.

#### Constraints and Limitations

- Without `mergeParams: true`, `req.params` inside the child router will only contain the child's own parameters, not the parent's.
- Route order matters: mount the child router after any specific routes on the parent.
- Deeply nested routers can make debugging and tracing request flow more difficult; use logging middleware to trace the chain.
- Middleware defined on the parent router runs before the child router's middleware.

### Annotated Code Example

```js
// nested-routers.js
const express = require('express');
const app = express();

// --- Child router (comments) ---
const commentRouter = express.Router({ mergeParams: true });

// GET /posts/:postId/comments
commentRouter.get('/', (req, res) => {
  res.json({
    postId: req.params.postId,        // Available because mergeParams: true
    comments: ['Great post!', 'Thanks!']
  });
});

// GET /posts/:postId/comments/:commentId
commentRouter.get('/:commentId', (req, res) => {
  res.json({
    postId: req.params.postId,
    commentId: req.params.commentId
  });
});

// --- Parent router (posts) ---
const postRouter = express.Router({ mergeParams: true });

// GET /posts
postRouter.get('/', (req, res) => {
  res.json({ posts: ['Post 1', 'Post 2'] });
});

// Mount comment router as a sub-router
postRouter.use('/:postId/comments', commentRouter);

// --- Mount parent router on app ---
app.use('/posts', postRouter);

app.listen(3000, () => console.log('Nested routers on port 3000'));
```

**Expected Output (for `GET /posts`):**
```
{"posts":["Post 1","Post 2"]}
```

**Expected Output (for `GET /posts/7/comments`):**
```
{"postId":"7","comments":["Great post!","Thanks!"]}
```

**Expected Output (for `GET /posts/7/comments/3`):**
```
{"postId":"7","commentId":"3"}
```

**Why this output:** The `postRouter` is mounted at `/posts`. Its sub-router (`commentRouter`) is mounted at `/:postId/comments`. When a request arrives at `/posts/7/comments/3`, the path is split: `/posts` matches the app-level mount, `/:postId/comments` matches the parent router's mount, and `/:commentId` matches the child router's route. Because both routers use `mergeParams: true`, the child router has access to `req.params.postId` from the parent router.

### Real-World Cases

- **Blog platforms:** `/posts/:postId/comments/:commentId` for post → comment hierarchies.
- **E-commerce:** `/categories/:categoryId/products/:productId/reviews/:reviewId` for deep product hierarchies.
- **Project management:** `/projects/:projectId/tasks/:taskId/subtasks/:subtaskId` for nested task hierarchies.
- **API versioning:** `/api/:version/:resource` where each version is a router and each resource is a sub-router.

---

## Core Concept 5: Router-Level Middleware — Scoping Middleware to Specific Router Files

### Definitions

**Core Definition:** Router-level middleware is middleware that is bound to an instance of `express.Router()` rather than to the main application, allowing middleware to be scoped exclusively to a specific router's routes.

**Technical Definition:** Router-level middleware works in the same way as application-level middleware, except it is bound to an instance of `express.Router()`. It is loaded using the `router.use()` and `router.METHOD()` functions. Because it is scoped to the router, it only executes for requests that pass through that router, not for requests handled by other routers or the main application. To skip the rest of the router's middleware functions, call `next('router')` to pass control back out of the router instance.

**Beginner-Friendly Explanation:** Router-level middleware is like a security checkpoint that only exists in one wing of a building. If you're entering the "admin" wing, you go through the admin security check. But if you're just visiting the "public" wing, you never encounter that checkpoint. In Express, you can attach middleware to a router so it only runs for routes defined in that router, not for the rest of your application.

### Purposes

- To scope security guards or performance layers exclusively to specific router files.
- To apply authentication or authorisation only to the routes that need it.
- To add logging, rate limiting, or data transformation to a specific feature area.
- To validate request data before it reaches the router's route handlers.
- To avoid running unnecessary middleware on unrelated routes.

### Syntax Rules and Structure

#### General Syntax

```js
const router = express.Router();

// Middleware runs for all routes in this router
router.use((req, res, next) => {
  // pre-processing
  next();
});

// Middleware runs only for a specific path within the router
router.use('/specific', (req, res, next) => {
  // ...
  next();
});

// Middleware runs only for a specific HTTP method and path
router.get('/path', middleware1, middleware2, handler);
```

| Component | Breakdown |
|-----------|-----------|
| `router.use(fn)` | Middleware runs for **all** routes in the router. |
| `router.use('/path', fn)` | Middleware runs only for routes matching `/path` within the router. |
| `router.METHOD(path, middleware, handler)` | Middleware runs only for the specified method and path. |
| `next()` | Passes control to the next middleware or handler. |
| `next('router')` | Skips the rest of the router's middleware and returns to the parent. |

#### Syntax Rules

- Router-level middleware must be bound to a `Router` instance, not to `app`.
- Middleware is executed in the order it is registered.
- Middleware can be registered with or without a mount path.
- `router.use()` with no path applies to all routes in the router.
- `router.use(path, fn)` applies only to routes that match `path` within the router.
- `next('router')` exits the router and passes control back to the parent application or router.
- `next('route')` skips the remaining handlers for the **current route** (works only in `router.METHOD()` callbacks, not in `router.use()`).

#### Constraints and Limitations

- Router-level middleware does **not** run for requests that do not enter the router (i.e., requests that do not match the router's mount path).
- Middleware defined on the parent application runs **before** router-level middleware.
- If `next()` is not called and the response is not ended, the request will hang.
- Error-handling middleware defined inside a router only catches errors from that router's routes.

### Annotated Code Example

```js
// router-middleware.js
const express = require('express');
const app = express();

// --- Admin router with authentication middleware ---
const adminRouter = express.Router();

// Router-level middleware: runs only for admin routes
adminRouter.use((req, res, next) => {
  const apiKey = req.headers['x-api-key'];
  if (apiKey !== 'secret-admin-key') {
    return res.status(401).json({ error: 'Unauthorized: invalid admin key' });
  }
  next();                                 // Proceed to the route handler
});

adminRouter.get('/dashboard', (req, res) => {
  res.json({ page: 'Admin Dashboard', access: 'granted' });
});

adminRouter.get('/settings', (req, res) => {
  res.json({ page: 'Admin Settings', access: 'granted' });
});

// --- Public router with no authentication ---
const publicRouter = express.Router();

publicRouter.get('/home', (req, res) => {
  res.json({ page: 'Home', access: 'public' });
});

publicRouter.get('/about', (req, res) => {
  res.json({ page: 'About', access: 'public' });
});

// Mount routers
app.use('/admin', adminRouter);            // Admin middleware applies here
app.use('/public', publicRouter);          // No middleware applies here

app.listen(3000, () => console.log('Router middleware on 3000'));
```

**Expected Output (for `GET /admin/dashboard` without API key):**
```
{"error":"Unauthorized: invalid admin key"}
```

**Expected Output (for `GET /admin/dashboard` with `x-api-key: secret-admin-key`):**
```
{"page":"Admin Dashboard","access":"granted"}
```

**Expected Output (for `GET /public/home`):**
```
{"page":"Home","access":"public"}
```

**Why this output:** The `adminRouter.use()` middleware runs for **every** request that enters the `/admin` router. It checks the `x-api-key` header and either returns a 401 or calls `next()`. The `publicRouter` has no such middleware, so its routes are accessible without authentication. The middleware is scoped exclusively to the admin router; it never runs for public routes.

#### Using `next('router')` to Exit a Router

```js
// next-router-example.js
const express = require('express');
const app = express();

const router = express.Router();

// Middleware that skips the rest of the router
router.use((req, res, next) => {
  if (req.query.skip === 'true') {
    return next('router');                 // Exit this router entirely
  }
  next();
});

router.get('/data', (req, res) => {
  res.send('Router data');
});

app.use('/api', router);

// Fallback route outside the router
app.get('/api/data', (req, res) => {
  res.send('Fallback data');
});

app.listen(3000, () => console.log('next(router) example on 3000'));
```

**Expected Output (for `GET /api/data?skip=true`):**
```
Fallback data
```

**Expected Output (for `GET /api/data`):**
```
Router data
```

**Why this output:** When `skip=true` is present, the middleware calls `next('router')`, which exits the router entirely and passes control back to the application. The application then matches the `/api/data` route defined on `app`, which sends `'Fallback data'`. Without the skip parameter, `next()` is called, and the router's own `/data` route handles the request.

### Real-World Cases

- **Authentication guards:** `router.use(requireAuth)` on an admin router to protect all admin routes.
- **Rate limiting:** `router.use(rateLimiter)` on a public API router to limit requests per IP.
- **Logging:** `router.use(requestLogger)` on a router to log only requests to that feature.
- **Data validation:** `router.use(validateRequestBody)` on a router that accepts POST/PUT requests.
- **CORS headers:** `router.use(corsMiddleware)` on a router that serves a different frontend origin.

---

## Core Concept 6: Separating Routes by Resource — Designing Clean File Systems

### Definitions

**Core Definition:** Separating routes by resource is the practice of organising route files in a directory structure where each resource or domain concern has its own dedicated file, router, and optionally controller, keeping the codebase modular and maintainable.

**Technical Definition:** In a resource-based file structure, the `routes/` directory contains one file per resource (e.g., `users.js`, `products.js`, `orders.js`). Each file creates an `express.Router()`, defines the routes for that resource, and exports the router. The main application file imports all routers and mounts them at their respective prefixes. This structure is often combined with a `controllers/` directory (for business logic) and a `models/` directory (for data models), following the Model–View–Controller (MVC) pattern.

**Beginner-Friendly Explanation:** Separating routes by resource is like organising a filing cabinet. Instead of throwing every document into one drawer, you have a drawer for "Users," a drawer for "Products," and a drawer for "Orders." Each drawer contains only documents related to that one thing. When you need to find something about users, you know exactly which drawer to open. In Express, this means one file per resource in your `routes/` folder, with each file handling only that resource's routes.

### Purposes

- To design clean file systems that isolate unique domain concerns from each other.
- To make the codebase easier to navigate by grouping related routes together.
- To reduce the risk of merge conflicts in team environments.
- To enable independent development, testing, and deployment of each resource's routes.
- To scale the application by simply adding new resource files.
- To follow established architectural patterns (MVC, layered architecture).

### Syntax Rules and Structure

#### Recommended File Structure

```
project/
├── app.js                        # Main application entry
├── routes/
│   ├── index.js                  # Optional: aggregates all routers
│   ├── users.js                  # User routes
│   ├── products.js               # Product routes
│   └── orders.js                 # Order routes
├── controllers/
│   ├── userController.js         # User business logic
│   ├── productController.js
│   └── orderController.js
├── models/
│   ├── User.js
│   ├── Product.js
│   └── Order.js
├── middleware/
│   ├── auth.js
│   └── logger.js
└── config/
    └── database.js
```

#### Router File Template

```js
// routes/users.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');

// GET /users
router.get('/', userController.getAll);

// GET /users/:id
router.get('/:id', userController.getById);

// POST /users
router.post('/', userController.create);

// PUT /users/:id
router.put('/:id', userController.update);

// DELETE /users/:id
router.delete('/:id', userController.remove);

module.exports = router;
```

#### Main Application File

```js
// app.js
const express = require('express');
const app = express();

app.use(express.json());

// Mount resource routers
app.use('/users', require('./routes/users'));
app.use('/products', require('./routes/products'));
app.use('/orders', require('./routes/orders'));

app.listen(3000, () => console.log('Server on port 3000'));
```

| Component | Breakdown |
|-----------|-----------|
| `routes/users.js` | One file per resource. |
| `controllers/userController.js` | Business logic separated from routing. |
| `models/User.js` | Data model for the resource. |
| `middleware/` | Shared middleware (auth, logging). |

#### Syntax Rules

- Each resource should have exactly one route file (e.g., `users.js`).
- Each route file should export a single `express.Router()` instance.
- The main application should mount each router at a prefix matching the resource name.
- Controllers should contain the business logic; routes should only wire paths to controllers.
- Shared middleware should be imported into route files or mounted at the application level.
- The `routes/index.js` file (optional) can aggregate all routers into a single module for cleaner imports.

#### Constraints and Limitations

- Over-modularisation can make it difficult to trace a request through many layers; balance file count with clarity.
- Circular dependencies can occur if route files require each other; use a dependency injection or service layer to avoid this.
- Route order across different routers is determined by the order of `app.use()` calls; place more specific routers first if paths overlap.

### Annotated Code Example

#### Step 1: Create the user controller

```js
// controllers/userController.js
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' }
];

exports.getAll = (req, res) => {
  res.json(users);
};

exports.getById = (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ error: 'User not found' });
  res.json(user);
};

exports.create = (req, res) => {
  const newUser = { id: users.length + 1, name: req.body.name };
  users.push(newUser);
  res.status(201).json(newUser);
};

exports.update = (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ error: 'User not found' });
  user.name = req.body.name || user.name;
  res.json(user);
};

exports.remove = (req, res) => {
  const index = users.findIndex(u => u.id === parseInt(req.params.id));
  if (index === -1) return res.status(404).json({ error: 'User not found' });
  users.splice(index, 1);
  res.status(204).send();
};
```

#### Step 2: Create the user route file

```js
// routes/users.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');

router.get('/', userController.getAll);
router.get('/:id', userController.getById);
router.post('/', userController.create);
router.put('/:id', userController.update);
router.delete('/:id', userController.remove);

module.exports = router;
```

#### Step 3: Create the main application file

```js
// app.js
const express = require('express');
const app = express();

app.use(express.json());

// Mount resource routers
app.use('/users', require('./routes/users'));

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output (for `GET /users`):**
```
[{"id":1,"name":"Alice"},{"id":2,"name":"Bob"}]
```

**Expected Output (for `POST /users` with `{ "name": "Charlie" }`):**
```
{"id":3,"name":"Charlie"}
```

**Expected Output (for `DELETE /users/1`):**
```
(204 No Content)
```

**Why this output:** The route file (`routes/users.js`) defines the URL structure and delegates to the controller (`controllers/userController.js`) for business logic. The controller manipulates the simulated database and sends responses. The main application file (`app.js`) simply mounts the router. This separation means the routing configuration is isolated from the business logic, making both easier to understand and test.

### Real-World Cases

- **E-commerce API:** `routes/products.js`, `routes/carts.js`, `routes/orders.js`, `routes/users.js` — each resource in its own file.
- **Social media API:** `routes/posts.js`, `routes/comments.js`, `routes/likes.js`, `routes/follows.js`.
- **Content management system:** `routes/articles.js`, `routes/categories.js`, `routes/tags.js`, `routes/media.js`.
- **Learning management system:** `routes/courses.js`, `routes/lessons.js`, `routes/enrollments.js`, `routes/grades.js`.

---

## References

- Express.js 5.x Router API — https://expressjs.com/en/5x/api/router/
- Express.js 4.x Router API — https://expressjs.com/en/4x/api/router/
- Express.js 5.x API — `express.Router()` — https://expressjs.com/en/5x/api/express/#expressrouter
- Express.js Using Middleware Guide — https://expressjs.com/en/guide/using-middleware/
- Express.js Routing Guide — https://expressjs.com/en/guide/routing.html
- Express.js 5.x Application Object — `app.use()` — https://expressjs.com/en/5x/api/application/#appuse
- Express.js 5.x Router — `router.use()` — https://expressjs.com/en/5x/api/router/#routeruse
- Express.js 5.x Router — `router.METHOD()` — https://expressjs.com/en/5x/api/router/#routermethod
- Express.js 5.x Router — `router.all()` — https://expressjs.com/en/5x/api/router/#routerall
- Express.js 5.x Router — `router.param()` — https://expressjs.com/en/5x/api/router/#routerparam
- Express.js 5.x Router Options — `mergeParams` — https://expressjs.com/en/5x/api/router/#router
- Express.js 5.x Migration Guide — https://expressjs.com/en/guide/migrating-5.html
- Compile-N-Run — Express Route Organization — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/1-express-routing/10-express-route-organization.mdx
- Compile-N-Run — Express Nested Routes — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/1-express-routing/9-express-nested-routes.mdx
- Educative — Express Router and Modular Routing — https://www.educative.io/courses/express-router-and-modular-routing
- Requestly — Express.js Guide: Mastering Routers — https://requestly.com/blog/express-js-guide-mastering-routers-and-get-query-parameters/
- GeeksforGeeks — What is the Use of Router in Express.js? — https://www.geeksforgeeks.org/what-is-the-use-of-router-in-express-js/