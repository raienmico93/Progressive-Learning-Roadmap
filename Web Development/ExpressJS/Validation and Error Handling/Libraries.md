# Validation Libraries — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Validation libraries are third-party packages that provide declarative, schema-based mechanisms for verifying that external input — such as HTTP request bodies, query parameters, and route parameters — conforms to expected types, formats, and constraints before it is used by application logic.

**Technical Definition:** Validation libraries abstract the repetitive task of manually checking `typeof`, regex matching, and boundary enforcement into reusable schema definitions. In Express, they are typically integrated as middleware that runs before controllers, rejecting invalid requests with structured error responses. The three dominant libraries — Zod, Joi, and express-validator — differ in their API design, TypeScript integration, and ecosystem positioning. Zod is a TypeScript-first schema validation library with static type inference. Joi is an object schema description language and validator for JavaScript objects, letting you describe data using a simple, intuitive, and readable language. express-validator is a set of Express.js middlewares that wraps validator.js validator and sanitizer functions.

**Beginner-Friendly Explanation:** Validation libraries are like having a team of specialist inspectors instead of one generalist. Each inspector (library) has their own method: Zod uses TypeScript to check types before your code even runs, Joi describes validation rules in plain English-like chains, and express-validator acts as a security guard at the Express door. Instead of writing hundreds of `if (typeof x !== 'string')` checks, you describe what valid data looks like, and the library does the checking for you.

### Key Characteristics

- **Declarative schemas:** Validation rules are defined as data structures, not imperative if-else chains.
- **Type safety:** Zod infers TypeScript types from schemas automatically, eliminating manual type declarations.
- **Middleware integration:** express-validator and Joi (via custom middleware) integrate directly into Express's request pipeline.
- **Sanitisation:** Libraries also transform and clean input (trimming, escaping, type coercion).
- **Structured errors:** Validation failures produce machine-readable error objects with field paths and messages.
- **Ecosystem maturity:** Combined weekly downloads exceed 220 million across the three libraries.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x; v14+ for express-validator).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript or TypeScript knowledge:** Functions, objects, and asynchronous programming.
- **A validation library:** `npm install zod` or `npm install joi` or `npm install express-validator`.

### Related Programming Areas

- **Input Validation:** The direct application of validation libraries at the request boundary.
- **Error Handling:** Validation failures feed into global error-handling middleware.
- **Dependency Injection:** Validation schemas can be injected into middleware factories.
- **Schema-Driven Validation:** Sharing the same schema between frontend and backend to reduce duplicate logic.
- **Type Systems:** Zod's type inference bridges runtime validation and compile-time TypeScript types.

### Core Concepts

1. **Zod** — type-safe validation with deep, native TypeScript inference.
2. **Joi** — object schema validation widely used across JavaScript and older Express codebases.
3. **express-validator** — middleware-driven validation wrapping validator.js.
4. **Schema-Driven Validation** — centralising schemas to share structural validation between frontend and backend.

---

## Core Concept 1: Zod

### Definitions

**Core Definition:** Zod is a TypeScript-first schema declaration and validation library that automatically infers static TypeScript types from schema definitions, providing both runtime validation and compile-time type safety from a single source of truth.

**Technical Definition:** Zod is a TypeScript-first schema validation library with static type inference. You define a schema using Zod's primitive and composite types, and Zod infers the corresponding TypeScript type via `z.infer<typeof Schema>`. For parsing, `schema.parse(data)` validates an input and returns a strongly-typed deep clone; if validation fails, it throws a `ZodError` with granular information. The `schema.safeParse(data)` method returns a plain result object containing either the parsed data or a `ZodError`, avoiding `try/catch` blocks. Zod also provides `z.coerce` for automatic type coercion, `.transform()` for data conversion, and `.refine()` for custom validation logic.

**Beginner-Friendly Explanation:** Zod is like a bilingual translator who speaks both TypeScript (at compile time) and JavaScript (at runtime). You write one schema, and Zod tells TypeScript what type the data will be, while also checking at runtime that the data actually matches. If you define `z.string()`, TypeScript knows the result is a string, and Zod will reject any number or boolean at runtime.

### Purposes

- To provide type-safe validation with deep, native TypeScript inference capabilities.
- To eliminate the gap between runtime validation and compile-time type checking.
- To parse and transform external data into trusted, strongly-typed internal structures.
- To offer a single source of truth for both schema definition and type derivation.

### Syntax Rules and Structure

#### Schema Definition

```typescript
import * as z from "zod";

const UserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().int().positive().optional(),
  role: z.enum(['user', 'admin']).default('user')
});

type User = z.infer<typeof UserSchema>;
// Type: { name: string; email: string; age?: number; role: 'user' | 'admin' }
```

| Component | Breakdown |
|-----------|-----------|
| `z.object({...})` | Defines an object schema. |
| `z.string().min(1).max(100)` | String with length constraints. |
| `z.number().int().positive()` | Integer with positivity constraint. |
| `.optional()` | Field may be absent or undefined. |
| `.default(value)` | Fallback if field is absent. |
| `z.infer<typeof Schema>` | Extracts the TypeScript type from the schema. |

#### Parsing Methods

| Method | Behaviour | Use Case |
|--------|-----------|----------|
| `.parse(data)` | Returns parsed data or throws `ZodError`. | When you want exceptions. |
| `.safeParse(data)` | Returns `{ success, data | error }`. | In Express middleware. |
| `.validate(data)` | Returns boolean; no error object built. | Quick checks, up to 16x faster. |
| `.parseAsync(data)` | For schemas with async refinements/transforms. | Async validation. |

#### Coercion

```typescript
const PaginationSchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  limit: z.coerce.number().int().max(100).default(20)
});
// Input: { page: "2", limit: "10" } → Output: { page: 2, limit: 10 }
```

**Rules:**
- `z.coerce` uses JavaScript constructors: `Number()`, `String()`, `Boolean()`, `BigInt()`, `new Date()`.
- Boolean coercion follows truthiness: `"false"` is `true`, `""` is `false`, `0` is `false`.
- Use `.transform()` for custom coercion logic when `z.coerce` is insufficient.

**Constraints:**
- `z.coerce.boolean()` follows truthiness rules, which may surprise developers expecting `"false"` to be `false`.
- Async refinements require `.parseAsync()` or `.safeParseAsync()`.

### Annotated Code Examples

#### Example 1: Basic Validation with safeParse

```typescript
// validators/user.validator.ts
import { z } from 'zod';

export const CreateUserSchema = z.object({
  name: z.string().min(1, 'Name is required').max(100),
  email: z.string().email('Invalid email format'),
  password: z.string().min(8, 'Password must be at least 8 characters')
});
```

```typescript
// middleware/validate.ts
import { ZodSchema } from 'zod';
import { Request, Response, NextFunction } from 'express';

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
    req.valid = result.data;  // Parsed, typed data
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
  "data": { "id": "1", "name": "Alice", "email": "alice@example.com" }
}
```

**Expected Output (for invalid request):**
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

**Why this output:** `safeParse` returns a discriminated union. If `success` is `true`, `result.data` contains the parsed object. If `false`, `result.error.flatten()` produces a flat object keyed by field name. The controller is never invoked when validation fails.

#### Example 2: Transformation and Refinement

```typescript
const EventSchema = z.object({
  title: z.string().min(1),
  startDate: z.string().datetime().transform((s) => new Date(s)),
  endDate: z.string().datetime().transform((s) => new Date(s))
}).refine(
  (data) => data.endDate >= data.startDate,
  { message: 'End date must be after start date', path: ['endDate'] }
);
```

**Expected Output (for valid dates):**
```json
{
  "title": "Meetup",
  "startDate": "2026-02-15T18:00:00.000Z",
  "endDate": "2026-02-15T20:00:00.000Z"
}
```

**Expected Output (for invalid date range):**
```json
{ "endDate": ["End date must be after start date"] }
```

**Why this output:** `.transform()` converts ISO date strings into `Date` objects during parsing. The `.refine()` check compares the transformed `Date` values. The output type has `Date` instances, not strings.

### Real-World Cases

- **API request validation:** Validating POST/PUT/PATCH bodies in Express APIs.
- **Shared frontend/backend schemas:** Using the same Zod schema in React/Vue forms and Express routes.
- **Configuration validation:** Validating environment variables against a Zod schema at startup.
- **Type generation:** Inferring TypeScript types from schemas instead of writing manual interfaces.

---

## Core Concept 2: Joi

### Definitions

**Core Definition:** Joi is an object schema description language and validator for JavaScript objects, allowing developers to create blueprints for JavaScript objects to ensure validation of key information using a simple, intuitive, and readable language.

**Technical Definition:** Joi lets you describe your data using a simple, intuitive, and readable language. A schema is constructed using provided types and constraints, and the value is validated against the schema using `schema.validate(value)`. Joi schemas are immutable — every additional rule returns a new schema object. Joi provides over 150 built-in validators across strings, numbers, dates, arrays, objects, and binaries, with chainable rules that read like English. The `validate()` method returns `{ error, value }`; if valid, `error` is `undefined`.

**Beginner-Friendly Explanation:** Joi is like a specification document for your data. You write rules like "username must be alphanumeric, at least 3 characters, at most 30, and required" — and Joi enforces every rule. It's been around since the early days of Node.js, so it's widely used in older Express codebases and enterprise applications.

### Purposes

- To provide object schema validation widely used across JavaScript and older Express codebases.
- To offer clear, human-readable error messages for validation failures.
- To handle complex validation scenarios: conditional validation (`.when()`), cross-field references, and alternatives.
- To support type coercion (e.g., converting strings to numbers).

### Syntax Rules and Structure

#### Schema Definition

```javascript
const Joi = require('joi');

const userSchema = Joi.object({
  username: Joi.string().alphanum().min(3).max(30).required(),
  email: Joi.string().email({ minDomainSegments: 2, tlds: { allow: ['com', 'net'] } }),
  password: Joi.string().pattern(new RegExp('^[a-zA-Z0-9]{3,30}$')),
  age: Joi.number().integer().min(18).max(120),
  role: Joi.string().valid('user', 'admin').default('user')
}).with('username', 'birth_year').xor('password', 'access_token');
```

| Component | Breakdown |
|-----------|-----------|
| `Joi.object({...})` | Defines an object schema. |
| `Joi.string().alphanum().min(3).max(30).required()` | String with constraints. |
| `.with('a', 'b')` | Requires `a` to be accompanied by `b`. |
| `.xor('a', 'b')` | Requires exactly one of `a` or `b`. |
| `.valid(...)` | Allow-list of values. |
| `.default(value)` | Fallback if absent. |

#### Validation

```javascript
const { error, value } = userSchema.validate(userData, {
  abortEarly: false,     // Return all errors
  stripUnknown: true     // Remove unknown keys
});
if (error) {
  const errors = error.details.map(d => ({
    field: d.path.join('.'),
    message: d.message
  }));
}
```

#### Common Validation Rules

| Type | Methods |
|------|---------|
| String | `.alphanum()`, `.email()`, `.uri()`, `.pattern(regex)`, `.min(n)`, `.max(n)` |
| Number | `.integer()`, `.min(n)`, `.max(n)`, `.positive()`, `.precision(n)` |
| Date | `.max('now')`, `.greater('now')`, `.iso()` |
| Array | `.items(type)`, `.min(n)`, `.max(n)` |
| Object | `.keys({...})`, `.unknown(true)` |
| Alternatives | `Joi.alternatives().try(Joi.string(), Joi.number())` |

**Rules:**
- Values (and keys in objects) are **optional by default** — use `.required()` to enforce presence.
- Schemas are **immutable** — each rule returns a new schema.
- `abortEarly: false` returns all validation errors, not just the first.
- `stripUnknown: true` removes keys not defined in the schema.

**Constraints:**
- Joi does not provide TypeScript type inference natively (unlike Zod).
- Large schemas may have a small performance overhead compared to Zod.

### Annotated Code Example

```javascript
// middleware/validate.js
const Joi = require('joi');

function validate(schema) {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, {
      abortEarly: false,
      stripUnknown: true
    });
    if (error) {
      return res.status(400).json({
        success: false,
        errors: error.details.map(d => ({
          field: d.path.join('.'),
          message: d.message
        }))
      });
    }
    req.body = value;
    next();
  };
}

// routes/user.routes.js
const createUserSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required(),
  username: Joi.string().alphanum().min(3).max(30).required()
});

app.post('/api/users', validate(createUserSchema), (req, res) => {
  res.json({ success: true, user: req.body });
});
```

**Expected Output (for valid request):**
```json
{
  "success": true,
  "user": {
    "email": "alice@example.com",
    "password": "SecurePass123",
    "username": "alice"
  }
}
```

**Expected Output (for invalid request):**
```json
{
  "success": false,
  "errors": [
    { "field": "email", "message": "\"email\" must be a valid email" },
    { "field": "password", "message": "\"password\" length must be at least 8 characters long" }
  ]
}
```

**Why this output:** The Joi schema validates each field against its constraints. With `abortEarly: false`, all errors are collected. The `stripUnknown: true` option removes any extra fields not defined in the schema. The parsed `value` replaces `req.body` so the controller receives clean data.

### Real-World Cases

- **Enterprise Express APIs:** Joi is widely adopted in older and enterprise codebases.
- **Complex conditional validation:** `.when()` for fields that depend on other fields' values.
- **Cross-field validation:** `.xor()`, `.with()`, and `.without()` for relationship constraints.
- **Configuration validation:** Validating environment variables and configuration objects.

---

## Core Concept 3: express-validator

### Definitions

**Core Definition:** express-validator is a set of Express.js middlewares that wraps validator.js validator and sanitizer functions, allowing developers to validate and sanitize Express requests using chainable method calls directly in route definitions.

**Technical Definition:** express-validator is a set of Express.js middlewares that wraps the extensive collection of validators and sanitizers offered by validator.js. It provides functions like `body()`, `query()`, `param()`, and `check()` that create validation chains. Each chain can include validators (e.g., `.isEmail()`, `.isLength()`) and sanitizers (e.g., `.trim()`, `.escape()`). The `validationResult(req)` function collects errors from the request object and returns them in a structured format. The library validates `req.body`, `req.cookies`, `req.headers`, `req.params`, and `req.query`.

**Beginner-Friendly Explanation:** express-validator is like a security guard that checks each visitor's documents at the door. You tell the guard "check that this person's ID is an email address" and "check that their password is at least 5 characters" — and the guard does exactly that. It's built specifically for Express, so it integrates directly into your routes.

### Purposes

- To provide middleware-driven validation wrapping the imperative validator.js library.
- To validate and sanitize Express requests with chainable method calls.
- To integrate validation directly into route definitions for maximum visibility.
- To leverage validator.js's extensive collection of 100+ validators and sanitizers.

### Syntax Rules and Structure

#### Basic Validation

```javascript
const { body, validationResult } = require('express-validator');

app.post('/user',
  body('username').isEmail(),
  body('password').isLength({ min: 5 }),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    // Process valid request
  }
);
```

| Component | Breakdown |
|-----------|-----------|
| `body('username')` | Selects the `username` field in `req.body`. |
| `.isEmail()` | validator.js validator. |
| `.isLength({ min: 5 })` | Length constraint. |
| `validationResult(req)` | Collects all errors from the request. |
| `.array()` | Returns errors as an array of objects. |

#### Location-Specific Functions

| Function | Validates |
|----------|-----------|
| `body('field')` | `req.body.field` |
| `query('field')` | `req.query.field` |
| `param('field')` | `req.params.field` |
| `check('field')` | All locations, in order: params, query, body |
| `header('field')` | `req.headers.field` |

#### Sanitization

```javascript
body('email').trim().normalizeEmail().isEmail(),
body('name').trim().escape(),
body('age').toInt()
```

| Sanitizer | Effect |
|-----------|--------|
| `.trim()` | Removes leading/trailing whitespace. |
| `.escape()` | Escapes HTML characters. |
| `.normalizeEmail()` | Normalizes email format. |
| `.toInt()` | Converts to integer. |
| `.toBoolean()` | Converts to boolean. |

**Rules:**
- Validation chains are middleware — they can be passed directly to route handlers.
- `validationResult(req)` must be called in the route handler to check for errors.
- Sanitizers run **after** validators in the chain by default.
- `check()` searches all request locations in order: params, query, body.

**Constraints:**
- express-validator does not provide TypeScript type inference.
- The error format is different from Zod and Joi — it uses `{ location, msg, param }` objects.
- Sanitizers modify `req` in place — use with caution in middleware chains.

### Annotated Code Example

```javascript
// routes/user.routes.js
const { body, query, param, validationResult } = require('express-validator');

app.post('/api/users',
  // Validation and sanitization chains
  body('username')
    .trim()
    .isEmail().withMessage('Username must be a valid email'),
  body('password')
    .isLength({ min: 5 }).withMessage('Password must be at least 5 characters'),
  body('age')
    .optional()
    .isInt({ min: 18, max: 120 }).withMessage('Age must be between 18 and 120')
    .toInt(),
  body('name')
    .trim()
    .escape()
    .isLength({ min: 1, max: 100 }),
  // Route handler
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({
        success: false,
        errors: errors.array().map(e => ({
          field: e.param,
          message: e.msg
        }))
      });
    }
    res.status(201).json({
      success: true,
      user: {
        email: req.body.username,
        name: req.body.name,
        age: req.body.age
      }
    });
  }
);

// Query validation for pagination
app.get('/api/users',
  query('page').optional().isInt({ min: 1 }).toInt(),
  query('limit').optional().isInt({ min: 1, max: 100 }).toInt(),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    res.json({ page: req.query.page || 1, limit: req.query.limit || 20 });
  }
);

// Param validation for UUID
app.get('/api/users/:id',
  param('id').isUUID().withMessage('ID must be a valid UUID'),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    res.json({ userId: req.params.id });
  }
);
```

**Expected Output (for valid POST request):**
```json
{
  "success": true,
  "user": { "email": "alice@example.com", "name": "Alice", "age": 30 }
}
```

**Expected Output (for invalid POST request):**
```json
{
  "success": false,
  "errors": [
    { "field": "username", "message": "Username must be a valid email" },
    { "field": "password", "message": "Password must be at least 5 characters" }
  ]
}
```

**Expected Output (for `GET /api/users?page=abc`):**
```json
{
  "errors": [
    { "location": "query", "msg": "Invalid value", "param": "page" }
  ]
}
```

**Why this output:** Each validation chain runs as middleware before the route handler. `validationResult(req)` collects all errors. Sanitizers (`.trim()`, `.escape()`, `.toInt()`) modify the request data in place. The route handler checks `errors.isEmpty()` and either returns a 400 with details or proceeds with the sanitized data.

### Real-World Cases

- **Express-first APIs:** When you want validation logic visible directly in route definitions.
- **Simple validation scenarios:** Email, password, and basic field checks.
- **Sanitization pipelines:** Trimming, escaping, and normalizing input before storage.
- **Legacy Express codebases:** Already using validator.js and wanting middleware integration.

---

## Core Concept 4: Schema-Driven Validation

### Definitions

**Core Definition:** Schema-driven validation is an architectural approach where a single schema definition serves as the authoritative contract for data structure and validation rules across both frontend and backend applications, eliminating duplicated validation logic.

**Technical Definition:** Schema-driven validation treats the data definition as the core driver of the application. Instead of hardcoding validation rules directly inside UI components, the rules live in a central schema that determines how the rest of the app behaves. The schema acts as a shared contract — when a field is added or a validation rule changes, both the frontend and backend update automatically. This is typically implemented using a library like Zod (for TypeScript projects) or JSON Schema (for framework-agnostic projects), with the schema shared in a monorepo package or published as an npm module.

**Beginner-Friendly Explanation:** Schema-driven validation is like having a single master blueprint for a house that both the electrician and the plumber follow. Instead of the electrician making their own copy and the plumber making another, they both read from the same blueprint. When the architect changes the blueprint, both trades update their work automatically — no more mismatched outlets and pipes.

### Purposes

- To centralise schemas to share structural validation between frontend clients and backend APIs.
- To reduce duplicate logic — write validation rules once, use them everywhere.
- To eliminate inconsistencies where the frontend accepts data the backend rejects.
- To enable automatic TypeScript type inference across the entire stack.
- To make schema changes propagate automatically to all consumers.

### Syntax Rules and Structure

#### Shared Schema Module

```
packages/
├── schema/
│   ├── package.json
│   └── src/
│       ├── user.schema.ts
│       └── index.ts
├── frontend/
│   └── src/
│       └── forms/
│           └── UserForm.tsx
└── backend/
    └── src/
        └── routes/
            └── user.routes.ts
```

#### Shared Schema Definition

```typescript
// packages/schema/src/user.schema.ts
import { z } from 'zod';

export const CreateUserSchema = z.object({
  name: z.string().min(1, 'Name is required').max(100),
  email: z.string().email('Invalid email format'),
  password: z.string().min(8, 'Password must be at least 8 characters')
});

export type CreateUserInput = z.infer<typeof CreateUserSchema>;
```

#### Backend Usage

```typescript
// backend/src/routes/user.routes.ts
import { CreateUserSchema } from '@myapp/schema';
import { validateBody } from '../middleware/validate';

router.post('/users', validateBody(CreateUserSchema), userController.create);
```

#### Frontend Usage

```typescript
// frontend/src/forms/UserForm.tsx
import { CreateUserSchema, CreateUserInput } from '@myapp/schema';

const form = useForm<CreateUserInput>({
  resolver: zodResolver(CreateUserSchema)
});
```

| Component | Breakdown |
|-----------|-----------|
| `packages/schema/` | Shared workspace package containing schemas. |
| `CreateUserSchema` | Single Zod schema used by both layers. |
| `CreateUserInput` | Inferred TypeScript type from the schema. |
| `zodResolver` | Frontend form library integration. |
| `validateBody` | Backend Express middleware using the same schema. |

**Rules:**
- The schema must be defined in a shared package accessible to both frontend and backend.
- Zod is preferred for TypeScript projects because it provides type inference.
- JSON Schema can be used for framework-agnostic sharing (e.g., Vue + Express with Ajv).
- Changes to the schema must be versioned to avoid breaking existing clients.

**Constraints:**
- Frontend and backend may have different validation needs (e.g., password confirmation is frontend-only).
- Shared schemas require a monorepo or published package setup.
- Schema versioning becomes critical when clients are deployed independently.

### Annotated Code Example

```typescript
// packages/schema/src/auth.schema.ts
import { z } from 'zod';

export const LoginSchema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(1, 'Password is required')
});

export const RegisterSchema = LoginSchema.extend({
  name: z.string().min(1, 'Name is required').max(100),
  confirmPassword: z.string()
}).refine(
  (data) => data.password === data.confirmPassword,
  { message: 'Passwords do not match', path: ['confirmPassword'] }
);

export type LoginInput = z.infer<typeof LoginSchema>;
export type RegisterInput = z.infer<typeof RegisterSchema>;
```

```typescript
// backend/src/middleware/validate.ts
import { ZodSchema } from 'zod';

export function validateBody(schema: ZodSchema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(422).json({
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
// backend/src/routes/auth.routes.ts
import { RegisterSchema } from '@myapp/schema';
import { validateBody } from '../middleware/validate';

router.post('/auth/register', validateBody(RegisterSchema), authController.register);
router.post('/auth/login', validateBody(LoginSchema), authController.login);
```

```typescript
// frontend/src/pages/Register.tsx
import { RegisterSchema, RegisterInput } from '@myapp/schema';
import { zodResolver } from '@hookform/resolvers/zod';

function RegisterForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<RegisterInput>({
    resolver: zodResolver(RegisterSchema)
  });
  // The same validation rules apply on the client
}
```

**Expected Output (frontend — invalid password confirmation):**
```json
{
  "confirmPassword": ["Passwords do not match"]
}
```

**Expected Output (backend — same invalid data):**
```json
{
  "error": "Validation failed",
  "details": {
    "confirmPassword": ["Passwords do not match"]
  }
}
```

**Why this output:** The same `RegisterSchema` is imported by both the frontend form (via `zodResolver`) and the backend middleware (via `validateBody`). The password confirmation rule is defined once and enforced identically on both sides. Changing the rule requires editing only the shared schema.

### Real-World Cases

- **Monorepo applications:** Frontend and backend in the same repository, sharing a `packages/schema` module.
- **Full-stack TypeScript:** Using Zod for end-to-end type safety from form to database.
- **Micro-frontends:** Multiple frontends sharing validation schemas with a common backend.
- **API contracts:** Publishing schemas as npm packages for external consumers.

---

## References

- Zod Documentation — https://zod.dev
- Zod — Basic Usage — https://zod.dev/basics
- Zod — Defining Schemas — https://zod.dev/api
- Zod — Parsing — https://zod.dev/basics?id=parsing-data
- Zod npm — https://www.npmjs.com/package/zod
- Joi API v17 — https://joi.dev/api/17.x.x
- Joi npm — https://www.npmjs.com/package/joi
- How to use Joi for validation in Node.js — CoreUI — https://coreui.io/answers/how-to-use-joi-for-validation-in-nodejs/
- Joi npm: Security Review and Safe Usage Guide — safeguard.sh — https://safeguard.sh
- express-validator Documentation — https://express-validator.github.io/docs/
- express-validator — Getting Started — https://express-validator.github.io/docs/6.12.0/
- express-validator — Schema Validation — https://express-validator.github.io/docs/schema-validation/
- express-validator — check API — https://express-validator.github.io/docs/api/check/
- express-validator npm — https://www.npmjs.com/package/express-validator
- Validazione Input in Node.js: Zod, Joi, express-validator [2026] — https://shattered.io/it/validazione-input-nodejs/
- Stop fighting forms: The schema-driven approach to validation — LogRocket Blog — https://blog.logrocket.com/stop-fighting-schema-driven-form-validation
- Shared Zod Validation for Backend and Frontend — GitHub — https://github.com/ritik4ever/stellar-stream/pull/46
- Shared Validation Library for End-to-End Type Safety — GitHub — https://github.com/alx-sch/GRIT/issues/59
- @atomictemplate/validations — npm — https://www.npmjs.com/package/@atomictemplate/validations
- Zod vs Joi vs express-validator — npm-compare.com — https://npm-compare.com/express-validator,joi,sherif,validator,yup,zod
- validator.js — GitHub — https://github.com/validatorjs/validator.js
- OWASP — Top Ten 2021: A03 Injection — https://owasp.org/Top10/A03_2021-Injection/