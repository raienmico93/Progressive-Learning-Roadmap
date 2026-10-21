# Express.js Validation Patterns — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Validation patterns in Express.js are the recurring architectural approaches for checking that data arriving from different parts of an HTTP request—body, query string, route parameters, and headers—conforms to expected rules before it reaches route handlers and business logic.

**Technical Definition**
Express requests carry input in multiple distinct locations: `req.body` (JSON or form payloads), `req.query` (URL query parameters), `req.params` (dynamic route segments), and `req.headers` (HTTP metadata). Each location has different characteristics—`body` may contain structured objects, `query` and `params` are always strings (or arrays), and `headers` are case-insensitive string maps. Validation patterns address these differences through **location-specific chain creators** (e.g., `body()`, `query()`, `param()`, `header()` in express-validator), **schema-based middleware** that validates multiple locations in one pass (Zod, Joi), or **custom middleware** that inspects headers and other inputs. The central pattern is always the same: validate at the request boundary, aggregate errors, and attach validated data to the request before calling `next()`.

**Beginner-Friendly Explanation**
Every request carries data in several “compartments”: the body (what you send in a POST), the query string (the `?page=2` part), the route parameters (the `:id` in `/users/:id`), and the headers (metadata like authorization tokens). Validation patterns are the standard ways to check each compartment. Some tools let you target each compartment individually; others let you validate the whole request against one schema. Either way, the goal is the same: reject bad data before it can cause problems.

---

### Key Characteristics

- **Location-aware** — Validation must account for the different data types and semantics of body, query, params, and headers.
- **Middleware-centric** — Validation runs as Express middleware at the request boundary, before controllers and services.
- **Schema or chain driven** — Libraries provide either declarative schemas (Zod, Joi) or method chains (express-validator) for expressing rules.
- **Aggregated errors** — Best-practice implementations collect all validation failures and return structured field-level details.
- **Validated-data attachment** — Successful validation replaces or augments request properties with sanitized, typed values for downstream use.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- A validation library installed: `express-validator`, `zod`, `joi`, or a middleware wrapper such as `express-zod-validations` or `express-joi-validation`.
- Understanding of Express middleware and the `(req, res, next)` signature.
- Familiarity with `req.body`, `req.query`, `req.params`, and `req.headers`.

---

### Related Programming Areas

- **Validation libraries** — express-validator, Zod, Joi, Yup, Ajv, and class-validator provide the underlying engines.
- **Error handling** — Validation failures integrate with centralized error middleware.
- **Security** — Boundary validation prevents injection, mass assignment, and malformed-input attacks.
- **API design** — Validation schemas define the input contract of an API.
- **Sanitization** — Validation is often paired with transformation (trimming, type coercion).

---

## Core Concept 1: Request Body Validation

### Definitions

**Core Definition**
Request body validation checks the payload of POST, PUT, PATCH, and similar requests—typically JSON or form data—against expected types, required fields, and constraints before the payload is used.

**Technical Definition**
`req.body` contains parsed request data, made available by body-parsing middleware such as `express.json()` or `express.urlencoded()`. Body validation inspects this object and rejects requests whose payload does not conform to the schema. In express-validator, `body('field')` creates a validation chain restricted to `req.body`. In schema-based libraries, the schema is applied to `req.body` directly, often with `stripUnknown` or `.strict()` to reject unrecognized fields and prevent mass assignment. Body validation is the most common validation pattern because bodies carry the richest and most security-sensitive data.

**Beginner-Friendly Explanation**
Body validation is checking the main payload of a request—the data a user submits in a form or that an API client sends as JSON. It ensures required fields are present, types are correct, and values meet constraints before your code uses them.

---

### Purposes

- To ensure that structured payloads conform to the expected schema before persistence or processing.
- To reject missing required fields, incorrect types, and out-of-range values.
- To prevent mass assignment by stripping or rejecting unknown fields.
- To attach a sanitized, typed body to the request for downstream handlers.

---

### Syntax Rules and Structure

#### Complete General Syntax: express-validator Body Chains

```javascript
const { body, validationResult } = require('express-validator');

app.post('/users',
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 }),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    res.status(201).json({ user: req.body });
  }
);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `body('field')` | Creates a chain targeting `req.body.field` |
| `.isEmail()` | Validates format |
| `.normalizeEmail()` | Sanitizes the value |
| `validationResult(req)` | Collects body validation errors |

#### Complete General Syntax: Schema-Based Body Middleware (Zod)

```typescript
const CreateUserSchema = z.object({
  username: z.string().min(3).max(20),
  email: z.email(),
  password: z.string().min(8)
}).strict(); // reject unknown fields

function validateBody<T extends z.ZodTypeAny>(schema: T) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(400).json({ errors: result.error.issues });
    }
    req.body = result.data; // typed, sanitized body
    next();
  };
}
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `schema.safeParse(req.body)` | Validates without throwing |
| `.strict()` | Rejects unrecognized keys |
| `req.body = result.data` | Replaces raw body with validated data |

#### Syntax Rules

- Body parsers must be registered before validation middleware (`app.use(express.json())`).
- Use `stripUnknown: true` (Joi) or `.strict()` (Zod) to control unknown fields.
- Use safe parsing (`.safeParse()`, `{ error, value }`) to avoid uncaught throws in Express.

#### Constraints and Limitations

- `req.body` is `undefined` if no body parser is registered or the content type is not parsed.
- `express-validator`'s standard validators operate on strings; non-string body values are converted first.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: express-validator Body Validation

```javascript
const express = require('express');
const { body, validationResult } = require('express-validator');
const app = express();
app.use(express.json());

app.post('/users',
  body('email').isEmail().withMessage('Valid email required').normalizeEmail(),
  body('password').isLength({ min: 8 }).withMessage('Password too short'),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    res.status(201).json({ user: req.body });
  }
);

app.listen(3000);
```

**Expected Output**

```
POST /users  { "email": "alice@example.com", "password": "secret123" }
→ 201 { "user": { "email": "alice@example.com", "password": "secret123" } }

POST /users  { "email": "invalid", "password": "123" }
→ 400 { "errors": [
  { "path": "email", "msg": "Valid email required" },
  { "path": "password", "msg": "Password too short" }
] }
```

**Why This Output Occurs**

- The body chains validate both fields; `validationResult` collects the errors, and the handler returns them together.

---

### Real-World Cases

- **User registration** — Validating email, password, and profile fields.
- **E-commerce orders** — Validating line items, quantities, and shipping details.
- **Content management** — Validating article titles, bodies, and metadata.

---

## Core Concept 2: Query Validation

### Definitions

**Core Definition**
Query parameter validation checks the key–value pairs in the URL query string (`?key=value`) against expected types, allowed values, and constraints, typically for pagination, filtering, sorting, and search parameters.

**Technical Definition**
`req.query` is an object containing parsed query-string parameters. Because query parameters are always transmitted as strings (or arrays of strings), validation typically includes **type coercion**—converting `page=2` from the string `"2"` to the number `2`. In express-validator, `query('field')` creates a chain restricted to `req.query`. In Express 5, `req.query` is a getter and is **immutable**; sanitizers do not mutate it, so validated values must be retrieved via `matchedData()` or attached to a separate property. Schema-based middleware can validate `req.query` against a schema and replace it with coerced values.

**Beginner-Friendly Explanation**
Query validation checks the `?page=2&limit=20` part of the URL. Because these values arrive as text, you often need to convert them to numbers or booleans. It is essential for pagination, filtering, and search, where invalid values can cause errors or resource exhaustion.

---

### Purposes

- To validate pagination parameters (`page`, `limit`) and prevent unbounded queries.
- To coerce string query values into numbers, booleans, or enums.
- To restrict filters and sort fields to allowed values.
- To reject malformed or malicious query strings before they reach data-access logic.

---

### Syntax Rules and Structure

#### Complete General Syntax: express-validator Query Chains

```javascript
const { query, validationResult, matchedData } = require('express-validator');

app.get('/search',
  query('page').optional().isInt({ min: 1 }).toInt(),
  query('limit').optional().isInt({ min: 1, max: 100 }).toInt(),
  query('sort').optional().isIn(['price', 'date', 'name']),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    const data = matchedData(req, { locations: ['query'] });
    res.json(data);
  }
);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `query('field')` | Targets `req.query.field` |
| `.optional()` | Allows the parameter to be omitted |
| `.toInt()` | Coerces the string to a number |
| `matchedData()` | Retrieves sanitized query values (required in Express 5) |

#### Complete General Syntax: Zod Query Middleware

```typescript
const SearchSchema = z.object({
  page: z.coerce.number().int().min(1).default(1),
  limit: z.coerce.number().int().min(1).max(100).default(10),
  sort: z.enum(['price', 'date', 'name']).optional()
});

function validateQuery<T extends z.ZodTypeAny>(schema: T) {
  return (req, res, next) => {
    const result = schema.safeParse(req.query);
    if (!result.success) return res.status(400).json({ errors: result.error.issues });
    req.validatedQuery = result.data;
    next();
  };
}
```

#### Syntax Rules

- Use `z.coerce.number()` or `.toInt()` to convert string query values to numbers.
- In Express 5, retrieve sanitized query values via `matchedData()` rather than relying on mutation of `req.query`.
- Use `.optional()` or `.default()` for parameters that may be omitted.

#### Constraints and Limitations

- Query parameters can be repeated (`?tag=a&tag=b`), producing arrays; validate both single and array forms.
- `z.coerce.boolean()` uses truthiness, so `"false"` becomes `true`; use explicit transformation for booleans.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Pagination Query Validation

```javascript
const express = require('express');
const { query, validationResult, matchedData } = require('express-validator');
const app = express();

app.get('/items',
  query('page').optional().isInt({ min: 1 }).toInt(),
  query('limit').optional().isInt({ min: 1, max: 100 }).toInt(),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    const data = matchedData(req, { locations: ['query'] });
    res.json(data);
  }
);

app.listen(3000);
```

**Expected Output**

```
GET /items?page=2&limit=20
→ 200 { "page": 2, "limit": 20 }

GET /items?page=abc
→ 400 { "errors": [{ "path": "page", "msg": "Invalid value" }] }
```

**Why This Output Occurs**

- `toInt()` coerces valid strings to numbers; invalid strings fail `.isInt()`. `matchedData` retrieves the sanitized values.

---

### Real-World Cases

- **Search APIs** — Validating `q`, `page`, `limit`, and `sort` parameters.
- **Filtered listings** — Ensuring filter values belong to an allowed set.
- **Date-range queries** — Validating ISO date strings for `from` and `to`.

---

## Core Concept 3: Parameter Validation

### Definitions

**Core Definition**
Route parameter validation checks the dynamic segments of the URL path—such as `:id` in `/users/:id`—against expected formats such as numeric IDs, UUIDs, or slugs.

**Technical Definition**
`req.params` contains route parameters extracted from the URL path by Express's router. These values are always strings and may originate from user input, so they must be validated before being used in database queries or business logic. In express-validator, `param('id')` creates a chain restricted to `req.params`. Schema-based middleware validates `req.params` against a schema, typically with coercion for numeric IDs. Route parameter validation is a critical defense against path traversal, injection, and invalid database lookups.

**Beginner-Friendly Explanation**
Route parameter validation checks the `:id` part of `/users/:id`. If your route expects a number, validation rejects `/users/abc` with a `400` before the handler tries to use “abc” as an ID.

---

### Purposes

- To ensure route parameters match the expected format (integer, UUID, slug) before database queries.
- To coerce string parameters into the correct type for downstream logic.
- To prevent invalid or malicious path segments from reaching data-access code.
- To provide clear `400` responses for malformed route parameters.

---

### Syntax Rules and Structure

#### Complete General Syntax: express-validator Param Chains

```javascript
const { param, validationResult } = require('express-validator');

app.get('/users/:id',
  param('id').isInt({ min: 1 }).withMessage('ID must be a positive integer').toInt(),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    res.json({ userId: req.params.id });
  }
);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `param('id')` | Targets `req.params.id` |
| `.isInt({ min: 1 })` | Validates the format |
| `.toInt()` | Coerces to a number |

#### Complete General Syntax: Zod Param Middleware

```typescript
const ParamsSchema = z.object({
  id: z.coerce.number().int().positive()
});

app.get('/users/:id', validateParams(ParamsSchema), (req, res) => {
  res.json({ userId: req.params.id }); // validated and coerced
});
```

#### Syntax Rules

- Route parameters are always strings; use coercion for numeric IDs.
- Validate parameters **before** the handler uses them in queries.
- Use `param()` (express-validator) or a dedicated params schema to target `req.params` specifically.

#### Constraints and Limitations

- Express 5 makes `req.query` immutable, but `req.params` can generally be replaced with validated values in middleware.
- Parameters with custom formats (UUIDs, slugs) require appropriate validators or regex patterns.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Numeric ID Validation

```javascript
const express = require('express');
const { param, validationResult } = require('express-validator');
const app = express();

app.get('/users/:id',
  param('id').isInt({ min: 1 }).withMessage('ID must be a positive integer').toInt(),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    res.json({ userId: req.params.id });
  }
);

app.listen(3000);
```

**Expected Output**

```
GET /users/42  → 200 { "userId": 42 }
GET /users/abc → 400 { "errors": [{ "path": "id", "msg": "ID must be a positive integer" }] }
```

**Why This Output Occurs**

- `.isInt()` rejects non-numeric IDs; `.toInt()` converts valid strings to numbers.

---

### Real-World Cases

- **REST resource identifiers** — Validating `:id` across CRUD endpoints.
- **UUID routes** — Ensuring parameters match UUID format.
- **Slug-based content** — Validating slugs for blog posts or products.

---

## Core Concept 4: Header Validation

### Definitions

**Core Definition**
Header validation checks HTTP request headers—such as `Authorization`, `Content-Type`, and custom API keys—against expected presence, formats, and values before the request is processed.

**Technical Definition**
`req.headers` is an object containing request headers, with header names lowercased. Header validation is commonly used for authentication tokens (`Authorization: Bearer ...`), API keys (`X-API-Key`), content-type enforcement, and correlation IDs. In express-validator, `header('field')` creates a chain restricted to `req.headers`. Dedicated middleware packages such as `express-zod-validations` expose `validateHeaders(schema)`, and Joi-based packages such as `express-joi-validation` expose `validator.headers(schema)`. Simple custom middleware can also require the presence of a specific header without a full validation library.

**Beginner-Friendly Explanation**
Header validation checks the metadata of the request—things like the authorization token or API key. For example, if an endpoint requires a Bearer token, header validation ensures the `Authorization` header exists and starts with `"Bearer "` before the request proceeds.

---

### Purposes

- To enforce the presence and format of authentication tokens and API keys.
- To validate content-type and accept headers before body processing.
- To ensure required correlation or tracing headers are present.
- To reject requests with malformed or missing security-critical headers.

---

### Syntax Rules and Structure

#### Complete General Syntax: express-validator Header Chain

```javascript
const { header, validationResult } = require('express-validator');

app.get('/protected',
  header('authorization').exists().withMessage('Authorization required')
    .matches(/^Bearer\s.+/).withMessage('Must be a Bearer token'),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(401).json({ errors: errors.array() });
    res.json({ ok: true });
  }
);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `header('authorization')` | Targets `req.headers.authorization` |
| `.exists()` | Requires the header to be present |
| `.matches(/^Bearer\s.+/)` | Validates the token format |

#### Complete General Syntax: Custom Header Middleware

```typescript
function requireHeader(headerName: string) {
  return (req, res, next) => {
    if (!req.headers[headerName]) {
      return res.status(400).json({ error: `Missing required header: ${headerName}` });
    }
    next();
  };
}

app.get('/protected', requireHeader('authorization'), handler);
```

#### Complete General Syntax: Schema-Based Header Validation (Zod)

```typescript
app.get('/protected',
  validateHeaders(z.object({
    authorization: z.string().startsWith('Bearer ')
  })),
  handler
);
```

#### Syntax Rules

- Header names are case-insensitive; express-validator and `req.headers` handle casing automatically.
- Use `.exists()` to require a header; use `.optional()` to allow its absence.
- Validate token formats with `.matches()` or `.startsWith()`.

#### Constraints and Limitations

- Some validation libraries historically did not support header sanitization in the same way as body/query sanitization.
- Headers are always strings; parse numeric or boolean values explicitly.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Authorization Header Validation

```javascript
const express = require('express');
const { header, validationResult } = require('express-validator');
const app = express();

app.get('/protected',
  header('authorization').exists().withMessage('Authorization header required')
    .matches(/^Bearer\s.+/).withMessage('Must be a Bearer token'),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(401).json({ errors: errors.array() });
    res.json({ message: 'Access granted' });
  }
);

app.listen(3000);
```

**Expected Output**

```
GET /protected  (Authorization: Bearer abc123)
→ 200 { "message": "Access granted" }

GET /protected  (no Authorization header)
→ 401 { "errors": [{ "path": "authorization", "msg": "Authorization header required" }] }

GET /protected  (Authorization: Basic abc123)
→ 401 { "errors": [{ "path": "authorization", "msg": "Must be a Bearer token" }] }
```

**Why This Output Occurs**

- `.exists()` requires the header; `.matches()` rejects non-Bearer formats.

---

### Real-World Cases

- **API authentication** — Validating Bearer tokens and API keys.
- **Webhook security** — Verifying signature headers from Stripe or GitHub.
- **Content negotiation** — Enforcing `Content-Type: application/json` for JSON endpoints.

---

## Core Concept 5: Validation Middleware

### Definitions

**Core Definition**
Validation middleware is an Express middleware function—typically produced by a factory that closes over a schema or rule set—that validates one or more parts of the request before the route handler executes.

**Technical Definition**
Validation middleware follows the `(req, res, next)` signature. The factory pattern is standard: `validate(schema)` or `validate({ body, query, params })` returns a middleware function that captures the schemas in its closure and runs them against the corresponding request properties on every request. On success, the middleware typically replaces request properties with validated, sanitized values and calls `next()`; on failure, it short-circuits with a `400` or `422` response containing structured error details. Dedicated packages such as `express-zod-validations` and `express-joi-validation` provide ready-made factories for body, query, params, and headers.

**Beginner-Friendly Explanation**
Validation middleware is the “checkpoint” every request passes through before reaching your route handler. You write a factory that accepts a schema and returns a checkpoint function; every route that needs validation plugs in the appropriate checkpoint. This keeps your handlers clean and your validation consistent.

---

### Purposes

- To enforce validation at the request boundary, before controllers and services.
- To eliminate repetitive validation code from individual route handlers.
- To apply the same validation rules consistently across multiple endpoints.
- To attach validated, sanitized data to the request for downstream use.

---

### Syntax Rules and Structure

#### Complete General Syntax: Multi-Location Validation Factory (Zod)

```typescript
type ValidationOptions = {
  body?: z.ZodSchema;
  params?: z.ZodSchema;
  query?: z.ZodSchema;
  headers?: z.ZodSchema;
};

export const validate = (schemas: ValidationOptions) => {
  return (req: Request, res: Response, next: NextFunction) => {
    try {
      if (schemas.body) req.body = schemas.body.parse(req.body);
      if (schemas.params) req.params = schemas.params.parse(req.params);
      if (schemas.query) req.query = schemas.query.parse(req.query);
      if (schemas.headers) req.headers = schemas.headers.parse(req.headers);
      next();
    } catch (error) {
      return res.status(400).json({ errors: error.issues });
    }
  };
};
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `validate(schemas)` | Factory that closes over schemas |
| `.parse()` | Validates each location; throws on failure |
| `req.body = ...` | Replaces raw values with validated data |
| `catch` | Returns `400` with structured errors |

#### Syntax Rules

- The factory runs once at startup; the returned middleware runs per request.
- Use `.safeParse()` or `try/catch` to avoid uncaught throws.
- Replace request properties with validated values so downstream code uses sanitized data.
- Register validation middleware **before** route handlers.

#### Constraints and Limitations

- `.parse()` throws; use `.safeParse()` or a `try/catch` to return `400` responses.
- Express 5's `req.query` is a getter and may not be assignable; attach validated query data to a separate property if needed.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Multi-Location Zod Validation Middleware

```typescript
import express from 'express';
import { z } from 'zod';

const app = express();
app.use(express.json());

const validate = (schemas: any) => (req: any, res: any, next: any) => {
  try {
    if (schemas.body) req.body = schemas.body.parse(req.body);
    if (schemas.params) req.params = schemas.params.parse(req.params);
    if (schemas.query) req.validatedQuery = schemas.query.parse(req.query);
    if (schemas.headers) req.headers = schemas.headers.parse(req.headers);
    next();
  } catch (error: any) {
    return res.status(400).json({ errors: error.issues });
  }
};

app.put('/posts/:id',
  validate({
    params: z.object({ id: z.string().length(24) }),
    body: z.object({ title: z.string(), content: z.string().min(10) }),
    headers: z.object({ authorization: z.string().startsWith('Bearer ') })
  }),
  (req, res) => {
    res.json({ params: req.params, body: req.body });
  }
);

app.listen(3000);
```

**Expected Output**

```
PUT /posts/abc123def456ghi789jkl012  { "title": "Hello", "content": "Long enough content" }
Authorization: Bearer token
→ 200 { "params": { "id": "abc123def456ghi789jkl012" }, "body": { "title": "Hello", "content": "Long enough content" } }

PUT /posts/short  { "title": "", "content": "short" }
→ 400 { "errors": [ ... ] }
```

**Why This Output Occurs**

- The factory validates params, body, and headers in one middleware; valid data is attached, invalid data produces a `400`.

---

### Real-World Cases

- **REST APIs** — A single validation factory applied to every route with body, params, query, and header schemas.
- **Full-stack TypeScript** — Shared Zod schemas between frontend and backend.
- **Microservices** — Consistent validation across services using a shared middleware package.

---

## Core Concept 6: Validation Patterns Best Practices

### Definitions

**Core Definition**
Validation best practices are the design, security, and maintainability patterns that keep validation complete, safe, and consistent across all request locations in an Express application.

**Technical Definition**
Best practices include: validating all relevant request locations (`body`, `query`, `params`, `headers`); using factory-based middleware for reuse; aggregating all errors rather than stopping at the first; using safe parsing (`.safeParse()`, `{ error, value }`) to avoid uncaught throws; coercing query and param strings to the correct types; stripping unknown fields to prevent mass assignment; attaching validated data to the request for downstream use; and returning consistent, field-level error responses with appropriate `400`/`422` status codes.

**Beginner-Friendly Explanation**
Best practices are the habits that keep validation reliable. Validate every part of the request—body, query, params, and headers. Use factories so you write the middleware once. Collect all errors at once. Strip out extra fields attackers might inject. And always return clear, field-level errors.

---

### Purposes

- To ensure comprehensive coverage of every input location.
- To prevent security vulnerabilities such as mass assignment and injection.
- To keep validation logic reusable and maintainable through middleware factories.
- To provide consistent, user-friendly error responses across the API.

---

### Syntax Rules and Structure

#### Complete General Syntax: Best-Practice Multi-Location Validation

```typescript
const validate = (schemas: ValidationOptions) => (req, res, next) => {
  const results = {
    body: schemas.body?.safeParse(req.body),
    query: schemas.query?.safeParse(req.query),
    params: schemas.params?.safeParse(req.params),
    headers: schemas.headers?.safeParse(req.headers)
  };

  const errors = Object.entries(results)
    .filter(([_, r]) => r && !r.success)
    .flatMap(([location, r]) =>
      r.error.issues.map((issue) => ({
        location,
        field: issue.path.join('.'),
        message: issue.message
      }))
    );

  if (errors.length > 0) {
    return res.status(422).json({
      error: { code: 'VALIDATION_ERROR', message: 'Invalid input', details: errors }
    });
  }

  if (results.body?.success) req.body = results.body.data;
  if (results.params?.success) req.params = results.params.data;
  if (results.query?.success) req.validatedQuery = results.query.data;
  if (results.headers?.success) req.headers = results.headers.data;

  next();
};
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| All locations | Validates body, query, params, and headers |
| Safe parsing | `.safeParse()` avoids uncaught throws |
| Aggregated errors | Flattens issues from all locations |
| Structured response | `422` with field-level details |
| Validated data | Replaces request properties or attaches to dedicated properties |

#### Syntax Rules

- **Validate all locations** — Body, query, params, and headers as appropriate.
- **Use factories** — Write `validate(schemas)` once; reuse it across routes.
- **Use safe parsing** — `.safeParse()` (Zod) or `{ error, value }` (Joi) instead of throwing validators.
- **Aggregate errors** — Collect failures from all locations before responding.
- **Strip unknown fields** — `.strict()` (Zod) or `stripUnknown: true` (Joi).
- **Attach validated data** — Replace request properties or store in `req.validated*`.
- **Return field-level errors** — Include location, field, and message in the `details` array.

#### Constraints and Limitations

- `.parse()` throws; use `.safeParse()` or `try/catch` to return `400`/`422`.
- Express 5's immutable `req.query` requires attaching validated query data to a separate property.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Complete Multi-Location Validation Pipeline

```typescript
import express from 'express';
import { z } from 'zod';

const app = express();
app.use(express.json());

const validate = (schemas: any) => (req: any, res: any, next: any) => {
  const errors: any[] = [];
  const check = (location: string, schema: any) => {
    if (!schema) return;
    const result = schema.safeParse(req[location]);
    if (!result.success) {
      errors.push(...result.error.issues.map((i: any) => ({
        location, field: i.path.join('.'), message: i.message
      })));
    } else {
      req[location] = result.data;
    }
  };
  check('body', schemas.body);
  check('params', schemas.params);
  check('query', schemas.query);
  check('headers', schemas.headers);
  if (errors.length > 0) {
    return res.status(422).json({ error: { code: 'VALIDATION_ERROR', details: errors } });
  }
  next();
};

app.post('/users/:id',
  validate({
    body: z.object({ name: z.string().min(2), email: z.email() }).strict(),
    params: z.object({ id: z.coerce.number().int().positive() }),
    query: z.object({ notify: z.coerce.boolean().optional() }),
    headers: z.object({ authorization: z.string().startsWith('Bearer ') })
  }),
  (req, res) => res.json({ ok: true })
);

app.listen(3000);
```

**Expected Output**

```
POST /users/42?notify=true  { "name": "Alice", "email": "alice@example.com" }
Authorization: Bearer token
→ 200 { "ok": true }

POST /users/abc  { "name": "A", "email": "invalid", "isAdmin": true }
→ 422 { "error": { "code": "VALIDATION_ERROR", "details": [
  { "location": "body", "field": "", "message": "Unrecognized key(s): 'isAdmin'" },
  { "location": "body", "field": "name", "message": "Too small..." },
  { "location": "body", "field": "email", "message": "Invalid email" },
  { "location": "params", "field": "id", "message": "Expected number, received nan" }
] } }
```

**Why This Output Occurs**

- All four locations are validated; `.strict()` rejects the unknown `isAdmin` field; all errors are aggregated with their location and field paths.

---

### Real-World Cases

- **Production APIs** — A single validation factory applied to every route ensures consistent coverage of body, query, params, and headers.
- **Security-sensitive applications** — Strict schemas and unknown-field rejection prevent mass assignment and parameter pollution.
- **Full-stack TypeScript** — Shared Zod schemas and inferred types keep frontend and backend contracts aligned.

---

## References

- express-validator: Validation middlewares / check API — https://express-validator.github.io/docs/6.0.0/check-api/
- express-validator: checkSchema API — https://express-validator.github.io/docs/next/api/check-schema/
- express-validator 实战：Express 数据校验完全指南 — https://www.guance.com/learn/articles/express-validator-nodejs
- Validating Path and Query Parameters with Middleware — Steve Kinney — https://stevekinney.com/courses/full-stack-typescript/validating-query-and-path-parameters
- Adding Types to Headers and Cookies with Express — Steve Kinney — https://github.com/stevekinney/stevekinney.net/blob/main/courses/full-stack-typescript/typing-headers-and-cookies-in-express.md
- express-zod-validations (npm) — https://www.npmjs.com/package/express-zod-validations
- express-joi-validation (npm) — https://www.npmjs.com/package/express-joi-validation
- Input Validation in Node.js: A Complete Guide — https://www.srprogramist.com/blog/input-validation-nodejs
- req-valid-express (npm) — https://www.npmjs.com/package/req-valid-express