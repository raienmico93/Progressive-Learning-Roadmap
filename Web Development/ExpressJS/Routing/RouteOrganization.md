# Express.js Route Organization — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Route organization is the practice of structuring an Express.js application's routing layer into modular, resource-specific files and routers, mounted at distinct URL prefixes, to improve maintainability, scalability, and separation of concerns.

**Technical Definition:** In Express, route organization is achieved by creating multiple `express.Router()` instances — each responsible for a specific domain (users, authentication, products, orders, admin) — and mounting them onto the main application via `app.use(path, router)`. Each router encapsulates its own middleware stack, parameter handlers, and HTTP method routes. API versioning is layered on top by mounting separate router instances at versioned prefixes (e.g., `/api/v1`, `/api/v2`) or by resolving the version from request headers.

**Beginner-Friendly Explanation:** Imagine your Express application as a large office building. Instead of having one giant room where everyone does everything, you create separate departments: a "Users" department, an "Auth" department, a "Products" department, and so on. Each department has its own staff (middleware) and its own set of responsibilities (routes). When a visitor (request) arrives, they are directed to the correct department based on the URL. This is route organization — keeping things tidy so you can find what you need and change one thing without breaking everything else.

### Key Characteristics

- **Resource-based separation:** Each domain resource (users, products, orders) has its own route file and router instance.
- **Prefix-based mounting:** Routers are mounted at distinct URL prefixes (`/users`, `/products`, `/orders`) via `app.use()`.
- **Middleware scoping:** Security guards, logging, and validation middleware can be scoped exclusively to a router.
- **Version isolation:** Different API versions (`/api/v1`, `/api/v2`) are separate router instances, allowing independent evolution.
- **Controller delegation:** Route handlers delegate business logic to controllers, keeping routing thin.
- **Testability:** Each router can be imported and tested in isolation from the main application.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript knowledge:** Functions, modules (`require`/`module.exports`), and asynchronous code.
- **Understanding of Express routing and middleware.**
- **Familiarity with REST API design principles.**
- **Basic understanding of JWT or session-based authentication** (for auth routes).

### Related Programming Areas

- **Middleware architecture:** Routers are middleware-aware containers with their own stacks.
- **MVC pattern:** Routes (V) delegate to controllers (C) and models (M).
- **API versioning:** A specialised application of route organization.
- **Authentication and authorisation:** Auth routes and admin routes handle identity and permissions.
- **E-commerce systems:** Product and order routes model the core business domain.
- **Testing:** Isolated routers are unit-testable without starting the full application.

### Core Concepts

1. **User Routes** — profile management, account settings, and resource fetch endpoints.
2. **Authentication Routes** — login, registration, token refreshes, password resets, and session termination.
3. **Product Routes** — catalog browsing, inventory tracking, metadata lookups, and pricing APIs.
4. **Order Routes** — checkout lifecycles, payment processing hooks, order status updates, and history logs.
5. **Admin Routes** — high-privilege management consoles, system metrics, and analytical endpoints.
6. **API Versioning** — `/api/v1`, `/api/v2`, and alternative versioning strategies.

---

## Core Concept 1: User Routes

### Definitions

**Core Definition:** User routes handle all endpoints related to user identity, profile management, account settings, and user-specific resource retrieval.

**Technical Definition:** User routes are typically mounted at `/users` or `/user` and handle CRUD operations on user profiles, account settings, and user-owned resources. They are protected by authentication middleware for most operations, with only registration and login handled by separate auth routes.

**Beginner-Friendly Explanation:** User routes are like the "HR department" of your application — they handle everything about a user's own account: viewing their profile, updating their email, changing their password, and fetching their personal data.

### Purposes

- To provide endpoints for profile management (view, edit, update).
- To manage account settings (email, password, preferences).
- To fetch user-owned resources (e.g., a user's posts, orders, or settings).
- To separate user-facing identity operations from authentication flows.

### Syntax Rules and Structure

```js
// routes/users.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');
const { requireAuth } = require('../middleware/auth');

// Profile management (authenticated)
router.get('/profile', requireAuth, userController.getProfile);
router.put('/profile', requireAuth, userController.updateProfile);

// Account settings (authenticated)
router.put('/settings/password', requireAuth, userController.changePassword);
router.put('/settings/email', requireAuth, userController.changeEmail);

// User-specific resources (authenticated)
router.get('/me/orders', requireAuth, userController.getMyOrders);

// Public user profile (no auth)
router.get('/:id', userController.getPublicProfile);

module.exports = router;
```

| Route | Method | Purpose |
|-------|--------|---------|
| `/profile` | GET | Retrieve the authenticated user's profile. |
| `/profile` | PUT | Update the authenticated user's profile. |
| `/settings/password` | PUT | Change password (requires current password). |
| `/settings/email` | PUT | Update email address. |
| `/me/orders` | GET | Fetch the user's order history. |
| `/:id` | GET | Public profile view (sanitised fields only). |

### Annotated Code Example

```js
// routes/users.js
const express = require('express');
const router = express.Router();
const { requireAuth } = require('../middleware/auth');

// Simulated user data
let users = [
  { id: 1, name: 'Alice', email: 'alice@example.com', role: 'user' }
];

// GET /users/profile — retrieve authenticated user's profile
router.get('/profile', requireAuth, (req, res) => {
  const user = users.find(u => u.id === req.userId);
  if (!user) return res.status(404).json({ error: 'User not found' });
  res.json(user);
});

// PUT /users/profile — update user's profile
router.put('/profile', requireAuth, (req, res) => {
  const user = users.find(u => u.id === req.userId);
  if (!user) return res.status(404).json({ error: 'User not found' });
  user.name = req.body.name || user.name;
  res.json(user);
});

// PUT /users/settings/password — change password
router.put('/settings/password', requireAuth, (req, res) => {
  const { currentPassword, newPassword } = req.body;
  if (!currentPassword || !newPassword) {
    return res.status(400).json({ error: 'Both passwords required' });
  }
  // In production: verify currentPassword hash, hash newPassword
  res.json({ message: 'Password updated successfully' });
});

module.exports = router;
```

**Expected Output (for `GET /users/profile` with valid token):**
```
{"id":1,"name":"Alice","email":"alice@example.com","role":"user"}
```

**Why this output:** The `requireAuth` middleware verifies the JWT token and sets `req.userId`. The handler finds the user by ID and returns their profile. The `PUT` handler updates the `name` field only if provided.

### Real-World Cases

- **Social media:** `/users/profile`, `/users/:id` for public profiles.
- **SaaS platforms:** `/users/settings/billing`, `/users/settings/notifications`.
- **E-commerce:** `/users/me/orders`, `/users/me/addresses` for user-specific data.
- **Healthcare:** `/patients/profile`, `/patients/settings` for HIPAA-compliant account management.

---

## Core Concept 2: Authentication Routes

### Definitions

**Core Definition:** Authentication routes handle identity verification, session establishment, and credential management — including login, registration, token refreshes, password resets, and logout.

**Technical Definition:** Auth routes are typically mounted at `/auth` and are the only routes accessible without prior authentication (except token refresh and logout, which require a valid refresh token). They issue access tokens (JWTs) and refresh tokens, validate credentials, and manage the lifecycle of user sessions.

**Beginner-Friendly Explanation:** Auth routes are like the "front door" of your application. They check who you are (login), let new people in (registration), give you a key (token), and handle what happens when you forget your key (password reset) or want to leave (logout).

### Purposes

- To register new users and create their accounts.
- To authenticate users and issue access tokens.
- To refresh expired access tokens using refresh tokens.
- To initiate and complete password reset flows.
- To terminate sessions (logout) and invalidate refresh tokens.

### Syntax Rules and Structure

```js
// routes/auth.js
const express = require('express');
const router = express.Router();
const authController = require('../controllers/authController');

router.post('/register', authController.register);        // Create account
router.post('/login', authController.login);              // Issue tokens
router.post('/refresh', authController.refresh);          // Refresh access token
router.post('/forgot-password', authController.forgotPassword);
router.post('/reset-password/:token', authController.resetPassword);
router.post('/logout', authController.logout);            // Invalidate tokens

module.exports = router;
```

| Route | Method | Purpose |
|-------|--------|---------|
| `/register` | POST | Create a new user account. |
| `/login` | POST | Authenticate and issue access + refresh tokens. |
| `/refresh` | POST | Exchange a valid refresh token for a new access token. |
| `/forgot-password` | POST | Send password reset email with token. |
| `/reset-password/:token` | POST | Reset password using the token. |
| `/logout` | POST | Invalidate refresh token and end session. |

**Rules:**
- Auth routes must be mounted **before** any router-level authentication middleware on the main app, or excluded from global auth.
- Passwords must be hashed (bcrypt, argon2) before storage; never store plaintext.
- Access tokens should be short-lived (15 minutes); refresh tokens longer-lived (7–30 days).
- Refresh tokens should be rotated on each use and stored securely (HTTP-only cookies or database).

### Annotated Code Example

```js
// routes/auth.js
const express = require('express');
const router = express.Router();
const jwt = require('jsonwebtoken');

const ACCESS_SECRET = 'access-secret';
const REFRESH_SECRET = 'refresh-secret';

// POST /auth/register
router.post('/register', (req, res) => {
  const { email, password } = req.body;
  if (!email || !password) {
    return res.status(400).json({ error: 'Email and password required' });
  }
  // In production: hash password with bcrypt
  res.status(201).json({ message: 'User registered', email });
});

// POST /auth/login
router.post('/login', (req, res) => {
  const { email, password } = req.body;
  // In production: verify against hashed password in DB
  if (password !== 'correct-password') {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  const accessToken = jwt.sign({ userId: 1, email }, ACCESS_SECRET, {
    expiresIn: '15m'
  });
  const refreshToken = jwt.sign({ userId: 1 }, REFRESH_SECRET, {
    expiresIn: '7d'
  });
  res.json({ accessToken, refreshToken });
});

// POST /auth/refresh
router.post('/refresh', (req, res) => {
  const { refreshToken } = req.body;
  if (!refreshToken) {
    return res.status(400).json({ error: 'Refresh token required' });
  }
  try {
    const payload = jwt.verify(refreshToken, REFRESH_SECRET);
    const newAccessToken = jwt.sign(
      { userId: payload.userId }, ACCESS_SECRET, { expiresIn: '15m' }
    );
    res.json({ accessToken: newAccessToken });
  } catch {
    res.status(401).json({ error: 'Invalid or expired refresh token' });
  }
});

// POST /auth/logout
router.post('/logout', (req, res) => {
  // In production: invalidate refresh token in DB
  res.json({ message: 'Logged out successfully' });
});

module.exports = router;
```

**Expected Output (for `POST /auth/login` with `{ "email": "alice@example.com", "password": "correct-password" }`):**
```
{"accessToken":"eyJhbGciOiJIUzI1NiIs...","refreshToken":"eyJhbGciOiJIUzI1NiIs..."}
```

**Expected Output (for `POST /auth/refresh` with valid refresh token):**
```
{"accessToken":"eyJhbGciOiJIUzI1NiIs..."}
```

**Why this output:** The login handler verifies credentials (simulated), then signs two JWTs: a short-lived access token and a longer-lived refresh token. The refresh handler verifies the refresh token and issues a new access token without requiring the user to re-enter credentials.

### Real-World Cases

- **JWT-based APIs:** Access + refresh token rotation (used by most modern SPAs).
- **OAuth 2.0 flows:** `/auth/google`, `/auth/github` for social login.
- **Enterprise SSO:** `/auth/saml`, `/auth/oidc` for corporate identity providers.
- **Multi-factor authentication:** `/auth/verify-otp`, `/auth/verify-mfa`.

---

## Core Concept 3: Product Routes

### Definitions

**Core Definition:** Product routes handle catalog browsing, inventory tracking, product metadata lookups, and pricing APIs for an e-commerce or inventory management system.

**Technical Definition:** Product routes are mounted at `/products` (or `/api/products`) and typically expose a combination of public endpoints (catalog browsing, product details) and protected endpoints (inventory updates, pricing changes) that require admin or manager privileges.

**Beginner-Friendly Explanation:** Product routes are like the "storefront" of an e-commerce application. They let customers browse what's available (catalog), look at details (metadata), check prices, and let administrators manage stock levels and product information.

### Purposes

- To provide catalog browsing with filtering, sorting, and pagination.
- To expose product metadata (description, images, categories, specifications).
- To enable inventory tracking (stock levels, reservations, restock alerts).
- To serve pricing APIs (base price, discounts, tiered pricing).

### Syntax Rules and Structure

```js
// routes/products.js
const express = require('express');
const router = express.Router();
const productController = require('../controllers/productController');
const { requireAuth, requireRole } = require('../middleware/auth');

// Public catalog browsing
router.get('/', productController.listProducts);          // List with filters
router.get('/search', productController.searchProducts);   // Full-text search
router.get('/:id', productController.getProduct);          // Product details

// Inventory tracking (admin/manager)
router.get('/:id/inventory', requireAuth, requireRole('admin', 'manager'), productController.getInventory);
router.patch('/:id/inventory', requireAuth, requireRole('admin', 'manager'), productController.updateInventory);

// Pricing (admin only)
router.put('/:id/pricing', requireAuth, requireRole('admin'), productController.updatePricing);

// Metadata management (admin/manager)
router.patch('/:id/metadata', requireAuth, requireRole('admin', 'manager'), productController.updateMetadata);

module.exports = router;
```

| Route | Method | Access | Purpose |
|-------|--------|--------|---------|
| `/` | GET | Public | List products with filters, sort, pagination. |
| `/search` | GET | Public | Full-text search across product fields. |
| `/:id` | GET | Public | Retrieve a single product's details. |
| `/:id/inventory` | GET | Admin/Manager | Get stock levels and reservations. |
| `/:id/inventory` | PATCH | Admin/Manager | Update stock quantity. |
| `/:id/pricing` | PUT | Admin | Update base price, discounts, tiers. |
| `/:id/metadata` | PATCH | Admin/Manager | Update description, images, categories. |

### Annotated Code Example

```js
// routes/products.js
const express = require('express');
const router = express.Router();

const products = [
  { id: 1, name: 'Laptop', price: 1200, stock: 10, category: 'electronics' },
  { id: 2, name: 'Phone', price: 800, stock: 25, category: 'electronics' },
  { id: 3, name: 'Desk', price: 350, stock: 5, category: 'furniture' }
];

// GET /products — list with filtering, sorting, pagination
router.get('/', (req, res) => {
  let result = [...products];
  const { category, minPrice, maxPrice, sortBy, order } = req.query;

  if (category) result = result.filter(p => p.category === category);
  if (minPrice) result = result.filter(p => p.price >= Number(minPrice));
  if (maxPrice) result = result.filter(p => p.price <= Number(maxPrice));

  if (sortBy) {
    const dir = order === 'desc' ? -1 : 1;
    result.sort((a, b) => (a[sortBy] > b[sortBy] ? dir : -dir));
  }

  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const offset = (page - 1) * limit;

  res.json({
    data: result.slice(offset, offset + limit),
    meta: { total: result.length, page, limit }
  });
});

// GET /products/:id — single product details
router.get('/:id', (req, res) => {
  const product = products.find(p => p.id === parseInt(req.params.id));
  if (!product) return res.status(404).json({ error: 'Product not found' });
  res.json(product);
});

module.exports = router;
```

**Expected Output (for `GET /products?category=electronics&sortBy=price&order=asc`):**
```
{"data":[{"id":2,"name":"Phone","price":800,"stock":25,"category":"electronics"},{"id":1,"name":"Laptop","price":1200,"stock":10,"category":"electronics"}],"meta":{"total":2,"page":1,"limit":10}}
```

**Why this output:** The handler filters by `category=electronics`, sorts by `price` ascending, and returns the two matching products. The `meta` object provides pagination information.

### Real-World Cases

- **E-commerce catalogues:** `/products?category=electronics&sortBy=price`.
- **Inventory management:** `PATCH /products/:id/inventory` to update stock after a sale.
- **Pricing APIs:** `PUT /products/:id/pricing` to apply discounts or tiered pricing.
- **Marketplace platforms:** `/products/search?q=wireless+headphones`.

---

## Core Concept 4: Order Routes

### Definitions

**Core Definition:** Order routes manage the checkout lifecycle, payment processing hooks, order status updates, and order history logs for an e-commerce or transactional system.

**Technical Definition:** Order routes are mounted at `/orders` and typically require authentication. They handle the creation of orders from a cart, initiate payment processing (often integrating with payment gateways like Stripe or PayPal), update order status through a state machine (pending → paid → shipped → delivered), and provide order history retrieval.

**Beginner-Friendly Explanation:** Order routes are like the "checkout counter" and "order tracking" system of an online store. They turn a shopping cart into an order, process the payment, track where the order is in its journey, and let customers view their past orders.

### Purposes

- To create orders from a user's cart or direct purchase.
- To process payments and integrate with payment gateways.
- To update order status through a defined lifecycle.
- To provide order history and tracking for users and admins.
- To handle refunds, cancellations, and returns.

### Syntax Rules and Structure

```js
// routes/orders.js
const express = require('express');
const router = express.Router();
const orderController = require('../controllers/orderController');
const { requireAuth } = require('../middleware/auth');

router.post('/', requireAuth, orderController.createOrder);             // Create order
router.get('/', requireAuth, orderController.getMyOrders);              // Order history
router.get('/:id', requireAuth, orderController.getOrder);              // Order details
router.patch('/:id/status', requireAuth, orderController.updateStatus); // Update status
router.post('/:id/pay', requireAuth, orderController.processPayment);   // Payment hook
router.post('/:id/refund', requireAuth, orderController.refundOrder);   // Refund
router.get('/:id/payments', requireAuth, orderController.getPaymentHistory);

module.exports = router;
```

| Route | Method | Purpose |
|-------|--------|---------|
| `/` | POST | Create a new order from cart. |
| `/` | GET | Retrieve the authenticated user's order history. |
| `/:id` | GET | Get a specific order's details. |
| `/:id/status` | PATCH | Update order status (pending → paid → shipped → delivered). |
| `/:id/pay` | POST | Process payment for an order. |
| `/:id/refund` | POST | Initiate a refund. |
| `/:id/payments` | GET | Retrieve payment history for an order. |

**Rules:**
- Orders should be created in a `pending` state; only move to `paid` after payment confirmation.
- Payment hooks should be **idempotent** (the same event processed twice should not double-charge).
- Use webhooks from payment gateways to update order status asynchronously.
- Order history should be paginated and sortable.

### Annotated Code Example

```js
// routes/orders.js
const express = require('express');
const router = express.Router();

const orders = [
  { id: 1, userId: 1, items: ['Laptop'], total: 1200, status: 'paid', createdAt: '2026-01-15' }
];

// POST /orders — create a new order
router.post('/', (req, res) => {
  const { items, total } = req.body;
  if (!items || !items.length) {
    return res.status(400).json({ error: 'Items are required' });
  }
  const newOrder = {
    id: orders.length + 1,
    userId: req.userId,
    items,
    total,
    status: 'pending',
    createdAt: new Date().toISOString()
  };
  orders.push(newOrder);
  res.status(201).json(newOrder);
});

// GET /orders — user's order history
router.get('/', (req, res) => {
  const userOrders = orders.filter(o => o.userId === req.userId);
  res.json(userOrders);
});

// PATCH /orders/:id/status — update order status
router.patch('/:id/status', (req, res) => {
  const order = orders.find(o => o.id === parseInt(req.params.id));
  if (!order) return res.status(404).json({ error: 'Order not found' });

  const validTransitions = {
    pending: ['paid', 'cancelled'],
    paid: ['shipped', 'refunded'],
    shipped: ['delivered'],
    delivered: ['refunded']
  };

  const allowed = validTransitions[order.status] || [];
  if (!allowed.includes(req.body.status)) {
    return res.status(400).json({
      error: `Cannot transition from ${order.status} to ${req.body.status}`
    });
  }

  order.status = req.body.status;
  res.json(order);
});

// POST /orders/:id/pay — payment processing hook
router.post('/:id/pay', (req, res) => {
  const order = orders.find(o => o.id === parseInt(req.params.id));
  if (!order) return res.status(404).json({ error: 'Order not found' });
  if (order.status !== 'pending') {
    return res.status(400).json({ error: 'Order is not pending payment' });
  }
  // In production: call Stripe/PayPal API, handle webhook confirmation
  order.status = 'paid';
  res.json({ message: 'Payment processed', orderId: order.id });
});

module.exports = router;
```

**Expected Output (for `POST /orders` with `{ "items": ["Phone"], "total": 800 }`):**
```
{"id":2,"userId":1,"items":["Phone"],"total":800,"status":"pending","createdAt":"2026-01-15T12:00:00.000Z"}
```

**Expected Output (for `PATCH /orders/2/status` with `{ "status": "paid" }`):**
```
{"id":2,"userId":1,"items":["Phone"],"total":800,"status":"paid","createdAt":"..."}
```

**Expected Output (for `PATCH /orders/1/status` with `{ "status": "pending" }`):**
```
{"error":"Cannot transition from paid to pending"}
```

**Why this output:** The status transition handler enforces a state machine: `pending` can only move to `paid` or `cancelled`, `paid` to `shipped` or `refunded`, etc. Invalid transitions are rejected. The payment hook updates the order to `paid` and would integrate with a payment gateway in production.

### Real-World Cases

- **E-commerce checkout:** `POST /orders` → `POST /orders/:id/pay` → webhook confirms payment.
- **Food delivery:** Order status lifecycle (placed → preparing → out-for-delivery → delivered).
- **Subscription billing:** `POST /orders/:id/refund` for cancelled subscriptions.
- **Marketplace:** `/orders/:id/payments` to track split payments to sellers.

---

## Core Concept 5: Admin Routes

### Definitions

**Core Definition:** Admin routes expose high-privilege management consoles, system metrics, and analytical endpoints that are restricted to users with administrative roles.

**Technical Definition:** Admin routes are mounted at `/admin` (or `/api/admin`) and are protected by both authentication and role-based authorisation middleware. They provide endpoints for user management, system health monitoring, analytics dashboards, and configuration changes.

**Beginner-Friendly Explanation:** Admin routes are like the "control room" of your application — a restricted area where authorised staff can manage users, view system health, check analytics, and configure settings. Regular users cannot access these routes.

### Purposes

- To provide management consoles for user administration (create, edit, suspend users).
- To expose system metrics (CPU, memory, uptime, request rates).
- To serve analytical endpoints (revenue reports, user growth, product performance).
- To enable configuration changes (feature flags, maintenance mode).
- To provide audit logs and security monitoring.

### Syntax Rules and Structure

```js
// routes/admin.js
const express = require('express');
const router = express.Router();
const adminController = require('../controllers/adminController');
const { requireAuth, requireRole } = require('../middleware/auth');

// All admin routes require authentication + admin role
router.use(requireAuth);
router.use(requireRole('admin'));

// User management
router.get('/users', adminController.listUsers);
router.patch('/users/:id/role', adminController.updateUserRole);
router.delete('/users/:id', adminController.deleteUser);

// System metrics
router.get('/metrics', adminController.getSystemMetrics);
router.get('/health', adminController.getHealthCheck);

// Analytics
router.get('/analytics/revenue', adminController.getRevenueAnalytics);
router.get('/analytics/users', adminController.getUserGrowth);
router.get('/analytics/products', adminController.getProductPerformance);

// Configuration
router.get('/config', adminController.getConfig);
router.put('/config', adminController.updateConfig);

// Audit logs
router.get('/audit-logs', adminController.getAuditLogs);

module.exports = router;
```

| Route | Method | Purpose |
|-------|--------|---------|
| `/users` | GET | List all users with filters and pagination. |
| `/users/:id/role` | PATCH | Update a user's role. |
| `/metrics` | GET | System metrics (CPU, memory, uptime). |
| `/analytics/revenue` | GET | Revenue reports by period. |
| `/analytics/users` | GET | User growth and retention metrics. |
| `/config` | PUT | Update application configuration. |
| `/audit-logs` | GET | Retrieve audit trail of admin actions. |

**Rules:**
- **All** admin routes must be protected by authentication **and** role-based authorisation.
- Apply `requireRole('admin')` as router-level middleware so it applies to every route in the file.
- Log all admin actions for audit purposes.
- Rate-limit admin endpoints more aggressively than public endpoints.

### Annotated Code Example

```js
// routes/admin.js
const express = require('express');
const router = express.Router();
const { requireAuth, requireRole } = require('../middleware/auth');

// Router-level middleware: all admin routes require auth + admin role
router.use(requireAuth);
router.use(requireRole('admin'));

// GET /admin/dashboard — system statistics
router.get('/dashboard', (req, res) => {
  res.json({
    totalUsers: 1250,
    activeUsers: 890,
    totalOrders: 3400,
    revenue: 245000,
    systemHealth: 'healthy'
  });
});

// GET /admin/metrics — system metrics
router.get('/metrics', (req, res) => {
  res.json({
    cpu: '45%',
    memory: '62%',
    uptime: process.uptime(),
    requestsPerMinute: 120
  });
});

// GET /admin/analytics/revenue — revenue analytics
router.get('/analytics/revenue', (req, res) => {
  const { period = '30d' } = req.query;
  res.json({
    period,
    total: 245000,
    byDay: [
      { date: '2026-01-01', revenue: 8000 },
      { date: '2026-01-02', revenue: 9500 }
    ]
  });
});

module.exports = router;
```

**Expected Output (for `GET /admin/dashboard` with admin token):**
```
{"totalUsers":1250,"activeUsers":890,"totalOrders":3400,"revenue":245000,"systemHealth":"healthy"}
```

**Expected Output (for `GET /admin/dashboard` without token):**
```
{"error":"Unauthorized"}
```

**Why this output:** The `router.use(requireAuth)` and `router.use(requireRole('admin'))` middleware run before every route handler. A request without a valid admin token is rejected before reaching the dashboard handler.

### Real-World Cases

- **SaaS platforms:** `/admin/users` to manage customer accounts and subscriptions.
- **E-commerce:** `/admin/analytics/revenue` for sales dashboards.
- **DevOps:** `/admin/health` and `/admin/metrics` for monitoring integrations.
- **Content platforms:** `/admin/audit-logs` for moderation and compliance.

---

## Core Concept 6: API Versioning

### Definitions

**Core Definition:** API versioning is the practice of exposing multiple versions of an API simultaneously, allowing clients to migrate at their own pace while the server evolves without breaking existing integrations.

**Technical Definition:** In Express, API versioning is implemented by creating separate router instances for each version and mounting them at distinct URL prefixes (`/api/v1`, `/api/v2`), or by resolving the requested version from request headers (e.g., `Accept` or `X-API-Version`) and routing accordingly. URL path versioning is the most common and explicit approach; header versioning keeps URLs clean but is less visible.

**Beginner-Friendly Explanation:** API versioning is like having two editions of a book — the old edition (v1) and the new edition (v2). Readers who have the old edition can keep using it, while new readers get the updated version. In Express, you create separate routers for each version and mount them at `/api/v1` and `/api/v2`.

### Purposes

- To introduce breaking changes without disrupting existing clients.
- To support multiple client generations simultaneously during migration.
- To provide a clear deprecation timeline for old versions.
- To enable independent evolution of different API versions.

---

### Sub-Feature 6.1: `/api/v1` — First-Generation Stable API

#### Syntax Rules and Structure

```js
// routes/v1/users.js
const v1Router = express.Router();

v1Router.get('/users', (req, res) => {
  res.json([{ id: 1, name: 'Alice' }]);  // Flat array (v1 format)
});

module.exports = v1Router;
```

```js
// app.js
app.use('/api/v1', require('./routes/v1/users'));
```

| Component | Breakdown |
|-----------|-----------|
| `/api/v1` | URL prefix for version 1. |
| `v1Router` | Separate router instance for v1. |
| Response format | Original schema (e.g., flat array). |

**Rules:**
- v1 should remain **unchanged** once clients depend on it.
- Bug fixes are allowed; breaking changes are not.
- Set deprecation headers (`Deprecation: true`, `Sunset: <date>`) when v1 is scheduled for removal.

---

### Sub-Feature 6.2: `/api/v2` — Non-Breaking Optimisations or Breaking Overhauls

#### Syntax Rules and Structure

```js
// routes/v2/users.js
const v2Router = express.Router();

v2Router.get('/users', (req, res) => {
  res.json({
    data: [{ id: 1, name: 'Alice', email: 'alice@example.com' }],  // Envelope
    meta: { total: 1, page: 1 }
  });
});

module.exports = v2Router;
```

```js
// app.js
app.use('/api/v1', require('./routes/v1/users'));
app.use('/api/v2', require('./routes/v2/users'));
```

| Change type | v1 | v2 |
|-------------|----|----|
| Response structure | Flat array | Envelope with `data` + `meta` |
| New fields | Not present | `email` added |
| Pagination | Not supported | `page`, `limit`, `total` |

**Rules:**
- v2 can introduce breaking changes (new response format, renamed fields).
- Non-breaking additions (new optional fields) do not require a version bump.
- Document migration guides for clients moving from v1 to v2.

---

### Sub-Feature 6.3: Alternative Versioning Strategies

#### URL Path Versioning vs. Header Versioning

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| **URL Path** | `GET /api/v1/users` | Explicit, cache-friendly, easy to route | URL proliferation, clients must update URLs |
| **Header (Accept)** | `Accept: application/vnd.myapp.v2+json` | Clean URLs, RESTful content negotiation | Harder to test in browser, less visible |
| **Header (Custom)** | `X-API-Version: 2` | Explicit, easy to read | Non-standard header |
| **Query Parameter** | `GET /api/users?version=2` | Easy to test | Not RESTful, caching complexity |

#### Header Versioning Implementation

```js
// middleware/extractVersion.js
function extractVersion(defaultVersion = '1') {
  return (req, res, next) => {
    let version = req.headers['x-api-version'];
    if (!version) {
      const accept = req.get('accept') || '';
      const m = accept.match(/application\/vnd\.myapp\.v(\d+)\+json/);
      version = m ? m[1] : defaultVersion;
    }
    req.apiVersion = version;
    res.setHeader('Vary', 'API-Version');  // Required for caching
    next();
  };
}

// Usage
app.use(extractVersion('1'));

app.get('/api/users', (req, res) => {
  if (req.apiVersion === '2') {
    return res.json({ data: getUsersV2(), meta: { total: 100 } });
  }
  res.json(getUsersV1());  // Flat array
});
```

**Expected Output (for `GET /api/users` with `X-API-Version: 2`):**
```
{"data":[...],"meta":{"total":100}}
```

**Expected Output (for `GET /api/users` without header):**
```
[{"id":1,"name":"Alice"}]
```

**Why this output:** The `extractVersion` middleware reads the `X-API-Version` header. If present and valid, `req.apiVersion` is set; otherwise, it defaults to `'1'`. The route handler branches based on `req.apiVersion`, returning the appropriate response format.

### Real-World Cases

- **Stripe:** URL-based versioning (`/v1/charges`, `/v1/customers`) with date-based API versions.
- **GitHub:** Header-based versioning (`Accept: application/vnd.github.v3+json`).
- **Twitter/X:** URL-based versioning (`/1.1/statuses`, `/2/tweets`).
- **Internal microservices:** Header versioning for cleaner internal URLs.

---

## References

- Express.js Routing Guide — https://expressjs.com/en/guide/routing.html
- Express.js 5.x Router API — https://expressjs.com/en/5x/api/router/
- Express.js Using Middleware Guide — https://expressjs.com/en/guide/using-middleware/
- Express.js 5.x Application Object — `app.use()` — https://expressjs.com/en/5x/api/application/#appuse
- Split a Monolithic Express Router into Domain Routes — https://docs.devinenterprise.com/use-cases/gallery/refactor-module
- Compile-N-Run — Express Route Organization — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/1-express-routing/10-express-route-organization.mdx
- API Versioning Strategies — https://raw.githubusercontent.com/glennguilloux/llm-knowledge-base/refs/heads/main/api-design/versioning.md
- API Versioning Strategies (Comprehensive Guide) — https://raw.githubusercontent.com/vasilyu1983/AI-Agents-public/45953e53d00de7c0035c263c9d11c59a55001108/frameworks/shared-skills/skills/dev-api-design/references/versioning-strategies.md
- Implementing API Header Versioning in Node.js — https://dev.to/silentwatcher_95/implementing-api-header-versioning-in-nodejs-4e29
- Requestly — Mastering Express Routers — https://requestly.com/blog/express-routes/
- Express.js Migration Guide (v4 to v5) — https://expressjs.com/en/guide/migrating-5.html
- E-Commerce API — Product Catalog — https://mintlify.wiki
- How to Implement API Versioning in Express — https://oneuptime.com
- MDN — HTTP Accept Header — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept