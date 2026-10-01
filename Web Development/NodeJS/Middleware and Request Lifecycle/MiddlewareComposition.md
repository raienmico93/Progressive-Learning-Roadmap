# Middleware Composition — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Middleware composition is the practice of assembling multiple middleware functions into an ordered pipeline that processes HTTP requests and responses in a controlled, predictable sequence.

**Technical Definition:** An Express application is essentially a series of middleware function calls executed during the request-response cycle. Middleware functions have access to the request object (`req`), the response object (`res`), and the next middleware function in the application's request-response cycle, commonly named `next`. If a middleware function does not end the request-response cycle, it must call `next()` to pass control to the next middleware function; otherwise, the request will be left hanging.

**Beginner-Friendly Explanation:** Middleware composition is like an assembly line in a factory. Each worker (middleware function) does something to the product (the request) before passing it to the next worker. One worker parses the body, another checks authentication, another logs the request, and the final worker sends the response. The order of the workers matters — if you check authentication before parsing the body, you might process unauthenticated data.

### Key Characteristics

- **Sequential execution:** Middleware functions execute in the exact order they are registered.
- **Shared context:** Every middleware receives the same `req` and `res` objects, enabling data to be attached and read by downstream middleware.
- **Pipeline termination:** A middleware must either call `next()` to continue the chain or end the response to terminate it.
- **Error propagation:** Calling `next(err)` skips all remaining regular middleware and jumps to the first error-handling middleware.
- **Composability:** Middleware can be grouped into arrays, mounted on routers, or applied globally.

### Prerequisites

- **Node.js runtime:** Node.js 18+ recommended.
- **Express fundamentals:** Routing, `app.use()`, and route handlers.
- **HTTP fundamentals:** Request methods, headers, status codes, and bodies.
- **JavaScript async/await:** Promises and asynchronous middleware patterns.

### Related Programming Areas

- **Express.js:** The most common framework for middleware-based architecture.
- **Security:** Helmet, CORS, rate limiting, and CSRF protection.
- **Logging:** Morgan, Pino, and Winston.
- **Validation:** Zod, Joi, and AJV.
- **Context Management:** AsyncLocalStorage and request-scoped state.

### Core Concepts

1. **Middleware Order** — execution flow, short-circuiting, and response interception.
2. **Conditional Middleware** — dynamic execution based on environment, path, or feature flags.
3. **Route-Specific Middleware** — route grouping, sub-router isolation, and inline guards.
4. **Global Middleware** — application-wide bootstrap, third-party integration, and rate limiting.

---

## Core Concept 1: Middleware Order

### Sub-Feature 1.1: Execution Flow (Onion Model vs. Linear Pipeline)

#### Definitions

**Core Definition:** Middleware execution follows a **linear pipeline** model in Express: request flows through middleware in registration order, and the response flows back through the same middleware in reverse order (the "onion model" pattern).

**Technical Definition:** The order in which middleware is defined is critical. They are invoked sequentially, so the order defines middleware precedence. A logger should be the very first middleware so that every request gets logged. The **onion model** describes the full lifecycle: the request passes through each middleware's pre-`next()` logic in order, reaches the handler, and the response passes back through each middleware's post-`next()` logic in reverse order.

**Beginner-Friendly Explanation:** Imagine an onion. The request goes in through the outer layers, reaches the centre (the route handler), and the response comes back out through the same layers. Each layer has two moments: before passing the request inward, and after the response comes back out.

#### Purposes

- To ensure that foundational middleware (logging, security) runs before business logic.
- To guarantee that cleanup and response transformation run in the correct order.
- To enable tracing and timing across the full request-response cycle.

#### Syntax Rules and Structure

```javascript
// Linear pipeline: request flows top-to-bottom
app.use(logger);        // Runs first
app.use(authenticate);  // Runs second
app.use(validate);      // Runs third
app.get('/data', handler); // Runs fourth (handler)

// Onion model: response flows bottom-to-top through post-next logic
app.use((req, res, next) => {
  console.log('Before');  // Pre-next: runs in order
  next();
  console.log('After');   // Post-next: runs in reverse order
});
```

| Phase | Order | Example |
|-------|-------|---------|
| Pre-`next()` | Registration order (top-to-bottom) | Logging, auth, validation |
| Handler | Middle | Route handler |
| Post-`next()` | Reverse order (bottom-to-top) | Response headers, cleanup |

**Constraints and Limitations:**
- Middleware must call `next()` or end the response; otherwise, the request hangs.
- Post-`next()` logic runs after the response is sent, so `res.send()` cannot be called again.

#### Annotated Code Example

```javascript
// onion-model.js
const express = require('express');
const app = express();

app.use((req, res, next) => {
  console.log('1. Pre-next (outer)');
  next();
  console.log('6. Post-next (outer)');
});

app.use((req, res, next) => {
  console.log('2. Pre-next (inner)');
  next();
  console.log('5. Post-next (inner)');
});

app.get('/test', (req, res) => {
  console.log('3. Handler');
  res.json({ message: 'Hello' });
  console.log('4. After res.json()');
});

app.listen(3000);
```

**Expected Output:**
```
1. Pre-next (outer)
2. Pre-next (inner)
3. Handler
4. After res.json()
5. Post-next (inner)
6. Post-next (outer)
```

**Why this output:** The request enters through the outer middleware, passes to the inner middleware, reaches the handler, and the response exits in reverse order. This demonstrates the onion model: each middleware has two execution moments separated by `next()`.

### Sub-Feature 1.2: Early Exit / Short-Circuiting Mechanics

#### Definitions

**Core Definition:** Short-circuiting is the practice of ending the request-response cycle early by sending a response or calling `next('route')` to skip remaining middleware, preventing unnecessary processing.

**Technical Definition:** If a middleware function does not end the request-response cycle, it must call `next()`. Otherwise, the request is left hanging. Middleware can short-circuit by: (1) sending a response (`res.send()`, `res.json()`) without calling `next()`, or (2) calling `next('route')` to skip the remaining middleware functions in a router middleware stack and pass control to the next route.

**Beginner-Friendly Explanation:** Short-circuiting is like a security guard who turns you away at the door instead of letting you walk through the entire building. If authentication fails, there's no point in running validation, logging, or the handler — so the middleware sends an error response immediately.

#### Purposes

- To reject invalid or unauthorized requests as early as possible.
- To avoid unnecessary computation and database queries.
- To implement pre-conditions on routes (e.g., skip to an alternative handler).

#### Syntax Rules and Structure

```javascript
// Short-circuit by sending a response
app.use((req, res, next) => {
  if (!req.headers.authorization) {
    return res.status(401).json({ error: 'Unauthorized' });
    // next() is NOT called — chain ends here
  }
  next();
});

// Skip to next route (only in app.METHOD() or router.METHOD())
app.get('/user/:id', (req, res, next) => {
  if (req.params.id === '0') {
    return next('route'); // Skip to the next matching route
  }
  res.json({ id: req.params.id });
});

app.get('/user/:id', (req, res) => {
  res.json({ special: true }); // Handles id=0
});
```

| Mechanism | Effect | Where Valid |
|-----------|--------|-------------|
| `res.send()` / `res.json()` without `next()` | Ends the chain | Any middleware |
| `next('route')` | Skips remaining callbacks in the current route | `app.METHOD()` or `router.METHOD()` only |
| `next('router')` | Skips the rest of the router's middleware | Router-level middleware |
| `next(err)` | Jumps to error-handling middleware | Any middleware |

**Constraints and Limitations:**
- `next('route')` works **only** in middleware loaded via `app.METHOD()` or `router.METHOD()`, not in `app.use()` middleware.
- Once a response is sent, calling `next()` may cause errors ("Cannot set headers after they are sent").

#### Annotated Code Example

```javascript
// short-circuit.js
const express = require('express');
const app = express();

// Auth middleware: short-circuits on missing token
function authenticate(req, res, next) {
  const token = req.headers.authorization;
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  req.user = { id: 1 };
  next();
}

// Validation middleware: short-circuits on invalid ID
function validateId(req, res, next) {
  const id = parseInt(req.params.id, 10);
  if (isNaN(id)) {
    return res.status(400).json({ error: 'Invalid ID' });
  }
  req.params.id = id;
  next();
}

app.get('/users/:id', authenticate, validateId, (req, res) => {
  res.json({ userId: req.params.id, user: req.user });
});

app.listen(3000);
```

**Expected Output (for `GET /users/abc` with valid token):**
```json
{"error":"Invalid ID"}
```

**Expected Output (for `GET /users/42` without token):**
```json
{"error":"No token provided"}
```

**Why this output:** The `authenticate` middleware short-circuits if the token is missing. The `validateId` middleware short-circuits if the ID is not a number. Only when both pass does the handler execute.

### Sub-Feature 1.3: Intercepting Responses (Upstream vs. Downstream Processing)

#### Definitions

**Core Definition:** Response interception is the practice of modifying the response after the handler has executed but before it is sent to the client, using the post-`next()` phase or `res.on('finish')`.

**Technical Definition:** In the onion model, middleware can perform logic after calling `next()` — this runs after the downstream handler has executed. Additionally, `res.on('finish')` fires after the response has been sent, enabling post-response tasks like logging or metrics.

**Beginner-Friendly Explanation:** Response interception is like a quality inspector who checks a product after it comes off the assembly line. The handler produces the response, and the interceptor can add headers, transform data, or log the outcome before it reaches the client.

#### Purposes

- To add response headers (e.g., `X-Response-Time`, `X-Request-ID`).
- To transform the response body into a standard envelope.
- To log response status codes and timing.
- To clean up resources after the response is sent.

#### Syntax Rules and Structure

```javascript
// Post-next() interception
app.use((req, res, next) => {
  const start = Date.now();
  next();
  const duration = Date.now() - start;
  res.setHeader('X-Response-Time', `${duration}ms`);
});

// Post-response interception
app.use((req, res, next) => {
  res.on('finish', () => {
    console.log(`${req.method} ${req.url} ${res.statusCode}`);
  });
  next();
});
```

**Constraints and Limitations:**
- Headers must be set before the response is sent; post-`next()` runs before `res.send()` is called by the handler, so headers can still be set.
- `res.on('finish')` fires after the response is sent; headers cannot be modified at this point.

#### Annotated Code Example

```javascript
// response-interception.js
const express = require('express');
const app = express();

// Upstream: timing middleware
app.use((req, res, next) => {
  const start = process.hrtime.bigint();
  next(); // Handler runs
  const duration = Number(process.hrtime.bigint() - start) / 1e6;
  res.setHeader('X-Response-Time', `${duration.toFixed(2)}ms`);
  console.log(`[${req.method} ${req.url}] ${duration.toFixed(2)}ms`);
});

// Downstream: response formatting
app.use((req, res, next) => {
  const originalJson = res.json.bind(res);
  res.json = (body) => {
    return originalJson({
      data: body,
      meta: { timestamp: new Date().toISOString() },
    });
  };
  next();
});

app.get('/api/users', (req, res) => {
  res.json([{ id: 1, name: 'Alice' }]);
});

app.listen(3000);
```

**Expected Output:**
```
[GET /api/users] 2.34ms
```

**Response headers:** `X-Response-Time: 2.34ms`

**Response body:**
```json
{
  "data": [{ "id": 1, "name": "Alice" }],
  "meta": { "timestamp": "2026-01-15T12:00:00.000Z" }
}
```

**Why this output:** The timing middleware captures the start time before `next()`, then sets the `X-Response-Time` header after the handler runs. The response formatting middleware overrides `res.json` to wrap the payload in a standard envelope.

---

## Core Concept 2: Conditional Middleware

### Sub-Feature 2.1: Dynamic Execution Based on Environment Variables

#### Definitions

**Core Definition:** Conditional middleware dynamically enables or disables middleware based on runtime conditions such as `NODE_ENV`, configuration flags, or other environment variables.

**Technical Definition:** Middleware can be conditionally registered using standard JavaScript conditionals. For example, logging middleware is often enabled only in development, while security headers are enabled in all environments.

**Beginner-Friendly Explanation:** Conditional middleware is like turning on the lights in a room only when someone is there. In development, you want detailed logging; in production, you might want less verbose logging but stricter security.

#### Purposes

- To enable verbose logging only in development.
- To disable debugging endpoints in production.
- To apply different security policies based on environment.
- To toggle experimental features without code changes.

#### Syntax Rules and Structure

```javascript
const isDev = process.env.NODE_ENV !== 'production';

// Conditional: only in development
if (isDev) {
  app.use(morgan('dev'));
  app.get('/debug', debugHandler);
}

// Conditional: only in production
if (!isDev) {
  app.use(helmet());
  app.use(rateLimit({ windowMs: 60000, limit: 100 }));
}
```

**Constraints and Limitations:**
- Conditionals must be evaluated at startup, not per-request (for performance).
- Environment variables should be validated at boot (e.g., with Zod).

#### Annotated Code Example

```javascript
// conditional-environment.js
const express = require('express');
const morgan = require('morgan');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const app = express();

const isProduction = process.env.NODE_ENV === 'production';

// Development-only middleware
if (!isProduction) {
  app.use(morgan('dev')); // Verbose logging
  app.get('/debug/config', (req, res) => {
    res.json({ env: process.env.NODE_ENV, memory: process.memoryUsage() });
  });
}

// Production-only middleware
if (isProduction) {
  app.use(helmet()); // Security headers
  app.use(rateLimit({ windowMs: 60000, limit: 100 }));
}

app.get('/api/data', (req, res) => {
  res.json({ data: 'Hello' });
});

app.listen(3000);
```

**Expected Output (in development):**
```
GET /api/data 200 3.2ms - 12
```
*(Morgan logs the request.)*

**Expected Output (in production):**
```
Request succeeds with security headers; /debug/config returns 404.
```

**Why this output:** The `isProduction` flag controls which middleware is registered. In development, Morgan is active and the debug endpoint exists. In production, Helmet and rate limiting are active, and the debug endpoint is not registered.

### Sub-Feature 2.2: Path-Matching Exclusions and Regex Filtering

#### Definitions

**Core Definition:** Path-matching exclusions skip middleware for certain paths, while regex filtering applies middleware only to paths matching a pattern.

**Technical Definition:** The `express-unless` package conditionally skips middleware when a condition is met. The `path` option can be a string, regexp, or array of either; if the request path matches, the middleware will not run. Alternatively, `app.use()` with a regex path applies middleware only to matching paths.

**Beginner-Friendly Explanation:** Path exclusions are like a bouncer who lets everyone in except people wearing red shirts. Regex filtering is like a bouncer who only lets in people whose names start with "A."

#### Purposes

- To skip authentication for public paths (e.g., `/login`, `/register`).
- To apply middleware only to API routes (`/api/*`).
- To exclude static assets from logging or rate limiting.

#### Syntax Rules and Structure

```javascript
// express-unless: skip middleware for specific paths
const { unless } = require('express-unless');
app.use(authenticate.unless({ path: ['/login', '/register'] }));

// Regex path matching
app.use('/api', apiMiddleware);       // Prefix match
app.use(/^\/admin\/.*/, adminGuard);  // Regex match
app.use(/^(?!\/public).*/, authGuard); // Negative lookahead
```

| Technique | Syntax | Effect |
|-----------|--------|--------|
| Path prefix | `app.use('/api', mw)` | Runs for `/api/*` |
| Regex | `app.use(/^\/admin/, mw)` | Runs for paths matching regex |
| Negative lookahead | `app.use(/^(?!\/public)/, mw)` | Runs for all paths except `/public` |
| `express-unless` | `mw.unless({ path: ['/login'] })` | Skips middleware for `/login` |

**Constraints and Limitations:**
- `express-unless` must be attached to the middleware function before use.
- Negative lookaheads can be difficult to read and maintain.
- Order matters: exclusion middleware must be registered before the middleware it excludes.

#### Annotated Code Example

```javascript
// conditional-path.js
const express = require('express');
const { unless } = require('express-unless');
const app = express();

// Simulated auth middleware
function authenticate(req, res, next) {
  const token = req.headers.authorization;
  if (!token) return res.status(401).json({ error: 'Unauthorized' });
  req.user = { id: 1 };
  next();
}
authenticate.unless = unless;

// Apply auth to all routes EXCEPT /login and /register
app.use(authenticate.unless({
  path: ['/login', '/register'],
}));

// Public routes (no auth required)
app.post('/login', (req, res) => res.json({ loggedIn: true }));
app.post('/register', (req, res) => res.json({ registered: true }));

// Protected routes
app.get('/dashboard', (req, res) => res.json({ user: req.user }));

// Regex: apply logging only to /api/* routes
app.use('/api', (req, res, next) => {
  console.log(`API request: ${req.method} ${req.url}`);
  next();
});

app.get('/api/data', (req, res) => res.json({ data: 'API data' }));

app.listen(3000);
```

**Expected Output (for `POST /login` without token):**
```json
{"loggedIn":true}
```

**Expected Output (for `GET /dashboard` without token):**
```json
{"error":"Unauthorized"}
```

**Expected Output (for `GET /api/data` with token):**
```
API request: GET /api/data
{"data":"API data"}
```

**Why this output:** `authenticate.unless({ path: ['/login', '/register'] })` skips authentication for the login and register routes. All other routes require a token. The `/api` middleware applies only to paths starting with `/api`.

### Sub-Feature 2.3: Feature-Flagged Middleware Activation

#### Definitions

**Core Definition:** Feature-flagged middleware is conditionally activated based on a feature flag value, enabling controlled rollouts, A/B testing, and gradual feature deployment.

**Technical Definition:** Feature flags can be evaluated in middleware to gate routes. The middleware asks a narrow question: "Is this feature enabled for this user?" If true, it calls `next()`; otherwise, it returns a fallback response or skips the route.

**Beginner-Friendly Explanation:** Feature-flagged middleware is like a "coming soon" sign that can be flipped on or off without redeploying. You can enable a feature for 10% of users, or only for internal testers, by changing a flag value.

#### Purposes

- To roll out features gradually (canary releases).
- To A/B test different middleware behaviour.
- To disable problematic features instantly without redeployment.
- To gate beta endpoints for specific user groups.

#### Syntax Rules and Structure

```javascript
function featureGate(flagName) {
  return (req, res, next) => {
    if (featureFlags.isEnabled(flagName, req.user)) {
      return next(); // Feature enabled — proceed
    }
    return res.status(404).json({ error: 'Feature not available' });
  };
}

app.get('/beta/analytics', featureGate('newAnalytics'), analyticsHandler);
```

**Constraints and Limitations:**
- Feature flags add branching complexity; remove them once features are fully rolled out.
- Flag evaluation must be fast to avoid adding latency.
- Always provide a fallback path when the flag is disabled.

#### Annotated Code Example

```javascript
// feature-flagged.js
const express = require('express');
const app = express();

// Simulated feature flag service
const featureFlags = {
  isEnabled(flagName, user) {
    const flags = {
      newDashboard: true,
      betaApi: false,
    };
    return flags[flagName] === true;
  },
};

function featureGate(flagName) {
  return (req, res, next) => {
    if (featureFlags.isEnabled(flagName, req.user)) {
      return next();
    }
    return res.status(404).json({ error: `Feature '${flagName}' not available` });
  };
}

// Simulated auth (attaches req.user)
app.use((req, res, next) => {
  req.user = { id: 1, role: 'user' };
  next();
});

// Feature-flagged routes
app.get('/dashboard', featureGate('newDashboard'), (req, res) => {
  res.json({ dashboard: 'v2' });
});

app.get('/api/beta', featureGate('betaApi'), (req, res) => {
  res.json({ beta: true });
});

app.listen(3000);
```

**Expected Output (for `GET /dashboard`):**
```json
{"dashboard":"v2"}
```

**Expected Output (for `GET /api/beta`):**
```json
{"error":"Feature 'betaApi' not available"}
```

**Why this output:** The `newDashboard` flag is enabled, so the dashboard route proceeds. The `betaApi` flag is disabled, so the beta route returns 404. Changing the flag value in the `featureFlags` object instantly changes behaviour without code changes.

---

## Core Concept 3: Route-Specific Middleware

### Sub-Feature 3.1: Route Grouping and Sub-Router Isolation

#### Definitions

**Core Definition:** Route grouping uses `express.Router()` instances to create modular, isolated middleware stacks that apply only to a specific set of routes.

**Technical Definition:** Router-level middleware works the same way as application-level middleware, except it is bound to an instance of `express.Router()`. Router-level middleware allows you to break your app into modular components and apply middleware selectively to a group of routes.

**Beginner-Friendly Explanation:** A router is like a department in a company. The company (app) has many departments (routers), each with its own staff (middleware) and responsibilities (routes). The authentication department handles auth routes; the user department handles user routes. Each department has its own rules that don't affect other departments.

#### Purposes

- To modularise the application into feature-based or domain-based modules.
- To apply middleware only to a specific group of routes.
- To enable independent development and testing of route modules.
- To isolate concerns (e.g., admin routes vs. public routes).

#### Syntax Rules and Structure

```javascript
const router = express.Router();

// Router-level middleware (applies only to this router)
router.use(loggingMiddleware);
router.use(authMiddleware);

// Routes on the router
router.get('/', listHandler);
router.get('/:id', getHandler);

// Mount the router on the app
app.use('/api/users', router);
```

| Aspect | Application-Level | Router-Level |
|--------|------------------|--------------|
| Binding | `app.use()` | `router.use()` |
| Scope | Entire app | Router only |
| Modularity | Monolithic | Modular |
| Reusability | Limited | High |

**Constraints and Limitations:**
- Middleware added via one router may run for other routers if its routes match.
- Parameters from parent routers require `{ mergeParams: true }` on the child router.

#### Annotated Code Example

```javascript
// router-grouping.js
const express = require('express');
const app = express();

// Users router
const usersRouter = express.Router();
usersRouter.use((req, res, next) => {
  console.log('Users router middleware');
  next();
});
usersRouter.get('/', (req, res) => res.json({ users: [] }));
usersRouter.get('/:id', (req, res) => res.json({ userId: req.params.id }));

// Admin router (different middleware)
const adminRouter = express.Router();
adminRouter.use((req, res, next) => {
  console.log('Admin router middleware');
  next();
});
adminRouter.get('/dashboard', (req, res) => res.json({ admin: true }));

// Mount routers
app.use('/api/users', usersRouter);
app.use('/api/admin', adminRouter);

app.listen(3000);
```

**Expected Output (for `GET /api/users/42`):**
```
Users router middleware
{"userId":"42"}
```

**Expected Output (for `GET /api/admin/dashboard`):**
```
Admin router middleware
{"admin":true}
```

**Why this output:** Each router has its own middleware that runs only for routes within that router. The users router's middleware does not run for admin routes, and vice versa. This demonstrates router isolation.

### Sub-Feature 3.2: Inline Route Guards

#### Definitions

**Core Definition:** Inline route guards are middleware functions passed directly as arguments to route handlers, providing route-specific pre-condition checks.

**Technical Definition:** Route handlers enable you to define multiple routes for a path. Multiple callbacks can be provided, and all are treated equally, behaving just like middleware — except these callbacks may invoke `next('route')` to bypass the remaining route callback(s).

**Beginner-Friendly Explanation:** Inline guards are like a personal assistant who checks your credentials before you enter a specific meeting room. Other meeting rooms have their own guards, but this one checks you specifically for this room.

#### Purposes

- To apply validation or authorization to a single route.
- To keep route-specific logic close to the handler.
- To avoid polluting global middleware with route-specific concerns.

#### Syntax Rules and Structure

```javascript
app.get('/users/:id', authenticate, validateId, (req, res) => {
  res.json({ userId: req.params.id });
});
```

**Constraints and Limitations:**
- Guards must be listed before the handler.
- `next('route')` works only in route-level middleware.

#### Annotated Code Example

```javascript
// inline-guards.js
const express = require('express');
const app = express();

function requireAdmin(req, res, next) {
  if (req.user?.role !== 'admin') {
    return res.status(403).json({ error: 'Admin required' });
  }
  next();
}

app.use((req, res, next) => {
  req.user = { id: 1, role: 'user' };
  next();
});

app.delete('/users/:id', requireAdmin, (req, res) => {
  res.json({ deleted: true });
});

app.delete('/own/:id', (req, res, next) => {
  if (req.user.id !== parseInt(req.params.id)) {
    return res.status(403).json({ error: 'Not your resource' });
  }
  next();
}, (req, res) => {
  res.json({ deleted: true });
});

app.listen(3000);
```

**Expected Output (for `DELETE /users/42` with role `user`):**
```json
{"error":"Admin required"}
```

**Expected Output (for `DELETE /own/1`):**
```json
{"deleted":true}
```

**Why this output:** The `requireAdmin` guard rejects non-admin users. The inline `own` guard checks that the user owns the resource. Each guard is specific to its route.

### Sub-Feature 3.3: Controller-Level Decorators (NestJS Comparison)

#### Definitions

**Core Definition:** Controller-level decorators are a NestJS pattern (not native Express) that apply middleware to all routes within a controller class, providing declarative middleware composition.

**Technical Definition:** In NestJS, decorators like `@UseGuards()`, `@UseInterceptors()`, and `@UseFilters()` can be applied at the controller class level, affecting all routes in that controller. This is a higher-level abstraction not available in native Express.

**Beginner-Friendly Explanation:** Decorators are like putting a sign on a door that says "all visitors must check in at reception." Every person entering that room goes through reception, without the receptionist having to be listed individually for each visitor.

#### Purposes

- To apply guards, interceptors, or filters to all routes in a controller.
- To reduce boilerplate by centralising cross-cutting concerns.
- To enable declarative middleware composition.

#### Syntax Rules and Structure

```typescript
@Controller('cats')
@UseGuards(AuthGuard)
@UseInterceptors(LoggingInterceptor)
export class CatsController {
  @Get()
  findAll() { return []; }
}
```

**Constraints and Limitations:**
- Native Express does not support controller-level decorators; this is a NestJS feature.
- For Express, use router-level middleware mounted on a router to achieve similar grouping.

#### Annotated Code Example

```javascript
// Express equivalent of controller-level grouping
const catsRouter = express.Router();

// Apply to all routes in this router (like controller-level decorator)
catsRouter.use(authGuard);
catsRouter.use(loggingInterceptor);

catsRouter.get('/', (req, res) => res.json([]));
catsRouter.get('/:id', (req, res) => res.json({ id: req.params.id }));

app.use('/cats', catsRouter);
```

**Expected Output (for `GET /cats` without token):**
```json
{"error":"Unauthorized"}
```

**Why this output:** The `catsRouter` has `authGuard` applied at the router level, so every route within it requires authentication. This is the Express equivalent of a controller-level guard.

---

## Core Concept 4: Global Middleware

### Sub-Feature 4.1: Application-Wide Bootstrap Middleware

#### Definitions

**Core Definition:** Global middleware is registered with `app.use()` and applies to every request that reaches the application, regardless of path.

**Technical Definition:** Express ships with no security by default — no security headers, no CSRF protection, no input validation, and no rate limiting. Security middleware must be registered before routes, or requests reach handlers before the controls apply.

**Beginner-Friendly Explanation:** Global middleware is like the building's main entrance security — everyone who enters must pass through it, no matter which office they're visiting.

#### Purposes

- To apply security headers (Helmet) to all responses.
- To parse request bodies for all routes.
- To log every incoming request.
- To enforce rate limits globally.

#### Syntax Rules and Structure

```javascript
// Bootstrap order: security first, error handler last
app.use(helmet());                          // 1. Security headers
app.use(express.json({ limit: '100kb' }));  // 2. Body parsing
app.use(morgan('combined'));                // 3. Logging
app.use(rateLimit({ windowMs: 60000, limit: 100 })); // 4. Rate limiting
app.use(authMiddleware);                    // 5. Authentication
// ... routes ...
app.use(errorHandler);                      // 6. Error handling (4-arg, last)
```

| Order | Middleware | Purpose |
|-------|-----------|---------|
| 1 | Helmet | Security headers |
| 2 | Body parser | Parse request bodies |
| 3 | Logger | Request logging |
| 4 | Rate limiter | Abuse prevention |
| 5 | Auth | Authentication |
| 6 | Routes | Business logic |
| 7 | Error handler | Error formatting (4 args, last) |

**Constraints and Limitations:**
- Middleware registered after routes will not apply to those routes.
- Error handlers must be registered last and have exactly 4 parameters.

#### Annotated Code Example

```javascript
// global-bootstrap.js
const express = require('express');
const helmet = require('helmet');
const morgan = require('morgan');
const rateLimit = require('express-rate-limit');
const app = express();

// 1. Security headers
app.use(helmet());

// 2. Body parsing (with size limit)
app.use(express.json({ limit: '100kb' }));

// 3. Logging
app.use(morgan('combined'));

// 4. Rate limiting
app.use(rateLimit({
  windowMs: 60 * 1000,
  limit: 100,
  standardHeaders: 'draft-7',
}));

// 5. Routes
app.get('/api/data', (req, res) => {
  res.json({ data: 'Hello' });
});

// 6. Error handler (must be last, 4 args)
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: 'Internal Server Error' });
});

app.listen(3000);
```

**Expected Output (for `GET /api/data`):**
```
Response includes security headers from Helmet, rate limit headers, and request is logged by Morgan.
```

**Why this output:** Helmet sets security headers, the body parser handles JSON, Morgan logs the request, and the rate limiter tracks the request. The error handler catches any unhandled errors.

### Sub-Feature 4.2: Third-Party Middleware Integration (Helmet for Security Headers)

#### Definitions

**Core Definition:** Helmet is a collection of middleware functions that set HTTP response headers to protect against well-known web vulnerabilities, including XSS, clickjacking, and MIME sniffing.

**Technical Definition:** Helmet sets a bundle of protective HTTP headers in one line. A tuned Content-Security-Policy is the part worth the extra effort: it's your last line of defence against XSS, because even an injected script won't execute if it violates the policy.

**Beginner-Friendly Explanation:** Helmet is like putting a security system on your house. It adds locks (headers) that prevent burglars (attackers) from breaking in through known weak points.

#### Purposes

- To set Content-Security-Policy (CSP) to prevent XSS.
- To set Strict-Transport-Security (HSTS) to enforce HTTPS.
- To set X-Content-Type-Options to prevent MIME sniffing.
- To set X-Frame-Options to prevent clickjacking.

#### Syntax Rules and Structure

```javascript
import helmet from 'helmet';

// Default: sensible headers
app.use(helmet());

// Tuned CSP
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    objectSrc: ["'none'"],
    frameAncestors: ["'none'"],
  },
}));
```

**Constraints and Limitations:**
- Helmet must be registered before routes.
- CSP may break legitimate functionality if configured too strictly.

#### Annotated Code Example

```javascript
// helmet-config.js
const express = require('express');
const helmet = require('helmet');
const app = express();

app.use(helmet());

app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", 'https://cdn.example.com'],
    styleSrc: ["'self'", "'unsafe-inline'"],
    imgSrc: ["'self'", 'data:', 'https:'],
    objectSrc: ["'none'"],
    frameAncestors: ["'none'"],
  },
}));

app.get('/', (req, res) => {
  res.send('<h1>Hello</h1>');
});

app.listen(3000);
```

**Expected Output (response headers):**
```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.example.com; ...
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
```

**Why this output:** Helmet sets the default security headers. The custom CSP allows scripts only from the same origin and a trusted CDN, blocks object embeds, and prevents framing.

### Sub-Feature 4.3: Rate Limiting and DDoS Protection at the Global Level

#### Definitions

**Core Definition:** Global rate limiting middleware enforces a maximum number of requests per client (typically per IP) within a time window, protecting against abuse, brute-force attacks, and denial-of-service.

**Technical Definition:** Express will happily accept unlimited requests per second, which is a denial-of-service vector. Cap them with `express-rate-limit`. Apply a stricter limit to authentication endpoints — login and password-reset routes are where credential-stuffing lands.

**Beginner-Friendly Explanation:** Global rate limiting is like a bouncer at the door who counts how many people enter per minute. If a group sends 1,000 people at once, only 100 get in per minute — the rest wait or are turned away.

#### Purposes

- To prevent brute-force attacks on login endpoints.
- To protect against request floods and denial-of-service.
- To enforce fair usage across all clients.
- To communicate limits via standard headers.

#### Syntax Rules and Structure

```javascript
const rateLimit = require('express-rate-limit');

// Global limiter
app.use(rateLimit({
  windowMs: 60 * 1000,  // 1 minute
  limit: 100,            // 100 requests per IP per minute
  standardHeaders: 'draft-7',
}));

// Stricter limiter for auth routes
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  limit: 5,                  // 5 attempts
});
app.use('/api/auth', authLimiter);
```

| Option | Description | Recommended |
|--------|-------------|-------------|
| `windowMs` | Time window in ms | 60,000 |
| `limit` | Max requests per window | 100 |
| `standardHeaders` | Emit `RateLimit-*` headers | `'draft-7'` |
| `store` | Storage backend | Redis for multi-pod |

**Constraints and Limitations:**
- Default MemoryStore does not share state across instances; use Redis for distributed deployments.
- `trust proxy` must be configured correctly for accurate IP detection.

#### Annotated Code Example

```javascript
// rate-limit-global.js
const express = require('express');
const rateLimit = require('express-rate-limit');
const app = express();

// Global rate limit
app.use(rateLimit({
  windowMs: 60 * 1000,
  limit: 100,
  standardHeaders: 'draft-7',
  legacyHeaders: false,
}));

// Strict limit for auth
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  limit: 5,
  standardHeaders: 'draft-7',
  legacyHeaders: false,
  message: { error: 'Too many login attempts' },
});

app.post('/api/auth/login', authLimiter, (req, res) => {
  res.json({ token: 'jwt' });
});

app.get('/api/data', (req, res) => {
  res.json({ data: 'Hello' });
});

app.listen(3000);
```

**Expected Output (for the 6th login attempt within 15 minutes):**
```http
HTTP/1.1 429 Too Many Requests
RateLimit-Limit: 5
RateLimit-Remaining: 0
Retry-After: 847

{"error":"Too many login attempts"}
```

**Expected Output (for the 50th API request within 1 minute):**
```http
HTTP/1.1 200 OK
RateLimit-Limit: 100
RateLimit-Remaining: 50
RateLimit-Reset: 30
```

**Why this output:** The global limiter allows 100 requests per minute. The auth limiter allows 5 attempts per 15 minutes. The `standardHeaders: 'draft-7'` emits the `RateLimit-*` headers, communicating limits to clients.

---

## References

- Express.js — Using Middleware — https://expressjs.com/en/guide/using-middleware.html
- Express.js — Router — https://expressjs.com/en/guide/routing.html#express-router
- express-unless — Conditionally Skip Middleware — https://app.unpkg.com/express-unless
- Safeguard.sh — Securing Express.js Applications (Express 5) — https://safeguard.sh/resources/blog/securing-expressjs-applications
- DEV Community — Express Middleware Patterns: Composition, Error Handling, and Auth (2026 Guide) — https://dev.to/young_gao/middleware-patterns-in-express-composition-error-handling-and-auth-k16
- Statsig — Feature Flags in NodeJS ExpressJS — https://docs.statsig.com/feature-flags/express
- Helmet.js — Security Headers — https://helmet.js.org/
- express-rate-limit — Rate Limiting Middleware — https://github.com/express-rate-limit/express-rate-limit
- FreeCodeCamp — Router-Level Middleware — https://raw.githubusercontent.com/Ksound22/freeCodeCamp/main/curriculum/challenges/english/blocks/lecture-express-middleware/69f42d414b7626e21a922793.md
- GitHub — Express Middleware Onion Model — https://github.com/wpCN/redux-simple-tutorial/blob/master/middleware-onion-model.md