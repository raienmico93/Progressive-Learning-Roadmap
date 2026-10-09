# Integration Testing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Integration testing is the practice of verifying that multiple components of an application — routes, middleware, controllers, database layers, and external services — work together correctly as a cohesive system, rather than in isolation as unit tests do.

**Technical Definition:** Integration testing exercises the full HTTP request–response lifecycle through an Express application: the request enters through a route, passes through middleware (authentication, validation, rate limiting), reaches a controller, interacts with a database or external service, and returns a response through global error-handling middleware. In Node.js, Supertest drives the Express `app` object directly in-process without binding to a network port, eliminating network flakiness and enabling fast, deterministic assertions on status codes, headers, and response bodies. Jest or Mocha provides the test runner and assertion framework.

**Beginner-Friendly Explanation:** Unit tests check one small piece of code at a time. Integration tests check that all the pieces fit together — that a request actually reaches your route, that your authentication middleware rejects bad tokens, that your database gets updated, and that errors are formatted correctly. You send real HTTP requests to your app (without starting a real server) and check the responses.

### Key Characteristics

- **Full request lifecycle:** Each test exercises routing, middleware, controllers, and data layers together.
- **Separate app from server:** Export the Express `app` without calling `app.listen()` so Supertest can bind it to an ephemeral port.
- **In-process execution:** Supertest drives the app directly — no real network calls, no port conflicts.
- **Test database isolation:** Use a separate test database (in-memory MongoDB, SQLite, or a Docker container) to avoid polluting development data.
- **Cleanup between tests:** Clear database collections, reset mocks, and close connections after each test or suite.
- **Assertion-rich:** Verify status codes, response bodies, headers, database state, and side effects.

### Prerequisites

- **Node.js runtime** (v18 or higher).
- **Express.js installed:** `npm install express`.
- **Jest and Supertest installed:** `npm install --save-dev jest supertest @types/jest @types/supertest`.
- **A test database** (MongoDB Memory Server, SQLite in-memory, or a Docker Postgres).
- **Supertest** for HTTP assertions; **Jest** (or Mocha) for test organisation.

### Related Programming Areas

- **Unit Testing:** Isolated tests of individual functions and services.
- **End-to-End Testing:** Full-stack tests including the browser or client.
- **API Contract Testing:** Verifying the API matches its OpenAPI specification.
- **Test Databases:** In-memory or containerised databases for isolated integration tests.
- **CI/CD Pipelines:** Integration tests run on every commit to catch regressions.

### Core Concepts

1. **Testing Routes** — verifying that endpoints return correct status codes and bodies.
2. **Testing Middleware** — verifying that middleware processes requests correctly.
3. **Testing Databases** — verifying that routes correctly interact with the data layer.
4. **Testing Authentication Flows** — login, token issuance, and protected route access.
5. **Testing Authorization and RBAC** — role-based access control enforcement.
6. **Testing Global Error-Handling Middleware** — ensuring uncaught errors format correctly.
7. **Testing Rate Limiting and CORS** — verifying security middleware configurations.
8. **Testing File Uploads/Downloads** — multipart/form-data handling and binary responses.

---

## Core Concept 1: Testing Routes

### Definitions

**Core Definition:** Route integration testing verifies that HTTP endpoints respond correctly to requests — including status codes, response bodies, headers, and database side effects — when exercised through the full Express middleware stack.

**Technical Definition:** Supertest creates a test agent from the Express `app` object and sends HTTP requests using fluent methods (`.get()`, `.post()`, `.put()`, `.patch()`, `.delete()`). Assertions are chained (`.expect(200)`, `.expect('Content-Type', /json/)`) or made on the `response` object (`response.status`, `response.body`, `response.headers`). Because the app is not listening on a port, Supertest binds it to an ephemeral port internally, eliminating port conflicts.

**Beginner-Friendly Explanation:** You send a request to your endpoint — `GET /api/users` — and check that you get back a 200 status and an array of users. Supertest handles all the HTTP details so you can focus on what the response should look like.

### Purposes

- To verify that routes return the correct status codes and response bodies.
- To verify that query parameters, path parameters, and request bodies are handled correctly.
- To verify that database operations triggered by the route actually persist data.
- To catch regressions in route behaviour before deployment.

### Syntax Rules and Structure

```js
const request = require('supertest');
const app = require('../app');  // Export app WITHOUT app.listen()

// GET request
const response = await request(app).get('/api/users').expect(200);

// POST request with body
const response = await request(app)
  .post('/api/users')
  .send({ name: 'Alice', email: 'alice@example.com' })
  .set('Accept', 'application/json')
  .expect(201);

// Assertions
expect(response.body).toHaveProperty('id');
expect(response.body.email).toBe('alice@example.com');
```

| Method | Purpose |
|--------|---------|
| `.get(path)` | Send a GET request. |
| `.post(path).send(body)` | Send a POST request with JSON body. |
| `.put(path).send(body)` | Send a PUT request. |
| `.patch(path).send(body)` | Send a PATCH request. |
| `.delete(path)` | Send a DELETE request. |
| `.query({ key: value })` | Add query parameters. |
| `.set(header, value)` | Set a request header. |
| `.expect(status)` | Assert the response status. |
| `.expect(header, value)` | Assert a response header. |

**Rules:**
- Export the Express `app` without calling `app.listen()` — Supertest binds it to an ephemeral port.
- Use `async/await` with Supertest — all request methods return Promises.
- Chain `.expect()` calls for status code and header assertions.
- Assert on `response.body` for JSON payloads and `response.text` for text responses.

### Annotated Code Example

```js
// test/integration/users.test.js
const request = require('supertest');
const app = require('../../app');
const User = require('../../models/User');

describe('User API Endpoints', () => {
  beforeEach(async () => {
    await User.deleteMany({});  // Clean database before each test
  });

  it('should return an empty list when no users exist', async () => {
    const response = await request(app)
      .get('/api/users')
      .expect('Content-Type', /json/)
      .expect(200);

    expect(response.body).toEqual([]);
  });

  it('should create a new user and return 201', async () => {
    const newUser = {
      name: 'Alice',
      email: 'alice@example.com',
      password: 'securepassword123'
    };

    const response = await request(app)
      .post('/api/users')
      .send(newUser)
      .set('Accept', 'application/json')
      .expect(201);

    expect(response.body).toHaveProperty('id');
    expect(response.body.name).toBe(newUser.name);
    expect(response.body.email).toBe(newUser.email);
    expect(response.body.password).toBeUndefined();  // Never expose password

    // Verify it actually reached the database
    const userInDb = await User.findOne({ email: newUser.email });
    expect(userInDb).not.toBeNull();
    expect(userInDb.name).toBe(newUser.name);
  });

  it('should return 400 for invalid email', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ name: 'Test', email: 'not-an-email', password: 'pass123' })
      .expect(400);

    expect(response.body.error).toContain('email');
  });

  it('should retrieve a user by ID', async () => {
    const user = await User.create({
      name: 'Bob',
      email: 'bob@example.com',
      password: 'hashedpassword'
    });

    const response = await request(app)
      .get(`/api/users/${user.id}`)
      .expect(200);

    expect(response.body.name).toBe('Bob');
    expect(response.body.email).toBe('bob@example.com');
  });

  it('should return 404 for non-existent user', async () => {
    const response = await request(app)
      .get('/api/users/507f1f77bcf86cd799439011')
      .expect(404);

    expect(response.body.error).toMatch(/not found/i);
  });
});
```

**Expected Output (Jest console):**
```
PASS  test/integration/users.test.js
  User API Endpoints
    ✓ should return an empty list when no users exist (45 ms)
    ✓ should create a new user and return 201 (62 ms)
    ✓ should return 400 for invalid email (38 ms)
    ✓ should retrieve a user by ID (41 ms)
    ✓ should return 404 for non-existent user (35 ms)

Tests: 5 passed, 5 total
```

**Why this output:** Each test sends a real HTTP request through the full Express middleware stack. The POST test verifies the database persistence by querying directly after the request. The invalid email test verifies validation middleware rejects bad input before the controller runs. The password is never returned in the response.

### Real-World Cases

- **REST APIs:** Verifying CRUD endpoints return correct status codes and bodies.
- **E-commerce:** Testing product listing, creation, and retrieval endpoints.
- **SaaS platforms:** Testing user management, billing, and subscription routes.

---

## Core Concept 2: Testing Middleware

### Definitions

**Core Definition:** Middleware integration testing verifies that middleware functions correctly process requests — authenticating users, validating input, logging, and passing control to subsequent handlers — when exercised through the full Express pipeline.

**Technical Definition:** Supertest sends requests through the Express app, which executes middleware in registration order. Assertions verify that middleware either passes control (via `next()`) or short-circuits the request with a specific response. Middleware can be tested in isolation using `middleware-supertest` (which exposes the middleware as a testable endpoint) or in context by asserting on the final response.

**Beginner-Friendly Explanation:** You test that your authentication middleware rejects requests without a token, that your logging middleware records the request, and that your validation middleware returns 400 for bad data — all by sending requests through your app and checking the responses.

### Purposes

- To verify that middleware functions execute in the correct order.
- To verify that authentication middleware rejects unauthorised requests.
- To verify that validation middleware returns appropriate error responses.
- To verify that logging middleware records request information.
- To catch middleware bugs that would only surface in production.

### Syntax Rules and Structure

**Testing middleware in context:**
```js
it('rejects requests without authentication token', async () => {
  const response = await request(app)
    .get('/api/protected')
    .expect(401);

  expect(response.body.error).toBe('Unauthorized');
});
```

**Testing middleware in isolation (middleware-supertest):**
```js
const { mwsupertest } = require('middleware-supertest');

it('should set req.user when valid token provided', async () => {
  const result = await mwsupertest(authMiddleware)
    .request()
    .set('Authorization', 'Bearer valid-token')
    .expect(200);

  expect(result.req.user).toBeDefined();
});
```

**Rules:**
- Test middleware in context first — it's the most realistic scenario.
- Use `middleware-supertest` for isolated middleware testing when the middleware is complex and needs focused verification.
- Assert on both the response (status, body) and any request mutations the middleware performs.
- Test middleware ordering by verifying that earlier middleware short-circuits before later middleware runs.

### Annotated Code Example

```js
// middleware/auth.js
function authMiddleware(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  try {
    req.user = { id: 1, role: 'user', token };
    next();
  } catch (err) {
    res.status(401).json({ error: 'Invalid token' });
  }
}

module.exports = { authMiddleware };
```

```js
// test/integration/middleware.test.js
const request = require('supertest');
const express = require('express');
const { authMiddleware } = require('../../middleware/auth');

// Create a minimal test app with the middleware
const testApp = express();
testApp.use(express.json());
testApp.get('/protected', authMiddleware, (req, res) => {
  res.json({ user: req.user });
});

describe('Auth Middleware', () => {
  it('should reject request without Authorization header', async () => {
    const response = await request(testApp)
      .get('/protected')
      .expect(401);

    expect(response.body.error).toBe('Unauthorized');
  });

  it('should reject request with malformed Authorization header', async () => {
    const response = await request(testApp)
      .get('/protected')
      .set('Authorization', 'InvalidFormat')
      .expect(401);

    expect(response.body.error).toBe('Unauthorized');
  });

  it('should pass through with valid token and set req.user', async () => {
    const response = await request(testApp)
      .get('/protected')
      .set('Authorization', 'Bearer valid-token-xyz')
      .expect(200);

    expect(response.body.user).toBeDefined();
    expect(response.body.user.id).toBe(1);
    expect(response.body.user.token).toBe('valid-token-xyz');
  });
});
```

**Expected Output:**
```
PASS  test/integration/middleware.test.js
  Auth Middleware
    ✓ should reject request without Authorization header (12 ms)
    ✓ should reject request with malformed Authorization header (8 ms)
    ✓ should pass through with valid token and set req.user (10 ms)
```

**Why this output:** A minimal test app isolates the middleware from the rest of the application. Each test sends a request and asserts on the response. The middleware correctly rejects missing and malformed tokens, and passes through valid ones while populating `req.user`. The third test verifies both the response and the request mutation (`req.user`).

### Real-World Cases

- **Authentication middleware:** Verifying JWT validation and user injection.
- **Validation middleware:** Verifying that invalid input is rejected with 400.
- **Logging middleware:** Verifying that request details are recorded.
- **CORS middleware:** Verifying that correct headers are set.

---

## Core Concept 3: Testing Databases

### Definitions

**Core Definition:** Database integration testing verifies that routes correctly interact with the database — creating, reading, updating, and deleting records — using a real (test) database rather than a mock.

**Technical Definition:** Database integration tests use a dedicated test database (MongoDB Memory Server for MongoDB, SQLite in-memory for relational databases, or a Docker container) to exercise real ORM or query-builder operations. Setup hooks (`beforeAll`, `afterAll`) establish and tear down the connection, while `beforeEach` or `afterEach` clears collections to ensure test isolation. Assertions verify both the HTTP response and the database state.

**Beginner-Friendly Explanation:** When you test a route that creates a user, you want to verify not just that the API returns 201, but that the user is actually saved in the database. You use a special test database that gets wiped clean between tests, so each test starts with a blank slate.

### Purposes

- To verify that routes correctly persist, retrieve, update, and delete data.
- To catch ORM query bugs that unit tests with mocked repositories would miss.
- To verify database constraints (unique indexes, foreign keys) are enforced.
- To test transaction rollback behaviour and error handling.

### Syntax Rules and Structure

**MongoDB Memory Server setup:**
```js
const mongoose = require('mongoose');
const { MongoMemoryServer } = require('mongodb-memory-server');

let mongoServer;

beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create();
  await mongoose.connect(mongoServer.getUri());
});

afterEach(async () => {
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});
```

| Hook | Purpose |
|------|---------|
| `beforeAll` | Start database server and connect. |
| `afterEach` | Clear collections between tests. |
| `afterAll` | Disconnect and stop database server. |

**Rules:**
- Always use a separate test database — never run integration tests against development or production data.
- Clear collections between tests to ensure isolation.
- Assert on both the HTTP response and the database state.
- Use `MongoMemoryServer` for MongoDB, or a Docker container for Postgres/MySQL.

### Annotated Code Example

```js
// test/integration/database.test.js
const mongoose = require('mongoose');
const { MongoMemoryServer } = require('mongodb-memory-server');
const request = require('supertest');
const app = require('../../app');
const Product = require('../../models/Product');

let mongoServer;

beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create();
  await mongoose.connect(mongoServer.getUri());
});

afterEach(async () => {
  await Product.deleteMany({});
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});

describe('Product Database Integration', () => {
  it('should create a product and verify it in the database', async () => {
    const productData = {
      name: 'Laptop',
      price: 1200,
      category: 'electronics'
    };

    const response = await request(app)
      .post('/api/products')
      .send(productData)
      .expect(201);

    // Verify HTTP response
    expect(response.body.name).toBe('Laptop');
    expect(response.body.price).toBe(1200);

    // Verify database persistence
    const productInDb = await Product.findById(response.body.id);
    expect(productInDb).not.toBeNull();
    expect(productInDb.name).toBe('Laptop');
    expect(productInDb.price).toBe(1200);
    expect(productInDb.category).toBe('electronics');
  });

  it('should reject duplicate product names', async () => {
    await Product.create({ name: 'Laptop', price: 1200 });

    const response = await request(app)
      .post('/api/products')
      .send({ name: 'Laptop', price: 1500 })
      .expect(409);

    expect(response.body.error).toMatch(/already exists/i);

    // Verify only one product exists
    const count = await Product.countDocuments({ name: 'Laptop' });
    expect(count).toBe(1);
  });

  it('should update a product and verify the change persists', async () => {
    const product = await Product.create({
      name: 'Phone',
      price: 800
    });

    await request(app)
      .patch(`/api/products/${product.id}`)
      .send({ price: 750 })
      .expect(200);

    // Verify database was updated
    const updated = await Product.findById(product.id);
    expect(updated.price).toBe(750);
  });

  it('should delete a product and verify it is removed', async () => {
    const product = await Product.create({ name: 'Tablet', price: 600 });

    await request(app)
      .delete(`/api/products/${product.id}`)
      .expect(204);

    // Verify database deletion
    const deleted = await Product.findById(product.id);
    expect(deleted).toBeNull();
  });
});
```

**Expected Output:**
```
PASS  test/integration/database.test.js
  Product Database Integration
    ✓ should create a product and verify it in the database (78 ms)
    ✓ should reject duplicate product names (52 ms)
    ✓ should update a product and verify the change persists (45 ms)
    ✓ should delete a product and verify it is removed (41 ms)

Tests: 4 passed, 4 total
```

**Why this output:** Each test exercises a real database operation through the HTTP API, then verifies the database state directly. The duplicate name test verifies that the unique index constraint is enforced. The update and delete tests verify that changes are persisted. `MongoMemoryServer` provides a real MongoDB instance that is automatically cleaned up.

### Real-World Cases

- **E-commerce:** Verifying product inventory updates after orders.
- **SaaS:** Verifying user subscription changes persist correctly.
- **Financial systems:** Verifying transaction records are created and retrievable.

---

## Core Concept 4: Testing Authentication Flows

### Definitions

**Core Definition:** Authentication flow integration testing verifies the complete login lifecycle: a user submits credentials, the server validates them, issues a token (JWT or session), and the token grants access to protected routes.

**Technical Definition:** Integration tests for authentication typically follow a three-step flow: (1) POST valid credentials to the login endpoint and assert that a token is returned; (2) use the token in the `Authorization: Bearer <token>` header to access a protected route; (3) assert that the protected route returns the authenticated user's data. Additional tests verify that invalid credentials are rejected and that expired or malformed tokens produce 401 responses.

**Beginner-Friendly Explanation:** You test the complete login process: log in with a username and password, get back a token, then use that token to access a page that only logged-in users can see. You also test what happens when someone uses a wrong password or a fake token.

### Purposes

- To verify that valid credentials produce a valid token.
- To verify that invalid credentials are rejected with 401.
- To verify that protected routes reject requests without a token.
- To verify that protected routes accept requests with a valid token.
- To verify that expired or malformed tokens are rejected.

### Syntax Rules and Structure

```js
// Step 1: Login and capture token
const loginResponse = await request(app)
  .post('/api/auth/login')
  .send({ email: 'alice@example.com', password: 'password123' })
  .expect(200);

const { token } = loginResponse.body;

// Step 2: Use token to access protected route
const profileResponse = await request(app)
  .get('/api/users/me')
  .set('Authorization', `Bearer ${token}`)
  .expect(200);

expect(profileResponse.body.email).toBe('alice@example.com');
```

**Rules:**
- Store the token in a variable and reuse it across subsequent requests within the same test.
- Test both the success path (valid credentials) and failure paths (invalid credentials, missing token, expired token).
- Verify that the token is actually a valid JWT (check structure or decode it).
- Test that the protected route returns the correct user's data — not just any authenticated user.

### Annotated Code Example

```js
// test/integration/auth.test.js
const request = require('supertest');
const app = require('../../app');
const User = require('../../models/User');

describe('Authentication Flow', () => {
  let testUser;
  let validToken;

  beforeAll(async () => {
    // Create a test user
    testUser = await User.create({
      name: 'Alice',
      email: 'alice@example.com',
      password: 'hashedpassword123',  // Already hashed by model hook
      role: 'user'
    });
  });

  describe('POST /api/auth/login', () => {
    it('should return a token with valid credentials', async () => {
      const response = await request(app)
        .post('/api/auth/login')
        .send({ email: 'alice@example.com', password: 'password123' })
        .expect(200);

      expect(response.body).toHaveProperty('token');
      expect(typeof response.body.token).toBe('string');
      expect(response.body.token.split('.')).toHaveLength(3);  // JWT structure
      validToken = response.body.token;
    });

    it('should return 401 with invalid password', async () => {
      const response = await request(app)
        .post('/api/auth/login')
        .send({ email: 'alice@example.com', password: 'wrongpassword' })
        .expect(401);

      expect(response.body.error).toMatch(/invalid credentials/i);
    });

    it('should return 401 with non-existent email', async () => {
      const response = await request(app)
        .post('/api/auth/login')
        .send({ email: 'nobody@example.com', password: 'password123' })
        .expect(401);

      expect(response.body.error).toMatch(/invalid credentials/i);
    });
  });

  describe('GET /api/users/me (protected route)', () => {
    it('should return current user with valid token', async () => {
      const response = await request(app)
        .get('/api/users/me')
        .set('Authorization', `Bearer ${validToken}`)
        .expect(200);

      expect(response.body.email).toBe('alice@example.com');
      expect(response.body.name).toBe('Alice');
    });

    it('should return 401 without token', async () => {
      const response = await request(app)
        .get('/api/users/me')
        .expect(401);

      expect(response.body.error).toMatch(/unauthorized/i);
    });

    it('should return 401 with malformed token', async () => {
      const response = await request(app)
        .get('/api/users/me')
        .set('Authorization', 'Bearer not-a-real-token')
        .expect(401);

      expect(response.body.error).toMatch(/unauthorized|invalid/i);
    });

    it('should return 401 with expired token', async () => {
      // Create a token with past expiry
      const jwt = require('jsonwebtoken');
      const expiredToken = jwt.sign(
        { id: testUser.id, role: 'user' },
        process.env.JWT_SECRET,
        { expiresIn: '-1h' }  // Expired 1 hour ago
      );

      const response = await request(app)
        .get('/api/users/me')
        .set('Authorization', `Bearer ${expiredToken}`)
        .expect(401);

      expect(response.body.error).toMatch(/unauthorized|expired/i);
    });
  });
});
```

**Expected Output:**
```
PASS  test/integration/auth.test.js
  Authentication Flow
    POST /api/auth/login
      ✓ should return a token with valid credentials (52 ms)
      ✓ should return 401 with invalid password (41 ms)
      ✓ should return 401 with non-existent email (38 ms)
    GET /api/users/me (protected route)
      ✓ should return current user with valid token (45 ms)
      ✓ should return 401 without token (32 ms)
      ✓ should return 401 with malformed token (35 ms)
      ✓ should return 401 with expired token (40 ms)

Tests: 7 passed, 7 total
```

**Why this output:** The test suite covers the complete authentication lifecycle. The login test verifies JWT structure (three dot-separated parts). The protected route tests verify that valid tokens grant access, while missing, malformed, and expired tokens all produce 401 responses. The expired token test manually creates a token with a past expiry using `jsonwebtoken`.

### Real-World Cases

- **SaaS platforms:** Testing login, token refresh, and logout flows.
- **E-commerce:** Testing customer authentication and order history access.
- **Banking apps:** Testing secure login and session management.

---

## Core Concept 5: Testing Authorization and RBAC

### Definitions

**Core Definition:** Authorization integration testing verifies that role-based access control (RBAC) is correctly enforced — users with insufficient roles receive 403 Forbidden, while users with the required role can access the resource.

**Technical Definition:** RBAC middleware checks `req.user.role` (set by authentication middleware) against a required role or permission. Integration tests create users with different roles, generate tokens for each, and assert that the API returns 200 for authorised roles and 403 for unauthorised roles. Testing all role-route combinations can be expensive; a common pattern is to deconstruct the middleware and test the role-checking logic directly, then verify integration for critical routes.

**Beginner-Friendly Explanation:** You test that an admin can access the admin dashboard, but a regular user cannot. You create a token for each role and send requests to the protected route, checking that the right people get in and the wrong people get a 403 error.

### Purposes

- To verify that admin-only routes reject regular users with 403.
- To verify that user routes reject anonymous requests with 401.
- To verify that role checks are enforced consistently across all protected routes.
- To verify that the correct role hierarchy is respected (e.g., admin > user).

### Syntax Rules and Structure

```js
// Create tokens for different roles
const adminToken = generateToken({ id: 1, role: 'admin' });
const userToken = generateToken({ id: 2, role: 'user' });

// Admin route — admin succeeds
await request(app)
  .get('/api/admin/users')
  .set('Authorization', `Bearer ${adminToken}`)
  .expect(200);

// Admin route — user gets 403
await request(app)
  .get('/api/admin/users')
  .set('Authorization', `Bearer ${userToken}`)
  .expect(403);
```

**Rules:**
- Create test users with different roles in `beforeAll` or `beforeEach`.
- Generate valid tokens for each role using the same JWT secret the app uses.
- Test every role-route combination for critical endpoints (admin, user, anonymous).
- Verify that the 403 response includes an appropriate error message, not a 401 (which would indicate authentication failure rather than authorization failure).
- For large route sets, deconstruct the middleware and test role-checking logic directly to avoid combinatorial explosion.

### Annotated Code Example

```js
// middleware/rbac.js
function requireRole(...allowedRoles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({
        error: 'Forbidden',
        message: `Role '${req.user.role}' is not authorised`
      });
    }
    next();
  };
}

module.exports = { requireRole };
```

```js
// test/integration/rbac.test.js
const request = require('supertest');
const jwt = require('jsonwebtoken');
const express = require('express');
const { requireRole } = require('../../middleware/rbac');

const app = express();
app.use(express.json());

// Simulate auth middleware setting req.user from token
app.use((req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (token) {
    try {
      req.user = jwt.verify(token, process.env.JWT_SECRET);
    } catch (e) { /* ignore */ }
  }
  next();
});

// Protected routes with different role requirements
app.get('/admin/dashboard', requireRole('admin'), (req, res) => {
  res.json({ message: 'Admin dashboard' });
});

app.get('/user/profile', requireRole('admin', 'user'), (req, res) => {
  res.json({ message: 'User profile' });
});

describe('RBAC Middleware', () => {
  const adminToken = jwt.sign({ id: 1, role: 'admin' }, process.env.JWT_SECRET);
  const userToken = jwt.sign({ id: 2, role: 'user' }, process.env.JWT_SECRET);
  const viewerToken = jwt.sign({ id: 3, role: 'viewer' }, process.env.JWT_SECRET);

  describe('GET /admin/dashboard', () => {
    it('should allow admin', async () => {
      await request(app)
        .get('/admin/dashboard')
        .set('Authorization', `Bearer ${adminToken}`)
        .expect(200);
    });

    it('should reject user with 403', async () => {
      const response = await request(app)
        .get('/admin/dashboard')
        .set('Authorization', `Bearer ${userToken}`)
        .expect(403);

      expect(response.body.error).toBe('Forbidden');
      expect(response.body.message).toContain('user');
    });

    it('should reject viewer with 403', async () => {
      await request(app)
        .get('/admin/dashboard')
        .set('Authorization', `Bearer ${viewerToken}`)
        .expect(403);
    });

    it('should reject anonymous with 401', async () => {
      await request(app)
        .get('/admin/dashboard')
        .expect(401);
    });
  });

  describe('GET /user/profile', () => {
    it('should allow admin', async () => {
      await request(app)
        .get('/user/profile')
        .set('Authorization', `Bearer ${adminToken}`)
        .expect(200);
    });

    it('should allow user', async () => {
      await request(app)
        .get('/user/profile')
        .set('Authorization', `Bearer ${userToken}`)
        .expect(200);
    });

    it('should reject viewer with 403', async () => {
      await request(app)
        .get('/user/profile')
        .set('Authorization', `Bearer ${viewerToken}`)
        .expect(403);
    });
  });
});
```

**Expected Output:**
```
PASS  test/integration/rbac.test.js
  RBAC Middleware
    GET /admin/dashboard
      ✓ should allow admin (8 ms)
      ✓ should reject user with 403 (6 ms)
      ✓ should reject viewer with 403 (5 ms)
      ✓ should reject anonymous with 401 (4 ms)
    GET /user/profile
      ✓ should allow admin (5 ms)
      ✓ should allow user (5 ms)
      ✓ should reject viewer with 403 (4 ms)

Tests: 7 passed, 7 total
```

**Why this output:** Each test generates a token with a specific role and sends a request to the protected route. The middleware checks `req.user.role` against the allowed roles. Admin succeeds on both routes; user succeeds only on the user route; viewer is rejected from both; anonymous receives 401. The 403 response includes the role that was rejected, making debugging easier.

### Real-World Cases

- **Admin panels:** Verifying that only admins can access user management.
- **Multi-tenant SaaS:** Verifying that users can only access their own tenant's data.
- **Healthcare:** Verifying that only authorised clinicians can access patient records.

---

## Core Concept 6: Testing Global Error-Handling Middleware

### Definitions

**Core Definition:** Global error-handling middleware testing verifies that uncaught errors — thrown in routes or passed via `next(err)` — are caught by the Express error handler and formatted into a consistent, user-safe response.

**Technical Definition:** Express error-handling middleware has four arguments: `(err, req, res, next)`. It must be registered last, after all routes. Integration tests trigger errors by sending requests that cause route handlers to throw or call `next(err)`, then assert that the error handler returns the correct status code, error format (e.g., RFC 7807 Problem Details), and does not leak stack traces.

**Beginner-Friendly Explanation:** When your code crashes, Express normally returns a generic 500 error. You test that your custom error handler catches the crash, formats a clean JSON error response, and doesn't expose sensitive information like stack traces.

### Purposes

- To verify that uncaught exceptions are caught by the error handler.
- To verify that errors are formatted consistently (e.g., `{ error, message, statusCode }`).
- To verify that sensitive information (stack traces, database errors) is not exposed.
- To verify that the correct HTTP status codes are returned for different error types.

### Syntax Rules and Structure

```js
// Error handler (must have 4 arguments, must be last)
app.use((err, req, res, next) => {
  console.error(err);
  res.status(err.statusCode || 500).json({
    error: err.name || 'InternalServerError',
    message: err.message || 'Something went wrong',
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
});
```

**Rules:**
- The error handler must be registered **after** all routes and other middleware.
- The handler must have exactly **4 arguments** — Express identifies error handlers by arity.
- Test both `next(err)` propagation and thrown errors in async route handlers (Express 5 catches async errors automatically; Express 4 requires a wrapper).
- Verify that the response does not include stack traces in production mode.

### Annotated Code Example

```js
// test/integration/errorHandler.test.js
const request = require('supertest');
const express = require('express');

// Create a test app with error-triggering routes
const app = express();
app.use(express.json());

// Route that throws synchronously
app.get('/throw-sync', (req, res) => {
  throw new Error('Sync error');
});

// Route that throws asynchronously (Express 5)
app.get('/throw-async', async (req, res) => {
  throw new Error('Async error');
});

// Route that passes error via next()
app.get('/next-error', (req, res, next) => {
  next(new Error('Error via next'));
});

// Route that throws a custom error with status code
app.get('/custom-error', (req, res) => {
  const err = new Error('Resource not found');
  err.statusCode = 404;
  err.name = 'NotFoundError';
  throw err;
});

// Global error handler
app.use((err, req, res, next) => {
  res.status(err.statusCode || 500).json({
    error: err.name || 'InternalServerError',
    message: err.message,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
});

describe('Global Error Handler', () => {
  it('should catch synchronous errors', async () => {
    const response = await request(app)
      .get('/throw-sync')
      .expect(500);

    expect(response.body.error).toBe('Error');
    expect(response.body.message).toBe('Sync error');
  });

  it('should catch async errors', async () => {
    const response = await request(app)
      .get('/throw-async')
      .expect(500);

    expect(response.body.message).toBe('Async error');
  });

  it('should catch errors passed via next()', async () => {
    const response = await request(app)
      .get('/next-error')
      .expect(500);

    expect(response.body.message).toBe('Error via next');
  });

  it('should use custom statusCode when provided', async () => {
    const response = await request(app)
      .get('/custom-error')
      .expect(404);

    expect(response.body.error).toBe('NotFoundError');
    expect(response.body.message).toBe('Resource not found');
  });

  it('should not expose stack trace in production', async () => {
    const originalEnv = process.env.NODE_ENV;
    process.env.NODE_ENV = 'production';

    const response = await request(app)
      .get('/throw-sync')
      .expect(500);

    expect(response.body.stack).toBeUndefined();

    process.env.NODE_ENV = originalEnv;
  });
});
```

**Expected Output:**
```
PASS  test/integration/errorHandler.test.js
  Global Error Handler
    ✓ should catch synchronous errors (8 ms)
    ✓ should catch async errors (6 ms)
    ✓ should catch errors passed via next() (5 ms)
    ✓ should use custom statusCode when provided (5 ms)
    ✓ should not expose stack trace in production (4 ms)

Tests: 5 passed, 5 total
```

**Why this output:** Each test triggers a different error path and verifies the error handler catches it. The custom error test verifies that `statusCode` and `name` are respected. The production test verifies that stack traces are not exposed when `NODE_ENV=production`.

### Real-World Cases

- **Public APIs:** Ensuring error responses never leak internal details.
- **Compliance:** Ensuring PII is not exposed in error messages.
- **Debugging:** Including stack traces in development for faster diagnosis.

---

## Core Concept 7: Testing Rate Limiting and CORS

### Definitions

**Core Definition:** Rate limiting integration testing verifies that the rate limiter returns 429 after the threshold is exceeded; CORS integration testing verifies that the correct `Access-Control-Allow-Origin` and related headers are present.

**Technical Definition:** `express-rate-limit` tracks request counts per IP and returns 429 when the threshold is crossed. Integration tests send requests up to and beyond the limit, asserting on the 429 response and the `Retry-After` header. CORS middleware (`cors`) sets response headers based on the request's `Origin` header; tests assert on `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, and `Access-Control-Allow-Headers`.

**Beginner-Friendly Explanation:** You test that after 5 login attempts, the 6th gets a 429 "Too Many Requests" response. You also test that when a request comes from `https://myapp.com`, the server includes the correct CORS header allowing it.

### Purposes

- To verify that rate limiting returns 429 after the threshold.
- To verify that the `Retry-After` header is present and correct.
- To verify that CORS headers are set correctly for allowed origins.
- To verify that requests from disallowed origins do not receive CORS headers.

### Sub-Feature 7.1: Testing Rate Limiting

```js
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 60 * 1000,
  max: 5,
  message: { error: 'Too many requests' }
});

app.use('/api/auth', limiter);

// Test: 6th request should be 429
for (let i = 0; i < 5; i++) {
  await request(app).post('/api/auth/login').send(validCredentials).expect(200);
}
const response = await request(app).post('/api/auth/login').send(validCredentials).expect(429);
expect(response.body.error).toBe('Too many requests');
```

**Rules:**
- Send requests up to the limit, then one beyond, and assert on the 429.
- Verify the `Retry-After` header is present in the 429 response.
- Use a short `windowMs` in tests to avoid long waits.
- Reset the rate limiter between tests (or use a new app instance).

### Sub-Feature 7.2: Testing CORS

```js
const cors = require('cors');

app.use(cors({
  origin: ['https://myapp.com', 'https://admin.myapp.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  credentials: true
}));

// Test: allowed origin receives CORS header
const response = await request(app)
  .get('/api/data')
  .set('Origin', 'https://myapp.com')
  .expect(200);

expect(response.headers['access-control-allow-origin']).toBe('https://myapp.com');
```

### Annotated Code Example

```js
// test/integration/securityMiddleware.test.js
const request = require('supertest');
const express = require('express');
const rateLimit = require('express-rate-limit');
const cors = require('cors');

const app = express();
app.use(express.json());

// Rate limiter
const limiter = rateLimit({
  windowMs: 60 * 1000,
  max: 3,
  message: { error: 'Too many requests', retryAfter: 60 }
});

// CORS
app.use(cors({
  origin: ['https://myapp.com', 'https://admin.myapp.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true
}));

app.get('/api/data', (req, res) => {
  res.json({ data: 'sensitive' });
});

app.post('/api/login', limiter, (req, res) => {
  res.json({ token: 'abc123' });
});

describe('Security Middleware', () => {
  describe('Rate Limiting', () => {
    it('should allow requests under the limit', async () => {
      for (let i = 0; i < 3; i++) {
        await request(app)
          .post('/api/login')
          .send({ email: 'test@example.com', password: 'pass' })
          .expect(200);
      }
    });

    it('should return 429 after exceeding the limit', async () => {
      // Exhaust the limit
      for (let i = 0; i < 3; i++) {
        await request(app).post('/api/login').send({});
      }

      const response = await request(app)
        .post('/api/login')
        .send({})
        .expect(429);

      expect(response.body.error).toBe('Too many requests');
      expect(response.body.retryAfter).toBe(60);
    });
  });

  describe('CORS', () => {
    it('should set CORS headers for allowed origin', async () => {
      const response = await request(app)
        .get('/api/data')
        .set('Origin', 'https://myapp.com')
        .expect(200);

      expect(response.headers['access-control-allow-origin']).toBe('https://myapp.com');
      expect(response.headers['access-control-allow-credentials']).toBe('true');
    });

    it('should not set CORS headers for disallowed origin', async () => {
      const response = await request(app)
        .get('/api/data')
        .set('Origin', 'https://evil.com')
        .expect(200);

      expect(response.headers['access-control-allow-origin']).toBeUndefined();
    });

    it('should respond to preflight OPTIONS requests', async () => {
      const response = await request(app)
        .options('/api/data')
        .set('Origin', 'https://myapp.com')
        .set('Access-Control-Request-Method', 'POST')
        .expect(204);

      expect(response.headers['access-control-allow-methods']).toContain('POST');
      expect(response.headers['access-control-allow-headers']).toContain('Authorization');
    });
  });
});
```

**Expected Output:**
```
PASS  test/integration/securityMiddleware.test.js
  Security Middleware
    Rate Limiting
      ✓ should allow requests under the limit (15 ms)
      ✓ should return 429 after exceeding the limit (12 ms)
    CORS
      ✓ should set CORS headers for allowed origin (5 ms)
      ✓ should not set CORS headers for disallowed origin (4 ms)
      ✓ should respond to preflight OPTIONS requests (4 ms)

Tests: 5 passed, 5 total
```

**Why this output:** The rate limiting tests verify the 429 response and the `Retry-After` value. The CORS tests verify that allowed origins receive the correct headers, disallowed origins do not, and preflight OPTIONS requests are handled with the correct `Access-Control-Allow-Methods` and `Access-Control-Allow-Headers`. Note that a fresh `app` instance is not created between rate limit tests, so the second test inherits the exhausted counter from the first — this is intentional to test the 429 behaviour.

### Real-World Cases

- **Authentication endpoints:** Rate limiting login attempts to prevent brute force.
- **Public APIs:** Rate limiting per API key to prevent abuse.
- **SPAs:** Testing CORS headers for frontend-backend communication.

---

## Core Concept 8: Testing File Uploads/Downloads

### Definitions

**Core Definition:** File upload/download integration testing verifies that multipart/form-data requests correctly reach Multer middleware, that uploaded files are stored or processed, and that binary download responses are correctly streamed.

**Technical Definition:** Supertest's `.attach(field, filePath)` method sends a file as part of a `multipart/form-data` request. For in-memory files, `.attach(field, Buffer.from('content'), 'filename.txt')` sends a Buffer as a file. For downloads, `.responseType('blob')` causes Supertest to buffer the response body as a `Buffer`, which can be written to a file or inspected for size and content.

**Beginner-Friendly Explanation:** You test that when someone uploads a file, it actually arrives at your server and gets saved. For downloads, you test that the file comes back with the right content and headers.

### Purposes

- To verify that multipart uploads reach the controller and are stored correctly.
- To verify that file validation (type, size) rejects invalid uploads.
- To verify that downloads return the correct content type and file size.
- To verify that download endpoints stream binary data correctly.

### Syntax Rules and Structure

**Upload:**
```js
await request(app)
  .post('/api/upload')
  .attach('file', '/path/to/test-file.pdf')
  .field('description', 'Test document')
  .expect(201);
```

**Upload with Buffer (no file on disk):**
```js
await request(app)
  .post('/api/upload')
  .attach('avatar', Buffer.from('fake-image-data'), 'avatar.png')
  .expect(201);
```

**Download:**
```js
const response = await request(app)
  .get('/api/download/report.pdf')
  .responseType('blob')
  .expect(200);

expect(response.headers['content-type']).toBe('application/pdf');
expect(response.body.length).toBeGreaterThan(0);
```

| Method | Purpose |
|--------|---------|
| `.attach(field, filePath)` | Attach a file from disk. |
| `.attach(field, buffer, filename)` | Attach a Buffer as a file. |
| `.field(name, value)` | Add a form field alongside the file. |
| `.responseType('blob')` | Buffer binary response as `Buffer`. |

**Rules:**
- Use `.attach()` for file uploads and `.field()` for accompanying text fields.
- For in-memory testing, use `Buffer.from()` instead of a real file path.
- For downloads, use `.responseType('blob')` to get the body as a `Buffer`.
- Verify the uploaded file exists on disk (or in memory) after the request.
- Test file size limits and type validation rejection.

### Annotated Code Example

```js
// test/integration/fileUpload.test.js
const request = require('supertest');
const express = require('express');
const multer = require('multer');
const fs = require('fs');
const path = require('path');

const app = express();

// Multer with disk storage
const upload = multer({
  dest: 'test-uploads/',
  limits: { fileSize: 1024 * 1024 },  // 1MB
  fileFilter: (req, file, cb) => {
    const allowed = ['image/jpeg', 'image/png', 'application/pdf'];
    if (allowed.includes(file.mimetype)) {
      cb(null, true);
    } else {
      cb(new Error('Invalid file type'), false);
    }
  }
});

app.post('/api/upload', upload.single('file'), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file uploaded' });
  }
  res.status(201).json({
    filename: req.file.filename,
    originalname: req.file.originalname,
    size: req.file.size,
    mimetype: req.file.mimetype
  });
});

app.get('/api/download/:filename', (req, res) => {
  const filePath = path.join('test-uploads', req.params.filename);
  if (!fs.existsSync(filePath)) {
    return res.status(404).json({ error: 'File not found' });
  }
  res.download(filePath);
});

describe('File Upload and Download', () => {
  afterAll(() => {
    // Clean up test uploads
    if (fs.existsSync('test-uploads')) {
      fs.rmSync('test-uploads', { recursive: true });
    }
  });

  describe('POST /api/upload', () => {
    it('should upload a PDF file successfully', async () => {
      const response = await request(app)
        .post('/api/upload')
        .attach('file', Buffer.from('fake pdf content'), {
          filename: 'test.pdf',
          contentType: 'application/pdf'
        })
        .expect(201);

      expect(response.body.originalname).toBe('test.pdf');
      expect(response.body.mimetype).toBe('application/pdf');
      expect(response.body.size).toBe(16);

      // Verify file was written to disk
      const filePath = path.join('test-uploads', response.body.filename);
      expect(fs.existsSync(filePath)).toBe(true);
    });

    it('should reject a file with invalid MIME type', async () => {
      const response = await request(app)
        .post('/api/upload')
        .attach('file', Buffer.from('malicious script'), {
          filename: 'script.exe',
          contentType: 'application/x-executable'
        })
        .expect(500);

      expect(response.body.error).toBe('Invalid file type');
    });

    it('should reject a file exceeding the size limit', async () => {
      const largeBuffer = Buffer.alloc(2 * 1024 * 1024);  // 2MB
      const response = await request(app)
        .post('/api/upload')
        .attach('file', largeBuffer, 'large.pdf')
        .expect(500);

      expect(response.body.error).toMatch(/File too large|LIMIT_FILE_SIZE/);
    });
  });

  describe('GET /api/download/:filename', () => {
    it('should download a file with correct content type', async () => {
      // First upload a file
      const uploadResponse = await request(app)
        .post('/api/upload')
        .attach('file', Buffer.from('download test content'), {
          filename: 'download-test.txt',
          contentType: 'text/plain'
        })
        .expect(201);

      // Then download it
      const response = await request(app)
        .get(`/api/download/${uploadResponse.body.filename}`)
        .responseType('blob')
        .expect(200);

      expect(response.body).toBeInstanceOf(Buffer);
      expect(response.body.toString()).toBe('download test content');
    });

    it('should return 404 for non-existent file', async () => {
      await request(app)
        .get('/api/download/does-not-exist.pdf')
        .expect(404);
    });
  });
});
```

**Expected Output:**
```
PASS  test/integration/fileUpload.test.js
  File Upload and Download
    POST /api/upload
      ✓ should upload a PDF file successfully (42 ms)
      ✓ should reject a file with invalid MIME type (28 ms)
      ✓ should reject a file exceeding the size limit (35 ms)
    GET /api/download/:filename
      ✓ should download a file with correct content type (38 ms)
      ✓ should return 404 for non-existent file (12 ms)

Tests: 5 passed, 5 total
```

**Why this output:** The upload tests use `.attach()` with Buffers to avoid creating real files on disk. The valid upload verifies the response metadata and confirms the file exists on disk. The invalid type test verifies the `fileFilter` rejects executables. The size limit test verifies Multer's `LIMIT_FILE_SIZE` error. The download test verifies the response is a `Buffer` with the correct content.

### Real-World Cases

- **Profile picture uploads:** Testing JPEG/PNG validation and storage.
- **Document management:** Testing PDF upload and download.
- **Media platforms:** Testing video upload with size limits.

---

## References

- Compile-N-Run: Express Integration Testing — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/12-express-testing/2-express-integration-testing.mdx
- TRAE-Skills: API Integration Testing (Supertest + Jest) — https://github.com/MarcoNasi/TRAE-Skills/blob/main/testing/API_Integration_Testing_Supertest.md
- OneUptime: How to Write Tests for Express with Jest — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-02-02-express-jest-testing/README.md
- EdAncerys: Express API with Prisma and Jest Testing — https://github.com/EdAncerys/express-prisma-app
- Integration Testing Skill (aj-geddes) — https://raw.githubusercontent.com/aj-geddes/useful-ai-prompts/3f5182cfd739fc113f4af5244a1cf342ad7f7911/docs/_skills/integration-testing.md
- SuperTest Node.js API Testing Complete Guide 2026 — https://qaskills.sh/blog/supertest-node-api-testing-complete-guide
- middleware-supertest on npm — https://www.npmjs.com/package/middleware-supertest
- Testing Roles (CIS 526 Textbook) — https://textbooks.cs.ksu.edu/testing-roles
- Rate Limiting in Node.js (Shattered.io) — https://shattered.io/rate-limiting-nodejs/
- Testing with CORS (Loadmill) — https://docs.loadmill.com
- Supertest File Upload Reference — https://github.com/laurigates/claude-plugins/blob/b133ce1a/skills/api-testing/REFERENCE.md
- Supertest Download Testing (Stack Overflow) — https://stackoverflow.com/questions/33791873
- Testing Error Handling Middleware (paperclipai) — https://github.com/paperclipai/paperclip/commit/26bab3b
- RBAC Integration Test Harness — https://github.com/Yeraze/meshmonitor/pull/3986
- Users API Updates (CIS 526 Textbook) — https://textbooks.cs.ksu.edu/users-api-updates
- Auth9 Express Middleware Testing — https://github.com/c9r-io/auth9/blob/main/docs/qa/sdk/05-express-middleware.md
- Jest Mock Functions — https://jestjs.io/docs/mock-functions
- Supertest GitHub Repository — https://github.com/visionmedia/supertest
- MongoDB Memory Server — https://github.com/typegoose/mongodb-memory-server