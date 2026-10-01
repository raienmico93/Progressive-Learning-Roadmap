# Layered Architecture (N-Tier Architecture) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Layered architecture (also called N-Tier architecture) is a software architectural pattern that organises an application into a series of horizontal layers, where each layer has a specific responsibility and communicates only with the layer directly beneath it, creating a clean separation of concerns that enhances maintainability, testability, and scalability.

**Technical Definition:** The layered architecture pattern, also referred to as n-tier architecture, structures a software system into a series of horizontal layers. Each layer has a distinct role — routes map HTTP endpoints to controllers, controllers parse requests and structure responses, services encapsulate domain/business rules, repositories abstract data access behind interfaces, and the database provides persistence. In its strictest form, information flow is linear — a layer can only communicate with the layer immediately below it. The N in N-tier refers to the number of physical tiers, which may or may not correspond to the number of logical layers.

**Beginner-Friendly Explanation:** Imagine a restaurant. The **routes** are the front door — they decide where you go when you walk in. The **controllers** are the waiters — they take your order and bring you your food, but they don't cook. The **services** are the chefs — they know all the recipes (business rules) and decide how everything should be prepared. The **repositories** are the pantry managers — they know exactly where every ingredient is stored and how to retrieve it. The **database** is the pantry itself — the physical storage. Each person has one job, and they only talk to the person next to them in the chain. This is layered architecture.

### Key Characteristics

- **Horizontal separation:** Layers are organised horizontally, each addressing a single technical concern. 
- **Adjacent communication:** In strict layering, a layer communicates only with the layer directly below it. 
- **Abstract interfaces:** Layers communicate through well-defined interfaces, ensuring modularity and technological independence. 
- **Testability:** Each layer can be tested independently using mocks or stubs for adjacent layers. 
- **Replaceability:** Changing one layer (e.g., swapping a database) affects only the layers directly connected to it. 
- **Separation of concerns:** Each layer focuses solely on its role, making the system easier to understand and maintain. 

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript knowledge:** Functions, modules, and asynchronous programming.
- **Understanding of Express routing and middleware.**
- **Familiarity with a database or ORM** (e.g., Sequelize, Prisma, Mongoose).

### Related Programming Areas

- **MVC Architecture:** Layered architecture is a broader pattern; MVC is a specialised form focusing on the presentation layer. 
- **Repository Pattern:** The data access layer is typically implemented using the repository pattern. 
- **Service Layer Pattern:** The business logic layer is implemented using the service layer pattern. 
- **Dependency Injection:** Services and repositories often use DI to receive their dependencies. 
- **Domain-Driven Design:** Layered architecture is a foundational pattern that DDD builds upon. 
- **Microservices:** Each microservice can internally follow a layered architecture. 

### Core Concepts

1. **Routes** — mapping HTTP endpoints and HTTP methods to controller methods.
2. **Controllers** — parsing requests and structuring HTTP responses; strictly no business logic.
3. **Services** — the heart of the application; isolated domain rules, orchestration, and transactions.
4. **Repositories (Data Access Layer)** — abstracting raw database queries behind interface functions.
5. **Database** — the persistence engine (SQL, NoSQL, Cache).

---

## Core Concept 1: Routes

### Definitions

**Core Definition:** The routes layer is the entry point of the application — it maps incoming HTTP requests (defined by URL path and HTTP method) to the appropriate controller method.

**Technical Definition:** In Express, the routes layer uses `express.Router()` to define endpoints. Each route specifies an HTTP method, a path, and one or more middleware functions or controller methods. Routes are mounted on the main application using `app.use()`. The routes layer should be kept thin — its only responsibility is to define the HTTP surface and apply route-specific middleware (authentication, validation, rate limiting). All business logic is delegated to controllers and services.

**Beginner-Friendly Explanation:** Routes are like the directory in a building lobby. When you walk in (a request arrives), the directory tells you which floor and room to go to (which controller to invoke). The directory doesn't do any work itself — it just points you in the right direction.

### Purposes

- To define all HTTP endpoints and their corresponding HTTP methods.
- To apply route-specific middleware (authentication, validation, rate limiting).
- To route requests to the appropriate controller method.
- To serve as the single source of truth for the API's URL structure.

### Syntax Rules and Structure

```js
// routes/users.routes.js
const { Router } = require('express');
const { UserController } = require('../controllers/user.controller');
const { validateRequest } = require('../middleware/validate');
const { createUserSchema, updateUserSchema } = require('../validators/user.validator');

const router = Router();
const controller = new UserController();

router.get('/', controller.getAll);
router.get('/:id', controller.getById);
router.post('/', validateRequest(createUserSchema), controller.create);
router.put('/:id', validateRequest(updateUserSchema), controller.update);
router.delete('/:id', controller.delete);

module.exports = router;
```

```js
// app.js — mount routes
const express = require('express');
const app = express();
app.use('/api/users', require('./routes/users.routes'));
```

| Component | Breakdown |
|-----------|-----------|
| `Router()` | Creates a new router instance. |
| `router.get('/')` | Defines a GET endpoint at the router's root. |
| `controller.getAll` | The controller method to invoke. |
| `validateRequest(schema)` | Route-specific middleware for validation. |
| `app.use('/api/users', router)` | Mounts the router at a base path. |

#### Syntax Rules

- Routes should be defined in dedicated files (e.g., `routes/users.routes.js`).
- Each route file should export an `express.Router()` instance.
- Route-specific middleware should be applied at the route level, not globally.
- Routes should **not** contain any business logic — delegate everything to controllers.
- Route paths should be resource-based and RESTful.

#### Constraints and Limitations

- **Never put business logic in routes** — this leads to fat routes, which are worse than fat controllers.
- Routes are ordered — more specific routes must be defined before more general ones.
- Route-level middleware runs before the controller, so it can short-circuit the request.

### Annotated Code Example

```js
// routes/posts.routes.js
const express = require('express');
const router = express.Router();
const postController = require('../controllers/post.controller');
const { requireAuth } = require('../middleware/auth');
const { validate } = require('../middleware/validate');
const { createPostSchema } = require('../validators/post.validator');

// Public route — no authentication
router.get('/', postController.index);

// Public route with path parameter
router.get('/:id', postController.show);

// Protected route — authentication required
router.post('/', requireAuth, validate(createPostSchema), postController.create);

// Protected route with multiple middleware
router.put('/:id', requireAuth, postController.update);

// Protected route — only the author can delete
router.delete('/:id', requireAuth, postController.destroy);

module.exports = router;
```

**Expected Output (for `GET /api/posts`):**
```json
{
  "data": [
    { "_id": "1", "title": "MVC Guide", "author": { "name": "Alice" } }
  ]
}
```

**Why this output:** The route `router.get('/', postController.index)` maps the GET request to the `/api/posts` path to the `index` method of the post controller. The controller handles the response, not the route.

### Real-World Cases

- **REST APIs:** `/api/users`, `/api/products`, `/api/orders` — each resource has its own route file.
- **Versioned APIs:** `/api/v1/users`, `/api/v2/users` — separate routers for each version.
- **Public vs. private routes:** Public routes defined before authentication middleware; private routes after.

---

## Core Concept 2: Controllers

### Definitions

**Core Definition:** Controllers are the layer that parses HTTP requests (headers, query parameters, body) and structures HTTP responses. They act as intermediaries between the HTTP layer and the business logic layer, and they must contain strictly no business logic.

**Technical Definition:** The controller's job is to act as the ultimate middleman. It knows which questions it wants to ask the service layer, but lets the service layer do all the heavy lifting. Controllers handle HTTP request/response logic, extract and validate input, call services, and return responses. The controller should be thin — its primary responsibility is to translate between the HTTP protocol and the application's domain.

**Beginner-Friendly Explanation:** Controllers are like waiters in a restaurant. They take your order (the request), pass it to the kitchen (the service layer), and bring you your food (the response). Waiters don't cook the food — that's the chef's job. Their job is to communicate between the customer and the kitchen.

### Purposes

- To parse HTTP requests (headers, query parameters, body, route parameters).
- To validate input at the HTTP boundary before passing it to services.
- To call service methods and receive domain results.
- To structure HTTP responses (status codes, headers, response body).
- To handle HTTP-specific concerns (content negotiation, serialisation).

### Syntax Rules and Structure

```js
// controllers/user.controller.js
const { UserService } = require('../services/user.service');

class UserController {
  constructor() {
    this.userService = new UserService();
  }

  getAll = async (req, res, next) => {
    try {
      const users = await this.userService.findAll();
      res.json({ data: users });
    } catch (error) {
      next(error);
    }
  };

  getById = async (req, res, next) => {
    try {
      const { id } = req.params;
      const user = await this.userService.findById(id);
      if (!user) {
        return res.status(404).json({ error: 'User not found' });
      }
      res.json({ data: user });
    } catch (error) {
      next(error);
    }
  };

  create = async (req, res, next) => {
    try {
      const user = await this.userService.create(req.body);
      res.status(201).json({ data: user });
    } catch (error) {
      next(error);
    }
  };
}

module.exports = { UserController };
```

| Component | Breakdown |
|-----------|-----------|
| `req.params` | Route parameters (e.g., `:id`). |
| `req.body` | Parsed request body. |
| `this.userService` | The injected service layer. |
| `res.json()` | Sends a JSON response. |
| `next(error)` | Passes errors to the error-handling middleware. |

#### Syntax Rules

- Controllers should be defined in a dedicated `controllers/` directory.
- Each controller should handle one resource (e.g., `user.controller.js`).
- Controllers should **never** contain business logic — delegate to services.
- Keep controllers thin — extract complex logic to helper functions or the service layer.
- Controllers receive `(req, res, next)` just like middleware.
- Use `async/await` for asynchronous service calls.

#### Constraints and Limitations

- **Bloated controllers** are an anti-pattern — controllers stuffed with business logic are harder to test and maintain.
- **Direct database calls** from controllers are forbidden — always go through services and repositories.
- **Mixing infrastructure concerns** (emailing, caching) into controllers violates the separation of concerns.

### Annotated Code Example

```js
// controllers/post.controller.js
const { PostService } = require('../services/post.service');

class PostController {
  constructor() {
    this.postService = new PostService();
  }

  // GET /posts
  index = async (req, res, next) => {
    try {
      const { page, limit, sort } = req.query;
      const posts = await this.postService.findAll({ page, limit, sort });
      res.json({ data: posts });
    } catch (error) {
      next(error);
    }
  };

  // GET /posts/:id
  show = async (req, res, next) => {
    try {
      const post = await this.postService.findById(req.params.id);
      res.json({ data: post });
    } catch (error) {
      next(error);
    }
  };

  // POST /posts
  create = async (req, res, next) => {
    try {
      const post = await this.postService.create({
        ...req.body,
        authorId: req.user.id
      });
      res.status(201).json({ data: post });
    } catch (error) {
      next(error);
    }
  };

  // DELETE /posts/:id
  destroy = async (req, res, next) => {
    try {
      await this.postService.delete(req.params.id, req.user.id);
      res.sendStatus(204);
    } catch (error) {
      next(error);
    }
  };
}

module.exports = { PostController };
```

**Expected Output (for `GET /api/posts?page=1&limit=10`):**
```json
{
  "data": [
    { "_id": "1", "title": "MVC Guide", "author": { "name": "Alice" } }
  ]
}
```

**Why this output:** The controller extracts `page`, `limit`, and `sort` from `req.query`, passes them to `postService.findAll()`, and returns the result as JSON. The controller does not contain any business logic — it only orchestrates the request/response flow.

### Real-World Cases

- **REST APIs:** `userController`, `productController`, `orderController` — one per resource.
- **GraphQL resolvers:** Resolvers act as controllers, calling services and returning data.
- **Webhook handlers:** Controllers parse webhook payloads and delegate to services.

---

## Core Concept 3: Services

### Definitions

**Core Definition:** The services layer is the heart of the application. It contains all isolated domain and business rules, orchestrates multiple repositories, and handles transactional flows. Services are agnostic of the HTTP transport — they don't know what `req` or `res` are.

**Technical Definition:** The service layer contains the core business logic of the application. Services are responsible for data transformation, business rules, and orchestrating interactions between repositories and external services. A service layer is an architectural pattern that acts as an abstraction between route handlers (controllers) and the data access layer. Services encapsulate domain logic, enforce invariants, and coordinate multi-step operations that may span several repositories.

**Beginner-Friendly Explanation:** Services are the chefs in a restaurant. They know all the recipes (business rules), how to combine ingredients (orchestrate repositories), and how to ensure every dish meets quality standards (validation, transactions). The chef doesn't talk to customers directly — the waiter (controller) handles that. The chef only cares about cooking the perfect meal.

### Purposes

- To contain all isolated domain and business rules.
- To orchestrate multiple repositories for complex operations.
- To handle transactional flows (e.g., create an order and decrement inventory atomically).
- To enforce domain invariants and business validations.
- To transform data between the repository layer and the controller layer.
- To integrate with external services (payment gateways, email providers).

### Syntax Rules and Structure

```js
// services/user.service.js
const { UserRepository } = require('../repositories/user.repository');
const { hashPassword } = require('../utils/password');
const { AppError } = require('../utils/errors');

class UserService {
  constructor() {
    this.userRepository = new UserRepository();
  }

  async findAll() {
    return this.userRepository.findAll();
  }

  async findById(id) {
    return this.userRepository.findById(id);
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

  async update(id, data) {
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new AppError('User not found', 404);
    }
    return this.userRepository.update(id, data);
  }

  async delete(id) {
    return this.userRepository.delete(id);
  }
}

module.exports = { UserService };
```

| Component | Breakdown |
|-----------|-----------|
| `UserRepository` | The injected data access layer. |
| `hashPassword()` | A utility function for password hashing. |
| `AppError` | A custom error class for domain errors. |
| `findByEmail()` | A repository method for finding a user by email. |

#### Syntax Rules

- Services should be defined in a dedicated `services/` directory.
- Each service should handle one domain or resource (e.g., `user.service.js`).
- Services should **never** import `req` or `res` — they are transport-agnostic.
- Services should throw domain-specific errors (e.g., `AppError`) rather than HTTP errors.
- Services should orchestrate repositories, not directly access the database.
- Use dependency injection to provide repositories to services.

#### Constraints and Limitations

- Services that become too large (e.g., 1000+ lines) may need to be split into smaller, focused services.
- Services should not know about HTTP status codes — that's the controller's responsibility.
- Transaction management can be complex; use a unit of work or transaction manager for multi-repository operations.

### Annotated Code Example

```js
// services/order.service.js
const { OrderRepository } = require('../repositories/order.repository');
const { ProductRepository } = require('../repositories/product.repository');
const { PaymentService } = require('./payment.service');
const { AppError } = require('../utils/errors');

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
      await this.productRepository.decrementStock(
        item.productId,
        item.quantity
      );
    }

    return order;
  }
}

module.exports = { OrderService };
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

**Why this output:** The service orchestrates multiple operations: it validates products, charges payment, creates the order, and decrements inventory. All business rules are enforced in the service layer — the controller simply calls `orderService.createOrder(userId, items)` and returns the result.

### Real-World Cases

- **E-commerce:** `OrderService` orchestrating payment, inventory, and order creation.
- **Authentication:** `AuthService` handling password hashing, JWT generation, and token validation.
- **Content platforms:** `PostService` enforcing publication rules and sanitising content.
- **SaaS platforms:** `SubscriptionService` handling plan changes, billing, and feature access.

---

## Core Concept 4: Repositories (Data Access Layer)

### Definitions

**Core Definition:** The repositories layer (also called the Data Access Layer or DAL) abstracts raw database queries or ORM calls behind interface functions, allowing the database technology to change without breaking business logic.

**Technical Definition:** The repository pattern separates data access from business logic by wrapping ORM queries behind a typed interface. A repository is responsible for all database operations related to a specific entity — creating, reading, updating, and deleting records. It exposes a collection-like interface for accessing domain entities and keeps raw queries out of business services. The repository layer ensures that the service layer never directly interacts with the database, making the domain logic independent of the database implementation.

**Beginner-Friendly Explanation:** Repositories are like pantry managers in a restaurant. They know exactly where every ingredient is stored, how to retrieve it, and how to restock it. The chef (service) asks the pantry manager for ingredients, and the pantry manager gets them — the chef doesn't need to know whether the ingredients are in a refrigerator, a freezer, or a dry storage room. If the restaurant switches suppliers (changes databases), only the pantry manager needs to learn the new system.

### Purposes

- To abstract raw database queries or ORM calls behind interface functions.
- To allow the database technology to change without breaking business logic.
- To centralise all data access logic for a specific entity.
- To provide a collection-like interface for domain entities.
- To keep raw queries out of business services.

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
| `update(id, data)` | Updates an existing record. |
| `delete(id)` | Removes a record. |

#### Syntax Rules

- Repositories should be defined in a dedicated `repositories/` directory.
- Each repository should handle one entity (e.g., `user.repository.js`).
- Repositories should expose methods that match the domain's needs, not just generic CRUD.
- Repositories should return domain entities, not raw database rows.
- Services should **never** import models directly — they must go through repositories.

#### Constraints and Limitations

- Repositories should not contain business logic — that belongs in services.
- Complex queries that span multiple entities may require a repository method that returns a composite result.
- The repository interface should be stable; changing it affects all services that depend on it.

### Annotated Code Example

```js
// repositories/order.repository.js
const { Order, OrderItem } = require('../models');

class OrderRepository {
  async create(data) {
    return Order.create(data, {
      include: [{ model: OrderItem, as: 'items' }]
    });
  }

  async findById(id) {
    return Order.findByPk(id, {
      include: [{ model: OrderItem, as: 'items' }]
    });
  }

  async findByUserId(userId) {
    return Order.findAll({
      where: { userId },
      include: [{ model: OrderItem, as: 'items' }],
      order: [['createdAt', 'DESC']]
    });
  }

  async updateStatus(id, status) {
    await Order.update({ status }, { where: { id } });
    return this.findById(id);
  }
}

module.exports = { OrderRepository };
```

**Expected Output (for `findByUserId('user-1')`):**
```json
[
  {
    "id": "order-1",
    "userId": "user-1",
    "total": 2400,
    "status": "paid",
    "items": [{ "productId": "prod-1", "quantity": 2 }]
  }
]
```

**Why this output:** The repository method encapsulates the database query, including the `include` (join) for order items and the `order` (sort) clause. The service layer calls `orderRepository.findByUserId(userId)` without knowing the underlying SQL.

### Real-World Cases

- **E-commerce:** `OrderRepository`, `ProductRepository`, `UserRepository`.
- **Content platforms:** `PostRepository`, `CommentRepository`.
- **SaaS platforms:** `SubscriptionRepository`, `InvoiceRepository`.
- **Database migration:** Swapping from MongoDB to PostgreSQL requires only rewriting the repository layer — services remain unchanged. 

---

## Core Concept 5: Database

### Definitions

**Core Definition:** The database is the persistence engine — the physical storage layer where data is stored, retrieved, and managed. It can be a relational database (SQL), a NoSQL database, or a cache.

**Technical Definition:** The database layer is the lowest layer in the layered architecture. It is accessed exclusively through the repository layer — no other layer interacts with the database directly. The choice of database technology (PostgreSQL, MySQL, MongoDB, Redis) is an implementation detail that is abstracted behind the repository interface. In N-tier architecture, the database may be a separate physical tier (a dedicated database server) or a logical layer within the same process.

**Beginner-Friendly Explanation:** The database is like the pantry itself — the physical storage room where all ingredients are kept. The pantry manager (repository) knows how to get things from the pantry, but the chef (service) never goes into the pantry directly. The pantry can be reorganised (database migration) without the chef even noticing, as long as the pantry manager knows the new layout.

### Purposes

- To provide persistent storage for application data.
- To enforce data integrity through constraints, transactions, and indexes.
- To support efficient querying through indexes and query optimisation.
- To serve as the single source of truth for application state.

### Syntax Rules and Structure

```js
// config/database.js
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize(
  process.env.DB_NAME,
  process.env.DB_USER,
  process.env.DB_PASSWORD,
  {
    host: process.env.DB_HOST,
    dialect: 'postgres',
    logging: false,
    pool: { max: 5, min: 0, idle: 10000 }
  }
);

module.exports = sequelize;
```

| Component | Breakdown |
|-----------|-----------|
| `Sequelize` | The ORM instance. |
| `dialect` | The database type (`postgres`, `mysql`, etc.). |
| `pool` | Connection pool configuration. |

#### Syntax Rules

- The database should be configured in a dedicated `config/` file.
- Credentials should be stored in environment variables, never in source code.
- The database should only be accessed through the repository layer.
- Use connection pooling to manage database connections efficiently.
- Migrations should be version-controlled and applied consistently across environments.

#### Constraints and Limitations

- Database performance is a critical bottleneck — indexes must be designed carefully.
- ORMs add overhead and can generate inefficient queries — monitor and optimise.
- Database changes (schema migrations) must be handled carefully in production.

### Annotated Code Example

```js
// repositories/product.repository.js
const { Product } = require('../models');

class ProductRepository {
  async findWithFilters({ category, minPrice, maxPrice }) {
    const where = {};
    if (category) where.category = category;
    if (minPrice) where.price = { ...where.price, [Op.gte]: minPrice };
    if (maxPrice) where.price = { ...where.price, [Op.lte]: maxPrice };

    return Product.findAll({ where });
  }

  async decrementStock(productId, quantity) {
    return Product.decrement('stock', {
      by: quantity,
      where: { id: productId, stock: { [Op.gte]: quantity } }
    });
  }
}

module.exports = { ProductRepository };
```

**Expected Output (for `findWithFilters({ category: 'electronics', minPrice: 100 })`):**
```json
[
  { "id": "prod-1", "name": "Laptop", "price": 1200, "stock": 10, "category": "electronics" }
]
```

**Why this output:** The repository method builds a dynamic `WHERE` clause based on the provided filters, using Sequelize's `Op` operators. The service layer calls this method without knowing the underlying SQL. The `decrementStock` method uses `Product.decrement()` with a `WHERE` clause that ensures stock doesn't go below zero.

### Real-World Cases

- **E-commerce:** PostgreSQL for transactional order data, Redis for session caching.
- **Content platforms:** MongoDB for flexible document storage.
- **Analytics:** ClickHouse or TimescaleDB for time-series data.
- **Real-time features:** Redis for pub/sub and caching.

---

## References

- Layered Architecture Pattern — https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/ch01.html
- Layered Architecture (N-Tier) — Microsoft Azure Architecture Center — https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/n-tier
- N-Tier Architecture — Software Engineering for the World Wide Web (GMU) — https://cs.gmu.edu/~aabduraz/classes/642/slides/642Lec02a-NTier.pdf
- Backend Development Guidelines (Layered Architecture) — https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/bbeierle12/skill-mcp-claude/backend-dev-guidelines/SKILL.md
- Architecting Backend Applications in JavaScript — https://medium.com/@hrishabhbharati/architecting-backend-applications-in-javascript-a-practical-guide-for-beginner-to-intermediate-cd1998580a2a
- Service Layer Pattern — https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/bbeierle12/skill-mcp-claude/backend-dev-guidelines/SKILL.md
- Repository Pattern with MongoDB — https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/bbeierle12/skill-mcp-claude/backend-dev-guidelines/SKILL.md
- Layered Architecture — Baeldung — https://www.baeldung.com/cs/layered-architecture
- Node.js Project Architecture Best Practices — LogRocket — https://blog.logrocket.com/node-js-project-architecture-best-practices/
- Express.js Routing Guide — https://expressjs.com/en/guide/routing.html
- Express.js Using Middleware — https://expressjs.com/en/guide/using-middleware.html
- Sequelize Documentation — https://sequelize.org/docs/v6/
- Prisma Documentation — https://www.prisma.io/docs
- Mongoose Documentation — https://mongoosejs.com/docs/