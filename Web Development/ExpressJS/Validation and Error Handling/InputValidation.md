# Input Validation — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Input validation is the process of ensuring that the data your application receives — from HTTP request bodies, query parameters, route parameters, and headers — is of the correct type, format, and within acceptable boundaries before it is used by business logic.

**Technical Definition:** Input validation is the first line of defence against SQL injection, XSS, command injection, and other attacks classified by OWASP as A03:2021. Express does not validate input by default — everything arriving in `req.body`, `req.params`, and `req.query` is an untrusted string without explicit validation. Validation libraries such as Zod, Joi, and express-validator provide schema-based validation, type coercion, sanitisation, and standardised error reporting.

**Beginner-Friendly Explanation:** Input validation is like a security checkpoint at an airport. Before anyone (a request) is allowed into the terminal (your business logic), their bags are scanned and their documents are checked. If something doesn't match the rules, they're turned away immediately. This prevents dangerous items (malicious data) from ever reaching the plane.

### Key Characteristics

- **Boundary-first:** Validation happens at the outermost layer, before controllers or business logic.
- **Schema-driven:** Validation rules are defined declaratively in schemas (Zod, Joi) or chains (express-validator).
- **Type coercion:** Libraries like Zod coerce strings to numbers, booleans, and dates automatically.
- **Sanitisation:** Trimming, escaping, and normalising input is part of validation.
- **Standardised errors:** Validation failures produce structured, machine-readable error responses.
- **Defence in depth:** Body, query, params, and headers all require validation.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **A validation library:** `npm install zod` (recommended for TypeScript) or `npm install joi` or `npm install express-validator`.
- **Basic JavaScript or TypeScript knowledge:** Functions, schemas, and asynchronous programming.

### Related Programming Areas

- **Error handling:** Validation failures should be handled by a global error interceptor.
- **Dependency injection:** Validation schemas can be injected into middleware factories.
- **Type systems:** Zod schemas generate TypeScript types at compile time.
- **ORM/query builders:** Validated data flows into parameterised database queries.
- **Security:** Validation prevents injection, prototype pollution, and privilege escalation.

### Core Concepts

1. **Request-Body Validation** — verifying payloads for POST, PUT, and PATCH mutations.
2. **Query Validation** — sanitising and typing query strings.
3. **Parameter Validation** — checking URL route parameters (UUID, ObjectId).
4. **Type Validation** — enforcing strict primitive types.
5. **Required vs. Optional Fields** — differentiating missing from null/empty.
6. **Length & Boundary Constraints** — restricting string sizing, array bounds, regex formats.
7. **Range Constraints** — handling upper and lower numeric boundaries.
8. **"Parse, Don't Validate" Pattern** — transforming raw input into trusted typed structures.

---

## Core Concept 1: Request-Body Validation

### Definitions

**Core Definition:** Request-body validation verifies the payload of POST, PUT, and PATCH requests against a schema, ensuring that mutation operations receive correctly structured data before they modify application state.

**Technical Definition:** Request-body validation applies a schema to `req.body` after body-parsing middleware has populated it. The schema defines required fields, types, formats, and constraints. If validation passes, the parsed (and potentially transformed) data is used; if it fails, a 400 or 422 response is returned before the controller executes. Zod's `z.object()` defines the shape of the body, while express-validator's `body()` chains validators directly on the request body.

**Beginner-Friendly Explanation:** Request-body validation is like checking a package before it's accepted for shipping. The delivery (POST/PUT/PATCH) must contain exactly what was ordered, in the right quantities and formats. If the contents don't match the manifest, the package is rejected.

### Purposes

- To verify payloads for POST, PUT, and PATCH mutations before they modify data.
- To prevent malformed or malicious data from reaching the database.
- To provide immediate, structured feedback to clients when their payload is invalid.
- To enforce data integrity across all mutation endpoints.

### Syntax Rules and Structure

```typescript
// Zod schema for request body
const CreateUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  password: z.string().min(8)
});

// Middleware factory
function validateBody(schema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(422).json({
        error: 'Validation failed',
        details: result.error.flatten().fieldErrors
      });
    }
    req.valid = result.data; // Sanitised data
    next();
  };
}
```

| Component | Breakdown |
|-----------|-----------|
| `z.object({...})` | Defines the expected shape of `req.body`. |
| `safeParse()` | Validates without throwing; returns `{ success, data | error }`. |
| `result.error.flatten().fieldErrors` | Zod's structured field-level error format. |
| `req.valid` | The parsed, sanitised data for downstream use. |

**Rules:**
- Body-parsing middleware (`express.json()`) must run **before** validation middleware.
- Validation schemas should be reusable and defined in a dedicated `validators/` directory.
- On failure, respond with 422 Unprocessable Entity and **do not** call `next()`.
- On success, attach the parsed data to `req.valid` — never use the raw `req.body` downstream.

**Constraints:**
- Zod's `.parse()` throws on failure; `.safeParse()` is preferred for Express middleware.
- express-validator uses chainable validators (`body('email').isEmail()`) instead of schemas.

### Annotated Code Example

```typescript
// validators/user.validator.ts
import { z } from 'zod';

export const CreateUserSchema = z.object({
    name:     z.string().min(1, 'Name is required').max(100),
    email:    z.string().email('Invalid email format'),
    password: z.string().min(8, 'Password must be at least 8 characters'),
    role:     z.enum(['user', 'editor', 'admin']).default('user')
});
```

```typescript
// middleware/validate.ts
import { ZodSchema } from 'zod';

export function validateBody(schema: ZodSchema) {
    return (req: Request, res: Response, next: NextFunction) => {
        const result = schema.safeParse(req.body);

        if (!result.success) {
            return res.status(422).json({
                success: false,
                error: 'Validation failed',
                details: result.error.flatten().fieldErrors
            });
        }
        
        req.valid = result.data;
        next();
    };
}
```

```typescript
// routes/user.routes.ts
router.post('/users', validateBody(CreateUserSchema), userController.create);
```

**Expected Output (for valid request):**
```json
{
    "data": {
        "id": "1",
        "name": "Alice",
        "email": "alice@example.com",
        "role": "user"
    }
}
```

**Expected Output (for invalid request — missing email):**
```json
{
    "success": false,
    "error": "Validation failed",
    "details": {
        "email": ["Invalid email format"],
        "password": ["Password must be at least 8 characters"]
    }
}
```

**Why this output:** The schema validates that `email` is present and matches the email format, and that `password` is at least 8 characters. If validation fails, the middleware returns a 422 with field-level details; the controller is never invoked. If validation passes, `req.valid` contains the parsed data with the default `role` applied.

### Real-World Cases

- **User registration:** Validating email format and password strength before creating an account.
- **Product creation:** Validating product name, price, and category for POST /products.
- **Order placement:** Validating cart items, quantities, and shipping address for POST /orders.
- **Profile updates:** Validating name and email changes for PATCH /users/:id.

---

## Core Concept 2: Query Validation

### Definitions

**Core Definition:** Query validation sanitises and types query string parameters — which are always strings in HTTP — into their intended types, such as converting `?page=1` into the integer `1`.

**Technical Definition:** Query strings are always strings. Zod's `z.coerce.number()` converts query-string values into numbers, so `page=2` becomes the number `2`. The `safeParse` method then validates those coerced values against schema constraints such as `.int().min(1)`. Defaults provide fallbacks for missing parameters. express-validator's `query()` function provides similar functionality with chainable validators.

**Beginner-Friendly Explanation:** Query validation is like translating a handwritten order form into a digital database entry. The form says "page: two" (a string), but the database needs the number 2. Query validation does that translation and checks that the result makes sense (e.g., page number must be at least 1).

### Purposes

- To sanitise and type query strings (e.g., forcing pagination strings like `?page=1` into numeric integers).
- To provide default values for missing query parameters.
- To enforce boundaries (e.g., maximum page size) on query-driven operations.
- To prevent type confusion bugs caused by string-to-number comparisons.

### Syntax Rules and Structure

```typescript
const PaginationSchema = z.object({
    page:      z.coerce.number().int().positive().default(1),
    limit:     z.coerce.number().int().positive().max(100).default(20),
    sortBy:    z.enum(['createdAt', 'name', 'updatedAt']).default('createdAt'),
    sortOrder: z.enum(['asc', 'desc']).default('desc')
});
```

| Component | Breakdown |
|-----------|-----------|
| `z.coerce.number()` | Converts the query string to a number. |
| `.int()` | Requires an integer (no decimals). |
| `.positive()` | Requires a value greater than 0. |
| `.max(100)` | Enforces an upper boundary. |
| `.default(1)` | Provides a fallback if the parameter is absent. |

**Rules:**
- Always use `z.coerce` for query parameters — they arrive as strings.
- Provide defaults for optional query parameters to avoid undefined values.
- Enforce maximum values (e.g., `limit.max(100)`) to prevent abuse.
- Use `.safeParse()` and return a 422 on failure.

**Constraints:**
- Coercion to `NaN` (e.g., `?page=abc`) will fail validation.
- Boolean coercion must be explicit — `"false"` is truthy in JavaScript.

### Annotated Code Example

```typescript
// middleware/validateQuery.ts
import { z } from 'zod';

const PaginationSchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  limit: z.coerce.number().int().positive().max(100).default(20),
  sortBy: z.enum(['createdAt', 'name', 'updatedAt']).default('createdAt'),
  sortOrder: z.enum(['asc', 'desc']).default('desc')
});

export function validateQuery(schema = PaginationSchema) {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.query);
    if (!result.success) {
      return res.status(422).json({
        error: 'Invalid query parameters',
        details: result.error.flatten().fieldErrors
      });
    }
    req.query = result.data; // Replace with coerced values
    next();
  };
}
```

```typescript
// routes/user.routes.ts
router.get('/users', validateQuery(), userController.index);
```

**Expected Output (for `GET /users?page=2&limit=10`):**
```json
{
  "data": [...],
  "pagination": { "page": 2, "limit": 10, "sortBy": "createdAt", "sortOrder": "desc" }
}
```

**Expected Output (for `GET /users?page=abc`):**
```json
{
  "error": "Invalid query parameters",
  "details": {
    "page": ["Expected number, received nan"]
  }
}
```

**Why this output:** `z.coerce.number()` converts `"2"` to `2` and `"10"` to `10`. Defaults provide `createdAt` and `desc` when absent. Invalid values like `"abc"` produce `NaN`, which fails the `.int().positive()` checks, producing a descriptive error.

### Real-World Cases

- **Pagination:** Converting `?page=2&limit=20` into integers for database `LIMIT`/`OFFSET`.
- **Sorting:** Validating `?sortBy=price&sortOrder=desc` against an allow-list of sortable fields.
- **Filtering:** Coercing `?minPrice=10&maxPrice=50` into numeric range filters.
- **Date ranges:** Coercing `?startDate=2026-01-01` into a Date object.

---

## Core Concept 3: Parameter Validation

### Definitions

**Core Definition:** Parameter validation checks URL route parameters — the dynamic segments defined with `:id` or `:uuid` — to ensure they match the expected format, such as a valid UUID or MongoDB ObjectId.

**Technical Definition:** Route parameters are always strings. A MongoDB ObjectId is a 24-character hexadecimal string; a UUID follows the RFC 4122 format. express-validator provides `.isMongoId()` and `.isUUID()` validators, while Zod offers `z.string().uuid()` and custom regex patterns. Validating these at the route level prevents invalid IDs from reaching the database, which would otherwise throw errors.

**Beginner-Friendly Explanation:** Parameter validation is like checking that a ticket number has the right format before you try to look up the reservation. If the ticket should be 8 digits and someone hands you a 20-character string, you know immediately that it's invalid — no need to search the database.

### Purposes

- To check URL route parameters (e.g., enforcing that `:id` matches a valid UUID or MongoDB ObjectId format).
- To prevent database errors caused by invalid ID formats.
- To provide immediate feedback when a resource identifier is malformed.
- To sanitise and cast route parameters before they reach the controller.

### Syntax Rules and Structure

```typescript
// Zod schema for UUID parameter
const UserIdSchema = z.object({
  id: z.string().uuid('Invalid user ID format')
});

// Zod schema for MongoDB ObjectId
const PostIdSchema = z.object({
  id: z.string().regex(/^[a-f\d]{24}$/i, 'Invalid post ID format')
});
```

| Component | Breakdown |
|-----------|-----------|
| `z.string().uuid()` | Validates RFC 4122 UUID format. |
| `z.string().regex(/^[a-f\d]{24}$/i)` | Validates MongoDB ObjectId (24 hex chars). |
| `.regex()` | Custom pattern matching. |

**express-validator equivalent:**
```js
const { param } = require('express-validator');
router.get('/users/:id', param('id').isUUID(), userController.getById);
router.get('/posts/:id', param('id').isMongoId(), postController.getById);
```

**Rules:**
- Parameter validation should run **before** the controller.
- Use `z.string().uuid()` for UUIDs and a 24-character hex regex for MongoDB ObjectIds.
- express-validator's `.isMongoId()` and `.isUUID()` provide built-in validators.
- On failure, return 400 or 422 — not 404 — because the ID format is invalid, not the resource itself.

**Constraints:**
- MongoDB ObjectIds are 24-character hex strings; UUIDs follow the RFC 4122 format.
- Custom IDs (e.g., slugs) require custom regex patterns.

### Annotated Code Example

```typescript
// validators/param.validator.ts
import { z } from 'zod';

export const UuidParamSchema = z.object({
  id: z.string().uuid('ID must be a valid UUID')
});

export const MongoIdParamSchema = z.object({
  id: z.string().regex(/^[a-f\d]{24}$/i, 'ID must be a valid MongoDB ObjectId')
});
```

```typescript
// middleware/validateParams.ts
export function validateParams(schema: ZodSchema) {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.params);
    if (!result.success) {
      return res.status(400).json({
        error: 'Invalid route parameter',
        details: result.error.flatten().fieldErrors
      });
    }
    req.params = result.data;
    next();
  };
}
```

```typescript
// routes/user.routes.ts
router.get('/users/:id', validateParams(UuidParamSchema), userController.getById);
router.get('/posts/:id', validateParams(MongoIdParamSchema), postController.getById);
```

**Expected Output (for `GET /users/123e4567-e89b-12d3-a456-426614174000`):**
```json
{ "data": { "id": "123e4567-e89b-12d3-a456-426614174000", "name": "Alice" } }
```

**Expected Output (for `GET /users/not-a-uuid`):**
```json
{
  "error": "Invalid route parameter",
  "details": { "id": ["ID must be a valid UUID"] }
}
```

**Why this output:** The UUID validator checks the format before the controller runs. A valid UUID passes; an invalid string like `"not-a-uuid"` produces a 400 response with a descriptive message. The controller never attempts a database lookup with an invalid ID.

### Real-World Cases

- **User profiles:** `/users/:id` where `id` must be a UUID.
- **Blog posts:** `/posts/:id` where `id` is a MongoDB ObjectId.
- **Transactions:** `/transactions/:uuid` where `uuid` is a payment transaction ID.
- **Multi-tenant SaaS:** `/orgs/:orgId/users/:userId` — both parameters validated independently.

---

## Core Concept 4: Type Validation

### Definitions

**Core Definition:** Type validation enforces that each field in the request conforms to a strict primitive type — string, number, boolean, object, or array — rejecting values that do not match.

**Technical Definition:** Zod's primitive schemas — `z.string()`, `z.number()`, `z.boolean()`, `z.object()`, `z.array()` — validate that a value is of the expected type. Unlike TypeScript's compile-time types, which vanish at runtime, Zod performs actual runtime checks. If `req.body.age` is the string `"30"` instead of the number `30`, `z.number()` will reject it unless `z.coerce.number()` is used.

**Beginner-Friendly Explanation:** Type validation is like a bouncer checking that each person entering the club is wearing the right kind of wristband — blue for adults, green for teenagers, red for staff. If someone's wristband doesn't match, they're turned away, regardless of who they are.

### Purposes

- To enforce strict primitive types (strings, numbers, booleans, objects, arrays).
- To prevent type confusion bugs caused by implicit coercion.
- To ensure that downstream code receives values of the expected type.
- To provide clear error messages when types are mismatched.

### Syntax Rules and Structure

```typescript
const ProductSchema = z.object({
  name: z.string(),                    // Must be a string
  price: z.number().positive(),        // Must be a positive number
  inStock: z.boolean(),                // Must be a boolean
  tags: z.array(z.string()),           // Must be an array of strings
  metadata: z.object({                 // Must be an object
    weight: z.number(),
    dimensions: z.object({
      width: z.number(),
      height: z.number()
    })
  }).optional()
});
```

| Schema | Validates |
|--------|-----------|
| `z.string()` | String type. |
| `z.number()` | Number type (finite, not NaN). |
| `z.boolean()` | Boolean type. |
| `z.array(z.string())` | Array of strings. |
| `z.object({...})` | Nested object with defined shape. |

**Rules:**
- Use `z.coerce` only when automatic type conversion is desired (e.g., query parameters).
- Use `z.number().int()` for integer-only fields.
- Nested objects should have their own schemas for clarity and reuse.
- Arrays can be validated with `.min(1)` and `.max(n)` for length constraints.

**Constraints:**
- Zod does not accept `Infinity` for numbers by default — all numbers must be finite.
- `z.boolean()` rejects truthy/falsy values — only `true` and `false` pass.

### Annotated Code Example

```typescript
const EventSchema = z.object({
  title: z.string().min(1),
  capacity: z.number().int().positive(),
  isPublic: z.boolean().default(true),
  tags: z.array(z.string()).max(5).default([]),
  location: z.object({
    latitude: z.number().min(-90).max(90),
    longitude: z.number().min(-180).max(180)
  })
});
```

**Expected Output (for valid payload):**
```json
{
  "title": "Node.js Meetup",
  "capacity": 50,
  "isPublic": true,
  "tags": ["node", "express"],
  "location": { "latitude": 40.7128, "longitude": -74.0060 }
}
```

**Expected Output (for invalid payload — capacity as string):**
```json
{
  "details": {
    "capacity": ["Expected number, received string"]
  }
}
```

**Why this output:** `z.number()` rejects the string `"50"` because no coercion is applied. The error message is descriptive and machine-readable. When the client sends a valid number, the schema passes and the data is used.

### Real-World Cases

- **API payloads:** Ensuring `price` is a number and `inStock` is a boolean.
- **Form submissions:** Validating that `age` is a number and `agreeToTerms` is a boolean.
- **Configuration endpoints:** Validating nested configuration objects.
- **Bulk operations:** Validating arrays of items with consistent types.

---

## Core Concept 5: Required vs. Optional Fields

### Definitions

**Core Definition:** Required fields must be present in the request; optional fields may be omitted. Validators must distinguish between a missing field and a field explicitly set to `null` or an empty string.

**Technical Definition:** In Zod, fields are required by default. Adding `.optional()` makes a field optional — it may be `undefined` or absent. Adding `.nullable()` allows `null` as a value. Adding `.default(value)` provides a fallback when the field is absent. This distinction is critical: `{ "name": null }` is different from `{}` (missing name), and validators should handle each case appropriately.

**Beginner-Friendly Explanation:** Required vs. optional fields are like a job application. Your name and email are required — you cannot submit without them. Your middle name is optional — you can leave it blank. But there's a difference between leaving it blank (missing) and writing "N/A" (explicitly empty). Validators should treat these differently.

### Purposes

- To differentiate missing fields from explicitly passed null or empty values.
- To enforce that critical fields are always present.
- To allow flexible payloads where some fields are genuinely optional.
- To provide defaults for fields that are omitted.

### Syntax Rules and Structure

```typescript
const UserSchema = z.object({
  name: z.string().min(1),                    // Required
  email: z.string().email(),                  // Required
  phone: z.string().optional(),               // Optional (may be absent or undefined)
  bio: z.string().nullable().optional(),      // Optional AND nullable
  role: z.enum(['user', 'admin']).default('user')  // Default if absent
});
```

| Modifier | Behaviour |
|----------|-----------|
| (none) | Required — must be present. |
| `.optional()` | May be `undefined` or absent. |
| `.nullable()` | May be `null`. |
| `.default(value)` | Uses `value` if absent. |

**Rules:**
- Required fields fail validation if absent, `null`, or `undefined`.
- `.optional()` allows `undefined` but not `null` unless combined with `.nullable()`.
- `.default()` provides a value when the field is absent (but not when `null`).
- Use `.nullish()` as shorthand for `.nullable().optional()`.

**Constraints:**
- A required string field with `.min(1)` rejects empty strings.
- A required field with `null` fails unless `.nullable()` is added.

### Annotated Code Example

```typescript
const ProfileSchema = z.object({
  username: z.string().min(3).max(30),
  email: z.string().email(),
  displayName: z.string().optional(),
  avatarUrl: z.string().url().nullable().optional(),
  bio: z.string().max(500).default(''),
  role: z.enum(['user', 'editor', 'admin']).default('user')
});
```

**Expected Output (for `{ "username": "alice", "email": "alice@example.com" }`):**
```json
{
  "username": "alice",
  "email": "alice@example.com",
  "displayName": undefined,
  "avatarUrl": undefined,
  "bio": "",
  "role": "user"
}
```

**Expected Output (for `{ "username": "alice", "email": "alice@example.com", "avatarUrl": null }`):**
```json
{
  "username": "alice",
  "email": "alice@example.com",
  "bio": "",
  "role": "user",
  "avatarUrl": null
}
```

**Expected Output (for `{ "username": "alice" }` — missing email):**
```json
{
  "details": { "email": ["Required"] }
}
```

**Why this output:** `username` and `email` are required. `displayName` and `avatarUrl` are optional. `bio` defaults to an empty string. `role` defaults to `"user"`. The missing email produces a "Required" error.

### Real-World Cases

- **User profiles:** Name and email required; phone and bio optional.
- **Product listings:** Title and price required; description optional.
- **Search filters:** All filters optional; pagination defaults applied.
- **PATCH updates:** Only the fields being updated are required; others are omitted.

---

## Core Concept 6: Length & Boundary Constraints

### Definitions

**Core Definition:** Length and boundary constraints restrict text string sizing, array bounds, and match specialized formats (emails, phone numbers, URLs) using regex patterns and built-in validators.

**Technical Definition:** Zod provides `.min(n)`, `.max(n)`, and `.length(n)` for strings and arrays. Joi provides the same methods with chainable syntax. For format validation, Zod offers `.email()`, `.url()`, `.uuid()`, and `.regex(pattern)`. Joi offers `.email()`, `.pattern(regex)`, and `.length(n)`. These constraints prevent oversized payloads, enforce business rules (e.g., password length), and validate specialized formats.

**Beginner-Friendly Explanation:** Length and boundary constraints are like the rules on a form: "Your name must be between 2 and 50 characters" and "Your phone number must be exactly 10 digits." They ensure that the data fits within acceptable limits and follows the expected format.

### Purposes

- To restrict text string sizing (minimum, maximum, exact length).
- To restrict array bounds (minimum and maximum number of items).
- To match specialized formats (emails, phone numbers, URLs, UUIDs) via regex.
- To prevent buffer overflows and resource exhaustion from oversized inputs.

### Syntax Rules and Structure

```typescript
const SignupSchema = z.object({
  username: z.string().min(3).max(30).regex(/^[a-zA-Z0-9_]+$/, 'Only alphanumeric and underscores'),
  email: z.string().email().max(320),
  phone: z.string().regex(/^\+?[0-9]{7,15}$/, 'Invalid phone number'),
  password: z.string().min(8).max(128),
  tags: z.array(z.string().max(50)).max(10).default([])
});
```

| Constraint | Applies To | Description |
|-----------|-----------|-------------|
| `.min(n)` | String, Array | Minimum length or size. |
| `.max(n)` | String, Array | Maximum length or size. |
| `.length(n)` | String, Array | Exact length or size. |
| `.email()` | String | Valid email format. |
| `.regex(pattern)` | String | Custom regular expression. |

**Joi equivalent:**
```js
const schema = Joi.object({
  username: Joi.string().alphanum().min(3).max(30).required(),
  email: Joi.string().email().required(),
  phone: Joi.string().pattern(/^\+?[0-9]{7,15}$/),
  password: Joi.string().min(8).max(128).required()
});
```

**Rules:**
- Always set an upper bound on string lengths (e.g., `.max(255)` for names).
- Use `.regex()` for specialized formats not covered by built-in validators.
- Array bounds (`.max(10)`) prevent abuse via oversized arrays.
- Email validation should use `.email()` rather than a custom regex.

**Constraints:**
- Regex patterns must be carefully tested — catastrophic backtracking can cause ReDoS.
- `.length(n)` requires an exact match, which is rarely appropriate for names.

### Annotated Code Example

```typescript
const ArticleSchema = z.object({
  title: z.string().min(5, 'Title too short').max(200, 'Title too long'),
  body: z.string().min(50, 'Body too short').max(50000, 'Body too long'),
  slug: z.string().regex(/^[a-z0-9-]+$/, 'Slug must be lowercase alphanumeric with hyphens'),
  tags: z.array(z.string().min(1).max(30)).min(1).max(5).default([]),
  authorEmail: z.string().email().max(320)
});
```

**Expected Output (for valid article):**
```json
{
  "title": "Input Validation in Express",
  "body": "...",
  "slug": "input-validation-express",
  "tags": ["node", "express"],
  "authorEmail": "alice@example.com"
}
```

**Expected Output (for invalid title — too short):**
```json
{
  "details": { "title": ["Title too short"] }
}
```

**Expected Output (for invalid tags — too many):**
```json
{
  "details": { "tags": ["Array must contain at most 5 element(s)"] }
}
```

**Why this output:** The title is validated against a minimum and maximum length. The tags array is restricted to between 1 and 5 items, each with a maximum length of 30 characters. Errors are descriptive and field-specific.

### Real-World Cases

- **User registration:** Password length (8–128), username length (3–30).
- **Content platforms:** Article title (5–200), body (50–50,000), tags (1–5).
- **E-commerce:** Product name (1–100), description (0–5,000), SKU pattern.
- **Contact forms:** Email format, phone number pattern, message length.

---

## Core Concept 7: Range Constraints

### Definitions

**Core Definition:** Range constraints enforce upper and lower mathematical boundaries on numeric fields, rejecting values that fall outside the acceptable range.

**Technical Definition:** Zod provides six methods for constraining numeric ranges: `.gt(n)` (greater than), `.gte(n)` (greater than or equal), `.lt(n)` (less than), `.lte(n)` (less than or equal), with aliases `.min()` and `.max()`. For bigint values, the same methods apply with bigint arguments. Integer constraints use `.int()` to reject decimal values.

**Beginner-Friendly Explanation:** Range constraints are like the rules on a thermostat: "Temperature must be between 60°F and 80°F." Values outside that range are rejected. In API validation, range constraints ensure that ages, prices, quantities, and other numeric fields stay within sensible limits.

### Purposes

- To handle upper and lower mathematical boundaries for numeric fields.
- To enforce business rules (e.g., age ≥ 18, price > 0).
- To prevent invalid data from reaching calculations or database queries.
- To provide clear error messages when values fall outside the acceptable range.

### Syntax Rules and Structure

```typescript
const ProductSchema = z.object({
  price: z.number().positive().max(100000, 'Price cannot exceed $100,000'),
  quantity: z.number().int().min(1).max(1000),
  discount: z.number().min(0).max(100).default(0),
  rating: z.number().min(1).max(5).optional()
});
```

| Method | Alias | Meaning |
|--------|-------|---------|
| `.gt(n)` | — | Greater than `n`. |
| `.gte(n)` | `.min(n)` | Greater than or equal to `n`. |
| `.lt(n)` | — | Less than `n`. |
| `.lte(n)` | `.max(n)` | Less than or equal to `n`. |
| `.int()` | — | Integer only. |

**Rules:**
- Use `.positive()` for values that must be > 0.
- Use `.nonnegative()` for values that must be ≥ 0.
- Combine `.int()` with `.min()` and `.max()` for integer ranges.
- Range constraints can be applied to any numeric schema, including coerced query parameters.

**Constraints:**
- Zod v4 does not accept `Infinity` by default — all numbers must be finite.
- BigInt values use `5n` syntax for range arguments.

### Annotated Code Example

```typescript
const EventSchema = z.object({
  capacity: z.number().int().min(1).max(10000),
  ticketPrice: z.number().positive().max(5000),
  ageRestriction: z.number().int().min(0).max(21).default(0),
  duration: z.number().positive().max(1440)  // minutes, max 24 hours
});
```

**Expected Output (for valid event):**
```json
{
  "capacity": 100,
  "ticketPrice": 49.99,
  "ageRestriction": 18,
  "duration": 120
}
```

**Expected Output (for invalid capacity — zero):**
```json
{
  "details": { "capacity": ["Number must be greater than or equal to 1"] }
}
```

**Expected Output (for invalid ticketPrice — negative):**
```json
{
  "details": { "ticketPrice": ["Number must be greater than 0"] }
}
```

**Why this output:** The capacity must be an integer between 1 and 10,000. The ticket price must be positive and no more than $5,000. The age restriction defaults to 0 but can be up to 21. The duration must be positive and no more than 1,440 minutes (24 hours).

### Real-World Cases

- **E-commerce:** Price > 0, quantity ≥ 1, discount between 0 and 100.
- **Event management:** Capacity between 1 and 10,000, age restriction 0–21.
- **Analytics:** Page numbers ≥ 1, limits between 1 and 100.
- **Healthcare:** Age between 0 and 150, dosage between 0.1 and 1000 mg.

---

## Core Concept 8: "Parse, Don't Validate" Pattern

### Definitions

**Core Definition:** The "Parse, Don't Validate" pattern transforms external, untrusted data into a trusted, strongly typed internal representation at the system boundary, rather than scattering ad-hoc boolean checks throughout the codebase.

**Technical Definition:** The core idea is simple: instead of writing validation functions that return booleans and then casting the data with `as`, you parse external data once at the boundary and produce a new value with a guaranteed type. Zod's `.transform()` and `.pipe()` methods enable this pattern. A schema can parse a string into a `Date`, trim and lowercase an email, or convert a numeric string into an integer. The result is a value whose type accurately reflects its runtime structure.

**Beginner-Friendly Explanation:** Traditional validation is like checking a box on a form and then hoping everything inside is correct. "Parse, Don't Validate" is like opening the box, taking out each item, inspecting it, and placing it in a new, labelled container. You never have to check the box again because you know exactly what's inside and what type it is.

### Purposes

- To transform and cast raw input safely into trusted, typed runtime structures.
- To eliminate the gap between validation (boolean check) and type assertion (`as`).
- To move validation logic from scattered checks to a single schema at the boundary.
- To produce values whose TypeScript types match their runtime structure.

### Syntax Rules and Structure

```typescript
// ❌ TRADITIONAL: Validate then cast
function isUser(data: unknown): boolean {
  return typeof data === 'object' && data !== null &&
         typeof (data as any).name === 'string' &&
         typeof (data as any).email === 'string';
}
const user = raw as { name: string; email: string }; // Unsafe cast

// ✅ PARSE: Transform at the boundary
const UserSchema = z.object({
  name: z.string(),
  email: z.string().email(),
  createdAt: z.string().transform((s) => new Date(s))
});
type User = z.output<typeof UserSchema>;

const user = UserSchema.parse(raw); // Type-safe, runtime-verified
```

| Approach | Type Safety | Runtime Safety | Transformation |
|----------|-------------|----------------|----------------|
| Boolean validation + `as` | Compile-time only (unsafe) | Manual checks | None |
| Parse with Zod | Compile-time + runtime | Guaranteed | Yes |

**Rules:**
- Parse external data (API responses, request bodies, localStorage) at the boundary.
- Use `.transform()` to convert types (string → Date, string → number).
- Use `.pipe()` to chain transformations with validation.
- The parsed result has a type derived from the schema (`z.output<typeof Schema>`).

**Constraints:**
- Transformations should be pure and synchronous where possible.
- Async transforms require `.parseAsync()`.
- Parsing is more expensive than boolean checks — apply at boundaries only, not in hot loops.

### Annotated Code Example

```typescript
// Schema with transformation
const EventSchema = z.object({
  id: z.string().uuid(),
  title: z.string().min(1),
  startDate: z.string().iso.datetime().transform((s) => new Date(s)),
  endDate: z.string().iso.datetime().transform((s) => new Date(s)),
  isAllDay: z.coerce.boolean().default(false)
});

// Cross-field validation on transformed values
const ValidEventSchema = EventSchema.refine(
  (event) => event.endDate >= event.startDate,
  { message: 'End date must be after start date', path: ['endDate'] }
);

// Parse at the boundary
function parseEvent(raw: unknown) {
  const result = ValidEventSchema.safeParse(raw);
  if (!result.success) {
    throw new ValidationError(result.error.flatten().fieldErrors);
  }
  return result.data; // Type: { id: string; title: string; startDate: Date; endDate: Date; isAllDay: boolean }
}
```

**Expected Output (for valid raw data):**
```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "title": "Node.js Meetup",
  "startDate": "2026-02-15T18:00:00.000Z",
  "endDate": "2026-02-15T20:00:00.000Z",
  "isAllDay": false
}
```

**Expected Output (for invalid date range):**
```json
{
  "endDate": ["End date must be after start date"]
}
```

**Why this output:** The schema parses the ISO date strings into `Date` objects during validation. The `refine()` check then compares the transformed `Date` values. The result is a typed object where `startDate` and `endDate` are `Date` instances, not strings. Downstream code can call `event.startDate.getTime()` without any type assertion.

### Real-World Cases

- **API boundaries:** Parsing incoming JSON bodies into domain objects with correct types.
- **Configuration:** Parsing environment variable strings into typed config objects.
- **Form submissions:** Converting string inputs into numbers, dates, and booleans.
- **Third-party APIs:** Parsing external API responses into internal domain types.

---

## References

- Zod Documentation — https://zod.dev
- Joi Documentation — https://joi.dev
- express-validator Documentation — https://express-validator.github.io
- Shattered.io — Validazione Input in Node.js: Zod, Joi, express-validator [2026] — https://shattered.io/it/validazione-input-nodejs/
- DEV Community — Parse, Don't Validate in TypeScript: A Practical Tutorial — https://dev.to/paradane/parse-dont-validate-in-typescript-a-practical-tutorial-1ahc
- Express-validator — Schema Validation — https://express-validator.github.io/docs/6.9.0/schema-validation/
- Zod — Schemas — https://mintlify.wiki/colinhacks/zod/concepts/schemas
- Safeguard.sh — Express Node.js Security Hardening Guide — https://safeguard.sh/resources/blog/express-nodejs-security-hardening
- Zod — Transform & Coercion Examples — https://github.com/agents-inc/skills/blob/main/dist/plugins/web-forms-zod-validation/skills/web-forms-zod-validation/examples/transforms.md
- Zod — Number Schemas — https://mintlify.wiki/colinhacks/zod/concepts/schemas
- express-validator — param() API — https://express-validator.github.io/docs/api/check/#param
- express-validator — matchedData() — https://express-validator.github.io/docs/api/matched-data/
- MongoDB ObjectId Validation — https://www.npmjs.com/package/valid-oid
- OWASP — SQL Injection Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- OWASP — Top Ten 2021: A03 Injection — https://owasp.org/Top10/A03_2021-Injection/
- Node.js — `querystring` Module — https://nodejs.org/api/querystring.html
- RFC 4122 — A Universally Unique IDentifier (UUID) URN Namespace — https://datatracker.ietf.org/doc/html/rfc4122