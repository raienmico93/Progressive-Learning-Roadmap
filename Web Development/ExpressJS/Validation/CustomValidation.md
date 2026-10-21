# Express.js Custom Validation — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Custom validation in Express.js is the practice of writing application-specific validation logic — beyond what built-in validators and third-party libraries provide — and integrating it into the request pipeline as reusable middleware that rejects invalid data before it reaches route handlers.

**Technical Definition**
Custom validation encompasses three distinct but interrelated concerns. First, **custom validators** are user-defined functions that check application-specific rules (database uniqueness, cross-field consistency, business constraints) and are attached to validation chains via `.custom()` (express-validator) or `.refine()`/`.superRefine()` (Zod). Second, **validation middleware** is the Express middleware layer — typically a factory function that closes over a schema or rule set and returns a `(req, res, next)` handler — that executes validation at the request boundary. Third, **validation errors** are the structured `400` or `422` responses produced when validation fails, containing field-level details that clients can map to form inputs. Best practices tie these together: validate at the boundary, aggregate all errors, use safe parsing, sanitize alongside validation, and centralize error formatting.

**Beginner-Friendly Explanation**
Built-in validators can check whether an email looks like an email or a password is long enough, but they cannot know that “this username is already taken” or “this end date must be after the start date.” Custom validation is how you teach your app your own rules. You write a function that performs the check, wrap it in middleware so it runs before your route handler, and return a clear error if the data fails — so bad data never reaches your database or business logic.

---

### Key Characteristics

- **Application-specific logic** — Custom validators handle rules that no generic library can know: database uniqueness, cross-field dependencies, and domain constraints.
- **Middleware-integrated** — Custom validation runs as Express middleware, before route handlers, using the `(req, res, next)` signature and factory functions for configurability.
- **Async-capable** — Custom validators may return Promises to perform asynchronous checks such as database lookups or external API calls.
- **Structured error output** — Validation failures produce field-level error details (field name, message, location) rather than vague “Validation failed” responses.
- **Factory-based reuse** — Reusable validation middleware is implemented as a higher-order function that accepts a schema or rule set and returns a configured middleware.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- The `express-validator` package installed (or Zod/Joi for schema-based approaches).
- Understanding of Express middleware and the `(req, res, next)` signature.
- Familiarity with `req.body`, `req.query`, `req.params`, and asynchronous JavaScript (Promises, `async`/`await`).

---

### Related Programming Areas

- **Validation libraries** — `express-validator`, Zod, and Joi provide the scaffolding on which custom validators are built.
- **Error handling** — Custom validation errors integrate with centralized error-handling middleware.
- **Sanitization** — Custom sanitizers complement custom validators by transforming data.
- **Security** — Custom validation prevents injection, mass assignment, and business-logic abuse.
- **API design** — Validation error contracts are part of the public API surface.

---

## Core Concept 1: Custom Validators

### Definitions

**Core Definition**
A custom validator is a user-defined function that checks an application-specific rule against a field value and signals validity or invalidity to the validation framework.

**Technical Definition**
In `express-validator`, a custom validator is attached via `.custom(validator)`. The validator function receives `(value, { req, location, path })` and must return a truthy value for valid data, or a falsy value, throw, or reject a Promise for invalid data. Custom validators may be asynchronous: returning a Promise that resolves indicates validity, while rejection indicates invalidity. In Zod, custom validation is expressed with `.refine(check, params?)` for single-issue checks and `.superRefine((val, ctx) => ...)` for multi-issue checks, where `ctx.addIssue()` registers failures. Custom validators are the mechanism for rules that built-in validators cannot express: database uniqueness, password confirmation, date-range ordering, and conditional business constraints.

**Beginner-Friendly Explanation**
A custom validator is your own rule-checking function. You write the logic — “is this email already in the database?” or “do these two passwords match?” — and plug it into the validation chain. If the rule passes, the request continues; if it fails, you provide a clear error message.

---

### Purposes

- To enforce application-specific business rules that built-in validators cannot express.
- To perform asynchronous checks such as database uniqueness lookups.
- To validate cross-field relationships (password confirmation, date ranges, dependent fields).
- To produce custom, user-friendly error messages for domain-specific failures.

---

### Syntax Rules and Structure

#### Complete General Syntax: `express-validator` Custom Validator

```javascript
body('username').custom(async (value, { req, location, path }) => {
  const existing = await User.findByUsername(value);
  if (existing) {
    throw new Error('Username is already taken');
  }
  return true;
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `.custom(validator)` | Attaches the custom validator to the chain |
| `value` | The field value being validated |
| `{ req, location, path }` | Context: the request, the request location (`body`, `query`, etc.), and the field path |
| `throw new Error(...)` | Signals invalidity with a custom error message |
| `return true` | Signals validity (required for synchronous validators) |

#### Complete General Syntax: Zod Custom Validator (`.refine` and `.superRefine`)

```javascript
const schema = z.object({
  password: z.string().min(8),
  confirm: z.string()
}).refine((data) => data.password === data.confirm, {
  message: "Passwords don't match",
  path: ["confirm"]
});
```

```javascript
const schema = z.array(z.string()).superRefine((val, ctx) => {
  if (val.length > 3) {
    ctx.addIssue({ code: z.ZodIssueCode.too_big, maximum: 3, type: 'array', inclusive: true, message: 'Too many items' });
  }
  if (val.length !== new Set(val).size) {
    ctx.addIssue({ code: z.ZodIssueCode.custom, message: 'No duplicates allowed' });
  }
});
```

**Component Breakdown**

| Method | Role |
|--------|------|
| `.refine(check, params?)` | Single custom check; returns truthy to pass |
| `.superRefine((val, ctx) => ...)` | Multi-issue check; uses `ctx.addIssue()` |
| `path` | Attaches the error to a specific field |

#### Syntax Rules

- **`express-validator`**: synchronous custom validators must return a truthy value; asynchronous ones must resolve (not reject) to pass. Throwing or rejecting uses the thrown value as the error message.
- **Zod**: `.refine()` returns a boolean; `.superRefine()` must call `ctx.addIssue()` to register failures. Async refinements require `.parseAsync()`.
- Custom validators should not perform side effects; they are checks, not mutations.

#### Constraints and Limitations

- **`express-validator`**: `.custom()` runs in the order it appears in the chain; placing it after `.bail()` will skip it if an earlier validator failed.
- **Zod**: `.refine()` can only report one issue; use `.superRefine()` for multiple.
- Async validators add latency; database lookups should be efficient and indexed.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Async Custom Validator for Email Uniqueness (express-validator)

```javascript
const express = require('express');
const { body, validationResult } = require('express-validator');
const app = express();
app.use(express.json());

app.post('/register',
  body('email').isEmail().custom(async (value) => {
    const existing = await User.findByEmail(value);
    if (existing) {
      throw new Error('This email is already registered');
    }
    return true;
  }),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    res.status(201).json({ message: 'Registered' });
  }
);

app.listen(3000);
```

**Step-by-Step Explanation**

1. **Chain composition** — `body('email')` creates a chain; `.isEmail()` validates format; `.custom()` adds the async uniqueness check.
2. **Async execution** — The custom validator returns a Promise; `express-validator` awaits it.
3. **Error signaling** — Throwing `new Error('...')` marks the field invalid with that message.

**Expected Output**

```
POST /register  { "email": "new@example.com" }
→ 201 { "message": "Registered" }

POST /register  { "email": "taken@example.com" }
→ 400 { "errors": [{ "path": "email", "msg": "This email is already registered" }] }
```

**Why This Output Occurs**

- The async custom validator queries the database; an existing user causes a thrown error that `validationResult` collects into the `400` response.

---

#### Example 2: Cross-Field Validation (express-validator)

```javascript
app.post('/change-password',
  body('newPassword').isLength({ min: 8 }),
  body('confirmPassword').custom((value, { req }) => {
    if (value !== req.body.newPassword) {
      throw new Error('Password confirmation does not match');
    }
    return true;
  }),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    res.json({ message: 'Password updated' });
  }
);
```

**Expected Output**

```
POST /change-password  { "newPassword": "secret123", "confirmPassword": "different" }
→ 400 { "errors": [{ "path": "confirmPassword", "msg": "Password confirmation does not match" }] }
```

**Why This Output Occurs**

- The custom validator accesses `req.body.newPassword` through the context object and compares it with the field value.

---

#### Example 3: Multi-Issue Custom Validation (Zod `.superRefine`)

```javascript
const schema = z.object({
  items: z.array(z.string()).superRefine((val, ctx) => {
    if (val.length > 3) {
      ctx.addIssue({ code: z.ZodIssueCode.too_big, maximum: 3, type: 'array', inclusive: true, message: 'Too many items' });
    }
    if (val.length !== new Set(val).size) {
      ctx.addIssue({ code: z.ZodIssueCode.custom, message: 'No duplicates allowed' });
    }
  })
});

const result = schema.safeParse({ items: ['a', 'a', 'b', 'c', 'd'] });
// result.error.issues → 2 issues
```

**Expected Output**

```
2 issues: ['Too many items', 'No duplicates allowed']
```

**Why This Output Occurs**

- Both conditions fail, so `.superRefine()` registers two issues in a single validation pass.

---

### Real-World Cases

- **User registration** — Checking email and username uniqueness against the database.
- **Password management** — Ensuring confirmation fields match before hashing.
- **Booking systems** — Verifying that an end date is after a start date and that a resource is available.
- **Inventory** — Confirming that a requested quantity does not exceed stock.

---

## Core Concept 2: Validation Middleware

### Definitions

**Core Definition**
Validation middleware is an Express middleware function — typically produced by a factory that closes over a schema or rule set — that runs validation logic at the request boundary before the route handler executes.

**Technical Definition**
Validation middleware follows the `(req, res, next)` signature. The factory pattern is standard: `validate(schema)` returns a middleware function that captures the schema in its closure and runs it against `req.body` (or `req.query`/`req.params`) on every request. On success, the middleware typically replaces `req.body` with the validated/sanitized value and calls `next()`. On failure, it short-circuits with a `400` or `422` response containing structured error details. This separation keeps route handlers free of validation noise and makes validation logic reusable across endpoints.

**Beginner-Friendly Explanation**
Validation middleware is the “checkpoint” your requests pass through before reaching your route handler. You write a factory that takes a schema and returns a checkpoint function. Every route that needs validation calls the factory with its schema, and the resulting middleware runs the check automatically. This keeps your handlers clean and your validation consistent.

---

### Purposes

- To enforce validation at the request boundary, before business logic or database access.
- To eliminate repetitive validation code from individual route handlers.
- To apply the same validation rules consistently across multiple endpoints.
- To attach validated, sanitized data to the request for downstream use.

---

### Syntax Rules and Structure

#### Complete General Syntax: Validation Middleware Factory (Zod)

```typescript
import { z } from 'zod';
import type { Request, Response, NextFunction } from 'express';

function validate<T extends z.ZodTypeAny>(schema: T) {
  return (req: Request, res: Response, next: NextFunction): void => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      const errors = result.error.issues.map((issue) => ({
        field: issue.path.join('.'),
        message: issue.message
      }));
      res.status(400).json({ success: false, errors });
      return;
    }
    req.body = result.data; // typed as z.infer<T>
    next();
  };
}
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `validate(schema)` | Factory that closes over the schema (runs once at setup) |
| `(req, res, next)` | The middleware returned by the factory (runs per request) |
| `schema.safeParse(req.body)` | Validates without throwing |
| `result.data` | Validated, typed, sanitized data |
| `res.status(400)` | Short-circuits with a client error |
| `return` | Ensures the handler does not run after a validation failure |

#### Complete General Syntax: Validation Middleware Factory (Joi)

```javascript
function validate(schema) {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, {
      abortEarly: false,
      stripUnknown: true
    });
    if (error) {
      const details = error.details.map((e) => ({
        field: e.path.join('.'),
        issue: e.message
      }));
      return res.status(422).json({
        error: { code: 'VALIDATION_ERROR', message: 'Invalid input', details }
      });
    }
    req.body = value;
    next();
  };
}
```

#### Syntax Rules

- The factory runs **once** at application startup; the returned middleware runs **per request**.
- Use `safeParse()` (Zod) or `{ error, value }` (Joi) instead of throwing validators to avoid crashing the handler.
- Replace `req.body` with the validated value so downstream code uses sanitized, typed data.
- Return after sending the error response; otherwise `next()` may still be called.

#### Constraints and Limitations

- The factory pattern is necessary because Express middleware only receives `(req, res, next)` — the schema must be closed over.
- Validating only `req.body` misses query parameters and route parameters; extend the factory to validate all sources as needed.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Reusable Zod Validation Middleware

```typescript
import express from 'express';
import { z } from 'zod';

const app = express();
app.use(express.json());

const CreateUserSchema = z.object({
  username: z.string().min(3).max(20),
  email: z.email(),
  password: z.string().min(8)
});

function validate<T extends z.ZodTypeAny>(schema: T) {
  return (req: express.Request, res: express.Response, next: express.NextFunction) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      const errors = result.error.issues.map((i) => ({
        field: i.path.join('.'),
        message: i.message
      }));
      return res.status(400).json({ success: false, errors });
    }
    req.body = result.data;
    next();
  };
}

app.post('/users', validate(CreateUserSchema), (req, res) => {
  res.status(201).json({ user: req.body });
});
```

**Expected Output**

```
POST /users  { "username": "alice", "email": "alice@example.com", "password": "secret123" }
→ 201 { "user": { "username": "alice", "email": "alice@example.com", "password": "secret123" } }

POST /users  { "username": "a", "email": "invalid", "password": "123" }
→ 400 { "success": false, "errors": [
  { "field": "username", "message": "Too small: expected string to have >=3 characters" },
  { "field": "email", "message": "Invalid email address" },
  { "field": "password", "message": "Too small: expected string to have >=8 characters" }
] }
```

**Why This Output Occurs**

- The factory closes over `CreateUserSchema`; `safeParse` validates the body, and on failure the middleware returns field-level errors without invoking the handler.

---

#### Example 2: Joi Validation Middleware with `abortEarly: false`

```javascript
const Joi = require('joi');

const schema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required()
});

function validate(schema) {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, {
      abortEarly: false,
      stripUnknown: true
    });
    if (error) {
      return res.status(422).json({
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Invalid input',
          details: error.details.map((d) => ({ field: d.path.join('.'), issue: d.message }))
        }
      });
    }
    req.body = value;
    next();
  };
}

app.post('/login', validate(schema), (req, res) => res.json({ ok: true }));
```

**Expected Output**

```
POST /login  { "email": "invalid", "password": "123" }
→ 422 { "error": { "code": "VALIDATION_ERROR", "message": "Invalid input", "details": [
  { "field": "email", "issue": "\"email\" must be a valid email" },
  { "field": "password", "issue": "\"password\" length must be at least 8 characters long" }
] } }
```

**Why This Output Occurs**

- `abortEarly: false` collects all validation failures; `stripUnknown: true` removes extra fields before the handler runs.

---

### Real-World Cases

- **REST APIs** — A single validation middleware factory applied to every route that accepts a request body.
- **Form handling** — Reusable middleware for login, registration, and profile update schemas.
- **Multi-tenant SaaS** — Validation middleware that enforces tenant-specific schemas before data access.

---

## Core Concept 3: Validation Errors

### Definitions

**Core Definition**
Validation errors are the structured client-error responses (`400` or `422`) produced when request data fails validation, containing field-level details that identify which fields failed and why.

**Technical Definition**
When validation middleware detects invalid data, it responds with an HTTP `400 Bad Request` or `422 Unprocessable Entity` status and a JSON body. A well-formed validation error response includes: an error `code` (e.g., `VALIDATION_ERROR`), a human-readable `message`, and a `details` array where each entry contains the `field` (path), the validation `message`, the offending `value`, the request `location` (`body`, `query`, `params`), and optionally constraint context such as `minLength`. `express-validator`'s `validationResult(req).array()` produces an array of error objects with `path`, `msg`, `location`, `value`, and `type` properties. Zod's `error.issues` array contains `path`, `message`, and `code`. Vague responses such as `{ "error": "Validation failed" }` without field association are considered unhelpful and should be avoided.

**Beginner-Friendly Explanation**
When validation fails, your app needs to tell the client exactly what went wrong and where. Instead of a vague “Validation failed,” a good validation error response says: “The `email` field is invalid: must be a valid email address. The `password` field is too short: must be at least 8 characters.” This lets the frontend highlight the right form fields.

---

### Purposes

- To provide clients with actionable, field-level feedback for correcting invalid input.
- To enable frontend frameworks to map errors to specific form fields.
- To distinguish validation failures from authentication, authorization, and server errors.
- To maintain a consistent error contract across the API.

---

### Syntax Rules and Structure

#### Complete General Syntax: Structured Validation Error Response

```javascript
{
  error: {
    code: 'VALIDATION_ERROR',
    message: 'One or more fields have invalid values',
    details: [
      {
        field: 'email',
        message: 'Must be a valid email address',
        value: 'not-an-email',
        location: 'body'
      },
      {
        field: 'password',
        message: 'Password must be at least 8 characters',
        value: 'short',
        location: 'body',
        constraints: { minLength: 8, actualLength: 5 }
      }
    ]
  }
}
```

**Component Breakdown**

| Field | Role |
|-------|------|
| `code` | Stable, machine-readable error identifier |
| `message` | Human-readable summary |
| `details[].field` | The request field that failed |
| `details[].message` | Why it failed |
| `details[].value` | The offending value (use with caution) |
| `details[].location` | Where the field was found (`body`, `query`, `params`) |
| `details[].constraints` | Optional constraint context (min length, allowed range) |

#### Syntax Rules

- **Always include field-level details**; a bare “Validation failed” is insufficient.
- Use `400 Bad Request` for malformed or semantically invalid input; `422 Unprocessable Entity` is also used for semantic validation failures depending on API convention.
- Do not expose stack traces or internal implementation details in validation errors.
- Keep the error envelope consistent across all endpoints.

#### Constraints and Limitations

- Including the offending `value` can be useful for debugging but may echo sensitive data; omit it or mask it for password fields.
- `422` vs. `400` is a matter of API convention; pick one and apply it consistently.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Field-Level Errors from `express-validator`

```javascript
const { body, validationResult } = require('express-validator');

app.post('/users', [
  body('email').notEmpty().withMessage('Email is required').isEmail().withMessage('Must be a valid email address'),
  body('password').isLength({ min: 8 }).withMessage('Password must be at least 8 characters'),
  body('username').isLength({ min: 3, max: 30 }).withMessage('Username must be 3-30 characters')
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'One or more fields have invalid values',
        details: errors.array().map((err) => ({
          field: err.path,
          message: err.msg,
          value: err.value,
          location: err.location
        }))
      }
    });
  }
  res.status(201).json({ created: true });
});
```

**Expected Output**

```
POST /users  { "email": "not-an-email", "password": "short", "username": "ab" }
→ 400 {
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "One or more fields have invalid values",
    "details": [
      { "field": "email", "message": "Must be a valid email address", "value": "not-an-email", "location": "body" },
      { "field": "password", "message": "Password must be at least 8 characters", "value": "short", "location": "body" },
      { "field": "username", "message": "Username must be 3-30 characters", "value": "ab", "location": "body" }
    ]
  }
}
```

**Why This Output Occurs**

- `validationResult(req).array()` produces one error object per failed field; the handler maps them into the `details` array with field-level context.

---

#### Example 2: Formatted Errors from Zod `flattenError`

```typescript
import { z } from 'zod';

const schema = z.object({
  email: z.email({ error: 'Invalid email format' }),
  password: z.string().min(8, { error: 'Password must be at least 8 characters' })
});

const result = schema.safeParse({ email: 'invalid', password: '123' });

if (!result.success) {
  const fieldErrors = z.flattenError(result.error).fieldErrors;
  // { email: ['Invalid email format'], password: ['Password must be at least 8 characters'] }
  res.status(400).json({ error: { code: 'VALIDATION_ERROR', details: fieldErrors } });
}
```

**Expected Output**

```
{
  "error": {
    "code": "VALIDATION_ERROR",
    "details": {
      "email": ["Invalid email format"],
      "password": ["Password must be at least 8 characters"]
    }
  }
}
```

**Why This Output Occurs**

- `z.flattenError()` converts the issue array into a form-friendly object keyed by field name.

---

### Real-World Cases

- **Form validation** — Mapping field-level errors to the corresponding inputs.
- **API error contracts** — Stable `code` values and `details` arrays for client SDKs.
- **Multi-step forms** — Identifying which step contains invalid fields.

---

## Core Concept 4: Validation Best Practices

### Definitions

**Core Definition**
Validation best practices are the design, security, and maintainability patterns that keep custom validation reliable, comprehensive, and safe across an Express application.

**Technical Definition**
Best practices include: validating at the request boundary as middleware before business logic; using factory functions for reusable, configurable validation middleware; aggregating all validation errors rather than stopping at the first; using safe parsing (`.safeParse()`, `{ error, value }`) to avoid uncaught throws; sanitizing input alongside validation; stripping unknown fields to prevent mass assignment; attaching validated data to the request for downstream use; and returning consistent, field-level error responses with appropriate `400`/`422` status codes. Validation should be paired with database constraints (unique indexes, foreign keys) for defense in depth.

**Beginner-Friendly Explanation**
Best practices are the habits that keep validation from becoming a source of bugs. Validate everything at the door, collect all errors at once, strip out extra fields attackers might inject, and always return clear field-level errors. Use factories so you write the middleware once and reuse it everywhere.

---

### Purposes

- To ensure comprehensive coverage of all input sources and edge cases.
- To prevent security vulnerabilities such as mass assignment and injection.
- To keep validation logic maintainable and reusable through middleware factories.
- To provide consistent, user-friendly error responses across the API.
- To avoid crashes caused by throwing validators in Express handlers.

---

### Syntax Rules and Structure

#### Complete General Syntax: Best-Practice Validation Pipeline

```typescript
import { z } from 'zod';

// 1. Validation middleware factory
function validate<T extends z.ZodTypeAny>(schema: T) {
  return (req: express.Request, res: express.Response, next: express.NextFunction) => {
    const result = schema.safeParse({
      body: req.body,
      query: req.query,
      params: req.params
    });

    if (!result.success) {
      const errors = result.error.issues.map((issue) => ({
        field: issue.path.join('.'),
        message: issue.message
      }));
      return res.status(422).json({
        error: { code: 'VALIDATION_ERROR', message: 'Invalid input', details: errors }
      });
    }

    // 2. Attach validated data (do not mutate raw request properties)
    req.valid = result.data;
    next();
  };
}

// 3. Schema with strict unknown-field rejection
const CreateUserSchema = z.object({
  username: z.string().min(3).max(20),
  email: z.email(),
  password: z.string().min(8)
}).strict(); // rejects unknown fields (mass assignment prevention)

app.post('/users', validate(CreateUserSchema), (req, res) => {
  // req.valid.body is typed and sanitized
  res.status(201).json({ user: req.valid.body });
});
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Boundary validation | Factory middleware before the handler |
| All sources | Validate `body`, `query`, and `params` together |
| Safe parsing | `.safeParse()` avoids uncaught throws |
| Unknown-field rejection | `.strict()` (Zod) or `stripUnknown: true` (Joi) |
| Structured errors | `issues` mapped to `{ field, message }` |
| Validated data | `req.valid` or `req.body = result.data` |

#### Syntax Rules

- **Use factories** — Write `validate(schema)` once; reuse it across routes.
- **Validate all sources** — Body, query, and params; extend to headers if needed.
- **Use safe parsing** — `.safeParse()` (Zod) or `{ error, value }` (Joi) instead of throwing validators.
- **Aggregate errors** — Use `abortEarly: false` (Joi) or `safeParse()` (Zod) to collect all failures.
- **Strip unknown fields** — `.strict()` (Zod) or `stripUnknown: true` (Joi) to prevent mass assignment.
- **Return field-level errors** — Never send a vague “Validation failed” without details.
- **Pair with database constraints** — Validation is the first line of defense; unique indexes and foreign keys are the last.

#### Constraints and Limitations

- `z.coerce.boolean()` uses truthiness; `"false"` becomes `true` — use `.transform()` for precise boolean parsing.
- `abortEarly: false` may produce a large error array for severely malformed input; consider limiting the number of reported errors for public APIs.
- Custom async validators add latency; database lookups should be indexed and efficient.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Complete Validation Pipeline with Factory and Structured Errors

```typescript
import express from 'express';
import { z } from 'zod';

const app = express();
app.use(express.json());

// Reusable middleware factory
function validate<T extends z.ZodTypeAny>(schema: T) {
  return (req: express.Request, res: express.Response, next: express.NextFunction) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      const errors = result.error.issues.map((i) => ({
        field: i.path.join('.'),
        message: i.message
      }));
      return res.status(400).json({
        error: { code: 'VALIDATION_ERROR', message: 'Invalid input', details: errors }
      });
    }
    req.body = result.data;
    next();
  };
}

// Schema with strict unknown-field rejection
const CreateUserSchema = z.object({
  username: z.string().min(3).max(20),
  email: z.email(),
  password: z.string().min(8)
}).strict();

app.post('/users', validate(CreateUserSchema), (req, res) => {
  res.status(201).json({ user: req.body });
});

app.listen(3000);
```

**Expected Output**

```
POST /users  { "username": "alice", "email": "alice@example.com", "password": "secret123" }
→ 201 { "user": { "username": "alice", "email": "alice@example.com", "password": "secret123" } }

POST /users  { "username": "alice", "email": "alice@example.com", "password": "secret123", "isAdmin": true }
→ 400 { "error": { "code": "VALIDATION_ERROR", "message": "Invalid input", "details": [
  { "field": "", "message": "Unrecognized key(s) in object: 'isAdmin'" }
] } }

POST /users  { "username": "a", "email": "invalid", "password": "123" }
→ 400 { "error": { "code": "VALIDATION_ERROR", "message": "Invalid input", "details": [
  { "field": "username", "message": "Too small: expected string to have >=3 characters" },
  { "field": "email", "message": "Invalid email address" },
  { "field": "password", "message": "Too small: expected string to have >=8 characters" }
] } }
```

**Why This Output Occurs**

- The factory validates with `safeParse`; `.strict()` rejects the unexpected `isAdmin` field, preventing mass assignment. All field-level errors are collected and returned together.

---

### Real-World Cases

- **Production APIs** — A validation factory applied to every route ensures consistent validation and error formatting.
- **Security-sensitive applications** — Strict schemas prevent attackers from injecting fields like `isAdmin` or `tenantId`.
- **Full-stack TypeScript** — Shared Zod schemas between frontend forms and backend handlers ensure the error contract matches.

---

## References

- Custom validators/sanitizers — express-validator — https://express-validator.github.io/docs/6.11.0/custom-validators-sanitizers/
- Customizing express-validator — https://express-validator.github.io/docs/next/guides/customizing/
- Validation Chain API — express-validator — https://express-validator.github.io/docs/6.0.0/validation-chain-api/#customvalidator
- express-validator 实战：Express 数据校验完全指南 — https://www.guance.com/learn/articles/express-validator-nodejs
- Validation middleware — Scrimba — https://docs.scrimba.com/cybersecurity/validation-middleware
- 5 Express.js Middleware Patterns You'll Use in Every App — DEV Community — https://dev.to/armorbreak/5-expressjs-middleware-patterns-youll-use-in-every-app-4c4k#1
- Include Validation Error Details — voku/agent-skills — https://github.com/voku/agent-skills/blob/main/skills/api-design-patterns/rules/error-validation-details.md#1
- api-validation-error-handler — npm — https://www.npmjs.com/package/api-validation-error-handler
- Typed-Error Middleware with Express — Steve Kinney — https://stevekinney.com/courses/full-stack-typescript/typed-error-middleware-with-express
- Zod Documentation — https://zod.dev/
- Joi Documentation — https://joi.dev/
- OWASP Input Validation Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
- OWASP Mass Assignment Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html