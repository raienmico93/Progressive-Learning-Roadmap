# Separation of Concerns (SoC) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Separation of Concerns (SoC) is a software design principle that advocates for dividing an application into distinct, non-overlapping units, where each unit addresses a single, well-defined concern or responsibility.

**Technical Definition:** Separation of Concerns is a principle used in programming to separate an application into units, with minimal overlapping between the functions of the individual units. The separation of concerns is achieved using modularization, encapsulation, and arrangement in software layers. The SoC principle identifies the parts of an application with a specific purpose and encapsulates these parts in closed units. These units only communicate with each other using specified interfaces. 

**Beginner-Friendly Explanation:** Imagine building a house. You wouldn't ask the electrician to also do the plumbing, and you wouldn't ask the plumber to install the windows. Each professional has one job, and they work on their part without interfering with the others. Separation of Concerns applies the same idea to code: routing should only handle routing, business logic should only handle business rules, and database queries should only live in the data access layer. When each part does one thing well, the whole system becomes easier to build, test, and maintain.

### Key Characteristics

- **Modularity:** The application is divided into discrete, self-contained units. 
- **Single Responsibility:** Each unit has one clear purpose and one reason to change. 
- **Explicit Interfaces:** Units communicate through well-defined, minimal interfaces. 
- **Testability:** Isolated units can be tested independently without the full application context. 
- **Replaceability:** Changing one unit (e.g., swapping a database) should not cascade changes through unrelated units. 
- **Framework Independence:** Core business rules should be portable and testable without HTTP state or database connections. 

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript knowledge:** Functions, modules, and asynchronous programming.
- **Understanding of Express routing and middleware.**
- **Familiarity with a database or ORM** (e.g., Sequelize, Prisma, Mongoose).

### Related Programming Areas

- **Layered Architecture (N-Tier):** SoC is the fundamental principle behind organising code into horizontal layers. 
- **MVC Architecture:** Model-View-Controller is a specialised application of SoC for interactive applications. 
- **Modular Architecture:** Feature modules apply SoC at the feature level, encapsulating all concerns for a single domain.
- **Domain-Driven Design:** SoC underpins the separation of domain logic from infrastructure concerns.
- **Clean Architecture:** The dependency rule (inner layers know nothing of outer layers) is a direct application of SoC. 

### Core Concepts

1. **Routing Logic** — restricting responsibility solely to directing URL variations and forwarding requests.
2. **Business Logic** — isolating calculations, data permutations, and execution logic from the HTTP framework layer.
3. **Database Logic** — restricting data querying optimisation, transaction queries, and hydration purely to the repository layer.
4. **Validation** — utilising dedicated, reusable middleware early in the lifecycle to block execution before hitting controllers.
5. **Authentication & Authorization** — decoupling user identification layers entirely from the underlying resource logic via route-level middleware.
6. **Error Handling** — offloading exception formatting entirely to an independent global error interceptor rather than writing try/catch syntax inside controllers.

---

## Core Concept 1: Routing Logic

### Definitions

**Core Definition:** Routing logic is the responsibility of mapping incoming HTTP requests — defined by their URL path and HTTP method — to the appropriate controller function, without containing any application-specific logic beyond that mapping.

**Technical Definition:** In Express, routing logic is expressed using `express.Router()`. Each route specifies an HTTP method, a path, and one or more middleware functions or controller methods. The route should be only in charge of routing, not any business logic or operations, other than handing the task to a function (i.e., the controller). Routes should delegate to services, not contain business logic. 

**Beginner-Friendly Explanation:** Routing logic is like the directory in a building lobby. When you walk in (a request arrives), the directory tells you which floor and room to go to. The directory doesn't do any work itself — it just points you in the right direction. In Express, a route's only job is to say "for this URL and this HTTP method, call this controller function."

### Purposes

- To define the API's URL structure and HTTP method surface.
- To map incoming requests to the correct controller actions.
- To apply route-specific middleware (authentication, validation, rate limiting).
- To serve as the single source of truth for the API's endpoint surface.

### Syntax Rules and Structure

```js
// routes/users.routes.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/user.controller');
const { requireAuth } = require('../middleware/auth');
const { validate } = require('../middleware/validate');
const { createUserSchema } = require('../validators/user.validator');

// Public route
router.get('/', userController.getAll);
router.get('/:id', userController.getById);

// Protected route with validation
router.post('/', requireAuth, validate(createUserSchema), userController.create);
router.put('/:id', requireAuth, userController.update);
router.delete('/:id', requireAuth, userController.delete);

module.exports = router;
```

| Component | Breakdown |
|-----------|-----------|
| `express.Router()` | Creates a new router instance. |
| `router.get('/', ...)` | Maps GET requests at the router root to a controller method. |
| `userController.getAll` | The controller function to invoke. |
| `requireAuth` | Route-specific middleware for authentication. |
| `validate(createUserSchema)` | Route-specific middleware for request validation. |

**Rules:**
- Routes should be defined in dedicated files under a `routes/` directory. 
- Each route file should export an `express.Router()` instance.
- Route-specific middleware should be applied at the route level, not globally.
- Routes should **never** contain business logic, database queries, or response formatting. 
- Route paths should be resource-based and RESTful.

**Constraints and Limitations:**
- **Fat routes** (routes containing business logic) are the most severe violation of SoC — worse than fat controllers. 
- Routes are order-dependent: more specific routes must be defined before more general ones.
- Route-level middleware runs before the controller, so it can short-circuit the request.

### Annotated Code Example

```js
// ❌ BAD: Business logic mixed into the route
router.post('/users', async (req, res) => {
  if (!req.body.email) return res.status(400).json({ error: 'Email required' });
  const existing = await User.findOne({ email: req.body.email });
  if (existing) return res.status(409).json({ error: 'Email exists' });
  const hashed = await bcrypt.hash(req.body.password, 12);
  const user = await User.create({ ...req.body, password: hashed });
  res.status(201).json({ data: user });
});

// ✅ GOOD: Route delegates to controller; logic lives elsewhere
router.post('/users', validate(createUserSchema), userController.create);
```

**Expected Output (for the good example):**
```json
{
  "data": {
    "id": "1",
    "name": "Alice",
    "email": "alice@example.com"
  }
}
```

**Why this output:** The route file contains only the mapping — path, method, middleware, and controller reference. The controller handles request parsing; the service handles business rules; the repository handles database access. Each concern lives in its own layer. 

### Real-World Cases

- **REST APIs:** `/api/users`, `/api/products`, `/api/orders` — each resource has its own route file. 
- **Versioned APIs:** `/api/v1/users` and `/api/v2/users` — separate routers for each version.
- **Public vs. private routes:** Public routes defined before authentication middleware; private routes after.

---

## Core Concept 2: Business Logic

### Definitions

**Core Definition:** Business logic is the set of rules, calculations, and domain-specific operations that define how the application behaves — independent of any HTTP framework, database, or presentation concern.

**Technical Definition:** The service layer contains the core business logic of the application. Services are responsible for data transformation, business rules, and orchestrating interactions between repositories and external services. Business logic does not depend on infrastructure — clean architecture forces a separation of concerns, ensuring core business rules are testable in absolute isolation from HTTP state or database connections. 

**Beginner-Friendly Explanation:** Business logic is the "brain" of your application — the rules that make your business unique. For example, an e-commerce app's business logic knows that "a customer cannot order more items than are in stock" and "orders over $50 get free shipping." These rules have nothing to do with Express, databases, or HTTP. They're pure business rules that would be the same whether you're building a web app, a mobile app, or a CLI tool.

### Purposes

- To isolate calculations, data permutations, and execution logic from the HTTP framework layer.
- To ensure business rules are testable without HTTP state or database connections. 
- To orchestrate multiple repositories for complex operations.
- To enforce domain invariants and business rule violations.
- To integrate with external services (payment gateways, email providers).

### Syntax Rules and Structure

```js
// services/user.service.js
const { UserRepository } = require('../repositories/user.repository');
const { hashPassword } = require('../shared/crypto');
const { AppError } = require('../shared/errors/AppError');

class UserService {
  constructor() {
    this.userRepository = new UserRepository();
  }

  async create(data) {
    // Business rule: email must be unique
    const existing = await this.userRepository.findByEmail(data.email);
    if (existing) {
      throw new AppError('Email already exists', 409);
    }

    // Business rule: hash password before storage
    const hashedPassword = await hashPassword(data.password);

    return this.userRepository.create({
      ...data,
      password: hashedPassword
    });
  }

  async updateProfile(userId, data) {
    const user = await this.userRepository.findById(userId);
    if (!user) throw new AppError('User not found', 404);

    // Business rule: email change requires verification
    if (data.email && data.email !== user.email) {
      // Orchestrate email verification flow
      await this.sendVerificationEmail(data.email);
    }

    return this.userRepository.update(userId, data);
  }
}

module.exports = { UserService };
```

| Component | Breakdown |
|-----------|-----------|
| `UserRepository` | The injected data access layer. |
| `hashPassword()` | A shared utility for password hashing. |
| `AppError` | A custom error class for domain errors. |
| `sendVerificationEmail()` | An orchestration call to an external service. |

**Rules:**
- Services should be defined in a dedicated `services/` directory. 
- Services should **never** import `req` or `res` — they are transport-agnostic.
- Services should throw domain-specific errors rather than HTTP errors.
- Services should orchestrate repositories, not directly access the database.
- Business logic should be testable without spinning up an HTTP server. 

**Constraints and Limitations:**
- Services that become too large (e.g., 1000+ lines) may need to be split into smaller, focused services.
- Services should not know about HTTP status codes — that is the controller's responsibility.
- Transaction management across multiple repositories can be complex; use a unit of work or transaction manager.

### Annotated Code Example

```js
// services/order.service.js
const { OrderRepository } = require('../repositories/order.repository');
const { ProductRepository } = require('../repositories/product.repository');
const { PaymentService } = require('./payment.service');
const { AppError } = require('../shared/errors/AppError');

class OrderService {
  constructor() {
    this.orderRepository = new OrderRepository();
    this.productRepository = new ProductRepository();
    this.paymentService = new PaymentService();
  }

  async createOrder(userId, items) {
    // Business rule: validate items exist and are in stock
    let total = 0;
    for (const item of items) {
      const product = await this.productRepository.findById(item.productId);
      if (!product) {
        throw new AppError(`Product ${item.productId} not found`, 404);
      }
      if (product.stock < item.quantity) {
        throw new AppError(`Insufficient stock for ${product.name}`, 409);
      }
      total += product.price * item.quantity;
    }

    // Orchestrate: charge payment
    const payment = await this.paymentService.charge(userId, total);

    // Orchestrate: create order
    const order = await this.orderRepository.create({
      userId,
      items,
      total,
      paymentId: payment.id,
      status: 'paid'
    });

    // Orchestrate: decrement inventory
    for (const item of items) {
      await this.productRepository.decrementStock(item.productId, item.quantity);
    }

    return order;
  }
}
```

**Expected Output (for a valid order):**
```json
{
  "id": "order-1",
  "userId": "user-1",
  "items": [{ "productId": "prod-1", "quantity": 2 }],
  "total": 2400,
  "paymentId": "pay-123",
  "status": "paid"
}
```

**Why this output:** The service orchestrates multiple operations — validating products, charging payment, creating the order, and decrementing inventory. All business rules are enforced in the service layer. The controller simply calls `orderService.createOrder(userId, items)` and returns the result. The service knows nothing about HTTP. 

### Real-World Cases

- **E-commerce:** `OrderService` orchestrating payment, inventory, and order creation.
- **Authentication:** `AuthService` handling password hashing, JWT generation, and token validation.
- **SaaS platforms:** `SubscriptionService` handling plan changes, billing, and feature access.

---

## Core Concept 3: Database Logic

### Definitions

**Core Definition:** Database logic is the responsibility of querying, optimising, hydrating, and persisting data — and it should be confined exclusively to the repository layer, keeping raw queries out of controllers and services.

**Technical Definition:** The data-access layer is where the app holds code that interacts with the database. It should externalise an interface that returns or gets plain JavaScript objects that are database-agnostic — also known as the repository pattern. This layer involves database helper utilities like query builders, ORMs, database drivers, and other implementation libraries. The repository pattern separates data access from business logic by wrapping ORM queries behind a typed interface. 

**Beginner-Friendly Explanation:** Database logic is like the pantry manager in a restaurant. The pantry manager knows exactly where every ingredient is stored, how to retrieve it, and how to restock it. The chef (service) asks the pantry manager for ingredients, and the pantry manager gets them — the chef doesn't need to know whether the ingredients are in a refrigerator, a freezer, or a dry storage room. If the restaurant switches suppliers (changes databases), only the pantry manager needs to learn the new system.

### Purposes

- To restrict data querying optimisation, transaction queries, and hydration purely to the repository layer.
- To abstract raw database queries behind interface functions.
- To allow the database technology to change without breaking business logic. 
- To centralise all data access logic for a specific entity.
- To keep raw queries out of controllers and services.

### Syntax Rules and Structure

```js
// repositories/user.repository.js
const { User } = require('../models/User');

class UserRepository {
  async findAll() {
    return User.findAll();
  }

  async findById(id) {
    return User.findByPk(id);
  }

  async findByEmail(email) {
    return User.findOne({ where: { email } });
  }

  async create(data) {
    return User.create(data);
  }

  async update(id, data) {
    await User.update(data, { where: { id } });
    return this.findById(id);
  }

  async delete(id) {
    return User.destroy({ where: { id } });
  }
}

module.exports = { UserRepository };
```

| Component | Breakdown |
|-----------|-----------|
| `User` | The ORM model. |
| `findAll()` | Retrieves all records. |
| `findById(id)` | Retrieves a record by primary key. |
| `findByEmail(email)` | A custom query method. |
| `create(data)` | Inserts a new record. |

**Rules:**
- Repositories should be defined in a dedicated `repositories/` directory. 
- Each repository should handle one entity (e.g., `user.repository.js`).
- Repositories should expose methods that match the domain's needs, not just generic CRUD.
- Services should **never** import models directly — they must go through repositories.
- Repositories should return domain entities, not raw database rows.

**Constraints and Limitations:**
- Repositories should not contain business logic — that belongs in services.
- Complex queries that span multiple entities may require a repository method that returns a composite result.
- The repository interface should be stable; changing it affects all services that depend on it.

### Annotated Code Example

```js
// ❌ BAD: Database query in the controller
app.get('/api/users/:id', async (req, res) => {
  const user = await User.findByPk(req.params.id); // Database logic in controller
  res.json(user);
});

// ✅ GOOD: Controller delegates to service; service delegates to repository
// controllers/user.controller.js
const { UserService } = require('../services/user.service');

class UserController {
  constructor() {
    this.userService = new UserService();
  }

  getById = async (req, res, next) => {
    try {
      const user = await this.userService.findById(req.params.id);
      if (!user) return res.status(404).json({ error: 'User not found' });
      res.json({ data: user });
    } catch (error) {
      next(error);
    }
  };
}
```

**Expected Output (for `GET /api/users/1`):**
```json
{
  "data": {
    "id": "1",
    "name": "Alice",
    "email": "alice@example.com"
  }
}
```

**Why this output:** The controller calls `userService.findById()`. The service calls `userRepository.findById()`. The repository executes the ORM query. Each layer has a single concern. If the database changes from PostgreSQL to MongoDB, only the repository layer needs modification. 

### Real-World Cases

- **E-commerce:** `OrderRepository`, `ProductRepository`, `UserRepository` — each abstracts queries for one entity. 
- **Content platforms:** `PostRepository`, `CommentRepository`.
- **Database migration:** Swapping from MongoDB to PostgreSQL requires only rewriting the repository layer — services remain unchanged. 

---

## Core Concept 4: Validation

### Definitions

**Core Definition:** Validation is the process of checking that incoming request data (body, query parameters, path parameters) conforms to expected types, formats, and constraints — performed by dedicated, reusable middleware early in the request lifecycle, before the request reaches the controller.

**Technical Definition:** Validation middleware uses a schema definition (from Zod, Joi, or express-validator) to validate and sanitise `req.body`, `req.query`, and `req.params`. If validation fails, the middleware returns a 422 Unprocessable Entity or 400 Bad Request response and **never calls `next()`** — the request never reaches the controller. A valid request passes through `next()` unchanged and reaches the controller. 

**Beginner-Friendly Explanation:** Validation middleware is like a quality control inspector on an assembly line. Before any product (request data) moves to the next station (the controller), the inspector checks it against a specification. If it meets the spec, it passes through. If not, it's rejected immediately with a detailed report of what went wrong. The controller never sees invalid data.

### Purposes

- To utilise dedicated, reusable middleware early in the lifecycle to block execution before hitting controllers. 
- To sanitise and validate user input before it reaches business logic.
- To prevent injection attacks by rejecting malicious input.
- To enforce data integrity by ensuring required fields are present and correctly typed.
- To provide consistent, structured error responses for invalid input.

### Syntax Rules and Structure

```js
// middleware/validate.js — reusable validation factory
const { z } = require('zod');

function validate(schemas) {
  return (req, res, next) => {
    try {
      if (schemas.body) req.body = schemas.body.parse(req.body);
      if (schemas.params) req.params = schemas.params.parse(req.params);
      if (schemas.query) req.query = schemas.query.parse(req.query);
      next();
    } catch (error) {
      return res.status(422).json({
        error: 'Validation failed',
        details: error.flatten().fieldErrors
      });
    }
  };
}
```

```js
// validators/user.validator.js
const { z } = require('zod');

const createUserSchema = z.object({
  name: z.string().min(1, 'Name is required').max(100),
  email: z.string().email('Invalid email format'),
  password: z.string().min(8, 'Password must be at least 8 characters')
});

module.exports = { createUserSchema };
```

| Component | Breakdown |
|-----------|-----------|
| `validate(schemas)` | Factory function that returns validation middleware. |
| `schemas.body` | Zod schema for `req.body`. |
| `schemas.params` | Zod schema for `req.params`. |
| `schemas.query` | Zod schema for `req.query`. |
| `error.flatten().fieldErrors` | Zod's structured error format. |

**Rules:**
- Validation middleware must be registered **before** the controller in the route chain. 
- Validation middleware must call `next()` only if validation succeeds.
- On failure, the middleware must respond directly and **not** call `next()`. 
- Validation schemas should be reusable and defined in a dedicated `validators/` directory. 
- Server-side validation is mandatory even if client-side validation exists. 

**Constraints and Limitations:**
- Validation middleware should validate all relevant request parts (body, params, query).
- Overly complex schemas may be slow; keep validation focused on the HTTP boundary.
- Validation does not replace business-rule validation — business rules belong in the service layer.

### Annotated Code Example

```js
// routes/user.routes.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/user.controller');
const { validate } = require('../middleware/validate');
const { createUserSchema } = require('../validators/user.validator');

// Validation middleware runs BEFORE the controller
router.post('/users',
  validate({ body: createUserSchema }),  // Validation blocks invalid requests
  userController.create                  // Controller only receives valid data
);

module.exports = router;
```

**Expected Output (for a valid request):**
```json
{
  "data": {
    "id": "1",
    "name": "Alice",
    "email": "alice@example.com"
  }
}
```

**Expected Output (for an invalid request — missing email):**
```json
{
  "error": "Validation failed",
  "details": {
    "email": ["Invalid email format"]
  }
}
```

**Why this output:** The `validate` middleware parses the request body against the Zod schema. If validation succeeds, `next()` is called and the controller creates the user. If validation fails, the middleware returns a 422 response and the controller is **never invoked**. 

### Real-World Cases

- **User registration:** Validating email format and password strength before creating an account.
- **E-commerce orders:** Validating product IDs, quantities, and shipping addresses.
- **API gateways:** Validating all incoming requests before routing to services.
- **Form submissions:** Ensuring required fields are present and correctly formatted.

---

## Core Concept 5: Authentication & Authorization

### Definitions

**Core Definition:** Authentication is the process of verifying user identity; authorization is the process of checking whether the authenticated user has permission to access a resource. Both should be decoupled from the underlying resource logic and applied via route-level middleware.

**Technical Definition:** Authentication middleware extracts credentials from the request (typically the `Authorization` header), validates them, and attaches the user object to `req.user`. Authorization middleware reads `req.user` and checks the user's role or permissions against the requirements of the route. **Order matters: authentication attaches `req.user`; authorization reads it. Reverse the order and authorization fails.** Both concerns should be implemented as middleware, not mixed into controllers. 

**Beginner-Friendly Explanation:** Authentication is the bouncer checking your ID at the door — it verifies who you are. Authorization is the security guard inside the building who checks whether your badge gives you access to the executive floor. You can't check badges before you've verified identities. And neither the bouncer nor the security guard should be doing the actual work of the office — they just control access.

### Purposes

- To decouple user identification layers entirely from the underlying resource logic via route-level middleware.
- To verify client identity before allowing access to protected routes.
- To attach the authenticated user object to `req.user` for downstream use.
- To enforce role-based access control (RBAC) and permission checks.
- To reject unauthenticated requests with a 401 status and unauthorized requests with a 403 status.

### Syntax Rules and Structure

```js
// middleware/auth.js
const jwt = require('jsonwebtoken');
const { AppError } = require('../shared/errors/AppError');

function requireAuth(req, res, next) {
  const authHeader = req.headers.authorization;
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return next(new AppError('Authentication required', 401));
  }

  const token = authHeader.split(' ')[1];
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = { id: decoded.userId, role: decoded.role };
    next();
  } catch (err) {
    next(new AppError('Invalid or expired token', 401));
  }
}

function requireRole(...roles) {
  return (req, res, next) => {
    if (!req.user) {
      return next(new AppError('Authentication required', 401));
    }
    if (!roles.includes(req.user.role)) {
      return next(new AppError('Insufficient permissions', 403));
    }
    next();
  };
}

module.exports = { requireAuth, requireRole };
```

```js
// routes/admin.routes.js — router-level authorization
const express = require('express');
const router = express.Router();
const { requireAuth, requireRole } = require('../middleware/auth');

// Apply to ALL routes in this router
router.use(requireAuth);
router.use(requireRole('admin'));

router.get('/dashboard', (req, res) => res.json({ dashboard: 'data' }));
router.get('/metrics', (req, res) => res.json({ metrics: 'data' }));

module.exports = router;
```

| Component | Breakdown |
|-----------|-----------|
| `requireAuth` | Verifies the JWT and sets `req.user`. |
| `requireRole(...roles)` | Checks `req.user.role` against the allow-list. |
| `router.use(requireAuth)` | Applies authentication to all routes in the router. |
| `router.use(requireRole('admin'))` | Applies authorization to all routes in the router. |

**Rules:**
- Authentication middleware must run **before** authorization middleware.
- Router-level middleware (`router.use()`) can apply authentication and authorization to all routes in a router at once. 
- `req.user` must be set by authentication before authorization reads it.
- Public routes (login, registration) must be defined **before** the authentication middleware. 
- Authentication and authorization middleware should **never** contain business logic.

**Constraints and Limitations:**
- JWT tokens cannot be revoked without a blacklist or token versioning system.
- Session-based authentication requires server-side session storage.
- Authentication middleware should be mounted only on routes that require it.

### Annotated Code Example

```js
// app.js — public routes BEFORE authentication middleware
app.post('/auth/login', authController.login);
app.post('/auth/register', authController.register);

// Authentication middleware for all /api routes
app.use('/api', requireAuth);

// Protected routes
app.use('/api/users', userRouter);
app.use('/api/admin', adminRouter);  // adminRouter has requireRole('admin')
```

**Expected Output (for `GET /api/admin/dashboard` with a `user` role token):**
```json
{
  "error": "Insufficient permissions",
  "statusCode": 403
}
```

**Expected Output (for `POST /auth/login` without a token):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Why this output:** The login route is defined before the `requireAuth` middleware, so it is publicly accessible. The `/api/admin` router has `requireRole('admin')` applied at the router level, so any user without the admin role receives a 403 Forbidden. The authentication and authorization concerns are completely decoupled from the dashboard route logic. 

### Real-World Cases

- **API protection:** `requireAuth` on all `/api/*` routes except `/api/auth/*`. 
- **Admin panels:** `requireRole('admin')` on `/admin/*` routes.
- **Resource ownership:** `requireOwner` middleware checking `req.user.id === req.params.userId`.
- **Multi-tenant SaaS:** `requireTenantAccess` middleware checking the user's organisation against the requested resource.

---

## Core Concept 6: Error Handling

### Definitions

**Core Definition:** Error handling is the process of catching, formatting, and responding to errors that occur during request processing. This responsibility should be offloaded entirely to an independent global error-handling middleware rather than being scattered as try/catch blocks inside controllers.

**Technical Definition:** Express recognises error-handling middleware by its arity — it must have exactly four arguments: `(err, req, res, next)`. When an error is passed to `next(err)`, Express skips all remaining non-error middleware and invokes the error-handling middleware. The global error handler should be registered **last**, after all other middleware and routes. It should distinguish between operational errors (expected, with meaningful messages and status codes) and programming errors (unexpected, logged and reported generically). 

**Beginner-Friendly Explanation:** Error-handling middleware is like a safety net at the bottom of a trapeze act. If something goes wrong during the performance (the request), the safety net catches the performer (the error). It must be at the very bottom — if it's in the middle, the trapeze artists below it won't be caught. And the trapeze artists (controllers) shouldn't be building their own safety nets (try/catch blocks) — the global safety net handles everything.

### Purposes

- To offload exception formatting entirely to an independent global error interceptor rather than writing try/catch syntax inside controllers.
- To provide a centralised location for error formatting and logging.
- To send appropriate error responses to the client (e.g., 500, 404).
- To prevent unhandled errors from crashing the application.
- To distinguish between operational errors and programming errors. 

### Syntax Rules and Structure

```js
// middleware/errorHandler.js
function errorHandler(err, req, res, next) {
  // Log the error
  console.error({
    message: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method,
    timestamp: new Date().toISOString()
  });

  // Operational errors (expected)
  if (err.isOperational) {
    return res.status(err.statusCode || 500).json({
      success: false,
      error: {
        message: err.message,
        code: err.code
      }
    });
  }

  // Programming errors (unexpected)
  console.error('UNEXPECTED ERROR:', err);
  res.status(500).json({
    success: false,
    error: { message: 'Internal server error' }
  });
}

module.exports = errorHandler;
```

```js
// app.js — error handler MUST be last
app.use(express.json());
app.use('/api', routes);
app.use(errorHandler); // ← Last middleware
```

| Component | Breakdown |
|-----------|-----------|
| `(err, req, res, next)` | Four arguments — Express detects error middleware by arity. |
| `err.isOperational` | Flag distinguishing expected errors from bugs. |
| `err.statusCode` | HTTP status code attached to the error. |
| `app.use(errorHandler)` | Must be registered **after** all routes. |

**Rules:**
- Error-handling middleware **must** have exactly four arguments. 
- It must be registered **after** all other `app.use()` and route definitions. 
- Controllers should call `next(err)` or throw errors — not format error responses directly. 
- Async errors in Express 5 are automatically forwarded to error handlers. 
- In Express 4, async errors require a wrapper (`asyncHandler`) or try/catch with `next(err)`. 

**Constraints and Limitations:**
- Error handlers should not expose stack traces in production.
- Multiple error handlers can be defined for different error types.
- If an error handler calls `next(err)`, it passes to the next error handler.
- Controllers that contain try/catch blocks for error formatting violate SoC — the global handler should do this. 

### Annotated Code Example

```js
// ❌ BAD: try/catch in every controller
app.get('/api/users/:id', async (req, res) => {
  try {
    const user = await userService.findById(req.params.id);
    if (!user) return res.status(404).json({ error: 'User not found' });
    res.json({ data: user });
  } catch (error) {
    res.status(500).json({ error: 'Something went wrong' }); // Inconsistent formatting
  }
});

// ✅ GOOD: Controller throws; global handler formats
// controllers/user.controller.js
getById = async (req, res, next) => {
  try {
    const user = await this.userService.findById(req.params.id);
    if (!user) throw new AppError('User not found', 404);
    res.json({ data: user });
  } catch (error) {
    next(error); // Delegate to global handler
  }
};

// middleware/errorHandler.js
function errorHandler(err, req, res, next) {
  res.status(err.statusCode || 500).json({
    success: false,
    error: { message: err.message }
  });
}
```

**Expected Output (for `GET /api/users/999`):**
```json
{
  "success": false,
  "error": {
    "message": "User not found"
  }
}
```

**Why this output:** The controller calls `next(error)` instead of formatting the response itself. The global error handler — which is the single source of truth for error formatting — catches the error and produces a consistent response structure. The controller is free to focus solely on request/response orchestration. 

### Real-World Cases

- **API error formatting:** Returning consistent JSON error structures across all endpoints.
- **Database errors:** Catching connection failures or query errors and converting them to meaningful responses.
- **Validation errors:** Formatting 422 responses with field-level details.
- **404 handling:** A catch-all error handler for unmatched routes.
- **Process-level errors:** Handling `uncaughtException` and `unhandledRejection` for graceful shutdown. 

---

## References

- Separation of Concerns — SAP Help Portal — https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-us/ABENSEPARATION_CONCERNS_GUIDL.html
- How to Structure Express.js Applications for Scale — OneUptime Blog — https://github.com/OneUptime/blog/blob/master/posts/2026-01-26-expressjs-structure-scale/README.md
- Node.js Project Architecture Best Practices — LogRocket Blog — https://blog.logrocket.com/node-js-project-architecture-best-practices/
- MongoDB Backend Core Architecture Playbook — KabriAcid/stakeflow — https://github.com/KabriAcid/stakeflow/blob/main/BACKEND_MONGODB_CORE_ARCHITECTURE.md
- Express Service Layer — Separation of Concerns — https://raw.githubusercontent.com
- Validating Path and Query Parameters with Middleware — Steve Kinney — https://stevekinney.com/courses/full-stack-typescript/validating-query-and-path-parameters
- api-contract-guard — npm — https://www.npmjs.com/package/api-contract-guard
- Users API Updates (Role-Based Authorization) — CIS 526 Textbook — https://textbooks.cs.ksu.edu/cis526/x-examples/04-authentication/11-user-updates/
- How to handle errors globally in Node.js — CoreUI — https://coreui.io/answers/how-to-handle-errors-globally-in-nodejs/
- express-global-error — npm — https://www.npmjs.com/package/express-global-error
- Express.js Error Handling Guide — https://expressjs.com/en/guide/error-handling.html
- Node.js Backend Architecture Patterns for 2026 — safeguard.sh — https://safeguard.sh
- Express router and controllers in different files — Stack Overflow — https://stackoverflow.com/questions/30674227/
- Layer your app, keep Express within its boundaries — https://raw.githubusercontent.com
- Repository Pattern — Martin Fowler — https://martinfowler.com/eaaCatalog/repository.html
- Domain-Driven Design: Separation of Concerns — Eric Evans
- On the role of scientific thought — Edsger W. Dijkstra (1974)